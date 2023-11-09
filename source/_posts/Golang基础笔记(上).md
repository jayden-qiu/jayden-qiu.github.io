---
title: Golang基础笔记(上)
date: 2021-11-16 13:12:00
tags:
categories:
- Golang
description:
---

# 一、环境安装
## 下载地址
[https://studygolang.com/dl](https://studygolang.com/dl)

## 选择合适平台后安装，后验证是否安装完成
```shell
go version
```
<!--more-->

## 设置 go module 为开启状态
```shell
go env -w GO111MODULE=on
```

## 设置 go package 国内镜像地址
```shell
go env -w GOPROXY=https://goproxy.cn,direct
```

# 二、变量定义
## 例子
```go
// 正常定义（自动类型推断）
var aa = 1
var bb = "kan"

// 简略定义（自动类型推断）
var (
    cc = 2
    dd = 3
)

// 完整定义
var a bool = true
var b string = "ha"
var c int = 123

// 缩写定义（只能函数内）
g := true
h := "ha"
i := 123

// 联合定义
var j,k,l = true,123,"xi"
```

## 打印相关 fmt.Printf
| code   | desc         |
| ------ | ------------ |
| %v     | 原样输出     |
| %T     | 打印类型     |
| %t     | bool类型     |
| %s     | 字符串       |
| %f     | 浮点         |
| %d     | 10进制的整数 |
| %b     | 2进制的整数  |
| %o     | 8进制        |
| %x、%X | 16进制       |
| %x     | 0-9，a-f     |
| %X     | 0-9，A-F     |
| %c     | 打印字符     |
| %p     | 打印地址     |

# 三、内置变量类型
## bool

## string

## 数字型
| type     | desc                                                 |
| -------- | ---------------------------------------------------- |
| (u)int   | (无符号)不规定长度，在32位系统就32位，64位系统就64位 |
| (u)int8  | (无符号)规定长度8                                    |
| (u)int16 | (无符号)规定长度16                                   |
| (u)int32 | (无符号)规定长度32                                   |
| (u)int64 | (无符号)规定长度64                                   |
| unitptr  | 指针，长度跟操作系统相关                             |

## 字符型
| type | desc            |
| ---- | --------------- |
| byte | 长度8位（8bit） |
| rune | 长度32位        |
*ps: rune 相当于 go 的 char*

## 浮点型
| type       | desc        |
| ---------- | ----------- |
| float32    |             |
| float64    |             |
| complex64  | 64位的复数  |
| complex128 | 128位的复数 |

# 四、常量与枚举
```go
// 常量定义
const a int = 123
const b,c = "str",false
const (
	d int = 1
	e string = "xi"
)

// const 数值可作为各种类型使用
const f,g = 3,4
var h int = int(math.Sqrt(f*f + g*g))

// 枚举类型
const (
	aa = 0
	bb = 1
	cc = 2
	dd = 3
)

// 枚举类型 iota
const (
	a1 = iota
	b1
	c1
	d1
)
// iota 后面的值都会自增1

// 枚举类型iota中间跳过定义
const (
	a2 = iota
	_
	c2
	d2
)
fmt.Println(a2,c2,d2) // 0 2 3

// iota 还可以参与运算
const (
	ap = 1 << (10 * iota)
	bp
	cp
)
```

# 五、条件语句
## 完整版switch
```go
func eval(a,b int,op string) int {
    var result int

    switch op {
    case "+":
        result = a + b
    case "-":
        result = a - b
    case "*":
        result = a * b
    case "/":
        result = a / b
    default:
        panic("错误运算符" + op)
    }

    return result
}
```
## 简洁版switch
```go
func grade (score int) string {
    g := ""

    switch {
    case score < 0 || score > 100 :
        panic(fmt.Sprintf("分数错了")) // panic 用与手动抛出错误
    case score < 60:
        g = "f"
    case score < 80:
        g = "c"
    case score < 90:
        g = "b"
    case score <= 100:
        g = "a"
    }

    return g
}
```

## switch 的 fallthrough 语句
**Go里面switch默认相当于每个case最后带有break，匹配成功后不会自动向下执行其他case，而是跳出整个switch, 但是可以使用fallthrough强制执行后面的case代码**
```go
switch "a" {
case "a":
    fmt.Println("a")
    fallthrough
case "b":
    fmt.Println("b")
case "c":
    fmt.Println("c")
default:
}
```

## 条件语句
```go
filename := "a.txt"
contents, err := ioutil.ReadFile(filename)

// 正常条件语句
if err != nil {
	fmt.Println(err)
} else {
	fmt.Printf("%s\n",contents)
}

// 先执行一条语句，再进行判断
if contents1, err1 := ioutil.ReadFile(filename); err != nil {
	fmt.Println(err1)
} else {
	fmt.Printf("%s\n",contents1)
}

fmt.Printf("%s",contents1) // 报错！ if语句里面定义的contents1、err1，出了这个块就访问不到了
```

# 六、循环
```go
// 一般 for 循环
sum := 0
for i := 1; i <= 100; i++ {
	sum += 1
}
fmt.Println(sum)


// 省略起始条件
n := 8 //将8转二进制
result := ""
for ; n > 0; n /= 2 {
	temp := n % 2
	result = strconv.Itoa(temp) + result
}
fmt.Println(result)


// 只有结束条件
file, err := os.Open("a.txt")
if err != nil {
	panic(err)
}
scanner := bufio.NewScanner(file)

for scanner.Scan() {
	fmt.Println(scanner.Text()) // 将文本里面内容一行行打印出来
}


// 死循环
for {
	fmt.Println("abc")
}
```

# 七、函数
```go
// 一个返回值函数
func testfn1(a int,b int) int {
    return a + b
}

// 多个返回值函数
func testfn2(a int, b int)(c int, d string) {
    c = a + b
    d = "haha"
    return c,d
}

// 还可简写成
func testfn3 (a int, b int) (int, string) {
    return a + b, "haha"
}

// 可以新增报错参数
func testfn4 (a int, b int) (int,error) {
    if b > 4 {
        return 0,fmt.Errorf("错了%d",b)
    } else {
        return b, nil
    }
}

// 函数可以作为参数
func testfn5(op func (int, int) int, a, b int) int {
    p := reflect.ValueOf(op).Pointer()
    opName := runtime.FuncForPC(p).Name() // 获取到函数名字
    fmt.Println(opName) // main.main.func1 包名.入口函数名.函数名
    return op(a,b)
}

// 函数的可变参数列表
func sum(numbers ...int) int {
    s := 0
    for i := range numbers {
        s += numbers[i]
    }
    return s
}
```

# 八、指针
```go
// 指针 (两个值交换)
func swap(a, b *int) { // *int表示这两个整型是指针地址
    *b, *a = *a, *b
}

// 普通的两数值交换
func normalSwap(c,d int) (int, int) {
    return d,c
}

func main() {
    // go里面只有值传递，没有引用传递，但是用指针可以实现引用传递的效果

    // 指针 (两个值交换)
    a := 1
    b := 2
    swap(&a, &b) // 将a,b的指针地址传入
    fmt.Println(a,b) // 2 1

    // 普通的两个值交换
    c := 1
    d := 2
    c,d = normalSwap(c,d)
    fmt.Println(c,d) // 2 1
}
```

# 九、数组
```go
// 数组作为参数传递，长度一定要达到要求
func acceptArr(a [5]int) {
    fmt.Println("测试数组长度要求",a)
}

// go 是值传递，函数里改变数组并不影响原来的值
func changeArr(a [5]int) {
    a[0] = 999
}

// 指针方式改变数组的值
func changeArr1(a *[5]int) {
    a[0] = 999
}

func main() {

    // 一维数组
    var arr1 [5]int // int类型的数组，长度为5
    arr2 := [3]int{1,2,3} // int类型数组，长度为3，值为1，2，3
    arr3 := [...]int{1,1,1,1,1} // int类型数组，长度不固定，值为....
    fmt.Println(arr1)
    fmt.Println(arr2)
    fmt.Println(arr3)

    // 二维数组
    var grid [2][3]bool // bool类型数组，2维数组，每个数组有3个元素
    fmt.Println(grid)

    // 正常数组遍历
    for i := 0; i < len(arr3); i++ {
        fmt.Println(arr3[i])
    }

    // range 方法遍历
    for i,v := range arr2 { // i是index，v是value
        fmt.Println(i,v)
    }

    // range 方法遍历（省略index）
    for _,v := range arr2 {
        fmt.Println(v)
    }

    acceptArr(arr3) // 正常
    //acceptArr(arr2) // 错误，长度达不到要求

    // 尝试改变数组值
    changeArr(arr3)
    fmt.Println("尝试改变数组值",arr3) // 失败

    // 指针方式改变数组
    changeArr1(&arr3)
    fmt.Println("尝试改变数组值",arr3) // 成功
}
```
# 十、切片
```go
// 通过slice，更新array
func updateSlice(a []int) {
    a[0] = 333
}

func main() {

    // Slice(切片)：本身没有数据，是对底层array的一个view。 []里面带有: 的都是切片
    arr := [...]int{0,1,2,3,4,5,6,7}

    // 从arr下标1开始，往后取到下标4
    arr1 := arr[1:5]
    fmt.Println(arr1) // [1,2,3,4]

    // 从头开始取到下标为5
    arr2 := arr[:6]
    fmt.Println(arr2) // [0 1 2 3 4 5]

    // 从下标3开始，取到最后
    arr3 := arr[3:]
    fmt.Println(arr3) // [3,4,5,6,7]

    // 打印出整个数组的view
    arr4 := arr[:]
    fmt.Println(arr4)

    // 从 slice 里面再取 slice
    arr5 := arr[2:5] // [2,3,4]
    arr6 := arr5[1:3] // [3,4]
    fmt.Println(arr6)

    // slice本身没有数据，改变slice会改变原来的array
    updateSlice(arr6)
    fmt.Println(arr)

    // Reslice ：即对slice的重复赋值
    s := arr[2:6]
    s = s[:3]
    s = s[1:]
    s = arr[:]
    fmt.Println(s)

    // slice 扩展：从slice里面再取slice，要是下标超出就看cap，只要没超过capacity就正常
    arr7 := arr6[0:5] // [3,4,5,6,7]
    fmt.Println(arr7)

    // 容量（capacity）：是指slice的起始下标，到原array的结束位置
    arr = [...]int{7,6,5,4,3,2,1,0}
    fmt.Println("原数组：",arr,"capacity:",cap(arr)) // 原数组： [7 6 5 4 3 2 1 0] capacity: 8

    s1 := arr[4:8]
    fmt.Println("原数组：",s1,"capacity:",cap(s1)) // 原数组： [3 2 1 0] capacity: 4

    s2 := s1[2:]
    fmt.Println("原数组：",s2,"capacity:",cap(s2)) // 原数组： [1 0] capacity: 2
}
```

# 十一、切片操作
```go
// slice 添加元素
arr := [...]int{1,2,3}

// 因为arr定义时已经固定好 length 跟 capacity，所以添加元素只能另起一个数组
arr1 := append(arr[:],4) // 第一个参数是slice，第二个是新增元素
arr2 := append(arr1,5)
// 必须接收 append 的返回值
fmt.Println(arr1)
fmt.Println(arr2)

// 简单实例
var s []int // 初始化一个slice，此时 s == nil

for i := 1; i <= 100; i++ {
	s = append(s, 2*i+1)
}
fmt.Println(s)

// 简单创建 slice
s1 := []int{1,2,3}
fmt.Println(s1)

// make 方法创建 slice
s2 := make([]int,10) //新建slice，长度10，容量默认等于长度
s3 := make([]int,10,32) //新建slice，长度10，容量32
fmt.Println("元素：",s2,"长度：",len(s2),"容量：",cap(s2))
fmt.Println("元素：",s3,"长度：",len(s3),"容量：",cap(s3))

// copy 方法复制 slice
copy(s2,s1) // 将s1 复制到 s2里面去
fmt.Println(s2) // [1 2 3 0 0 0 0 0 0 0]

// 删除 slice 里面的元素（如将2删掉）
s2 = append(s2[:1],s2[2:]...)
fmt.Println(s2)

// 删除头部
s2 = s2[1:]

// 删除尾部
s2 = s2[:len(s2)-1]
fmt.Println(s2)
```

# 十二、Map
```go
// 直接定义 map
m := map[string]int {
	"age":18,
	"name":11,
	"otherName":12,
}
fmt.Println(m)

// make 定义 map
m1 := make(map[string]int) // m1 == empty map
fmt.Println(m1)

// var 定义 map
var m2 map[string]int // m2 == nil
fmt.Println(m2)

// 遍历 map (不保证顺序)
for i,v := range m {
	fmt.Println(i,v) // i是key，v是value
}

// 获取map里面键值
fmt.Println(m["age"])
fmt.Println(m["ages"]) // 若key不存在就会返回默认初始值

// 判断map里面key存不存在
if value,ok := m["age"];ok {
	fmt.Println("存在，值为：",value)
}

// 删除map里面的key
delete(m,"age")
fmt.Println(m)
```

# 十三、结构体和方法
```go
// 新建结构体类型
type treeType struct {
    name string
    age int
    address string
}

// 给结构体新增方法
func (node treeType) echoName() { // 前面括号表示为哪个结构体的方法；后面echoName为方法名称
    println(node.name)
}

// 只有使用指针才能改变结构内容。（调用时正常调用，不用传&）
func (node *treeType) setAge(value int) {
    node.age = value
}

func main() {
    // 结构体

    // key-value方式，创建新的实例
    newStruct := treeType{name:"张三",age:18,address: "北京"} // 可以省略任意值，忽略的值为0或空
    fmt.Println(newStruct)

    // 直接赋值方式，创建新的实例
    newStruct1 := treeType{"李四",88,"广州"} // 这方式不能省略任意一个值
    fmt.Println(newStruct1)

    // 通过 . 符号，创建新的实例
    var hongTree treeType
    hongTree.name = "红树"
    hongTree.age = 100
    hongTree.address = "云南"
    fmt.Println(hongTree)

    // 通过 . 符号，访问结构体成员信息
    fmt.Println(hongTree.name)

    // 结构体可作为函数参数（略）

    // 结构体指针
    var redTree treeType
    redTree.age = 1
    redTree.name = "red指针"
    redTree.address = "河南"

    // 传递指针参数
    print(&redTree)

    // 当然 slice 的类型也可以用 刚定义的类型
    oslice := []treeType  {
        {age:2},
        {},
        {"王五",3,"西藏"},
    }
    fmt.Println(oslice)

    // 使用结构体的方法
    redTree.echoName() // 异步操作
    redTree.setAge(999)
    fmt.Println(redTree.age)
}

// 接收指针参数，进行打印
func print (arg *treeType) {
    fmt.Println(arg.address)
    fmt.Println(arg.age)
    fmt.Println(arg.name)
}

// 工厂函数定义
func createNode(v int) *treeType {
    return &treeType{age: v}
}
// 返回局部变量地址可以正常使用
```

# 十四、包和封装
**前置命令**
```shell
mkdir learngo && cd learngo && touch main.go
go mod init learngo
```

## main.go 下调用 op 文件夹里面的数据 
```go
package main

import (
    "fmt"
    "learngo/op"
)

func main() {
    fmt.Println(op.Add(3,3))
}
```

## op/op.go 内容为
```go
package op
// 目录名可以和包名不一样
// 一个目录一个包。main包，包含可执行入口（main函数）
// 为结构定义的方法必须放在同一个包内，但可以是不同文件
// 名字一般使用CamelCase

// public 方法，首字母大写
func Add(a,b int) int {
    return a+b
}

// private 方法，首字母小写
func sub(a,b int) int  {
    return a-b
}
```

# 十五、扩展已有类型
## 通过别名方法，扩展已有类型
```go
// 定义了新类型Queue，该类型具有几种方法
type Queue []int

// 因为需要改参数，所以传地址
func (q *Queue) Push(v int) {
    *q = append(*q, v)
}

func (q *Queue) Pop() int {
    head := (*q)[0] // 注意加括号
    *q = (*q)[1:]
    return head
}

func (q *Queue) IsEmpty() bool{
    return len(*q) == 0
}
```
## 通过组合方法，扩展已有类型
```go
type myStructName struct {
    name *package.targetStruct
}

func (name *myStructName)funcName() {
    //.....扩展方法
}
```
## 通过内嵌方法，扩展已有类型

# 十六、依赖管理 go mod 使用
## 尝试安装 zap 包
```shell
go get -u go.uber.org/zap
```

## go.mod 文件自动生成依赖目录
```go
module gomodTest

go 1.17

require (
    go.uber.org/atomic v1.9.0 // indirect
    go.uber.org/multierr v1.7.0 // indirect
    go.uber.org/zap v1.19.1 // indirect
)
```
*go.mod 目录下 生成 go.sum 校验文件*
*下载的依赖放在了go安装的位置，如 D:\go\bin\pkg\mod\*

## 指定 依赖的版本安装
```go
go get -u go.uber.org/zap@v1.11
```

## 清除 .sum 文件对历史包的引用
```go
go mod tidy
```	