# 切片

切片(Slice)并不是数组或者数组指针，而是数组的一个引用，

切片本身是一个标准库中实现的一个特殊的结构体，这个结构体中有三个属性，分别代表数组指针、长度、容量。

具体可以查看 golang 源码仓库中 src/runtime/slice.go 文件中：

```go
type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
}
```

# 1.13.1 声明与初始化切片

切片的申明方式与声明数组的方式非常相似，与数组相比，切片不用声明长度:

```go
var <slice name> []<type>
```

初始化切片，代码示例：

```go
// 方式1，声明并初始化一个空的切片
var s1 []int = []int{}

// 方式2，类型推导，并初始化一个空的切片
var s2 = []int{}

// 方式3，与方式2等价
s3 := []int{}

// 方式4，与方式1、2、3 等价，可以在大括号中定义切片初始元素
s4 := []int{1, 2, 3, 4}

// 方式5，用make()函数创建切片，创建[]int类型的切片，指定切片初始长度为0
s5 := make([]int, 0)

// 方式6，用make()函数创建切片，创建[]int类型的切片，指定切片初始长度为2，指定容量参数4
s6 := make([]int, 2, 4) 

// 方式7，引用一个数组，初始化切片
a := [5]int{6,5,4,3,2}
// 从数组下标2开始，直到数组的最后一个元素
s7 := arr[2:]
// 从数组下标1开始，直到数组下标3的元素，创建一个新的切片
s8 := arr[1:3]
// 从0到下标2的元素，创建一个新的切片
s9 := arr[:2]
```

当切片是基于同一个数组指针创建出来时，修改数组中的值时，同样会影响到这些切片。

代码示例：

```go
func main() {
    a := [5]int{6, 5, 4, 3, 2}
    // 从数组下标2开始，直到数组的最后一个元素
    s7 := a[2:]
    // 从数组下标1开始，直到数组下标3的元素，创建一个新的切片
    s8 := a[1:3]
    // 从0到下标2的元素，创建一个新的切片
    s9 := a[:2]
    fmt.Println(s7)
    fmt.Println(s8)
    fmt.Println(s9)
    a[0] = 9
    a[1] = 8
    a[2] = 7
    fmt.Println(s7)
    fmt.Println(s8)
    fmt.Println(s9)
}
```

# 1.13.2 使用切片

## 1.13.2.1 访问切片

访问切片中的元素，与访问数组一样，代码示例：

```go
s1 := []int{5, 4, 3, 2, 1}
// 下标访问切片
e1 := s1[0]
e2 := s1[1]
e3 := s1[2]
fmt.Println(s1)

// 向指定位置赋值
s1[0] = 10
s1[1] = 9
s1[2] = 8
fmt.Println(s1)

// range迭代访问切片
for i, v := range s1 {
    fmt.Println("before modify, s1[%d] = %d", i, v)
}
```

切片还可以使用 `len()` 和 `cap()` 函数访问切片的长度和容量。

长度表示切片可以访问到底层数组的数据范围。

容量表示切片引用的底层数组的长度。

当切片是 nil 时，`len()` 和 `cap()` 函数获取的到值都是 0。

切片的长度小于等于切片的容量。

代码示例：

```go
var nilSlice []int
fmt.Println("nilSlice length:", len(nilSlice))
fmt.Println("nilSlice capacity:", len(nilSlice))

s2 := []int{9, 8, 7, 6, 5}
fmt.Println("s2 length: ", len(s2))
fmt.Println("s2 capacity: ", cap(s2))
```

## 1.13.2.2 切片添加元素

切片是变长的，可以向切片追加新的元素，可以使用内置的 `append()` 向切片追加元素。

内置函数 `append()` 只有切片类型可以使用，第一个参数必须是切片类型，后面追加的元素参数是变长类型，一次可以追加多个元素到切片。并且每次 `append()` 都会返回一个新的切片引用。

代码示例：

```go
s3 := []int{}
fmt.Println("s3 = ", s3)

// append函数追加元素
s3 = append(s3)
s3 = append(s3, 1)
s3 = append(s3, 2, 3)
fmt.Println("s3 = ", s3)
```

除了使用 `append()` 函数向切片追加元素以外，还可以使用 `append()` 向指定位置添加元素，以及移除指定位置的元素。

向指定位置添加元素的代码示例：

```go
s4 := []int{1, 2, 4, 5}
s4 = append(s4[:2], append([]int{3}, s4[2:]...)...)
fmt.Println("s4 = ", s4)
```

移除指定位置元素代码示例：

```go
s5 := []int{1,2,3,5,4}
s5 = append(s5[:3], s5[4:]...)
fmt.Println("s5 = ", s5)
```

## 1.13.2.3 复制切片

可以使用内置函数 `copy()` 把某个切片中的所有元素复制到另一个切片，复制的长度是它们中最短的切片长度。

代码示例：

```go
src1 := []int{1, 2, 3}
dst1 := make([]int, 4, 5)

src2 := []int{1, 2, 3, 4, 5}
dst2 := make([]int, 3, 3)

fmt.Println("before copy, src1 = ", src1)
fmt.Println("before copy, dst1 = ", dst1)

fmt.Println("before copy, src2 = ", src2)
fmt.Println("before copy, dst2 = ", dst2)

copy(dst1, src1)
copy(dst2, src2)

fmt.Println("before copy, src1 = ", src1)
fmt.Println("before copy, dst1 = ", dst1)

fmt.Println("before copy, src2 = ", src2)
fmt.Println("before copy, dst2 = ", dst2)
```

