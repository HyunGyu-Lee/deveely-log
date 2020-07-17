---
layout: post
title: "Redis 개념과 설치, 활용방안"
date:   2020-07-15 13:04:19
category: redis
tags: [redis,in-memory]
comments: true
draft: false
---

## Redis란?
Redis는 REmote Dictionary Server의 약자로 "key-value" 기반 인메모리비 관계형 데이터 베이스다.
모든 데이터를 메모리에 저장하고 조회하기 때문에 빠른 Read, Write 속도를 보장한다. 
다양한 value에 다양한 자료구조를 지원해 사용자 애플리케이션 개발 시 활용도가 높다.

## Redis vs Memcached
Redis를 검색해보면 개념 설명과 함께 Memcached와의 비교글을 많이 볼 수 있다.
Memcached는 메모리 기반이라 처리속도가 빠르고 데이터에 만료 시간을 지정할 수 있고, 저장소 공간이 없으면 LRU 알고리즘에 의해
삭제되는 특징이 있어, 대형 포털에서 Static Page나 검색 결과 등 캐싱 용도로 많이 사용된다. 
다만 프로세스가 죽거나 장비가 shutdown되면
저장된 데이터가 사라지기 때문에 중요 데이터를 저장하면 안된다.

Redis는 메모리 기반 빠른 처리 속도 등의 Memcached와 차이 없는 처리속도를 보장함과 동시에, 데이터를 디스크에도 저장해 영속화가 가능(데이터가 유실될 위험이 없음)하다.
또한 문자열만 저장할 수 있는 Memcached와 달리 List, Set, Sorted Set, Hash 등 여러 자료 구조를 지원한다는 장점이 있다.

##  Mac에 Redis 설치하기
Homebrew를 이용해 redis 서버 설치
```bash
brew install redis
```

관련 커맨드
```bash
# redis 서버 시작 (기본 설정으로)
redis-server

# redis 서버 시작
redis-server [설정 파일 경로]

# redis 서버 백그라운드로 실행
redis-server --daemonize yes

# redis 서버 실행상태 확인
redis-cli ping # PONG 응답이 오면 정상 실행

# redis 서버 중지
redis-cli shutdown
```

기본 설정파일 내용 확인
```
cat /usr/local/etc/redis.conf
```
    
##  Redis 자료구조와 커맨드

### 1. String
가장 간단한 자료 구조, 쉽게 생각하면 문자열에 다른 문자열을 매핑하는 것.
전체 커맨드: https://redis.io/commands#string
```bash
> set hello world
OK

> get hello
"world"
```

### 2. List
일반적인 연결 리스트의 특성을 갖고 있음.
LPUSH / LPOP / RPUSH / RPOP 등 명령어로 양 끝단 엘리먼트 조작을 빠르게 할 수 있고.
LSET / LINDEX 등 명령어로 특정 인덱스의 엘리먼트에 대한 조작도 할 수 있다.

Pub-Sub(생산자-소비자)와 잘 어울리는 자료구조. (생산자가 List에 PUSH하면 소비자가 POP)
Blocking 연산도 지원해 소비자가 POP 시도 시 List에 원소가 없으면 들어올 때 까지 대기하게 할 수도 있다.

전체 커맨드: https://redis.io/commands#list

### 3. Hash
Key하나에 여러 field-value 쌍을 저장할 수 있는 자료구조로 RDB의 테이블과 유사하며 비교해보면 아래와 같다.

| RDB | Redis |
| --- | --- |
| PK | Key |
| Column | Hash Field |
| Value | Hash Value |

즉 Redis Key하나 당 RDB Table의 row라고 생각하면 된다.

만약 아래와 같은 구조를 같는 User 테이블이 있다고하면
| ID | Name | Age |
| --- | --- | --- |
| 1 | Kim | 29 |
| 2 | Lee | 28 |

