

https://kms.fineres.com/pages/viewpage.action?pageId=959021781#

### 禁止直接使用不可信数据来拼接SQL语句



### 避免ReDos攻击
**ReDoS 攻击（正则表达式拒绝服务攻击 (Regular Expression Denial of Service)）攻击者可构造特殊的字符串，导致正则表达式运行会消耗大量的内存和 cpu 导致服务器资源被耗尽**

**可通过外部网站[ReDoS checker | Devina.io](https://devina.io/redos-checker)检测自己写的正则表达式是否存在redos攻击漏洞**

包含具有自我重复的重复性分组的正则
```
^(\d+)+$
^(\d*)*$
^(\d+)*$
^(\d+|\s+)*$
```

包含替换的重复性分组
```
^(\d|\d|\d)+$
^(\d|\d?)+$
```

### 禁止将敏感信息硬编码在程序中

**说明：如果将敏感信息（包括口令和加密密钥）硬编码在程序中，可能会将敏感信息暴露给攻击者**。任何能够访问到class文件的人都可以反编译class文件并发现这些敏感信息。因此，不能将信息硬编码在程序中。同时，硬编码敏感信息会增加代码管理和维护的难度。例如，在一个已经部署的程序中修改一个硬编码的口令需要发布一个补丁才能实现。

```java
//恶意用户可以使用javap -c IPaddress命令来反编译class来发现其中硬编码的服务器IP地址。反编译器的输出信息透露了服务器的明文IP地址：172.16.254.1
public class IPaddress{
  private String ipAddress = "172.16.254.1";
  public static void main(String[] args) {
    //...
  }
}
```

