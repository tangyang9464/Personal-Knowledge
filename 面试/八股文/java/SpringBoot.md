#### SpringBoot

##### 自动装配原理

自动装配就是SpringBoot提供的只需要少量的注解和配置就能直接使用功能

1. 通过`@EnableAutoConfiguration`开启自动装配。

2. SpringBoot 在启动时会扫描外部引用 jar 包中的`META-INF/spring.factories`自动配置类文件，将文件中配置的类型信息加载到 Spring 容器
3. 自动配置类文件就是通过`@Conditional`按需加载的配置类
	- @ConditionalOnBean：当容器里有指定 Bean 的条件下
	- `@ConditionalOnMissingBean`：当容器里没有指定 Bean 的情况下

##### 与Sping区别

SpringBoot是Spring的扩展，消除了Spring繁琐的XML配置。

内嵌Tomcat，通过引入starter就能自动装配应用。

自动装配原理如上。

