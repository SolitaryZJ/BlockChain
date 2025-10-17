# 1.19 并发 - select 关键字

select 语义是和 channel 绑定在一起使用的，select 可以实现从多个 channel 收发数据，可以使得一个 goroutine 就可以处理多个 channel 的通信。

语法上和 switch 类似，有 case 分支和 default 分支，只不过 select 的每个 case 后面跟的是 channel 的收发操作。

定义方式：

```go
select {
case channel_name <- varaible_name_or_value: // send data to channeldo sthcase var_name = <-ch2 : // receive data from channeldo sthcase data, ok := <-ch3:
    do sth
case value_name, ok_flag := <- channel_name:
    do sth
default:
    do sth
}
```

语法上和<u>switch</u>的一些区别：

- select 关键字和后面的 `{` 之间，不能有表达式或者语句。
- 每个 case 关键字后面跟的必须是 channel 的发送或者接收操作
- 允许多个 case 分支使用相同的 channel，case 分支后的语句甚至可以重复

代码示例：

```go
package main

import (
    "fmt"
)

func main() {
    ch1 := make(chan int, 10)
    ch2 := make(chan int, 10)
    ch3 := make(chan int, 10)
    go func() {
        for i := 0; i < 10; i++ {
            ch1 <- i
            ch2 <- i
            ch3 <- i
        }
    }()
    for i := 0; i < 10; i++ {
        select {
        case x := <-ch1:
            fmt.Printf("receive %d from channel 1\n", x)
        case y := <-ch2:
            fmt.Printf("receive %d from channel 2\n", y)
        case z := <-ch3:
            fmt.Printf("receive %d from channel 3\n", z)
        }
    }
}
```

在执行 select 语句的时候，如果当下那个时间点没有一个 case 满足条件，就会走 default 分支。

至多只能有一个 default 分支。

如果没有 default 分支，select 语句就会阻塞，直到某一个 case 满足条件。

如果 select 里任何 case 和 default 分支都没有，就会一直阻塞。

如果多个 case 同时满足，select 会随机选一个 case 执行。