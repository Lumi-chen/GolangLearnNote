# Part 1

# 前言

最近虽然一直在投简历，但是面试很少，在家闲着也是闲着。之前虽然也是在学习Vue3，自己搭框架写些有的没的。但是没有后端写起来总觉得不得劲，所以想着学一学后端。

关于为什么选择GO：第一，Java目前来说实在是太卷了。第二，之前在面试过程中，也是问过一些面试官，得到的回复也是推荐学习GO。第三，了解过后发现越来越多的大厂在使用Go。

所以，最终决定给自己定个目标：学好GO并搭配前端自己写个小项目！接下来，会陆陆续续写一些GO的学习笔记。开始吧～

# 概览

Go，又称Golang，是由Google开发的开源编程语言。在语法上与C语言相近，但增加了一些功能，如：垃圾回收机制、内存安全等。Go语言的应用场景广泛，不仅可用于网络编程和系统编程，还适用于并发编程和分布式编程。Go语言的推出旨在降低代码的复杂性，同时不损失应用程序的性能。它具有“部署简单、并发性好、语言设计良好、执行性能好”等优势。（摘自网络）

## Go的特性
1. **自动垃圾回收**：所有的内存分配动作都会被在运行时记录，同时任何对该内存的使用也都会被记录，然后垃圾回收器会对所有已经分配的内存进行跟踪监测，一旦发现有些内存已经不再被任何人使用，就阶段性地回收这些没人用的内存。
2. **丰富的内置类型**：map、slice（数组切片）；不需要再费事添加依赖，减少工作量，简洁代码。
3. **函数多返回值**
4. **错误处理**：三个关键字 defer、panic、recover；可以大量减少代码量，让开发者也无需仅仅为了程序安全性而添加大量一层套一层的try-catch语句。
5. **匿名函数和闭包**
6. **类型和接口**：不支持继承和重载，而只是支持了最基本的类型组合功能。
7. **并发编程**：引入协程（goroutine）概念。当一个协程阻塞的时候，调度器就会自动把其他协程安排到另外的线程中去执行，从而实现了程序无等待并行化运行。实现了CSP（通信顺序进程，Communicating Sequential Process）模型来作为goroutine间的推荐通信方式。
8. **反射**：通过反射，你可以获取对象类型的详细信息，并可动态操作对象。常用场景是做对象的序列化。
9. **语言交互性**


# 环境准备
## 安装Go
通过[Go官网](https://golang.google.cn/dl/)下载，根据需要下载相应安装包。mac也可以通过brew安装`brew install go`
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/542d3cd0e26d4cc0b9a25ce2c2596f7e~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=2402&h=944&s=230309&e=png&b=fdfdfd)

通过`go version`检查是否安装成功。
```
$ go version
go version go1.22.2 darwin/arm64
```

## VSCode 配置
- 打开扩展，搜索GO插件并安装

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d967c9106f3a4b7385079eb58b7f946f~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=954&h=618&s=106181&e=png&b=1c1c1c)

- 安装插件Tools
  -  `Command + Shift + P` 打开设置，输入`Go install`
  -  选择`Go: Install/Update Tools`

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8f8bf9a949f7474c919b253446c58a1a~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1218&h=402&s=81115&e=png&b=232323)
  -  全部勾选，点击确认。
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c361da4bf7cf486ead53613a9c25a4e9~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1264&h=470&s=101690&e=png&b=242424)

> PS： 如果出现大量的下载失败，那么打开终端，输入：\
> `go env -w GO111MODULE=on`\
> `go env -w GOPROXY=https://proxy.golang.com.cn,direct`\
> 然后重启Vscode

# 基础学习
## 初步学习
### 文件结构
写一个helloworld
```go
/* 声明包，可执行程序的包名通常为main */
package main 
/* 导入包，按字母排序 */
import "fmt"
/* 函数定义，main函数是程序入口点。*/
func main() {
    fmt.Println("Hello World!")
}
```

### `:=` 操作符
又称短变量声明操作符，用于声明变量并初始化。在函数内部使用，用于创建新局部变量，且隐式赋予对应类型。

语法：`variable := expression`

*需要注意，声明变量并初始化时，左侧变量不能是被声明过的，否则会编译报错。*

### 程序编译
- 直接编译运行：`go run hello.go`
- 生成编译结果：`go build hello.go`

### 注意项
1. 不得包含没有用到的包，否则会编译报错
2. 强制左花括号`{`，另起一行会导致编译错误
3. 不要求开发者在每个语句后面加上分号表示语句结束

待补充...

## 变量

