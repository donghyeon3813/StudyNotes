# 도메인 모델 시작하기
## 도메인이란?

- 도메인 : 소프트웨어로 해결하고자 하는 문제 영역
  - ex) 온라인 서점에서 책을 판매하는 데 필요한 상품 조회, 구매, 결제, 배송 추적 등의 기능들
- 한 도메인은 다시 하위 도메인으로 나눌 수 있다.

## 도메인 전문가와 개발자 간 지식 공유
- 온라인 홍보, 정산, 배송 등 각 영역에는 전문가가 존재.
  - 이들 전문가는 해당 도메인에 대한 지식과 경험을 바탕으로 본인들이 원하는 기능 개발을 요구.
    - 회계 담당자 : 정산 금액 계산을 자동화 하는 기능 요구
    - AS 기사 : 고객에게 보내는 문자 메시지 템플릿 기능 요구
  - 개발자는 이런 요구사항을 분석하고 설계하여 코드를 작성하며 테스트하고 배포한다.
  - 이 과정에서 `요구사항`은 첫 단추와 같다.
  - 코딩에 앞서 요구사항을 올바르게 이해하는 것이 매우 중요
    - 요구사항을 잘못 이해 => 다시 만들어야 할 코드가 많아지고, 일정이 밀리고 등등 안좋은 일이 아주 많음

>요구사항을 올바르게 이해하려면?
- 비교적 간단한 방법은 개발자와 전문가가 직접 대화하는 것
  - 개발자와 전문가 사이에 내용을 전파하는 전달자가 많으면 많을수록 정보가 왜곡되고 손실이 발생하게 되며, 개발자는 최초에 전문가가 요구한 것과는 다른 무언가를 만들게 됨.
- 도메인 전문가 만큼은 아니겠지만 이해관계자와 개발자도 도메인 지식을 갖춰야 함.

## 도메인 모델
- 기본적으로 도메인 모델은 특정 도메인을 개념적으로 표현한 것.
- 도메인 모델을 사용하면 여러 관계자들이 동일한 모습으로 도메인을 이해하고 도메인 지식을 공유하는 데 도움이 된다.
    - 객체 모델
    - 다이어그램
    - 그래프를 이용한 모델링
    - 수학 공식을 활용한 모델링
    - 도메인을 이해하는 데 도움이 된다면 표현 방식이 무엇인지는 중요하지 않음.
- 도메인 모델은 기본적으로 도메인 자체를 이해하기 위한 개념 모델
- 이를 이용해서 바로 코드를 작성할 수 있는 것은 아니기에 구현 기술에 맞는 구현 모델이 따로 필요.
- 개념 모델과 구현 모델은 서로 다른 것이지만 구현 모델이 개념 모델을 최대한 따르도록 할 수 는 있다.

> - 각 하위 도메인이 다루는 영역은 서로 다르기 때문에 같은 용어라도 하위 도메인마다 의미가 달라질 수 있다.
> - 도메인에 따라 용어 의미가 결정되므로 여러 하위 도메인을 하나의 다이어그램에 모델링하면 안 된다.
> - 모델의 각 구성요소는 특정 도메인으로 한정할 때 비로소 의미가 완전해지기 때문에 각 하위 도메인마다 별도로 모델을 만들어야 한다.
> ex) 카탈로그 하위 도메인 모델과 배송 하위 도메인 모델 - 두 도메인 모델에서의 상품(상세 내용 / 배송)의 의미가 다름.

## 도메인 모델 패턴
- 일반적인 애플리케이션의 아키텍처
  - 사용자 - 표현 - 응용 - 도메인 - 인프라스트럭처 - DB

| 영역                               | 설명                                                                         |
|:---------------------------------|----------------------------------------------------------------------------|
| 사용자 인터페이스(UI) or 표현 Presentation | 사용자의 요청을 처리하고 사용자에게 정보를 보여준다. 여기서 사용자는 소프트웨어를 사용하는 사람뿐만 아니라 외부 시스템일 수도 있다. |
| 응용 Application                   | 사용자가 요청한 기능을 실행한다. 업무 로직을 직접 구현하지 않으며 도메인 계층을 조합해서 기능을 실행한다.               |
| 도메인                              | 시스템이 제공할 도메인 규칙을 구현한다.                                                     |
| 인프라스트럭처 Infrastructure           | 데이터베이스나 메시징 시스템과 같은 외부 시스템과의 연동을 처리한다.                                     |

