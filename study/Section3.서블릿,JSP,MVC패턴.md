# Section 3. 서블릿, JSP, MVC 패턴

## 회원 관리 웹 애플리케이션 요구사항

### 회원 정보

- 이름: username
- 나이: age



### 기능 요구사항

- 회원 저장
- 회원 목록 조회



### 회원 도메인 모델

``` java
package hello.servlet.domain;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class Member {
    
    private Long id;
    private String username;
    private int age;
    
    public Member() {
        
    }
    
    public Member(String username, int age) {
        this.username = username;
        this.age = age;
    }
}
```

- id 는 Member 를 회원 저장소에 저장하면 회원 저장소가 할당한다.



### 회원 저장소

``` java
package hello.servlet.domain.member;

import hello.servlet.domain.Member;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * 동시성 문제가 고려되어 있지 않음, 실무에서는 ConcurrentHashMap, AtomicLong 사용 고려
 */
public class MemberRepository {

    private static Map<Long, Member> store = new HashMap<>(); // static 사용
    private static long sequence = 0L; // static 사용

    private static final MemberRepository instance = new MemberRepository();

    public static MemberRepository getInstance() {
        return instance;
    }

    private MemberRepository() {};
    
    public Member save(Member member) {
        member.setId(++sequence);
        store.put(member.getId(), member);
        return member;
    }

    public Member findById(Long id) {
        return store.get(id);
    }
    
    public List<Member> findAll() {
        return new ArrayList<>(store.values());
    }
    
    public void clearStore() {
        store.clear();
    }
}
```

회원 저장소는 싱글턴 패턴을 적용했다. 스프링을 사용하면 스프링 빈으로 등록하면 되지만, 지금은 최대한 스프링 없이 순수 서블릿 만으로 구현하는 것이 목적이다. 싱글턴 패턴은 객체를 단 하나만 생성해서 공유해야 하므로 생성자를 `private` 접근자로 막아둔다.



### 회원 저장소 테스트 코드

``` java
package hello.servlet.domain.member;

import static org.assertj.core.api.Assertions.assertThat;

import hello.servlet.domain.Member;
import java.util.List;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;

class MemberRepositoryTest {
    MemberRepository memberRepository = MemberRepository.getInstance();

    @AfterEach
    void afterEach() {
        memberRepository.clearStore();
    }

    @Test
    void save() {
        // given
        Member member = new Member("hello", 20);
        // when
        Member savedMember = memberRepository.save(member);
        // then
        Member findMember = memberRepository.findById(savedMember.getId());
        assertThat(findMember).isEqualTo(savedMember);
    }

    @Test
    void findAll() {
        // given
        Member member1 = new Member("member1", 20);
        Member member2 = new Member("member2", 30);

        memberRepository.save(member1);
        memberRepository.save(member2);
        
        // when
        List<Member> result = memberRepository.findAll();
        
        // then
        assertThat(result.size()).isEqualTo(2);
        assertThat(result).contains(member1, member2);
    }
}
```

회원을 저장하고, 목록을 조회하는 테스트를 작성했다. 각 테스트가 끝날 때, 다음 테스트에 영향을 주지 않도록 각 테스트의 저장소를 `clearStore()` 를 호출해서 초기화했다.



## 서블릿으로 회원 관리 웹 애플리케이션 만들기

### MemberFormServlet - 회원 등록 폼

``` java
package hello.servlet.web;

import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet(name = "memberFormServlet", urlPatterns = "/servlet/members/new-form")
public class MemberFormServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html");
        resp.setCharacterEncoding("utf-8");
        PrintWriter w = resp.getWriter();
        w.write("<!DOCTYPE html>\n" +
                "<html>\n" +
                "<head>\n" +
                "    <meta charset=\"UTF-8\">\n" +
                "    <title>Title</title>\n" +
                "</head>\n" +
                "<body>\n" +
                "<form action=\"/servlet/members/save\" method=\"post\">\n" +
                "    username: <input type=\"text\" name=\"username\" />\n" +
                "    age:      <input type=\"text\" name=\"age\" />\n" +
                " <button type=\"submit\">전송</button>\n" + "</form>\n" +
                "</body>\n" +
                "</html>\n");
    }
}
```

`MemberFormServlet` 은 단순하게 회원 정보를 입력할 수 있는 HTML Form 을 만들어서 응답한다. 자바 코드로 HTML을 제공해야 하므로 쉽지 않은 작업이다.

- 실행: http://localhost:8080/servlet/members/new-form
- HTML Form 데이터를 POST로 전송해도, 전달받는 서블릿을 아직 만들지 않았다. 그래서 오류가 발생하는 것이 정상이다.



