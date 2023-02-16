# Spring.IOC-IOC基础（注解驱动与组件扫描）

## 注解驱动IOC容器

在 xml 驱动的 IOC 容器中，使用的是 `ClassPathXmlApplicationContext` ，它对应的是类路径下的 xml 驱动。

对于注解配置的驱动，使用的是 `AnnotationConfigApplicationContext` 。

### 注解驱动IOC的依赖查找

注解驱动需要配置类,一个配置类就可以类似的理解为一个xml文件.配置类没有特殊要求,只需要在类上标注一个注解`@Configuration`即可.

```java
@Configuration
public class QuickstartConfiguration {

}
```

在配置类中声明Bean,是通过`@Bean`注解

```java
@Bean
public Person person() {
    return new Person();
}
```

以上这种方式,可以解释为: 向IOC容器注册一个类型为`Person`,id为`person`的Bean.方法的返回值代表注册的类型,方法名代表Bean的id.也可以在`@Bean`注解上显示的声明Bean的id,只不过在这称之为`name`:

```java
@Bean(name = "aaa") // 4.3.3之后可以直接写value
public Person person() {
    return new Person();
}
```

#### 启动类初始化注解IOC容器

使用`AnnotationConfigApplicationContext`来驱动注解IOC容器,发现是可以输出`Person`的.

```java
public class AnnotationConfigApplication {
    
    public static void main(String[] args) throws Exception {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(QuickstartConfiguration.class);
        Person person = ctx.getBean(Person.class);
        System.out.println(person);
    }
}
```

### 注解驱动IOC的依赖注入

只做演示,实际开发中不建议这么注入.

```java
@Bean
public Person person() {
    Person person = new Person();
    person.setName("person");
    person.setAge(123);
    return person;
}
```

```java
@Bean
public Cat cat() {
    Cat cat = new Cat();
    cat.setName("test-cat-anno");
    // 直接拿上面的person()方法作为返回值即可，相当于ref
    cat.setMaster(person());
    return cat;
}
```

### 注解IOC容器的其他用法

`AnnotationConfigApplicationContext` 的构造方法，可以发现它还有一个方法，是传入一组 `basePackage` ，翻译过来是 “根包” 的意思，这就涉及到了：**组件注册与扫描**。

## 组件注册和扫描

###  一切组件注册的根源：@Component

在类上标注`@Component`注解,代表该类会被注册到IOC容器中作为一个Bean.

```java
@Component
public class Person {
    
}
```

可以跟`@Bean`一样直接显示声明 **value ** (id / name ) 属性：

```java
@Component("aaa")
public class Person { }
```

注意⚠️: 如果不指定 Bean 的名称，它的默认规则是 **“类名的首字母小写”**（例如 `Person` 的默认名称是 `person` ，`DepartmentServiceImpl` 的默认名称是 `departmentServiceImpl` ）。

### 组件扫描

只声明了组件，在写配置类时如果还是只写 `@Configuration` 注解，随后启动 IOC 容器，那它是感知不到有 `@Component` 存在的，一定会报 `NoSuchBeanDefinitionException` 。所以需要配合新的注解`@ComponentScan`,

在配置类上额外标注一个 `@ComponentScan` ，并指定要扫描的路径，就可以**扫描指定路径包及子包下的所有 `@Component` 组件**,如果不指定扫描路径，则**默认扫描本类所在包及子包下的所有 `@Component` 组件**.

```java
@Configuration
@ComponentScan("com.linkedbear.spring.annotation.c_scan.bean")
public class ComponentScanConfiguration {
    
}
```

#### 不使用@ComponentScan的组件扫描

如果不写 `@ComponentScan` ，也是可以做到组件扫描的。在 `AnnotationConfigApplicationContext` 的构造方法中有一个类型为 String 可变参数的构造方法：

```java
ApplicationContext ctx = new AnnotationConfigApplicationContext("com.linkedbear.spring.annotation.c_scan.bean");
```

这样声明好要扫描的包，也是可以直接扫描到那些标注了 `@Component` 的 Bean 的。

#### xml中启用组件扫描

组件扫描可不是注解驱动 IOC 的专利，对于 xml 驱动的 IOC 同样可以启用组件扫描，它只需要在 xml 中声明一个标签即可：

```xml
<context:component-scan base-package="com.linkedbear.spring.annotation.c_scan.bean"/>
<!-- 注意标签是package，不是packages，代表一个标签只能声明一个根包 -->
```

之后使用 `ClassPathXmlApplicationContext` 驱动，也是可以获取到 `Person` 的。

SpringFramework 为了配合 Web 开发时的三层架构，它额外提供了三个注解：`@Controller` 、`@Service` 、`@Repository` ，分别代表表现层、业务层、持久层。这三个注解的作用与 `@Component` 完全一致，其实它们的底层也就是 `@Component` ：

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller { ... }
```

有了这几个注解，在进行符合三层架构的开发时，对于那些 ServiceImpl ，就可以直接标注 `@Service` 注解，而不用一个一个的写 `<bean>` 标签或者 `@Bean` 注解了。

### @Configuration也是@Component

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component //也被标注了`@Component`注解,所以也会被视为Bean.被注册到IOC容器中
public @interface Configuration { ... }
```

## 注解驱动与xml驱动互通

如果一个应用中，既有注解配置，又有 xml 配置，这个时候就需要由一方引入另一方了。两种方式咱都介绍一下。

### 3.1 xml引入注解

在 xml 中要引入注解配置，需要开启注解配置，同时注册对应的配置类：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd 
        http://www.springframework.org/schema/context 
        https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 开启注解配置 -->
    <context:annotation-config />
    <bean class="com.linkedbear.spring.annotation.d_importxml.config.AnnotationConfigConfiguration"/>
</beans>
```

### 3.2 注解引入xml

在注解配置中引入 xml ，需要在配置类上标注 `@ImportResource` 注解，并声明配置文件的路径：

```java
@Configuration
@ImportResource("classpath:annotation/beans.xml")
public class ImportXmlAnnotationConfiguration {
    
}
```