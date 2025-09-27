package main

import (
	"fmt"
	"time"
)

// 如果不用协程，则可以用如下方法判断到三个协程是否走完
var wait int

func shopping(name string) {
	fmt.Println(name, "beginning to buy stuff")
	time.Sleep(1 * time.Second)
	fmt.Println(name, "ending to buy stuff")
	wait--
}

func main() {
	wait = 3
	startTime := time.Now()
	go shopping("张三")
	go shopping("李四")
	go shopping("王五")

	//for{}是死循环 ,判断 if wait == 0 即是表示三个协程都走完了，则break
	for {
		if wait == 0 {
			break
		}
	}

	time.Sleep(1 * time.Second)
	fmt.Println("总共花了", time.Since(startTime), "时间")
}

----------------------------------------------------------

package main

import (
	"fmt"
	"sync"
	"time"
)

/* sync.WaitGroup 的核心是 “计数器”，通过 Add() 加计数、Done() 减计数、Wait() 等计数归 0，实现 goroutine 同步。
主 goroutine：wg.Add(子任务数) → 初始化计数器。​
启动 N 个子 goroutine，每个子 goroutine 内 defer wg.Done()。​
主 goroutine：wg.Wait() → 阻塞等所有子任务完成。​
计数器归 0 → 主 goroutine 继续执行。 */

func shopping(name string, wait *sync.WaitGroup) {
	fmt.Println(name, "beginning to buy stuff")
	time.Sleep(1 * time.Second)
	fmt.Println(name, "ending to buy stuff")
	wait.Done()
}

func main() {
	var wait sync.WaitGroup
	wait.Add(3)
	startTime := time.Now()
	go shopping("张三", &wait)
	go shopping("李四", &wait)
	go shopping("王五", &wait)

	wait.Wait()
	time.Sleep(1 * time.Second)
	fmt.Println("总共花了", time.Since(startTime), "时间")
}

--------------------------------------------------------------

package main

import (
	"fmt"
	"sync"
	"time"
)

/*
channel（通道）是 Go 语言中goroutine 间通信的核心机制，
实现 “通过通信共享内存”（而非 “通过共享内存通信”），同时
保证并发安全，避免数据竞争。本质是一个 “数据管道”，可
在 goroutine 间传递数据。
1. 定义语法
 声明通道（未初始化的通道为nil，无法直接使用）​
var ch chan 数据类型​
 初始化通道（必须用make创建，指定容量）​
ch = make(chan 数据类型, [容量])
*/

// 声明一个长度为0的信道 如果make(chan int, 2) 长度为2
var moneyChan = make(chan int)

/*
 2. 两大类型​
（1）无缓冲通道（容量 = 0）​
定义：ch := make(chan int)（未指定容量，默认容量 0）。​
核心特性：发送和接收操作必须同时就绪，否则会阻塞。​
发送方（goroutine）：发送数据时，会阻塞直到有接收方接收。​
接收方（goroutine）：接收数据时，会阻塞直到有发送方发送。​
适用场景：goroutine 间需 “同步通信”（如传递信号、确保操作顺序）。​
（2）有缓冲通道（容量 > 0）​
定义：ch := make(chan int, 3)（容量为 3，可缓存 3 个 int 数据）。​
核心特性：缓冲未满时发送不阻塞，缓冲未空时接收不阻塞。​
发送方：仅当缓冲满时，发送才阻塞，直到有接收方取走数据。​
接收方：仅当缓冲空时，接收才阻塞，直到有发送方存入数据。​
适用场景：goroutine 间 “异步通信”（如批量传递数据，平衡生产 / 消费速度）。
*/

/**
*注意：​
*数据类型必须与通道声明的类型一致（如 int 通道不能发送 string）。​
*向已关闭的通道发送数据会触发 panic。​
*向 nil 通道发送数据会永久阻塞。
 */

/*
 3. 关闭通道（close (ch)）​
    语法：close(ch)（关闭已初始化的通道）。​
    注意：​
    只能由发送方关闭（接收方关闭通道不合理，可能导致发送方 panic）。​
    重复关闭通道会触发 panic。​
    关闭 nil 通道会触发 panic。​
    通道关闭后，无法再发送数据，但可接收剩余数据。
*/
func pay(name string, moneyNumber int, wait *sync.WaitGroup) {
	fmt.Println(name, "beginning to buy stuff")
	time.Sleep(1 * time.Second)
	fmt.Println(name, "ending to buy stuff")

	moneyChan <- moneyNumber
	wait.Done()
}

