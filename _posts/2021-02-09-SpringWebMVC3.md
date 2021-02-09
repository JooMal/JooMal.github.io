---
layout: post
title: "Spring Web MVC 3"
date: 2021-02-09
tags: [spring]
category : [study]
comments: false

---



[백기선님의 스프링 웹 MVC]([https://www.inflearn.com/course/%EC%9B%B9-mvc](https://www.inflearn.com/course/웹-mvc))를 수강하며 개인적으로 정리한 자료입니다. 생략된 내용도 많고, 실제 강의에서는 코드를 작성하며 구현과 동작을 보여주시니 깊은 이해를 위해서는 강의를 수강하시기를 적극 추천합니다. 



# 스프링 MVC 구성 요소



#### 디스패처 서블릿이 사용하는 여러 인터페이스

디스패처 서블릿은 초기화 과정에서 여러 인터페이스를 초기화해주게 됩니다. 각각의 인터페이스를 살펴보면,



##### multipartResolver

- 파일 업로드 요청을 처리하는 데 사용하는 인터페이스

인터페이스 구현체 bean이 디스패처 서블릿에 등록되어있다. 디스패처 서블릿이 이 구현체 빈을 갖고 있어야 파일 업로드를 요청할 수 있다. 해당하는 타입은 바이너리 데이터를 부분부분 쪼개서 보내는데, 이를 처리할 수 있는 특별한 로직이 필요하다. 이러한 요청을 multipartResolver를 이용해 사용할 수 있다.

구체적으로는 http servlet request를 multipart http servlet request로 변환해주어 요청이 담고있는 파일을 꺼낼 수 있는 API를 제공한다. 

##### multipartResolver의 구현체

1. CommonsMultipartResolver
2. StandardServletMultipartResolver (-> 스프링부트 default multipartResolver)

기본 디스패처 서블릿에는 아무런 multipartResolver가 등록되어있지않지만, 스프링부트를 사용하면 기본적으로 StandardServletMultipartResolver 멀티파트 리졸버 하나가 들어오게 된다. 그래서 파일 업로드 처리하는 핸들러를 손쉽게 만들 수 있다.



##### localeResolver

- 클라이언트 로케일(지역) 정보 확인

요청이 디스패처서브릿에 들어왔을 때, 요청을 분석하는 단계에서 사용된다. 로케일 리졸버는 클라이언트의 로케일 정보를 확인하는 요청이 어느 지역에서 온건지 확인하는, 지역 정보를 확인하기 위하여 사용한다. 지역에 따라서 지역에 해당하는 적절한 지역언어의 메시지를 출력해줄 수 있다.

로케일 정보를 분석하는 방법도 여러가지가 있는데, 여러개의 구현체가 있을 수 있지만 기본으로 사용되는 구현체는 디스패처 서블릿 기준으로 **AcceptHeader 구현체**이다. 요청에 들어있는 accept-language를 이용해서 영어권인지, 한국어권인지 등등을 판단한다. 이 외에도 Session기반으로 판단하는 SessionLocaleResolver 등도 있다. 



##### ThemeResolver

- css와 script를 바꾼다 (ex. 다크모드)

그 다음은 테마리졸버 ThemeResolver인데, 버튼을 누르면 css와 script가 샥 바뀌는 사이트가 있다. 그런 기능을 구현하는 다양한 방법이 있지만 스프링 MVC는 Theme이라는 기능을 제공해줘서, Theme이라는 키값을 뷰에 전달하고 뷰가 그 키값에 해당하는 적절한 리졸버를 읽어온다. 가령 적절한 css를 읽어와서, 화면의 테마, 전체적인 분위기를 테마에 맞게 바꿀 수 있다. ThemeResolver는 Cookie, Session, Fixed가 있는데 기본값은 Fixed로 사실상 기본적으로는 사용하지 않는 것이다.



##### HandlerMapping

- 요청이 들어왔을 때 그 요청을 표현하는, 처리하는 핸들러를 찾아주는 인터페이스

HandlerMapping은 많이 다뤘으니 간단하게 다룬다. 요청이 들어왔을 때 그 요청을 표현하는, 처리하는 핸들러를 찾아주는 인터페이스이다. 요청을 웹 브라우저에서 웹/앱으로 헬로 라고 보내면 HandlerMapping이 해당 요청을 처리할 핸들러를 찾아준다. 이후 해당 메소드(핸들러)에 대한 정보를 담고있는 핸들러 자체가 리턴이 된다. 이런 걸 해주는게 HandlerMapping이 해주는 일이다. 핸들러매핑이 해주는 일은 기본적으로 두 개가 등록이 된다. 

1. RequestMappingHandler는 애노테이션 기반으로 핸들러를 찾아주는 것
2. BeanNameUrlHandler는 빈 이름을 기반으로, 이 요청에 해당하는 빈이 있는가 없는가를 찾는다. 



##### HandlerAdapter

- 각각의 핸들러를 **실제로 처리**할 수 있는 인터페이스

그다음에 찾아볼 HandlerAdapter는 각각의 핸들러를 **실제로 처리**할 수 있는 인터페이스이다. 스프링 MVC 확장력의 핵심이다. 핸들러를 얼마든지 원하는대로 커스터마이징할 수 있다. 함수형 스타일로 만들고 싶다면 그것도 가능하다. 펑션 스타일로 정의하고, 핸들러 매핑과 핸들러 어댑터를 구현한다면 동작한다. 



##### HandlerExceptionResolver

- 핸들러의 예외 처리를 해주는 인터페이스

애노테이션 기반의 MVC를 사용해보신 분들은 ExceptionHandler라는 애노테이션을 사용해서 예외처리하는 과정을 해보셨을텐데요. 이렇게 핸들러 어댑터나 요청을 처리하는 데에 있어서 예외가 발생하면 익셉션 리졸버가 해당 익센셥을 처리하게 됩니다. 기본적으로 익셉션 리졸버도 여러가지가 등록이 되어 있는데, 우리가 주로 사용하는건 ExceptionHandlerExceptionResolver라는 겁니다. `@ExceptionHandler` 라는 어노테이션을 나중에 많이 사용하게 됩니다.



##### RequestToViewNameTransrator

- 뷰 이름을 추측하는 인터페이스

RequestToViewNameTransrator는, 리턴값인 뷰 이름을 생략해도 요청을 기반으로 판단해줍니다. 뷰 이름을 리턴할 때 안줬는데, 혹은 ModelAndView에 파라미터값으로 viewName을 넘겨주지 않는 경우에는, 스프링은 요청을 갖고 판단을 합니다. 가령 sample로 들어왔으니까, 뷰 이름도 sample이겠지? 하는 기본 전략을 사용합니다. 구현체가 DefaultRequestToViewNameTransrator 하나밖에 없습니다. 요청에서 뷰 이름을 추측할 수 있는 그런 인터페이스였구요.



##### ViewResolver

- 뷰를 찾아내는 인터페이스

ViewResolver는 뷰 이름이 명시적으로 리턴되든, 요청에서 뷰 이름을 추측하던, 뷰 이름이 나오면 해당하는 뷰를 찾아내는 인터페이스이다. 기본적으로는 InternalResourceViewResolver가 등록되어있고, 여러개의 리졸버가 등록이 될 수 있다. 기본값으로는 하나만 등록이 되어 있다. 스프링부트에서는 뷰 리졸버가 좀 더 등록이 되어 있구요. InternalResourceViewResolver는 기본적으로 JSP를 사용하고, 이에 따라 우리가 jsp를 사용할 수 있었습니다.



##### FlashMapManager

- FlashMapManager는 화면에서 리프레시했을 때, **같은 데이터를 또 보내오지 않도록 방지**하기 위한 일종의 패턴 -> url path나 url request parameter 등으로 이미 요청한 내용을 세션에 저장해두고, 중복 전송을 방지하는 듯?

FlashMapManager는 최근에 들어온 인터페이스입니다. 다른 것들은 스프링 초기, 대략 2003년부터 있었습니다. 버전이 명시되어있지 않은 인터페이스는 처음부터 있던 인터페이스입니다. 

redirect를 할 때, 보통 어떤 데이터를 받고, 요청해서 데이터를 받아서 그걸 저장을 하고 redirect를 하게 된다. redirect를 하지 않으면 화면에서 리프레쉬를 할 때 요청 데이터 등이 또 넘어오고, 그걸 매번 처리해주게 된다. FlashMapManager는 화면에서 리프레시했을 때, **같은 데이터를 또 보내오지 않도록 방지**하기 위한 일종의 패턴이다.

post 요청을 받은 다음에는 리다이렉션을 하고, get 요청으로 redirect를 하는 거에요. 그래서 GET해서 VIEW를 보여주는 거죠. 그럼 그 상황에서는 브라우저 리프레시를 해도, 다시 한 번 form 서브미션이 일어나는게 아니라 **get 요청이 다시 한번 가게 되는거**죠. 그런 식으로 form 서브미션 중복을 조금이라도 방지하기 위한 일종의 요청 처리 패턴이다. 



# 스프링 MVC 동작 원리 마무리



- 처음엔 스프링 부트로 프로젝트를 만들어서 web.xml도 없고 그냥 SpringBooTApplication이라는 어노테이션을 사용해 main method를 갖고 있는 클래스를 실행시키고, 컨트롤러와 타임리프를 사용

- 이렇게 동작하는 기반에는 서블릿이!

 결국에는 스프링 mvc, 웹 mvc도 서블릿 기반으로 동작하는 어플리케이션이고, 따라서 서블릿 컨테이너가 필요하다. 그래서 아주 전형적인 서블릿 애플리케이션을 하나 만들어서 스프링 mvc를 한 번 사용해보았습니다. 처음에는 서블릿을 만들어서 실행을 해봤고, 그 서비스가 delegation(위임)하는 post, do get 등을 사용해보았구요. filter와 listner도 보았습니다. 가장 중요했던건 디스패처 서블릿입니다. 이 디스패처 서블릿의 동작원리, 특히 초기화할 때 특정 인터페이스들에 해당하는 빈을 모두 찾아서 전략을 사용하는데, 전략이 없다면 **DispatcherServlet.properties에 정의된 기본전략을 사용**합니다. web.xml에 지금까지는 서블릿을 등록을 했지만, 사실 web.xml없이고 애플리케이션을 만들 수 있어요. 



#### web.xml을 대체하여 자바 코드로 Dispatcher Servlet을 등록하는 법

```java
public class WebApplication implements WebApplicationInitializer {
    @Override
    public void onStartUp(ServletContext servletContext) throws ServletException {
        // 이 안에서 서블릿을 만들어 등록한다.
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.register(WebConfig.calss);
        context.refresh();
        
        DispatcherServlet dispatcherServlet = new DispatcherServlet(context);
        ServletRegistration.Dynamic dynamic = servletContext.addServlet("app", dispatcherServlet); //등록
        app.addMapping("/app/*"); //매핑
    }
}
```

dispatcher servlet을 만들 때 이 어플리케이션 컨텍스트를 넣어준다. 그리고 등록을 해주고, 매핑을 한다. 위처럼 web.xml말고, 자바 파일만을 갖고도 서블릿을 등록해서 사용할 수가 있다. 이 기능은 스프링 3.1, 서블릿 3.0 부터 지원하는 기능입니다. 이는 **스프링부트를 사용하지 않고 Dispatcher Servlet을 사용**하는 방법 중 하나입니다. 이는 <u>톰캣, 웹 애플리케이션 안에 Dispatcher Servlet이 들어가는 형태</u>인거고,

**스프링부트를 사용하는 경우**에는 제일 처음 만들었던 자바 어플리케이션이 동작을 할 때, 임베디드 톰캣이라는 서블릿 컨테이너가 임베디드 형태로(이 어플리케이션에 종속되는 형태로) 구동이 됩니다. 그리고 그 <u>임베디드 톰캣에 디스패처 서블릿을 등록</u>을 해줍니다.

스프링부트는 이처럼 스프링부트가 갖고있는 주관이 있어요. 스프링 기반의 웹 애플리케이션을 구현하는 개발자들은, 이러한 설정이 편리할 것이다라는 주관에 따라서 기본값을 설정해두고 있습니다. 가령 뷰 리졸버도 있을 것이구요. 뷰 리졸버간의 순서, 어떤 것을 기본값으로 쓸지 등등도 개발자들이 편리하도록 설정을 해두고 있습니다. 그러나 스프링 MVC를 직접 사용하는 경우에는, 너무 기초적인 수준의 기본 전략만 가지고 사용을 하거나, 아니면 일일히, 특히 유용한 ViewResolver들을 얼마나 꼼꼼히, 많이 설정을 하느냐에 따라 동작하는 방법이 달라집니다. 스프링부트를 사용하실 때에는 스프링 MVC가 마치 아무것도 안 해도 되는 것처럼 보이지만, 자동설정파일에 의하여 동작을 하는 것입니다.



# 스프링 MVC 빈 설정

이전 첫번째 파트를 학습하신 분들은 이해하시겠지만, 별다른 설정을 하지 않아도 대스패처 서블릿과 properties에 정의되어있는 기본전략을 사용하게 됩니다. 문제는 기본전략을 사용할 때에는 기본적인 클래스들이 갖고 있는 기본값이 적용이 된다는 점입니다. 인터널 리소스 뷰 리졸버의 경우에는 프리픽스와 서픽스를 설정할 수 있지만, 설정되어있지 않은 상태로 사용하게 된다는 것입니다. 원한다면 우리가 직접 setPrefix, setSuffix로 설정을 해서 편리하게 사용할 수 있겠죠. 뷰 리졸버뿐만 아니라 대부분의 빈들이 아래의 코드처럼 기본값만을 사용하고 있습니다.



```java
@Bean
public Handlermapping handlerMapping() {
    RequestMappingHandlerMapping handlerMapping = new RequestMappingHandlerMapping();
    return handlerMapping;
}
```



그런데 어떤 것들을 설정할 수 있는지를 보여드리겠습니다.



```java
@Bean
public HandlerMapping handlerMapping() {
    RequestMappingHandlerMapping handlerMapping = new RequestMappingHandlerMapping();
    handlerMapping.setInterceptors();
    handlerMapping.setOrder(Ordered.HIGHEST_PRECEDENCE);
    return handlerMapping;
}
```

setInterceptors : 핸들러 매핑은 요청이 들어왔을 때 처리할 수 있는 핸들러를 찾아주는 인터페이스였습니다. 핸들러를 찾아서 처리하기 전, 또는 이후에 서블릿 필터와 비슷한 핸들러 인터셉터라는 개념이 있습니다. 이 핸들러 인터셉터를 설정해줄 수가 있습니다. 핸들러 인터셉터는 빈으로 등록될 수 있구요, 그래서 스프링 IoC 컨테이너의 장점을 더욱 활용할 수 있습니다. 이는 나중에 더 자세히 살펴볼 것이구요.

핸들러 매핑에다가 이런 식으로 인터셉터도 설정할 수 있구요. setOrder라는 건 가장 우선순위(order)가 높은 핸들러를 매핑해오고, 찾게되면 바로 리턴하는 것입니다. 오더가 중요한 상황이라면 오더를 설정할 수도 있구요. 가령 Ordered.HIGHEST_PRECEDENCE를 주면 가장 높은 우선순위를 갖게 됩니다. 핸들러 인터셉터에 대해서는 잠시 뒤에 다른 서블릿에서 살펴보겠습니다.

```java
@Bean
public HandlerAdapter handlerAdapter() {
    RequestMapingHandlerAdapter handlerAdapter = new RequestMapingHandlerAdapter();
    handlerAdapter.set
    return handlerAdapter;
}
```

핸들러 매핑뿐만 아니라 핸들러 어댑터도 마찬가지로, 여러 설정을 추가해줄 수 있습니다. 가령 ArgumentResolver를 사용해 원하는 argument를 설정해줄 수 있으며, MessageConveters를 사용해 요청 본문에 들어오는 값을 Message Body로 파라미터에 바인딩해줄 수 있습니다. ResponseBody의 경우에도 MessageConverters를 사용하구요.

이 모든 것들을 일일히 설정을 하기 위해서 빈 설정을 하게 됩니다. 얘네들이 갖고 있는 기본값들도 있을 것입니다. 기본적으로 Message Converter들에 추가되는 컨버터들이 있긴 합니다. 주로 사용하는 stringHttpMessageConverter 역시도 기본적으로 추가되고 있습니다. 요즘은 빈 설정을 직접 하진 않고, 스프링부트가 나오기 이전에도 스프링 MVC에서 제공해주는 조금 더 편한 방법을 사용해왔습니다. 그냥 이런 식으로 일일히 빈을 설정해줄 수 있구나, 정도면 아시면 될 것 같습니다.



# @EnableWebMvc

- 일일히 빈으로 등록하는 방법 대신
- MVC와 같은 애노테이션 기반의 컨트롤러를 사용할 때, 자바 기반의 설정도 가능하고 사용도 편리하도록 스프링에서 지원하는 애노테이션
- 사용법
  1. @Configuration이 있는 파일에 @EnableWebMvc을 주고
  2. WebApplicationContext에 context.setServletContext(servletContext)

일일히 빈으로 등록을 하기보단, 조금 더 자바 기반의 설정에서, 애노테이션 기반의 컨트롤러, 즉 MVC를 사용할 때에 더 편리하도록 스프링이 EnableWebMvc라는 애노테이션을 지원합니다. 이 애노테이션은 Configuration에 같이 주면 되구요. ComponentScan은 있어도 되고, 없어도 됩니다

WebApplicationContext에 context.setServletContext(servletContext)를 하는 이유 : @EnableWebMvc를 사용할 때 서블릿 컨텍스트를 종종 참조함. 서블릿 컨텍스트가 항상 설정이 되어있어야 합니다.

#### 어떻게 가능한지?

1. EnableWebMvc를 해주면
2. DelegatingWebMvcConfiguration을 import
3. DelegatingWebMvcConfiguration가 상속받고 있는 인터페이스에 가보면 핸들러 매핑, 기본적인 Interceptor들 등이 추가되어있다.

#### 직접 Bean 등록하는 것과 무슨 차이가?

지금까지는 Bean을 직접 등록해서 webMVC에 등록하고, 직접 등록하지않은 것은 디스패처 서블릿과 properties에 있는 것을 사용하고 있었습니다. EnableWebMvc를 쓰면 어떻게 설정이 달라질까요? EnableWebMvc 어노테이션을 붙여주고, 이 상태로 톰캣을 구동하면 어떻게 되는지 살펴보겠습니다.

핸들러매핑부터 보면, RequestMappingHandlerMapping의 order(우선순위)가 0으로 되어있습니다. 즉 <u>RequestMappingHandlerMapping으로 먼저 확인하고, BeanNameUrlHandlerMapping으로 그 다음에 확인</u>을 하게 됩니다. 우선순위는 음수부터 양수값이 있는데, 0이면 딱 가운데로 되어있는 것입니다. <u>handlerAdapter에서도 RequestMappingHandlerAdapter가 우선순위가 높습니다</u>. 

어노테이션 기반의 웹 MVC를 먼저 사용한다면, 애초에 RequestMappingHandlerAdapter 위주로 사용하게 되므로 **우선순위가 높은 것만 확인하고 낮은 BeanNameUrlHandlerMapping까지는 확인하지 않을 것이므로 더욱 최적화**가 될 것입니다. handlerMappings, handlerAdapters 모두 RequestMappingHandlerMapping과 RequestMappingHandlerAdapter를 주로 사용하므로 따로 설정을 주어 최적화를 시키는 상황입니다.

동적으로 메세지 컨버터를 추가해주는 것도 가능합니다. EnableWebMvc를 사용하면 좋은게, Delegation 구조(어딘가에 위임하는 식의 구조)로 빈을 불러옵니다. 이에 기존의 빈에, HandlerMapping, Adapter에서 추가하는 설정들이 손쉽게 가능해집니다. **<u>처음부터 핸들러 매핑, 어댑터를 등록해야하는게 아니라, 이 클래스가 등록해주는 매핑, 어댑터에 조금만 더 수정을 하는 식으로 빈 설정을 할 수 있다</u>**는 것입니다. 

