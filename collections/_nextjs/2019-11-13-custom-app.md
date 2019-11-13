---
title: "Custom `<App>`"
excerpt: "Custom `<App>`"
last_modified_at: 2019-11-13T20:00:00
---

## Custom `<App>`

Next.js는 `App` component를 페이지를 초기화 하는데 사용합니다. 사용자는 이를 오버라이드 할 수 있고 또한 페이지 초기화 과정을 제어할 수 있습니다. 아래에 명시된 것들을 가능하게 해줍니다.

- 페이지 변화사이의 layout을 지속할 수 있습니다.
- 페이지를 네비게이팅하는 동안 state를 유지할 수 있습니다.
- `componentDidCatch`를 이용하여 커스텀 에러 핸들링이 가능합니다.
- 페이지에 추가적인 데이터를 주입할 수 있습니다. (for example by processing GraphQL queries)

오버라이드 하기 위해선, `./pages/_app.js` 파일을 만들고, 아래와 같이 App class 를 오버라이드 해야합니다.

```javascript
import React from "react";
import App from "next/app";

class MyApp extends App {
  // Only uncomment this method if you have blocking data requirements for
  // every single page in your application. This disables the ability to
  // perform automatic static optimization, causing every page in your app to
  // be server-side rendered.
  //
  // static async getInitialProps(appContext) {
  //   // calls page's `getInitialProps` and fills `appProps.pageProps`
  //   const appProps = await App.getInitialProps(appContext);
  //
  //   return { ...appProps }
  // }

  render() {
    const { Component, pageProps } = this.props;
    return <Component {...pageProps} />;
  }
}

export default MyApp;
```

> Note: 커스텀 `getInitialProps`를 App 에 추가하면 [Automatic Static Optimization](https://nextjs.org/docs/#automatic-static-optimization)이 동작하지 않습니다.

#### Automatic Static Optimization

Next.js는 자동으로 해당 페이지가 blocking data가 필요로 하지 않는 경우 자동적으로 페이지를 prerender합니다. 이 여부는 해당 페이지 내에 `getInitialProps`의 여부로 판단합니다.

`getInitialProps` 가 존재하면, Next.js 는 해당 페이지를 정적으로 최적화하지 않습니다. 대신에 Next.js 는 기본 동작을 사용하여 요청에 따라 요청시 페이지를 렌더링합니다.(서버 측 렌더링을 의미합니다.)

`getInitialProps` 가 존재하지 않으면, Next.js가 자동적으로 정적인 html을 prerendering함으로써 페이지를 정적으로 최적화합니다. prerendering 하는 동안에는 제공할 `query`정보가 존재하지 않기 때문에 router의 `query` 오브젝트는 비어있습니다. 모든 `query` 값은 수화 후 클라이언트 측에 채워집니다.

이 기능을 통해 Next.js는 서버 렌더링 및 정적으로 생성 된 페이지가 모두 포함 된 하이브리드 애플리케이션을 생성 할 수 있습니다. 따라서 Next.js 는 항상 기본적으로 빠른 응용 프로그램을 생성합니다.

## Example

### Using `_app.js` for layout

```javascript
import React from "react";
import App from "next/app";

class Layout extends React.Component {
  render() {
    const { children } = this.props;
    return <div className="layout">{children}</div>;
  }
}

export default class MyApp extends App {
  render() {
    const { Component, pageProps } = this.props;
    return (
      <Layout>
        <Component {...pageProps} />
      </Layout>
    );
  }
}
```
