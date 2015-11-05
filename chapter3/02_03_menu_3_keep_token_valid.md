# 获取access_token并保证长期有效

access_token是请求一些接口所必须的一个令牌，只有有了这个access_token我们才能成功调用这个接口。根据微信官方文档的解释，access_token的有效期只有两个小时，因此我们必须每两个小时就更新access_token。因此如何动态更新access_token就成了一个问题。

我们实现access_token长期有效的基本思路：Web服务器启动时就加载一个Servlet，在Servlet的init()方法中启动一个线程，在线程的run()方法中通过死循环+Thread.sleep()的方式定期获取access_token，然后将获取到的access_token保存在public static修饰的变量中。

在项目中的其他类，可以通过调用 TokenThread.accessToken.getToken() 来得到接口访问凭证access_token。

下面我们看看我们如何通过Java来实现动态更新access_token。

## 创建Initservlet类

```java
package com.chanshuyi.wechat.servlet;

import com.chanshuyi.wechat.thread.TokenThread;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;

/**
 * 定时获取access_token服务
 *
 * @author liuyq
 * @date 2013-05-02
 */
public class InitServlet extends HttpServlet {

    private static Logger log = LoggerFactory.getLogger(InitServlet.class);

    public void init() throws ServletException {
        //将web.xml中的appid,appsecret赋值到TokenThread中
        log.info("**************************************************");
        log.info("**** Start init ACCESS_TOKEN Servlet. ");
        TokenThread.appid = getInitParameter("appid");
        TokenThread.appsecret = getInitParameter("appsecret");

        log.info("**** Weixin api appid:{}", TokenThread.appid);
        log.info("**** Weixin api appsecret:{}", TokenThread.appsecret);

        // 未配置appid、appsecret时给出提示
        if ("".equals(TokenThread.appid) || "".equals(TokenThread.appsecret)) {
            log.error("appid and appsecret configuration error, please check carefully.");
        } else {
            // 启动定时获取access_token的线程
            new Thread(new TokenThread()).start();
        }
    }
}
```

在web.xml增加如下配置：

```xml
<!-- 定期获取AccessToekn、创建菜单 -->
<servlet>
    <servlet-name>initServlet</servlet-name>
    <servlet-class>
        com.chanshuyi.wechat.servlet.InitServlet
    </servlet-class>
    <!-- 配置获取access_token所需参数appid和appsecret -->
    <init-param>
        <param-name>appid</param-name>
        <param-value>WECHAT_APPID</param-value>
    </init-param>
    <init-param>
        <param-name>appsecret</param-name>
        <param-value>WECHAT_APPSECRET</param-value>
    </init-param>
    <load-on-startup>0</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>initServlet</servlet-name>
    <url-pattern>/kjff343424dius</url-pattern>
</servlet-mapping>
```

将WECHAT_APPID、WECHAT_APPSECRET替换为你自己的。在这个类中我们创建了一个在启动项目的时候就运动的Servlet，并另外启动了一个定时更新Token值的线程：TokenThread类。

其中TokenThread是一个定时发送请求以更新token的线程类，代码如下：

```java
package com.chanshuyi.wechat.thread;

import com.chanshuyi.pojo.AccessToken;
import com.chanshuyi.util.WeixinUtil;
import com.chanshuyi.wechat.main.MenuManager;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * 定时获取微信access_token的线程
 *
 * @author chenyr
 * @date 2013-05-02
 */
public class TokenThread implements Runnable {
    private static Logger log = LoggerFactory.getLogger(TokenThread.class);
    // 第三方用户唯一凭证
    public static String appid = "";
    // 第三方用户唯一凭证密钥
    public static String appsecret = "";
    public static AccessToken accessToken = null;

    public void run() {
        //向微信服务器请求创建菜单
        log.info("**** Start to create self-define Menu.");
        log.info("**** Ready to get ACCESS_TOKEN one time every two hours.");
        while (true) {
            try {
                accessToken = WeixinUtil.getAccessToken(appid, appsecret);
                if (null != accessToken) {
                    log.info("get access_token success，valid time {} second token:{}", accessToken.getExpiresIn(), accessToken.getToken());
                    // 休眠7000秒
                    Thread.sleep((accessToken.getExpiresIn() - 200) * 1000);
                } else {
                    // 如果access_token为null，60秒后再获取
                    Thread.sleep(60 * 1000);
                }
            } catch (InterruptedException e) {
                try {
                    Thread.sleep(60 * 1000);
                } catch (InterruptedException e1) {
                    log.error("{}", e1);
                }
                log.error("{}", e);
            }
        }
    }
}
```

上面的代码里，我们让线程在其token无效前200秒就重新向微信服务器请求token，以保持token一直是有效的。

其中`accessToken = WeixinUtil.getAccessToken(appid, appsecret);` 是一个向微信服务器请求HTTPS请求以获取token的方法，我们将其放入WeixinUtil类下，代码如下：

```java
/**
     * 获取access_token
     *
     * @param appid 凭证
     * @param appsecret 密钥
     * @return
     */
    public static AccessToken getAccessToken(String appid, String appsecret) {
        log.info("**** ready to get ACCESS_TOKEN ");
        AccessToken accessToken = null;

        String requestUrl = access_token_url.replace("APPID", appid).replace("APPSECRET", appsecret);
        JSONObject jsonObject = httpRequest(requestUrl, "GET", null);
        // 如果请求成功
        if (null != jsonObject) {
            try {
                accessToken = new AccessToken();
                accessToken.setToken(jsonObject.getString("access_token"));
                accessToken.setExpiresIn(jsonObject.getInt("expires_in"));
            } catch (JSONException e) {
                accessToken = null;
                // 获取token失败
                log.error("获取token失败 errcode:{} errmsg:{}", jsonObject.getInt("errcode"), jsonObject.getString("errmsg"));
            }
        }

        log.info("**** Appid:{}" + appid);
        log.info("**** AppSecret:{}" + appsecret);
        log.info("**** Token:{}" + accessToken.getToken());
        log.info("**** Expires_In:{}" + accessToken.getExpiresIn());
        return accessToken;
    }
```

到这里自定义创建菜单的介绍就到此结束了。下面我们将介绍微信的九大高级接口，如果你的公众号是服务号并且通过了微信认证的话，就可以调用这些接口。
