---
title: go grammer
tags: go grammer
grammar_cjkRuby: true
---


## basic variable grammer
#### variable definition
- var count = 1
- count := 1

## object creation and variable assignment
- new方法: 创建一个指定type的对象，并将其全部初始化为0  conn := new(Connection)
- make方法: 
- type直接创建赋值法：
``` javascript
conn := &Connection{serverIP:"", serverPort:9876}
```

## channel
- channel分为with-buffer和without-buffer两种。

``` 
var ch = make(chan int)            // without buffer
var ch = make(chan int, 10)         // with buffer which size is 10
```
without-buffer,是指channel缓冲大小为0，此时read和write操作都会阻塞，知道某一刻read和write同时进行。由此，可以知道，不要再同一个函数或协程里面使用without-buffer channel，有可能会造成deadlock.
```
package main

import "fmt"

func Afuntion(ch chan int) {
	fmt.Println("finish")
	<-ch
}

func main() {
	ch := make(chan int) //无缓冲的channel
	//只是把这两行的代码顺序对调一下
	ch < -1
	go Afuntion(ch)

	// 输出结果：
	// 死锁，无结果
}
```
对于无缓存的channel,放入channel和从channel中向外面取数据这两个操作不能放在同一个协程中，防止死锁的发生；同时应该先利用go 开一个协程对channel进行操作，此时阻塞该go 协程，然后再在主协程中进行channel的相反操作（与go 协程对channel进行相反的操作），实现go 协程解锁．即必须go协程在前，解锁协程在后．
with-buffer channel - 只要channel缓存不满，则可以一直向channel中存入数据，直到缓存已满。只要缓存不空，则可以一直向外取数据，直到缓存空了才会阻塞。由此，带缓存channel不易造成思索，可以同时在一个goroutine中放心使用

- 创建/关闭 channel
必须在producer一侧关闭channel。 channel关闭后，不能再写入数据，但可以从channel中继续读取数据
``` 
var ch = make(chan int)  // 创建channel
close(ch)             // 关闭channel
```

- 向channel中添加数据
```
ch <- data
```
- 从channel中读取数据
```
data <- ch
```
- channel阻塞超时处理
这种写法有点奇怪。select相当于非阻塞式的轮巡。
```
package main

 import (
    "fmt"
    "time"
)

func main() {
    c := make(chan int)
    o := make(chan bool)
    go func() {
        for {
            select {
            case i := <-c:
                fmt.Println(i)
            case <-time.After(time.Duration(3) * time.Second):    //设置超时时间为３ｓ，如果channel　3s钟没有响应，一直阻塞，则报告超时，进行超时处理．
                fmt.Println("timeout")
                o <- true
                break
            }
        }
    }()
    <-o
}

```
## goroutine


## defer

## exception handler
- panic - 抛出异常
- recover - 检测异常，相当于getlasterr in windows。 一般放在defer语句段中。

## type system
- golang没有类型层次结构，即没有基于继承概念的class体系
- golang有struct结构 

``` javascript
type Connection struct {
    ServerIp string
    ServerPort int
}
```
## interface
- 只要某type实现了某interface内的函数，则可以认为该type实现了该interface

``` 
type ConnectionOperations interface {
}
```

## function 

```
func (acceptor Acceptor) foo(listener net.Listener) (conn Conn) {
}
```

## some details
- go没有范围控制关键字(private/public等)，其以函数名是否大写来判断函数的范围


## 设计哲学与体悟
- 简化，即使看起来很山寨。 e.g. 无类型继承体系；无函数作用域关键字；
- 系统扁平化的思想，用组合代替继承；
- interface不光是指action，甚至用interface代替实体

## const enum

``` vbscript
const (
    Sunday Weekday = 0
    Monday Weekday = 1
)
```

## select
The select statement lets a goroutine wait on multiple communication operations.
A select blocks until one of its cases can run, then it executes that case. It chooses one at random if multiple are ready.
Otherwise, keywork 'default' also gives another possibility: 'select' will not block if 'default' case exists eventhough no other cases were met.

