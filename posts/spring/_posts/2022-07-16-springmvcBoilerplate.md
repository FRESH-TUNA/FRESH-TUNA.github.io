---
layout: post
title: "서블릿 탐구"
tags: [Java]
comments: true
---

## 예시 프로젝트 링크

[https://github.com/FRESH-TUNA/SpringMVC-Boilerplate](https://github.com/FRESH-TUNA/SpringMVC-Boilerplate)

## 서블릿

Servlet은 웹서버에서 요청을 처리하고 응답을 생성하는 자바 언어로 작성된 컴포넌트이다. `javax.servlet`
 , `javax.servlet.http` 패키지를 통해 서블릿을 작성하는것을 돕고 있다.

왜 서블릿이 탄생했을까? 웹 요청과 웹 응답은 웹표준에 의거하여 작성해주는것을 도와준다. 개발자가 비즈니스로직에 집중할수 있도록 도와준다. 만약 서블릿이 없다면 http 패킷의 start line, header, body를 자바로 구현하기 위해 오버헤드가 걸릴것이다.

## 서블릿 컨테이너

서블릿은 단독으로 실행되지 않는다. 서블릿을 실행하는 환경을 제공하는 소프트웨어를 서블릿 컨테이너라고 한다. 톰켓이나 netty가 있다.

## 서블릿 컨텍스트 ( javax.servlet.ServletContext )

서블릿컨테이너와 통신하기 위한 인터페이스(역활)를 의미한다. 톰켓과 스프링 프레임워크를 예시로 들어보겠다.
서블릿컨텍스트는 웹어플리케이션 단위로 생성된다.

- 톰켓을 실행한다
- 톰켓은 등록된 war 파일 혹은 폴더들을 검사하여 `WebApplicationInitializer` 를 상속한 클래스를 찾는다. 만약 찾으면 해당 클래스의 `onStartup` 메소드를 실행한다.
- 유저는 `WebApplicationInitializer` 를 상속하여 `onStartup` 를 오버라이딩하여, 스프링 webmvc가 제공하는 DispatcherServlet를 톰켓의 `ServletContext` 에 주입한다.

```java
/**
 * 톰캣 예제
 * 톰캣은 웹 프로젝트를 시작할 때, WebApplicationInitializer 구현 클래스를 찾아서 기본 설정을 하게 만든다.
 */
public class WebApplication implements WebApplicationInitializer {
    /**
     * @param servletContext: tomcat에서 생성한 서블렛콘텍스트
     * startup 메소드를 통해 서블렛에 대한 설정을 하게 된다.
     * @throws ServletException
     */
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {

        /**
         * 프로젝트의 bean등을 관리해주는 ApplicationContext 생성
         * 어노테이션 기반의 bean을 관리할수 있게 도와준다.
         */
        AnnotationConfigWebApplicationContext annotationConfigWebApplicationContext = new AnnotationConfigWebApplicationContext();

        /**
         * tomcat servlet의 정보를 가져오기 위해
         * 사용하는것으로 추정
         * @Override
         * public String getApplicationName() {
         *     return (this.servletContext != null ? this.servletContext.getContextPath() : "");
         * }
         */
        annotationConfigWebApplicationContext.setServletContext(servletContext);

        /**
         * context에 com.boilerplate.core.WebConfiguration 로드 -> 컴포넌트 스캔 진행
         */
        annotationConfigWebApplicationContext.register(WebConfiguration.class);
        annotationConfigWebApplicationContext.refresh();

        /**
         * 스프링의 DispatcherServlet를 servletContext의 서블렛에 등록한다.
         * DispatcherServlet은 스프링 web이 제공하는 서블릿으로 @Controller 어노테이션과의 편리한 연동을 제공해준다. 
         * DispatcherServlet에는 아까만든 AnnotationConfigWebApplicationContext 를 주입한다.
         * 하나의 어플리케이션은 여러 디스패쳐서블릿으로 구성될수 있다.
         */
        DispatcherServlet dispatcherServlet = new DispatcherServlet(annotationConfigWebApplicationContext);
        ServletRegistration.Dynamic app = servletContext.addServlet("app", dispatcherServlet);
        app.addMapping("/app/*");
    }
}

```

## 주의점

- 패키지를 반드시 설정해야만 했다. (폴더를 만들어서라도) → 안설정하면 오류 발생
- war 빌드시 intelij가 아닌 gradle의 plugin을 사용한다.
- 맥OS 기준 톰캣경로는 다음과 같다
 `/usr/local/opt/tomcat@<버전>/libexec`
- 톰캣에 ApplicationContext 경로는 탑재하는 어플리케이션의 경로가 된다.
예를 들어 `/abc` 면 경로는 <서버url>/abc가 된다.
    
    ![스크린샷 2022-07-17 오후 7.51.29.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b1fce57f-53c2-489d-b1cb-2dea34285d77/스크린샷_2022-07-17_오후_7.51.29.png)