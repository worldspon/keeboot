백기선의 스프링 데이터 JPA 강좌
=============================


관계형 데이터베이스와 자바 
------------------------

### postgres,docker 설정 
> -p 포트 맵핑 포스트그레스는 5432를 사용한다 내 로컬호스트 5432로 매핑 
> -e 환경변수
> -i interective 모드 
> -t 타겟 

- 도커 postgres 설치 
> docker run -p 5432:5432 -e POSTGRES_PASSWORD=pass -e POSTGRES_USER=shs -e POSTGRES_DB=springdata --name postgres_boot -d postgres
- 도커 postgres 실행 
> docker exec -i -t postgres_boot bash
- root에서 postgres DB 연결
> psql -U keesun springboot
- table 확인 
> \dt 

```java
#application 구동될때마다 새로 스키마 생성 
spring.jpa.hibernate.ddl-auto=create

#postgres 가 createClub을 지원하지 않아서 warning뜨는 것을 방지 
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
```

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
        - 하이버네이트는 어떠한 DB에 맞게 SQL을 생성하는지만 알려주면 된다. 그것을 dialect라고 부른다. (방언)
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


엔티티 맵핑 
----------


- @Entity 
    - **엔티티**는 객체 세상에서 부르는 이름. 
    - 보통 클래스와 같은 이름을 사용하기 때문에 값을 변경하지 않음. 
    - 엔티티의 이름은 JPQL에서 쓰임. 
    - getter setter 가 없어도 테이블 맵핑된다.
 
