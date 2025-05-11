# 基本介绍

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.4.5</version>
</parent>
```

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>3.4.4</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

# 属性装配

## YAML

```bash
# yaml， yml都支持
# properties的属性，优先级比yaml/yml高
```

```bash
# 简单数据类型，可以使用@Value注入
1. key ---- value； 空格控制层级关系，遵循左对齐的原则， 大小写敏感

2. 字符串：       a. 默认不加引号
                 b. " "  特殊字符     -----  不转义
                 c. ' '  特殊字符     -----  转义 
    
# 复杂数据，不能用@Value来进行注入    
3. 对象，Map:          a. 多行写法，      b. 行内写法

4. List，数组：   a. 多行写法        b. 行内写法

5. 上面的不同数据，都可以进行组合
```

```yaml
student:
  age: ${random.int}
  name: ${random.uuid}
  email: ${student.age}_1037289945@qq.com
  address: ${student.name:shanxi}_beijing
  isActive: true
  birthday: 2022/09/20
  info: { color: red, size: 10 }
  friends: [ erick, tom, lucy ]
  phone:
    - apple
    - huawei
    - xiaomi
      
# el表达式
# ${student.age}_1037289945@qq.com：  将该配置的属性和常量字符串拼接
# ${student.age}_1037289945@qq.com：  如果找不到，则将该表达式直接转换为字符串
# ${student.name:shanxi}_beijing：    如果找不到，则提供默认值shanxi、
```



## Conditional

- 条件装配：满足指定条件的，则进行组件注入
- spring本身的注解，package org.springframework.context.annotation。springboot对该注解进行了扩展

```java
// 1. 作用在类上      ：不满足条件时，该类的下面的所有的注解都不生效
// 2. 作用在方法上     ：满足该条件后，该方法会执行
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {
	Class<? extends Condition>[] value();
}
```

### ConditionalOnBean

- ConditionalOnMissingBean，ConditionalOnBean
- package org.springframework.boot.autoconfigure.condition
- 容器中存在或不存在某个组件时，才会去加载
- 在一个配置类中，@Bean的加载顺序一般是从上而下的，不绝对

#### 基本使用

```bash
# 同一个配置类
- 注意@Bean的书写顺序，不同的书写顺序会导致不同的结果
- 调换两个@Bean的顺序，就会产生不一样的结果

# 不同配置类
- 尽量避免两个有依赖的类写在同一个@Configuration里面，写在不同的配置类中，避免因为代码顺序的问题出错
```

```java
package com.citi.customized;

import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.autoconfigure.condition.ConditionalOnBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Slf4j
@Configuration
public class FirstConfig {

    @Bean
    @ConditionalOnBean(value = {Dog.class})
    public DogOwner getDogOwner() {
        return new DogOwner();
    }

    @Bean
    public Dog getDog() {
        return new Dog();
    }

    public static class Dog {
        public Dog() {
            log.info("dog created");
        }
    }

    public static class DogOwner {
        public DogOwner() {
            log.info("dog-owner created");
        }
    }
}
```

#### 匹配条件

```java
package com.erick.daydreamer.config;

