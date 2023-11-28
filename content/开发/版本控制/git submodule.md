
子模块允许你将一个 Git 仓库作为另一个 Git 仓库的子目录。 它能让你将另一个仓库克隆到自己的项目中，同时还保持提交的独立

场景: 当项目依赖并跟踪一个开源的第三方库时，将第三方库设置为submodule

## 初始化模块 - git submodule add

```bash
cd third && git submodule add <https://github.com/hibernate/hibernate-orm>
```

子模块现在已经关联到主仓库，并且clone了子模块；

如果后续我们clone主项目，子模块的代码默认会是空的，我们可以

```bash
git clone  --recurse-submodules git@github.com:HelloAnner/DFJ.git 
```

这样会递归地将项目中所有子模块的代码拉取

另外一种可行的方式是，在当前主项目中执行：

```bash
git submodule init
git submodule update
```

会根据主项目的配置信息，拉取更新子模块中的代码

## 更新子模块

对于子模块而言，并不需要知道引用自己的主项目的存在。对于自身来讲，子模块就是一个完整的 Git 仓库，按照正常的 Git 代码管理规范操作即可

### 子模块有未跟踪的内容变动

在开发环境中，直接修改子模块文件夹中的代码

`git status`能够看到关于子模块尚未暂存以备提交的变更,但是于主项目使用 `git add/commit`对其也不会产生影响

在此情景下，通常需要进入子模块文件夹，按照子模块内部的版本控制体系提交代码 （git add），然后下述情况

### 子模块有版本变化

当子模块版本变化时，在主项目中使用 `git status`查看仓库状态时，会显示子模块有新的提交

在这种情况下，可以使用 `git add/commit`将其添加到主项目的代码提交中，实际的改动就是那个子模块 `文件`所表示的版本信息

通常当子项目更新后，主项目修改其所依赖的版本时，会产生如下这种情景的 commit 提交信息:

```bash
git diff HEAD HEAD^
diff --git a/project-sub-1 b/project-sub-1
index ace9770..7097c48 160000
--- a/project-sub-1
+++ b/project-sub-1
@@ -1 +1 @@
-Subproject commit ace977071f94f4f88935f9bb9a33ac0f8b4ba935
+Subproject commit 7097c4887798b71cee360e99815f7dbd1aa17eb4
```

### 子模块远程有更新

需要让主项目主动进入子模块拉取新版代码，进行升级操作

```bash
git pull origin master
```

当主项目的子项目特别多时，可能会不太方便，此时可以使用 `git submodule`的一个命令 `foreach`执行：

```bash
git submodule foreach 'git pull origin master'
```

如果在主项目 git pull ， 会递归地抓取子模块的更改， 但是不会更新子模块，需要:

```bash
git pull  --recurse-submodules

# 等价于

git submodule update --init --recursive
```

## 删除子模块

根据官方文档的说明，应该使用 `git submodule deinit`命令卸载一个子模块。这个命令如果添加上参数 `--force`，则子模块工作区内即使有本地的修改，也会被移除