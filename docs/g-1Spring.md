# Spring 核心技术

> **本笔记使用的Spring版本为6.0**

Spring 是为了解决在 javaWeb 开发过慢的问题，在 JavaWeb 中，我们已经具备了开发网站的能力，但使用 Java Web 进行开发有着诸多不便和繁琐之处，因此，我们需要一些框架技术，来简化和规范我们的Java开发。

[Spring](https://docs.spring.io/spring-framework/docs/6.0.10/reference/html/core.html#spring-core) 就是这样的一个框架，它就是为了简化开发而生，它是轻量级的 **IoC** 和 **AOP** 的容器框架，主要是针对 **Bean** 的生命周期进行管理的轻量级容器，并且它的生态已经发展得极为庞大。那么，首先一问，什么是**IoC**和**AOP**，什么又是**Bean**呢？

## IOC 容器技术

Spring框架最核心的其实它的 IoC 容器，这是我们开启Spring学习的第一站。

### IOC理论介绍

在我们之前的 Web 应用程序中，我们可以发现，整个程序都是依靠各个部分相互协作，完成一个操作，比如我们要查询某一个信息列表，我们首先需要使用 Servlet 进行请求和操作的响应数据处理，然后交给 Server 层处理，最后当 Server 发现要从数据库中取出数据时，在向对应的 Mapper 发起请求。

它们就像连起来得到齿轮一样，谁也无法离开彼此，

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231128131333.png#pic_left)

这样的有点非常明显，就是逻辑清晰，代码简洁，同时整个流程也能够让人快速了解，但这样存在一个很严重的问题，那就是不易于修改和更新，只要某一个类发生更新变动，那么相关联的类就也会受到影响，就像齿轮 D 消失后，会对齿轮 B 和 C 产生影响。比如说我们修改了某个类名。

~~~java
public class A{ 	// 此处 A -> ReA
    public void FunctionA();
}
public class B{
    public void functionB(){
        new A();	// 此处会发生错误
        ....
    }
}
~~~

可以看到，因为类 A 和类 B 之间的关联太强了，导致修改了 A 就会影响到 B，之前所有使用了 A 的类都需要修改，这会花费大量的时间。

或者说某个 Server 的实现类我们不想用了，想要更换成其他的实现类使用不同的逻辑，我们也必须挨个进行修正，当项目庞大的时候，这就是一笔难以接受的时间浪费。

因此，高耦合度带来的缺点是非常明显的，也是现代软件快速开发中一项很致命的问题。为了改善这种情况，我们只能将个模块进行解耦，让各个模块之间的依赖性不再那么强，也就是说，需要让程序来决定 接口 Server 的实现类而不是由我们人来决定，我们开发的所有实现类，都由程序来管理；所有对象之间的关系，都由程序动态决定，这样子IoC理论也就产生了。

IOC是 Inversion of Control 的缩写，翻译为：“控制反转”，把复杂系统分解成相互合作的对象，这些对象类通过封装以后，内部实现对外部是透明的，外部无需关注内部是如何实现，从而降低了解决问题的复杂度，而且可以灵活地被重用和扩展。

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231128133323.png)

我们可以将类和对象交给 IoC 容器进行管理，比如当我们需要一个接口的实现时，由 IoC 根据配置文件来自动匹配实现类，这样，我们就可以不用再关心我们要去使用哪一个实现类了，我们只需要关心给予 IoC 容器的是一个可以正常使用的实现类，具体是怎么实现，里面写了什么都无需关注，这样，我们就可以放心地让不同的人去写视图层的代码和业务层的代码。

项目 Bean 依赖示意图：

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231128143637.png)![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231128143551.png)

------

### Spring模块结构

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231128143824.png)

首先要看的就是 Core Container，也就是 Spring 的核心容器模块，只有了解了 Spring 的核心技术，我们才能真正认识这个框架为我们带来的便捷之处。

Spring 是一个非入侵式的框架，就像一个工具库一样，它可以很简单地加入到我们已有的项目中，因此，我们只需要直接导入其依赖就可以使用了，Spring 核心框架的 Maven 依赖坐标：

~~~xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.0.10</version>
</dependency>
~~~

