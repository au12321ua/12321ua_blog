---
title: makefile
date: 2024-09-02 16:40:19
tags: ["makefile"]
categories: ["tools"]
excerpt : "makefile的基本用法，待补充..."
---


上一学年，我对C的编译过程都是一知半解，以至于我都不知道如何编译一个稍微复杂一点的项目（之前都是靠vs或者dev-c++自带的工具实现）。在暑期短学期时，稍微系统的学习了程序的编译过程，但对于如何编译一个项目还是不怎么了解，这几天闲着没什么事，便搜索了一下相关内容，发现了make这个工具，进而了解了makefile的基本用法，这里也是简单写写我认识的一个简单过程。

## C程序编译过程

一个C程序的编译过程大概可以分为以下几个步骤：

1. 预处理：预处理器会把所有的#include、#define、#undef等预处理指令展开，并将其替换为实际的C代码，生成`.i`文件。
2. 编译：编译器会把预处理后的代码编译成汇编代码，生成`.s`文件。
3. 汇编：汇编器会把汇编代码转换成机器码,生成`.o`文件。
4. 链接：链接器会把多个目标文件和库文件链接成一个可执行文件。

由于这里不是介绍编译过程的教程，所以我就不详细介绍了。但简单来说，如果用gcc编译器，那么一个简单的项目可以用以下命令实现编译：

```shell
gcc -c source.c -o source.o
gcc -c main.c -o main.o
gcc source.o main.o -o program
```

{% folding green::另外一些可能用到的参数 %}
新增头文件路径 : `-I<path>`
识别`.c`文件头文件依赖：`-MM`
编译过程中开启更多的警告：`-Wall` `-Wextra` 
编译过程中将所有警告视为报错：`-Werror`
{% endfolding %}

## makefile

### 使用makefile想要达到的目的

makefile的主要作用是自动化编译过程，而如何自动化，我希望我每次输入`make`至少应该做到：

1. 如果这个工程没有编译过，那么我们的所有C文件都要编译并被链接。
2. 如果这个工程的某几个C文件被修改，那么我们只编译被修改的C文件，并链接目标程序。
3. 如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的C文件，并链接目标程序。

同时我还希望我编写的makefile本身尽可能的简单与自动化，以便于编写。下面就围绕以上这些展开。

### makefile的基本结构

makefile的基本结构如下：

```makefile
target : prerequisites
    command #注意，这里的命令不能有空格，只能用制表符或换行符分隔
    command
   ...
```

其中意思是目标（target）依赖于前置条件（prerequisites），如果目标不存在或者前置条件中的文件有更新，那么就会执行命令（command）来生成新的目标。具体的命令makefile并不关心，可以是编译命令，也可以是`cat`、`mv`等任何命令。

### makefile的一些规则

#### 伪目标

makefile中的目标可以是文件名，也可以是伪目标。伪目标是一种特殊的目标，它不是一个文件，而更类似于一些操作。比如，我们可以定义一个伪目标`clean`，用于清除所有的中间文件乃至目标文件，这样我们只需要输入`make clean`就可以自动化清除。

具体实例：

```makefile
clean :
    rm -f *.o $(TARGET) 
```

{% folding green::关于伪目标的好习惯 %}
一般来说，`clean`、`install`都是约定俗成的伪目标名称，不会有这样名称的文件或目录。

但以防这种情况出现，一般会在伪目标前声名：

```makefile
.PHONY: <伪目标名1> <伪目标名2>...
```

这样的话，即使有`clean`这样的文件或目录，`make clean`也能正常发挥它的作用。
{% endfolding %}

#### 执行多条命令

makefile中的命令可以是一条，也可以是多条。如果一条命令执行失败，那么makefile会停止执行。

例如：

```makefile
test : 
    pwd
    cd ..
    pwd
```

但make针对每一行指令都会单独创建一个shell环境，如果想要在同一个shell环境中执行，需要在同一行使用`;`间隔（也可以使用`\`换行）。

例如：

```makefile
test : 
    pwd; \
    cd ..; \ 
    pwd
```

还可以使用`&&`执行多行命令，好处是如果前一条命令执行失败，那么后面的命令就不会执行。

#### 控制打印与错误

默认情况下，make会打印出它执行的每一条命令。如果我们不想打印某一条命令，可以在命令前加上`@`，表示不打印命令（但是仍然会执行）。

例如：

```makefile
test : 
    @echo "This command will not be printed"
    ls -l /usr/bin/ | grep "bin"
```

make在执行命令时，会检查每一条命令的返回值，如果返回错误（非0值），就会中断执行。有些时候，我们想忽略错误，继续执行后续命令，可以在需要忽略错误的命令前加上`-`。对于执行可能出错，但不影响逻辑的命令，可以用`-`忽略。

例如：

```makefile
test : 
    rm a.txt
    -cat a.txt
