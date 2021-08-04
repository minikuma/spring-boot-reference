### **Introduce**

---
스프링 프레임워크 5.3.9 문서 기준으로 작성되었습니다.       
(원문) https://docs.spring.io/spring-framework/docs/5.3.9/reference/html/web.html#mvc     

---     

### 1.Spring Web MVC    

---

Spring Web MVC는 서블릿 API 기반의 웹 프레임워크이다. "Spring Web MVC" 공식적인 명칭은 ```spring-webmvc``` 모듈의 이름에서 착안했다. 하지만 "Spring MVC" 라는 이름이 우리에게는 더 친숙하다.          

Spring Web MVC와 더불어 스프링 프레임워크 5.0에서는 "Spring WebFlux"라 불리는 리액티브 스텍 기반의 웹 프레임워크도 소개해 주고 있다. "Spring WebFlux"의 기본 모듈은 ```spring-webflux``` 이다. 이번 섹션에서는 "Spring Web MVC"을 다루고 다음 장에서는 "Spring WebFlux"를 다룰 예정이다.      

---
### 1.1.DispatcherSevlet    
---
다른 여러 웹 프레임워크와 비슷하게 Spring MVC는 메인 ```Sevlet``` 을 가지고 있는 컨트롤러 패턴으로 디자인 되었다. 이것이 바로 ```DispatcherServlet``` 이다. ```DispatcherServlet``` 은 여러 요청을 공유하면서 실제 요청에 대한 처리는 다른 컴포넌트에 위임하는 구조로 되어 있다. 이 구조는 다양한 워크플로우를 지원할 수 있는 유연한 구조이다.        

```DispatcherServlet```을 사용하기 위해서는 ```Sevlet``` 스펙을 준수하는 정의가 필요한다. 보통은 Java Configuration을 사용하는 방법과 ```web.xml``` 파일에 정의하는 방법을 많이 사용한다. 이러한 설정이 필요한 이유는  ```DispatcherServlet``` 은 요청과 맵핑된 정보, view 처리, 예외 치리 등과 같은 여러 처리를 위해 실제 컴포넌트를 찾는 과정이 필요하기 때문이다.       

