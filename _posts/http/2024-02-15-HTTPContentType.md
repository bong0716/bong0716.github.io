---
layout: post
title: "[HTTP] HTTP Header에 담긴 Content-Type이란?"
date: 2024-02-15 10:00:23 +0900
categories: "HTTP"
tag: HTTP
---  

아이티윌의 국비지원 [스프링부트 SNS 포토그램 프로젝트] 강의를 수강하며 정리한 내용입니다. 

<br>

## HTTP Header에 담긴 Content-Type 이해하기  
---
### Content-Type이란?
HTTP Body의 데이터 형식을 의미한다. 

<br>

### Content-Type 종류
- <span style="background-color:#9FEEC3"><strong> x-www-form-urlencoded </strong></span>
> 데이터 형식 : Key = Value
- <span style="background-color:#9FEEC3"><strong> text/plain </strong></span>
> 데이터 형식 : 안녕(평문)
- <span style="background-color:#9FEEC3"><strong> application/json </strong></span>   
> 데이터 형식 : {"username" : "홍길동"}


스프링부트는 기본적으로 x-www-form-urlencoded 타입을 파싱(분석)해준다. 

<br>

### Content-Type이 필요한 이유?   
Body Data를 전송할 때 Content-Type을 명시해야 받는 쪽에서 수월하게 처리할 수 있기 때문이다.   
> 다시 말해, 곡식을 나르는 A가 포대에 담긴 내용물이 무엇인지 적어두지 않고 B에게 건낸다면 일일히 열어봐야 하는 번거러움이 있기 때문에 포대에 미리 적어두고 나르는 것과 같다고 보면 된다.   

Content-Type은 POST와 PUT 요청 땐 필수사항이고, GET과 DELETE 요청 땐 Body Data가 없기 때문에 필요 없다.   

<br>

## 실습
--- 
**[x-www-form-urlencoded]**
```java
@RestController
public class HttpBodyController {
    
    private static final Logger log = LoggerFactory.getLogger(HttpBodyController.class);

    @PostMapping("/body1")
    public String xwwwformurlencoded(String username) { // 매개변수명이 전송된 데이터의 key 값과 일치해야 받을 수 있음.

        log.info(username);
        return "key=value 전송옴";
    }
}
```
매개변수명이 전송된 데이터의 key 값과 일치해야 받을 수 있다.

---
<br>

**[text/plain]**
```java
@RestController
public class HttpBodyController {

    private static final Logger log = LoggerFactory.getLogger(HttpBodyController.class);

    @PostMapping("/body2")
    public String textplain(@RequestBody String data) { // text/plain 형식의 데이터를 받으려면 @RequestBody 어노테이션을 붙여주어야함.

        log.info(data);
        return "text/plain 전송옴";
    }
}
```
text/plain 형식의 데이터를 받으려면 @RequestBody 어노테이션을 붙여주어야 한다.

---
<br>

**[application/json]**
```java
@RestController
public class HttpBodyController {

    private static final Logger log = LoggerFactory.getLogger(HttpBodyController.class);

    @PostMapping("/body3")
    public String applicationjson(@RequestBody String data) { // json형식의 경우 @RequestBody 어노테이션을 붙여주면 {"username" : "홍길동"}이 그대로 날라옴.  

        log.info(data);
        return "application/json 전송옴";
    }

    @PostMapping("/body4")
    public String applicationjsonToObject(@RequestBody User user) { // User 클래스를 만들어 Java 객체로 받아주면 data 값만 받아오기 가능.

        log.info(user.getUsername()); // username의 data 값만 받아옴.
        return "application/json 전송옴";
    }
}
``` 
Json형식의 경우 @RequestBody 어노테이션을 붙여주면 {"username" : "홍길동"}이 그대로 날라온다. MessageConverter가 자동으로 JavaObject를 Json으로 변경해서 통신을 통해 응답 해주는 것. @RestController 일때만 MessageConverter가 작동함.  
User 클래스를 만들어 Java 객체로 받아주면 Data 값만 받아오는게 가능하다.
