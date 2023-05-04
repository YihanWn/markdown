# 搭建verilator仿真环境

Prerequisites

```bash
sudo apt-get install git perl python3 make autoconf g++ flex bison ccache
sudo apt-get install libgoogle-perftools-dev numactl perl-doc
sudo apt-get install libfl2 # Ubuntu only (ignore if gives error)
sudo apt-get install libfl-dev # Ubuntu only (ignore if gives error)
sudo apt-get install zlibc zlib1g zlib1g-dev # Ubuntu only (ignore if gives >error)
```

安装`verilator`

```bash
sudo apt-get install verilator   # On Ubuntu
```
通过github
```bash
git clone https://github.com/verilator/verilator
unsetenv VERILATOR_ROOT # For csh; ignore error if on bash
unset VERILATOR_ROOT # For bash
cd verilator
git pull # Make sure git repository is up-to-date
git tag # See what versions exist
#git checkout master # Use development branch (e.g. recent bug fixes)
#git checkout stable # Use most recent stable release
git checkout v4.210 # Switch to specified release version

autoconf # Create ./configure script
./configure # Configure and create Makefile
make -j nproc # Build Verilator itself (if error, try just ‘make’)
sudo make install
```
注意环境变量的配置

# 使用verilator进行实验

## 准备文件

rtl文件：

```verilog
module our_OnOff(
  input a,
  input b,
  output f
);
  assign f = a ^ b;
endmodule
```

测试用`main.c`文件：

注意：==此处必须为`main.c`文件==

因为使用`verilator -Wno-fatal our_OnOff.v main.c --top-module our_OnOff --cc --trace --exe`

生成的目标文件夹`obj_dir`中的`Vour_OnOff.mk`文件中，指定：编译生成main.o文件需要main.c文件

```c
#include "verilated_vcd_c.h" //可选，如果要导出vcd则需要加上
#include "Vour_OnOff.h"
#include "stdio.h"
#include <stdlib.h>

vluint64_t main_time = 0;  //initial 仿真时间
 
double sc_time_stamp()
{
    return main_time;
}
 
int main(int argc, char **argv)
{
    Verilated::commandArgs(argc, argv); 
    Verilated::traceEverOn(true); //导出vcd波形需要加此语句
 
    VerilatedVcdC* tfp = new VerilatedVcdC; //导出vcd波形需要加此语句
 
    Vour_OnOff *top = new Vour_OnOff("top"); //调用VAccumulator.h里面的IO struct
 
    top->trace(tfp, 0);   
    tfp->open("wave.vcd"); //打开vcd
 
    while (sc_time_stamp() < 20 && !Verilated::gotFinish()) { //控制仿真时间
        int a = rand() & 1;
        int b = rand() & 1;
		top->a = a;
		top->b = b;
		top->eval();
		printf("a = %d, b = %d, f = %d\n", a, b, top->f);
		tfp->dump(main_time); //dump wave
        main_time++; //推动仿真时间
    }
    top->final();
    tfp->close();
    delete top;
 
    return 0;
}
```

## 编译过程

生成目标文件夹：

`verilator -Wno-fatal our_OnOff.v main.c --top-module our_OnOff --cc --trace --exe`

编译：

`make -C obj_dir -f Vour_OnOff.mk`

运行：

`./obj_dir/Vour_OnOff`

# 编译命令解析

（以二选一选择器为例）

### 1、.v文件与模块名

​	若.v文件为mux21，模块名为m_mux21

### 2、仿真代码（main.c）

仿真代码中注意引用的.h文件名称为：==V模块名.h，如#include “Vm_mux21.h”==

### 3、生成目标文件夹

`verilator -Wno-fatal mux21.v main.c --top-module m_mux21 --cc --trace --exe`

