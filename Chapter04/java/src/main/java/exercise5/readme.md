## Exeicise 5 - using baggage

지금까지의 예제에서 사용한 분산 tracing metadata는 
OpenTracing에서는 general purpose(다목적의) ***distributed context propagation***을 지원하기 위한 `baggage`라는 metadata를 지원한다. 이는 프로세스간에 RPC 호출을 넘마들때 마치 context을 여행짐(baggage)안에 넣어 다니는 것 같은 역할을 한다. 

baggage의 가장 장점은 기존 마이크로서비스의 API를 변경하지 않아도 된다는 것이다. 
Jaeger library는 다음과 같은 HTTP Header를 인식힌다. 
***jaeger-baggage: k1=v1, k2=v2...***

예를 들어 다음과 같이 baggae에 값에 따라서 Hello -> Bonjour로 바꾸게 할 수 있다. 

```bash
$ curl -H 'jaeger-baggage: greeting=Bonjour' \
        http://localhost:8080/sayHello/Kevin
Bonjour, Kevin!
```
```java
@GetMapping("/formatGreeting")
    public String formatGreeting(@RequestParam String name, @RequestParam String title,
            @RequestParam String description, HttpServletRequest request) {
        Span span = startServerSpan("/formatGreeting", request);
        try (Scope scope = tracer.scopeManager().activate(span, false)) {
            String greeting = tracer.activeSpan().getBaggageItem("greeting");
            if (greeting == null) {
                greeting = "Hello";
            }

```