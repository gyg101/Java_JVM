# Java_JVM
This is my Java JVM learn notes

JVM: java虚拟机
	一：上篇——内存与垃圾回收器
	二：中篇——字节码与类的加载
	三：下篇——性能监控与调优篇
  
  一: 上篇——内存与垃圾回收器
	架构: jvm依赖的架构: 栈架构/寄存器架构		栈架构
JVM的生命周期: 
  
		1.启动	通过引导类加载器(Bootstrap class loader)创建一个初始类(Initial Class)来完成
		2.执行	执行一个所谓的Java程序时，真正的执行的是一个叫做Java虚拟机的进程
		3.退出	程序正常结束；程序遇到错误或异常时终止运行；Runtime或System类调用exit()方法或Runtime调用half()方法

JVM的框架: 

		执行引擎: (字节)解释器 + JIT(java即时编译器)	前者是用 PC计数器 来依次编译每一行代码解释为本地机器指令; 后者是通过 寻找热点代码 进行即时编译为本地机器指令；
		GC: 垃圾回收器
主要的三款商用虚拟机JVM: 
  
		1) HotSpot JVM	特点: 热点代码探索技术,通过编译器与解释器协同工作		应用于服务器、桌面到移动端、嵌入式
		2) JRockit	 JVM	特点: 只有JIT编译器,全面的java运行时解决方案	专注于服务器端应用
		3) J9 JVM (IBM)	市场定位为: 应用于服务器、桌面到移动端、嵌入式

类加载器子系统:  
  
		1.加载阶段: 引导类加载器 - > 拓展类加载器 -> 系统类加载器  -> (用户自定义类加载器)
		2.链接	
			验证Verify	加密或解密
			准备prepare	静态变量与静态final常量的初始化不同
			解析reslover
		3.初始化阶段: 
			初始化: 初始化阶段就是执行类构造器方法<Clinit>()的过程	针对类变量
				<Clinit>()不同于类的构造器
			
引导类加载器:(或者是启动类加载器)Bootstrap Class Loader
		底层用C/C++实现, 加载D:\Program Files\Java\JRE\lib\rt.jar目录下的类
	自定义类加载器(Application ClassLoader): 包含拓展类加载器、系统类加载器、用户自定义类加载器
		用java实现，
	双亲委派机制: 
		原理: 当一个类收到类加载请求，并不会自己先去加载，而是把这个请求委托给父类加载器去执行；
			若该父类加载器还存在父类加载器，依次向上委托直到Bootstrap Class Loader(核心类加载器)
			若父类加载器可以完成类加载任务，则成功返回；否则子类加载器才会尝试自己去加载
		优点: 
			1.避免类的重复加载
			2.防止用户对核心API的恶意篡改(沙箱安全机制)
	ClassLoader的获取: 抽象类，为自定义类加载器的父类
  
		1) Class.forName().getClassLoader();
		2) Thread.currentThread.getContextClassLoader();
		3) ClassLoader.getSystemClassLoader().getParent();	
二: 运行时数据区: 

		(一)、堆、方法区: 一个进程中所共享的资源,有GC机制
		(二)、程序计数器、栈、本地栈: 每个线程所独有的资源,除pc计数器是没有GC和OOM机制外,其余都有OOM机制。
		1) PC寄存器: 用来存储指向下一条指令的地址; 也即将要执行的指令代码，由执行引擎读取下一条指令。 
			为什么需要PC寄存器? 	因为cpu需要在不同线程之间进行切换，若每个线程没有自己的PC寄存器,则线程切换后不能返回原来即将执行的位置。
		2) 虚拟机栈: 
			栈: 是运行时的单元；堆: 是内存的单元；
			Hotspot JVM基于栈结构: 优点: 跨平台性, 指令集小，编译器易于实现; 缺点是性能下降，实现同样的功能需要比寄存器结构更多指令
			特点: 线程所私有的，生命周期同线程
			栈的存储结构与运行原理: 
				栈的基本单位为栈帧, PC寄存器指向的栈帧为当前栈帧; 当前栈帧对应的方法为当前方法; 当前方法所处的类为当前类
			栈帧的内部结构: 
				局部变量表
				操作数栈
				动态链接(指向运行时常量池的方法引用)
				方法返回地址
				一些附加信息
		①局部变量表: 
			1) 看成一维数字数组[] 
				其实基本数据都可以看成数字(char -> ASCII码表, false用0表示,true用非0表示, 引用类型用返回地址表示)
			2) 最基本的存储单元是Solt(槽)	32位以内的类型占据一个Solt,64位(long/double)的占据两个Solt; Solt空间可重复利用
			3) 静态变量与局部变量的区别: 
				静态变量: 属于成员变量, 在创建类时已经对其进行了默认初始化，若需要可在静态代码块中进行显示赋值;
				局部变量: 属于每个栈帧，在声明该变量时必须对其进行初始化, 否则不能使用
			4) 局部变量表中第一个数据为this(若该方法是静态方法)或参数列表中的参数(若该方法有参数)
		②操作数栈: 
			作用: 用于存放操作数的结构，只支持push和pop操作
			Hotspot JVM采用数组结构存储, 在编译器即确定操作数栈的大小
		③动态链接: 
			作用: 将这些符号引用转换为调用方法的直接引用
		④方法返回地址: 
			存储调用该方法的PC寄存器的值。
			方法的结束: 1)正常结束; 2) 出现未处理的异常, 非正常退出
			无论哪种情况,方法退出后都返回到该方法被调用的位置。前者: 调用者的pc寄存器的值作为返回地址，即调用该方法的指令的下一条指令的地址。
      
