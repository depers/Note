---
typora-root-url: E:\markdown笔记\笔记图片
---

# Git

## Git基础

### 1.Git安装

![9-4](/9-4.png)

![9-3](/9-3.png)

![9-2](/9-2.png)

![9-1](/9-1.png)

安装教程：https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git

### 2.最小配置

![9-5](/9-5.png)

![9-6](/9-6.png)

### 3.创建第一个仓库并配置local用户信息

![9-7](/9-7.png)

* 添加local信息

  * 进入项目下，设置local的用户名和邮箱

    `git config --local user.name 'depers'`

    `git config --local user.email 'dev_fengxiao@163.com'`

    `git config --local --list`

  * 在本仓库设置了local信息，local参数的优先级高于global

### 4.通过几次commit来认识工作区和暂存区

![9-8](/9-8.png)

这几次提交使用的命令依次是：

* `git add -u`提交所有被删除和修改的文件到数据暂存区
* `git status`查看相关文件的状态
* `git commit -m ''`主要是将暂存区里的改动给提交到本地的版本库，-m 参数表示可以直接输入后面的“message”
* `git commit -am ''`可以用于提交已跟踪过，但未在暂存区的文件
* `git log`
  查看我们的提交。经过几次的提交，我们将代码提交到了git的暂存区，最后我们使用`git push`再将暂存区的代码提交到远程仓库里。
* `git add -u`、`git add -A与`git add .`的区别
  * `git add -A`  提交所有变化
  * `git add -u`  提交被修改(modified)和被删除(deleted)文件，不包括新文件(new)
  * `git add .`  提交所有变化
* `git commit -m`与`git commit -am`的区别
  * `git commit -m`只能提交暂存区中的文件到版本历史中
  * `git commit -am`可以用于提交已跟踪过到版本历史中，但未在暂存区的文件不能被提交到版本历史中。未在暂存区的文件必须先提交到暂存区才能被提交

### 5.给文件重命名的简便方法

例如我们要将项目中的readme文件改为readme.md文件

* 一般的方法

  * 执行`mv readme readme.md`
  * 执行`git status` 就会提示让我们进行下面的操作，`git add readme.md`,`git rm readme`
  * 执行`git add readme.md`
  * 执行`git rm readme`

  此时我们的修改就会被添加到暂存区。

* git的方法

  在进行这次操作之前，我们需要将刚才做的操作撤销，即将暂存区中的数据清除。执行`git reset --hard`

  * 只需执行`git mv readme readme.md`，就可以见重命名的操作添加到暂存区

  然后`git commit -m 'Move readme to readme.md'`

### 6.通过git log 查看版本演变历史

* `git log --oneline`查看

  `--oneline`参数可以将每条日志的输出为一行，如果日志比较多的话，用这个参数能够使结果看起来比较醒目。

* `git log -2`

  `-2`显示最近的2两条提交记录，可以与`--oneline`联用

* `git branch -v`查看本地分支

* `git checkout -b branch_name`创建并切换分支，`-b`参数表示创建并切换。相当于下面两条语句：

  * 创建分支： `git branch branch_name`
  * 切换分支： `git checkout branch_name`

* `git log --all` 查看所有分支演进历史

* `git log --all --graph` 用图形化的查看所有分支历史

* `git log --all --oneline --graph -4`用图形化的查看所有分支历史的前四个

* `git log v1.0 --oneline --graph -4`指定分支，用图形化的查看分支历史的前四个

* `git help --web log`在浏览器里查看git log的相关帮助


### 7.通过gitk图形化界面来管理git

切换到git管理的目录，在命令行中直接输入`gitk`，就可以使用图形界面的方式管理git

### 8. .git目录

* HEAD：存放了当前git工作的分支

* config：本地仓库相关的配置信息

* refs：

  存储分支的引用

  * heads

    在heads文件夹下有一个master文件，与HEAD文件中的内容相对应。master文件存储本地仓库分支最近一次commit对象的sha-1值。然后复制部分sha-1值使用`git cat-file -t sha-1部分值`查看类型，发现是**commit类型**

  * remote

    在该文件夹下有也有/orign/master文件，master文件远程仓库分支最近一次commit对象的sha-1值

  * tags

    存储了我们创建的标签信息，使用`cat tag_name`，查看标签内存储的sha-1值。然后复制部分sha-1值使用`git cat-file -t sha-1部分值`查看类型，发现是**tag类型**，然后使用`git cat-file -p sha-1部分值`根据对象的类型，以优雅的方式显式对象内容。我们可以看到创建该标签的相关信息

* objects

  在该文件中有许多两个字符命名的文件，其中有一个pack文件，在该文件夹下松散的文件如果过多的话，git就会将这些文件打包放到pack文件夹下面。其实objects放的就是我们每次提交的文件记录。

  我们打开一个名为**e8**的文件，会有一个**bdce6396b5fae03790136c3079b1435ac84a71**的文件，我们使用`git cat-file -t e8bdce6396b5fae0`这里注意这个sha-1值一定要加上文件夹e8才行。然后发现他的类型是**tree类型**。然后我们`git cat-file -p e8bdce6396b5fae0`，会有如下信息：

  ```
  100644 blob 0d1846c01d734464a30220a0a0f5293803266540    "\345\216\237\345\255\220\346\200\247\344\270\200.md"
  100644 blob d5475cd82f7be2a17697507d971cf1ebacf7a85e    "\345\216\237\345\255\220\346\200\247\344\272\214\342\200\224\342\200\224\351\224\201.md"
  100644 blob 266e4e0cb5f7bbf616b2537e1d7d5e135fec33eb    "\345\217\257\350\247\201\346\200\247.md"
  100644 blob 8341c51fdf4aadb2f968bb5ff5ef001a6c9c0df0    "\346\234\211\345\272\217\346\200\247.md"
  ```

  然后我们输入`git cat-file -t 0d1846c01d734464a30220a0a0f5293803266540`显示文件类型为**blob类型**，然后我们`git cat-file -p 0d1846c01d734464a30220a0a0f5293803266540`就可以看到文件的信息。

  **在git中如果文件的内容相同，那么他就是一个唯一的blob。**

### 9.commit、tree和blob三个对象之间的关系

![9-9](/9-9.png)

在上面这幅图中，我们通过`git cat-file -p 415c5c`可以看到这个commit下面的内容，这里面有一棵树tree 912fa6，然后我们`git cat-file -p 912fa6`。我们可以看到图中黄色的部分的第一个区域，里面有两个tree和两个blob。接着我们通过`git cat-file -p 6ad4c6`查看这个blob，发现他是一个我们曾经提交的html文件。

在git中如果文件的内容一样，那么他就是被唯一存储的blob。

以上图片和描述就是commit，tree和blob三个对象之间的关系。

### 10.小练习：数一数tree的个数

在使用git的时候，我们的文件夹下面必须有文件，如果只有文件夹而没有文件，git是不会理会我们的。

![9-10](/9-10.png)

当我们提交commit之前，在暂存区内只有blob对象，在我们提交commit之后，在暂存区内就会出现commit和tree对象。他们的关系就如上图所示。

下面是我们操作的命令：

```
冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects
$ git init
Initialized empty Git repository in E:/Dome/git/watch_objects/.git/

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ find .git/objects/ -type f

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ echo hello.world > readme

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ mkdir doc

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ ls
doc/  readme

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ mv readme doc/
l
冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ ls
doc/

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        doc/

