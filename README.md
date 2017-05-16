[![Go Report Card](https://goreportcard.com/badge/github.com/qiniu/qfetch)](https://goreportcard.com/report/github.com/qiniu/qfetch)

# qfetch

### 简介
qfetch是一个数据迁移工具，利用七牛提供的[fetch](http://developer.qiniu.com/docs/v6/api/reference/rs/fetch.html)功能来抓取指定文件列表中的文件。在文件列表中，你只需要提供资源的外链地址和要保存在七牛空间中的文件名就可以了。

使用该工具进行资源抓取的时候，可以根据需要随时可以中断任务的执行，下次重新使用原命令执行的时候，会自动跳过已经抓取成功的资源。

### 适用场景

该工具适用的场景需要满足如下条件：

1. 资源所在的源站必须具有较大的可用带宽，这样可以在业务低峰期进行资源抓取
2. 根据源站可用带宽和文件的平均大小，以及能够进行抓取的时间段，计算出数据迁移所需要的时间是否满足需求

### 下载

**建议下载最新版本**

|版本     |支持平台|链接|
|--------|---------|----|
|qfetch v1.7|Linux, Windows, Mac OSX|[下载](http://devtools.qiniu.com/qfetch-v1.7.zip)|

### 安装

该工具不需要安装，只需要从上面的下载链接下载zip包之后解压即可使用。其中文件名和对应系统关系如下：

|文件名|描述|
|-----|-----|
|qfetch_linux_386|Linux 32位系统|
|qfetch_linux_amd64|Linux 64位系统|
|qfetch_linux_arm|Linux ARM CPU|
|qfetch_windows_386.exe|Windows 32位系统|
|qfetch_windows_amd64.exe|Windows 64位系统|
|qfetch_darwin_386|Mac 32位系统，这种系统很老了|
|qfetch_darwin_amd64|Mac 64位系统，主流的系统|

注意，如果在Linux或者Mac系统上遇到`Permission Denied`的错误，请使用命令`chmod +x qfetch`来为文件添加可执行权限。这里的`qfetch`是上面文件重命名之后的简写。

对于Linux或者Mac，如果希望能够在任何位置都可以执行，那么可以把`qfetch`所在的目录加入到环境变量`$PATH`中去。或者最简单的方法如下：

```
sudo mv qfetch /usr/local/bin
```
另外，由于本工具是一个命令行工具，在Windows下面请先打开命令行终端，然后输入工具名称执行，不要双击打开。如果你希望可以在任意目录下使用qfetch，请将qfetch工具可执行文件所在目录添加到系统的环境变量中。

### 使用
该工具是一个命令行工具，需要指定相关的参数来运行。

```
Usage of qfetch:
  -ak="": qiniu access key
  -sk="": qiniu secret key
  -bucket="": qiniu bucket
  -job="": job name to record the progress
  -file="": resource list file to fetch
  -worker=0: max goroutine in a worker group
  -log="": run log output file
  -check-exists=false: check whether file already exists in bucket, if exists, skip fetch
  -zone="nb": qiniu zone, nb, bc, hn or na0
```


|命令|描述| 必须指定 |
|--------|---------|-----------|
|ak|七牛账号的AccessKey，可以从七牛的后台获取|是|
|sk|七牛账号的SecretKey，可以从七牛的后台获取|是|
|bucket|文件抓取后存储的空间，为空间的名字|是|
|job|任务的名称，指定这个参数主要用来将抓取成功的文件放在本地数据库中，便于后面核对|是|
|file|待抓取资源链接所在文件的本地路径，内容由待抓取的资源外链和对应的保存在七牛空间中的文件名组成的行构成|是|
|worker|抓取的并发数量，可以适当地指定较大的并发请求数量来提高批量抓取的效率，可根据目标源站实际带宽和文件平均大小来计算得出|是|
|check-exists|在抓取之前检查空间是否存在同名文件，如果存在则跳过，不抓取，当指定这个参数的时候为true，不指定为false|否|
|log|抓取过程中打印的一些日志信息输出文件，如果不指定，则输出到终端|否|
|zone|请求发送到的入口机房，可以不指定，默认为nb，即七牛宁波机房；可选设置为bc，即七牛北京机房|否|

**多机房支持**

|机房|zone值|
|----|----|
|华东|nb|
|华北|bc|
|华南|hn|
|北美|na0|

**模式一:**

上面的`file`参数指定的待抓取资源链接所在文件的行格式如下：

```
文件链接1\t保存名称1
文件链接2\t保存名称2
文件链接3\t保存名称3
...
```

其中`\t`表示Tab分隔符号。

例如：

```
http://img.abc.com/0/000/484/0000484193.fid	2009-10-14/2922168_b.jpg
http://img.abc.com/0/000/553/0000553777.fid	2009-07-01/2270194_b.jpg
http://img.abc.com/0/000/563/0000563511.fid	2009-03-01/1650739_s.jpg
http://img.abc.com/0/000/563/0000563514.fid	2009-05-01/1953696_m.jpg
http://img.abc.com/0/000/563/0000563515.fid	2009-02-01/1516376_s.jpg
```

上面的方式最终抓取保存在空间中的文件名字是：

```
2009-10-14/2922168_b.jpg
2009-07-01/2270194_b.jpg
2009-03-01/1650739_s.jpg
2009-05-01/1953696_m.jpg
2009-02-01/1516376_s.jpg
```

**模式二:**

上面的`file`参数指定的待抓取资源链接所在文件的行格式如下：

```
文件链接1
文件链接2
文件链接3
...
```

上面的方式也是支持的，这种方式的情况下，文件保存的名字将从指定的文件链接里面自动解析。

例如：

```
http://img.abc.com/0/000/484/0000484193.fid
http://img.abc.com/0/000/553/0000553777.fid
http://img.abc.com/0/000/563/0000563511.fid
http://img.abc.com/0/000/563/0000563514.fid
http://img.abc.com/0/000/563/0000563515.fid
```

其抓取后保存在空间中的文件名字是：

```
0/000/484/0000484193.fid
0/000/553/0000553777.fid
0/000/563/0000563511.fid
0/000/563/0000563514.fid
0/000/563/0000563515.fid
```


### 日志
抓取成功的文件在本地都会写入以`job`参数指定的值为名称的本地leveldb数据库中。该数据库名称格式为`<jobName>.job`，由于该leveldb名称以`.`开头，所以在Linux或者Mac系统下面是个隐藏文件。在整个文件索引都抓取完成后，可以使用[leveldb](https://github.com/jemygraw/leveldb)工具来导出所有的成功的文件列表，和原来的列表比较，就可以得出失败的抓取列表。上面的方法也可以被用来验证抓取的完整性。

另外抓取过程中发现回复为404的列表，单独放到`.<jobName>.404.job`的leveldb数据库中，抓取结束之后可以导出这部分数据做检查。

其中`<jobName>`就是参数`-job`所指定的名字，左右两边的`<`和`>`只是表示这个是参数，实际不存在。

### 示例
抓取指令为：

```
qfetch -ak 'x98pdzDw8dtwM-XnjCwlatqwjAeed3lwyjcNYqjv' -sk 'OCCTbp-zhD8x_spN0tFx4WnMABHxggvveg9l9m07' -bucket 'image' -file 'diff.txt' -job 'diff' -worker 100 -log 'run.log' -check-exists
```
**注意：** Windows系统下面使用该工具时，指定的参数两边不需要单引号。

上面的指令抓取文件索引`diff.txt`里面的文件，存储到空间`piccenter`里面，并发请求数量`300`，任务的名称叫做`diff`，成功列表日志文件名称是`.diff.job`。另外由于该命令打印的报警日志输出到终端，所以可以使用`tee`命令将内容复制一份到日志文件中。

导出成功列表：

```
leveldb -export '.diff.job' >> success.list.txt
```

导出404列表：

```
leveldb -export '.diff.404.job' >> 404.list.txt
```

注意，上面任务的名字是`diff`，而任务对应的的leveldb的名字是`.diff.job`。

**经验**

一般来讲，如果是抓取任务的话，不一定需要导出最终成功的列表，只需要检查原始列表文件行数和抓取成功的文件行数一致就可以了。

使用下面方式获取列表行数：
```
$ wc -l diff.txt 
```

使用下面方式获取成功抓取数量：
```
$ leveldb -count '.diff.job'
```

然后比较一致即可，如果发现数量不一致，可以重新运行原始命令（设置太大并发的情况下，存在失败的可能性）。
只要最后的结果没有错误或者都是404的错误，那么就是抓取成功了。404的错误可以后面跟进解决。


### 帮助
如果您遇到任何问题，可以加QQ群：343822521，我将乐意帮助您，非技术问题勿扰。