func main() {
	var wait sync.WaitGroup
	wait.Add(3)
	startTime := time.Now()
	go pay("张三", 3, &wait)
	go pay("李四", 4, &wait)
	go pay("王五", 5, &wait)

	/**
	*go func(){ ... } 定义了一个匿名的 goroutine 函数（函数体包含在 {} 中）
	*最后的 () 表示立即执行这个匿名函数，将其启动为一个独立的 goroutine
	 */
	go func() {
		defer close(moneyChan)
		wait.Wait()
	}()

	/* 写法一
	 for {
		money, ok := <-moneyChan
		if !ok {
			break
		} else {
			fmt.Println(money, ok)
		}
	} */

	//写法二
	var moneyList []int

	/* channel 的 for range 遍历 go语言做过特殊处理，
	它本质上就是在循环 “自动接收 channel 数据”，且等价
	于 底层 “循环调用 <-moneyChan 接收数据”不仅不会报
	错，还是处理 channel 数据的常用优雅写法 */
	for money := range moneyChan {
		moneyList = append(moneyList, money)
	}

	fmt.Println(moneyList)
	time.Sleep(1 * time.Second)
	fmt.Println("总共花了", time.Since(startTime), "时间")
}

-----------------------------------------------------------

package main

import (
	"fmt"
	"sync"
	"time"
)

/* select 是 Go 语言用于监听多个 channel 操作的控制语句，
能同时等待多个 channel 的发送 / 接收事件，当任意一个
channel 操作就绪时，就执行对应分支逻辑，实现 goroutine
间的高效同步与通信。 */

/*
随机选择：若多个 channel 同时就绪，select 会随机挑选一个分支执行，避免固定顺序导致的资源倾斜。​
阻塞与非阻塞：无 default 分支时，select 会阻塞，直到有一个通道操作就绪；有 default 分支时，所有通道未就绪则立即执行 default，不阻塞。​
仅监听 channel：case 分支只能是 channel 的发送或接收操作，不能是普通条件判断（区别于 switch）。​
支持 nil 通道：若 case 中的通道是 nil，该分支会被永久忽略，不会参与就绪判断。

在 Go 中，从一个已关闭的 channel读取数据时，会立即返回该 channel 类型的零值,这里触发了select里面的doneChan
*/

var moneyChan = make(chan int)
var nameChan = make(chan string)
var doneChan = make(chan struct{})

func pay(name string, moneyNumber int, wait *sync.WaitGroup) {
	fmt.Println(name, "beginning to buy stuff")
	time.Sleep(3 * time.Second)
	fmt.Println(name, "ending to buy stuff")

	moneyChan <- moneyNumber
	nameChan <- name
	wait.Done()
}

func main() {
	var wait sync.WaitGroup
	wait.Add(3)
	startTime := time.Now()
	go pay("张三", 3, &wait)
	go pay("李四", 4, &wait)
	go pay("王五", 5, &wait)

	go func() {
		defer close(moneyChan)
		defer close(nameChan)
		defer close(doneChan)
		wait.Wait()
	}()

	var moneyList []int
	var nameList []string

	var event = func() bool {
		for {
			select {
			case money, ok1 := <-moneyChan:
				if !ok1 {
					fmt.Println("此通道已关闭")
				}
				moneyList = append(moneyList, money)
			case name := <-nameChan:
				nameList = append(nameList, name)
			case <-doneChan: //由close(doneChan)触发
				doneChan = nil // 设为nil，后续select不再处理该case
				return true
			default: /* default 分支的作用是：当所有 case 中的 channel
				            操作都需要阻塞时，立即执行 default，
							避免整个整个 select 陷入等待。 */
				fmt.Println("所有通道都未就绪2")
			}
		}
	}
	yesorno := event()
	fmt.Print(yesorno)
	fmt.Println(moneyList)
	fmt.Println(nameList)
	time.Sleep(1 * time.Second)
	fmt.Println("总共花了", time.Since(startTime), "时间")
}

---------------------------------------------------------------

package main

import (
	"fmt"
	"time"
)

/* select 是 Go 语言用于监听多个 channel 操作的控制语句，
能同时等待多个 channel 的发送 / 接收事件，当任意一个
channel 操作就绪时，就执行对应分支逻辑，实现 goroutine
间的高效同步与通信。 */

/*
随机选择：若多个 channel 同时就绪，select 会随机挑选一个分支执行，避免固定顺序导致的资源倾斜。​
阻塞与非阻塞：无 default 分支时，select 会阻塞，直到有一个通道操作就绪；有 default 分支时，所有通道未就绪则立即执行 default，不阻塞。​
仅监听 channel：case 分支只能是 channel 的发送或接收操作，不能是普通条件判断（区别于 switch）。​
支持 nil 通道：若 case 中的通道是 nil，该分支会被永久忽略，不会参与就绪判断。

在 Go 中，从一个已关闭的 channel读取数据时，会立即返回该 channel 类型的零值,这里触发了select里面的doneChan
*/

var doneChan = make(chan struct{})

func overTime() {
	time.Sleep(3 * time.Second)
	close(doneChan)
}

func main() {
	go overTime()

	var event = func() {
		for {
			select {
			case <-doneChan: //由close(doneChan)触发
				doneChan = nil // 设为nil，后续select不再处理该case
				return
			case <-time.After(time.Second * 2): //这里如果后面有default分支 会导致 永远执行不到这行
				fmt.Println("超时")
			}
		}
	}
	event()
}
