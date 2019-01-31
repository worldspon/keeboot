Spring REST Client/Spring Boot 운영 
===================================

RestTemplate와 WebClient
------------------------

- Spring 기반 프로젝트를 진행하면 컴포넌트 내부에서 URL을 요청해야하는 경우가 생긴다.
- Spring 에서는 Http요청을 간단하게 이용할 수 있도록 Blocking I/O 기반의 `RestTemplate`, Non-Blocking I/O 기반의 `WebClient` 모듈을 제공하고 있다. 

### RestTempate
    - Blocking I/O 기반의 Synchronous API  
    - RestTemplateAutoConfiguration 
- 프로젝트에 spring-web 모듈이 있다면 RestTemplate​Builder​를 빈으로 등록해 줍니다.

- 예제 
```java
@RestController
public class HelloController {

    @GetMapping("/hello")
    public String hello() throws InterruptedException {
        Thread.sleep(5000);
        return "hello";
    }

    @GetMapping("/world")
    public String world() throws InterruptedException {
        Thread.sleep(3000);
        return "world";
    }
}
```
```java
@Component
public class RestRunner implements ApplicationRunner {

    @Autowired
    RestTemplateBuilder restTemplateBuilder;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        RestTemplate restTemplate = restTemplateBuilder.build();

        StopWatch stopWatch = new StopWatch();
        stopWatch.start();

        String helloResult = restTemplate.getForObject("http://localhost:8080/hello", String.class);
        System.out.println(helloResult);

        String worldResult = restTemplate.getForObject("http://localhost:8080/world", String.class);
        System.out.println(worldResult);

        stopWatch.stop();
        System.out.println(stopWatch.prettyPrint());
    }
}
```

### WebClient
- webClien는 Non-Blocking I/O 기반이기 때문에 각 Http 요청이 비동기적으로 발생하게 됩니다. 
- 따라서 위 RestTemplate를 이용하여 Http 요청을 진행했을 때와 다르게 동작하게 되며 총 합쳐 대략 8초 정도가 걸리는 것이 아닌 각각 5초, 3초 걸리는 Http 요청을 동시에 처리하게 됩니다.
- Mono는 WebClient의 결과를 0 또는 1개의 결과를 받는 추상클래스며 Publisher 인터페이스를 구현하여 작성되었습니다. 이 Publisher는 바로 즉각적으로 로직을 실행하는 것이 아닌 subscribe 메서드를 통해 결과를 받아올 코드가 실행될 시 그때서야 로직을 실행하게 됩니다.

```java
@Component
public class RestRunner implements ApplicationRunner {

    @Autowired
    WebClient.Builder webClientBuilder;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        WebClient webClient = webClientBuilder.build();

        StopWatch stopWatch = new StopWatch();
        stopWatch.start();

        Mono<String> helloMono = webClient.get().uri("http://localhost:8080/hello")
                                    .retrieve().bodyToMono(String.class);
        helloMono.subscribe(s-> {
            System.out.println(s);
            if(stopWatch.isRunning()){
                stopWatch.stop();
            }
            System.out.println(stopWatch.prettyPrint());
            stopWatch.start();
        });

        Mono<String> worldMono = webClient.get().uri("http://localhost:8080/world")
                                    .retrieve().bodyToMono(String.class);
        worldMono.subscribe(s -> {
            System.out.println(s);
            if(stopWatch.isRunning()){
                stopWatch.stop();
            }
            System.out.println(stopWatch.prettyPrint());
            stopWatch.start();
        });
    }
}

```

커스터마이징  
------------
```java
public class WebclientApplication {

    public static void main(String[] args) {
        SpringApplication.run(WebclientApplication.class, args);
    }

    @Bean
    public WebClientCustomizer webClientCustomizer() {
        return webClientBuilder ->  webClientBuilder.baseUrl("http://localhost:8080");
    }

    @Bean
    public RestTemplateCustomizer restTemplateCustomizer() {
        return restTemplate -> restTemplate.setRequestFactory(new HttpComponentsClientHttpRequestFactory());
    }
}
``` 

- WebClient를 이용하는 경우 WebClientCustomizer를 반환하는 WebClientBuilder를 통하여 WebClient에 대한 설정을 할 수 있습니다. 위
- 예시는 baseUrl을 설정함으로써 WebClient를 통한 Http 요청을 할 때 주소를 생략하고 해당 자원을 요청하는 (/hello, /world) 부분만 명시하도록 한 것입니다.
- tRestTemplate도 마찬가지로 커스터마이징이 가능하며 위 예시는 아파치의 HttpClient를 쓰도록 커스터마이징 한 것입니다. 기본적으로는 java.net.HttpURLConnection을 사용하지만 위 설정을 통해 HttpClient로 Http 요청을 하도록 바꾼 것입니다.

### Actuator

- 스프링 부트 어플리케이션은 actuator라는 모듈을 통해 어플리케이션 상태를 종합적으로 정리해서 제공해준다.
- 이를 통해 스프링 부트 어플리케이션 운영을 손쉽게 할 수 있다. 
- actuator는 다양한 Endpoints(사용자나 디바이스같은 IT 서비스의 최종 목적지)를 제공하여 이 다양한 Endpoints를 통해 어플리케이션의 운영 정보를 알 수 있다. 
- JMX 또는 HTTP를 통해 접근 가능하다. 
- shutdown을 제외한 모든 Endpoint는 기본적으로 활성화 상태이다. 
- 활성화 옵션 조정
    - management.endpoints.enabled-by-default=false
    - management.endpoint.info.enabled=true

- 의존성 추가 
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### JMX와 HTTP 

- 의존성 추가
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

- JConsole 사용하기 
    - 터미널에서 jconsole을 입력하면 된다.
- VisualVm 사용하기 
    - https://visualvm.github.io/download.html 다운로드 후 사용 

- HTTP 사용하기
    - http://localhost:8080/actuator 접속
    - health와 info를 제외한 대부분의 Endpoint가 기본적으로 비공개 상태일 것이다. 
    - 공개 옵션 설정 
        - management.endpoints.web.exposure.include=*
        - management.endpoints.web.exposure.exclude=env,beans


### 스프링부트 Admin

- 스프링 Admin은 actuator 정보를 UI로 쉽게 볼 수 있는 오픈소스 프로젝트이다. 

### Admin 서버 설정 
- 의존성 추가 
```java
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
    <version>2.1.1</version>
</dependency>
```
```java
@SpringBootApplication
@EnableAdminServer
public class SpringAdminApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringAdminApplication.class, args);
    }

}
```

### Client 설정 

- 의존성 추가
```java
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>2.1.1</version>
</dependency>
```
```java
spring.boot.admin.client.url=http://localhost:8080
management.endpoints.web.exposure.include=*
server.port = 18080
```
- 로컬에서 테스트할 경우 포트가 겹치니 바꿔줌. 

- Admin 서버와 Client 서버를 구동시킨 후 http://localhost:8080 주소로 접속하게되면 Spring Boot Admin 페이지가 나온다.
### 반드시 Spring Security와 같이 사용하여야 한다. 노출되었을 때, 위험한 정보들이 너무나도 많다. 


