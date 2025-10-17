# 1.16 类型转换

类型转换用于将一种数据类型的变量转换为另外一种类型的变量。

在 Go 中，类型转换的基本格式如下：

```go
<type_name>(<expression>)
```

type_name 为类型。

expression 为有返回值的类型。

# 1.16.1 数字类型转换

数字类型之间相互转换比较简单，并且位数较多的类型向位数较少的类型转换时，高位数据会被直接截去。

代码示例：

```go
package main

import "fmt"

func main() {
    var i int32 = 17
    var b byte = 5
    var f float32
    
    // 数字类型可以直接强转
    f = float32(i) / float32(b)
    fmt.Printf("f 的值为: %f\n", f)
    
    // 当int32类型强转成byte时，高位被直接舍弃
    var i2 int32 = 256
    var b2 byte = byte(i2)
    fmt.Printf("b2 的值为: %d\n", b2)
}
```

# 1.16.2 字符串类型转换

前面的部分章节会提到 string 类型、[]byte 类型与[]rune 类型之间可以类似数字类型那样相互转换，并且数据不会有任何丢失。

代码示例：

```go
package main

import "fmt"

func main() {
    str := "hello, 123, 你好"
    var bytes []byte = []byte(str)
    var runes []rune = []rune(str)
    fmt.Printf("bytes 的值为: %v \n", bytes)
    fmt.Printf("runes 的值为: %v \n", runes)

    str2 := string(bytes)
    str3 := string(runes)
    fmt.Printf("str2 的值为: %v \n", str2)
    fmt.Printf("str3 的值为: %v \n", str3)
}
```

但是也会经常有数字与字符串相互转换的需求，这时需要使用到 go 提供的标准库 `strconv`。

strconv 可以把数字转成字符串，也可以把字符串转换成数字。

代码示例：

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    str := "123"
    num, err := strconv.Atoi(str)
    if err != nil {
        panic(err)
    }
    fmt.Printf("字符串转换为int: %d \n", num)
    str1 := strconv.Itoa(num)
    fmt.Printf("int转换为字符串: %s \n", str1)

    ui64, err := strconv.ParseUint(str, 10, 32)
    fmt.Printf("字符串转换为uint64: %d \n", num)

    str2 := strconv.FormatUint(ui64, 2)
    fmt.Printf("uint64转换为字符串: %s \n", str2)
}
```

最常见的转换是字符串与 int 类型之间相互转换。也就是 `Atoi` 方法与 `Itoa` 方法。

当需要把字符串转换成无符号数字时，目前只能转换成 `uint64` 类型，需要其他位的数字类型需要从 `uint64` 类型转到所需的数字类型。

同时可以看到当使用 `ParseUint` 方法把字符串转换成数字时，或者使用 `FormatUint` 方法把数字转换成字符串时，都需要提供第二个参数 `base`，这个参数表示的是数字的进制，即标识字符串输出或输入的数字进制。

# 1.16.3 接口类型转换

接口类型只能通过断言将转换为指定类型。

```go
<variable_name>.(<type_name>)
```

variable_name 是变量名称，type_name 是类型名称。

通过断言方式可以同时得到转换后的值以及转换是否成功的标识。

代码示例：

```go
package main

import "fmt"

func main() {
    var i interface{} = 3
    a, ok := i.(int)
    if ok {
        fmt.Printf("'%d' is a int \n", a)
    } else {
        fmt.Println("conversion failed")
    }
}
```

可以通过转换成功标识符确认转换是否成功。

上面的方式有一个使用 switch 关键字的变体。

代码示例：

```go
package main

import "fmt"

func main() {
    var i interface{} = 3
    switch v := i.(type) {
    case int:
        fmt.Println("i is a int", v)
    case string:
        fmt.Println("i is a string", v)
    default:
        fmt.Println("i is unknown type", v)
    }
}
```

使用 switch 的方式可能更常见一些。

把一个接口类型转换成具体的结构体接口类型。

代码示例：

```go
package main

import "fmt"

type Supplier interface {
    Get() string
}

type DigitSupplier struct {
    value int
}

func (i *DigitSupplier) Get() string {
    return fmt.Sprintf("%d", i.value)
}

func main() {
    var a Supplier = &DigitSupplier{value: 1}
    fmt.Println(a.Get())
    
    b, ok := (a).(*DigitSupplier)
    fmt.Println(b, ok)
}
```

# 1.16.4 结构体类型转换

结构体类型之间在一定条件下也可以转换的。

当两个结构体中的字段名称以及类型都完全相同，仅结构体名称不同时，这两个结构体类型即可相互转换。

代码示例：

```go
package main

import "fmt"

type SameFieldA struct {
    name  string
    value int
}

type SameFieldB struct {
    name  string
    value int
}

func (s *SameFieldB) getValue() int {
    return s.value
}

func main() {
    a := SameFieldA{
        name:  "a",
        value: 1,
    }

    b := SameFieldB(a)
    fmt.Printf("conver SameFieldA to SameFieldB, value is : %d \n", b.getValue())
    
    // 只能结构体类型实例之间相互转换，指针不可以相互转换
    // var c interface{} = &a
    // _, ok := c.(*SameFieldB)
    // fmt.Printf("c is *SameFieldB: %v \n", ok)
}
```