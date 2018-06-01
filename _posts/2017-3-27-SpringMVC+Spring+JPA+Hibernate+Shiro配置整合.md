---
layout: post
title: SpringMVC+Spring+JPA+Hibernate+Shiro配置整合
description: SpringMVC+Spring+JPA+Hibernate+Shiro配置整合
category: javaweb
---
# SpringMVC+Spring+JPA+Hibernate+Shiro配置整合
* 开发工具：IntelliJ IDEA
* 构建工具：Gradle
* 语言：java
* 框架：SpringMVC+Spring+JPA+Hibernate+Shiro

## Gradle+Maven仓库新建WEB项目
### Gradle构建web项目
New->Project...->选择Project SDK->填写项目名称等即可，大致流程如下图：
![1](https://gaoyuyue.github.io/picture/2017-3-27_2.png)
![2](https://gaoyuyue.github.io/picture/2017-3-27_3.png)
![3](https://gaoyuyue.github.io/picture/2017-3-27_4.png)
![4](https://gaoyuyue.github.io/picture/2017-3-27_5.png)
### Grale项目设置
打开项目，我们将看到根目录下有一个以build.gradle命名的文件，这个文件是Gradle项目的配置文件，接下来我们将添加配置到build.gradle文件。具体配置如下：

```
group 'yourGroupName'
version '1.0-SNAPSHOT'
buildscript {
    repositories {
        mavenLocal()
        maven { url "http://maven.aliyun.com/nexus/content/groups/public/" }
        jcenter()
        mavenCentral()
        maven { url "http://repo.spring.io/release" }
        maven { url "https://repo.spring.io/libs-snapshot" }
    }
    dependencies {
    }
}
```
下面为引入插件配置
```
apply plugin: "java"
apply plugin: "war"
apply plugin: "jacoco"
apply plugin: "idea"

war {
    baseName = "yourProgectName"
}

configurations {
    providedRuntime
}

sourceCompatibility = 1.8
targetCompatibility = 1.8
def SpringVersion = "4.3.4.RELEASE"
def HibernateVersion = "5.2.4.Final"
def ShiroVersion = "1.3.2"
def JacksonVersion = "2.8.4"
```
下面为maven仓库地址配置，这里我们采用国内的阿里云maven仓库
```
repositories {
    mavenLocal()
    maven { url "http://maven.aliyun.com/nexus/content/groups/public/" }
    jcenter()
    mavenCentral()
    maven { url "http://repo.spring.io/release" }
    maven { url "https://repo.spring.io/libs-snapshot" }
}
```
下面为引入依赖包配置
```
dependencies {
    compile(

            // spring framework
            "org.springframework:spring-context:${SpringVersion}",
            "org.springframework:spring-context-support:${SpringVersion}",//启用ehcache缓存
            "org.springframework:spring-web:${SpringVersion}",
            "org.springframework:spring-webmvc:${SpringVersion}",
            "org.springframework:spring-test:${SpringVersion}",
            "org.springframework:spring-beans:${SpringVersion}",
            "org.springframework:spring-expression:${SpringVersion}",
            "org.springframework:spring-aop:${SpringVersion}",
            "org.springframework:spring-aspects:${SpringVersion}",
            "org.springframework:spring-core:${SpringVersion}",
            "org.springframework:spring-instrument:${SpringVersion}",
            "org.springframework:spring-orm:${SpringVersion}",

            "javax.servlet:javax.servlet-api:3.1.0",


            //Spring-Session
            "org.springframework.session:spring-session:1.2.2.RELEASE",
            "org.springframework.data:spring-data-redis:1.7.4.RELEASE",

            //jedis
            "redis.clients:jedis:2.9.0",
            "org.apache.commons:commons-pool2:2.4.2",

            //apache
            "commons-fileupload:commons-fileupload:1.3.2",
            "org.apache.commons:commons-lang3:3.4",//深拷贝

            // MySQL
            "mysql:mysql-connector-java:6.0.4",

            //Hibernate
            "org.hibernate:hibernate-core:${HibernateVersion}",
            "org.hibernate:hibernate-validator:5.3.0.Final",

            //ehcache
            "org.ehcache:ehcache:3.1.3",
            "org.hibernate:hibernate-ehcache:${HibernateVersion}",

            //C3P0
            "com.mchange:c3p0:0.9.5.2",
            "org.hibernate:hibernate-c3p0:${HibernateVersion}",

            //shiro
            "org.apache.shiro:shiro-core:${ShiroVersion}",
            "org.apache.shiro:shiro-spring:${ShiroVersion}",
            "org.apache.shiro:shiro-web:${ShiroVersion}",
            "org.apache.shiro:shiro-ehcache:${ShiroVersion}",

            //Lombok
            "org.projectlombok:lombok:1.16.10",

            //slf4j
            "org.slf4j:jcl-over-slf4j:1.7.21",//jcl-over-slf4j 把 Commons-Logging 桥接到 Slf4J，然后 Logback。
            "ch.qos.logback:logback-core:1.1.7",
            "ch.qos.logback:logback-classic:1.1.7",

            //@NotNull
            "org.jetbrains:annotations:15.0",

            //Groovy
            "org.codehaus.groovy:groovy:2.4.7",

            //JSON
            "com.fasterxml.jackson.core:jackson-core:${JacksonVersion}",
            "com.fasterxml.jackson.core:jackson-databind:${JacksonVersion}",
            "com.fasterxml.jackson.core:jackson-annotations:${JacksonVersion}",

            "org.json:json:20160810",

            //xstream
            "com.thoughtworks.xstream:xstream:1.4.9",

            //http
            "org.apache.httpcomponents:httpcore:4.4.6",
            "org.apache.httpcomponents:httpclient:4.5.3",

    )
    testCompile(
            //jUnit
            "junit:junit:4.12",

            //Mockito
            "org.mockito:mockito-core:2.2.1",
    )
}
```
下面为gradle构建脚本
```
task copyJars(type: Copy) {
    from configurations.runtime
    into "lib" //复制到lib目录
}

//让gradle支持中文
tasks.withType(JavaCompile) {
    options.encoding = "UTF-8"
}

test {
    useJUnit()

    jacoco {
        destinationFile = file("$buildDir/jacoco/test.exec")
    }
}

tasks.withType(Test) {
    testLogging {
        // set options for log level LIFECYCLE
        events "passed", "skipped", "failed", "standardOut"
        showExceptions true
        exceptionFormat "full"
        showCauses true
        showStackTraces true

        // set options for log level DEBUG and INFO
        debug {
            events "started", "passed", "skipped", "failed", "standardOut", "standardError"
            exceptionFormat "full"
        }
        info.events = debug.events
        info.exceptionFormat = debug.exceptionFormat

        afterSuite { desc, result ->
            if (!desc.parent) { // will match the outermost suite
                def output = "结果: ${result.resultType} (${result.testCount} 个测试, ${result.successfulTestCount} 个成功, ${result.failedTestCount} 个失败, ${result.skippedTestCount} 个跳过)"
                def repeatLength = output.length()
                println('\n' + ('——' * repeatLength) + '\n' + output + '\n' + ('——' * repeatLength))
            }
        }
    }
}

jacocoTestReport {
    reports {
        xml.enabled false
        csv.enabled false
        html.destination "${buildDir}/jacocoHtml"
    }
}

build.dependsOn jacocoTestReport

task integrationTest(type: Test) {
    include "test/java/**"
}
```

## SpringMVC+Spring+JPA+Hibernate+Shiro项目结构的搭建
### 修改项目路径下的 web.xml 文件如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<web-appxmlns="http://xmlns.jcp.org/xml/ns/javaee"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
		version="3.1">
```

```
    <context-param>
        <param-name>spring.profiles.default</param-name>
        <param-value>development</param-value>
    </context-param>

    <!--Spring配置-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:Spring.xml</param-value>
    </context-param>
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <!--SpringMVC配置-->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath*:SpringMVC.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <filter>
        <filter-name>getMethodConvertingFilter</filter-name>
        <filter-class>cn.attackme.escort.Config.GetMethodConvertingFilter</filter-class>
    </filter>

    <filter-mapping>
        <filter-name>getMethodConvertingFilter</filter-name>
        <url-pattern>/*</url-pattern>
        <dispatcher>FORWARD</dispatcher>
    </filter-mapping>


    <!-- 配置Spring字符编码过滤器 -->
    <filter>
        <filter-name>encodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
        <init-param>
            <param-name>forceEncoding</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>encodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>


    <!--首页-->
    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>

    <!-- 错误页 -->
    <error-page>
        <error-code>404</error-code>
        <location>/404.jsp</location>
    </error-page>
    <error-page>
        <error-code>500</error-code>
        <location>/500.jsp</location>
    </error-page>


    <!--支持特殊字体文件-->
    <mime-mapping>
        <extension>woff</extension>
        <mime-type>application/x-font-woff</mime-type>
    </mime-mapping>

    <mime-mapping>
        <extension>ttf</extension>
        <mime-type>application/octet-stream</mime-type>
    </mime-mapping>

    <mime-mapping>
        <extension>otf</extension>
        <mime-type>application/octet-stream</mime-type>
    </mime-mapping>


    <!-- Shiro filter -->
    <!-- 通常会将其放置到最前面(即其他filter-mapping前面),以保证它是过滤器链中第一个起作用的 -->
    <filter>
        <filter-name>shiroFilter</filter-name>
        <filter-class>
            org.springframework.web.filter.DelegatingFilterProxy
        </filter-class>
        <init-param>
            <param-name>targetFilterLifecycle</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>shiroFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>


    <filter>
        <filter-name>hibernateFilter</filter-name>
        <filter-class>
            org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter
        </filter-class>
    </filter>
    <filter-mapping>
        <filter-name>hibernateFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

### 添加数据库配置信息，这里项目配置的数据库为 mysql，在 resources 下新建 db.properties 配置文件，设置如下：
```
jdbc.driverClass=com.mysql.cj.jdbc.Driver
jdbc.connectionURL=jdbc:mysql:///escort?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC&useSSL=false
jdbc.userName=root
jdbc.password=root
jdbc.checkoutTimeout=30000
jdbc.idleConnectionTestPeriod=10
jdbc.initialPoolSize=1
jdbc.maxIdleTime=30
jdbc.maxPoolSize=5
jdbc.minPoolSize=1
jdbc.maxStatements=5
jdbc.autoCommitOnClose=true


hibernate.show_sql=true
hibernate.format_sql=true
hibernate.hbm2ddl.auto=update
```

### 在 resources 下新建 Spring.xml 配置文件，设置如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:task="http://www.springframework.org/schema/task"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                            http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context
                            http://www.springframework.org/schema/context/spring-context.xsd
                            http://www.springframework.org/schema/aop
                            http://www.springframework.org/schema/aop/spring-aop.xsd http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task.xsd">

    <!--启用AspectJ自动代理-->
    <aop:aspectj-autoproxy/>

    <import resource="Spring-JPA.xml"/>
    <import resource="Spring-Shiro.xml"/>

    <!--开发环境-->
    <beans profile="development">
        <context:component-scan base-package="cn.attackme">
            <!--spring配置不扫描以下包-->
            <context:exclude-filter type="annotation"
                                    expression="org.springframework.web.bind.annotation.ControllerAdvice"/>
            <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
        </context:component-scan>
    </beans>
    <!--测试环境-->
    <beans profile="test">
        <context:component-scan base-package="cn.attackme">
            <!--spring配置不扫描以下包-->
            <context:exclude-filter type="annotation"
                                    expression="org.springframework.web.bind.annotation.ControllerAdvice"/>
            <!--<context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>-->
        </context:component-scan>
    </beans>
    <!--生产环境-->
    <beans profile="production">
        <context:component-scan base-package="cn.attackme">
            <!--spring配置不扫描以下包-->
            <context:exclude-filter type="annotation"
                                    expression="org.springframework.web.bind.annotation.ControllerAdvice"/>
            <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
        </context:component-scan>
    </beans>

</beans>
```

### 在 resources 下新建 SpringMvc.xml 配置文件，设置如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <context:component-scan base-package="cn.attackme.escort.Controller,cn.attackme.Wechat.Controller">
        <!--springMVC配置 只扫描以下包-->
        <context:include-filter type="annotation"
                                expression="org.springframework.stereotype.Controller"/>
        <context:include-filter type="annotation"
                                expression="org.springframework.web.bind.annotation.ControllerAdvice"/>
    </context:component-scan>
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver"
          p:prefix="/WEB-INF/views/"
          p:suffix=".jsp"/>

    <!--<mvc:annotation-driven/>-->
    <!--打开default-servlet-handler在springMVC找不到对应的映射时会采用默认映射-->
    <mvc:default-servlet-handler/>


    <!--直达相应页面，不经过SpringMVC映射-->
    <!--<mvc:view-controller path="test" view-name="list"/>-->

    <!--上传文件需要注册 还有其他配置-->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver"
          p:defaultEncoding="UTF-8"/>

    <!--注册拦截器-->
    <!--<mvc:interceptors>
        <bean class="AuthInterceptor"/>
    </mvc:interceptors>

    <mvc:interceptors>
        <bean class="AuthorityByRole"/>
    </mvc:interceptors>-->

    <mvc:annotation-driven>
        <mvc:message-converters register-defaults="false">
            <bean class="org.springframework.http.converter.ByteArrayHttpMessageConverter"/>
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <property name="supportedMediaTypes" value="text/plain;charset=UTF-8"/>
            </bean>
            <!--避免IE执行AJAX时,返回JSON出现下载文件-->
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
                <property name="supportedMediaTypes" value="application/json;charset=UTF-8"/>
                <property name="objectMapper" ref="jacksonObjectMapper"/>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>

    <bean id="jacksonObjectMapper" class="cn.attackme.escort.Config.JsonMapper"/>

    <!-- shiro为集成spring -->
    <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <property name="exceptionMappings">
            <props>
                <prop key="org.apache.shiro.authz.UnauthorizedException">/403.jsp</prop>
            </props>
        </property>
    </bean>

    <!-- AOP式方法级权限检查 -->
    <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
          depends-on="lifecycleBeanPostProcessor"
          p:proxyTargetClass="true"/>

    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>

    <!--静态资源映射 设置Cache-Control头，缓存时间为365天-->
    <!--生产环境-->
    <beans profile="production">
        <mvc:resources mapping="/assets/**" location="/assets/">
            <mvc:cache-control max-age="31536000" cache-public="true"/>
        </mvc:resources>
    </beans>

</beans>
```

### 在 resources 下新建 Spring-JPA.xml 配置文件，设置如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<!--suppress SpringPlaceholdersInspection -->
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                            http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/tx
                            http://www.springframework.org/schema/tx/spring-tx.xsd">

    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"
          p:user="${jdbc.userName}"
          p:password="${jdbc.password}"
          p:driverClass="${jdbc.driverClass}"
          p:jdbcUrl="${jdbc.connectionURL}"
          p:checkoutTimeout="${jdbc.checkoutTimeout}"
          p:idleConnectionTestPeriod="${jdbc.idleConnectionTestPeriod}"
          p:initialPoolSize="${jdbc.initialPoolSize}"
          p:maxIdleTime="${jdbc.maxIdleTime}"
          p:maxPoolSize="${jdbc.maxPoolSize}"
          p:minPoolSize="${jdbc.minPoolSize}"
          p:maxStatements="${jdbc.maxStatements}"
          p:autoCommitOnClose="${jdbc.autoCommitOnClose}"
    />
    <!-- jpa Entity Factory 配置 -->
    <bean id="entityManagerFactory"
          class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <property name="packagesToScan" value="cn.attackme.escort.Model"/>
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter"/>
        </property>
        <!-- 配置 JPA 的基本属性. 例如 JPA 实现产品的属性 -->
        <property name="jpaProperties">
            <props>
                <prop key="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</prop>
                <!--生产环境关闭show sql，format sql-->
                <prop key="hibernate.show_sql">${hibernate.show_sql}</prop>
                <prop key="hibernate.format_sql">${hibernate.format_sql}</prop>
                <prop key="hibernate.hbm2ddl.auto">${hibernate.hbm2ddl.auto}</prop><!--validate update-->
                <prop key="cache.use_second_level_cache">true</prop>
                <prop key="hibernate.cache.region.factory_class">org.hibernate.cache.SingletonEhCacheRegionFactory
                </prop>
                <prop key="cache.use_query_cache">true</prop>
                <prop key="connection.isolation">2</prop>
                <prop key="use_identifier_rollback">true</prop>
                <prop key="hibernate.c3p0.max_size">20</prop>
                <prop key="hibernate.c3p0.min_size">1</prop>
                <prop key="c3p0.acquire_increment">2</prop>
                <!--多长时间检测一次池内的所有链接对象是否超时-->
                <prop key="c3p0.idle_test_period">2000</prop>
                <prop key="c3p0.timeout">2000</prop>
                <!--缓存 Statement 对象的数量-->
                <prop key="c3p0.max_statements">10</prop>
                <!-- 设定 JDBC 的 Statement 读取数据的时候每次从数据库中取出的记录条数 MySql不支持-->
                <prop key="hibernate.jdbc.fetch_size">100</prop>
                <!-- 设定对数据库进行批量删除，批量更新和批量插入的时候的批次大小 MySql不支持-->
                <prop key="jdbc.batch_size">30</prop>
            </props>
        </property>
        <property name="jpaDialect">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaDialect"/>
        </property>
    </bean>

    <!-- 配置 JPA 使用的事务管理器 -->
    <bean id="transactionManager"
          class="org.springframework.orm.jpa.JpaTransactionManager"
          p:entityManagerFactory-ref="entityManagerFactory"/>

    <!-- 配置支持基于注解式事务配置 注解方式配置@EnableTransactionManager -->
    <tx:annotation-driven transaction-manager="transactionManager"/>

    <!-- 2. 配置事务属性, 需要事务管理器 -->
    <!--<tx:advice id="txAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="get*" read-only="true"/>
            <tx:method name="*"/>
        </tx:attributes>
    </tx:advice>-->

    <!--3. 配置事务切点, 并把切点和事务属性关联起来 -->
    <!--<aop:config>
        <aop:pointcut expression="execution(* com.zhangzhihao.SpringMVCSeedProject.Dao.*.*(..))"
                      id="txPointcut"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
    </aop:config>-->


    <!-- 配置数据源 -->
    <!-- 导入资源文件 -->

    <!-- 开发环境配置文件 -->
    <beans profile="development">
        <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
            <property name="locations">
                <list>
                    <value>classpath:development/db.properties</value>
                </list>
            </property>
            <property name="ignoreUnresolvablePlaceholders" value="true"/>
        </bean>
    </beans>

    <!-- 测试环境配置文件 -->
    <beans profile="test">
        <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
            <property name="locations">
                <list>
                    <value>classpath:test/db.properties</value>
                </list>
            </property>
            <property name="ignoreUnresolvablePlaceholders" value="true"/>
        </bean>
    </beans>

    <!-- 生产环境配置文件 -->
    <beans profile="production">
        <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
            <property name="locations">
                <list>
                    <value>classpath:production/db.properties</value>
                </list>
            </property>
            <property name="ignoreUnresolvablePlaceholders" value="true"/>
        </bean>
    </beans>

</beans>
```

### 在 resources 下新建 Spring-Shiro.xml 配置文件，设置如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--安全管理器-->
    <bean id="securityManager"
          class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <property name="realm" ref="shiroRealm"/>
        <property name="rememberMeManager" ref="rememberMeManager"/>

        <!--可选项 默认使用ServletContainerSessionManager，直接使用容器的HttpSession，可以通过配置sessionManager，使用DefaultWebSessionManager来替代-->
        <property name="sessionManager" ref="sessionManager"/>
        <!--可选项 最好使用;SessionDao 中 doReadSession 读取过于频繁了-->
        <property name="cacheManager" ref="shiroEhcacheManager"/>
    </bean>

    <!-- 項目自定义的Realm -->
    <bean id="shiroRealm" class="cn.attackme.escort.Repository.ShiroRealm"/>

    <!--cookie-->
    <bean id="rememberMeCookie" class="org.apache.shiro.web.servlet.SimpleCookie">
        <constructor-arg value="rememberMe"/>
        <property name="httpOnly" value="true"/>
        <property name="maxAge" value="604800"/><!-- “记住我”有效期7天 -->
    </bean>

    <!-- rememberMe管理器 -->
    <bean id="rememberMeManager" class="org.apache.shiro.web.mgt.CookieRememberMeManager">
        <!-- rememberMe cookie加密的密钥 建议每个项目都不一样 默认AES算法 密钥长度（128 256 512 位）-->
        <property name="cipherKey"
                  value="#{T(org.apache.shiro.codec.Base64).decode('6GvVhmFLUs0KTA3Kprsdag==')}"/>
        <property name="cookie" ref="rememberMeCookie"/>
    </bean>

    <!--session管理器-->
    <bean id="sessionManager"
          class="org.apache.shiro.web.session.mgt.DefaultWebSessionManager">
        <!-- 设置全局会话超时时间，默认30分钟(1800000) -->
        <property name="globalSessionTimeout" value="1800000"/>
        <!-- 是否在会话过期后会调用SessionDAO的delete方法删除会话 默认true-->
        <property name="deleteInvalidSessions" value="false"/>
        <!-- 是否开启会话验证器任务 默认true -->
        <property name="sessionValidationSchedulerEnabled" value="false"/>
        <!-- 会话验证器调度时间 -->
        <property name="sessionValidationInterval" value="1800000"/>
        <property name="sessionFactory" ref="sessionFactory"/>
        <property name="sessionDAO" ref="cachingShiroSessionDao"/>
        <!-- 默认JSESSIONID，同tomcat/jetty在cookie中缓存标识相同，修改用于防止访问404页面时，容器生成的标识把shiro的覆盖掉 -->
        <property name="sessionIdCookie">
            <bean class="org.apache.shiro.web.servlet.SimpleCookie">
                <constructor-arg name="name" value="SHRIOSESSIONID"/>
            </bean>
        </property>
        <property name="sessionListeners">
            <list>
                <ref bean="sessionListener"/>
            </list>
        </property>
    </bean>

    <!-- 自定义Session工厂方法 返回会标识是否修改主要字段的自定义Session-->
    <bean id="sessionFactory"
          class="cn.attackme.escort.ShiroSessionOnRedis.Session.ShiroSessionFactory"/>

    <!--缓存管理器-->
    <!-- 用户授权信息Cache, 采用EhCache，本地缓存最长时间应比中央缓存时间短一些，以确保Session中doReadSession方法调用时更新中央缓存过期时间 -->
    <bean id="shiroEhcacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager"
          p:cacheManagerConfigFile="classpath:ehcache-shiro.xml"/>

    <bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor"
          p:securityManager-ref="securityManager"/>


    <bean id="sessionListener"
          class="cn.attackme.escort.ShiroSessionOnRedis.Listener.ShiroSessionListener"
          p:sessionDao-ref="cachingShiroSessionDao"
          p:shiroSessionService-ref="shiroSessionService"/>

    <bean id="cachingShiroSessionDao"
          class="cn.attackme.escort.ShiroSessionOnRedis.Repository.CachingShiroSessionDao"
          p:sessionRepository-ref="shiroSessionRepository"/>

    <bean id="shiroSessionRepository"
          class="cn.attackme.escort.ShiroSessionOnRedis.Repository.ShiroSessionRepository"
          p:redisTemplate-ref="redisTemplate"/>

    <bean id="shiroSessionService"
          class="cn.attackme.escort.ShiroSessionOnRedis.Service.ShiroSessionService"
          p:redisTemplate-ref="redisTemplate"
          p:sessionDao-ref="cachingShiroSessionDao"/>

    <!-- Shiro Filter -->
    <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <property name="securityManager" ref="securityManager"/>
        <property name="loginUrl" value="/Account/Login"/>
        <property name="successUrl" value="/"/>
        <property name="unauthorizedUrl" value="/Account/Login"/>
        <property name="filterChainDefinitions">
            <value>
                /Wechat/** = anon
                /OAuth2/** = anon
                /Account/** = anon
                /login = anon
                /assets/** = anon
                /app/** = anon
                /** = user
            </value>
        </property>
    </bean>

</beans>
```
到这里 SpringMVC+Spring+JPA+Hibernate+Shiro的基本配置文件基本就完成了,大致的项目结构如下：
![结构](https://gaoyuyue.github.io/picture/2017-3-27_1.png)

本篇主要以代码示例为主，下一篇我将简述一个基于这个框架的简单应用。

转载请注明出处：[https://gaoyuyue.github.io/2017/03/27/SpringMVC+Spring+JPA+Hibernate+Shiro配置整合/](https://gaoyuyue.github.io/2017/03/27/SpringMVC+Spring+JPA+Hibernate+Shiro配置整合/)