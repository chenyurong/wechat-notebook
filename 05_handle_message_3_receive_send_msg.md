# 处理微信服务器的消息 - 各种消息的接收与响应

从微信公众平台接口消息指南中可以了解到，当用户向公众帐号发消息时，微信服务器会将消息通过POST方式提交给我们在接口配置信息中填写的URL，而我们就需要在URL所指向的请求处理类CoreServlet的doPost方法中接收消息、处理消息和响应消息。

修改后的 `CoreServlet.doPost()` 方法如下：

```java
/**
 * 处理微信服务器发来的消息
 */
public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    // 将请求、响应的编码均设置为UTF-8（防止中文乱码）
    request.setCharacterEncoding("UTF-8");
    response.setCharacterEncoding("UTF-8");

    // 调用核心业务类接收消息、处理消息
    String respMessage = CoreService.processRequest(request);

    // 响应消息
    PrintWriter out = response.getWriter();
    out.print(respMessage);
    out.close();
}
```

这里我们通过调用CoreService类的processRequest方法接收、处理消息的，这样做的目的是为了解耦，即业务相关的操作都不在Servlet里处理，而是完全交由业务核心类CoreService去做。


