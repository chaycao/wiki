## 1. IOC 和 DI

IOC，控制反转，将原本在程序中手动创建对象的控制权，交由Spring框架管理。

DI，依赖注入，在Spring框架负责创建Bean对象时，动态的将依赖对象注入到Bean组件 。

Spring中依赖注入有三种方式：

1. 构造器注入
2. setter方法注入
3. 注解注入



## 2. BeanFactory 接口和 ApplicationContext 接口

ApplicationContext 接口继承BeanFactory接口。

Spring核心工厂是BeanFactory，BeanFactory采取延迟加载，第一次getBean时才会初始化Bean。 ApplicationContext是会在加载配置文件时初始化Bean。

ApplicationContext是对BeanFactory扩展，它可以进行国际化处理、事件传递和bean自动装配以及各种不同应用层的Context实现。

开发中基本都在使用ApplicationContext, web项目使用WebApplicationContext ，很少用到BeanFactory。



## 3. bean实例化的三种方式

XML 配置中实例化 bean 的三种方式：

1. **普通构造方法**

   ```xml
   <bean class="org.sang.User" id="user"/>
   ```

2. **静态工厂**

   ```xml
   <bean id="user2" class="org.sang.User2Factory" factory-method="getInstance"/>
   ```

3. **实例工厂**

   ```xml
   <bean class="org.sang.User3Factory" id="user3Factory"/>
   <bean id="user3" factory-bean="user3Factory" factory-method="getUser3"/>
   ```

   

## 4. bean的生命周期和作用域

对于 prototype 作用域的 Bean，Spring容器仅负责创建，当容器创建 Bean 实例后，Bean 实例由客户端代码管理，容器不再跟踪其生命周期。即 **Spring 无法管理 prototype 作用域的 Bean**。

对于 singleton 作用域的 Bean，客户端代码不能控制 Bean 的销毁，Spring 容器负责跟踪 Bean 实例的产生、销毁。容器知道 Bean 何时实例化结束、何时销毁，**Spring 可以管理 Singleton 作用域的 Bean 实例化结束之后和销毁之前的行为**。



Spring管理Bean的生命周期由两组回调方法组成：

1. 初始化之后调用的回调方法
2. 销毁之前调用的回调方法

Spring 通过下面4种方法管理bean的生命周期事件：

1. 实现 InitializingBean 和 DisposableBean 接口
2. 配置文件中的 init-method，destory-method属性
3. @PostConstruct 和 @PreDestory 注解方式
4. 针对特殊行为的其他 Aware 接口



**作用域：**

1. singleton：默认，容器内只会一个 bean 的实例
2. prototype：为每一个请求提供一个实例
3. request：在一次HTTP请求中，一个bean定义对应一个实例；即每次HTTP请求将会有各自的bean实例， 它们依据某个bean定义创建而成。该作用 域仅在基于web的Spring ApplicationContext情形下有效。 
4. session：在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。
5. global session：在一个全局的HTTP Session中，一个bean定义对应一个实例。典型情况下，仅在使用portlet context的时候有效。该作用域仅在基于 web的Spring ApplicationContext情形下有效。 