
跨域只有在前端才会发生，因为浏览器的同源策略导致

#### JSONP

比较老的方案

前端： dataType 为  jsonp ， 使用 callback 方式
后端：后端执行 callback ，将值返回给前端
![[attachments/Pasted image 20231102180607.png|500]]

只支持 get 请求

前后端都需要写代码 

#### CORS

只需要写后端的代码

前端直接 ajax 发起跨域请求就可以完成

##### @crossOrigin 允许单个接口

##### 实现一个config bean , 添加 addCrossMappings 

##### corsFilter 可以限制所有的接口支持

需要浏览器支持 ， 即  需要浏览器帮助添加信息 ， 第一次发起 option 请求  ， 第二次才传真实的跨域请求

#### Nginx 反向代理 - 最推荐

只需要在 Nginx 配置一下 

