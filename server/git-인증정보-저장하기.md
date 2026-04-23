# git 인증정보 저장하기

우분투 서버에서 git clone 또는 git pull 받을때 마다 인증정보를 입력해야 되서 해결방법을 찾아보았다. 아래와 같이 명령어를 실행하면 ~/.git-credentials에 저장되어 다음부터 안 물어본다.

```shell
git config --global credential.helper store
```