### 声明
纯粹的变量声明，引入关键字`var`, 类型信息放在变量后。如果需要初始化，则`var`关键字则非必要，同时也可省略类型，编译器可以自动推导出类型。
```go
// 单个变量声明
var a int
var b string
// 多个变量合并声明
var (
    a int
    b string
)
// 变量声明并初始化
var value1 int = 10 // 写法一
var value2 = 20 // 写法二：省略类型
value3 := 30 // 写法三【推荐】
```
### 赋值
普通变量赋值与其他语言一致。但go引入了`多重赋值`功能，显著减少代码行数。
```go
// 普通变量声明赋值：
var a int 
a = 123
// 多重赋值
i, j = j, i
// 等同于如下：其他没有多重赋值功能语言，需要引入一个中间变量。
// t = i; i = j; j = t;
```

### 匿名变量
传统强类型语言编程中，调用函数取值时，因为返回多个值而定义一些无用的变量。go通过`匿名变量`可以避免。🌰：
```go
func GetUserInfo() (userName, nickName, tel string) {
    return "Lumi", "麦浪冒险家", "12345678988"
}
// 获取手机号码
_, _, tel = GetUserInfo()
// 获取userName
userName, _, _ = GetUserInfo()
// 获取nickName
_, nick, _ = GetUserInfo()
```

## 常量
`字面常量`指程序中硬编码的常量，是无类型的，只要这个常量在相应类型的值域范围内，就可以作为该类型的常量。

### 定义
用关键字`const`定义，可以限定类型，但非必需。其右值也可以是一个**编译期运算**的常量表达式，但不能是任何需要运行期才能得到结果的表达式。
```go
// 🌰
const a float64 = 3.14159265358979323846 // 完整写法
const b = 10.0 // 无类型浮动常数
const (
    c int = 100
    d = "Gi"
) // 多个常量声明
const x, y, z int = 3,4,5 // 多重赋值
const val1, val2, val3 = "hello", "world", 0 // 多重无类型赋值
const make = val3 << 3 // 常量表达式
```

### 预定义常量
go预定的常量： `true`,`false`,`iota`

`iota`：一个可被编译器修改的常量，在每个`const`关键字出现时重置为0，下一个`const`出现前，每出现一次`iota`，代表数字+1
```go
// 🌰
const ( // iota被重设为0
    c0 = iota // c0 == 0
    c1 = iota // c1 == 1
    c2 = iota // c2 == 2
    // 可以简写为：
    // c0 = iota
    // c1
    // c2 
)
```

### 枚举
go不支持其他语言常见的`enum`关键字。举例常规枚举：以大写字母开头的常量在包外可见。
```go
const (
    Sunday = iota
    Monday
    Tuesday
    Wednesday
    Thursday
    Friday
    Saturday
    numberOfDays // 这个常量没有导出，为包内私有
)
```


## 类型
### 基础类型
#### 整型
分`有符号整型`和`无符号整型`
| 类型        | 长度（字节） | 值范围                                                 |
| ----------- | ------------ | ------------------------------------------------------ |
| int8        | 1            | -128～127                                              |
| int16       | 2            | -32 768 ~ 32 767                                       |
| int32       | 4            | -2 147 483 648 ~ 2 147 483 647                         |
| int64       | 8            | -9 223 372 036 854 775 808 ~ 9 223 372 036 854 775 807 |
| int         | 平台相关     | 平台相关                                               |
| uint8(byte) | 1            | 0 ~ 255                                                |
| uint16      | 2            | 0 ~ 65 535                                             |
| uint32      | 4            | 0 ~ 4 294 967 295                                      |
| uint64      | 8            | 0 ~ 18 446 744 073 709 551 615                         |
| uint        | 平台相关     | 平台相关                                               |
| uintptr     | 同指针       | 在32位平台下为4字节，64位平台下为8字节                 |
> 需要注意⚠️：\
> 编译器不会自动做类型转换，如 int 和 int32比较，需要通过强制类型转换。\
> 强制类型转换需要注意精度丢失或值溢出问题。

**运算**

加减乘除、取余、比较运算 与多数其他语言一致。Go语言还支持位运算：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a2f4053ccdb4ecbbca560e6a64bce88~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=770&h=422&s=26411&e=png&b=ffffff)

#### 浮点型
用于包含小数点的数据。定义了两个类型 `float32` 和 `float64`。编译器自动推导的float，会被自动设为float64。两种类型的数比较需要强转类型。

关于浮点数的两数比较：因为浮点数在计算机中的表示是不精确的，存在舍入误差。所以通过设置一个容差值，计算两数差的绝对值是否小于容差值。结果为true的话，表示两数接近相等。
```go
func IsEqual(f1, f2, p float64) bool {
	return math.Abs(f1-f2) < p
}
```
#### 布尔
关键字为 `bool`，可赋值`true` 和`false`。不接受其他类型赋值，不支持自动或强制类型转换。
```go
// 🌰
var b bool
// 错误写法❌
// b = 1
// b = bool(1)

// 正确写法✔️
b = true
b = (1 != 0)
c := (1 != 2)
```
#### 复数类型
定义了两种类型：`complex64`、`complex128`。由两个实数组成，一个表示实部、一个表示虚部。自动推导为`complex128`类型。

