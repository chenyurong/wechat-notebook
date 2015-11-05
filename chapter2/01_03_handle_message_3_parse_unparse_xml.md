# 处理微信服务器消息 - 解析/封装XML消息

上篇文章我们封装接收和响应消息的POJO对象。在这篇文章，我们将介绍如何将XML文档解析为POJO对象，以及如何封装POJO对象为XML文档。

这里需要先加入如下的依赖：

```xml
<!-- 用于解析来自微信服务器的xml数据 -->
<dependency>
    <groupId>dom4j</groupId>
    <artifactId>dom4j</artifactId>
    <version>1.6.1</version>
</dependency>
<!-- 将model转为xml数据 -->
<dependency>
    <groupId>com.thoughtworks.xstream</groupId>
    <artifactId>xstream</artifactId>
    <version>1.3.1</version>
</dependency>
```

## 解析XML消息

这里我们借助于开源框架dom4j去解析xml（这里使用的是dom4j-1.6.1.jar），然后将解析得到的结果存入HashMap。我们将解析XML消息的方法放入MessageUtil类中，其中解析请求消息的方法如下：

```java
/**
 * 解析微信发来的请求（XML）
 *
 * @param request
 * @return
 * @throws Exception
 */
@SuppressWarnings("unchecked")
public static Map<String, String> parseXml(HttpServletRequest request) throws Exception {
    // 将解析结果存储在HashMap中
    Map<String, String> map = new HashMap<String, String>();

    // 从request中取得输入流
    InputStream inputStream = request.getInputStream();
    // 读取输入流
    SAXReader reader = new SAXReader();
    Document document = reader.read(inputStream);
    // 得到xml根元素
    Element root = document.getRootElement();
    // 得到根元素的所有子节点
    List<Element> elementList = root.elements();

    // 遍历所有子节点
    for (Element e : elementList)
        map.put(e.getName(), e.getText());

    // 释放资源
    inputStream.close();
    inputStream = null;

    return map;
} 
```

## 封装XML消息

这里就涉及到如何将响应消息转换成xml返回的问题，这里我们将采用开源框架xstream来实现Java类到xml的转换（这里使用的是xstream-1.3.1.jar）。我们将以下方法放入MessageUtil类中，方法代码如下：

```java
/**
 * 文本消息对象转换成xml
 *
 * @param textMessage 文本消息对象
 * @return xml
 */
public static String textMessageToXml(TextMessage textMessage) {
    xstream.alias("xml", textMessage.getClass());
    return xstream.toXML(textMessage);
}

/**
 * 图片消息对象转换成xml
 *
 * @param imageMessage 图片消息对象
 * @return xml
 */
public static String imageMessageToXml(ImageMessage imageMessage) {
    xstream.alias("xml", imageMessage.getClass());
    return xstream.toXML(imageMessage);
}

/**
 * 音频消息对象转换成xml
 *
 * @param voiceMessage 音频消息对象
 * @return xml
 */
public static String voiceMessageToXml(VoiceMessage voiceMessage) {
    xstream.alias("xml", voiceMessage.getClass());
    return xstream.toXML(voiceMessage);
}

/**
 * （小）视频消息对象转换成xml
 *
 * @param videoMessage 视频消息对象
 * @return xml
 */
public static String videoMessageToXml(VoiceMessage videoMessage) {
    xstream.alias("xml", videoMessage.getClass());
    return xstream.toXML(videoMessage);
}

/**
 * 音乐消息对象转换成xml
 *
 * @param musicMessage 音乐消息对象
 * @return xml
 */
public static String musicMessageToXml(MusicMessage musicMessage) {
    xstream.alias("xml", musicMessage.getClass());
    return xstream.toXML(musicMessage);
}

/**
 * 图文消息对象转换成xml
 *
 * @param articleMessage 图文消息对象
 * @return xml
 */
public static String articleMessageToXml(ArticleMessage articleMessage) {
    xstream.alias("xml", articleMessage.getClass());
    xstream.alias("item", new Article().getClass()); //将Article起个别名item
    return xstream.toXML(articleMessage);
}

/**
 * 扩展xstream，使其支持CDATA块
 *
 * @date 2013-05-19
 */
private static XStream xstream = new XStream(new XppDriver() {
    public HierarchicalStreamWriter createWriter(Writer out) {
        return new PrettyPrintWriter(out) {
            // 对所有xml节点的转换都增加CDATA标记
            boolean cdata = true;

            @SuppressWarnings("unchecked")
            public void startNode(String name, Class clazz) {
                super.startNode(name, clazz);
            }

            protected void writeText(QuickWriter writer, String text) {
                if (cdata) {
                    writer.write("<![CDATA[");
                    writer.write(text);
                    writer.write("]]>");
                } else {
                    writer.write(text);
                }
            }
        };
    }
});
```

上面一共有6个方法，分别对应将文本、图片、音频、视频、音乐、图文消息转化为XML文档。

说明：由于xstream框架本身并不支持CDATA块的生成，40~62行代码是对xtream做了扩展，使其支持在生成xml各元素值时添加CDATA块。

到这里对于微信服务器消息的处理我们已经介绍完了。下面将介绍如何开发自定义菜单。