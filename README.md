## DispacherServlet 
스프링 MVC도 프론트 컨트롤러 패턴으로 구현되어있다.  
스프링 MVC의 프론트 컨트롤러가 바로 디스패처서블릿이다.  
그리고 이 디스패처서블릿이 바로 스프링 MVC의 핵심이다.

### DispacherServlet 등록
- `DispacherServlet`도 부모 클래스에서 `HttpServlet`을 상속 받아서 사용하고, 서블릿으로 동작한다.
- 스프링 부트는 `DispacherServlet`을 서블릿으로 자동으로 등록하면서 모든 경로 (`urlPatterns="/"`)에 대해서 매핑한다.

### DispacherServlet.doDispatch() 간략화 버전
```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
    HttpServletRequest processedRequest = request;
    HandlerExecutionChain mappedHandler = null;
    boolean multipartRequestParsed = false;
    
    // 1. 핸들러 조회
    mappedHandler = getHandler(processedRequest);
    if (mappedHandler == null) {
        noHandlerFound(processedRequest, response);
        return;
    }

    // 2. 핸들러 어댑터 조회 - 핸들러를 처리할 수 있는 어댑터를 조회
    HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
    
    // 3. 핸들러 어댑터 실행 -> 4. 핸들러 어댑터를 통해 핸들러 실행 -> 5. ModelAndView 반환
    mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
    
    processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
    }

private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
                                    HandlerExecutionChain mappedHandler, ModelAndView mv, Exception exception) throws Exception {
    // 뷰 렌더링 호출
    render(mv, request, response);
}

protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
    View view;
    String viewName = mv.getViewName();
    
    // 6. 뷰 리졸버를 통해 뷰 조회, 7. View 반환
    view = resolveViewName(viewName, mv.getModel(), locale, request);
    
    // 8. 뷰 렌더링
    view.render(mv.getModel(), request, response);
}
```

## 핸들러매핑
```java
package com.example.springbasic.web.springmvc.old;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.servlet.ModelAndView;
import org.springframework.web.servlet.mvc.Controller;

@Component("/springmvc/old-controller")
public class OldController implements Controller {
    @Override
    public ModelAndView handleRequest(HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("OldController.handleRequest");
        return null;
    }
}
```

- HandlerMapping
  - 핸들러 매핑에서 이 컨트롤러를 찾을 수 있어야 한다.
  - 예) 스프링 빈의 이름으로 핸들러를 찾을 수 있는 핸들러 매핑이 필요하다.
- HandlerAdapter
  - 핸들러 매핑을 통해서 찾은 핸들러를 실행할 수 있는 핸들러 어댑터가 필요하다.
  - 예) `Controller` 인터페이스를 실행할 수 있는 핸들러 어댑터를 찾고 실행해야한다.

### 스프링부트가 자동 등록하는 핸들러 매핑과 핸들러 어댑터
(실제로는 더 많지만, 중요한 부분 위주로 설명)
- 핸들러매핑
  1. `RequestMappingHandlerMapping`: 애노테이션 기반의 컨트롤러인 `@RequestMapping`에서 사용
  2. `BeanNameUrlHandlerMapping`: 스프링 빈의 이름으로 핸들러를 찾아주는 핸들러 매핑
- 핸들러어댑터
  - `RequestMappingHandlerAdapter`: 애노테이션 기반의 컨트롤러인 `@RequestMapping`에서 사용
  - `HttpRequestHandlerAdapter`: `HttpRequestHandler` 처리
  - `SimpleControllerHandlerAdapter`: `Controller` 인터페이스(애노테이션X, 과거에 사용) 처리

### 실행순서 (OldController 예시)
1. 핸들러 매핑으로 핸들러 조회
   1. `HandlerMapping`을 실행해서 핸들러(`OldController`)를 찾는다.
   2. 이 경우 빈 이름으로 핸들러를 찾아야 하기 때문에 이름 그대로 빈 이름으로 핸들러를 찾아주는 
   `BeanNameUrlHandlerMapping`가 실행에 성공하고 핸들러인 `OldController`를 반환한다.
2. 핸들러 어댑터 조회
   1. `HandlerAdapter`의 `supports()`를 순서대로 호출한다.
   2. `SimpleControllerHandlerAdapter`가 `Controller` 인터페이스를 지원하므로 이 어댑터가 선택된다.
3. 핸들러 어댑터 실행
   1. 디스패쳐서블릿이 조회한 `SimpleControllerHandlerAdapter`를 실행하면서 핸들러 정보도 함께 넘겨준다.
   2. `SimpleControllerHandlerAdapter`는 핸들러인 `OldController`를 내부에서 실행하고, 그 결과를 반환한다.

## 뷰 리졸버
스프링부트가 자동 등록하는 뷰 리졸버
(실제로는 더 많지만, 중요한 부분 위주로 설명)
1. BeanNameViewResolver: 빈 이름으로 뷰를 찾아서 반환한다.
2. InternalResourceViewResolver: JSP 같은 전통적인 뷰를 처리한다.

### new-form 예시

#### 1. 핸들러 어댑터 호출
핸들러 어댑터를 통해 new-form이라는 뷰 이름을 획득한다.

#### 2. ViewResolver 호출 
- new-form 이라는 뷰 이름으로 viewResolver를 순서대로 호출한다.
- `BeanNameViewResolver`는 빈 이름으로 뷰를 찾아서 반환한다. 하지만 빈으로 등록된 뷰를 찾을 수 없다.
- `InternalResourceViewResolver`가 호출된다.

#### 3. InternalResourceViewResolver
이 뷰 리졸버는 InternalResourceView를 반환한다.

#### 4. InternalResourceView
- `InternalResourceView`는 JSP처럼 포워드 `forward()`를 호출해서 처리할 수 있는 경우에 사용한다.

#### 5. view.render()
view.render()가 호출디ㅗ고 InternalResourceView는 forward()를 호출해서 JSP를 실행한다.

