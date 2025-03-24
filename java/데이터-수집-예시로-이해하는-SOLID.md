# 데이터 수집을 예시로 이해하는 SOLID

## 너무 유명한 SOLID

객체 지향 설계 5대 원칙이다. 그동안 내가 본 예시는 각각의 원칙은 잘 설명 하였으나 각 항목마다 연결성이 부족했다. 예를들어 SRP는 동물 클래스로 설명하다가 LSP는 SimpleClass 이렇게 설명하다보니 아쉬운 부분이 많았다. 마침 내가 작업한 데이터 수집를 예시로 만들면 5가지 원칙이 잘 연결되는것 같아 정리할겸 작성해 본다.

## 기본적으로

SOLID에서 말하는 의는 간단히 정리하고 시작하자

- SRP(Single responsibility principle): 단일 책임 원칙
- OCP(Open/closed principle): 개방-폐쇄 원칙
- LSP(Liskov substitution principle): 리스코프 치환 원칙
- ISP(Interface segregation principle): 인터페이스 분리 원칙
- DIP(Dependency inversion principle): 의존관계 역전 원칙

## 간단하게 상황을 가정하면

나는 신규 화장품 브랜드 회사에 다니고 있으며, 우리 회사 제품을 구글, 네이버, 카카오 3가지 플랫폼에서 광고를 집행하고 있다. 그런데 광고 집행 레포트를 각 플랫폼마다 접속해서 확인하고 정리 할려고 하니 너무 불편했다. 그래서 모두의 업무 시간을 단축시키기 위해 통합 대시보드를 만들기로 하였고, 나는 각 플랫폼마다 레포트를 수집하려고 한다.

## SRP(Single responsibility principle)

단일 책임 원칙으로 **한 객체는 한개의 책임**을 가져야 한다. 광고 집행 레포트를 수집하는 객체는 레포트 수집하는 역할만 해야한다. 갑자기 관련 데이터 분석 기능을 추가한다면 SRP에 위반한다.

```java
// Bad SRP :(
public class GoogleReportCollector {

    public void collect(){
        .. 레포트 수집
    }

    public void analyisData(){
        .. 레포트 결과 분석
    }
}

// Good SRP :D
public class GoogleReportCollector {
    public void collect(){
        .. 레포트 수집
    }
}
```

간단하게 보면 1개의 객체에서 1가지 책임을 가지고 있으니까 SRP를 지키는 것은 어렵지 않아 보인다. 하지만 실제 개발을 하다보면 복잡한 요구사항과 도메인 지식, 경험의 차이로 객체에 어떤 기능을 얼만큼 넣을지 어려운 상황이 많다.

바로 위에 경우를 다시 보자. 명확하게 GooglreReportCollector라는 이름으로 구글 광고 레포트를 수집하는 객체로 만들었기 때문에 analyisData() 메소드가 SRP에 위반한다고할 뿐이다. 만약 다양한 데이터를 처리하는 의미로 GoogleDataReport라는 객체로 만들었으면 SRP를 위반했다고 말할 수 있을까?

```java
// SRP를 위반했을까?
public class GoogleDataReport {

    public void collect(){
        .. 레포트 수집
    }

    public void analyisData(){
        .. 레포트 결과 분석
    }
}
```

## OCP(Open/closed principle)

가장 유명한 설명으로 확장에는 열려있고, 변경에는 닫혀있어여야 한다는 원리이다. 좀 더 쉽게 풀어서 설명하면 요구사항의 변경이나 추가사항으로 기존 코드에 수정이 없거나 쉽게 확장해서 사용할수 있어야 한다. 데이터 수집 로직을 생각나는대로 만들면 아래처럼 만들 수 있다

```java
public void run(String platform){
    if(platform.equal("GOOGLE"){
        ... 복잡한 구글 데이터 수집 작업
    } else if(platform.equal("NAVER"){
        ... 복잡한 네이버 레포트 수집 작업
    }
}
```

예제 코드는 간단하게 작성 되었지만 데이터 수집 로직은 생각보다 복잡하다. 그리고 이렇게 만들면 플랫폼이 추가될때 마다 복잡한 비즈니스 로직이 `else if` 로 계속 추가되어 run 메소드가 하는 1) 역할이 많아지고 2) 코드가 복잡해 지며 3) 그로인해 유지보수와 변경에 취약해 진다.

그래서 앞서 설명한 **SRP**와 **인터페이스**를 사용해서 문제를 해결할 수 있다.

