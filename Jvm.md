## Jvm

### Jvm Memory Model

#### 程序计数器 

> 概述: 也被称为Pc寄存器 

> 作用及其特点:
>
> - 当前线程所执行的字节码行号指示器
> - 线程执行Native方法, 程序计数器中的值是undefined
> - 不会出现OutOfMemoryError情况

#### Java栈 

> Java栈是Java方法执行的内存模型

> 每一个栈帧对应一个方法, 栈帧中包括:
>
> - ==局部变量表==: 存储方法中的局部变量, 基础-> 值, 引用类型 -> 对象引用. 大小在编译器确定, 不用改变
> - ==操作数栈==: 用来对象表达式进行求值.
> -  ==指向当前方法所属的类的运行时常量池的引用==: 方法运行时可能需要类中的常量
> - ==方法返回地址==: 执行完一个方法后, 要返回之前调用它的地方

注: ==本地方法栈和Java栈不同==

#### 堆

> 所有线程共享的一块内存区域, 用于存放对象实例, 虚拟机启动时创建.

> 堆分为两个区域:
>
> - 新生代
>   - eden区(占用80%)
>   - s0(From Survivor, 占用10%)
>   - s1(To Survivor, 占用10%)
> - 老年代

#### 方法区

> 方法区和堆一样是线程共享的. 存放每个类的信息, 静态变量, 常量以及编译器编译后的代码等

#### 运行时常量池

> 运行时常量池是方法区的一部分, Class文件中除了有类的版本, 字段, 方法, 接口等信息. 
>
> 还有常量池, 用于存放编译期生成的各种字面量和符号引用, 类加载后进行运行时常量池

> 具有动态性,  运行期间也可以将新的常量放入运行时常量池

#### 疑问

##### 堆区域为什么需要分代?

==分代目的是为了性能优化, 提高GC的效率==

##### 新生代为什么分区?

==不分区, 进行一次Minor GC, 老年代会被快速填满, 会触发Full GC, 因为老年代的内存空间远大于新生代, 进行了一次Full GC的事件比Minor GC 的时间长.==

==Survivor区作用是减少被送往老年代的对象, 进而减少Full GC的发生. Survivor的预筛选保证, 只有经历16次Minor GC还能在新生代中存活的对象, 才会被送到老年代.==

##### 堆如何分配内存?

==空闲列表 -- 指针碰撞==

指针碰撞: 假设Java堆中内存是规整的, 用过的内存放一边, 空闲的放一边, 中间放个指针作为分界点的指示器, 所分配内存就仅仅是把哪个指针向空闲空间那边挪动一段与对象大小相等的举例.这叫==指针碰撞==

空闲列表: 有一个列表, 其中记录中哪些内存块有用, 在分配的时候从列表中找到一块足够大的空间划分给对象实例, 然后更新列表中的记录. 这就叫做==空闲列表==

##### 对象引用如何找到堆中的对象实例?

==句柄访问--直接指针访问==

句柄访问: Java堆中会划分出一块内存作为句柄池. 引用变量中存储的就是对象的句柄地址, 而句柄中包含了对象==实例数据==和==类型数据==各自的具体地址信息.

直接指针访问: 引用变量中存储的就直接是对象地址了

优缺点:

- 使用句柄访问的优点是, 引用变量中存储的是稳定的句柄, 不会随着对象的移动而改变
- 使用直接指针访问的优点是, 速度更快, 节省了一次指针定位的开销, 但对象移动时, 需要改变引用变量地址



### GC

GC需要完成的三件事?

> 那些内存需要回收? 
>
> - 引用计数法
> - 可达性分析法

> 什么时候回收? 

> 如何回收? (收集器)
>
> - 回收策略
> - 垃圾收集器
>   - Serial收集器

#### 那些内存需要回收?



##### 根搜索法

- 概念:

​	犹豫引用计数法的缺陷, JVM采用一种新的算法, 根搜索法. 设立若干种根对象, 当任何一个根对象到某一		个对象均不可达时, 则认为这个对象是可以被回收的

