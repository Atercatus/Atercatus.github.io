---
title: "Prototype"
excerpt: "Prototype 이해 예제"
last_modified_at: 2020-01-14T12:06:00
---

## Prototype 이해 예제

```javascript
function Person() {}

const person = new Person();

console.log(person.__proto__ === Person.prototype);
console.log(Person.__proto__ === Function.prototype);
console.log(Person.prototype.constructor === Person);
console.log(Person.prototype.__proto__ === Object.prototype);
console.log(Function.__proto__ === Function.prototype);
console.log(Function.prototype.constructor === Function);
console.log(Function.prototype.__proto__ === Object.prototype);
console.log(Function.__proto__.__proto__ === Object.prototype);
console.log(Object.__proto__ === Function.prototype);
console.log(Object.prototype.__proto__ === null);
console.log(Object.prototype.constructor === Object);
```

위의 답은 모두 true 입니다.

```javascript
function Test() {
  this.value = 0;
}

const before = new Test();

Test.prototype = {
  another: 3
};

const after = new Test();

console.log(before);
console.log(after);

console.log(before.__proto__.constructor === Test);
console.log(after.__proto__.constructor === Object);
```

도중에 프로토타입 객체를 변경하면 위와 같이 동작합니다.

## 참조링크

- [poiemaweb - Prototype](https://poiemaweb.com/js-prototype)
