# 处理微信服务器消息 - 简单的例子

本节将用一个简单的例子介绍如何接收并响应微信服务器消息，实现的效果为：用户发送某种类型的消息（文字或语音或图片等）时，将用户发送的消息类型返回给用户。

下面看下代码。

修改后的 `CoreServlet.doPost()` 方法如下：

```java
/**
 * 处理微信服务器发来的消息
 */
public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    // 将请求、响应的编码均设置为UTF-8（防止中文乱码）
    request.setCharacterEncoding("UTF-8");
    response.setCharacterEncoding("UTF-8");

    // 调用核心业务类接收消息、处理消息
    String respMessage = CoreService.processRequest(request);

    // 响应消息
    PrintWriter out = response.getWriter();
    out.print(respMessage);
    out.close();
}
```

这里我们通过调用CoreService类的processRequest方法接收、处理消息的，这样做的目的是为了解耦，即业务相关的操作都不在Servlet里处理，而是完全交由业务核心类CoreService去做。

下面创建CoreService类来处理请求，代码如下：

```java
package com.chanshuyi.wechat.service;

import com.chanshuyi.common.MessageTypeDef;
import com.chanshuyi.pojo.message.resp.TextMessage;
import com.chanshuyi.util.MessageUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.servlet.http.HttpServletRequest;
import java.util.Date;
import java.util.Map;

/**
 * @author chenyr
 * @date 2015.10.04
 */
public class CoreService {

    /**************************************************************
     ***** 您可以通过切换processRequest方法的注释来观看不同的Demo效果****
     **************************************************************/

    private static Logger log = LoggerFactory.getLogger(CoreService.class);

    /**
     * 1.处理微信发来的请求
     * 演示效果：返回当前用户所发送消息的消息类型。
     * 比如：当用户发送文字消息时，公众号回复『您发送的是文本消息』；
     * 当用户发送的是图片消息时，公众号回复『您发送的是图片消息』……
     * @param request
     * @return
     */
    public static String processRequest(HttpServletRequest request) {
        log.info("Start to deal the POST REQUEST.");
        String respMessage = null;
        try {
            // 默认返回的文本消息内容
            String respContent = "请求处理异常，请稍候尝试！";

            // xml请求解析
            Map<String, String> requestMap = MessageUtil.parseXml(request);

            // 发送方帐号（open_id）
            String fromUserName = requestMap.get("FromUserName");
            // 公众帐号
            String toUserName = requestMap.get("ToUserName");
            // 消息类型
            String msgType = requestMap.get("MsgType");

            // 回复文本消息
            TextMessage textMessage = new TextMessage();
            textMessage.setToUserName(fromUserName);
            textMessage.setFromUserName(toUserName);
            textMessage.setCreateTime(new Date().getTime());
            textMessage.setMsgType(MessageTypeDef.RESP_MESSAGE_TYPE_TEXT);

            // 文本消息
            if (msgType.equals(MessageTypeDef.REQ_MESSAGE_TYPE_TEXT)) {
                respContent = "您发送的是文本消息！";
            }
            // 图片消息
            else if (msgType.equals(MessageTypeDef.REQ_MESSAGE_TYPE_IMAGE)) {
                respContent = "您发送的是图片消息！";
            }
            // 地理位置消息
            else if (msgType.equals(MessageTypeDef.REQ_MESSAGE_TYPE_LOCATION)) {
                respContent = "您发送的是地理位置消息！";
            }
            // 链接消息
            else if (msgType.equals(MessageTypeDef.REQ_MESSAGE_TYPE_LINK)) {
                respContent = "您发送的是链接消息！";
            }
            // 音频消息
            else if (msgType.equals(MessageTypeDef.REQ_MESSAGE_TYPE_VOICE)) {
                respContent = "您发送的是音频消息！";
            }
            // 事件推送
            else if (msgType.equals(MessageTypeDef.REQ_MESSAGE_TYPE_EVENT)) {
                // 事件类型
                String eventType = requestMap.get("Event");
                // 订阅
                if (eventType.equals(MessageTypeDef.EVENT_TYPE_SUBSCRIBE)) {
                    respContent = "谢谢您的关注！";
                }
                // 取消订阅
                else if (eventType.equals(MessageTypeDef.EVENT_TYPE_UNSUBSCRIBE)) {
                    // TODO 取消订阅后用户再收不到公众号发送的消息，因此不需要回复消息
                }
                // 自定义菜单点击事件
                else if (eventType.equals(MessageTypeDef.EVENT_TYPE_CLICK)) {
					//TODO
                }
            }

            textMessage.setContent(respContent);
            respMessage = MessageUtil.textMessageToXml(textMessage);
        } catch (Exception e) {
            e.printStackTrace();
        }

        log.info("/END to deal the POST REQUEST.");
        return respMessage;
    }
}
```

上面的例子会对用户发送的消息类型做出判断，并将用户发送的消息类型以文字的形式回复给用户。其中：

- `TextMessage`是封装的一个响应文本类对象   

- `MessageTypeDef`是封装的一个消息类型静态类

- `MessageUtil.textMessageToXml(textMessage)` 是封装的响应微信服务器消息的方法

从上面这个例子我们可以看到我们对服务器的处理主要分为几个部分：

1. 接收微信服务器转发的消息
2. 判断消息类型并进行逻辑处理
3. 回复消息（文本、图片等）给用户

其中第1点需要将XML消息转化为我们可操作的对象、第3点需要将我们回复给用户的POJO对象转化成XML文档。即我们需要处理以下几点：

1. 接收消息和响应消息的POJO对象
2. XML文档操作类（将XML文档转为POJO对象，将POJO对象转为XML文档）

在接下来几篇文章中，我们将分别讲述如何定义POJO对象以及XML文档操作类。



