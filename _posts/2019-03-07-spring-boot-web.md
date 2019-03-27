---
layout: post
title: 'Spring Boot Web 开发注解篇'
date: 2019/3/7 10:14:55 
author: 吴文杰
tags: jekyll
---


# Spring Boot Web 开发注解篇
摘要: 原创出处:www.bysocket.com 泥瓦匠BYSocket 希望转载
本文提纲

**一、spring-boot-starter-web 依赖概述**
​	在 Spring Boot 快速入门中，只要在 pom.xml 加入了 spring-boot-starter-web 依赖，即可快速开发 web 应用。可见，Spring Boot 极大地简化了 Spring 应用从搭建到开发的过程，做到了「开箱即用」的方式。Spring Boot 已经提供很多「开箱即用」的依赖，如上面开发 web 应用使用的 spring-boot-starter-web ，都是以 spring-boot-starter-xx 进行命名的。 
​	Spring Boot 「开箱即用」 的设计，对开发者非常便利。简单来说，只要往 Spring Boot 项目加入相应的 spring-boot-starter-xx 依赖，就可以使用对应依赖的功能，比如加入 spring-boot-starter-data-jpa 依赖，就可以使用数据持久层框架 Spring Data JPA 操作数据源。相比 Spring 以前需要大量的XML配置以及复杂的依赖管理，极大的减少了开发工作量和学习成本。 
​	当开发一个特定类型的应用程序时，特定的 Starter 提供所需的依赖关系，并且将对应的 Bean 注册到 Spring 容器中。spring-boot-starter-web 依赖就是提供开发 Web 应用的。  
1.1 spring-boot-starter-web 职责
​	spring-boot-starter-web 是一个用于构建 Web 的 Starter ，包括构建 RESTful 服务应用、Spring MVC 应用等。并且不需要额外配置容器，默认使用 Tomcat 作为嵌入式容器。
1.2 spring-boot-starter-web 依赖关系
spring-boot-starter-web 这么强大，它的组成如下表：
spring-boot-starter  核心包，包括了自动化配置支持、日志、YAML 文件解析的支持等。
spring-boot-starter-json 读写 JSON 包
spring-boot-starter-tomcat Tomcat 嵌入式 Servlet 容器包
hibernate-validator Hibernate 框架提供的验证包
spring-web Spring 框架的 Web 包
spring-webmvc Spring 框架的 Web MVC 包

