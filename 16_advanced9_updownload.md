# 高级接口使用——上传及下载多媒体文件

通过这个接口，用户可以上传和下载素材（图片、视频、语音），使得与用户的互动变得更加丰富。

## 素材上传接口

上传素材支持四种类型：image、voice、video、thumb，分别是图片、语音、视频、缩略图。

我们在WeixinUtil.java类中增加uploadMedia()方法：

```java
/**
     * 上传媒体文件
     * @param accessToken
     * @param type 媒体文件类型（image、voice、video和thumb）
     * @param mediaFileUrl 媒体文件的url
     * @return
     */
    public static WeixinMedia uploadMedia(String accessToken, String type, String mediaFileUrl){
        WeixinMedia weixinMedia = null;
        //拼装请求地址
        String uploadMediaUrl = "http://api.weixin.qq.com/cgi-bin/media/upload?access_token=ACCESS_TOKEN&type=TYPE";
        uploadMediaUrl = uploadMediaUrl.replace("ACCESS_TOKEN", accessToken).replace("TYPE", type);

        //定义数据分隔符
        String boundary = "------------7da2e536604c8";
        try{
            URL uploadUrl = new URL(uploadMediaUrl);
            HttpURLConnection uploadConn = (HttpURLConnection)uploadUrl.openConnection();
            uploadConn.setDoOutput(true);
            uploadConn.setDoInput(true);
            uploadConn.setRequestMethod("POST");
            //设置请求头Content-Type
            uploadConn.setRequestProperty("Content-Type", "multipart/form-data;boundary=" + boundary);
            //获取媒体文件上传的输出流（往微信服务器写数据）
            OutputStream outputStream = uploadConn.getOutputStream();

            URL mediaUrl = new URL(mediaFileUrl);
            HttpURLConnection mediaConn = (HttpURLConnection)mediaUrl.openConnection();
            mediaConn.setDoInput(true);
            mediaConn.setRequestMethod("GET");

            //从请求头中获取内容类型
            String contentType = mediaConn.getHeaderField("Content-Type");
            //根据内容类型判断用户扩展名
            String fileExt = CommonUtil.getFileExt(contentType);
            //请求体开始
            outputStream.write(("--" + boundary + "\r\n").getBytes());
            outputStream.write(String.format("Content-Disposition:form-data;name=\"media\"; filename=\"file1%s\"\r\n", fileExt).getBytes());
            outputStream.write(String.format("Content-Type:%s\r\n\r\n", contentType).getBytes());

            //获取媒体文件的输入流
            BufferedInputStream bis = new BufferedInputStream(mediaConn.getInputStream());
            byte[] buf = new byte[8096];
            int size = 0;
            while((size = bis.read(buf)) != -1){
                //将媒体文件写道输出流（往微信服务器写数据）
                outputStream.write(buf, 0, size);
            }
            //请求体结束
            outputStream.write(("\r\n--" + boundary + "--\r\n").getBytes());
            outputStream.close();
            bis.close();
            mediaConn.disconnect();

            //获取媒体文件上传的输入流（从微信服务器读数据）  读取返回状态
            InputStream inputStream = uploadConn.getInputStream();
            InputStreamReader inputStreamReader = new InputStreamReader(inputStream, "utf-8");
            BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
            StringBuffer buffer = new StringBuffer();
            String str = null;
            while((str = bufferedReader.readLine()) != null){
                buffer.append(str);
            }
            bufferedReader.close();
            inputStreamReader.close();
            //释放资源
            inputStream.close();
            inputStream = null;
            uploadConn.disconnect();

            //使用JSON-lib解析返回结果
            JSONObject jsonObject = JSONObject.fromObject(buffer.toString());
            System.out.println(jsonObject);
            weixinMedia = new WeixinMedia();
            weixinMedia.setType(jsonObject.getString("type"));
            //type等于thumb时的返回结果和其他类型不一样
            if("thumb".equals(type)){
                weixinMedia.setMediaId(jsonObject.getString("thumb_media_id"));
            }else{
                weixinMedia.setMediaId(jsonObject.getString("media_id"));
                weixinMedia.setCreatedAt(jsonObject.getInt("created_at"));
            }
        }catch (Exception e){
            weixinMedia = null;
            log.error("上传媒体文件失败：{}" + e);
        }

        return weixinMedia;
    }
```

