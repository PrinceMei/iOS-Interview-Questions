# 内存管理
## 1.什么情况使用 weak 关键字，相比 assign 有什么不同？
- 什么情况使用 weak 关键字？

    在 ARC 中,在有可能出现循环引用的时候,往往要通过让其中一端使用 weak 来解决,比如: delegate 代理属性

    自身已经对它进行一次强引用,没有必要再强引用一次,此时也会使用 weak,自定义 IBOutlet 控件属性一般也使用 weak；当然，也可以使用strong。在下文也有论述：《IBOutlet连出来的视图属性为什么可以被设置成weak?》

- 不同点：

    weak 此特质表明该属性定义了一种“非拥有关系” (nonowning relationship)。为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。此特质同assign类似， 然而在属性所指的对象遭到摧毁时，属性值也会清空(nil out)。 而 assign 的“设置方法”只会执行针对“纯量类型” (scalar type，例如 CGFloat 或 NSlnteger 等)的简单赋值操作。

    assign 可以用非 OC 对象,而 weak 必须用于 OC 对象

## 2.如何让自己的类用 copy 修饰符？如何重写带 copy 关键字的 setter？

- 若想令自己所写的对象具有拷贝功能，则需实现 NSCopying 协议。如果自定义的对象分为可变版本与不可变版本，那么就要同时实现 NSCopying 与 NSMutableCopying 协议。

    具体步骤：

    需声明该类遵从 NSCopying 协议
    
    实现 NSCopying 协议。该协议只有一个方法:
    
    ``` c 
    - (id)copyWithZone:(NSZone *)zone;
    ```

    注意：一提到让自己的类用 copy 修饰符，我们总是想覆写copy方法，其实真正需要实现的却是 “copyWithZone” 方法。

- 重写带 copy 关键字的 setter，例如：
    
    ``` c 
    - (void)setName:(NSString *)name {
        //[_name release];
        _name = [name copy];
    }
    ```

## 3.@property 的本质是什么？ivar、getter、setter 是如何生成并添加到这个类中的

- @property 的本质是实例变量（ivar）+存取方法（access method ＝ getter + setter）,即 @property = ivar + getter + setter;
    
    “属性” (property)作为 Objective-C 的一项特性，主要的作用就在于封装对象中的数据。 Objective-C 对象通常会把其所需要的数据保存为各种实例变量。实例变量一般通过“存取方法”(access method)来访问。其中，“获取方法” (getter)用于读取变量值，而“设置方法” (setter)用于写入变量值。

- ivar、getter、setter 是自动合成这个类中的
    
    完成属性定义后，编译器会自动编写访问这些属性所需的方法，此过程叫做“自动合成”(autosynthesis)。需要强调的是，这个过程由编译 器在编译期执行，所以编辑器里看不到这些“合成方法”(synthesized method)的源代码。除了生成方法代码 getter、setter 之外，编译器还要自动向类中添加适当类型的实例变量，并且在属性名前面加下划线，以此作为实例变量的名字。在前例中，会生成两个实例变量，其名称分别为 _firstName 与 _lastName。也可以在类的实现代码里通过 @synthesize 语法来指定实例变量的名字.

## 4.@protocol 和 category 中如何使用 @property

- 在 protocol 中使用 property 只会生成 setter 和 getter 方法声明,我们使用属性的目的,是希望遵守我协议的对象能实现该属性

- category 使用 @property 也是只会生成 setter 和 getter 方法的声明,如果我们真的需要给 category 增加属性的实现,需要借助于运行时的两个函数：objc_setAssociatedObject和objc_getAssociatedObject

## 5.简要说一下 @autoreleasePool 的数据结构？？

简单说是双向链表，每张链表头尾相接，有 parent、child指针

每创建一个池子，会在首部创建一个 哨兵 对象,作为标记

最外层池子的顶端会有一个next指针。当链表容量满了，就会在链表的顶端，并指向下一张表。

## 6.BAD_ACCESS在什么情况下出现？

访问了悬垂指针，比如对一个已经释放的对象执行了release、访问已经释放对象的成员变量或者发消息。 死循环

## 7.使用CADisplayLink、NSTimer有什么注意点？

CADisplayLink、NSTimer会造成循环引用，可以使用YYWeakProxy或者为CADisplayLink、NSTimer添加block方法解决循环引用

## 8.iOS内存分区情况

- 栈区（Stack）

    由编译器自动分配释放，存放函数的参数，局部变量的值等
    
    栈是向低地址扩展的数据结构，是一块连续的内存区域

- 堆区（Heap）

    由程序员分配释放
    
    是向高地址扩展的数据结构，是不连续的内存区域

- 全局区

    全局变量和静态变量的存储是放在一块的，初始化的全局变量和静态变量在一块区域，未初始化的全局变量和未初始化的静态变量在相邻的另一块区域

    程序结束后由系统释放

- 常量区

    常量字符串就是放在这里的
    
    程序结束后由系统释放

- 代码区

    存放函数体的二进制代码

