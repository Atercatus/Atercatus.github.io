---
title: "Injection scopes of NestJS "
excerpt: "NestJS 의 Injection scopes에 대해 알아보자"
last_modified_at: 2019-10-31T18:00:00
---

## Injection scopes

Nest.js 에서 서비스(Provider)들은 기본적으로 Singleton scope로 동작합니다.

하지만 Middleware 에서 Request 객체에 어떠한 값을 첨부할 경우가 발생할 때, 한 Request에 해당하는 생명주기동안만 사용하고자 한다면 그 scope를 변경해주어야 합니다.

Nest.js 에서는 Provider는 총 3개의 scope를 가집니다.

| Scope       |                                                                                                                                Description                                                                                                                                 |
| ----------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| `SINGLETON` | A single instance of the provider is shared across the entire application. The instance lifetime is tied directly to the application lifecycle. Once the application has bootstrapped, all singleton providers have been instantiated. Singleton scope is used by default. |
| `REQUEST`   |                                                       A new instance of the provider is created exclusively for each incoming request. The instance is garbage-collected after the request has completed processing.                                                       |
| `TRANSIENT` |                                                                Transient providers are not shared across consumers. Each consumer that injects a transient provider will receive a new, dedicated instance.                                                                |

### 사용방법

```javascript
import { Injectable, Scope } from "@nestjs/common";

@Injectable({ scope: Scope.REQUEST })
export class InjectionService {
  cache: string[] = [];
  push(value: string) {
    this.cache.push(value);
  }
}
```

Scope를 제한하기 위해서는 Service를 선언할 때 `@Injectable` 데코레이터 안쪽에 scope를 명시해주는 옵션(객체)를 넣어주어야합니다.

이 Service의 Scope가 Request에 한정되어있는지를 확인하기 위해 배열을 만들어 원소를 하나씩 넣도록 구성했습니다.

만약 이 Service의 Scope가 Request라면 요청을 보낼때 마다 배열의 원소는 하나만 있을 것이고, 그게 아니라면 누적될 것입니다.

AModule에 이를 import하여 사용해보겠습니다.

- a.module.ts

```javascript
import { Module } from "@nestjs/common";
import { AAAService } from "./A.service";
import { AAAController } from "./A.controller";
import { InjectionService } from "../injection/injection.service";

@Module({
  imports: [InjectionService],
  controllers: [AAAController],
  providers: [AAAService, InjectionService]
})
export class AAAModule {}
```

- a.service.ts

```javascript
import { Injectable } from '@nestjs/common';
import { InjectionService } from '../injection/injection.service';

@Injectable()
export class AAAService {
  constructor(
    private readonly injectionService: InjectionService,
  ) {}

  test() {
    this.injectionService.push('this');
    console.log(this.injectionService.cache);
  }
}

```

![image](https://user-images.githubusercontent.com/32104982/67933534-5277d100-fc09-11e9-962d-139a0cab75af.png)

postman을 이용하여 같은 요청을 3번 보냈을 때 위와 같이 원소가 하나만 들어있는 것을 확인 할 수 있습니다.

## Custom provider로 구현해보기

Custom provider를 이용하면 Service 자체는 Scope에 대한 명세를 하지않아도 주입할 때 그 범위를 지정해 줄 수 있습니다.

- Injection.service.ts

```javascript
import { Injectable, Scope } from "@nestjs/common";

@Injectable()
export class InjectionService {
  cache: string[] = [];
  push(value: string) {
    this.cache.push(value);
  }
}
```

- A.module.ts

```javascript
import { Module, Scope } from "@nestjs/common";
import { BBBModule } from "../B/B.module";
import { AAAService } from "./A.service";
import { AAAController } from "./A.controller";
import { InjectionService } from "../injection/injection.service";

const mockInjectionService = {
  provide: "TEST_SERVICE",
  useClass: InjectionService,
  scope: Scope.REQUEST
};

@Module({
  imports: [BBBModule, InjectionService],
  controllers: [AAAController],
  providers: [AAAService, mockInjectionService]
})
export class AAAModule {}
```

위와 같은 결과를 얻으실 수 있습니다.
