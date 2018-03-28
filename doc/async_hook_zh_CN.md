# Async Hooks

稳定性：1 - 实验性的

`async_hooks`模块提供了一个APi，通过注册回调的方式追踪在Node.js应用中创建的异步资源的生命周期。引用方法如下：
```js
const async_hooks = require('async_hooks');
```

## 术语

每个异步资源代表了一个含有关联回调的对象。这个回调也许会被调用很多次，例如，`net.createServer`中的`connection`时间，亦或是类似`fs.open`仅调用一次。资源也可以在回调调用之前关闭。AsyncHook没有在这些例子之中做明确区分，但是会将他们表示为资源的抽象概念。

## 开放的API

### 概述

下面是一个简单的开放出来的API的概述。
```js
const async_hooks = require('async_hooks');

// 返回当前正在执行的上下文的id
const eid = async_hooks.executionAsyncId();

// 返回当前执行上下文要触发的回调的句柄id
const tid = async_hooks.triggerAsyncId();

// 创建一个新的AsyncHook实例。所有的回调都是可配置的
const asyncHook =
    async_hooks.createHook({ init, before, after, destroy, promiseResolve });

// 允许新创建的AsyncHook实例的回调被调用. 在调用了这个构造函数之后不会隐性
// 调用这个方法， 必须显式调用此方法才会开始执行实例的回调。
asyncHook.enable();

// 关闭对新的异步事件的监听。
asyncHook.disable();

//
// 下面的函数是可以传递给createHook()中的回调.
//

// init在对象创建的时候调用。 在这个回调（init）运行的时候可能资源还没有创建完，
// 所以被“asyncId”代表的资源也许还缺少一些字段。
function init(asyncId, type, triggerAsyncId, resource) { }

// before会在资源的回调调用前进行调用. 它可以为了句柄调用0~N次，（例如 TCPWrap），亦或是为了发起请求只调用一次（例如 FSReqWrap）。
function before(asyncId) { }

// after会在资源的回调运行之后运行。.
function after(asyncId) { }

// destroy 会在AsyncWrap实例被销毁的时候运行。.
function destroy(asyncId) { }

// promiseResolve只会被promise资源调用，在把`resolve`函数传递给`Promise`构造函数的时候才会被调用（或者是直接或通过别的方式使得promise被resolve的时候）。
function promiseResolve(asyncId) { }
```

### `async_hooks.createHook(callbacks)`

添加于：v8.1.0

- `callbacks` <Object> 注册的钩子回调
  - `init` <Function> init 回调
  - `before` <Function> before 回调
  - `after` <Function> after 回调
  - `destroy` <Function> destory回调
- 返回：用来开启或关闭钩子的`{AstncHook}`实例

注册函数在每个异步操作的不同生命周期中被调用。

`init()`/`before()`/`after()`/`destory()`会在资源生命周期各自的异步事件中被调用。

所有的回调都是可配置的。例如，如果只需要在资源回收的时候跟踪，那么只需要传递`destroy`回调。在`Hook Callbacks`章节中有明确的可以传递给`callbacks`的所有函数。

```js
const async_hooks = require('async_hooks');

const asyncHook = async_hooks.createHook({
  init(asyncId, type, triggerAsyncId, resource) { },
  destroy(asyncId) { }
});
```
注意：回调可以通过原型链继承：
```js
class MyAsyncCallbacks {
  init(asyncId, type, triggerAsyncId, resource) { }
  destroy(asyncId) {}
}

class MyAddedCallbacks extends MyAsyncCallbacks {
  before(asyncId) { }
  after(asyncId) { }
}

const asyncHook = async_hooks.createHook(new MyAddedCallbacks());
```

#### 错误处理

如果任何`AsyncHook`回调抛错，应用都会显示相应的堆栈跟踪的信息然后退出。退出路径会遵循未捕获异常，但是所有的`uncaughtException`事件监听都会被移除，因此会强制进程退出。`exit`回调会被调用，除非你的应用开启了`--abort-on-uncaught-exception`参数，在这种情况下应用的堆栈信息会打印出来，并且在应用退出的时候会留下一份内存快照。

错误处理这么表现的原因是这些回调会在一个对象生命周期的不稳定阶段运行，例如在类的创建和销毁期间。正是如此，nodejs认为快速关闭这个进程是很有必要的，目的是阻止将来意外中止的发生。这是未来一个需要改进的主题：提供一个更加全面的分析去确保在执行的时候异常可以遵循正常的工作流程，没有多余的副作用。

