---
layout: post
title:  "[SpringBoot와 AWS로 웹서비스 출시]2장. JPA로 간단한 API 만들기"
date:   2022-01-12 23:29:00
author: Heeveloper
categories: SpringBoot
tags: SpringBoot Gradle AWS EC2 JPA
excerpt: SpringBoot
mathjax : false
---
{% include adsense_content.html %}

* content
{:toc}

이번 시간에는 JPA를 사용하여 간단한 API를 만들 예정입니다.

프로젝트 코드는 [Github](https://github.com/heeveloper/springboot-webservice)에 업로드되어 있으며, 본 세션의 코드는 브랜치 [chapter2](https://github.com/heeveloper/springboot-webservice/tree/chapter2)에서 작업하였습니다.


## 2-1. 도메인 코드 만들기

지난 시간에 이어 아래와 같이 controller 패키지와 domain 패키지를 나누어 생성합니다.

<img src="/img/SpringbootWebService/2-1.png">

Post.java

~~~java
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@Entity
@Table(name = "post")
public class Post {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    @Builder
    public Post(String title, String content, String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }
}
~~~

여기서 Post 클래스는 실제 DB 테이블과 매칭되는 클래스로서, `Entity` 클래스라고 합니다.  
JPA를 사용하면 DB 데이터에 작업할 경우 쿼리를 사용하기 보다는 이 Entity 클래스의 수정을 통해 작업합니다.  
위에서 사용한 JPA에서 제공하는 어노테이션들은 아래와 같습니다.

- `@Entity`
	- 테이블과 매핑될 클래스임을 나타냅니다.
	- underscore(_) case로 이름을 매핑합니다. (ex. SalesManager.java -> sales_manager table)

- `@Table(name="post")` 
	- 혹시나 Entity 클래스명과 다르게 테이블명을 설정하고 싶을 때 해당 어노테이션 옵션으로 변경할 수 있습니다.

- `@Id`
	- 해당 테이블의 PK 필드를 지정합니다.

- `@GeneratedValue`
	- PK 생성 규칙을 나타냅니다.
	- 기본값은 AUTO이며, MySQL의 auto_increment와 같이 자동증가하는 정수형 값이 됩니다.
    - GenerationType.IDENTITY 옵션을 주어 데이터베이스에 위임(MySQL) 하였습니다.
	- **SpringBoot 2.0**부터는 해당 옵션 추가해야 auto_increment가 됩니다!

- `@Column`
	- 테이블의 컬럼을 의미하며, 선언하지 않더라도 Entity 클래스의 필드는 기본적으로 모두 컬럼이 됩니다.
	- 기본값 외에 변경이 필요한 옵션이 있는 경우 사용합니다.  
	ex) 문자열의 경우 VARCHAR(255)이 기본인데 사이즈를 늘리거나( `title`), 타입을 TEXT로 바꿀 때(`content`) 사용.

그 외 보이는 어노테이션은  프로젝트 생성시 추가했던 **Lombok 라이브러리의 어노테이션**들 입니다. 

- `@NoArgsConstructor` : 기본 생성자 자동 추가  
	- `access = AccessLevel.PROTECTED` : 기본생성자 접근 권한을 protected로 제한.  
	Entity 클래스를 프로젝트 코드상에서 기본생성자로 생성하는 것은 막되, JPA에서 Entity 클래스를 생성하는 것은 허용하기 위해 추가.

- `@Getter` : 클래스 모든 필드에 Getter 메소드 자동 생성

- `@Builder` : 해당 클래스에 빌드 패턴 클래스 생성. (생성자 상단에 선언시 생성자에 포함된 필드만 빌드에 포함)

서비스 구축 단계에선 테이블(Entity 클래스) 설계 및 수정이 빈번하게 변경되는데요, Lombok의 어노테이션을 사용하면 코드 변경량을 최소화 시켜주기 때문에 강력 추천하는 라이브러리입니다.
<br>  
<br>

