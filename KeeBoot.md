Spring Boot 정리 
================

Bean등록 순서
-------------

- Bean을 등록할 때 첫번째로 @ComponetScan에 의해 등록된다. 
- 어떠한 class에 @Component 어노테이션을 달아놓았다면 그 class가 bean에 등록이 된다.
- 두번째로 @EnableAutoConfiguration에 의해 등록된다. 스프링 부트의 자동 설정은 추가한 jar 의존성을 기반으로 Spring 애플리케이션을 자동으로 설정하려고 시도한다.
- xxxApplication.java 부터 Component scan이 진행되기 때문에  
- xxxApplication class 패키지경로 하위의 bean만 등록이 가능하다. 


annotation 
----------

- @SpringBootApplication  
    - (@EnableAutoConfiguration + @Configuaration + @ComponentScan) 이다.
    - 암시적으로 특정 항목에 대한 기본 검색 패키지를 정의한다.
    - 예를 들어 JPA응용 프로그램을 작성하는 경우 @SpringBootApplication 어노테이션이 달린 클래스의 패키지를 사용하여 @Entity 항목을 검색한다.

- @ComponentScan, @EntityScan, @SpringBootApplication은 모든 jar에서 모든 class를 읽기 때문에 class를 선언할 때 패키지 선언을 해주어야 한다.

- @Configuration 
    - 설정을 위한 어노테이션으로 개발자가 생성한 class를 Bean으로 생성할 때 Single Tone으로 한번만 생성한다.
    - Single Tone이란 한번만 생성하여 계속해서 재사용하는 것. 매번 사용할때마다 다른객체를 사용하게되는 비효율적인 측면을 방지할 수 있다.

- @Component  
    - Bean을 생성할 때 사용하는 어노테이션이다. java class를 Spring Bean이라고 표시하는 역할을 한다. 
- @ComponentScan  
    - @Component, @Configuaration, @Repository, @Service, @Controller, @RestController 어노테이션이 붙은 class를 Bean객체로 만들어준다. 

- @EnableAutoConriguration 

    - 필요한 bean을 유추해서 구성해주는 Class이다. 자동구성 Class는 ClassPath및 앱에 정의한 Bean에 따라 적용 여부가 결정된다. 
    - spring boot는 앱 타입을 유추할 때 사용할 자동 구성 클래스를 모두 spring.factories파일에 몰아 넣는다.
    - 구성할 클래스를 모두 찾아주는 org.springframework.boot.autoconfigure.AutoConfigurationImportSelector Class 중에서 getCandidateConfigurations()는 실제로 자동 구성을 유발한다. 
    - org.springframework.core.io.support.SpringFactoriesLoader Class의 loadFactoryNames() 는 spring-boot-autoconfigure Jar에 있는 META-INF/spring.factories 파일에 프로퍼티 형식으로 열거된 구성 클래스를 읽어들여 List<String> 타입으로 반환한다. 
    - 설정파일에 이름을 따서 명명된 속성들을 찾는것이다.
    - @Configuaration, @Conditionalxxx 어노테이션이 달린 설정파일이나 설정파일의 조건을 읽어서 자동설정을 하게된다. 

- @Conditional  
    - 구성을 활성화 시키기위한 조건을 걸어둘 수 있다. @Configuaration Class가 특정 Class의 유무에 따라 포함되도록 한다.

- @ConditionOnMissingBean 
    - Bean이 존재하지 않을 때 활성화 한다. 덮어쓰기를 방지한다.
    - ex) @ConditionalOnMissingBean(name = "helloConfigSample") helloConfigSample 존재 하지 않을때 실행하라.

- @ConditionOnBean
    - Bean이 존재할 때 실행하는 어노테이션.

- @ConditionalOnProperty
    - prefix 속성에 해당되는 프로퍼티 값이 있으면 실행한다.

- @ConfigurationProperties(prefix = "example")
    - properties파일의 데이터를 해당 prefix 설정값으로 찾아올 수 있다.

- @EnableConfigurationProperties
    - 위 ConditionalOnProperty,ConfigurationProperties Class 등과 함께 사용되는 어노테이션이다. 
    - 프로퍼티의(예를 들어) autoconfig.sample.id 가 있다면 id 값을 저장 해놓기 위한 class가 필요 하다. 그용도로 사용한다.
    - EnableConfigurationProperties 만 있어도 사용은 가능하지만 프로퍼티의 autoconfig.sample.id 설정하지 않았다면 값이 null 로 나온다. 만약 사용한다면 위와같은 방식으로 사용하길 권장한다.
    - ex) @EnableConfigurationProperties(SampleProperties.class)

- @ConditionOnClass
    - 하나 또는 여러 개의 Class가 Class 경로에 있는 경우에만 구성을 활성화 한다.

    - 등등 조건을 위한 Conditional 어노테이션은 엄청나게 많다 document 참조바람.  
    - 한글문서 참조 http://wonwoo.ml/index.php/post/20 
	
                 

내장 웹 서버 이해 
------------------

### 스프링부트는 서버가 아니다.
- Tomcat 객체를 생성하는법 : Tomcat tomcat = new Tomcat(); 
- 포트 설정하기 : 자바코드로 작성 - tomcat.setPort(8090); , application.properties, yml에 작성해도 된다.
- port 를 0으로 설정하게되면 랜덤포트를 사용한다.

### Port값 얻기    
- ApplicationListener<ServletWebServerInitializedEvent> interface를 상속받아 얻을 수 있다.

```java
@Override
public void onApplicationEvent(ServletWebServerInitializedEvent event) {

    ServletWebServerApplicationContext context = event.getApplicationContext();
    System.out.println("port : " + context.getWebServer().getPort());
} 
```

### 간단한 설정만으로 tomcat 대신 다른 was를 사용 가능하다.

