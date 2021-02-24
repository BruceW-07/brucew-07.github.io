# Git 学习笔记（详细版）


## 原因
最近又开始尝试搭建自己的博客了，还是 github 托管的方案，不过从 Hexo 换成了 Hugo。

上次尝试失败的原因貌似是不知道怎么将博客迁移到其他电脑上（毕竟我还没有自己的电脑，所以有在机房不同电脑上迁移的需求）。

现在博客的配置已经基本完成了，为防止再次弃坑，打算来系统学一下 Git，顺便丰富一下自己的知识库。

现在是 2021-01-29，星期五，晚上组里休息，那就开始吧。

参考资料 [[洛谷日报#199]Git 简单上手指南](https://studyingfather.blog.luogu.org/git-guide) by [Studying Father](https://studyingfather.blog.luogu.org/)

## 配置
### 用户信息
```bash
~$ git config --global user.name Example
~$ git config --global user.email Example@Example.com
```
这里的 `--global` 表示对系统中的所有仓库均有效，如果只想对单个仓库进行配置，只需在仓库的目录下执行去掉 `--global` 的命令即可。

### 设置编辑器
```bash
~$ git config --global core.editor emacs
```

## 基础操作

### 新建仓库
首先新建一个文件夹作为你的仓库，例如

```bash
~$ mkdir xxx
```

然后进入这个文件夹，并使用 `git init` 命令初始化仓库

```bash
~$ cd xxx
~/xxx$ git init
```

Git 将在 `~/xxx` 目录中新建一个文件夹 `.git`。输入指令 `ls -a`，将会看到
```bash
~/xxx$ ls -a
.  ..  .git
```

### 拷贝仓库
使用 `git clone` 命令可以远程拷贝先前创建的仓库，如
```bash
~$ git clone https://github.com/BruceW-07/brucew-07.github.io.git
```
~~（是的这是我博客部署文件的仓库地址）~~

### 跟踪文件
我们对仓库进行了一些修改后，需要将这些修改纳入版本管理中（我将其理解为推送到云备份上）。
而当我们不确定在上一次备份后进行了那些修改时，我们可以使用 `git status` 命令来查看当前仓库文件的状态。

当我们没有对仓库进行修改时，输入 `git status` 命令，将会显示以下内容

```bash
~/xxx$ git status
On branch master

Initial commit

nothing to commit (create/copy files and use "git add" to track)
```

而当我们对仓库进行了修改，比如说新建了一个 `README.md` 文件后，再输入 `git status` 命令，将会显示

```bash
~/xxx$ touch README.md
~/xxx$ git status
On branch master

Initial commit

Untracked files:
  (use "git add <file>..." to include in what will be committed)

	README.md

nothing added to commit but untracked files present (use "git add" to track)
```

这里 `Untracked files` 指的是 Git 之前没有纳入版本跟踪的文件。
参考资料中说
> 如果文件没有纳入版本跟踪，我们对该文件的修改不会被 Git 记录。

不是特别理解，~~不过大概就那个意思吧。~~

然后我们可以使用 `git add` 命令将这个文件纳入版本跟踪
```bash
~/xxx$ git add README.md
```

这时我们再输入 `git status`，就会显示
```
~/xxx$ git status
On branch master

Initial commit

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)

	new file:   README.md
```

表示 `README.md` 已经纳入了版本跟踪。

但需要注意的是，`git add` 命令只会将文件放入**暂存区**，并没有真正提交这次修改，我们需要使用 `git commit` 命令来将暂存区的文件提交到 Git 上

```bash
~/xxx$ git commit
```

接下来会弹出一个编辑器窗口，里面显示
```

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
#
# Initial commit
#
# Changes to be committed:
#	new file:   README.md
#
```

我们只需要在第一行输入 `commit`
```
commit
# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
# On branch master
#
# Initial commit
#
# Changes to be committed:
#	new file:   README.md
#
```

然后保存并退出即可。

终端中便会显示
```bash
[master (root-commit) 8c68837] commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 README.md
```
表示 `README.md` 已经成功从暂存区提交到 Git 上了。

我们再使用 `git status` 命令来查看一下仓库状态
```bash
~/xxx$ git status
On branch master
nothing to commit, working directory clean
```
~~一干二净~~

<br/>

我们再来考虑一种情况：对于一个文件 `README.md`，假如我们先用 `git add` 命令将它放入暂存区中，这时我们先不使用 `git commit` 命令提交，而是再修改一下 `README.md`，然后不把它重新放入暂存区，而是直接提交，会发生什么呢？

让我们来尝试一下。

先修改一波 `README.md`
```bash
~/xxx$ emacs README.md
# 随便写些东西（记得保存）
```

然后把它放进暂存区
```bash
~/xxx$ git add README.md
```

检查一下
```bash
~/xxx$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   README.md
```

然后再修改一波
```bash
~/xxx$ emacs README.md
# 再随便写些东西
```

看看现在的状态
```bash
~/xxx$ git status 
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   README.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   README.md
```

然后现在我们提交，并查看仓库状态
```bash
~/xxx$ git commit
# 在弹出的编辑器窗口中输入 "commit"，保存并退出
[master ac7a776] commit
 1 file changed, 1 insertion(+)
~/xxx$ git status 
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

发现在暂缓区中的 `README.md` 被提交到 Git 上了，而非暂缓区中的 `README.md` 仍只保存在本地

我们发现在刚才使用 `git status` 命令中，有这么两句话
```bash
(use "git reset HEAD <file>..." to unstage)
```
和
```bash
  (use "git checkout -- <file>..." to discard changes in working directory)
```

从字面意思~~以及结合上下文理解~~上看，第一条命令会使得暂缓区中对 `README.md` 的修改会被撤回，而第二条命令会使得非暂缓区中对 `README.md` 的修改被撤销。

我们一个个来试一下。

首先还是先修改 `README.md`，设内容为 `first`，
然后 `git add`，
然后再修改 `README.md`，设内容为 `second`，
再输入 `git status`，显示

```bash
~/xxx$ git status 
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   README.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   README.md
```

接着按照它的提示，输入
```bash
~/xxx$ git reset README.md
Unstaged changes after reset:
M	README.md
```

再输入 `git status`
```bash
~/xxx$ git status 
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   README.md

no changes added to commit (use "git add" and/or "git commit -a")
```

发现暂存区中的 `README.md` 消失了。打开 `README.md`，内容为 `second`

在试试第二个操作。这里，为了搞清楚操作后本地的文件是和 Git 上的同步还是和暂存区的同步，我们依次对 `README.md` 进行 3 次修改。

第一次内容为 `first`，修改后将其放入暂缓区，并提交至 Git 上；
第二次内容为 `second`，修改后将其放入暂缓区，但不提交；
第三次内容为 `third`，修改后不放入暂缓区。

操作完后，输入
```bash
~/xxx$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   README.md

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   README.md
```

然后输入
```bash
~/xxx$ git checkout README.md
```

查看状态
```bash
~/xxx$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

	modified:   README.md
```

打开 `README.md`，发现内容为 `second`。说明 `git checkout` 命令会使得非暂存区的文件与暂存区的保持一致。