静态绑定与动态绑定: 
			静态绑定: 被调用的方法在编译期可知, 且运行时不变	即早期绑定
			动态绑定: 	被调用的方法在编译期无法被确定下来			即晚期绑定
		
虚方法与非虚方法: 
			非虚方法: 
				def: 若方法在编译期就确定了具体的调用版本,该版本在运行时是不变的, 这样的方法称为非虚方法;
				静态方法、私有方法、final方法、实例构造器、父类方法都是非虚方法;
			虚方法: 其余方法都是虚方法
		 		invokestatic, invokespecial, 指令调用的方法称为非虚方法;
				invokevirtual 和 invokeinterface(除final)外称为虚方法
			方法重写的本质: 
				1.找到操作数栈顶的第一个元素所执行的对象的实际类型C
				2.在类型C中查找与常量中的描述符合简单名称都相符的方法，进行访问权限的校验; 通过则之间返回该方法的直接引用; 若校验失败则返回Java.lang.IllegalAccessError
				3.否则按照父类继承关系依次向上搜索和验证; 
				4.若始终没找到合适的方法, 则抛出Java.lang.AbstractMethodError异常;
			invokedynamic: 使java代码也具有动态指向的地址
			虚方法表: 为了快速查找到子类已重写的方法和未重写的方法。
      
有关虚拟机栈的面试题: 

			1.列举栈溢出的情况: StackOverFlowError		可通过-Xss命令动态拓展，但是还是有可能出现OOM
			2.调整栈的大小，就能够保证内存不出现溢出么？	不能，若线程请求分配的栈容量超出Java虚拟机栈允许的最大容量，依旧出现溢出
			3. 分配的栈大小越大越好么？为什么		不是，因为栈越大，可能会占用其余线程的内存空间，减少线程数
			4. 垃圾回收是否会涉及虚拟机栈？		不会，因为虚拟机栈和本地方法栈都只有Error, 只有进栈和出栈操作, 不存在GC
			5. 方法中定义的局部变量是否是线程安全的？	具体问题具体分析: 取决于该方法内的变量是否是自己产生，自己消亡

    3)本地方法栈: 
			主要是调用本地方法native method ;  " C stack"
			没有方法体, 与abstract方法不一样，不能与abstract关键字匹配出现 ,
			可以访问JAVA虚拟机栈, 
			
		4)堆空间: 
			几乎所有的对象和数组存储的空间; 主要考虑GC处理
			为同一进程中的所有线程所共有; 
堆空间的分类: 

			JAVA7及7之前,堆在物理上可以划分为: 新生代 + 老年代 + 永久代
			JAVA8及8之后,堆在物理上可以划分为: 新生代 + 老年代 + 元空间
			实际上设置的内存大小只是针对新生代和老年代: 
				-Xms600m	设置堆空间的起始大小; 	ms表示Memeory Start
				-Xmx600m	设置堆空间的最大内存	一般将起始大小与最大内存设置为同一值
			新生代: Eden + survivor 1(from) + survivor 2(to)
			-XX:NewRatio 	设置新生代与老年代的比例: 	默认是2
			-XX:SurvivorRatio	设置Eden区与Survivor 1区的比例，默认是8 但是也需要显式设置
			-XX:UseAdaptiveSizePolicy	关闭自适应的内存分配策略	(后两者基本不适用)
			-Xmn: 设置新生代的空间大小
		Summary: 
			1.关于s0与s1区谁是from谁是to的问题？ 当进行对象复制和转移时，谁是空谁是to
			2.关于GC频率问题: 8成左右的对象在Eden区被GC处理，很少对老年代中的对象进行垃圾回收; 几乎不对永久代或元空间进行GC

Minor GC 、Major GC与Full GC之间的区别: 
    
			前两者都是部分收集，后者是整堆收集; 
			Minor GC: 针对新生代的垃圾回收，触发条件: Eden区空间不足, Eden区发生YGC, 此时会将整个新生代进行GC处理
			Major GC: 针对老年代的垃圾回收: 触发机制: 当对象从老年代中消失时，会触发Full GC	目前只有CMS GC有单独收集老年代的行为。	·S0区或S1区中的对象达到阈值时，会将对象转移到老年代中
			Full GC: 针对整个java堆空间和方法区的GC机制，进行Full GC时会影响用户线程(STW)

		堆中为啥要分区？为了优化GC性能, 提高系统效率
		堆内存的分配策略(对象提升Promation规则): 
			1) 优先分配到Eden区
			2) 大对象直接分配到老年代	reason: 避免程序中出现过多的大对象
			3) 长期存活的对象分配到老年代
			4) 动态对象年龄判断		若Survivor区中相同年龄的对象大小的总和大于Survivor空间的一半, 年龄大于或等于该年龄的对象可以直接进入老年代，无需到达MaxTenuringThreshold阈值年龄
			5) 空间分配担保		-XX:HandlePromotionFailure

