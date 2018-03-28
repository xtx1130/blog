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

#### Error Handling

如果任何`AsyncHook`回调抛错，应用都会显示相应的堆栈跟踪的信息然后退出。退出路径会遵循未捕获异常，但是所有的`uncaughtException`事件监听都会被移除，因此会强制进程退出。`exit`回调会在你开启`--abort-on-uncaught-exception`参数之后才会被调用，




