---
layout: post
title: "Objective-C中Block的使用"
date: 2014-04-03 17:20
comments: true
categories: iOS
tags: [block]
---

> 看了几篇关于block的不错的博文，记录下来，方便查阅。

#### Block简介

> Block作为C语言的扩展，并不是高新技术，和其他语言的闭包或lambda表达式是一回事。需要注意的是由于Objective-C在iOS中不支持GC机制，使用Block必须自己管理内存，而内存管理正是使用Block坑最多的地方，错误的内存管理 要么导致retain cycle内存泄漏,要么内存被提前释放导致crash。 Block的使用很像函数指针，不过与函数最大的不同是：Block可以访问函数以外、词法作用域以内的外部变量的值，函数不可以；另外一个不同：Block是一个“仿”对象，可以使用自动释放池管理管理内存。Object-C中的对象都是继承自NSObject对象，NSObject对象的实现中有一个isa指针，是一个objc_class指针，指向Class的property；block对象中也有一个isa指针，指向的是_NSConcrete*Block对象，用于指明这个block的类型。  

---

#### Block语法

声明block变量:

As a local variable:

	returnType (^blockName)(parameterTypes) = ^returnType(parameters) {...};

As a property:

	@property (nonatomic, copy) returnType (^blockName)(parameterTypes);

As a method parameter:

	- (void)someMethodThatTakesABlock:(returnType (^)(parameterTypes))blockName;

As an argument to a method call:

	[someObject someMethodThatTakesABlock: ^returnType (parameters) {...}];

As a typedef:

	typedef returnType (^TypeName)(parameterTypes);
	TypeName blockName = ^returnType(parameters) {...};

---

#### Block数据格式

如下图:

![](http://www.galloway.me.uk/media/images/2013-05-26-a-look-inside-blocks-episode-3-block-copy/block_layout.png)

    struct Block_descriptor {
        unsigned long int reserved;
        unsigned long int size;
        void (*copy)(void *dst, void *src);
        void (*dispose)(void *);
    };
    
    struct Block_layout {
        void *isa;
        int flags;
        int reserved;
        void (*invoke)(void *, ...);
        struct Block_descriptor *descriptor;
        /* Imported variables. */
    };

一个block实例实际上由6部分构成：

- isa指针，所有对象都有该指针，用于实现对象相关的功能。
- flags，用于按bit位表示一些block的附加信息，本文后面介绍block copy的实现代码可以看到对该
- 变量的使用。
- reserved，保留变量。
- invoke，函数指针，指向具体的block实现的函数调用地址。
- descriptor， 表示该block的附加描述信息，主要是size大小，以及copy和dispose函数的指针。
- variables，capture过来的变量，block能够访问它外部的局部变量，就是因为将这些变量（或变量的地址）复制到了结构体中。

---

#### Block类型

1._NSConcreteGlobalBlock 保存在代码段(类似于函数)，不会访问任何外部变量。  

	{
		//create a NSGlobalBlock
		float (^sum)(float, float) = ^(float a, float b){
			return a + b;
		};
		NSLog(@"block is %@", sum); //block is <__NSGlobalBlock__: 0x>
	}

2._NSConcreteStackBlock 保存在栈中的block，当函数返回时会被销毁。  

	{
    	NSArray *testArr = @[@"1", @"2"];
 
    	NSLog(@"block is %@", ^{
 
        	NSLog(@"test Arr :%@", testArr);
    	});
    	// block is <__NSStackBlock__: 0x>
    	// 打印可看出block是一个 NSStackBlock, 即在栈上, 当函数返回时block将无效
	}

3._NSConcreteMallocBlock 保存在堆中的block，当引用计数为0时会被销毁，对_NSConcreteStackBlock进行copy操作。

	{
    	NSArray *testArr = @[@"1", @"2"];	
	
    	void (^TestBlock)(void) = ^{
 
        	NSLog(@"testArr :%@", testArr);
    	}; 
    	NSLog(@"block is %@", TestBlock);
    	// block is <__NSMallocBlock__: 0x>
    	// 上面这句在非arc中打印是 NSStackBlock, 但是在arc中就是NSMallocBlock
    	// 区别就是ARC下strong指针过了一趟手，block被copy到堆上了
	}

4._NSConcreteWeakBlockVariable  
5._NSConcreteAutoBlock  
6._NSConcreteFinalizingBlock  
后三种用于GC，不做讨论。

---

#### Block的生命周期

Block的isa指针指向的是_NSConcreteStackBlock，这个prototype重新定义了一下咱们熟悉(NSObject所定义的)的retain/copy/release等函数。

**非ARC环境下:**

测试代码:

    NSLog(@"_NSConcreteStackBlock %@", [_NSConcreteStackBlock class]); // _NSConcreteStackBlock __NSStackBlock__
    
    __block int val = 10;
    blk stackBlock = ^{NSLog(@"val = %d", ++val);};
    NSLog(@"stackBlock: %@", stackBlock); // stackBlock: <__NSStackBlock__: 0xbfffdb28>
    
    tempBlock = [stackBlock copy];
    NSLog(@"tempBlock: %@", tempBlock); // tempBlock: <__NSMallocBlock__: 0x756bf20>
    
    // 结果: 经过copy之后，对象的类型从__NSStackBlock__变为了__NSMallocBlock__

得出的结论:  

{% img /images/mm.jpeg %} 

+ NSGlobalBlock类型(理解为静态block)：retain、copy、release都不会影响block的生命周期。

+ 当一个block对象在堆上了，生命周期和普通的NSObject对象的管理一样，copy和release一一对应，不然会存在系统级别的内存泄露，在Instruments里跟不到问题触发点。

+ NSMallocBlock支持retain、release，虽然retainCount始终是1，但内存管理器中仍然会增加、减少计数。copy之后不会生成新的对象，只是增加了一次引用，类似NSObject的retain操作，多个block指针指向的是heap中的同一块内存，[图解][14]；

+ NSStackBlock：NSStackBlock在函数返回后，Block内存将被回收，即使retain也没用，容易犯的错误是[[mutableAarry addObject:stackBlock]，在函数出栈后，从数组中取到的stackBlock已经被回收，变成了野指针。正确的做法是先将stackBlock copy到堆上，然后加入数组：[mutableAarry addObject:[[stackBlock copy] autorelease]]。如果要长期持有block对象请把她移到堆上。

+ 在向外传递block的时候参考类似NSObject的规则，一定要传递一个autorelease的\_\_NSMallocBlock\_\_对象，类似于return [[stackBlock copy] autorelease];

**ARC环境下：**

测试代码:

	__block int val = 10;
    __strong blk strongPointerBlock = ^{NSLog(@"val = %d", ++val);};
    NSLog(@"strongPointerBlock: %@", strongPointerBlock); 
    // strongPointerBlock: <__NSMallocBlock__: 0x7625120> strong指针指向的block已经放到堆上
        
    __weak blk weakPointerBlock = ^{NSLog(@"val = %d", ++val);};
    NSLog(@"weakPointerBlock: %@", weakPointerBlock); 
    // weakPointerBlock: <__NSStackBlock__: 0xbfffdb30> weak指针指向的block还在栈上
        
    NSLog(@"mallocBlock: %@", [weakPointerBlock copy]); 
    // mallocBlock: <__NSMallocBlock__: 0x714ce60> copy会将block从栈移动到堆上
        
    NSLog(@"test %@", ^{NSLog(@"val = %d", ++val);}); 
    // test <__NSStackBlock__: 0xbfffdb18> 匿名的block在栈上

得出的结论: 

- 只要block在strong指针底下过一道都会放到堆上，变成NSMallocBlock类型。
- 匿名block和weak指针指向的block在栈上。
- block做为函数返回参数时，block也会自动被移到堆上，所以无需非ARC的规则。
- ARC下的注意事项，block指针过一下strong指针，他好，我也好。

---

#### Block对外部变量的管理

#####1、基本类型变量

**局部自动变量**：Block**定义**时copy变量的值，在Block中作为常量使用，所以即使变量的值在Block外改变，也不影响在Block中的值(根本不是一个变量)。

	{
    	int base = 100;
    	long (^sum)(int, int) = ^ long (int a, int b) {
 
        	return base + a + b;
    	};
 
    	base = 0;
    	printf("%ld\n",sum(1,2));
    	// 这里输出是103，而不是3, 因为块内base为拷贝的常量 100
	}

**STATIC修饰符的全局变量**:全局变量或静态变量在内存中的地址是固定的，Block在读取该变量值的时候是直接从其所在内存读出，获取到的是最新值，而不是在定义时copy的常量.

	{
    	static int base = 100;
    	long (^sum)(int, int) = ^ long (int a, int b) {
        	base++;
        	return base + a + b;
    	};
 
    	base = 0;
    	printf("%ld\n",sum(1,2));
    	// 这里输出是4，而不是103, 因为base被设置为了0
    	printf("%d\n", base);
    	// 这里输出1， 因为sum中将base++了
	}

**__BLOCK修饰的变量**:被__block修饰的变量称作Block变量，基本类型的Block变量等效于全局变量、或静态变量(同一个变量)，当存在NSMallocBlock类型的block和NSStackBlock类型的block同时引用\_\_block变量，\_\_block变量将会被移到heap中。

#####2、对象

block对于objc对象的内存管理较为复杂，这里要分static变量、global变量、local变量、block变量分析、还要分非arc和arc分析。

###### 2.1、非ARC中的对象

	@interface MyClass : NSObject {
    	NSObject* _instanceObj;
	}
	@end
 
	@implementation MyClass
 
	NSObject* __globalObj = nil;
 
	- (id) init {
    	if (self = [super init]) {
        	_instanceObj = [[NSObject alloc] init];
    	}
    	return self;
	}
 
	- (void) test {
    	static NSObject* __staticObj = nil;
    	__globalObj = [[NSObject alloc] init];
    	__staticObj = [[NSObject alloc] init];
 
    	NSObject* localObj = [[NSObject alloc] init];
    	__block NSObject* blockObj = [[NSObject alloc] init];
 
    	typedef void (^MyBlock)(void) ;
    	MyBlock aBlock = ^{
        	NSLog(@"%@", __globalObj);
        	NSLog(@"%@", __staticObj);
        	NSLog(@"%@", _instanceObj);
        	NSLog(@"%@", localObj);
        	NSLog(@"%@", blockObj);
    	};
    	aBlock = [[aBlock copy] autorelease];
    	aBlock();
 
    	NSLog(@"%d", [__globalObj retainCount]);
    	NSLog(@"%d", [__staticObj retainCount]);
		NSLog(@"%d", [_instanceObj retainCount]);
    	NSLog(@"%d", [localObj retainCount]);
    	NSLog(@"%d", [blockObj retainCount]);
	}
	@end
 
	int main(int argc, char *argv[]) {
    	@autoreleasepool {
        	MyClass* obj = [[[MyClass alloc] init] autorelease];
        	[obj test];
        	return 0;
    	}
	}

	// 执行结果:1 1 1 2 1

得出的结论：  

+ __globalObj和__staticObj在内存中的位置是确定的，所以Block copy时不会retain对象。
+ _instanceObj在Block copy时也没有直接retain _instanceObj对象本身，但会retain self。所以在Block中可以直接读写_instanceObj变量。
+ localObj在Block copy时，系统自动retain对象，增加其引用计数。
+ blockObj在Block copy时也不会retain。

###### 2.2、ARC中的对象

由于arc中没有retain，retainCount的概念。只有强引用和弱引用的概念。当一个变量没有__strong的指针指向它时，就会被系统释放。因此我们可以通过下面的代码来测试。

    NSString *__globalString = nil;
     
    - (void)testGlobalObj
    {
        __globalString = @"1";
        void (^TestBlock)(void) = ^{
     
            NSLog(@"string is :%@", __globalString); 
        };
     
        __globalString = nil;
     
        TestBlock(); // 结果：string is null
    }
     
    - (void)testStaticObj
    {
        static NSString *__staticString = nil;
        __staticString = @"1";
     
        printf("static address: %p\n", &__staticString);    //static address: 0x6a8c
     
        void (^TestBlock)(void) = ^{
     
            printf("static address: %p\n", &__staticString); //static address: 0x6a8c
     
            NSLog(@"string is : %@", __staticString); //string is null
        };
     
        __staticString = nil;
     
        TestBlock();
    }
     
    - (void)testLocalObj
    {
        NSString *__localString = nil;
        __localString = @"1";
     
        printf("local address: %p\n", &__localString); //local address: 0xbfffd9c0
     
        void (^TestBlock)(void) = ^{
     
            printf("local address: %p\n", &__localString); //local address: 0x71723e4
     
            NSLog(@"string is : %@", __localString); //string is : 1
        };
     
        __localString = nil;
     
        TestBlock();
    }
     
    - (void)testBlockObj
    {
        __block NSString *_blockString = @"1";
     
        void (^TestBlock)(void) = ^{
     
            NSLog(@"string is : %@", _blockString); // string is null
        };
     
        _blockString = nil;
     
        TestBlock();
    }
     
    - (void)testWeakObj
    {
        NSString *__localString = @"1";
     
        __weak NSString *weakString = __localString;
     
        printf("weak address: %p\n", &weakString);  //weak address: 0xbfffd9c4
        printf("weak str address: %p\n", weakString); //weak str address: 0x684c
     
        void (^TestBlock)(void) = ^{
     
            printf("weak address: %p\n", &weakString); //weak address: 0x7144324
            printf("weak str address: %p\n", weakString); //weak str address: 0x684c
     
            NSLog(@"string is : %@", weakString); //string is :1
        };
     
        __localString = nil;
     
        TestBlock();
    }

得出的结论：  

+ 只有在使用local变量时，block会复制指针，且强引用指针指向的对象一次。其它如全局变量、static变量、block变量等，block不会拷贝指针,只会强引用指针指向的对象一次。
+ 即使标记了为__weak或__unsafe_unretained的local变量。block仍会强引用指针对象一次。（这个不太明白，因为这种写法可在后面避免循环引用的问题）
+ 用NSString *__localString = @”1″;分配的1是在静态存储区，所以在Block中打印printf(“weak str address: %p\n”, weakString);也会输出，测试对象的时候最好不要用NSString，应为它很特别，最好用UILabel，UIView等，__block是复制了内容的；而__weak在block是空的

---

#### Block引用不当导致的回环retain cycle

block在拷贝到堆上的时候，会retain其引用的外部变量，那么如果block中如果引用了他的宿主对象，那很有可能引起循环引用，如：

	self.myblock = ^{
 
        [self doSomething];
    };


ARC环境测试代码如下：

    - (void)dealloc
    {
     
        NSLog(@"no cycle retain");
    }
     
    - (id)init
    {
        self = [super init];
        if (self) {
     
    #if TestCycleRetainCase1
     
            //会循环引用
            self.myblock = ^{
     
                [self doSomething];
            };
    #elif TestCycleRetainCase2
     
            //会循环引用
            __block TestCycleRetain *weakSelf = self;
            self.myblock = ^{
     
                [weakSelf doSomething];
            };
     
    #elif TestCycleRetainCase3
     
            //不会循环引用
            __weak TestCycleRetain *weakSelf = self;
            self.myblock = ^{
     
                [weakSelf doSomething];
            };
     
    #elif TestCycleRetainCase4
     
            //不会循环引用
            __unsafe_unretained TestCycleRetain *weakSelf = self;
            self.myblock = ^{
     
                [weakSelf doSomething];
            };
     
    #endif
     
            NSLog(@"myblock is %@", self.myblock);
     
        }
        return self;
    }
     
    - (void)doSomething
    {
        NSLog(@"do Something");
    }
     
    int main(int argc, char *argv[]) {
        @autoreleasepool {
            TestCycleRetain* obj = [[TestCycleRetain alloc] init];
            obj = nil;
            return 0;
        }
    }


在加了__weak和__unsafe_unretained的变量引入后，TestCycleRetain方法可以正常执行dealloc方法，而不转换和用__block转换的变量都会引起循环引用。
因此防止循环引用的方法如下：
__unsafe_unretained TestCycleRetain *weakSelf = self;

---

#### 参考文章

[1、对Objective-C中Block的追探][1]  
[2、block使用小结、在arc中使用block、如何防止循环引用][2]  
[3、谈Objective-C Block的实现][3]  
[4、A look inside blocks: Episode 1][4]  [中文][9]  
[5、A look inside blocks: Episode 2][5]  [中文][10]  
[6、A look inside blocks: Episode 3 (Block_copy)][6]  [中文][11]   
[7、iOS Block -浅析][7]  
[8、正确使用Block避免Cycle Retain和Crash][8]  
[9、初识block][12]  
[10、objc中的block][13]

[1]:http://www.cnblogs.com/biosli/archive/2013/05/29/iOS_Objective-C_Block.html
[2]:http://www.cnbluebox.com/?p=255
[3]:http://blog.devtang.com/blog/2013/07/28/a-look-inside-blocks/
[4]:http://www.galloway.me.uk/2012/10/a-look-inside-blocks-episode-1/
[5]:http://www.galloway.me.uk/2012/10/a-look-inside-blocks-episode-2/
[6]:http://www.galloway.me.uk/2013/05/a-look-inside-blocks-episode-3-block-copy/
[7]:http://blog.sina.com.cn/s/blog_7b9d64af0101c4sh.html
[8]:http://tanqisen.github.io/blog/2013/04/19/gcd-block-cycle-retain/
[9]:http://beyondvincent.com/blog/2013/07/09/99/
[10]:http://beyondvincent.com/blog/2013/07/10/100/
[11]:http://beyondvincent.com/blog/2013/07/11/101/
[12]:http://beyondvincent.com/blog/2013/07/08/98/
[13]:http://blog.ibireme.com/2013/11/27/objc-block/
[14]:http://www.cnblogs.com/studentdeng/archive/2012/02/03/2336863.html