- @Table 
    - **릴레이션** 세상에서 부르는 이름. 
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
    > @TableGenerator(name = "POSTGRES_SEQ", table = "MY_SEQUNCES" ,pkColumnValue = "MY_SEQ", allocationSize = 1)
        
    - IDENTITY : 기본키 생성을 데이터베이스에 위임한다. 데이터베이스의 auto_increment와 같은 기능을 사용할 때 쓴다. 
        - 이 전략을 사용하면 JPA는 기본 키 값을 얻어오기위해 데이터베이스를 추가로 조회한다. 
        - 따라서 이 전략을 사용하는 엔티티를 새로 생성하여 식별자 값을 할당하려면 1차 캐시를 넘어서 데이터베이스에서 insert한 후에 기본 키 값을 조회한다. 
        - 즉 호출하는 즉시 insert SQL이 데이터베이스에 전달되므로 트랜잭션을 지원하는 **쓰기 지연이 동작하지 않는다.**
        
    - SEQUENCE : 데이터베이스의 특별한 오브젝트 시퀀스를 사용하여 기본키를 생성한다.
        - 시퀀스 생성해주기 : @SequenceGenerator(name="USER_SEQ_GEN", //시퀀스 제너레이터 이름 sequenceName="USER_SEQ", //시퀀스 이름 initialValue=1, //시작값 allocationSize=1 //메모리를 통해 할당할 범위 사이즈)

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


영속성 컨텍스트란/ 엔티티 상태와 Cascade
--------------------------------------

### Persistence Context (영속성 컨텍스트)란 : 엔티티를 저장하는 논리적인 저장공간이다. 영속성 컨텍스트는 엔티티 매니저를 생성할 때 같이 만들어 지며 엔티티 매니저를 통해서 영속성 컨텍스트에 접근할 수 있다. 즉, 엔티티 매니저가 하는 어떤 행위는 영속성 컨텍스트에 반영되고 이것이 최종적으로 DB에 반영된다.

#### 특징
- 식별자 값
    - 영속성 컨텍스트는 엔티티를 식별자 값으로 구분한다(@Id 애노테이션)

- 데이터베이스와의 관계
    - 영속성 컨텍스트에 엔티티가 저장된다고 바로 DB에 반영되는 것이 아니라 일반적으로 트랜잭션이 커밋될 때 DB에 반영된다.

- 영속성 컨텍스트에 엔티티를 관리할 때의 장점
    - 1차캐시
    - 동일성 보장 (쉽게말해 싱글톤)
    - 트랜잭션이 지원되는 지연 쓰기
    - 변경 감지
    - 지연 로딩

### 엔티티 상태 

#### Transient: JPA가 모르는 상태
- 데이터베이스에 들어갈지 안들어갈지도 전혀 모르는 상태
```java
new Object() //Acount 객체를 선언해놓고  set() 메소드로 set만 해놓은 상태 
```

#### Persistent: JPA가 관리중인 상태
- Persistent 상태가 됐다고 바로 Insert가 발생해서 데이터베이스에 저장하지 않음
- Persistent 에서 관리하고 있던 객체가 데이터베이스에 넣는 시점에 데이터를 저장함
```java
Session.save(), Session.get(), Session.load(), Query.iterate() ...
Session.update(), Session.merge(), Session.saveOrUpdate()
```
- **영속성 컨텍스트에서 엔티티 조회**
    - 영속성 컨텍스트 내부에 존재하는 캐시를 1차 캐시라고 하면 영속 상태의 엔티티는 1차 캐시에 저장된다. 
    - 1차 캐시는 자바컬렉션의  Map형태로이며 Key는 식별자값(id)이며 데이터베이스 기본키와 매핑되어 있다. 또한 Value는 데이터를 의미(객체, Member)가 저장된다.
    - 조회시에 1차 캐시에 데이터가 없으면 데이터베이스에서 조회를 한다. 이때 1차캐시에도 같이 저장한다. 1차 캐시에 데이터가 존재하면 데이터베이스가 아닌 1차 캐시에서 데이터를 조회한다.
    - 데이터가 변경되지 않고 같은 데이터를 계속 조회한다면 SQL은 데이터베이스에서 계속 데이터를 가져오지만 JPA는 한번만 데이터베이스에 접근하고 나머지는 1차 캐시에서 가져온다. 또한 1차 캐시에서 가져온 데이터는 '==' 비교 연산자로 비교가 가능하다.

- **영속성 컨텍스트에 엔티티 등록**
    - 엔티티 매니저는 트랜잭션을 커밋하기 직전까지 데이터베이스에 엔티티를 저장하지 않고 내부 쓰기 지연 저장소에 저장해 둔다.
    - 엔티티를 영속상태로 만들면 동시에 쿼리는 **쓰기지연** 저장소 저장한다. 
    - 그 후에 트랜잭션을 커밋하면 엔티티 매니저는 영속성 컨텍스트를 플러시해서 데이터베이스와 동기화 적업을 진행한다. 
    - 즉 쓰기 지연 저장소에 저장되어 있는 쿼리를 데이터베이스에 전송하여 동기화하게 된다.

- **영속성 컨텍스트의 엔티티 수정**
    - 엔티티를 영속성 컨텍스트에 저장할 때 최초 상태를 복사(스냅샷)해서 저장한다. 
    - 엔티티 매니저가 플러시를 호출하면 엔티티와 스냅샷을 비교하여 변경된 내용이 있으면 수정 쿼리를 쓰지 지연 저장소에 저장한다. 
    - 영속성 컨텍스트는 변경감지 기능을 가지고 있다.
    - 변경 감지는 영속성 컨텍스트가 관리하는 **영속상태의 엔티티**에만 적용된다.

- **1차 캐시**: Persistent Context(EntityManager, Session)에 인스턴스를 넣은 것
> 아직 저장이 되지 않은 상태에서 다시 인스턴스를 달라고 하면 이미 객체가 있으므로 데이터베이스에 가지 않고 캐시하고 있는 것을 줌

- **Dirty Checking** : 이 객체의 변경사항을 계속 감지
- **Write Behind** : 객체의 상태의 변화를 데이터베이스에 최대한 늦게 가장 필요한 시점에 적용을 함
> 원래 가지고 있던 객체의 값과 동일한경우 변경사항을 적용하지 않음

#### Detached: JPA가 더이상 관리하지 않는 상태
- Transaction 이 끝났을 때, 이미 데이터베이스에 저장이되고 Session이 끝난 상태
```java
Session.evict(), Session.clear(), Session.close()
```

#### Removed: JPA가 관리하긴 하지만 삭제하기로 한 상태

```java
Session.delete()
```
- 영속성 컨텍스트(1차 캐시, 쓰기지연 저장소)에서 쿼리와 엔티티를 제거하며 데이터베이스에 delete쿼리를 전송한다.

cascade 
-------
- 엔티티의 상태변화를 전파시키는 옵션.
- 영속성 전이라는 것은 연관관계를 맺은 엔티티들도 함께 준영속 혹은 비영속상태에서 영속상태로 만들어 CRUD작업이 가능하도록 하는 것이다.
- 부모 엔티티를 저장할 때 연관된 자식 엔티티도 함께 저장할 수 있다. 또는 부모 엔티티를 삭제하면 연관된 자식 엔티티도 데이터도 삭제할 수 있다.

- cascade 옵션 종류 

    - ALL : 모두 적용 
    - PERSIST : 영속
    - MERGE : 병합 
    - REMOVE : 삭제 
    - REFRESH : REFRESH 
    - DETACH : DETACH 


#### cascade상태 테스트 
- 1차 캐쉬에 저장된 인스턴스 가져오는 테스트

```java
@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        /** Transient 상태 **/
        Account account = new Account();
        account.setUsername("freelife");
        account.setPassword("hibernate");

        Study study = new Study();
        study.setName("Spring Data JPA");

        account.addStudy(study);

        /** Persistent 상태 **/
        session.save(account);
        session.save(study);

        // 데이터베이스에 가지 않고 이미 1차 캐쉬에 저장된 인스턴스를 가져옴 
        Account freelife = session.load(Account.class, account.getId());
        freelife.setUsername("ironman");
        System.out.println("=====================");
        System.out.println(freelife.getUsername());
    }
}
```

#### 데이터 변경이 일어 나지 않는 테스트
- 여러번 적용해서 마지막에 데이터가 1차캐시의 객체 값과 같다면 데이터를 변경하지 않는다
```java
Account freelife = session.load(Account.class, account.getId());
freelife.setUsername("ironman");
freelife.setUsername("superman");
freelife.setUsername("freelife");
System.out.println("=====================");
System.out.println(freelife.getUsername());  // update 쿼리가 발생하지 않는다. 
                                            // Username이 1차캐시의 객체 값과 같기 때문이다. 
```

### Parent 와 Child 예시

### Cascade를 적용하지 않은 테스트

#### Parent인 Post 클래스 생성
```java
@Entity
public class Post {

    @Id @GeneratedValue
    private Long id;
    private String title;

    @OneToMany(mappedBy = "post")
    private Set<Comment> comments = new HashSet<>();
    public void addComment(Comment comment) {
        this.getComments().add(comment);
        comment.setPost(this);
    }
}
```

#### Child인 Comment 클래스 생성
```java
@Entity
public class Comment {

    @Id @GeneratedValue
    private Long id;
    private String comment;

    @ManyToOne
    private Post post;
}
```
#### JpaRunner에 post 데이터 저장 테스트
- Cascade를 적용하지 않으면 post만 저장되고 comment에 전파되지 않는다

```java
@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Post post = new Post();
        post.setTitle("Spring Data JPA 언제 보나...");

        Comment comment = new Comment();
        comment.setComment("빨리 보고 싶엉.");
        post.addComment(comment);

        Comment comment1 = new Comment();
        comment1.setComment("곧 보여드릴께요.");
        post.addComment(comment1);

        Session session = entityManager.unwrap(Session.class);
        session.save(post);
    }
}
```

#### Cascade를 적용한 테스트
    - Post에 cascade = CascadeType.PERSIST 옵션 적용
    - Post를 저장할 때 저장하는 Persistent를 comments에 전파
    - post 라는 인스턴스가 Transient에서 Persistent 상태로 넘어갈때
    - Child에 연관관계에 있어 참조하고 있던 객체들도 같이 Persistent 상태가 되면서 같이 저장이 됨

```java
@OneToMany(mappedBy = "post", cascade = CascadeType.PERSIST)
private Set<Comment> comments = new HashSet<>();
public void addComment(Comment comment) {
    this.getComments().add(comment);
    comment.setPost(this);
}
```
> Cascade Persistent와 Removed를 적용한 테스트
> Post에 cascade = {CascadeType.PERSIST, CascadeType.REMOVE}) 옵션 적용

```java
@OneToMany(mappedBy = "post", cascade = {CascadeType.PERSIST, CascadeType.REMOVE})
private Set<Comment> comments = new HashSet<>();
public void addComment(Comment comment) {
    this.getComments().add(comment);
    comment.setPost(this);
}
```
#### Post 삭제 테스트
    - post의 1에 해당되는 것을 가져와 삭제해달라고 하면
    - delete를 호출하는 순간 Removed 상태가 되고 Removed 상태가 전파되어 comment들도 같이 Removed 상태가 된다음
    - Transaction이 Commit이 일어날때 데이터가 모두 Remove 됨
