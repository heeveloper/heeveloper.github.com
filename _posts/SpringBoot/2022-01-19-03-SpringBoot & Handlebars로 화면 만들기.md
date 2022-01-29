---
layout: post
title:  "[SpringBoot와 AWS로 웹서비스 출시]3장. Handlebars로 게시판 화면 만들기"
date:   2022-01-19 20:49:00
author: Heeveloper
categories: SpringBoot
tags: SpringBoot Gradle AWS EC2 JPA Handlebars
excerpt: SpringBoot
mathjax : false
---
{% include adsense_content.html %}

* content
{:toc}

이번 시간에는 SpringBoot & Handlebars로 간단한 게시판 화면을 만들 예정입니다.

프로젝트 코드는 [Github](https://github.com/heeveloper/springboot-webservice)에 업로드되어 있으며, 본 세션의 코드는 브랜치 [chapter3](https://github.com/heeveloper/springboot-webservice/tree/chapter3)에서 작업하였습니다.

## 3-1 Handlebars란?

Handlebars는 흔히 사용하는 Freemarker, Velocity와 같은 서버 템플릿 엔진입니다.  
JSP는 서버 템플릿 역할만 하지 않기 때문에 JSP와 완전히 똑같은 역할을 한다고 볼순 없지만, **View 측면**만 봤을 땐 똑같다고 할 수 있습니다.

즉 URL 요청시, 파라미터와 상태에 맞춰 적절한 HTML 응답을 생성해 전달하는 역할이라고 보시면 됩니다.

> **Tip)**
>
> JSP, Freemarker, Velocity, Thymeleaf, Handlebars 등의 템플릿 엔진이 꾸준히 업데이트되어 오고 있지만, SpringBoot에서는 가장 높은 성능을 보이는 Handlebars를 개인적으로 추천합니다.  
> (Spring 진영에선 Thymeleaf를 밀고 있습니다.)  
>
> 1) 문법이 다른 템플릿 엔진보다 간단하고  
> 2) 로직 코드를 사용할 수 없어 **View의 역할과 서버의 역할을 명확히 분리**할 수 있으며  
> 3) Handlebars.js와 Handlebars.java 2가지가 다 있어, 하나의 문법으로 클라이언트 템플릿, 서버 템플릿 모두 사용할 수 있습니다.
>
> 개인적으로 View 템플릿 엔진은 View의 역할에만 충실하면 된다고 생각합니다.  
> 너무 많은 기능을 제공하면 로직이 API와 View 템플릿 엔진, JS에 분산되고 유지보수하기가 어려워집니다. 



## 3-2. Handlebars 연동

Handlebars는 아직 정식 Springboot Starter 패키지가 존재하지 않아, 많은 분들이 사용중이신 [handlebars-spring-boot-start](https://github.com/allegro/handlebars-spring-boot-starter) 를 사용할 예정입니다.

### 의존성 추가

build.gradle에 0.3.4 버전의 handlebars를 추가해줍니다.

<img src="/img/SpringbootWebService/3-1.png">

~~~java
compile 'pl.allegro.tech.boot:handlebars-spring-boot-starter:0.3.4'
~~~

의존성을 추가하면 추가 설정없이 설치가 끝납니다.  

Handlebars도 다른 서버 템플릿 스타터 패키지와 마찬가지로 기본 경로는 src/main/resources/templates가 됩니다.

> **Tip)**
>
> 혹시 IntelliJ를 사용중이시라면 아래와 같이 Handlebars 플러그인을 설치하여 문법체크와 같은 지원 등을 받을 수 있습니다.
>
> <img src="/img/SpringbootWebService/3-2.png">

{% include adsense_content.html %}

### 메인 페이지 생성

실제로 사용할 Handlebars 파일을 생성하겠습니다.  
src/main/resources/templates에 **main.hbs** 파일을 생성합니다.

~~~handlebars
<!DOCTYPE HTML>
<html>
<head>
    <title>스프링부트 웹서비스</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />

</head>
<body>
    <h1>스프링부트로 시작하는 웹 서비스</h1>
</body>
</html>
~~~

메인 페이지가 생성되었으니, URL 요청시 main.hbs를 응답할 수 있는 Controller를 만들어보겠습니다.  
controller/web 패키지 안에 WebController를 만들겠습니다.

~~~java
@Controller
@AllArgsConstructor
public class WebController {

    @GetMapping("/")
    public String main() {
        return "main";
    }
}
~~~

