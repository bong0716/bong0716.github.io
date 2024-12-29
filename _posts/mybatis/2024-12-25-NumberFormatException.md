---
layout: post
title: "[MyBatis] java.lang.NumberFormatException Error 해결"
date: 2024-12-25 09:00:23 +0900
categories: "MyBatis"
tag: ["MyBatis", "Error"]
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
### java.lang.NumberFormatException : For input string: "Y"
문자열 비교 구문에서 뜬금없이 NumberFormatException이 발생했다.   


### 문제가 된 구문
``` xml
     <if test="value == 'Y'">
          AND STATUS = 'active'
     </if>
```

<br>

---

### 원인   
**OGNL(Object Graph Navigation Language) 인터프리터**에서 'Y'같이 한 글자를 홑따옴표('')에 담으면 char형으로 인식해 나는 오류였다. 

---

<br>

### 해결
``` xml
     <if test='value == "Y"'>
          AND STATUS = 'active'
     </if>
```
쌍따옴표를 **홑따옴표로 바꾸니 해결**됐다. 



<br>

---
Mybatis에서 동적쿼리를 사용하는 경우 홑따옴표('')를 사용해 자잘한 오류를 예방하자. 



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
