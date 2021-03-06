<!-- TOC -->

- [**一、Class文件结构**](#一class文件结构)
- [**二、字节码指令集**](#二字节码指令集)
- [**三、类的加载过程**](#三类的加载过程)
- [**四、类的加载器**](#四类的加载器)

<!-- /TOC -->


# **一、Class文件结构**
1. 想要让一个Java程序正确地运行在JVM中，Java源码就必须要被编译为符合JVM规范的字节码。前端编译器的主要任务就是负责将**符合Java语法规范的Java代码**转换为**符合JVM规范的字节码**。

2. *字节码文件里是什么?*源代码经过编译器编译之后便会生成一个字节码文件，字节码是一种**二进制的类文件**，它的内容是JVM的指令，而不像C、C++经由编译器直接生成机器码。(字节码是面向JVM的；而机器指令是面向处理器的)

3. *什么是字节码指令(byte code)?*Java虚拟机的指令由**一个字节长度的**、代表着某种特定操作含义的**操作码(opcode)**以及跟随其后的零至多个代表此操作所需参数的**操作数(operand)**所构成。虚拟机中许多指令并不包含操作数，只有一个操作码。

4. class文件格式:Class 的结构不像XML等描述语言，由于它没有任何分隔符号。所以在其中的数据项，无论是**字节顺序还是数量**，都是被严格限定的，哪个字节代表什么含义，长度是多少，先后顺序如何，都不允许改变。class文件格式采用一种类似于C语言结构体的方式进行数据存储，这种结构中只有两种数据类型**无符号数和表**。
    - 无符号数属于基本的数据类型，以**u1、u2、u4、u8**来分别代表1个字节、2个字节、4个字节和8个字节的无符号数，无符号数可以用来描述数字、索引引用、数量值或者按照UTF-8编码构成字符串值。
    - 表是由多个**无符号数或者其他表**作为数据项构成的复合数据类型，所有表都习惯性地以“_info”结尾。表用于描述有层次关系的复合结构的数据，整个Class文件本质上就是一张表。由于表没有固定长度，所以通常会在其前面加上个数说明

5. **class文件的结构并不是一成不变的**，随着Java虚拟机的不断发展，总是不可避免地会对Class文件结构做出一些调整，但是其基本结构和框架是非常稳定的。

6. class文件结构： 
    | 类型 | 名称 | 说明 | 长度 | 数量 |
    | :-----| :---- | :---- |:---- |:---- |
    | u4 | magic  | 魔数，识别Class文件格式 | 4个字节  | 1 |
    | u2 | minor_version | 副版本号(小本版) | 2个字节  | 1 | 
    | u2 | major_version | 主版本号(大版本) | 2个字节  | 1 | 
    | u2 | constant_pool_count | 常量池计数器 | 2个字节 | 1 | 
    | cp_info | constant_pool  | 常量池表  | n个字节  | constant_pool_count-1 | 
    | u2 | access_flags | 访问标识 | 2个字节 | 1 | 
    | u2 | this_class   | 类索引   | 2个字节 | 1 | 
    | u2 | super_class  | 父类索引 | 2个字节 | 1  | 
    | u2 | interface_count | 接口计数器 | 2个字节 | 1 | 
    | u2 | interface | 接口索引集合 | 2个字节 | interfa_count | 
    | u2 | field_count | 字段计数器 | 2个字节 | 1 | 
    | field_info | fields | 字段表集合 | n个字节  | field_count | 
    | u2 | methods_count | 方法计数器 | 2个字节 | 1 | 
    | method_info | methods | 方法表集合 | n个字节 | method_count | 
    | u2 | attributes_count | 属性计数器 | 2个字节  | 1 | 
    | attribute_info | attributes | 属性表集合 | n个字节  | attributes_count | 

7. 魔数(0xCAFEBABY)：使用魔数而不是扩展名来进行识别主要是基于安全方面的考虑，因为文件扩展名可以随意地改动。(**Java文件标识**)

8. 副版本号&主版本号：不同版本的Java编译器编译的Class文件对应的版本是不一样的。目前，高版本的Java虚拟机可以执行由低版本编译器生成的Class文件,但是低版本的Java虚拟机不能执行由高版本编译器生成的Class文件。否则JVM会抛出java.lang.UnsupportedClassVersionError异常。(**向下兼容**)

9. 常量池计数器&常量池：常量池是Class文件中内容最为丰富的区域之一。常量池对于Class文件中的字段和方法解析也有着至关重要的作用.随着Java虚拟机的不断发展，常量池的内容也日渐丰富。可以说，**常量池是整个class文件的基石**。

10. 常量池表项中，用于存放编译时期生成的各种**字面量和符号引用**，这部分内容将在类加载后进入方法区的运行时常量池中存放。注意：**常量池计数器的值是从1而不是从0开始计数！**即constant_pool_count = 1，表示常量池中的没有任何项。

11. 通常我们写代码时都是从O开始的，但是这里的常量池却是从1开始，因为它把第0项常量空出来了。这是为了满足后面某些指向常量池的索引值的数据在特定情况下需要表达“**不引用任何一个常量池项目**”的含义，这种情况可用索引值0来表示。

12. 常量池包含了class文件结构及其子结构中引用的所有字符串常量、类或接口名、字段名和其他常量。常量池中的每一项都具备相同的特征。第1个字节作为类型标记，用于确定该项的格式，即常量数据类型，这个字节称为tag byte(标记字节、标签字节)。**常量类型与标记对应关系**如下表(共14中数据类型)：
    | 类型 | 标记 | 描述 |
    | :--- | :--- | :--- |
    | CONSTANT_utf8_info | 1 | UTF-8编码的字符串 |
    | CONSTANT_Integer_info | 3 | 整型字面量 | 
    | CONSTANT_Float_info | 4 | 浮点型字面量 |
    | CONSTANT_Long_info | 5 | 长整型字面量 | 
    | CONSTANT_Double_info | 6 | 双精度浮点型字面量 |
    | CONSTANT_Class_info | 7 | 类或接口的符号引用 | 
    | CONSTANT_String_info| 8 | 字符串类型字面量 |
    | CONSTANT_Fieldref_info | 9 | 字段的符号引用 |
    | CONSTANT_Methodref_info | 10 | 类中方法的符号引用 |
    | CONSTANT_InterfaceMethodref_info | 11 | 接口中方法的符号引用 |
    | CONSTANT_NameAndType_info | 12 | 字段或方法的符号引用 |
    | CONSTANT_MethodHandle_info | 15 | 表示方法句柄 |
    | CONSTANT_MethodType_info | 16 | 标志方法类型 |
    | CONSTANT_InvokeDynamic_info | 18 | 表示一个动态方法调用点 |

13. 字面量(Literal)包括**文本字符串、声明为final的常量值**；符号引用(Sysbolic References)包括**类和接口**的全限定名、**字段**的名称和描述符、**方法**的名称描述符。

14. 几个重要的概念：
    - **全限定名**：java/lang/String这个就是类的全限定名，仅仅是把包名的"."替换成"/"，为了使连续的多个全限定名之间不产生混淆，在使用时最后一般会加入一个“;”表示全限定名结束。
    - **简单名称**：不带类型和参数的方法或者字段名称，上面例子中的类的add()方法和num字段的简单名称分别是add和num
    - **描述符**：描述符的作用是用来描述字段的数据类型、方法的参数列表(包括数量、类型以及顺序)和返回值。根据描述符规则基本数据类型(byte、char、double、float、int、long、short、boolean)以及代表无返回值的void类型都用一个大写字符来表示，而对象类型则用字符L加对象的全限定名来表示。用描述符来描述方法时，按照先参数列表，后返回值的顺序描述，参数列表按照参数的严格顺序放在一组小括号“()”之内。如方法java.lang.String toString()的描述符为()Ljava/lang/String;，方法int abc(int[]x, int y)的描述符为([II)I。

15. 虚拟机在加载Class文件时才会进行动态链接，也就是说，Class文件中**不会保存各个方法和字段的最终内存布局信息**，因此，这些字段和方法的符号引用不经过转换是无法直接被虚拟机使用的。当虚拟机运行时，需要从常量池中获得对应的符号引用，再在类加载过程中的链接阶段的**解析环节**(链接阶段分为三个步骤 检验、准备、解析)将其替换为直接引用，并翻译到具体的内存地址中。这里说明下符号引用和直接引用的区别与关联:
    - **符号引用**:符号引用以一组**符号来描述所引用的目标**，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可。**符号引用与虚拟机实现的内存布局无关**，引用的目标并不一定已经加载到了内存中。
    - **直接引用**:直接引用可以是**直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄**。**直接引用是与虚拟机实现的内存布局相关的**，同一个符号引用在不同虚拟机实例上翻译出来的直接引用一般不会相同。如果有了直接引用，那说明引用的目标必定己经存在于内存之中了。

16. 常量池总结：
    - 这14种表(或者常量项结构)的共同点是:表开始的**第一位是一个u1类型的标志位**(tag)，代表当前这个常量项使用的是哪种表结构，即哪种常量类型。
    - 在常量池列表中，CONSTANT_utf8_info常量项是一种使用改进过的UTF-8编码格式来存储诸如文字字符串、类或者接口的全限定名、字段或者方法的简单名称以及描述符等常量字符串信息。
    - 这14种常量项结构还有一个特点是，其中13个常量项占用的字节固定，只有CONSTANT_utf8_info占用字节不固定，其大小由length决定。因为从常量池存放的内容可知，其存放的是字面量和符号引用，最终这些内容都会是一个个字符串，这些字符串的大小是在编写程序时才确定，比如你定义一个类，类名可以取长取短，所以在没编译前，大小不固定，编译后，通过utf-8编码，就可以知道其长度。
    - 常量池:可以理解为Class文件之中的**资源仓库**, 它是Class文件结构中与其他项目关联最多的数据类型(后面的很多数据类型都会指向此处)，也是占用class文件空间最大的数据项目之一。
    - Java代码在进行Javac编译的时候，并不像C和C++那样有“连接”这一步骤，而是在虚拟机加载Class文件的时候进行动态链接。也就是说，**在Class文件中不会保存各个方法、字段的最终内存布局信息**，因此这些字段、方法的符号引用不经过运行期转换(解析)的话无法得到真正的内存入口地址，也就无法直接被虚拟机使用。当虚拟机运行时，需要从常量池获得对应的符号引用，再在类创建时或运行时解析、翻译到具体的内存地址之中。

17. **访问标记**:该标记使用两个字节表示(通常以ACC_开头)，用于识别一些类或者接口层次的访问信息，包括:这class是类还是接口;是否定义为 public类型;是否定义为 abstract类型;如果是类的话，是否被声明为 final
等。各种访问标记如下所示:
    | 标志名称 | 标志值 | 含义 |
    | :------ | :----- | :----|
    | ACC_PUBLIC | 0x0001 | 标志为public类型 |
    | ACC_FINAL | 0x0010 | 标志被声明为final，只有类可以设置 |
    | ACC_SUPER | 0x0020 | 标志允许使用invokespecial字节码指令的新语义，JDK1.0.2之后编译出来的类的这个标志默认为真。(使用增强的方法调用父类方法)|
    | ACC_INTERFACE | 0x0200 | 标志这是一个接口 |
    | ACC_ABSTRACT | 0x0400 | 是否为abstract类型，对于接口或者抽象类来说，此标志值为真，其他类型为假 |
    | ACC_SYNTHETIC | 0x1000 | 标志此类并非由用户代码产生(即:由编译器产生的类，没有源码对应) |
    | ACC_ANNOTATION | 0x2000 | 标志这是一个注解 |
    | ACC_ENUM | 0x4000 | 标志这是一个枚举 |

18. **访问标记总结**：
    - 1.带有ACC_INTERFACE标志的class文件表示的是接口而不是类，反之则表示的是类而不是接口。如果一个class文件被设置了ACC_INTERFACE标志，那么同时也得设置ACC_ABSTRACT标志。同时它不能再设置 ACC_FINAL、Acc_SUPER或ACC_ENUM标志；如果没有设置ACC_INTERFACE标志，那么这个class文件可以具有上表中除 ACC_AINOTATTON外的其他所有标志。当然，ACC_FINAL和ACC_ABSTRACT这类互斥的标志除外。这两个标志不得同时设置。
    - 2.ACC_SUPER标志用于确定类或接口里面的invokespecial指令使用的是哪一种执行语义。针对Java虚拟机指令集的编译器都应当设置这个标志。对于Java SE 8及后续版本来说，无论class文件中这个标志的实际值是什么，也不管class文件的版本号是多少，Java虚拟机都认为每个class文件均设置了ACC_SUPER标志。
    - 3.Acc_SYNTHETIC标志意味着该类或接口是由编译器生成的，而不是由源代码生成的；注解类型必须设置ACC_ANNOTATION标志。如果设置了ACC_ANNOTATION标志，那么也必须设置ACC_INTERFACE标志；ACC_ENUM标志表明该类或其父类为枚举类型。(synthetics 人工合成的)

19. **类索引&父类索引&接口索引集合**：**类索引用于确定这个类的全限定名**；**父类索引用于确定这个类的父类的全限定名**；由于Java语言不允许多重继承，所以父类索引只有一个，除了
java.lang.Object 之外，所有的Java类都有父类，因此除了java.lang.Object外，所有Java类的父类索引都不为空；**接口索引集合就用来描述这个类实现了哪些接口**，这些被实现的接口将按implements语句(如果这个类本身是一个接口，则应当是extends语句)后的接口顺序从左到右排列在接口索引集合中。

20. **字段表集合**：用于描述接口或类中声明的变量。字段(field)包括类级变量以及实例级变量，但是**不包括方法内部、代码块内部声明的局部变量以及从父类或父类接口继承的那些字段**。字段叫什么名字、字段被定义为什么数据类型，这些都是无法固定的，只能引用常量池中的常量来描述。它指向常量池索引集合，它描述了每个字段的完整信息。比如字段的标识符、访问修饰符(public、private或protected)、是类变量还是实例变量(static修饰符)、是否是常量(final修饰符)等。

21. 字段表集合中不会列出从父类或者实现的接口中继承而来的字段，但有**可能列出原本Java代码之中不存在的字段，譬如在内部类中为了保持对外部类的访问性，会自动添加指向外部类实例的字段。**在Java语言中字段是无法重载的，两个字段的数据类型、修饰符不管是否相同，都必须使用不一样的名称，但是对于字节码来讲，如果两个字段的描述符不一致，那字段重名就是合法的。

22. 一个字段的信息包括如下这些信息。这些信息中，各个修饰符都是布尔值，要么有，要么没有。**作用域**(public、 private、protected修饰符)；**是实例变量还是类变量**(static修饰符)；**可变性**(final)；**并发可见性**(volatile修饰符，是否强制从主内存读写)；**可否序列化**(transient修饰符)；**字段数据类型**(基本数据类型、对象、数组)；**字段名称**

23. 字段表(field_info)结构如下：
    | 类型 | 名称 | 含义 | 数量 |
    | :--- | :--- | :--- | :--- |
    | u2 | access_flags | 字段访问标志 | 1 |
    | u2 | name_index | 字段名索引 | 1 |
    | u2 | descriptor_index | 描述符索引 | 1 |
    | u2 | attributes_count | 属性计数器 | 1 |
    | attribute_info | attributes | 属性集合 | attributes_count |

24. **方法表集合**：指向常量池索引集合，它完整描述了每个方法的签名。在字节码文件中，每一个method_info项都对应着一个类或者接口中的方法信息。比如方法的访问修饰符(public、private或protected),方法的返回值类型以及方法的参数信息等。一方面，**methods表只描述当前类或接口中声明的方法，不包括从父类或父接口继承的方法**。另一方面，**methods表有可能会出现由编译器自动添加的方法**，最典型的便是编译器产生的方法信息(比如:类(接口)初始化方法< clinit>()和实例初始化方法< init>())。

25. 方法表注意事项:在**Java语言中**，要重载(Overload)一个方法，除了要与原方法具有相同的简单名称之外，还要求必须拥有一个与原方法不同的**特征签名(函数签名)**，特征签名就是一个方法中各个参数在常量池中的字段符号引用的集合，也就是因为返回值不会包含在特征签名之中，因此**Java语言里无法仅仅依靠返回值的不同来对一个已有方法进行重载**。但在Class文件格式中，特征签名的范围更大一些，只要描述符不是完全一致的两个方法就可以共存。也就是说，如果两个方法有相同的名称和特征签名，但**返回值不同，那么也是可以合法共存于同一个class文件中**。也就是说，尽管Java语法规范并不允许在一个类或者接口中声明多个方法签名相同的方法，但是和Java语法规范相反，字节码文件中却恰恰允许存放多个方法签名相同的方法，唯一的条件就是这些方法之间的返回值不能相同。

26. methods表中的每个成员都必须是一个method_info结构，用于表示**当前类或接口中某个方法的完整描述**。如果某个method_info结构的access_flags项既没有设置ACC_NATIVE标志也没有设置ACC_ABSTRACT标志，那么该结构中也应包含实现这个方法所用的Java虚拟机指令。method_info结构可以表示类和接口中定义的所有方法，包括实例方法、类方法、实例初始化方法和类或接口初始化方法方法表的结构实际跟字段表是一样的，方法表结构(method_info)如下:
    | 类型 | 名称 | 含义 | 数量 |
    | :--- | :--- | :--- | :--- |
    | u2 | access_flags | 方法访问标志 | 1 |
    | u2 | name_index | 方法名索引 | 1 |
    | u2 | descriptor_index | 描述符索引 | 1 |
    | u2 | attributes_count | 属性计数器 | 1 |
    | attribute_info | attributes | 属性集合 | attributes_count |

27. **属性表集合**：指的是class文件所携带的**辅助信息**，比如该class文件的源文件的名称。字段表、方法表都可以有自己的属性表。用于描述某些场景专有的信息。属性表集合的限制没有那么严格，不再要求各个属性表具有严格的顺序，并且只要不与己有的属性名重复，任何人实现的编译器都可以向属性表中写入自己定义的属性信息，但Java虚拟机运行时会忽略掉它不认识的属性。

28. 属性表的每个项的值必须是attribute_info结构。属性表的结构比较灵活，各种不同的属性只要满足以下结构即可。属性表(attribute_info)的通用格式如下：
    | 类型 | 名称 | 含义 | 数量 |
    | :--- | :--- | :--- | :--- |
    | u2 | attribute_name_index | 属性名索引 | 1 |
    | u4 | attribute_length | 属性长度(唯一一个长度标识为4个字节的) | 1 |
    | u1 | info | 属性表 | attribute_length |

29. 反解析指令使用组合 javap -v -p ***.class

30. class文件结构官方参考文档 [The ClassFile Structure](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-4.html)


# **二、字节码指令集**
1. Java虚拟机的指令由**一个字节长度的**、代表着某种特定操作含义的数字(称为**操作码，Opcode**)以及跟随其后的零至多个代表此操作所需参数(称为**操作数，Operands**)而构成。由于Java虚拟机采用面向操作数栈而不是寄存器的结构，所以大多数的指令都不包含操作数，只有一个操作码。由于限制了Java虚拟机操作码的长度为一个字节(即0~255)，这意味着指令集的操作码总数不可能超过256条。熟悉虚拟机的指令对于动态字节码生成、反编译Class文件、Class文件修补都有着非常重要的价值。

2. 字节码执行模型：如果不考虑异常处理的话，那么Java虚拟机的解释器可以使用下面这个伪代码当做最基本的**执行模型**来理解:
    ```java
    do{
        自动计算PC寄存器的值加1;
        根据PC寄存器的指示位置，从字节码流中取出操作码;
        if(字节码存在操作数)从字节码流中取出操作数;
        执行操作码所定义的操作;
    }while(字节码长度>0);
    ```

3. 大部分的指令都没有支持整数类型byte、char和short，甚至没有任何指令支持boolean类型。编译器会在编译期或运行期将byte和short类型的数据**带符号扩展(Sign-Extend)**为相应的int类型数据，将boolean和char类型数据**零位扩展(Zero-Extend)**为相应的int类型数据。与之类似，**在处理boolean、byte、short和char类型的数组时，也会转换为使用对应的int类型的字节码指令来处理**。因此，大多数对于boolean、byte、short和char类型数据的操作，实际上都是使用相应的int类型作为运算类型。

4. JVM中的字节码指令集按用途可大致分成9类：
    - 加载与存储指令：用于将数据从栈帧的局部变量表和操作数栈之间来回传递
    - 算术指令
    - 类型转换指令
    - 对象的创建与访问指令
    - 方法调用与返回指令
    - 操作数栈管理指令
    - 比较控制指令
    - 异常处理指令
    - 同步控制指令

5. 在做值相关操作时:
    - 一个指令，可以从局部变量表、常量池、堆中对象、方法调用、系统调用中等取得数据，这些数据(可能是值，可能是对象的引用)被压入操作数栈。
    - 一个指令，也可以从操作数栈中取出一到多个值(pop多次)，完成赋值、加减乘除、方法传参、系统调用等等操作。

6. 常用的加载与存储指令：
    - 【局部变量压栈指令】将一个**局部变量**加载到操作数栈: xload、xload_< n>(其中x为、1、f、d、a, n为日到3)
    - 【常量入栈指令】将一个**常量**加载到操作数栈: bipush、sipush、ldc、ldc_w、ldc2_w、aconst_null、iconst_m1、iconst_< i>、lconst_< l>、fconst_< f>、dconst_< d>
    - 【出栈装入局部变量表指令】将一个**数值从操作数栈存储到局部变量表**: xstore、xstore_< n>(其xi、1、f、d、a，n为0到3) ; xastore(其中xoi、1、f、d、a、b、c、s)
    - 扩充局部变量表的访问索引的指令: wide。(很少使用)

7. 对于这若干组特殊指令来说，它们表面上没有操作数，不需要进行取操作数的动作，但操作数都隐含在指令中。操作byte、char、short和boolean类型数据时，**经常用int类型的指令来表示**。

8. **操作数栈**：执行每一条指令之前，Java虚拟机要求该指令的操作数已被压入操作数栈中。在执行指令时，Java虚拟机会将该指令所需的操作数弹出，并且将指令的结果重新压入栈中。

9. **局部变量表**：Java方法栈桢的另外一个重要组成部分则是局部变量区，字节码程序可以将计算的结果缓存在局部变量区之中。实际上，Java虚拟机将局部变量区当成一个数组，依次存放this指针(仅非静态方法)，所传入的参数，以及字节码中的局部变量。和操作数栈一样，long类型以及double类型的值将占据两个单元，其余类型仅占据一个单元。

10. 常量入栈指令的功能是将常数压入操作数栈，根据数据类型和入栈内容的不同，又可以分为**const系列**、**push系列**和**ldc指令**。(**压入栈数据的宽度依次递增**)
    - 指令const系列:**用于对特定的常量入栈**，入栈的常量隐含在指令本身里。指令有: iconst_< i>(i从-1到5)、lconst_< l>(1从0到1)、fconst_< f>(f从0到2)、dconst_< d> (d从0到1)、aconst_null。比如，iconst_m1将-1压入操作数栈;iconst_x (x为9到5)将x压入栈:lconst_0、lconst_1分别将长整数0和1压入栈;fconst_0、fconst_1、fconst_2分别将浮点数0、1、2压入栈;dconst_0和dconst_1分别将double型0和1压入栈。aconst_null将null压入操作数栈;从指令的命名上不难找出规律，指令助记符的第一个字符总是表示数据类型，i表示整数，1表示长整数，f表示浮点数，d表示双精度浮点，习惯上用a表示对象引用。**如果指令隐含操作的参数，会以下划线形式给出。**
    - 指令push系列:主要包括bipush和sipush。它们的区别在于接收数据类型的不同，**bipush接收8位整数作为参数，sipush接收16位整数**，它们都将参数压入栈。
    - 指令ldc系列:如果以上指令都不能满足需求，那么可以使用万能的ldc指令，它可以**接收一个8位的参数该参数指向常量池中的int、float或者String的索引，将指定的内容压入堆栈**。类似的还有ldc_w，它接收两个8位参数，能支持的索引范围大于ldc。如果要压入的元素是long或者double类型的,则使用ldc2_w指令，使用方式都是类似的。

11. 注意：常量入栈指令中的n和局部变量压栈指令中的n不一样，**常量入栈指令的n代表数值或者对象，而不是局部变量表中的下标**。

12. 出栈装入局部变量表指令用于将操作数栈中栈顶元素弹出后，装入局部变量表的指定位置，用于给局部变量赋值。这类指令主要以**store**的形式存在，比如xstore(x为i、l、f、d、a)、 xstore_n (x为i、l、f、d、a，n为0至3)。其中，指令istore_n将从操作数栈中弹出一个整数，并把它赋值给局部变量索引n位置。指令xstore由于没有隐含参数信息，故需要提供一个**byte类型**的参数类指定目标局部变量表的位置。

13. 注意：在方法没有运行的时候，根据字节码文件就可以计算出需要几个槽位，字节码中的Maximum local variables **属性指的是局部变量表中所有变量所占槽位总和，而不是局部变量表中局部变量的数目**(doble&long占用两个槽位，其他基础类型以及引用类型均占一个槽位，非静态方法不要忽略this局部变量)

14. 算术指令用于对操作数栈上一个或以上的值进行某种特定运算，并把**结果重新压入操作数栈**。大体上可以分为两种:对**整型数据**进行运算的指令与对**浮点类型数据**进行运算的指令。在每一大类中，都有针对Java虚拟机具体数据类型的专用算术指令。但没有直接支持byte、short、char和boolean类型的算术指令，对于这些数据的运算，都使用int类型的指令来处理。此外，在处理boolean、byte、short和char类型的数组时，也会**转换为使用对应的int类型的字节码指令来处理**。

15. 运算时溢出：**数据运算可能会导致溢出**，例如两个很大的正整数相加，结果可能是一个负数。其实Java虚拟机规范并无明确规定过整型数据溢出的具体结果，仅规定了在处理整型数据时，只有除法指令以及求余指令中当出现除数为0时会导致虚拟机抛出异常ArithmeticException。

16. 运算模式：
    - **向最接近数舍入模式:JVM要求在进行浮点数计算时**，所有的运算结果都必须舍入到适当的精度，非精确结果必须舍入为可被表示的最接近的精确值，如果有两种可表示的形式与该值一样接近，将优先选择最低有效位为零的;(**四舍五入**)
    - **向零舍入模式:将浮点数转换为整数时**，采用该模式，该模式将在目标数值类型中选择一个最接近但是不大于原值的数字作为最精确的舍入结果;(**取整运算**)
  
17. NaN值使用：当一个操作产生溢出时，将会使用有符号的无穷大(Infinity)表示，如果某个操作结果没有明确的数学定义的话，将会使用NaN值来表示。而且所有使用NaN值作为操作数的算术操作，结果都会返回NaN;

18. 所有的算术指令包括:
    - 加法指令: iadd、ladd、fadd、dadd
    - 减去指令: isub、lsub、fsub、dsub
    - 乘法指令:imul、1mul、 fmul、dmul
    - 除法指令: idiv、ldiv、fdiv、ddiv
    - 求余指令: irem、lrem、frem、drem
    - 取反指令: ineg、1neg、fneg、dneg
    - 自增指令: iinc
    - 位运算指令，又可分为:
        - 位移指令: ishl、 ishr、 iushr、lshl、lshr、lushr;
        - 按位或指令: ior、lor;
        - 按位与指令: iand、land;
        - 按位异或指令: ixor、lxor;
        - 比较指令: dcmpg、dcmpl、fcmpg、fcmp1、lcmp

19. 比较指令的说明:
    - 比较指令的作用是比较栈顶两个元素的大小，并将**比较结果入栈**。
    - 比较指令有: dcmpg, dcmpl、fcmpg、fcmpl、lcmp.
    - 与前面讲解的指令类似，首字符d表示double类型，f表示float,l表示long
    - 对于double和float类型的数字，**由于NaN的存在，各有两个版本的比较指令**。以float为例，有fcmpg和fcmpl两个指令，它们的区别在于在数字比较时，若遇到NaN值，处理结果不同。
    - 指令fcmpg和fcmpl都从栈中弹出两个操作数，并将它们做比较，设栈顶的元素为v2,栈顶顺位第2位的元素为v1,若v1=v2,则压入0;若v1>v2则压入1;若v1 < v2则压入-1。两个指令的不同之处在于，如果遇到NaN值，fcmpg会压入1,而fcmpl会压入-1。

20. 类型转换指令说明:
    - 类型转换指令可以将两种不同的数值类型进行相互转换。
    - 这些转换操作一般用于实现用户代码中的显式类型转换操作(**强制类型转换**)，或者用来处理字节码指令集中数据类型相关指令无法与数据类型一一对应的问题(**自动类型转换)**。

21. **宽化类型转换(Widening Numeric Conversions)**
    - 1.转换规则:Java虚拟机直接支持以下数值的宽化类型转换(小范围类型向大范围类型的**安全转换**,但同样存在精度丢失情况)。也就是说，并不需要指令执行而是自动完成转换的，包括:
        - 从int类型到long、float或者double类型。对应的指令为: i21、i2f、i2d
        - 从long类型到float、double类型。对应的指令为:12f、12d
        - 从float类型到double类型。对应的指令为:f2d
        - 简化为: **int --> long --> float --> double**
    - 2.精度损失问题：
        - 宽化类型转换是不会因为超过目标类型最大值而丢失信息的，例如，从int转换到 long，或者从int转换到double，都不会丢失任何信息，转换前后的值是精确相等的。
        - 从int、long类型数值转换到float，或者1ong类型数值转换到double时，**将可能发生精度丢失—―可能丢失掉几个最低有效位上的值**，转换后的浮点数值是根据IEEE754最接近舍入模式所得到的正确整数值。
        - 尽管宽化类型转换实际上是可能发生精度丢失的，但是这种转换**永远不会导致Java虚拟机抛出运行时异常**。

22. 宽化类型转换补充说明：从byte、char和short类型到int类型的宽化类型转换实际上是不存在的。**对于byte类型转为int,虚拟机并没有做实质性的转化处理，只是简单地通过操作数栈交换了两个数据**。而将byte转为long时，使用的是i2l,可以看到**在内部byte在这里已经等同于int类型处理**，类似的还有short类型，这种处理方式有两个特点:
    - **一方面可以减少实际的数据类型**，如果为short和byte都准备一套指令，那么指令的数量就会大增，而虚拟机目前的设计上，只愿意使用一个字节表示指令，因此指令总数不能超过256个，为了节省指令资源，将short和byte当做int处理也在情理之中。
    - **另一方面，由于局部变量表中的槽位固定为32位(4个字节)，无论是byte或者short存入局部变量表都会占用32位空间**。从这个角度说，也没有必要特意区分这几种数据类型。

23. **窄化类型转换(Narrowing Numeric Conversion)**
    - 1.转换规则:Java虚拟机也直接支持以下窄化类型转换(大范围类型向小范围类型的转换):
        - 从int类型至byte、short或者char类型。对应的指令有: i2b、i2c、i2s
        - 从long类型到int类型。对应的指令有:l2i
        - 从float类型到int或者long类型。对应的指令有:f2i、f2l
        - 从double类型到int、long或者float类型。对应的指令有: d2i、d2l、d2f
    - 2.精度损失问题
        - 窄化类型转换可能会导致转换结果具备不同的正负号、不同的数量级，因此，**转换过程很可能会导致数值丢失精度**。
        - 尽管数据类型窄化转换可能会发生上限溢出、下限溢出和精度丢失等情况，但是Java虚拟机规范中明确规定数值类型的**窄化转换指令永远不可能导致虚拟机抛出运行时异常**。

24. **窄化类型转换补充说明:**
    - 1.当将一个浮点值窄化转换为整数类型T(T限于int或long类型之一)的时候，将遵循以下转换规则:
        - 如果浮点值是NaN，那转换结果就是int或long类型的0。
        - 如果浮点值不是无穷大的话，浮点值使用IEEE 754的向**零舍入模式取整**，获得整数值v，如果v在目标类型T(int或long)的表示范围之内，那转换结果就是v。否则，将根据v的符号，转换为T所能表示的最大或者最小正数
    - 2.当将一个double类型窄化转换为float类型时，将遵循以下转换规则:通过**向最接近数舍入模式**舍入一个可以使用float类型表示的数字。最后结果根据下面这3条规则判断:
        - 如果转换结果的绝对值太小而无法使用float来表示，将返回float类型的正负零。
        - 如果转换结果的绝对值太大而无法使用float来表示，将返回float类型的正负无穷大。
        - 对于double类型的NaN值将按规定转换为float类型的NaN值。

25. 注意：从float、double、long等类型往byte、short、char类型转换的时候，需要先把前面几种类型转换成int类型，然后在从int类型转换到后面这几种类型，所以**int类型相当于byte、short、char类型转换时的一种过渡类型**。

26. Java是面向对象的程序设计语言，虚拟机平台从字节码层面就对面向对象做了深层次的支持。有一系列指令专门用于对象操作，可进一步细分为创建指令、字段访问指令、数组操作指令、类型检查指令。

27. 创建指令:虽然类实例和数组都是对象，但**Java虚拟机对类实例和数组的创建与操作使用了不同的字节码指令**:
    - 1.创建类实例的指令:
        - 创建类实例的指令: new(**执行new指令后只是在操作数栈中创建一个引用类型并指向在堆空间中开辟的一块空的内存区域，空的内存区域只有在执行< init>指令后才能将具体的实例对象数据填入该空间**)
        - 它接收一个操作数，为指向常量池的索引，表示要创建的类型，执行完成后，将对象的引用压入栈。
    - 2.创建数组的指令:
        - 创建数组的指令: **newarray、anewarray、multianewarray**

28. 字段访问指令:对象创建后，就可以通过对象访问指令获取对象实例或数组实例中的字段或者数组元素
    - **访问类字段**(static字段，或者称为类变量)的指令:getstatic、putstatic(**get相关指令是将静态变量或实例变量数据取出并压入操作数栈**)
    - **访问类实例字段**(非static字段，或者称为实例变量)的指令: getfield、putfield(**put相关指令是将对应数据由操作数栈写入到静态变量或实例变量，而非局部变量表**)

29. 数组操作指令：数组操作指令主要有:xastore和xaload指令。具体为:
    - 把一个数组元素加载到操作数栈的指令:baload、caload、saload、iaload、laload、faload、daload、aaload
    - 将一个**操作数栈的值存储到数组元素中(而非局部变量表)**的指令:bastore、castore、sastore、iastore、lastore、fastore、dastore、aastore
    - 取数组长度的指令:arraylength，该指令弹出栈顶的数组元素，获取数组的长度，将长度压入栈。

30. 数组操作指令说明：
    - 指令xaload表示将数组的元素压栈，比如saload、caload分别表示压入short数组和char数组。指令xaload在执行时，要求操作数中栈顶元素为数组索引i,栈顶顺位第2个元素为数组引用a,该指令会弹出栈顶这两个元素，并将a[i]重新压入栈(取**哪个数组&哪个位置**的数据)。
    - xastore则专门针对数组操作，以iastore为例，它用于给一个int数组的给定索引赋值。在iastore执行前，操作数栈顶需要以此准备3个元素:值、索引、数组引用，iastore会弹出这3个值并将值赋给数组中指定索引的位置。(向**哪个数组&哪个位置&写入值**)

31. 类型检查指令:检查类实例或数组类型的指令:instanceof、checkcast。
    - 指令checkcast用于检查**类型强制转换是否可以进行**。如果可以进行，那么checkcast指令不会改变操作数栈，否则它会抛出ClassCastException异常。
    - 指令instanceof用来判断**给定对象是否是某一个类的实例**，它会将判断结果压入操作数栈。

32. 方法调用指令: invokevirtual、invokeinterface、invokespecial、invokestatic、invokedynamic。以下5条指令用于方法调用:
    - invokevirtual指令用于调用对象的实例方法，根据对象的实际类型进行分派(虚方法分派)，**支持多态**。这也是Java语言中最常见的方法分派方式。
    - invokeinterface指令用于调用接口方法，它会在运行时搜索由**特定对象所实现的这个接口方法**，并找出适合的方法进行调用。(通过接口引用调用实现类的方法)
    - invokespecial指令用于调用一些需要特殊处理的实例方法，包括实例初始化方法(构造器)、私有方法和父类方法。**这些方法都是静态类型绑定的，不会在调用时进行动态分派**。
    - invokestatic指令用于调用命名类中的**类方法**(static方法)。这是**静态绑定**的。(invokespecial&invokestatic 所调用的是不可重写的方法)
    - invokedynamic调用动态绑定的方法，这个是JDK1.7后新加入的指令。用于在运行时动态解析出调用点限定符所引用的方法，并执行该方法。**前面4条调用指令的分派逻辑都固化在java虚拟机内部，而invokedynamic指令的分派逻辑是由用户所设定的引导方法决定的**。(使用较少)

33. 方法返回指令:方法调用结束前，需要进行返回。方法返回指令是根据返回值的类型区分的。包括ireturn(当返回值是boolean、byte、char、short和int类型时使用)、lreturn、freturn、
dreturn和areturn。另外还有一条**return指令供声明为void的方法、实例初始化方法以及类和接口的类初始化方法使用**。

34. 通过*return指令,将当前函数操作数栈的顶层元素弹出，并将这个元素压入调用者函数的操作数栈中(因为调用者非常关心函数的返回值)，所有在当前函数操作数栈中的其他元素都会被丢弃。如果当前返回的是synchronized方法，那么还会执行一个隐含的monitorexit指令，退出临界区。最后，会丢弃当前方法的整个帧,恢复调用者的帧，并将控制权转交给调用者。


35. 操作数栈管理指令：如同操作一个普通数据结构中的堆栈那样，JVM提供的操作数栈管理指令，可以用于直接操作操作数栈的指令,这些指令属于**通用型，对栈的压入或者弹出无需指明数据类型**。这类指令包括如下内容:
    - 将一个或两个元素从栈顶弹出，并且直接废弃:pop.pop2;
    - 复制栈顶一个或两个数值并将复制值或双份的复制值重新压入栈顶: dup，dup2，dup_x1，dup2_x1,dup_x2，dup2_×2;
    - 将栈最顶端的两个Slot数值位置交换:swap。Java虚拟机没有提供交换两个64位数据类型(long、double)数值的指令。
    - 指令nop,是一个非常特殊的指令，它的字节码为0x00。和汇编语言中的nop一样，它表示什么都不做。这条指令一般可用于调试、占位等。

36. 操作数栈管理指令总结:(**所涉及的操作数目均槽位个数**)
    - 不带_x的指令是复制栈顶数据并压入栈顶。包括两个指令，dup和dup2. **dup的系数代表要复制的slot个数，而非数据个数**。
    - dup开头的指令用于复制1个Slot的数据。例如1个int或1个reference类型数据；dup2开头的指令用于复制2个Slot的数据。例如1个long，或2个int，或1个int+1个float类型数据
    - 带_x的指令是复制栈顶数据并插入栈顶以下的某个位置。共有4个指令, dup_x1，dup2_x1,dup_x2， dup2_x2。对于带_x的复制插入指令，只要将指令的dup和x的系数相加，结果即为需要插入的位置。因此dup_x1插入位置:1+1=2，即栈顶2个slot下面. dup_x2插入位置:1+2=3，即栈顶3个slot下面dup2_x1插入位置:2+1=3，即栈顶3个slot下面. dup2_x2插入位置:2+2=4，即栈顶4个Slot下面
    - pop:将栈顶的1个Slot数值出栈。例如1个short类型数值；pop2:将栈顶的2个S1ot数值出栈。例如1个double类型数值，或者2个int类型数值

37. 控制转义指令：程序流程离不开条件控制，为了支持条件跳转，虚拟机提供了大量字节码指令，大体上可以分为1)比较指令、2)条件跳转指令、3)比较条件跳转指令、4)多条件分支跳转指令、5)无条件跳转指令等。

