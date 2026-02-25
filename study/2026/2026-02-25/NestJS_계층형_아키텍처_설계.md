# NestJS 기반 계층형 아키텍처 설계

### **1. 전체 흐름**

- **계층형 아키텍처(Layered Architecture)**

→ 애플리케이션을 역할에 따라 여러 계층으로 분리해 관심사를 분리(Separation of Concerns)하는 설계 방식. NestJS에서는 일반적으로 아래와 같은 흐름으로 이동한다.

> Client ➡️ **Controller** ➡️ (**Facade**) ➡️ **Service** ➡️ **Repository** ➡️ Database
> 
- 의존성은 항상 위에서 아래로만 흐름
- 순환 참조 발생하지 않도록 설계해야함
- 상위 계층은 하위 계층을 의존할 수 있으나 하위 계층은 상위 계층을 절대 참조하지 않음

---

### **2. 계층별 역할 상세 설명**

**1️⃣ Controller (입구)**

- **프론트엔드 비유:** `Router` + `Event Handler`
- **역할:** 클라이언트의 HTTP 요청을 가장 먼저 받아 유효성을 검사하고 응답을 돌려준다.
- **핵심:** 비즈니스 로직은 몰라야 하며, **"어떤 서비스로 보낼지"**만 결정한다.

```tsx
// posts.controller.ts
@Controller('posts')
export class PostsController {
  constructor(private readonly postsFacade: PostsFacade) {}

  @Post()
  async createPost(@Body() createPostDto: CreatePostDto) {
    // 1. 요청을 받아 유효성 검사를 거친 후 Facade(또는 Service)로 넘김
    return await this.postsFacade.createPostWithNotification(createPostDto);
  }
}
```

**2️⃣ Facade (중재자 / 선택 사항)**

- **프론트엔드 비유:** 복합적인 로직을 처리하는 `Custom Hook`
- **역할:** 여러 개의 서비스를 조합해야 할 때 사용함. (예: 결제 서비스 + 메일 발송 서비스 + 재고 서비스 호출). 선택사항
- **핵심:** 서비스 간의 결합도를 낮추는 **지휘자** 역할

```tsx
// posts.facade.ts
@Injectable()
export class PostsFacade {
  constructor(
    private readonly postsService: PostsService,
    private readonly alarmService: AlarmService, // 다른 서비스 주입
  ) {}

  async createPostWithNotification(dto: CreatePostDto) {
    // 2. 비즈니스 로직들을 조합 (글 저장 + 알림 발송)
    const post = await this.postsService.savePost(dto);
    await this.alarmService.sendSlackMessage('새 글이 등록되었습니다!');
    return post;
  }
}
```

**3️⃣ Service (핵심 로직)**

- **프론트엔드 비유:** `Business Logic Functions`
- **역할:** 실제 핵심 규칙(계산, 필터링, 정책 적용 등) 구현
- **핵심:** 데이터베이스에 어떻게 저장되는지는 모르며, 오직 **기능 처리**에만 집중

```tsx
// posts.service.ts
@Injectable()
export class PostsService {
  constructor(private readonly postsRepository: PostsRepository) {}

  async savePost(dto: CreatePostDto) {
    // 3. 비즈니스 규칙 적용 (예: 제목에 금지어 체크 등)
    if (dto.title.includes('금지어')) {
      throw new BadRequestException('부적절한 제목입니다.');
    }
    return await this.postsRepository.create(dto);
  }
}
```

**4️⃣ Repository (데이터 관리)**

- **프론트엔드 비유:** `Axios Instance` / `API Fetcher`
- **역할:** 데이터베이스(DB)에 직접 접근하여 데이터를 CRUD(생성, 읽기, 수정, 삭제)함. 도메인 계층과 DB 사이의 추상화 계층
- **핵심:** **Prisma**나 **TypeORM** 같은 도구를 사용하여 DB와 대화

```tsx
// posts.repository.ts (TypeORM 예시)
@Injectable()
export class PostsRepository {
  constructor(
    @InjectRepository(PostEntity)
    private readonly repository: Repository<PostEntity>,
  ) {}

  async create(dto: CreatePostDto) {
    // 4. 실제로 DB 테이블에 Insert 실행
    return await this.repository.save(dto);
  }
}
```

---

### **3. NestJS + Prisma 코드 예시**

```tsx
// prisma.service.ts
import { Injectable, OnModuleInit } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';

@Injectable()
export class PrismaService extends PrismaClient implements OnModuleInit {
  async onModuleInit() {
    await this.$connect(); // 앱 실행 시 DB 연결
  }
}
```

```tsx
// posts.repository.ts
@Injectable()
export class PostsRepository {
  constructor(private prisma: PrismaService) {} // DI로 Prisma 주입

  async create(data: CreatePostDto) {
    // 프론트엔드 개발자에게 익숙한 객체 기반 쿼리
    return await this.prisma.post.create({
      data: {
        title: data.title,
        content: data.content,
        authorId: data.authorId,
      },
    });
  }

  async findMany() {
    return await this.prisma.post.findMany({
      include: { author: true }, // Join 같은 복잡한 관계도 객체로 처리
    });
  }

```

```tsx
// posts.service.ts
@Injectable()
export class PostsService {
  constructor(private readonly repository: PostsRepository) {}

  async savePost(dto: CreatePostDto) {
    // 1. 비즈니스 로직 (예: 권한 체크 등)
    // 2. DB 저장 요청
    return await this.repository.create(dto);
  }
}
```

---

### **4. 의존성 주입 (DI, Dependency Injection)**

- **정의:** 필요한 부품(Service, Repository 등)을 내부에서 직접 `new`로 만들지 않고, **외부(NestJS 컨테이너)로부터 주입**받는 패턴
- **프론트엔드 비유:** 부모 컴포넌트에서 자식으로 내려주는 **Props**나 **Context API**와 유사
- **장점:** 코드가 유연해지고, 가짜(Mock) 데이터를 넣기 쉬워져서 **테스트 코드 작성**에 매우 유리

---

### **5. 장점**

- 유지보수: DB가 바뀌어도 Repository만 수정하면 됨
- 재사용성: 하나의 Service 로직을 여러 Controller에서 사용 가능
- 테스트: UI나 DB 없이 로직(Service)만 따로 떼어 검증 가능

---

### 6. 단점

- 코드 증가: 단순 CRUD도 여러 계층을 거쳐야할 수 있음
- Service 비대화 위험: 모든 로직인 Service에 몰리면 God Object가 됨
- 과설계 위험: 소규모 프로젝트라면 오히려 복잡성을 증가시킬 수 있