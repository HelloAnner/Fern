
### 规模复杂度

分而治之，小既是美。

- Unix设计哲学：单一职责、团结合作
    - 不在老的东西越来越复杂、而是创建一个新的东西完成 → 但是这样的会存在多个小的命令 - 如何分离
    - 每一个命令的输出，变为另外一个新的命令的输入 - 如何合并

### 结构复杂度

系统 存在 多个 复杂的模块 ， 某一个模块里面的代码又是错综复杂，这个时候就是结构复杂度。

清晰直观且易于理解的分层（DIP - 依赖倒置）

首要任务就是确定业务逻辑和技术逻辑的边界。

- 不关心技术逻辑，而是只关心业务
    
 ![[attachments/Pasted image 20231027151457.png|300]]
    

复杂度 - 横切 、 纵切

横切 - 按照分层：用户接口层、应用层、领域层、基础设施层

![[attachments/Pasted image 20231027151519.png|120]]

纵切 - 业务上 划分

### 变化复杂度

拥抱变化、只被第一课子弹击中。

分离关注点 - SOC

分而治之 - HTML 、 CSS 、JS

## 通用语言 - 沟通的标准
![[attachments/Pasted image 20231027151547.png|400]]

客户的需求可能是明确清晰的，也可能是一个想法、需要一起交流才可以确定；

基本上，行业内客户基本都是清晰的、创业的客户基本都是一个想法。
![[attachments/Pasted image 20231027152134.png|400]]
通用语言：消除领域专家和开发者之间的沟通失调

- 准确、高效、通用
- 不包含任务IT 术语
- 只包含术语用例

领域术语：需要明白一个行业的内部黑话

- 强调动词的准确性
![[attachments/Pasted image 20231027152150.png|400]]
避免：

同名的业务词汇和实际业务关系不清楚

在沟通过程中，需要注意 - 基础咨询问题框架：

- why：为了什么目的
    
- what：要做什么事情
    
- who：谁会与此相关
    
- when：什么时间做
    
- where：在什么地方做
    
- how：怎么做
    
- **how much：需要什么资源**
    
- **exception：例外情况**
    
- pre - condition: 必须的前提
    
- post - condition: 应该的结果 - 与 why 对应
    
- “高端客户优惠” 例子
    
    ![[attachments/Pasted image 20231027152210.png|400]]
    ![[attachments/Pasted image 20231027152223.png|400]]

最好是在团队中形成一个相对固定的场景分析模式：

- 用例图 use case
    
    文字描述
    ![[attachments/Pasted image 20231027152523.png|400]]
    
    图描述 - 这里看一下 include 说明子部分 ， extend 说明条件扩展部分
    
    ![[attachments/Pasted image 20231027152538.png|400]]
- 用户故事 - use store
    
    文字描述 + 验收标准
    ![[attachments/Pasted image 20231027152557.png|400]]
    
    用户故事卡 - 短信验证码的例子 - 术语 + 用例
    
    ![[attachments/Pasted image 20231027152611.png|400]]
    
- 测试驱动开发 - test drive development
    
    先说需求，然后需求分析优先、任务分解优先
    
    先写接口，再完成实现
    
    以一个失败的测试为起点，逐步修复，完成这个单元测试
    
    ![[attachments/Pasted image 20231027152623.png|400]]
    

## 领域

领域 - 问题的范围
![[attachments/Pasted image 20231027153019.png|200]]

子域：

- 核心子域： 核心竞争力所在的子域
- 支撑域：支撑核心域的子域
- 通用域：服务整个领域的子域

### 界限上下文：问题的解决方案

- 界限 Bounded
- 上下文 Context

动态的业务流程会在边界上进行上下文切换
![[attachments/Pasted image 20231027153044.png|400]]

不同的上下文中，相同的名词的含义可能完全不同。

所以使用界限上下文的概念划分，可以完成消除歧义。

上下文可以区分业务，即可以使用多个表完成业务，而不是一个超大的宽表

![[attachments/Pasted image 20231027153059.png|400]]

