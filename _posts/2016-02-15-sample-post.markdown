---
layout: post
title: Android 内存泄漏总结
date: 2017-02-15 15:32:24.000000000 +09:00
---
内存管理的目的是我们在开发的过程中有效的避免出现内存泄漏的问题。
 ####`概念`
内存泄漏简单的讲就是该释放的内存没有被释放，一直被某个或某些实例所持有却不再被使用导致GC不能回收。
内存泄露的原因
首先了解`java内存分类策略`
#####java内存分类策略
java程序运行时内存分配策略有三种 静态分配 栈式分配和堆式分配，三种存储策略也对应使用的内存空间分别是静态存储区（方法区），栈区和堆区
+ 静态存储区（方法区）：主要用来存放静态数据，全局static数据和常量，这块内存区域在程序编译阶段就已经分配好，在程序整个运行阶段都存在
+ 栈区：当方法被执行时，方法的局部变量（包括基本数据类型和对象的引用）都在栈上创建，并在方法结束时这些局部变量所持有的内存会被自动释放，因为栈内存分配运算内置于处理器的指令集中，效率很高，但分配内存有限。
+ 堆区：又称作动态内存分配，通常是指程序运行时直接new出来的内存，也就是对象的实例，这部分内存在不使用时需要由java垃圾回收器来负责回收。

接下来说说`堆和栈的区别`
栈内存：在方法体内定义的`（局部变量）`一些基本类型的变量和对象引用变量都是方法的占、栈内存分配的，当在一段方法块内定义一个变量时，java就会在栈内为该变量分配北村空间，当超过该变量的作用域后，该变量也就无效了，分配给他的内存空间也将释放掉，该空间可以被重新利用。
堆内存：存放多有由new创建出来的对象（包括该对象其中的所有成员变量）和数组，在堆中分配的内存，将有java回收器来自动管理，在堆中产生一个数组或者对象后，还可以在栈中定义一个特殊的变量，这个变量的取值等于数组或者对象在堆内存的首地址，这个特殊的变量就是我们上面说的引用变量，我们可以通过引用变量来访问堆中的
对象或者数组。
######例子
{% highlight ruby %}
public class Sample {
    int s1 = 0;
    Sample mSample1 = new Sample();

    public void method() {
        int s2 = 1;
        Sample mSample2 = new Sample();
    }
}
Sample mSample3 = new Sample();
{% endhighlight %}

Sample中S1 mSample1 等类的成员变量存放在堆内存中，S2和 sample2 的引用存放在栈内存中，但是sample2所指向的对象存放在堆内存中，而sample3所指向的实体对象存放在堆上，包括这个对象的成员变量s1和mSample1,而自己存于栈中。
######结论、
局部变量的基本数据类型和引用存储在栈中，引用的对象实体存储于堆中---属于方法中的变量，生命周期随着方法而结束
成员变量全部存储于堆中（包括基本数据类型，引用和饮用的对象实体）---因为她们属于类，类对象是new出来用的。
java的内存管理就是对象的分配和释放问题。GC监控对象的状态就是为了更加准确，及时的释放对象，而释放对象根本的原则就是该对象不再被引用。
Java使用有向图的方式进行内存管理，可以消除引用循环的问题，例如有三个对象，相互引用，只要它们和根进程不可达的，那么GC也是可以回收它们的。这种方式的优点是管理内存的精度很高，但是效率较低。另外一种常用的内存管理技术是使用计数器，例如COM模型采用计数器方式管理构件，它与有向图相比，精度行低(很难处理循环引用的问题)，但执行效率很高。
内存泄漏的两个条件
在有向图中 存在通路与之连接 对象是无用的 永远也不会被引用满足这两个条件就是内存泄漏
内存泄漏是指无用对象持续占有内存或者无用对象内存得不到及时释放
Java内存泄漏的根本原因是什么呢？长生命周期的对象持有短生命周期对象的引用就很可能发生内存泄漏，尽管短生命周期对象已经不再需要，但是因为长生命周期持有它的引用而导致不能被回收，这就是Java中内存泄漏的发生场景。
静态集合类引起内存泄漏
静态变量的生命周期和应用程序一样，她们所持有的对象不能被释放
例如
{% highlight ruby %}
Static Vector v = new Vector(10);
for (int i = 1; i<100; i++)
{
Object o = new Object();
v.add(o);
o = null;
}
{% endhighlight %}
在这个例子中，循环申请Object 对象，并将所申请的对象放入一个Vector 中，如果仅仅释放引用本身（o=null），那么Vector 仍然引用该对象，所以这个对象对GC 来说是不可回收的。因此，如果对象加入到Vector 后，还必须从Vector 中删除，最简单的方法就是将Vector对象设置为null。

