关注性能监控 与 优化
====

v8内存
----

v8 内存限制： 64位为1.4GB，32位为0.7GB， 为何有限制？

>V8之所以限制了内存的大小，表面上的原因是V8最初是作为浏览器的JavaScript引擎而设计，不太可能遇到大量内存的场景，而深层次的原因则是由于V8的垃圾回收机制的限制。由于V8需要保证JavaScript应用逻辑与垃圾回收器所看到的不一样，V8在执行垃圾回收时会阻塞JavaScript应用逻辑，直到垃圾回收结束再重新执行JavaScript应用逻辑，这种行为被称为“全停顿”（stop-the-world）。若V8的堆内存为1.5GB，V8做一次小的垃圾回收需要50ms以上，做一次非增量式的垃圾回收甚至要1秒以上。这样浏览器将在1s内失去对用户的响应，造成假死现象。如果有动画效果的话，动画的展现也将显著受到影响


突破：

 - 通过启动时参数： --max-old-space-size=1700 （老生代大小 MB） --max-new-space-size=1024 (新生代大小 KB)
 - 使用 Buffer


内存组成
----

V8的堆其实并不只是由老生代和新生代两部分构成，可以将堆分为几个不同的区域：

*　新生代内存区：大多数的对象被分配在这里，这个区域很小但是垃圾回特别频繁
*　老生代指针区：属于老生代，这里包含了大多数可能存在指向其他对象的指针的对象，大多数从新生代晋升的对象会被移动到这里
*　老生代数据区：属于老生代，这里只保存原始数据对象，这些对象没有指向其他对象的指针
*　大对象区：这里存放体积超越其他区大小的对象，每个对象有自己的内存，垃圾回收其不会移动大对象
*　代码区：代码对象，也就是包含JIT之后指令的对象，会被分配在这里。唯一拥有执行权限的内存区
*　Cell区、属性Cell区、Map区：存放Cell、属性Cell和Map，每个区域都是存放相同大小的元素，结构简单

垃圾回收器只会针对新生代内存区、老生代指针区以及老生代数据区进行垃圾回收.

V8采用了一种分代回收的策略，将内存分为两个生代：新生代和老生代。新生代的对象为存活时间较短的对象，老生代中的对象为存活时间较长或常驻内存的对象。分别对新生代和老生代使用不同的垃圾回收算法来提升垃圾回收的效率。对象起初都会被分配到新生代，当新生代中的对象满足某些条件（后面会有介绍）时，会被移动到老生代（晋升）

 - 新生代采用 复制清理算法。
 - 老生代采用 标记清除，标记整理

v8的进一步优化：

 - 增量标记，将整个标记拆分成多个部分
 - 惰性清理
 - other v8后续还引入了增量式整理（incremental compaction），以及并行标记和并行清理，通过并行利用多核CPU来提升垃圾回收的性能



references
----

 - [v8垃圾回收机制](http://segmentfault.com/a/1190000000440270)
 - [如何自己检查NodeJS的代码是否存在内存泄漏](http://www.w3ctech.com/topic/842)
 - [Debugging Memory Leaks in Node.js Applications](http://dudu.zhihu.com/story/7181000)

 - [Keeping Node.js Fast: Tools, Techniques, And Tips For Making High-Performance Node.js Servers](https://www.smashingmagazine.com/2018/06/nodejs-tools-techniques-performance-servers/)
 - [The Ultimate Guide to Performance Monitoring in Node.js](https://cdn.nlark.com/yuque/0/2018/pdf/84137/1532019646540-c93c6be5-e24c-46d8-94cc-35d221680baf.pdf?OSSAccessKeyId=LTAI1M4etAl6H5AN&Expires=1539091882&Signature=SXEQpxQyfQdLfjjYsWAfZjJAyVc%3D)