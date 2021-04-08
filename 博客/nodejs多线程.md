# nodejs 多线程

参考资料：

[理解Node.js中的"多线程"](https://zhuanlan.zhihu.com/p/74879045)
[node child_process模块](https://www.cnblogs.com/cangqinglang/p/9886657.html)

## node server的高性能IO模型

**node server从接收一个请求到发出响应，做了些什么：**

1.首先请求会通过网线到达到达网卡的buffer缓冲区。如果当前网卡正在处理其它请求，那么这个请求会在buffer里面阻塞
2.当网卡的buffer队列轮到这个请求的时候，网卡会发出一个时钟打断给CPU，CPU收到时钟打断的时候会切换到内核态，并运行node server来处理这个请求
3.node server是单线程的，这个时候如果它正在处理其它请求，那么会被阻塞，直到事件循环轮到这个请求。
4.当node server的事件循环轮到这个请求的时候，它开始处理这个请求，并开始阻塞所有其他请求
5.node server对这个请求进行解包，拿出需要的参数，并通过计算，生成响应数据，并包装成http响应发回给网卡，此时阻塞结束，node server可以处理其它请求
6.网卡的buffer接收http响应，如果此时正在发送其它http响应，就会在buffer响应队列里面开始阻塞
7.当网卡的buffer响应队列轮到这个请求的时候，才会把这个http响应发送出去，阻塞结束

可以看到，主要的**耗时流程**是这样的：1.网卡buffer阻塞（自己被阻塞） -> 2.node server 单线程计算（自己被阻塞）-> 3.node server 单线程计算（阻塞别人） -> 4.网卡buffer阻塞（自己被阻塞）。其中1和4都是IO造成的阻塞，2和3都是CPU计算造成的阻塞。

需要注意的是，流程中的1和4并不会阻塞node server，并且node server由于是单线程不需要在各个进程线程来回切换，所以这就是node server高性能的两个原因。

## 进程级并发

从上面我们可以看到，对于IO密集型来说，主要时间都花在了1和4，此时node server没有被阻塞，能正常处理各种事情，所以他在IO密集型系统上来说性能很高；但是对于CPU密集型来说，主要时间都花在了2和3，此时node server是被阻塞的，不能处理其它请求，所以性能会很低。

庆幸的是，网络服务器大多都是IO密集型而非CPU密集型，所以node server作为网络服务器非常适用。但是CPU密集型操作仍然是一个需要解决的问题。

为了解决这个问题nodejs首先推出了**child_process子进程模块**和**cluster集群模块**。它们的区别是：child_process子进程模块不仅能够实现并发功能，还有其它进程相关的一些功能，比如运行系统命令等；而cluster集群模块则是基于child_process子进程模块的专注于并发的解决方案。

我们来实际编程体验一下。首先我们写一个**耗时的计算任务**：求出一个范围内的所有质数：

```js
// block-primes.js
const min = 2
const max = 1e7

function generatePrimes(start, range) {
  let primes = []
  let isPrime = true
  let end = start + range
  for (let i = start; i < end; i++) {
    for (let j = min; j < Math.sqrt(end); j++) {
      if (i !== j && i % j === 0) {
        isPrime = false
        break
      }
    }
    if (isPrime) {
      primes.push(i)
    }
    isPrime = true
  }
  return primes
}
const primes = generatePrimes(min, max)
console.log(primes.length)
```

我们使用linux的**原生time命令**查看这段代码的运行时间和CPU占用率：

```bash
time node block-primes.js
```

输出结果如下。可以看到，用户态运行时间为9.01s，内核态运行时间为0.02s，CPU占用率为100%，总运行时间为9.029秒。

```txt
664579
node block-primes.js  9.01s user 0.02s system 100% cpu 9.029 total
```

**再来看一下使用child_process的情况：**

```js
// child-process-main.js
const { fork } = require('child_process')
const worker = fork(__dirname + '/child-process-worker.js')
var numCPUs = require('os').cpus().length

// 接收工作进程计算结果
let max = 1e7
let min = 2
let start = 2
let primes = []

const range = Math.ceil((max - min) / numCPUs)

for (var i = 0; i < numCPUs; i++) {
  worker.send({ start: start, range: range })
  start += range
  worker.on('message', (msg) => {
    primes = primes.concat(msg.data)
    worker.kill()
  })
}

// child-process-worker.js
// 素数的计算
function generatePrimes(start, range) {
  let primes = []
  let isPrime = true
  let end = start + range
  for (let i = start; i < end; i++) {
    for (let j = 2; j < Math.sqrt(end); j++) {
      if (i !== j && i % j === 0) {
        isPrime = false
        break
      }
    }

    if (isPrime) {
      primes.push(i)
    }

    isPrime = true
  }
  return primes
}


// 监听子进程发送的信息
process.on('message', (msg) => {
  const {
    start,
    range
  } = msg
  console.log(msg)
  const data = generatePrimes(start, range)
  // 在工作进程中，这会发送消息给主进程
  process.send({
    data: data
  })
})

// 收到kill信息，进程退出
process.on('SIGHUP', function () {
  process.exit()
})
```

运行结果如下。可以看到，用户态运行时间为7.12s，内核态运行时间为0.06s，CPU占用率为100%，总运行时间为7.148秒。内核态运行时间比之前高了，因为要在内核态创建和切换多个进程。

```txt
(node:55928) MaxListenersExceededWarning: Possible EventEmitter memory leak detected. 11 message listeners added to [ChildProcess]. Use emitter.setMaxListeners() to increase limit
(Use `node --trace-warnings ...` to show where the warning was created)
{ start: 2, range: 833334 }
{ start: 833336, range: 833334 }
{ start: 1666670, range: 833334 }
{ start: 2500004, range: 833334 }
{ start: 3333338, range: 833334 }
{ start: 4166672, range: 833334 }
{ start: 5000006, range: 833334 }
{ start: 5833340, range: 833334 }
{ start: 6666674, range: 833334 }
{ start: 7500008, range: 833334 }
{ start: 8333342, range: 833334 }
{ start: 9166676, range: 833334 }
node child-process-main.js  7.12s user 0.06s system 100% cpu 7.148 total
```

**再来看一下使用cluster的情况：**

```js
// 计算 start, 至 start + range 之间的素数
function generatePrimes(start, range) {
  let primes = []
  let isPrime = true
  let end = start + range
  for (let i = start; i < end; i++) {
    for (let j = min; j < Math.sqrt(end); j++) {
      if (i !== j && i % j === 0) {
        isPrime = false
        break
      }
    }

    if (isPrime) {
      primes.push(i)
    }

    isPrime = true
  }
  return primes
}


/**
 * - 加载clustr模块
 * - 设定启动进程数为cpu个数
 */
var cluster = require('cluster')
var numCPUs = require('os').cpus().length

// 素数的计算
const min = 2
const max = 1e7 // = 10000000
let primes = []


if (cluster.isMaster) {
  const range = Math.ceil((max - min) / numCPUs)
  let start = min

  for (var i = 0; i < numCPUs; i++) {
    const worker = cluster.fork() // 启动子进程
    //  在主进程中，这会发送消息给特定的工作进程
    worker.send({
      start: start,
      range: range
    })

    start += range

    worker.on('message', (msg) => {
      primes = primes.concat(msg.data)
      worker.kill()
    })
  }
  // 当任何一个工作进程关闭的时候，cluster 模块都将会触发 'exit' 事件
  cluster.on('exit', function (worker, code, signal) {
    console.log('worker ' + worker.process.pid + ' died')
  })
} else {
  // 监听子进程发送的信息
  process.on('message', (msg) => {
    console.log(msg)
    const {
      start,
      range
    } = msg
    const data = generatePrimes(start, range)
    // 在工作进程中，这会发送消息给主进程
    process.send({
      data: data
    })
  })
}
```

运行结果如下。可以看到，用户态运行时间为7.43s，内核态运行时间为0.25s，CPU占用率为483%，总运行时间为1.588秒。内存占用率高于100%表示有多个CPU在同时工作；由于是在多个CPU上面工作，所以总运行时间1.588秒少于用户态时间7.43秒。

```txt
{ start: 2, range: 833334 }
{ start: 3333338, range: 833334 }
{ start: 2500004, range: 833334 }
{ start: 833336, range: 833334 }
{ start: 1666670, range: 833334 }
{ start: 5000006, range: 833334 }
{ start: 4166672, range: 833334 }
{ start: 5833340, range: 833334 }
{ start: 6666674, range: 833334 }
{ start: 7500008, range: 833334 }
{ start: 8333342, range: 833334 }
{ start: 9166676, range: 833334 }
worker 56660 died
worker 56661 died
worker 56664 died
worker 56663 died
worker 56665 died
worker 56662 died
worker 56666 died
worker 56667 died
worker 56668 died
worker 56669 died
worker 56670 died
worker 56671 died
node cluster-primes.js  7.43s user 0.25s system 483% cpu 1.588 total
```

可以看到当使用cluster的时候，性能提升到了9.029/1.588=568.6%

## 线程级并发

使用worker_threads工作线程模块的代码如下：

```js
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads')

function generatePrimes(start, range) {
  let primes = []
  let isPrime = true
  let end = start + range
  for (let i = start; i < end; i++) {
    for (let j = 2; j < Math.sqrt(end); j++) {
      if (i !== j && i%j === 0) {
        isPrime = false
        break
      }
    }
    if (isPrime) {
      primes.push(i)
    }
    isPrime = true
  }
  return primes
}


if (isMainThread) {
  const max = 1e7
  const min = 2
  let primes = []

  const threadCount = +process.argv[2] || 2
  const threads = new Set()
  console.log(`Running with ${threadCount} threads...`)
  const range = Math.ceil((max - min) / threadCount)
  let start = min

  for (let i = 0; i < threadCount - 1; i++) {
    const myStart = start
    threads.add(new Worker(__filename, { workerData: { start: myStart, range }}))
    start += range
  }

  threads.add(new Worker(__filename, { workerData: { start, range: range + ((max - min + 1) % threadCount)}}))

  for (let worker of threads) {
    worker.on('error', (err) => { throw err })
    worker.on('exit', () => {
      threads.delete(worker)
      console.log(`Thread exiting, ${threads.size} running...`)
      if (threads.size === 0) {
        // console.log(primes.join('\n'))
      }
    })

    worker.on('message', (msg) => {
      primes = primes.concat(msg)
    })
  }
} else {
  const data = generatePrimes(workerData.start, workerData.range)
  parentPort.postMessage(data)
}
```

运行结果如下。可以看到，用户态运行时间为7.73s，内核态运行时间为0.10s，CPU占用率为510%，总运行时间为1.533秒。内核态运行时间远小于集群的方案，因为worker_threads创建的是线程而不是进程，更加轻量；并且总运行时间要稍微小于集群的方案。

```txt
Running with 12 threads...
Thread exiting, 11 running...
Thread exiting, 10 running...
Thread exiting, 9 running...
Thread exiting, 8 running...
Thread exiting, 7 running...
Thread exiting, 6 running...
Thread exiting, 5 running...
Thread exiting, 4 running...
Thread exiting, 3 running...
Thread exiting, 2 running...
Thread exiting, 1 running...
Thread exiting, 0 running...
node worker-threads.js 12  7.73s user 0.10s system 510% cpu 1.533 total
```

## 网络服务器的并发方案

首先我们**使用express搭建一个简单的服务器，主页先进行一些耗时计算**，然后返回Hello world，代码如下：

```js
const express = require('express')
const sleep = require('sleep')
const crypto = require('crypto')

const port = +process.argv[2] || 1234
const app = express()

app.listen(port, () => {
  let message = 'server start'
  console.log(message)
})
app.get('/', (req, res) => {
  //CPU intensive task
  const randSleep = Math.round(10000 + (Math.random() * 10000))
  sleep.usleep(randSleep)
  const arrSize = Math.round(5000 + (Math.random() * 4000))
  crypto.createHmac('sha256', 'secret').update(new Array(arrSize).fill('a').join('.')).digest('hex')
  res.send('Hello world')
})
```

运行服务器之后使用 loadtest 进行压测：

```bash
loadtest -c 100 -t 10 http://localhost:1234
```

结果如下。可以看到，QPS 为 55，平均延迟为 1656.9 ms。

```txt
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO Target URL:          http://localhost:1234
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO Max time (s):        10
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO Concurrency level:   100
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO Agent:               none
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO Completed requests:  547
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO Total errors:        0
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO Total time:          10.003378301000001 s
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO Requests per second: 55
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO Mean latency:        1656.9 ms
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO Percentage of the requests served within a certain time
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO   50%      1809 ms
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO   90%      1844 ms
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO   95%      1852 ms
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO   99%      1871 ms
[Thu Apr 08 2021 17:18:20 GMT+0800 (中国标准时间)] INFO  100%      1883 ms (longest request)
```

然后我们使用集群cluster的方案来并发处理服务器请求，代码如下：

```js
const cluster = require('cluster')
const express = require('express')
const sleep = require('sleep')
const crypto = require('crypto')

const port = +process.argv[2] || 1234
const numCPUs = require('os').cpus().length
const app = express()

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);
  // Fork workers.
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork()
  }
  cluster.on('exit', (worker, code, signal) => {
    console.log(`worker ${worker.process.pid} died`)
  })
} else {
  // Workers can share any TCP connection
  app.listen(port, () => {
    let message = `Worker : ${process.pid}`
    console.log(message)
  })
  app.get('/', (req, res) => {
    //CPU intensive task
    const randSleep = Math.round(10000 + (Math.random() * 10000))
    sleep.usleep(randSleep)
    const arrSize = Math.round(5000 + (Math.random() * 4000))
    crypto.createHmac('sha256', 'secret').update(new Array(arrSize).fill('a').join('.')).digest('hex')
    res.send(`Worker : ${process.pid}`)
  })
  console.log(`Worker ${process.pid} started`);
}
```

运行服务器之后使用 loadtest 进行压测，结果如下。可以看到，QPS 为 674，平均延迟为 146.9 ms。性能提升了10倍多。

```txt
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO Target URL:          http://localhost:1234
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO Max time (s):        10
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO Concurrency level:   100
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO Agent:               none
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO Completed requests:  6739
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO Total errors:        0
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO Total time:          10.001519736 s
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO Requests per second: 674
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO Mean latency:        146.9 ms
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO Percentage of the requests served within a certain time
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO   50%      146 ms
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO   90%      157 ms
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO   95%      160 ms
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO   99%      165 ms
[Thu Apr 08 2021 17:24:42 GMT+0800 (中国标准时间)] INFO  100%      180 ms (longest request)
```

## 其它想法

1.happypack和thread-loader的原理和优化（使用worker_threads）
2.线程安全和锁（nodejs怎么解决这类问题）
3.其它运用场景：1.生成locale多语言文件。2.ssr的多线程渲染。
4.worker_threads的通信