可以通过`real(z)`获取复数实部，用`imag(z)`获取复数虚部。
```go
var value1 complex64 // 由2个float32构成的复数类型
value1 = 3.2 + 12i
value2 := 3.2 + 12i // value2是complex128类型
value3 := complex(3.2, 12) // value3结果同 value2
```

#### 字符串
`string`，定义和初始化都非常简单。
- 字符串内容不能在初始化后被改变
- 用`len()`获取字符串长度。
- `+` 字符串连接
- `s[i]`：取字符 
字符串遍历有两种方法：以字节数组方式遍历 、 以Unicode字符遍历
```go
// 方法一：
str := "Hello, world"
n := len(str)
for i := 0; i< n; i++ {
    ch := str[i]
    fmt.Println(i, ch)
}
//输出结果ch类型为byte

// 方法二：
str := "hello, world"
for i, ch := range str {
    fmt.Println(i. ch)
}
// 输出结果ch类型为rune
// 早期go用int类型表示Unicode

// Go仅支持UTF-8和Unicode编码。
```

### 复合类型
#### 数组
数组就是指一系列同一类型数据的集合。常规声明：
```go
[32]byte // 长度为32的数组，每个元素为一个字节
[2*N] struct { x, y int32 } // 复杂类型数组
[1000]*float64 // 指针数组
[3][5]int // 二维数组
[2][2][2]float64 // 等同于[2]([2]([2]float64))
```
数组长度在定义后不可更改，声明时长度可以是一个常量/常量表达式。用`len(arr)`获取数组个数/长度。数组遍历：
```go
// 方法一
for i := 0; i < len(arr); i++ {
    fmt.println(arr[i])
}
// 方法二：
for i, v := range arr {
    fmt.Println(arr[i], v)
}
```
数组是一个值类型，赋值和作参传递时会产生一次复制动作。作参时，函数内操作的数组只是所传入数组的副本。那么，如何才能在函数内操作外部的数据结构呢？我们可以使用`数组切片slice`实现。

#### 数组切片
数组切片弥补了数组的不足。数组切片拥有自己的数据结构，可以抽象为三部分：
- 指向原生数组的指针。
- 数组切片中的元素个数。
- 数组切片已分配的存储空间。

**基于数组创建**：可以只使用数组的一部分元素或全部元素；也可以创建比基于数组更大的数组切片。
```go
// 使用arry[first:last]方法
myArray := [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
mySlice1 := myArray[:]  // 全部
mySlice2 := myArray[:5] // 前5个
mySlice3 := myArray[5:] // 第5个开始
```
**基于数组切片创建**：
```go
// 使用arry[first:last]方法
myArray := [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
mySlice1 := myArray[:]  // 全部
mySlice2 := mySlice1[:5] // 前5个
```
**直接创建**： 通过内置函数`make()`创建，Go底层还会有一个匿名数组被创建。
```go
mySlice1 := make([]int, 5) // 初始元素个数为5的数组切片，元素初始值为0
mySlice2 := make([]int, 5, 10) // 初始元素个数为5的数组切片，元素初始值为0，并预留10个元素的存储空间
mySlice3 := []int{1, 2, 3, 4, 5} // 创建并初始化包含5个元素的数组切片
```
**支持内置函数**
- `len()` 返回当前存储元素个数
- `cap()` 返回数组切片分配空间大小
- `append(arr, 元素1，...)` 新增元素：可以按自己需求添加若干元素，也可以是一个数组切片（如果第二参数是数组切片则必须加上省略号...，相当于打散解构后传入)
- `copy()`: 将一个数组切片复制到另一个数组切片，如果长度不一样大，按小的数组切片元素个数复制。

**元素遍历**：类似数组元素遍历

#### map
C++/Java中，`map`一般都以库的方式提供，而Go中，`map`不需要引入任何库。`map`是一堆键值对的未排序集合。

**声明**：`var myMap map[keyType]valueType`, keyType是 键类型，valueType是值类型\
**创建**：`make(map[keyType] valueType [, size])`\
**赋值**：`map[key] = value`\
**删除**：`delete(map, key)`\
**查找**：
```go
//判断是否成功找到特定的键，不需要检查取到的值是否为nil，只需查看第二个返回值ok，这让表意清晰很多。
value, ok := myMap["1234"]
if ok { // 找到了
    // 处理找到的value
}
```