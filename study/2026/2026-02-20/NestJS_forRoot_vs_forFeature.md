# forRoot vs forFeature

**NestJS의 동적 모듈(Dynamic Module) 패턴에서 사용하는 정적 메서드**

### 1. forRoot

- DB 연결
- 전역 환경 설정
- 글로벌 설정

→ 앱의 기반 셋팅. 단 한 번만 사용

```tsx
// app.module.ts
TypeOrmModule.forRoot({
  type: 'mysql',
  host: 'localhost',
  port: 3306,
  username: 'root',
  password: '1234',
  database: 'test',
  entities: [User],
  synchronize: true,
})
```

### 2. forFeature

- 특정 엔티티 등록
- 특정 모델 등록
- 기능 단위 provider 연결

→ 특정 모듈에서 사용할 기능 등록. 여러 번 사용 가능

```tsx
// user.module.ts
TypeOrmModule.forFeature([User])
```

---

### 3. 내부 동작 차이

- forRoot
    - DynamicModule 반환
    - Provider 생성
    - Global 설정 가능
    - Singleton기반
- forFeature
    - 이미 생성된 Provider를 기반으로 특정 토큰을 추가 등록
    - Repository/Model injection 가능하게 만듦