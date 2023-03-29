## Intel 80386 Notes Book

---

#### **Secation Ⅰ - Base Model**

##### **Memory struct:**

​	2^32-1=4G

​	平坦地址空间是由最多4G字节组成的单个数组。pointer_32

​	段地址空间可以由16,383个线性地址空间组成，每个4G。pointer_16(Segement)+pointer_32(Move 32)

​	程序设计者可以任意选择存储模式，两种模式均提供存储器保护。

##### **Type of Value**

​	字节/字（高8，低8）/双字

​	整形

​	短指针 32 bit Logical Pointer Adress，段内偏移，用于平坦模式和段模式

​	长指针 48 bit Logical Pointer Adress，系统设计者设计选择段模式时采用

##### **Register**

​	16个应用寄存器(其他不予介绍),

​	*通用寄存器*：32bit 用于数学计算与逻辑运算

​		EAX,EBX,ECX,EDX,EBP,ESP,ESI,EDI

​		允许互换使用，存储逻辑和算数操作数，也可以互换的用于地址计算。

​		**ESP不可作为索引操作数**

​		为兼容8086和80286 16bit 操作，每个寄存器的低16bit 可以单独使用

​		AX,BX,CX,DX,BP,SP,SI,DI

​		同样，AX,BX,CX,DX 可以单独处理8 bit寄存器

​		AH,BH,CH,DH;

​		AL,BL,CL,DL

![image-20230322170723350](C:\Users\TangBao\AppData\Roaming\Typora\typora-user-images\image-20230322170723350.png)

​	

​	*段寄存器*：允许系统设计者使用平坦模式或者段模式。6个寄存器决定了任何时候那段存储器可以被寻址

​	CS,DS,SS,ES,FS,GS

​	代码段 - CS

​	堆栈操作 - SS 可以显示加载，允许程序动态定义堆栈

​	DS,ES,FS,GS允许声明4个数据段，都可被正在执行的程序寻址

​	任何时间，段存储寄存器可以关联六个存储器段

​	状态和指令寄存器：80386处理器的一些特征

​	![image-20230322170736725](C:\Users\TangBao\AppData\Roaming\Typora\typora-user-images\image-20230322170736725.png)

**堆栈实现**

​	SS -堆栈寄存器   <= 4G

​	ESP -堆栈指针寄存器 ， *指向向下增长的堆栈TOS顶部* ， PUSH POP 中断 子程序调用返回隐式操作ESP

​	EBP - 堆栈帧基指针 访问堆栈中的数据结构，变量和动态分配的工作空间最好的选择

![image-20230322171411334](C:\Users\TangBao\AppData\Roaming\Typora\typora-user-images\image-20230322171411334.png)

**标志位寄存器**

​	32 bit Register EFLAGS，下16bit 可以用作80286、8086

​	标志位可以分为3组：状态标志位、控制标志位、系统标志位

![image-20230322171606091](C:\Users\TangBao\AppData\Roaming\Typora\typora-user-images\image-20230322171606091.png)	状态标志位：

​	OF,SF,ZF,AF,PF

​	控制标志位

​	DF

***指令指针**

​	EIP包含代码段下一个将执行的指令序列的地址偏移量

**异常中断**

| 0    | 除数错误            |
| ---- | ------------------- |
| 1    | 调试异常            |
| 2    | 不可屏蔽（NMI）中断 |
| 3    | 中断点              |
| 4    | INTO检测溢出        |
| 5    | BOUND越界           |
| 6    | 非法操作码          |
| 7    | 协处理器不可得      |
| 8    | 双精度异常          |
| 9    | 协处理器段溢出      |
| 10   | 非法任务状态段      |
| 11   | 段缺失              |
| 12   | 堆栈错误            |
| 13   | 通用保护            |
| 14   | 页错误              |
| 15   | （保留）            |

#### Section Ⅱ System Register



**寄存器分类**

​	EFLAGS 标志寄存器

​	Memory Mangement Register 内存管理寄存器

​	Control Register 控制寄存器

​	*Debug Register 调试寄存器*

​	*Test Register 测试寄存器*



**Memory Mangement Control Register**

