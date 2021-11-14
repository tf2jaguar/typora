[toc]

不学习底层知识可能不会阻碍你称为一个称职的程序员，但也许会阻碍你成为一个优秀的程序员。我所理解的底层知识，是指编程或开发所依赖的平台(或者框架、工具)的知识。对于 Java 开发者来 说，虚拟机、字节码就是其底层知识。

# 1. Hello, World

这篇文章我们以输出 "Hello, World" 来开始字节码之旅，如果之前没有怎么接触过字节码的话，这篇文章应该能够让你对字节码有一个最基本的认识。 

## 0x01 java 文件如何变成 .class文件

新建一个 Hello.java 文件，源码如下:

```java
public class Hello {
	public static void main(String[] args) {
		System.out.println("Hello, World"); 
  }
}
```

Java 从源文件到执行的过程如下图所示

<img src="./pic/JVM字节码从入门到精通/image-20211114091656737.png" alt="image-20211114091656737" style="zoom:50%;" />

JDK 工具 javac(java 编译器)帮我们完成了源文件编译成 JVM 可识别的 class 文件的工作。 在命令行中执行javac Hello.java，可以看到生成了 Hello.class 文件。用xxd 命令以 16 进制的方式查看这个 class 文件。

```shell
xxd Hello.class

00000000: cafe babe 0000 0034 0022 0a00 0600 1409 .......4."...... 
00000010: 0015 0016 0800 170a 0018 0019 0700 1a07 ................ 
00000020: 001b 0100 063c 696e 6974 3e01 0003 2829 .....<init>...() 
00000030: 5601 0004 436f 6465 0100 0f4c 696e 654e V...Code...LineN 
00000040: 756d 6265 7254 6162 6c65 0100 124c 6f63 umberTable...Loc 
00000050: 616c 5661 7269 6162 6c65 5461 626c 6501 alVariableTable. 
00000060: 0004 7468 6973 0100 074c 4865 6c6c 6f3b ..this...LHello;
```

## 0x02 魔数 0xCAFEBABE

<img src="pic/JVM字节码从入门到精通/image-20211114092002409.png" alt="image-20211114092002409" style="zoom:50%;" />

class 文件的头四个字节称为魔数(Magic Number)，可以看到 class 的魔数为 0xCAFEBABE。很多文件都以魔数来进行文件类型的区分，比如 PDF 文件的魔数是%PDF-(16进制0x255044462D)，png 文件的 魔数是\x89PNG(0x89504E47)。文件格式的制定者可以自由的选择魔数值，只要魔数值还没有被广泛的采用过且不会引起混淆即可。

Java 早期开发者选用了这样一个浪漫气息的魔数，高司令有解释这一段 轶事。这个魔数值在 Java 还成为 Oak 语言的时候就已经确定下来了。

这个魔数是 JVM 识别 .class 文件的标志，虚拟机在加载类文件之前会先检查这四个字节，如果不是 0xCAFEBABE 则拒绝加载该文件，更多关于字节码格式的说明，我们会在后面的文章中慢慢介绍。

## 0x03 javap 详解

类文件是二进制块，想直接与它打交道比较艰难，但是很多情况下我们必须理解类文件。比如服务器上的接又出了 bug，重新打包部署以后问题并没有解决，为了找出原因你可能需要看一下部署以后的 class 文件究竟是不是我们想要的。还有一种情况跟你合作的开发商跑路了，只给你留下一堆编译过的代码，没有源代码，当出 bug 时我们需要研究这些类文件，看问题出在哪里。 好在 JDK 提供了专门 用来分析类文件的工具:javap，用来方便的窥探 class 文件内部的细节。javap 有比较多的参数选项，其中-c -v -l -p -s 是最常用的。

```she
Usage: javap <options> <classes> where possible options include:

-help --help -? 								Print this usage message
-version 												Version information
-v -verbose 										Print additional information
-l 															Print line number and local variable tables
-public 												Show only public classes and members
-protected 											Show protected/public classes and members
-package 												Show package/protected/public classes and members (default)
-p -private 										Show all classes and members
-c 															Disassemble the code
-s 															Print internal type signatures
-sysinfo 												Show system info (path, size, date, MD5 hash) of class being processed
-constants 											Show final constants
-classpath <path> 							Specify where to find user class files
-cp <path> 											Specify where to find user class files
-bootclasspath <path> 					Override location of bootstrap class files
```

### -c  选项

最常用的选项是 -c，可以对类进行反编译，执行 javap -c Hello 的输出结果如下：

```shell
Compiled from "Hello.java" 
public class Hello {
	public Hello(); 
		Code:
				0: aload_0 
				1: invokespecial #1 		// Method java/lang/Object."<init>":()V
		  	4: return
	public static void main(java.lang.String[]);
		Code:
				0: getstatic #2 				// Field java/lang/System.out:Ljava/io/PrintStream; 
				3: ldc #3 							// String Hello, World 
				5: invokevirtual #4 		// Method java/io/PrintStream.println:(Ljava/lang/String;)V 
				8: return
}
```