import com.erick.daydreamer.entity.Dog;
import org.springframework.boot.autoconfigure.condition.ConditionalOnBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SecondConfig {


    /*1. 类的class对象： Class<?>[] value() default {}*/
    /*1.1 本项目的类: 可以通过导入Dog类的方式引入*/
    @Bean
    @ConditionalOnBean(value = {Dog.class})
    public DogOwner getFirstDogOwner01() {
        System.out.println("类的class对象-本项目的类");
        return new DogOwner();
    }

    /*1.2. 其他项目的类
     *       - 使用类的全限定类名来获取class对象
     *       - 其他项目识别该SecondConfig后，如果对应的@ConditionalOnBean中包含了该类，则IDEA正常，
     *       - 否则IDEA爆红，导入对应的爆红的类的依赖即可*/
    @Bean
    @ConditionalOnBean(value = {ch.qos.logback.core.BasicStatusManager.class})
    public DogOwner getFirstDogOwner02() {
        System.out.println("类的class对象-外部的类");
        return new DogOwner();
    }

    /*2. 类的String全限定类名： String[] type() default {};*/
    @Bean
    @ConditionalOnBean(type = {"com.erick.daydreamer.entity.Dog"})
    public DogOwner getSecondDogOwner01() {
        return new DogOwner();
    }

    @Bean
    @ConditionalOnBean(type = {"ch.qos.logback.core.BasicStatusManager.class"})
    public DogOwner getSecondDogOwner02() {
        return new DogOwner();
    }

    /*3. name匹配：某个名字的组件，String[] name() default {}; */
    @Bean
    @ConditionalOnBean(name = {"dog001"})
    public DogOwner getThirdDogOwner01() {
        return new DogOwner();
    }

    /*什么都不写的话，代表默认检查当前方法返回值的类的类型：是否在容器中存在
     * 1. 一般是 @ConditionalOnMissingBean用的比较多，用来确保该class类型的组件，只存在一份*/
    @Bean
    public DogOwner defaultMethod() {
        System.out.println("default");
        return new DogOwner();
    }

    public static class DogOwner {
    }
}
```

### ConditionalOnClass

- 类路径下包含某个Class才会自动加载

```java
package com.erick.daydreamer.config;

import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DatasourceConfig {

    @Bean
    @ConditionalOnClass(value = {org.apache.logging.log4j.simple.SimpleLogger.class})
    public String text01() {
        return "success";
    }
}
```

### ConditionalOnProperty

- 根据配置文件做开关，选择性加载某些bean

```java
package com.erick.daydreamer.config;

import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class JmsConfig {

    /*返回值是String: 只是为了让该bean加载*/

    /*prefix：前缀
     * name: 对应某个属性
     * havingValue： 值是不是相同，严格相等，忽略大小写
     * matchIfMissing： 如果该属性没配置，则默认不加载*/
    @Bean
    @ConditionalOnProperty(prefix = "app.datasource", name = "flag", havingValue = "AA", matchIfMissing = false)
    public String get01() {
        return "success";
    }
}
```

```yaml
app:
  datasource:
    flag: aa
```

## ConfigurationProperties

- 默认加载application-xxx文件，将其中的属性一一对应到一个JavaBean中(一般是xxxProperties)
- 只是将配置和JavaBean属性绑定，但是JavaBean还没放在容器中，需要结合下面几种注解

### 自动提示

- 写了配置文件后，一定要重新编译项目，才能使其生效

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
</dependency>
```

![image-20250501095157264](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250501095157264.png)

#### 静态内部类

- xxxProperties嵌套了其他类，其他类只会在本类中使用
- 使用了对应的setter和getter进行属性赋值

```java
package com.citi.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.NestedConfigurationProperty;
import org.springframework.context.annotation.Configuration;

/*默认值：true: 配置文件的属性，VO不包含，则VO不解析*/
@Data
@ConfigurationProperties(prefix = "citi.ems.server", ignoreUnknownFields = true)
public class JmsProperties {
    /*代码的注释：可以在写配置文件的时候进行提示*/
    /**
     * ems username
     */
    private String userName;

    /**
     * ems password
     */
    private String password;

    /**
     * queue config
     */
    /*为了在idea中的代码提示*/
    @NestedConfigurationProperty
    private QueueConfig queueConfig;

    @Data
    public static class QueueConfig {
        /**
         * queue-name
         */
        private String queueName;
    }
}
```

#### 多个类

- 如果嵌套类也会在其他地方使用

```java
package com.citi.config;

import com.citi.customized.QueueConfig;
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.context.properties.NestedConfigurationProperty;
import org.springframework.context.annotation.Configuration;

@Data
@ConfigurationProperties(prefix = "citi.ems.server")
@Configuration
public class JmsProperties {
    private String userName;
    private String password;
    /*为了在idea中的代码提示*/
    @NestedConfigurationProperty
    private QueueConfig queueConfig;
}
```

```java
package com.citi.customized;

import lombok.Data;

@Data
public class QueueConfig {
    private String queueName;
}
```

### Component/Configuration 

- 应用场景：本项目自己配置的，不需提供给外界服务

```java
package com.citi.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Data
@ConfigurationProperties(prefix = "citi.ems.server")
@Configuration
public class JmsProperties {
    private String userName;
}
```