#### AsyncHooks 回调的打印

由于打印到console是一个异步的操作，`console.log()`会导致AsyncHooks回调的调用。所以在AsyncHooks的回调函数中使用`console.log()`或者类似的异步操作会导致无限递归。一个十分简单的解决办法是在调试的时候使用同步的日志操作，例如`fs.writeSync(1, msg)`。这会打印到stdout，因为`1`是stdout的文件描述符，并且不会递归调用AsyncHooks因为它是同步的。
```js
const fs = require('fs');
const util = require('util');

function debug(...args) {
  // use a function like this one when debugging inside an AsyncHooks callback
  fs.writeSync(1, `${util.format(...args)}\n`);
}
```
如果一个异步操作需要记录日志，尽可能的使用AsyncHooks自己提供的信息来跟踪整体异步操作的流程。日志应该忽略由于日志自己的原因导致AsyncHooks回调调用。

### `asyncHook.enable()`

- 返回：<AsyncHook> `asyncHook`的引用
开启已有的`AsyncHook`实例的回调。如果没有提供回调函数，开启的会是一个空函数。

`AsyncHook`实例默认是关闭的。如果要在创建之后立刻开启，可以参考下面的形式。
```js
const async_hooks = require('async_hooks');

const hook = async_hooks.createHook(callbacks).enable();
```

### `asyncHook.disable()`

- 返回：<AsyncHook> `asyncHook`的引用
将某个`AsyncHook`实例的回调在全局asyncHook回调池中禁用掉。钩子在被禁用掉之后，只有通过重新启用才能继续呗调用到。  
为了API的一致性，`disable()`也会返回`AsyncHook`的实例。

### 回调钩子

异步事件生命周期中关键事件被分为四块：实例化、回调之前/之后、以及实例被销毁时。

#### `init(asyncId, type, triggerAsyncId, resource)`

- `asyncId` <number> 异步资源的唯一id
- `type` <string> 异步资源的类型
- `triggerAsyncId` <number> 异步资源在执行上下文创建时候的唯一ID
- `resource` <Object> 异步操作的资源的引用，在被 *销毁* 的时候需要释放

这个钩子会在 *有可能* 发布异步事件的类实例化的时候调用。这并 *不代表* 这个实例一定会在`destroy`回调调用前调用`before`/`after`回调，只是存在这种可能性。

通过打开资源，然后在资源可用之前关闭它，可以观察到这种行为。下面的片段演示了这一点。
```js
require('net').createServer().listen(function() { this.close(); });
// OR
clearTimeout(setTimeout(() => {}, 10));
```
每个新资源都分配了一个在当前进程范围内唯一的ID。

##### `type`

`type`是一个字符串，用来鉴别调用`init`的资源的类型。通常情况，它和资源的constructor是对应关系。
```js
FSEVENTWRAP, FSREQWRAP, GETADDRINFOREQWRAP, GETNAMEINFOREQWRAP, HTTPPARSER,
JSSTREAM, PIPECONNECTWRAP, PIPEWRAP, PROCESSWRAP, QUERYWRAP, SHUTDOWNWRAP,
SIGNALWRAP, STATWATCHER, TCPCONNECTWRAP, TCPSERVER, TCPWRAP, TIMERWRAP, TTYWRAP,
UDPSENDWRAP, UDPWRAP, WRITEWRAP, ZLIB, SSLCONNECTION, PBKDF2REQUEST,
RANDOMBYTESREQUEST, TLSWRAP, Timeout, Immediate, TickObject
```
资源类型为`PROMISE`也应该算在里面，用来跟踪`Promise`实例以及它们安排的异步工作。

使用者可以通过embedder API来自定义自己的`type`。

注意：类型名是有可能冲突的。Embedders鼓励使用独特的前缀，例如npm包的名字，来避免在监听钩子时候的冲突。

##### triggerId

`triggerAsyncId`是引起（或触发）新资源初始化的资源`asyncId`，并且它会导致`init`的调用。`async_hooks.executionAsyncId()`仅说明 资源是 *什么时候* 创建的，但是`triggerAsyncId`说明 的是 *为什么*这个资源会被创建。

