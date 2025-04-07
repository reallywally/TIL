# api 응답 포맷을 어떻게할까

신규 프로젝트를 하면서 API를 개발하는데 단건 조회 응답과 페이징 응답을 분리할지, 각각의 응답 포맷은 어떻게 할지 고민이 많았다.
그래서 유명한 회사에서 사용하는 응답 형태를 벤치마킹하기로 했다. 그 과정에서 회사별 응답 형태를 정리해 보았다.

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

- 상품 조회 목록 응답

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

- 상품 상세 조회 응답
  - 쿠팡은 로켓 상품 여부에 따라 응답 형태가 달랐다.

```json
{
    "fbtTrackingTagBase": null,
    "fbtTrackingTag": "",
    "fbtItemList": [],
    "abtTrackingTagBase": "rmdId=26b4e...",
    "abtTrackingTag": "rmdId0.0:p209550...",
    "abtItemList": [
        {
            "vendorItemId": 88459220150,
            "vendorId": "A00010028",
            "unitPrice": "1개당 14,300원",
        }
        ...
    ]
}
```

### 요즘IT

- 매거진 목록 조회 응답

```json
{
    "ok": true,
    "data": {
        "count": 243,
        "next": "https://api.wishket.com/yozmit/news/?category=ai&ordering=new&page=2",
        "previous": null,
        "results": [
            {
                "id": 3031,
                "title": "기업용 생성 AI 시장은 어떻게 판을 바꾸고 있을까?",
                "author": {
                    "id": 210,
                    "name": "테크스낵",
                    "author_url_identifier": "Techsnack",
                    "tagline": "작가",
                    "image_url": "https://cdn.wishket.com/news_profiles/282218_20240126_95568dceb5e25442.png"
                },
                "description": "지금까지 ...",
                "date_published": "2025-03-24T14:00:17+09:00",
                "repr_date_published": "1시간 전",
                "featured": 1,
                ...
            },
            ...
        ]
    }
}
```

- 매거진 상세 조회 응답
  - 서버사이드 렌더링 같다.

### 클로드 답변

- 목록 조회 응답

```json
{
  "content": [
    {
      "id": 1,
      "title": "첫 번째 게시글입니다",
      "author": "홍길동",
      "createdAt": "2025-03-23T15:30:45",
      "viewCount": 42,
      "summary": "게시글 내용의 일부분..."
    },
    {
      "id": 2,
      "title": "두 번째 게시글입니다",
      "author": "김철수",
      "createdAt": "2025-03-22T09:15:22",
      "viewCount": 18,
      "summary": "게시글 내용의 일부분..."
    }
    // 더 많은 게시글 항목들...
  ],
  "pageable": {
    "page": 0,
    "size": 10,
    "sort": [
      {
        "property": "createdAt",
        "direction": "DESC"
      }
    ]
  },
  "totalElements": 157,
  "totalPages": 16,
  "first": true,
  "last": false,
  "empty": false,
  "number": 0,
  "size": 10,
  "numberOfElements": 10
}
```

- 상세 조회 응답

```json
{
  "id": 1,
  "title": "첫 번째 게시글입니다",
  "content": "여기에 게시글의 전체 내용이 들어갑니다.",
  "author": {
    "id": 100,
    "name": "홍길동",
    "email": "hong@example.com",
    "profileImage": "https://example.com/images/profile/hong.jpg"
  },
  "createdAt": "2025-03-23T15:30:45",
  "updatedAt": "2025-03-23T16:45:12",
  "viewCount": 42,
  "likeCount": 7,
  "commentCount": 3,
  "tags": ["공지사항", "중요"],
  "attachments": [
    {
      "id": 1001,
      "fileName": "첨부파일.pdf",
      "fileSize": 1024000,
      "contentType": "application/pdf",
      "downloadUrl": "https://example.com/api/files/1001"
    }
  ],
}
```

## 결론

회사마다 포맷과 이름이 너무 달랐다. 그러니 각자 상황에 맞는 포맷으로 정하면 될것같다.
지금은 가장 일반적이고 심플한 형태로 사용하기로 했다.

```json
// 목록 
{
  "page": 1,
  "size": 20,
  "totalCount": 40,
  "totalPage": 2,
  "items": [
    {
      "id": 1,
      "title": "게시글 제목 1",
      "author": "작성자 1",
      "createdAt": "2025-03-24T14:30:00Z"
    },
    {
      "id": 2,
      "title": "게시글 제목 2",
      "author": "작성자 2",
      "createdAt": "2025-03-24T15:00:00Z"
    }
    ...
  ]
}

// 상세
{
  "id": 1,
  "title": "게시글 제목",
  "content": "게시글 본문 내용...",
  "author": "작성자",
  "createdAt": "2025-03-24T14:30:00Z",
  "updatedAt": "2025-03-24T16:45:00Z",
  "viewCount": 42
}
```
