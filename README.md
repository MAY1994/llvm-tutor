llvm-tutor
=========
[![Build Status](https://travis-ci.org/banach-space/llvm-tutor.svg?branch=master)](https://travis-ci.org/banach-space/llvm-tutor)
[![Build Status](https://github.com/banach-space/llvm-tutor/workflows/x86-Ubuntu/badge.svg?branch=master)](https://github.com/banach-space/llvm-tutor/actions?query=workflow%3Ax86-Ubuntu+branch%3Amaster)
[![Build Status](https://github.com/banach-space/llvm-tutor/workflows/x86-Darwin/badge.svg?branch=master)](https://github.com/banach-space/llvm-tutor/actions?query=workflow%3Ax86-Darwin+branch%3Amaster)


Example LLVM passes - based on **LLVM 9**

LLVM -tutor是一个自包含引用LLVM传递的集合。这是一个针对新手和有抱负的LLVM开发人员的教程。关键功能:完成-包括CMake构建脚本，点亮测试和CI设置的源代码-构建一个二进制LLVM安装(不需要从源代码构建LLVM)现代-基于最新版本的LLVM(并更新每一个版本)

llvm-tutor是一个自包含引用LLVM pass的集合。这是一个针对新手和有抱负的LLVM开发人员的教程。关键功能：

- **完整** - 包括`CMake` 构建脚本, LIT 测试以及CI set-up
- **无需源码** - 根据LLVM的二进制安装文件进行构建(不需要从源代码构建LLVM)
- **现代化** - 基于最新版本的LLVM(并更新每一个版本)



### 关于
本LLVM教程的目的是展示LLVM API并演示使用它的乐趣，功能和便捷性。通过使用惯用的LLVM实现的自包含，可测试的示例来演示API。

本文档说明了如何设置环境，构建示例或进行调试。它包含示例的高级概述，并演示了如何运行它们。

除了代码本身，源文件还包含一些注释，这些注释将指导您完成实现。所有示例均辅以LIT测试，这些测试可验证每次通过是否按预期进行。

最基本的介绍性示例见：[**HelloWorld**](https://github.com/MAY1994/llvm-tutor/tree/master/HelloWorld/)。这是一个带有专用`CMake`脚本的独立子项目。

### Table of Contents
* [HelloWorld](#helloworld)
* [开发环境](#开发环境)
* [构建和测试](#构建测试)
* [Pass概述](#Passes概述)
* [调试](#调试)
* [关于LLVM中的PassManager](#关于LLVM中的PassManagers)
* [Credits & References](#credits)
* [License](#license)


HelloWorld
==========
[HelloWorld.cpp](https://github.com/MAY1994/llvm-tutor/tree/master/HelloWorld/HelloWorld.cpp)中的**HelloWorld** pass是一个自包含的*参考实施例*。 相应的[CMakeLists.txt](https://github.com/MAY1994/llvm-tutor/tree/master/HelloWorld/CMakeLists.txt)实现了源外pass的最小设置。

对于输入模块中定义的每个函数，**HelloWord**均会打印其名称和所用参数的数量。您可以这样构建它：

```bash
export LLVM_DIR=<installation/dir/of/llvm/9>
mkdir build
cd build
cmake -DLT_LLVM_INSTALL_DIR=$LLVM_DIR <source/dir/llvm/tutor>/HelloWorld/
make
```
在测试之前，您需要准备一个输入文件：
```bash
# Generate an llvm test file
$LLVM_DIR/bin/clang -S -emit-llvm <source/dir/llvm/tutor/>inputs/input_for_hello.c -o input_for_hello.ll
```
最后，通过[**opt**](http://llvm.org/docs/CommandGuide/opt.html)运行**HelloWorld**：
```bash
# Run the pass on the llvm file
$LLVM_DIR/bin/opt -load-pass-plugin libHelloWorld.dylib -passes=hello-world -disable-output input_for_hello.ll
# The expected output
(llvm-tutor) Hello from: foo
(llvm-tutor)   number of arguments: 1
(llvm-tutor) Hello from: bar
(llvm-tutor)   number of arguments: 2
(llvm-tutor) Hello from: fez
(llvm-tutor)   number of arguments: 3
(llvm-tutor) Hello from: main
(llvm-tutor)   number of arguments: 2
```
**HelloWorld** pass 不修改输出模块. `-disable-output`标志用于防止**opt**打印bitecode文件。

# 开发环境

## 平台支持和要求

该项目已经在**Linux 18.04**和**Mac OS X 10.14.4**上进行了测试。为了构建**llvm-tutor，**您将需要：
  * LLVM 9
  * 支持C++ 14的C++编译器
  * CMake 3.4.3或更高版本

为了运行pass，您将需要：
  * **clang-9** (生成LLVM输入文件)
  * **opt** (运行pass)

测试还有其他要求（通过安装LLVM-9可以满足这些要求）：
  * [**lit**](https://llvm.org/docs/CommandGuide/lit.html) (又名 **llvm-lit**，用于执行测试的LLVM工具)
  * [**FileCheck**](https://llvm.org/docs/CommandGuide/lit.html) (LIT要求，用于检查测试是否生成预期的输出)

## 在Mac OS X上安装LLVM-9

在Darwin上，您可以使用[Homebrew](https://brew.sh/)安装LLVM 9 ：
```bash
brew install llvm@9
```
这将在 `/usr/local/opt/llvm/`中安装所有必需的头文件，库和工具。

## 在Ubuntu上安装LLVM-9

在Ubuntu Bionic上, 你可以从官方[repository](http://apt.llvm.org/)中安装[LLVM](https://blog.kowalczyk.info/article/k/how-to-install-latest-clang-6.0-on-ubuntu-16.04-xenial-wsl.html)：
```bash
wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key | sudo apt-key add -
sudo apt-add-repository "deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic-9 main"
sudo apt-get update
sudo apt-get install -y llvm-9 llvm-9-dev clang-9 llvm-9-tools
```
这将在 `/usr/lib/llvm-9/`中安装所有必需的头文件，库和工具。

## 从源代码构建LLVM-9

从源代码进行构建可能很慢且难以调试。这不是必需的，但它可能是获取LLVM-9的首选方法。在Linux和Mac OS X上运行以下步骤：
```bash
git clone https://github.com/llvm/llvm-project.git
cd llvm-project
git checkout release/9.x
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD=X86 <llvm-project/root/dir>/llvm/
cmake --build .
```
有关更多详细信息，请阅读[官方文档](https://llvm.org/docs/CMake.html)。

构建&测试
===================
您可以按照以下方式构建**llvm-tutor**（以及所有提供的pass）：
```bash
cd <build/dir>
cmake -DLT_LLVM_INSTALL_DIR=<installation/dir/of/llvm/9> <source/dir/llvm/tutor>
make
```

`LT_LLVM_INSTALL_DIR` 变量应设置为LLVM 9的安装目录或构建目录的根. 它用于查找`LLVMConfig.cmake`用于设置包含和库路径的相应脚本。

为了运行测试，您需要安装**llvm-lit**（又名**lit**）。它没有与LLVM 9软件包捆绑在一起，但是您可以使用**pip**安装它：

```bash
# Install lit - note that this installs lit globally
pip install lit
```
运行测试非常简单：
```bash
$ lit <build_dir>/test
```
瞧！您应该看到所有测试都通过了。

Passes概述
======================
   * [**HelloWorld**](#HelloWorld) - 在输入模块中打印函数并打印参数的数量
   * [**InjectFuncCall**](#注入printf的调用-injectfunccall) - 通过插入对`printf`的调用来检测输入模块
   * [**StaticCallCounter**](#编译时函数调用计数staticcallcounter) - 编译时的直接函数调用计数
   * [**DynamicCallCounter**](#运行时函数调用计数dynamiccallcounter) - 运行时直接函数调用计数
   * [**MBASub**](#mbasub) - 整数`sub`指令的代码转换
   * [**MBAAdd**](#mbaadd) - 8-bit 整数`add`指令的代码转换
   * [**RIV**](#可达整数值-riv) - 查找每个基本块的可达整数值
   * [**DuplicateBB**](#重复基本块-duplicatebb) - 重复基本块，需要RIV分析结果

一旦你建立了这个项目，你就可以分别地进行试验。假设您的机器路径中已经有clang和opt。所有的pass都可以使用LLVM文件。你可以生成一个类似这样的:

```bash
export LLVM_DIR=<installation/dir/of/llvm/9>
# Textual form
$LLVM_DIR/bin/clang  -emit-llvm input.c -S -o out.ll
# Binary/bit-code form
$LLVM_DIR/bin/clang  -emit-llvm input.c -o out.bc
```
选择二进制形式(没有-S)还是文本形式(有-S)并不是重点，但显然后者对人类更友好。除了HelloWorld之外的所有pass都在下面进行了描述。

## 注入Printf的调用 (**InjectFuncCall**)

此pass是代码intrumentation的HelloWorld示例。对于输入module中定义的每个函数，InjectFuncCall将添加(注入)以下对printf的调用:

```bash
printf("(llvm-tutor) Hello from: %s\n(llvm-tutor)   number of arguments: %d\n", FuncName, FuncNumArgs)
```

这个调用会加到每个函数的开头（例如，在所有其他指令之前）。`FuncName`是这个函数的名字而`FuncNumArgs`是这个函数携带的参数数量。

你可能会注意到**InjectFuncCall**与**HelloWorld**有些类似，在两个例子中，pass都会访问所有函数，打印它们的函数名以及参数数量。为了去理解这两个例子的区别，让我们使用`input_for_hello.c`来测试**InjectFuncCall**:

```bash
export LLVM_DIR=<installation/dir/of/llvm/9>
# Generate an LLVM file to analyze
$LLVM_DIR/bin/clang  -emit-llvm -c <source_dir>/inputs/input_for_hello.c -o input_for_hello.bc
# Run the pass through opt
$LLVM_DIR/bin/opt -load <build_dir>/lib/libInjectFuncCall.dylib -legacy-inject-func-call input_for_hello.bc -o instrumented.bin
```

上述指令会生成`instrumented.bin`，它是`input_for_hello.bc`的instrumented版本。为了验证**InjectFuncCall**是否像我们期望的那样执行，你可以检查输出文件（并验证它是否包含对`pritf`的额外调用）或者直接运行它：

```bash
$LLVM_DIR/bin/lli instrumented.bin
(llvm-tutor) Hello from: main
(llvm-tutor)   number of arguments: 2
(llvm-tutor) Hello from: foo
(llvm-tutor)   number of arguments: 1
(llvm-tutor) Hello from: bar
(llvm-tutor)   number of arguments: 2
(llvm-tutor) Hello from: foo
(llvm-tutor)   number of arguments: 1
(llvm-tutor) Hello from: fez
(llvm-tutor)   number of arguments: 3
(llvm-tutor) Hello from: bar
(llvm-tutor)   number of arguments: 2
(llvm-tutor) Hello from: foo
(llvm-tutor)   number of arguments: 1
```

这个输出内容与**HelloWorld**例子输出内容确实非常相似。但是，两者打印`Hello from`的次数不同。实际上，在**InjectFuncCall**中，每当调用一个函数时，都会打印一次`Hello from`，而在HelloWorld中，只在函数定义时打印一次。这很有道理，也暗示了这两种方法的不同之处。是否打印`Hello`是通过以下决定:

- HelloWorld的编译时(pass运行时)

- InjectFuncCall的运行时(instrumented module模块运行时)。

最后但并非最不重要的是，注意在InjectFuncCall的情况下，我们必须使用opt运行pass，然后执行仪表化的IR模块来查看输出。对于HelloWorld来说，使用opt进行传球就足够了。

最后，注意在**InjectFuncCall**情况下，我们必须通过opt运行pass，然后执行instrumented IR module来查看输出。对于**HelloWorld**来说，通过opt运行pass就足够了。

## 编译时函数调用计数(**StaticCallCounter**)

StaticCallCounter将会对编译期间可见的那些输入的LLVM文件中的函数调用进行计数（例如，如果一个函数在一个循环中被调用，这种调用只计数一次）。只考虑直接的函数调用。

```bash
export LLVM_DIR=<installation/dir/of/llvm/9>
# Generate an LLVM file to analyze
$LLVM_DIR/bin/clang  -emit-llvm -c <source_dir>/inputs/input_for_cc.c -o input_for_cc.bc
# Run the pass through opt
$LLVM_DIR/bin/opt -load <build_dir>/lib/libStaticCallCounter.dylib -legacy-static-cc -analyze input_for_cc.bc
```

## 运行时函数调用计数(**DynamicCallCounter**)

DynamicCallCounter将会对运行时的函数调用进行计数。通过instrumenting输入文件来实现，我们要做的就是插入call-counting指令，然后每当函数被调用时执行该指令。这个pass仅会对那些被定义于要计数的module中的函数进行计数。

尽管这个pass的主要目标是分析函数计数，但它还是修改了输入文件。因此它是一个transformation或者instrumentation的pass。通过以下所提供的例子进行测试：

```bash
export LLVM_DIR=<installation/dir/of/llvm/9>
# Generate an LLVM file to analyze
$LLVM_DIR/bin/clang  -emit-llvm -c <source_dir>/inputs/input_for_cc.c -o input_for_cc.bc
# Instrument the input file
$LLVM_DIR/bin/opt -load <build_dir>/lib/libDynamicCallCounter.dylib -legacy-dynamic-cc input_for_cc.bc -o instrumented_bin
# Run the instrumented binary
./instrumented_bin
```

你将会看到以下输出：

```bash
=================================================
LLVM-TUTOR: dynamic analysis results
=================================================
NAME                 #N DIRECT CALLS
-------------------------------------------------
foo                  13
bar                  2
fez                  1
main                 1
```

如果您对一个关于instrumentation代码的介绍性示例感兴趣，那么您可能想先尝试一下InjectFuncCall。

## MBASub混合布尔算术转换

这些pass实现了[混合布尔算术转换](https://tel.archives-ouvertes.fr/tel-01623849/document)。在代码混淆中通常使用类似的转换（您也可以从[Hacker's Delight中](https://www.amazon.co.uk/Hackers-Delight-Henry-S-Warren/dp/0201914654)了解它们），并且很好地说明了LLVM传递可以用于什么以及如何使用。

### **MBASub**
**MBASub** pass实现了下面这个相当基础的表达式:

```
a - b == (a + ~b) + 1
```
基本上，它根据上述公式替换了所有整数除法的实例，相应的LIT测试可以验证公式和实现是否正确。您可以如下运行此过程：

```bash
export LLVM_DIR=<installation/dir/of/llvm/9>
$LLVM_DIR/bin/clang -emit-llvm -S inputs/input_for_mba_sub.c -o input_for_sub.ll
$LLVM_DIR/bin/opt -load <build_dir>/lib/libMBASub.so -legacy-mba-sub input_for_sub.ll -o out.ll
```

### **MBAAdd**
**MBAAdd** pass实现了一个稍微复杂的公式，它仅适用于8位整数：

```
a + b == (((a ^ b) + 2 * (a & b)) * 39 + 23) * 151 + 111
```
它会根据上述标识替换所有的整数`add`指令实例，但仅适用于8位整数。LIT测试验证公式和实现是否正确。您可以像这样运行**MBAAdd**：

```bash
export LLVM_DIR=<installation/dir/of/llvm/9>
$LLVM_DIR/bin/clang -O1 -emit-llvm -S inputs/input_for_mba.c -o input_for_mba.ll
$LLVM_DIR/bin/opt -load <build_dir>/lib/libMBAAdd.so -legacy-mba-add input_for_mba.ll -o out.ll
```
您还可以指定_混淆_的级别，范围为0.0到1.0。`0`表示没有混淆，`1`表示所有`add`指令
将被替换为`(((a ^ b) + 2 * (a & b)) * 39 + 23) * 151 + 111`，例如:

```bash
$LLVM_DIR/bin/opt -load <build_dir>/lib/libMBAAdd.so -legacy-mba-add -mba-ratio=0.3 inputs/input_for_mba.c -o out.ll
```

## 可达整数值 (**RIV**)

对于模块中的每个基本块，**RIV**计算可达的整数值（即可以在特定基本块中使用的值）。有一些LIT测试可以验证这确实是正确的。您可以通过以下方式运行pass：

```bash
export LLVM_DIR=<installation/dir/of/llvm/9>
$LLVM_DIR/bin/opt -load <build_dir>/lib/libRIV.so -riv inputs/input_for_riv.c
```

请注意，与之前的流程不同，此流程将仅生成有关原始模块的IR表示的信息。 如果想要了解原始的C或C ++输入文件，它将不会很有用。

## 重复基本块 (**DuplicateBB**)

这个pass将复制模块中的所有基本块，但没有可达整数值（通过**RIV** pass标识）的基本块除外。这种基本块的一个示例是函数中的入口块，该入口块：
* 不带参数，
* 嵌入在未定义全局值的模块中。

该pass依赖于**RIV** pass，因此您也需要加载它以使**DuplicateBB**起作用：
```bash
export LLVM_DIR=<installation/dir/of/llvm/9>
$LLVM_DIR/bin/opt -load <build_dir>/lib/libRIV.so -load <build_dir>/lib/libDuplicateBB.so -riv inputs/input_for_duplicate_bb.c
```
通过插入`if-then-else`构造并将所有指令（[PHI节点](https://en.wikipedia.org/wiki/Static_single_assignment_form)除外）克隆到新的基本块中来复制基本块。

# 调试

在运行调试器之前，您可能需要分析[LLVM_DEBUG](http://llvm.org/docs/ProgrammersManual.html#the-llvm-debug-macro-and-debug-option) 和 [STATISTIC](http://llvm.org/docs/ProgrammersManual.html#the-statistic-class-stats-option) 宏的输出 。例如，对于**MBAAdd**：
```bash
export LLVM_DIR=<installation/dir/of/llvm/9>
$LLVM_DIR/bin/clang -emit-llvm -S -O1 inputs/input_for_mba.c -o input_for_mba.ll
$LLVM_DIR/bin/opt -load-pass-plugin <build_dir>/lib/libMBAAdd.dylib -passes=mba-add input_for_mba.ll -debug-only=mba-add -stats -o out.ll
```
请注意命令行中的`-debug-only=mba-add`和`-stats`标志——这将启用以下输出：
```bash
  %12 = add i8 %1, %0 ->   <badref> = add i8 111, %11
  %20 = add i8 %12, %2 ->   <badref> = add i8 111, %19
  %28 = add i8 %20, %3 ->   <badref> = add i8 111, %27
===-------------------------------------------------------------------------===
                          ... Statistics Collected ...
===-------------------------------------------------------------------------===

3 mba-add - The # of substituted instructions
```
如您所见，您从**MBAAdd**获得了不错的概要。在许多情况下，这足以了解可能出了什么问题。

对于棘手的问题，只需使用debugger即可。下面我将演示如何调试 [**MBAAdd**](https://github.com/MAY1994/llvm-tutor#mbaadd)。更具体地说，如何在进入`MBAAdd::run`时设置断点。希望这足以使您开始。

## Mac OS X
OS X上的默认调试器是[LLDB](http://lldb.llvm.org/)。通常，您将像这样使用它：
```bash
export LLVM_DIR=<installation/dir/of/llvm/9>
$LLVM_DIR/bin/clang -emit-llvm -S -O1 inputs/input_for_mba.c -o input_for_mba.ll
lldb -- $LLVM_DIR/bin/opt -load-pass-plugin <build_dir>/lib/libMBAAdd.dylib -passes=mba-add input_for_mba.ll -o out.ll
(lldb) breakpoint set --name MBAAdd::run
(lldb) process launch
```
或等效地，通过使用LLDB别名：
```bash
export LLVM_DIR=<installation/dir/of/llvm/9>
$LLVM_DIR/bin/clang -emit-llvm -S -O1 inputs/input_for_mba.c -o input_for_mba.ll
lldb -- $LLVM_DIR/bin/opt -load-pass-plugin <build_dir>/lib/libMBAAdd.dylib -passes=mba-add input_for_mba.ll -o out.ll
(lldb) b MBAAdd::run
(lldb) r
```
此时，LLDB应该在`MBAAdd::run`的入口处中断。

## Ubuntu
在大多数Linux系统上，[GDB](https://www.gnu.org/software/gdb/) 是最受欢迎的调试器。一个典型的例子如下所示：
```bash
export LLVM_DIR=<installation/dir/of/llvm/9>
$LLVM_DIR/bin/clang -emit-llvm -S -O1 inputs/input_for_mba.c -o input_for_mba.ll
gdb --args $LLVM_DIR/bin/opt -load-pass-plugin <build_dir>/lib/libMBAAdd.so -passes=mba-add input_for_mba.ll -o out.ll
(gdb) b MBAAdd.cpp:MBAAdd::run
(gdb) r
```
此时，GDB 应该在`MBAAdd::run`的入口处中断。

# 关于LLVM中的PassManagers

LLVM是一个相当复杂的项目(to put it mildly)，而传递则位于其中心--对于任何[multi-pass
编译器](https://en.wikipedia.org/wiki/Multi-pass_compiler<Paste>)都是如此。为了管理pass，编译器需要Pass Managers。LLVM现在有两个Pass Managers。这很重要，因为根据您的决定使用哪个Pass Managers，实现（尤其是pass注册）会稍有不同。我已尽力使源代码中的区别非常清楚。

## LLVM中的Pass Managers概述

像前面提到的, LLVM中有两个pass managers:
* _Legacy Pass Manager_，当前是默认的通行证管理器
	* 在_legacy_ namespace中实现
	* 有很好的[文档](http://llvm.org/docs/WritingAnLLVMPass.html)（更确切地说，有关于用Legacy PM编写和注册pass的文档也很好）
* _New Pass Manager_ 又名[_Pass Manager_](https://github.com/llvm-mirror/llvm/blob/ff8c1be17aa3ba7bacb1ef7dcdbecf05d5ab4eb7/include/llvm/IR/PassManager.h#L458)（这就是在代码库中的引用方式）
	* 我知道它将[很快成为](http://lists.llvm.org/pipermail/llvm-dev/2019-August/134326.html) LLVM中的默认通行证管理器
	* 源代码有非常详尽的注释, but otherwise I am only aware of
		this great [blog series](https://medium.com/@mshockwave/writing-llvm-pass-in-2018-preface-6b90fa67ae82) by Min-Yih Hsu.

最好的方法是为两个pass manager都实现你的pass。幸运的是，一旦您有一个适用于其中一个的实现，则将其进行相对较简单的扩展使其同样适用于另一个即可。LLVM中的所有pass都为这两者提供了一个接口，这也是我一直试图在此实现的接口。

## New vs Legacy PM When Running Opt
**MBAAdd**为两个pass manager都实现了接口。这是legacy pass manager运行它的方式：

```bash
$LLVM_DIR/bin/opt -load <build_dir>/lib/libMBAAdd.so -legacy-mba-add input_for_mba.ll -o out.ll
```

这就是使用new pass manager运行它的方式：
```bash
$LLVM_DIR/bin/opt -load-pass-plugin <build_dir>/lib/libMBAAdd.so -passes=mba-add input_for_mba.ll -o out.ll
```
有两个区别：
* 插件加载方式：`-load` vs `-load-pass-plugin`
* 您指定运行哪个pass/plugin的方式：`-legacy-mba-add` vs `-passes=mba-add`

命令行选项有所不同，因为使用legacy pass manager可以向**opt** *注册*一个新的命令行选项，而使用new pass manager只需定义pass pipeline 即可（通过`-passes=`）。

Credits
========
This is first and foremost a community effort. This project wouldn't be
possible without the amazing LLVM [online
documentation](http://llvm.org/docs/), the plethora of great comments in the
source code, and the llvm-dev mailing list. Thank you!

It goes without saying that there's plenty of great presentations on YouTube,
blog posts and GitHub projects that cover similar subjects. I've learnt a great
deal from them - thank you all for sharing! There's one presentation/tutorial
that has been particularly important in my journey as an aspiring LLVM
developer and that helped to _democratise_ out-of-source pass development:
* "Building, Testing and Debugging a Simple out-of-tree LLVM Pass" Serge
  Guelton, Adrien Guinet
  ([slides](https://llvm.org/devmtg/2015-10/slides/GueltonGuinet-BuildingTestingDebuggingASimpleOutOfTreePass.pdf),
  [video](https://www.youtube.com/watch?v=BnlG-owSVTk&index=8&list=PL_R5A0lGi1AA4Lv2bBFSwhgDaHvvpVU21))

Adrien and Serge came up with some great, illustrative and self-contained
examples that are great for learning and tutoring LLVM pass development. You'll
notice that there are similar transformation and analysis passes available in
this project. The implementations available here reflect what **I** (aka
banach-space) found most challenging while studying them.

I also want to thank Min-Yih Hsu for his [blog
series](https://medium.com/@mshockwave/writing-llvm-pass-in-2018-preface-6b90fa67ae82)
_"Writing LLVM Pass in 2018"_. It was invaluable in understanding how the new
pass manager works and how to use it. Last, but not least I am very grateful to
[Nick Sunmer](https://www.cs.sfu.ca/~wsumner/index.html) (e.g.
[llvm-demo](https://github.com/nsumner/llvm-demo)) and [Mike
Shah](http://www.mshah.io) (see Mike's Fosdem 2018
[talk](http://www.mshah.io/fosdem18.html)) for sharing their knowledge online.
I have learnt a great deal from it, thank you! I always look-up to those of us
brave and bright enough to work in academia - thank you for driving the
education and research forward!

## 参考
Below is a list of LLVM resources available outside the official online
documentation that I have found very helpful. Where possible, the items are sorted by
date.

* **LLVM IR**
  *  _”LLVM IR Tutorial-Phis,GEPs and other things, ohmy!”_, V.Bridgers, F.
Piovezan, EuroLLVM, ([slides](https://llvm.org/devmtg/2019-04/slides/Tutorial-Bridgers-LLVM_IR_tutorial.pdf),
  [video](https://www.youtube.com/watch?v=m8G_S5LwlTo&feature=youtu.be))
  * _"Mapping High Level Constructs to LLVM IR"_, M. Rodler ([link](https://mapping-high-level-constructs-to-llvm-ir.readthedocs.io/en/latest/))
* **Legacy vs New Pass Manager**
  * _"New PM: taming a custom pipeline of Falcon JIT"_, F. Sergeev, EuroLLVM 2018
    ([slides](http://llvm.org/devmtg/2018-04/slides/Sergeev-Taming%20a%20custom%20pipeline%20of%20Falcon%20JIT.pdf),
     [video](https://www.youtube.com/watch?v=6X12D46sRFw))
  * _"The LLVM Pass Manager Part 2"_, Ch. Carruth, LLVM Dev Meeting 2014
    ([slides](https://llvm.org/devmtg/2014-10/Slides/Carruth-TheLLVMPassManagerPart2.pdf),
     [video](http://web.archive.org/web/20160718071630/http://llvm.org/devmtg/2014-10/Videos/The%20LLVM%20Pass%20Manager%20Part%202-720.mov))a
  * _”Passes in LLVM, Part 1”_, Ch. Carruth, EuroLLVM 2014 ([slides](https://llvm.org/devmtg/2014-04/PDFs/Talks/Passes.pdf), [video](https://www.youtube.com/watch?v=rY02LT08-J8))
* **Examples in LLVM**
  * Examples in LLVM source tree in
    [llvm/examples/IRTransforms/](https://github.com/llvm/llvm-project/tree/bf142fc43347d8a35a71f46f7dda7e2a0a992e0d/llvm/examples/IRTransforms).
    This was recently added in the following commit:

```
commit 7d0b1d77b3d4d47df477519fd1bf099b3df6f899
Author: Florian Hahn <flo@fhahn.com>
Date:   Tue Nov 12 14:06:12 2019 +0000

[Examples] Add IRTransformations directory to examples.
```
* **LLVM Pass Development**
  * _"Getting Started With LLVM: Basics "_, J. Paquette, F. Hahn, LLVM Dev Meeting 2019 (not yet uploaded)
  * _"Writing an LLVM Pass: 101"_, A. Warzyński, LLVM Dev Meeting 2019 (not yet uploaded)
  * _"Writing LLVM Pass in 2018"_, Min-Yih Hsu, [blog series](https://medium.com/@mshockwave/writing-llvm-pass-in-2018-preface-6b90fa67ae82)
  * _"Building, Testing and Debugging a Simple out-of-tree LLVM Pass"_ Serge Guelton, Adrien Guinet, LLVM Dev Meeting 2015 ([slides](https://llvm.org/devmtg/2015-10/slides/GueltonGuinet-BuildingTestingDebuggingASimpleOutOfTreePass.pdf), [video](https://www.youtube.com/watch?v=BnlG-owSVTk&index=8&list=PL_R5A0lGi1AA4Lv2bBFSwhgDaHvvpVU21))
* **LLVM Based Tools Development**
  * _"Introduction to LLVM"_, M. Shah, Fosdem 2018, [link](http://www.mshah.io/fosdem18.html)
  *  [llvm-demo](https://github.com/nsumner/llvm-demo), by N Sumner
  * _"Building an LLVM-based tool. Lessons learned"_, A. Denisov, [blog post](https://lowlevelbits.org/building-an-llvm-based-tool.-lessons-learned/), [video](https://www.youtube.com/watch?reload=9&v=Yvj4G9B6pcU)

License
========
The MIT License (MIT)

Copyright (c) 2019 Andrzej Warzyński

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in
the Software without restriction, including without limitation the rights to
use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies
of the Software, and to permit persons to whom the Software is furnished to do
so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
