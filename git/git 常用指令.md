## git 基础

### git 基础命令

```
git config --local
git config --system
git config --global

git config --list

git mv fileName  fileNewName

git log
-- oneline
-- n10
-- all
-- graph 

git help git https://git.github.io/htmldocs/git.html
```



### .git 文件

```
.git 目录
HEAD 当前指向的分支
config 当前仓库的local配置
refs 引用的存放
- heads 分支
- tags 标签
objects

git 对象类型：commit（提交）、tree（树）、blob(文件)


git cat-file -t hashcode 查看对象类型
git cat-file -p hashcode 查看对象内容
git help git cat-file
```

commit tree blob 的关系

commit 中的tree：commit时仓库的快照。

tree：想象成一个文件夹，内部可以存文件夹（tree），也可以包含文件（blob）

问题：新建的git仓库，有且仅有一个commit，仅包含/t/read 那么，会生成几个对象，每个对象类型分别有几个？（1，2，1）

<img src="/Users/chenjiehao/Library/Mobile Documents/com~apple~CloudDocs/study/image/image-20211225233011139.png" alt="image-20211225233011139" style="zoom:33%;" />



### 分离头指针

detached HEAD 分离头指针：此时没有对应的分支与对应的操作进行挂钩

```
git checkout commitId   会导致分离头指针

git checkout -b branchName  commitId   从对应的commit 产生branch
```



### HEAD与branch

HEAD不仅可以指向branch，还可以指向commit。因为branch本身就是指向对应的commit

```
git diff branch  branch  比较分支
git diff HEAD  HEAD^1 比较commit
git diff HEAD  HEAD~1
git diff HEAD  HEAD^^
git diff HEAD  HEAD~2
```

