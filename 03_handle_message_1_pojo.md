# 处理微信服务器的消息 - 定义消息类对象

通过阅读微信公众号开发文档可以知道，我们与微信服务器进行数据接收和发送主要是通过XML文档的方式进行的。因此，我们需要将各类请求、响应消息封装成POJO对象，以方便我们的开发。

## 请求类消息对象

我们接收的消息以供有7种类型，分别是：

- 文本消息
- 图片消息
- 语音消息
- 视频消息
- 小视频消息
- 地理位置消息
- 链接消息

上面7种消息都由一些共同的字段，我们把这些字段提取出来封装成一个基类。

```java
package com.chanshuyi.pojo.message.req;

/**
 * 请求类，消息基类
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class BaseMessage {
    // 开发者微信号
    private String toUserName;
    // 发送方帐号（一个OpenID）
    private String fromUserName;
    // 消息创建时间 （整型）
    private long createTime;
    // 消息类型（text/image/location/link）
    private String msgType;
    // 消息id，64位整型
    private long msgId;

    public String gettoUserName() {
        return toUserName;
    }

    public void settoUserName(String toUserName) {
        toUserName = toUserName;
    }

    public String getfromUserName() {
        return fromUserName;
    }

    public void setfromUserName(String fromUserName) {
        fromUserName = fromUserName;
    }

    public long getcreateTime() {
        return createTime;
    }

    public void setcreateTime(long createTime) {
        createTime = createTime;
    }

    public String getmsgType() {
        return msgType;
    }

    public void setmsgType(String msgType) {
        msgType = msgType;
    }

    public long getmsgId() {
        return msgId;
    }

    public void setmsgId(long msgId) {
        msgId = msgId;
    }
}
```

### 文本消息

```java
package com.chanshuyi.pojo.message.req;

/**
 * 请求类，文本消息
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class TextMessage extends BaseMessage {
    // 消息内容
    private String content;

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        content = content;
    }
}
```

### 图片消息

```java
package com.chanshuyi.pojo.message.req;

/**
 * 请求类，图片消息
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class ImageMessage extends BaseMessage {
    // 图片链接
    private String picUrl;

    public String getPicUrl() {
        return picUrl;
    }

    public void setPicUrl(String picUrl) {
        picUrl = picUrl;
    }
}
```

### 语音消息

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
    private String mediaId;
    // 语音格式
    private String format;

    public String getmediaId() {
        return mediaId;
    }

    public void setmediaId(String mediaId) {
        mediaId = mediaId;
    }

    public String getformat() {
        return format;
    }

    public void setformat(String format) {
        format = format;
    }
}
```

### 视频消息

```java
package com.chanshuyi.pojo.message.req;

/**
 * 请求类，视频消息
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class VideoMessage extends BaseMessage{
    //视频消息媒体id，可以调用多媒体文件下载接口拉取数据
    private String mediaId;
    //视频消息缩略图的媒体id，可以调用多媒体文件下载接口拉取数据
    private String thumbMediaId;

    public String getMediaId() {
        return mediaId;
    }

    public void setMediaId(String mediaId) {
        this.mediaId = mediaId;
    }

    public String getThumbMediaId() {
        return thumbMediaId;
    }

    public void setThumbMediaId(String thumbMediaId) {
        this.thumbMediaId = thumbMediaId;
    }
}
```

### 小视频消息

```java
package com.chanshuyi.pojo.message.req;

/**
 * 请求类，小视频消息（与视频消息一样）
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class ShortVideoMessage extends VideoMessage{
    //属性与VideoMessage（视频消息）完全一样
}
```

### 地理位置消息

```java
package com.chanshuyi.pojo.message.req;

/**
 * 请求类，位置消息
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class LocationMessage extends BaseMessage {
    // 地理位置维度
    private String Location_X;
    // 地理位置经度
    private String Location_Y;
    // 地图缩放大小
    private String Scale;
    // 地理位置信息
    private String Label;

    public String getLocation_X() {
        return Location_X;
    }

    public void setLocation_X(String location_X) {
        Location_X = location_X;
    }

    public String getLocation_Y() {
        return Location_Y;
    }

    public void setLocation_Y(String location_Y) {
        Location_Y = location_Y;
    }

    public String getScale() {
        return Scale;
    }

    public void setScale(String scale) {
        Scale = scale;
    }

    public String getLabel() {
        return Label;
    }

    public void setLabel(String label) {
        Label = label;
    }
}
```

### 链接消息

```java
package com.chanshuyi.pojo.message.req;

/**
 * 请求类，链接消息
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class LinkMessage  extends BaseMessage {
    // 消息标题
    private String Title;
    // 消息描述
    private String Description;
    // 消息链接
    private String Url;

    public String getTitle() {
        return Title;
    }

    public void setTitle(String title) {
        Title = title;
    }

    public String getDescription() {
        return Description;
    }

    public void setDescription(String description) {
        Description = description;
    }

    public String getUrl() {
        return Url;
    }

    public void setUrl(String url) {
        Url = url;
    }
}
```

[官网文档-接收消息-接收普通消息](http://mp.weixin.qq.com/wiki/10/79502792eef98d6e0c6e1739da387346.html)

## 响应类消息对象

回复给微信服务器的消息有6种类型，分别是：

- 文本消息
- 图片消息
- 语音消息
- 视频消息
- 音乐消息
- 图文消息

同样地，我们将共有字段抽取出来作为一个基类：

```java
package com.chanshuyi.pojo.message.resp;
/**
 * 响应类，基类
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class BaseMessage {
    //接收方帐号（收到的OpenID）
    private String ToUserName;
    //开发者微信号
    private String FromUserName;
    //消息创建时间 （整型）
    private long CreateTime;
    //消息类型（text/voice/video/music/news）
    private String MsgType;

    public String getToUserName() {
        return ToUserName;
    }

    public void setToUserName(String toUserName) {
        ToUserName = toUserName;
    }

    public String getFromUserName() {
        return FromUserName;
    }

    public void setFromUserName(String fromUserName) {
        FromUserName = fromUserName;
    }

    public long getCreateTime() {
        return CreateTime;
    }

    public void setCreateTime(long createTime) {
        CreateTime = createTime;
    }

    public String getMsgType() {
        return MsgType;
    }

    public void setMsgType(String msgType) {
        MsgType = msgType;
    }
}
```

### 文本消息

```java
package com.chanshuyi.pojo.message.resp;

/**
 * 请求类，文本消息
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class TextMessage extends BaseMessage {
    // 消息内容
    private String Content;

    public String getContent() {
        return Content;
    }

    public void setContent(String content) {
        Content = content;
    }
}
```

### 图片消息

```java
package com.chanshuyi.pojo.message.resp;

/**
 * 响应类，图片消息
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class ImageMessage extends BaseMessage{

    private Image Image;

    public Image getImage() {
        return Image;
    }

    public void setImage(Image image) {
        Image = image;
    }
}

```

Image类定义如下：

```
package com.chanshuyi.pojo.message.resp;

/**
 * 图片类
 * @author chenyr
 * @date 2015.10.04
 */