### Import

- 配置类可能是三方包，本项目包扫描不到，将对应的配置类，通过@Import的方式扔进来

```bash
# @ConfigurationProperties + @Import 
- @ConfigurationProperties + @Import这两个注解，还是没有放在容器中，因为@Import必须作用在其他容器类注解的类上
- 比如@Import+@Configuration(三方包如果不把对应的@ConfigurationProperties放在容器中，IDEA会爆红)

# 多个@Import类可以继承
- @Import是springframework自身的实现
```

#### 三方包

```java
package com.lucy;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;

@Data
@ConfigurationProperties(prefix = "erick.desk")
public class DeskProperties {
    private String brand;
    private String address;
}
```

```java
package com.lucy;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;

@Data
@ConfigurationProperties(prefix = "citi.jms", ignoreUnknownFields = true)
public class JmsProperties {
    private String userName;
    private String password;
    private int maxPoolSize;
}
```

```java
package com.third.lucy;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Configuration // 不加容器注解，三方包的IDEA会爆红
@Import(value = {DeskProperties.class, JmsProperties.class})
public class ThirdAutoConfig {
}
```

#### 本项目

```java
package com.erick;

import com.lucy.ThirdAutoConfig;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Import;

@SpringBootApplication
@Import(value = ThirdAutoConfig.class)
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class);
    }
}
```

### EnableConfigurationProperties

```bash
# @EnableConfigurationProperties
- 将导入的类的组件，装配到容器中

# 应用场景 :适用于三方包中
- 主要解决： 被@ConfigurationProperties标注的三方包中的类，本地项目包扫不到的
```

#### 三方包

```java
package com.lucy;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;

@Data
@ConfigurationProperties(prefix = "citi.jms", ignoreUnknownFields = true)
public class JmsProperties {
    private String userName;
    private String password;
    private int maxPoolSize;
}
```

```java
package com.lucy;

import org.springframework.boot.context.properties.EnableConfigurationProperties;

/*1. 开启JmsProperties的装配功能
 * 2. 在其他项目中的配置类中，只需要@Import进JmsAutoconfig即可*/
@EnableConfigurationProperties(value = JmsProperties.class)
public class JmsAutoconfig {
}
```

#### 本项目

```java
package com.erick;

import com.lucy.JmsAutoconfig;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Import;

@SpringBootApplication
@Import(JmsAutoconfig.class) // 随便找个配置类，将@Import(JmsAutoconfig.class) 扔进去即可
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class);
    }
}
```

### PropertySource

- 自动装配的类的属性，不是在application.yaml中指定的，则需要该注解来显示指定
- 这两个注解，只是自动装配，不会注入组件

#### 三方包

```java
package com.third;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;


@Data
@ConfigurationProperties(prefix = "erick.jdbc")
@PropertySource(value = "classpath:jdbc.properties")
public class ErickJdbcProperties {
    private String url;
    private String userName;
    private String password;
    private String port;
}
```

```java
package com.third;

import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Import;

@Configuration
@Import(ErickJdbcProperties.class)
public class ErickJdbcAutoconfiguration {
}
```

#### 调用方

```properties
jdbc.driverClass = com.mysql.cj.jdbc.Driver
jdbc.url = jdbc:mysql://10.5.16.76:3306/stu_db
jdbc.username = root
jdbc.password = 123456
```

```java
@Import(value = {ErickJdbcAutoconfiguration.class})
```

# 自动装配

## 1. @SpringBootApplication

- 下面三个注解的合成

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
```

### 1.1 @SpringBootConfiguration

- 和@Configuration一样，声明主启动类就是一个配置类

```java
package org.springframework.boot;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.annotation.AliasFor;
import org.springframework.stereotype.Indexed;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
@Indexed
public @interface SpringBootConfiguration {
	@AliasFor(annotation = Configuration.class)
	boolean proxyBeanMethods() default true;
}
```

### 1.2 @ComponentScan

- 只是排除了一些不需要的扫描的包，并不是扫描本项目的包

```java
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
```

### 1.3 @EnableAutoConfiguration

- 容器注册的高层注解

## 2. @EnableAutoConfiguration

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {
	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";
	Class<?>[] exclude() default {};
	String[] excludeName() default {};
}
```

