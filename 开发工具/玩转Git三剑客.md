# 玩转Git三剑客——极客时间

## 第一章 Git基础

### 1.Git安装

![](../笔记图片/9/9-4.png)

![9-3](../笔记图片/9/9-3.png)

![9-2](../笔记图片/9/9-2.png)

![9-1](../笔记图片/9/9-1.png)

安装教程：https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git

### 2.最小配置

![9-5](../笔记图片/9/9-5.png)

![9-6](../笔记图片/9/9-6.png)

### 3.创建第一个仓库并配置local用户信息

![9-7](../笔记图片/9/9-7.png)

* 添加local信息

  * 进入项目下，设置local的用户名和邮箱

    `git config --local user.name 'depers'`

    `git config --local user.email 'dev_fengxiao@163.com'`

    `git config --local --list`

  * 在本仓库设置了local信息，local参数的优先级高于global

### 4.通过几次commit来认识工作区和暂存区

![9-8](../笔记图片/9/9-8.png)

#### 这里需要注意的是

* `git add`命令会将文件交由git进行跟踪，添加到暂存区
* 暂存区会暂时的保存工作目录中的文件，你可以将暂存区的修改进行恢复，重新覆盖到工作目录
* 版本历史就是git的本地仓库

#### 这几次提交使用的命令依次是：

* `git add -u`提交所有被删除和修改的文件到数据暂存区
  * -u：更新已被跟踪的文件
* `git status`查看相关文件的状态
* `git commit -m ''`主要是将暂存区里的改动给提交到本地的版本库，-m 参数表示可以直接输入后面的“message”
* `git commit -am ''`可以用于提交已跟踪过但未在暂存区的文件（提交所有变化的文件）
* `git log`
  查看我们的提交。经过几次的提交，我们将代码提交到了git的暂存区，最后我们使用`git push`再将暂存区的代码提交到远程仓库里。

#### `git add -u`、`git add -A`与`git add .`的区别

* `git add -A`  提交所有变化
* `git add -u`  提交被修改(modified)和被删除(deleted)文件，**不包括新文件(new)，因为新文件并没有被跟踪**
* `git add .`  提交所有变化

#### `git commit -m`与`git commit -am`的区别

* `git commit -m`：提交暂存区中的文件到版本历史中
* `git commit -am`：提交跟踪过的文件到版本历史中，但未在暂存区的文件（也就是没有被跟踪的文件）不能被提交到版本历史中。未在暂存区的文件必须先提交到暂存区才能被提交（也就是要先`git add`）

### 5.给文件重命名的简便方法

例如我们要将项目中的readme文件改为readme.md文件

* 一般的方法

  * 执行`mv readme readme.md`
  * 执行`git status` 就会提示让我们进行下面的操作，`git add readme.md`,`git rm readme`
  * 执行`git add readme.md`
  * 执行`git rm readme`

  此时我们的修改就会被添加到暂存区。

* git的方法（推荐）

  在进行这次操作之前，我们需要将刚才做的操作撤销，即将**暂存区**中的数据清除。执行`git reset --hard`

  * 只需执行`git mv readme readme.md`，就可以见重命名的操作添加到**暂存区**

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

切换到git管理的目录，在命令行中直接输入`gitk`，就可以使用图形界面的方式管理git。

在下面这幅图中有两个点我这边着重强调一下：

* Patch和Tree

  * Patch：指的是本次commit涉及修改的文件内容是什么
  * Tree：指的是这个项目的目录结构

* Author和Committer

  正常情况下，Author和Committer是一个人，但是有一种情况cherry-pick，你挑拣过来的commit中就会生成新的commit，这个新的commit的Author就是原来的作者，这样做是为了尊重版权。而你作为挑拣代码的人，你就是新的commit的committer。

![](../笔记图片/9/52.jpg)

### 8. .git目录

* .git：.git下面的文件就是裸仓库(bare repository)

* HEAD：存放了当前git工作分支的引用

  ![](../笔记图片/9/55.jpg)

* config：本地仓库相关的配置信息

  ![](../笔记图片/9/56.jpg)

  关于git config的几条命令：

  * 查看本地配置：`git config --local --list`
  * 查看本地user.name：`git config --local user.name`
  * 修改参数：`git config --local user.name 'suling'`

