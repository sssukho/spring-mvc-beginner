# Section 2. 서블릿

### 프로젝트 생성

- packaging 을 War 로 해야 JSP가 돌아감



## Hello 서블릿

스프링 부트 환경에서 서블릿을 등록하고 사용해보자.

> 참고: 서블릿은 톰캣 같은 웹 애플리케이션 서버를 직접 설치하고, 그 위에 서블릿 코드를 클래스 파일로 빌드해서 올린 다음, 톰캣 서버를 실행하면 된다. 하지만 이 과정은 매우 번거롭다. 스프링 부트는 톰캣 서버를 내장하고 있으므로, 톰캣 서버 설치 없이 편리하게 서블릿 코드를 실행할 수 있다.



### 스프링 부트 서블릿 환경 구성

- `@ServletComponentScan`: 스프링부트는 서블릿을 직접 등록해서 사용할 수 있도록 `@ServletComponentScan` 을 지원한다.

``` java
package hello.servlet;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletComponentScan;

@ServletComponentScan // 서블릿 자동 등록
@SpringBootApplication
public class ServletApplication {

	public static void main(String[] args) {
		SpringApplication.run(ServletApplication.class, args);
	}
}
```



### 서블릿 등록하기

``` java
package hello.servlet.basic;

import java.io.IOException;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        System.out.println("HelloServlet.service");
        System.out.println("request = " + request);
        System.out.println("response = " + response);

        String username = request.getParameter("username");
        System.out.println("username = " + username);

        response.setContentType("text/plain"); // http header에 들어감
        response.setCharacterEncoding("utf-8"); // http header에 들어감
        response.getWriter().write("hello " + username); // http body에 들어감
    }
}
```

- `@WebServlet` 서블릿 애노테이션

  - name: 서블릿 이름
  - urlPatterns: URL 매핑

- HTTP 요청을 통해 매핑된 URL이 호출되면 서블릿 컨테이너는 다음 메서드를 실행한다.

  `protected void service(HttpServletRequest request, HttpServletResponse response)`

- 웹 브라우저 실행

  - `http://localhost:8080/hello?username=world`
  - 결과: hello world

- 콘솔 실행 결과

  ```
  HelloServlet.service
  request = org.apache.catalina.connector.RequestFacade@5e4e72
  response = org.apache.catalina.connector.ResponseFacade@37d112b6
  username = world
  ```

> [주의]
>
> Intellij 무료 버전을 사용하는데, 서버가 정상 실행되지 않는다면 프로젝트 생성 -> Intellij Gradle 대신에 자바 직접 실행에 있는 주의 사항을 읽어보자.



### HTTP 요청 메시지 로그로 확인하기

- application.properties 내 아래를 추가한다.

  - `logging.level.org.apache.coyote.http11=debug`

- 서버를 다시 시작하고, 요청해보면 서버가 받은 HTTP 요청 메시지를 출력하는 것을 확인할 수 있다.

  ```
  ...o.a.coyote.http11.Http11InputBuffer: Received [GET /hello?username=servlet
  HTTP/1.1
  Host: localhost:8080
  Connection: keep-alive
  Cache-Control: max-age=0
  sec-ch-ua: "Chromium";v="88", "Google Chrome";v="88", ";Not A Brand";v="99"
  sec-ch-ua-mobile: ?0
  Upgrade-Insecure-Requests: 1
  User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 11_2_1) AppleWebKit/537.36
  (KHTML, like Gecko) Chrome/88.0.4324.150 Safari/537.36
  Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/
  webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
  Sec-Fetch-Site: same-origin
  Sec-Fetch-Mode: navigate
  Sec-Fetch-User: ?1
  Sec-Fetch-Dest: document
  Referer: http://localhost:8080/basic.html
  Accept-Encoding: gzip, deflate, br
  Accept-Language: ko,en-US;q=0.9,en;q=0.8,ko-KR;q=0.7
  ]
  ```

> [참고]
>
> 운영 서버에 이렇게 모든 요청 정보를 다 남기면 성능 저하가 발생할 수 있다. 개발 단계에서만 적용할 것



### 서블릿 컨테이너 동작 방식

- 내장 톰캣 서버 생성

  ![2-1](./img/2-1.png)

- HTTP 요청, HTTP 응답 메시지

  ![2-2](./img/2-2.png)

- 웹 애플리케이션 서버의 요청 응답 구조

  ![2-3](./img/2-3.png)

> [참고]
>
> HTTP 응답에서 Content-Length는 웹 애플리케이션 서버가 자동은로 생성해준다.



### welcome 페이지 추가

지금부터 개발할 내용을 편리하게 참고할 수 있도록 welcome 페이지를 만들어둔다.

`webapp` 경로에 `index.html`을 두면 루트(http://localhost:8080) 호출시 index.html 페이지가 열린다.

- main/webapp/index.html

  ``` html
  <!DOCTYPE html>
  <html>
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  
  <body>
  <ul>
      <li><a href="basic.html">서블릿 basic</a></li>
  </ul>
  </body>
  </html>
  ```

- main/webapp/basic.html : 이번 장에서 학습할 내용은 다음 basic.html 이다.

  ``` html
  <!DOCTYPE html>
  <
  ```

  