堆中所有的空间都是线程共享的么？

			不是; TLAB: Thread Local Allocation Buffer
			TLAB: 为了解决 线程中的操作同一地址的同时存在线程安全产生的分配速率问题
			JVM为每一个线程分配了一个私有缓存空间，提升性能的同时解决非线程安全问题, 提升了内存分配的吞吐量, 该内存分配策略称为快速分配策略
堆空间常用的参数设置: 

			-Xms: 初始堆内存空间大小(默认为物理内存的1/64)
			-Xms: 最大堆内存空间(默认为物理内存的1/4)
			-Xmn: 设置年轻代堆内存空间
			-XX: NewRatio 设置堆内存新生代与老年代的比例( 默认1:4)
			-XX: SurvivorRatio 设置堆内存Eden与Survivor的比例( 默认8:1)
			-XX: MaxTenuringThreashold: 设置新生代进入老年代的最大年龄
			-XX: +PrintGCDetails: 输出详细的GC处理日志
			-XX:HandlePromotionFailure: 是否设置空间分配担保
		堆不是对象分配的唯一内存: 
			还有可能根据对象的逃逸方法确定是否放在栈中: 若对象发生逃逸，则将该对象优化分配到栈中
			如何快速判断对象是否发生逃逸: 	判断new的该对象实体是否有可能在方法外被调用
		Summary: 开发中能使用局部变量的，就不要使用定义在方法外的。
逃逸分析: 代码优化		+DoEscapeAnalysis
    
			1.栈上分配: 对于未发生逃逸的对象，优先分配到栈空间汇总，减少GC次数提高内存效率	+DoEscapeAnalysis	
			2.同步消除: 对于未发生逃逸的对象进行同步操作，可以消除同步机制		
			3.标量替换: 对于未发生逃逸的对象，若该对象是聚合量，可以将该聚合量分解为其他聚合量和标量，再将其分配到栈空间	+EliminateAllocations 开启标量替换，允许将对象打散分配到栈上
java运行时数据区特点分析：

		Runtime Data Area:		(是否存在)GC	OOM		是否线程共享
		栈: 
			虚拟机栈: 		否		是		否
			本地方法栈: 	否		是		否
		PC寄存器: 		否		否		否
		堆: 			主要YGC		是		是
		方法区(元空间): 		几乎不		是		是
		
5)方法区: 		java规范中方法区的实现
			Hotspot虚拟机中, 可将永久代看成是接口, 元空间看成是实现这一接口的实例;
			(jdk7及之前)永久代与(jdk8及以后)元空间区别: 	
      
				1.名字不同, 内部结构也不同(前者是虚拟机中设置的内存, 后者是本地内存)
				2.设置永久代或元空间大小: 
					-XX:PermSize=10m; -XX:MaxPermSize=10m
					-XX:MetaspaceSize=10m; -XX:MaxMetaspaceSize=10m
				3.元空间的内存大小可以设置为固定值, 也可以根据所使用的情况进行动态拓展或压缩

内部结构: 		存储类型信息、常量、静态变量、即时编译器变异后的代码缓存等(类信息 + 运行时常量池)

			①类型信息: 	类、接口、父类、枚举、注解
			②域(Field)信息:	 访问修饰符、域类型、域名称、域值
			③方法(Method)信息: 方法名称 + 方法返回类型 + 方法参数的数量与类型 + 方法的修饰符 (+异常表)
			④运行时常量池: 
				1).class文件中的常量池: java中的字节码需要数据支持, 这种数据可以存储到常量池中，字节码包含了指向常量池的引用。 
				Summary: 常量池可以看成是一张表，虚拟机指令根据这张常量表找到要执行的类名、方法名、参数列表、字面量等类型
				2)运行时常量池: 
					用于存放编译期生成的各种字面量和符号引用，这些内容将在类加载后存放在方法区的运行时常量池中。
					与Class文件常量池的区别: 具有动态性
6) 元空间: 
			产生: 
				JDK7及之前永久代是存放在JVM虚拟机中, JDK7逐步将静态变量与字符串常量存储在堆空间
				JDK8以后取消了永久代的概念，元空间存储在本地内存，元空间中存放类信息及域信息、方法信息、JIT编译缓存、运行时常量池; 而jvm虚拟机中存放方法的引用
			Why: 
				为永久代设置内存大小是很难的
				永久代的性能调优是很难的(GC 常量池中不再使用的常量和类)
		7) 静态变量存放在哪？
		public class StaticTest{
			static ObjectHolder staticObj = new ObjectHolder();
			ObjectHolder instanceObj= new ObjectHolder();	
			void newObj(){
				ObjectHolder localObj = new ObjectHolder();
				System.out.println("Done");
			}
		}
