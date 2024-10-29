go学习

### 1.入门

#### Echo

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

一般用前两种方式，第一种方式只能在函数内部使用，不能用于包变量，第二种方式依赖于默认初始化机制

前面用到 s += sep + arg，这个会将s的原始值丢弃，在适当的时机进行垃圾回收，但如果涉及到的数据量很大，这种方式代价高昂，所以可以使用strings包的Join函数

```go
func main() {
    fmt.Println(strings.Join(os.Args[1:], " "))
}
```

#### 查找重复行

```go
// Dup1 prints the text of each line that appears more than
// once in the standard input, preceded by its count.
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
    counts := make(map[string]int)
    input := bufio.NewScanner(os.Stdin)
    for input.Scan() {
        counts[input.Text()]++
    }
    // NOTE: ignoring potential errors from input.Err()
    for line, n := range counts {
        if n > 1 {
            fmt.Printf("%d\t%s\n", n, line)
        }
    }
}
```

map存储的是键/值的集合，对集合元素提供常数时间的存/取/测试操作，键和值都可以是任意类型，**内置函数是make，用于为map创建空间**

上面输出用的是**Printf**，这个函数提供了完全的格式控制，需要格式字符串和相应的参数，需要转义字符

```go
%d          十进制整数
%x, %o, %b  十六进制，八进制，二进制整数。
%f, %g, %e  浮点数： 3.141593 3.141592653589793 3.141593e+00
%t          布尔：true或false
%c          字符（rune） (Unicode码点)
%s          字符串
%q          带双引号的字符串"abc"或带单引号的字符'c'
%v          变量的自然形式（natural format）
%T          变量的类型
%%          字面上的百分号标志（无操作数）
```

如果用Println就不需要转义字符，简单地输出其参数，参数之间自动添加空格

```go
fmt.Println(n, "\t", line)
```

程序要么从标准输入中读取数据，要么从一系列具名文件中读取数据，如下面代码

```go
// Dup2 prints the count and text of lines that appear more than once
// in the input.  It reads from stdin or from a list of named files.
package main

import (
    "bufio"
    "fmt"
    "os"
)

func main() {
    counts := make(map[string]int)
    files := os.Args[1:]
    if len(files) == 0 {
      //files为0，就走标准输入去数行
        countLines(os.Stdin, counts)
    } else {
      //否则从具名文件中读取数据
        for _, arg := range files {
            f, err := os.Open(arg)
            if err != nil {
                fmt.Fprintf(os.Stderr, "dup2: %v\n", err)
                continue
            }
          //将打开的文件传入数行函数中
            countLines(f, counts)
            f.Close()
        }
    }
    for line, n := range counts {
        if n > 1 {
            fmt.Printf("%d\t%s\n", n, line)
        }
    }
}

func countLines(f *os.File, counts map[string]int) {
    input := bufio.NewScanner(f)
    for input.Scan() {
        counts[input.Text()]++
    }
    // NOTE: ignoring potential errors from input.Err()
}
```

这里需要注意两个点：

1. go函数传递参数是传递值，所以想改变原来值的话需要传递指针
2. bufio.NewScanner(f)可以用于处理来自不同类型的输入源，包括标准输入，文件等

打开文件除了用os.Open(arg)，然后对f进行NewScanner，也可以直接使用ioutil.ReadFile，直接读取文件中的内容，代码下

```go
package main

import (
    "fmt"
    "io/ioutil"
    "os"
    "strings"
)

func main() {
    counts := make(map[string]int)
    for _, filename := range os.Args[1:] {
      //这里返回的是一个字节切片，必须转位string类型，才能用split
        data, err := ioutil.ReadFile(filename)
        if err != nil {
            fmt.Fprintf(os.Stderr, "dup3: %v\n", err)
            continue
        }
        for _, line := range strings.Split(string(data), "\n") {
            counts[line]++
        }
    }
    for line, n := range counts {
        if n > 1 {
            fmt.Printf("%d\t%s\n", n, line)
        }
    }
}
```

#### GIF动画