handlebars-spring-boot-starter 덕분에 컨트롤러에서 반환되는 문자열의 앞과 뒤에 자동으로 path와 파일 확장자가 지정됩니다.  
(prefix: src/main/resources/templates, suffix: .hbs)  
즉, 최종적으로 src/main/resources/templates/main.hbs로 전환되어 View Resolver가 처리하게 됩니다.  
(View Resolver : URL 요청 결과를 전달할 타입과 값을 지정하는 관리자)

잘 적용되었는지 검증하기 위해 테스트 코드를 작성해보겠습니다.

### 메인 페이지 테스트 코드

src/test/java/com/heeveloper/webservice/controller/web 패키지를 생성 후 WebControllerTest 클래스를 생성하겠습니다.

~~~java
import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.Matchers.*;


@ExtendWith(SpringExtension.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
public class WebControllerTest {
    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    public void 메인페이지_로딩() {
        // when
        String body = this.restTemplate.getForObject("/", String.class);

        // then
        assertThat(body, containsString("스프링부트로 시작하는 웹 서비스"));
    }
}
~~~

TestRestTemplate으로 "/"를 호출했을 때 main.hbs에 포함된 코드들이 있는지 확인하는 간단한 테스트입니다.

<img src="/img/SpringbootWebService/3-3.png">

정상적으로 PASS되었습니다!  
이제 실제로 화면이 나오는지 확인하기 위해 Application 메인 메소드를 실행 후 localhost:8080으로 접속해보겠습니다.

<img src="/img/SpringbootWebService/3-4.png">

정상적으로 화면이 노출되었는데요👏  
기본적인 화면이 확인되었으니, 기능 화면을 하나씩 추가해보겠습니다! 

{% include adsense_content.html %}

## 3-3. 게시글 등록

이전 시간에 게시글을 등록하는 Controller와 Service 클래스는 이미 생성하였었는데요,  
Handlebars로 입력 화면을 생성해 보겠습니다.

### 입력 화면

백엔드 개발자가 CSS를 전부 구성하기에는 무리가 있어 오픈소스인 [Bootstrap](https://getbootstrap.com/)을 활용하겠습니다.  

부트스트랩, jQuery 등 프론트엔드 라이브러리를 사용할 수 있는 방법은 크게 2가지가 있습니다.  
하난 외부 CDN을 사용하는 것이고, 다른 하나는 직접 라이브러리를 받아서 사용하는 방법입니다.  
(npm/bower + grunt/gulp/webpack 등을 통한 방법도 후자에 속한다고 보시면 됩니다.)

CDN을 통한 방법이란 외부 서버를 통해 라이브러리를 받는 방식을 얘기하는데요, 본인 프로젝트에 직접 다운받아 사용할 필요도 없고, 사용 방법도 HTML/JSP/Handlebars에 코드만 추가하면 되니깐 굉장히 간단합니다.  
하지만 이 방법을 실제 서비스에는 잘 사용하지는 않는데요, 결국에 외부 서비스에 의존성이 생기기 때문입니다. 

그래서 우리는 사용할 2개의 라이브러리를 직접 다운받아 사용하겠습니다.

[Bootstrap](https://getbootstrap.com/)

- 우측 상단의 Download 버튼을 클릭해 Compiled CSS and JS 최신 버전(v4.6.1)을 다운받습니다.
- bootstrap-4.6.1-dist 폴더 아래에 있는 css 폴더에서 bootstrap.min.css를 src/main/resources/static/**css/lib**으로 복사합니다.
- bootstrap-4.6.1-dist 폴더 아래에 있는 js 폴더에서 bootstrap.min.js를 src/main/resources/static/**js/lib**으로 복사합니다.

[jQuery](https://jquery.com/download/)

- `Download the compressed, production jQuery 3.6.0` 을 클릭해 다운받습니다.
- jquery-3.6.0.min.js -> jquery.min.js로 이름 변경
- src/main/resources/static/js/lib으로 복사해 줍니다.

<img src="/img/SpringbootWebService/3-6.png">

파일을 다 복사하셨으면 main.hbs 파일에 아래와 같이 라이브러리를 추가해줍니다.

~~~handlebars
<!DOCTYPE HTML>
<html>
<head>
    <title>스프링부트 웹서비스</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />

    <!--부트스트랩 css 추가-->
    <link rel="stylesheet" href="/css/lib/bootstrap.min.css">
</head>
<body>
    <h1>스프링부트로 시작하는 웹 서비스</h1>

    <!--부트스트랩 js, jquery 추가-->
    <script src="/js/lib/jquery.min.js"></script>
    <script src="/js/lib/bootstrap.min.js"></script>
</body>
</html>
~~~

css와 js의 호출 주소를 보면 `/` 로 바로 시작하는데요,  
SpringBoot는 기본적으로 src/main/resources/static을 기본 path(`/`)로 지정합니다.

main.hbs를 보시면 css는 `<head>` 에, js는 `<body>` 최하단에 두었는데요,  
이렇게 하는 이유는 **페이지 로딩 속도를 높이기 위함**입니다.

HTML은 최상단에서부터 코드가 실행되기 때문에 head가 다 실행되고 나서야 body가 실행되는데요,  
js의 로직 연산이 길어질수록 body 부분의 실행이 늦어지기 때문에 js는 body 하단에 두어 화면이 다 그려진 뒤에 호출하는 것이 좋습니다.

반면 css는 화면을 그리는 역할이기 때문에 head에 불러오는 것이 좋습니다.

추가로, bootstrap.js의 경우 jquery가 꼭 있어야하기 때문에 jquery를 먼저 호출하도록 작성하였습니다.  
(이런 상황을 bootstrap.js가 jquery에 의존한다라고 합니다.)

라이브러리는 다 추가되었으니, 이어서 main.hbs에 입력 화면을 만들겠습니다.

~~~handlebars
<!DOCTYPE HTML>
<html>
<head>
    <title>스프링부트 웹서비스</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />

    <!--부트스트랩 css 추가-->
    <link rel="stylesheet" href="/css/lib/bootstrap.min.css">
</head>
<body>
    <h1>스프링부트로 시작하는 웹 서비스</h1>
    </div>

    <!--  Modal 버튼 영역-->
    <div class="col-md-12">
        <button type="button" class="btn btn-primary" data-toggle="modal" data-target="#savePostModal">글 등록</button>
    </div>
    
    <!-- Modal 영역 -->
    <div class="modal fade" id="savePostModal" tabindex="-1" role="dialog" aria-labelledby="savePostLabel" aria-hidden="true">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="savePostLabel">게시글 등록</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <div class="modal-body">
                    <form>
                        <div class="form-group">
                            <label for="title">제목</label>
                            <input type="text" class="form-control" id="title" placeholder="제목을 입력하세요">
                        </div>
                        <div class="form-group">
                            <label for="author"> 작성자 </label>
                            <input type="text" class="form-control" id="author" placeholder="작성자를 입력하세요">
                        </div>
                        <div class="form-group">
                            <label for="content"> 내용 </label>
                            <textarea class="form-control" id="content" placeholder="내용을 입력하세요"></textarea>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-dismiss="modal">취소</button>
                    <button type="button" class="btn btn-primary" id="btn-save">등록</button>
                </div>
            </div>
        </div>
    </div>
    
    <!--부트스트랩 js, jquery 추가-->
    <script src="/js/lib/jquery.min.js"></script>
    <script src="/js/lib/bootstrap.min.js"></script>
    <!--custom js 추가-->
    <script src="/js/app/main.js"></script>
</body>
</html>
~~~

{% include adsense_content.html %}

여기까지 하시면 입력 UI 부분이 완성되었습니다.  
Application을 실행시키고, localhost:8080으로 접근해보겠습니다.

<img src="/img/SpringbootWebService/3-7.png">

Modal 화면의 등록 버튼은 아직 기능이 없는데요,  
그래서 static/js 아래에 app 디렉토리를 생성 후, main.js를 만들겠습니다.

~~~js
var main = {
    init : function () {
        var _this = this;
        $('#btn-save').on('click', function () {
            _this.save();
        });
    },
    save : function () {
        var data = {
            title: $('#title').val(),
            author: $('#author').val(),
            content: $('#content').val()
        };

        $.ajax({
            type: 'POST',
            url: '/post',
            dataType: 'json',
            contentType:'application/json; charset=utf-8',
            data: JSON.stringify(data)
        }).done(function() {
            alert('글이 등록되었습니다.');
            location.reload();
        }).fail(function (error) {
            alert(error);
        });
    }

};

main.init();
~~~

작성한 main.js를 main.hbs에 추가하겠습니다.

~~~handlebars
    <!--부트스트랩 js, jquery 추가-->
    <script src="/js/lib/jquery.min.js"></script>
    <script src="/js/lib/bootstrap.min.js"></script>

    <!--custom js 추가-->
    <script src="/js/app/main.js"></script>

</body>
~~~

다시 Application을 실행시켜 localhost:8080으로 접속한 뒤 localhost:8080/h2-console에 접속하면 아래와 같이 데이터가 들어간 것을 확인할 수 있습니다! 😀

<img src="/img/SpringbootWebService/3-8.png">

{% include adsense_content.html %}

## 3-4. 게시글 목록

### 사전 데이터 입력

실제 코드를 작성하기 전에 먼저 해야하는 작업이 있는데요  
지금까지 h2-console을 통해 DB를 접속하시면 아셨듯이 H2 DB는 메모리 DB이기 때문에 프로젝트를 실행할 때마다 초기화가 됩니다.  

게시글 목록 UI가 잘 나오는 지 확인/수정하는 과정에서 사전에 데이터를 입력하는 설정 작업을 진행하겠습니다.  
resources 아래에 **data-h2.sql** 파일을 생성합니다.

~~~sql
insert into POST (title, author, content, created_at, modified_at) values ('테스트1', '123', '테스트1의 본문', now(), now());
insert into POST (title, author, content, created_at, modified_at) values ('테스트2', '456', '테스트2의 본문', now(), now());
~~~

그리고 위 insert qeury sql 파일을 프로젝트 실행시 자동으로 수행되도록 필요한 설정들을 application.yml 코드에 추가해줍니다.

~~~yaml
spring:
  profiles:
    active: local # 기본 환경 선택

# local 환경
---
spring:
  config:
    activate:
      on-profile: local
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: create-drop
    defer-datasource-initialization: true # 초기화시 data-h2.sql 채우기 위해서 필요한 옵션
  h2:
    console:
      enabled: true
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb # db 연결 위한 설정
    #username: sa
    #password: 1234
  sql:
    init:
      data-locations: classpath:data-h2.sql
~~~

코드가 대폭 수정되었는데요,  
가장 상단의 spring.profile 옵션은 Application 실행시 파라미터로 넘어오는 게 없으면 active 값을 보게 됩니다. 추후 개발 환경(local 혹은 dev 등)과 운영 환경(real 또는 prod 등)을 구분하기 위해 사용합니다. 

`---`을 기준으로 상단은 공통 영역, 하단은 각 profile의 설정 영역입니다. 

다시 Application을 실행 후 localhost:8080/h2-console 접속하여 Post 테이블 조회시 해당 데이터가 들어가 있는 것을 볼 수 있습니다.  
<img src="/img/SpringbootWebService/3-9.png">

### 목록 UI

main.hbs에 목록 UI를 추가해 보겠습니다.

~~~handlebars
<!DOCTYPE HTML>
<html>
<head>
    <title>스프링부트 웹서비스</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />

    <!--부트스트랩 css 추가-->
    <link rel="stylesheet" href="/css/lib/bootstrap.min.css">
</head>
<body>
    <h1>스프링부트로 시작하는 웹 서비스</h1>

    <!-- 목록 출력 영역 -->
    <table class="table table-horizontal table-bordered">
        <thead class="thead-strong">
        <tr>
            <th>게시글번호</th>
            <th>제목</th>
            <th>작성자</th>
            <th>생성일</th>
            <th>수정일</th>
        </tr>
        </thead>
        <tbody id="tbody">
        {{#each post}}
            <tr>
                <td>{{id}}</td>
                <td>{{title}}</td>
                <td>{{author}}</td>
                <td>{{createdAt}}</td>
                <td>{{modifiedAt}}</td>
            </tr>
        {{/each}}
        </tbody>
    </table>

    <!--  Modal 버튼 영역-->
    <div class="col-md-12">
        <button type="button" class="btn btn-primary" data-toggle="modal" data-target="#savePostModal">글 등록</button>
    </div>

    <!-- Modal 영역 -->
    <div class="modal fade" id="savePostModal" tabindex="-1" role="dialog" aria-labelledby="savePostLabel" aria-hidden="true">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" id="savePostLabel">게시글 등록</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                        <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <div class="modal-body">
                    <form>
                        <div class="form-group">
                            <label for="title">제목</label>
                            <input type="text" class="form-control" id="title" placeholder="제목을 입력하세요">
                        </div>
                        <div class="form-group">
                            <label for="author"> 작성자 </label>
                            <input type="text" class="form-control" id="author" placeholder="작성자를 입력하세요">
                        </div>
                        <div class="form-group">
                            <label for="content"> 내용 </label>
                            <textarea class="form-control" id="content" placeholder="내용을 입력하세요"></textarea>
                        </div>
                    </form>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-secondary" data-dismiss="modal">취소</button>
                    <button type="button" class="btn btn-primary" id="btn-save">등록</button>
                </div>
            </div>
        </div>
    </div>

    <!--부트스트랩 js, jquery 추가-->
    <script src="/js/lib/jquery.min.js"></script>
    <script src="/js/lib/bootstrap.min.js"></script>

    <!--custom js 추가-->
    <script src="/js/app/main.js"></script>
</body>
</html>
~~~

`#each post`는 post라는 리스트를 순회하며 하나씩 꺼내 필드값을 채워 테이블에 출력시킵니다.

{% include adsense_content.html %}

다음 Controller와 Service, 응답 포맷인 PostResponseDto 클래스 코드를 작성하겠습니다.

~~~java
@Getter
public class PostResponseDto {
    private Long id;
    private String title;
    private String author;
    private String content;
    private String createdAt;
    private String modifiedAt;

    public PostResponseDto(Post post) {
        this.id = post.getId();
        this.title = post.getTitle();
        this.author = post.getAuthor();
        this.content = post.getContent();
        this.createdAt = toStringDateTime(post.getCreatedAt());
        this.modifiedAt = toStringDateTime(post.getModifiedAt());
    }

    private String toStringDateTime(LocalDateTime localDateTime) {
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        return Optional.ofNullable(localDateTime)
                .map(formatter::format)
                .orElse("");
    }
}
~~~

createdAt과 modifiedAt은 LocalDateTime을 사용하지 않고 String을 사용하였는데요,  
View 영역에선 LocalDateTime 타입을 모르기 때문에 인식할 수 있도록 별도 메소드를 만들어 문자열로 변환하여 등록하였습니다.

> **Tip)**
>
> Post Entity 내에서 `convertDto()`와 같은 메소드를 만들어 변환해도 되지 않나?라고 생각하실 수도 있습니다.  
> 하지만 절대 그렇게 해서는 안됩니다.  
> **DTO는 Entity를 사용해도 되지만, Entity는 DTO에 대해 전혀 모르게 코드를 구성해야합니다.**
>
> Entity는 말 그대로 가장 core한 클래스인 반면, DTO는 View 혹은 외부 환경과 관련이 있는 클래스입니다.  
> 때문에 View나 외부 요청에 따라 자주 변경될 수 있으며 또는 여러 형태의 DTO가 필요하여 `convertDto2()` 와 같이 모든 변화에 맞춰 Entity를 변경해줘야 하는 일이 발생하게 됩니다.
>
> 프로젝트 규모가 커져 프로젝트를 분리해야할 때도 Entity가 DTO에 의존하고 있으면 분리가 굉장히 어려워지기 때문에 DTO가 Entity에 의존하도록 코드를 작성하시길 바랍니다. 🙏

~~~java
@Service
@AllArgsConstructor
public class PostService {
    ...

    // 추가
    // readOnly 옵션을 추가하면 트랜잭션 범위는 유지하되, 조회기능만 남겨두어 조회 속도가 개선된다.
    @Transactional(readOnly = true)
    public List<PostResponseDto> findAllDesc() {
        return postRepository.findAll(Sort.by(Sort.Direction.DESC, "id")).stream()
                .map(PostResponseDto::new)  // .map(posts -> new PostsMainResponseDto(posts))
                .collect(Collectors.toList());
    }
}
~~~

SpringDataJpa에서 제공하는 기본 메소드인 `findAll()` 에 정렬 옵션을 주고 응답 포맷 Dto로 변환하는 작업 입니다.  
람다식을 모르면 `map(PostResponseDto::new)` 코드가 조금 생소하실 수도 있는데요 실제로 `map(posts -> new PostsMainResponseDto(posts))`와 동일합니다.

마지막으로 WebController를 변경합니다.

~~~java
@Controller
@AllArgsConstructor
public class WebController {

    private PostService postService;

    @GetMapping("/")
    public String main(Model model) {
        model.addAttribute("post", postService.findAllDesc());
        // prefix: src/main/resources/templates, suffix: .hbs
        // src/main/resources/templates/main.hbs로 전환되어 View Resolver가 처리
        return "main";
    }
}
~~~

사실 Post와 관련된 API는 Rest API Controller인 PostController에서 처리하는 것이 맞으나 본 프로젝트는 간단한  화면을 구성하기에, 직접 View을 반환하는 WebController에서 직접적으로 처리한다는 것을 참고해 주시기 바랍니다. 

{% include adsense_content.html %}

자 코드가 모두 완성되었으니 한 번 프로젝트를 재실행하면 다음고 같이 최종 화면이 노출됩니다 👏

<img src="/img/SpringbootWebService/3-9.png">

글 등록 기능을 통해 데이터가 추가되는 것을 확인해보세요



## 마무리

여기까지 SpringBoot와 JPA, Handlebar 등을 이용해서 간단한 게시판을 만들어 보았습니다.

다음 시간부터는 AWS 서버 구축, 배포환경 구축 등을 진행하겠습니다.

여기까지 오시느라 고생하셨습니다.👏👏