![img](https://www.bysocket.com/wp-content/uploads/2017/08/WX20170727-181441.png)        

spring-boot-starter-web 包含了 Tomcat 和 Spring MVC ，那启动流程是这样的。 标识 @SpringBootApplication 的应用，初始化经过 spring-boot-starter  核心包中的自动化配置，构建了 Spring 容器，并通过 Tomcat 启动 Web 应用。很多 Starters 只支持 Spring MVC，一般会将 spring-boot-starter-web 依赖加入到应用的 Classpath。
另外，spring-boot-starter-web 默认使用 Tomcat 作为嵌入式 Servlet 容器，在 pom.xml 配置 spring-boot-starter-jetty 和 spring-boot-starter-undertow 就可以替换默认容器。
**二、Spring MVC on Spring Boot**
Spring MVC 是 Spring Web 重要的模块。内容包括 MVC 模式的实现和 RESTful 服务的支持。
2.1 Spring MVC 体系温故知新
spring-webmvc 模块里面包：

- org.springframework.web.servlet 提供与应用程序上下文基础结构集成的 Servlet，以及 Spring web MVC 框架的核心接口和类。
- org.springframework.web.servlet.mvc Spring 附带的 Servlet MVC 框架的标准控制器实现。
- org.springframework.web.servlet.mvc.annotation 用于基于注解的 Servlet MVC 控制器的支持包。
- org.springframework.web.servlet.mvc.condition 用于根据条件匹配传入请求的公共 MVC 逻辑。
- org.springframework.web.servlet.mvc.method 用于处理程序方法处理的基于 Servlet 的基础结构，基于在 org.springframework.web.method 包上。
- org.springframework.web.servlet.view 提供标准的 View 和 ViewResolver 实现，包括自定义实现的抽象基类。
- org.springframework.web.servlet.view.freemarker 支持将 FreeMarker 集成为 Spring Web 视图技术的类。
- org.springframework.web.servlet.view.json 支持提供基于 JSON 序列化的 View 实现的类。
   上面列出来核心的包。org.springframework.web.servlet.view 包中， View 视图实现有常见的：JSON 、FreeMarker 等。org.springframework.web.servlet.mvc 包中，Controller 控制层实现包括了注解、程序方法处理等封装。自然，看源码先从 org.springframework.web.servlet 包看其核心的接口和类。

**2.2 重要的类**
DispatcherServlet 类：调度 HTTP 请求控制器（或者处理器 Handler）。
View 视图层 ModelAndView 类：模型和视图的持有者。
View 接口：MVC WEB 交互。该接口的实现负责呈现视图或者暴露模型。
Controller 控制层 HandlerMapping 接口： 请求从 DispacherServlet 过来，该接口定义请求和处理程序对象之间的映射。
HandlerInterceptor 接口：处理程序的执行链接口。
Spring MVC 框架模型

![img](http://www.bysocket.com/wp-content/uploads/2017/08/WechatIMG485.jpeg)

**2.3 Spring Boot MVC**
以前 Spring MVC 开发模式是这样的:

1. 在 web.xml 配置 DispatcherServlet，用于截获并处理所有请求

2. 在 Spring MVC 配置文件中，声明预定义的控制器和视图解析器等

3. 编写预定义的处理请求控制器

4. 编写预定义的视图对象，比如 JSP、Freemarker 等
    在 Spring Boot MVC 中，Web 自动化配置会帮你减少上面的两个步骤。默认使用的视图是 ThymeLeaf，在下面小节会具体讲

5. 编写预定义的处理请求控制器

6. 编写默认 ThymeLeaf 视图对象
    例如下面会展示用户列表案例：
    第一步：处理用户请求控制器
    UserController.java

  ```html
  /**
   * 用户控制层
   *
   * Created by bysocket on 24/07/2017.
   */
  @Controller
  @RequestMapping(value = "/users")     // 通过这里配置使下面的映射都在 /users
  public class UserController {
   
      @Autowired
      UserService userService;          // 用户服务层
   
      /**
       *  获取用户列表
       *    处理 "/users" 的GET请求，用来获取用户列表
       *    通过 @RequestParam 传递参数，进一步实现条件查询或者分页查询
       */
      @RequestMapping(method = RequestMethod.GET)
      public String getUserList(ModelMap map) {
          map.addAttribute("userList", userService.findAll());
          return "userList";
      }
  }

7.  @Controller 注解在 UserController 类上，标识其为一个可接收 HTTP 请求的控制器
  @RequestMapping(value = "/users") 注解 ，标识 UserController 类下所有接收的请求路由都是 /users 开头的。注意：类上的 @RequestMapping 注解是不必需的
  @RequestMapping(method = RequestMethod.GET) 注解，标识该 getUserList(ModelMap map) 方法会接收并处理 /users 请求，且请求方法是 GET
  getUserList(ModelMap map) 方法返回的字符串 userList ，代表着是视图，会有视图解析器解析成为一个具体的视图对象，然后经过视图渲染展示到浏览器
  第二步：用户列表 ThymeLeaf 视图对象

  ```html
  <!DOCTYPE html>
  <html lang="zh-CN">
      <head>
          <script type="text/javascript" th:src="@{https://cdn.bootcss.com/jquery/3.2.1/jquery.min.js}"></script>
          <link th:href="@{https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css}" rel="stylesheet"/>
          <link th:href="@{/css/default.css}" rel="stylesheet"/>
          <link rel="icon" th:href="@{/images/favicon.ico}" type="image/x-icon"/>
          <meta charset="UTF-8"/>
          <title>用户列表</title>
      </head>
   
      <body>
   
          <div class="contentDiv">
   
              <h5> 《 Spring Boot 2.x 核心技术实战》第二章快速入门案例</h5>
   
              <table class="table table-hover table-condensed">
                  <legend>
                      <strong>用户列表</strong>
                  </legend>
                  <thead>
                      <tr>
                          <th>用户编号</th>
                          <th>名称</th>
                          <th>年龄</th>
                          <th>出生时间</th>
                          <th>管理</th>
                      </tr>
                  </thead>
                  <tbody>
                      <tr th:each="user : ${userList}">
                          <th scope="row" th:text="${user.id}"></th>
                          <td><a th:href="@{/users/update/{userId}(userId=${user.id})}" th:text="${user.name}"></a></td>
                          <td th:text="${user.age}"></td>
                          <td th:text="${user.birthday}"></td>
                          <td><a class="btn btn-danger" th:href="@{/users/delete/{userId}(userId=${user.id})}">删除</a></td>
                      </tr>
                  </tbody>
              </table>
   
              <div><a class="btn btn-primary" href="/users/create" role="button">创建用户</a></div>
          </div>
   
      </body>
  </html>
  ```

  一个 table 展示用户列表，引入了 jquery.min.js 和 bootstrap.min.css ，更好的展示页面效果。具体 ThymeLeaf 语法下面会讲到。

  代码共享在：<https://github.com/JeffLi1993/spring-boot-core-book-demo>

  2.3.1 控制器  
  什么是控制器？控制器就是控制请求接收和负责响应到视图的角色。
  @Controller 注解标识一个类作为控制器。DispatcherServlet 会扫描所有控制器类，并检测 @RequestMapping 注解配置的方法。Web 自动化配置已经处理完这一步骤。
  @RequestMapping 注解标识请求 URL 信息，可以映射到整个类或某个特定的方法上。该注解可以表明请求需要的。

  使用 value 指定特定的 URL ，比如 @RequestMapping(value = "/users”) 和 @RequestMapping(value = "/users/create”) 等
  使用 method 指定 HTTP 请求方法，比如 RequestMethod.GET 等
  还有使用其他特定的参数条件，可以设置 consumes 指定请求时的请求头需要包含的 Content-Type 值、设置 produces 可确保响应的内容类型
  MVC on REST ful 场景

  在 HTTP over JSON （自然 JSON、XML或其他自定义的媒体类型内容等均可）场景，配合上前后端分离的开发模式，我们经常会用 @ResponseBody 或 @RestController 两种方式实现 RESTful HTTP API 。
  老方式：
  @ResponseBody 注解标识该方法的返回值。这样被标注的方法返回值，会直接写入 HTTP 响应体（而不会被视图解析器认为是一个视图对象）。
  新方式：
  @RestController 注解，和 @Controller 用法一致，整合了 @Controller 和 @ResponseBody 功能。这样不需要每个 @RequestMapping 方法上都加上 @ResponseBody 注解，这样代码更简明。
  使代码更简明，还有常用便捷注解 @GetMapping、@PostMapping 和 @PutMapping 等
  HTTP 协议相关知识回顾，可以看看我以前的博文《图解 HTTP 协议》http://www.bysocket.com/?p=282



  **2.3.2 数据绑定**
  数据绑定，简单的说就是 Spring MVC 从请求中获取请求入参，赋予给处理方法相应的入参。主要流程如下：

8. DataBinder 接受带有请求入参的 ServletRequest 对象

9. 调用 ConversionService 组件，进行数据类型转换、数据格式化等工作

10. 然后调用 Validator 组件，进行数据校验等工作

11. 绑定结果到 BindingResult 对象

12. 最后赋予给处理方法相应的入参
      @ModelAttribute 注解添加一个或多个属性（类对象）到 model 上。例如

    @RequestMapping(value = "/create", method = RequestMethod.POST)
    public String postUser(@ModelAttribute User user)
      @PathVariable 注解通过变量名匹配到 URI 模板中相对应的变量。例如

    @RequestMapping(value = "/update/{id}", method = RequestMethod.GET)
    public String getUser(@PathVariable("id") Long id, ModelMap map)
      @RequestParam 注解将请求参数绑定到方法参数。
      @RequestHeader 注解将请求头属性绑定到方法参数。
      2.3.3 视图和视图解析
      视图的职责就是渲染模型数据，将模型里面的数据展示给用户。
      请求到经过处理方法处理后，最终返回的是 ModeAndView 。可以从 Spring MVC 框架模型 看出，最终经过 ViewResolver 视频解析器得到视图对象 View。可能是我们常见的 JSP ，也可能是基于 ThymLeaf 、FreeMarker 或 Velocity 模板引擎视图，当然还有可能是 JSON 、XML 或者 PDF 等各种形式。
      业界流行的模板引擎有如下的 Starters 支持：
      spring-boot-starter-thymeleaf Thymeleaf 模板视图依赖，官方推荐
      spring-boot-starter-freemarker Freemarker 模板视图依赖
      spring-boot-starter-groovy-templates Groovy 模板视图依赖
      spring-boot-starter-mustache Mustache 模板视图依赖
      具体，spring-boot-starter-thymeleaf 使用案例在 GitHub ：https://github.com/JeffLi1993/spring-boot-core-book-demo 的 chapter-2-spring-boot-quick-start 工程。
      三、小结
      本文主要介绍了 Spring Boot 在 Web 开发中涉及到的 HTTP 协议，还有一些 Spring MVC 相关的知识。
      推荐：
      开源项目 springboot-learning-example https://github.com/JeffLi1993/springboot-learning-example
      开源项目 spring-boot-core-book-demo https://github.com/JeffLi1993/spring-boot-core-book-demo
      资料：

- 官方文档 http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle