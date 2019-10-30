---
title: "Introduction of Nest.js"
excerpt: "Nest.js 의 전체적인 개념과 주요 요소들에 대해서 배워보자"

header:
  # teaser: /assets/images/random-cover/1.png
  teaser: /assets/images/nestjs/nest-logo-small.png
  overlay_image: /assets/images/nestjs/nest-og.png
  og_image: /assets/images/nestjs/nest-og.png

last_modified_at: 2019-10-19T20:00:00
---

## Controller

Controller는 들어오는 request에 따른 reponse를 반환해주는 목적을 가진다.

각각의 Controller는 하나 이상의 route를 가지고, route에 따라 다른 동작을 수행한다.

> Controller 를 사용하기 위해서는 Controller decorator를 사용해야한다.
>
> ```javascript
> import { Controller } from "@nestjs/common";
> ```

## Middleware

기본적으로 express의 Middleware와 같이 동작한다.

> Middleware functions can perform the following tasks:
>
> - execute any code.
> - make changes to the request and the response objects.
> - end the request-response cycle.
> - call the next middleware function in the stack.
> - if the current middleware function does not end the request-response cycle, it must call `next()` to pass control to the next middleware function. Otherwise, the request will be left hanging

### 선언방법

```javascript
import { Injectable, NestMiddleware } from "@nestjs/common";
import { Request, Response } from "express";

@Injectable()
export class DeserializerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: Function) {
    console.log("Deserialize");
    next();
  }
}
```

### 적용방법

```javascript
import { Module, NestModule, MiddlewareConsumer } from "@nestjs/common";
import { AppController } from "./app.controller";
import { AppService } from "./app.service";
import { DeserializerMiddleware } from "./core/middlewares/deserializer/deserializer.middleware";
import { TestMiddleware } from "./core/middlewares/test.middleware";

@Module({
  controllers: [AppController],
  providers: [AppService]
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(TestMiddleware, DeserializerMiddleware).forRoutes("auth");
  }
}
```

`apply` 에 들어간 순서대로 실행이 됩니다.
