---
title: jQuery each메소드 사용 시 break, continue 제어
date: 2016-12-13 20:57:21
category: frontend-jQuery
draft: false
---
jQuery.each(callback) 메소드 사용 시 break, continue 등의 제어를 해야하는 경우가 있다.  

이 때 false를 return 하면 break, true를 return 하면 continue와 같은 역할을 한다.  

```js
$('.category-item').each(function(){
    if($(this).hasClass('highlight')) {
        return false;    // each문을 빠져나간다. break; 와 같음
    }
});
```
