# Spring
目前对Spring的了解：
Spring实现了IoC控制反转，或者说依赖注入DI，是一个问题的两个说法，以前传统的设计思想是A控制C的话，需要在A中创建C类再进行操作，IoC的思想就是通过IoC/DI容器创建C类的对象，再将A类所需的数据，注入到A类中，实现了控制反转，也叫依赖注入。  降低了耦合。

依赖注入：程序依赖容器注入程序所需的外部资源  
控制反转：容器控制程序，将程序所需的外部资源注入程序

AOP切面编程的话，即比如说是将每模块的日志、权限、业务抽象出来，按功能统一进行划分，就是切面。减少了代码的冗余，提高了代码复用。

Spring的IoC建立在反射机制之上 。

## 一、概述

### Spring的优点

1.降低了组件之间的耦合性 ，实现了软件各层之间的**解耦**   
2.可以使用容易提供的众多服务，如**事务管理**，**消息服务**等  
3.容器提供单例模式支持  
4.容器提供了**AOP**技术，利用它很容易实现如权限拦截，运行期监控等功能  
5.容器提供了众多的辅助类，能加快应用的开发  
6.spring对于主流的应用框架提供了集成支持，如hibernate，JPA，Struts等  
7.spring属于**低侵入式**设计，代码的污染极低  
8.独立于各种应用服务器  
9.spring的DI机制降低了业务对象替换的复杂性  
10.Spring的高度开放性，并不强制应用完全依赖于Spring，开发者可以自由选择spring的部分或全部  

## 二、IOC

什么是DI机制？  
依赖注入（Dependecy Injection）是控制反转（Inversion of Control）的一种实现方式，具体的讲：当某个角色需要另外一个角色协助的时候，在传统的程序设计过程中，通常由调用者来创建被调用者的实例。但在spring中创建被调用者的工作不再由调用者来完成，因此称为控制反转。创建被调用者的工作由spring来完成，然后注入调用者。因此也称为依赖注入。 

spring以动态灵活的方式来管理对象。

**参数注入**

优点：使用set方法注入对象

**构造注入**

优点：在构造方法中注入所用对象。包括下标匹配、类型匹配（不建议）、name匹配(value/ref )

**接口注入**

在声明的时候注入对象（省略了set、get方法）

不管是走的哪个注入方式，都是先构建了对象的类，提供set入口或者有参构造函数，然后再xml中通过对应的方式生成bean



spring提供了@autowiring自动完成以上装配过程

## 三、AOP

### 3.1 概述

面向切面编程（AOP）完善spring的依赖注入（DI），面向切面编程在spring中主要表现为两个方面 

1. 面向切面编程提供声明式事务管理 
2. spring支持用户自定义的切面 

面向切面编程（aop）是对面向对象编程（oop）的补充， 
面向对象编程将程序分解成各个层次的对象，面向切面编程将程序运行过程分解成各个切面。 
AOP从程序运行角度考虑程序的结构，提取业务处理过程的切面，oop是静态的抽象，aop是动态的抽象， 
是对应用执行过程中的步骤进行抽象，从而获得步骤之间的逻辑划分。

aop框架具有的两个特征：  
1. 各个步骤之间的良好隔离性 
2. 源代码无关性 

### 3.2 代理模式

* 静态代理
* 动态代理

#### 3.2.1 静态代理

#### 3.2.2 动态代理

依据反射实现

### 3.3 XXX

Spring的**事务管理机制**实现的原理，就是通过这样一个动态代理对所有需要事务管理的Bean进行加载，并根据配置在invoke方法中对当前调用的 方法名进行判定，并在method.invoke方法前后为其加上合适的事务管理代码，这样就实现了Spring式的事务管理。Spring中的AOP实 现更为复杂和灵活，不过基本原理是一致的。



## 四、事务

ACID

也是基于mysql

[事务]: https://www.cnblogs.com/mseddl/p/11577846.html

### 事务配置

1.编程式事务：（侵入式）使用TransactionTemplate和直接使用PlatformTransactionManager（配置bean时）。通过显式的写execute和status.setRollbackOnly()进行执行或者回滚。

