开发简单 web 服务程序 cloudgo，了解 web 服务器工作原理。
>任务目标
熟悉 go 服务器工作原理
基于现有 web 库，编写一个简单 web 应用类似 cloudgo。
使用 curl 工具访问 web 程序
对 web 执行压力测试
基本要求
编程 web 服务程序 类似 cloudgo 应用。
要求有详细的注释
是否使用框架、选哪个框架自己决定 请在 README.md 说明你决策的依据
使用 curl 测试，将测试结果写入 README.md
使用 ab 测试，将测试结果写入 README.md。并解释重要参数。

[项目传送门](https://github.com/Kate0516/ServiceComputing/tree/master/week9)
主要使用了go语言提供的http包，可以简单地搭建web服务，设置路由，cookie等。
使用了negroni库，是一个很地道的 Web 中间件，它是一个具备微型、非嵌入式、鼓励使用原生 net/http 库特征的中间件。可以把自己的http.Handler加入到Negroni的中间件链中，Negroni会自动调用处理的HTTP Request。
[飞雪无情的博客](https://www.flysnow.org/tags/negroni/)
测试：
进入根目录，运行main.go，监听9090端口。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191111135636367.png)
访问9090端口。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191111135910776.png)
服务器终端显示访问记录。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191111140050756.png)
访问localhost:9090/login

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019111114085222.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191111140915485.png)
填写用户名和密码gaozhen；gaozhen
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191111141001852.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1aUZlaURlVHVvTmlhb0da,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191111141017485.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019111114105881.png)
使用curl测试。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191111141236240.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1aUZlaURlVHVvTmlhb0da,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/2019111114131242.png)
ab压力测试。
安装apache2-utils
测试localhost:9090，1000个请求，每次100
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191111141729200.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191111142558376.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1aUZlaURlVHVvTmlhb0da,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191111142612982.png)
测试login
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191111142428136.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1aUZlaURlVHVvTmlhb0da,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191111142441623.png)
参数解释：
命令：
-n 执行的请求数量

-c 并发请求个数

-t 测试所进行的最大秒数

-p 包含了需要POST的数据的文件

-T POST数据所使用的Content-type头信息

-k 启用HTTP KeepAlive功能，即在一个HTTP会话中执行多个请求，默认时，不启用KeepAlive功能

返回：
Server Software: 服务器软件版本

Server Hostname: 请求的URL

Server Port: 请求的端口号

Document Path: 请求的服务器的路径

Document Length: 页面长度 单位是字节

Concurrency Level: 并发数

Time taken for tests: 一共使用了的时间

Complete requests: 总共请求的次数

Failed requests: 失败的请求次数

Total transferred: 总共传输的字节数 http头信息

HTML transferred: 实际页面传递的字节数

Requests per second: 每秒多少个请求

Time per request: 平均每个用户等待多长时间

Time per request: 服务器平均用多长时间处理

Transfer rate: 传输速率

Connection Times: 传输时间统计

Percentage of the requests served within a certain time: 确定时间内服务请求占总数的百分比

