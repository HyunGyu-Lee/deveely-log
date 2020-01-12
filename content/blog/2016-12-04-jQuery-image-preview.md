---
title: jQuery를 이용한 input file 이미지 선택 시 미리보기
date: 2016-12-04 17:18:21
category: frontend-jQuery
draft: false
---
```html
<!-- 미리보기 영역으로 사용할 img 태그 -->
<img src="" id="preview"/>

<input type="file" id="imgSelector"/>
```

파일 선택 시 발생하는 input file태그 change 이벤트를 이용하는 방법이다.   
선택한 파일을 읽어 base64인코드 형식으로 img태그에 지정해준다.

```js
$('#imgSelector').change(function(){
    setImageFromFile(this, '#preview');
});

function setImageFromFile(input, expression) {
    if (input.files && input.files[0]) {
        var reader = new FileReader();
        reader.onload = function (e) {
            $(expression).attr('src', e.target.result);
        }
        reader.readAsDataURL(input.files[0]);
    }
}
```
