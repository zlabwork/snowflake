snowflake - 短ID版本; 53bits
====
[![GoDoc](https://godoc.org/github.com/bwmarrin/snowflake?status.svg)](https://godoc.org/github.com/bwmarrin/snowflake) [![Go report](http://goreportcard.com/badge/bwmarrin/snowflake)](http://goreportcard.com/report/bwmarrin/snowflake) [![Build Status](https://travis-ci.org/bwmarrin/snowflake.svg?branch=master)](https://travis-ci.org/bwmarrin/snowflake) [![Discord Gophers](https://img.shields.io/badge/Discord%20Gophers-%23info-blue.svg)](https://discord.gg/0f1SbxBZjYq9jLBk)

Go语言版snowflake ID生成器，原项目参考[bwmarrin/snowflake](https://github.com/bwmarrin/snowflake)。

* 非常简单的Twitter snowflake生成器.
* 可以解析snowflake IDs.
* 可以将snowflake ID转换为其他数据类型.
* 提供Marshal/Unmarshal方法，用于JSON API.

本项目在原项目基础上调整了三处
1. Epoch设置为2017-06-28 09:10对应的时间戳
2. 初始序列从0修改为0-9的随机数(避免后续ID取余算法时不均匀)
3. 32位时间戳，5位节点，16位自增

### ID格式
* ID共使用53位，兼顾前端js精度问题
* 32 位秒级精度用于存储时间戳, 使用自定义纪元时间.
* 5 位用来存储节点ID - 范围从 0 到 32.
* 16 位用来存储自增序列号 - 范围 0 到 65536.

### 自定义开始时间
默认时间开始于 1498612200(2017-06-28 09:10:00).
Twitter 默认的开始时间是 1288834974657 (2010-11-04 01:42:54).

### 注意事项
如需设置自定义时间纪元或者自定义位数时，必须先设置好，才能使用，否则在使用过程中修改会产生错误

### 工作原理
* 使用ID的32位存储秒精度的时间戳。
* 然后将节点id添加到后续的位中。
* 然后添加序列号，从0开始，对在相同秒内生成的每个ID递增。如果在同一秒内生成的id足够多，以致序列会发生翻转或溢出，则生成函数将暂停到下一秒。

Twitter snowflake 默认格式如下.
```
+--------------------------------------------------------------------------+
| 11 Bit Unused | 32 Bit Timestamp |  5 Bit NodeID  |   16 Bit Sequence ID |
+--------------------------------------------------------------------------+
```

默认设置每个节点每秒可产生65536个ID
备注：常规服务器每秒最多可产生400万左右(基本少于这个数)，新浪微博高峰期每秒发送微博数32000个(2012年数据)

本人实测自用笔记本 MacBook Pro 2013(i5 2.4 GHz 8 GB) 高峰期大概每毫秒产生127个ID，远小于4096 (2019年1月数据)

## 开始使用

### 安装

需要先安装GoLang的运行环境, 如果没有请移步[这里](https://golang.org/doc/install).

```sh
go get github.com/xxtime/snowflake-short
```


### Usage

导入包，然后通过赋值一个唯一的节点标识构造一个节点Node，默认的节点唯一标识范围0-1023(即10 bits)。如果修改了节点标识位数则对应节点标识范围也应该响应调整。使用Generate()方法生成唯一的snowflake ID

您创建的每个节点都必须有唯一的节点编号,如果不保持节点编号唯一，则不能保证所有节点生成的ID唯一。


**实力程序:**

```go
package main

import (
	"fmt"

	"github.com/xxtime/snowflake-short"
)

func main() {

	// 创建一个节点并设置节点ID=1
	node, err := snowflake.NewNode(1)
	if err != nil {
		fmt.Println(err)
		return
	}

	// 生成一个snowflake ID.
	id := node.Generate()

	// 不同的集中方式输出ID
	fmt.Printf("Int64  ID: %d\n", id)
	fmt.Printf("String ID: %s\n", id)
	fmt.Printf("Base2  ID: %s\n", id.Base2())
	fmt.Printf("Base64 ID: %s\n", id.Base64())

	// 打印ID的时间戳
	fmt.Printf("ID Time  : %d\n", id.Time())

	// 打印ID的节点
	fmt.Printf("ID Node  : %d\n", id.Node())

	// 打印ID的序列号
	fmt.Printf("ID Step  : %d\n", id.Step())

	// 生成ID并打印
	fmt.Printf("ID       : %d\n", node.Generate().Int64())
}
```