-Wno: 忽略非 fatal 的 warning
mux21.v: 是设计文件
main.c"是主程序
--top-module:顶层模块名，注意是模块名，不是文件名
--cc:表明是C++，不过 c 程序也是支持的
--trace 表明会追踪波形，如果需要导出vcd 或者 fst 等其他波形文件，需要加上这个选项
--exe:生成可执行文件
也可以加 --build, verilator自己执行obj_dir中的.mk文件，不需要单独写compile

verilator -Wno-fatal ==mux21.v== main.c --top-module ==m_mux21== --cc --trace --exe

​								aaa										 bbb

==注意文件名与模块名==，否则会报错：在aaa文件中没有找到bbb模块



生成的目标文件为：V + 模块名 + xxx，如：Vm_mux21.mk， Vm_mux21.h 

注意此处生成的是：==V模块名.h！！！不是V文件名.h==

### 4、编译

`make -C obj_dir -f Vm_mux21.mk`

 -C 选项改变目录，make 命令首先切到特定的目录下，在那执行

 -f 选项将其它文件看作 Makefile,通过这种方法，make 命令会选择扫描 .mk 来代替 Makefile。

Vm_mux21.mk 也是生成出来的一个文件，在 obj_dir 文件夹里面，用于自动化的编译控制
最后一个参数是输出可执行文件的文件名，最好不要乱改，就"V" + “模块名”

### 5、运行

`./obj_dir/Vm_mux21`

运行后生成vcd文件

### 6、makefile文件

vim makefile编辑make文件

```makefile
all: cre_tar compile sim

cre_tar:
        verilator -Wno-fatal mux21.v main.c --top-module m_mux21 --cc --trace --exe
compile:
        make -C obj_dir -f Vm_mux21.mk

sim:
        ./obj_dir/Vm_mux21

clean:
        rm -r obj_dir *.vcd
```

在终端中输入make compile就等同于执行vcs test.v add.v -debug_all
在终端中输入make sim就等同于执行./simv -gui
在终端中输入make all就等同于先compile，再sim，编译执行一步到位
在终端中输入make clean 就等同于删除所有编译和执行时产生的文件，只保留了add.v,test.v以及makefile文件

# 接入NVBoard

## 拷贝NVBoard源码

1. 将项目拷贝到本地，

   ````bash
   git clone https://github.com/NJU-ProjectN/nvboard.git
   ````

2. 通过

   ````bash
   sudo apt-get install libsdl2-dev libsdl2-image-dev`
   ````

   安装SDL2和SDL2-image

3. 把本项目的目录设置成环境变量`NVBOARD_HOME`

```bash
vim /etc/profile
```

在文件末尾加

```bash
export PATH=/home/yihan/nvboard/include:$NVBOARD_HOME
```

使用`source /etc/profile`更新环境变量

## 接入verilator步骤

### API说明

NVBoard提供了以下几组API

- ```c
  void nvboard_init()
  ```

  初始化NVBoard

- ```c
  void nvboard_quit()
  ```

   退出NVBoard

- ```c
  void nvboard_bind_pin(void *signal, bool is_rt, bool is_output, int len, ...)
  ```

  将HDL的信号signal连接到NVBoard里的引脚上，具体地

  - `is_rt`为`true`时，表示该信号为实时信号，每个周期都要更新才能正确工作，如键盘和VGA相关信号； `is_rt`为`false`时，表示该信号为普通信号，可以在NVBoard更新画面时才更新，从而提升NVBoard的性能，如拨码开关和LED灯等，无需每个周期都更新
  - `is_output`为`true`时，表示该信号方向为输出方向(从RTL代码到NVBoard)；否则为输入方向(从NVBoard到RTL代码)
  - `len`为信号的长度，大于1时为向量信号
  - 可变参数列表`...`为引脚编号列表，编号为整数；绑定向量信号时，引脚编号列表从MSB到LSB排列

- ```c
  void nvboard_update()
  ```

  更新NVBoard中各组件的状态，每当电路状态发生改变时都需要调用该函数

## nvboard运行实例

