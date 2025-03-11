# JWT에 대하여

## JWT란

JWT(JSON Web Token)는 인증과 정보를 안전하게 전달하기 위한 토큰 기반 인증 방식이다. 클라이언트가 서버에 요청을 보낼 때 JWT를 포함하면 서버는 이를 검증하여 사용자의 신원을 확인하고 권한을 부여할수 있다.

## JWT의 구성 요소

JWT는 Header.Payload.Signature 과 같이 .(점) 단위로 3가지 구성 요소를 가지고 있다. 각각의 요소는 다음과 같다.

### Header

토큰 유형과 서명 생에 어떤 알고리즘을 사용했는지 저장한다.

```json
{
   "typ": "JWT",
   "alg": "HS256"
}
```

### Payload

사용자 정보와 추가적인 클레임을 포함

```json
{
  "sub": "user@example.com",
  "role": "USER",
  "iat": 1710115200,
  "exp": 1710122400
}
```

### Signature

```text
HMACSHA256(
    base64UrlEncode(header) + "." + base64UrlEncode(payload),
    secret
)
```

## 참고 자료

- <https://brunch.co.kr/@jinyoungchoi95/1>
- <https://velog.io/@vamos_eon/JWT%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80-%EA%B7%B8%EB%A6%AC%EA%B3%A0-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B0%80-1>
