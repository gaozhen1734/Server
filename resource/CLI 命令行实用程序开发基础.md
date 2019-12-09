##### 概述
CLI（Command Line Interface）实用程序是Linux下应用开发的基础。正确的编写命令行程序让应用与操作系统融为一体，通过shell或script使得应用获得最大的灵活性与开发效率。Linux提供了cat、ls、copy等命令与操作系统交互；go语言提供一组实用程序完成从编码、编译、库管理、产品发布全过程支持；容器服务如docker、k8s提供了大量实用程序支撑云服务的开发、部署、监控、访问等管理任务；git、npm等都是大家比较熟悉的工具。尽管操作系统与应用系统服务可视化、图形化，但在开发领域，CLI在编程、调试、运维、管理中提供了图形化程序不可替代的灵活性与效率。
##### 开发实践
使用 golang 开发 开发[开发 Linux 命令行实用程序](https://www.ibm.com/developerworks/cn/linux/shell/clutil/index.html)中的 selpg。
##### 设计与实现
首先读取命令行输入，对参数进行处理，判断参数是否符合规范，然后读取文件，按照参数进行处理，将结果输出到指定位置，过程中如果出现错误则抛出。

定义储存参数的结构体，必须的参数包括开始页码及结束页码，可选的参数有自定义页长和根据换页符进行换页（两者互斥），以及输出地址。

```C#
type selpgArgs struct {
	startPage  int
	endPage    int
	inFileName string
	pageLen    int
	pageType   bool
	printDest  string
}
```
使用 pflag 替代 goflag 以满足 Unix 命令行规范，参考[Golang之使用Flag和Pflag](https://o-my-chenjian.com/2017/09/20/Using-Flag-And-Pflag-With-Golang/)。通过 pflag.Parse()方法让pflag 对标识和参数进行解析。之后就可以直接使用绑定的值。

```C#
func getArgs(args *selpgArgs) {

	pflag.IntVarP(&(args.startPage), "startPage", "s", -1, "Define startPage")
	pflag.IntVarP(&(args.endPage), "endPage", "e", -1, "Define endPage")
	pflag.IntVarP(&(args.pageLen), "pageLength", "l", 20, "Define pageLength")
	pflag.StringVarP(&(args.printDest), "printDest", "d", "", "Define printDest")
	pflag.BoolVarP(&(args.pageType), "pageType", "f", false, "Define pageType")
	pflag.Parse()

	argLeft := pflag.Args()
	if len(argLeft) > 0 {
		args.inFileName = string(argLeft[0])
	} else {
		args.inFileName = ""
	}
}
```
其中xxxVarP方法可以取出命令行参数的值，例如pflag.IntVarP(&(args.startPage), "startPage", "s", -1, "Define startPage")取出名字为startPage，在命令行中输入-s的值，指定其默认值为-1，之后是对这个参数的描述。通过Args()方法获取未定义的文件名参数。

获取完参数后，对其进行检查，即时抛出错误，将错误信息输出。

```C#
func checkArgs(args *selpgArgs) {

	if (args.startPage == -1) || (args.endPage == -1) {
		fmt.Fprintf(os.Stderr, "\n[Error]The startPage and endPage can't be empty! Please check your command!\n")
		os.Exit(2)
	} else if (args.startPage <= 0) || (args.endPage <= 0) {
		fmt.Fprintf(os.Stderr, "\n[Error]The startPage and endPage can't be negative! Please check your command!\n")
		os.Exit(3)
	} else if args.startPage > args.endPage {
		fmt.Fprintf(os.Stderr, "\n[Error]The startPage can't be bigger than the endPage! Please check your command!\n")
		os.Exit(4)
	} else if (args.pageType == true) && (args.pageLen != 20) {
		fmt.Fprintf(os.Stderr, "\n[Error]The command -l and -f are exclusive, you can't use them together!\n")
		os.Exit(5)
	} else if args.pageLen <= 0 {
		fmt.Fprintf(os.Stderr, "\n[Error]The pageLen can't be less than 1 ! Please check your command!\n")
		os.Exit(6)
	} else {
		pageType := "page length."
		if args.pageType == true {
			pageType = "The end sign /f."
		}
		fmt.Printf("\n[ArgsStart]\n")
		fmt.Printf("startPage: %d\nendPage: %d\ninputFile: %s\npageLength: %d\npageType: %s\nprintDestation: %s\n[ArgsEnd]", args.startPage, args.endPage, args.inFileName, args.pageLen, pageType, args.printDest)
	}

}
```
首先检查开始页与结束页数是否有输入，如果有，检查是否是正数，开始页是否小于结束页，然后检查是否同时设置了自定义页长和根据换页符换页，最后检查自定义页长是否为正数。

参数处理完毕后开始读取文件进行处理，如果没有给定文件名，则从标准输入中读取，然后调用checkFileAccess()检查文件是否存在。之后打开文件，如果没有-d参数，则直接在标准输出中输出，否则从指定的打印通道中输出。

```C#
func excuteCMD(args *selpgArgs) {
	var fin *os.File
	if args.inFileName == "" {
		fin = os.Stdin
	} else {
		checkFileAccess(args.inFileName)
		var err error
		fin, err = os.Open(args.inFileName)
		checkError(err, "File input")
	}

	if len(args.printDest) == 0 {
		output2Des(os.Stdout, fin, args.startPage, args.endPage, args.pageLen, args.pageType)
	} else {
		output2Des(cmdExec(args.printDest), fin, args.startPage, args.endPage, args.pageLen, args.pageType)
	}
}

func checkError(err error, object string) {
	if err != nil {
		fmt.Fprintf(os.Stderr, "\n[Error]%s:", object)
		panic(err)
	}
}

func checkFileAccess(filename string) {
	_, errFileExits := os.Stat(filename)
	if os.IsNotExist(errFileExits) {
		fmt.Fprintf(os.Stderr, "\n[Error]: input file \"%s\" does not exist\n", filename)
		os.Exit(7)
	}
}
```

将输入的文件按照页码读取输出到指定的位置（标准输出或管道），其中bufio提供了带缓存的io操作，ReadString(symbol)方法每次读取字符串到symbol。利用空接口作为fout，判断类型进行相应的操作。
```C#
func output2Des(fout interface{}, fin *os.File, pageStart int, pageEnd int, pageLen int, pageType bool) {

	lineCount := 0
	pageCount := 1
	buf := bufio.NewReader(fin)
	for true {

		var line string
		var err error
		if pageType {
			//If the command argument is -f
			line, err = buf.ReadString('\f')
			pageCount++
		} else {
			//If the command argument is -lnumber
			line, err = buf.ReadString('\n')
			lineCount++
			if lineCount > pageLen {
				pageCount++
				lineCount = 1
			}
		}

		if err == io.EOF {
			break
		}
		checkError(err, "file read in")

		if (pageCount >= pageStart) && (pageCount <= pageEnd) {
			var outputErr error
			if stdOutput, ok := fout.(*os.File); ok {
				_, outputErr = fmt.Fprintf(stdOutput, "%s", line)
			} else if pipeOutput, ok := fout.(io.WriteCloser); ok {
				_, outputErr = pipeOutput.Write([]byte(line))
			} else {
				fmt.Fprintf(os.Stderr, "\n[Error]:fout type error. ")
				os.Exit(8)
			}
			checkError(outputErr, "Error happend when output the pages.")
		}
	}
	if pageCount < pageStart {
		fmt.Fprintf(os.Stderr, "\n[Error]: startPage (%d) greater than total pages (%d), no output written\n", pageStart, pageCount)
		os.Exit(9)
	} else if pageCount < pageEnd {
		fmt.Fprintf(os.Stderr, "\n[Error]: endPage (%d) greater than total pages (%d), less output than expected\n", pageEnd, pageCount)
		os.Exit(10)
	}
}
```

当指定了输出的地址时需要用到os/exec包，将命令行的输入管道赋给fout，返回作为输出位置。

```C#
func cmdExec(printDest string) io.WriteCloser {
	cmd := exec.Command("lp", "-d"+printDest)
	fout, err := cmd.StdinPipe()
	checkError(err, "StdinPipe")
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr
	errStart := cmd.Start()
	checkError(errStart, "CMD Start")
	return fout
}
```
##### 程序测试

测试文件中包含两个换页符在5和23行。一共有31行，默认页长为20行。
selpg -s1 -e1 -l10 test.txt，将前10行打印至屏幕。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191007233056991.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1aUZlaURlVHVvTmlhb0da,size_16,color_FFFFFF,t_70)
selpg -s1 -e1 < test.txt，selpg读取标准输入，而标准输入被shell重定向为来自test.txt。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191007233829933.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1aUZlaURlVHVvTmlhb0da,size_16,color_FFFFFF,t_70)
selpg -s2 -e2 test.txt > output.txt，将第2页写入output.txt中。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191007234104666.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1aUZlaURlVHVvTmlhb0da,size_16,color_FFFFFF,t_70)
selpg -s1 -e0 test.txt 2>error.txt，将错误消息写入test.txt中。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191007234224734.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1aUZlaURlVHVvTmlhb0da,size_16,color_FFFFFF,t_70)
selpg -s3 -e5 -l4 test.txt >out.txt 2>error.txt
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191007234435765.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1aUZlaURlVHVvTmlhb0da,size_16,color_FFFFFF,t_70)
selpg -s1 -e2 -f test.txt
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191008000051201.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0h1aUZlaURlVHVvTmlhb0da,size_16,color_FFFFFF,t_70)
使用虚拟打印机插件输出。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191008021103464.png)
代码上传GitHub[传送门](https://github.com/Kate0516/ServiceComputing/tree/master/week5)