```

#### 隐式规则

当make执行过程中遇见一个未定义的目标时，它会试图使用隐式规则来生成这个目标。隐式规则是指make根据文件名、文件扩展名、目录结构等信息，自动生成命令。

例如对于C程序的`.o`文件，如果我们没有对应的命令，就会自动应用以下的规则：

```makefile
xyz.o: xyz.c
	gcc -c -o xyz.o xyz.c
```

但这一隐式规则的缺陷在于此时`.o`文件无法跟踪`.h`文件的变化。

#### 变量

makefile中的变量可以用于简化makefile，并提高可读性。变量的定义格式为`变量名 = 值`或者`变量名 := 值`，变量名和等号之间不能有空格，通常变量名全大写。引用变量用`$(变量名)`

例如：

```makefile
CC = gcc
CFLAGS = -Wall -g
TARGET = program
OBJS = main.o source.o

$(TARGET) : $(OBJS)
	$(CC) $(CFLAGS) $(OBJS) -o $(TARGET)

clean :
	rm -f $(OBJS) $(TARGET)
```

除了自设的变量外，makefile还提供了一些内置变量与自动变量。

内置变量比如`CC`，默认值为`cc`，`CFLAGS`，默认值为`-c`。当然，我们也可以自定义这些内置变量，比如在上面的例子。

自动变量比如`$@`表示目标文件名，`$^`表示所有前置条件，`$<`表示第一个前置条件。如果存在歧义，那么需要用`$()`括起变量名，就如调用其他变量一般。

### makefile中可能用到的一些函数

- `$(wildcard pattern)`：匹配pattern模式的文件名，返回匹配的文件名列表。
- `$(addprefix prefix,names)`：在names前面添加prefix，返回新的字符串列表。
- `$(addsuffix suffix,names)`：在names后面添加suffix，返回新的字符串列表。
- `$(sort names)`：对names进行排序，返回新的字符串列表。
- `$(dir names)`：返回names的目录部分。
- `$(patsubst pattern,replacement,names)`: 用replacement替换names中所有匹配pattern的部分。
- `sed 's/pattern/replacement/g' file`：用replacement替换file中所有匹配pattern的部分。`(linux环境)

### 一份自动生成依赖的makefile示例

> 引自[廖雪峰的官方网站-makefile教程](https://liaoxuefeng.com/books/makefile/complete/index.html)

目录结构：

```
<project>
├── Makefile
├── build
└── src
    ├── hello.c
    ├── hello.h
    └── main.c
```

makefile内容：

```makefile
SRC_DIR = ./src
BUILD_DIR = ./build
TARGET = $(BUILD_DIR)/world.out

CC = cc
CFLAGS = -Wall

# ./src/*.c
SRCS = $(shell find $(SRC_DIR) -name '*.c')
# ./src/*.c => ./build/*.o
OBJS = $(patsubst $(SRC_DIR)/%.c,$(BUILD_DIR)/%.o,$(SRCS))
# ./src/*.c => ./build/*.d
DEPS = $(patsubst $(SRC_DIR)/%.c,$(BUILD_DIR)/%.d,$(SRCS))

# 默认目标:
all: $(TARGET)

# build/xyz.d 的规则由 src/xyz.c 生成:
$(BUILD_DIR)/%.d: $(SRC_DIR)/%.c
	@mkdir -p $(dir $@); \
	rm -f $@; \
	$(CC) -MM $< >$@.tmp; \
	sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.tmp > $@; \
	rm -f $@.tmp

# build/xyz.o 的规则由 src/xyz.c 生成:
$(BUILD_DIR)/%.o: $(SRC_DIR)/%.c
	@mkdir -p $(dir $@)
	$(CC) $(CFLAGS) -c -o $@ $<

# 链接:
$(TARGET): $(OBJS)
	@echo "buiding $@..."
	@mkdir -p $(dir $@)
	$(CC) -o $(TARGET) $(OBJS)

# 清理 build 目录:
clean:
	@echo "clean..."
	rm -rf $(BUILD_DIR)

# 引入所有 .d 文件:
-include $(DEPS)
```

补充说明：

- `include`的作用类似于C中的`#include`，用于引入其他makefile或者文件。被引用的文件会原模原样的放在当前文件的包含位置。

## 未尽的内容

makefile还有一些更高级的用法，比如条件判断、函数、自动变量、多目标、多重目标等等，这些内容我还没看，因为暂时还没有遇到必须要使用的场景。

此外，我一开始是想要学习`cmake`的使用，但我对c++的了解还不够，还有我目前还不需要构建复杂项目，所以也就没有深入研究。

最后，这篇文章参考了陈皓的CSDN专栏文章[《跟我一起写 Makefile》](https://blog.csdn.net/whitefish520/article/details/103968609)（这里链接给的是汇总版），同时也参考了[廖雪峰的官方网站](https://liaoxuefeng.com/books/makefile/introduction/index.html)，感谢。