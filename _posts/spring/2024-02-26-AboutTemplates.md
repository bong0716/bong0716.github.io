---
layout: post
title: "[Spring] 템플릿 엔진이란? Mustache, JSP 파일 응답하기"
date: 2024-02-26 10:00:23 +0900
categories: "Spring"
tag: Spring
---  
아이티윌의 국비지원 [스프링부트 SNS 포토그램 프로젝트] 강의를 수강하며 정리한 내용입니다.

<br>

## 오늘의 실습
- .txt 파일 응답하기 (기본경로는 resources/static)  
- 스프링부트가 공식 지원하는 .mustache 파일 응답하기  
- 스프링부트가 버린 .jsp 파일 응답하기   

<br>

## 🔎그 전에 알아야할 것
---
### 템플릿 엔진이란?
템플릿 엔진은 서버 측에서 동작하여 동적으로 HTML을 생성하는 데 사용된다. HTML 파일은 정적인 내용만을 담고 있어서 자체적으로는 동적 데이터를 처리할 수 없다.   

### 템플릿 파일 종류 
- **JSP(JavaServer Pages)** 파일은 Java 코드를 HTML에 포함하여 동적인 웹 페이지를 생성하는 데 사용된다. JSP 파일은 웹 애플리케이션 서버(예: 톰캣)에 의해 실행되며, 서버 측에서 Java 코드를 해석하고 실행하여 최종적으로 HTML 페이지를 생성한다. 이렇게 생성된 HTML 페이지가 클라이언트(브라우저)에게 전달되어 화면에 표시된다. 
- **Mustache**와 같은 다른 템플릿 엔진도 비슷한 원리로 동작한다. 템플릿 엔진은 템플릿 파일에 동적 데이터를 삽입할 수 있는 기능을 제공하며, 서버 측에서 해당 템플릿 파일을 해석하여 HTML을 생성한 후 클라이언트에 전달한다. 이렇게 함으로써 클라이언트는 동적으로 생성된 HTML을 받아 화면에 표시할 수 있게 된다.

