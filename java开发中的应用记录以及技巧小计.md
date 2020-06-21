# java开发中的应用记录以及技巧

### 一、项目的搭建

#### 1、idea和eclipse创建项目的区别

在idea中没有工作空间的概念，创建的project就是相当于eclipse中的工作空间。可以在一个project创建多个		模块，每个模块就相当于与eclipse工作空间中的一个个项目，当然，在idea中一个project也可作为一个单独		的项目存在。

#### 2、maven项目

- 下载安装本地的maven仓库

- 配置maven的镜像

  - 公司指定的公司内部镜像
  - 华为的镜像
  - 阿里的镜像（springboot不支持，无法下载springboot的依赖）

- 配置jar包的存放位置

- 普通的maven可以进行父子工程的版本管理

  - 子工程作为父工程的model模块

#### 3、ssm项目

- 添加spring、springMVC、springweb、mybatis、等相关依赖

- 添加tomcat的插件

- 修改web.xml的版本

  - **配置DispatcherServlet**，需要两个初始化的参数:springmvc的配置文件路径和contextConfigLocation加载其他中间件的配置文件,contextConfigLocation是一个监听器listener

    需要和context-param配合使用(==如果web项目不使用servlet只是提供，就只使用监听器加载中间件配置文件即可==)

    ```xml
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/springmvc.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
    
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:spring/spring-*.xml</param-value>
    </context-param>
    
    <listener>
        <listener-class>
            org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    ```

  - **配置CharacterEncodingFilter处理post请求的乱码问题**

    ```xml
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
          <param-name>encoding</param-name>
          <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
          <param-name>forceEncoding</param-name>
          <param-value>true</param-value>
        </init-param>
      </filter>
      <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
    ```

  - **配置mybatis的主配置文件**

    - 配置数据源

      ```xml
      <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
      		destroy-method="close">
      		<property name="url" value="${jdbc.url}" />
      		<property name="username" value="${jdbc.username}" />
      		<property name="password" value="${jdbc.password}" />
      		<property name="driverClassName" value="${jdbc.driver}" />
      		<property name="maxActive" value="10" />
      		<property name="minIdle" value="5" />
      </bean>
      ```

      - 配置加载属性资源文件的路径

        ```xml
        <context:property-placeholder location="classpath*:properties/*.properties" />
        ```

      - 配置SqlSessionFactoryBean加载数据源和SQL映射文件的路径

        ```xml
        <bean id="sqlSessionFactory" 		class="org.mybatis.spring.SqlSessionFactoryBean">
        		<!-- 数据库连接池 -->
        		<property name="dataSource" ref="dataSource" />
        		<!-- 加载mybatis的全局配置文件 -->
        		<property name="configLocation" value="classpath:mybatis/SqlMapConfig.xml" />
        </bean>
        ```

      - 配置MapperScannerConfigurer扫描接口的路径

        ```xml
        <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        	<property name="basePackage" value="com.zl.mapper" />
        </bean>
        ```

#### 4、springboot的项目

- 选择spring initializr

- 选择自己使用到的中间件或者在pom.xml文件中添加对象的场景启动器

- application.properties/application.yml中配置项目使用到的参数，通过xxx.xxx.xx的形式设置参数名，参数名与参数之间要有一个空格，参数名层级之间缩进两个字符

- 响应的注解支持在默认的启动类上加上响应的注解

- static /public /rescouces/mate-inf作为静态资源的存放路径

- 在application.yml中配置数据源信息和mybatis的主配置信息

  ```yaml
  spring:
    datasource:
      name: datasource
      type: com.alibaba.druid.pool.DruidDataSource
      druid:
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost:3306/users?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true&serverTimezone=GMT%2B8
        username: root
        password: 123
        filters: stat
        initial-size: 1
        min-idle: 1
        max-active: 20
        max-wait: 60000
        time-between-eviction-runs-millis: 60000
        min-evictable-idle-time-millis: 300000
        validation-query: SELECT 'x'
        test-while-idle: true
        test-on-borrow: false
        test-on-return: false
        pool-prepared-statements: false
        max-pool-prepared-statement-per-connection-size: 20
        
  mybatis:
    #映射文件的存放位置
    mapper-locations: classpath:mapper/*.xml
    type-aliases-package: com.zl.springbootdemo1.pojo
  ```