```java
// 레포트 관련 클래스
public interface ReportCollector {
    void collect();
}

public class GoogleReportCollector implements ReportCollector {
    @Override
    public void collect(){
        .. 복잡한 레포트 수집
    }
}

public class NaverReportCollector implements ReportCollector {
    @Override
    public void collect(){
        .. 복잡한 레포트 수집
    }
}

public class KakaoReportCollector implements ReportCollector {
    @Override
    public void collect(){
        .. 복잡한 레포트 수집
    }
}

// 레포트 수집 로직
public void run(String platform){
    ReportCollector reportCollector = null
  
    if(platform.equal("GOOGLE"){
        GoogleReportCollector reportCollector = new	GoogleReportCollector();
    } else if(platform.equal("NAVER"){
        NaverReportCollector reportCollector = new NaverReportCollector();
    } else if(platform.equal("KAKAO"){
        KakaoReportCollector reportCollector = new KakaoReportCollector();
    } 

    if(reportCollector != null){
        reportCollector.collect();
    }
}
```

이렇게 각 역할별로 클래스를 분리하고 인터페이스를 이용하여 카카오 레포트를 수집을 추가해도 `KakaoReportCollector` 를 생성하고 `else if` 로 KAKAO만 추가하면 된다. 스프링을 이용하면 더 좋은 코드로 개선할 수도 있지만 SOLID 내용에 집중하기 위해 생략한다.

## LSP(Liskov substitution principle)

이름으로 추측이 안되는 원칙이다. 쉽게 설명하면 서브 타입(자식 객체)는 항상 기반 타입(부모 객체)로 교체할수 있어여 한다는 원칙이다.

위에서 설명한 레포트 수집 로직은 LSP를 위반하지 않았다. 이유는 상속을 사용하지 않기 때문이다. 상속을 사용하지 않은 이유는 나중에는 상속 관련하여 정리할 예정이다. 그래서 조금 아쉽지만 이 부분만 다른 내용으로 설명한다.

```java
public class Bird {
    public void fly() {
        System.out.println("flying!!!");
    }
}

public class Penguin extends Bird {
    @Override
    public void fly() {
        throw new UnsupportException("peguins cannot fly");
    }
}
```

위와 같이 `Penguin` 클래스는 `Bird` 클래스를 상속 받았지만 펭귄은 날 수 없기 때문에 부모인 `Bird` 클래스로 대체될 수 없다. 이를 해결하기 위해서는 `fly`를 클래스가 아닌 인터페이스로 만들어 구현하도록 해야한다.

## ISP(Interface segregation principle)

자신이 사용하지 않는 인터페이스는 구현하지 말아야 한다. 네이버 레포트는 API로 데이터를 바로 주는데 구글은 레포트를 생성하고 정상적으로 생성이 완료된 후 수집할 수 있다고 상황을 추가해보자. 만약 ISP를 위배 한다면 이렇게 작성된다

```java
// 레포트 수집 인터페이스
public interface ReportCollector {
    void collect();
    Boolen checkReportGenComplete();
}

public class GoogleReportCollector implements ReportCollector {
    @Override
    public void collect(){
        .. 복잡한 레포트 수집
    }
    @Override
    public Boolen checkReportGenComplete(){
        ... 레포트 생성 확인
    }
}

public class NaverReportCollector implements ReportCollector {
    @Override
    public void collect(){
        .. 복잡한 레포트 수집
    }
    @Override
    public Boolen checkReportGenComplete(){
        System.out.println("None");
    }
}
```

`NaverReportCollector` 는 레포트 생성을 체크할 필요가 없지만 ReportCollector를 구현해야하기 때문에 억지로 텍스트를 출력하는 기능을 추가하였다. 이 경우 ISP를 위반하지 않기 위해서 인터페이스를 분리하면 된다.

```java
// 레포트 수집 인터페이스
public interface ReportCollector {
    void collect();
}
// 레포트 확인 인터페이스
public interface ReportChecker{
  Boolen checkReportGenComplete();
}

public class GoogleReportCollector implements ReportCollector, ReportChecker {
    @Override
    public void collect(){
        .. 복잡한 레포트 수집
    }
    @Override
    public Boolen checkReportGenComplete(){
        ... 레포트 생성 확인
    }
}

public class NaverReportCollector implements ReportCollector {
    @Override
    public void collect(){
        .. 복잡한 레포트 수집
    }
}
```

## DIP(Dependency inversion principle)

의존 역전 원칙으로 **고수준 모듈은 저수준 모듈의 구현에 의존해서는 안 된다.** 라는 설명을 많이 사용한다. 위에 코드 역시 DIP를 위반 하지 않았다. 이유는 **인터페이스를 사용**했기 때문이다. 각 플랫폼별 레포트 수집 클래스는 인터페이스 `ReportCollector` 의 `collect` 메소르를 구현하였기에 레포트 수집 로직이 어떻게 변경되든 상관없이(의존하지 않고) `collect` 만 호출하여 레포트 수집을 할수 있다.

## 내 생각에

핵심 가치 2가지를 요약하면 1) 각 역할에 맞는 기능으로 클래스를 잘 분리하고 2) 적절한 인터페이스 구현만 잘해도 좋은 설계가 되는거 같다.

## 참고자료

- <https://www.nextree.co.kr/p6960/>
- <https://www.youtube.com/watch?v=PDEFSRaeslE&t=629s>
