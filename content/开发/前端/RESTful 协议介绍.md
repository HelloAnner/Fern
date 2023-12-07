### REST介绍

REST（英文：Representational State Transfer，简称REST，直译过来表现层状态转换）是一种软件架构风格、设计风格，而不是标准，只是提供了一组设计原则和约束条件。它主要用于客户端和服务器交互类的软件。基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存等机制。

RESTful是一种风格而不是标准，而这个风格大致有以下几个主要 **特征** ：

**以资源为基础**  ：资源可以是一个图片、音乐、一个XML格式、HTML格式或者JSON格式等网络上的一个实体，除了一些二进制的资源外普通的文本资源更多以JSON为载体、面向用户的一组数据(通常从数据库中查询而得到)。  
**统一接口** : 对资源的操作包括获取、创建、修改和删除，这些操作正好对应HTTP协议提供的GET、POST、PUT和DELETE方法。换言而知，使用RESTful风格的接口但从接口上你可能只能定位其资源，但是无法知晓它具体进行了什么操作，需要具体了解其发生了什么操作动作要从其HTTP请求方法类型上进行判断。具体的HTTP方法和方法含义如下：

- GET（SELECT）：从服务器取出资源（一项或多项）。
- POST（CREATE）：在服务器新建一个资源。
- PUT（UPDATE）：在服务器更新资源（客户端提供完整资源数据）。
- PATCH（UPDATE）：在服务器更新资源（客户端提供需要修改的资源数据）。
- DELETE（DELETE）：从服务器删除资源。


#### REST架构限制条件

Fielding在论文中提出REST架构的6个 **限制条件** ，也可称为RESTful 6大原则， 标准的REST约束应满足以下6个原则：

**客户端-服务端（Client-Server）** : 这个更专注客户端和服务端的分离，服务端独立可更好服务于前端、安卓、IOS等客户端设备。

**无状态（Stateless）** ：服务端不保存客户端状态，客户端保存状态信息每次请求携带状态信息。

**可缓存性（Cacheability）**  ：服务端需回复是否可以缓存以让客户端甄别是否缓存提高效率。

**统一接口（Uniform Interface）** ：通过一定原则设计接口降低耦合，简化系统架构，这是RESTful设计的基本出发点。当然这个内容除了上述特点提到部分具体内容比较多详细了解可以参考这篇[REST论文内容](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)。

**分层系统（Layered System）** ：客户端无法直接知道连接的到终端还是中间设备，分层允许你灵活的部署服务端项目。

**按需代码（Code-On-Demand，可选）** ：按需代码允许我们灵活的发送一些看似特殊的代码给客户端例如JavaScript代码。

REST架构的一些风格和限制条件就先介绍到这里，后面就对RESTful风格API具体介绍。


### RESTful API设计规范

#### URL设计规范

```
URI = scheme "://" host  ":"  port "/" path [ "?" query ][ " fragment ]
```

通常一个RESTful API的path组成如下：
```
/{version}/{resources}/{resource_id}
```
version：API版本号，有些版本号放置在头信息中也可以，通过控制版本号有利于应用迭代。  
resources：资源，RESTful API推荐用小写英文单词的复数形式。  
resource_id：资源的id，访问或操作该资源。

当然，有时候可能资源级别较大，其下还可细分很多子资源也可以灵活设计URL的path，例如：
```
/{version}/{resources}/{resource_id}/{subresources}/{subresource_id}
```
此外，有时可能增删改查无法满足业务要求，可以在URL末尾加上action，例如
```
/{version}/{resources}/{resource_id}/action
```

从大体样式了解URL路径组成之后，对于RESTful API的URL具体设计的规范如下：

1. 不用大写字母，所有单词使用英文且小写。
2. 连字符用中杠`"-"`而不用下杠`"_"`
3. 正确使用 `"/"`表示层级关系,URL的层级不要过深，并且越靠前的层级应该相对越稳定
4. 结尾不要包含正斜杠分隔符`"/"`
5. **URL中不出现动词，用请求方式表示动作**
6. 资源表示用复数不要用单数
7. 不要使用文件扩展名

#### HTTP动词

```
GET /collection：从服务器查询资源的列表（数组）
GET /collection/resource：从服务器查询单个资源
POST /collection：在服务器创建新的资源
PUT /collection/resource：更新服务器资源
DELETE /collection/resource：从服务器删除资源
```

上述四个HTTP请求方法的安全性和幂等性如下：

|HTTP Method|安全性|幂等性|解释|
|---|---|---|---|
|GET|安全|幂等|读操作安全，查询一次多次结果一致|
|POST|非安全|非幂等|写操作非安全，每多插入一次都会出现新结果|
|PUT|非安全|幂等|写操作非安全，一次和多次更新结果一致|
|DELETE|非安全|幂等|写操作非安全，一次和多次删除结果一致|

#### 状态码和返回数据

我们首先要正确使用各类状态码来表示该请求的处理执行结果。状态码主要分为五大类：

> 1xx：相关信息  
> 2xx：操作成功  
> 3xx：重定向  
> 4xx：客户端错误  
> 5xx：服务器错误

每一大类有若干小类，状态码的种类比较多，而主要常用状态码罗列在下面：

200 `OK - [GET]`：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。  
201 `CREATED - [POST/PUT/PATCH]`：用户新建或修改数据成功。  
202 `Accepted - [*]`：表示一个请求已经进入后台排队（异步任务）  
204 `NO CONTENT - [DELETE]`：用户删除数据成功。  
400 `INVALID REQUEST - [POST/PUT/PATCH]`：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。  
401 `Unauthorized - [*]`：表示用户没有权限（令牌、用户名、密码错误）。  
403 `Forbidden - [*]` 表示用户得到授权（与401错误相对），但是访问是被禁止的。  
404 `NOT FOUND - [*]`：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。  
406 `Not Acceptable - [GET]`：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。  
410 `Gone -[GET]`：用户请求的资源被永久删除，且不会再得到的。  
422 `Unprocesable entity - [POST/PUT/PATCH]` 当创建一个对象时，发生一个验证错误。  
500 `INTERNAL SERVER ERROR - [*]`：服务器发生错误，用户将无法判断发出的请求是否成功。

针对不同操作，服务器向用户返回数据，而各个团队或公司封装的返回实体类也不同，但都返回JSON格式数据给客户端