nothing added to commit but untracked files present (use "git add" to track)

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ git add doc

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ find .git/objects/ -type f
.git/objects/72/b8294600e1ae132a15547d105c6a063958d07b

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ git cat-file -p 72b8294600
hello.world

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

        new file:   doc/readme


冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ git commit -m "Add readme"
[master (root-commit) 9a65f84] Add readme
 1 file changed, 1 insertion(+)
 create mode 100644 doc/readme

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ find .git/objects/ -type f
.git/objects/68/d6960edc84071d868de172ecb4852b69188f93
.git/objects/72/b8294600e1ae132a15547d105c6a063958d07b
.git/objects/9a/65f84cd5d451d7cbcbc43329d195e44f409548
.git/objects/b2/16938055751e4aad69b68c77e9786ce98fb800

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ git cat-file -t 68d6960edc8
tree

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ git cat-file -p 68d6960edc8
100644 blob 72b8294600e1ae132a15547d105c6a063958d07b    readme

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ git cat-file -t 72b8294600e
blob

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ git cat-file -p 72b8294600e
hello.world

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ git cat-file -t 9a65f84cd5d45
commit

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ git cat-file -p 9a65f84cd5d45
tree b216938055751e4aad69b68c77e9786ce98fb800
author depers <dev_fengxiao@163.com> 1552026977 +0800
committer depers <dev_fengxiao@163.com> 1552026977 +0800

Add readme

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ git cat-file -t b21693805
tree

冯晓@fulicpu MINGW64 /e/Dome/git/watch_objects (master)
$ git cat-file -p b21693805
040000 tree 68d6960edc84071d868de172ecb4852b69188f93    doc
```

其中有一条命令`find .git/objects/ -type f`:

根据文件类型来查找文件

* -type

  * f     // 普通文件
  * d     //目录文件
  * l     //链接文件
  * b     //块设备文件
  * c     //字符设备文件
  * p     //管道文件
  * s     //socket文件

### 11.分离头指针情况下的注意事项

> `detached HEAD`：分离头指针

* 首先输入`git branch -av`，查看当前分支

* 然后输入`git log`查看日志

* 进入分离头指针状态`git checkout 6108e78b90b13479eb `，然后会有如下输出：

  ```
  Note: checking out '6108e78b90b13479eb'.
  
  You are in 'detached HEAD' state. You can look around, make experimental
  changes and commit them, and you can discard any commits you make in this
  state without impacting any branches by performing another checkout.
  
  If you want to create a new branch to retain commits you create, you may
  do so (now or later) by using -b with the checkout command again. Example:
  
    git checkout -b <new-branch-name>
  
  HEAD is now at 6108e78... Modify readme 2
  ```

  分离头指针的风险就是，你可以基于这个commit的分类头指针进行开发 ，提交commit，并且不会影响其它分支。从本质上来说就是分离头指针是没有基于某个分支的去做的。但是如果有一天你切换到master去开发，git很可能会删除这个基于分离头指针开发的提交。

  他的好处：如果你想做一些尝试性的变更，你可以随时将他丢掉。你只需切换到其他的分支就好了。但是如果你认为他是重要的，一定要为他建立分支才行。

### 12.进一步理解HEAD和branch

当我们输入`cat .git/HEAD`我们就可以看到HEAD文件中存储的信息`ref: refs/heads/fix_readme`，储存着当前我们所指向的分支。接着我们输入`git cat-file -t HEAD`发现这个引用指向的属性`commit`，其实这样的话我们就可以巧用HEAD文件了，利用它指代的引用

* `git diff  HEAD HEAD^`、`git diff HEAD HEAD~`查看最后一次commit和他的上一次commit的区别
* `git diff HEAD HEAD^^`、`git diff HEAD HEAD~2`查看最后一次提交和上上一次提交的区别

## 用Git时的常见场景

### 13.删除不需要的分支

* 首先我们应输入`gitk --all`查看各个分支的情况
* 然后输入`git branch -d branch_name`
* 此时如果遇到`The branch xxxx is not fully merged`，我们应该输入`git branch -D branch_name`

