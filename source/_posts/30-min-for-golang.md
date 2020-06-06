---
title: 30 min for Golang
url: 440.html
id: 440
categories:
  - 未分类
date: 2019-03-23 00:00:00
tags:
---

前面的话
----

想想自己说是学 `Go` 已经想了了半年了都，可自己以前一直想的 **WeGene** 一样。从有想法到付款，一下子晃了一年过去了。

**看着价钱从 299 到 499** ,突然和进行看的《时间管理》书里面写到，`迟早都要做，晚做不如早做好` 。想想好像的确该认清点什么了。

* * *

对了，这篇文章 全篇参考

> *   [Go语言教程 -- RUNOOB](http://www.runoob.com/go/go-tutorial.html)

Go 简介
-----

> 先弄懂哪里使得他如此有魅力

Go 自 2007 年开始研发，在2009 年开源。 Go 的语言的特色如下：

*   简洁、快速、安全
*   并行、有趣、开源
*   内存管理、数组安全、编译迅速

最大的亮点在于与身俱来的并发能力，用于服务端的开发。之前也有遇到很多的基于GO的项目：`frp，docker-compose，etc.` 。

* * *

这里大量应用 《Go语言程序设计》这本书里的几句箴言（几大惊喜）

1.  大道至简的哲学 语言最小化
    
2.  并行支持
    
3.  非侵入式接口
    
4.  极度简化但是完备的OOP
    
5.  错误处理规范E
    
        f, err :- os.Open(file)
        if err != nil {
           //err
        }
        defer f.Close()
    
6.  功能的内聚。
    
7.  消除了堆和栈的边界
    
8.  Go 对C的支持

Go 基础
-----

### hello world 开始

传统？Go 程序的基础组成由 `包声明，引入包，函数，变量，语句&注释，注释`

    package main
    // 导入格式化输入的包
    import &quot;fmt&quot;
    func main() {
        /*注释与C类似*/
        fmt.Print(&quot;test&quot;)
    }

* * *

Go 的精致的地方，在朋友当时吹吹这个用 首字母大大小写来区别函数的可见性的时候，就觉得不一般了。这里强制规定了：**左花括号不能换行**（消灭异端？？） 。

### 基础语法

*   左花括号不可换行
    
*   其实感觉吸取JS 的风格，不需要 `;` 如果同行多语句，需要使用 分号进行分隔。
    
*   注释风格同 C
    
*   标识符大小写敏感，同C，不可以数字开头，不可占用关键字，不可带运算符
    
*   关键字 ， （虽然不知道怎么用，先认识一些）
    
    -
    
    -
    
    -
    
    -
    
    -
    
    case
    
    defer
    
    go
    
    map
    
    struct
    
    chan
    
    else
    
    goto
    
    package
    
    switch
    
    const
    
    fallthrough
    
    if
    
    range
    
    type
    
    continue
    
    for
    
    import
    
    return
    
    var
    
    break
    
    default
    
    func
    
    interface
    
    select
    
*   基本数据类型
    
    布尔，数字，字符串，其他派生（指针，数组，结构体，Channel，函数，切片，接口，Map）
    
    这里直接 合并了 C 里面常用的几个 `typedef` ：`uint8,int64,float64,complex128`
    

### 变量

*   类似与 JS 使用 var 关键字进行变量定义
    
        var <identifer> <type>
        var num1 int64
        var num2 = 123    # 编译语言，自动确定类型
        num := 123        # 省略 var 关键字， := 为声明语句
    
*   多变量声明的方式
    
        a,b,c := 1,2,3
        var (
        a int8
        b int64
        )
    
*   值类型和引用类型
    
    这里结合 C 的思想，这里的变量是内存中的值。通过 `&` 对变量进行取值。
    
    但是，引类型是不一样的，这里理解为 指针吧，引用的值是保存其值的内存地址。
    
*   局部变量不允许声明但不使用，全局变量支持。
    
*   两个值进行交换 `a,b = b,a`
    
*   空白标识符用于抛弃值
    
        a,_,c = 1,2,5
    

### 常量

*   同多数语言使用 `const` 标识符。
    
        const a,c,d = 1,3,4
        const (
          Unknown = 0
          Female = 1
          Male = 2
        )
    
*   `iota` 这里不是 C 里面的 `int to ascii` ，可以理解为，编译器维护的一个全局的常量。**iota 在 const关键字出现时将被重置为 0(const 内部的第一行之前)，const 中每新增一行常量声明将使 iota 计数一次(iota 可理解为 const 语句块中的行索引)。**
    
        const (
          a = iota   //0
          b          //1
          c          //2
          d = "ha"   //独立值，iota += 1
          e          //"ha"   iota += 1
          f = 100    //iota +=1
          g          //100  iota +=1
          h = iota   //7,恢复计数
          i          //8
        )
    

### 运算符

*   基础的都支持 包括 \+\+ 和 \-\- 。
    
*   关系运算符也是一样的。
    
*   逻辑运算也一样
    
*   位运算也一样 （^ 这个叫做脱字符），支持位移
    
*   复合运算符，较之于 C 有了拓展
    
    = ， += ，-=， *= ，<<= ...
    
*   取值，指向 &*
    
*   优先级...记不住，用括号吧

### 条件语句

*   `if/if-else` , `switch`, `select` 多了个没有用过的， 下面给出基本格式
    
        if true {
          fmt.Print("123")
        } else {
          fmt.Print("NO")
        }
        switch val {
          case 1:
              fmt.Print("123")
          case 2:
              fmt.Print("123")
          case grade == "F":
            fmt.Printf("不及格\n" )    
        }
        select {
          case communication clause  :
             statement(s);      
          case communication clause  :
             statement(s); 
          /* 你可以定义任意数量的 case */
          default : /* 可选 */
             statement(s);
        }
    
*   Type Switch 这个可以根据 类型来决定 Case，异常一样。**不过很奇怪啊，是编译语言，类型还能变吗**
    
        switch x.(type){
          case type:
             statement(s);      
          case type:
             statement(s);
        }
    
*   `fallthrough` 强制的执行下一个 Case 的内部代码，（感觉很大的提高了灵活性）
    
*   select 之前没有见过 不过看其解释如下：
    
    > select是Go中的一个控制结构，类似于用于通信的switch语句。每个case必须是一个通信操作，要么是发送要么是接收。
    > 
    > select随机执行一个可运行的case。如果没有case可运行，它将阻塞，直到有case可运行。一个默认的子句应该总是可运行的。
    
    **这样就突然清楚了，LinuxC 里面的用于异步IO的 `select()` 在这里被表现为一个结构了，妙啊**
    
    关于这个结构的介绍，应该是和并发是息息相关的，现在看的有点蒙。先跳吧
    

### 循环语句

*   缩减关键词，去掉了 while 使用for 实现 当和直到。形式如下
    
        for init; condition; post { } // 同 for(1;2;4) {3}
        for condition { }             // 同 while(condition){}
        for { }                           // 同 for(;;){}
    
*   使用 for 也可以很方便的对元素进行迭代
    
        for key, value := range oldMap {
          newMap[key] = value
        }
    
*   break，continue， goto
    
    brk 和 ctnu 一样的功能，记得switch 也需要 brk
    
    没想到这里还留着 goto ，一样的 `goto lable` ，一般不要乱跳，用来统一异常处理也行。
    
*   `for true {fmt.print('dala')}`

### 函数

*   和 C 一样，`main()` 为必须的函数入口。
    
*   函数定义格式如下：
    
        func function_name( [parameter list] ) [return_types] {
         //函数体
        }
        
        func test (a, b int) int {
          return a + b
        }
    
*   多返回值 和python类似，可以进行多个值的返回
    
        func swap(x, y string) (string, string) {
         return y, x
        }
        // 多个参数进行接收
        a,b := swap("a", "sd")
        _,c := swap("c", "ddd")
    
*   值传递，和引用传递，类似于 CPP 语法
    
        /* 定义交换值函数*/
        func swap(x *int, y *int) {
         var temp int
         temp = *x    /* 保持 x 地址上的值 */
         *x = *y      /* 将 y 值赋给 x */
         *y = temp    /* 将 temp 值赋给 y */
        }
    
    这里是直接传递引用的，如果在 CPP 的惯性下，可能更愿意理解为指针。
    

### 变量的作用域Scope

*   局部变量，全局变量，形式变量。
*   全局变量可以在整个包甚至外部包（被导出后）使用。
*   同CPP里面的变量相同，记得初始化局部变量，全局变量，默认初始化为 0/nil

### 数组

*   和其他语言一样，常见的顺序的数据结构。定义形式如下：
    
        // 声明
        var variable_name [SIZE] variable_type
        var balance [10] float32
        
        // 定义形式比较诡异
        var balance = [5]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
    
*   可以很方便的使用 for range 进行遍历：
    
        array := [5]int{1, 3, 4, 5, 7}
        for i := range array {
        fmt.Println(array[i])
        } 
    
*   二维数组一样，行列的形式。
    
*   数组作为参数 使用 C 的惯性，写出以下的三种形式。
    
        func func1(a []int) int {
        return a[1]
        }
        // func2 在编译的时候是会报错的。
        func func2(a *int) int {
        return a[1]
        }
        func func3(a [5]int) int {
        return a[1]
        }
    

### 指针

*   这里也是 Go 的精髓 ，保留了指针，但是为了保证稳定， 削减了其Hack 的灵活性
*   指针的内容大体上和 C 类似
*   空指针被定义为了 `nil`
*   大体上的用法和 C 类似，如果具体讲起可能需要一本 《Go与指针》了XD
*   `var pptr **int` 像是 `char ** argv` 一样的是存在的

### 结构体

*   这个在 Go 里面可是个好东西。当时在 C 里面辛辛苦苦一大堆的函数指针，图啥呢
    
*   其定义格式 以及初始化如下：
    
        type struct_variable_type struct {
         member definition;
         member definition;
         ...
         member definition;
        }
        variable_name := structure_variable_type {value1, value2...valuen}
        // 成员名，可以作为键进行映射
        variable_name := structure_variable_type { key1: value1, key2: value2..., keyn: valuen}
    
*   其用法和一般 OO语言类似
    
        type Books struct {
         title string
         author string
         subject string
         book_id int
        }
        
        var Book1 Books        /* 声明 Book1 为 Books 类型 */
        var Book2 Books        /* 声明 Book2 为 Books 类型 */
        
        /* book 1 描述 */
        Book1.title = "Go 语言"
        Book1.author = "www.runoob.com"
        Book1.subject = "Go 语言教程"
        Book1.book_id = 6495407
    
*   一样的存在结构体（对象）的指针。**得力于GC不需要delete就不用紧张了**
    
        var Book1 books;
        var struct_pointer *Books
        struct_pointer = &Book1;
        
        a := book{123, 456}
        b := book{a: 123, b: 123}
        fmt.Print(a, b)
        
        type book struct {
        a int
        b int
        }
        
    

### 切片

*   很方便的提供了和python 的类似的 数组切片方式
    
        s := arr[startIndex:endIndex] 
    
*   `len()` 和 `cap()` 分别是获取数组元素个数，和可以容纳的最大元素数。
    
*   `append()` 和 `copy()`
    
    这里的数组是基础的元素，不能使用 append
    
        a := [5]int{123,123,123}
        a.append(123) // 这里报错
        a = append(a, 1)
        // 使用 make 可以创建一个新的数组
        b := make([]int, len(numbers), (cap(numbers))*2)
    

### Range

*   用于遍历结构的一个常用功能。 感觉类似 Python 的 range
*   range 会返回两个参数，分别是 index，和元素

### Map

*   和py一样的键值对
    
*   声明形式一样很奇怪
    
        /* 声明变量，默认 map 是 nil */
        var map_variable map[key_data_type]value_data_type
        
        /* 使用 make 函数 */
        map_variable := make(map[key_data_type]value_data_type)
    
*   值得注意的是，键值的类型在声明的时候已经指定。
    
*   遍历键值
    
        for country := range countryCapitalMap {
          fmt.Println(country, "首都是", countryCapitalMap [country])
        }
    
*   GO 的错误处理，虽然不是这个部分的，不过从这里的代码细节可以看出，双返回值其中的一个内容就是错误码
    
          capital, ok := countryCapitalMap [ "美国" ] /*如果确定是真实的,则存在,否则不存在 */
          /*fmt.Println(capital) */
          /*fmt.Println(ok) */
          if (ok) {
              fmt.Println("美国的首都是", capital)
          } else {
              fmt.Println("美国的首都不存在")
          }
    

### 递归

*   Go 支持递归调用

### 接口

*   接口是一种数据类型，
    
*   任何其他类型，只要实现了这些方法就是实现了这个接口
    
*   接口定义的示例代码：
    
        /* 定义接口 */
        type interface_name interface {
         method_name1 [return_type]
         method_name2 [return_type]
         method_name3 [return_type]
         ...
         method_namen [return_type]
        }
        
        /* 定义结构体 */
        type struct_name struct {
         /* variables */
        }
        
        /* 实现接口方法 */
        func (struct_name_variable struct_name) method_name1() [return_type] {
         /* 方法实现 */
        }
        ...
        func (struct_name_variable struct_name) method_namen() [return_type] {
         /* 方法实现*/
        }
    
*   非侵入式 的接口。这里给了个例子，慢慢发现真的很精妙。
    
        package main
        import "fmt"
        
        type Phone interface {
          call()
        }
        
        type NokiaPhone struct {
        }
        
                       fmt.Println("I am Nokia, I can call you!")
        }
        
        type IPhone struct {
        }
        
        func (iPhone IPhone) call() {
          fmt.Println("I am iPhone, I can call you!")
        }
        
        func main() {
          var phone Phone           // 声明接口
          phone = new(NokiaPhone)   // 赋值到对象
          phone.call()          // 调用接口
          phone = new(IPhone)
          phone.call()
        }  
    