```java
@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Session session = entityManager.unwrap(Session.class);
        Post post = session.get(Post.class, 1l);
        session.delete(post);
    }
}
```

#### 일반적인 Cascade 옵션 설정
- 보통은 아래와 같이 cascade = CascadeType.ALL 로 주어서 다 전파하면 됨
```java
@OneToMany(mappedBy = "post", cascade = CascadeType.ALL)
private Set<Comment> comments = new HashSet<>();
public void addComment(Comment comment) {
    this.getComments().add(comment);
    comment.setPost(this);
}
```

Fectch
-------

### Fetch
- 연관 관계의 엔티티의 정보를 지금(Eager) 나중에(Lazy) 가져올지 설정한다.
- 잘 조정해야 성능을 향상시킬 수 있음

- @OneToMany의 기본값은 Lazy:
    - 기본적으로 해당 Entity의 정보를 가져올때 Lazy가 적용된 @OneToMany 관계의 Entity의 정보를 가져오지는 않음
    - 얼마나 많이 있을 지도 모르고 사용하지도 않을 값들을 다 가져오면 객체에 불필요한 정보를 로딩할 수도 있으므로

- @ManyToOne의 기본값은 Eager: 
    - 해당 Entity의 정보를 가져올때 Eager로 설정된 @ManyToOne 관계의 Entity의 정보도 같이 가져옴

#### Eager 테스트
- EAGER가 적용된 Entity 정보를 미리 다 가져와서 불필요한 조회를 더이상 하지 않음

- Account 클래스의 studies에 EAGER 를 적용
```java
@OneToMany(mappedBy = "owner", cascade = CascadeType.ALL, fetch = FetchType.EAGER)
private Set<Study> studies = new HashSet<>();
```
- JpaRunner 에서 테스트
```java
Session session = entityManager.unwrap(Session.class);
Post post = session.get(Post.class, 4l);
System.out.println("========================");
System.out.println(post.getTitle());
```

#### Lazy 테스트
- Lazy가 적용된 Entity 정보를 미리 가져 오지 않음

- Account 클래스의 studies에 LAZY를 적용
```java
@OneToMany(mappedBy = "owner", cascade = CascadeType.ALL)
private Set<Study> studies = new HashSet<>();
```

- n+1 테스트
- 재현되지 않음
- fetch에 Lazy 옵션이 적용되어 있더라도 n에 해당되는 것을 한번에 다가져와서 처리하므로 재현되지 않음
- Hibernate의 기능 개선으로 성능상의 이슈가 크게 없을 것으로 보임
- 너무 많은 데이터를 객체에 로딩하는 문제는 있을 수 있음

- EAGER 테스트
```java
Session session = entityManager.unwrap(Session.class);
Post post = session.get(Post.class, 4l);
System.out.println("========================");
System.out.println(post.getTitle());

post.getComments().forEach(c -> {
    System.out.println("--------------");
    System.out.println(c.getComment());
});
```

#### Hibernate
- load: 가져오려 할때 없으면 예외를 던짐 Proxy로도 가져올 수 있음
- get: 무조건 DB에서 가져옴 해당하는게 없으면 예외를 던지지 않고 무조건 레퍼런스를 null로 만듬


Query
-----
> JPA, Hibernate를 사용할 때는 항상 무슨 Query를 발생시키는지 그게 의도한 것인지 확인해야된다.

### JPQL (HQL)
- Java Persistence Query Language / Hibernate Query Language
- 데이터베이스 테이블이 아닌, 엔티티 객체 모델 기반으로 쿼리 작성
- JPA 또는 하이버네이트가 해당 쿼리를 SQL로 변환해서 실행함
- https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#hql

#### JPQL 예시
> Post는 테이블 이름이 아니라 Entity 이름  
> JPA 2.0 부터는 Type을 지정할 수 있고 지정한 Type의 리스트로 출력이 됨  
> 이전에는 Object Type으로 나와서 다 변환해줘야됐었음  

#### Post에 title toString 추가
> toString()에 comment(자식 엔티티) 까지 출력하도록 하면 하이버네이트가 comment에 대한 쿼리까지 발생시킨다. (주의)
```java
@Override
public String toString() {
    return "Post{" +
            "title='" + title + '\'' +
            '}';
}
```

#### JpaRunner에서 테스트 로직 구현
```java
TypedQuery<Post> query = entityManager.​createQuery​("SELECT p FROM Post As p", Post.class);
List<Post> posts = query.getResultList();
posts.forEach(System.out::println); //메소드 레퍼런스 람다식을 더 줄여준다.
```

### Criteria
https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#criteria
#### 타입 세이프 쿼리
```java
CriteriaBuilder builder = entityManager.​getCriteriaBuilder​();
CriteriaQuery<Post> criteria = builder.createQuery(Post.class);
Root<Post> root = criteria.from(Post.class);
criteria.select(root);
List<Post> posts = entityManager.​createQuery​(criteria).getResultList();
```

## Native Query
https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#sql
> Typed 메서드가 아니더라도 지정한 Type으로 결과값을 리턴해줌  
#### SQL 쿼리 실행하기
```java
List<Post> posts = entityManager
                .createNativeQuery("SELECT * FROM Post", Post.class)
                .getResultList();
```


스프링 데이터 JPA 소개 및 원리
=============================

## JpaRepository<Entity, Id> 인터페이스
- 매직 인터페이스
- @Repository가 없어도 빈으로 등록해 줌

## @EnableJpaRepositories
- 매직의 시작은 여기서 부터

## 매직은 어떻게 이뤄지나?
> JpaRepository 로 구현한 Repository가 어떻게 자동으로 빈으로 등록됐는지는 아래의 클래스에서 확인  
- 시작은 @Import(​JpaRepositoriesRegistrar.class​)
- 핵심은 ​ImportBeanDefinitionRegistrar​ 인터페이스
  - SpringFrameWork의 인터페이스이며 구현체가 다양함
  - 빈을 프로그래밍을 통해서 등록할 수 있게 해줌
  - JpaRepository를 상속받은 모든 인터페이스들을 찾아서 빈으로 등록해줌

### ​ImportBeanDefinitionRegistrar​ 예제
> Freelife를 빈으로 등록하는 예제  

