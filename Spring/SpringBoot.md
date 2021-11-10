Spring Boot에 대해 알아보겠습니다.  
Spring Boot를 알아보기 위해선 Spring과의 차이점은 무엇이고, 왜 Spring Boot가 나오게 되었는지의 이유를 알아야 합니다.

그럼 먼저 Spring Framework에 대해 알아보겠습니다.

<br><br>

# Spring Framework
스프링 프레임워크는 대표적으로 의존정 주입(DI, Dependency Injection)과 제어의 역전(IoC, Inversion Of Control)의 기능을 가지고 있습니다. 이러한 개발방식으로 개발한 응용프로그램은 단위 테스트가 용이하기 때문에 보다 퀄리티 높은 프로그램을 개발할 수 있습니다.

또한 스프링이 제공하는 기능중에 관점 지향 프로그래밍(AOP, Aspect Oriented Programming)은 스프링 프레임워크에서 아주 강력한 기능입니다. 객체지향 프로그래밍에서 중요 키포인트는 Class입니다. 반면 AOP에서는 관점(Aspect)입니다. 예를 들어, 기존 프로젝트에 security나 logging 등을 추가하고 싶을 때. 기존 비즈니스 로직에는 손을 대지 않고 AOP를 활용하여 추가할 수 있습니다. 특정 메소드가 끝날 때 호출할 수도 있고, 호출 직전, 메소드가 리턴한 직후 혹은 예외가 발생했을 때 등 여러가지 상황에 활용할 수 있습니다.

<br><br>

# Spring Boot
그럼 이렇게 Spring Framework 가 많은 기능을 제공하는데 왜 Spring boot가 필요했을까요??

기본적으로 Spring Framework는 ransaction Manager, Hibernate Datasource, Entity Manager, Session Factory와 같은 설정을 하는데에 어려움이 많이 있습니다. 

즉, 개발을 시작하기 전에 설정을 위해 해야할 일이 너무 많은 것입니다.

그래서 Spring Boot는 다음과 같은 것을 지원해줍니다.

1. 스프링부트는 자동설정(AutoConfiguration)을 지원합니다.  
    
    만약 DataSource를 설정하려고하면 기존의 Spring에서는 어떻게 해야하는지 보겠습니다.

    ```java
    @Bean
    public DataSource getDefaultDataSource() {
            DataSource dataSource = new DataSource();
            dataSource.setUrl(env.getProperty("jdbc.url"));
            dataSource.setUsername(env.getProperty("jdbc.user"));
            dataSource.setPassword(env.getProperty("jdbc.password"));

            dataSource.setDriverClassName(env.getProperty("jdbc.driverClassName"));
    }

    example.jdbc.url=jdbc:mysql://localhost:3306/test
    example.jdbc.user=root
    example.jdbc.password=test
    example.jdbc.driverClassName=com.mysql.jdbc.Driver

    ```
    
    <br>

    위의 설정 코드를 Springboot의 AutoConfiguration을 적용하여 설정하면 아래와 같이 바뀝니다.

    ```java
    spring.datasource.url=jdbc:h2:~/test:DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
    spring.datasource.driverClassName=org.h2.Driver
    spring.datasource.username=sa
    spring.datasource.password=
    ```

    이렇게 **application.properties**에 간단한 정보만 입력만으로도 설정이 가능하게 됩니다.

2. 스프링부트는 호환되는 버전을 자동으로 찾아 선택해줍니다.
    
    우리가 웹애플리케이션을 개발하는 동안 우리는 사용하려는 jar, 사용할 jar 버전, 함께 연결하는 방법이 필요합니다. 모든 웹 애플리케이션에는 Spring MVC, Jackson Databind, Hibernate 코어 및 Log4j와 같은 유사한 요구사항들이 있고 이러한 jar들의 서로 호환되는 버전들을 따로 선택을 해주어야 했습니다. 이런 복잡도를 줄이기 위해서 스프링 부트는 SpringBoot Starter라고 불리는것을 도입했습니다.

    즉 Spring Boot를 사용하면 Start를 제공하는데 예를들어 Starter를 도입하면 스프링의 jar파일은 Dispatcher Servlet으로 Hibernate의 jar파일은 datasource로 자동 설정해주고, 각각의 jar들이 호환되는 버전을 자동으로 선택해줍니다. (dependency 자동화) 

