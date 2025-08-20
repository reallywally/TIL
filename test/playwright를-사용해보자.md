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

    test("올바른 계정 정보로 로그인 성공", async ({ page }) => {
    await page
      .locator('input[type="text"], input[id="loginId"]')
      .first()
      .fill(TEST_USERNAME);
    await page
      .locator('input[type="password"], input[name="password"]')
      .first()
      .fill(TEST_PASSWORD);

    await page
      .locator(
        'button[type="submit"], button:has-text("로그인"), button:has-text("Login")',
      )
      .first()
      .click();

    await page.waitForURL("http://localhost:5174/fp/feature", {
      timeout: 10000,
    });

    await expect(page.locator("text=Feature Processing").first()).toBeVisible({
      timeout: 5000,
    });
  });
})
```

## 궁금했던 내용

Q. locator로 엘리먼트를 선택할때 렌더링된 HTML(DOM)으로 해야하는지 React 컴포넌트로 해야하는지  
A. 테스트할 때는 React/MUI 컴포넌트 자체가 아니라, 그것이 렌더링한 HTML 결과물을 기준으로 selector를 해야함

Q. 브라우저를 여러개 테스트하면 각각 동작하는가?  
A. 그렇다. 그래서 등록을 하면 브라우저 수 만큼 등록이 된다. 따라서 특정 속성이 중복 등록될수 없거나, 삭제 처럼 이미 삭제되어 다른 브라우저에서 삭제할 수 없거나 등 주의해야 할점이 있다. 보통은 아래와 같은 패턴으로 중복을 방지하여 CRUD를 테스트 하거나 Playwright의 network mocking 기능으로 가짜 응답을 돌려준다.

```typescript
const title = `테스트 글 - ${test.info().project.name}`;
const title = `테스트 글 - ${Date.now()}`;
```
