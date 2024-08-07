---
layout: post
title: "[Spring] Spring Framework(스프링 프레임워크)란?"
date: 2023-12-14 13:00:23 +0900
categories: "Spring"
tag: Spring
---
<br>

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-3561381376929023"
     crossorigin="anonymous"></script>
<ins class="adsbygoogle"
     style="display:block; text-align:center;"
     data-ad-layout="in-article"
     data-ad-format="fluid"
     data-ad-client="ca-pub-3561381376929023"
     data-ad-slot="1405810651"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>


<div style="text-align:center;">
  <img src="https://github.com/bong0716/photogram/assets/119990564/2f712f21-7a44-4b07-9e60-505622a82187" alt="spring logo">   
</div>

<br><br>

Spring Framework(스프링 프레임워크)란
자바 엔터프라이즈 개발을 편하게 해주는 오픈 소스 경량급 애플리케이션 프레임워크이다.     
> *프레임워크 : 애플리케이션 개발을 위한 뼈대를 제공   
*엔터프라이즈 : 기업/사업

<br>

### 스프링 핵심
---  
### **IoC(Inversion of Control) : 제어의 역전**   
객체의 생성부터 소멸까지의 제어흐름을 사용자가 아닌 특별한 객체에게 위임하는 것을 말한다. 스프링 프레임워크에서는 애플리케이션 컨텍스트가 흐름을 제어하기 때문에 IoC 프레임워크다.      
<br>
어떤 객체의 사용 여부가 타 객체에 의해 수동적으로 결정되어 사용된다면 IoC 개념이 적용된 것이라고 볼 수 있다. 

<br>

### **DI(Dependency Injection) : 의존성 주입**   
각 클래스 간의 의존성을 빈 설정 정보를 바탕으로 런타임 시점에 컨테이너가 자동으로 주입하는 것이다. 외부에서 의존성을 넣어줬다고 하여 의존성 주입이라고 한다. DI는 스프링 프레임워크가 가진 차별화 요소이다. 

<br>

### **DL(Dependency Lookup) : 의존성 검색**   
의존관계가 있는 객체를 외부에서 주입 받는 것이 아닌 의존관계가 필요한 객체에서 직접 검색하는 방식을 말한다. 해당 객체는 의존하고자 하는 인터페이스 타입만 지정해서 검색할 뿐 해당 인터페이스를 구현한 구체적인 클래스 객체에 대한 결정과 해당 객체에 대한 생명주기는 IoC 컨테이너에서 책임진다. 

<br>

### 스프링 특징
---  
### **AOP(Aspect Oriented Programming) : 관점 지향 프로그래밍**   
횡단 관심사를 모듈로 분리하여 중복을 없애고 효율적인 유지보수와 재활용성을 극대화한 프로그래밍 패러다임을 뜻한다.   
> *회당관심사 : 보안이나 로그, 트랜잭션과 같이 비즈니스 로직은 아니지만 반드시 처리가 필요한 부가적인 공통 기능을 일컫는다.  
 
<br>

### **POJO(Plain Old Java Object)**
getter/setter만 가진 단순한 자바 오브젝트를 뜻한다. 스프링을 사용하면 자바 코드를 이용해서 객체를 구성하는 방식을 그대로 사용할 수 있다. 그럼 생산성에 유리하고, 코드 테스트 작업에 유리하다는 장점이 생긴다. 

<br>

### **MVC(Model View Controller) 패턴**
Model, View, Controller의 세 구성요소로 나누어 사용자 인터페이스와 비지니스 로직을 분리해 개발하는 방법을 말한다. MVC에서는 Model1과 Model2로 나누어져 있으며 일반적인 MVC는 Model2를 지칭한다.     


- **Model**    
Model은 애플리케이션의 데이터와 비즈니스 로직을 담당하는 부분이다. 주로 데이터를 다루고 가공하는 역할을 수행하며, View나 Controller와 직접적으로 상호작용하지 않는다. 


- **View**    
View는 사용자에게 보여지는 부분으로, 사용자 인터페이스(UI)를 담당한다. 웹 애플리케이션의 경우 HTML, CSS, JavaScript 등을 이용하여 화면을 구성하고 사용자에게 정보를 시각적으로 전달한다. View는 주로 사용자가 보는 부분을 생성하며, 사용자의 요청을 Controller로 전달한다.


- **Controller**   
Controller는 Model과 View를 연결하는 부분으로, 사용자가 어떠한 요청을 하면 Controller가 그 요청을 받아들이고, 해당 요청에 대한 처리를 Model에게 지시하고, Model이 데이터를 처리한 후 다시 Controller로 결과를 전달하고, Controller는 View에게 전달하여 사용자에게 최종 결과를 보여준다.    

<br> 

MVC 패턴은 각 부분이 서로 독립적으로 구성되어 유지보수 및 확장이 용이하며, 코드의 가독성과 유지보수성을 높여주는 장점이 있다
<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-3561381376929023"
     crossorigin="anonymous"></script>
<!-- 4 -->
<ins class="adsbygoogle"
     style="display:block"
     data-ad-client="ca-pub-3561381376929023"
     data-ad-slot="5661901813"
     data-ad-format="auto"
     data-full-width-responsive="true"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>
<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-3561381376929023"
     crossorigin="anonymous"></script>
<ins class="adsbygoogle"
     style="display:block; text-align:center;"
     data-ad-layout="in-article"
     data-ad-format="fluid"
     data-ad-client="ca-pub-3561381376929023"
     data-ad-slot="1405810651"></ins>
<script>
     (adsbygoogle = window.adsbygoogle || []).push({});
</script>
