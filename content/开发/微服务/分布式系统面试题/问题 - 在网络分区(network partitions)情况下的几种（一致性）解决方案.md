
网络分区指的是分布式系统中的节点或网络部分无法相互通信，导致系统中的不同部分形成孤立的子系统
在这种情况下，不同子系统之间的消息传递可能失败或延迟，可能导致数据不一致性的问题。

1. 强一致性（Strong Consistency）：强一致性是指系统保证在任何时间点，无论网络分区的存在与否，所有节点都能够看到相同的数据副本。在网络分区的情况下，强一致性方案将会阻塞或拒绝写入操作，直到网络分区解决或者达到一定的超时时间。
2. 弱一致性（Weak Consistency）：弱一致性是指系统在网络分区的情况下，允许不同节点之间的数据副本存在一定的延迟和不一致性。系统会尽力在分区解决后将数据同步，但不能保证实时一致性。常见的弱一致性模型包括最终一致性（Eventual Consistency）和会话一致性（Session Consistency）。 ]]
3. 最终一致性（Eventual Consistency）：最终一致性是指系统保证在没有新的更新操作时，经过一段时间后，所有节点最终会达到一致的状态。系统会尽力在网络分区解决后将数据同步，但在同步过程中可能存在延迟和数据冲突。
4. 会话一致性（Session Consistency）：会话一致性是一种介于强一致性和最终一致性之间的模型。它通过限制操作的范围或者使用特定的会话标识符，将一组操作绑定到同一个会话中，保证在同一个会话中的操作具有一定的顺序性，但不保证全局的一致性

[[分布式系统一致性和共识]]