title: Linux curl 命令
date: 2018/09/22
categories:
- Linux
tags:
- Linux
---
在工作中经常见到`curl`这个命令，当客户端或者前端某些页面显示不对时，服务端的同学往往先通过`curl`命令进行排查。因此，不论是开发同学还是测试同学，都需要对此命令有足够的了解。
## 简单介绍
`curl` 是一个可以在命令行下使用的文件传输工具，它可以发起一个 `HTTP`(`GET` 或者 `POST`) 请求，它支持文件的上传和下载。按传统，习惯称`curl`为下载工具。作为一款强力工具，`curl`支持包括`HTTP、HTTPS、ftp`等众多协议，还支持`POST、cookies`认证、从指定偏移处下载部分文件、用户代理字符串、限速、文件大小、进度条等特征。

## 常用方法

尝试使用`curl`命令去访问谷歌主页：
```
curl www.google.com
```
得到返回结果如下：
![curl 访问 Google](https://github.com/taoclouds/taoclouds.github.io/blob/Source/Source/images/curl_google.jpg)

可以看到，返回结果直接就是网站的源码。。。可以使用`-o`命令将此网页保存下来。
```
curl -o Google.html www.google.com
```
但是，我们从上面的命令得到的谷歌的主页内容显然是不对的。这是因为谷歌网址做了自动跳转（并不是我没有科学上网哦）。因此，对于这种使用了重定向的网址，我们可以使用`-L`命令，将请求重定向到正确的位置就可以啦。这样便可以得到谷歌主页的内容。

有的时候，我们需要分析服务器给我们的返回内容，即`Http Response`的内容，分析返回内容有助于我们排查 `bug` 的原因。使用`-i`命令可以显示`response`的返回内容：
```
curl -i -L www.google.com
```
![返回内容](https://github.com/taoclouds/taoclouds.github.io/blob/Source/Source/images/response.jpg)
这样便可以查看服务器的返回内容，包括状态码等一些信息。不过需要注意的是，`-i`参数只显示返回头的信息。具体返回体的信息是看不到的。

如果我们需要查看一次`HTTP`通信的过程是怎样的，那么使用`-v`命令即可，它可以显示一次`http`通信的过程，包括端口链接和`request`的头信息。当然，`curl`还支持`--trace`命令，使用它即可查看整个通信的过程，也可以将其写入到指定的文件中：`--trace output.txt`。

## 使用`GET`/`POST`发送数据
使用`GET`或者`POST`可以将数据发送给指定服务器，简单点说，前者是将数据携带在链接后，后者需要用到`--data`参数，将数据与链接分开来。
```
GET: curl www.xxx.com/para?data=***
POST: curl -X POST --data "data=xxx" www.xxx.com/para
```
`curl`还支持给数据进行编码，将`--data`替换为`--data-urlencode`即可。`curl`默认是`GET`，可以使用`-X`参数来更改请求方式。

## 上传文件
对于本地文件，可以这样将它传到远端`ftp`服务器。
```
curl -T local/filename.txt xxxx.com/fileDoc
```
如果是提交表单，那么可以这样：
```
curl --form upload=@localfilename --form press=OK xxxx.com/url_link
```
## 其他
当你需要告诉服务器你是从哪里跳入这个界面时：
```
curl --referer http://www.example.com http://www.example.com
```
此外，`curl`还可以模拟手机或者时其他设备，使用`user agent`字段即可，一般服务器会根据这个值来判断当前请求的设备，然后返回不同的网页，以此来适应屏幕。
```
curl --user-agent "User Agent" [URL]
```
有些网站要求你必须登录，所以我们可以使用`--cookie`命令，带上登录的`cookie`：
```
curl --cookie "name=xxx" www.example.com
```
`cookie`值是服务端返回的，只要你的账号和密码对的，服务端会乖乖给你返回一个可用 `cookie`的。
