백기선의 스프링 데이터 JPA 강좌
-----------------------------


관계형 데이터베이스와 자바 
------------------------

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

ORM의 개요 Object-Relation Mapping
---------------------------------- 

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

ORM 
----

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


스프링 부트 스타터 JPA
----------------------

- Entitymanager가 내부적으로 하이버네이트를 사용한다
- 우리는 JPA와 하이버네이트 둘다 사용한다. 하지만 
- spring data jpa를 사용하여 간접적으로 사용한다. 
- jpa의존 받았을 경우 hibernate.autoconfiguration에 의해 여러가지 자동설정이 된다. 

- @PersistenceContext : 어노테이션을 사용하면 JPA에서 가장 핵심적인 EntityManager 클래스를 주입받을 수 있다.
- @Transactional : 엔티티매니저와 관련된 오퍼레이션들은 한 트렌젝션 내에서 일어나야된다.
- EntityManager entityManager; //entity manager 가장 핵심적인 class를 주입받을 수 있다.   

```java
entityManager.persist(account); //JPA 스펙 persist: 영속화 (저장한다.)

Session session = entityManager.unwrap(Session.class); 
session.save(account);
```
- session은 하이버네이트 api인데 entitymanager를 이용해 사용가능하다  


```java
#application 구동될때마다 새로 스키마 생성 
spring.jpa.hibernate.ddl-auto=create

#postgres 가 createClub을 지원하지 않아서 warning뜨는 것을 방지 
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
```



엔티티 맵핑 
----------

### 영속성 컨텍스트란 : 엔티티를 저장하는 논리적인 저장공간이다. 영속성 컨텍스트는 엔티티 매니저를 생성할 때 같이 만들어 지며 엔티티 매니저를 통해서 영속성 컨텍스트에 접근할 수 있다. 즉, 엔티티 매니저가 하는 어떤 행위는 영속성 컨텍스트에 반영되고 이것이 최종적으로 DB에 반영된다.

#### 특징
- 식별자 값
    - 영속성 컨텍스트는 엔티티를 식별자 값으로 구분한다(@Id 애노테이션)

- 데이터베이스와의 관계
    - 영속성 컨텍스트에 엔티티가 저장된다고 바로 DB에 반영되는 것이 아니라 일반적으로 트랜잭션이 커밋될 때 DB에 반영된다.(flush)

- 영속성 컨텍스트에 엔티티를 관리할 때의 장점
    - 1차캐시
    - 동일성 보장 (쉽게말해 싱글톤)
    - 트랜잭션이 지원되는 지연 쓰기
    - 변경 감지
    - 지연 로딩



출처: https://feco.tistory.com/93?category=195517 [wmJun]

```java
public class JpaMain {
    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
        EntityManager em = emf.createEntityManager();
        EntityTransaction tx = em.getTransaction();
        try {
            tx.begin();
            logic(em);
            tx.commit();
        } catch(Exception ex) {
            tx.rollback();
        } finally {
            em.close();
        }
        emf.close();
    }
 
    private static void logic(EntityManager em) {
        String id = "2";
        Member member = new Member();
        member.setId(id);
        member.setUsername("yellowh");
        member.setAge(2);
        em.persist(member);
        member.setAge(20);
    }
}
```

#### 생명 주기 
- new / transient(비영속) : 영속성 컨텍스트완느 관련이 없는상태.
- managed (영속)          : 영속성 컨텍스트에 저장된 상태.
- detached (준영속)       : 영속성 컨텍스트에 저장되었다가 분리된 상태.
- removed (삭제)          : 삭제된 상태.

- 비영속 : 엔티티를 생성만 하고 그 어떠한 작업도 하지 않은 상태를 말한다. em.persist(member); 하기전 
```java
Member member = new Member();
        member.setId(id);
        member.setUsername("yellowh");
        member.setAge(2);
```
- 영속 : 엔티티를 영속성 컨텍스트에 저장한 상태 (데이터 검색이 가능한 상태)
```java
TypedQuery<Member> query = em.createQuery("select m from Member m", Member.class);
        List<Member> members = query.getResultList();
        System.out.println(members); 
```
- 준영속 : 영속성 컨텍스트가 관리하던 영속상태의 엔티티를 영속성 컨텍스트가 관리하지 않은 상태. 
- em.detach(); 호출하면 된다.

- 삭제 : 엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제한다. 
- em.remove(member);

#### 영속성 컨텍스트에서 엔티티 조회 
- 영속성 컨텍스트 내부에 존재하는 캐시를 1차 캐시라고 하면 영속 상태의 엔티티는 1차 캐시에 저장된다. 
```java
String id = "2";
Member member = new Member();
member.setId(id);
member.setUsername("yellowh");
member.setAge(2);
em.persist(member);
```
- em.persist()를 하면 1차 캐시에 회원 엔티티를 저장한다. 1차 캐시는 자바컬렉션의  Map형태로이며 Key는 식별자값(id)이며 데이터베이스 기본키와 매핑되어 있다. 또한 Value는 데이터를 의미(객체, Member)가 저장된다.
- em.find(); 조회시에 1차 캐시에 데이터가 없으면 데이터베이스에서 조회를 한다. 이때 1차캐시에도 같이 저장한다. 1차 캐시에 데이터가 존재하면 데이터베이스가 아닌 1차 캐시에서 데이터를 조회한다.
데이터가 변경되지 않고 같은 데이터를 계속 조회한다면 SQL은 데이터베이스에서 계속 데이터를 가져오지만 JPA는 한번만 데이터베이스에 접근하고 나머지는 1차 캐시에서 가져온다. 또한 1차 캐시에서 가져온 데이터는 '==' 비교 연산자로 비교가 가능하다.