### @AutoConfigurationPackage

- 具体的功能：AutoConfigurationPackages.Registrar.class实现

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {

	String[] basePackages() default {};

	Class<?>[] basePackageClasses() default {};
}
```

#### AutoConfigurationPackages.Registrar

- 扫描本项目的包的bean

```java
// 实现了ImportBeanDefinitionRegistrar，就是批量向容器中注册组件的，参考@Import的讲解
static class Registrar implements ImportBeanDefinitionRegistrar, DeterminableImports {

		@Override
		public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
      // 根据主动启动类上的包名得到默认扫描包：metadata获取到项目扫描包 "com.citi"
      // 将包下的所有组件，批量注册
			register(registry, new PackageImports(metadata).getPackageNames().toArray(new String[0]));
		}

		@Override
		public Set<Object> determineImports(AnnotationMetadata metadata) {
			return Collections.singleton(new PackageImports(metadata));
		}
	}
```

### @Import(AutoConfigurationImportSelector.class)

- 主要是实现了ImportSelector，来向容器中批量注册组件

![AutoConfigurationImportSelector](https://skillset.oss-cn-shanghai.aliyuncs.com/AutoConfigurationImportSelector.png)

```java
// 自动装配	
@Override
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
    // 批量导入组件
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(annotationMetadata);
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}
```

```java
	protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
		if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
    // 获取需要导入的组件的类全名
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
		configurations = removeDuplicates(configurations);
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
		checkExcludedClasses(configurations, exclusions);
		configurations.removeAll(exclusions);
		configurations = getConfigurationClassFilter().filter(configurations);
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return new AutoConfigurationEntry(configurations, exclusions);
	}
```

```java
// 获取自动加载的组件	
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
		ImportCandidates importCandidates = ImportCandidates.load(this.autoConfigurationAnnotation,
				getBeanClassLoader());
		List<String> configurations = importCandidates.getCandidates();
		Assert.notEmpty(configurations,
				"No auto configuration classes found in " + "META-INF/spring/"
						+ this.autoConfigurationAnnotation.getName() + ".imports. If you "
						+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}
```

```java
	public static ImportCandidates load(Class<?> annotation, ClassLoader classLoader) {
		Assert.notNull(annotation, "'annotation' must not be null");
		ClassLoader classLoaderToUse = decideClassloader(classLoader);
    // META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
    // 会加载springboot对应的组件
		String location = String.format(LOCATION, annotation.getName());
		Enumeration<URL> urls = findUrlsInClasspath(classLoaderToUse, location);
		List<String> importCandidates = new ArrayList<>();
		while (urls.hasMoreElements()) {
			URL url = urls.nextElement();
			importCandidates.addAll(readCandidateConfigurations(url));
		}
		return new ImportCandidates(importCandidates);
	}
