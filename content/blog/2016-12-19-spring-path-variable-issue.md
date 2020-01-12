---
layout: post
title:  "Spring Path variable 사용 시 확장자 문제"
date:   2016-12-19 19:34:31
category: backend-spring-framework
tags: [Java,Spring Framework,Path Variable]
comments: true
draft: false
---
```java
@RequestMapping("/temp/{filename}")
public ResponseEntity<byte[]> tempView(@PathVariable String filename) throws IOException {
        logger.info("Return Temp image data");
        logger.info(filename);
        return null;
}
```
<!--more-->
위와같이 매핑한 후

```sh
curl localhost:8080/content/temp/testFile.jpg
```
위 주소로 요청을 날렸더니 에러가 발생했다.

문제는 로그에 찍힌 filename에 .jpg는 사라진채 testFile만 들어와지는 것이었다.

```java
@RequestMapping("/temp/{filename:.+}")
```

리퀘스트 매핑을 위와같이 바꿔주니 확장자까지 잘 들어옴을 확인할 수 있었다.
