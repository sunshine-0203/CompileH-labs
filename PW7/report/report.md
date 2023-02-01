# PW6 实验报告

## 问题回答

### 1-1

由一些带有标号的语句块组成，每个语句块最后是一个`ret`（返回）或者`br`（跳转）指令。大部分块是以条件跳转指令结尾的，代表`while`语句的判断部分；最后一个语句块是无条件跳转指令结尾的，代表进入下一次循环。

如：

```assembly
3:                                                ; preds = %11, %0
  %4 = load i32, i32* @a, align 4
  %5 = icmp sgt i32 %4, 0				;  %5是i1类型存储icmp返回值。icmp指令：sgt是指有符号数是否大于等于的比较，被比较的数可以是寄存器里的值也可以是常数。如果条件满足（%4>0）,返回1；否则是0.
  br i1 %5, label %6, label %9			;  条件转移指令：如果%5是1,转移到标签6对应的代码处；否则转移到标签9标记的代码处。

6:                                                ; preds = %3
  %7 = load i32, i32* %2, align 4
  %8 = icmp slt i32 %7, 10
  br label %9							; 无条件跳转指令：跳转到标签9对应的语句块

9:                                                ; preds = %6, %3
  %10 = phi i1 [ false, %3 ], [ %8, %6 ]	; 如果执行流通过3分支，则%10为false；如果执行流通过6分支，则%10为%8。
  br i1 %10, label %11, label %17		; 条件转移指令：如果%10是1,转移到标签11对应的代码处；否则转移到标签17标记的代码处。

11:                                               ; preds = %9
  %12 = load i32, i32* %2, align 4
  %13 = load i32, i32* @a, align 4
  %14 = add nsw i32 %12, %13
  store i32 %14, i32* %2, align 4
  %15 = load i32, i32* @a, align 4
  %16 = sub nsw i32 %15, 1
  store i32 %16, i32* @a, align 4
  br label %3							; 无条件跳转指令：跳转到标签3对应的语句块
```



### 1-2

以func_test中调用自定义的加法函数为例：

```assembly
%5 = call i32 @add(i32 %3, i32 %4)
```

%5用于存调用函数的结果，和函数返回值类型有关。如果函数返回void，则不需要存值；否则用一个寄存器（如此处%5）存储返回值。

call 指令用于函数调用。

i32表示函数返回值。

后面是`@FuncName()`，括号里是参数的类型和需要传递的参数的值。



### 2-1

- `%2 = getelementptr [10 x i32], [10 x i32]* %1, i32 0, i32 %0`

  %1是指向一组指针的指针，可以看成是一组指针的首地址。每个指针指向一个数组，数组是`[10 x i32]`类型的，也就是由10个i32类型组成。

  第一个`i32 0`指的是“在%1指向的这一组指针中取第一个”，第二个`i32 0`指的是“在这个指针的基础上偏移0”，因此%2是第一个数组的第一个元素的地址。

  这条语句用于声明数组之后对数组的使用。

- `%2 = getelementptr i32, i32* %1, i32 %0`

  %1是一个指针，指向i32类型数据；可以看成是数组的首地址。在%1的基础上偏移0，也就是得到指向数组首元素的指针。因此%2是指向数组首元素的指针。

  这条语句用于传递参数时对数组的传递和引用。



### 3-1

`pipeline`和`bind`.

### 3-2

功能是生成并输出LLVM IR 中间表示。

`is_IR`用于判断命令行参数里是否含有`-IR`，如果有，那么需要执行手工构建的IR Module。然后LLVM 初始化本地目标程序以及本地LLVM IR的printer函数。将生成的LLVM IR写入`_M`中。最后查看`_OutFile`参数，如果为空那么结果直接输出，否则输出到文件中。



## 实验设计

### task1

本关需要掌握如何将C语言代码转换成对应的IR。

首先研究给出的样例，学习了基本指令和函数调用的写法，完成了前面几个手工编写的任务。在最后一个io相关的.ll文件编写时遇到了一点困难，因为涉及到调用外部函数，于是参考了`clang -S -emit-llvm`的输出后进行编写。



### task2

本关需要给SysYF程序手工编写C++代码，以生成并输出和sy文件功能相同的ll文件。

同样是参考demo就可以写出前几个程序。在编写`io_gen.cpp`的时候也对如何调用外部函数感到了困惑，但是在参考助教在`io_gen.cpp`中给出的类似“声明”外部函数的语句`getint = scope->find("getint", true)`，我发现声明之后调用该函数就和普通的函数一样即可。

编写`qsort_gen.cpp`的过程类似于将前面各个功能模块（赋值语句，循环语句，条件语句，函数调用，io等）整合起来，难度不大，但比较繁琐。



### task3

本关任务是根据 `LLVM IR API` 提供的中间表示构建方式，构建快速排序（QuickSort）算法的 `LLVM IR` 代码，并在`driver`中加载`LLVM IR`程序，通过`JIT`执行。

1. 编写`include/IR/qs_generator.hpp`.

   我参考了给出的样例，发现和task2的原理相差不大，所以就用了和task2类似的思路，将助教给出的quicksort程序编写成对应的.hpp文件。

2. 在`main.cpp`中调用该实现获得`IR Module`并执行

   仿照调用gcd的方式，更改名称即可。

3. 在`task3`中的`src/runtime/runtime.cpp`中，补全输入输出函数以及计时函数在`LLVM IR Module`中的声明，以使得`DynamicLibrary`可以将这些函数都加入到符号表中。

   仿照示例，在`runtime.h`中对应的每个参数按照返回值和类型加入即可。

## 实验难点及解决方案

因为不会高级的debug方法，只会把结果打印出来debug，于是在这方面花费了大量时间。

因为实验的一些语句重复性比较高，所以我在实验中经常直接复制相似的语句，更改其中的“关键词”。但是，在第二关和第三关都遇到了把Add指令复制之后没有改成Sub，导致越界出错的情况。在第二关中，我将生成的LLVM IR打印出来然后运行，在这个代码里面通过输出一些变量，找出了错误。再根据逻辑更改用于生成它的.cpp文件。但是第三关我一开始不知道如何生成LLVM IR，于是只能在.hpp文件里勉强增加一些输出语句，这样效率很低。后来发现更改命令行参数就可以输出LLVM IR中间代码，于是用了和第二关同样的方法找出了bug。

实验三还遇到了不知道如何处理`const int`的情况，定义的语句没有出现问题，但是看了很久也不知道该如何在后面使用它。于是定义数组的时候，只好直接使用4096这个值来代替原本的`buf_size`了。

## 实验总结

本次实验熟悉了中间代码的生成过程，让我觉得这是一份有些繁琐但也很神奇很有意思的工作。也让我意识到写实验的时候需要细致耐心，否则一点小问题都会花很多时间来找bug。

## 实验反馈

我认为task2和task3的工作稍有重复，task3的内容很大一部分似乎是把task2的思路借鉴过来然后改一些接口，希望这部分的工作量可以减小一些。