#### 영속성 컨텍스트에 엔티티 등록 
- 엔티티 매니저는 트랜잭션을 커밋하기 직전까지 데이터베이스에 엔티티를 저장하지 않고 내부 쓰기 지연 저장소에 저장해 둔다.
- 엔티티를 영속상태로 만들면 동시에 쿼리는 *쓰기지연* 저장소 저장한다. 
- 그 후에 트랜잭션을 커밋하면 엔티티 매니저는 영속성 컨텍스트를 플러시해서 데이터베이스와 동기화 적업을 진행한다. 
- 즉 쓰기 지연 저장소에 저장되어 있는 쿼리를 데이터베이스에 전송하여 동기화하게 된다.

#### 영속성 컨텍스트의 엔티티 수정 
- 엔티티를 영속성 컨텍스트에 저장할 때 최초 상태를 복사(스냅샷)해서 저장한다. 
- 엔티티 매니저가 플러시를 호출하면 엔티티와 스냅샷을 비교하여 변경된 내용이 있으면 수정 쿼리를 쓰지 지연 저장소에 저장한다. 
- 이렇듯 영속성 컨텍스트는 변경감지 기능을 가지고 있다.
- 변경 감지는 영속성 컨텍스트가 관리하는 *영속상태의 엔티티*에만 적용된다.

- *수정전*
MariaDB [test]> select * from MEMBER;
+----+---------+------+
| ID | NAME    | AGE  |
+----+---------+------+
| 2  | yellowh |   20 |
+----+---------+------+

- 수정소스
```java
private static void logic(EntityManager em) {
    Member member = em.find(Member.class, "2");
    member.setUsername("tistory");
    member.setAge(27);
}
```

- *수정후*
MariaDB [test]> select * from MEMBER;
+----+---------+------+
| ID | NAME    | AGE  |
+----+---------+------+
| 2  | tistory |   27 |
+----+---------+------+


#### 영속성 컨텍스트의 엔티티 삭제 
- 영속성 컨텍스트(1차 캐시, 쓰기지연 저장소)에서 쿼리와 엔티티를 제거하며 데이터베이스에 delete쿼리를 전송한다.

#### flush 
- 플러시는 영속성 컨텍스트의 변경내용을 데이터베이스에 반영한다.
- 영속성 컨텍스트 플러시 방법 
    - em.flush();
    - 트랜잭션 커밋시 자동으로 플러시 
    - JPQL쿼리 실행시 자동으로 플러시 

#### 준영속
- 영속상태의 엔티티가 영속성 컨텍스트에서 분리된 상태. 준영속 상태의 엔티티는 영속성 컨텍스트가 제공하는 기능을 사용할 수 없다.

- 준영속을 만드는 방법

    - em.detach(entity) : 특정 엔티티만 준영속상태로
    - em.clear() : 영속성 컨텍스트를 완전히 초기화
    - em.close() : 영속성 컨텍스트 종료 

- 준영속 상태를 영속상태로 변경하는 방법
    - entity = em.merge(entity);


- @Entity 
    - `엔티티`는 객체 세상에서 부르는 이름. 
    - 보통 클래스와 같은 이름을 사용하기 때문에 값을 변경하지 않음. 
    - 엔티티의 이름은 JQL에서 쓰임. 
    - getter setter 가 없어도 테이블 맵핑된다.
 
- @Table 
    - `릴레이션` 세상에서 부르는 이름. 
    - @Entity의 이름이 기본값. 
    - 테이블의 이름은 SQL에서 쓰임. 
 
- @Id 
    - 엔티티의 주키를 맵핑할 때 사용. 
    - 자바의 모든 primitive 타입과 그 랩퍼 타입을 사용할 수 있음 
    - Date랑 BigDecimal, BigInteger도 사용 가능. 
    - 복합키를 만드는 맵핑하는 방법도 있지만 그건 논외로.. 
 
