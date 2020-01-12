---
layout: post
title:  "IDE없이 기본 프로젝트 환경 구성"
date:   2017-03-07 11:22:11
category: backend-java
tags: [Java]
comments: true
draft: false
---
#### 1. 프로젝트 디렉토리 생성
1. 소스파일(.java)은 src 디렉토리에 저장
2. 컴파일된 클래스파일(.class)은 class 디렉토리에 저장

```sh
mkdir sample-project
cd sample-project
mkdir src
mkdir class
```
<!--more-->
#### 2. 개발된 소스파일을 src디렉토리에 위치시킨후 컴파일 진행
src디렉토리 이하 모든 자바파일을 컴파일 대상으로 하며, 결과를 bin디렉토리에 저장한다는 의미이다.

```sh
javac ./src/`*`.java -d ./bin
```

#### 3. javadoc 생성
```sh
javadoc -version -author -protected -d ./docs
```

#### 4. 실행가능한 jar파일 생성
- main메소드가 여러개인 경우 메니페스트파일을 통해 실행될 main클래스를 지정할 수 있다.
- 프로젝트 디렉토리에 META-INF 디렉토리 생성 후 MANIFEST.MF 파일 작성
- 메니페스트 파일은 반드시 맨 아래 공백라인 추가해야한다.

```sh
Main-Class: my.sample.project.DemoMain
```

- 이후 `jar`명령을 이용해 메니페스트 파일을 지정하여 jar 파일을 다음과 같이 생성한다.

```sh
jar cvfm MyDemo.jar META-INF/MANIFECT.MF -C ./bin
```
