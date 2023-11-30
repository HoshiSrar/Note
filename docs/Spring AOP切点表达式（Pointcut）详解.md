Spring AOP切点表达式（Pointcut）详解

Spring 的 AOP 中的一个核心概念是切点（Pointcut），切点表达式定义通知（Advice）执行的范围。
理解 AOP 通知参阅：《Spring AOP通知（Advice）详解》

## 一、概述
Spring AOP 只支持 Spring Bean 的方法切入，所以切点表达式只会匹配 Bean 类中的方法。

AOP术语

重点为通知和切点以及切面，通知表示切入时以何种类型切入，注解中可以添加切点；切点表示被切入的位置，通常是方法，也可是注释；                                            切面为通知和切点二者结合，在注解开发表示为一个切面类。

1. 通知（Advice）
     切面的工作被称为通知。通知定义了切面是什么以及何时使用。除了描述切面要完成的工作，通知还解决了何时执行这个工作的问题。
   5种通知类型：

> **前置通知**（Before）：在目标方法被调用之前调用通知功能
>
> **后置通知**（After）：在目标方法完成之后调用通知，此时不会关心方法的输出是什么
>
> **返回通知**（After-returning）：在目标方法成功执行之后调用通知
>
> **异常通知**（After-throwing）：在目标方法抛出异常后调用通知
>
> **环绕通知**（Around）：通知包裹了被通知的方法，在被通知的方法调用之前和之后执行自定义的行为

> 后置通知和返回通知的区别是，后置通知是不管方法是否有异常，都会执行该通知；而返回通知是方法正常结束时才会执行。

1. 连接点（Join point）
     连接点是在应用执行过程中能够插入切面的一个位置。
2. 切点（Pointcut）
     **切点定义了需要执行在哪些连接点上会执行通知。**一个切面并不需要通知应用的所有连接点，切点有助于缩小切面所通知的连接点范围。如果说通知定义了切面的“什么”和“何时”的话，那么切点就定义了“何处”。
3. 切面（Aspect）
     切面是通知和切点的结合。通知和切点共同定义了切面的全部内容——它是什么，在何时和在何处完成其功能。
4. 引入（Introduction）
     引入允许我们向现有的类添加新方法或属性。
5. 织入（Weaving）
     织入是把切面应用到目标对象并创建新的代理对象的过程。切面在指定的连接点被织入到目标对象中。在目标对象的生命周期中有很多个点可以进行织入：

## 二、切点表达式配置

### 1. 内置配置
定义切面通知，在 @Before 或 @AfterReturning 等通知注解中指定表达式。

~~~java
@Aspect
@Component
public class DemoAspect {
    
	//注解定义在哪个方法会执行该方法
    @Before("execution(* cn.codeartist.spring.aop.advice.*.*(..))")
    public void doBefore() {
        // 自定义逻辑，
    }
}
~~~

### 2. 注解配置
创建切面类，定义一个方法并使用 @Pointcut 注解的表达式来指定切入点。
然后在定义切面通知时，在通知注解中指定定义表达式的方法签名。

```java
@Aspect
@Component
public class DemoAspect {

    @Pointcut("execution(* cn.codeartist.spring.aop.aspectj.*.*(..))")
    private void pointcut() {
        // 切点表达式定义方法，方法修饰符可以是private或public
    }

    @Before("pointcut()")
    public void doBefore(JoinPoint joinPoint) {
        // 自定义逻辑
    }
}
```

### 3. 公共配置
在任意类中，定义一个公共方法并使用 @Pointcut 注解来指定表达式。

~~~java
public class CommonPointcut {
    @Pointcut("execution(* cn.codeartist.aop.*..*(..))")
	public void pointcut() {
    // 注意定义切点的方法的访问权限为public
    }
}
~~~

在切面类中定义切面通知时，在通知注解中指定定义表达式的方法签名全路径。

~~~java
@Aspect
@Component
public class DemoAspect {

    @Before("cn.codeartist.aop.CommonPointcut.pointcut()")
    public void commonPointcut() {
        // 自定义逻辑
    }
}
~~~

------



