# 自定义菜单开发 - 设置自定义菜单

微信允许公众号通过开发接口进行自定义菜单的开发，这样我们就可以让自定义菜单有更加丰富的功能。

我们通过开发接口开发自定义菜单的时候，只需要向`菜单创建接口`发送一次请求即可，而不需要每次启动项目都启动。但这里为了方便，我直接将创建菜单放在上一篇说到的InitServlet里。

自定义菜单中的菜单类型可以分为3种不同的类型：

- 没有二级菜单的一级菜单、二级菜单（CommonButton）
- 有二级菜单的一级菜单（ComplexButton）
- 点击跳转到页面的菜单（ViewButton）

按照上面的分析，我们创建出我们需要的POJO类对象

## 菜单类对象

基本按钮对象Button类，所有其他按钮类都将继承此类。

```java
package com.chanshuyi.pojo.menu;

/**
 * 按钮的基类
 *
 * @author chenyr
 * @date 2015-09-29
 */
public class Button {
    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

普通按钮类CommonButton

```java
package com.chanshuyi.pojo.menu;

/**
 * 普通按钮（子按钮）
 * 没有子菜单的菜单项，有可能是二级菜单项，也有可能是不含二级菜单的一级菜单。
 *
 * @author chenyr
 * @date 2015-09-29
 */
public class CommonButton extends Button {

    private String type;

    private String key;

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public String getKey() {
        return key;
    }

    public void setKey(String key) {
        this.key = key;
    }
}
```

复杂按钮类ComplexButton

```java
package com.chanshuyi.pojo.menu;

/**
 * 复杂按钮（父按钮）
 * 包含有二级菜单项的一级菜单
 *
 * @author chenyr
 * @date 2015-09-29
 */
public class ComplexButton extends Button {
    private Button[] sub_button;

    public Button[] getSub_button() {
        return sub_button;
    }

    public void setSub_button(Button[] sub_button) {
        this.sub_button = sub_button;
    }
}
```

ViewButton类

```java
package com.chanshuyi.pojo.menu;

/**
 * view类型的菜单
 *
 * @author chenyr
 * @date 2015.09.30
 */
public class ViewButton extends Button {
    private String type;
    private String url;

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }
}
```

最后，我们根据微信对于一级和二级菜单的要求创建一个Menu类，代表整个菜单。

```java
package com.chanshuyi.pojo.menu;

/**
 * 菜单
 *
 * @author chenyr
 * @date 2015-09-29
 */
public class Menu {
    private Button[] button;

    public Button[] getButton() {
        return button;
    }

    public void setButton(Button[] button) {
        this.button = button;
    }
}
```

## MenuManger 类

下面创建一个MenuManger类，这个类将完成向微信服务器请求创建自定义菜单额所有工作。

```java
package com.chanshuyi.wechat.main;

import com.chanshuyi.pojo.AccessToken;
import com.chanshuyi.pojo.menu.*;
import com.chanshuyi.util.WeixinUtil;
import com.chanshuyi.wechat.thread.TokenThread;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * 菜单管理器类
 *
 * @author chenyr
 * @date 2015.10.04
 */
public class MenuManager {
    private static Logger log = LoggerFactory.getLogger(MenuManager.class);

    public static void createMenu(){
        // 调用接口获取access_token
        AccessToken at = WeixinUtil.getAccessToken(TokenThread.appid, TokenThread.appsecret);

        if (null != at) {
            // 调用接口创建菜单
            int result = WeixinUtil.createMenu(getMenu(), at.getToken());

            // 判断菜单创建结果
            if (0 == result)
                log.info("Menu create SUCCESS!");
            else
                log.info("Menu create FAIL, ERROR CODE:" + result);
        }
    }

