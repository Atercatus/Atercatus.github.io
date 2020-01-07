---
title: "DB Migration"
excerpt: "Typeorm 을 이용해 DB Migration file을 작성해보자"
last_modified_at: 2019-11-13T19:00:00
---

## DB Migration

### Typeorm-cli 설정

typeorm-cli 를 통한 migration 파일을 생성할 때, `.ts`의 확장자로 파일이 생성된다. 이후 migration을 적용하려고 하면 에러가 발생한다.
따라서 typescript로 작성된 코드를 js로 변환한 뒤 마이그레이션을 실시해야한다.

ts-node를 설치한 이후 아래와 같은 스크립트를 추가한다.

- package.json

```javascript
    "typeorm": "ts-node ./node_modules/typeorm/cli.js"
```

이후, migration을 적용할 때는 `npm run typeorm migration:run` 또는 `npm run typeorm migration:revert` 과 같이 명령어를 작성한다.

### dotenv 를 통한 설정

```
TYPEORM_CONNECTION=
TYPEORM_HOST=
TYPEORM_USERNAME=
TYPEORM_PASSWORD=
TYPEORM_DATABASE=
TYPEORM_PORT=
TYPEORM_SYNCHRONIZE=
TYPEORM_LOGGING=
TYPEORM_ENTITIES=

TYPEORM_MIGRATIONS=src/migration/*.ts // 해당 디렉토리에서 migration 파일을 로드해서 적용한다.
TYPEORM_MIGRATIONS_DIR=src/migration  // 새 migration 파일을 생성할 디렉토리
```

### 참조

- [Typeorm-cli](https://github.com/typeorm/typeorm/blob/master/docs/using-cli.md)
- [Typeorm-migration](https://github.com/typeorm/typeorm/blob/master/docs/migrations.md)