```go
// Lissajous generates GIF animations of random Lissajous figures.
package main

import (
    "image"
    "image/color"
    "image/gif"
    "io"
    "math"
    "math/rand"
    "os"
    "time"
)

var palette = []color.Color{color.White, color.Black}

const (
    whiteIndex = 0 // first color in palette
    blackIndex = 1 // next color in palette
)

func main() {
    // The sequence of images is deterministic unless we seed
    // the pseudo-random number generator using the current time.
    // Thanks to Randall McPherson for pointing out the omission.
    rand.Seed(time.Now().UTC().UnixNano())
    lissajous(os.Stdout)
}

func lissajous(out io.Writer) {
    const (
        cycles  = 5     // number of complete x oscillator revolutions
        res     = 0.001 // angular resolution
        size    = 100   // image canvas covers [-size..+size]
        nframes = 64    // number of animation frames
        delay   = 8     // delay between frames in 10ms units
    )

    freq := rand.Float64() * 3.0 // relative frequency of y oscillator
  //struct类型
    anim := gif.GIF{LoopCount: nframes}
    phase := 0.0 // phase difference
    for i := 0; i < nframes; i++ {
        rect := image.Rect(0, 0, 2*size+1, 2*size+1)
        img := image.NewPaletted(rect, palette)
        for t := 0.0; t < cycles*2*math.Pi; t += res {
            x := math.Sin(t)
            y := math.Sin(t*freq + phase)
            img.SetColorIndex(size+int(x*size+0.5), size+int(y*size+0.5),
                blackIndex)
        }
        phase += 0.1
      //将结果append到anim中的帧列表末尾，并设置一个默认的80ms的延迟值
        anim.Delay = append(anim.Delay, delay)
        anim.Image = append(anim.Image, img)
    }
    gif.EncodeAll(out, &anim) // NOTE: ignoring encoding errors
}


```

[]color.Color{...}和gif.GIF{...}这两个表达是复合声明，前者生成一个slice切片，后者是生成一个struct结构体

#### 并发获取多个URL

```go
// Fetchall fetches URLs in parallel and reports their times and sizes.
package main

import (
    "fmt"
    "io"
    "io/ioutil"
    "net/http"
    "os"
    "time"
)

func main() {
  //记录程序开始时间
    start := time.Now()
  //创建通信通道
    ch := make(chan string)
  //遍历输入链接，对于每个输入链接都创建一个goroutine
    for _, url := range os.Args[1:] {
        go fetch(url, ch) // start a goroutine
    }
  //从通道中读取每个fetch结果
    for range os.Args[1:] {
        fmt.Println(<-ch) // receive from channel ch
    }
  //计算耗时
    fmt.Printf("%.2fs elapsed\n", time.Since(start).Seconds())
}

func fetch(url string, ch chan<- string) {
    start := time.Now()
    resp, err := http.Get(url)
    if err != nil {
        ch <- fmt.Sprint(err) // send to channel ch
        return
    }
  //这个io.Copy会将响应体的内容拷贝到输出流中
    nbytes, err := io.Copy(ioutil.Discard, resp.Body)
    resp.Body.Close() // don't leak resources
    if err != nil {
        ch <- fmt.Sprintf("while reading %s: %v", url, err)
        return
    }
    secs := time.Since(start).Seconds()
    ch <- fmt.Sprintf("%.2fs  %7d  %s", secs, nbytes, url)
}

```

当一个goroutine尝试在一个通道（channel）上发送或接收数据时，它会阻塞，直到另一个相应的操作完成

指针：go语言提供了指针，指针是直接存储变量的内存地址的数据结构，&操作符可以返回一个变量的内存地址，*可以获取指针指向的变量内容，但在Go中没有指针运算

### 2.程序结构

#### 命名

Go中命名还是依照：一个名字必须以一个字母或下划线开头，后面可以跟任意数量的字母、数字或下划线。区分大小写

名字定义的**位置**决定了它的有效范围，在函数内定义则仅在函数内有效，若在函数外定义，则在当前包所有文件中都可访问

名字**开头字母的大小**决定了它在包外是否可见，如果是大写字母开头且定义在函数外，则可以被外部包访问

#### 声明

go有四种类型的声明语句：var、const、type、func，分别对应变量、常量、函数和函数实体对象的声明

一个go程序对应一个或多个以go为文件后缀名的源文件，**每个源文件中以包声明语句开始，说明该源文件属于哪个包**

#### 变量

var声明语句可以创建一个特定类型的变量，然后给变量附加一个名字，设置初始值

```go
var 变量名字 类型 = 表达式
```

其中“类型”或“=表达式”可以省略其中一个

**简短变量声明**

在函数内部，可以使用简短变量声明语句，如下所示

```go
变量名字 := 表达式
```

和var形式声明语句一样，简短变量声明语句也可以用来声明和初始化一组变量

```go
i, j := 0, 1
```

如果这一组变量里面有之前声明过的，就只赋值，但这一组里面至少有一个是需要声明的，否则就会报错

**指针**

