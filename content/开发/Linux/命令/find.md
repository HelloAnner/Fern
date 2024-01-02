### 查找到后的操作

exec 参数后面跟的是command命令，它的终止是以;为结束标志的 , 虑到各个系统中分号会有不同的意义，所以前面加反斜杠

{} 花括号代表前面find查找出来的文件名

```shell
find . -type f -exec ls -l {} \;

find . -type f -exec rm -rf {} \;
```

- [ ] 复习 find (@2024-01-23)