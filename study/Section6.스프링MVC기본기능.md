# Section 6. 스프링 MVC - 기본 기능

## 프로젝트 생성

- 스프링 부트 스타터 사이트로 이동해서 스프링 프로젝트 생성: https://start.spring.io

- packaing: jar
- Java: 11
- Dependencies: Spring Web, Thyemeleaf, Lombok



> [주의]
>
> Packaging은 War 가 아니라 Jar로 선택한다. JSP를 사용하지 않기 때문에 Jar를 사용하는 것이 좋다. 앞으로 스프링 부트를 사용하면 이 방식을 주로 사용하게 된다.
>
> Jar를 사용하면 항상 내장 서버(톰캣 등)르 사용하고, webapp 경로도 사용하지 않는다. 내장 서버 사용에 최적화 되어 있는 기능이다. 최근에는 주로 이 방식을 사용한다.
>
> War를 사용하면 내장 서버도 가능하지만 주로 외부 서버에 배포하는 목적으로 사용한다.



### Welcome 페이지 만들기

이번 장에서 학습할 내용을 편리하게 참고하기 위해 Welcome 페이지를 만들자.

스프링 부트에 jar를 사용하면 `/resources/static/` 위치에 `index.html` 파일을 두면 welcome 페이지로 처리해준다. (스프링 부트가 지원하는 정적 컨텐츠 위치에 /index.html 이 있으면 된다.)

``` html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<ul>
    <li>로그 출력
        <ul>
            <li><a href="/log-test">로그 테스트</a></li>
        </ul>
    </li>
    <!-- -->
    <li>요청 매핑
        <ul>
            <li><a href="/hello-basic">hello-basic</a></li>
            <li><a href="/mapping-get-v1">HTTP 메서드 매핑</a></li>
            <li><a href="/mapping-get-v2">HTTP 메서드 매핑 축약</a></li>
            <li><a href="/mapping/userA">경로 변수</a></li>
            <li><a href="/mapping/users/userA/orders/100">경로 변수 다중</a></li>
            <li><a href="/mapping-param?mode=debug">특정 파라미터 조건 매핑</a></li>
            <li><a href="/mapping-header">특정 헤더 조건 매핑(POST MAN 필요)</a></
            <li><a href="/mapping-consume">미디어 타입 조건 매핑 Content-Type(POST MAN 필요)</a></li>
            <li><a href="/mapping-produce">미디어 타입 조건 매핑 Accept(POST MAN 필요)</a></li>
        </ul>
    </li>
    <li>요청 매핑 - API 예시
        <ul>
            <li>POST MAN 필요</li>
        </ul>
    </li>
    <li>HTTP 요청 기본
        <ul>
            <li><a href="/headers">기본, 헤더 조회</a></li>
        </ul>
    </li>
    <li>HTTP 요청 파라미터
        <ul>
            <li><a href="/request-param-v1?username=hello&age=20">요청 파라미터v1</a></li>
            <li><a href="/request-param-v2?username=hello&age=20">요청 파라미터v2</a></li>
            <li><a href="/request-param-v3?username=hello&age=20">요청 파라미터v3</a></li>
            <li><a href="/request-param-v4?username=hello&age=20">요청 파라미터v4</a></li>
            <li><a href="/request-param-required?username=hello&age=20">요청 파라미터 필수</a></li>
            <li><a href="/request-param-default?username=hello&age=20">요청파라미터 기본 값</a></li>
            <li><a href="/request-param-map?username=hello&age=20">요청 파라미터 MAP</a></li>
            <li><a href="/model-attribute-v1?username=hello&age=20">요청 파라미터@ModelAttribute v1</a></li>
            <li><a href="/model-attribute-v2?username=hello&age=20">요청 파라미터@ModelAttribute v2</a></li>
        </ul>
    </li>
    <li>HTTP 요청 메시지
        <ul>
            <li>POST MAN</li>
        </ul>
    </li>
    <li>HTTP 응답 - 정적 리소스, 뷰 템플릿
        <ul>
            <li><a href="/basic/hello-form.html">정적 리소스</a></li>
            <li><a href="/response-view-v1">뷰 템플릿 v1</a></li>
            <li><a href="/response-view-v2">뷰 템플릿 v2</a></li>
        </ul>
    </li>
    <li>HTTP 응답 - HTTP API, 메시지 바디에 직접 입력
        <ul>
            <li><a href="/response-body-string-v1">HTTP API String v1</a></li>
            <li><a href="/response-body-string-v2">HTTP API String v2</a></li>
            <li><a href="/response-body-string-v3">HTTP API String v3</a></li>
            <li><a href="/response-body-json-v1">HTTP API Json v1</a></li>
            <li><a href="/response-body-json-v2">HTTP API Json v2</a></li>
        </ul>
    </li>
</ul>
</body>
</html>
```



