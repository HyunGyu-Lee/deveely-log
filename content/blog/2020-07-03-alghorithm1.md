---
layout: post
title: "[프로그래머스] 완주하지 못한 자들"
date:   2020-07-03 13:04:19
category: alghorithm
tags: [Spring Framework,Spring Security]
comments: true
draft: false
---
## 문제 정보
제목 : [완주하지 못한 자들](https://programmers.co.kr/learn/courses/30/lessons/42576?language=java)

카테고리: 해시

#### 문제 설명 및 제약사항
```
# 설명
수많은 마라톤 선수들이 마라톤에 참여하였습니다. 단 한 명의 선수를 제외하고는 모든 선수가 마라톤을 완주하였습니다.
마라톤에 참여한 선수들의 이름이 담긴 배열 participant와 완주한 선수들의 이름이 담긴 배열 completion이 주어질 때, 완주하지 못한 선수의 이름을 return 하도록 solution 함수를 작성해주세요.

# 입출력 예시
participant	completion	return
[leo, kiki, eden]	[eden, kiki]	leo
[marina, josipa, nikola, vinko, filipa]	[josipa, filipa, marina, nikola]	vinko
[mislav, stanko, mislav, ana]	[stanko, ana, mislav]	mislav

# 입출력 예 설명
예제 #1
leo는 참여자 명단에는 있지만, 완주자 명단에는 없기 때문에 완주하지 못했습니다.

예제 #2
vinko는 참여자 명단에는 있지만, 완주자 명단에는 없기 때문에 완주하지 못했습니다.

예제 #3
mislav는 참여자 명단에는 두 명이 있지만, 완주자 명단에는 한 명밖에 없기 때문에 한명은 완주하지 못했습니다.

# 제약사항
- 마라톤 경기에 참여한 선수의 수는 1명 이상 100,000명 이하입니다.
- completion의 길이는 participant의 길이보다 1 작습니다.
- 참가자의 이름은 1개 이상 20개 이하의 알파벳 소문자로 이루어져 있습니다.
- 참가자 중에는 동명이인이 있을 수 있습니다.
```

## 나의 솔루션
```java
import java.util.Map.Entry;
import java.util.Map;
import java.util.HashMap;

class Solution {
    public String solution(String[] participant, String[] completion) {
        String answer = "";
        Map<String, Integer> hm = new HashMap<>();

        for (String player : participant) hm.put(player, hm.getOrDefault(player, 0) + 1);
        for (String player : completion) hm.put(player, hm.get(player) - 1);
        for (Entry<String, Integer> entry : hm.entrySet()) {
            if (entry.getValue() != 0) {
                answer = entry.getKey();
                break;
            }
        }
        return answer;
    }
}
```

#### 풀이과정
- 문제의 제약조건에 동명이인이 있을수 있다는 가능성이 있기 때문에 참가자 / 완료자 이름 기준으로 카운트를 계산했다.
- 참가자 명단에서는 +1을 해주고 완료자 명단에서는 -1을 해주었다.
- 만약 완료를 했다면 결과 Map에서 값은 0일 겻이다.