## 개요
- 학습 목표
    - 해당 기술들이 왜 필요한지
    - 각 기술의 장단점이 무엇인지
- 다양한 데이터 접근 기술 방식
    - SQLMapper : 개발자가 SQL만 작성하면 그 결과를 객체로 편리하게 매핑
        - JdbcTemplate
        - MyBatis
    - ORM 관련 기술 : 저장하고 싶은 데이터를 자바 컬렉션에 저장하고 조회하듯이 사용 가능하게 함, SQL을 만들어줌
        - JPA, Hibernate
        - 스프링 데이터 JPA
        - Querydsl
---
## Jdbc Template
- 반복작업 해결
  - 커넥션 획득
  - `statement`를 준비하고 실행
  - 결과를 반복하도록 루프를 실행
  - 커넥션/ `statement`/ `resultset` 종료
  - 트랜잭션을 다루기 위한 커넥션 동기화
  - 예외 발생 시 스프링 예외 변환기 실행
- 단점 
  - 동적 sql을 해결하기 어렵다.
  
### Build Code
```groovy
implementation 'org.springframework.boot:spring-boot-starter-jdbc'
```

### 기능
- 순서 기반 파라미터 바인딩 (defalut)
- 이름 지정 파라미터 `NamedParameterJdbcTemplate`
  - 이때까진 sql문의 ? 에 바인딩 할 때, 순서에 맞게 지정을 해주어야 했다.
  - 하지만 이런 제약은 버그를 유발할 수 있기 때문에 순서에 상관 없이 가능하게 해당 기능을 사용한다. 
  - 바인딩에 사용되는 파라미터 종류
    1. `Map`
    2. `SqlParameterSource`
       - `BeanPropertySqlParameterSource`
       - `MapSqlParameterSource`
  - `BeanPropertyRowMapper` : 언더스코어 표기법을 카멜로 자동 변환해준다.
- `SimpleJdbcInsert` : Insert SQL 편리하게 이용하게 해줌 
- `SimpleJdbcCall` : 스토어드 프로시저를 편리하게 호출 가능

### 동적 쿼리 - MyBatis 
- sql을 직접 사용할 때 동적 쿼리를 쉽게 작성할 수 있다. 

---
## 데이터베이스에 연동하는 테스트 
- `@SpringBootTest`는 `@SpringBootApplacation`을 찾아서 설정으로 사용한다.

### 테스트 원칙
- 테스트는 다른 테스트와 격리해야 한다.
- 테스트는 반복해서 실행할 수 있어야 한다.

### 데이터베이스 분리
- /test 폴더 안의 properties 파일에 db 연결 정보를 바꿔준다. 

### 데이터 롤백 
- 트랜잭션 시작 -> 테스트 실행 -> 트랜잭션 롤백 
- `@BeforeEach` , `@AfterEach` 활용
- `@Transational`을 테스트에 사용하면, 트랜잭션을 시작시킨 후 테스트 로직 수행 후, 롤백되도록 수행된다.

### 임베디드 테스트 데이터베이스
- 임베디드 DB용 트랜잭션 매니저 새로 생성하는 방법  
  - 테스트용 DriverManagerDataSource 생성
  ```java
    @Bean
    @Profile("test")
    public DataSource dataSource() {
        log.info("메모리 데이터베이스 초기화");
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("org.h2.Driver");
        dataSource.setUrl("jdbc:h2:mem:db;DB_CLOSE_DELAY=-1");
        dataSource.setUsername("sa");
        dataSource.setPassword("");
        return dataSource;
    }
    ```
  - schema.sql : table을 생성하기 위한 파일
- 스프링 부트와 임베디드 모드 
  - 스프링 부트는 데이터베이스에 대한 별다른 설정이 없으면 임베디드 데이터베이스를 사용한다. 


---
## etc
### *Dto
- 데이터를 옮기기 위한 객체 
### `@EventListener(ApplicationReadyEvent.class)`
- 실행 시점 : 스프링 컨테이너가 완전히 초기화를 끝내고 실행 준비가 되었을 때 발생
- 강의에서는 휘발성 메모리에 기본 데이터를 실행 시 마다 저장되도록 사용 
- `@PostConstruct`와 비슷한 기능을 하지만 AOP 관련 코드가 다 처리되지 않은 상태에서 코드가 실행되는 것을 방지할 수 있다.
### 프로필 기능
- 프로필은 로컬, 운영 환경, 테스트 실행 등 다양한 환경에 따라서 다른 설정을 할 때 사용하는 정보이가. 
- `spring.profiles.active=local`
- `@Bean` `@Profile("local")`
- 스프링은 로딩 시점에 `application.properties`의 `spring.profiles.active=local` 속성을 읽어서 프로필로 사용한다.
  ```java
    @Import(MemoryConfig.class)
    @SpringBootApplication
    public class ItemServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(ItemServiceApplication.class, args);
    }

    @Bean
    @Profile("local")
    public TestDataInit testDataInit(ItemRepository itemRepository) {
        return new TestDataInit(itemRepository);
        }
    }
    ```
### 구현체가 아닌 인터페이스로 테스트를 작성하는 기법

