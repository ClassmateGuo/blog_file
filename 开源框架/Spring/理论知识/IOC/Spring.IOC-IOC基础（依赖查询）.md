# Spring.IOC-IOC基础（依赖查询）

# 依赖查找

## BeanFactory与ApplicationContext

### ofType

假如一个接口有多个实现,如果想一次性把所有的都拿到,那么使用`getBean`方法显然是不够用的了,需要使用额外的方式.

#### 声明bean和配置文件
代码结构

```
-dao

--impl

---DemoMySQLDao

---DemoOracleDao

---DemoPostgreDao

--DemoDao
```
quickstart-oftype.xml 文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="demoMySQLDao" class="com.linkedbear.spring.basic_dl.c_oftype.dao.impl.DemoMySQLDao"/>
    <bean id="demoOracleDao" class="com.linkedbear.spring.basic_dl.c_oftype.dao.impl.DemoOracleDao"/>
    <bean id="demoPostgreDao" class="com.linkedbear.spring.basic_dl.c_oftype.dao.impl.DemoPostgresDao"/>
</beans>
```
#### BeanFactory没有一次性获取的接口
<img width="798" alt="image" src="https://user-images.githubusercontent.com/118878596/204947200-2de3999f-3867-457b-be5b-7052486b87ce.png">

#### 使用ApplicationContext获取

```java
public class OfTypeApplication {
    
    public static void main(String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("quickstart-oftype.xml");
      //传入一个接口 / 抽象类  
      Map<String, DemoDao> beans = ctx.getBeansOfType(DemoDao.class);
      //遍历容器中所有的实现类 / 子类
        beans.forEach((beanName, bean) -> {
            System.out.println(beanName + " : " + bean.toString());
        });
    }
}
```

这样可以实现传入一个接口/抽象类,返回容器中所有的实现类/子类.



## BeanFactory 与 ApplicationContext

`ApplicationContext` 也是一个接口，而且通过接口继承关系发现它是 `BeanFactory` 的子接口。

<img width="1486" alt="image" src="https://user-images.githubusercontent.com/118878596/204947658-8d290351-8cb2-47e1-84fa-fb3d4b8ca457.png">

### 来自官网的解释

[官方文档]([Core Technologies (spring.io)](https://docs.spring.io/spring-framework/docs/5.2.x/spring-framework-reference/core.html#beans-introduction)),这一段落解释了BeanFactory与ApplicationContext的关系:

> The and packages are the basis for Spring Framework’s IoC container. The [`BeanFactory`](https://docs.spring.io/spring-framework/docs/5.2.22.RELEASE/javadoc-api/org/springframework/beans/factory/BeanFactory.html) interface provides an advanced configuration mechanism capable of managing any type of object. [`ApplicationContext`](https://docs.spring.io/spring-framework/docs/5.2.22.RELEASE/javadoc-api/org/springframework/context/ApplicationContext.html) is a sub-interface of . It adds:`org.springframework.beans``org.springframework.context``BeanFactory``
>
> **`org.springframework.beans` 和 `org.springframework.context` 包是 SpringFramework 的 IOC 容器的基础。`BeanFactory` 接口提供了一种高级配置机制，能够管理任何类型的对象。`ApplicationContext` 是 `BeanFactory` 的子接口。它增加了：**
>
> - Easier integration with Spring’s AOP features
>   - **更轻松地与Spring的AOP功能集成**
> - Message resource handling (for use in internationalization)
>   - **消息资源处理（用于国际化）**
> - Event publication
>   - **事件发布**
> - Application-layer specific contexts such as the for use in web applications.`WebApplicationContext`
>   - **应用程序层特定的上下文，例如在web应用程序中使用的上下文-> 'WebApplicationContext'**

根据解释可以了解到:`ApplicationContext`包含了`BeanFactory`的所有功能,并且还扩展了特性.