```java
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
      <!-- Tomcat 의존성 제외 -->
      <exclusion>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-tomcat</artifactId>
      </exclusion>
  </exclusions>
</dependency>

<!-- Jetty Server 의존성 추가 -->
<dependency>
  <groupId>org.springframework.boot</groupId> 			<!-- 이런방식으로 Jetty Server 이외에 Server도 의존성을 추가해주면 사용 가능하다. -->
  <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

### SSL,HTTPS
- HTTP는 Hypertext Transfer Protocol의 약자이며, HTTPS 에서 S 는 Over Secure Socket Layer의 약자이다. 
- HTTPS는 HTTP보다 보안이 강화된 HTTP이다. 
- HTTP 단점 
- 평문(암호화 하지 않은) 통신이기 때문에 도청이 가능하다. 
- 통신 상대를 확인하지 않기 때문에 위장이 가능하다 완전성을 증명할 수 없기 때문에 변조가 가능하다.
- HTTPS는 직접 TCP와 통신하지않고 SSL과 통신을 하게된다. SSL을 사용함으로써 암호화,증명서,완전성 보호를 이용할 수 있게된다.

### SSL 
- SSH인증서는 클라이언트와 서버간의 통신을 제3자가 보증해주는 전자화된 문서다.
- 통신 내용이 공격자에게 노출되는 것을 막을 수 있다. 
- 클라이언트가 접속하려는 서버가 신뢰 할 수 있는 서버인지를 판단할 수 있다.
- 통신 내용의 악의적인 변경을 방지할 수 있다.

### SSH 인증서 생성 
- Spring Boot SSL key generate L 명령어 수행한 위치에 키스토어가 생성된다. 
- $ keytool -genkey -alias tomcat -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 4000

- application.properties, yml에 설정

```java
server.port=8443
server.ssl.key-store=keystore.p12
server.ssl.key-password=tlagustjq
server.ssl.key-store-type=PKCS12
server.ssl.key-alias=undertow
```

- 이렇게 SSL 키를 등록할 수 있다. 설정이 완료되면 앞으로는 애플리케이션으로의 모든 접근은 https로 해야한다. 
- 추가적으로 http접근도 가능하게 설정하려면 http 요청을 받기 위한 connector를 추가해주면 된다. 
- 대신 properties,yml 에서 https의 포트를 변경해준다. 

```java
@Bean
public ServletWebServerFactory serverFactory() {
    TomcatServletWebServerFactory tomcat = new TomcatServletWebServerFactory();
    tomcat.addAdditionalTomcatConnectors(createStandardConnector()); 
    return tomcat;
}

private Connector createStandardConnector() {
    Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
    connector.setPort(8080);
    return connector;
}
```

- HTTP2 설정은 SSL이 기본적으로 적용되어있는 상태에서 server.http2.enabled=를 true로 할당해주면 된다.
- 추가적으로 해줘야하는 작업은 각 웹서버마다 다르다 (undertow는 https 설정이 되어있으면 추가적인 설정 없이 http2 enable만 true로 할당하면되고, tomcat은 9.X버전과 JDK9 이상을 쓰면 추가적인 설정없이 http2를 적용할 수 있다.)

### 독립적으로 실행 가능한 JAR 
- mvn package 를 하면 실행 가능한 JAR 파일 하나가 생성된다. 
- spring-maven-plugin이 해주는 일이다. (패키징)

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>	<!-- jar,war 파일로 묶기 위한 spring boot 플러그인 -->
        </plugin>												<!-- 이 플러그인 덕분에 mvn package 명령어로 전체 앱을 jar,war 파일로 묶을 수 있다.-->
    </plugins>
</build>
```
	
- 기본적으로 JAVA에는 내장 JAR를 로딩하는 표준적인 방법이 없다.
- mvn package를 해서 생성된 JAR를 풀어보면 lib안에 우리가 의존성을 추가하여 사용한 모든 내장 JAR(library)들이 들어가 있다. 
- org.springframework.boot.loader.jar.JarFile 을 사용해서 내장 JAR를 읽는다. 
- org.springframework.boot.loader.Launcher 를 사용해서 내장 JAR를 실행한다. 그래서 문제없이 mvn package를 하여 만든 JAR 하나만으로도 실행이 되는것이다. 
-  JAR말고도 WAR와 같은 다른 런처도 존재한다. MENIFEST.MF 파일을 살펴보면 Main-Class가 애플리케이션이 아닌 런처로 지정돼있는 것을 알 수 있다. 해당 런처가 Start-Class인 우리의 애플리케이션을 실행한다.


SpringApplication 
-----------------
	
### SpringApplication Custom

- SpringApplication.run(Application.class, args); // 1
- SpringApplication app = new SpringApplication(Application.class, args); // 2
```java
new SpringApplicationBuilder()
    .source(Application.class)
    .run(args); // 3
```

- 1번과 같이 스태틱메소드를 이용해서 앱을 실행하면 커스터마이징을 할 수가 없다. 
- 배너를 수정하는 것처럼 커스터마이징이 필요하다면 2, 3번과 같이 `직접` 애플리케이션을 생성하거나 빌더를 이용한 방법을 사용해야 한다.	

### 기본 로그레벨은 INFO
- 변경법
Eclips를 사용하는 경우 상단탭 Run - Configuarations
Program arguments : --debug 
VM 				  : -Ddebug  라고 설정해주면 로그레벨이 DEBUG로 설정된다.

### FailureAnalyzer  
에러가 발생했을 때, 에러 로그를 보기좋게 출력해준다. 

### ApplicationEvent 등록
- ApplicationConText를 만들기 전에 사용하는 리스너는 @Bean으로 등록할 수 없다. 
- Context는 모든 bean의 정보를 가지고 있다. 
- 스프링부트에서 기본적으로 제공해주는 이벤트가 존재한다. 앱시작,앱컨텍스트를 만들었을 때, refresh 됬을 때, 구동이 됬을 때 등등 

```java
@Component
public class SampleListener implements ApplicationListener<ApplicationStartingEvent> {

    @Override
    public void onApplicationEvent(ApplicationStartingEvent event) {
        System.out.println("=======================");
        System.out.println("Application is Starting");  // 동작 x 
        System.out.println("=======================");
    }
}
```

- 주의해야 할 점은 `이벤트가 발생하는 시점`이다. ApplicationConText가 생성되는 시점을 기준으로, `Context가 생성된 이후` 발생한 이벤트들은 컨테이너의 빈을 `실행할 수 있다.`
    - ApplicationStartingEvent가 발생하는 시점이 컨텍스트 생성 이후라면 SampleListener 빈을 실행할 수 있다.

