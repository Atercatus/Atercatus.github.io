---
title: "blur event and css unset"
excerpt: "blur event and css unset"
last_modified_at: 2019-11-17T16:34:00
---

## blur

`blur` 이벤트는 엘리먼트의 포커스가 해제되었을때 발생합니다. 이 이벤트와 focusout 이벤트의 가장 다른점은 focusout 은 이벤트 버블링이 발생합니다.

- [참조](https://developer.mozilla.org/ko/docs/Web/Events/blur)

## unset

`unset` CSS 키워드는 `initial` 및 `inherit`키워드의 조합(combination)입니다. 이 다른 CSS 키워드 둘처럼, CSS 단축(shorthand) 속성 `all` 포함 모든 CSS 속성에 적용될 수 있습니다. 이 키워드는 속성을 부모로부터 상속받은 경우 그 상속값으로 아니면 초기값으로 재설정(reset) 합니다. 다시 말해서, 첫 번째는 `inherit` 키워드 처럼 두 번째는 `initial` 키워드 처럼 행동합니다.

- [참조](https://developer.mozilla.org/ko/docs/Web/CSS/unset)

## initial

`initial` CSS 키워드는 속성의 초기값(initial value)을 요소에 적용합니다. 모든 CSS 속성에 허용되고 요소가 속성의 초기값을 사용하도록 합니다.

- [참조](https://developer.mozilla.org/ko/docs/Web/CSS/initial)

## inherit

`inherit` CSS 값은 요소가 부모 요소로부터 속성(property)의 계산값(computed value)을 갖도록 지정되게 합니다. 모든 CSS 속성에 허용됩니다.

상속되는 속성(inherited properties)의 경우, 이는 기본 동작(behavior)을 강화하고 오직 다른 규칙을 재정의(override)해야 합니다. 상속되지 않는 속성(non-inherited properties)은, 이는 보통 비교적 거의 의미가 없는 동작을 지정하고 당신은 대신 `initial` 혹은 `all` 속성에 `unset` 사용을 고려할 지도 모릅니다.

상속(inheritance)은 심지어 부모 요소가 포함(containing) 블록이 아니더라도, 항상 문서 트리 내 부모 요소로부터입니다.