[官方文档]([Core Technologies (spring.io)](https://docs.spring.io/spring-framework/docs/5.2.x/spring-framework-reference/core.html#context-introduction-ctx-vs-beanfactory))还解释了程序员为什么要用`ApplicationContext`而不是`BeanFactory`:

> This section explains the differences between the `BeanFactory` and `ApplicationContext` container levels and the implications on bootstrapping.
>
> **本节解释了“BeanFactory”和“ApplicationContext”容器级别之间的差异以及对引导的影响。**
>
> You should use an `ApplicationContext` unless you have a good reason for not doing so, with `GenericApplicationContext` and its subclass `AnnotationConfigApplicationContext` as the common implementations for custom bootstrapping. These are the primary entry points to Spring’s core container for all common purposes: loading of configuration files, triggering a classpath scan, programmatically registering bean definitions and annotated classes, and (as of 5.0) registering functional bean definitions.
>
> **您应该使用 'ApplicationContext'，除非您有充分的理由不这样做，'GenericApplicationContext' 及其子类 'AnnotationConfigApplicationContext' 作为自定义引导的常见实现。这些是Spring核心容器的主要入口点，用于所有常见的目的: 加载配置文件、触发类路径扫描、以编程方式注册bean定义和带注释的类，以及 (截至5.0) 注册功能bean定义。**

下表列出了BeanFactory和ApplicationContext接口和实现提供的功能。

| Feature                                                      | `BeanFactory` | `ApplicationContext` |
| :----------------------------------------------------------- | :------------ | :------------------- |
| Bean instantiation/wiring**(Bean的实例化和属性注入)**        | Yes           | Yes                  |
| Integrated lifecycle management**(综合生命周期管理)**        | No            | Yes                  |
| Automatic `BeanPostProcessor` registration**(自动“BeanPostProcessor”注册)** | No            | Yes                  |
| Automatic `BeanFactoryPostProcessor` registration**(自动“BeanFactoryPostProcessor”注册)** | No            | Yes                  |
| Convenient `MessageSource` access (for internationalization)**(便捷的 “消息来源” 访问 (用于国际化))** | No            | Yes                  |
| Built-in `ApplicationEvent` publication mechanism**(内置的 “应用程序事件” 发布机制)** | No            | Yes                  |



## 其他的查找方式

### 根据注解来查找

OC 容器除了可以根据一个父类 / 接口来找实现类，还可以根据类上标注的注解来查找对应的 Bean 。

#### 声明bean + 注解 + 配置文件

代码结构

```
-anno
--Color
-bean
--Black
--Dog
--Red
-WithAnnoApplication
```

注释

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface Color {

}
```

配置文件省略,与上文一致,直接声明好bean目录下的几个类即可.

getBeansWithAnnotation()方法,通过传入`注解.class`来返回所有被这个注解标识的bean

```java
public class WithAnnoApplication {
    
    public static void main(String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("basic_dl/quickstart-withanno.xml");
        Map<String, Object> beans = ctx.getBeansWithAnnotation(Color.class);
        beans.forEach((beanName, bean) -> {
            System.out.println(beanName + " : " + bean.toString());
        });
    }
}
```

### 查找IOC容器中所有Bean

假如要取出当前 IOC 容器中的所有 bean ,这个时候可以用到 `ApplicationContext` 的另一个方法了：`getBeanDefinitionNames`:

```java
public class BeannamesApplication {
    
    public static void main(String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("basic_dl/quickstart-withanno.xml");
        String[] beanNames = ctx.getBeanDefinitionNames();
        Stream.of(beanNames).forEach(System.out::println);
    }
}
```

 ## 依赖查询-延迟查找

### 使用现有方案实现Bean缺失时的缺省加载

设计简单一些，准备两个 bean ：`Cat` 和 `Dog` ，但是在 xml 中只注册 `Cat` ，这样 IOC 容器中就只有 `Cat` ，没有 `Dog` 。

编写启动类。由于 Dog 没有在 IOC 容器中，所以调用 `getBean` 方法时会报 `NoSuchBeanDefinitionException` ，为了保证能在没有找到 Bean 的时候启用缺省策略，可以在 catch 块中手动创建，实现代码如下：

```java
public class ImmediatlyLookupApplication {
    
    public static void main(String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("basic_dl/quickstart-lazylookup.xml");
        Cat cat = ctx.getBean(Cat.class);
        System.out.println(cat);
        
        Dog dog;
        try {
            dog = ctx.getBean(Dog.class);
        } catch (NoSuchBeanDefinitionException e) {
           // 找不到Dog时手动创建
           dog = new Dog();
        }
        System.out.println(dog);
    }
}
```

但是这种方式是极其不优雅,不提倡的.



### 改良-获取之前先检查

作为一个容器，能获取自然就能有检查，`ApplicationContext` 中有一个方法就可以专门用来检查容器中是否有指定的 Bean ：`containsBean`

```java
 Dog dog = ctx.containsBean("dog") ? (Dog) ctx.getBean("dog") : new Dog();
```

但是，这个 `containsBean` 方法只能传 bean 的 id ，**不能查类型**，所以虽然可以改良前面的方案，但还是有问题：如果 Bean 的名不叫 dog ，叫 wangwang ，那这个方法就没有效果了.

### 改良-延迟查找

是否可以有一种机制，在获取一个 Bean 的时候，可以**先不报错，先给一个包装让我拿着，回头用的时候再拆开决定里面有还是没有**，这样是不是就省去了 IOC 容器报错的麻烦事了呢？在 SpringFramework 4.3 中引入了一个新的 API ：**`ObjectProvider`** ，它可以实现延迟查找:

```java
public class LazyLookupApplication {
    
    public static void main(String[] args) throws Exception {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("basic_dl/quickstart-lazylookup.xml");
        Cat cat = ctx.getBean(Cat.class);
        System.out.println(cat);
        // 这一行代码会报Bean没有定义 NoSuchBeanDefinitionException
        // Dog dog = ctx.getBean(Dog.class);
    
        // 这一行代码不会报错
        ObjectProvider<Dog> dogProvider = ctx.getBeanProvider(Dog.class);
    }
}
```

`ApplicationContext` 中有一个方法叫 `getBeanProvider` ，它就是返回上面说的那个**“包装”**。如果直接 `getBean` ，那如果容器中没有对应的 Bean ，就会报 `NoSuchBeanDefinitionException`；如果使用这种方式，运行 `main` 方法后发现并没有报错，只有调用 `dogProvider` 的 `getObject` ，真正要取包装里面的 Bean 时，才会报异常。所以总结下来，`ObjectProvider` 相当于**延后了 Bean 的获取时机，也延后了异常可能出现的时机**。

但是，上面的问题还没有被解决呀，调用 `getObject` 方法还是会报异常.

### 延迟查找-方案实现

`ObjectProvider` 中还有一个方法：`getIfAvailable` ，它可以在**找不到 Bean 时返回 null 而不抛出异常**。使用这个方法，就可以避免上面的问题了:

```java
  Dog dog = dogProvider.getIfAvailable();
    if (dog == null) {
        dog = new Dog();
    }
```

### ObjectProvider在jdk8的升级

随着 SpringFramework 5.0 基于 jdk8 的发布，函数式编程也被大量用于 SpringFramework 中。`ObjectProvider` 中新加了几个方法，可以使编码更佳优雅。

`ObjectProvider` 在 SpringFramework 5.0 后扩展了一个带 `Supplier` 参数的 `getIfAvailable` ，它可以在找不到 Bean 时直接用 **`Supplier`** 接口的方法返回默认实现，由此上面的代码还可以进一步简化为：

```java
Dog dog = dogProvider.getIfAvailable(() -> new Dog());
```

或者更简单的，使用方法引用：

```java
Dog dog = dogProvider.getIfAvailable(Dog::new);
```

一般情况下，取出的 Bean 都会马上或者间歇的用到，`ObjectProvider` 还提供了一个 `ifAvailable` 方法，可以在 Bean 存在时执行 `Consumer` 接口的方法：

```java
dogProvider.ifAvailable(dog -> System.out.println(dog)); // 或者使用方法引用
```

以上就是关于延迟查找的内容，这种方案可以使用，但在日常开发中可能使用的不是很多,这部分或许会在封装组件和底层时用到。