​	4个特定寄存器寻址特别的数据结构，用来实现段式内存管理1.

​	**GDTR (Global Descriptor Table Register)**全局描述表寄存器

​	**LDTR(Local Descriptor Table Register)**局部描述表寄存器

​	寄存器指向GDT,IDT

​	**IDTR(Interrput Descriptor Table Register)**中断描述符表寄存器

​	指向IDT表

​	**TR(Task Register)**任务寄存器

​	指向当前任务信息存放处

**Control Register**

​	CR0,CR2,CR3可被程序员通过特殊的MOV指令进行*访问*

​	**CR0**控制系统的整体运行

​		其中的bit EM,ET,MP,*PE*,PG,TS

​		PE(Protection Enable)保护模式使能位，让处理器工作在保护模式下，复位PE将返回到实模式

​		PG(Page Enable)指明处理器是否通过页表将线性地址转换到物理地址

​	**CR2,CR3**

​		CR2，处理缺页中断时使用，存放缺页中断产生的缺页异常地址

​		CR3，只有PG位被打开时，使用，用于存放当前页目录表

​	**Debug / Test Register**

​		Debug 用于调试中断，Test 用于测试TLB中的地址转换信息。几乎无用

​	

##### **System Instrucations**



系统指令主要功能

​	1.检测指针参数(Verification of Pointer parameters)
​		ARPL - adjust PRL

​		LAR - load Access Limit - 加载访问权限

​		LSL - laod Segement Limit  - 加载**段界限**

​			**段界限即为段内最大偏移值**

​		VERR - Verify for Reading - 读检验

​		VERW - Verify for Writing -  写检验

​	2.寻址描述符表(Addressing Descriptor tables)

​		LLDT,SLDT,LGDT,SGDT

​		分别对饮光加载,存储全局/局部描述符表

​	3.多任务(MultiTasking )

​		LTR,STR 加载存储任务寄存器

​	4. 协处理器和多处理器(Coprocessing and Multiprocessing)

​		CLTR- 清楚任务转换标志

​		ESC- 转译指令

​		WAIT- 等待协处理器空闲

​		LOCK- 引发总线锁信号

​	5. 输入输出(Input and Output)

​		IN,OUT,INS,OUTS

​		INS/OUTS - string

​	6. 中断控制(Interrupt Control)

​		CLI - 清除中断允许标志位

​		STI - 设置中断允许标志位

​		LIDT - 加载中断描述符表寄存器

​		SIDT -  存储中断描述符表寄存器	

​    7.调试(Debuging)
​		MOV - 向调试寄存器中输入或者输出

​	8.系统控制

​		HLT - 处理器挂起

#### **Section Ⅲ -Memory Control**



​	80386 转换逻辑地址到物理地址两步骤：

​		转化逻辑地址(段式地址)到线性地址，而后将线性地址转化为物理地址.

​	**Segemnt Translation**

​	逻辑地址转化为线性地址

​	Using struct of  deate

​	1.描述符

​		由编译器、连接器、加载器、或者操作系统生成

​		段描述符字段：基址(BASE)，描述一个段在线性4G地址空间中的位置

​			界限(LMIT)决定一个段的大小。参考粒度位(Granularity bit)决定界限值的倍数

​			即其位为1时,界限值被解析为4K一个单元。但大小为一个字节时，为1M

​			类型(TYPE)描述不同类型的描述符·

​			

​	2.描述符表

​		存在两个描述符表

​		一个全局描述符表一个局部描述符表,最多是8192(2^13)便是处理器是不会使用全局描述符的第一个的 (INDEX=0)。处理器用GDTR,LDTR来定位内存中的全局描述符表和当前的局部描述符表。这些寄存器存储了这些表的线性地址的基质和段长界限。指令**LGDT和SGDT**是用业访问全局描述符表寄存器的,而指令**LLDT和SLDT**是用来访问局部描述符表寄存器的。



​	3.选择子

​		选择描述表和该表中索引一个描述符的。选择子可以作为指针变量的一部分，从而对应用程序员是可见的，但一般是由连接加载器来设置。



​	4.段寄存器

​	