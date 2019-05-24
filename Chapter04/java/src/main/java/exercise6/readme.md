## Exeicise 6 - auto-instrumentation

지금까지는 매뉴얼로 직접 trace를 추가했지만, 이미 다양한 오픈소스 프레임워크들인 수작업을 최소화 시키고 boiler-plate 코드를 자동화 하도록 구현되어 있다. 
자세한 사항은 https://github.com/opentracing-contrib/meta 참조 

Spring Framework에서는 아래 Dependency 를 추가하는 것 만으로도 꽤 자세한 사항을 trace할 수 있다. 
예제에서는 기존에 추가했던 모든 instrument code를 삭제하고 pom.xml에 주석처리 되어 있는 아래 dependency를 추가한다. 

```xml
<dependency>
    <groupId>io.opentracing.contrib</groupId>
    <artifactId>opentracing-spring-cloud-starter</artifactId>
    <version>0.1.13</version>
</dependency>
```

```shell
$ ./mvnw spring-boot:run -Dmain.class=exercise6.bigbrother.BBApp
$ ./mvnw spring-boot:run -Dmain.class=exercise6.formatter.FApp
$ ./mvnw spring-boot:run -Dmain.class=exercise6.HelloApp
$ curl http://localhost:8080/sayHello/Gru
Hello, Felonius Gru! Where are the minions?%
```

Jaeger UI를 통해 기존에 했던 내용과 비교해 보자.
추가적인 사항은 p150 참조