# 1.18 并发-goroutine 与 channel

go 支持并发的方式，就是通过 goroutine 和 channel 提供的简洁且高效的方式实现的。

# 1.18.1 goroutine

goroutine 是轻量线程，创建一个 goroutine 所需的资源开销很小，所以可以创建非常多的 goroutine 来并发工作。

它们是由 Go 运行时调度的。调度过程就是 Go 运行时把 goroutine 任务分配给 CPU 执行的过程。

但是 goroutine 不是通常理解的线程，线程是操作系统调度的。

在 Go 中，想让某个任务并发或者异步执行，只需把任务封装为一个函数或闭包，交给 goroutine 执行即可。

声明方式 1，把方法或函数交给 goroutine 执行：

```go
go <method_name>(<method_params>...)
```

声明方式 2，把闭包交给 goroutine 执行：

```go
go func(<method_params>...){
    <statement_or_expression>
    ...
}(<params>...)
```

代码示例：

```go
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
    go func() {
        fmt.Println("run goroutine in closure")
    }()
    go func(string) {
    }("gorouine: closure params")
    go say("in goroutine: world")
    say("hello")
}
```

go 中并发同样存在线程安全问题，因为 Go 也是使用共享内存让多个 goroutine 之间通信。并且大部分时候为了性能，所以 go 的大多数标准库的数据结构默认是非线程安全的。

```go
package main

import (
    "fmt"
    "sync"
    "time"
)

// 线程安全的计数器
type SafeCounter struct {
    mu    sync.Mutex
    count int
}

// 增加计数
func (c *SafeCounter) Increment() {
    c.mu.Lock()
    defer c.mu.Unlock()
    c.count++
}

// 获取当前计数
func (c *SafeCounter) GetCount() int {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.count
}

type UnsafeCounter struct {
    count int
}

// 增加计数
func (c *UnsafeCounter) Increment() {
    c.count += 1
}

// 获取当前计数
func (c *UnsafeCounter) GetCount() int {
    return c.count
}

func main() {
    counter := UnsafeCounter{}

    // 启动100个goroutine同时增加计数
    for i := 0; i < 1000; i++ {
        go func() {
            for j := 0; j < 100; j++ {
                counter.Increment()
            }
        }()
    }

    // 等待一段时间确保所有goroutine完成
    time.Sleep(time.Second)

    // 输出最终计数
    fmt.Printf("Final count: %d\n", counter.GetCount())
}
```

# 1.18.2 channel

channel 是 Go 中定义的一种类型，专门用来在多个 goroutine 之间通信的线程安全的数据结构。

可以在一个 goroutine 中向一个 channel 中发送数据，从另外一个 goroutine 中接收数据。

channel 类似队列，满足先进先出原则。

定义方式：

```go
// 仅声明
var <channel_name> chan <type_name>

// 初始化
<channel_name> := make(chan <type_name>)

// 初始化有缓冲的channel
<channel_name> := make(chan <type_name>, 3)
```

channel 的三种操作：发送数据，接收数据，以及关闭通道。

声明方式：

```go
// 发送数据
channel_name <- variable_name_or_value

// 接收数据
value_name, ok_flag := <- channel_name
value_name := <- channel_name

// 关闭channel
close(channel_name)
```

channel 还有两个变种，可以把 channel 作为参数传递时，限制 channel 在函数或方法中能够执行的操作。

声明方式：

```go
//仅发送数据
func <method_name>(<channel_name> chan <- <type>)

//仅接收数据
func <method_name>(<channel_name> <-chan <type>)
```

代码示例：

```go
package main

import (
    "fmt"
    "time"
)

// 只接收channel的函数
func receiveOnly(ch <-chan int) {
    for v := range ch {
        fmt.Printf("接收到: %d\n", v)
    }
}

// 只发送channel的函数
func sendOnly(ch chan<- int) {
    for i := 0; i < 5; i++ {
        ch <- i
        fmt.Printf("发送: %d\n", i)
    }
    close(ch)
}

func main() {
    // 创建一个带缓冲的channel
    ch := make(chan int, 3)

    // 启动发送goroutine
    go sendOnly(ch)

    // 启动接收goroutine
    go receiveOnly(ch)

    // 使用select进行多路复用
    timeout := time.After(2 * time.Second)
    for {
        select {
        case v, ok := <-ch:
            if !ok {
                fmt.Println("Channel已关闭")
                return
            }
            fmt.Printf("主goroutine接收到: %d\n", v)
        case <-timeout:
            fmt.Println("操作超时")
            return
        default:
            fmt.Println("没有数据，等待中...")
            time.Sleep(500 * time.Millisecond)
        }
    }
}
```

# 1.18.3 锁与 channel

在 Go 中，当需要 goroutine 之间协作地方，更常见的方式是使用 channel，而不是 sync 包中的 Mutex 或 RWMutex 的互斥锁。但其实它们各有侧重。

大部分时候，流程是根据数据驱动的，channel 会被使用得更频繁。

channel 擅长的是**数据流动的场景**：

1. 传递数据的所有权，即把某个数据发送给其他协程。
2. 分发任务，每个任务都是一个数据。
3. 交流异步结果，结果是一个数据。

而锁使用的场景更偏向**同一时间只给一个协程访问数据的权限**：

1. 访问缓存
2. 管理状态