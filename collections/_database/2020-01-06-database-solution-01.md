---
title: "Optimizer 입문 및 역정규화와 Join 비교"
excerpt: "Nested loop join 시의 Optimizer 동작, 올바른 Join 과 역정규화 비교"
last_modified_at: 2020-01-06T12:06:00
---

## Nested loop join 시의 Optimizer

![image](https://user-images.githubusercontent.com/32104982/71911463-13d93680-31b7-11ea-81c3-b11318179538.png)

위와 같이 두 개의 `dept` 테이블과 `emp` 테이블이 있다고 가정합니다.
아래는 주어진 상황에 대한 쿼리 예시입니다

```query
Select B.dname, A.empno, A.ename, A.sal
  from emp A, dept B
  where A.deptno = B.deptno
```

Nested loop join을 수행한다 했을 때, Optimizer는 1번과 2번 어느 방향으로 동작할 것인가?

### 2번의 경우

- 100건 짜리 `dept` 테이블을 먼저 읽는다는 것 => full scan
- B.deptno를 상수로 바꾸겠다는 의미
- `emp(10만건)` 테이블을 전부 읽으면서 B.deptno(101, 102, ...)를 찾는다
- 이 때, 부서번호(`emp`의 deptno) 에는 Index가 없다.
- 결론적으로 10만건의 `emp`테이블을 full scan 100번 수행
- I/O block 의 양 = 10만 X 100 = 천만건의 I/O block

### 1번의 경우

- 10만건 짜리 `emp` 테이블을 먼저 읽는다는 것 => full scan
- A.deptno를 상수로 바꾸겠다는 의미
- `dept` 테이블을 전부 읽으면서 A.deptno(101, 102, ...)를 찾는다
- 이 때, 부서번호(`dept` 의 deptno) 에는 Index가 존재한다.
- 따라서 `dept` 테이블을 full scan 하지 않고 RowId로 테이블 액세스하여 찾아온다.
- 다음 사원의 deptno도 같을 경우 메모리에 올라와있는 data를 접근하여 불러온다
- 결론적으로 `emp(10만건)` 한번의 full scan + `dept` 에서의 unique index access

### 결론

따라서 1번의 경우가 2번의 경우보다 성능이 좋다.
쿼리의 성능은 CPU, memory 보다 I/O 에서 성능의 차이를 보인다.

## Join VS Denormalization(역정규화 또는 반정규화)

deptno 를 `emp` 테이블에 추가하는 반정규화를 한 예제입니다.

```
Select A.dname, A.empno, A.ename, A.sal
  from emp A
```

- emp 1record 가 20 Byte => 20Byte X 10만
  => 전체 데이터: 약 2MB

- dept 1record가 30 Byte => 30Byte X 100
  => 전체 데이터: 약 3KB

- Nested loop join 이 아닌 Hash join, Sort-merge join 수행할 시

  - 2MB + 3KB 정도의 I/O

- dname(20Byte)를 column으로 추가 시(반정규화)
  - 4MB

따라서 Join을 한 경우가 성능이 더 좋습니다.

### 결론

역정규화가 무조건적인 성능 향상을 불러오지 않습니다. 올바른 Join을 수행해야합니다.

Join시에 Optimizer는 Index가 없는 테이블을 derieve 합니다.
