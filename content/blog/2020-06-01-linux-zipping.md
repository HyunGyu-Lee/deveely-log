---
layout: post
title: "리눅스 zip, unzip 압축, 압축 풀기"
date:   2020-06-01 20:43:19
category: etc-unix/linux
tags: [Linux,Unix,zip,unzip]
comments: true
draft: false
---
리눅스 환경에서 zip으로 압축 및 .zip파일을 압축해제하는 `zip`, `unzip` 명령어에 대해 정리합니다.

## 1. zip
`zip` 명령어를 통해 특정 파일 혹은 디렉토리를 압축할 수 있습니다.
```sh
zip [압축 파일명.zip] [압축 대상] [압축 대상]...
```

#### 특정 디렉토리 압축
`-r` 옵션은 대상 디렉토리 하위에 또 다른 폴더가 있을경우 전부 포함시키라는 옵션입니다.
```sh
zip -r [압축 파일명.zip] [압축 대상 디렉토리]

# test 폴더 전체 압축해 test.zip 생성
zip -r test.zip test/*
```

#### 2. unzip
`unzip` 명령어를 통해 zip파일 압축 해제를 할 수 있습니다.

#### 현재 폴더에 압축 풀기
```sh
unzip [압축 파일명.zip]
```

#### 특정 디렉토리에 압축 풀기
```sh
unzip [압축 파일명.zip] -d [압축 풀 경로]

# test.zip 파일 압축을 /home/test 위치에 품
unzip test.zip -d /home/test
```