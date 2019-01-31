Spring Boot 정리 
================

Spring Boot 란? 
---------------
스프링 부트(Spring Boot)는 스프링 프레임워크 기반 어플리케이션을 더 빠르고 쉽게 개발할 수 있게 해주는 오픈소스 프로젝트이다.
스프링 프레임워크만으로 개발할 때보다 간단한 설정만으로도 쉽게 웹 어플리케이션을 제작할 수 있다.


Bean등록 순서
-------------

- `첫번째`로 `@ComponetScan`에 의해 등록된다. 
    - 어떠한 class에 @Component 어노테이션을 달아놓았다면 그 class가 bean에 등록이 된다.
    - application.java 부터 Component scan이 진행되기 때문에  
    - application class 패키지경로 하위의 bean만 등록이 가능하다. 
- `두번째`로 `@EnableAutoConfiguration`에 의해 등록된다. 
    - 스프링 부트의 자동 설정은 추가한 jar 의존성을 기반으로 Spring 애플리케이션을 자동으로 설정하려고 시도한다.
    - spring.factories 에 등록되어있는 class들을 모두 Bean으로 등록. 자동설정한다.
    - @Configuration이 달려있는 Class를 Bean으로 등록.
    - @ConditionalOnXXX 이 달려있는 Class를 Bean으로 등록.


annotation 
----------

- @SpringBootApplication  
    - @EnableAutoConfiguration, @Configuaration, @ComponentScan 3개의 어노테이션들을 붙인것과 동일한 기능을 한다. 
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
    - org.springframework.core.io.support.SpringFactoriesLoader Class의 loadFactoryNames() 는 spring-boot-autoconfigure 에 있는 META-INF/spring.factories 에 포함되있는 jar파일들을 찾는다.
    - 그리고 나서 SpringFactoriesLoader는 설정파일의 이름을 따서 명명된 속성들을 찾는다. 
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
-----------------

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


### 독립적으로 실행 가능한 JAR 
- mvn package 를 하면 실행 가능한 JAR 파일 하나가 생성된다. 
- spring-maven-plugin이 해주는 일이다. (패키징)

```java
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>	
        </plugin>												
    </plugins>
</build>
```
- jar,war 파일로 묶기 위한 spring boot 플러그인.
- 이 플러그인 덕분에 mvn package 명령어로 전체 앱을 jar,war 파일로 묶을 수 있다.
	
- 기본적으로 JAVA에는 내장 JAR를 로딩하는 표준적인 방법이 없다.
- mvn package를 해서 생성된 JAR를 풀어보면 lib안에 우리가 의존성을 추가하여 사용한 모든 내장 JAR(library)들이 들어가 있다. 
- org.springframework.boot.loader.jar.JarFile 을 사용해서 내장 JAR를 읽는다. 
- org.springframework.boot.loader.Launcher 를 사용해서 내장 JAR를 실행한다. 그래서 문제없이 mvn package를 하여 만든 JAR 하나만으로도 실행이 되는것이다. 
- JAR말고도 WAR와 같은 다른 런처도 존재한다. MENIFEST.MF 파일을 살펴보면 Main-Class가 애플리케이션이 아닌 런처로 지정돼있는 것을 알 수 있다. 해당 런처가 Start-Class인 우리의 애플리케이션을 실행한다.


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

애플리케이션 실행한 뒤 뭔가 실행하고 싶을 때 
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
- @RequestBody를  이용하여 파라미터를 받으면 HTTP 요청 메세지 본문을 분석하는 것이 아닌 그냥 하나의 통으로 받아서 이를 적절한 클래스 객체로 변환하게 되고 
- @ResponseBody를 사용하여 특정 값을 리턴하면 View 개념이 아닌 HTTP 응답 메세지에 그 객체를 바로 전달할 수 있다.

### MessageConverter의 종류

- HttpMessageConverters는 HttpMessageConvertersAutoConfiguration Class에 인해서 적용이 된다. 
- StringHttpMessageConverter
- FormHttpMessageConverter
- ByteArrayMessageConverter
- MarshallingHttpMessageConverter
- MappingJacksonHttpMessageConverter
- 각 데이터 타입에 맞는 Converter가 사용된다. 

- xml 메시지 컨버터 추가하기 
```java
<dependency>    
    <groupId>​com.fasterxml.jackson.dataformat​</groupId>    
    <artifactId>​jackson-dataformat-xml​</artifactId>    
    <version>​2.9.6​</version> 
</dependency> 
```

### ViewResolver
- ViewResolver : 스프링에서 Controller에서 반환한 값(ModelAndView 혹은 Model)을 통해 뷰를 찾는 역할. 
- ContentNegotiatingViewResolver : 동일한 URI에서 HTTP Request에 있는 Content-type 및 Accept 헤더를 기준으로 다양한 Content-type으로 응답할 수 있게 하는 ViewResolver.
- Controller 단에서 따로 json 타입에 대한 정보를 명시하지 않아도 스프링부트에 등록 되어있는 스프링 웹 MVC의 ContentNegotiatingViewResolver를 통해 자동적으로 json형식으로 데이터를 반환하도록 스프링부트에서 제공한다.
- 이 ViewResolver는 Converter와 연관되어 있어 Content-type을 기준으로 어떤 Converter를 쓸지 결정한다. 


정적 리소스 지원 
---------------

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


