---
layout: post
title: "[Spring] Model을 활용해 데이터 전달 및 출력하기"
date: 2024-03-03 10:00:23 +0900
categories: "Spring"
---  
아이티윌의 국비지원 [스프링부트 SNS 포토그램 프로젝트] 강의를 수강하며 정리한 내용입니다.

<br>

## 오늘의 실습
- Model을 사용하여 View 페이지로 데이터 값 넘기기
- Model로 넘어온 데이터 값을 View 페이지에서 출력하기 

<br>

## 🔎그 전에 알아야할 것
---
### Model?   
Model은 Spring Framework에서 사용되는 인터페이스로, 컨트롤러에서 뷰로 데이터를 전달하는 데 사용된다.      

### addAttribute()?
addAttribute 메서드는 Model 인터페이스에서 제공하는 메서드 중 하나로, 뷰로 전달할 데이터를 추가하는 데 사용된다. 이 메서드를 사용하여 key-value 쌍([HashMap](https://bong0716.github.io/java/2023/11/06/hash.html))을 추가할 수 있으며, 이후 뷰 페이지에서 해당 key를 사용하여 데이터를 추출할 수 있다.

<br>

## 실습
---

**[Model을 사용하여 View 페이지로 데이터 값 넘기기]** 
 ```java
@Controller 
public class JavaToJspController {

    @GetMapping("/jsp/java/model")
    public String jspToJavaToModel(Model model) { // Model 선언
		
        User user = new User();
        user.setUsername("ssar");
        
        model.addAttribute("username", user.getUsername()); // addAttribute 함수로 데이터 전달
            
        return "e";
    }
}
 ```
 컨트롤러에서는 Model을 파라미터로 받아 이를 활용해 뷰로 데이터를 전달하고, addAttribute 메서드를 사용하여 전달할 데이터를 설정한다. 이렇게 함으로써 뷰는 컨트롤러가 전달한 데이터를 활용하여 동적으로 페이지를 생성할 수 있다. 

---
<br>

**[Model로 넘어온 데이터 값을 View 페이지에서 출력하기]**   
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
<h1>This is jsp</h1>
<h3>${username}</h3>
</body>
</html>
``` 
데이터 값을 출력하기 위해선 ${} 안에 key 값을 적어주면 된다. 

---
<br><br><br>

<p align="center"><img src="https://github.com/bong0716/bong0716.github.io/assets/119990564/275a19e8-e116-4ab9-88cc-e0929079d4c7"></p>
<p align="center">
<제대로 출력되는지 확인!>
</p>


