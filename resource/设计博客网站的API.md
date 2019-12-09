##### 什么是API
API是Application Programming Interface（应用程序接口）的缩写，它是拿来描述一个类库的特征或是如何去运用它。如今很多人常常参考一种通过网络分享应用数据的HTTP API。
当人们开始去实现自己的API接口时，问题就出现了：没有一个标准的方法来命名URL，人们总是要参考API才得知它是如何运作的。
##### 什么是REST
REST是Representational State Transfer（表现层状态转移）的缩写，它是由罗伊·菲尔丁（Roy Fielding）提出的，是用来描述创建HTTP API的标准方法的，他发现这四种常用的行为（查看（view），创建（create），编辑（edit）和删除（delete））都可以直接映射到HTTP 中已实现的GET,POST,PUT和DELETE方法。
##### 资源
资源是 REST 中最关键的抽象概念，它们是能够被远程访问的应用程序对象。一个资源就是一个标识单位，任何可以被访问或被远程操纵的东西都可能是一个资源。
任何重要的资源都应该能够通过一个唯一的标识被访问。RESTful HTTP 使用 URI 来识别资源。URI 提供了 Web 通用的识别机制，它包含了客户端直接与被引用的资源进行交互时需要的所有信息。

REST API 围绕资源设计，资源是可由客户端访问的任何类型的对象、数据或服务。

每个资源有一个标识符，即，唯一标识该资源的 URI。

客户端通过交换资源的表示形式来与服务交互。 许多 Web API 使用 JSON 作为交换格式。

##### 博客网站API
网站地址：

```go
https://api.exampleblog.com
```
API访问通过HTTP，数据发送与接收使用json。
###### 登录

```go
curl -u username https://api.exampleblog.com
```
输入账户名称，之后输入相应的密码，登录成功后可以不受限地访问博客网站。

用户名不存在或者密码不匹配时，返回401 Unauthorized

```go
curl -i https://api.exampleblog.com -u invalid_name：invalid_password 
HTTP/1.1 401 Unauthorized
{
  "message": "Bad credentials",
}
```

###### 获取简介
在获取所有资源时，不需要显示资源的全部属性，比如在获取某用户的所有博客时，显示博客的简要内容。

```go
GET /user/articles
```
获得的响应大致为：

```go
Status：200 OK
--------------------------
{
  "total_quntity": 52,
  "items": [
    {
      "articleID": 234819,
      "language": "Chinese",
      "title": "servicecomputing",
      "author": {
        "name": "gaozhen",
        "id": 1734,
        "url": "blog url",
        "type": "User"
      },
      "article_url": "article url"
      "private": false,
      "description": "...",
      "visits": 105,
      "words": 1024,
      "content":"...."
      "created_at": "2018-05-23T23:42:51Z",
      "updated_at": "2019-05-16T17:25:15Z",
    },
    ...
  ]
}

```

###### 获取博客具体内容
当想要查看某篇具体博客时，获取该资源的全部属性。

```go
GET /user/articles/23481
```

###### 在列表中搜索文章

```go
GET /user/search/articles?title=servicecomputing&order=visits
```
有两个参数，文章名和排序方式,这样能找到名字为servicecomputing的文章，按阅读量排序。


###### 错误
发送无效或错误的JSON会得到400 Bad Request响应。

```go
HTTP/1.1 400 Bad Request
Content-Length: 45

{"message":"Body should be a JSON object"}
```
发送无效的字段得到 422 Unprocessable Entity响应。

```go
HTTP/1.1 422 Unprocessable Entity
Content-Length: 132

{
  "message": "Validation Failed",
  "errors": [
    {
      "resource": "Issue",
      "field": "title",
      "code": "missing_field"
    }
  ]
}
```
###### 发布文章
```go
POST /user/articles
data 
{
	"title":"",
	"content":"",
	"private": true/false,
	"description":""
}
curl -u -i -d '{"title":"...","content":"...","private":false,"description":"..."}' https://api.example.com/gaozhen/articles

```
###### 修改文章

```go
PUT /user/articles/{id}
data //发布文章的数据参数
{
	"title":"",
	"content":"",
}
curl -u -i -d '{"title":"...","content":"..."}' https://api.example.com/gaozhen/articles/15/

```

###### 删除文章

```go
DELETE /user/articles/{id}
```
###### 添加评论

```go
POST /user/articleID/comment
data
{
	"content":"..."
}
curl -i -u https://api.exampleblog/gaozhen/15/comment -d {"content":"..."}
```
