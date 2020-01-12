---
layout: post
title:  "이클립스 단축키 모음"
date:   2018-08-17 22:17:03
category: etc-ide
tags: [Eclipse]
comments: true
draft: false
---
이클립스 단축키 잘 정리된 글이 있어 공유합니다. [출처](http://w3devlabs.net/wp/?p=16778)
<!--more-->
#### 실행
Ctrl + F11 : 바로 전에 실행했던 클래스 실행
<hr>

#### 소스 이동 관련
Ctrl + 마우스커서(혹은 F3) : 클래스나 메소드 혹은 멤버를 상세하게 검색하고자 할때  
Alt + Left, Alt + Right : 이후, 이전  
Ctrl + O : 해당 소스의 메소드 리스트를 확인하려 할때  
F4 : 클래스명을 선택하고 누르면 해당 클래스의 Hierarchy 를 볼 수 있다.  
Alt + <-(->) : 이전(다음) 작업 화면  
<hr>

#### 문자열 찾기
Ctrl + K : 찾고자 하는 문자열을 블럭으로 설정한 후 키를 누른다.  
Ctrl + Shift + K : 역으로 찾고자 하는 문자열을 찾아감.  
Ctrl + J : 입력하면서 찾을 수 있음.  
Ctrl + Shift + J : 입력하면서 거꾸로 찾아갈 수 있음.  
Ctrl + F : 기본적으로 찾기  
<hr>

#### 소스 편집
Ctrl + Space : 입력 보조장치(Content Assistance) 강제 호출 => 입력하는 도중엔 언제라도 강제 호출 가능하다.  
F2 : 컴파일 에러의 빨간줄에 커서를 갖져다가 이 키를 누르면 에러의 원인에 대한 힌트를 제공한다.  
Ctrl + L : 원하는 소스 라인으로 이동(로컬 히스토리 기능을 이용하면 이전에 편집했던 내용으로 변환이 가능하다.)  
Ctrl + Shift + Space : 메소드의 가로안에 커서를 놓고 이 키를 누르면 파라미터 타입 힌트를 볼 수 있다.  
Ctrl + D : 한줄 삭제  
Ctrl + W : 파일 닫기  
Ctrl + I : 들여쓰기 자동 수정  
Ctrl + Shift + / : 블록 주석(`/* */`)  
Ctrl + Shift + \ : 블록 주석 제거  
Ctrl + / : 여러줄이 한꺼번에 주석처리됨. 주석 해제하려면 반대로 하면 된다.  
Alt + Up(Down) : 위(아래)줄과 바꾸기  
Alt + Shift + 방향키 : 블록 선택하기  
Ctrl + Shift + Space : 메소드의 파라메터 목록 보기  
Ctrl + Shift + O : 자동으로 import 하기  
Ctrl + Shift + F4 : 열린 파일 모두 닫기  
Ctrl + M : 전체화면 토글  
Ctrl + Alt + Up(Down) : 한줄(블럭) 복사  
Ctrl + , or . : 다음 annotation(에러, 워닝, 북마크 가능)으로 점프  
Ctrl + 1 : 퀵 픽스  
F3 : 선언된 변수로 이동, 메소드 정의부로 이동  
Ctrl + T : 하이어라키 팝업창 띄우기(인터페이스 구현 클래스간 이동시 편리)  
Ctrl + O : 메소드나 필드 이동하기  
Ctrl + F6 : 창간 전환, UltraEdit 나 Editplus 의 Ctrl + Tab 과 같은 기능  
<hr>

#### 템플릿 사용
sysout 입력한 후 Ctrl + Space 하면 System.out.println(); 으로 바뀐다.  
try 입력한 후 Ctrl + Space 하면 try-catch 문이 완성된다.  
for 입력한 후 Ctrl + Space 하면 여러가지 for 문을 완성할 수 있다.  
템플릿을 수정하거나 추가하려면 환경설정/자바/편집기/템플릿 에서 할 수 있다.  
<hr>

#### 메소드 쉽게 생성하기
클래스의 멤버를 일단 먼저 생성한다.  
override 메소드를 구현하려면, 소스->메소드대체/구현 에서 해당 메소드를 체크한다.  
기타 클래스의 멤버가 클래스의 오브젝트라면, 소스->위임메소드 생성에서 메소드를 선택한다.  
<hr>

#### 에디터 변환
에디터가 여러 파일을 열어서 작업중일때 Ctrl + F6 키를 누르면 여러파일명이 나오고 F6키를 계속 누르면 아래로 Ctrl + Shift + F6 키를 누르면 위로 커서가 움직인다.  
Ctrl + F7 : 뷰간 전환  
Ctrl + F8 : 퍼스펙티브간 전환  
F12 : 에디터로 포커스 위치  
<hr>

#### 기타 유용한 단축키 목록
Ctrl + / : 주석 처리 – 한 라인/블록에 대해 주석 처리 (추가 및 제거)   
Ctrl + L : 특정 라인으로 이동  
Ctrl + F6 : Editor 창간의 이동  
Ctrl + F7 : View 이동 메뉴  
Ctrl + F8 : Prespectives 이동 메뉴  
Ctrl + D : 한라인 삭제 – 커서가 위치한 라인 전체를 삭제 한다.  
Ctrl + J : Incremental find 이클립스 하단 상태 표시줄에 Incremental find 라고 표시되어 한 글자자씩 누를 때 마다 코드내의 일치하는 문자열로 이동 , 다시 Ctrl + J 를 누르면 그 문자열과 일치 하는 부분을 위/아래 방향키로 탐색이 가능하다.  
Ctrl + N : 새로운 파일 / 프로젝트 생성  
Ctrl + 1 (빠른교정) – 문 맥에 맞게 소스 교정을 도와 준다. 변수를 선언하지 않고 썼을경우 빨간색 에러 표시되는데 이 단축키를 적용하면 변수에 맞는 선언이 추가 되도록 메뉴가 나타난다.  
Ctrl + 0 : 클래스 구조를 트리로 보기  
Ctrl + Space :  Cotent Assist – 소스 구문에서 사용 가능한 메소드, 멤버들의 리스트 메뉴를 보여준다.  
Ctrl + PageUp , Ctrl + PageDown : Edit 창 좌우 이동 – Edit 창이 여러개 띄워져 있을경우 Edit 창간의 이동 한다.  
Ctrl + Shift + Down : 클래스 내에서 다음 멤버로 이동  
Ctrl + Shift + M : 해당 객체의 Import 문을 자동 생성 – import 추가 할 객체에 커서를 위치 시키고 단축키를 누르면 자동적으로 import 문이 생성   
Ctrl + Shift + O : import 문을 자동 생성 – 전체 소스 구문에서 import 안된 클래스의 import 문을 생성해 준다.   
Ctrl + Shift + G : 해당 메서드 / 필드를 쓰이는 곳을 표시 – View 영역에 Search 탭에 해당 메서드 / 필드를 사용하는 클래스를 표시 해준다.  
Alt + Shift + R : Refactoring (이름변경) – Refactoing 으로 전체 소스에서 이름변경에 의한 참조 정보를 변경해 준다.  
F3 : 선언 위치로 이동  
F11 : 디버깅 시작  
F8 : 디버깅 계속  
F6 : 디버깅 한줄씩 실행(step over)  
F5 : 디버깅 한줄씩 실행 함수 내부로 들어감 (step into)  
F12 : Editor 창으로 이동 (Debugging 등 자동적으로 포커스가 이동 됐을경우 편리)  
Alt + Up , Alt + Down : 줄 바꿈 – 해당 라인을 위 / 아래로 이동 시킨다.  
Alt + Shift + S : Source Menu – 소스메뉴 (Import 추가 , Comment 추가 , 각종 Generator 메뉴) 가 나타난다.  
Alt + Shift + Up : 블록설정 – 소스 코드를 블록 단위로 설정해 준다.  
Alt + Shift + Down : 블록해제 – 소스 코드를 블록 단위로 해제한다.  
Alt + Shift + J : 주석 생성 – 해당 메서드/클래스에 대한 주석을 템플릿을 생성해 준다.  
Alt + Shift + Z : Surround With 메뉴 – try / catch 문이나 for , do , while 등을 해당 블록에 감싸주는 메뉴가 나타난다.  
Ctrl + Shift + F : 코드 포맷팅 – 코드 내용을 문법 템플릿에 맞게 포맷팅(들여쓰기) 해준다.  
Ctrl + Alt + Down: 한줄 복사후 아래에 복사 넣기 – Copy&Paste 대체하는 단축키. 커서가 위치한 라인을 복사해 밑줄에 생성해 준다.  
Ctrl + Shift +X : 대문자로 변환  
Ctrl + Shift + Y : 소문자로 변환  
Ctrl + Shift + L : 모든 단축키의 내용을 표시해준다.  
Ctrl + Shift + B : 현재 커서 라인에 Break point 설정  
Ctrl + Shift + T : 클래스 찾기  
