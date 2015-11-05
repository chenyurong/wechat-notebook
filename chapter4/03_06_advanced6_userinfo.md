# 高级接口使用——获取用户基本信息

当用户关注公众账号时，公众账号服务器只能获取到用户的OpenID（OpenID是加密后的微信号，能够唯一标识一个用户），对于用户的其他信息一无所知。通过调用获取用户基本信息接口，开发者能够根据用户的OpnID获取用户的其他信息，包括昵称、头像、性别、所在城市、语言和关注时间。

正常情况下，微信返回的JSON数据包为：

```xml
{
    "subscribe": 1, 
    "openid": "o6_bmjrPTlm6_2sgVt7hMZOPfL2M", 
    "nickname": "Band", 
    "sex": 1, 
    "language": "zh_CN", 
    "city": "广州", 
    "province": "广东", 
    "country": "中国", 
    "headimgurl":    "http://wx.qlogo.cn/mmopen/g3MonUZtNHkdmzicIlibx6iaFqAc56vxLSUfpb6n5WKSYVY0ChQKkiaJSgQ1dZuTOgvLLrhJbERQQ4eMsv84eavHiaiceqxibJxCfHe/0", 
   "subscribe_time": 1382694957,
   "unionid": " o6_bmasdasdsad6_2sgVt7hMZOPfL"
   "remark": "",
   "groupid": 0
}
```

我们先定义一个POJO模型来存放它：

```java
package com.chanshuyi.pojo;

/**
 * @author chenyr
 * @date 2015.10.10
 */
public class WeixinUserInfo {
    //用户是否订阅该公众号
    private int subscribe;
    //用户的标识，对当前公众号唯一
    private String openId;

    private String nickname;

    private int sex;

    private String language;

    private String city;

    private String province;

    private String country;
    //用户头像，最后一个数值代表正方形头像大小（有0、46、64、96、132数值可选，0代表640*640正方形头像），用户没有头像时该项为空。若用户更换头像，原有头像URL将失效。
    private String headimgurl;
    //用户关注时间，为时间戳
    private int subscribeTime;
    //只有在用户将公众号绑定到微信开放平台帐号后，才会出现该字段。
    private String unionid;
    //公众号运营者对粉丝的备注，公众号运营者可在微信公众平台用户管理界面对粉丝添加备注
    private String remark;
    //用户所在的分组ID
    private int groupid;

    public int getSubscribe() {
        return subscribe;
    }

    public void setSubscribe(int subscribe) {
        this.subscribe = subscribe;
    }

    public String getOpenId() {
        return openId;
    }

    public void setOpenId(String openId) {
        this.openId = openId;
    }

    public String getNickname() {
        return nickname;
    }

    public void setNickname(String nickname) {
        this.nickname = nickname;
    }

    public int getSex() {
        return sex;
    }

    public void setSex(int sex) {
        this.sex = sex;
    }

    public String getLanguage() {
        return language;
    }

    public void setLanguage(String language) {
        this.language = language;
    }

    public String getCity() {
        return city;
    }

    public void setCity(String city) {
        this.city = city;
    }

    public String getProvince() {
        return province;
    }

    public void setProvince(String province) {
        this.province = province;
    }

    public String getCountry() {
        return country;
    }

    public void setCountry(String country) {
        this.country = country;
    }

    public String getHeadimgurl() {
        return headimgurl;
    }

    public void setHeadimgurl(String headimgurl) {
        this.headimgurl = headimgurl;
    }

    public int getSubscribeTime() {
        return subscribeTime;
    }

    public void setSubscribeTime(int subscribeTime) {
        this.subscribeTime = subscribeTime;
    }

    public String getUnionid() {
        return unionid;
    }

    public void setUnionid(String unionid) {
        this.unionid = unionid;
    }

    public String getRemark() {
        return remark;
    }

    public void setRemark(String remark) {
        this.remark = remark;
    }

    public int getGroupid() {
        return groupid;
    }

    public void setGroupid(int groupid) {
        this.groupid = groupid;
    }

    @Override
    public String toString(){
        return openId + "\n" + subscribe + "\n" + nickname + "\n" + sex + "\n" + language + "\n" + city + "\n" + province + "\n" + country + "\n" 
                + "\n" + headimgurl + "\n" + subscribeTime + "\n" + unionid + "\n" + remark + "\n" + groupid;
    }
}
```

之后我们在WeixinUtil类中增加getUserInfo()方法，这个方法将向微信服务器发送https请求，请求用户的基本信息。

```java
public static WeixinUserInfo getUserInfo(String accessToken, String openId, String lang){
    WeixinUserInfo weixinUserInfo = null;

    //拼接请求地址
    String requestUrl = "https://api.weixin.qq.com/cgi-bin/user/info?access_token=ACCESS_TOKEN&openid=OPENID&lang=LANG";
    requestUrl = requestUrl.replace("ACCESS_TOKEN", accessToken);
    requestUrl = requestUrl.replace("OPENID", openId);
    requestUrl = requestUrl.replace("LANG", lang);

    //发送HTTPS请求
    JSONObject jsonObject = httpsRequest(requestUrl, "GET", null);
    if(null != jsonObject){
        try{
            weixinUserInfo = new WeixinUserInfo();
            weixinUserInfo.setOpenId(jsonObject.getString("openid"));
            weixinUserInfo.setSubscribe(jsonObject.getInt("subscribe"));
            weixinUserInfo.setSubscribeTime(jsonObject.getInt("subscribe_time"));
            weixinUserInfo.setNickname(jsonObject.getString("nickname"));
            weixinUserInfo.setSex(jsonObject.getInt("sex"));
            weixinUserInfo.setCountry(jsonObject.getString("country"));
            weixinUserInfo.setProvince(jsonObject.getString("province"));
            weixinUserInfo.setCity(jsonObject.getString("city"));
            weixinUserInfo.setLanguage(jsonObject.getString("language"));
            weixinUserInfo.setHeadimgurl(jsonObject.getString("headimgurl"));
        }catch(Exception e){
            if(0 == weixinUserInfo.getSubscribe()){
                log.error("用户{}已取消关注", weixinUserInfo.getOpenId() );
            }else{
                int errorCode = jsonObject.getInt("errcode");
                String errorMsg = jsonObject.getString("errmsg");
                log.error("获取用户信息失败 errcode:{} errmsg:{}", errorCode, errorMsg );
            }
        }
    }

    return weixinUserInfo;
}
```

之后我们写一个测试类测试一下：

```java
@Test
public void getUserInfoTest(){
    String accessToken = "2zBI3XFTkfoiY6rJZ5hwyiZ-VsM4_6DIeuAzgtnwlEYkUOBblHMDu4PhtMimqbJzzpfJRe4L_ltovNlApL9WVI3X-aQcjvNl7FfhuFEQ3Vs";
    WeixinUserInfo user = WeixinUtil.getUserInfo(accessToken, "ovnbes-KpItRnUlNSrDsxVL4ZTCE", "zh_CN");
    System.out.println(user.toString());
}
```
