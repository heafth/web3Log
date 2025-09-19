package main

type Owner struct {
	id   int
	name string
}

func (o *Owner) Id() int {
	return o.id
}

func (o *Owner) Name() string {
	return o.name
}

func (o *Owner) SetId(id int) {
	o.id = id
}

func (o *Owner) SetName(name string) {
	o.name = name
}

func main() {
	var owner Owner
	owner.SetId(1)
	owner.SetName("张三")
	println("Owner ID:", owner.Id())
	println("Owner Name:", owner.Name())
}

注意：func (o *Owner) Id()  与   func (o Owner) Id() 不同在于
  getter 方法：通常用值接收者或指针接收者均可（因为不修改数据）
  setter 方法：必须用指针接收者（否则无法修改原结构体）
  对于大型结构体，优先使用指针接收者（避免频繁拷贝的性能损耗）
  在之前的示例中，如果把 Age() 改为值接收者，虽然仍能获取到年龄，但会产生结构体拷贝，没有必要。而 SetAge() 必须用指针接收者，否则无法真正修改年龄。



package main

import (
	"fmt"
	// "practice/study0005/studyCopilot"
)

func hello() {
	fmt.Println("Hello, Go!")
	fmt.Println("age =", age)
}

var age = 12

const keyword string = "123456"

// 定义多个变量
var (
	a3 = 3
	a4 = 4
)

func main() {
	//先声明
	var name1 string
	//后赋值
	name1 = "张三"
	fmt.Println(name1)

	//声明并赋值
	var name2 string = "李四"
	fmt.Println(name2)

	//可以省略 string 自动推导类型
	var name3 = "王五"
	fmt.Println(name3)

	//声明并赋值 省略 var 自动推导类型
	name4 := "赵六"
	fmt.Println(name4)

	hello()

	//定义多个变量
	var a1, a2 = 1, 2
	fmt.Println(a1, a2)

	fmt.Println(a3, a4)

	fmt.Println("keyword =", keyword)

	//fmt.Println(copilot.Copilot.Version)
}