## 三、切点表达式类型

Spring AOP 支持以下几种切点表达式类型。

### **execution**

匹配方法切入点。根据表达式描述匹配方法，是最通用的表达式类型，可以匹配方法、类、包。

表达式模式：

~~~java
 execution(modifier? ret-type declaring-type?name-pattern(param-pattern) throws-pattern?)
~~~

表达式解释：

* modifier：匹配修饰符，public, private 等，省略时匹配任意修饰符

* ret-type：匹配返回类型，使用 * 匹配任意类型

* declaring-type：匹配目标类，省略时匹配任意类型

​		.. 匹配包及其子包的所有类
* name-pattern：匹配方法名称，使用 * 表示通配符

  匹配任意方法
  set* 匹配名称以 set 开头的方法

* param-pattern：匹配参数类型和数量

  ​	() 匹配没有参数的方法
  ​	(..) 匹配有任意数量参数的方法
  ​	(*) 匹配有一个任意类型参数的方法
  ​	(*,String) 匹配有两个参数的方法，并且第一个为任意类型，第二个为 String 类型

* throws-pattern：匹配抛出异常类型，省略时匹配任意类型

使用示例：

~~~java
// 匹配public方法
execution(public * *(..))

// 匹配名称以set开头的方法
execution(* set*(..))

// 匹配AccountService接口或类的方法
execution(* com.xyz.service.AccountService.*(..))

// 匹配service包及其子包的类或接口
execution(* com.xyz.service..*(..))
~~~

### within

匹配指定类型。匹配指定类的任意方法，不能匹配接口。

表达式模式：

~~~java
within(declaring-type)
~~~


使用示例：

~~~java
// 匹配service包的类
within(com.xyz.service.*)

// 匹配service包及其子包的类
within(com.xyz.service..*)

// 匹配AccountServiceImpl类
within(com.xyz.service.AccountServiceImpl)

~~~



### this

匹配代理对象实例的类型，匹配在运行时对象的类型。

注意：基于 JDK 动态代理实现的 AOP，this 不能匹配接口的实现类，因为代理类和实现类并不是同一种类型，详情参阅《Spring中的AOP和动态代理》

表达式模式：

~~~java 
this(declaring-type)
~~~

使用示例：
~~~java
// 匹配代理对象类型为service包下的类
this(com.xyz.service.*)

// 匹配代理对象类型为service包及其子包下的类
this(com.xyz.service..*)

// 匹配代理对象类型为AccountServiceImpl的类
this(com.xyz.service.AccountServiceImpl)
~~~

### target
匹配目标对象实例的类型，匹配 AOP 被代理对象的类型。

表达式模式：
~~~java
target(declaring-type)
~~~

使用示例：
~~~java
// 匹配目标对象类型为service包下的类
target(com.xyz.service.*)

// 匹配目标对象类型为service包及其子包下的类
target(com.xyz.service..*)

// 匹配目标对象类型为AccountServiceImpl的类
target(com.xyz.service.AccountServiceImpl)
~~~



三种表达式匹配范围如下：

| 表达式匹配范围 | within | this | target |
| -------------- | ------ | ---- | ------ |
| 接口           | ✘      | ✔    | ✔      |
| 实现接口的类   | ✔      | 〇   | ✔      |
| 不实现接口的类 | ✔      | ✔    | ✔      |



### args

匹配方法参数类型和数量，参数类型可以为指定类型及其子类。

使用 execution 表达式匹配参数时，不能匹配参数类型为子类的方法。

表达式模式：

~~~java
args(param-pattern)
~~~


使用示例：

~~~java
// 匹配参数只有一个且为Serializable类型（或实现Serializable接口的类）
args(java.io.Serializable)

// 匹配参数个数至少有一个且为第一个为Example类型（或实现Example接口的类）
args(cn.codeartist.spring.aop.pointcut.Example,..)
~~~



### bean

通过 bean 的 id 或名称匹配，支持 * 通配符。

表达式模式：

~~~java
bean(bean-name)
~~~


使用示例：

~~~java
// 匹配名称以Service结尾的bean
bean(*Service)

