---
title: "Add styled components"
excerpt: "Add styled components"
last_modified_at: 2019-11-13T20:00:00
---

# [Advanced Usage of styled-components](https://www.styled-components.com/docs/advanced)

## Theming

styled-components는 wrapper component인 `<ThemeProvider>`를 export 함으로써 full theming 을 지원합니다. 이 컴포넌트 프로바이더는 컨텍스트 API를 통해 자신의 아래에 존재하는 모든 React 컴포넌트에 테마를 제공합니다. 렌더 트리에서 스타일이 지정된 모든 컴포넌트는 깊이가 여러 수준인 경우에도 제공된 테마에 액세스 할 수 있습니다.

> Note: `<ThemeProvider>` returns its children when rendering, so it must only wrap a single child node as it may be used as the root of the `render()` method.

### Function themes

theme prop으로 함수 또한 넘겨줄 수 있습니다. 이 함수는 부모 테마를 매개변수로 받게됩니다. 이 때 부모 테마는 트리 상위에 있는 `<ThemeProvider>`를 의미합니다.

```javascript
// Define our button, but with the use of props.theme this time
const Button = styled.button`
  color: ${props => props.theme.fg};
  border: 2px solid ${props => props.theme.fg};
  background: ${props => props.theme.bg};
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border-radius: 3px;
`;
// Define our `fg` and `bg` on the theme
const theme = {
  fg: "palevioletred",
  bg: "white"
};
// This theme swaps `fg` and `bg`
const invertTheme = ({ fg, bg }) => ({
  fg: bg,
  bg: fg
});
render(
  <ThemeProvider theme={theme}>
    <div>
      <Button>Default Theme</Button>
      <ThemeProvider theme={invertTheme}>
        <Button>Inverted Theme</Button>
      </ThemeProvider>
    </div>
  </ThemeProvider>
);
```

## Server Side Rendering

### Basic API

```javascript
import { renderToString } from "react-dom/server";
import { ServerStyleSheet } from "styled-components";

const sheet = new ServerStyleSheet();
try {
  const html = renderToString(sheet.collectStyles(<YourApp />));
  const styleTags = sheet.getStyleTags(); // or sheet.getStyleElement();
} catch (error) {
  // handle error
  console.error(error);
} finally {
  sheet.seal();
}
```

`collectStyles` 는 사용자의 element를 provider 내부로 감쌉니다.

```javascript
import { renderToString } from "react-dom/server";
import { ServerStyleSheet, StyleSheetManager } from "styled-components";

const sheet = new ServerStyleSheet();
try {
  const html = renderToString(
    <StyleSheetManager sheet={sheet.instance}>
      <YourApp />
    </StyleSheetManager>
  );
  const styleTags = sheet.getStyleTags(); // or sheet.getStyleElement();
} catch (error) {
  // handle error
  console.error(error);
} finally {
  sheet.seal();
}
```

### 프로젝트에 적용

- [commit](https://github.com/connect-foundation/2019-10/commit/ec824442a6dd1525a8945a5c9723519544d6afcf)
- \_app.js

```javascript
import App from "next/app";
import React from "react";
import { ThemeProvider } from "styled-components";

const theme = {
  colors: {
    primary: "red"
  }
};

export default class MyApp extends App {
  render() {
    const { Component, pageProps } = this.props;
    return (
      <ThemeProvider theme={theme}>
        <Component {...pageProps} />
      </ThemeProvider>
    );
  }
}
```

- \_document.js

```javascript
import Document, { Html, Head, Main, NextScript } from "next/document";
import { ServerStyleSheet as StyledComponentsSheet } from "styled-components";
import { ServerStyleSheets as MaterialSheet } from "@material-ui/styles";
import CssBaseline from "@material-ui/core/CssBaseline";

export default class MyDocument extends Document {
  static async getInitialProps(ctx) {
    const styledComponentsSheet = new StyledComponentsSheet();
    const materialSheet = new MaterialSheet();
    const originalRenderPage = ctx.renderPage;

    try {
      ctx.renderPage = () =>
        originalRenderPage({
          enhanceApp: App => props =>
            materialSheet.collect(
              styledComponentsSheet.collectStyles(
                <>
                  <CssBaseline />
                  <App {...props} />
                </>
              )
            )
        });

      const initialProps = await Document.getInitialProps(ctx);
      return {
        ...initialProps,
        styles: (
          <>
            {initialProps.styles}
            {materialSheet.getStyleElement()}
            {styledComponentsSheet.getStyleElement()}
          </>
        )
      };
    } finally {
      styledComponentsSheet.seal();
    }
  }
}
```
