-X GET

-H 添加请求head

-I 输出响应头

-L 自动跳转


-i 参数打印出服务器回应的 HTTP 标头

-k 参数指定跳过 SSL 检测


-o参数将服务器的回应保存成文件，等同于wget命令
-O参数将服务器回应保存成文件，并将 URL 的最后部分当作文件名
```
curl -o example.html https://www.example.com
```

-s参数将不输出错误和进度信息


-v参数输出通信的整个过程，用于调试

- [ ] 复习 curl (@2024-01-23)