# Git 学习笔记（简略版）


## 配置
### 用户信息
```bash
~$ git config --global user.name Example
~$ git config --global user.email Example@Example.com

### 编辑器
```bash
~$ git config --global core.editor emacs
```

## 基础操作

### 新建仓库
```bash
~/xxx$ git init
```

### 拷贝仓库
```bash
~$ git clone https://github.com/BruceW-07/brucew-07.github.io.git
```

### 跟踪文件
#### 查看仓库状态

```bash
~/xxx$ git status
```

#### 将文件放入暂存区

```bash
~/xxx$ git add README.md
```

如果需要将所有文件全部放入暂存区，可以使用参数 `-A`
```bash
~/xxx$ git add -a
```

但需要注意的是，`git add` 命令只会将文件放入**暂存区**，并没有真正提交这次修改。

#### 将文件提交至 Git

```bash
~/xxx$ git commit
```

接下来会弹出一个编辑器窗口，输入 `commit` 并保存退出即可。

## 远程操作
#### 将 Git 中信息推送至服务器
例如，将信息推送至 origin 仓库中的 master 分支
```bash
~/xxx$ git push origin master
```
