# Spring.IOC-入门IOC

## 依赖查找

### DL-byName

配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.linkedbear.spring.basic_dl.a_quickstart_byname.bean.Person"></bean>
</beans>
```

测试代码:通过bean的名称`person`去查询依赖
```java
public static void main(String[] args) throws Exception {
    BeanFactory factory = new ClassPathXmlApplicationContext("basic_dl/quickstart-byname.xml");
    Person person = (Person) factory.getBean("person");
    System.out.println(person);
}
```



### DL-byType

配置文件(省略文件头)

```xml
<bean class="com.linkedbear.spring.basic_dl.b_bytype.bean.Person"></bean>
```

测试代码:通过bean的类型`Person.class`去查询依赖

```java
public static void main(String[] args) throws Exception {
    BeanFactory factory = new ClassPathXmlApplicationContext("basic_dl/quickstart-bytype.xml");
    Person person = factory.getBean(Person.class);
    System.out.println(person);
}
```



## 依赖注入

### 简单属性值注入

声明2个类

```java
public class Person {
    private String name;
    private Integer age;
    // getter and setter ......
}

public class Cat {
    private String name;
    private Person master;
    // getter and setter ......
}
```

配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="person" class="com.linkedbear.spring.basic_di.a_quickstart_set.bean.Person"></bean>

    <bean id="cat" class="com.linkedbear.spring.basic_di.a_quickstart_set.bean.Cat"></bean>
</beans>
```

测试代码:输出的person和cat的所有属性都是null

```java
public class QuickstartInjectBySetXmlApplication {
    public static void main(String[] args) throws Exception {
        BeanFactory beanFactory = new ClassPathXmlApplicationContext("basic_di/inject-set.xml");
        Person person = beanFactory.getBean(Person.class);
        System.out.println(person);
        
        Cat cat = beanFactory.getBean(Cat.class);
        System.out.println(cat);
    }
}
```

修改配置文件,给属性赋值

```xml
<bean id="person" class="com.linkedbear.spring.basic_di.a_quickstart_set.bean.Person">
    <property name="name" value="test-person-byset"/>
    <property name="age" value="18"/>
</bean>
```



### 关联Bean赋值

`property` 标签，除了可以声明 `value` 之外，还可以声明另外一个属性：**`ref`** ，它代表**要关联赋值的 Bean 的 id** 。 由此，对于 cat 中的 master 属性，可以有如下赋值方法：

```xml
<bean id="cat" class="com.linkedbear.spring.basic_di.a_quickstart_set.bean.Cat">
    <property name="name" value="test-cat"/>
    <!-- ref引用上面的person对象 -->
    <property name="master" ref="person"/>
</bean>
```



## 依赖查找与依赖注入的对比

-   作用目标不同
    -   依赖注入的作用目标通常是类成员
    -   依赖查找的作用目标可以是方法体内，也可以是方法体外
-   实现方式不同
    -   依赖注入通常借助一个上下文被动的接收
    -   依赖查找通常主动使用上下文搜索
