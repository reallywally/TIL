# Docker 컨테이너에서 SSHD 권한 에러 해결하기

## 문제 상황

사용하던 도커를 재시작 했더니 잘 되던 포트포워딩이 갑자기 안되었다. 도커의 로그를 보니 아래오 같이 sshd 관련 에러가 있었다.

```plaintext
Starting OpenBSD Secure Shell server sshd 
/run/sshd must be owned by root and not group or world-writable. [fail]
```

## 원인 분석

결론 적인 원인은 SSH 보안 정책 때문입이다.

- 보안 검증: OpenSSH는 보안상의 이유로 /run/sshd 디렉토리의 소유권과 권한을 엄격하게 검사한다.
- 권한 문제: 디렉토리가 root 소유가 아니거나, 그룹/기타 사용자에게 쓰기 권한이 있으면 보안 위험으로 간주한다.

그리고 에러 로그를 따라 컨테이너 안에 /run/sshd를 확인해 보니 파일 권한 777이었다.

```shell
drwxrwxrwx 4 root root   4096 Nov 14  2024 ./
drwxr-xr-x 1 root root   4096 Sep 10 08:52 ../
drwxrwxrwx 1 root root   4096 Nov 14  2024 sshd/
```

## 해결 방법

여러 해결 방법이 있겠지만 가장 간단하고 바로 해결할 수 있게 권한을 755로 변경하였다.

```shell
chmod 755 /run/sshd
```

## 후기

권한이 부족해서 에러가 난 경우는 자주 있었지만 권한이 많아서 에러가 발생한 경우는 처음이었다.
