项目名称：一体化博客项目 

架构模式：前后端分离 

技术点：springboot+任何模版 

难度：简单 

环境：云计算 

==========项目隐私========== 

项目分类：总后端，个人分离独立前端（blog） 

项目安全：springsecurity（使用AOP写了一个filter类，一看就懂，不懂交流） 

项目数据库：mysql，redis（简单实现，相互交流）

 项目合伙人：pofe ，sunbing1992 ，新时代好青年（一人学习气氛不够，一人之力不成美景，一起共勉！）

===========以上为初建思路，后期改动再添加介绍===========
项目初搭建基于架构图：![图片](https://dev.tencent.com/api/project/4036345/files/4553716/imagePreview) 
1.第一步搭建nginx（https://www.cnblogs.com/edward2013/p/5506588.html）  
--启动成功：  http://193.112.33.91:8011

2.搭建eureka（集群3台，环境为idea）：
-application.yml配置文件：
```
###服务端口号
server:
  port: 8100
###eureka 基本信息配置
spring:
  application:
    name: eureka-server
eureka:
  instance:
    ###注册到eurekaip地址
    hostname: 127.0.0.1
  client:
    serviceUrl:
      ###相互注册端口号
      defaultZone: http://${eureka.instance.hostname}:8200/eureka/,http://${eureka.instance.hostname}:8300/eureka/
    ###因为自己是为注册中心，不需要自己注册自己
    register-with-eureka: true
    ###因为自己是为注册中心，不需要检索服务
    fetch-registry: true
  server:
    #开启自我保护模式
    enable-self-preservation: false
    #清理无效节点,默认60*1000毫秒,即60秒
    eviction-interval-timer-in-ms: 5000


```
另外2个配置文件需要更改端口号，三个文件实际部署需要修改ip（注意）
-启动类代码：
```
package pofe;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

/**
 * @author a1
 */
@SpringBootApplication
@EnableEurekaServer
public class EurekaServer1Application {

	public static void main(String[] args) {
		SpringApplication.run(EurekaServer1Application.class, args);
	}

}

```
-pom.xml 信息为：
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.1.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.dingpengfei</groupId>
	<artifactId>spud-eureka-server-1</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>eureka-server-1</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.RC2</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
		</repository>
	</repositories>

</project>
```
3.搭建eureka客户端（服务提供者，服务消费者，zuul网关，security，config中心）：
-服务提供者（分布式，无需集群）
1.pom.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.1.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.dingpengfei</groupId>
	<artifactId>springcloud-eureka-client-schooling-a</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>springcloud-eureka-client-schooling-a</name>
	<description>springcloud project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.RC2</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>

		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
		</repository>
	</repositories>

</project>

```
2.application.yml
```
server:
  port: 8001
spring:
  application:
    name: app-provider-a
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8100/eureka,http://localhost:8200/eureka,http://localhost:8300/eureka
    register-with-eureka: true
    fetch-registry: true
security:
  basic:
    enabled: false
  management:
    security:
      enabled: false
```
3.app.class
```
package pofe;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class SpringcloudEurekaClientSchoolingAApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringcloudEurekaClientSchoolingAApplication.class, args);
	}

}

```
然后，可以再接着配置项目b，c，d....类同。

-服务消费者（分布式，无需集群）
1.pom.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.1.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.dingpengfei</groupId>
	<artifactId>springcloud-eureka-client-consumer-a</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>springcloud-eureka-client-consumer-a</name>
	<description>spring project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.RC2</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>

		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
		</repository>
	</repositories>

</project>

```
2.application.yml
```
server:
  port: 8003
spring:
  application:
    name: app-c
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8100/eureka,http://localhost:8200/eureka
    register-with-eureka: true
    fetch-registry: true

#security:
#  basic:
#    enabled: false
#  management:
#    security:
#      enabled: false
#security:
#  management:
#    endpoints:
#      web:
#        exposure:
#          include=*:
```
3.app.class
```
package pofe;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableEurekaClient
public class SpringcloudEurekaClientConsumerAApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringcloudEurekaClientConsumerAApplication.class, args);
	}

	@Bean
	@LoadBalanced
	RestTemplate restTemplate() {
		return new RestTemplate();
	}

}

