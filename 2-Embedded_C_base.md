# 嵌入式C语言基础

## 关键字

关键字 >> 预定义字符串

共**32**个关键字

### sizeof

```c
int a;
printf("the a is %lu\n", sizeof(a));
```

注意：sizeof并不是函数，而是个关键字

sizeof是编译器给我们查看内存空间容量的一个工具

### return

返回

在linux中return 0 表示正常返回

### 数据类型

嵌入式C语言操作的对象为：资源/内存（内存类型的资源，LCD缓存、LED灯）

数据类型所占内存的大小与编译器有关

```c
int a;
sizeof(a) = 4;
```

sizeof(a)并不是定值，数据类型的大小由编译器决定

| 类型  | 字节   |
| ----- | ------ |
| char  | 1      |
| int   | 4      |
| long  | 4 /? 8 |
| short | 2      |

#### char

硬件芯片操作的最小单位：

 **1 bit**

软件操作的最小单位：

    char a;

    **1Byte**(8 bit)

应用场景：

    硬件处理的最小单位

 **char buf[xx];**

char陷阱：

一个char是8个bit，即一个字节

即一个char最大为256

**char a = 300;	则a是谁？**

char类型溢出问题 => char = 44（300-256）

#### int

int的大小：

    根据编译器来决定

编译器最优处理大小：

    系统一个周期，所能接受的最大处理单位，为int

例如32bit系统 => 4B => 一个int的大小

例如16bit单片机 => 2B => 一个int的大小

#### 进制表示

    十进制	D

    八进制	O`int a = 010;	//识别为八进制`

    十六进制  H`int a = 0x10;	//识别为十六进制`

    二进制	B

#### long short

特殊长度的限制符才用到，具体看编译器

#### signed

无符号	unsigned		代表数据

有符号	signed		代表数字

**是否将该数据类型的最高位作为符号位**

unsigned int a;

signed int a;

默认处理是signed int;

尽量写为unsigned int全称

#### float double

大小

float  4B

double  8B

1.0  1.1  浮点类型常量是使用double格式

1.0f  1.1f  定义该常量为float格式

#### void

void是一种声明标志，语义符

### 自定义数据结构

#### struct

结构体类型，按照定义顺序去开辟空间，头尾相接开辟空间（累加）

```c
struct parameterName{
	unsigned int a;
	unsigned int b;
	unsigned int b;
	unsigned int d;
};
```

#### union

共用体类型，**共用起始地址**，然后开辟空间（重叠）

```c
union parameterName{
	char a;
	unsigned int b;
};
```

#### enum

枚举类型(enumerate)，里面是**逗号分隔**

**枚举变量的大小就是一个整数的大小**

```c
enum parameterName{
};

enum week{
	Monday = 0, Tuesday = 1, Wednesday = 2,
	Thursday, Friday,
	Saturday, Sunday
};
```

enum关键词地位比较低，使用率比较小

**不赋值则继承前面+1**

**默认第一个值为0**

#### typedef

数据类型的别名

```c
len_t a = 170;
time_t b = 3600;
```

`typedef int a;		//a是int的别名`

`xxx_t `往往表示typedef定义的别名

例如：`time_t`	 `len_t`

### 类型修饰符

数据类型 >> 定义资源的大小

类型修饰符 >> 对资源属性中位置的限定（定义只读段，定义只写）

#### auto

`char a` 等价于 `auto char a`

默认情况下都会实现的关键字

分配至可读可写区域

**如果在{}中，则分配至栈空间**

#### register

`register int a` 与 `auto int a` 的区别

register限制变量定义在寄存器上的修饰符

将变量分配到寄存器中

**使用频繁的变量，直接分配到寄存器中，不需要频繁从内存中读入到CPU中**

定义一些频繁快速访问的变量

注意：但是，编译器仅仅会尽量安排CPU的寄存器去存放这个变量，如果寄存器不足，则还是将变量分配到了内存中。

**&（取地址符号）对register不起作用**

#### static

静态变量

应用场景：

    修饰3种数据；

    1、函数内部的变量

```c
int fun(){
	static int a;
}
```

    2、函数外部的变量（全局变量）

```c
static int a;
int fun(){

}
```

    3、修饰函数

```c
static int fun();
```

#### const

常量的定义

只读变量（是可以通过一些方法改变的）

`const int a = 100;`

const会把变量分配到内存的R区

#### extern

外部申明

#### volatile

告知**编译器编译方法**的关键字，不优化编译（与地址关系很大）

修饰变量的值的修改，不仅仅可以通过软件，也可以通过其他方式（硬件外部的用户）

```c
int a = 100;
while(a == 100);
mylcd();

-------------------------------------
[a]：代表a的地址

f1:
LDR R0, [a]	//这里并没有必要每次循环都读入R0
f2:
CMP R0, #100
f3:
JMPEQ f1	//如果编译器打开优化选项，则变为JMPEQ f2
f4:
J mylcd 
```

如果不用volatile修饰，则编译器自动优化到f2

如果使用volatile修饰，则编译器会编译到f1，即你定义的a会因为外部因素发生变化，这个时候就不要让编译器优化到f2

## 运算符

### 算术运算符

加减乘除很简单，没什么好说的，尽量保存运算对象类型一致即可

```c
int a = b * 10	//CPU要多个周期进行
int a = b + 10	//CPU一个周期可以处理
//在CPU层面执行的是不同的
```

这里学过计算机组成原理很好理解

%取模运算(求余)

0 % n = 0

n % m = res  res属于[0 ~ m-1]

用途：

1.取一个范围的数

我有一个随机数，但是我只想要1 - 100以内的值，则(n % 100) + 1，结果就是1 ~ 100

2.得到M进制的一个个位数

D % 10

B % 2

H % 16

O % 8

3.循环数据结构的下标


### 逻辑运算

结果不是True 就是 False

返回结果就是1 或者 0

**0就是假，其余都是真（负数也是真）**

注意：`int f = -1`  `f是True`

A || B 与 B || A是不同的（**短路**效应）（&&同理)

||  &&  ！属于逻辑层面（逻辑与或非）


### 位运算

#### 移位运算

左移 << ： 相当于乘以2 （二进制下的移位），溢出补0

`m << 1`   等价于  m * 2

右移 >> ：相当于除以2，右移与符号变量有关

如果是**有符号数**，则为**算术右移**

如果是**无符号数**，则为**逻辑右移**

使用移位操作进行乘除法只会使用CPU一个周期，乘除法尽量用移位操作


#### 按位与或非

按位与或非异或是& | ~ ^


&按位与

& 屏蔽：

`a & 0 ----> 0`

`a & 0xff00;	屏蔽低八位`

& 取出：

`a & 1 ----> a`

`a & 0xff00;	取出高八位`

&一般在硬件中作为清零器


|按位或

| 保留：

`a | 0 ----> a`

| 置1：

`a | 1 ----> 1`

| 一般在硬件中作为设置器


例：

设置一个资源的bit5为高电平，其他位不变

`int a;`

`伪代码：a = a | 1 0 0 0 0`

`a = a | (0x1 << 5);	移位和按位运算结合`

清除第五位，其他位不变

`a = a & ~(0x1 << 5);`

**置1用| 清0用&**


^按位异或

相同位0，相反为1

算法： AES  SHA1

例：

使用异或交换两个数，不引入第三个变量

```c
int func(){
	int a = 20;
	int b = 30;
	a = a ^ b;
	b = a ^ b;
	a = a ^ b;
}
```


~按位取反

**注：~按位取反，!逻辑非**