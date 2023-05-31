在共享内存的并发模型里，线程之间共享程序的公共状态，通过写-读内存中的公共状态进行隐式通信。在消息传递的并发模型里，线程之间没有公共状态，线程之间必须通过发送消息来显式进行通信

同步是指程序中用于控制不同线程间操作发生相对顺序的机制。在共享内存并发模型里，同步是显式进行的。程序员必须显式指定某个方法或某段代码需要在线程之间互斥执行。在消息传递的并发模型里，由于消息的发送必须在消息的接收之前，因此同步是隐式进行的

## 1. Java内存模型的抽象结构
Java的并发采用的是共享内存模型，Java线程之间的通信总是隐式进行，整个通信过程对程序员完全透明

在Java中，所有实例域、静态域和数组元素都存储在堆内存中，堆内存在线程之间共享。局部变量，方法定义参数和异常处理器参数不会在线程之间共享，它们不会有内存可见性问题，也不受内存模型的影响

Java线程之间的通信由Java内存模型JMM控制，JMM决定一个线程对共享变量的写入何时对另一个线程可见。从抽象的角度来看，JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存中，每个线程都有一个私有的本地内存，本地内存中存储了该线程以读/写共享变量的副本。本地内存是JMM的一个抽象概念，并不真实存在。它涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器优化

<img src="D:\Project\IT-notes\Java\并发\img\JMM抽象结构示意.png" style="width:750px;height:500px;" />

## 2. 从源代码到指令序列的重排序
在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排序：
- 编译器优化的重排序：不改变单线程程序语义，重新安排语句执行顺序
- 指令集并行的重排序：将多条指令重叠执行，对于不存在数据依赖性的可以改变执行顺序
- 内存系统的重排序：使用缓存以及读写缓冲区，使得加载和存储操作看上去乱序执行

<img src="D:\Project\IT-notes\Java\并发\img\从源码到最终执行的指令序列.png" style="width:700px;height:100px;" />

对于编译器，JMM的编译器重排序规则会禁止特定类型的编译器重排序。对于处理器重排序，JMM的处理器重排序规则会要求Java编译器在生成指令序列时，插入特定类型的内存屏障指令，通过内存屏障指令来禁止特定类型的处理器重排序

## 3. 内存屏障
内存屏障分为两种：**Load Barrier**和**Store Barrier**，即读屏障和写屏障
内存屏障有两个作用：1. 阻止屏障两侧的指令重排序；2. 强制把写缓冲区/高速缓存中的脏数据等写回主内存，让缓存中相应的数据失效

- 对于`Load Barrier`来说，在指令前插入`Load Barrier`，可以让高速缓存中的数据失效，强制重新从主内存加载数据；
- 对于`Store Barrier`来说，在指令后插入`Store Barrier`，能让写入缓存中的最新数据更新写入主内存，让其他线程可见

Java的内存屏障通常所谓的四种即**LoadLoad、StoreStore、LoadStore、StoreLoad**
- `LoadLoad屏障`：对于这样的语句`Load1;LoadLoad;Load2`，在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕
- `StoreStore屏障`：对于这样的语句`Store1;StoreStore;Store2`，在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见
- `LoadStore屏障`：对于这样的语句`Load1;LoadStore;Store2`，在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕
- `StoreLoad屏障`：对于这样的语句`Store1;StoreLoad;Load2`，在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见

## 4. happens-before

## 5. 数据依赖性
如果两个操作访问同一个变量，且这两个操作中有一个为写操作，此时这两个操作之间就存在数据依赖性
编译器和处理器可能会对操作做重排序。编译器和处理器在重排序时，会遵守数据依赖性，编译器和处理器不会改变存在数据依赖关系的两个操作的执行顺序
这里所说的数据依赖性仅针对单个处理器中执行的指令序列和单个线程中执行的操作，不同处理器之间和不同线程之间的数据依赖性不被编译器和处理器考虑

## 6. as-if-serial

## 7. 顺序一致性

## 8. volatile内存语义
对volatile变量的单个读写，看成是使用同一个锁对这些单个读写操作做了同步
锁的`happens-before`规则保证释放锁和获取锁的两个线程之间的内存可见性，这意味着对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入

volatile变量自身具有下列特性：
- 可见性：对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入
- 原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性

volatile写的内存语义：当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存

volatile读的内存语义：当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效。线程接下来将从主内存中读取共享变量

<img src="D:\Project\IT-notes\Java\并发\img\volatile重排序规则表.png" style="width:700px;height:180px;" />

从表中可知：
- 当第二个操作是volatile写时，不管第一个操作是什么，都不能重排序；
- 当第一个操作是volatile读时，不管第二个操作是什么，都不能重排序；
- 当第一个操作是volatile写，第二个操作是volatile读时，不能重排序

实现volatile内存语义时使用内存屏障：
1. 在每个volatile写操作的前面插入一个`StoreStore`屏障
2. 在每个volatile写操作的后面插入一个`StoreLoad`屏障
3. 在每个volatile读操作的后面插入一个`LoadLoad`屏障
4. 在每个volatile读操作的后面插入一个`LoadStore`屏障

从整体执行效率的角度考虑，JMM最终选择了在每个volatile写的后面插入一个`StoreLoad`屏障。因为volatile写-读内存语义的常见使用模式是：一个写线程写volatile变量，多个读线程读同一个volatile变量。当读线程的数量大大超过写线程时，选择在volatile写之后插入`StoreLoad`屏障将带来可观的执行效率的提升

