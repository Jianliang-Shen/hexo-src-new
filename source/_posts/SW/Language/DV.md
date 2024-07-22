---
layout: poster
title: 芯片验证
date: 2021-01-24 12:50:55
tags:
    - SystemVerilog
categories: 
    - Language
---

验证就好比测试，不知道将来还需要多少人才。

<!-- more -->


System Verilog与芯片验证
-----

- [芯片验证基础](#芯片验证基础)
  - [验证内容](#验证内容)
  - [验证周期](#验证周期)
- [Systemverilog基础](#systemverilog基础)
  - [数据类型](#数据类型)
    - [logic](#logic)
    - [二值逻辑](#二值逻辑)
    - [数据类型](#数据类型-1)
    - [自定义类型](#自定义类型)
    - [枚举类型](#枚举类型)
    - [结构体类型](#结构体类型)
    - [字符串类型](#字符串类型)
    - [类型转换](#类型转换)
  - [数组](#数组)
    - [非组合型数组](#非组合型数组)
    - [组合型数组](#组合型数组)
    - [数组拷贝](#数组拷贝)
    - [foreach遍历](#foreach遍历)
    - [数组的系统函数](#数组的系统函数)
    - [动态数组](#动态数组)
    - [队列](#队列)
    - [关联数组](#关联数组)
    - [数组的高级操作](#数组的高级操作)
  - [SV的设计特性](#sv的设计特性)
    - [过程语句块](#过程语句块)
      - [always\_comb](#always_comb)
      - [always\_latch](#always_latch)
      - [always\_ff](#always_ff)
    - [赋值操作](#赋值操作)
    - [增强的case语句](#增强的case语句)
      - [unique case](#unique-case)
      - [priority case](#priority-case)
    - [接口](#接口)
      - [接口的引入](#接口的引入)
      - [接口的声明](#接口的声明)
      - [接口的例化](#接口的例化)
      - [modport](#modport)
      - [总结](#总结)
    - [QuestaSim使用](#questasim使用)
  - [验证环境结构](#验证环境结构)
    - [验证环境组件](#验证环境组件)
      - [激励发生器](#激励发生器)
      - [检测器](#检测器)
      - [比较器](#比较器)
  - [类](#类)
    - [类的定义和实例化](#类的定义和实例化)
    - [静态成员](#静态成员)
    - [this关键字](#this关键字)
    - [类的赋值和拷贝](#类的赋值和拷贝)
    - [类的数据隐藏和封装](#类的数据隐藏和封装)
    - [类的继承](#类的继承)
    - [成员覆盖](#成员覆盖)
    - [包的使用](#包的使用)
  - [环境运行](#环境运行)
    - [随机约束](#随机约束)
      - [随机](#随机)
      - [约束](#约束)
      - [约束块](#约束块)
      - [权重分布](#权重分布)
      - [条件约束](#条件约束)
      - [迭代约束](#迭代约束)
      - [软约束](#软约束)
      - [随机控制](#随机控制)
    - [任务和函数](#任务和函数)
      - [任务task](#任务task)
      - [函数function](#函数function)
    - [线程控制](#线程控制)
    - [进程间同步和通信](#进程间同步和通信)
      - [事件event](#事件event)
      - [旗语semaphore](#旗语semaphore)
      - [信箱mailbox](#信箱mailbox)
    - [虚方法--多态](#虚方法--多态)
  - [验证平台实现](#验证平台实现)
    - [MCDT测试平台结构](#mcdt测试平台结构)
    - [channel stimulator](#channel-stimulator)
      - [数据包类](#数据包类)
      - [initiator](#initiator)
      - [generator](#generator)
    - [channel monitor](#channel-monitor)
    - [mcdt monitor](#mcdt-monitor)
    - [mcdt checker](#mcdt-checker)
    - [实现顶层环境](#实现顶层环境)
      - [agent](#agent)
      - [mcdt\_root\_test](#mcdt_root_test)
    - [实现测试用例](#实现测试用例)
      - [基础测试](#基础测试)
  - [验证量化管理](#验证量化管理)
    - [概述](#概述)
    - [覆盖率种类](#覆盖率种类)
    - [代码覆盖率](#代码覆盖率)
      - [行覆盖率statement/line](#行覆盖率statementline)
      - [分支覆盖率branch](#分支覆盖率branch)
      - [条件覆盖率condition/expression](#条件覆盖率conditionexpression)
      - [状态机覆盖率FSM](#状态机覆盖率fsm)
      - [跳转覆盖率toggle](#跳转覆盖率toggle)
      - [100%代码覆盖率](#100代码覆盖率)
    - [功能覆盖率](#功能覆盖率)
      - [覆盖组covergroup](#覆盖组covergroup)
      - [覆盖点coverpoint](#覆盖点coverpoint)
      - [仓(bin)](#仓bin)
      - [视频51/52未记录](#视频5152未记录)
- [UVM核心特性](#uvm核心特性)
  - [UVM总览](#uvm总览)
    - [工厂机制的引入](#工厂机制的引入)
    - [阶段pahse的引入](#阶段pahse的引入)
    - [消息管理机制](#消息管理机制)
    - [固定资产和动态资源的划分](#固定资产和动态资源的划分)
  - [基础结构](#基础结构)
    - [核心基类uvm\_project](#核心基类uvm_project)
      - [定义](#定义)




# 芯片验证基础
芯片验证平台包括：
* 激励simulator
* 监测器monitor
* 比较器checker
* 复位/时钟

芯片验证应当与RTL设计分开独立。芯片开发流程包括：用户需求，设计结构和产品描述，系统设计，模块功能详述，硬件设计，HDL语言开发，功能验证，验证环境文件，后端综合，芯片产品。设计与验证都围绕功能描述文档。

> 芯片开发流程包括下面哪几项 ABCDE
> A 功能验证
> B 系统设计
> C 用户需求
> D 硬件设计
> E 后端综合

## 验证内容
* 有没有按照功能描述文档执行
* 有没有多余设计
* 有没有处理边界 corner case
* 有没有处理一些情况如error response

模块验证 → 子系统验证 → 系统级验证，按时（milestone关键时间点一个不能少）、保质（保证在硅前验证出来）、低耗（尽量少的人力时间成本）。

> 验证人员如果发现了设计问题，应该首先 B
> A 验证人员修改设计代码
> B 将问题反馈给设计人员
> C 等待设计人员重新提交设计

> 下列哪些事务是验证人员的职责范围 BC
> A 将功能描述文档翻译为RTL文件
> B 阅读功能描述文档
> C 发送激励并检查结果
> D 将功能需求翻译为功能描述文档

量化芯片验证的指标：时间和缺陷类型
## 验证周期
* 创建计划
* 开发验证环境
* 调试环境和HDL文件
* 递归测试（完备性检查）
* 硅后系统测试
* 展开逃逸分析
* 吸取教训



> 从成本角度考虑，设计漏洞在下面哪个阶段被发现所带来的成本是最小的 B
> A 门级验证
> B RTL阶段
> C 硅后测试
> D 用户测试

> 就设计缺陷而言，验证人员应该做出下列哪些行为 ABD
> A 在验证后期阶段不应该再出现基本设计缺陷
> B 使得缺陷增长曲线逐步收敛
> C 采取先复杂再简单的测试序列（解析：反了）
> D 需要绘制出缺陷增长曲线

# Systemverilog基础
SV在Verilog设计语言特性基础上扩展了验证语言特征：
* 接口
* 面向对象
* 约束随机
* 线程控制通信
* 功能覆盖率
* 外部语言编程接口
* 断言

## 数据类型
Verilog包括``variable``变量类型和``nets``线网型类型。均为四值逻辑，分为``0``，``1``，``X``（未知，0/1），``Z``（无驱动，高阻态）。例如``reg``，``integer``，``time``，``wire``，``wor``，``wand``等类型。线网的值由驱动器模拟，驱动器包括门，模块以连续赋值。变量在``initial``，``always``，``task``，``function``内赋值。``integer``可以是有符号的，而``reg``，``time``是无符号的。一般reg描述逻辑，``integer``描述设计，real在系统模型中使用，``time/realtime``在仿真中使用。reg用来存储数据但不一定是``register``，也可以是锁存器。

SV增加了``logic``四值逻辑给设计工程师使用，增加了二值逻辑给验证工程师使用。在V的基础上，V的类型包括变量和线网，SV增加了数据类型，即``logic``和``bit``之类。

### logic
```sv
logic           resetN;         //1 bit wide 四值变量
logic [63:0]    data;           //64 bit wide
logic [0:7]     array [0:255];  //8 bit 数组，共256个元素
```
可以在前面加前缀``reg``，``wire``等，默认为``var``。``logic``同样不能多驱动。
### 二值逻辑
| 类型     | 宽（bit） | 等效于C  |
| -------- | --------- | -------- |
| bit      | 1         | integer  |
| byte     | 8         | char     |
| shortint | 16        | short    |
| int      | 32        | int      |
| longint  | 64        | longlong |
### 数据类型
* 有符号：``integer``（有符号四值逻辑），``byte``，``shortint``，``int``，``longint``
* 无符号：``logic``，``bit``，以及构成的``vector``是无符号的

> 下列数据类型哪些是有符号的类型 ABCD
> A int
> B integer（32位四值逻辑，其余为二值逻辑）
> C byte
> D shortint

转换：
```sv
byte        //[-128, 127]
bit [7:0]   //[0, 255]，相当于byte unsigned
```
四值逻辑``reg``，``logic``，``integer``在仿真开始值为``X``，二值逻辑``bit``初始值为``0``。四值逻辑转换为二值逻辑时，``X/Z``转换为``0``。

``void``同C，表示空。SV：``shortreal``同``C float``，32位单精度。V：``real``双精度，同C的``double``。

### 自定义类型
约定后缀``_t``为自定义类型。
```sv
typedef int unsigned uint_t;
uint_t a, b;
typedef bit [63:0] uint_t
```


> ```sv
> typedef enum {WAITE, LOAD, READY} states_t;
> int sum = WAITE + LOAD + READY;
> ```
> sum的结果是 D
> A 0
> B 1
> C 2
> D 3

### 枚举类型
```sv
enum {RED, GREEN, BLUE} RGB; //RGB是一个枚举类型的变量，enum {RED, GREEN, BLUE}整体是匿名的

typedef enum {RED, GREEN, BLUE} RGB; //RGB是一个自定义的枚举类型
RGB rgb_u; 
```
类似于verilog中的`` `define``和``parameter``。枚举类型默认为32位``int``类型，从0开始，也可以声明具体的类型，并在声明时初始化。
```sv
enum bit {TRUE = 1, FALSE = 0} Boolean;
enum logic [1:0] {WAITE, LOAD, READY} state;
enum logic [2:0] {WAITE = 3'b001,
                  LOAD   = 3'b010,
                  READY  = 3'b100} state;
```
枚举类型的赋值比较严格，左边为枚举类型，右边若为``int``类型，则需要显式的转换：
```sv
state_t state;
int a;
state = state_t'(state + a);
```
### 结构体类型
```sv
typedef struct {
    int a, b;
    opcode_t opcode;
    logic [23:0] address;
    bit error;
}Instruction_Word;
Instruction_Word IW;
IW.address = 24'b0;
IW = '{100, 3, 8'hff, 0};
IW = '{address: 0, opcode: 8'hff, a:100};
```
SV的结构体也有“对齐”的说法，成为组合型和非组合型。即存储的连续性和非连续性。

### 字符串类型
``string``关键字，每个单元为``byte``，长N，索引为0~N-1，结尾没有``'\0'``，内存动态分配。string可以通过``{,}``拼接，但不可以使用加法。``string``的方法：
```sv
str.len()
str.putc(idx, c) == str[idx] = c
str.getc(idex)
str.atoi() str.atohex() str.itoa() str.hextoa()
$display("%s", s)
$sformatf(s, "%s", name)
```

### 类型转换
静态转换操作符不对转换值进行检查。转换时指定目标类型，并在要转化的表达式前加上单引号。Verilog自带隐式的转化。
```sv
int i;
real r;
i = int'(10.0 - 0.1);
r = real'(42);
```
动态转换：我们总是可以将子类的句柄赋值给父类的句柄，但是我们将父类的句柄赋值给子类的句柄时将编译报错。``$cast()``系统函数可以将父类句柄转换为子类句柄，只要该父类句柄指向的是一个子类的对象。见[类的继承](#类的继承)
```sv
function int $cast(singular dest_var, singular source_exp);
                        目标类型           待转化类型
```

## 数组
### 非组合型数组
非组合型数组``unpacked``，数组中成员之间（高维）存储数据都相互独立。
```sv
//verilog
reg [15:0] RAM [0:4095] //每个元素为16bit数据的长为4096的数组，16bit是连续的，4096不连续
wire [7:0] table [3:0]

//SV
reg [7:0] arr [1023:0];         //arr[1024][8]
int arr [7:0][1023:0];          //arr[8][1024]
logic [31:0] data [1023:0];
logic [31:0] data [1024];       //都是合法的
```
### 组合型数组
定义在左侧，连续存储，规范存储方式。初始化可以展开为一维向量的方式。组合型也适用于结构体。
```sv
logic [3:0][7:0] arr = 32'h0;       //arr[4][8]
                                    //存储次序： arr[3][7] arr[3][6] ... arr[0][1] arr[0][0]
logic [31:0] data = arr;            //大小一致，合法
logic [3:0][7:0] c = {16{2'b01}};   //大小一致，合法
```
非组合型数组赋初值可以逐一赋值也可以``default``关键字：
```sv
int d [0:1][0:3] = '{'{1,2,3,4}, '{5,6,7,8}};
int a [0:1023][0:7] = '{default: 8'h55}
```

> ``int arr[7:3][3:0]`` 和 ``integer arr2[8][4]``各占据多少WORD空间？B
> A 32，32
> B 32，64
> C 64，64
> D 64，32
> 解析：WORD占32位，int二值逻辑32位；integer四值逻辑32位，2bit存储一个logic

> ``logic [3:0][7:0] data;``变量data占据几个word空间 B（logic四值逻辑占2bit）
> A 1
> B 2
> C 4
> D 8

> ``bit word [3:0][7:0];``对数组的赋值下列哪些是正确的 C
> A ``word = '{'h11, 'h22, 'h33, 'h44};``
> B ``word = 'h11223344;``
> C ``word[0][0] = 1; word [0][4] = 1;``
>   ``word[1][1] = 1; word [1][5] = 1;``
>   ``...``
> D ``word = {'h11, 'h22, 'h33, 'h44};``
> 解析：A, ``word[0]`` 相当于``bit arr [7:0]``，仍然是非组合的，A适合给``bit [7:0] word [3:0]``赋值；数组是非组合的，D是对组合数组赋值（``{,}``直接拼接，前面不加``'``）

### 数组拷贝
组合型视为向量赋值，在尺寸不一样时会发生截断。非组合型数组拷贝需要大小维度严格一致。非组合型数组与组合型数组之间不能直接拷贝。

### foreach遍历
foreach无需考虑数组的边界：
```sv
foreach( sum[i,j])
    sum[i][j] = i + j;
```
### 数组的系统函数
```sv
$dimensions(arr)                   //返回维度
$left(arr, dimension)              //返回指定维度的最左索引值。
logic [1:2][7:0] word [0:3][4:1];  //还有right， low， high
           最低维     最高维
$left(word, 1) = 0                 //1表示最高维
$left(word, 2) = 4
$left(word, 3) = 1
$left(word, 4) = 7
$size(arr, dimension)
$increment(array, dimension)       //如果左>=右，返回1
$bits()                            //返回比特数
```

### 动态数组
声明时使用``[]``，初始化使用``new[N]``。
```sv
int dyn[], d2[];
dyn = new[5];
d2 = dyn;
dyn = new[20](dyn);     //allocate 20 ints and copy
dyn = new[100];         //old data lost
dyn.delete();           //删除
```

### 队列
声明时使用``[$]``，$为最后一个索引。方法有``push_back``, ``push_front``, ``pop_back``, ``pop_front``, ``delete``。

### 关联数组
适用于大数组却只访问局部片段的情况，类似高级语言``hash``，字典等。foreach同样可以遍历关联数组。关联数组的方法，``first``，``next``，``num``，``exits``。关联数组的索引值可以是字符串等非整数类型。


> 对于定长数组，动态数组，队列及关联数组，谁具有内建方法size()？BCD
> A 定长数组
> B 动态数组
> C 队列
> D 关联数组

### 数组的高级操作
* 缩减：``sum``，``product``（求积），``and``，``or``，``xor``
* 排序：``reverse``，``sort``（正序），``rsort``，``shuffle``（打乱）
* 定位：``min``（返回队列），``max``，``unique``，``find with``，``find_first with``，``find_index with``，``find_first_index with``。

## SV的设计特性
* 添加接口``interface``
* 添加自定义类型，枚举类型和结构体
* 添加类C的数据类型
* 添加类型转换``$cast()``或``type'()``
* 添加包和类
* 添加``++``，``+=``，``===``
* 添加``priority``和``unique case``
* 添加``always_comb``，``always_latch``以及``always_ff``

> 下列always语句块哪些可以在SV中编译 ABC
> A always
> B always_comb
> C always_latch
> D always_reg

### 过程语句块

SV添加了新的面向硬件的过程语句块，从而使得该语句块可以更清楚地表达设计者的意图。always语句块被细分为了：
* 组合逻辑语句块：``always_comb``
* 锁存逻辑语句块：``always_latch``
* 时序逻辑语句块：``always_ff``

#### always_comb
``always_comb``可以自动嵌入敏感列表。可以禁止共享变量，即赋值左侧的变量无法被另—个过程块所赋值。软件工具会检查该过程块，如果其所表示的不是组合逻辑，那么会发出警告。在仿真的0时刻也会自动触发一次，无论0时刻是否有敏感信号列表中的信号变化。

verilog的``@*``敏感列表声明方式不同``always_comb``。 ``@*``：

* ``@*``不要求可综合的建模要求，但``always_comb``则会限制其他过程块对同一变量进行赋值。
* ``@*``的敏感列表可能不完全，例如如果一个过程块调用一个函数，那么``@*``则只会将该函数的形式参数自动声明到敏感列表，而不将该函数展开。
* ``always_comb``则将被调用函数中可能参与运算的其它信号也声明到敏感列表中。

> 关于always@*和always_conb的区别，下列说法正确的是 B
> A always_comb中被赋值的变量，不能在其它过程块中被赋值（解析：两者都不允许其他过程块赋值）
> B always_comb的敏感列表要比always@*更完全（解析：函数）
> C always@*与always_comb在0时刻都会被自动触发执行
> D always@*与always_comb都表示组合电路（解析：前者不止组合电路）

#### always_latch
表示锁存逻辑，且自动插入敏感列表。并且EDA工具会检查``always_ latch``过程块是否真正实现了锁存逻辑。
```sv
always_comb()
    if(en)
        q = a;
    else
        q = q;   //如果缺了这个分支则为锁存逻辑

always_latch()
    if(en)
        q <= a;
```

#### always_ff
表示时序逻辑，敏感列表必须指明``posedge``或者``negedge``，这是综合要求的。使得EDA工具实现同步或者异步的复位逻辑。EDA工具也会检查``always_ff``过程块语句是否实现了时序逻辑。


### 赋值操作
Verilog没有简单的方法可以对向量填充``1``。SV可以通过``'0``，``'1``，``'z``和``'×``来分别填充0，1，z和x。通过这种方法，代码会根据向量的宽度自动填充，这提高了代码的便捷性和复用性。
```sv
parameter N=64;
reg [N-l:0] data_bus;
data_bus = 64' hFFFFFFFFFFFFFFF; //set all bits of data_bus to 1
SV:
data_bus = '0;
```

sv在比较数据中，对于全匹配，通常使用``===``；可以使用``==?``进行通配比较。在比较操作符的**右侧**操作数，如果在某些位置有X或者Z，那么它表示的是在该位置上会与左侧操作数的相同位置的任何值相匹配（如果左边有XZ右边没有返回``unknown``）。``==``比较不能识别X/Z，否则返回``unknown``，如果这时候的判断是if的条件判断，返回``unknown``则转换为0（false）

| a    | b    | a==b    | a===b | a==?b   | a!=b    | a!==b     | a!=?b   |
| ---- | ---- | ------- | ----- | ------- | ------- | --------- | ------- |
| 0000 | 0000 | true    | true  | true    | false   | false     | false   |
| 0000 | 0101 | false   | false | false   | true    | true      | true    |
| 010Z | 0101 | unknown | false | unknown | unknown | **false** | unknown |
| 010Z | 010Z | unknown | true  | true    | unknown | false     | false   |
| 010X | 010Z | unknown | false | true    | unknown | true      | false   |
| 010X | 010X | unknown | true  | true    | unknown | false     | false   |

加粗的为``false``，因为``a=0100``或``0101``，``a!==b``即``!(0100 === 0101 || 0101 === 0101)``，返回``false``。

SV中添加了``inside``操作符来检查数值是否在一系列值的集合中。下面的if语句中的多个条件判断可以变得更为简化:
```sv
logic [2: 0] a ;
if(( a==3'b001) || (a==3'b010) || (a==3'b100))...
替换为
if(a inside {3'b001, 3'b010, 3'bl00} )...
```

### 增强的case语句
仿真和综合可以会将case语句做不同的翻译。Verilog定义case语句在执行时按照优先级，而综合编译器则会优化case语句中多余的逻辑。为了保持仿真与综合的一致性，SV提供了unique和priority的声明，结合case, casex和casez来进一步实现case对应的硬件电路。

unique和priority的声明也可以结合if…else条件语句使用。unique和priority使得仿真行为同设计者希望实现的综合电路保持一致。

#### unique case
unique case 要求每次case选择必须**只能满足一个case项**，同时**不能有重复的选项**，既满足多个选项的条件。unique case 可以并行执行，并且case选项必须完备。

#### priority case
priority case表示至少有一个case选项满足要求。如果有多个case选项满足时，第一个满足的分支将被执行。 priority case的逻辑同if…else的逻辑一致。

### 接口
SV在veriliog语言的基础上拓展了接口。接口提供了一种新型的对抽象性建模的方式。接口 的使用简化了建模和验证大型复杂的设计。verilog上通过模块之间进行端口连接来完成模块间的通信的。对于复杂的连接，按照Verilog的方式，将按照以下步骤进行：
* 对每一个子模块进行端口声明。
* 在上层环境，需要声明非常多的网线。
* 将上层环境的网线在各个模块之间进行连接。

这种方式使得一些常用总线端口也不得不在多个模块重复声明。—些通信协议也不得不在多个模块中重复定义。在不同模块之间的连接可能会出现不匹配的信号声明和连接。—个设计发生了变化，可能会影响多个模块的端口声明和连接。
#### 接口的引入
SV添加了新的抽象端口类型interface。interface允许多个信号被整合到一起用来表示一个单一的抽象端口。多个模块因此可以使用同一个interface，继而避免分散的多个端口信号连接。

接口不单单可以包含变量或者线网，它还可以封装模块之间通信的协议。接口中还可以嵌入与协议有关的断言检查、功能覆盖率收集等模块。接口不同于模块(module) 的地方在于，接口不允许包含设计层次，即接口无法例化module，但是接口可以例化接口，module中也可以例化接口。接口中可以进一步步明modport来约束不同模块连接时的信号方向。

#### 接口的声明
接口的定义类似模块定义。接口可以有端口，例如外部接入的时钟或者复位信号。接口内部可以声明所有的变量或者线网类型。

#### 接口的例化
接口的例化方式同模块例化，模块的端口如果声明为input，output或者inout，那么在例化时可以不连接。模块端口如果声明为interface，那么在例化时必须连接到一个接口实例，或者另外一个接口端口。
如果一个模块拥有一个接口类型端口，那么要索引该接口中的信号，需要通过以下方式进行:
```sv
<port _name>.<internal_interface_signal_name>
always @(posedge bus.clock, negedge bus.resetN)
```
#### modport
接口中的变量或者线网信号，对于连接到该接口的不同模块则可能具备这不同的连接方向。所以接口引入了modport来作为module port的缩写，表示不同的模块看到同一组信号时的视角(连接方向)。在接口中声明modport，需要指明modport中各个信号的方向。

接口中的变量或者线网信号，对于连接到该接口的不同模块则可能具备着不同的连接方向。接口引入了modport来作为module port的缩写,表示不同的模块看到同一组信号时的视角(连接方向)。在接口中声明modport,需要指明modpot中各个信号的方向。

当一个模块在例化时，可以选择连接到interface端口中具体的某一个modport。这种方式可以降低方向连接错误的可能，进而避免信号多驱动的情况。

#### 总结
接口对于设计复用非常有利。接口减少了模块之间错误连接的可能性。如果要添加新的信号，只需要在接口中声明，而不必在模块中声明。由于接口将有关信号都集合在一起，因此在使用这些信号时需要多添加一个层次(接口实例名)。接口往往会将有关的信号集合在一起，这意味着对于拥有多组不相关信号的设计而言，它可能需要有多个接口实例才能完成与其它模块的连接。


### QuestaSim使用
* 新建project
* 添加设计和仿真文件
* 添加设计文件编译顺序并编译all
* 在work中找到仿真文件模块并仿真
* 仿真run
* 看波形，放大，边沿跳转，看数据
* log -r /*预先将所有信号记录



## 验证环境结构
验证平台是整个验证系统的总称。它包括验证结构中的各个组件，组件之间的连接关系，测试平台的配置和控制。更系统来讲，它还包括编译仿真的流程、结果分析报告和覆盖率检查等。狭义上说，主要关注验证平台的结构和组件部分，他们可以产生设计所需要的各种输入，也会在此基础上进行设计功能的检查。

各个组件之间是相互独立的，验证组件与设计之间需要连接，验证组件之间也需要通信，验证环境也需要时钟和复位信号驱动。

> 下面对测试平台的描述哪些是正确的 ABC
> A 测试平台中的各个组件既保持独立又相互连接
> B 测试平台中的组件之间发生通信
> C 广义而言测试平台还包括编译流程、结果分析，覆盖率收集等
> D 测试平台不同于设计，它不需要时钟和复位信号

### 验证环境组件
#### 激励发生器

stimulator 激励发生器：是验证环境中的重要部件，有些时候也称为driver（驱动器），bfm（总线功能模型），behavioral（行为模型）或者generator（发生器）。

* 主要职责是模拟和DUT相邻设计的接口协议。
* 需要关注如何模拟接口信号使得可以以真实的接口协议来发送激励给DUT，并且stimulator不应该违反协议。
* stimulator的接口主要是同DUT之间的连接，也可以有其他的配置接口用来控制接口的数据生成。
* 也可以有存储接口数据生成历史的功能。

从stimulator同DUT的连接关系来说，可以进一步分为initiator（发起器） 和responder（ 响应器）。
* 发起器：主动发起接口数据传输，考虑协议和空闲周期，传输速率
* 响应器：对接口的数据发送请求作响应

> 对于stimulator下面哪些描述是正确的 CD
> A stimulator是为了向DUT内部信号发送激励（向端口而非内部）
> B stimulator按照同DUT的连接关系，可以分为initiator和driver
> C stimulator不应该违反接口协议去发送激励信号
> D stimulator可以存储生成过的属于用于后期数据比较调试

#### 检测器
monitor（监测器）的主要功能是用来观察DUT的边界或者**内部信号**，并且经过打包整理传送给其他验证平台的组件，例如checker（比较器）。从检测信号的层次来划分monitor的功能，它们可以分为观察DUT边界信号和观察DUT内部信号。

* 观察DUT边界信号 ：对于系统信号如时钟，可以检测其频率变化，对于总线信号，可以检测总线的传输类型和数据内容，以及检查总线时许是否符合协议。
* 观察DUT内部信号 ：从灰盒验证的手段来看，往往需要窥探DUT内部信号，用来指导stimulator的激励发送，或者完成覆盖率收集，又或者完成内部功能的检查。

没有特别需要，应该采用灰盒验证的策略。观察内部信号应当尽量少，应当是表示状态的信号。不建议采集中间变量信号的原因在于，这些信号的时序、逻辑甚至留存性都不稳定，这种不稳定对于验证环境的收敛是有害的。可以通过接口信息计算的，就尽量少去监测内部信号，因为这种方式也有悖于假定设计有缺陷的验证思想。观测到的内部信号在被环境采纳之前也有必要确认它们的逻辑正确性，这一要求可以通过动态检查或者断言触发的方式来实现。

>下面对Monitor描述哪些是正确的 AB
> A Monitor可以从DUT的外部接口观测数据
> B Monitor可以从DUT的内部接口观测数据
> C Monitor可以从Simulator获取激励数据（不可以获取，Monitor看DUT输入端口最准确）
> D Monitor可以将观测到的输入输出数据进行比较，检查DUT功能

#### 比较器
checker 是最需要时间投入的验证组件。checker肩负来模拟设计行为和功能验证检查的任务，缓存从各个monitor收集到数据，将DUT输入接口测的数据汇集给内置的reference model（参考模型）。reference model模拟了硬件功能。通过数据比较的方法，检查世纪收集到的DUT输出端口的数据是否与reference model产生的期望数据一致。

比较方式：对于设计内部关键功能模块，也有相应的线程进行独立的检查。
* 线上比较（online check） ：在仿真时收集数据和在线比较，并且实时报告。
* 线下比较（offline check） ：将仿真时收集到的数据记录在文件中，仿真结束后通过脚本或者其他手段进行比较。

预测：将checker添加到验证环境中，需要它分析DUT的边界激励，理解数据的输入，并且按照硬件功能来预测输出的数据内容。这种预测(prediction)的过程，发生在reference model中,有时我们也将其称为predictor。

reference model也会内置一些缓存，分别存放从DUT输入端观察到的数据，以及经过功能转换的数据，同时checker也有其它缓存来存放从输出端采集到的数据。

比较器实现建议：
* 对于复杂的系统验证，倾向于集中管理stimulator和checker，因为它们两者都需要主动给出激励或者判断结果，也需要较多的协调处理。
* 而monitor则相对更独立，因为它只是作为监测方，将其观察到的数据—交给checker即可，monitor并不需要关心checker的用法。
* 因此,stimulator和monitor是——对应，我们通常将它们进—步封装在agent单元组件中，而checker则最终集群搁置在中心化的位置。

> 下面对checker的描述哪些是正确的 ABCD
> A checker的主要任务是检查设计功能
> B checker的数据比较方式可以分为线上比较和线下比较
> C 通过输入数据来预测输出数据时，需要reference model来模拟设计行为
> D reference model和checker都可以内置缓存用来存放数据

## 类

### 类的定义和实例化
类是包含数据和方法的类型，类中可以包含指令，地址，队列ID，时间戳和数据。

packet类：

```sv
class Packet;

    bit [3:0] command;
    bit [40:0] address;
    ...
    integer time_requested;
    typedef enum {ERR_OVERFLOW = 10, ERR_UNDERFLOW = 1123} PCKT_TYPE;
    const integer buffer_size;

    function new();  //任何一个类都得有new的初始化方法
        command = 4'd0;
        address = 41'b0;
        ...
    endfunction

    //用户自定义的部分
    task clean();
        command = 0;
        address = 0;
        ...
    endtask

    function integer current status();
        current_status = status;
    endfunction

endclass
```
类例化对象，正如module的例化。面向对象编程OOP，使用户能够创建复杂的数据类型，并与程序紧密结合。用户可以在更加抽象的层次建立测试平台和系统级模型，通过调用函数来执行一个动作而不是简单地改变信号的电平。在验证环境中，包括simulator，monitor，checker以及其他组件都按照OOP的方式构建。面向对象编程包括**类class**，**对象object**，**句柄handle**（指向对象的指针，SV没有指针指向整型和字符串），**原型prototype**（程序的声明部分，包含程序名、返回类型和参数列表）

SV的类和C++的区别：
* 不需要复杂的存储空间开辟和销毁，自动的
* 只需要定义构建函数（如new），不需要定义析构函数

**类定义时可以不定义构建函数，系统会自动定义一个空的。**对象创建时需要先声明再例化（new()），也可以同时完成，如Packet p = new(); p是指向一个对象的句柄，而非实例。**new函数不需要添加返回值类型**（真实返回的时Packet类型的句柄），在构建时不需要声明void。类允许定义时变量定义时声明默认值，但new中初始化，则以初始化值为准。

类的成员，变量和方法都是动态的，在实例化的时候开始生命周期（开辟空间）。C++有析构函数，但SV不需要回收。SV在程序中没有句柄指向实例时，其生命周期结束。

### 静态成员
静态成员：多个对象共享同一个成员（变量/方法），那么可以为其添加关键词**static**。
* 多个对象可以共享这一个成员变量或者方法（也是唯一的一个）
* 访问该成员时无需进行对象的例化。

```sv
clase Packet；
    static integer a;
    ...
endclass

Packet p;
c = p.a;    //这里p不需要new()实例化即可访问静态变量a，
            //实际上a并不是存储在实例中，而是在其他段（全局的），
            //使得本代码中所有即将实例化的对象或声明的句柄都可以访问到。
c = Packet :: a;  //另一种写法，域的索引也可以索引类型如
Packet :: PCKT_TYPE enum_test;
```

成员方法也可以是静态的，静态方法无法访问非静态成员的变量或者方法，否则会编译报错。

> 下列对象哪些一定是动态的 A
> A 由类创建的对象
> B 例化的模块（伴随始终，静态的）
> C 类的成员方法
> D 类的成员变量（static则为静态的）

### this关键字

this用来明确索引当前所在对象的成员，只能用来在类的非静态成员、约束和覆盖组中使用。this的使用可以明确所指向变量的作用域。
```sv
class Demo;
    integer x;
    function new(integer x);
        this.x = x; //右边的x是参数的x
    endfunction
endclass
```

### 类的赋值和拷贝
```sv
Packet p1, p2;
p1 = new();
p2 = p1; //p1,p2都指向同一个实例
```
浅拷贝shallow copy
```sv
Packet p1, p2;
p1 = new();
p2 = new p1; //p2拷贝了p1的所有成员，p1,p2指向两个实例
```

* 变量的声明是声明的句柄（指针），默认不指向任何实例。
* 如果要想使null的句柄指向实例，可以通过new来实例化。
* 如果两个两个句柄指向同一个对象，则一个改变另一个也改变。
* 拷贝对象使用``new + handle``的形式。

注意浅拷贝只针对对象中的成员变量，假设有其他类的句柄在其中，浅拷贝只会复制一个句柄，不会对其指向的对象再做拷贝。深拷贝（deep copy）则会递归拷贝。**深拷贝函数需要用户自定义实现。**
```sv
class rgb;
    byte red;
    byte green;
    byte blue;
endclass

class pixel;
    int x;
    int y;
    rgb color;

    function new();
        ...
        color = new();              //实例化 color句柄
    endfunction

    function pixel copy();          //深拷贝函数
        pixel t = new this;         //复制一个当前的实例（浅拷贝）
        t.color = new this.color;   //复制一个已经实例的color句柄
        return t;
    endfunction
endclass
```

### 类的数据隐藏和封装
公共的，自身和外部都可以访问该成员；隐藏，使得可以访问的接口更加精简。
* local，对成员限定，只有当前类可以访问此成员，子类和外部无法访问
* protected，对成员限定，表示该类和子类可以访问，外部无法访问。

```sv
class Packet;
    local integer i;                //p.i无法访问
    function integer get_i();       //p.get_i()可以访问到i
        return this.i;
    endfunction
endclass
```

### 类的继承
父类Packet，可以定义一个子类LinkedPacket，使用**extends**关键字。子类继承了父类的成员，所以LinkedPacket对象也是Packet对象，Packet句柄可以指向LinkedPacket对象。
```sv
class LinkedPacket extends Packet;
    ...
endclass

LinkedPacket lp = new();
Packet p = lp;   //合法且安全
```
* ``p``，``lp``都指向子类对象
* ``lp``可以访问子类+父类的成员
* 但是p只能访问父类的成员

**父类的句柄不可以拷贝给子类的句柄**。父类的句柄转化为子类的句柄的方式：
```sv
LinkedPacket lp = new();
Packet p = lp;
LinkedPacket lp2;
$cast(lp2, p); //p指向子类，但p不能访问子类的成员，此时可以转化类型，p可以访问子类成员

Packet p = new();
LinkedPacket lp2;
$cast(lp2, p); //p只指向父类，不能转化
```

> 关于父类和子类的说法下列哪些是正确的 ACD
> A 可以将子类句柄直接赋值给父类句柄
> B 可以将父类句柄直接赋值给子类句柄
> C 子类可以拥有与父类同名的成员方法
> D 子类可以拥有与父类同名的成员变量

### 成员覆盖
如果子类定义了与父类同名的成员，那么子类中对其同名成员的访问都指向子类，父类的成员将被隐藏。可以通过super访问被隐藏的父类成员。区别于this，this是当前类的成员，super是父类的成员，如果是同名的，则必须加super，如果不同名，则可以直接访问。

PS：没有this或者super修饰的变量的查找声明的顺序：由近及远
* 检查当前函数列表，如果是函数参数，则为函数参数
* 检查当前类的this成员
* 检查父类的成员
* 检查全局声明的变量
```sv
class Packet;
    integer i = 1;
    function integer get();
        get = i;
    endfunction
endclass

class LinkedPacket extends Packet;
    integer i = 2;
    function integer get();
        get = -i;
    endfunction
endclass

LinkedPacket lp = new();
Packet p = lp;

//p.i = 1
//lp.i = 2
//lp.super.i = 1
//p.get() = 1
//lp.get() = -2
```
### 包的使用
通过包package可以将并联的类和方法并入到同一个逻辑联系组当中，避免重名的库、类、函数等。
module/interface\class均可以访问包，可以通过域``::``访问或者import导入。
```sv
package definitions;
    parameter VERSION = "1.1";
    ....
    function xxx();
    endfunction

    class yyy;
    endclass
endpackage

definitions :: VERSION
module M;
    import definitions :: instructions_t;
    instructions_t inst;
endmodule

module M;
    import definitions :: *;  //所有类型，注意不是所有都引进来，而是接下来用到的类型才会引进来
    instructions_t inst;
endmodule
```

## 环境运行
### 随机约束
#### 随机
定向激励无法完成检查功能完整性，随机约束测试CRT（Constrained-Random Test），通过回归测试、替换随机种子提高测试覆盖率。
简单生成随机数：
```sv
randomize(addr, data, rd_wr);
$urandom() //生成32位无符号随机数
$urandom_range(maxval, minval = 0)
```

> 下列关于随机的说法哪些是正确的 ACD
> A 随机数可以由系统函数$urandom()获得
> B 可以对实数类型做随机化
> C 可以对动态数组做随机化
> D 随机化可以置定变量的随机数值范围

#### 约束
约束是为了让随机的输入符合协议、满足测试需求。需要一个类存放变量和约束，而类的成员变量可以声明位随机属性，用rand、randc表示。任何类中的整型bit/byte/int都可以声明为rand/randc，randc是循环随机，避免重复。定长数组、动态数组、关联数组和队列都可以声明为rand/randc，可以对动态数组和队列的长度加以约束。对于rand修饰符，表示在可生成范围内每个值的可能性时相同的，连续两次是可以重复的。randc则随机并遍历其可取值范围。
```sv
rand bit [7:0] len;
rand integer data[];
constraint db { data.size == lem;}
```
指向对象的句柄成员，也可以声明为rand（不是随机指向对象），不能声明为randc，随机时该句柄指向对象中的随机变量也会一并被随机。非组合型结构体可以声明为rand，其成员可以声明为rand/randc。
```sv
typedef struct{
    randc int addr = 1 + constant;
    int crc;
    rand byte data [] = {1,2,3,4};
}header;
rand header h1;
```
带有随机变量的简单类：
```sv
class Packet;
    rand bit [31:0] src, dst, data[8];
    randc bit [7:0] kind;

    constraint c {src > 10;
                  src < 15;}
    //constraint d {src inside {[11,14]};}
endclass

Packet p;
initial begin
    p = new();          //所有成员将随机化
    if(!p.randomize())  //判断是否随机化成功
        $finish;
    transmit(p);
end
```

> 如果上面的句柄p没有调用randomize函数，p.src = () A
> A 0
> B 介于[0: 2^32-1]
> C 介于[11:14]
> 32'hFFFFFFFF
#### 约束块
**约束块**：有用的激励不仅仅是随机值，还有相互关系。没有约束的随机变量包含许多无效和非法值，需要一个多个约束表达式来约束定义这些相互关系。约束块支持整型通过set操作符来设置他们的可取值范围。
```sv
rand integer x, y, z;
constraint c1 {x inside {3, 5, [9:15], [24, 32], [y: 2*y], z};}

rand integer a, b, c;
constraint c2  {a inside {b, c};}

integer fives[4] = '{5, 10, 15, 20}
rand integer v;
constraint c3 {v inside {fives};}
```

> ```sv
> class SimpleSum;
>       rand bit [7:0] x, y;
>       bit [7:0] z;
>       constraint c {z == x + y;}
> endclass
> ```
> 下面哪组值可能是SimpleSum对象随机化的值？D
> A x=1, y=1, z=2
> B x=1, y=0, z=1
> C x=0, y=1, z=1
> D x=0, y=0, z=0（z不能随机化，默认值为0）

#### 权重分布
**权重分布**：除了成员集合设置，约束块也支持设置可取值的同时也为其设置随机时的权重。对于``:=``操作符，表示每一个值的权重是相同的。对于``:/``，表示权重会平均分配到每一个值。
```sv
x dist {[100:102] := 1, 2-- := 2, 300 := 5} //p(x=100) = 1/(1+1+1+2+5)
x dist {[100:102] /= 1, 2-- := 2, 300 := 5} //p(x=100) = (1/3)/(1+2+5)
```

**unique**可以约束一组变量，使得其在随机后变量之间不会有相同的数值。
```sv
rand byte a[5];
rand byte b;
rand byte excluded;
constraint u {unique {b, a[2:3], excluded};} //这三个变量不会重复
```

#### 条件约束
**条件约束**：使用**if else**或``->``操作符表示条件约束。
```sv
mode == little -> len < 10;
mode == big -> len > 100;

rand bit [3:0] a, b;
constraint c {(a==0) -> (b==1);}

if(mode == littele)
    len < 10;
else if(mode == big)
    len > 100;
```

#### 迭代约束
**foreach**可以**迭代约束**数组中的元素。也可以使用数组缩减的方法做迭代约束。
```sv
class C;
    rand byte A[];
    constraint c1 {foreach (A[i]) A[i] inside {2, 4, 8 ,16};}
    constraint c2 {foreach (A[j]) A[j] > 2 * j;}
    constraint c3 {A.size() == 5;}
    constraint c4 {A.sum() < 100;}
endclass
```
函数调用：
```sv
function int count_ones(bit[9:0] w);
    for(count_ones = 0; w!=0 w = w >> 1)
        count_ones += w & 1'b1;
    return count_ones;
endfunction
rand bit [9:0] v;
rand integer length;
constraint c {length == count_ones(v);}
```
#### 软约束
* 使用soft关键字
* 指定变量的默认值和权重
* 对一个变量有多次约束，如果有冲突，则二次约束的硬约束可以覆盖掉软约束，如果没有软约束，冲突则报错。

```sv
class Packet;
    rand int length;
    constraint deflt {soft length inside {32, 1024};}
endclass

Packet p = new();
p.randomize() with {length == 1512;} //使用with添加额外约束
```

随机方法：randomize()，如果随机化成功返回1。指向模糊和指向明确。
```sv
class C1;
    rand integer x;
endclass

class C2;
    integer x, y;

    task doit(C1 f, integer x, integer z);
        int result;
        result = f.randomize() with {x < y + z;}        //这里三个x，但是这个x是C1::x
        result = f.randomize() with (x) {x < y + z;}    //x指向明确，是C1::x
        result = f.randomize() with {x < loacl::x;}     //第一个x是C1::x，第二个x是task::x
        result = f.randomize() with {this.x < y + z;}   //x仍然是C1::x
    endtask
endclass
```

> ```sv
> class SimpleSum;
>       rand bit [7:0] x, y, z;
>       constraint c {soft z == x + y;}
> endclass
> SimpleSum s = new();
> s.randomize() with {z > x + y;};
> ```
> 下面哪组值可能是SimpleSum对象随机化的值？BCD
> A x=1, y=1, z=2
> B x=1, y=1, z=3
> C x=1, y=0, z=2
> D x=0, y=0, z=1

#### 随机控制
随机控制：rand_mode可以用来使能禁止随机变量。当随机数被禁止时，和普通变量一样。可以在函数声明中全局控制其中所有的随机变量：
```sv
task object[.random_variable]::rand_mode(bit on_off);  //对object中的random_variable变量控制
function int object.random_variable::rand_mode();

class Packet;
    rand integer src, dst;
endclass

int ret;
Packet p = new();
p.rand_mode(0);             //全部关闭
p.src.rand_mode(1);         //打开src
ret = p.dst.rand_mode();    //ret = 0，因为dst没打开
```
约束块也可以控制使能，关键字是constraint_mode，用法同上。
**内嵌变量控制**：randomize传递参数时，只随机化传递的参数，无论类内部这些变量是否是rand/randc修饰。
```sv
class CA;
    rand byte x, y;
    byte v, w;
endclass

a.randomize()       //x,y
a.randomize(w,x)    //w,x
```

### 任务和函数
任务和函数可以提高代码的复用性和整洁性。任务和函数的参数列表都可以为空。目的都在于将大的过程块切分为小片段，而便于阅读和代码维护。区别：
* function不会消耗仿真时间，而task则可能会消耗仿真时间。
* function无法调用task，而task可以调用function。
* 一个可以返回数据的function只能返回一个单一数值，而任务或者void function不会返回数值。
* —个可以返回数据的function可以作为一个表达式中的操作数，而该操作数的值即function的返回值。

#### 任务task
任务定义可以指定参数，input，output，inout，ref都可。任务可以消耗仿真时间。**任务可以调用其他任务或者函数**。**task没有返回值**，但是可以利用return来使得task结束。
```sv
task mytask(output int x, input logic y);
    ...
endtask
```

#### 函数function
函数不可以调用任务，函数可以调用函数。void函数不会返回数值。函数列表方向同任务，没有给出方向和第一个默认保持一致。函数的返回类型有两种，return或者声明函数名，**return会立即返回，而赋值给函数同名变量后，函数代码继续执行**。如果调用函数有返回值，但是又不打算用该返回值时，建议添加void'()进行转换。
```sv
void'(some_function())
```

参数传递：
* **input 参数**方向在方法调用时，属于值传递。即传递的过程中，外部变量的值会经过拷贝，赋值于形式参数。output、inout也会在传入或者传出的时候发生值传递的过程。值传递的过程值发生在方法调用时和返回时。
* **ref参数**在传递时不会发生值拷贝，而是指针传递到方法中，在方法内部对该参数的操作将会影响外部变量。为了避免外部传入的ref参数会被方法修改，可以添加const修饰符，来表示变量是只读变量。

SV允许方法声明参数的默认值，参数的方向可以是input，inout，output和ref。带有参数默认值的方法被调用时，如果这些参数没有被传递值，那么编译器将会为这些参数传入对应的默认值。SV允许类似于模块例化，可由参数位置顺序在调用方法时传递参数，也可以由参数名字调用方式时绑定参数。

```sv
function int fun(int j = 1, string s = "no");

endfunction

fun(.j(2), .s("yes"));      //fun(2, "yes");
fun(.s("yes"))              //fun(1, "yes")  j默认值1
```

> 下列关于函数和任务的说法哪些是正确的 CD
> A 函数可以调用任务
> B 任务可以有返回值
> C 函数和任务的参数列表都可以为空
> D 函数和任务的参数可以有默认值

### 线程控制
Verilog中与顺序线程begin...end相对的是并行线程fork...join，SV引入了join_any和join_none
* fork...join需要所有并行的线程都结束后才会继续执行
* fork...join_any则会等到任意一个线程结束以后就继续执行
* fork...join_none则不会等待其子线程而继续执行

fork...join_any和fork...join_none继续执行后，其一些未完成的子程序仍将在后台继续执行。如果要等待这些子程序执行完毕，阻塞住，使用wait_fork。如果要停止这些子程序，使用disable_fork。

时序控制：SV可以通过延迟控制或者时间等待来对过程块完成时序控制。延迟控制通过``#``来完成。事件event通过``@``完成。wait语句也可以与事件或者表达式结合来完成。**时序控制只能在task中出现**，因为函数不允许延迟。
```sv
#10 rega = regb;

@r rega = regb;
@(posedge clock) rega = regb;

initial wait(AOR.size() > 0) ...;
```
### 进程间同步和通信
#### 事件event
测试平台中的所有线程都需要同步并交换数据。一个线程等待另一个，例如验证环境需要等待所有的激励结束才可以结束仿真，比如监测器需要将监测到的数据发送至比较器，比较器又需要从不同的缓存获取数据进行比较。
* 可以通过event来声明一个event变量，并去触发它。（个人感觉event就是Linux高级编程的信号）
* 这个命名event可以用来控制进程的执行
* 可以通过``->``来触发事件
* 其他等待该事件的进程可以通过``@``操作符或者wait()来检查event触发状态来完成

```sv
event done, blast;
event done_too = done; //done_too和done是同一个事件

task trigger(event ev);
    -> ev;
endtask

fork
    @ done_too;        //done被触发后这行执行同时fork退出
    #1 trigger(done);  //这行先执行
join

fork
    -> blast;
    wait (blast.triggered);   //等待blast触发，注意如果这是是@ blast将有可能错过blast，因为两者是同时的，wait支持对已经触发的event查询
join
```
wait_order，按照列表的顺序从左到右依次完成，顺序不对则返回失败。

> 下列关于event的说法哪些是正确的 ABD
> A event之间可以拷贝
> B 被拷贝的event与拷贝的event指同一个event
> C 如果event先发生再@event，不会被阻塞
> D 如果event先发生，再wait(event.triggered)，不会被阻塞

#### 旗语semaphore
旗语从概念上讲是一个容器，一个类。在创建旗语时，会为其分配固定的钥匙数量。使用旗语的进程必须先获得钥匙，才可以继续执行。旗语的钥匙数量可以有多个，等待旗语钥匙的进程也可以同时有多个。旗语通常用于互斥，对共享资源的访问控制，以及基本的同步。
```sv
semaphore sm;   //创建旗语，并分配钥匙
sm = new();     //缺省值0，返回旗语的句柄，可以放入超出keyCount数量，而不是删除
new(N=0);       //创建固定钥匙数量的旗语
get(N=1);       //获得钥匙（阻塞），缺省值1，钥匙不足的时候将等待
                //旗语的等待队列时先进先出FIFO
put(N=1);       //还指定数量的钥匙，缺省值为1
try_get(N=1);   //尝试获取钥匙（非阻塞），缺省值为1，钥匙不足返回0，足够则返回正数（实际拿到的）
```

#### 信箱mailbox
类似高级语言的消息队列，FIFO。
* 创建信箱：new()，可以指定长度（正数），缺省值为0，表示不限制大小
* 信箱声明时需要指定存储类型
* 将信息写入信箱：put()，信箱满阻塞
* 写入但不阻塞：try_put()，信箱满则返回0，写成功返回1
* 获取信息：get()获取并取出，peek()获取不取出（拷贝），如果空则阻塞
* 取出不阻塞：try_get()/try_peek()，信箱为空返回0，读取成功返回1
* 获取mailbox的信息数量：num()，结合num()避免get/put阻塞
```sv
typedef mailbox #(string) s_mbox;

s_mbox sm = new();
string s;
sm.put("Helo");

sm.get(s);   //s = "Hello"
```

> 下列关于信箱的说法，哪些是正确的 AB
> A 可以置定信箱存储的数据类型
> B 可以不限制信箱的容量大小
> C 在信箱写满时继续写入，数据会溢出丢失
> D 在信箱为空时，调用get()方法将返回数值0

### 虚方法--多态
类的成员方法可以加修饰词virtual，虚方法。虚方法是一种基本的多态polymorphic结构。一个虚方法可以覆盖基类的同名方法。在父类和子类中声明的虚方法，其方法名、参数名、参数方向等应当保持一致。在调用虚方法时，它将调用句柄指向对象的方法，而不受句柄类型的影响。virtual不能修饰变量，只能修饰成员。

举例，父类中定义了虚方法，子类中定义了同名的虚方法（子类可以不用virtual修饰，只要同名同参数），**将子类的句柄赋值给父类的句柄**，父类的句柄此时指向子类的对象，按照普通的逻辑，父类句柄只能访问父类的方法和成员。**但当其中有虚方法时，父类句柄调用的是子类的方法**。即子类的方法覆盖了父类的方法。如果子类想访问父类的成员和方法使用super关键字。

## 验证平台实现
### MCDT测试平台结构
![MCDT](./mcdt.png)
channel initiator和channel monitor一一对应，initiator和monitor被封装在了agent，由于arbiter不需要握手需要，因此只外置了mcdt monitor而不需要mcdt responder。所有的monitor都将检测到的数据送至mcdt checker。mcdt checker内置若干FIFO，接收从MCDT的输入端和输出端监测到的数据。由于checker输入和输出端的FIFO存储的数据类型一致，这为数据比较带来了方便。
### channel stimulator
#### 数据包类
首先定义数据包``chnl_trans``的类，包含数据，id，间隔时间，回复信号标志等，对这些变量建立随即约束，并加入clone的深拷贝方法。
```sv
class chnl_trans;
    rand bit[31:0] data[];
    rand int ch_id;
    rand int pkt_id;
    rand int data_nidles;
    rand int pkt_nidles;
    bit rsp;
    local static int obj_id = 0;
    constraint cstr{   //随机约束
        soft data.size inside {[4:8]};
        foreach(data[i])
            data[i] == 'hC000_0000 + (this.ch_id<<24) + (this.pkt_id<<8) + i;
        soft ch_id == 0;
        soft pkt_id == 0;
        data_nidles inside {[0:2]};
        pkt_nidles inside {[1:10]};
    };

    function new();
        this.obj_id++;  //每个对象的id是独一无二的，因为是local静态的
    endfunction

    function chnl_trans clone();
        chnl_trans c = new();
        c.data = this.data;
        c.ch_id = this.ch_id;
        c.pkt_id = this.pkt_id;
        c.data_nidles = this.data_nidles;
        c.pkt_nidles = this.pkt_nidles;
        c.rsp = this.rsp;
        return c;
    endfunction
endclass: chnl_trans
```
#### initiator
定义initiator模块，将generator的数据发送给slave_fifo模块。initiator通过成员变量name认证，并建立模块之间通信的mailbox，这个mailbox还是在类内部，之后会联合generator类内部的mailbox一起。
* set_interface从外部导入接口
* drive方法从generator的输出mailbox获取数据包，拷贝并rsp置位返回给generator模块；
* chnl_write方法将数据输出给salve_fifo模块
* chnl_idle方法空闲打拍

```sv
class chnl_initiator;
    local string name;
    local virtual chnl_intf intf;   //用virtual修饰interface的指针 
    mailbox #(chnl_trans) req_mb;   //与硬件通信的，请求信箱，类型是chnl_trans，句柄，例化在gennerator中例化
    mailbox #(chnl_trans) rsp_mb;   //回复信箱，发送回generator
  
    function new(string name = "chnl_initiator");
        this.name = name;
    endfunction
  
    function void set_interface(virtual chnl_intf intf); //传递接口指针进来并赋值给内部
        if(intf == null)
            $error("interface handle is NULL, please check if target interface has been intantiated");
        else
            this.intf = intf;
    endfunction

    task run();
        this.drive();
    endtask

    /*
    ------------------       get        ---------------------
    |          req_mb  |--------------->|---x-----------------|-------->chnl write
    |                  |                |   |                 |
    |    generator     |                |   |clone  initiator |
    |                  |      put       |   |                 |
    |          rsp_mb  |<---------------|<--x                 |
    ------------------                  ---------------------
    */
    task drive();
        chnl_trans req, rsp;      //声明两个请求和回复的句柄
        @(posedge intf.rstn);     //在interface的复位时
        forever begin             //永远运行
            this.req_mb.get(req);
            this.chnl_write(req);
            rsp = req.clone();
            rsp.rsp = 1;
            this.rsp_mb.put(rsp);
        end
    endtask
  
    task chnl_write(input chnl_trans t);
        foreach(t.data[i]) begin
            @(posedge intf.clk);
            intf.ch_valid <= 1;         //写数据拉高
            intf.ch_data <= t.data[i];  //写一位数据
            @(negedge intf.clk);
            wait(intf.ch_ready === 'b1);
            $display("%0t channel initiator [%s] sent data %x", $time, name, t.data[i]);
            repeat(t.data_nidles) chnl_idle();   //重复chnl_idle data_nidles次，等拍
        end
        repeat(t.pkt_nidles) chnl_idle();        //重复chnl_idle pkt_nidles次，等拍
    endtask
    
    task chnl_idle();
        @(posedge intf.clk);
        intf.ch_valid <= 0;
        intf.ch_data <= 0;
    endtask
endclass: chnl_initiator
```
![](initiator.png)

#### generator
generator模块直接生成数据包，并从initiator获取response（initiator直接复制而来的）。
```sv
class chnl_generator;
    rand int pkt_id = -1;           //为什么是-1
    rand int ch_id = -1;
    rand int data_nidles = -1;
    rand int pkt_nidles = -1;
    rand int data_size = -1;
    rand int ntrans = 10;

    mailbox #(chnl_trans) req_mb;   //和initiator中的mailbox对应
    mailbox #(chnl_trans) rsp_mb;

    constraint cstr{
        soft ch_id == -1;
        soft pkt_id == -1;
        soft data_size == -1;
        soft data_nidles == -1;
        soft pkt_nidles == -1;
        soft ntrans == 10;
    }

    function new();
        this.req_mb = new();       //此处例化信箱
        this.rsp_mb = new();
    endfunction

    task run();
        repeat(ntrans) send_trans();   //重复发送ntrans次
        run_stop_flags.put();          //旗语放钥匙，告诉其他任务这里run stop，发送完毕
    endtask

    // generate transaction and put into local mailbox
    task send_trans();
        chnl_trans req, rsp;
        req = new();
        //assert判断随机是否成功
        //随机约束嵌套，随机化一个req，如果generator例化时ch_id大于0（为什么初始值为-1，是一个判断条件），
        //那么req的ch_id也大于0（即req的随即约束与generator一致），否则采用req的自己的默认例化方式
        assert(req.randomize with {local::ch_id >= 0 -> ch_id == local::ch_id; 
                                  local::pkt_id >= 0 -> pkt_id == local::pkt_id;
                                  local::data_nidles >= 0 -> data_nidles == local::data_nidles;
                                  local::pkt_nidles >= 0 -> pkt_nidles == local::pkt_nidles;
                                  local::data_size >0 -> data.size() == local::data_size; 
                                })
            else $fatal("[RNDFAIL] channel packet randomization failure!");
        this.pkt_id++;         //0-9，这个值会传递给req的pkt_id
        this.req_mb.put(req);  //往输出信箱放包
        this.rsp_mb.get(rsp);  //接收回复信箱的包
        assert(rsp.rsp)        //判断返回的包是不是initiator clone的，initiator会赋值rsp为1
            else $error("[RSPERR] %0t error response received!", $time);
    endtask
endclass: chnl_generator
```
### channel monitor
channel monitor获取从输入的总线里面获取有效数据，并通过mailbox传递给checker。monitor是一直运行的。
```sv
typedef struct packed {   //合并型，空间更紧凑
    bit[31:0] data;       //数据
    bit[1:0] id;          //channel的id，0，1，2
} mon_data_t;

class chnl_monitor;
    local string name;
    local virtual chnl_intf intf;
    mailbox #(mon_data_t) mon_mb;     //这个只是个句柄，没有例化，例化在checker里面

    function new(string name="chnl_monitor");
        this.name = name;
    endfunction

    function void set_interface(virtual chnl_intf intf);
        if(intf == null)
            $error("interface handle is NULL, please check if target interface has been intantiated");
        else
            this.intf = intf;
    endfunction

    task run();
        this.mon_trans();
    endtask

    task mon_trans();
        mon_data_t m;
        forever begin   //死循环
            @(posedge intf.clk iff (intf.ch_valid==='b1 && intf.ch_ready==='b1));  //上升沿并且valid=1，ready=1
            m.data = intf.ch_data;     //m是结构体
            mon_mb.put(m);             //获取输入的数据（有效数据），并放在mon_mb信箱中，将传递给checker
            $display("%0t %s monitored channle data %8x", $time, this.name, m.data);
        end
    endtask
endclass

```

### mcdt monitor
mcdt monitor从arbiter的输出总线获取有效数据并通过mailbox传递给checker。
```sv
class mcdt_monitor;
    local string name;
    local virtual mcdt_intf intf;
    mailbox #(mon_data_t) mon_mb;          //同样没有例化
    function new(string name="mcdt_monitor");
        this.name = name;
    endfunction
    task run();
        this.mon_trans();
    endtask

    function void set_interface(virtual mcdt_intf intf);
        if(intf == null)
            $error("interface handle is NULL, please check if target interface has been intantiated");
        else
            this.intf = intf;
    endfunction

    task mon_trans();
        mon_data_t m;
        forever begin
            @(posedge intf.clk iff intf.mcdt_val==='b1);    //时钟上升沿和valid为高
            m.data = intf.mcdt_data;
            m.id = intf.mcdt_id;                            //将id也写入到mailbox中
            mon_mb.put(m);
            $display("%0t %s monitored mcdt data %8x and id %0d", $time, this.name, m.data, m.id);
        end
    endtask
endclass

```

### mcdt checker

```sv
class mcdt_checker;
    local string name;
    local int error_count;
    local int cmp_count;
    mailbox #(mon_data_t) in_mbs[3];    //3个channel的monitor mailbox
    mailbox #(mon_data_t) out_mb;       //arbiter的输出信箱

    function new(string name="mcdt_checker");
        this.name = name;
        foreach(this.in_mbs[i])
            this.in_mbs[i] = new();
        this.out_mb = new();
        this.error_count = 0;
        this.cmp_count = 0;
    endfunction

    task run();
        this.do_compare();
    endtask

    task do_compare();
        mon_data_t im, om;
        forever begin
            out_mb.get(om);                 //从arbiter的输出拿数据
            case(om.id)                     //检查id
                0: in_mbs[0].get(im);       //从对应channel拿数据，这里不会真的阻塞，因为当arbiter有输出时，输入数据应当已经被channel monitor送过来了
                1: in_mbs[1].get(im);
                2: in_mbs[2].get(im);
                default: $fatal("id %0d is not available", om.id);
            endcase
            if(om.data != im.data) begin    //判断两个数据是否一致
                this.error_count++;
                $error("[CMPFAIL] Compared failed! mcdt out data %8x ch_id %0d is not equal with channel in data %8x", om.data, om.id, im.data);
            end
            else begin
                $display("[CMPSUCD] Compared succeeded! mcdt out data %8x ch_id %0d is equal with channel in data %8x", om.data, om.id, im.data);
            end
            this.cmp_count++;
        end
    endtask
endclass
```
### 实现顶层环境
#### agent
agent将channel monitor和channel initiator打包一起。
```sv
class chnl_agent;
    local string name;
    chnl_initiator init;
    chnl_monitor mon;
    virtual chnl_intf vif;
    function new(string name = "chnl_agent");
        this.name = name;
        this.init = new({name, ".init"});   //加后缀，并例化
        this.mon = new({name, ".mon"});
    endfunction

    function void set_interface(virtual chnl_intf vif);
        this.vif = vif;
        init.set_interface(vif);
        mon.set_interface(vif);
    endfunction
    task run();
        fork
            init.run();
            mon.run();   //两个进程都是一直运行的，并行
        join
    endtask
endclass: chnl_agent
```

#### mcdt_root_test
mcdt_root_test将各模块例化并连接，这是一个顶层的测试用例，其他测试都是继承这个类。这个类主要功能：
* new实例化各模块并完成连接
* set_interface接口的传递
* 环境的运行
  * 留下do_config初始化配置和gen_stop_callback的虚方法
  * 定义run_stop_callback的虚方法，默认是generator跑完
  * 定义run的虚方法

```sv
class mcdt_root_test;
    chnl_generator gen[3];      //3个gennerator
    chnl_agent agents[3];       //3个agent，包含initiator和monitor
    mcdt_monitor mcdt_mon;      //mcdt monitor
    mcdt_checker chker;         //mcdt checker
    protected string name;
    event gen_stop_e;           //generator停下来的事件

    function new(string name = "mcdt_root_test");
        this.name = name;
        this.chker = new();     //例化checker
        foreach(agents[i]) begin
            this.agents[i] = new($sformatf("agents[%0d]",i));
            this.gen[i] = new();                                //例化每个agent的generator
            this.agents[i].init.req_mb = this.gen[i].req_mb;    //将agent的initiator的req_mb指向generator实例化的req_mb
            this.agents[i].init.rsp_mb = this.gen[i].rsp_mb;    //将agent的initiator的rsp_mb指向generator实例化的rsp_mb
            this.agents[i].mon.mon_mb = this.chker.in_mbs[i];   //将agent的monitor的mon_mb指向checker实例化的in_mbs
        end
        this.mcdt_mon = new();                                  //实例化mcdt monitor
        this.mcdt_mon.mon_mb = this.chker.out_mb;               //将mcdt monitor的mon_mb指向checker实例化的out_mb
        $display("%s instantiated and connected objects", this.name);
    endfunction

    virtual task gen_stop_callback();       //虚方法
      // empty 主动触发gen_stop_e，默认不触发，留给子类
    endtask

    virtual task run_stop_callback();       //虚方法
        $display("run_stop_callback enterred");
        // by default, run would be finished once generators raised 'finish'
        // flags 
        $display("%s: wait for all generators have generated and tranferred transcations", this.name);
        run_stop_flags.get(3);              //三个channel的generator都还完钥匙后，停止仿真
        $display($sformatf("*****************%s finished********************", this.name));
        $finish();
    endtask

    virtual task run();
        $display($sformatf("*****************%s started********************", this.name));
        this.do_config();                   //配置数据
        fork
            agents[0].run();                //agent等一直运行的开始运行，generator除外
            agents[1].run();
            agents[2].run();
            mcdt_mon.run();
            chker.run();
        join_none

        // run first the callback thread to conditionally disable gen_threads
        fork
            this.gen_stop_callback();                //如果是空的，则fork join_none结束，下一行等待不会执行
            @(this.gen_stop_e) disable gen_threads;  //等待gen_stop_e的事件，随时关掉gen_thread线程
        join_none

        fork : gen_threads                           //开始运行gen_threads
            gen[0].run();                            //generator如果运行结束后，还钥匙，generator最多只发十次数据
            gen[1].run();
            gen[2].run();
        join

        run_stop_callback(); // wait until run stop control task finished
    endtask

    virtual function void set_interface(virtual chnl_intf ch0_vif, virtual chnl_intf ch1_vif, virtual chnl_intf ch2_vif, virtual mcdt_intf mcdt_vif);
        agents[0].set_interface(ch0_vif);
        agents[1].set_interface(ch1_vif);
        agents[2].set_interface(ch2_vif);
        mcdt_mon.set_interface(mcdt_vif);
    endfunction

    virtual function void do_config();              //虚方法
        //empty
    endfunction
endclass
```
### 实现测试用例
实际的测试用例需要继承mcdt_root_test并具体定义产生的数据和虚方法。
#### 基础测试
```sv
class mcdt_basic_test extends mcdt_root_test;
    function new(string name = "mcdt_basic_test");
        super.new(name);        //父类实例化，必须
    endfunction
    virtual function void do_config();
        super.do_config();      //父类do_config()如果有代码并子类想继承，此时还是需要执行的
        assert(gen[0].randomize() with {ntrans==100;
                                        data_nidles==0;
                                        pkt_nidles==1;
                                        data_size==8;})
            else $fatal("[RNDFAIL] gen[0] randomization failure!");
        assert(gen[1].randomize() with {ntrans==50;
                                        data_nidles inside {[1:2]};
                                        pkt_nidles inside {[3:5]};
                                        data_size==6;})
            else $fatal("[RNDFAIL] gen[1] randomization failure!");
        assert(gen[2].randomize() with {ntrans==80;
                                        data_nidles inside {[0:1]};
                                        pkt_nidles inside {[1:2]};
                                        data_size==32;})
            else $fatal("[RNDFAIL] gen[2] randomization failure!");
    endfunction
endclass: mcdt_basic_test
```

## 验证量化管理
### 概述
* 是否所有设计的功能在验证计划中都已经验证？
* 代码中的某些部分是否从未执行过？

覆盖率是回答以上问题的指标，覆盖率已经被广泛采用，作为衡量验证过程中的重要数据。覆盖率就是用来衡量验证精度和完备性的数据指标。覆盖率可以告诉我们在仿真时哪些结构被触发和未被触发。

只有满足以下三个条件，才可以在仿真中实现高质量的验证：
* 测试平台必须产生合适的激励来触发一个设计错误
* 测试平台仍然需要产生合适的激励使得被触发的错误可以进一步传导到输出端口
* 测试平台需要包含一个监测器 (monitor）用来监测被激活的设计错误，以及在它传播的某个节点（内部或者外部）可以捕捉到它

### 覆盖率种类
* 没有任何一种单一的覆盖率可以完备地去衡量验证过程
* **即使我们可以达到100%的代码覆盖率，但这并不意味着100%的功能覆盖率**。原因在于代码覆盖率并不是用来衡量设计内部的功能运转，或者模块之问的互动，或者功能时序的触发等。
* 类似地，我们即便达到了100%功能覆盖率，也可能只达到了90%的代码覆盖率。原因可能在于我们疏漏了去测试某些功能，或者一些实现的功能并没有被描述。
* 从上述关于代码覆盖率和功能覆盖率简单的论述就可以证明，如果想要得到全面的验证精度，我们就需要多个覆盖率种类的指标。


| class     | Implementation | Specification                                       |
| -------- | -------------- | --------------------------------------------------- |
| **Implicit** | Code Coverage  | Area of research(Currently no industrial solutions) |
| **Explicit** | Assertions     | Functional Coverage/Assertions                      |


最常见的划分覆盖率的两种方法
* 按照覆盖率生成的方法，即隐性（自动）生成还是显性（人为）生成。
* 按照覆盖率朔源，即它们从功能描述而来还是从设计实现而来。

例如功能覆盖率是显性的需要人为定义的覆盖率，而代码覆盖率则是隐性覆盖率这是因为仿真工具可以自动从RTL代码来生成。

如果将上述两个分类的方式进行组合，那么可以将代码覆盖率、断言覆盖率以及功能覆盖率分别置入到不同的象限。但是需要注意，目前有一个象限仍然处于研究阶段，没有隐性的可以以功能描述生成某种覆盖率的方法，这也是为什么功能覆盖率依然需要人为定义的原因。

### 代码覆盖率

代码覆盖是一种技术，可以识别在验证设计中已执行的代码。包含末知错误的设计的问题在于它看起来就像一个非常好的设计。我们绝对不可能知道被验证的设计在功能上是完全正确的。即便所有测试平台都成功份真，但是否有部分RTL代码末运行，因此末触发可能的功能错误？这是代码覆盖可以帮助回答的问
题

代码覆盖率并不是SV独有的，这项技术己经在软件工程中使用了相当长的一段时间。代**码覆盖率的一个优势在于它可以由仿真工具自动收集，继而用来指出在测试程序中设计源代码哪些被激活触发，而哪些则一直处于非激活的状态**。由于代码覆盖率自动的特性，在仿真过程中使能它变得很简单，它不需要修改设计或者验证环境。

源代码首先进行检测即在源代码的各个位置添加检查点，以在仿真中记录是否执行了某些特定轨迹。**代码覆盖的目的是确定我们是否忘记在设计中执行某些代码，无需跟踪测试平台的代码并确认它已执行**。如果测试平台中有相当数量的代码没有执行，这也将意味着设计中与该测试意图对应的代码也并未执行。只有在检测到错误时才执行大部分测试平台代码，因此我们对测试平台代码的代码覆盖率指标不太感兴趣。

**代码覆盖率100%并不意味着足够的功能覆盖率**。研究发现，在回归测试实现了90%的代码覆盖率时，平均只有54%的代码会被监测，这意味着即便代码覆盖率的完备性满足，但依然可能在这些代码中存在漏洞。

上述的推断是因为不是所有被覆盖的代码都会得到足够的监测，也由于没有得到足够的监测，因此一些即便被触发的漏洞也在传播的过程中没有到达监测点上一一漏洞便从眼皮底下〝逃逸〞了。另外，代码覆盖率的数据无法直接映射到哪些设计功能被测试了所以从这一点来看，代码覆盖率和功能覆盖率之间又是互相独立。

#### 行覆盖率statement/line
* 用来衡量源代码哪些代码行被执行过，以此来指出哪些代码行没有被执行过。
* 从每一行执行的次数，如果设置最小的执行次数，也可以用来做代码测试的一项指标。
* 代码覆盖率可以指出在一些缺之输入激励的情况下，某些赋值的代码行没有被执行的情况，它也可以指出在一些漏洞影响或者无用代码的影响下，一些代码行也无法被执行的情况。
* 对于那些无用的代码，也就是永远不会被执行的代码，在代码分析时，我们可以将它们从覆盖率数据中过滤掉。


行覆盖率也可以称为块覆盖率，其中块是在执行单个语句时执行的语句序列。只要if语句中的表达式求值为true，就会完全执行名为acked的块。因此，计算该块的执行等同于计算该块中的四个单独语句的执行。
```sv
if (dtack == 'b1) begin: acked
    as      <= l'b0;
    data    <= 16'hZZZZ;
    bus_rq  <= 1'b0;
    state   <= IDLE;
end: acked
```

语句块可能末必明确分隔。wait之后的块可能等待而尚未执行。
```sv
address <=16'hFFED;
ale <= 1'bl;
IW <= 1'bl;
wait (dtack == 1'b1);
read_data = data;
ale <= 1'b0;
```

语句、行或块覆盖率可以衡量验证执行的总代码行数。图形用户界面通常允许用户浏兌源代码并快速识别末执行的语句。
```sv
if (parity == ODD| parity == EVEN) begin
    tx <= compute_parity (data, parity);
    #(tx_time) ;
end
tx <= 1'b0;
#(tx_time);
if (stop_bits == 2) begin
    tx <= l'b0;
    #(tx_time)
end
```

要使语句覆盖率度量达到100%，有必要了解导致执行去覆盖语句所需的条件。在这种情况下，奇偶校验必须设置为ODD或EVEN。一旦确定了条件，就必须了解它们从末发生过的原因。这是一个永远不会发生的情况吗？它是否应该由现有测试环境来验证？或者这是一个被遗忘的情况？如果导致未覆盖的语句执行的条件应该被测试，则表明测试平台在功能上不正确或不完整。如果条件完全被遗忘，则需要添加到测试平台，创建一个全新的测试用例或使用不同的种子进行额外的运行。

#### 分支覆盖率branch
对if/case语句分析，指出其执行的分支轨迹。
#### 条件覆盖率condition/expression
条件覆盖率是用来衡量一些布尔表达式中各个真伪判断的执行轨迹，即穷举并列举出已经实现的条件。
#### 状态机覆盖率FSM
仿真工具由于可以自动识别状态机，因此在收集覆盖率时，也可以将覆盖率状态的执行情况监测到。每个状态的进入次数，状态之间的跳转次数，以及多个状态的跳转顺序都可以由仿真工具记录下来。因为FSM中的每个状态通常使用case语句中的选项进行编码，所以任何末访问的状态都可以通过未覆盖的语句清楚地识别。（不能记录相邻状态的跳转）

为了完全验证状态机的实现（所有的跳转序列），有必要确保设计更具所有转换的期望。

* 覆盖率工具将自动从FSM的实现中提取状态的转换。
* 覆盖工具无法确定转换是否是意图的一部分，或者是否缺少预期的转换。
* 检查提取的状态转换以确保仅有的和所有预期的转换发生。
* FSM仅显示五种指定状态。一旦合成到硬件中，就需要一个3位状态寄存器。这留下了三个未指定的可能状态值，如果收到外界干扰状态值进入末指定状态，设计的正确性会受到影响吗？
* 在未知状态下，除非应用复位，否则无法让设计恢复到指定的行为。
* 设计安全性和可靠性问题因此也在验证工程师工作的范围。

#### 跳转覆盖率toggle
用来衡量寄存器跳转的次数（0->1, 1->0）。一般项目会要求模块的端口实现至少1次0到1，以及1次1到0的跳转来保证模块的集成和模块之间的互动。也有的项目会要求所有的奇存器都应该同端口一样满足跳转的最少次数。**端口跳转覆盖率经常用来测试IP模块之间的基本连接性**，例如检查一些输入端口是否没有连接，或者已经连接的两个端口的比特位数是否不匹配，又或者一些已经连接的输入是否被给定的固定值等，都可以通过跳转覆盖率来发现问题。

#### 100%代码覆盖率
* 简而言之，它表示整个设计已经执行。
* 代码覆盖率表示整个验证套件运行源代码的彻底程度，但它并末以任何方式提供有关验证套件正确性或完整性的指示。
* 代码覆盖率无助于验证设计意图，只是完全执行了RTL代码，无论是否正确。
* **代码覆盖率的结果应该用于帮助识别验证末执行的边界情况或依赖于设计实现的特性**。
* 对于没有覆盖到的测试场景（例如耦合），我们还应考虑其是否相关，或者是处于覆盖率工具的盲区所导致的。

### 功能覆盖率
* 功能验证的目标在于确定设计有关的功能描述是否被全部实现了？
* 这一检查中可能会存在一些不期望的情况：
  * 一些功能没有被实现
  * 一些功能被错误地实现了
  * 一些没有被要求的功能也被实现了
* 我们无法通过代码覆盖率得知要求的功能是否被实现了，而需要显性地通过功能覆盖率与设计功能描述做映射，继而量化功能验证的进程。

在约束随机测试流行于验证时，由于随机测试在仿真时会自动产生上千条测试激励，但我们却无法知道随机产生的激励究竟测试了什么功能。**功能覆盖率是最好的可以协助在回归测试时自动监测哪些功能被激活的方法**。创建功能覆盖率模型需要完成以下两个步骤：
* 从功能描述文档提取拆分需要测试的功能点
* 将功能点量化为与设计实现对应的测试场景和功能覆盖率代码

#### 覆盖组covergroup
覆盖组与类相似，在一次定义以后便可以多次进行例化。覆盖组含有覆盖点(coverpoint)、选项 (option)、形式参数 (argument) 和可选触发 (trigger event)一个覆盖组包含了一个或者多个数据点，全都在同一时间采集。覆盖组可以定义在类里，也可以定义在模块或者程序(program)中。

覆盖组可以采集任何可见的变量，比如程序或模块变量、接口信号或者设计中的任何信号。在类中的覆盖组也可以采集类的成员变量。覆盖组应该定义在适当的抽象层次上。对任何事务的采柱都必须等到数据被待测没计接收到以后。一个类也可以包含多个覆盖组，每个覆盖组可以根据需要将它们使能或者禁止。

```sv
enum { red, green, blue } color;
bit [3:0] pixel_adr, pixel_offset, pixel_hue;
covergroup g2 @ (posedge clk);
    Hue: coverpoint pixel_hue;
    Offset: coverpoint pixel_offset;
    Ax: cross color, pixel_adr;    // cross 2 variables
    all: cross color, Hue, Offset; // cross 1 VARs and 2 CPs
endgroup
g2 cg_inst = new();
```

covergroup...endgroup来定义覆盖组。内部可以定义多个coverpoint。如果不在covergroup声明时指定采样事件，那么默认该覆盖组只能依赖于其另外一个手动采样函数sample()。

我们更多地将covergroup定义在类中，从而可以去覆盖类的成员变量。在类中查明的covergroup的方式被称为嵌入式覆盖组声明，以下的方式将会声明一个匿名的覆盖组类型和一个覆盖组变量cov1
```sv
class xyz;
    bit (3:0] m_x;
    int m_y;
    bit m_z:
    covergroup cov1 @m_z; // embedded covergroup，匿名的类型
        coverpoint m_x;
        coverpoint m_y;
    endgroup
    function new();
        cov1 = new:
    endfunction
endclass
```

一个类中可以声明多个covergroup
```sv
class MC;
    1ogic [3:0]m_x;
    local logic m_z;
    bit m_e;
    covergroup cv1 @ (posedge clk);
        coverpoint m_x:
    endgroup
    covergroup cv2 @ m_e
        coverpoint m_z;
    endgroup
endclass
```

covergroup可以与其它对象同步 (@ m_obj.m_ev)
```sv
class Helper;
    int m_ev;
endclass

class MyClass;
    Helper m_obj;
    int m_a;
    covergroup Cov @ (m_obj.m_ev);
        coverpoint m_a;
    endgroup
    function new();
        m_obj = new;
        // Create embedded covergroup after creating m obj
        Cov = new;
    endfunction
endclass
```

#### 覆盖点coverpoint
—个covergroup可以包含一个或者多个coverpoint, 一个coverpoint可以用来采样数据或者数据的变化。—个coverpoint可以对应多个``bin``（仓），这些仓可以显性指定，也可以隐性指定。coverpoint对数据的采样发生在covergroup采样的时候。


在定义coverpoint时可以不给名字或者给名字，我们建议给不同的coverpoint以不同的名字。这些有名字的coverpoint可以用来做更进一步的处理，例如在交叉覆盖率中使用某个coverpoint。

可以通过``iff``在一些情况下禁止coverpoint的采集：
```sv
covergroup g4 ;
    coverpoint s0 iff (!reset);    //在没有reset的情况下采样
endgroup
```

```sv
covergroup cg (ref int x, ref int y, input int c);   //必须用指针，否则不能正常采样
    coverpoint x;       //创建CBx
    b: coverpoint y;    //创建CE b
    cx: coverpoint x;   //创建CB cx
    option.weight = c;
    bit [7:0] d: coverpoint y[31:24]；//创建CP d
    e: coverpoint x {
        option.weight = 2;
    }
    cross x, y{         //创建CroSSCB xxy
        option.weight
    }

endgroup
```


#### 仓(bin)
关键词bins可以用来将每个感兴趣的数值均对一个独立的bin，或者将所有值对应到一个共同的bin。iff语句也可以用在bin的定义，它表示条件为false，那么采集该bin的时候，该bin的采样数目不会增长。
```sv
bit [9:0] v_a;
covergroup cg @ (posedge clk);
    coverpoint v_a {
    bins a = { [0:63],65 };
    bins b[] = { [127:150], [148:191] };   //44个bin
    bins c[] = { 200,201,202 };
    bins d = { [1000:$] };   //1000~1023
    bins others[] = default;
}
endgroup
```


covergroup的参数也可以被传递到bin的定义中。
```sv
covergroup cg (ref int ra, input int low, int high)@ (posedge clk);
    coverpoint r {
        bins good = { [low : high] };
        bins bad[] = default;
    }
endgroup
int va, vb;
cg cl = new( va, 0, 50 ) ;
cg c2 = new( vb, 120, 600 ) ;
```

在定义bin时，可以使用with来进一步限定其关心的数值。with可以用表达式或者函数来衡量。item是特殊关键字，范围内的元素。
```sv
a: coverpoint x { bins mod3 [] = {[0:255]} with (item % 3 == 0) ;}
coverpoint b { bins func[] = b with (myfunc(item));}
```

除了可以覆盖数值，还可以覆盖数值的变化。
```sv
valuel => value2
valuel => value3 => value4 => value5

range_list1 => range_list2
1, 5 => 6,7       //四种变化可能
trans item [* repeat range ]

3 [* 5]   表示 3=>3=>3=>3=>3
3 [* 3:5] 表示 3=>3=>3，3=>3=>3=>3 或 3=>3=>3=>3=>3
```

除了使用``=>``来表示相邻采样点的变化，也可以使用``->``来表示非相邻采样点的数值变化，在``->``序列后的下一个时序**必须紧跟**``=>``序列的最后一个事件:
```sv
3 [->3] 表示 ...=>3...=>3...=>3
1 => 3 [-> 3] => 5 表示 1...=>3...=>3...=>3 =>5  //这里紧跟5
```

与``[->repeat_range]``类似的有 ``[= repeat_range]`` 也表示非连续的循环，只是与``->``有区别的在于，跟随``=``序列的下一次值变化可以发生在``=``结束后的任何时刻。
```sv
3 [=2] 表示 ...=>3...=>3
1 => 3 [=2] => 6 表示 1...=>3...=>3...=>6
```

```sv
bit [4:1] v_a;
covergroup cg @ (posedge clk) ;
    coverpoint v_a {

        bins sa = (4 => 5 => 6) , ([7: 9],10 => 11,12);
        bins sb[] = (4 => 5 => 6) , ([7:9],10=>11,12);
        bins sc = (12 => 3 [-> 1]);
    }
endgroup
```

如果coverpoint没有指定任何bin，那么SV将会为其自动生成bin，遵循的原则是：
* *如果变量是枚举类型，那么bin的数量是枚举类型的基数（所有枚举数值的合）
* 如果变量是整形（M位宽），那么bin的类型将是2^M和auto_bin_max选项的较小值

#### 视频51/52未记录

# UVM核心特性
## UVM总览
UVM是开源库，包含了很多预定义好的验证环境组件。工厂机制的引入方便了组件创建、环境配置和类型覆盖。UVM从开始到结束的整个过程引入了阶段phase的概念。消息管理机制提高了信息打印的可控性。将验证环境和测试用例隔离，提高了环境的复用性。

预定义的验证环境组件：
uvm_driver
uvm_sequencer
uvm_monitor
uvm_agent
uvm_scoreboard
uvm_env
uvm_test

### 工厂机制的引入

factory使得任何UVM类型在定义时均需要注册。在例化UVM类型时必须使用工厂来创建对象。利用工厂机制创建的过程中，不但自然建立了软件对象之间的层次，也建立了通过〝字符串"实现的层次，这为稍后通过宇符串进行层次化的配置提供了便利。由于创建对象必须通过工厂，这也使得在创建对象之前还有机会可以通过工厂来替换创建类型，这使得类型或者对象覆盖成为了
可能。

### 阶段pahse的引入
在学习SV和完成实验的过程中，我们经历了验证组件从创建到连接，最后再到运行和报告的阶段。在复杂层次化的验证环境建立和运行过程中，有必要让所有的组件在各个阶段中按照层次逐个完成，UVM引入了〝阶段〞(phase)，这使得verifier不需要考虑各个阶段的执行，而由UVM帮助完成。

### 消息管理机制
在没有引入消息管理机制之前，各个组件打印的消息格式、出处、等级、冗余度都无法得到很好的管理。UVM消息管理不断对消息验证严重等级、冗余度、格式等都做好了规范，而旦还可以结合层次化配置，在不同的测试阶段或者测试用例中做出个性化的配置。

### 固定资产和动态资源的划分
类似于静态的硬件结构，在UVM环境创建时，所有的组件会伴随着UVM页层结构一同创建，这属于UVM测试平台的固定资产。在接下来的测试过程中，对于发送的激励、监测到的总线事务等都属于动态资源，即它们会在一定的时刻 （不一定是0时刻）创建，而又在一定的时刻被回收。固定资产一般是uvm_component类型，动态资源一般是um_object类型。
## 基础结构

### 核心基类uvm_project
* uvm_report_object <-- uvm_component
* uvm_transaction <-- uvm_sequence_item <-- uvm_sequence

uvm_object是UV环境结构(静态资产+动态资源）的根类或者也可称为核心基类。uvm_object内置了一些常用方法，包括复制、比较、打印等。由uvm_object的继承，定义了两个子类，uvm_component为所有环境组件 （静态资产）的父类，uvm_transaction为所有测试用例（动态资源）的父类。

#### 定义

```sv
class box extends uvm object;   //继承
    int volume = 120:
    color t color = WHITE;
    string name = "box";
    `uvm_object_utils_begin (box)  //注册
        `uvm_field_int(volume, UVM_ALL_ON)
        `uvm_field_enum(color t, color, UVM_ALL_ON)
        `uvm_field_string(name, UVM_ALL_ON)
    `uvm_object_utils_end
    function new (string name="box");  //构建
        super.new(name) ;
        this.name = name ;
    endfunction
endclass
```

<待续...>
