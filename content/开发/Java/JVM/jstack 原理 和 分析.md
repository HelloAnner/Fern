

## 分析实战


### 分析 CPU 占有率高的线程信息

top 找出占用 CPU 高的进程 PID [[../../Linux/命令/top|top]]

top -p PID -H 命令查出进程中占用 CPU 最高的线程

根据线程 ID（需要十进制转成十六进制），从线程栈中找出步骤2查出的线程 printf 0x%x 43845

jstack -l PID 命令打印出线程栈


- [ ]  复习 jstack 原理 和 分析 (@2024-01-31)