# JWT에 대하여

## JWT란

JWT(JSON Web Token)는 인증과 정보를 안전하게 전달하기 위한 토큰 기반 인증 방식이다. 클라이언트가 서버에 요청을 보낼 때 JWT를 포함하면 서버는 이를 검증하여 사용자의 신원을 확인하고 권한을 부여할수 있다.

## JWT의 구성 요소

JWT는 Header.Payload.Signature와 같이 .(점) 단위로 3가지 구성 요소를 가지고 있다. 각각의 요소는 다음과 같다.

### Header(헤더)

토큰 유형과 서명에 어떤 알고리즘을 사용했는지 저장한다.

```json
{
   "typ": "JWT",
   "alg": "HS256"
}
```

### Payload(페이로드)

사용자 정보와 추가적인 claim(클레임)을 포함한다.
클레임이란 데이터 각각의 Key를 의미한다.
아래 예시를 기준으로는 sub, role, iat, exp가 claim이다.

```json
{
  "sub": "user@example.com",
  "role": "USER",
  "iat": 1710115200,
  "exp": 1710122400
}
```

claim이 필수는 아니지만 표준 스펙으로 사용이 권장되며, 많이 사용되는 claim 목록은 다음과 같다.

1. iss(Issuer): 토큰 발급자
2. sub(subject): 토큰 제목
3. aud(Audience): 토큰 대상자
4. exp(Expiration Time): 토큰 만료 시간
5. nbf(Not Before): 토큰 활성 날짜
6. iat(Issued At): 토큰 발급 시간
7. jti(JWT ID): JWT 식별자

그외 필요한 custom claim을 추가해도 무방하지만 JWT를 디코딩하면 누구나 내용을 확인할 수 있기 때문에 주민번호, 비밀번호 등 민감한 정보는 담지 않는것이 좋다.

### Signature(시그니처)

```text
HMACSHA256(
    base64UrlEncode(header) + "." + base64UrlEncode(payload),
    secret
)
```

## 참고 자료

- 클레임 목록: <https://datatracker.ietf.org/doc/html/rfc7519#section-4.1>
- <https://brunch.co.kr/@jinyoungchoi95/1>
- <https://velog.io/@vamos_eon/JWT%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B0%80-1>
