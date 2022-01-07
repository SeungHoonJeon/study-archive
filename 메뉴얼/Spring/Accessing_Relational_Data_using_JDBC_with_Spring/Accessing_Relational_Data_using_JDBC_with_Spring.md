## 스프링 JDBC 관계형 데이터 조작 방법

본 내용은 Spring 공식 문서인 [Accessing Relational Data using JDBC with Spring](https://spring.io/guides/gs/relational-data-access/)를 정리한 내용입니다.

## 다룰 내용

스프링 JDBC의 **JdbcTemplate**를 사용하여 관계형 데이터를 조작하는 방법에 대해서 배웁니다.

## 실습 환경

- 텍스트 에디터 또는 통합 개발 환경(IDE, Integrated Development Environment)
- JDK 1.8 버전 또는 그 이상
- 그레이들 4 또는 메이븐 3.2

## 스프링 이니셜라이즈로 시작하기

원격 레포지토리에서 프로젝트를 클론해도 되지만, 스프링 이니셜라이즈로 프로젝트틀 생성하여

한 단계씩 내용들을 살펴보겠습니다.

[프로젝트 생성 사이트](https://start.spring.io/)로 이동하여, 프로젝트를 설정합니다.

### 메이븐 프로젝트를 생성합니다.

![메이븐 프로젝트 생성](<./Accessing_Relational_Data_using_JDBC_with_Spring(1).png>)

### 의존성 라이브러리를 설정합니다.

![의존성 라이브러리 설정](<./Accessing_Relational_Data_using_JDBC_with_Spring(2).png>)

- **JDBC API** : Java로 관계형 데이터베이스에 연결하거나 쿼리문을 실행시키기 위한 표준 인터페이스 API 입니다.
- **H2 Database** : Java로 만들어진 인 메모리 관계형 데이터베이스 엔진입니다.

## 데이터를 저장하는 클래스 작성하기

애플리케이션에는 다양한 데이터들이 있습니다. 이 데이터들을 데이터베이스에 저장하기 위해서는

클래스로 변환을 해야 합니다. 즉, 데이터를 담을 그릇을 정의해야합니다.

다음 **Customer** 클래스를 작성합니다.

```
public class Customer {

	private long id;
	private String firstName, lastName;

	public Customer(long id, String firstName, String lastName) {
		this.id = id;
		this.firstName = firstName;
		this.lastName = lastName;
	}

	@Override
	public String toString() {
		return "Customer [id=" + id + ", firstName=" + firstName + ", lastName=" + lastName + "]";
	}
}
```

## 데이터를 저장하고 조회하기

스프링은 관계형 데이터베이스와 JDBC API를 쉽게 조작할 수 있도록 **JdbcTemplate** 클래스를 제공합니다.

## 애플리케이션 메인 메소드

```
@SpringBootApplication
public class DemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}
```

프로젝트를 생성하면, 애플리케이션을 실행시키기 위한 main 메소드가 존재합니다.

main 메소드는 스프링 부트의 run 메소드를 호출하여 애플리케이션을 실행합니다.

## @SpringBootApplication

애플리케이션을 실행시키는 DemoApplication 클래스 선언 위에 **@SpringBootApplication**이 존재합니다.

이 어노테이션은 **@Configuration**, **@EnableAutoConfiguration**, **@ComponentScan**등을 포함합니다.

- **@Configuration** : 해당 클래스가 스프링의 설정 클래스임을 나타내는 어노테이션입니다.
- **@EnableAutoConfiguration** : 스프링 부트의 자동 설정 기능을 활성화 시키는 어노테이션입니다.
- **@ComponentScan** : DemoApplication 클래스가 속해 있는 패키지를 기준으로 다른 컴포넌트(객체)를 인식하는 어노테이션입니다.

## JdbcTemplate 의존성 주입

앞서 프로젝트의 의존성 라이브러리 설정에서 **Spring JDBC 라이브러리**를 추가했습니다.

따라서 스프링 부트 프로젝트에서 자동으로 **JdbcTemplate** 객체가 생성됩니다.

**@Autowired 어노테이션**에 의해서 jdbcTemplate 참조 변수에 JdbcTemplate 객체가 할당됩니다.

```
@SpringBootApplication
public class DemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

  @Autowired
  JdbcTemplate jdbcTemplate; // JdbcTemplate 객체 로딩
  // ...
}
```

## JdbcTemplate: excute 메소드

DDL(Data Definition Language) 쿼리문을 실행시키기 위해서, execute 메소드를 호출합니다.

이때 실행문을 확인하기 위해서 DemoApplication 클래스가 **CommandLineRunner 인터페이스**를 구현합니다.

```
	@Override
	public void run(String... args) throws Exception {
		jdbcTemplate.execute("DROP TABLE customers IF EXISTS");
	  jdbcTemplate.execute("CREATE TABLE customers(id SERIAL, first_name VARCHAR(255), last_name VARCHAR(255))");
	}
}
```

- **첫 번째 SQL 문**은 customers 테이블이 존재하면 기존 테이블을 삭제합니다.
- **두 번째 SQL 문**은 id, first_name, last_name 열(Column)을 갖는 새로운 customers 테이블을 생성합니다.

## jdbcTemplate: batchUpdate, query 메소드

```
@Override
	public void run(String... args) throws Exception {
      // 생략...

      // 이름(first_name)과 성(last_name)의 리스트를 원소로 가지는 splitUpNames 리스트를 생성합니다.
	    List<Object[]> splitUpNames=Arrays.asList("John Woo", "Jeff Dean", "Josh Bloch", "Josh Long")
          .stream() // 스트림 객체 생성
	    		.map(name -> name.split(" "))
	    		.collect(Collectors.toList());

      // 로그 메시지를 콘솔 창에 출력합니다.
	    splitUpNames.forEach(name -> log.info(String.format("Inserting customer record for %s %s", name[0], name[1])));

      // INSERT 쿼리문을 실행합니다.
	    jdbcTemplate.batchUpdate("INSERT INTO customers(first_name, last_name) VALUES (?, ?)", splitUpNames);

      // SELECT 쿼리문을 실행합니다.
	    log.info("Querying for customer records where first_name = 'Josh':");
	    jdbcTemplate.query(
	    		"SELECT id, first_name, last_name FROM customers WHERE first_name = ?",
	    		(rs, rowNum) -> {
	    			Customer customer = new Customer(
	    					rs.getLong("id"),
	    					rs.getString("first_name"),
	    					rs.getString("last_name")
	    			);
	    			return customer;
	    		},
	    		"Josh").forEach(customer -> log.info(customer.toString()));
	}
```

- **batchUpdate** : INSERT 쿼리문을 실행하는 메소드입니다. JdbcTemplate의 **insert 메소드**도 존재하지만, 여러 개의 insert 쿼리문을 실행하는 경우 batchUpdate 메소드 사용을 권장합니다. 첫 번째 파라미터는 SQL 쿼리문을 문자열로 전달합니다. 마지막 파라미터는 object 인스턴스의 배열을 전달하여, SQL 쿼리문의 인덱스 파라미터(?)를 배열의 요소 값으로 대체합니다. 인덱스 파라미터를 사용하면, SQL 인젝션(Injection) 공격에 대비할 수 있습니다.

- **query** : 복수 개의 행을 조회하는 메소드입니다. 단일 행을 조회하는 경우 **queryForObject 메소드**를 사용합니다. 첫 번째 파라미터는 SQL 쿼리문을 문자열로 전달합니다. 두 번째 파라미터는 람다식을 사용하여 조회한 결과의 각 행을 customer 객체로 변환합니다. 마지막 파라미터는 SQL 쿼리문의 인덱스 파라미터(?)의 값을 콤마로 구분하여 지정합니다. query 메소드가 반환한 stream 객체의 forEach 메소드를 호출하여 조회한 결과를 확인합니다.

## 실행결과 확인

![INSERT, SELECT 쿼리문 실행 결과 확인](<./Accessing_Relational_Data_using_JDBC_with_Spring(4).png>)

# 레퍼런스

- [CommandLineRunner](https://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/CommandLineRunner.html)
- [Arrays](https://docs.oracle.com/javase/7/docs/api/java/util/Arrays.html)
- [Stream](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)
