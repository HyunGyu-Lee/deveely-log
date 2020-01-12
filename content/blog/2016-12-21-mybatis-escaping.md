---
layout: post
title:  "MyBatis 사용 시 부등호 등 이스케이프(escape) 처리"
date:   2016-12-21 22:19:31
category: backend-spring-framework
tags: [Java,Spring Framework,MyBatis,XML]
comments: true
draft: false
---
XML에서 쿼리를 작성할 때 <, >와 같은 비교 연산자를 사용하면

>The content of elements must consist of well-formed character data or markup.

이런 오류메세지가 나타날 수 있다.   
MyBatis에선 XML파일을 통해 쿼리를 작성하는 기능을 지원하는 때 이때 위와같은 문제가 발생할 수 있다.  
XML에서 <, > 를 비교연산자로 판단하지 않고 `<select>` 등 태그의 시작과 끝으로 인식되어 생기는 일이다.  
해결하기 위해서는 비교연산 사용하는 부분에을 <![CDATA[ ]]> 로 감싸주면 된다.  
<!--more-->
```xml
<select id="selectList">
    SELECT *
      FROM SOME_TABLE
     WHERE <![CDATA[ id > 5 ]]>
</select>
```
