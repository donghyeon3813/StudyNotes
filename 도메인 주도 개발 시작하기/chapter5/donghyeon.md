
# ✅ 도메인 주도 개발 시작하기 - 5장 정리

---

## 5.1 시작에 앞서 - CQRS

- **CQRS (Command Query Responsibility Segregation)**  
  → 명령 모델과 조회 모델을 **분리**하는 패턴  
  → 복잡한 도메인에서 읽기/쓰기 요구사항이 다르므로, **명령은 트랜잭션 처리**, **조회는 성능 최적화**에 집중 가능

---

## 5.2 검색을 위한 스펙

- 단순 검색 조건은 `findByName(String name)` 등으로 처리
- 조건이 복잡하고 다양할 경우 → **`Specification<T>` 인터페이스** 사용
- JPA Criteria API 기반으로 동적 조건 생성 가능

```java
Specification<Member> spec = Specification.where(nameEquals("홍길동")).and(cityEquals("서울"));
```

⚠️ **실무에서는 조회 성능 저하 우려 → 과도한 사용 주의**

---

## 5.3 스프링 데이터 JPA를 이용한 스펙 구현

- `Specification<T>` 는 **람다식 사용 가능 (함수형 인터페이스)**

```java
Specification<Member> spec = (root, query, cb) -> cb.equal(root.get("name"), "홍길동");
```

- 제네릭 타입 `T`는 JPA 엔티티 타입

---

## 5.4 리포지터리/DAO에서 스펙 사용하기

- 스펙을 파라미터로 넘겨서 동적 조건 검색 가능

```java
List<Member> result = memberRepository.findAll(spec);
```

- 다양한 스펙 조합이 필요한 경우, 이 방식으로 간단히 해결

---

## 5.5 스펙 조합

- `Specification` 의 `and()`, `or()` 를 활용해 스펙을 조합

```java
Specification<Member> spec = spec1.and(spec2).or(spec3);
```

- **조건이 null일 경우 생략하도록 구현하면 유연한 조합 가능**

---

## 5.6 정렬 지정하기

- **정렬이 고정된 경우: 메서드명에 `OrderBy` 사용**

```java
List<Order> findByOrdererIdOrderByNumberDesc(String ordererId);
```

- **정렬이 동적인 경우: `Sort` 객체 사용**

```java
Sort sort = Sort.by(Sort.Direction.DESC, "number");
List<Order> orders = orderRepository.findByOrdererId("user123", sort);
```

---

## 5.7 페이징 처리하기

- **Pageable 객체 사용**

```java
Page<Order> result = orderRepository.findByOrdererId("user123", PageRequest.of(0, 10));
```

- 리턴 타입이 `Page<T>`이면 → count 쿼리도 함께 실행
- 리턴 타입이 `List<T>`이면 → count 쿼리 없이 목록만 반환

---

## 5.8 스펙 조합을 위한 스펙 빌더 클래스

- 조건에 따라 동적으로 스펙을 조합할 수 있는 **Builder 클래스** 활용

```java
public class MemberSpecBuilder {
    public static Specification<Member> build(String name, String city) {
        Specification<Member> spec = Specification.where(null);
        if (name != null) spec = spec.and(nameEquals(name));
        if (city != null) spec = spec.and(cityEquals(city));
        return spec;
    }
}
```

➡ 코드 가독성과 유지보수성 ↑

---

## 5.9 동적 인스턴스 생성 (JPQL의 DTO 매핑)

- JPQL에서 DTO를 직접 생성하여 반환할 수 있음

```java
SELECT new com.example.dto.MemberDto(m.id, m.name) FROM Member m
```

- **지연 로딩 이슈 없이 필요한 데이터만 조회 가능**
- 표현 계층에 맞는 DTO 구조 제공 가능

---

## 5.10 하이버네이트 @Subselect 사용

- `@Subselect`를 이용해 **쿼리 결과를 읽기 전용 엔티티**로 매핑 가능

```java
@Entity
@Subselect("SELECT o.id AS id, o.status AS status FROM orders o WHERE o.status = 'SHIPPED'")
@Immutable
@Synchronize("orders")
public class ShippedOrderView { ... }
```

- `@Immutable`: 읽기 전용 → DB 반영 안 됨
- `@Synchronize`: 엔티티 변경 사항을 flush 후 조회
- ⚠ JPA 표준 아님 → 네이티브 SQL 또는 MyBatis로 대체 가능

---