summary:

			只要是对象实例，必然会在java堆中分配
			staticObj对象引用和instance对象引用存储在堆中: localObj存储在局部变量表中
8) 方法区的垃圾回收行为: 
			java虚拟机规范: 对方法区的约束是非常宽松的，不要求虚拟机在方法区中进行垃圾收集
			方法区的垃圾收集主要是两部分: 常量池中废弃的常量和不再使用的类型
对象实例化的几种方式: 方法区、栈、堆的联系

		1) new -> 最常见的方式; Xxx的静态方法(单例模式); XxxBuilder和XxxFactory的静态方法
		2) Class的newInstance() -> 反射的方式, 只能调用空参构造函数并且要求访问类型为Public (jdk9后被废弃)
		3) Construct的newInstance() -> 反射的方式, 可以调用空参和有参的构造器, 没有权限限制
		4) clone() -> 不调用构造器, 但是要求当前类实现Clonable接口, 重写clone()方法
		5) 反序列化 - >可以从文件、网络中获取一个对象的二进制流
		6) 第三方的工具: Objenesis
对象加载的步骤: 

		① 判断类是否加载(加载类元信息) 
		② 为对象分配内存
		③ 处理并发问题
		④ 属性的默认初始化(零值初始化)
		⑤ 设置对象头的信息
		⑥ 属性的显式初始化、代码块初始化、构造器初始化
对象的内存布局: 

		1) 对象头 		
			①运行时元数据	obj.hashCode();GC分代年龄; 锁状态标志 
			②类型指针	obj.getClass()存储在方法区中
			若该对象是数组对象, 还需要存储长度
		2)实例数据	对象真正存储的有效信息: 包括从各种类型字段(包括父类的字段)
		3) 对齐填充	占位符的作用
对象的访问方式: 

		1) 句柄访问	->对象指向更稳定些;  但需要堆中开辟一段空间存储句柄池(①指向实例数据的指针; ②指向对象类型的指针)
		2) 直接指针(Hotspot)	-> 效率稍高些, 虚拟机栈中的引用指向堆中的对象空间(包括实例数据), 然后由类型指针指向方法区中的对象类型	
	
直接内存 		-> (Jdk8之后)元空间存储的位置
				IO		NIO(Non-Blocking IO)	后者使用directByteBuffer
		传输工具(结构)	byte[] char[]	Buffer
		传输流		Stream		Channel		
	Java Process memory = java heap + java direct memory
	执行引擎: 		-> 解释器 + JIT编译器(此处属于后端编译，不同于.class文件编译为字节码文件, 称为前端编译)
		作用: 将高级语言解释/编译为机器指令, 传递给操作系统执行
工作过程: 

			1) 执行引擎在执行的过程中执行什么样的字节码完全取决于PC寄存器; 
			2) 每执行完一行字节码, pc寄存器将移向下一条即将执行的指令地址; 
			3) 在方法的执行过程中, 执行引擎可能通过栈帧的局部变量表中的对象引用准确定位到堆中的对象实例, 并通过对象头中的运行时元数据定位到目标对象的类型信息

java语言被称为半解释型半编译型语言	

			-> 解释器/JIT编译器 
			解释器: 通过解释器将java源码进行前端编译成.class字节码文件(抽象语法树)后, 通过逐行解释的方式执行。
			编译器: jvm将源代码直接编译成本地机器平台相关的机器语言。
		解释器逐条解释执行效率如此低下, 为何hotspot还保留呢？
			reason: 解释器可以在程序启动时就开始编译, 响应速度快; 其次当JIT在宕机时可以作为"应急方案"
				JIT编译器在启动后一段时间才开始热点代码搜索, 此时才开始一边编译一边执行, 效率提高
		何时采用JIT编译器: 		热点代码及探测方式	
			热点探测功能: 	基于计数器的热点探测功能
				1) 方法调用计数器  (统计方法的调用次数)
				2) 回边计数器	(统计循环体执行的循环次数)
JVM执行引擎的模式切换: 

			-Xint: 切换到只用解释器模式
			-Xcomp:切换到只用编译器模式
			-Xmixed: 切换到默认混合模式
			JIT的两个编译器: Client + Server
			Client(C1): 
				对字节码进行简单可靠的优化, 耗时短, 响应时间快, 但是效率低
				优化策略: 方法内联 + 去虚拟化 + 冗余消除
			Server(C2): 
				进行耗时较长的优化, 以及激进优化, 优化后的代码执行效率高, 但是启动时间长
				优化策略: 标量替换 + 栈上分配 + 同步消除
			Graal编译器: 
		AOT(Ahead Of Time)编译器(静态提前编译器): 与JIT编译器并列的编译器 , 主要借助Graal编译器
		String的不可变性: 
			位置: String pool 、heap
			结构: String pool 是一个固定大小的Hashtable, 默认长度是1009(jdk8之前)	修改StringTable的长度: -XX:StringTableSize
			分配位置: 完全相同的字符串字面量, 应该包含同样的Unicode字符序列, 并且必须是指向同一个String类实例
			