2.声明式事务：通过AOP（或注解@Transactional）在方法执行前后加入事务的开始和提交回滚。作用到方法级别。

### 传播机制

- PROPAGATION_REQUIRED propagation_required
	Spring**默认**的传播机制，能满足绝大部分业务需求，如果外层有事务，则当前事务加入到外层事务，一块提交，一块回滚。如果外层没有事务，新建一个事务执行
- PROPAGATION_REQUES_NEW
	该事务传播机制是每次都会新开启一个事务，同时把外层事务挂起，当当前事务执行完毕，恢复上层事务的执行。如果外层没有事务，执行当前新开启的事务即可
- PROPAGATION_SUPPORT
	如果外层有事务，则加入外层事务，如果外层没有事务，则直接使用非事务方式执行。完全依赖外层的事务
- PROPAGATION_NOT_SUPPORT
	该传播机制不支持事务，如果外层存在事务则挂起，执行完当前代码，则恢复外层事务，无论是否异常都不会回滚当前的代码
- PROPAGATION_NEVER never
	该传播机制不支持外层事务，即如果外层有事务就抛出异常
- PROPAGATION_MANDATORY mandatory
	与NEVER相反，如果外层没有事务，则抛出异常
- PROPAGATION_NESTED nested
	该传播机制的特点是可以保存状态保存点，当前事务回滚到某一个点，从而避免所有的嵌套事务都回滚，即各自回滚各自的，如果子事务没有把异常吃掉，基本还是会引起全部回滚的。



## 五、Bean

Spring Bean是被实例的，组装的及被Spring容器管理的Java对象。

Spring容器会自动完成@bean对象的实例化。

创建应用对象之间的协作关系的行为称为：**装配(wiring)**，这就是依赖注入的本质。

### Bean作用域

singleton：单例

prototype：原型

request：

session：

application：

websocket：

### BeanFactory

### Bean生命周期

实例化

属性赋值

初始化（）

销毁（容器关闭）



Spring给Bean的生命周期提供的扩展功能非常多

## 六、Spring常用注解

SpringBoot的**启动**类注解：@SpringBootApplication

第一部分

@Controller 标注Controller组件

@Service 标注业务层组件

第二部分

@ResponseBody 将java对象转为json（xml）格式的数据，写入Response对象的body数据区。

@RequestMapping 处理请求地址映射的注解，可以作用于类和方法上

@Autowired 它可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。 通过 @Autowired的使用来消除 set ，get方法

@RequestParam 控制层获取前端参数

@Crossorigin 跨域

## 七、Spring、SpringBoot和Spring Cloud

Spring是整个生态，主要依存于SSH框架（Struts+Spring+Hibernate数据持久化）这个MVC框架
Spring framework是
SpringMVC是基于Spring的一个 MVC 框架
SpringBoot基于Spring的一套快速开发整合包（简言之就是让搭建SpringMVC的过程更简单了，配置简单，也提供了很多常用工具）
Spring Cloud是基于Spring的一整套解决方案——服务注册与发现，服务消费，服务保护与熔断，网关，分布式调用追踪，分布式配置管理等（对标Dubbo，不过dubbo是rpc框架，spring cloud基于http的restful框架）

## SpringMVC

### controller和service

controller层：可以看做是view和Model之间进行沟通的桥梁，可以**分发用户的请求**，并**选择恰当的视图**以用于显示，同时可以解释用户的输入并映射为模型层可以执行的操作。

控制器Controller 负责处理由DispatcherServlet 分发的请求，它把用户请求的数据经过业务处理层处理之后封装成一个Model ，然后再把该Model 返回给对应的View 进行展示。

service层：在接触Spring框架时会了解到面向接口编程，表示层调用控制层，控制层调用**业务**层，业务层调用数据访问层

初期也许都是new对象去调用下一层，比如你在业务层new一个DAO类的对象，调用DAO类方法访问数据库，这样写是不对的，因为在业务层中是不应该含有具体对象，最多只能有引用，如果有具体对象存在，就耦合了。当那个对象不存在，我还要修改业务的代码，这不符合逻辑。