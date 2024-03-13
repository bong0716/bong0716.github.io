---
layout: post
title: "[Spring] Spring Boot 3.x 버전 SecurityConfig 설정하기"
date: 2024-03-06 10:00:23 +0900
categories: "Spring"
---  
아이티윌의 국비지원 [스프링부트 SNS 포토그램 프로젝트] 강의를 수강하며 정리한 내용입니다.

<br>

## 오늘의 실습
- SecurityConfig 설정하기

<br>

## 그 전에 알아야할 것
---
### SecurityConfig를 설정해주는 이유?
사용자가 시스템에 로그인할 때 인증되어야 하고, 인증된 사용자에게는 특정한 권한이나 역할을 부여하여 특정 리소스에 접근할 수 있도록 해야 한다. SecurityConfig는 이러한 인증 및 인가 규칙을 정의하여 시스템의 보안을 관리하는데 유용하다.

<br>

### Redirect란? 
설명은 [여기서](https://bong0716.github.io/spring/2024/03/06/redirect.html) 참고

<br>

## 실습
---
**[Spring Boot 3.x 버전 SecurityConfig 코드]**
```java
package com.cos.photogramstart.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.web.SecurityFilterChain;

import com.cos.photogramstart.config.oauth.OAuth2DetailsService;

import lombok.RequiredArgsConstructor;

@RequiredArgsConstructor
@Configuration //IoC
public class SecurityConfig {
	
	private final OAuth2DetailsService oAuth2DetailsService;
	
	@Bean
	BCryptPasswordEncoder encoder() {
		return new BCryptPasswordEncoder();
	}
	
	@Bean
	SecurityFilterChain configure(HttpSecurity http) throws Exception {
			
		http.csrf().disable();
		http.authorizeRequests()
					.antMatchers("/", "/user/**", "/image/**", "/subscribe/**", "/comment/**", "/api/**").authenticated()
					.and()
					.formLogin()
					.loginPage("/auth/signin") // GET
					.loginProcessingUrl("/auth/signin") // POST -> 스프링시큐리티가 로그인 프로세스 진행
					.defaultSuccessUrl("/")
					.and()
					.oauth2Login() // form로그인도 하는데, oauth2 로그인도 할 거다. 
					.userInfoEndpoint() // oauth2 로그인을 하면 최종응답에 회원정보를 바로 받을 수 있다.
					.userService(oAuth2DetailsService);
		return http.build();
	}
}

```

1. > `http.csrf().disable();`   
CSRF(Cross-Site Request Forgery) 공격 방지 기능을 비활성화( RESTful API를 사용할 때는 CSRF 토큰을 사용하지 않는 것이 일반적임)

2. > `.antMatchers("/", "/user/**", "/image/**", "/subscribe/**", "/comment/**", "/api/**").authenticated()`   
해당 주소에 대해선 인증이 필요 

3. > `.and()`  
  `.formLogin()`  
  `.loginPage("/auth/signin")`  
  인증이 필요한 페이지로 요청이 왔을 때, 리다이렉트 되는 로그인 페이지 경로를 "/auth/signin"으로 설정   

4. > `.loginProcessingUrl("/auth/signin")`  
  실제로 로그인을 처리하는 URL을 설정   


5. > `.defaultSuccessUrl("/")`  
  로그인이 정상적으로 수행됐으면 "/"로 이동   

6. > `.oauth2Login()`  
  OAuth 2.0 프로바이더를 사용하여 로그인을 처리   

7. > `.userInfoEndpoint()`  
  OAuth 2.0 로그인을 통해 사용자 정보를 얻을 때 사용되는 엔드포인트를 설정   

8. > `.userService(oAuth2DetailsService)`  
  OAuth 2.0 로그인 후에 사용자 정보를 처리하는데 사용되는 사용자 서비스를 지정   

<br>

## 이전 버전과의 차이점
---
**[Spring Boot 2.x 버전 SecurityConfig 코드]**
```java
@EnableWebSecurity // 해당 파일로 시큐리티를 활성화
@Configuration // Ioc
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // super 삭제 - 기존 시큐리티가 가지고 있는 기능이 다 비활성화
        http.authorizeRequests()
                                .antMatchers("/", "/user/**", "/image/**", "/subscribe/**", "/comment/**").authenticated()
                                .anyRequest().permitAll()
                                .and()
                                .formLogin("/auth/signin")
                                .loginPage("/auth/signin")
                                .defaultSuccessUrl("/");
    }
}

```
- ### @EnableGlobalMethodSecurity(prePostEnabled = true)가 deprecated 됐다.    
3.x 에서는 @EnableMethodScurity를 사용해주면된다. 기본값이 true이기 때문에 굳이 지정해주지 않아도 된다.
- ### WebSecurityConfigurerAdapter 인퍼테이스를 상속받지 않는다.   
SecurityFilterChain 인터페이스 함수를 하나 만들어서 Bean으로 등록해준 후 그 안에서 기존에 설정했던 코드를 구현한다. 이때 HttpSecurity를 주입받아 사용하면 된다.
- ### 설정 코드를 구현하는 방식이 non-lamda-DSL에서 lamda-DSL로 변경되었다.
