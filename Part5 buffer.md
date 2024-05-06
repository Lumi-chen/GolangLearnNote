---
theme: smartblue
---
# buffer
缓冲区 用于临时存储数据，以便更有效地进行I/O操作或其他数据处理任务。提供了很多有用的方法，实现了`io.Reader`、`io.Writer`、`io.ByteWriter`、`io.ReaderFrom`、`io.WriterTo`和`io.Stringer`接口。

**创建**：
1. `NewBuffer(buf []byte)`，`buf`可选，用于指定缓冲区初识容量，默认为64字节的缓冲区 
```go
buf := bytes.NewBufferString("hello world")
fmt.Println(buf.String()) // 输出：hello world
```
2. `bytes.Buffer`

```go
var buf bytes.Buffer
// 向缓冲区写入数据
buf.WriteString("Hello, ")
buf.WriteString("World!")
fmt.Println(buf.String()) // 输出：Hello, World!
```
**写入**：提供了多种写入方法，`Write(字节切片)`、`WriteString`、`WriteByte`、`WriteRune`\
**读取**：提供了多种读取方法，`Read(字节切片)`、`ReadString`、`ReadByte`、`ReadRune`\
**转换**：将缓冲区内容转为字符切片`Bytes()`；转为字符串`String()`\
**截取**：`Truncate(n int)`，如果缓冲区长度超过n，则从尾部开始截取，只保留前n个字节；如果不足n，则不做操作， 报错`panic: bytes.Buffer: truncation out of range`。\
**扩容**：`Grow(n int)`，n 小于等于缓冲区的剩余容量，则不做任何操作。否则，会将缓冲区的容量扩大到原来的 2 倍或者加上 n，取两者中的较大值。*扩容后的缓冲区不保证是连续的。*\
**重置**：`Rest()`，将缓冲区清空并重置为初识状态。\
**获取缓冲区未读部分长度和缓存区的总容量**：`Len()`和`Cap()`

以下举🌰：
```go
var buf bytes.Buffer
// 向缓冲区写入数据
buf.WriteString("Hello, ")
buf.Write([]byte("My name is Lily!"))、
// 转换
fmt.Print(buf.String(), "\n")
fmt.Print(buf.Bytes(), "\n") // 输出 [72 101 108 108 111 44 32 77 121 32 110 97 109 101 32 105 115 32 76 105 108 121 33]
// 长度容量
fmt.Print(buf.Len(), "\n") // 输出 23
fmt.Print(buf.Cap(), "\n") // 输出 64
// 读取
data := make([]byte, 50)
n, _ := buf.Read(data)
fmt.Print(string(data[:n]), "\n")
fmt.Print(buf.Len(), "\n") // 输出 0
fmt.Print(buf.Cap(), "\n") // 输出 64；读取后容量不变

// 截取扩容
buf.WriteString("Hello, ")
buf.Write([]byte("My name is Lily!"))
buf.Truncate(5)
fmt.Print(buf.String(), ";", buf.Len(), "\n") // 输出 Hello; 5
fmt.Print("before:", buf.Cap(), "\n") // 输出 before: 64
buf.Grow(80)
fmt.Print("after:", buf.Cap(), "\n") // 输出 after: 112
buf.Reset()
fmt.Print("Reset：", buf.String()) // 输出 Reset：

```
**序列化和反序列化**：由于 bytes.Buffer 类型支持读写操作，它可以用于序列化和反序列化结构体、JSON、XML 等数据格式。使用 encoding包（`encoding/json`、`encoding/xml`、`encoding/gob` 等）进行操作。举🌰JSON序列化和反序列化：
```go
type Person struct {  
    Name string `json:"name"`  
    Age  int    `json:"age"`  
}
// 创建Person实例
p := Person{Name: "lily", Age: 18}
// 创建缓冲区存储序列化后数据
var buf bytes.Buffer
fmt.Println("before p:", p) // before p: {lily, 18}
// 序列化Person实例到缓冲区
json.NewEncoder(&buf).Encode(p)
fmt.Println("after buf:", buf.String()) // after buf: {"name":"lily","age":18}
// 创建新实例用于反序列
var p2 Person
// 反序列
json.NewDecoder(&buf).Decode(&p2)
fmt.Println("after p2:", p2) // after p2: {lily 18}
```
# I/O
Go提供了io包，是对原始I/O操作的一系列接口。
## Reader
用于读取数据。只要实现了 `Read(p []byte)` ，那它就是一个读取器
```go
type Reader interface {
        Read(p []byte) (n int, err error)
}
```
## Writer
用于写入数据。只要实现了 `Write(p []byte)` ，那它就是一个编写器
```go
type Writer interface {
    //Write() 方法有两个返回值，一个是写入到目标资源的字节数，一个是发生错误时的错误。
    Write(p []byte) (n int, err error)
}
```