- default case
``` 
// Basic sends and receives on channels are blocking.
// However, we can use `select` with a `default` clause to
// implement _non-blocking_ sends, receives, and even
// non-blocking multi-way `select`s.

package main

import "fmt"

func main() {
	messages := make(chan string)
	signals := make(chan bool)

	// Here's a non-blocking receive. If a value is
	// available on `messages` then `select` will take
	// the `<-messages` `case` with that value. If not
	// it will immediately take the `default` case.
	select {
	case msg := <-messages:
		fmt.Println("received message", msg)
	default:
		fmt.Println("no message received")
	}

	// A non-blocking send works similarly. Here `msg`
	// cannot be sent to the `messages` channel, because
	// the channel has no buffer and there is no receiver.
	// Therefore the `default` case is selected.
	msg := "hi"
	select {
	case messages <- msg:
		fmt.Println("sent message", msg)
	default:
		fmt.Println("no message sent")
	}

	// We can use multiple `case`s above the `default`
	// clause to implement a multi-way non-blocking
	// select. Here we attempt non-blocking receives
	// on both `messages` and `signals`.
	select {
	case msg := <-messages:
		fmt.Println("received message", msg)
	case sig := <-signals:
		fmt.Println("received signal", sig)
	default:
		fmt.Println("no activity")
	}
}

```
- timer case

```
package main

import "fmt"
import "time"

func main() {
    c := make(chan int)
    timeout := time.After(5 * time.Second)
    for {
        select {
        case s := <-c:
            fmt.Println(s)
        case <-timeout:
            fmt.Println("You talk too much.")
            return
        }
    }
}
```
- server.go中有一种对channel的特殊用法：将功能函数放入channel中，并等待功能函数执行完成；功能函数将在另一个主循环中执行；

## init function
- 执行顺序： 依赖的其他包 > 本包变量初始化 > init > main
- 每个包可以有多个init函数

## synchronization
- sync.Mutex
mutex lock in golang

- sync.Once
ensure things only runs once
sync.Once起作用的范围，由sync.Once的定义范围决定

- sync.WaitGroup
```
srv.loopWG.Add(1)
```

## single instance
- implement with sync.Once

## object oriented in golang
- 成员函数与this指针

## array and slice
- slice的本质

- 初始化

- array 与 slice互相转换

```
b[:] = a[1:]
```

- 二维slice的初始化 
```
Topics: [][]common.Hash{}
```
- slice之间的深拷贝与浅拷贝
- 删除slice内的元素

``` 
a = append(a[:i], a[i+1:]...)
```
- append元素到slice


```
func append(s []T, vs ...T) []T
```
The first parameter s of append is a slice of type T, and the rest are T values to append to the slice.

The resulting value of append is a slice containing all the elements of the original slice plus the provided values.

If the backing array of s is too small to fit all the given values a bigger array will be allocated. The returned slice will point to the newly allocated array.

## interface{}的接口查询与类型转换


## reference 类型与value 类型


## context


## channel 原理

## map的删除
- map构成
- 删除原理
- 内存回收

## reflection
- reflect.ValueOf()
- Value.Type()
- Type.Elem()
Elem returns a type's element type.
It panics if the type's Kind is not Array, Chan, Map, Ptr, or Slice.

- Type.ChanDir
ChanDir returns a channel type's direction.
```
const (
	RecvDir ChanDir             = 1 << iota // <-chan
	SendDir                                 // chan<-
	BothDir = RecvDir | SendDir             // chan
)
```

- Type.Kind()
```
const (
	Invalid Kind = iota
	Bool
	Int
	Int8
	Int16
	Int32
	Int64
	Uint
	Uint8
	Uint16
	Uint32
	Uint64
	Uintptr
	Float32
	Float64
	Complex64
	Complex128
	Array
	Chan
	Func
	Interface
	Map
	Ptr
	Slice
	String
	Struct
	UnsafePointer
)
```