# 1.13.3 切片底层原理

切片类型实际上是比较特殊的指针类型，当声明一个切片类型时，就是声明了一个指针。

这个指针指向的切片结构体，切片结构体中记录的三个属性：数组指针、长度、容量。这几个属性在创建一个切片时就定义好，并且在之后都不能再被修改。

先看代码示例：

```go
func main() {
    s := make([]int, 3, 6)
    fmt.Println("s length:", len(s))
    fmt.Println("s capacity:", cap(s))
    fmt.Println("initial, s = ", s)
    s[1] = 2
    fmt.Println("set position 1, s = ", s)

    modifySlice(s)
    fmt.Println("after modifySlice, s = ", s)
}

func modifySlice(param []int) {
    param[0] = 1024
}
```

将会输出日志：

```go
s length: 3
s capacity: 6
initial, s =  [0 0 0]
set position 1, s =  [0 2 0]
after modifySlice, s =  [1024 2 0]
```

在不使用 `append()` 函数的情况下，在函数内部对切片的修改，都会影响到原始实例。

使用 append()函数时，需要分两种情况：

当没有触发切片扩容时：

```go
func main() {
    s := make([]int, 3, 6)
    fmt.Println("initial, s =", s)
    s[1] = 2
    fmt.Println("after set position 1, s =", s)

    s2 := append(s, 4)
    fmt.Println("after append, s2 length:", len(s2))
    fmt.Println("after append, s2 capacity:", cap(s2))
    fmt.Println("after append, s =", s)
    fmt.Println("after append, s2 =", s2)

    s[0] = 1024
    fmt.Println("after set position 0, s =", s)
    fmt.Println("after set position 0, s2 =", s2)

    appendInFunc(s)
    fmt.Println("after append in func, s =", s)
    fmt.Println("after append in func, s2 =", s2)
}

func appendInFunc(param []int) {
    param = append(param, 1022)
    fmt.Println("in func, param =", param)
    param[2] = 512
    fmt.Println("set position 2 in func, param =", param)
}
```

将输出日志：

```shell
initial, s = [0 0 0]
after set position 1, s = [0 2 0]
after append, s2 length: 4
after append, s2 capacity: 6
after append, s = [0 2 0]
after append, s2 = [0 2 0 4]
after set position 0, s = [1024 2 0]
after set position 0, s2 = [1024 2 0 4]
in func, param = [1024 2 0 1022]
set position 2 in func, param = [1024 2 512 1022]
after append in func, s = [1024 2 512]
after append in func, s2 = [1024 2 512 1022]
```

当使用 `append()` 函数之后。

原来的切片引用，长度和容量不变，新追加的值超过切片可访问范围，访问不到新追加的值。

新的切片引用，与原始切片引用相比，长度加一，容量不变，可以访问到新追加的值。

在方法内，使用原始切片作为参数，使用 `append()` 函数追加元素后，同样会创建一个新的切片引用，新追加的值会覆盖之前的值。

并且修改这个切片，其修改同样会反应到原始切片以及新的切片引用上。

当 `append()` 函数触发扩容时：

```go
func main() {
    s := make([]int, 2, 2)
    fmt.Println("initial, s =", s)

    s2 := append(s, 4)
    fmt.Println("after append, s length:", len(s))
    fmt.Println("after append, s capacity:", cap(s))

    fmt.Println("after append, s2 length:", len(s2))
    fmt.Println("after append, s2 capacity:", cap(s2))
    fmt.Println("after append, s =", s)
    fmt.Println("after append, s2 =", s2)

    s[0] = 1024
    fmt.Println("after set position 0, s =", s)
    fmt.Println("after set position 0, s2 =", s2)

    appendInFunc(s2)
    fmt.Println("after append in func, s2 =", s2)
}

func appendInFunc(param []int) {
    param1 := append(param, 511)
    param2 := append(param1, 512)
    fmt.Println("in func, param1 =", param1)
    param2[2] = 500
    fmt.Println("set position 2 in func, param2 =", param2)
}
```

输出日志：

```shell
initial, s = [0 0]
after append, s length: 2
after append, s capacity: 2
after append, s2 length: 3
after append, s2 capacity: 4
after append, s = [0 0]
after append, s2 = [0 0 4]
after set position 0, s = [1024 0]
after set position 0, s2 = [0 0 4]
in func, param1 = [0 0 4 511]
set position 2 in func, param2 = [0 0 500 511 512]
after append in func, s2 = [0 0 4]
```

当 `append()` 函数触发扩容后，实际上是新创建了一个数组实例，把原来的数组中的数据复制到了新数组中，然后创建一个新的切片实例并返回。

这时原始切片中持有的数组指针指向的数组与新切片引用中的数组指针指向的数组是两个不同的数组，修改并不会相互影响。

1. 切片触发扩容前，切片一直共用相同的数组；
2. 切片触发扩容后，会创建新的数组，并复制这些数据；
3. 切片本身是一个特殊的指针，go 针对切片类型添加了一些语法糖，方便使用。