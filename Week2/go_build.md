# 1.20 编译可执行文件

在 Go 中，当需要发布程序时，项目都会先编译成可执行文件。

# 1.20.1 无参编译

如果声明 main 函数的文件就在项目目录下，只需要在项目根目录下执行 go build 命令：

```go
go build
```

Go 就会根据当前目录下声明的 go.mod 文件中，声明的 module 名称，把当前项目的执行文件输出到 module 名称同名的文件中。

如果想要修改路径和编译出来的可执行文件的名称，还需要添加 `-o` 参数：

```go
go build -o <output_path>
```

另外，Go 的工程标准，一般会把声明 main 函数的文件放到项目目录的 `cmd` 目录下，所以需要再加上这个文件的路径：

```go
go build -o <output_path_with_file_name> <go_file_path>
```

# 1.20.2 交叉编译

而为了让程序在不同平台下正常运行，Go 还让 build 命令在执行时，可以读到环境变量中的一些参数以及 tags 参数。

其中两比较重要的变量分别是：

- **GOOS**：指定运行操作系统，目前支持 `linux`、`windows` 和 `darwin`(MacOS)，三种类型。
- **GOARCH**：指定运行的架构，可以配置 `amd64`, `arm64`, `i386` 等。
- **CGO_ENABLED**：如果项目依赖于另外的 C 库或其他工具，就需要在交叉编译时配置 `CGO_ENABLED` 变量，默认情况下是 0，也就是关闭，当为 1 时，表示开启。

当需要使用到这些变量时，可以配置成环境变量，也可以直接在命令中定义，即可开始编译：

```go
//编译 Linux 64位
GOOS=linux GOARCH=amd64 go build -o demo-amd64 main.go

//编译 Windows 64位
GOOS=windows GOARCH=amd64 go build -o demo-windows-amd64.exe main.go

//编译 macOS 64位
GOOS=darwin GOARCH=amd64 go build -o demo-darwin-amd64 main.go

//编译 Linux 32位
GOOS=linux GOARCH=i386 go build -o myapp-linux-386 main.go
```

这样就可以编译适配不同架构的可执行文件了。

# 1.20.3 条件编译

有时候在不同架构或系统下，由于各种各样的原因，不同系统提供的函数是不同的，那么在为了保证在不同系统下，程序能够正常运行。

又或者公司的项目，为了区分收费版与免费版的程序，也可以使用条件编译的方式，让程序能够在一个项目下，编译出不同行为的可执行文件。

使用条件编译也很简单，在 go 代码文件头，`package` 语句前面添加特定的 build 标签如下所示：

创建 func_linux.go 文件，以下是文件代码

```go
// +build linux
package main

import "fmt"

func platformSpecificFunction() {
    fmt.Println("This is the Linux implementation.")
}
```

创建 func_windows.go 文件：

```go
// +build windows
package main

import "fmt"
func platformSpecificFunction() {
    fmt.Println("This is the Windows implementation.")
}
```

创建 func_darwin.go 文件：

```go
// +build darwin
package main

import "fmt"
func platformSpecificFunction() {
    fmt.Println("This is the Darwin implementation.")
}
```

注：在 go1.17 版本以上，上面这几个文件的代码会被 IDE 自动添加一行代码：`//go:build xxx`，这是 go 新引入的条件编译的格式，是为了替换 `+ build`。

在 `main.go` 中，可以直接调用 `platformSpecificFunction`。

```go
package main

func main() {
    platformSpecificFunction()
}
```

可以直接使用 `go build` 命令编译并执行。

这里添加 `tags` 参数，是因为 `linux`、`windows` 以及 `darwin` 这三个 tag 比较特殊，会根据 GOARCH 自动添加。

假如把 tag 名称改成其他的值，就需要在编译时特意指定，定义格式：

```go
go build -tags="<tag1> <tag2> <...>"
```

## 条件编译的 tag 逻辑

在一个包中使用多个标签时，它们会使用布尔逻辑进行交互，具体取决于我们如何进行声明。

Build tags 遵循以下三个规则：

- 以空格分隔的标签将在 OR 逻辑下进行解释。
- 逗号分隔的标签将在 AND 逻辑下进行解释。
- 每个术语都是一个字母数字单词，如果前面有! 它意味着它被否定。

### or 标签逻辑

给定标签：

```go
// +build tag1 tag2
```

当编译时的 `tags` 参数只要声明了 tag1、tag2 这两个 tag 值任意一个，则编译时包含此文件。

### and 标签逻辑

声明格式：

```go
// +build tag1, tag2
```

或者如下声明：

```go
// +build tag1
// +build tag2
```

当编译时的 `tags` 参数需要同时声明 tag1 和 tag2 时，这个 go 文件才能包含在编译中。

### ! - 非标签逻辑

如果我们使用标签

```go
// +build !tag1
```

当编译时的 `tags` 参数没有 `tag1` 的值时，这个 go 文件才会 build 编译。

### 新版 Tag 逻辑关系

新版为了让 tag 之间的逻辑关系更加明显且容易辨认与编写， 简化了 tag 声明，并且为了与老版 tag 声明区分，修改的声明格式：

```go
// go:build linux && amd64 || darwin
```

从 `+ build` 变成了 `go:build`，并且逻辑关系也更加明显，直接使用逻辑运算符来声明 tag 之间的逻辑关系。

目前当声明旧的条件编译条件时，go 的开发环境工具会帮我们自动生成新版 tag 的声明。