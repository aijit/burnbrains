## Go 深耕：goroutine 原理及使用

Go 语言内核提供了 goroutine 和 channel 对象，用以支持并发特性。
其中，goroutine 是一种由 Go runtime 进行调度的、轻量级、用户态线程； channel 用于建立 goroutine 间的通信渠道。
这两个内建对象，使得 Golang 多道程序设计，相对其他典型开发场景（如 Linux 平台的多进程、Windows 平台的多线程）都更为简洁、易用。
本篇只讨论 goroutine 相关内容。

以下是由 [A tour of Go](https://go.dev/tour/concurrency/1) 提供的一段例程，`go` 关键字 + `函数名`，用一行代码创建了一个子 goroutine 输出 100 次 `"world"`，在主 goroutine 中输出一次 `"hello"`。
```golang
package main

import (
	"fmt"
	"time"
)

func say(s string) {
	for i := 0; i < 5; i++ {
		time.Sleep(100 * time.Millisecond)
		fmt.Println(s)
	}
}

func main() {
	go say("world")
	say("hello")
}

```


### 线程模型

没有找到一个恰当的翻译，不引起与线程(thread)、协程(coroutine)概念上的混淆。
全文仅使用 goroutine 的名字，也是旨在强调 goroutine 与两者的截然不同。

[《Effective Go》](https://go.dev/doc/effective_go#goroutines)
中介绍了 goroutine 使用的并发模型：

> 不同 goroutine 运行在相同的地址空间，开销非常轻量，初始栈也很小（运行过程中按需分配堆内存）；
>
> goroutine 多路复用地 (multiplexed) 运行在多个线程上，当其中一个 goroutine 阻塞时，不会引起其他 goroutine 的阻塞。

在传统编程语言中，更常见的线程模型是类似于
[pthread](https://www.gnu.org/software/hurd/libpthread.html)
的 1:1 内核态多线程并行 (parallel)，
或者像 python、lua 的 coroutine 那样的 N:1 用户态多协程并发 (concurrency)。与 goroutine 线程模型相比，有各自明显的特点，以及不同场景下的优劣表现。

* 内核态线程并行

    线程由操作系统内核创建和调度；

    一个线程的阻塞，不会引起其他线程的阻塞；

    线程的切换要陷入 OS 内核，开销较大；

    多线程的并行，发挥了多核计算机的优势，但线程数量过多时调度开销明显。

* 用户态协程并发

    协程在用户空间创建、调度；

    多个协程绑定到一个内核线程上，减少了切换开销；

    若一个协程阻塞前没有主动让出 cpu，其他协程也会阻塞；

    多任务交替顺序执行，不能发挥计算机多核优势。

* goroutine 混合线程模型

    多个 goroutine 对应于多个操作系统线程；

    线程仍在内核态创建和调度；

    goroutine 数量一般多于对应的线程数量，在用户态创建；

    goroutine 不须主动让出 cpu 时间（由 go runtime 在用户态调度）；
    
    一个 goroutine 阻塞也不会影响其他 goroutine 的运行；

    兼顾（折中）地利用 cpu 多核并行提速（优于 coroutine），同时降低总的调度开销（优于 thread）。


### GMP 模型

[《Scalable Go Scheduler Design Doc》](https://docs.google.com/document/d/1TTj4T2JO42uD5ID9e89oa0sLKhJYD0Y_kqxDv3I3XMw)
介绍了 G(Goroutine)、M(Machines)、P(Processors) 对象的意义：

> G 表示 goroutine，程序初始化时创建的第一个 G 对象 g0 负责全局 goroutine 的调度。
>
> M 代表一个操作系统 thread，会随着线程池的伸缩，创建和销毁；M 只与 P 关联（在早期的实现中，M 关联了若干 G，直到引入 P 接管 G；go 程序仍可通过 
> [runtime.LockOSThread](https://pkg.go.dev/runtime#LockOSThread)
> 方法，强行将 goroutine 绑定到一个 thread 上）。
>
> P 用于管理不同运行状态的 G (替代了早期版本中 M 与 G 的关联)，P 的数量可以由
[GOMAXPROCS](https://docs.google.com/document/d/1At2Ls5_fhJQ59kDK2DFVhFu3g5mATSXqqV5QrxinasI)
环境变量指定，或者 runtime 根据物理 CPU 数量决定。


[《Go Preemptive Scheduler Design Doc》](https://docs.google.com/document/d/1ETuA2IOmnaQ4j81AtTGT40Y4_Jr6_IDASEKg0t0dBR8)
介绍了 goroutine 的抢占式调度设计。
