---
layout: post
title:  "이클립스 Maven Local Repository 변경방법"
date:   2018-07-14 01:33:01
category: etc-ide
tags: [Eclipse,Maven]
comments: true
draft: false
---
maven 설치 디렉토리 (maven home) 내 settings.xml 또는 새로운 settings.xml을 작성한 후 <localRepository> 태그 사이에 repository로 이용할 경로를 기술한다.

```xml
<localRepository>C:\app\platform\repository\.m2\customRepository</localRepository>
```

이 후 이클립스에서 Window > Preferences 창의 Maven탭에서 Global 또는 User Settings를 위에서 작성한 settings.xml을 지정해준후

프로젝트 우클릭 > Maven > Update Project 수행 시 설정에 따라 새 repository가 구축된다.
