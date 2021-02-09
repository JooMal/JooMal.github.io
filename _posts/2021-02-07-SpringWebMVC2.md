---
layout: post
title: "Spring Web MVC 2"
date: 2021-02-08
tags: [spring]
category : [study]
comments: false
---





[백기선님의 스프링 웹 MVC]([https://www.inflearn.com/course/%EC%9B%B9-mvc](https://www.inflearn.com/course/웹-mvc))를 수강하며 개인적으로 정리한 자료입니다. 생략된 내용도 많고, 실제 강의에서는 코드를 작성하며 구현과 동작을 보여주시니 깊은 이해를 위해서는 강의를 수강하시기를 적극 추천합니다.



# 스프링 MVC 연동



지난 시간까지는 ContextLoaderListener를 사용했었는데, 이는 스프링이 제공해주는 IoC 컨테이너를 사용하는 방법이었고 스프링MVC는 아니었다. 서블릿 자체를 직접 구현해서 사용했기 때문이다. 기존에 우리가 만들어둔 서블릿 어플리케이션에 IoC 컨테이너를 연동한 것이었다.

이번 시간엔 프론트 컨트롤러인 DispatcherServlet를 사용해본다. 만들었던 서블릿을 대체하여, RestController를 만들고 GetMapping으로 URI를 매핑해준다. 그렇게 되면, 이제 핸들러 쪽으로 요청을 Dispatch해주고, @RestController라는 어노테이션을 이해하고, return을 http response로 만들어줄 수 있는 DispatcherServlet을 써야 한다. 서블릿을 등록해주는데, servlet-class로 DispatcherServlet을 넣어준다.

```java
<servlet>
    	<servlet-name>app</servlet>
    	<servlet-class>...servlet.DispatcherServlet</servlet>
        <init-param>
            <param-name>contextClass, contextConfigLocation, ...</param-name>
            <param-value>각각의 위치</param-value>
        </init-param>
        ...
</servlet>
```



### Dispatcher Servlet이 대체 무엇인가?

서블릿 컨테이너에서 HTTP 프로토콜을 통해 들어오는 모든 요청을 받아 나누어주는 프론트 컨트롤러이다. 클라이언트로부터의 요청을 **톰캣**과 같은 서블릿 컨테이너가 받는데, 이 앞에서 모든 요청을 우선 처리하는 컨트롤러를 프론트 컨트롤러라 부른다. (스프링에서 이렇게 정의함)

그래서 Dispatcher Servlet이 먼저 요청을 받고, 처리하도록 지정받은 url 패턴을 사용하여 세부 컨트롤러들에게 나누어주는 시스템인 모양이다. 보통 /*.do 와 같은 형태로 url 패턴이 지정된다.

#### 왜 나왔냐?

기존에는 URL 매핑을 위해 web.xml에 온갖 매핑을 다 구겨넣었는데, 이를 dispatcher-servlet이 대신하여 요청을 **핸들링**해준다. 

![image-20210208161131485](C:\Users\Miss Jo\Documents\GitHub\JooMal.github.io\assets\img\image-20210208161131485.png)

모든 요청을 처리한다는 장점이 있지만, 이미지나 HTML 파일을 불러오는 요청, 나아가 js나 css 파일에 대한 요청까지 서블릿이 가로채서 핸들링을 하다보니 느리고 복잡해지는 모양이다. 이에 대한 해결책으로

1. 클라이언트의 요청이 apps의 URL로 접근하면 Dispatcher Servlet이, resources의 URL로 접근하면 다른 놈이 컨트롤을 맡는 방법
2. 필요한 모든 요청을 컨트롤러에 등록해주는 방법

이 있는데, 이중 1의 방법을 직접 URL 매핑을 등록해주며 구현할 수도 있겠지만, Spring이 쉽고 편한 방법을 제공해준다. 바로 `<mvc:resources />`를 이용하여, Dispatcher Servlet에서 **해당 요청에 대한 컨트롤러를 찾을 수 없는 경우에만, 2차적으로 설정된 경로에서 요청하여 자원을 찾는 것**이다. 



# 다시 원래 내용으로

Bean인 Controller는 DispatcherServlet을 만드는 ApplicationContext에 등록이 되어야하고, 서비스는 ContextLoaderListener가 만들어주는 ApplicationContext에 등록이 되어야 한다. 이렇게 Bean을 가려서 사용해주기 위해서는, @ComponentScan에 Filter를 사용하여 excludeFilter로 Controller만 Bean 등록에서 제외해주어야 한다. 

항상 이렇게 계층구조로 만들어주어야하는건 아니고, Config 하라고 전부 처리해주어도 된다. 이렇게 되면 Controller와 Service가 다 하나에 의해 처리되고,모든 빈이 해당 Config에 등록이 된다. 즉 DispatcherServlet을 하나로 사용하게 된다. 이렇게 되면 ContextLoaderListener를 더이상 사용하지 않고, Root WebApplicationContext를 사용하지 않는 방법도 있다. 

DispatcherServlet이 있고, 다른 서블릿에서 ApplicationContext를 써야할 경우에만 저렇게 계층구조를 만들고, 최근에는 그런 경우가 드물다. 그리고 해당 내용은 스프링부트와는 많이 다르고, 이러한 코딩은 서블릿 컨테이너가 먼저 뜨고, 그 다음에 서블릿 컨테이너 안에 등록되는 Web Application, Servlet Application에다가 스프링을 연동하는 방법이다. (DispatcherServlet을 등록한다던가, ContextLoaderListener를 등록한다던가)

반대로 스프링부트는 스프링 부트 어플리케이션이 먼저 뜨고, 그 안에 톰캣이 내장서버로 가동된다. DispatcherServlet을 포함한 이러한 서블릿은 내장 톰캣에 코드로 등록을 하게 된다. 자바 코드로 등록할 수도 있지만, 이건 서블릿 컨텍스트 안에 스프링이 들어가는 구조라면 스프링부트 애플리케이션은 스프링이라는 자바 애플리케이션 안에 톰캣이 들어가는 형태이다. 구조가 상당히 다르다. 우리는 아직 스프링부트로 가지 않은 상태이다. 톰캣 안에 스프링을 넣어둔 형태이다. 스프링 부트에선 스프링 부트 애플리케이션 안에 톰캣이 들어가있는 형태라고 그림을 다르게 그려야 한다. 

아무튼 이번 시간엔 MVC의 핵심인 DispathcerServlet을 스프링과 연동하여 사용하는 방법을 알아보았다. DispatcherServlet 사용을 위해 Controller와 Service단이 연결되는 Component Scan을 분리해줄 수도, 안해줄 수도 있다는 것을 학습했다.



# Dispatcher Servlet

어떻게 Dispatcher Servlet을 사용해야 저렇게 Annotation으로 사용할 수 있는지를 살펴볼 것이다.(근데 이거 따로 설정 안해주고 AppConfig만 해줘도 자동으로 되었던 것 같은데.. 아마 뭔가 깊고 디테일하게 설정하는 방법이 있는 것 같다.)

백기선님의 경우에는 **디버거**를 사용해서 동작을 살펴보실 예정이다. 디버거 모드로 실행을 해두고, 매번 요청이 DispatcherServlet들을 거쳐가는 동작을 살펴볼 수 있다. F8(Step Over:다음 라인으로 나가는 거), F7(Step Into)을 사용하여 디버깅을 진행해볼 수 있다. 

DispatcherServlet의 동작은

	1. 요청을 분석 
 	2. 요청을 처리할 핸들러를 찾는다 -> HandlerMapping을 사용하여 찾는다.
 	3. 해당 핸들러를 실행할 수 있는 핸들러 어댑터를 찾는다.
 	4. 찾아낸 핸들러 어댑터를 사용하여 핸들러의 응답을 처리한다.
     - **핸들러의 리턴값**에 따라 어떻게 응답을 처리할지를 결정한다.
       - RestController : 가령 모든 메서드마다의 ResponseBody를 생략하는 RestController라면 컨버터를 사용해서 HTTP 응답 본문을 만들어주게 된다.
       - 뷰가 있는 경우 : return값을 따라 View 파일을 찾았기 때문에, ModelAndView가 null이 아니라서 해당 View를 렌더링해준다.
 	5. 예외가 발생했다면, 예외처리핸들러에 요청처리를 위임한다.
 	6. 최종적으로 응답을 보낸다.



### View가 있는 경우의 동작

뷰가 있는 경우에는 어떤 식으로 동작할까? 즉, http 본문에 출력할 String 타입 값을 return해주는 것이 아니라, jsp파일의 이름을 String으로 return해주는 경우에는 어떻게 동작하게 될까?  예를 들어 다음과 같다.

```java
@GetMapping("/simple")
public String simple() {
    return "WEB-INF/simple.jsp";
}
```

RestController를 사용한 이전 예시에서는 핸들러 어댑터가 요청을 처리하고 나왔을 때, ModelAndView가 null이었다. 반면 View가 있는 경우에선 ModelAndView가 null이 아니다. 리턴 값에 따라서 뷰를 찾았기 때문에, 핸들러 어댑터는 해당 View를 찾아서 들어가게 된다.



#### 어떻게 ModelAndView가 null이 아닌걸까?

이러한 동작은 어떻게 가능한걸까? 위의 예시를 풀어서 다시 작성해보면 다음과 같다.

```java
@Controller("/simple")
public class SimpleController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) {
        return new ModelAndView("/WEB-INF/simple.jsp");
    }
}
```

위의 코드 형식으로 매핑과 Model을 넘겨주는 return을 해주게 된다. 위처럼 ModelAndView가 만들어져 넘겨지기 때문에 ModelAndView가 null이 아니었던 것이고, 이에 따라 View를 찾아들어가 렌더링할 수 있게된 것이다.



#### 어떻게 핸들러를 찾는걸까?

Web에 Simple이라는 요청을 하면 

1. ResponseServlet이 요청을 받아서 처리를 해주고, Handler를 찾아나선다.
2. getHandler가 handlerMapping으로 핸들러를 찾아온다.
3. HandlerMapping 중 BeanNameUrlHandlerMapping이 Simple Controller라는 핸들러를 찾아낸다.
4. SimpleControllerHandlerAdapter가 응답을 실행(처리)해준다.
   - SimpleControllerHandlerAdapter는 Controller를 implements하는 구현체이다.
5. 응답을 처리할 때, Internal Resource View Resolver가 View이름에 해당하는 실제 리소스를 찾아내준다.
6. 만약 일련의 과정 중에 예외가 발생하면, 등록된 Exception Resolver 중 가장 적절한 리졸버를 찾아서 처리하게 된다.



## Dispatcher Servlet이 제공해주는 Handler Mapping, Handler Adapter, View Resolver는 어떻게 가져올까?



Dispatcher Servlet이 제공하는 Handler Mapping, Handler Adapter, View Resolver 등을 사용하는 컨트롤러 코드를 위에서 작성했다. 이러한 것들은 어떻게 오게 되는걸까?

Dispatcher Servlet을 쭉 타고 들어가다보면 여러 Resolver들을 init해주는 **initStrategies 메서드**가 있다. 그 중 initViewResolver를 보면 **viewResolver 타입의 bean들을 전부 다 찾아와서 뷰 리졸버 목록에 넣어준다**. 빈이 **없다면 기본전략**을 가져온다. 

또, initHandlerMapping을 보면 마찬가지로 우선 Application Context에서 HandlerMapping 타입의 빈을 다 꺼내서 넣어준다. 이때 detectAllHandlerMapping의 기본값이 true인데, true인 경우에는 모든 빈을 모두 꺼내서 살펴보고, false인 경우에는 정확히 이름이 일치하는 bean만을 context에서 getBean해서 꺼내온다. 근데 false로 주고 쓰지는 않는다고 한다. 성능을 지독하게 최적화하게 싶다면 false로 줄 수도 있겠다. 어쨌거나 현재는 bean으로 등록한게 하나도 없으니까 handelerMapping 역시 null이 될거고, 이 경우에도 **기본 전략**을 가져와서 쓰게 된다.

그렇다면 뷰 리졸버를 직접 등록해서 사용해보자. 기본 전략을 가져와서 쓰되, 약간의 설정을 해주어서 더욱 편하게 사용할 수 있다.

```java
@Configuration
@ComponentScan
public class WebConfig {
    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }
}
```

이렇게 하면 다음처럼 코드를 생략해서 사용할 수 있다.

```java
@GetMapping("/simple")
public String simple() {
    return "simple.jsp";
}
```

요청을 하면 initStrategies 메서드가 실행되고(서버를 껐다 켰으니까), detectAllViewResolver 플래그가 true라서 모든 뷰 리졸버를 찾아보게 되고, 우리가 설정해서 만들어준 뷰 리졸버를 찾아서 들어오며 기본 뷰 리졸버 대신 우리의 viewResolver가 등록되게 된다. 

중단점을 다 제거하고, 디스패처 서블릿에서 뷰리졸버 부분만 중단점을 다시 설정해본다.  이미 최초에 init과정에서만 한 번 해주었으므로, **중단점이 걸리지 않는 것**을 확인할 수 있다. 서블릿 생명주기를 보면 init은 한 번만 호출되기 때문이다. (서버 재시작 후 init 작업을 미리 해두는걸 웜업이라고 한다.)



# Ref

[망나니개발자 : Dispatcher-Servlet이란?](https://mangkyu.tistory.com/18)

[스프링 컨테이너에 대한 요약](https://velog.io/@ehdrms2034/Spring-MVC-Application-Context.xml)