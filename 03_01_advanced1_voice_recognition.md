# 高级接口使用——语音识别接口

上面的章节中我们介绍了自定义菜单的开发，下面的章节中，我们将介绍微信的九大高级接口。

这一节，我们介绍微信的语音识别接口。

开通语音识别功能，用户每次发送语音给公众号时，微信会在推送的语音消息XML数据包中，增加一个Recongnition字段。因此我们只需要在之前的语音消息类VoiceMessage里增加一个Recognition字段就可以了，修改后的VoiceMessage类代码如下：

```java
package com.chanshuyi.pojo.message.req;

/**
 * 请求类，语音消息
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class VoiceMessage extends BaseMessage {
    // 媒体ID
    private String MediaId;
    // 语音格式
    private String Format;
    // 语音识别结果
    private String Recognition;

    public String getMediaId() {
        return MediaId;
    }

    public void setMediaId(String mediaId) {
        MediaId = mediaId;
    }

    public String getFormat() {
        return Format;
    }

    public void setFormat(String format) {
        Format = format;
    }


    public String getRecognition() {
        return Recognition;
    }

    public void setRecognition(String recognition) {
        Recognition = recognition;
    }
}
```

修改一下之前的CoreService.processRequest()方法，实现当用户发送语音消息时，我们将语音识别的结果回复给用户。

```java
// 音频消息
else if (msgType.equals(MessageTypeDef.REQ_MESSAGE_TYPE_VOICE)) {
    respContent = "您发送的是音频消息！你说的是：" + requestMap.get("Recognition");
}	
```

语音识别的高级接口就是这么简单。

下一节我们将介绍客服接口。