* refs：

  存储分支的引用

  * heads

    在heads文件夹下有一个master文件（也就是master分支），与HEAD文件中的内容相对应。master文件存储本地仓库分支最近一次commit对象的sha-1值。然后复制部分sha-1值使用`git cat-file -t sha-1部分值`查看类型，发现是**commit类型**

    ![](../笔记图片/9/57.jpg)

  * remote

    在该文件夹下有也有/orign/master文件，master文件远程仓库分支最近一次commit对象的sha-1值

  * tags

    存储了我们创建的标签信息，使用`cat tag_name`，查看标签内存储的sha-1值。然后复制部分sha-1值使用`git cat-file -t sha-1部分值`查看类型，发现是**tag类型**，然后使用`git cat-file -p sha-1部分值`根据对象的类型，以优雅的方式显式对象内容。我们可以看到创建该标签的相关信息

* objects

  在该文件中有许多两个字符命名的文件夹，其中有一个pack文件夹，在该文件夹下松散的文件如果过多的话，git就会将这些文件打包放到pack文件夹下面。其实objects放的就是我们每次提交的文件记录。

  ![](../笔记图片/9/58.jpg)

  我们打开一个名为**e8**的文件，会有一个**bdce6396b5fae03790136c3079b1435ac84a71**的文件，我们使用`git cat-file -t e8bdce6396b5fae0`这里注意这个sha-1值一定要加上文件夹`e8`才行。然后发现他的类型是**tree类型**。然后我们`git cat-file -p e8bdce6396b5fae0`，会有如下信息：

  ```
  100644 blob 0d1846c01d734464a30220a0a0f5293803266540    "\345\216\237\345\255\220\346\200\247\344\270\200.md"
  100644 blob d5475cd82f7be2a17697507d971cf1ebacf7a85e    "\345\216\237\345\255\220\346\200\247\344\272\214\342\200\224\342\200\224\351\224\201.md"
  100644 blob 266e4e0cb5f7bbf616b2537e1d7d5e135fec33eb    "\345\217\257\350\247\201\346\200\247.md"
  100644 blob 8341c51fdf4aadb2f968bb5ff5ef001a6c9c0df0    "\346\234\211\345\272\217\346\200\247.md"
  ```

  操作示例：进入一个叫做fb的文件（图文无关，图片只是解释）

  ![](../笔记图片/9/59.jpg)

  操作示例：查看一个tree（图文无关，图片只是解释）

  ![](../笔记图片/9/60.jpg)

  然后我们输入`git cat-file -t 0d1846c01d734464a30220a0a0f5293803266540`显示文件类型为**blob类型**，然后我们`git cat-file -p 0d1846c01d734464a30220a0a0f5293803266540`就可以看到文件的信息。

  操作示例：查看一个blob（图文无关，图片只是解释）

  ![](../笔记图片/9/61.jpg)

  从上面的实验中我们学习了git的三种文件类型，分别是commit、tree和blob。

  **彩蛋：** 在git中如果文件的内容相同，那么他就是一个唯一的blob。

### 9.commit、tree和blob三个对象之间的关系

![9-9](../笔记图片/9/9-9.png)

在上面这幅图中，我们通过`git cat-file -p 415c5c`可以看到这个commit下面的内容，这里面有一棵树tree 912fa6，然后我们`git cat-file -p 912fa6`。我们可以看到图中黄色的部分的第一个区域，里面有两个tree和两个blob。接着我们通过`git cat-file -p 6ad4c6`查看这个blob，发现他是一个我们曾经提交的html文件。

在git中如果文件的内容一样，那么他就是被唯一存储的blob。

以上图片和描述就是commit，tree和blob三个对象之间的关系。

### 10.小练习：数一数tree的个数

在使用git的时候，我们的文件夹下面必须有文件，如果只有文件夹而没有文件，git是不会理会我们的。

![9-10](../笔记图片/9/9-10.png)

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

### 14.怎么修改最新commit的message？

* 首先查看最新的commit：`git log -1`

* 输入修改最新commit命令：`git commit --amend`，然后会出现这样的界面：

  ![11](../笔记图片/9/11.PNG)

  然后根据Linux操作vim的方式修改commit信息就行，最后输入`wq!`即可。

* 最后在输入`git log -1`就可以看到最新的commit信息已经修改了。

* 如果要想将此修改信息推送的github，应使用命令`git push --force`

### 15.怎么修改老旧commit的message[变基操作]？

* 这次我们修改最后第二次的commit信息，首先查看一下最近三次的commit信息：

  ![12](../笔记图片/9/12.PNG)

* 使用命令`git rebase -i 上一次commit的id号（切记，不是最后第二次的id，而是他父级也就是上一级的id） `，这里应该填写的命令是：`git rebase -i `