아래 예제는 Java Configuration으로 ```DispatcherServlet`` 의 정보를 등록 및 초기화하는 예제이다. 해당 정보는 서블릿 컨테이너가 자동으로 감지할 수 있다.     

```java
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) {

        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();
        context.register(AppConfig.class);

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(context);
        ServletRegistration.Dynamic registration = servletContext.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```     

아래는 동일한 기능을 하는 ```web.xml``` 로 설정한 모습이다.     

```xml
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/app-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>

</web-app>
```      

> **(Info)**      
스프링 부트와 같은 경우는 다른 초기화 순서를 가지고 있다. 외부 서블릿 컨테이너 관리에 포함시키는 대신 스프링 부트는 스프링 구성을 활용하여 내장 서블릿 컨테이너에 포함시킨다. ```Filter``` 및 ```Servlet``` 선언은 스프링 구성에 감지되고 서블릿 컨테이너에 등록된다.         

---
### **1.1.1.Context Hierarchy**    
---
```DispatcherServlet``` 에는 ```WebApplicationContext``` 가 있을 거라 예상해 볼 수 있다. ```WebApplicationContext``` 는 ```ApplicationContext``` 을 사용하여 웹 환경에 맞게 확장한 클래스라 보면 된다. ```WebApplicationContext``` 에는 ```SevletContext``` 및 서블릿과 연관된 링크가 있다. 애플리케이션에 접근하기 위해 ```WebApplicationContext``` 를 찾아야 하는데 이것을 조회하기 위해서는 ```RequestContextUtils``` 메소드를 사용해야 한다. 그래서 ```RequestContextUtils``` 메소드를 사용할 수 있도록 ```ServletContext``` 바인딩해 주는 게 필요하다.       

대부분의 애플리케이션은 하나의 ```WebApplicationContext``` 만 있어도 충분하다. 또한 계층적인 컨텍스트 구조도 가능한데, 하나의 루트 ```WebApplicationContext``` 에서 다른 여러 개의 ```DispatcherServlet``` 에도 접근 가능하고 자식 ```WebApplicationContext``` 설정에도 접근 가능하다.       


루트 ```WebApplicationContext``` 에는 멀티 서블릿 환경세서 필요한 data repositories, business service와 같은 Bean 이 설정되어 있다. 이 Bean은 상속 구조로 되어 있어 재 사용이 가능하다.    

아래는 ```WebApplicationContext``` 계층구조 설정이다.    

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[] { RootConfig.class };
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { App1Config.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/app1/*" };
    }
}
```    

> **(Info)**      
애플리케이션 계층 구조가 필요하지 않은 경우에는 ```getServletConfigClasses()``` 에서 ```getRootConfigClasses()``` 와 ```null``` 로 반환할 수 있다.    

동일한 설정을 ```web.xml``` 했을 때는 아래와 같다.     

```xml
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/root-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app1</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/app1-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app1</servlet-name>
        <url-pattern>/app1/*</url-pattern>
    </servlet-mapping>

</web-app>
```     

> **(Info)**      
애플리케이션 컨텍스트 계층이 필요하지 않는 경우에는 루트 컨텍스트만 구성하고 ```contextConfiguration``` 서블릿의 매개변수는 비워 둘 수 있다.      


---

### **1.1.2.Special Bean Types**    
---
```DispatcherServlet``` 은 요청과 그에 따른 응답을 처리하기 위한 특별한 빈 타입을 가지고 있다. "specila bean" 이라고 불린다. "specila bean"은 스프링에서 관리하는 오브젝트 인스턴스이다. 물론 스프링에서 관리한다는 의미는 스프링 프레임워크의 스펙을 잘 준수한 오브젝트임을 의미한다. 사용자가 원하면 해당 기능들을 확장할 수도 있다.     

아래는 ```DispatcherServlet```가 감지할 수 있는 "special bean" 리스트이다.   

* ```HandlerMapping```     
   * 사전/사후 처리를 위해 인터셉터 리스트와 함께 핸들러에 맵핑
   * ```RequestMappingHandlerMapping``` 구현체 (```@RequestMapping```)
   * ```SimpleUrlHandlerMapping``` 구현체 (URL Pattern)    
* ```HandlerAdaptor```      
   * 실제 호출되는 방식과 관계없이 요청에 맵핑된 핸들러를 호출 
* ```HandlerExceptionResolver```     
   * 예외 해결     
* ```ViewResolver```     
   * 핸들러에서 반환되는 문자 기반의 View Name을 렌더링할 실제 View에 맵핑   
* ```ThemeResolver```    
* ```MultipartResolver```      
* ```FlashMapManager```     
---

### **1.1.3.Web-MVC Config**      
---
애플리케이션은 요청을 처리하기 위해 "Special Bean Types" 사용하였다. ```DispatcherServlet``` 은 ```WebApplicationContext``` 의 "Special Bean Types"를 확인한다. 만약 일치하는 게 없다면 기본 타입(DispatcherSevlet.properties)으로 세팅된다.     
(https://github.com/spring-projects/spring-framework/blob/main/spring-webmvc/src/main/resources/org/springframework/web/servlet/DispatcherServlet.properties)     


가장 좋은 상황은 MVC Config로 세팅하는 것이다. 필요한 빈들을 xml 혹은 java 로 정의할 수 있다.     

> **(Info)**    
스프링 부트에선 MVC 환경 설정은 java configuration으로 한다. 많은 편리한 추가 옵션들을 제공해 주고 있다.     

---
### **1.1.4.Sevlet Config**     
---
서블릿 3.0 이상에서는 서블릿 컨테이너 환경 설정을 커스텀할 수 있다. ```web.xml``` 설정과 함께 프로그래밍으로 가능하다.    


```java
import org.springframework.web.WebApplicationInitializer;

public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext container) {
        XmlWebApplicationContext appContext = new XmlWebApplicationContext();
        appContext.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");

        ServletRegistration.Dynamic registration = container.addServlet("dispatcher", new DispatcherServlet(appContext));
        registration.setLoadOnStartup(1);
        registration.addMapping("/");
    }
}
```     

```WebApplicationInitializer``` 는 인터페이스로 자동으로 서블릿 3 컨테이너를 감지할 수 있도록 도와준다. 또한 ```AbstractDispatcherServletInitializer``` 추상 클래스도 ```DispatcherServlet``` 등록을 쉽게 할 수 있도록 도움을 준다.   


      
          
아래는 Java Base Configuration인 경우이다.

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { MyWebConfig.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```     

만약 XML Base Configuration으로 해야 한다면 ```AbstractDispatcherServletInitializer``` 을 상속받아 진행한다.    

```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    @Override
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }

    @Override
    protected WebApplicationContext createServletApplicationContext() {
        XmlWebApplicationContext cxt = new XmlWebApplicationContext();
        cxt.setConfigLocation("/WEB-INF/spring/dispatcher-config.xml");
        return cxt;
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }
}
```     

추가로 ```AbstractDispatcherServletInitializer``` 는 ```Filter``` 기능을 쉽게 추가할 수 있다.     

```java
public class MyWebAppInitializer extends AbstractDispatcherServletInitializer {

    // ...

    @Override
    protected Filter[] getServletFilters() {
        return new Filter[] {
            new HiddenHttpMethodFilter(), new CharacterEncodingFilter() };
    }
}
```       

각각의 필터는 구체적인 타입으로 선언되어야 하며, 선언되 타입들은 ```DispatcherSevlet``` 에 맵핑된다.   

---
