# 1.11 变量的作用域

# 1.11.1 局部变量

在函数内声明的变量，作用域范围只在函数体内。

同理，函数的参数和返回值也是局部变量。

代码示例：

```go
func localVariable(parameter int) (res int) {
    var decVar int 
}
```

上面示例中 `parameter`、`res`、`decVar` 都是局部变量，仅在当前函数中有效。

比较特殊的是 if 语句、for 语句、switch 语句、select 语句、匿名代码块中声明的变量，它们一般比在函数内声明的变量作用域范围更小，仅在小的代码块内有效。

代码示例：

```go
func main() {
    var a int
    if b := 1; b == 0 {
        fmt.Println("b == 0")
    } else {
        c := 2
        fmt.Println("declare c = ", c)
        fmt.Println("b == 1")
    }
    // fmt.Println(b)
    // fmt.Println(c)

    switch d := 3; d {
    case 1:
        e := 4
        fmt.Println("declare e = ", e)
        fmt.Println("d == 1")
    case 3:
        f := 4
        fmt.Println("declare f = ", f)
        fmt.Println("d == 3")
    }
    // fmt.Println(e)
    // fmt.Println(f)

    for i := 0; i < 1; i++ {
        forA := 1
        fmt.Println("forA = ", forA)
    }
    // fmt.Println("forA = ", forA)

    select {
    case <-time.After(time.Second):
        selectA := 1
        fmt.Println("selectA = ", selectA)
    }
    // fmt.Println("selectA = ", selectA)

    // 匿名代码块
    {
        blockA := 1
        fmt.Println("blockA = ", blockA)
    }
    // fmt.Println("blockA = ", blockA)

    fmt.Println("a = ", a)
}
```

# 1.11.2 全局变量

全局变量在函数外声明，全局变量作用域可以是当前整个包甚至外部包（公开的全局变量）使用。

有一种比较特殊的情况，即当全局变量和局部变量重名时，函数内会使用局部变量，超出局部变量作用域之后，才会重新使用全局变量。

代码示例：

```go
var a int

func main() {
    {
        fmt.Println("global variable, a = ", a)
        a = 3
        fmt.Println("global variable, a = ", a)

        a := 10
        fmt.Println("local variable, a = ", a)
        a--
        fmt.Println("local variable, a = ", a)
    }
    fmt.Println("global variable, a = ", a)
}
```

这种优先使用作用域更小的变量的规则，同样适用于局部变量：

```go
func main() {
    var b int = 4
    fmt.Println("local variable, b = ", b)
    if b := 3; b == 3 {
        fmt.Println("if statement, b = ", b)
        b--
        fmt.Println("if statement, b = ", b)
    }
    fmt.Println("local variable, b = ", b)
}
```

实际代码使用中，经常会有各种方法返回 error，error 会赋值给 err 变量。