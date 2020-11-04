---
layout: post
title: "Java에서 파일의 Mime type을 판별하는 방법"
date:   2020-07-21 13:04:19
category: backend-java
tags: [java]
comments: true
draft: false
---

얼마전, 로컬 파일 시스템에 저장되어 있는 파일을 바이너리 형태로 내려주는 REST API를 작성할 일이 있었습니다.
파일을 내려줄 때 `Content-Type` 헤더에 Mime Type을 알맞게 지정해줘야 하는데요. 찾아보니 다음과 같은 방법으로 할 수 있었습니다.

```java
Path filePath = Paths.get("file/save/path");
String fileContentType = Files.probeContentType(path);
```

시스템에 따라 파일 타입을 감지하지 못하는 경우도 있는데요. 제 경우엔 Mac OSX에서 파일 타입을 감지하지 못하는 경우가 있었습니다.
때문에 알아보니 URLConnection을 이용하는 방법도 있었습니다.

```java
URLConnection.guessContentTypeFromName("file/save/path");
```

위 2가지 방법에 기반하여 아래와 같이 유틸 메소드를 작성했습니다.

```java
/**
 * 파일 Mime Type 탐지
 * @param filePath 파일 저장 경로
 * @return 파일 Mime Type
 */
public String detectFileMimeType(String filePath) {
  Path path = Paths.get(filePath);
  String detectedMimeType = null;
  try {
    detectedMimeType = Files.probeContentType(path);
  } catch (IOException e) {
    // ignore
  }
  return detectedMimeType != null ? detectedMimeType : URLConnection.guessContentTypeFromName(filePath);
}
```

상황에 맞춰 사용하면 될 것 같습니다.