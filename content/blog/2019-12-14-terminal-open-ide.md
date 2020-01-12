---
layout: post
title: "mac terminal 에서 특정 프로젝트를 vscode 또는 IntelliJ 실행하기"
date:   2019-12-14 11:43:19
category: etc-ide
tags: [vscode,IntelliJ,mac]
comments: true
draft: false
---
mac 사용하다 보면 ide에서 Finder UI를 통해 특정 프로젝트를 open 하는 일이 자주 있는데요.
저의 경우 vscode와 IntelliJ에서 그런 경우가 자주 있는 편입니다.
<!--more-->

## 특정 프로젝트를 vscode로 실행하기
vscode 의 경우 아주 간단합니다.
터미널에서 프로젝트 경로로 이동한 후 `code .` 명령어를 입력하면 해당 프로젝트 open으로 vscode가 실행됩니다.
```sh
cd 프로젝트경로
code .
```

## 특정 프로젝트를 IntelliJ로 실행하기
IntelliJ의 경우도 vscode와 유사하지만 1가지 차이점이 있습니다.
IntelliJ 실행 시 사용하는 `idea` 명령어가 있는 경로가 환경변수에 등록되어 있지 않기 때문인데요. 이는 IntelliJ에서 간단히 추가가 가능합니다.
IntelliJ를 실행 후 `Tools > Create Command-line Launcher...` 메뉴를 클릭하면 터미널에서 `idea`명령어를 바로 사용할 수 있게됩니다.

명령어를 환경변수에 추가한 후 실행할 프로젝트로 이동해 아래 명령어를 입력하면 해당 프로젝트 open 으로 IntelliJ가 실행되게 됩니다.
아래 예시의 경우 제가 실행할 프로젝트가 maven 프로젝트여서 `pom.xml`을 대상 파일로 지정했구요. gradle이면 `build.gradle` 등 다른 타입의 프로젝트는 
해당하는 파일을 지정하면 됩니다.

```sh
cd 프로젝트경로
idea pom.xml
```