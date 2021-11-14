[toc]

不学习底层知识可能不会阻碍你称为一个称职的程序员，但也许会阻碍你成为一个优秀的程序员。我所理解的底层知识，是指编程或开发所依赖的平台(或者框架、工具)的知识。对于 Java 开发者来 说，虚拟机、字节码就是其底层知识。

# 1. Hello, World

这篇文章我们以输出 "Hello, World" 来开始字节码之旅，如果之前没有怎么接触过字节码的话，这篇文章应该能够让你对字节码有一个最基本的认识。 

## **0x01 java** 文件如何变成 **.class** 文件

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

## **0x02** 魔数 **0xCAFEBABE**

<img src="pic/JVM字节码从入门到精通/image-20211114092002409.png" alt="image-20211114092002409" style="zoom:50%;" />

class 文件的头四个字节称为魔数(Magic Number)，可以看到 class 的魔数为 0xCAFEBABE。很多文件都以魔数来进行文件类型的区分，比如 PDF 文件的魔数是%PDF-(16进制0x255044462D)，png 文件的 魔数是\x89PNG(0x89504E47)。文件格式的制定者可以自由的选择魔数值，只要魔数值还没有被广泛的采用过且不会引起混淆即可。

Java 早期开发者选用了这样一个浪漫气息的魔数，高司令有解释这一段 轶事。这个魔数值在 Java 还成为 Oak 语言的时候就已经确定下来了。

这个魔数是 JVM 识别 .class 文件的标志，虚拟机在加载类文件之前会先检查这四个字节，如果不是 0xCAFEBABE 则拒绝加载该文件，更多关于字节码格式的说明，我们会在后面的文章中慢慢介绍。

## **0x03 javap** 详解

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

### 操作数栈

每个栈帧内部都包含了⼀个称为操作数栈的后进先出（LIFO）栈，栈的⼤⼩同样也是在编译期间确定。Java 虚拟机提供的⼀些字节码指令⽤来从局部变量表或者对象实例的字段中复制常量或者变量到操 作数栈，也有⼀些指令⽤于从操作数栈取⾛数据、操作数据和把操作结果重新⼊栈。在⽅法调⽤时，操作数栈也⽤来准备调⽤⽅法的参数和接收⽅法返回的结果。

⽐如 iadd 指令⽤来将两个 int 类型的数值相加，它要求执⾏之前操作数栈已经存在两个由前⾯其它指令放⼊的 int 型数值，在 iadd 指令执⾏时，两个 int 值从操作数栈中出栈，相加求和，然后将求和的结 果重新⼊栈。