---
title: "Hooks API"
excerpt: "Hooks API"
last_modified_at: 2020-01-24T22:06:00
---

## Hooks API

Hook 은 함수 컴포넌트에서 React state 와 lifecycle features 를 연동(hook into) 할 수 있게 해주는 함수입니다. class 없이 React를 사용할 수 있게 해줍니다.

### useState

```javascript
const [state, setState] = useState(initialState);
```

상태를 가지고 있는 값과 그 상태를 변경할 수 있는 함수를 반환합니다.

컴포넌트가 최초로 렌더링 될 때, `state` 는 `initialState`를 가지고 있습니다.

`setState`의 매개변수로 들어간 값을 통해 `state` 이 갱신됩니다. 새 `state` 값을 받아 컴포넌트 리렌더링을 큐에 등록합니다. 다음 리렌더링을 수행 할 때, 그 갱신이 이루어집니다.

`setState`는 리렌더링 시에 다시 생성되지 않습니다.

값을 갱신할 때 이전의 값이 필요하다면 `setState` 매개변수로 함수를 지정하면됩니다. 함수의 매개변수에는 이전값이 들어갑니다.

```javascript
setState(prevValue => preValue + 1);
```

`useState` 의 매개변수로 지정된 값은 초기값입니다. 이후에 발생하는 렌더링에는 이 값이 무시됩니다. 만약 초기값이 고비용의 계산 결과라면, 초기 렌더링 시에만 실행될 함수를 대신 제공해야합니다.

```javascript
const [state, setState] = useState(() => {
  return highCostFunction();
});
```

다만 이때 함수가 아니라 함수의 실행을 통한 결과값을 반환하도록 한다면, 렌더링 될 때 마다 그 `state` 는 바뀌지 않지만 함수가 매번 호출되므로 조심해야합니다

```javascript
function fn() {
  console.log("fn");
  return 0;
}
const [test, setTest] = useState(fn()); // 렌더링마다 fn 이 출력됩니다.
```