```
4.controller类
```
package pofe.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;

@RestController
public class CcConsumer {

    @Autowired
    private RestTemplate restTemplate;

    @RequestMapping("/")
    public String index() {
        //因为注册到了服务治理之中，所以url是provider-client+方法映射
        return restTemplate.getForObject("http://app-provider-a/",String.class);
    }
    @RequestMapping("/c")
    public String c() {
        return "c";
    }
}
```

然后，可以再接着配置项目b，c，d....类同。

-zuul网关（集群3台）
1.pom.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.1.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.dingpengfei</groupId>
	<artifactId>springcloud-eureka-zuul</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>springcloud-eureka-zuul</name>
	<description>springcloud project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.RC2</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-zuul</artifactId>
		</dependency>

		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
		</repository>
	</repositories>

</project>

```
2.application.yml
```
###注册 中心
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8200/eureka/
server:
  port: 8010
###网关名称
spring:
  application:
    name: service-zuul
### 配置网关反向代理
zuul:
  routes:
    api-a:
      ### 以 /api-member/访问转发到会员服务
      path: /api-provider-a/**
      serviceId: app-provider-a
    api-b:
      ### 以 /api-order/访问转发到订单服务
      path: /api-provider-b/**
      serviceId: app-provider-b
```
3.app.class
```
package pofe;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableZuulProxy
@EnableEurekaClient
public class SpringcloudEurekaZuulApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringcloudEurekaZuulApplication.class, args);
	}

}

```
4.配置filter器
```
略（具体使用，具体配置）
```
-配置中心
1.pom.xml
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.dingpengfei</groupId>
  <artifactId>spud-eureks-client-config</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>pofe</name>
  <description>spud</description>
  
  <parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.1.RELEASE</version>
	</parent>
	<!-- 管理依赖 -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Finchley.M7</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!-- actuator监控中心 -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		<!--spring-cloud 整合 config-server -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-config-server</artifactId>
		</dependency>
		<!-- SpringBoot整合eureka客户端 -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
		</dependency>
		
	</dependencies>
	<!-- 注意： 这里必须要添加， 否者各种依赖有问题 -->
	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/libs-milestone</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>
</project>
```
2.applicatiom.yml
```
###服务注册到eureka地址
eureka:
  client:
    service-url:
           defaultZone: http://localhost:8100/eureka
spring:
  application:
    ####注册中心应用名称
    name: config-server
  cloud:
    config:
      server:
        git:
          ###git环境地址
          uri: https://git.dev.tencent.com/SH_appearance/IntegratedBlogProject.git 
          ####搜索目录
          search-paths:
            - config  
          password:  
          username: 
      ####读取分支      
      label: master
####端口号      
server:
  port: 8888
```
3.APP.class
```
package pofe;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
@EnableConfigServer
public class EurekaConfigApp {

	public static void main(String[] args) throws Exception {
		SpringApplication.run(EurekaConfigApp.class, args);
	}

}

```
4.读取配置文件(config-client)
```
@RestController
public class IndexController {
	@Value("${name}")
	private String name;

	@RequestMapping("/name")
	private String name() {
		return name;
	}

}

```
5.读取配置中心的application.yml
```
  application:
    ####注册中心应用名称（特别注意：这个服务别名，对应了git服务器配置文件的前置名称-dev.properties，一一对应的关系）
    name:  config-client
  cloud:
    config:
    ####读取后缀
      profile: dev
      ####读取config-server注册地址
      discovery:
        service-id: config-server
        enabled: true