- `Context가 만들어지기 이전`에 발생한 이벤트는 `문제`가 된다. ApplicationStartingEvent는 애플리케이션 맨 처음에 발생하므로, 해당 시점에 애플리케이션 컨텍스트가 만들어지지 않았다. 
그래서 해당 이벤트가 발생하더라도 SampleListener가 동작하지 않는다.

<hr/>

```java
	public class SampleListener implements ApplicationListener<ApplicationStartingEvent> {

		@Override
		public void onApplicationEvent(ApplicationStartingEvent event) {
			System.out.println("=======================");
			System.out.println("Application is Starting");
			System.out.println("=======================");
		}
	}

	@SpringBootApplication
	public class Application {

		public static void main(String[] args) {
			SpringApplication app = new SpringApplication(Application.class);
			app.addListeners(new SampleListener()); // add 
			app.run(args);
		}
	}
```
- 이를 해결하기 위해서는 코드를 수정해야한다. SpringApplication 인스턴스의 addListeners 메소드를 이용하여 SampleListener를 추가해야한다. 
SampleListener는 빈으로 등록할 필요가 없고, 메인에서 직접 생성하기 때문에 @Component를 지운다. 앱을 실행해보면 컨텍스트 생성 로그가 출력되기 이전에 SampleListener가 실행된다.



```java
@Component
public class SampleListener implements ApplicationListener<ApplicationStartedEvent> {

    @Override
    public void onApplicationEvent(ApplicationStartedEvent event) {
        System.out.println("=======================");
        System.out.println("Started");
        System.out.println("=======================");
    }
}

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(Application.class);
        app.run(args);
    }
}
```
- ApplicationStartedEvent 이벤트는 컨텍스트 생성 로그가 먼저 출력되고 Started 로그가 출력되면서 SampleListener가 실행된다.


### WebApplicationType 설정
- org.springframework.boot.WebApplicationType 열거형 클래스에는 3가지의 타입이 존재한다. 
- SpringApplication의 setWebApplicationType메소드로 앱 타입을 지정할 수 있다. 
- `ServletWebMvc`나 `SpringMvc`가 설정되어있으면 `WebApplicationType.SERVLET`으로 설정된다. 
- `SpringWebFlux`가 설정되어있으면 `WebApplicationType.REACTIVE`으로 설정된다. 
- `둘 다` 없다면 `WebApplicationType.NONE`로 설정된다. 
- SpringMvc와 SpringWebFlux가 `모두` 들어있다면 `WebApplicationType.SERVLET`으로 설정된다. 
- 가장 먼저 서블릿의 존재를 체크하기 때문에 WebFlux가 들어있어도 무시된다.

### Application Argument 사용하기
- Application Argument -> -- 
- VM Argument          -> -D 

```java
@Component
public class SampleListener  {

    public SampleListener(ApplicationArguments arguments) {
        System.out.println(arguments.containsOption("bar"));
    }
}
```
- ApplicationArguments를 이용해 애플리케이션에 전달된 인자를 사용한다.

애플리케이션 실핸한 뒤 뭔가 실행하고 싶을 때 
-----------------------------------------

### ApplicationRunner
```java
@Component
public class SampleListener implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) throws Exception {
        
    }
}
```
- 애플리케이션이 실행된 후에 특정 작업을 수행하기 위해서는 ApplicationRunner을 구현한 빈을 등록해야한다. 
- run 메소드의 인자는 ApplicationArguments가 전달되며 애플리케이션에 전달된 인자에 접근할 수 있다.

### CommandLineRunner
```java
@Component
public class SampleListener implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
        
    }
}
```
- 해당 인터페이스는 인자로 String배열이 전달된다. 
- 백기선 강사님은 ApplicationRunner를 사용하는 것을 추천한다. 
- Runner를 구현한 빈이 여러 개 존재할 경우 @Order를 이용하여 순서를 지정할 수 있다. 
- @Order의 value가 낮을수록 우선순위가 높다.

외부설정
-------

### 프로퍼티 우선 순위 

1. 유저 홈 디렉토리에 있는 spring-boot-dev-tools.properties

2. 테스트에 있는 @TestPropertySource

3. @SpringBootTest 애노테이션의 properties 애트리뷰트

4. 커맨드라인 아규먼트

5. SPRING_APPLICATION_JSON (환경 변수 또는 시스템 프로퍼티)에 들어있는 프로퍼티

6. ServletConfig 파라미터

7. ServletContext 파라미터

8. Java:comp/env JNDI 애트리뷰트

9. System.getProperties() 자바 시스템 프로퍼티

10. OS 환경 변수

11. RandomValuePropertySource

12. JAR 밖에 있는 특정 프로파일용 application.properties

13. JAR 안에 있는 특정 프로파일용 application.properties

14. JAR 밖에 있는 application.properties

15. JAR 안에 있는 application.properties

16. @PropertySource

18. 기본 프로퍼티(SpringApplication.setDefaultProperties)

### application.properties 우선 순위(높은게 낮은걸 덮어 쓴다.)

1. file:./config/

2. file:./

3. classpath:/config/

4. classpath:/

### properties에서 랜덤값 설정하기
- ${random.자료형}
- server.port의 경우 0을 할당해야 가용범위 안의 포트를 찾아서 맵핑해줌

### 타입-세이프 프로퍼티 @ConfigurationProperties
- 여러 프로퍼티를 묶어서 읽어올 수 있다.
- 빈으로 등록해서 다른 빈에 주입할 수 있다.
    - @EnableConfigurationProperties
    - @Component 
    - @Bean
    - 스프링부트 애플리케이션에서는 @EnableConfigurationProperties가 등록이 되어 있으므로 @ConfigurationProperties가 선언되어있는 클래스에 @Component를 추가하여 빈으로 만들어 주기만 하면 된다.
    - (최상단 annotation 설명 부분에 @EnableConfigurationProperties, @ConfigurationProperties 설명 되어있음.)

