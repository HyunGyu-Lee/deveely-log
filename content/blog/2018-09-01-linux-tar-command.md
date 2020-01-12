---
layout: post
title:  "리눅스 tar, gz를 이용한 압축"
date:   2018-09-01 01:32:11
category: etc-unix/linux
tags: [Linux,Unix,tar,gz]
comments: true
draft: false
---
tar 명령어는 굉장히 많은 옵션을 사용할 수 있다.

이 포스트에서는 주로 사용하는 패턴인 tar 압축 / 압축해제, tar.gz 압축 / 압축해제 명령어를 기술한다.
<!--more-->
1. tar 압축 / 해제

```sh
# [압축할 디렉토리]를 [압축파일명.tar]로 압축
tar -cvf [압축파일명.tar] [압축할 디렉토리]

# [압축파일명.tar]파일 압축해제
tar -xvf [압축파일명.tar]
```

2. tar.gz 압축 / 해제

```sh
# [압축할 디렉토리]를 [압축파일명.tar.gz]로 압축
tar -zcvf [압축파일명.tar.gz] [압축할 디렉토리]

# [압축파일명.tar.gz]파일 압축해제
tar -zxvf [압축파일명.tar.gz]
```

굉장히 간단하다.

(c)vf / (x)vf => 압축 / 압축해제
(z)cvf / (z)xvf => gzip 옵션 활성

간략하게나마 옵션들의 의미를 설명하면 다음과 같다.

| 옵션 | 의미 |
|---|:---:|
| `-c` | tar 압축 |
| `-v` | 파일을 묶거나 풀 때 대상 파일 목록을 화면에 출력 |
| `-f` | 파일명 지정 |
| `-x` | tar 압축해제 |
| `-z` | gzip 옵션 활성 |
