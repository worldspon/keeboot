외부설정
========

프로퍼티 우선 순위 
-----------------

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

application.properties 우선 순위(높은게 낮은걸 덮어 쓴다.)
--------------------------------------------------------

1. file:./config/

2. file:./

3. classpath:/config/

4. classpath:/

properties에서 랜덤값 설정하기
-----------------------------
- ${random.자료형}
- server.port의 경우 0을 할당해야 가용범위 안의 포트를 찾아서 맵핑해줌

타입-세이프 프로퍼티 @ConfigurationProperties
--------------------------------------------
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

프로퍼티 타입 컨버전 
-------------------
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

프로퍼티 값 검증 
---------------
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

위와같은 에러메세지가 나오는데 FailureAnalyzer 덕분에 이쁘게 나온다. 
