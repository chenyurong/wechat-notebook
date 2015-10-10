# 高级接口——生成带参数的二维码

为了满足用户渠道推广分析的需要，公众平台提供了生成带参数二维码的接口。使用该接口可以获得多个带不同场景值的二维码，用户扫描后，公众号可以接收到事件推送。

目前有2种类型的二维码：

>1、临时二维码，是有过期时间的，最长可以设置为在二维码生成后的7天（即604800秒）后过期，但能够生成较多数量。临时二维码主要用于帐号绑定等不要求二维码永久保存的业务场景   
2、永久二维码，是无过期时间的，但数量较少（目前为最多10万个）。永久二维码主要用于适用于帐号绑定、用户来源统计等场景。

用户扫描带场景值二维码时，可能推送以下两种事件：

如果用户还未关注公众号，则用户可以关注公众号，关注后微信会将带场景值关注事件推送给开发者。
如果用户已经关注公众号，在用户扫描后会自动进入会话，微信也会将带场景值扫描事件推送给开发者。

用户扫描二维码后收到的JSON数据格式可以查询[这里->接收事件推送](http://mp.weixin.qq.com/wiki/2/5baf56ce4947d35003b86a9805634b1e.html#.E6.89.AB.E6.8F.8F.E5.B8.A6.E5.8F.82.E6.95.B0.E4.BA.8C.E7.BB.B4.E7.A0.81.E4.BA.8B.E4.BB.B6)


获取带参数的二维码的过程包括两步，首先创建二维码ticket，然后凭借ticket到指定URL换取二维码。

## 创建二维码ticket

这里我们在MessageUtil中增加三个方法：makeQRCode/makeTemporaryQRCode/makePermanentQRCode，其中后两个分别代表创建临时二维码JSON数据、创建永久二维码JSON数据，他们都调用了makrQRCode方法。

```java
   /**
     * 创建临时二维码数据
     * @param expireSeconds
     * @param sceneId
     * @return
     */
    public static String makeTemporaryQRCode(int expireSeconds, int sceneId){
        return makeQRCode("QR_SCENE", expireSeconds, sceneId);
    }

    /**
     * 创建永久二维码数据
     * @param expireSeconds
     * @param sceneId
     * @return
     */
    public static String makePermanentQRCode(int expireSeconds, int sceneId){
        return makeQRCode("QR_LIMIT_SCENE", expireSeconds, sceneId);
    }

    /**
     * 组装二维码POST数据
     * @param action_name QR_SCENE | QR_LIMIT_SCENE
     * @param expireSeconds 过期时间。该二维码有效时间，以秒为单位。最大不超过604800（即7天）。
     * @param sceneId 场景ID
     * @return
     */
    public static String makeQRCode(String action_name, int expireSeconds, int sceneId){
        JSONObject sceneIdJson = new JSONObject();
        sceneIdJson.put("scene_id", sceneId);

        JSONObject sceneJson = new JSONObject();
        sceneJson.put("scene", sceneIdJson.toString());

        JSONObject json = new JSONObject();
        json.put("expire_seconds", expireSeconds);
        json.put("action_name", action_name);
        json.put("action_info", sceneJson.toString());

        return json.toString();
    }
```

之后创建WeixinUtil.createQRCode方法，作用是向微信服务器发送创建二维码的请求。

```java
    /**
     * 发送创建二维码请求
     * @param accessToken 授权码
     * @param msg POST数据
     * @return
     */
    public static WeixinQRCode createQRCode(String accessToken, String msg){
        WeixinQRCode weixinQRCode = null;

        //拼接请求地址
        String requestUrl = "https://api.weixin.qq.com/cgi-bin/qrcode/create?access_token=ACCESS_TOKEN";
        requestUrl = requestUrl.replace("ACCESS_TOKEN", accessToken);
        //需要提交的JSON数据
        JSONObject jsonObject = httpsRequest(requestUrl, "POST", msg);

        if(null != jsonObject){
            try{
                weixinQRCode = new WeixinQRCode();
                weixinQRCode.setTicket(jsonObject.getString("ticket"));
//                weixinQRCode.setExpireSeconds(jsonObject.getInt("expire_seconds")); //微信官方文档中存在，但是实际返回数据里没有
                weixinQRCode.setUrl(jsonObject.getString("url"));
            }catch(Exception e){
                weixinQRCode = null;
                int errorCode = jsonObject.getInt("errcode");
                String errorMsg = jsonObject.getString("errmsg");
                log.error("Create QRCode fail errorcode:{] errmsg:{]", errorCode, errorMsg);
            }
        }

        return weixinQRCode;
    }
```

## 用ticket换取二维码

在WeixinUtil类中增加getQRCode方法，用来向微信服务器发送换取二维码的请求：

```java
    public static String getQRCode(String ticket, String savePath){
        String filePath = null;
        //拼接请求地址
        String requestUrl = "https://mp.weixin.qq.com/cgi-bin/showqrcode?ticket=TICKET";
        requestUrl = requestUrl.replace("TICKET", MessageUtil.urlEncodeUTF8(ticket));
        //发送请求并存储数据
        try{
            URL url = new URL(requestUrl);
            HttpsURLConnection conn = (HttpsURLConnection)url.openConnection();
            conn.setDoInput(true);
            conn.setRequestMethod("GET");

            if(!savePath.endsWith("/")){
                savePath += "/";
            }

            //将ticket作为文件名
            filePath = savePath + ticket + ".jpg";

            //将微信服务器返回的输入流写入文件
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
            log.info("Get QRCode form ticket success. filePath:" + filePath);
        }catch(Exception e){
            filePath = null;
            log.error("Get QRCode form ticket fail.");
            e.printStackTrace();
        }
        return filePath;
    }
```

getQRCode()方法的ticket可能包含加好"+"，所以必须对ticket进行了URL编码。MessageUtil.urlEncodeUTF8()方法代码如下：

```java
/**
 * 进行utf-8的url编码
 * @param source
 * @return
 */
public static String urlEncodeUTF8(String source){
    String result = source;
    try{
        result = URLEncoder.encode(source, "utf-8");
    }catch(Exception e){
        e.printStackTrace();
    }
    return result;
}
```

我们用一个测试类来测试一下：

```java
@Test
public void getQRCodeTest(){
    String accessToken = "hb2bW5RAXTuPYfeghF2j3StOELCTqgaGPOA6yagLi-mik-07SOkERh_pOxPYTb10ra0HgP0d9s5O-9XxUtvFEXPH6a051NiXrlmH1e-ovDQ";
    WeixinQRCode qrCode = WeixinUtil.createQRCode(accessToken, MessageUtil.makePermanentQRCode(3600, 127));
    System.out.println(">>>>" + JSONObject.fromObject(qrCode).toString() + "<<<<<<<<<<<");
    System.out.println(WeixinUtil.getQRCode(qrCode.getTicket(), "/Users/yurongchan/TEMP/wechat_temp"));
}	
```

其中将你的accessToken换成你自己的，将图片保存地址也修改一下。

运行测试方法，没问题的话就会得到一张图片。

已关注和未关注的用户扫描二维码后，服务器会得到不同的JSON数据，具体可以查询上面提到的事件推送文档。

我们这里修改CoreService类的部分代码：

```java
// 事件推送
else if (msgType.equals(MessageTypeDef.REQ_MESSAGE_TYPE_EVENT)) {
    //...more
    }else if(requestMap.get("Event").equals("SCAN")){
        //无论之前关注或未关注公众号，都发送消息
        respContent = "场景事件，场景ID为：" + requestMap.get("EventKey");
    }
}
```

其中`requestMap.get("Event").equals("SCAN")`是为了识别已关注用户扫描这种情况。

```java
// 订阅
if (eventType.equals(MessageTypeDef.EVENT_TYPE_SUBSCRIBE)) {
    respContent = "谢谢您的关注！";
    if(requestMap.get("EventKey").contains("qrscene")){
        String value = requestMap.get("EventKey");
        //如果是扫描场景二维码进来的
        respContent = respContent + "场景二维码为：" + value.substring(value.indexOf("_") + 1); 
    }
}
```                

`requestMap.get("EventKey").contains("qrscene")`是为了识别未关注用户扫描后关注公众号这种情况。

## JSON数据格式

关注过公众号，用扫一扫扫描：

```xml
{
    "Ticket": "gQGC8DoAAAAAAAAAASxodHRwOi8vd2VpeGluLnFxLmNvbS9xL0VFalJ2SC1sVVZrbzMwZHFqR1I2AAIEYakYVgMEAAAAAA==",
    "FromUserName": "ovnbes5FThdB-1YI2zSzNgCv8H3A",
    "EventKey": "125",
    "Event": "SCAN",
    "CreateTime": "1444457691",
    "Encrypt": "edDtkw5rWsv14+MIIJLXIm+ZQSwRVKJRIFiKAbIYboS39BMELiiI+Xqc+wiUsc9gjBxHw02OftdFfsDWTOrfKmToh83apYMpIkN3a/d/6hKzoYLQsbG/7MNvRF9WO6S7VBzZwRXB4Yc2D/QiaIrwWlEybRBbyQmFrxRByDsfyrC+8bNoCOmu6HNIIYug/UOWriHy3brNJvBnDqRsypm8TS7V563WcMVyueKEhHvo/8MOv3zgWZIQkvGGyTrIP1Qr/rEvNrzr5T/fpXG/exQh3SRFa8cUKLEU529/d6+3q/ClxP+ueuE8HVLrqGTFgUZAxoKqBa9LOqN/N/j9I4jT2hDqOZbEQwO58b2XeE+b3y4t7/udyzGQIcaY7MfBM1QNq34W9O01l/ka+ohPW7jmZLUAYGwgllI4VRXGfZijKu1pKhNLAB/JB+Kq5RlyRQByqDt1ELxeQv6xSnD8qQWQDHZog/5yoEHBVfPlve+3hRZBxQiALnxK5TkETBjGOdu3+Kb1EVBSJx5dWV3+h6qxDYmJFzidpBoBKsYWHUjz6EYNIBteK6lsV0KqxKdTuz24UYQ5PAfzyXv8Z/5d9HGEqA==",
    "ToUserName": "gh_cb590214c0ba",
    "MsgType": "event"
}
```

未关注微信号，用扫一扫扫描，之后点击关注：

```xml
{
    "Ticket": "gQGC8DoAAAAAAAAAASxodHRwOi8vd2VpeGluLnFxLmNvbS9xL0VFalJ2SC1sVVZrbzMwZHFqR1I2AAIEYakYVgMEAAAAAA==",
    "FromUserName": "ovnbes5FThdB-1YI2zSzNgCv8H3A",
    "EventKey": "qrscene_125",
    "Event": "subscribe",
    "CreateTime": "1444458425",
    "Encrypt": "h0ByQBNBjBLULLYHhSTxXFci48O3k4SmNt/8RunqlSp8a3vukgEd2DSS3Q/75P5wz4LPGDSR+pBZzEuIaYkYT08VA+T3bW9HshcaM6dx39Yt5tsJtYdQ6YUjJlX6XWZDyChwU+my9NM8TK+fVzKwYJ8grRx8jD68U1XgDmvKnfBpf4RN2uAcKgs4X0uQPflRbE/qJ74YjI+/vx11jmuwSVJyHWL7sn65sl5VumTwuWTAJW/l0f5zR31DfPYDWzsIFG0hMaAx/lTCr69H8MTW1eONgpA4IQv5C+On16IbQFQOl4tHLhAkAAO6T0c+IgMnqEiLyENGFAozUppCuKyL09SNTBp7nM9AG2/lfBlGeXnGvUFL3JzwCQLM7ktLItGgUSJUl9vuxmn++CvKKznvQu79dJYJNoeCbUUfai8KBRx9+24WZWxJT+VHDdboReYDGSuUQ186HBrju7a0qQRQdB5W5MPqZ6l/yQsHfooDJRSL3ypMCbpFse3sJKFgKkKCzhCVrdT1L+cpxJCyQGU4kg+SoIkkVxLH3vim9RCfC2ZP3Xky9leuJUI76az/vM2oc7SfIXgc4FUyfz/LdW0TgRlG9dJaAHK2+XdXyIm2FxNMsuuHtAD60h9eK7jcSmii",
    "ToUserName": "gh_cb590214c0ba",
    "MsgType": "event"
}
```

通过搜索公众号关注（普通订阅事件）：

```xml
{
    "FromUserName": "ovnbes-i2A1V5ddWWpsA1mFOg3f8",
    "EventKey": "",
    "Event": "subscribe",
    "CreateTime": "1444458970",
    "Encrypt": "1qcA0BRLJfn/xM8SGPXqacwyuNZspyIKP0z4bDveo6k8XPN8TaPT/nZyB6HoSyGbObzM49DbfEgxDwz/SnJUIDGqwPAj9Jf8WUV8P23xC31aoRR2wOuzj24QFOG6ejJtSHCx5+QMenxdcjjlWd9qGxtMyABG9rGOao4Dd9+vJfFRoEqego8aOZ+/zqYJjqS9ZksqWmp8KRUU6znKwdjy4KnYPiCKHSN38r1T6BhhN95bGEfdx1IiQqPK4PlFIcjj26PzxJ9Vut/+oRNNnBwDkMvGS3u+hmZN1e/0p4+TM4+06qsEJMTysC1r1pIioJ/40jl3+zdzuepLdUGnahU8gYngzGADp8pBTN0ePR93wxUcDoMCg+3bWO314z+KGWSAzthij+7Agwg9LCeZBBuJdC91WyNXbXCCGim+RpuQgRo=",
    "ToUserName": "gh_cb590214c0ba",
    "MsgType": "event"
}
```

NOTE：如果一开始用二维码扫描进入关注公众号，之后又取消关注。最后又通过搜索关注微信公众号，这时候接收到的JSON数据可能是【未关注微信号，用扫一扫扫描，之后点击关注】推送的数据。原因不详。

点击菜单时推送的数据格式：

```xml
{
    "FromUserName": "ovnbes-i2A1V5ddWWpsA1mFOg3f8",
    "EventKey": "11",
    "Event": "CLICK",
    "CreateTime": "1444459114",
    "Encrypt": "OitTQbn6NfaMmFZl2eL5ceWBLvsdTBlnDjNE1+Gyhe0CKNApmcMxFZ0TTA1JkeJqts5OxA752W2xjhBU4w2y1Ms7mOZV8ufrK7v5gvzqirlsoQFoVCaXjVr14YpRGyOu8tTnfBUw6Plv0V2lIX7kjkdcfUpOOdypxuuaLwEXLV/q+CJ4u4gwBgNql8x2LS3iSmdNZgj1vubcui5tAtSQArr1P5IYKscFc4goPF0RAKclLc3kPmRscXpSpq8z0a0aowqEwHoqyOMVKpNEanU23PDOMGsicL2jXjjZyzAUkBNcw5YPaYdhml7A/OqnwV/VquhqMxUJSR8HWwPLKSv51FSxSu13brWCYcm41sepTwVdIdxL+7Ul1ccO0Rzcwr28NS6IUzn7x/bxW/cODUzJ5AjXonnaUsY+HVVzWExU9Uk=",
    "ToUserName": "gh_cb590214c0ba",
    "MsgType": "event"
}
```