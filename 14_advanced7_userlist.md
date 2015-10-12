# 高级接口使用——获取关注者列表

公众号可通过本接口来获取帐号的关注者列表，关注者列表由一串OpenID（加密后的微信号，每个用户对每个公众号的OpenID是唯一的）组成。一次拉取调用最多拉取10000个关注者的OpenID，可以通过多次拉取的方式来满足需求。

正常情况下，微信返回的JSON数据包为：

```xml
{
    "total": 2,
    "count": 2,
    "data": {
        "openid": [
            "OPENID1",
            "OPENID2",
            "OPENID3"
        ]
    },
    "next_openid": "NEXT_OPENID"
}
```

我们先定义一个POJO模型来存放它：

```java
package com.chanshuyi.pojo;

import java.util.List;

/**
 * @author chenyr
 * @date 2015.10.12
 */
public class WeixinUserList {

    //总关注数
    private int total;
    //拉取数量
    private int count;
    //关注者ID
    private List<String> openIdList;
    //下一个关注者的ID
    private String nextOpenId;

    public int getTotal() {
        return total;
    }

    public void setTotal(int total) {
        this.total = total;
    }

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }

    public List<String> getOpenIdList() {
        return openIdList;
    }

    public void setOpenIdList(List<String> openIdList) {
        this.openIdList = openIdList;
    }

    public String getNextOpenId() {
        return nextOpenId;
    }

    public void setNextOpenId(String nextOpenId) {
        this.nextOpenId = nextOpenId;
    }
}
```

之后我们在WeixinUtil类中增加getUserList()方法，这个方法将向微信服务器发送https请求，请求用户的基本信息。

```java
public static WeixinUserList getUserList(String accessToken, String nextOpenId){
    WeixinUserList weixinUserList = null;

    //拼接请求地址
    String requestUrl = "https://api.weixin.qq.com/cgi-bin/user/get?access_token=ACCESS_TOKEN&next_openid=NEXT_OPENID";
    requestUrl = requestUrl.replace("ACCESS_TOKEN", accessToken);
    requestUrl = requestUrl.replace("NEXT_OPENID", nextOpenId);

    //发送请求
    JSONObject jsonObject = httpsRequest(requestUrl, "GET", "");

    if(null != jsonObject){
        try{
            weixinUserList = new WeixinUserList();
            weixinUserList.setTotal(jsonObject.getInt("total"));
            weixinUserList.setCount(jsonObject.getInt("count"));

            JSONObject dataObject = (JSONObject)jsonObject.get("data");
            weixinUserList.setOpenIdList(JSONArray.toList(dataObject.getJSONArray("openid")));
            weixinUserList.setNextOpenId(nextOpenId);
        }catch(Exception e){
            weixinUserList = null;
            int errCode = jsonObject.getInt("errcode");
            String errMsg = jsonObject.getString("errmsg");
            log.error("获取关注者列表失败：errcode:{} errmsg{}", errCode, errMsg);
        }
    }

    return weixinUserList;
}
```

之后我们写一个测试类测试一下：

```java
@Test
public void getUserListTest(){
    String accessToken = "ACCESS_TOKEN";
    WeixinUserList userList = WeixinUtil.getUserList(accessToken, "");
    log.info("总关注人数：{}" + userList.getTotal());
    log.info("本次拉取人数：{}" + userList.getCount());
    log.info("下一个关注者OPENID：{}" + userList.getNextOpenId());
    log.info("******拉取关注着OPENID列表********");
    for(String openid : userList.getOpenIdList()){
        log.info(openid);
    }
}
```