**注意：**与旧版教程不同的是，Spring 6 要求你使用的Java版本为`17`及以上，包括后面我们在学习 Spring Mvc 时，要求 Tomcat 版本必须为 10 以上。这个依赖中包含了如下依赖：

![](https://raw.githubusercontent.com/HoshiSrar/Note_Images/main/img/20231128163217.png)

这里出现的都是Spring核心相关的内容，如Beans、Core、Context、SpEL以及非常关键的AOP框架，在本章中，我们都会进行讲解。

> **注意**：
>
> 如果在使用Spring框架的过程中出现如下警告：
>
> ~~~ java
> 12月 17, 2022 3:26:26 下午 org.springframework.core.LocalVariableTableParameterNameDiscoverer inspectClass
> 警告: Using deprecated '-debug' fallback for parameter name resolution. Compile the affected code with '-parameters' instead or avoid its introspection: XXXX
> ~~~
>
> 这是因为LocalVariableTableParameterNameDiscoverer在Spring 6.0.1版本已经被标记为过时，并且即将移除，请在Maven配置文件中为编译插件添加`-parameters`编译参数：
>
> ~~~xml
> <build>
>     <pluginManagement>
>         <plugins>
>             <plugin>
>                 <artifactId>maven-compiler-plugin</artifactId>
>                 <version>3.10.1</version>
>                 <configuration>
>                     <compilerArgs>
>                         <arg>-parameters</arg>
>                     </compilerArgs>
>                 </configuration>
>             </plugin>
>         </plugins>
>     </pluginManagement>
> </build>
> ~~~

在使用 Spring 时，Spring 会为我们提供一个 IoC 容器，我们需要提供一个配置文件来告诉 Spring 我们的容器有哪些 Bean 和 Bean 属性等等。（可以使用 XML、Java类、Java注解等方式提供配置文件）

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean name="user" class="org.example.entity.User"/>
</beans>
~~~

~~~java
public static void main(String[] args) throws FileNotFoundException {
    	// 通过 ClassPathXMlApplicationContext 读取配置文件
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("SpringApplication.xml");
        User user = (User) context.getBean("user");
        System.out.println(user);
    }
~~~

当然，为了追求效率还是最好使用注解开发，这样更快，XML就不详细解释了。

### 注解开发

#### 1.Java 配置类配置 Bean。

注解配置类：我们所配置的 Bean 都在这里。

~~~java
@Configuration
public class MainConfiguration {
    // 使用 @Bean 注解给返回 Bean 对象的方法
    @Bean("Username")
    public User user(){
        return new User();
    }
}
~~~

@Bean的参数

> @Bean(value="",name = "", initMethod = "", destroyMethod = "", autowireCandidate = false)

~~~java
@Bean
@Lazy(true)     //对应lazy-init属性
@Scope("prototype")    //对应scope属性
@DependsOn("teacher")    //对应depends-on属性
public Student student(){
    return new Student();
}
~~~

获取IoC容器，参数使用配置类的 class。

~~~java
ApplicationContext context = new AnnotationConfigApplicationContext(MainConfiguration.class);
~~~

@improt 可以导入其他配置类

~~~java
@Import(LBWConfiguration.class)  //在讲解到Spring原理时，我们还会遇到它，目前只做了解即可。
@Configuration
public class MainConfiguration {
~~~

需要引入其他Bean进行的注入，可以直接将其作为形参放到方法中：

~~~java
@Configuration
public class MainConfiguration {
    @Bean
    public Teacher teacher(){
        return new Teacher();
    }

    @Bean
    public Student student(Teacher teacher){//彷入参数中，Spring会自动注入
        return new Student(teacher);
    }
}
~~~

也可以配合 @Resource 或者 @Autowired 进行自动装备

~~~java
public class Student {
    @Autowired   //使用此注解来进行自动装配，由IoC容器自动为其赋值
    @Resource
    private Teacher teacher;
}
~~~

@Autowired默认采用byType的方式进行自动装配，也就是说会使用类型进行配，那么要是出现了多个相同类型的Bean，可以配合@Qualifier进行名称匹配：

~~~java
@Bean("a")
public Teacher teacherA(){
    return new Teacher();
}

@Bean("b")
public Teacher teacherB(){
    return new Teacher();
}
// 分割——————————————————
public class Student {
    @Autowired
    @Qualifier("a")   // 匹配名称为a的Teacher类型的Bean
    private Teacher teacher;
}
~~~

**注意**：随着Java版本的更新迭代，某些javax包下的包，会被逐渐弃用并移除。在JDK11版本以后，javax.annotation这个包被移除并且更名为jakarta.annotation，其中有一个非常重要的注解，叫做@Resource，它的作用与@Autowired时相同，也可以实现自动装配，但是在IDEA中并不推荐使用@Autowired 注解对成员字段进行自动装配，而是推荐使用@Resource，如果需要使用这个注解，还需要额外导入包：

~~~XML
<dependency>
    <groupId>jakarta.annotation</groupId>
    <artifactId>jakarta.annotation-api</artifactId>
    <version>2.1.1</version>
</dependency>
~~~

只不过，他们两有些机制上的不同：

- @Resource 默认**ByName** 如果找不到则 **ByType**，可以添加到set方法、字段上。
- @Autowired 默认是 **byType**，只会根据类型寻找，可以添加在构造方法、set 方法、字段、方法参数上。

因为 @Resource 的匹配机制更加合理高效，因此官方并不推荐使用 @Autowired 字段注入，当然，实际上 Spring 官方更推荐我们使用基于构造方法或是 Setter 的 @Autowired 注入，比如 Setter  注入的一个好处是，Setter 方法使该类的对象能够在以后重新配置或重新注入。其实，最后使用哪个注解都一样，随意。

除了这个注解之外，还有@PostConstruct和@PreDestroy，它们效果和init-method和destroy-method是一样的：

~~~java
@PostConstruct
public void init(){
    System.out.println("我是初始化方法");
}

@PreDestroy
public void destroy(){
    System.out.println("我是销毁方法");
}
~~~

#### 2.@Component 配置 Bean

我们可以在需要注册为 Bean 的类上添加`@Component`注解来将一个类进行注册，容器会自己通过反射来获取构造方法去生成对象。

不过要实现这样的方式，我们需要添加自动扫描来告诉 Spring ，它需要在哪些包中查找我们提供的`@Component`声明的Bean。

~~~java
@Component("BeanName")   //同样可以自己起名字
public class Student {
}
// 分割线——————————
@Configuration
@ComponentScan("com.test.bean")   // 包扫描，Spring 会自动扫描对应包下所有的类
public class MainConfiguration {
}
~~~

Spring 在扫描对应包下所有的类时，会自动将那些添加了@Component的类注册为Bean。只不过这种方式只适用于我们自己编写类，如果是第三方包提供的类，只能使用前者来完成注册，并且这种方式并不是那么的灵活。

不过，无论是通过 @Bean 还是 @Component 形式注册的 Bean ，Spring 都会为其添加一个默认的 name 属性，默认名称生成规则一类名并按照首字母小写的驼峰命名法来的，所以说 Student 对应的就是  student ，比如：

~~~java
@Component
public class Student {
}
public static void main(String[] args) {
    Student student = (Student) context.getBean("student");   //这样同样可以获取到
}
~~~

但如果是通过 @Bean 注册的，默认名称是对应的方法名称：

~~~java
@Bean
public Student artStudent(){
    return new Student();
}
public static void main(String[] args) {
    Student student = (Student) context.getBean("artStudent");   //这样同样可以获取到
}
~~~

相比传统的XML配置方式，注解形式的配置更加方便。并且，对于使用`@Component`注册的 Bean 来说，如果其构造方法不是默认无参构造，那么默认会对其每一个参数都进行自动注入：

~~~java
@Component
public class Student {
    Teacher teacher;
    
    public Student(Teacher teacher){   //如果有Teacher类型的Bean，那么这里的参数会被自动注入
        this.teacher = teacher;
    }
}
~~~

最后，对于使用的工厂模式，Spring也提供了接口，我们可以直接实现接口表示这个Bean是一个工厂Bean：

~~~java
@Component
public class StudentFactory implements FactoryBean<Student> {
    @Override
    public Student getObject() {   //生产的Bean对象
        return new Student();
    }

    @Override
    public Class<?> getObjectType() {   //生产的Bean类型
        return Student.class;
    }

    @Override
    public boolean isSingleton() {   //生产的Bean是否采用单例模式
        return false;
    }
}
~~~

顺带一提，使用注解虽然可以省事很多，代码也能变得更简洁，但是这并不代表XML配置文件就是没有意义的，它们有着各自的优点，在不同的场景下合理使用，能够起到事半功倍的效果，官方原文：

> Are annotations better than XML for configuring Spring?
>
> The introduction of annotation-based configuration raised the question of whether this approach is “better” than XML. The short answer is “it depends.” The long answer is that each approach has its pros and cons, and, usually, it is up to the developer to decide which strategy suits them better. Due to the way they are defined, annotations provide a lot of context in their declaration, leading to shorter and more concise configuration. However, XML excels at wiring up components without touching their source code or recompiling them. Some developers prefer having the wiring close to the source while others argue that annotated classes are no longer POJOs and, furthermore, that the configuration becomes decentralized and harder to control.
>
> No matter the choice, Spring can accommodate both styles and even mix them together. It is worth pointing out that through its [JavaConfig](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-java) option, Spring lets annotations be used in a non-invasive way, without touching the target components source code and that, in terms of tooling, all configuration styles are supported by the [Spring Tools for Eclipse](https://spring.io/tools).

## Spring 高级特性

### Bean Aware

Spring 提供了一些以 Aware 结尾的接口，实现了 Aware接 口的 bean 在被初始化之后，可以获取相应资源。Aware 的中文意思为**感知**。简单来说，它就是一个标识，和序列化标识一样，实现此接口的类会获得某些感知能力，Spring 容器会在 Bean 被加载时，根据类实现的感知接口，会调用类中实现的对应感知方法。

比如 BeanNameAware 之类的以 Aware 结尾的接口，这个接口获取的资源就是 BeanName：

~~~java
@Component
public class Student implements BeanNameAware {   //我们只需要实现这个接口就可以了
    
    @Override
    public void setBeanName(String name) {   //Bean在加载的时候，容器就会自动调用此方法，将Bean的名称给到我们
        System.out.println("我在加载阶段获得了Bean名字："+name);
    }
}
~~~

又比如 BeanClassLoaderAware ，它能够使得我们可以在 Bean 加载阶段就获取到当前 Bean 的类加载器：

~~~java
@Component
public class Student implements BeanClassLoaderAware {

    @Override
    public void setBeanClassLoader(ClassLoader classLoader) {
        System.out.println(classLoader);
    }
}
~~~

### 任务调度（定时器）

为了执行某些任务，我们可能需要一些非常规的操作，比如我们希望使用多线程来处理我们的结果或是执行一些定时任务，到达指定时间再去执行。这时我们首先想到的就是创建一个新的线程来处理，或是使用 TimerTask 来完成定时任务，但是我们有了 Spring 框架之后，就不用这样了，因为 Spring 框架为我们提供了更加便捷的方式进行任务调度。

**首先我们来看异步任务执行**，需要使用 Spring 异步任务支持，我们需要在配置类上添加 `@EnableAsync` 注解。

~~~java
@EnableAsync
@Configuration
public class MainConfiguration {
}
~~~

接着我们只需要在需要异步执行的方法上，添加 `@Async `注解即可将此方法标记为异步，当此方法被调用时，会异步执行，也就是新开一个线程执行，而不是在当前线程执行。：

~~~java
@Component
public class Student {
    public void syncTest() throws InterruptedException {
        System.out.println(Thread.currentThread().getName()+"我是同步执行的方法，开始...");
        Thread.sleep(3000);
        System.out.println("我是同步执行的方法，结束！");
    }

    @Async
    public void asyncTest() throws InterruptedException {
        System.out.println(Thread.currentThread().getName()+"我是异步执行的方法，开始...");
        Thread.sleep(3000);
        System.out.println("我是异步执行的方法，结束！");
    }
}
~~~

异步执行的任务并不是在当前线程启动的，而是在其他线程启动的，所以说不会阻塞当前线程。

因此，当我们要将Bean的某个方法设计为异步执行时，就可以直接添加这个注解。但是需要注意，添加此注解要求方法的返回值只能是  void 或是 Future （JUC相关）类型才可以。

**接下来是定时任务**，定时任务其实就是指定在哪个时候再去执行，我们使用过 TimerTask 来执行定时任务。 Spring 中的定时任务是全局性质的，当 Spring 程序启动后，定时任务也就跟着启动了，我们可以在配置类上添加`@EnableScheduling`注解来启用定时任务：

~~~java
@EnableScheduling
@Configuration
@ComponentScan("com.test.bean")
public class MainConfiguration {
}
~~~

接着我们可以直接在配置类里面编写定时任务，把我们要做的任务写成方法，并添加`@Scheduled`注解：

~~~java
@Scheduled(fixedRate = 2000)   //单位依然是毫秒，这里是每两秒钟打印一次
public void task(){
    System.out.println("我是定时任务！"+new Date());
}
~~~

`@Scheduled`中有很多参数，我们需要指定'cron', 'fixedDelay(String)', or 'fixedRate(String)'的其中一个，否则无法创建定时任务，他们的区别如下：

- fixedDelay：在上一次定时任务执行完之后，间隔多久继续执行。
- fixedRate：无论上一次定时任务有没有执行完成，两次任务之间的时间间隔。
- cron：如果嫌上面两个不够灵活，你还可以使用cron表达式来指定任务计划。

这里简单讲解一下cron表达式：https://blog.csdn.net/sunnyzyq/article/details/98597252

@Scheduled注解属性

~~~ java
//corn表达式
@Scheduled(cron = "0/5 * * * * ? *")

### corn表达式
    
详细百度 ：6个数字  秒~分~时~天~月~周	 * * * * * *
~~~



### 监听器

监听实际上就是等待某个事件的触发，当事件触发时，对应事件的监听器就会被通知，只是在Spring中使用不是很频繁。简单介绍：

~~~java
@Component
public class TestListener implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent event) {
        System.out.println(event.getApplicationContext());   //可以直接通过事件获取到事件相关的东西
    }
}
~~~

要编写监听器，我们只需要让 Bean 继承 ApplicationListener 就可以了，并且将类型指定为对应的 Event 事件，这样，当发生某个事件时就会通知我们，比如 ContextRefreshedEvent，这个事件会在 Spring 容器初始化完成会触发一次。

Spring内部有各种各样的事件，当然我们也可以自己编写事件，然后在某个时刻发布这个事件到所有的监听器：

~~~java
public class TestEvent extends ApplicationEvent {   //自定义事件需要继承ApplicationEvent
    public TestEvent(Object source) {
        super(source);
    }
}

@Component
public class TestListener implements ApplicationListener<TestEvent> {
    @Override
    public void onApplicationEvent(TestEvent event) {
        System.out.println("发生了一次自定义事件，成功监听！");
    }
}
~~~

比如现在我们希望在定时任务中每秒钟发生一次这个事件：

> 我们需要使用 Spring 的 ApplicationEventPublisher  (应用事件发布器)来发布我们自定义的事件，使用 BeanAware 感知获取，或者通过注入获取。
>
> ~~~
>  @Resourcprivate 
> ApplicationEventPublisher publisher 
> ~~~

~~~java
@Component
public class TaskComponent  implements ApplicationEventPublisherAware {  
  	//要发布事件，需要拿到 ApplicationEventPublisher，这里我们通过 Aware 在初始化的时候拿到
  	//实际上我们的 ApplicationContext 就是 ApplicationEventPublisher 的实现类，这里拿到的就是
  	//我们创建的 ApplicationContext 对象
    ApplicationEventPublisher publisher;

    @Scheduled(fixedRate = 1000)   //一秒一次
    public void task(){
      	//直接通过 ApplicationEventPublisher 的 publishEvent 方法发布事件
      	//所有监听这个事件的监听器，都能监听到该事件发生。this表示发布的源头，这里指本类 TaskComponent
        publisher.publishEvent(new TestEvent(this));
    }

    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher publisher) {
        this.publisher = publisher;
    }
}
~~~

