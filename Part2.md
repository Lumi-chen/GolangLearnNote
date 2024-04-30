# Part2

# 流程控制
流程控制是整个程序的骨架，一般起三个作用：选择、循环、跳转。Go语言提供以下几种流程控制语句：

## 条件语句
条件语句与其他语言相似，但有几点需要注意
1. 条件表达式无需用括号包含
2. 无论执行体有几条语句，`{}`都是必须的
3. `{`必须和`if`/`else`在同一行
4. 在`if`之后，条件表达式之前，可以添加变量初始化语句，使用`;`间隔
5. 在`有返回值的函数`中，不允许将 **“最终的”return** 语句包含在`if...else...`结构中，否则会编译失败
```go
// 格式
if 条件表达式 {
    // 执行体
} else {
    // 执行体
}
// 关于第4点，举🌰
func main() {
    x := 10
    if y := 20; y > x {
        return y
    } else {
        return x
    }
}

// 关于第5点，举错误🌰❌
func example(x int) int {
    if x == 0 {
        return 5
    } else {
        return x
    }
}
```
## 选择语句
关于switch结构使用，有几点需要注意：
1. `{`必须与`switch`同一行
2. 条件表达式不限制为常量/整数
3. 不需要使用`break`明确一个退出`case`
4. 只有在`case`中明确添加`fallthrough`关键字，才会继续执行紧跟着的下一个`case`
5. 可以不设定`switch`之后的条件表达式，等同于多个`if...else..`的逻辑作用
```go
// 格式
switch 条件表达式 {
    case 条件表达式1:
    case 条件表达式2:
    case 条件表达式3:
    case 条件表达式4, 条件表达式5, 条件表达式6:
    ...
    defalut:
    
}
// 关于第4点
switch i {
    ...
    case 3:
        fallthrough
    case 4:
        fmt.Println("4")
    ...
}
// 当i= 3时，输出4
```
## 循环语句
Go中循环语句只支持`for`关键字，不支持`while`和`do-while`。需要注意的点：
1. `{`必须与`for`同一行
2. 允许在循环条件中定义和初始化变量，但go不支持以逗号为间隔的多个赋值语句，必须使用**平行赋值**的方式初始化多个变量
3. 支持`continue`和`beak`控制循环，且提供了一个高级`break`，可以选择中断哪个循环：给**某一层循环**设置一个Label,指定跳过某一个Label。
```go
// 格式
for 初始化; 循环条件; 更新部分; {
    // 循环体
}
// 关于第2点, 举🌰：
for i, j := 0, 1;  i < j; i++ {
    ...
}
// 关于第3点，举🌰：
for i := 0; i < 10; i++ {
Loop： // 标记该循环为Loop
    for j :=0 ; j < 5; j++ {
        if j == 2 {
            break Loop // 跳出Loop标记的循环
        }
    }
}
```

## 跳转语句
`goto` 语句用于无条件地跳转到程序中的指定标签（label）。

*尽量避免使用，过度使用 `goto` 可能会导致代码结构混乱，难以理解和维护。*
```go
goto label
// ... 其他代码 ...
label:
// 跳转到这里的代码
```
# 函数
函数的组成：`func`关键字、函数名、参数列表、返回值、函数体和返回语句。
```go
// 格式
func 函数名(参数列表) 返回值类型 {
    // 函数体
    ...
    return 返回语句
}
// 🌰：
func Add(a, b int) int {
    sum := a + b
    return sum
}
```
**⚠️需要注意**： 小写字母开头的函数只在本包可用，大写字母开头的函数才能导出包外用。【规则适用于变量和类型】

## 不定参数
指函数传入参数个数为不定数量。使用`...type`格式（一个语法糖），且**必须是最后一个参数**，其本质是一个数组切片。`type`类型可以是**指定类型**，也可以是任意类型`interface{}`
```go
// 指定类型
func myfun(args ...int) {
    fmt.Println(args)
}
myfun(1,2,3) // 输入 [1, 2, 3]

// 任意类型
func myfun(args ...interface{}) {
    fmt.Println(args)
}
myfun(1,"hello", true) // 输出 [1, hello, true] 
```
**参数传递**：如果有另一个变参函数，可以使用`...`传递，如下：
```go
func myfun(args ...int) {
    // 原样传递
   myfun2(args...)
    // 片段传递
   myfun3(arg[1:]...)
}
```
## 多返回值
Go语言的函数或者成员的方法可以有多个返回值，还可以为返回值命名【不强制命名】，像函数的输入参数一样。返回值被命名之后，它们的值在函数开始的时候被自动初始化为空。在函数中执行不带任何参数的return语句时，会返回对应的返回值变量的值。

