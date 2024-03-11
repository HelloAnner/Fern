Systemd 并不是一个命令，而是一组命令，涉及到系统管理的方方面面。



## systemctl

`systemctl`是 Systemd 的主命令，用于管理系统。

```shell
# 重启系统
$ sudo systemctl reboot

# 关闭系统，切断电源
$ sudo systemctl poweroff

# CPU停止工作
$ sudo systemctl halt

# 暂停系统
$ sudo systemctl suspend

# 让系统进入冬眠状态
$ sudo systemctl hibernate

# 让系统进入交互式休眠状态
$ sudo systemctl hybrid-sleep

# 启动进入救援状态（单用户状态）
$ sudo systemctl rescue
```

## Unit

Systemd 可以管理所有系统资源。不同的资源统称为 Unit（单位）。

- Service unit：系统服务
- Target unit：多个 Unit 构成的一个组
- Device Unit：硬件设备
- Mount Unit：文件系统的挂载点
- Automount Unit：自动挂载点
- Path Unit：文件或路径
- Scope Unit：不是由 Systemd 启动的外部进程
- Slice Unit：进程组
- Snapshot Unit：Systemd 快照，可以切回某个快照
- Socket Unit：进程间通信的 socket
- Swap Unit：swap 文件
- Timer Unit：定时器


### Unit 管理

```
# 立即启动一个服务
$ sudo systemctl start apache.service

# 立即停止一个服务
$ sudo systemctl stop apache.service

# 重启一个服务
$ sudo systemctl restart apache.service

# 杀死一个服务的所有子进程
$ sudo systemctl kill apache.service

# 重新加载一个服务的配置文件
$ sudo systemctl reload apache.service

# 重载所有修改过的配置文件
$ sudo systemctl daemon-reload

# 显示某个 Unit 的所有底层参数
$ systemctl show httpd.service

# 显示某个 Unit 的指定属性的值
$ systemctl show -p CPUShares httpd.service

# 设置某个 Unit 的指定属性
$ sudo systemctl set-property httpd.service CPUShares=500
```

### 依赖关系

Unit 之间存在依赖关系：A 依赖于 B，就意味着 Systemd 在启动 A 的时候，同时会去启动 B。

```
systemctl list-dependencies nginx.service
```

### Unit 的配置文件

每一个 Unit 都有一个配置文件，告诉 Systemd 怎么启动这个 Unit 。

Systemd 默认从目录`/etc/systemd/system/`读取配置文件。

但是，里面存放的大部分文件都是符号链接，指向目录`/usr/lib/systemd/system/`，真正的配置文件存放在那个目录。

```shell
$ sudo systemctl enable clamd@scan.service
# 等同于
$ sudo ln -s '/usr/lib/systemd/system/clamd@scan.service' '/etc/systemd/system/multi-user.target.wants/clamd@scan.service'
```

如果配置文件里面设置了开机启动，`systemctl enable`命令相当于激活开机启动。

与之对应的，`systemctl disable`命令用于在两个目录之间，撤销符号链接关系，相当于撤销开机启动。

```shell
sudo systemctl disable clamd@scan.service
```

配置文件的后缀名，就是该 Unit 的种类，比如`sshd.socket`。如果省略，Systemd 默认后缀名为`.service`，所以`sshd`会被理解成`sshd.service`。


`systemctl list-unit-files`命令用于列出所有配置文件。

```
# 列出所有配置文件
$ systemctl list-unit-files

# 列出指定类型的配置文件
$ systemctl list-unit-files --type=service
```
注意，从配置文件的状态无法看出，该 Unit 是否正在运行。这必须执行前面提到的`systemctl status`命令。


`systemctl cat`命令可以查看配置文件的内容。
```shell
$ systemctl cat atd.service

[Unit]
Description=ATD daemon

[Service]
Type=forking
ExecStart=/usr/bin/atd

[Install]
WantedBy=multi-user.target
```

## 例子 - 创建一个服务


在 `/lib/systemd/system`目录下创建一个脚本文件tomcat.service
```
[Unit]
Description=Tomcat
After=network.target

[Service]
Type=forking
PIDFile=/usr/local/tomcat/pid
ExecStart=/usr/local/tomcat/bin/catalina.sh start
ExecReload=/usr/local/tomcat/bin/catalina.sh restart
ExecStop=/usr/local/tomcat/bin/catalina.sh stop

[Install]
WantedBy=multi-user.target
```

[Unit] 表示这是基础信息
Description 是描述
After 是在那个服务后面启动，一般是网络服务启动后启动

[Service] 表示这里是服务信息
Type 是服务类型
PIDFile 是服务的pid文件路径， 开启后，必须在tomcat的bin/catalina.sh中加入CATALINA_PID参数
ExecStart 是启动服务的命令
ExecReload 是重启服务的命令
ExecStop 是停止服务的指令

[Install] 表示这是是安装相关信息
WantedBy 是以哪种方式启动：multi-user.target表明当系统以多用户方式（默认的运行级别）启动时，这个服务需要被自动运行。



创建软链接:
```
sudo systemctl enable tomcat.service
```


如果需要找到真实的进程，需要转一个用户空间，才可以使用类似 arthas 来 attach
```shell
nsenter -m -t <PID>
```





## 参考

[Systemd 入门教程：命令篇 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html)

