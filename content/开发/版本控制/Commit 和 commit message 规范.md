```
<JIRA任务编号> <type> : <subject>
```

### type

用于说明git commit的类别，只允许使用下面的标识。

feat：新功能（feature）。

fix：修复bug，可以是QA发现的BUG，也可以是研发自己发现的BUG。

docs：文档（documentation）。

style：格式（不影响代码运行的变动）。

refactor：重构（即不是新增功能，也不是修改bug的代码变动）。

perf：优化相关，比如提升性能、体验。

test：增加测试。

chore：构建过程或辅助工具的变动。

revert：回滚到上一个版本。

merge：代码合并。

sync：同步主线或分支的Bug。

point：埋点
subject
subject是commit目的的简短描述，不超过50个字符。

DEC-123 fix : 修复权限问题

DEC-345 chore : 完善打包环境

pr message 规范
<JIRA任务编号> [type] <subject>

JIRA任务编号: 包含多个任务用&分隔
subject: 简单描述此次PR内容,若只有单个commit,可以用创建pr是自动生成的title
安全性任务模糊规范
对于设计安全性修改的任务,统一对任务<subject>模糊处理,统一文案为"增强系统安全性"

如何修改commit message
1.代码还未push，想修改最近一次提交

$ git commit --amend

修改完毕之后就会形成一次新的提交

如果用sourcetree的话右下角的提交选项选修改上一次提交就好了

2.已经push过了

https://www.jianshu.com/p/ec45ce13289f 可以参考这个

不过建议就保持现状，不要修改了。下次注意就好了