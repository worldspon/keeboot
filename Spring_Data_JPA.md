Spring-Data, JPA 정리 및 Docker
===============================

인-메모리 데이터베이스 
--------------------

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

DBCP(DataBase Connection Pool)
-------------------------------

- DBCP
    - DB에 커넥션 객체를 미리 만들어 놓고 그 커넥션이 필요할 때마다 어플리케이션에 할당하는 개념.
    - 마치 어떤 풀(저장소)에 아이템을 미리 담가놓고 필요할 때 꺼내는 것이다.
    - 커넥션 객체를 만드는 것이 큰 비용을 소비하기 때문에 미리 만들어진 커넥션 정보를 재사용하기 위해 나온 테크닉이다.
    - 스프링 부트에서는 기본적으로 `HikariCP`라는 DBCP를 기본적으로 제공한다. (속도가 가장 빠르다고함)

DBCP 설정 
---------

- DBCP 설정은 애플리케이션 성능에 중요한 영향을 미치므로 신중히 고려해야하는 부분이다.
- 커넥션 풀의 커넥션 개수를 많이 늘린다고 해서 제대로된 성능이 나오는 것이 아니다. 왜냐하면 커넥션은 CPU 코어의 개수만큼 스레드로 동작하기 때문입니다.
- 스프링에서 DBCP를 설정하는 방법 : spring.datasource.hikari.maximum-pool-size=4  커넥션 객체의 최대 수를 4개로 설정하겠다는 의미이다. 

```s
# application.properties
spring.datasource.hikari.maximum-pool-size=4
#커넥션 객체의 최대 수를 4개로 설정하겠다.
```

Spring-Data-JPA란
-----------------

- ORM은 "관계형 데이터베이스의 구조화된 데이터와 자바와 같은 객체 지향 언어 간의 구조적 불일치를 어떻게 해소할 수 있을까"라는 질문에서 나온 객체-관계 매핑 프레임워크이다.
- 객체와 릴레이션을 매핑할 때 생기는 다양한 문제들을 해결할 수 있는 솔루션이다.
- JPA은 ORM을 위한 자바 EE 표준이며 Spring-Data-JPA는 JPA를 쉽게 사용하기 위해 스프링에서 제공하고 있는 프레임워크이다.
- 추상화 정도는 Spring-Data-JPA -> JPA -> Hibernate -> Datasource (왼쪽에서 오른쪽으로 갈수록 구체화)
- 참고로 Hibernate는 ORM 프레임워크이며 DataSource는 스프링과 연결된 MySQL, PostgreSQL 같은 DB를 연결한 인터페이스이다. 

Spring-Data-JPA 연동 ( + PostgreSQL )
------------------------------------- 

### Docker 

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
    
### MySQL 5.* 최신버전 사용할 때
- 문제 : Sat Jul 21 11:17:59 PDT 2018 WARN: Establishing SSL connection without server's identity verification is not recommended. ​According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set.​ For compliance with existing applications not using SSL the ​verifyServerCertificate property is set to 'false'​. You need either to explicitly disable SSL by setting useSSL=false​, or set ​useSSL=true and provide truststore​ for server certificate verification.
- 해결 : jdbc:mysql:/localhost:3306/springboot?​useSSL=false

### MySQL 8.* 최신버전 사용할 때
- 문제 : com.mysql.jdbc.exceptions.jdbc4.MySQLNonTransientConnectionException : Public Key Retrieval is not allowed 
- 해결 : jdbc:mysql:/localhost:3306/springboot?useSSL=false&​allowPublicKeyRetr ieval=true


### 도커에 PostgreSQL 설치및 연동 - 리눅스 
```
docker run -p 5432:5432 -e POSTGRES_PASSWORD=pass -e POSTGRES_USER=keesun -e POSTGRES_DB=springboot --name postgres_boot -d postgres
```