[문서](https://ko.reactjs.org/docs/hooks-reference.html#bailing-out-of-a-state-update) 에 의하면 React는 `Object.is`를 통해 값을 비교하여 현재의 state와 동일한 값으로 갱신하는 경우 React는 자식을 렌더링 하거나 무엇을 실행하는 것을 회피하고 그 처리를 종료한다고 되어있습니다.

하지만 그 다음 단락을 보시면 아래와 같은 내용이 있습니다.

> "실행을 회피하기 전에 React에서 특정 컴포넌트를 다시 렌더링하는 것이 여전히 필요할 수도 있다는 것에 주의하세요. React가 불필요하게 트리에 그 이상으로 더 깊게는 관려하지 않을 것이므로 크게 신경쓰지 않으셔도 됩니다만, 만약 렌더링 시에 고비용의 계산을 하고 있다면 `useMemo`를 사용하여 그것들을 최적화 할 수 있습니다."

이게 무슨 뜻인지 실험을 하기 위해 아래와 같은 예제를 실행해보았습니다.

```javascript
import React, { useState } from "react";

const Child = () => {
  console.log("child");

  return <div>Child</div>;
};

const obj = { count: 0 };

export const Counter = () => {
  const [count, setCount] = useState({ count: 0 });
  console.log("Rendering");

  return (
    <div>
      <button
        onClick={() => {
          obj.count = obj.count + 1;
          console.log(obj);
          setCount(obj);
        }}
      >
        {count.count * Math.random()}
      </button>
      <Child />
    </div>
  );
};
```

처음 렌더링이 되고 버튼을 계속해서 누르게 되면 아래와 같은 결과가 나옵니다. `<click>`은 버튼 클릭이벤트를 나타냅니다.

```javascript
Rendering
child
<click>
{count:1}
Rendering
child
<click>
{count:2}
Rendering
<click>
{count:3}
<click>
{count:4}
```

처음 버튼을 누르게되면 count가 갱신되므로 `Counter` 컴포넌트 `Child` 컴포넌트 둘 다 리렌더링됩니다. 버튼의 값도 0 에서 다른 값으로 변경됩니다. 그다음 버튼을 누르게되면 그 obj.count의 값은 변경이 되었지만 `Object.is`를 통해 이전값과 새로운 값을 비교해 보았을때, `obj`로 그 값이 동일하므로 버튼의 값은 변경되지 않습니다. 하지만 Rendering 이라는 값이 출력됩니다. 자식은 렌더링 되지 않습니다. 다시한번 버튼을 누르게 되면 `count`만 출력이되고 Rendering은 출력 되지 않습니다.

이를 통해 정확히 그 내부원리는 알 수 없습니다만, 추측할 수 있는 것은

1. `state`가 동일한 값으로 갱신되려하면 View에는 반영이 되지 않지만 함수 컴포넌트 내부의 동작을 수행합니다.
2. 하지만 그 자식은 렌더링되지 않습니다.

따라서 렌더링될 때 만약 고비용의 동작이 수행된다면 이를 최적화 해줄 필요가 있습니다. 현재 등록된 [이슈](https://github.com/facebook/react/issues/14994)를 보시면 이전값과 현재 갱신될 값이 같다면 값을 변경하는 함수 자체를 실행하지 않도록 하는 방법이 존재합니다.

위의 내용은 렌더링시의 고비용의 연산이 들어가고 그 최적화가 필요한 경우에만 신경쓰면 됩니다. 그 외에는 크게 고민할 필요없다고 합니다.

#### 참조

- [useState not bailing out when state does not change](https://github.com/facebook/react/issues/14994)
- [React hooks useState setValue still rerender one more time when value is equal](https://stackoverflow.com/questions/57652176/react-hooks-usestate-setvalue-still-rerender-one-more-time-when-value-is-equal)
- [What are the differences when re-rendering after state was set with Hooks compared to the class-based approach?](https://stackoverflow.com/questions/55373878/what-are-the-differences-when-re-rendering-after-state-was-set-with-hooks-compar)
- [Why is my component rendering when useState is called with the same state?](https://stackoverflow.com/questions/57850595/why-is-my-component-rendering-when-usestate-is-called-with-the-same-state)

---

### useEffect

```javascript
useEffect(update);
```

side effect 즉, 부작용을 일으키는 함수를 매개변수로 받습니다.

여기서 말하는 부작용은 React 컴포넌트 안에서 데이터를 가져오거나, 구독, DOM 조작, 변형, 타이머, 로깅 등을 말합니다. 이들은 함수 컴포넌트 본문 안에서 작성되는 것이 허용되지 않습니다. 왜냐하면 예상치못한 버그와 UI와 데이터의 불일치를 불러오기 때문입니다.

`useEffect`는 clean-up 함수를 반환할 수 있습니다. 이는 UI에서 컴포넌트가 제거되기전에 수행됩니다. 따라서 여러번 렌더링 시에는 다음 effect가 수행되지 전에 이전의 effect는 clean-up 함수에 의해서 정리됩니다.

- Example

```javascript
useEffect(() => {
  const interval = setInterval(() => {}, 500);

  return () => clearInterval(interval);
});
```

기본적으로 `useEffect`는 reflow 와 repaint 가 수행된 이후에 발생합니다. 이러한 side effect 는 브라우저가 화면을 그리는 것을 blocking 해서는 안되기 때문입니다. 또한 React는 새로운 갱신을 하기 앞서 이전에 수행해야할 렌더링을 항상 완료하므로 이후에 발생할 렌더링 이전에 `useEffect`가 수행 될 것을 보장합니다.

그러나 사용자에게 보여지는 DOM 변경은 화면과 불일치를 피하기 위해서 그리기를 수행하기 이전에 동기화 되어야 합니다. 이러한 경우에는 `useLayoutEffect`를 사용해야합니다.

`useEffect` 에는 두번째 매개변수로 dependency 배열을 줄 수 있습니다. 배열 내의 값이 변경될 때마다 `useEffect` 내의 함수가 실행됩니다. 빈 배열시 최초 한번만 실행되게 됩니다. 이 때, 주어진 dependency 들은 첫번째 인자로 주어진 effect 함수의 인자로 전달되지는 않습니다.

---

### useContext

```javascript
const value = useContext(MyContext);
```

`React.createContext`로 생성된 context 객체를 매개변수로 받아 그 context 가 현재 가지고 있는 값을 반환하는 Hook 입니다. 이 때 반환하는 값은 호출한 함수와 가장 가까이에 있는 Provider의 `value` prop 에 의해 결정됩니다. context 를 만들 때 사용된 `defaultValue`는 트리 안에서 적절한 Provider를 찾지 못할 때 사용됩니다.

- Example

```javascript
import React, { createContext, useContext } from "react";

const Context = createContext(3);

const Consumer = () => {
  const context = useContext(Context);

  console.log(context);

  return <div>{context}</div>;
};

const Provider = () => {
  return <Consumer />; // 3 출력
};

const Provider = () => {
  return (
    <Context.Provider value={5}>
      <Consumer /> // 5 출력
    </Context.Provider>
  );
};

export { Provider };
```

컴포넌트에서 가장 가까운 Provider가 갱신되면 가장 최신의 context value를 이용하여 렌더러를 트리거합니다. 상위 컴포넌트에서 `React.memo` 또는 `shouldComponentUpdate`를 사용하더라도 `useContext`를 사용하고 있는 컴포넌트에서부터 리렌더링됩니다.

---

### useReducer

```javascript
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

`useState` 의 대체 함수로써 많은 매개변수가 필요한 복잡한 로직을 수행하는 경우에 선호됩니다. 또는 하나의 state 에 대해 여러 컴포넌트에서 이에 대한 변경을 수행하는 데, 그 동작이 각각 다르고 그 로직이 복잡한 경우에 유용합니다.

첫 번째 매개변수에는 상태를 변경하는 함수를 받습니다. 이를 `reducer`라 부릅니다. `(state, action) => newState` 의 형태로 `state` 는 이전의 `state`를, `action`은 매개변수로 주어진 값을 의미합니다.

`dispatch` 함수는 리렌더링 시에 다시 생성되지 않습니다. 따라서 `useEffect` 나 `useCallback` 의 의존성 목록에 포함하지 않아도 됩니다.

`useReducer` 의 두 번째 매개변수로 초기 상태를 넘길 수 있습니다. 그 초기화를 지연 시키기 위해서는 세 번째 매개변수로 초기화 함수를 넘겨야 합니다.

```javascript
import React, { useReducer } from "react";

const init = initialValue => {
  return { count: initialValue };
};

const reducer = (state, action) => {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    case "reset":
      return init(action.payload);
  }
};

export const Counter = () => {
  const [state, dispatch] = useReducer(reducer, 5, init);

  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: "reset", payload: 3 })}>
        Reset
      </button>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
    </>
  );
};
```

---

### useCallback

```javascript
const callback = useCallback(() => {
  work(dependency);
}, [dependency]);
```

메모이제이션 된 콜백을 반환합니다.

두 번째 매개변수로 주어진 의존성이 변경 되었을 때에만 새로운 버전의 콜백을 반환합니다.

---

### useMemo

```javascript
const value = useMemo(() => compute(dependency), [dependency]);
```

메모이제이션된 값을 반환합니다.

매개변수로 생성함수와 의존성 배열이 필요합니다. `useCallback` 과 같이 의존성이 변경되었을 때만 새로운 값을 생성함수를 통해 생성해 반환합니다.

배열이 없을 경우에는 매 렌더링 때 마다 새값을 계산해서 반환합니다.

`useCallback(fn, [dep])` 는 `useMemo(()=>fn,[dep])` 와 같습니다. 이들의 사용이 무조건 적인 최적화를 보장하지 않습니다. 올바른 사용법은 해당 [포스트](/react/2020-01-07-useMemo-useCallback) 를 참조해주십시오.

---

### useRef

```javascript
const container = useRef(initialValue);
```

`useRef` 는 `current` 프로퍼티로 전달된 `initialValue`로 초기화된 ref 객체를 반환합니다. 이 ref 객체는 변경이 가능하며 컴포넌트와 그 생명주기를 같이합니다.

자식에게 직접 접근하는 경우에 사용합니다. 이 때, 자식은 React 컴포넌트의 인스턴스 일 수도 있고, DOM 엘리먼트일 수도 있습니다.

`ref` 속성과 `useRef`의 차이점은 `useRef`가 순수 자바스크립트 객체를 생성한다는 것입니다. `useRef` 는 매번 렌더링 할 때 동일한 ref 객체를 제공합니다.

`useRef`는 내용이 변경될 때 그것을 알려주지 않습니다. 즉, `current` 속성이 변화하는 것이 리렌더링을 유발하지 않습니다.

`콜백 ref`를 사용하면 React 가 DOM 노드에 ref를 attach 하거나 detach 할 때 해당 코드를 실행할 수 있습니다.

- [Example](https://ko.reactjs.org/docs/refs-and-the-dom.html#callback-refs)
  - 아래의 예제는 [https://ko.reactjs.org/docs/refs-and-the-dom.html#callback-refs] 에서 가져온 것 입니다.

```javascript
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);

    this.textInput = null;

    this.setTextInputRef = element => {
      this.textInput = element;
    };

    this.focusTextInput = () => {
      // DOM API를 사용하여 text 타입의 input 엘리먼트를 포커스합니다.
      if (this.textInput) this.textInput.focus();
    };
  }

  componentDidMount() {
    // 마운트 되었을 때 자동으로 text 타입의 input 엘리먼트를 포커스합니다.
    this.focusTextInput();
  }

  render() {
    // text 타입의 input 엘리먼트의 참조를 인스턴스의 프로퍼티
    // (예를 들어`this.textInput`)에 저장하기 위해 `ref` 콜백을 사용합니다.
    return (
      <div>
        <input type="text" ref={this.setTextInputRef} />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focusTextInput}
        />
      </div>
    );
  }
}
```

---

## 참조 링크

- [Hooks API Reference](https://ko.reactjs.org/docs/hooks-reference.html)