> **중요)**  
>Entity 클래스를 생성 시 주의할 점은 DDD관점에서 Setter 메소드를 만들지 않는 것입니다.  
>setter를 사용하여 해당 클래스의 인스턴스 값들이 변경되는 경우 불변성이 깨져 안정성이 떨어지고, 코드상으로 해당 변경의 의미가 희미해져 차후 도메인이 복잡해짐에 따라 유지, 보수 비용이 많이 발생하게 됩니다.  
>
>해당 필드값 변경이 필요한 경우 아래와 같이 그 목적과 의도를 나타낼 수 있는 메소드를  Entity 클래스 내부에 추가하거나 새로운 객체를 생성하는 것을 권장합니다.
>
>~~~java
>public class Money {
>  private int value;
>  
>  public Money add(Money money) {
>    return new Money(this.value + money.getValue());
>  }
>}
>~~~

<br>
{% include adsense_content.html %}
<br>

PostRepository.java
~~~java
public interface PostRepository extends JpaRepository<Post, Long> {
}
~~~

JPA에선 Repository라고 부르고 ibatis/MyBatis 등에선 DAO라고 부르는 DB Layer 접근자입니다.  
인터페이스 생성 후, `JpaRepository<Entity클래스, PK타입>` 을 상속하면 기본적인 CRUD 메소드가 자동 생성됩니다.  
따로 `@Repository` 를 추가할 필요는 없습니다. 

이제 잘 동작하는지 테스트 코드로 검증해보겠습니다.

## 2-2. 테스트 코드 만들기

PostRepositoryTest.java

~~~java
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.*;


@ExtendWith(SpringExtension.class)
@SpringBootTest
public class PostRepositoryTest {
    
    @Autowired
    PostRepository postRepository;

    @Test
    public void 게시글저장_불러오기() {
        // given
        postRepository.save(Post.builder()
                .title("테스트 타이틀")
                .author("테스트 작가")
                .content("테스트 컨텐츠")
                .build()
        );

        // when
        List<Post> posts = postRepository.findAll();

        // then
        Post post = posts.get(0);
        assertThat(post.getTitle(), is("테스트 타이틀"));
        assertThat(post.getAuthor(), is("테스트 작가"));
        assertThat(post.getContent(), is("테스트 컨텐츠"));
    }
}
~~~

위 코드는 **JUnit5**라는 테스트 프레임워크로 작성된 코드인데요, 예상값과 결과값을 비교하여 검증할 수 있습니다.  
JUnit5는 spring-boot-starter-test에 들어있기 때문에 별도로 build.gradle에 추가하실 필요가 없습니다.  
해당 라이브러리에는 JUnit4와의 하위 호환성을 가지고 있으며 별다른 설정이 없으면 JUnit5를 사용합니다.  

- `@ExtendWith(SpringExtension.class)`  
	- JUnit4에선 `RunWith(SpringRunner.class)` 으로 사용되던 게 변경되엇습니다.

- `given, when, then`
	- BDD(Behaviour-Driven Development)에서 사용하는 용어입니다.
	- JUnit에선 이를 명시적으로 지원해주지 않아 주석으로 표현하였습니다.

- `assetThat`
	- 결과값 비교에 사용되는 메서드로 `spring-boot-starter-test` 라이브러리에 포함되어 있는 `Hamcrest`에서 제공하는 기능입니다.

<br>
아래와 같이 테스트 메소드 `게시글저장_불러오기()`를 실행시켜 테스트 코드가 통과했음을 확인 할 수 있습니다 👍

<img src="/img/SpringbootWebService/2-2.png">

> Tip)
>
> DB가 설치가 안되어 있는데 Repository를 사용할 수 있는 이유는, SpringBoot에서의 테스트 코드는 메모리 DB인 H2를 기본적으로 사용하기 때문입니다.  
> 테스트 코드를 실행하는 시점에 H2 DB를 실행시키며 테스트가 끝나면 같이 종료됩니다.

