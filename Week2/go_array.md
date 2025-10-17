# 数组

Go 中提供了数组类型的数据结构。

数据是具有相同类型的一组已编号且长度固定的数据项序列，这种类型可以是任意的基础数据类型、自定义类型、指针以及其他数据结构。

数组元素通过索引（下标）来读取或修改，索引从 0 开始，第一个元素索引为 0，第二个索引为 1，以此类推，最后一个元素索引为数组的长度 - 1。

# 1.12.1 声明数组

声明方式 1，仅声明，数组本身已经初始化好了，其中的元素的值为类型的零值：

```go
var <array name> [<length>]<type>
```

声明方式 2，在声明以及初始化：

```go
var <array name> = [<length>]<type>{<element1>, <element2>,...}
<array name> := [<length>]<type>{<element1>, <element2>,...}
```

声明方式 3，可以使用...代替数组长度，编译器会根据初始化时元素个数推断数组长度：

```go
var <array name> = [...]<type>{<element1>, <element2>,...}
<array name> := [...]<type>{<element1>, <element2>,...}
```

声明方式 4，在已指定数组长度的情况下，对指定下标的元素初始化：

```go
var <array name> = [<length>]<type>{<position1>:<element value1>, <position2>:<element value2>,...}
<array name> := [<length>]<type>{<position1>:<element value1>, <position2>:<element value2>,...}
```

代码示例：

```go
func main() {
    // 仅声明
    var a [5]int
    fmt.Println("a = ", a)

    var marr [2]map[string]string
    fmt.Println("marr = ", marr)
    // map的零值是nil，虽然打印出来是非空值，但真实的值是nil
    // marr[0]["test"] = "1"

    // 声明以及初始化
    var b [5]int = [5]int{1, 2, 3, 4, 5}
    fmt.Println("b = ", b)

    // 类型推导声明方式
    var c = [5]string{"c1", "c2", "c3", "c4", "c5"}
    fmt.Println("c = ", c)

    d := [3]int{3, 2, 1}
    fmt.Println("d = ", d)

    // 使用 ... 代替数组长度
    autoLen := [...]string{"auto1", "auto2", "auto3"}
    fmt.Println("autoLen = ", autoLen)

    // 声明时初始化指定下标的元素值
    positionInit := [5]string{1: "position1", 3: "position3"}
    fmt.Println("positionInit = ", positionInit)
    
    // 初始化时，元素个数不能超过数组声明的长度
    //overLen := [2]int{1, 2, 3}
}
```

# 1.12.2 访问数组

方式 1，使用下标读取数组中的元素：

```go
<value> := <array name>[<position>]
```

方式 2，使用 range 遍历：

```go
for <i>,<v> := range <array name> {
    <expression>
    ...
}
```

另外，还可以获取数组长度：

```go
<length variable name> := len(<array name>)
```

代码示例：

```go
func main() {
    a := [5]int{5, 4, 3, 2, 1}

    // 方式1，使用下标读取数据
    element := a[2]
    fmt.Println("element = ", element)

    // 方式2，使用range遍历
    for i, v := range a {
        fmt.Println("index = ", i, "value = ", v)
    }

    for i := range a {
        fmt.Println("only index, index = ", i)
    }

    // 读取数组长度
    fmt.Println("len(a) = ", len(a))
    // 使用下标，for循环遍历数组
    for i := 0; i < len(a); i++ {
        fmt.Println("use len(), index = ", i, "value = ", a[i])
    }
}
```

# 1.12.3 多维数组

声明方式，go 中没有限制多维数组的嵌套层数：

```go
var <array name> [<length1>][<length2>]... <type>
```

代码示例：

```go
func main() {
    // 二维数组
    a := [3][2]int{
        {0, 1},
        {2, 3},
        {4, 5},
    }
    fmt.Println("a = ", a)

    // 三维数组
    b := [3][2][2]int{
        {{0, 1}, {2, 3}},
        {{4, 5}, {6, 7}},
        {{8, 9}, {10, 11}},
    }
    fmt.Println("b = ", b)

    // 也可以省略各个位置的初始化,在后续代码中赋值
    c := [3][3][3]int{}
    c[2][2][1] = 5
    c[1][2][1] = 4
    fmt.Println("c = ", c)
}
```

访问多维数组与访问普通数组的方式一致：

```go
func main() {
    // 三维数组
    a := [3][2][2]int{
        {{0, 1}, {2, 3}},
        {{4, 5}, {6, 7}},
        {{8, 9}, {10, 11}},
    }

    layer1 := a[0]
    layer2 := a[0][1]
    element := a[0][1][1]
    fmt.Println(layer1)
    fmt.Println(layer2)
    fmt.Println(element)

    // 多维数组遍历时，需要使用嵌套for循环遍历
    for i, v := range a {
        fmt.Println("index = ", i, "value = ", v)
        for j, inner := range v {
            fmt.Println("inner, index = ", j, "value = ", inner)
        }
    }
}
```

# 1.12.4 数组作为参数

数组的部分特性类似基础数据类型，当数组作为参数传递时，在函数中并不能改变外部实参的值。

如果想要修改外部实参的值，需要把数组的指针作为参数传递给函数。

代码示例：

```go
type Custom struct {
    i int
}

var carr [5]*Custom = [5]*Custom{
    {6},
    {7},
    {8},
    {9},
    {10},
}

func main() {
    a := [5]int{5, 4, 3, 2, 1}
    fmt.Println("before all, a = ", a)
    for i := range carr {
        fmt.Printf("in main func, carr[%d] = %p, value = %v \n", i, &carr[i], *carr[i])
    }
    printFuncParamPointer(carr)

    receiveArray(a)
    fmt.Println("after receiveArray, a = ", a)

    receiveArrayPointer(&a)
    fmt.Println("after receiveArrayPointer, a = ", a)
}

func receiveArray(param [5]int) {
    fmt.Println("in receiveArray func, before modify, param = ", param)
    param[1] = -5
    fmt.Println("in receiveArray func, after modify, param = ", param)
}

func receiveArrayPointer(param *[5]int) {
    fmt.Println("in receiveArrayPointer func, before modify, param = ", param)
    param[1] = -5
    fmt.Println("in receiveArrayPointer func, after modify, param = ", param)
}

func printFuncParamPointer(param [5]*Custom) {
    for i := range param {
        fmt.Printf("in printFuncParamPointer func, param[%d] = %p, value = %v \n", i, &param[i], *param[i])
    }
}
```