### 템플릿 엔진 사용방법
템플릿 엔진을 사용하려면 [Maven Repository](https://mvnrepository.com/)에서 해당하는 템플릿 엔진 라이브러리 코드를 가져와 pom.xml에 넣어주어야 한다. 코드는 dependencies 태그 안 아무 곳이나 넣으면 된다. 버전은 Repository 숫자가 높은 걸 사용하면 되고, 라이브러리 추가 후엔 반드시 서버 재실행 하기!   
> mustache : Spring Boot Starter Mustache   
> jsp : Tomcat Jasper

<br>

## 실습
---

### 1) .txt 파일 응답하기
```java
@Controller // 파일을 리턴
public class HttpRespController {

    @GetMapping("/txt")
    public String txt() {
        return "a.txt"; //a.txt를 응답
    }
}
```

<br>

![1](https://github.com/bong0716/photogram/assets/119990564/58cb806d-2c88-4a4d-87bc-1737dfa49e9c)    
<p align="center">
Spring 프레임워크에서 정적 파일들은 resources/static 폴더 내부가 디폴트 경로이다.(약속)      
</p>
<p align="center">
*정적 파일? image, txt, mp3 등 
</p>
<br>
![2](https://github.com/bong0716/photogram/assets/119990564/6fbe9ce3-02ef-46fb-a19a-0eb30a0b2643) 


<p align="center"><br>
resources/static 내부에 a.txt 파일을 생성하여 내용을 입력한 후 실행하면
</p>


<br>

![3](https://github.com/bong0716/photogram/assets/119990564/e4af365d-cae3-4b94-b6e8-90f14dee8b23) 

<p align="center">
=> 파일 속 내용을 브라우저가 읽는 것을 확인할 수 있다. 
</p>

<br>

---

### 2) .mustache 파일 응답하기
```java
@Controller // 파일을 리턴
public class HttpRespController {

    @GetMapping("/mus")
    public String mus() {
        return "b.mustache";
    }
}
```

<br>



![4](https://github.com/bong0716/photogram/assets/119990564/4a8638b5-b865-4178-a799-a2f3a20f1eee)

<p align="center">
마찬가지로 static 폴더 아래에 b.html 파일 생성 후 우클릭 -> Refactor -> Rename 확장자를 .mustache로 변경 후 
</p>

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<h1>This is mustache</h1>
</body>
</html>
```
<p align="center">위 내용 입력</p>

<p align="center">
서버 실행 후 http://localhost:8080/mus를 브라우저에 띄우면?
</p>


![5](https://github.com/bong0716/photogram/assets/119990564/103f4b0a-ffcb-4eee-8a05-a7483cee1a2c)


<p align="center">
 b.mustache 파일을 다운로드 해버린다.
자바코드 해석이 안되고 바로 응답했기 때문이다.  
</p>

<br>

```java
@Controller // 파일을 리턴
public class HttpRespController {

    @GetMapping("/mus")
    public String mus() {
        return "b";
    }
}
```
그렇다면 해결 방법은? static 폴더에서 templates 폴더로 경로를 이동 시키고(없다면 생성) 컨트롤러의 return문에서 확장자를 지우면 된다. 확장자를 지워도 잘 찾아가는 이유는 spring에서 mustache를 지원하기 때문이다. 

<br>

![mus2](https://github.com/bong0716/photogram/assets/119990564/49805e9b-b465-4831-8774-ae4cf9be3a8c)

<p align="center">
다시 실행해보면 제대로 응답하는 것을 확인할 수 있다. <br>   
응답이 안된다면 템플릿 엔진 라이브러리를 넣었는지 확인할 것.
</p>

<br>

---

### 3) .jsp 파일 응답하기

<br>

![JSP1](https://github.com/bong0716/photogram/assets/119990564/34610a03-0c26-4a07-872e-1a47ae9b7a2e)

<p align="center">
jsp 사용시 디폴트 경로는 src/main/webapp이다. WEB-INF/views까지 직접 생성해 주기  
</p> 

<br>

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<h1>This is c.JSP</h1>
</body>
</html>
```
<p align="center">views 아래에 c.jsp 파일 생성 후 코드 입력.</p>

<br>

```java
@Controller // 파일을 리턴
public class HttpRespController {

    @GetMapping("/jsp")
    public String jsp() {
        return "c";
    }
}
```

<p align="center">
컨트롤러까지 생성 후 실행시키면 Whitelabel Error Page가 뜬다. return "c"; 만으로는 jsp 파일을 찾을 수 없기 떄문이다. 이는 <strong>스프링 부트가 jsp를 지원하지 않기 때문이므로 별도의 설정이 필요</strong>하다. 
</p>

<br><br>

![JSP2](https://github.com/bong0716/photogram/assets/119990564/10c177ee-8af8-4962-a73b-b6511fb8368c)

<p align="center">
application.properties로 되어 있는 걸 application.yml로 변경
</p>

<br><br>

![JSP3](https://github.com/bong0716/photogram/assets/119990564/a134e9de-ba8f-4424-acad-7aa6380f4ecf)

<p align="center">
application.yml에 입력하기. ViewResolver를 설정해주는 것!
</p>
<p align="center">   
jsp 사용시 디폴트 경로는 src/main/webapp이기 때문에 그 밑인 /WEB-INF/views를 prefix로 입력한 것이다.
</p>
<p align="center"> 
spring.mvc.view.prefix 입력 후 ctrl + space 누르면 알아서 줄 바꿈 밑 들여쓰기 됨. 
</p>

<br><br>

![JSP4](https://github.com/bong0716/photogram/assets/119990564/0b573a60-a7a3-4cad-be4c-61160a20f782)

<p align="center"> 
페이지가 제대로 뜬다면 설정 성공이다. 
</p>

<br>

---
## 결론
mustache는 Spring Boot에서 지원하므로 ViewResolver 설정이 자동으로 되어 있다.   
반면 JSP는 Spring Boot에서 지원하지 않으므로 ViewResolver 설정이 별도로 필요하다. 

<br> 