다음은 직접 어플리케이션 코드를 짜서 진짜 DB에 데이터가 들어가는지 확인해보겠습니다.

{% include adsense_content.html %}

## 2-3 Controller & Service & DTO 구현

아래와 같이 post에 대한 `controller 패키지`와 `service 패키지`를 생성해줍니다.  
사실 이정도 작은 규모의 프로젝트에는 이러한 패키징이 크게 필요는 없지만 최대한 DDD 관점에서 패키지를 나누었습니다. 
> **기본 Architecture**  
> Controller - (AppService) - DomainService - Infra(DB, ..)

<img src="/img/SpringbootWebService/2-3.png">

PostController.java

~~~java
@RestController
@AllArgsConstructor
public class PostController {
    private PostService postService;

    @PostMapping("post")
    public void add(@RequestBody AddPostRequestDto request) {
        postService.add(request);
    }
}
~~~

PostService.java

~~~java
@Service
@AllArgsConstructor
public class PostService {
    private PostRepository postRepository;

    @Transactional
    public void add(AddPostRequestDto requst) {
        postRepository.save(requst.toEntity());
    }
}
~~~

위의 PostService는 DDD에서 **도메인 서비스 계층(Domain Service Layer)**에 속하는데요, 도메인 로직을 직접 구현하는 계층이라고 할 수 있습니다.
(자세한 내용이 궁금하시면 [DDD Start 7장. 도메인 서비스](https://heeveloper.github.io/2020/08/27/07-%EB%8F%84%EB%A9%94%EC%9D%B8-%EC%84%9C%EB%B9%84%EC%8A%A4/)을 참고해주세요😀)

또한 `@Transaction` 어노테이션 추가되어 있는데요, 해당 어노테이션이 선언된 메소드는 하나의 트랜잭션으로 관리되어 보통 도메인 서비스 또는 응용 서비스에서 사용됩니다.

> Transaction이란?
>
> 일반적으로 DB 데이터를 등록/수정/삭제하는 도메인 서비스, 응용 서비스의 메소드에서 사용합니다.  
> 메소드에 이 어노테이션을 적용하게되면 내부 로직은 하나의 트랜잭션 단위로 묶이게 되며 아래 ACID 성질을 띄게 됩니다.  
>
> - 원자성(Atomicity) : 한 트랜잭션 내 작업들은 하나로 간주. 모두 성공 또는 모두 실패
> - 일관성(Consistency) : 일관성있는 데이터베이스 상태 유지
> - 격리성(Isolation) : 동시에 실행되는 트랜잭션은 서로 영향을 미치지 않는다. 미칠경우 하나만 실행.
> - 지속성(Durability) : 트랜잭션이 성공하면 결과는 항상 저장된다.
>
> 예를 들어, `@Transactional` 이 선언된 메소드에서 `save()`를 통해 10개의 데이터를 저장한다고 했을 때, 한 경우라도 Exception이 발생하면 전부 반영이 되지 않는다고 보면 되겠습니다.


또한 `postService`와 `postRepository` 필드에 `@Autowired`가 없습니다.  
스프링프레임워크에선 Bean을 주입하는 방식이 아래와 같이 있는데요,

- `@Autowired`
- setter
- 생성자

이 중 가장 권장하는 방식이 생성자로 주입하는 방식입니다. (`@Autowired` 는 비권장방식)  
위 코드에서 생성자는 `@AllArgsConstructor` 를 사용하여 자동 생성해주어, 해당 클래스의 의존성 관계가 변경될 때마다 생성자 코드를 수정하는 번거로움을 해결해주고 있습니다.

이제 Controller에서 사용할 요청 DTO 클래스를 생성하겠습니다.

AddPostRequestDto.java

~~~java
@Getter
@Setter
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class AddPostRequestDto {
    private String title;
    private String author;
    private String content;

    public Post toEntity() {
        return Post.builder()
                .title(title)
                .author(author)
                .content(content)
                .build();
    }
}
~~~

이 경우엔 `@Setter` 를 선언해주었는데요, Controller의 `@RequestBody` 를 통해 외부에서 데이터를 받는 경우엔 **기본생성자 + setter 메소드를 통해서만 값이 할당**되기 때문입니다.

> **중요)**  
> RequestDTO를 보면 Entity 클래스와 유사한 형태임에도 따로 생성했는데요, 절대로 테이블과 매핑되는 Entity 클래스를 Request/Response 로 사용하면 안됩니다.  
>
> Entity 클래스는 가장 Core한 도메인 클래스라고 볼 수 있는데요, 수많은 비즈니스 로직들이 Entity 클래스를 기준으로 동작하게 됩니다.  
> Entity 클래스의 변경은 곧 비즈니스 로직 변경이라는 큰 의미를 가진 반면, Request/Response 클래스는 View를 위한 클래스이기 때문에 가볍게 자주 변경될 수 있습니다.
>
> 따라서 View Layer와 Domain Layer는 철저하게 역할을 분리하는 것이 좋습니다.

Service 메소드가 추가되었으니, 잘 작동하는지 간단한 테스트 코드를 작성하여 검증해보겠습니다.

src/test/java/com/heeveloper/webservice/service 패키지를 생성 후, PostServiceTest 클래스를 생성합니다.

~~~java
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.*;


@ExtendWith(SpringExtension.class)
@SpringBootTest
public class PostServiceTest {

    @Autowired
    private PostService postService;

    @Autowired
    private PostRepository postRepository;

    @AfterEach
    public void cleanup() {
        postRepository.deleteAll();
    }

    @Test
    public void Dto데이터가_posts테이블에_저장된다 () {
        // given
        AddPostRequestDto dto = AddPostRequestDto.builder()
                .author("heeveloper")
                .content("테스트content")
                .title("테스트title")
                .build();

        // when
        postService.add(dto);

        // then
        Post post = postRepository.findAll().get(0);
        assertThat(post.getAuthor(), is("heeveloper"));
        assertThat(post.getContent(), is("테스트content"));
        assertThat(post.getTitle(), is("테스트title"));
    }
}
~~~

<img src="/img/SpringbootWebService/3-5.png">

## 2-4 Postman + 웹 콘솔로 검증

별도의 입력화면이 아직 없기 때문에 Postman을 이용해 Post 데이터를 전송해 검증해보겠습니다.   

<br>
Local 환경에선 DB로 H2를 사용하기 때문에 SpringBoot에서 H2를 활성화시키도록 옵션을 추가하겠습니다.   
src/main/resources에 있는 application.properties를 application.yml로 변경후 아래와 같이 코드를 입력합니다.

<img src="/img/SpringbootWebService/2-4.png">

~~~yaml
spring:
  h2:
    console:
      enabled: true
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb
    #username: sa
    #password: 1234
~~~

username과 password를 설정하지 않으면 sa 계정으로 바로 로그인이 가능합니다.

> **Tip)**  
> application.properties를 사용해도 무방하지만, yaml로 변경한 이유는 properties에 비해 유연한 구조를 가졌기 때문입니다.  
> yml은 상위 계층에 대한 표현 및 List 등을 완전하게 표현할 수가 있습니다.   
> 최근 많은 도구들이 yml 설정을 지원하기 때문에 이참에 시작해보는 것을 추천드립니다. 

