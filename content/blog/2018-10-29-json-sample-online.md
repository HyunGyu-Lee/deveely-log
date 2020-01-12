---
layout: post
title:  "JSON 샘플 데이터 제공 사이트 소개"
date:   2018-10-29 00:11:25
category: etc-it-basic
tags: [FrontEnd,JSON]
comments: true
draft: false
---
개발하다보면 JSON 샘플 데이터가 필요한 경우가 자주 있습니다.

>[JSONPlaceholder](https://jsonplaceholder.typicode.com)

위 사이트에서는 게시글, 댓글, 사진 등 다양한 컨셉의 샘플 데이터를 제공하며 사진의 경우 image 주소까지 제공하여 편리하게 사용할 수 있습니다.
<!--more-->
사용방법은 다음과 같이 아주 간단합니다.
---
```js
fetch('https://jsonplaceholder.typicode.com/comments')
    .then(response => response.json())
    .then(json => console.log(json));
```

콘솔에 comments(댓글) API로 제공되는 500개의 샘플 데이터가 출력되는 것을 확인할 수 있습니다.  
백엔드가 구축되어있지 않은 상태에서 프론트엔드 개발 시 유용하게 사용할 수 있을 것 같습니다.
