# 数组和切片(slice)的区别；
  - 数组是固定长度的，切片是可变长度的；
  - 切片实际上是一个比较特殊的指针类型，声明了一个切片就相当于声明了一个指针。指针指向切片结构体（数组指针、长度、容量）；）
  - 可以使用append()向切片追加元素；
  - 切片触发扩容前，切片使用的都是同一个数据
  - 切片触发扩容后，会创建新的数组，并复制原切片元素；
# map的底层实现
  - 底层数据结构是一个hashtable，对key进行hash，而value映射到bucket中。每个bucket最多存储8个元素
  当bucket满了则创建over bucket存储
  - 新增元素时，会先通过负载因子判断总bucket是否满了，满了则进行扩容。之后计算key的hash值，
  通过该hash来定位bucket，并将数据存储该bucket中空闲位置。如果bucket已满则将数据存储over bucket中，over bucket结构与bucket一样，满了则创建自己对应的over bucket，以此类推来解决hash冲突问题
  - 获取元素时，计算key的hash值，使用hash低位获取bucket位置，hash高8位tophash在bucket快速比较，如果匹配则比较完整key，如果未找到则遍历over bucket，找到返回value，未找到返回零值。
  - 删除元素时，类似查找过程定位到元素，将tophash值置为空标记，如果删除导致bucket变空，则触发内存整理，之后count-1
  - 扩容条件：1.负载因子超过6.5（元素个数/总bucket个数>6.5）；2. over bucket过多（B < 15且溢出桶≥2^B 或 B >= 15且溢出桶≥2^15）
  - 扩容类型：
    - 等量扩容：
      - 触发条件: 溢出桶过多但负载因子不高
      - 目的: 整理哈希表，减少溢出桶
      - 结果: 桶数量不变，重新分布元素
    - 增量扩容：
      - 触发条件: 负载因子超过阈值(6.5)
      - 目的: 增加桶数量，降低负载
      - 结果: 桶数量翻倍(2^B → 2^(B+1))
    - 渐进式扩容原理：
```
初始状态:
hmap.buckets    → [桶0][桶1][桶2][桶3]  (2^B=4个桶)
hmap.oldbuckets → nil

开始扩容:
hmap.buckets    → [新桶0]...[新桶7]     (2^(B+1)=8个桶)
hmap.oldbuckets → [旧桶0][旧桶1][旧桶2][旧桶3]

搬迁过程:
每次操作时，检查操作的桶是否已搬迁
如未搬迁，则搬迁该桶及其溢出桶
搬迁完成后，oldbuckets置为nil
```

# channel的底层实现
  - 专门用来在多个goroutine之间进行通信的线程安全通道。类似队列，先进先出
  - 使用场景：
    - 传递参数的所有权，即把参数传递给其他goroutine
    - 分发任务，每个任务都是一个数据
    - 交流异步结果，结果是一个数据
  - 数据结构：
      - 用来保存goroutine之间传递数据的循环链表。=====> buf。
      - 用来记录此循环链表当前发送或接收数据的下标值。=====> sendx和recvx。
      - 用于保存向该chan发送和从改chan接收数据的goroutine的队列。=====> sendq 和 recvq
      - 保证channel写入和读取数据时线程安全的锁。 =====> lock
  - 原理：
    - 假设G1为生产者，G2消费者
    - 初始hchan结构体的buf为空，sendx和recvx都为0
    - G1发送数据时，会将buf加锁。然后将数据copy到buf（buf[sendx]）中，然后更新sendx的值为(sendx+1)%cap(buf)，最后释放buf
    - G2接收数据时，会将buf加锁。然后将数据copy到对应任务的变量地址中，然后更新recvx++，最后释放buf


# select case的用法，有哪些运用场景
  - select 语义是和channel绑定在一起使用的，select可以实现从多个channel收发数据，可以使一个goruntine就可以处理多个channel通信
  - 语法上和 switch 类似，有 case 分支和 default 分支，只不过 select 的每个 case 后面跟的是 channel 的收发操作
  - 1.多通道读取 —— 谁先就绪就处理谁

```go
select {
case v := <-ch1:
    fmt.Println("ch1:", v)
case v := <-ch2:
    fmt.Println("ch2:", v)
}
```

  - 2.非阻塞试探 —— default 兜底

```go
select {
case v := <-ch:
    use(v)
default:
    // 通道空，立即走这里，不阻塞
}
```

  - 3.超时控制 —— time.After 一支路

```go
select {
case res := <-worker:
    fmt.Println("result", res)
case <-time.After(3 * time.Second):
    fmt.Println("timeout")
}
```
  - 4.退出/取消信号 —— done channel 模式

```go
for {
    select {
    case job := <-jobs:
        process(job)
    case <-done:          // 主协程关闭 done，通知本协程立即退出
        return
    }
}
```
  - 5.非阻塞发送 —— 防止满通道卡死

```go
select {
case ch <- value:        // 能塞就塞
default:                 // 缓冲区满，立即丢弃或落盘
    log.Println("drop")
}
```
  - 6.随机公平 —— 主动避免饥饿

```go
for i := 0; i < 10; i++ {
    select {
    case <-ch1: fmt.Println("ch1")
    case <-ch2: fmt.Println("ch2")
    }
}
```

  - 7.空 select —— 永久阻塞（极少用）

```go
select {} // 当前 goroutine 永远挂起，常用于 main 占位或调试
```


  - 8.常见组合模式
    - for + select = 事件循环（服务器 goroutine 标配）
    - ctx.Done() + case = 标准取消信号
    - tick + time.After = 定时任务与超时并存

# GMP实现底层原理
  - 作用：将海量的 G（goroutine）以低成本、高效率的方式调度到 M（内核线程）上执行，通过 P（调度器上下文）协调，实现 M 和 G 之间的解耦。
  - 核心目标：通过复用操作系统线程M的执行时间，将其动态分配给轻量级的goroutine任务，避免了线程创建、销毁和上下文切换
  - 调度流程：
```text
1. 启动阶段
   - 启动时初始化多个 P，每个 P 尝试找 M 来绑定运行
   - M 必须绑定一个 P 才能执行 G，单独的 M 无法调度 G
2. G的创建
    - 当用户调用 go func() 时，创建 G（Goroutine）
    - G 被加入当前 P 的本地队列或全局队列（满时）
3. M 调用 P 执行 G
    - M 绑定一个 P，从 P 的队列中取出 G 执行
    - G 执行完成或发生阻塞（例如 syscall），M 会进行下一轮调度
4. G 被挂起或阻塞（GMP调度模型是如何优雅地应对G阻塞）
    - 若 G 调用 syscall 等阻塞操作：
    - M 可能阻塞，P 会被解绑
    - 调度器会创建新 M 来继续执行剩余的 G
5. G 的抢占
    - 为了防止 G 长时间执行，Go 1.14 起引入 异步抢占，防止“长任务”阻塞调度
    - 运行时间长的 G 会被强制抢占，挂起后重新入队
6. 调度策略：Work Stealing
   - 如果一个 P 的 G 队列空了，会尝试从其他 P 偷一半 G（work stealing）
   - 避免某些 P 长期空闲，提升 CPU 利用率
```
