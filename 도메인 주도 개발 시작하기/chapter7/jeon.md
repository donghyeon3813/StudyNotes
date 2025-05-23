# 도메인 서비스

## 여러 애그리거트가 필요한 기능
- ex) 결제 금액 계산 로직
  - 상품 | 주문 | 할인쿠폰 | 회원 애그리거트의 조합
  - 단일 애그리거트로는 총 결제 금액을 계산할 수 없다.
  - 이는 도메인 기능을 별도 서비스로 구현하여 해결할 수 있다.

## 도메인 서비스
- 도메인 서비스는 도메인 영역에 위치한 도메인 로직을 표현할 때 사용한다.
- 주로 다음 상황에서 도메인 서비스를 사용한다.
  - 계산 로직
    - 여러 애그리거트가 필요한 계산 로직이나, 한 애그리거트에 넣기에는 다소 복잡한 계산로직
  - 외부 시스템 연동이 필요한 도메인 로직
    - 구현하기 위해 타 시스템을 사용해야 하는 도메인 로직

### 계산 로직과 도메인 서비스
- 도메인 서비스는 상태 없이 로직만 구현한다.
- 도메인의 의미가 드러나는 용어를 타입과 메서드 이름으로 갖는다.
- 주체는 애그리거트가 될 수도 있고, 응용 서비스가 될 수도 있다.
- 애그리거트 객체에 도메인 서비스를 전달하는 것은 응용 서비스 책임이다.

> - 도메인 서비스 객체를 애그리거트에 주입하지 않기.

- 도메인 서비스의 기능을 실행할 때 애그리거트를 전달하기도 한다.
- 이런 식으로 동작하는 것 중 하나가 계좌이체 기능.
- 두 계좌 애그리거트가 관여
- 도메인 서비스는 도메인 로직을 수행하지 응용 로직을 수행하지는 않는다.
  - 트랜잭션 처리와 같은 로직은 응용 로직이므로 도메인 서비스가 아닌 응용 서비스에서 처리해야 한다.
    - 특정 기능이 응용 서비스인지 도메인서비스인지 감을 잡기 어려울 때는 해당 로직이 애그리거트의 상태를 변경하거나 애그리거트의 상태 값을 계산하는지 검사해보면 안다.

### 외부 시스템 연동과 도메인 서비스
- 외부 시스템이나 타 도메인과의 연동 기능도 도메인 서비스가 될 수 있다.
- 도메인 로직 관점에서 인터페이스를 작성.
  - 구현 클래스는 인프라스트럭쳐 영역
- 응용 서비스는 이 도메인 서비스를 이용해서 생성 권한을 검사한다.

### 도메인 서비스의 패키지 위치
- 다른 도메인 구성요소와 동일한 패키지에 위치

### 도메인 서비스의 인터페이스와 클래스
- 도메인 로직을 외부 시스템이나 별도 엔진을 이용해서 구현할 때 인터페이스와 클래스를 분리하게 된다.
- 도메인 서비스 인터페이스 : 도메인 영역
- 구현체 : 인프라스트럭처 영역