---
layout: post
title: "[Spring] 회원정보 수정 구현"
date: 2024-07-01 10:00:23 +0900
categories: "Spring"
tag: ["Spring", "Security"]
---  

아이티윌의 국비지원 [스프링부트 SNS 포토그램 프로젝트] 강의를 수강하며 정리한 내용입니다.

<br>

## 오늘의 실습
- 회원정보 수정 구현

<br>

## 🔎그 전에 알아야할 것
---
### 


<br>

---

## 실습

### **사용자 세션 넘기는 방법 2가지**
 
### 방법1. 인증된 사용자의 세션 정보를 Model에 담아서 넘겨주기
```java
@Controller
public class UserController {
	
	@GetMapping("/user/{id}/update")
	public String update(@PathVariable int id, @AuthenticationPrincipal PrincipalDetails principalDetails, Model model) {
		model.addAttribute("principal", principalDetails.getUser());
		return "user/update";
	}
}
```
```html
<h2>${principal.username}</h2>
```
update.jsp 파일에서 EL표현식으로 변수명을 적어주면 getter가 호출된다.    

<br>

`@PathVariable` : URL 경로에서 변수를 추출할 때 사용한다. 

<br>

---

### 방법2. spring-security-taglibs을 사용하기
```java
<dependency>
			<groupId>org.springframework.security</groupId>
			<artifactId>spring-security-taglibs</artifactId>
</dependency>
```
spring-security-taglibs를 의존성 주입 해준 후 

<br>

```html
<%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags"%>

<sec:authorize access="isAuthenticated()">
  <sec:authentication property="principal" var="principal" />
</sec:authorize>
```
모든 페이지에 포함되어 있는 파일에(필자는 layout폴더에 header.jsp파일) 태그 라이브러리를 넣어준다.   

<br>

```html
<h2>${principal.user.username}</h2>
```
model로 넘겨받지 않아도 principal 변수로 principalDetails에 접근이 가능해짐!

