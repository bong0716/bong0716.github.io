---
layout: post
title: "[JPA] 연관관계 개념 이해하기(일대일, 다대일, 다대다)"
date: 2024-07-08 10:00:23 +0900
categories: "JPA"
tag: JPA
---  

---
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

### 연관관계란❓
**엔티티 간의 관계를 정의하고 관리**하는 것을 의미한다.

### 연관관계 매핑이란❓
관계형 데이터베이스에서는 테이블 간의 관계를 외래 키를 통해 관리하는데, JPA에서는 엔티티 간의 관계를 매핑하여 이러한 관계를 객체 지향적으로 다룰 수 있게 해준다. **객체의 참조와 테이블의 외래키를 매핑**하는 것! 

---

JPA에서의 연관관계는 크게 세 가지로 분류된다.

<br>

### 1. 일대일(1:1) 관계란
- 하나의 엔티티가 다른 하나의 엔티티와 1:1로 매핑되는 관계이다. 
- `@OneToOne` 어노테이션을 사용한다.
- 주로 외래키를 통해 두 엔티티를 연결한다.  

<br>

주로 주 테이블과 보조 테이블 사이에서 사용된다. 예를 들어, 사람과 주민등록번호 정보처럼 한 엔티티가 다른 엔티티에 대해 유일한 정보를 가질 때, 보통 주 테이블에 외래 키를 추가하여 연결한다.  

```java
@Entity
public class Person {
    @Id @GeneratedValue
    private Long id;
    private String name;

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "identity_id")
    private Identity identity;
}

@Entity
public class Identity {
    @Id @GeneratedValue
    private Long id;
    private String ssn;

    @OneToOne(mappedBy = "identity")
    private Person person;
}
```
Person 엔티티가 주 테이블이고 Identity 엔티티가 보조 테이블로, 외래 키는 Person 엔티티에 위치한다.  

<br>

`@JoinColumn(name = "identity_id")` : 외래 키 identity_id로 Identity와 연결   
`@OneToOne(mappedBy = "identity")` : Person 엔티티의 identity 필드에 의해 매핑   

---

### 2. 일대다/다대일(1:N/N:1) 관계란
- 하나의 엔티티가 여러 개의 다른 엔티티와 매핑되는 관계이다. 
- `@OneToMay`와 `@ManyToOne` 어노테이션을 사용한다.
- **외래키는 N 측의 엔티티에 존재한다.**

<br>

많은 수의 엔티티가 한 개체나 위치에 연결되는 경우에 적합하다. 예를 들어, 여러 사람이 한 도시에 거주할 때, 도시 엔티티가 외래 키를 가지고 연결된다. 

```java
@Entity
public class City {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @OneToMany(mappedBy = "city")
    private List<Person> residents = new ArrayList<>();
}

@Entity
public class Person {
    @Id @GeneratedValue
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "city_id")
    private City city;
}
```
`@OneToMany` 어노테이션을 사용하여 Person 엔티티와의 일대다 관계를 설정   
`@ManyToOne` 어노테이션을 사용하여 City 엔티티와의 다대일 관계를 설정하며, 외래 키(city_id)를 통해 관계를 정의  

---

### 3. 다대다(N:N) 관계란
- 여러 개의 엔티티가 서로 다수와 매핑되는 관계
- `@ManyToMany` 어노테이션을 사용한다.
- **중간 테이블을 사용하여 두 엔티티를 연결한다.**

<br>

많은 수의 엔티티가 서로 다수의 관계를 가질 수 있는 경우에 사용된다. 예를 들어, 학생과 강의 관계에서 한 학생이 여러 강의를 수강할 수 있고, 한 강의도 여러 학생에게 개설될 수 있을 때, 중간테이블로 연결된다.  

```java
@Entity
public class Student {
    @Id @GeneratedValue
    private Long id;
    private String name;

    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private List<Course> courses = new ArrayList<>();
}

@Entity
public class Course {
    @Id @GeneratedValue
    private Long id;
    private String title;

    @ManyToMany(mappedBy = "courses")
    private List<Student> students = new ArrayList<>();
}
```
`@JoinTable(name = "student_course")` : student_course라는 중간 테이블을 자동으로 생성    
`joinColumns = @JoinColumn(name = "student_id")` : 중간 테이블의 student_id 외래 키 컬럼이 Student 엔티티의 기본 키를 참조   
`inverseJoinColumns = @JoinColumn(name = "course_id")` : 중간 테이블의 course_id 외래 키 컬럼이 Course 엔티티의 기본 키를    참조


