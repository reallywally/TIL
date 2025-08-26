# Playwright를 사용해보자

## Playwright란

E2E(End-to-End) 테스트 자동화 프레임워크이다. 간단히 말해, **사람 대신 웹 브라우저를 조작해 주는 프로그램입**으로 브라우저들을 직접 실행해서 마우스로 클릭하고, 키보드로 글을 입력하고, 화면을 스크롤하는 모든 행동을 코드로 만들 수 있다. 주요 특징은 다음과 같다.

- 멀티 브라우저 지원 (Chromium, Firefox, WebKit)
- 자동 대기(auto-waiting) → DOM 요소가 나타날 때까지 자동으로 기다림
- 테스트 병렬 실행 및 멀티 탭, 멀티 브라우저 컨텍스트 지원
- 네트워크 요청/응답 모킹 가능
- 스크린샷, 비디오 레코딩 지원
  - 코드가 실행되는 과정을 화면 녹화하여 보여줌

## 설치 방법

```shell
npx playwright install
```

## 테스트 코드 예시

```typescript
import { test, expect } from "@playwright/test";

// 로그인 정보를 환경변수 또는 설정 파일로 관리하는 것을 권장
const LOGIN_URL = "http://localhost:5174/login";
const TEST_USERNAME = process.env.TEST_USERNAME || "wally";
const TEST_PASSWORD = process.env.TEST_PASSWORD || "1111";

test.describe("로그인 테스트", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto(LOGIN_URL);
  });

  test("로그인 페이지 로드 확인", async ({ page }) => {
    await expect(page).toHaveTitle("Innorules P4A");

    await expect(
      page.locator('input[type="text"], input[id="loginId"]').first(),
    ).toBeVisible();
    await expect(
      page.locator('input[type="password"], input[id="password"]').first(),
    ).toBeVisible();
  });

  test("잘못된 계정 정보로 로그인 시도", async ({ page }) => {
    await page
      .locator('input[type="text"], input[id="loginId"]')
      .first()
      .fill("wronguser");
    await page
      .locator('input[type="password"], input[name="password"]')
      .first()
      .fill("wrongpass");

    await page.locator('button:has-text("로그인")').first().click();

    await page.waitForTimeout(2000);

    await expect(
      page.locator("text=로그인 ID 또는 비밀번호가 일치하지 않습니다.").first(),
    ).toBeVisible({ timeout: 5000 });
  });
})
```

## 테스트 순서 처리

기본적으로 Playwright는 테스트를 병렬로 돌린다. 그래서 등록 테스트를 하고 등록된걸 삭제하는 테스트를 만들려면 반드시 등록이 완료된 이후에 삭제 테스트가 실행되어야 한다.
순서를 보장하기 위한 방법으로 **test.step**와 **test.describe.serial**가있는데 차이점을 알아보자

## 어떤 내용을 테스트 코드로 작성해야 할까?

처음 작성할때는 백엔드 테스트 코드와 비슷하게 CRUD를 중심으로 테스트 코드를 작성하였다.
그런데 화면 기준으로 테스트 코드를 작성하다보니 메뉴B에서 테스트를 하기 위해서는 메뉴A에서 미리 등록이 필요한 부분이 많은데 CRUD 테스트 코드까지 만들기에는 비효율적인이란 생각이 들었다.
그리고 E2E의 의미가 사용자의 관점에서 애플리케이션의 처음부터 끝까지 전체 흐름을 검증하는건데 CRUD 위주의 테스트는 적합하지 않은것 같았다. 그래서 내부적으로는 **"우리 서비스에서 유저가 반드시 성공적으로 해야 하는 주요 시나리오"**를 정리하여 작성하기로 하였다.

## 기타 궁금했던 내용

Q. locator로 엘리먼트를 선택할때 렌더링된 HTML(DOM)으로 해야하는지 React 컴포넌트로 해야하는지  
A. 테스트할 때는 React/MUI 컴포넌트 자체가 아니라, 그것이 렌더링한 HTML 결과물을 기준으로 selector를 해야함

Q. 브라우저를 여러개 테스트하면 각각 동작하는가?  
A. 그렇다. 그래서 등록을 하면 브라우저 수 만큼 등록이 된다. 따라서 특정 속성이 중복 등록될수 없거나, 삭제 처럼 이미 삭제되어 다른 브라우저에서 삭제할 수 없거나 등 주의해야 할점이 있다. 보통은 아래와 같은 패턴으로 중복을 방지하여 CRUD를 테스트 하거나 Playwright의 network mocking 기능으로 가짜 응답을 돌려준다.

```typescript
const title = `테스트 글 - ${test.info().project.name}`;
const title = `테스트 글 - ${Date.now()}`;
```

Q. locator에 입력하는 셀렉터는 CSS 셀렉터와 동일한가?
A. CSS 셀럭터를 포함하여 더 많은 기능을 제공한다. 대표적으로 text 확인 셀렉터가 있는데 테스트 환경에서 사용하기에 매우 편하다.

```typescript
page.locator('text=클릭하기')
page.locator(':has-text("부분")')
```

## 레퍼런스

- <https://tech.kakaoent.com/front-end/2023/230209-e2e/>
- <https://yozm.wishket.com/magazine/detail/2877/>
<https://www.inflearn.com/community/questions/1369146/%ED%86%B5%ED%95%A9%ED%85%8C%EC%8A%A4%ED%8A%B8-%EC%9D%B8%EC%88%98%ED%85%8C%EC%8A%A4%ED%8A%B8-e2e-%ED%85%8C%EC%8A%A4%ED%8A%B8%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90%EC%9D%B4-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C%EC%9A%94?srsltid=AfmBOopLOGVBuWljJIstm3jMAZ5J9j-pt30TqiDdn2pvGqKXjj7eCQkX>