```java
@Component
@ConfigurationProperties("tester")
public class testerProperties {
    String name;  //tester.name 매칭
    int age;      //tester.age  매칭
    ... //getter, setter 생성 --BEAN 규칙에 따른다
}
```

- @Value와의 차이 

1. type safe하지 않다.
    - @Value의 value에 오타를 칠 수도 있다.
    - 하지만 getter 메소드를 사용함으로써 type safe할 수 있다.

2. Meta-data 지원 여부
    - 위에서 봤듯이 @ConfigurationProperties은 메타데이터를 지원함으로써
    - application.properties(or yml)생성시 자동완성을 지원한다.

3. Relaxed binding(융통성 있는 바인딩) 지원 여부

    - 스프링 부트는 @ConfigurationProperties 빈에 `Environment프로퍼티`를 바인딩 할때 융통성 있는(Relaxed) 규칙을 적용해서
    - Environment프로퍼티 이름과 Bean 프로퍼티 이름이 완벽하게 같지 않아도 된다.
    - 대소문자 차이(ex : PORT vs port)
    - dash(-)구분, (ex : context-path vs contextPath)등을 알아서 구분한다.
    - 다음 4가지는 모두 동일하게 정상 작동하게 된다.

    - context-path : kebab-case (properties,yml에서 사용권장)
    - context_path : Underscore notation (properties,yml의 다른 포맷)
    - contextPath : Standard Camel case
    - CONTEXTPATH : Upper case (시스템 환경 변수에서 권장되는)
        (@Value는 이걸 지원하지 않는다.)

4. SpEL 지원 여부
`@ConfigurationProperties`는 SpEL을 지원하지 않는다.
`@Value`가 SpEL 표현을 지원하지만 프로퍼티 파일에서는 처리되지 않음을 명심하자.

### 프로퍼티 타입 컨버전 
- @Duration
- properties에서는 DataType을 적어주진 않지만 컨버전에 의해 잘 매핑된다.
- 스프링 부트에는 이외에도 스프링 부트만이 제공하는 독특한 컨버전이 존재하는데 그 중 하나가 duration이다.
- java.time.Duration의 Duration 클래스는 자바 8부터 존재하며 특정시간A와 특정시간 B 사이의 시간 차이를 초나 nano초의 시간의 양으로 모델링 한다.

```java
@Component
@ConfigurationProperties("tester")
public class TesterProperties {

//   내용 스킵

    @DurationUnit(ChronoUnit.SECONDS)
    private Duration ssessionTimeout = Duration.ofSeconds(30);
}
```
- 이렇게 하면 properties에 ssessionTimeout이 없으면 기본값 30s를 가지게 되며 ssessionTimeout가 있으면 그값으로 주입이 된다.

### 프로퍼티 값 검증 
- @Validated
- @ConfigurationProperties 클래스들이 스프링의 @Validated어노테이션이 붙으면 스프링 부트가 데이터 검증(validate)을 시도한다.
- @NotEmpty 어노테이션을 달아놓으면 null값을 체크해준다. 

```java
@Component
@Validated
@ConfigurationProperties("tester")
public class TesterProperties {
    @Size(min = 3, max = 15) ///이름 3~15크기
    private String name;
```
- 값을 properties에 다르게 세팅한 후 

```java
2018-10-31 15:04:48.380 ERROR 26696 --- [main] o.s.b.d.LoggingFailureAnalysisReporter   :

***************************
APPLICATION FAILED TO START
***************************

Description:

Binding to target org.springframework.boot.context.properties.bind.BindException: Failed to bind properties under 'tester' to me.rkaehdaos.TesterProperties failed:

    Property: tester.name
    Value: G
    Origin: class path resource [application.properties]:1:13
    Reason: 반드시 최소값 3과(와) 최대값 15 사이의 크기이어야 합니다.
```

이와같은 에러메세지가 나오는데 FailureAnalyzer 덕분에 이쁘게 나온다. 

로깅
----

### 로깅 파사드
    - Commons Logging (Spring은 Commons Logging을 사용함)
    - SLF4j
    - 로깅 파사드는 실제 로깅을 하지 않고, 로거 API들을 추상화 해놓은 인터페이스들이다.
    - 로깅파사드의 장점은 로거들을 바꿔서 사용할 수 있다는 것이다. 

### 로거
    - JUL(Java Utility Logging)
    - Log4j2
    - Logback

- 스프링부트에서 찍히는 로그는 Commons Logging -> SLF4j2 -> Logback의 흐름을 타고 결국은 Logback이 로그를 찍는다. 
- spring-boot-starter-logging 의존성을 통해 알 수 있다. 

### 스프링부트 기본 로깅 
- --debug : 일부 코어 라이브러리(embedded container, Hibernate, Spring Boot)만 디버깅 모드로
- --trace : 전부 다 디 버깅 모드로
- 컬러 출력 : spring.output.ansi.enabled=always
- 파일 출력 : 
    - logging.file 또는 logging.path
    - 로그파일은 기본적으로 10M까지 저장되고, 넘치면 아카이빙하는 등 여러가지 설정도 할 수 있다.
- 로그 레벨 조정 
    - logging.level.패키지 = 로그 레벨

### 커스텀 로깅 
- 커스텀 로그 설정 파일 사용하기

- Logback: logback-spring.xml
    - https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html#howto-configure-logback-for-logging

- Log4J2: log4j2-spring.xml
- JUL(비추천): logging.properties
- Logback extension
- logback-spring.xml을 사용하면 logback.xml을 사용하는 것과 같고, 스프링부트에서 추가로 익스텐션을 사용할 수 있게 제공한다.

- 로거를 Log4j2로 변경하기
    - https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html#howto-configure-log4j-for-logging


테스트
------

- spring-boot-starter-test 의존성을 추가해준다. 

### @springBootTest
- @RunWith(SpringRunner.class)와 같이 써야함.
- 빈 설정은 안해주어도 된다. 알아서 @SpringBootApplication을 찾는다.
- webEnvironment 
    - MOCK : WebApplicationContext를 제공하고 가짜 서블릿 환경을 제공함
    - RANDOM_PORT, DEFINED_PORT : ServletWebServerApplicationContext를 로드하고 진짜 서블릿 환경을 제공함. 내장된 서블릿 컨테이너를 구동하고 random port로 리스닝 한다.
    - NONE : SpringApplication 을 사용하면서 ApplicationContext 이 로드된다. 그러나 서블릿 환경은 제공되지 않는다.

