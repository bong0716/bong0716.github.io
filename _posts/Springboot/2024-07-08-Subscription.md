---
layout: post
title: "[Springboot] 포토그램 구독/구독취소 구현"
date: 2024-07-08 10:00:23 +0900
categories: "Springboot"
tag: ["Springboot", "Photogram"]
---  

아이티윌의 국비지원 [스프링부트 SNS 포토그램 프로젝트] 강의를 수강하며 정리한 내용입니다.

<br>

## 오늘의 실습
- 구독 모델 및 API 제작

<br>

## 🔎그 전에 알아야할 것
---
### 연관관계 개념
[여기 참고](https://bong0716.github.io/posts/Table-Relationships/)

---

## 실습
### **[Subscribe.java]**
```java
@Builder
@AllArgsConstructor
@NoArgsConstructor
@Data
@Entity
@Table()
public class Subscribe {
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private int id;
	
	@JoinColumn(name = "fromUserId")
	@ManyToOne // User 1 : Sub N
	private User fromUser;
	
	@JoinColumn(name = "toUserId")
	@ManyToOne
	private User toUser;
	
	private LocalDateTime createDate;
	
	@PrePersist
	public void createDate() {
		this.createDate = LocalDateTime.now();
	}
}
```
`@Table` : 1번 유저가 2번 유저를 중복해서 팔로우 할 수 없으니 유니크 제약 조건을 추가한다.   
`@JoinColumn` : 칼럼의 이름을 설정해주는 어노테이션으로 설정해주지 않으면 알아서 이름을 만들어 준다.   
`@ManyToOne` : 다대일 관계에 사용

![image](https://github.com/bong0716/photogram/assets/119990564/c8f6dc48-d714-43b2-a2c6-7d9bd5c5a303)

---

![image](https://github.com/bong0716/photogram/assets/119990564/ed967859-87fc-4523-8d92-e9ea92e1acf9)  
구독테이블과 구독유저테이블, 구독받는유저테이블 관계를 보면    
subscribe N : fromUserId 1    
subscribe N : toUserId 1    
이므로 `@ManyToOne` 어노테이션을 붙여줘야 한다. (Subscribe 엔티티가 여러 User 엔티티와 다대일 관계를 갖는다.)

<br>

서버를 실행했을 때 subscribe 테이블이 자동 생성되어 있는 것을 확인!

### **[SubscribeRepository.java]**
```java
public interface SubscribeRepository extends JpaRepository<Subscribe, Integer>{

	@Modifying // INSERT, DELETE, UPDATE 를 네이티브 쿼리로 작성하려면 해당 어노테이션 필요
	@Query(value = "INSERT INTO subscribe(fromUserId, toUserId, createDate) VALUES(:fromUserId, :toUserId, now())", nativeQuery = true)
	void mSubscribe(@Param("fromUserId") int fromUserId, @Param("toUserId") int toUserId);
	
	@Modifying
	@Query(value= "DELETE FROM subscribe WHERE fromUserId=:fromUserId AND toUserId=:toUserId", nativeQuery=true)
	void mUnSubscribe(@Param("fromUserId") int fromUserId, @Param("toUserId") int toUserId);
}

```
`@Modifying` : INSERT, DELETE, UPDATE 를 네이티브 쿼리로 작성하려면 해당 어노테이션 필요하다.   

<br>

fromUserId와 toUserId의 타입이 테이블에선 User이고 Service에선 int이기에 nativeQuery를 만들어주었다. 

### **[SubscribeService.java]**
```java
@RequiredArgsConstructor
@Service
public class SubscribeService {

	private final SubscribeRepository subscribeRepository;
	
	@Transactional
	public void 구독하기(int fromUserId, int toUserId) {
		try {
			subscribeRepository.mSubscribe(fromUserId, toUserId);
		}catch(Exception e) {
			throw new CustomApiException("이미 구독을 하였습니다.");
		}
	}
	
	@Transactional
	public void 구독취소하기(int fromUserId, int toUserId) {
		subscribeRepository.mUnSubscribe(fromUserId, toUserId);
	}
}
```
구독취소의 경우 오류날 상황이 없다고 생각해 예외처리 안 함! 

### **[SubscribeApiController.java]**
```java
@RequiredArgsConstructor
@RestController
public class SubscribeApiController {

	private final SubscribeService subscribeService;
	
	@PostMapping("/api/subscribe/{toUserId}")
	public ResponseEntity<?> subscribe(@AuthenticationPrincipal PrincipalDetails principalDetails, @PathVariable int toUserId){
		subscribeService.구독하기(principalDetails.getUser().getId(), toUserId);
		return new ResponseEntity<>(new CMRespDto<>(1, "구독 성공", null), HttpStatus.OK);
	}
	
	@DeleteMapping("/api/subscribe/{toUserId}")
	public ResponseEntity<?> unSubscribe(@AuthenticationPrincipal PrincipalDetails principalDetails, @PathVariable int toUserId){
		subscribeService.구독취소하기(principalDetails.getUser().getId(), toUserId);
		return new ResponseEntity<>(new CMRespDto<>(1, "구독 취소성공", null), HttpStatus.OK);
	}
}
```

---

### 포스트맨으로 동작 확인

![image](https://github.com/bong0716/photogram/assets/119990564/8a9a8b54-f747-4cc1-aa12-a2df91c01335)   
로그인을 먼저 해준 후

<br>

![image](https://github.com/bong0716/photogram/assets/119990564/9ee35d7d-6983-45df-923f-4b161b435328)   
구독 요청을 보냈을 때 잘 동작하는 것을 확인!   

<br>

![image](https://github.com/bong0716/photogram/assets/119990564/34d9af55-5f40-4e84-9d03-8ad285f84fc3)   
중복해서 구독을 요청할 경우 예외처리 핸들러가 잘 동작하는 것도 확인!

<br>

![image](https://github.com/bong0716/photogram/assets/119990564/170d61d9-7955-4f10-97c8-27f4fde34be5)    
DB에도 구독정보 삽입되는지 확인!