* 在下面这幅图中可以看到，`pick`和`reword`两个命令参数。这里我们需要使用的是reword，这个命令参数的意思是：**我们仍然保留commit的变更的文件，但是我只修改commit信息**。

  ![13](../笔记图片/9/13.png)

* 所以我们应该修改上面的命令参数位reword，因为我们只改最后第二条的commit信息，所以只reword他。最后`wq!`：

  ![14](../笔记图片/9/14.png)
  
* 然后可以看到下图，在这里我们就可以修改我们的commit信息了，修改完之后`wq!`即可。

  ![16](../笔记图片/9/16.png)
* 如果要想将此修改信息推送的github，应使用命令`git push --force`

### 16.怎样把连续的多个commit整理成1个？

* 使用git log查看提交日志，我们想将下面红线圈住的多次提交合为一次提交：

  ![17](../笔记图片/9/17.png)

* 这里我们输入`git rebase -i f09122cf`，这里的id是这四次提交的父id：

  ![18](../笔记图片/9/18.png)

* 这里我们将需要合并提交的前三次前面的命令参数改为`squash`简写s，然后`wq!`：

  ![19](../笔记图片/9/19.png)

* 注意，这里需要在`# Thisis combination of 4 commits`下输入本次合并提交的commit信息，然后`wq!`：

  ![20](../笔记图片/9/20.png)

* 然后使用git log查看，发现已经刚才的4项提交已经合为一项了。

  ![21](../笔记图片/9/21.png)

### 18.怎样把间隔的几个commit整理成1个？

下图中有三个commit，此时我们有一个需求就是将最新的commit和第一个commit 进行合并。

![22](../笔记图片/9/22.png)

此时我们使用`git rebase -i 2642930adaf5696`，其中2642930adaf5696位第一次提交的id。接着我们会进入这样一个界面：

![23](../笔记图片/9/23.PNG)

此时我们需要将第一个commit信息添加进来`pick 2642930`到第一行。然后再将最新的一次commit放到第二行，修改为`s c3e54dd edit readme`，s指的是squash。然后`wq!`

![25](../笔记图片/9/24.PNG)

接着我们并没有像把连续的多个commit整理成1个那样进入commit信息的修改界面。此时我们输入`git status`提示我们输入`git rebase --continue`。然后我们才进入commit信息的修改界面，如下图：

![25](../笔记图片/9/25.PNG)

接着我们添加合并后的commit信息，然后`wq!`：

![26](../笔记图片/9/26.png)

看到下图说明我们已经合并成功了！

![27](../笔记图片/9/27.png)

此时刚开始的三个commit log已经变成了两个，我们已经成功的将第一个和第三个commit合并了：

![28](../笔记图片/9/28.png)

### 19.怎么比较暂存区和HEAD所含文件的差异？

* 查看git状态：`git status`，发现当前暂存区没有可以内容是可以提交的

* 然后编辑index.html，添加`add`

  ![29](../笔记图片/9/29.png)

* 接着`git add .`将修改的内容添加到暂存区

* 然后输入：`git diff --cached`比较暂存区和HEAD所含文件的差异，HEAD指向了我们最近一次的commit。

  ![30](../笔记图片/9/30.png)

### 20.怎么比较工作区和暂存区所含文件的差异？

比较工作区和暂存区的文件差异命令：`git diff `，该命令会展示所有文件的工作区和暂存区的区别

如果你想查看特定文件的工作区和暂存区的区别：`git diff -- ./css/style.css`

### 21.如何让暂存区恢复成和HEAD的一样？

* 应用场景：暂存区的内容不够成熟，我们需要将暂存区的文件恢复成HEAD的那次提交。使用命令：`git reset HEAD`,原有的那些操作退回到工作区。

* 注意：如果你在使用`git reset HEAD`命令之前，在工作区中对文件又做了改动。这个不用担心，使用`git reset HEAD`命令后退回的文件内容与之前的改动都在，也就是说暂存区恢复不会导致工作区内容的丢失。

### 22.如何让工作区的文件恢复为和暂存区一样？

* 应用场景：比如你修改了index.html文件并将它加入了暂存区，然后你接着修改index.html，然后你又发现新的index.html还不如在暂存区的index.html。所以**你想将工作区的文件恢复为和暂存区一样**。
* 命令：`git checkout -- index.html`

### 23.怎样取消暂存区部分文件的更改（暂存区一部分文件恢复成和HEAD一样）？