字符串的拼接: 

				1) 只要拼接符+ 的前后出现了变量, 则将对象存储在堆中, 相当于在堆空间中new String(), 具体内容为拼接的结果
				2) intern()方法: 判断常量池中是否存在"javaEEhadoop"字符串，若存在则返回该对象地址;若不存在，则在常量池中加载一个"javaEEhadoop"，并返回此对象的地址
				3) 执行细节: ① StringBuilder sb = new StringBuilder(); ② sb.append("javaEE"); ③ sb.append("hadoop"); ④ sb.toString(); 类似于new String("javaEEhadoop")
				4) 并非所有的字符串的拼接都是采用Stringbuilder的方式, 若拼接符左右两边是字符串常量或常量引用时，采用非StringBuilder方式
			字符串的拼接与StringBuilder的append()方法的效率比较: 
				1)  StringBuilder中的append()方法从始至终只创建一个StringBuilder对象; 而字符串的拼接底层每次都需要创建一个StringBuilder对象, 占用的内存空间更大;
				2) 字符串的拼接创建多个StringBuilder对象 , 当需要对其进行GC时, 需要更多的执行时间
如何保证变量s指向的是字符串常量池中的数据: 

				方式一: String s = "shkstart"; //字面量定义的方式
				方式二: 调用intern()方法
					 String s = new String("shkstart").intern();
					String s = new StringBuilder("shkstart").toString().intern();
			Q1: new String("ab");创建了几个对象？
				A1: 两个		-> 堆中一个new String对象; 字符串常量池中一个对象"ab"
			Q2: new String("a")  + new String("b");创建了几个对象？
				A2: 五个		-> 一个StringBuilder + 两个new String对象 +两个字符串常量池中的对象
			进阶: intern()的面试难题:	 jdk6和jdk7/8中，判断下列的输出
			① String s = new String("1");	s.intern(); String s2 = "1"; System.out.println(s == s2);//jdk6 : false ; jdk7/8: false
			② String s = new String("1") + new String("1"); s.intern(); String s2 = "11"; System.out.println(s == s2);//jdk6: false ; jdk7/8: true	jdk6中创建了一个新的对象"11"也就有了新的地址 ; jdk7/8中并没有创建一个新的new String("11"); 而是创建一个指向堆中的对象new String("11")的地址
			intern的空间效率测试: 
				对于系统需要创建大量的String对象时, 特别是有许多重复的对象, 调用string的intern()可以节省许多内存
垃圾回收: 

	def: 垃圾是指运行程序中没有任何指针指向的对象
	WHY: 	
		如果不对其进行垃圾回收，则这些对象所占据的内存空间会一直保留到程序运行结束，而且这些空间不能被其他对象所利用，甚至可能导致OOM
		内存迟早被消耗完；为了应对更为庞大的用户数据需求。内存泄露: 没有指针指向该对象，但是还不能对其进行垃圾回收的情况。
	WHO: 
		java堆是垃圾回收器的工作重点: 
		次数上来讲: 
			频繁收集Young区；较少收集Old区；基本不动Perm区(元空间)
	WHEN: 
		内存溢出
		内存泄露: 当没有指针指向不再使用的对象时，但是不能对进行垃圾回收的方式
	HOW: 
		标记阶段: 
			1) 引用计数算法: 
				引用计数器属性 
				优点: 实现简单垃圾对象便于辨识; 判断效率高, 回收没有延迟性
				缺点: 存储空间的开销; 运行时间的开销; 无法处理循环引用的情况, 会产生内存泄漏
			2) *可达性分析算法: 		-> 较好的解决了引用计数算法的循环引用问题
				GC Roots: 
					包括一下几类元素: 
					① 虚拟机栈中引用的对象: 各个线程被调用的方法中使用到的参数、局部变量等; 
					② 本地方法栈JNI(通常说的本地方法)引用对象
					③ 方法区中类静态属性引用的对象: java类的引用类型静态变量
					④ 方法区中常量引用的对象: 字符串常量池中的引用
					⑤ 被同步锁Synchronized所持有的对象; 基本数据类型的Class对象; 常驻的异常对象; 类加载器		其他对象"临时性"的加入: 分代收集与局部回收(Partial GC)
				Tips: 由于Root采用栈方式存储变量和指针, 所以如果一个指针, 它保存了堆内存里面的对象, 但是它又不在堆内存中, 那么他就是一个Root	
Finallization机制: 	-> java语言提供了对象终止(finalization)机制来允许开发人员提供了对象销毁之前的自定义处理逻辑

				finalize()方法允许在子类中进行重写，用于对象在被回收时进行资源释放(如关闭文件、套接字和数据库连接等) 	但永远不要主动地调用某个对象的finalize()方法, 应该交给垃圾回收机制调用
				由于finalize()方法的存在，虚拟机中的对象一般处于三种可能的状态: 
					1) *可触及的;	2) 可复活的: 	3) 不可触及的:finalize()方法只能被执行一次
				*判断一个对象是否可以回收，至少要经历两次标记过程	
					-> ① 若对象objA到GC Root没有引用链，则进行第一次标记 
					② 若子类中没有重写了finalize()方法或finalize()方法已经被执行一次，则将该对象判为不可触及; 
					若已重写finalize()方法且未被执行, 需要将该对象插入F-Queue队列中(finalizer线程触发)进行第二次标记, 判断是否与其他对象建立了联系，若有联系则移出"即将回收"集合; 否则直接回收