2.当集合里面的属性被修改后 在调用remove方法不起作用。
{% highlight ruby %}
public static void main(String[] args)
{
Set<Person> set = new HashSet<Person>();
Person p1 = new Person("唐僧","pwd1",25);
Person p2 = new Person("孙悟空","pwd2",26);
Person p3 = new Person("猪八戒","pwd3",27);
set.add(p1);
set.add(p2);
set.add(p3);
System.out.println("总共有:"+set.size()+" 个元素!"); //结果：总共有:3 个元素!
p3.setAge(2); //修改p3的年龄,此时p3元素对应的hashcode值发生改变

set.remove(p3); //此时remove不掉，造成内存泄漏

set.add(p3); //重新添加，居然添加成功
System.out.println("总共有:"+set.size()+" 个元素!"); //结果：总共有:4 个元素!
for (Person person : set)
{
System.out.println(person);
}
}
{% endhighlight %}
3、监听器

在java 编程中，我们都需要和监听器打交道，通常一个应用当中会用到很多监听器，我们会调用一个控件的诸如addXXXListener()等方法来增加监听器，但往往在释放对象的时候却没有记住去删除这些监听器，从而增加了内存泄漏的机会。在ondestroy()方法中置为null
4、各种连接

比如数据库连接（dataSourse.getConnection()），网络连接(socket)和io连接，除非其显式的调用了其close（）方法将其连接关闭，否则是不会自动被GC 回收的。对于Resultset 和Statement 对象可以不进行显式回收，但Connection 一定要显式回收，因为Connection 在任何时候都无法自动回收，而Connection一旦回收，Resultset 和Statement 对象就会立即为NULL。但是如果使用连接池，情况就不一样了，除了要显式地关闭连接，还必须显式地关闭Resultset Statement 对象（关闭其中一个，另外一个也会关闭），否则就会造成大量的Statement 对象无法释放，从而引起内存泄漏。这种情况下一般都会在try里面去的连接，在finally里面释放连接。

5、内部类和外部模块的引用

内部类的引用是比较容易遗忘的一种，而且一旦没释放可能导致一系列的后继类对象没有释放。此外程序员还要小心外部模块不经意的引用，例如程序员A 负责A 模块，调用了B 模块的一个方法如： public void registerMsg(Object b); 这种调用就要非常小心了，传入了一个对象，很可能模块B就保持了对该对象的引用，这时候就需要注意模块B 是否提供相应的操作去除引用
6、单例模式

不正确使用单例模式是引起内存泄漏的一个常见问题，单例对象在初始化后将在JVM的整个生命周期中存在（以静态变量的方式），如果单例对象持有外部的引用，那么这个对象将不能被JVM正常回收，导致内存泄漏，考虑下面的例子：

android中常见的内存泄漏汇总
###单例造成的内存泄漏

由于单例的静态特性使得其生命周期跟应用的生命周期一样长，所以如果使用不恰当的话，很容易造成内存泄漏。比如下面一个典型的例子，

public class AppManager {
private static AppManager instance;
private Context context;
private AppManager(Context context) {
this.context = context;
}
public static AppManager getInstance(Context context) {
if (instance == null) {
instance = new AppManager(context);
}
return instance;
}
}
这是一个普通的单例模式，当创建这个单例的时候，由于需要传入一个Context，所以这个Context的生命周期的长短至关重要：

1、如果此时传入的是 Application 的 Context，因为 Application 的生命周期就是整个应用的生命周期，所以这将没有任何问题。

2、如果此时传入的是 Activity 的 Context，当这个 Context 所对应的 Activity 退出时，由于该 Context 的引用被单例对象所持有，其生命周期等于整个应用程序的生命周期，所以当前 Activity 退出时它的内存并不会被回收，这就造成泄漏了。

正确的方式应该改为下面这种方式：

