# NestJS_forwardRef(), DDD

### forwardRef()

- 순환 참조(Circular Dependency)가 발생했을 때 의존성 해결을 지연시키기 위한 함수
- NestJS에서 클래스를 생성할 때 DI 컨테이너는 의존성을 미리 모두 파악하는데 서로가 서로를 참조하면 문제가 발생함.

```tsx
// AService
constructor(private readonly bService: BService) {}

// BService
constructor(private readonly aService: AService) {}
```

---

### 해결방법

```tsx
// AService
constructor(
  @Inject(forwardRef(() => BService))
  private readonly bService: BService,
) {}
```

```tsx
// BService
constructor(
  @Inject(forwardRef(() => AService))
  private readonly aService: AService,
) {}
```

- `forwardRef(() => BService`
    - 즉시 BService를 참조하지 않고, 나중에 이 함수를 실행해서 가져오라고 Nest에게 알려줌(지연).

---

### 사례

- 서로 다른 도메인이 서로를 호출
    - UserService → OrderService
    - OrderService → UserService
- 이벤트 없이 직접 호출

---

### 더 좋은 해결방법

- 공통 서비스로 분리
    - `UserService, OrderService → CommonService`
- 이벤트 기반 구조(EventEmitter)
    - Order 생성 후
        
        ```tsx
        this.eventEmitter.emit('order.created', order);
        ```
        
    - UserService는 이벤트만 구독
        
        → 직접 의존 제거
        
- Application Service/Facade 도입

---

### DDD(Domain Driven Design)

- 보통 4계층 구조
    
    Interface → Application → Domain → Infrastructure
    
- 복잡한 비즈니스 규칙 OR 도메인 개념이 중요 OR 트랜잭션 정합성이 중요 OR 대규모 팀 협업 케이스에 적합

---

### 계층별 역할(NestJS)

- DDD에서는 도메인은 다른 도메인을 직접 호출하지 않고, 이벤트 또는 Application 계층에서 조합하기에 순환 참조 구조 자체가 생기지 않는다
- **Interface Layer**
    - Controller
    - DTO
    - Request/Response 처리
        
        → 외부 세계와의 연결 지점
        
- **Application Layer**
    - 유스케이스 조합
    - 트랜잭션 경게
    - 도메인 객체 호출
- **Domain Layer**
    - Entity
    - Value Object
    - Domain Service
    - Aggregate
    - Domain Event
- **Infrastructure Layer**
    - Prisma
    - TypeORM
    - 외부 API
    - 메시지 브로커

---

### Aggregate

- 하나의 트랜잭션 단위로 관리되는 도메인 묶음
- 하나의 루트 엔티티(Aggregate Root)를 가지며, 이 루트 엔티티를 통해서만 aggregate 내부의 다른 객체에 접근하고 변경할 수 있음

```
Order
- OrderItem
- Payment
```

- 외부에서는 반드시 Order를 통해서만 접근

```
order.addItem(...)
order.cancel()
```

---

### Entity

- 고유한 식별자를 가지며, 시간의 흐름에 따라 상태가 변경될 수 있는 객체
- 동일한 속성을 갖더라도 식별자가 다르면 다른 엔티티로 간주됨

---

### Repository

- 도메인 모델의 영속성(Persistence)를 담당하는 인터페이스
- Aggregate Root를 저장하고 조회하는 작업 추상화
- 도메인 로직이 데이터베이스 접근 로직과 섞이는 것을 방지해 도메인 모델을 순수하게 유지함. DB 변경에도 도메인 계층에 미치는 영향 최소

---

### Factory

- 복잡한 도메인 객체의 생성 로직을 캡슐화하는 객체
- 객체 생성의 복잡성을 외부에서 숨기고, 항상 유효한 상태의 객체가 생성되도록 보장함