## 文件相关操作
**创建新文件**： 返回一个文件对象
1. 提供文件名创建：`Create(name string) (file *File, err Error)`
2. 提供文件描述创建：`NewFile(fd unitptr, name string) *File`

**打开**：
1. 只读：`Open(name string) (file *File, err Error)`
2. 可选权限打开：`OpenFile(name string, flag int, perm uint32) (file *File, err Error)`；flag 是打开方式（只读/读写..）；perm 是权限

| 打开模式    | 说明                                         | 权限          | 说明                                                       |
| ----------- | -------------------------------------------- | ------------- | ---------------------------------------------------------- |
| os.O_RDONLY | 只读                                         | 0644          | 文件拥有者有读写权限，组和其他用户只有读权限               |
| os.O_WRONLY | 只写                                         | 0755          | 文件拥有者有读、写和执行权限，组和其他用户有读和执行权限。 |
| os.O_RDWR   | 读写                                         | os.ModeDir    | 表示这是一个目录。                                         |
| os.O_CREATE | 文件不存在，则创建它                         | os.ModeAppend | 表示文件只能以追加模式打开。                               |
| os.O_TRUNC  | 文件已存在并且为普通文件，则将其长度截断为零 | os.FileMode   | 表示文件权限的类型。你可以使用位运算符来组合多个权限。     |
| os.O_APPEND | 每次写入都添加到文件的末尾                   |               |                                                            |
| os.O_EXCL   | 文件已存在，则返回一个错误                   |               |                                                            |


**写入**：
1. 写入byte类型：`func (file *File) Write(b []byte) (n int, err Error)`
2. 指定位置写入：`func (file *File) WriteAt(b []byte, off int64) (n int, err Error)`
3. 写入String类型：`func (file *File) WriteString(s string) (ret int, err Error)`


**读取**：
1. 全部读取：`func (file *File) Read(b []byte) (n int, err Error)`
2. 指定读取：`func (file *File) ReadAt(b []byte, off int64) (n int, err Error)`

**删除**：`func Remove(name string) Error`\
**关闭**：`Close()`

```go
// 创建
file, err := os.Create("test.txt")
if err != nil {
    log.Fatal(err)
}
defer file.Close()
// 写入
file.WriteString("Hello World! My name is Lily! \n")
file.Write([]byte("I am 18"))
file.WriteAt([]byte("\n"), 12)
// 读取
b := make([]byte, 12)
n, _ := file.ReadAt(b, 12)
fmt.Println(n, string(b))
```
**重置文件光标位置**：使用`Seek`方法，指定一个偏移量（从文件开始处计算），以及一个基准位置（可以是文件的开始、当前位置或结束），然后移动文件的光标到那个位置。\
重置到文件开头的话，以0为偏移量，以`io.SeekStart`为基准位置。
```go
file, err := os.Open("test.txt")
if err != nil {
    log.Fatal(err)
}
defer file.Close()
// 读取一些数据（假设这里已经读取了一些）  
// ...  
// 重置文件光标到开始位置  
_, err = file.Seek(0, io.SeekStart)  
if err != nil {  
    log.Fatal(err)  
}
// 现在可以从文件的开始重新读取数据了  
// ..
```