其中WeixinMedia定义如下：

```java
package com.chanshuyi.pojo;

/**
 * @author chenyr
 * @date 2015.10.31
 */
public class WeixinMedia {
    //媒体文件类型
    private String type;
    //媒体文件标识或缩略图的媒体文件标识
    private String mediaId;
    //媒体文件上传时间
    private int createdAt;

    public String getMediaId() {
        return mediaId;
    }

    public void setMediaId(String mediaId) {
        this.mediaId = mediaId;
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public int getCreatedAt() {
        return createdAt;
    }

    public void setCreatedAt(int createdAt) {
        this.createdAt = createdAt;
    }
}
```

CommonUtil.getFileExt()方法如下：

```java
public static String getFileExt(String contentType){
    String fileExt = "";
    if("image/jpeg".equals(contentType)){
        fileExt = ".jpg";
    }else if("audio/mpeg".equals(contentType)){
        fileExt = ".mp3";
    }else if("audio/amr".equals(contentType)){
        fileExt = ".amr";
    }else if("video/mp4".equals(contentType)){
        fileExt = ".mp4";
    }else if("video/mpeg4".equals(contentType)){
        fileExt = ".mp4";
    }
    return  fileExt;
}
```

之后我们在本地启动一个Tomcat服务器，将一张图片放置在Tomcat服务器里。之后写一个测试类来进行测试：

```java
@Test
public void uploadMediaTest(){ 
    WeixinMedia weixinMedia = WeixinUtil.uploadMedia(accessToken, "voice", "http://localhost:8080/test.mp3");
    System.out.println(weixinMedia.getMediaId());
    System.out.println(weixinMedia.getType());
    System.out.println(weixinMedia.getCreatedAt());
}
```

## 下载素材文件

我们在WeixinUtil里增加一个getMedia方法，来下载素材文件：

```java
/**
 * 下载媒体文件（只支持下载图片、语音和缩略图文件，不支持下载视频文件）
 * @param accessToken
 * @param mediaId
 * @param savePath
 * @return
 */
public static String getMedia(String accessToken, String mediaId, String savePath){
    String filePath = null;
    //拼接请求地址
    String requestUrl = "https://api.weixin.qq.com/cgi-bin/media/get?access_token=ACCESS_TOKEN&media_id=MEDIA_ID";
    requestUrl = requestUrl.replace("ACCESS_TOKEN", accessToken).replace("MEDIA_ID", mediaId);

    try{
        URL url = new URL(requestUrl);
        HttpURLConnection conn = (HttpURLConnection)url.openConnection();
        conn.setDoInput(true);
        conn.setRequestMethod("GET");
        if(!savePath.endsWith(File.separator)){
            savePath += File.separator;
        }
        //根据内容类型获取拓展名
        String fileExt = CommonUtil.getFileExt(conn.getHeaderField("Content-Type"));
        //将mediaId作为文件名
        filePath = savePath + mediaId + fileExt;

        BufferedInputStream bis = new BufferedInputStream(conn.getInputStream());
        FileOutputStream fos = new FileOutputStream(new File(filePath));
        byte[] buf = new byte[8096];
        int size = 0;
        while((size = bis.read(buf)) != -1){
            fos.write(buf, 0, size);
        }
        fos.close();
        bis.close();

        conn.disconnect();
        log.info("下载媒体文件成功, filePath=" + filePath);
    }catch (Exception e){
        filePath = "";
        log.error("下载媒体文件失败：{}", e);
    }

    return filePath;
}
```

之后写一个测试类：

```java
@Test
public void downloadMediaTest(){
    WeixinUtil.getMedia(accessToken, "AHA2K_W0yh6skaZCykpV_DgLdXR8WjAIhBOsBQiQfSBEcocLtEY_rufFR-uOvPZo", "d:\\");
}
```

这里要将你传入的mediaId换成你自己的。