```

![image-20250501104919522](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250501104919522.png)

- 虽然加载了所有的组件，但是spring是按需开启，结合@Conditional系列的注解

## 3. 按需加载

- 所有的自动配置类都在该文件中

![image-20250501105341577](https://skillset.oss-cn-shanghai.aliyuncs.com/image-20250501105341577.png)

### AopAutoConfiguration

```java
/*
 * Copyright 2012-2022 the original author or authors.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      https://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package org.springframework.boot.autoconfigure.aop;

import org.aspectj.weaver.Advice;

import org.springframework.aop.config.AopConfigUtils;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.boot.autoconfigure.AutoConfiguration;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@AutoConfiguration
@ConditionalOnProperty(prefix = "spring.aop", name = "auto", havingValue = "true", matchIfMissing = true)
public class AopAutoConfiguration {

	@Configuration(proxyBeanMethods = false)
	@ConditionalOnClass(Advice.class)
	static class AspectJAutoProxyingConfiguration {

		@Configuration(proxyBeanMethods = false)
		@EnableAspectJAutoProxy(proxyTargetClass = false)
		@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "false")
		static class JdkDynamicAutoProxyConfiguration {

		}

		@Configuration(proxyBeanMethods = false)
		@EnableAspectJAutoProxy(proxyTargetClass = true)
		@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true",
				matchIfMissing = true)
		static class CglibAutoProxyConfiguration {

		}

	}

	@Configuration(proxyBeanMethods = false)
	@ConditionalOnMissingClass("org.aspectj.weaver.Advice")
	@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true",
			matchIfMissing = true)
	static class ClassProxyingConfiguration {

		@Bean
		static BeanFactoryPostProcessor forceAutoProxyCreatorToUseClassProxying() {
			return (beanFactory) -> {
				if (beanFactory instanceof BeanDefinitionRegistry registry) {
					AopConfigUtils.registerAutoProxyCreatorIfNecessary(registry);
					AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
				}
			};
		}

	}

}
```

### DispatcherServletAutoConfiguration

- WebMvcProperties： 对应配置文件，取值后和通过new的方式，对另外一个VO进行映射，即使属性相同
- 组件改名：如果容器中包含某个组件，但是组件名不对，可以通过这种方式对组件进行更改名字

```java
@AutoConfigureOrder(Ordered.HIGHEST_PRECEDENCE)
// AutoConfiguration通过after和before属性，来进行自动注册的顺序
@AutoConfiguration(after = ServletWebServerFactoryAutoConfiguration.class)
@ConditionalOnWebApplication(type = Type.SERVLET)
@ConditionalOnClass(DispatcherServlet.class)
public class DispatcherServletAutoConfiguration {

    public static final String DEFAULT_DISPATCHER_SERVLET_BEAN_NAME = "dispatcherServlet";

    public static final String DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME = "dispatcherServletRegistration";

    @Configuration(proxyBeanMethods = false)
    @Conditional(DefaultDispatcherServletCondition.class)
    @ConditionalOnClass(ServletRegistration.class)
    // 从配置文件的配置文件中去取值，以后容器中就会有该WebMvcProperties组件
    @EnableConfigurationProperties(WebMvcProperties.class)
    protected static class DispatcherServletConfiguration {

        @Bean(name = DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
        public DispatcherServlet dispatcherServlet(WebMvcProperties webMvcProperties) {
            // 自己new好，然后从WebMvcProperties中去取值，然后将对象放在容器中
            // WebMvcProperties和DispatcherServlet不要做成一个对象，哪怕两个属性一摸一样
            DispatcherServlet dispatcherServlet = new DispatcherServlet();
            dispatcherServlet.setDispatchOptionsRequest(webMvcProperties.isDispatchOptionsRequest());
            dispatcherServlet.setDispatchTraceRequest(webMvcProperties.isDispatchTraceRequest());
            dispatcherServlet.setThrowExceptionIfNoHandlerFound(webMvcProperties.isThrowExceptionIfNoHandlerFound());
            dispatcherServlet.setPublishEvents(webMvcProperties.isPublishRequestHandledEvents());
            dispatcherServlet.setEnableLoggingRequestDetails(webMvcProperties.isLogRequestDetails());
            return dispatcherServlet;
        }

        @Bean
        @ConditionalOnBean(MultipartResolver.class)
        @ConditionalOnMissingBean(name = DispatcherServlet.MULTIPART_RESOLVER_BEAN_NAME)
        // 组件改名：但是就会组件重复了
        // 如果容器中有某个组件，但是不知道组件名是什么，通过这种方式，通过方法名或者bean的属性，来对容器中的组件进行改名
        public MultipartResolver multipartResolver(MultipartResolver resolver) {
            // Detect if the user has created a MultipartResolver but named it incorrectly
            return resolver;
        }

    }
```

### HttpEncodingAutoConfiguration

- ConditionalOnMissingBean： 容器中如果用户没配置，spring才会去给你一个兜底的bean
- spring设计原则一： 底层配置了所有的东西，用户如果配置，则用用户的
- spring设计原则二： 用户不单独配置，但是用户可以去改对应的属性Encoding

```java
@AutoConfiguration
@EnableConfigurationProperties(ServerProperties.class) // 自动属性导入
@ConditionalOnWebApplication(type = ConditionalOnWebApplication.Type.SERVLET)
@ConditionalOnClass(CharacterEncodingFilter.class)
@ConditionalOnProperty(prefix = "server.servlet.encoding", value = "enabled", matchIfMissing = true)
public class HttpEncodingAutoConfiguration {

    private final Encoding properties;

    // 将ServerProperties从容器中去拿，因为这个bean只有一个构造方法，所以省略@Autowired
    public HttpEncodingAutoConfiguration(ServerProperties properties) {
        this.properties = properties.getServlet().getEncoding();
    }

    @Bean
    /**
     * 如果容器中用户没这种组件，才会去使用spring提供的
     */
    @ConditionalOnMissingBean
    public CharacterEncodingFilter characterEncodingFilter() {
        CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
        filter.setEncoding(this.properties.getCharset().name());
        filter.setForceRequestEncoding(this.properties.shouldForce(Encoding.Type.REQUEST));
        filter.setForceResponseEncoding(this.properties.shouldForce(Encoding.Type.RESPONSE));
        return filter;
    }
