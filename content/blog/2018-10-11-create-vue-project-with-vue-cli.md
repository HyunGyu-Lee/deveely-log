---
layout: post
title:  "Vue-CLI를 이용한 Vue프로젝트 생성"
date:   2018-10-11 23:39:22
category: frontend-vue
tags: [NodeJS,NPM,VueCLI,Vue]
comments: true
draft: false
---
먼저 Vue-CLI 를 이용하기 위해서는 Node.js과 NPM이 설치되어있어야 한다.  
설치가 안된 경우 [Node.js 및 NPM 설치](/articles/2018-10/install-nodejs-and-npm) 을 참고하여 설치할 수 있다.   
NPM을 이용하여 Vue-CLI 설치 (3.0.x)   
NPM이 설치되어있으면 아래 명령어로 Vue-CLI 를 매우 간단하게 설치할 수 있다.  
<!--more-->

#### Vue-CLI 설치
```sh
npm i -g @vue/cli
```

#### 버전 확인을 통해 정상설치 확인
```sh
vue --version
```

`-g` 옵션은 글로벌 패키지로 설치한다는 옵션으로 해당 PC 개발환경의 모든 프로젝트에서 사용이 가능하도록 하는 것이다.   

#### Vue-CLI를 이용하여 프로젝트 생성
먼저 CMD창에서 프로젝트를 생성할 경로로 이동한다.  
만약 C:\work 디렉토리 밑에 생성하고 싶다면 cd C:\work 로 해당 경로로 이동하면 된다.  

해당 경로에서 다음 명령어를 입력한다.

```sh
vue create [생성할 프로젝트 명]
```

위 명령어를 입력하면 default 설정을 따를지 물어보는데 이는 생성 이후에도 변경 가능한 부분이므로 default를 선택한다.  
한창 생성이 진행되고 Successfully created project... 메시지가 보이면 생성이 완료된 것이다.  

디렉토리를 확인해보면 지정한 프로젝트명으로 디렉토리가 생성되있다.  

이제 NPM에 내장된 웹팩 개발 서버를 통해 간단히 실행해보자  

```sh
cd [프로젝트 디렉토리명]
npm run serve
```

위 명령어를 입력하면 개발서버가 구동되며 http://localhost:8080에 접속하면 generate된 프로젝트를 확인해볼수 있다.
