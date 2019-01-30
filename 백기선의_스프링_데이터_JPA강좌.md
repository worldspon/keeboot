백기선의 스프링 데이터 JPA 강좌
-----------------------------


## 관계형 데이터베이스와 자바 

-p 포트 맵핑 포스트그레스는 5432를 사용한다 내 로컬호스트 5432로 매핑 
-e 환경변수
-i interective 모드 
-t 타겟 

- 도커 postgres 설치 
docker run -p 5432:5432 -e POSTGRES_PASSWORD=pass -e POSTGRES_USER=shs -e POSTGRES_DB=springdata --name postgres_boot -d postgres
- 도커 postgres 실행 
docker exec -i -t postgres_boot bash
- root에서 postgres DB 연결
psql -U keesun springboot
- table 확인 
\dt 

- ORM을 사용하지 않을 때 불편한점 
    - sql 문법이 DB마다 다르다
    - 반복적인 코드가 발생한다.
    - Lazy Loading 전략을 사용하기가 쉽지 않다. 

## ORM의 개요 Object-Relation Mapping 
- jdbc 사용 
```java
try(Connection connection = DriverManager.getConnection(url, username, password)) {             

    System.out.println("Connection created: " + connection);             
    String sql = "INSERT INTO ACCOUNT VALUES(1, 'keesun', 'pass');";             

    try(PreparedStatement statement = connection.prepareStatement(sql)) {
        statement.execute();             
    }         
}
```
- 도메인 모델 사용  
```java
Account account = new Account(“keesun”, “pass”); 
accountRepository.save(account);
```

### ORM 
- ORM은 애플리케이션의 클래스와 SQL 데이터베이스의 테이블 사이의 `​맵핑 정보를 기술한 메타데이터​`를 사용하여, 자바 애플리케이션의 객체를 SQL 데이터베이스의 테이블에 자동으로 (또 깨끗하게) 영속화​ 해주는 기술입니다.
  
- 장점 
    - 생산성
    - 유지보수성
    - 성능
        - 객체와 DB 사이에는 캐쉬가 존재하는데 객체의 변화를 감지하여 정말로 DB에 반영해야되는 시점에만 반영을 한다. 
        - 만약 비밀번호를 INSERT한다고 하자. 한 transection 내에서 비밀번호의 변경이 4번 일어나게될 경우 하이버네이트는 쿼리를 한번 날리지만 SQL로 구현할 경우 INSERT 쿼리를 한번 날리고 UPDATE쿼리를 3번 날리게 된다.
    - 벤더 독립성 
        - 어떠한 DB를 쓰느냐에따라 SQL문법이 달라진다. 벤더마다 SQL문법이 다르다.
        - 하이버네이트는 어떠한 DB에 맞게 SQL을 생성하는지만 알려주면 된다. 그것을 다이렉트라고 부른다. (방언)
- 단점
    - 학습비용이 어마어마어마하다 ~ 





