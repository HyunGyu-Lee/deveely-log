---
layout: post
title:  "이클립스 Organize Import 단축키 동작하지 않을때 조치방법"
date:   2018-08-05 21:35:23
category: etc-ide
tags: [Eclipse,Radeon]
comments: true
draft: false
---
이클립스 "Organize Imports" 기능은 클래스에 불필요한 import구문을 제거해주고, 필요한 구문은 자동으로 추가해주는 기능으로 보통 "Ctrl + Shift + O"를 눌러 실행시킨다.   

얼마 전 PC를 새로 맞추고 기쁜 마음으로 이클립스도 새로 깔고 코딩 중 Organize Imports 기능이 작동하지 않는 현상이 발생했다.  
<!--more-->
1. 이클립스 단축키 설정화면으로 이동한다. ( Window > Preferences > General > Keys )  
2. "Organize Imports"를 검색한다.  
3. 검색된 단축키 설정의 Binding에 올바른 단축키가 지정되어있는지, When에 "Editing Java Source" 가 잘 지정되어있는지 확인한 후, 잘못설정된 게 있으면 수정 후 적용한다.  
4. 3번으로도 해결이 안되면, Binding에 "Ctrl + Shift + O"로 바인딩 된 다른 단축키가 있는지 확인해본 후 사용하지 않는다면 Unbind Command 버튼을 통해 단축키에서 해제한다.  

보통 여기까지하면 대부분 고쳐지는듯하다. 근데 여기까지 했는데도 안됐다.....    

그냥 쓰지 말까도 했지만 보통 자주 사용하는 기능이 안되니 너무 답답하여 계속 검색해본 결과  

[관련이슈](https://stackoverflow.com/questions/45256038/eclipse-organize-imports-shortcut-ctrlshifto-is-not-working)가 있었다.

>Please keep in mind that if you are using an AMD GPU your Radeon Driver could block Ctrl+Shift+O which it uses to toggle an ingame-overlay. It can be changed to other keys as follows: Games->global settings->performance monitoring

라데온 계열 그래픽카드를 사용하는 경우 Ctrl + Shift + O 에 이미 바인딩된 단축키가 있어 안돼는것이었다..... 라데온 설정에 들어가서 변경하니 아주아주 잘됐다.
