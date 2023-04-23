# 阅读YEMU

## 首先查看make的过程
强制重新编译重新编译，并在vim中查看，可以在vim中处理，提高可读性
```bash
make -nB | vim - 
```
使用该命令，只保留gcc或g++开头的行
```bash
:%!grep "^\(gcc\|g++\)"
```
:  后面跟shell命令
%  编辑区中所有的内容
:% 将编辑区中所有的文字送到后面的grep命令中，grep后面加正则表达式完成过滤功能

将环境变量$NEMU_HOME所指示的
```bash
:%!sed -e "s+$NEMU_HOME+\$NEMU_HOME+g"
```

将\$NEMU_HOME/build/obj-risc64-nemu-interpreter替换为$OBJ_DIR
```bash
:%s+\$NEMU_HOME/build/obj-risc64-nemu-interpreter+$OBJ_DIR+g
```

将-c之前的内容替换为$CFLAGS
```bash
:%s/-O2.*=risc64/$CFLAGS/g
```

将最后一行的空格替换成换行并缩进两格
```bash
:$s/ */\r /g
```

## 查看完make过程可查看Makefile的内容


# 搭建NEMU

```bash
bash init.sh nemu
source ~/.bashrc
make run
```

```bash
vim nemu/src/monitor/monitor.c
```
注释掉`assert(0)`,再次编译代码
```bash
make run
```

# 阅读NEMU

## 找main函数
```bash
1、find . | grep main  # 返回含“main”的文件的路径
2、grep -nr "\bmain\b" newmu/src   # main函数前后没有字母，使用\bmain\b, \b表示字符边界
```
使用GDB的方法找main函数（推荐），状态机模型的启发，状态机是动态的
```bash
make menuconfig
make clean
make gdb
(gdb) b main   # 在main处打断点，找到main函数所在文件及其行数
```


## GDB的使用
通过TUI（Text User Interface）在GDB打开一个源码窗口
```bash
（gdb）layout src  # 可以边调试边查看源码
```
`b`设置断点/`watch`设置监视点

`r`重新开始运行

`s`运行下一条代码

`n`运行下一条代码，不进入函数

`p`打印变量或寄存器的值

`x`扫描内存

`bt`查看调用栈

`help xxx`查看xxx命令的帮助

`finish`执行直到当前函数返回

其他命令查手册