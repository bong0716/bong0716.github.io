---
layout: post
title: "[Springboot] Controller(컨트롤러)란? | 동작 이해와 사용 방법"
date: 2024-02-13 10:00:23 +0900
categories: "Springboot"
tag: ["Springboot", "Photogram"]
---  
아이티윌의 국비지원 [스프링부트 SNS 포토그램 프로젝트] 강의를 수강하며 정리한 내용입니다.

<br>

---
## 1. Controller란?   
사용자의 요청을 처리한 후 정해진 뷰에 모델객체를 넘겨주는 역할을 한다. 

<br>

### **- 요청을 할 때마다 Java 파일이 호출된다.**   
로그인을 요청하려면 login.java 파일을, 회원가입을 요청하려면 join.java 파일을 호출해야 한다. (당연히 파일 이름은 자유)

<br>

### **- 요청의 종류가 3개이면 3개의 Java 파일이 필요하다.**  
![1](https://github.com/bong0716/photogram/assets/119990564/6073ba9e-23e0-4cf6-933f-400aadfdc0ef)

위 그림처럼 요청마다 그에 해당하는 파일을 불러오는 건 비효율적이다. 

---
<br>

### **- 때문에 하나의 Java 파일에서 모든 요청을 받는 FrontController를 사용한다.**  

![2](https://github.com/bong0716/photogram/assets/119990564/6a1fa24d-0fbf-45b6-b629-aeec79f12ec0)    
FrontController는 컨트롤러의 공통영역을 처리해주고 컨트롤러를 순수한 자바로 만들어준다. 

---
<br>

### **- 이렇게 FrontController 패턴을 적용하면 한 곳에서 모든 사용자의 요청을 컨트롤할 수 있다.**
![3](https://github.com/bong0716/photogram/assets/119990564/a0391964-ef7f-4b03-91f6-8786130f9305)
위 그림에서 각 컨트롤러가 프론트컨트롤러로 동작한다. 

### **- 프론트컨트롤러가 여러개인 이유는 너무 많은 요청이 한곳으로 모이는 것을 방지하기 위해 도메인 별로 분기한 것이다.**  

![4](https://github.com/bong0716/photogram/assets/119990564/509702a4-67e3-4553-a613-a9fa5949f758)  

도메인?  
-> 범주를 가지는 것 ex) 남자와 여자  

로그인, 회원가입 요청이 들어왔을 땐 UserController로, 글 쓰기와 글 삭제 요청이 들어왔을 땐 BoardController로, 상품등록과 목록보기 요청이 들어왔을 땐 ProductController로 요청을 처리한다. 

---
<br>

### **- 여기서 분기의 일은 DispatcherServlet이 해준다.**  
![5](https://github.com/bong0716/photogram/assets/119990564/b0ba62c3-21fc-4e22-bdff-03f14d3a3fe5)

Spring 프레임워크엔 이 Dispatcher-servlet이 만들어져 있다. 

<br>

---
## 2. Spring에서의 Controller 사용 방법
### Spring에서 Controller는 두 종류다.  
- **@Controller**
- **@RestController**   

<br>
먼저 **[@Controller]** 사용법과 용도에 대해 알아보자. 


``` java
@Controller // 컨트롤러 클래스 바로 위에 @Controller 어노테이션을 붙여준다.
public class BoardController {
	
	@GetMapping("/board/write") // http://localhost:8080/[프로젝트명]/board/write 경로가 요청되면
	public String get() {
		return "/board/wrtie"; // http://localhost:8080/[프로젝트명]/board/write가 응답된다. 프로젝트에서 board 폴더 아래 write라는 파일이 존재해야 한다.
	}
	
	@PostMapping("/board/write")
	public String post() {
		// DB에 게시물 정보를 넣는 서비스 로직 필요
		return "/board/list";
	}
}
```

`@Controller` 어노테이션이 붙은 클래스는 File을 응답할 때 사용하는 컨트롤러로, 클라이언트가 브라우저이면 .html파일을 응답한다. 즉, return문에 반환할 View를 적어주면 된다.  
<br>
위 예시에서 **get() 메소드**는 사용자가 게시물 등록 페이지를 요청했을 때, 해당 페이지를 응답하고 있고, **post() 메소드**는 사용자가 게시물 등록 페이지에서 게시물 등록 절차를 마치고 등록 버튼을 눌렀을 때, DB에 게시물 정보 등을 넣는 작업을 마치고 게시물 정보가 모여있는 list 페이지로 응답하고 있다. 
<br>  
src/main/board/write.jsp와 같이 전체 경로와 확장자를 붙이지 않아도 잘 응답되는 이유는 ViewResolver라는 아이가 있어서인데, 이 부분은 따로 포스팅하도록 하겠다.

<br> 
<hr>
<br>

다음은 **[@RestController]**의 사용법과 용도에 대해 알아보자.
``` java
@RestController // 컨트롤러 클래스 바로 위에 @RestController 어노테이션을 붙여준다.
public class HttpController {
	
	@GetMapping("/get") // http://localhost:8080/[프로젝트명]/get 경로가 요청되면
	public String get() {
		return "get 요청됨"; // get 요청됨 이라는 글자가 페이지에 띄워진다. 
	}
	
	@PostMapping("/post")
	public String post() {
		return "post요청됨";
	}

        @PutMapping("/put")
        public String put() {
            return "put요청됨";
        }

        @DeleteMapping("/delete")
        public String delete() {
            return "delete요청됨";
        }
}
```

`@RestController` 어노테이션이 붙은 클래스는 Data를 응답할 때 사용하는 컨트롤러로 JSON 형태의 객체 데이터 반환이 가능하다. `(@Controller에 @ResponseBody가 추가된 것이다.)` 데이터를 응답으로 제공하는 REST API를 개발할 때 주로 사용된다. 

<br>

