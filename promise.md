## JavaScript异步机制

* 回调函数
* Promise
* 封装ajax
* async/await 


JavaScript的单线程机制是指浏览器在指向js代码时只开启了一个js引擎线程来解析执行js，并不代表浏览器的工作方式是单线程的，一些I/O操作，定时器都是由别的线程来执行。

单线程也就意味着同一时间只能执行一个任务，下一个任务必须等待当前任务的结束，遇到比较耗时的任务页面就会被挂起，异步机制就是用来解决这个问题

### 回调函数
最早用来实现异步的方法是回调函数
``` JavaScript
function f1(callback) {
  setTimeout(() => {
    //执行回调函数
    callback()
  },1000)
} 

f1()
console.log('我会先被执行')
```

在上面的代码中，使用了定时器来实现了一个简单的异步操作，f1的执行并不会阻塞代码。回调函数实现异步的方式很简单，也比较容易理解，但当回调嵌套过多时，也就出现了**回调地狱**的问题，代码变的难以阅读并且不好调试


### Promise
Promise的出现解决了地狱回调的问题，可以使用链式的方式来执行异步操作，直观上是同步的，从上往下执行

**链式操作**
``` JavaScript
function run1() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log('第一个异步任务')
      resolve(1)
    },1000)
  })
}

function run2() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log('第二个异步任务')
      resolve(2)
    },1000)
  })
}

function run3() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      console.log('第三个异步任务')
      resolve(3)
    },1000)
  })
}

run1()
.then(data => {
  console.log(data)
  return run2()
})
.then(data => {
  console.log(data)
  return run3()
})
.then(data => {
  console.log(data)
})

```

上面代码的执行结果是每隔一秒依次输出异步回调中的内容，Promise通过状态来维护异步状态，分别是pending（等待）、resolved（完成）和rejected（拒绝），用来表示执行成功和执行失败的回调，reject的状态就能在catch中被捕获。

**Promise.all:**提供了并行执行异步操作的能力，这里的并行并不是并发执行的意思，Promise.all()能接受一个异步函数数组，等它们都执完后的结果会由一个then导出，结果也是一个数组，Promise.all()的应用场景比如说资源加载比较多的页面，当所有的资源加载完后再渲染到页面。

**Promise.race:**race本身表达的含义就是竞速，执行完第一个异步操作就结束，谁快谁先执行，与Promise.all相反，Promise.all是等待最慢的一个执行结束一起输出，Promise.race()常见的应用场景是异步任务超时提醒

``` JavaScript
//模拟资源异步请求
function getImg() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const data = 'urlxxx'
      resolve(data)
    },2000)
  })
}
//超时函数
function timeout() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject('请求图片超时')
    },1500)
  })
}

Promise.race([getImg(), timeout()])
.then(data => {
  console.log(data)
})
.catch(err => {
  console.log(err)
})
```

### 封装ajax

原生ajax实现异步的方式是回调函数，也存在了地狱回调的问题
``` JavaScript
let myAjax = function (method, url, success) {
  let xhr = new XMLHttpRequest()
  xhr.open(method, url)
  xhr.onreadystatechange = function () {
    if (xhr.readyState === 4) {
      if(xhr.status === 200) {
        success(xhr.responseText)        //成功回调
      } else {
        console.log('请求失败')
      }
    }
  }
  xhr.send()
}
//现在的逻辑是需要在a.json请求成功的前提上再去请求b.json
myAjax('GET','./files/a.json',data => {
  console.log(data)
  myAjax('GET','./files/b.json',data => {        
    console.log(data)
  })P
})
```

使用Promise封装ajax来解决该问题

``` JavaScript
let myAjax = function(method, url) {
  return new Promise((resolve, reject) => {
    let xhr = new XMLHttpRequest()
    xhr.open(method, url)
    xhr.onreadystatechange = function() {
      if(xhr.readyState === 4) {
        if(xhr.status === 200) {
          resolve(xhr.responseText)
        }else{
          reject(xhr.status)
        }
      }
    }
    xhr.send()
  })
}

myAjax('GET', './files/a.json')                //使用promise封装后的ajax
.then(data => {
  console.log('请求a成功',data)
  return myAjax('GET', './fxiles/b.json')
})
.then(data => {
  console.log('请求b成功',data)
})
.catch(err => {
  console.log(err)
})
```
异步代码被*同步化*，回调地狱的问题解决了。

Promise解决了地狱回调的问题，将异步流程的代码用同步的方式表达出来，但是Promise也并不是一个完美的异步解决方案，Promise并没有完全摆脱回调，then链式写法也不是很清晰简单

### async/await 

ES7中提出了async/await的方式来书写异步流程，它本质上是语法糖，集合Promise彻底摆脱了回调的问题，真正用“同步”的方式来写异步函数

**async:**会将函数转换成一个async function，此时函数返回的是一个Promise对象

**await:**必须在async function内部使用，置于异步函数之前，暂停在该代码，直到promise完成

使用async/await重写Promise的链式调用
``` JavaScript
myAjax('GET', './files/a.json')
.then(data => {
  console.log('请求a成功',data)
  return myAjax('GET', './fxiles/b.json')
})
.then(data => {
  console.log('请求b成功',data)
})
.catch(err => {
  console.log(err)
})

//使用async/await重写

async function fun() {
  let data1 = await myAjax('GET','./files/a.json')
  let data2 = await myAjax('GET','./files/b.json')
  console.log(data1,data2)
}
fun()
```

async/await使得代码非常直观，如果想使用异常捕获，可以使用同步的try...catch的结构来写

