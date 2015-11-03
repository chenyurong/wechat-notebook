# 自定义菜单开发 - 定义菜单类POJO对象

微信允许公众号通过开发接口进行自定义菜单的开发，这样我们就可以让自定义菜单有更加丰富的功能。

我们通过开发接口开发自定义菜单的时候，只需要向`菜单创建接口`发送一次请求即可，而不需要每次启动项目都请求一次。

微信公众号自定义菜单中的菜单类型可以分为3种不同的类型：

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





