# 网络

## defer async的 区别是什么

  defer是在HTML解析后才会执行的，如果有多个，按加载顺序依次执行
  async是在加载完之后立即执行，如果是多个，执行顺序与加载顺序无关

## 缓存的分类

强缓存

  1. 特点： 不请求， 直接使用缓存
  2. 相关的HTTP头部字段：
     1. Expires：过期时间，是个绝对时间，下发的是服务器时间，比较用的是客户端的时间，所以会有偏差
     2. Cache-Control：过期时间，是个相对时间，优先级高，以客户端的相对时间为准，浏览器拿到资源之后的多少时间内都不会再去服务器请求

协商缓存

  1. 特点：浏览器不确定备份是否过期，需与服务器请求确认
  2. 相关的HTTP头部字段：
     1. Last-Modified/If-Modified-Since：服务器下发时间，客户端请求时带上下发时间，服务器判断文件是否过期。存在的问题服务器下发的时间难以定义
     2. Etag/If-None-Match：服务器下发hash值，客户端请求时带上hash值，服务器判断文件是否过期。优先级高

## cdn

  特点：在不同的地点缓存内容，将用户的请求定向到最合适的缓存服务器上去获取内容
  优点：解决Internet网络拥堵状况，提高用户访问网络的响应速度。

## 预解析DNS

  背景：DNS预解析会消耗前端的性能，优化建议是：减少DNS的请求次数，进行DNS预解析
  方式：是让具有此属性的域名自动在后台解析，从而减少用户的等待时间，提升用户体验

  ```html

  <meta http-equiv = "x-dns-prefetch-control" content="on">
  <!-- （强制打开a标签的DNS预解析，https下默认关闭） -->
  <link rel="dns-prefetch" href="//host_name_to_prefetch.com"/>

  ```

## 同源策略

同源：协议，域名，端口
  非同源的限制：

  1. cookie、localStorage、indexDB无法读取
  2. DOM无法获得
  3. Ajax 请求不能发送
  
  前后端通信
    Ajax ： 同源下的通信方式
    WebSocket：不受同源策略限制
    CORS：支持跨域通信，也支持同源通信

## 状态码表示的含义

1. 1XX：指示信息：请求已接收，继续处理
2. 2XX：成功，请求已被成功接收
3. 3XX：重定向，完成请求需要进一步的操作
4. 4XX：客户端错误，请求有语法错误或请求无法实现
5. 5XX：服务器错误：服务端未能实现合法的请求

## 常用http 状态码

1. 200：OK，客户端请求成功
2. 206：Partial Content：客户端发送一个带有Range头的GET请求，服务器完成了他
3. 301：Moved Permanently：所请求的页面已转移至新的URL
4. 302：Found：所请求的页面已经临时转移到新的URL
5. 304：Not Modified：客户端有缓存的文档，并发出一个条件性的请求，服务器告诉客户端，原来的缓存的文档可以继续使用
6. 400：Bad Request：客户端请求有语法错误，不能被服务器所理解
7. 401：Unauthorized：请求未经授权，必须与WWW-Authenticate报头域一起使用
8. 403：Forbidden：对被请求页面的访问被禁止
9. 404：Not Found：请求资源不存在
10. 500：Internal Server Error：服务器发生不可预期的错误
11. 503：Server Unavailable：请求未完成，服务器临时过载或宕机

## tcp的三次握手四次挥手

### 三次握手

  1. 首先客户端向服务端发送一个带有 SYN 连接请求标志，以及随机生成的序号 seq=x 的报文，等待服务端确认
  2. 服务端收到报文后返回一个 seq=y，ACK=x+1 的报文给客户端以确认连接请求
  3. 客户端收到确认后，检查ACK是否为 x+1，如果正确则发送带有 ACK=y+1 的报文给服务端。服务端检查ack是否为 y+1，如果正确则连接建立成功，客户端和服务端进入ESTABLISHED状态，完成三次握手，随后客户端与服务端之间可以开始传输数据了

### 四次挥手 客户端和服务端都可以先开始断开连接

  1. 客户端发送带有 FIN 标识，seq=M 的报文给服务端，请求通信关闭
  2. 服务端收到信息后，回复 ACK=M+1 的报文给客户端，答应关闭通信请求
  3. 服务端发送带有 FIN 标识，seq=N 的报文给客户端，也请求关闭通信
  4. 客户端回应 ACK=N+1 给服务端，答应关闭服务端的通信请求