![image](https://github.com/bong0716/photogram/assets/119990564/f2eb566e-8780-4f04-8612-4c69cc9e877c)    
회원가입 때 입력했던 내용이 회원가입 수정 페이지에 뜨는 것을 확인!

---

### **회원가입 수정**

### **[update.jsp]**
```html
<form id="profileUpdate" onsubmit="update(${principal.user.id})">


<script src="/js/update.js"></script>
```
폼에서 수정버튼 클릭 시 update 함수가 호출되도록 해준다. 

### **[update.js]**
```javascript
function update(userId, event) {
	event.preventDefault(); // 폼태그 액션을 막기
	let data = $("#profileUpdate").serialize(); // key=value 형식의 데이터 받아올 때 
	console.log(data);
	
	$.ajax({
		type: "put",
		url: `/api/user/${userId}`,
		data: data,
		contentType: "application/x-www-form-urlencoded; charset=utf-8",
		dataType: "json"
	}).done(res=>{ // HttpStatus 상태코드 200번대
		console.log("성공", res);
		location.href=`/user/${userId}`;
	}).fail(error=>{ // HttpStatus 상태코드 200번대 아닐 때
		if(error.data == null) {
			alert(error.responseJSON.message);
		}else {
			alert(JSON.stringify(error.responseJSON.data));
		}
	})
}
```


### **[UserDataDto.java]**
```java
@Builder
@Data
public class UserUpdateDto {
	private String name;
	private String password;
	private String website;
	private String bio;
	private String phone;
	private String gender;

	public User toEntity() {
		return User.builder()
				.name(name)
				.password(password)
				.website(website)
				.bio(bio)
				.phone(phone)
				.gender(gender)
				.build();
	}
}
```
회원수정 때 사용할 DTO를 생성해준다. 

### **[UserService.java]**
```java
@RequiredArgsConstructor
@Service
public class UserService {

	private final UserRepository userRepository;
	private final BCryptPasswordEncoder bCryptPasswordEncoder;
	
	@Transactional
	public User 회원수정(int id, User user) {
		// 1. 영속화
		User userEntity = userRepository.findById(id).orElseThrow(() -> {return new CustomValidationApiException("찾을 수 없는 ID입니다.");});
		
		String rawPassword = user.getPassword();
		String encPassword = bCryptPasswordEncoder.encode(rawPassword);
		
		// 2. 영속화된 오브젝트 수정 - 더티체킹 (업데이트 완료)
		userEntity.setName(user.getName());
		userEntity.setPassword(encPassword);
		userEntity.setBio(user.getBio());
		userEntity.setWebsite(user.getWebsite());
		userEntity.setPhone(user.getPhone());
		userEntity.setGender(user.getGender());
		
		return userEntity;
	} // 더티체킹이 일어나서 업데이트가 완료됨. 
}

```
userEntity의 필드를 user 객체의 값으로 수정해주고 userEntity 객체를 반환한다. 해당 객체는 트랜잭션이 끝날 때 자동으로 데이터베이스에 업데이트 됨.  
<br>

`@Transactional`은 Spring Framework에서 제공하는 어노테이션으로, 메서드 또는 클래스에 트랜잭션 관리 기능을 적용할 수 있다. 트랜잭션은 데이터베이스 작업의 논리적 단위로, 여러 작업이 하나의 작업처럼 처리되어야 할 때 사용된다. 트랜잭션은 성공적으로 완료되면 모든 작업이 데이터베이스에 반영되고, 하나라도 실패하면 모든 작업이 롤백되어 일관성을 유지한다. 

<br>
**더티 체킹이란?** JPA의 기능으로, 영속성 컨텍스트에서 관리되는 엔티티 객체의 변경 사항을 자동으로 감지하여 트랜잭션이 끝날 때 데이터베이스에 반영하는 것.

### **[UserApiController.java]**
```java
@RequiredArgsConstructor
@RestController
public class UserApiController {

	private final UserService userService;
	
	@PutMapping("/api/user/{id}")
	public CMRespDto<?> update(@PathVariable int id, 
			@Valid UserUpdateDto userUpdateDto,
			BindingResult bindingResult,
			@AuthenticationPrincipal PrincipalDetails principalDetails) {
		
		if(bindingResult.hasErrors()) {
			Map<String, String> errorMap = new HashMap<>();
			
			for(FieldError error : bindingResult.getFieldErrors()) {
				errorMap.put(error.getField(), error.getDefaultMessage());
				System.out.println(error.getDefaultMessage());
			}
			throw new CustomValidationApiException("유효성 검사 실패함", errorMap);
		} else {
			User userEntity = userService.회원수정(id, userUpdateDto.toEntity());
			principalDetails.setUser(userEntity);
			return new CMRespDto<>(1, "회원수정 완료", userEntity);
		}
	}
}
```
인증된 사용자 정보를 업데이트된 사용자 정보로 갱신해주고 응답 객체를 생성하여 반환했다. CMRespDto는 커스텀 응답 DTO(Data Transfer Object)로, 상태 코드, 메시지, 데이터를 포함한다.

<br>

### **예외 처리 메소드**

### **[CustomValidationApiException.java]**
```java
public class CustomValidationApiException extends RuntimeException {

	// 객체를 구분할 때 씀(중요X)
	private static final long serialVersionUID = 1L;

	private Map<String, String> errorMap;
	
	public CustomValidationApiException(String message) {
		super(message);
	}

	public Map<String, String> getErrorMap() {
		return errorMap;
	}
}
```

### **[ControllerExceptionHandler.java]**
```java
@RestController
@ControllerAdvice // 모든 Exception을 낚아챔
public class ControllerExceptionHandler {
	
	@ExceptionHandler(CustomValidationApiException.class)
	public ResponseEntity<?> validationApiException(CustomValidationApiException e) { // 상태코드 전달을 위해 ResponseEntity 사용
		return new ResponseEntity<>(new CMRespDto<>(-1, e.getMessage(), e.getErrorMap()), HttpStatus.BAD_REQUEST);
	}
}
```
`@ControllerAdvice` : 모든 컨트롤러에서 발생하는 예외를 낚아채 처리해준다.

   <br>

CustomValidationApiException이 발생할 때 호출되며, HTTP 응답 본문과 상태 코드를 포함하는 응답을 생성 한다.

