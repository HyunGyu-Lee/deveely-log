---
layout: post
title:  "Node.js 및 NPM 설치"
date:   2018-10-11 23:26:02
category: frontend-vue
tags: [NodeJS,NPM]
comments: true
draft: false
---
Node.js는 Javascript로 서버사이드 개발을 가능하게 해주는 Javascript 런타임으로 공식사이트에서는 다음과 같이 소개하고 있다.

>Node.js®는 Chrome V8 JavaScript 엔진으로 빌드된 JavaScript 런타임입니다. Node.js는 이벤트 기반, Non 블로킹 I/O 모델을 사용해 가볍고 효율적입니다. Node.js의 패키지 생태계인 npm은 세계에서 가장 큰 오픈 소스 라이브러리 생태계이기도 합니다.

NPM은 Node Package Manager 혹은 Node Package Modules의 약자로 Node.js 기반의 모듈을 모아둔 집합 저장소이다.
<!--more-->
#### Node.js / NPM 설치 (윈도우 기준)
1. https://nodejs.org/en/download/ 접속  
2. LTS 탭에서 Windows Installer로 다운로드
3. 다운받은 msi 파일 실행

별도로 환경변수로 PATH를 잡아주거나 할 필요없이 msi 파일을 통해 한번에 설치가 가능하다.  
설치 확인을 위해 CMD창에서 다음 명령어를 실행해본다.

```sh
# 설치된 Node.js 버전 확인
node -v

# 설치된 NPM 버전 확인
npm -v
```
