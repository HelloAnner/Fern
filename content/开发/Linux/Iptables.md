### 规则
规则分别指定了源地址、目的地址、传输协议（如TCP、UDP、ICMP）和服务类型（如HTTP、FTP和SMTP）等。

当数据包与规 则匹配时，iptables就根据规则所定义的方法来处理这些数据包，如放行（accept）、拒绝（reject）和丢弃（drop）等。

配置防火墙的 主要工作就是添加、修改和删除这些规则。

### 链
链（chains）是数据包传播的路径，每一条链其实就是众多规则中的一个检查清单，每一条链中可以有一 条或数条规则。当一个数据包到达一个链时，iptables就会从链中第一条规则开始检查，看该数据包是否满足规则所定义的条件。如果满足，系统就会根据 该条规则所定义的方法处理该数据包；否则iptables将继续检查下一条规则，如果该数据包不符合链中任一条规则，iptables就会根据该链预先定 义的默认策略来处理数据包。


- INPUT——进来的数据包应用此规则链中的策略
- OUTPUT——外出的数据包应用此规则链中的策略
- FORWARD——转发数据包时应用此规则链中的策略
- PREROUTING——对数据包作路由选择前应用此链中的规则
- POSTROUTING——对数据包作路由选择后应用此链中的规则

### 表

表（tables）提供特定的功能，iptables内置了4个表，即filter表、nat表、mangle表和raw表，分别用于实现包过滤，网络地址转换、包重构(修改)和数据跟踪处理。


![[e1fc20cb2b3a72e299cf78a39126b944_MD5.jpeg|400]]

- filter表——三个链：INPUT、FORWARD、OUTPUT  作用：过滤数据包  内核模块：iptables_filter.
- Nat表——三个链：PREROUTING、POSTROUTING、OUTPUT   作用：用于网络地址转换（IP、端口） 内核模块：iptable_nat
- Mangle表——五个链：PREROUTING、POSTROUTING、INPUT、OUTPUT、FORWARD   作用：修改数据包的服务类型、TTL、并且可以配置路由实现QOS内核模块：iptable_mangle(别看这个表这么麻烦，咱们设置策略时几乎都不会用到它)
- Raw表——两个链：OUTPUT、PREROUTING   作用：决定数据包是否被状态跟踪机制处理  内核模块：iptable_raw

Raw——mangle——nat——filter

### 数据包流向
![[d45d7d5271dabd04e53b991472a3d18f_MD5.jpeg|400]]

① 当一个数据包进入网卡时，它首先进入PREROUTING链，内核根据数据包目的IP判断是否需要转送出去。

② 如果数据包就是进入本机的，它就会沿着图向下移动，到达INPUT链。数据包到了INPUT链后，任何进程都会收到它。本机上运行的程序可以发送数据包，这些数据包会经过OUTPUT链，然后到达POSTROUTING链输出。

③ 如果数据包是要转发出去的，且内核允许转发，数据包就会如图所示向右移动，经过FORWARD链，然后到达POSTROUTING链输出。


### 例子


-A 在指定链的末尾添加（append）一条新的规则
-D  删除（delete）指定链中的某一条规则，可以按规则序号和内容删除
-I  在指定链中插入（insert）一条新的规则，默认在第一行添加
-R  修改、替换（replace）指定链中的某一条规则，可以按规则序号和内容替换
-L  列出（list）指定链中所有的规则进行查看
-E  重命名用户定义的链，不改变链本身
-F  清空（flush）
-N  新建（new-chain）一条用户自己定义的规则链
-X  删除指定表中用户自定义的规则链（delete-chain）
-P  设置指定链的默认策略（policy）
-Z 将所有表的所有链的字节和数据包计数器清零
-n  使用数字形式（numeric）显示输出结果
-v  查看规则表详细信息（verbose）的信息


-j :
ACCEPT 允许数据包通过
DROP 直接丢弃数据包，不给任何回应信息
REJECT 拒绝数据包通过，必要时会给数据发送端一个响应的信息。
LOG在/var/log/messages文件中记录日志信息，然后将数据包传递给下一条规则


规则的保存
```shell
iptables-save > /etc/sysconfig/iptables

service iptables save # 规则自动保存在/etc/sysconfig/iptables
```

iptables [-t 表名] 命令选项 ［链名］ ［条件匹配］ ［-j 目标动作或跳转］

```shell
# 拒绝进入防火墙的所有ICMP协议数据包
iptables -I INPUT -p icmp -j REJECT

# 允许防火墙转发除ICMP协议以外的所有数据包
iptables -A FORWARD -p ! icmp -j ACCEPT

# 拒绝转发来自192.168.1.10主机的数据，允许转发来自192.168.0.0/24网段的数据
# 注意要把拒绝的放在前面不然就不起作用了
iptables -A FORWARD -s 192.168.1.11 -j REJECT
iptables -A FORWARD -s 192.168.0.0/24 -j ACCEPT

# 丢弃从外网接口（eth1）进入防火墙本机的源地址为私网地址的数据包
iptables -A INPUT -i eth1 -s 192.168.0.0/16 -j DROP
iptables -A INPUT -i eth1 -s 172.16.0.0/12 -j DROP
iptables -A INPUT -i eth1 -s 10.0.0.0/8 -j DROP

# 封堵网段（192.168.1.0/24），两小时后解封
iptables -I INPUT -s 10.20.30.0/24 -j DROP
iptables -I FORWARD -s 10.20.30.0/24 -j DROP
at now 2 hours at> iptables -D INPUT 1 at> iptables -D FORWARD 1
```