监听器和发布都需要被注册为 Bean 被 Spring 所管理。

------

## SpringEL表达式

> SpEL 是一种强大，简洁的装配 Bean 的方式，它可以通过运行期间执行的表达式将值装配到我们的属性或构造函数当中，更可以调用 JDK 中提供的静态常量，获取外部 Properties/aml 文件中的的配置。

配置文件：

~~~yaml
test: 张三
~~~

配置类添加 @PropertySource(value = "classpath:application.yml") 注解获取 yml 或 Properties 配置文件

~~~java
@PropertySource(value = "classpath:application.yml")
~~~

注入使用（可能有 Bug，yam 格式不支持多级的参数注入，也就是 ${user.name} 这种注入失败，Properties 不会）

```java
@Value("${test}")
String name;
```

@Value 注解将自动读取配置文件，将数据对应映射。

## AOP面向切面

AOP（Aspect Oriented Programming，面向切面编程）。较为官方说法：在运行时，动态地将代码切入到类的指定方法、指定位置上。也就是说，我们可以使用AOP来帮助我们在方法执行前或执行之后，做一些额外的操作，本质上就是代理。

 AOP 是对 OOP （Object Oriented Programming，面向对象编程）的有益补充，相比于OOP， AOP 的编程思想与 OOP 不同，在代码的调用方式相反。举例来说，AOP 的思路：**我们需要 A 方法中的某个位置调用 B 方法，在 OOP 中，是由 A 方法主动调用 B ，而 AOP 是由 B 方法自己决定在 A 方法执行的某个时刻（前/中/后）将会执行。**对于 B 方法来说，B 从 OOP的 被动等待其他类调用，变为主动决定将会在某些方法触发时执行，从被动转为主动，这就是 AOP 的核心思路。

