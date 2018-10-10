title: 【设计模式】Effective Objective-C 2.0
date: 2018/09/13
categories:
- Go
tags:
- 编程
- Golang
---

## Go 学习之旅
先跟完官网版的[Go 指南](https://tour.go-zh.org/welcome/1)。
在函数这里，`Go`中的函数可以没有参数或者接受多个参数，但是形参的类型在变量名称之后。这点看起来比较奇怪。据说是为了从左到右读起来方便：
```
package main
import "fmt"
func add(x int, y int) int {
    return x + y
}

func main() {
    fmt.Println(add(32,14))
}
```
当函数形参类型一样时，除了最后一个，其他都可以省略。而且函数还允许返回多个返回值：
```
package main
import "fmt"
func add(x, y int, z float32) (int, float32) {
    return x + y, z
}

func main() {
    a, b := add(32,14,3.0)
    fmt.Println(add(32,14,3.0))
}
```
此外，函数还允许返回值命名。`return`语句代表返回所有命名的返回值。
```
package main
import "fmt"
func add(x, y int) {
    x = x + 10
    y = y + 20
    return
}

func main() {
    a, b := add(32,14)
    fmt.Println(a, b)
}
```