- 도메인 모델은 아키텍처 상의 도메인 계층을 객체 지향 기법으로 구현하는 패턴을 말한다.
- **도메인 계층은 도메인의 핵심 규칙을 구현한다.**

<pre>
    <code>
public class Order {
    private OrderState state;
    private ShippingInfo shippingInfo;
    
    public void changeShippingInfo(ShippingInfo newShippingInfo) {
        if (!state.isShippingChangeable()) {
            throw new ~~
        }
        this.shippingInfo = new ShippingInfo;
    }
    ...
}

public enum OrderState {
    PAYMENT_WAITING {
        public boolean isShippingChangeable() {
            return true;
        }
    },
    PREPARING {
        public ... {}
    },
    SHIPPED, DELIVERING, DELIVERY_COMPLETED;
    
    public boolean isShippingChangeable() {
        return false;
    }

}
    </code>
</pre>
    
- 이 코드는 주문 도메인의 일부 기능을 도메인 모델 패턴으로 구현한 것.
- OrderState 는 주문 대기 중이거나 상품 준비 중에는 배송지를 변경할 수 있다는 도메인 규칙을 구현하고 있음.
- 큰 틀에서 보면 OrderState 는 Order 에 속한 데이터이므로 배송지 정보 변경 가능 여부를 판단하는 코드를 Order로 이동할 수도 있다.
<pre>
<code>
public class Order {
    private OrderState state;
    private ShippingInfo shippingInfo;

    public void changeShippingInfo(ShippingInfo newShippingInfo) {
        if (!isShippingChangeable()) {
            throw new ~~
        }
        this.shippingInfo = new ShippingInfo;
    }

    private boolean isShippingChangeable() {
        return state == OrderState.PAYMENT_WAITING || OrderState.PREPARING;
    }
    ...
}
</code>
</pre>

- 배송지 변경이 가능한지를 판단할 규칙이 주문 상태와 다른 정보를 함께 사용한다면 OrderState 만으로는 배송지 변경 가능 여부를 판단할 수 없으므로 Order 에서 로직을 구현해야 한다.
- 중요한 점은 주문과 관련된 중요 업무 규칙을 주문 도메인 모델인 Order나 OrderState에서 구현한다는 점
- 핵심 규칙을 구현한 코드는 도메인 모델에만 위치하기 때문에 규칙이 바뀌거나 규칙을 확장해야 할 때 다른 코드에 영향을 덜 주고 변경 내역을 모델에 반영할 수 있게 된다.


#### 개념 모델과 구현 모델
- 개념 모델을 처음부터 완벽하게 만들기보다 전반적인 개요를 알 수 있는 수준으로 개념 모델을 작성 (초기에는 도메인에 대한 전체 윤곽을 이해하는 데 집중)
- 구현하는 과정에서 개념 모델을 구현 모델로 점진적으로 발전시켜 나가야 한다.

## 도메인 모델 도출
- 도메인을 모델링 할 때 기본이 되는 작업은 모델을 구성하는 핵심 `구성요소, 규칙, 기능`을 찾는 것이다.
  - ex) 주문
  - 최소 한 종류 이상의 상품을 주문해야 한다.
  - 한 상품을 한 개 이상 주문할 수 있다.
  - 총 주문 금액은 각 상품의 구매 가격 합을 모두 더한 금액이다.
  - 등등.
- 이를 통해 어떠한 기능을 제공하는지 추출 가능.
- 어떤 데이터로 구성되는지 알려준다.
- 각 속성간의 관계를 알려준다. (ex. Order / OrderLine(product, price, quantity))
- 도메인을 구현하다 보면 특정 조건이나 상태에 따라 제약이나 규칙이 달리 적용되는 경우가 많다.
- 다른 요구사항을 확인해서 추가로 존재할 수 있는 상태를 분석한 뒤 구현
- 요구사항을 분석하면서 도메인에 대한 정보를 더 잘 알게 되기 때문에 점진적으로 도메인 모델을 만들어 나가며, 조금 더 정확한 의미를 포함(메소드명같은..)하여 구현 가능.
  - 이렇게 만든 모델은 요구사항 정련을 위해 도메인 전문가나 다른 개발자와 논의하는 과정에서 공유하기도 한다.
  - 모델을 공유할 때는 화이트보드나 위키와 같은 도구를 사용해서 누구나 쉽게 접근할 수 있도록 하면 좋다.

