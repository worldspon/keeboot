SSL,HTTPS,HTTP2,CORS
==============

SSL,HTTPS
---------

- HTTP는 Hypertext Transfer Protocol의 약자이며, HTTPS 에서 S 는 Over Secure Socket Layer의 약자이다. 
- HTTPS는 HTTP보다 보안이 강화된 HTTP이다. 
- HTTP 단점 
- 평문(암호화 하지 않은) 통신이기 때문에 도청이 가능하다. 
- 통신 상대를 확인하지 않기 때문에 위장이 가능하다 완전성을 증명할 수 없기 때문에 변조가 가능하다.
- HTTPS는 직접 TCP와 통신하지않고 SSL과 통신을 하게된다. SSL을 사용함으로써 암호화,증명서,완전성 보호를 이용할 수 있게된다.

SSL 
---
- SSL 인증서는 클라이언트와 서버간의 통신을 제3자가 보증해주는 전자화된 문서다.
- 통신 내용이 공격자에게 노출되는 것을 막을 수 있다. 
- 클라이언트가 접속하려는 서버가 신뢰 할 수 있는 서버인지를 판단할 수 있다.
- 통신 내용의 악의적인 변경을 방지할 수 있다.

SSL 인증서 생성 
---------------
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

HTTP2
-----
- HTTP2 설정은 SSL이 기본적으로 적용되어있는 상태에서 server.http2.enabled=를 true로 할당해주면 된다.
- 추가적으로 해줘야하는 작업은 각 웹서버마다 다르다 (undertow는 https 설정이 되어있으면 추가적인 설정 없이 http2 enable만 true로 할당하면되고, tomcat은 9.X버전과 JDK9 이상을 쓰면 추가적인 설정없이 http2를 적용할 수 있다.)

CORS
----

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


@CrossOrigin 어노테이션으로 적용하는방법
--------------------------------------

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

WebMvcConfigurer를 상속받아서 설정하는 방법 
-----------------------------------------

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