- 可达性分析:

​	从根(GC Roots)的对象作为起始点, 向下搜索, 走过的路径称为 "==引用链==", 当一个对象到GC Roots没有可用的引用链相连, 则证明这个对象是不可用的.

- 根:

  GC Roots的对象有一下几种:

   - 栈中引用的对象
  - 方法区中的静态变量
  - 方法区中的常量引用的对象(全局变量)
  - 本地方法栈中JNI(Native方法)引用的对象

##### 引用计数法

对象添加一个引用计数器. 每当一个地方引用它时, 计数器加1, 失效时减1

缺点:

​	引用和去引用伴随加法和减法, 影响性能

​	无法解决对象循环引用

#### 如何回收?

##### 回收策略

###### 标记-清除

垃圾回收分为两个阶段: ==标记阶段== , ==清除阶段==

- 标记阶段: 通过根节点, 标记所有从根节点开始的可达对象, 具体做法: ==遍历所有的GC Roots, 然后将所有GC Roots可达的对象标记为存活对象==
- 清除阶段: 遍历堆中所有的对象, ==清除未被标记的对象.==

==注:该算法会暂停程序==

缺点:

	> 效率比较低(递归与全堆对象遍历), 导致 Stop the world时间变长, 交互式的程序不能接受

> 清理后的内存空间不是连续的, 这样分配大对象时, 内存不好找

###### 复制

将对象分为两块, 每次使用其中一块, 垃圾回收时, ==将正在使用的内存中存活对象复制到未使用的内存块中, 之后清理正在使用的内存块中的对象==

==不适合老年代, 适合做新生代==

==造成空间浪费, 但是新生代分区Eden区和Survivor区, 比例 8:1, 进而减少了空间浪费比例==

###### 标记-整理

垃圾回收分为两个阶段: ==标记阶段==, ==整理阶段==

==标记阶段==: 和标记-清除算法一致

==整理阶段==: 移动所有存活的对象, 且按照内存地址次序依次排序, 然后将==末端内存地址以后==的内存全部回收. 

缺点:

- 效率低, 需要遍历整个内存区域, 低于复制算法

##### 垃圾收集器

###### Serial

特征:

​	最基本, 历史最久的收集器, 采用==复制算法==的单线程收集器, 会暂停其他所有工作线程, 直到收集结束.

应用场景:

​	Serial收集器依然是虚拟机运行在Client模式下的默认新生代收集器

优势:

​	简单, 高效(相比于其他单线程收集器). 没有线程交互的开销.

设置参数:

​	"-XX:+UseSerialGC " : 添加该参数来显示的使用串行垃圾收集器

###### ParNew

特性:

​	ParNew是Serial收集器的多线程版本. 

应用场景:

​	ParNew许多运行在==Server模式==下虚拟机的首选==新生代==收集器, 它只能与==CMS收集器==配合工作

设置参数:

​	-XX:+UseConcMarkSweepGC : 指定使用CMS后, 会默认使用ParNew作为新生代收集器

​	-XX:+UseParNewGC : 强制使用ParNew

​	-XX:ParallelGCThreads : 指定垃圾收集的线程数,  ParNew默认开启和CPU数量一致的线程数

###### Parallel Scavenge

特性:

​	新生代收集器, 使用复制算法, 并行的多线程收集器

应用场景:

​	Parallel Scavenge收集器是虚拟机运行在Server模式下的默认垃圾收集器, 可以控制吞吐量.

​	适合在后台运算而不需要太多交互的任务.

设置参数:

​	-XX:+MaxGCPauseMillis : 控制最大垃圾收集==停顿时间==, 大于0的毫秒值.

​	-XX:GCTimeRatio : 设置垃圾收集时间占总时间的比率, 取值范围: 0 - 100, 相当于设置吞吐量的大小.

​	默认值是 1% -- 1 / (1 + 99), 即n = 99.

​	例如: -XX:GCTimeRatio=19, 设置垃圾收集时间占总时间的5% -- 1/(1+19)	

