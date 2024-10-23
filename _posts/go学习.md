go学习

```go
// Echo2 prints its command-line arguments.
package main

import (
    "fmt"
    "os"
)

func main() {
    s, sep := "", ""
    for _, arg := range os.Args[1:] {
        s += sep + arg
        sep = " "
    }
    fmt.Println(s)
}

```

每次循环迭代时，range会产生一对值，**索引以及在该索引处的元素值**，这个例子不需要索引，但按照range的语法，要必须处理索引，有两种思路：

1. 把索引值赋给一个临时变量，然后忽略掉这个临时变量，但go不允许使用无用局部变量，这会导致编译错误。
2. 用空标识符（_），用于在任何语法需要变量名，但程序编译不需要的时候丢弃不要的值

声明一个变量有下面几种方式：

```go
s := ""
var s string
var s = ""
var s string = ""
```