- 分页插件的配置

  ```yaml
  pagehelper:
    #使用的数据库的类型
    helperDialect: mysql
    #参数的合理优化
    reasonable: true
    supportMethodsArguments: true
    params: count=countSql
  ```

- 在主配置文件上添加@MapperScan注解

#### 5、springcloud项目

- springcloud项目依赖于springboot项目

- 配置注册中心eureka server

  ```yaml
  eureka:
    instance:
      hostname: localhost
    client:
    	#设置自己本身只是作为注册中心，不对外发布服务
      register-with-eureka: false
      fetch-registry: false
      service-url:
        defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
  
  spring:
    application:
      name: register
      
  ```

  - 在启动类上添加@EnableEurekaServer

- eureka client组件发布服务到注册中心

  ```yaml
  eureka:
    client:
      serviceUrl:
        defaultZone: http://localhost:8761/eureka/
        
  spring:
    application:
      name: client
      
      #spring.application.name 这个一定要设置，这是用来调用服务时候的标识的
  ```

  - 在启动类上添加@EnableEurekaClient注解

- 服务消费者（本身也是服务）

  - **ribbon+restTemplate**（使用模板请求服务）
    - restTemplate通过@LoadBalanced开启负载均衡功能
    - 在启动类@EnableDiscoveryClien注解，表示可以去发现其他的服务
    - restTemplate.getForObject(“http://服务名/restful请求,请求的返回值类型)
  - feign（自己添加服务的相同的接口）
    - 在启动类上添加@EnableFeignClients注解
    - 自己编写接口，添加服务的一摸一样的方法，在接口上添加@ FeignClient（“‘服务的名称’）
    - 通过接口调用方法来实现方法的调用
  
- 配置config server

  ```yaml
  cloud:
      config:
        server:
          git:
            uri: 
            username: 
            password: 
  ```

  - 在启动类上添加@EnableConfigServer注解

- 熔断hystrix

  ```xml
  feign.hystrix.enabled: true
  ```

  - 在本地自实现服务的方法，通过@FeignClient注解来知识本地的处理

- 网关zuul

  ```yaml
  zuul:
    routes:
      api-a:
        path: /api-a/**
        serviceId: service-ribbon
      api-b:
        path: /api-b/**
        serviceId: service-feign
  ```

  - 在启动类上添加@EnableZuulProxy注解

  - 继承zuulFilter抽象类实现过滤认证功能

    ```java
    package com.zl.servicezuul.service;
    
    import com.netflix.zuul.ZuulFilter;
    import com.netflix.zuul.context.RequestContext;
    import com.netflix.zuul.exception.ZuulException;
    import org.springframework.stereotype.Component;
    
    import javax.servlet.http.HttpServletRequest;
    import java.io.IOException;
    
    /**
     * @author: 丢了风筝的线
     * @Date: 2020/6/21-06-21-16:27
     * @Description:实现路由的过滤规则，做权限校验等
     */
    @Component
    public class myZuul extends ZuulFilter {
    
        /**
         * 过滤器的类型
         * @return
         */
        @Override
        public String filterType() {
            return "pre";
        }
    
        /**
         * 过滤器的级别，0123，数字越小级别越高
         * @return
         */
        @Override
        public int filterOrder() {
            return 0;
        }
    
        /**
         * 是否开启过滤功能
         * @return
         */
        @Override
        public boolean shouldFilter() {
            return true;
        }
    
        /**
         * 过滤的业务逻辑
         * @return
         * @throws ZuulException
         */
        @Override
        public Object run() throws ZuulException {
    
            //获取当前的请求
            RequestContext currentContext = RequestContext.getCurrentContext();
            HttpServletRequest request = currentContext.getRequest();
    
            //获取请求的用户信息
            String token = request.getParameter("token");
    
            if(token==null ||! "zl".equals(token)){
    
                //关闭路由的响应
                currentContext.setSendZuulResponse(false);
                //设置请求响应的状态码
                currentContext.setResponseStatusCode(401);
    
                try {
                    currentContext.getResponse().setContentType("text/html;charset=utf8");
                    currentContext.getResponse().getWriter().write("身份信息不合法");
                } catch (IOException e) {
                    e.printStackTrace();
                }
    
                return null;
            }
    
            //身份验证成功
    
            return null;
        }
    }
    
    ```

    

### **二、框架（中间件）**

### 三、技术选型