## 匿名函数
不需要定义函数名的函数实现方式。可以像普通变量一样被传递/使用，可以直接赋值给一个变量或直接执行：
```go
// 格式：
func(参数列表) 返回值类型 {
    // 函数体
    ...
    return 返回语句
}
// 赋值变量🌰：
f := func(x, y int) int {
    return x + y 
}
// 直接执行🌰: 花括号后直接跟参数列表表示函数调用
func (x, y int) {
    fmt.Println(x, y)
}(1,2) // 输出: 1 2
```
## 闭包
闭包是可以包含自由变量的代码块，变量不在代码块内或任意全局上下文定义，在定义代码块的环境中定义。🌰：匿名函数就是闭包。

闭包可以用于**函数对象**或**匿名函数**，举🌰：
```go
func adder() func(int) int {  
    sum := 0 // 外部变量，被闭包引用  
    return func(x int) int {  
        sum += x // 修改外部变量  
        return sum  
    }  
}  
  
func main() {  
    addFive := adder() // 调用 adder() 返回闭包  
    fmt.Println(addFive(1)) // 输出 1  
    fmt.Println(addFive(2)) // 输出 3（因为 sum 的值在上一次调用后增加了）  
  
    // 创建一个新的闭包  
    addSeven := adder()  
    fmt.Println(addSeven(1)) // 输出 1（新的闭包，新的 sum 变量）  
}
```
# 错误处理
## error接口
一个错误处理的标准模式，`error`接口。定义如下：
```go
type error interface {
    Error() string
}
```
**自定义error类型**：因为Go的接口灵活性，不需要从`error`接口继承。用于更复杂的错误处理，比如包含错误码、堆栈跟踪或其他元数据时，自定义错误类型就非常有用了。举🌰：
```go
// 自定义错误类型
type MyCustomError struct {
	Code    int    // 错误码
	Message string // 错误描述
}
// 实现 error 接口的 Error() 方法
func (e *MyCustomError) Error() string {
	return fmt.Sprintf("code=%d, message=%s", e.Code, e.Message)
}
// 一个可能会返回自定义错误的函数
func doSomething(x int) (int, error) {
    if x == 1 {
        return 1, nil
    } else {
        return 0, &MyCustomError{Code: 404, Message: "Not Found"}
    }
}

func main() {
    // 调用函数并处理可能的错误
    _, err := doSomething(1)
    if err != nil {
        fmt.Println("An error occurred:", err)
        // 你还可以进行类型断言，以检查是否是特定的错误类型
        if customErr, ok := err.(*MyCustomError); ok {
            fmt.Printf("Custom error details: code=%d, message=%s\n", customErr.Code, customErr.Message)
        }
    } else {
        fmt.Println("No Error")
    }
}
```
## defer 
关键字`defer`是Go引入的一个有意思的特性。用于在函数返回之前执行一些清理操作或延迟执行的语句:
1.  资源释放：如文件关闭、网络连接断开、锁释放等。
2.  清理操作：如记录日志、更新状态等。
3.  延迟执行：确保某些操作在函数返回之前被执行。

一个函数中可以有多个`defer`语句，其调用遵照先进后出原则（LIFO）。举🌰：
```go
func main() {  
    defer fmt.Println("First defer")  
    defer fmt.Println("Second defer")  
    fmt.Println("Hello, World!")  
}  
  
// 输出：  
// Hello, World!  
// Second defer  
// First defer
```
## panic() 和 recover()
是用于处理运行时错误和异常的两个内建函数。
`panic` 可以用来在程序运行时抛出一个错误，这个错误会立即中断当前的函数执行，并逐层向上冒泡，直到被捕获或程序崩溃。`panic`接收任意类型数据。\
使用 `panic` 的一个常见场景是在检测到不可能恢复的错误时，比如索引越界、空指针引用等。

`recover` 则是用来捕获 `panic` 的，它只能用在 `defer` 函数中，用来阻止 `panic` 的继续冒泡，并恢复正常的程序执行流程。。如果没有在发生异常的`goroutine`中明确调用恢复过程（使用recover关键字），会导致该`goroutine`所属的进程打印异常信息后直接退出。\
`recover` 的返回值是触发 `panic` 时传递的参数，如果 `panic` 没有传递参数，那么 `recover` 返回 `nil`。

举🌰：
```go
func main() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered from panic:", r)
        }
    }()
    // 触发 panic
    goFunc()
    // 程序能够继续执行
    fmt.Println("After potential panic")
}

func goFunc() {
    defer func() {
        if r := recover(); r != nil {
            fmt.Println("Recovered in goFunc:", r)
        }
    }()
    // 触发 panic
    panic("something bad happened in goFunc")
}
// 思路理解：
// goFunc：defer延后执行，遇到panic，触发并中断当前执行，又defer里有recover，阻止了panic，所以fmt.Println("After potential panic")能继续执行。
// 又因为panic已经被阻止了，所以main函数里的recover()为<nil>，所以，if判断为false。
```