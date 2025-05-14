# 스프링 데이터 JPA를 이용한 조회 기능


## 시작에 앞서
- CQRS : 아래 두 모델을 분리하는 패턴
  - 명령(Command) 모델
    - 상태를 변경하는 기능을 구현
  - 조회(Query) 모델
    - 데이터를 조회하는 기능을 구현

## 검색을 위한 스펙
- 검색 조건을 다양하게 조합해야 할 때 사용할 수 있는것이 바로 스펙(Specification)
<pre>
<code>
public interface Specification<T> {
    public boolean isSatisfiedBy(T agg);
}
</code>
</pre>
- isSatisfiedBy() 메서드의 agg는 검사 대상이 되는 객체
  - 검사 대상 객체가 조건을 충족하면 true를 리턴
<pre>
<code>
public class OrdererSpec implements Specification<Order> {
    private String ordererId;

    public OrdererSpec(String ordererId) {
        this.ordererId = ordererId;
    }

    public boolean isSatisfiedBy(Order agg) {
        return agg.getOrdererId().getMemberId().getId().equals(ordererId);
    }
}
</code>
</pre>

## 스프링 데이터 JPA를 이용한 스펙 구현
<pre>
<code>
public class OrderSummarySpecs {
    public static Specification<OrderSummary> ordererId(String ordererId) {
        return (Root<OrderSummary> root, CriteriaQuery<?> query, CriteriaBuilder cb) -> 
                cb.equals(root.<String>get("ordererId"), ordererId);
    }
    ...

}
</code>
</pre>

## 리포지터리/DAO에서 스펙 사용하기
- findAll(spec)

## 스펙 조합
- and() / or() ...
- spec1.and(spec2)

## 정렬 지정하기
- 메서드 이름에 OrderBy를 사용해서 정렬 기준 지정
  - findByXXXOrderByYYYDesc
- Sort를 인자로 전달
  - Sort.by("number").ascending()
  - findByXXX(xxx, sort)

## 페이징 처리하기
- Pageable
  - PageRequest.of(1, 10)
    - 1 : 페이지 번호 (0부터 시작)
    - 10 : 한 페이지의 개수
    - sort 도 파라미터로 받을 수 있음.
    - 전체 개수도 구할 수 있음
  - 리턴 타입이 Page일 경우 count 쿼리도 실행.

## 스펙 조합을 위한 스펙 빌더 클래스
<pre><code>
SpecBuilder.builder(MemberData.class)
.ifTrue(searchRequest.isOnlyNotBlocked(), () -> MemberDataSpecs.nonBlocked())
.ifHasText(...)
.toSpec();
</code></pre>

## 동적 인스턴스 생성
- JPQL 에 select절 에 new Dto로 값을 받을 수 있음(생성자 방식)

## 하이버네이트 @Subselect
<pre>
<code>
@Entity
@Immutable
@Subselect(
    """
        select 컬럼
          from 테이블
          join.. where...
    """
)
@Synchronize({"purchase_order", "order_line", "product"})
public class OrderSummary {
    ...
}
</code>
</pre>

- @Subselect
  - View 기능 / 수정 불가
- @Immutable
  - 더티체킹 안함 -> DB 반영 않고 무시
- @Synchronize
  - 엔티티를 로딩하기 전에 지정한 테이블과 관련된 변경이 발생하면 flush를 먼저 진행.
- @Entity와 같기 때문에 EntityManager를 이용한 조회 기능을 사용할 수 있다는 장점이 있음.