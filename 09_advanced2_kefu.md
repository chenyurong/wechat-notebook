# 高级接口使用——客服接口

[点击查看官方文档](http://mp.weixin.qq.com/wiki/7/12a5a320ae96fecdf0e15cb06123de9f.html)

当用户主动发消息给公众号的时候（包括发送信息、点击自定义菜单、订阅事件、扫描二维码事件、支付成功事件、用户维权），微信将会把消息数据推送给开发者，开发者在一段时间内（目前修改为48小时）可以调用客服消息接口，通过POST一个JSON数据包来发送消息给普通用户，在48小时内不限制发送次数。此接口主要用于客服等有人工消息处理环节的功能，方便开发者为用户提供更加优质的服务。

向用户发送数据可以分为两步，第一步组装发送的数据，第二步向特定接口发送https请求。

## 组装发送的JSON数据

```java
/**
 * 组装文本客服消息
 * @param openId 消息发送对象
 * @param content 文本消息内容
 * @return
 */
    public static String makeTextCustomMessage(String openId, String content){
        JSONObject textJson = new JSONObject();
        textJson.put("content", content);

        JSONObject jsonObject = new JSONObject();
        jsonObject.put("touser", openId);
        jsonObject.put("msgtype", "text");
        jsonObject.put("text", textJson.toString());

        return jsonObject.toString();
    }

    /**
     * 组装图片客服消息
     * @param openid 消息发送对象
     * @param media_id 媒体ID
     * @return
     */
    public static String makeImageCustomMessage(String openid, String media_id){
        JSONObject imageJson = new JSONObject();
        imageJson.put("media_id", media_id);

        JSONObject json = new JSONObject();
        json.put("touser", openid);
        json.put("msgtype", "image");
        json.put("image", imageJson.toString());

        return json.toString();
    }

    /**
     * 组装语音客服消息
     * @param openid 消息发送对象
     * @param media_id 媒体ID
     * @return
     */
    public static String makeVoiceCustomMessage(String openid, String media_id){
        JSONObject voiceJson = new JSONObject();
        voiceJson.put("media_id", media_id);

        JSONObject json = new JSONObject();
        json.put("touser", openid);
        json.put("msgtype", "voice");
        json.put("voice", voiceJson.toString());

        return json.toString();
    }

    /**
     * 组装视频客服消息
     * @param openid
     * @param media_id
     * @param thumb_media_id
     * @return
     */
    public static String makeVideoCustomMessage(String openid, String media_id, String thumb_media_id, String title, String description){
        JSONObject videoJson = new JSONObject();
        videoJson.put("media_id", media_id);
        videoJson.put("thumb_media_id", thumb_media_id);
        videoJson.put("title", title);
        videoJson.put("description", description);

        JSONObject json = new JSONObject();
        json.put("touser", openid);
        json.put("msgtype", "voice");
        json.put("video", videoJson.toString());

        return json.toString();
    }

    /**
     * 组装音乐客服消息
     * @param openid
     * @param music_title
     * @param musci_description
     * @param music_url
     * @param hq_music_url
     * @param thumb_media_id
     * @return
     */
    public static String makeMusicCustomMessage(String openid, String music_title, String musci_description, String music_url, String hq_music_url, String thumb_media_id){
        JSONObject musicJson = new JSONObject();
        musicJson.put("title", music_title);
        musicJson.put("description", musci_description);
        musicJson.put("musicurl", music_url);
        musicJson.put("hqmusicurl", hq_music_url);
        musicJson.put("thumb_media_id", thumb_media_id);

        JSONObject json = new JSONObject();
        json.put("touser", openid);
        json.put("msgtype", "music");
        json.put("video", musicJson.toString());

        return json.toString();
    }

    /**
     * 创建图文客服消息
     * @param openid
     * @param newsList
     * @return
     */
    public static String makeNewsCustomMessage(String openid, List<News> newsList){
        JSONArray newsJsonArray = new JSONArray();
        for(News news : newsList){
            JSONObject newsJson = new JSONObject();
            newsJson.put("title", news.getTitle());
            newsJson.put("description", news.getDescription());
            newsJson.put("url", news.getUrl());
            newsJson.put("picurl", news.getPicUrl());
            newsJsonArray.add(newsJson);
        }

        JSONObject newsJsonObject = new JSONObject();
        newsJsonObject.put("articles", newsJsonArray);

        JSONObject json = new JSONObject();
        json.put("touser", openid);
        json.put("msgtype", "news");
        json.put("news", newsJsonObject.toString());

        return json.toString();
    }
```

## 向特定接口发送https请求

在WeixinUtil类中加入方法：

```java
/**
 * 发送客服消息
 * @param accessToken 接口访问凭证
 * @param jsonMsg json格式的客服消息（包括touser、msgtype和消息内容）
 * @return true | false
 */
public static boolean sendCustomMessage(String accessToken, String jsonMsg){
    log.info("消息内容：{}" + jsonMsg);
    boolean result = false;
    //拼接地址
    String requestUrl = "https://api.weixin.qq.com/cgi-bin/message/custom/send?access_token=ACCESS_TOKEN";
    requestUrl = requestUrl.replace("ACCESS_TOKEN", accessToken);
    //发送客服消息
    log.info("Final requestUrl is :  >>>>>>>>" + requestUrl + "<<<<<<<<<<<<<<<<<<");
    log.info("Final Message is :  >>>>>>>>" + jsonMsg + "<<<<<<<<<<<<<<<<<<");
    JSONObject jsonObject = httpsRequest(requestUrl, "POST", jsonMsg);
    if(null != jsonObject){
        int errorCode = jsonObject.getInt("errcode");
        String errorMsg = jsonObject.getString("errmsg");
        if(0 == errorCode){
            result = true;
            log.info("客服消息发送成功 errorcode:{} errmsg:{}", errorCode, errorMsg );
        }else{
            log.error("客服消息发送失败 errorcode:{} errmsg:{}", errorCode, errorMsg );
        }
    }
    return result;
}
```

下面我们测试一下，修改CoreService类中的处理逻辑。

当用户发送的文字包含『帅哥』时，除了恢复『你发送的是文本消息』之外，还会再2秒后再在2秒后发送一条消息给用户。

```java
// 文本消息
if (msgType.equals(MessageTypeDef.REQ_MESSAGE_TYPE_TEXT)) {
    respContent = "您发送的是文本消息！";
    if(requestMap.get("Content").contains("帅哥")){
        new Thread(new TestThread(fromUserName)).start();
    }
}
```

其中TestThread类代码如下：

```java
package com.chanshuyi.wechat.thread;

import com.chanshuyi.util.MessageUtil;
import com.chanshuyi.util.WeixinUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * @author chenyr
 * @date 2015.10.09
 */
public class TestThread implements Runnable{

    private Logger log = LoggerFactory.getLogger(TestThread.class);

    private String openId;

    public TestThread(String openId) {
        this.openId = openId;
    }

    @Override
    public void run() {
        try{
            log.info("{} Seconds later to send message.", 2);
            Thread.sleep(2000);
        }catch(Exception e){
            e.printStackTrace();
        }

        String sendMsg = MessageUtil.makeTextCustomMessage(openId, "2秒到了，收到公众号的消息了嘛？");
        log.info("accessToken:[" + TokenThread.accessToken.getToken() + "].   Send the message:" + sendMsg);
        WeixinUtil.sendCustomMessage(TokenThread.accessToken.getToken(), sendMsg);
        log.info("Send message successfully.");
    }
}
```

    




