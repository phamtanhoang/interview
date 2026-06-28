# 📚 Interview Prep — Node.js / NestJS (Mid/Senior)

> Tổng hợp câu hỏi phỏng vấn theo chủ đề, đúc kết trong quá trình xây dự án **Blog/CMS API**.
> Bấm vào từng câu hỏi để mở đáp án + ví dụ. File này được làm giàu dần sau mỗi bài học.

## Mục lục
1. [Middleware (Express) nền tảng](#1-middleware-express-nền-tảng)
2. [NestJS Architecture & Dependency Injection](#2-nestjs-architecture--dependency-injection)
3. [Configuration & Environment](#3-configuration--environment)
4. [Database, TypeORM & Entities](#4-database-typeorm--entities)
5. [DTO, Validation & Pipes](#5-dto-validation--pipes)
6. [REST API & Controllers](#6-rest-api--controllers)
7. [Authentication & Authorization](#7-authentication--authorization)
8. [Docker & Debugging thực chiến](#8-docker--debugging-thực-chiến)
9. [TypeORM Relations](#9-typeorm-relations)
10. [Interceptors & Serialization](#10-interceptors--serialization)
11. [Pagination, Filtering & QueryBuilder](#11-pagination-filtering--querybuilder)
12. [Exception Filters & Error Handling](#12-exception-filters--error-handling)
13. [Testing (Jest)](#13-testing-jest)
14. [Prisma (ORM)](#14-prisma-orm)
15. [Advanced Authentication](#15-advanced-authentication)
16. [Web Security](#16-web-security)
17. [Observability and Logging](#17-observability-and-logging)
18. [Advanced NestJS](#18-advanced-nestjs)
19. [Database, Transactions and Concurrency](#19-database-transactions-and-concurrency)
20. [Advanced Node.js](#20-advanced-nodejs)
21. [Advanced Testing](#21-advanced-testing)
22. [System Design and Scaling](#22-system-design-and-scaling)
23. [API and REST Design](#23-api-and-rest-design)
24. [Scheduled and Background Jobs](#24-scheduled-and-background-jobs)
25. [Worked Examples for Core Concepts](#25-worked-examples-for-core-concepts)
26. [GraphQL và Realtime trong NestJS](#26-graphql-và-realtime-trong-nestjs)
27. [Tư duy và Giải quyết vấn đề (Backend)](#27-tư-duy-và-giải-quyết-vấn-đề-backend)
28. [MongoDB và Mongoose (NoSQL)](#28-mongodb-và-mongoose-nosql)
29. [Git Workflow và Cộng tác](#29-git-workflow-và-cộng-tác)
30. [CI/CD và Triển khai (Docker, Cloud)](#30-cicd-và-triển-khai-docker-cloud)
31. [SQL Thuần và Thiết kế CSDL](#31-sql-thuần-và-thiết-kế-csdl)

---

## 1. Middleware (Express) nền tảng

<details>
<summary><b>1.1 Middleware trong Express là gì? Mô hình req → middleware → handler, vai trò của next() và vài middleware hay dùng?</b></summary>

**📌 Câu trả lời mẫu:**

"Middleware là một hàm nằm giữa request và route handler, có chữ ký `(req, res, next)`. Khi request đi vào, nó chạy lần lượt qua từng middleware theo thứ tự đăng ký rồi mới tới handler cuối cùng, nên mình gọi là một 'pipeline' xử lý. Mỗi middleware có thể đọc/sửa `req` và `res`, ví dụ gắn `req.user` sau khi xác thực; rồi nó gọi `next()` để chuyển quyền cho middleware kế tiếp. Nếu mình không gọi `next()` và cũng không trả response thì request sẽ bị treo, còn nếu gọi `next(err)` thì Express nhảy thẳng tới error-handling middleware. Những middleware hay dùng nhất là logging (ghi method/URL), authentication/authorization (kiểm tra JWT, chặn 401), parse body như `express.json()`, và error handler đặt cuối cùng. Bên NestJS của Jira Clone thì ý tưởng tương tự nhưng được chia rõ thành Middleware, Guard, Interceptor và Exception Filter — ví dụ check JWT thường viết bằng Guard thay vì middleware thuần."

---

- Middleware = hàm `(req, res, next)` chạy **giữa** request và handler, theo **đúng thứ tự đăng ký**.
- `next()` → chuyển sang middleware/handler kế; **không** gọi `next()` và không trả response → request **treo**.
- `next(err)` → bỏ qua phần còn lại, nhảy tới **error-handling middleware** (hàm có 4 tham số `(err, req, res, next)`).
- Có thể **đọc/sửa** `req`/`res` (gắn `req.user`, set header) hoặc **chặn sớm** (trả 401, 400).
- Middleware phổ biến: **log**, **auth**, body parser (`express.json()`), **error handler** (đặt cuối).
- NestJS chia mịn hơn: Middleware / Guard / Interceptor / Exception Filter (Guard thường lo auth).

```
req ──► [log] ──► [auth] ──► [parse body] ──► handler ──► res
            │ next()  │ next()      │ next()
            └─ next(err) ──────────────────────► [error handler] ──► res

// Ví dụ middleware log + auth (Express)
app.use((req, res, next) => {        // log
  console.log(req.method, req.url);
  next();
});

function auth(req, res, next) {       // auth
  if (!req.headers.authorization) return res.status(401).json({ msg: 'No token' });
  req.user = verify(req.headers.authorization); // gắn vào req
  next();
}

// error handler: 4 tham số, đặt CUỐI
app.use((err, req, res, next) => {
  res.status(500).json({ error: err.message });
});
```

</details>

---

## 2. NestJS Architecture & Dependency Injection

<details>
<summary><b>2.1 Dependency Injection (DI) là gì? Lợi ích?</b></summary>

**📌 Câu trả lời mẫu:**

"Dependency Injection là cách mà một class không tự khởi tạo những thứ nó cần, mà chỉ khai báo ra ở constructor rồi để framework tự tạo và 'tiêm' vào giúp. Trong NestJS có một IoC Container giữ sẵn instance của từng provider, nên khi mình viết `constructor(private prisma: PrismaService)` thì Nest tự nhìn vào đó và cấp đúng instance, mình không phải `new` thủ công. Bản chất là đảo ngược quyền điều khiển (Inversion of Control): thay vì class tự quản lý dependency của nó thì framework lo việc đó. Lợi ích lớn nhất với mình là loose coupling và dễ test — ví dụ trong Jira Clone, khi test `IssuesService` mình chỉ cần tiêm một mock của `PrismaService` thay cho bản thật, không cần đụng tới database. Nó cũng giúp tái sử dụng và thay đổi implementation dễ hơn vì các class không phụ thuộc cứng vào nhau."

---

- Class chỉ **khai báo** thứ nó cần (qua constructor), framework tự tạo và "tiêm" vào — không tự `new`.
- NestJS dùng **IoC Container** giữ sẵn instance của mỗi provider.
- Lợi ích: **Inversion of Control**, **loose coupling**, và **dễ test** (tiêm mock thay bản thật).

```ts
constructor(private readonly usersService: UsersService) {}
// Nest tự cấp, không cần new.
```
</details>

<details>
<summary><b>2.2 Phân biệt Controller và Service/Provider?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình phân biệt theo trách nhiệm: Controller lo phần HTTP, còn Service lo phần nghiệp vụ. Controller chỉ nhận request, lấy dữ liệu từ body/param/query rồi gọi service và trả response về — nên mình hay nói Controller phải 'mỏng', gần như không chứa logic. Ngược lại Service (provider) là nơi 'dày', chứa toàn bộ business logic như kiểm tra điều kiện, gọi database, xử lý quy tắc. Ví dụ trong Jira Clone, `IssuesController` chỉ nhận DTO tạo issue rồi gọi `issuesService.create()`, còn việc kiểm tra project tồn tại hay gán người tạo thì nằm trong service. Lý do tách như vậy là để dễ test (test logic ở service mà không cần dựng HTTP), dễ tái dùng (nhiều controller có thể gọi chung một service) và code rõ ràng theo từng tầng."

---

- **Controller**: xử lý HTTP & routing — "mỏng", chỉ nhận request, trả response.
- **Service (Provider)**: chứa **business logic** — "dày".
- Nguyên tắc: *Controller mỏng, Service dày* → tách bạch, dễ test, dễ tái dùng.
</details>

<details>
<summary><b>2.3 Module trong NestJS để làm gì?</b></summary>

**📌 Câu trả lời mẫu:**

"Module trong NestJS mình hình dung như một 'chiếc hộp' gom các thành phần liên quan tới cùng một nghiệp vụ lại với nhau. Một module khai báo bốn thứ chính: `controllers` (xử lý HTTP), `providers` (các service), `imports` (mượn module khác để dùng provider của họ) và `exports` (mở provider của mình ra cho module khác). Cả ứng dụng Nest thực chất là tập hợp nhiều module ghép lại, bắt đầu từ `AppModule`. Cách chia này giúp dự án lớn như Jira Clone gọn gàng — ví dụ mình tách `IssuesModule`, `ProjectsModule`, `AuthModule` riêng, mỗi cái tự quản lý phạm vi của nó. Nhờ vậy code dễ đọc, dễ bảo trì và các phần không bị rối vào nhau."

---

- Là "chiếc hộp" gom nhóm các thành phần liên quan.
- Gồm: `controllers`, `providers`, `imports` (mượn module khác), `exports` (cho module khác dùng provider của mình).
- App Nest = tập hợp nhiều module.
</details>

<details>
<summary><b>2.4 Module encapsulation: làm sao dùng provider của module khác?</b></summary>

**📌 Câu trả lời mẫu:**

"Điểm cốt lõi là provider mặc định chỉ 'sống' bên trong module khai báo nó, không tự động dùng được ở mọi nơi — Nest gọi đây là module encapsulation. Nên muốn module B xài provider của module A thì phải làm hai bước: module A `exports` provider đó ra, và module B `imports` module A vào. Ví dụ trong Jira Clone, `AuthModule` cần dùng `UsersService` để tìm user lúc đăng nhập, nên `UsersModule` phải export `UsersService`, rồi `AuthModule` import `UsersModule`. Nếu thiếu một trong hai bước này thì Nest sẽ báo lỗi không resolve được dependency. Cơ chế encapsulation này tuy hơi phiền lúc đầu nhưng rất tốt vì nó buộc mình khai báo rõ ràng module nào phụ thuộc module nào, tránh việc mọi thứ phơi ra toàn cục."

---

- Provider chỉ "sống" trong module khai báo nó.
- Muốn module B dùng provider của A: **A `exports`** provider, **B `imports`** module A.
- Ví dụ: `UsersModule` exports `UsersService` để `AuthModule` import & dùng.
</details>

<details>
<summary><b>2.5 Scope mặc định của provider là gì?</b></summary>

**📌 Câu trả lời mẫu:**

"Mặc định provider trong NestJS có scope là Singleton, tức là Nest chỉ tạo đúng một instance lúc khởi động rồi dùng chung cho toàn bộ ứng dụng và mọi request. Đây là lựa chọn mặc định vì nó hiệu quả nhất — không phải tạo đi tạo lại object cho mỗi request, và phần lớn service như `IssuesService` đều stateless nên dùng chung hoàn toàn an toàn. Ngoài ra còn hai scope khác: `REQUEST` tạo một instance mới cho mỗi request (hợp khi cần dữ liệu riêng theo từng request, nhưng tốn hiệu năng hơn), và `TRANSIENT` tạo instance mới cho mỗi nơi inject vào. Trong thực tế mình gần như luôn để mặc định Singleton, chỉ cân nhắc `REQUEST` khi thật sự cần giữ state riêng theo từng request."

---

- Mặc định **Singleton** (`DEFAULT`): tạo **một lần**, dùng chung toàn app.
- `REQUEST`: mỗi request một instance.
- `TRANSIENT`: mỗi nơi inject một instance.
</details>

<details>
<summary><b>2.6 NestJS là gì và vì sao dùng Nest thay vì Express thuần? Đánh đổi là gì?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình hiểu NestJS là một framework Node.js xây trên nền Express (có thể đổi sang Fastify), được viết TypeScript-first và áp một kiến trúc module hoá rõ ràng. Lý do mình chọn Nest thay vì Express thuần là vì Nest là framework opinionated: nó định sẵn cách tổ chức code thành Module, Controller, Service nên cả team đi theo một cấu trúc thống nhất, dự án lớn như Jira Clone vẫn dễ đọc dễ bảo trì. Quan trọng nhất là Nest có sẵn Dependency Injection: ví dụ mình chỉ cần khai báo `PrismaService` ở constructor là Nest tự tiêm vào, không phải tự `new` và truyền tay như Express, nhờ vậy code loose coupling và rất dễ viết unit test bằng cách tiêm mock. Ngoài ra Nest tích hợp sẵn nhiều thứ như validation (DTO + ValidationPipe), Guard cho auth, Interceptor, Exception Filter mà với Express mình phải tự lắp ráp từng thư viện rời. Đánh đổi là Nest học khó hơn, nhiều khái niệm và decorator phải nắm, đồng thời 'nặng' và nhiều magic hơn Express thuần — với một service siêu nhỏ thì Express có thể gọn và linh hoạt hơn. Nhưng với một API nghiệp vụ lớn, đi đường dài thì lợi ích về cấu trúc và khả năng bảo trì của Nest vượt trội."

---

- **NestJS**: framework Node.js, **TypeScript-first**, mặc định chạy trên Express (đổi được sang Fastify).
- **Opinionated + module hoá**: ép một cấu trúc thống nhất (Module/Controller/Service) → team đông, dự án lớn vẫn dễ đọc, dễ bảo trì.
- **DI có sẵn**: khai báo dependency ở constructor, Nest tự tiêm → loose coupling, dễ test (tiêm mock).
- **Tích hợp sẵn**: validation, Guard, Interceptor, Filter... thay vì tự ghép thư viện rời như Express.
- **Đánh đổi**: learning curve cao, nhiều khái niệm/decorator, "nặng" và nhiều magic hơn → service siêu nhỏ thì Express gọn và linh hoạt hơn.

```
Express thuần: bạn TỰ quyết cấu trúc + TỰ new/truyền dependency + TỰ ghép từng middleware.
NestJS:        cấu trúc CÓ SẴN + DI tự tiêm + Pipe/Guard/Interceptor/Filter tích hợp.
            → đổi sự tự do lấy tính nhất quán & khả năng bảo trì.
```

</details>

<details>
<summary><b>2.7 Các decorator cơ bản trong NestJS làm gì (@Module, @Controller, @Injectable, @Get/@Post, @Body/@Param/@Query)? Cho ví dụ một controller nhỏ.</b></summary>

**📌 Câu trả lời mẫu:**

"Trong NestJS mình hay nói decorator là cách gắn 'nhãn' (metadata) lên class hoặc tham số để framework biết phải xử lý chúng thế nào. `@Module` khai báo một khối gom `controllers`, `providers`, `imports`, `exports` lại với nhau — ví dụ `IssuesModule` trong Jira Clone. `@Controller('issues')` đánh dấu class là nơi nhận HTTP request và đặt prefix route là `/issues`. `@Injectable()` đánh dấu một class là provider để DI container quản lý và tiêm vào nơi khác, như `IssuesService`. `@Get()`/`@Post()` map một method với HTTP verb cộng path tương ứng. Còn `@Body()` lấy body request (thường bind vào DTO), `@Param('id')` lấy route param như `/issues/:id`, và `@Query()` lấy query string như `?status=DONE`. Nhờ vậy mình không phải tự đọc `req.body` hay `req.params` thủ công, code rất gọn và rõ ràng."

---

- `@Module`: gom nhóm `controllers` / `providers` / `imports` / `exports` thành một khối — đơn vị tổ chức của app.
- `@Controller('prefix')`: class xử lý HTTP, đặt prefix route chung.
- `@Injectable()`: đánh dấu provider (service) để DI container tạo và tiêm.
- `@Get()` / `@Post()` (và `@Patch`, `@Delete`...): map method với HTTP verb + path.
- `@Body()`: lấy request body (hay bind vào DTO); `@Param('id')`: lấy route param; `@Query()`: lấy query string.
- Bản chất chung: decorator chỉ gắn metadata, Nest đọc metadata đó để định tuyến và resolve dependency.

```ts
@Controller('issues')              // route prefix: /issues
export class IssuesController {
  constructor(private readonly issuesService: IssuesService) {} // DI tiêm vào

  @Get()                           // GET /issues?status=DONE
  findAll(@Query('status') status: string) {
    return this.issuesService.findAll(status);
  }

  @Get(':id')                      // GET /issues/123
  findOne(@Param('id') id: string) {
    return this.issuesService.findOne(id);
  }

  @Post()                          // POST /issues
  create(@Body() dto: CreateIssueDto) {
    return this.issuesService.create(dto);
  }
}
```

</details>

<details>
<summary><b>2.8 Một HTTP request trong NestJS đi qua những thành phần nào và theo thứ tự nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Khi một request đi vào NestJS, nó đi qua một chuỗi thành phần theo thứ tự cố định. Đầu tiên là **Middleware** (chạy sớm nhất, ở tầng Express), rồi đến **Guard** để kiểm tra quyền truy cập (ví dụ trong Jira Clone mình dùng `JwtAuthGuard` xác thực token), tiếp theo là **Interceptor** (phần *trước* handler), rồi **Pipe** để validate và transform dữ liệu đầu vào (ví dụ `ValidationPipe` kiểm tra DTO). Sau đó request mới tới **Route Handler** trong controller để chạy logic. Trên đường trả về, response đi ngược qua **Interceptor** (phần *sau* handler) để biến đổi output. Nếu bất kỳ bước nào ném lỗi, **Exception Filter** sẽ bắt và định dạng response lỗi. Cách nhớ đơn giản của mình là: Middleware vào trước, Filter ra sau, và Interceptor là thứ duy nhất chạy ở cả hai chiều."

---

- Thứ tự đi vào: **Middleware → Guard → Interceptor (pre) → Pipe → Handler**.
- Thứ tự đi ra: **Handler → Interceptor (post) → response**; có lỗi ở bất kỳ đâu → **Exception Filter** bắt.
- **Guard** quyết định request có được đi tiếp không (`true/false`); chạy *trước* Pipe.
- **Pipe** validate/transform input (ví dụ DTO qua `ValidationPipe`), chạy *ngay trước* handler.
- **Interceptor** là thành phần duy nhất bao quanh handler (chạy cả pre và post — dùng RxJS).
- **Exception Filter** là điểm cuối cùng cho mọi lỗi, dù lỗi xảy ra ở Guard, Pipe hay Handler.

```
Request
  → Middleware
  → Guard          (cho qua hay chặn?)
  → Interceptor    (pre: trước handler)
  → Pipe           (validate/transform DTO)
  → Route Handler  (logic trong controller/service)
  → Interceptor    (post: biến đổi response)
  → Response
        │
        └─(nếu có lỗi ở bất kỳ bước nào)─→ Exception Filter
```

</details>

<details>
<summary><b>2.9 Phân biệt Pipe, Guard, Interceptor và Exception Filter — mỗi cái dùng để làm gì và khi nào dùng?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình hình dung 4 cái này theo đúng thứ tự chúng chạy trong vòng đời request. **Guard** chạy trước, trả lời câu hỏi 'request này có được phép đi tiếp không?' — tức là authentication/authorization; ví dụ trong Jira Clone mình dùng `JwtAuthGuard` để chặn user chưa đăng nhập và `RolesGuard` để chỉ cho admin xoá project. **Pipe** chạy ngay trước handler, lo việc transform và validate dữ liệu đầu vào; ví dụ `ValidationPipe` kiểm tra DTO tạo issue, hay `ParseIntPipe` đổi `:id` từ string sang number. **Interceptor** thì bọc quanh handler, can thiệp được cả trước và sau khi handler chạy — hợp cho logging, đo thời gian, hoặc bọc lại response theo format chung. Cuối cùng **Exception Filter** chỉ kích hoạt khi có lỗi ném ra, để bắt exception và trả về response lỗi đúng chuẩn (status code, message). Nói gọn: Guard quyết định *được/không được*, Pipe lo *dữ liệu vào*, Interceptor *bọc trước/sau*, còn Filter *xử lý lỗi*. Đừng nhầm dùng Guard để validate dữ liệu hay dùng Pipe để check quyền."

---

- **Guard** → quyết định *cho phép hay không* (authN/authZ). Trả `true/false`. VD: `JwtAuthGuard`, `RolesGuard`.
- **Pipe** → *transform & validate* input ngay trước handler. VD: `ValidationPipe` (DTO), `ParseIntPipe`.
- **Interceptor** → *bọc quanh* handler, chạy **trước + sau** (logging, đo thời gian, biến đổi response).
- **Exception Filter** → *bắt lỗi* được ném ra, trả response lỗi đúng format. VD: bắt `NotFoundException` → 404.
- Thứ tự chạy: **Guard → Interceptor (before) → Pipe → Handler → Interceptor (after) → Filter (nếu có lỗi)**.
- Đừng nhầm vai trò: check quyền dùng Guard (không dùng Pipe); validate body dùng Pipe (không dùng Guard).

```
Request
  └─> Guard        (được đi tiếp?)        --reject--> Exception Filter
        └─> Interceptor (before)
              └─> Pipe (validate/transform input)
                    └─> Handler (controller) --throw--> Exception Filter
              <─ Interceptor (after / map response)
  <─ Response
```

</details>

<details>
<summary><b>2.10 Middleware trong NestJS là gì, đăng ký thế nào, và khác Guard / Interceptor ở điểm nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Middleware là hàm chạy **sớm nhất** trong vòng đời request, **trước khi tới route handler**, và nó làm việc trực tiếp với object `req`/`res` của Express (giống middleware Express truyền thống). Mình thường dùng nó cho mấy việc cross-cutting ở tầng thấp như log request, gắn `requestId`, hoặc parse header. Để đăng ký, mình tạo class implement `NestMiddleware` với hàm `use(req, res, next)`, rồi khai báo trong module qua `configure(consumer)` của `MiddlewareConsumer`, chọn route áp dụng bằng `forRoutes(...)`; phải gọi `next()` để request đi tiếp. Điểm khác cốt lõi: middleware chạy *trước cả Guard*, và nó **không** thấy được context của Nest như biết handler/class nào sắp chạy hay metadata gắn bằng decorator. Còn **Guard** chạy sau middleware, có `ExecutionContext` nên dùng để quyết định cho phép request đi tiếp hay không (auth/role) — trong Jira Clone mình dùng Guard để check JWT và quyền trong project. **Interceptor** thì bọc *quanh* handler (chạy cả trước và sau), thấy được response để biến đổi hay đo latency. Nói gọn: middleware = sớm nhất + cấp Express thuần; Guard = chặn/cho qua; Interceptor = bọc và biến đổi input/output."

---

- **Middleware**: chạy **đầu tiên**, trước route handler và trước Guard; thao tác trực tiếp `req`/`res` (Express), **bắt buộc gọi `next()`** để đi tiếp.
- Đăng ký bằng `configure(consumer)` trong module (`implements NestModule`) — không dùng decorator như Guard/Interceptor.
- **Không** có `ExecutionContext` → không biết handler/controller nào sắp chạy, không đọc được metadata decorator.
- **Guard** (sau middleware): có `ExecutionContext`, trả `true/false` để **cho qua hay chặn** (auth/role).
- **Interceptor**: **bọc quanh** handler (trước + sau), thấy và **biến đổi** response, đo thời gian (RxJS).
- Thứ tự: **Middleware → Guard → Interceptor (before) → Pipe → Handler → Interceptor (after)**.

```ts
// logger.middleware.ts
@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log(`${req.method} ${req.url}`);
    next(); // bắt buộc, nếu không request bị "treo"
  }
}

// app.module.ts
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer.apply(LoggerMiddleware).forRoutes('*');
  }
}
```

</details>

<details>
<summary><b>2.11 DTO là gì trong NestJS? ValidationPipe + class-validator/class-transformer hoạt động ra sao và vì sao validate ở DTO lại quan trọng?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình hiểu DTO (Data Transfer Object) là một class mô tả hình dạng dữ liệu đi vào (hoặc đi ra) của API — coi như hợp đồng cho request body. Trong NestJS, mình gắn các decorator của class-validator lên từng field, ví dụ trong Jira Clone có `CreateIssueDto` với `@IsString() title` và `@IsUUID() projectId`. Khi bật `ValidationPipe` (thường bật global trong `main.ts`), mỗi request tới handler sẽ được nó kiểm tra: class-transformer dùng `transform` để biến JSON thô thành instance đúng class và ép kiểu (ví dụ `"5"` thành `5`), còn class-validator chạy các luật trên decorator; sai luật thì Nest tự trả về 400 với danh sách lỗi, mình không phải viết tay. Mình hay bật thêm `whitelist: true` để tự động loại bỏ field thừa không khai báo trong DTO, tránh dữ liệu rác lọt vào. Điều này quan trọng vì nó là tuyến phòng thủ đầu tiên ngay ở biên: service và Prisma luôn nhận dữ liệu sạch, đúng kiểu, nên giảm bug và tránh các trường hợp dữ liệu không hợp lệ chạm tới DB."

---

- DTO = class mô tả "hình dạng + luật" của dữ liệu vào/ra — hợp đồng API, không phải model DB (Prisma model là thứ khác).
- `ValidationPipe` (bật global trong `main.ts`) chạy trước handler, dùng **class-transformer** + **class-validator**.
- `transform: true` → biến plain JSON thành **instance DTO** và ép kiểu theo khai báo (vd `"5"` → `5`).
- `whitelist: true` → tự động **xoá field không khai báo** trong DTO (chống dữ liệu thừa/rác).
- Sai luật → Nest tự trả **400 Bad Request** kèm message lỗi, không cần check tay trong service.
- Vì sao quan trọng: validate ở biên giúp service/Prisma luôn nhận dữ liệu **sạch & đúng kiểu** → ít bug, an toàn hơn.

```ts
// create-issue.dto.ts
export class CreateIssueDto {
  @IsString() @MinLength(1)
  title: string;

  @IsUUID()
  projectId: string;
}

// main.ts — bật global
app.useGlobalPipes(new ValidationPipe({ whitelist: true, transform: true }));
// body sai luật -> 400 tự động, không tới service
```

</details>

---

## 3. Configuration & Environment

<details>
<summary><b>3.1 Quản lý biến môi trường/secret trong NestJS thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Để quản lý biến môi trường và secret trong NestJS, mình dùng `@nestjs/config`, cụ thể là `ConfigModule` để đọc file `.env`, rồi truy cập giá trị qua `ConfigService.get('KEY')` thay vì đọc thẳng `process.env` rải rác khắp nơi. Nguyên tắc quan trọng nhất là không bao giờ hardcode secret như database URL hay JWT secret vào code, và file `.env` phải được gitignore để không lỡ tay đẩy thông tin nhạy cảm lên Git. Để team vẫn biết cần những biến nào, mình commit một file `.env.example` chỉ chứa tên biến mà không có giá trị thật, coi như mẫu để người mới copy ra. Ở production thì các giá trị này thường được inject qua biến môi trường của hệ thống (Docker, CI/CD) chứ không dùng file `.env` thật. Cách làm này giúp tách config ra khỏi code, mỗi môi trường dev/staging/prod chỉ cần đổi giá trị mà không phải sửa code."

---

- Dùng `@nestjs/config` (`ConfigModule`) đọc file `.env`, truy cập qua `ConfigService.get('KEY')`.
- `.env` phải được **gitignore**, không hardcode secret vào code.
- Commit `.env.example` (chỉ tên biến, không giá trị) làm mẫu cho team.

</details>

<details>
<summary><b>3.2 <code>forRoot</code> vs <code>forRootAsync</code> khác gì?</b></summary>

**📌 Câu trả lời mẫu:**

"Khác biệt nằm ở chỗ cấu hình có cần phụ thuộc thứ khác hay không. `forRoot()` dùng khi mình biết sẵn cấu hình ngay lúc viết code, truyền vào một object tĩnh là xong. Còn `forRootAsync()` dùng khi giá trị cấu hình phải lấy bất đồng bộ hoặc phụ thuộc vào một provider khác phải được nạp trước — phổ biến nhất là phụ thuộc `ConfigService` để đọc biến môi trường. Lúc đó mình dùng `useFactory` để trả về object cấu hình và `inject` để khai báo dependency cần tiêm vào factory đó. Ví dụ kết nối database: mình không hardcode connection string mà dùng `forRootAsync` rồi lấy giá trị từ `ConfigService` đã đọc từ `.env`. Nói gọn, cần config động hoặc dựa trên dependency thì dùng `forRootAsync`, còn config cứng đơn giản thì `forRoot` cho nhanh."

---

- `forRoot({...})`: cấu hình **tĩnh**, biết sẵn lúc viết code.
- `forRootAsync({...})`: cấu hình **phụ thuộc dependency phải nạp trước** (vd `ConfigService`).
- Dùng `useFactory` + `inject` để lấy giá trị bất đồng bộ.

```ts
TypeOrmModule.forRootAsync({
  inject: [ConfigService],
  useFactory: (cfg: ConfigService) => ({ /* ... */ }),
});
```

</details>

<details>
<summary><b>3.3 <code>synchronize: true</code> làm gì? Có nên bật ở production?</b></summary>

**📌 Câu trả lời mẫu:**

"`synchronize: true` là tính năng của TypeORM tự động đồng bộ schema database cho khớp với entity mỗi lần app khởi động — tức là nó tự chạy `CREATE` hoặc `ALTER` bảng theo những gì mình khai báo trong entity. Lúc dev thì rất tiện vì mình sửa entity là DB tự cập nhật, không phải viết migration. Nhưng tuyệt đối không được bật ở production, vì nó có thể tự ý xoá cột hoặc đổi cấu trúc bảng dẫn tới mất dữ liệu thật mà không hề cảnh báo, và mình không kiểm soát được nó làm gì. Ở production thì cách đúng là dùng migration — tức là viết các script thay đổi schema có kiểm soát, được review trước khi chạy và rollback được nếu có sự cố. Trong Jira Clone mình dùng Prisma thì tinh thần cũng tương tự: dev có thể `db push` nhanh, nhưng production luôn đi qua `migrate` để mọi thay đổi schema đều có lịch sử rõ ràng."

---

- Tự động `CREATE/ALTER` bảng cho khớp entity **mỗi lần khởi động**.
- Tiện khi dev, nhưng 🔴 **KHÔNG** dùng production: có thể xoá cột → **mất dữ liệu thật**.
- Production dùng **migrations** (script đổi schema có kiểm soát, review & rollback được).

</details>

<details>
<summary><b>3.4 Biến môi trường được đọc khi nào? Sửa <code>.env</code> lúc app đang chạy có ăn không?</b></summary>

**📌 Câu trả lời mẫu:**

"Biến môi trường được đọc đúng một lần lúc tiến trình khởi động, cụ thể là khi `ConfigModule.forRoot()` chạy lúc bootstrap app. Sau khi app đã chạy rồi, nếu mình mở file `.env` ra sửa giá trị thì thay đổi đó hoàn toàn không có hiệu lực, vì giá trị đã được nạp vào bộ nhớ từ trước. Một điểm hay khiến người mới nhầm là chạy `nest start --watch`: chế độ watch chỉ theo dõi các file `.ts` để compile lại khi code đổi, nó không reload file `.env`. Nên mỗi khi đổi biến môi trường, mình phải restart lại app thì giá trị mới mới được đọc. Hiểu điều này giúp mình tránh ngồi debug nhầm khi sửa `.env` mãi mà app vẫn chạy giá trị cũ."

---

- Đọc **một lần** lúc tiến trình khởi động (`ConfigModule.forRoot()` khi bootstrap).
- Sửa `.env` lúc app đang chạy **KHÔNG** có hiệu lực.
- `nest start --watch` chỉ theo dõi file `.ts`, không reload `.env` → phải **restart** app.

</details>

---

## 4. Database, TypeORM & Entities

<details>
<summary><b>4.1 Active Record vs Data Mapper? NestJS dùng cái nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Đây là hai pattern để map object với database. Active Record là khi bản thân entity tự biết cách lưu mình, ví dụ `user.save()` — tức là logic truy cập DB nằm luôn trên model. Còn Data Mapper thì tách ra: entity chỉ là dữ liệu thuần, và có một `Repository` riêng lo việc đọc/ghi, kiểu `repo.save(user)`. TypeORM hỗ trợ cả hai, nhưng NestJS thường đi theo Data Mapper / Repository pattern. Lý do là nó hợp với Dependency Injection — mình inject repository vào service như một dependency bình thường, dễ mock khi test, và tách bạch rõ logic nghiệp vụ khỏi logic lưu trữ. Trong Jira Clone mình dùng Prisma, về tinh thần cũng giống Data Mapper: `PrismaService` đóng vai trò lớp truy cập dữ liệu riêng, còn các model chỉ là dữ liệu thuần, nên service vẫn gọn và dễ test."

---

- **Active Record**: entity tự lưu (`user.save()`), logic nằm trên model.
- **Data Mapper**: `Repository` riêng lo lưu (`repo.save(user)`), entity là data thuần.
- TypeORM hỗ trợ cả hai; NestJS dùng **Data Mapper / Repository pattern** vì hợp **DI**, tách logic, dễ test.

</details>

<details>
<summary><b>4.2 UUID vs auto-increment (serial) cho khóa chính?</b></summary>

**📌 Câu trả lời mẫu:**

"Khi chọn kiểu khóa chính, về cơ bản mình cân giữa UUID và serial (auto-increment). UUID là một chuỗi ngẫu nhiên 128-bit, ưu điểm là duy nhất trên toàn hệ thống và khó đoán, nên client không thể dò ID kiểu tăng dần 1, 2, 3 để mò sang tài nguyên của người khác; nó cũng rất hợp khi có nhiều service hoặc nhiều DB tự sinh ID mà không sợ trùng. Đổi lại UUID tốn 16 byte và mặc định không tuần tự, nên việc chèn ngẫu nhiên vào B-tree index dễ gây phân mảnh, hơi chậm hơn — cái này có thể khắc phục bằng uuidv7 vì nó có phần thời gian nên gần như tăng dần. Còn serial thì nhỏ gọn, tăng tuần tự nên ghi vào index nhanh và đẹp, nhưng nhược điểm là dễ đoán và khó gộp dữ liệu giữa nhiều nguồn vì ID dễ đụng nhau. Với Jira Clone mình thiên về UUID cho các entity public như issue hay project, vừa an toàn vừa dễ mở rộng sau này."

---

- **UUID**: duy nhất toàn cục, khó đoán (chống dò ID), hợp hệ phân tán. Nhược: 16 byte, không tuần tự → phân mảnh index (`uuidv7` khắc phục).
- **Serial**: nhỏ gọn, tuần tự, nhanh. Nhược: dễ đoán, khó merge giữa nhiều DB.

</details>

<details>
<summary><b>4.3 Soft delete là gì?</b></summary>

**📌 Câu trả lời mẫu:**

"Soft delete là kiểu xóa 'mềm': thay vì thật sự xóa dòng dữ liệu khỏi DB, mình chỉ đánh dấu nó đã bị xóa bằng cách set một cột thời gian như deletedAt. Bản chất là bản ghi vẫn còn nguyên trong bảng, nhưng các truy vấn bình thường sẽ tự động bỏ qua những dòng đã có deletedAt. Lợi ích lớn nhất là mình giữ được lịch sử và có thể khôi phục lại khi lỡ xóa nhầm, đồng thời không làm vỡ các quan hệ tham chiếu tới nó. Trong một dự án như Jira Clone, soft delete rất hợp cho issue hay comment vì người dùng có thể muốn khôi phục, và team cũng cần audit xem ai đã xóa cái gì. Đánh đổi là bảng sẽ phình to dần và mọi query phải nhớ lọc deletedAt, nên với dữ liệu thật sự cần biến mất vĩnh viễn (ví dụ theo yêu cầu GDPR) thì mình vẫn phải hard delete."

---

- Không xoá hẳn dòng, chỉ set cột `deletedAt` (`@DeleteDateColumn` + `softRemove`/`softDelete`).
- Giữ được lịch sử, khôi phục được.
- Query mặc định tự bỏ qua bản ghi đã soft-delete.

</details>

---

## 5. DTO, Validation & Pipes

<details>
<summary><b>5.1 DTO khác Entity thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình phân biệt DTO và Entity theo đúng vai trò của chúng. DTO (Data Transfer Object) là class mô tả hình dạng và luật của dữ liệu đi vào hoặc đi ra qua API — nó như một hợp đồng giữa client và server, ví dụ CreateIssueDto quy định request body phải có title và projectId. Còn Entity (hoặc model bên Prisma) là thứ ánh xạ trực tiếp tới bảng trong database. Lý do mình tách hai thứ này ra là để API và DB không bị dính cứng vào nhau: nếu sau này mình đổi cấu trúc bảng DB thì hợp đồng API không vỡ, và ngược lại mình có thể thêm trường nội bộ trong DB mà không vô tình lộ ra ngoài cho client. Một lợi ích thực tế nữa là DTO giúp mình kiểm soát chính xác client được gửi gì và mình trả về gì, tránh lộ những cột nhạy cảm như passwordHash."

---

- **DTO** (Data Transfer Object): mô tả hình dạng + luật của dữ liệu **vào/ra** — hợp đồng API.
- **Entity**: ánh xạ **bảng DB**.
- Tách biệt để API và DB không phụ thuộc cứng: đổi DB không vỡ API và ngược lại.

</details>

<details>
<summary><b>5.2 Pipe trong NestJS là gì? Chạy lúc nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Pipe trong NestJS là thành phần lo việc biến đổi (transform) và kiểm tra (validate) dữ liệu đầu vào trước khi nó tới tay route handler. Về thời điểm chạy, Pipe nằm sau Guard và ngay trước handler trong vòng đời request — tức là khi đó Nest đã biết request được phép đi tiếp, giờ mới làm sạch dữ liệu. Có hai nhóm việc Pipe hay làm: một là transform, ví dụ ParseIntPipe đổi :id từ string sang number; hai là validate, điển hình là ValidationPipe dùng class-validator và class-transformer để kiểm tra DTO theo các decorator như @IsString hay @IsUUID. Nếu dữ liệu sai luật thì Pipe tự ném lỗi và Nest trả về 400, request không bao giờ chạm tới service. Trong Jira Clone mình bật ValidationPipe global trong main.ts, coi nó như tuyến phòng thủ đầu tiên để service luôn nhận được dữ liệu sạch và đúng kiểu."

---

- Pipe **biến đổi / validate** dữ liệu đầu vào.
- Chạy **trước handler** (sau Guard) trong request lifecycle.
- `ValidationPipe` dùng `class-validator` + `class-transformer` để kiểm tra DTO.

```ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true, forbidNonWhitelisted: true, transform: true,
}));
```

</details>

<details>
<summary><b>5.3 <code>whitelist</code> vs <code>forbidNonWhitelisted</code> vs <code>transform</code>?</b></summary>

**📌 Câu trả lời mẫu:**

"Đây là ba option mình hay bật cho ValidationPipe và chúng giải quyết các việc khác nhau. whitelist sẽ âm thầm loại bỏ những field không được khai báo trong DTO — ví dụ client gửi thừa một field lạ thì Nest tự cắt đi, giúp dữ liệu rác không lọt vào service. forbidNonWhitelisted thì mạnh tay hơn: thay vì cắt im lặng, nó báo lỗi 400 ngay khi phát hiện field lạ, hữu ích khi mình muốn API nghiêm khắc và bắt client gửi đúng. Còn transform thì khác hẳn, nó biến payload JSON thô thành đúng instance của class DTO và ép kiểu theo khai báo, ví dụ '5' từ query string thành số 5. Trong Jira Clone mình thường bật cả ba: whitelist và transform để dữ liệu vừa sạch vừa đúng kiểu, và bật forbidNonWhitelisted khi muốn phát hiện sớm client gửi sai schema thay vì âm thầm bỏ qua."

---

- `whitelist`: **âm thầm xoá** field không khai báo trong DTO.
- `forbidNonWhitelisted`: gửi field lạ → **báo lỗi 400**.
- `transform`: chuyển payload thô thành **instance DTO** + ép kiểu (vd `"5"` → `5`).

</details>

<details>
<summary><b>5.4 <code>PartialType</code> dùng để làm gì?</b></summary>

**📌 Câu trả lời mẫu:**

"PartialType là một hàm tiện ích của NestJS giúp mình tạo nhanh một DTO mới bằng cách kế thừa toàn bộ field và luật validation từ một DTO có sẵn, nhưng biến mọi field thành optional. Bản chất nó để xử lý đúng tinh thần của PATCH — cập nhật một phần — vì khi PATCH thì client chỉ gửi vài field cần đổi chứ không gửi đủ. Ví dụ trong Jira Clone mình đã có CreateIssueDto với các luật như @IsString cho title và @IsUUID cho projectId; thay vì viết lại UpdateIssueDto và lặp y nguyên đống decorator đó nhưng thêm optional, mình chỉ cần viết PartialType(CreateIssueDto). Lợi ích chính là DRY: luật validation chỉ định nghĩa một chỗ, nên khi sửa luật mình không phải sửa hai nơi và tránh được tình trạng hai DTO bị lệch nhau."

---

- Kế thừa **toàn bộ luật validation** nhưng biến mọi field thành **optional**.
- Hợp cho `PATCH` (cập nhật một phần).
- Giúp **DRY**, không lặp lại `@IsEmail`, `@MinLength`...

Ví dụ: `PartialType(CreateUserDto)`.

</details>

<details>
<summary><b>5.5 Vì sao truyền id sai định dạng lại ra 500 thay vì 404?</b></summary>

**📌 Câu trả lời mẫu:**

"Cái này mình từng gặp và bản chất nằm ở thứ tự xử lý. Khi client truyền một id sai định dạng — ví dụ một chuỗi không phải UUID hợp lệ vào một cột kiểu uuid — thì câu truy vấn sẽ vỡ ngay ở tầng database với lỗi kiểu 'invalid input syntax for type uuid'. Lỗi này là một exception không lường trước nên Nest mặc định trả về 500, chứ không phải 404. Điểm mấu chốt là lỗi xảy ra TRƯỚC khi code chạy tới logic 'tìm không thấy thì ném NotFoundException', vì DB còn chưa chạy nổi câu query để mà biết có tìm thấy hay không. Cách khắc phục của mình là chặn ngay ở biên bằng ParseUUIDPipe trên param: nếu id không đúng định dạng UUID thì Pipe trả về 400 luôn trước khi chạm DB, vừa đúng ngữ nghĩa (client gửi sai) vừa tránh phơi lỗi nội bộ ra ngoài."

---

- Param sai kiểu cột (chuỗi không phải UUID vào cột `uuid`) → truy vấn **vỡ ngay ở tầng DB** (`invalid input syntax for type uuid`) → 500.
- Lỗi xảy ra **trước** logic "không tìm thấy → 404".
- Khắc phục: `ParseUUIDPipe` chặn param sai → **400** trước khi chạm DB.

</details>

---

## 6. REST API & Controllers

<details>
<summary><b>6.1 PUT vs PATCH?</b></summary>

**📌 Câu trả lời mẫu:**

"PUT và PATCH đều dùng để cập nhật resource nhưng khác nhau ở phạm vi. PUT mang ý nghĩa thay thế toàn bộ resource, nên về nguyên tắc client phải gửi đủ tất cả các field; những field không gửi có thể bị coi là rỗng. Còn PATCH là cập nhật một phần, client chỉ gửi đúng những field muốn đổi, còn lại giữ nguyên. Trong thực tế ở các API NestJS như Jira Clone, mình gần như luôn dùng PATCH cho việc sửa, ví dụ chỉ đổi status của một issue mà không phải gửi lại cả title lẫn description. Đi kèm với PATCH mình dùng PartialType cho DTO để mọi field thành optional mà vẫn giữ nguyên luật validation. Một điểm hay nhắc nữa là PUT có tính idempotent — gọi nhiều lần cho cùng dữ liệu thì kết quả không đổi — còn PATCH thì không phải lúc nào cũng đảm bảo điều đó, tùy cách mình thiết kế."

---

- **PUT**: thay **toàn bộ** resource (gửi đủ mọi field).
- **PATCH**: cập nhật **một phần** (chỉ field cần đổi).
- Trong Nest thường dùng PATCH + `PartialType` cho DTO.

</details>

<details>
<summary><b>6.2 Nested route (route lồng nhau) dùng khi nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Nested route, hay route lồng nhau, là khi mình thiết kế URL phản ánh quan hệ phụ thuộc giữa các tài nguyên — thường dùng khi một resource con không tồn tại độc lập mà luôn thuộc về một resource cha. Ví dụ trong Jira Clone, một comment luôn thuộc về một issue, nên mình đặt route kiểu /issues/:issueId/comments; khi đó tạo comment là POST /issues/123/comments, và mình lấy id của cha qua @Param('issueId'). Cách này giúp URL tự diễn đạt được ngữ cảnh theo đúng chuẩn REST, người đọc API nhìn vào là hiểu ngay quan hệ. Đánh đổi là không nên lồng quá sâu; thường mình chỉ lồng một cấp, vì URL kiểu /a/:x/b/:y/c/:z vừa khó dùng vừa khó maintain. Nếu resource con cũng có ý nghĩa độc lập thì mình vẫn nên cho nó một route phẳng riêng."

---

- Khi một tài nguyên **phụ thuộc** tài nguyên khác (vd comment thuộc post).
- URL phản ánh quan hệ theo chuẩn REST; lấy id cha qua `@Param('postId')`.
- Ví dụ: `@Controller('posts/:postId/comments')` -> `POST /posts/<id>/comments` thêm comment vào bài đó.

</details>

---

## 7. Authentication & Authorization

<details>
<summary><b>7.1 Authentication vs Authorization?</b></summary>

**📌 Câu trả lời mẫu:**

"Hai khái niệm này nghe giống nhau nhưng vai trò khác hẳn. Authentication, viết tắt AuthN, trả lời câu hỏi 'bạn là ai?' — tức là xác minh danh tính, ví dụ user đăng nhập bằng email và mật khẩu, hoặc server verify chữ ký của JWT để biết request đến từ ai. Còn Authorization, viết tắt AuthZ, trả lời câu hỏi 'bạn được phép làm gì?' — tức là sau khi đã biết bạn là ai rồi, hệ thống mới kiểm tra bạn có quyền với hành động đó không, ví dụ chỉ admin mới được xóa project. Nguyên tắc là luôn AuthN trước rồi mới AuthZ, vì phải biết bạn là ai thì mới xét được quyền. Trong NestJS của Jira Clone mình tách rõ: JwtAuthGuard lo phần xác thực danh tính, còn RolesGuard và logic kiểm tra ownership lo phần phân quyền. Một điểm hay nhầm là mã lỗi: chưa xác thực thì trả 401, còn đã xác thực nhưng không đủ quyền thì trả 403."

---

- **Authentication (AuthN)**: "bạn **là ai**?" — xác minh danh tính (đăng nhập email/mật khẩu).
- **Authorization (AuthZ)**: "bạn **được phép** làm gì?" — kiểm tra quyền (vd chỉ admin xoá bài).
- Luôn AuthN trước, AuthZ sau.

</details>

<details>
<summary><b>7.2 Vì sao hash mật khẩu? bcrypt khác MD5/SHA thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình hash mật khẩu vì nếu database bị lộ thì kẻ tấn công cũng không đọc được mật khẩu gốc, do hash là hàm một chiều — không thể đảo ngược ra mật khẩu ban đầu. Điểm khác biệt là bcrypt được thiết kế CHẬM có chủ đích và tự sinh salt cho mỗi user. Salt làm cho hai người dùng cùng mật khẩu vẫn ra hash khác nhau, chống được rainbow table; còn việc chậm khiến brute-force trở nên cực kỳ tốn kém. Ngược lại MD5/SHA được thiết kế để chạy thật nhanh phục vụ kiểm tra toàn vẹn dữ liệu, nên kẻ tấn công có thể thử hàng tỷ mật khẩu mỗi giây — hoàn toàn không phù hợp cho mật khẩu. Trong dự án Jira Clone mình dùng bcrypt với cost factor để cân bằng giữa bảo mật và tốc độ đăng nhập, ngoài ra argon2 cũng là lựa chọn hiện đại tương đương."

---

- Hash để DB lộ thì mật khẩu không "trần trụi"; hash là **một chiều**, không đảo ngược.
- bcrypt **có salt** (chống rainbow table) và **chậm có chủ đích** (cost factor → chống brute-force).
- MD5/SHA quá nhanh → không hợp cho mật khẩu.
- Lựa chọn khác: `argon2`.

</details>

<details>
<summary><b>7.3 JWT gồm mấy phần? Vì sao gọi là stateless?</b></summary>

**📌 Câu trả lời mẫu:**

"JWT gồm 3 phần ngăn cách bằng dấu chấm: Header chứa thuật toán ký, Payload chứa thông tin user như sub, role, exp, và Signature là chữ ký được tạo từ secret của server. Gọi là stateless vì toàn bộ thông tin cần thiết đã nằm sẵn trong token rồi, server không cần lưu session ở đâu cả. Khi nhận request, server chỉ cần dùng secret để verify chữ ký là biết token có hợp lệ và chưa bị sửa hay không, từ đó xác định được user là ai. Ưu điểm là dễ scale ngang vì không có session chung giữa các server, nhưng đánh đổi là khó thu hồi token ngay lập tức — đó là lý do mình thường để access token thời hạn ngắn."

---

- 3 phần ngăn bởi `.`: **Header** (thuật toán) . **Payload** (`sub`, `role`, `exp`...) . **Signature** (chữ ký bằng secret).
- Server chỉ cần **verify chữ ký** bằng secret là biết user là ai.
- Không cần lưu session ⇒ stateless.

</details>

<details>
<summary><b>7.4 Nên để gì trong JWT payload?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình chỉ để dữ liệu định danh tối thiểu trong payload, chủ yếu là sub (id của user, đây là claim chuẩn của JWT), role để phân quyền, và đôi khi email. Lý do quan trọng nhất là payload chỉ được mã hóa base64 chứ không hề được mã hóa bí mật — bất kỳ ai cầm token đều decode đọc được nội dung, chỉ có chữ ký mới được bảo vệ. Vì vậy tuyệt đối không bỏ dữ liệu nhạy cảm như mật khẩu hay token khác vào đó. Mình cũng tránh nhồi quá nhiều dữ liệu vì token sẽ gửi kèm mỗi request, để nhỏ gọn sẽ nhẹ hơn; những thông tin chi tiết khác mình query lại từ DB khi cần."

---

- Chỉ dữ liệu định danh tối thiểu: `sub` (id user — chuẩn JWT), `role`, có thể `email`.
- **Không** để dữ liệu nhạy cảm (mật khẩu, token khác).
- Payload chỉ **base64** → ai cũng giải mã đọc được; chỉ chữ ký mới được bảo vệ.

</details>

<details>
<summary><b>7.5 Vì sao message lỗi đăng nhập nên mập mờ?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình trả về message chung chung kiểu 'Email hoặc mật khẩu không đúng' thay vì nói rõ cái nào sai. Bản chất là để chống user enumeration — nếu hệ thống báo 'email không tồn tại' thì kẻ tấn công có thể thử hàng loạt email để dò xem tài khoản nào có thật, rồi tập trung tấn công vào đó. Khi message mập mờ, họ không phân biệt được là email sai hay mật khẩu sai, nên không thu thập được thông tin gì hữu ích. Đây là một đánh đổi nhỏ về trải nghiệm để đổi lấy bảo mật, và trong context đăng nhập thì bảo mật quan trọng hơn."

---

- Trả "Email hoặc mật khẩu không đúng" — không nói rõ cái nào sai.
- Tránh **lộ email nào tồn tại** trong hệ thống (chống dò tài khoản / user enumeration).

</details>

<details>
<summary><b>7.6 Guard là gì? Chạy lúc nào trong request lifecycle?</b></summary>

**📌 Câu trả lời mẫu:**

"Guard trong NestJS là thành phần quyết định một request có được đi tiếp vào handler hay không, thông qua method canActivate() trả về true hoặc false. Trong request lifecycle nó chạy theo thứ tự Middleware rồi đến Guard, sau đó mới tới Interceptor, Pipe và cuối cùng là Handler. Vị trí này rất hợp lý vì Guard chạy trước khi vào logic xử lý, nên dùng nó để làm xác thực (authentication) và phân quyền (authorization) là chuẩn nhất — nếu user không hợp lệ thì chặn ngay từ đầu, không tốn công chạy tiếp. Trong Jira Clone mình dùng JwtAuthGuard để bảo vệ các route cần đăng nhập, ví dụ route lấy profile."

---

- Guard quyết định request có được đi tiếp không (`canActivate()` trả `boolean`).
- Thứ tự: **Middleware → Guard → Interceptor → Pipe → Handler**.
- Hợp cho **AuthN/AuthZ**.

```ts
@UseGuards(JwtAuthGuard)
@Get('profile')
getProfile(@CurrentUser() user) { return user; }
```

</details>

<details>
<summary><b>7.7 <code>AuthGuard('jwt')</code> kết nối với <code>JwtStrategy</code> thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Sợi dây nối giữa AuthGuard('jwt') và JwtStrategy chính là cái tên 'jwt' — đó là tên mặc định mà passport-jwt đăng ký, nên khi mình viết AuthGuard('jwt') thì Nest biết phải gọi đúng strategy đó. Khi một request đi qua guard, Passport sẽ chạy strategy: nó lấy token từ header Authorization Bearer, verify chữ ký và kiểm tra hạn, rồi gọi method validate(payload). Giá trị mà validate trả về sẽ được Passport tự động gắn vào request.user, nhờ vậy ở trong controller mình truy cập được thông tin user đang đăng nhập. Nói ngắn gọn, guard là cái kích hoạt, còn strategy là nơi định nghĩa cách verify token và trích xuất user."

---

- Kết nối qua **tên strategy** `'jwt'` (mặc định của `passport-jwt`).
- Guard kích hoạt → Passport chạy strategy: lấy token từ header `Authorization: Bearer`, verify chữ ký + hạn, gọi `validate(payload)`.
- Giá trị `validate` trả về được gắn vào **`request.user`**.

</details>

<details>
<summary><b>7.8 Guard khác Middleware thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Khác biệt lớn nhất là Guard biết được ngữ cảnh của Nest thông qua ExecutionContext — nó biết request này sẽ vào handler nào, class nào, và đọc được metadata gắn trên route, ví dụ role yêu cầu. Nhờ vậy Guard rất hợp để phân quyền theo từng route hoặc theo role. Còn Middleware về bản chất chỉ là hàm Express thuần, chạy sớm hơn trong pipeline và không hề biết handler nào sẽ xử lý request, nên nó hợp với các việc chung chung như logging, parse body, gắn header CORS. Quy tắc của mình là: cần logic liên quan tới route hoặc quyền thì dùng Guard, còn xử lý kỹ thuật chung cho mọi request thì dùng Middleware."

---

- Guard biết **ngữ cảnh Nest** (`ExecutionContext` — handler/class, metadata) → hợp phân quyền theo route/role.
- Middleware là hàm Express thuần, chạy sớm hơn, không biết handler nào sẽ xử lý.

</details>

<details>
<summary><b>7.9 JWT hết hạn thì sao?</b></summary>

**📌 Câu trả lời mẫu:**

"Khi token hết hạn, với cấu hình ignoreExpiration: false thì Passport tự kiểm tra claim exp và trả về 401 ngay, không cần mình viết thêm logic. Lúc đó client buộc phải đăng nhập lại để lấy token mới. Để trải nghiệm tốt hơn mà vẫn an toàn, cách phổ biến là dùng cơ chế refresh token: access token để thời hạn ngắn (vài phút đến vài chục phút) để giảm rủi ro nếu bị lộ, còn refresh token thời hạn dài hơn được lưu an toàn để xin cấp lại access token mới mà không bắt user gõ lại mật khẩu. Đây là đánh đổi giữa bảo mật và tiện lợi — token càng ngắn càng an toàn nhưng phải làm mới thường xuyên hơn."

---

- Với `ignoreExpiration: false`, Passport tự trả **401** khi token quá hạn (`exp`).
- Client phải đăng nhập lại.
- Hoặc dùng **refresh token** (access token ngắn hạn + refresh token dài hạn) để cấp lại mà không bắt login.

</details>

<details>
<summary><b>7.10 RBAC là gì?</b></summary>

**📌 Câu trả lời mẫu:**

"RBAC là Role-Based Access Control, tức là kiểm soát truy cập dựa trên vai trò của người dùng thay vì gán quyền lẻ tẻ cho từng người. Ý tưởng là mỗi user thuộc một role như admin, author hay reader, và mỗi route sẽ khai báo những role nào được phép truy cập; chỉ khi role của user khớp thì mới cho qua. Đây là hình thức authorization phổ biến nhất vì nó đơn giản và dễ quản lý — khi thêm người mới mình chỉ cần gán role thay vì cấu hình lại từng quyền. Trong Jira Clone mình dùng RBAC để phân biệt ai được tạo project, ai chỉ được xem. Khi hệ thống cần quyền chi tiết hơn theo từng tài nguyên thì người ta mới chuyển sang các mô hình phức tạp hơn như ABAC."

---

- **Role-Based Access Control** — kiểm soát truy cập dựa trên **vai trò** (admin/author/reader...).
- Mỗi route khai báo role được phép; user có role phù hợp mới qua.
- Là hình thức **Authorization** phổ biến nhất.

</details>

<details>
<summary><b>7.11 <code>@Roles</code> + <code>RolesGuard</code> phối hợp thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Hai cái này phối hợp theo kiểu một bên gắn nhãn, một bên đọc nhãn. Decorator @Roles('admin') thực chất dùng SetMetadata để gắn thông tin role yêu cầu lên handler hoặc controller. Khi request đến, RolesGuard sẽ dùng Reflector để đọc lại cái nhãn đó, rồi so sánh với role của user đang đăng nhập trong request.user. Nếu khớp thì cho đi tiếp, không khớp thì trả về 403 Forbidden. Điểm hay của cách này là logic phân quyền được tách bạch và khai báo rất rõ ràng ngay trên route — nhìn vào là biết route đó cần quyền gì. Một lưu ý là nếu route không gắn @Roles thì guard nên cho qua, để các route công khai hoặc chỉ cần đăng nhập không bị chặn nhầm."

---

- `@Roles('admin')` dùng `SetMetadata` **gắn nhãn** role yêu cầu lên handler.
- `RolesGuard` dùng `Reflector` **đọc lại** nhãn rồi so với `request.user.role`.
- Khớp → cho qua; không → **403**.

```ts
export const Roles = (...roles: UserRole[]) => SetMetadata('roles', roles);

@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}
  canActivate(ctx: ExecutionContext): boolean {
    const required = this.reflector.getAllAndOverride<UserRole[]>('roles', [
      ctx.getHandler(), ctx.getClass(),
    ]);
    if (!required?.length) return true;
    const { user } = ctx.switchToHttp().getRequest();
    return required.some((r) => user?.role === r);
  }
}
```

</details>

<details>
<summary><b>7.12 Thứ tự nhiều Guard trong <code>@UseGuards(A, B)</code>?</b></summary>

**📌 Câu trả lời mẫu:**

"Khi mình viết `@UseGuards(A, B)` thì các Guard chạy lần lượt từ trái sang phải, tức A chạy trước rồi mới tới B. Điểm quan trọng là thứ tự này có ý nghĩa khi mình kết hợp xác thực và phân quyền: trong Jira Clone mình luôn đặt `JwtAuthGuard` trước `RolesGuard`. Lý do là `JwtAuthGuard` mới là cái verify token rồi gắn thông tin user vào `request.user`; còn `RolesGuard` lại cần đọc `request.user.role` để biết user có đủ quyền không. Nếu mình đảo ngược, để `RolesGuard` chạy trước thì lúc đó `request.user` còn chưa có gì nên check role sẽ sai hoàn toàn. Nói gọn là Guard nào tạo ra dữ liệu thì phải đứng trước Guard dùng dữ liệu đó."

---

- Chạy **lần lượt** trái sang phải: A trước, B sau.
- Với auth phải để `JwtAuthGuard` **trước** `RolesGuard`.
- Vì `RolesGuard` cần `request.user` (do JwtAuthGuard/strategy gắn) mới biết role để kiểm tra.

</details>

<details>
<summary><b>7.13 Vì sao không nên cho user tự đăng ký role <code>admin</code>?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình không bao giờ cho phép client tự gửi field `role` khi đăng ký, vì đó chính là lỗ hổng privilege escalation: nếu DTO đăng ký nhận `role` thì bất kỳ ai cũng có thể thêm `role: 'admin'` vào body và tự biến mình thành admin. Nguyên tắc của mình là quyền hạn phải do hệ thống quyết định, không tin dữ liệu từ phía client. Cụ thể, khi user đăng ký thì server tự gán role mặc định thấp nhất (ví dụ `MEMBER` hoặc `reader`), còn quyền admin chỉ được tạo qua seed/migration lúc khởi tạo hệ thống, hoặc do một admin có sẵn cấp cho người khác. Vì vậy mình cố tình không để `role` trong register DTO, và nếu lỡ client gửi lên thì `whitelist: true` của ValidationPipe cũng tự loại bỏ."

---

- Đó là lỗ hổng **privilege escalation** — ai cũng tự thành admin.
- Role nên do **hệ thống** quyết định (mặc định `reader`); admin tạo qua **seed/migration** hoặc do admin khác cấp.
- Không để `role` trong DTO đăng ký công khai.

</details>

<details>
<summary><b>7.14 RBAC vs ABAC (ownership-based)?</b></summary>

**📌 Câu trả lời mẫu:**

"Cả hai đều là cách phân quyền nhưng dựa trên tiêu chí khác nhau. RBAC (Role-Based) xét theo vai trò của user, ví dụ admin, author hay reader, kiểu 'role này được làm gì'. Còn ABAC hay ownership-based thì xét theo thuộc tính của chính tài nguyên, ví dụ luật 'chỉ tác giả mới được sửa bài của mình' tức là so `post.author.id === user.id`. RBAC đơn giản và đủ dùng cho các quyền chung chung, nhưng nó không diễn đạt được luật phụ thuộc ngữ cảnh kiểu 'của ai thì người đó sửa'. Trong thực tế mình thường kết hợp cả hai: trong Jira Clone, admin của project thì vượt quyền làm mọi thứ (RBAC), còn người tạo issue thì mới được sửa issue đó (ownership). Như vậy vừa gọn vừa linh hoạt."

---

- **RBAC** xét **vai trò** (admin/author/reader).
- **ABAC / ownership-based** xét **thuộc tính của tài nguyên** — vd "chỉ tác giả mới sửa bài đó" (`post.author.id === user.id`).
- ABAC linh hoạt hơn cho luật phụ thuộc ngữ cảnh; thường kết hợp cả hai (admin vượt quyền + chủ sở hữu).

</details>

<details>
<summary><b>7.15 401 Unauthorized vs 403 Forbidden?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình phân biệt hai cái này theo câu hỏi 'danh tính' hay 'quyền hạn'. 401 Unauthorized nghĩa là chưa xác thực: user chưa đăng nhập, hoặc token bị thiếu, sai, hết hạn, nên hệ thống còn chưa biết bạn là ai. Còn 403 Forbidden nghĩa là đã biết bạn là ai rồi nhưng bạn không đủ quyền làm việc này, ví dụ một member bình thường cố xoá project mà chỉ admin mới được xoá, hoặc cố sửa issue không phải của mình. Cách nhớ của mình rất đơn giản: 401 là về việc bạn là ai, 403 là về việc bạn được phép làm gì. Trong NestJS thì `JwtAuthGuard` thường ném ra 401, còn `RolesGuard` hay check ownership thì ném ra 403."

---

- **401** = **chưa xác thực** (chưa đăng nhập / token sai/thiếu/hết hạn).
- **403** = **đã xác thực nhưng không đủ quyền** (sai role, không phải chủ tài nguyên).
- Nhớ: 401 về *danh tính*, 403 về *quyền hạn*.

</details>

---

## 8. Docker & Debugging thực chiến

<details>
<summary><b>8.1 Ý nghĩa <code>5433:5432</code> trong Docker? Hai service cùng đòi 1 cổng thì sao?</b></summary>

**📌 Câu trả lời mẫu:**

"Trong cấu hình Docker, dạng `5433:5432` có nghĩa là `HOST:CONTAINER`: số bên trái là cổng trên máy thật của mình, số bên phải là cổng bên trong container. Postgres trong container luôn lắng nghe ở 5432, nhưng mình ánh xạ nó ra cổng 5433 trên máy host. Lý do mình phải làm vậy là vì máy mình đã có một bản PostgreSQL cài native đang chiếm sẵn cổng 5432 rồi; nếu container cũng đòi bind 5432 trên host thì sẽ xung đột và một bên không khởi động được. Nói chung là hai service không thể cùng chiếm một cổng host. Cách xử lý là đổi cổng host sang số khác như 5433, hoặc tắt cái service đang giữ 5432 đi. Lưu ý là chỉ phần host đổi, còn connection string trỏ vào container thì vẫn dùng 5432 nếu kết nối nội bộ giữa các container."

---

- Định dạng **`HOST:CONTAINER`**: trái là cổng trên máy host, phải là cổng trong container.
- Hai service cùng giành 1 cổng host → **xung đột**, một bên không bind được.
- Cách xử lý: đổi cổng host (vd `5433:5432`) hoặc tắt service kia.

> Ví dụ: máy có PostgreSQL native chiếm 5432 → container map ra `5433` để khỏi đụng.

</details>

<details>
<summary><b>8.2 Làm sao biết tiến trình nào đang chiếm một cổng?</b></summary>

**📌 Câu trả lời mẫu:**

"Khi gặp lỗi cổng bị chiếm, việc đầu tiên mình làm là tìm xem tiến trình nào đang giữ cổng đó. Trên Windows mình hay dùng `netstat -ano | findstr :5432` để lấy PID, hoặc lệnh PowerShell `Get-NetTCPConnection -LocalPort 5432`, rồi từ PID tra ngược ra tên process trong Task Manager. Trên Linux hay Mac thì dùng `lsof -i :5432`. Sau khi biết được thủ phạm là gì, mình mới quyết định: nếu đó là một Postgres native đang chạy ngầm thì mình tắt nó hoặc đổi cổng cho container, còn nếu là process rác thì mình kill nó. Đây là bước cô lập rất quan trọng trước khi sửa cấu hình, tránh việc đoán mò."

---

- Windows: `Get-NetTCPConnection -LocalPort 5432` hoặc `netstat -ano | findstr :5432` → lấy PID → tra process.
- Linux/Mac: `lsof -i :5432`.

</details>

<details>
<summary><b>8.3 Dữ liệu trong Docker nằm ở đâu? <code>down</code> vs <code>down -v</code>?</b></summary>

**📌 Câu trả lời mẫu:**

"Dữ liệu của Postgres trong Docker không nằm trong container mà nằm trong một named volume mà mình khai báo ở compose, nhờ vậy khi mình xoá container rồi tạo lại thì dữ liệu vẫn còn nguyên. Khác biệt quan trọng giữa hai lệnh là: `docker compose down` chỉ xoá container và network nhưng giữ lại volume, nên dữ liệu vẫn an toàn; còn `docker compose down -v` thì xoá luôn cả volume, tức là mất sạch dữ liệu. Mình rất cẩn thận với cái flag `-v` này, chỉ dùng khi thật sự muốn reset DB về trạng thái trắng, vì nếu lỡ tay thì toàn bộ dữ liệu trong DB sẽ bay hết. Khi dev mình hay dùng `down` bình thường để giữ state, chỉ thỉnh thoảng mới `-v` để làm sạch."

---

- Dữ liệu nằm trong **named volume** (xoá/tạo lại container vẫn giữ state).
- `docker compose down`: xoá container + network, **giữ volume** (dữ liệu còn).
- `docker compose down -v`: xoá **cả volume** → **mất sạch dữ liệu**.
- Cẩn thận `-v` và tránh xoá nhầm volume của project khác.

</details>

<details>
<summary><b>8.4 Cách đọc lỗi để khoanh vùng (debug mindset)?</b></summary>

**📌 Câu trả lời mẫu:**

"Cách mình debug là đọc kỹ thông báo lỗi để khoanh vùng nguyên nhân, vì mỗi loại lỗi nói cho mình biết vấn đề nằm ở tầng nào. Ví dụ `ECONNREFUSED` nghĩa là mình không tới được server, thường là sai host, sai cổng hoặc DB chưa chạy, tức vấn đề ở tầng kết nối mạng. Còn `password authentication failed` (mã 28P01) thì ngược lại, mình đã tới được DB rồi nhưng sai user hoặc mật khẩu, tức vấn đề ở tầng credential. Tư duy chính của mình là cô lập từng biến số một: thử kết nối thẳng vào DB, in ra config đang đọc được, kiểm tra biến môi trường, rồi từ đó loại trừ dần đến khi tìm ra đúng chỗ sai. Mình thấy nhà tuyển dụng quan tâm cách suy luận có hệ thống này hơn là việc mình thuộc lòng từng mã lỗi."

---

- `ECONNREFUSED` = không tới được server (mạng/cổng).
- `password authentication failed (28P01)` = tới được nhưng sai thông tin đăng nhập.
- **Cô lập biến số** (test trực tiếp DB, parse config, check env) rồi suy ra phần còn lại.
- Đây là tư duy debug mà nhà tuyển dụng Mid/Senior muốn thấy.

</details>

---

## 9. TypeORM Relations

<details>
<summary><b>9.1 One-to-Many / Many-to-One: khóa ngoại (FK) nằm ở bảng nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Trong quan hệ One-to-Many / Many-to-One, khoá ngoại luôn nằm ở phía 'nhiều' (Many). Lý do rất tự nhiên: một bản ghi cha có thể có nhiều con, nên không thể nhét nhiều id con vào một dòng cha được; ngược lại mỗi con chỉ thuộc về đúng một cha, nên đặt FK ở con là gọn và hợp lý nhất. Ví dụ trong Jira Clone, một Project có nhiều Issue, thì bảng `issues` sẽ giữ cột `projectId` trỏ ngược về `projects.id`, chứ bảng project không chứa danh sách id issue. Nắm được nguyên tắc 'FK ở phía nhiều' giúp mình thiết kế schema đúng ngay từ đầu mà không phải đoán."

---

- FK luôn ở phía **"nhiều" (Many)**.
- Lý do: một cha có nhiều con → không thể nhét nhiều id vào 1 dòng cha; mỗi con chỉ 1 cha → để FK ở con là hợp lý.

Ví dụ: User (1) ─< Post (N) → bảng `posts` giữ cột `authorId` trỏ về `users.id`.

</details>

<details>
<summary><b>9.2 <code>onDelete</code> có những lựa chọn nào?</b></summary>

**📌 Câu trả lời mẫu:**

"`onDelete` quy định chuyện gì xảy ra với các bản ghi con khi bản ghi cha bị xoá, và mình chọn tuỳ theo ý nghĩa nghiệp vụ. `CASCADE` là xoá cha thì xoá luôn con, hợp khi con không có ý nghĩa nếu thiếu cha, ví dụ xoá một issue thì xoá luôn các comment của nó. `SET NULL` là set khoá ngoại của con về null, biến con thành 'mồ côi' nhưng vẫn giữ lại, hợp khi mình muốn giữ dữ liệu con dù mất cha. Còn `RESTRICT` hay `NO ACTION` thì cấm xoá cha chừng nào còn con tham chiếu, dùng khi mình muốn bảo vệ dữ liệu, bắt người dùng xử lý con trước. Mình thường mặc định nghiêng về RESTRICT cho an toàn, chỉ dùng CASCADE ở những quan hệ thật sự là 'con phụ thuộc hoàn toàn vào cha'."

---

- `CASCADE`: xoá cha → xoá kèm con.
- `SET NULL`: set FK về null (con "mồ côi").
- `RESTRICT` / `NO ACTION`: cấm xoá cha nếu còn con tham chiếu.

</details>

<details>
<summary><b>9.3 Eager vs Lazy loading? Vấn đề N+1 là gì? <i>(sẽ học kỹ ở bài truy vấn quan hệ)</i></b></summary>

**📌 Câu trả lời mẫu:**

"Eager loading là tự động kéo luôn dữ liệu quan hệ mỗi khi mình query bản ghi chính, còn lazy loading là chỉ khi nào mình thật sự đụng tới quan hệ đó thì query mới chạy để lấy. Mặc định thì ORM không tự load quan hệ — trong Prisma mình phải khai báo rõ bằng `include` hoặc `select` thì nó mới lấy kèm. Vấn đề N+1 là khi mình lấy ra N bản ghi rồi lặp qua từng cái để query thêm quan hệ, kết quả là 1 query đầu cộng thêm N query con; ví dụ trong Jira Clone mình lấy danh sách issue rồi vòng lặp đi lấy assignee của từng issue, dữ liệu nhiều là chậm thấy rõ. Cách khắc phục là lấy tất cả trong một lần bằng `include` để Prisma tự gom lại, thay vì query lẻ tẻ trong vòng lặp. Đánh đổi là include tiện nhưng nếu lạm dụng sẽ kéo về dư dữ liệu không cần, nên mình chỉ include đúng quan hệ thực sự cần cho response đó thôi."

---

- **Eager**: tự động load quan hệ kèm mỗi query. **Lazy**: chỉ load khi truy cập (trả Promise).
- Mặc định TypeORM không load quan hệ trừ khi `eager: true`, `relations: ['author']` hoặc `leftJoinAndSelect`.
- **N+1**: lấy N bài rồi lặp query tác giả từng bài → 1 + N query → chậm.
- Khắc phục: JOIN (`leftJoinAndSelect`) hoặc `relations` để lấy 1 lần.

</details>

<details>
<summary><b>9.4 (Security) Vì sao <code>authorId</code> phải lấy từ JWT, không từ body client gửi?</b></summary>

**📌 Câu trả lời mẫu:**

"Vì body là thứ client tự gửi nên mình không bao giờ được tin nó cho những dữ liệu định danh chủ sở hữu. Nếu mình cho phép client truyền `authorId` trong body, thì một user hoàn toàn có thể đặt id của người khác vào và tạo bản ghi 'nhân danh' người ta — đó là lỗ hổng mạo danh. Nguyên tắc của mình là danh tính người dùng phải lấy từ token đã được xác thực: sau khi Guard verify JWT, Nest gắn user vào request và mình đọc qua decorator `@CurrentUser()`, từ đó lấy id thật sự của người đang đăng nhập. Vì vậy `authorId` không bao giờ nằm trong DTO; nó được service gán từ token. Trong Jira Clone mình cũng làm y vậy với `reporterId` khi tạo issue — luôn lấy từ JWT chứ không nhận từ phía client."

---

- Chống **mạo danh**: nếu client tự gửi `authorId`, họ có thể tạo bài "nhân danh" người khác.
- Mọi dữ liệu **định danh chủ sở hữu** phải lấy từ **token đã xác thực** (`@CurrentUser()`), không tin body.
- Vì vậy `authorId` **không** nằm trong DTO.

</details>

<details>
<summary><b>9.5 Quan hệ Many-to-Many lưu dữ liệu liên kết ở đâu?</b></summary>

**📌 Câu trả lời mẫu:**

"Quan hệ Many-to-Many không thể lưu trực tiếp khoá ngoại trong hai bảng gốc, vì cả hai phía đều 'nhiều' — một bản ghi bên này nối với nhiều bản ghi bên kia và ngược lại, nhét id vào một cột là không đủ. Cách giải quyết chuẩn là tách ra một bảng nối (join table) chứa hai khoá ngoại trỏ về hai bảng gốc, mỗi dòng trong bảng nối là một cặp liên kết. Ví dụ một issue có nhiều label và một label gắn cho nhiều issue, thì mình có bảng nối kiểu `issue_labels(issueId, labelId)`. Với Prisma thì mình chỉ cần khai báo quan hệ many-to-many trong schema, Prisma tự sinh bảng nối ngầm cho mình; còn khi cần lưu thêm dữ liệu trên chính mối nối (ví dụ thời điểm gắn, ai gắn) thì mình tự tạo một model trung gian rõ ràng (explicit join table) để thêm các cột đó."

---

- Ở một **bảng nối (join table)** chứa 2 khóa ngoại, mỗi dòng là một cặp liên kết.
- Không thể nhét FK vào 2 bảng gốc vì cả hai phía đều "nhiều". TypeORM tự tạo bảng nối.

Ví dụ: `posts_tags(postId, tagId)`.

</details>

---

## 10. Interceptors & Serialization

<details>
<summary><b>10.1 Interceptor là gì? Chạy lúc nào trong request lifecycle?</b></summary>

**📌 Câu trả lời mẫu:**

"Interceptor là thành phần đặc biệt vì nó bọc quanh handler, tức là chạy được cả ở chiều đi vào (trước khi handler chạy) lẫn chiều đi ra (sau khi handler trả kết quả). Nó dùng RxJS để can thiệp vào luồng response, nên mình hay dùng cho mấy việc cross-cutting như logging, đo thời gian xử lý request, bọc lại response theo một format chung, hoặc cache. Về vị trí trong vòng đời thì nó nằm sau Guard và trước Pipe ở chiều vào, rồi quay lại biến đổi response ở chiều ra: `Guard → Interceptor(trước) → Pipe → Handler → Interceptor(sau) → Filter`. Trong Jira Clone mình dùng một interceptor để bọc mọi response thành dạng thống nhất `{ data, ... }` cho client dễ xử lý. Điểm cần phân biệt là Interceptor lo phần 'bao quanh và biến đổi', còn Guard chỉ quyết định cho qua hay không."

---

- Thành phần **bọc quanh handler**, chạy **cả trước và sau** handler.
- Dùng **RxJS** biến đổi luồng response: loại field, bọc dữ liệu, logging, đo thời gian, cache.
- Vị trí: `Guard → Interceptor(trước) → Pipe → Handler → Interceptor(sau) → Filter`.

</details>

<details>
<summary><b>10.2 <code>ClassSerializerInterceptor</code> + <code>@Exclude()</code> làm gì?</b></summary>

**📌 Câu trả lời mẫu:**

"`ClassSerializerInterceptor` là một interceptor dựng sẵn của Nest, nhiệm vụ của nó là biến instance class thành plain object trước khi trả response, và trong lúc biến đổi đó nó tôn trọng các decorator của class-transformer. Cái mình hay dùng nhất là `@Exclude()` gắn lên field nhạy cảm như `password` — khi đó dù object có chứa field này thì lúc serialize nó sẽ tự bị loại khỏi response, mình không phải nhớ xoá tay ở từng chỗ. Ngoài ra có `@Expose()` để hiện hoặc đổi tên field. Cái lợi lớn nhất là nó tập trung việc ẩn dữ liệu nhạy cảm vào một nơi (ngay trên định nghĩa field) thay vì rải rác khắp các service. Một lưu ý quan trọng: nó chỉ chạy đúng khi handler trả về instance của class có gắn decorator; nếu mình trả plain object thường thì decorator không có tác dụng. Trong Jira Clone mình bật global interceptor này để chắc chắn `password` của user không bao giờ lọt ra ngoài."

---

- Interceptor dựng sẵn: tự **chuyển entity instance → plain object** khi trả response.
- Tôn trọng decorator `class-transformer`: `@Exclude` loại field, `@Expose` đổi tên/hiện field.
- Dùng để ẩn dữ liệu nhạy cảm (vd `password`) tập trung cho mọi response.

```ts
@Exclude()
password: string;

// main.ts
app.useGlobalInterceptors(new ClassSerializerInterceptor(app.get(Reflector)));
```

</details>

<details>
<summary><b>10.3 <code>select: false</code> khác <code>@Exclude()</code> thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Hai cái này cùng nhằm giấu một field nhạy cảm nhưng chặn ở hai tầng khác nhau. `select: false` chặn ngay ở tầng truy vấn database: cột đó mặc định không được load lên, muốn lấy phải chủ động select thêm — nghĩa là dữ liệu thậm chí không nằm trong bộ nhớ. Còn `@Exclude()` chặn ở tầng serialize response: dữ liệu vẫn được load vào object trong bộ nhớ, nhưng khi trả ra ngoài thì bị loại bỏ. Mình thường dùng cả hai để bảo vệ hai lớp: ví dụ với `password`, mình cấu hình không select mặc định để service bình thường không thấy nó, nhưng vẫn select khi cần so sánh lúc đăng nhập; đồng thời `@Exclude()` làm lưới an toàn phòng khi field lỡ lọt vào response. Bên Prisma thì tương đương: mình kiểm soát ở tầng query bằng `select`/`omit` để không kéo password lên, và vẫn có thể dùng class-transformer ở tầng serialize. Nói gọn: một cái lo 'không lấy lên', một cái lo 'không trả ra'."

---

- `select: false` (TypeORM): chặn ở tầng **truy vấn DB**, cột không load trừ khi `addSelect`.
- `@Exclude()` (class-transformer): chặn ở tầng **serialize response**, không lộ ra ngoài dù field đang trong bộ nhớ.
- Dùng cả hai = bảo vệ **2 lớp**.

</details>

<details>
<summary><b>10.4 Khi nào <code>@Exclude()</code> KHÔNG có tác dụng?</b></summary>

**📌 Câu trả lời mẫu:**

"`@Exclude()` chỉ có tác dụng khi handler trả về một instance của class có gắn decorator, vì `ClassSerializerInterceptor` cần đúng instance đó mới đọc được metadata để biết field nào phải loại. Nếu mình trả về một plain object — ví dụ `return { ...user }` hay một object literal tự ghép — thì object đó không mang metadata `@Exclude`, interceptor không có gì để dựa vào, nên field nhạy cảm vẫn lọt ra ngoài. Đây là lỗi rất hay gặp: lúc đầu trả entity thì password được ẩn đúng, nhưng sau đó ai đó refactor thành spread object là vô tình rò rỉ. Cách tránh là luôn trả về instance class (ví dụ `plainToInstance` hoặc trả thẳng đối tượng class), hoặc kiểm soát thêm ở tầng query để dữ liệu nhạy cảm không bao giờ được load lên ngay từ đầu — như mình làm với password trong Jira Clone."

---

- Khi handler trả về **plain object** (vd `return { ...user }`) thay vì **instance class entity**.
- `ClassSerializerInterceptor` cần instance để transform; plain object không mang metadata `@Exclude`.

</details>

---

## 11. Pagination, Filtering & QueryBuilder

<details>
<summary><b>11.1 Phân trang trong TypeORM thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Ý tưởng phân trang là không trả về cả nghìn bản ghi một lúc, mà chia thành từng trang có `page` và `limit`. Cách phổ biến nhất là offset pagination: mình bỏ qua `(page - 1) * limit` bản ghi đầu rồi lấy `limit` bản ghi tiếp theo — trong TypeORM là `skip().take()`, còn bên Prisma là `skip` và `take`. Đồng thời mình đếm tổng số bản ghi để tính `totalPages = Math.ceil(total / limit)`, và trả về một response chuẩn dạng `{ data, total, page, limit, totalPages }` để client biết còn bao nhiêu trang. Trong Jira Clone mình dùng cách này cho danh sách issue, và để lấy data với total trong một lượt thì TypeORM có `getManyAndCount()`, còn Prisma mình gọi `findMany` cùng `count` trong một transaction. Lưu ý là nên luôn kèm `orderBy` ổn định (ví dụ theo `createdAt` hoặc `id`) để thứ tự các trang không bị xáo trộn."

---

- Dùng `skip((page-1)*limit).take(limit)` để phân trang.
- `getManyAndCount()` trả `[data, total]`.
- Tính `totalPages = Math.ceil(total/limit)`.
- Response chuẩn: `{ data, total, page, limit, totalPages }`.

```ts
const [data, total] = await qb.skip((page-1)*limit).take(limit).getManyAndCount();
```

</details>

<details>
<summary><b>11.2 Offset pagination vs Cursor pagination?</b></summary>

**📌 Câu trả lời mẫu:**

"Offset pagination là kiểu mình quen nhất: lấy theo `page` và `limit`, bỏ qua một số bản ghi rồi lấy trang tiếp. Ưu điểm là đơn giản và cho phép nhảy tới trang bất kỳ, hợp với bảng dữ liệu có nút trang 1, 2, 3. Nhược điểm là khi offset lớn database vẫn phải quét và bỏ qua toàn bộ bản ghi phía trước nên càng về sau càng chậm; thêm nữa nếu có bản ghi được thêm hoặc xoá giữa lúc người dùng lật trang thì dễ bị lệch, trùng hoặc sót item. Cursor pagination thì thay vì đếm offset, mình dùng một con trỏ là giá trị của bản ghi cuối trang trước (thường là `id` hoặc `createdAt`) rồi lấy các bản ghi sau con trỏ đó. Cách này ổn định kể cả khi dữ liệu thay đổi và nhanh với tập dữ liệu lớn, nhưng đổi lại không nhảy tới trang tuỳ ý được. Nên với feed kiểu cuộn vô tận hoặc danh sách activity real-time thì mình chọn cursor, còn với bảng quản trị cần phân trang số thì offset là đủ và tiện hơn."

---

- **Offset** (`skip/take` theo page): đơn giản, nhảy trang tuỳ ý.
- Nhược offset: chậm khi offset lớn (DB vẫn quét qua), dễ lệch/trùng khi dữ liệu thêm/xoá giữa chừng.
- **Cursor** (dựa `id`/`createdAt` của bản ghi cuối): ổn định, nhanh với dữ liệu lớn/real-time; nhược: không nhảy trang bất kỳ.
- Feed/mạng xã hội thường dùng cursor.

</details>

<details>
<summary><b>11.3 Query param luôn là string — validate kiểu number/boolean thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Vấn đề là query string trên URL luôn là chuỗi, kể cả khi mình gửi `?page=2` thì `page` vẫn là `'2'` chứ không phải số. Nên nếu mình gắn `@IsInt()` thẳng vào thì nó sẽ báo lỗi vì giá trị thực ra là string. Cách mình xử lý là dùng class-transformer ép kiểu trước rồi mới validate: gắn `@Type(() => Number)` lên field và bật `transform: true` trong `ValidationPipe`, lúc đó Nest đổi `'2'` thành `2` rồi `@IsInt()`/`@Min(1)` mới chạy đúng. Với boolean thì query không tự thành true/false được, nên mình thường nhận dạng chuỗi bằng `@IsBooleanString()` (chấp nhận `'true'`/`'false'`) rồi tự so `=== 'true'`, hoặc dùng `@Transform` để đổi sang boolean. Mình hay gom mấy field này vào một DTO query riêng cho endpoint phân trang/lọc trong Jira Clone, vừa gọn vừa đảm bảo dữ liệu vào service đã đúng kiểu."

---

- Dùng `@Type(() => Number)` (class-transformer) + `transform: true` trong ValidationPipe để **ép kiểu** trước khi `@IsInt`/`@Min` kiểm tra.
- Boolean dạng query: `@IsBooleanString()` (nhận `'true'`/`'false'`) rồi tự so `=== 'true'`.

</details>

---

## 12. Exception Filters & Error Handling

<details>
<summary><b>12.1 Exception Filter là gì? Chạy lúc nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Exception Filter là nơi mình bắt mọi exception ném ra trong ứng dụng rồi quyết định response lỗi trả về cho client trông như thế nào. Nó nằm ở cuối vòng đời request và chỉ kích hoạt khi có lỗi, dù lỗi xảy ra ở Guard, Pipe hay trong handler. Lợi ích lớn nhất là mình chuẩn hoá được format lỗi cho toàn app, ví dụ mọi lỗi đều trả về cùng một cấu trúc gồm statusCode, message, timestamp, nên client xử lý lỗi nhất quán. Đồng thời đây cũng là chỗ tốt để log lỗi tập trung thay vì rải try/catch khắp các controller. Trong Jira Clone mình dùng một global filter để bắt hết, lỗi 5xx thì log và báo cảnh báo, còn 4xx thì trả message gọn cho người dùng."

---

- Thành phần **bắt exception** ném ra trong app và định hình response lỗi.
- Nằm ở **cuối** request lifecycle, kích hoạt khi có lỗi.
- Dùng để **chuẩn hoá format lỗi** + **log tập trung**.

</details>

<details>
<summary><b>12.2 Cấu trúc <code>HttpException</code>?</b></summary>

**📌 Câu trả lời mẫu:**

"HttpException là class cơ sở cho các lỗi HTTP trong NestJS, nó mang hai phần chính: status code lấy qua getStatus() và phần nội dung lấy qua getResponse(), mà nội dung này có thể là một string đơn giản hoặc một object dạng { statusCode, message, error }. Mấy exception dựng sẵn như NotFoundException hay BadRequestException đều kế thừa từ nó, nên mình chỉ cần throw đúng loại là Nest tự map ra status tương ứng. Một điểm hay nhầm là khi lỗi validation thì message thường là một mảng các lỗi chứ không phải một chuỗi, nên trong filter mình phải xử lý cả hai trường hợp. Còn với những lỗi không phải HttpException, ví dụ một lỗi lập trình bất ngờ hay lỗi từ Prisma, thì mình nên mặc định coi là 500 và tuyệt đối không để lộ stack trace ra ngoài."

---

- `getStatus()`: mã HTTP; `getResponse()`: string hoặc object `{ statusCode, message, error }`.
- Lỗi validation: `message` là **mảng**.
- Các exception dựng sẵn (`NotFoundException`...) đều kế thừa `HttpException`.
- **Lưu ý:** lỗi không phải HttpException → nên coi là **500**.

</details>

<details>
<summary><b>12.3 Filter toàn cục: <code>useGlobalFilters</code> vs <code>APP_FILTER</code>?</b></summary>

**📌 Câu trả lời mẫu:**

"Cả hai cách đều đăng ký filter ở phạm vi toàn cục, nhưng khác nhau ở chỗ có dùng được Dependency Injection hay không. Nếu mình gọi app.useGlobalFilters(new MyFilter()) trong main.ts thì rất gọn, nhưng vì mình tự new instance nên filter đó không inject được service nào của Nest. Còn nếu mình khai báo filter như một provider với token APP_FILTER trong module, thì Nest sẽ tự tạo và quản lý nó qua DI container, nhờ vậy mình inject được các service khác vào, ví dụ một logger hay client gửi lỗi lên Sentry. Nên quy tắc của mình là khi filter cần phụ thuộc service thì luôn dùng APP_FILTER. Cách này cũng áp dụng tương tự cho APP_GUARD, APP_INTERCEPTOR và APP_PIPE."

---

- `app.useGlobalFilters(new MyFilter())`: đơn giản nhưng **không inject** được dependency.
- Provider **`APP_FILTER`** (trong module): Nest quản lý qua DI → **inject được service** (vd gửi lỗi lên Sentry/logger ngoài).
- Tương tự có `APP_GUARD`, `APP_INTERCEPTOR`, `APP_PIPE`.

</details>

---

## 13. Testing (Jest)

<details>
<summary><b>13.1 Unit vs Integration vs E2E test?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình phân biệt ba loại test theo phạm vi chúng kiểm tra. Unit test kiểm tra một đơn vị nhỏ một cách cô lập, ví dụ một service, và mình mock hết các dependency như Prisma đi, nên nó chạy rất nhanh và ổn định; đây là loại mình viết nhiều nhất. Integration test thì ghép vài thành phần lại với nhau và để một phần là thật, ví dụ test service chạy thật với một database test, để chắc rằng các mảnh ghép phối hợp đúng. Còn E2E test thì kiểm tra cả luồng đi qua HTTP giống như người dùng thật, thường dùng supertest gọi vào app cùng một DB test, để xác nhận từ controller xuống tận DB hoạt động đúng. Mình hình dung theo kim tự tháp test: đáy là nhiều unit test rẻ và nhanh, lên trên ít dần, đỉnh là vài E2E cho các luồng quan trọng vì chúng chậm và tốn công bảo trì hơn."

---

- **Unit**: test 1 đơn vị (vd service) **cô lập**, mock hết dependency → nhanh, ổn định.
- **Integration**: nhiều thành phần ghép, một phần thật (vd service + DB test).
- **E2E**: test cả luồng qua HTTP như user thật (supertest + app + DB test).
- Tháp test: nhiều unit, ít e2e.

</details>

---

## 14. Prisma (ORM)

<details>
<summary><b>14.1 So sánh Prisma vs TypeORM: ưu/nhược điểm, khi nào chọn Prisma?</b></summary>

**📌 Câu trả lời mẫu:**

"Điểm khác biệt cốt lõi là Prisma theo hướng schema-first: mình mô tả toàn bộ model trong file schema.prisma, coi đó là nguồn sự thật duy nhất, rồi Prisma sinh ra một client type-safe. Nhờ vậy lúc viết query mình có autocomplete đầy đủ và kiểu trả về được TypeScript kiểm tra, trải nghiệm dev rất tốt và migration thì rõ ràng dễ theo dõi. Đổi lại Prisma kém linh hoạt hơn với những query rất phức tạp, lúc đó mình phải dùng raw query, và luôn phải nhớ chạy bước prisma generate sau khi đổi schema. TypeORM thì ngược lại, nó dùng decorator gắn trên entity và có QueryBuilder rất mạnh, động hơn, nhưng type safety yếu hơn và dễ bị drift schema, nhất là khi lỡ bật synchronize true. Một lưu ý mình hay nói rõ là Prisma không phải Active Record hay Data Mapper, nó là một query client sinh sẵn. Trong Jira Clone mình chọn Prisma vì ưu tiên type safety và migration dễ kiểm soát; còn nếu dự án cần query cực kỳ động thì TypeORM hợp hơn."

---

- Prisma: schema-first (`schema.prisma` là single source of truth), sinh Prisma Client type-safe → type safety + DX tốt (autocomplete, migration rõ ràng).
- Nhược Prisma: kém linh hoạt với query rất phức tạp (raw query, hoặc `relationJoins` Preview); bắt buộc bước `prisma generate`.
- Lưu ý: Prisma KHÔNG phải Active Record/Data Mapper, nó là query builder/client sinh sẵn. TypeORM mới theo Active Record + Data Mapper (decorator trên entity, QueryBuilder mạnh) nhưng type safety yếu, dễ drift (nhất là `synchronize: true`).
- Chọn Prisma khi ưu tiên type safety ở tầng truy vấn + schema tập trung + migration predictable; chọn TypeORM khi cần entity OOP/QueryBuilder rất động.

```prisma
model Issue {
  id    String @id @default(cuid())
  key   String @unique
  title String
}
```

</details>

<details>
<summary><b>14.2 Phân biệt `prisma migrate dev` và `prisma migrate deploy`?</b></summary>

**📌 Câu trả lời mẫu:**

"Hai lệnh này dùng cho hai môi trường khác nhau. migrate dev là dành cho lúc phát triển: nó so sánh schema.prisma với database, tạo ra file migration mới, áp dụng nó và tự động chạy luôn prisma generate cho mình. Nhưng vì nó hướng tới dev nên khi phát hiện database bị lệch (drift), nó có thể hỏi reset cả DB, tức là xoá sạch dữ liệu, nên mình tuyệt đối không chạy lệnh này trên production. Còn migrate deploy là lệnh dùng trong CI/CD và production: nó chỉ áp dụng đúng những migration đã có sẵn trong thư mục prisma/migrations theo thứ tự, không sinh migration mới, không reset gì cả, và idempotent nên chạy lại nhiều lần vẫn an toàn. Quy trình chuẩn của mình là dev tạo migration bằng migrate dev, commit thư mục migrations vào git, rồi pipeline chỉ việc chạy migrate deploy. Một lưu ý nhỏ là migrate deploy không tự chạy generate nên mình cấu hình chạy riêng, thường là trong bước postinstall."

---

- `migrate dev` (development): so sánh schema với DB, tạo migration mới, áp dụng + tự chạy `prisma generate`.
- `migrate dev` có thể HỎI reset DB (mất data) khi phát hiện drift → tuyệt đối không chạy trên production.
- `migrate deploy` (CI/CD, prod): chỉ áp dụng migration đã có trong `prisma/migrations/` theo thứ tự, không sinh mới, không reset, idempotent. KHÔNG tự chạy `generate` (chạy riêng/postinstall).
- Quy trình chuẩn: dev tạo migration bằng `migrate dev`, commit `migrations/`, pipeline chạy `migrate deploy`.

```bash
npx prisma migrate dev --name add_issue_key   # dev
npx prisma migrate deploy                      # CI/CD, prod
```

</details>

<details>
<summary><b>14.3 `select` và `include` khác nhau thế nào và dùng để tránh over-fetch ra sao?</b></summary>

**📌 Câu trả lời mẫu:**

"Cả hai đều liên quan tới việc lấy dữ liệu gì về, nhưng mục đích khác nhau. include dùng để kéo thêm các relation, ví dụ lấy issue kèm assignee, nhưng nó vẫn trả về tất cả các scalar field của model gốc. Còn select cho mình chọn chính xác từng field cần lấy, và select còn lồng được vào trong relation nữa, nên nó là công cụ chính để tránh over-fetch. Một điểm hay vướng là không được dùng select và include cùng một cấp, Prisma sẽ báo lỗi; muốn vừa chọn field gốc vừa kéo relation thì mình dùng select rồi lồng select bên trong relation. Việc lấy đúng cột cần thiết giúp giảm payload, giảm tải cho DB, và quan trọng là tránh lộ dữ liệu nhạy cảm, ví dụ không bao giờ select passwordHash. Nên với các API trả danh sách, mình luôn select tường minh thay vì trả nguyên cả model ra ngoài."

---

- `include` mở rộng relation nhưng vẫn lấy TẤT CẢ scalar field của model gốc; `select` chọn chính xác field cần (lồng được trong relation) → là công cụ chính tránh over-fetch.
- Không dùng chung `select` và `include` cùng một cấp (Prisma báo lỗi); chỉ lồng bên trong relation.
- Lấy đúng cột giúp giảm payload, giảm tải DB, tránh lộ PII (không bao giờ `select passwordHash`).
- API list nên luôn `select` tường minh thay vì trả nguyên model.

```ts
const issue = await prisma.issue.findUnique({
  where: { key: 'PROJ-12' },
  select: {
    id: true, title: true,
    assignee: { select: { id: true, name: true } },
  },
});
```

</details>

<details>
<summary><b>14.4 Khi nào dùng `$queryRaw` và làm sao tránh SQL injection?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình dùng $queryRaw khi Prisma Client chưa hỗ trợ thứ mình cần, ví dụ window function, CTE phức tạp, full-text search đặc thù, hay khi cần tối ưu hiệu năng theo cách riêng. Điều quan trọng nhất là chống SQL injection, và cách an toàn là dùng tagged template, tức là viết $queryRaw với dấu backtick rồi nhúng biến vào bằng cú pháp dollar-ngoặc; khi đó Prisma tự động tham số hoá mọi giá trị, không nối chuỗi nên rất an toàn. Nếu cần ghép SQL động, mình dùng Prisma.sql cùng Prisma.join, còn Prisma.raw chỉ dành cho phần định danh tin cậy như tên cột. Trái lại, $queryRawUnsafe nhận một chuỗi thường, mình chỉ dùng khi thật sự bắt buộc và tuyệt đối không bao giờ nối thẳng dữ liệu người dùng vào đó. Nhược điểm chung của raw query là kết quả không còn type-safe, mình phải tự khai báo kiểu trả về, và câu SQL cũng mất tính portability giữa các loại database."

---

- Dùng raw khi Prisma Client chưa hỗ trợ: window function, CTE phức tạp, full-text search đặc thù, tối ưu hiệu năng đặc biệt.
- An toàn: dùng tagged template `` $queryRaw`...` `` → mọi giá trị tự động tham số hóa, chống SQL injection. Ghép SQL động an toàn bằng `Prisma.sql` (+ `Prisma.join`, `Prisma.raw` cho identifier tin cậy).
- `$queryRawUnsafe(sql, ...params)` nhận CHUỖI thường + tham số vị trí, không đi kèm `Prisma.sql`; chỉ dùng khi bắt buộc, tuyệt đối không nối chuỗi giá trị người dùng.
- Nhược: kết quả KHÔNG type-safe (khai báo generic kiểu trả về); mất portability giữa các DB.

```ts
const users = await prisma.$queryRaw<
  { id: string; email: string }[]
>`SELECT id, email FROM "User" WHERE email = ${email}`;
```

</details>

---

## 15. Advanced Authentication

<details>
<summary><b>15.1 Phân biệt access token và refresh token? Tại sao cần cả hai thay vì một token sống lâu?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình tách thành hai loại token với hai vai trò khác nhau. Access token thì đoản thọ, ví dụ sống 15 phút, và được gửi kèm theo mỗi request; vì nó stateless nên server chỉ cần verify chữ ký là biết user là ai, không phải query DB. Refresh token thì sống lâu hơn, ví dụ 7 ngày, nhưng nó chỉ có một việc duy nhất là dùng để xin cấp lại access token mới. Lý do mình tách hai cái thay vì dùng một token sống lâu là để giảm thiệt hại khi bị lộ: nếu access token bị đánh cắp thì cửa sổ tấn công rất ngắn, còn refresh token thì được bảo vệ kỹ hơn, đặt trong httpOnly cookie, scope hẹp, và quan trọng là thu hồi được qua DB. Một token sống lâu duy nhất vừa khó thu hồi vừa làm tăng bề mặt tấn công. Một cái bẫy mình hay lưu ý là refresh token nên là một chuỗi random opaque và lưu dạng hash trong DB, chứ không nên là một JWT mang cùng payload như role hay sub."

---

- **Access token**: đoản thọ (ví dụ 15m), gửi kèm mỗi request; stateless nên chỉ verify chữ ký, không query DB.
- **Refresh token**: sống lâu (ví dụ 7d), chỉ dùng để cấp lại access token mới.
- Tách hai loại để giảm thiệt hại khi lộ: access bị đánh cắp thì cửa sổ rất ngắn; refresh được bảo vệ kỹ (httpOnly cookie, scope hẹp, thu hồi được từ DB).
- Một token sống lâu duy nhất vừa khó thu hồi vừa mở rộng bề mặt tấn công.
- **Bẫy**: refresh token nên là **opaque random** (lưu HASH trong DB), KHÔNG nên là JWT mang cùng payload (role, sub).

```ts
const accessToken = this.jwt.sign({ sub: user.id, role: user.globalRole }, { expiresIn: '15m' });
const refreshToken = crypto.randomBytes(32).toString('hex'); // opaque, khong phai JWT
await this.prisma.refreshToken.create({ data: { userId: user.id, tokenHash: await argon2.hash(refreshToken) } });
res.cookie('refresh_token', refreshToken, { httpOnly: true, secure: true, sameSite: 'strict', path: '/auth/refresh' });
```

</details>

<details>
<summary><b>15.2 Lưu token ở httpOnly cookie vs localStorage khác nhau thế nào? Trade-off XSS và CSRF?</b></summary>

**📌 Câu trả lời mẫu:**

"Khác biệt chính nằm ở việc JavaScript có đọc được token hay không. Nếu lưu trong localStorage thì JS đọc được, nên chỉ cần trang dính XSS là attacker có thể trích token ra và đánh cắp. Còn httpOnly cookie thì JS không đọc được, nên nó chống được việc trích token qua XSS; nhưng vì trình duyệt tự động đính kèm cookie vào mọi request tới domain đó, nên nó lại mở ra rủi ro CSRF. Một điểm mình muốn nói rõ là httpOnly không miễn nhiễm XSS đâu: dù không lấy được token ra ngoài, attacker vẫn có thể gọi API thay người dùng vì cookie tự gửi, nên mình vẫn phải phòng XSS từ gốc bằng escape và CSP. Cách làm đúng của mình là dùng httpOnly cộng với secure và sameSite cho cookie, rồi chống CSRF bằng sameSite strict hoặc lax, kiểm tra Origin/Referer, hoặc thêm CSRF token kiểu double-submit. Nói gọn, mỗi cách giải quyết một rủi ro nên phải phòng thủ theo nhiều lớp."

---

- **localStorage**: JS đọc được -> dễ bị đánh cắp token nếu trang dính **XSS**.
- **httpOnly cookie**: JS không đọc được -> chống **trích xuất** token qua XSS, nhưng cookie tự gửi kèm nên mở cửa **CSRF**.
- **Bẫy**: httpOnly KHÔNG miễn nhiễm XSS — attacker vẫn gọi API thay user (cookie tự gửi), chỉ là không trích token ra ngoài. Vẫn phải phòng XSS từ gốc (escape, CSP).
- Cách đúng: httpOnly + `secure` + `sameSite`, rồi chống CSRF bằng sameSite strict/lax, kiểm tra Origin/Referer, hoặc CSRF token (double-submit).

```ts
res.cookie('access_token', token, {
  httpOnly: true, secure: true, sameSite: 'strict', maxAge: 15 * 60 * 1000,
});
```

</details>

<details>
<summary><b>15.3 JWT (stateless) vs session (stateful) khác nhau ra sao? Khi nào chọn cái nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Khác biệt cốt lõi là nơi giữ trạng thái đăng nhập. Với JWT (stateless), toàn bộ thông tin user nằm ngay trong token đã ký, nên mỗi request server chỉ cần verify chữ ký mà không phải hỏi database hay store nào cả; nhờ vậy nó scale ngang rất tốt và hợp với microservice vì service nào cũng tự verify được. Đổi lại điểm yếu lớn nhất là khó thu hồi token trước khi nó hết hạn, vì bản thân token đã đủ hợp lệ rồi. Còn session (stateful) thì server lưu phiên trong Redis hoặc DB, client chỉ cầm một session id; ưu điểm là muốn logout hay khóa user là vô hiệu tức thì, nhưng nhược là mỗi request đều phải lookup store. Trong thực tế mình thường dùng kiểu lai như ở Jira Clone: access token JWT đoản hạn để nhẹ và nhanh, cộng thêm refresh token có lưu state trong DB để vẫn thu hồi được. Một lưu ý mình hay nhắc là verify JWT không hề miễn phí, nó vẫn tốn CPU (RSA/EC còn nặng hơn HMAC), điểm lợi thật sự là không phải đi I/O."

---

- **JWT stateless**: user info nằm trong token, server chỉ verify chữ ký (HMAC/RSA/EC), không query store -> dễ scale ngang, hợp microservice. Nhược: **khó thu hồi trước hạn**.
- **Session stateful**: server lưu session (Redis/DB), client giữ session id; dễ thu hồi tức thì nhưng mỗi request phải lookup store.
- Thực tế thường **lai**: access token JWT đoản hạn + refresh token có state trong DB để thu hồi được.
- **Bẫy**: `jwt.verify` vẫn tốn **CPU** (RSA/EC đắt hơn HMAC); điểm khác chính là **không I/O**, chứ không phải miễn phí.

```ts
const payload = jwt.verify(token, PUBLIC_KEY);        // khong I/O, chi ton CPU
const session = await redis.get(`sess:${sessionId}`); // session: lookup moi request
```

</details>

<details>
<summary><b>15.4 Mô tả flow OTP / email verification an toàn? Cần lưu OTP thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Flow OTP an toàn của mình gồm vài bước. Đầu tiên sinh mã ngẫu nhiên bằng nguồn an toàn như `crypto.randomInt` (ví dụ 6 chữ số), rồi điểm mấu chốt là KHÔNG bao giờ lưu OTP dạng plaintext mà lưu bản hash kèm thời điểm hết hạn (thường 5-10 phút) vào DB, y như cách mình lưu mật khẩu. Khi user nhập mã, mình hash lại để so sánh, kiểm tra mã chưa hết hạn và chưa từng dùng; OTP phải là single-use, verify xong thì đánh dấu đã dùng và đánh dấu email đã xác thực luôn. Quan trọng không kém là phải rate limit và giới hạn số lần thử, vì không gian 6 số chỉ có 1 triệu khả năng nên brute-force rất nhanh nếu không chặn. Một cái bẫy nhỏ mình hay để ý là `crypto.randomInt(min, max)` có `max` exclusive, nên muốn gồm cả 999999 thì phải truyền `randomInt(100000, 1000000)`."

---

- Sinh OTP ngẫu nhiên an toàn (`crypto.randomInt`, ví dụ 6 chữ số) -> lưu **HASH + expiresAt** (5-10 phút) vào DB, KHÔNG lưu plaintext.
- Verify: hash lại và so sánh, kiểm tra chưa hết hạn và chưa dùng; OTP **single-use**, verify xong đánh dấu `emailVerified` và invalidate.
- **Bắt buộc rate limit / giới hạn số lần thử**: không gian 6 số chỉ 1 triệu khả năng, rất dễ brute force.
- **Bẫy**: `crypto.randomInt(min, max)` có `max` **exclusive** -> để gồm 999999 dùng `randomInt(100000, 1000000)`. HMAC-SHA256 đủ an toàn cho giá trị thọ sống ngắn; mấu chốt là không lưu plaintext + có rate limit.

```ts
const otp = crypto.randomInt(100000, 1000000).toString();
await this.prisma.otp.create({ data: { email, codeHash: await argon2.hash(otp), expiresAt: new Date(Date.now() + 5 * 60_000) } });
// verify: lay OTP moi nhat, check expiresAt, argon2.verify, danh dau usedAt
```

</details>

<details>
<summary><b>15.5 Phân biệt JWT, OAuth2 và Session/Cookie — chúng khác tầng nhau thế nào và khi nào dùng cái nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Điều mình muốn làm rõ trước là ba cái này không cùng một tầng nên không phải lựa chọn loại trừ nhau. Session/Cookie và JWT là hai cách **lưu trạng thái đăng nhập**: Session là stateful, server giữ session trong store (Redis/DB) còn client chỉ cầm một session id, nên thu hồi tức thì rất dễ nhưng mỗi request phải lookup store; JWT thì stateless, thông tin nằm trong token đã ký, server chỉ verify chữ ký không cần I/O nên scale ngang tốt, đổi lại **khó thu hồi trước hạn** vì token tự nó đã đủ hợp lệ. Còn OAuth2 thì khác hẳn — nó là một **framework ủy quyền** để app của mình xin quyền truy cập tài nguyên thay người dùng, hoặc đăng nhập qua bên thứ ba (Google, GitHub) qua OIDC; bản thân OAuth2 không nói mình phải dùng session hay JWT để giữ phiên sau đó. Trong Jira Clone của mình, sau khi user 'Login with Google' (OAuth2/OIDC) thì backend NestJS vẫn tự cấp cặp access token JWT đoản hạn + refresh token có state trong Postgres — tức là OAuth2 lo bước xác thực ban đầu, còn JWT/Session lo việc duy trì phiên. Quy tắc chọn của mình: cần thu hồi tức thì và đơn server thì Session/Redis gọn; cần scale ngang nhiều service thì JWT đoản hạn; cần đăng nhập/ủy quyền qua bên thứ ba thì OAuth2 — và thực tế thường lai cả ba."

---

- **Session/Cookie (stateful)**: server giữ state trong store (Redis/DB), client cầm session id -> thu hồi tức thì dễ, nhưng mỗi request phải lookup store; khó scale nếu store là điểm nghẽn.
- **JWT (stateless)**: state nằm trong token đã ký, server chỉ verify chữ ký, không I/O -> scale ngang tốt, hợp microservice; nhược: **khó revoke trước hạn**.
- **OAuth2 KHÁC TẦNG**: là framework **ủy quyền** (delegated authorization); thêm OIDC mới thành đăng nhập. Nó quyết định *làm sao có được danh tính/quyền*, KHÔNG quyết định *giữ phiên bằng gì sau đó*.
- Sau OAuth2 vẫn phải chọn Session hay JWT để duy trì phiên -> ba cái **bổ sung**, không loại trừ.
- **Đánh đổi thu hồi vs scale**: Session = revoke dễ, scale nặng store; JWT = scale nhẹ, revoke khó (phải dùng refresh token có state hoặc blocklist `jti` -> mất bớt tính stateless).
- **Khi nào dùng**: đơn server + cần logout tức thì -> Session/Redis; nhiều service stateless -> JWT đoản hạn; login/ủy quyền qua bên thứ ba -> OAuth2 (+ thường lai JWT/refresh).
- **Bẫy phỏng vấn**: nói "JWT vs OAuth2" như hai lựa chọn là sai — OAuth2 có thể *cấp* token (access token thường là JWT) chứ không cạnh tranh với JWT.

```
OAuth2 / OIDC  ->  (Google trả về danh tính)  ->  Backend Jira Clone tự cấp phiên
  [tang xac thuc/uy quyen]                          |
                                                     +-- JWT access (15m, stateless, verify chu ky)
                                                     +-- refresh token (state trong Postgres -> revoke duoc)
```

</details>

---

## 16. Web Security

<details>
<summary><b>16.1 Helmet làm gì và những security header nào quan trọng nhất?</b></summary>

**📌 Câu trả lời mẫu:**

"Helmet là một middleware giúp mình set sẵn một loạt HTTP security header để giảm các tấn công phổ biến; mình coi nó là một lớp defense-in-depth chứ không thay thế cho việc validate input hay encode output. Mấy header quan trọng nhất theo mình là: `Content-Security-Policy` whitelist nguồn được phép tải script/style để chống XSS, và directive `frame-ancestors` của nó còn chống clickjacking thay cho `X-Frame-Options` cũ. Tiếp theo là `Strict-Transport-Security` (HSTS) để ép trình duyệt luôn dùng HTTPS, chống SSL stripping. Rồi `X-Content-Type-Options: nosniff` để chặn trình duyệt đoán sai kiểu nội dung. Một điểm hay bị hiểu nhầm là Helmet GỠ BỎ header `X-Powered-By` để giảm lộ thông tin công nghệ, chứ không phải set một giá trị giả để che. Trong Nest mình chỉ cần `app.use(helmet())` để bật mặc định, nhưng nếu có serve assets hay Swagger thì phải tinh chỉnh lại CSP cho khỏi chặn nhầm."

---

- Helmet là middleware set một loạt HTTP security header để giảm tấn công phổ biến; là defense-in-depth, không thay validation/output encoding.
- `Content-Security-Policy`: whitelist nguồn script/style/connect chống XSS; directive `frame-ancestors` chống clickjacking (dần thay `X-Frame-Options`).
- `Strict-Transport-Security` (HSTS): ép HTTPS, chống SSL stripping (chỉ tác dụng trên HTTPS, do trình duyệt thực thi).
- `X-Content-Type-Options: nosniff`: chặn MIME sniffing.
- Bẫy: Helmet **gỡ bỏ** `X-Powered-By` (giảm fingerprinting) chứ không set giá trị che giấu.

```ts
app.use(helmet()); // bật default; cần tinh chỉnh CSP nếu serve assets/Swagger
```

</details>

<details>
<summary><b>16.2 Rate limiting/throttling với @nestjs/throttler hoạt động ra sao, và vì sao auth endpoint phải strict hơn?</b></summary>

**📌 Câu trả lời mẫu:**

"Throttler giới hạn số request trong một cửa sổ thời gian theo một key, mặc định là IP; mục đích chính là chống brute-force, credential stuffing và một phần DoS ở tầng ứng dụng. Trong NestJS mình cấu hình `ThrottlerModule.forRoot([...])` rồi đăng ký `ThrottlerGuard` làm `APP_GUARD` cho toàn app, và có thể override riêng từng route bằng `@Throttle`. Lý do auth endpoint như login, OTP hay forgot-password phải siết chặt hơn là vì đó chính là chỗ attacker ngồi đoán mật khẩu hoặc mã OTP; mình thường để cỡ 5 request mỗi phút để mỗi lần đoán trở nên rất tốn kém. Hai điều cần lưu ý: từ v5 trở đi `ttl` tính bằng mili-giây nên `60_000` mới là 60 giây; và khi chạy nhiều instance thì bộ đếm in-memory của mỗi pod là riêng rẽ, nên muốn chính xác phải dùng storage chia sẻ như Redis, đồng thời nên khóa theo cả account chứ không chỉ IP để tránh attacker xoay vòng IP né."

---

- Throttler giới hạn số request trong cửa sổ thời gian theo key (mặc định IP) → chống brute-force, credential stuffing, một phần DoS tầng app.
- Cấu hình: `ThrottlerModule.forRoot([...])` + `ThrottlerGuard` làm `APP_GUARD`, override per-route bằng `@Throttle`.
- Auth endpoint (login, OTP, forgot-password) strict hơn vì là nơi attacker đoán mật khẩu/OTP; siết ~5 req/phút làm brute-force tốn kém.
- Bẫy: từ v5+, `ttl` tính bằng **mili-giây** (`60_000` = 60s); có helper `seconds()`/`minutes()`.
- Đa instance: in-memory mỗi pod đếm rời rạc → phải dùng storage chia sẻ (Redis). Nên khóa theo cả account, không chỉ IP (tránh né bằng IP xoay vòng).

```ts
@Throttle({ default: { limit: 5, ttl: 60_000 } }) // login: 5 lần / 60 giây
@Post('login') login(@Body() dto: LoginDto) {}
```

</details>

<details>
<summary><b>16.3 CORS là gì, cấu hình thế nào cho an toàn khi dùng cookie auth?</b></summary>

**📌 Câu trả lời mẫu:**

"CORS là cơ chế cho phép một origin (ví dụ frontend) gọi tài nguyên ở origin khác (backend). Bản chất là server KHAI BÁO những origin nào được phép, còn TRÌNH DUYỆT mới là bên thực thi việc chặn hay cho đọc response; vì vậy curl hay Postman không bị CORS ràng buộc. Mình hay nhấn mạnh CORS không phải là một lớp authorization: nó chỉ kiểm soát trình duyệt nào được đọc response cross-origin chứ không thay cho việc check quyền ở server. Khi auth dựa trên httpOnly cookie, mình phải bật `credentials: true` để cookie được gửi kèm cross-origin, và tuyệt đối không được dùng `origin: '*'` vì spec cấm kết hợp wildcard với credentials; thay vào đó mình whitelist đúng origin của frontend, như trong Jira Clone là domain của app. Một cái bẫy là đừng phản chiếu (reflect) origin từ request một cách vô điều kiện, vì như vậy chẳng khác gì mở cho mọi website."

---

- CORS cho phép một origin gọi tài nguyên ở origin khác: server **khai báo** origin được phép, **trình duyệt thực thi/chặn** (curl/Postman không bị ràng buộc).
- Không phải lớp authorization: chỉ kiểm soát trình duyệt nào đọc được response cross-origin, KHÔNG thay auth phía server.
- Với httpOnly cookie auth: bật `credentials: true`, **cấm** `origin: '*'` (spec cấm wildcard + credentials) → echo đúng origin trong whitelist.
- Bẫy: tránh phản chiếu (reflect) origin từ request vô điều kiện vì tương đương mở cho mọi site.

```ts
app.enableCors({
  origin: ['https://app.jira-clone.com'], // whitelist cụ thể
  credentials: true, // gửi/nhận cookie cross-origin
});
```

</details>

<details>
<summary><b>16.4 CSRF là gì, khi nào dễ bị, và vì sao dùng cookie lại làm lộ rủi ro này?</b></summary>

**📌 Câu trả lời mẫu:**

"CSRF lợi dụng đúng một hành vi của trình duyệt: nó TỰ ĐỘNG đính kèm cookie vào mọi request gửi tới một domain, kể cả request do một website độc hại kích hoạt. Nhờ vậy site độc hại có thể gửi một POST/PUT/DELETE tới API của mình và backend tưởng đó là user thật vì cookie hợp lệ vẫn được gắn theo. Đây chính là lý do dùng cookie để auth lại lộ rủi ro này, còn nếu mình gửi token qua header `Authorization` thì gần như miễn nhiễm, bởi JavaScript phải chủ động gắn token vào, trình duyệt không tự thêm hộ. Để phòng chống mình kết hợp thuộc tính `SameSite` cho cookie, CSRF token (synchronizer hoặc double-submit), và kiểm tra Origin/Referer. Về `SameSite`: `Strict` chặn mọi request cross-site; `Lax` (mặc định) chặn POST và iframe cross-site nhưng vẫn cho top-level GET nên mình không được dùng GET để đổi state; còn `None` thì bắt buộc đi kèm `Secure` và vẫn cần CSRF token."

---

- CSRF lợi dụng việc trình duyệt **tự đính kèm cookie** vào mọi request tới domain → site độc hại kích hoạt mutation (POST/PUT/DELETE) dưới danh tính nạn nhân.
- Rủi ro khi auth dựa trên cookie ambient; token qua header `Authorization` gần miễn nhiễm (JS phải chủ động gắn, trình duyệt không tự thêm).
- Phòng chống: `SameSite` + CSRF token (synchronizer/double-submit) + kiểm Origin/Referer.
- `SameSite`: `Strict` chặn mọi cross-site; `Lax` (mặc định) chặn POST/iframe/img cross-site nhưng vẫn gửi với top-level GET → đừng dùng GET đổi state; `None` bắt buộc `Secure` + cần CSRF token.

```ts
res.cookie('access_token', token, {
  httpOnly: true, secure: true,
  sameSite: 'lax', // chặn POST cross-site tự động
});
```

</details>

<details>
<summary><b>16.5 Phân biệt input validation và sanitization; làm sao chống XSS và mass assignment?</b></summary>

**📌 Câu trả lời mẫu:**

"Hai khái niệm này bổ sung cho nhau chứ không thay nhau. Validation là từ chối dữ liệu sai định dạng hoặc sai ràng buộc (kiểu, độ dài, enum) ngay từ đầu vào; còn sanitization là làm sạch hoặc escape dữ liệu trước khi lưu hay render. Về mass assignment, mình dựa vào `ValidationPipe` với `whitelist: true` để tự loại bỏ field thừa và `forbidNonWhitelisted: true` để báo lỗi 400 khi có field lạ, nhờ vậy client không thể tự gắn thêm những field nhạy cảm như `role: 'ADMIN'` hay `authorId` vào body. Với XSS, mình lưu ý là một API trả JSON thuần thì rủi ro chủ yếu là stored XSS — dữ liệu bẩn được lưu vào DB rồi sau đó client render ra; nên chỗ phòng đúng tầng là output encoding ở nơi render, ví dụ React tự escape, còn nếu phải dùng `dangerouslySetInnerHTML` thì mình sanitize bằng DOMPurify. Tư duy chung của mình là phòng thủ theo từng tầng: chống mass assignment ở server, còn chống XSS chủ yếu ở chỗ render."

---

- Validation: từ chối dữ liệu sai định dạng/ràng buộc (kiểu, độ dài, enum). Sanitization: làm sạch/escape trước khi lưu/render. Bổ sung nhau, không thay nhau.
- Mass assignment: `ValidationPipe` với `whitelist: true` (bỏ field thừa) + `forbidNonWhitelisted: true` (400 nếu có field lạ) chặn client gắn `role: 'ADMIN'`, `authorId`...
- XSS với API JSON thuần chủ yếu là **stored XSS** (data bẩn lưu DB rồi client render). Phòng đúng tầng là **output encoding ở nơi render** (React tự escape; `dangerouslySetInnerHTML` thì sanitize bằng DOMPurify).
- Phòng thủ theo tầng: chống mass assignment ở server, chống XSS chủ yếu ở chỗ render.

```ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true, forbidNonWhitelisted: true, transform: true,
}));
```

</details>

<details>
<summary><b>16.6 Quản lý secret/env an toàn và masking PII trong log như thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Về secret như DB URL, JWT secret hay API key, nguyên tắc đầu tiên của mình là không commit vào git: để trong `.env` đã được `.gitignore`, và mình validate luôn lúc khởi động bằng một schema (Joi hoặc zod) để app fail-fast nếu thiếu biến quan trọng, tránh chạy được rồi mới lỗi giữa chừng. Ở production thì mình không dùng file phẳng mà inject qua secret manager hoặc biến môi trường của hạ tầng như Vault hay AWS Secrets Manager. Một điều quan trọng là nếu secret lỡ bị lộ thì phải ROTATE (đổi mới) chứ không chỉ xóa khỏi git history, vì lịch sử commit hay bản clone cũ vẫn có thể còn. Về log, mình mask PII và credential như email, token, password, OTP, cookie ở một chỗ tập trung trong logger hoặc interceptor, và phải xử lý cả object lồng nhau bằng đệ quy chứ không chỉ field cấp một. Còn Sentry thì mình chỉ bật ở prod và cấu hình scrubbing để nó không vô tình thu thập dữ liệu nhạy cảm."

---

- Secret (DB URL, JWT secret, API key) không commit: để trong `.env` đã `.gitignore`; validate khi khởi động bằng schema (Joi/zod) để fail-fast.
- Production inject qua secret manager/biến môi trường hạ tầng (Vault, AWS Secrets Manager), không file phẳng.
- Bẫy: secret lỡ lộ phải **rotate**, không chỉ xóa khỏi git history.
- Log: mask PII/credential (email, token, password, OTP, cookie) tập trung trong logger/interceptor, xử lý cả **nested object** (đệ quy). Sentry chỉ bật ở prod + cấu hình scrubbing.

```ts
const SECRET = ['password', 'token', 'otp', 'authorization', 'cookie'];
function sanitize(obj) {
  return Object.fromEntries(Object.entries(obj).map(([k, v]) =>
    SECRET.includes(k.toLowerCase()) ? [k, '***']
    : v && typeof v === 'object' ? [k, sanitize(v)] : [k, v]));
}
```

</details>

<details>
<summary><b>16.7 Kể vài lỗ hổng OWASP Top 10 phổ biến và cách bạn phòng chống trong NestJS?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình sẽ kể vài lỗ hổng hay gặp nhất theo OWASP Top 10 và cách mình xử lý trong NestJS. Đầu tiên là Broken Access Control (A01) — đây là cái mình lo nhất: mình dùng JwtAuthGuard kết hợp RolesGuard để kiểm quyền ở server, nhưng quan trọng hơn là check object-level (chống IDOR), ví dụ trong Jira Clone thì user chỉ được xem/sửa issue thuộc project mà họ là thành viên, chứ không chỉ dựa vào role chung. Với Injection (A03) mình dùng Prisma nên query đã được parameterized sẵn, cộng thêm class-validator để validate input đầu vào. Về Auth Failures (A07) mình hash password bằng bcrypt/argon2 chứ không bao giờ lưu plaintext hay MD5, có rate limit cho login và refresh token rotation. Còn Security Misconfiguration (A05) thì mình bật helmet, tắt stack trace chi tiết ở prod, và không để Swagger public trên production. Nguyên tắc xuyên suốt của mình là least privilege và defense-in-depth — không tin một lớp bảo vệ duy nhất, ví dụ vừa check quyền ở guard vừa check lại ownership trong service."

---

- **A01 Broken Access Control**: RBAC + guard, kiểm quyền ở server, cả object-level (IDOR — user chỉ truy cập resource của mình).
- **A03 Injection**: parameterized query/Prisma + validation.
- **A07 Auth Failures**: hash bằng bcrypt/argon2 (không plaintext/MD5), rate limit login, refresh token rotation, khóa account sau nhiều lần sai.
- **A05 Security Misconfiguration**: helmet, tắt verbose error/stack trace ở prod, không bật Swagger/debug ở prod.
- **A02 Cryptographic Failures**: HTTPS/TLS, không log secret, thuật toán mạnh. **A06**: theo dõi dependency.
- Nguyên tắc: least privilege + defense-in-depth.

```ts
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles('PROJECT_LEAD', 'PROJECT_ADMIN')
@Delete('projects/:id') remove(@Param('id') id: string) {}
```

</details>

---

## 17. Observability and Logging

<details>
<summary><b>17.1 Tại sao nên dùng structured logging (JSON) thay vì log dạng text thuần? Lợi ích là gì?</b></summary>

**📌 Câu trả lời mẫu:**

"Structured logging nghĩa là mỗi dòng log là một object JSON với các field cố định như timestamp, level, message, requestId, userId, thay vì một chuỗi text tự do. Lợi ích lớn nhất là log trở nên parse và index được bởi các hệ thống như Elasticsearch, Loki hay Datadog, nên mình có thể query, filter và aggregate theo field — ví dụ lọc tất cả log của một `requestId` để lần theo một request, hay đếm lỗi theo `userId`. Còn text thuần thì rất khó parse và dễ vỡ mỗi khi mình đổi format chuỗi. Một cái bẫy riêng của NestJS mình hay nhắc là `logger.log(message, context)` có tham số thứ hai là *context* dạng string chứ không phải một object metadata, nên muốn structured thật sự thì mình dùng Pino (qua `nestjs-pino`) hoặc Winston với JSON formatter. Và ở production mình tắt pretty-print, xuất log một dòng JSON cho hệ thống thu thập dễ ingest, còn lúc dev thì mới bật pretty cho dễ đọc."

---

- Structured logging: mỗi log là một JSON object với field cố định (timestamp, level, message, requestId...) thay vì chuỗi text tự do.
- Lợi ích: log có thể **parse và index** bởi Elasticsearch, Loki, Datadog -> query/filter/aggregate theo field (lọc theo `requestId`, `userId`). Text thuần khó parse, dễ vỡ khi đổi format.
- Bẫy NestJS: `logger.log(message, context)` — tham số 2 là *context* (string), không phải metadata object. Muốn structured thật sự dùng **Pino** (`nestjs-pino`) hoặc Winston + JSON formatter.
- Production: tắt pretty-print, xuất one-line JSON cho dễ ingest.

```ts
// Voi pino-style logger:
logger.info(
  { userId, issueKey: key, projectId }, // metadata fields
  'Issue created',                       // message
); // pino tu them timestamp & level
```

</details>

<details>
<summary><b>17.2 Phân biệt các log level INFO / WARN / ERROR. Khi nào dùng từng loại?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình phân biệt ba mức này theo mức độ nghiêm trọng và việc cần hành động. INFO là sự kiện bình thường nhưng đáng quan tâm về nghiệp vụ, ví dụ user login, tạo issue, hay một cron job chạy xong. WARN là chuyện bất thường nhưng hệ thống vẫn xử lý được, như sắp chạm ngưỡng rate limit, một lần retry thành công, hay gọi vào một API đã deprecated — nó như một cảnh báo để mình để mắt. ERROR là thất bại thật sự cần người vào xem và xử lý, ví dụ một exception bất ngờ, lỗi 5xx, hay rớt kết nối DB. Một quy tắc mình rất coi trọng là các lỗi 400/401/403/404 KHÔNG nên log thành ERROR, vì đó là lỗi phía client và là chuyện kỳ vọng sẽ xảy ra; nếu log ERROR thì dashboard sẽ đầy nhiễu và gây alert fatigue, nên mình chỉ để WARN hoặc INFO. Cách mình hay làm trong Jira Clone là cho Exception Filter map status >= 500 thành ERROR còn 4xx thành WARN, nhờ vậy dashboard ERROR chỉ chứa lỗi server thật. Ngoài ra còn DEBUG/TRACE cho chi tiết kỹ thuật (tắt ở prod) và FATAL khi process không thể tiếp tục."

---

- **INFO**: sự kiện bình thường, đáng quan tâm nghiệp vụ (user login, tạo issue, cron xong).
- **WARN**: bất thường nhưng vẫn xử lý được (gần chạm rate limit, retry thành công, gọi deprecated API).
- **ERROR**: thất bại thật sự cần hành động (exception bất ngờ, 5xx, rớt kết nối DB).
- Quy tắc quan trọng: **400/401/403/404 KHÔNG phải ERROR** — lỗi client, kỳ vọng xảy ra; chỉ log WARN/INFO để tránh nhiễu và alert fatigue.
- Còn có **DEBUG/TRACE** (chi tiết kỹ thuật, tắt ở prod) và **FATAL** (process không thể tiếp tục).

Ví dụ: Exception Filter map `statusCode >= 500` -> ERROR, 4xx -> WARN, nên dashboard ERROR chỉ chứa lỗi server thật.

</details>

<details>
<summary><b>17.3 Kiến trúc log request: tại sao dùng Interceptor cho success và Exception Filter cho lỗi?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình tách việc log theo đúng vòng đời request cho gọn. Với request chạy thành công thì mình dùng Interceptor, vì Interceptor bọc quanh handler nên rất hợp để đo thời gian xử lý (latency) và log thông tin response khi nó trả về bình thường. Còn khi có lỗi ném ra thì Exception Filter mới là nơi chạy, nên mình để toàn bộ việc log lỗi và format lại error response ở đó cho nhất quán. Lợi ích lớn nhất là mỗi bên lo một luồng riêng, mình không phải rải try/catch khắp các controller nữa. Một cái bẫy mình hay nhắc là trong Interceptor nếu chỉ viết tap(() => ...) thì nó chỉ chạy ở nhánh success; muốn đo latency cả khi lỗi thì phải dùng tap({ next, error }) hoặc finalize, còn không thì cứ để Exception Filter lo phần lỗi."

---

- Phân tách trách nhiệm theo vòng đời request. **Interceptor** bọc quanh handler: đo latency và log khi response trả về bình thường.
- **Exception Filter** chỉ chạy khi có exception: nơi tập trung log lỗi nhất quán và format error response.
- Tách 2 nơi -> mỗi bên lo một luồng, tránh `try/catch` rải rác trong controller.
- Bẫy: `tap` chỉ chạy ở nhánh **next** (success). Muốn đo latency cả khi lỗi dùng `tap({ next, error })` hoặc `finalize`. Chỉ `tap(() => ...)` thì request lỗi không được log latency (Exception Filter lo phần đó).

```ts
intercept(ctx: ExecutionContext, next: CallHandler) {
  const start = Date.now();
  const req = ctx.switchToHttp().getRequest();
  return next.handle().pipe(
    tap({ next: () => this.logger.log({
      method: req.method, url: req.url,
      requestId: req.requestId, durationMs: Date.now() - start,
    })}),
  );
}
```

</details>

<details>
<summary><b>17.4 Tại sao chỉ gửi Sentry cho lỗi 5xx và chỉ ở môi trường production?</b></summary>

**📌 Câu trả lời mẫu:**

"Sentry với mình là công cụ để alert những lỗi server thật sự cần fix, nên mình chỉ gửi lỗi 5xx lên thôi. Mấy lỗi 4xx như validation sai, 401, 404 thực chất là lỗi phía client và xảy ra rất thường xuyên, nếu đẩy hết lên thì chỉ tạo noise, tốn quota và làm loãng những lỗi quan trọng. Mình cũng chỉ bật ở môi trường production, vì khi đang code ở local thì lỗi là chuyện bình thường, báo lên chỉ gây nhiễu và làm sai metric. Cách mình làm thường là Sentry.init với enabled theo NODE_ENV, như vậy captureException ở dev vẫn an toàn vì nó thành no-op chứ không crash. Mình hay set thêm environment để tách dashboard dev/prod, và gắn requestId vào tag để dễ lần ngược lại log của đúng request bị lỗi."

---

- **Chỉ 5xx**: Sentry để alert lỗi server thật cần fix. Đẩy hết 4xx (validation, 401, 404 — lỗi client, xảy ra thường xuyên) -> noise, tốn quota, loãng lỗi quan trọng.
- **Chỉ production**: lỗi local/dev khi đang code là bình thường, báo lên gây nhiễu và sai metric.
- Bật/tắt theo env nên dùng `Sentry.init({ enabled: NODE_ENV === 'production' })` (init nhưng không gửi khi disabled) — an toàn hơn không gọi `init`, vì `captureException` vẫn là no-op an toàn.
- Nên set `environment: NODE_ENV` để tách dashboard. NestJS có `@sentry/nestjs` cung cấp sẵn filter/integration.

```ts
const status =
  exception instanceof HttpException ? exception.getStatus() : 500;
if (status >= 500) {
  Sentry.captureException(exception, { tags: { requestId: req.requestId } });
}
```

</details>

<details>
<summary><b>17.5 Health check endpoint dùng để làm gì và nên kiểm tra những gì?</b></summary>

**📌 Câu trả lời mẫu:**

"Health check endpoint, ví dụ /health, là để những hệ thống bên ngoài như load balancer, Kubernetes hay uptime monitor biết service của mình có đang sẵn sàng phục vụ không. Mình thường tách hai loại: liveness chỉ kiểm tra process còn sống hay không, nếu fail thì orchestrator restart lại pod; còn readiness kiểm tra các dependency thiết yếu như DB, Redis có OK không, nếu fail thì ngừng route traffic vào nhưng không restart. Endpoint này mình để @Public vì nó không cần đăng nhập, và giữ cho nó nhẹ, trả về trạng thái từng thành phần để biết chính xác cái nào hỏng. Bên NestJS thì mình dùng @nestjs/terminus cho tiện. Một bẫy cần lưu ý là đừng vội đưa disk vào readiness, vì disk gần đầy sẽ làm readiness fail và pod bị rút khỏi load balancer, dễ gây cascading failure; nhiều nơi chỉ để disk ở mức cảnh báo riêng."

---

- `/health` cho **load balancer, Kubernetes, uptime monitor** biết service có sẵn sàng phục vụ không.
- Tách **liveness** (process còn sống — fail thì orchestrator restart pod) và **readiness** (dependency thiết yếu: DB, Redis OK không — fail thì ngừng route traffic nhưng không restart).
- NestJS có `@nestjs/terminus`. Endpoint phải `@Public` (không cần auth), nhẹ, trả status từng thành phần để biết cái nào hỏng.
- Bẫy: cẩn thận đưa disk vào readiness — disk gần đầy làm readiness fail -> pod bị rút khỏi LB, gây cascading failure. Nhiều nơi chỉ đưa disk vào liveness/cảnh báo riêng.

```ts
@Public() @Get('health') @HealthCheck()
check() {
  return this.health.check([
    () => this.db.pingCheck('database'),
    () => this.disk.checkStorage('disk', { thresholdPercent: 0.9, path: '/' }),
  ]);
}
```

</details>

---

## 18. Advanced NestJS

<details>
<summary><b>18.1 Mô tả pattern global JwtAuthGuard + @Public(). Làm sao một route opt-out khỏi xác thực?</b></summary>

**📌 Câu trả lời mẫu:**

"Ý tưởng của mình là làm hệ thống 'secure by default', tức là mọi route mặc định đều phải có JWT mới vào được. Mình đăng ký JwtAuthGuard ở mức global qua APP_GUARD, nên không cần nhớ gắn guard cho từng route nữa, mặc nhiên route nào cũng được bảo vệ. Với những route công khai như login hay health check thì mình tạo một decorator @Public() bằng SetMetadata để đánh dấu nó được bỏ qua auth. Bên trong guard, mình dùng Reflector.getAllAndOverride đọc metadata public ở cả handler và class; nếu thấy là public thì canActivate trả true ngay, không verify token, còn không thì gọi super.canActivate() chạy logic JWT bình thường. Cách này an toàn hơn việc đi gắn guard thủ công từng route, vì lỡ quên một route thì nó vẫn được bảo vệ chứ không bị hở."

---

- "Secure by default": đặt `JwtAuthGuard` global qua `APP_GUARD` nên mọi route yêu cầu JWT.
- `@Public()` (tạo bằng `SetMetadata`) đánh dấu route bỏ qua auth.
- Trong guard: `Reflector.getAllAndOverride` đọc key public ở handler + class; nếu public thì `canActivate` trả `true` ngay, không verify token.
- Lưu ý: `super.canActivate()` trả `boolean | Promise | Observable`, nên khai báo đúng kiểu trả về (hoặc dùng `async/await`), không ép về `boolean`.

```ts
export const IS_PUBLIC_KEY = 'isPublic';
export const Public = () => SetMetadata(IS_PUBLIC_KEY, true);

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {
  constructor(private reflector: Reflector) { super(); }
  canActivate(ctx: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
    const isPublic = this.reflector.getAllAndOverride<boolean>(IS_PUBLIC_KEY, [
      ctx.getHandler(), ctx.getClass(),
    ]);
    if (isPublic) return true;
    return super.canActivate(ctx);
  }
}
```

</details>

<details>
<summary><b>18.2 Custom param decorator (@CurrentUser) hoạt động thế nào? Khác với class/method decorator ở chỗ nào?</b></summary>

**📌 Câu trả lời mẫu:**

"@CurrentUser là một custom param decorator mình tạo bằng createParamDecorator, nó chạy lúc Nest resolve tham số cho handler, tức là sau guard. Điểm cốt lõi là nó trả về giá trị trực tiếp để Nest tiêm thẳng vào argument của handler, ví dụ mình viết @CurrentUser() user: User thì lấy luôn req.user mà không phải đụng vào request thô. Đây chính là chỗ khác với class hoặc method decorator như @Roles dùng SetMetadata: mấy loại kia chỉ gắn metadata lên route để Guard hay Interceptor đọc lại qua Reflector chứ bản thân chúng không tự chèn giá trị vào tham số. Param decorator của mình còn nhận thêm tham số data nên mình có thể chọn lấy đúng một trường, ví dụ @CurrentUser('id') để lấy mỗi userId. Trong Jira Clone mình dùng nó để lấy user đang đăng nhập ở hầu hết các handler, code rất gọn và dễ đọc."

---

- Tạo bằng `createParamDecorator((data, ctx) => ...)`, chạy lúc Nest resolve tham số handler (sau guard, cùng giai đoạn pipe).
- Nó **trả về giá trị** trực tiếp vào argument handler, KHÔNG gán metadata.
- Khác class/method decorator (`SetMetadata`): loại kia chỉ gán metadata để Guard/Interceptor đọc qua `Reflector`, không tự chèn giá trị.
- Tham số `data` cho phép chọn trường cụ thể; pipe vẫn áp dụng được lên giá trị trả về.

```ts
export const CurrentUser = createParamDecorator(
  (data: string | undefined, ctx: ExecutionContext) => {
    const req = ctx.switchToHttp().getRequest();
    return data ? req.user?.[data] : req.user;
  },
);
// dung: @CurrentUser() user: User  hoac  @CurrentUser('id') userId: string
```

</details>

<details>
<summary><b>18.3 Phân biệt useClass, useValue, useFactory, useExisting. Khi nào dùng useFactory với inject?</b></summary>

**📌 Câu trả lời mẫu:**

"Đây là bốn cách khai báo một provider trong NestJS, khác nhau ở chỗ Nest lấy giá trị từ đâu. useClass là để Nest tự khởi tạo một class, hay dùng khi mình muốn swap implementation cho cùng một token. useValue là cung cấp sẵn một giá trị hoặc object có sẵn, ví dụ một hằng số config hay một mock khi viết test. useExisting chỉ là alias trỏ tới provider khác để dùng chung một instance singleton chứ không tạo mới. Còn useFactory là chạy một hàm để tạo giá trị động, nó hỗ trợ inject các provider khác và có thể async. Mình dùng useFactory kèm inject khi giá trị cần tính lúc runtime hoặc phụ thuộc provider khác, điển hình là cần ConfigService để đọc DATABASE_URL rồi mới dựng options; chỉ cần nhớ thứ tự trong mảng inject phải khớp đúng thứ tự tham số của factory."

---

- `useClass`: Nest tự khởi tạo class (swap implementation theo token).
- `useValue`: cung cấp giá trị/object có sẵn (mock, hằng số, config).
- `useFactory`: chạy hàm tạo giá trị động, hỗ trợ `inject: []` và có thể `async`.
- `useExisting`: alias trỏ tới provider khác (chung instance singleton, không tạo mới).
- Dùng `useFactory` khi giá trị cần tính runtime, cần async, hoặc phụ thuộc provider khác. Thứ tự `inject` phải khớp thứ tự tham số factory.

```ts
{
  provide: 'PRISMA_OPTIONS',
  useFactory: (config: ConfigService) => ({ url: config.get('DATABASE_URL') }),
  inject: [ConfigService],
}
```

</details>

<details>
<summary><b>18.4 Tại sao nên validate biến môi trường (.env) và làm sao làm trong NestJS?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình luôn validate biến môi trường vì nếu không, một biến bị thiếu hoặc sai như DATABASE_URL hay JWT_SECRET sẽ chỉ lộ ra lúc runtime, thường là sau khi đã deploy, rất khó debug và nguy hiểm. Ví dụ JWT_SECRET rỗng thì app vẫn ký token được nhưng hoàn toàn không an toàn, mà mình lại không hề hay biết. Nên triết lý của mình là 'fail fast': nếu config sai thì app phải báo lỗi ngay lúc khởi động chứ đừng chạy với cấu hình lỗi. Trong NestJS mình làm bằng ConfigModule.forRoot với validationSchema dùng Joi, hoặc dùng validate callback với class-validator. Mình khai báo rõ biến nào required, kiểu gì, giá trị mặc định ra sao; nếu env không khớp schema thì app ném lỗi ngay khi bootstrap, đỡ phải dò lỗi mơ hồ về sau."

---

- Không validate: biến sai/thiếu (`DATABASE_URL`, `JWT_SECRET`) chỉ lộ lúc runtime, thường khi đã deploy — khó debug, nguy hiểm (secret rỗng vẫn ký JWT nhưng không an toàn).
- "Fail fast" lúc bootstrap là chuẩn production.
- Cách làm: `ConfigModule.forRoot({ validationSchema })` với Joi, hoặc `validate` callback với `class-validator`. Schema không khớp → app ném lỗi ngay lúc bootstrap.

```ts
ConfigModule.forRoot({
  isGlobal: true,
  validationSchema: Joi.object({
    NODE_ENV: Joi.string().valid('development', 'production', 'test').default('development'),
    DATABASE_URL: Joi.string().uri().required(),
    JWT_SECRET: Joi.string().min(32).required(),
    JWT_EXPIRES_IN: Joi.string().default('15m'),
  }),
});
```

</details>

---

## 19. Database, Transactions and Concurrency

<details>
<summary><b>19.1 $transaction trong Prisma hoạt động thế nào và nó đảm bảo ACID ra sao?</b></summary>

**📌 Câu trả lời mẫu:**

"$transaction của Prisma về bản chất là gom nhiều query vào một transaction DB — hoặc tất cả commit, hoặc rollback hết nếu có lỗi. Cần nói rõ là ACID được đảm bảo ở tầng PostgreSQL chứ Prisma chỉ làm nhiệm vụ mở BEGIN/COMMIT/ROLLBACK thôi. Có hai dạng: sequential, tức truyền vào một mảng query chạy tuần tự không chèn được logic JS; và interactive, truyền vào một callback async tx => {...} cho phép mình viết logic giữa các bước. Trong Jira Clone mình hay dùng interactive, ví dụ khi tạo issue thì vừa increment counter của project vừa insert issue trong cùng transaction để đảm bảo issue key không bị lệch. Cái bẫy lớn nhất là interactive transaction giữ connection và lock cho tới khi callback chạy xong, nên mình tuyệt đối không đặt I/O chậm như gọi API ngoài hay gửi mail vào trong đó — mấy việc đó mình đẩy ra BullMQ chạy sau khi commit. Mình cũng set maxWait và timeout hợp lý để tránh transaction treo giữ lock quá lâu gây deadlock."

---

- `$transaction` gom nhiều query vào một transaction DB: tất cả commit, hoặc rollback toàn bộ nếu lỗi.
- ACID đảm bảo ở tầng DB (Postgres); Prisma chỉ mở `BEGIN/COMMIT/ROLLBACK`: Atomicity, Consistency, Isolation, Durability.
- Hai dạng: *sequential* `$transaction([q1, q2])` (chạy tuần tự, không chèn JS) và *interactive* `$transaction(async tx => {...})` (cho phép logic giữa các bước).
- Bẫy: interactive giữ connection + lock đến khi callback xong → KHÔNG đặt I/O chậm (gọi API, gửi mail) trong callback, dễ deadlock.
- Tham số: `maxWait` (thời gian chờ lấy connection), `timeout` (thời gian tối đa transaction chạy).

```ts
await prisma.$transaction(async (tx) => {
  const acc = await tx.account.update({
    where: { id }, data: { balance: { decrement: 100 } },
  });
  if (acc.balance < 0) throw new Error('Insufficient funds'); // rollback toan bo
  await tx.ledger.create({ data: { accountId: id, amount: -100 } });
}, { maxWait: 5000, timeout: 10000 });
```

</details>

<details>
<summary><b>19.2 Làm sao sinh issue key tuần tự PROJECT-N không bị trùng khi có nhiều request đồng thời?</b></summary>

**📌 Câu trả lời mẫu:**

"Lỗi kinh điển ở đây là đọc counter ra app rồi cộng 1 rồi ghi lại, kiểu read-modify-write; vì khi hai request chạy đồng thời chúng đọc cùng một giá trị nên sẽ sinh ra key trùng nhau. Cách đúng của mình là để DB tự tăng một cách atomic, trong Prisma là dùng increment: 1, nó sinh ra câu SQL kiểu SET counter = counter + 1. Khi đó DB tự khóa row nên các update bị serialize lại, mỗi request nhận về một giá trị duy nhất, không sợ trùng. Trong Jira Clone mình làm thế để sinh issue key dạng JIRA-42, vừa update counter của project vừa lấy luôn giá trị mới ra để ghép key. Mình cũng đặt thêm một unique constraint trên cặp (projectId, number) làm lớp bảo vệ cuối cùng, lỡ có sai sót gì thì DB vẫn chặn được. Lưu ý là với atomic increment thì không cần SELECT FOR UPDATE, cái đó chỉ cần khi mình đọc rồi mới quyết định ghi qua nhiều bước."

---

- KHÔNG đọc counter ra app rồi +1 rồi ghi lại (read-modify-write) → hai request đọc cùng giá trị, trùng key.
- Đúng: atomic increment tại DB qua `{ increment: 1 }` (sinh `SET counter = counter + 1`); DB tự khóa row → các update bị serialize, mỗi request nhận giá trị duy nhất.
- Thêm unique constraint `(projectId, number)` làm lớp bảo vệ cuối.
- Lưu ý: với atomic `increment` KHÔNG cần `SELECT ... FOR UPDATE`; `FOR UPDATE` chỉ cần khi đọc rồi mới quyết định ghi qua nhiều bước.

```ts
const key = await prisma.$transaction(async (tx) => {
  const p = await tx.project.update({
    where: { id: projectId },
    data: { issueCounter: { increment: 1 } },
    select: { code: true, issueCounter: true },
  });
  return `${p.code}-${p.issueCounter}`; // VD: "JIRA-42"
});
```

</details>

<details>
<summary><b>19.3 Race condition là gì? Nêu các chiến lược chính để xử lý trong backend?</b></summary>

**📌 Câu trả lời mẫu:**

"Race condition là khi kết quả cuối cùng phụ thuộc vào thứ tự và thời điểm của nhiều thao tác chạy đồng thời trên cùng một dữ liệu; điển hình nhất là tình huống read-modify-write, hai request cùng đọc rồi cùng ghi đè lên nhau. Để xử lý mình có vài chiến lược chính. Đơn giản nhất là dùng atomic DB operation như increment/decrement hoặc UPDATE kèm điều kiện WHERE để DB tự lo. Khi tranh chấp cao thì mình dùng pessimistic lock, tức SELECT FOR UPDATE trong transaction để khóa row lại. Khi tranh chấp thấp thì optimistic lock hợp hơn, dùng một cột version rồi retry nếu phát hiện bị ghi đè. Ngoài ra còn có unique constraint làm chốt chặn cuối, và với mấy case phức tạp như write-skew thì nâng isolation lên Serializable. Nói chung mình chọn theo mức tranh chấp: thấp thì optimistic, cao thì pessimistic, còn đơn giản thì cứ atomic là gọn nhất."

---

- Là khi kết quả phụ thuộc thứ tự/timing của nhiều thao tác đồng thời trên cùng dữ liệu, điển hình read-modify-write.
- Chiến lược: (1) Atomic DB op (`increment`/`decrement`, `UPDATE ... WHERE`); (2) Pessimistic lock `SELECT ... FOR UPDATE` (trong transaction); (3) Optimistic lock cột `version` + retry; (4) Unique constraint; (5) Serializable isolation cho case phức tạp (write-skew).
- Chọn theo mức tranh chấp: thấp → optimistic, cao → pessimistic, đơn giản → atomic.

```sql
-- Atomic: chi tru neu con du ton kho, tranh oversell
UPDATE inventory SET qty = qty - 1
WHERE product_id = $1 AND qty > 0;
-- rowCount === 0 => het hang, dat that bai
```

</details>

<details>
<summary><b>19.4 Index là gì? Phân biệt B-tree, composite, unique và partial index?</b></summary>

**📌 Câu trả lời mẫu:**

"Index là cấu trúc dữ liệu phụ giúp DB tìm row nhanh thay vì phải quét cả bảng. Loại mặc định và dùng nhiều nhất là B-tree, hợp cho so sánh bằng, range như lớn hơn/nhỏ hơn/BETWEEN và cả ORDER BY. Composite index là index trên nhiều cột, ví dụ (project_id, created_at), nhưng phải nhớ left-prefix rule: nó dùng được khi mình lọc theo project_id hoặc project_id cộng created_at, còn nếu chỉ lọc mỗi created_at thì gần như vô dụng — nên thứ tự cột rất quan trọng. Unique index thì vừa tăng tốc tìm kiếm vừa enforce tính duy nhất, thực ra Postgres dùng chính unique index để thực thi unique constraint. Partial index chỉ index tập con thỏa điều kiện WHERE nên nhỏ và ghi nhanh hơn. Trong Jira Clone mình tạo composite index (project_id, created_at DESC) để list issue trong một board sắp theo ngày cho nhanh, và dùng partial index WHERE deleted_at IS NULL để chỉ index issue chưa bị soft-delete. Đánh đổi cần nhớ là index làm đọc nhanh nhưng ghi chậm hơn và tốn dung lượng, nên mình chỉ tạo index bám theo query thực tế chứ không tạo bừa."

---

- Index: cấu trúc phụ giúp tìm row nhanh thay vì quét toàn bảng.
- B-tree (mặc định): tối ưu `=`, range `<,>,BETWEEN`, hỗ trợ `ORDER BY`.
- Composite `(a, b)`: theo left-prefix rule — dùng được cho `a` hoặc `a`+`b`, không hiệu quả nếu chỉ lọc `b`; thứ tự cột quan trọng.
- Unique: vừa tăng tốc vừa enforce duy nhất (Postgres dùng unique index để thực thi unique constraint).
- Partial: chỉ index tập con thỏa `WHERE` → nhỏ gọn, ghi nhanh.
- Đánh đổi: index tăng đọc nhưng chậm write + tốn dung lượng → chỉ tạo theo truy vấn thực tế.

```sql
-- Composite: loc theo project roi sort theo ngay tao
CREATE INDEX idx_issue_project_created ON issue (project_id, created_at DESC);
-- Partial: chi index issue chua xoa mem
CREATE INDEX idx_issue_active ON issue (project_id) WHERE deleted_at IS NULL;
```

</details>

<details>
<summary><b>19.5 Vấn đề N+1 query là gì và xử lý trong Prisma như thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"N+1 query là khi mình chạy một query lấy về N bản ghi, rồi lặp qua từng cái và mỗi vòng lại bắn thêm một query để lấy dữ liệu quan hệ, thành ra tổng cộng 1 cộng N query, rất chậm khi N lớn. Ví dụ trong Jira Clone là lấy danh sách issue rồi lặp từng issue để query assignee của nó. Cách xử lý chính trong Prisma là dùng include hoặc select để lấy luôn quan hệ trong một lần gọi, và tuyệt đối tránh gọi DB bên trong vòng lặp. Một điểm hay là Prisma mặc định không JOIN mà chạy thêm một query findMany với WHERE id IN (...) rồi gộp lại ở tầng app, nên thực ra nó chỉ thêm đúng một query chứ không phải N; từ Prisma 5.x mình có thể bật relationLoadMode join nếu muốn JOIN thật. Để phát hiện thì mình bật log query lên xem, nếu thấy một loạt query lặp lại giống nhau là dấu hiệu của N+1."

---

- N+1: 1 query lấy N bản ghi, rồi lặp N lần mỗi lần thêm 1 query lấy quan hệ → 1+N query, rất chậm.
- Giải pháp chính trong Prisma: dùng `include`/`select` cho quan hệ; tránh gọi DB trong vòng lặp.
- Lưu ý: Prisma mặc định KHÔNG JOIN mà chạy nhiều query rồi gộp ở app (thêm 1 `findMany ... WHERE id IN (...)`, tức chỉ thêm MỘT query); Prisma 5.x có thể bật `relationLoadMode: 'join'`.
- Phát hiện: bật `log: ['query']`; hoặc batch thủ công bằng `where: { id: { in: [...] } }`.

```ts
// XAU: N+1
const issues = await prisma.issue.findMany();
for (const i of issues) {
  i.assignee = await prisma.user.findUnique({ where: { id: i.assigneeId } });
}
// TOT: gom thanh it query
const issues = await prisma.issue.findMany({
  include: { assignee: true, labels: true },
});
```

</details>

<details>
<summary><b>19.6 Cascade delete và soft delete khác nhau thế nào? Khi nào dùng soft delete?</b></summary>

**📌 Câu trả lời mẫu:**

"Khác biệt cốt lõi là cascade delete xóa thật khỏi DB, còn soft delete chỉ đánh dấu một cờ như `deletedAt` chứ dòng vẫn nằm đó. Với cascade, khi mình xóa cha thì DB tự dọn hết con phụ thuộc — tiện và sạch, nhưng dữ liệu mất vĩnh viễn không lấy lại được. Với soft delete thì mình giữ được lịch sử để audit và khôi phục, đổi lại mọi câu query phải nhớ lọc `WHERE deletedAt IS NULL`, nếu quên một chỗ là lòi data đã xóa ra. Trong Jira Clone mình xài soft delete cho những thứ cần audit hoặc cần undo như issue, project; còn các bản ghi mà mất cha là vô nghĩa, ví dụ comment của một issue đã xóa, thì để cascade cho gọn. Một cái bẫy mình hay nhắc là unique constraint: email 'đã xóa' vẫn chiếm chỗ nên người mới không đăng ký lại được, lúc đó phải dùng partial unique index chỉ áp lên dòng chưa xóa."

---

- Cascade delete (`onDelete: Cascade`): xóa cha → DB tự xóa con phụ thuộc; tiện khi con vô nghĩa nếu cha mất, nhưng mất data vĩnh viễn.
- Soft delete: không xóa thật, đánh dấu `deletedAt`/`isDeleted`, mọi query phải lọc `WHERE deletedAt IS NULL`; giữ audit, cho khôi phục.
- Bẫy soft delete: phải nhớ lọc khắp nơi (tập trung qua Prisma Client Extension/repository chung; `$use` đã deprecated) và unique constraint phức tạp (email "đã xóa" vẫn chiếm chỗ).
- Dùng soft delete khi cần audit/recovery; cascade khi con thực sự hết giá trị.

```prisma
model Issue {
  id        String   @id @default(cuid())
  projectId String
  project   Project  @relation(fields: [projectId], references: [id], onDelete: Cascade)
  deletedAt DateTime? // soft delete
  @@index([projectId])
}
// Partial index WHERE deletedAt IS NULL phai dat trong migration SQL.
```

</details>

---

## 20. Advanced Node.js

<details>
<summary><b>20.1 Cơ chế xử lý lỗi (error handling) trong Node async khác gì giữa callback, Promise và EventEmitter?</b></summary>

**📌 Câu trả lời mẫu:**

"Ba cơ chế này lan truyền lỗi theo ba kiểu hoàn toàn khác nhau nên cách bắt lỗi cũng khác. Callback kiểu Node đặt lỗi ở tham số đầu `(err, data)`, nên nếu mình quên kiểm tra `err` thì lỗi bị nuốt im lặng. Promise và async/await thì lỗi đi qua rejection, mình bọc `try/catch` quanh `await` là bắt được; nhưng nếu quên `await` thì nó thành `unhandledRejection` trôi đi mất. EventEmitter là cái khác biệt nguy hiểm nhất: lỗi phát qua sự kiện `'error'`, và nếu không có listener nào lắng nghe thì Node sẽ ném thẳng và crash cả process — như stream trong Jira Clone mình bắt buộc phải gắn `.on('error')`. Một điểm dễ sai nữa là lỗi ném bên trong callback async, ví dụ trong `setTimeout`, thì `try/catch` bên ngoài không bắt được vì lúc đó stack đã khác rồi. Nên ở tầng app mình luôn cài global guard cho `unhandledRejection` và `uncaughtException` để log lại rồi thoát có kiểm soát thay vì để chết âm thầm."

---

- **Callback** kiểu Node: lỗi ở tham số đầu (`(err, data) => ...`); quên check `err` là mất lỗi âm thầm.
- **Promise/async-await**: lỗi lan qua rejection → `try/catch` quanh `await` hoặc `.catch()`; `await` không bọc → `unhandledRejection`.
- **EventEmitter**: lỗi qua sự kiện `'error'`; KHÔNG có listener `'error'` thì Node NÉM và crash process (khác biệt then chốt).
- Lỗi ném trong callback async (vd `setTimeout`) không bắt được bởi `try/catch` ngoài.
- Nên cài global guard `unhandledRejection`/`uncaughtException` để LOG rồi thoát có kiểm soát, không nuốt lỗi.

```js
try { setTimeout(() => { throw new Error('boom'); }, 0); }
catch (e) { /* khong bao gio chay -> process crash */ }
stream.on('error', (err) => logger.error(err)); // PHAI lang nghe, neu khong se crash
```

</details>

---

## 21. Advanced Testing

<details>
<summary><b>21.1 Test pyramid là gì? Tại sao nên có nhiều unit test hơn e2e test?</b></summary>

**📌 Câu trả lời mẫu:**

"Test pyramid là mô hình gợi ý tỷ lệ các loại test: nhiều unit test ở đáy, ít integration test ở giữa, và rất ít e2e ở đỉnh. Lý do nên nhiều unit test là vì chúng nhanh, chạy cách ly nên khi fail là chỉ thẳng ra chỗ lỗi, và rất rẻ để viết với chạy. Còn e2e thì đi xuyên qua HTTP, DB, network nên chậm và dễ flaky, mình chỉ nên dùng để phủ vài luồng quan trọng nhất. Cái cần tránh là 'ice cream cone' — kim tự tháp lộn ngược, quá nhiều e2e mà ít unit — làm CI chạy lâu và mỗi lần đỏ là rất khó biết hỏng ở đâu. Trong Jira Clone mình test kỹ logic sinh issue key bằng unit, còn luồng login với httpOnly cookie thì chỉ cần một vài e2e là đủ."

---

- Mô tả tỷ lệ lý tưởng: nhiều **unit test** ở đáy (nhanh, cách ly, rẻ), ít hơn **integration** ở giữa, rất ít **e2e** ở đỉnh (chậm, dễ flaky, đắt).
- Unit test cho feedback gần như tức thì và chỉ ra chính xác nơi lỗi.
- E2e chạy qua nhiều lớp (HTTP, DB, network) nên chậm và dễ flaky.
- Anti-pattern: "ice cream cone" (quá nhiều e2e, ít unit) khiến CI chậm, khó debug.

Ví dụ: logic sinh issue key `PROJECT-N` test kỹ bằng unit (mock Prisma); luồng login + httpOnly cookie chỉ cần vài e2e.

</details>

<details>
<summary><b>21.2 Phân biệt mock, spy, stub và fake. Khi nào dùng loại nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Bốn cái này đều là test double nhưng mục đích khác nhau. Stub là mình nhét sẵn giá trị trả về để lái luồng test đi theo hướng mình muốn, mình không quan tâm nó được gọi mấy lần. Spy là để theo dõi: nó ghi lại hàm được gọi với tham số gì, bao nhiêu lần — và điểm hay nhầm là `jest.spyOn` mặc định vẫn gọi hàm thật, chỉ biến thành stub khi mình gắn thêm `.mockResolvedValue` hay `.mockImplementation`. Mock thì mạnh hơn, mình đặt trước kỳ vọng về tương tác và nó sẽ fail nếu tương tác xảy ra sai. Còn fake là một bản cài đặt thật nhưng đơn giản, ví dụ một in-memory repository thay cho DB. Quy tắc của mình là khi cần kiểm chứng kết quả (state) thì dùng stub hoặc fake; khi bản chất hành vi là một tác động phụ cần xác nhận, như có gọi gửi email hay không, thì mới dùng spy hoặc mock để tránh test quá dính vào chi tiết cài đặt."

---

- **Stub**: trả về giá trị có sẵn để điều khiển luồng (state-based), không quan tâm gọi mấy lần.
- **Spy**: ghi lại các lần gọi (arguments, số lần); `jest.spyOn` mặc định vẫn gọi hàm thật, chỉ thành stub khi gắn `.mockImplementation`/`.mockResolvedValue`.
- **Mock**: đặt trước kỳ vọng về tương tác và kiểm chứng (behavior verification); **fail nếu tương tác sai**.
- **Fake**: bản cài đặt thật nhưng đơn giản (in-memory repository thay DB).

```ts
// stub: thay the de tra ve (khong goi ham that)
jest.spyOn(repo, 'findUser').mockResolvedValue(user);

// spy: theo doi, VAN goi ham that
const spy = jest.spyOn(logger, 'warn');
service.handle(dirtyInput);
expect(spy).toHaveBeenCalledWith('PII redacted');
```

</details>

<details>
<summary><b>21.3 Code coverage đo cái gì và giới hạn của nó là gì? 100% coverage có nghĩa là không còn bug?</b></summary>

**📌 Câu trả lời mẫu:**

"Code coverage đo phần trăm code được thực thi khi chạy test — thường tính theo dòng, nhánh, hàm hay statement. Điểm mấu chốt là nó chỉ cho biết code có chạy hay không, chứ không hề đảm bảo mình có assert đúng. Mình hoàn toàn có thể đạt 100% coverage mà test vô nghĩa, ví dụ gọi hàm rồi không kiểm tra kết quả gì cả, hoặc vẫn sót edge case như null, race condition, lỗi mạng. Cho nên 100% coverage không có nghĩa là hết bug, nó chỉ là một chỉ báo bổ sung chứ không phải mục tiêu cuối. Mình thường quan tâm branch coverage cho mấy chỗ logic quan trọng, ví dụ phần phân quyền trong Jira Clone, và nếu muốn đánh giá chất lượng assertion thật sự thì dùng mutation testing như Stryker — nó sửa code rồi xem test có bắt được không."

---

- Đo **% code được thực thi** khi test (line/branch/function/statement); chỉ biết code **chạy**, không đảm bảo **kiểm chứng đúng**.
- Có thể đạt 100% mà không có assertion nghĩa lý, hoặc vẫn sót edge case (null, concurrency, lỗi mạng).
- Coverage chỉ là **chỉ báo bổ sung**, không phải mục tiêu cuối.
- Nên tập trung **branch coverage** cho logic quan trọng và dùng **mutation testing** (Stryker) để đánh giá chất lượng assertion.

```ts
// 100% line coverage nhung vo nghia: khong he assert
it('cong so', () => { sum(2, 2); });
```

</details>

<details>
<summary><b>21.4 Integration test khác unit test và e2e test thế nào? Khi nào nên viết integration test?</b></summary>

**📌 Câu trả lời mẫu:**

"Ba loại này khác nhau ở chỗ chạy bao nhiêu thành phần thật. Unit test kiểm tra một đơn vị cách ly, mock hết phụ thuộc xung quanh. Integration test cho nhiều thành phần thật làm việc cùng nhau, ví dụ service nối với Prisma và DB thật, nhưng thường bỏ qua tầng HTTP. E2e thì chạy toàn bộ qua HTTP đúng như người dùng thật gọi vào. Mình viết integration test khi rủi ro nằm ở các ranh giới mà mock hay che giấu lỗi — như mapping của ORM, câu SQL thực sự sinh ra, transaction hay unique constraint. Ví dụ trong Jira Clone, logic sinh issue key mình phải test với DB thật để chắc rằng `$transaction` cộng unique constraint chặn được trùng key khi hai request vào cùng lúc; cái này mock Prisma không bao giờ phát hiện ra được vì mock đâu có chạy SQL thật."

---

- **Unit**: kiểm tra đơn vị cách ly (mock toàn bộ phụ thuộc).
- **Integration**: nhiều thành phần thật cùng làm việc (service + Prisma + DB thật), thường bỏ qua tầng HTTP/controller.
- **E2e**: chạy toàn bộ qua HTTP như người dùng thật.
- Integration đặc biệt giá trị khi rủi ro nằm ở **ranh giới** (ORM mapping, SQL sinh ra, transaction, constraint) — nơi mock che giấu lỗi.

Ví dụ: integration test repo sinh issue key với DB thật để chắc `$transaction` + unique constraint chặn được trùng key khi 2 request song song (mock Prisma không bao giờ bắt được).

</details>

<details>
<summary><b>21.5 Flaky test là gì, nguyên nhân phổ biến và cách phòng tránh?</b></summary>

**📌 Câu trả lời mẫu:**

"Flaky test là test lúc pass lúc fail dù code không hề thay đổi, và nó rất nguy hiểm vì làm cả team mất niềm tin vào CI — riết rồi ai cũng bấm chạy lại cho đỏ thành xanh, che luôn bug thật. Nguyên nhân phổ biến nhất là phụ thuộc thời gian như `sleep` cứng, phụ thuộc thứ tự test do state rò rỉ giữa các test, race condition do quên await promise, rồi clock, random, timezone, hoặc gọi mạng và dịch vụ ngoài thật. Cách mình phòng tránh là cô lập dữ liệu cho mỗi test để không dính nhau, dùng fake timers thay cho sleep, mock luôn thời gian và random cho kết quả ổn định, await cho hết mọi promise, và stub các service bên ngoài. Nguyên tắc chung là test phải tất định — cùng input thì luôn cùng kết quả."

---

- Flaky test: lúc pass lúc fail dù code không đổi — làm mất niềm tin vào CI.
- Nguyên nhân: phụ thuộc **thời gian/`sleep` cứng**, **thứ tự test** (state rò rỉ), **bất đồng bộ** (race, promise không await), **clock/random/timezone**, gọi **mạng/dịch vụ ngoài** thật.
- Cách tránh: cách ly dữ liệu mỗi test, dùng fake timers thay sleep, mock thời gian + random, await hết promise, stub service ngoài.

```ts
jest.useFakeTimers();
jest.setSystemTime(new Date('2026-06-25'));
jest.advanceTimersByTime(1000); // thay vi sleep(1000)
// jest.useRealTimers(); // khoi phuc sau test
```

</details>

<details>
<summary><b>21.6 Làm thế nào test code bất đồng bộ đúng cách? AAA pattern là gì?</b></summary>

**📌 Câu trả lời mẫu:**

"Với code bất đồng bộ, quy tắc số một là phải await Promise hoặc return nó ra; nếu quên await thì assertion chạy sau khi test đã kết thúc, dẫn tới test xanh giả — nó báo pass nhưng thật ra chưa kiểm tra gì cả. Khi muốn kiểm tra một hàm phải reject thì mình dùng `await expect(fn()).rejects.toThrow(...)`. Còn với Observable của RxJS thì dùng `firstValueFrom` hoặc marble testing. AAA là cách bố cục một test cho dễ đọc: Arrange là chuẩn bị dữ liệu và mock, Act là thực hiện đúng một hành động cần kiểm, Assert là kiểm tra kết quả. Mỗi test chỉ nên tập trung vào một hành vi thôi, như test 'rotate token hết hạn thì ném Unauthorized' trong Jira Clone — chia rõ ba phần thì sau này đọc lại hiểu ngay test đang khẳng định điều gì."

---

- Code async: luôn **await** Promise (hoặc return nó) — quên await khiến assertion chạy sau khi test kết thúc (test xanh giả).
- Kiểm tra reject: `await expect(fn()).rejects.toThrow(...)`.
- Observable (RxJS): dùng `firstValueFrom` hoặc marble testing.
- **AAA** = **Arrange** (chuẩn bị dữ liệu/mock), **Act** (gọi hàm, lý tưởng 1 hành động), **Assert** (kiểm kết quả); mỗi test tập trung 1 hành vi.

```ts
it('nem khi refresh token het han', async () => {
  jest.spyOn(repo, 'find').mockResolvedValue(expiredToken); // Arrange
  // Act + Assert (phai await, neu khong test pass truoc khi reject)
  await expect(auth.rotate(token)).rejects.toThrow(UnauthorizedException);
});
```

</details>

<details>
<summary><b>21.7 Test data builder / factory là gì và tại sao quan trọng cho test đạt chất lượng?</b></summary>

**📌 Câu trả lời mẫu:**

"Test data builder hay factory là một hàm tạo sẵn entity hợp lệ với các giá trị mặc định, và mình chỉ override đúng những trường liên quan tới điều đang test. Vấn đề khi không có nó là mình phải dựng object dài dòng lặp đi lặp lại trong từng test, vừa khó đọc vừa rất dễ vỡ — schema đổi một trường là phải sửa hàng chục chỗ. Dùng factory thì giảm trùng lặp, và quan trọng hơn là làm nổi bật intent: nhìn vào `makeUser({ role: 'DEVELOPER' })` là biết ngay test này quan tâm tới vai trò, mấy trường còn lại chỉ là nhiễu. Trong Jira Clone mình hay viết factory cho user, project, issue; với DB thật thì có thể dùng thư viện như Fishery hoặc tự viết factory dựa trên seed của Prisma. Khi schema thay đổi mình chỉ cần sửa một chỗ trong factory là toàn bộ test vẫn chạy."

---

- Tạo dữ liệu thủ công (object dài dòng, lặp lại) khiến test khó đọc, dễ vỡ khi schema đổi.
- **Factory/Builder** tạo entity hợp lệ với giá trị mặc định, chỉ override trường liên quan — làm nổi bật "điều gì đang được kiểm".
- Lợi ích: giảm trùng lặp, rõ intent, schema đổi chỉ sửa một chỗ.
- Với DB thật dùng thư viện như Fishery hoặc viết factory từ seed Prisma.

```ts
const makeUser = (overrides: Partial<User> = {}): User => ({
  id: 'u1', email: 'a@b.c', role: 'DEVELOPER', ...overrides,
});

it('chan DEVELOPER xoa project', () => {
  const user = makeUser({ role: 'DEVELOPER' }); // chi truong nay quan trong
  expect(() => service.deleteProject(user, 'p1')).toThrow(ForbiddenException);
});
```

</details>

---

## 22. System Design and Scaling

<details>
<summary><b>22.1 Scale ngang app stateless: vì sao không được lưu state in-memory? Ảnh hưởng gì đến thiết kế?</b></summary>

**📌 Câu trả lời mẫu:**

"Khi scale ngang, mình chạy nhiều instance giống nhau và load balancer phân phối request, nên cùng một user có thể trúng instance khác nhau giữa các request. Vì vậy mình không được lưu state in-memory như session hay counter rate-limit ngay trong RAM của process: mỗi instance sẽ giữ một bản riêng, dẫn tới hành vi không nhất quán, và khi restart hay scale-down thì state đó mất luôn. Cách thiết kế đúng là làm app stateless — đẩy mọi state chia sẻ ra ngoài: Redis cho session, cache và rate-limit; Postgres cho dữ liệu bền; S3 cho file. Trong Jira Clone mình dùng JWT đặt trong httpOnly cookie nên gần như stateless, mọi instance chỉ cần shared secret hoặc public key là verify được, không phải hỏi DB. Đánh đổi là JWT khó revoke ngay, nên khi cần logout hoặc đổi mật khẩu có hiệu lực tức thì thì mình vẫn phải thêm một blocklist trên Redis."

---

- Load balancer phân phối request đến nhiều instance; một user có thể trúng instance khác nhau giữa các request.
- State **in-memory** (session, rate-limit counter) thì mỗi instance một bản -> hành vi không nhất quán, mất data khi restart/scale-down.
- Phải **stateless**: đẩy state chia sẻ ra ngoài — Redis (session/cache/rate-limit), Postgres (data bền), S3 (file).
- JWT (httpOnly cookie) gần stateless: mọi instance verify bằng shared secret/public key, không cần DB. Nhưng muốn revoke sớm (logout, đổi mật khẩu) vẫn cần blocklist trên Redis.

```ts
// SAI khi scale ngang: counter chi dung tren 1 instance
const loginAttempts = new Map<string, number>();
// DUNG: day ra Redis, atomic, set TTL
const attempts = await redis.incr(`login:attempts:${userId}`);
if (attempts === 1) await redis.expire(`login:attempts:${userId}`, 900);
```

</details>

<details>
<summary><b>22.2 Caching với Redis: giải thích cache-aside và các chiến lược invalidation?</b></summary>

**📌 Câu trả lời mẫu:**

"Cache-aside, hay còn gọi lazy loading, là pattern mình hay dùng nhất với Redis: khi đọc thì check cache trước, nếu hit thì trả luôn, nếu miss thì query DB rồi ghi kết quả vào cache kèm TTL. Điểm cốt lõi là app tự quản lý cache còn DB vẫn là source of truth. Về invalidation thì có vài cách: đơn giản nhất là để TTL tự hết hạn nếu chấp nhận dữ liệu stale một chút; chủ động hơn là explicit delete key mỗi khi ghi; hoặc write-through ghi cả DB lẫn cache cùng lúc. Một quy tắc mình luôn theo là xóa key an toàn hơn update cache trực tiếp, vì update dễ dính race condition khiến giá trị cũ đè lên giá trị mới — và thứ tự đúng là update DB trước rồi mới del cache. Trong Jira Clone mình cache thông tin project và board vì chúng đọc nhiều ghi ít, mỗi lần update project là mình del key project:{id} ngay. Cuối cùng cần lưu ý cache stampede: khi một key hot hết hạn cùng lúc, hàng loạt request cùng miss và đập thẳng vào DB — mình xử lý bằng lock single-flight hoặc stale-while-revalidate để chỉ một request đi rebuild cache."

---

- **Cache-aside** (lazy loading): đọc cache trước; **miss** -> query DB rồi ghi lại cache với TTL; **hit** -> trả luôn. App quản lý cache, DB là source of truth.
- Invalidation: (1) TTL (chấp nhận stale ngắn); (2) explicit delete khi ghi; (3) write-through (ghi DB + cache cùng lúc); (4) event-based (invalidate phân tán).
- Quy tắc vàng: **xóa key an toàn hơn update cache trực tiếp** (tránh race ghi giá trị cũ đè lên). Thứ tự: update DB trước rồi del cache.
- Lưu ý cache stampede: nhiều request cùng miss khi key hết hạn -> dùng lock (single-flight) hoặc stale-while-revalidate.

```ts
async getProject(id: string) {
  const cached = await redis.get(`project:${id}`);
  if (cached) return JSON.parse(cached);          // hit
  const project = await prisma.project.findUnique({ where: { id } });
  if (project) await redis.set(`project:${id}`, JSON.stringify(project), 'EX', 300); // chi cache khi co data
  return project;
}
// Update: xoa key thay vi ghi de
await prisma.project.update({ where: { id }, data: dto });
await redis.del(`project:${id}`);
```

</details>

<details>
<summary><b>22.3 Message queue (Bull/RabbitMQ): khi nào dùng cho tác vụ nặng/async và lợi ích gì?</b></summary>

**📌 Câu trả lời mẫu:**

"Message queue là một hàng đợi nằm giữa nơi tạo việc và nơi xử lý việc, giúp mình tách phần xử lý nặng ra khỏi luồng HTTP. Bản chất là khi có một tác vụ tốn thời gian hoặc có thể làm bất đồng bộ — như gửi email, export PDF, resize ảnh hay gọi API bên thứ ba — thay vì bắt người dùng đợi, mình chỉ đẩy một job vào queue rồi trả response ngay, còn worker chạy nền sẽ nhặt job ra xử lý sau. Cái lợi lớn nhất là response nhanh và hệ thống chịu tải tốt hơn: nếu có một đợt request dồn dập thì queue hấp thụ bớt, đồng thời mình có sẵn cơ chế retry với backoff khi job lỗi, và kiểm soát được số job chạy song song để không làm sập dịch vụ phía sau. Trong NestJS mình hay dùng BullMQ chạy trên Redis cho job queue đơn giản, còn RabbitMQ thì mạnh hơn về routing và giao tiếp giữa nhiều service. Một điểm phải nhớ là queue thường giao hàng kiểu at-least-once, tức một job có thể bị chạy lại, nên consumer bắt buộc phải idempotent để chạy hai lần vẫn không sinh dữ liệu trùng."

---

- Dùng khi việc **tốn thời gian** hoặc **làm bất đồng bộ** được với HTTP request: gửi email, export PDF/CSV, xử lý ảnh, gọi API bên thứ ba, notification hàng loạt.
- Lợi ích: trả response ngay (low latency); **retry + backoff** tự động; **rate control / concurrency** tránh quá tải downstream; scale worker độc lập; hấp thụ burst tải.
- **Bull/BullMQ** chạy trên Redis (`@nestjs/bullmq`), hợp job queue trong NestJS; **RabbitMQ** là broker thật, mạnh về routing (exchange/binding), pub-sub, giao tiếp đa service.
- Queue thường at-least-once -> consumer phải **idempotent** vì một job có thể chạy lại.

```ts
// Producer: day job, tra ve ngay
await this.emailQueue.add('send-otp', { userId, email },
  { attempts: 3, backoff: { type: 'exponential', delay: 2000 }, removeOnComplete: true });
// Consumer (BullMQ)
@Processor('email')
export class EmailProcessor extends WorkerHost {
  async process(job: Job) {
    if (job.name === 'send-otp') await this.resend.send(job.data.email);
  }
}
```

</details>

<details>
<summary><b>22.4 Idempotency: tại sao cần và triển khai idempotency key cho API thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Idempotency nghĩa là một thao tác dù gọi nhiều lần cũng cho kết quả như gọi đúng một lần. Mình cần nó vì trong thực tế client rất hay retry: ví dụ người dùng bấm nút thanh toán, mạng timeout nhưng thật ra server đã xử lý xong, client không biết nên gửi lại — nếu không xử lý thì sẽ bị charge hai lần hoặc tạo trùng tài nguyên. Cách triển khai phổ biến là cho client gửi kèm một header Idempotency-Key dạng UUID; server lưu key đó cùng kết quả, lần sau gặp lại đúng key thì trả lại response cũ thay vì chạy lại logic. Điểm khó là phải xử lý race khi hai request cùng key đến gần như đồng thời, nên mình dùng unique constraint trong DB hoặc lệnh SET NX của Redis để đảm bảo chỉ một request được phép thực thi, còn lại phải chờ hoặc nhận kết quả đã có. GET/PUT/DELETE thì bản thân đã idempotent theo định nghĩa HTTP, riêng POST tạo mới là không, nên cơ chế này chủ yếu dùng cho POST và các tác vụ nhạy cảm như thanh toán."

---

- Idempotency: gọi thao tác **nhiều lần cho kết quả như gọi một lần** — quan trọng với POST tạo tài nguyên, thanh toán, hoặc khi client retry do network timeout.
- Triển khai: client gửi header `Idempotency-Key` (UUID); server lưu key + kết quả; nếu key đã có thì trả lại response cũ thay vì thực thi lại.
- GET/PUT/DELETE vốn idempotent theo định nghĩa HTTP; POST thì không, nên cần cơ chế này.
- Phải xử lý race khi hai request cùng key đến gần đồng thời: dùng unique constraint DB hoặc `SET NX` Redis để chỉ một request thực thi. Consumer queue (at-least-once) càng cần.

```ts
async createWithIdempotency(key: string, dto: CreateDto) {
  const acquired = await redis.set(`idem:${key}`, 'PENDING', 'EX', 86400, 'NX'); // chi giu duoc neu chua ton tai
  if (!acquired) {
    const existing = await redis.get(`idem:${key}`);
    return existing && existing !== 'PENDING' ? JSON.parse(existing) : null;
  }
  const result = await this.create(dto);
  await redis.set(`idem:${key}`, JSON.stringify(result), 'EX', 86400);
  return result;
}
```

</details>

---

## 23. API and REST Design

<details>
<summary><b>23.1 Trình bày các REST conventions bạn tuân thủ khi đặt tên resource và thiết kế endpoint?</b></summary>

**📌 Câu trả lời mẫu:**

"Khi thiết kế API cho Jira Clone, mình bám vào vài quy ước REST cốt lõi. Resource luôn là danh từ số nhiều như `/projects` hay `/issues`, không nhét động từ vào URL vì bản thân HTTP method (GET/POST/PATCH/DELETE) đã nói lên hành động rồi. Quan hệ cha-con thì mình dùng nested route, ví dụ `/projects/:id/issues` để lấy issue của một project, nhưng mình cố không lồng quá 2 cấp — sâu hơn nữa thì tách thành resource độc lập như `/issues/:id` rồi lọc bằng query param cho đỡ rối. Mọi chuyện lọc, sắp xếp, phân trang mình đẩy hết vào query string kiểu `?status=OPEN&sort=-createdAt&limit=20` thay vì đẻ ra path mới. Với những thao tác không map gọn vào CRUD — như chuyển trạng thái issue trên board — mình mô hình hóa thành sub-resource hành động kiểu `POST /issues/:id/transitions`, vì cố ép nó thành PATCH một field trạng thái sẽ giấu mất logic nghiệp vụ phía sau. Cuối cùng mình phân biệt rõ PATCH cho cập nhật một phần và PUT cho thay thế toàn bộ. Cái mình ưu tiên nhất là tính nhất quán: thà chọn một quy ước rồi áp đều toàn hệ thống, còn hơn mỗi endpoint một kiểu khiến frontend phải nhớ ngoại lệ."

---

- Resource là danh từ số nhiều (`/projects`, `/issues`), không dùng verb vì HTTP method đã thể hiện hành động.
- Quan hệ lồng nhau dùng nested route (`/projects/:id/issues`) nhưng không lồng quá 2 cấp — sâu hơn thì tách resource độc lập + query filter.
- Path dùng kebab-case; hành động đặc biệt (không map CRUD) mô hình hóa thành sub-resource/state transition.
- Lọc/sắp xếp/phân trang qua query param (`?status=OPEN&sort=-createdAt&limit=20`), không tạo path mới.
- Phân biệt PATCH (cập nhật 1 phần) vs PUT (thay thế toàn bộ).

```
GET    /projects/:id/issues        # list issues cua project
POST   /projects/:id/issues        # tao issue
GET    /issues/:id                 # chi tiet (resource doc lap)
PATCH  /issues/:id                 # cap nhat 1 phan
POST   /issues/:id/transitions     # action: chuyen trang thai (khong phai CRUD)
```

</details>

<details>
<summary><b>23.2 Bạn chọn HTTP status code như thế nào cho các case: tạo mới, validation fail, chưa đăng nhập, không đủ quyền, conflict?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình chọn status code theo đúng ngữ nghĩa của từng tình huống để client nhìn code là biết chuyện gì xảy ra. Khi tạo mới thành công mình trả 201 Created, lý tưởng là kèm header Location trỏ tới resource vừa tạo; còn update hay delete thành công mà không cần trả body thì dùng 204 No Content. Với lỗi validation, ví dụ DTO tạo issue sai schema, mình trả 400 Bad Request. Hai code dễ nhầm nhất là 401 và 403: 401 là chưa xác thực — thiếu token, token sai hoặc hết hạn; còn 403 là đã đăng nhập rồi nhưng không đủ quyền, kiểu một VIEWER cố sửa project. Khi có xung đột trạng thái hoặc vi phạm ràng buộc unique, chẳng hạn tạo trùng slug, mình dùng 409 Conflict. Nguyên tắc bao trùm là phân biệt rõ 4xx là lỗi do client gửi sai, còn 5xx là lỗi từ phía server — và quan trọng nhất là nhất quán trên toàn API để frontend không phải đoán."

---

- `201 Created` (kèm header `Location`) khi tạo mới; `204 No Content` khi update/delete thành công không trả body.
- `400 Bad Request` khi validation/schema sai; `422` (tùy chọn, WebDAV/RFC 4918) khi JSON hợp lệ nhưng vi phạm nghiệp vụ — chọn 1 quy ước và nhất quán.
- `401 Unauthorized` = chưa xác thực/credential sai-hết hạn (thực chất là "unauthenticated"); `403 Forbidden` = đã xác thực nhưng không đủ quyền.
- `409 Conflict` khi xung đột trạng thái / trùng ràng buộc unique.
- Phân biệt rõ 4xx (lỗi client) vs 5xx (lỗi server).

```
201 Created      POST /issues        -> tao issue moi (kem Location)
204 No Content   DELETE /issues/:id  -> xoa thanh cong, khong body
400 Bad Request  body sai schema (DTO validation)
401 Unauthorized JWT thieu/het han/khong hop le
403 Forbidden    user khong du quyen (VIEWER sua project)
409 Conflict     tao trung slug (vi pham unique)
```

</details>

<details>
<summary><b>23.3 Idempotency là gì? GET, POST, PUT, DELETE — cái nào idempotent, cái nào safe, và xử lý POST tạo trùng thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Idempotent nghĩa là gọi một thao tác nhiều lần thì trạng thái server vẫn giống như gọi đúng một lần — code trả về có thể khác nhưng dữ liệu không bị nhân lên. Còn safe là thao tác hoàn toàn không thay đổi trạng thái server. Theo đó GET vừa safe vừa idempotent vì chỉ đọc; PUT và DELETE thì idempotent — PUT vì ghi đè cùng một nội dung bao nhiêu lần kết quả vẫn vậy, DELETE vì xóa lần đầu xong thì các lần sau trạng thái vẫn là 'đã xóa'. Riêng POST thì không idempotent, gọi hai lần là tạo ra hai resource. Đây chính là vấn đề khi client retry do timeout: dễ tạo trùng. Cách xử lý của mình là cho client gửi kèm header Idempotency-Key; server lưu lại key cùng kết quả, nếu thấy key đã xử lý rồi thì trả response cũ chứ không chạy lại logic tạo mới. Cách này đặc biệt quan trọng với các thao tác như thanh toán."

---

- Idempotent = gọi N lần cho cùng **trạng thái server** như gọi 1 lần (status code có thể khác).
- Safe = không đổi trạng thái server (`GET`, `HEAD`).
- `GET`/`PUT`/`DELETE` idempotent (`GET` còn safe); `POST` không idempotent (gọi 2 lần tạo 2 resource).
- `DELETE` idempotent vì sau lần xóa đầu trạng thái vẫn là "đã xóa".
- Chống tạo trùng do client retry: dùng **Idempotency-Key** header — server lưu key + kết quả, gặp lại key đã xử lý thì trả response cũ.

```http
POST /payments
Idempotency-Key: 9f1c-uuid-from-client
# Server: neu key da ton tai -> tra response da luu, khong charge lai
```

</details>

<details>
<summary><b>23.4 Các chiến lược API versioning? Bạn chọn cái nào và tại sao?</b></summary>

**📌 Câu trả lời mẫu:**

"Có ba cách versioning API chính: nhét version vào URI như `/v1/users`, để trong header như `X-API-Version` hoặc qua media-type trong `Accept`, và để trong query param như `?version=1`. Mình thường chọn URI versioning vì nó rõ ràng nhất — nhìn URL là biết ngay đang gọi version nào, dễ test bằng browser hay curl, và dễ cache. Đánh đổi của nó là về lý thuyết REST thì hơi 'không thuần', vì URI lẽ ra nên trỏ tới một resource ổn định bất kể version; cách header thì sạch hơn về mặt đó nhưng lại khó debug và khó cache hơn. Mình coi version như một hợp đồng với client, nên chỉ bump major khi thực sự có breaking change, còn lại cố giữ tương thích ngược bằng cách thêm field mới thay vì đổi hay xóa field cũ. NestJS hỗ trợ sẵn versioning qua `enableVersioning`, nên việc khai báo version cho từng controller khá gọn."

---

- 3 cách: URI (`/v1/users`), header (custom `X-API-Version: 1` hoặc media-type qua `Accept`), query param (`?version=1`).
- URI rõ ràng, dễ cache, dễ test bằng browser/curl nên phổ biến nhất; đánh đổi là "không thuần REST" (URI nên trỏ resource ổn định bất kể version).
- Header/media-type sạch hơn về resource nhưng khó debug và khó cache.
- Coi version là hợp đồng: chỉ bump major khi breaking change; ưu tiên backward-compatible bằng cách thêm field thay vì đổi/xóa field cũ.
- NestJS hỗ trợ natively `VersioningType.URI/HEADER/MEDIA_TYPE/CUSTOM`.

```ts
app.enableVersioning({ type: VersioningType.URI }); // -> /v1/...

@Controller({ path: 'issues', version: '1' })
export class IssuesV1Controller {}
```

</details>

<details>
<summary><b>23.5 So sánh offset pagination và cursor pagination — khi nào dùng cái nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Offset pagination là kiểu LIMIT/OFFSET quen thuộc, ví dụ lấy 20 dòng bỏ qua 100 dòng đầu. Ưu điểm là đơn giản, nhảy tới trang bất kỳ được và hiển thị được tổng số trang. Nhưng nó có hai nhược: khi offset lớn thì chậm vì DB vẫn phải duyệt qua rồi bỏ đi các dòng đầu, và bị 'page drift' — nếu giữa các lần gọi có ai đó thêm hoặc xóa bản ghi thì trang sau có thể bị lặp hoặc nhảy cóc dữ liệu. Cursor pagination thì dùng một mốc đã được index làm con trỏ, ví dụ id hoặc cặp createdAt và id của bản ghi cuối, rồi lấy tiếp từ mốc đó. Cách này ổn định kể cả khi dữ liệu đang thay đổi và nhanh vì tận dụng được index. Đổi lại nó chỉ đi tiến hoặc lùi tuần tự, không nhảy tới trang bất kỳ. Nên mình dùng offset cho trang admin cần phân trang số kiểu cổ điển, còn dùng cursor cho feed hoặc danh sách dữ liệu lớn, real-time như danh sách issue cuộn vô tận. Lưu ý cột làm cursor phải duy nhất và thứ tự ổn định, nếu sort theo createdAt có thể trùng thì thêm id làm tie-breaker."

---

- Offset (`LIMIT 20 OFFSET 100`): đơn giản, nhảy trang bất kỳ, hiện tổng số trang.
- Nhược của offset: deep offset chậm (DB phải duyệt-bỏ qua các dòng đầu) và "page drift" khi có insert/delete giữa các lần gọi.
- Cursor: dùng mốc ổn định đã index (`id` hoặc `(createdAt, id)`) làm con trỏ — ổn định với dữ liệu thay đổi và nhanh vì tận dụng index.
- Đánh đổi của cursor: chỉ đi tiến/lùi tuần tự, không nhảy tùy ý trang.
- Cột làm cursor phải duy nhất + thứ tự ổn định; nếu sort theo `createdAt` có thể trùng thì thêm `id` làm tie-breaker.

```ts
const issues = await prisma.issue.findMany({
  take: 20,
  ...(cursor && { skip: 1, cursor: { id: cursor } }),
  orderBy: { id: 'desc' },
});
const nextCursor = issues.length === 20 ? issues.at(-1)?.id : null;
```

</details>

<details>
<summary><b>23.6 Response envelope { message, data } và error format nhất quán — bạn thiết kế ra sao trong NestJS?</b></summary>

**📌 Câu trả lời mẫu:**

"Ý tưởng response envelope là mọi response trả về đều có một cấu trúc cố định để client xử lý nhất quán, không phải đoán. Với case thành công mình bọc lại thành dạng `{ message, data }`, còn lỗi thì trả một shape cố định gồm statusCode, message và error, có thể thêm một mã code nội bộ. Trong NestJS mình tách làm hai chỗ: dùng một Interceptor để bọc mọi response thành công vào envelope, nhờ vậy controller chỉ cần `return data` cho gọn; còn lỗi thì dùng Exception Filter để chuẩn hóa. Điểm quan trọng phải nhớ là Interceptor không chạy khi có exception — lúc đó luồng đi thẳng sang Exception Filter — nên phần định dạng success và phần định dạng error là hai nơi tách biệt, không đụng nhau. Với endpoint phân trang thì ngoài data mình thêm một khối meta chứa nextCursor hoặc hasMore để client biết còn dữ liệu nữa không."

---

- Envelope đồng nhất giúp client xử lý predictable: success trả `{ message, data }`, lỗi trả shape cố định (`statusCode`, `message`, `error`, optional `code`).
- Dùng **Interceptor** bọc mọi response thành công, **ExceptionFilter** chuẩn hóa lỗi — controller chỉ cần `return data`.
- Với pagination thêm `meta: { nextCursor, hasMore }` bên cạnh `data`.
- Lưu ý: Interceptor **không** chạy khi có exception (luồng đi thẳng sang ExceptionFilter) — success-shape và error-shape xử lý ở hai nơi tách biệt.

```ts
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, { message: string; data: T }> {
  intercept(ctx: ExecutionContext, next: CallHandler<T>) {
    return next.handle().pipe(map((data) => ({ message: 'success', data })));
  }
}
```

</details>

<details>
<summary><b>23.7 Rate limiting và caching trong REST API — bạn thiết kế ra sao và dùng header nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Rate limiting là để chống lạm dụng và DoS, đảm bảo công bằng giữa các client. Khi một client vượt ngưỡng cho phép, mình trả 429 Too Many Requests kèm header Retry-After để báo khi nào được gọi lại, cùng nhóm X-RateLimit-Limit/Remaining/Reset để client biết hạn mức còn lại. Thuật toán phổ biến là token bucket hoặc sliding window; trong NestJS mình dùng @nestjs/throttler, và nếu chạy nhiều instance thì phải lưu bộ đếm chung ở Redis chứ không đếm trong bộ nhớ từng instance, nếu không mỗi instance đếm riêng sẽ cho qua quá hạn mức. Về caching, mình tận dụng HTTP caching qua header Cache-Control để client/CDN giữ lại dữ liệu, kèm conditional request bằng ETag và If-None-Match — nếu dữ liệu không đổi thì server trả 304 Not Modified với body rỗng, đỡ tốn băng thông. Ở phía server thì mình cache dữ liệu đọc nhiều ghi ít vào Redis, nhưng phải có chiến lược invalidation rõ ràng, tức xóa key khi resource thay đổi, nếu không sẽ trả dữ liệu cũ."

---

- Rate limiting chống lạm dụng/DoS, đảm bảo công bằng; vượt ngưỡng trả `429 Too Many Requests` kèm `Retry-After` + `X-RateLimit-Limit/Remaining/Reset`.
- Thuật toán: token bucket, sliding window. NestJS dùng `@nestjs/throttler`; chạy nhiều instance thì lưu counter ở Redis để đếm chung.
- HTTP caching: `Cache-Control` (`max-age`, `no-cache`, `private/public`) + conditional request `ETag`/`If-None-Match` -> `304 Not Modified` (không gửi lại body) khi không đổi.
- Server-side: Redis cache cho dữ liệu đọc nhiều/ghi ít, kèm invalidation rõ ràng (xóa key khi resource đổi) để tránh stale.

```http
GET /issues/42
If-None-Match: "v3-abc"
--- response ---
304 Not Modified            # body rong, dung lai cache cua client
```

</details>

<details>
<summary><b>23.8 Phân biệt PUT và PATCH, và làm sao xử lý cập nhật đồng thời (concurrent update / lost update)?</b></summary>

**📌 Câu trả lời mẫu:**

"PUT là thay thế toàn bộ resource, tức client phải gửi đủ mọi field, field nào thiếu coi như bị reset; còn PATCH chỉ cập nhật một phần, gửi field nào sửa field đó. Thực tế mình hay dùng PATCH cho update vì gọn và đúng ý 'chỉ đổi cái cần đổi'. Vấn đề đáng nói hơn là lost update khi có cập nhật đồng thời: hai người cùng mở một issue, cùng đọc một phiên bản, rồi người sau lưu đè mất thay đổi của người trước mà không ai hay. Cách mình xử lý là optimistic concurrency control — thêm một cột version cho bản ghi, khi ghi thì điều kiện update phải khớp đúng version mà client đã đọc, và mỗi lần ghi thành công thì tăng version lên. Nếu version đã lệch nghĩa là có người sửa trước rồi, update sẽ không trúng dòng nào, lúc đó mình ném 409 Conflict để báo client tải lại dữ liệu mới rồi thử lại. Cách này nhẹ hơn khóa pessimistic và rất hợp với web vì xung đột thực tế hiếm xảy ra."

---

- PUT thay thế **toàn bộ** resource (field thiếu bị set null/default tùy thiết kế), idempotent.
- PATCH cập nhật **một phần**, không nhất thiết idempotent (vd JSON Patch `add` vào mảng); thực tế thường dùng PATCH cho update.
- **Lost update**: hai client đọc cùng phiên bản rồi ghi đè nhau.
- Giải pháp: **optimistic concurrency control** — dùng cột `version` (hoặc `updatedAt`/`ETag`) làm điều kiện khi ghi; version lệch -> `409 Conflict` (hoặc `412 Precondition Failed` khi dùng `If-Match`).

```ts
const result = await prisma.issue.updateMany({
  where: { id, version: clientVersion },
  data: { title, version: { increment: 1 } },
});
if (result.count === 0) throw new ConflictException('Issue da bi thay doi, vui long tai lai');
```

</details>

---

## 24. Scheduled and Background Jobs

<details>
<summary><b>24.1 @nestjs/schedule hoạt động thế nào? Phân biệt @Cron, @Interval, @Timeout và vai trò của ScheduleModule.forRoot()</b></summary>

**📌 Câu trả lời mẫu:**

"@nestjs/schedule là module giúp mình chạy các tác vụ theo lịch ngay trong ứng dụng Nest. Khi mình gọi `ScheduleModule.forRoot()`, nó sẽ quét toàn bộ provider để tìm các method có gắn decorator lịch, đăng ký chúng vào một SchedulerRegistry và bắt đầu chạy khi app khởi động xong. Có ba decorator chính: @Cron chạy theo biểu thức cron, hợp cho việc định kỳ theo thời điểm cụ thể như 3 giờ sáng mỗi ngày dọn log; bên dưới nó dùng thư viện cron thật chứ không phải setInterval, nên hỗ trợ cả trường giây và cả timeZone. @Interval thì lặp lại sau mỗi N mili-giây, bản chất là setInterval. Còn @Timeout chỉ chạy đúng một lần sau N mili-giây, bản chất là setTimeout. Một lưu ý quan trọng là cả ba đều chạy trong cùng event loop của app, nên handler phải non-blocking và mình nên tự bọc try/catch — vì nếu để lỗi văng ra thì @Interval và @Timeout sẽ thành unhandled rejection, còn @Cron thì thư viện nuốt lỗi và lặng lẽ bỏ qua lần chạy đó, rất khó phát hiện."

---

- `ScheduleModule.forRoot()` khởi tạo `SchedulerRegistry`, quét provider tìm handler có decorator, đăng ký vào registry và chạy khi `onApplicationBootstrap`.
- `@Cron`: chạy theo biểu thức cron, dùng lớp `CronJob` của package `cron` (hỗ trợ 6 trường có giây và `timeZone`), KHÔNG phải `setInterval`.
- `@Interval`: lặp mỗi N ms (dưới là `setInterval`); `@Timeout`: chạy một lần sau N ms (dưới là `setTimeout`).
- Cả ba chạy trong cùng event loop nên handler phải non-blocking và tự `try/catch`: `@Interval`/`@Timeout` exception không bắt thành unhandled rejection; `@Cron` thì package `cron` nuốt lỗi và bỏ lần chạy.

```ts
@Cron('0 0 3 * * *', { name: 'log-retention', timeZone: 'Asia/Ho_Chi_Minh' })
async handle() {
  try { await this.logService.deleteOlderThan(30); }
  catch (err) { this.logger.error('log-retention failed', err); }
}
```

</details>

<details>
<summary><b>24.2 Vì sao background job phải idempotent và bạn đảm bảo idempotency như thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Idempotent nghĩa là dù job chạy một lần hay chạy lại nhiều lần thì kết quả cuối cùng vẫn như nhau. Điều này quan trọng vì trong thực tế một job rất dễ bị chạy lặp: nó có thể bị retry khi lỗi tạm thời, bị nhiều worker cùng nhận, hoặc app crash giữa chừng rồi job được đẩy lại vào queue. Nếu job không idempotent thì hậu quả rất nặng, ví dụ trừ tiền hai lần hay gửi cùng một email cho khách nhiều lần. Cách mình hay làm là tạo một idempotency key có ràng buộc unique trong DB, rồi dựa chính vào ràng buộc đó: nếu insert bị trùng key thì nghĩa là job đã chạy rồi, mình bỏ qua luôn thay vì làm lại. Với các tác vụ có side effect ra ngoài như gửi email hay thanh toán, mình thường ghi trạng thái PENDING trong transaction trước, gọi dịch vụ ngoài, rồi mới cập nhật thành SENT. Một lưu ý quan trọng là lời gọi ra dịch vụ ngoài không nên nằm trong DB transaction, vì email đã gửi đi thì không thể rollback được."

---

- Job có thể chạy lại do retry, nhiều worker cùng nhận, hoặc crash giữa chừng rồi re-queue; không idempotent gây double-charge, email trùng, sai dữ liệu.
- Cách phổ biến: idempotency key có ràng buộc `unique` trong DB, dùng chính ràng buộc làm điểm "check-then-act" (đọc-rồi-ghi vẫn race khi chạy song song).
- Tác vụ có side effect ngoài (email, thanh toán): ghi trạng thái PENDING trong transaction → gọi dịch vụ ngoài → cập nhật SENT.
- Lưu ý: gọi dịch vụ ngoài KHÔNG nằm trong DB transaction (không rollback được email đã gửi).

```ts
try {
  await this.prisma.otpSend.create({ data: { userId, key, status: 'PENDING' } });
} catch (e) {
  if (isUniqueViolation(e)) return; // da co -> bo qua, tranh gui trung
  throw e;
}
await this.resend.send(/* ... */);
await this.prisma.otpSend.update({ where: { userId_key: { userId, key } }, data: { status: 'SENT' } });
```

</details>

<details>
<summary><b>24.3 Khi nào dùng queue-based processing (Bull/BullMQ) thay vì @Cron? So sánh</b></summary>

**📌 Câu trả lời mẫu:**

"Mình phân biệt hai cái này theo bản chất công việc. `@Cron` hợp cho việc chạy định kỳ theo lịch, ví dụ mỗi đêm xoá log cũ hay tạo báo cáo; nó đơn giản, chạy ngay trong tiến trình app, nhưng nhược điểm là nếu app down đúng lúc đến lịch thì mất luôn lượt đó. Còn queue như BullMQ (chạy trên Redis) hợp cho việc theo sự kiện và những tác vụ nặng hoặc chậm, ví dụ gửi email, resize ảnh, gọi API bên thứ ba — mình đẩy job vào queue để không làm nghẽn request HTTP của user. Queue mạnh ở chỗ nó cho mình kiểm soát concurrency, có sẵn retry/backoff, rate limiting, ưu tiên job, theo dõi trạng thái, và quan trọng là độ bền: job nằm trong Redis nên restart app vẫn còn, không mất như `@Cron`. Trong thực tế mình hay kết hợp cả hai: dùng `@Cron` (hoặc repeatable job của BullMQ) để enqueue theo lịch, còn worker mới là nơi xử lý thật sự."

---

- `@Cron`: việc định kỳ theo lịch (retention, báo cáo), đơn giản, chạy in-process, mất lượt nếu app down.
- Queue (BullMQ trên Redis): việc theo sự kiện, nặng/chậm (email, resize ảnh, gọi API ngoài) mà không block request HTTP.
- Queue cho: concurrency control, retry/backoff, rate limiting, ưu tiên job, theo dõi trạng thái, và độ bền (job nằm trong Redis nên không mất khi restart).
- Thực tế kết hợp: `@Cron` (hoặc BullMQ repeatable) để enqueue theo lịch, worker xử lý thực sự.

```ts
await this.emailQueue.add('welcome', { userId }, {
  attempts: 3,
  backoff: { type: 'exponential', delay: 2000 },
  removeOnComplete: true,
  removeOnFail: 1000, // giu lai job loi de dieu tra
});
```

</details>

<details>
<summary><b>24.4 Retry và backoff trong xử lý job: bạn cấu hình thế nào và lưu ý gì?</b></summary>

**📌 Câu trả lời mẫu:**

"Khi xử lý job mình luôn cấu hình retry kèm exponential backoff, tức là lần thử lại đầu chờ 1 giây, rồi 2 giây, 4 giây tăng dần, và giới hạn số lần bằng `attempts` để không retry vô tận. Mình cũng thường thêm chút jitter (ngẫu nhiên hoá thời gian chờ) để tránh thundering herd, tức là khi nhiều job cùng fail rồi cùng retry một lúc làm dồn tải lên hệ thống. Nguyên tắc quan trọng là chỉ retry những lỗi có thể phục hồi như timeout, lỗi mạng, hay 503; còn lỗi nghiệp vụ như 400 hay 404 thì retry vô nghĩa nên dừng ngay — BullMQ có `UnrecoverableError` để fail luôn không thử lại. Khi hết số lần thử, BullMQ chuyển job vào tập `failed` để mình điều tra; lưu ý đây chưa phải Dead Letter Queue có sẵn, DLQ là pattern mình tự xây qua listener `failed`. Và một điều cốt lõi: retry chỉ thực sự an toàn khi job đã idempotent, không thì retry lại gây ra side effect trùng."

---

- Retry với exponential backoff (1s, 2s, 4s...) thêm jitter để tránh thundering herd, giới hạn số lần (`attempts`).
- Chỉ retry lỗi có thể phục hồi (timeout, 503, lỗi mạng); lỗi nghiệp vụ (400, 404) dừng ngay — BullMQ có `UnrecoverableError` để fail luôn.
- Hết lượt, BullMQ chuyển job vào tập `failed` (KHÔNG phải DLQ có sẵn — DLQ là pattern tự xây qua listener `failed`).
- Backoff `exponential` mặc định KHÔNG tự thêm jitter; muốn jitter phải dùng custom backoff. Retry chỉ an toàn khi job idempotent.

```ts
await queue.add('sync-issue', payload, {
  attempts: 5,
  backoff: { type: 'exponential', delay: 1000 }, // ~1s,2s,4s,8s,16s
});
// throw new UnrecoverableError('email format invalid'); // khong retry
```

</details>

---

## 25. Worked Examples for Core Concepts

<details>
<summary><b>25.1 Request lifecycle trong NestJS — thứ tự Guard → Pipe → Interceptor → Handler → Filter chạy như thế nào? Ảnh hưởng tới ký tự nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Một request trong NestJS đi qua các thành phần theo thứ tự cố định: Middleware, rồi Guard, rồi Interceptor (phần trước handler), rồi Pipe, tới Handler, sau đó Interceptor (phần sau handler), và nếu có lỗi thì Exception Filter bắt. Guard chạy đầu để lo authorization: nếu `canActivate()` trả về false thì request bị chặn ngay với 403 và Pipe lẫn Handler đều không chạy. Pipe chạy ngay trước handler để validate và transform dữ liệu, sai luật thì trả 400 và không vào handler. Interceptor thì bọc quanh handler nên can thiệp được cả hai chiều, hợp cho logging hay biến đổi response; còn Filter là nơi cuối cùng bắt mọi exception. Điểm mình thấy hay nhất và cũng là một bẫy cần nhớ là: vì Guard chạy trước Pipe, nên khi Guard chặn thì hệ thống không bao giờ phải xử lý dữ liệu của một user trái phép — đây là một điểm cộng về bảo mật. Trong Jira Clone mình dùng đúng trật tự này: `JwtAuthGuard` rồi `RolesGuard`, sau đó `ValidationPipe` mới kiểm tra DTO."

---

- Thứ tự: **Middleware → Guard → Interceptor (before) → Pipe → Handler → Interceptor (after) → Filter**.
- **Guard**: kiểm tra authorization; `canActivate()` false → 403, **không chạy Pipe/Handler**.
- **Pipe**: validate & transform; throw → 400, không chạy Handler.
- **Interceptor**: bọc request/response (logging, transform); **Filter**: bắt mọi exception.
- Bẫy quan trọng: Guard fail → Pipe không chạy → không bao giờ xử lý dữ liệu của user trái phép (tốt cho bảo mật).

```ts
@Controller('posts')
@UseGuards(JwtAuthGuard, RolesGuard)   // chạy lần lượt
@UseInterceptors(LoggingInterceptor)
export class PostsController {
  @Post() @Roles('ADMIN')
  createPost(
    @Body(new ValidationPipe({ whitelist: true })) dto: CreatePostDto,
    @CurrentUser() user,
  ) {
    return this.postsService.create(dto, user); // chạy khi Guard + Pipe đã qua
  }
}
// 1.Guard verify token & role → 2.Interceptor before → 3.Pipe validate dto → 4.Handler → 5.Interceptor after
```

</details>

<details>
<summary><b>25.2 JWT Refresh Token Rotation — vì sao cần? Cơ chế cấp lại token thế nào mà khó bị CSRF?</b></summary>

**📌 Câu trả lời mẫu:**

"Ý tưởng là tách làm hai loại token. Access token thì ngắn hạn, ví dụ 15 phút, gửi qua header `Authorization: Bearer`; còn refresh token thì dài hạn, ví dụ 7 ngày, và mình lưu trong cookie httpOnly để JavaScript phía client không đọc được, qua đó chống XSS. Cookie đó mình đặt thêm `secure` và `sameSite: 'strict'` để chống CSRF, vì trình duyệt sẽ không gửi kèm cookie này trong các request từ site khác. Phần rotation là điểm cốt lõi: mỗi lần client gọi `/auth/refresh`, server phát ra một cặp token mới và đồng thời vô hiệu hoá refresh token cũ. Lợi ích là nếu kẻ tấn công có chộp được refresh token thì nó chỉ dùng được đúng một lần; ngay khi token đó bị xài, request hợp lệ tiếp theo của người dùng thật với token cũ sẽ bị 401, nhờ vậy mình phát hiện được dấu hiệu bị xâm nhập. Nói ngắn gọn, access token ngắn hạn giảm cửa sổ rủi ro, còn rotation giúp một refresh token bị lộ không thể bị lạm dụng lâu dài."

---

- **Access token** ngắn hạn (15p) gửi qua header `Authorization: Bearer`; **Refresh token** dài hạn (7d) lưu **httpOnly cookie** (chống XSS).
- Cookie đặt `secure` + `sameSite: 'strict'` → chống CSRF.
- **Rotation**: mỗi lần `/auth/refresh`, server phát cặp token MỚI và **vô hiệu hóa token cũ**.
- Lợi ích: attacker chộp được refresh token chỉ dùng được 1 lần → request kế tiếp với token cũ → 401 → phát hiện xâm nhập.

```ts
@Post('refresh')
async refresh(@Req() req: Request, @Res() res: Response) {
  const old = req.cookies.refreshToken;
  const user = await this.authService.verifyRefreshToken(old);
  // rotation: phát mới + revoke cũ
  const { accessToken, refreshToken } = await this.authService.generateTokens(user, old);
  res.cookie('refreshToken', refreshToken, {
    httpOnly: true, secure: true, sameSite: 'strict',
    maxAge: 7 * 24 * 60 * 60 * 1000,
  });
  return res.json({ accessToken });
}
```

</details>

<details>
<summary><b>25.3 Exception Filter vs Global Exception Filter (@Catch) — khi nào dùng custom filter? Thứ tự xử lý nếu có nhiều filter?</b></summary>

**📌 Câu trả lời mẫu:**

"Exception Filter là nơi mình bắt các exception ném ra từ handler, service, pipe hay interceptor, rồi định hình lại response lỗi cho đẹp và nhất quán. Mình dùng custom filter trong vài trường hợp: khi từng loại exception cần một response khác nhau, khi cần log đặc biệt như đẩy lên Sentry hay ghi audit, hoặc khi gặp những lỗi không phải `HttpException` — ví dụ `QueryFailedError` từ DB — cần map sang đúng mã HTTP. Về thứ tự khi có nhiều filter: filter cụ thể như `@Catch(QueryFailedError)` nên đặt trước, rồi tới `@Catch(HttpException)` ở giữa, và cuối cùng là `@Catch()` không tham số làm fallback bắt mọi thứ còn lại. Khi một exception khớp với loại nào thì đúng filter đó xử lý và gửi response, các filter khác không chạy nữa. Ví dụ thực tế mình hay làm là bắt lỗi unique constraint của Postgres (mã 23505) rồi trả về 409 thay vì để lộ ra 500 chung chung — vừa rõ nghĩa cho client vừa không lộ chi tiết nội bộ."

---

- Filter bắt exception từ handler/service/pipe/interceptor và định hình response.
- Dùng **custom filter** khi: loại exception cần response khác nhau, cần log đặc biệt (Sentry/audit), hoặc exception không phải `HttpException` (vd `QueryFailedError`) cần map sang HTTP.
- Thứ tự: filter cụ thể (`@Catch(QueryFailedError)`) trước, `@Catch(HttpException)` giữa, `@Catch()` (fallback) **sau cùng**.
- Exception match loại nào → filter đó xử lý + gửi response → các filter khác không chạy.

```ts
@Catch(QueryFailedError)
export class DatabaseExceptionFilter implements ExceptionFilter {
  catch(ex: QueryFailedError, host: ArgumentsHost) {
    const res = host.switchToHttp().getResponse();
    const { code } = ex.driverError;
    if (code === '23505') return res.status(409).json({ message: 'Resource already exists' });
    if (code === '23503') return res.status(400).json({ message: 'Referenced resource not found' });
    return res.status(500).json({ message: 'Internal server error' });
  }
}

// app.module: thứ tự cụ thể → chung → fallback
providers: [
  { provide: APP_FILTER, useClass: DatabaseExceptionFilter },
  { provide: APP_FILTER, useClass: HttpExceptionFilter },   // @Catch(HttpException)
  { provide: APP_FILTER, useClass: GlobalExceptionFilter },  // @Catch() fallback
];
```

</details>

---

*Cập nhật lần cuối: bổ sung §14–§25 (Prisma, Advanced Auth, Web Security, Observability, Advanced NestJS, Transactions & Concurrency, Advanced Node, TypeScript, Testing, System Design, API/REST, Scheduled Jobs, Worked Examples) — bám sát dự án Jira Clone. File này được bổ sung sau mỗi section mới.*
---

## 26. GraphQL và Realtime trong NestJS

<details>
<summary><b>26.1 @WebSocketGateway và @SubscribeMessage trong NestJS hoạt động ra sao? Vòng đời kết nối?</b></summary>

**📌 Câu trả lời mẫu:**

"NestJS có hỗ trợ WebSocket riêng, tách biệt với GraphQL subscription, và mặc định chạy trên Socket.IO. Để tạo một WebSocket server, mình đánh dấu một class bằng `@WebSocketGateway`, ở đó có thể cấu hình namespace hay CORS; dùng `@WebSocketServer()` để inject server vào nếu muốn chủ động broadcast; còn `@SubscribeMessage('event')` thì giống như một route handler nhưng cho message mà client emit lên, ví dụ sự kiện 'joinIssue'. Về vòng đời kết nối, gateway có các hook tương ứng: `afterInit` khi gateway khởi tạo xong, `handleConnection` khi một client kết nối vào, và `handleDisconnect` khi client ngắt — mình thường dùng `handleConnection` để xác thực token và `handleDisconnect` để dọn dẹp. Một tính năng rất hữu ích là room: client có thể `join` vào một room, rồi server `emit` tới đúng room đó, nhờ vậy mình phát sự kiện chọn lọc thay vì broadcast cho tất cả. Trong Jira Clone, mình cho mỗi issue hoặc board một room riêng, để chỉ những người đang xem issue đó mới nhận được cập nhật real-time."

---

- NestJS có WebSocket riêng (độc lập GraphQL subscription), mặc định **Socket.IO**.
- `@WebSocketGateway(opts)`: đánh dấu class là gateway (đặt `namespace`, `cors`); `@WebSocketServer()`: inject server để broadcast; `@SubscribeMessage('event')`: handler cho message client emit.
- Lifecycle: `OnGatewayInit` (`afterInit`), `OnGatewayConnection` (`handleConnection`), `OnGatewayDisconnect` (`handleDisconnect`).
- Hữu ích: **room** (`client.join` / `server.to(room).emit`) cho phép phát chọn lọc — chỉ client quan tâm 1 issue/board nhận event, không broadcast toàn bộ.

```ts
@SubscribeMessage('joinIssue')
onJoin(@ConnectedSocket() client: Socket, @MessageBody() issueId: string) {
  client.join(`issue:${issueId}`); // room theo issue
}
```

</details>

<details>
<summary><b>26.2 Khi nào chọn REST, GraphQL hay WebSocket? Tiêu chí quyết định?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình chọn theo bản chất nhu cầu chứ không theo cái nào 'hiện đại' hơn. REST hợp cho CRUD và public API đơn giản, đặc biệt khi mình muốn tận dụng HTTP cache và CDN, cùng các status code rõ ràng; nhược điểm là dễ bị over-fetch hoặc under-fetch và đôi khi phải gọi nhiều lần mới đủ dữ liệu. GraphQL hợp khi client — nhất là mobile hay nhiều màn hình khác nhau — cần lấy đúng tập field và dữ liệu có nhiều quan hệ lồng nhau, giúp giảm số lần gọi; đổi lại mình phải lo vấn đề N+1, giới hạn độ phức tạp của query và việc cache khó hơn vì chỉ có một endpoint. WebSocket thì dành cho real-time hai chiều độ trễ thấp như chat, notification hay live board, nhưng phải quản lý kết nối, scale (dùng Redis adapter) và auth làm theo kiểu khác. Thực tế một hệ thống thường dùng cả ba cho từng phần phù hợp. Mình cũng lưu ý là nếu chỉ cần server đẩy cập nhật thưa và một chiều thì SSE hoặc polling còn đơn giản hơn WebSocket, không nhất thiết phải mở kết nối hai chiều."

---

- **REST**: CRUD, public API đơn giản, cần tận dụng HTTP cache/CDN và status code rõ; rủi ro over/under-fetch, nhiều round-trip.
- **GraphQL**: client (mobile/đa màn hình) cần lấy đúng tập field, dữ liệu nhiều quan hệ lồng nhau, giảm round-trip; đổi lại phải xử lý N+1, giới hạn complexity, khó cache.
- **WebSocket**: real-time/hai chiều độ trễ thấp (chat, notification, live board); rủi ro quản lý kết nối, scale (Redis adapter), auth khác kiểu.
- Thực tế dùng cả ba; cập nhật thưa có thể dùng SSE/polling cho đơn giản.

| | REST | GraphQL | WebSocket |
|---|---|---|---|
| Mô hình | Req/res, resource | Req/res, chọn field | Hai chiều, server đẩy |
| Caching | Dễ (CDN) | Khó (1 endpoint) | Không áp dụng |

</details>

---

*Cập nhật lần cuối: bổ sung §26 (GraphQL và Realtime trong NestJS — code-first vs schema-first, resolver/@ResolveField, N+1 & DataLoader, subscription, WebSocket Gateway/Socket.IO, Redis adapter, REST vs GraphQL vs WebSocket, bảo mật GraphQL) — bám sát dự án Jira Clone. File này được bổ sung sau mỗi section mới.*
---

## 27. Tư duy và Giải quyết vấn đề (Backend)

<details>
<summary><b>27.1 N+1 query là gì, vì sao nó "ẩn" cho tới khi data lớn, và cách phát hiện/khắc phục với Prisma, TypeORM và GraphQL?</b></summary>

**📌 Câu trả lời mẫu:**

"N+1 query là khi mình chạy một query để lấy N bản ghi, rồi lặp qua từng cái và chạy thêm một query con để lấy quan hệ của nó, kết quả là 1 cộng N round-trip xuống DB. Nó hay 'ẩn' vì vấn đề tỉ lệ thuận với lượng dữ liệu: lúc dev mình chỉ có 5 bản ghi nên vẫn nhanh, không ai để ý; nhưng lên production với 5000 bản ghi thì nó thành 5001 query, lúc đó mới chậm rõ. Cách phát hiện đơn giản là bật query log rồi đếm số query trên mỗi request — nếu thấy cùng một mẫu query lặp lại nhiều lần thì gần như chắc chắn là N+1. Cách khắc phục là lấy hết trong một lần thay vì query lẻ trong vòng lặp: với Prisma thì dùng `include`, với TypeORM thì `relations` hoặc `leftJoinAndSelect`. Riêng GraphQL thì N+1 sinh ra ở field resolver chạy cho từng phần tử của list, nên mình dùng DataLoader để gom các key trong cùng một tick rồi batch thành một query dạng `WHERE id IN (...)` và cache trong vòng đời request. Đánh đổi cần nhớ là eager load dễ over-fetch, nên mình chỉ include đúng quan hệ thực sự cần cho response đó."

---

- **N+1**: 1 query lấy N bản ghi rồi chạy thêm N query con cho quan hệ từng bản ghi → N+1 round-trip. Ẩn vì vấn đề tỉ lệ với kích thước data: dev 5 record vẫn nhanh, prod 5000 record thành 5001 query.
- **Phát hiện**: bật query log, đếm query/request; thấy cùng một mẫu query lặp nhiều lần → gần như chắc chắn N+1.

```ts
// XẤU: N+1
for (const i of issues) i.author = await prisma.user.findUnique({ where: { id: i.authorId } });
// TỐT (Prisma): một query với include
const issues = await prisma.issue.findMany({ where: { projectId }, include: { author: true } });
```

- **TypeORM**: `relations: ['author']` hoặc `leftJoinAndSelect` thay vì lazy access trong vòng lặp.
- **GraphQL**: N+1 sinh ở **field resolver** chạy từng phần tử list → dùng **DataLoader** gom key trong cùng tick, batch thành 1 query `WHERE id IN (...)`, cache trong vòng đời request (nhớ trả về đúng thứ tự key).
- **Trade-off**: eager load có thể over-fetch → chỉ `include`/`select` đúng cái cần.

</details>

<details>
<summary><b>27.2 Khi đứng trước một quyết định kiến trúc, bạn cân nhắc trade-off theo khung tư duy nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Quan điểm của mình là không có kiến trúc nào đúng tuyệt đối, chỉ có cái phù hợp với ràng buộc hiện tại. Khung tư duy mình hay dùng gồm mấy bước: đầu tiên làm rõ cả yêu cầu functional lẫn non-functional như throughput, latency, độ sẵn sàng, mức nhất quán, ngân sách và quy mô team — phần lớn quyết định kiến trúc thực ra bị các yếu tố non-functional chi phối. Sau đó mình liệt kê 2-3 phương án, rồi đánh giá theo các trục như độ phức tạp vận hành, chi phí, khả năng scale, tốc độ ship, và đặc biệt là khả năng đảo ngược: quyết định nào dễ đảo thì làm nhanh, còn loại 'one-way door' như chọn DB chính hay thiết kế API public thì phải cẩn trọng. Cuối cùng mình ghi lại lý do bằng một ADR để sau này team hiểu vì sao chọn vậy. Mình cũng luôn nhớ CAP và PACELC — mọi cache, replica hay queue đều mua hiệu năng bằng cái giá là độ nhất quán hoặc độ phức tạp — và tránh hai anti-pattern là chọn công nghệ vì 'cho ngầu' và tối ưu quá sớm cho quy mô chưa tồn tại. Ví dụ với một app giai đoạn đầu vài nghìn user thì một modular monolith NestJS cộng Postgres và Redis là đủ, tách microservice lúc đó chỉ thêm phức tạp."

---

- Không có kiến trúc "đúng tuyệt đối", chỉ có cái **phù hợp ràng buộc hiện tại**.
- Khung: (1) làm rõ yêu cầu **functional + non-functional** (throughput, latency, availability, consistency, ngân sách, quy mô team — phần lớn quyết định bị non-functional chi phối); (2) liệt kê **2-3 phương án**; (3) đánh giá theo trục: phức tạp vận hành, chi phí, scale, tốc độ ship, **khả năng đảo ngược**; (4) quyết định dễ đảo thì làm nhanh, **one-way door** (DB chính, API public, phân chia service) thì cẩn trọng; (5) ghi lại bằng **ADR**.
- Định luật nền tảng: **CAP** (khi partition, chọn C hay A) và **PACELC** (kể cả không partition, vẫn trade Latency vs Consistency). Mọi cache/replica/queue đều mua hiệu năng bằng giá nhất quán/phức tạp.
- **Anti-pattern**: resume-driven development (chọn Kafka/k8s vì "ngầu") và premature optimization cho quy mô chưa có.
- Ví dụ: Blog/CMS giai đoạn đầu (vài nghìn user) → **modular monolith** NestJS + Postgres + Redis là đủ; tách microservice chỉ thêm phức tạp.

</details>

<details>
<summary><b>27.3 Khi nào nên tách một microservice ra khỏi monolith, và khi nào KHÔNG nên?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình coi microservice là cách giải bài toán về tổ chức và quy mô, chứ không phải mục tiêu để theo trend. Bản chất là mình đánh đổi sự đơn giản của monolith (một lần deploy, một transaction, gọi hàm in-process) để lấy khả năng scale và phát triển độc lập, nhưng phải trả 'thuế phân tán': mạng không đáng tin, dữ liệu chỉ eventual consistency, debug và tracing khó hơn nhiều. Mình nên tách khi một phần có nhu cầu scale rất khác (ví dụ xử lý ảnh/video ngốn CPU), khi ranh giới team đã rõ và muốn deploy riêng, hoặc cần cô lập rủi ro như module payment. Ngược lại, mình KHÔNG nên tách khi team còn nhỏ, sản phẩm chưa ổn định, hoặc domain boundary chưa rõ — tách sớm thì cứ phải vẽ lại đường biên liên tục; lúc đó một modular monolith là đủ. Trong Jira Clone, mình sẽ đi theo lộ trình monolith rồi modular monolith, chỉ tách khi thật sự đau, và bước trung gian an toàn là đẩy việc nặng async ra worker BullMQ. Cái bẫy lớn nhất mình luôn tránh là distributed monolith — tách service ra nhưng vẫn gọi đồng bộ chằng chịt và share chung database, nhận đủ nhược điểm của cả hai mô hình."

---

- **Microservice là giải pháp cho vấn đề tổ chức/quy mô, không phải mục tiêu**. Đánh đổi sự đơn giản (1 deploy, 1 transaction, gọi in-process) lấy scale/phát triển độc lập — kèm *thuế phân tán* (network không tin cậy, eventual consistency, tracing, deploy phức tạp).
- **Nên tách khi**: một phần có **profile mở rộng khác biệt** (vd xử lý ảnh/video ngốn CPU); **ranh giới team rõ** muốn deploy độc lập (Conway's Law); cần **cô lập lỗi/bảo mật** (vd payment) hoặc khác stack/tuân thủ.
- **KHÔNG nên** (giữ **modular monolith**): team nhỏ, chưa có product-market fit, domain boundary chưa rõ (tách sớm phải sửa biên giới liên tục); nhu cầu chỉ là "code sạch hơn" (giải bằng module hóa).
- Lộ trình: **Monolith → Modular Monolith → tách dần khi đau thật**. NestJS có `@nestjs/microservices` (TCP/Redis/NATS/RabbitMQ/Kafka); bước trung gian an toàn là tách tác vụ async nặng ra **worker BullMQ**.
- **Bẫy lớn nhất**: **distributed monolith** — tách service nhưng gọi đồng bộ chằng chịt và share database, nhận đủ nhược điểm cả hai.

</details>

<details>
<summary><b>27.4 Bạn nhận một yêu cầu (ticket) mơ hồ, ví dụ "thêm export báo cáo cho admin". Bạn làm gì trước khi viết code?</b></summary>

**📌 Câu trả lời mẫu:**

"Nguyên tắc của mình là làm rõ trước, code sau, vì viết code dựa trên một yêu cầu mơ hồ rất dễ build nhầm rồi phải làm lại. Đầu tiên mình hỏi 'what' và 'why': ai sẽ dùng tính năng export này và để giải quyết vấn đề gì — họ muốn tự phân tích bằng Excel hay gửi cho khách? Sau đó mình đào các yêu cầu phi chức năng: định dạng và phạm vi (CSV/XLSX/PDF, cần những cột nào, lọc theo gì), và đặc biệt là khối lượng dữ liệu — vài trăm dòng thì xuất đồng bộ trong request là được, còn hàng triệu dòng thì mình phải đẩy ra một background job (BullMQ), stream ra file rồi lưu storage và gửi link, tránh giữ HTTP connection mở quá lâu và ngốn RAM. Mình cũng làm rõ quyền hạn (role nào được export, có dính PII cần audit không) và các edge case như dữ liệu rỗng hay timezone, rồi chốt acceptance criteria trước khi ước lượng. Một mẹo giao tiếp mình hay dùng là không hỏi mở mà đề xuất một giả định cụ thể để xác nhận, kiểu 'Em hiểu là export CSV các issue đã đóng trong tháng, lọc theo project, chạy nền rồi gửi link đúng không ạ?' — câu hỏi đóng giúp lấy quyết định nhanh hơn. Trong Jira Clone, có lần 'export report' hóa ra là báo cáo velocity theo sprint chứ không phải xuất raw toàn bộ issue, nếu không hỏi kỹ thì mình đã làm sai hướng."

---

- Nguyên tắc: **làm rõ trước, code sau** — đặt câu hỏi có cấu trúc.
- (1) Làm rõ **"what" & "why"**: ai dùng, giải quyết vấn đề gì (export để tự phân tích Excel hay gửi khách?); (2) đào **non-functional**: định dạng & phạm vi (CSV/XLSX/PDF, cột, lọc), **khối lượng** (vài trăm dòng → xuất đồng bộ; hàng triệu → job BullMQ + **stream** ra file + lưu storage rồi gửi link, tránh giữ HTTP mở lâu & ngốn RAM), quyền hạn (role/tenant), PII/audit; (3) xác định **edge case** (rỗng, timezone, đồng thời); (4) chốt **acceptance criteria** rồi ước lượng, đề xuất chia phase nếu rộng.
- **Mẹo giao tiếp**: thay vì hỏi mở, **đề xuất giả định cụ thể để xác nhận**: "Em hiểu là export CSV issue đã đóng trong tháng, lọc theo project, chạy nền gửi link — đúng không ạ?" (câu đóng lấy quyết định nhanh hơn).
- Ví dụ: Jira Clone "export report" hóa ra là báo cáo velocity theo sprint — không hỏi dễ build nhầm thành export raw toàn bộ issue.

</details>

<details>
<summary><b>27.5 Bạn được giao maintain một codebase backend lớn, lạ, không có tài liệu. Bạn đọc hiểu nó như thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Khi nhận một codebase lớn lạ không có tài liệu, mình không đọc tuần tự từng file mà đi từ ngoài vào trong, từ luồng thực tế tới chi tiết. Việc đầu tiên là dựng nó chạy được ở local và gọi thử vài endpoint, vì hiểu được cách build/test/run thường rõ ràng hơn mọi tài liệu. Tiếp theo mình tìm các entry point — với NestJS thì là `main.ts`, `AppModule` và cây module/import, vì cây module chính là sơ đồ kiến trúc của app; rồi mình trace một luồng end-to-end từ controller xuống service, repository tới response, đặt breakpoint hoặc log để thấy dữ liệu chảy thế nào. Mình đọc kỹ schema DB (với dự án dùng Prisma thì xem `schema.prisma`) vì mô hình dữ liệu tiết lộ domain rõ hơn cả code, và mình coi test như tài liệu sống vì nó mô tả hành vi mong đợi. Mình cũng dùng `git log`/`blame` để hiểu vì sao một đoạn code tồn tại và ai là người nên hỏi, rồi mới hỏi đúng người những câu cụ thể sau khi đã tự tìm hiểu. Nguyên tắc bao trùm của mình là học just-in-time — chỉ đào sâu phần liên quan tới task đang làm — và thực hiện một thay đổi nhỏ, an toàn đầu tiên để xác nhận mình đã hiểu đúng vòng đời dev."

---

- Không đọc tuần tự từng file — đi từ **ngoài vào trong, từ luồng thực tới chi tiết**.
- (1) **Chạy được nó trước** (dựng local, gọi vài endpoint — hiểu build/test/run rõ hơn mọi tài liệu); (2) tìm **entry points** (NestJS: `main.ts`, `AppModule`, `controllers`, `imports` — cây module là sơ đồ kiến trúc); (3) **trace một luồng end-to-end** controller → service → repo → response (breakpoint/log); (4) đọc **schema DB** (Prisma/entity — mô hình dữ liệu tiết lộ domain rõ hơn code); (5) dùng **test như tài liệu sống** (mô tả hành vi & cách dùng API); (6) khảo sát **git log/blame** để biết *vì sao* và ai nên hỏi; (7) **hỏi đúng người câu cụ thể** sau khi đã tự tìm hiểu.
- Nguyên tắc bao trùm: **học just-in-time** (chỉ đào sâu phần liên quan task), ghi chú khi hiểu một mảnh, để CI/test làm lưới an toàn; một thay đổi nhỏ-an-toàn đầu tiên giúp xác nhận đã hiểu vòng đời dev.

</details>

## 28. MongoDB và Mongoose (NoSQL)

<details>
<summary><b>28.1 Khi nào bạn chọn MongoDB thay vì PostgreSQL? Cho ví dụ document model vs relational model.</b></summary>

**📌 Câu trả lời mẫu:**

"Mình không nghĩ theo kiểu 'NoSQL tốt hơn SQL', mà chọn dựa trên việc mô hình dữ liệu nào khớp với access pattern của ứng dụng. MongoDB lưu dữ liệu dạng document (BSON), schema linh hoạt, rất hợp khi mình đọc/ghi xoay quanh một aggregate tự chứa và muốn scale ngang bằng sharding — ví dụ giỏ hàng, log, hay nội dung CMS, mỗi bài viết có thể nhét luôn author và comments vào trong một document. PostgreSQL thì mạnh ở ràng buộc toàn vẹn (foreign key, constraint), JOIN nhiều bảng, và transaction ACID chặt chẽ. Nên mình chọn MongoDB khi dữ liệu tự chứa theo một khóa, schema hay thay đổi, hoặc cần write throughput cao; còn chọn PostgreSQL khi có nhiều quan hệ nhiều-nhiều phức tạp, cần báo cáo JOIN tùy biến, hoặc bắt buộc toàn vẹn giao dịch như tài chính, đặt vé. Với Jira Clone mình chọn PostgreSQL vì dữ liệu rất quan hệ — user, project, issue, comment liên kết chằng chịt và cần toàn vẹn — nên một relational DB cùng Prisma là lựa chọn tự nhiên, chứ một dữ liệu kiểu activity log đơn lẻ thì document model lại tiện hơn."

---

- Không phải "NoSQL tốt hơn SQL" mà là **mô hình dữ liệu khớp với access pattern**.
- **MongoDB (document/BSON)**: schema linh hoạt, đọc/ghi theo một aggregate root, scale ngang bằng sharding tốt.
- **PostgreSQL (relational)**: mạnh ở ràng buộc toàn vẹn (FK/constraint), JOIN nhiều bảng, transaction ACID chặt.
- Chọn **MongoDB** khi dữ liệu tự chứa theo một khoá (giỏ hàng, log, CMS content, catalog), schema hay đổi, cần write throughput cao.
- Chọn **PostgreSQL** khi quan hệ nhiều-nhiều phức tạp, cần báo cáo JOIN tuỳ biến, toàn vẹn giao dịch bắt buộc (tài chính, đặt vé).

```js
// MongoDB: một bài viết là MỘT document tự chứa
{
  _id: ObjectId("..."),
  title: "NoSQL vs SQL",
  author: { id: "u1", name: "Apollo" },              // embed thông tin hiển thị
  comments: [{ user: "u2", text: "Hay!" }]
}
```

</details>

---

## 29. Git Workflow và Cộng tác

<details>
<summary><b>29.1 So sánh GitFlow, GitHub Flow và Trunk-Based Development? Khi nào chọn mô hình nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Ba mô hình này khác nhau chủ yếu ở số lượng branch sống lâu và chu kỳ release. GitFlow có nhiều long-lived branch như `main`, `develop`, `release/*`, `hotfix/*`, hợp với sản phẩm phát hành theo version và phải maintain nhiều bản song song; nhưng nó khá nặng nề với một web service deploy liên tục. GitHub Flow đơn giản hơn nhiều: chỉ có `main` cộng các feature branch ngắn, mỗi PR merge xong là deploy được luôn, nên nó hợp với đa số web app hiện đại. Trunk-Based thì branch sống cực ngắn, mọi người tích hợp liên tục về `main`, code chưa hoàn thiện thì ẩn sau feature flag — cách này giảm được 'merge hell' nhưng đòi hỏi test tự động và CI rất mạnh để an toàn. Quan điểm của mình là chọn theo chu kỳ release chứ không theo trend: deploy hằng ngày thì dùng GitHub Flow hoặc Trunk-Based, còn nếu phải đóng gói và hỗ trợ nhiều version thì GitFlow mới đáng. Với một dự án như Jira Clone deploy thường xuyên, mình thấy GitHub Flow là phù hợp nhất vì nó vừa đơn giản vừa đủ kỷ luật."

---

- **GitFlow**: nhiều long-lived branch (`main`, `develop`, `release/*`, `hotfix/*`); hợp sản phẩm phát hành theo version, nhiều bản song song. Nặng nề với web service deploy liên tục.
- **GitHub Flow**: chỉ `main` + feature branch ngắn, mỗi PR merge xong là deploy được. Đơn giản, hợp đa số web app.
- **Trunk-Based**: branch sống cực ngắn, tích hợp liên tục về `main`, code chưa xong ẩn sau **feature flag**. Giảm "merge hell" nhưng cần test tự động + CI mạnh.
- Chọn theo **chu kỳ release**, không theo trend: deploy hằng ngày → GitHub Flow/Trunk-Based; đóng gói nhiều version → GitFlow.

```bash
# GitHub Flow: branch ngắn từ main -> PR -> merge -> xoá
git switch -c feat/issue-comments
git push -u origin feat/issue-comments
```

</details>

<details>
<summary><b>29.2 Phân biệt merge, rebase và squash merge? Khi nào dùng cái nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Ba cái này khác nhau ở chỗ chúng định hình lịch sử commit thế nào. Merge commit (`--no-ff`) giữ nguyên toàn bộ commit của feature branch và thêm một merge commit nối lại — lịch sử trung thực nhất nhưng đồ thị rẽ nhánh nhằng nhịt khi nhìn `git log --graph`. Rebase thì phát lại các commit của mình lên đỉnh `main` mới nhất, cho lịch sử thẳng tuyến tính, nhưng nó viết lại hash nên mình tuyệt đối không rebase nhánh người khác đã pull về. Squash merge gộp cả PR thành đúng một commit trên `main`, giúp lịch sử main siêu sạch và revert dễ, đổi lại mất chi tiết từng bước. Trong nhóm làm Jira Clone, quy ước của mình là: dùng rebase để cập nhật branch riêng cho khỏi tụt hậu so với main; dùng squash merge khi gộp PR vào main để mỗi feature là một dòng lịch sử gọn gàng; còn merge `--no-ff` mình để dành cho nhánh release khi cần giữ trọn vết để audit. Một lưu ý mình luôn nhắc: sau rebase phải push bằng `--force-with-lease` chứ không phải `--force`, vì nó sẽ từ chối nếu remote có commit mới mình chưa thấy, tránh ghi đè nhầm lên việc của đồng đội."

---

- **Merge commit** (`--no-ff`): giữ trọn commit feature + 1 merge commit; lịch sử trung thực nhưng đồ thị rẽ nhánh nhiều.
- **Rebase**: phát lại commit lên đỉnh `main`, lịch sử thẳng tuyến tính; **viết lại hash** → không rebase nhánh người khác đã pull.
- **Squash merge**: gộp cả PR thành 1 commit trên `main`; `main` sạch, dễ revert, nhưng mất lịch sử chi tiết.
- Quy ước: **rebase** để cập nhật branch riêng; **squash merge** khi gộp PR; **merge `--no-ff`** khi cần giữ trọn lịch sử release/audit.

```bash
git switch feat/auth
git rebase origin/main
git push --force-with-lease   # AN TOÀN nếu đã push trước đó
```

</details>

<details>
<summary><b>29.3 Quy trình giải quyết merge conflict ra sao? Conflict thường phát sinh từ đâu?</b></summary>

**📌 Câu trả lời mẫu:**

"Merge conflict xảy ra khi hai phía cùng sửa một vùng dòng trong cùng một file, hoặc một bên xóa trong khi bên kia sửa — lúc đó Git không biết giữ phiên bản nào nên đánh dấu vùng xung đột bằng `<<<<<<<`, `=======` và `>>>>>>>` rồi nhờ mình quyết định. Quy trình của mình là `git fetch` để lấy code mới nhất, rebase hoặc merge `main` vào, rồi mở từng file bị conflict, đọc hiểu cả hai phía và hòa lại cho đúng ý nghĩa nghiệp vụ, xóa hết marker đi. Sau đó mình `git add` các file đã giải quyết, chạy `git rebase --continue` (hoặc `git commit` nếu là merge), và quan trọng nhất là chạy lại test/build để chắc chắn việc hòa code không làm vỡ logic. Để giảm conflict ngay từ đầu thì mình giữ branch ngắn, merge `main` thường xuyên cho khỏi tụt hậu, chia module rõ ràng để hai người ít đụng cùng file. Với mấy file đặc biệt như lock file thì mình không gộp tay mà lấy hẳn một phía rồi cài lại dependency, và nếu rebase rối quá thì `git rebase --abort` để quay về trạng thái an toàn trước đó."

---

- Conflict xảy ra khi **hai phía cùng sửa một vùng dòng** (hoặc một bên xoá, một bên sửa); Git đánh dấu bằng `<<<<<<<`, `=======`, `>>>>>>>`.
- Quy trình: `git fetch` → rebase/merge `main` → mở file hoà hai phía, **xoá hết marker** → `git add` → `git rebase --continue` (hoặc `git commit`) → **chạy lại test/build**.
- Giảm conflict: branch ngắn, merge thường xuyên, chia module rõ, dùng `git rerere`. Với lock file thì lấy một phía rồi cài lại, đừng gộp tay.

```bash
git rebase origin/main
# sửa file, xoá marker
git add src/issue/issue.service.ts
git rebase --continue
git rebase --abort     # nếu rối, quay về trước rebase
```

</details>

<details>
<summary><b>29.4 Conventional Commits là gì và nó liên kết với Semantic Versioning như thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Conventional Commits là một quy ước viết commit theo cấu trúc `type(scope): subject`, ví dụ `feat`, `fix`, `docs`, `refactor`, `chore`. Lợi ích là lịch sử commit đọc rất dễ, có thể tự sinh CHANGELOG và đặc biệt là tự suy ra version một cách máy móc. Nó liên kết với Semantic Versioning (`MAJOR.MINOR.PATCH`) theo quy tắc rõ ràng: commit `fix:` làm tăng PATCH (sửa lỗi tương thích ngược), `feat:` làm tăng MINOR (thêm tính năng mới mà không phá vỡ gì), còn nếu có `BREAKING CHANGE:` trong body hoặc dấu `!` sau type thì tăng MAJOR vì có thay đổi phá vỡ tương thích. Nhờ vậy mình dùng được công cụ như `semantic-release` để CI tự đọc commit, tự bump version, tạo git tag và CHANGELOG mà không phải sửa `package.json` bằng tay. Trong Jira Clone mình áp quy ước này để mỗi PR vào main có message chuẩn, việc release trở nên tự động và minh bạch."

---

- **Conventional Commits**: commit theo `type(scope): subject` (`feat`, `fix`, `docs`, `refactor`, `chore`...). Lợi ích: lịch sử đọc được, tự sinh CHANGELOG, tự suy ra version.
- Ánh xạ sang **SemVer** `MAJOR.MINOR.PATCH`:
  - `fix:` → **PATCH**
  - `feat:` → **MINOR**
  - `BREAKING CHANGE:` / dấu `!` → **MAJOR**
- `semantic-release`/`standard-version` đọc commit để tự bump version, tạo tag + CHANGELOG trong CI — không sửa `package.json` tay.

```bash
feat(issue): add bulk status update     # MINOR
fix(auth): reject expired refresh token # PATCH
feat(api)!: rename /tasks to /issues     # MAJOR
```

</details>

<details>
<summary><b>29.5 Code review nên tập trung vào những gì? Vì sao PR nhỏ lại tốt hơn PR lớn?</b></summary>

**📌 Câu trả lời mẫu:**

"Khi review code, mình để chuyện format và code style cho linter với Prettier lo, còn reviewer thì tập trung vào những thứ máy không kiểm được. Quan trọng nhất là correctness — logic có đúng không, đã xử lý edge case và lỗi chưa, có race condition không; rồi tới bảo mật — có lỗ hổng injection, thiếu kiểm tra quyền (authorization), lộ secret hay log nhầm dữ liệu nhạy cảm không. Mình cũng nhìn thiết kế và khả năng bảo trì như đặt code đúng layer, tránh trùng lặp, đặt tên rõ ràng, cộng với việc có test đi kèm và liệu PR có làm breaking change hợp đồng API hay không. Mình ưu tiên PR nhỏ, tầm dưới 300 dòng, vì PR nhỏ thì review được kỹ nên bắt bug tốt hơn, merge nhanh, ít conflict và dễ revert; còn PR lớn dễ bị 'LGTM' cho qua mà không ai đọc kỹ, lại hay treo lâu. Nguyên tắc của mình là một PR chỉ làm một việc, và tách phần refactor ra khỏi phần thêm feature để review tập trung hơn."

---

- Format để cho linter/Prettier; reviewer tập trung thứ máy không kiểm được:
  - **Correctness**: logic, edge case, xử lý lỗi, race condition.
  - **Bảo mật**: injection, thiếu authz, lộ secret, log dữ liệu nhạy cảm.
  - **Thiết kế/bảo trì**: đúng layer, tránh trùng lặp, đặt tên rõ.
  - **Test** và **hợp đồng API** (breaking change).
- **PR nhỏ (<~300 dòng)**: review kỹ bắt được bug, merge nhanh, ít conflict, dễ revert; PR lớn dễ bị "LGTM" cho qua, treo lâu.
- Nguyên tắc: 1 PR làm **một việc**; tách refactor và feature riêng.

</details>

<details>
<summary><b>29.6 Phân biệt git revert và git reset? Trên branch chung phải dùng cái nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Khác biệt cốt lõi là `git revert` thì giữ nguyên lịch sử, còn `git reset` thì viết lại lịch sử. `git revert` tạo ra một commit mới có nội dung đảo ngược commit cũ, nên lịch sử vẫn còn nguyên vết và ai cũng thấy được mình đã hoàn tác cái gì — vì thế nó an toàn cho branch chung như `main`. Còn `git reset` dời con trỏ branch lùi lại, xóa các commit phía sau khỏi lịch sử, nên rất nguy hiểm nếu dùng trên branch chung vì đồng đội đã pull các commit đó về. `reset` có ba mode: `--soft` giữ lại thay đổi ở vùng staged, `--mixed` (mặc định) bỏ stage nhưng vẫn giữ code, còn `--hard` thì xóa luôn cả code chưa commit nên phải cực kỳ cẩn thận. Quy tắc của mình rất đơn giản: trên branch chung thì luôn dùng `git revert`, còn `reset --hard` chỉ dùng trên branch riêng của mình và khi chắc chắn không mất gì quan trọng."

---

- **`git revert`**: tạo **commit mới** đảo ngược commit cũ; lịch sử giữ nguyên → an toàn cho branch chung.
- **`git reset`**: dời con trỏ branch, **xoá** commit phía sau → viết lại lịch sử, nguy hiểm trên branch chung.
- Ba mode reset: `--soft` (giữ staged), `--mixed` (bỏ stage, giữ code), `--hard` (**xoá cả code chưa commit**).
- Quy tắc: branch chung → luôn **`git revert`**; `reset --hard` chỉ trên branch riêng và khi chắc chắn.

```bash
git revert a1b2c3d        # an toàn trên main
git reset --soft HEAD~1   # chỉ branch riêng: bỏ commit cuối, giữ code
```

</details>

<details>
<summary><b>29.7 .gitignore dùng làm gì, và lỡ commit secret rồi thì xử lý thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"`.gitignore` là file liệt kê những đường dẫn mình không muốn Git theo dõi, ví dụ `node_modules`, thư mục build `dist`, file log, và đặc biệt là các file secret như `.env`. Một điểm dễ nhầm là `.gitignore` chỉ có tác dụng với file chưa được track; nếu một file đã trót commit rồi thì thêm vào `.gitignore` cũng không gỡ nó ra, mình phải chạy `git rm --cached <file>` rồi commit lại để Git ngừng theo dõi. Còn nếu secret đã được push lên remote thì coi như đã lộ, kể cả mình xóa sau đó, vì repo có thể đã bị người khác clone hoặc nằm trong lịch sử. Việc quan trọng nhất lúc đó là rotate ngay — đổi key, password, token bị lộ — đây là ưu tiên số một; sau đó mới xóa nó khỏi lịch sử bằng `git filter-repo` hoặc BFG rồi force-push, nhưng bước này không thay thế được việc rotate. Để phòng ngừa thì mình commit `.env.example` chỉ chứa tên biến, đọc secret từ env hoặc secret manager, và bật secret-scanning trong CI để chặn từ đầu."

---

- `.gitignore` liệt kê đường dẫn Git **không track**: `node_modules`, `dist`, log, và đặc biệt secret (`.env`).
- Chỉ tác dụng với file **chưa track**: file đã commit phải `git rm --cached <file>` rồi commit lại.
- Secret đã **push lên remote = đã lộ**:
  1. **Rotate ngay** key/password/token (quan trọng nhất, vì repo có thể đã bị clone).
  2. Xoá khỏi lịch sử bằng `git filter-repo`/BFG rồi force-push (không thay thế rotate).
- Phòng ngừa: `.env.example`, đọc secret từ env/secret manager, bật secret-scanning CI.

```bash
git rm --cached .env
git commit -m "chore: stop tracking .env"
```

</details>

---

## 30. CI/CD và Triển khai (Docker, Cloud)

<details>
<summary><b>30.1 Phân biệt CI, Continuous Delivery và Continuous Deployment? Ranh giới giữa ba khái niệm này nằm ở đâu?</b></summary>

**📌 Câu trả lời mẫu:**

"Cả ba khái niệm này nói về cùng một dây chuyền tự động hóa, chỉ khác nhau ở chỗ tự động tới đâu thôi. CI (Continuous Integration) là phần nền: cứ mỗi commit hay PR mình merge thường xuyên, pipeline sẽ tự chạy lint, test và build để bắt lỗi sớm, tránh dồn việc rồi merge một cục lớn rất khó gỡ. Continuous Delivery là bước tiến lên: ngoài CI ra, artifact lúc nào cũng ở trạng thái sẵn sàng deploy, nhưng để lên production thì vẫn cần một người bấm nút duyệt. Còn Continuous Deployment là tự động hoàn toàn — commit nào qua hết pipeline là tự lên prod, không ai duyệt tay. Nên ranh giới rõ nhất nằm ở chỗ deploy lên prod: thủ công hẳn, có nút duyệt, hay tự động hoàn toàn. Deployment nhanh nhất nhưng đổi lại đòi hỏi test coverage cao, feature flag, monitoring và rollback ngon; nên thực tế đa số team như dự án Jira Clone của mình dừng ở mức tự động tới staging rồi duyệt tay lên prod cho an toàn."

---

- **CI**: mỗi commit/PR merge thường xuyên, pipeline tự chạy `lint + test + build` để phát hiện lỗi sớm, tránh "big bang merge".
- **Continuous Delivery**: mở rộng CI — artifact luôn sẵn sàng deploy nhưng lên prod cần **bấm tay duyệt** (manual approval).
- **Continuous Deployment**: mọi commit qua pipeline **tự động lên prod**, không duyệt thủ công.
- Continuous Deployment nhanh nhất nhưng đòi hỏi test coverage cao + feature flag + monitoring + rollback tốt; đa số team dừng ở **Delivery cho prod** (tự động tới staging, duyệt tay lên prod).

| | CI | Delivery | Deployment |
|---|---|---|---|
| Lint+test+build | Có | Có | Có |
| Artifact sẵn sàng | Không bắt buộc | Có | Có |
| Deploy prod | Thủ công | Bấm duyệt | Tự động |

```yaml
deploy-prod:
  environment: production   # GitHub chặn chờ approval -> Continuous Delivery
```

</details>

<details>
<summary><b>30.2 Một pipeline CI cho service NestJS nên gồm những giai đoạn nào và thứ tự ra sao? Vì sao thứ tự đó quan trọng?</b></summary>

**📌 Câu trả lời mẫu:**

"Nguyên tắc xương sống của mình khi sắp xếp pipeline là fail fast: bước nào rẻ và nhanh thì đẩy lên trước để phản hồi sớm, không phí phút runner cho commit chắc chắn hỏng. Với service NestJS mình thường xếp theo thứ tự: install dependencies với `--frozen-lockfile`, rồi lint/format, rồi type-check và build bằng `tsc --noEmit`, sau đó unit test, tiếp đến integration/e2e (phần này cần một Postgres chạy dưới dạng service container), rồi mới build Docker image, và cuối cùng deploy. Thứ tự này quan trọng vì nếu mình build Docker — vốn tốn vài phút — trước cả lint vài giây, thì một lỗi format vớ vẩn cũng bắt mình chờ cả phút build mới biết, rất phí. Một cái bẫy mình từng dính là quên `--frozen-lockfile`: CI tự cập nhật lockfile, thế là 'chạy được trên CI nhưng khác với máy dev', nên giờ mình luôn bắt buộc frozen trong pipeline. Riêng e2e của Jira Clone mình còn chạy `prisma migrate deploy` trên DB container trước khi test để schema khớp với code."

---

- Nguyên tắc **fail fast**: đặt bước rẻ/nhanh lên trước để phản hồi sớm, không tốn phút runner cho commit chắc chắn hỏng.
- Thứ tự: Install (`--frozen-lockfile`) → Lint/format → Type check/build (`tsc --noEmit`) → Unit test → Integration/e2e (cần DB qua service container) → Build Docker image → Deploy.
- Vì sao thứ tự quan trọng: build Docker (chậm vài phút) trước lint (vài giây) thì lỗi format nhỏ vẫn tốn cả phút build mới báo.
- **Bẫy**: bỏ `--frozen-lockfile` khiến CI tự cập nhật lockfile → "chạy được trên CI nhưng khác máy dev". Luôn dùng frozen/ci trong pipeline.

```yaml
- run: pnpm install --frozen-lockfile
- run: pnpm lint
- run: pnpm exec tsc --noEmit
- run: pnpm test --ci --coverage
```

</details>

<details>
<summary><b>30.3 Viết một multi-stage Dockerfile cho NestJS sao cho image nhỏ, build nhanh và chạy non-root. Giải thích từng stage.</b></summary>

**📌 Câu trả lời mẫu:**

"Ý tưởng cốt lõi của multi-stage là tách rõ giai đoạn build với giai đoạn runtime ra hai stage riêng. Stage build cần đủ đồ nặng như devDependencies và TypeScript compiler để biên dịch code; nhưng cái đó không nên dính vào image cuối, nên ở stage runtime mình chỉ copy sang phần code đã compile cộng với production dependencies thôi. Nhờ vậy image cuối nhỏ hơn hẳn và bề mặt tấn công cũng ít đi. Một mẹo quan trọng nữa là copy lockfile vào trước rồi mới copy source sau, để Docker cache lại layer install — mình đổi code mà package không đổi thì khỏi phải cài lại dependencies, build nhanh hơn nhiều. Mình cũng dùng `USER node` để container chạy non-root, tránh nguyên tắc chạy bằng root. Một cái bẫy mình hay nhắc anh em là phải có `.dockerignore`, không thì `node_modules` của máy host lọt vào image làm nó phình to và còn dễ sai binary native như bcrypt vì khác kiến trúc."

---

- Multi-stage tách **build** (cần devDeps + TS compiler) khỏi **runtime** (chỉ code compiled + prod deps) → image cuối nhỏ, ít bề mặt tấn công.
- **COPY lockfile trước, source sau**: tận dụng layer cache, đổi code không rebuild layer `install`.
- `pnpm prune --prod` loại devDeps; `node:20-alpine` (~50MB), cân nhắc `distroless`; `USER node` chạy non-root.
- **Bẫy**: quên `.dockerignore` → `node_modules` của host lọt vào image (phình to, sai binary native như `bcrypt`).

```dockerfile
FROM node:20-alpine AS build
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile
COPY . . && RUN pnpm build && pnpm prune --prod

FROM node:20-alpine AS runtime
ENV NODE_ENV=production
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
USER node
CMD ["node", "dist/main.js"]
```

</details>

<details>
<summary><b>30.4 Trình bày cách deploy một container NestJS lên cloud (ví dụ Cloud Run hoặc ECS). Những yêu cầu hạ tầng nào service phải tuân thủ để chạy được?</b></summary>

**📌 Câu trả lời mẫu:**

"Mô hình chung là mình build image rồi đẩy lên registry, sau đó nền tảng như Cloud Run hay ECS sẽ kéo image đó về chạy như một stateless container và tự scale theo lượng traffic. Để chạy được trên các nền tảng đó, service NestJS phải tuân vài quy ước hạ tầng. Quan trọng nhất là phải listen đúng cổng mà nền tảng inject vào qua `process.env.PORT`, và bind vào `0.0.0.0` chứ không phải `localhost`, nếu không thì health check không vào được. Service phải stateless — session thì đẩy ra Redis, file upload thì đẩy lên S3 hay GCS, vì container có thể bị kill và tạo lại bất cứ lúc nào. Ngoài ra cần graceful shutdown bằng `enableShutdownHooks` để đóng kết nối sạch sẽ khi bị scale down, một health endpoint cho probe gọi, và toàn bộ config thì cấp qua env và secret. Về lựa chọn, Cloud Run đơn giản nhất vì scale-to-zero và trả theo request, còn ECS Fargate thì hợp khi mình đã ở sẵn trong hệ sinh thái AWS."

---

- Mô hình chung: cung cấp image trong registry → nền tảng chạy như **stateless container**, tự scale theo traffic.
- Yêu cầu service NestJS: (1) **Listen đúng `PORT`** nền tảng inject (`process.env.PORT`); (2) **Stateless** — session→Redis, file→S3/GCS; (3) **Bind `0.0.0.0`**; (4) **Graceful shutdown** (`enableShutdownHooks`); (5) **Health endpoint** cho probe; (6) cấu hình qua env/secret.
- So sánh: **Cloud Run** đơn giản nhất (scale-to-zero, trả theo request); **ECS Fargate** hợp khi đã ở AWS; **Render** dễ cho team nhỏ.

```bash
gcloud run deploy api \
  --image .../api:$GIT_SHA --region asia-southeast1 --port 3000 \
  --set-env-vars NODE_ENV=production \
  --set-secrets DATABASE_URL=db-url:latest
```

</details>

<details>
<summary><b>30.5 Quản lý biến môi trường và secret theo từng môi trường (dev/staging/prod) như thế nào cho an toàn? `.env` file đóng vai trò gì và không nên làm gì?</b></summary>

**📌 Câu trả lời mẫu:**

"Nguyên tắc mình bám theo là 12-factor: config phải tách hẳn khỏi code và cấp qua environment, tuyệt đối không commit secret vào Git. Mình phân lớp theo môi trường: ở local thì dùng file `.env` đã gitignore, kèm một `.env.example` để đồng đội biết cần những biến nào; trên CI thì dùng environment secrets của pipeline; còn staging với production thì dùng secret manager như AWS Secrets Manager, GCP Secret Manager hay Vault. Trong NestJS mình dùng `@nestjs/config` kèm `validationSchema` bằng Joi để app fail ngay lúc khởi động nếu thiếu hoặc sai biến, thay vì chạy được một lúc rồi mới chết giữa chừng. Vai trò của `.env` chỉ là tiện cho dev ở máy local thôi. Còn những điều không nên làm: đừng commit `.env` thật lên Git, lỡ lộ là phải rotate ngay; đừng nhúng secret vào image qua `ENV` hay `ARG` vì nó nằm lại trong layer history ai cũng xem được; và đừng dùng chung một secret cho mọi môi trường. Secret nên được inject vào lúc runtime."

---

- Nguyên tắc 12-factor: **config tách khỏi code**, cấp qua environment, **không commit secret** vào Git.
- Phân lớp: local dùng `.env` (đã gitignore) + `.env.example`; CI dùng environment secrets; prod/staging dùng **secret manager** (AWS/GCP/Vault).
- NestJS dùng `@nestjs/config` với `validationSchema` (Joi) để **fail sớm** khi thiếu/sai biến; `ignoreEnvFile` ở production (biến đến từ container/secret manager).
- **Không** nên: commit `.env` thật (lỡ lộ phải **rotate** ngay); in `process.env`; dùng chung secret mọi môi trường; nhúng secret vào image qua `ENV`/`ARG` (nằm lại layer history) — inject lúc runtime.

```ts
ConfigModule.forRoot({
  validationSchema: Joi.object({
    DATABASE_URL: Joi.string().required(),
    JWT_SECRET: Joi.string().min(32).required(),
  }),
  ignoreEnvFile: process.env.NODE_ENV === 'production',
});
```

</details>

---

## 31. SQL Thuần và Thiết kế CSDL

> **Cơ bản → Nâng cao.** Phần này tập trung kỹ năng *viết và đọc SQL thuần* (query + thiết kế bảng) — phần bạn còn yếu — và **bổ trợ** cho [Phần 19](#19-database-transactions-and-concurrency) (đã lo transaction, lock, isolation, deadlock). Ví dụ dùng schema Jira Clone trên **PostgreSQL**: `project(id, key, name)`, `issue(id, project_id, assignee_id, status, priority, story_points, sprint_id, parent_id, created_at, deleted_at)`, `users(id, name, email)`, `comment(id, issue_id, author_id, created_at)`, `sprint(id, project_id, name)`.

### 🟢 Cơ bản

<details>
<summary><b>31.1 SQL là gì? Phân biệt nhóm lệnh DDL, DML, DCL, TCL?</b></summary>

**📌 Câu trả lời mẫu:**

"SQL là ngôn ngữ để làm việc với cơ sở dữ liệu quan hệ, và điểm hay là nó declarative — mình chỉ mô tả muốn lấy gì, còn làm thế nào để lấy thì để database tự quyết. Người ta thường chia các lệnh SQL thành bốn nhóm cho dễ nhớ. DDL (Data Definition) lo về cấu trúc, tức là định nghĩa cái khung bảng, gồm CREATE, ALTER, DROP, TRUNCATE. DML (Data Manipulation) thì thao tác trên dữ liệu bên trong, gồm SELECT, INSERT, UPDATE, DELETE — đây là nhóm mình dùng hằng ngày nhiều nhất. DCL (Data Control) lo phân quyền với GRANT và REVOKE. Còn TCL (Transaction Control) gom nhiều thay đổi thành một giao dịch, gồm BEGIN, COMMIT, ROLLBACK, SAVEPOINT. Cách mình nhớ nhanh là: DDL đổi cái khung, DML đổi nội dung, còn TCL gói nhiều thay đổi lại thành một đơn vị nguyên vẹn."

---

- **SQL** = ngôn ngữ truy vấn CSDL quan hệ; bạn mô tả *muốn gì* (declarative), DB tự quyết *làm thế nào*.
- **DDL** (Data Definition) — định nghĩa cấu trúc: `CREATE`, `ALTER`, `DROP`, `TRUNCATE`.
- **DML** (Data Manipulation) — thao tác dữ liệu: `SELECT`, `INSERT`, `UPDATE`, `DELETE`.
- **DCL** (Data Control) — phân quyền: `GRANT`, `REVOKE`.
- **TCL** (Transaction Control) — gom giao dịch: `BEGIN`, `COMMIT`, `ROLLBACK`, `SAVEPOINT`.
- Mẹo nhớ: DDL đổi *khung*, DML đổi *nội dung*, TCL gói *nhiều thay đổi thành 1 đơn vị* (xem 19.1).

```sql
CREATE TABLE project (id serial PRIMARY KEY, key text UNIQUE, name text); -- DDL
INSERT INTO project (key, name) VALUES ('JIRA', 'Jira Clone');           -- DML
```

</details>

<details>
<summary><b>31.2 Thứ tự thực thi LOGIC của một câu SELECT là gì? Vì sao không alias được ở WHERE?</b></summary>

**📌 Câu trả lời mẫu:**

"Điểm dễ gây nhầm là mình viết câu SELECT theo thứ tự `SELECT ... FROM ... WHERE ...`, nhưng database lại xử lý logic theo một thứ tự khác hẳn: đầu tiên là FROM và JOIN để lấy dữ liệu nguồn, rồi WHERE lọc dòng, rồi GROUP BY gom nhóm, rồi HAVING lọc nhóm, sau đó mới tới SELECT, rồi DISTINCT, ORDER BY, và cuối cùng LIMIT. Hiểu được cái thứ tự này là giải thích được rất nhiều lỗi khó hiểu. Ví dụ tại sao không alias được ở WHERE: tại vì WHERE chạy trước SELECT, mà alias được đặt ở SELECT, nên ở thời điểm WHERE chạy thì cái tên alias đó chưa tồn tại. Ngược lại, ORDER BY chạy sau SELECT nên mình lại dùng được alias thoải mái. Tương tự, GROUP BY chạy trước SELECT nên trong SELECT mình chỉ được nêu cột đã group hoặc bọc trong hàm tổng hợp thôi. Nắm cái này thì mấy lỗi kiểu 'column does not exist' hay 'must appear in GROUP BY' mình nhìn là biết nguyên nhân ngay."

---

- Viết theo `SELECT ... FROM ...` nhưng DB **xử lý logic theo thứ tự**: `FROM/JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT → ORDER BY → LIMIT/OFFSET`.
- Hệ quả quan trọng: `WHERE` chạy **trước** `SELECT` nên **không dùng được alias** đặt ở SELECT; còn `ORDER BY` chạy sau SELECT nên **dùng được** alias.
- `GROUP BY` xảy ra trước `SELECT` → trong SELECT chỉ được nêu cột đã group hoặc hàm tổng hợp.
- Hiểu thứ tự này giúp giải thích đa số lỗi "column does not exist" và "must appear in GROUP BY".

```sql
SELECT status, COUNT(*) AS total
FROM issue
WHERE deleted_at IS NULL   -- chua thay alias "total" o day duoc
GROUP BY status
ORDER BY total DESC;       -- nhung o day thi duoc
```

</details>

<details>
<summary><b>31.3 Các loại JOIN (INNER, LEFT, RIGHT, FULL, CROSS) khác nhau thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"JOIN dùng để ghép dữ liệu từ nhiều bảng, và các loại khác nhau ở chỗ giữ lại dòng nào khi không khớp. INNER JOIN chỉ giữ những dòng khớp ở cả hai bảng. LEFT JOIN thì giữ toàn bộ bảng bên trái, bên phải không khớp thì điền NULL — ví dụ trong Jira Clone mình muốn liệt kê mọi issue kể cả những issue chưa có người được giao thì dùng LEFT JOIN tới bảng users. RIGHT JOIN thì ngược lại với LEFT, nhưng thực tế ít dùng vì người ta hay viết lại thành LEFT cho dễ đọc. FULL OUTER JOIN giữ cả hai bên, bên nào thiếu thì NULL. Còn CROSS JOIN là tích Descartes, ghép mọi cặp với nhau, cái này phải cẩn thận vì số dòng bùng nổ rất nhanh. Một bẫy kinh điển mình hay gặp là khi LEFT JOIN mà lại đặt điều kiện lọc bảng bên phải vào WHERE, nó vô tình biến thành INNER JOIN luôn, vì dòng NULL bị WHERE loại mất — đúng ra điều kiện đó phải đặt trong ON."

---

- **INNER JOIN**: chỉ giữ dòng **khớp ở cả hai** bảng.
- **LEFT JOIN**: giữ **toàn bộ bảng trái**, bên phải không khớp thì điền `NULL` (vd: lấy mọi issue kể cả chưa có assignee).
- **RIGHT JOIN**: ngược lại LEFT (ít dùng — thường viết lại thành LEFT cho dễ đọc).
- **FULL OUTER JOIN**: giữ cả hai bên, thiếu thì `NULL`.
- **CROSS JOIN**: tích Descartes (mọi cặp) — cẩn thận bùng nổ số dòng.
- Bẫy hay gặp: lọc bảng phải của LEFT JOIN trong `WHERE` sẽ **vô tình biến nó thành INNER JOIN** → điều kiện đó phải đặt trong `ON`.

```sql
-- Moi issue + ten nguoi duoc giao (NULL neu chua giao)
SELECT i.id, i.status, u.name AS assignee
FROM issue i
LEFT JOIN users u ON u.id = i.assignee_id;
```

</details>

<details>
<summary><b>31.4 GROUP BY và các hàm tổng hợp (COUNT/SUM/AVG/MIN/MAX) hoạt động ra sao?</b></summary>

**📌 Câu trả lời mẫu:**

"GROUP BY gom các dòng có cùng giá trị lại thành một nhóm, rồi hàm tổng hợp như COUNT, SUM, AVG sẽ tính ra một giá trị cho mỗi nhóm đó. Ví dụ trong Jira Clone mình group theo project_id thì mỗi project thành một nhóm, và mình đếm được số issue hay tính tổng story point cho từng project. Một quy tắc bắt buộc phải nhớ là mọi cột trong SELECT thì hoặc phải nằm trong GROUP BY, hoặc phải được bọc trong một hàm tổng hợp, không thì database báo lỗi ngay. Về mấy hàm COUNT thì cần phân biệt rõ: COUNT(*) đếm mọi dòng kể cả dòng có NULL, COUNT(col) chỉ đếm những dòng mà cột đó khác NULL, còn COUNT(DISTINCT col) đếm số giá trị duy nhất. Một điểm dễ làm mình bất ngờ là AVG và SUM đều bỏ qua NULL chứ không coi NULL như số 0, nên kết quả trung bình có thể khác với cái mình tưởng nếu trong cột có nhiều giá trị NULL."

---

- `GROUP BY` gom các dòng cùng giá trị thành **một nhóm**; hàm tổng hợp tính ra **một giá trị mỗi nhóm**.
- Mọi cột trong `SELECT` phải **hoặc** nằm trong `GROUP BY` **hoặc** bọc trong hàm tổng hợp.
- Khác biệt cần nhớ: `COUNT(*)` đếm **mọi dòng** (kể cả NULL); `COUNT(col)` đếm dòng có `col` **khác NULL**; `COUNT(DISTINCT col)` đếm giá trị **duy nhất**.
- `AVG/SUM` **bỏ qua NULL** (không coi NULL là 0) — dễ ra kết quả "bất ngờ".

```sql
-- So issue va tong story point moi project
SELECT project_id, COUNT(*) AS so_issue, SUM(story_points) AS tong_diem
FROM issue
GROUP BY project_id;
```

</details>

<details>
<summary><b>31.5 WHERE và HAVING khác nhau ở đâu?</b></summary>

**📌 Câu trả lời mẫu:**

"WHERE và HAVING đều là lọc, nhưng khác nhau ở chỗ lọc tại thời điểm nào trong quá trình xử lý. WHERE lọc từng dòng trước khi gom nhóm, nên ở WHERE mình không dùng được hàm tổng hợp, vì lúc đó các nhóm còn chưa được hình thành. Còn HAVING lọc sau khi GROUP BY đã chạy xong, tức là lọc trên kết quả tổng hợp của từng nhóm, nên ở HAVING mình mới dùng được COUNT, SUM và các hàm khác. Ví dụ muốn tìm những project có trên 10 issue chưa đóng, thì điều kiện status khác DONE mình đặt ở WHERE để lọc dòng, còn điều kiện COUNT lớn hơn 10 thì đặt ở HAVING để lọc nhóm. Về tối ưu, nguyên tắc của mình là đẩy càng nhiều điều kiện lọc dòng vào WHERE càng tốt, vì lọc sớm thì lượng dữ liệu phải gom nhóm ít đi, chạy nhanh hơn; chỉ để lại ở HAVING những điều kiện thật sự tính trên cả nhóm."

---

- `WHERE` lọc **từng dòng trước khi** gom nhóm → **không** dùng được hàm tổng hợp.
- `HAVING` lọc **sau khi** `GROUP BY`, trên **kết quả tổng hợp** của nhóm → dùng được `COUNT/SUM/...`.
- Tối ưu: đẩy điều kiện lọc dòng vào `WHERE` (giảm dữ liệu sớm), chỉ để điều kiện trên nhóm ở `HAVING`.

```sql
-- Cac project co tren 10 issue chua dong
SELECT project_id, COUNT(*) AS so_issue
FROM issue
WHERE status <> 'DONE'        -- loc dong
GROUP BY project_id
HAVING COUNT(*) > 10;         -- loc nhom
```

</details>

<details>
<summary><b>31.6 NULL trong SQL có gì đặc biệt? Các bẫy thường gặp?</b></summary>

**📌 Câu trả lời mẫu:**

"NULL trong SQL đặc biệt vì nó không có nghĩa là 0 hay chuỗi rỗng, mà mang nghĩa 'không biết' — giá trị chưa xác định. Chính vì thế mọi so sánh với NULL bằng dấu bằng hay khác đều không trả về TRUE mà trả về UNKNOWN, nên muốn kiểm tra NULL mình bắt buộc phải dùng IS NULL hoặc IS NOT NULL chứ không viết = NULL được. Cái này dẫn tới vài bẫy hay gặp. Thứ nhất, NULL bị loại khỏi COUNT(col), SUM và AVG nên kết quả có thể lệch so với mình tưởng. Thứ hai, một điều kiện kiểu `WHERE col <> 'X'` sẽ vô tình bỏ sót những dòng mà col là NULL, vì so sánh ra UNKNOWN nên dòng đó không lọt qua — đây là lỗi rất hay quên. Để xử lý thì mình hay dùng COALESCE để thay NULL bằng một giá trị mặc định, ví dụ hiển thị 'Chưa giao' thay cho NULL ở cột assignee; còn NULLIF thì trả về NULL khi hai giá trị bằng nhau, tiện để tránh lỗi chia cho 0."

---

- `NULL` nghĩa là "**không biết**", không phải 0 hay chuỗi rỗng.
- So sánh với NULL bằng `=`/`<>` luôn cho **UNKNOWN** (không TRUE) → phải dùng `IS NULL` / `IS NOT NULL`.
- `NULL` bị loại khỏi `COUNT(col)`, `SUM`, `AVG`; và `WHERE col <> 'X'` sẽ **bỏ sót** dòng có `col IS NULL`.
- `COALESCE(a, b)` trả về giá trị non-NULL đầu tiên; `NULLIF(a, b)` trả NULL nếu a=b (tránh chia 0).

```sql
-- Hien thi 'Chua giao' thay vi NULL, va dem dung ca dong NULL
SELECT COALESCE(u.name, 'Chua giao') AS assignee, COUNT(*) 
FROM issue i LEFT JOIN users u ON u.id = i.assignee_id
GROUP BY u.name;
```

</details>

<details>
<summary><b>31.7 Phân biệt PRIMARY KEY, FOREIGN KEY và UNIQUE constraint?</b></summary>

**📌 Câu trả lời mẫu:**

"Cả ba đều là ràng buộc để giữ dữ liệu sạch nhưng vai trò khác nhau. PRIMARY KEY là cái định danh duy nhất cho mỗi dòng, nó vừa phải duy nhất vừa không được NULL, và mỗi bảng chỉ có một cái — mình thường dùng nó làm id chính như id của issue. FOREIGN KEY là ràng buộc tham chiếu sang khóa của bảng khác, đảm bảo toàn vẹn tham chiếu, tức là không thể có một issue trỏ tới một project_id không tồn tại, và mình có thể gắn thêm hành vi như ON DELETE CASCADE để xóa project thì issue con cũng dọn theo. UNIQUE thì giống PRIMARY KEY ở chỗ chống trùng, nhưng nó cho phép NULL và một bảng có thể có nhiều UNIQUE constraint — ví dụ trong Jira Clone mình bắt cặp (project_id, key) phải unique để mã JIRA-1 không trùng trong cùng project. Một điểm hay được hỏi là Postgres thực thi PK và UNIQUE bằng unique index ngầm, nên tự nhiên đã có index rồi, không cần tạo thêm."

---

- **PRIMARY KEY**: định danh duy nhất mỗi dòng — **duy nhất + NOT NULL**, mỗi bảng chỉ một (có thể gồm nhiều cột = composite key).
- **FOREIGN KEY**: ràng buộc tham chiếu sang khóa của bảng khác, đảm bảo **toàn vẹn tham chiếu** (không trỏ tới id không tồn tại); kèm hành vi `ON DELETE CASCADE/SET NULL/RESTRICT` (xem 19.6).
- **UNIQUE**: đảm bảo không trùng nhưng **cho phép NULL** (Postgres coi nhiều NULL là khác nhau); một bảng có nhiều unique constraint.
- Postgres thực thi PK/UNIQUE bằng **unique index** ngầm → tự có index, không cần tạo thêm.

```sql
CREATE TABLE issue (
  id serial PRIMARY KEY,
  project_id int NOT NULL REFERENCES project(id) ON DELETE CASCADE,
  key text NOT NULL,
  UNIQUE (project_id, key)   -- "JIRA-1" duy nhat trong moi project
);
```

</details>

### 🟡 Trung bình

<details>
<summary><b>31.8 Subquery: phân biệt scalar, IN, EXISTS và correlated subquery?</b></summary>

**📌 Câu trả lời mẫu:**

"Subquery là query lồng trong query, và tùy cách dùng thì nó có vài dạng. Scalar subquery trả về đúng một giá trị nên mình dùng nó như một biểu thức, ví dụ so sánh story point của issue với giá trị trung bình. IN (subquery) thì lọc theo một tập giá trị, nhưng cần cẩn thận: nếu là NOT IN mà subquery có chứa NULL thì kết quả trả về rỗng, vì so sánh với NULL ra UNKNOWN — đây là bẫy kinh điển. EXISTS chỉ trả TRUE hoặc FALSE theo kiểu 'có dòng nào khớp không', xử lý NULL an toàn nên mình hay dùng NOT EXISTS thay cho NOT IN. Còn correlated subquery là loại tham chiếu ngược lại cột của query ngoài, nên về mặt logic nó chạy lại cho từng dòng ngoài, dễ chậm khi dữ liệu lớn; lúc đó mình cân nhắc viết lại thành JOIN. Quy tắc của mình là khi chỉ cần kiểm tra tồn tại thì ưu tiên EXISTS, khi cần lấy giá trị cụ thể thì mới dùng scalar hoặc IN."

---

- **Scalar**: trả đúng 1 giá trị, dùng như một biểu thức (vd so sánh với trung bình).
- **`IN (subquery)`**: lọc theo tập giá trị; cẩn thận `NOT IN` mà subquery có NULL → trả **rỗng** (vì so sánh ra UNKNOWN).
- **`EXISTS (subquery)`**: trả TRUE/FALSE theo "có dòng nào không"; xử lý NULL an toàn, thường dùng `NOT EXISTS` thay cho `NOT IN`.
- **Correlated**: subquery tham chiếu cột của query ngoài → chạy **lại cho mỗi dòng ngoài** (có thể chậm; cân nhắc viết lại thành JOIN).

```sql
-- Project co it nhat 1 issue dang mo (EXISTS dung khi chi can "co/khong")
SELECT p.* FROM project p
WHERE EXISTS (
  SELECT 1 FROM issue i WHERE i.project_id = p.id AND i.status <> 'DONE'
);
```

</details>

<details>
<summary><b>31.9 Vì sao LEFT JOIN rồi COUNT(*) bị sai, mà COUNT(cột) lại đúng?</b></summary>

**📌 Câu trả lời mẫu:**

"Bản chất là do LEFT JOIN luôn giữ lại dòng bên trái kể cả khi không có dòng nào khớp bên phải. Ví dụ mình LEFT JOIN project với issue để đếm số issue mỗi project: với project không có issue nào, LEFT JOIN vẫn sinh ra một dòng nhưng phần cột của issue toàn là NULL. Lúc này COUNT(*) đếm cả cái dòng 'ảo' đó nên ra 1 thay vì 0, đó là sai. Còn COUNT(i.id) thì chỉ đếm những giá trị i.id khác NULL, nên project không có issue sẽ ra đúng là 0. Nên quy tắc mình luôn nhớ là khi đếm phía bảng được LEFT JOIN thì đếm theo cột khóa của bảng đó, đừng bao giờ dùng COUNT(*). Đây là lỗi rất hay gặp và cũng hay được hỏi để bẫy."

---

- Với project **không có issue nào**, LEFT JOIN vẫn sinh **1 dòng** với phần issue toàn `NULL`.
- `COUNT(*)` đếm cả dòng "ảo" đó → ra **1** thay vì **0** (sai).
- `COUNT(i.id)` chỉ đếm `i.id` khác NULL → ra **0** đúng. Đây là lỗi kinh điển hay được hỏi.
- Quy tắc: khi đếm phía bảng được LEFT JOIN, **đếm theo cột khóa** của bảng đó, đừng dùng `COUNT(*)`.

```sql
SELECT p.id, COUNT(i.id) AS so_issue   -- KHONG dung COUNT(*)
FROM project p
LEFT JOIN issue i ON i.project_id = p.id
GROUP BY p.id;
```

</details>

<details>
<summary><b>31.10 UNION vs UNION ALL? Khi nào dùng cái nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Cả hai đều dùng để nối kết quả của hai query lại theo chiều dọc, tức là thêm dòng, miễn là chúng cùng số cột và kiểu dữ liệu tương thích. Khác biệt chính là UNION sẽ loại bỏ các dòng trùng nhau, mà để khử trùng thì database phải sort hoặc hash toàn bộ kết quả, nên tốn tài nguyên hơn. UNION ALL thì giữ lại mọi dòng kể cả trùng, không phải khử nên nhanh hơn. Quan điểm của mình là mặc định nên dùng UNION ALL nếu mình biết chắc dữ liệu không trùng hoặc trùng cũng không sao — chỉ dùng UNION khi thật sự cần loại trùng. Cũng cần phân biệt với JOIN: JOIN nối theo chiều ngang, tức thêm cột bằng cách ghép các bảng lại, còn UNION nối theo chiều dọc. Ví dụ trong Jira Clone, mình có thể gom issue và comment mới trong 7 ngày thành một feed hoạt động chung bằng UNION ALL."

---

- Cả hai nối kết quả 2 query (phải **cùng số cột, kiểu tương thích**).
- `UNION` **loại trùng** → phải sort/hash để khử trùng → **tốn hơn**.
- `UNION ALL` giữ **mọi dòng** (kể cả trùng) → **nhanh hơn**; mặc định nên dùng nếu chắc chắn không có/không cần khử trùng.
- Khác `JOIN`: UNION nối **theo chiều dọc** (thêm dòng), JOIN nối **theo chiều ngang** (thêm cột).

```sql
SELECT id, 'issue' AS loai FROM issue WHERE created_at > now() - interval '7 day'
UNION ALL
SELECT id, 'comment' FROM comment WHERE created_at > now() - interval '7 day';
```

</details>

<details>
<summary><b>31.11 Dùng CASE WHEN để "pivot" / conditional aggregation thế nào?</b></summary>

**📌 Câu trả lời mẫu:**

"CASE WHEN về cơ bản là biểu thức điều kiện kiểu if-else ngay trong SQL, mình dùng được trong SELECT, ORDER BY và đặc biệt là bên trong các hàm tổng hợp. Mẹo mạnh nhất là conditional aggregation: mình bọc CASE WHEN vào trong COUNT hoặc SUM để biến dữ liệu từ dạng dòng thành dạng cột, kiểu pivot. Ví dụ trong Jira Clone mình muốn một bảng dashboard mỗi project một dòng, có các cột riêng đếm số issue ở trạng thái TODO, IN_PROGRESS, DONE — chỉ cần một query với mấy COUNT có điều kiện là ra, thay vì phải chạy ba query riêng cho từng status rồi ghép lại. Trong Postgres còn có cú pháp gọn hơn là COUNT(*) FILTER (WHERE ...), về bản chất nó tương đương COUNT(CASE WHEN ... THEN 1 END) nhưng dễ đọc hơn. Lợi ích là gom nhiều thống kê vào một lần quét bảng, vừa nhanh vừa gọn."

---

- `CASE WHEN ... THEN ... ELSE ... END` là biểu thức điều kiện, dùng được trong `SELECT`, `ORDER BY`, và **bên trong hàm tổng hợp**.
- Mẹo mạnh: gộp nhiều `COUNT(CASE WHEN ...)` để biến **dòng thành cột** (pivot) — đếm issue theo từng trạng thái trên một dòng.
- Thay cho việc chạy nhiều query riêng cho mỗi status.

```sql
-- Bang dashboard: moi project 1 dong, cot theo trang thai
SELECT project_id,
  COUNT(*) FILTER (WHERE status = 'TODO')        AS todo,
  COUNT(*) FILTER (WHERE status = 'IN_PROGRESS') AS doing,
  COUNT(*) FILTER (WHERE status = 'DONE')        AS done
FROM issue GROUP BY project_id;
-- FILTER (WHERE...) la cu phap Postgres; tuong duong COUNT(CASE WHEN ... THEN 1 END)
```

</details>

<details>
<summary><b>31.12 Chuẩn hóa (1NF, 2NF, 3NF) là gì? Khi nào cố ý denormalize?</b></summary>

**📌 Câu trả lời mẫu:**

"Chuẩn hóa là cách tổ chức bảng để giảm trùng lặp và tránh sai lệch dữ liệu. 1NF nghĩa là mỗi ô phải là giá trị nguyên tử, không được nhồi nhiều giá trị hay một mảng vào một cột — ví dụ thay vì nhét 'label1,label2' vào một cột text, mình tách ra bảng riêng issue_label. 2NF là khi đã đạt 1NF và không còn cột nào chỉ phụ thuộc vào một phần của khóa chính ghép, tức mọi cột phải phụ thuộc toàn bộ khóa. 3NF là đạt 2NF và không còn phụ thuộc bắc cầu, ví dụ mình không lưu project_name trong bảng issue mà chỉ giữ project_id, vì tên project phụ thuộc vào project chứ không phải issue. Mục tiêu chung là dữ liệu chỉ nằm một chỗ để khỏi sửa nhiều nơi. Tuy nhiên đôi khi mình cố ý denormalize, tức là chấp nhận lặp dữ liệu, khi cần tốc độ đọc cao — ví dụ cache sẵn comment_count trên issue để khỏi phải đếm lại mỗi lần. Đánh đổi là mình phải tự lo đồng bộ con số đó cho đúng."

---

- **1NF**: mỗi ô là **giá trị nguyên tử**, không nhóm lặp/mảng nhồi trong 1 cột (vd không nhét "label1,label2" vào một cột text → tách bảng `issue_label`).
- **2NF**: đạt 1NF + **không phụ thuộc một phần** vào *một phần* khóa chính ghép (mọi cột phải phụ thuộc **toàn bộ** khóa).
- **3NF**: đạt 2NF + **không phụ thuộc bắc cầu** (cột non-key không phụ thuộc cột non-key khác — vd đừng lưu `project_name` trong bảng `issue`, chỉ giữ `project_id`).
- Mục tiêu: giảm trùng lặp & sai lệch dữ liệu. **Denormalize** (cố ý lặp dữ liệu) chỉ khi cần **tốc độ đọc** cao và chấp nhận tự đồng bộ (vd cache `comment_count`) — đánh đổi đúng-đắn lấy hiệu năng.

```sql
-- Chuan hoa quan he nhieu-nhieu issue <-> label
CREATE TABLE issue_label (
  issue_id int REFERENCES issue(id),
  label_id int REFERENCES label(id),
  PRIMARY KEY (issue_id, label_id)
);
```

</details>

<details>
<summary><b>31.13 Index giúp gì? "Sargable" là gì? Vì sao nhiều WHERE không dùng được index?</b></summary>

**📌 Câu trả lời mẫu:**

"Index, kiểu phổ biến nhất là B-tree, giúp database tìm nhanh đúng dòng cần thay vì quét toàn bộ bảng, giống như mục lục của một cuốn sách. Nhưng index chỉ phát huy được khi điều kiện WHERE 'sargable', tức là so sánh trực tiếp lên chính cột đó, ví dụ col = giá trị, col > giá trị, BETWEEN, hay LIKE 'abc%'. Cái làm index thành vô dụng là khi mình bọc một hàm lên cột, ví dụ LOWER(email) = ... hay DATE(created_at) = ..., hoặc tính toán trên cột, hoặc LIKE mở đầu bằng dấu phần trăm — vì lúc đó database không còn dò được theo thứ tự đã sắp trong index nên phải quét tuần tự cả bảng. Cách chữa là một trong hai: hoặc viết lại điều kiện về dạng range trên cột gốc, ví dụ thay DATE(created_at) = một ngày bằng created_at nằm trong khoảng từ đầu ngày đến đầu ngày sau; hoặc tạo functional index trên đúng biểu thức đó như index trên LOWER(email). Nói ngắn gọn, mình luôn cố giữ cột 'sạch' trong điều kiện lọc để index còn dùng được."

---

- Index (B-tree) cho phép DB **tìm nhanh** thay vì quét toàn bảng (chi tiết loại index ở 19.4).
- **Sargable** = điều kiện mà index dùng được: so sánh **trực tiếp trên cột** (`col = $1`, `col > $1`, `col BETWEEN`, `LIKE 'abc%'`).
- **Không sargable** (làm index vô dụng): bọc **hàm lên cột** (`LOWER(email) = ...`, `DATE(created_at) = ...`), `LIKE '%abc%'` (mở đầu bằng `%`), hoặc tính toán trên cột (`story_points + 1 > 5`).
- Cách chữa: tạo **functional index** (`CREATE INDEX ... ON users (LOWER(email))`), hoặc viết lại điều kiện về dạng range trên cột gốc.

```sql
-- KHONG sargable: ham boc cot -> seq scan
WHERE DATE(created_at) = '2026-06-27'
-- Sargable: range tren cot goc -> dung duoc index
WHERE created_at >= '2026-06-27' AND created_at < '2026-06-28'
```

</details>

<details>
<summary><b>31.14 Phân trang OFFSET vs keyset (cursor) — cái nào tốt cho danh sách issue lớn?</b></summary>

**📌 Câu trả lời mẫu:**

"OFFSET/LIMIT là cách phân trang đơn giản nhất, nhưng vấn đề là database phải đọc qua rồi vứt bỏ toàn bộ số dòng trước OFFSET mới tới được trang mình cần, nên trang càng sâu thì càng chậm. Nó còn dễ bị lệch hoặc trùng dòng nếu dữ liệu thay đổi giữa lúc người dùng lật trang. Keyset, hay còn gọi cursor hoặc seek pagination, thì thay vì đếm bỏ, mình nhớ lại giá trị mốc của dòng cuối trang trước, ví dụ cặp created_at và id, rồi trang sau chỉ cần lọc những dòng nhỏ hơn mốc đó. Cách này bám trực tiếp vào index nên nhanh và ổn định ở mọi trang, dù là trang 1 hay trang 1000. Đánh đổi là keyset không nhảy thẳng tới 'trang 57' tùy ý được, chỉ đi next/prev, nên nó hợp với infinite scroll. Với danh sách issue lớn trong Jira Clone, mình ưu tiên keyset; còn nếu UI cần hiển thị số trang cụ thể thì mới dùng OFFSET."

---

- **OFFSET/LIMIT**: đơn giản nhưng DB phải **đọc rồi bỏ** OFFSET dòng đầu → trang càng sâu **càng chậm**; còn bị **lệch/trùng** nếu dữ liệu thay đổi giữa các trang.
- **Keyset (cursor/seek)**: nhớ giá trị mốc của trang trước (vd `created_at, id`) rồi lọc `WHERE (created_at, id) < (...)` → **nhanh ổn định** mọi trang vì bám vào index, không "đếm bỏ".
- Đánh đổi: keyset không nhảy tới "trang 57" tùy ý (chỉ next/prev) → hợp **infinite scroll**; OFFSET hợp khi cần số trang.

```sql
-- Keyset: trang tiep theo sau (created_at, id) cua dong cuoi trang truoc
SELECT * FROM issue
WHERE project_id = $1 AND (created_at, id) < ($2, $3)
ORDER BY created_at DESC, id DESC
LIMIT 20;
```

</details>

### 🔴 Nâng cao

<details>
<summary><b>31.15 CTE (WITH) là gì? Recursive CTE để duyệt cây subtask/epic ra sao?</b></summary>

**📌 Câu trả lời mẫu:**

"CTE là cú pháp WITH name AS (...) cho phép mình đặt tên cho một query con rồi dùng lại ở query chính. Lợi ích chủ yếu là code dễ đọc, tách bạch từng bước logic, thay vì lồng subquery sâu nhiều tầng nhìn rất rối. Mạnh hơn nữa là recursive CTE, viết WITH RECURSIVE, dùng để duyệt các cấu trúc phân cấp như cây. Trong Jira Clone đây đúng là tình huống lấy toàn bộ subtask con cháu của một issue, hoặc đi từ epic xuống story rồi xuống subtask qua cột parent_id. Recursive CTE gồm hai phần nối với nhau bằng UNION ALL: phần anchor là điểm gốc mình bắt đầu, và phần đệ quy liên tục nối thêm các con của những dòng đã tìm được, lặp tới khi không còn con nào. Một lưu ý quan trọng là phải đảm bảo cây không có vòng lặp, nếu không nó sẽ chạy đệ quy vô tận."

---

- **CTE** (`WITH name AS (...)`): đặt tên cho một query con để **đọc dễ, tái dùng**, tách bước logic (thay cho subquery lồng sâu).
- **Recursive CTE** (`WITH RECURSIVE`): duyệt cấu trúc **phân cấp/đồ thị** (cây subtask qua `parent_id`, epic → story → subtask) — gồm phần **anchor** (gốc) `UNION ALL` phần **đệ quy** (nối con).
- Lưu ý: cần điều kiện dừng (cây không vòng) để tránh lặp vô tận.

```sql
-- Lay toan bo subtask con-chau cua issue goc #100
WITH RECURSIVE tree AS (
  SELECT id, parent_id FROM issue WHERE id = 100          -- anchor
  UNION ALL
  SELECT c.id, c.parent_id
  FROM issue c JOIN tree t ON c.parent_id = t.id          -- de quy
)
SELECT * FROM tree;
```

</details>

<details>
<summary><b>31.16 SQL vs NoSQL: chọn dựa trên tiêu chí nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Mình không chọn theo kiểu 'NoSQL nhanh hơn' mà chọn dựa trên access pattern và mức nhất quán nghiệp vụ cần. SQL quan hệ mạnh ở chỗ schema chặt, JOIN tốt, và transaction ACID xuyên nhiều bảng — rất hợp với dữ liệu quan hệ chặt và cần consistency cao. Jira Clone của mình là ví dụ điển hình: project, issue, user, comment ràng buộc lẫn nhau, và khi kéo-thả issue trên board mình cần cập nhật trạng thái cùng vài bảng liên quan trong một transaction, nên Postgres là lựa chọn tự nhiên. NoSQL — document, key-value hay wide-column — thắng khi schema cần linh hoạt, scale ngang dễ, và mình chủ yếu đọc/ghi theo một aggregate độc lập với throughput lớn, như log, event hay catalog. Đánh đổi là mình phải tự lo consistency và JOIN ở tầng ứng dụng. Quan điểm của mình là phần lớn app nên khởi đầu bằng Postgres, vì nó còn có `jsonb` để chứa phần dữ liệu linh hoạt, và chỉ thêm NoSQL khi có nhu cầu thật rõ ràng về scale hay đặc thù truy cập — chẳng hạn nếu cần lưu activity feed khổng lồ thì mình mới cân nhắc tách riêng."

---

- **SQL (quan hệ)**: schema chặt, **JOIN** mạnh, **transaction ACID** đa bảng, truy vấn linh hoạt ad-hoc → hợp dữ liệu **quan hệ chặt, tính nhất quán cao** (đa số nghiệp vụ như Jira: project–issue–user–comment).
- **NoSQL** (document/key-value/wide-column): schema linh hoạt, scale ngang dễ, đọc/ghi theo **một aggregate** rất nhanh → hợp dữ liệu **phi quan hệ, throughput lớn, hình dạng đa dạng** (log, event, catalog).
- Đừng chọn theo "hot tech": phần lớn app khởi đầu **nên dùng Postgres** (có cả `jsonb` cho phần linh hoạt); chỉ thêm NoSQL khi có nhu cầu rõ (scale/đặc thù truy cập). Chi tiết MongoDB ở [Phần 30](#30-mongodb-và-mongoose-nosql).
- Câu trả lời "ăn điểm": nói về **mô hình truy cập (access pattern)** và **tính nhất quán cần có**, không nói chung chung "NoSQL nhanh hơn".

</details>

<details>
<summary><b>31.17 Một query danh sách issue rất chậm — bạn debug và tối ưu theo các bước nào?</b></summary>

**📌 Câu trả lời mẫu:**

"Nguyên tắc đầu tiên của mình là đo trước, đoán sau — đừng tối ưu theo cảm tính. Mình sẽ bật query log, ví dụ Prisma có log query, để tìm chính xác câu nào chậm, rồi chạy EXPLAIN ANALYZE để xem kế hoạch thực thi. Mình đọc plan để tìm dấu hiệu xấu như Seq Scan trên một bảng lớn, hay số dòng actual lệch hẳn so với estimated — thường là do thống kê cũ — và xem node nào ngốn thời gian nhất. Sau đó mình sửa theo thứ tự ưu tiên: đầu tiên là thêm hoặc chỉnh index composite cho đúng thứ tự cột trong WHERE rồi tới ORDER BY; tiếp theo viết lại điều kiện cho sargable, bỏ những hàm bọc lên cột; rồi giảm lượng dữ liệu trả về bằng cách chỉ SELECT cột cần và phân trang kiểu keyset; sau đó xử lý vấn đề N+1 bằng JOIN hoặc include; và chỉ tới bước cuối mình mới tính đến cache hay denormalize vì đó là giải pháp đắt và phức tạp nhất. Quan trọng là sau mỗi thay đổi mình đo lại để chắc chắn đúng nguyên nhân chứ không sửa lung tung. Một mẹo nhỏ trong Jira Clone là mình hay dùng partial index, ví dụ index chỉ trên những issue chưa bị xóa mềm, để index gọn và hiệu quả hơn."

---

- **Đo trước, đoán sau**: bật query log (Prisma `log:['query']` / TypeORM `maxQueryExecutionTime`) tìm đúng query chậm, rồi `EXPLAIN ANALYZE`.
- **Đọc plan**: Seq Scan trên bảng lớn? actual ≫ estimated (thống kê cũ)? node nào tốn thời gian nhất?
- **Sửa theo ưu tiên**: (1) thêm/chỉnh **index** composite theo đúng thứ tự `WHERE` rồi `ORDER BY`; (2) viết lại điều kiện **sargable** (bỏ hàm bọc cột); (3) **giảm dữ liệu** trả về (chỉ `SELECT` cột cần, phân trang **keyset**); (4) xử lý **N+1** bằng JOIN/`include` (xem 19.5); (5) cuối cùng mới tính cache/denormalize.
- Đo lại sau mỗi thay đổi để xác nhận đúng nguyên nhân.

```sql
-- Vd: query loc + sort -> index phu hop
CREATE INDEX idx_issue_proj_created ON issue (project_id, created_at DESC)
  WHERE deleted_at IS NULL;   -- partial: bo qua issue da xoa mem
```

</details>

<details>
<summary><b>31.18 Vài "gotcha" SQL hay bị hỏi để bẫy?</b></summary>

**📌 Câu trả lời mẫu:**

"Mấy cái bẫy SQL hay gặp thường xoay quanh NULL và JOIN. Về NULL, mình hay nhắc tới việc hàm aggregate như AVG hay COUNT(col) sẽ bỏ qua dòng NULL, nên nếu muốn coi NULL là 0 thì phải COALESCE trước; rồi NOT IN với subquery có NULL sẽ trả về rỗng vì cách so sánh ba trạng thái của SQL, nên mình ưu tiên NOT EXISTS cho an toàn. Về JOIN, cái bẫy kinh điển là LEFT JOIN nhưng lại đặt điều kiện của bảng bên phải trong WHERE — như vậy vô tình lọc mất dòng NULL và biến nó thành INNER JOIN, đúng ra phải để điều kiện đó trong ON. Một bẫy nữa là JOIN một-nhiều làm nhân dòng lên, ví dụ trong Jira Clone một issue có nhiều comment thì SUM(story_points) bị cộng trùng, lúc đó mình phải gom comment riêng hoặc dùng subquery. Cuối cùng là đừng bao giờ tin vào 'thứ tự tự nhiên' của dữ liệu — không có ORDER BY thì thứ tự trả về hoàn toàn không được đảm bảo. Bản chất mấy lỗi này đều khó phát hiện vì query vẫn chạy không báo lỗi, chỉ ra kết quả sai âm thầm, nên mình luôn để ý kỹ khi có NULL hoặc quan hệ một-nhiều."

---

- `COUNT(*)` vs `COUNT(col)` vs `COUNT(DISTINCT col)`: đếm mọi dòng / dòng non-NULL / giá trị duy nhất.
- `AVG(col)` **bỏ NULL** khỏi cả tử lẫn mẫu — muốn coi NULL là 0 phải `AVG(COALESCE(col,0))`.
- `NOT IN (subquery)` mà subquery chứa NULL → trả **rỗng**; dùng `NOT EXISTS` thay thế.
- LEFT JOIN nhưng đặt điều kiện bảng phải trong `WHERE` → biến thành INNER JOIN (mất dòng NULL).
- JOIN to-many làm **nhân dòng** → `SUM`/`COUNT` bị thổi phồng; cần group hoặc đếm distinct.
- `ORDER BY` không có thì **thứ tự không đảm bảo** — đừng dựa vào "thứ tự tự nhiên".

```sql
-- Bay nhan dong: moi issue co nhieu comment -> SUM(story_points) bi nhan len
-- Sai:
SELECT i.project_id, SUM(i.story_points)
FROM issue i JOIN comment c ON c.issue_id = i.id
GROUP BY i.project_id;
-- Dung: tong hop comment rieng, hoac SUM(DISTINCT...)/subquery
```

</details>
