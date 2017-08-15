---
yout: post
title: httpclient详解
category: java
tags: [java]
---
#### httpclient详解
#### 一、简介
* HTTP 协议可能是现在 Internet 上使用得最多、最重要的协议了，越来越多的 Java 应用程序需要直接通过 HTTP 协议来访问网络资源。虽然在 JDK 的 java.net 包中已经提供了访问 HTTP 协议的基本功能，但是对于大部分应用程序来说，JDK 库本身提供的功能还不够丰富和灵活。HttpClient 是 Apache Jakarta Common 下的子项目，用来提供高效的、最新的、功能丰富的支持 HTTP 协议的客户端编程工具包，并且它支持 HTTP 协议最新的版本和建议。HttpClient 已经应用在很多的项目中，比如 Apache Jakarta 上很著名的另外两个开源项目 Cactus 和 HTMLUnit 都使用了 HttpClient。更多信息请关注[httpclient](http://hc.apache.org/)

#### 二、特性
* 1. 基于标准、纯净的Java语言。实现了Http1.0和Http1.1
* 2. 以可扩展的面向对象的结构实现了Http全部的方法（GET, POST, PUT, DELETE, HEAD, OPTIONS, and TRACE）。
* 3. 支持HTTPS协议。
* 4. 通过Http代理建立透明的连接。
* 5. 利用CONNECT方法通过Http代理建立隧道的https连接。
* 6. Basic, Digest, NTLMv1, NTLMv2, NTLM2 Session, SNPNEGO/Kerberos认证方案。
* 7. 插件式的自定义认证方案。
* 8. 便携可靠的套接字工厂使它更容易的使用第三方解决方案。
* 9. 连接管理器支持多线程应用。支持设置最大连接数，同时支持设置每个主机的最大连接数，发现并关闭过期的连接。
* 10. 自动处理Set-Cookie中的Cookie。
* 11. 插件式的自定义Cookie策略。
* 12. Request的输出流可以避免流中内容直接缓冲到socket服务器。
* 13. Response的输入流可以有效的从socket服务器直接读取相应内容。
* 14. 在http1.0和http1.1中利用KeepAlive保持持久连接。
* 15. 直接获取服务器发送的response code和 headers。
* 16. 设置连接超时的能力。
* 17. 实验性的支持http1.1 response caching。
* 18. 源代码基于Apache License 可免费获取。
#### 三、使用方法
使用HttpClient发送请求、接收响应很简单，一般需要如下几步即可。
* 1. 创建HttpClient对象。
* 2. 创建请求方法的实例，并指定请求URL。如果需要发送GET请求，创建HttpGet对象；如果需要发送POST请求，创建HttpPost对象。
* 3. 如果需要发送请求参数，可调用HttpGet、HttpPost共同的setParams(HetpParams params)方法来添加请求参数；对于HttpPost对象而言，也可调用setEntity(HttpEntity entity)方法来设置请求参数。
* 4. 调用HttpClient对象的execute(HttpUriRequest request)发送请求，该方法返回一个HttpResponse。
* 5. 调用HttpResponse的getAllHeaders()、getHeaders(String name)等方法可获取服务器的响应头；调用HttpResponse的getEntity()方法可获取HttpEntity对象，该对象包装了服务器的响应内容。程序可通过该对象获取服务器的响应内容。
* 6. 释放连接。无论执行方法是否成功，都必须释放连接

```maven
<dependency> 
         <groupId>org.apache.httpcomponents</groupId> 
          <artifactId>httpclient</artifactId> 
         <version>4.1.2</version>         
        </dependency> 
        <dependency> 
         <groupId>org.apache.httpcomponents</groupId> 
          <artifactId>httpclient-cache</artifactId> 
         <version>4.1.2</version>         
        </dependency> 
        <dependency> 
         <groupId>org.apache.httpcomponents</groupId> 
          <artifactId>httpmime</artifactId> 
         <version>4.1.2</version>         
  </dependency>
```

#### 实例
```java
package cn.tzz.apache.httpclient;


import java.io.File;
import java.io.IOException;
import java.net.URL;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;

import org.apache.http.HttpEntity;
import org.apache.http.NameValuePair;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.conn.ssl.DefaultHostnameVerifier;
import org.apache.http.conn.util.PublicSuffixMatcher;
import org.apache.http.conn.util.PublicSuffixMatcherLoader;
import org.apache.http.entity.ContentType;
import org.apache.http.entity.StringEntity;
import org.apache.http.entity.mime.MultipartEntityBuilder;
import org.apache.http.entity.mime.content.FileBody;
import org.apache.http.entity.mime.content.StringBody;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;


public class HttpClientUtil {
	private RequestConfig requestConfig = RequestConfig.custom()
            .setSocketTimeout(15000)
            .setConnectTimeout(15000)
            .setConnectionRequestTimeout(15000)
            .build();
	
	private static HttpClientUtil instance = null;
	private HttpClientUtil(){}
	public static HttpClientUtil getInstance(){
		if (instance == null) {
			instance = new HttpClientUtil();
		}
		return instance;
	}
	
	/**
	 * 发送 post请求
	 * @param httpUrl 地址
	 */
	public String sendHttpPost(String httpUrl) {
		HttpPost httpPost = new HttpPost(httpUrl);// 创建httpPost  
		return sendHttpPost(httpPost);
	}
	
	/**
	 * 发送 post请求
	 * @param httpUrl 地址
	 * @param params 参数(格式:key1=value1&key2=value2)
	 */
	public String sendHttpPost(String httpUrl, String params) {
		HttpPost httpPost = new HttpPost(httpUrl);// 创建httpPost  
		try {
			//设置参数
			StringEntity stringEntity = new StringEntity(params, "UTF-8");
			stringEntity.setContentType("application/x-www-form-urlencoded");
			httpPost.setEntity(stringEntity);
		} catch (Exception e) {
			e.printStackTrace();
		}
		return sendHttpPost(httpPost);
	}
	
	/**
	 * 发送 post请求
	 * @param httpUrl 地址
	 * @param maps 参数
	 */
	public String sendHttpPost(String httpUrl, Map<String, String> maps) {
		HttpPost httpPost = new HttpPost(httpUrl);// 创建httpPost  
		// 创建参数队列  
		List<NameValuePair> nameValuePairs = new ArrayList<NameValuePair>();
		for (String key : maps.keySet()) {
			nameValuePairs.add(new BasicNameValuePair(key, maps.get(key)));
		}
		try {
			httpPost.setEntity(new UrlEncodedFormEntity(nameValuePairs, "UTF-8"));
		} catch (Exception e) {
			e.printStackTrace();
		}
		return sendHttpPost(httpPost);
	}
	
	
	/**
	 * 发送 post请求（带文件）
	 * @param httpUrl 地址
	 * @param maps 参数
	 * @param fileLists 附件
	 */
	public String sendHttpPost(String httpUrl, Map<String, String> maps, List<File> fileLists) {
		HttpPost httpPost = new HttpPost(httpUrl);// 创建httpPost  
		MultipartEntityBuilder meBuilder = MultipartEntityBuilder.create();
		for (String key : maps.keySet()) {
			meBuilder.addPart(key, new StringBody(maps.get(key), ContentType.TEXT_PLAIN));
		}
		for(File file : fileLists) {
			FileBody fileBody = new FileBody(file);
			meBuilder.addPart("files", fileBody);
		}
		HttpEntity reqEntity = meBuilder.build();
		httpPost.setEntity(reqEntity);
		return sendHttpPost(httpPost);
	}
	
	/**
	 * 发送Post请求
	 * @param httpPost
	 * @return
	 */
	private String sendHttpPost(HttpPost httpPost) {
		CloseableHttpClient httpClient = null;
		CloseableHttpResponse response = null;
		HttpEntity entity = null;
		String responseContent = null;
		try {
			// 创建默认的httpClient实例.
			httpClient = HttpClients.createDefault();
			httpPost.setConfig(requestConfig);
			// 执行请求
			response = httpClient.execute(httpPost);
			entity = response.getEntity();
			responseContent = EntityUtils.toString(entity, "UTF-8");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				// 关闭连接,释放资源
				if (response != null) {
					response.close();
				}
				if (httpClient != null) {
					httpClient.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return responseContent;
	}

	/**
	 * 发送 get请求
	 * @param httpUrl
	 */
	public String sendHttpGet(String httpUrl) {
		HttpGet httpGet = new HttpGet(httpUrl);// 创建get请求
		return sendHttpGet(httpGet);
	}
	
	/**
	 * 发送 get请求Https
	 * @param httpUrl
	 */
	public String sendHttpsGet(String httpUrl) {
		HttpGet httpGet = new HttpGet(httpUrl);// 创建get请求
		return sendHttpsGet(httpGet);
	}
	
	/**
	 * 发送Get请求
	 * @param httpPost
	 * @return
	 */
	private String sendHttpGet(HttpGet httpGet) {
		CloseableHttpClient httpClient = null;
		CloseableHttpResponse response = null;
		HttpEntity entity = null;
		String responseContent = null;
		try {
			// 创建默认的httpClient实例.
			httpClient = HttpClients.createDefault();
			httpGet.setConfig(requestConfig);
			// 执行请求
			response = httpClient.execute(httpGet);
			entity = response.getEntity();
			responseContent = EntityUtils.toString(entity, "UTF-8");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				// 关闭连接,释放资源
				if (response != null) {
					response.close();
				}
				if (httpClient != null) {
					httpClient.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return responseContent;
	}
	
	/**
	 * 发送Get请求Https
	 * @param httpPost
	 * @return
	 */
	private String sendHttpsGet(HttpGet httpGet) {
		CloseableHttpClient httpClient = null;
		CloseableHttpResponse response = null;
		HttpEntity entity = null;
		String responseContent = null;
		try {
			// 创建默认的httpClient实例.
			PublicSuffixMatcher publicSuffixMatcher = PublicSuffixMatcherLoader.load(new URL(httpGet.getURI().toString()));
			DefaultHostnameVerifier hostnameVerifier = new DefaultHostnameVerifier(publicSuffixMatcher);
			httpClient = HttpClients.custom().setSSLHostnameVerifier(hostnameVerifier).build();
			httpGet.setConfig(requestConfig);
			// 执行请求
			response = httpClient.execute(httpGet);
			entity = response.getEntity();
			responseContent = EntityUtils.toString(entity, "UTF-8");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				// 关闭连接,释放资源
				if (response != null) {
					response.close();
				}
				if (httpClient != null) {
					httpClient.close();
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
		return responseContent;
	}
}

```
#### 测试代码
```java
package cn.tzz.apache.httpclient;

import java.io.File;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.junit.Test;

public class HttpClientUtilTest {
	
	@Test
	public void testSendHttpPost1() {
		String responseContent = HttpClientUtil.getInstance()
				.sendHttpPost("http://localhost:8089/test/send?username=test01&password=123456");
		System.out.println("reponse content:" + responseContent);
	}
	
	@Test
	public void testSendHttpPost2() {
		String responseContent = HttpClientUtil.getInstance()
				.sendHttpPost("http://localhost:8089/test/send", "username=test01&password=123456");
		System.out.println("reponse content:" + responseContent);
	}
	
	@Test
	public void testSendHttpPost3() {
		Map<String, String> maps = new HashMap<String, String>();
		maps.put("username", "test01");
		maps.put("password", "123456");
		String responseContent = HttpClientUtil.getInstance()
				.sendHttpPost("http://localhost:8089/test/send", maps);
		System.out.println("reponse content:" + responseContent);
	}
	@Test
	public void testSendHttpPost4() {
		Map<String, String> maps = new HashMap<String, String>();
		maps.put("username", "test01");
		maps.put("password", "123456");
		List<File> fileLists = new ArrayList<File>();
		fileLists.add(new File("D://test//httpclient//1.png"));
		fileLists.add(new File("D://test//httpclient//1.txt"));
		String responseContent = HttpClientUtil.getInstance()
				.sendHttpPost("http://localhost:8089/test/sendpost/file", maps, fileLists);
		System.out.println("reponse content:" + responseContent);
	}

	@Test
	public void testSendHttpGet() {
		String responseContent = HttpClientUtil.getInstance()
				.sendHttpGet("http://localhost:8089/test/send?username=test01&password=123456");
		System.out.println("reponse content:" + responseContent);
	}
	
	@Test
	public void testSendHttpsGet() {
		String responseContent = HttpClientUtil.getInstance()
				.sendHttpsGet("https://www.baidu.com");
		System.out.println("reponse content:" + responseContent);
	}

}

```

```java
@RequestMapping(value = "/test/send")
	@ResponseBody
	public Map<String, String> sendPost(HttpServletRequest request) {
		Map<String, String> maps = new HashMap<String, String>();
		String username = request.getParameter("username");
		String password = request.getParameter("password");
		maps.put("username", username);
		maps.put("password", password);
		return maps;
	}
	
	@RequestMapping(value = "/test/sendpost/file",method=RequestMethod.POST)
	@ResponseBody
	public Map<String, String> sendPostFile(@RequestParam("files") MultipartFile [] files,HttpServletRequest request) {
		Map<String, String> maps = new HashMap<String, String>();
		String username = request.getParameter("username");
		String password = request.getParameter("password");
		maps.put("username", username);
		maps.put("password", password);
		
		try {
			for(MultipartFile file : files){
				String fileName = file.getOriginalFilename();
				fileName = new String(fileName.getBytes(),"UTF-8");
				InputStream is = file.getInputStream();
				if (fileName != null && !("".equals(fileName))) {
					File directory = new File("D://test//httpclient//file");
					if (!directory.exists()) {
						directory.mkdirs();
					}
					String filePath = ("D://test//httpclient//file") + File.separator + fileName;
					FileOutputStream fos = new FileOutputStream(filePath);
					byte[] buffer = new byte[1024];
					while (is.read(buffer) > 0) {
						fos.write(buffer, 0, buffer.length);
					}
					fos.flush();
					fos.close();
					maps.put("file--"+fileName, "uploadSuccess");
				}
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		return maps;
	}
```
