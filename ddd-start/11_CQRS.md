# Chapter 11: CQRS

<br>

## 💡 책에서 기억하고 싶은 내용

<br>

### 단일 모델의 단점

- 여러 애그리거트의 데이터를 사용하는 화면은 조회 속도가 느려질 수 있다
  - `식별자`를 이용한 애그리거트 참조 방식은 즉시 로딩 방식과 같은 JPA의 쿼리 관련 최적화 기능을 사용할 수 없다
  - `직접 참조` 하는 방식으로 연결해도 조회 화면 특성에 따라 같은 연관도 즉시 로딩이나 지연 로딩으로 처리해야 할 수 있다
- 이런 고민이 발생하는 이유는 `시스템 상태`를 `변경` 할 때와 `조회` 할 때 `단일 도메인 모델` 을 사용하기 때문이다
  - 상태 변경을 위한 모델과 조회를 모델을 분리하여 해당 문제를 해결할 수 있다

### CQRS

- 도메인 모델 관점에서 `상태 변경 기능` 은 주로 한 애그리거트의 상태를 변경한다
  - 반면에 `조회 기능` 에 필요한 데이터를 표시하려면 두 개 이상의 애그리거트가 필요할 때가 많다
- 상태를 변경하는 범위와 상태를 조회하는 범위가 정확하게 일치하지 않기 때문에 `단일 모델` 로 두 종류의 기능을 구현하려면 모델이 불필요하게 복잡해진다
  - 단일 모델을 사용할 때 발생하는 `복잡도`를 해결하기 위해 `CQRS` 를 사용한다
- CQRS는 Command Query Responsibility Segregation의 약자로, 상태를 변경하는 `명령 (Command)` 를 위한 모델과 상태를 제공하는 `조회 (Query)` 를 위한 모델을 분리하는 패턴이다
  - `명령 모델`
    - 애플리케이션은 변경 요청을 명령 모델로 처리
    - 명령 모델이 `도메인 로직` 실행
  - `조회 모델`
    - 조회 모델을 이용해서 표현 계층에 데이터 출력
- CQRS를 사용하면 각 모델에 맞는 `구현 기술` 을 선택할 수 있다
  - 조회 모델에서 단순히 데이터를 읽어와 조회하는 기능은 응용 로직이 복잡하지 않기 때문에 `컨트롤러에서 바로 DAO를 실행` 해도 무방하다
- 명령 모델과 조회 모델의 설계 예
  - 상태 변경을 위한 `명령 모델`은 `객체` 를 기반으로 한 `도메인 모델` 을 이용해서 구현한다
    - 상태를 변겨앟는 도메인 로직을 수행하는 데 초점이 맞춰져 있다
  - `조회 모델` 은 주문 요약 목록을 제공할 때 필요한 정보를 담고 있는 데이터 타입을 이용한다
    - 화면에 보여줄 데이터를 조회하는 데 초점을 맞춰 설계했다
- 명령 모델과 조회 모델이 서로 다른 저장소를 사용할 수도 있다
  - ex)
    - 명령 모델 - 트랜잭션을 지원하는 RDBMS
    - 조회 모델 - 조회 성능이 좋은 메모리 기반 NoSQL
  - 두 데이터 저장소 간 데이터 동기화는 `이벤트` 를 활용해서 처리한다
    - 명령 모델에서 상태를 변경하면, 이에 해당하는 이벤트가 발생하고,
    - 그 이벤트를 조회 모델에 전달해서 `변경 내역` 을 `반영` 하면 된다
  - 명령 모델과 조회 모델이 서로 다른 데이터 저장소를 사용할 경우 `데이터 동기화 시점` 에 따라 `구현 방식` 이 달라질 수 있다
    - 명령 모델에서 `데이터가 바뀌자마자` 변경 내역을 바로 조회 모델에 반영해야 한다면,
      - `동기 이벤트` 와 `글로벌 트랜잭션` 을 사용해서 실시간으로 동기화 할 수 있다
      - but, `동기 이벤트` 와 `글로벌 트랜잭션` 을 사용하면 전반적인 성능 (응답 속도와 처리량) 이 떨어지는 단점이 있다
    - 서로 다른 저장소의 데이터를 `특정 시간 안에만` 동기화해도 된다면, `비동기로 데이터를 전송` 하면 된다

### CQRS 장단점

- CQRS 패턴을 적용할 때 얻을 수 있는 `장점`
  - `명령 모델` 을 구현할 때 `도메인 자체에 집중` 할 수 있다
    - 복잡한 도메인은 주로 `상태 변경 로직` 이 복잡한데, 명령 모델과 조회 모델을 구분하면 조회 성능을 위한 코드가 명령 모델에 없으므로 도메인 로직을 구현하는 데 집중할 수 있다
      - 또한 명령 모델에서 조회 관련 로직이 사라져 `복잡도` 가 낮아진다
  - `조회 성능` 을 향상시키는 데 유리하다
    - 조회 단위로 `캐시 기술` 을 적용할 수 있고, 조회에 특화된 쿼리를 마음대로 사용할 수도 있다
    - 조회 전용 저장소를 사용하면 `조회 처리량` 을 대폭 늘릴 수도 있다
    - 조회 전용 모델을 사용하기 때문에 `조회 성능을 높이기 위한 코드`가 명령 모델에 영향을 주지 않는다
- `단점`
  - 구현해야 할 코드가 더 많다
  - 더 많은 구현 기술이 필요하다
- `정리`
  - 도메인이 복잡하지 않은데 CQRS를 도입하면 두 모델을 유지하는 비용만 높아지고 얻을 수 있는 이점은 없다
  - 반면 트래픽이 높은 서비스인데 단일 모델을 고집하면 `유지 보수 비용` 이 더 높아질 수 있으므로 CQRS 도입을 고려하자