## http2 特点

1. 二进制分帧
2. 多路复用
3. 头部压缩
4. 请求优先级
5. 服务端推送

## https 通讯过程

1. 客户端发出https请求， 请求服务端建立ssl连接
2. 服务端收到请求， 申请或自制数字证书， 得到公钥和服务器私钥， 并将公钥发给客户端
3. 客户端验证公钥， 不通过验证则发出警告， 通过验证则产生一个随机的客户端私钥
4. 客户端将公钥和客户端私钥进行对称加密后传给服务端
5. 服务端收到加密内容后，通过服务端私钥进行非对称解密， 得到客户端私钥
6. 服务端将客户私钥和内容进行对称加密， 并将加密内容发送给客户端
7. 客户端收到加密内容后，通过客户端私钥进行对称解密，得到内容

## http编码--压缩

为什么要对内容进行编码

  编码的目的就是为了压缩报文实体内容的大小，而通过压缩服务器响应报文传输的内容实体，在一定程度上就可以加快响应的速度

HTTP 的“压缩协议”

  请求头中的 `Accept-Encoding`
    客户端为了告知服务端当前支持的压缩编码，可以在请求头中，增加 Accept-Encoding 这个头部字段，用来指定当前客户端支持的压缩编码，如果有多个可以使用逗号 , 进行分割。
    为了满足优先级，其实是可以通过 `,` 分割的顺序来指定的。HTTP 协议中，还可以使用 Q 值来说明编码的优先级，Q 值的取值范围是 0.0 ~ 1.0。0.0 表示客户端不想接受此编码，而 1.0 则表示希望使用此编码

  响应头中的 `content-encoding`
    服务端为了在响应报文里体现当前对内容压缩使用的编码格式，会在响应头中使用 Content-Encoding 标记，它是一个明确值，所以只可能有一个
    编码的目的就是为了压缩，所以当服务端选择压缩内容实体的时候，同时还会修改 `Content-Length` 来明确表示当前实体被编码压缩后的长度

HTTP 的编码类型

  比较常用的算法有：
  
  1. gzip：表明实体采用 GNU zip 编码。
  2. compress：表明实体采用 Unix 的文件压缩程序。
  3. deflate：表明使用是用 zlib 的格式压缩的。
  4. br：表明实体使用 Brotli 算法的压缩格式。
  5. identity：表明没有对实体进行编码，为默认值。

关于gzip

  gzip 编码是采用的 GNU Zip 编码，是一种无损的压缩算法，用于减少传输报文实体的大小，它是可逆的压缩算法，不会导致信息损失

  gzip 的压缩效率相对较高，并且使用也是最为广泛的，我们在工作中如果不特殊说明，说到的 HTTP 压缩，通常就是指的 gzip

  gzip 的原理，简单来说，就是会去扫描整个文本的字符串，找到一样的字符串，就只保留一个并分配一个标识，然后将其他相同的字符串使用这个标识替换，使整个文件变小。在还原的时候，只需要将每个标识代表的字符串，替换还原，就可以还原成最初的内容实体

  gzip 具体能压缩多少，完全取决于压缩的实体内容，内容文本中，包含越多相同的字符串，压缩率就越高，相反则越低。在理想状态下，gzip 的压缩率能高达 70%

## jsonp 原理

jsonp是一种跨域通信的手段，它的原理其实很简单

1. 首先是利用script标签的src属性来实现跨域
2. 通过将前端方法作为参数传递到服务器端，然后由服务器端注入参数之后再返回，实现服务器端向客户端通信
3. 由于使用script标签的src属性，因此只支持get方法

```js

// 前端代码

function jsonp(req){
    var script = document.createElement('script');
    var url = req.url + '?callback=' + req.callback.name;
    script.src = url;
    document.getElementsByTagName('head')[0].appendChild(script); 
}

function hello(res){
    alert('hello ' + res.data);
}
jsonp({
    url : '',
    callback : hello 
});

```

```js
// 服务端代码

var http = require('http');
var urllib = require('url');

var port = 8080;
var data = {'data':'world'};

http.createServer(function(req,res){
    var params = urllib.parse(req.url,true);
    if(params.query.callback){
        console.log(params.query.callback);
        //jsonp
        var str = params.query.callback + '(' + JSON.stringify(data) + ')';
        res.end(str);
    } else {
        res.end();
    }
    
}).listen(port,function(){
    console.log('jsonp server is on');
});


```
