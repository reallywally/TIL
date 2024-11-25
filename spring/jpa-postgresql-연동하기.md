# 개요

- jpa와 postgresql을 연동해보자

## 기본 설정

1. 의존성 추가

    ```gradle
    dependencies {
        implementation 'org.postgresql:postgresql:42.6.0'
    }
    ```

2. PostgreSQL 설정

    ```yml
    spring:
      datasource:
        url: jdbc:postgresql://localhost:5432/mydb
        username: myuser
        password: mypassword
        driver-class-name: org.postgresql.Driver
    ```

## 이슈 있던던 내용

### 스키마가 있는데 인식 못함

1. yml 수정

    ```yml
    spring:
      jpa:
        hibernate:
          naming:
            physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl # 스키마명 대문자 인식하도록
    ```

2. @Table 애노테이션 수정

    ```java
    // 쌍따옴표 인식하도록 \"로 입력
    @Table(name = "post", schema = "\"FP_P\"")
    @Entity
    @Getter
    @NoArgsConstructor(access = AccessLevel.PUBLIC)
    public class Post {
        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;

        private String title;
    ...
    ```
