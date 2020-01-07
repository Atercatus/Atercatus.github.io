---
title: "useCallback 과 useMemo를 제대로 사용 하는 법"
excerpt: "useMemo 와 useCallback의 올바른 사용법에 대해 알아보자"
last_modified_at: 2020-01-07T22:06:00
---

## useCallback 과 useMemo를 제대로 사용 하는 법

Kent C. Dodds 님의 [When to useMemo and useCallback](https://kentcdodds.com/blog/usememo-and-usecallback)을 읽으면서 정리한 글입니다.

### useCallback

React 문서에는 아래와 같이 설명되어 있습니다.

```tsx
const memorizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]); // function , dependencies
```

메모이제이션된 콜백을 반환합니다.

콜백과 그것의 의존성 값의 배열을 전달해야합니다. `useCallback`은 콜백의 메모이제이션된 버전을 반환할 것입니다. 그 메모이제이션된 버전은 콜백의 의존성이 변경되었을 때에만 변경됩니다. 이것은, 불필요한 렌더링을 방지하기 위해(예로 `shouldComponentUpdate`를 사용하여) 참조의 동일성에 의존적인 최적화된 자식 컴포넌트에 콜백을 전달할 때 유용합니다.

`useCallback(fn, deps)`은 `useMemo(()=>fn, deps)`와 같습니다.

### useMemo

React 문서에는 아래와 같이 설명되어 있습니다.

```tsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

메모이제이션된 값을 반환합니다.

"생성(create)" 함수와 그것의 의존성 값의 배열을 전달해야합니다. `useMemo`는 의존성이 변경되었을 때에만 메모이제이션된 값만 다시 계산할 것입니다. 이 최적화는 모든 렌더링 시의 고비용 계산을 방지하게 해 줍니다.

`useMemo`로 전달된 함수는 렌더링 중에 실행된다는 것을 기억하세요.

배열이 없는 경우 매 렌더링 때마다 새 값을 계산하게 될 것입니다.

### 최적화는 진짜로 필요할때까지 기다리는 것

위 글에서 보여주는 예시코드입니다.

```tsx
function CandyDispenser() {
  const initialCandies = ["snickers", "skittles", "twix", "milky way"];
  const [candies, setCandies] = React.useState(initialCandies);
  const dispense = candy => {
    setCandies(allCandies => allCandies.filter(c => c !== candy));
  };
  return (
    <div>
      <h1>Candy Dispenser</h1>
      <div>
        <div>Available Candy</div>
        {candies.length === 0 ? (
          <button onClick={() => setCandies(initialCandies)}>refill</button>
        ) : (
          <ul>
            {candies.map(candy => (
              <li key={candy}>
                <button onClick={() => dispense(candy)}>grab</button> {candy}
              </li>
            ))}
          </ul>
        )}
      </div>
    </div>
  );
}
```

렌더링 마다 새롭게 만들어지는 `dispense` 함수를 `useCallback`을 사용했을 때 과연 성능이 좋아질것인가? 하는 질문이 있습니다. `useCallback`을 쓰면 함수를 재생성하지 않기 때문에 성능이 좋아질 것이라고 판단했습니다만 저자는 이에 대해 이런 지적을 합니다.

**실행되는 모든 코드는 한줄한줄이 비용이 소모된다**

`useCallback`을 사용한 코드를 이렇게 변경할 수 있다.

```tsx
const dispense = candy => {
  setCandies(allCandies => allCandies.filter(c => c !== candy));
};
const dispenseCallback = React.useCallback(dispense, []);
```

이부분이 실행될때 마다 함수가 선언되고 빈배열을 정의해야 합니다. 그리고 `useCallback` 또한 호출되어야 합니다. 게다가 함수 자체를 메모이제이션 해 두기 위해 추가로 메모리까지 사용되게 됩니다.

이 `dispense`함수가 핸들러로 추가될 때 `onClick={() => dispense(candy)}` 와 같이 선언되어있습니다. 이 때문에 매 렌더링시에 새로운 함수가 생성되어 전달되고 button 은 다시 렌더링이 이루어 집니다.
이렇게 되면 `useCallback` 으로 감싸진 함수는 메모리에 남아있는 상태에서 새로운 함수가 다시 생성되므로 메모이제이션의 이점도 사라집니다.

### `initialCandies` 에 `useMemo`를 사용하면?

```tsx
const initialCandies = ['snickers', 'skittles', 'twix', 'milky way']
function CandyDispenser() {
  const [candies, setCandies] = React.useState(initialCandies)
  // ...
```

어차피 변경되지 않을 값이라면 컴포넌트 외부에 선언하는 것이 더 올바른 방법입니다. `useMemo`를 사용하게 될 경우 **생성 함수**를 만들어야하고, 의존성 배열을 생성해야하며, `useMemo`를 추가적으로 호출해야합니다.

## `useMemo`와 `useCallback`을 사용해야하는 시점

`useMemo`와 `useCallback`을 사용해야하는 상황을 두 가지로 표현하고 있습니다.

1. 참조 동일성(Referential equality)
2. 고비용의 복잡한 연산

### 참조 동일성

참조 동일성의 경우 레퍼런스를 비교하는 상황을 말합니다. non-primitive type 일 경우 레퍼런스로 비교를 하게 되는데 객체 내부의 값은 같더라도 그 레퍼런스가 다르게 되는 경우에도 렌더링이 발생하는 상황을 막고 싶을 때 사용해야한다는 의미입니다.

본문에 사용된 `DualCounter`의 예를 보면 더 이해하기가 쉽습니다

```tsx
function CountButton({ onClick, count }) {
  return <button onClick={onClick}>{count}</button>;
}
function DualCounter() {
  const [count1, setCount1] = React.useState(0);
  const increment1 = () => setCount1(c => c + 1);
  const [count2, setCount2] = React.useState(0);
  const increment2 = () => setCount2(c => c + 1);
  return (
    <>
      <CountButton count={count1} onClick={increment1} />
      <CountButton count={count2} onClick={increment2} />
    </>
  );
}
```

`CountButton` 의 prop로 들어가는 `count1or2`와 `increment1or2` 는 `DualCounter` 가 렌더링 될 때마다 새롭게 생성되므로 두 개의 `CountButton` 중 하나만 눌려도 두 개의 버튼 모두다 렌더링 됩니다.

불필요한 렌더링이 이루어 지는 이 상황을 해결할 때 `useMemo`와 `useCallback`을 사용하면 쉽게 해결할 수 있습니다.

```tsx
const CountButton = React.memo(function CountButton({ onClick, count }) {
  return <button onClick={onClick}>{count}</button>;
});
function DualCounter() {
  const [count1, setCount1] = React.useState(0);
  const increment1 = React.useCallback(() => setCount1(c => c + 1), []);
  const [count2, setCount2] = React.useState(0);
  const increment2 = React.useCallback(() => setCount2(c => c + 1), []);
  return (
    <>
      <CountButton count={count1} onClick={increment1} />
      <CountButton count={count2} onClick={increment2} />
    </>
  );
}
```

`useMemo`의 생성함수를 보시면 매개변수로 넘어오는 `onClick`과 `count`가 같다면 같은 값이 리턴되므로 prop이 바뀔때만 리렌더링이 이루어집니다.

```tsx
const CountButton = React.memo(
  function CountButton({ onClick, count }) {
    return <button onClick={onClick}>{count}</button>;
  },
  [onClick, count]
);
```

prop으로 넘어오는 `onClick`과 `count` 같은 경우에도 `useCallback` 을 사용하며 dependency에 빈 배열을 넣었기 때문에 최초의 값을 메모이제이션을 합니다. 따라서 `count` 가 변경되었을 때만 렌더링을 수행할 수 있습니다.

```tsx
const [count1, setCount1] = React.useState(0);
const increment1 = React.useCallback(() => setCount1(c => c + 1), []);
const [count2, setCount2] = React.useState(0);
const increment2 = React.useCallback(() => setCount2(c => c + 1), []);
```

### 고비용의 복잡한 연산

이는 `useMemo`가 리액트의 hook에 built-in 되어있는 또다른 이유 라고합니다.(이것은 `useCallback`의 경우에는 적용되지 않습니다.)

```tsx
function RenderPrimes({ iterations, multiplier }) {
  const primes = React.useMemo(() => calculatePrimes(iterations, multiplier), [
    iterations,
    multiplier
  ]);
  return <div>Primes! {primes}</div>;
}
```

위와 같이 `caculatePrimse`와 같이 연산에 비용이 많이드는 함수가 존재할 때, 렌더링마다 이 함수가 호출되게 구현했더라도 그 값이 정말 변경되었을 때만 호출되게 할 수 있습니다.

## 결론

저자는 결론적으로 최적화가 필요하기 전까지는 이를 고려하는데 많은 비용을 사용하지 말라고 말합니다. 정확한 측정과 검증 없이 섣부른 최적화는 피하라는 뜻입니다. 최적화가 반드시 필요해지는 경우가 오지 않는 이상 이에 대한 투자는 피하고 오히려 현재 자신이 React 를 얼마나 잘 쓰고 있는지 그에 대한 투자가 더 필요합니다.