이번에는 HTML Form 에서 데이터를 입력하고 전송을 누르면 실제 회원 데이터가 저장되도록 해보자. 전송 방식은 POST HTML Form에서 학습한 내용과 같다.

### MemberSaveServlet - 회원 저장

``` java
package hello.servlet.web.servlet;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet(name = "memberSaveServlet", urlPatterns = "/servlet/members/save")
public class MemberSaveServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("MemberSaveServlet.service");
        String username = req.getParameter("username");
        int age = Integer.parseInt(req.getParameter("age"));

        Member member = new Member(username, age);
        System.out.println("member = " + member);
        memberRepository.save(member);

        resp.setContentType("text/html");
        resp.setCharacterEncoding("utf-8");

        PrintWriter w = resp.getWriter();
        w.write("<html>\n" +
                "<head>\n" +
                " <meta charset=\"UTF-8\">\n" + "</head>\n" +
                "<body>\n" +
                "성공\n" +
                "<ul>\n" +
                "    <li>id=" + member.getId() + "</li>\n" +
                "    <li>username=" + member.getUsername() + "</li>\n" +
                " <li>age=" + member.getAge() + "</li>\n" + "</ul>\n" +
                "<a href=\"/index.html\">메인</a>\n" + "</body>\n" +
                "</html>");
    }
}
```

`MemberSaveServlet` 은 다음 순서로 동작한다.

1. 파라미터를 조회해서 Member 객체를 만든다.
2. Member 객체를 MemberRepository를 통해서 저장한다.
3. Member 객체를 사용해서 결과 화면용 HTML 을 동적으로 만들어서 응답한다.



- 실행: http://localhost:8080/servlet/members/new-form
- 데이터가 전송되고, 저장 결과를 확인할 수 있다.



### MemberListServlet - 회원 목록

``` java
package hello.servlet.web.servlet;

import hello.servlet.domain.member.Member;
import hello.servlet.domain.member.MemberRepository;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.List;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet(name = "memberListServlet", urlPatterns = "/servlet/members")
public class MemberListServlet extends HttpServlet {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html");
        response.setCharacterEncoding("utf-8");

        List<Member> members = memberRepository.findAll();

        PrintWriter w = response.getWriter();
        w.write("<html>");
        w.write("<head>");
        w.write("    <meta charset=\"UTF-8\">");
        w.write("    <title>Title</title>");
        w.write("</head>");
        w.write("<body>");
        w.write("<a href=\"/index.html\">메인</a>");
        w.write("<table>");
        w.write("    <thead>");
        w.write("    <th>id</th>");
        w.write("    <th>username</th>");
        w.write("    <th>age</th>");
        w.write("    </thead>");
        w.write("    <tbody>");

				/*w.write("    <tr>");
        w.write("       <td>1</td>");
        w.write("       <td>userA</td>");
        w.write("       <td>10</td>");
        w.write("    </tr>");*/

        for (Member member : members) {
            w.write("    <tr>");
            w.write("        <td>" + member.getId() + "</td>");
            w.write("        <td>" + member.getUsername() + "</td>");
            w.write("        <td>" + member.getAge() + "</td>");
            w.write("    </tr>");
        }

        w.write("    </tbody>");
        w.write("</table>");
        w.write("</body>");
        w.write("</html>");
    }
}
```

1. memberRepostiory.findAll() 을 통해 모든 회원을 조회한다.
2. 회원 목록 HTML 을 for 루프를 통해서 회원 수 만큼 동적으로 생성하고 응답한다.



- 실행: http://localhost:8080/servlet/members
- 저장된 회원 목록을 확인할 수 있다.



### 템플릿 엔진으로

지금까지 서블릿과 자바 코드만으로 HTML을 만들어보았다. 서블릿 덕분에 동적으로 원하는 HTML을 마음껏 만들 수 있다. 정적인 HTML 문서라면 화면이 계속 달라지는 회원의 저장결과라던가, 회원 목록 같은 동적인 HTML 을 만드는 일은 불가능 할 것이다.

그런데, 코드에서 보듯이 이것은 매우 복잡하고 비효율적이다. 자바 코드로 HTML을 만들어 내는 것보다 차라리 HTML 문서에 동저긍로 변경해야 하는 부분만 자바 코드를 넣을 수 있다면 더 편리할 것이다. 이것이 바로 템플릿 엔진이 나온 이유이다. 템플릿 엔진을 사용하면 HTML 문서에서 필요한 곳만 코드를 적용해서 동적으로 변경할 수 있다.