38. 条件跳转指令：条件跳转指令通常和比较指令结合使用。**在条件跳转指令执行前，一般可以先用比较指令进行栈顶元素的准备，然后进行条件跳转**。条件跳转指令有: ifeq， iflt， ifle， ifne，ifgt，ifge，ifnull，ifnonnull。这些指令都接收**两个字节的操作数**，用于计算跳转的位置(16位符号整数作为当前位置的offset)。它们的统一含义为:**弹出栈顶元素，测试它是否满足某一条件，如果满足条件，则跳转到给定位置。**

39. 条件跳转指令总结:
    - 与前面运算规则一致:对于boolean、byte、char、short类型的条件分支比较操作，都是使用int类型的比较指令完成；对于long、float、double类型的条件分支比较操作，则会先执行相应类型的比较运算指令，运算指令会返回一个整型值到操作数栈中，随后再执行int类型的条件分支比较操作来完成整个分支跳转。
    - 由于各类型的比较最终都会转为int类型的比较操作，所以Java虚拟机提供的int类型的条件分支指令是最为丰富和强大的。

40. 比较条件跳转指令：比较条件跳转指令类似于比较指令和条件跳转指令的结合体，它将比较和跳转两个步骤合二为一。这类指令有: if_icmpeq、if_icmpne、if_icmplt、if_icmpgt、if_icmple、if_icmpge、if_acmpeq和if_acmpne。其中指令助记符加上“if_”后，以字符“i”开头的指令针对in型整数操作(也包括short和byte类型)，以字符“a”开头的指令表示对象引用的比较。这些指令都接收两个字节的操作数作为参数，用于计算跳转的位置。同时在执行指令时，栈顶需要准备两个元素进行比较。指令执行完成后，栈顶的这两个元素被清空，且没有任何数据入栈。**如果预设条件成立，则执行跳转，否则，继续执行下一条语句。**

