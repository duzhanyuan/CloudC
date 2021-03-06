# CloudC By Go

一个Golang写的多协程网络爬虫

## 结构

1. 任务队列
2. 分发器
3. 工作协程
4. 数据库模板
5. 集群处理器
6. 工作者协程
7. IO复用协程

### 任务队列


`[{
	"type":"get",		
	"cookie":{
		"need":"no"
		},
	"range":"url",		
	"url":"http://ke.qq.com/course/list",
	"params":[
		{
			"name":"mt",
			"type":"int",	
			"range":[{
				"start":1001,
				"end":1009}]	
		},
		{
			"name":"st",
			"type":"int",
			"range":[{
				"start":2004,
				"end":2008}]
		},
		{
			"name":"tt",
			"type":"int",
			"range":[{
				"start":3026,
				"end":3030}]
		}],
	"pattern":"<li class=\"course-card-item\"><a .*? href=\"(.*?)\".*?>.*?<img src=\"(.*?)\".*?title=\"(.*?)\"",
	"databaseTemplate":"tecentKt"
}]`

1. type

`type`参数决定了CloudC如何处理该任务，他的取值如下：
'get':简单Get，适用与大多数页面，同时也是极为高效的处理方法
'phantomGet':使用phantomJS模拟加载页面，适用于有大量复杂cookie-session请求，或者是使用js显示内容，或使用ajax显示内容的页面，典型代表：网易云课堂

### 数据库模板

>dt/tencentKt.dt

`{
	"database":"CloudKt",
	"table":"videos",
	"bind":["href","img","title"]
}`

会将正则匹配（pattern）取得的三个，会插入CloudKt库的videos表的href，img，title字段

### 分发器

本系统支持分布式部署，集群爬虫，和本机爬虫，在config.json中可以设置：

`{
	"workers":7,
	"servers":{
		"num":1,
		"serverPool":[
			"127.0.0.1"
		]
	}
}`

workers为本机同时运行的协程数（建议设置为核心数 × 每个核心线程数（1/2) - 1)，servers下的num为服务器数量，serverpool为服务器池

#### 分发策略

在单机模式下，采用基于任务的分发为主，基于查询的分发为辅，在查询有多级嵌套的形式切请求数目巨大的情况下（比如上文引用的任务，三层参数，如果每个参数范围在100，总共就是1000000个请求），我们会将请求也分发到多个协程
在集群模式下，采用完全基于任务的分发

### 虚拟化处理（云计算）

为进一步提高CPU利用率，采用Docker分发CPU资源