```

```java
public class Encoding {
    public static final Charset DEFAULT_CHARSET;
    private Charset charset;

    public Encoding() {
       // 默认值：自己随便去实现 
        this.charset = DEFAULT_CHARSET;
    }

    public Charset getCharset() {
        return this.charset;
    }

    static {
        DEFAULT_CHARSET = StandardCharsets.UTF_8;
    }
}
```

# 自定义Starter

[官方starter](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters)

- 官方：spring-boot-starter-data-redis
- 个人定制：xxx-spring-boot-starter

## 入门案例

### 引入原因

- springboot的多个项目中，包含相同的代码(spring代码或者普通java代码)，需要进行抽取成三方包
- 三方包，因为包结构不同，其他项目直接导入依赖的话，不会扫描三方包中的spring组件
- 因此，spring官方推荐使用xxx-spring-starter

### 功能代码

- 功能代码抽取，其他项目使用时，因为spring的包扫描规则，不能扫描到该三方类
- 直接在主启动类上加@ComponentScan扫第三方的包才能将下面的代码加载到容器中，但不推荐

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.nike</groupId>
    <artifactId>email-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.4.5</version>
    </parent>
    <properties>
        <maven.compiler.source>17</maven.compiler.source>
        <maven.compiler.target>17</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

```java
package com.nike.service;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Slf4j
@Service
public class AccountService {
    public void sayHello() {
        log.info("Hello World!");
    }
}
```

```java
package com.nike.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Configuration;

@Data
@ConfigurationProperties(prefix = "nike.email")
@Configuration
public class JmsProperties {
    private String region;
    private String userName;
    private String password;
    private int price;
}
```

```java
package com.nike.service;

import com.nike.config.JmsProperties;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

@Slf4j
@Service
@RequiredArgsConstructor
public class EmailService {
    private final AccountService accountService;
    private final JmsProperties jmsProperties;

    public void getInfo() {
        accountService.sayHello();
        log.info("jsmProperties={}", jmsProperties);
    }
}
```

### 基本抽取-@Import

#### starter方

- 提供自动配置类，@Import所有功能代码中的，被spring管理的组件

```java
package com.nike.out;

import com.nike.config.JmsProperties;
import com.nike.service.AccountService;
import com.nike.service.EmailService;
import org.springframework.context.annotation.Import;

@Import(value = {EmailService.class, JmsProperties.class, AccountService.class})
public class EmsAutoConfiguration {
}
```

#### 调用方

- 调用方的包名时com.citi，三方包的是com.nike

```xml
<dependency>
    <groupId>com.nike</groupId>
    <artifactId>email-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

```java
@Import(EmsAutoConfiguration.class)
```

```yaml
nike:
  email:
    password: 1
    user-name: 'shuzhan'
    price: 10
    region: 'xian'
```

### 再次抽取-注解

- 上面调用方需要知道具体要@Import对应的类，比较麻烦

#### starter方

- 添加对应的注解，声明starter要暴露出去的starter

```java
package com.nike.out;

import org.springframework.context.annotation.Import;

import java.lang.annotation.*;

@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(EmsAutoConfiguration.class)
public @interface EnableEms {
}
```

#### 调用方

- 在任意的配置类上添加@EnableEms即可

### 终极抽取-SPI

