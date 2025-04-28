# 리포지터리와 모델 구현

## JPA를 이용한 리포지터리 구현

- RDBMS 사용 시, ORM만한게 없음.

### 모듈 위치
- 리포지터리 인터페이스 : 애그리거트와 함께 도메인 영역
- 구현 클래스 : 인프라스트럭처 영역
- 팀 표준에 따라 구현 클래스를 domain.impl과 같은 패키지 가능 (타협안 같은거임 좋은 설계 원칙을 따르는 것은 아님)
- 인프라스트럭처에 대한 의존을 낮추자

### 리포지터리 기본 기능 구현
- ID로 애그리거트 조회하기
- 애그리거트 저장하기
<pre>
<code>
public interface OrderRepository {
    Order findById(OrderNo no);
    void save(Order order);
}
</code>
</pre>

- 인터페이스는 애그리거트 루트를 기준으로 작성한다.
  - OrderLine, Orderer, ShippingInfo 등 다양한 객체를 포함하는데, 이 구성요소 중에서 루트 엔티티인 Order를 기준으로 리포지터리 인터페이스를 작성한다.
- findBy프로퍼티이름(프러퍼티 값) : 널리 사용되는 규칙.
- null 사용 싫다면 Optional

- 애그리거트를 수정한 결과를 저장소에 반영하는 메서드를 추가할 필요는 없다.
- JPA를 사용하면 트랜잭션 범위에서 변경한 데이터를 자동으로 DB에 반영하기 때문....
<pre>
<code>
public class ChangeOrderService {
  @Transactinoal
  public void changeShippingInfo(OrderNo no, ShippingInfo newShippingInfo) {
    Optional<Order> orderOpt = orderRepository.findById(no);
    Order order = orderOpt.orElseThrow(() -> new OrderNotFoundException());
    order.changeShippingInfo(newShippingInfo);
  }
}
</code>
</pre>

- ID 외에 다른 조건으로 애그리거트를 조회할 때에는 JPA의 Criteria나 JPQL도 사용 가능

## 스프링 데이터 JPA를 이용한 리포지터리 구현
- JPARepository<T, ID> 인터페이스 상속
- T는 엔티티 타입을 지정하고 ID는 식별자 타입을 지정.

... > 이건 인프라스트럭처,

## 매핑 구현

### 엔티티와 밸류 기본 매핑 구현
- 애그리거트 루트는 엔티티이므로 @Entity로 매핑 설정한다.
- 밸류는 @Embeddable로 매핑 설정한다.
- 밸류 타입 프로퍼티는 @Embedded로 매핑 설정한다.
<pre>
@Entity
@Table(name = "purchase_order")
public class Order {
  ...
  @Embedded
  private Orderer orderer;

  @Embedded
  private ShippingInfo shippingInfo;
  ...
}

@Embeddable
public class Orderer {
  // MemberId에 정의된 컬럼 이름을 변경하기 위해 @AttributeOverride 사용.
  @Mebedded
  @AttributeOverrides(
    @AttributeOverride(name = "id", column = @Column(name = "orderer_id"))
  )
  private MemberId memberId;

  @Column(name = "orderer_name")
  private String name;
}

@Embeddable
public class MemberId implements Serializable {
  @Column(name = "member_id")
  private String id;
}

@Embeddable
public class ShippingInfo {
  @Embedded
  ...
}
</pre>

- 주문 애그리거트에서 루트 엔티티 Order : @Entity
- Orderer 밸류 : @Embeddable
- memberId는 Member 애그러기트를 ID로 참조.
- 루트 엔티티인 Order 클래스는 @Embedded를 이용해서 밸류 타입 프로퍼티를 설정한다.

### 기본 생성자
- 밸류타입과 같은 밸류는 불변 타입 -> 기본 생성자 필요 없지
- 하지만, JPA에서는 @Entity와 @Embeddable로 클래스를 매핑하려면 기본 생성자를 제공해야 함.
  - DB에서 데이터를 읽어와 매핑된 객체를 생성할 때 기본 생성자를 사용해서 객체를 생성하기 때문.
  - 기본 생성자는 JPA 프로바이더가 객체를 생성할 때만 사용한다.
  - 접근제어자 `protected`


### 필드 접근 방식 사용
- get/set 메서드를 추가하면 도메인의 의도가 사라지고 객체가 아닌 데이터 기반으로 엔티티를 구현할 가능성이 높아진다.
- 특히 set : 캡슐화를 깨는 원인이 됨.
  - 대신 의도가 잘 드러나는 기능을 제공해야 함.
- 객체가 제공할 기능 중심으로 엔티티를 구현하게끔 유도하려면 JPA 매핑 처리를 프로퍼티 방식이 아닌 필드 방식으로 선택해서 불필요한 get/set 메서드를 구현하지 말아야 함.
<pre>
<code>
@Entity
@Access(AccessType.FIELD)
public class Order {
  @EmbeddedId
  private OrderNo nubmer;

  @Colunm(name = "state")
  @Enumerated(EnumType.STRING)
  private OrderState state;

  // cancel(), changeShippingInfo() 등 도메인 기능 구현
  // 필요한 get 메서드 제공.
}
</code>
</pre>
- JPA 구현체인 하이버네이트는 @Access 를 이용해서 명시적으로 접근 방식을 지정하지 않으면 @Id나 @EmbeddedId가 어디에 위치했느냐에 따라 접근 방식을 결정한다.
- @Id나 @EmbeddedId가 필드에 위치하면 필드 접근 방식을 선택하고 get 메서드에 위치하면 메서드 접근 방식을 선택한다.
  - ` @Access는 결국 get 메서드에 비즈니스 로직이 있는 경우 JPA 하이버네이트에서 필드를 읽어서 처리하도록 하기 위한건가??`

### AttributeConverter를 이용한 밸류 매핑 처리