```
docker exec -i -t postgres_boot bash
실행된 도커의 bash shell에 접속하기 위한 명령어.

su - postgres
psql springboot -U saelobi

postgres로 유저명을 바꾸고 난 후 saelobi 계정으로 docker 커맨드 옵션에 입력했던 springboot DB에 접속할 수 있다.
```

- postgreSQL 의존성 추가 

```java
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
```

```s
spring.datasource.hikari.maximum-pool-size=4

spring.datasource.url=jdbc:postgresql://localhost:5432/springboot
spring.datasource.username=saelobi
spring.datasource.password=pass

# 드라이버가 createClub을 지원하지 않아서 warning 뜨는 것을 방지
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
```

스프링 부트 데이터베이스 초기화 
-----------------------------

### JPA를 사용한 데이터베이스 초기화 
- spring.jpa.hibernate.ddl-auto=  create,create-drop,update,validate,none 옵션을 설정할 수 있다.
    - create : JPA가 DB와 상호작용할 때 기존에 있던 스키마(테이블)을 삭제하고 새로 만듬.
    - create-drop : JPA 종료 시점에 기존에 있었던 테이블을 삭제.
    - update : JPA에 의해 변경된 부분만 변경된다. (백기선씨는 update를 주로 사용하심)
    - validate : 엔티티와 테이블이 정상 매핑되어 있는지만 검증한다. 운영시 주로 사용하고 엔티티에 변경사항이 있어 테이블과 매핑이 안되는 경우 서버 구동시 에러가 발생한다. 
    - none : 초기화 동작을 사용하지 않는다.

- spring.jpa.generate-dll 
    - spring.jpa.hibernate.dll-auto 옵션을 사용할 것인지를 결정하는 프로퍼티. 기본적으로 false로 되어있기 때문에 JPA에 의한 데이터베이스 자동 초기화 기능을 사용하려면 true로 세팅해야한다.

- spring.jpa.show-sql: JPA가 생성한 SQL문을 보여줄 지에 대한 여부를 알려주는 프로퍼티.
- spring.jpa.properties.hibernate.format_sql=true : 콘솔에 표시되는 쿼리를 좀 더 가독성 있게 표시해준다. 
- spring.jpa.properties.hibernate.use_sql_comments=true : 콘솔에 표시되는 쿼리문 위에 어떤 실행을 하려는지 HINT를 표시한다. HINT를 보면 실제 어떤 객체를 이용하여 INSERT/SELECT하는지에 대해 나온다.

### SQL 스크립트 파일을 통한 초기화 방법
- schema.sql 혹은 schema-${platform}.sql을 다음과 같이 추가하여 데이터베이스를 자동적으로 초기화 할 수 있다.
- data.sql 혹은 data-${platform}.sql
- ${platform} 값은 ex) spring.datasource.platform=postgres 로 설정 가능하다. 
- spring.jpa.hibernate.dll-auto를 validate로 정의해도 스프링 부트에서 자동적으로 schema.sql의 SQL문을 실행하기 때문에 자동적으로 테이블이 삭제되었다가 생성됩니다.

### 데이터베이스 마이그레이션 

- 대표적인 마이그레이션 툴은 Flyway와 Liquibase이다.
#### Flyway란
- 오픈소스 마이그레이션 툴이며 java나 c++같은 프로그램의 소스코드는 svn,git과 같은 형상관리 툴로 쉽게 관리할 수 있지만 테이블의 스키마나
- 데이터는 위와같은 툴로 변경이력을 관리할 수 없다.그래서 SQL문을 실행하거나 직접 DB콘솔이나 Toad같은 툴을 통해 직접 수동으로 처리해줘야하는 단점이 있다.
- Flyway는 버전관리 목적인 SCHEMA_VERSION 테이블을 통해 SQL스크립트의 변화를 추적하면서 자동적으로 관리하는방법으로 위와같은 문제를 해결한다.

