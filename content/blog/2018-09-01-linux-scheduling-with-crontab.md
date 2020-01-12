---
layout: post
title:  "리눅스 스케쥴러 crontab 사용"
date:   2018-09-01 01:21:01
category: etc-unix/linux
tags: [Linux,Unix,crontab]
comments: true
draft: false
---
크론탭이란 특정 작업을 특정 시기마다 수행하고 싶을때 이용할 수 있는 리눅스 스케줄 기능입니다.
크론탭의 기본 사용방법은 다음과 같습니다.
<!--more-->
#### 기본 사용법
```sh
# 크론탭 조회, 현재 머신에 등록된 스케줄 목록이 조회됩니다.
crontab -l

# 크론탭 등록 / 수정, vi편집기 형태로 크론탭 스케줄을 편집(edit)할 수 있습니다.
crontab -e

# 크론탭 삭제
crontab -d
```

#### 스케쥴 작성법
크론탭 표현식의 기본적인 형식은 다음과 같습니다.

```bash
[분(0-59)] [시(0-23)] [일(1-31)] [월(1-12)] [요일(0-7)] [실행할 명령어]
```

[실행할 명령어]를 언제 실행할 지 앞의 5자리를 통해 결정하는 형태입니다.

```sh
# 금요일 오전 1시 30분에 runbatch.sh을 실행
30 1 * * 5 /app/script/runbatch.sh

# 매 0분, 20분, 40분 마다 runbatch.sh를 실행
0,20,40 * * * * /app/script/runbatch.sh

# 매 분 마다 실행
* * * * * /app/script/runbatch.sh

# 보통 매 분마다 실행은 가독성을 위해 */1로 명시적으로 작성합니다.
*/1 * * * * /app/script/runbatch.sh

# 매 3분 마다 실행
*/3 * * * * /app/script/runbatch.sh
```

크론탭은 반드시 한 줄에 하나의 스케줄이 작성되야합니다. 즉 명령어 또한 한 줄이어야 하고 보통은 쉘 스크립트를 작성한 후 그 스크립트를 등록하는 방식을 이용합니다.

#### 로깅
출력 리다이렉션을 통해 스케줄 실행 결과를 로그로 남길 수 있습니다.
```sh
# runbatch.sh 수행로그를 runbatch.log 파일로 남김 (표준 출력만 해당)
* * * * * /app/script/runbatch.sh > /app/logs/runbatch.log

# runbatch.sh 수행로그를 runbatch.log 파일로 남김 (표준 에러 출력을 표준 출력으로 리다이렉트)
* * * * * /app/script/runbatch.sh > /app/logs/runbatch.log 2>&1
```

리눅스에서 1은 표준출력, 2는 표준에러출력을 의미하고 > 를 통해 출력을 내보낼 수 있습니다.   
`>>` 를 하면 append하게됩니다.  
"2>&1" 의 의미는 "2(표준에러출력)을 1(표준출력)으로 내보낸다" 라는 의미를 갖습니다.