###### Serial Old

特性:

​	Serial Old是Serial的老年代版本, 使用==标记-整理==算法

应用场景:

- Client模式
- Server模式

###### Parallel Old

特性:

​	Parallel的老年代收集版本, 采用==标记-整理==算法

应用场景:

​	在注重吞吐量及CPU资源敏感的场合, 都可以优先考虑 Parallel Scavenge + Parallel Old 收集器

​	JDK1.6提供.

设置参数:

​	-XX:+UseParallelOldGC : 指定使用Parallel Old收集器



###### CMS

特性:

- 基于==标记-清除==算法实现, 特点: 并发收集, 低停顿.
- ==CMS是一种以获取最短回收停顿时间为目标的收集器==

应用场景:

​	与用户交互较多的场景, 停顿时间短, 适合 B/S系统.

运行过程:

- **初始标记(Cms initail mark)**

  标记一下GC Roots能直接关联到的对象, 速度很快, 需要Stop the world

- **并发标记(Cms concurrent mark)**

  并发标记阶段就是进行GC Roots Tracing的过程, 刚才产生的集合中标记出存活对象

- **重新标记(Cms remark)**

  重新标记阶段是为了**修正**并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录, 需要Stop the world.

- **并发清除(Cms concurrent sweep)**

  并发清除阶段会清除对象, 回收所有的垃圾对象.

缺点:

- **Cms对CPU资源敏感**

  **Cms默认启动的回收线程数 = (CPU数量+3) / 4**, 随着CPU数量的增加而下降 

- **Cms无法处理浮动垃圾**

  

- **Cms会产生大量空间碎片**

  使用标记-清除算法. 导致分配大对象时很麻烦, 不得不提前触发一次Full GC

  **-XX:+UseCMSCompactAtFullCollection** : 参数来使得CMS出现上面这种情况时不进行Full GC，而开启内存碎片的合并整理过程； 但合并整理过程无法并发，停顿时间会变长.

  **-XX:+CMSFullGCsBeforeCompaction** : 设置执行多少次不压缩的Full GC后，来一次压缩整理； 为减少合并整理过程的停顿时间； 默认为0，也就是说每次都执行Full GC，不会进行压缩整理.

###### G1



#### 疑问

##### Minor GC和Full GC的区别?

新生代GC(Minor GC): 新生代的垃圾收集动作, 回收速度比较快, 次数比较频繁

老年代GC(Major GC/Full GC): 老年代的垃圾收集动作, 速度是Minor GC的十倍.



### Class Loader

类加载的步骤: ==加载--验证--准备--解析-初始化--使用--卸载==等七个步骤. 并且加载--验证--准备--初始化--卸载这五个步骤是确定的.

#### 加载

##### 步骤

- **通过一个类的全限定类名获取定义此类的二进制字节流**
- **将这个字节流所代表的的静态存储结构转换为方法区的运行时数据结构 **
- **在内存中生成一个代表这个类的java.lang.Class对象, 作为方法区这个类的各种数的访问入口**

##### 特性

因为没有限制==如何获取==, ==哪里获取==二进制字节流? 常用:

- zip包中获取, jar, ear, war
- 网络中获取
- 运行时获取, 动态代理技术生成
- 其他文件生成, 典型: JSP应用
- 从数据库中读取

加载阶段可以使用系统提供的==引导类加载器==来完成, 也可以使用用户==自定义的类加载器==去完成.

数组类



#### 验证

校验一共完成四个阶段: ==文件格式校验, 元数据校验, 字节码校验, 符号引用校验==

##### 作用

==确保Class文件的字节流中包含的信息符合当前虚拟机的要求==

##### 文件格式校验

- 是否已**魔数0xCAFEBABE**开头
- ==主次版本号==是否在当前虚拟机处理范围内
- 常量池中的常量是否有不被支持的常量类型
- 指向常量的各种索引值中是否有指向不存在的常量或不符合类型的常量 
- CONSTANT_Utf8_info型的常量中是否有不符合UTF8编码的数据
- Class文件中各个部分及文件本身是否有被删除的或附加的其他信息

