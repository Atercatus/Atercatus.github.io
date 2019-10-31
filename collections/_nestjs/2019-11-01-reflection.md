---
title: "Reflection"
excerpt: "Nest.js에서의 Reflection 에 대해서 알아보자"
last_modified_at: 2019-11-01T00:48:00
---

## Reflection

Reflection을 wiki에 검색하면 이와같은 결과가 나옵니다.

> 반영(Reflection)은 컴퓨터 과학 용어로, 컴퓨터 프로그램에서 런타임 시점에 사용되는 자신의 구조와 행위를 관리(type introspection)하고 수정할 수 있는 프로세스를 의미한다. "type introspection"은 객체 지향 프로그램언어에서 런타임에 객체의 형(type)을 결정할 수 있는 능력을 의미한다.

정확히 와닿지는 않지만 보통 구체적인 클래스의 타입에 대해서 알지 못한채로, 그 클래스의 정보를 얻어낼 수 있음을 뜻합니다.

즉, 객체(인스턴스)를 통해(런타임 시점에) 그 클래스의 정보(구조와 행위)를 알 수 있는 것이라고 정리할 수 있을 것 같습니다.

## Nest.js 에서 사용되는 곳

Nestjs에서 Guard를 사용하게 되면 Guard에서 execution context에 접근하여 그에 따라 현재 Request를 route에 접근을 허가할지 불허할지 결정해야하는 경우가 발생할 수 있습니다.

이때 Custom Decorator를 통해 Controller에 정의된 함수들에 추가적인 데이터를 부여할 수 있습니다.

예를 들면, 관리자만이 이 요청을 할 수 있다고 한다면 그에 해당하는 Handler에 `@Role('admin')` 과 같이 명시할 수 있겠습니다.

```javascript
  @Get()
  @Roles('admin')
  get() {
    this.testService.test();
  }
```

그러면 Guard에서는 이를 읽어서 Request를 허가할지 불허할지 판단하는 로직을 작성해주어야 합니다.

이 때, Guard에서는 현재 수행될 Controller의 Handler의 타입을 알 수 없습니다. 어떤 요청이 들어올 지 모르고 어떤 Handler가 실행될지 컴파일 시점에는 알 수 없기 때문입니다.

따라서 우리는 **Reflection**을 이용하여 이를 수행해야 합니다.

우선 `@Roles` Decorator를 작성해줍니다.

- roles.decorator.ts

```javascript
import { SetMetadata } from "@nestjs/common";

export const Roles = (...roles: string[]) => {
  const ret = SetMetadata("roles", roles);
  console.log("META:", ret);
  return ret;
};
```

Nestjs에서 제공해주는 `SetMetadata` 함수를 이용합니다.

- `SetMetadata` 코드

```javascript
export const SetMetadata = <K = any, V = any>(
  metadataKey: K,
  metadataValue: V
) => (target: object, key?: any, descriptor?: any) => {
  if (descriptor) {
    Reflect.defineMetadata(metadataKey, metadataValue, descriptor.value);
    return descriptor;
  }
  Reflect.defineMetadata(metadataKey, metadataValue, target);
  return target;
};
```

여기서 `metadataKey`는 `'roles'`를 나타내고 `metadataValue`는 `roles:string[]` 를 나타냅니다.

- `console.log(ret)`의 결과

```javascript
(target, key, descriptor) => {
  if (descriptor) {
    Reflect.defineMetadata(metadataKey, metadataValue, descriptor.value);
    return descriptor;
  }
  Reflect.defineMetadata(metadataKey, metadataValue, target);
  return target;
};
```

이후 아까 위와 같이 Controller에서 사용할 때는 아래와 같이 사용됩니다.

```javascript
  @Get()
  @Roles('admin')
  get() {
    this.testService.test();
  }
```

여기서 `Roles('admin`)`에서`'admin'`은 바로위에서 나온`metadataValue`즉,`roles`가 되고 'target'은`get`함수가 됩니다.

그러면 Controller에 작성된 `get`함수에는 현재 키는 `'roles'` 이고 값은 `['admin']` 인 메타데이터가 등록되어 있습니다.

이후 Guard에서 이를 접근할 수 있습니다.

- authorization.guard.ts

```javascript
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Reflector } from '@nestjs/core';

@Injectable()
export class AuthorizationGuard implements CanActivate {
  constructor(private readonly reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const roles = this.reflector.get<string[]>('roles', context.getHandler());
    console.log(roles); // ['admin']
    console.log(context.getHandler()); // [Function: get]

    return true;
  }
}
```

우선 Guard를 작성하려면 `CanActivate`를 import 해주어야합니다.
Guard는 이를 implements 하고 생성자에서 **Reflector**를 주입받습니다.

Reflector는 target에 대해 메타데이터를 이용하여 접근가능하도록 도와주는 helper class 입니다.

`context.getHandler()`는 현재의 컨텍스트에 있는 핸들러를 반환해줍니다. 여기서는 Controller에서 보셨던 `get`함수를 의미합니다.

여기서 보면 Guard에서는 현재 context에 존재하는 handler에 대해 알 수 없습니다. 하지만 Reflector를 통해 메타데이터 키 값으로 그에 해당하는 값을 가져올 수 있습니다.

이상 Guard를 이용한 Nest.js 의 Reflection 이였습니다.
