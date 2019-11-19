---
title: "Maybe Types"
excerpt: "Maybe Types"
last_modified_at: 2019-11-17T20:34:00
---

## Maybe Types

```typescript
interface Example {
  key1?: string;
  key2?: number;
}
```

타입스크립트에서 strictNullChecks 를 true로 설정했을 경우, 모든 타입은 `null` 또는 `undefined` 옵션을 가질수 없다.
