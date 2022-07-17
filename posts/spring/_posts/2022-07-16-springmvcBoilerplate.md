---
layout: post
title: "Spring webmvc 기반 환경 구축"
tags: [Java]
comments: true
---

## 프로젝트 링크

[https://github.com/FRESH-TUNA/SpringMVC-Boilerplate](https://github.com/FRESH-TUNA/SpringMVC-Boilerplate)

## 동작방식

1. 톰켓을 실행한다
2. 톰켓은 등록된 war 파일 혹은 폴더들을 검사하여 `WebApplicationInitializer` 를 상속한 클래스를 찾는다. 만약 찾으면 해당 클래스의 `onStartup` 메소드를 실행한다.
    
    유저는 `WebApplicationInitializer` 를 상속하여 `onStartup` 를 오버라이딩하여, 스프링 webmvc가 제공하는 DispatcherServlet를 톰켓의 `ServletContext` 에 주입한다.
    

```java
/**
 *  톰캣은 웹 프로젝트를 시작할 때, WebApplicationInitializer 구현 클래스를 찾아서 기본 설정을 하게 만든다.
 */
public class WebApplication implements WebApplicationInitializer {
    /**
     * @param servletContext: (tomcat의 context)
     * @throws ServletException
     */
    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {

        /**
         * 프로젝트의 bean등을 관리해주는 ApplicationContext 생성
         */
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();

        /**
         * tomcat servlet의 정보를 가져오기 위해
         * 사용하는것으로 추정
         * ex
         * @Override
         * public String getApplicationName() {
         *     return (this.servletContext != null ? this.servletContext.getContextPath() : "");
         * }
         */
        context.setServletContext(servletContext);

        /**
         * context에 com.boilerplate.core.WebConfiguration 로드 -> 컴포넌트 스캔 진행
         */
        context.register(WebConfiguration.class);
        context.refresh();

        /**
         * 스프링의 DispatcherServlet를 servletContext의 서블렛에 등록한다.
         * DispatcherServlet에는 아까만든 AnnotationConfigWebApplicationContext 를 주입한다.
         */
        DispatcherServlet dispatcherServlet = new DispatcherServlet(context);
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