通过AOP我们可以在保证原有代码不变的情况下，添加额外的操作，比如我们的某些方法执行完成之后，需要打印日志，那么这个时候，我们就可以使用AOP来帮助我们完成，它可以批量地为同样的方法添加动作。相当于将我们原有的方法，在不改变源代码的基础上进行了增强处理。

**相关概念：**

> ###  1.通知（Advice）
>
> 　　即我们想要添加的功能，比如日志、安全、权限等，一般是具体的某个方法。我们事先定义好一些想要使用的功能，然后在想用的地方。  
>
> ### 2.连接点（JoinPoint）
>
> 　　即我们可以使用通知的地方，可以是每个方法，也可以是某个异常抛出都可以作为连接点，它的意义代表着这些地方都可以被我们所增强，但 spring 只支持方法连接点.其他如 aspectJ 还可以让你在构造器或属性注入时都行，只要记住，和方法有关的前前后后（抛出异常），都是连接点。
>
> ###   3.切点（Pointcut）
>
> 　切点是指具有相同特性可以被划为一组的切点集合，AOP将可被抽取共性功能的**方法**称为**切入点**。
>
> 一个类里，假设有 5 个方法，就至少有 5 个连接点，但是我们并不想在所有**连接点附近**都使用通知（使用通知叫织入），你只想让某几个位置使用，就使用定义切点来包含这写方法，使用切点来筛选连接点，选中我们想要使用通知的方法。
>
> * 切点是连接点的一个子集，因为它仅表示将会应用切面的一些特定连接点。
> * 关键点：切点定义了哪些连接点将会得到（或者说运用？）通知
>
> ###   4.切面（Aspect）
>
> 　　切面是通知和切点的结合，同时还定义了通知执行在连接点（切点）附近的具体位置（切点之前/中/后）。在切面中，通知决定将会做什么，而切点觉得了在哪些位置（指定到底是哪些方法），同时切面还定义了什么时候做（切点之前/中/后，before，around,after）。这三项合起来就是一个完整的切面定义。
>
> * 实际上我们并不会使用连接点而只使用切点。
>
> ------
>
> ###   5.引入（introduction）
>
> 　　允许我们向现有的目标类（连接点）添加新方法。就是把切面（也就是新方法属性：通知定义的）用到目标类的切点中
>
> ###   6.目标（target）
>
> 　　引入中所提到的目标类，也就是要被通知的对象，是我们真正的业务逻辑，它可以在毫不知情的情况下，被我们所织入切面。而目标本身可以专注于业务本身的逻辑。
>
> ###   7.代理(proxy)
>
> 　　如何实现整套 aop 机制的，都是通过代理。
>
> ###   8.织入(weaving)
>
> 　　把切面应用到目标对象来创建新的代理对象的过程。有3种方式，spring采用的是运行时。



