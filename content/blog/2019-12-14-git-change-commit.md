---
layout: post
title: "git commit 날짜, author 변경하기"
date:   2019-12-14 01:43:19
category: git
tags: [git,commit]
comments: true
draft: false
---
git을 사용하면 거의 대부분 원격 저장소로 github을 많이 이용합니다.
github엔 저장소에 commit 내역을 시각화하여 보여주는 기능(잔디밭..)이 있는데요. 가끔 실수로 날짜를 못지키거나, author를 잘못지정해 색칠이 안되는 경우가 있습니다.
잔디밭에 신경을 쓰시는 분이라면 빈 구멍이 굉장히 신경이 쓰이실수 있는데요. 이 때 사용할 수 있는 방법을 공유합니다.
<!--more-->

## git commit 변경
꼭 잔디밭이 아니더라도 commit의 날짜를 변경해야하는 경우는 종종 있을것입니다.

기본적으로 commit 변경은 변경할 commit을 찾는것에서 시작합니다.

#### 1. commit 히스토리 확인
```sh
git log
```

`git log` 명령어를 실행하면 아래와 같이 commit 히스토리를 확인할 수 있습니다.
아래 내역에서 수정하고싶은 commit의 직전 commit의 해쉬값을 복사합니다.
이 예시에서 저는 `책 목차 추가` 커밋을 수정하기 위해 그 아래 커밋인 `65068f594d469e848beab5fd475219f428339436`값을 사용하겠습니다.
log 명령 결과화면에서 `q`를 입력하면 화면이 종료됩니다.

```
commit 5b52b3874068dfd0b1f77023eac03113f3b6db9a
Author: HyunGyu-Lee <gusrb0808@naver.com>
Date:   Mon Dec 9 00:32:41 2019 +0900

    컨텐츠 추가(DDD Start!)

commit 0c6be06e2ede8cac5bc1b78b511620583e159bf5
Author: HyunGyu-Lee <gusrb0808@naver.com>
Date:   Fri Nov 8 13:35:32 2019 +0900

    책 목차 추가

commit 65068f594d469e848beab5fd475219f428339436
Author: HyunGyu-Lee <gusrb0808@naver.com>
Date:   Tue Nov 5 01:01:48 2019 +0900

    collapse 태그 인식 불가 분제 fix

```

#### 2. 날짜/author 변경
해쉬값을 확인한 후 `git rebase` 명령을 입력합니다.

```sh
git rebase -i 65068f594d469e848beab5fd475219f428339436
```

그럼 아래와 같은 UI가 나오는데요. 변경할 대상 커밋의 커멘드를 `edit`으로 변경하고 저장합니다. (vi)

자세한 커멘드는 아래 주석에 적혀있습니다. 

```
edit 0c6be06 책 목차 추가
pick 5b52b38 컨텐츠 추가(DDD Start!)

# Rebase 65068f5..d896625 onto 65068f5 (4 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

그럼 다음과 같이 rebase 모드에 진입하게 되고 해당 커밋으로 이동하게됩니다.
수정을 하려면 `git commit --amend`를 수정하지 않고 넘어가려면 `git rebase --continue`를 입력하면 됩니다.
```sh
>> git rebase -i 65068f594d469e848beab5fd475219f428339436
Stopped at 0c6be06...  책 목차 추가
You can amend the commit now, with

  git commit --amend

Once you are satisfied with your changes, run

  git rebase --continue
>> 
```

변경할 커밋에서 amend 를 통해 author 혹은 날짜를 수정하면 됩니다.


```sh
# 날짜 변경
GIT_COMMITTER_DATE="{날짜}" git commit --amend --no-edit --date "{날짜}"

# author 변경
git commit --amend --author "username <email>"
```

그다음 아까 언급한것처럼 수정을 하려면 `git commit --amend`를 수정하지 않고 넘어가려면 `git rebase --continue`를
입력하면 되고, 더 이상 수정할 것이 없으면 rebase 모드가 종료되게 됩니다. 수정을 마치고 push 하여 마무리 해주면 github에도 반영되게 됩니다.

