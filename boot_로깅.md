로깅
====

로깅 파사드
-----------
    - Commons Logging (Spring은 Commons Logging을 사용함)
    - SLF4j
    - 로깅 파사드는 실제 로깅을 하지 않고, 로거 API들을 추상화 해놓은 인터페이스들이다.
    - 로깅파사드의 장점은 로거들을 바꿔서 사용할 수 있다는 것이다. 

로거
----
    - JUL(Java Utility Logging)
    - Log4j2
    - Logback

- 스프링부트에서 찍히는 로그는 Commons Logging -> SLF4j2 -> Logback의 흐름을 타고 결국은 Logback이 로그를 찍는다. 
- spring-boot-starter-logging 의존성을 통해 알 수 있다. 

스프링부트 기본 로깅 
-------------------
- --debug : 일부 코어 라이브러리(embedded container, Hibernate, Spring Boot)만 디버깅 모드로
- --trace : 전부 다 디 버깅 모드로
- 컬러 출력 : spring.output.ansi.enabled=always
- 파일 출력 : 
    - logging.file 또는 logging.path
    - 로그파일은 기본적으로 10M까지 저장되고, 넘치면 아카이빙하는 등 여러가지 설정도 할 수 있다.
- 로그 레벨 조정 
    - logging.level.패키지 = 로그 레벨

커스텀 로깅 
-----------
- 커스텀 로그 설정 파일 사용하기

- Logback: logback-spring.xml
    - https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html#howto-configure-logback-for-logging

- Log4J2: log4j2-spring.xml
- JUL(비추천): logging.properties
- Logback extension
- logback-spring.xml을 사용하면 logback.xml을 사용하는 것과 같고, 스프링부트에서 추가로 익스텐션을 사용할 수 있게 제공한다.

- 로거를 Log4j2로 변경하기
    - https://docs.spring.io/spring-boot/docs/current/reference/html/howto-logging.html#howto-configure-log4j-for-logging