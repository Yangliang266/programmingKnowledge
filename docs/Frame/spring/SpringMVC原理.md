## springmvc 框架结构



<img src="https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/frame/spring/springmvc/springmvcspringmvc01.png" alt="springmvc01" style="zoom: 50%;" />

**架构流程**

1. 用户发送请求至前端控制器DispatcherServlet

2. DispatcherServlet收到请求调用HandlerMapping处理器映射器。

3. 处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。

4. DispatcherServlet通过HandlerAdapter处理器适配器调用处理器

5. 执行处理器(Controller，也叫后端控制器)。

6. Controller执行完成返回ModelAndView

7. HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet

8. DispatcherServlet将ModelAndView传给ViewReslover视图解析器

9. iewReslover解析后返回具体View

10. DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）。

11. DispatcherServlet响应用户

**组件说明**

以下组件通常使用框架提供实现：

◆ DispatcherServlet：前端控制器

用户请求到达前端控制器，它就相当于mvc模式中的c， dispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求， dispatcherServlet的存在降低了组件之间的耦合性。

◆ HandlerMapping：处理器映射器

HandlerMapping负责根据用户请求url找到Handler即处理器， springmvc提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

◆ Handler：处理器

Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。

由于Handler涉及到具体的用户业务请求，所以一般情况需要程序员根据业务需求开发Handler。

◆ HandlAdapter：处理器适配器

通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。

◆ ViewResolver：视图解析器

View Resolver负责将处理结果生成View视图， View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。

◆ View：视图

springmvc框架提供了很多的View视图类型的支持，包括： jstlView、freemarkerView、 pdfView等。我们最常用的视图就是jsp。

一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，

需要由程序员根据业务需求开发具体的页面。



> 说明：在 springmvc 的各个组件中，处理器映射器、处理器适配器、视图解析器称为 springmvc 的三大组件。需要用户开发的组件有 handler、 view