堆内存查看工具: MAT(Eclipse) 与Jprofiler

		清除阶段: 
			1) *标记-清除(Mark-Sweep)算法	-> 执行过程会STW
				标记阶段: 从根节点出发开始遍历并标记所有的可达对象
				清除阶段: 对堆内存进行线性遍历，对所有的未标记为可达对象进行垃圾回收
				优缺点: 基础且常见; 效率不算高; stw时用户体验差; 产生内存碎片
			2) *复制(Copy)算法		-> YGC中的S0区与S1区
				优点: 没有标记和清除阶段，执行效率高; 复制之后的内存空间是连续的，不会存在内存碎片; 	特别的, 所需复制的对象较少时, 复制算法较为理想
				缺点: 需要两倍的内存空间; 对于对象的复制需要维护region区的之间对象引用关系, 需要的内存开销和时间开销也不小; 
			3) *标记-压缩(Mark-Compact)算法
others: 

			1) 分代收集算法	-> 堆中的Young Gen: copy算法; Tenured Gen: Mark-Sweep算法或与Mark-Compact算法混合实现(eg: JVM中的CMS回收器)
			2) 增量收集算法	-> 由于STW时间过长 提出垃圾收集线程与应用程序线程交替执行(并发执行)	缺点: 造成系统吞吐量的下降	垃圾收集线程每次只收集一小片区域的内存空间, 接着切换到应用程序执行。依次往复, 直到垃圾收集完成。
			3) *分区算法	-> G1回收器 解决STW问题	缺点: 造成系统吞吐量下降(与低延迟相对应)
GC相关概念: 

		1) System.gc()	-> 默认情况下, 通过System.gc()或者Runtime.getRuntime().gc()的调用会显式触发Full GC, 同时对老年代和新生代进行回收, 尝试释放被丢弃的对象所占用的内存
			注: 免责声明: System.gc()无法保证对垃圾回收器的调用	但是如果采用System.runFinalization()会强制调用使用引用的对象的finalizae()方法
		2) 内存溢出与内存泄露: 
			OOM(OutOfMemoryError): 没有空闲内存, 且垃圾回收器也无法提供更多内存
			注: 在抛出OutOfMemoryError之前, 通常垃圾收集器会被触发, 尽其所能的清理出空间；也不是所有的情况都会触发垃圾回收器(eg: 存在超大对象)
			Memory Leak: 只有对象不能被程序用到了, 但GC又不能回收他们的情况, 才叫内存泄露(也叫内存渗漏)。内存泄露可能导致OOM
				eg: ① 单例模式: 持有对外部对象的引用
				② 一些提供close的资源未关闭导致内存泄露: DataSourse.getConnection(), Socket, io 连接必须手动关闭close, 否则不能被回收
		3) Stop The World		-> 普遍性: 与采用哪款GC无关系
			停顿产生时整个应用程序都会被暂停, 没有任何响应
			可达性分析算法中根节点(GC Roots)会导致所有的java执行线程停顿: 分析工作必须在一个能确保一致性的快照中进行(即GC Roots的数目不再改变)

		4) 程序的并发与并行: 
			并发(Concurrent): 在同一时间段内有多个线程在运行		单核cpu; 互相抢占资源
			并行(Parallel): 在同一时间点上有多个应用程序在运行		多cpu或单cpu多核处理器; 不互相抢占资源		适合科学计算
		GC中的并行与并发: 
			并行: 多条垃圾回收线程并行工作
			串行: 单线程执行
			并发: 用户线程与垃圾回收线程同时执行(不一定真是并行，可能存在STW)	eg: CMS, G1
		安全点与安全区域: 
			Safepoint: 只有在特定的位置才能停顿下来执行GC
			Q: 如何判断所有的线程到达最近的安全点停顿下来呢？
				抢占式中断: 目前没有JVM采用该方式
				主动式中断: 设置中断标志, 然后每个线程执行到safepoint时主动轮询该标志, 若为true则将该线程中断挂起
			Safearea: 			
				一段代码片段中，对象的引用关系不会发生改变，在这段区域的任何位置开始GC都是安全的
		5) 引用关系: 
			强引用: 不回收		强引用对象是可触及的，那垃圾回收器就永远不回收掉被引用的对象	是引起内存泄露的主要原因
			软引用: 内存不足即回收	只被软引用关联的对象在系统发生内存溢出异常之前，会将这些对象列进可回收范围内，进行二次回收；若内存还是不足时抛出OOM异常	应用场景: 高速缓存
			弱引用: 发现即回收		只被弱引用关联的对象只能存活到下一次垃圾收集发生之前	应用场景: 与软引用一样，都适用于存储可有可无的缓存数据
			虚引用: 对象回收跟踪		构造器: PlantomReference pf = new PlantomReference(obj, plantomQueue);一旦将对象进行回收，就会将此虚引用存放到引用队列中
			终结器引用: 用于实现finalize()方法	与虚引用类似, 可加入引用队列
