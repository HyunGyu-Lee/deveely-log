---
layout: post
title:  "Spring Boot Build 시 MyBatis Type Alias 미적용 문제"
date:   2018-12-17 23:39:19
category: backend-spring-framework
tags: [Spring Boot,MyBatis,VFS]
comments: true
draft: false
---
MyBatis를 이용하여 개발하면 주로 resultType에 적어줄 타입에 alias를 적용하여 사용한다.

보통 다음과 같이 SqlSessionFactoryBean을 정의할 때 setTypeAliasesPackage 메소드를 이용하여 정의한다.

```java
SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
sessionFactory.setTypeAliasesPackage("packages for alias");
```
<!--more-->
헌데 위와같은 방식으로 alias를 적용한 후 IDE를 통해 실행시키면 문제없이 잘 실행되는데

배포를 위해 jar로 빌드한 후 실행하자  ClassNotFoundException 이 발생하며 alias된 타입들을 못찾는 문제가 발생했다.

​때문에 Mybatis에서 Alias 처리하는 부분을 분석해보니

```java
public <T> Class<T> resolveAlias(String string) {
    try {
        if (string == null) {
            return null;
        }
        String key = string.toLowerCase(Locale.ENGLISH);
        Class<T> value;
        if (TYPE_ALIASES.containsKey(key)) {
            value = (Class<T>) TYPE_ALIASES.get(key);
        } else {
            value = (Class<T>) Resources.classForName(string);
        }
        return value;
    } catch (ClassNotFoundException e) {
        throw new TypeException("Could not resolve type alias '" + string + "'.  Cause: " + e, e);
    }
}
```
TYPE_ALIASES (HashMap<String, Class<?>> ) 에 등록되어 있으면 해당 Class를 반환하고 그렇지 않으면 클래스로더를 통해 클래스를 로딩한도록 되어있는데, 오류 로그를 통해 확인되는 부분은 일단 IDE에서는 잘만 찾아지는 alias들이 TYPE_ALIASES에 등록이 안되어 있다는 점이었다.

때문에 이번엔 alias를 등록하는 부분을 살펴봤다..

```java
public ResolverUtil<T> find(Test test, String packageName) {
    String path = getPackagePath(packageName);

    try {
        List<String> children = VFS.getInstance().list(path);
        for (String child : children) {
            if (child.endsWith(".class")) {
                addIfMatching(test, child);
            }
        }
    } catch (IOException ioe) {
        log.error("Could not read package: " + packageName, ioe);
    }

    return this;
}
```
파라미터로 전달된 패키지 이하 디렉토리의 모든 리소스를 대상으로 확장자가 class인 파일 중 특정 조건에 부합하는 클래스들을 alias로 등록하도록 구현되어있다. (특정 조건은 지정된 슈퍼타입에 대해 isAssignableFrom로 체크, 지정되어있지 않으면 Object클래스가 슈퍼타입)

그리고, 리소스에 접근할 때 VFS라는 클래스를 통해 특정 경로의 리소스 목록을 얻어오도록 되어있다..

​VFS는 Virtual File System의 약자로 실제 파일 시스템 위, 응용 프로그램 계층에 추상화된 가상 파일시스템을 의미하는 용어인 것 같은데 위 소스코드 문맥상 Mybatis에서 파일 등 시스템 리소스에 접근할 때 이용하는 클래스 인 것 같았다. (조금 더 자세히 조사해볼 필요있음..)

​여튼 위 클래스에 대한 구현체가 아무런 지정을 하지 않으면 DefaultVFS라는 구현체가 사용되는데 IDE 위에서 구동될 때는 target 디렉토리 이하 classes 파일들에 문제없이 접근할 수 있으나 Spring Boot프로젝트 배포한 jar에서는 형태가 달라 classes에 접근이 안되어 alias들이 등록되지 않았던 것이다.

이렇게 원인을 찾은 후 검색을 해보니 역시나.......

```java
SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
sessionFactory.setVfs(SpringBootVFS.class);  // Spring Boot 전용 VFS 사용하도록 지정
sessionFactory.setTypeAliasesPackage("packages for alias");
```
위처럼 Spring Boot 용 VFS 구현체를 지정해주는 아주 간단한 방법으로 해결할 수 있었다.
