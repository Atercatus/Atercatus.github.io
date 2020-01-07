---
title: "Value object VS Data transfer object"
excerpt: "Value object VS Data transfer object"
last_modified_at: 2019-11-25T23:34:00
---

- [위키피디아](https://en.wikipedia.org/wiki/Value_object)
- [Martin Fowler - VO](https://martinfowler.com/bliki/ValueObject.html)
- [Martin Fowler - DTO](https://martinfowler.com/eaaCatalog/dataTransferObject.html)

---

## Value object

computer science 에서 `Value object`는 동일성(equality)을 기반으로 하지 않는 단순한 entity를 나타냅니다.

즉, 두 개의 value object는 같은 값을 가지면 같다라고 합니다만 이 두 객체는 같은 객체를 뜻하지는 않습니다.

Value object는 반드시 immutable 해야합니다. 동일하게 생성 된 두 개의 value object는 동일하게 유지되어야 한다는 암묵적인 계약이 필요합니다.

## Data Transfer Object

![image](https://user-images.githubusercontent.com/32104982/69497158-340da880-0f1d-11ea-8262-dd4d3704220e.png)

> 출처 : https://martinfowler.com/eaaCatalog/dataTransferObject.html

원격에 있는 interface를 호출 할 때, 각각의 호출은 많은 비용이 소요됩니다. 즉, 호출의 빈도를 줄인다면 비용을 줄일 수 있습니다.
이것을 가능하게 하는 방법 중 하나는 많은 매개변수를 사용하는 것입니다. - 이는 오직 하나의 매개변수만 반환하게 하는 자바와 같은 언어에서는 불가능 할때가 있습니다.

이를 해결 하기 위해서는 Data Transfer Object를 생성해서 이 객체가 호출에 대해 모든 데이터를 가지고 있게하면 됩니다.
이는 serializable 되어 connection 너머에 전달됩니다. 보통 assembler는 server side 에서 DTO와 어떤 domain object 사이에 데이터를 전송하기 위해 사용됩니다.

DTO는 multiple remote call을 하나의 call로 일괄처리 하기 위해 사용되는 것이 주요한 기능이지만, 전송되는 데이터에 대한 serialization mechanism을 캡슐화 할 수 있다는 또다른 장점이 존재합니다.
