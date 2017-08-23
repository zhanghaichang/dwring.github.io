---
layout: post
title: 其他
category: java
tags: [java]
---
# HTTP Header 详解
HTTP（HyperTextTransferProtocol）即超文本传输协议，目前网页传输的的通用协议。HTTP协议采用了请求/响应模型，浏览器或其他客户端发出请求，服务器给与响应。就整个网络资源传输而言，包括message-header和message-body两部分。首先传递message-header，即**http** **header**消息 **。**http header 消息通常被分为4个部分：general  header, request header, response header, entity header。但是这种分法就理解而言，感觉界限不太明确。根据维基百科对http header内容的组织形式，大体分为Request和Response两部分。
## Requests部分 
<table border="1" cellpadding="0" width="100%">
<tbody>
<tr>
<th>Header</th><th width="35%">解释</th><th width="40%">示例</th>
</tr>
<tr>
<td>Accept</td>
<td width="35%">指定客户端能够接收的内容类型</td>
<td width="40%">Accept: text/plain, text/html</td>
</tr>
<tr>
<td>Accept-Charset</td>
<td width="35%">浏览器可以接受的字符编码集。</td>
<td width="40%">Accept-Charset: iso-8859-5</td>
</tr>
<tr>
<td>Accept-Encoding</td>
<td width="35%">指定浏览器可以支持的web服务器返回内容压缩编码类型。</td>
<td width="40%">Accept-Encoding: compress, gzip</td>
</tr>
<tr>
<td>Accept-Language</td>
<td width="35%">浏览器可接受的语言</td>
<td width="40%">Accept-Language: en,zh</td>
</tr>
<tr>
<td>Accept-Ranges</td>
<td width="35%">可以请求网页实体的一个或者多个子范围字段</td>
<td width="40%">Accept-Ranges: bytes</td>
</tr>
<tr>
<td>Authorization</td>
<td width="35%">HTTP授权的授权证书</td>
<td width="40%">Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==</td>
</tr>
<tr>
<td>Cache-Control</td>
<td width="35%">指定请求和响应遵循的缓存机制</td>
<td width="40%">Cache-Control: no-cache</td>
</tr>
<tr>
<td>Connection</td>
<td width="35%">表示是否需要持久连接。（HTTP 1.1默认进行持久连接）</td>
<td width="40%">Connection: close</td>
</tr>
<tr>
<td>Cookie</td>
<td width="35%">HTTP请求发送时，会把保存在该请求域名下的所有cookie值一起发送给web服务器。</td>
<td width="40%">Cookie: $Version=1; Skin=new;</td>
</tr>
<tr>
<td>Content-Length</td>
<td width="35%">请求的内容长度</td>
<td width="40%">Content-Length: 348</td>
</tr>
<tr>
<td>Content-Type</td>
<td width="35%">请求的与实体对应的MIME信息</td>
<td width="40%">Content-Type: application/x-www-form-urlencoded</td>
</tr>
<tr>
<td>Date</td>
<td width="35%">请求发送的日期和时间</td>
<td width="40%">Date: Tue, 15 Nov&nbsp;2010 08:12:31 GMT</td>
</tr>
<tr>
<td>Expect</td>
<td width="35%">请求的特定的服务器行为</td>
<td width="40%">Expect: 100-continue</td>
</tr>
<tr>
<td>From</td>
<td width="35%">发出请求的用户的Email</td>
<td width="40%">From: user@email.com</td>
</tr>
<tr>
<td>Host</td>
<td width="35%">指定请求的服务器的域名和端口号</td>
<td width="40%">Host: www.zcmhi.com</td>
</tr>
<tr>
<td>If-Match</td>
<td width="35%">只有请求内容与实体相匹配才有效</td>
<td width="40%">If-Match: &ldquo;737060cd8c284d8af7ad3082f209582d&rdquo;</td>
</tr>
<tr>
<td>If-Modified-Since</td>
<td width="35%">如果请求的部分在指定时间之后被修改则请求成功，未被修改则返回304代码</td>
<td width="40%">If-Modified-Since: Sat, 29 Oct 2010 19:43:31 GMT</td>
</tr>
<tr>
<td>If-None-Match</td>
<td width="35%">如果内容未改变返回304代码，参数为服务器先前发送的Etag，与服务器回应的Etag比较判断是否改变</td>
<td width="40%">If-None-Match: &ldquo;737060cd8c284d8af7ad3082f209582d&rdquo;</td>
</tr>
<tr>
<td>If-Range</td>
<td width="35%">如果实体未改变，服务器发送客户端丢失的部分，否则发送整个实体。参数也为Etag</td>
<td width="40%">If-Range: &ldquo;737060cd8c284d8af7ad3082f209582d&rdquo;</td>
</tr>
<tr>
<td>If-Unmodified-Since</td>
<td width="35%">只在实体在指定时间之后未被修改才请求成功</td>
<td width="40%">If-Unmodified-Since: Sat, 29 Oct 2010 19:43:31 GMT</td>
</tr>
<tr>
<td>Max-Forwards</td>
<td width="35%">限制信息通过代理和网关传送的时间</td>
<td width="40%">Max-Forwards: 10</td>
</tr>
<tr>
<td>Pragma</td>
<td width="35%">用来包含实现特定的指令</td>
<td width="40%">Pragma: no-cache</td>
</tr>
<tr>
<td>Proxy-Authorization</td>
<td width="35%">连接到代理的授权证书</td>
<td width="40%">Proxy-Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==</td>
</tr>
<tr>
<td>Range</td>
<td width="35%">只请求实体的一部分，指定范围</td>
<td width="40%">Range: bytes=500-999</td>
</tr>
<tr>
<td>Referer</td>
<td width="35%">先前网页的地址，当前请求网页紧随其后,即来路</td>
<td width="40%">Referer: http://www.zcmhi.com/archives/71.html</td>
</tr>
<tr>
<td>TE</td>
<td width="35%">客户端愿意接受的传输编码，并通知服务器接受接受尾加头信息</td>
<td width="40%">TE: trailers,deflate;q=0.5</td>
</tr>
<tr>
<td>Upgrade</td>
<td width="35%">向服务器指定某种传输协议以便服务器进行转换（如果支持）</td>
<td width="40%">Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11</td>
</tr>
<tr>
<td>User-Agent</td>
<td width="35%">User-Agent的内容包含发出请求的用户信息</td>
<td width="40%">User-Agent: Mozilla/5.0 (Linux; X11)</td>
</tr>
<tr>
<td>Via</td>
<td width="35%">通知中间网关或代理服务器地址，通信协议</td>
<td width="40%">Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)</td>
</tr>
<tr>
<td>Warning</td>
<td width="35%">关于消息实体的警告信息</td>
<td width="40%">Warn: 199 Miscellaneous warning</td>
</tr>
</tbody>
</table>
## Responses部分 
<table border="1" cellpadding="0" width="100%">
<tbody>
<tr>
<th>Header</th><th width="35%">解释</th><th width="40%">示例</th>
</tr>
<tr>
<td>Accept-Ranges</td>
<td width="35%">表明服务器是否支持指定范围请求及哪种类型的分段请求</td>
<td width="40%">Accept-Ranges: bytes</td>
</tr>
<tr>
<td>Age</td>
<td width="35%">从原始服务器到代理缓存形成的估算时间（以秒计，非负）</td>
<td width="40%">Age: 12</td>
</tr>
<tr>
<td>Allow</td>
<td width="35%">对某网络资源的有效的请求行为，不允许则返回405</td>
<td width="40%">Allow: GET, HEAD</td>
</tr>
<tr>
<td>Cache-Control</td>
<td width="35%">告诉所有的缓存机制是否可以缓存及哪种类型</td>
<td width="40%">Cache-Control: no-cache</td>
</tr>
<tr>
<td>Content-Encoding</td>
<td width="35%">web服务器支持的返回内容压缩编码类型。</td>
<td width="40%">Content-Encoding: gzip</td>
</tr>
<tr>
<td>Content-Language</td>
<td width="35%">响应体的语言</td>
<td width="40%">Content-Language: en,zh</td>
</tr>
<tr>
<td>Content-Length</td>
<td width="35%">响应体的长度</td>
<td width="40%">Content-Length: 348</td>
</tr>
<tr>
<td>Content-Location</td>
<td width="35%">请求资源可替代的备用的另一地址</td>
<td width="40%">Content-Location: /index.htm</td>
</tr>
<tr>
<td>Content-MD5</td>
<td width="35%">返回资源的MD5校验值</td>
<td width="40%">Content-MD5: Q2hlY2sgSW50ZWdyaXR5IQ==</td>
</tr>
<tr>
<td>Content-Range</td>
<td width="35%">在整个返回体中本部分的字节位置</td>
<td width="40%">Content-Range: bytes 21010-47021/47022</td>
</tr>
<tr>
<td>Content-Type</td>
<td width="35%">返回内容的MIME类型</td>
<td width="40%">Content-Type: text/html; charset=utf-8</td>
</tr>
<tr>
<td>Date</td>
<td width="35%">原始服务器消息发出的时间</td>
<td width="40%">Date: Tue, 15 Nov 2010 08:12:31 GMT</td>
</tr>
<tr>
<td>ETag</td>
<td width="35%">请求变量的实体标签的当前值</td>
<td width="40%">ETag: &ldquo;737060cd8c284d8af7ad3082f209582d&rdquo;</td>
</tr>
<tr>
<td>Expires</td>
<td width="35%">响应过期的日期和时间</td>
<td width="40%">Expires: Thu, 01 Dec 2010 16:00:00 GMT</td>
</tr>
<tr>
<td>Last-Modified</td>
<td width="35%">请求资源的最后修改时间</td>
<td width="40%">Last-Modified: Tue, 15 Nov 2010 12:45:26 GMT</td>
</tr>
<tr>
<td>Location</td>
<td width="35%">用来重定向接收方到非请求URL的位置来完成请求或标识新的资源</td>
<td width="40%">Location: http://www.zcmhi.com/archives/94.html</td>
</tr>
<tr>
<td>Pragma</td>
<td width="35%">包括实现特定的指令，它可应用到响应链上的任何接收方</td>
<td width="40%">Pragma: no-cache</td>
</tr>
<tr>
<td>Proxy-Authenticate</td>
<td width="35%">它指出认证方案和可应用到代理的该URL上的参数</td>
<td width="40%">Proxy-Authenticate: Basic</td>
</tr>
<tr>
<td>refresh</td>
<td width="35%">应用于重定向或一个新的资源被创造，在5秒之后重定向（由网景提出，被大部分浏览器支持）</td>
<td width="40%">
<div><span style="font-family: monospace;">&nbsp;</span></div>
<p><span style="font-family: monospace;">&nbsp;</span></p>
<div id="_mcePaste"><span style="font-family: Georgia,'Times New Roman','Bitstream Charter',Times,serif;">Refresh: 5; url=</span></div>
<div><span style="font-family: Georgia,'Times New Roman','Bitstream Charter',Times,serif;">http://www.zcmhi.com/archives/94.html</span></div>
</td>
</tr>
<tr>
<td>Retry-After</td>
<td width="35%">如果实体暂时不可取，通知客户端在指定时间之后再次尝试</td>
<td width="40%">Retry-After: 120</td>
</tr>
<tr>
<td>Server</td>
<td width="35%">web服务器软件名称</td>
<td width="40%">Server: Apache/1.3.27 (Unix) (Red-Hat/Linux)</td>
</tr>
<tr>
<td>Set-Cookie</td>
<td width="35%">设置Http Cookie</td>
<td width="40%">Set-Cookie: UserID=JohnDoe; Max-Age=3600; Version=1</td>
</tr>
<tr>
<td>Trailer</td>
<td width="35%">指出头域在分块传输编码的尾部存在</td>
<td width="40%">Trailer: Max-Forwards</td>
</tr>
<tr>
<td>Transfer-Encoding</td>
<td width="35%">文件传输编码</td>
<td width="40%"><span style="font-family: monospace;"><span style="font-family: Georgia,'Times New Roman','Bitstream Charter',Times,serif;">Transfer-Encoding:chunked</span></span></td>
</tr>
<tr>
<td>Vary</td>
<td width="35%">告诉下游代理是使用缓存响应还是从原始服务器请求</td>
<td width="40%">Vary: *</td>
</tr>
<tr>
<td>Via</td>
<td width="35%">告知代理客户端响应是通过哪里发送的</td>
<td width="40%">Via: 1.0 fred, 1.1 nowhere.com (Apache/1.1)</td>
</tr>
<tr>
<td>Warning</td>
<td width="35%">警告实体可能存在的问题</td>
<td width="40%">Warning: 199 Miscellaneous warning</td>
</tr>
<tr>
<td>WWW-Authenticate</td>
<td width="35%">表明客户端请求实体应该使用的授权方案</td>
<td width="40%">WWW-Authenticate: Basic</td>
</tr>
</tbody>
</table>