템플릿 엔진에는 JSP, Thymeleaf, Freemarker, Velocity 등이 있다.

> [참고]
>
> JSP는 성능과 기능면에서 다른 템플릿 엔진과의 경쟁에서 밀리면서, 점점 사장되어 가는 추세다. 템플릿 엔진들은 각각 장단점이 있는데, 강의에서는 JSP는 앞부분에서 잠깐 다루고, 스프링과 잘 통합되는 Thymeleaf를 사용한다.





### Welcome 페이지 변경

다음에 연결될 강의들과 관련된 웰컴 페이지를 편리하게 참고할 수 있도록 변경한다.

- `main/webapp/index.html`

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
      <li>서블릿
          <ul>
              <li><a href="/servlet/members/new-form">회원가입</a></li>
              <li><a href="/servlet/members">회원목록</a></li>
          </ul>
      </li>
      <li>JSP
          <ul>
              <li><a href="/jsp/members/new-form.jsp">회원가입</a></li>
              <li><a href="/jsp/members.jsp">회원목록</a></li>
          </ul>
      </li>
      <li>서블릿 MVC
          <ul>
              <li><a href="/servlet-mvc/members/new-form">회원가입</a></li>
              <li><a href="/servlet-mvc/members">회원목록</a></li>
          </ul>
      </li>
      <li>FrontController - v1
          <ul>
              <li><a href="/front-controller/v1/members/new-form">회원가입</a></li>
              <li><a href="/front-controller/v1/members">회원목록</a></li>
          </ul>
      </li>
      <li>FrontController - v2
          <ul>
              <li><a href="/front-controller/v2/members/new-form">회원가입</a></li>
              <li><a href="/front-controller/v2/members">회원목록</a></li>
          </ul>
      </li>
      <li>FrontController - v3
          <ul>
              <li><a href="/front-controller/v3/members/new-form">회원가입</a></li>
              <li><a href="/front-controller/v3/members">회원목록</a></li>
          </ul>
      </li>
      <li>FrontController - v4
          <ul>
              <li><a href="/front-controller/v4/members/new-form">회원가입</a></li>
              <li><a href="/front-controller/v4/members">회원목록</a></li>
          </ul>
      </li>
      <li>FrontController - v5 - v3
      </li>
      <li>FrontController - v5 - v4
          <ul>
              <li><a href="/front-controller/v5/v4/members/new-form">회원가입</a></li>
              <li><a href="/front-controller/v5/v4/members">회원목록</a></li>
          </ul>
      </li>
      <li>SpringMVC - v1
          <ul>
              <li><a href="/springmvc/v1/members/new-form">회원가입</a></li>
              <li><a href="/springmvc/v1/members">회원목록</a></li>
          </ul>
      </li>
      <li>SpringMVC - v2
          <ul>
              <li><a href="/springmvc/v2/members/new-form">회원가입</a></li>
  
              <li><a href="/springmvc/v2/members">회원목록</a></li>
          </ul>
      </li>
      <li>SpringMVC - v3
          <ul>
              <li><a href="/springmvc/v3/members/new-form">회원가입</a></li>
              <li><a href="/springmvc/v3/members">회원목록</a></li>
          </ul>
      </li>
  </ul>
  </body>
  </html>
  ```

  

## JSP로 회원 관리 웹 애플리케이션 만들기

### JSP 라이브러리 추가

JSP를 사용하려면 먼저 다음 라이브러리를 추가해야 한다.

```
// JSP 추가 시작
implementation 'org.apache.tomcat.embed:tomcat-embed-jasper'
implementation 'javax.servlet:jstl'
// JSP 추가 끝
```



### 회원 등록 폼 JSP

`main/webapp/jsp/members/new-form.jsp`

``` jsp
<%@ page contentType="text/html; charset=UTF-8" language="java" %>
<html>
    <head>
        <title>Title</title>
    </head>
    <body>
        <form action="/jsp/members/save.jsp" method="post">
            username: <input type="text" name="username" />
            age:      <input type="text" name="age" />
            <button type="submit">전송</button>
        </form>
    </body>