- 上面的方案，还是需要调用方来进行标注。希望调用方只要导入了starter的包，就把其对应的spring组件全部加载进来
- 只对starter进行修改，调用方只需要引入依赖和对应的配置文件，就能够导入starter的所有spring的组件
- 项目启动后，会自动扫描当前类路径下的包，识别到该类型文件，再进一步扫描

```bash
# 1. 删除starter的注解

# 2. 在starter的resource目录下，添加META-INF/spring目录下，添加文件
org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

```bash
# 文件中是要暴露的自动配置类的全限定类名
com.aws.email.EmsAutoConfiguration
```

![image-20230606172839733](https://erick-typora-image.oss-cn-shanghai.aliyuncs.com/img/image-20230606172839733.png)

## 进阶案列

- 类似springboot官方的案例

### starter

#### @Autowired方式

```java
package com.taobao;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;

@Data
@ConfigurationProperties(prefix = "taobao.good")
public class GoodProperties {
    private String brand;
    private String name;
    private int price;
}
```

```java
package com.taobao.service;

import com.taobao.GoodProperties;
import org.springframework.beans.factory.annotation.Autowired;

/*默认service不要放在容器中, 而是通过自动配置，把这个service交给容器*/
public class AccountService {

    @Autowired
    private GoodProperties goodProperties;

    public void showInfo() {
        System.out.println(goodProperties);
    }
}
```

```java
package com.taobao;
/*选择性的把service放在容器中*/

import com.taobao.service.AccountService;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
// 把properties属性进行填充并放在容器中
@EnableConfigurationProperties(value = GoodProperties.class)
public class AccountAutoConfiguration {

    @Bean
    /*容器中没有AccountService时才生效*/
    @ConditionalOnMissingBean
    public AccountService getAccountService() {
        /*new的时候，自动会把里面的GoodProperties进行初始化*/
        System.out.println("starter中的AccountService");
        return new AccountService();
    }
}
```

```bash
com.taobao.AccountAutoConfiguration
```

#### 常用方式

```java
package com.taobao.service;

import com.taobao.GoodProperties;

/*默认service不要放在容器中, 而是通过自动配置，把这个server交给容器*/
public class AccountService {

    /*属性也不要通过set来注入*/
    private final GoodProperties goodProperties;

    public AccountService(GoodProperties goodProperties) {
        this.goodProperties = goodProperties;
    }

    public void showInfo() {
        System.out.println(goodProperties);
    }
}
```

```java
package com.taobao;
/*选择性的把service放在容器中*/

import com.taobao.service.AccountService;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
// 把properties属性进行填充并放在容器中
@EnableConfigurationProperties(value = GoodProperties.class)
public class AccountAutoConfiguration {

    @Bean
    /*容器中没有AccountService时才生效
     * 参数是从容器中拿*/
    @ConditionalOnMissingBean
    public AccountService getAccountService(GoodProperties properties) {
        /*new的时候，自动会把里面的GoodProperties进行初始化*/
        System.out.println("starter中的AccountService");
        return new AccountService(properties);
    }
}
```

### 调用方

```bash
# 不配置的时候，用starter提供的@Bean

# 调用方配置了三方包的bean的时候，就用配置方的
- 并且也会进行属性的@Autowired自动注入
```

```java
package com.erick.daydreamer.config;

import com.taobao.service.AccountService;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class TaobaoConfig {

    /*虽然是自己new的，但是因为AccountService中有@Autowired的属性，
    所以goodProperties属性会自动填充*/
    @Bean
    public AccountService getAccountService() {
        System.out.println("调用方的AccountService");
        return new AccountService();
    }
}
```

# Best Practices 

## 文件读取

- 从resource目录下读取文件，需要能够在本地运行，打成jar包后也能运行
- 读取json文件，得到对应的json的字符串

### ClassPathResource

- spring提供的一个工具

```java
package com.citi.replay.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.core.io.ClassPathResource;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;

@Slf4j
public class LoadFile {

    /*文件目录在resources下*/
    public String loadFile() throws IOException {
        ClassPathResource resource = new ClassPathResource("scripts/erick.json");
        String fileContent = new String(Files.readAllBytes(Paths.get(resource.getURI())));
        log.info("fileContent={}", fileContent);
        return fileContent;
    }
}
```

### ResourceLoader

- spring提供的一个工具

```java
package com.citi.replay.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;

