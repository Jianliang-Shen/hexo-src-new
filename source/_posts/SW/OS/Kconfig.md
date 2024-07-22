---
title: Kconfig快速上手指南
date: 2022-09-06 00:30:09
tags: 
    - Linux
    - Kconfig
categories: 
    - OS
---

Kconfig是一个常用于Linux内核开发的配置系统。它允许开发人员根据特定需求来自定义内核的编译选项和功能。

<!-- more -->

## Kconfig介绍

Kconfig广泛应用于各种软件的编译系统。如Linux等，和cmake以及make可以很好的搭配起来配置编译的选项，处理编译选项之间的依赖关系。并且支持menuconfig在命令行创建ui供用户手动选择。

在编译系统中，Kconfig包括以下内容：
* defconfig：用户配置文件，内容形式为

```c
CONFIG_FOO=y
# CONFIG_FOO is not set
```

* Kconfig：枚举config，并含有规则约束的文件，内容形式为

```c
config MODVERSIONS
      bool "Set version information on all module symbols"
      depends on MODULES
      help
        Usually, modules have to be recompiled whenever you switch to a new
        kernel.  ...
```

* .config：经过工具或者menuconfig解析和约束后生成的配置选项，包含系统所有可配置的选项清单，内容形式和defconfig一致

* config.h：以header file的形式供编译系统使用，是.config文件的转化

## Kconfig语法

### 菜单属性

* type：bool，int，hex，string，每个配置选项都必须有一个类型。只有两种基本类型：三态和字符串；其他类型基于这两种类型。
* prompt，提示：展示给user，不含prompt将隐藏，以下效果是一致的
```c
bool "Networking support"

bool
prompt "Networking support"
```

* default：配置选项可以具有任意数量的默认值。**如果看到多个默认值，则只有第一个定义的值处于活动状态**。默认值不限于定义它们的菜单条目。这意味着默认值可以在其他地方定义，也可以被早期定义覆盖。只有当用户没有设置其他值（通过上面的输入提示符）时，默认值才会分配给配置符号。如果看到输入提示符，默认值将显示给用户，并可以由他重写。或者，只能使用此默认值的依赖项可以添加为“if”。默认值故意默认为“n”，以避免使构建臃肿。除了少数例外，新的配置选项不应该改变这一点。其目的是让“make oldconfig”从发布到发布尽可能少地添加到配置中。

> 值得“默认y”的东西包括：
>* 以前总是构建的东西的新Kconfig选项应该是“默认y”。
>* 隐藏/显示其他Kconfig选项（但不会生成自己的任何代码）的新门禁Kconfig选项应该是“默认y”，以便人们会看到这些其他选项。
>* “默认n”驱动程序的子驱动程序行为或类似选项。这允许您提供理智的默认值。
>* 每个人都期望的硬件或基础设施，例如CONFIG_NET或CONFIG_BLOCK。这些都是罕见的例外。

* depends on：和if等效，表示依赖于某个选项

```c
config FOO
    bool "foo" if BAR
    // default y if BAR

config FOO
    bool "foo"
    depends on BAR
    default y
```
* select：反选，select FOO [if \<expr\>]。虽然正常依赖项减少了符号的上限（见下文），但反向依赖项可用于强制另一个符号的下限。使用当前菜单符号的值作为最小值<符号>可以设置为。如果多次选择<符号>，则限制设置为最大选择。反向依赖项只能与布尔或三态符号一起使用。

>select应该小心使用。选择将强制符号为一个值，而无需访问依赖项。通过滥用选择，即使FOO依赖于未设置的BAR，您也可以选择符号FOO。**一般情况下，仅对不可见符号（任何地方都没有提示）和没有依赖项的符号使用选择**。这将限制有用性，但另一方面，避免到处都是非法配置。

* imply：弱反选，这类似于“select”，因为它对另一个符号强制执行了下限，但是有依赖的或者可见的选项设置为n的无法被修改。如下例子中，FOO想打开BAZ，但后者依赖BAR，如果BAR为N，那么反选将失败。使用这个可以方便在自动化配置的同时允许用户选择。而select一般是后台自动选择。

```c
config FOO
    bool "foo"
    imply BAZ

config BAZ
    bool "baz"
    depends on BAR
```
>上面的配置，如果BAZ是非常值得选择的，在不使用select的情况下，因为select会跳过depend检查，应当同时imply BAR：

```c
config FOO
    bool "foo"
    imply BAR
    imply BAZ

config BAZ
    bool "baz"
    depends on BAR
```

* visible if：仅适用于menu，即是否显示子menu，和if不同在于，if是就地展开块以内的configs，并有缩进分层。visible则会显示menu标题，用户将进入子菜单看到包含的config内容。

* range：设置数值类型的范围
* help：config的备注，在menuconfig按`?`将弹出帮助
* module：Linux的模块编译选项

### 块

#### 菜单
菜单下的所有子选项都依赖依赖的配置：
```c
menu "Network device support"
      depends on NET
      # visible if NET #与depends on等同

config NETDEVICES
      ...

endmenu
```

menuconfig可以实现选中当前选项后展开菜单：
```
# 选择前：
    [ ] M ---
# 选择后：
    [Y] M -->
# 进入菜单：
    [ ] C1
    [ ] C2

# 写法：
(1):
menuconfig M
if M
    config C1
    config C2
endif

(2):
menuconfig M
config C1
    depends on M
config C2
    depends on M
```

#### 选择

choice是一组互斥的config的组合，默认选择第一个，不可以设置默认值。

```
"choice" [symbol]
<choice options>
<choice block>
"endchoice"
```

#### 其他
* source，导入Kconfig文件。Kconfig还支持对osource，rsource的扩展
* if，判断语句
* comment，是否显示注释在输出文件中

## 使用帮助

### 添加常见功能并可配置

实现与某些架构相关但不是所有架构相关的特征/功能是一个常见的风格。推荐的方法是使用名为H`HAVE_*`的配置变量，该变量在公共Kconfig文件中定义，并由相关架构选择。一个例子是通用的IOMAP功能。
```c
# Generic IOMAP is used to ...
config HAVE_GENERIC_IOMAP

config GENERIC_IOMAP
      depends on HAVE_GENERIC_IOMAP && FOO

config X86
      select ...
      select HAVE_GENERIC_IOMAP
      select ...
```

### 添加需要编译器支持的功能
```c
config STACKPROTECTOR
      bool "Stack Protector buffer overflow detection"
      depends on $(cc-option,-fstack-protector)
```

### 平台架构的选项应当适当隐藏

对于特殊平台的测试，应当添加显示的依赖，在其他平台或架构下，不显示该选项。
```c
config FOO
    bool "user prompt" if ARCH_FOO_VENDOR  
    # 或者 prompt "" if ARCH_FOO_VENDOR
     # bool “Support for foo hardware” depends on ARCH_FOO_VENDOR || COMPILE_TEST
```

### defconfig和Kconfig

defconfig是面向用户的可配置选项，如果一个option，譬如`HAVE_`前缀的，是由ARCH或者PLATFORM决定的后台选项（没有prompt属性），defconfig将无法修改配置这些选项。所以还需要Kconfig文件来约束这些选项，以为每个目标设定隐藏的选项。
