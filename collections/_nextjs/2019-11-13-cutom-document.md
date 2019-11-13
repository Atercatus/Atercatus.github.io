---
title: "Custom `<Document>`"
excerpt: "Custom `<Document>`"
last_modified_at: 2019-11-13T20:00:00
---

## Custom `<Document>`

Custom `<Document>`는 보통 사용자의 애플리케이션의 `<html>` 과 `<body>` 태그들을 확장시킬 때 사용합니다. Next.js 페이지들은 document의 주변 markup 정의를 건너뛰기 때문에 이러한 작업이 필요합니다.

이는 사용자에게 css-in-JS 라이브러리들(styled-components or emotion)에 대한 server-side rendering을 지원할 수 있게 합니다.

Custom `<Document>`는 또한 비동기 서버 렌더링 데이터를 표현하는 `getInitialProps` 를 포함합니다.

> Note: `<Document>`의 `getInitialProps` 는 client-side transition이나 페이지가 automatically statically optimized 된 경우에서 호출되지 않습니다.

Custom `<Document>`를 사용하기 위해서 사용자는 반드시 `./pages/_document.js` 파일을 만들어야 합니다. 그리고 `Document` 클래스를 `extends` 해야 합니다.

```javascript
import Document, { Html, Head, Main, NextScript } from "next/document";

class MyDocument extends Document {
  static async getInitialProps(ctx) {
    const initialProps = await Document.getInitialProps(ctx);
    return { ...initialProps };
  }

  render() {
    return (
      <Html>
        <Head />
        <body>
          <Main />
          <NextScript />
        </body>
      </Html>
    );
  }
}

export default MyDocument;
```

`<Html>`, `<Head/>`, `<Main />`, `<NextScript />` 는 모두 페이지가 렌더링 되는데에 필요한 요소들입니다.

> `<Main />` 외부에 위치한 React-component들은 브라우저에 의해서 초기화되지 않습니다. 어플리케이션 로직을 여기에 더하지 마십시오. 만약 모든 페이지에서 공유되어야하는 컴포넌트가 생기면, `<App>` component를 대신 이용하십시오.

`ctx` 객체는 모든 getInitialProps hook 에서 수신된 객체와 동일하며, 하나만 추가됩니다. - `renderPage`(`Funcion`)은 실제 React 렌더링 로직을 실행하는 콜백입니다. 이는 Aphrodite의 `renderStatic`과 같은 서버사이드 렌더링을 도와주기 위한 함수를 decorate하기에 적합합니다.

### Customizing `renerPage`

여러분이 `renderPage`를 사용자 정의해야하는 유일한 이유는 서버 렌더링과 올바르게 작동하도록 응용 프로그램을 랩핑해야하는 css-in-js 라이브러리에서 사용하기위한 것입니다.

```javascript
import Document from "next/document";

class MyDocument extends Document {
  static async getInitialProps(ctx) {
    const originalRenderPage = ctx.renderPage;

    ctx.renderPage = () =>
      originalRenderPage({
        // useful for wrapping the whole react tree
        enhanceApp: App => App,
        // useful for wrapping in a per-page basis
        enhanceComponent: Component => Component
      });

    // Run the parent `getInitialProps` using `ctx` that now includes our custom `renderPage`
    const initialProps = await Document.getInitialProps(ctx);

    return initialProps;
  }
}

export default MyDocument;
```
