## 前端错误捕获与上报

> 前端监控涉及到数据监控，也就是监听用户行为；性能监控，监听页面渲染时间；以及异常监控，目前仅需要异常监控，在最近的项目中增加了前端错误捕获以及上报的需求，前端错误上报和后台错误日志一样重要，用于快速定位错误原因和提前发现错误。

* 异常处理
* 错误上报

### 1、异常处理

#### 1.1、try-catch语句

try-catch语句是js中用于处理异常的常用方式，代码块在try中执行，如果出现错误，会停止代码的运行，将错误信息捕获到catch中输出。除了try和catch，还有一个**finally**子句，它表示无论如何都会执行最后一个代码块。

``` JavaScript
try {
  const a = 1
  a = 2
  console.log(a)
} catch (error) {
  console.log(error)                      // 打印出错误原因
  return false
} finally {
  console.log('anyhow,I will appear')     // 正常打印出，没有受到catch中return的影响
}
```

**注意**：只要在代码中使用了finally，try和catch中的return语句都会被忽略

#### 1.2 常见错误类型

**Error:** 是所有错误类型的基类，一般用于自定义错误类型，例子：throw new Error('意想不到的错误')

**EvalError:** 简单理解，没有把eval当成函数来使用

**RangeError:** 数值超出相应范围时报错，例子：let arr1 = new Array(-1)，数组长度不能为负数

**ReferenceError:** 引用错误，访问了不存在的对象，找不到该对象就会报错

**TypeError:** 类型错误，最常见的错误类型，例子：const a = 1;a = 2;修改了const类型的常量

#### 1.3 错误事件

try-catch针对于可能出现错误位置的异常处理，try-catch也不是越多越好。所以需要全局的异常捕获事件，没有被try-catch所捕获的事件都会触发window对象的error事件，并且try-catch无法捕获异步异常

``` JavaScript
// window.onerror
window.onerror = function (errMessage, URI, row, column, error) {
  console.log('errMessage:',errMessage)             // 错误信息
  console.log('URI:',URI)                           // 出错的文件路径
  console.log('row:', row)                          // 错误行数
  console.log('column:', column)                    // 错误列数   
  console.log('error:', error)                      // 详细堆栈信息
  return true
}
```
在window.onerror()中最后需要return true，异常才不会向上抛出，window.onerror()也存在一些问题，它无法捕获后台接口访问异常，因为网络请求异常不会**事件冒泡**。可以采用window.addEventListener()来替换window.onerror()

``` JavaScript
// window.addEventListener
window.addEventListener('error', function(errMessage, URI, row, column, error) {
  console.log('errMessage:',errMessage)             // 错误信息
  console.log('URI:',URI)                           // 出错的文件路径
  console.log('row:', row)                          // 错误行数
  console.log('column:', column)                    // 错误列数   
  console.log('error:', error)                      // 详细堆栈信息
}, true)
```

这种方法虽然可以捕获到网络请求的异常，但是无法捕获一些重要的信息，比如HTTP状态码等，还需要配合服务端的日志。

#### 1.4、捕获Promise异常

无论是winow.onerror还是window.addEveneListener都无法捕获promise的异常，需要监听unhandledrejection事件来捕获未被catch的promise错误。

#### 1.5、vue中的异常处理

vue提供了一个全局配置errorHandler，用于捕获vue运行时出现的错误

``` JavaScript
Vue.config.errorHandler = function (err, vm, info) {
  // handle error
  // `info` 是 Vue 特定的错误信息，比如错误所在的生命周期钩子
  // 只在 2.2.0+ 可用
  let componentName = formatComponentName(vm);
  console.log('err', err)
  console.log('componentName', componentName)
  console.log('info', info)
}
// 获取组件的名称
function formatComponentName(vm) {
  if (vm.$root === vm) return 'root';
  let name = vm._isVue
    ? (vm.$options && vm.$options.name) ||
    (vm.$options && vm.$options._componentTag)
    : vm.name;
  return (
    (name ? 'component <' + name + '>' : 'anonymous component') +
    (vm._isVue && vm.$options && vm.$options.__file
      ? ' at ' + (vm.$options && vm.$options.__file)
      : '')
  );
}
```

### 2、错误上报

#### 2.1、使用ajax发送异常数据

使用ajax来上报异常本身就可能会出现问题，如果前端日志服务器是独立出来的，那么就会出现跨域的问题，也需要先解决跨域问题。

#### 2.2、使用img标签的形式

由于图片请求不会涉及到跨域的问题。

``` JavaScript
function report(error) {
  let reportUrl = 'http://xxxx'
  new Image().src = `${reportUrl}?logs=${error}`
}
```

**小结：** 关于前端的错误捕获与上报，分了几种情况，具体问题具体分析，跨域组合一些方式来对绝大多数的异常进行捕获。

1、对于可能发送错误的区域设置try-catch，比如获取异步数据async-await中使用

2、全局异常监控winow.onerror()，不能捕获访问资源异常

3、window.addEventListener("error",callback())，可以捕获到访问资源的异常，但捕获的信息不是很完善

4、对于没有被catch的promise异常，全局设置一个unhandledrejection来统一处理

5、vue中可以使用全局的errorHandler来处理

6、上报可以直接使用空的imageUrl来构造，避免跨域的问题










