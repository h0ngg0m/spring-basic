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