~~~xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>6.0.10</version>
</dependency>
~~~

Spring是支持AOP编程的框架之一（实际上它整合了AspectJ框架的一部分），使用AOP需要导入相关依赖。

那么，如何使用AOP呢？首先我们要明确，要实现AOP操作，我们需要知道这些内容：

> 1. 需要切入的类，类的哪个方法需要被切入
> 2. 切入之后需要执行什么动作
> 3. 是在方法执行前切入还是在方法执行后切入
> 4. 如何告诉 Spring 需要进行切入

### 使用 XML

 我们先用XML配置的方式来理解。比如现在我们希望对这个学生对象的`study`方法进行增强，在不修改源代码的情况下，增加一些额外的操作：

~~~java
public class Student {
    public void study(){
        System.out.println("室友还在打游戏，我狠狠的学Java，太爽了"); 
      	//现在我们希望在这个方法执行完之后，打印一些其他的内容，在不修改原有代码的情况下，该怎么做呢？
    }
}
~~~

那么我们按照上面的流程，依次来看，首先需要解决的问题是，找到需要切入的类，很明显，就是这个Student类，我们要切入的是这个`study`方法。

第二步，我们切入之后要做什么呢？这里我们直接创建一个新的类，并将要执行的操作写成一个方法：

~~~java
public class StudentAOP {
  	//这个方法就是我们打算对其进行的增强操作
    public void afterStudy() {
        System.out.println("为什么毕业了他们都继承家产，我还倒给他们打工，我努力的意义在哪里...");
    }
}
~~~

