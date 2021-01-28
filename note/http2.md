# 快速了解Http/2

## HTTP简介
1.客户端（浏览器）与服务器之间一问一答的交互过程必须遵循一定的规则，这个规则就是HTTP协议。
  
2.HTTP是TCP/IP协议中的一个应用层协议，用于定义数据交换的过程以及数据本身的格式。  

3.版本：HTTP/0.9  HTTP/1.0  HTTP/1.1  HTTP/2

## HTTP/1.0会话方式
如图:  

![http1](https://raw.githubusercontent.com/liuyuai/myBlog/master/assets/http1.png 'http')

客户端与服务器的连接过程是短暂的，每次连接只处理一个请求和响应。每一个请求的访问，都需要单独建立一个连接。

## HTTP/1.1会话方式（持久连接）
如图:  

![http2](https://raw.githubusercontent.com/liuyuai/myBlog/master/assets/http2.png 'http')

HTTP/1.1持久连接在默认情况下是激活的。当连接建立后，除非客户端通知服务器关闭连接外，连接永远不会关闭。

持久连接发送的请求完成后，可直接发送在队列中等待的请求。


### 持久连接的优势
HTTP1.0  

![http3](https://raw.githubusercontent.com/liuyuai/myBlog/master/assets/http3.png 'http')

Http2.0  

![http4](https://raw.githubusercontent.com/liuyuai/myBlog/master/assets/http4.png 'http')

## HTTP报文

HTTP报文是简单的格式化数据块。

1.对报文进行描述的起始行(start line),请求的基本信息（url，method，状态码，状态码信息）  

2.包含属性的首部(header),请求的附加信息（客户端环境，cookie，缓存信息等）  

3.包含数据的主体部分(body),HTTP请求真正想要传输的部分。

具体案例如下:

![http5](https://raw.githubusercontent.com/liuyuai/myBlog/master/assets/http5.png 'http')


## HTTP/2新特性 ---头部压缩
 
### 为什么要压缩？

随着web功能越来越复杂，每个页面的请求也越来越多。一个页面可能发送的请求数几十个甚至上百个请求。每次都要传输 UserAgent、Cookie 这类不会频繁变动的内容。
这么多的请求导致消耗在头部的流量越来越多。

### 技术原理--静态字典

维护一份相同的静态字典(Static Table)，包含常见的头部名称，以及特别常见的头部名称与值的组合；对于完全匹配的头部键值对，例如 :method :GET ，可以直接使用一个索引表示。

![http6](https://raw.githubusercontent.com/liuyuai/myBlog/master/assets/http6.png 'http')

### 技术原理--动态字典

维护一份相同的动态字典(Dynamic Table),用于动态地添加内容；

![http7](https://raw.githubusercontent.com/liuyuai/myBlog/master/assets/http7.png 'http')

### 技术原理--哈夫曼编码

对于静态、动态字典中不存在的非标准的头部内容，还可以使用哈夫曼编码来减小体积。

![http8](https://raw.githubusercontent.com/liuyuai/myBlog/master/assets/http8.png 'http')

## HTTP/2新特性----多路复用(Multipexing )

### 什么是多路复用

在单个TCP/IP连接上实现同时进行多个业务单元数据的传输。


### 优点

解决HTTP/1的线头阻塞问题。实现并发请求，提高访问速度。

![http9](https://raw.githubusercontent.com/liuyuai/myBlog/master/assets/http9.png 'http')


## HTTP/2新特性----服务端推送(Server Push)

### 什么是服务端推送
1.HTTP/2 允许服务器为一次客户端请求发送多个响应(并行的)  

2.服务器必须遵循请求 - 响应机制，借着对请求的响应推送资源。也就是说，服务器并不能无缘无故推送流；

![http10](https://raw.githubusercontent.com/liuyuai/myBlog/master/assets/http10.png 'http')

## HTTP/2给WEB前端带来的影响

1.合并请求，降低请求次数，已经成为了过去时  

2.WEB前端已经进入组件化模块化开发阶段，某一个页面说依赖的js文件可能会有多个。

3.因为HTTP/1.1时代，报文头部的信息冗余以及连接队列延时的问题，所以会将多个文件进行合并，统一发送一次请求。

![http11](https://raw.githubusercontent.com/liuyuai/myBlog/master/assets/http11.png 'http')

服务端推送实现预加载:

![http12](https://raw.githubusercontent.com/liuyuai/myBlog/master/assets/http12.png 'http')

![http13](https://raw.githubusercontent.com/liuyuai/myBlog/master/assets/http13.png 'http')

客户端对HTTP/2的支持情况

![http14](https://raw.githubusercontent.com/liuyuai/myBlog/master/assets/http14.png 'http')


## 补充

### SPDY

SPDY是Google开发的下一代网络协议，它并不是一种用于替代HTTP的协议，而是对HTTP协议的增强。谷歌表示，引入SPDY协议后，在实验室测试中页面加载速度比原先快64%，并且应用到Gmail中，目前业界支持SPDY的服务器有Netty和Nginx。

这也是HTTP2的早期模型。




















