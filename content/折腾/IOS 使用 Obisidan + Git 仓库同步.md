
apple store 下载 ish 可以连接到自己的ipad os 的shell 上。

> iSH是一个模拟器，用来在ARM架构的iOS设备上模拟x86架构，让iOS设备在本地运行Linux Shell环境
> 
> iSH使用的Linux系统镜像是Alpine Linux



将IOS系统中的本地文件挂载到ISH模拟的系统中
```shell
mount -t ios . obsidian

# 如果需要取消挂载
umount obsidian
```


接下来就是开始配置 Git 仓库的密钥了， 不继续赘述

最后开始拉仓库，定时拉仓库和推送的逻辑交给 obsidian git 插件完成

git方案的多设备协同容易存在文件冲突的问题，所以建议将 obsidian git 的 push 策略修改停止编辑3分钟提交；
其他设备继续使用前，设置为1分钟 pull 一次 ， 防止冲突频率过高。




### 参考

[ios上使用iSH的git同步obsidian - 经验分享 - Obsidian 中文论坛](https://forum-zh.obsidian.md/t/topic/10083)