两者都需要注册为Bean

~~~xml
<bean class="org.example.entity.Student"/>
<bean id="studentAOP" class="org.example.entity.StudentAOP"/>
~~~

第三步，我们要明确这是在方法执行之前切入还是执行之后切入，很明显，按照上面的要求，我们需要执行之后进行切入。

第四步，最关键的来了，我们怎么才能告诉Spring我们要进行切入操作呢？这里我们需要在配置文件中进行AOP配置：

~~~xml
<aop:config>
</aop:config>
~~~

接着我们需要添加一个新的切点，首先填写ID，这个随便起都可以：

~~~xml
<aop:pointcut id="test" expression=""/>
~~~

然后就是通过后面的`expression`表达式来选择到我们需要切入的方法，这个表达式支持很多种方式进行选择，Spring AOP支持以下AspectJ切点指示器（PCD）用于表达式：

> - `execution`：用于匹配方法执行连接点。这是使用Spring AOP时使用的主要点切割指示器。
> - `within`：限制匹配到某些类型的连接点（使用Spring AOP时在匹配类型中声明的方法的执行）。
> - `this`：限制与连接点匹配（使用Spring AOP时方法的执行），其中bean引用（Spring AOP代理）是给定类型的实例。
> - `target`：限制匹配连接点（使用Spring AOP时方法的执行），其中目标对象（正在代理的应用程序对象）是给定类型的实例。
> - `args`：限制与连接点匹配（使用Spring AOP时方法的执行），其中参数是给定类型的实例。
> - `@target`：限制匹配连接点（使用Spring AOP时方法的执行），其中执行对象的类具有给定类型的注释。
> - `@args`：限制匹配到连接点（使用Spring AOP时方法的执行），其中传递的实际参数的运行时类型具有给定类型的注释。
> - `@within`：限制与具有给定注释的类型中的连接点匹配（使用Spring AOP时在带有给定注释的类型中声明的方法的执行）。
> - `@annotation`：与连接点主体（在Spring AOP中运行的方法）具有给定注释的连接点匹配的限制。

