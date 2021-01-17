---
title: 스프링 MVC 패턴 흐름
date: 2020-07-25 22:01:72
category: spring
draft: false
---

기본적으로 전체 요청에 대한 관리를 servlet이 한다.

1. Dispatcher Servlet: 요청을 받는다.
2. HandlerMapping: 해당 request에 맞는 handler 찾기
3. Interceptor (미들웨어같은거?): handler 실행 전처리, 후처리
4. Handler: 내부 로직 실행 (모델, 뷰 등을 줌) 후 dispatcher servlet
5. Dispatcher servlet 에서 View Resolver: 응답에 필요한 데이터와 view에 맞추어서 view resolver가 view를 생성한다.
6. Dispatcher View: Model / 컨트롤러에 전달하여 응답 생성. 클라이언트에 반환.

### Dispatch Servlet

한 request에 대해서 전체 처리를 담당하는 servlet
Spring MVC에서 제공해주는 친구이다.

### Interceptor Servlet

Handler 전후처리용. HandlerInterceptorAdapter를 상속받아서 구현하면 된다.

### View resolver

View name에 맞는 view를 만들어주는 클래스. 기본적으로 spring 프레임워크에서 제공해준다.
어디서 jsp파일을 가져올 지 그 경로를 설정해주면 알아서 그에 맞춰서 가지고 온다.

어떤 Interceptor와 View resolver를 사용할지 설정은 xml을 통해서 해주면 된다.