41. 多条件分支跳转指令:多条件分支跳转指令是专为switch-case语句设计的，**主要有tableswitch和lookupswitch**。从助记符上看，两者都是switch语句的实现，它们的区别:
    - tableswitch要求多个**条件分支值是连续的**，它内部只存放起始值和终止值，以及若干个跳转偏移量，通过给定的操作数index,可以立即定位到跳转偏移量位置，**因此效率比较高**。
    - lookupswitch内部存放着各个**离散的case-offset对**，每次执行都要搜索全部的case-offset对，找到匹配的case值，并根据对应的offset计算跳转地址，**因此效率较低**。

42. 指令lookupswitch处理的是离散的case值，**但是出于效率考虑，将case-offset对按照case值大小排序**，给定index时，需要查找与index相等的case,获得其offset ,如果找不到则跳转到default。

43. 无条件跳转指令:目前主要的无条件跳转指令为goto。指令goto接收**两个字节的操作数**，共同组成一个带符号的整数用于指定指令的偏移量，指令执行的目的就是跳转到偏移量给定的位置处。如果指令偏移量太大，超过双字节的带符号整数的范围，则可以使用指令goto_w,它和goto有相同的作用，但是它接收4个字节的操作数，可以表示更大的地址范围。指令jsr、jsr_w、 ret虽然也是无条件跳转的，但主要用于try-finally语句，且已经被虚拟机逐渐废弃，故不在这里介绍这两个指令。

