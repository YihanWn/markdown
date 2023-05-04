# First Exploration with GNU/Linux

## Setting APT source file

```bash
sudo bash -c 'echo "deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse" > /etc/apt/sources.list'
```

## Updating APT package information

```shell
sudo apt-get update
```

## Installing tools for PAs

```shell
sudo apt-get install build-essential    # build-essential packages, include binary utilities, gcc, make, and so on
sudo apt-get install man                # on-line reference manual
sudo apt-get install gcc-doc            # on-line reference manual for gcc
sudo apt-get install gdb                # GNU debugger
sudo apt-get install git                # revision control system
sudo apt-get install libreadline-dev    # a library used later
sudo apt-get install libsdl2-dev        # a library used later
sudo apt-get install llvm llvm-dev      # llvm project, which contains libraries used later
sudo apt-get install llvm-11 llvm-11-dev # only for ubuntu20.04
```

## Installing Chinese input method

### 1、安装中文语言包

`Setting` -> `Region & Language` -> `Manage Install Language`

在 `Language Support` 窗口中，单击 `Install/Remove Languages...`

在弹出的 `Installed Languages` 窗口中，找到 `Chinese(simplified)`，然后将其勾选，最后单击 `Apply`

### 2、安装中文输入法

```shell
sudo apt-get install ibus-pinyin
```

点击 `Settings`，在弹出的窗口中，点击 `Keyboard`

在 `Keyboard` 窗口中，单击 `Input Sources` 下的 `+`，随后弹出新窗口

在 `Add an Input Source` 窗口中选择 `Chinese`，然后选择 `Chinese(Intelligent Pinyin)` 并单击 `Add` 按钮

如果没有`Chinese(Intelligent Pinyin)`可以进行重启（其他输入法无法输入中文）

通过 `shift` 键或者 `Windows + space` 组合键就可以进行切换输入法

# Vim

```shell
sudo apt-get install vim
```

## Enabling syntax highlight

VIM语法高亮 

[VIM学习笔记 语法高亮 (Syntax Highlight) - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/25292625)

`diff` 查看文件改动，`diff`格式是一种描述文件改动的常用方式

https://www.ruanyifeng.com/blog/2012/08/how_to_read_diff.html

## Enabling more vim features

设置 `/etc/vim/vimrc`，`"`为注释掉的功能，通过以下选项设置vim

```shell
" setlocal noswapfile " 不要生成swap文件
" set bufhidden=hide " 当buffer被丢弃的时候隐藏它
" colorscheme evening " 设定配色方案
set number " 显示行号
" set cursorline " 突出显示当前行
" set ruler " 打开状态栏标尺
" set shiftwidth=2 " 设定 << 和 >> 命令移动时的宽度为 2
" set softtabstop=2 " 使得按退格键时可以一次删掉 2 个空格
" set tabstop=2 " 设定 tab 长度为 2
" set nobackup " 覆盖文件时不备份
" set autochdir " 自动切换当前目录为当前文件所在的目录
" set backupcopy=yes " 设置备份时的行为为覆盖
" set hlsearch " 搜索时高亮显示被找到的文本
" set noerrorbells " 关闭错误信息响铃
" set novisualbell " 关闭使用可视响铃代替呼叫
" set t_vb= " 置空错误铃声的终端代码
" set matchtime=2 " 短暂跳转到匹配括号的时间
" set magic " 设置魔术
" set smartindent " 开启新行时使用智能自动缩进
" set backspace=indent,eol,start " 不设定在插入状态无法用退格键和 Delete 键删除回车符
" set cmdheight=1 " 设定命令行的行数为 1
" set laststatus=2 " 显示状态栏 (默认值为 1, 无法显示状态栏)
" set statusline=\ %<%F[%1*%M%*%n%R%H]%=\ %y\ %0(%{&fileformat}\ %{&encoding}\ Ln\ %l,\ Col\ %c/%L%) " 设置在状态行显示的信息
" set foldenable " 开始折叠
" set foldmethod=syntax " 设置语法折叠
" set foldcolumn=0 " 设置折叠区域的宽度
" setlocal foldlevel=1 " 设置折叠层数为 1
" nnoremap <space> @=((foldclosed(line('.')) < 0) ? 'zc' : 'zo')<CR> " 用空格键来开关折叠
```

# SFTP SERVER

```bash
sudo apt-get install vsftpd
sudo vim /etc/vsftpd.conf
```

取消掉两行注释

`local_enable=YES `

`write_enable=YES`

保存后重启`sftp`服务

```bash
sudo /etc/init.d/vsftpd restart
```



# More Exploration

## 1、tmux

`tmux` is a terminal multiplexer. With it, you can create multiple terminals in a single screen. It is very convenient when you are working with a high resolution monitor. To install `tmux`, just issue the following command:

```shell
sudo apt-get install tmux
```

Now you can run `tmux`, but let's do some configuration first. Go back to the home directory:

```text
cd ~
```

New a file called `.tmux.conf`:

```text
vim .tmux.conf
```

Append the following content to the file:

```text
bind-key c new-window -c "#{pane_current_path}"
bind-key % split-window -h -c "#{pane_current_path}"
bind-key '"' split-window -c "#{pane_current_path}"
```

These three lines of settings make `tmux` "remember" the current working directory of the current pane while creating new window/pane.

Maximize the terminal windows size, then use `tmux` to create multiple normal-size terminals within single screen. For example, you may edit different files in different directories simultaneously. You can edit them in different terminals, compile them or execute other commands in another terminal, without opening and closing source files back and forth. You can scroll the content in a `tmux` terminal up and down. For how to use `tmux`, please STFW.

[Tmux 使用教程 - 阮一峰的网络日志 (ruanyifeng.com)](https://www.ruanyifeng.com/blog/2019/10/tmux.html)

# Getting Source Code for PAs

## github添加ssh key

```shell
cd ~/.ssh
ls
```

查看是否有`id_rsa  id_rsa.pub`

若没有，则使用

```shell
ssh-keygen -t rsa -C 1286711270@qq.com
```

本地创建 ssh key

参数含义：
-t： 指定密钥类型，默认rsa，可省略
-C：设置注释文字，比如邮箱（会放在公钥里）
-f： 指定密钥文件存储文件名

在github上添加ssh key

登录github

右上角点击Settings
\>>SSH and GPG keys
\>>new SSH key
\>>将复制的公钥（id_rsa.pub）代码添加到Key中，并键入Title
\>>点击add SSH key
\>>done

测试是否完成

```shell
ssh -T git@github.com
```

输入yes，continue connection，若successfully authenticated，则成功



## 获取一生一芯框架代码

```shell
git clone -b ysyx2204 git@github.com:OSCPU/ysyx-workbench.git
```

修改`ysyx-workbench/Makefile`中的学号和姓名时