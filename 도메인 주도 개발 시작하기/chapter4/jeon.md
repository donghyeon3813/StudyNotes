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
- 밸류 타입의 프로퍼티를 한 개의 컬럼에 매핑하기 위함.
<pre>
<code>
public class Length { 
  private int value;
  private String unit;
}
// WIDTH VARCHAR(20)
// 1000mm 로 저장
</code>
</pre>

- 두 개 이상의 프로퍼티를 가진 밸류 타입을 한 개 컬럼에 매핑하려면 @Embeddable 애너테이션으로는 처리 불가.
  - @AttributeConverter 사용

<pre>
<code>
package javax.persistence;

public interface AttributeConverter< X, Y> {
  public Y convertToDatabaseColumn(X attribute);  // 밸류 -> DB
  public X convertToEntityAttribute(Y data);       // DB -> 밸류
}
</code>
</pre>
- X : 밸류 타입
- Y : DB 타임

<pre>
<code>
@Converter(autoApply = true)
public class MoneyConverter implements AttributeConverter< Money, Integer> {
  
  @Override
  public Integer convertToDatabaseColumn(Money money) {
    return money == null ? null : money.getValue();
  }

  @Override
  public Money convertToEntityAttribute(Money money) {
    return value == null ? null : new Money(value);
  }
}
</code>
</pre>

- AttributeConverter 인터페이스를 구현한 클래스는 @Converter 애너테이션을 적용.
- @autoApply
  - true : 자동으로 적용
  - false : 프로퍼티 값을 변환할 때 사용할 컨버터를 직접 지정해야 함.

<pre>
<code>
@Entity
@Table(name = "purchase_order")
public class Order {
  @Column(name = "total_amounts")
  @Convert(converter = MoneyConverter.class)
  private Money totalAmounts;
}
</code>
</pre>


## 밸류 컬렉션 : 별도 테이블 매핑
- 밸류 컬렉션을 별도 테이블로 매핑할 때는 @ElementCollection / @CollectionTable을 함께 사용한다.
<pre>
<code>
@ElementCollection(fetch = FetchType.EAGER)
@CollectionTable(name = "order_line", joinColumns = @JoinColumn(name = "order_number"))
@OrderColumn(name = "line_idx")
private List<OrderLine> orderLines;

@Embeddable
public class OrderLine {
  @Embedded
  private ProductId productId;

  @Column(name="price")
  private Money price;
  ...
}
</code>
</pre>

## 밸류 컬렉션 : 한 개 컬럼 매핑
- @AttributeConverter

## 밸류를 이용한 ID 매핑
- @Id 대신 @EmbeddedId
- JPA에서 식별자 타입은 Serializable 타입이어야 하므로 식별자로 사용할 밸류 타입은 Serializable 인터페이스를 상속받아야 한다.
- 밸류 타입으로 식별자를 구현할 때 얻을 수 있는 장점
  - 식별자에 기능을 추가할 수 있다.

## 별도 테이블에 저장하는 밸류 매핑
- 애그리거트에서 루트 엔티티를 뺀 나머지 구성요소는 대부분 밸류.
  - 루트 엔티티 외에 다른 엔티티가 있다면 진짜 엔티티인지 의심해봐야 함.
    - 단지 별도 테이블에 데이터를 저장한다고 해서 엔티티인 것은 아님!!!
  - 자신만의 독자적인 라이프 사이클을 갖는다면 구분되는 애그리거트일 가능성이 높다.
    - Product / Review : 라이프사이클이 다르고, 변경 주체도 다름.
- 밸류 : @Embeddable로 매핑
  - 이 때 밸류를 매핑 한 테이블을 지정하기 위해 @SecondaryTable / @AttributeOverride 사용

<pre>
<code>
@Entity
@Table(name = "article")
@SecondaryTable(
  name = "article_content",
  pkJoinColumns = @PrimaryKeyJoinColumn(name = "id")
)
public class Article {
  ...

  @AttributeOverrides({
    @AttributeOverride(
      name = "content",
      column = @Column(table = "article_content", name = "content)),
    @AttributeOverride(
      name = "contentType",
      column = @Column(table = "article_content", name = "content_type"))
  })
  @Embedded
  private ArticleContent content;
}

// @SecondaryTable 로 매핑된 article_content 테이블을 조인
Article article = entityManager.find(Article.class, 1L);
// N+1 문제 발생 -> 지연로딩 방식 설정 가능 
// 그러나, 밸류인 모델을 엔티티로 만드는 것이므로 좋은 방법은 아니다. 조회 전용 기능을 구현하는 방법을 사용하는 것이 좋다. -> 5장
</code>
</pre>

## 밸류 컬렉션을 @Entity로 매핑하기

