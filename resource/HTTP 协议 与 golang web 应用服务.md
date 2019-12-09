学习了C/S架构和HTTP协议基础知识后，大体上了解到web服务的实现思路，即在服务器和连接的客户端之间传输限定格式的文本。golang的包提供了很多方便使用的功能。
使用http包搭建一个简单的web服务器。

```go
package main

import (
    "fmt"
    "net/http"
    "strings"
    "log"
)

func sayhelloName(w http.ResponseWriter, r *http.Request) {
    r.ParseForm()  //解析参数，默认是不会解析的
    fmt.Println(r.Form)  //这些信息是输出到服务器端的打印信息
    fmt.Println("path", r.URL.Path)
    fmt.Println("scheme", r.URL.Scheme)
    fmt.Println(r.Form["url_long"])
    for k, v := range r.Form {
        fmt.Println("key:", k)
        fmt.Println("val:", strings.Join(v, ""))
    }
    fmt.Fprintf(w, "Hello astaxie!") //这个写入到w的是输出到客户端的
}

func main() {
    http.HandleFunc("/", sayhelloName)       //设置访问的路由
    err := http.ListenAndServe(":9090", nil) //设置监听的端口
    if err != nil {
        log.Fatal("ListenAndServe: ", err)
    }
}
```
函数sayhelloName解析Request，打印到服务器端，对客户端输出hello信息。
使用go run运行，此时已经在9090端口监听请求了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191104192723317.png)
使用curl看结果。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191104192834303.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1aUZlaURlVHVvTmlhb0da,size_16,color_FFFFFF,t_70)
在连接了客户端的socket后，需要处理客户端的请求，交给相应的handler去处理，在之前的代码中，golang调用了ListenAndServe，实际上是初始化了一个server对象，使用TCP搭建了一个服务。
整个http的处理过程，可以通过go的源码看到：

```go
func (srv *Server) Serve(l net.Listener) error {
	defer l.Close()
	var tempDelay time.Duration // how long to sleep on accept failure
	for {
		rw, e := l.Accept()
		if e != nil {
			if ne, ok := e.(net.Error); ok && ne.Temporary() {
				if tempDelay == 0 {
					tempDelay = 5 * time.Millisecond
				} else {
					tempDelay *= 2
				}
				if max := 1 * time.Second; tempDelay > max {
					tempDelay = max
				}
				log.Printf("http: Accept error: %v; retrying in %v", e, tempDelay)
				time.Sleep(tempDelay)
				continue
			}
			return e
		}
		tempDelay = 0
		c, err := srv.newConn(rw)
		if err != nil {
			continue
		}
		go c.serve()
	}
}
```
在监控端口之后，调用了srv.Serve(net.Listener)函数来处理客户端的请求信息，在for循环中，先接收请求，然后创建一个Conn，go c.serve()，这样实现了高并发。
#### go语言的context库
goroutine在go语言官方文档中被描述成轻量级的执行线程，相比线程，管理它们消耗的资源相对更少。
在main函数中调用的goroutine经常在main函数退出时没完成，为了让main函数等待，需要用到channel。
channel是goroutine之间的沟通渠道，可以将任何类型的信息从一个routine传递到另一个。
在go中，context包可以传递一个context（上下文）的程序，比如超时。
创建context

```go
ctx := context.Background()
```
产生一个空的context，只能用于main或顶级处理请求，用于派生其他context。

context.TODO()

```go
ctx := context.TODO()
```
创建一个空context，与background不同的是，静态工具可以使用它验证context是否正确传递。

```go
ctx := context.WithValue(context.Background(), key, "test")
```
接收context并返回派生context，值与键关联，通过context树与context一同传递。

```go
ctx, cancel := context.WithCancel(context.Background())
```
函数创建从传入的父context派生的新context，返回了一个取消函数，只有创建它的函数才能调用它取消此context。

```go
ctx, cancel := context.WithDeadline(context.Background(), time.Now().Add(2 * time.Second))
ctx, cancel := context.WithTimeout(context.Background(), 2 * time.Second)
```
截止时间后自动取消。

利用context设置截止日期的例子：

```go
func sleepRandomContext(ctx context.Context, ch chan bool) {
    //在此处应该清除任务，由于这里没有context被创建，所以也不用cancel
    defer func() {
        fmt.Println("sleepRandomContext complete")
        ch <- true
    }()
    //建立一个channel
    sleeptimeChan := make(chan int)
    //在goroutine中开始一个进程，用chan通信
    go sleepRandom("sleepRandomContext", sleeptimeChan)
    select {
    case <-ctx.Done():
        //如果context到期了
        fmt.Println("Time to return")
    case sleeptime := <-sleeptimeChan:
        //如果进程在context取消前就执行完了
        fmt.Println("Slept for ", sleeptime, "ms")
    }
}
```
<-ctx.Done() 一旦 Done 通道被关闭，则<-ctx.Done(): 被选择。一旦发生这种情况，函数应该放弃运行并准备返回，此时应该关闭所有打开的管道，释放资源并从函数返回。

有关context包的更多知识：[Go语言并发模型：使用 context](https://segmentfault.com/a/1190000006744213)
