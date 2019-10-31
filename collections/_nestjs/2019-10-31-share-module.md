---
title: "Shared module"
excerpt: "Nest.js에서 Module을 공유하는 방법에 대해서 알아보자"
last_modified_at: 2019-10-31T20:00:00
---

## Shared module

Nest.js 는 기본적으로 Module을 그래프 형식으로 관리하기를 원합니다.

최상위 Module인 AppModule은 다른 모듈을 import 해서 관리하는데, 거기에서 모든 의존성은 주입되는 것을 지향하고 다른 서비스들이 이 곳을 거치지 않고 추가되는것을 피해야합니다.

때론 그래프 중간 지점에서 가장 하단에 있는 Module에서 제공하는 Service를 사용하고 싶은 경우가 발생합니다.

이 때 어떻게 해야하는지 예제를 통해 알아보도록 합시다.

현재 App-A-B-C 와 같은 모습으로 그래프가 생성되어있습니다.

A는 B를 import 하고 B는 C를 import합니다.
A에서 C module 에서 제공하는 Service를 사용하고 싶을 때는 B module 에서 C module을 re-exporting 해주어야합니다.
[문서](https://docs.nestjs.com/modules#module-re-exporting)

- C.service.ts

```javascript
import { Injectable } from "@nestjs/common";

@Injectable()
export class CCCService {
  test(): boolean {
    console.log("this is test");
    return true;
  }
}
```

여기서 제공하는 `test`를 A의 Service에서 주입받아 사용해보도록 합시다.

- C.module.ts

```javascript
import { Module } from "@nestjs/common";
import { CCCService } from "./C.service";

@Module({
  providers: [CCCService],
  exports: [CCCService]
})
export class CCCModule {}
```

- B.module.ts

```javascript
import { Module } from "@nestjs/common";
import { CCCModule } from "../C/C.module";

@Module({
  imports: [CCCModule],
  exports: [CCCModule]
})
export class BBBModule {}
```

위와 같이 imports 와 exports를 모두 해주어야 이를 imports 하는 Module에서 이를 사용가능합니다.

- A.module.ts

```javascript
import { Module } from "@nestjs/common";
import { BBBModule } from "../B/B.module";
import { AAAService } from "./A.service";
import { AAAController } from "./A.controller";

@Module({
  imports: [BBBModule],
  controllers: [AAAController],
  providers: [AAAService]
})
export class AAAModule {}
```

- A.service.ts

```javascript
import { Injectable } from '@nestjs/common';
import { CCCService } from '../C/C.service';

@Injectable()
export class AAAService {
  constructor(private readonly cService: CCCService) {}

  test() {
    this.cService.test();
  }
}

```

- A.controller.ts

```javascript
import { Controller, Get } from '@nestjs/common';
import { AAAService } from './A.service';

@Controller('test')
export class AAAController {
  constructor(private readonly aService: AAAService) {}

  @Get()
  test(): void {
    this.aService.test();
    return;
  }
}

```

`http://localhost:port/test` 에 GET 요청을 보낼경우 `this is test` 라는 문구가 출력 되는 것을 확인 할 수 있습니다.

많이 하위의 많은 모듈을 상위에서 사용되어야한다면 다소 복잡해 지는 구조가 발생하지만 Module 간의 관계를 graph 형태로 유지하고자 하는 철학에 맞추어야하므로 불가피한 구조라고 생각됩니다.

다른 방법이 있으시면 댓글로 남겨주세요!.
