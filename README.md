# blog
这里面包含我平常工作中遇到的坑（主要是node方面的），以及node源码粗读系列文章（我会持续更新）。  
[issue](https://github.com/xtx1130/blog/issues)

### node 源码粗读系列
整体文章比较干，也不太适合node入门者阅读，需要有一定的node基础。

虽然叫粗读，但是大部分文章都会深扒细节，不是对api或者文件的简单介绍，而是深入去了解这个api怎么实现的或者这个文件是做什么用、在哪里调用的。

希望这些文章可以成为你在读node源码时候的参考。

*listing:*  
- [node源码粗读（1）：一个简单的nodejs文件从运行到结束都发生了什么](https://github.com/xtx1130/blog/issues/5)
- [node源码粗读（2）：node编译过程详解及如何在本地进行源码修改和调试](https://github.com/xtx1130/blog/issues/9)
- [node源码粗读（3）：node_js2c的前世今生](https://github.com/xtx1130/blog/issues/10)
- [node源码粗读（4）：process对象底层实现](https://github.com/xtx1130/blog/issues/12)
- [node源码粗读（5）：通过调试./lib库的js代码来看javascript在node中运行环境的变化](https://github.com/xtx1130/blog/issues/14)
- [node源码粗读（6）：从./lib/timers.js来看timers相关API底层实现](https://github.com/xtx1130/blog/issues/15)
- [node源码粗读（7）：nextTick和microtasks从bootstrap到event-loop全阶段解读](https://github.com/xtx1130/blog/issues/16)
- [node源码粗读（8）：setImmediate注册+触发全流程解析](https://github.com/xtx1130/blog/issues/19)
- [node源码粗读（9）：nextTick、timers API、MicroTasks注册到执行全阶段解读](https://github.com/xtx1130/blog/issues/20)
- [node源码粗读（10）：通过fs.write来看异步I/O和回调执行的整体流程](https://github.com/xtx1130/blog/issues/23)
### 踩坑系列
工作中遇到的难踩的坑

*listing:*
- [针对于extends koa2 导致babel转义时造成错误的处理方案](https://github.com/xtx1130/blog/issues/1)
- [关于chrome浏览器无法正确处理favicon.ico返回的问题](https://github.com/xtx1130/blog/issues/3)
- [AnimateCC和createjs遇到的坑](https://github.com/xtx1130/blog/issues/11)
- [弹幕效果遇到的坑](https://github.com/xtx1130/blog/issues/13)
- [H5全屏视频+暂停交互遇到的坑和解决办法](https://github.com/xtx1130/blog/issues/17)
### 入门系列
一些包或框架的用法或原理

*listing*
- [docker + node 入门](https://github.com/xtx1130/blog/issues/4)
- [Babylon和babel-traverse详解](https://github.com/xtx1130/blog/issues/7)
- [nextjs cli部分源码解读](https://github.com/xtx1130/blog/issues/18)
- [nextTick中为什么需要async_hooks](https://github.com/xtx1130/blog/issues/22)
### 文档翻译
- [node v8.x async_hook](https://github.com/xtx1130/blog/blob/master/doc/async_hook_zh_CN.md)
