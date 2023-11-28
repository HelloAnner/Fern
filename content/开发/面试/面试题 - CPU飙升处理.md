
top -H -p [PID]  找到CPU占用最高的

-H : 显示每一个进程的线程 
-p : 指定自己需要的进程的PID

---

将PID转为 16 进制
```
printf '0x%x\n' 18745
```


---

jstack  显示了线程状态
```
jstack 12345 | grep 0x4939 -A 20
```

grep -A : 显示匹配后的20行文本

