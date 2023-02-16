# Spring.IOC - IOC是怎么来的?

## 概念:

### 类之间的依赖关系

#### 紧耦合

```java
public class A{
 		public static B getB(){
     		return new B(); 
    }
}
```

当源码中缺少某个Java类(例如上面的B类),导致编译都无法通过的,这种现象就可以描述为**"A强依赖B"**,也就是平时说的**"紧耦合"**.

```java
public class A{
 		public static B getB(){
     		return (B) Class.forName("com.xxx.B").newInstance();
    }
}
```

#### 弱依赖

使用反射之后，错误现象不再是在编译器就出现，而是在工程启动后，由于 `A` 要构造 `B` 时确实还没有该类，所以抛出 `ClassNotFoundException` 异常。这样 **`A` 对 `B` 的依赖程度**就相当于**降低**了，也就可以算作“**弱依赖**”了。



### 硬编码

如上面的将**类的全限定名**写死在A的源码中,导致每次更改都需要**重新编译工程才可以正常运行**.



### 引入外部化配置文件

可以利用`I/O`来实现文件存储配置,然后`A`被初始化的时候可以去读取配置文件,这样就不会出现硬编码的现象了.



### 外部化配置

对于可能会变化的配置,属性等.通常不会直接硬编码在源码中,而是抽取为一些配置文件的形式(properties,xml,json,yml等),配置程序对配置文件的加载和解析,从而达到动态配置,降低配置耦合的目的.



### 引入缓存

如果对于没有必要创建多个对象的组件,利用一种机制保证整个程序运行过程中只存在一个对象,那么可以大大的减少资源消耗.

```java
public class BeanFactory {
    private static Properties properties;
    //缓存,保存已经创建好的对象
    private static Map<String, Object> beanMap = new HashMap();

    public static Object getBean(String beanName) {

        if (!beanMap.containsKey(beanName)) {
            synchronized (BeanFactory.class) {
                if (!beanName.contatinsKey(beanName)) {
                    // 过了双检锁，证明确实没有，可以执行反射创建
                    try {
                        Class<?> beanClazz = Class.forName(properties.getProperty(beanName));
                        Object bean = beanClazz.newInstance();
                        // 反射创建后放入缓存再返回
                        beanMap.put(beanName, bean);
                    } catch (ClassNotFoundException e) {
                        throw new RuntimeException("BeanFactory have not [" + beanName + "] bean!", e);
                    } catch (IllegalAccessException | InstantiationException e) {
                        throw new RuntimeException("[" + beanName + "] instantiation error!", e);
                    }
                }
            }
        }
        return beanMap.get(beanName);
    }
}
```



### IOC的思想引入

```java
private B b = (B)BeanFactory.getBean("b");
```

不再是自己去声明，而是**将获取对象的方式交给了 `BeanFactory`** 。这种**将控制权交给别人**的思想，就可以称作：**控制反转（ Inverse of Control , IOC ）**。而 `BeanFactory` 根据指定的 `beanName` 去获取和创建对象的过程，就可以称作：**依赖查找（ Dependency Lookup , DL ）**。