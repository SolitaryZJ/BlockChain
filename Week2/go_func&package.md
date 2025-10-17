# 函数、闭包与方法

# 1.10.1 函数

函数只有三个主要部分，分别是名称、参数列表、返回类型列表。

其中名称是必须的，参数列表和返回类型列表是可选的，也就是说函数可以没有参数，也没有返回值。

定义方式：

```go
func <function_name>(<parameter list>) (<return types>) {
    <expressions>
    ...
}
```

代码示例：

```go
func custom() {
    fmt.Println("Hello, world!")
}
```

# 1.10.2 闭包

闭包，也被称为匿名函数，顾名思义，即没有函数名，通常在函数内或者方法内定义，或者作为参数、返回值进行传递。

匿名函数的优势是可以直接使用当前函数内在匿名函数声明之前声明的变量。

定义方式：

```go
// 声明函数变量
var <closure name> func(<parameter list>) (<return types>)

// 声明闭包
var  <closure name> func(<parameter list>) (<return types>) = func(<parameter list>) (<return types>) {
    <expressions>
    ...
}

// 声明并立刻执行
func(<parameter list>) (<return types>) {
    <expressions>
    ...
}(<value list>)

// 作为参数，并调用
func <function name>(...,<name> func(<parameter list>) (<return types>), ...) {
    ...
    <var1>,... := <name>(<value list>)
    ...
}

// 作为返回值
func <function name>(...) (func(<parameter list>) (<return types>)) {
    ...
    <var1>,... := <name>(<value list>)
    ...
}
```

代码示例：

```go
type A struct {
    i int
}

func (a *A) add(v int) int {
    a.i += v
    return a.i
}

// 声明函数变量
var function1 func(int) int

// 声明闭包
var squart2 func(int) int = func(p int) int {
    p *= p
    return p
}

func main() {
    a := A{1}
    // 把方法赋值给函数变量
    function1 = a.add
    
    // 声明一个闭包并直接执行
    // 此闭包返回值是另外一个闭包（带参闭包）
    returnFunc := func() func(int, string) (int, string) {
        fmt.Println("this is a anonymous function")
        return func(i int, s string) (int, string) {
            return i, s
        }
    }()

    // 执行returnFunc闭包并传递参数
    ret1, ret2 := returnFunc(1, "test")
    fmt.Println("call closure function, return1 = ", ret1, "; return2 = ", ret2)

    fmt.Println("a.i = ", a.i)
    fmt.Println("after call function1, a.i = ", function1(1))
    fmt.Println("a.i = ", a.i)
}
```

# 1.10.3 方法

与函数相比，方法是一个包含接受者的函数，大部分情况下可以通过类型的实例调用。

也可以把方法赋值给一个函数变量，使用函数变量调用这个方法，调用方式类似闭包。

定义可以参考文档  [结构体](https://github.com/RemoteCodeCamp/golang-v2/blob/main/02.%E7%AC%AC%E4%B8%80%E7%AB%A0%20Golang%E7%BC%96%E7%A8%8B%E5%9F%BA%E7%A1%80/1.5%20%E7%BB%93%E6%9E%84%E4%BD%93/%E7%BB%93%E6%9E%84%E4%BD%93.md)

代码示例：

```go
type A struct {
    i int
}

// 定义方法
func (a *A) add(v int) int {
    a.i += v
    return a.i
}

// 声明函数变量
var function func(int) int

func main() {
    a := A{1}
    function = a.add

    fmt.Println("after call function, a.i = ", function(1))
    fmt.Println("a.i = ", a.i)
}
```