## Spring中开启切面代理
在JavaEE开发中，经常遇到日志记录/登录检查的需求，如果要给每个方法都加上同样的代码，费时费力还不容易扩展，于是就有了切面这种东西，说白了就是生成一个代理，在某方法执行前后加上自己的逻辑，Spring中有两种代理模式，一种是基于JDK的接口代理（不能代理类），一种是基于CGLIB的代理（接口和类都可以），CGLIB是依赖于ASM的字节码操作类库，可以方便的生成指定类的子类代理，实现方法增强

Spring中有两种切面代理方式：基于注解和基于表达式，基于注解一般用于被代理的类方法较少的时候，不然写完代理每个方法上面都要加上注解，基于表达式可以进行匹配，例如com.test包下面以qry开头的方法加上切面代理
#### 环境
- jar包

在项目中引入如下jar包，我的Spring版本是4.1.4.RELEASE
```xml
// gradle 
compile(
    'org.aspectj:aspectjweaver:1.9.2',
    'org.aspectj:aspectjrt:1.8.13'
)

//maven 
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjrt</artifactId>
    <version>1.8.13</version>
</dependency>
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.2</version>
</dependency>
```
- 配置
在spring-mvc.xml中配置如下内容，开启切面（web.xml中org.springframework.web.servlet.DispatcherServlet引入了spring-mvc.xml）
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.1.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.1.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">
<!--重点关注下面两句-->
<!--注解扫描-->
<context:component-scan base-package="com.test" />
<!--AOP切面 proxy-target-class="true" 表示开启CGLIB代理-->
<aop:aspectj-autoproxy proxy-target-class="true"/>
<!--...省略内容...-->
</beans>
```

#### 基于注解的切面代理
- 首先定义一个注解
```java
package com.test.support;
import java.lang.annotation.*;
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
@Documented
public @interface AOP {
    String desc() default "自定义AOP注解";
}
```
- 其次定义切面代理类，下面12345代表执行顺序，可以看到，@Around是最先被执行的，其次才是@Before
```java
package com.test.support;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;
@Aspect    //说明这是一个切面bean
@Component //交给Spring去管理bean
public class AOPSupport {
    /**
    * 基于注解定义切点，该方法无方法体,主要为方便同类中其他方法使用此处配置的切入点 
    */
    @Pointcut("@annotation(com.test.support.AOP)")
    public void addLog() {
    }
    @Before("addLog()")
    public void before(JoinPoint point){
        System.out.println("2:Before...");
    }
    //环绕切面
    @Around("addLog()")
    public Object around(ProceedingJoinPoint jp) throws Throwable{
        System.out.println("1:Around before...");
        Object res = jp.proceed();//执行被代理的方法
        System.out.println("3:Around after...");
        return res;
    }
    @After("addLog()")
    public void after(JoinPoint point){
        System.out.println("4:After...");
    }
    @AfterReturning("addLog()")
    public void afterReturning(JoinPoint point){
        System.out.println("5:AfterReturning...");
    }
}
```
- 在要被切入的方法体上面添加注解

```java
    @AOP(desc = "我只是一个描述，表明我在此方法上面添加了切面")
    @RequestMapping("/exit")
    public String exit() {
        System.out.println("exit....");
        return "exit";
    }
```
- 调用该方法，执行结果
```
1:Around before...
2:Before...
exit....
3:Around after...
4:After...
5:AfterReturning...
```

#### 基于表达式的切面代理
- 定义切面代理类，
```java
package com.test.support;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;
@Aspect    //说明这是一个切面bean
@Component //交给Spring去管理bean
public class AOPSupportExp {
    /**
    * 基于表达式定义切点，该方法无方法体,主要为方便同类中其他方法使用此处配置的切入点
    */
    @Pointcut("within(com.test.controller.*)")//表示com.test.controller包下面所有方法都使用该切点
    public void addLog() {
    }
    @Before("addLog()")
    public void before(JoinPoint point){
        System.out.println("2:Before...");
    }
    //环绕切面
    @Around("addLog()")
    public Object around(ProceedingJoinPoint jp) throws Throwable{
        System.out.println("1:Around before...");
        Object res = jp.proceed();//执行被代理的方法
        System.out.println("3:Around after...");
        return res;
    }
    @After("addLog()")
    public void after(JoinPoint point){
        System.out.println("4:After...");
    }
    @AfterReturning("addLog()")
    public void afterReturning(JoinPoint point){
        System.out.println("5:AfterReturning...");
    }
}
```
- 调用com.test.controller包下面任意类的任意方法，结果如下
```
1:Around before...
2:Before...
original method exec....
3:Around after...
4:After...
5:AfterReturning...
```

#### 关于切面表达式
切面表达式具有很强的描述性，举几个例子
```java
//切入包下面所有方法
@Pointcut("within(com.test.controller.*)")

/**
 * 切入TestServiceImpl所有qry开头的方法
 * execution表示在方法执行时触发
 * * 代表任意返回类型
 * com.test.service.TestServiceImpl.qry*表示该类下所有qry开头的方法
 * (..)表示任意参数
 */
@Pointcut("execution(* com.test.service.TestServiceImpl.qry*(..))")

/**
 * 在执行表达式的时候，我们可以通过逻辑运算符&&(and) , ||(or) , !(not)对表达式进行搭配
 * 切入在com.test.service包下面TestServiceImpl所有qry开头的方法
 */
@Pointcut("execution(* com.test.service.TestServiceImpl.qry(..) && within(com.test.service.*))")

/**
 * 切入在com.test.service包下面所有public开头的方法
 * 第一个*表示所有返回值类型
 * 第二个*表示所有方法名称
 * (..)表示任意参数
 */
@Pointcut("execution(public * *(..)) && within(com.test.service.*))")


/**
 * 切入在com.test.service包下面所有以To结尾的方法
 * 第一个*表示所有返回值类型
 * *To表示所有以To结尾的方法
 * (..)表示任意参数
 */
@Pointcut("execution(* *To(..)) && within(com.test.service.*))")


/**
 * 切入在com.test包下面所有方法（除了类Login下面的login方法）
 * 第一个*表示所有返回值类型
 * (..)表示任意参数
 */
@Pointcut("within(com.test.*) && !execution(* com.test.Login.login(..))")
```