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

이와같은 에러메세지가 나오는데 FailureAnalyzer 덕분에 이쁘게 나온다. 

