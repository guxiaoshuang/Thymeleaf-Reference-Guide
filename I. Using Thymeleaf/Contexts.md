### 上下文

为了处理我们的模版，我们将会创建一个实现了`IGTVGController`接口的`HomeController`类。如下：
```java
public class HomeController implements IGTVGController {

    public void process(
            final HttpServletRequest request, final HttpServletResponse response,
            final ServletContext servletContext, final ITemplateEngine templateEngine)
            throws Exception {
        
        WebContext ctx = 
                new WebContext(request, response, servletContext, request.getLocale());
        
        templateEngine.process("home", ctx, response.getWriter());
        
    }

}
```
我们第一眼看到的就是上下文的创建。一段Thymeleaf的上下文就是一个实现了`org.thymeleaf.context.IContext`接口的对象。上下文应当包含，在变量的映射关系中执行模版引擎需要的所有数据，同时指明了外部化的信息必须用到的地区。
```java
public interface IContext {

    public Locale getLocale();
    public boolean containsVariable(final String name);
    public Set<String> getVariableNames();
    public Object getVariable(final String name);
    
}
```
这个接口有一个专门的扩展：`org.thymeleaf.context.IWebContext`，用在基于ServletAPI的网络应用里（比如SpringMVC）。
```java
public interface IWebContext extends IContext {
    
    public HttpServletRequest getRequest();
    public HttpServletResponse getResponse();
    public HttpSession getSession();
    public ServletContext getServletContext();
    
}
```
Thymeleaf核心库提供了这些接口里每一个的实现：

- `org.thymeleaf.context.Context`实现了`IContext`接口
- `org.thymeleaf.context.WebContext`实现了`IWebContext`接口

如同你在controller的代码里看到的那样，我们使用了`WebContext`。In fact we have to, because the use of a `ServletContextTemplateResolver` requires that we use a context implementing `IWebContext`.
```java
WebContext ctx = new WebContext(request, response, servletContext, request.getLocale());
```
Only three out of those four constructor arguments are required because the default locale for the system will be used if none is specified (although you should never let this happen in real applications).

There are some specialized expressions that we will be able to use to obtain the request parameters and the request, session and application attributes from the `WebContext` in our templates. For example:

- `${x}` will return a variable `x` stored into the Thymeleaf context or as a request attribute.
- `${param.x}` will return a request parameter called `x` (which might be multivalued).
- `${session.x}` will return a session attribute called `x`.
- `${application.x}` will return a servlet context attribute called `x`.