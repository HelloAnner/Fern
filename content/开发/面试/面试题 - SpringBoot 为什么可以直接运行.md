即 Springboot jar 的执行原理

spring-boot-maven-plugin 插件打包后的jar ，完成直接运行的逻辑
帮助我们生成 Fat jar 包 （jar 包里面包含了项目依赖的所有的jar）
![[Pasted image 20231107190450.png|400]]


java -jar 里面 JVM 规范里面，会从  META-INF/MANIFEST.MF 文件 寻找 Main-Class 的内容
![[Pasted image 20231107190635.png|400]]

对于 springboot jar ， 那么启动的入口就是 spring-boot-loader.jar 里面的  JarLaunch.class

![[Pasted image 20231107190926.png|600]]

创建了一个 类加载器，开始去读取jar包当中的jar

那么如何走到我们当前应用的main方法呢？

那个插件会帮助我们维护一个 Start-Class （见上面的截图）

![[Pasted image 20231107191102.png|600]]

开启一个子进程启动main方法