</html>
```

- `<%@ page contentType="text/html; charset=UTF-8" language="java" %>` : JSP 문서라는 뜻이다. JSP 문서는 이렇게 시작해야 한다.

회원 등록 폼 JSP를 보면 첫 줄을 제외하고는 완전히 HTML과 똑같다. JSP는 서버 내부에서 서블릿으로 변환되는데, 우리가 만들었던 MemberFormServlet과 거의 비슷한 모습으로 변환된다.

- 실행: http://localhost:8080/jsp/members/new-form.jsp
  - 실행시 `.jsp` 까지 함꼐 적어주어야 한다.



### 회원 저장 JSP

`main/webapp/jsp/members/save.jsp`

``` jsp
<%@ page import="hello.servlet.domain.member.MemberRepository" %>
<%@ page import="hello.servlet.domain.member.Member" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<%
// request, response 사용 가능
	MemberRepository memberRepository = MemberRepository.getInstance();
	System.out.println("save.jsp");
	String username = request.getParameter("username");
	int age = Integer.parseInt(request.getParameter("age"));

	Member member = new Member(username, age);
	System.out.println("member = " +member);
	memberRepository.save(member);
%>
<html>
  <head>
    <meta charset="UTF-8">
  </head>
  <body>
    성공
    <ul>
      <li>id=<%=member.getId()%></li>
      <li>username=<%=member.getUsername()%></li>
      <li>age=<%=member.getrAge()%></li>
    </ul>
    <a href="/index.html">메인</a>
  </body>
</html>
```

JSP는 자바 코드를 그대로 다 사용할 수 있다.

- `<%@ page import="hello.servlet.domain.member.MemberRepository" %>`
  - 자바의 import 문과 같다.
- `<% ~~ %>`
  - 이 부분에는 자바 코드를 입력할 수 있다.
- `<%= ~~ %>`
  - 이 부분에는 자바 코드를 출력할 수 있다.

회원 저장 JSP를 보면, 회원 저장 서블릿 코드와 같다. 다른 점이 있다면, HTML을 중심으로 하고, 자바 코드를 부분부분 입력해주었다. `<% ~ %>` 를 사용해서 HTML 중간에 자바 코드를 출력하고 있다.



### 회원 목록 JSP

`main/webapp/jsp/members.jsp`

``` jsp
<%@ page import="java.util.List" %>
<%@ page import="hello.servlet.domain.member.MemberRepository" %>
<%@ page import="hello.servlet.domain.member.Member" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%
	MemberRepository memberrepository = MemberREpository.getInstance();
	List<Member> members = memberRepository.findAll();
%>
<html>
  <head>
    <meta charset="UTF-8">
    <title>Title</title>
  </head>
  
  <body>
    <a href="/index.html">메인</a>
    <table>
      <thead>
        <th>id</th>
        <th>username</th>
        <th>age</th>
      </thead>
      
      <tbody>
<%
      for (Member member : members) {
        out.write("		<tr>");
        out.write("			  <td>" + member.getId() + "</td>");
        out.write("				<td>" + member.getUsername() + "</td>");
        out.write("				<td>" + member.getAge() + "</td>");
        out.write("		</tr>");
      }
%>
      </tbody>
    </table>
  </body>
</html>
```

회원 리포지토리를 먼저 조회하고, 결과 List를 사용해서 중간에 `<tr><td>` HTML 태그를 반복해서 출력하고 있다.



### 서블릿과 JSP의 한계

서블릿으로 개발할 때는 뷰(View) 화면을 위한 HTML을 만드는 작업이 자바 코드에 섞여서 지저분하고 복잡했다.

JSP를 사용한 덕분에 뷰를 생성하는 HTML 작업을 깔끔하게 가져가고, 중간중간 동적으로 변경이 필요한 부분에만 자바 코드를 적용했다. 그런데 이렇게 해도 해결되지 않는 몇가지 고민이 남는다.

회원 저장 JSP를 보자. 코드의 상위 절반은 회원을 저장하기 위한 비즈니스 로직이고, 나머지 하위 절반만 결과를 HTML로 보여주기 위한 뷰 영역이다. 회원 목록의 경우에도 마찬가지다.

코드를 잘 보면, JAVA 코드, 데이터를 조회하는 리포지토리 등등 다양한 코드가 모두 JSP에 노출되어 있다. JSP가 너무 많은 역할을 한다. 이렇게 작은 프로젝트도 벌써 머리가 아파오는데, 수백 수천줄이 넘어가는 JSP를 떠올려보면 정말 지옥과 같을 것이다.



### MVC 패턴의 등장

비즈니스 로직은 서블릿처럼 다른곳에서 처리하고, JSP는 목적에 맞게 HTML로 화면(View)을 그리는 일에 집중하도록 하자. 과거 개발자들도 모두 비슷한 고민이 있었고, 그래서 MVC 패턴이 등장했다. 우리도 직접 MVC 패턴을 적용해서 프로젝트를 리팩토링 해보자.



