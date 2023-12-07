### 服务器端

socket 服务端需要开一个端口监听（listen）socket 请求就行，然后一个端口，理论上来说，最大能支持2的32次方（ip 数）×2 的 16 次方（port 数）个连接，但是 linux 对 [打开文件数有限制](https://garden.3dot141.top/1-Inputs/Article/%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6)（65536个，每个 socket 连接占用一个文件）