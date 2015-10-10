# 高级接口使用——获取用户地理位置

[官方文档-获取用户地理位置](http://mp.weixin.qq.com/wiki/8/1b86529d05db9f960e48c3c7ca5be288.html)

获取用户的地理位置，首先需要再公众号里开通『上报地理位置』接口。这个接口有两个选项，一个是每次进入公众号会话时，都会在进入时上报地理位置，或在进入会话后每5秒上报一次地理位置，公众号可以在公众平台网站中修改以上设置。

开通了上报地理位置接口的公众号，用户在关注后进入公众号会话时，会弹框让用户确认是否允许公众号使用其地理位置。弹框只在关注后出现一次，用户以后可以在公众号详情页面进行操作。

当用户进入公众号或者每5秒后，服务器就会接收到微信服务器发来的XML数据包。XML数据包实例：

```xml
<xml>
<ToUserName><![CDATA[toUser]]></ToUserName>
<FromUserName><![CDATA[fromUser]]></FromUserName>
<CreateTime>123456789</CreateTime>
<MsgType><![CDATA[event]]></MsgType>
<Event><![CDATA[LOCATION]]></Event>
<Latitude>23.137466</Latitude>
<Longitude>113.352425</Longitude>
<Precision>119.385040</Precision>
</xml>
```

参数说明如下：

|参数|描述|
|---|---|
|ToUserName|开发者微信号|
|FromUserName|发送方帐号（一个OpenID）|
|CreateTime	|消息创建时间 （整型）|
|MsgType|消息类型，event|
|Event|事件类型，LOCATION|
|Latitude|地理位置纬度|
|Longitude|地理位置经度|
|Precision|地理位置精度|

我们在MessageTypeDef类中增加对应的常量：

```java
/**
 * 事件类型(Event)：地理位置
 */
public static final String EVENT_TYPE_LOCATION = "LOCATION";
```

修改CoreService中的逻辑，增加对地理位置消息的处理：

```java
// 自定义菜单点击事件
...
//扫描二维码事件
...
//地理位置事件
else if(eventType.equals(MessageTypeDef.EVENT_TYPE_LOCATION)){
    String latitude = requestMap.get("Latitude");
    String longitude = requestMap.get("Longitude");
    String precision = requestMap.get("Precision");
    String locationInfo = "LOCATION INFO:" + latitude + "," + longitude + "," + precision;
    log.info(locationInfo);
    respContent = locationInfo;
}
```

这样当收到用户的地理位置信息时，我们会将获取到的用户地理位置信息返还给用户。





