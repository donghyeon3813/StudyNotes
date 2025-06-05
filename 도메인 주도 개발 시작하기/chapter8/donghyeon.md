# 8. 애그리거트와 트랜잭션

## 8.1 애그리거트와 트랜잭션

- 여러 트랜잭션이 동시에 같은 애그리거트를 수정하려 하면 동시성 문제가 발생할 수 있으며, 이로 인해 데이터 일관성이 깨질 수 있다.
- 이를 해결하기 위해 선점 잠금(Pessimistic Locking)과 비선점 잠금(Optimistic Locking) 방식을 사용할 수 있다.

---

## 8.2 선점 잠금

- 선점 잠금은 먼저 애그리거트를 점유한 스레드가 작업을 마칠 때까지 다른 스레드가 해당 애그리거트를 수정하지 못하도록 막는 방식이다.
- JPA에서는 `EntityManager`의 `LockModeType`을 이용해 선점 잠금을 구현할 수 있다.

```java
entityManager.find(MyEntity.class, id, LockModeType.PESSIMISTIC_WRITE);
```

- 하이버네이트는 `PESSIMISTIC_WRITE`를 사용할 경우 내부적으로 `SELECT ... FOR UPDATE` 쿼리를 실행한다.
- 잠금으로 인해 교착 상태(Deadlock)가 발생할 수 있으므로, 잠금 시간 제한을 설정하는 것이 좋다.

```java
Map<String, Object> props = new HashMap<>();
props.put("javax.persistence.lock.timeout", 5000); // 최대 5초 대기
entityManager.find(MyEntity.class, id, LockModeType.PESSIMISTIC_WRITE, props);
```

---

## 8.3 비선점 잠금

- 비선점 잠금은 동시에 접근이 허용되지만, 최종 업데이트 시점에 버전을 비교하여 충돌 여부를 확인하는 방식이다.
- 애그리거트에 버전(version) 필드를 추가하고, JPA의 `@Version` 어노테이션을 사용한다.

```java
@Version
private Long version;
```

- 업데이트 시점에 버전이 불일치하면 예외가 발생하여 충돌을 감지할 수 있다.
- 예외 예시:
  - `OptimisticLockException` (JPA 표준)
  - `ObjectOptimisticLockingFailureException` (Spring Data JPA)

- 주의할 점은, 애그리거트 루트가 아닌 하위 엔티티만 수정되는 경우 `@Version`이 제대로 동작하지 않을 수 있다. 이 경우 루트 엔티티에서 버전을 강제로 증가시키는 방식으로 동시성을 제어할 수 있다.

---

## 8.4 오프라인 선점 잠금

- 오프라인 선점 잠금은 클라이언트가 애그리거트를 편집하는 동안, 다른 사용자의 수정 요청 자체를 차단하는 방식이다.
- 일반적으로 UI 상에서 '수정 중' 상태로 표현되며, 서버에서는 해당 리소스에 대한 잠금 상태를 유지한다.

### 구현 방식

1. **DB 기반**
   - 별도의 잠금 테이블에 리소스 ID, 사용자 ID, 잠금 시작 시간 등을 저장한다.
2. **캐시 기반 (예: Redis)**
   - Redis의 `SETNX` 명령어와 TTL을 사용하여 잠금을 구현할 수 있다.

```text
SETNX lock:order:1234 user1 EX 300  // 5분간 잠금
```

### 주의 사항

- 클라이언트가 비정상 종료되거나 잠금 해제를 하지 않으면 영구적으로 잠긴 상태가 될 수 있다.
- 이를 방지하기 위해 반드시 TTL(잠금 유효시간)을 설정해야 하며, 주기적으로 잠금 상태를 확인하고 정리하는 작업이 필요하다.
