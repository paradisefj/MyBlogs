### 注入Bean

@Bean 代表将方法返回的POJO对象装配到IoC容器中，属性name定义该Bean的名称，如果没有配置它，这将方法名称作为Bean的名称。
```
@Bean(name = "user")
public User initUser() {
    return new User();
}
```
上面例子中，会将一个名为user的Bean装配到Ioc容器中，如果没有`(name = "user")`，则将一个名为initUser的Bean装配到Ioc容器中。

@Component("user") 表明这个类将被Spring IoC 容器扫描装配，其中配置名称user则作为Bean的名称，如果不配置该名称，Ioc容器会将类名第一个字母小写后的名称作为Bean的名称。

### 获取Bean

@Autowired 根据属性的类型(by type) 找到对应的Bean进行注入。他会根据类型找到对应的Bean，如果对应类型的Bean不是唯一的，则他会根据其属性名称和Bean的名称进行匹配。如果匹配的上，就会使用该Bean；无法匹配，就会抛出异常。除了标注属性外，还可以标注方法和标注带有参数的构造方法；
```
@Component
public class BusinessPerson {
    private Animal animal;
    public BusinessPerson(@Autoweired @Qualified("dog") Animal animal) {
        this.animal = animal;
    }
}
```

@Primary 告诉Ioc容器，当发现有多个同类型的Bean时，优先使用被其注解的Bean。

@Qualifier("dog") 和@Autowired 组合在一起，通过类型和名称去查找Bean进行注入。

### 生命周期

1. Spring通过配置，如@ComponentScan定义的扫描路径去找到带有@Component的类，这个过程就是一个资源定位的过程。
2. 一旦找到资源，它就开始解析，并将定义的信息保存起来。注意，此时还没有初始化Bean，也就没有Bean的实例，它有的仅仅是Bean的定义。
3. 然后就会把Bean定义发布到Spring IoC容器中。此时IoC容器中也只有Bean的定义，还没有Bean的实例生成。
4. 在默认情况下，Spring会继续完成Bean的实例化和依赖注入，这样从IoC容器中就可以得到一个依赖注入完成的Bean。
5. 有时候希望会在取出Bean的时候完成初始化和依赖注入，通过在ComponentScan中配置项lazyInit=true来实现。

### 条件装配Bean

@Conditional 需要和接口Condition配合

```
@Bean(name="dataSource")
@Conditional(DatabaseConditional.class)
public DataSource dataSource(@Value("${database.driverName}") String driverName) {
    this.driverName = driverName;
}
```
```
public class DatabaseConditional implements Condition {
    /** 数据库装配条件，true 装配Bean,否则不装配 */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment env = context.getEnviroment();
        return env.containsProperty("database.driverName");
    }
}
```

### Bean的作用域

BeanFactory中有 isSingleton 和 isPrototype 两个方法，其中isSingleton返回true，则是单例存在，isPrototype 返回true，每次获取Bean的时候，IoC容器都会创建一个Bean。

<table>
    <thead>
        <th>作用域类型</th>
        <th>使用范围</th>
        <th>描述</th>
    </thead>
    <tbody>
        <tr>
            <td>singleton</td>
            <td>所有Spring应用</td>
            <td>默认值</td>
        </tr>
        <tr>
            <td>prototype</td>
            <td>所有Spring应用</td>
            <td>每次获取Bean的时候，IoC容器都会创建一个Bean。</td>
        </tr>
        <tr>
            <td>session</td>
            <td>Spring Web应用</td>
            <td>HTTP回话</td>
        </tr>
        <tr>
            <td>application</td>
            <td>Spring Web应用</td>
            <td>Web工程生命周期</td>
        </tr>
        <tr>
            <td>request</td>
            <td>Spring Web应用</td>
            <td>web工程单次请求</td>
        </tr>
        <tr>
            <td>globalSession</td>
            <td>Spring Web应用</td>
        </tr>
    </tbody>
</table>

### 引入XML配置Bean

@ImportResource






