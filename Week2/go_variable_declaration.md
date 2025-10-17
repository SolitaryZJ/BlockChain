# 1.3 定义变量

在 Go 中，变量主要分为两类：全局变量、局部变量。

# 1.3.1 全局变量

在函数外定义的变量就是全局变量，作用域为的整个程序的声明周期，定义方式如下：

方式 1，完整的声明：

```go
var <name> <type> = <value>
```

方式 2，仅声明，但未赋值，为类型默认零值：

```go
// 仅声明
var <name> <type>
```

方式 3， 不声明类型，但 Go 会根据表达式或字面量自动推导此变量的类型：

```go
// 某些类型可以直接推导出来，不需要声明
var <name> = <value>
```

方式 4，对全局变量分组声明：

```go
// 声明多个时，可以用小括号包裹，此方式不限制声明次数
var (
  <name1> <type1> = <value1>
  <name2> <type2>
  <name3> = <value3>
)
```

代码示例：

```go
var s1 string = "Hello"
var zero int
var b1 = true

var (
    i int = 123
    b2 bool
    s2 = "test"
)

var (
    group = 2
)
```

注：**全局变量允许声明后不使用**。

# 1.3.2 局部变量

在函数内或方法内定义的变量叫做局部变量。定义方式如下：

方式 1，与全局变量的声明方式完全一致：

```go
var <name> <type> = <value>
```

方式 2，也是与全局变量声明方式完全相同，仅声明，为类型默认零值：

```go
var <name> <type>
```

方式 3，无需关键字 var，也无需声明类型，Go 通过字面量或表达式推导此变量类型：

```go
<name> := <value>
```

方式 4，这种方式是全局变量没有的，可以直接在返回值中声明，相当于在方法一开始就声明了这些变量：

```go
func method() (<name1> <type1>, <name2> <type2>) {
    return
}

func method() (<name1> <type1>, <name2> <type2>) {
    return <value1>, <value2>
}
```

代码示例：

```go
func method1() {
    // 方式1，类型推导，用得最多
    a := 1
    // 方式2，完整的变量声明写法
    var b int = 2
    // 方式3，仅声明变量，但是不赋值，
    var c int
    fmt.Println(a, b, c)
}

// 方式4，直接在返回值中声明
func method2() (a int, b string) {
    // 这种方式必须声明return关键字
    // 并且同样不需要使用，并且也不用必须给这种变量赋值
    return 1, "test"
}

func method3() (a int, b string) {
    a = 1
    b = "test"
    return
}

func method4() (a int, b string) {
    return
}
```

# 1.3.3 多变量定义

全局变量和局部变量都支持一次声明和定义多个变量。

全局变量声明方式：

方式 1：

```go
var <name1>, <name2>, ... <type> = <value1>, <value2>, ...
```

方式 2：

```go
var <name1>, <name2>, ... <type>
```

方式 3：

```go
var <name1>, <name2>, ... = <value1>, <value2>, ...
```

局部变量声明方式：

基本与全局变量的声明方式相同，额外多了不用关键字 var 的方式：

```go
<name1>, <name2>, ... := <value1>, <value2>, ...
```

代码示例：

```go
var a, b, c int = 1, 2, 3

var e, f, g int

var h, i, j = 1, 2, "test"

func method() {
    var k, l, m int = 1, 2, 3
    var n, o, p int
    q, r, s := 1, 2, "test"
    fmt.Println(k, l, m, n, o, p, q, r, s)
}
```