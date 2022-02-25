
## 目录
- [目录](#目录)
  - [@configuration](#configuration)
  - [@ComponentScan](#componentscan)
    - [扫描时指定排除和包含哪些注解,一定要禁用掉默认的扫描规则](#扫描时指定排除和包含哪些注解一定要禁用掉默认的扫描规则)
    - [可以按照类型排除或者包含要扫描的组件:](#可以按照类型排除或者包含要扫描的组件)
    - [自定义规则，指定要包含或者要排除的组件:](#自定义规则指定要包含或者要排除的组件)
  - [@Scope](#scope)
  - [@Lazy懒加载](#lazy懒加载)
  - [@Import](#import)
    - [自定义选择器导入组件](#自定义选择器导入组件)
    - [ImportBeanDefinitionRegistrar 手动注册Bean到容器](#importbeandefinitionregistrar-手动注册bean到容器)
  - [属性赋值](#属性赋值)
    - [@Value](#value)
    - [@PropertySource配置的用法](#propertysource配置的用法)
  - [自动装配](#自动装配)
    - [@Autowired和@Qualifier](#autowired和qualifier)
    - [@Primary](#primary)
### @configuration
`@Configuration`标注在类上，相当于把该类作为spring的xml配置文件中的`<beans>`，作用为：配置spring容器(应用上下文)
```java
package com.dxz.demo.configuration;
import org.springframework.context.annotation.Configuration;
@Configuration
public class TestConfiguration {
    public TestConfiguration() {
        System.out.println("TestConfiguration容器启动初始化。。。");
    }
}
```
相当于

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xmlns:context="http://www.springframework.org/schema/context" xmlns:jdbc="http://www.springframework.org/schema/jdbc"  
xmlns:jee="http://www.springframework.org/schema/jee" xmlns:tx="http://www.springframework.org/schema/tx"
xmlns:util="http://www.springframework.org/schema/util" xmlns:task="http://www.springframework.org/schema/task" xsi:schemaLocation="
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
            http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-4.0.xsd
            http://www.springframework.org/schema/jee http://www.springframework.org/schema/jee/spring-jee-4.0.xsd
            http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd
            http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-4.0.xsd" default-lazy-init="false"/>
```

### @ComponentScan
包扫描，只要标注了@Controller,@Service,@Repository,@Component注解都会被扫描进来
#### 扫描时指定排除和包含哪些注解,一定要禁用掉默认的扫描规则
```java
@Configuration //标注当前是一个配置类
@ComponentScan(value = "com.controller",includeFilters = {
        //按照注解排除，排除掉Controller注解
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = Controller.class)
},           //这里禁用掉默认的扫描规则
        useDefaultFilters = false)
public class MyConfig
{ }
```
```java
@Configuration //标注当前是一个配置类
@ComponentScan(value = "com.controller",excludeFilters = {
        //按照注解排除，排除掉Controller注解
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = Controller.class)
},           //这里禁用掉默认的扫描规则
        useDefaultFilters = false)
public class MyConfig
{ }
```
#### 可以按照类型排除或者包含要扫描的组件:
```java
@Configuration //标注当前是一个配置类
@ComponentScan(value = "com.controller",includeFilters= {
        //按照注解包含，排除掉Controller注解
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = Controller.class),
                                        //按照类型包含要扫描的组件
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,classes = peoController.class)
},           //这里禁用掉默认的扫描规则
        useDefaultFilters = false)
public class MyConfig
{ }
```
#### 自定义规则，指定要包含或者要排除的组件:
继承TypeFilter接口，重写扫描匹配方法:
```java
public class MyTypeFilter implements TypeFilter
{
   //MetadataReader:读取到当前正在扫描的类的相关信息
    //MetadataReaderFactory： 可以获取到其他任何类的信息
    public boolean match(MetadataReader metadataReader,
                         MetadataReaderFactory metadataReaderFactory) throws IOException {
        //获取当前类注解信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        //获取当前正在扫描的类的类信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        //获取当前类资源(类路径)
        Resource resource = metadataReader.getResource();

        String className = classMetadata.getClassName();
        //如果当前类名包含peo，则不扫描
        if(className.contains("peo"))
            return false;
        //否则扫描
        return true;
    }
}
```
配置类：
```java
@Configuration //标注当前是一个配置类
@ComponentScan(value = "com.controller",includeFilters= {
        //按照自定义规则
        @ComponentScan.Filter(type = FilterType.CUSTOM,classes = {MyTypeFilter.class})

},           //这里禁用掉默认的扫描规则
        useDefaultFilters = false)
public class MyConfig
{ }
```
### @Scope
```java
@Configuration //标注当前是一个配置类
@ComponentScan(value = "com.controller")
public class MyConfig
{
    @Scope("prototype")
    @Bean(value = "dhy")//指定id
    public people getPeople()
    {
        return new people("大忽悠",18);
    }
}
```
**prototype:多实例的**
ioc容器启动并不会去调用方法创建对象放在容器中，而是每次获取的时候才会调用方法创建对象
**singleton:单实例的(默认值)**
ioc容器启动时会调用方法创建对象放到ioc容器中，以后每一次获取就是直接从容器(map.get())中拿
**request:同一次请求创建一个实例**
**session:同一个session创建一个实例**

### @Lazy懒加载
```java
@Configuration //标注当前是一个配置类
@ComponentScan(value = "com.controller")
public class MyConfig
{
    @Lazy
    @Bean(value = "dhy")//指定id
    public people getPeople()
    {
        return new people("大忽悠",18);
    }
}
```
**懒加载:**
单实例bean:默认在容器启动的时候创建对象
懒加载：容器启动不创建对象，第一次使用(获取)Bean创建对象，并初始化
### @Import
@Import导入组件,id默认是全类名

配置类：
``` java
@Configuration //标注当前是一个配置类
//这里没有进行包扫描，所以在不进行导入的情况下peoController是不会被注册到容器中的
@Import({peoController.class,people.class})
public class MyConfig
{
    @Conditional({WindowsCondition.class})
    @Bean
    public people getWindows()
    {
        return new people("windows系统",18);
    }

    @Conditional(LinuxCondition.class)
    @Bean
    public people getLinux()
    {
        return new people("Linux系统",18);
    }
}
```
peoController:
```java
@Controller("大忽悠")//这里同样可以起一个别名
public class peoController
{}
```
#### 自定义选择器导入组件
MyImportSelector:

```java
//自定义逻辑返回需要导入的组件
public class MyImportSelector implements ImportSelector
{
    //返回值就是要导入到容器中的组件的全类名
     //AnnotationMetadata:当前标注@Import注解的类的所有信息
    public String[] selectImports(AnnotationMetadata annotationMetadata) {
    //方法不要返回null
        return new String[]{"com.Pojo.rea", "com.Pojo.ha"};
    }
}
```
MyConfig:
```java
@Configuration //标注当前是一个配置类
//加入自定义导入选择器
@Import({peoController.class,people.class,MyImportSelector.class})
public class MyConfig
{
    @Conditional({WindowsCondition.class})
    @Bean
    public people getWindows()
    {
        return new people("windows系统",18);
    }

    @Conditional(LinuxCondition.class)
    @Bean
    public people getLinux()
    {
        return new people("Linux系统",18);
    }
}
```
#### ImportBeanDefinitionRegistrar 手动注册Bean到容器
MyImportBeanDefinitionRegister：
```java
public class MyImportBeanDefinitionRegister implements ImportBeanDefinitionRegistrar
{
/*
* AnnotationMetadata:当前类的注解信息
* BeanDefinitionRegistry：BeanDefinition注册类
* 把所有需要添加到容器中的Bean，调用
* BeanDefinitionRegistry.registerBeanDefinition手工注册起来
* */
    public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {
        boolean ha = beanDefinitionRegistry.containsBeanDefinition("com.Pojo.rea");
        boolean rea = beanDefinitionRegistry.containsBeanDefinition("com.Pojo.ha");
        if(ha&&rea)
        {
            //指定Bean定义信息(Bean的类型，Bean的作用域...)
            RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(dhy.class);
            //指定Bean名
            beanDefinitionRegistry.registerBeanDefinition("手动组成的Bean",rootBeanDefinition);
        }
    }
}
```
MyConfig：
```java
@Configuration //标注当前是一个配置类
//加入自定义导入选择器
@Import({people.class,MyImportSelector.class,MyImportBeanDefinitionRegister.class})
public class MyConfig
{
    @Conditional({WindowsCondition.class})
    @Bean
    public people getWindows()
    {
        return new people("windows系统",18);
    }

    @Conditional(LinuxCondition.class)
    @Bean
    public people getLinux()
    {
        return new people("Linux系统",18);
    }
}
```
### 属性赋值
#### @Value
@Value注解里面参数可填内容:
1. 基本数值
2. 可以写SPEL表达式==>#{}
3. 可以写${},取出配置文件中的值(在运行环境变量里面的值)

```java
@Data
@Component
public class Dhy
{
    @Value("大忽悠")
    String name;
    @Value("#{3*6}")
    String age;
    public Dhy()
    {
        System.out.println("Dhy创建中...");
    }
}
```
* @Value也可以加在方法的参数上，从配置文件中取出值，赋值给参数

#### @PropertySource配置的用法

* 加载指定的属性文件(*.properties)到 Spring 的 Environment 中。可以配合 @Value 和
@ConfigurationProperties 使用。

* @PropertySource 和 @Value组合使用，可以将自定义属性文件中的属性变量值注入到当前类的使用@Value注解的成员变量中。

* @PropertySource 和 @ConfigurationProperties组合使用，可以将属性文件与一个Java类绑定，将属性文件中的变量值注入到该Java类的成员变量中。

@PropertySource读取外部配置文件中的k/v保存到运行的环境变量中,加载完外部配置文件中的值后使用${}取出配置文件中的值
```java
@Data
@Component
public class Dhy
{
    @Value("${name}")
    String name;
    @Value("#{3*6}")
    String age;
    public Dhy()
    {
        System.out.println("Dhy创建中...");
    }
}
```
获取环境变量中的值⇒ ioc.getEnvironment.getProperty
```java
public class Main
{
    //传入的是配置类的位置，一开始是加载配置类，之前是加载配置文件的位置
  private  ApplicationContext ioc= new AnnotationConfigApplicationContext(MyConfig.class);
  @Test
    public void test()
    {
        Dhy bean = ioc.getBean(Dhy.class);
        System.out.println(bean);
        //获取环境变量中的保存的name
        Environment env = ioc.getEnvironment();
        String name = env.getProperty("name");
        System.out.println(name);
    }
}
```
### 自动装配
#### @Autowired和@Qualifier
@Autowired:自动注入：

1. 默认优先按照类型去容器中寻找对应的组件
2. 如果找到多个相同类型的组件，再将属性的名称作为组件的id去容器中查找
3. @Qualifier(“book”):使用@Qualifier指定需要装配的组件的id，而不是使用属性名
4. 自动装配默认一定要将属性赋值好，没有就会报错

可以使用@Autowired（required=false）;

#### @Primary
@Primary:让spring进行自动装配的时候，默认使用首选的bean也可以继续使用@Qualifier指定需要装配的bean的名字

