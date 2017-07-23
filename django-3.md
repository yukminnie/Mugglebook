# django-3



---
# 与服务器对话
### **request方法**

```
get 
post
head
put
options
connect
trace
delete

提交意见 交易后 删除头部连接
```

### **reqeust包含**

GET
```
GET /?q=yiciyuan HTTP/1.1

Host: duckduckgo.com

User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:54.0) Gecko/20100101 Firefox/54.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8

Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3

Accept-Encoding: gzip, deflate, br

Referer: https://duckduckgo.com/settings

Cookie: ak=-1

Connection: keep-alive

Upgrade-Insecure-Requests: 1


```
POST

```
POST / HTTP/1.1

Host: duckduckgo.com

User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:54.0) Gecko/20100101 Firefox/54.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8

Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3

Accept-Encoding: gzip, deflate, br

Content-Type: application/x-www-form-urlencoded

Content-Length: 10

Referer: https://duckduckgo.com/

Cookie: ak=-1; g=p

Connection: keep-alive

Upgrade-Insecure-Requests: 1


q	yiciyuan
```


### **response包含**

```
HTTP/1.1 304 Not Modified

Server: NWS_TCloud_S1

Connection: keep-alive

Date: Sat, 15 Jul 2017 05:31:05 GMT

Cache-Control: max-age=43200

Expires: Sat, 15 Jul 2017 17:31:05 GMT

Content-Type: text/html;charset=utf-8

Content-Length: 0

x-nws-log-uuid: 3037c904-e18b-45fc-8ef2-981337681cf8

```