### 错误处理

*   Go 的错误处理是接口特性的很好的体现
    
*   具体内容比较核心，属于语言特性，后面再深入学习

### 并发

*   Go 的极重要的特性
    
*   使用关键字 `go` 来启动 goruntine
    
        go foo(a,b,c) // 有点像Node的异步
        go foo1(x,y,z)    
    
*   **通道 Channel**
    
    *   用来传递数据的一个数据结构
        
    *   用于两个 goroutine 之间的的值来同步和通信
        
    *   通道声明：
    
        ch := make(chan int)
        
        ch <- v    // 把 v 发送到通道 ch
        v := <-ch  // 从 ch 接收数据
                   // 并把值赋给 v
    
    *   通道缓存实例：
    
        func main() {
                // 这里我们定义了一个可以存储整数类型的带缓冲通道
                // 缓冲区大小为2
                ch := make(chan int, 2)
        
                // 因为 ch 是带缓冲的通道，我们可以同时发送两个数据
                // 而不用立刻需要去同步读取数据
                ch <- 1
                ch <- 2
        
                // 获取这两个数据
                fmt.Println(<-ch)
                fmt.Println(<-ch)
        }
    

后面的话
----

至此30分钟的教程完结，是自己针对有编程经验前提下的精简吧。

后面的并发，和错误处理的部分，是Go 的特性，还会进行详尽的学习。