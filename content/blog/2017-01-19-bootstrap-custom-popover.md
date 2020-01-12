---
layout: post
title:  "Bootstrap popover(팝오버) 커스터마이징"
date:   2017-01-19 19:35:11
category: frontend-jQuery
tags: [javascript,jQuery,Bootstrap]
comments: true
draft: false
---
부트스트랩 홈페이지에서 볼 수 있는 예제는 다음과 같다.
<!--more-->
```js
var options = {
  html : true,
  container : 'body',
  trigger : EVENT(click, hover 등),
  content : function() {
    return 'popover로 보여질 내용 HTML';
  }
};

$('selector').popover(options);
```

위처럼하면 selector 요소에 trigger에 이벤트 발생 시 팝오버가 보여진다.

하지만 trigger를 hover지정 시 content에 지정한 function이 2번 실행하는 문제가 발생했다.

구글링 결과 Bootstrap이 popover를 위해 내부적으로 2번 호출되는 메소드가 있어서 그렇다는것도 있고, hover에 `mouseenter`, `mouseover` 등 바인딩되는 이벤트가 많아서 그렇다는 말도 있었다.

아무튼 그래서 다음과 같이 해결했다.

```js
$('selector').on(
  mouseenter : function() {
    $(this).popover({
      /* trigger를 manual로 지정, 다른 옵션은 사용하는 상황에 따라 입력 */
      trigger : 'manual'
    });
    $(this).popover('show');
  },
  mouseleave : function() {
    $(this).popover('destroy');
  }
)
```