## 로깅 간단히 알아보기

운영 시스템에서는 `System.out.println()` 같은 시스템 콘솔을 사용해서 필요한 정보를 출력하지 않고, 별도의 로깅 라이브러리를 사용해서 로그를 출력한다.

참고로 로그 관련 라이브러리도 많고, 깊게 들어가면 끝이 없기 때문에, 여기서는 최소한의 사용 방법만 알아본다.



### 로깅 라이브러리

스프링 부트 라이브러리를 사용하면 스프링 부트 로깅 라이브러리(`spring-boot-starter-logging`) 가 함께 포함된다. 스프링 부트 로깅 라이브러리는 기본으로 다음 로깅 라이브러리를 사용한다.

- SLF4
- Logback

로그 라이브러리는 Logback, Log4j, Log4J2 등등 수많은 라이브러리가 있는데, 그것을 통합해서 인터페이스로 제공하는 것이 바로 SLF4j 라이브러리다.

쉽게 이야기해서 SLF4j는 인터페이스고, 그 구현체로 Logback 같은 로그 라이브러리를 선택하면 된다. 실무에서는 스프링 부트가 기본으로 제공하는 Logback을 대부분 사용한다.



### 로그 선언 방법

- `private Logger log = LoggerFactory.getLogger(getClass())`
- `private static final Logger log = LoggerFactory.getLogger(xxx.class)`
- `@Slf4j` : 롬복 사용 가능



### 로그 호출

- `log.info("hello")`
- `System.out.println("hello")`

시스템 콘솔로 직접 출력하는 것보다 로그를 사용하면 다음과 같은 장점이 있다. 실무에서는 항상 로그를 사용해야 한다.



### LogTestController

``` java
package hello.springmvc.basic;

import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@Slf4j
@RestController
public class LogTestController {

    @RequestMapping("/log-test")
    public String logTest() {
        String name = "Spring";

        log.trace("trace log={}", name);
        log.debug("debug log={}", name);
        log.info(" info log={}", name);
        log.warn(" warn log={}", name);
        log.error("error log={}", name);

        // 로그를 사용하지 않아도 a+b 계산 로직이 먼저 실행됨, 이런 방식으로 사용하면 X
        log.debug("String concat log=" + name);
        return "ok";
    }
}
```

- 실행: http://localhost:8080/log-test
- 매핑 정보
  - `@RestController`
    - `@Controller` 는 반환 값이 String 이면 뷰 이름으로 인식된다. 그래서 뷰를 찾고 뷰가 랜더링 된다.
    - `@RestController` 는 반환 값으로 뷰를 찾는 것이 아니라, HTTP 메시지 바디에 바로 입력한다. 따라서 실행 결과로 ok 메세지를 받을 수 있다. @ResponseBody 와 관련이 있다. (뒤에서 설명)
- 테스트
  - 로그가 출력되는 포멧 확인
    - 시간, 로그레벨, 프로세스 ID, 쓰레드명, 클래스명, 로그메시지
  - 로그 레벨 설정을 변경해서 출력 결과를 보자.
    - LEVEL: `TRACE > DEBUG > INFO > WARN > ERROR`
    - 개발서버는 debug 출력
    - 운영 서버는 info 출력
  - `@Slf4j` 로 변경



### 로그 레벨 설정

``` properties
# application.properties
# 전체 로그 레벨 설정(기본 info)
logging.level.root=info

# hello.springmvc 패키지와 그 하위 로그 레벨 설정
logging.level.hello.springmvc=debug
```



### 올바른 로그 사용법

- `log.debug("data="+data)`
  - 로그 출력 레벨을 info 로 설정해도 해당 코드에 있는 "data="+data 가 실제 실행이 되어 버린다. 결과적으로 문자 더하기 연산이 발생한다.
- `log.debug("data={}", data)`
  - 로그 출력 레벨을 info로 설정하면 아무일도 발생하지 않는다. 따라서 앞과 같은 의미없는 연산이 발생하지 않는다.



### 로그 사용시 장점

- 쓰레드 정보, 클래스 이름 같은 부가 정보를 함께 볼 수 있고, 출력 모양을 조정할 수 있다.
- 로그 레벨에 따라 개발 서버에서는 모든 로그를 출력하고, 운영 서버에서는 출력하지 않는 등 로그를 상황에 맞게 조절할 수 있다.
- 시스템 아웃 콘솔에만 출력하는 것이 아니라, 파일이나 네트워크 등 로그를 별도의 위치에 남길 수 있다. 특히 파일로 남길 때는 일별, 특정 용량에 따라 로그를 분할하는 것도 가능하다.
- 성능도 일반 System.out 보다 좋다. (내부 버퍼링, 멀티 쓰레드 등등) 그래서 실무에서는 꼭 로그를 사용해야 한다.

