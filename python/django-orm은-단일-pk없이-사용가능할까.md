# Django ORM은 단일 PK없이 사용가능할까?

## 정리

### 상황

* DB설계상 단일 PK없이 복합키로만 설계된 테이블이 있었고 이를 Django ORM으로 관리하려했음
* 그런데 자꾸 PK 관련 오류가 발생하였고 확인해 보니 Django ORM은 반드시 단일 PK가 필요함

### 조치

* 갑자기 단일 PK 컬럼을 추가하자니 전체 DB설계 변경이 필요함
* DB 설계를 바꾸자니 작업이 너무 크고 Native Query만 사용하자니 Django를 쓰는 의미가 없음
* 그래서 Flask로 변경하고 ORM을 사용하지 않음

### 그냥 생각

* DB설계는 내가 한게 아니라서 어쩔수 없었지만 어쨎든 단일 PK는 있는게 좋은거 같음
* 역시 스프링이 좋은거 같음

### 참고자료

* <https://forum.djangoproject.com/t/creating-table-models-for-table-that-does-not-have-a-primary-key/22589/9>