**这阶段是基于二进制字节流进行的, 只有通过了这个阶段验证, 字节流才会进入内存的方法区中进行存储,后面三个阶段都是基于方法区的存储结构进行的.**

##### 元数据校验

作用:

​	对字节码描述的信息进行==语义分析==

- 这个类是否有父类(除了java.lang.Object之 外 ,所有的类都应当有父类)。
- 这个类的父类是否继承了不允许被继承的类(被final修饰的类)。
- 如果这个类不是抽象类,是否实现了其父类或接口之中要求实现的所有方法。
- 类中的字段、方法是否与父类产生矛盾(例如覆盖了父类的final字段 ,或者出现不符合规则的方法重载,例如方法参数都一致,但返回值类型却不同等)

##### 字节码校验

作用:

**通过数据类和控制流的分析, 确定程序语义是合法的, 符合逻辑的.** 主要校验类中的方法体

校验事件:

- 保证任意时刻操作数栈的数据类型与指令代码序列都能配合工作,例如不会出现类似这样的情况:在操作栈放置了一个int类型的数据,使用时却按long类型来加载入本地变量表中。
- 保证跳转指令不会跳转到方法体以外的字节码指令上。
- 保证方法体中的类型转换是有效的,例如可以把一个子类对象赋值给父类数据类型,这是安全的,但是把父类对象赋值给子类数据类型,甚至把对象赋值给与它毫无继承关系、完全不相干的一个数据类型,则是危险和不合法的.

##### 符号引用校验

概述:

**发生于虚拟机将符号引用转换成直接引用的时候.**

**符号引用验证可以看做是对类自身以外(常量池中的各种符号引用)的信息进行匹配性校验**

校验:

- 符号引用中通过字符串描述的全限定名是否能找到对应的类。
- 在指定类中是否存在符合方法的字段描述符以及简单名称所描述的方法和字段。
- 符号引用中的类、字段、方法的访问性(private、protected、public、default)是否可被当前类访问

目的:

符号引用验证的目的是确保解析动作能正常执行

如果无法通过抛出异常: `java.lang.IncompatibleClassChangeError 子类` 

>  `java.lang.IllegalAccessError、 java.lang.NoSuchFieldError、 java.lang.NoSuchMethodError `

#### 准备

概述:

准备阶段是正式为类变量==分配内存==并设置==类变量初始值==, **这些变量所使用的内存都将在==方法区==中进行分配**

**进行内存分配的仅包括类变量, 而不包括==实例变量==, 实例变量将会在对象实例化时随着对象一起分配到Java堆**

初始值:

**`int--0, long--0L, boolean--false, float--0.0f, double--0.0d, short--(short)0, char--'\u0000'`**

**`byte--(byte)0, reference--null`**

#### 解析

概述:

**将常量池内的符号引用替换成直接引用的过程**

##### 符号引用

符号引用以一组符号来描述所引用的目标, 符号可以是==任何形式的字面量==, 只要使用时能无歧义的定位到目标.

各种虛拟机实现的内存布局可以各不相同 ,但是它们能接受的符号引用必须都是一致的,因为符号引用的字面量形式明确定义在Java虚拟机规范的Class文件格式中.

##### 直接引用

直接引用可以是直接指向目标的==指针==, ==相对偏移量==或是一个能间接定位到目标的==句柄==.

**直接引用是和虚拟机实现的内存布局相关的**

##### 触发

要求执行`anewarray、 checkcast、getfield、getstatic、instanceof、invokedynamic、invokeinterface、invokespecial、invokestatic、invokevirtual、ldc、ldc_w、multianewarray、new、putfield和putstatic `这16个用于操作符号引用的字节码指令之前, 对它们所使用的符号引用进行解析. 

虚拟机实现可以根据需要来判断到底是在类被加载器加载时就对常量池中的符号引用进行解析,还是等到一个符号引用将要被使用前才去解析它