44. 抛出异常指令:athrow指令在Java程序中显示抛出异常的操作(throw语句)都是由athrow指令来实现。除了使用throw语句显示抛出异常情况之外，JVM规范还规定了许多运行时异常会在其他Java虚拟机指令检测到异常状况时自动抛出。例如，在之前介绍的整数运算时，当除数为零时，虚拟机会在idiv或ldiv指令中抛出ArithmeticException异常。

45. 注意：正常情况下，操作数栈的压入弹出都是一条条指令完成的。**唯一的例外情况是在抛异常时，Java虚拟机会清除操作数栈上的所有内容，而后将异常实例压入调用者操作数栈上**。如果使用throw new Exception()这种形式来抛出异常，那就会在代码中出现athrow指令，而在方法上使用throws语句声明该方法可能抛出的异常时，字节码中方法表中会出现一个属性Exceptions(属性列表)

46. 在Java虚拟机中，处理异常(catch语句)不是由字节码指令来实现的(早期使用jsr、ret指令)，而是采用**异常表**来完成的。如果一个方法定义了一个try-catch或者try-finally的异常处理，就会创建一个异常表。它包含了每个异常处理或者finally块的信息。**异常表保存了每个异常处理信息**。比如:起始位置；结束位置；程序计数器记录的代码处理的偏移地址；被捕获的异常类在常量池中的索引等(大于等于Start PC，小于End PC，Java中很多与区间有关的知识点均为左闭右开)。

