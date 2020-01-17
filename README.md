# SpringBoot搭建WebSocket服务

# 一、环境准备

+ 开发工具：STS、HBuilderX
+ JDK：1.8
+ SpringBoot版本：2.2.2

# 二、后端搭建WebSocket服务
+ 1.新建SpringBoot项目，并勾选WebSocket服务
+ 2.项目目录结构如下
+ 3.新建类文件WebSocket，文件位置如项目结构所示，代码编写如下
+ 4.新建类文件WebSocketConfig，文件位置如项目结构所示，代码编写如下
## 1.新建SpringBoot项目，并勾选WebSocket服务
<img src="https://img-blog.csdnimg.cn/2020011320424874.png"  width="50%"/>

## 2.项目目录结构如下

<img src="https://img-blog.csdnimg.cn/20200113204536992.png"  width="35%"/>

## 3.新建类文件WebSocket，文件位置如项目结构所示，代码编写如下

```java
package com.websocket.websocket;

import java.io.IOException;
import java.util.concurrent.ConcurrentHashMap;

import javax.websocket.OnClose;
import javax.websocket.OnError;
import javax.websocket.OnMessage;
import javax.websocket.OnOpen;
import javax.websocket.Session;
import javax.websocket.server.PathParam;
import javax.websocket.server.ServerEndpoint;

import org.springframework.web.bind.annotation.RestController;

/**
 * WebSocket
 * 
 * @author ZhongLi
 * @date 2020-01-12
 * @version 1.0.0
 */
@ServerEndpoint("/WebSocket/{id}")
@RestController
public class WebSocket {

	// 存储会话
	private static ConcurrentHashMap<String, WebSocket> webSocket = new ConcurrentHashMap<String, WebSocket>();

	private String id;
	private Session session;

	/**
	 * 接入连接回调
	 * 
	 * @param session 会话对象
	 * @param id      会话ID
	 * @throws Exception 异常
	 */
	@OnOpen
	public void onOpen(Session session, @PathParam("id") String id) throws Exception {
		this.id = id;
		this.session = session;
		webSocket.put(id, this);
		// 检验后端能否正常给前端发送信息
		sendMessageToId(this.id, "前端你好，我是后端，我正在通过WebSocket给你发送消息");
		System.out.println(id + "接入连接");
	}

	/**
	 * 关闭连接回调
	 */
	@OnClose
	public void onClose() {
		webSocket.remove(this.id);
		System.out.println(this.id + "关闭连接");
	}

	/**
	 * 收到客户端发来消息回调
	 * 
	 * @param message
	 * @param session
	 */
	@OnMessage
	public void onMessage(String message, Session session) {
		System.out.println(this.id + "发来消息：" + message);
	}

	/**
	 * 会话出现错误回调
	 * 
	 * @param session 会话对象
	 * @param error   错误信息
	 */
	@OnError
	public void onError(Session session, Throwable error) {

	}

	/**
	 * 发送消息给客户端
	 * 
	 * @param message 消息
	 * @throws IOException 异常
	 */
	public void sendMessage(String message) throws IOException {
		this.session.getBasicRemote().sendText(message);
	}

	/**
	 * 给指定的会话发送消息
	 * 
	 * @param id      会话ID
	 * @param message 消息
	 * @throws IOException 异常
	 */
	public void sendMessageToId(String id, String message) throws IOException {
		webSocket.get(id).sendMessage(message);
	}

	/**
	 * 群发消息
	 * 
	 * @param message 消息
	 * @throws IOException 异常
	 */
	public void sendMessageToAll(String message) throws IOException {
		for (String key : webSocket.keySet()) {
			try {
				webSocket.get(key).sendMessage(message);
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

}

```

## 4.新建类文件WebSocketConfig，文件位置如项目结构所示，代码编写如下

```java
package com.websocket.websocket;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.socket.server.standard.ServerEndpointExporter;

/**
 * WebSocketConfig
 * 
 * @author ZhongLi
 * @date 2020-01-12
 * @version 1.0.0
 */
@Configuration
public class WebSocketConfig {
	@Bean
	public ServerEndpointExporter serverEndpointExporter() {
		return new ServerEndpointExporter();
	}
}

```

# 三、新建前端项目
+ 1.新建index.html文件，代码如下

## 1.新建index.html文件，代码如下


```javascript
<!DOCTYPE html>
<html>
	<head>
		<meta charset="utf-8">
		<title>WebSocket</title>
	</head>
	<body>
	</body>
	<script type="text/javascript">
		var websocket = null;
		//判断当前浏览器是否支持WebSocket
		if ('WebSocket' in window) {
			websocket = new WebSocket("ws://127.0.0.1:8080/WebSocket/zhongli");
		} else {
			alert('当前浏览器 Not support websocket')
		}

		//连接发生错误的回调方法
		websocket.onerror = function() {
			console.log("WebSocket连接发生错误");
		};

		//连接成功建立的回调方法
		websocket.onopen = function() {
			console.log("WebSocket连接成功");
		}

		//接收到消息的回调方法
		websocket.onmessage = function(event) {
			console.log(event.data);
		}

		//连接关闭的回调方法
		websocket.onclose = function() {
			console.log("WebSocket连接关闭");
		}

		//发送消息
		function sendMsg(msg) {
			websocket.send(msg);
		}
	</script>

</html>

```

# 四、项目演示
+ 1.启动SpringBoot项目，项目正常启动
+ 2.运行前端项目并查看控制台，浏览器控制台提示成功
+ 3.检验后端是否正常给前端发送消息，查看浏览器控制台正常收到消息
+ 4.检验前端是否正常给后端发送消息，浏览器控制台运行sendMsg()方法，正常给后端发送消息

## 1.启动SpringBoot项目，项目正常启动
<img src="https://img-blog.csdnimg.cn/20200113210853420.png">

## 2.运行前端项目并查看控制台，浏览器控制台提示成功
<img src="https://img-blog.csdnimg.cn/20200113211042494.png">

## 3.检验后端是否正常给前端发送消息，查看浏览器控制台正常收到消息
<img src="https://img-blog.csdnimg.cn/20200113211150807.png">

## 4.检验前端是否正常给后端发送消息，浏览器控制台运行sendMsg()方法，正常给后端发送消息

<img src="https://img-blog.csdnimg.cn/20200113211633406.png">