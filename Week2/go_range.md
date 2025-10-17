# 1.15 range 迭代

在 Go 中，range 关键字用于 for 循环迭代字符串(string)、数组(array)、切片(slice)、通道(channel)或映射集合(map)中的元素。

# 1.15.1 对字符串迭代

在 Go 中，string 类型是一个比较特殊的类型，可以与 rune 切片类型、byte 切片类型相互转换，同时还可以使用 range 关键字来遍历一个字符串。

方式 1，仅使用 range 获取下标索引：

```go
package main

import "fmt"

func main() {
    str1 := "abc123"
    for index := range str1 {
        fmt.Printf("str1 -- index:%d, value:%d\n", index, str1[index])
    }

    str2 := "测试中文"
    for index := range str2 {
        fmt.Printf("str2 -- index:%d, value:%d\n", index, str2[index])
    }
    fmt.Printf("len(str2) = %d\n", len(str2))

    runesFromStr2 := []rune(str2)
    bytesFromStr2 := []byte(str2)
    fmt.Printf("len(runesFromStr2) = %d\n", len(runesFromStr2))
    fmt.Printf("len(bytesFromStr2) = %d\n", len(bytesFromStr2))
}
```

示例代码输出内容：

```go
str1 -- index:0, value:97
str1 -- index:1, value:98
str1 -- index:2, value:99
str1 -- index:3, value:49
str1 -- index:4, value:50
str1 -- index:5, value:51
str2 -- index:0, value:230
str2 -- index:3, value:232
str2 -- index:6, value:228
str2 -- index:9, value:230
len(str2) = 12
len(runesFromStr2) = 4
len(bytesFromStr2) = 12
```

这里需要强调一点，在 Go 中，所有字符串都是按照 Unicode 编码的。

第一 for 循环中，遍历了变量 `str1`，它有六个字符，而且这些字符都可以使用一个 byte 表示，所以循环了六次才退出循环。

而第二个 for 循环中，遍历了变量 `str2`，它有四个中文字符，那么按照 Unicode 编码的标准，很显然不能仅仅被四个 byte 表示，但它还是只循环了 4 次，恰好是中文字符的长度。
那么也就是说，在 Go 中，遍历字符串时，实际上是在遍历从字符串转换来的 rune 切片，只是恰好在某些时候，字符串转换成 byte 切片和字符串转换成 rune 切片之后的长度相同，看起来是在逐个遍历 byte 切一样。

方式 2，使用 range 获取下标和下标位置的字符：

```go
package main

import "fmt"

func main() {
    str1 := "a1中文"
    for index, value := range str1 {
        fmt.Printf("str1 -- index:%d, index value:%d\n", index, str1[index])
        fmt.Printf("str1 -- index:%d, range value:%d\n", index, value)
    }
}
```

示例代码输出：

```go
str1 -- index:0, index value:97
str1 -- index:0, range value:97
str1 -- index:1, index value:49
str1 -- index:1, range value:49
str1 -- index:2, index value:228
str1 -- index:2, range value:20013
str1 -- index:5, index value:230
str1 -- index:5, range value:25991
```

当直接使用下标取字符串某个下标位置上的值时，取出来的是 byte 值。

但是当使用 range 关键字直接获取到某个下标位置的值时，取出的是一个完整的 rune 类型的值。

# 1.15.2 对数组与切片迭代

在 Go 中，实际代码执行过程中，使用 range 迭代数组和切片，它们两者的体验是相同的。

遍历一维数组与切片：

```go
package main

import "fmt"

func main() {
    array := [...]int{1, 2, 3}
    slice := []int{4, 5, 6}

    // 方法1：只拿到数组的下标索引
    for index := range array {
        fmt.Printf("array -- index=%d value=%d \n", index, array[index])
    }
    for index := range slice {
        fmt.Printf("slice -- index=%d value=%d \n", index, slice[index])
    }
    fmt.Println()

    // 方法2：同时拿到数组的下标索引和对应的值
    for index, value := range array {
        fmt.Printf("array -- index=%d index value=%d \n", index, array[index])
        fmt.Printf("array -- index=%d range value=%d \n", index, value)
    }
    for index, value := range slice {
        fmt.Printf("slice -- index=%d index value=%d \n", index, slice[index])
        fmt.Printf("slice -- index=%d range value=%d \n", index, value)
    }
    fmt.Println()
}
```

遍历二维数组与切片：

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    array := [...][3]int{{1, 2, 3}, {4, 5, 6}}
    slice := [][]int{{1, 2}, {3}}
    // 只拿到行的索引
    for index := range array {
        // array[index]类型是一维数组
        fmt.Println(reflect.TypeOf(array[index]))
        fmt.Printf("array -- index=%d, value=%v\n", index, array[index])
    }

    for index := range slice {
        // slice[index]类型是一维数组
        fmt.Println(reflect.TypeOf(slice[index]))
        fmt.Printf("slice -- index=%d, value=%v\n", index, slice[index])
    }

    // 拿到行索引和该行的数据
    fmt.Println("print array element")
    for row_index, row_value := range array {
        fmt.Println(row_index, reflect.TypeOf(row_value), row_value)
    }

    fmt.Println("print array slice")
    for row_index, row_value := range slice {
        fmt.Println(row_index, reflect.TypeOf(row_value), row_value)
    }

    // 双重遍历，拿到每个元素的值
    for row_index, row_value := range array {
        for col_index, col_value := range row_value {
            fmt.Printf("array[%d][%d]=%d ", row_index, col_index, col_value)
        }
        fmt.Println()
    }
    for row_index, row_value := range slice {
        for col_index, col_value := range row_value {
            fmt.Printf("slice[%d][%d]=%d ", row_index, col_index, col_value)
        }
        fmt.Println()
    }
}
```

切片与数组相比，特殊的地方就在于其长度可变，所以在构成二维时，切片中元素的数量可以随意设置，而数组是定长的。

使用 range 迭代时，两者体验完全一致。

# 1.15.3 对通道迭代

通道除了可以使用 for 循环配合 select 关键字获取数据以外，也可以使用 for 循环配合 range 关键字获取数据。

因为通道结构的特殊性，当使用 range 遍历通道时，只给一个迭代变量赋值，而不像数组或字符串一样能够使用 index 索引。

当通道被关闭时，在 range 关键字迭代完通道中所有值后，循环就会自动退出。

```go
package main

import (
    "fmt"
    "time"
)

func addData(ch chan int) {
    size := cap(ch)
    for i := 0; i < size; i++ {
        ch <- i
        time.Sleep(1 * time.Second)
    }
    close(ch)
}

func main() {
    ch := make(chan int, 10)

    go addData(ch)

    for i := range ch {
        fmt.Println(i)
    }
}
```

# 1.15.4 对映射集合迭代

在 Go 中，使用 range 关键字迭代映射集合时，一种是拿到 key，一种是拿到 key 和 value，并且 range 关键字在迭代映射集合时，其中的 key 是乱序的。

```go
package main

import "fmt"

func main() {
    hash := map[string]int{
        "a": 1,
        "f": 2,
        "z": 3,
        "c": 4,
    }

    for key := range hash {
        fmt.Printf("key=%s, value=%d\n", key, hash[key])
    }

    for key, value := range hash {
        fmt.Printf("key=%s, value=%d\n", key, value)
    }
}
```