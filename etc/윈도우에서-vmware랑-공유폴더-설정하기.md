# 윈도우에서 vmware랑 공유폴더 설정하기

## 작업 환경

호스트는 윈도우 11, vmware에 설치된 리눅스는 록키 9.4이다. 호스트는 상관 없겠지만 리눅스는 록키 버전, 우분투 등 상세 작업 환경에 따라 명령어가 조금 달라질수 있는점 참고하자.

## 사전 작업

1. c드라이브 밑에 공유할 폴더 생성
2. vmware의 가상머신 setting 메뉴에서 options 탭의 shared folders를 활성화
3. add 버튼을 클릭하여 1번에서 생성한 폴더 선택 후 저장

## 작업 순서

### 1. open-vm-tools 설치

```bash
sudo dnf install -y open-vm-tools open-vm-tools-desktop
```

⚠️ Rocky 9에서는 open-vm-tools-desktop 없으면 hgfs가 안 붙는 경우가 있다고한다.

### 2️. 서비스 시작 & 자동 실행

```bash
sudo systemctl enable vmtoolsd --now

# 상태 확인
systemctl status vmtoolsd
```

### 3. mount 헬퍼 존재 확인

```bash
ls /usr/bin/vmhgfs-fuse

# 정상이라면 아래와 같은 결과가 표시 된다.
/usr/bin/vmhgfs-fuse
```

### 4. hgfs 디렉토리 생성

마운트 시킬 폴더를 만드는 것으로 반드시 ```/mnt/hgfs```일 필요는 없다. 사용하기 편한 위치에 디렉토리를 만들자.

```bash
sudo mkdir -p /mnt/hgfs
```

### 5. 마운트

```bash
sudo vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other
```

이렇게 마운트 하면 사전 작업에 등록했던 공유 폴더가 보인다.

### 6.  자동 마운트

5번까지 작업을 하면 재부팅시에 마운트가 풀려버린다. 재부팅 후에도 마운트가 유지되고 싶으면 추가 설정이 필요하다.

## 기타

### virtualbox vs vmware