이를 Redis Hash 구조에서는 아래와 같이 표현된다.
User-1 (HashKey)
| Name | Kim |
| --- | --- |
| Age | 29 |

User-2 (HashKey)
| Name | Lee |
| --- | --- |
| Age | 28 |

전체 커맨드: https://redis.io/commands#hash

### 4. Set
정렬되지 않는 문자열의 모음으로, Set인만큼 원소의 중복은 불가하다.
집합연산 (교집합, 합집합, 차집합) 연산을 지원하기 때문에 객체관의 관계를 표현하는데 용이하다.

예를들어 어떤 블로그 포스트에 연결된 태그 ID [1, 2, 3, 4, 5]를 저장해야하는 경우가 있다면
```
sadd post:1234:tags 1 2 3 4 5

smembers post:1234:tags
```

이렇게 추가할 수 있다. (참고로 Key "post:1234:tags"는 설계하기 나름이지만 여기서는 1234번 포스트의 태그라는 의미로 저렇게 부여함)

참고로 집합연산은 SINTER (교집합), SUNION (합집합), SDIFF (차집합) 커맨드로 수행할 수 있다.

전체 커맨드: https://redis.io/commands#set

### 5. Sorted Set
Set과 마찬가지로 Key하나에 중복되지않는 여러 엘리먼트를 저장할 수 있는 자료구조이다.
단, 원소 추가 시 score를 같이 입력해주고, 내부적으로 이 score를 기준으로 정렬을 유지시키는 특징을 갖고 있다.
ZRANGE (인덱스로 범위 검색), ZRANGEBYSCORE (스코어로 범위검색) 등 연산을 지원한다.

## Redis 키 설계와 사용
Redis 키는 문자열이기 때문에 공백문자 ("")부터 JPEG 파일 등 이진형태로 표현되는 모든 값을 키로 사용할 수 있다. (최대 512MB)

키는 너무 길지 않게 설계하는 것을 권장(조회 성능때문), 너무 길어지면 차라리 Hash를 이용하는 것이 나을수도 있다.

키는 데이터 스키마를 기준으로 "object-type:id" 설계하는 것을 추천한다.

예를들면, "user:1234"라는 키는 User 타입의 ID가 1234인 오브젝트를 의미한다.

주로 사용하는 부호로는  `:`, `.`, `-`등이 있다.

예를들면 "post:123:comments"라는 키는 ID가 123인 포스트(post)의 댓글목록(comments)를 의미한다.

키 관련 대표적인 명령어

| Command | Description |
| --- | --- |
| SORT | 해당 키의 Value를 정렬해서 반환 |
| EXISTS | 해당 키가 존재하는지 여부 반환 |
| DEL | 해당 키를 삭제 |
| TYPE | 해당 키의 Value가 어떤 자료구조인지 반환 |

## Redis와 Expire 기능
https://redis.io/commands/expire

Redis는 인메모리 DB인 만큼 저장할 수 있는 데이터량이 한정적.
메모리에 더 이상 저장할 수 없는 경우 가장 먼저 들어온 데이터를 삭제하거나 가장 오랫동안 사용하지 않은 데이터를 삭제. 혹은, 꽉 차서 더 이상 입력이 불가능해질수도 있음.

가장 좋은 방법은 삭제 / 만료를 직접 설정하는 것이다.
키에 timeout을 걸거나, 유닉스 타임스태프를 지정해 해당 시간에 자동으로 삭제되도록 할 수 있다. (DEL 명령어 사용과 같은 효과)

timeout의 경우 동일한 키의 변경이 있을 경우 갱신된다. 
기본적으로 모든 데이터에 Expire 처리를 할 것을 권장하며, 너무 짧으면 레디스에 부하가, 너무 길면 의미가 없어지니 적절한 시간을 찾는것이 중요하다.

참고
https://medium.com/garimoo/개발자를-위한-레디스-튜토리얼-01-92aaa24ca8cc
https://redis.io/