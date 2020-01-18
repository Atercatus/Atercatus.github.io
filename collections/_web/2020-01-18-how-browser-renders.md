---
title: "브라우저의 동작 원리"
excerpt: "브라우저의 동작 원리"
last_modified_at: 2020-01-18T08:06:00
---

## 브라우저의 동작 원리

- [브라우저는 어떻게 동작하는가?](https://d2.naver.com/helloworld/59361)

- [Critical rendering path - Crash course on web performance (Fluent 2013)](https://www.youtube.com/watch?v=PkOBnYxqj3k)

## 학습 노트

이 글은 [Critical rendering path - Crash course on web performance (Fluent 2013)](https://www.youtube.com/watch?v=PkOBnYxqj3k) 강의를 보며 정리한 것입니다.

### 파싱 (DOM and CSSOM)

네트워크 요청을 통해서 HTML 파일을 우선 받아옵니다. 이때 전체파일을 한번에 받지 않고 여러 패킷으로 나누어 받습니다.

HTML 패킷을 받으면 파싱을 수행합니다. 파싱은 Bytes => Characters => Tokens => Nodes => DOM 순으로 이루어 집니다. 이는 점진적으로 이루어집니다.

그 과정에서 `<link>` 태그를 만나게 되면 네트워크 요청을 보냅니다. 그와 동시에 계속 DOM은 생성되고 있습니다. `<link>` 를 통해 CSS 파일을 요청했다고 가정합시다.

DOM 생성이 완료되면 CSS 파일 요청을 기다립니다. HTML 파일과는 다르게 CSS 파일 패킷이 도착해도 점진적으로 파싱이 수행되지 않고 전체 파일이 모두 받아질때 까지 대기하게 됩니다.

CSS 파일이 모두 받아지게 되면 CSSOM을 생성합니다. DOM 과 CSSOM 모두 트리형태를 유지합니다. 둘을 토대로 Render Tree를 생성합니다. CSS 에서 `display: none;` 과 같은 스타일이 적용되면 CSSOM 과 DOM에 해당 엘리먼트가 존재해도 렌더트리에는 존재하지 않습니다.

### Render Tree

렌더트리는 하나만 존재하지 않습니다. 어떠한 엘리먼트들은 다른 엘리먼트와 다르게 구현되어있습니다(예. `<video>`). 하드웨어 자원 가속, GPU 가속과 같은 추가적인 동작이 필요할 수도 있습니다. 이 때문에 별개의 렌더 트리가 존재하고 이 모든 트리들을 동기화 맞춰주는 작업이 필요합니다.

### Performance rules

1. HTML 파싱은 점진적으로 이루어 집니다.

   - 때문에 HTML 파일 전체를 여러 패킷으로 나누어 전송하는 것이 유리합니다.
   - 또한 CSS 파일을 좀 더 빨리 찾을 수 있으므로 더 빠른 렌더트리 생성에 유리합니다.

2. 렌더링은 CSS 에서 멈추게 됩니다.

   - CSS 파일 전체가 다 받아지기 전까지 렌더링은 멈춥니다.

3. `<script>` 태그 역시 HTML 파싱 과정에서 만나게 되면 멈추게 됩니다.

   - JS 코드 내에서 DOM을 조작할 수 있기 때문입니다.
   - JS 파일이 모두 불러와지고 실행되기 전까지 DOM은 생성되지 않습니다.
   - 보통 CSS 파일은 HTML 파일 상단부, JS 파일은 HTML 파일 하단부에 위치합니다.
   - CSS 파일은 빨리 발견하여 빠르게 fetch 하여 렌더링을 빠르게 진행할 수 있게하고 JS 파일은 늦게 발견하여 렌더링이 block 되었을 때, 많은 부분이 이미 렌더링 되어 있게 하기 위해섭니다.
   - 이는 정확한 결과를 보장하지 못합니다.

4. JavaScript 는 CSSOM을 쿼리할 수 있으며 이를 수정할 수 있습니다.

- 이는 또한 CSS 파싱과정을 block 할 수 있음을 의미합니다.

5. JavaScript 는 DOM을 쿼리할 수 있으며 이를 수정할 수 있습니다.

- 이는 DOM 생성을 block 할 수 있습니다.

### async script

```html
<script src="demo_async.js" async></script>
```

- [Definition and Usage](https://www.w3schools.com/tags/att_script_async.asp)

- 파싱과 렌더링 과정이 block 되지 않습니다.

- 다운로드는 수행하나 그 실행은 비동기로 이루어지게 됩니다.

### UX를 위해

사용자에게 주요한 컨텐츠가 포함된 페이지를 보여주는데 1초를 넘기지 않도록 해야합니다. 중요한 부분을 렌더링 하는데 필요한 부분을 우선 요청하고 렌더링이 완료되면 이후 점진적으로 필요한 부분을 load 해야합니다.

큰 CSS 파일을 split 하여 중요한 스타일은 `inline style`을 통해 적용합니다.

scroll 이벤트를 예를 들어서, 매 scroll 이벤트가 발생할 때 함수를 실행하기 보다 콜백함수를 queue에 집어넣은 후 브라우저가 하는 일이 없을 때 이를 실행하도록 합니다.

긴 실행시간을 가진 JS 코드가 있다면 이를 분리하여 한번에 수행하지 않고 나눠서 실행하도록 하여 critical 한 작업이 있다면 도중에 먼저 실행되도록 합니다.
