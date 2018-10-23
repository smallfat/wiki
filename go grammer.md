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