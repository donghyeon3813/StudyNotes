
# 📘 도메인 주도 개발 시작하기 - 6장 정리

## 6.1 표현 영역과 응용 영역
- 응용 영역과 표현 영역은 사용자와 도메인을 연결해주는 매개체 역할을 한다.
- **표현 영역**: 사용자의 요청 데이터를 받는 곳.
- **응용 영역**: 사용자가 원하는 기능을 제공하는 곳.
- 표현 영역에서는 응용 영역의 서비스를 호출하기 전에 응용 서비스에 적합한 데이터로 변환하고, 응답을 받은 뒤에는 사용자에게 적합한 데이터로 다시 변환해 전달한다.

## 6.2 응용 서비스의 역할
- 도메인 객체를 사용하여 사용자의 요청을 처리한다.
- 도메인 영역과 표현 영역을 연결해주는 창구 역할을 한다.
- **도메인 로직의 일부를 응용 서비스에 구현하는 것은 지양해야 한다.**
  - 코드 중복, 로직 분산 → 코드 품질 저하 (응집도 저하, 유지보수 어려움)
- **트랜잭션 처리** 또한 응용 서비스의 주요 책임이다.

## 6.3 응용 서비스의 구현

### 두 가지 방식
1. 한 클래스에 도메인의 모든 기능을 구현
   - 동일 로직에 대한 코드 중복을 줄일 수 있음
   - 하지만 클래스 크기 증가 및 연관성 낮은 코드가 혼재할 수 있음
2. 기능별로 응용 서비스 클래스를 분리
   - 클래스 수 증가
   - 코드 품질을 일정 수준으로 유지하고, 기능 간 영향 최소화 가능

### 6.3.2 응용 서비스의 인터페이스와 클래스
- 보통 **응용 서비스 인터페이스는 거의 필요하지 않다.**
- 구현체를 나눠서 테스트하거나 대체 구현이 필요한 경우에만 도입한다.

### 6.3.3 메서드 파라미터와 값 리턴
- 응용 서비스는 도메인을 이용해 사용자의 요구를 처리할 수 있도록 **필요한 파라미터를 전달**받는다.
- **DTO나 커맨드 객체를 사용하는 것이 일반적**이다. (ex: Spring에서는 바인딩, 검증 등에 유리)
- **리턴 값은 필요한 데이터만 반환**하는 것이 응집도에 좋다. (→ Aggregate 전체를 반환하는 것보다 DTO 사용 권장)

### 6.3.4 표현 영역에 의존하지 않기
- 응용 서비스는 표현 영역의 기술에 **의존하면 안 된다.**
  - ex) `HttpServletRequest`, `HttpSession` 등은 사용 금지
- 이유: 테스트 어려움, 계층 간 역할 붕괴

### 6.3.5 트랜잭션 처리
- 트랜잭션 관리는 **응용 서비스가 책임진다.**
- 한 응용 서비스 메서드 내에서 여러 도메인 기능을 호출하며 하나의 트랜잭션 경계 유지

## 6.4 표현 영역
- 사용자가 시스템을 사용할 수 있도록 흐름을 제어하고 서비스 호출을 담당한다.
- 기능:
  - 사용자의 요청을 응용 서비스로 위임
  - 응답 결과를 사용자에게 전달
  - 사용자 세션 관리

## 6.5 값 검증
- 검증 위치:
  - **표현 영역**: 필수 값, 형식, 범위 등 (ex: 빈값, 형식 오류 등)
  - **응용 서비스**: 비즈니스 로직 관점의 오류 (ex: 존재하지 않는 회원)
- 응용 서비스에서 **검증 오류를 수집해 한 번에 전달**하는 방식도 있다.

## 6.6 권한 검사
- 권한 검사 위치:
  - **표현 영역**: 인증 여부 확인 (ex: 로그인 여부)
  - **응용 서비스**: URL만으로 제어가 어려운 경우, 메서드 단위에서 검사 (AOP 사용 가능)
  - **도메인 영역**: 애그리거트를 먼저 로딩해야 검사 가능한 경우

## 6.7 조회 전용 기능과 응용 서비스
- 단순 조회(추가 로직, 트랜잭션 없음)의 경우 **응용 서비스 생략 가능**
- ex) `@Transactional(readOnly = true)` 없이 Repository 직접 호출 가능
