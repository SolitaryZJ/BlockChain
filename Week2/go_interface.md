# 1.17 interface 接口

在 Go 中接口是一种抽象类型，是一组方法的集合，里面只声明方法，而没有任何数据成员。

而在 Go 中实现一个接口也不需要显式的声明，只需要其他类型实现了接口中所有的方法，就是实现了这个接口。

定义一个接口：

```go
type <interface_name> interface {
    <method_name>(<method_params>) [<return_type>...]
    ...
}
```

代码示例：

```go
package main

import "fmt"

// PaymentMethod 接口定义了支付方法的基本操作
type PayMethod interface {
    Account
    Pay(amount int) bool
}

type Account interface {
    GetBalance() int
}

// CreditCard 结构体实现 PaymentMethod 接口
type CreditCard struct {
    balance int
    limit   int
}

func (c *CreditCard) Pay(amount int) bool {
    if c.balance+amount <= c.limit {
        c.balance += amount
        fmt.Printf("信用卡支付成功: %d\n", amount)
        return true
    }
    fmt.Println("信用卡支付失败: 超出额度")
    return false
}

func (c *CreditCard) GetBalance() int {
    return c.balance
}

// DebitCard 结构体实现 PaymentMethod 接口
type DebitCard struct {
    balance int
}

func (d *DebitCard) Pay(amount int) bool {
    if d.balance >= amount {
        d.balance -= amount
        fmt.Printf("借记卡支付成功: %d\n", amount)
        return true
    }
    fmt.Println("借记卡支付失败: 余额不足")
    return false
}

func (d *DebitCard) GetBalance() int {
    return d.balance
}

// 使用 PaymentMethod 接口的函数
func purchaseItem(p PayMethod, price int) {
    if p.Pay(price) {
        fmt.Printf("购买成功，剩余余额: %d\n", p.GetBalance())
    } else {
        fmt.Println("购买失败")
    }
}

func main() {
    creditCard := &CreditCard{balance: 0, limit: 1000}
    debitCard := &DebitCard{balance: 500}

    fmt.Println("使用信用卡购买:")
    purchaseItem(creditCard, 800)

    fmt.Println("\n使用借记卡购买:")
    purchaseItem(debitCard, 300)

    fmt.Println("\n再次使用借记卡购买:")
    purchaseItem(debitCard, 300)
}
```

使用过程中需要注意以下几点：

1. Go 中接口声明的方法并不要求需要全部公开。
2. 直接用接口类型作为变量时，如果方法都是值接收者，可以用值或指针。如果任何方法是指针接收者，则必须用指针。

```go
package main

import "fmt"

type Account interface {
    getBalance() int
}

type CreditCard struct {
    balance int
    limit   int
}

func (c *CreditCard) getBalance() int {
    return c.balance
}

func main() {
    c := CreditCard{balance: 100, limit: 1000}
    var a Account = &c
    fmt.Println(a.getBalance())
}
```

1. 接口可以嵌套。
2. 接口中声明的方法，参数可以没有名称。
3. 如果函数参数使用 interface{}可以接受任何类型的实参。同样，可以接收任何类型的值也可以赋值给 interface{}类型的变量

代码示例：

```go
package main

import "fmt"

type PayMethod interface {
    Pay(int)
}

type CreditCard struct {
    balance int
    limit   int
}

func (c *CreditCard) Pay(amout int) {
    if c.balance < amout {
        fmt.Println("余额不足")
        return
    }
    c.balance -= amout
}

func anyParam(param interface{}) {
    fmt.Println("param: ", param)
}

func main() {
    c := CreditCard{balance: 100, limit: 1000}
    c.Pay(200)
    var a PayMethod = &c
    fmt.Println("a.Pay: ", a)

    var b interface{} = &c
    fmt.Println("b: ", b)

    anyParam(c)
    anyParam(1)
    anyParam("123")
    anyParam(a)
}
```