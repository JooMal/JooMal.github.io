---
layout: post
title: "Java Stream"
date: 2020-12-08
tags: [java]
category : [etc]
comments: true
---

### Java Stream

##### 왜 쓰는가?
- 배열과 컬렉션을 함수형으로 처리
- 간단한 병렬처리(멀티 스레딩), 즉 많은 요소들을 빠르게 처리할 수 있다. (이를 위해서는 parallelStream()을 사용해야한다.)

##### Test Set

```java
ArrayList<string> list = new ArrayList<>(Arrays.asList("Apple", "Banana", "Melon", "Grape", "Strawberry"));
System.out.println(list);
```

---

##### map

```java
list.stream().map(s->s.toUpperCase());
list.stream().map(String::toUpperCase);
```

요소들을 특정 조건에 해당하는 값으로 변환해준다.

##### filter

```java
list.stream().filter(t->t.length() > 5)
```

요소를 특정 기준으로 걸러내줍니다.

##### sorted

```java
list.stream().sorted()
```
리스트의 요소를 정렬합니다.


---

##### 가공한 결과값을 가져오는 joining, toList
- Collectors.joining을 이용해 리스트를 조인의 기준으로 배치할 수 있다. String으로 리턴
- Collectors.toList를 이용해 리스트로 리턴


---
[레퍼런스]
- 자바 스트림 [포스트](https://dpdpwl.tistory.com/81)
- 더 읽으면 좋은 글 : 스트림 메소드 [포스트](https://futurecreator.github.io/2018/08/26/java-8-streams/)
