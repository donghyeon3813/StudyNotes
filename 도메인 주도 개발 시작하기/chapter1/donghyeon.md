# 도메인 주도 개발 시작하기 - 정리

## 1-1. 도메인이란

- **도메인**: 소프트웨어로 해결하고자 하는 문제 영역
- 도메인은 **하위 도메인(Subdomain)** 으로 나눌 수 있으며, 이는 **상황에 따라 달라질 수 있어 고정되지 않음**

> 📌 예시:  
> 온라인 쇼핑몰의 도메인  
> → 상품 관리, 주문, 결제, 배송 등으로 나뉘는 하위 도메인들이 존재함

---

## 1-2. 도메인 전문가와 개발자 간 지식 공유

- 요구사항을 제대로 이해하지 못하면 **잘못된 시스템**이 만들어질 수 있음
- 이를 해결하기 위해:
  - 도메인 전문가, 관계자, 개발자가 **같은 지식**을 공유하고 **소통**하는 것이 중요함
  - **공통 언어(Ubiquitous Language)** 를 사용하여 지식 간극을 줄일 수 있음

> 📌 예시:
> - "고객", "주문", "장바구니"와 같은 용어는 모두가 같은 의미로 이해하고 사용해야 함
> - 개발자, 기획자, 디자이너, 도메인 전문가가 모두 이 용어를 기반으로 대화

> 💡 참고:  
> 도메인 지식 공유를 위한 방법에는 모델링 워크숍, 이벤트 스토밍 등의 기법이 있음

---

## 1-3. 도메인 모델

- **도메인 모델**: 특정 도메인을 **개념적으로 표현**한 것
- 이를 통해:
  - 여러 **요구사항**을 명확히 할 수 있음
  - 여러 관계자들과의 **지식 공유**에 도움이 됨
- 도메인 모델을 표기하는 방식은 다양하며, **이해에 도움이 된다면 방식은 중요하지 않음**

> 📌 예시:
> - 클래스 다이어그램, 유비쿼터스 언어로 구성된 텍스트 모델 등  
> - 복잡한 비즈니스 로직을 코드로 옮기기 전 단계에서 모델을 그려보며 이해

---

## 1-4. 도메인 모델 패턴

- 일반적인 애플리케이션 구성:

  | 계층 | 설명 |
  |------|------|
  | **표현(UI)** | 사용자의 요청 처리 및 정보 표시 |
  | **응용(Application)** | 사용자의 기능 요청을 실행, 도메인 계층을 조합해 기능 구현 |
  | **도메인(Domain)** | 핵심 비즈니스 로직, 규칙이 담긴 계층 |
  | **인프라스트럭처(Infrastructure)** | 외부 시스템 연동 (DB, 메시지 큐 등) |

- **핵심 규칙을 구현한 코드는 도메인 계층에만 위치**해야 함  
  → 변경 시 **다른 계층에 미치는 영향이 적고**, 유연하게 대응 가능함

---
## 1-5. 도메인 모델 도출

- **요구사항**을 분석하면 필요한 **기능(유스케이스)** 과 **도메인 모델**을 도출할 수 있음
- 도출된 모델은 추상적인 설계가 아닌, 실제 구현에 활용될 수 있는 기반이 됨

> 📌 예시:
> - "상품을 주문한다"는 요구사항 → `Order`, `Product`, `Customer` 등의 도메인 모델과 기능이 도출됨

> 💡 참고:
> - 이 과정은 반복적이며, 도메인 전문가와의 대화를 통해 점진적으로 발전함

---

## 1-6. 엔티티와 밸류

### ✅ 엔티티(Entity)

- **식별자(ID)** 를 가지는 객체  
- 같은 식별자를 가진 두 객체는 **같은 엔티티**로 판단함  
- Java의 경우 `equals()`, `hashCode()`를 식별자 기준으로 오버라이딩하는 경우가 많음

### ✅ 밸류(Value Object)

- **의미 있는 개념을 하나의 값 객체로 표현**
- 상태는 같지만 식별자는 없음 → 값으로서 동등성 판단
- 도메인 개념을 **더 명확하게 표현**할 수 있도록 해줌

> 📌 예시:
> - `Address`, `Money`, `Email` 등은 밸류로 자주 사용됨  
> - `Order` 안의 배송지 정보는 `Address`라는 밸류 타입으로 표현 가능

> ⚠️ 주의:
> - 도메인 모델에 **Setter 메서드를 남용하면**, 도메인의 **불변성이나 의도**가 훼손될 수 있음  
> - 대신 생성자나 팩토리 메서드로 명확하게 의도를 표현할 것

---

## 1-7. 도메인 용어와 유비쿼터스 언어

- **도메인 용어**를 코드에 정확히 반영하지 않으면,  
  시간이 지나 **코드 해석이 어려워지고 오해를 초래**할 수 있음

> 📌 예시:
> - `STEP1`, `STEP2` 대신  
> - `PAYMENT_WAITING`, `PREPARING` 등 실제 도메인 용어를 사용하는 것이 바람직함

### ✅ 유비쿼터스 언어(Ubiquitous Language)

- **도메인 전문가와 개발자가 공통으로 사용하는 언어**
- 코드, 문서, 회의 등에서 일관되게 사용함으로써 **팀 내 소통 혼란을 방지**

> 💡 참고:
> - 유비쿼터스 언어는 도메인 모델을 중심으로 정의되며, **모델을 탐색하고 발전시키는 핵심 도구**가 됨

---
