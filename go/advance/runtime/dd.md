---
description: 收藏阶段
---

# 一拳搞定 Golang \| Goroutine 调度原理

## **并发编程模型**

**并发编程模型**

并发和多线程编程在难度上声名远扬。 是由于诸如 pthread 等复杂设计，部分原因是过分强调低级别细节，如互斥锁，条件变量和内存屏障。更高级别的接口可以实现更简单的代码，即使仍然存在互斥。

通信顺序进程（CSP）是为并发提供高级语言支持的最成功模式之一。 Occam 和 Erlang 是源于 CSP 的两种众所周知的语言。 Go 的并发基元来自家族树的不同部分，其主要贡献是通道作为一类对象的强大概念。 对几种早期语言的经验表明，CSP模型非常适合程序语言框架。

**CSP**：通信顺序进程，是 Go 语言采用的并发同步模型，是一种形式语言，用来描述并发系统间进行交互的模式。

**Actor 模型**：参与者模型采用了 _everything is an actor_ 哲学。所有参与者都是独立的运行单元，参与者可以修改自己的私有状态，参与者之间只能通过发送消息通信（避免任何锁的使用）。

{% embed url="https://iswade.github.io/articles/go\_concurrency/" %}

## 线程模型

### G-P-M 模型

* G: 表示 Goroutine，每个 Goroutine 对应一个 G 结构体，G 存储 Goroutine 的运行堆栈、状态以及任务函数，可重用。G 并非执行体，每个 G 需要绑定到 P 才能被调度执行。
* P: Processor，表示逻辑处理器， 对 G 来说，P 相当于 CPU 核，G 只有绑定到 P\(在 P 的 local runq 中\)才能被调度。对 M 来说，P 提供了相关的执行环境\(Context\)，如内存分配状态\(mcache\)，任务队列\(G\)等，P 的数量决定了系统内最大可并行的 G 的数量（前提：物理 CPU 核数 &gt;= P 的数量），P 的数量由用户设置的 GOMAXPROCS 决定，但是不论 GOMAXPROCS 设置为多大，P 的数量最大为 256。
* M: Machine，OS 线程抽象，代表着真正执行计算的资源，在绑定有效的 P 后，进入 schedule 循环；而 schedule 循环的机制大致是从 Global 队列、P 的 Local 队列以及 wait 队列中获取 G，切换到 G 的执行栈上并执行 G 的函数，调用 goexit 做清理工作并回到 M，如此反复。M 并不保留 G 状态，这是 G 可以跨 M 调度的基础，M 的数量是不定的，由 Go Runtime 调整，为了防止创建过多 OS 线程导致系统调度不过来，目前默认最大限制为 10000 个。
* 每个 P 维护一个 G 的本地队列；
* 当一个 G 被创建出来，或者变为可执行状态时，就把他放到 P 的本地可执行队列中，如果满了则放入Global；
* 当一个 G 在 M 里执行结束后，P 会从队列中把该 G 取出；如果此时 P 的队列为空，即没有其他 G 可以执行， M 就随机选择另外一个 P，从其可执行的 G 队列中取走一半。

{% embed url="https://studygolang.com/articles/29227?fr=sidebar" %}

{% embed url="https://studygolang.com/articles/13875" %}

{% embed url="https://colobu.com/2019/04/28/gopher-2019-concurrent-in-action/" %}

{% embed url="https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/" %}

\*\*\*\*