#### 문서화
> - 문서화를 하는 주된 이유는 지식을 공유하기 위함.
> - 실제 구현은 코드에 있으므로 코드를 보면 다 알 수 있지만, 코드는 상세한 모든 내용을 다루고 있음.
> - 전반적인 기능 목록이나 모듈 구조, 빌드 과정은 코드를 보고 직접 이해하는 것보다 상위 수준에서 정리한 문서를 참조하는 것이 빠름.
> - 코드를 보면서 도메인을 깊게 이해하게 되므로 코드 자체도 문서화의 대상이 된다.
> - 도메인 지식이 잘 묻어나도록 코드를 작성하지 않으면 코드의 동작 과정은 해석할 수 있어도 도메인 관점에서 왜 코드를 그렇게 작성했는지 이해하는 데 도움이 되지 않는다.

## 엔티티와 밸류
### 엔티티
- 식별자를 가짐
  - 엔티티 객체마다 고유해서 각 엔티티는 서로 다른 식별자를 갖는다.
  - ex) 주문 도메인에서 각 주문은 주문번호를 가지고 있는데 이 주문번호는 각 주문마다 서로 다르다. > 주문번호가 주문의 식별자
  - Order가 엔티티가 되며 주문번호를 속성으로 갖게 된다.
  - 식별자는 바뀌지 않음.
- 엔티티의 식별자 생성
  - 특정 규칙에 따라 생성
  - UUID 같은 고유 식별자 생성기 사용
  - 값을 직접 입력
  - 일련번호 사용(시퀀스 등)
### 밸류 타입
- 밸류 타입은 개념적으로 완전한 하나를 표현할 때 사용한다.
- ShippingInfo 클래스는 받는 사람과 주소에 대한 데이터를 가지고 있다.
  - 이는 받는 사람 (Receiver) 주소 (Address) 의 밸류 타입으로 표현이 가능함.
<pre>
<code>
public class ShippingInfo {
    // 받는 사람
    private String receiverName;
    private String receiverPhoneNumber;
    // 주소
    private String shippingAddress1;
    private String shippingAddress2;
    private String shippingZipcode;
}
// ->
public class ShippingInfo {
    private Receiver receiver;
    private Address address;
}

public class Receive {
    private String name;
    private String phoneNumber;
}

public class Address {
    private String address1;
    private String address2;
    private String zipcode;
}

</code>
</pre>
- 밸류 타입이 꼭 두 개 이상의 데이터를 가져야 하는 것은 아니다.
- 의미를 명확하게 표현하기 위해 밸류 타입을 사용하는 경우도 있다.
  - ex) Money(int value)
- 밸류 타입의 장점은 밸류 타입을 위한 기능을 추가할 수 있다는 것.
  - ex) Money 타입은 돈 계산을 위한 기능을 추가할 수 있음.
    - 정수 타입 연산이 아닌 돈 계산의 의미가 전달이 됨.
<pre>
<code>
public class Money{
    private int value;

    public Money add(Money money) {
        return new Money(this.value + money.value);
    }

    public Money multiply(int multiplier) {
        return new Money(value * multiplier);
    }
}
</code>
</pre>
- <span style="color:brown">밸류 객체의 데이터를 변경할 때는 기존 데이터를 변경하기보다는 변경한 데이터를 갖는 새로운 밸류 객체를 생성하는 방식을 선호한다.</span>
  - Money 에서의 value를 변경할 수 있는 메서드가 없다?
- 데이터 변경 기능을 제공하지 않는 타입을 불변이라고 표현함.
- 밸류 타입을 불변으로 구현하는 여러 이유가 있음.
- 가장 중요한 이유는 안전한 코드를 작성할 수 있다는 데 있다.
  - 외부에서 변경 가능. 
    <pre><code>Money price = new Money(1000);
    OrderLine line = new OrderLine(product, price, 2);  // [price=1000, quantity=2, amounts=2000]
    price.setValue(2000);                               // [price=2000, quantity=2, amounts=2000]
    </code></pre>
  - 위의 문제를 방지하려면 OrderLine 생성자는 Money 객체를 복사하여 새로운 객체를 생성하는 식으로 해야 함.
    - Money가 불변이라면 이런 코드 작성 필요 없음.
    - Money 데이터를 바꿀 수 없기 때문에 파라미터로 전달받은 price를 안전하게 사용 가능.
- 두 밸류 객체를 비교할 때는 모든 속성이 같은 지 비교(equals)