#### Freelife 클래스 구현
```java
public class Freelife {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

#### FreelifeRegistrar 클래스 구현
> 빈으로 등록하는 프로그래밍 과정  
```java
public class FreelifeRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        GenericBeanDefinition beanDefinition = new GenericBeanDefinition();
        beanDefinition.setBeanClass(Freelife.class);
        beanDefinition.getPropertyValues().add("name", "Superman");

        registry.registerBeanDefinition("freelife", beanDefinition);
    }
}
```

#### @Import 설정
> 최종적으로 아래와 같이 설정하면 프로그래밍에 의해서 자동으로 빈으로 등록됨  
```java
@SpringBootApplication
@Import(FreelifeRegistrar.class)
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```

#### 등록된 빈을 주입받아서 출력 테스트
```java
@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @Autowired
    Freelife freelife;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println("=====================");
        System.out.println(freelife.getName());
    }
}
```

### 예전 방식
> 코드도 작성하고 테스트도 해야되고 매우 번거로움  
```java
@Repository
@Transactional
public class PostRepository {

    @PersistenceContext
    EntityManager entityManager;

    public Post add(Post post) {
        entityManager.persist(post);
        return post;
    }

    public void delete(Post post) {
        entityManager.remove(post);
    }

    public List<Post> findAll() {
        return entityManager.createQuery("SELECT p FROM Post AS p", Post.class)
                .getResultList();
    }
}
```

#### 예전 정의 방식
> 예전에는 기본적인 코드들은 만들어서 정의해서 사용하는 프레임워크가 유행했었음
```java
@Repository
public class PostRepository extends GenericRepository<Post, Long> {

}
```

### 현재의 방식
> PostRepository 라는 interface를 만들고 JpaRepository라는 interface를 상속받음  
> JpaRepository 첫번째 타입은 Entity 타입이고 두번째 타입은 Entity에서 사용하는 PK의 Type  
> @EnableJpaRepositories 는 스프링부트가 자동 설정해줌  
> 아래와 같이 구현하면 @Repository를 지정할 필요없이 빈으로 등록됨  
#### PostRepository 인터페이스 구현
```java
public interface PostRepository extends JpaRepository<Post, Long> {
}
```

#### JpaRunner 테스트 구현
> EntityManager로 복잡하게 구현했던 Code를 SpringDataJPA를 통해 안정적으로 검증된 Code를 사용하여 간결하게 구현가능하다.  
> 생산성, 유지보수성, 코드의 간결함, 간결한 코드로 인해 테스트 작성이 불필요함  
```java
@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @Autowired
    PostRepository postRepository;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        postRepository.findAll().forEach(System.out::println);
    }
}
```

# 스프링 데이터 Common: Repository 
  
## 스프링 데이터 Common
- Repository: Marker 용
- CrudRepository: 기본적인 CRUD 오퍼레이션들을 제공
- PagingAndSortingRepository

## @NoRepositoryBean
> 스프링 데이터 JPA 또는 스프링 데이터 다른 저장소용 Repository가 적용된 빈의 실제 빈을 만들지 않도록  
> 방지하기 위하여 CrudRepository에 적용함  

## JPA 슬라이싱 테스트
### @DataJpaTest
> 스프링 부트에서 제공하는 데이터 엑세스 레이어만 테스트 하는 애노테이션  
> Repository 관련된 빈들만 등록됨  

### 테스트 실습
> h2는 기본적으로 @Transactional 이 붙어있으면 모든 테스트는 Rollback을 함  
> hibernate는 필요할때만 데이터베이스에 Query를 Sync하는데 Rollback한 Query라서 데이터를 Insert하지 않음  
> 결론적으로 ID 만 가져오고 Insert Query를 날리지 않고 끝냄  
#### Rollback 되는 테스트
```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class PostRepositoryTest {

    @Autowired
    PostRepository postRepository;

    @Test
    public void crudRepository() {
        // Given - 이런 조건 하에서
        Post post = new Post();
        post.setTitle("hello spring boot common");
        assertThat(post.getId()).isNull();

        // When - 이렇게 했을때
        Post newPost = postRepository.save(post);

        // Then - 이렇게 되길 바란다
        assertThat(newPost.getId()).isNotNull();
    }
}
```

#### Rollback이 되지 않도록 @Rollback(false) 로 지정
- newPost가 PersistenceContext에 들어가고 findAll은 Query를 만들어서 실행하니까 실제 Query가 날아감  
- 내가 PersistenceContext에 가지고 있는 객체가 데이터베이스의 전체 데이터라는 보장은 없으니까 Query를 보냄  
- page는 `page`라는 도메인으로 나옴 요청시에는 `pageable` 이라는 Parameter로 요청해야함  
```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class PostRepositoryTest {

    @Autowired
    PostRepository postRepository;

    @Test
    @Rollback(false)
    public void crudRepository() {
        // Given - 이런 조건 하에서
        Post post = new Post();
        post.setTitle("hello spring boot common");
        assertThat(post.getId()).isNull();

        // When - 이렇게 했을때
        Post newPost = postRepository.save(post);

        // Then - 이렇게 되길 바란다
        assertThat(newPost.getId()).isNotNull();

        // when
        List<Post> posts = postRepository.findAll();

        // Then
        assertThat(posts.size()).isEqualTo(1);
        assertThat(posts).contains(newPost); // newPost 인스턴스를 posts 컬렉션이 가지고 있어야 된다

        // When - 0페이지 부터 10개의 페이지를 달라고 요청
        Page<Post> page = postRepository.findAll(PageRequest.of(0, 10));
        assertThat(page.getTotalElements()).isEqualTo(1); // 전체 페이지 개수
        assertThat(page.getNumber()).isEqualTo(0); // 현재 페이지 넘버
        assertThat(page.getSize()).isEqualTo(10); // 요청했던 사이즈
        assertThat(page.getNumberOfElements()).isEqualTo(1); // 현재 페이지에 들어올 수있는 개수
    }
}
```

### Custom 메서드 추가
https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories
  
> 이러한 Repository Interface는 Custom한 메서드를 추가 할 수있음  
> 어떠한 특정한 키워드를 가지고 있는 Post 목록을 페이징을 해서 찾겠다 라는 메서드 추가  
> 메서드의 이름을 분석해서 SpringDataJPA가 Query를 만들어줌  
#### PostRepository에 Custom 메서드 추가
```java
import org.springframework.data.domain.Page ;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;

public interface PostRepository extends JpaRepository<Post, Long> {

    Page<Post> findByTitleContains(String title, Pageable pageable);
}
```

#### 테스트 코드 
> findAll 과 결과가 같은 것을 확인할 수 있음  
```java
page = postRepository.findByTitleContains("spring", PageRequest.of(0, 10));