3. 내부 Tomcat을 가지고 있어 Tomcat을 설치하거나 관리해줄 필요가 없다.


이렇게 Spring Boot는 설정에 대한 강력한 기능을 지원해주므로써 개발자가 좀더 개발에만 집중할 수 있도록 도와주고 있습니다.

다만 Spring Boot가 Spring을 포함하는 더 큰 범위라고 생각하기보다는 Spring Framework를 도와주는 도구라는 개념으로 생각하는 것이 좀 더 정확한 표현인것같습니다.

<br><br>

# @SpringBootApplication

SpringBoot를 시작하면 `@SpringBootApplication`이라는 애노테이션을 볼 수 있습니다. `@SpringBootApplication`은 다양한 기능을 제공해주는데 이에 대해 알아보겠습니다.

@SpringBootApplication = ComponentScan + configuration + EnableAutoConfiguration을 합친 애노테이션이라고 볼 수 있습니다.

<br>

### ComponentScan

`@ComponentScan`은 `@Component` 애노테이션(`@Service, @Repository, @Controller`)을 스캔하여 Bean으로 등록해주는 애노테이션입니다.

<br>

### EnableAutoConfiguration

@EnableAutoConfiguration 은 사전에 정의한 라이브러리들을 Bean으로 등록해주는 애노테이션입니다. 사전에 정의한 라이브러리들 모두가 등록되는것은 아니고 특정 Condition(조건)이 만족될 경우 Bean으로 등록합니다.

> 사전 정의 파일 위치  
Dependencies → spring-boot-autoconfigure → META-INF → spring.factories

<br>

![image1](/Img/SpringBoot.png)

<br>

```java
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
...
```

<br>

`"org.springframework.boot.autoconfigure.EnableAutoConfiguration="` 에 등록된 클래스들이 자동으로 등록되는 Bean입니다. 해당 Class들은 상단에 `@Configuration`이라는 애노테이션을 가지고 있습니다. 이러한 키값을 통하여 명시된 많은 Class들이 AutoConfiguration의 대상이 됩니다.

하지만 모든 Class들이 다 Bean으로 등록되진 않는데요.

각 Bean은 `OnBeanCondition`, `OnClassCondition`, `OnWebApplicationCondition` 애노테이션 조건에 의해 등록 여부가 결정됩니다.

- `@OnBeanCondition` : 특정 Bean이 사전에 생성되어있지 않을 경우에 조건이 만족됩니다.
- `@ConditionalOnBean` : 특정 Bean이 이미 생성되어있을 경우에 조건이 만족됩니다.
- `@ConditonalOnClass` : Classpath에 특정 class가 존재할 경우에 조건이 만족됩니다.

<br>

### Configuration

컨텍스트에 추가 빈을 등록하거나 추가 구성 클래스를 가져올 수 있습니다.

해당 클래스에서 1개 이상의 Bean을 생성하고 있을때는 꼭 **@Configuration** 애노테이션을 사용하여야 한다.

@Bean 애노테이션의 경우 아래와 같은 상황에서 주로 사용한다.

1. 개발자가 직접 제어가 불가능한 라이브러리를 활용할 때
2. 초기에 설정을 하기 위해 활용할 때

<br>

# 정리
Spring을 공부하면서도 느꼈지만 Spring Boot도 정말 많은 기능들이 개발자를 위해 자동화 되어있음을 느낄수 있었다. 그만큼 편리하지만 나중에 버그가 생겼을때, 오류가 발생했을때 잘 대처하기 위해서는 추상화된 많은 부분에 대한 공부는 필수인것 같다. 

꼼꼼하게 추상화된 기능들을 공부해나가면 나중에 많은 도움이 될것 같다.