界限上下文就是微服务类型的一个指导思想。

可以在划分领域的时候，可以先写出 **行为**

界限上下文意味着安全、安全意味着可控

界限上下文不仅可以完成技术的边界，也可以完成团队的划分。

“完美的设计不是包罗万象无所不有，而是完整自治、不可精简”

避免出现 “上帝类” ，知道的太多、做的太多、依赖的太多。

### 上下文映射 - Context Mapping

解决如何联系多个上下文。

上游 U：依赖发起方

下游 D： 依赖上游
![[attachments/Pasted image 20231027153121.png|400]]
团队协作模式：

- 合作关系
    
    演化初期的关系，同生共死 ， 不是一个很好的协作关系
    
    ![[attachments/Pasted image 20231027153143.png|400]]
    
    ![[attachments/Pasted image 20231027153159.png|400]]
    
- 共享内核
    
    ![[attachments/Pasted image 20231027154858.png|400]]
    
- 消费供应 - 比较不错 C-S
    
    ![[attachments/Pasted image 20231027154922.png|400]]
    
- 遵奉者 Conformist
    
    消费供应的反模式
    ![[attachments/Pasted image 20231027154939.png|400]]
    
- 另谋他路
    ![[attachments/Pasted image 20231027154953.png|400]]
    

代码上的上下文关系模式：

- 防腐层 -ACL
    
    OCP 的思想设计
    
    ![[attachments/Pasted image 20231027155013.png|400]]
    
- 开放主机服务 - OHS
    
    PL - 发布语言，一般结合使用
    
    ![[attachments/Pasted image 20231027155024.png|400]]
    
    上游避免下游的多样性的设计
    
- 发布-订阅事件
    
    ![[attachments/Pasted image 20231027155032.png|400]]
    ![[attachments/Pasted image 20231027155041.png|400]]
    
    吞吐量高、解耦
    
    但是很难保证一致性。
    
    如果失败了，基于消息的失败之后，可以基于 SAGA 完成，即基于补偿消息
    
![[attachments/Pasted image 20231027155059.png|400]]
## 架构

宏观建模和微观建模衔接的部分
![[attachments/Pasted image 20231027155115.png|400]]
右侧的 DDD 分层设计中 ， Application Layer 的作用是什么？

编排业务逻辑：

- 以前的 业务逻辑层 - 报纸中标题和正文放在了一起 ， 流程和细节耦合在了一起
- 现在加了应用层 ， 流程和细节分离； 将报纸修改为杂志；符合单一抽象层次原则。

### DDD 分层

现在看看贫血模型：
![[attachments/Pasted image 20231027155126.png|400]]
Service 里面承担了所有的逻辑；

要求调用方是一个“全能型”选手，必须知道所有的业务逻辑；

技术和业务相分离：

![[attachments/Pasted image 20231027155140.png|400]]

同时支持输入的多样性
![[attachments/Pasted image 20231027155154.png|400]]

综上，一个DDD分层会去这样设计：
![[attachments/Pasted image 20231027155207.png|450]]
### 六边形架构 - 整洁架构
![[Pasted image 20231027155225.png|400]]
按照业务逻辑转一圈，实现一个三色同心圆 ，即一个多层的架构

业务的边界全部是接口，由接口定义好，实现技术和业务分离 ， 整体符合 DIP 原则
![[attachments/Pasted image 20231027155238.png|400]]
即依赖注入，技术使用接口挂接进去了
![[attachments/Pasted image 20231027155247.png|400]]
### 微服务架构

- 微服务使用界限上下文划分
- 每一个服务都使用独立的数据存储

### CQRS 架构

写模型 和 读模型分离

写的时候可以写窄表，读的时候可以使用宽表
![[attachments/Pasted image 20231027155256.png|400]]
读和写模型可以使用消息等等完成通信；

可能不同的界限上下文可以使用的架构完成。

接下来可以看看一些基础的设计的原则： [SOLID](https://www.notion.so/SOLID-8624c41e134a4a8a9c38a06743059d1e?pvs=21)