assertThat(page.getTotalElements()).isEqualTo(1); // 전체 페이지 개수
assertThat(page.getNumber()).isEqualTo(0); // 현재 페이지 넘버
assertThat(page.getSize()).isEqualTo(10); // 요청했던 사이즈
assertThat(page.getNumberOfElements()).isEqualTo(1); // 현재 페이지에 들어올 수있는 개수
```

### 또 다른 Custom 메서드 추가
> 좀 더 다양한 Custom 메서드를 추가 할 수 있음  
```java
public interface PostRepository extends JpaRepository<Post, Long> {

    long countByTitleContains(String title);
}
```

#### 테스트 코드
```java
// when - spring을 가지고 있는 개수를 모두 센다
long spring = postRepository.countByTitleContains("spring");
// then
assertThat(spring).isEqualTo(1); // 개수는 1이다
```

스프링 데이터 Common: Repository 인터페이스 정의하기
==================================================

## Repository 인터페이스로 공개할 메소드를 직접 일일히 정의하고 싶다면
> JpaRepository 에서 들어오는 기능들이 싫고 직접 정의하고 싶다면 두가지 방법이 있음  

### 특정 리포지토리 당 @RepositoryDefinition
> (Spring Data JPA Reference중) - 만약 당신이 스프링 데이터 인터페이스를 상속하기를 원하지 않는다면 당신은 당신의 리파지토리에 @RepositoryDefinition를 붙일 수 있습니다.
> CrudRepository를 상속하는 것은 당신의 엔티티를 다루는 메소드들의 집합을 노출시킵니다. 
> 만약 당신이 노출되는 메소드들에 관해서 선택적으로 하고 싶다면, 단순히 CrudRepository에서 당신이 노출시키는 부분을 복사하여 당신의 도메인 리파지토리에 붙여보세요.
> 내가 사용할 기능을 내가 직접 메서드를 정의  
> 메서드에 해당하는 기능을 스프링 데이터 JPA가 구현할 수 있는 것이라면 구현해서 기본으로 제공해줌  
```java
@RepositoryDefinition(domainClass = Comment.class, idClass = Long.class)
public interface CommentRepository {
    Comment save(Comment comment);
    List<Comment> findAll();
}
```

### 공통 인터페이스 정의 @NoRepositoryBean
```java
@NoRepositoryBean
public interface MyRepository<T, ID extends Serializable> extends Repository<T, ID> {
    <E extends T> E save(E entity);
    List<T> findAll();
    long count();
}
```

#### 정의한 공통 인터페이스를 상속받아서 사용
```java
public interface CommentRepository extends MyRepository<Comment, Long>{
}
```

### 테스트 실습
```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class CommentRepositoryTest {

    @Autowired
    CommentRepository commentRepository;

    @Test
    public void crud() {
        Comment comment = new Comment();
        comment.setComment("Hello Comment");
        commentRepository.save(comment);

        List<Comment> all = commentRepository.findAll();
        assertThat(all.size()).isEqualTo(1);

        long count = commentRepository.count();
        assertThat(count).isEqualTo(1);
    }
}
```

스프링 데이터 Common: Null 처리하기
==================================

## 스프링 데이터 2.0 부터 자바 8의 Optional 지원
- 단일 값을 사용하는 경우 Optional로 처리하는 것을 추천  
- 콜렉션은 Null을 리턴하지 않고, 비어있는 콜렉션을 리턴  
- Optional 인터페이스가 제공하는 메서드를 사용해서 검사할 수 있음  
```java
Optional<Post> findById(Long id);
```

### Optional 메서드
- isPresent() : 값이 있는지 없는지 확인
- orElse : 없는 경우 다른 대체적인 인스턴스를 리턴하게 할 수도 있음
- orElseThrow : 없는 경우 예외를 던짐

### Optonal 단일 값 리턴 실습 코드
#### MyRepository에 Optional<E> findById 추가
```java
@NoRepositoryBean
public interface MyRepository<T, Id extends Serializable> extends Repository<T, Id> {
    <E extends T> E save(E entity);
    List<T> findAll();
    long count();

    <E extends T> Optional<E> findById(Id id);
}
```

#### CommentRepositoryTest 테스트 코드 작성
> Optional 타입의 인스턴스가 나오므로 Null로 체크 하는 것이 아님  
> Comment가 null이어서 안이 비어있다고 isEmpty로 Optional 을 검사  
```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class CommentRepositoryTest {

    @Autowired
    CommentRepository commentRepository;

    @Test
    public void crud() {
        Optional<Comment> byId = commentRepository.findById(100l);
        assertThat(byId).isEmpty();
        //Comment comment = byId.orElse(new Comment()); // Optional 객체가 비어있으면 Comment 만들어서 리턴
        //Comment comment = byId.orElseThrow(() -> new IllegalArgumentException());
        Comment comment = byId.orElseThrow(IllegalArgumentException::new); // lambda를 메서드 레퍼런스로 변환 
    }
}
```

#### Optional 을 사용하지 않고 처리 하면 아래와 같이 구현함
```java
Comment comment = commentRepository.findById(100l);
if (comment == null)
    throw new IllegalArgumentException();
```

### List 리턴 실습 코드
> 스프링 데이터 JPA가 지원하는 Repository의 컬렉션 타입들은 null이 되지 않고 비어있는 컬렉션 타입을 리턴해줌  
> 컬렉션을 null 체크하는 로직은 필요없음  
```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class CommentRepositoryTest {

    @Autowired
    CommentRepository commentRepository;

    @Test
    public void crud() {
        List<Comment> comments = commentRepository.findAll();
        assertThat(comments).isEmpty();
    }
}
```

## 스프링 프레임워크 5.0부터 지원하는 Null 애노테이션 지원.
- 런타임체크지원함
- JSR 305 애노테이션을 메타 애노테이션으로 가지고 있음 (IDE 및 빌드 툴 지원)

### Null 애노테이션
- @NonNullApi: package 레벨에 붙이는 애노테이션 
- @NonNull: Null이 아니다
- @Nullable: Null일 수도 있다 메서드에 적용하면 리턴값이고 파라메터에 적용하려면 파라메터 옆에 붙임
> package-info.java 파일을 생성 하고 적용  
> 해당 package 안에 모든 메서드 파라메터 리턴타입에 `@NonNull`이 적용됨  
> 적용하고 허용하는 부분만 `@Nullable`을 붙여주는 용도로 사용할 수도 있음  
> 단점은 package 레벨에 적용하는 애노테이션을 적용하면 툴 지원을 받을 수 없게 됨   
```java
@org.springframework.lang.NonNullApi
package me.freelife.springjpaprograming;
```

### Null 체크 실습
#### MyRepository 에 Null 애노테이션 적용
```java
@NoRepositoryBean
public interface MyRepository<T, Id extends Serializable> extends Repository<T, Id> {
    <E extends T> E save(@NonNull E entity);
    List<T> findAll();
    long count();