- 注：

    - 在 iOS 中，堆区的内存是应用程序共享的，堆中的内存分配是系统负责的
    
    - 系统使用一个链表来维护所有已经分配的内存空间（系统仅仅记录，并不管理具体的内容）
    
    - 变量使用结束后，需要释放内存，OC 中是判断引用计数是否为 0，如果是就说明没有任何变量使用该空间，那么系统将其回收
    
    - 当一个 app 启动后，代码区、常量区、全局区大小就已经固定，因此指向这些区的指针不会产生崩溃性的错误。而堆区和栈区是时时刻刻变化的（堆的创建销毁，栈的弹入弹出），所以当使用一个指针指向这个区里面的内存时，一定要注意内存是否已经被释放，否则会产生程序崩溃（也即是野指针报错）

## 9.iOS内存管理方式

- Tagged Pointer（小对象）

    Tagged Pointer 专门用来存储小的对象，例如 NSNumber 和 NSDate
    
    Tagged Pointer 指针的值不再是地址了，而是真正的值。所以，实际上它不再是一个对象了，它只是一个披着对象皮的普通变量而已。所以，它的内存并不存储在堆中，也不需要 malloc 和 free
    
    在内存读取上有着 3 倍的效率，创建时比以前快 106 倍
    
    objc_msgSend 能识别 Tagged Pointer，比如 NSNumber 的 intValue 方法，直接从指针提取数据
    
    使用 Tagged Pointer 后，指针内存储的数据变成了 Tag + Data，也就是将数据直接存储在了指针中
    
- NONPOINTER_ISA （指针中存放与该对象内存相关的信息）
    苹果将 isa 设计成了联合体，在 isa 中存储了与该对象相关的一些内存的信息，原因也如上面所说，并不需要 64 个二进制位全部都用来存储指针。
    
    isa 的结构：
    ``` c 
    // x86_64 架构
    struct {
        uintptr_t nonpointer        : 1;  // 0:普通指针，1:优化过，使用位域存储更多信息
        uintptr_t has_assoc         : 1;  // 对象是否含有或曾经含有关联引用
        uintptr_t has_cxx_dtor      : 1;  // 表示是否有C++析构函数或OC的dealloc
        uintptr_t shiftcls          : 44; // 存放着 Class、Meta-Class 对象的内存地址信息
        uintptr_t magic             : 6;  // 用于在调试时分辨对象是否未完成初始化
        uintptr_t weakly_referenced : 1;  // 是否被弱引用指向
        uintptr_t deallocating      : 1;  // 对象是否正在释放
        uintptr_t has_sidetable_rc  : 1;  // 是否需要使用 sidetable 来存储引用计数
        uintptr_t extra_rc          : 8;  // 引用计数能够用 8 个二进制位存储时，直接存储在这里
    };
    
    // arm64 架构
    struct {
        uintptr_t nonpointer        : 1;  // 0:普通指针，1:优化过，使用位域存储更多信息
        uintptr_t has_assoc         : 1;  // 对象是否含有或曾经含有关联引用
        uintptr_t has_cxx_dtor      : 1;  // 表示是否有C++析构函数或OC的dealloc
        uintptr_t shiftcls          : 33; // 存放着 Class、Meta-Class 对象的内存地址信息
        uintptr_t magic             : 6;  // 用于在调试时分辨对象是否未完成初始化
        uintptr_t weakly_referenced : 1;  // 是否被弱引用指向
        uintptr_t deallocating      : 1;  // 对象是否正在释放
        uintptr_t has_sidetable_rc  : 1;  // 是否需要使用 sidetable 来存储引用计数
        uintptr_t extra_rc          : 19;  // 引用计数能够用 19 个二进制位存储时，直接存储在这里
    };
    ```
    这里的 has_sidetable_rc 和 extra_rc，has_sidetable_rc 表明该指针是否引用了 sidetable 散列表，之所以有这个选项，是因为少量的引用计数是不会直接存放在 SideTables 表中的，对象的引用计数会先存放在 extra_rc 中，当其被存满时，才会存入相应的 SideTables 散列表中，SideTables 中有很多张 SideTable，每个 SideTable 也都是一个散列表，而引用计数表就包含在 SideTable 之中。
    
- 散列表（引用计数表、弱引用表）
    
    引用计数要么存放在 isa 的 extra_rc 中，要么存放在引用计数表中，而引用计数表包含在一个叫 SideTable 的结构中，它是一个散列表，也就是哈希表。而 SideTable 又包含在一个全局的 StripeMap 的哈希映射表中，这个表的名字叫 SideTables。
    
    当一个对象访问 SideTables 时：
    
    - 首先会取得对象的地址，将地址进行哈希运算，与 SideTables 中 SideTable 的个数取余，最后得到的结果就是该对象所要访问的 SideTable
        
    - 在取得的 SideTable 中的 RefcountMap 表中再进行一次哈希查找，找到该对象在引用计数表中对应的位置
        
    - 如果该位置存在对应的引用计数，则对其进行操作，如果没有对应的引用计数，则创建一个对应的 size_t 对象，其实就是一个 uint 类型的无符号整型
        
    弱引用表也是一张哈希表的结构，其内部包含了每个对象对应的弱引用表 weak_entry_t，而 weak_entry_t 是一个结构体数组，其中包含的则是每一个对象弱引用的对象所对应的弱引用指针。