其中，我们主要学习的`execution`填写格式如下：

~~~xml
修饰符 包名.类名.方法名称(方法参数)
~~~

> 1. 修饰符：public、protected、private、包括返回值类型、static等等（使用*代表任意修饰符）
> 2. 包名：如com.test（* 代表全部，比如com.*代表com包下的全部包）
> 3. 类名：使用*也可以代表包下的所有类
> 4. 方法名称：可以使用*代表全部方法
> 5. 方法参数：填写对应的参数即可，比如(String, String)，也可以使用*来代表任意一个参数，使用..代表所有参数。

详情可查看[Spring AOP切点表达式（Pointcut）详解](./Spring AOP切点表达式（Pointcut）详解)

也可以使用其他属性来进行匹配，比如`@annotation`可以用于表示标记了哪些注解的方法被切入，这里我们就只是简单的执行，所以说只需要这样写就可以了：

~~~xml
<aop:pointcut id="test" expression="execution(* org.example.entity.Student.study())"/>
~~~

这样，我们就指明了连接点，也就是待被切入的方法，然后就是将我们的增强方法，我们在里面继续添加`aop:aspect`标签，并使用`ref`属性将其指向我们刚刚注册的AOP类Bean：

~~~xml
<aop:config>
    <aop:pointcut id="test" expression="execution(* org.example.entity.Student.study())"/>
    <aop:aspect ref="studentAOP">
        <aop:before method="afterStudy" pointcut-ref="test"/>
    </aop:aspect>
