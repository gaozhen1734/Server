##### 概述
>命令行实用程序并不是都象 cat、more、grep 是简单命令。go 项目管理程序，类似 java 项目管理 maven、Nodejs 项目管理程序 npm、git 命令行客户端、 docker 与 kubernetes 容器管理工具等等都是采用了较复杂的命令行。即一个实用程序同时支持多个子命令，每个子命令有各自独立的参数，命令之间可能存在共享的代码或逻辑，同时随着产品的发展，这些命令可能发生功能变化、添加新命令等。因此，符合 OCP 原则 的设计是至关重要的编程需求。
>任务目标
熟悉 go 命令行工具管理项目
综合使用 go 的函数、数据结构与接口，编写一个简单命令行应用 agenda
使用面向对象的思想设计程序，使得程序具有良好的结构命令，并能方便修改、扩展新的命令,不会影响其他命令的代码
项目部署在 Github 上，合适多人协作，特别是代码归并
支持日志（原则上不使用debug调试程序）

开闭原则（OCP）
Open Closed Principle
Software entities like classes, modules and functions should be open for extension but closed for modifications(一个软件实体如类，模块和函数应该对扩展开放，对修改关闭）。
就是说软件开发应该通过扩展来实现变化，而不是通过修改原有代码。
可以分为两个层次，对抽象定义的修改，比如对象公开的接口，包括方法的名称、参数与返回类型。接口就是标准，要保障接口的稳定，就应该对对象进行合理的封装。一般的设计原则之所以强调方法参数尽量避免基本类型，原因正在于此。
对具体实现的修改，原则上，要做到避免对源代码的修改，即使仅修改具体实现，也需要慎之又慎。这是因为具体实现的修改，可能会给调用者带来意想不到的结果，这一结果并非我们预期的，甚至可能与预期相反。
##### GO命令
使用go help获取最新命令说明。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191015221914929.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1aUZlaURlVHVvTmlhb0da,size_16,color_FFFFFF,t_70)
GOPATH：有三个子目录：
src存放源代码(比如：.go .c .h .s等)   按照golang默认约定，go run，go install等命令的当前工作路径（即在此路径下执行上述命令）。
pkg编译时生成的中间文件（比如：.a）golang编译包时。
bin编译后生成的可执行文件。
GOROOT：golang 的安装路径。
>2.2 go 命令分类
环境显示： version、env
构建流水线： clean、build、test、run、（publish/git）、get、install
包管理： list, get, install
杂项：fmt，vet，doc，tools …
具体命令格式与参数使用 go help [topic]
##### 准备知识或资源
>3.1 Golang 知识整理
这里推荐 time-track 的个人博客，它的学习轨迹与课程要求基本一致。以下是他语言学习的笔记，可用于语言快速浏览与参考：
《Go程序设计语言》要点总结——程序结构
《Go程序设计语言》要点总结——数据类型
《Go程序设计语言》要点总结——函数
《Go程序设计语言》要点总结——方法
《Go程序设计语言》要点总结——接口
以上仅代表作者观点，部分内容是不准确的，请用批判的态度看待网上博客。 切记：
GO 不是面向对象(OOP) 的。 所谓方法只是一种语法糖，它是特定类型上定义的操作（operation）
指针是没有 nil 的，这可以避免一些尴尬。 p.X 与 v.x (p 指针， v 值) 在语义上是无区别的，但实现上是有区别的 p.x 是实现 c 语言 p->x 的语法糖
zero 值好重要

下载安装cobra
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191019165641327.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1aUZlaURlVHVvTmlhb0da,size_16,color_FFFFFF,t_70)
go语言会自动搜索GOPATH下src文件夹，在go-online中遇到如下问题时，
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191023140308666.png)
将头文件中的地址写完整，
从src开始写。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191023140359457.png)
使用pwd可以查看当前位置，使用echo $GOPATH可以查看GOPATH。
编译运行。
![在这里插入图片描述](https://img-blog.csdnimg.cn/201910231401454.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1aUZlaURlVHVvTmlhb0da,size_16,color_FFFFFF,t_70)
先创建注册和登陆命令。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191023141528258.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1aUZlaURlVHVvTmlhb0da,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191023141708518.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1aUZlaURlVHVvTmlhb0da,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191023142531709.png)
注册后，用户信息保存在data/User.txt中，用户名密码正确则登录。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191023211053696.png)
登陆后可以创建会议。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191023214833389.png)
查询会议。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191023215006728.png)
注册新用户需要登出。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191023215046101.png)
项目地址：[传送门](http://139.9.57.167:20080/share/bmo5mc676kvs669u248g?secret=false)