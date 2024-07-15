---
layout: post
title: "[Springboot] 포토그램 회원가입 기능 구현하기(2) - 비밀번호 암호화"
date: 2024-03-13 10:00:23 +0900
categories: "Springboot"
tag: ["Springboot", "Photogram"]
---  
아이티윌의 국비지원 [스프링부트 SNS 포토그램 프로젝트] 강의를 수강하며 정리한 내용입니다.

<br>

## 오늘의 실습
- 회원가입 시 비밀번호 암호화 하기

<br>

## 🔎그 전에 알아야할 것
---
### BCryptPasswordEncoder이란?
Spring Security에서 제공하는 비밀번호 인코딩 클래스이다. 사용자의 비밀번호를 안전하게 해시화하여 저장하고, 사용자가 로그인할 때 입력한 비밀번호를 저장된 해시화된 비밀번호와 비교하여 인증하는 데 사용된다. 

<br>

## 실습
---
**[SecurityConfig.java]**
```java
@RequiredArgsConstructor
@Configuration //IoC
public class SecurityConfig {
	
	@Bean // 추가
	BCryptPasswordEncoder encoder() {
		return new BCryptPasswordEncoder();
	}
	
	@Bean
	SecurityFilterChain configure(HttpSecurity http) throws Exception {
			
		http.csrf().disable();
		http.authorizeRequests()
					... 생략
		return http.build();
	}
}

```
BCryptPasswordEncoder를 빈으로 등록해준다. 그럼 다른 곳에서 이 빈을 주입받아 사용할 수 있다. 

---
<br>

**[AuthService.java]**
```java
package com.cos.photogramstart.service;

@RequiredArgsConstructor
@Service // 1. IoC 2. 트랜잭션 관리
public class AuthService {
    
    private final UserRepository userRepository;
    private final BCryptPasswordEncoder bCryptPasswordEncoder; // 추가

    @Transactional
    public User 회원가입(User user) {
        // 회원가입 진행
        String rawPassword = user.getPassword(); // 추가
        String encPassword = bCryptPasswordEncoder.encode(rawPassword); // 추가
        user.setPassword(encPassword); // 추가
        user.setRole("ROLE_USER"); // 추가
        User userEntity = userRepository.save(user);
        return userEntity;
    }
}
```

<span style="background-color:#FFC9AF"><strong> private final BCryptPasswordEncoder bCryptPasswordEncoder; </strong></span>   
> BCryptPasswordEncoder는 Spring Security에서 제공하는 비밀번호 인코딩 기능을  사용하기 위한 빈.  

<span style="background-color:#FFC9AF"><strong> String rawPassword = user.getPassword(); </strong></span>   
> 사용자 객체에서 비밀번호를 가져와서 rawPassword 변수에 저장.  

<span style="background-color:#FFC9AF"><strong> String encPassword = bCryptPasswordEncoder.encode(rawPassword) </strong></span>   
> BCryptPasswordEncoder를 사용하여 rawPassword를 해시화하여 암호화된 비밀번호 생성.

<span style="background-color:#FFC9AF"><strong> user.setPassword(encPassword); </strong></span>   
> 사용자 객체에 암호화된 비밀번호 설정. 

<span style="background-color:#FFC9AF"><strong> user.setRole("ROLE_USER"); </strong></span> 
> 사용자의 역할을 "ROLE_USER"로 설정. 

---

![123](https://github.com/bong0716/photogram/assets/119990564/73a7d91b-b692-4d8f-83f3-1b1b8218d52d)

<br>
DB에 비밀번호가 암호화 되어 있는 것을 확인!   
같은 이름으로 회원가입을 했을 때 오류 페이지가 뜨는 것을 확인했다면 성공!