application.yml 설정이 끝났다면 다시 Application.java를 실행시키고, 브라우저에 http://localhost:8080/h2-console 로 접속합니다.

<img src="/img/SpringbootWebService/2-5.png" height="500" length="500">

connect 버튼을 눌러 접속한 후, 조회 쿼리를 실행합니다.

~~~java
SELECT * FROM POST
~~~

<br>

<img src="/img/SpringbootWebService/2-6.png">

현재는 데이터가 없는 것이 확인되었습니다.  
이제 Postman을 사용하여 데이터를 전송해보겠습니다.

<img src="/img/SpringbootWebService/2-7.png">

Send 버튼을 눌러 전송후 다시 localhost:8080/h2-console에서 조회 쿼리를 입력하면 아래와 같이 데이터가 입력된 것을 확인할 수 있습니다 👍

<img src="/img/SpringbootWebService/2-8.png">

{% include adsense_content.html %}

## 2-5 생성시간/수정시간 자동화 - JPA Auditing

보통 Entity에는 유지 보수측면에서 해당 데이터의 생성시간과 수정시간이 중요하기 때문에 해당 필드를 갖게됩니다.  
때문에 매번 DB에 insert, update 시에 아래와 같이 직접  시간을 입력하게 되는데요,

~~~java
public void savePost() {
  ...
  post.setCreateDate(new LocalDate());
  postRepository.add(post);
  ...
}
~~~

