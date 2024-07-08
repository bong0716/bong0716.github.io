---
layout: post
title: "[Spring] 전/후처리 개념 및 유효성 검사 - AOP"
date: 2024-06-28 13:00:23 +0900
categories: "Spring"
tag: ["Spring", "Validation", "Photogram"]
---  

아이티윌의 국비지원 [스프링부트 SNS 포토그램 프로젝트] 강의를 수강하며 정리한 내용입니다.

<br>

## 오늘의 실습
- Validation을 이용한 유효성 검사(전처리)
- ExceptionHandler를 이용한 유효성 검사(후처리)

<br>

## 🔎그 전에 알아야할 것
---
### 유효성 검사란?   
데이터가 특정 기준이나 조건에 맞는지를 확인하는 과정이다.
- 전처리 : 클라이언트에서 서버로 데이터가 전송되기 전에 수행되는 작업
- 후처리 : 데이터베이스에 데이터를 저장하기 전에 수행되는 작업

### AOP(Aspect-Oriented Programming)란?
**관점지향프로그래밍**이란 뜻으로, 프로그램의 여러 부분에서 **공통적으로 사용되는 기능이나 로직을 분리하여 모듈화**하는 방법을 의미하는 SW개발 패러다임 중 하나다.    
- 핵심기능 : 처리해야 할 중요한 로직
- 공통기능 : 없어도 되지만 더 나은 핵심기능을 위해 추가로 넣는 로직   
ex) 회원가입(핵심기능)을 위한 전처리와 후처리(공통기능)를 서비스로직이 아닌 따로 처리하는 것. 

<br>

---

## 실습

먼저 mvnRepository에서 vaildation을 검색하고 **Spring Boot Starter Validation**을 추가해준다. (버전은 많이 사용한 것으로)   

<br>
(SpringBoot 2.3 이상부턴 dependency 해주어야 한다.)    
그럼 Controller에 `@Vaild`라는 어노테이션을 사용할 수 있게 된다!    

<br>

```java
@PostMapping("/auth/signup")
	public String signup(@Valid SignupDto signupDto, BindingResult bindingResult) {
		
		if(bindingResult.hasErrors()) {
			Map<String, String> errorMap = new HashMap<>();
			
			for(FieldError error : bindingResult.getFieldErrors()) {
				errorMap.put(error.getField(), error.getDefaultMessage());
			}
			throw new CustomValidationException("유효성 검사 실패함", errorMap);
		} else {
			User user = signupDto.toEntity();
			authService.회원가입(user);
			return "/auth/signin";
		}
	}
```
컨트롤러에 `@Valid` 및 `BindingResult` 추가   
`@Valid`에서 발생한 에러는 `bindingResult`의 `getFiledErrors`라는 컬렉션에 모아진다.    
모아진 에러를 errorMap에 담아 CustomValidationException가 낚아채도록 한다.  

+ 예외를 가로챌 ControllerExceptionHandler 클래스는 생성해주어야 한다. (아래에 계속)   
+ `throw new CustomValidationException("유효성 검사 실패함", errorMap);`에서 String과 Map을 같이 응답하기 위해 CMRespDto라는 응답 Dto를 생성해주어댜 한다.

---

```java
@Data
public class SignupDto {

	@Size(min=2, max=20) // 길이가 최소 2, 최대 20
	@NotBlank
	private String username;
	@NotBlank // 빈값 불가능
	private String password;
	@NotBlank
	private String email;
	@NotBlank
	private String name;
	
	public User toEntity() {
		return User.builder()
				.username(username)
				.password(password)
				.name(name)
				.email(email)
				.build();
	}
}
```
Dto에서 각 데이터에 맞는 검증 어노테이션 추가 

---

```java
@AllArgsConstructor
@NoArgsConstructor
@Data
@Entity
public class User {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private int id;
	
	@Column(length = 20, unique = true)
	private String username;
	
	@Column(nullable = false)
	private String password;
	
	@Column(nullable = false)
	private String name;
	
	private String website;
	private String bio;
	
	@Column(nullable = false)
	private String email;
	
	private String gender;
	private String phone;
	
	private String profileImageUrl;
	private String role;
	
	private LocalDateTime createDate;
	
	@PrePersist
	public void createDate() {
		this.createDate = LocalDateTime.now();
	}
}
```
도메인에서 각 데이터에 맞는 검증 어노테이션 추가   
(여기선 Valid가 아니라 javax.persistance에 있는 어노테이션임)

<br>

어노테이션 추가 후엔 ddl-auto를 create로 바꾼 후 실행해주기!    

---

### **[ControllerExceptionHandler.java]**
```java
@RestController
@ControllerAdvice // 모든 Exception을 낚아챔
public class ControllerExceptionHandler {

	@ExceptionHandler(CustomValidationException.class) // RuntimeException이 발생하는 모든 예외를 가로챔
	public CMRespDto<?> validationException(CustomValidationException e) {
		return new CMRespDto<Map<String, String>>(-1, e.getMessage(), e.getErrorMap());
	}
}
```
handler 패키지를 생성 후 ControllerExceptionHandler를 생성해준다.    

<br>

### **[CustomValidationException.java]**
```java
public class CustomValidationException extends RuntimeException {

	// 객체를 구분할 때 씀(중요X)
	private static final long serialVersionUID = 1L;

	private String message;
	private Map<String, String> errorMap;
	
	public CustomValidationException(String message, Map<String, String> errorMap) {
		this.message = message;
		this.errorMap = errorMap;
	}

	public Map<String, String> getErrorMap() {
		return errorMap;
	}
}

```
handler 패키지 안에 ex라는 패키지를 생성하고 CustomValidationException 클래스를 생성해준다. 

<br>

### **[CMRespDto.java]**
```java
@AllArgsConstructor
@NoArgsConstructor
@Data
public class CMRespDto<T> {
	private int code; // 1(성공), -1(실패) 
	private String message;
	private T data;
}
```
CMRespDto는 전역적으로 쓰이기 때문에 타입을 T로 설정해 ErrorMap, User 오브젝트, String 등을 받을 수 있도록 한다. 

<br>

![image](https://github.com/bong0716/photogram/assets/119990564/ac34313f-de6d-4a6d-be2a-6496ee0e4adc) 

포스트맨으로 요청했을 때 에러메시지가 잘 뜨는 것을 확인! 

---

<br>

### **[Script.java]**
```java
public class Script {
	
	public static String back(String msg) {
		StringBuffer sb = new StringBuffer();
		sb.append("<script>");
		sb.append("alert('"+msg+"');");
		sb.append("history.back();");
		sb.append("/<script>");
		
		return sb.toString();
	}
}
```
사용자에게 에러사항을 스크립트로 보여주기 위해 util 패키지에 Script 클래스 추가 

<br>

### **[ControllerExceptionHandler.java 코드 수정]**
```java
@RestController
@ControllerAdvice // 모든 Exception을 낚아챔
public class ControllerExceptionHandler {

	@ExceptionHandler(CustomValidationException.class) // RuntimeException이 발생하는 모든 예외를 가로챔
	public String validationException(CustomValidationException e) {
		return Script.back(e.getErrorMap().toString());
	}
}

```
클라이언트에게 응답할 땐 스크립트가 좋음.    
Ajax 통신이나 Android 통신 때에는 CMRespDto를 씀. 
<br>

![image](https://github.com/bong0716/photogram/assets/119990564/795416a9-8d31-4900-b90f-5faaffe6e63b)   
<br>
스크립트 뜨는 것 확인! 


