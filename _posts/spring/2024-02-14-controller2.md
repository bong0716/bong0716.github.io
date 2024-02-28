---
layout: post
title: "[Spring] Controller 쿼리스트링과 주소변수매핑"
date: 2024-02-14 10:00:23 +0900
categories: "Spring"
---  
아이티윌의 국비지원 [스프링부트 SNS 포토그램 프로젝트] 강의를 수강하며 정리한 내용입니다.

<br>

### 쿼리스트링과 주소변수매핑 실습
---
```java
@RestController
public class QueryPathController{

    @GetMapping("/chicken")
    public String chickenQuery(String type) {
        return type + "배달갑니다. (쿼리스트링)";
    }

    @GetMapping("/chicken/{type}")
    public String chickenPath(@PathVariable String type) {
        return type + "배달갑니다. (주소 변수매핑)";
    }

}
```
<br>

![쿼리스트링1](https://github.com/hyejinyoon20010716/hyejinyoon20010716.github.io/assets/119990564/fbe1b13b-b3b3-4055-a215-6e3a4af755ef) 

<br>

![쿼리스트링2](https://github.com/hyejinyoon20010716/hyejinyoon20010716.github.io/assets/119990564/9de65c4d-8551-474f-b827-f2dc127d1c73)

<br>

![주소변수매핑](https://github.com/hyejinyoon20010716/hyejinyoon20010716.github.io/assets/119990564/30498fb1-b2a4-4bb3-9e51-7c089674c500)


SpringBoot에서는 거의 주소 변수매핑 방식을 사용한다고 한다!

<br>