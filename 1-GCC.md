# GCC

## GCC基础

GCC >> GNU C Compiler  >> GNU编译器

G >> 开源社区

后来发展为 GNU Compiler Collection

查看版本(版本不是越新越好)

```shell
gcc -v
```

组织结构模糊化 -o  >> output

模糊化会使gcc根据后缀名去匹配翻译

**-o 和输出文件名之间不能带任何其他选项**

```shell
gcc -o outputFile inputFile
```

### helloworld

在linux中**return 0 表示返回成功**，其余均为返回失败

```c
#include <stdio.h>
int main(){
	printf("hello world!\n");

	return 0;
}
```

```shell
gcc -o build helloworld.c
```

在linux中运行指令为

```shell
./build
```

-v打印出gcc具体工作的所有信息

```shell
gcc -v -o build build helloworld.c
```

## gcc编译过程

gcc -v -o outputFile inputFile后信息

### 编译

```shell
/usr/lib/gcc/x86_64-linux-gnu/7/cc1 -o build.s helloworld.c
```

cc1程序将.c文件编译为.s文件

```shell
/usr/lib/gcc/x86_64-linux-gnu/7/cc1 -quiet -v -imultiarch x86_64-linux-gnu helloworld.c -quiet -dumpbase helloworld.c -mtune=generic -march=x86-64 -auxbase helloworld -version -fstack-protector-strong -Wformat -Wformat-security -o /tmp/ccVupwJK.s
```

等价于

```shell
gcc -S -o xxx xxx
```

### 汇编

将编译生成的.s文件作为输入

```shell
as -v --64 -o /tmp/cc9Ejhq2.o /tmp/ccVupwJK.s
```

等价于

```shell
as -o a.o a.s
```

等价于

```shell
gcc -c
```

如果输入

```shell
gcc -c -o build.o helloworld.c
```

因为是从.c开始的，因此会先调用编译器cc1再调用as的汇编器

### 链接

```shell
/usr/lib/gcc/x86_64-linux-gnu/7/collect2 -o build helloworld.o 加上 ....一堆其余的东西
```

等价于

```shell
gcc -o 
```

如果输入文件是.o那么直接进行链接

如果输入文件是.c那么则先调用cc1再调用as再调用collect2，同上

### 预处理

```shell
cpp -o a.i helloworld.c
```

等价于

```shell
gcc -E a.i helloworld.c
```

预处理指令有

#include

#define

#类指令在预处理阶段进行编译

.i的内容最后是我们原来写的代码

预处理过程更像是替换，只会替换#命令内容

### 总结

```shell
gcc -o build helloworld.c
```

执行上述指令的过程为：

gcc -S  ===> .S文件

gcc -c  ===> .o文件

gcc -o  ===> 可执行文件

【问题】define include这些是关键字吗？

define和include是在**预处理**阶段进行的，并不是编译过程中的，因此并不是关键字！

int define;

int include;

是可以通过编译的！

## C语言常见错误

```c
#include "name"		//在当前目录下开始寻找
#include <name>		//在系统环境变量中寻找
```

### 头文件错误

No such file or directory

解决方案

```shell
gcc -I查找头文件的目录
```

例如：

```shell
gcc -I./inc -o build helloworld.c
```

### 编译错误

    语法错误： ; {} ()之类的

### 链接错误

    原材料不够：

```c
#include "stdio.h"
#include "abc.h"

void fun(void);

int main(){
	int a = ABC;
	printf("hello world!\n");

	fun();
	return 0;
}
```

这段指令是可以通过编译的，语法层面没有错误

但是无法通过链接，错误为：

```shell
undefined reference to `fun`
collect2: ld returned 1 ......
```

collect2就是上一节提到的，编译过程执行的程序！

解决方案1：不声明了，实现

```c
#include "stdio.h"
#include "abc.h"
void fun(void){

}

int main(){
	int a = ABC;
	printf("hello world!\n");    fun();
	return 0;
}
```

解决方案2：再定义一个.c，多个打包在一起

```shell
gcc -o build 001.c abc.c
```

更推荐这样写（逻辑上从原材料到主体）

```shell
gcc -c -o a.o 001.c
gcc -c -o b.o abc.c
```

再生成主体

```shell
gcc -o build a.o b.o
```

原材料多了：

    定义了两个fun函数，则编译器不知道用哪个

    multiple definition of 'fun'	过多定义

   多次实现了标签，最后只保留一个

## 预处理高级使用

预处理能认识的东西基本都是#

#include	头文件

#define	宏定义

    #define的最终目的就是替换

*#define 宏名 宏体*

```c
#define ABC 5+3  
printf("the %d\n", ABC*5);
```

ABC*5并不是我们期待的值（因为替换）因此，使用 宏体尽量加入()`#define ABC (5+3)`

`#define ABC(x) ((5+x))`

根据输入的x定替换的值

```c
ABC(3) => ((5+3))
ABC(1) => ((5+1))
```

### 预定义宏

编译器已经设计好了，与操作系统关系不大，可以直接使用

```c
__FUNCTION__：函数名
__LINE__：行号
__FILE__：文件名
```

```c
#include <stdio.h>


int main(){
        printf("the %s, %s, %d", __FUNCTION__, __FILE__, __LINE__);
        return 0;
}
```

```shell
# ./a
the main, 001.c, 5
```

### 条件编译

#ifdef

#else

#endif


调试版本，发行版本，在同一个代码文件中

```c
#include <stdio.h>

int main(){
	//第一行为调试信息
	printf("=======%s=========\n", __FILE__);
	printf("hello world!\n");
	return 0;
}
```

我们让第一行在调试时显示，在运行时不显示

如果ABC没有定义则不执行#ifdef与#endif中的代码，在预处理阶段就不会放入.i文件中

```c
#include <stdio.h>

int main(){
#ifdef ABC
	printf("=======%s=========\n", __FILE__);
#endif
	printf("hello world!\n");
	return 0;
}
```

使用 `gcc -E -o a.i a.c` 查看a.i的底部

```c
int main(){

 printf("hello world!\n");
 return 0;
}
```

#### 推荐方法

`gcc -D`

在预处理前编译器加入#define语句

    gcc -DABC1  ==> #define ABC1

则实现了条件开关

`gcc -DABC -o build a.c`

实现了上述代码开关的打开，即调试模式


#### 宏展开#与##

#字符串化

##连接符号


#define ABC(x) #x

ABC(a) 	=>  "a"

#define ABC(x) day##x

ABC(a)	=>  "daya"



**技巧性赋值**

例：

```c
#include <stdio.h>

#define ABC(x)	#x
#define DAY(x)	myday##x

int main(){
	int myday1 = 10;
	int myday2 = 20;
	printf(ABC(ab\n));

	printf("the day is %d\n", DAY(2));
	return 0;
}
```

输出：

```shell
ab
the day is 20
```
