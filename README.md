tailMail
========

日志文件监控及邮件通知小工具 V0.4


# 功能描述 #
监控日志类型文本文件的变化，如果有新增内容时，截取最新的10000个字符之内的内容，通过配置的邮件发送给相关人。

典型用途
+ 监控系统自动产生的异常日志、业务日志,并定时发送邮件。


# 使用说明 #

## 编译应用 ##
在 cmd 目录下，执行下面命令完成编译。 -o 参数为控制输出的文件名。

	$ go build -o=tailMail

## 配置和进度文件 ##
配置文件支持两种格式： toml(config.toml) 和 json(config.json)
默认是 toml.
启动参数的 -ct 就是配置这个的， 默认 toml。

### 初始化一个配置文件 ### 
执行下面命令可以产生一个空白的配置文件。配置的内容是 /config/initConfig.go 中配置的参数。

	$ ./tailMail -i=true

###  配置文件 
配置文件指 config.toml 或 config.json 文件。   
这里目前发送邮件的配置密码是瞎写的，使用者需要修改成自己的账户密码。

json格式配置文件的校验请使用： http://jsonlint.com/

toml 文件的注释符是 #    
 # This is a full-line comment    
key = "value" # This is a comment at the end of a line
请参考： https://github.com/toml-lang/toml    


###  进度文件 progress.toml。  
测试时进度文件是可以随时删除的， 删除掉意味着下次当新文件来处理。

###  每日统计文件 stat_20160524.toml。  
某天统计数据的文件。
这个文件可以被删除，这样昨天的报告就无法发送了。
报告发送完毕后，会自动删除昨天的统计数据文件。

###  模版文件
####  通知模板文件，
template.html 文件是有变更被监控到时，发送的邮件正文模版文件。
可以根据自己的情况进行定制修改。

####  每天统计报表邮件模版
templateStat.html 文件是每日统计报表邮件正文的模版文件。
可以根据自己的情况进行定制修改。


## 命令参数 ##

### -o 日志输出参数 
tailMail 本身的日志输出是否要输出到文件，默认 false， 如果设置成true 则输出到当前执行目录下，每天一个文件。
命令例子：

	$ tailMail -o=true 

### -p 配置执行目录参数
当我们部署在 crontab 时，由于 crontab 的执行目录是crontab的当前目录，不是我们期望的代码部署目录，这时候需要指定这个参数。
不指定这个参数，执行时，则直接取当前目录。

使用 crontab 配置时，建议 -o -p 都使用。

下面的例子就带了这两个参数  

	$ /Users/ghj1976/project/mygocode/src/github.com/ghj1976/tailMail/cmd/tailMail -o=true -p=/Users/ghj1976/project/mygocode/src/github.com/ghj1976/tailMail/cmd

###  -i 初始化配置文件参数
这个参数会产生一个参考的配置文件。
注意，这里的邮箱、密码都是瞎写的，需要改成自己需要的。

	$ ./tailMail -i=true

### -r 发送昨日统计报告参数，
命令如下：

	$ ./tailMail -r=true	


## crontab 配置例子 ##


注意，这里路径都需要完整路径，这样才能避免发生 crontab 当前目录跟执行文件目录不一致问题。

*/5 * * * * root /Users/ghj1976/project/mygocode/src/github.com/ghj1976/tailMail/cmd/tailMail -o=true -p=/Users/ghj1976/project/mygocode/src/github.com/ghj1976/tailMail/cmd

* 3 * * * root /Users/ghj1976/project/mygocode/src/github.com/ghj1976/tailMail/cmd/tailMail -r=true -p=/Users/ghj1976/project/mygocode/src/github.com/ghj1976/tailMail/cmd


http://linuxtools-rst.readthedocs.org/zh_CN/latest/tool/crontab.html

## 监控文件模版使用 ##

在配置文件中，如果配置的 FileNameUseTemplate 为 false， 则不启用文件名模版， 如果是 true ，则启用。 比如下面的部分配置就是启用了。
"FileName": "/wangapp/tomcat-wxmember/logs/localhost.{{formatNow \"2006-01-02\"}}.log",
"FileNameUseTemplate": true,

这里的 formatNow 参数接受的值是 go 时间格式描述符。
go 的格式化时间是用的一个特殊的时间做格式化的，参考如下：
2006-01-02T15:04:05


# 变更历史 #

+ V0.1 
	+ 在畅游时（2013年）完成第0.1版
+ V0.2 
	+ 在微智全景时(2015-12-21)，由于邮箱使用的是SSL方式才能发送，重构出第0.2版。
+ V0.3 
    + 配置文件默认修改为 toml 格式 (2015-12-24)。
    + 调整截取算法，确保完整的截取一行（如果发生这个逻辑，丢弃第一行）（2015-12－29）。
+ V0.4 
	+ 增加每日发送统计报告邮件功能，config.toml 增加服务器配置、启动每日统计报告邮件功能配置 ,
	 templateStat.html 为发送每日统计报告邮件的模版文件。
	+ 分功能重构代码， 避免大量代码混杂在一起。
	+ 放弃对 json 格式的配置文件的支持， 只支持 toml 格式的配置文件、进度文件。
