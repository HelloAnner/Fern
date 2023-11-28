```embed
title: "MySQL创建联合索引报key长度超3072 bytes的[42000][1071]错误-CSDN博客"
image: "https://img-blog.csdnimg.cn/c1040320a9ea43ce8d4c1562de44a9d1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQXBwIFN0b3Jl5YWs5LyX5Y-35bCP56iL5bqPOuWIhuS6q-W9lQ==,size_20,color_FFFFFF,t_70,g_se,x_16"
description: "文章浏览阅读7.8k次。问题时这样的，我在建表时加了联合索引结果报key长度超过3072个字节了，如下图。[42000][1071] Specified key was too long; max key length is 3072 bytes先说解决方案：1.调整索引字段，包括修改字段长度、更换字段；2.使用前缀索引在MySQL 5.6以及以前的版本，InnoDB引擎默认索引长度不能超过767 bytes，在MySQL 5.5以后支持4个字节的utf8mb4（mb4全称maximun of4bytes p.._3072 bytes"
url: "https://blog.csdn.net/xubingtao/article/details/123824946"
```

utf-8编码一个字符会占用3个字节，而utf8mb4编码一个字符会占用4个字节

在开启INNODB_LARGE_PREFIX参数时，索引键前缀限制变为3072字节