---
layout: post
title: "[MyBatis] 동적쿼리 정리"
date: 2024-12-08 09:00:23 +0900
categories: "MyBatis"
tag: ["MyBatis", "DynamicQuery"]
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
  <img src="https://github.com/user-attachments/assets/d6ec2e3f-10b8-4b6b-9ea9-11ca1d98354f" alt="MyBatis logo">   
</div>

<br><br>
 
---
### **MyBatis 동적쿼리란?**
실행 시점의 조건에 따라 **SQL문을 동적으로 생성**할 수 있도록 MyBatis에서 제공하는 기능으로, 유연한 쿼리 작성을 돕는다.     

---
### **`<if>`**
특정 조건에 따라 SQL문을 포함하거나 제외한다.
참인 조건이 모두 실행된다. 

``` xml
<select id="findUsers" parameterType="userDto" resultType="User">
     SELECT *
     FROM USER
     WHERE 1=1
     <if test= 'name != null and name !=""'>
          AND NAME = #{name}
     </if>
     <if test= 'age != null and age !=""'>
          AND AGE = #{age}
     </if>
</select>
```

---

### **`<choose>`, `<when>`, `<otherwise>`**
Switch문처럼 여러 조건 중 하나를 선택한다.   
참인 조건이 나오면 그 조건만 실행하고 종료된다는 점에서 if문과 다르다. 
``` xml
<select id="findUsers" parameterType="userDto" resultType="User">
     SELECT *
     FROM USER
     WHERE 1=1
     <choose>
          <when test='name != null and name !=""'>
               AND NAME = #{name}
          </when>
          <when test='age != null and age !=""'>
               AND AGE = #{age}
          </when>
          <otherwise>
               AND STATUS = 'active'
          </otherwise>
     </choose>
</select>
```

---  
### **`<where>`**
조건에 따라 where절을 자동으로 추가하며, 첫 번째 조건 앞의 불필요한 AND/OR를 자동 제거해준다.  
``` xml
<select id="findUsers" parameterType="userDto" resultType="User">
     SELECT *
     FROM USER
     <where>
          <if test='name != null and name !=""'>
               AND NAME = #{name} 
          </if>
          <if test='age != null and age !=""'>
               AND AGE = #{age}
          </if>
     </where>
</select>
```

첫 번째 조건만 충족 시 실행 쿼리
```xml
SELECT *
FROM USER
WHERE NAME = #{name}
```
두 번째 조건만 충족 시 실행 쿼리
```xml
SELECT *
FROM USER
WHERE AGE = #{age}
```
모두 충족 시 실행 쿼리
```xml
SELECT *
FROM USER
WHERE NAME = #{name}
AND   AGE = #{age}
```

---  
### **`<trim>`**
where절이나 SQL의 특정 부분에서 접두어(prefix)나 접미어(suffix)를 제어한다.  
- prefix : 쿼리문이 시작될 때, 삽입할 문자열이다.
- suffix : 쿼리문이 종료될 때, 삽입할 문자열이다.
- prefixOverids : 실행될 퀴리문 맨 앞단의 단어가 설정값과 동일하다면 해당 문자를 지운다. 
- suffixOverids : 실행될 퀴리문 맨 뒷단의 단어가 설정값과 동일하다면 해당 문자를 지운다. 

```xml
SELECT *
FROM USER
<trim prefix="WHERE" prefixOverrides="AND | OR">
          <if test='name != null and name != ""'>
               NAME = #{name} 
          </if>
          <if test='age != null and age != ""'>
               AGE = #{age}
          </if>
</trim>
```

---  
### **`<foreach>`**
배열이나 컬렉션 데이터를 반복적으로 처리한다.
- collection : 전달받은 인자, List 또는 Array만 가능하다.
- item : 전달받은 인자 값을 alias명으로 대체한다.
- open : 쿼리문이 시작될 때, 삽입할 문자열이다.
- close : 쿼리문이 종료될 때, 삽입할 문자열이다.
- separaator : 반복 사이에 출력할 문자열이다. 
- index : 반복되는 구문 번호이다. 0부터 순차적으로 증가한다. 

```xml
SELECT *
FROM USER
WHERE NAME IN
<foreach collection="nameList" item="name" open="(" separator="," close=")">
          #{name} 
</foreach>
```
nameList=["홍길동", "심청이", "갑을병"] 일 때, 아래와 같이 실행된다. 
```xml
SELECT *
FROM USER
WHERE NAME IN ('홍길동', '심청이', '갑을병')
```


---  
### **`<set>`**
UPDATE문에서 동적으로 컬럼을 설정하며, 불필요한 쉼표(,)를 제거한다. 

```xml
<update id="updateUser" parameterType="userDto">
     UPDATE user
     <set>
          <if test='name != null and name != ""'>
               NAME = #{name},
          </if>
          <if test='age != null and age != ""'>
               AGE = #{age},
          </if>
     </set>
     WHERE ID = #{id}
</update>
```

<script async src="https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js?client=ca-pub-3561381376929023"
     crossorigin="anonymous">
</script>
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
