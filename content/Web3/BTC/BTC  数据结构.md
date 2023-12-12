
### Hash 指针

- 普通的指针是存储结构体在内存中的地址；
- hash 指针 ： 保存地址  + 地址的hash 值

区块链中和普通的链表的区别：
- 使用hash 指针代替了普通的指针 , 即计算了前面整个区块的内容全部取hash 



创世纪块 genesis block

使用hash 指针链接的链表可以实现 tamper-evident log ， 即任何一个块内容修改了，所有的哈希指针都会全部进行修改 ， 所以只需要最后一个位置的hash 值 ， 就可以保证整个链的内容没有修改。
所以比特币只需要保存最近时间的区块 ， 如果需要其他的区块，需要问其他节点拿？
那么如何保证拿到的是正确 的呢？ 可以计算hash值，和本地的对比就可以完成了

### Merkle Tree

相比于 binary tree , 一个区别是使用 hash 指针代替了普通的指针
![[attachments/edd3a4685c352bb124705cbc77e27c28_MD5.jpeg]]


最下面的一层是数据块 data block ，上面的内部节点都是hash 指针 

这个数据结构只要记录根hash值，就可以检测出对树的任何位置的修改 ， 效率比链表的形式高。

比特币中 ， 每一个链都是一个 merkle tree ，每一个data block 都是一个交易 (tx) ， 每一个区块分为块头(block header)和块身 (block body)

block header 里面存储 根hash 值 ; 

比特币的节点分位全节点 （保存节点的所有信息）、轻节点 （只保存一个 block header） ， 那么如何向一个轻节点证明交易已经写入到区块链里面了呢? 
merkle tree 可以提供 merkle proof  ： 从交易的位置一路向上一直到根节点的路径
![[attachments/5d357f799f450cd7b69522d8e5af4ad5_MD5.jpeg]]

轻节点向全节点发出请求，然后tree开始计算上图红色的内容的hash值返回；
然后轻节点计算交易的tx的绿色的hash值，将二者的信息组合到一起 ， 就可以一路交叉向上到根节点的内容；
轻节点将计算的根hash值和 block header 里面的根hash值对比就可以知道这个tx是不是在区块链里面了。


==> 

上图红色的hash值是无法保证的，那么我直接修改红色的hash值， 使得计算第一个绿色和红色的hash 不变？

不行，这就属于人为制造hash碰撞了 



proof of merbership: 证明节点属于merkle tree ， 对于轻节点，计算 merkle proof 的复杂度是对数级别的


proof of non-merbership : 证明节点不属于这个 merkle tree - 不存在什么高效率的办法，只能线性计算每一个节点 ，  因为无法保证节点出现在任何的位置。
如果叶子节点的排序按照交易的hash值排列， 那么就可以先tx计算的hash值 ，然后计算其兄弟节点一路向上计算 ， 最后对比根节点的hash值 ， 这样就可以正确其确实是的兄弟。
这种子节点排序好的 为 sorted merkle tree
比特币不需要做这样的不存在证明。



<u>只要是一个无环结构 ， 都可以hash指针</u>
但是如果是有环的，就变为循环依赖计算hash值了，就会无法构建。




#web3 