上⾯代码前⾯的数字表⽰从⽅法开始算起的字节码偏移量 

- 3 ~ 7 ⾏：可以看到虽然没有写 Hello 类的构造器函数，编译器会⾃动加上⼀个默认构造器函数 
- 5 ⾏：aload_0 这个操作码是 aload_x 格式操作码中的⼀个。它们⽤来把对象引⽤加载到操作数栈。 x 表⽰正在被访问的局部变量数组的位置。在这⾥的 0 代表什么呢？我们知道⾮静态的函数都有第 ⼀个默认参数，那就是 this，这⾥的 aload_0 就是把 this ⼊栈 
- 6 ⾏：invokespecial #1，invokespecial 指令调⽤实例初始化⽅法、私有⽅法、⽗类⽅法，#1 指的是常量池中的第⼀个，这⾥是⽅法引⽤java/lang/Object."<init>":()V，也即构造器函数 
- 7 ⾏：return，这个操作码属于 ireturn、lreturn、freturn、dreturn、areturn 和 return 操作码组中的⼀员，其中 i 表⽰ int，返回整数，同类的还有 l 表⽰ long，f 表⽰ float，d 表⽰ double，a 表⽰ 对象引 ⽤。没有前缀类型字母的 return 表⽰返回 void 

到此为⽌，默认构造器函数就讲完了，接下来，我们来看 9 ~ 14 ⾏的 main 函数 

- 11 ⾏：getstatic #2，getstatic 获取指定类的静态域，并将其值压⼊栈顶，#2 代表常量池中的第 2 个，这⾥表⽰的是java/lang/System.out:Ljava/io/PrintStream;，其实就是java.lang.System 类的静态 变量 out（类型是 PrintStream） 
- 12 ⾏：ldc #3、，ldc ⽤来将常量从运⾏时常量池压栈到操作数栈，#3 代表常量池的第三个（字符串 Hello, World） 
- 13 ⾏：invokevirtual #4，invokevirutal 指令调⽤⼀个对象的实例⽅法，#4 表⽰ PrintStream.println(String) 函数引⽤，并把栈顶两个元素出栈

### -p 选项

默认情况下，javap 会显⽰访问权限为 public、protected 和默认（包级 protected）级别的⽅法，加上 -p 选项以后可以显⽰ private ⽅法和字段

### -v 选项 

javap 加上 -v 参数的输出更多详细的信息，⽐如栈⼤⼩、⽅法参数的个数。

```shell
public Hello(); 
		stack=1, locals=1, args_size=1

public static void main(java.lang.String[]); 
		stack=2, locals=1, args_size=1
```

为什么Hello() 和main()的args_size都等于1呢？明明 Hello 的构造器函数没有参数的呀？ 对于⾮静态函数，this对象会作为函数的隐式第⼀个参数，所以 Hello() 的args_size=1 对于静态main函数，不需 要this对象，它的参数就是String[] args这个数组，也等于1

### -s 选项

javap 还有⼀个好⽤的选项 -s，可以输出签名的类型描述符。我们可以看下 Hello.java 所有的⽅法签名

```shell
javap -s Hello 

Compiled from "Hello.java" 
public class Hello {
	public Hello();
		descriptor: ()V
	public static void main(java.lang.String[]);
		descriptor: ([Ljava/lang/String;)V 
}
```

