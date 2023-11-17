> 本文基于图解HTTP此书进行学习，感谢原作者以及翻译人员的辛勤劳动。

#  HTTP

## 1.了解Web网络基础以及什么是HTTP

## 2.HTTP协议及其网络通信过程

## 3.HTTP报文内信息

### 3.1HTTP状态码

## 4 与HTTP协作的服务器，代理，网关，隧道

## 5.HTTP首部

> HTTP 协议的请求和响应报文中必定包含 HTTP 首部，只是我们平时在使用 Web 的过程中感受不到。

### 5.1HTTP报文首部

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231118014824.png)

HTTP 协议的请求和响应报文中必定包含 HTTP 首部。首部内容为客户端和服务器分别处理请求和响应所需要的信息。对于客户端用户来说，这些信息中的大部分内容都无须亲自查看。 报文首部由几个字段构成。

**HTTP请求报文**

在请求中，HTTP 报文由方法、URI、HTTP 版本、HTTP 首部字段等部分构成。

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231118015430.png)

### 5.2HTTP首部字段

### 5.3HTTP/1.1通用首部字段

### 5.4实体首部字段

### 5.5Cookie服务的首部字段以及其他字段

## 6.确保Web安全的HTTPS

### 6.1HTTP的缺点

### 6.2HTTP + 加密 + 认证 + 完整性保护 = HTTPS

## 7.确认访问用户身份的认证

### 7.1什么是认证？认证方法有哪些？

#### 7.1.1BASIC认证

#### 7.1.2DIGEST认证

#### 7.1.3SSL客户端认证

#### 7.1.4基于表单认证

## 8.其他

### HTTP的功能追加协议

#### 基于HTTP的协议

#### 消除HTTP瓶颈的SPDY

#### 使用浏览器进行全双工通信的WebSocket

#### HTTP/2.0

#### Web服务器管理文件的WebDav

### 构建Web内容技术

#### HTML

#### 动态HTML

#### Web应用

#### 数据发布的格式及语言

### Web攻击技术

#### 针对Web的攻击技术

#### 因输出值转义不完全引发的安全漏洞

#### 因设计或设计上的缺陷引发的安全漏洞

#### 其他安全漏洞







