---
title: "Use React Context effectively"
excerpt: "How to use React Context efectively"
last_modified_at: 2019-10-28T12:06:00
---

## Global State and Dispatch

Reducer를 사용할 때 이를 Context API를 통해 다른 컴포넌트에서 사용하게 할 수 있습니다.

이 때 하나의 컨텍스트에 state와 dispatch를 모두 저장할 경우, dispatch만 필요한 컴포넌트에서 state가 변경될 경우 다시 렌더링 되는 문제가 발생합니다.

따라서 각각의 다른 컨텍스트로 분리하여 제공하는 것이 좋습니다.

- Context.js

```javascript
import React, { createContext, useReducer, useContext } from "react";

const stateContext = createContext();
const { Provider: GlobalStateProvider } = stateContext;
const dispatchContext = createContext();
const { Provider: DispatchProvider } = dispatchContext;
const context = createContext();
const { Provider } = context;

function reducer(state, { type }) {
  console.log("reducer");
  switch (type) {
    case "A":
      return "A";
    case "B":
      return "B";
    default:
      return;
  }
}

function useMyContext() {
  const res = useContext(context);
  console.log(res);
  return res;
}
function useGlobalState() {
  return useContext(stateContext);
}
function useDispatch() {
  return useContext(dispatchContext);
}

function Context(props) {
  const [state, dispatch] = useReducer(reducer);

  return (
    <GlobalStateProvider value={state}>
      <DispatchProvider value={dispatch}>{props.children}</DispatchProvider>
    </GlobalStateProvider>
  );
}

// 합쳐져 있는 경우 state없이 dispatch만 사용하는 컴포넌트에서도 리렌더링이 발생합니다.

// function Context(props) {
//   const [state, dispatch] = useReducer(reducer);
//   const value = { state, dispatch };

//   return <Provider value={value}> {props.children}</Provider>;
// }

export { Context, useMyContext, useGlobalState, useDispatch };
```

- A.js

```javascript
import React from "react";
import { useMyContext, useGlobalState, useDispatch } from "./Context";

function A() {
  // const { state, dispatch } = useMyContext();
  const state = useGlobalState();
  const dispatch = useDispatch();

  console.log("A rerender!");

  return <div onClick={() => dispatch({ type: "A" })}>A: {state}</div>;
}
export default A;
```

- B.js

```javascript
import React from "react";
import { useMyContext, useDispatch, useGlobalState } from "./Context";

function B() {
  // const { state, dispatch } = useMyContext();
  const dispatch = useDispatch();

  console.log("B rerender!");

  return <div onClick={() => dispatch({ type: "B" })}>B</div>;
}

export default B;
```

- App.js

```javascript
import React from "react";
import "./App.css";

import A from "./Components/A";
import B from "./Components/B";
import { Context } from "./Components/Context";

function App() {
  return (
    <Context>
      <div className="App">
        <>
          <A />
          <B />
        </>
      </div>
    </Context>
  );
}

export default App;
```