可以看到 main 函数的⽅法签名是 ([Ljava/lang/String;)V。JVM 内部使⽤⽅法签名与我们⽇常阅读的⽅法签名不太⼀样，但是后⾯会频繁遇到，主要分为两部分字段描述符和⽅法描述符。 字段描述符（Field Descriptor），是⼀个表⽰类、实例或局部变量的语法符号，它的表⽰形式是紧凑的，⽐如 int 是⽤ I 表⽰的。完整的类型描述符如下表

| 描述符       | 类型                                         |
| ------------ | -------------------------------------------- |
| B            | byte                                         |
| C            | char                                         |
| D            | double                                       |
| F            | float                                        |
| I            | int                                          |
| J            | long                                         |
| L Classname; | 引用类型（比如Ljava/lang/String;用于字符串） |
| S            | short                                        |
| Z            | boolean                                      |
| [            | Array-of                                     |

⽅法描述符（Method Descriptor）表⽰⼀个⽅法所需参数和返回值信息，表⽰形式为( ParameterDescriptor* ) ReturnDescriptor 。 ParameterDescriptor 表⽰参数类型，ReturnDescriptor表⽰返回值信息， 当没有返回值时⽤V表⽰。⽐如⽅法Object foo(int i, double d, Thread t) 的描述符为(IDLjava/lang/Thread;)Ljava/lang/Object;

<img src="pic/JVM字节码从入门到精通/image-20211114143304360.png" alt="image-20211114143304360" style="zoom:50%;" />

## 0x04 小结

这篇⽂章讲解了⼀个输出 "Hello, World" 的字节码的细节，⼀起来回顾⼀下要点：

- 第⼀，class ⽂件的魔数是具有浪漫⽓息的 0xCAFEBABE； 
- 第⼆，我们讲解了字节码分析的利器 javap 的各个参数详细的⽤法；
- 第三，讲解了字段描述符与⽅法描述符在 JVM 层⾯的表⽰规则，⽅便我们后⾯⽂章的理解。

## 0x05 思考

最后，给你留⼀个思考题，javap 的 -l 参数有什么⽤？ 欢迎你在留⾔区留⾔，和我⼀起讨论。

---

# 2. jvm的运行

## 0x01 虚拟机：stack based vs register based

<img src="pic/JVM字节码从入门到精通/image-20211114143935360.png" alt="image-20211114143935360" style="zoom:50%;" />

虚拟机常见的实现⽅式有两种：Stack based 的和 Register based。⽐如基于 Stack 的虚拟机有Hotspot JVM、.net CLR，这种基于 Stack 实现虚拟机是⼀种⼴泛的实现⽅法。⽽基于 Register 的虚拟机有 Lua 语 ⾔虚拟机 LuaVM 和 Google 开发的安卓虚拟机 DalvikVM。 

两者有什么不同呢？举⼀个计算两数相加的例⼦：c = a + b 基于 HotSpot JVM 的源码和字节码如下

源码 

```java
void bar(int a, int b) { 
  int c = a + b; 
}
```

对应字节码

```shell
0: iload_1 	// 将 a 压⼊操作数栈 
1: iload_2 	// 将 b 压⼊操作数栈 
2: iadd 		// 将栈顶两个值出栈，相加，然后将结果放回栈顶 
3: istore_3 // 将栈顶值存⼊局部变量表中第 3 个 slot 中
```

基于寄存器的 LuaVM 的 lua 源码和字节码如下，查看字节码使⽤ luac -l -l -v -s test.lua 命令

源码 

```lua
local function my_add(a, b) 
  return a + b; 
end
```

对应字节码 

```shell
1 [3] ADD			2 0 1
```

基于寄存器的 add 指令直接把寄存器 R0 和 R1 相加，结果保存在寄存器 R2 中。 



基于栈和寄存器的指令集各有优缺点，基于栈的指令集移植性更好，代码更加紧凑、编译器实现更加简单，但完成相同功能所需的指令数⼀般⽐寄存器架构多，需要频繁的⼊栈出栈，栈架构指令集的执 ⾏速度会相对⽽⾔慢⼀些。

为了理解字节码的细节，我们需要详细了解字节码的执⾏过程。众所周知，Hotspot JVM 是⼀个基于栈的虚拟机，每个线程都有⼀个虚拟机栈，存储了「栈帧」。每次⽅法调⽤都伴随着栈帧的创建销 毁。

## 0x02 栈帧

栈帧（Stack Frame）是⽤于⽀持虚拟机进⾏⽅法调⽤和⽅法执⾏的数据结构 栈帧随着⽅法调⽤⽽创建，随着⽅法结束⽽销毁，栈帧的存储空间分配在 Java 虚拟机栈中，每个栈帧拥有⾃⼰的局部变量表 （Local Variables）、操作数栈（Operand Stack） 和 指向运⾏时常量池的引⽤

<img src="pic/JVM字节码从入门到精通/image-20211114144544496.png" alt="image-20211114144544496" style="zoom:50%;" />

### 局部变量表

每个栈帧内部都包含⼀组称为局部变量表（Local Variables）的变量列表，局部变量表的⼤⼩在编译期间就已经确定。Java 虚拟机使⽤局部变量表来完成⽅法调⽤时的参数传递，当⼀个⽅法被调⽤时，它 的参数会被传递到从 0 开始的连续局部变量列表位置上。当⼀个实例⽅法（⾮静态⽅法）被调⽤时，第 0 个局部变量是调⽤这个实例⽅法的对象的引⽤（也就是我们所说的 this ）

<img src="pic/JVM字节码从入门到精通/image-20211114154220381.png" alt="image-20211114154220381" style="zoom:50%;" />

### 操作数栈

每个栈帧内部都包含了⼀个称为操作数栈的后进先出（LIFO）栈，栈的⼤⼩同样也是在编译期间确定。Java 虚拟机提供的⼀些字节码指令⽤来从局部变量表或者对象实例的字段中复制常量或者变量到操 作数栈，也有⼀些指令⽤于从操作数栈取⾛数据、操作数据和把操作结果重新⼊栈。在⽅法调⽤时，操作数栈也⽤来准备调⽤⽅法的参数和接收⽅法返回的结果。

⽐如 iadd 指令⽤来将两个 int 类型的数值相加，它要求执⾏之前操作数栈已经存在两个由前⾯其它指令放⼊的 int 型数值，在 iadd 指令执⾏时，两个 int 值从操作数栈中出栈，相加求和，然后将求和的结 果重新⼊栈。

⽐如 1 + 2 这样的指令执⾏过程如下

<img src="pic/JVM字节码从入门到精通/image-20211114154739107.png" alt="image-20211114154739107" style="zoom:50%;" />

整个 JVM 指令执⾏的过程就是局部变量表与操作数栈之间不断 load、store 的过程

<img src="pic/JVM字节码从入门到精通/image-20211114154812393.png" alt="image-20211114154812393" style="zoom:50%;" />

我们再来看⼀个稍微复杂⼀点的例⼦

```java
public class ScoreCalculator {
	public void record(double score) { 
  }
	public double getAverage() { 
    return 0; 
  }
} 

public static void main(String[] args) { 
  ScoreCalculator calculator = new ScoreCalculator();
	
  int score1 = 1; 
  int score2 = 2;
	
  calculator.record(score1); 
  calculator.record(score2);
  
	double avg = calculator.getAverage();
}
```

javap 查看字节码输出如下

```shell
public static void main(java.lang.String[]); 
descriptor: ([Ljava/lang/String;)V 
flags: ACC_PUBLIC, ACC_STATIC 
Code:
	stack=3, locals=6, args_size=1 
	0: new #2 								// class ScoreCalculator 
	3: dup 
	4: invokespecial #3 			// Method ScoreCalculator."<init>":()V 
	7: astore_1

	8: iconst_1 
	9: istore_2
	
	10: iconst_2 
	11: istore_3
	
	12: aload_1 
	13: iload_2 
	14: i2d 
	15: invokevirtual #4			// Method ScoreCalculator.record:(D)V

	18: aload_1 
	19: iload_3 
	20: i2d 
	21: invokevirtual #4			// Method ScoreCalculator.record:(D)V

	24: aload_1 
	25: invokevirtual #5 			// Method ScoreCalculator.getAverage:()D
	28: dstore 4

	30: return
```

- 0 ~ 7：新建了⼀个 ScoreCalculator 对象，使⽤ astore_1 存储在局部变量 calculator 中：astore_1 的含义是把栈顶的值存储到局部变量表下标为 1 的位置上，这⾥为什么会有⼀个 dup，我们后⾯会讲到 
- 8 ~ 11：iconst_1 和 iconst_2 ⽤来将整数 1 和 2 加载到栈顶，istore_2 和 istore_3 ⽤来将栈顶的元素存储到局部变量表 2 和 3 的位置上 
- 12 ~ 15：可以看到 *store* 指令会把栈顶元素移除，所以下次我们要⽤到这些局部变量时，需要使⽤ load 命令重新把它加载到栈顶。⽐如我们要执⾏calculator.record(score1)，对应的字节码如 下

> 12: aload_1 
>
> 13: iload_2 
>
> 14: i2d 
>
> 15: invokevirtual #4 			// Method ScoreCalculator.record:(D)V
>
> 可以看到 aload_1 先从局部变量表中 1 的位置加载 calculator 对象，iload_2 从 局部变量表中 2 的位置加载⼀个整型值，i2d 这个指令⽤来将整型值转为 double 并将新的值重新⼊栈，到⽬前为⽌参数全部就 绪，可以⽤ invokevirtual 执⾏⽅法调⽤了

- 24 ~ 28：同样是⼀个普通的⽅法调⽤，流程还是先 aload_1 加载 calculator 对象，invokevirtual 调⽤ getAverage ⽅法，并将 栈顶元素存储到局部变量表下标为 4 的位置上 有⼀点需要注意的是 javap 输出的 locals=6，但是我们⽬前看到的局部变量只有args、calculator、score1、score2、avg这 5 个，为什么这⾥等于 6 呢？这是因为 avg 为 double 型变量，需要两个槽位 （slot） 整个过程局部变量表如下图所⽰

![image-20211114155744056](pic/JVM字节码从入门到精通/image-20211114155744056.png)

其实局部变量表可以通过 javap ⽤ -l 参数直接输出，但是我们⽤ javap -v -p -l MyLocalVariableTest 并没有输出任何局部变量表相关的信息。这是因为默认情况下局部变量表属于调试级别的信 息，javac 编译的时候并没有编译进字节码，我们可以加上 javac -g ⽣成字节码的时候同时⽣成所有的调试信息，如下所⽰

```shell
javac -g MyLocalVariableTest.java 
javap -v -p -l MyLocalVariableTest 

LocalVariableTable:
Start Length Slot Name Signature
		0 		31 		0 args [Ljava/lang/String; 
		8 		23 		1 calculator LScoreCalculator; 
		10 		21 		2 score1 	I 
		12 		19 		3 score2 	I 
		30 		 1 		4 avg 		D
```

## 0x03 从⼆进制看 class ⽂件和字节码

```java
public class Get { String name;
	public String getName() { 
	  return name; 
	}
}
```

javap 查看字节码如下

```shell
public java.lang.String getName(); 
descriptor: ()Ljava/lang/String; 
flags: ACC_PUBLIC 
Code:
	stack=1, locals=1, args_size=1 
	0: aload_0 
	1: getfield #2 / Field name:Ljava/lang/String; 
	4: areturn
```

![image-20211114160236099](pic/JVM字节码从入门到精通/image-20211114160236099.png)

直接从⼆进制来看下这个 class ⽂件 xxd Get.class

<img src="pic/JVM字节码从入门到精通/image-20211114160307987.png" alt="image-20211114160307987" style="zoom:50%;" />

我们可以⼿动⽤ 16 进制编辑器去修改这些字节码⽂件，只是⽐较容易出错，所以产⽣了⼀些字节码操作的⼯具，最出名的莫过于 ASM 和 Javassist。我们后⾯讲到软件破解的时候，会介绍直接修改字节 码和通过 ASM 动态修改字节码这两种⽅式

## 0x04 ⼩结

⼀起来回顾⼀下这篇⽂章的要点：

- 第⼀，基于栈和基于寄存器指令集的优劣势； 
- 第⼆，讲解了 JVM 栈帧的构成（局部变量表、操作数栈、指向运⾏时常量池的引⽤），顺带讲解了 javap -l 参数和其在局部变量表中的应⽤；
-  第三，从类⽂件⼆进制⾓度看字节码的实现，并引出 ASM 字节码改写技术。

## 0x05 思考

最后，给你留⼀道作业，⽤ Java 写⼀个简单的 class ⽂件解析⼯具⽀持输出函数列表。



---



# 3. 控制转移指令

控制转移指令，也就是 if-else、三⽬表达式、switch-case 背后的指令。

## 0x01 概述

根据字节码的不同⽤途，可以⼤概分为如下⼏类 

- 加载和存储指令，⽐如 iload 将⼀个整形值从局部变量表加载到操作数栈 
- 控制转移指令，⽐如条件分⽀ ifeq 
- 对象操作，⽐如创建类实例的指令 new 
- ⽅法调⽤，⽐如 invokevirtual 指令⽤于调⽤对象的实例⽅法 
- 运算指令和类型转换，⽐如加法指令 iadd 
- 线程同步，monitorenter 和 monitorexit 两条指令来⽀持 synchronized 关键字的语义 
- 异常处理，⽐如 athrow 显式抛出异常

## 0x02 控制转移指令

<img src="pic/JVM字节码从入门到精通/image-20211114160749418.png" alt="image-20211114160749418" style="zoom:50%;" />

控制转移指令根据条件进⾏分⽀跳转，我们常见的 if-then-else、三⽬表达式、for 循环、异常处理等都属于这个范畴。对应的指令集包括：

- 条件分⽀：ifeq、iflt、ifle、ifne、ifgt、ifge、ifnull、ifnonnull、if_icmpeq、 if_icmpne、if_icmplt, if_icmpgt、if_icmple、if_icmpge、if_acmpeq 和 if_acmpne。
- 复合条件分⽀：tableswitch、lookupswitch 
- ⽆条件分⽀：goto、goto_w、jsr、jsr_w、ret

看⼀个 for 循环的例⼦

```java
public class MyLoopTest { 
  public static int[] numbers = new int[]{1, 2, 3}; 
  public static void main(String[] args) { 
    ScoreCalculator calculator = new ScoreCalculator(); 
    for (int number : numbers) { 
      calculator.record(number); 
    } 
  } 
}
```

对应的字节码

```shell
public static void main(java.lang.String[]); 
Code:
	0: new 						#2 			// class ScoreCalculator 
	3: dup 
	4: invokespecial  #3 			// Method ScoreCalculator."<init>":()V 
	7: astore_1

	8: getstatic 			#4			// Field numbers:[I
 11: astore_2 
 12: aload_2 
 13: arraylength 
 14: istore_3 
 15: iconst_0 
 16: istore					4

 18: iload 					4 
 20: iload_3 
 21: if_icmpge 			43 
 24: aload_2 
 25: iload 4				4 
 27: iaload 
 28: istore 				5
 30: aload_1 
 31: iload 					5 
 33: i2d 
 34: invokevirtual  #5			// Method ScoreCalculator.record:(D)V		
 37: iinc 					4, 1
 40: goto						18
 41: return
```

先把局部变量表的⽰意图放出来便于理解

![image-20211114161611284](pic/JVM字节码从入门到精通/image-20211114161611284.png)

- 0 ~ 7：new、dup 和⼀个 的 invokespecial 表⽰创建新类实例，下⼀节会重点介绍 
- 8 ~ 16：是初始化循环控制变量的⼀个过程。加载静态变量数组引⽤，存储到局部变量下标为 2 的位置上，记为$array，aload_2 加载 $array到栈顶，调⽤ arraylength 指令 获取数组长度存储到栈 顶，随后调⽤ istore_3 将数组长度存储到局部变量表中第 3 个位置，记为 $len。

Java 虚拟机指令集使⽤不同的字节码来区分不同的操作数类型，⽐如 iconst_0、istore_1、iinc、if_icmplt 都只针对于 int 数据类型。

同时注意到 istore_3 和 istore 4 使⽤了不同形式的指令类型，它们的作⽤都是把栈顶元素存⼊到局部变量表的指定位置。对于 3 采⽤了 istore_3，它属于 istore_<i>指令组，其中 i 只能是0 1 2 3 。其实 把istore_3 写成 istore 3 也能获取正确的结果，但是编译的字节码会变长，在字节码执⾏时也需要获取和解析 3 这个额外的操作数。

类似的做法还有 iconst_<i>，可以将 -1 0 1 2 3 4 5 压⼊操作数栈。这么做的⽬的是把使⽤最频繁的⼏个操作数融⼊到指令中，使得字节码更简洁⾼效

iconst_0 将整型值 0 加载到栈顶，随后将它存储到局部变量表第 4 个位置，记为 $i，写成伪代码就是

```
$array = numbers; 
$len = $array.arraylength 
$i = 0
```

- 18 ~ 34：是真正的循环体。⾸先加载 $i和 $len到栈顶，然后调⽤ if_icmpge 进⾏⽐较，如果 $i >= $len ，直接跳转到指令 43，也就是 return，函数结束。如果$i < $len ，执⾏循环体，加 载$array、$i，然后 iaload 指令把下标为 $i 的数组元素加载到操作数栈上，随后存储到局部变量表下标为 5 的位置上，记为$item。随后调⽤ invokevirtual 指令来执⾏ record ⽅法 
- 37 ~ 40：执⾏循环后的 $i ⾃增操作。

iinc 这个指令⽐较特殊，之前介绍的指令都是基于操作数栈来实现功能，iinc 是⼀个例外，它直接对局部变量进⾏⾃增操作，不要先⼊栈、加⼀、再出栈，因此效率⾮常⾼，适合循环结构。

这⼀段写成伪代码就是

```
@start: if ($i >= $len) return; 
$item = $array[$i] 
++ $i 
goto @start
```

整段代码是不是⾮常熟悉，看看下⾯这个代码

```
for (int i = 0; i < numbers.length; i++) { 
	calculator.record(numbers[i]); 
}
```

由此可见，for(item : array) 就是⼀个语法糖，javac 会让它现出原形，回归到它的本质

## 0x02 你不⼀定知道的 switch 底层实现

<img src="pic/JVM字节码从入门到精通/image-20211114162508242.png" alt="image-20211114162508242" style="zoom:50%;" />

如果让你来设计⼀个 switch-case 的底层实现，你会如何来实现？是⼀个个 if-else 来判断吗？ 实际上编译器将使⽤ tableswitch 和 lookupswitch 两个指令来⽣成 switch 语句的编译代码。为什么会有两个呢？ 这充分体现了效率上的考量。

```java
int chooseNear(int i) { 
	switch (i) { 
		case 100: return 0; 
		case 101: return 1; 
		case 104: return 4; 
		default: return -1; 
	} 
}
```

字节码如下：

```shell
0: iload_1 
1: tableswitch { // 100 to 104
				100: 36 
				101: 38 
				102: 42 
				103: 42 
				104: 40 
		default: 42 
}
36: iconst_0 		// return 0
37: ireturn 
38: iconst_1 		// return 1
39: ireturn 
40: iconst_4 		// return 4
41: ireturn 
42: iconst_m1 	// return -1
43: ireturn
```

细⼼的同学会发现，代码中的 case 中并没有出现 102、103，为什么字节码中出现了呢？ 编译器会对 case 的值做分析，如果 case 的值⽐较紧凑，中间有少量断层或者没有断层，会采⽤ tableswitch 来实现 switch-case，有断层的会⽣成⼀些虚假的 case 帮忙补齐连续，这样可以实现 O(1) 时间复杂度的查找：因为 case 已经被补齐为连续的，通过游标就可以⼀次找到。

伪代码如下

```java
int val = pop(); 								// pop an int from the stack
if (val < low || val > high) {  // if its less than <low> or greater than <high>, 
	pc += default; 								// branch to default
} else {												// otherwise
	pc += table[val - low]; 			// branch to entry in table
}
```

我们来看⼀个 case 值断层严重的例⼦

```java
int chooseFar(int i) { 
  switch (i) { 
    case 1: return 1; 
    case 10: return 10; 
    case 100: return 100; 
    default: return -1; 
  } 
}
```

对应字节码

```shell
0: iload_1 
1: lookupswitch { // 3
					1: 36 
				 10: 38 
				100: 41 
		default: 44 
}
```

如果还是采⽤上⾯那种 tableswitch 补齐的⽅式，就会⽣成上百个假 case，class ⽂件也爆炸式增长，这种做法显然不合理。lookupswitch应运⽽⽣，它的键值都是经过排序的，在查找上可以采⽤⼆分查找的 ⽅式，时间复杂度为 O(log n)

结论是：switch-case 语句 在 case ⽐较稀疏的情况下，编译器会使⽤ lookupswitch 指令来实现，反之，编译器会使⽤ tableswitch 来实现

### 补充内容：稀疏与否到底是什么意思

读者提问，下⾯的代码编译出的 switch-case 语句为什么采⽤了 lookupswitch，⽽不是 tableswitch，不是说「如果 case 的值⽐较紧凑，中间有少量断层或者没有断层，会采⽤ tableswitch 来实现 switchcase」吗？

```java
public static void foo() {
	int a = 0; 
  switch (a) { 
    case 0:
			System.out.println("#0"); 
      break; 
    case 1:
			System.out.println("#1"); 
      break; 
    default: 
      System.out.println("default"); 
      break;
	}
}
```

对应字节码

```shell
public static void foo(); 
	0: iconst_0 
	1: istore_0 
	2: iload_0 
	3: lookupswitch { // 2 
						0: 28 
						1: 39 
			default: 50 
	}
```



这个问题⽐较有意思，我调试了⼀下 javac 的源码，主要是 tableswitch 和 lookupswitch 代价的估算。

```c
hi=1 
lo=0 
nlabels = 2

// table_space_cost = 4 + (1 - 0 + 1) = 6 
long table_space_cost = 4 + ((long) hi - lo + 1); // words

// table_time_cost = 3 
long table_time_cost = 3; // comparisons

// lookup_space_cost = 3 + 2 * 2 = 7 
long lookup_space_cost = 3 + 2 * (long) nlabels;

// lookup_time_cost = 2 
long lookup_time_cost = nlabels;

// table_space_cost + 3 * table_time_cost = 6 + 3 * 3 = 15 
// lookup_space_cost + 3 * lookup_time_cost = 7 + 3 * 2 = 13 
// opcode = 15 <= 13 ? tableswitch : lookupswich

int opcode = nlabels > 0 && 
  table_space_cost + 3 * table_time_cost <= lookup_space_cost + 3 * lookup_time_cost 
  ? tableswitch : lookupswitch;
```

所以在 case 值只有 0， 1 两个的情况下，代价的计算是 table_space_cost + 3 * table_time_cost > lookup_space_cost + 3 * lookup_time_cost，lookupswich代价更⼩选 lookupswich 



如果有三个呢？

```c
hi=2 
lo=0 
nlabels = 3

// table_space_cost = 4 + (2 - 0 + 1) = 7 
long table_space_cost = 4 + ((long) hi - lo + 1); // words

// table_time_cost = 3 
long table_time_cost = 3; // comparisons

// lookup_space_cost = 3 + 2 * 3 = 9 
long lookup_space_cost = 3 + 2 * (long) nlabels;

// lookup_time_cost = 3 
long lookup_time_cost = nlabels;

// table_space_cost + 3 * table_time_cost = 7 + 3 * 3 = 16 
// lookup_space_cost + 3 * lookup_time_cost = 9 + 3 * 3 = 18 
// opcode = 16 <= 18 ? tableswitch : lookupswich

int opcode = nlabels > 0 &&
table_space_cost + 3 * table_time_cost &lt;= lookup_space_cost + 3 * lookup_time_cost
? tableswitch : lookupswitch;
```

所以在 case 值只有 0， 1，2 三个的情况下，代价的计算是 table_space_cost + 3 * table_time_cost < lookup_space_cost + 3 * lookup_time_cost，tableswitch 代价更⼩选 tableswitch 其实在数量极少的情况下，两个的差别不⼤，只是 javac 这⾥的算法导致选择了 lookupswitch

## 0x03 ⼩结

这篇⽂章以 for 和 switch-case 语句为例讲解了控制转移指令的实现细节，⼀起来回顾⼀下要点： 

- 第⼀，for(item : array) 语法糖实际上会改写为for (int i = 0; i < numbers.length; i++) 的形式；
- 第⼆，switch-case 语句 在 case 稀疏程度不同的情况下会分别采⽤ lookupswitch 和 tableswitch 指令来实现。

## 0x04 思考

最后，给你留⼀个道作业题，switch-case 语句⽀持枚举类型，你能通过分析字节码写出其底层的实现原理吗？

---

# 对象相关字节码

## 0x01 new, \<init> & \<clinit>

在 Java 中 new 是⼀个关键字，在字节码中也有⼀个指令 new。当我们创建⼀个对象时，背后发⽣了哪些事情呢？

```java
ScoreCalculator calculator = new ScoreCalculator();
```

对应的字节码如下：

```shell
0: new #2 					// class ScoreCalculator
3: dup 
4: invokespecial #3	// Method ScoreCalculator."<init>":()V

7: astore_1
```

⼀个对象创建的套路是这样的：new、dup、invokespecial，下次遇到同样的指令要形成条件反射。 



为什么创建⼀个对象需要三条指令呢？ 

⾸先，我们需要清楚类的构造器函数是以 <init>函数名出现的，被称为实例的初始化⽅法。调⽤ new 指令时，只是创建了⼀个类的实例，但是还没有调⽤构造器函 数，使⽤ invokespecial 调⽤了 <init> 后才真正调⽤了构造器函数，正是因为需要调⽤这个函数才导致中间必须要有⼀个 dup 指令，不然调⽤完<init>函数以后，操作数栈为空，就再也找不回刚刚创建 的对象了。

<img src="pic/JVM字节码从入门到精通/image-20211114202341900.png" alt="image-20211114202341900" style="zoom:50%;" />

前⾯我们知道 <init> 会调⽤构造器函数，<clinit> 是类的静态初始化 ⽐ <init> 调⽤得更早⼀些，<clinit> 不会直接被调⽤，它在下⾯这个四个指令触发调⽤：new, getstatic, putstatic or invokestatic 。也就是说，初始化⼀个类实例、访问⼀个静态变量或者⼀个静态⽅法，类的静态初始化⽅法就会被触发。

看⼀个具体的例⼦

```java
public class Initializer { 
  static int a; 
  static int b; 
  static {
    a = 1; b = 2; 
  } 
} 
```

部分字节码如下 

```shell
static {}; 
	0: iconst_1 
	1: putstatic 		#2				// Field a:r
	4: iconst_2 
	5: putstatic 		#3				// Field b:I
	8: return
```

上⾯的 static {} 就对应我们刚说的 <clinit>

## 0x02 相关⾯试题分析

某东的⼀个⾯试题如下，类 A 和类 B 的关系如下

```java
public class A { 
  static { 
    System.out.println("A init"); 
  } 
  public A() { 
    System.out.println("A Instance" ); 
  } 
}

public class B extends A { 
  static { 
    System.out.println("B init"); 
  } 
  public B() { 
    System.out.println("B Instance" ); 
  } 
}
```

### 问题 1： A a = new B(); 输出结果及正确的顺序？ 

要彻底搞清楚这个问题，需要弄清楚 B 构造器函数的字节码。

```shell
public B();
	0: aload_0 	
	1: invokespecial #1 		// Method A."<init>":()V
	4: getstatic #2 				// Field java/lang/System.out:Ljava/io/PrintStream;
	7: ldc #3 							// String B Instance
	9: invokevirtual #4 		// Method java/io/PrintStream.println:(Ljava/lang/String;)V
 12: return
```

从 B 的构造器函数字节码可以看到它⾸先默默的帮忙调⽤了 A 的构造器函数 所以刚刚的过程是 new B() 的 <init> 触发了 B 的静态初始化 <clinit>，但这时⽗类 A 还没有进⾏静态初始化，会先进⾏ A 的静态初始化，然后执⾏ B 的构造器函数时，先调⽤了 A 的 <init> 构造器函数，最后执⾏ B 的构造器函数。

<img src="pic/JVM字节码从入门到精通/image-20211114204451086.png" alt="image-20211114204451086" style="zoom:50%;" />

所以上述答案是：

```
A init 
B init 
A Instance 
B Instance
```

### 问题 2：B[] arr = new B[10] 会输出什么？

这涉及到数组的初始化指令，对应字节码如下：

```shell
bipush 10 
anewarray 'B' 
astore 1
```



# invokeXXX 指令

## 0x01 ⽅法的静态绑定与动态绑定



## invokestatic



## invokevirtual vs invokespecial





# HSDB



# 匿名内部类





# i++ vs ++i



# syntactic sugar



# try-catch-finally 



# try-with-resource



# Kotlin



# synchronized



# java泛型



# javac 编译原理

## javac 源码调试



# Java Instrumentation 包



# ASM



# CGLIB



# CRACK



# APM