由于volatile仅仅保证对单个volatile变量的读/写具有原子性，而锁的互斥执行的特性可以确保对整个临界区代码的执行具有原子性。在功能上，锁比volatile更强大；在可伸缩性和执行性能上，volatile更有优势

## 9. 锁的内存语义
当线程释放锁时，JMM会把该线程对应的本地内存中的共享变量刷新到主内存中；当线程获取锁时，JMM会把该线程对应的本地内存置为无效，从而使得被监视器保护的临界区代码必须从主内存中读取共享变量

在使用AQS框架的各子类锁源码中，获取锁时会读取内部volatile变量或者使用CAS操作更新volatile变量，释放锁时会写入内部volatile变量--对于ReentrantLock，公平锁与非公平锁锁释放时都会写入volatile变量state，公平锁获取则会读volatile变量，非公平锁获取会使用CAS更新volatile变量（CAS操作具有volatile读和volatile写的内存语义）

锁获取内存语义=读volatile变量内存语义；锁释放内存语义=写volatile变量内存语义

<img src="D:\Project\IT-notes\Java\并发\img\concurrent包实现.png" style="width:700px;height:500px;" />

## 10. final域的内存语义
对于final域，编译器和处理器要遵守如下重排序规则：
- 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序
- 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序

### 写final域的重排序规则
- JMM禁止编译器把final域的写重排序到构造函数之外
- 编译器会在final域的写之后，构造函数return之前，插入一个StoreStore屏障。这个屏障禁止处理器把final域的写重排序到构造函数之外

写final域的重排序规则可以确保：在对象引用为任意线程可见之前，对象的final域已经被正确初始化过了，而普通域不具有这个保障

### 读final域的重排序规则
读final域的重排序规则是，在一个线程中，初次读对象引用与初次读该对象包含的final域，JMM禁止处理器重排序这两个操作。编译器会在读final域操作的前面插入一个LoadLoad屏障

初次读对象引用与初次读该对象包含的final域，这两个操作之间存在间接依赖关系。由于编译器遵守间接依赖关系，因此编译器不会重排序这两个操作。大多数处理器也会遵守间接依赖，也不会重排序这两个操作

## 11. 双重检查锁定与延迟初始化
### 基于volatile的解决方案
```java
public class SafeDoubleCheckedLocking {                       // 1 
	private static volatile Instance instance;            // 2 
	public static Instance getInstance() {                // 3 
		if (instance == null) {                           // 4:第一次检查 
			synchronized (SafeDoubleCheckedLocking.class) {   // 5:加锁 
				if (instance == null)                     // 6:第二次检查 
					instance = new Instance();            // 7:问题的根源出在这里 
				}                                         // 8 
			}                                             // 9 
		return instance;                                  // 10 
	}                                                     // 11 
}
```


### 基于类初始化的解决方案
```java
public class InstanceFactory { 
	private static class InstanceHolder { 
		public static Instance instance = new Instance(); 
	} 
	public static Instance getInstance() { 
		return InstanceHolder.instance; // 这里将导致InstanceHolder类被初始化 
	} 
}
```

<img src="D:\Project\IT-notes\Java\并发\img\两个线程并发执行双重检查锁.png" style="width:700px;height:450px;" />

初始化一个类，包括执行这个类的静态初始化和初始化在这个类中声明的静态字段。在首次发生下列任意一种情况时，一个类或接口类型T将被立即初始化
- T是一个类，而且一个T类型的实例被创建
- T是一个类，且T中声明的一个静态方法被调用
- T中声明的一个静态字段被赋值
- T中声明的一个静态字段被使用，而且这个字段不是一个常量字段
- T是一个顶级类，而且一个断言语句嵌套在T内部被执行

由于Java语言是多线程的，多个线程可能在同一时间尝试去初始化同一个类或接口，对于每一个类或接口C，都有一个唯一的初始化锁LC与之对应。从C到LC的映射，由JVM的具体实现去自由实现。JVM在类初始化期间会获取这个初始化锁，并且每个线程至少获取一次锁来确保这个类已经被初始化过了

<img src="D:\Project\IT-notes\Java\并发\img\类初始化-第1阶段.png" style="width:700px;height:500px;" />

```
1. A1 尝试获取 Class 对象的初始化锁。这里假设线程 A 获取到了初始化锁
2. B1 尝试获取 Class 对象的初始化锁，由于线程 A 获取到了锁，线程 B 将一直等待获取初始化锁
3. 线程 A 看到线程还未被初始化（因为读取到 state == noInitialization），线程设置 state = initializing，线程 A 释放初始化锁
```

<img src="D:\Project\IT-notes\Java\并发\img\类初始化-第2阶段.png" style="width:700px;height:500px;" />

```
1. 线程 A 执行类的静态初始化和初始化类中声明的静态字段
2. 线程 B 获取到初始化锁，读取到 state == initializing，释放初始化锁，在初始化锁的 condition 中等待
```

<img src="D:\Project\IT-notes\Java\并发\img\类初始化-第3阶段.png" style="width:700px;height:500px;" />

```
1. 线程 A 获取初始化锁，设置 state = initialized，唤醒在 condition 中等待的所有线程，释放初始化锁
2. 线程 B 获取初始化锁，读取到 state == initialized，释放初始化锁，线程 B 的类初始化处理过程完成
```

<img src="D:\Project\IT-notes\Java\并发\img\类初始化-第4阶段.png" style="width:700px;height:500px;" />

<img src="D:\Project\IT-notes\Java\并发\img\类初始化-第5阶段.png" style="width:700px;height:400px;" />

