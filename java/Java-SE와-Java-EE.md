# Java SE와 Java EE

## Java SE(Standard Edition)

- 자바 언어의 기본 기능(JVM, 컬렉션, 람다, 스트림 등)을 제공하는 표준 플랫.
- Java 8, Java 11, Java 17 등은 Java SE의 버전.

## Java EE / Jakarta EE (Enterprise Edition)

- Java SE를 기반으로, 대규모 서버/웹/기업용 애플리케이션 개발을 위한 추가 API와 스펙
- 예: Servlet, JSP, JPA, EJB 등

## Java EE와 Jakarta EE의 차이점

Java EE가 Oracle에서 Eclipse 재단으로 이관된 후의 새 이름으로 Jakarta EE로 변경되었다. 그리고 EE는 9부터 패키지명이 ```javax```에서 ```jakarta```로 변경되었다.

## 호환성

Java SE가 Java 8, Java 11, Java 17와 같이 버전을 가지는것 처럼 JJava EE도 버전이 있다.
EE 플랫폼은 SE 버전과 호환성에 맞게 릴리스 되어 EE7, 8, 9게 증가한다.  
EE별 SE 버전 호환성은 다음과 같다

- Java EE 8 -> Java SE 8
- Jakarta EE 8 -> Java SE 8
- Jakarta EE 9 -> Java SE 8
- Jakarta EE 9.1 -> Java SE 11
- Jakarta EE 10 -> Java SE 11 이상

이는 스프링과도 직접적인 영향이 있다. 스프링6, 스프링 부트 3.3 환경이라면 자바 17 이상, Jakarta EE 9를 필요로 한다. 그래서 만약 어떤 jar를 import 했는데 그 jar에서 Java EE 8을 사용하였다면 스프링은 패키지명을 ```jakarta```로 인식하는 반면 jar에서는 패키지명을 ```javax```로 사용하기 때문에 패키지명이 안맞아서 에러가 발생한다.