### MockBean 
- ApplicationContext에 들어있는 빈을 Mock으로 만든 객체로 교환한다.

    - ex)  
    ```java
    @MockBean
    private UserController userController;
    ```
    UserController class를 대신하여 Mock객체로 교체한다. 
    
- 모든 @Test 마다 자동으로 리셋.

### 슬라이스 테스트 
- @DataJpaTest  
    - SpringBoot에서 JPA만 테스트할 수 있도록 제공하는 어노테이션 
    - 단위 테스트가 끝날때 마다 자동으로 DB를 롤백시켜준다.

- @WebMvcTest 
    - Controller를 위한 테스트 어노테이션이다. 
    - MockMvc를 자동으로 지원하고 있어 별도의 HTTP서버 없이 Controller테스트를 진행할 수 잇다. 

- @JsonTest
    - 편하게 JSON serialization과 deserialization을 테스트할 수 있다. 

- @RestClientTest
    - 자신이 서버 입장이 아니라 클라이언트 입장이 되는 코드를 테스트할때 유용하다.
    - Apache HttpClient나 Spring의 RestTemplate을 사용하여 외부 서버에 웹 요청을 보내는 경우가 있습니다. 
    - @RestClientTest는 요청에 반응하는 가상의 Mock 서버를 만든다고 생각하면 됩니다.

- @OutputCapture 
    - System.out, System.err 를 출력하는데 사용할 수 있다. 
    - @Rule 어노테이션을 사용하여 OutputCapture 객체를 생성한 후 toString()을 사용하여 출력 가능하다. 

스프링 웹 MVC 
-------------

### HttpMessageConverters

- HTTP 요청 본문을 객체로 변경하거나, 객체를 HTTP응답 본문으로 변경할 때 사용한다.
- @RequestBody
- @ResponseBody
    - @RestController를 사용하게 되면 자동으로 적용된다. 

- Srping MVC는 Http request와 response를 변환하기 위해 HttpMessageConverter 인터페이스를 사용한다.
- 일반적으로 spring framework에서는 Spring MVC를 이용해서 컨트롤러에 값을 주고 받을 때는 HTTP 요청 프로퍼티를 분석하여 그에 따라 특정 클래스로 바인딩 되게끔 하고 특정 객체를 Model Object에 집어 넣어 View를 리턴하는 식으로 주고받게 된다.
- 그러나 메세지 컨버터는 그런 개념이 아니라 HTTP 요청 메세지 본문과 HTTP 응답 메세지 본문을 통째로 하나의 메세지로 보고 이를 처리한다. 
- Spring MVC에서 이러한 작업을 하는데 사용되는 어노테이션이 바로 @RequestBody와 @ResponseBody이다.
- @ResponseBody를 이용하여 파라미터를 받으면 HTTP 요청 메세지 본문을 분석하는 것이 아닌 그냥 하나의 통으로 받아서 이를 적절한 클래스 객체로 변환하게 되고 
- @ResponseBody를 사용하여 특정 값을 리턴하면 View 개념이 아닌 HTTP 응답 메세지에 그 객체를 바로 전달할 수 있다.

### MessageConverter의 종류

- HttpMessageConverters는 HttpMessageConvertersAutoConfiguration Class에 인해서 적용이 된다. 
- StringHttpMessageConverter
- FormHttpMessageConverter
- ByteArrayMessageConverter
- MarshallingHttpMessageConverter
- MappingJacksonHttpMessageConverter
- 각 데이터 타입에 맞는 Converter가 사용된다. 

### ViewResolver
- 스프링부트에 등록 되어있는 스프링 웹 MVC의 ContentNegotiatingViewResolver 가 어떤 contentType일 때 어떤 응답을 보내고, accept header 요청에 의해서 해당 요청에 맞는 응답을 보내는 작업을 알아서 해준다.

## 정적 리소스 지원 

### 정적 리소스 맵핑 "/**". 루트로 맵핑된다.

### 기본 리소스 위치

- classpath:/static
- classpath:/public
- classpath:/resources/
- classpath:/META-INF/resources
- 예) "/hello.html" 접근시 /static/hello.html 응답