垃圾回收器: 

		1) GC分类与性能指标说明: 
		① 分类: 
			按线程数分类: 串行垃圾回收器 + 并行垃圾回收器
			按工作模式分类: 并发式垃圾回收器 + 独占式垃圾回收器	应用程序与并发式垃圾回收线程交替工作，减少应用程序的停顿时间; 独占式垃圾回收器(STW)一旦运行, 就停止应用程序中的所有用户线程，直到GC过程结束
			按碎片处理方式分类: 压缩式垃圾回收器(Mark-Compact 指针碰撞) + 非压缩式垃圾回收器(Mark-Clean 空闲列表)
			按工作的内存区间分类: 年轻代垃圾回收器 + 老年代垃圾回收器
		② 性能指标的整体说明: 
			吞吐量: 程序执行时间占总运行时间的比例	总运行时间: 程序执行时间 + GC执行时间
			暂停时间(STW): 执行垃圾收集时，程序的工作线程被暂停的时间
			内存大小: JVM中堆所占的内存大小	前三者构成不可能三角
			垃圾收集开销: 吞吐量的补数，垃圾收集的时间占总运行时间的比例
			收集频率: 相对于应用程序的执行，收集操作的频率
			快速: 一个对象从诞生到被回收所经历的时间
		吞吐量与暂停时间: 
			吞吐量越高，程序运行速度越快; 对于交互式软件，低延迟是程序关注的重点
			吞吐量与低暂停时间是相互竞争的目标，一个GC只可能针对两个目标之一: 降低暂停时间的同时，会减少吞吐量；
			现在策略: 在最大吞吐量优先的情况下，降低停顿时间
经典的7款垃圾回收器: 

			串行垃圾回收器: Serial(Yong) ,Serial Old(Old)
			并行垃圾回收器: ParNew , Parallel Scavenge, Parallel Old
			并发垃圾回收器: CMS , G1
匹配关系与查看默认的垃圾回收器: 
			-XX:+PrintCommandLineFlags:
			Cmd命令行查看: jinfo -flag 相关垃圾回收器参数 进程ID
      
		Serial GC(串行垃圾回收器): 
			优点: 简单而高效	-在单核CPU环境中，没有线程交互的开销，效率高	
			应用场景: 适用于Client模式下交互较少的场景
			新生代: Serial GC 采用复制算法及Stop The World机制，串行回收; 老年代: Serial Old采用标记-整理算法及STW机制，串行回收
		ParNew GC(并行垃圾回收器): 
			优点: 在多核环境下效率高于串行垃圾回收器	-与CMS配合使用，在JDK9之后被移除
			配置使用对应的垃圾回收器: -XX:+UseParNewGC
		Parallel GC(吞吐量优先 并行垃圾回收器):
			Parallel Scavenge GC采用机制: 复制算法、并行回收、STW机制
			优点: 1)可控制的吞吐量		2) 自适应调节策略
			应用场景: 适合于后台运算而无需过多交互的任务, 常在服务器环境中使用-批量处理、订单处理、科学计算等
		CMS GC: 	
			Concurrent Mark Sweep第一款并发式垃圾回收器		复制算法与标记-清除算法	思考: 为何不采用标记-整理算法
			工作原理: 初始标记阶段(Mark) - 并发标记阶段 - 重新标记阶段 - 并发清除阶段
			优缺点: 1)并发收集	2) 低延迟		缺点: 内存碎片化; CMS收集器对cpu资源非常敏感(吞吐量下降); 无法处理浮动垃圾
			应用场景: 适用于与用户交互频繁的场景		
		Summary: 如何选择各种垃圾收集器？
			1)最小化使用内存与并行开销，选择Serial GC;
			2) 最大化应用程序的吞吐量，选择Parallel GC;
			3) 最小化GC的中断时间或停顿时间，选择CMS GC;	
注: CMS GC在jdk9之后被废弃, 在jdk14之后被删除

		G1回收器: 区域化分代式
			优势: 
				1) 并行与并发
				2) 分代收集	区分新生代与老年代
				3) 空间整合	region之间采用复制算法; 整体上来看是标记-整理算法
				4) 可预测的停顿时间模型(Soft real-time)	分区算法可以只选区部分区域进行垃圾回收, 缩小回收范围, 这样避免了全区的STW; G1可以根据允许的收集时间, 优先回收价值最大的Region; 保证在有限的时间内获得尽可能高的收集效率
			劣势: G1的进行垃圾收集产生的内存占用和程序执行时的额外执行负载都要高于CMS; 小内存(6-8G以下)应用上表现大概率弱于CMS
			G1参数设置与性能调优: 
				① 设置G1回收器: -XX:UseG1GC
				② 设置每个Region大小: -XX:G1HeapRegionSize
				③ 设置期望达到的最大GC暂停时间指标: -XX:MaxGCPauseMillis