47. **当一个异常被抛出时，JVWM会在当前的方法里寻找一个匹配的处理，如果没有找到，这个方法会强制结束并弹出当前栈帧，并且异常会重新抛给上层调用的方法(在调用方法栈帧)**。如果在所有栈帧弹出前仍然没有找到合适的异常处理，这个线程将终止。**如果这个异常在最后一个非守护线程里抛出，将会导致JVM自己终止**，比如这个线程是个main线程。不管什么时候抛出异常，如果异常处理最终匹配了所有异常类型，代码就会继续执行。在这种情况下，如果方法结束后没有抛出异常，仍然执行finally块，**在return前**，它直接跳到finally块来完成目标。

48. java虚拟机支持两种同步结构:**方法级的同步和方法内部一段指令序列的同步**，这两种同步都是使用monitor来支持的。

49. 方法级的同步:是**隐式的**，即无须通过字节码指令来控制，它实现在方法调用和返回操作之中。虚拟机可以从常量池中的**方法表结构中的ACC_SYNCHRONIZED**访问标志得知一个方法是否声明为同步方法;当调用方法时，调用指令将会检查方法的ACC_SYNCHRONIZED访问标志是否设置。如果设置了，执行线程将先持有同步锁，然后执行方法。最后在方法完成(**无论是正常完成还是非正常完成，这也意味着同步方法会自动添加异常表来专门处理同步锁无法正常释放的问题**)时释放同步锁。
    - 在方法执行期间，执行线程持有了同步锁，其他任何线程都无法再获得同一个锁。
    - 如果一个同步方法执行期间抛出了异常，并且在方法内部无法处理此异常，那这个**同步方法所持有的锁将在异常抛到同步方法之外时自动释放**。

50. 对于同步方法而言，当虚拟机通过方法表中的访问标识符判断是一个同步方法时，会自动在方法调用前进行加锁，**当同步方法执行完毕后，不管方法是正常结束还是有异常抛出，均会由虚拟机释放这个锁**。因此，对于**同步方法而言，monitorenter和monitorexit指令是隐式存在的，并未直接出现在字节码中**。

51. 同步一段指令集序列:通常是由java中的**synchronized语句块来表示**的。jvm的指令集有monitorenter和monitorexit两条指令来支持synchronized关键字的语义。当一个线程进入同步代码块时，它使用monitorenter指令请求进入。**如果当前对象的监视器计数器为0,则它会被准许进入，若为1,则判断持有当前监视器的线程是否为自己，如果是，则进入，否则进行等待，直到对象的监视器计数器为0,才会被允许进入同步块。**当线程退出同步块时，需要使用monitorexit声明退出。在Java虚拟机中，任何对象都有一个监视器与之相关联，用来判断对象是否被锁定，当监视器被持有后，对象处于锁定状态。指令monitorenter和monitorexit在执行时，都需要在操作数栈顶压入对象,之后monitorenter和monitorexit的锁定和释放都是针对这个对象的监视器进行的。(监视器即为类或对象的锁)

52. 为了保证在方法异常完成时monitorenter和monitorexit指令依然可以正确配对执行，**编译器会自动产生一个异常处理器，这个异常处理器声明可处理所有的异常**，它的目的就是用来执行monitorexit指令。



# **三、类的加载过程**
1. 在Java中数据类型分为基本数据类型和引用数据类型。**基本数据类型由虚拟机预先定义，引用数据类型则需要进行类的加载**。

2. 按照Java虚拟机规范，从class文件到加载到内存中的类，到类卸载出内存为止，它的整个生命周期包括如下7个阶段:**加载(Loading) -> 连接(Linking){校验(Verification) -> 准备(Preparation) -> 解析(Resolution)} -> 初始化(Initialization) -> 使用(Using) -> 卸载(Unloading)**

3. 所谓加载，简而言之就是将Java类的**字节码文件加载到机器内存中，并在内存中构建出Java类的原型 — 类模板对象(Class对象，用于反射机制)**。所谓类模板对象，**其实就是Java类在JVM内存中的一个快照**，JVM将从字节码文件中解析出的常量池、类字段、类方法等信息存储到类模板中，这样JVW在运行期便能通过类模板而获取Java类中的任意信息，能够对Java类的成员变量进行遍历，也能进行Java方法的调用。

