---
layout: post
title:  "Lombok 사용 중 constructor ... is already defined in class 오류 발생 해결"
date:   2018-11-17 23:43:19
category: backend-java
tags: [Lombok]
comments: true
draft: false
---
Lombok 버전 : 1.16.22

스프링 Boot 프로젝트 배포를 위해 Maven Build 를 수행하던 중 Lombok 어노테이션을 적용해둔 도메인 클래스에서 컴파일 오류가 발생했다.

구글링 해본 결과 @Data와 @NoArgsConstructor를 같이 쓸 때 발생하는 버그로 지금은 fix된듯 하다.

[Github에 등록된 Lombok 이슈 확인](https://github.com/rzwitserloot/lombok/issues/1703)

```java
@NoArgsConstructor
@Data
public class SomeDomain {
    ....
}
```
