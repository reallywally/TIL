# 리서치 react crontab parsor

리액트에서 Crontab 표현식의 유효성을 검사하고 표현식 해석, 다음 실행 시간 등 기능을 제공하는 라이브러리를 찾아보자.

## 1. cron-parser

- github: <https://github.com/harrisiirak/cron-parser>
- 설치: npm install cron-parser
- 제공 기능: crontab 표현식 유효성 검사만 제공
- 샘플 코드:

    ```typescript
    import { CronExpressionParser } from "cron-parser";

    const CrontabParseorTest = () => {
    const crontab = "15 14 1 * *";
    const interval = CronExpressionParser.parse(crontab);

    console.log("Next:", interval.next().toISOString());
    console.log(
        "Next 3:",
        interval.take(3).map((date) => date.toString())
    );

    return <></>;
    };

    export default CrontabParseorTest;
    ```

- 간단후기: 표현식 해석이 없어서 아쉬움

## 2. cronstrue

- github: <https://github.com/bradymholt/cRonstrue>
- 설치: npm install cronstrue
- 제공 기능: crontab 표현식 해석
- 샘플 코드:

    ```typescript
    import cronstrue from "cronstrue";
    import "cronstrue/locales/ko";

    const CrontabParseorTest = () => {
    const crontab = "15 14 1 * *";

    console.log(cronstrue.toString(crontab));
    console.log(cronstrue.toString(crontab, { locale: "ko" }));

    return <></>;
    };

    export default CrontabParseorTest;
    ```

  - 간단후기: 표현식 해석만 있어서 아쉽고 한글 번역이 어색함

## 3. react-cron-generator

- github: <https://github.com/sojinantony01/react-cron-generator>
- live demo: <https://sojinantony01.github.io/react-cron-generator/>
- 제공 기능: crontab 표현식을 GUI로 작성 가능
- 샘플코드: 라이브 데모가 있어서 샘플 코드는 작성하지 않음
- 간단 후기: GUI로 제공하는건 좋으나 사용하기 불편함

## 결론

안타깝게도 1개의 라이브러리 모든걸 해결할수는 없었다. 표현식 검사는 cron-parser, 해석은 cronstrue를 사용하자.

## 기타

- 다른 crontab parsor도 많이 있었지만 관리가 안되는 라이브러리는 제외함
- 온라인 crotab parsor: <https://crontab.guru/>