    @Nullable
    <E extends T> Optional<E> findById(Id id);
}
```

#### 테스트 로직
> Hibernate 가기 전에 이미 Parameter의 애노테이션을 보고 Null 체크를 함  
```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class CommentRepositoryTest {

    @Autowired
    CommentRepository commentRepository;

    @Test
    public void crud() {
      commentRepository.save(null);
    }
}
```

스프링 데이터 Common 4: 쿼리 만들기 실습
======================================

## 실습 예제
> Repository bean을 생성할 때 쿼리 메서드에 해당하는 쿼리도 생성을 하고  
> 애플리케이션의 빈에 등록이 되기 때문에 쿼리를 제대로 못만드는 경우 에러가 남  
#### findByCommentContains 메서드 추가
```java
public interface CommentRepository extends MyRepository<Comment, Long>{

    //keyword에 해당하는게 있는지 찾는다.
    List<Comment> findByCommentContains(String keyword);
}
```

#### 쿼리 테스트
> Spring의 S가 대문자라서 해당하는 Comment가 없다고 에러가 남  
```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class CommentRepositoryTest {

    @Autowired
    CommentRepository commentRepository;

    @Test
    public void crud() {
        Comment comment = new Comment();
        comment.setComment("spring data jpa");
        commentRepository.save(comment);

        List<Comment> comments = commentRepository.findByCommentContains("Spring");
        assertThat(comments.size()).isEqualTo(1);
    }
}
```

### IgnoreCase 추가
> upper를 적용하여 값을 모두 대문자로 변경해서 조회하므로 에러가 나지 않음  
```java
// IgnoreCase는 upper를 적용하여 값을 모두 대문자로 변경 조회한다.
List<Comment> findByCommentContainsIgnoreCase(String keyword);
```

### likeCount가 특정 숫자보다 높을 때 조건 추가
```java
List<Comment> findByCommentContainsIgnoreCaseAndLikeCountGreaterThan(String keyword, int likeCount);
```

#### likeCount 값을 초기화
```java
private Integer likeCount = 0;
```

#### likeCount 적용 쿼리 테스트
```java
Comment comment = new Comment();
comment.setLikeCount(1);
comment.setComment("spring data jpa");
commentRepository.save(comment);

List<Comment> comments = commentRepository.findByCommentContainsIgnoreCaseAndLikeCountGreaterThan("Spring", 10);
assertThat(comments.size()).isEqualTo(0);
```

### Comment를 likeCount 높은 순으로 정렬 
```java
List<Comment> findByCommentContainsIgnoreCaseOrderByLikeCountDesc(String keyword);
```

```java
@Test
public void crud() {
    this.createComment(100, "spring data jpa");
    this.createComment(55, "HIBERNATE SPRING");

    List<Comment> comments = commentRepository.findByCommentContainsIgnoreCaseOrderByLikeCountDesc("Spring");
    assertThat(comments.size()).isEqualTo(2); // 2개의 결과가 있고
    assertThat(comments).first().hasFieldOrPropertyWithValue("likeCount", 100); // 첫번째 값의 likeCount = 100
}

private void createComment(int likeCount, String comment) {
    Comment newComment = new Comment();
    newComment.setLikeCount(likeCount);
    newComment.setComment(comment);
    commentRepository.save(newComment);
}

//나의 코드 

    Comment comment = new Comment();
    comment.setComment("심현섭10따리");
    comment.setLikeCount(10);
    
    Comment comment2 = new Comment();
    comment2.setComment("심현섭20따리");
    comment2.setLikeCount(20);
    
    Comment comment3 = new Comment();
    comment3.setComment("심현섭30따리");
    comment3.setLikeCount(30);
    
    List<Comment> commentList = new ArrayList<Comment>();
    commentList.add(comment);
    commentList.add(comment2);
    commentList.add(comment3);
    
//		commentList.forEach( x -> {
//			commentRepository.save(x);
//		});
    commentList.forEach(commentRepository :: save);
    
    List<Comment> result = commentRepository.findByCommentContainsIgnoreCaseOrderByLikeCountDesc("심현섭");
    assertThat(result.size()).isEqualTo(3);
    assertThat(result).first().hasFieldOrPropertyWithValue("likeCount", 30);


```

#### ASC 도 테스트
```java
List<Comment> findByCommentContainsIgnoreCaseOrderByLikeCountAsc(String keyword);
```

```java
List<Comment> comments = commentRepository.findByCommentContainsIgnoreCaseOrderByLikeCountAsc("Spring");
assertThat(comments.size()).isEqualTo(2); // 2개의 결과가 있고
assertThat(comments).first().hasFieldOrPropertyWithValue("likeCount", 55); // 첫번째 값의 likeCount = 55
```

### Pageable로 동적으로 정렬
```java
//Pageable에 sort기능도 포함되어있다.
Page<Comment> findByCommentContainsIgnoreCase(String keyword, Pageable pageable);
```

```java
Comment comment = new Comment();
comment.setComment("심현섭10따리");
comment.setLikeCount(10);

Comment comment2 = new Comment();
comment2.setComment("심현섭20따리");
comment2.setLikeCount(20);

Comment comment3 = new Comment();
comment3.setComment("심현섭30따리");
comment3.setLikeCount(30);

List<Comment> commentList = new ArrayList<Comment>();
commentList.add(comment);
commentList.add(comment2);
commentList.add(comment3);

//		commentList.forEach( x -> {
//			commentRepository.save(x);
//		});
commentList.forEach(commentRepository :: save);

PageRequest pageRequest = PageRequest.of(0,10);

Page<Comment> result = commentRepository.findByCommentContainsIgnoreCase("심현섭", pageRequest); 
assertThat(result.getNumberOfElements()).isEqualTo(3); //페이지의 현재 갯수 , getTotalElements는 전제의 갯수  
assertThat(result).first().hasFieldOrPropertyWithValue("likeCount", 10); //첫번째로 발견되는 likecount의 값 10이 발견되는것으로 보아 Asc인듯.
```

#### DESC로 정렬
```java
PageRequest pageRequest = PageRequest.of(0,10,Sort.by(Sort.Direction.DESC, "LikeCount"));