public class Image {

    //通过素材管理接口上传多媒体文件，得到的id。
    private String MediaId;

    public String getMediaId() {
        return MediaId;
    }

    public void setMediaId(String mediaId) {
        MediaId = mediaId;
    }
}
```

### 语音消息

```java
package com.chanshuyi.pojo.message.resp;

/**
 * 响应类，语音消息
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class VoiceMessage extends BaseMessage{

    private Voice Voice;

    public Voice getVoice() {
        return Voice;
    }

    public void setVoice(Voice voice) {
        Voice = voice;
    }
}
```

Voice类定义：

```
package com.chanshuyi.pojo.message.resp;

/**
 * 音频类
 * @author chenyr
 * @date 2015.10.04
 */
public class Voice {

    //通过素材管理接口上传多媒体文件，得到的id
    private String MediaId;

    public String getMediaId() {
        return MediaId;
    }

    public void setMediaId(String mediaId) {
        MediaId = mediaId;
    }
}
```

### 视频消息

```java
package com.chanshuyi.pojo.message.resp;

/**
 * 响应类，视频消息
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class VideoMessage extends BaseMessage{

    private Video Video;

    public Video getVideo() {
        return Video;
    }

    public void setVideo(Video video) {
        Video = video;
    }
}
```

Video类定义：

```
package com.chanshuyi.pojo.message.resp;

/**
 * 视频类
 * @author chenyr
 * @date 2015.10.04
 */
public class Video {

    //通过素材管理接口上传多媒体文件，得到的id
    private String MediaId;
    //视频消息的标题
    private String Title;
    //视频消息的描述
    private String Description;

    public String getMediaId() {
        return MediaId;
    }

    public void setMediaId(String mediaId) {
        MediaId = mediaId;
    }

    public String getTitle() {
        return Title;
    }

    public void setTitle(String title) {
        Title = title;
    }

    public String getDescription() {
        return Description;
    }

    public void setDescription(String description) {
        Description = description;
    }
}
```

### 音乐消息

```java
package com.chanshuyi.pojo.message.resp;

