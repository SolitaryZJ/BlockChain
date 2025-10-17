# 1.8 流程控制

# 1.8.1 if 语句

if 语句由一个或多个布尔表达式组成，且布尔表达式可以不加括号。

if/else 语句声明方式：

```go
if <expression> {
    <do sth1>
} else {
    <do sth2>
}
```

If/else if 语句使用方式：

```go
if <expression1> {
    <do sth1>
} else if <expression2> {
    <do sth2>
} else {
    <do sth2>
}
```

if/else 嵌套：

```go
if <expression1> {
    if <expression2> {
        <do sth1>
    } else {
        <do sth2>
    }
} else if <expression3> {
    <do sth3>
} else {
    <do sth4>
}
```

if/else 语句还可以在布尔表达式之前额外增加声明赋值语句，来声明作用域仅在当前 if 作用域内的变量：

```go
var <name1> <type>
if <name2> := <method or expression>; <expression> {
    <do sth1>
} else {
    <do sth2>
}
```

代码示例：

```go
func main() {
    var a int = 10
    if b := 1; a > 10 {
        b = 2
        // c = 2
        fmt.Println("a > 10")
    } else if c := 3; b > 1 {
        b = 3
        fmt.Println("b > 1")
    } else {
        fmt.Println("其他")
        if c == 3 {
            fmt.Println("c == 3")
        }
        fmt.Println(b)
        fmt.Println(c)
    }
}
```

# 1.8.2 switch 语句

基于不同条件执行不同的动作。

每个 case 分之都是唯一的，从上往下逐一判断，直到匹配为止。如果某些 case 条件重复，编译时会报错。

默认情况下 case 分支自带 break 效果，无需在每个 case 中声明 break，中断匹配。

switch 使用方式 1：

```go
switch <variable> {
case <value1>:
    <do sth1>
case value2:
    <do sth2>
case <value3>, <value4>: // 可以匹配多个值，只要一个满足条件即可
    <do sth34>
case value5:
    <do sth5>
default:
    <do sth>
}
```

switch 使用方式 2：

```go
switch <variable> := <method or expression>; <variable> {
case <value1>:
    <do sth1>
case value2:
    <do sth2>
case <value3>, <value4>:
    <do sth34>
case value5:
    <do sth5>
default:
    <do sth>
}
```

switch 使用方式 3，case 分支的 expression 的结果必须是 `bool` 类型：

```go
switch {
case <expression1>:
    <do sth1>
case <expression2>:
    <do sth2>
case <expression3>, <expression4>:
    <do sth34>
default:
    <do sth>
}
```

switch 使用方式 4，仅适用于接口和泛型：

```go
switch v := x.(type) {
case <type1>:
    <do sth1>
case <type2>:
    <do sth2>
case <type3>:
    <do sth3>
default:
    <do sth>
}
```

代码示例：

```go
func main() {
    a := "test string"

    // 1. 基本用法
    switch a {
    case "test":
        fmt.Println("a = ", a)
    case "s":
        fmt.Println("a = ", a)
    case "t", "test string": // 可以匹配多个值，只要一个满足条件即可
        fmt.Println("catch in a test, a = ", a)
    case "n":
        fmt.Println("a = not")
    default:
        fmt.Println("default case")
    }

    // 变量b仅在当前switch代码块内有效
    switch b := 5; b {
    case 1:
        fmt.Println("b = 1")
    case 2:
        fmt.Println("b = 2")
    case 3, 4:
        fmt.Println("b = 3 or 4")
    case 5:
        fmt.Println("b = 5")
    default:
        fmt.Println("b = ", b)
    }

    // 不指定判断变量，直接在case中添加判定条件
    b := 5
    switch {
    case a == "t":
        fmt.Println("a = t")
    case b == 3:
        fmt.Println("b = 5")
    case b == 5, a == "test string":
        fmt.Println("a = test string; or b = 5")
    default:
        fmt.Println("default case")
    }

    var d interface{}
    // var e byte = 1
    d = 1
    switch t := d.(type) {
    case byte:
        fmt.Println("d is byte type, ", t)
    case *byte:
        fmt.Println("d is byte point type, ", t)
    case *int:
        fmt.Println("d is int type, ", t)
    case *string:
        fmt.Println("d is string type, ", t)
    case *CustomType:
        fmt.Println("d is CustomType pointer type, ", t)
    case CustomType:
        fmt.Println("d is CustomType type, ", t)
    default:
        fmt.Println("d is unknown type, ", t)
    }
}
```