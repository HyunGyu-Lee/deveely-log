---
layout: post
title:  "Expected linebreaks to be 'LF' but found 'CRLF' 오류 관련"
date:   2018-10-14 01:22:32
category: frontend-vue
tags: []
comments: true
draft: false
---
빌드 시 개행문자 관련 경고가 발생했다.

.eslintrc.js 파일의 rules에 'linebreak-style'을 0으로 하는 설정을 추가하는 것으로 해결했다.

```js
# .eslintrc.js

module.exports = {
    ....
    rules: {
        'linebreak-style': 0,
        ....
    }
}
```