应用场景: 
				1) 面向服务器端, 针对大内存和多处理器的机器
				2) 需要低GC延迟, 并具有大堆应用程序提供解决方案

			一、分区region: 化整为零
				Eden、Survivor、Old、Humongous
				一个region只可能属于一个角色, Humongous区主要用于存放大对象, 当对象的大小大于1.5region时, 就放在H区，若其为短期存在的大对象, 不适合放在Old区，可放在连续的H区 
			二、G1回收器垃圾回收过程: 
				YGC -> YGC + Concurrent Mark -> Mixed GC	可能存在单线程、独占式、高强度的Full GC, 作为指标评估失败保护机制, 即强力回收
				准备知识: 一个Region中的对象可能被多个Region(可能是E区可能是O区)中的对象所引用, 进行YGC时判断对象存活, 需要扫描整个java堆才能保证准确？降低minor gc效率 	-> Remembered Set每个region都有一个对应的记忆集(用于存储该区域中的对象被其他Region中的对象引用情况)，在每一个Reference类型数据写操作时, 会产生一个Write Barrier暂停中断操作
				1) 年轻代GC: 
					扫描根		(static变量指向的引用 + 正在执行的方法链表上的局部变量) + RSet记录的外部引用作为扫描存货对象的入口; 
					更新RSet		(RSet后可准确的反映对所在内存分段中的对象的引用)
					处理RSet		被老年代所指向的Eden区的对象被认为是存活的对象
					复制对象
					处理引用
				2) YGC + Concurrent Mark
					初始标记阶段; 根区域扫描; 并发标记; 再次标记; 独占清除(计算存活对象与GC回收比例, 排序,识别可以混合回收的区域 STW); 并发清理阶段 
				3) Mixed GC
					原则: 垃圾占内存分段的比例越高, 越会先被回收
					注: 此阶段除了回收所有的Yong Region, 还要回收部分的Old Region, Mixed GC也不是Full GC
				Summary: G1回收器回收部分Region, 停顿时间是用户可控制的；而考虑回收阶段与用户线程一起并发执行, 则是主要考虑低延迟垃圾收集器-ZGC的特性
如何选择垃圾收集器: 

					1) 优先调整堆的大小让JVM自适应完成;
					2) 当内存小于100M时, 使用串行收集器;
					3) 当应用场景是单核CPU、单机程序、没有停顿时间的要求, 可使用串行垃圾收集器
					4) 如果是多cpu、需要吞吐量、允许暂停时间超过1s，选择并行或JVM自己选择
					5) 如果是多cpu、追求低停顿时间、需快速响应(如延时不超过1s， 如互联网应用)，使用并发收集器CMS\G1(官方推荐G1,性能高)
				Think: 垃圾收集算法; 如何判断一个对象是否可回收; 垃圾收集器工作的基本流程
		GC日志: +XX:PrintGCDetails
		内存分析: [PSYoung]0k->100k(200k)
		日志查看工具: GCViewer、GCEasy
		令人震惊的、革命性的ZGC: 	与Shenandoah GC类似，主要考虑低延迟, 暂停时间与堆内存大小无关
			def: 一款基于Region分区、不设分代的, 使用了读屏障、染色指针、内存多重映射等技术实现可并发标记-压缩算法、以低延迟为首要目标的一款垃圾收集器。
			工作过程: 并发标记 + 并发预备重分配 + 并发重分配 + 并发重映射
			未来将在服务器、大内存、低延迟应用的首选垃圾收集器
二: 中篇——字节码与类的加载

		java语言: 跨平台的语言(Write once ,Write Anywhere)	其优势逐步被python/PHP/Perl等语言的强大解释器抹平
		JVM虚拟机: java虚拟机	不和包括java语言在内的任何语言绑定, 它只与"Class"这种二进制文件格式所关联; Class文件结构是java虚拟机的基石、桥梁。
			想让一个java程序正确的运行在JVM中, Java源码就必须编译为符合JVM规范的字节码
			——前端编译器(javac): 将符合java语言规范的java程序编译为符合JVM规范的字节码文件
			4个步骤: 词法解析、语法解析、词义解析及生成字节码
		前端编译器与后端编译器: 
			前端编译器: java源代码——(通过javac编译器编译为字节码文件)——Class文件		特点: 逐行编译, 效率低
			后端编译器: java源代码——(通过JIT即时编译器的编译解释到运行时数据区中)		特点: 热点代码搜索, 效率高
		Eclipse与IDEA采用的编译器: 前者采用的是Eclipse Compiler for Java(ECJ), 后者采用的是java默认的javac编译器
		Q1: Integer i = 128;Integer j = 128;System.out.println(i== j);
		Q2: String s1 = new String("Hello") + new String("World"); String s2 = new String("HelloWorld");System.out.println(i== j);
		查看字节码文件的几种方式: 1.IDEA中默认的javap的方式	 2.安装jclassLib插件查看
		字节码的基本内容: 操作码 (操作数)
	
