---
layout: post
title: XMLHttpRequest Level 1
---

[XMLHttpRequest Level 1](http://www.w3.org/TR/XMLHttpRequest/)是W3C的草案，个人在看《Web 性能权威指南》时看到，试着翻译一下。

## 摘要
----

XMLHttpRequest规范定义了在客户端和服务器端传递数据的API，其用提供客户端的脚本化的功能。

> 注： W3C网站上的只是[XMLHttpRequest Living Specification](https://xhr.spec.whatwg.org/)的一个快照。

##1 Introduction

> 本节是非规范的。

XMLHttpRequest对象是用来下载资源的API。XMLHttpRequest的命名源自历史，并且已变的名不符其实。下面是一个简单的代码实例，处理从网络上下载下来的数据。

```javascript
function processData(data) {
  // taking care of data
}

function handler() {
  if(this.readyState == this.DONE) {
    if(this.status == 200 &&
       this.responseXML != null &&
       this.responseXML.getElementById('test').textContent) {
      // success!
      processData(this.responseXML.getElementById('test').textContent);
      return;
    }
    // something went wrong
    processData(null);
  }
}

var client = new XMLHttpRequest();
client.onreadystatechange = handler; // 这里其实就是将client作为this对象传递给handler函数
client.open("GET", "unicorn.xml");   // url 参数都不传?
client.send();    // 发送http请求
```

如果想要在服务器上记录消息，可以通过如下的代码设置: 

```javascript
function log(message) {
  var client = new XMLHttpRequest();
  client.open("POST", "/log");
  client.setRequestHeader("Content-Type", "text/plain;charset=UTF-8"); // 设置HTTP的请求头
  client.send(message);
}
```

或者，想要检查服务器上的文档的状态: 

```javascript
function fetchStatus(address) {
  var client = new XMLHttpRequest();
  client.onreadystatechange = function() {
    // in case of network errors this might not give reliable results
    if(this.readyState == this.DONE)
      returnStatus(this.status);
  }
  client.open("HEAD", address);  // 设置HEAD请求，不需要返回内容体
  client.send();
}
```

### 1.1 Specification history

XMLHttpRequest对象的初始定义来自WHATWG关于HTML的努力(基于Microsoft之前的实现)。并在2006年，工作移交给W3C。XMLHttpRequest的相关扩展，例如处理事件(progress events)和跨域请求(cross-origin requests)，在2011年之前，还是单独的草案(XMLHttpRequest Level 2)。自2011之后，两份草案合并成一份预标准，并在2012年回到WHATWG讨论组。

## 2 约定

本规范中的所有图表、样例以及注释都是非规范的，一些非规范的章节显式标注。除此以外，其他的内容都是规范的。

在规范描述中出现的"MUST", "MUST NOT", "REQUIRED", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY"以及"OPTIONAL"关键字，在[RFC2119]()已详细解释。为了可读性，本文档中这些词不再以大写形式出现。

### 2.1 Extensibility

用户代理(User agents), 工作小组以及其他的兴趣组，建议去WHATWG社区进行讨论。

## 3 术语

本规范对术语使用交叉连接，例如 DOM, DOM Parsing and Serialization, Encoding, Fetch, File API, HTML, HTTP, URL, Web IDL以及XML ，一律写作[DOM] [DOMPS] [ENCODING] [FETCH] [FILEAPI] [HTML] [HTTP] [URL] [WEBIDL] [XML] [XMLNS]。交叉链接使用HTML的排版格式。

本规范中的用户认证包含：cookies， HTTP授权，客户端SSL认证。但是，不包含代理授权和源头部。[COOKIES]

## 4 XMLHttpRequest的接口

下文的XMLHttpRequest接口使用的是一种类似接口定义语言描述的。

```javascript
[NoInterfaceObject, Exposed=(Window,Worker)]
interface XMLHttpRequestEventTarget : EventTarget {
  // 支持的事件处理器，共7个
  attribute EventHandler onloadstart;
  attribute EventHandler onprogress;
  attribute EventHandler onabort;
  attribute EventHandler onerror;
  attribute EventHandler onload;
  attribute EventHandler ontimeout;
  attribute EventHandler onloadend;
};

[Exposed=(Window,Worker)]
interface XMLHttpRequestUpload : XMLHttpRequestEventTarget {
};

enum XMLHttpRequestResponseType {
  "",             //空类型
  "arraybuffer",  //缓冲数组
  "blob",         //blob - 二进制大字段类型
  "document",     //xml文档
  "json",         //json数据
  "text"          //纯文本
};

[Constructor, Exposed=(Window,Worker)]
interface XMLHttpRequest : XMLHttpRequestEventTarget {
  // XMLHttpRequest新增的事件处理器
  attribute EventHandler onreadystatechange;

  // xhr对象的状态, 共5个状态
  const unsigned short UNSENT = 0;
  const unsigned short OPENED = 1;
  const unsigned short HEADERS_RECEIVED = 2;
  const unsigned short LOADING = 3;
  const unsigned short DONE = 4;
  readonly attribute unsigned short readyState;

  // 请求
  void open(ByteString method, USVString url);
  void open(ByteString method, USVString url, boolean async, optional USVString? username = null, optional USVString? password = null);
  void setRequestHeader(ByteString name, ByteString value);
           attribute unsigned long timeout;       // 耗时
           attribute boolean withCredentials;     // 身份验证
  readonly attribute XMLHttpRequestUpload upload; // 关联的独一无二的XMLHttpRequestUpload的对象
  void send(optional (Document or BodyInit)? body = null);
  void abort();

  // 响应
  readonly attribute USVString responseURL;  // 响应URL
  readonly attribute unsigned short status;  // 响应状态
  readonly attribute ByteString statusText;  // 状态相关的文本描述
  ByteString? getResponseHeader(ByteString name);  // 响应头
  ByteString getAllResponseHeaders();
  void overrideMimeType(DOMString mime);
           attribute XMLHttpRequestResponseType responseType;  // 响应类型
  readonly attribute any response;
  readonly attribute USVString responseText;  // 响应文本
  [Exposed=Window] readonly attribute Document? responseXML;
};
```

每个XMLHttpRequest对象都有一个与之关联的独一无二的XMLHttpRequestUpload对象。

### 4.1 Constructors

XMLHttpRequest对象拥有一个设置对象。

```javascript
client = new XMLHttpRequest() // Returns a new XMLHttpRequest object. 
```
注: 对象的创建存在这样的几种方法是- 对象字面值、new xxx、Object.create()。

`XMLHttpRequest()`初始化器必须执行如下的步骤:

- 将xhr赋值为new XMLHttpRequest
- 将xhr的设置对象映射到其全局接口所关联的对象
- 返回xhr. 

### 4.2 Garbage collection

An XMLHttpRequest object must not be garbage collected if its state is OPENED and the send() flag is set, its state is HEADERS_RECEIVED, or its state is LOADING and it has one or more event listeners registered whose type is one of readystatechange, progress, abort, error, load, timeout, and loadend.

以下情况中，XMLHttpRequest对象不会被垃圾回收: 

* stats为`OPENED`且设置了send()
* stats为`HEADERS_RECEIVED`
* status为`LOADING`并且注册了readystatechange, progress, abort, error, load, timeout以及loadend事件中的一个或多个

如果XMLHttpRequest对象已被gc，但其连接依然开着，用户代理(浏览器)必须终结该请求。

### 4.3 Event handlers

下表中列出了事件处理器(及其对应的事件处理类型)，这些事件处理器必须实现`XMLHttpRequestEventTarget`中的接口。

event handler |	event handler event type 
------------- | --------------------------------
onloadstart 	| loadstart
onprogress 	  | progress
onabort 	    | abort
onerror 	    | error
onload 	      | load
ontimeout 	  | timeout
onloadend 	  | loadend

如下的事件处理器(及其对应的事件类型)必须作为XMLHttpRequest对象的属性:

 event handler     |	event handler event type
------------------ | ---------------------------
onreadystatechange |	readystatechange

### 4.4 States

```javascript
client.readyState // 返回当前状态 
```

XMLHttpRequest可以处于一系列的状态。 readyState属性返回当前状态的属性，其可为如下的这些值: 

* UNSENT (值为 0)  对象已创建
* OPENED (值为 1)  成功调用了`open()`方法。在此状态下，请求头部可以使用`setRequestHeader()`进行设置，请求可通过`send()`方法发送
* HEADERS_RECEIVED (值为 2)  All redirects (if any) have been followed and all HTTP headers of the response have been received. 
* LOADING (值为 3)  响应体被接受 
* DONE (值为 4)  数据转换完成，或者转换时出错了(例如：无限重定向)

初始化后的XMLHttpRequest对象的状态为`UNSENT`。send()的flag表明send()方法被调用，其起初未设置，并在OPENED状态下使用。

### 4.5 Request

Each XMLHttpRequest object has the following request-associated concepts: request method, request URL, author request headers, request body, synchronous flag, upload complete flag, and upload events flag.

每个XMLHttpRequest对象都存在如下的请求相关的概念: 请求方法，请求URL，请求头，请求体，异步标记，上传完成标识以及上传事件标识。

请求头初始为空列表，请求体初始为null，同步标识、上传完成标识以及上传事件标识起初未设置。

可以致命的理由终止XMLHttpRequest对象的fetch算法，从而终止请求。

#### 4.5.1 The open() method

```javascript
/*
 * 如果请求方法为无效HTTP方法或URL不能被解析，则抛出SyntaxError异常
 * 如果请求方法为`CONNECT`, `TRACE`或`TRACK`，则抛出SecurityError异常
 * Throws an InvalidAccessError exception if async is false, the JavaScript global environment is a document environment, and either the timeout attribute is not zero, the withCredentials attribute is true, or the responseType attribute is not the empty string.
 */
client.open(method, url [, async = true [, username = null [, password = null]]]) // 设置请求方法，请求URL以及请求标识
```
如果javascript全局环境为文档环境时，不能将async参数设置为false，因为这回严重影响终端用户的体验。在开发者模式中的，用户代理(web浏览器)将会对种方式发出强烈警告，并抛出InvalidAccessError异常，从而将其从平台上移除。

`open(method, url, async, username, password)`方法需要运行如下的步骤:

1. 如果设置对象(settings object)的响应文档不完全活跃，则抛出InvalidStateError异常。
2. 将设置对象(settings object)的API base URL设为base的值
3. 将设置对象的起始源设置为source origin
4. 如果设置对象的API引用源为文档，则将其设置为referer source
5. 如果method不为HTTP方法(包含CONNECT, DELETE, GET, HEAD, OPTIONS, POST, PUT, TRACE 或 TRACK)，抛出SyntaxError异常
6. 如果method为被禁止的方法(CONNECT, TRACE或者TRACK)，则抛出SecurityError异常
7. 以base解析url的值，从而将其设置为URL的值。如果解析失败了，则抛出TypeError
8. 如果未提供async参数，则将其设置为true，并设置用户名和密码为null
9. 如果设置了parsedURL的相对标识，如果username和password均不为空，则将其设置到parsedURL的对应属性上
10. 如果async为false, JavaScript全局环境为文档环境，并且timeout属性不为0,withCredentials为true，responseType属性值不为空字符串，则抛出InvalidAccessError异常
11. 出现各种异常后，终结请求。否则，请求将蓄势待发，并设置如下的关联的变量:
    * 将method设置为请求方法
    * 将parsedURL设置为URL
    * 设置同步标识，如果async为false，则取消同步标识的设置
    * 清空请求头
    * 设置响应为network错误
    * 设置响应的ArrayBuffer、Blob、Document以及JSON对象为null
12. 如果状态不为OPENED, 运行如下步骤:
    * 将状态转为OPENED
    * 触发名为readystatechange的事件

#### 4.5.2 The setRequestHeader() method

```javascript
/* 组成请求的头部
 * 如果xhr的状态不为OPENED或者send()标志没有设置，则抛出InvalidStateError异常
 * 如果name或value不是合法的数据，则抛出SyntaxError异常
 */
client.setRequestHeader(name, value)
```

**注意**: 下面的算法表示某些请求头能不能设置由用户代理决定。此外，如果用户没有显式设置某些请求头，那么在`send()`方法前，用户代理将自己设置这些请求头。

`setRequestHeader(name, value)`方法必须执行如下的步骤: 

1. 如果状态不为OPENED，抛出InvalidStateError异常
2. 如果`send()`标志未设置，抛出InvalidStateError异常
3. 如果name和value非法，抛出SyntaxError异常
4. 空字节序列表示空的请求头
5. 如果name为禁用的HTTP请求头，则终止请求。ajax中禁用的请求头有: Accept-Charset, Accept-Encoding, Access-Control-Request-Headers, Access-Control-Request-Method, Connection, Content-Length, Cookie, Cookie2, Date, DNT, Expect, Host, Keep-Alive, Origin, Referer, TE, Trailer, Transfer-Encoding, Upgrade, User-Agent, Via以及那些以`Proxy-`和`Sec-`开头的请求头。
6. 将name/value组合成请求头部

下面的简单的代码演示了，相同的首部被设置两次的效果：

```javascript
// 如下的代码将在请求头中包含`X-Test: one, two`
var client = new XMLHttpRequest();
client.open('GET', 'demo.cgi');
client.setRequestHeader('X-Test', 'one');
client.setRequestHeader('X-Test', 'two');
client.send();
```

#### 4.5.3 `timeout`属性

```javascript
// 以毫秒计数。设置为非零时，将在给定的时间过去后，如果请求未完成，并且同步标志未设置，将触发timeout事件，或者抛出TimeoutError异常(在send方法中)。
// 当设置同步标识，且js的全局环境是document环境，为其赋值时将会抛出InvalidAccessError异常。
client.timeout
```

`timeout`属性必须要返回其值，其初始化的值为0。设置timeout属性必须要运行如下的步骤:

1. 当设置同步标识，且js的全局环境是document环境，抛出InvalidAccessError异常
2. 将其设置为新值

这意味着，但读取依然在进行时，就可设置timeout属性。如果发生那种情况，将依然以读取开始的时间作为度量。

#### 4.5.4 The withCredentials attribute

```javascript
/* 在跨源请求中，用户证书(user credentials)必须设置为true。当其被排除在跨源请求中且cookies在响应中被忽略时，设置为false。默认为false
 * 如果state为UNSENT、OPENED或设置send()标识时，给其赋值将抛出InvalidStateError异常
 * 如果设置同步标识且javascript的全局环境为文档环境，给其赋值将抛出InvalidStateError异常
 */
client.withCredentials
```

`withCredentials`属性必须返回值，其初，其值为false。withCredentials的赋值必须处理如下的步骤:

1. 如果state不为UNSENT或OPENED，则抛出InvalidStateError异常
2. 如果设置send()标志，则抛出InvalidStateError异常
3. 如果JavaScript全局对象为文档对象，且设置同步标志，抛出InvalidAccessError异常
4. 将其设置为给定的值

`withCredentials`属性在获取同源资源时，没有作用。估计其主要是用来验证请求的。

#### 4.5.5 The upload attribute

```javascript
// 返回关联的XMLHttpRequestUpload对象。当数据被传送到服务器时，可以用来收集传输信息。
client.upload
```
    Returns the associated XMLHttpRequestUpload object. It can be used to gather transmission information when data is transferred to a server. 

`upload`属性必须返回关联的XMLHttpRequestUpload对象，并且每个XMLHttpRequest都拥有一个关联的XMLHttpRequestUpload对象。

#### 4.5.6 The send() method

```javascript
// 启动请求，可选的参数提供请求对象。如果请求方法是`GET`或`HEAD`，则忽略参数。
// 如果state不为OPENED或设置了send()标志，则抛出InvalidStateError异常
client . send([data = null])
```

`send(data)` method must run these steps:

1. 如果state不为OPENED或设置了send()标志，则抛出InvalidStateError异常
2. 如果请求方法为GET或HEAD，将body设置为null
3. 如果data为空，跳到下一步。否则，将encoding、Content-Type设置为null，然后，根据body遵循如下的规则:
   * ArrayBufferView , 将data设置原始值
   * Blob，如果对象类型不为空，则将mine类型设置为其类型
   * Document 
     * 设置encoding为`UTF-8`
     * 如果body是HTML文档，则将 Content-Type设置为`text/html`或`application/xml`，并给Content-Type添加`;charset=UTF-8`
     * 将请求体设置为body的值，序列化，转换为utf-8编码Unicode，并进行utf-8编码。在序列化过程中，可能抛出任何异常，比如InvalidStateError异常
   * String, 将编码设置为`UTF-8`，设置请求体为body，并将设置mine类型为"text/plain;charset=UTF-8" 
   * FormData, 将请求体设置为对data运行`multipart/form-data`编码算法的结果，并将`utf-8`设置为显式编码类型。将mine类型设置为 "multipart/form-data;"，U+0020空白字符， "boundary="以及由`multipart/form-data`编码算法生成的边界字符串进行连接。
   
   如果请求头中包含`Content-Type`头部，且其值为有效的MIME类型(charset参数不为大小写敏感，且encoding不为空); 如果不包含`Content-Type`，则设置`Content-Type`为mine类型的值。
4. 如果设置同步标志，则释放存储互斥体
5. 重置 error标志，upload完成标志以及upload事件标志
6. 如果没有请求实体或其值为空，则设置upload完成标志
7. If the synchronous flag is unset and one or more event listeners are registered on the XMLHttpRequestUpload object, set the upload events flag.
8. If the synchronous flag is unset, run these substeps:

  *  Set the send() flag.
  *  Fire a progress event named loadstart.
  *  If the upload complete flag is unset, fire a progress event named loadstart on the XMLHttpRequestUpload object.
  *  Return the send() method call, but continue running the steps in this algorithm.

9. If the source origin and the request URL are same origin

      These are the same-origin request steps.

      Fetch the request URL from origin source origin, using referrer source as override referrer source, with the synchronous flag set if the synchronous flag is set, using HTTP method request method, taking into account the request entity body, list of author request headers, and the rules listed at the end of this section.

      If the synchronous flag is set

          While making the request also follow the same-origin request event rules.

          The send() method call will now be returned by virtue of this algorithm ending.
      If the synchronous flag is unset

          Make upload progress notifications.

          Make progress notifications.

          While processing the request, as data becomes available and when the user interferes with the request, queue tasks to update the response entity body and follow the same-origin request event rules.

    Otherwise

        These are the cross-origin request steps.

        Make a cross-origin request, passing these as parameters:

        request URL
            The request URL.
        request method
            The request method.
        author request headers
            The list of author request headers.
        request entity body
            The request entity body.
        source origin
            The source origin.
        referrer source
            The referrer source. 
        omit credentials flag
            Set if withCredentials attribute's value is false. 
        force preflight flag
            Set if the upload events flag is set. 

        If the synchronous flag is set

            While making the request also follow the cross-origin request event rules.

            The send() method call will now be returned by virtue of this algorithm ending.
        If the synchronous flag is unset

            While processing the request, as data becomes available and when the end user interferes with the request, queue tasks to update the response entity body and follow the cross-origin request event rules.

If the user agent allows the end user to configure a proxy it should modify the request appropriately; i.e., connect to the proxy host instead of the origin server, modify the Request-Line and send Proxy-Authorization headers as specified.

If the user agent supports HTTP Authentication and Authorization is not in the list of author request headers, it should consider requests originating from the XMLHttpRequest object to be part of the protection space that includes the accessed URIs and send Authorization headers and handle 401 Unauthorized requests appropriately.

If authentication fails, source origin and the request URL are same origin, Authorization is not in the list of author request headers, request URL's username is the empty string and request URL's password is null, user agents should prompt the end user for their username and password.

Otherwise, if authentication fails, user agents must not prompt the end user for their username and password. [HTTPAUTH]

Unfortunately end users are prompted because of legacy content constraints. However, when possible this behavior is prohibited, as it is bad UI. E.g. that is why the same origin restriction is made above.

If the user agent supports HTTP State Management it should persist, discard and send cookies (as received in the Set-Cookie response header, and sent in the Cookie header) as applicable. [COOKIES]

If the user agent implements a HTTP cache it should respect Cache-Control headers in author request headers (e.g. Cache-Control: no-cache bypasses the cache). It must not send Cache-Control or Pragma request headers automatically unless the end user explicitly requests such behavior (e.g. by reloading the page).

For 304 Not Modified responses that are a result of a user agent generated conditional request the user agent must act as if the server gave a 200 OK response with the appropriate content. The user agent must allow author request headers to override automatic cache validation (e.g. If-None-Match or If-Modified-Since), in which case 304 Not Modified responses must be passed through. [HTTP]

If the user agent implements server-driven content-negotiation it must follow these constraints for the Accept and Accept-Language request headers:

    Both headers must not be modified if they are in author request headers.

    If not in author request headers, Accept-Language with an appropriate value should be appended to it.

    If not in author request headers, Accept with value */* must be appended to it. 

Responses must have the content-encodings automatically decoded. [HTTP]

Besides the author request headers, user agents should not include additional request headers other than those mentioned above or other than those authors are not allowed to set using setRequestHeader(). This ensures that authors have a predictable API.

#### 4.5.7 The abort() method

```javascript
// 取消网络活动
client.abort()
```

The abort() method must run these steps:

    Terminate the request.

    If the state is OPENED with the send() flag set, HEADERS_RECEIVED, or LOADING, run the request error steps for event abort.

    Change the state to UNSENT.

    No readystatechange event is dispatched. 

### 4.6 Response

An XMLHttpRequest has an associated response. Unless stated otherwise it is a network error.

#### 4.6.1 The responseURL attribute

The responseURL attribute must return the empty string if response's url is null and its serialization with the exclude fragment flag set otherwise.
#### 4.6.2 The status attribute

The status attribute must return the response's status.

#### 4.6.3 The statusText attribute

The statusText attribute must return the response's status message.

#### 4.6.4 The getResponseHeader() method

The getResponseHeader(name) method must run these steps:

    If response's header list has multiple headers whose name is name, return their values in list order as a single byte sequence separated from each other by a 0x2C 0x20 byte pair.

    If response's header list has one header whose name is name, return its value.

    Return null. 

The Fetch Standard filters response's header list. [FETCH]

For the following script:

```javascript
var client = new XMLHttpRequest();
client.open("GET", "unicorns-are-teh-awesome.txt", true);
client.send();
client.onreadystatechange = function() {
  if(this.readyState == 2) {
    print(client.getResponseHeader("Content-Type"));
  }
}
```

The print() function will get to process something like:

text/plain; charset=UTF-8

#### 4.6.5 The getAllResponseHeaders() method

The getAllResponseHeaders() method must return response's header list, in list order, as a single byte sequence with each header separated by a 0x0D 0x0A byte pair, and each name and value of a header separated by a 0x3A 0x20 byte pair.

The Fetch Standard filters response's header list. [FETCH]

For the following script:

```ruby
var client = new XMLHttpRequest();
client.open("GET", "narwhals-too.txt", true);
client.send();
client.onreadystatechange = function() {
  if(this.readyState == 2) {
    print(this.getAllResponseHeaders());
  }
}
```

The print() function will get to process something like:

```
Date: Sun, 24 Oct 2004 04:58:38 GMT
Server: Apache/1.3.31 (Unix)
Keep-Alive: timeout=15, max=99
Connection: Keep-Alive
Transfer-Encoding: chunked
Content-Type: text/plain; charset=utf-8
```

#### 4.6.6 Response body

The response MIME type is the MIME type the `Content-Type` header contains excluding any parameters and converted to ASCII lowercase, or null if the response header can not be parsed or was omitted. The override MIME type is initially null and can get a value if overrideMimeType() is invoked. Final MIME type is the override MIME type unless that is null in which case it is the response MIME type.

The response charset is the value of the charset parameter of the `Content-Type` header or null if there was no `charset` parameter or the header could not be parsed or was omitted. The override charset is initially null and can get a value if overrideMimeType() is invoked. Final charset is the override charset unless that is null in which case it is the response charset.

An XMLHttpRequest object has an associated response ArrayBuffer object, response Blob object, response Document object, and a response JSON object. Their shared initial value is null.

An arraybuffer response is the return value of these steps:

    If response ArrayBuffer object is non-null, return it.

    Let bytes be the empty byte sequence, if response's body is null, and response's body otherwise.

    Set response ArrayBuffer object to a new ArrayBuffer object representing bytes and return it. 

A blob response is the return value of these steps:

    If response Blob object is non-null, return it.

    Let type be the empty string, if final MIME type is null, and final MIME type otherwise.

    Let bytes be the empty byte sequence, if response's body is null, and response's body otherwise.

    Set response Blob object to a new Blob object representing bytes with type type and return it. 

A document response is the return value of these steps:

    If response Document object is non-null, return it.

    Let bytes be response's body.

    If bytes is null, return null.

    If final MIME type is not null, text/html, text/xml, application/xml, or does not end in +xml, return null.

    If responseType is the empty string and final MIME type is text/html, return null.

    This is restricted to responseType being "document" in order to prevent breaking legacy content.

    If final MIME type is text/html, run these substeps:

        Let charset be the final charset.

        If charset is null, prescan the first 1024 bytes of bytes and if that does not terminate unsuccessfully then let charset be the return value.

        If charset is null, set charset to utf-8.

        Let document be a document that represents the result parsing bytes following the rules set forth in the HTML Standard for an HTML parser with scripting disabled and a known definite encoding charset. [HTML]

        Flag document as an HTML document. 

    Otherwise, let document be a document that represents the result of running the XML parser with XML scripting support disabled on bytes. If that fails (unsupported character encoding, namespace well-formedness error, etc.), return null. [HTML]

    Resources referenced will not be loaded and no associated XSLT will be applied.

    If charset is null, set charset to utf-8.

    Set document's encoding to charset.

    Set document's content type to final MIME type.

    Set document's URL to response's url.

    Set document's origin to settings object's origin.

    Set response Document object to document and return it. 

A JSON response is the return value of these steps:

    If response JSON object is non-null, return it.

    Let bytes be response's body.

    If bytes is null, return null.

    Let JSON text be the result of running utf-8 decode on byte stream bytes.

    Let JSON object be the result of invoking the initial value of the parse property of the JSON object, with JSON text as its only argument. If that threw an exception, return null. [ECMASCRIPT]

    Set response JSON object to JSON object and return it. 

A text response is the return value of these steps:

    Let bytes be response's body.

    If bytes is null, return the empty string.

    Let charset be the final charset.

    If responseType is the empty string, charset is null, and final MIME type is either null, text/xml, application/xml or ends in +xml, use the rules set forth in the XML specifications to determine the encoding. Let charset be the determined encoding. [XML] [XMLNS]

    This is restricted to responseType being the empty string to keep the non-legacy responseType value "text" simple.

    If charset is null, set charset to utf-8.

    Return the result of running decode on byte stream bytes using fallback encoding charset. 

Authors are strongly encouraged to always encode their resources using utf-8.

### 4.6.7 The overrideMimeType() method

client . overrideMimeType(mime)

    Sets the `Content-Type` header for response to mime.

    Throws an InvalidStateError exception if the state is LOADING or DONE.

    Throws a SyntaxError exception if mime is not a valid MIME type. 

The overrideMimeType(mime) method must run these steps:

    If the state is LOADING or DONE, throw an InvalidStateError exception.

    If parsing mime analogously to the value of the `Content-Type` header fails, throw a SyntaxError exception.

    If mime is successfully parsed, set override MIME type to its MIME type, excluding any parameters, and converted to ASCII lowercase.

    If a `charset` parameter is successfully parsed, set override charset to its value. 

#### 4.6.8 The responseType attribute

client.responseType [ = value ]

    Returns the response type.

    Can be set to change the response type. Values are: the empty string (default), "arraybuffer", "blob", "document", "json", and "text".

    When set: setting to "document" is ignored if the JavaScript global environment is a worker environment

    When set: throws an InvalidStateError exception if the state is LOADING or DONE.

    When set: throws an InvalidAccessError exception if the synchronous flag is set and the JavaScript global environment is a document environment. 

The responseType attribute must return its value. Initially its value must be the empty string.

Setting the responseType attribute must run these steps:

    If the JavaScript global environment is a worker environment and the given value is "document", terminate these steps.

    If the state is LOADING or DONE, throw an InvalidStateError exception.

    If the JavaScript global environment is a document environment and the synchronous flag is set, throw an InvalidAccessError exception.

    Set the responseType attribute's value to the given value. 

#### 4.6.9 The response attribute

client.response

    Returns the response's body. 

The response attribute must return the result of running these steps:

If responseType is the empty string or "text"

        If the state is not LOADING or DONE, return the empty string.

        Return the text response. 

Otherwise

        If the state is not DONE, return null.

        If responseType is "arraybuffer"

            Return the arraybuffer response. 
        If responseType is "blob"

            Return the blob response. 
        If responseType is "document"

            Return the document response. 
        If responseType is "json"

            Return the JSON response. 


#### 4.6.10 The responseText attribute

client . responseText

    Returns the text response.

    Throws an InvalidStateError exception if responseType is not the empty string or "text". 

The responseText attribute must return the result of running these steps:

    If responseType is not the empty string or "text", throw an InvalidStateError exception.

    If the state is not LOADING or DONE, return the empty string.

    Return the text response. 


#### 4.6.11 The responseXML attribute

client . responseXML

    Returns the document response.

    Throws an InvalidStateError exception if responseType is not the empty string or "document". 

The responseXML attribute must return the result of running these steps:

    If responseType is not the empty string or "document", throw an InvalidStateError exception.

    If the state is not DONE, return null.

    Return the document response. 

The responseXML attribute has XML in its name for historical reasons. It also returns HTML resources as documents.

### 4.7 Events summary

注: 本节为非规范

如下的事件由XMLHttpRequest/XMLHttpRequestUpload对象分发并处理:  

Event name        | Interface     |  Dispatched when…
----------------- | ------------- | ---------------------------------------------------------------
readystatechange 	| Event 	      | The readyState attribute changes value, except when it changes to UNSENT.
loadstart 	      | ProgressEvent |	The fetch initiates.
progress 	        | ProgressEvent |	Transmitting data.
abort 	          | ProgressEvent |	When the fetch has been aborted. For instance, by invoking the abort() method.
error 	          | ProgressEvent |	The fetch failed.
load 	            | ProgressEvent |	The fetch succeeded.
timeout           |	ProgressEvent |	The author specified timeout has passed before the fetch completed.
loadend 	        | ProgressEvent |	The fetch completed (success or failure).

## 5 Interface FormData

typedef (File or USVString) FormDataEntryValue;

```javascript
[Constructor(optional HTMLFormElement form),
 Exposed=(Window,Worker)]
interface FormData {
  void append(USVString name, Blob value, optional USVString filename);
  void append(USVString name, USVString value);
  void delete(USVString name);
  FormDataEntryValue? get(USVString name);
  sequence<FormDataEntryValue> getAll(USVString name);
  boolean has(USVString name);
  void set(USVString name, Blob value, optional USVString filename);
  void set(USVString name, USVString value);
  iterable<USVString, FormDataEntryValue>;
};
```

The FormData object represents an ordered list of entries. Each entry consists of a name and a value.

For the purposes of interaction with other algorithms, an entry's type is "string" if value is a string and "file" otherwise. If an entry's type is "file", its filename is the value of entry's value's name attribute.

To create an entry for name, value, and optionally a filename, run these steps:

    Let entry be a new entry.

    Set entry's name to name.

    If value is a Blob object and not a File object, set value to a new File object, representing the same bytes, whose name attribute value is "blob".

    If value is a File object and filename is given, set value to a new File object, representing the same bytes, whose name attribute value is filename.

    Set entry's value to value.

    Return entry. 

The FormData(form) constructor must run these steps:

    Let fd be a new FormData object.

    If form is given, set fd's entries to the result of constructing the form data set for form.

    Return fd. 

The append(name, value, filename) method must run these steps:

    Let entry be the result of create an entry with name, value, and filename if given.

    Append entry to FormData object's list of entries. 

The delete(name) method must remove all entries whose name is name.

The get(name) method must return the value of the first entry whose name is name, and null otherwise.

The getAll(name) method must return the values of all entries whose name is name, in list order, and the empty sequence otherwise.

The set(name, value) method must run these steps:

    Let entry be the result of create an entry with name, value, and filename if given.

    If there are any entries whose name is name, replace the first such entry with entry and remove the others.

    Otherwise, append entry to FormData object's list of entries. 

The has(name) method must return true if there is an entry whose name is name, and false otherwise.

The value pairs to iterate over are the entries with the key being the name and the value the value.

## 6 Interface ProgressEvent

```javascript
[Constructor(DOMString type, optional ProgressEventInit eventInitDict),
 Exposed=(Window,Worker)]
interface ProgressEvent : Event {
  readonly attribute boolean lengthComputable;
  readonly attribute unsigned long long loaded;
  readonly attribute unsigned long long total;
};

dictionary ProgressEventInit : EventInit {
  boolean lengthComputable = false;
  unsigned long long loaded = 0;
  unsigned long long total = 0;
}
```

Events using the ProgressEvent interface indicate some kind of progression.

The lengthComputable, loaded, and total attributes must return the value they were initialized to.

### 6.1 Firing events using the ProgressEvent interface

To fire an progress event named e given transmitted and length, fire an event named e with an event using the ProgressEvent interface that also meets these conditions:

    Set the loaded attribute value to transmitted.

    If length is not 0, set the lengthComputable attribute value to true and the total attribute value to length. 

### 6.2 Suggested names for events using the ProgressEvent interface

This section is non-normative.

The suggested type attribute values for use with events using the ProgressEvent interface are summarized in the table below. Specification editors are free to tune the details to their specific scenarios, though are strongly encouraged to discuss their usage with the WHATWG community to ensure input from people familiar with the subject.

类型属性值  | 描述         |   次数     |  触发时机
----------- | ------------ | ---------- | -----------------------------
loadstart   |	处理开始     | 	Once. 	  | First.
progress 	  | 处理中       | Once or more. | 	After loadstart has been dispatched.
error 	    | 处理失败     | Zero or once (mutually exclusive). |	After the last progress has been dispatched.
abort 	    | 处理终结
timeout 	  | Progression is terminated due to preset time expiring.
load 	      | Progression is successful.
loadend 	  | Progress has stopped. 	Once. 	After one of error, abort, timeout or load has been dispatched.

The error, abort, timeout, and load event types are mutually exclusive.

Throughout the web platform the error, abort, timeout and load event types have their bubbles and cancelable attributes initialized to false, so it is suggested that for consistency all events using the ProgressEvent interface do the same.

### 6.3 Security Considerations

For cross-origin requests some kind of opt-in, e.g. the CORS protocol defined in the Fetch Standard, has to be used before events using the ProgressEvent interface are dispatched as information (e.g. size) would be revealed that cannot be obtained otherwise. [FETCH]

### 6.4 Example

In this example XMLHttpRequest, combined with concepts defined in the sections before, and the HTML progress element are used together to display the process of fetching a resource.

```html
<!DOCTYPE html>
<title>Waiting for Magical Unicorns</title>
<progress id=p></progress>
<script>
  var progressBar = document.getElementById("p"),
      client = new XMLHttpRequest()
  client.open("GET", "magical-unicorns")
  client.onprogress = function(pe) {
    if(pe.lengthComputable) {
      progressBar.max = pe.total
      progressBar.value = pe.loaded
    }
  }
  client.onloadend = function(pe) {
    progressBar.value = pe.loaded
  }
  client.send()
</script>
```

Fully working code would of course be more elaborate and deal with more scenarios, such as network errors or the end user terminating the request.

## 引用

- [COOKIES](http://tools.ietf.org/html/rfc6265) HTTP State Management Mechanism, Adam Barth. IETF. 
- [DOM](http://www.w3.org/TR/dom/) DOM, Anne van Kesteren, Aryeh Gregor and Ms2ger. WHATWG. 
- [DOMPS](http://www.w3.org/TR/DOM-Parsing/) DOM Parsing and Serialization, Travis Leithead. W3C. 
- [ECMASCRIPT](http://www.ecma-international.org/publications/standards/Ecma-262.htm) ECMAScript Language Specification. ECMA. 
- [ENCODING](http://encoding.spec.whatwg.org/) Encoding, Anne van Kesteren. WHATWG. 
- [FETCH]() Fetch, Anne van Kesteren. WHATWG. 
- [FILEAPI](http://www.w3.org/TR/FileAPI/) File API, Arun Ranganathan and Jonas Sicking. W3C. 
- [HTML](http://www.w3.org/TR/html5/) HTML, Ian Hickson. WHATWG. 
- [HTTP](http://tools.ietf.org/html/rfc2616) Hypertext Transfer Protocol -- HTTP/1.1, Roy Fielding, James Gettys, Jeffrey Mogul et al.. IETF. 
- [RFC2119](http://tools.ietf.org/html/rfc2119)  Key words for use in RFCs to Indicate Requirement Levels, Scott Bradner. IETF. 
- [URL](http://url.spec.whatwg.org/) URL, Anne van Kesteren. WHATWG. 
- [WEBIDL](http://dev.w3.org/2006/webapi/WebIDL/) Web IDL, Cameron McCormack. W3C. 
- [XML](http://www.w3.org/TR/xml/) Extensible Markup Language, Tim Bray, Jean Paoli, C. M. Sperberg-McQueen et al.. W3C. 
- [XMLNS](http://www.w3.org/TR/xml-names/)  Namespaces in XML, Tim Bray, Dave Hollander, Andrew Layman et al.. W3C. 