public class AppManager {
private static AppManager instance;
private Context context;
private AppManager(Context context) {
this.context = context.getApplicationContext();// 使用Application 的context
}
public static AppManager getInstance(Context context) {
if (instance == null) {
instance = new AppManager(context);
}
return instance;
}
}
或者这样写，连 Context 都不用传进来了：

在你的 Application 中添加一个静态方法，getContext() 返回 Application 的 context，

...

context = getApplicationContext();

...
   /**
     * 获取全局的context
     * @return 返回全局context对象
     */
    public static Context getContext(){
        return context;
    }

public class AppManager {
private static AppManager instance;
private Context context;
private AppManager() {
this.context = MyApplication.getContext();// 使用Application 的context
}
public static AppManager getInstance() {
if (instance == null) {
instance = new AppManager();
}
return instance;
}
}
因为非静态内部类默认会持有外部类的引用
###匿名内部类

android开发经常会继承实现Activity/Fragment/View，此时如果你使用了匿名类，并被异步线程持有了，那要小心了，如果没有任何措施这样一定会导致泄露
Handler 的使用造成的内存泄漏问题应该说是最为常见了，很多时候我们为了避免 ANR 而不在主线程进行耗时操作，在处理网络任务或者封装一些请求回调等api都借助Handler来处理，但 Handler 不是万能的，对于 Handler 的使用代码编写一不规范即有可能造成内存泄漏。另外，我们知道 Handler、Message 和 MessageQueue 都是相互关联在一起的，万一 Handler 发送的 Message 尚未被处理，则该 Message 及发送它的 Handler 对象将被线程 MessageQueue 一直持有。
在Android应用的开发中，为了防止内存溢出，在处理一些占用内存大而且声明周期较长的对象时候，可以尽量应用软引用和弱引用技术。

不要在类初始时初始化静态成员。可以考虑lazy初始化。 架构设计上要思考是否真的有必要这样做，尽量避免。如果架构需要这么设计，那么此对象的生命周期你有责任管理起来。

###一些不良代码造成的内存压力

有些代码并不造成内存泄露，但是它们，或是对没使用的内存没进行有效及时的释放，或是没有有效的利用已有的对象而是频繁的申请新内存。

比如： Bitmap 没调用 recycle()方法，对于 Bitmap 对象在不使用时,我们应该先调用 recycle() 释放内存，然后才它设置为 null. 因为加载 Bitmap 对象的内存空间，一部分是 java 的，一部分 C 的（因为 Bitmap 分配的底层是通过 JNI 调用的 )。 而这个 recyle() 就是针对 C 部分的内存释放。 构造 Adapter 时，没有使用缓存的 convertView ,每次都在创建新的 converView。这里推荐使用 ViewHolder。

##总结

对 Activity 等组件的引用应该控制在 Activity 的生命周期之内； 如果不能就考虑使用 getApplicationContext 或者 getApplication，以避免 Activity 被外部长生命周期的对象引用而泄露。

尽量不要在静态变量或者静态内部类中使用非静态外部成员变量（包括context )，即使要使用，也要考虑适时把外部成员变量置空；也可以在内部类中使用弱引用来引用外部类的变量。

对于生命周期比Activity长的内部类对象，并且内部类中使用了外部类的成员变量，可以这样做避免内存泄漏：

    将内部类改为静态内部类
    静态内部类中使用弱引用来引用外部类的成员变量
Handler 的持有的引用对象最好使用弱引用，资源释放时也可以清空 Handler 里面的消息。比如在 Activity onStop 或者 onDestroy 的时候，取消掉该 Handler 对象的 Message和 Runnable.

在 Java 的实现过程中，也要考虑其对象释放，最好的方法是在不使用某对象时，显式地将此对象赋值为 null，比如使用完Bitmap 后先调用 recycle()，再赋为null,清空对图片等资源有直接引用或者间接引用的数组（使用 array.clear() ; array = null）等，最好遵循谁创建谁释放的原则。

正确关闭资源，对于使用了BraodcastReceiver，ContentObserver，File，游标 Cursor，Stream，Bitmap等资源的使用，应该在Activity销毁时及时关闭或者注销。

保持对对象生命周期的敏感，特别注意单例、静态对象、全局性集合等的生命周期。

[jekyll-docs]: http://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
