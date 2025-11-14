# HTML을 워드파일로 만들어보자

현재 업무상 quilljs 에디터로 작성한 내용은 워드 파일로 저장하는 작업이 필요했다. qulljs로 작성된 내용은 HTML로 저장되어 결국 HTML 내용을 워드파일로 변경하는 작업이 되었다. 여러 방법이 나오는데 리서칭하면서 경험한 내용을 정리해본다.

## quilljs(에디터)에서 직접 워드파일로 저장

가능하다면 서버를 거치지 않고 프론트에서 처리하고 싶었다. 직접 구현하거나 npm 패키지가 있다. 직접 구현은 GPT씨가 만들어줬는데 코드가 너무 길었다. 어찌어찌 파일로 저장되어도 기본 텍스트를 제외한 다른 부분에서 깨지는 부분이 많았다. 거의 대부분을 커스텀해서 관리해야되는데 현실적으로 어려워 npm 패키지를 알아보았다. 패키지는 quill-to-word 하나 있는데 마지막 커밋이 21년 1월이라서 관리가 잘 안되는 것 같았다. 결국 서버에서 처리하는 방식으로 결정했다.

- quill-to-word: <https://github.com/andrewfhw/quill-to-word>

## pandoc CLI 사용

HTML을 워드파일로 저장할때 대표적으로 나오는 문서 변환기이다. 그래서 HTML파일로 저장 후 java processbuilder로 pandoc를 실행하면 될것 같았다. 원래 가장 걱정 했던 부분은 수식이 잘 들어가는지 여부였다. 그런데 워드파일에서 수식을 넣는것과 동일하게 들어가서 깜딱 놀랬다. 문제는 테이블이었다. 테이블 자체가 안들어 가는건 아닌데 배경, 테두리 등 **스타일이 하나도 적용되지 않았다**. 원인은 pandoc이 CSS를 완벽하게 지원하지 않아 일부 CSS가 누락된것이었다. 다른 부분은 좋았지만 테이블 적용이 안되서 이 방법은 탈락되었다.

- CLI 예시

    ```bash
    pandoc source2.html -o result2.docx
    ```

## libre office CLI 사용

클로드씨가 말해줘서 테스트 했는데 pandoc과 다르게 기본 변환부터 잘 안되었다. 그리고 CLI를 사용하기 위해 libre office를 설치해야 되는데 사용하지도 않는 오피스 프로그램이 설치되는것도 별로였다. 더이상 테스트는 하지 않고 바로 탈락되었다.

## doc4j 사용

작성일 기준 거의 이 방법으로 확정되었다. 기본 변환도 잘 되고 서식, 테이블로 변환이 큰 이슈는 없었다. 코드도 복잡한 구현 없이 docx4j라는 라이브러리를 사용해서 간결하게 해결되었다. 조금 아쉬운 부분이 수식은 그대로 넣을 수 없어 이미지로 변환해서 삽입했다. 그래서 텍스트로 작성된것 처럼 이쁘게 들어가진 않았다. 하지만 GPT씨 기준 수식은 완벽하게 호환시키기가 어려워 이 방법을 많이 사용한다고 한다.

- 의존성

    ```xml
    <!-- docx4j -->
    <dependency>
        <groupId>org.docx4j</groupId>
        <artifactId>docx4j-JAXB-ReferenceImpl</artifactId>
        <version>8.3.14</version>
    </dependency>

    <dependency>
        <groupId>org.docx4j</groupId>
        <artifactId>docx4j-ImportXHTML</artifactId>
        <version>8.3.11</version>
    </dependency>

    <!-- 수식 -->
    <dependency>
        <groupId>org.scilab.forge</groupId>
        <artifactId>jlatexmath</artifactId>
        <version>1.0.7</version>
    </dependency>
    ```

- 샘플 코드

    ```java
    import org.docx4j.openpackaging.packages.WordprocessingMLPackage;
    import org.docx4j.convert.in.xhtml.XHTMLImporterImpl;
    import org.jsoup.Jsoup;
    import org.jsoup.nodes.Document;
    import org.jsoup.nodes.Element;
    import org.jsoup.select.Elements;
    import org.scilab.forge.jlatexmath.TeXFormula;
    import org.scilab.forge.jlatexmath.TeXConstants;

    import javax.imageio.ImageIO;
    import java.awt.*;
    import java.awt.image.BufferedImage;
    import java.io.*;
    import java.util.concurrent.atomic.AtomicInteger;
    /**
     * HTML 파일을 Word 문서로 변환하는 예제
     * Java 8 + docx4j 라이브러리 사용
     */
    public class HtmlToWordConverter {

        public static void main(String[] args) throws Exception {
        String html = loadHtml("source.html"); // Quill HTML을 로드

        // 1️⃣ HTML 파싱
        Document doc = Jsoup.parse(html);
        Elements formulas = doc.select("span.ql-formula");

        AtomicInteger index = new AtomicInteger(1);
            for (Element formulaEl : formulas) {
            String latex = formulaEl.attr("data-value");
            String imgFileName = "C:/rim/formula_" + index.getAndIncrement() + ".png";

            // 2️⃣ LaTeX → PNG 변환
            generateLatexImage(latex, imgFileName);

            // 3️⃣ 수식 span을 <img>로 교체
            Element imgEl = new Element("img");
            imgEl.attr("src", imgFileName);
            imgEl.attr("style", "vertical-align: middle;");
            formulaEl.replaceWith(imgEl);
        }

        doc.outputSettings()
                .syntax(Document.OutputSettings.Syntax.xml)
                .escapeMode(org.jsoup.nodes.Entities.EscapeMode.xhtml);

        // 4️⃣ 수정된 HTML을 Word로 변환
        WordprocessingMLPackage wordMLPackage = WordprocessingMLPackage.createPackage();
        XHTMLImporterImpl importer = new XHTMLImporterImpl(wordMLPackage);
            wordMLPackage.getMainDocumentPart().getContent().addAll(importer.convert(doc.html(), null));

            wordMLPackage.save(new File("QuillWithMath.docx"));
            System.out.println("✅ Word 파일 생성 완료: QuillWithMath.docx");
        }

        private static String loadHtml(String path) throws IOException {
            return new String(java.nio.file.Files.readAllBytes(new File(path).toPath()), "UTF-8");
        }

        private static void generateLatexImage(String latex, String outputPath) throws IOException {
            TeXFormula formula = new TeXFormula(latex);
            BufferedImage image = (BufferedImage) formula.createBufferedImage(
                    TeXConstants.STYLE_DISPLAY, 12, Color.BLACK, Color.WHITE);
            ImageIO.write(image, "png", new File(outputPath));
        }
    }
    ```