### 엔티티 식별자와 밸류 타입
- 식별자를 위한 밸류 타입을 사용해서 의미가 잘 드러나도록 할 수 있다.
  - `String orderNo` -> `OrderNo id`
  - 나는 전자가 더 나아보이긴 함... (충분히 의미가 잘 드러나게 변수명을 지으면 되지 않나... 굳이?)

### 도메인 모델에 set 메서드 넣지 않기
- 도메인 모델에 get/set 메서드를 무조건 추가하는 것은 좋지 않은 버릇이다.
- set 메서드는 도메인의 핵심 개념이나 의도를 코드에서 사라지게 한다.

- `changeShippingInfo()` 가 배송지 정보를 새로 변경한다는 의미를 가졌다면, setShippingInfo() 메서드는 단순히 배송지 값을 설정한다는 것을 의미한다.
- 관련된 도메인 지식을 코드로 구현하는 것이 자연스러운데, set() 메서드는 단순히 상태 값만 변경할지 아니면 상태값에 따라 다른 처리를 위한 코드를 함께 구현할지 애마하다.
- 필드값만 변경하고 끝나면 상태 변경과 관련된 도메인 지식이 코드에서 사라지게 된다..
- 또한, set 메서드는 도메인 객체를 생성할 때 온전하지 않은 상태가 될 수 있다.
<pre>
<code>
// set 메서드로 데이터를 전달하도록 구현하면 처음 Order를 생성하는 시점에 order는 완전하지 않다.
Order order = new Order();

// set 메서드로 필요한 모든 값을 전달해야 함
order.setOrderLine(lines);
order.setShippingInfo(shippingInfo);

// 주문자(Orderer)를 설정하지 않은 상태에서 주문 완료 처리.
order.setState(OrderState.PREPARING);
</code>
</pre>
- 위 코드는 주문자 설정을 누락하고 있다.
- 주문자 정보를 담고 있는 필드인 orderer가 null인 상황에서 order.setState() 메서드를 호출해서 상품 준비 중 상태로 바꿈.
- orderer가 정상인지 확인하기 위해 null인지 검사하는 코드를 setState() 메서드에 위치하는 것도 맞지 않다.
- 도메인 객체가 불완전한 상태로 사용되는 것을 막으려면 생성 시점에 필요한 것을 전달해주어야 한다.
  - 즉, 생성자를 통해 필요한 데이터를 모두 받아야 한다.
    <pre><code>Order order = new Order(orderer, lines, shippingInfo, OrderState.PREPARING);</code></pre>
- 생성자로 필요한 것을 모두 받으므로 생성자를 호출하는 시점에 필요한 데이터가 올바른지 검사 가능.

- private set() 메소드
  - 클래스 내부에서 데이터를 변경할 목적으로 사용 가능.(null 체크 포함 등..)
  - private 이기 때문에 외부에서 데이터를 변경할 목적으로 사용할 수 없다.
- **불변 밸류 타입을 사용하면 자연스럽게 밸류 타입에는 set 메서드를 구현하지 않는다.**
- set 메서드를 구현해야 할 특별한 이유가 없다면 불변 타입의 장점을 살릴 수 있도록 밸류 타입은 불변으로 구현한다.

> DTO의 get/set 메서드
> - 이전 프레임워크에서는 set 메서드가 필요로 함.
> - DTO가 도메인 로직을 담고 있지는 않기에 get/set 메서드를 제공해도 도메인 객체의 데이터 일관성에 영향을 줄 가능성이 높지 않다.
> - 요즘 개발 프레임워크나 개발 도구는 set 메서드가 아닌 private 필드에 직접 값을 할당할 수 있는 기능을 제공
> - set 메서드를 만드는 대신 해당 기능을 최대한 활용하자
> - 이렇게 하면 DTO도 불변 객체가 되어 불변의 장점을 DTO까지 확장가능.


### 도메인 용어와 유비쿼터스 언어
- 코드를 작성할 때 도메인에서 사용하는 용어는 매우 중요.

<pre>
<code>public OrderState {STEP1, STEP2, STEP3,...}</code>
</pre>
->
<pre>
<code>
public enum OrderState {
    PAYMENT_WAITING, PREPARING, ...
}
</code>
</pre>
- 코드를 도메인 용어로 해석하거나 도메인 용어를 코드로 해석하는 과정이 줄어든다.
- 이는 코드의 가독성을 높여서 코드를 분석하고 이해하는 시간을 줄여준다.
- 최대한 도메인 용어를 사용해서 도메인 규칙을 코드로 작성하게 되므로(의미를 변환하는 과정에서 발생하는) 버그도 줄어든다.