#####    eureka服务注册地址    
eureka:
  client:
    service-url:
           defaultZone: http://localhost:8100/eureka    
server:
  port: 8882
### post请求（手动刷新）
management:
  endpoints:
    web:
      exposure:
        include: "*"
```
6.手动刷新（发送post请求）更改后的刷新!
```
http://127.0.0.1:8882/actuator/refresh 
```
-security（eclipse环境代码）
1.pom.xml
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.dingpengfei</groupId>
  <artifactId>po-ot-security</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>pofe</name>
  <description>this is a sp-ot security
</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.1.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-freemarker</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<optional>true</optional>
			<scope>true</scope>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<fork>true</fork><!-- 如果没有该配置，热部署的devtools不生效 -->
				</configuration>
			</plugin>
			<!-- 自定义配置spring Boot使用的JDK版本 -->
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
```
2.application.yml
```
没有（暂略）
```
3.app.class
```
package pofe;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;

@SpringBootApplication
//@ComponentScan(basePackages = {"pofe"})
public class SpOtApplication {

	public static void main(String[] args) {
		ApplicationContext application = SpringApplication.run(SpOtApplication.class, args);
		System.out.println(application);
	}

}


```
4.ajax域名拦截（但是集成security无效，被security拦截了，需要开放security的权限设置，但是就不能其认证授权的效果了，见下5.代码）
```
package pofe.filter;

import org.springframework.stereotype.Component;

import javax.servlet.*;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component
public class AjaxOriginFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        response.setHeader("Access-Control-Allow-Origin", "*");
        response.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE,PUT");
        response.setHeader("Access-Control-Max-Age", "3600");
        response.setHeader("Access-Control-Allow-Headers", "x-requested-with");
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {

    }
}
```
5.security配置权限
```
package pofe.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Bean;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.password.NoOpPasswordEncoder;
import org.springframework.stereotype.Component;

/**
 * authentication意味认证，如果httpBasic取消会得到认证不通过（走401），但是formLogin只有授权与授权不足（走403），还有一种是没有授权
 * ，即为没有用户，不会出现认证不合格；formLogin的401时走认证不通过的处理函数；
 * 授权与角色权限绑定（需要用户名+密码），认证与用户匹配（只需要用户名），最后一个完整的security需要正面的全配置信息（包括：密码加密，
 * 数据库存储三张表，分别为角色表，权限表，用户表；还有配置403页面，登录自定义页面，错误重定向／error／状态码信息，还有授权成功与失败
 * 页面的重定向页面）
 */
@Component
@EnableWebSecurity
public class Security extends WebSecurityConfigurerAdapter {
    @Autowired
    private MyAuthenticationSuccessHandler successHandler;
    @Autowired
    private MyAuthenticationFailureHandler failHandler;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().withUser("admin").password("123").authorities("ALL_ROLE");
        auth.inMemoryAuthentication().withUser("a").password("123").authorities("A_ROLE");
        auth.inMemoryAuthentication().withUser("kpl").password("kpl123").authorities("WINTER_ROLE");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/admin").hasAnyAuthority("ALL_ROLE")
                .antMatchers("/a").hasAnyAuthority("A_ROLE")
                .antMatchers("/winter").hasAnyAuthority("WINTER_ROLE")
                .antMatchers("/**").fullyAuthenticated()
                .and().formLogin().failureHandler(failHandler);
    }

    @Bean
    public static NoOpPasswordEncoder passwordencoder(){
        return (NoOpPasswordEncoder) NoOpPasswordEncoder.getInstance();
    }
}

```
6.security的web页面所有基本会的重定向配置（三个类配置在一起）
```
package pofe.config;

import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.boot.web.server.ErrorPage;
import org.springframework.boot.web.servlet.server.ConfigurableServletWebServerFactory;
import org.springframework.context.annotation.Bean;

import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpStatus;

@Configuration
public class WebServerAutoConfiguration {
    @Bean
    public ConfigurableServletWebServerFactory webServerFactory(){
        TomcatServletWebServerFactory factory = new TomcatServletWebServerFactory();
        ErrorPage errorPage400 = new ErrorPage(HttpStatus.BAD_REQUEST, "/error/400");
        ErrorPage errorPage401 = new ErrorPage(HttpStatus.UNAUTHORIZED, "/error/401");
        ErrorPage errorPage403 = new ErrorPage(HttpStatus.FORBIDDEN, "/error/403");
        ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error/404");
        ErrorPage errorPage415 = new ErrorPage(HttpStatus.UNSUPPORTED_MEDIA_TYPE, "/error/415");
        ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error/500");
        factory.addErrorPages(errorPage400, errorPage401, errorPage403, errorPage404, errorPage415, errorPage500);
        return factory;
    }
}

package pofe.config;

import org.springframework.security.web.authentication.AuthenticationFailureHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component
public class MyAuthenticationFailureHandler implements AuthenticationFailureHandler {

    @Override
    public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, org.springframework.security.core.AuthenticationException e) throws IOException, ServletException {
        System.out.println("用户认证失败");
        httpServletResponse.sendRedirect("http://dingpengfei.com/index.html");
    }
}

package pofe.config;

import org.springframework.security.core.Authentication;
import org.springframework.security.web.authentication.AuthenticationSuccessHandler;
import org.springframework.stereotype.Component;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Component
public class MyAuthenticationSuccessHandler implements AuthenticationSuccessHandler {
    @Override
    public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
        System.out.println("用户认证成功");
//        httpServletResponse.sendRedirect("http://www.dingpengfei.com/winter.html");

    }
}

```
6.controller类
```
package pofe.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@RestController
public class AdminController {
    @RequestMapping("/error/403")
    public void error(HttpServletResponse response){
        try {
            response.sendRedirect("http://www.dingpengfei.com/index.html");
        }catch (IOException e) {
            e.getMessage();
        }
    }

    @RequestMapping("/admin")
    public String admin(){
        return "admin!";
    }
    @RequestMapping("/a")
    public String a(){
        return "a!";
    }

    @RequestMapping("/winter")
    public void winter(HttpServletResponse response){
        System.out.println("静茹了winter");
        try {
            response.sendRedirect("http://www.dingpengfei.com/winter.html");
        }catch (IOException e) {
            e.getMessage();
        }
    }
}
```

