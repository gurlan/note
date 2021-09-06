# GO语言基础

## 数组

### 概念

数组是一个由固定长度的特定类型元素组成的序列，一个数组可以由零个或多个元素组成。数组的长度是数组类型的组成部分。

### 定义方式

```go
var a [3]int                    // 定义长度为3的int型数组, 元素全部为0
var b = [...]int{1, 2, 3}       // 定义长度为3的int型数组, 元素为 1, 2, 3
var c = [...]int{2: 3, 1: 2}    // 定义长度为3的int型数组, 元素为 0, 2, 3
var d = [...]int{1, 2, 4: 5, 6} // 定义长度为6的int型数组, 元素为 1, 2, 0, 0, 5, 6
```

### 内存结构

数组的内存结构比较简单。比如下面是一个`[4]int{2,3,5,7}`数组值对应的内存结构：

![img](https://chai2010.cn/advanced-go-programming-book/images/ch1-7-array-4int.ditaa.png)

## 字符串

### 概念

一个字符串是一个不可改变的字节序列，字符串通常是用来包含人类可读的文本数据。和数组不同的是，字符串的元素不可修改，是一个只读的字节数组。每个字符串的长度虽然也是固定的，但是字符串的长度并不是字符串类型的一部分。

### 内存结构

Go语言字符串的底层结构在`reflect.StringHeader`中定义：

```go
type StringHeader struct {
    Data uintptr
    Len  int
}
```

字符串结构由两个信息组成：第一个是字符串指向的底层字节数组，第二个是字符串的字节的长度。字符串其实是一个结构体，因此字符串的赋值操作也就是`reflect.StringHeader`结构体的复制过程，并不会涉及底层字节数组的复制。在前面数组一节提到的`[2]string`字符串数组对应的底层结构和`[2]reflect.StringHeader`对应的底层结构是一样的，可以将字符串数组看作一个结构体数组。

![img](https://chai2010.cn/advanced-go-programming-book/images/ch1-8-string-1.ditaa.png)

## 切片

### 概念

简单地说，切片就是一种简化版的动态数组。

### 定义方式

```go
var (
    a []int               // nil切片, 和 nil 相等, 一般用来表示一个不存在的切片
    b = []int{}           // 空切片, 和 nil 不相等, 一般用来表示一个空的集合
    c = []int{1, 2, 3}    // 有3个元素的切片, len和cap都为3
    d = c[:2]             // 有2个元素的切片, len为2, cap为3
    e = c[0:2:cap(c)]     // 有2个元素的切片, len为2, cap为3
    f = c[:0]             // 有0个元素的切片, len为0, cap为3
    g = make([]int, 3)    // 有3个元素的切片, len和cap都为3
    h = make([]int, 2, 3) // 有2个元素的切片, len为2, cap为3
    i = make([]int, 0, 3) // 有0个元素的切片, len为0, cap为3
)
```

### 内存结构

我们先看看切片的结构定义，`reflect.SliceHeader`：

```go
type SliceHeader struct {
    Data uintptr
    Len  int
    Cap  int
}
```

可以看出切片的开头部分和Go字符串是一样的，但是切片多了一个`Cap`成员表示切片指向的内存空间的最大容量（对应元素的个数，而不是字节数）。下图是`x := []int{2,3,5,7,11}`和`y := x[1:3]`两个切片对应的内存结构。

![img](https://chai2010.cn/advanced-go-programming-book/images/ch1-10-slice-1.ditaa.png)

和数组一样，内置的`len`函数返回切片中有效元素的长度，内置的`cap`函数返回切片容量大小，容量必须大于或等于切片的长度。也可以通过`reflect.SliceHeader`结构访问切片的信息（只是为了说明切片的结构，并不是推荐的做法）。切片可以和`nil`进行比较，只有当切片底层数据指针为空时切片本身为`nil`，这时候切片的长度和容量信息将是无效的。如果有切片的底层数据指针为空，但是长度和容量不为0的情况，那么说明切片本身已经被损坏了（比如直接通过`reflect.SliceHeader`或`unsafe`包对切片作了不正确的修改）。

### **添加切片元素**

内置的泛型函数`append`可以在切片的尾部追加`N`个元素：

```go
var a []int
a = append(a, 1)               // 追加1个元素
a = append(a, 1, 2, 3)         // 追加多个元素, 手写解包方式
a = append(a, []int{1,2,3}...) // 追加一个切片, 切片需要解包
```

不过要注意的是，在容量不足的情况下，`append`的操作会导致重新分配内存，可能导致巨大的内存分配和复制数据代价。即使容量足够，依然需要用`append`函数的返回值来更新切片本身，因为新切片的长度已经发生了变化。

除了在切片的尾部追加，我们还可以在切片的开头添加元素：

```go
var a = []int{1,2,3}
a = append([]int{0}, a...)        // 在开头添加1个元素
a = append([]int{-3,-2,-1}, a...) // 在开头添加1个切片
```

**`append`函数返回新的切片**

### 删除切片元素

根据要删除元素的位置有三种情况：从开头位置删除，从中间位置删除，从尾部删除。其中删除切片尾部的元素最快：

```go
a = []int{1, 2, 3}
a = a[:len(a)-1]   // 删除尾部1个元素
a = a[:len(a)-N]   // 删除尾部N个元素
```

删除开头的元素可以直接移动数据指针：

```go
a = []int{1, 2, 3}
a = a[1:] // 删除开头1个元素
a = a[N:] // 删除开头N个元素
```

删除开头的元素也可以不移动数据指针，但是将后面的数据向开头移动。可以用`append`原地完成（所谓原地完成是指在原有的切片数据对应的内存区间内完成，不会导致内存空间结构的变化）：

```go
a = []int{1, 2, 3}
a = append(a[:0], a[1:]...) // 删除开头1个元素
a = append(a[:0], a[N:]...) // 删除开头N个元素
```

## 面向并发的内存模型

### Goroutine和系统线程

运行时有一个`runtime.GOMAXPROCS`变量，用于控制当前运行正常非阻塞Goroutine的系统线程数目。

### 原子操作

所谓的原子操作就是并发编程中“最小的且不可并行化”的操作。

#### 加锁实现

```go
import (
    "sync"
)

var total struct {
    sync.Mutex
    value int
}

func worker(wg *sync.WaitGroup) {
    defer wg.Done()

    for i := 0; i <= 100; i++ {
        total.Lock()
        total.value += i
        total.Unlock()
    }
}

func main() {
    var wg sync.WaitGroup
    wg.Add(2)
    go worker(&wg)
    go worker(&wg)
    wg.Wait()

    fmt.Println(total.value)
}
```

#### 自带实现

```go
import (
    "sync"
    "sync/atomic"
)

var total uint64

func worker(wg *sync.WaitGroup) {
    defer wg.Done()

    var i uint64
    for i = 0; i <= 100; i++ {
        atomic.AddUint64(&total, i)
    }
}

func main() {
    var wg sync.WaitGroup
    wg.Add(2)

    go worker(&wg)
    go worker(&wg)
    wg.Wait()
}
```

go的单例模式

```go
var (
    instance *singleton
    once     sync.Once
)

func Instance() *singleton {
    once.Do(func() {
        instance = &singleton{}
    })
    return instance
}
```

## 常见的并发模型

#### 并发版本的Hello world

##### 加锁实现

```go
func main() {
    var mu sync.Mutex

    mu.Lock()
    go func(){
        fmt.Println("你好, 世界")
        mu.Unlock()
    }()

    mu.Lock()
}
```

##### 管道实现

```go
func main() {
    done := make(chan int)

    go func(){
        fmt.Println("你好, 世界")
        <-done
    }()

    done <- 1
}
```

上面的代码虽然可以正确同步，但是对管道的缓存大小太敏感：如果管道有缓存的话，就无法保证main退出之前后台线程能正常打印了。更好的做法是将管道的发送和接收方向调换一下，这样可以避免同步事件受管道缓存大小的影响：

```go
func main() {
    done := make(chan int, 1) // 带缓存的管道

    go func(){
        fmt.Println("你好, 世界")
        done <- 1
    }()

    <-done
}
```

##### WaitGroup实现

```go
func main() {
    var wg sync.WaitGroup

    // 开N个后台打印线程
    for i := 0; i < 10; i++ {
        wg.Add(1)

        go func() {
            fmt.Println("你好, 世界")
            wg.Done()
        }()
    }

    // 等待N个后台线程完成
    wg.Wait()
}
```

#### 生产者消费者模型

```go
// 生产者: 生成 factor 整数倍的序列
func Producer(factor int, out chan<- int) {
    for i := 0; ; i++ {
        out <- i*factor
    }
}

// 消费者
func Consumer(in <-chan int) {
    for v := range in {
        fmt.Println(v)
    }
}
func main() {
    ch := make(chan int, 64) // 成果队列

    go Producer(3, ch) // 生成 3 的倍数的序列
    go Producer(5, ch) // 生成 5 的倍数的序列
    go Consumer(ch)    // 消费 生成的队列

    // 运行一定时间后退出
    time.Sleep(5 * time.Second)
}
```

我们开启了2个`Producer`生产流水线，分别用于生成3和5的倍数的序列。然后开启1个`Consumer`消费者线程，打印获取的结果。我们通过在`main`函数休眠一定的时间来让生产者和消费者工作一定时间。正如前面一节说的，这种靠休眠方式是无法保证稳定的输出结果的。

我们可以让`main`函数保存阻塞状态不退出，只有当用户输入`Ctrl-C`时才真正退出程序：

```go
func main() {
    ch := make(chan int, 64) // 成果队列

    go Producer(3, ch) // 生成 3 的倍数的序列
    go Producer(5, ch) // 生成 5 的倍数的序列
    go Consumer(ch)    // 消费 生成的队列

    // Ctrl+C 退出
    sig := make(chan os.Signal, 1)
    signal.Notify(sig, syscall.SIGINT, syscall.SIGTERM)
    fmt.Printf("quit (%v)\n", <-sig)
}
```

我们这个例子中有2个生产者，并且2个生产者之间并无同步事件可参考，它们是并发的。因此，消费者输出的结果序列的顺序是不确定的，这并没有问题，生产者和消费者依然可以相互配合工作。

#### 发布订阅模型

发布订阅（publish-and-subscribe）模型通常被简写为pub/sub模型。在这个模型中，消息生产者成为发布者（publisher），而消息消费者则成为订阅者（subscriber），生产者和消费者是M:N的关系。在传统生产者和消费者模型中，是将消息发送到一个队列中，而发布订阅模型则是将消息发布给一个主题。

为此，我们构建了一个名为`pubsub`的发布订阅模型支持包：

```go
// Package pubsub implements a simple multi-topic pub-sub library.
package pubsub

import (
    "sync"
    "time"
)

type (
    subscriber chan interface{}         // 订阅者为一个管道
    topicFunc  func(v interface{}) bool // 主题为一个过滤器
)

// 发布者对象
type Publisher struct {
    m           sync.RWMutex             // 读写锁
    buffer      int                      // 订阅队列的缓存大小
    timeout     time.Duration            // 发布超时时间
    subscribers map[subscriber]topicFunc // 订阅者信息
}

// 构建一个发布者对象, 可以设置发布超时时间和缓存队列的长度
func NewPublisher(publishTimeout time.Duration, buffer int) *Publisher {
    return &Publisher{
        buffer:      buffer,
        timeout:     publishTimeout,
        subscribers: make(map[subscriber]topicFunc),
    }
}

// 添加一个新的订阅者，订阅全部主题
func (p *Publisher) Subscribe() chan interface{} {
    return p.SubscribeTopic(nil)
}

// 添加一个新的订阅者，订阅过滤器筛选后的主题
func (p *Publisher) SubscribeTopic(topic topicFunc) chan interface{} {
    ch := make(chan interface{}, p.buffer)
    p.m.Lock()
    p.subscribers[ch] = topic
    p.m.Unlock()
    return ch
}

// 退出订阅
func (p *Publisher) Evict(sub chan interface{}) {
    p.m.Lock()
    defer p.m.Unlock()

    delete(p.subscribers, sub)
    close(sub)
}

// 发布一个主题
func (p *Publisher) Publish(v interface{}) {
    p.m.RLock()
    defer p.m.RUnlock()

    var wg sync.WaitGroup
    for sub, topic := range p.subscribers {
        wg.Add(1)
        go p.sendTopic(sub, topic, v, &wg)
    }
    wg.Wait()
}

// 关闭发布者对象，同时关闭所有的订阅者管道。
func (p *Publisher) Close() {
    p.m.Lock()
    defer p.m.Unlock()

    for sub := range p.subscribers {
        delete(p.subscribers, sub)
        close(sub)
    }
}

// 发送主题，可以容忍一定的超时
func (p *Publisher) sendTopic(
    sub subscriber, topic topicFunc, v interface{}, wg *sync.WaitGroup,
) {
    defer wg.Done()
    if topic != nil && !topic(v) {
        return
    }

    select {
    case sub <- v:
    case <-time.After(p.timeout):
    }
}
```

下面的例子中，有两个订阅者分别订阅了全部主题和含有"golang"的主题：

```go
import "path/to/pubsub"

func main() {
    p := pubsub.NewPublisher(100*time.Millisecond, 10)
    defer p.Close()

    all := p.Subscribe()
    golang := p.SubscribeTopic(func(v interface{}) bool {
        if s, ok := v.(string); ok {
            return strings.Contains(s, "golang")
        }
        return false
    })

    p.Publish("hello,  world!")
    p.Publish("hello, golang!")

    go func() {
        for  msg := range all {
            fmt.Println("all:", msg)
        }
    } ()

    go func() {
        for  msg := range golang {
            fmt.Println("golang:", msg)
        }
    } ()

    // 运行一定时间后退出
    time.Sleep(3 * time.Second)
}
```

在发布订阅模型中，每条消息都会传送给多个订阅者。发布者通常不会知道、也不关心哪一个订阅者正在接收主题消息。订阅者和发布者可以在运行时动态添加，是一种松散的耦合关系，这使得系统的复杂性可以随时间的推移而增长。在现实生活中，像天气预报之类的应用就可以应用这个并发模式。

#### 素数筛

在“Hello world 的革命”一节中，我们为了演示Newsqueak的并发特性，文中给出了并发版本素数筛的实现。并发版本的素数筛是一个经典的并发例子，通过它我们可以更深刻地理解Go语言的并发特性。“素数筛”的原理如图：

![img](https://chai2010.cn/advanced-go-programming-book/images/ch1-13-prime-sieve.png)

*图 1-13 素数筛*

我们需要先生成最初的`2, 3, 4, ...`自然数序列（不包含开头的0、1）：

```go
// 返回生成自然数序列的管道: 2, 3, 4, ...
func GenerateNatural() chan int {
    ch := make(chan int)
    go func() {
        for i := 2; ; i++ {
            ch <- i
        }
    }()
    return ch
}
```

`GenerateNatural`函数内部启动一个Goroutine生产序列，返回对应的管道。

然后是为每个素数构造一个筛子：将输入序列中是素数倍数的数提出，并返回新的序列，是一个新的管道。

```go
// 管道过滤器: 删除能被素数整除的数
func PrimeFilter(in <-chan int, prime int) chan int {
    out := make(chan int)
    go func() {
        for {
            if i := <-in; i%prime != 0 {
                out <- i
            }
        }
    }()
    return out
}
```

`PrimeFilter`函数也是内部启动一个Goroutine生产序列，返回过滤后序列对应的管道。

现在我们可以在`main`函数中驱动这个并发的素数筛了：

```go
func main() {
    ch := GenerateNatural() // 自然数序列: 2, 3, 4, ...
    for i := 0; i < 100; i++ {
        prime := <-ch // 新出现的素数
        fmt.Printf("%v: %v\n", i+1, prime)
        ch = PrimeFilter(ch, prime) // 基于新素数构造的过滤器
    }
}
```

我们先是调用`GenerateNatural()`生成最原始的从2开始的自然数序列。然后开始一个100次迭代的循环，希望生成100个素数。在每次循环迭代开始的时候，管道中的第一个数必定是素数，我们先读取并打印这个素数。然后基于管道中剩余的数列，并以当前取出的素数为筛子过滤后面的素数。不同的素数筛子对应的管道是串联在一起的。

素数筛展示了一种优雅的并发程序结构。但是因为每个并发体处理的任务粒度太细微，程序整体的性能并不理想。对于细粒度的并发程序，CSP模型中固有的消息传递的代价太高了（多线程并发模型同样要面临线程启动的代价）。

#### 并发的安全退出

有时候我们需要通知goroutine停止它正在干的事情，特别是当它工作在错误的方向上的时候。Go语言并没有提供在一个直接终止Goroutine的方法，由于这样会导致goroutine之间的共享变量处在未定义的状态上。但是如果我们想要退出两个或者任意多个Goroutine怎么办呢？

Go语言中不同Goroutine之间主要依靠管道进行通信和同步。要同时处理多个管道的发送或接收操作，我们需要使用`select`关键字（这个关键字和网络编程中的`select`函数的行为类似）。当`select`有多个分支时，会随机选择一个可用的管道分支，如果没有可用的管道分支则选择`default`分支，否则会一直保存阻塞状态。

基于`select`实现的管道的超时判断：

```go
select {
case v := <-in:
    fmt.Println(v)
case <-time.After(time.Second):
    return // 超时
}
```

通过`select`的`default`分支实现非阻塞的管道发送或接收操作：

```go
select {
case v := <-in:
    fmt.Println(v)
default:
    // 没有数据
}
```

通过`select`来阻止`main`函数退出：

```go
func main() {
    // do some thins
    select{}
}
```

当有多个管道均可操作时，`select`会随机选择一个管道。基于该特性我们可以用`select`实现一个生成随机数序列的程序：

```go
func main() {
    ch := make(chan int)
    go func() {
        for {
            select {
            case ch <- 0:
            case ch <- 1:
            }
        }
    }()

    for v := range ch {
        fmt.Println(v)
    }
}
```

我们通过`select`和`default`分支可以很容易实现一个Goroutine的退出控制:

```go
func worker(cannel chan bool) {
    for {
        select {
        default:
            fmt.Println("hello")
            // 正常工作
        case <-cannel:
            // 退出
        }
    }
}

func main() {
    cannel := make(chan bool)
    go worker(cannel)

    time.Sleep(time.Second)
    cannel <- true
}
```

但是管道的发送操作和接收操作是一一对应的，如果要停止多个Goroutine那么可能需要创建同样数量的管道，这个代价太大了。其实我们可以通过`close`关闭一个管道来实现广播的效果，所有从关闭管道接收的操作均会收到一个零值和一个可选的失败标志。

```go
func worker(cannel chan bool) {
    for {
        select {
        default:
            fmt.Println("hello")
            // 正常工作
        case <-cannel:
            // 退出
        }
    }
}

func main() {
    cancel := make(chan bool)

    for i := 0; i < 10; i++ {
        go worker(cancel)
    }

    time.Sleep(time.Second)
    close(cancel)
}
```

我们通过`close`来关闭`cancel`管道向多个Goroutine广播退出的指令。不过这个程序依然不够稳健：当每个Goroutine收到退出指令退出时一般会进行一定的清理工作，但是退出的清理工作并不能保证被完成，因为`main`线程并没有等待各个工作Goroutine退出工作完成的机制。我们可以结合`sync.WaitGroup`来改进:

```go
func worker(wg *sync.WaitGroup, cannel chan bool) {
    defer wg.Done()

    for {
        select {
        default:
            fmt.Println("hello")
        case <-cannel:
            return
        }
    }
}

func main() {
    cancel := make(chan bool)

    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go worker(&wg, cancel)
    }

    time.Sleep(time.Second)
    close(cancel)
    wg.Wait()
}
```

现在每个工作者并发体的创建、运行、暂停和退出都是在`main`函数的安全控制之下了。

# Go语言核心36讲

## 01 | 工作区和GOPATH

- GOROOT：Go 语言安装根目录的路径，也就是 GO 语言的安装路径。
- GOPATH：若干工作区目录的路径。是我们自己定义的工作空间。
- GOBIN：GO 程序生成的可执行文件（executable file）的路径。

![img](https://static001.geekbang.org/resource/image/2f/3c/2fdfb5620e072d864907870e61ae5f3c.png)