## Exercise 1 : The Hello Application

첫번째 예제는 Spring Boot를 기반으로 HTTP 서비스를 제공하는 "Hello" 애플리케이션이다. 
아래처럼 Path Parameter를 이름으로 받아서 MySQL에서 데이터를 조회해서 응답을 보내는 간단한 서비스 이다.  

```bash

$ curl http://localhost:8080/sayHello/John
Hello, John!

$ curl http://localhost:8080/sayHello/Vector
Hello, Vector! Committing crimes with both direction and magnitude!

```


애플리케이션을 전체적인 윤곽은 다음과 같다. 

Jaeger Client library를 사용하기 위해 pom.xml에 Dependency를 추가했다.  

```xml
//pom.xml
<dependency>
    <groupId>io.jaegertracing</groupId>
    <artifactId>jaeger-client</artifactId>
    <version>0.31.0</version>
</dependency>
```

실행은 다음과 같이 maven wrapper를 이용해서 실행한다. 

```bash
$ ./mvnw spring-boot:run -Dmain.class=exercise1.HelloApp
```


MySQL에서 JPA를 이용해서 "Person" 조회를 위한 Entity가 /lib/people/Person.java에 존재한다. 

```java
//lib/people/Person.java
@Entity
@Table(name = "people")
public class Person {  
    @Id
    private String name;
 
    @Column(nullable = false)
    private String title;
 
    @Column(nullable = false)
    private String description;

    public Person() {}

    public Person(String name) {
        this.name = name;
    }
```

```java
//lib/people/PersonRepository.java
public interface PersonRepository extends CrudRepository<Person, String> {
}

```

JPA, MySQL Connection 정보

```properties
# src/main/resources/application.properties
spring.jpa.hibernate.ddl-auto=none
spring.datasource.url=jdbc:mysql://localhost:3306/chapter04
spring.datasource.username=root
spring.datasource.password=mysqlpwd
```

Data Model과 Repository interface를 Auto-discover하도록 "lib.people" 패키지 설정 

```java
// HelloApp.java
@EnableJpaRepositories("lib.people") 
@EntityScan("lib.people")
@SpringBootApplication
public class HelloApp {

    public static void main(String[] args) {
        SpringApplication.run(HelloApp.class, args);
    }
}
```

REST 서비스를 위한 컨트롤러

```java
// HelloController.java
@RestController
public class HelloController {

    @Autowired
    private PersonRepository personRepository;

    @GetMapping("/sayHello/{name}")
    public String sayHello(@PathVariable String name) {
        Person person = getPerson(name);
        String response = formatGreeting(person);
        return response;
    }

    private Person getPerson(String name) { ... }

    private String formatGreeting(Person person) { ... }
}

```

실행을 위해서는 
  1. MySQL를 docker로 먼저 실행하고 이 레포지토리에서 제공되는 `"database.sql"`를 실행해서 데이터를 적재한다. 
  2. mvnw를 이용해 소스를 실행한다. 

```bash
$ docker run -d --name mysql56 -p 3306:3306 \
-e MYSQL_ROOT_PASSWORD=mysqlpwd mysql:5.6


$ docker exec -i mysql56 mysql -uroot -pmysqlpwd < $CH04/database.sql
Warning: Using a password on the command line interface can be insecure.


$./mvnw spring-boot:run -Dmain.class=exercise1.HelloApp
```

