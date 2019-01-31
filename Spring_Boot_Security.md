Spring Boot Security 정리 
=========================


## 스프링 부트에서는 웹 접근시 로그인같은 인증과정을 쉽게 구현할 수 있도록 시큐리티 모듈을 제공한다.

스프링 부트 시큐리티 연동/커스터마이징 하기
----------------------------------------

- 의존성 추가 

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

#### index.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>This is Security Test</h1>
    <a href="/my">my</a>
    <a href="/hello">hello</a>
</body>
</html>

#### my.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>my</h1>
</body>
</html>

#### hello.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1>Hello</h1>
</body>
</html>
```

```java
@Controller
public class SimpleController {

    @GetMapping("/hello")
    public String hello(){
        return "hello";
    }

    @GetMapping("/my")
    public String my(){
        return "my";
    }
}

@Component
public class Security extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/", "/hello").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .httpBasic();
    }
}
```
- WebSecurityConfigurerAdapter를 상속받아 configure 메서드를 오버라이딩함으로써 시큐리티 설정을 할 수 있다. 
- 오버라이딩을 하지 않았다면 자동설정에 의해 어떤 url을 입력하더라도 시큐리티 formLogin화면이 호출되었을 것이다.
- index.html, hello.html 파일에 접근할 때( / , /hello )를 제외하고 모든 사용자 요청은 인증을 받도록 설정.
- httpBasic 그리고 formLogin 둘 동시에 인증을 받도록 설정하였습니다.
- http://localhost:8080/my 로 접근할 시 formLogin화면이 호출된다. 다른 경로로 접근시 호출되지않는다.
- 이 때 로그인에 필요한 정보는 기본적으로 username: user, password는 스프링부트가 실행될 때 콘솔창에 출력된다.

DB를 통한 유저 정보 생성 및 인증 
-------------------------------

- postgreSQL과 jpa를 사용하기위해 의존성 추가 
```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
</dependency>
```

#### Entity 
```java
@Entity
public class Account {

    @Id
    @GeneratedValue
    private Long id;
    private String userName;
    private String password;

    //getter(), setter() , toString() 생성 
}
```

#### Repostitory
```java
public interface AccountRepository extends JpaRepository<Account, Long> {
    Optional<Account> findByUserName(String username); 
    //userName을 기준으로 데이터를 가져올 수 있다. 
}
```

#### AccountAddRunner
```java
@Component
public class AccountAddRunner implements ApplicationRunner {
    
    //사용자 정보를 임의로 넣어주는 코드 작성 

    @Autowired
    AccountService accountService;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        Account saelobi = accountService.createAccount("saelobi", "1234");
    }
}
```

#### Service
```java
@Service
public class AccountService implements UserDetailsService {

    @Autowired
    private AccountRepository accountRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    public Account createAccount(String username, String password) {
        Account account = new Account();
        account.setUsername(username);
        account.setPassword(passwordEncoder.encode(password));
        return accountRepository.save(account);
    }

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Optional<Account> byUserName = accountRepository.findByUserName(username);
        Account account = byUserName.orElseThrow(() -> new UsernameNotFoundException(username));
        return new User(account.getUsername(), account.getPassword(), authorities());
    }

    private Collection<? extends GrantedAuthority> authorities() {
        return Arrays.asList(new SimpleGrantedAuthority("ROLE_USER"));
    }
}
```
- createAccount() : 임의로 데이터를 넣기위한 메소드
- UserDetailsService의 loadUserByUsername은 사용자 인증 처리를 할 시,사용자가 보내온 인증 정보와 DB에 적재된 사용자 로그인 데이터의 일치 여부를 확인하는 중요한 역할을 하는 메서드이다.

```java
@Component
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .antMatchers("/", "/hello").permitAll()
                .anyRequest().authenticated()
                .and()
                .formLogin()
                .and()
                .httpBasic();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }
}
```
- 스프링 부트 시큐리티 설정 클래스에 PasswordEncoder 에 대한 반환값을 생성하는 메서드를 작성
- PasswordEncoder를 반환하는 메서드를 구현하지 않으면 스프링 부트에서는 PassEncoder에 대한 정보를 찾을 수 없다면서 예외를 발생시킨다.
- 따라서 스프링 부트에서 권장하는 사항을 그대로 소스에 반영하는 것이 좋다.
- password는 우리가 임의대로 입력한 "1234"가 PasswordEncoder.encode()에 의해 {bcrypt}$2a$10$Rdo3hxnVrcoJbW4i7NA 이런식으로 저장되어있다.
