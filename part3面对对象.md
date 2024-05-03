---
theme: smartblue
---
## 定义新类型
go语言中，使用`type`加上类型可以定一个新的类型，新定义的类型与其基本类型没有本质不同，但我们可以为新类型增加新方法。
```go
🌰：
type myInt int
// 面对对象写法
func (a myInt) Less(b myInt) bool {
    return a < b
}
// 面对过程写法
func myIntLess(a myInt, b myInt) bool {
    return a < b
}
func main() {
    var a myInt = 1
    if a.Less(2) {
        fmt.Println(a, "Less 2")
    }
    if myIntLess(a, 2) {
        fmt.Println(a, "Less 2")
    }
}
```

## 匿名组合
Go语言也提供了继承，但是采用了组合的文法，所以我们将其称为匿名组合。
🌰:
首先定义一个父类（父结构体），再定义一个类，“继承“自父类，并改写其成员函数。
```go
type Father struct {
	Name string
}

func (f *Father) goTo() {
	fmt.Println(f.Name + " go to work")
}

type Child struct {
	Father
	Name string
}

func (c *Child) goTo() {
	c.Father.goTo()
	fmt.Println(c.Name + " go to school")
}

func main() {
	f := Father{Name: "KK"}
	f.goTo()
	c := Child{Father: f, Name: "cc"}
	c.goTo()
}
```

**注意**

> go中没有`private`、`protected`、`public`关键字，如果要使某变量/函数对其他包可见，需要将其以**大写字母**开头。

## 类型断言 type assertion
类型断言是go中重要的特性，可以判断一个接口的实际类型是否是预期的类型。

**基本语法**：`变量a, ok = 变量b.(类型)`\
判断并返回是否成功，成功则返回对应的类型值， ok为true；失败则a为对应断言类型的默认值，ok为false。一般搭配`if`条件判断使用。

**直接断言**：`变量a := 变量b.(类型)`\
直接返回对应类型的值，成功则 a == b，失败则`panic`

**switch断言**：需要一次性断言多种类型，用`变量.(type)`作为判断条件。

```go
// 基本语法
var i interface{} = "hello"
if s, ok := i.(string); ok {
    fmt.Println(s)
} else {
    fmt.Println(ok)
}
if b, ok := i.(bool); ok {
    fmt.Println(b)
} else {
    fmt.Println(ok)
}
// 直接断言
f := i.(float64)
fmt.Println(f) // panic: interface conversion: interface {} is string, not float64

// switch断言
witch v := i.(type) {
    case string:
        fmt.Println("String:", v)
    case float64:
        fmt.Println("Float64:", v)
    default:
        fmt.Println("Unknown type")
}
```
# 接口

Go接口特性：
1. 隐式实现：不需要显式声明实现了某接口。
2. 非侵入性：可以在不修改任何现有代码的情况下，为已有的类型添加新的接口实现。
3. 组合：接口可以组合，即一个接口可以嵌入（或继承）另一个接口的所有方法。
4. 无方法签名：不包含方法的实现，只定义了方法签名（即方法名、参数列表和返回类型）。
5. 类型断言：提供了类型断言（type assertion）机制，用于在运行时检查接口值所持有的具体类型，并获取该类型值。
6. 空接口：空接口可以表示任意类型的值，因此可以作为函数参数或返回值的通用类型。

go中，一个类只要实现了接口所要求的**所有函数**，就说该类实现了该接口。
## 定义实现
**定义接口**：`type Name interface`
**实现**：定义一个类，实现某接口某方法。只有类实现了某接口才可以被赋值给该接口类型的变量，并通过调用该变量调用接口中方法。
```go
🌰：
// 定义一个接口
type Shape interface {
    Area() float64
}
// 定义一个矩形类
type Rectangle struct {
    width, height float64
}
// 矩形实现Shape接口Area方法
func(r Rectangle) Area() float64 {
    return r.width * r.height
}

func main() {
    // 创建Shape类型变量
    var s Shape
    // 赋值矩形
    s = Rectangle{width: 3, height: 4}
    // 调用
    fmt.Println(s.Area()) // 12
}
```
## 接口赋值
接口的赋值除了上面说的将对象实例赋值给接口还可以是将一个接口赋值给另一个接口。在go中，只要两个接口拥有相同的方法列表，那么他们就是等同的，可以相互赋值。
```go
// 一个接口在one包
package one
type ReadWriter interface {
    Read(buf []byte) (n int, err error)
    Write(buf []byte) (n int, err error)
}

// 一个接口在two包
package two
type IStream interface {
    Write(buf []byte) (n int, err error)
    Read(buf []byte) (n int, err error)
}
// 两者实际上并无区别，实现one.ReadWriter接口的类均实现了two.IStream。
```
接口赋值不要求两个接口等价。如果接口A的方法列表是接口B的子集，那么B可以赋值给A，但A不可以赋值给B。

## 接口组合
接口组合是类型匿名组合的一个特定场景。举例：ReadWriter接口包含Writer接口和Read接口能做的，写法等同如下：
```go
type ReadWriter inteface {
    Read
    Writer
}
等同于
type ReadWriter interface {
    Read(p []byte) (n int, e error)
    Write(p []byte) (n int, e error)
}
```

## 空接口

go中任何对象实例都满足空接口`interface{}`，其可以表示任意类型的值。当函数可以接受任意对象实例时，可以将其声明为`interface{}`。

一些使用场景：
```go
// 1.作为函数参数
func printAnything(v interface{}) {
    fmt.Println(v)  
}
// 2.作为map的值类型
myMap := make(map[string]interface{})  
myMap["one"] = 1  
myMap["two"] = "two"  
myMap["three"] = true  

// 3.作为通道的元素类型
ch := make(chan interface{})  
go func() {  
    ch <- "hello"  
    time.Sleep(time.Second)  
    ch <- 42  
}()
```