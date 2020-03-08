### 简介

lldb提供了3种指令来打印变量，分别是po、p、v、今天我们就来看看这3种指令的实现方式和区别。

### po指令

![429_lldb_beyond_pox](/Users/liyuzhu/Desktop/429_lldb_beyond_pox.jpg)

po指令可以看成是打印变量的固定指令，当我们使用po的时候，像上图一样我们得到的是文本形式的对象描述。po不仅仅能打印变量，还能像下图一样拿到对象的名字并计算名字的大写版本。

![429_lldb_beyond_pos](/Users/liyuzhu/Desktop/429_lldb_beyond_pos.jpg)

事实上它可以计算任何表达式，其实po只是一个expression表达式的别名，你可以用command alias指令实现自定义的po指令

![429_lldb_beyond_pocc](/Users/liyuzhu/Desktop/429_lldb_beyond_pocc.jpg)

下面我们来看看po的底层是如何工作的

![429_lldb_beyond_po](/Users/liyuzhu/Desktop/429_lldb_beyond_po.jpg)

为了提供完整描述，首先它会根据你输入的表达式生成一段代码，然后会在debug程序里编译并执行这段代码来计算表达式的结果，LLDB拿到结果之后会生成另一短代码来格式化结果输出字符串

### p指令



![429_lldb_beyond_por](/Users/liyuzhu/Desktop/429_lldb_beyond_por.jpg)

p指令和po指令打印出来的格式有点不一样，但是内容是一样的。而且返回值被起名为$R0，这是LLDB的一个约定，每个表达式的结果会被赋予一个递增的名字。像$R0, $R1,并且这个被赋予的名字可以用在稍后的表达式中。你可以像使用其他变量一样使用$R0

![429_lldb_beyond_poRRQ](/Users/liyuzhu/Desktop/429_lldb_beyond_poRRQ.jpg)

下面让我们看看p指令在后台是如何运行的

![429_lldb_beyond_p](/Users/liyuzhu/Desktop/429_lldb_beyond_p.jpg)

事实上p指令底层的第一部分跟po指令是一样的，会先计算计算表达式的结果，一旦拿到结果LLDB会对结果执行动态类型解析。至于什么是动态类型解析，可以用一个例子说明一下

![429_lldb_beyond_poddd](/Users/liyuzhu/Desktop/429_lldb_beyond_poddd.jpg)

在上面的例子中，Trip遵守了Activity协议，在swift中源码中的静态类型和运行时的动态类型可以是不一样的，像上面的例子一样，可以用协议的类型去声明cruise对象。cruise的静态类型是Acitivity，但是运行时类型是Trip，如果我们打印cruise我们得到的也是Trip类型，因为LLDB会根据元数据展示给定变量的精确类型。这就是动态类型解析

在p指令下，动态类型解析只会对表达式的结果执行，当我们想打印cruise的name的时候，这时cruise的类型是Activity，没有name属性，所以就会出错，如果要消除错误可以把对象转化为其动态类型

当对结果执行完动态类型解析，LLDB会把解析的结果传递给格式化子系统来打印出可读的字符串

如果你想之后没有格式化的时候数据是什么样，可以使用 expression --raw --指令查看

![429_lldb_beyond_poggg](/Users/liyuzhu/Desktop/429_lldb_beyond_poggg.jpg)

### v指令

![](/Users/liyuzhu/Desktop/429_lldb_beyond_vvvv.jpg)



v指令的输出也是依赖LLDB的格式化子系统，v指令是在xcode10才引入的，不像其他两个指令，v不会编译和执行代码，所以v的执行更快速。v指令有自己的语法，而且它的语法可以跟你在调试的程序的语言的语法不一样。它可以用.和下标去获取变量。

![429_lldb_beyond_pojjjjj](/Users/liyuzhu/Desktop/429_lldb_beyond_pojjjjj.jpg)

v指令不能计算表达式，因为计算需要执行代码，如果要执行计算表达式可以使用po或者p

![429_lldb_beyond_vvvd](/Users/liyuzhu/Desktop/429_lldb_beyond_vvvd.jpg)

v根据程序描述去在内存种定位变量的位置，然后从内存中读取变量，然后对结果执行动态类型解析，当用于想访问子字段，它就对每一个子字段重复执行动态类型解析，一旦完成，就会把结果移交给格式化子系统。

v可能会执行多次动态类型解析，但是格式化只有一次，只是在最后才格式化

### 总结

下面是三种指令的对比

![429_lldb_beyond_pocccc](/Users/liyuzhu/Desktop/429_lldb_beyond_pocccc.jpg)

### 参考

#### [WWDC 2018.LLDB: Beyond "po"](https://developer.apple.com/videos/play/wwdc2019/429/)







