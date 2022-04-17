# Transaction

- 트랜잭션
  - 트랜잭션 사용하는 이유
  - ACID (트랜잭션 특징)
  - 트랜잭션 연산
  - 트랜잭션 상태
- 트랜잭션 격리 수준
  - Isolation level 격리 수준
  - Isolation level 필요성
  - Isolation level 종류
- 트랜잭션 격리 수준 설정할 때 발생하는 문제점들



## 트랜잭션

| **데이터베이스의 상태를 변화시키기 위해 수행하는 논리적 작업 단위**

- `상태 변화시킨다는 것` -> SQL 질의어를 통해 DB에 접근 
  - 예를 들어, SELECT, INSERT ..

- `작업 단위` -> 많은 SQL 명령문들을 사람이 정하는 기준에 따라 정해짐

  > 예시 : A가 B에게 만원 송금한다. 
  > A 계좌 차감(출금 UPDATE) + B계좌 입금(입금 UPDATE)
  > -> 통틀어 하나의 트랜잭션이라고 함
  > -> 두 쿼리가 성공적으로 완료되어야 트랜잭션이 완료됨   `Commit`
  > -> 작업 단위의 쿼리 중 하나라도 실패하면 모든 쿼리문을 취소하고 이전상태로 돌려놓음 `Rollback`

### 트랜잭션을 사용하는 이유

트랜잭션은 하나의 논리적인 작업의 단위이기 때문에, 여러개의 작업을 하나의 논리적인 단위로 묶어서 반영과 복구를 할 수 있다. 

트랜잭션을 통해 데이터베이스의 회복과 병행 제어가 가능하다. 즉, 데이터베이스에서 오류가 발생하면, 빠른 회복, 여러 사용자가 동시에 데이터베이스를 사용할 수 있도록 제어해주는 역할을 한다.

### 트랜잭션 특징 ACID

- 원자성(Atomicity)

  > 트랜잭션이 DB에 모두 반영되거나, 혹은 전혀 반영되지 않아야 된다.

  - 커밋 OR 롤백 연산을 통해 원자성을 보장

- 일관성(Consistency)

  > 트랜잭션의 작업 처리 결과는 항상 일관성 있어야 한다.

  - 명시적 일관성 : 기본키, 외래키 등 무결성 제약조건을 해치지 않는 데이터들 대해서만 트랜잭션 수행되어야 한다.
  - 비명시적 일관성 :  예) 계좌 이체에서, A 계좌에서 출금이 일어나고 그 돈이 B 계좌로 입금된다 했을 때, 트랜잭션의 전과 후 두 계좌 잔고의 합이 같아야 한다.

- 독립성(Isolation)

  > 둘 이상의 트랜잭션이 동시에 병행 실행되고 있을 때, 어떤 트랜잭션도 다른 트랜잭션 연산에 끼어들 수 없다.

- 지속성(Durability)

  > 트랜잭션이 성공적으로 완료되었으면, 결과는 영구적으로 반영되어야 한다.

### 트랜잭션 연산

- COMMIT

하나의 트랜잭션이 성공적으로 끝났고, DB가 일관성있는 상태일 때 이를 알려주는 연산

- ROLLBACK

트랜잭션 포함된  쿼리 중 하나라도 실패하면 모든 쿼리문을 취소하고 이전상태로 돌려놓음 -> 롤백하면 해당 트랜잭션 재시작하거나 폐기

### 트랜잭션 상태

