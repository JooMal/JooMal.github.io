---
layout: post
title: "Spring Web MVC 4"
date: 2021-02-15
tags: [spring]
category : [study]
comments: false

---



[백기선님의 스프링 웹 MVC]([https://www.inflearn.com/course/%EC%9B%B9-mvc](https://www.inflearn.com/course/웹-mvc))를 수강하며 개인적으로 정리한 자료입니다. 생략된 내용도 많고, 실제 강의에서는 코드를 작성하며 구현과 동작을 보여주시니 깊은 이해를 위해서는 강의를 수강하시기를 적극 추천합니다. 



---



### WebMvcConfigurer

- 스프링부트 없이 웹MVC 사용하는 방법

deligation 구조로 되어있는 것은 확장이 편하게 하기 위해서입니다. implements로 WebMvcConfigurer를 상속받으면, 뷰 리졸버 등을 직접 다 구현하지 않으면서도 같은 효과를 받을 수 있습니다. 웹 MVC Configurer가 제공하는 확장 기능 중 configureViewResolvers를 이용해 커스터마이징을 이렇게 해줄 수 있다.

```java
@Override
public void configureViewResolvers(ViewREsolverRegistry registry) {
    registry.jsp("/WEB-INF/", ".jsp");
}
```

앞은 prefix, 뒤는 suffix가 됩니다. 코딩으로 확장할 때에는 이 인터페이스를 가장 자주 사용하게 된다. formatter를 추가하게 된다던가, 인터셉터 등을 추가할 수 있다. 근데 스프링부트를 쓰면 더 쉽게 사용할 수 있다. 대부분의 스프링 개발을 스프링부트를 갖고 하기 때문에, 어디까지가 스프링부트가 제공하는거고, 어디부터가 스프링에서 제공하는건지 파악하기 어려울 수 있다. 경계를 나누어서 생각하는 것도 학습에 많이 도움이 된다. 전부 다 스프링으로 간주해도 크게 지장은 없지만, 실질적으로 어디에서 제공하는 빈인지 알고있어야 기능을 확장하는 데에 좋을 것이다.