4. 反射的机制即基于类的加载机制。如果JVM没有将Java类的声明信息存储起来，则JVM在运行期也无法反射。加载阶段，即**查找并加载类的二进制数据，生成Class的实例**。在加载类时，Java虚拟机必须完成以下3件事情:
- 通过类的全名，**获取类的二进制数据流**
- **解析类的二进制数**据流为方法区内的数据结构(Java类模型)
- **创建java.lang.Class类的实例**，表示该类型。作为方法区这个类的各种数据的访问入口

5. **类模型**(类型信息)的位置：加载的类在JVM中创建相应的类结构，类结构会**存储在方法区**(JDK1.8之前:永久代;JDK1.8及之后:元空间)。

6. class实例的位置：类将.class文件加载至方法区后，会在堆中创建一个Java.lang.Class对象，用来封装类位于方法区内的数据结构，该Class对象是在加载类的过程中创建的，每个类都对应有一个class类型的对象，class类型对象中指向方法区中的类模型的指针。(**class对象存储在堆空间，类模型位于方法区**)

7. 数组类的加载：创建数组类的情况稍微有些特殊，**因为数组类本身并不是由类加载器负责创建**，而是由JVM在运行时根据需要而直接创建的，但**数组的元素类型仍然需要依靠类加载器去创建**。创建数组类(下述简称A)的过程:
    - 1.如果数组的元素类型是引用类型，那么就遵循定义的加载过程递归加载和创建数组A的元素类型;
    - 2.JVM使用指定的元素类型和数组维度来创建新的数组类。
    - 说明：如果数组的元素类型是引用类型，数组类的可访问性就由元素类型的可访问性决定。否则数组类的可访问性将被缺省定义为public。

8. 验证环节(Verification)：当类加载到系统后，就开始链接操作，验证环节是链接操作的第一步，它的目的是保证加载的字节码是合法、合理并符合规范的。验证的项目比较繁多，大致可分为以下几点：
    - **格式检查**：魔数检查；版本检查；长度检查
    - **语义检查**：是否继承final；是否有父类；抽象方法是否有实现
    - **字节码验证**：跳转指令是否指向正确位置；操作数类型是否合理
    - **符号引用验证**：符号引用的直接引用是否存在

9. 验证的内容则涵盖了类数据信息的格式验证、语义检查、字节码验证，以及符号引用验证等。**其中格式验证会和加载阶段一起执行**。验证通过之后，类加载器才会成功将类的二进制数据信息加载到方法区中。**格式验证之外的验证操作将会在方法区中进行**。链接阶段的验证环节虽然拖慢了加载速度，但是它**避免了在字节码运行时还需要进行各种检查**。(磨刀不误砍柴工)

10. 验证环节具体说明：
    - 1.格式验证:是否以魔数OxCAFEBABE开头，主版本和副版本号是否在当前Java虚拟机的支持范围内，数据中每一个项是否都拥有正确的长度等。
    - 2.Java虚拟机会进行字节码的语义检查，但凡在语义上不符合规范的，虚拟机也不会给予验证通过。
        - 是否所有的类都有父类的存在(在Java里，除了Object外，其他类都应该有父类)
        - 是否一些被定义为final的方法或者类被重写或继承了
        - 非抽象类是否实现了所有抽象方法或者接口方法
        - 是否存在不兼容的方法(比如方法的签名除了返回值不同，其他都一样，这种方法会让虚拟机无从下手调度;abstract情况下的方法，就不能是final的了)
    - 3.Java虚拟机还会进行字节码验证，字节码验证也是验证过程中最为复杂的一个过程。它试图通过对字节码流的分析，判断字节码是否可以被正确地执行。比如:
        - 在字节码的执行过程中，是否会跳转到一条不存在的指令
        - 函数的调用是否传递了正确类型的参数
        - 变量的赋值是不是给了正确的数据类型等
    - 4．校验器还将进行符号引用的验证。Class文件在其常量池会通过字符串记录自己将要使用的其他类或者方法。因此，在验证阶段，虚拟机就会检查这些类或者方法确实是存在的，并且当前类有权限访问这些数据，如果一个需要使用类无法在系统中找到，则会抛出NoClassDefFoundError,如果一个方法无法被找到，则会抛出NoSuchMethodError。此阶段在解析环节才会执行。


11. 栈映射帧(StackMapTable)就是在这个阶段，用于检测在特定的字节码处，其局部变量表和操作数栈是否有着正确的数据类型。但遗憾的是，**100%准确地判断一段字节码是否可以被安全执行是无法实现的，因此，该过程只是尽可能地检查出可以预知的明显的问题**。如果在这个阶段无法通过检查，虚拟机也不会正确装载这个类。但是，如果通过了这个阶段的检查，也不能说明这个类是完全没有问题的。在前面3次检查中，已经排除了文件格式错误、语义错误以及字节码的不正确性。但是依然不能确保类是没有问题的。

12. 准备环节(Preparation)，简言之，**为类的静态变量分配内存**，并将其初始化为默认值。当一个类验证通过时，虚拟机就会进入准备阶段。在这个阶段，虚拟机就会为这个类分配相应的内存空间，并设置默认初始值。该环节需注意：
    - 1．这里不包含基本数据类型的字段用static final修饰的情况，**因为final在编译的时候就会分配了，准备阶段会显式赋值**。
    - 2．注意**这里不会为实例变量分配初始化**，类变量会分配在方法区中(java8以前)，而实例变量是会随着对象一起分配到Java堆中。
    - 3．在这个阶段并不会像初始化阶段中那样会有初始化或者代码被执行。

13. 对于static final修饰的情况，需要注意一下几点(**解析环节没有字节码层面的代码执行操作**)：
    - 1、非final修饰的静态变量会在**准备阶段赋初始值(默认初始化)，然后在初始化阶段中的< clint>方法中显示赋值**
    - 2、静态常量(**基本数据类型、String类型字面量("XXX"这种情况)**)在编译阶段会初始化赋值，然后在准备环节就会显示赋值
    - 3、引用数据类型的静态常量，如new String("XXX")**需要执行相应字节码指令操作后才能对引用类型赋值的形式，都是在初始化中的< clint>中进行显示赋值**
    - 4、如果在static静态代码块中具有显示赋值操作，那肯定就是在初始化中的< clint>方法中显示赋值

14. 解析环节(Resolution)：**将类、接口、字段和方法的符号引用转为直接引用**。符号引用就是一些字面量的引用，和虚拟机的内部数据结构和和内存布局无关。比较容易理解的就是在Class类文件中，通过常量池进行了大量的符号引用。但是在程序实际运行时，只有符号引用是不够的，比如当如下println()方法被调用时，系统需要明确知道该方法的位置。

15. 初始化阶段(Initialization)：为**类的静态变量赋予正确的初始值**(即显示初始化)。类的初始化是类装载的最后一个阶段。如果前面的步骤都没有问题，那么表示类可以顺利装载到系统中。**此时，类才会开始执行Java字节码**。(即:到了初始化阶段，才真正开始执行类中定义的 Java程序代码，如静态代码块，对应的字节码层面的方法是< clint>)

16. 初始化阶段的重要工作是执行类的初始化方法:< clinit>()方法。该**方法仅能由Java编译器生成并由JVM调用**，程序开发者无法自定义一个同名的方法，更无法直接在Java程序中调用该方法，虽然该方法也是由字节码指令所组成。它是由**类静态成员的赋值语句以及static语句块合并产生的**。

17. 初始化阶段说明：在加载一个类之前，虚拟机总是会试图加载该类的父类，因此父类的< clinit>总是在子类< clinit>之前被调用。也就是说，父类的static块优先级高于子类。**Java编译器并不会为所有的类都产生< clinit>()初始化方法**。*哪些类在编译为字节码后，字节码文件中将不会包含< clinit>()方法?*
    - 一个类中并没有声明任何的类变量，也没有静态代码块时
    - 一个类中声明类变量，但是没有明确使用类变量的初始化语句以及静态代码块来执行初始化操作时
    - 一个类中包含static final修饰的基本数据类型的字段，这些类字段初始化语句采用编译时常量表达式(**在准备环节就完成显示初始化，对应的字段表下面的有属性ConstantValue**)

18. 对于< clinit>()方法的调用，也就是类的初始化，**虚拟机会在内部确保其多线程环境中的安全**，虚拟机会保证一个类的< clinit>()方法在多线程环境中被正确地加锁、同步，如果多个线程同时去初始化一个类，那么只会有一个线程去执行这个类的< clinit>()方法，其他线程都需要阻塞等待，直到活动线程执行< clinit>()方法完毕。正是因为函数< clinit>()带锁线程安全的，因此，如果在一个类的< clinit>()方法中有耗时很长的操作，就可能造成多个线程阻塞，引发死锁。并且这种死锁是很难发现的，因为看起来它们并没有可用的锁信息。如果之前的线程成功加载了类，则等在队列中的线程就没有机会再执行< clinit>()方法了。那么，当需要使用这个类时，虚拟机会直接返回给它已经准备好的信息。(**类初始化阶段引发的死锁问题很难发现**)