Page<Comment> result = commentRepository.findByCommentContainsIgnoreCase("심현섭", pageRequest); 
assertThat(result.getNumberOfElements()).isEqualTo(3); //페이지의 현재 갯수 , getTotalElements는 전제의 갯수  
assertThat(result).first().hasFieldOrPropertyWithValue("likeCount", 30); // PageRequest.of() 를통해 Sort 설정을 desc로 바꾸었다.
```

### Stream으로 받아오기
```java
Stream<Comment> findByCommentContainsIgnoreCase(String keyword, Pageable pageable);
```

#### Stream 으로 받아 오는 로직
> Stream은 Stream을 받고 난다음 닫아줘아함 그래서 try With resource를 사용하였다. 
```java
PageRequest pageRequest = PageRequest.of(0,10,Sort.by(Sort.Direction.DESC, "LikeCount"));

try (Stream<Comment> result = commentRepository.findByCommentContainsIgnoreCase("심현섭", pageRequest)){
    Comment firstComment = result.findFirst()
                                    .get(); //첫번째 값 조회하기 ! 
    assertThat(firstComment.getLikeCount()).isEqualTo(30);
}
```

## 기본 예제
```java
List<Person> findByEmailAddressAndLastname(EmailAddress emailAddress, String lastname);
// distinct
List<Person> findDistinctPeopleByLastnameOrFirstname(String lastname, String firstname);
List<Person> findPeopleDistinctByLastnameOrFirstname(String lastname, String firstname);
// ignoring case
List<Person> findByLastnameIgnoreCase(String lastname);
// ignoring case
List<Person> findByLastnameAndFirstnameAllIgnoreCase(String lastname, String firstname);
```

## 정렬
```java
List<Person> findByLastnameOrderByFirstnameAsc(String lastname);
List<Person> findByLastnameOrderByFirstnameDesc(String lastname);
```

## 페이징
```java
Page<User> findByLastname(String lastname, Pageable pageable);
Slice<User> findByLastname(String lastname, Pageable pageable);
List<User> findByLastname(String lastname, Sort sort);
List<User> findByLastname(String lastname, Pageable pageable);
```

## 스트리밍
> try-with-resource 사용할 것. (Stream을 다 쓴다음에 close() 해야 함)  
```java
Stream<User> readAllByFirstnameNotNull();
```

2-7. 스프링 데이터 Common: 커스텀 리포지토리
==========================================

> 쿼리 메소드(쿼리 생성과 쿼리 찾아쓰기)로 해결이 되지 않는 경우 직접 코딩으로 구현 가능  
- 스프링 데이터 리포지토리 인터페이스에 기능 추가
- 스프링 데이터 리포지토리 기본 기능 덮어쓰기 가능

## 구현 방법
1. 커스텀 리포지토리 인터페이스 정의 
2. 인터페이스 구현 클래스 만들기 (기본 접미어는 Impl)
3. 엔티티 리포지토리에 커스텀 리포지토리 인터페이스 추가
 
## 기능 추가하기 예제 실습
#### Post 클래스 생성
```java
@Entity
public class Post {
    @Id @GeneratedValue
    private Long id;

    private String title;

    @Lob  //255자 넘을 때 사용. 
    private String content;

    @Temporal(TemporalType.TIMESTAMP)
    private Date created;
}
```

#### PostCustomRepository 인터페이스 생성
> Custom Repository 인터페이스  
```java
public interface PostCustomRepository {
    List<Post> findMyPost();
}
```

#### PostCustomRepositoryImpl 구현체 생성
> PostCustomRepository 구현체 추가  
```java
@Repository
@Transactional
public class PostCustomRepositoryImpl implements PostCustomRepository{

    @Autowired
    EntityManager entityManager;

    @Override
    public List<Post> findMyPost() {
        System.out.println("custom findMyPost");
        return entityManager.createQuery("SELECT p FROM Post AS p ", Post.class).getResultList());
    }
}
```

#### PostRepository 인터페이스 생성
> PostRepository 인터페이스에 JpaRepository와 PostCustomRepository 상속  
```java
public interface PostRepository extends JpaRepository<Post, Long>, PostCustomRepository {
}
```
 
## 기본 기능 덮어쓰기 delete 기능 덮어쓰기 예제
> 스프링 데이터 JPA는 항상 내가 Custom하게 구현한 구현체를 우선순위를 높게 줌  

#### PostCustomRepository 인터페이스에 delete 추가
```java
public interface PostCustomRepository<T> {
    void delete(T entity);
}
```

#### delete 메서드 기능 구현
```java
@Repository
@Transactional
public class PostCustomRepositoryImpl implements PostCustomRepository<Post>{

    @Autowired
    EntityManager entityManager;

