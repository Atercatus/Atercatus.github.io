---
title: "타입스크립트 lint 룰 적용 및 commit 전 lint 검사하기"
excerpt: "타입스크립트 lint 룰 적용 및 commit 전 lint 검사하기"
last_modified_at: 2019-11-13T19:00:00
---

# 타입스크립트 lint 룰 적용 및 commit 전 lint 검사하기

## 1. lerna를 활용하여 모노레포에서 복수개의 프로젝트 관리하기

lerna 라이브러리를 통해 모노레포에서 복수개의 프로젝트를 관리할 수 있습니다. wedev 서비스의 경우에는 한개의 레포지터리에 클라이언트와 서버 프로젝트 두 가지가 공존합니다. 이 때, lerna 를 활용하면 복수의 프로젝트를 손쉽게 관리할 수 있습니다.

### 사용하기

먼저, 레포지터리의 root에서 npm 프로젝트를 생성합니다.

```
npm init
```

그 다음 lerna를 설치합니다.

```
npm install lerna -D
```

그리고 lerna 프로젝트를 생성합니다.

```
lerna init
```

다음과 같은 디렉토리 구조가 생성됩니다.

```
repo/
  packages/
  package.json
  lerna.json
```

### 프로젝트 구성하기

저희가 구성할 프로젝트 client와 server를 생성해야합니다. lerna로 관리되는 프로젝트는 생성된 `packages`디렉토리에 생성합니다.

해당 디렉토리에 각각의 프로젝트를 작성합니다.

```
repo/
  packages/
    client/
      ...
      package.json
    server/
      ...
      package.json
  package.json
  lerna.json
```

## 2. 프로젝트에 tslint 규칙 작성하기

각 프로젝트에 tslint 규칙을 작성합니다.

tslint cli를 통해서 쉽게 생성할 수 있습니다.

```
tslint --init
```

그 다음 각 프로젝트의 린트룰을 쉽게 검사할 수 있도록, 스크립트를 추가합니다.

각 프로젝트의 `package.json`파일에 다음과 같은 내용을 추가합니다.

```
  "scripts": {
    ...,
    "lint": "tslint -p tsconfig.json -c tslint.json"
  },
```

## 3. husky를 통해 커밋 전 작업 설정하기

이제 커밋을 작성할 때, 레포에서 관리하고 있는 `client`와 `server`프로젝트의 린트 검사를 실행하여 린트 규칙에 부합하는지 확인하고 커밋이 진행될 수 있도록 작업을 설정할 수 있습니다. 이 때, husky를 활용하여 위와 같은 작업을 손쉽게 설정할 수 있습니다.

레포의 루트 디렉토리에서 husky 라이브러리를 설치합니다.

```
npm install husky -D
```

그 다음, package.json에 다음 설정을 추가합니다.

```
// package.json
{
  ...,
  "husky": {
    "hooks": {
      "pre-commit": "lerna run lint"
    }
  }
}
```

이제, 커밋을 작성하면 자동으로 린트 검사가 동작하고, 검사 결과가 성공적일 때만 커밋이 등록됩니다.
