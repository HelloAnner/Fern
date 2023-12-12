
## cryptographic hash function

### collision resistance

存在hash碰撞，但是无法人为制造 ： 两个不同的输入，存在相同的输出 ； 因为输入空间远大于输出空间 ， 所以必存在hash碰撞 ，但是现在没有高效的方法可以人为的制造hash 碰撞。 目前仅存在蛮力操作。

但是上述性质，数学上无法证明。

### hiding

hash 函数的计算过程是单向不可逆的。
hiding 性质要求输入空间很大且很均匀。

Hiding +  collision resistance 可以实现 digital commitment ： 公布出去的hash值 ，因为存在 hiding ，所以无法知道原值 ； 实现一个 sealed envelope 

### puzzle friendly

hash 值的计算是不可以被预测的 ， 只能一一的尝试。没有任何的捷径。
所以这个过程才可以被作为一个工作量证明。

虽然找到nonce很难，但是验证nonce很容易 ，即 difficult to solve , but easy to verify

H(black header) <= target 

---

目前使用的是 sha256 - secure hash algorithm 


## 签名

本地创建一个 (public key , private key) 对 即是一个账户

对称加密实现的前提就是存在一个理论安全的渠道传递密钥；
非对称加密使用双方的公钥加密，都不需要知道对方的私钥。 [[../../开发/安全/加密|加密]]

当开始发起一个交易，使用我的私钥去签名，其他人使用我的公钥去验证
[[../../开发/安全/签名/签名基础|签名基础]]

目前不考虑公私钥对的相同的情况，前提是有好的随机源。
同时签名的时候存在好的随机源。

比特币系统一半是对message取hash ，然后对hash内容进行签名。

#web3