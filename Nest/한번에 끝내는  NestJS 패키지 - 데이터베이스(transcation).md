![alt text](/Nest/image/codeFactory.png)
출처 : https://fastcampus.co.kr/dev_online_nestjs

## Transaction

  <br>

- Transaction은 여러 오퍼레이션을 하나의 논리적인 작업으로 실행하는 기능이다.

  <br>

![alt text](/Nest/image/image14.png)

<br>

![alt text](/Nest/image/image15.png)

<br>

![alt text](/Nest/image/image16.png)

<br>

![alt text](/Nest/image/image17.png)

<br>

## Lost Update

<br>

- 두개의 트랜잭션이 같은 데이터를 읽고 업데이트한다.
- 나중에 진행된 트랜잭션이 먼저 진행된 트랜잭션의 결과를 덮어쓴다.
- 먼저 진행된 트랜잭션의 작업은 유실된다.
- Optimistic Lock 전략으로 해결 가능하다.

<br>

```sql
-- 초기상태
-- Table: account
-- | id | balance |
-- |----+---------|
-- | 1  | 1000    |

-- Transaction 1
BEGIN TRANSACTION;
SELECT balance FROM account WHERE id = 1; -- 1000 반환
UPDATE account SET balance = balance - 100 WHERE id = 1; -- 900으로 변경

-- Transaction 2 (happens concurrently)
BEGIN TRANSACTION;
SELECT balance FROM account WHERE id = 1; -- Returns 1000
UPDATE account SET balance = balance - 200 WHERE id = 1; -- 800으로 변경

-- Transaction 1 완료
COMMIT; -- Balance = 900

-- Transaction 2 완료
COMMIT; -- Balance = 800

-- Final State
-- | id | balance |
-- |----+---------|
-- | 1  | 800     |
```

<br>

## Dirty Read

<br>

- 아직 커밋되지 않은 다른 트랜잭션의 데이터를 읽었을때 생기는 문제다.
- 변경한 데이터를 커밋하지 않고 롤백할 경우 롤백전에 데이터를 읽은 다른 트랜잭션은 잘못된 정보로 로직을 진행한다.
- Read Committed 트랜잭션으로 해결 가능하다.

<br>

```sql
-- 초기상태
-- Table: account
-- | id | balance |
-- |----+---------|
-- | 1  | 1000    |

-- Transaction 1
BEGIN TRANSACTION;
UPDATE account SET balance = balance - 100 WHERE id = 1; -- balance = 900

-- Transaction 2
BEGIN TRANSACTION;
SELECT balance FROM account WHERE id = 1; -- 900 반환

-- Transaction 1 롤백
ROLLBACK; -- Balance 1000으로 되돌림

-- Transaction 2 완료
-- Transaction 2 에서 읽은 balance 값은 잘못된 값임.
```

<br>

## Non-repeatable Reads

- 트랜잭션이 데이터를 읽은 상태에서 다른 트랜잭션이 데이터를 변경할경우 같은 데이터를 다시 읽었을때 기존 읽었던 데이터가 재구현되지 않는 현상을 이야기한다.
- Repeatable Read 트랜잭션으로 해결 가능하다.

<br>

```sql
-- 초기상태
-- Table: account
-- | id | balance |
-- |----+---------|
-- | 1  | 1000    |

-- Transaction 1
BEGIN TRANSACTION;
SELECT balance FROM account WHERE id = 1; -- 1000 반환

-- Transaction 2
BEGIN TRANSACTION;
UPDATE account SET balance = balance - 100 WHERE id = 1; -- balance = 900
COMMIT;

-- Transaction 1 continues
SELECT balance FROM account WHERE id = 1; -- 900 반환 (non-repeatable read)
COMMIT;
```

<br>

## Phantom Reads

- 트랜잭션이 여러 Row를 불러오는 필터링 쿼리를 진행 후 다른 트랜잭션에서 쿼리의 조건에 맞는 새로운 데이터를 생성 했을때 같은 쿼리가 다른 결과를 반환하는 것을 이야기한다.
- Serializable 트랜잭션으로 해결 가능하다.

<br>

```sql
-- 초기상태
-- Table: account
-- | id | balance |
-- |----+---------|
-- | 1  | 1000    |
-- | 2  | 1500    |

-- Transaction 1
BEGIN TRANSACTION;
SELECT * FROM account WHERE balance > 1000; -- account 2 반환

-- Transaction 2
BEGIN TRANSACTION;
INSERT INTO account (id, balance) VALUES (3, 1200);
COMMIT;

-- Transaction 1
SELECT * FROM account WHERE balance > 1000; -- account 2 and account 3 반환 (phantom read)
COMMIT;
```

![alt text](/Nest/image/image18.png)

## Transaction 문법

```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN TRANSACTION;
-- SQL 작업하기
COMMIT;
```