이러한 단순하고 반복적인 코드가 모든 테이블과 서비스 메소드에 포함되는 것을 막기 위해 **JPA Auditing**을 사용할 수 있습니다.

기존 Post 클래스에서 아래와 같이 생성시간, 수정시간 필드와 함께 Auditing 기능을 어노테이션으로 적용시켜줍니다.

Application.java

~~~java
@EnableJpaAuditing  // JPA Auditing 활성화
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}

~~~



~~~java
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@Entity
@Table(name = "post")
@EntityListeners(AuditingEntityListener.class) // Auditing 기능 포함
public class Post {

    @Id
    @GeneratedValue
    private Long id;

    @Column(length = 500, nullable = false)
    private String title;

    @Column(columnDefinition = "TEXT", nullable = false)
    private String content;

    private String author;

    @CreatedDate
    private LocalDateTime createdAt; // 생성 시간

    @LastModifiedDate
    private LocalDateTime modifiedAt; // 수정 시간


    @Builder
    public Post(String title, String content, String author) {
        this.title = title;
        this.content = content;
        this.author = author;
    }
}
~~~

<br>

- `@EntityListeners(AuditingEntityListener.class)` : Auditing 기능을 포함시킵니다.

- `@CreatedDate` : Entity가 생성되어 저장될 때 시간이 자동 저장됩니다.

- `@LastModifiedDate` : 조회한 Entity가 값을 변경할 때 시간이 자동 저장됩니다.

> **LocalDateTime** 
>
> Java8부터 LocalDate와 LocalDateTime이 등장하였는데요,  
> 그간 Java의 기본 날짜 타입인 Date의 문제점을 고친 타입이라 Java8 이상부턴 무조건 쓴다고 보면 됩니다.  
>
> 또 이전 SpringDataJpa 버전에선 LocalDate와 LocalDateTime이 Datebase에 저장시 제대로 전환이 안되는 이슈가 있는데요,  
> 현재는 SpringDateJpa 코어 모듈인 Hibernate core 5.2.10부터 해결되어서 따로 신경쓸 필요가 없습니다.  
> (기본적으로 Springboot 2.0부터 Hibernate 5.2.16을 지원하였습니다.)

<br>

다시 Application을 실행시켜 Postman으로 동일하게 요청후 h2 console에서 확인해보시면,

<img src="/img/SpringbootWebService/2-9.png">

기존에 없던 `CREATED_AT`, `MODIFIED_AT` 이 추가되고 값이 들어간 것을 확인할 수 있습니다! 👏

<br>
<br>

SpringBoot를 해보신 분들이라면 쉬우셨을텐데요, 실제로 SpringBoot & JPA는 회사에서도 많이 사용하는 방법이니  
처음이신 분들도 직접 해보시면서 익숙해 지시는 것을 추천드리겠습니다. 😄  

다음 시간엔 View Template을 사용하여 실제 보여지는 화면을 만들어보겠습니다.

감사합니다! 