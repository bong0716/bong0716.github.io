---
layout: post
title: "[Spring] UserDetailsService란?"
date: 2024-06-28 14:00:23 +0900
categories: "Spring"
tag: ["Spring", "Photogram"]
---  

아이티윌의 국비지원 [스프링부트 SNS 포토그램 프로젝트] 강의를 수강하며 정리한 내용입니다.

<br>

## 오늘의 실습
- UserDetailsService를 이용한 로그인

<br>

## 🔎그 전에 알아야할 것
---
### UserDetails란?
사용자의 정보를 담는 역할을 수행하는 Spring Security의 인터페이스이다. 사용자 인증 및 권한 부여를 위해 필요한 정보를 제공한다. 

### UserDetailsService란?   
사용자의 정보를 제공하는 역할을 수행하는 Spring Security의 인터페이스이다. 주로 사용자 인증 시 사용자의 정보를 조회하기 위해 사용한다.      
기본 오버라이드 메소드로 `loadUserByUsername(String username)`이 있다. 해당 메소드는 주어진 사용자 이름(username)을 통해 사용자의 정보를 불러과 `UserDetails`객체로 반환한다. 

### 사용자의 세션 값을 얻어오는 과정
Spring Security에서 인증된 사용자의 정보를 얻기 위해선 세션(Session)에 저장된 값을 통해 접근할 수 있다. 아래에서 실습 코드를 통해 부연 셜명하겠다.

<br>

---

## 실습

### **[SecurityConfig.java]**
```java
@Configuration
public class SecurityConfig {

	@Bean
	BCryptPasswordEncoder encode() {
		return new BCryptPasswordEncoder();
	}
	
	@Bean
	SecurityFilterChain configure(HttpSecurity http) throws Exception {
	    http
	        .csrf(csrf -> csrf.disable())
	        .authorizeRequests(authorizeRequests ->
	            authorizeRequests
	                .antMatchers("/", "/user/**", "/image/**", "/subscripbe/**", "/comment/**").authenticated()
	                .anyRequest().permitAll()
	        )
	        .formLogin(formLogin ->
	            formLogin
	                .loginPage("/auth/signin") // GET
	                .loginProcessingUrl("/auth/signin") // POST -> 스프링시큐리티가 로그인 프로세스 진행
	                .defaultSuccessUrl("/")
	        );

	    return http.build();
	}
}

```
 `.loginProcessingUrl("/auth/signin")` 추가   
 => 로그인 요청 시 UserDetailsService가 낚아채게 됨. 

 <br>

### **[PrincipalDetailsService.java]**
```java
@RequiredArgsConstructor
@Service
public class PrincipalDetailsService implements UserDetailsService{

	private final UserRepository userRepository;
	
	// 패스워드는 알아서 체킹하므로 신경쓸 필요 없다. 
	// 리턴이 잘되면 자동으로 UserDetails 타입을 세션으로 만든다.
	
	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		
		User userEntity = userRepository.findByUsername(username);
		
		if(userEntity == null) {
			return null;
		} else {
			return new PrincipalDetails(userEntity); // PrincipalDetails가 리턴되면서 세션에 저장될 때 userEntity를 활용할 수 있게 됨
		}
	}

}
```
UserDetailsService 인터페이스를 구현하는 PrincipalDetailsService 생성.   
=> IoC에는 기존에 있던 UserDetailsService가 아닌 PrincipalDetailsService만 남게 됨.   

<br>

username만 받는 이유 : password는 알아서 처리해줌. 

<br>

### **[PrincipalDetails.java]**
```java
@Data
public class PrincipalDetails implements UserDetails{

	private static final long serialVersionUID = 1L;
	
	private User user;
	
	public PrincipalDetails(User user) {
		this.user = user;
	}

	@Override
	public Collection<? extends GrantedAuthority> getAuthorities() {
		
		Collection<GrantedAuthority> collector = new ArrayList<>();
		collector.add(() -> { return user.getRole(); });
		return collector;
	}

	@Override
	public String getPassword() {
		return user.getPassword();
	}

	@Override
	public String getUsername() {
		return user.getUsername();
	}

	@Override
	public boolean isAccountNonExpired() {
		return true;
	}

	@Override
	public boolean isAccountNonLocked() {
		return true;
	}

	@Override
	public boolean isCredentialsNonExpired() {
		return true;
	}

	@Override
	public boolean isEnabled() {
		return true;
	}

}
```
UserDetails 인터페이스를 구현하는 PrincipalDetails 생성.

| 메소드                       | 설명                                                                                                     |
|------------------------------|----------------------------------------------------------------------------------------------------------|
| `getAuthorities()`           | 사용자의 권한을 반환 |
| `getPassword()`              | 사용자의 비밀번호를 반환                  |
| `getUsername()`              | 사용자의 사용자 이름을 반환               |
| `isAccountNonExpired()`      | 계정의 만료 여부를 반환. (true : 계정이 만료되지 않았음)                                    |
| `isAccountNonLocked()`       | 계정의 잠금 여부를 반환. (true : 계정이 잠금되지 않았음)                                |
| `isCredentialsNonExpired()`  | 자격 증명(비밀번호)의 만료 여부를 반환. (true : 자격 증명이 만료되지 않았음)                   |
| `isEnabled()`                | 계정의 활성화 여부를 반환. (true : 계정이 활성화되어 있음)                               |

<br>

---

## 세션 정보 받아오는 방법 2가지

### **[1. @AuthenticationPrinciapl 어노테이션 활용(추천)]**
```java
@GetMapping("/user/{id}/update")
	public String update(@PathVariable int id, @AuthenticationPrincipal PrincipalDetails principalDetails) {
		System.out.println("세션 정보 " + principalDetails.getUser());
		
		return "user/update";
}
```
principalDetails 객체에 바로 접근 가능하다.

<br>

### **[2. 직접 접근(추천 X)]**
```java
@GetMapping("/user/{id}/update")
	public String update(@PathVariable int id) {
		Authentication auth = SecurityContextHolder.getContext().getAuthentication();
		PrincipalDetails principalDetails = (PrincipalDetails) auth.getPrincipal();
		System.out.println("세션 정보 " + principalDetails.getUser());
		
		return "user/update";
}
```
**`SecurityContextHolder`** : Spring Security는 인증된 사용자의 정보를 SecurityContextHolder에 저장한다.   
**`SecurityContext`** : SecurityContextHolder에서 SecurityContext를 가져온다. 이 객체는 현재 인증된 사용자와 관련된 모든 보안 정보를 담고 있다.    
**`Authentication`**: SecurityContext에서 Authentication 객체를 가져온다.    
**`PrincipalDetails`**: Authentication 객체에서 Principal을 가져오는데, 이는 보통 UserDetails의 구현체이다. 이를 통해 사용자 정보를 얻을 수 있다.

