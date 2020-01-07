---
title: "Remote에 있는 모든 branch 당겨오기"
excerpt: "Remote에 있는 모든 branch 당겨오기"
last_modified_at: 2019-11-13T19:00:00
---

- 아래와 같은 명령어를 이용한다.

```
git branch -r | grep -v '\->' | while read remote; do git branch --track "${remote#origin/}" "$remote"; done
git fetch --all
git pull --all
```
