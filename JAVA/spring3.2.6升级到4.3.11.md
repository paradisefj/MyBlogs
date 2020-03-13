# JAVA

## spring3.2.6升级到4.3.11

1. 修改spring相关的版本到4.3.11
    包括spring 和spring mvc 相关的jar包

2. 修改xml配置中的xsd文件的版本号或者直接去掉版本号
    例如: 
   由 `xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd`
改为
    `xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd`

3. JSON解析器
    `dispatch-servlet.xml`中JSON解析器
    由`org.springframework.http.converter.json.MappingJacksonHttpMessageConverter` 
    改为`rg.springframework.http.converter.json.MappingJackson2HttpMessageConverter`
    `<bean id="jackson_hmc" class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"/>`

4. DAO
    dao中有使用类`import org.springframework.jdbc.core.simple.ParameterizedBeanPropertyRowMapper;`
    修改为`import org.springframework.jdbc.core.BeanPropertyRowMapper`

5. dispacther-servlet 中内容协商视图解析器修改
```
<bean
            class="org.springframework.web.servlet.view.ContentNegotiationViewResolver">
        <property name="defaultContentType" value="text/html" />
            class="org.springframework.web.servlet.view.ContentNegotiatingViewResolver">
        <property name="mediaTypes">
             <map>
                <!-- 告诉视图解析器，返回的类型为json格式 -->
                <entry key="html" value="text/html;charset=UTF-8"/>
                <entry key="json" value="application/json"/>
             </map>
         </property>
        <property name="viewResolvers">
            <list>
                <bean
                        class="org.springframework.web.servlet.view.InternalResourceViewResolver">
                    <property name="viewClass"
                              value="org.springframework.web.servlet.view.JstlView"/>
                    <property name="prefix" value="/WEB-INF/pages/"/>
                    <property name="suffix" value=".jsp"/>
                </bean>
            </list>
        </property>
        <property name="defaultViews">
            <list>
                <bean
                        class="org.springframework.web.servlet.view.json.MappingJacksonJsonView"/>
            </list>
        </property>
 </bean>
``` 
改为 

```
<bean
        class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
    <property name="favorPathExtension" value="true" />
    <property name="favorParameter" value="true" />
    <property name="ignoreAcceptHeader" value="true"></property>
    <property name="defaultContentType" value="text/html" />
    <property name="mediaTypes">
        <map>
            <!-- 告诉视图解析器，返回的类型为json格式 -->
            <entry key="json" value="application/json" />
            <entry key="xml" value="application/xml" />
            <entry key="htm" value="text/htm" />
            <entry key="file" value="application/octet-stream" />
            <entry key="image" value="image/*" />
        </map>
    </property>
</bean>
```
6. dispatch-servlet 中添加注解驱动配置
`<mvc:annotation-driven/>`
> 非常重要，因为缺少该配置，导致spring4新增的注解无法使用

7. cors 由自己写的实现改为注解配置
dispatch-servlet.xml 中添加cors配置
```
<!--跨域-->
<mvc:cors>
    <mvc:mapping path="/**" allowed-methods="*"/>
</mvc:cors>
```