虚拟机实现可以对第一次解析的结果进行缓存(在运行时常量池中记录直接引用,并把常量标识为已解析状态 )从而避免解析动作重复进行.

无论是否真正执行了多次解析动作,虚拟机需要保证的是在**同一个实体中,如果一个符号引用之前已经被成功解析过,那么后续的引用解析请求就应当一直成功;同样的,如果第一次解析失败了,那么其他指令对这个符号的解析请求也应该收到相同的异常** 

##### invokedynamic

对于`invokedynamic` 指令, 上面的规则不成立. 它是用于支持动态语言支持. 所对应的引用称为"**动态调用点限定符**", ==动态==: 必须等待程序实际运行到这条指令时,解析动作才发生.

##### 类和接口解析

在类D中, 把一个从未解析过的符号引用N解析为一个类或接口C的直接引用, 那个需要以下三个步骤:

- 如果C不是数组类型, 虚拟机将会把代表N的全限定类名传递给D的类加载器去加载C.
- 如果C是数组类型, 并且元素类型为对象, 那么N的描述符会**类似**`[Ljava/lang/Integer`的形式, 会按照上面的步骤加载数组元素类型. 如果N的描述符为`java.lang.Integer`, 接着由虚拟机生成一个代表此数组维度和元素的数组对象 
- 如果以上步骤没有出现任何异常, 那么C在虚拟机中实际上已经成为一个有效的类或接口了, 但在解析完成之前还要进行符号引用验证, 确定D是否具备对C的访问权限, 不具备抛出`java.lang.IllegalAccessError`

##### 字段解析

解析一个为被解析过的字段符号引用, 首先需要对字段所属的类或接口的符号引用进行解析. 

解析成功, 这个字段所属的类或接口用C表示. 虚拟机要求对C按照以下步骤进行后续字段搜索:

- 如果C本身包含了简单名称和字段描述符都与目标相匹配的字段, 则返回这个字段的直接引用, 查找结束.
- 否则, 如果在C中实现了接口, 将会按照继承关系从下往上递归搜索各个接口和它的父接口, 如果接口中包含简单名称和字段描述符都与目标相匹配的字段, 则返回这个字段的直接引用, 查询找结束
- 否则, 如果C不是`java.lang.Object`的话, 将会按照继承关系从下往上递归搜索器父类, 如果父类中包含了简单名称和字段描述符对于目标相匹配的字段, 则返回这个字段的直接引用, 查找结束
- 否则, 查找失败, 抛出`java.lang.NoSuchFieldError`异常

如果查找过程成功, 将会对这个字段进行权限验证, 如果发现不具备对字段的访问权限, 将抛出`java.lang.IllegalAccessError`异常

##### 类方法解析

类方法解析的第一个步骤与字段解析一样  

如果解析成功,我们依然用C表示这个类,接下来虚拟机将会按照如下步骤进行后续的类方法搜索 :

- 类方法和接口方法符号引用的常量类型定义是分开的,如果在类方法表中发现class_index中索引的C是个接口,那就直接拋出java.lang.IncompatibleClassChangeError异常。
- 如果通过了第1步 ,在类C中查找是否有简单名称和描述符都与目标相匹配的方法, 如果有则返回这个方法的直接引用,查找结束。
- 否则,在类C的父类中递归查找是否有简单名称和描述符都与目标相匹配的方法,如果有则返回这个方法的直接引用,查找结束。
- 否则,在类C实现的接口列表及它们的父接口之中递归查找是否有简单名称和描述符都与目标相匹配的方法,如果存在匹配的方法,说明类C是一个抽象类,这时查找结束,拋出`java.lang.AbstractMethodError`异常。
- 否则,宣告方法查找失败,拋出`java.lang.NoSuchMethodError`异常

##### 接口方法解析

接口方法也需要先解析出接口方法表的class_index项中索引的方法所属的类或接口的符号引用 ,如果解析成功,依然用C表示这个接口,接下来虚拟机将会按照如下步骤进行后续的接口方法搜索:

- 与类方法解析不同,如果在接口方法表中发现class_index中的索引C是个类而不是接口 ,那就直接拋出`java.lang.IncompatibleClassChangeError`异常。
- 否则,在接口C中查找是否有简单名称和描述符都与目标相匹配的方法,如果有则返回这个方法的直接引用,查找结束。
- 否则,在接口C的父接口中递归查找,直到`java.lang.Object`类(查找范围会包括Object类 )为止 ,看是否有简单名称和描述符都与目标相匹配的方法,如果有则返回这个方法的直接引用,查找结束。
- 否则,宣告方法查找失败,拋出`java.lang.NoSuchMethodError`异常

#### 初始化

##### 主动引用

- 遇到 `new, getstatic, putstatic, invokestatic` 这四条字节码指令时,进行初始化
  - `new : ` 关键字实例化对象
  - `getstatic, putstatic : ` 读取或设置一个类的静态字段
  - `invokestatic :`  调用一个类的静态方法
- 使用 `java.lang.reflect` 包的方法进行类的反射调用.
- 父类会优先被初始化.
-  主程序类会被初始化(`Main 方法的类`)
- Jdk1.7动态语言支持, `java.lang.invoke.MethodHandle`实例最后解析结果 `REF_getStatic, REF_putStatic, REF_invokeStatic`的方法句柄, 这个句柄所对应的类

这五种场景中的行为称为对一个类进行==主动引用==

##### 被动引用

除了主动引用的几种情况外, 其他引用类的方法 都不会触发初始化

- **子类引用父类的静态字段, 不会导致子类加载**

  ```java
  public class ClassLoaderInitTest {
  
      public static void main(String[] args) {
  
          System.out.println(SubClass.value);
      }
  }
  
  class SuperClass {
  
      public static int value = 1;
  
      static {
          System.out.println("SuperClass is init");
      }
  }
  
  class SubClass extends SuperClass {
  
      static {
          System.out.println("SubClass is init");
      }
  }
  ```

- **通过数组定义引用类, 不会触发此类的加载**

  ```java
  public class ClassLoaderInitTest {
  
      public static void main(String[] args) {
  
          SuperClass[] superClasses = new SuperClass[10];
      }
  }
  
  class SuperClass {
  
      static {
          System.out.println("SuperClass is init");
      }
  }
  ```

- **引用类的常量,该类不会被加载**

  ```java
  public class ClassLoaderInitTest {
  
      public static void main(String[] args) {
  
          System.out.println(SuperClass.value);
      }
  }
  
  class SuperClass {
  
      public static final int value = 1;
  
      static {
          System.out.println("SuperClass is init");
      }
  }
  ```

  常量在编译时被存储到==常量池==(当前ClassLoaderInitTest类)中, 转换成为ClassLoaderInitTest对自身常量池的引用. 和SuperClass不存在符号引用.

##### 概述

初始化阶段是执行类构造器`<clinit>()`方法的过程

##### clinit

概述:

**`<clinit>()`方法是由编译器自动收集类中的所有类变量的賦值动作和静态语句块(static{}块)中的语句合并产生的,编译器收集的顺序是由语句在源文件中出现的顺序所决定的**.

**静态语句块中只能访问到定义在静态语句块之前的变量,定义在它之后的变量,在前面的静态语句块可以賦值,但是不能访问**

特性:

`<clinit>()`方法与类的构造函数(或者说实例构造器`<init>()`方法)不同,它不需要显式地调用父类构造器,虚拟机会保证在子类的`<cinit>()`方法执行之前,父类的`<clinit>()`方法已经执行完毕。因此在虚拟机中第一个被执行的`<clinit>()`方法的类肯定是`java.lang.Object`

`<clinit>()`对于接口不是必须的.但接口与类不同的是,执行接口的`<clinit>()`方法不需要先执行父接口的`<clinit>()` 方法。只有当父接口中定义的变量使用时,父接口才会初始化。另外,接口的实现类在初始化时也一样不会执行接口的`<clinit>()` 方法 