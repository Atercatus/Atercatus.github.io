---
title: "this binding"
excerpt: "this binding 의 이해"
last_modified_at: 2020-01-14T12:06:00
---

## `this` binding

`this` 가 무엇을 참조할 것인지 결정짓는 방법에는 크게 3가지가 있습니다.

- Global Execution Context 내의 `this`
- Function Execution Context 내의 `this`
- Arrow Function 내의 `this`

### Global Execution Context 내의 `this`

Global Execution Context 에서는 `this` 가 전역객체를 가리킵니다.

[MDN this](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/this)에 따르면 엄격모드 여부에 관계없이 전역 객체를 참조한다고 나와있습니다.

### Function Execution Context

Function Execution Context 에서는 `this` 를 호출한 방식에 따라 그 값이 다르게 설정됩니다.

#### 단순호출

기본적으로 전역객체를 가리키며 strict mode 에서는 undefined 를 가리킵니다. 엄격 모드에서는 `this` 가 Execution Context 에 진입할 때 설정되는 값을 유지하기 때문입니다. 성능과 보안 이슈 때문에 이렇게 동작하는데 자세한 내용은 [MDN strict mode](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Strict_mode) 를 참조하시면 됩니다.

#### bind 메서드

ES5 에서 도입된 `Function.prototype.bind` 를 이용하면 `this` 를 고정할 수 있습니다. 첫번째 매개변수로는 함수내에서 사용되는 `this` 를 전달할 수 있고, 이후 매개변수는 해당 함수의 매개변수를 고정할 수 있습니다.

고정된 매개변수는 외부에서 변경할 수 없고, 함수 호출 시 필요한 매개변수가 2개였는데 첫번째 매개변수가 `bind` 에 의해서 고정되었다면 실행할 때 하나의 매개변수만이 필요하게 됩니다.

#### Arrow Function

Arrow function 에서는 `this` 가 함수의 lexical context 를 가리킵니다. 즉, 함수가 호출되는 시점에 바인딩되는 다른 예시와는 다르게 화살표함수 내의 `this` 는 그것이 생성될 때 결정됩니다.

아래의 예제는 [MDN this](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/this) 에서 가져왔습니다.

```javascript
const obj = {
  foo: function() {
    const fn = () => this;
    return fn;
  }
};

const fn = obj.foo();

console.log(fn() === obj); // true

const fn2 = obj.foo;
console.log(fn2()() === window); // true or false(strict mode)
```

`fn = obj.foo()` 에서 화살표 함수를 생성합니다. 이 때, 화살표 함수 내의 `this` 는 그 lexical context 인 obj 를 가리키게되고 `fn()` 은 `obj` 를 리턴하게 됩니다.

`fn2 = obj.foo` 에서 아직 화살표 함수는 생성되지않았습니다. 이후 `fn2()()` 에서 화살표 함수가 생성됩니다. 이 때, 화살표 함수의 lexical context 는 global context 입니다. 따라서, window 와 같은 값을 가지게 됩니다.

#### 객체 내부 함수 참조

객체를 참조하여 그 객체가 가지고있는 함수를 호출하게 되면 `this` 는 해당 객체를 가리킵니다.

#### constructor function

생성자 함수로 `new` 키워드와 함께 객체를 생성하게 되면 `this` 는 새롭게 생성하는 객체를 가리킵니다.

## 참조 링크

- [MDN this](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/this)
- [MDN Strict mode](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Strict_mode)
- [MDN 화살표함수](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/%EC%95%A0%EB%A1%9C%EC%9A%B0_%ED%8E%91%EC%85%98)
