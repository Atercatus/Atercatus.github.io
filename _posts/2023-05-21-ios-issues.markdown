---
layout: post
title: '[Web] ios에서 발생하는 이슈 모음'
date: 2023-05-21 09:43:28 +0900
categories: [web, ios]
---

# 1. ios 에서 overflow: hidden 과 border-radius 가 동작하지 않는 이슈

ios 낮은 버전에서 발생하는 이슈입니다. `overflow: hidden` 인 요소에 적용되어있는 `border-radius` 가 동작하지 않는 경우가 있습니다.

## 해결방안

`overflow: hidden; border-radius` 를 적용할 엘리먼트에 추가로 아래 스타일(`-webkit-mask-image: -webkit-radial-gradient(white, black);` ) 적용하면 해결됩니다.

{% highlight css %}
// Add on element with overflow
-webkit-mask-image: -webkit-radial-gradient(white, black);
{% endhighlight %}

# 2. ios 에서 합성어 입력 시, 이전 문자열이 입력되는 경우

채팅 입력 기능을 개발할 때, 메시지를 입력하고 전송 버튼을 눌러도 input 태그의 포커스가 유지되도록 구현하는 경우가 있습니다. 이는 키패드가 내려가는 것을 방지하기 위한 방식입니다. 그러나 이 경우 새로운 텍스트를 입력할 때 이전에 전송된 텍스트와 합쳐지는 문제가 발생할 수 있습니다. 이 문제를 해결하려면 iOS에서 한글과 같은 조합형 언어가 입력되는 방식을 이해해야 합니다.

## iOS에서 한글 입력 과정

iOS에서 한글과 같은 조합형 언어를 입력할 때는, 글자가 완성되기 전까지 입력된 부분을 버퍼에 저장합니다. 예를 들어, **"각"**을 입력한다고 가정해봅시다.

사용자가 "가"를 입력하면, "가"는 버퍼에 저장됩니다.
이후, 사용자가 "ㄱ"을 입력하면, 버퍼에 있던 "가"를 삭제하고 받침 "ㄱ"을 추가해 "각"이 완성됩니다.
이 과정은 input 요소에 이벤트 리스너를 추가하면 확인할 수 있습니다.

## 이슈의 원인

채팅 입력 시 포커스를 강제로 유지하려고 하면, 한글 입력 과정에서 버퍼에 저장된 글자가 조합되는 도중에 원치 않은 입력 동작이 발생할 수 있습니다. 즉, 이전 텍스트가 지워지지 않고 새로운 텍스트와 합쳐지는 문제가 생깁니다.

## 해결 방법

이 문제를 해결하려면 보이지 않는 input 요소를 활용해 조합 과정이 완료된 후 포커스를 전환하는 방법을 사용할 수 있습니다:

사용자가 텍스트 입력을 완료하여 조합이 종료되면, 보이지 않는 input 요소로 포커스를 이동시킵니다.
이후, 다시 원래의 input 요소로 포커스를 복원합니다.
이 방식은 한글 조합 과정에서 발생하는 입력 충돌 문제를 방지하면서도 포커스를 유지해 키패드가 내려가는 것을 방지할 수 있습니다.

```html
<input style="width: 1px; height: 1px; z-index: -1" (focus)="onFocusBlankInput()" #blankInput />
```

```tsx
send(): void {
  this.blankInput?.nativeElement.focus();
	// ...
}

onFocusBlankInput(): void {
  this.messageInput?.nativeElement.focus();
}
```

# 3. unload, beforeunload, pagehide, visibilitychange 관련 이슈

# 4. 유저스크롤 도중 문서 조작시 발생하는 이슈

# 5. ios input focus 되지 않는 이슈

유저 인터랙션 없이 focus 불가능. 하지만 이미 다른 input 에 포커스 되어있었다면 그걸 옮기는건 가능

# 6. 자동재생이 되지 않는 경우

# 7. 하단 노치 대응

# 8. and 와 가상 키보드 동작 차이 & 채팅창과 컨텐츠 함께 보는 방법