- resource 하위에 db.migration 디렉토리를 추가해 V1__init.sql 파일을 만든다. (Flyaway가 관리하는 SCHEMA_VERSION역할을 하는 테이블이 됨.)
- SCHEMA_VERSION역할을 하는 테이블은 `꼭` 따라야 하는 네이밍 컨벤션이 있다. 
    1. V숫자__이름.sql
    2. V는 꼭 대문자로.
    3. 숫자는 순파적으로(타임스탬프 권장)
    4. 이름은 가능한 서술적으로

NoSQL 
-----

### Redis 연동하기 
- 레디스는 Key-Value 기반인 인메모리 데이터 저장소로서 주로 캐쉬 솔루션으로 쓰이고 있는 오픈 소스 프로젝트이다.
- 레디스를 이용하게 되면 JVM위에서 동작하지 않고 어떤 데이터를 캐싱할 수 있고,따라서 GC 대상이 되지 않고 그로 인한 오버헤드가 줄어드는 장점이 있다.

- 의존성 추가 
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
- 레디스 도커 설치 및 실행 
    - docker run -p 6379:6379 --name redis_boot -d redis 
    - docker exec -i -t redis_boot redis-cli

- redis 커맨드 
    - keys *  (키값 조회)
    - get {keyname} : {keyname}의 value 조회. 

- 소스코드 
```java
@Component
public class RedisRunner implements ApplicationRunner {

    @Autowired
    StringRedisTemplate redisTemplate;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        values.set("name", "keesun");
        values.set("framework", "spring");
        values.set("message", "hello world");

        //docker에서 redis를 실행한뒤 get name 명령어를 입력, keesun이 나오면 성공. 
    }
}
```

- 스프링 부트에서는 RedisTemplate, StringRedisTemplate를 통해 레디스에 쉽게 접근할 수 있다.
- opsForValue를 통해서 레디스에 Key-Value 기반인 데이터를 캐싱하거나 그 값을 얻어올 수 있다.

### MongoDB 연동하기 
- 몽고DB는 데이터 객체들이 컬렉션 내부에서 독립된 문서로 저장되는, 문서 모델 기반(Document-Based)으로 하는 NoSQL 데이터베이스이다.
- 컬렉션이라는 것은 몽고DB에서 용도가 같거나 유사한 문서들을 그룹으로 묶는 것을 말하며 이 컬렉션들은 기존 SQL의 데이터베이스의 테이블처럼 동작한다.
- 몽고DB의 문서 모델은 JSON 기반 이다. 따라서 유여하게 데이터를 질의,조작할 수 있다.

- 의존성 추가 
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

- 몽고DB 도커 설치 및 실행 
    - docker run -p 27017:27017 --name mongo_boot -d mongo
    - docker exec -i -t mongo_boot bash

- 소스코드 
```java
@Document(collection = "accounts")
public class Account {

    @Id
    private String id;
    private String username;
    private String email;

    public String getId() {
        return id;
    }
    public void setId(String id) {
        this.id = id;
    }
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public String getEmail() {
        return email;
    }
    public void setEmail(String email) {
        this.email = email;
    }
}

@Component
public class AppRunner implements ApplicationRunner {

    @Autowired
    MongoTemplate mongoTemplate;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Account account = new Account();
        account.setEmail("aaa@bbb");
        account.setUsername("aaa");

        mongoTemplate.insert(account);
    }
}
```

- 결과확인 
- 몽고 DB 도커 실행 후 
- mongo 
- db
- test
- db.accounts.find({})

### Neo4j 설치 및 연동 

- Neo4j : 노드간의 연관관계를 영속화 하는데 유리한 그래프 데이터베이스.
- 의존성 추가 
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-neo4j</artifactId>
</dependency>
```

- Neo4j 도커설치 및 실행 

    - docker run -p 7474:7474 -p 7687:7687 -d --name noe4j_boot neo4j 
    - http://localhost:7474/browser

- 스프링 데이터 Neo4J 주요 Class
    - SessionFactory 
    - Neo4jRepositor

    
