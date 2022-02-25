### SpringBoot 项目的一切都要从 `@SpringBootApplication` 这个注解开始说起。

`@SpringBootApplication` 标注在某个类上说明：
- 这个类是 SpringBoot 的主配置类。
- SpringBoot 就应该运行这个类的 main 方法来启动 SpringBoot 应用。

`@SpringBootApplication`的定义如下：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {...}
```
可以看出这个注解主要由`@SpringBootConfiguration`、`@EnableAutoConfiguration`和`@ComponentScan`三个注解组合而成。
- `@SpringBootConfiguration`：该注解表示这是一个 SpringBoot 的配置类，等价于`@Configuration`注解
- `@EnableAutoConfiguration`：为这个类开启自动配置的，是自动配置的核心
- `@ComponentScan`：开启组件扫描

下面着重解释`@EnableAutoConfiguration`

### `@EnableAutoConfiguration`

`@EnableAutoConfiguration`的内容如下：

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {...}
```
可以看到此注解主要由`@AutoConfigurationPackage`和`@Import`两个注解组成。

`@AutoConfigurationPackage`：自动配置包，源码如下：

```JAVA
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({Registrar.class})
public @interface AutoConfigurationPackage {...}
```
其内部也是一个`@Import`注解，import了一个`Registrar`组件。在 `Registrar` 类中的 `registerBeanDefinitions` 方法上打上断点，可以看到返回了一个包名，该包名其实就是主配置类所在的包。

即`@AutoConfigurationPackage`注解就是将主配置类（@SpringBootConfiguration标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器中。**所以说，默认情况下主配置类包及子包以外的组件，Spring 容器是扫描不到的。**

### `@Import({AutoConfigurationImportSelector.class})`
该注解给当前配置类导入另外的 N 个自动配置类。

#### 配置类导入规则
`AutoConfigurationImportSelector`的`selectImports()`方法 就是用来返回需要导入的组件的全类名数组的，那么如何得到这些数组呢？

![](./img/屏幕截图%202022-02-24%20214836.png)

`selectImports()`方法调用了`getAutoConfigurationEntry()`方法

![](./img/屏幕截图%202022-02-24%20215203.png)

`getAutoConfigurationEntry()`方法又调用了`getCandidateConfigurations()`方法

![](./img/屏幕截图%202022-02-24%20215354.png)

`getCandidateConfigurations()`方法调用了`loadFactoryNames()`方法，`loadFactoryNames()`方法传入了`getSpringFactoriesLoaderFactoryClass()`这个方法，这个方法的返回值是一个`EnableAutoConfiguration.class`。

`loadFactoryNames()`中关键的三步：
1. 从当前项目的类路径中获取所有 `META-INF/spring.factories` 这个文件下的信息.
2. 将上面获取到的信息封装成一个 Map 返回。
3. 从返回的 Map 中通过刚才传入的 `EnableAutoConfiguration.class` 参数，获取该 key 下的所有值。

![](./img/屏幕截图%202022-02-24%20220410.png)

#### `META-INF/spring.factories`探究

![](./img/屏幕截图%202022-02-24%20220747.png)

在`META-INF/spring.factories`中可以看到 `EnableAutoConfiguration` 下面有很多类，这些就是我们项目进行自动配置的类。

一句话：将类路径下 `META-INF/spring.factories` 里面配置的所有 `EnableAutoConfiguration` 的值加入到 `Spring` 容器中。

#### `HttpEncodingAutoConfiguration`
>通过上面方式，所有的自动配置类就被导进主配置类中了。但是这么多的配置类，明显有很多自动配置我们平常是没有使用到的，没理由全部都生效吧。
>

以 `HttpEncodingAutoConfiguration`为例来看一个自动配置类是怎么工作的
```java
@Configuration 
@EnableConfigurationProperties({HttpProperties.class}) 
@ConditionalOnWebApplication( 
type = Type.SERVLET 
) 
@ConditionalOnClass({CharacterEncodingFilter.class}) 
@ConditionalOnProperty( 
prefix = "spring.http.encoding", 
value = {"enabled"}, 
matchIfMissing = true 
) 
public class HttpEncodingAutoConfiguration {...}
```
- `@Configuration`：标记为配置类。
- `@ConditionalOnClass`：指定的类（依赖）存在才生效。
- `@ConditionalOnProperty`：主配置文件中存在指定的属性才生效。
- `@EnableConfigurationProperties({HttpProperties.class})`：启动指定类的`ConfigurationProperties`功能；将配置文件中对应的值和 `HttpProperties` 绑定起来；并把 `HttpProperties` 加入到 IOC 容器中。

因为` @EnableConfigurationProperties({HttpProperties.class})` 把配置文件中的配置项与当前 `HttpProperties` 类绑定上了。然后在` HttpEncodingAutoConfiguration` 中又引用了 `HttpProperties` ，所以最后就能在 `HttpEncodingAutoConfiguration` 中使用配置文件中的值了。最终通过 `@Bean` 和一些条件判断往容器中添加组件，实现自动配置。（当然该`Bean`中属性值是从 `HttpProperties` 中获取）

#### `HttpProperties`

>`HttpProperties` 通过 `@ConfigurationProperties` 注解将配置文件与自身属性绑定。
>

所有在配置文件中能配置的属性都是在 xxxProperties 类中封装着；配置文件能配置什么就可以参照某个功能对应的这个属性类。

```java
@ConfigurationProperties( 
prefix = "spring.http" 
)// 从配置文件中获取指定的值和bean的属性进行绑定 
public class HttpProperties {} 
```

**小结**
1. SpringBoot启动会加载大量的自动配置类。
2. 我们看需要的功能有没有SpringBoot默认写好的自动配置类。
3. 我们再来看这个自动配置类中到底配置了那些组件（只要我们要用的组件有，我们就不需要再来配置了）。
4. 给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这些属性的值。
* xxxAutoConfiguration：自动配置类给容器中添加组件。
* xxxProperties：封装配置文件中相关属性。



作者：你在我家门口
链接：https://juejin.cn/post/6844903849178873870
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

