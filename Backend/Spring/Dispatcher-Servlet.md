# Dispatcher-Servlet

[https://mangkyu.tistory.com/18]
Dispatcher-Servlet은 HTTP 프로토콜로 들어오는 모든 요청을 가장 먼저 받아 적합한 컨트롤러에 위임해주는 프론트 컨트롤러(Front Controller)라고 정의할 수 있다.

클라이언트 --요청> Tomcat(Servlet Container) -> Dispatcher-Servlet (Front Controller)
여기서 Front Controller는 주로 서블릿 컨테이너의 제일 앞에서 서버로 들어오는 클라이언트의 모든 요청을 받아서 처리해주는 컨트롤러로써, MVC 구조에서 함꼐 사용되는 디자인 패턴이다.

Dispatcher-Servlet은 web.xml을 대체해서 해당 어플리케이션으로 들어오는 모든 요청을 핸들링해주고 공통 작업을 처리하여 컨트롤러만 구현해두면 Dispatcher-Servlet 가 알아서 적합한 컨트롤러로 위임해주는 구조가 됐다.

Dispatcher-Servlet이 요청을 컨트롤러로 넘겨주는 방식은 효율적이지만, 모든 요청을 처리하기에 이미지나 HTML/CSS/Javascript와 같은 정적 파일에 대한 요청마저 모두 가로채 정적 자원을 불러오지 못하는 상황이 발생했다.
이를 해결하는 방법은

1. 정적 자원 요청과 애플리케이션 요청 분리
2. 애플리케이션 요청을 탐색하고 없으면 정적 자원 요청으로 처리
   두가지 방법이 있다.

1번 방법은 /apps로 api접근을 하면 Dispatcher-Servlet으로 처리하게 냅두고 /resources로 접근하면 그냥 바로 파일에 접근하도록 처리하는 것이다. 하지만 이는 코드의 복잡성을 높히는 방식이다.
두번째 방법은 Dispatcher-Servlet이 요청을 처리할 컨트롤러를 먼저 찾고, 요청에 대한 컨트롤러를 찾을 수 없는 경우에, 2차적으로 설정된 자원 경로를 탐색하여 자원을 탐색하는 것이다. 이는 탐색에도 이로우며, 추후 확장을 용이하게 해준다는 장점이 있다.
![Pasted image 20230929005027](https://github.com/Owonie/TIL/assets/70142275/6aa82ca5-6e98-491d-b88a-f9c73c094575)
Dispatcher-Servlet은 Front Controller이기에 가장 먼저 http 요청을 핸들한다. 그렇기에 Interceptor는 Dispatcher-Servlet와 Controller 사이에 있다.

어느 컨트롤러가 요청을 처리할 수 있는지 식별하는 과정을 HandlerMapping이 해준다.
@Controller방식은 RequestMappingHandlerMapping가 처리한다. 이는 @Controller로 작성된 모든 컨트롤러를 찾고 파싱하여 HashMap으로 <요청 정보, 처리할 대상> 관리한다.
처리할 대상은 HandlerMethod객체로 컨트롤러, 메소드 등을 갖고 있는데, 이는 스프링이 리플렉션을 이용해 요청을 위임하기 때문이다.

요청이 오면 요청 정보를 만들고 HashMap에서 요청을 처리할 대상(HandlerMethod)를 찾은 후에 HandlerExecutionChain으로 감싸서 반환한다. HandlerExecutionChain으로 감싸는 이유는 컨트롤러로 요청을 넘겨주기 전에 처리해야하는 인터셉터 등을 포함하기 위해서다.
중요한건 HandlerMapping에서 위임을 하는 것이 아닌 HashMap에 요청을 처리할 대상(HandlerMethod)를 HandlerExecutionChain(중요한건 반복하자)으로 감싸서 **반환한다**는 것이다. 실제 요청을 컨트롤러에 위임하는 건 핸들러 어댑터가 한다.

Dispatcher-Servlet은 컨트롤러로 요청을 직접 위임하는 것이 아니라 HandlerAdapter를 통해 위임한다. 그 이유는 컨트롤러의 구현 방식이 다양하기 때문이다. 다양한 컨트롤러에 대응하기 위해 HandlerAdapter라는 어댑터 인터페이스를 통해 어댑터 패턴을 적용함으로써 컨트롤러의 구현 방식에 상관없이 요청을 위임할 수 있다.

handler adapter가 컨트롤러로 요청을 위임한 전/후에 공통적인 전/후처리 과정이 필요하다. 대표적으로 인터셉터들을 포함해 요청 시에 @RequestParam, @RequestBody등을 처리하기 위한 ArgumentResolver들과 응답 시에 ResponseEntity의 Body를 Json으로 직렬화 하는 등의 처리를 하는 ReturnValueHandler등이 핸들러 어댑터에서 처리된다. ArgumentResolver등을 통해 파라미터가 준비 되면 리플렉션을 이용해 컨트롤러로 요청을 위임한다. 여기서 중요한 개념은 직렬화는 ReturnValueHandler에서 치라하며, 파라미터가 준비 완료되면 컨트롤러에 요청을 위침할 땐 리플렉션을 이용한다는 사실이다.

조금 더 깊은 내용은 아래에 소스코드와 함께 볼 수 있다.
https://mangkyu.tistory.com/216

![Pasted image 20230929013015](https://github.com/Owonie/TIL/assets/70142275/8789aad0-95ba-44d3-83ee-773063ea7547)
