# 고민

- 신규 프로젝트를 하면서 API를 개발하는데 단건 조회 응답과 페이징 응답을 분리할지, 각각의 응답 포맷은 어떻게 할지 고민이 많았음
- 나중에 변경하면 번거로울거 같아서 여러 케이스를 리서칭 후 결정하기로함

## 리서칭 해보자

### 네이버 메일

- 메일 목록 조회 응답

```json
    {
    "currentTime": 1732152818,
    "lastPage": 656,
    "Message": "",
    "pageSize": 45,
    "unreadCount": 25810,
    "mailData": [
        {
            "mailSN": 84889,
            "subject": "쇼핑 주문 내역 확인해 주세요"
            ...
        },
        ...
    ],
    "folderName": "받은메일함",
    "totalCount": 29517,
    "listCount": 45,
    "Result": "OK"
}
```

- 메일 상세 조회 응답

```json
{
    "currentTime": 1732155964,
    "nextMailData": {
        "mailSN": 84919,
        ...
    },
    "threadData": [],
    "Message": "",
    "threadTotalCount": 0,
    "ndriveServer": "mybox.naver.com",
    "emailListFromMail": {},
    "threadUnreadCount": 0,
    "mailInfo": {
        "mailSN": 84925,
        "subject": "쇼핑 주문 내역 확인해 주세요",
        ...
    },
    "attachList": {
        ...
    },
    "prevMailData": null,
    "Result": "OK"
}
```

### 쿠팡 상품 목록

- 상품 조회 목록

```json
{
    "productList": [
        {
            "productTitle": "나이키 덩크 로우",
            "brands": [
                "나이키"
            ],
            "categoryId": 5557,
            ...
        }
    ],
    "cdnUrl": "//image6.coupangcdn.com/image/",
    "dynamicPadding": -85,
    "memberStatusTarget": true,
    "wotTarget": false,
    "winbackTarget": false,
    "loyaltyMember": false
}
```

- 상품 상세 조회

```json

```

### chatGPT

```json

```

### 기타 리서치

```json

```

## 정리

- 네이버는 페이징 정보가 