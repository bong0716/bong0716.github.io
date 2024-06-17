---
layout: post
title: "[Spring] Controller 쿼리스트링과 주소변수매핑"
date: 2024-02-14 10:00:23 +0900
categories: "Spring"
tag: Spring
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

![쿼리스트링1](https://github.com/bong0716/photogram/assets/119990564/d8e602d0-a31f-4d35-a122-a2a735c2f4c1)    
<div style="text-align: center">(쿼리스트링에 값이 없을 때)</div>
<br>

---

![쿼리스트링2](https://github.com/bong0716/photogram/assets/119990564/2f15dee8-9688-42f0-b54e-4f5c5ad67d6d)    
<div style="text-align: center">(쿼리스트링에 값을 양념으로 줬을 때)</div>
<br>

---

![주소변수매핑](https://github.com/bong0716/photogram/assets/119990564/95dc8e9d-d3a7-4e56-8678-258f70bea099)    
<div style="text-align: center">(주소에 값을 포함했을 때) </div>

---

<br>

SpringBoot에서는 거의 주소 변수매핑 방식을 사용한다고 한다!

<br>