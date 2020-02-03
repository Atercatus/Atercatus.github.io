---
title: "Hoisting, Scope and Closure"
excerpt: "Hoisting, Scope and Closure"
last_modified_at: 2020-01-14T12:06:00
---

본 포스트는 [The Ultimate Guide to Hoisting, Scopes, and Closures in JavaScript](https://tylermcginnis.com/ultimate-guide-to-execution-contexts-hoisting-scopes-and-closures-in-javascript/) 과 [Understanding Execution Context and Execution Stack in Javascript](https://blog.bitsrc.io/understanding-execution-context-and-execution-stack-in-javascript-1c9ea8642dd0)를 읽은 후 해석한 내용입니다.

## Execution Context

실행 컨텍스트는 자바스크립트 코드가 평가(eval)되고 실행되는 환경을 추상화하여 표현한 것 입니다. 이는 코드를 해석하고 실행하는 복잡한 과정을 좀 더 수월하게 하기 위해 존재합니다. 3가지의 `Execution Context`의 타입이 존재합니다.

### Global Execution Context

기본 `Execution Context` 으로서 프로그램당 하나만 존재합니다. 코드가 실행되기 위해 적재되면 자바스크립트 엔진은 `Global Execution Context`를 만듭니다. 이는 `global object` 와 `this` 객체를 생성하는 과정을 포함합니다. 이때 `this`는 `global object`인 `window(browser)`또는 `global(node.js)`를 참조합니다.

### Functional Execution Context

함수가 **호출(invoked)**될 때 마다 생성됩니다. 각 함수는 자신만의 `Functional Execution Context`를 가집니다.

### `Eval` Function Execution Context

`eval` 함수는 자신 만의 `Execution Context` 를 가집니다.

## Phase

`Execution Context`는 자바스크립트 엔진에 의해서 두 가지의 phase를 거치면서 생성됩니다.

1. Creation Phase
2. Execution Phase

### Creation Phase

`Execution Context` 는 Creation phase 동안 생성됩니다. 생성되는 동안 두 가지 일이 발생합니다.

1. **LexicalEnviroment** 컴포넌트 생성
2. **VariableEnvironment** 컴포넌트 생성

#### Lexical Environment

Lexical Environment 는 변수의 이름과 그 변수가 가지는 값(primitive, reference of <array, function, object, etc>)을 맵핑하고 있는 구조입니다. 이 컴포넌트는 또 3개의 작업을 포함합니다.

1. Environment Record
2. Reference to the outer environment
3. This binding

##### Environment Record

변수와 함수의 정의를 LexicalEnvironment 컴포넌트 내에 저장합니다. 이들은 `Environment Record` 내의 `Declarative Environment Record` 라고 불리는 곳에 저장됩니다.

![image](https://user-images.githubusercontent.com/32104982/72327049-b03d9480-36f3-11ea-9d2e-5df7b676050d.png)

이 과정에서 변수와 함수의 영역이 메모리에 할당되며 할당된 메모리에는 `var`로 선언된 변수의 경우 `undefined` 값이 할당됩니다. 함수의 경우 그 선언부가 메모리에 적재됩니다.

이 때문에 코드상에서 `var`로 선언된 변수나 함수 선언부 위쪽에서 `console.log`를 찍거나 함수를 실행했을 때 `undefined`나 함수가 실행되는 경우가 발생합니다. 변수는 `undefined`로 할당되어 메모리에 적재되어 있고, 함수는 선언부만이 메모리에 적재되어 있고 호출 될 경우 `Function Execution Context`가 생성되므로 이같은 동작이 가능합니다.
우린 이를 `Hoisting`이라고 부릅니다. 즉 우리 `Hoisting` 은 작성한 코드가 최상단부로 이동된다고 하는 것은 오해입니다. 선언부가 메모리에 적재되는 방식이 위와 같으므로 이러한 동작이 가능한 것입니다.

> `var` 이 아니라 `let` 또는 `const` 로 선언된 변수의 경우 값이 초기화되지 않은 상태로 남아있습니다.

`Global Execution Context` 일 경우 global object 를 추가로 저장합니다. object 내의 함수의 변수를 포함하는 object의 `property`들을 바인딩하는 작업을 포함합니다. 이들은 `Object Environment Record`에 저장합니다.

`Function Execution Context`는 정의된 함수가 호출 될 때 생성됩니다. 생성될 때 `arguments` 를 추가로 저장합니다. 이는 매개변수로 넘어온 객체들의 인덱스와, 값, 매개변수의 개수를 포함합니다.

- [MDN hoisting](https://developer.mozilla.org/ko/docs/Glossary/Hoisting)
- [MDN arguments](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/arguments)

##### Reference to the outer environment

현재 Context 외부에 있는 `Lexical Environment` 의 레퍼런스를 생성합니다.

![image](https://user-images.githubusercontent.com/32104982/72328473-5094b880-36f6-11ea-873b-36b033af1507.png)

위 그림을 보면 `print` 함수의 `Execution Context` 에는 변수 `a` 가 존재하지 않습니다. 따라서 `console.log(a)` 가 실행 될 때, 현재 컨택스트 외부의 컨택스트를 참조하게 됩니다. 이때 위에 설명된 레퍼런스를 사용하게 됩니다.

이처럼 현재 컨택스트에 없는 변수나 함수를 찾기 위해서 그 상위 컨택스트를 참조해가는 과정을 `Scope chain` 이라고 합니다. `Scope` 란 현재 `Execution Context`를 말합니다. 이를 이용하면 `Closure` 에 대한 설명이 가능합니다. 아래에 다시 설명하겠습니다.

![image](https://user-images.githubusercontent.com/32104982/72403712-65c12400-3796-11ea-919d-767868e6e28b.png)

위의 예제를 보면 `[[Scopes]]` 의 존재를 알 수 있습니다. 이를 통해 외부 컨택스트를 접근할 수 있습니다.

- Chrome version

  ```javascript
  function foo() {}

  console.dir(foo);
  ```

- Node.js version

  ```javascript
  global.a = () => {
    /* test function */
  };

  const s = new (require("inspector").Session)();
  s.connect();

  let objectId;
  s.post("Runtime.evaluate", { expression: "a" }, (err, { result }) => {
    objectId = result.objectId;
  });
  s.post(
    "Runtime.getProperties",
    { objectId },
    (err, { internalProperties }) => {
      console.dir(internalProperties);
    }
  );
  ```

  - [Chrome DevTools Viewer](https://chromedevtools.github.io/devtools-protocol/tot/Runtime)

##### This Binding

`Global Execution Context` 에서는 아시다시피 `this` 는 `global object`를 가리키게 됩니다.

`Function Execution Context` 에서는 그 함수가 호출된 경우에 따라 다르게 바인딩 됩니다. 그 함수가 object 내에서 호출된 경우 즉, object 레퍼런스를 통해 호출된 경우 `this`는 그 object를 가리키게 되고, 외부에서 호출될 경우 `this`는 `global object` 를 가리킵니다. `strict mode` 일 경우 `undefined` 를 가리킵니다. 아래의 예제 코드를 참조하십시오.

```javascript
const author = {
  name: "Atercatus",
  state: "hungry",
  print() {
    console.log(`${this.name} is ${this.state}`);
  }
};

author.print(); // Atercatus is hungry

const fakerPrint = author.print;

fakerPrint(); // undefined is undefined
```

#### Variable Environment

`Variable Environment`는 위에서 설명한 `Lexical Environment` 와 같습니다. 하나 다른점이 있다면 ES6 에서 `Lexical Environment` 에서는 `var` 로 선언된 변수를 저장하고 `Variable Environment` 에서는 `let` 과 `const`로 선언된 변수를 저장합니다.

### Execution Phase

말그대로 자바스크립트 엔진이 코드 한줄 한줄을 해석하고 실행하는 과정입니다. 변수 할당문을 실행하면 변수에 값을 할당하고 함수 실행문을 실행하면 `Function Execution Context`를 생성합니다.

아까 `Creation Phase` 에서 설명했듯이 `let` 과 `const` 로 선언된 변수는 `undefined` 로 초기화 않습니다. 이후 `Execution Phase` 에서 다른 값으로 초기화 되지 않고 선언만 될 경우, 이후 코드상에서 그 값을 참조했을 때 그 값을 찾을 수 없으므로 `undefined`로 그 값이 할당됩니다.

### Closure

`Scope chain`에 대해서 설명할 때, 이를 이용하면 `Closure` 를 이해할 수 있다고 했습니다. `Closure`에 대한 명세는 검색하는 곳에 따라 다양하게 나타납니다. [MDN](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Closures) 에는 함수와 함수가 선언된 어휘적 환경의 조합이다. 라고 명시되어 있습니다. 자바스크립트는 함수를 리턴하면 `Closure Scope` 를 생성합니다. 즉, 함수와 함수가 선언된 `Execution Context` 의 조합이라고 생각합니다.

![image](https://user-images.githubusercontent.com/32104982/72351648-1a6e2d80-3724-11ea-9225-5a8084dbd0d5.png)

위와 같이 `getClosure` 내에 `innerFunction` 을 선언했습니다. `innerFunction` 내부에는 `value` 와 `b`가 존재하지 않습니다.

![image](https://user-images.githubusercontent.com/32104982/72351684-29ed7680-3724-11ea-8807-efaa86112e13.png)

생성된 `Closure Scope` 에는 `value` 와 `b` 가 존재합니다.

![image](https://user-images.githubusercontent.com/32104982/72351725-3d004680-3724-11ea-8232-64df2f062226.png)

`closure` 함수가 실행 될 때 `innerFunction` 의 `Execution Context` 내에는 `b` 와 `value` 가 존재하지 않습니다. 따라서 `Scope chain` 을 통해 상위 컨택스트(`Closure Scope`)의 `b`와 `value`를 참조합니다.

## 참조링크

- [The Ultimate Guide to Hoisting, Scopes, and Closures in JavaScript](https://tylermcginnis.com/ultimate-guide-to-execution-contexts-hoisting-scopes-and-closures-in-javascript/)

- [Understanding Execution Context and Execution Stack in Javascript](https://blog.bitsrc.io/understanding-execution-context-and-execution-stack-in-javascript-1c9ea8642dd0)