// 匹配名称为demoServiceImpl的bean
bean(demoServiceImpl)
~~~

### @within

匹配指定类型是否含有注解。当定义类时使用了注解，该类的方法会被匹配，但在接口上使用注解不匹配。

使用示例：

~~~java
// 匹配使用了Demo注解的类
@within(cn.codeartist.spring.aop.pointcut.Demo)
~~~

### @target

匹配目标对象实例的类型是否含有注解。当运行时对象实例的类型使用了注解，该类的方法会被匹配，在接口上使用注解不匹配。

使用示例：

~~~java
// 匹配对象实例使用了Demo注解的类
@target(cn.codeartist.spring.aop.pointcut.Demo)
~~~

### @annotation

匹配方法是否含有注解。当方法上使用了注解，该方法会被匹配，在接口方法上使用注解不匹配。

使用示例：
~~~java
// 匹配使用了Demo注解的方法
@annotation(cn.codeartist.spring.aop.pointcut.Demo)
~~~

### @args

匹配方法参数类型是否含有注解。当方法的参数类型上使用了注解，该方法会被匹配。

使用示例：
~~~java
// 匹配参数只有一个且参数类使用了Demo注解
@args(cn.codeartist.spring.aop.pointcut.Demo)

// 匹配参数个数至少有一个且为第一个参数类使用了Demo注解
@args(cn.codeartist.spring.aop.pointcut.Demo,..)
~~~
切点表达式的参数匹配

切点表达式中的参数类型，可以和通知方法的参数通过名称绑定，表达式中不需要写类或注解的全路径，而且能直接获取到切面拦截的参数或注解信息。
~~~java
@Before("pointcut() && args(name,..)")
public void doBefore(String name) {
    // 切点表达式增加参数匹配，可以获取到name的信息
}

@Before("@annotation(demo)")
public void doBefore(Demo demo) {
    // 这里可以直接获取到Demo注解的信息
}
~~~
切点表达式的参数匹配同样适用于 @within, @target, @args

怎样编写一个好的切点表达式？

要使切点的匹配性能达到最佳，编写表达式时，应该尽可能缩小匹配范围，切点表达式分为三大类：

* 类型表达式：匹配某个特定切入点，如 execution

* 作用域表达式：匹配某组切入点，如 within

* 上下文表达式：基于上下文匹配某些切入点，如 this、target 和 @annotation


一个好的切点表达式应该至少包含前两种（类型和作用域）类型。
作用域表达式匹配的性能非常快，所以表达式中尽可能使用作用域类型。
上下文表达式可以基于切入点上下文匹配或在通知中绑定上下文。
单独使用类型表达式或上下文表达式比较消耗性能（时间或内存使用）。

## 四、切点表达式组合

使用 &&、|| 和 ! 来组合多个切点表达式，表示多个表达式“与”、“或”和“非”的逻辑关系。
这可以用来组合多种类型的表达式，来提升匹配效率。

~~~java
// 匹配doExecution()切点表达式并且参数第一个为Account类型的方法
@Before("doExecution() && args(account,..)")
public void validateAccount(Account account) {
    // 自定义逻辑
}
~~~

## 五、附录

### 常用注解

| 注解      | 描述           |
| --------- | -------------- |
| @Pointcut | 指定切点表达式 |

### 切点表达式类型

| 表达式类型  | 描述                               |
| ----------- | ---------------------------------- |
| execution   | 匹配方法切入点                     |
| within      | 匹配指定类型                       |
| this        | 匹配代理对象实例的类型             |
| target      | 匹配目标对象实例的类型             |
| args        | 匹配 方法参数                      |
| bean        | 匹配 bean 的 id 或名称             |
| @within     | 匹配类型是否含有注解               |
| @target     | 匹配目标对象实例的类型是否含有注解 |
| @annotation | 匹配方法是否含有注解               |
| @args       | 匹配方法参数类型是否含有注解       |

1. 示例代码
    Gitee 仓库：https://gitee.com/code_artist/spring


原文链接：https://blog.csdn.net/weixin_43793874/article/details/124753521