    /**
     * 组装菜单数据
     *
     * @return
     */
    private static Menu getMenu() {
        CommonButton btn11 = new CommonButton();
        btn11.setName("天气预报");
        btn11.setType("click");
        btn11.setKey("11");

        CommonButton btn12 = new CommonButton();
        btn12.setName("公交查询");
        btn12.setType("click");
        btn12.setKey("12");

        CommonButton btn13 = new CommonButton();
        btn13.setName("周边搜索");
        btn13.setType("click");
        btn13.setKey("13");

        CommonButton btn14 = new CommonButton();
        btn14.setName("历史上的今天");
        btn14.setType("click");
        btn14.setKey("14");

        CommonButton btn21 = new CommonButton();
        btn21.setName("歌曲点播");
        btn21.setType("click");
        btn21.setKey("21");

        CommonButton btn22 = new CommonButton();
        btn22.setName("经典游戏");
        btn22.setType("click");
        btn22.setKey("22");

        CommonButton btn23 = new CommonButton();
        btn23.setName("美女电台");
        btn23.setType("click");
        btn23.setKey("23");

        CommonButton btn24 = new CommonButton();
        btn24.setName("人脸识别");
        btn24.setType("click");
        btn24.setKey("24");

        CommonButton btn25 = new CommonButton();
        btn25.setName("聊天唠嗑");
        btn25.setType("click");
        btn25.setKey("25");

        CommonButton btn31 = new CommonButton();
        btn31.setName("Q友圈");
        btn31.setType("click");
        btn31.setKey("31");

        CommonButton btn33 = new CommonButton();
        btn33.setName("幽默笑话");
        btn33.setType("click");
        btn33.setKey("33");

        CommonButton btn34 = new CommonButton();
        btn34.setName("无反馈菜单");
        btn34.setType("click");
        btn34.setKey("34");

        ViewButton btn32 = new ViewButton();
        btn32.setName("使用帮助");
        btn32.setType("view");
        btn32.setUrl("http://www.cnblogs.com/chanshuyi");

        ComplexButton mainBtn1 = new ComplexButton();
        mainBtn1.setName("生活助手");
        mainBtn1.setSub_button(new Button[] { btn11, btn12, btn13, btn14 });

        ComplexButton mainBtn2 = new ComplexButton();
        mainBtn2.setName("休闲驿站");
        mainBtn2.setSub_button(new Button[] { btn21, btn22, btn23, btn24, btn25 });

        ComplexButton mainBtn3 = new ComplexButton();
        mainBtn3.setName("更多");
        mainBtn3.setSub_button(new Button[] { btn31, btn33, btn34, btn32 });

        /**
         * 这是公众号xiaoqrobot目前的菜单结构，每个一级菜单都有二级菜单项<br>
         *
         * 在某个一级菜单下没有二级菜单的情况，menu该如何定义呢？<br>
         * 比如，第三个一级菜单项不是“更多体验”，而直接是“幽默笑话”，那么menu应该这样定义：<br>
         * menu.setButton(new Button[] { mainBtn1, mainBtn2, btn33 });
         */
        Menu menu = new Menu();
        menu.setButton(new Button[] { mainBtn1, mainBtn2, mainBtn3 });

        return menu;
    }
}
```

`int result = WeixinUtil.createMenu(getMenu(), at.getToken());` 表示向微信服务器发送HTTPS请求创建自定义菜单，代码如下：

```java
/**
     * 创建菜单
     *
     * @param menu 菜单实例
     * @param accessToken 有效的access_token
     * @return 0表示成功，其他值表示失败
     */
    public static int createMenu(Menu menu, String accessToken) {
        log.info("Create menu....");
        int result = 0;

        // 拼装创建菜单的url
        String url = menu_create_url.replace("ACCESS_TOKEN", accessToken);
        // 将菜单对象转换成json字符串
        String jsonMenu = JSONObject.fromObject(menu).toString();
        // 调用接口创建菜单
        JSONObject jsonObject = httpRequest(url, "POST", jsonMenu);

        if (null != jsonObject) {
            if (0 != jsonObject.getInt("errcode")) {
                result = jsonObject.getInt("errcode");
                log.error("创建菜单失败 errcode:{} errmsg:{}", jsonObject.getInt("errcode"), jsonObject.getString("errmsg"));
            }
        }

        log.info("/Create menu....");
        return result;
    }
```

最后在InitServlet的init方法里增加`new MenuManager().createMenu()`代码以创建自定义菜单即可。修改后的init方法如下：

```java
    public void init() throws ServletException {
        //将web.xml中的appid,appsecret赋值到TokenThread中
        log.info("**************************************************");
        log.info("**** Start init ACCESS_TOKEN Servlet. ");
        TokenThread.appid = getInitParameter("appid");
        TokenThread.appsecret = getInitParameter("appsecret");

        log.info("**** Weixin api appid:{}", TokenThread.appid);
        log.info("**** Weixin api appsecret:{}", TokenThread.appsecret);

        //向微信服务器请求创建菜单
        log.info("**** Start to create self-define Menu.");
        new MenuManager().createMenu();

        // 未配置appid、appsecret时给出提示
        if ("".equals(TokenThread.appid) || "".equals(TokenThread.appsecret)) {
            log.error("appid and appsecret configuration error, please check carefully.");
        } else {
            // 启动定时获取access_token的线程
            new Thread(new TokenThread()).start();
        }
    }
```