@Slf4j
@Service
public class LoadFileService {

    @Autowired
    private ResourceLoader resourceLoader;

    public String readFiles() throws IOException {
        Resource resource = resourceLoader.getResource("classpath:scripts/erick.json");
        String fileContent = new String(Files.readAllBytes(Paths.get(resource.getURI())));
        log.info("fileContent={}", fileContent);
        return fileContent;
    }
}
```

## 项目启动后执行某个方法

- 一般使用CommandLineRunner接口和ApplicationRunner接口较多

### @PostConstruct

- 所在的类必须放在容器中
- 初始化该容器的时候，会触发

```java
package com.citi.replay.controller;


import jakarta.annotation.PostConstruct;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

@Slf4j
@Component
public class InitService {
    @PostConstruct
    public void init() {
        log.info("=====================================");
    }
}
```

### CommandLineRunner接口

- 接口提供了一个`方法，这个方法会在Spring Application的上下文加载完成后立即执行
- 在应用启动时执行一些数据库迁移、初始化数据等操作非常有用
- 必须放在容器中

```java
package com.citi.replay.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

@Component
@Slf4j
public class LoadService implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
       log.info("CommandLineRunner=============================CommandLineRunner");
    }
}
```

### ApplicationRunner接口

- 必须放在容器中
- 项目完全启动后执行的动作

```java
package com.citi.replay.controller;

import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;

@Component
@Slf4j
public class LoadSecondService implements ApplicationRunner {
    @Override
    public void run(ApplicationArguments args) throws Exception {
        log.info("ApplicationRunner===========================ApplicationRunner");
    }
}
```

## CSV

- Comma Separated Value

### Apache Common CSV

[GitHub](https://github.com/apache/commons-csv)

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-csv</artifactId>
    <version>1.13.0</version>
</dependency>
```

#### 导出文件

- springboot中，生成csv文件，前端发送请求后，自动下载文件

```java
package com.citi.file;

import lombok.AllArgsConstructor;
import lombok.Data;

@Data
@AllArgsConstructor
public class Student {
    private int id;
    private String name;
    private String email;
    private String address;
}
```

```java
package com.citi.file;

import lombok.extern.slf4j.Slf4j;
import org.apache.commons.csv.CSVFormat;
import org.apache.commons.csv.CSVPrinter;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.io.StringWriter;
import java.util.List;

@Service
@Slf4j
public class CSVService {

    public String convertDataToString(List<Student> data, String[] headers) {
        /*1. 定义CSV的header，分割符*/
        CSVFormat csvFormat = CSVFormat.DEFAULT
                .builder()
                .setHeader(headers)
                .setDelimiter(',')
                .get();

        /*2. 读取数据到文件中 */
        try (StringWriter sw = new StringWriter();
             CSVPrinter printer = new CSVPrinter(sw, csvFormat)) {
            for (Student student : data) {
                printer.printRecord(student.getId(), student.getName(), student.getEmail(), student.getAddress());
            }
            /*3. 得到具体的CSV文件的字符串*/
            return sw.toString();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

```java
package com.citi.controller;

import com.citi.file.CSVService;
import com.citi.file.Student;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpHeaders;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.List;

@Slf4j
@RestController
@RequestMapping("/export")
public class ReplayExportController {

    @Autowired
    private CSVService csvService;

    @GetMapping("/csv")
    public void exportCSV(HttpServletResponse response) throws IOException {
        String fileName = "data.csv";
        response.setContentType("text/csv");
        response.setHeader(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + fileName + "\"");
        response.setCharacterEncoding(StandardCharsets.UTF_8.name()); // 确保中文不会乱码
        response.getWriter().write(initStudent());
    }


    public String initStudent() {
        List<Student> students = new ArrayList<>();
        students.add(new Student(123, "张三", "beijing", "192.com"));
        students.add(new Student(345, "李四", "nanjing", "183.com"));
        students.add(new Student(456, "王五", "美国", "qq.com"));
        String[] headers = {"id", "姓名", "email", "address"};
        return csvService.convertDataToString(students, headers);
    }
}
```

