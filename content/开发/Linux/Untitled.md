

```
# 查看CPU 核数
nproc
lscpu | grep "CPU(s):"

# 查看 CPU 信息
cat /etc/os-release
```


```
# 内存占用情况
free -h

```


```
# 磁盘挂载的基本情况
bash-4.4# df -h

Filesystem        Size  Used Avail Use% Mounted on
overlay            59G   14G   43G  24% /
tmpfs              64M     0   64M   0% /dev
shm                64M     0   64M   0% /dev/shm
/dev/vda1          59G   14G   43G  24% /etc/hosts
/host_mark/Users  927G  419G  508G  46% /etc/mysql/conf.d
tmpfs             6.0G     0  6.0G   0% /sys/firmware
```