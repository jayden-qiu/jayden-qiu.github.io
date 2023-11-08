---
title: Go标准库flag
date: 2023-11-08 16:11:28
tags:
description: 包flag实现了命令行标志解析
categories:
- Golang
---

## 程序为
```go
package main

import (
  "fmt"
  "flag"
)

var (
  intflag int
  boolflag bool
  stringflag string
)

// 初始化变量
func init() {
  flag.IntVar(&intflag, "intflag", 0, "int flag value")
  flag.BoolVar(&boolflag, "boolflag", false, "bool flag value")
  flag.StringVar(&stringflag, "stringflag", "default", "string flag value")
}

func main() {
  // 将用户输入的变量解析为变量值
  flag.Parse()

  fmt.Println("int flag:", intflag)
  fmt.Println("bool flag:", boolflag)
  fmt.Println("string flag:", stringflag)
}
```

## 使用方式
```shell
./main -h
```
或
```go
./main -intflag 1 -boolflag false -stringflag "haha"
```