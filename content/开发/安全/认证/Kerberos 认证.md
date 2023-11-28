我，对方，户政系统，是个三方沟通过程，我的身份对方无法判断是不是我本人，他也需要向一个可以被认可的认证中心去验证。验证完成后。

MIT最早提出了Kerberos认证协议来实现这个过程，同时MIT也提供了其实现


- **密钥分发中心（Key Distribution Center，KDC）**，而密钥分发中心一般又分为两部分，分别是：
    - **AS（Authentication Server）**：认证服务器，专门用来认证客户端的身份并发放客户用于访问TGS的TGT（票据授予票据）
    - **TGT（Ticket Granting Ticket）**：票据授予服务器，用来发放整个认证过程以及客户端访问服务端时所需的服务授予票据（Ticket）


![[Pasted image 20231109183615.png|600]]

### 第一次通信

由于客户端是第一次访问KDC，此时KDC也不确定该客户端的身份，所以**第一次通信的目的为KDC认证客户端身份，确认客户端是一个可靠且拥有访问KDC权限的客户端**，过程如下：
![[Pasted image 20231109184641.png|600]]

第一次通信完成后，客户端会有一个TGT

### 第二次通信


![[Pasted image 20231109184945.png|600]]

### 第三次通信


![[Pasted image 20231109185026.png|600]]


整个kerberos认证的过程较为复杂，三次通信中都使用了密钥，且密钥的种类一直在变化，并且为了防止网络拦截密钥，这些密钥都是临时生成的Session Key，即他们只在一次Session会话中起作用，即使密钥被劫持，等到密钥被破解可能这次会话都早已结束。

### JDK 结合 Kerberos

一般来讲就是MIT的kerberos实现，java本身也是基于MIT kerberos，集成到了GSS-API（Generic Security Service Application Program Interface）和JAAS（Java Authentication and Authorization Service）框架

JAAS 使用 Subject 和 Principal 对象来表示用户的身份信息，这个框架基本流程可以在源码学习中看到。  
在 Kerberos 身份验证中，通过 JAAS，你可以将 Kerberos 生成的票据（Ticket）包装成 Subject 对象，使得 Java 应用程序能够方便地获取用户的身份信息。

![[Pasted image 20231109185641.png|500]]

数据库自己也是需要支持的 ， 驱动需要支持 JAAS 认证；
对于 kerberos ，也是有自己的DB 存储了自己支持的数据库

### LoginContext

loginContext 属于 JAAS 框架的一部分，一般在 Javax 里面

- 第一步，login，成功或者失败
- 第二步，commit，收尾login的信息，成功是设置Subject，失败是清除动作（啥都不做也行）
- 第二步（如果失败），如果失败abort，它也属于第二步动作，终止登录

如果login方法登录成功，就会调用commit方法将登陆成功的principal和凭据保存到Subject对象中
```java
public void login() throws LoginException {  
  
    loginSucceeded = false;  
  
    if (subject == null) {  
        subject = new Subject();  
    }  
  
    try {  
        // module invoked in doPrivileged  
        invokePriv(LOGIN_METHOD);  
        invokePriv(COMMIT_METHOD);  
        loginSucceeded = true;  
    } catch (LoginException le) {  
        try {  
            invokePriv(ABORT_METHOD);  
        } catch (LoginException le2) {  
            throw le;  
        }  
        throw le;  
    }  
}
```

### Kerberos 源码

Subject 是任意的主体   ，Principal 是 一个身份。
>例如，一个 Subject 是一个人 Alice，可能有两个 Principal：一个将“Alice Bar”（她的驾驶执照上的名字）绑定到该 Subject，另一个将“999-99-9999”绑定她的学生证上的号码。尽管每个Principal具有不同的名称，但这些Principal都属于同一个主体。

![[Pasted image 20231109190351.png]]

### Hadoop - UserGroupInformation （UGI）

UserGroupInformation是Hadoop提个的Kerberos登录套件，前面分析过Subject的概念和作用，我们知道Subject表示的一个主体，包括了多重的身份。UGI本质上也是这么一个作用，它封装了LoginContext到Subject这套过程，可以直接和Subject相比较。

![[Pasted image 20231109190819.png]]

但是它的不同之处在于，给开发者更明确的“登录感知”，并且提供了一系列类似容易理解的方法，比如它提供了login方法需要传keytab文件路径和principla进去。必须：

- **先执行登录UserGroupInformation.loginUserFromKeytab**
- **在获取登录信息进行业务执行UserGroupInformation.getCurrentUser().doAs**

这里的getCurrentUser也是个容易理解的概念，尽管它实际上还是Subject
![[Pasted image 20231109191025.png]]

### Impala 

**大部分数据库的Kerberos认证都是UGI实现**，其实登录过程我们自己实现也行，不过就是PrivilegedAction或者PrivilegedExceptionAction不太好实现

impala 的连接方式属于是第二种链接方式

- keyTab 二进制文件 存储 priciple
- krb5.conf 存储 KDC信息 realm 信息
- jaas.conf 存储了 keyTab path


**impala依赖jaas文件配置登录，其它一概不要**


### 常见问题

Unable to obtain Principal Name for authentication (出现在impala数据库)
该报错是比较明显的principal不匹配以及不正确的keytab的路径，一般先检查匹配问题，再检查keytab路径是否正常的问题


[问题排查手册](https://kms.fineres.com/pages/viewpage.action?pageId=363630554)
### 参考

```embed
title: "详解kerberos认证原理"
image: "http://seevae.github.io/img/kerberos1.jpg"
description: "前言Kerberos协议是一个专注于验证通信双方身份的网络协议，不同于其他网络安全协议的保证整个通信过程的传输安全，kerberos侧重于通信前双方身份的认定工作，帮助客户端和服务端解决“证明我自己是我自己”的问题，从而使得通信两端能够完全信任对方身份，在一个不安全的网络中完成一次安全的身份认证继而进行安全的通信。由于整个Kerberos认证过程较为复杂繁琐且网上版本较多，特整理此文以便复习与分享"
url: "https://seevae.github.io/2020/09/12/%E8%AF%A6%E8%A7%A3kerberos%E8%AE%A4%E8%AF%81%E6%B5%81%E7%A8%8B/"
```