- @GeneratedValue 
    - 주키의 생성 방법을 맵핑하는 애노테이션 
    - 생성 전략과 생성기를 설정할 수 있다. 
    - 기본 전략은 AUTO: 사용하는 DB에 따라 적절한 전략 선택 
    - TABLE, SEQUENCE, IDENTITY 중 하나. 

    - AUTO(default) : JPA 구현체가 자동으로 생성 전략을 결정한다.
    - TABLE : 키 생성 전용 테이블을 하나 만들고 여기에 이름과 값으로 사용할 컬럼을 만들어 데이터베이스의 시퀀스처럼 동작하게 하는 전략. 
        @TableGenerator(name = "POSTGRES_SEQ", table = "MY_SEQUNCES" ,pkColumnValue = "MY_SEQ", allocationSize = 1)
        
    - IDENTITY : 기본키 생성을 데이터베이스에 위임한다. 데이터베이스의 auto_increment와 같은 기능을 사용할 때 쓴다. 
        - 이 전략을 사용하면 JPA는 기본 키 값을 얻어오기위해 데이터베이스를 추가로 조회한다. 따라서 이 전략을 사용하는 엔티티를 새로 
        - 생성하여 식별자 값을 할당하려면 1차 캐시를 넘어서 데이터베이스에서 insert한 후에 기본 키 값을 조회한다. 즉 호출하는 즉시 
        - insert SQL이 데이터베이스에 전달되므로 트랜잭션을 지원하는 *쓰기 지연이 동작하지 않는다.*
        
    - SEQUENCE : 데이터베이스의 특별한 오브젝트 시퀀스를 사용하여 기본키를 생성한다.
        - 시퀀스 생성해주기 : @SequenceGenerator(name="USER_SEQ_GEN", //시퀀스 제너레이터 이름 sequenceName="USER_SEQ", //시퀀스 이름 initialValue=1, //시작값 allocationSize=1 //메모리를 통해 할당할 범위 사이즈)

출처: https://dololak.tistory.com/479 [코끼리를 냉장고에 넣는 방법]

- SEQUENCE 기본 (정리)
    - 순차적인 숫자 발생기
    - 유일한 값을 생성해주는 객체이다. 
    - 시퀀스를 생성하면 기본키와 같이 순차적으로 증가하는 컬럼을 자동으로 생성할 수 있다.
    - 보통 PK값을 생성하기 위해 사용한다.
    - 시퀀스는 테이블과 독립적으로 저장되고 생성된다.

- @Column 
    - unique 
    - nullable 
    - length 
    - columnDefinition 
    - ... 
 
- @Temporal 
    - 현재 JPA 2.1까지는 Date와 Calendar만 지원. 
    - JPA 2.2 부터는 date 타입 더많이 지원 
 
- @Transient 
    - 컬럼으로 맵핑하고 싶지 않은 멤버 변수에 사용. 
 
- application.properries 
- spring.jpa.show-sql=true // sql보기 
- spring.jpa.properties.hibernate.format_sql=true 좀더 편하게 보여줌 

Value 타입 맵핑 
--------------

- 엔티티 타입과 Value 타입 구분 
    - Entity 타입은 독립적값이 존재한다 ID라는 식별자
    - 식별자가 있어야 하는가. 
    - 독립적으로 존재해야 하는가. 
 
- Value 타입 종류 
    - 기본 타입 (String, Date, Boolean, ...) 
    - Composite Value 타입 
    - Collection Value 타입 
        - 기본 타입의 콜렉션 
        - 컴포짓 타입의 콜렉션 
    
- Composite Value 타입 
    - Composite한 value 타입은 조금더 큰 범위의 타입이다. Entity라고 하기는 애매한 존재. 
    - Entity에 속해있는. 생명주기가 Entity타입에 종속적인 타입이다. 
    - 맵핑
    - @Embadable 
    - @Embadded 
    - @AttributeOverrides 
    - @AttributeOverride

```java
@Embedded
@AttributeOverrides({
    @AttributeOverride (name="street", column= @Column(name="home_street"))
})
private Address address;
```
####  맵핑하는법 : Address에 있는 street을 Account 테이블을 생성할 때 home_street으로 만들어 넣겠다.

1대다 맵핑 (관계맵핑)
--------------------

 - @MappedBy :
    - 테이블은 외래키 하나로 두 테이블의 연관관계를 관리하는데 엔티티를 단방향으로 매핑하면 참조를 하나만 사용하고
    - 양방향 관계로 설정하면 객체의 참조는 양쪽에서 하나씩 둘인데, 외래키는 하나이므로 두 엔티티중 하나를 정해서 테이블의 외래키를 관리해야 한다.
    - Many쪽이 owner이며 mappedBy는 @OneToMany쪽의 컬렉션 column에 기술하여 owner가 아님을 정의한다. 
    - mappedBy는 @OneToOne, @OneToMany, @ManyToMany 어노테이션에서 사용할 수 있으며 mappedBy가 없으면 JPA에서 양방향 관계라는것을 모르고 
    두 엔티티의 매핑 테이블을 생성한다. 
    - mappedBy는 OWBER 엔티티의 필드나 속성과 대응된다.
    - ManyToOne 양방향 관계에서 Many측에는 mappedBy요소를 사용할 수 없다.(MANY 쪽이 OWNER)
    - OneToOne 양방향 관계에서 OWNER는 반대쪽(INVERSE SIDE)에 대한 FK를 가지는 쪽이다.
    - ManyToMany 양방향 관계는 양쪽 중 아무나 OWNER가 될 수 있다.




