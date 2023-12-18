테스트 도중 LocalDateTime 포맷을 `yyyy-MM-dd HH:mm:ss` 로 받을지 `yyyy-MM-dd'T'HH:mm:ss`로 받을지 고민하다가 찾아본 내용을 정리한다.

### 결론은
`yyyy-MM-dd HH:mm:ss`가 틀렸다고 할수는 없지만 `yyyy-MM-dd'T'HH:mm:ss` 가 선호된다고 한다. 

### `yyyy-MM-dd'T'HH:mm:ss`가 선호되는 이유
1. 값이 비정상적으로 들어올 수 있다.
	아직까지 경험해보진 못했지만 `yyyy-MM-dd HH:mm:ss`는 띄어쓰기 때문에 값이 안넘어오거나 잘못 넘어올 수 있다고 한다. 잘못된 값이 들어올 위험이 있는데 굳이 `yyyy-MM-dd HH:mm:ss`를 써야할 이유는 없다고 생각한다.
2. 표준을 사용하자.
	`yyyy-MM-dd'T'HH:mm:ss`는 ISO 8601 표준이고 많은 JSON API서버에서 표준으로 많이 사용한다.

### `@DateTimeFormat` vs `@JsonFormat`
쉽게 정리하면 다음과 같다.
- 클라이언트 -> 서버: `@DateTimeFormat`
- 서버 -> 클라이언트: `@JsonFormat`

그런데 가끔 두개 중 어떤걸 사용해야 하는지 헷갈릴때가 있다. 이때 `@DateTimeFormat`은 Spring 어노테이션으로 서버에서 받을때 사용하고, `@JsonFormat`은 Jackson 어노테이션으로 JSON으로 내보낼때 사용한다고 이해하면 안헷갈린다.


참고 자료
- https://jojoldu.tistory.com/361