![image](https://user-images.githubusercontent.com/93963499/162607638-61275162-752d-499f-9685-a9347518d597.png)

**1. 활성화(Active)** **:** 트랜잭션이 작업을 시작하여 실행 중인 상태

**2. 실패(Failed) :** 트랜잭션에 오류가 발생하여 실행이 중단된 상태

**3. 철회(Aborted) :** 트랜잭션이 비정상적으로 종료되어 Rollback 연산을 수행한 상태

**4. 부분 완료(Partially commited) :** 트랜잭션의 마지막 연산까지 실행하고 commit 요청이 들어온 직후의 상태. 최종 결과를 데이터베이스에 아직 반영하지 않은 상태.

**5. 완료(Commited) :** 트랜잭션이 성공적으로 종료되어 commit 연산을 실행한 후의 상태

#### 질문 1. 트랜잭션의 Commit 연산에 대해서 트랜잭션의 상태를 통해 설명해주세요.

트랜잭션이 시작되면 Active 상태가 되고, 모든 연산이 실행되면 Partially commited 상태가 된다. 그리고 Commit 연산을 실행하면 Commited 상태가 되어 트랜잭션을 성공적으로 종료한다.

#### 질문 2. 트랜잭션의 Rollback 연산에 대해서 트랜잭션의 상태를 통해 설명해주세요.

트랜잭션이 시작되면 Active 상태가 되고, 트랜잭션 중 오류가 발생하여 중단되면 실패(Failed) 상태가 된다. 그리고 Rollback 연산을 수행한 후, 철회(Aborted) 상태가 되어 트랜잭션이 종료되고 실행 이전 상태로 돌아간다.



## 트랜잭션 격리 수준 (Isolation level)

### Isolation level

트랜잭션에서 일관성 없는 데이터를 허용하도록 하는 수준

### Isolation level 필요성

트랜잭션 ACID의 독립성과 관련이 있다.

트랜잭션 독립성을 보장하기 위해 Locking을 통해, 트랜잭션이 DB를 다루는 동안 다른 트랜잭션이 관여하지 못하도록 막는 것이 필요하다.

하지만 무조건 Locking으로 동시에 수행되는 수많은 트랜잭션들을 순서대로 처리하는 방식으로 구현하게 되면 데이터베이스의 성능은 떨어지게 될 것이다.

그렇다고 해서, 성능을 높이기 위해 Locking의 범위를 줄인다면, 잘못된 값이 처리될 문제가 발생하게 된다.

따라서 최대한 효율적인 Locking 방법이 필요하다.

### Isolation level 종류

트랜잭션 격리(고립화) 수준이 중요한 이유는, 격리 레벨을 어떻게 설정하느냐에 따라 읽기 일관성이 달라지기 때문이다. 다시 말하면,  **트랜잭션 격리 수준에 따라 데이터 조회 결과가 달라질 수 있다는 말이다.**

이처럼 트랜잭션 격리 수준에 따라 데이터 조회 결과가 달라지게 하는 기술을 `MVCC(Multi Version Concurrency Consistency)` 라고 한다.

#### 레벨 0 : Read Uncommitted

- 트랜잭션에서 처리 중인, 아직 커밋 되지 않은 데이터를 다른 트랜잭션에서 읽는 것을 허용

> 트랜잭션1이 A라는 데이터를 B라는 데이터로 변경하는 동안 트랜잭션2는 아직 완료되지 않은(Uncommitted) 트랜잭션1이지만 데이터B를 읽을 수 있다.

- 데이터베이스의 일관성을 유지하는 것이 불가능함
- 발생할 수 있는 문제
  - `Dirty Read, Non-Repeatable Read, Phantom Read 현상 발생`
  - 예시에서는 Dirty Read 발생

~~~
1. A 트랜잭션에서 10번 사원의 나이를 27살에서 28살로 바꿈
2. 아직 커밋하지 않음
3. B 트랜잭션에서 10번 사원의 나이를 조회함
4. 28살이 조회됨 ->  이를 더티 리드(Dirty Read)라고 한다
5. A 트랜잭션에서 문제가 발생해 ROLLBACK함
6. B 트랜잭션은 10번 사원이 여전히 28살이라고 생각하고 로직을 수행함
~~~

데이터 정합성에 문제가 많으므로, RDBMS 표준에서는 격리수준으로 인정하지도 않는다.

#### 레벨 1 : Read Committed

- 커밋된 데이터를 다른 트랜잭션이 조회하는 것을 허용
- SQL 서버가 Default로 사용하는 Isolation Level임

> 트랜잭션1이 A라는 데이터를 B라는 데이터로 변경하는 동안 트랜잭션2는 해당 데이터에 접근이 불가능함

- 발생할 수 있는 문제
  - Non-Repeatable Read, Phantom Read 현상은 여전히 발생
  - 예시에서는 `NON-REPETABLE READ` 부정합 문제가 발생

~~~
1. B 트랜잭션에서 10번 사원의 나이를 조회
2. 27살이 조회됨
3. A 트랜잭션에서 10번 사원의 나이를 27살에서 28살로 바꾸고 커밋
4. B 트랜잭션에서 10번 사원의 나이를 다시 조회(변경되지 않은 이름이 조회됨)
5. 28살이 조회됨
~~~

하나의 트랜잭션 내에서 똑같은 SELECT를 수행했을 경우 항상 같은 결과를 반환해야 한다는 REPEATABLE READ 정합성에 어긋나는 것이다.

일반적으로 큰 문제가 되지 않을 수도 있으나, 금융쪽에서는 문제가 발생할 수 있다.

예를 들어 여러 트랜잭션에서 입금/출금 처리가 계속 진행되는 트랜잭션들이 있고 오늘의 입금 총 합을 보여주는 트랜잭션이 있다고하면, 총합을 계산하는 SELECT 쿼리는 실행될 때 마다 다른 결과값을 가져올 것이다.

#### 레벨 2 : Repeatable Read

- **트랜잭션이 시작되기 전에 커밋된 내용에 대해서만 조회할 수 있는 격리수준**
- MySQL  InnoDB 에서 기본으로 사용

> 트랜잭션1 이 읽은 데이터는 트랜잭션1이 종료될 때가지 트랜잭션2이 `갱신하거나 삭제하는 것은 불허함`으로써 같은 데이터를 두 번 쿼리했을 때 일관성 있는 결과를 리턴, 하지만 삽입은 가능

- 발생할 수 있는 문제
  - Phantom Read 현상은 여전히 발생
  - 이 예시는 InnoDB 에서 발생한 Non-Repeatable Read 발생

~~~
1. 10번 트랜잭션이 500000번 사원을 조회
2. 12번 트랜잭션이 500000번 사원의 이름을 변경하고 커밋
3. 10번 트랜잭션이 500000번 사원을 다시 조회
4. 백업된 데이터 반환
5. 자신의 트랜잭션 번호보다 낮은 트랜잭션 번호에서 변경된(+커밋된) 결과 조회됨
~~~

REPETABLE READ 격리수준에서는 트랜잭션이 시작된 시점의 데이터를 일관되게 보여주는 것을 보장해야 하기 때문에 한 트랜잭션의 실행시간이 길어질수록 해당 시간만큼 계속 멀티 버전을 관리해야 하는 단점이 있다. (?)

#### 레벨 3 : Serializable Read

- 가장 단순하고 가장 엄격한 격리수준이다.

- 완벽한 읽기 일관성 모드를 제공함

- 다른 사용자는 트랜잭션 영역에 해당되는 데이터에 대한 수정 및 입력 불가능

>  트랜잭션1 이 읽은 데이터를 트랜잭션2 이 갱신하거나 삭제하지 못할 뿐만 아니라 중간에 새로운 레코드를 삽입하는 것도 막아줌

이러한 특성 때문에 동시처리 능력이 다른 격리수준보다 떨어지고, 성능저하가 발생하게 된다.

## 트랜잭션 격리 수준 설정할 때 발생하는 문제점들

트랜잭션 격리 수준을 너무 낮게(0 레벨)하면 읽기 일관성을 제대로 보장할 수 없고, 반면 너무 높게하면, 읽기 일관성은 완벽하게 보장하지만 데이터를 처리하는 속도가 느려지게 된다.

- Read Uncommitted > Read Committed > Repeatable Read > Serializable Read
  - ReadUncommitted 로 갈 수록 동시성은 높아지고, 일관성은 떨어진다.
  - Serializable Read 로 갈 수록 동시성은 떨어지고, 일관성은 높아진다.

- 따라서, 트랜잭션 격리 수준은 `일관성` 및 `동시성`과도 연관이 있다는 것을 알 수있다.

### 낮은 단계 Isolation Level을 활용할 때 발생하는 현상들

- **Dirty Read**

  커밋되지 않은 수정중인 데이터를 다른 트랜잭션에서 읽을 수 있도록 허용할 때 발생하는 현상

  어떤 트랜잭션에서 아직 실행이 끝나지 않은 다른 트랜잭션에 의한 변경사항을 보게되는 경우

- **Non-Repeatable Read**

  한 트랜잭션에서 같은 쿼리를 두 번 수행할 때 그 사이에 다른 트랜잭션 값을 수정 또는 삭제하면서 두 쿼리의 결과가 상이하게 나타나는 일관성이 깨진 현상

- **Phantom Read**

  한 트랜잭션 안에서 일정 범위의 레코드를 두 번 이상 읽었을 때, 첫번째 쿼리에서 없던 레코드가 두번째 쿼리에서 나타나는 현상

  트랜잭션 도중 새로운 레코드 **삽입을 허용하기 때문에** 나타나는 현상임 

  ![image](https://user-images.githubusercontent.com/93963499/162610051-3c4f66e6-62ff-40d9-8a10-2087b62d69dc.png)

## QnA



### Reference

- https://github.com/NKLCWDT/cs/blob/main/Database/Transaction.md
- https://joont92.github.io/db/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98-%EA%B2%A9%EB%A6%AC-%EC%88%98%EC%A4%80-isolation-level/

- https://gyoogle.dev/blog/computer-science/data-base/Transaction%20Isolation%20Level.html

- https://github.com/Seogeurim/CS-study/tree/main/contents/database

- https://rebro.kr/162