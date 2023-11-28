## 背景

CRM 内存 查询缓存占用 4502 MB 
![[Pasted image 20231107132639.png]]
  

查询主要是 authorityObjectEntity 对象的查看的缓存。

![[Pasted image 20231107132703.png]]

## 历史的方案

[正式服任务处理速度严重下降问题排查](https://kms.fineres.com/pages/viewpage.action?pageId=102012619)

swift 之前遇到的内容 ， 属于是 IN 参数过多 ，由于Swift 的业务逻辑 ， IN 参数的数量和参数内容都不一样，导致的 query plan cache 过大。

但是在目录这边还不一样。

  

swift 的 hibernate 体系的查询和平台是分开的，swift 后面因为这个场景，已经在配置文件中优化了一下
![[企业微信截图_16993220193552.png]]
在平台这边还是没有使用这个参数。

## 目录这边的原因

目录这边主要是因为长SQL拼接导致的参数过多。

主要是因为在对角色鉴权，查到了关联到的所有的目录信息 ，每一个目录都需要查对应的所有的子目录 ， 所以拼接的SQL很长。

主要在 [com.fr](http://com.fr).decision.authority.controller.AbstractController#subQueryAllChildIdsByAuthorityId

  

对于 hibernate 的查询缓存，会按照hsql 里面的参数数量和参数内容来判断是不是需要再次缓存 ， 如果系统中，每一个人、角色查到的目录数量都不同，那么缓存的hsql 就会很多。

查询缓存主要是对 hsql 的内容、 hsql 的参数值、查询到的结果的主键ID进行缓存 。

  

那么对于长SQL ， 和历史的 IN 参数的场景不同 ，此类这个参数可能优化效果就很有限。

  

## 方案原理

在 WEB-INF/config/db.properties 文件中添加

|   |
|---|
|hibernate.query.in_clause_parameter_padding=true|

这个参数在swift那边已经验证了 ， 也是hibernate 提供的优化方案 ， 主要是通过添加虚拟的参数来模拟in参数相同的场景

即hibernate 将参数的数量进行修改 ， 之前为 in (1,2,3,4,5) 和  in (1,2,3,4,5,6)  修改为 in(1,2,3,4,5,5,5,5) in (1,2,3,4,5,6,6,6)  , 来固定参数数量，这样HSQL 语句就只有一个了，只有参数列表不同。

|   |
|---|
|By default, the IN clause expands to include all bind parameter values. However, for database systems supporting execution plan caching, there's a better chance of hitting the cache if the number of possible IN clause parameters lowers. For this reason, we can expand the bind parameters to power-of-two: 4, 8, 16, 32, 64. This way, an IN clause with 5, 6, or 7 bind parameters will use the 8 IN clause, therefore reusing its execution plan. If you want to activate this feature, you need to set this property to true. The default value is false.  <br>Since: 5.2.17|

  
**优化前的HSQL，需要缓存四次**
```sql
`Query:["`

    `SELECT  p.id AS id1_0_, p.title AS title2_0_`

    `FROM    post p`

    `WHERE   p.id IN (? , ? , ?)`

`"],`

`Params:[`

    `1``,` `2``,` `3`

`]`

`Query:["`

    `SELECT  p.id AS id1_0_, p.title AS title2_0_`

    `FROM    post p`

    `WHERE   p.id IN (?, ?, ?, ?)`

`"],`

`Params:[`

    `1``,` `2``,` `3``,` `4`

`]`

`Query:["`

    `SELECT  p.id AS id1_0_, p.title AS title2_0_`

    `FROM    post p`

    `WHERE   p.id IN (? , ? , ? , ? , ?)`

`"],`

`Params:[`

    `1``,` `2``,` `3``,` `4``,` `5`

`]`

`Query:["`

    `SELECT  p.id AS id1_0_, p.title AS title2_0_`

    `FROM    post p`

    `WHERE   p.id IN (? , ? , ? , ? , ? , ?)`

`"],`

`Params:[`

    `1``,` `2``,` `3``,` `4``,` `5``,` `6`

`]`
```

**优化后的HSQL ，缓存两次**
```sql
`Query:["`

    `SELECT  p.id AS id1_0_, p.title AS title2_0_`

    `FROM    post p`

    `WHERE   p.id IN (?, ?, ?, ?)`

`"],`

`Params:[`

    `1``,` `2``,` `3``,` `3`

`]`

`Query:["`

    `SELECT  p.id AS id1_0_, p.title AS title2_0_`

    `FROM    post p`

    `WHERE   p.id IN (?, ?, ?, ?)`

`"],`

`Params:[`

    `1``,` `2``,` `3``,` `4`

`]`

`Query:["`

    `SELECT  p.id AS id1_0_, p.title AS title2_0_`

    `FROM    post p`

    `WHERE   p.id IN (? , ? , ? , ? , ? , ? , ? , ?)`

`"],`

`Params:[`

    `1``,` `2``,` `3``,` `4``,` `5``,` `5``,` `5``,` `5`

`]`

`Query:["`

    `SELECT  p.id AS id1_0_, p.title AS title2_0_`

    `FROM    post p`

    `WHERE   p.id IN (? , ? , ? , ? , ? , ? , ? , ?)`

`"],`

`Params:[`

    `1``,` `2``,` `3``,` `4``,` `5``,` `6``,` `6``,` `6`

`]`
```

结合之前 Swift 已经使用 和 原理 ， 这个参数对性能没有影响，会优化一下 缓存占比大小。

## 其他查询缓存参数

|   |
|---|
|# The maximum number of entries including ， 默认 2GB<br>hibernate.query.plan_cache_max_size<br><br># The maximum number of soft references maintained by org.hibernate.engine.query.spi.QueryPlanCache. Default is 2048.<br>hibernate.query.plan_cache_max_soft_references<br><br># The maximum number of strong references maintained by org.hibernate.engine.query.spi.QueryPlanCache. Default is 128.<br>hibernate.query.plan_cache_max_strong_references<br><br># When you use javax.persistence.InheritanceType.JOINED strategy for inheritance mapping and query a value from an entity, all superclass tables are joined in the query regardless you need them. With this setting set to true only superclass tables which are really needed are joined.<br>The default value is true.<br>hibernate.query.omit_join_of_superclass_tables|

## 结论

可以选择将 参数 hibernate.query.in_clause_parameter_padding 加上， 其将优化关于IN 参数的场景 （我们使用IN参数的场景也很多） ， 对于长SQL 里面的参数，设计到IN逻辑的可以优化。

## 后续

...