19. 主动使用:**Class只有在必须要首次使用的时候才会被装载，Java虚拟机不会无条件地装载Class类型**。Java虚拟机规定，一个类或接口在初次使用前，必须要进行初始化。这里指的“使用”，是指主动使用，主动使用只有下列几种情况:(即:如果出现如下的情况，则会对类进行初始化操作。而初始化操作之前的加载、验证、准备已经完成。)
    - 1．当创建一个类的实例时，比如使用new关键字，或者通过反射、克隆、反序列化。
    - 2．当调用类的静态方法时，即当使用了字节码invokestatic指令。
    - 3．当使用类、接口的静态字段时(final修饰特殊考虑)，比如，使用getstatic或者putstatic指令。(对应访问变量、赋值变量操作)
    - 4．当使用java.lang.reflect包中的方法反射类的方法时。比如: Class.forName("java.lang.String")
    - 5．当初始化子类时，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
    - 6．如果一个接口定义了default方法，那么直接实现或者间接实现该接口的类的初始化，该接口要在其之前被初始化。
    - 7．当虚拟机启动时，用户需要指定一个要执行的主类(包含main()方法的那个类)，虚拟机会先初始化这个主类。

20. 针对5补充说明:当Java虚拟机初始化一个类时，要求它的所有父类都已经被初始化，但是这**条规则并不适用于接口**：
    - 1.在初始化一个类时，并不会先初始化它所实现的接口；
    - 2.在初始化一个接口时，并不会先初始化它的父接口。因此，一个父接口并不会因为它的子接口或者实现类的初始化而初始化。只有当程序首次使用特定接口的静态字段时，才会导致该接口的初始化。

21. 针对7补充说明:JVM启动的时候通过引导类加载器加载一个初始类。这个类在调用public static void main(String[] agrs)方法之前被链接和初始化。这个方法的执行将依次导致所需的类的加载，链接和初始化。

22. 被动使用:除了以上的情况属于主动使用，其他的情况均属于被动使用。**被动使用不会引起类的初始化(但这并不意味着类没有进行加载)**。也就是说:并不是在代码中出现的类，就一定会被加载或者初始化。如果不符合主动使用的条件，类就不会初始化。
    - 1．当访问一个静态字段时，只有真正声明这个字段的类才会被初始化。即当通过子类引用父类的静态变量，不会导致子类初始化
    - 2．通过数组定义类引用，不会触发此类的初始化
    - 3．引用常量不会触发此类或接口的初始化。因为常量在链接阶段中的准备环节就已经被显式赋值了。
    - 4．调用ClassLoader类的loadClass()方法加载一个类，并不是对类的主动使用，不会导致类的初始化。(不同于反射！Class.forName())

23. 类、类的加载器、类的实例之间的引用关系：在类加载器的内部实现中，用一个**Java集合来存放所加载类的引用**。另一方面，一个**Class对象总是会引用它的类加载器**，调用Class对象的**getClassLoader()**方法，就能获得它的类加载器。由此可见，代表某个类的Class实例与其类的加载器之间为**双向关联关系**。一个**类的实例总是引用代表这个类的Class对象**。在Object类中定义了**getClass()方法**，这个方法返回代表对象所属类的Class对象的引用。此外，所有的Java类都有一个静态属性class，它引用代表这个类的Class对象。

24. 类的生命周期:当Sample类被加载、链接和初始化后，它的生命周期就开始了。当代表Sample类的Class对象不再被引用，即不可触及时，Class对象就会结束生命周期，Sample类在方法区内的数据也会被卸载，从而结束Sample类的生命周期。**一个类何时结束生命周期，取决于代表它的Class对象何时结束生命周期**。

25. **类的卸载：(条件极为苛刻)**
    - 1.启动类加载器加载的类型在整个运行期间是不可能被卸载的(jvm和jls规范)
    - 2.被系统类加载器和扩展类加载器加载的类型在运行期间不太可能被卸载，因为系统类加载器实例或者扩展类的实例基本上在整个运行期间总能直接或者间接的访问的到，其达到unreachable的可能性极小。
    - 3.被开发者自定义的类加载器实例加载的类型只有在很简单的上下文环境中才能被卸载，而且一般还要借助于强制调用虚拟机的垃圾收集功能才可以做到。可以预想，稍微复杂点的应用场景中(比如:很多时候用户在开发自定义类加载器实例的时候采用缓存的策略以提高系统性能)，被加载的类型在运行期间也是几乎不太可能被卸载的(至少卸载的时间是不确定的)。

26. 综合以上三点，**一个已经加载的类型被卸载的几率很小至少被卸载的时间是不确定的**。同时我们可以看的出来，开发者在开发代码时候，不应该对虚拟机的类型卸载做任何假设的前提下，来实现系统中的特定功能。

27. 判定一个常量是否“废弃”还是相对简单，而要判定一个类型是否属于“不再被使用的类”的条件就比较苛刻了。需要**同时满足**下面三个条件:
    - 该类所有的实例都已经被回收。也就是Java堆中不存在该类及其任何派生子类的实例。
    - 加载该类的类加载器已经被回收。这个条件除非是经过精心设计的可替换类加载器的场景，如OSGi、3SP的重加载等，否则通常是很难达成的。 
    - 该类对应的java.lang.Class对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。



# **四、类的加载器**
1. ClassLoader的作用:ClassLoader是Java的核心组件，所有的Class都是由classLoader进行加载的，ClassLoader负责通过各种方式将Class信息的二进制数据流读入JVM内部，转换为一个与**目标类对应的java.lang.class对象实例**。然后交给Java虚拟机进行链接、初始化等操作。因此，ClassLoader在整个装载阶段，**只能影响到类的加载**，而无法通过ClassLoader去改变类的链接和初始化行为。至于它是否可以运行，则由Execution Engine决定。

2. 类的加载分类:**显式加载vs隐式加载**
    - class文件的显式加载与隐式加载的方式是指JVM加载class文件到内存的方式。**显式加载指的是在代码中通过调用ClassLoader加载class对象**，如直接使用class.forName(name)或this.getclass().getclassLoader().loadClass()加载class对象。
    - 隐式加载则是不直接在代码中调用ClassLoader的方法加载class对象，**而是通过虚拟机自动加载到内存中**，如在加载某个类的class文件时，该类的class文件中引用了另外一个类的对象，此时额外引用的类将通过JVM自动加载到内存中。

3. *何为类的唯一性?*对于任意一个类，都需要由加载它的**类加载器和这个类本身**一同确认其在Java虚拟机中的唯一性。**每一个类加载器，都拥有一个独立的类名称空间**:比较两个类是否相等，只有在这两个类是由同一个类加载器加载的前提下才有意义。否则，即使这两个类源自同一个Class文件，被同一个虚拟机加载，只要加载他们的类加载器不同，那这两个类就必定不相等。

4. **类加载器的命名空间(在大型应用中，我们往往借助这一特性，来运行同一个类的不同版本)**
    - 每个类加载器都有自己的命名空间，命名空间由该加载器及所有的父加载器所加载的类组成
    - 在同一命名空间中，不会出现类的完整名字(包括类的包名)相同的两个类
    - 在不同的命名空间中，有可能会出现类的完整名字(包括类的包名)相同的两个类

5. 通常类加载机制有三个基本特征:
    - **双亲委派模型**:但不是所有类加载都遵守这个模型，有的时候，启动类加载器所加载的类型，是可能要加载用户代码的，比如JDK内部的ServiceProvider/ServiceLoader机制，这种情况就不会用双亲委派模型去加载，而是利用所谓的上下文加载器。
    - **可见性**:子类加载器可以访问父加载器加载的类型，**但是反过来是不允许的**。不然，因为缺少必要的隔离，我们就没有办法利用类加载器去实现容器的逻辑。
    - **单一性**:由于父加载器的类型对于子加载器是可见的，所以父加载器中加载过的类型，就不会在子加载器中重复加载。但是注意，类加载器“邻居”间，同一类型仍然可以被加载多次，因为互相并不可见。

6. 数组类的Class对象，不是由类加载器去创建的，而是在Java运行期JVM根据需要自动创建的。对于数组类的类加载器来说，是通过Class.getClassLoader()返回的，与**数组当中元素类型的类加载器是一样的**;如果数组当中的元素类型是基本数据类型，数组类是没有类加载器的(**JVM预定义了基本数据类型**)。

7. 站在程序的角度看，引导类加载器与另外两种类加载器(系统类加载器和扩展类加载器)并不是同一个层次意义上的加载器，引导类加载器是使用C++语言编写而成的，而另外两种类加载器则是使用Java语言编写而成的。由于引导类加载器压根儿就不是一个Java类，因此在Java程序中只能打印出空值。

8. 在JDK1.2之前，在自定义类加载时，总会去继承ClassLoader类并重写loadClass方法，从而实现自定义的类加载类。但是在J**DK1.2之后已不再建议用户去覆盖loadClass()方法，而是建议把自定义的类加载逻辑写在findClass()方法中**，从前面的分析可知，findClass()方法是在loadclass()方法中被调用的，当loadclass()方法中父加载器加载失败后，则会调用自己的findClass()方法来完成类加载，这样就可以保证自定义的类加载器也符合**双亲委托模式**。(**loadClass()方法中实现了双亲委派的逻辑**)

9. defineClass()方法是用来将byte字节流解析成JVM能够识别的Class对象(ClassLoader中已实现该方法逻辑)，通过这个方法不仅能够通过class文件实例化class对象，也可以通过其他方式实例化class对象，如通过网络接收一个类的字节码，然后转换为byte字节流创建对应的Class对象。**defineClass()方法通常与findClass()方法一起使用**，一般情况下，在自定义类加载器时，会直接覆盖
ClassLoader的**findClass()方法并编写加载规则，取得要加载类的字节码文件后转换成流，然后调用defineClass()方法生成类的Class对象**。

10. Class.forName()与ClassLoader.loadclass():
    - Class.forName():是一个静态方法,最常用的是Class.forName(String className);根据传入的类的全限定名返回一个class对象。该方法在将Class 文件加载到内存的同时,**会执行类的初始化。**
    - ClassLoader.loadclass():这是一个实例方法,需要一个ClassLoader对象来调用该方法该方法将Class文件加载到内存时,**并不会执行类的初始化**,直到这个类第一次使用时才进行初始化。

