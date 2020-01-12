---
layout: post
title:  "새 이클립스 설치 후 진행해야할 설정 목록"
date:   2018-07-14 01:34:21
category: etc-ide
tags: [Eclipse]
comments: true
draft: false
---
#### 1. 자동 Validation Off
1. Window > Preferences 메뉴에서 Validation 탭 설정 중 Build 시 HTML, JSP, XML 등에 대한 Validation 체크 해제
2. 필요 시 수동 Validation을 위해 Manual은 체크상태 유지

#### 2. 스펠링 체크 해제
1. General > Editors > Text Editors > Spelling 탭에서 Enable spell checking 체크 해제

#### 3. Indentation Tab > Space 변경
1. General > Editors > Text Editors 탭에서 Insert spaces for tabs 체크
2. Java > Code Style > Formatter 탭에서 New 버튼을 클릭하여 새 프로필 등록 후 Indentation 항목의 Tab policy를 Spaces Only로 변경
3. Javascript > Code Style > Formatter 탭 또한 2번 과정 동일하게 적용
4. Web > CSS Files > Editor 탭에서 Indent using spaces 선택 및 Indentation size 4로 변경
5. Web > HTML Files > Editor, XML > XML Files > Editor 탭에 대해 4번 과정 동일하게 적용

#### 4. 검색 대화상자 단순화
1. Ctrl + H 누르면 나오는 검색상자의 Customize 버튼을 클릭, 사용할 탭만 체크한 후 나머지 제외