- spring.mvc.static-path-pattern: 맵핑 설정 변경 가능
- application.yml에서 spring.mvc.static-path-pattern: /static/** 으로 설정 변경시
- localhost:8080/hello.html => localhost:8080/static/hello.html로 접근
- spring.mvc.static-locations: 리소스 찾을 위치 변경 가능
- 기존의 기본 리소스 위치를 사용하지 않고 변경한 위치만 사용하므로 권장하지 않는 방법이다.
- 이방법 보다는 WebMvcConfigurer를 구현상속받아서 addResourceHandlers로 커스터마이징 하는 방법이 더 좋다. 기본 리소스 위치를 사용하면서 추가로 필요한 리소스위치만 정의해서 사용할 수 있다.
- localhost:8080/m/hello.html 접근시 resources/m/hello.html 리턴
- 여기서 주의할점은 캐시 설정을 따로 해야 한다는 것이다. 기본 리소스들은 기존의 캐싱 전략이 적용되어 있다.

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
​
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/m/**")
            .addResourceLocations("classpath:/m/")
            .setCachePeriod(20);
    }
}
```
- 기본 리소스 위치에 있는 리소스들은 ResourceHttpRequestHandler가 처리하는데, 브라우저에서 304 status을 내려주는 경우가 있다.
- html 파일이 변경되는 순간 파일에 Last-Modified 라는 최종 변경시간이 기록되는데,
- 브라우저에서는 처음에 html파일을 요청하고 200 status를 받으면 해당 시간을 If-Modified-Since에 기록해 놓는다.
- 그리고 브라우저에서 다시 html요청을 하고 응답을 받을때, resopnse 헤더에 넘어오는 Last-Modified 시간이 request 헤더에 보낸 If-Modified-Since와 같다면, html 파일의 변경이 없었다는 의미이므로 다시 리소스를 받아오지 않고 304 status와 캐시된 파일을 내려준다.
- 하지만 html이 변경됐을 경우 response 헤더에 넘어오는 Last-Modified 시간이 request 헤더에 넘긴  If-Modified-Since 시간 이후이므로 새로 리소스를 받아서 200 status를 반환한

### 웹 JAR 
- 자바스크립트 라이브러리를 webjar형태로 dependency를 추가해서 사용할 수 있다.
- 스프링 부트에서 추가로 제공하는 기능이있는데, jquery의 버전이 올라갈 때마다 버전을 일일히 바꿔주지 않아도 된다. 
- 이 기능을 사용하려면 webjars-locator-core 의존성을 추가해야 한다.
- 이것의 내부적인 동작은 springframework의 resource chaining에 의해서 이루어진다.

```javascript
<script src="/webjars/jquery/dist/jquery.min.js"></script>
```


### index 페이지와 파비콘 

- 스프링 부트의 정적 리소스 4가지 기본 위치중 아무 곳이나 index.html을 두면된다.

    - classpath:/static
    - classpath:/public
    - classpath:/resources/
    - classpath:/META-INF/resources

- 그러면 스프링부트는

    - index.html을 찾아보고 있으면 제공
    - index.템플릿 찾아보고 있으면 제공
    - 둘 다 없으면 에러페이지를 내보낸다.

- favicon.io

    - 기본리소스 위치에 위의 파일명으로 위치시킨다.
    - 파이콘 만들기: https://favicon.io/
    - 파비콘은 캐시가 되어있으므로, 크롬에서 캐시비우고 새로고침을 하면 확인할 수 있다.

### HtmlUnit
- HTML 템플릿 뷰 테스트를 보다 전문적으로 할 수있다.
- http://htmlunit.sourceforge.net/  
- http://htmlunit.sourceforge.net/gettingStarted.html  (참조)
- 의존성 추가
- WebClient로 요청페이지,태그,엘리먼트 등을 가져와 단위테스트를 할 수 있다.  
```java
<dependency>    
<groupId>​org.seleniumhq.selenium​</groupId>    
<artifactId>​htmlunit-driver​</artifactId>    
<scope>​test​</scope> 
</dependency> 

<dependency>    
<groupId>​net.sourceforge.htmlunit​</groupId>    
<artifactId>​htmlunit​</artifactId>    
<scope>​test​</scope> 
</dependency
```

### ExceptionHandler

- 스프링 @MVC 예외 처리 방법
    - @ControllerAdvice
    - @ExchangeHandler

- 스프링부트에서는 ExceptionHandler를 기본적으로 등록하여 Exception을 처리하고 있다.
- 기본 예외처리기는 스프링에서 자동적으로 등록하는 BasicErrorController에서 관리한다. (에러발생시 JSON형식으로 리턴)
- 커스텀 Exception 핸들러, 커스텀 Exception 클래스를 만들어서 예외를 처리할 수 있다.
- Http Status 코드에 맞게 예외 발생시 html 문서를 클라이언트에 전송할 수 있다.

- 스프링부트가 제공하는 기본 예외 처리기
- BasicErrorController
    - HTML과 JSON 응답 지원

- 커스터마이징 방법
    - ErrorController 구현

### @ExceptionHandler로 예외처리 하기 
```java
public class AppError {

    private String message;
    private String reason;

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    public String getReason() {
        return reason;
    }

    public void setReason(String reason) {
        this.reason = reason;
    }

    //Exception 발생 시 해당 Exception에 대한 정보를 저장하는 용도로 작성된 클래스
}

public class SampleException extends RuntimeException {
    //RuntimeException을 상속받은 Sample class만들어 놓음.
    //custom Exception 클래스를 표현한 샘플 클래스이다. 
    //ExceptionHandler에서 해당 Exception이 발생할 때 처리할 수 있는 로직을 구현할 수 있다.

}


@Controller
public class SampleController {

    @GetMapping("/hello")
    public String hello(Model model) {
        model.addAttribute("name", "nj");
        return "hello";
        throw new SampleException();
    }

    @ExceptionHandler(SampleException.class)
    public @ResponseBody AppError SampleError(SampleException e) {
        AppError appError = new AppError();
        appError.setMessage("error.app.key");
        appError.setReason("IDK IDK IDK");
        return appError;
    }
}

//사용자 측에서 /hello 요청이 왔을 때 위에서 작성한 SampleException을 발생시킨다. 
//이 때 @ExceptionHandler 어노테이션이 해당 예외를 받아서 처리할 수 있음
//appError 객체에 Exception 관련 정보를 넣고 사용자에게 JSON 형식으로 반환됨
```




### 커스텀 에러 페이지

- 상태 코드 값에 따라 에러 페이지 보여주기
- HTML 문서를 작성할 시 HTTP Status 코드에 맞게 Html 문서를 작성해야 한다.
- HTML 문서의 파일명이 상태코드와 같거나 아니면 5xx 와 같이 패턴을 맞추어서 만들어야 한다.


- src/main/resources/static/error/ 
    - 404.html
    - 5xx.html

```html
<!--404.html-->
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Title</title>
</head>
<body>
<h1>Hello 404</h1>
</body>
</html>
```
```html
<!--5xx.html-->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>Hello 505</h1>
</body>
</html>
```
```java
@Controller
public class SampleController {

    @GetMapping("/hello")
    public String hello()  {
        throw new SampleException();
    }
}
```
Root(/)를 처리하는 코드를 만들지 않았기 때문에 Not Found(Status code : 404)가 리턴되고 그에 맞춰 HTML 문서 404.html 이 반환된다.
/hello 요청을 처리하는 코드를 작성했지만 예외가 발생했기 때문에 Status code : 505 가 리턴되고 그에 맞춰 HTML 문서 5xx. html이 반환된다.

### HATEOAS 
- HATEOAS는 `H`ypermedia `A`s The `E`ngine `O`f `A`pplication `S`tate의 약자로 하이퍼미디어를 REST API의 상태 정보를 관리하기 위한 매커니즘으로 활용하는 것을 말합니다. 
REST API에서 클라이언트에 리소스를 넘겨줄 때 특정 부가적인 리소스의 링크 정보를 넘겨주게 되며 이를 통해 REST API의 리소스 상태에 따른 관리를 진행하게 됩니다.

- 하이퍼 미디어란 : 하이퍼미디어(hypermedia)란 하이퍼텍스트(hypertext)의 커다란 집합체라고 말 할 수 있다. 하이퍼텍스트는 일반적인 텍스트와 별 다른 차이가 없지만 하이퍼텍스트 링크 즉, 
하이퍼링크(hyperlink)라는 다른 문서로의 연결고리를 가진다는 큰 특징을 가지고 있다. 다시말해, 하이퍼텍스트란 어떤 문서내의 특정 단어 또는 문장으로써,그 단어 또는 문장 상에서 사용자가 마우스를 클릭하게 되면 스크린상에 새로운 문서를 나타나게 하는 텍스트를 의미한다.

- HATEOAS를 쓰는 이유는 다음과 같은 기존 REST API의 단점을 보완하기 위해서이다.
1. REST API는 앤드포인트 URL이 정해지고 나면 이를 변경하기 어렵다는 단점이 있습니다. 만일 API의 URL을 변경하게 되면 모든 클라이언트의 URL까지 수정해야하기 때문에 번거로워지므로 기존 다른 API를 지속적으로 추가하게 됩니다. 따라서URL 관리가 어렵게 됩니다. 
2. 전달받은 정적 자원의 상태에 따른 요소를 서버 단에서 구현하기 어렵기 때문에 클라이언트 단에서 이 부분에 대한 로직을 처리해야 합니다.

위 단점들을 links 요소를 통해 href 값의 형태로 보내주기 때문에 자원 상태에 대한 처리를 링크에 있는 URL을 통해 처리할 수 있게됩니다.

- 의존성 추가 
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```

```java 
//테스트 코드 
@RunWith(SpringRunner.class)
@WebMvcTest(SampleController.class)
public class SpringBootTutorialApplicationTests {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        mockMvc.perform(get("/hello"))
                .andDo(print())
                .andExpect(status().isOk())
                .andExpect(jsonPath("$._links.self").exists());
    }
}

// _links는 HATEOAS를 구현하기 위해 스프링 부트에서 생성한 JSON name이다.
// 그리고 그 뒤의 self는 자기 참조를 뜻하는 것을 JSON을 통해서 나타낸 것이다.

```

- 소스코드 
```java
public class Hello {

    private String prefix;

    private String name;

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString(){
        return prefix + " " + name;
    }
}

```
```java
@RestController
public class SampleController {

    @GetMapping("/hello")
    public Resource<Hello> hello(){
        Hello hello = new Hello();
        hello.setPrefix("Hey,");
        hello.setName("saelobi");

        Resource<Hello> helloResource = new Resource<>(hello);
        helloResource.add(linkTo(methodOn(SampleController.class).hello()).withSelfRel());

        return helloResource;
    }
}
//Resource 객체에 HATEOAS를 구현하기 위해 /hello URL의 링크 정보를 추가하는 것을 볼 수 있습니다.
//withSelfRel 메서드를 통해서 해당 URL이 자기 참조인 것을 나타내고 있습니다.
```

### CORS((Cross-Origin Resource Sharing)란 
- HTTP 요청은 기본적으로 Cross-Site HTTP Requests가 가능하다. <img> 태그로 다른 도메인의 이미지파일을 가져오거나, <link>태그로 다른 도메인의 
css를 가져오거나, <script> 태그로 다른 도메인의 javascript 라이브러리를 가져오는것이 모두 가능하다. 하지만 <script></script>로 둘려싸여 있는 
스크립트에서 생성된 Cross-Site HTTP Requests는 Same Origin Policy를 적용받기 때문에 Cross-Site HTTP Requests가 불가능하다. 즉 !! 프로토콜,호스트명,포트가 같아야만 요청이 가능한 것이다. 

### SOP (Same Origin Policy) - 동일 출처 정책 
- SOP란 한 출처에서 로드된 문서나 스크립트가 다른 출처 자원과 상호작용하지 못하도록 제약하는 것을 말한다. 

- CORS는 동일한 출처(Origin: 최초 자원이 서비스된 출처)가 아니여도 다른 출처에서의 자원을 요청하여 쓸 수 있게 허용하는 구조를 뜻한다.
- 보통 보안 상의 이슈(DOM을 통한 취약한 데이터 접근 시도) 때문에 동일 출처(Single Origin Policy)를 기본적으로 웹에서는 준수한다.
- 따라서 최초 자원을 요청한 출처 말고 다른 곳으로 스크립트를 통해 자원을 요청하는 것은 금지된다.
- CORS을 적용하려면 웹 어플리케이션에 그에 따른 처리를 해야하고 스프링 부트에서는 @CrossOrigin 어노테이션 혹은 WebConfig를 통해 CORS를 적용하는 방법을 제공하고있다.
- AJAX가 널리 사용되면서 <script></script>로 둘러싸여 있는 스크립트에서 생성되는 XMLHttpRequest에 대해서도 Cross-Site HTTP Requests가 가능해야한다는 요구가 늘어나자 W3C에서 CORS라는 권고안이 나오게 되었다. 

- Origin ? 
    - URI 스키마 (http, https)
    - hostname(whiteship.me, localhost)
    - 포트 (8080,18080)


### @CrossOrigin 어노테이션으로 적용하는방법

- Client가 될 프로젝트를 따로 생성하고 Port를 8080이외의 포트로 설정해준다. 
```javascript
    $(function(){
        $.ajax("http://localhost:8080/hello")
            .done(function(msg){
                alert(msg);
            })
            .fail(function(){
                alert("fail");
            })
    })
```
- CORS가 적용될 웹 어플리케이션 소스코드 ↓

```java
@RestController
public class SampleController {

    @GetMapping("/hello")
    public String hello(){
        return "hello";
    }
}
```
- ajax로 http://localhost:8080/hello에 요청을 하지만 실패하게 된다. 



- CORS를 적용한 웹 어플리케이션 ↓
```java
@RestController
public class SampleController {

    @CrossOrigin(origins="http://localhost:18080")
    @GetMapping("/hello")
    public String hello(){
        return "hello";
    }
}
```
- 해당 출처에서 스크립트를 통해 자원을 획득할 수 있도록 허용한다. 

#### WebMvcConfigurer를 상속받아서 설정하는 방법 

```java
@RestController
public class SampleController {

    @GetMapping("/hello")
    public String hello(){
        return "hello";
    }
}


@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:18080");
    }
}
```
- java 코드로 WebMvcConfigurer를 상속받는 설정 class파일을 만든 후 addCorsMappings()를 override하여 설정할 수 있다. 
- WebMvcConfigurer를 사용하면 글로벌 설정이 가능하다.


스프링 데이터 
-------------

### 인-메모리 데이터베이스 

- 디스크가 아닌 주 메모리에 모든 데이터를 보유하고 있는 데이터베이스.
- 디스크 검색보다 자료 접근이 훨씬 빠른 것이 큰 장점이다.
- 단점은 매체가 휘발성이기 때문에 DB 서버가 꺼지면 모든 데이터가 유실된다는 단점이 있다.
- 스프링 부트에서 H2, HSQL 같은 인메모리, 디스크 기반 DB를 지원한다.

의존성 추가 

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>

<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>
```
- H2 데이터베이스 의존성을 추가하고 난 후, 설정 파일에 아무 설정이 되어 있지 않으면 스프링 부트는 자동적으로 H2 데이터베이스를 기본 데이터베이스로 채택한다.
- spring-boot-starter-jdbc 의존성을 추가하면 DataSource, JdbcTemplate을 별다른 설정없이 @Autowired 같은 빈 주입 어노테이션만 가지고도 쓸 수 있다.
- 인메모리 데이터베이스 기본 연결정보 확인하는 방법 ! 
    - URL : "testdb"
    - user : "sa"
    - password : ""
- H2-console 사용하는 방법
    - spring-boot-devtools 추가하거나 
    - spring.h2.console.enabled=true만 추가 
    - /h2-console로 접속 (이 path도 바꿀 수 있음)

### DBCP(DataBase Connection Pool)

- DBCP
    - DB에 커넥션 객체를 미리 만들어 놓고 그 커넥션이 필요할 때마다 어플리케이션에 할당하는 개념.
    - 마치 어떤 풀(저장소)에 아이템을 미리 담가놓고 필요할 때 꺼내는 것이다.
    - 커넥션 객체를 만드는 것이 큰 비용을 소비하기 때문에 미리 만들어진 커넥션 정보를 재사용하기 위해 나온 테크닉이다.
    - 스프링 부트에서는 기본적으로 `HikariCP`라는 DBCP를 기본적으로 제공한다. (속도가 가장 빠르다고함)

### DBCP 설정 

- DBCP 설정은 애플리케이션 성능에 중요한 영향을 미치므로 신중히 고려해야하는 부분이다.
- 커넥션 풀의 커넥션 개수를 많이 늘린다고 해서 제대로된 성능이 나오는 것이 아니다. 왜냐하면 커넥션은 CPU 코어의 개수만큼 스레드로 동작하기 때문입니다.
- 스프링에서 DBCP를 설정하는 방법 : spring.datasource.hikari.maximum-pool-size=4  커넥션 객체의 최대 수를 4개로 설정하겠다는 의미이다. 

```s
# application.properties
spring.datasource.hikari.maximum-pool-size=4
#커넥션 객체의 최대 수를 4개로 설정하겠다.
```

## Docker 

- 도커(docker) : 오픈소스 컨테이너 프로젝트
- 기존 가상머신과 다르게 게스트 OS를 사용하지 않고 기존 운영체제와 커널을 공유하기 때문에 빠르게 기존 운영환경과 격리된 환경인 컨테이너를 로딩되고 실행할 수 있다.

### 도커에 My_SQL 설치하기 (Linux 명령어)
```
docker run -p 3306:3306 --name mysql_boot -e MYSQL_ROOT_PASSWORD=1 -e MYSQL_DATABASE=springboot -e MYSQL_USER=saelobi -e MYSQL_PASSWORD=pass -d mysql
```

- 각 옵션에 대한 설명 
    - `-p 3306:3306` → 도커의 3306 포트를 로컬 호스트의 3306 포트에 연결하라는 것을 나타낸다.
    - `--name mysql_boot` → 도커 컨테이너의 이름을 지정한다.
    - `-e MYSQL_ROOT_PASSWORD=1` → MySQL root 계정의 패스워드를 1로 설정한 것을 나타낸다.
    - `-e MYSQL_DATABASE=springboot` → MySQL에 springboot 데이터베이스를 만든다. 
    - `-e MYSQL_USER=saelobi -e MYSQL_PASSWORD=pass` → 유저의 정보를 입력한 옵션
    - `-d mysql` → 백그라운드에서 mysql 컨테이너를 띄우는 명령어 옵션

- 만일 mysql 이미지가 로컬에 없을 경우 mysql 이미지를 다운로드 받아 mysql 컨테이너를 로딩한다.

- docker 명령어 
    - docker ps  : 실행중인 프로세스를 보여줌
    - docker exec -i -t mysql_boot bash : MySQL 컨테이너 상에서 bash를 실행한다. 
    - exec : 컨테이너 외부에서 컨테이너 안의 명령을 실행하는 명령어 
    - -i, -interactive=false : 표준 입력을 활성화하며 컨테이너와 연결되어 있지 않더라도 표준 입력을 유지한다. 
    - -t, --tty=false :  TTY 모드를 사용한다. Bash를 사용하려면 이 옵션을 설정해야한다. 이옵션을 설정하지 않으면 명령을 입력할 수는 있지만 셸이 표시되지 않는다.

- MySQL 명령어 
    - mysql -u shs -p    : 비밀번호 입력후 MySQL 접속 
    - show databases;    : Database의 목록 확인
    - use (databasename) : 해당 database로 접근
    - show tables        : 테이블 보기 
    