</aop:config>
~~~

### 使用注解

创建切面,定义切点，切点附近具体位置（这里是由 @Before 决定）

~~~java
@Aspect
@Component
public class UserAop {
    // 定义切点，匹配规则为 public 的所有返回值类型（包括void），在entity 包下的所有名叫 name 的方法
    // 执行具体位置为 Before，切点之前
    @Before("execution(public * org.example.entity.*.name(..))")
    public void before(){
        System.out.println("entity下的所有 叫做name的方法");
    }
}
~~~

ps:需要开启 Spring 的 AOP 支持，这里的所有 Bean 都需要被注册并被扫描。SpringBoot 项目打在启动类上。

~~~java
@EnableAspectJAutoProxy
@ComponentScan("org.example.entity")
@Configuration
public class MainConfiguration {
}
~~~

环绕注解稍微特殊一点，且 ProceedingJoinPoint point 只支持环绕模式：

~~~java
@Around("execution(* org.example.entity.*.name(..))")
public Object around(ProceedingJoinPoint point) throws Throwable {
    System.out.println("方法执行之前！");
    Object val = point.proceed();
    System.out.println("方法执行之后！");
    return val;
}
~~~

我们可以填入使用 JoinPoint pointcut 来获取

~~~java
@Before("execution(public * org.example.entity.*.name(..))")
    public void before(JoinPoint pointcut){
       pointcut.getArgs()//获取方法中的参数
    }
~~~

JoinPoint 和 ProceedingJoinPoint 中都有大量和切点的相关信息，可以通过它们获取：

> * 注解中的值，
>
> * 切点方法，
>
> * 切点中的参数：point.getArgs()。
>
>   等等等等。

也可以使用命名绑定模式快速获得原方法的参数：
~~~java
// 需增强的业务代码
public void study(String str){
    ......
}
// AOP1
@Before(value = "execution(* org.example.entity.Student.study(..)) && args(str)")
//命名绑定模式就是根据下面的方法参数列表进行匹配
//这里args指明参数，注意需要跟原方法保持一致，然后在argNames中指明
public void before(String str){
    System.out.println(str);   //可以快速得到传入的参数
}

~~~

除了@Before，还有很多可以直接使用的注解，比如@AfterReturning、@AfterThrowing等，比如@AfterReturning：

~~~java
// AOP2
@AfterReturning(value = "execution(* org.example.entity.Student.study())", argNames = "returnVal", returning = "returnVal")   //使用returning 指定接收方法返回值的参数 returnVal
public void afterReturn(Object returnVal){
    System.out.println("返回值是："+returnVal);
}
~~~

多参数的例子：

~~~Java
	// 业务代码，双参数
	public String name(int a,String b){    
    }
	// AOP，这里注意args(b,a)中参数填入的顺序要与业务方法中的参数一致
	@Around(value = "execution(public * org.example.entity.*.name(..)) && args(a,b)", argNames = "point,a,b")
    public Object before(ProceedingJoinPoint point,int a, String b) throws Throwable {
        return point.proceed(new Object[]{a,b});
    }
~~~

