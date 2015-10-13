# 高级接口使用——获取关注者列表

开发者可以使用接口，对公众平台的分组进行查询、创建、修改操作，也可以使用接口在需要时移动用户到某个分组。

[官方文档>分组管理接口](http://mp.weixin.qq.com/wiki/13/be5272dc4930300ba561d927aead2569.html)

## 创建分组

在WeixinUtil类中增加createGroup方法：

```java
/**
 * 创建分组
 * @param accessToken
 * @param groupName
 * @return
 */
@SuppressWarnings({"unchecked", "deprecated"})
public static WeixinGroup createGroup(String accessToken, String groupName){
    WeixinGroup weixinGroup = null;

    //拼接请求地址
    String requestUrl = "https://api.weixin.qq.com/cgi-bin/groups/create?access_token=ACCESS_TOKEN";
    requestUrl = requestUrl.replace("ACCESS_TOKEN", accessToken);

    //拼接请求数据
    JSONObject nameJsonObject = new JSONObject();
    nameJsonObject.put("name", groupName);
    JSONObject groupJsonObject = new JSONObject();
    groupJsonObject.put("group", nameJsonObject);

    //发送请求
    JSONObject jsonObject = httpsRequest(requestUrl, "POST", groupJsonObject.toString());
    if(null != jsonObject){
        try{
            JSONObject rtnGrpJsonObject = (JSONObject)jsonObject.get("group");
            weixinGroup = new WeixinGroup();
            weixinGroup.setId(rtnGrpJsonObject.getInt("id"));
            weixinGroup.setName(rtnGrpJsonObject.getString("name"));
            weixinGroup.setCount(0);
        }catch(Exception e){
            weixinGroup = null;
            int errCode = jsonObject.getInt("errcode");
            String errMsg = jsonObject.getString("errmsg");
            log.error("创建分组失败：errcode:{} errmsg{}", errCode, errMsg);
        }
    }

    return weixinGroup;
}
```

其中WeixinGroup类为：

```java
package com.chanshuyi.pojo;

/**
 * @author chenyr
 * @date 2015.10.13
 */
public class WeixinGroup {

    private int id;
    //分组名
    private String name;
    //用户数
    private int count;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
    }
}
```

创建测试类：

```java
@Test
public void cerateGroupTest(){
    WeixinGroup weixinGroup = WeixinUtil.createGroup(accessToken, "VIP客户");
    log.info("分组ID：{}，组名：{}，用户数量：{}。", weixinGroup.getId(), weixinGroup.getName(), weixinGroup.getCount() );
}
```

## 查询所有分组

封装的方法如下：

```java
/**
 * 查询所有分组
 * @param accessToken
 * @return
 */
@SuppressWarnings({"unchecked", "deprecation"})
public static List<WeixinGroup> getGroups(String accessToken){
    List<WeixinGroup> weixinGroupList = null;

    //拼接请求地址
    String requestUrl = "https://api.weixin.qq.com/cgi-bin/groups/get?access_token=ACCESS_TOKEN";
    requestUrl = requestUrl.replace("ACCESS_TOKEN", accessToken);

    //发送请求
    JSONObject jsonObject = httpsRequest(requestUrl, "GET", "");
    if(null != jsonObject){
        try{
            JSONArray jsonArray = (JSONArray)jsonObject.get("groups");
            weixinGroupList = JSONArray.toList(jsonArray, WeixinGroup.class);
        }catch(Exception e){
            weixinGroupList = null;
            int errCode = jsonObject.getInt("errcode");
            String errMsg = jsonObject.getString("errmsg");
            log.error("获取关注者列表失败：errcode:{} errmsg{}", errCode, errMsg);
        }
    }

    return weixinGroupList;
}
```

## 查询用户所在分组

封装的方法如下：

```java
/**
 * 查询用户所在分组
 * @param accessToken
 * @param openid
 * @return
 */
public static int getGroupId(String accessToken, String openid){
    int groupId = -1;

    //拼接请求地址
    String requestUrl = "https://api.weixin.qq.com/cgi-bin/groups/getid?access_token=ACCESS_TOKEN";
    requestUrl = requestUrl.replace("ACCESS_TOKEN", accessToken);

    //拼接参数
    JSONObject paramJsonObject = new JSONObject();
    paramJsonObject.put("openid", openid);

    //发送请求
    JSONObject jsonObject = httpsRequest(requestUrl, "POST", paramJsonObject.toString());
    if(null != jsonObject){
        try{
            groupId = jsonObject.getInt("groupid");
        }catch (Exception e){
            int errCode = jsonObject.getInt("errcode");
            String errMsg = jsonObject.getString("errmsg");
            log.error("查询用户所在分组失败：errcode:{} errmsg{}", errCode, errMsg);
        }
    }

    return groupId;
}
```

## 修改分组名

封装的方法如下：

```java
/**
 * 修改分组名
 * @param accessToken
 * @param id
 * @param groupName
 * @return
 */
@SuppressWarnings({"unchecked", "deprecated"})
public static boolean updateGroup(String accessToken, int id, String groupName){
    boolean flag = false;
    //拼接请求地址
    String requestUrl = "https://api.weixin.qq.com/cgi-bin/groups/update?access_token=ACCESS_TOKEN";
    requestUrl = requestUrl.replace("ACCESS_TOKEN", accessToken);

    //拼接请求数据
    JSONObject infoJsonObject = new JSONObject();
    infoJsonObject.put("id", id);
    infoJsonObject.put("name", groupName);
    JSONObject groupJsonObject = new JSONObject();
    groupJsonObject.put("group", infoJsonObject);

    //发送请求
    JSONObject jsonObject = httpsRequest(requestUrl, "POST", groupJsonObject.toString());
    if(null != jsonObject){
        int errCode = jsonObject.getInt("errcode");
        String errMsg = jsonObject.getString("errmsg");
        if(0 == errCode){
            flag = true;
        }else{
            flag = false;
            log.error("创建分组失败：errcode:{} errmsg{}", errCode, errMsg);
        }
    }
    return flag;
}   
```

## 移动用户分组

封装的方法如下：

```java
/**
 * 移动用户分组
 * @param accessToken
 * @param openid
 * @param toGroupid
 * @return
 */
public static boolean updateUserGroup(String accessToken, String openid, int toGroupid){
    boolean flag = false;

    //拼接请求地址
    String requestUrl = "https://api.weixin.qq.com/cgi-bin/groups/members/update?access_token=ACCESS_TOKEN";
    requestUrl = requestUrl.replace("ACCESS_TOKEN", accessToken);

    //拼接请求参数
    JSONObject paramJsonObject = new JSONObject();
    paramJsonObject.put("openid", openid);
    paramJsonObject.put("to_groupid", toGroupid);

    //发送请求
    JSONObject jsonObject = httpsRequest(requestUrl, "POST", paramJsonObject.toString());
    if(null != jsonObject){
        int errCode = jsonObject.getInt("errcode");
        String errMsg = jsonObject.getString("errmsg");
        if(0 == errCode){
            flag = true;
        }else{
            flag = false;
            log.error("创建分组失败：errcode:{} errmsg{}", errCode, errMsg);
        }
    }

    return flag;
}
```