/**
 * 响应类，音乐消息
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class MusicMessage extends BaseMessage{

    private Music music;

    public Music getMusic() {
        return music;
    }

    public void setMusic(Music music) {
        this.music = music;
    }
}
```

Music类的定义如下：

```java
package com.chanshuyi.pojo.message.resp;

/**
 * 音乐类
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class Music {
    // 音乐名称
    private String Title;
    // 音乐描述
    private String Description;
    // 音乐链接
    private String MusicUrl;
    // 高质量音乐链接，WIFI环境优先使用该链接播放音乐
    private String HQMusicUrl;
    //缩略图的媒体id，通过素材管理接口上传多媒体文件，得到的id
    private String ThumbMediaId;

    public String getTitle() {
        return Title;
    }

    public void setTitle(String title) {
        Title = title;
    }

    public String getDescription() {
        return Description;
    }

    public void setDescription(String description) {
        Description = description;
    }

    public String getMusicUrl() {
        return MusicUrl;
    }

    public void setMusicUrl(String musicUrl) {
        MusicUrl = musicUrl;
    }

    public String getHQMusicUrl() {
        return HQMusicUrl;
    }

    public void setHQMusicUrl(String musicUrl) {
        HQMusicUrl = musicUrl;
    }

    public String getThumbMediaId() {
        return ThumbMediaId;
    }

    public void setThumbMediaId(String thumbMediaId) {
        ThumbMediaId = thumbMediaId;
    }
}
```

### 图文消息

```java
package com.chanshuyi.pojo.message.resp;

import java.util.List;

/**
 * 响应类，图文消息
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class NewsMessage extends BaseMessage{
    //图文消息个数，限制为10条以内
    private int NewsCount;
    //多条图文消息信息，默认第一个item为大图,注意，如果图文数超过10，则将会无响应
    private List<News> News;

    public int getNewsCount() {
        return NewsCount;
    }

    public void setNewsCount(int newsCount) {
        NewsCount = newsCount;
    }

    public List<News> getNews() {
        return News;
    }

    public void setNews(List<News> news) {
        News = news;
    }
}
```

News类的定义如下：

```java
package com.chanshuyi.pojo.message.resp;

/**
 * 文章类
 *
 * @author chenyr
 * @date 2015.10.03
 */
public class News {

    private String Title;

    private String Description;
    //图片链接，支持JPG、PNG格式，较好的效果为大图360*200，小图200*200
    private String PicUrl;
    //点击图文消息跳转链接
    private String Url;

    public String getTitle() {
        return Title;
    }

    public void setTitle(String title) {
        Title = title;
    }

    public String getDescription() {
        return Description;
    }

    public void setDescription(String description) {
        Description = description;
    }

    public String getPicUrl() {
        return PicUrl;
    }

    public void setPicUrl(String picUrl) {
        PicUrl = picUrl;
    }

    public String getUrl() {
        return Url;
    }

    public void setUrl(String url) {
        Url = url;
    }
}
```

[官网文档-发送消息-被动回复消息](http://mp.weixin.qq.com/wiki/14/89b871b5466b19b3efa4ada8e577d45e.html)

## 发送、接收消息类型

定义一个存放发送、接收常用消息类型（MsgType）的常量类：

```
package com.chanshuyi.common;

/**
 * @author chenyr
 * @date 2015.10.04
 */
public class MessageTypeDef {

    /**
     * 返回消息类型：文本
     */
    public static final String RESP_MESSAGE_TYPE_TEXT = "text";

    /**
     * 请求消息类型：图片
     */
    public static final String RESP_MESSAGE_TYPE_IMAGE = "image";

    /**
     * 请求消息类型：音频
     */
    public static final String RESP_MESSAGE_TYPE_VOICE = "voice";

    /**
     * 请求消息类型：视频
     */
    public static final String RESP_MESSAGE_TYPE_VIDEO = "video";

    /**
     * 返回消息类型：音乐
     */
    public static final String RESP_MESSAGE_TYPE_MUSIC = "music";

    /**
     * 返回消息类型：图文
     */
    public static final String RESP_MESSAGE_TYPE_NEWS = "news";


    /******************************************************************/

    /**
     * 请求消息类型：文本
     */
    public static final String REQ_MESSAGE_TYPE_TEXT = "text";

    /**
     * 请求消息类型：图片
     */
    public static final String REQ_MESSAGE_TYPE_IMAGE = "image";

    /**
     * 请求消息类型：链接
     */
    public static final String REQ_MESSAGE_TYPE_LINK = "link";

    /**
     * 请求消息类型：地理位置
     */
    public static final String REQ_MESSAGE_TYPE_LOCATION = "location";

    /**
     * 请求消息类型：音频
     */
    public static final String REQ_MESSAGE_TYPE_VOICE = "voice";

    /**
     * 请求消息类型：视频
     */
    public static final String REQ_MESSAGE_TYPE_VIDEO = "video";

    /**
     * 请求消息类型：小视频
     */
    public static final String REQ_MESSAGE_TYPE_SHORTVIDEO = "shortvideo";

    /**
     * 请求消息类型：推送
     */
    public static final String REQ_MESSAGE_TYPE_EVENT = "event";


    /******************************************************************/


    /**
     * 事件类型：subscribe(订阅)
     */
    public static final String EVENT_TYPE_SUBSCRIBE = "subscribe";

    /**
     * 事件类型：unsubscribe(取消订阅)
     */
    public static final String EVENT_TYPE_UNSUBSCRIBE = "unsubscribe";

    /**
     * 事件类型：CLICK(自定义菜单点击事件)
     */
    public static final String EVENT_TYPE_CLICK = "CLICK";
}
```