下面是`triggerAsyncId`的简单演示。
```js
async_hooks.createHook({
  init(asyncId, type, triggerAsyncId) {
    const eid = async_hooks.executionAsyncId();
    fs.writeSync(
      1, `${type}(${asyncId}): trigger: ${triggerAsyncId} execution: ${eid}\n`);
  }
}).enable();

require('net').createServer((conn) => {}).listen(8080);
```
使用`nc localhost 8000`指令击中服务器的时候会有如下输出：
```js
TCPSERVERWRAP(2): trigger: 1 execution: 1
TCPWRAP(4): trigger: 2 execution: 0
```
`TCPSERVERWRAP` 是接受链接的服务器。  
`TCPWRAP` 是来自客户端的新链接。当建立新的连接时，立即构建`TCPWrap`实例。这是在JavaScript栈外发生的（边注：`executionAsyncId()`为`0`时，意为它是通过c++执行的，且在这之上没有JavaScript栈）。通过这些信息，不可能根据谁创建的它们而将资源归类到一起。因此`triggerAsyncId`被赋予的任务是在新资源存在情况下表示出这个资源代表的是什么。

##### `resource`

`resource`代表已经初始化的异步资源的对象。这里面存储的信息与`type`的值相关。例如，对于`GETADDRINFOREQWRAP`资源类型，`rescource`提供在`net.Server.listen（）`中用来查找主机名的IP地址时使用的主机名。用来访问这些信息的API暂时不会公开，但是通过Embedder API，用户可以提供并记录属于他们自己的资源对象。例如，一个包含正在执行SQL查询的资源对象。

在Promises的情况下，`resource`会具有`promise`属性，该属性指向正在初始化的Promise,并且`parentId`属性设置为父Promise的`asyncId`，如果不存在父Promise则设为`undefined`。例如，`b = a.then(handler)`，`a`被认为是`b`的父Promise。

##### 异步上下文例子

下面是一个在`before`和`after`中间调用`init`的例子，特别注意`listen()`的回调会是什么样子。输出格式稍微复杂些，以使调用上下文更容易被观察到。
```js
let indent = 0;
async_hooks.createHook({
  init(asyncId, type, triggerAsyncId) {
    const eid = async_hooks.executionAsyncId();
    const indentStr = ' '.repeat(indent);
    fs.writeSync(
      1,
      `${indentStr}${type}(${asyncId}):` +
      ` trigger: ${triggerAsyncId} execution: ${eid}\n`);
  },
  before(asyncId) {
    const indentStr = ' '.repeat(indent);
    fs.writeSync(1, `${indentStr}before:  ${asyncId}\n`);
    indent += 2;
  },
  after(asyncId) {
    indent -= 2;
    const indentStr = ' '.repeat(indent);
    fs.writeSync(1, `${indentStr}after:   ${asyncId}\n`);
  },
  destroy(asyncId) {
    const indentStr = ' '.repeat(indent);
    fs.writeSync(1, `${indentStr}destroy: ${asyncId}\n`);
  },
}).enable();

require('net').createServer(() => {}).listen(8080, () => {
  // Let's wait 10ms before logging the server started.
  setTimeout(() => {
    console.log('>>>', async_hooks.executionAsyncId());
  }, 10);
});
```
仅开启服务器的时候的输出：
```js
TCPSERVERWRAP(2): trigger: 1 execution: 1
TickObject(3): trigger: 2 execution: 1
before:  3
  Timeout(4): trigger: 3 execution: 3
  TIMERWRAP(5): trigger: 3 execution: 3
after:   3
destroy: 3
before:  5
  before:  4
    TTYWRAP(6): trigger: 4 execution: 4
    SIGNALWRAP(7): trigger: 4 execution: 4
    TTYWRAP(8): trigger: 4 execution: 4
>>> 4
    TickObject(9): trigger: 4 execution: 4
  after:   4
after:   5
before:  9
after:   9
destroy: 4
destroy: 9
destroy: 5
```
注意：如示例中所示，`executionAsyncId()`和`execution`都指定了当前执行上下文的值；这是通过`before`和`after`的调用决定的。  
仅使用`execution`来绘制资源分配结果如下：
```js
TTYWRAP(6) -> Timeout(4) -> TIMERWRAP(5) -> TickObject(3) -> root(1)
```



