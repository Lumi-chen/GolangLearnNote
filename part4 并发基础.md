---
theme: smartblue
---
# 前言

什么是并发？是指在同一时间段内处理多个任务的能力。

什么是协程？本质上是一种用户态线程，不需要操作系统来进行抢占式调度，且在真正的实现中寄存于线程中，因此，系统开销极小，可以有效提高线程的任务并发性，而避免多线程的缺点。\
优点：编程简单、结构清晰\
缺点：需要语言支持。

> 用户态线程：在用户程序中实现的线程，不依赖操作系统核心。应用程序利用线程库提供创建、同步、调度和管理线程的函数来控制用户线程。

# Goroutine

`goroutine`是go中轻量级线程实现，由go runtime管理。

`go`在语言中是最重要的关键词，在一个函数前加上`go`，则这次调用会在一个新的`goroutine`中并发执行。函数返回则`goroutine`结束，如果这个函数有返回值，那么这个返回值会被丢弃。需要注意，程序不会主动等待其他非主`goroutine`结束。🌰：
```go
package main
import "fmt"
func Add(x, y int) {
    z := x + y
    fmt.Println(z)
}
func main() {
    for i := 0; i < 10; i++ {
        go Add(i, i)
    }
}

// 从程序启动到结束，控制台不会打印任何东西。因为此时被启动的10个非主`goroutine`没来得及返回
```
为了避免上面🌰的问题，我们在主`goroutine`中使用`time.sleep`等待使得其他`goroutine`得以执行完成。
```go
+ import "time"
func main() {
    ...
    + time.Sleep(10 * time.Second)
}
```
# Channel
常见的两种并发通信模型：共享数据和消息。\
共享数据是指多个并发单元分别保持对同一个数据的引用，实现对该数据的共享。\
消息：为每个并发单元是自包含的、独立的个体，并且都有自己的变量，但在不同并发单元间这些变量不共享。每个并发单元的输入和输出只有一种，那就是消息。

`channel`是go提供的`goroutine`之间的通信方式。`channel`是类型相关的，一个`channel`只能传递一种类型的值，需要在声明时指定。

## 基础语法
**声明定义**：`var chanName chan ElementType` 或者内置`make()`:`ch := make(chan ElementType)`\
**写入/发送**：`ch <- value`\
**读取**：`value := <- ch`\
**关闭**：`close(ch)`\
**判断是否关闭**：在读取时判断`x, ok := <-ch`，当ok为false则表示已经被关闭。\
**查看channel待接收元素数量**：`len()`；\
**channel容量**：`cap()`

对于需要持续传输大量数据的场景，我们可以给`channel`带上缓冲。\
**创建带缓存的channel**：`ch := make(chan ElementType, int)`，第二个参数传入的是缓冲区大小。

```go
func Count(ch chan int) {
	ch <- 1
}
func main() {
	chs := make([]chan int, 10)
	for i := 0; i < 10; i++ {
		chs[i] = make(chan int)
		go Count(chs[i])
	}
	for _, ch := range chs {
		value := <-ch
		fmt.Println(value)
	}
}
```
> **注意**： 如果向没有设置缓冲区的channel直接写入数据，会发生死锁。
> 报错信息：*fatal error: all goroutines are asleep - deadlock!*
## 单向channel
单向`channel`只用于发送或者接收数据。
**声明**：
```go
var ch1 chan<- float64 // 只用于写float64数据的单向channel
var ch2 <-chan string  // 只用于读string数据的单向channel
```
**初始化**：可以通过类型转换一个`channel`实现单向`channel`初始化。
```go
ch4 := make(chan int)
ch5 := <-chan int(ch4) // ch5就是一个单向的读取channel
ch6 := chan<- int(ch4) // ch6 是一个单向的写入channel
```
## Select
`select()`用来在多个通信操作中选择执行，常用于处理`goroutine`之间的通信。其用法与`switch`相似，每个选择条件由`case`语句描述。由例子可以看出`select`是直接去查看`case`语句，而不是判断条件。

`case`语句必须是立刻判断是否可以执行的操作，可以是以下几种操作：
1. 接受操作(`<-channel`)
2. 发送操作(`channel <- value)`
3. 默认操作(defalut)
4. 带时间的case（`<-time.After`或`<-time.AfterFunc(duration, func())`）
```go
select {
    case <-chan1:
    // 如果chan1成功读到数据，则进行该case处理语句
    case chan2 <- 1:
    // 如果成功向chan2写入数据，则进行该case处理语句
    default:
    // 如果上面都没有成功，则进入default处理流程
}
```
`select`语句会阻塞，直到其中某个case条件满足为止，如果出现多个满足，则会随机选择一个执行。
```go
func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)

	// 启动一个goroutine来发送数据到ch1
	go func() {
		time.Sleep(1 * time.Second)
		ch1 <- "from ch1"
	}()

	// 启动另一个goroutine来发送数据到ch2
	go func() {
		time.Sleep(2 * time.Second)
		ch2 <- "from ch2"
	}()

	// 使用select来等待ch1或ch2的数据
	select {
	case msg1 := <-ch1:
		fmt.Println("Received", msg1)
	case msg2 := <-ch2:
		fmt.Println("Received", msg2)
	case <-time.After(3 * time.Second):
		fmt.Println("Timeout occurred")
	}
}
// 如果 ch1在三秒内没有发送数据，而ch2发送了，则执行第二个case；
// 如果 ch1 和ch2都没有在三秒内发送数据，则触发第三个case
```
## 超时机制
在并发编程的通信过程中，最需要处理的就是超时问题，即向`channel`写数据时发现`channel`已满，或者从`channel`试图读取数据时发现`channel`为空。不正当处理会导致`goroutine`锁死。

go语言没有直接的超时处理机制，但是可以利用`select`机制和`time`实现。🌰：
```go
ch := make(chan string)
// 启动一个goroutine来模拟耗时操作，可能会向channel发送数据
go func() {
    time.Sleep(2 * time.Second) // 假设耗时操作需要2秒
    ch <- "result"
}()
// 使用select来等待channel的数据或超时
select {
    case result := <-ch: // 从channel成功接收到数据
        fmt.Println("Received result:", result)
    case <-time.After(1 * time.Second): // 超时发生，没有从channel接收到数据
        //`time.After`函数返回一个在指定时间后发送当前时间的channel
        fmt.Println("Operation timed out")
}
```

# 同步
协调多个goroutine的执行顺序，以确保它们在访问共享资源时能够保持数据的一致性和正确性。
go的`sync`包提供了两种锁`sync.Mutex`和`sync.RWMutex`。

## 互斥锁 sync.Mutex
用于保护共享资源，当一个`goroutine`获得锁时，其他试图获得锁的`goroutine`将被阻塞，直到锁被释放。\
提供两个主要方法：`Lock` 和 `Unlock`， 举🌰：
```go
var (
	counter int
	mu      sync.Mutex
)

func increment() {
	// 加锁
	mu.Lock()
	defer mu.Unlock() // 解锁操作放在defer中，确保无论函数如何退出都会执行

	// 临界区：只有获得锁的goroutine才能执行这里的代码
	counter++
	fmt.Println("Counter:", counter)

	// 模拟耗时操作
	time.Sleep(time.Millisecond * 100)
}

func main() {
	var wg sync.WaitGroup
	// 启动多个goroutine来并发增加计数器
	for i := 0; i < 100; i++ {
		wg.Add(1)
		go func() {
			defer wg.Done()
			increment()
		}()
	}

	// 等待所有goroutine执行完毕
	wg.Wait()
	fmt.Println("Final Counter:", counter) // 输出应该是 100
}
```
这里用到一个`sync.WaitGroup`，是一个用于等待一组`goroutine`完成的工具。提供三个方法：
-   `Add(delta int)`: 将等待组的计数器增加指定的值（通常为正数）。通常，在启动每个新的goroutine之前，你会调用`Add(1)`。
-   `Done()`: 将等待组的计数器减一。这通常在goroutine完成其任务后调用，通常放在`defer`语句中以确保即使发生panic也会被调用。
-   `Wait()`: 阻塞当前goroutine，直到等待组的计数器变为零。这通常用于主goroutine中，以等待所有其他goroutine完成。

## 读写锁 sync.RWMutex
是一个读写互斥锁，允许多个goroutine同时读取共享资源，但在写操作时则独占资源。提高读取的效率，因为读取操作通常比写入操作更频繁。\
提供读锁方法：`RLock()`、`RUnlock()`\
提供写锁方法：`Lock()`、`Unlock()`

```go
type SharedData struct {
	mu    sync.RWMutex
	value int
}

func (d *SharedData) Read() int {
	d.mu.RLock()         // 加读锁
	defer d.mu.RUnlock() // 释放读锁，使用defer确保无论函数如何退出都会执行
	return d.value
}

func (d *SharedData) Write(value int) {
	d.mu.Lock()         // 加写锁
	defer d.mu.Unlock() // 释放写锁，使用defer确保无论函数如何退出都会执行
	d.value = value
}

func main() {
	var data SharedData

	// 启动一个goroutine来写入数据
	go func() {
		time.Sleep(2 * time.Second) // 模拟写入前的准备时间
		data.Write(42)
	}()

	// 启动多个goroutine来读取数据
	for i := 0; i < 5; i++ {
		time.Sleep(time.Second)
		go func(i int) {
			fmt.Printf("Goroutine %d: Read value %d\n", i, data.Read())
		}(i)
	}

}
```