    @Override
    public void delete(Post entity) {
        System.out.println("custom delete");
        entityManager.remove(entity);
    }
}
```

#### PostRepository 타입 추가
```java
public interface PostRepository extends JpaRepository<Post, Long>, PostCustomRepository<Post> {
}
```

#### 테스트 로직 구현
> 아래와 같이 구현하면 Hibernate가 판단하여 쿼리를 수행하지 않음  
```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class PostRepositoryTest {

    @Autowired
    PostRepository postRepository;

    @Test
    public void crud() {
        Post post = new Post();
        post.setTitle("hibernate");
        postRepository.save(post);
        postRepository.delete(post);
    }
}
```

#### Insert와 Select가 일어나도록 구현
> 아래와 같이 구현하면 select가 필요하다고 판단하여 플러싱이 일어나서 데이터가 Insert 되고  
> select하여 데이터를 가져옴  
> delete는 객체상태에서 entityManager가 removed 상태로 만들었지만 실제로 데이터베이스에 sync는 하지 않는다  
> 하지만 기본적으로 **@Transactional**이 붙어있는 스프링의 모든 테스트는 그 트랜잭션을 모두 ROLLBACK 트랜잭션으로 간주함  
> Hibernate의 경우 ROLLBACK 트랜잭션의 경우 필요없는 Query는 아예 날리지 않음  
> 하지만 강제로 flush() 하면 delete Query를 날림  
```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class PostRepositoryTest {

    @Autowired
    PostRepository postRepository;

    @Test
    public void crud() {
        Post post = new Post();
        post.setTitle("hibernate");
        postRepository.save(post);
        postRepository.findMyPost();
        postRepository.delete(post);
        postRepository.flush();
    }
}
```
 
## 접미어 설정하기 
> 기본적으로 repositoryImplementationPostfix의 접미어는 `impl` 인데 변경하고 싶다면 아래와 같이 설정하면 된다  
```java
@SpringBootApplication
@EnableJpaRepositories(repositoryImplementationPostfix = "Default")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```


## JPA DELETE에 대하여
- detached 상태: 한번 언젠가 persistence 상태가 되었던 인스턴스가 persistence 상태에서 빠진 상태
  - 더이상 PersistentContext에서 관리를 받지않는 대상이된 객체
  - Transaction 밖으로 나옴
  - Transaction 이 끝나고 계속해서 남아있는 레퍼런스
  - 세션을 클리어 함
  - 예전에 Persistence 상태였다는 말은 맵핑되는 테이블에 데이터가 있다는 말임  
> detached 상태의 인스턴스를 예전에 맵핑이 됐던 인스턴스이므로 **merge**하면서 다시 persistentContext로 로딩을 해오면서 Sync를 맞춤  
> 그리고 entityManager가 해당 객체를 삭제하여 removed 상태로 만듬  
> cascade 처럼 entityManager로 로딩해서 처리하는 이유가 있음 단순히 성능적인 부분만 고려하면 안됨  

스프링 데이터 Common: 기본 리포지토리 커스터마이징
===============================================

> 모든 리포지토리에 공통적으로 추가하고 싶은 기능이 있거나 덮어쓰고 싶은 기본 기능이 있다면   
 
1. JpaRepository를 상속 받는 인터페이스 정의
   - @NoRepositoryBean
2. 기본 구현체를 상속 받는 커스텀 구현체 만들기
3. @EnableJpaRepositories에 설정
   - repositoryBaseClass

## 실습
> 어떤 Entity가 PersistentContext에 들어있는 지 확인하는 기능 구현  
#### JpaRepository를 상속받는 MyRepository 구현
> 중간에 들어가는 Repository는 반드시 @NoRepositoryBean을 등록해줘야함  
```java
@NoRepositoryBean
public interface MyRepository<T, ID extends Serializable> extends JpaRepository<T, ID> {
    boolean contains(T entity);
}
```

#### SimpleJpaRepository 구현
- 스프링 데이터 JPA가 제공해주는 가장많은 기능을 가지고 있는 가장 밑단에 있는 클래스
- JpaRepository를 상속받으면 가져오게 되는 구현체
- MyRepository도 상속 받음
- 부모에 super를 호출해야되므로 두개의 인자를 받는 생성자도 만들어줘야함
- 빈으로 주입하지 않고 사용자에게 전달을 받는 entityManager를 사용 
```java
public class SimpleMyRepository<T, ID extends Serializable> extends SimpleJpaRepository<T, ID> implements MyRepository<T, ID> {
 
    private EntityManager entityManager;
 
    public SimpleMyRepository(JpaEntityInformation<T, ?> entityInformation, EntityManager entityManager) {
        super(entityInformation, entityManager);
        this.entityManager = entityManager;
    }
 
    //entity를 전달 받아 PersistentContext 안에 들어 있는지 검증하는 로직 구현
    @Override
    public boolean contains(T entity) {
        return entityManager.contains(entity);
    }
}
```

#### repositoryBaseClass로 설정
```java
@EnableJpaRepositories(repositoryBaseClass = SimpleMyRepository.class)
```

#### PostRepository가 MyRepository를 상속받도록 구현
```java
public interface PostRepository extends MyRepository<Post, Long> {
}
```

#### 테스트 로직 구현
```java
@RunWith(SpringRunner.class)
@DataJpaTest
public class PostRepositoryTest {

    @Autowired
    PostRepository postRepository;

    @Test
    public void crud() {
        Post post = new Post();
        post.setTitle("hibernate");

        // save 하기 전에 postRepository.contains안에 post가 들어있는지 검증
        // post가 들어있지 않은 Transient 상태
        assertThat(postRepository.contains(post)).isFalse();

        postRepository.save(post);

        // post가 들어있는 Persist 상태
        assertThat(postRepository.contains(post)).isTrue();

        postRepository.delete(post);
        postRepository.flush();
    }
}
```

## JpaRepository에서 제공하는 다른 기능을 커스터마이징하고 싶다면?
> 구현체인 SimpleMyRepository에서 JpaRepository의 특정기능을 Overriding 해서 재구현하면 됨  
```java
public class SimpleMyRepository<T, ID extends Serializable> extends SimpleJpaRepository<T, ID> implements MyRepository<T, ID> {

    private EntityManager entityManager;

    public SimpleMyRepository(JpaEntityInformation<T, ?> entityInformation, EntityManager entityManager) {
        super(entityInformation, entityManager);
        this.entityManager = entityManager;
    }

    @Override
    public List<T> findAll() {
        return super.findAll();
    }
}
```



스프링 데이터 JPA: JPA Repository
=================================

> @EnableJpaRepositories 애노테이션을 사용해야 JpaRepository 인터페이스를 상속받은  
> Repository 인터페이스 타입의 Proxy 빈들을 등록 해준다  
## @EnableJpaRepositories
> 스프링 부트 사용할 때는 사용하지 않아도 자동 설정 됨  
> 스프링 부트 사용하지 않을 때는 @Configuration과 같이 사용  

### 설정 옵션
- EntityManager
- transactionManager
- basePackages: package scan을 시작할 base package  
  기본적으로는 @EnableJpaRepositories 애노테이션을 사용한 위치부터 찾기 시작함  
  best Practice는 항상 기본 Package에 위치  
 
## @Repository 애노테이션
> 안붙여도 된다 이미 붙어 있다 또 붙인다고 별일이 생기는건 아니지만 중복일 뿐  
 
## 스프링 @Repository
> SQLExcpetion 또는 JPA 관련 예외를 스프링의 DataAccessException 계층구조 하위클래스중 하나로 변환 해준다  
> 예외만 보더라도 어떤일이 발생했는지 이해하기 쉽도록 변환해줌  
> 기본적인 스프링 프레임워크의 기능  

### DataAccessException의 목적
> SQLExcpetion이 모든 데이터 엑세스 관련된 에러사항을 SQLException 하나로 던져주고  
> CODE 값을 확인해서 무슨 에러인지 찾아봐야 되는 불편함이 있었음  
> 그래서 DataAccessException 계층구조를 만들고 SQLExcpetion에 들어있는 CODE 값에 따라서  
> 굉장히 구체적인 하위 클래스들 중하나로 Mapping을 해서 클래스 이름만 봐도 알 수 있음  
> Hibernate, JPA 같은 경우는 이미 충분히 예외가 잘 만들어져 있음  
> JpaSystemException은 DataAccessException으로 변환 된것 임  