11. **双亲委派模型的定义**:如果一个类加载器在接到加载类的请求时，它首先不会自己尝试去加载这个类，而是把这个请求任务委托给父类加载器去完成，依次递归，如果父类加载器可以完成类加载任务，就成功返回。只有父类加载器无法完成此加载任务时，才自己去加载。

12. 双亲委派模型的本质规定了类加载的顺序是:**引导类加载器先加载，若加载不到，由扩展类加载器加载，若还加载不到，才会由系统类加载器或自定义的类加载器进行加载**。

13. 双亲委派机制优势:
    - 避免类的重复加载，确保一个类的**全局唯一性**。Java类随着它的类加载器一起具备了一种带有优先级的层次关系，通过这种层级关可以避免类的重复加载，当父亲已经加载了该类时，就没有必要子classLoader再加载一次。
    - **保护程序安全**，防止核心API被随意篡改

14. 双亲委派机制在java.lang.ClassLoader.loadClass(String,boolean)接口中体现。该接口的逻辑如下:
    - (1)先在当前加载器的**缓存中查找有无目标类，**如果有，直接返回。(当前加载器的缓存中若存在则说明该类按照双亲委派模型已经被加载过了)
    - (2)判断当前加载器的父加载器是否为空，如果不为空，则调用parent.loadClass(name，false)接口进行加载。(双亲委派的模型就隐藏在这第2和第3步中)
    - (3)反之，如果当前加载器的父类加载器为空，则调用findBootstrapClassOrNull(name)接口，让引导类加载器进行加载。
    - (4)如果通过以上3条路径都没能成功加载，则调用findClass(name)接口进行加载。该接口最终会调用java.lang.classLoader接口的defineClass系列的native接口加载目标Java类。

15. 如果在自定义的类加载器中重写java.lang.ClassLoader.loadClass(String)或java.lang.ClassLoader.loadClass(String, boolean)方法,抹去其中的双亲委派机制,仅保留上面这4步中的第1步与第4步，*那么是不是就能够加载核心类库了呢?*这也不行!因为JDK还为核心类库提供了一层保护机制。不管是自定义的类加载器，还是系统类加载器抑或扩展类加载器，最终都必须调用java.lang.ClassLoader.defineClass(String, byte[], int，int,ProtectionDomain)方法，而该方法会执行preDefineClass()接口，该接口中提供了对JDK核心类库的保护。

16. 双亲委托模式的弊端:检查类是否加载的委托过程是单向的，这个方式虽然从结构上说比较清晰，使各个ClassLoader的职责非常明确，但是同时会带来一个问题，即**顶层的ClassLoader无法访问底层的ClassLoader所加载的类**。通常情况下，启动类加载器中的类为系统核心类，包括一些重要的系统接口，而在应用类加载器中，为应用类。按照这种模式，应用类访问系统类自然是没有问障，但是系统类访问应用类就会出现问题。比如在系统类中提供了一个接口，该接口需要在应用类中得以实现，该接口还绑定一个工厂方法，用于创建该接口的实例，而接口和工厂方法都在启动类加载器中。这时，就会出现该工厂方法无法创建由应用类加载器加载的应用实例的问题。

17. 由于Java虚拟机规范并没有明确要求类加载器的加载机制一定要使用双亲委派模型，只是建议采用这种方式而已。比如在Tomcat中，类加载器所采用的加载机制就和传统的双亲委派模型有一定区别，当缺省的类加载器接收到一个类的加载任务时，首先会由它自行加载，当它加载失败时，才会将类的加载任务委派给它的超类加载器去执行，这同时也是Servlet规范推荐的一种做法。

18. 第一次破坏双亲委派机制:双亲委派模型的第一次“被破坏”其实发生在双亲委派模型出现之前，即DK 1.2面世以前的“远古”时代。由于双亲委派模型在JDK 1.2之后才被引入，但是类加载器的概念和抽象类java.lang.ClassLoader则在Java的第一个版本中就已经存在，面对已经存在的用户自定义类加载器的代码，Java设计者们引入双亲委派模型时不得不做出一些妥协，为了兼容这些已有代码，无法再以技术手段避免loadClass()被子类覆盖的可能性，只能在JDK1.2之后的java.lang.ClassLoader中添加一个新的protected方法findClass()，并引导用户编写的类加载逻辑时尽可能去重写这个方法，而不是在loadClass()中编写代码。上节我们已经分析过loadClass()方法，双亲委派的具体逻辑就实现在这里面，按照loadClass()方法的逻辑，如果父类加载失败，会自动调用自己的findClass()方法来完成加载，这样既不影响用户按照自己的意愿去加载类，又可以保证新写出来的类加载器是符合双亲委派规则的。**简单来说就是jdk1.2之前还没有引入双亲委派机制，所以jdk1.2之前就是破坏双亲委派机制的情况。**

19. 第二次破坏双亲委派机制:双亲委派模型的第二次“被破坏”是由这个模型自身的缺陷导致的，双亲委派很好地解决了各个类加载器协作时基础类型的一致性问题(越基础的类由越上层的加载器进行加载)，基础类型之所以被称为“基础”，是因为它们总是作为被用户代码继承、调用的API存在，但程序设计往往没有绝对不变的完美规则，如果有**基础类型又要调用回用户的代码**，那该怎么办呢?这并非是不可能出现的事情，一个典型的例子便是JNDI服务，JNDI现在已经是Java的标准服务，它的代码由启动类加载器来完成加载(在JDK1.3时加入到rt.jar的)，肯定属于Java中很基础的类型了。但JNDI存在的目的就是对资源进行查找和集中管理，它需要调用由其他厂商实现并部署在应用程序的ClassPath下的JNDI服务提供者接口(Service Provider Interface，SPI)的代码，现在问题来了，启动类加载器是绝不可能认识、加载这些代码的，那该怎么办?为了解决这个困境，Java的设计团队只好引入了一个不太优雅的设计:**线程上下文类加载器(Thread Context ClassLoader)**。这个类加载器可以通过java.lang.Thread类的setContextClassLoader()方法进行设置，如果创建线程时还未设置，它将会从父线程中继承一个，如果在应用程序的全局范围内都没有设置过的话，那这个类加载器默认就是应用程序类加载器。有了线程上下文类加载器，程序就可以做一些“舞弊”的事情了。JNDI服务使用这个线程上下文类加载器去加载所需的SPI服务代码，这是一种父类加载器去请求子类加载器完成类加载的行为，这种行为实际上是打通了双亲委派模型的层次结构来逆向使用类加载器，已经违背了双亲委派模型的一般性原则，但也是无可奈何的事情。Java中涉及SPI的加载基本上都采用这种方式来完成，例如JNDI、JDBC等。默认上下文加载器就是应用类加载器，这样**以上下文加载器为中介，使得启动类加载器中的代码也可以访问应用类加载器中的类**。

20. 第三次破坏双亲委派机制:双亲委派模型的第三次“被破坏”是由于用户对**程序动态性**的追求而导致的。如:代码热替换(Hot Swap〉、模块热部署(Hot Deployment)等。IBW公司主导的JSR-291(即OSGi)实现模块化热部署的关键是它自定义的类加载器机制的实现，**每一个程序模块(OSGi中称为Bundle)都有一个自己的类加载器**，当需要更换一个Bundle时，就把Bundle连同类加载器一起换掉以实现代码的热替换。在OSGi环境下，类加载器不再双亲委派模型推荐的树状结构，而是进一步发展为更加复杂的网状结构。

21. 热替换的实现:**热替换是指在程序的运行过程中，不停止服务，只通过替换程序文件来修改程序的行为**。热替换的关键需求在于服务不能中断，修改必须立即表现正在运行的系统之中。基本上大部分脚本语言都是天生支持热替换的，比如: PHP，只要替换了PHP源文件，这种改动就会立即生效，而无需重启web服务器。但对Java来说，热替换并非天生就支持，如果一个类已经加载到系统中，通过修改类文件，并无法让系统再来加载并重定义这个类。因此，在Java中实现这一功能的一个可行的方法就是灵活运用ClassLoader。注意:由不同ClassLoader加载的同名类属于不同的类型，不能相互转换和兼容。

22. 沙箱安全机制：保证程序安全；保护Java原生的JDK代码Java安全模型的核心就是Java沙箱(sandbox)。什么是沙箱?**沙箱是一个限制程序运行的环境。沙箱机制就是将Java代码限定在虚拟机(JVM)特定的运行范围中，并且严格限制代码对本地系统资源访问**。通过这样的措施来保证对代码的有限隔离，防止对本地系统造成破坏。沙箱主要限制系统资源访问，那系统资源包括什么?CPU、内存、文件系统、网络。不同级别的沙箱对这些资源访问的限制也可以不一样。所有的Java程序运行都可以指定沙箱，可以定制安全策略。

23. *为什么要自定义类加载器?*
    - **隔离加载类**：在某些框架内进行中间件与应用的模块隔离，把类加载到不同的环境。比如:阿里内某容器框架通过自定义类加载器确保应用中依赖的jar包不会影响到中间件运行时使用的jar包。再比如: Tomcat这类web应用服务器，内部自定义了好几种类加载器，用于隔离同一个web应用服务器上的不同应用程序。
    - **修改类加载的方式**：类的加载模型并非强制，除Bootstrap外，其他的加载并非一定要引入，或者根据实际情况在某个时间点进行按需进行动态加载
    - **扩展加载源**：比如从数据库、网络、甚至是电视机机顶盒进行加载
    - **防止源码泄漏**：Java代码容易被编译和篡改，可以进行**编译加密**。那么类加载也需要自定义(**类加载解密**)，还原加密的字节码。
    - 注意：在一般情况下，使用不同的类加载器去加载不同的功能模块，会提高应用程序的安全性。但是，如果涉及Java类型转换，则加载器反而容易产生不美好的事情。在做Java类型转换时，只有两个类型都是由同一个加载器所加载才能进行类型转换，否则转换时会发生异常。