* 应用场景：在暂存区中有很多的文件，但是只有一部分想恢复成和HEAD一样

  在下图中我想恢复暂存区的index.html和HEAD一样，这里我们执行`git reset HEAD -- index.html`

  ![32](../笔记图片/9/32.png)

  执行完上述命令之后，我们的之前对index.html添加到暂存区的变更已经被退回到工作区了。我们使用`git status`就可以看到。

  ![33](../笔记图片/9/33.png)

  然后我们再比较一下暂存区和HEAD所含文件的区别，使用`git diff --cached` ，发现他们的差别只有style.css，而index.html已经没有差别了：

  ![34](../笔记图片/9/34.png)

  如果想将多个在暂存区的文件恢复成和HEAD一样，可以使用`git reset HEAD -- style.css index.html`

### 24.消除最近的几次提交

应用场景：我们想将头指针指到倒数第三次提交上，消除最后两次提交。

![35](../笔记图片/9/35.png)

这时，使用命令：`git reset --hard 1ab01586488247b25c`

![36](../笔记图片/9/36.png)

### 25.查看不同提交的指定文件的差异

查看两次不同commit的文件差异：`git diff 1ab01586488247b f870e2abc8dca56`

查看不同提交的指定文件的差异：`git diff 0cefd906dce41cede5 1ab01586488247b2 -- index.html`

![37](../笔记图片/9/37.png)

### 26.正确删除文件的方法

可以参考第5节的重命名，步骤：

* `git rm readme`，这样会直接将删除操作体现在暂存区中
* `git commit -m ...`，就可以将删除操作提交到本地仓库中了

## 第七章 使用GitHub进行团队协作

### 48. 怎样选择适合自己团队的工作流？

![](../笔记图片/9/38.jpg)

![](../笔记图片/9/39.jpg)

![](../笔记图片/9/40.jpg)

![](../笔记图片/9/41.jpg)

![](../笔记图片/9/42.jpg)

![](../笔记图片/9/43.jpg)

![](../笔记图片/9/44.jpg)

### 49.如何挑选合适的分支集成策略？

github提供的三种分支合并策略，如下图，分别是merge commits、squash merging、rebase merging。

![](../笔记图片/9/45.jpg)

在视频中，作者演示项目的git提交的原本时间线如下：

![](../笔记图片/9/46.jpg)

#### merge commits

创建一个pr，在分支合并的时候采取的策略选择merge commits，该种策略会直接将Beijing这个分支合并到master分支上去，效果如下：

![](../笔记图片/9/47.jpg)

#### squash merging

新建一个pr，策略选择squash merging，该策略不改变原有分支，将Beijing分支的三次提交合并成一次提交，然后直接将这一次提交挂到master分支上去。这样做的话，master分支看上去就是一条直线。

![](../笔记图片/9/48.jpg)

#### rebase merging

新建一个pr，策略选择rebase merging，该策略不改变原有分支，他会将Beijing这个分支的三次提交直接挂载到master分支上去。这样做的话，master分支看上去就是一条直线。

![](../笔记图片/9/49.jpg)



### 50.启用issue跟踪需求和任务

**issue的定义**：将轻量级任务跟踪集成到存储库中。将项目与问题标签和里程碑保持在轨道上，并在提交消息中引用它们。

![](../笔记图片/9/51.jpg)

在github一个开源项目的Settings中可以设置issue是否开启并定制他的模板。定制好自己的模板后，模板文件会保存到项目代码的**.github**文件夹下。

我们可以将需求和问题直接提在issue中，和别人一起讨论。

### 51.如何用project管理issue？

Github上面对于Project的定义：GitHub上的项目板可以帮助您组织和优先排序您的工作。您可以为特定的特性工作、全面的路线图，甚至发布检查列表创建项目板。

下面我简单介绍一下，如下图所示，我新建了五个project：

![](../笔记图片/9/53.jpg)

test2到test5都是使用模板新建的，每一种模板的都有自己的特殊的功能，这里我就不多讲了。

例如带了review功能的自动看板：

![](../笔记图片/9/54.jpg)

他的一些流程的自动化的，在TODO那一栏除了可以管理issue和pull request，还可以书写一些笔记在里面。

### 52.项目内部怎么实施code review的

这节视频主要讲了通过github进行branch的设置，要求pr必须被审核人review之后才能被合并进入受保护的分支。

目前这个功能对于普通的github用户是关闭的。

# 其他

## 1. cherry pick的用法

参考：https://ruanyifeng.com/blog/2020/04/git-cherry-pick.html