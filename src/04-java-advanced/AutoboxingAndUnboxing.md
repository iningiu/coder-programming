![图片](xxxx.png)

> 每章一点正能量：每当你想要放弃的时候，就想想是为了什么才一路坚持到现在。


## 前言

最近在回顾复习Java基础中的一些知识点，发现了一些以前见过但是没有留意却特别有意思的知识特性，比如这次想分享的Java中一个常见的特性:`自动装箱与拆箱`。这个知识点和特性其实在我们开发过程中经常会遇到。同时我们也会去使用一些基本数据类型或者是封装数据类型，但是对于他们之间的一些转换等特性可能不是特别清楚。也可能出现在我们的面试中。本章部分内容从源码中解读一些自动装箱与拆箱的原理，以及会出现的一些陷阱已经性能等。如有错误还请大家及时指出~ 

>本文已同步至 [GitHub](https://github.com/CoderMerlin/coder-programming)/[Gitee](https://gitee.com/573059382/coder-programming)/公众号，感兴趣的同学帮忙点波关注~

问题：

- 基本数据类型与封装数据类型有哪些区别？
- 什么是装箱？什么是拆箱？
- 装箱和拆箱都是如何实现的？
- 使用时需要注意哪些问题？ 

## 1.基础知识回顾

1. Java把内存划分成两种：一种是栈内存，另一种是堆内存。

2. int是基本类型，直接存数值；而 Integer是类，产生对象时用一个引用指向这个对象。

3. 封装类位于java.lang包中。

4. 封装类是引用传递而基本类型是值传递

5. 基本类型的变量和对象的引用变量都是在函数的栈内存中分配 ，而实际的对象是在存储堆内存中


### 1.1 基本数据类型和封装类型的区别

我们来看下他们之间有哪些区别：

[图片1]()

八种基本数据类型分别是：byte、char、boolean、int、short、float、double、long;
对应的封装类型分别是：Byte、Character、Boolean、Integer、Short、Float、Double、Long。

| 基本类型| 封装类型| 字节长度 | 默认值 |
|---|---|---|---|
|boolean|Boolean|1|false|
|byte|Byte|1|0|
|char|Character|2|u0000|
|short|Short|2|0|
|int|integer|4|0|
|long|Long|8|0l或0L|
|float|Float|4|0.0f或0.0F|
|double|Double|8|0.0|


---

## 2. "==" 和 "equal()" 方法

在巩固了上面的基础知识点之后，我们再来看下另外的一个知识点 `"=="和"equal()" ` 这两个判断符在比较基本数据类型和封装类型的时候会做的一些事情。

" == ":比较的是基本数据类型，比较的是它们的值

"equals()": 比较的是引用数据类型，根据不同的数据类型调用不同的equals方法。在特殊情况下可以重写equals方法。

a==b并不能判断a等于b，而是判断是否为同一个Object。如果我们要判断他们的值怎么做呢？用equal或者Objects.equals()(JDK1.7之后新加 的语法)

**Objects.equals有什么好处呢？**

如果用a.equals(b) 如果a是null 的话，还会抛出空指针异常。但是用Objects.equals就没有问题。因此我们在使用引用类型的时候需要注意，当我们在赋值的时候，两个变量都是引用同一个Object。



我们以 `int与Integer` 作为例子，看下"=="和"equal()"方法：

1）基本型和封装类型进行"=="运算符的比较，封装类型将会自动拆箱变为基本型后再进行比较，因此Integer(0)会自动拆箱为int类型再进行比较。

2）两个Integer类型进行"=="比较，如果其值在-128至127，那么返回true，否则返回false, 这跟Integer.valueOf()的缓冲对象有关，后面会说。

3）两个封装类型进行equals()比较，首先equals()会比较类型，如果类型相同，则继续比较值，如果值也相同，返回true。

4）基本型封装类型调用equals(),但是参数是基本类型，这时候，先会进行自动装箱，基本型转换为其封装类型，再进行3中的比较。



## 3.什么是装箱和拆箱

基本数据(Primitive)类型的自动装箱(autoboxing)、拆箱(unboxing)是自J2SE 5.0开始提供的功能。Java语言规范中说道：在许多情况下包装与解包装是由编译器自行完成的（在这种情况下包装称为装箱，解包装称为拆箱）。
通俗的理解：装箱：基本类型转换成封装类型， 拆箱：封装类型转换成基本类型 这么一个过程。在上面有介绍八种基本类型和对应的封装类型。下面以Integer与int之间的转换作为理解：

### 3.1 自动装箱（Autoboxing）

```

Integer a = 2; //Boxing

```

简单的理解：将2装在一个箱子里，这个箱子的类型是Integer 。箱子这里面装的数值就是2，我们就完成了一次装箱操作。并把a指向2这个箱子。

```

Integer b = new Integer(2);//Boxing

```

显示装箱。生成一个新的箱子 new Integer(); 并且这个箱子的值为2.而且让b指向这个箱子。


### 3.2 拆箱（Unboxing）

故名思议就是将对象重新转化为基本数据类型

```

int v = a.intValue(); //Unboxing

```

简单的理解：将里面int的值取出来。拆箱有个很典型的用法就是在进行运算的时候：因为对象时不能直接进行运算的，而是要转化为基本数据类型后才能进行加减乘除。

例如：

```

Integer c = 5;
System.out.print(c--);//进行计算时隐含的有自动拆箱

```


## 4. 装箱拆箱结合源码分析

通过第四点我们知道装箱拆箱的基本概念知识，下面我们同样以Integer 为例，进入源码里面看看里面的乾坤。

我们首先看下Integer的大小。

### 4.1 Integer 大小


可以看出，其定义了Integer的最大值为2^31-1，最小值为-2^31。Integer的基本数据类型为int。

![Integer大小]()

### 4.2 Integer中的valueOf()方法

再来看看Integer中的valueOf()方法。

![Integer中的valueOf()]()

可以看出valueOf()方法是个静态方法。当传进来的变量值在一个区间之内，直接用IntegerCache.cache[]数组里面的数返回，否则new一个新对象。

接着我们来看看IntegerCache类。其实也是会出现坑的一个地方。

### 4.3 其中存在的陷阱

接着来说下Integer这儿的一个坑，也是比较有意思的地方。


![IntegerCache]()

初始化Integer后，IntegerCache会缓存[-128,127]之间的数据，这个区间的上限可以配置，取决于java.lang.Integer.IntegerCache.high这个属性，这个属性在VM参数里为-XX:AutoBoxCacheMax=2000进行设置调整或者VM里设置-Djava.lang.Integer.IntegerCache.high=2000。所以Integer在初始化完成后会缓存[-128,max]之间的数据。cache属于常量，存放在java的方法区中。

同样，在Long，Byte，Short，我们也可以看到缓存，其缓存数据长度均是-128到127。这里不做展开。

**另外其他陷阱：**
如：

```

System.out.println(Integer.valueOf(null));

```

Integer对象的值可以为null，所以编译器检查时不会出现检查时异常，但是在转换成int的时候就会抛出空指针异常。

## 4. 例题分析

我们通过几个经典的问题，来看看大家到底理解了`装箱与拆箱`的知识点没。

- new Integer(5) == 5?
- new Integer(5) == new Integer(5) ? 
- Integer.valueOf(5) == Integer.valueOf(5)?
- Integer.valueOf(5).intValue() == 5？
- new Integer(5).equals(new Integer(5))?

### 4.1 问题一：new Integer(5) == 5?

**答案：true。** 等号的左边是一个Object右边是一个数值，Object和数值怎么会相等的呢？Java的编译器很聪明，它会自己去做装箱和拆箱的操作。这边它将new Integer(5)做的是Unboxing，它会里面的value取出来，这时候发现取出来的5等于右边，所以就为true。


### 4.2 问题二：new Integer(5) == new Integer(5) ? 

**答案：false。** new Integer(5) 就是新建一个箱子，这个箱子的值就是5。 == 是判断这两个箱子是不是同一个箱子，不是说里面的值是不是一样.所以是false。因为他们不是同一个箱子。



### 4.3 问题三：Integer.valueOf(5) == Integer.valueOf(5)?


**答案: true。** Integer.valueOf(5)它会返回一个箱子给我们，箱子里面的值是5。但是在返回这个箱子给我们的时候，可能会新建一个新的箱子给我们，也可能会使用现有的一个箱子给我们。所以Integer.valueOf(5) == Integer.valueOf(5)。什么情况下才会相等呢？只有当系统已经将2这个箱子建立好了，并且缓存起来的情况下。会把箱子的引用同时发给等号的左边与右边。这样的情况，他们才会互相相等。Integer.valueOf() 是系统给我们分配的一个箱子，我们发现，每次调我们的箱子时候，系统都给了同一个箱子。这个我们的 Integer.valueOf(5) == Integer.valueOf(5)

**但是：** 可能为false。我们在上面介绍过，在low和high之间，它会返回一个系统已经生产的cache，否则它会生产一个新的出来。看源码可以看到low = -128  high = 127。所以当它的值超过了区间后，它就会返回新的箱子，所以就会为false。

我们不用5改用200试一试。

```
Integer.valueOf(200) == Integer.valueOf(200)

```

**答案：false。** 说明系统对小的数字会使用系统分配的箱子，对于大的数字，系统会重新new一个箱子。面试的时候，可以回答，他们可能相等，也可能不相等。是有系统决定的。

### 4.4 问题四：Integer.valueOf(5).intValue() == 5？

**答案： true。**  intValue()做了一个拆箱的操作，将里面的值5取出来，值5等于5，所以是true。


### 4.5 问题五：new Integer(5).equals(new Integer(5))?

**答案：true。** 这里我们没有用==而是用equals，equals判断相等是判断里面的值是不是相等，而不是判断这个箱子是不是同一个，所以我们的答案是true。我们来看看equals的源码。判断里面的值是不是相等。

![equals]()


### 打印结果：

![打印结果]()

## 文末

本章节主要简单介绍了`自动装箱与拆箱`的相关知识，希望对大家有所帮助~
今后我会在每张文章开头增加 **每章一点正能量** ，文末增加5个编程相关的**英语单词** 学点英语。希望大家和我一样每天都能积极向上，一起学习一同进步！

### 学点英语

- AWT(Abstract Window Toolkit)抽象窗口工具  
- API(Application Programming Interface)应用程序接口 
- AOP  Aspect Oriented Programming（面向切面编程），可以 通过预编译方式和运行期动态代理实现在不修改源代码的情况下给程序动态统一 添加功能的一种技术。  
- BMP  Bean-Managed Persistent（Bean管理的持久性），EJB中由 Bean自己负责持久性管理的方法，Bean的内容的同步（保存）需要自己编写代码 实现。 
- I18N  internationalization（国际化），这个单词的长度是20，然后取 其首尾字母，中间省略的字母刚好18个。 


>欢迎关注公众号：**Coder编程**
获取最新原创技术文章和相关免费学习资料，随时随地学习技术知识！

参考文章：

https://blog.csdn.net/u013309870/article/details/70229983

https://blog.csdn.net/jairuschan/article/details/7513045

https://www.cnblogs.com/dolphin0520/p/3780005.html




![微信公众号](https://user-gold-cdn.xitu.io/2019/4/16/16a26835c75c12fc?w=300&h=390&f=png&s=18217)

## 推荐阅读

[一篇带你读懂TCP之“滑动窗口”协议 ](https://mp.weixin.qq.com/s?__biz=MzIwMTg3NzYyOA==&mid=2247483706&idx=1&sn=8eed9d160013bd8ed6203ad511711553&chksm=96e67029a191f93fdd1543af2bf06025397d9c3bd0f0692c7fe247ab9c139cd869d69ab05498&token=1104592742&lang=zh_CN#rd)

[带你了解数据库中JOIN的用法 ](https://mp.weixin.qq.com/s?__biz=MzIwMTg3NzYyOA==&mid=2247483713&idx=1&sn=d61ad0aed42dc36d64d17732db352288&chksm=96e67052a191f9445bbe3d5825ce547ad3171c3874b571a93b97977d0668413e37a164c3e0bc&token=1144933717&lang=zh_CN#rd)

[ 带你了解数据库中group by的用法 ](https://mp.weixin.qq.com/s?__biz=MzIwMTg3NzYyOA==&mid=2247483717&idx=1&sn=157a8a021c29043a10480d0294b39ca0&chksm=96e67056a191f940668812ebb092fe9984b22eb619a18339cc052e1051c659a7e9d907c48814&token=1144933717&lang=zh_CN#rd)