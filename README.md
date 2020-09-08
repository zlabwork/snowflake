snowflake - 短ID版本; 53bits
====
[![GoDoc](https://godoc.org/github.com/bwmarrin/snowflake?status.svg)](https://godoc.org/github.com/bwmarrin/snowflake) [![Go report](http://goreportcard.com/badge/bwmarrin/snowflake)](http://goreportcard.com/report/bwmarrin/snowflake) [![Build Status](https://travis-ci.org/bwmarrin/snowflake.svg?branch=master)](https://travis-ci.org/bwmarrin/snowflake) [![Discord Gophers](https://img.shields.io/badge/Discord%20Gophers-%23info-blue.svg)](https://discord.gg/0f1SbxBZjYq9jLBk)

Go语言版snowflake ID生成器，原项目参考[bwmarrin/snowflake](https://github.com/bwmarrin/snowflake)。

本项目是snowflake的short版本，
* 生成更短的id (15-16位)  用作用户ID使用更合适
* 生成的ID长度为53bits便于在js上发生交互(js处理超过53bits的整数会失去精度，通常做法是转换为字符串后交给js处理，但是本项目生成的ID天然支持js不需要额外处理)
* 可以解析snowflake IDs.
* 可以将snowflake ID转换为其他数据类型.
* 提供Marshal/Unmarshal方法，用于JSON API.

本项目在原项目基础上调整了二处
1. Epoch设置为2017-06-28 09:10对应的时间戳
2. 39位时间戳10ms级，6位节点，8位自增

### ID格式
* ID共使用53bits，兼顾前端js精度问题，可使用170年时间
* 39 位秒级精度用于存储时间戳, 使用自定义纪元时间.
* 6 位用来存储节点ID - 范围从 0 到 64.
* 8 位用来存储自增序列号 - 范围 0 到 256.

### 自定义开始时间
默认时间开始于 1498612200000(2017-06-28 09:10:00).
Twitter 默认的开始时间是 1288834974657 (2010-11-04 01:42:54).

### 注意事项
如需设置自定义时间纪元或者自定义位数时，必须先设置好，才能使用，否则在使用过程中修改会产生错误

### 工作原理
* 使用39位存储时间, 精度为10ms, 即10ms为一个单位。
* 然后将节点id添加到后续的位中。
* 然后添加序列号，从0开始，每隔10ms内生成的每个ID递增。如果在同一个时间内生成的id足够多，为防止序列发生翻转或溢出，则生成函数将暂停到下一个10ms。

snowflake short 默认格式如下.
```
+--------------------------------------------------------------------------+
| 11 Bit Unused | 39 Bit Timestamp |  6 Bit NodeID  |   8 Bit Sequence ID |
+--------------------------------------------------------------------------+
```

默认设置每个节点每秒可产生25600个ID，最多可以设置64个节点服务器
新浪微博高峰期每秒发送微博数32000个(2012年数据)

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
