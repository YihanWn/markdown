# tmux学习

会话，任务：session

窗口：windows

窗格：pane

**普通的终端：**窗口和其中由于session（任务）而启动的进程是连在一起的，关闭窗口，session就结束了，session内部的进程也会终止，不管是否运行完。、

但是在具体使用中，我们==希望当前的session隐藏起来，在终端中做其他事情，但是又不希望session及其进程被关闭==。这样就需要用到tmux，对session进行解绑。之后再想继续出来这个session的时候，再次绑定就可以回到之前的工作状态。

对于一个session，可以创建好几个window

对于window可以理解为一个工作区，一个窗口，每一个窗口，都可以将其分解为几个pane小窗格

所以，关于session、window、pane的关系是：[pane∈window]∈session

## 1、安装、启动、退出

```bash
# 安装
sudo apt-get install tmux
# 启动tmux
tmux
# 启动命名tmux
tmux new -s <name>
# 退出（退出后无法恢复）
exit 或 Ctrl+D 或 tmux kill-session -t your-session-name
```

在终端窗口上，运行tmux，其实就打开了一个终端与tmux服务的会话。只不过我们可以在tmux会话上层，再次输入’会话‘命令，使tmux上层运行的'会话'与终端窗口进行分离。这里面tmux其实可以称之为伪窗口（它其实是会话）。

启动tmux后，底部[0] 表示第0个tmux伪窗口，再启动一个tmux伪窗口，则为[1],依次递增。

如果使用命令启动tmux，底部不再是数字，而是命名的名字

## 2、分离（隐藏）

在tmux窗口中，输入以下命令，就会将当前session与窗口分离（隐藏），==session转到后台执行==，上一个会话（session）并没有被关闭：

```bash
tmux detach
```

## 3、绑定、解绑、切换session

假设现在正处于session1，使用分离操作就是将session1进行解绑:

```bash
tmux detach
```

而如果你想再次绑定session1，可以使用命令：

```bash
tmux attach -t your-session-name
或
tmux attach -t your-session-name
```

注：该命令需要退出或隐藏该session，然后在终端中使用 `tmux attach -t your-session-name`命令进入对应的session

若在session中输入该命令，报错：`sessions should be nested with care, unset $TMUX to force`

错误原因：

已经打开了一个tmux 会话，然后在这个tmux会话中试图打开另一个tmux会话；

这种嵌套的，一层套一层的，在虚拟会话中声明活着打开另一个虚拟回话，是不好的。

解决方式：

直接在命令行重新打开或者新建会话。

## 4、重命名窗口

```bash
tmux rename-window -t old_name new_name
```

## 5、pane操作

### 5.1划分表格

```bash
# 划分为上下两个窗格
tmux split-window

# 划分左右两个窗格
tmux split-window -h

或

左右划分：ctrl+b %
上下划分：ctrl+b "
```

### 4.2 切换pane

```
ctrl+b arrow-key（方向键）
```

光标切换到其他窗格

### 4.3 交换窗格位置

```bash
# 当前窗格往上移
tmux swap-pane -U

# 当前窗格往下移
tmux swap-pane -D
```

### 4.4 关闭窗格

```
ctrl+d
```

如果只有一个窗格就是关闭window

## 5、其他操作

```bash
# 列出所有快捷键，及其对应的 Tmux 命令
$ tmux list-keys

# 列出所有 Tmux 命令及其参数
$ tmux list-commands

# 列出当前所有 Tmux 会话的信息
$ tmux info

# 重新加载当前的 Tmux 配置
$ tmux source-file ~/.tmux.conf
```

## 5.1 tmux上下翻屏

使用快捷键`ctrl+b [ `，就可以通过方向键上下移动使用`PageUp`和`PageDown`可以实现上下翻页



# 美化终端

备份原有配置
```bash
cp ~/.tmux.conf ~/.tmux.conf.bak
```

拉取新配置，并部署配置文件
```bash
cd ~/
git clone https://github.com/gpakosz/.tmux.git
ln -s -f .tmux/.tmux.conf
cp .tmux/.tmux.conf.local .
```
.tmux.conf.local 会被 .tmux.conf 自动导入。

更新之后，新配置不会生效，需要 reload tmux 配置；或者 kill 掉 tmux 进程，重启 tmux。

tmux reload config
```bash
tmux source-file ~/.tmux.conf
```
如果报错 no server running on /tmp/tmux-1000/default

修改 ~/.tmux.config, 注释掉配置, 该配置已废弃
```bash
set -g status-utf8 on 
```

# git

## init
建立一个文件夹，作为git repository

进入建立的文件夹(git_repository)中
git init

或指定目录作为Git仓库。
git init git_repository

git会在该目录下建立一个 .git 文件夹

## 添加新文件

我们有一个仓库，但仓库中什么也没有，建立一个文件夹git_test,使用add命令添加文件
git add git_test

## 提交版本

现在我们已经添加了这些文件，我们希望它们能够真正被保存在Git仓库。
为此，我们将它们提交到仓库。

git commit -m "Adding files"