=======================以上至此，微服务环境搭建完成====================
# 修改以上bug版本：（使用eclise导入）
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.dingpengfei</groupId>
  <artifactId>spud-eureka-server</artifactId>
  <version>0.0.1-SNAPSHOT</version>
  <name>pofe</name>
  <description>spud	</description>
  
  <parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.1.RELEASE</version>
	</parent>
	<!-- 管理依赖 -->
	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>Finchley.M7</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<!--SpringCloud eureka-server -->
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
		</dependency>
	</dependencies>
	<!-- 注意： 这里必须要添加， 否者各种依赖有问题 -->
	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/libs-milestone</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>

</project>
```
    

=======================以上如果有任何问题，可能是导包不正确==============

===
        端口号：
            nginx：80
            zuul：8010，8020，8030
            security：8040
            config：8050
            eureks-server：8100，8200，8300
            eureks-client：8001-8009，8101-8109，8201-8209，8301-8309
===

=======================下面细节实现各个服务的分类======================

-springboot整合mybatis（druid+mysql+mapperTk插件+http请求格式）
1.首先上传代码：
-pom.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.1.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.dingpengfei</groupId>
	<artifactId>blog-eureka-cli-liuyan</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>client-liuyan</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.RC2</spring-cloud.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.3.2</version>
		</dependency>
		<!--<dependency>-->
			<!--<groupId>org.springframework.cloud</groupId>-->
			<!--<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>-->
		<!--</dependency>-->
		<!--druid连接池-->
		<dependency>
			<groupId>tk.mybatis</groupId>
			<artifactId>mapper-spring-boot-starter</artifactId>
			<version>2.1.2</version>
		</dependency>
		<dependency>
			<groupId>com.github.pagehelper</groupId>
			<artifactId>pagehelper</artifactId>
			<version>4.1.6</version>
		</dependency>
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>druid</artifactId>
			<version>1.0.25</version>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

	<repositories>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
		</repository>
	</repositories>

</project>

```
-application.yml
```
### app端口号
server:
  port: 8001
  ### 配置所有路径请求形式
  servlet:
    path: /

spring:
  ### 配置文件选择(dev，pro)
  profiles:
    active: dev
  application:
    name: app-liuyan
  ### 国际化（消息源自动配置,springboot默认找出messages）
  messages:
    basename: i18n.messages
  datasource:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://121.196.219.130:3306/pofe?useUnicode=true&characterEncoding=utf8&serverTimezone=GMT%2B8
      username: root
      password: 920104
#      type: com.alibaba.druid.pool.DruidDataSource
  #连接池的配置信息
  ## 初始化大小，最小，最大
  druid:
    initialSize: 5
    minIdle: 5
    maxActive: 20
  ## 配置获取连接等待超时的时间
    maxWait: 60000
  # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
    timeBetweenEvictionRunsMillis: 60000
  # 配置一个连接在池中最小生存的时间，单位是毫秒
    minEvictableIdleTimeMillis: 300000
    validationQuery: SELECT 1 FROM DUAL
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    maxPoolPreparedStatementPerConnectionSize: 20
  # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
    filters: stat,wall,log4j
  # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
    connectionProperties:
      druid:
        stat:
          mergeSql: true;
    stat:
      slowSqlMillis: 5000

### 配置eureka服务
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8100/eureka,http://localhost:8200/eureka,http://localhost:8300/eureka
    register-with-eureka: true
    fetch-registry: true

#security:
#  basic:
#    enabled: false
#  management:
#    security:
#      enabled: false

#mybatis:
#  #  mapper文件
#  mapper-locations: pofe/mapper/Mapper.xml
#  #  实体类
#  type-aliases-package: pofe.pojo

### log日志
logging:
  level:
    pofe: debug
    org.springframework: debug

```
-controller
```
package pofe.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import pofe.entity.LiuyanObject;
import pofe.service.LiuyanService;

import java.util.List;

/**
 * @author a1
 */
@RestController
public class LiuyanController {

    @Autowired
    LiuyanService liuyanService;

    /**
     * 获取留言（web遍历LiuyanObject对象的list集合）
     */
    @GetMapping("getLiuyan")
    public List<LiuyanObject> getLiuyan() {
        return liuyanService.getLiuyan();
    }

    /**
     * 添加留言（web发送LiuyanObject对象的json字符串数据）
     */
    @PostMapping("addLiuyan")
    public String addLiuyan(LiuyanObject obj) {
        try{
            liuyanService.addLiuyan(obj);
        }catch (Exception e){
            return "添加数据失败 -- 原因不详 -- 联系站长";
        }
            return " - success - 200";
    }

    /**
     * 分页测试
     */
    @RequestMapping("itemsPage")//?currentPage='{currentPage}'&pageSize='{pageSize}'
    public List<LiuyanObject> itemsPage(int currentPage,int pageSize){
        return liuyanService.findItemByPage(currentPage, pageSize);

    }
    /**
     * test请求路径
     */
    @RequestMapping("itemsPages/{currentPage}/{pageSize}")//?currentPage='{currentPage}'&pageSize='{pageSize}'
    public List<LiuyanObject> itemsPages(@PathVariable int currentPage,@PathVariable int pageSize){
        return liuyanService.findItemByPage(currentPage, pageSize);

    }

}

```
-service
```
package pofe.service;

import com.github.pagehelper.PageHelper;
import lombok.extern.java.Log;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import pofe.dao.LiuyanDao;
import pofe.entity.LiuyanObject;
import pofe.utils.PageBean;

import java.util.List;

/**
 * @Author:pofe
 * @Description:XXX
 * @Date:2019-01-09
 *
 * @以下代码debug清单:
 *      1.XXX
 *      2.XXX
 */
@Service
@Slf4j
public class LiuyanService {

    @Autowired
    LiuyanDao liuyanDao;

    /**
     * 查询获取所有留言内容
     */
    public List<LiuyanObject> getLiuyan() {
        return liuyanDao.getLiuyan();
    }

    /**
     * 添加一条留言信息
     */
    public void addLiuyan(LiuyanObject object){
        String code = object.getCode();
        String author = object.getAuthor();
        String email = object.getEmail();
        String url = object.getUrl();
        String msg = object.getMsg();
        String time = object.getTime();
        log.info(time);
        LiuyanObject o1 = new LiuyanObject(code,author,email,url,msg,time);
        liuyanDao.addLiuyan(o1);
    }
    /**
     * 分页测试
     */
    @RequestMapping("/itemsPage?currentPage='{currentPage}'&pageSize='{pageSize}'")
    public List<LiuyanObject> itemsPage(@PathVariable int currentPage,@PathVariable int pageSize){
        return findItemByPage(currentPage, pageSize);

    }
    /**
     * 分页封装
     *        reference:
     *              说明一下下面代码的透明度，首先客户端发送了两个参数（第几页，每页显示多少数据），然后下面代码先启动了分页插件功能，
     *              这个分页插件会自动根据springboot项目导包整合配置（@Bean）来配置分页插件，分页插件是在数据库调用之前就已经配置好
     *              相关的分页的，然后使用一个pageBean来装载页面数据，需要根据查询的总数据和全部数据最后，里面注意命名规范问题就可以了。
     *        so：
     *          分页插件主要做了一件事情，那就是在查询数据之前对返回的List结果进行数据包装
     */
    public List<LiuyanObject> findItemByPage(int currentPage,int pageSize) {
        //设置分页信息，分别是当前页数和每页显示的总记录数【记住：必须在mapper接口中的方法执行之前设置该分页信息】
        PageHelper.startPage(currentPage, pageSize);//这个用于客户端指定第几页和每页显示的条数，然后调用原来的查询，就可以返回结果分页了
        List<LiuyanObject> allItems = liuyanDao.getLiuyan();        //全部商品
        int countNums = 10;            //总记录数   假设
        PageBean<LiuyanObject> pageData = new PageBean<>(currentPage, pageSize, countNums);
        pageData.setItems(allItems);
        return pageData.getItems();
    }

}

```
-dao
```
package pofe.dao;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;
import pofe.entity.LiuyanObject;
import pofe.mapper.LiuyanMapper;

import java.util.List;

/**
 * @Author:pofe
 * @Description:XXX
 * @Date:2019-01-09
 *
 * @以下代码debug清单:
 *      1.XXX
 *      2.XXX
 */
@Repository
public class LiuyanDao {

    @Autowired
    LiuyanMapper liuyanMapper;

    /**
     * 返回所有留言的List集合
     */
    public List<LiuyanObject> getLiuyan(){
        return liuyanMapper.getAllLiuyanList();
    }

    /**
     * 插入一行LiuYan对象
     */
    public void addLiuyan(LiuyanObject object){
        try {
//            liuyanMapper.addLiuyan(object);
            liuyanMapper.insert(object);
        }catch (Exception e){
            e.getStackTrace();
        }
    }
}

```
-mapper
```
package pofe.mapper;

import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Select;
import pofe.entity.LiuyanObject;

import java.util.List;

/**
 * @Author:pofe
 * @Description:XXX
 * @Date:2019-01-09
 *
 * @以下代码debug清单:
 *      1.XXX
 *      2.XXX
 */
@Mapper
public interface LiuyanMapper extends tk.mybatis.mapper.common.Mapper<LiuyanObject> {

    @Select("select * from LiuyanTable")
    List<LiuyanObject> getAllLiuyanList();

    @Insert("insert into LiuyanTable(code,author,email,url,msg,time) values('#{obj.code}','#{obj.author}','#{obj.email}','#{obj.url}','#{obj.msg}','#{obj.time}')")
    void addLiuyan(LiuyanObject obj);

}

```
-entity
```
package pofe.entity;

import lombok.Data;

import javax.persistence.Table;
import java.io.Serializable;
/**
 * @Author:pofe
 * @Description:XXX
 * @Date:2019-01-09
 *
 * @以下代码debug清单:
 *      1.XXX
 *      2.XXX
 */
@Data
@Table(name = "LiuyanTable")
public class LiuyanObject implements Serializable {

    private String code;
    private String author;
    private String email;
    private String url;
    private String msg;
    private String time;
    /**
     * 全参数构造函数
     */
    public LiuyanObject(String code, String author, String email, String url, String msg, String time) {
        this.code = code;
        this.author = author;
        this.email = email;
        this.url = url;
        this.msg = msg;
        this.time = time;
    }
}

```
-filter
```
package pofe.filter;

import org.springframework.stereotype.Component;

import javax.servlet.*;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
/**
 * @Author:pofe
 * @Description:
 *          这个允许跨域过滤器只能对普通方法有效，对security框架无效
 * @Date:2019-01-09
 *
 * @以下代码debug清单:
 *      1.XXX
 *      2.XXX
 */

@Component
public class OriginFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        // TODO Auto-generated method stub

    }

    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        HttpServletResponse response = (HttpServletResponse) res;
        response.setHeader("Access-Control-Allow-Origin", "*");
        response.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE,PUT");
        response.setHeader("Access-Control-Max-Age", "3600");
        response.setHeader("Access-Control-Allow-Headers", "x-requested-with");
        chain.doFilter(req, res);

    }

    @Override
    public void destroy() {
        // TODO Auto-generated method stub

    }

}

```
-appStart
```
package pofe;

import com.github.pagehelper.PageHelper;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import java.util.Properties;
//import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
//@EnableEurekaClient
public class ClientLiuyanApplication {

	public static void main(String[] args) {
		SpringApplication.run(ClientLiuyanApplication.class, args);
	}

	//配置mybatis的分页插件pageHelper
	@Bean
	public PageHelper pageHelper(){
		PageHelper pageHelper = new PageHelper();
		Properties properties = new Properties();
		properties.setProperty("offsetAsPageNum","true");
		properties.setProperty("rowBoundsWithCount","true");
		properties.setProperty("reasonable","true");
		properties.setProperty("dialect","mysql");    //配置mysql数据库的方言
		pageHelper.setProperties(properties);
		return pageHelper;
	}
}

```
#### 对于上面这个mybatis衍生的无产品项目总结几点：
1.mybatis的mapper类依赖注入dao中，添加@mapper，继承mapper-mybatis插件，dao中直接接口调用方法，其中mybatis配置之前需要保证java对象和数据库对象名称一致性问题，可以xml，可以java+注解，但是xml需要更多的配置，指定q看后文件关系就可以了，代码中都有痕迹。
2.PageHeper插件，这个看上面代码，springboot整合这个插件，这个插件帮助我们简化的事情就是在查询语句之前进行分页规则，其中需要传递参数，然后将对应查询数据返回到指定的bean对象之中，对后客户端指定怎样的数据分页，就返回怎么的数据分页。
3.关于springboot的请求格式，这里有一些规定：springboot需要配置来区分是rest请求风格还是其它eg：*.do,所以这个需要在application.yml中配置。
4.关于数据库连接池，使用druid就可以了，看上面代码，如果错误更换低一点的版本，ok👌！
