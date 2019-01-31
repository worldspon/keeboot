테스트
======

- spring-boot-starter-test 의존성을 추가해준다. 

@springBootTest
----------------

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.MOCK)
@AutoConfigureMockMvc
public class SampleSpringBootTest {

    @Autowired
    MockMvc mockMvc;

    @Test
    public void hello() throws Exception {
        mockMvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string("hello saelobi"))
                .andDo(print());
    }
}
```

- @RunWith(SpringRunner.class)
    - Junit 프레임워크가 내장된 Runner를 실행할 때 @Runwith 어노테이션을 통해 SpringRunner.class라는 확장된 클래스를 실행하라고 지시하는것과 다름없다. 
- @SpringBootTest 
    - @SpringBootApplication을 기준으로 스프링 빈을 등록함과 동시에 Maven 같은 빌드 툴에 의해 추가된 스프링부트 의존성도 제공해 준다.
    - @SpringBootTest 어노테이션에는 webEnvironment라는 값을 통해 웹 어플리케이션 테스트시 Mock으로 테스트할 것인지 실제 톰캣 같은 서블릿 컨테이너를 구동해서 테스트할 것인지를 정할 수 있다.
- webEnvironment 옵션 
    - MOCK : WebApplicationContext를 제공하고 가짜 서블릿 환경을 제공함
    - RANDOM_PORT, DEFINED_PORT : ServletWebServerApplicationContext를 로드하고 진짜 서블릿 환경을 제공함. 내장된 서블릿 컨테이너를 구동하고 random port로 리스닝 한다.
    - NONE : SpringApplication 을 사용하면서 ApplicationContext 이 로드된다. 그러나 서블릿 환경은 제공되지 않는다.
- @AutoConfigureMockMvc
    - Mock 테스트시 필요한 의존성을 제공해준다.


MockBean 
---------
- ApplicationContext에 들어있는 빈을 Mock으로 만든 객체로 교환한다.

    - ex)  
    ```java
    @MockBean
    private UserController userController;
    ```
    UserController class를 대신하여 Mock객체로 교체한다. 
    
- 모든 @Test 마다 자동으로 리셋.

슬라이스 테스트 
---------------
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


HtmlUnit
--------
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