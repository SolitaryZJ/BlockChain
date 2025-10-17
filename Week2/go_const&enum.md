# 1.6 常量与枚举

# 1.6.1 常量

由于常量的值是在编译期确定的，所以常量定义时**必须赋值**，并且不能使用方法的返回值为常量赋值。

常量被定义后，其值不能再被修改。

常量（包括全局常量和局部常量）被定义后可以不使用。

常量的定义方式与变量定义的方式基本相同，只是 var 关键字被更换成了 const：

方式 1：

```go
const <name> <type> = <value>
```

方式 2：

```go
// 某些类型可以直接推导出来，不需要声明
const <name> = <value>
```

方式 3：

```go
const <name3>, <name4>, ... = <value3>, <value4>, ...
```

方式 4：

```go
const <name5>, <name6>, ... <type> = <value5>, <value6>, ...
```

方式 5：

```go
// 声明多个时，可以用小括号包裹，此模式不限制声明次数
const (
  <name1> <type1> = <value1>
  <name2> = <value2>
  <name3>, <name4>, ... = <value3>, <value4>, ...
  <name5>, <name6>, ... <type> = <value5>, <value6>, ...
)
```

代码示例：

```go
// 方式1
const a int = 1

// 方式2
const b = "test"

// 方式3
const c, d = 2, "hello"

// 方式4
const e, f bool = true, false

// 方式5
const (
    h    byte = 3
    i         = "value"
    j, k      = "v", 4
    l, m      = 5, false
)

const (
    n = 6
)
```

注：Go 中，**常量只能使用基本数据类型**，即数字、字符串和布尔类型。不能使用复杂的数据结构，比如切片、数组、map、指针和结构体等。如果使用了非基本数据类型，会在编译期报错。

# 1.6.2 枚举

Go 中没有内置枚举类型，所以 Go 中的枚举是使用 const 来定义枚举的。

## 1.6.2.1 定义枚举

枚举的本质就是一系列的常量。所以 Go 中使用 const 定义枚举，比如：

```go
const (
    Male = "Male"
    Female = "Female"
)
```

除了直接定义值以外，还会使用类型别名，让常量定义的枚举类型的作用显得更直观，比如：

```go
type Gender string

const (
   Male   Gender = "Male"
   Female Gender = "Female"
)
```

当此枚举作为参数传递时，会使用 Gender 作为参数类型，而不是基础类型 string，比如：

```go
func method(gender Gender) {}
```

并且使用了类型别名后，还可以为这个别名类型声明自定义方法：

```go
func (g *Gender) String() string {
    switch *g {
    case Male:
        return "Male"
    case Female:
        return "Female"
    default:
        return "Unknown"
    }
}

func (g *Gender) IsMale() bool {
    return *g == Male
}
```

## 1.6.2.2 `iota` 关键字

除了上面的别名类型来声明枚举类型以外，还可以使用 iota 关键字，来自动为常量赋值。

在来接 iota 辅助声明枚举之前，先了解一下 iota 关键字的使用。

可以先看看标准库中 iota 的使用：

```go
// src/net/http/server.go
type ConnState int
const (
  StateNew ConnState = iota
  StateActive
  StateIdle
  StateHijacked
  StateClosed
)

// src/time/time.go
type Month int
const (
  January Month = 1 + iota
  February
  March
  April
  May
  June
  July
  August
  September
  October
  November
  December
)
```

iota 是一个方便定义常量的关键字。

iota 独立作用于每个 const 定义组，就是上面看到的 `const ( ``// ``code... )` 结构。

并且每个 const 语句算作是一个 const 定义组。

如果 iota 定义在 const 定义组中的第 n 行，那么 iota 的值为 n - 1。所以一定要注意 iota 出现在定义组中的第几行，而不是当前代码中它第几次出现。

```go
const pre int = 1
const a int = iota
const (
    b int = iota
    c
    d
    e
)
const (
    f = 2
    g = iota
    h
    i
)
```

使用 iota 关键字就是为了方便我们定义常量的值。

并且当这些枚举值仅作为判断条件使用时，修改非常方便，只需要的其分组增删即可。

注：iota 仅能与 const 关键字配合使用。

示例代码：

```go
type Gender byte
const (
    Male Gender = iota
    Female
)
```