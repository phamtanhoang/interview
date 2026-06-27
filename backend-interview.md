# 📚 Interview Prep — Node.js / NestJS (Mid/Senior)

> Tổng hợp câu hỏi phỏng vấn theo chủ đề, đúc kết trong quá trình xây dự án **Blog/CMS API**.
> Bấm vào từng câu hỏi để mở đáp án + ví dụ. File này được làm giàu dần sau mỗi bài học.

## Mục lục
1. [Node.js Core & Event Loop](#1-nodejs-core--event-loop)
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
21. [TypeScript for Backend](#21-typescript-for-backend)
22. [Advanced Testing](#22-advanced-testing)
23. [System Design and Scaling](#23-system-design-and-scaling)
24. [API and REST Design](#24-api-and-rest-design)
25. [Scheduled and Background Jobs](#25-scheduled-and-background-jobs)
26. [Worked Examples for Core Concepts](#26-worked-examples-for-core-concepts)
27. [GraphQL và Realtime trong NestJS](#27-graphql-và-realtime-trong-nestjs)
28. [Tư duy và Giải quyết vấn đề (Backend)](#28-tư-duy-và-giải-quyết-vấn-đề-backend)
29. [Kỹ năng mềm và Câu hỏi hành vi (Backend)](#29-kỹ-năng-mềm-và-câu-hỏi-hành-vi-backend)
30. [MongoDB và Mongoose (NoSQL)](#30-mongodb-và-mongoose-nosql)
31. [Git Workflow và Cộng tác](#31-git-workflow-và-cộng-tác)
32. [CI/CD và Triển khai (Docker, Cloud)](#32-cicd-và-triển-khai-docker-cloud)
33. [Tích hợp Third-party và Khả năng chống chịu](#33-tích-hợp-third-party-và-khả-năng-chống-chịu)

---
## 1. Node.js Core & Event Loop

<details>
<summary><b>1.1 Node.js là gì? Khác gì với JavaScript chạy trên trình duyệt?</b></summary>

Node.js là một **runtime** cho phép chạy JavaScript bên ngoài trình duyệt, xây trên engine **V8** (của Chrome) + thư viện **libuv** (cung cấp event loop & thread pool cho I/O bất đồng bộ) + **bindings** (cầu nối JS ↔ C++).

Khác trình duyệt: Node có các API hệ thống (`fs`, `http`, `process`, `Buffer`...) thay cho `window`, `document`, `DOM`. Mô hình nổi bật: **single-threaded, non-blocking I/O, event-driven** → rất hợp ứng dụng I/O-bound (API, realtime).
</details>

<details>
<summary><b>1.2 Node single-threaded mà sao xử lý được hàng nghìn request đồng thời?</b></summary>

JavaScript chạy trên **một thread chính**, nhưng các tác vụ **I/O** (đọc DB, gọi API, đọc file) được giao cho hệ điều hành / thread pool của **libuv** xử lý nền, **không chặn** thread chính. Khi I/O xong, callback được đẩy vào hàng đợi và **Event Loop** chạy chúng khi thread chính rảnh.

Ví dụ ví von: một đầu bếp bỏ mì vào nồi rồi quay sang phục vụ khách khác thay vì đứng đợi → một thread phục vụ được rất nhiều request.
</details>

<details>
<summary><b>1.3 Thứ tự chạy: process.nextTick vs Promise.then vs setTimeout vs setImmediate?</b></summary>

Quy luật: **code đồng bộ chạy hết trước** → rồi tới **microtask** (`process.nextTick` ưu tiên cao nhất → sau đó `Promise` callbacks) → cuối cùng tới **macrotask** (`setTimeout`, `setImmediate`, I/O). Sau **mỗi** macrotask, dọn sạch toàn bộ microtask rồi mới sang macrotask kế.

```js
console.log('A');                       // 1 - đồng bộ
setTimeout(() => console.log('E'), 0);  // macrotask
setImmediate(() => console.log('F'));   // macrotask
Promise.resolve().then(() => console.log('D')); // microtask
process.nextTick(() => console.log('C'));        // microtask ưu tiên cao
console.log('B');                       // 2 - đồng bộ
// In ra: A, B, C, D, rồi E/F (thứ tự E/F không xác định ở main module)
```
</details>

<details>
<summary><b>1.4 Khi nào KHÔNG nên dùng Node.js?</b></summary>

Với tác vụ **CPU-bound** nặng (xử lý ảnh/video, mã hoá, tính toán lớn) — vì chúng **chặn event loop** (thread duy nhất), làm toàn bộ server đứng. Giải pháp: Worker Threads, child process, hàng đợi job, hoặc dùng ngôn ngữ khác cho phần nặng đó.
</details>

<details>
<summary><b>1.5 Microtask vs Macrotask?</b></summary>

**Microtask** (`process.nextTick`, `Promise` callbacks): ưu tiên cao, được chạy **hết** sau mỗi tác vụ đồng bộ và sau mỗi macrotask. **Macrotask** (`setTimeout`, `setInterval`, `setImmediate`, I/O callbacks): chạy theo các pha của event loop, mỗi vòng một (vài) cái.
</details>

---

## 2. NestJS Architecture & Dependency Injection

<details>
<summary><b>2.1 Dependency Injection (DI) là gì? Lợi ích?</b></summary>

DI = class chỉ **khai báo** thứ nó cần (qua constructor), framework tự tạo và "tiêm" vào — thay vì class tự `new`. NestJS có **IoC Container** giữ sẵn instance của mỗi provider.

Lợi ích: **Inversion of Control** (đảo quyền tạo dependency cho framework), **loose coupling**, và quan trọng nhất là **dễ test** (tiêm bản mock thay bản thật).

```ts
constructor(private readonly usersService: UsersService) {}
// Không cần new UsersService() — Nest tự cấp.
```
</details>

<details>
<summary><b>2.2 Phân biệt Controller và Service/Provider?</b></summary>

**Controller** xử lý HTTP & routing — "mỏng", chỉ nhận request và trả response. **Service (Provider)** chứa **business logic** — "dày". Nguyên tắc: *Controller mỏng, Service dày* → tách bạch, dễ test, dễ tái dùng.
</details>

<details>
<summary><b>2.3 Module trong NestJS để làm gì?</b></summary>

Module là "chiếc hộp" gom nhóm các thành phần liên quan: `controllers`, `providers`, `imports` (mượn module khác), `exports` (cho module khác dùng provider của mình). App Nest = tập hợp nhiều module.
</details>

<details>
<summary><b>2.4 Vì sao phải khai báo provider trong mảng <code>providers</code>?</b></summary>

`providers: [...]` chính là động tác **đăng ký với IoC Container**. Không đăng ký → container không có instance → không inject được → lỗi `Nest can't resolve dependencies of ...`. Đây là lỗi phổ biến nhất với người mới.
</details>

<details>
<summary><b>2.5 Module encapsulation: làm sao dùng provider của module khác?</b></summary>

Provider chỉ "sống" trong module khai báo nó. Muốn module B dùng provider của module A: **A `exports` provider** đó, **B `imports` module A**. (Ví dụ thực tế: `UsersModule` exports `UsersService` để `AuthModule` import & dùng.)
</details>

<details>
<summary><b>2.6 Scope mặc định của provider là gì?</b></summary>

**Singleton** (`DEFAULT`) — mỗi provider chỉ được tạo **một lần** và dùng chung toàn app. Ngoài ra có scope `REQUEST` (mỗi request một instance) và `TRANSIENT` (mỗi nơi inject một instance).
</details>

<details>
<summary><b>2.7 NestJS biết kiểu dữ liệu để inject bằng cách nào?</b></summary>

Nhờ **decorator metadata** + thư viện `reflect-metadata`, được bật bởi `emitDecoratorMetadata: true` và `experimentalDecorators: true` trong `tsconfig.json`. TypeScript "phát" thông tin kiểu của tham số constructor để Nest đọc và resolve đúng provider.
</details>

---

## 3. Configuration & Environment

<details>
<summary><b>3.1 Quản lý biến môi trường/secret trong NestJS thế nào?</b></summary>

Dùng `@nestjs/config` (`ConfigModule`) đọc file **`.env`** (được **gitignore**), truy cập qua `ConfigService.get('KEY')`. Best practice: thêm `.env.example` (chỉ tên biến, không giá trị) để commit làm mẫu. **Không hardcode** secret vào code.
</details>

<details>
<summary><b>3.2 <code>forRoot</code> vs <code>forRootAsync</code> khác gì?</b></summary>

`forRoot({...})`: cấu hình **tĩnh**, có sẵn lúc viết. `forRootAsync({...})`: cấu hình **phụ thuộc thứ phải nạp trước** (vd `ConfigService`) → dùng `useFactory` + `inject` để lấy giá trị bất đồng bộ.

```ts
TypeOrmModule.forRootAsync({
  inject: [ConfigService],
  useFactory: (cfg: ConfigService) => ({ /* ... */ }),
});
```
</details>

<details>
<summary><b>3.3 <code>synchronize: true</code> làm gì? Có nên bật ở production?</b></summary>

Tự động `CREATE/ALTER` bảng cho khớp entity **mỗi lần khởi động**. ✅ tiện khi dev. 🔴 **KHÔNG** dùng production — có thể **xoá cột → mất dữ liệu thật**. Production dùng **migrations** (script thay đổi schema có kiểm soát, review & rollback được).
</details>

<details>
<summary><b>3.4 Biến môi trường được đọc khi nào? Sửa <code>.env</code> lúc app đang chạy có ăn không?</b></summary>

Đọc **một lần** lúc tiến trình khởi động (`ConfigModule.forRoot()` lúc bootstrap). Sửa `.env` khi app đang chạy **KHÔNG** có hiệu lực — `nest start --watch` chỉ theo dõi file `.ts`, không reload `.env`. Phải **restart** app.
</details>

---

## 4. Database, TypeORM & Entities

<details>
<summary><b>4.1 Entity là gì?</b></summary>

Class TypeScript được gắn decorator để ánh xạ tới một **bảng** DB: 1 class → 1 bảng, 1 property → 1 cột, 1 instance → 1 dòng. TypeORM đọc decorator (qua `reflect-metadata`) để suy ra schema.
</details>

<details>
<summary><b>4.2 Active Record vs Data Mapper? NestJS dùng cái nào?</b></summary>

**Active Record**: entity tự lo lưu (`user.save()`), logic nằm trên model. **Data Mapper**: `Repository` riêng lo việc lưu (`repo.save(user)`), entity là data thuần.

TypeORM hỗ trợ cả hai; với NestJS dùng **Data Mapper / Repository pattern** vì hợp **DI** (inject `Repository`), tách logic, dễ test.
</details>

<details>
<summary><b>4.3 UUID vs auto-increment (serial) cho khóa chính?</b></summary>

- **UUID**: toàn cục duy nhất, khó đoán (an toàn, chống dò ID), hợp hệ phân tán. Nhược: tốn chỗ hơn (16 byte), không tuần tự (dễ phân mảnh index — `uuidv7` khắc phục).
- **Serial/auto-increment**: nhỏ gọn, tuần tự, nhanh. Nhược: dễ đoán, khó merge giữa nhiều DB.
</details>

<details>
<summary><b>4.4 <code>forRoot</code> vs <code>forFeature</code> trong TypeOrmModule?</b></summary>

`forRootAsync` (ở AppModule): cấu hình **kết nối** DB (1 lần, global). `forFeature([Entity])` (ở module con): đăng ký **repository** cho entity cụ thể → để inject `Repository<Entity>` trong service của module đó.
</details>

<details>
<summary><b>4.5 Vì sao đặt <code>select: false</code> cho cột password?</b></summary>

Để mặc định các truy vấn `SELECT` (`find`, `findOne`) **không trả** cột password → tránh rò rỉ hash mật khẩu ra response. Khi *cần* (lúc login), dùng QueryBuilder `.addSelect('user.password')` để lấy ra.

> Kiểm chứng thực tế (TypeORM bản này): response tạo user **không** lộ password — `select: false` đã đủ (entity trả về từ `save()` cũng không kèm cột này). `@Exclude` + `ClassSerializerInterceptor` vẫn là **lớp phòng thủ thứ hai** hữu ích cho trường hợp bạn **chủ động** `addSelect('password')` (như khi login) rồi lỡ trả object đó ra response.
</details>

<details>
<summary><b>4.6 <code>@CreateDateColumn</code>/<code>@UpdateDateColumn</code> khác cột Date thường?</b></summary>

TypeORM **tự** quản lý giá trị: `createdAt` set lúc INSERT, `updatedAt` tự cập nhật mỗi lần `save()`. Không cần gán tay.
</details>

<details>
<summary><b>4.7 Cột enum trong Postgres có gì đặc biệt?</b></summary>

Postgres tạo một **kiểu enum riêng** (vd `users_role_enum`). Muốn thêm/bớt giá trị enum sau này thường phải qua **migration** (không sửa trực tiếp dễ như varchar).
</details>

<details>
<summary><b>4.8 Repository pattern là gì?</b></summary>

Lớp trừu tượng cho việc truy cập dữ liệu. Ta inject `Repository<User>` và gọi `find/save/...` thay vì viết SQL thô → dễ thay thế, dễ **mock** khi test.
</details>

<details>
<summary><b>4.9 <code>@InjectRepository</code> hoạt động được nhờ đâu?</b></summary>

Nhờ `TypeOrmModule.forFeature([Entity])` đã đăng ký **repository token** cho entity đó vào DI container của module. Thiếu `forFeature` → lỗi `Nest can't resolve dependencies`.
</details>

<details>
<summary><b>4.10 <code>create()</code> vs <code>save()</code> trong Repository?</b></summary>

`repo.create(data)`: chỉ tạo **instance trong bộ nhớ** (áp default), chưa đụng DB. `repo.save(entity)`: mới thực sự ghi DB (`INSERT` hoặc `UPDATE` nếu có id).
</details>

<details>
<summary><b>4.11 <code>preload</code> làm gì? <code>remove</code> vs <code>delete</code>?</b></summary>

`preload({ id, ...dto })`: tải bản ghi theo id rồi **merge** dữ liệu mới, trả entity đã merge (hoặc `undefined` nếu không có) → tiện cho PATCH + check 404.

`remove(entity)`: cần entity đã load, chạy lifecycle hook. `delete(criteria)`: xoá thẳng theo điều kiện/id, nhanh hơn, không cần load.
</details>

<details>
<summary><b>4.12 Soft delete là gì?</b></summary>

Không xoá hẳn dòng dữ liệu, chỉ set cột `deletedAt` (dùng `@DeleteDateColumn` + `softRemove`/`softDelete`). Giữ được lịch sử và khôi phục được; các query mặc định tự bỏ qua bản ghi đã soft-delete.
</details>

---

## 5. DTO, Validation & Pipes

<details>
<summary><b>5.1 DTO khác Entity thế nào?</b></summary>

**DTO** (Data Transfer Object) mô tả hình dạng + luật của dữ liệu **vào/ra** (hợp đồng API). **Entity** ánh xạ **bảng DB**. Tách biệt để API và DB không phụ thuộc cứng vào nhau (đổi DB không vỡ API, và ngược lại).
</details>

<details>
<summary><b>5.2 Pipe trong NestJS là gì? Chạy lúc nào?</b></summary>

Pipe **biến đổi / validate** dữ liệu đầu vào, chạy **trước handler** (sau Guard) trong request lifecycle. `ValidationPipe` dùng `class-validator` + `class-transformer` để kiểm tra DTO.

```ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true, forbidNonWhitelisted: true, transform: true,
}));
```
</details>

<details>
<summary><b>5.3 <code>whitelist</code> vs <code>forbidNonWhitelisted</code> vs <code>transform</code>?</b></summary>

- `whitelist`: **âm thầm xoá** field không khai báo trong DTO.
- `forbidNonWhitelisted`: gửi field lạ → **báo lỗi 400**.
- `transform`: chuyển payload JSON thô thành **instance DTO** + ép kiểu (vd `"5"` → `5`).
</details>

<details>
<summary><b>5.4 <code>PartialType</code> dùng để làm gì?</b></summary>

`PartialType(CreateUserDto)` kế thừa **toàn bộ luật validation** nhưng biến mọi field thành **optional** — hợp cho `PATCH` (cập nhật một phần). Giúp **DRY**, không lặp lại các `@IsEmail`, `@MinLength`...
</details>

<details>
<summary><b>5.5 Built-in pipes (<code>ParseUUIDPipe</code>, <code>ParseIntPipe</code>) — dùng khi nào?</b></summary>

Là pipe dựng sẵn để **validate & ép kiểu tham số** (param/query) trước handler. `@Param('id', ParseUUIDPipe)` đảm bảo `id` là UUID hợp lệ; sai → **400 Bad Request** sạch sẽ. Tương tự `ParseIntPipe`, `ParseBoolPipe`, `ParseEnumPipe`.
</details>

<details>
<summary><b>5.6 Vì sao truyền id sai định dạng lại ra 500 thay vì 404?</b></summary>

Nếu param (vd id) **không đúng kiểu cột** (truyền chuỗi không phải UUID vào cột `uuid`), truy vấn **vỡ ngay ở tầng DB** (`invalid input syntax for type uuid`) → 500, trước khi tới logic "không tìm thấy → 404". Khắc phục: `ParseUUIDPipe` chặn param sai → 400 trước khi chạm DB.
</details>

---

## 6. REST API & Controllers

<details>
<summary><b>6.1 Các decorator routing cơ bản?</b></summary>

`@Controller('users')` đặt tiền tố route; `@Get()`, `@Post()`, `@Patch(':id')`, `@Delete(':id')` ứng với HTTP method; `@Body()` lấy body, `@Param('id')` lấy param URL, `@Query()` lấy query string.
</details>

<details>
<summary><b>6.2 PUT vs PATCH?</b></summary>

**PUT** thay **toàn bộ** resource (gửi đủ mọi field). **PATCH** cập nhật **một phần** (chỉ field cần đổi). Trong Nest thường PATCH + `PartialType`.
</details>

<details>
<summary><b>6.3 Các HTTP exception dựng sẵn của Nest?</b></summary>

`NotFoundException` (404), `BadRequestException` (400), `UnauthorizedException` (401), `ForbiddenException` (403), `ConflictException` (409)... Ném ra là Nest tự trả status + JSON tương ứng. Xoá thành công thường trả `204 No Content` (dùng `@HttpCode(204)`).
</details>

<details>
<summary><b>6.4 Nested route (route lồng nhau) dùng khi nào?</b></summary>

Khi một tài nguyên **phụ thuộc** tài nguyên khác — vd comment thuộc về post: `@Controller('posts/:postId/comments')`. URL phản ánh quan hệ theo chuẩn REST; lấy id cha qua `@Param('postId')`. `POST /posts/<id>/comments` = thêm comment vào bài đó.
</details>

---

## 7. Authentication & Authorization

<details>
<summary><b>7.1 Authentication vs Authorization?</b></summary>

**Authentication (AuthN)** = "bạn **là ai**?" (đăng nhập đúng email/mật khẩu). **Authorization (AuthZ)** = "bạn **được phép** làm gì?" (vd chỉ admin xoá bài). Luôn AuthN trước, AuthZ sau.
</details>

<details>
<summary><b>7.2 Vì sao hash mật khẩu? bcrypt khác MD5/SHA thế nào?</b></summary>

Để nếu DB lộ thì mật khẩu không "trần trụi". Hash là **một chiều** (không đảo ngược). bcrypt khác MD5/SHA ở chỗ: **có salt** (chống rainbow table) và **chậm có chủ đích** (cost factor → chống brute-force). MD5/SHA quá nhanh → không hợp cho mật khẩu. (Lựa chọn khác: `argon2`.)
</details>

<details>
<summary><b>7.3 Salt là gì?</b></summary>

Chuỗi ngẫu nhiên thêm vào mỗi mật khẩu trước khi hash → 2 người cùng mật khẩu `123456` vẫn ra 2 hash khác nhau → vô hiệu hoá rainbow table. bcrypt tự sinh & nhúng salt vào chuỗi hash.
</details>

<details>
<summary><b>7.4 Khi login có "giải mã" mật khẩu không?</b></summary>

Không. Hash là một chiều. Ta dùng `bcrypt.compare(plainNhập, hashTrongDB)` — nó hash lại mật khẩu nhập (với salt lấy từ hash) rồi so khớp.
</details>

<details>
<summary><b>7.5 JWT gồm mấy phần? Vì sao gọi là stateless?</b></summary>

3 phần ngăn bởi dấu `.`: **Header** (thuật toán) . **Payload** (dữ liệu: `sub`, `role`, `exp`...) . **Signature** (chữ ký bằng secret). Server chỉ cần **verify chữ ký** bằng secret là biết user là ai → **không cần lưu session** ⇒ stateless.
</details>

<details>
<summary><b>7.6 Nên để gì trong JWT payload?</b></summary>

Chỉ dữ liệu định danh tối thiểu: `sub` (id user — chuẩn JWT), `role`, có thể `email`. **Không** để dữ liệu nhạy cảm (mật khẩu, token khác) vì payload chỉ **base64**, ai cũng giải mã đọc được (chỉ chữ ký mới được bảo vệ).
</details>

<details>
<summary><b>7.7 Vì sao message lỗi đăng nhập nên mập mờ?</b></summary>

Trả "Email hoặc mật khẩu không đúng" (không nói rõ cái nào sai) để tránh **lộ email nào tồn tại** trong hệ thống (chống dò tài khoản / user enumeration).
</details>

<details>
<summary><b>7.8 Guard là gì? Chạy lúc nào trong request lifecycle?</b></summary>

Guard quyết định request có được đi tiếp không (`canActivate()` trả `boolean`). Thứ tự: **Middleware → Guard → Interceptor → Pipe → Handler**. Guard hợp cho **AuthN/AuthZ**.

```ts
@UseGuards(JwtAuthGuard)
@Get('profile')
getProfile(@CurrentUser() user) { return user; }
```
</details>

<details>
<summary><b>7.9 <code>AuthGuard('jwt')</code> kết nối với <code>JwtStrategy</code> thế nào?</b></summary>

Qua **tên strategy** `'jwt'` (mặc định của `passport-jwt`). Khi guard kích hoạt, Passport chạy strategy: lấy token từ header `Authorization: Bearer`, verify chữ ký + hạn, rồi gọi `validate(payload)`. Giá trị `validate` trả về được gắn vào **`request.user`**.
</details>

<details>
<summary><b>7.10 <code>validate()</code> trong strategy trả gì? Nó đi đâu?</b></summary>

Passport gọi `validate(payload)` **sau khi** token đã verify xong (chữ ký đúng + còn hạn). Giá trị mà `validate` **return** sẽ được Nest gắn vào **`request.user`** → các handler và decorator (`@CurrentUser()`, `@Req()`) lấy ra dùng.

```ts
validate(payload: JwtPayload) {
  return { id: payload.sub, email: payload.email, role: payload.role };
  // => request.user = { id, email, role }
}
```
</details>

<details>
<summary><b>7.11 Guard khác Middleware thế nào?</b></summary>

Guard biết **ngữ cảnh Nest** (`ExecutionContext` — handler/class nào, metadata gì) → hợp cho phân quyền theo route/role. Middleware là hàm Express thuần, chạy sớm hơn, không biết handler nào sẽ xử lý.
</details>

<details>
<summary><b>7.12 JWT hết hạn thì sao?</b></summary>

Với `ignoreExpiration: false`, Passport tự trả **401** khi token quá hạn (`exp`). Client phải đăng nhập lại — hoặc dùng cơ chế **refresh token** (access token ngắn hạn + refresh token dài hạn) để cấp lại mà không bắt login.
</details>

<details>
<summary><b>7.13 <code>JwtModule.registerAsync</code> vs <code>register</code>?</b></summary>

`register` khi cấu hình tĩnh; `registerAsync` khi cần inject (vd lấy `secret`/`expiresIn` từ `ConfigService` qua `useFactory`/`inject`).
</details>

<details>
<summary><b>7.14 RBAC là gì?</b></summary>

**Role-Based Access Control** — kiểm soát truy cập dựa trên **vai trò** của user (admin/author/reader...). Mỗi route khai báo role được phép; user có role phù hợp mới qua. Là hình thức **Authorization** phổ biến nhất.
</details>

<details>
<summary><b>7.15 <code>@Roles</code> + <code>RolesGuard</code> phối hợp thế nào?</b></summary>

`@Roles('admin')` dùng `SetMetadata` **gắn nhãn** role yêu cầu lên handler. `RolesGuard` dùng `Reflector` **đọc lại** nhãn đó rồi so với `request.user.role` (do JWT strategy gắn). Khớp → cho qua; không → **403**.

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
<summary><b>7.16 <code>Reflector</code> là gì?</b></summary>

Tiện ích của NestJS để **đọc metadata** mà các decorator (như `@Roles`, `@SetMetadata`) gắn lên handler/class. Method hay dùng: `get`, `getAllAndOverride` (đọc cả handler lẫn class, method ghi đè class), `getAllAndMerge`.
</details>

<details>
<summary><b>7.17 Thứ tự nhiều Guard trong <code>@UseGuards(A, B)</code>?</b></summary>

Chạy **lần lượt** từ trái sang phải: A trước, B sau. Với auth phải để `JwtAuthGuard` **trước** `RolesGuard` — vì `RolesGuard` cần `request.user` (do JwtAuthGuard/strategy gắn) mới biết role để kiểm tra.
</details>

<details>
<summary><b>7.18 Vì sao không nên cho user tự đăng ký role <code>admin</code>?</b></summary>

Đó là lỗ hổng **privilege escalation** (leo thang đặc quyền) — ai cũng tự thành admin. Role nên do **hệ thống** quyết định (mặc định `reader`), admin được tạo qua **seed/migration** hoặc do một admin khác cấp; không để `role` trong DTO đăng ký công khai.
</details>

<details>
<summary><b>7.19 RBAC vs ABAC (ownership-based)?</b></summary>

**RBAC** xét **vai trò** (admin/author/reader). **ABAC / ownership-based** xét **thuộc tính của chính tài nguyên** — ví dụ "chỉ tác giả của bài mới được sửa bài đó" (`post.author.id === user.id`). ABAC linh hoạt hơn cho các luật phụ thuộc ngữ cảnh; thường kết hợp cả hai (admin vượt quyền + chủ sở hữu).
</details>

<details>
<summary><b>7.20 401 Unauthorized vs 403 Forbidden?</b></summary>

**401** = **chưa xác thực** (chưa đăng nhập / token sai/thiếu/hết hạn). **403** = **đã xác thực nhưng không đủ quyền** (sai role, không phải chủ tài nguyên). Nhớ: 401 về *danh tính*, 403 về *quyền hạn*.
</details>

<details>
<summary><b>7.21 Nên đặt kiểm tra ownership ở Guard hay Service?</b></summary>

- **Guard**: tách bạch & tái sử dụng tốt, nhưng phải **tự load tài nguyên** (truy vấn DB) trong guard để biết chủ sở hữu.
- **Service**: tiện khi handler đã load sẵn tài nguyên (vd `findOne` rồi so `author.id`); ít lặp truy vấn.

Không có đáp án "đúng tuyệt đối" — nêu được **trade-off** (tái sử dụng vs đơn giản, số lần query) là điểm cộng ở vòng Senior.
</details>

---

## 8. Docker & Debugging thực chiến

<details>
<summary><b>8.1 Ý nghĩa <code>5433:5432</code> trong Docker? Hai service cùng đòi 1 cổng thì sao?</b></summary>

Định dạng **`HOST:CONTAINER`** — trái là cổng trên máy host, phải là cổng trong container. Nếu hai service cùng giành một cổng host → **xung đột**, một bên không bind được. Cách xử lý: đổi cổng host (vd map `5433:5432`) hoặc tắt service kia.

> Tình huống thực: máy có PostgreSQL native chiếm 5432 → Docker container phải map ra `5433` để app không "đụng nhầm".
</details>

<details>
<summary><b>8.2 Làm sao biết tiến trình nào đang chiếm một cổng?</b></summary>

- Windows: `Get-NetTCPConnection -LocalPort 5432` hoặc `netstat -ano | findstr :5432` → lấy PID → tra process.
- Linux/Mac: `lsof -i :5432`.
</details>

<details>
<summary><b>8.3 Dữ liệu trong Docker nằm ở đâu? <code>down</code> vs <code>down -v</code>?</b></summary>

Dữ liệu nằm trong **named volume** (container có thể xoá/tạo lại, nhưng volume giữ state). `docker compose down` xoá container + network nhưng **giữ volume** (dữ liệu còn). `docker compose down -v` xoá **cả volume** → **mất sạch dữ liệu**. Cẩn thận với `-v` và với việc xoá nhầm volume của project khác.
</details>

<details>
<summary><b>8.4 Cách đọc lỗi để khoanh vùng (debug mindset)?</b></summary>

Đọc kỹ thông báo: `ECONNREFUSED` = không tới được server (mạng/cổng); `password authentication failed (28P01)` = tới được nhưng sai thông tin đăng nhập. **Cô lập biến số** (test trực tiếp DB, test parse config, kiểm tra env hệ thống) rồi **suy ra cái còn lại**. Đây là tư duy nhà tuyển dụng Mid/Senior muốn thấy.
</details>

<details>
<summary><b>8.5 npm vs pnpm? <code>node_modules</code> nằm ở đâu khi cài?</b></summary>

`npm/pnpm install` tác động lên `node_modules` + `package.json` của **thư mục hiện tại**. Cài sai thư mục → package lạc chỗ, `package.json` của project thiếu dependency (người khác clone về chạy lỗi). pnpm dùng cấu trúc **strict** (deps gián tiếp nằm trong `node_modules/.pnpm`), tiết kiệm ổ đĩa; phải dùng `pnpm add` để giữ lockfile nhất quán.
</details>

---

## 9. TypeORM Relations

<details>
<summary><b>9.1 One-to-Many / Many-to-One: khóa ngoại (FK) nằm ở bảng nào?</b></summary>

Luôn ở phía **"nhiều" (Many)**. Ví dụ User (1) ─< Post (N): bảng `posts` giữ cột FK `authorId` trỏ về `users.id`. (Một user nhiều bài → không thể nhét nhiều id vào 1 dòng user; mỗi bài 1 tác giả → để FK ở posts là hợp lý.)
</details>

<details>
<summary><b>9.2 <code>@ManyToOne</code> vs <code>@OneToMany</code> — cái nào tạo cột trong bảng?</b></summary>

Chỉ **`@ManyToOne`** (phía Many) sinh cột FK thật trong bảng. `@OneToMany` **không tạo cột nào** — nó chỉ là property "ảo" để truy cập ngược (vd `user.posts`).

```ts
// posts table có cột authorId
@ManyToOne(() => User, (u) => u.posts, { onDelete: 'CASCADE' })
author: User;

// users table KHÔNG có cột mới
@OneToMany(() => Post, (p) => p.author)
posts: Post[];
```
</details>

<details>
<summary><b>9.3 Vì sao quan hệ dùng arrow function <code>() => User</code>?</b></summary>

Để tham chiếu kiểu một cách **"lười" (lazy)** — tránh lỗi **circular dependency** khi hai entity import lẫn nhau (User cần Post, Post cần User). Arrow function trì hoãn việc đọc kiểu tới lúc runtime, nên thứ tự load không gây `undefined`.
</details>

<details>
<summary><b>9.4 <code>onDelete</code> có những lựa chọn nào?</b></summary>

- `CASCADE`: xoá bản ghi cha → xoá kèm bản ghi con (xoá user → xoá bài của họ).
- `SET NULL`: set FK về null (bài "mồ côi", author = null).
- `RESTRICT` / `NO ACTION`: cấm xoá cha nếu còn con tham chiếu.
</details>

<details>
<summary><b>9.5 Eager vs Lazy loading? Vấn đề N+1 là gì? <i>(sẽ học kỹ ở bài truy vấn quan hệ)</i></b></summary>

**Eager**: tự động load quan hệ kèm theo mỗi query. **Lazy**: chỉ load khi truy cập (trả Promise). Mặc định TypeORM không load quan hệ trừ khi khai báo `eager: true` hoặc dùng `relations: ['author']` / QueryBuilder `leftJoinAndSelect`.

**N+1**: lấy N bài rồi lặp truy vấn tác giả cho từng bài → 1 + N query → chậm. Khắc phục bằng JOIN (`leftJoinAndSelect`) hoặc `relations` để lấy 1 lần.
</details>

<details>
<summary><b>9.6 Làm sao load quan hệ khi truy vấn?</b></summary>

3 cách: (1) option `relations: ['author']` trong `find`/`findOne`; (2) QueryBuilder `.leftJoinAndSelect('post.author', 'author')`; (3) đặt `eager: true` trên decorator quan hệ (luôn tự load — dùng thận trọng vì load cả khi không cần). Cả (1) và (2) đều dịch ra JOIN → lấy 1 query, tránh N+1.

```ts
this.postRepository.find({ relations: ['author'] });
```
</details>

<details>
<summary><b>9.7 Gán quan hệ chỉ bằng <code>{ id }</code> được không?</b></summary>

Được. Để set khóa ngoại, TypeORM chỉ cần **id** của bản ghi liên quan — không phải load cả entity:
```ts
const post = this.postRepository.create({ ...dto, author: { id: authorId } as User });
// -> chỉ ghi cột authorId, không query bảng users
```
</details>

<details>
<summary><b>9.8 (Security) Vì sao <code>authorId</code> phải lấy từ JWT, không từ body client gửi?</b></summary>

Chống **mạo danh**: nếu client tự gửi `authorId`, họ có thể tạo bài "nhân danh" người khác. Mọi dữ liệu **định danh chủ sở hữu** phải lấy từ **token đã xác thực** (`@CurrentUser()`), không tin body. Vì vậy `authorId` **không** nằm trong DTO.
</details>

<details>
<summary><b>9.9 Một entity có nhiều <code>@ManyToOne</code> được không?</b></summary>

Được. Mỗi `@ManyToOne` sinh một cột FK riêng. Ví dụ `Comment` thuộc về cả `User` lẫn `Post` → bảng `comments` có hai cột `authorId` và `postId`.
</details>

<details>
<summary><b>9.10 Lọc & sắp xếp theo quan hệ trong TypeORM?</b></summary>

Lọc theo quan hệ: `where: { post: { id: postId } }` (TypeORM dịch ra `WHERE postId = ...`). Sắp xếp: `order: { createdAt: 'DESC' }`. Kết hợp `relations` để kèm dữ liệu liên quan.

```ts
this.commentRepository.find({
  where: { post: { id: postId } },
  relations: ['author'],
  order: { createdAt: 'DESC' },
});
```
</details>

<details>
<summary><b>9.11 Quan hệ Many-to-Many lưu dữ liệu liên kết ở đâu?</b></summary>

Ở một **bảng nối (join table)** chứa 2 khóa ngoại — vd `posts_tags(postId, tagId)`, mỗi dòng là một cặp liên kết. Không thể nhét FK vào 2 bảng gốc vì cả hai phía đều "nhiều". TypeORM tự tạo bảng nối này.
</details>

<details>
<summary><b>9.12 <code>@ManyToMany</code> + <code>@JoinTable</code> — đặt ở đâu?</b></summary>

`@ManyToMany` đặt ở **cả hai** entity. `@JoinTable` chỉ ở **MỘT phía** — **owning side** (bên quyết định/sở hữu bảng nối). Phía kia là inverse side, chỉ có `@ManyToMany`.

```ts
// Post (owning side)
@ManyToMany(() => Tag, (t) => t.posts, { cascade: true })
@JoinTable({ name: 'posts_tags' })
tags: Tag[];

// Tag (inverse side)
@ManyToMany(() => Post, (p) => p.tags)
posts: Post[];
```
</details>

<details>
<summary><b>9.13 <code>cascade: true</code> để làm gì? Tránh tạo bản ghi trùng (find-or-create)?</b></summary>

`cascade: true`: lưu entity cha kèm entity liên quan **mới** → TypeORM tự lưu chúng + tạo liên kết (đỡ save thủ công). Tránh trùng: đặt cột định danh (`name`) là `unique` + pattern **find-or-create** — tìm theo name, có thì tái dùng, chưa thì tạo mới.
</details>

---

## 10. Interceptors & Serialization

<details>
<summary><b>10.1 Interceptor là gì? Chạy lúc nào trong request lifecycle?</b></summary>

Thành phần **bọc quanh handler** — chạy **cả trước và sau** handler, dùng **RxJS** để biến đổi luồng response (loại field, bọc dữ liệu, logging, đo thời gian, cache). Vị trí: `Guard → Interceptor(trước) → Pipe → Handler → Interceptor(sau) → Filter`.
</details>

<details>
<summary><b>10.2 <code>ClassSerializerInterceptor</code> + <code>@Exclude()</code> làm gì?</b></summary>

Interceptor dựng sẵn: tự **chuyển entity instance → plain object** khi trả response và tôn trọng decorator của `class-transformer` (`@Exclude` loại field, `@Expose` đổi tên/hiện field). Dùng để ẩn dữ liệu nhạy cảm (vd `password`) khỏi mọi response một cách tập trung.

```ts
@Column({ select: false })
@Exclude()
password: string;

// main.ts
app.useGlobalInterceptors(new ClassSerializerInterceptor(app.get(Reflector)));
```
</details>

<details>
<summary><b>10.3 <code>select: false</code> khác <code>@Exclude()</code> thế nào?</b></summary>

`select: false` (TypeORM) chặn ở tầng **truy vấn DB** — cột không được load lên trừ khi `addSelect`. `@Exclude()` (class-transformer) chặn ở tầng **serialize response** — không lộ ra ngoài kể cả khi field đang có trong bộ nhớ. Dùng cả hai = bảo vệ 2 lớp.
</details>

<details>
<summary><b>10.4 Khi nào <code>@Exclude()</code> KHÔNG có tác dụng?</b></summary>

Khi handler trả về **plain object** (vd `return { ...user }`) thay vì **instance của class entity**. `ClassSerializerInterceptor` cần instance để transform; plain object không mang metadata `@Exclude`.
</details>

---

## 11. Pagination, Filtering & QueryBuilder

<details>
<summary><b>11.1 Phân trang trong TypeORM thế nào?</b></summary>

`skip((page-1)*limit).take(limit)` + `getManyAndCount()` → trả `[data, total]`. Tính `totalPages = Math.ceil(total/limit)`. Response chuẩn: `{ data, total, page, limit, totalPages }`.

```ts
const [data, total] = await qb.skip((page-1)*limit).take(limit).getManyAndCount();
```
</details>

<details>
<summary><b>11.2 <code>skip</code>/<code>take</code> khác <code>offset</code>/<code>limit</code>?</b></summary>

`skip`/`take` phân trang ở **cấp entity** — khi có join quan hệ "nhiều" (to-many), TypeORM tự dùng subquery để đếm/lấy **đúng số entity**, tránh nhân dòng do JOIN. `offset`/`limit` là SQL thô, áp lên kết quả join → dễ sai số khi join to-many.
</details>

<details>
<summary><b>11.3 Offset pagination vs Cursor pagination?</b></summary>

**Offset** (`skip/take` theo page): đơn giản, nhảy trang tuỳ ý; nhược: chậm khi offset lớn (DB vẫn quét qua), và dễ **lệch/trùng** khi dữ liệu thêm/xoá giữa chừng. **Cursor** (dựa trên `id`/`createdAt` của bản ghi cuối): ổn định & nhanh với dữ liệu lớn/real-time; nhược: không nhảy trang bất kỳ. Mạng xã hội/feed thường dùng cursor.
</details>

<details>
<summary><b>11.4 QueryBuilder vs Repository <code>find()</code> — khi nào dùng cái nào?</b></summary>

`find()`/`findOne()` gọn cho query đơn giản (where, relations, order). **QueryBuilder** linh hoạt cho query động/phức tạp: thêm `andWhere` tuỳ điều kiện, join tuỳ biến, subquery, aggregate, raw SQL fragment (`ILIKE`, hàm DB). Trong API có nhiều filter tuỳ chọn → QueryBuilder dễ kiểm soát hơn.
</details>

<details>
<summary><b>11.5 Query param luôn là string — validate kiểu number/boolean thế nào?</b></summary>

Dùng `@Type(() => Number)` (class-transformer) + `transform: true` trong ValidationPipe để **ép kiểu** trước khi `@IsInt`/`@Min` kiểm tra. Với boolean dạng query: `@IsBooleanString()` (nhận `'true'`/`'false'`) rồi tự so `=== 'true'`.
</details>

---

## 12. Exception Filters & Error Handling

<details>
<summary><b>12.1 Exception Filter là gì? Chạy lúc nào?</b></summary>

Thành phần **bắt exception** ném ra trong app và định hình response trả về. Nằm **cuối** request lifecycle (kích hoạt khi có lỗi). Dùng để chuẩn hoá format lỗi + log tập trung.
</details>

<details>
<summary><b>12.2 <code>@Catch()</code> vs <code>@Catch(HttpException)</code>?</b></summary>

`@Catch()` (không tham số) bắt **mọi** exception. `@Catch(HttpException)` chỉ bắt loại được chỉ định (có thể truyền nhiều loại). Filter triển khai `ExceptionFilter` với method `catch(exception, host)`.
</details>

<details>
<summary><b>12.3 <code>ArgumentsHost</code> là gì?</b></summary>

Đối tượng truy cập **ngữ cảnh thực thi trung lập** (HTTP / WebSocket / RPC). Với HTTP: `host.switchToHttp().getRequest()/getResponse()`. Nhờ vậy filter/guard/interceptor dùng được trên nhiều loại transport.
</details>

<details>
<summary><b>12.4 Cấu trúc <code>HttpException</code>?</b></summary>

Có `getStatus()` (mã HTTP) và `getResponse()` (string hoặc object `{ statusCode, message, error }`; với lỗi validation, `message` là **mảng**). Các exception dựng sẵn (`NotFoundException`...) đều kế thừa `HttpException`. Lỗi không phải HttpException → nên coi là **500**.
</details>

<details>
<summary><b>12.5 Filter toàn cục: <code>useGlobalFilters</code> vs <code>APP_FILTER</code>?</b></summary>

`app.useGlobalFilters(new MyFilter())`: đơn giản nhưng **không inject** được dependency. Provider **`APP_FILTER`** (trong module) cho Nest quản lý qua DI → filter có thể inject service (vd gửi lỗi lên Sentry/logger ngoài). Tương tự có `APP_GUARD`, `APP_INTERCEPTOR`, `APP_PIPE`.
</details>

---

## 13. Testing (Jest)

<details>
<summary><b>13.1 Unit vs Integration vs E2E test?</b></summary>

**Unit**: test 1 đơn vị (vd service) **cô lập**, mock hết dependency → nhanh, ổn định. **Integration**: nhiều thành phần ghép, một phần thật (vd service + DB test). **E2E**: test cả luồng qua HTTP như user thật (supertest + app + DB test). Tháp test: nhiều unit, ít e2e.
</details>

<details>
<summary><b>13.2 Vì sao mock dependency trong unit test?</b></summary>

Để **cô lập** đơn vị cần test — không phụ thuộc DB/mạng/service khác → test nhanh, ổn định, dễ tái lập. Đây là lợi ích trực tiếp của **DI**: dependency tiêm vào được nên thay bằng bản giả (mock) dễ dàng.
</details>

<details>
<summary><b>13.3 <code>Test.createTestingModule</code> + <code>getRepositoryToken</code>?</b></summary>

`Test.createTestingModule({ providers: [...] }).compile()` dựng một DI container riêng cho test. Cung cấp repository giả bằng `{ provide: getRepositoryToken(Entity), useValue: mockRepo }` — đúng token mà `@InjectRepository(Entity)` dùng, nên service nhận bản giả.

```ts
const module = await Test.createTestingModule({
  providers: [UsersService, { provide: getRepositoryToken(User), useValue: mockRepo }],
}).compile();
```
</details>

<details>
<summary><b>13.4 <code>jest.fn()</code>, <code>mockResolvedValue</code>, <code>clearAllMocks</code>?</b></summary>

`jest.fn()`: hàm giả, ghi lại số lần gọi + tham số (`.mock.calls`). `mockReturnValue`/`mockResolvedValue`: điều khiển giá trị trả (sync/async). `jest.clearAllMocks()` (trong `afterEach`): reset để các test độc lập.
</details>

<details>
<summary><b>13.5 Test một hàm async ném exception thế nào?</b></summary>

`await expect(service.findOne('x')).rejects.toThrow(NotFoundException)`. Với giá trị trả: `await expect(fn()).resolves.toEqual(...)`. Theo mẫu **AAA** (Arrange–Act–Assert).
</details>

<details>
<summary><b>13.6 E2E test trong NestJS setup thế nào?</b></summary>

`Test.createTestingModule({ imports: [AppModule] }).compile()` → `createNestApplication()` → `app.init()` → dùng **supertest** gọi `request(app.getHttpServer()).post(...).send(...).expect(201)`. `app.init()` (không `listen()`) dựng app trong bộ nhớ, không chiếm cổng. Nhớ `afterAll(() => app.close())`.
</details>

<details>
<summary><b>13.7 Vì sao global pipe/filter không tự hoạt động trong E2E test?</b></summary>

Vì **`main.ts` không chạy** trong môi trường test — `createNestApplication()` chỉ dựng app từ AppModule. Mọi thứ đăng ký global ở `main.ts` (`useGlobalPipes`, `useGlobalFilters`, `useGlobalInterceptors`) phải **tự áp lại** trong setup test để giống production. Đây là bẫy hay gặp.
</details>

<details>
<summary><b>13.8 Chiến lược database cho E2E test?</b></summary>

Dùng **DB test riêng** (vd `blog_test`, hoặc SQLite in-memory) tách khỏi dev/prod. **Dọn dữ liệu** giữa các lần chạy: truncate bảng, hoặc bọc mỗi test trong **transaction rollback**, hoặc tạo dữ liệu với khoá duy nhất (vd email theo timestamp). Mục tiêu: test **ổn định**, không phụ thuộc dữ liệu cũ.
</details>

---

## 14. Prisma (ORM)

<details>
<summary><b>14.1 So sánh Prisma vs TypeORM: ưu/nhược điểm, khi nào chọn Prisma?</b></summary>

Prisma dùng schema-first declarative (`schema.prisma` là single source of truth) và sinh ra Prisma Client type-safe, nên type safety và developer experience tốt (autocomplete, migration rõ ràng). Điểm yếu: kém linh hoạt hơn với truy vấn rất phức tạp (trước phải dùng raw query; nay có thêm `relationJoins` ở dạng Preview), và Client là generated code nên có bước `prisma generate` bắt buộc trong build.

Lưu ý về thuật ngữ: Prisma KHÔNG theo Active Record/Data Mapper — nó là query builder/client sinh sẵn. TypeORM mới là ORM theo phong cách Active Record và Data Mapper với decorator trên entity, mạnh về QueryBuilder và quan hệ phức tạp, nhưng type safety yếu hơn và migration dễ drift (nhất là khi dùng `synchronize: true`).

Chọn Prisma khi: ưu tiên type safety ở tầng truy vấn, team muốn schema tập trung một chỗ và migration predictable; chọn TypeORM khi cần mô hình OOP entity/QueryBuilder rất động.

```prisma
// schema.prisma: single source of truth, sinh Client type-safe
model Issue {
  id    String @id @default(cuid())
  key   String @unique
  title String
}
```

Ví dụ thực tế: Trong Jira Clone tôi chọn Prisma vì nhiều tầng RBAC và quan hệ workspace/project/issue cần type safety chặt chẽ; project blog-api học tập dùng TypeORM thì dễ drift schema hơn (do `synchronize`).
</details>

<details>
<summary><b>14.2 Phân biệt `prisma migrate dev` và `prisma migrate deploy`?</b></summary>

`migrate dev` dùng trong development: so sánh schema với DB, tạo file migration mới, áp dụng và tự động chạy `prisma generate`. Nếu phát hiện drift hoặc migration không áp dụng sạch, nó sẽ HỎI xác nhận reset DB (mất data) — vì thế tuyệt đối không chạy trên production.

`migrate deploy` dùng cho CI/CD và production: chỉ áp dụng các migration đã có sẵn trong `prisma/migrations/` theo đúng thứ tự, KHÔNG sinh migration mới, KHÔNG reset, idempotent và an toàn. Lưu ý: `migrate deploy` KHÔNG tự chạy `generate`, nên pipeline thường chạy `prisma generate` riêng (hoặc trong postinstall).

Quy trình chuẩn: dev tạo migration bằng `migrate dev`, commit thư mục `migrations/` vào git, pipeline chạy `migrate deploy` khi release.

```bash
# Local (dev): tao + ap dung migration + generate
npx prisma migrate dev --name add_issue_key

# CI/CD (prod): chi ap dung migration da commit
npx prisma migrate deploy
```

Ví dụ thực tế: Pipeline deploy Jira Clone chạy `migrate deploy` trước khi start app để schema PostgreSQL luôn đồng bộ, không bao giờ chạy `migrate dev` trên server.
</details>

<details>
<summary><b>14.3 `select` và `include` khác nhau thế nào và dùng để tránh over-fetch ra sao?</b></summary>

`include` mở rộng relation NHƯNG vẫn lấy TẤT CẢ scalar field của model gốc, còn `select` cho phép chọn chính xác field cần lấy (và có thể lồng `select` trong relation), nên `select` là công cụ chính để tránh over-fetch. Không được dùng chung `select` và `include` ở cùng một cấp (Prisma sẽ báo lỗi); chỉ có thể lồng `select`/`include` bên trong một relation.

Chỉ lấy cột cần thiết giúp giảm payload, giảm tải DB và tránh lộ PII (ví dụ không bao giờ `select` `passwordHash`). Với API list nên luôn `select` tường minh thay vì trả về nguyên model.

```ts
// Chi lay field can, ke ca trong relation -> khong over-fetch
const issue = await prisma.issue.findUnique({
  where: { key: 'PROJ-12' },
  select: {
    id: true,
    title: true,
    assignee: { select: { id: true, name: true } }, // khong lay passwordHash/email
  },
});
```

Ví dụ thực tế: Endpoint list issue của Jira Clone luôn `select` username/avatar của assignee thay vì `include: { assignee: true }` để tránh leak email/PII và giảm over-fetch.
</details>

<details>
<summary><b>14.4 `$transaction` dạng interactive và dạng array khác nhau thế nào, khi nào dùng cái nào?</b></summary>

Dạng array (`prisma.$transaction([...])`) nhận một mảng các Prisma promise và chạy tất cả trong CÙNG một transaction (BEGIN/COMMIT bao quanh, ABORT nếu một lệnh lỗi). Các lệnh được gửi theo thứ tự trong mảng, nhưng dạng này KHÔNG cho bạn dùng kết quả của bước trước làm input cho bước sau — phù hợp khi các thao tác độc lập.

Dạng interactive (`prisma.$transaction(async (tx) => {...})`) cho callback với client `tx`, cho phép logic điều kiện, đọc rồi ghi dựa trên kết quả trung gian. Nhược điểm: connection bị giữ trong suốt callback, nên phải giữ transaction NGẮN và tránh gọi I/O ngoài (HTTP, message queue) bên trong để không giữ lock/connection lâu. Có thể cấu hình `timeout`, `maxWait`, và `isolationLevel`.

```ts
// Interactive: sinh issue key PROJECT-N mot cach atomic
await prisma.$transaction(async (tx) => {
  const p = await tx.project.update({
    where: { id },
    data: { issueCounter: { increment: 1 } },
    select: { key: true, issueCounter: true },
  });
  return tx.issue.create({
    data: { key: `${p.key}-${p.issueCounter}`, title },
  });
});
```

Ví dụ thực tế: Trong Jira Clone, sinh issue key (PROJECT-N) phải atomic: tăng counter của project rồi tạo issue trong cùng interactive transaction để tránh trùng key khi tạo đồng thời.
</details>

<details>
<summary><b>14.5 Vấn đề N+1 trong Prisma xảy ra khi nào và cách tránh?</b></summary>

N+1 xảy ra khi bạn query một danh sách rồi loop qua từng phần tử để query relation riêng lẻ -> 1 query list + N query con. Trong Prisma, cách tránh chuẩn là dùng `include`/`select` để Prisma fetch relation trong cùng lần gọi: theo mặc định Prisma chạy thêm một query `findMany ... WHERE id IN (...)` cho relation (chứ KHÔNG phải N query riêng).

Ngoài ra Prisma có DataLoader nội bộ: tự động gom các `findUnique` có cùng where/selection trong cùng một tick sự kiện thành một `findMany` — hữu ích trong GraphQL resolver. Với quan hệ cần join thực sự ở tầng DB (một SQL duy nhất dùng LATERAL JOIN), bật Preview feature `relationJoins` rồi dùng `relationLoadStrategy: 'join'`.

```ts
// XAU: N+1 -> moi issue mot query assignee
const issues = await prisma.issue.findMany();
for (const i of issues) {
  const a = await prisma.user.findUnique({ where: { id: i.assigneeId } });
}
// TOT: mot lan, Prisma gom relation bang IN-query
const issues2 = await prisma.issue.findMany({
  include: { assignee: { select: { id: true, name: true } } },
});
```

Ví dụ thực tế: Màn hình board của Jira Clone load nhiều issue kèm assignee/labels — dùng `include`/`select` thay vì loop để tránh N+1 khi có hàng trăm issue.
</details>

<details>
<summary><b>14.6 Connection pool trong Prisma hoạt động sao và PG driver adapter để làm gì?</b></summary>

Prisma quản lý connection pool nội bộ. Với query engine (Prisma ORM v6 trở về trước), kích thước mặc định là `so_physical_cpus * 2 + 1` và chỉnh qua tham số `connection_limit` trên `DATABASE_URL`. (Từ v7 khi dùng driver adapter, default đổi sang cấu hình của driver, ví dụ `pg` mặc định `max = 10`.)

Trong serverless/nhiều instance phải rất cẩn thận: mỗi instance mở pool riêng -> dễ cạn kiệt `max_connections` của PostgreSQL. Lúc đó nên dùng connection pooler ngoài (PgBouncer, Prisma Accelerate) và hạ `connection_limit` (thường `=1` khi qua pooler chế độ transaction). Driver adapter (`@prisma/adapter-pg` với `pg`/node-postgres) cho phép Prisma dùng driver JS native thay cho query engine binary — hữu ích cho edge runtime và dễ dùng pool/pooler có sẵn.

```ts
// Driver adapter (v6.6.0+): truyen thang option cua driver
import { PrismaPg } from '@prisma/adapter-pg';
import { PrismaClient } from '@prisma/client';

const adapter = new PrismaPg({ connectionString: process.env.DATABASE_URL });
const prisma = new PrismaClient({ adapter });
```

Ví dụ thực tế: Backend Jira Clone chạy long-running NestJS nên pool nội bộ của Prisma là đủ; tôi chỉ chỉnh `connection_limit` khi scale nhiều pod để không vượt `max_connections` của PostgreSQL.
</details>

<details>
<summary><b>14.7 Khi nào dùng `$queryRaw` và làm sao tránh SQL injection?</b></summary>

Dùng raw query khi cần tính năng SQL mà Prisma Client chưa hỗ trợ: window function, CTE phức tạp, full-text search đặc thù, hoặc tối ưu hiệu năng đặc biệt.

Để an toàn, dùng tagged template `$queryRaw\`...\`` — Prisma tự động tham số hóa mọi giá trị nội vào template -> chống SQL injection. Để ghép các đoạn SQL động một cách AN TOÀN, dùng helper `Prisma.sql` (và `Prisma.join`, `Prisma.raw` cho identifier tin cậy) rồi vẫn truyền vào `$queryRaw`. Lưu ý chính xác: `$queryRawUnsafe(sql, ...params)` nhận một CHUỖI thường + tham số vị trí, KHÔNG đi kèm `Prisma.sql`, và chỉ nên dùng khi bắt buộc; tuyệt đối không nối chuỗi giá trị người dùng trực tiếp vào đó.

Nhược điểm: kết quả raw KHÔNG type-safe, nên khai báo generic kiểu trả về; và mất tính portability giữa các DB.

```ts
// An toan: gia tri duoc tham so hoa tu dong
const email = req.body.email;
const users = await prisma.$queryRaw<
  { id: string; email: string }[]
>`SELECT id, email FROM "User" WHERE email = ${email}`;
// NGUY HIEM: noi chuoi -> SQL injection
// prisma.$queryRawUnsafe(`... WHERE email = '${email}'`)
```

Ví dụ thực tế: Một vài báo cáo thống kê issue theo trạng thái trong Jira Clone tôi viết bằng `$queryRaw` (template tham số hóa), vẫn an toàn injection mà tối ưu được aggregation.
</details>

<details>
<summary><b>14.8 Enum trong Prisma được map xuống PostgreSQL và sinh type ra sao?</b></summary>

Khai báo `enum` trong `schema.prisma` sẽ được Prisma tạo thành native enum type trong PostgreSQL (`CREATE TYPE`) và sinh ra TypeScript enum tương ứng trong Prisma Client, nên giá trị được kiểm tra cả ở tầng DB lẫn compile time.

Lưu ý về migration: thêm giá trị enum là thay đổi an toàn, NHƯNG trên PostgreSQL `ALTER TYPE ... ADD VALUE` không chạy được bên trong transaction block, nên Prisma sẽ tách nó ra một migration riêng. Đổi/xóa giá trị enum thường cần migration thủ công vì ảnh hưởng data hiện có (phải backfill/cast). Enum giúp ràng buộc trường trạng thái/role gọn và type-safe hơn string tùy ý.

```prisma
enum IssueStatus {
  TODO
  IN_PROGRESS
  DONE
}

model Issue {
  id     String      @id @default(cuid())
  status IssueStatus @default(TODO)
}
```

Ví dụ thực tế: Các role RBAC nhiều tầng của Jira Clone (workspace OWNER/ADMIN/MEMBER/VIEWER, project LEAD/ADMIN/DEVELOPER/VIEWER) tôi mô hình bằng Prisma enum để vừa ràng buộc ở DB vừa có type cho guard kiểm quyền.
</details>

<details>
<summary><b>14.9 Optimistic vs pessimistic locking trong Prisma: tránh lost update ra sao?</b></summary>

Khi nhiều request cùng sửa một bản ghi (ví dụ kéo-thả issue trên board), có thể xảy ra lost update. Prisma không có decorator `@Version` tự động như TypeORM, nên optimistic locking phải tự làm: thêm cột `version` (hoặc dùng `updatedAt`) và đưa nó vào điều kiện `where` của `update`; nếu không khớp nghĩa là ai đó đã sửa trước -> retry.

Pessimistic locking thì dùng `SELECT ... FOR UPDATE` qua `$queryRaw` bên trong interactive transaction để khóa hàng cho đến khi commit.

```ts
// Optimistic: chi update neu version chua doi
const res = await prisma.issue.updateMany({
  where: { id, version: expectedVersion },
  data: { status: 'DONE', version: { increment: 1 } },
});
if (res.count === 0) throw new ConflictException('Issue da bi sua, hay tai lai');
```

Ví dụ thực tế: Trong Jira Clone, khi hai user cùng kéo một issue sang cột khác, tôi dùng optimistic locking bằng cột `version` để phát hiện xung đột và báo client reload thay vì để mất thay đổi.
</details>

<details>
<summary><b>14.10 Soft delete trong Prisma làm thế nào và có gì cần lưu ý?</b></summary>

Prisma không có soft delete tích hợp, nên pattern phổ biến là thêm cột `deletedAt DateTime?` và coi NULL = còn sống. Khi 'xóa' thì `update` set `deletedAt = now()` thay vì `delete`. Điểm cần lưu ý: mọi truy vấn đọc phải tự thêm `where: { deletedAt: null }`, nếu quên sẽ lộ bản ghi đã xóa. Để đỡ lặp lại, dùng Prisma Client Extensions (`$extends`) để chèn điều kiện tự động, hoặc middleware (`$use`, đã deprecated ở bản mới).

Ngoài ra unique constraint vẫn tính cả dòng đã soft-delete -> có thể cần partial unique index hoặc đưa `deletedAt` vào khóa duy nhất.

```ts
const prisma = basePrisma.$extends({
  query: {
    issue: {
      async findMany({ args, query }) {
        args.where = { ...args.where, deletedAt: null };
        return query(args);
      },
    },
  },
});
```

Ví dụ thực tế: Trong Jira Clone, issue và comment dùng soft delete (`deletedAt`) để còn audit/khôi phục; tôi bọc bằng Client Extension để các endpoint list mặc định chỉ trả về bản ghi chưa xóa.
</details>

---
## 15. Advanced Authentication

<details>
<summary><b>15.1 Phân biệt access token và refresh token? Tại sao cần cả hai thay vì một token sống lâu?</b></summary>

**Access token** là token đoản thọ (short-lived, ví dụ 15 phút), được gửi kèm mỗi request để xác thực; vì nó stateless nên server chỉ cần verify chữ ký, không cần query DB mỗi lần. **Refresh token** sống lâu (ví dụ 7 ngày), chỉ dùng để cấp lại access token mới khi hết hạn. Tách làm hai giảm thiệt hại khi token bị lộ: nếu access token bị đánh cắp qua XSS/log thì kẻ tấn công chỉ có cửa sổ rất ngắn, còn refresh token thì được bảo vệ kỹ hơn (httpOnly cookie, scope hẹp — chỉ gửi tới endpoint refresh, có thể thu hồi từ DB). Một token sống lâu duy nhất sẽ vừa khó thu hồi vừa mở rộng bề mặt tấn công.

Lưu ý quan trọng: refresh token **không nên** chứa cùng payload (role, sub...) như access token, và tốt nhất nên là **opaque random token** (ví dụ 32 bytes ngẫu nhiên) thay vì JWT — vì refresh token là thứ được lưu và đối chiếu trong DB, dùng JWT cho nó chỉ thêm rủi ro lộ claim mà không lợi ích gì.

```ts
async login(user: User, res: Response) {
  const payload = { sub: user.id, role: user.globalRole };
  const accessToken = this.jwt.sign(payload, { expiresIn: '15m' });
  // refresh token: opaque random, KHONG phai JWT
  const refreshToken = crypto.randomBytes(32).toString('hex');
  // luu HASH cua refresh token vao DB de co the thu hoi
  await this.prisma.refreshToken.create({
    data: { userId: user.id, tokenHash: await argon2.hash(refreshToken) },
  });
  res.cookie('refresh_token', refreshToken, {
    httpOnly: true, secure: true, sameSite: 'strict', path: '/auth/refresh',
  });
  return { accessToken };
}
```

Ví dụ thực tế: Trong Jira Clone, access token đoản (15m) đi trong httpOnly cookie và được validate bởi global JwtAuthGuard, còn refresh token (7d) chỉ dùng cho endpoint `/auth/refresh` (giới hạn bằng `path` của cookie).
</details>

<details>
<summary><b>15.2 Refresh token ROTATION là gì? Nó chống replay attack như thế nào?</b></summary>

Rotation nghĩa là mỗi lần dùng refresh token để lấy access token mới, server cấp luôn một refresh token MỚI và vô hiệu hóa (invalidate) token cũ. Nhờ vậy mỗi refresh token chỉ dùng được đúng một lần. Điều này chống **replay**: nếu kẻ tấn công đánh cắp một refresh token và dùng lại sau khi user thật đã refresh, thì token đó đã bị xoay (token cũ đã invalid). Khi server nhận một refresh token **đã bị đánh dấu used** mà lại được gửi lại, đó là dấu hiệu **token reuse** (có thể là attacker hoặc user dùng token cũ) -> server lập tức thu hồi toàn bộ **token family** của user, buộc đăng nhập lại. Nguyên tắc vàng là lưu token theo **family/chain** để phát hiện reuse.

Lưu ý đồng thời: hai thao tác đánh dấu `usedAt` token cũ và cấp token mới nên nằm trong **một transaction** để tránh race condition khi có nhiều request refresh song song.

```ts
async refresh(oldToken: string, res: Response) {
  return this.prisma.$transaction(async (tx) => {
    const record = await this.findValidToken(tx, oldToken);
    if (!record) throw new UnauthorizedException();
    if (record.usedAt || record.revokedAt) {
      // token da dung/da thu hoi -> reuse detected -> thu hoi ca family
      await tx.refreshToken.updateMany({
        where: { familyId: record.familyId, revokedAt: null },
        data: { revokedAt: new Date() },
      });
      throw new UnauthorizedException('Token reuse detected');
    }
    await tx.refreshToken.update({
      where: { id: record.id }, data: { usedAt: new Date() },
    });
    // cap token moi cung familyId
    return this.issueTokens(tx, record.userId, record.familyId, res);
  });
}
```

Ví dụ thực tế: Đây chính là cơ chế 'refresh token rotation' trong Jira Clone — mỗi refresh sinh token mới, reuse bị phát hiện sẽ revoke cả chuỗi (family) token.
</details>

<details>
<summary><b>15.3 Lưu token ở httpOnly cookie vs localStorage khác nhau thế nào? Trade-off XSS và CSRF?</b></summary>

**localStorage** truy cập được bằng JavaScript nên rất dễ bị đánh cắp nếu trang có lỗ hổng **XSS** — script đọc được token và gửi đi. **httpOnly cookie** thì JS không đọc được, nên chống được việc **đánh cắp token** qua XSS, nhưng cookie tự động gửi kèm mỗi request nên mở cửa cho **CSRF**.

Cần nói rõ: httpOnly cookie **không** miễn nhiễm hoàn toàn với XSS — nếu trang dính XSS, attacker vẫn có thể gọi API thay mặt user (vì cookie tự gửi kèm), chỉ là không **trích xuất được** token ra ngoài. Vì vậy phòng XSS từ gốc (escape output, CSP) vẫn là bắt buộc.

Cách làm đúng: dùng httpOnly + `secure` + `sameSite` cookie để token không bị trích xuất qua XSS, rồi chống CSRF bằng `sameSite=strict/lax`, kiểm tra Origin/Referer header, hoặc CSRF token (double-submit). Tóm lại httpOnly cookie an toàn hơn localStorage cho việc lưu token, nhưng phải bù lại bằng phòng CSRF.

```ts
res.cookie('access_token', token, {
  httpOnly: true,   // JS khong doc/trich xuat duoc token
  secure: true,     // chi gui qua HTTPS
  sameSite: 'strict', // chong CSRF
  maxAge: 15 * 60 * 1000,
});
```

Ví dụ thực tế: Jira Clone đặt JWT trong httpOnly cookie (không dùng localStorage) kết hợp helmet, CSP và sameSite để cân bằng giữa chống trích xuất token (XSS) và chống CSRF.
</details>

<details>
<summary><b>15.4 JWT (stateless) vs session (stateful) khác nhau ra sao? Khi nào chọn cái nào?</b></summary>

**JWT stateless**: thông tin user nằm trong token, server chỉ cần verify chữ ký bằng secret (HMAC) hoặc public key (RSA/EC), không query store — dễ scale ngang, hợp kiến trúc microservice. Nhược điểm là **khó thu hồi trước hạn**: token vẫn hợp lệ đến khi hết hạn. **Session stateful**: server lưu session trong store (Redis/DB), client chỉ giữ session id; dễ thu hồi (xóa session là xong) và revoke tức thì, nhưng mỗi request cần lookup store và khó scale hơn (dù với Redis thì chi phí này rất nhỏ). Thực tế thường **lai**: access token JWT đoản hạn + refresh token có state trong DB để vẫn thu hồi được.

Lưu ý: `jwt.verify` là thao tác CPU (kiểm tra chữ ký), không phải không tốn gì — với RSA/EC còn đắt hơn HMAC. Điểm khác biệt chính là nó **không I/O** (không chạm mạng/DB), chứ không phải 'miễn phí'.

```ts
// JWT: verify khong can DB (chi ton CPU kiem tra chu ky)
const payload = jwt.verify(token, PUBLIC_KEY); // khong I/O

// Session: phai lookup store moi request
const session = await redis.get(`sess:${sessionId}`);
if (!session) throw new UnauthorizedException();
```

Ví dụ thực tế: Jira Clone chọn mô hình lai — JWT access token stateless để validate nhanh qua global guard, refresh token lưu DB (stateful) để hỗ trợ thu hồi và rotation.
</details>

<details>
<summary><b>15.5 Mô tả flow OTP / email verification an toàn? Cần lưu OTP thế nào?</b></summary>

Flow: user đăng ký -> server sinh OTP ngẫu nhiên an toàn (ví dụ 6 chữ số qua `crypto.randomInt`) -> lưu **HASH của OTP** + thời điểm hết hạn (ví dụ 5-10 phút) vào DB, KHÔNG lưu plaintext -> gửi OTP qua email. Khi user nhập, server hash lại và so sánh, kiểm tra chưa hết hạn và chưa dùng. Phải có **rate limit / giới hạn số lần thử** để chống brute force (vì không gian 6 chữ số chỉ 1 triệu khả năng, rất dễ dò nếu không giới hạn), OTP chỉ dùng một lần (single-use). Sau khi verify thành công thì đánh dấu `emailVerified` và xóa/invalidate OTP.

Lưu ý: dùng `argon2`/`bcrypt` cho OTP cũng được và an toàn, nhưng vì OTP là chuỗi số ngẫu nhiên thọ sống rất ngắn, nhiều hệ thống dùng HMAC-SHA256 (nhanh hơn, đủ an toàn cho giá trị thọ sống ngắn) — điểm mấu chốt là **không lưu plaintext** và **có rate limit**. Argon2 đã có so sánh chống timing attack bên trong nên không cần tự so sánh thủ công.

```ts
async sendOtp(email: string) {
  const otp = crypto.randomInt(100000, 1000000).toString(); // 100000..999999
  await this.prisma.otp.create({
    data: {
      email,
      codeHash: await argon2.hash(otp),
      expiresAt: new Date(Date.now() + 5 * 60_000),
    },
  });
  await this.email.send(email, `Ma OTP cua ban: ${otp}`);
}

async verifyOtp(email: string, code: string) {
  const rec = await this.prisma.otp.findFirst({
    where: { email, usedAt: null },
    orderBy: { createdAt: 'desc' }, // lay OTP moi nhat
  });
  if (!rec || rec.expiresAt < new Date()) throw new BadRequestException('OTP het han');
  if (!(await argon2.verify(rec.codeHash, code))) throw new BadRequestException('OTP sai');
  await this.prisma.otp.update({ where: { id: rec.id }, data: { usedAt: new Date() } });
}
```

Lưu ý: `crypto.randomInt(min, max)` có `max` là **exclusive**, nên để bao gồm 999999 phải dùng `randomInt(100000, 1000000)`.

Ví dụ thực tế: Jira Clone dùng Resend gửi OTP qua email cho email verification, kết hợp @nestjs/throttler để rate limit endpoint gửi/verify OTP.
</details>

<details>
<summary><b>15.6 Với JWT stateless, làm sao implement logout và thu hồi token thực sự?</b></summary>

Với JWT thuần stateless thì 'logout' ở client chỉ là xóa cookie, nhưng token vẫn hợp lệ đến khi hết hạn — chưa phải thu hồi thật. Để thu hồi thực sự có hai hướng: (1) **thu hồi refresh token** trong DB (xóa hoặc đánh dấu revoked) nên lần refresh tiếp theo sẽ fail — nhờ access token đoản hạn nên cửa sổ rủi ro nhỏ; (2) dùng **blocklist** (lưu `jti` của token bị cấm vào Redis với TTL = thời gian còn lại của token) để chặn ngay cả access token còn hạn. Lưu ý hướng (2) khiến mỗi request phải lookup Redis -> đánh mất tính stateless của JWT, nên chỉ dùng khi thật sự cần revoke tức thì. Logout-all-devices thì revoke toàn bộ refresh token / token family của user.

```ts
async logout(userId: string, jti: string, exp: number, res: Response) {
  // 1. thu hoi refresh token
  await this.prisma.refreshToken.updateMany({
    where: { userId, revokedAt: null }, data: { revokedAt: new Date() },
  });
  // 2. (tuy chon) blocklist access token con han
  const ttl = exp - Math.floor(Date.now() / 1000);
  if (ttl > 0) await this.redis.set(`bl:${jti}`, '1', 'EX', ttl);
  res.clearCookie('access_token');
  res.clearCookie('refresh_token');
}
```

Lưu ý: để blocklist hoạt động, khi sign access token phải gắn `jti` (ví dụ `jwtid: crypto.randomUUID()`) và JwtAuthGuard phải kiểm tra Redis xem `jti` có bị block không.

Ví dụ thực tế: Trong Jira Clone, logout thu hồi refresh token trong DB; vì access token chỉ sống 15m nên không bắt buộc blocklist, dù vẫn có thể thêm cho yêu cầu security cao.
</details>

<details>
<summary><b>15.7 Tại sao access token nên hết hạn ngắn và refresh token hết hạn dài? Cấu hình expiry hợp lý?</b></summary>

Access token hết hạn **ngắn** (5-15 phút) để giới hạn thiệt hại khi bị lộ: vì nó đi kèm mỗi request và khó thu hồi (stateless), cửa sổ sống càng ngắn thì rủi ro càng nhỏ. Refresh token hết hạn **dài** (vài ngày đến vài tuần) để user không phải đăng nhập lại liên tục, và vì nó ít lộ diện hơn (chỉ gửi tới endpoint refresh, scope hẹp, lưu httpOnly), dễ thu hồi từ DB, lại được rotation. Đây là sự đánh đổi giữa **bảo mật** (token càng ngắn càng an toàn) và **trải nghiệm** (token quá ngắn thì phải refresh liên tục). Có thể thêm **sliding + absolute expiration**: gia hạn khi có hoạt động nhưng giới hạn tổng thời lượng tuyệt đối (ví dụ tối đa 30 ngày dù vẫn active).

```ts
const ACCESS_TTL = '15m';   // ngan: gioi han rui ro khi bi lo
const REFRESH_TTL = '7d';   // dai: UX tot, lo it va thu hoi duoc

const accessToken = jwt.sign(payload, accessSecret, { expiresIn: ACCESS_TTL });
// refresh token thuong la opaque random + cot expiresAt trong DB,
// neu van dung JWT thi tach secret rieng:
const refreshToken = jwt.sign(payload, refreshSecret, { expiresIn: REFRESH_TTL });
```

Lưu ý bảo mật: access token và refresh token nên dùng **secret/key riêng** (hoặc key id khác nhau), tránh dùng chung một secret cho cả hai.

Ví dụ thực tế: Jira Clone cấu hình access 15m + refresh 7d — đủ ngắn để an toàn, đủ dài để người dùng không phải đăng nhập lại mỗi giờ, kết hợp rotation để bù đắp thời gian sống dài của refresh token.
</details>

---

## 16. Web Security

<details>
<summary><b>16.1 Helmet làm gì và những security header nào quan trọng nhất?</b></summary>

Helmet là middleware set một loạt HTTP security header để giảm các tấn công phổ biến. Các header quan trọng:
- `Content-Security-Policy` (chống XSS/injection bằng cách whitelist nguồn script/style/connect; directive `frame-ancestors` cũng thuộc CSP và chống clickjacking, dần thay thế `X-Frame-Options`).
- `Strict-Transport-Security` (HSTS — ép trình duyệt dùng HTTPS ở các lần sau, chống SSL stripping).
- `X-Content-Type-Options: nosniff` (chặn MIME sniffing).
- `X-Frame-Options` (chống clickjacking với trình duyệt cũ).

Lưu ý chính xác: Helmet **gỡ bỏ** (remove) header `X-Powered-By` chứ không phải set giá trị che giấu — mục đích giảm fingerprinting. Helmet chỉ là defense-in-depth, không thay thế validation hay output encoding.

```ts
import helmet from 'helmet';
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.use(helmet()); // bật default headers; cần tinh chỉnh CSP nếu serve assets/Swagger
}
```

Lưu ý: HSTS chỉ có tác dụng trên kết nối HTTPS và do trình duyệt thực thi; nó không trực tiếp "bảo vệ cookie" mà bảo đảm toàn bộ request (bao gồm cookie) đi qua HTTPS. Với API trả JSON, CSP mặc định ít gây vướng nhưng vẫn nên giữ `nosniff` và HSTS.
</details>

<details>
<summary><b>16.2 Rate limiting/throttling với @nestjs/throttler hoạt động ra sao, và vì sao auth endpoint phải strict hơn?</b></summary>

Throttler giới hạn số request trong một cửa sổ thời gian theo key (mặc định là IP), chống brute-force, credential stuffing và một phần DoS ở tầng app. Ta cấu hình global qua `ThrottlerModule.forRoot([...])` + đăng ký `ThrottlerGuard` làm `APP_GUARD`, rồi override per-route bằng `@Throttle`. Auth endpoint (login, OTP, forgot-password) phải strict hơn vì đó là nơi attacker đoán mật khẩu/OTP; giới hạn ví dụ 5 request/phút làm brute-force tốn kém và chậm đi nhiều (chứ không tuyệt đối "bất khả thi"), trong khi API đọc dữ liệu bình thường có thể nới lỏng hơn.

Lưu ý quan trọng: trong @nestjs/throttler v5+, `ttl` tính bằng **mili-giây** (nên `60_000` = 60s), có thể dùng helper `seconds(60)`/`minutes(1)` cho rõ.

```ts
@Throttle({ default: { limit: 5, ttl: 60_000 } }) // login: 5 lần / 60 giây
@Post('login')
login(@Body() dto: LoginDto) { /* ... */ }
```

Ví dụ thực tế: throttler global lỏng cho route đọc, siết riêng `/auth/login` và `/auth/verify-otp`. Với hệ nhiều instance, in-memory mỗi pod sẽ đếm rời rạc nên rate limit không đúng tổng thể — phải dùng storage chia sẻ (vd `ThrottlerStorageRedisService`). Bổ sung: rate limit theo IP dễ bị né bằng IP xoay vòng, nên với login nên kết hợp khóa theo account/username, không chỉ IP.
</details>

<details>
<summary><b>16.3 CORS là gì, cấu hình thế nào cho an toàn khi dùng cookie auth?</b></summary>

CORS là cơ chế cho phép một origin gọi tài nguyên ở origin khác: server **khai báo** origin nào được phép qua `Access-Control-Allow-Origin`, còn việc **thực thi/chặn là do trình duyệt** làm (client tin tưởng các tool như curl/Postman không bị ràng buộc CORS). Khi auth bằng httpOnly cookie, phải bật `credentials: true` và **không được** dùng `origin: '*'` — spec cấm phối hợp wildcard với credentials, đồng thời nên echo lại đúng origin trong whitelist. CORS không phải lớp authorization: nó kiểm soát trình duyệt nào đọc được response cross-origin, KHÔNG thay cho auth/authorization phía server.

```ts
app.enableCors({
  origin: ['https://app.jira-clone.com'], // whitelist cụ thể, không dùng '*'
  credentials: true, // cho phép gửi/nhận cookie cross-origin
});
```

Ví dụ thực tế: vì JWT để trong httpOnly cookie, set `credentials: true` và liệt kê đúng origin của SPA; nếu để `*` thì trình duyệt sẽ chặn gửi cookie và đăng nhập hỏng. Tránh phản chiếu (reflect) origin từ request một cách vô điều kiện vì đó tương đương mở cho mọi site.
</details>

<details>
<summary><b>16.4 CSRF là gì, khi nào dễ bị, và vì sao dùng cookie lại làm lộ rủi ro này?</b></summary>

CSRF lợi dụng việc trình duyệt **tự động đính kèm cookie** vào mọi request tới domain đó: site độc hại có thể kích hoạt request đổi trạng thái (POST/PUT/DELETE) dưới danh tính nạn nhân. Rủi ro xuất hiện khi auth dựa trên cookie ambient. Ngược lại, token gửi qua header `Authorization` gần như miễn nhiễm CSRF **vì JS phải chủ động gắn token vào từng request** — trình duyệt không tự thêm header này cho request cross-site.

Phòng chống: `SameSite` cho cookie, CSRF token (synchronizer hoặc double-submit), và kiểm tra Origin/Referer. Lưu ý chính xác về `SameSite`:
- `Strict`: chặn cookie ở mọi request cross-site (kể cả khi click link sang site).
- `Lax` (mặc định của trình duyệt hiện nay): cookie KHÔNG gửi với POST/iframe/img cross-site, nhưng VẪN gửi với top-level GET navigation — nên Lax giảm mạnh CSRF cho mutation nhưng không phải tuyệt đối, đừng dùng GET để đổi state.
- `None`: bắt buộc kèm `Secure` và cần thêm CSRF token.

```ts
res.cookie('access_token', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'lax', // chặn POST cross-site tự động — phòng CSRF cơ bản
});
```

Ví dụ thực tế: JWT trong httpOnly cookie có bề mặt CSRF — giảm thiểu bằng `SameSite=Lax` + CORS whitelist; nếu cần cross-site thật (`SameSite=None`) thì phải thêm CSRF token cho mọi route mutation.
</details>

<details>
<summary><b>16.5 Phân biệt input validation và sanitization; làm sao chống XSS và mass assignment?</b></summary>

Validation là từ chối dữ liệu không đúng định dạng/ràng buộc (kiểu, độ dài, enum, format). Sanitization là làm sạch/loại bỏ/escape dữ liệu trước khi lưu hoặc render. Hai việc này bổ sung cho nhau, không thay thế nhau.

Trong NestJS dùng `class-validator` + global `ValidationPipe` với:
- `whitelist: true` (tự loại bỏ field không khai báo trong DTO),
- `forbidNonWhitelisted: true` (ném 400 nếu có field lạ) — đây chính là tuyến chống **mass assignment** (client gắn thêm `role: 'ADMIN'`, `authorId`...).

Về XSS: với API JSON thuần, server không render HTML nên XSS chủ yếu là **stored XSS** — dữ liệu bẩn lưu vào DB rồi client render ra DOM. Phòng chống đúng tầng là **output encoding ở nơi render** (React tự escape; nếu dùng `dangerouslySetInnerHTML` thì phải sanitize bằng DOMPurify). Server không nên "tin" input và có thể sanitize HTML đầu vào nếu cho phép rich text.

```ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,            // bỏ field thừa
  forbidNonWhitelisted: true, // ném lỗi nếu có field lạ (chống mass assignment)
  transform: true,            // ép kiểu theo DTO
}));
```

Ví dụ thực tế: DTO tạo issue/comment đều validate qua class-validator; `whitelist`/`forbidNonWhitelisted` chặn client tự set `authorId` hay `status`; nội dung comment được escape/sanitize phía frontend khi render để tránh stored XSS. Lưu ý phòng thủ theo tầng: chống mass assignment ở server, chống XSS chủ yếu ở chỗ render.
</details>

<details>
<summary><b>16.6 Vì sao Prisma/parameterized query chống được SQL injection? Khi nào vẫn có rủi ro?</b></summary>

SQL injection xảy ra khi input người dùng bị nối chuỗi trực tiếp vào câu SQL và bị diễn dịch như mã lệnh. Prisma Client và parameterized query tách **câu lệnh SQL** khỏi **dữ liệu**: giá trị được gửi như tham số bound, driver/DB không bao giờ diễn dịch chúng như mã SQL, nên injection bị vô hiệu.

Rủi ro vẫn còn khi: dùng `$queryRawUnsafe`/`$executeRawUnsafe` với chuỗi nối tay, hoặc nhúng tên bảng/cột động (identifier không thể parameterize như giá trị). Khi cần raw: dùng tagged template `$queryRaw\`...\`` (tự parameterize), hoặc ghép an toàn bằng helper `Prisma.sql` và `Prisma.join`; với identifier động phải whitelist tên cột/bảng thủ công.

```ts
import { Prisma } from '@prisma/client';
// AN TOÀN: tagged template -> parameterized
const users = await prisma.$queryRaw`SELECT * FROM "User" WHERE email = ${email}`;
// AN TOÀN khi ghép động: dùng Prisma.sql
const rows = await prisma.$queryRaw(Prisma.sql`SELECT * FROM "User" WHERE id = ${id}`);
// NGUY HIỂM: nối chuỗi trực tiếp
// prisma.$queryRawUnsafe(`SELECT * FROM "User" WHERE email = '${email}'`)
```

Ví dụ thực tế: hầu hết dùng Prisma Client typed query nên mặc định an toàn; chỗ duy nhất cần cẩn thận là vài thống kê dùng `$queryRaw` — luôn dùng tagged template, không bao giờ dùng `...Unsafe` với input người dùng.
</details>

<details>
<summary><b>16.7 Quản lý secret/env an toàn và masking PII trong log như thế nào?</b></summary>

Secret (DB URL, JWT secret, API key Resend) không bao giờ commit vào repo: để trong `.env` đã `.gitignore`, validate khi khởi động bằng schema (vd `@nestjs/config` + Joi/zod) để fail-fast nếu thiếu, và ở production inject qua secret manager/biến môi trường của hạ tầng (Vault, AWS Secrets Manager, env của platform) chứ không phải file phẳng. Khi secret lỡ lộ thì phải **rotate** chứ không chỉ xóa khỏi history.

Với log: masking PII và credential (email, token, password, OTP, cookie) trước khi ghi để tránh rò rỉ và đáp ứng compliance. Nên sanitize tập trung trong logger/interceptor, và xử lý cả nested object (ví dụ dưới chỉ mask field cấp 1, thực tế cần đệ quy).

```ts
const SECRET = ['password', 'token', 'otp', 'authorization', 'cookie'];
function sanitize(obj: Record<string, any>): Record<string, any> {
  return Object.fromEntries(
    Object.entries(obj).map(([k, v]) => {
      if (SECRET.includes(k.toLowerCase())) return [k, '***'];
      if (v && typeof v === 'object') return [k, sanitize(v)]; // đệ quy nested
      return [k, v];
    }),
  );
}
```

Ví dụ thực tế: structured logging có bước PII sanitization (mask token/email) trước khi ghi, cron retention dọn log cũ, Sentry chỉ bật ở prod và nên cấu hình scrubbing để access token/OTP không xuất hiện trong log hay error report.
</details>

<details>
<summary><b>16.8 Kể vài lỗ hổng OWASP Top 10 phổ biến và cách bạn phòng chống trong NestJS?</b></summary>

Một số mục OWASP Top 10 (2021) hay gặp:
- **A01 Broken Access Control**: RBAC + guard, luôn kiểm quyền ở server, kiểm cả object-level (IDOR — user chỉ truy cập resource của mình), không tin client.
- **A03 Injection**: parameterized query/Prisma + validation.
- **A07 Identification & Authentication Failures**: hash mật khẩu bằng bcrypt/argon2 (không bao giờ lưu plaintext/MD5), rate limit login, refresh token rotation, khóa tài khoản sau nhiều lần sai.
- **A05 Security Misconfiguration**: helmet, tắt verbose error/stack trace ở prod, không bật Swagger/debug ở prod.
- **A02 Cryptographic Failures**: HTTPS/TLS, không log secret, dùng thuật toán mạnh.
- **A06 Vulnerable & Outdated Components**: theo dõi dependency.

Nguyên tắc xuyên suốt: least privilege và defense-in-depth.

```ts
@UseGuards(JwtAuthGuard, RolesGuard)
@Roles('PROJECT_LEAD', 'PROJECT_ADMIN')
@Delete('projects/:id')
remove(@Param('id') id: string) { /* chỉ role đủ quyền mới xóa */ }
```

Ví dụ thực tế: global guard qua `APP_GUARD` + `@Public()` để mặc định mọi route đều cần auth (fail-safe), RBAC nhiều tầng (global/workspace/project/author-only) chống Broken Access Control, refresh token rotation chống replay — phòng nhiều mục OWASP cùng lúc.
</details>

<details>
<summary><b>16.9 [BỔ SUNG] JWT nên lưu ở đâu, đặt thời hạn thế nào, và xử lý access/refresh token ra sao?</b></summary>

Câu này hay được hỏi sâu vì auth là trung tâm của security. Các điểm cốt lõi:

**Lưu token ở đâu (client):**
- `localStorage`: tiện nhưng JS đọc được → dễ bị đánh cắp nếu có XSS, không nên cho token nhạy cảm.
- httpOnly cookie: JS không đọc được → chống XSS-exfiltration tốt hơn, nhưng phát sinh bề mặt CSRF (xem câu CSRF) → cần `SameSite` + có thể CSRF token. Đây thường là lựa chọn an toàn hơn cho web app.

**Thời hạn:** access token sống ngắn (vd 5–15 phút) để giảm thiệt hại khi lộ; refresh token sống dài hơn nhưng phải có cơ chế thu hồi.

**Refresh token rotation:** mỗi lần refresh phát token mới và vô hiệu token cũ; nếu phát hiện một refresh token cũ được tái sử dụng (reuse detection) → coi như bị đánh cắp, thu hồi cả family. JWT bản chất stateless nên không tự revoke được — muốn logout/ban tức thì cần lưu trạng thái refresh token (DB/Redis) hoặc blacklist jti.

```ts
async rotate(oldRefresh: string) {
  const record = await this.store.find(oldRefresh);
  if (!record || record.revoked) throw new UnauthorizedException(); // reuse detection
  await this.store.revoke(oldRefresh);
  return this.issuePair(record.userId); // access ngắn hạn + refresh mới
}
```

Lưu ý: luôn ký bằng secret/khoá đủ mạnh, set `expiresIn`, và validate `iss`/`aud` nếu có nhiều service.
</details>

<details>
<summary><b>16.10 [BỔ SUNG] Bảo mật dependency/supply-chain trong dự án Node/NestJS như thế nào?</b></summary>

Phần lớn code chạy trong app thực ra đến từ `node_modules`, nên dependency là bề mặt tấn công lớn (OWASP A06 Vulnerable & Outdated Components, và A08 Software/Data Integrity).

Các biện pháp thực tế:
- **Quét lỗ hổng định kỳ**: `pnpm audit`/`npm audit`, tích hợp Dependabot/Renovate hoặc Snyk vào CI để cảnh báo CVE và tự mở PR nâng cấp.
- **Khóa phiên bản**: commit lockfile (`pnpm-lock.yaml`) và cài bằng `--frozen-lockfile` trong CI để build tái lập, tránh kéo bản vá độc hại bất ngờ.
- **Giảm bề mặt**: hạn chế dependency không cần thiết, cảnh giác typosquatting (tên gói gần giống), và xem postinstall script lạ.
- **Tách secret khỏi build**: không để token CI/registry lộ trong log; dùng `provenance` khi publish package nội bộ.

```bash
pnpm audit --audit-level=high   # fail CI nếu có lỗ hổng mức high+
pnpm install --frozen-lockfile  # build tái lập, không tự đổi version
```

Ví dụ thực tế: cấu hình Dependabot cho repo, chạy `pnpm audit` trong pipeline và chặn merge nếu có CVE high/critical chưa xử lý — đây là phần hay bị bỏ quên nhưng reviewer Senior thường hỏi để đánh giá tư duy vận hành.
</details>

---

## 17. Observability and Logging

<details>
<summary><b>17.1 Tại sao nên dùng structured logging (JSON) thay vì log dạng text thuần? Lợi ích là gì?</b></summary>

Structured logging xuất mỗi log là một JSON object với các field cố định (timestamp, level, message, context, requestId...) thay vì một chuỗi text tự do. Lợi ích chính là log có thể được **parse và index** bởi các hệ thống như Elasticsearch, Loki, Datadog, cho phép query/filter/aggregate theo field (ví dụ lọc tất cả log của một `requestId` hoặc `userId`). Text thuần rất khó parse và dễ vỡ khi format thay đổi.

Lưu ý: trong NestJS `Logger` mặc định, `logger.log(message, context)` nhận tham số thứ hai là *context* (string), không phải metadata object — nếu truyền object vào nó sẽ không được tách field đúng ý. Để có structured logging thật sự nên dùng một logger như **Pino** (`nestjs-pino`) hoặc **Winston** với JSON formatter; ở môi trường production tắt pretty-print để xuất JSON một dòng (one-line JSON) cho dễ ingest.

```ts
// Thay vi: logger.log(`User ${userId} created issue ${key}`)
// Voi pino-style logger (vd: PinoLogger):
logger.info(
  { userId, issueKey: key, projectId },  // metadata fields
  'Issue created',                        // message
);
// pino tu them timestamp & level, khong can set tay
```

Ví dụ thực tế: Trong Jira Clone tôi log dạng JSON kèm `requestId` và `userId`, nhờ đó khi một user báo lỗi tôi có thể **query theo field** (không phải grep text) trong log aggregator để lấy toàn bộ request flow của họ trong vài giây.
</details>

<details>
<summary><b>17.2 Phân biệt các log level INFO / WARN / ERROR. Khi nào dùng từng loại?</b></summary>

**INFO**: sự kiện bình thường, đáng quan tâm về mặt nghiệp vụ (user đăng nhập, tạo issue, cron chạy xong). **WARN**: tình huống bất thường nhưng hệ thống vẫn xử lý được, cần để ý (rate limit gần chạm ngưỡng, retry một lần thành công, deprecated API được gọi). **ERROR**: thất bại thật sự cần hành động (exception không mong đợi, 5xx, kết nối DB rớt). Quy tắc quan trọng: **400/401/403/404 không phải ERROR** — đó là lỗi phía client và kỳ vọng xảy ra, chỉ nên log ở mức WARN hoặc INFO (hoặc thường là bỏ qua/log gọn) để tránh nhiễu và alert fatigue. Ngoài ra còn có **DEBUG/TRACE** (chi tiết kỹ thuật, thường tắt ở production) và **FATAL** (lỗi nghiêm trọng khiến process không thể tiếp tục).

Ví dụ thực tế: Trong Jira Clone, Exception Filter của tôi map `statusCode >= 500` thành level ERROR, còn 4xx thành WARN — nhờ vậy dashboard ERROR chỉ chứa lỗi server thật sự cần fix.
</details>

<details>
<summary><b>17.3 Kiến trúc log request: tại sao dùng Interceptor cho success và Exception Filter cho lỗi?</b></summary>

Đây là sự phân tách trách nhiệm theo vòng đời request của NestJS. **Interceptor** bọc quanh handler: nó đo được thời gian xử lý (latency) và log khi response trả về bình thường (qua `tap`/`finalize` của RxJS). **Exception Filter** chỉ được kích hoạt khi có exception bị ném ra — đây là nơi tập trung để log lỗi một cách nhất quán và định dạng response error. Tách 2 nơi giúp mỗi bên chỉ lo một luồng, tránh `try/catch` rải rác trong từng controller.

Lưu ý quan trọng: `tap` chỉ chạy ở nhánh **next** (success). Để đo latency kể cả khi lỗi, nên dùng `tap({ next, error })` hoặc `finalize` (chạy ở cả success lẫn error). Nếu chỉ dùng `tap(() => ...)` thì request bị lỗi sẽ **không** được log latency bởi interceptor — phần log lỗi sẽ do Exception Filter đảm nhận.

```ts
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(LoggingInterceptor.name);
  intercept(ctx: ExecutionContext, next: CallHandler) {
    const start = Date.now();
    const req = ctx.switchToHttp().getRequest();
    return next.handle().pipe(
      tap({
        next: () => this.logger.log({
          msg: 'request completed',
          method: req.method,
          url: req.url,
          requestId: req.requestId,
          durationMs: Date.now() - start,
        }),
      }),
    );
  }
}
```

Ví dụ thực tế: Trong Jira Clone, success đi qua LoggingInterceptor còn lỗi đi qua AllExceptionsFilter, cả hai cùng đưa `requestId` vào log nên ghép được một request hoàn chỉnh.
</details>

<details>
<summary><b>17.4 Tại sao logging không nên block request? Buffered/batched write giải quyết điều này như thế nào?</b></summary>

Nếu mỗi log là một lệnh ghi I/O đồng bộ (vd `fs.writeFileSync`, hoặc `await` một HTTP call tới log service) ngay trong luồng request, nó sẽ thêm latency vào mỗi response. Đặc biệt **synchronous I/O** (như `*Sync`) sẽ chặn event loop của Node và làm giảm throughput toàn service. Lưu ý: I/O bất đồng bộ (async) không chặn event loop, nhưng `await` nó trong request vẫn kéo dài thời gian phản hồi. Giải pháp là **buffer trong bộ nhớ** rồi **flush theo batch** định kỳ (theo số lượng hoặc theo thời gian) bằng ghi bất đồng bộ ở background. Request chỉ đẩy log vào buffer (gần như tức thời) rồi trả về ngay. Đánh đổi là rủi ro mất vài log cuối nếu process crash trước khi flush — nên flush khi shutdown (`onApplicationShutdown` / `beforeExit`).

```ts
private buffer: LogEntry[] = [];
private flushing = false;

write(entry: LogEntry) {
  this.buffer.push(entry);            // non-blocking
  if (this.buffer.length >= 100) void this.flush();
}

@Cron('*/5 * * * * *')                // flush moi 5s
async flush() {
  if (this.flushing || !this.buffer.length) return;
  this.flushing = true;
  const batch = this.buffer.splice(0); // lay het & reset
  try {
    await this.sink.writeBatch(batch); // ghi async
  } catch (e) {
    this.buffer.unshift(...batch);     // tra lai buffer neu ghi loi
  } finally {
    this.flushing = false;
  }
}
```

Lưu ý: các thư viện như **Pino** đã xử lý vấn đề này sẵn (ghi async, có thể qua worker thread với `pino.transport`), nên trong thực tế thường tận dụng thư viện thay vì tự viết buffer. Tự viết buffer cần cảnh giác mất log khi crash và race condition khi flush.

Ví dụ thực tế: Jira Clone gom log vào buffer và flush theo định kỳ, nên latency của endpoint không bị ảnh hưởng bởi tốc độ ghi log.
</details>

<details>
<summary><b>17.5 Correlation ID / Request ID là gì và tại sao nó quan trọng trong hệ thống?</b></summary>

Correlation ID (hay Request ID) là một định danh duy nhất (ví dụ UUID) gắn cho mỗi request ngay khi nó đi vào hệ thống, rồi được đính kèm vào **tất cả** log sinh ra trong quá trình xử lý request đó. Nó cho phép **truy vết xuyên suốt** một request — từ controller, service, đến DB query — chỉ bằng cách lọc theo ID, đặc biệt quan trọng khi có nhiều request đồng thời trộn lẫn trong log. Trong hệ microservice, ID này được truyền qua header (ví dụ `X-Request-Id`) giữa các service để trace toàn hệ thống. Thường dùng middleware + `AsyncLocalStorage` để lưu ID theo context mà không phải truyền tay qua mỗi hàm.

Lưu ý kỹ thuật: phải gọi `als.run(...)` sao cho **toàn bộ phần còn lại của request** chạy bên trong callback đó thì store mới còn sống. Với Express middleware, gọi `next()` bên trong `als.run()` là đúng; và trong context của interceptor/exception filter ta đọc lại ID qua `als.getStore()`.

```ts
import { AsyncLocalStorage } from 'node:async_hooks';
export const als = new AsyncLocalStorage<{ requestId: string }>();

export function requestIdMiddleware(req, res, next) {
  const id = req.headers['x-request-id'] ?? randomUUID();
  req.requestId = id;
  res.setHeader('x-request-id', id);
  als.run({ requestId: id }, () => next()); // moi xu ly sau next() thay duoc store
}
```

Ví dụ thực tế: Trong Jira Clone, mỗi log entry đều có `requestId`, nên khi user gửi ID này trong bug report tôi lọc ra được đúng luồng request gây lỗi.
</details>

<details>
<summary><b>17.6 Tại sao chỉ gửi Sentry cho lỗi 5xx và chỉ ở môi trường production?</b></summary>

**Chỉ 5xx**: Sentry dùng để alert và theo dõi lỗi server thật sự cần fix. Lỗi 4xx (validation fail, 401, 404) là lỗi client và xảy ra thường xuyên — nếu đẩy hết lên Sentry sẽ tạo noise, tốn quota và làm loãng những lỗi quan trọng. **Chỉ production**: lỗi ở local/dev khi đang code là bình thường, nếu báo cáo lên sẽ gây nhiễu và làm sai metric. Nên lọc theo status trong Exception Filter trước khi `captureException`. Về việc bật/tắt theo môi trường: nên điều khiển bằng `Sentry.init({ enabled: process.env.NODE_ENV === 'production' })` (Sentry vẫn init nhưng không gửi khi disabled) — đây an toàn hơn việc hoàn toàn không gọi `init`, vì khi đó các API như `captureException` vẫn là no-op an toàn thay vì có thể gây lỗi.

Lưu ý: nên set `Sentry.init({ environment: process.env.NODE_ENV })` để tách môi trường trên dashboard, và dùng `dsn` khác/empty cho dev. Với NestJS hiện có `@sentry/nestjs` cung cấp sẵn filter/integration thay vì gọi `captureException` thủ công.

```ts
// trong Exception Filter
const status =
  exception instanceof HttpException ? exception.getStatus() : 500;
if (status >= 500) {
  Sentry.captureException(exception, {
    tags: { requestId: req.requestId },
  });
  // viec co thuc su gui di hay khong da duoc kiem soat boi Sentry.init({ enabled })
}
```

Ví dụ thực tế: Dùng đúng kiến trúc này trong Jira Clone — Sentry chỉ bật khi prod và chỉ nhận 5xx, nên alert nhận được đều là vấn đề cần xử lý thực sự.
</details>

<details>
<summary><b>17.7 Cron retention xóa log cũ hoạt động thế nào và tại sao cần nó?</b></summary>

Log tăng liên tục sẽ làm đầy disk/DB và làm chậm truy vấn. Retention là job chạy định kỳ (ví dụ mỗi đêm qua `@Cron`) **xóa các log cũ hơn ngưỡng giữ lại** (ví dụ 30 ngày). Khi xóa khối lớn nên làm **theo batch** (vd xóa 10k dòng mỗi vòng, lặp đến khi hết) để tránh khóa bảng lâu, transaction quá to và làm phình WAL. Với DB, nên có index trên cột `createdAt`/`timestamp` để query xóa nhanh; với quy mô lớn hơn **table partitioning** + `DROP PARTITION` là cách tối ưu nhất vì gần như tức thời và không sinh ra bloat/vacuum nặng như `DELETE`.

Lưu ý: `Date.now() - 30 * 864e5` là cách viết tắt đúng (`864e5` = 86_400_000 ms = 1 ngày), nhưng nên viết rõ ràng `30 * 24 * 60 * 60 * 1000` cho dễ đọc. `prisma.deleteMany` xóa một lần tất cả bản ghi thỏa điều kiện — với bảng rất lớn nên tự chia batch + giới hạn số dòng mỗi lần thay vì một lệnh DELETE không giới hạn.

```ts
@Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)
async purgeOldLogs() {
  const ONE_DAY = 24 * 60 * 60 * 1000;
  const cutoff = new Date(Date.now() - 30 * ONE_DAY); // 30 ngay
  let total = 0;
  // xoa theo batch de tranh khoa bang lau
  for (;;) {
    const oldIds = await this.prisma.log.findMany({
      where: { createdAt: { lt: cutoff } },
      select: { id: true },
      take: 10_000,
    });
    if (oldIds.length === 0) break;
    const { count } = await this.prisma.log.deleteMany({
      where: { id: { in: oldIds.map((l) => l.id) } },
    });
    total += count;
  }
  this.logger.log({ msg: 'log retention done', deleted: total });
}
```

Ví dụ thực tế: Jira Clone có `@Cron` xóa log quá hạn mỗi ngày, giữ bảng log gọn nên query điều tra sự cố vẫn nhanh.
</details>

<details>
<summary><b>17.8 Health check endpoint dùng để làm gì và nên kiểm tra những gì?</b></summary>

Health check (ví dụ `/health`) là endpoint cho **load balancer, Kubernetes, hoặc uptime monitor** biết service có sẵn sàng phục vụ hay không. Nên tách **liveness** (process còn sống chưa — nếu fail, orchestrator restart pod) và **readiness** (các dependency thiết yếu — DB, Redis... có OK không — nếu fail, ngừng định tuyến traffic tới pod nhưng không restart). NestJS có `@nestjs/terminus` hỗ trợ sẵn. Endpoint này phải là `@Public` (không yêu cầu auth) và nên nhẹ, trả về status của từng thành phần để khi đỏ, biết thành phần nào hỏng.

Lưu ý: kiểm tra disk trong readiness cẩn thận — nếu disk gần đầy làm readiness fail, pod bị rút khỏi load balancer và có thể gây cascading failure; nhiều nơi chỉ đưa disk vào liveness/cảnh báo riêng chứ không gắn vào readiness của traffic. Với container, `path` nên trỏ tới volume thực tế (vd `'/'` hoặc đường dẫn data), không nên hardcode bước nhầm.

```ts
@Public()
@Get('health')
@HealthCheck()
check() {
  return this.health.check([
    () => this.db.pingCheck('database'),                 // readiness: DB
    () => this.disk.checkStorage('disk', {
      thresholdPercent: 0.9, path: '/',                  // canh bao khi disk > 90%
    }),
  ]);
}
```

Ví dụ thực tế: Trong Jira Clone health check được đánh dấu `@Public` để bỏ qua global JWT guard, và ping Postgres qua Prisma để readiness phản ánh đúng trạng thái DB.
</details>

---

## 18. Advanced NestJS

<details>
<summary><b>18.1 Global providers (APP_GUARD/APP_FILTER/APP_INTERCEPTOR) khác gì so với useGlobalGuards()? Tại sao nên dùng token này? Nhiều APP_GUARD có chạy đúng thứ tự không?</b></summary>

Khi đăng ký guard/filter/interceptor qua `app.useGlobalGuards()` thì instance được tạo thủ công **bên ngoài DI container**, nên không inject được dependency (ví dụ không inject được `Reflector` hay service). Ngược lại, đăng ký bằng provider token `APP_GUARD`, `APP_FILTER`, `APP_INTERCEPTOR` trong module thì NestJS tạo chúng qua DI container — vì vậy có thể inject `Reflector`, service, config... Một module có thể cung cấp nhiều provider cho cùng một token (ví dụ nhiều `APP_GUARD`).

LƯU Ý QUAN TRỌNG (bẫy thường gặp khi phỏng vấn): với nhiều guard đăng ký cùng qua token `APP_GUARD`, NestJS **không đảm bảo thứ tự thực thi** — vì chúng được resolve từ DI container chứ không được bind tuần tự như `@UseGuards(A, B)` hay `useGlobalGuards(a, b)` (hai cách này mới chạy đúng thứ tự A rồi B). Vì thế dùng nói "chúng chạy theo thứ tự khai báo" là không chính xác. Nếu cần đảm bảo thứ tự (ví dụ auth phải chạy trước authz), nên gộp logic vào một guard, hoặc đặt global guard auth qua `APP_GUARD` còn `RolesGuard` thì bind ở cấp controller/handler bằng `@UseGuards`.

```ts
@Module({
  providers: [
    // Thu tu giua hai guard nay KHONG duoc dam bao
    { provide: APP_GUARD, useClass: JwtAuthGuard },
    { provide: APP_GUARD, useClass: RolesGuard },
  ],
})
export class AppModule {}
```

Ví dụ thực tế: Trong Jira Clone tôi dùng `APP_GUARD` cho `JwtAuthGuard` global để mặc định mọi route đều cần xác thực, và guard này inject `Reflector` đọc metadata `@Public()`. Phần kiểm tra role thì `RolesGuard` tự đọc `req.user` (đã có sau khi qua auth) nên không phụ thuộc thứ tự hai guard, hoặc tôi bind `RolesGuard` ở cấp route cần phân quyền.
</details>

<details>
<summary><b>18.2 Mô tả pattern global JwtAuthGuard + @Public(). Làm sao một route opt-out khỏi xác thực?</b></summary>

Ý tưởng là "secure by default": đặt `JwtAuthGuard` global qua `APP_GUARD` nên mọi endpoint đều yêu cầu JWT, rồi dùng custom decorator `@Public()` (gán metadata bằng `SetMetadata`) để đánh dấu route không cần auth. Trong guard, `Reflector.getAllAndOverride` đọc key `IS_PUBLIC` ở cả handler và class; nếu là public thì `canActivate` trả về `true` ngay, bỏ qua việc verify token.

Lưu ý: `super.canActivate(ctx)` của `AuthGuard('jwt')` có thể trả về `boolean | Promise<boolean> | Observable<boolean>`, nên hàm `canActivate` của bạn nên khai báo kiểu trả về tương ứng (hoặc dùng `async`/`await`) thay vì ép về `boolean`.

```ts
import { Injectable, ExecutionContext, SetMetadata } from '@nestjs/common';
import { Reflector } from '@nestjs/core';
import { AuthGuard } from '@nestjs/passport';
import { Observable } from 'rxjs';

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

Ví dụ thực tế: Các route `login`, `register`, `refresh` trong Jira Clone gán `@Public()`, còn lại toàn bộ API được bảo vệ mặc định.
</details>

<details>
<summary><b>18.3 Custom param decorator (@CurrentUser) hoạt động thế nào? Khác với class/method decorator ở chỗ nào?</b></summary>

Custom param decorator tạo bằng `createParamDecorator`, nhận callback `(data, ctx: ExecutionContext)` chạy **lúc Nest resolve các tham số của handler** (sau khi guard đã chạy, và diễn ra trong cùng giai đoạn với pipe) — nó trích xuất giá trị từ request và trả trực tiếp vào argument của handler. Nó KHÔNG gán metadata, mà trả về giá trị. Trong khi đó class/method decorator (ví dụ tạo bằng `SetMetadata`) chỉ gán metadata để Guard/Interceptor đọc qua `Reflector` sau này, chứ không tự mình chèn giá trị vào handler. Param decorator có thể nhận tham số `data` để chọn trường cụ thể.

Lưu ý: pipe vẫn áp dụng lên giá trị mà param decorator trả về, nên bạn vẫn có thể làm `@CurrentUser(ValidationPipe)` hoặc validate giá trị trích xuất.

```ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';

export const CurrentUser = createParamDecorator(
  (data: string | undefined, ctx: ExecutionContext) => {
    const req = ctx.switchToHttp().getRequest();
    return data ? req.user?.[data] : req.user;
  },
);
// dung: @CurrentUser() user: User  hoac  @CurrentUser('id') userId: string
```

Ví dụ thực tế: Trong Jira Clone `@CurrentUser('id')` lấy userId từ payload JWT đã được Passport strategy gán vào `req.user` để lọc issue theo người tạo.
</details>

<details>
<summary><b>18.4 Phân biệt useClass, useValue, useFactory, useExisting. Khi nào dùng useFactory với inject?</b></summary>

Bốn cách cung cấp provider: `useClass` — Nest tự khởi tạo class (có thể swap implementation theo token); `useValue` — cung cấp một giá trị/object đã có sẵn (mock, hằng số, config object); `useFactory` — chạy hàm để tạo giá trị động, hỗ trợ `inject: []` để bơm dependency vào factory và có thể là `async` (return Promise); `useExisting` — tạo alias trỏ tới một provider khác đã tồn tại (chia sẻ cùng instance singleton, không tạo mới). Dùng `useFactory` khi giá trị cần tính toán lúc runtime, cần async, hoặc phụ thuộc provider khác.

Lưu ý: thứ tự phần tử trong mảng `inject` phải khớp với thứ tự tham số của factory.

```ts
{
  provide: 'PRISMA_OPTIONS',
  useFactory: (config: ConfigService) => ({ url: config.get('DATABASE_URL') }),
  inject: [ConfigService],
}
```

Ví dụ thực tế: Jira Clone dùng `useValue` cho Sentry chỉ ở prod (truyền client đã cấu hình), và `useFactory` + `inject: [ConfigService]` để khởi tạo client email Resend từ biến môi trường.
</details>

<details>
<summary><b>18.5 Injection scope DEFAULT/REQUEST/TRANSIENT khác nhau ra sao? Cái giá của REQUEST scope?</b></summary>

`DEFAULT` (singleton): một instance dùng chung toàn ứng dụng, khởi tạo một lần, hiệu năng tốt nhất. `REQUEST`: tạo instance mới cho mỗi request, có thể inject đối tượng `REQUEST` — nhưng có hiệu ứng **bubble up**: bất kỳ provider nào phụ thuộc (trực tiếp hoặc gián tiếp) vào một provider request-scoped cũng trở thành request-scoped, gây overhead vì phải khởi tạo lại theo mỗi request và ảnh hưởng throughput. `TRANSIENT`: mỗi consumer inject provider sẽ nhận một instance riêng (không chia sẻ giữa các nơi tiêu thụ).

Nên tránh REQUEST scope trên các provider được dùng rộng; thay vào đó có thể dùng `AsyncLocalStorage` để lưu context theo request mà vẫn giữ provider là singleton.

```ts
import { Injectable, Scope, Inject } from '@nestjs/common';
import { REQUEST } from '@nestjs/core';
import { Request } from 'express';

@Injectable({ scope: Scope.REQUEST })
export class RequestContextService {
  constructor(@Inject(REQUEST) private req: Request) {}
}
```

Ví dụ thực tế: Trong Jira Clone tôi giữ logger là singleton và dùng `AsyncLocalStorage` lưu requestId/userId thay vì REQUEST scope, để tránh kéo nhiều provider thành request-scoped.
</details>

<details>
<summary><b>18.6 Các lifecycle hooks onModuleInit và onApplicationShutdown dùng để làm gì? Làm sao bật shutdown hooks?</b></summary>

`onModuleInit` chạy sau khi các dependency của module đã được khởi tạo xong — thích hợp để khởi tạo kết nối, seed dữ liệu, warm cache. `onApplicationShutdown(signal)` chạy khi app nhận tín hiệu dừng (ví dụ SIGTERM/SIGINT) — dùng để đóng kết nối DB, flush buffer, hủy cron, graceful shutdown. Các shutdown hook chỉ kích hoạt khi bạn gọi `app.enableShutdownHooks()` (init hook thì luôn chạy, không cần bật).

Về thứ tự: với `onModuleInit`, module phụ thuộc (dependency) được khởi tạo trước rồi đến module phụ thuộc vào chúng; các shutdown hook chạy theo thứ tự ngược lại với init.

```ts
import { Injectable, OnApplicationShutdown } from '@nestjs/common';

@Injectable()
export class LogService implements OnApplicationShutdown {
  async onApplicationShutdown(signal?: string) {
    await this.flushBuffer(); // ghi not log con trong buffer
  }
}

// main.ts
app.enableShutdownHooks();
```

Ví dụ thực tế: Logger có buffer của Jira Clone flush hết log còn lại trong `onApplicationShutdown`, tránh mất log khi container bị kill lúc deploy.
</details>

<details>
<summary><b>18.7 Trình bày thứ tự thực thi: Middleware -> Guard -> Interceptor -> Pipe -> Handler -> (Interceptor) -> Filter. Filter chạy khi nào và resolve theo thứ tự nào?</b></summary>

Khi request vào: **Middleware** (cấp Express/Fastify, chưa có `ExecutionContext`) chạy trước; rồi **Guard** quyết định cho qua hay không (auth/authz); rồi **Interceptor (phần trước handler)** có thể biến đổi/wrap; rồi **Pipe** validate và transform tham số; sau đó **Handler** (controller method) chạy; tiếp theo **Interceptor (phần sau)** xử lý response trả về qua RxJS stream. **Exception Filter** chỉ kích hoạt khi có exception được ném ở bất kỳ giai đoạn nào (guard, interceptor, pipe, handler...), để biến lỗi thành response. Nhớ: guard chạy trước pipe, nên validation chỉ xảy ra sau khi đã qua auth.

Lưu ý về thứ tự resolve của filter: khác với các thành phần khác (global resolve trước), exception filter resolve từ **cụ thể đến tổng quát** — route-level trước, rồi controller-level, cuối cùng mới đến global filter; filter khớp đầu tiên sẽ xử lý exception.

```ts
// Interceptor minh hoa do hai phan truoc/sau handler
import { tap } from 'rxjs/operators';

intercept(ctx: ExecutionContext, next: CallHandler) {
  const start = Date.now();
  return next.handle().pipe(tap(() => log(Date.now() - start)));
}
```

Ví dụ thực tế: Jira Clone dùng interceptor đo thời gian xử lý + chuẩn hóa response, còn `AllExceptionsFilter` map lỗi Prisma (ví dụ `P2002` unique constraint) sang HTTP 409 và gửi Sentry nếu là lỗi 5xx.
</details>

<details>
<summary><b>18.8 Dynamic module là gì? Phân biệt forRoot và forRootAsync.</b></summary>

Dynamic module là module trả về cấu hình động qua một static method (thay vì chỉ dùng decorator cố định), cho phép truyền options và đăng ký provider/exports lúc runtime — là nền tảng cho các module tái sử dụng như `ConfigModule`, `JwtModule`. `forRoot(options)` nhận cấu hình **đồng bộ** (giá trị có sẵn ngay lúc gọi); `forRootAsync(options)` cho phép cấu hình **bất đồng bộ**, dùng `useFactory` + `inject` để lấy options từ provider khác (ví dụ `ConfigService`) — hữu ích khi config phụ thuộc biến môi trường/provider được load trước đó.

```ts
import { Module, DynamicModule } from '@nestjs/common';

export const MAIL_OPTIONS = 'MAIL_OPTIONS';

@Module({})
export class MailModule {
  static forRootAsync(opts: MailAsyncOptions): DynamicModule {
    return {
      module: MailModule,
      imports: opts.imports ?? [], // can import module chua provider duoc inject (vi du ConfigModule)
      providers: [
        { provide: MAIL_OPTIONS, useFactory: opts.useFactory, inject: opts.inject ?? [] },
      ],
      exports: [MAIL_OPTIONS],
    };
  }
}
```

Ví dụ thực tế: Trong Jira Clone `JwtModule.registerAsync` lấy secret và thời hạn token từ `ConfigService`, nhờ đó secret không hardcode mà đọc từ `.env` đã validate.
</details>

<details>
<summary><b>18.9 Tại sao nên validate biến môi trường (.env) và làm sao làm trong NestJS?</b></summary>

Câu bổ sung ý quan trọng còn thiếu. Nếu không validate, một biến sai/thiếu (ví dụ `DATABASE_URL`, `JWT_SECRET`) chỉ lộ ra lúc runtime, thường là khi đã deploy và ở đúng endpoint dùng đến nó — khó debug và nguy hiểm (ví dụ secret rỗng khiến JWT vẫn ký được nhưng không an toàn). "Fail fast" lúc khởi động là chuẩn production.

Trong NestJS: dùng `ConfigModule.forRoot({ validationSchema })` với Joi, hoặc `validate` callback với `class-validator` + `class-transformer`. Nếu schema không khớp, app ném lỗi ngay lúc bootstrap.

```ts
import * as Joi from 'joi';

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

Ví dụ thực tế: Jira Clone validate `.env` bằng Joi lúc bootstrap; thiếu `JWT_SECRET` thì app crash ngay, tránh deploy nhầm cấu hình thiếu lên production.
</details>

<details>
<summary><b>18.10 ValidationPipe global hoạt động thế nào? Vì sao nên bật whitelist/transform và dùng DTO + class-validator?</b></summary>

Câu bổ sung ý quan trọng còn thiếu (validation là phần core khi phỏng vấn NestJS). `ValidationPipe` dùng `class-validator` (đọc decorator như `@IsString()`, `@IsEmail()`) và `class-transformer` để validate + transform body/query/param theo DTO. Bật global qua `app.useGlobalPipes(new ValidationPipe(...))` hoặc token `APP_PIPE`.

Các option quan trọng: `whitelist: true` loại bỏ các field không khai báo trong DTO (chống over-posting); `forbidNonWhitelisted: true` ném lỗi nếu có field thừa; `transform: true` chuyển payload thành instance của DTO và ép kiểu primitive (ví dụ param `:id` string sang number nếu DTO khai báo number). Lưu ý `transform` cần có `enableImplicitConversion` hoặc decorator để ép kiểu số/boolean từ query string.

```ts
// main.ts
app.useGlobalPipes(new ValidationPipe({
  whitelist: true,
  forbidNonWhitelisted: true,
  transform: true,
}));

// dto
export class CreateIssueDto {
  @IsString() @MaxLength(200)
  title: string;

  @IsOptional() @IsEnum(IssueStatus)
  status?: IssueStatus;
}
```

Ví dụ thực tế: Jira Clone dùng `whitelist + forbidNonWhitelisted` để client không thể gán bừa field như `ownerId` qua body, và `transform` để DTO nhận đúng kiểu trước khi vào service.
</details>

---

## 19. Database, Transactions and Concurrency

<details>
<summary><b>19.1 $transaction trong Prisma hoạt động thế nào và nó đảm bảo ACID ra sao?</b></summary>

`$transaction` gom nhiều query vào một transaction DB duy nhất: hoặc tất cả commit, hoặc rollback toàn bộ nếu có lỗi. ACID được đảm bảo ở tầng DB (Postgres), Prisma chỉ là client mở `BEGIN/COMMIT/ROLLBACK`: **Atomicity** (all-or-nothing), **Consistency** (không vi phạm constraint/trigger), **Isolation** (theo isolation level cấu hình), **Durability** (sau commit dù crash vẫn còn). Prisma có hai dạng: *sequential* (`$transaction([q1, q2])` — chạy tuần tự, không chèn logic JS ở giữa) và *interactive* (`$transaction(async tx => {...})` — cho phép chạy logic/điều kiện giữa các bước).

Lưu ý quan trọng về interactive transaction: nó giữ connection và các lock mở cho đến khi callback kết thúc, nên MỌI đặt code I/O chậm (gọi API ngoài, gửi mail) vào trong callback — điều này kéo dài thời gian giữ lock, tăng lock contention và nguy cơ deadlock. Cấu hình tham số: `maxWait` (thời gian tối đa CHỜ để lấy được connection từ pool), `timeout` (thời gian tối đa transaction được phép chạy trước khi bị hủy). Đặt hai giá trị này hợp lý để tránh giữ connection/lock quá lâu.

```ts
await prisma.$transaction(async (tx) => {
  const acc = await tx.account.update({
    where: { id }, data: { balance: { decrement: 100 } },
  });
  if (acc.balance < 0) throw new Error('Insufficient funds'); // rollback toan bo
  await tx.ledger.create({ data: { accountId: id, amount: -100 } });
}, { maxWait: 5000, timeout: 10000 });
```

Ví dụ thực tế: Trong Jira Clone, khi tạo issue mình vừa tăng counter của project vừa insert issue trong cùng một `$transaction` interactive — nếu insert thất bại thì counter cũng được rollback, tránh "nhảy số" key. (Lưu ý: kiểm tra số dư ngay trên kết quả `update` như ví dụ chỉ an toàn vì `decrement` là atomic; nếu đọc-rồi-trừ riêng lẻ thì vẫn bị race, xem câu về race condition.)
</details>

<details>
<summary><b>19.2 Làm sao sinh issue key tuần tự PROJECT-N không bị trùng khi có nhiều request đồng thời?</b></summary>

Không được đọc counter ra app rồi +1 rồi ghi lại (read-modify-write) vì hai request có thể đọc cùng giá trị → trùng key. Cách đúng: thực hiện **atomic increment ngay tại DB**, dùng `{ increment: 1 }` (sinh ra `SET counter = counter + 1` trong một câu UPDATE). Khi đó DB tự lấy row lock trên row counter trong suốt câu UPDATE, nên các update đồng thời bị serialize và mỗi request nhận một giá trị duy nhất. Kết hợp **unique constraint** trên `(projectId, number)` làm lớp bảo vệ cuối cùng (vd nếu logic sinh key bị lỗi ở chỗ khác).

Lưu ý: với atomic `increment` thì KHÔNG cần `SELECT ... FOR UPDATE` riêng — bản thân câu UPDATE đã khóa row. `FOR UPDATE` chỉ cần khi bạn phải ĐỌC giá trị rồi mới quyết định ghi trong nhiều bước.

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

Ví dụ thực tế: Đây là cách Jira Clone sinh key. `increment` ở DB + `@@unique([projectId, number])` đảm bảo không bao giờ có hai issue cùng key dù 100 người tạo issue cùng lúc.
</details>

<details>
<summary><b>19.3 Race condition là gì? Nêu các chiến lược chính để xử lý trong backend?</b></summary>

Race condition xảy ra khi kết quả phụ thuộc vào thứ tự/timing của nhiều thao tác đồng thời trên cùng dữ liệu, điển hình là pattern read-modify-write (đọc tồn kho rồi trừ, đọc counter rồi tăng) — giữa bước đọc và bước ghi, request khác có thể chèn vào. Các chiến lược xử lý: (1) **Atomic DB operation** (`increment`/`decrement`, `UPDATE ... WHERE <dieu kien>`) — đẩy cả kiểm tra lẫn ghi xuống một câu lệnh DB; (2) **Pessimistic lock** với `SELECT ... FOR UPDATE` khóa row trước khi sửa (phải nằm trong transaction); (3) **Optimistic lock** với cột `version`, retry khi conflict; (4) **Unique constraint** để DB tự chặn bản ghi trùng; (5) **Serializable isolation** cho các case phức tạp như write-skew. Chọn dựa trên mức độ tranh chấp: tranh chấp thấp → optimistic, tranh chấp cao → pessimistic; trường hợp đơn giản → ưu tiên atomic.

```sql
-- Atomic: chi tru neu con du ton kho, tranh oversell
UPDATE inventory SET qty = qty - 1
WHERE product_id = $1 AND qty > 0;
-- Kiem tra so row bi anh huong (rowCount) === 0 => het hang, ket qua dat that bai
```

Ví dụ thực tế: Ở Jira Clone, khi hai user kéo cùng một card sang cột khác, mình dùng optimistic lock theo `version` để phát hiện conflict và báo client refetch thay vì ghi đè mù quáng.
</details>

<details>
<summary><b>19.4 Phân biệt optimistic lock (version) và pessimistic lock (SELECT FOR UPDATE)? Khi nào dùng cái nào?</b></summary>

**Optimistic lock**: không khóa row trước; thêm cột `version`, khi update thêm điều kiện `WHERE version = X` và đặt `version = version + 1`. Nếu rowCount = 0 nghĩa là không có row nào khớp (người khác đã update và đổi version trước đó) → app phát hiện conflict rồi retry hoặc báo lỗi (vd HTTP 409). Tốt khi tranh chấp THẤP: không giữ lock nên throughput cao. **Pessimistic lock**: `SELECT ... FOR UPDATE` khóa row ngay khi đọc và giữ đến hết transaction, request khác dùng cùng row phải chờ. Tốt khi tranh chấp CAO hoặc khi cần đảm bảo tuyệt đối không ai chèn vào giữa đọc và ghi, đổi lại giảm concurrency và có rủi ro deadlock. Tóm lại: optimistic = "cứ làm, nếu đụng độ thì retry"; pessimistic = "khóa trước cho chắc".

```sql
-- Optimistic
UPDATE issue SET status='DONE', version = version + 1
WHERE id = $1 AND version = $2;  -- rowCount=0 => conflict, retry

-- Pessimistic (phai trong transaction)
BEGIN;
SELECT * FROM issue WHERE id = $1 FOR UPDATE; -- khoa row
UPDATE issue SET status='DONE' WHERE id = $1;
COMMIT;
```

Ví dụ thực tế: Sinh issue key PROJECT-N thực chất là atomic increment trên row counter (UPDATE tự khóa row), không phải pessimistic lock thủ công; còn cập nhật nội dung issue khi có nhiều editor thì dùng optimistic `version` để tránh ghi đè của nhau.
</details>

<details>
<summary><b>19.5 Nêu các isolation level của PostgreSQL và các vấn đề chúng phòng tránh.</b></summary>

PostgreSQL hỗ trợ 4 mức (nhưng thực tế chỉ có 3 hành vi): **Read Committed** (mặc định) — mỗi statement chỉ thấy dữ liệu đã commit tại thời điểm statement đó bắt đầu, tránh dirty read nhưng vẫn có non-repeatable read và phantom read; **Repeatable Read** — dùng snapshot cố định suốt transaction, tránh dirty + non-repeatable read, và ở Postgres tránh luôn phantom read (mạnh hơn chuẩn SQL), NHƯNG vẫn còn write-skew và có thể báo lỗi serialization phải retry; **Serializable** — mạnh nhất, dùng SSI mô phỏng như các transaction chạy tuần tự, chống mọi anomaly KỂ CẢ write-skew, đổi lại đắt hiệu năng và dễ gặp lỗi `could not serialize access` phải retry. Postgres không có Read Uncommitted thật sự — yêu cầu nó sẽ hoạt động như Read Committed. Quy tắc: dùng level thấp nhất mà vẫn đúng; chỉ nâng lên Serializable khi có write-skew/anomaly thật sự, và luôn có cơ chế retry khi gặp lỗi serialization.

```ts
await prisma.$transaction(
  async (tx) => { /* ... */ },
  { isolationLevel: Prisma.TransactionIsolationLevel.Serializable },
);
```

Ví dụ thực tế: Phần lớn API Jira Clone chạy Read Committed là đủ; chỉ các thao tác kế toán/đếm nhạy cảm mới cần nâng isolation, và nhiều trường hợp dùng atomic increment hoặc unique constraint còn đơn giản và rẻ hơn nâng isolation.
</details>

<details>
<summary><b>19.6 Index là gì? Phân biệt B-tree, composite, unique và partial index?</b></summary>

Index là cấu trúc dữ liệu phụ giúp DB tìm row nhanh thay vì quét toàn bảng (sequential scan). **B-tree** (mặc định Postgres) tối ưu cho `=`, range `<,>,BETWEEN`, và hỗ trợ `ORDER BY` (tránh sort). **Composite index** trên nhiều cột `(a, b)` — tuân theo **left-prefix rule**: dùng được cho filter trên `a` hoặc `a` kết hợp `b`, nhưng không dùng hiệu quả nếu chỉ lọc một mình `b`, nên thứ tự cột rất quan trọng. **Unique index** vừa tăng tốc vừa enforce tính duy nhất — trong Postgres, unique constraint được thực thi BẰNG một unique index bên dưới. **Partial index** chỉ index tập con row thỏa điều kiện `WHERE`, giúp index nhỏ gọn, ghi nhanh hơn và phục vụ tốt các truy vấn luôn kèm điều kiện đó. Đánh đổi: index tăng tốc đọc nhưng làm chậm write (insert/update/delete phải cập nhật index) và tốn dung lượng, nên chỉ tạo theo truy vấn thực tế.

```sql
-- Composite: loc theo project roi sort theo ngay tao
CREATE INDEX idx_issue_project_created ON issue (project_id, created_at DESC);
-- Partial: chi index issue chua xoa mem
CREATE INDEX idx_issue_active ON issue (project_id) WHERE deleted_at IS NULL;
```

Ví dụ thực tế: Jira Clone dùng unique index `(projectId, number)` cho issue key, composite `(projectId, status)` cho board. Lưu ý: partial index `WHERE deleted_at IS NULL` phải tạo bằng raw SQL (migration) vì Prisma schema chưa hỗ trợ partial index trực tiếp — `@@index` trong schema KHÔNG sinh ra điều kiện WHERE.
</details>

<details>
<summary><b>19.7 Vấn đề N+1 query là gì và xử lý trong Prisma như thế nào?</b></summary>

N+1 xảy ra khi bạn chạy 1 query lấy danh sách N bản ghi, rồi lặp N lần, mỗi lần chạy thêm 1 query lấy quan hệ → tổng 1+N query, rất chậm. Trong Prisma, giải pháp chính là dùng `include`/`select` cho quan hệ. Lưu ý kỹ thuật: Prisma KHÔNG luôn JOIN — mặc định nó dùng chiến lược nhiều query rồi gộp kết quả ở tầng ứng dụng (vd với `findMany + include`, nó chạy thêm một `findMany ... WHERE id IN (...)` cho quan hệ, tức là chỉ thêm MỘT query chứ không phải N). Từ Prisma 5.x có thể bật `relationLoadMode: 'join'` (driver adapter) để gộp thành một SQL JOIN duy nhất. Dù cách nào, điểm mấu chốt là tránh gọi DB trong vòng lặp. Bật log query (`log: ['query']`) để phát hiện N+1; hoặc batch thủ công bằng `findMany` với `where: { id: { in: [...] } }`.

```ts
// XAU: N+1 (1 + N query trong vong lap)
const issues = await prisma.issue.findMany();
for (const i of issues) {
  i.assignee = await prisma.user.findUnique({ where: { id: i.assigneeId } });
}

// TOT: Prisma gom thanh it query nho include/select
const issues = await prisma.issue.findMany({
  include: { assignee: true, labels: true },
});
```

Ví dụ thực tế: Khi load board Jira Clone (issue + assignee + labels), mình dùng `include`/`select` có chọn lọc thay vì lặp qua từng issue, tránh hàng chục query thừa khi sprint có nhiều task.
</details>

<details>
<summary><b>19.8 Cascade delete và soft delete khác nhau thế nào? Khi nào dùng soft delete?</b></summary>

**Cascade delete** (`onDelete: Cascade`): khi xóa bản ghi cha, DB tự động xóa các bản ghi con phụ thuộc — tiện khi dữ liệu con không còn ý nghĩa nếu cha mất (vd xóa project → xóa các issue con), nhưng mất dữ liệu vĩnh viễn. **Soft delete**: không xóa thật, chỉ đánh dấu cột `deletedAt`/`isDeleted`, và mọi query phải lọc `WHERE deletedAt IS NULL`. Soft delete giữ lịch sử/audit, cho phép khôi phục, an toàn cho dữ liệu quan trọng; đổi lại phải nhớ lọc ở mọi truy vấn và cần index phù hợp. Nên tập trung filter ở một chỗ (vd Prisma Client Extension thay vmodel gọi tắt hoặc repository chung) để tránh sót — lưu ý middleware `$use` cũ đã deprecated, Prisma khuyên dùng Client Extensions. Soft delete cũng làm unique constraint phức tạp (vd email đã "xóa" vẫn chiếm chỗ), cần xử lý riêng. Dùng soft delete khi dữ liệu cần audit/recovery; dùng cascade khi dữ liệu con thực sự không còn giá trị.

```prisma
model Issue {
  id        String   @id @default(cuid())
  projectId String
  project   Project  @relation(fields: [projectId], references: [id], onDelete: Cascade)
  deletedAt DateTime? // soft delete
  @@index([projectId])
}
// Partial index loc deletedAt IS NULL phai dat trong migration SQL,
// Prisma @@index khong ho tro dieu kien WHERE.
```

Ví dụ thực tế: Jira Clone soft-delete issue/comment để user khôi phục và giữ audit log, nhưng dùng cascade cho các bảng join thuần túy như `IssueLabel` vì chúng không có ý nghĩa nếu issue bị xóa.
</details>

<details>
<summary><b>19.9 Deadlock là gì, vì sao xảy ra và làm sao tránh trong ứng dụng NestJS/Prisma?</b></summary>

Deadlock xảy ra khi hai (hoặc nhiều) transaction giữ lock và chờ lock của nhau theo vòng tròn: T1 khóa row A rồi chờ row B, T2 khóa row B rồi chờ row A — không ai đi tiếp được. Postgres tự phát hiện deadlock và chủ động hủy một transaction với lỗi `deadlock detected` (SQLSTATE 40P01); transaction còn lại chạy tiếp. Nguyên nhân phổ biến: cập nhật nhiều row theo THỨ TỰ KHÁC NHAU giữa các transaction, giữ transaction mở quá lâu (vd làm I/O chậm trong interactive transaction), hoặc escalation từ nhiều lock lẻ.

Cách tránh/giảm: (1) luôn khóa/cập nhật các row theo MỘT thứ tự nhất quán (vd sort theo id trước khi update hàng loạt); (2) giữ transaction NGẮN, không gọi API ngoài/gửi mail bên trong; (3) đặt `timeout`/`lock_timeout` hợp lý; (4) có cơ chế RETRY với backoff khi gặp lỗi 40P01 hoặc lỗi serialization 40001 — vì đây là lỗi tạm thời, retry thường thành công.

```ts
async function withRetry<T>(fn: () => Promise<T>, tries = 3): Promise<T> {
  for (let i = 0; ; i++) {
    try { return await fn(); }
    catch (e: any) {
      const code = e?.code; // P2034 (Prisma write conflict/deadlock) hoac 40P01/40001
      if (i < tries - 1 && (code === 'P2034' || code === '40P01' || code === '40001')) continue;
      throw e;
    }
  }
}
```

Ví dụ thực tế: Ở Jira Clone, khi reorder nhiều issue trong một sprint cùng lúc, mình sắp xếp các update theo id cố định và bọc trong `withRetry` để xử lý Prisma `P2034` thay vì trả lời 500 cho client.
</details>

---

## 20. Advanced Node.js

<details>
<summary><b>20.1 Giải thích các phase của event loop (timers, poll, check) và sự khác biệt giữa microtask và macrotask?</b></summary>

Event loop chạy theo các phase tuần tự trong mỗi vòng lặp (tick): **timers** (callback của `setTimeout`/`setInterval` đã hết hạn), **pending callbacks**, **idle/prepare** (nội bộ), **poll** (chờ và thực thi I/O callback), **check** (`setImmediate`), và **close callbacks**. Macrotask là các callback nằm trong các phase này.

Microtask thì KHÔNG thuộc phase nào. Thực tế có HAI hàng đợi ưu tiên cao hơn: hàng đợi `process.nextTick` (ưu tiên cao nhất) và hàng đợi microtask của Promise (`.then`, `queueMicrotask`). Sau MỖI callback macrotask hoàn tất, Node drain HẾT `nextTick` queue trước, rồi drain HẾT microtask queue, RỒI mới chuyển sang callback/phase tiếp theo. Vì vậy microtask luôn chạy trước macrotask kế tiếp; nếu sinh microtask (hoặc nextTick) vô hạn sẽ làm đói (starve) event loop.

```js
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
Promise.resolve().then(() => console.log('promise')); // microtask: chay truoc ca hai
// Trong main module, ket qua thuc te (Node v25): promise -> timeout -> immediate
// LUU Y: thu tu giua 'timeout' va 'immediate' o top-level la KHONG xac dinh
// (phu thuoc thoi diem vao loop). Chi khi goi tu BEN TRONG mot I/O callback
// thi 'immediate' moi luon chay truoc 'timeout'.
```

Ví dụ thực tế: hiểu rằng `await prisma.$transaction(...)` khi resolve sẽ tiếp tục qua microtask, nên đoạn code đặt NGAY sau `await` chạy sớm hơn callback macrotask mới (vd `setTimeout` của một retry) — lưu ý điều này chỉ đúng với các task chưa được xếp hàng trước đó, không phải lời hứa "luôn chạy trước mọi setTimeout".
</details>

<details>
<summary><b>20.2 process.nextTick starvation là gì và tại sao nó nguy hiểm hơn setImmediate?</b></summary>

`process.nextTick` có hàng đợi RIÊNG, được drain TRƯỚC cả microtask queue của Promise và trước khi event loop sang phase tiếp theo (nó không phải một microtask Promise, nhưng ưu tiên còn cao hơn). Nếu bạn đệ quy gọi `process.nextTick` liên tục, hàng đợi này không bao giờ rỗng nên event loop bị "đói" (starvation): I/O, timer và cả microtask Promise đều không được xử lý — server treo, không nhận request mới.

`setImmediate` an toàn hơn vì callback của nó chạy ở phase **check** mỗi vòng lặp; giữa các vòng lặp, phase **poll** vẫn có cơ hội xử lý I/O, nên việc lặp không chặn I/O hoàn toàn.

```js
// Starvation: server treo
function loop() { process.nextTick(loop); }
loop();

// An toan hon: van lap nhung KHONG chan I/O
function safeLoop() { setImmediate(safeLoop); }
safeLoop();
```

Nên dùng `process.nextTick` cho việc hoãn callback rất ngắn (vd đảm bảo một API luôn hành xử async, hoặc phát lỗi sau khi return), KHÔNG dùng cho công việc lặp/tính toán lớn — việc lớn nên chia nhỏ qua `setImmediate` hoặc đẩy xuống worker_threads.
</details>

<details>
<summary><b>20.3 Backpressure trong Streams là gì và cách xử lý đúng?</b></summary>

Backpressure xảy ra khi bên ghi (writable) tiêu thụ chậm hơn tốc độ bên đọc (readable) đẩy dữ liệu vào: dữ liệu dồn lại trong buffer nội bộ, tốn RAM và có thể OOM. Cơ chế đúng là dựa vào giá trị trả về của `writable.write()` — khi trả về `false` nghĩa là buffer đã vượt `highWaterMark`, bạn phải NGỪNG ghi thêm (vd `readable.pause()` nếu đọc thủ công) và đợi sự kiện `'drain'` của writable rồi mới tiếp tục.

Trong thực tế, hầu hết không nên tự xử lý: `pipe()` và `pipeline()` tự động điều phối backpressure. Ưu tiên `pipeline()` vì nó còn tự động dọn dẹp (destroy) tất cả stream và báo lỗi qua callback/Promise khi có stream nào lỗi — điều mà `pipe()` KHÔNG làm (dễ rò rỉ file descriptor khi lỗi).

```js
const { pipeline } = require('node:stream/promises');
const fs = require('node:fs');

await pipeline(
  fs.createReadStream('big.csv'),
  parseTransform,                 // mot Transform stream
  fs.createWriteStream('out.json'),
); // tu dong backpressure + cleanup khi error
```

Ví dụ thực tế: khi export hàng loạt issue ra CSV, tôi stream kết quả truy vấn (vd Prisma cursor/batch) qua một Transform rồi `pipeline` thẳng xuống HTTP response, tránh load toàn bộ vào bộ nhớ.
</details>

<details>
<summary><b>20.4 So sánh worker_threads, cluster và child_process — khi nào dùng cái nào cho tác vụ CPU-bound?</b></summary>

**cluster**: fork nhiều process Node giống nhau, chia sẻ cổng listen — dùng để scale tác vụ I/O-bound theo số CPU core; mỗi process có V8 instance và heap riêng (không chia sẻ memory). **child_process**: tạo tiến trình con (`spawn`/`exec`/`fork`), có thể chạy chương trình khác (vd `ffmpeg`), giao tiếp qua stdio/IPC — phù hợp chạy công cụ bên ngoài hoặc cách ly process. **worker_threads**: tạo THREAD trong CÙNG process, khởi tạo nhẹ hơn và rẻ hơn fork process, có thể chia sẻ dữ liệu nhị phân qua `SharedArrayBuffer` — là lựa chọn tốt nhất cho CPU-bound thuần JS (hash, nén, tính toán nặng) vì tránh chặn main event loop.

Lưu ý: với CPU-bound nặng, worker_threads thường tốt hơn cluster vì nhẹ hơn và truyền dữ liệu hiệu quả hơn (transferable/SharedArrayBuffer thay vì serialize qua IPC). Cluster phù hợp hơn để scale throughput của server I/O-bound.

```js
const { Worker } = require('node:worker_threads');
const w = new Worker('./heavy-hash.js', { workerData: payload });
w.once('message', result => res.json(result)); // main loop van phuc vu request khac
w.once('error', err => res.status(500).json({ error: err.message }));
```

Ví dụ thực tế: tính báo cáo burndown nặng hay xử lý ảnh đính kèm, tôi đẩy xuống worker_threads (qua một pool) để các API JWT/RBAC vẫn phản hồi mượt.
</details>

<details>
<summary><b>20.5 Các nguyên nhân rò rỉ bộ nhớ thường gặp trong Node (closure, event listener) và cách phát hiện?</b></summary>

Rò rỉ phổ biến: **event listener** đăng ký lặp lại mà không `removeListener`/`off` (thường kèm cảnh báo `MaxListenersExceededWarning`), **closure** vô tình giữ tham chiếu tới object lớn khiến nó sống mãi, và **cache/`Map` toàn cục** chỉ thêm mà không bao giờ xóa. Dấu hiệu: RSS/heap tăng dần và không giảm ngay cả sau khi GC chạy (`--expose-gc` để ép `global.gc()` khi test).

Để phát hiện: lấy nhiều heap snapshot trong Chrome DevTools (chạy với `--inspect`) rồi so sánh "retained size" / đếm object giữa các snapshot; hoặc log `process.memoryUsage()` theo thời gian; công cụ như `clinic doctor` hoặc `clinic heapprofiler` cũng hữu ích.

```js
const v8 = require('node:v8');
// Ghi mot heap snapshot ra file; mo trong tab Memory cua Chrome DevTools
const file = `heap-${Date.now()}.heapsnapshot`;
v8.writeHeapSnapshot(file);
console.log('wrote', file);
// Lap lai sau mot khoang thoi gian/tai, roi so sanh 2 file de tim object khong duoc giai phong
```

Thay cho cache vô hạn nên dùng LRU có giới hạn (vd `lru-cache`), và luôn hủy listener/timer trong `onModuleDestroy` của NestJS.
</details>

<details>
<summary><b>20.6 Làm sao detect event loop bị block trong production?</b></summary>

Event loop bị block khiến mọi request bị treo trong khi CPU có thể không cao theo kiểu deadlock. Cách đo tin cậy là **event loop delay/lag**: dùng `perf_hooks.monitorEventLoopDelay()` (histogram chính xác, đo bằng nanosecond) hoặc đo độ lệch giữa thời điểm `setInterval` dự kiến và thực tế. Khi lag vượt ngưỡng (vd p99 > 100ms) thường là dấu hiệu có code đồng bộ nặng: `JSON.parse`/`JSON.stringify` payload khổng lồ, vòng lặp lớn, hoặc crypto sync. Kết hợp `--prof` / `clinic flame` (flamegraph) để tìm hàm gây nghẽn.

```js
const { monitorEventLoopDelay } = require('node:perf_hooks');
const h = monitorEventLoopDelay({ resolution: 20 });
h.enable();
setInterval(() => {
  console.log('p99 lag(ms):', (h.percentile(99) / 1e6).toFixed(2));
  h.reset(); // reset de moi khoang phan anh trang thai gan day
}, 5000).unref();
```

Ví dụ thực tế: gắn metric event loop delay vào structured logging và cảnh báo (Sentry/Prometheus) khi lag cao — đây thường là triệu chứng sớm trước khi latency tăng và 5xx bùng phát.
</details>

<details>
<summary><b>20.7 Buffer là gì, khác String thế nào, và lưu ý bảo mật khi cấp phát?</b></summary>

`Buffer` là vùng nhớ nhị phân thuần, cấp phát NGOÀI V8 heap (off-heap), dùng để xử lý dữ liệu raw (file, socket, crypto) — vùng này không được tính trong giới hạn `--max-old-space-size`. String trong JS là UTF-16, immutable, không biểu diễn byte tùy ý một cách an toàn.

Lưu ý bảo mật: ưu tiên `Buffer.alloc(size)` (đã zero-fill) thay vì `Buffer.allocUnsafe(size)` — cái sau nhanh hơn nhưng trả về vùng nhớ CHƯA xóa, có thể chứa dữ liệu cũ nhạy cảm từ vùng heap đã giải phóng. Khi so sánh secret nên dùng `crypto.timingSafeEqual` để chống timing attack; vì hàm này ném lỗi nếu hai buffer khác độ dài, cần kiểm tra độ dài trước (lưu ý: chính việc kiểm tra độ dài cũng lộ rõ độ dài, nhưng không tránh khỏi).

```js
const crypto = require('node:crypto');

// 'utf8' la mac dinh; ghi ro encoding cho tuong minh
const a = Buffer.from(tokenFromCookie, 'utf8');
const b = Buffer.from(storedToken, 'utf8');
const ok = a.length === b.length && crypto.timingSafeEqual(a, b);
```

Ví dụ thực tế: khi so sánh refresh token (rotation), dùng `timingSafeEqual` thay vì `===` để chống timing attack. Tốt hơn nữa: chỉ lưu HASH của token và so sánh hash, tránh giữ token thô trong DB.
</details>

<details>
<summary><b>20.8 Làm thế nào để scale một ứng dụng Node trên 1 máy (cluster/PM2) và lưu ý khi có state?</b></summary>

Code JS của Node chạy trên 1 thread chính, nên một process chỉ tận dụng 1 core cho phần logic JS. Để dùng hết core, chạy nhiều instance bằng **cluster** module hoặc **PM2** ở chế độ cluster (`pm2 start app.js -i max`); kernel/PM2 sẽ phân phối kết nối giữa các worker.

Quan trọng: các instance KHÔNG chia sẻ memory, nên mọi state phải đẩy ra ngoài process — session/refresh token, rate-limit counter, cache đều nên lưu ở Redis. Việc lập lịch (`@Cron`) phải đảm bảo chỉ chạy trên 1 instance (leader election, distributed lock qua Redis, hoặc tách thành service worker riêng) để tránh chạy trùng. Với WebSocket/Socket.IO cần thêm adapter (vd Redis adapter) để broadcast giữa các worker.

```bash
# PM2: 1 process moi CPU core, tu restart khi crash
pm2 start dist/main.js -i max --name jira-api
```

Ví dụ thực tế: khi scale, tôi chuyển storage của `@nestjs/throttler` sang Redis (vd `ThrottlerStorageRedisService`) và cho cron retention/log chỉ chạy ở 1 worker — nếu không, rate limit sẽ sai (mỗi worker đếm riêng) và cron sẽ chạy trùng nhiều lần.
</details>

<details>
<summary><b>20.9 Cơ chế xử lý lỗi (error handling) trong Node async khác gì giữa callback, Promise và EventEmitter?</b></summary>

Đây là điểm hay sai trong production. Với **callback** kiểu Node, lỗi đi qua tham số đầu tiên (`(err, data) => ...`); quên kiểm tra `err` là mất lỗi âm thầm. Với **Promise/async-await**, lỗi lan truyền qua rejection — phải `try/catch` quanh `await` hoặc `.catch()`; một `await` không được bao bọc sẽ thành `unhandledRejection`. Với **EventEmitter**, lỗi phát qua sự kiện `'error'`: nếu KHÔNG có listener cho `'error'`, Node sẽ NÉM và làm crash process — đây là khác biệt then chốt so với Promise.

Ngoài ra, lỗi ném bên trong một callback async (vd trong `setTimeout`) KHÔNG thể bắt bởi `try/catch` ở ngoài vì ngữ cảnh đồng bộ đã kết thúc.

```js
// SAI: try/catch khong bat duoc loi nem trong callback async
try {
  setTimeout(() => { throw new Error('boom'); }, 0);
} catch (e) { /* khong bao gio chay -> process crash */ }

// Stream/EventEmitter: PHAI lang nghe 'error', neu khong se crash
stream.on('error', (err) => logger.error(err));
```

Nên cài global guard cho `process.on('unhandledRejection')` và `'uncaughtException')` để LOG rồi thoát có kiểm soát (let process manager restart), không nên "nuốt" lỗi và chạy tiếp với state không xác định.
</details>

<details>
<summary><b>20.10 AsyncLocalStorage là gì và vì sao hữu ích cho request context / tracing trong NestJS?</b></summary>

`AsyncLocalStorage` (module `node:async_hooks`) cho phép lưu một kho dữ liệu gắn với MỘT chuỗi thực thi async (vd vòng đời của 1 request) mà không phải truyền tham số xuyên suốt mọi hàm. Dữ liệu được bảo toàn qua các ranh giới async (`await`, callback, Promise) nhờ `async_hooks` theo dõi context — mỗi request có "store" riêng, không lẫn sang request khác.

Ứng dụng điển hình: gắn `requestId`/`traceId`, thông tin user, hoặc tenant vào store rồi logger đọc ra để mỗi dòng log đều có context, phục vụ tracing/correlation — thay cho việc nhồi `req` vào mọi service.

```js
const { AsyncLocalStorage } = require('node:async_hooks');
const als = new AsyncLocalStorage();

// Middleware: mo mot context cho moi request
app.use((req, res, next) => {
  als.run(new Map([['requestId', crypto.randomUUID()]]), () => next());
});

// O bat ky service nao sau do, khong can truyen req:
function log(msg) {
  const id = als.getStore()?.get('requestId');
  console.log(`[${id}] ${msg}`);
}
```

Lưu ý hiệu năng: có chi phí nhỏ do async_hooks; trong NestJS thường dùng qua `nestjs-cls`. Tránh dùng như một "biến toàn cục" thay thế dependency injection — chỉ nên dùng cho cross-cutting context (logging, tracing, tenant).
</details>

---

## 21. TypeScript for Backend

<details>
<summary><b>21.1 Generics là gì và khi nào bạn dùng chúng trong code backend?</b></summary>

Generics cho phép viết hàm/class/type hoạt động trên nhiều kiểu dữ liệu mà vẫn giữ type safety, thay vì dùng `any`. Compiler suy luận (infer) kiểu cụ thể tại chỗ gọi, nên bạn giữ được autocomplete và kiểm tra lỗi lúc compile. Trong backend tôi dùng generics cho các lớp wrapper tái sử dụng như repository, response DTO, hay hàm tiện ích. Điểm mạnh là ràng buộc generic bằng `extends` để giới hạn kiểu đầu vào.

```ts
// Wrapper phan hoi API dung chung cho moi kieu data
interface ApiResponse<T> {
  data: T;
  meta: { total: number };
}

// rang buoc T phai co id; tra ve ApiResponse boc mang T
function paginate<T extends { id: string }>(items: T[]): ApiResponse<T[]> {
  return { data: items, meta: { total: items.length } };
}
```

Ví dụ thực tế (Jira Clone): hàm cursor pagination của tôi có dạng `paginate<T>(items: T[], cursorField: keyof T)` dùng chung cho Issue, Project, Comment mà không cần viết lại logic cho từng entity. Dùng `keyof T` đảm bảo cursorField luôn là field hợp lệ của T.
</details>

<details>
<summary><b>21.2 Phân biệt các utility type Partial, Pick, Omit, Record, Required, Readonly?</b></summary>

Đây là các utility type có sẵn (phần lớn là mapped type): `Partial<T>` biến mọi field thành optional (hữu ích cho payload update); `Required<T>` làm ngược lại, bỏ `?`. `Pick<T, K>` lấy một tập con field, `Omit<T, K>` loại bỏ field (vd bỏ `password` khỏi User). `Record<K, V>` tạo object type với key thuộc K và value kiểu V. `Readonly<T>` khóa mọi field không cho gán lại (immutability lúc compile, không phải lúc runtime). Chúng giúp tạo DTO/type phái sinh từ một nguồn sự thật duy nhất, tránh trùng lặp định nghĩa.

```ts
interface User { id: string; email: string; password: string; role: string; }

type PublicUser = Omit<User, 'password'>;           // bo password
type UpdateUserDto = Partial<Pick<User, 'email' | 'role'>>;
type RoleMap = Record<'OWNER' | 'MEMBER', string[]>; // role -> danh sach permission
```

Ví dụ thực tế (Jira Clone): khi trả User ra API tôi dùng `Omit<User, 'password' | 'refreshTokenHash'>`, còn `UpdateIssueDto` thường là `Partial` của các field cho phép sửa, đảm bảo kiểu luôn đồng bộ với entity gốc. Lưu ý: các utility type chỉ ràng buộc kiểu, không tự động loại field khỏi dữ liệu runtime - việc đó vẫn phải làm bằng serializer (vd class-transformer).
</details>

<details>
<summary><b>21.3 type và interface khác nhau thế nào? Khi nào chọn cái nào?</b></summary>

`interface` chủ yếu mô tả hình dạng object/class, mở rộng qua `extends` và hỗ trợ declaration merging (khai báo trùng tên sẽ gộp lại). `type` là alias tổng quát hơn: ngoài object còn biểu diễn được union, tuple, primitive, conditional/mapped type mà interface không làm được. Lưu ý cả hai đều có thể được class `implements` và đều có thể mở rộng (interface dùng `extends`, type dùng intersection `&`). Quy tắc thực dụng của tôi: dùng `interface` cho contract object ổn định; dùng `type` cho union, intersection, và kiểu suy diễn phức tạp. Với shape object thuần, hai cái gần như tương đương.

```ts
interface Repo { find(id: string): Promise<unknown>; } // contract de class implements

type IssueStatus = 'TODO' | 'IN_PROGRESS' | 'DONE'; // union - interface khong lam duoc
type WithTimestamps<T> = T & { createdAt: Date; updatedAt: Date };
```

Điểm cần lưu ý: interface có thể bị merge ngoài ý muốn (vừa tiện cho augmentation, vừa dễ bị sửa lén ngoài kiểm soát), còn type thì không merge được, nên tùy mục tiêu mà chọn: public API library cần cho augmentation thì ưu tiên interface; còn type nội bộ cần chắc chắn không bị mở rộng thì type an toàn hơn.
</details>

<details>
<summary><b>21.4 Discriminated union là gì và vì sao nó hữu ích khi narrowing?</b></summary>

Discriminated (tagged) union là union của nhiều object cùng chia một field literal làm 'tag' (ví dụ `type`/`kind`/`status`). Khi bạn switch/if theo field đó, TypeScript tự động narrow về đúng nhánh và biết chính xác các field còn lại của nhánh đó. Nó biến các trạng thái loại trừ lẫn nhau thành kiểu an toàn, và kết hợp với `never` cho phép kiểm tra exhaustiveness lúc compile.

```ts
type Result =
  | { status: 'success'; data: string }
  | { status: 'error'; message: string };

function handle(r: Result): string {
  switch (r.status) {
    case 'success': return r.data;     // narrow: biet co .data
    case 'error':   return r.message;  // narrow: biet co .message
    default: {
      const _exhaustive: never = r; // them nhanh moi vao Result se bao loi o day
      return _exhaustive;
    }
  }
}
```

Ví dụ thực tế (Jira Clone): tôi model kết quả webhook/event theo discriminated union (`{ type: 'ISSUE_CREATED'; issue: Issue }` vs `{ type: 'COMMENT_ADDED'; comment: Comment }`) để handler xử lý từng loại mà không cần ép kiểu thủ công.
</details>

<details>
<summary><b>21.5 Type guard và narrowing hoạt động ra sao? Cho ví dụ custom type guard.</b></summary>

Narrowing là quá trình TypeScript thu hẹp kiểu trong một nhánh code dựa trên kiểm tra runtime: `typeof`, `instanceof`, `in`, hoặc so sánh literal. Khi các kiểm tra có sẵn không đủ, ta viết custom type guard - một hàm trả về `param is Type` (type predicate); nếu hàm trả `true`, compiler coi tham số là kiểu đó trong nhánh sau. Điều này quan trọng khi xử lý `unknown` từ input ngoài (request body, JSON parse) để chuyển an toàn sang kiểu đã biết.

```ts
interface AuthUser { id: string; role: string; }

function isAuthUser(x: unknown): x is AuthUser {
  return (
    typeof x === 'object' && x !== null &&
    'id' in x && typeof (x as Record<string, unknown>).id === 'string' &&
    'role' in x && typeof (x as Record<string, unknown>).role === 'string'
  );
}

function handler(req: unknown) {
  if (isAuthUser(req)) {
    console.log(req.role); // da narrow thanh AuthUser
  }
}
```

Lưu ý: type predicate là lời hứa với compiler - nếu logic bên trong sai, bạn tạo lỗ hổng an toàn kiểu, nên guard phải kiểm tra đầy đủ (kiểm tra cả sự tồn tại và kiểu của từng field, không chỉ `'id' in x`). Với trường hợp phức tạp nên dùng Zod/class-validator để vừa narrow vừa validate.
</details>

<details>
<summary><b>21.6 Decorators + reflect-metadata liên quan thế nào đến DI của NestJS?</b></summary>

Decorator là hàm gắn vào class/method/accessor/property/parameter để thêm metadata hoặc sửa hành vi, cần bật `experimentalDecorators` và `emitDecoratorMetadata` (với decorator legacy mà NestJS dùng). Với `emitDecoratorMetadata`, compiler phát ra metadata kiểu (vd `design:paramtypes`) qua thư viện `reflect-metadata`. NestJS đọc metadata này: khi bạn `@Injectable()` một service và khai báo constructor có tham số kiểu `UserService`, container đọc `design:paramtypes` của class để biết cần resolve dependency nào - đây chính là cơ chế DI dựa trên type. `@Inject(TOKEN)` dùng khi cần token tùy chỉnh thay vì suy từ type (vd interface, primitive, hoặc provider dùng string/symbol token).

```ts
import 'reflect-metadata';

class UserService {}

@Injectable() // class decorator - kich emit design:paramtypes cho constructor
class IssuesController {
  constructor(private readonly users: UserService) {}
}

// Doc lai metadata kieu cua constructor params:
const paramTypes = Reflect.getMetadata('design:paramtypes', IssuesController);
console.log(paramTypes); // [ [Function: UserService] ]
```

Lưu ý kỹ thuật: với PARAMETER decorator của constructor, chữ ký là `(target, propertyKey, parameterIndex)` và `propertyKey` là `undefined` (không phải string) - đây là điểm hay nhầm. Param decorator KHÔNG tự inject giá trị; nó chỉ lưu metadata (vd index cần resolve), còn việc resolve do container/guard làm.

Ví dụ thực tế (Jira Clone): tôi viết custom param decorator `@CurrentUser()` qua `createParamDecorator` và `@Public()` qua `SetMetadata` (global guard đọc lại bằng `Reflector`) - dùng đúng cơ chế metadata này.
</details>

<details>
<summary><b>21.7 satisfies và as const dùng để làm gì? Khác với type annotation thường?</b></summary>

`as const` làm literal thành readonly và narrow về kiểu hẹp nhất (vd `'TODO'` thay vì `string`), biến object/array thành readonly ở mức kiểu, rất hợp cho config/hằng số. `satisfies` (TS 4.9+) kiểm tra một expression có thỏa mãn một kiểu hay không nhưng KHÔNG làm rộng kiểu suy diễn về kiểu đó - khác với annotation `: T` vốn ép biến về đúng `T` và có thể làm mất thông tin literal/key cụ thể. Nhờ đó bạn vừa được kiểm tra ràng buộc, vừa giữ được kiểu cụ thể nhất để autocomplete chính xác.

```ts
type Config = Record<string, { port: number }>;

// annotation: kieu cua a la Config, key chi con la string chung
const a: Config = { db: { port: 5432 } };
// a.api -> khong bao loi luc truy cap (vi Record<string, ...>)

// satisfies: van check thoa man Config, nhung giu key 'db' cu the
const b = { db: { port: 5432 } } satisfies Config;
b.db.port; // OK, va autocomplete chi biet key 'db'

const ROLES = ['OWNER', 'MEMBER'] as const; // readonly ['OWNER', 'MEMBER']
```

Ví dụ thực tế (Jira Clone): bảng permission của RBAC tôi khai báo `... satisfies Record<Role, Permission[]>` để vừa bắt buộc mọi role đều có entry, vừa giữ key literal phục vụ narrowing khi check quyền.
</details>

<details>
<summary><b>21.8 Phân biệt unknown và any. Vì sao nên ưu tiên unknown?</b></summary>

`any` tắt hoàn toàn kiểm tra kiểu - gán vào biến `any` thì truy cập property/gọi hàm gì cũng qua, và `any` lan ra mọi nơi (vd gán `any` cho biến khác) làm mất an toàn. `unknown` là 'top type' an toàn (mọi kiểu đều gán được vào `unknown`): nó nhận mọi giá trị nhưng KHÔNG cho bạn thao tác gì cho đến khi narrow (qua type guard, `typeof`, `in`, hoặc ép kiểu có kiểm soát). Với dữ liệu từ bên ngoài (request body, response API, `JSON.parse`) nên dùng `unknown` rồi validate, vì nó buộc bạn xử lý trường hợp sai kiểu thay vì âm thầm bỏ qua.

```ts
function parse(json: string): unknown {
  return JSON.parse(json); // JSON.parse von tra ve any, ep kieu ve unknown an toan hon
}

const data = parse('{}');
// data.id;            // loi compile - phai narrow truoc
if (typeof data === 'object' && data !== null && 'id' in data) {
  console.log((data as { id: unknown }).id); // OK sau khi narrow
}
```

Ví dụ thực tế (Jira Clone): payload request luôn đi qua DTO + `ValidationPipe` (class-validator); các chỗ nhận dữ liệu thô (vd webhook) khai báo `unknown` rồi chạy type guard/validate - tránh tin tưởng mù quáng vào dữ liệu ngoài.
</details>

<details>
<summary><b>21.9 enum và union literal: nên dùng loại nào trong backend?</b></summary>

`enum` là cấu trúc có mặt lúc runtime: TypeScript phát ra một object thật sự (numeric enum còn có reverse mapping; string enum thì không), tốn thêm code và có vài điểm bất ngờ (vd numeric enum cho phép gán số tùy ý khi không khai báo chặt). Union literal type (vd `'TODO' | 'DONE'`) chỉ tồn tại lúc compile, zero runtime cost, narrow tốt và dễ đồng bộ với dữ liệu từ DB. Xu hướng hiện nay nghiêng về union literal hoặc `as const` object cho gọn và an toàn; chỉ dùng `enum` khi lý do rõ ràng. `const enum` thì inline lúc compile nhưng có hạn chế với `isolatedModules`/bundler nên thường tránh.

```ts
// Union literal - khong sinh runtime code
type Status = 'TODO' | 'IN_PROGRESS' | 'DONE';

// Pattern thay enum khi can ca value lan type
const Role = { OWNER: 'OWNER', MEMBER: 'MEMBER' } as const;
type Role = typeof Role[keyof typeof Role]; // 'OWNER' | 'MEMBER' (const va type cung ten, khac namespace nen hop le)
```

Ví dụ thực tế (Jira Clone): Prisma sinh kiểu cho cột status/role và ở các phiên bản gần đây xu hướng là const object/union literal (thay vì TS `enum`); tôi dùng trực tiếp kiểu Prisma sinh ra ở tầng DB, còn ở tầng DTO/logic nhẹ tôi thiên về union literal để narrowing và serialize JSON gọn. (Lưu ý: nên kiểm tra phiên bản Prisma cụ thể đang dùng - chứ không giả định một phiên bản tương lai.)
</details>

---
## 22. Advanced Testing

<details>
<summary><b>22.1 Test pyramid là gì? Tại sao nên có nhiều unit test hơn e2e test?</b></summary>

Test pyramid mô tả tỷ lệ lý tưởng giữa các loại test: nhiều **unit test** ở đáy (nhanh, cách ly, rẻ), ít hơn là **integration test** ở giữa, và rất ít **e2e test** ở đỉnh (chậm, dễ vỡ/flaky, đắt). Lý do: unit test cho feedback gần như tức thì và chỉ ra chính xác nơi lỗi; còn e2e test chạy qua nhiều lớp (HTTP, DB, network) nên chậm và dễ flaky. Anti-pattern thường gặp là "ice cream cone" (quá nhiều e2e, ít unit) khiến CI chậm và khó debug.

Ví dụ thực tế: Trong Jira Clone, logic sinh issue key `PROJECT-N` tôi test kỹ bằng unit test (mock Prisma), còn luồng đăng nhập + set httpOnly cookie thì chỉ cần vài e2e test xác nhận end-to-end.
</details>

<details>
<summary><b>22.2 Phân biệt mock, spy, stub và fake. Khi nào dùng loại nào?</b></summary>

Bốn khái niệm khác nhau ở mục đích:
- **Stub**: trả về giá trị có sẵn cho input, chỉ để điều khiển luồng (state-based). Không quan tâm gọi mấy lần.
- **Spy**: ghi lại các lần gọi (arguments, số lần). Mặc định `jest.spyOn` vẫn gọi qua hàm thật; chỉ khi gắn `.mockImplementation`/`.mockResolvedValue` thì nó mới thay thế logic thật.
- **Mock**: đặt trước kỳ vọng (expectation) về tương tác và kiểm chứng sau đó; test **fail nếu tương tác không đúng** (behavior verification).
- **Fake**: bản cài đặt thật nhưng đơn giản (ví dụ in-memory repository thay cho DB thật).

```ts
// stub: thay the de ep tra ve (khong goi ham that)
jest.spyOn(repo, 'findUser').mockResolvedValue(user);

// spy: chi theo doi, VAN goi ham that vi khong gan mock implementation
const spy = jest.spyOn(logger, 'warn');
service.handle(dirtyInput);
expect(spy).toHaveBeenCalledWith('PII redacted');
```

Lưu ý: trong Jest, `spyOn` có kèm `mockReturnValue/mockResolvedValue` thực chất đang đóng vai stub; còn muốn thuần túy theo dõi mà giữ logic thật thì dùng `spyOn` không override.

Ví dụ thực tế: Test `EmailService` (Resend) tôi **stub** hàm gửi để không bắn email thật; còn kiểm tra audit log có được gọi đúng không thì dùng **spy** trên logger.
</details>

<details>
<summary><b>22.3 Làm sao test một Guard, Interceptor và Pipe trong NestJS?</b></summary>

Cách hiệu quả nhất là **unit test** chúng như class thường: tự tạo `ExecutionContext`/`ArgumentMetadata` giả thay vì quay cả HTTP. Với **Guard**, gọi trực tiếp `canActivate(context)` với context mock chứa `request.user` và assert kết quả. **Pipe** thì gọi `transform(value, metadata)` và assert kết quả/lỗi. **Interceptor** thì mock `CallHandler.handle()` trả về một observable rồi assert phần transform (vì `intercept` trả về Observable, nên dùng `firstValueFrom` để lấy giá trị).

Lưu ý RBAC: thông thường nếu không đủ quyền, **guard trả về `false`** và chính NestJS sẽ ném `ForbiddenException`. Guard chỉ tự ném exception khi bạn chủ động viết `throw` bên trong (ví dụ để gắn thông điệp lỗi cụ thể). Hãy test đúng hành vi mà guard của bạn thực sự làm.

```ts
// Truong hop guard tu throw khi thieu role
it('chan user khong phai project LEAD', () => {
  const ctx = {
    switchToHttp: () => ({ getRequest: () => ({ user: { role: 'DEVELOPER' } }) }),
    getHandler: () => ({}), getClass: () => ({}),
  } as unknown as ExecutionContext;
  jest.spyOn(reflector, 'getAllAndOverride').mockReturnValue(['LEAD']);
  expect(() => guard.canActivate(ctx)).toThrow(ForbiddenException);
});

// Truong hop guard chi tra ve false (Nest se tu nem ForbiddenException)
it('tra ve false khi thieu role', () => {
  jest.spyOn(reflector, 'getAllAndOverride').mockReturnValue(['LEAD']);
  expect(guard.canActivate(ctx)).toBe(false);
});
```

Ví dụ thực tế: RBAC nhiều tầng của Jira Clone (workspace OWNER/ADMIN, project LEAD...) tôi cover phần lớn bằng unit test guard với reflector mock (dùng `getAllAndOverride` để gộp metadata method + class), chỉ giữ vài e2e cho cả tầng kết hợp.
</details>

<details>
<summary><b>22.4 Làm sao cô lập DB giữa các e2e test? So sánh truncate, transaction rollback và in-memory.</b></summary>

Mục tiêu là mỗi test bắt đầu từ trạng thái sạch, không rò rỉ dữ liệu. Ba chiến lược:
- **Truncate** bảng sau mỗi test: đơn giản, dùng schema thật, nhưng chậm hơn và cần xử lý khóa ngoại + sequence. Dùng `TRUNCATE ... RESTART IDENTITY CASCADE` để vừa xóa dữ liệu liên quan vừa reset auto-increment về đầu.
- **Transaction rollback**: mở transaction trước test, rollback sau test — rất nhanh và sạch, nhưng **khó áp dụng khi code under test tự mở transaction** (Prisma `$transaction` lồng nhau) hoặc khi có nhiều connection khác nhau (request HTTP chạy trên connection khác với test).
- **In-memory (SQLite)**: nhanh nhưng lệch behavior so với Postgres thật (kiểu dữ liệu, constraint, transaction isolation), dễ "green local, red prod".

Thực tế tôi ưu tiên **Testcontainers Postgres** + truncate giữa các test để sát production nhất.

Ví dụ thực tế: Vì Jira Clone dùng `$transaction` để sinh issue key atomic, tôi tránh rollback-per-test (sẽ mâu thuẫn với transaction lồng nhau + chạy e2e qua HTTP trên connection khác) mà dùng Postgres thật trong container + truncate.
</details>

<details>
<summary><b>22.5 Code coverage đo cái gì và giới hạn của nó là gì? 100% coverage có nghĩa là không còn bug?</b></summary>

Coverage đo **phần trăm code được thực thi** trong khi test (line/branch/function/statement); nó chỉ cho biết code có được **chạy** chứ không đảm bảo được **kiểm chứng đúng**. Bạn có thể đạt 100% coverage mà không hề có assertion có ý nghĩa, hoặc vẫn bỏ sót edge case (null, concurrency, lỗi mạng) hay sai logic mà test không hề kiểm. Vì vậy coverage chỉ nên là **chỉ báo bổ sung**, không phải mục tiêu cuối cùng — nên tập trung **branch coverage** cho logic quan trọng và dùng **mutation testing** (ví dụ Stryker) để đánh giá chất lượng assertion (xem test có thực sự bắt được lỗi khi code bị đột biến không).

```ts
// 100% line coverage cho ham sum nhung vo nghia: khong he assert ket qua
it('cong so', () => {
  sum(2, 2); // dong code chay (duoc tinh coverage) nhung khong expect gi
});
```

Ví dụ thực tế: Với logic sinh `PROJECT-N` tôi quan tâm branch coverage cho các case xử lý trùng key/race condition hơn là chỉ số % tổng thể.
</details>

<details>
<summary><b>22.6 Integration test khác unit test và e2e test thế nào? Khi nào nên viết integration test?</b></summary>

**Unit test** kiểm tra một đơn vị cách ly (mock toàn bộ phụ thuộc). **Integration test** kiểm tra **nhiều thành phần thật làm việc cùng nhau** — ví dụ service thật + Prisma thật + DB thật — nhưng thường vẫn bỏ qua tầng HTTP/controller. **E2e test** chạy toàn bộ qua HTTP như người dùng thật. Integration test đặc biệt có giá trị khi rủi ro nằm ở **ranh giới** (ORM mapping, SQL sinh ra, transaction, constraint) — nơi mà mock sẽ che giấu lỗi.

Ví dụ thực tế: Tôi viết integration test cho repository sinh issue key, dùng DB thật để chắc chắn `$transaction` + unique constraint thực sự chặn được trùng key khi có 2 request song song — điều mà unit test mock Prisma không bao giờ bắt được (vì mock luôn trả về đúng giá trị ta đặt sẵn, không có constraint thật).
</details>

<details>
<summary><b>22.7 Flaky test là gì, nguyên nhân phổ biến và cách phòng tránh?</b></summary>

Flaky test là test lúc pass lúc fail mà code không đổi — nó làm mất niềm tin vào CI. Nguyên nhân phổ biến: phụ thuộc **thời gian/`sleep` cứng**, **thứ tự test** (state rò rỉ, dữ liệu dùng chung giữa các test), **bất đồng bộ** (race, promise không await), phụ thuộc **clock/random/timezone**, và gọi **mạng/dịch vụ ngoài** thật. Cách tránh: cách ly dữ liệu mỗi test, dùng fake timers thay vì sleep, mock thời gian và random, await hết promise, và không gọi service ngoài trong test (dùng stub).

```ts
jest.useFakeTimers();
jest.setSystemTime(new Date('2026-06-25'));
// thay vi: await sleep(1000)
jest.advanceTimersByTime(1000);
// nho khoi phuc sau test de khong anh huong test khac:
// jest.useRealTimers();
```

Ví dụ thực tế: Job `@Cron` retention của Jira Clone tôi test bằng fake timers + mock `Date`, không bao giờ chạy theo thời gian thực, để tránh flaky.
</details>

<details>
<summary><b>22.8 Làm thế nào test code bất đồng bộ đúng cách? AAA pattern là gì?</b></summary>

Với code async, luôn **await** Promise hoặc return nó cho test runner — quên await khiến assertion chạy sau khi test đã kết thúc (false positive, test xanh giả). Kiểm tra reject bằng `await expect(fn()).rejects.toThrow(...)`. Với observable (RxJS trong interceptor) dùng `firstValueFrom` hoặc marble testing. **AAA** = **Arrange** (chuẩn bị dữ liệu/mock), **Act** (gọi hàm cần test, lý tưởng là 1 hành động), **Assert** (kiểm tra kết quả) — giúp test rõ ràng, dễ đọc; mỗi test nên tập trung kiểm một hành vi.

```ts
it('nem khi refresh token het han', async () => {
  // Arrange
  jest.spyOn(repo, 'find').mockResolvedValue(expiredToken);
  // Act + Assert (phai await, neu khong test se pass truoc khi promise reject)
  await expect(auth.rotate(token)).rejects.toThrow(UnauthorizedException);
});
```

Ví dụ thực tế: Test refresh token rotation của Jira Clone đều là async — tôi luôn `await expect(...).rejects` để chắc chắn lỗi hết hạn được ném đúng, tránh test xanh giả.
</details>

<details>
<summary><b>22.9 Làm sao test Prisma trong NestJS: mock Prisma hay dùng DB thật? Công cụ nào hỗ trợ?</b></summary>

Có hai hướng, dùng tùy tầng test:
- **Mock/stub PrismaService** (unit test): cách ly hoàn toàn, nhanh, không cần DB. Dùng `jest-mock-extended` (`mockDeep<PrismaClient>()`) để mock sâu các method lồng nhau (`prisma.issue.create(...)`). Phù hợp khi test logic nghiệp vụ của service, không quan tâm SQL.
- **DB thật** (integration/e2e): dùng **Testcontainers** spin lên Postgres thật, chạy migration, rồi test repository/luồng thật. Bắt được lỗi mà mock che giấu: unique constraint, cascade delete, transaction isolation, kiểu dữ liệu.

Nguyên tắc: dùng mock khi bạn kiểm **logic**, dùng DB thật khi bạn kiểm **tương tác với DB** (constraint, transaction, SQL). Mock Prisma rồi assert "đã gọi create với đúng tham số" chỉ xác nhận service gọi đúng, KHÔNG xác nhận DB chấp nhận dữ liệu đó.

```ts
// Unit test voi jest-mock-extended
const prisma = mockDeep<PrismaService>();
prisma.issue.findUnique.mockResolvedValue(null);
// ... inject prisma vao service, assert hanh vi
```

Ví dụ thực tế: Logic phân quyền + tình trạng issue của Jira Clone tôi mock Prisma để test nhanh; còn ràng buộc unique `(projectId, key)` thì bắt buộc test trên Postgres thật vì mock không thể mô phỏng constraint.
</details>

<details>
<summary><b>22.10 Test data builder / factory là gì và tại sao quan trọng cho test đạt chất lượng?</b></summary>

Khi viết nhiều test, tạo dữ liệu thủ công (object dài dòng, lặp lại) khiến test khó đọc và dễ vỡ khi schema đổi. **Factory/Builder pattern** giúp tạo entity hợp lệ với giá trị mặc định, chỉ override trường liên quan đến test — làm nổi bật "điều gì đang được kiểm".

```ts
// builder: mac dinh hop le, override truong can quan tam
const makeUser = (overrides: Partial<User> = {}): User => ({
  id: 'u1', email: 'a@b.c', role: 'DEVELOPER', ...overrides,
});

it('chan DEVELOPER xoa project', () => {
  const user = makeUser({ role: 'DEVELOPER' }); // chi truong nay quan trong
  expect(() => service.deleteProject(user, 'p1')).toThrow(ForbiddenException);
});
```

Lợi ích: giảm trùng lặp, test chỉ thể hiện đúng biến số quan trọng (rõ intent), và khi schema đổi chỉ sửa một chỗ. Với DB thật có thể dùng thư viện như Fishery hoặc viết factory từ seed Prisma.

Ví dụ thực tế: Trong Jira Clone tôi có `makeProjectMember(role)` để sinh nhanh thành viên với từng vai trò, giúp viết hàng loạt test RBAC ngắn gọn và rõ ý định thay vì dùng dữ liệu thừa.
</details>

---
## 23. System Design and Scaling

<details>
<summary><b>23.1 Modular monolith vs microservices: khi nào nên tách service? Đánh đổi là gì?</b></summary>

Bắt đầu bằng **modular monolith** là lựa chọn mặc định đúng đắn: deploy đơn giản, transaction xuyên module dễ đảm bảo (một `$transaction`), không có network latency hay distributed tracing phức tạp. Chỉ tách **microservices** khi có lý do thực sự: một module cần scale độc lập (vd module xử lý file/email tải cao trong khi REST API thấp), team lớn cần deploy độc lập để giảm coupling tổ chức, hoặc yêu cầu công nghệ khác biệt. Đánh đổi khi tách: mất ACID transaction xuyên service (phải dùng saga/eventual consistency), tăng độ phức tạp vận hành (service discovery, distributed tracing, network failure), và overhead serialize/deserialize qua network.

Nguyên tắc: đặt ranh giới module rõ ràng trong monolith trước (theo bounded context, mỗi module sở hữu data riêng, giao tiếp qua interface chứ không query thẳng bảng của nhau), khi đó việc tách sau này chỉ là "kéo module ra" thay vì viết lại.

Ví dụ thực tế: Jira Clone hiện là modular monolith với các module Auth, Workspace, Project, Issue. Nếu sau này module gửi email/OTP (Resend) và xử lý background job trở thành bottleneck, ta có thể tách riêng thành notification service tiêu thụ từ message queue, còn core API vẫn giữ nguyên.
</details>

<details>
<summary><b>23.2 Scale ngang app stateless: vì sao không được lưu state in-memory? Ảnh hưởng gì đến thiết kế?</b></summary>

Khi scale ngang, load balancer phân phối request đến nhiều instance, và một user có thể trúng vào instance khác nhau giữa các request. Nếu lưu state **in-memory** (session, rate-limit counter, đếm số) thì mỗi instance có một bản sao khác nhau -> hành vi không nhất quán, mất data khi instance restart/scale-down. Vì vậy app phải **stateless**: mọi state chia sẻ phải đẩy ra ngoài (Redis cho session/cache/rate-limit, Postgres cho dữ liệu bền vững, S3 cho file).

Một hệ quả quan trọng: JWT trong httpOnly cookie giúp auth gần stateless — bất kỳ instance nào cũng verify được token bằng shared secret (HMAC) hoặc public key (RSA/EC) mà không cần gọi DB hay sticky session. Lưu ý: nếu cần revoke token trước hạn (logout, đổi mật khẩu) thì vẫn phải kiểm tra một blocklist trên Redis, nên "stateless hoàn toàn" chỉ đúng khi chấp nhận token sống đến khi hết hạn.

```ts
// SAI khi scale ngang: counter chi dung tren 1 instance
const loginAttempts = new Map<string, number>();

// DUNG: day state ra Redis, moi instance deu thay; set TTL atomic
const attempts = await redis.incr(`login:attempts:${userId}`);
if (attempts === 1) {
  await redis.expire(`login:attempts:${userId}`, 900);
}
```

Ví dụ thực tế: Trong Jira Clone, structured logging dùng buffer in-memory trước khi flush — nếu scale ngang, mỗi instance có buffer riêng (chấp nhận được vì log độc lập, và nên đẩy log ra collector tập trung). Ngược lại, rate limiting của @nestjs/throttler PHẢI dùng Redis storage để đếm chung trên toàn cluster, nếu không mỗi instance sẽ cho phép gấp N lần giới hạn.
</details>

<details>
<summary><b>23.3 Caching với Redis: giải thích cache-aside và các chiến lược invalidation?</b></summary>

**Cache-aside** (lazy loading) là pattern phổ biến nhất: app đọc cache trước, nếu **miss** thì query DB rồi ghi ngược vào cache với TTL, nếu **hit** thì trả về luôn. Ứng dụng chủ động quản lý cache, DB vẫn là source of truth. Về **invalidation** có các hướng: (1) TTL — đơn giản, chấp nhận stale trong khoảng ngắn; (2) explicit delete khi ghi (xóa key liên quan sau khi update DB để lần đọc sau refresh); (3) write-through — ghi DB và cache trong cùng một bước (cache luôn mới nhưng phức tạp hơn và có thể lãng phí nếu ít đọc lại); (4) event-based — phát sự kiện để invalidate cache phân tán. Quy tắc vàng: **invalidate (xóa) key thường an toàn hơn update cache trực tiếp** vì tránh race condition ghi giá trị cũ đè lên.

Lưu ý cache stampede: khi key hết hạn và nhiều request cùng miss -> dùng lock (single-flight) hoặc stale-while-revalidate.

```ts
async getProject(id: string) {
  const key = `project:${id}`;
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached); // hit
  const project = await prisma.project.findUnique({ where: { id } });
  // chi cache khi co du lieu, tranh cache null gay nham lan voi cache miss
  if (project) {
    await redis.set(key, JSON.stringify(project), 'EX', 300); // miss -> fill
  }
  return project;
}
// Khi update: xoa key thay vi ghi de
async updateProject(id: string, dto: UpdateProjectDto) {
  const p = await prisma.project.update({ where: { id }, data: dto });
  await redis.del(`project:${id}`);
  return p;
}
```

Lưu ý: trong cache-aside, thứ tự nên là update DB trước rồi del cache; và để giảm rủi ro stale do race, có thể del cache một lần nữa sau một khoảng ngắn (delayed double delete). Ví dụ thực tế: trong Jira Clone, dữ liệu đọc nhiều/ghi ít như cấu hình workspace hoặc danh sách project member rất hợp cache-aside; khi đổi role member thì del key `workspace:${id}:members`.
</details>

<details>
<summary><b>23.4 Message queue (Bull/RabbitMQ): khi nào dùng cho tác vụ nặng/async và lợi ích gì?</b></summary>

Dùng message queue khi công việc **tốn thời gian** hoặc **có thể làm bất đồng bộ** với vòng đời của HTTP request: gửi email, sinh export PDF/CSV, xử lý ảnh, gọi API bên thứ ba, gửi notification hàng loạt. Lợi ích: (1) trả response cho client ngay (low latency), worker xử lý nền; (2) **retry + backoff** tự động khi lỗi tạm thời; (3) **rate control / concurrency** để không quá tải dịch vụ downstream; (4) tách tải — có thể scale worker độc lập với API; (5) hấp thụ burst tải (buffer) khi lượng request tăng đột biến. **Bull/BullMQ** chạy trên Redis, hợp cho job queue trong hệ NestJS (có `@nestjs/bullmq`); **RabbitMQ** là message broker thực sự, mạnh về routing (exchange/binding), pub-sub và giao tiếp giữa nhiều service.

Lưu ý: queue thường là at-least-once delivery, nên consumer phải xử lý idempotent vì một job có thể chạy lại (retry hoặc giao trùng).

```ts
// Producer: day job, tra ve ngay
await this.emailQueue.add('send-otp', { userId, email }, {
  attempts: 3,
  backoff: { type: 'exponential', delay: 2000 },
  removeOnComplete: true,
});

// Consumer (BullMQ): worker xu ly nen, ten queue khop voi @Processor
@Processor('email')
export class EmailProcessor extends WorkerHost {
  async process(job: Job) {
    if (job.name === 'send-otp') {
      await this.resend.send(job.data.email /* ... */);
    }
  }
}
```

Ví dụ thực tế: Jira Clone gửi OTP qua Resend — nếu làm đồng bộ trong request login sẽ chậm và dễ fail khi Resend timeout. Đẩy vào Bull queue giúp login trả về ngay, worker retry nếu gửi thất bại. Lưu ý: với OTP gửi qua queue, cần chấp nhận độ trễ nhỏ trước khi user nhận được mã.
</details>

<details>
<summary><b>23.5 Multi-tenancy: các mô hình isolation dữ liệu? Anh đang áp dụng kiểu nào?</b></summary>

Có 3 mô hình chính: (1) **Shared database, shared schema** — mỗi row có `tenantId`/`workspaceId`, isolation bằng điều kiện WHERE; rẻ nhất, dễ scale, nhưng rủi ro lớn nhất là quên filter tenant -> lộ dữ liệu chéo. (2) **Shared database, schema-per-tenant** — mỗi tenant một schema; isolation tốt hơn, backup/migration có thể làm riêng, nhưng nhiều tenant thì quản lý migration phức tạp (phải chạy trên tất cả schema). (3) **Database-per-tenant** — isolation mạnh nhất, hợp khách enterprise yêu cầu compliance, nhưng tốn kém và khó vận hành ở quy mô lớn. SaaS thông thường chọn mô hình (1) vì cân-bằng chi phí và đơn giản.

Để an toàn với mô hình shared, nên bật **row-level security (RLS)** ở DB (Postgres hỗ trợ sẵn) hoặc một lớp guard/middleware bắt buộc mọi query phải có tenant scope — tốt nhất là cả hai, vì guard ở app dễ bị quên còn RLS là lớp chặn cuối ở DB.

```ts
// Luon scope theo workspace, KHONG bao gio query "tran"
const issues = await prisma.issue.findMany({
  where: { project: { workspaceId: ctx.workspaceId } },
});
```

Ví dụ thực tế: Jira Clone dùng shared schema với `workspaceId` làm ranh giới tenant; hệ thống RBAC nhiều tầng (workspace OWNER/ADMIN/MEMBER/VIEWER) xác định quyền thao tác, còn việc scope mọi query theo `workspaceId` mới là lớp chống rò rỉ dữ liệu chéo tenant. Hai thứ này khác nhau: RBAC kiểm soát "được làm gì", tenant scope kiểm soát "thấy được data của ai".
</details>

<details>
<summary><b>23.6 Idempotency: tại sao cần và triển khai idempotency key cho API thế nào?</b></summary>

Idempotency đảm bảo gọi một thao tác **nhiều lần cho kết quả giống như gọi một lần** — quan trọng với POST tạo tài nguyên, thanh toán, hay khi client retry do network timeout (server đã xử lý thành công nhưng client không nhận được response). Cách triển khai phổ biến: client gửi header `Idempotency-Key` (UUID); server lưu key + kết quả; nếu key đã tồn tại thì trả lại response đã lưu thay vì thực thi lại. GET/PUT/DELETE vốn idempotent về bản chất (theo định nghĩa HTTP, dù kết quả trả về có thể khác), POST thì không nên cần cơ chế này.

Lưu ý triển khai đúng: phải xử lý race khi hai request cùng key đến gần như đồng thời — dùng khóa duy nhất (unique constraint ở DB hoặc `SET NX` trên Redis) để chỉ một request thực thi, request còn lại chờ/trả kết quả. Với consumer của message queue (at-least-once), idempotency càng quan trọng vì cùng một message có thể được giao 2 lần.

```ts
async createWithIdempotency(key: string, dto: CreateDto) {
  // SET NX: chi giu cho duoc neu key chua ton tai
  const acquired = await redis.set(`idem:${key}`, 'PENDING', 'EX', 86400, 'NX');
  if (!acquired) {
    const existing = await redis.get(`idem:${key}`);
    // can poll/cho neu van la PENDING; o day don gian hoa
    return existing && existing !== 'PENDING' ? JSON.parse(existing) : null;
  }
  const result = await this.create(dto);
  await redis.set(`idem:${key}`, JSON.stringify(result), 'EX', 86400);
  return result;
}
```

Ví dụ thực tế: Trong Jira Clone, sinh issue key dạng `PROJECT-N` chạy trong `$transaction` với atomic increment (vd `UPDATE ... SET seq = seq + 1 RETURNING seq`) — đây đảm bảo N không bị trùng khi nhiều request tạo issue đồng thời (đây là concurrency control, không hoàn toàn giống idempotency: idempotency là lặp lại cùng request cho cùng kết quả). Với API tạo issue từ client, thêm Idempotency-Key để tránh tạo trùng khi user double-click.
</details>

<details>
<summary><b>23.7 Database read replica: dùng để scale đọc thế nào? Cần lưu ý gì về replication lag?</b></summary>

Khi tải **đọc** lớn hơn tải ghi nhiều lần (rất phổ biến), ta dùng **read replica**: primary nhận tất cả write, các replica đồng bộ từ primary và phục vụ read query. Cách làm: định tuyến write -> primary, read -> replica (qua Prisma read replicas extension, hoặc proxy/load balancer). Vấn đề cốt lõi là **replication lag** — replica có thể chậm hơn primary vài chục/trăm ms (hoặc hơn khi tải ghi cao), gây **read-your-write inconsistency**: user vừa tạo issue, redirect sang trang list nhưng đọc từ replica chưa kịp đồng bộ -> không thấy. Giải pháp: với thao tác vừa-ghi-rồi-đọc-ngay thì đọc từ primary (read-after-write), hoặc cho ứng dụng chấp nhận eventual consistency ở những chỗ không nhạy cảm.

Lưu ý: PgBouncer là connection pooler, không tự định tuyến read/write — dùng nó để quản lý connection chứ không phải để tách replica. Replica còn phục vụ mục đích phụ: chạy query phân tích/báo cáo nặng mà không ảnh hưởng primary, và làm standby cho HA failover.

```ts
import { readReplicas } from '@prisma/extension-read-replicas';

const prisma = new PrismaClient().$extends(
  readReplicas({ url: process.env.REPLICA_URL! })
);
await prisma.issue.findMany();          // -> replica
await prisma.$primary().issue.create({ data }); // ep ghi/doc tren primary
```

Ví dụ thực tế: Jira Clone có màn hình board/backlog đọc rất nhiều issue; có thể đẩy các truy vấn list/filter sang read replica, còn thao tác tạo/cập nhật issue và đọc lại ngay sau đó thì ép về primary để tránh lag.
</details>

<details>
<summary><b>23.8 Rate limiting phân tán: tại sao throttler in-memory hỏng khi scale, và fix thế nào?</b></summary>

@nestjs/throttler mặc định lưu counter **in-memory** trên từng instance. Khi scale ngang sau load balancer, giới hạn thực tế bị nhân lên theo số instance: đặt 100 req/phút nhưng có 4 instance và request rải đều thì tổng có thể lên ~400 req/phút, và counter mất khi instance restart. Để có rate limit đúng trên toàn cluster, phải dùng **shared store** — phổ biến nhất là **Redis** (vd `@nest-lab/throttler-storage-redis`), nhờ atomic `INCR` + `EXPIRE` nên mỗi instance đều cộng vào cùng một counter. Thuật toán thường dùng: **sliding window** hoặc **token bucket**, thường triển khai bằng Lua script để gộp nhiều lệnh Redis thành một bước atomic.

Ngoài ra nên rate limit theo nhiều khóa (per-IP, per-user, per-endpoint) và đặt ở lớp edge/API gateway nếu có để chặn sớm; lưu ý lấy đúng client IP sau proxy (X-Forwarded-For / `trust proxy`).

```ts
ThrottlerModule.forRootAsync({
  useFactory: () => ({
    throttlers: [{ limit: 100, ttl: 60000 }],
    storage: new ThrottlerStorageRedisService(redis), // shared counter
  }),
});
```

Ví dụ thực tế: Jira Clone dùng @nestjs/throttler — ở một instance thì ổn, nhưng khi deploy nhiều replica cần chuyển storage sang Redis, đặc biệt cho endpoint login/OTP để chống brute-force đúng trên toàn hệ thống chứ không riêng từng instance.
</details>

---

## 24. API and REST Design

<details>
<summary><b>24.1 Trình bày các REST conventions bạn tuân thủ khi đặt tên resource và thiết kế endpoint?</b></summary>

Resource nên là danh từ số nhiều (`/projects`, `/issues`), không dùng verb vì HTTP method đã thể hiện hành động. Quan hệ lồng nhau thể hiện qua nested route (`/projects/:id/issues`), nhưng không nên lồng quá 2 cấp — nếu sâu hơn thì tách ra resource độc lập và dùng query filter. Dùng kebab-case cho path, và đặt hành động đặc biệt (không map được với CRUD) dưới dạng sub-resource hoặc state transition.

```
GET    /projects/:id/issues        # list issues cua project
POST   /projects/:id/issues        # tao issue
GET    /issues/:id                 # chi tiet (resource doc lap, khong long sau)
PATCH  /issues/:id                 # cap nhat 1 phan
PUT    /issues/:id                 # thay the toan bo resource
POST   /issues/:id/transitions     # action: chuyen trang thai (khong phai CRUD)
```

Lưu ý: PATCH là cập nhật 1 phần (partial), PUT là thay thế toàn bộ (full replace) — phân biệt rõ để chọn đúng. Với lọc/sắp xếp/phân trang, dùng query param (`?status=OPEN&sort=-createdAt&limit=20`) thay vì tạo path mới.

Ví dụ thực tế: Trong Jira Clone, việc đổi status issue (TODO -> IN_PROGRESS) tôi mô hình hóa như một transition resource thay vì nhồi vào PATCH chung, vì nó có ràng buộc nghiệp vụ (workflow) riêng.
</details>

<details>
<summary><b>24.2 Bạn chọn HTTP status code như thế nào cho các case: tạo mới, validation fail, chưa đăng nhập, không đủ quyền, conflict?</b></summary>

Phân biệt rõ 4xx (lỗi client) và 5xx (lỗi server). Tạo mới thành công trả `201 Created` kèm header `Location` trỏ tới resource vừa tạo; cập nhật/xóa thành công nhưng không trả body dùng `204 No Content`. Validation thất bại thường là `400 Bad Request`; một số team dùng `422 Unprocessable Entity` khi cú pháp JSON hợp lệ nhưng vi phạm ràng buộc nghiệp vụ/semantic (lưu ý 422 thuộc chuẩn WebDAV/RFC 4918, không bắt buộc — chọn một quy ước và nhất quán). Phân biệt `401 Unauthorized` (chưa xác thực hoặc credential sai/hết hạn — tên gọi gây hiểu nhầm, thật ra là "unauthenticated") với `403 Forbidden` (đã xác thực nhưng không đủ quyền). Xung đột trạng thái/trùng ràng buộc unique dùng `409 Conflict`.

```
201 Created      POST /issues        -> tao issue moi (kem header Location)
204 No Content   DELETE /issues/:id  -> xoa thanh cong, khong body
400 Bad Request  body sai schema (DTO validation)
401 Unauthorized JWT cookie thieu/het han hoac khong hop le
403 Forbidden    user khong phai LEAD/ADMIN cua project
409 Conflict     tao workspace trung slug (vi pham unique)
```

Ví dụ thực tế: RBAC nhiều tầng của Jira Clone trả `403` khi user có token hợp lệ nhưng vai trò project (VIEWER) không đủ quyền sửa, còn `401` chỉ xảy ra khi cookie JWT không hợp lệ hoặc hết hạn.
</details>

<details>
<summary><b>24.3 Idempotency là gì? GET, POST, PUT, DELETE — cái nào idempotent, cái nào safe, và xử lý POST tạo trùng thế nào?</b></summary>

Idempotent nghĩa là gọi N lần cho cùng **trạng thái server** như gọi 1 lần (status code trả về có thể khác). Cần phân biệt với **safe**: method safe là không thay đổi trạng thái server (`GET`, `HEAD`). `GET`, `PUT`, `DELETE` là idempotent; `GET` ngoài ra còn safe. `POST` không idempotent (gọi 2 lần tạo 2 resource). `PUT` idempotent vì thay thế toàn bộ resource tại một URI xác định — gọi lại cho cùng kết quả. `DELETE` idempotent vì sau lần xóa đầu, trạng thái vẫn là "đã xóa" (lần 2 thường trả 404 hoặc 204, nhưng trạng thái không đổi). Để chống tạo trùng do client retry vì lỗi mạng, dùng **Idempotency-Key** header: server lưu key kèm kết quả, nếu thấy key đã xử lý thì trả response cũ thay vì tạo mới.

```http
POST /payments
Idempotency-Key: 9f1c-uuid-from-client
# Server: neu key da ton tai -> tra response da luu, khong charge lai
```

Ví dụ thực tế: Khi sinh issue key `PROJECT-N` trong Jira Clone, tôi bọc trong `$transaction` + tăng counter atomic, nên đảm bảo tính nhất quán khi có request đồng thời. Lưu ý: transaction giải quyết concurrency/atomicity, còn chống **retry trùng** thì cần thêm Idempotency-Key (lưu key + kết quả) cho endpoint nhạy cảm.
</details>

<details>
<summary><b>24.4 Các chiến lược API versioning? Bạn chọn cái nào và tại sao?</b></summary>

Ba cách phổ biến: URI versioning (`/v1/users`), header versioning — có thể là custom header (`X-API-Version: 1`) hoặc media type qua `Accept` (`Accept: application/vnd.api.v1+json`, gọi là media-type versioning), và query param (`?version=1`). URI rõ ràng, dễ cache, dễ test bằng browser/curl nên phổ biến nhất; đánh đổi là "không thuần REST" về mặt lý thuyết (URI nên trỏ tới resource ổn định bất kể version). Header/media-type versioning sạch hơn về mặt resource nhưng khó debug và khó cache hơn. Quan trọng là coi version là hợp đồng: chỉ bump major khi có breaking change, và ưu tiên giữ backward-compatible bằng cách thêm field thay vì đổi/xóa field cũ.

```ts
// NestJS ho tro nhieu kieu versioning natively
const app = await NestFactory.create(AppModule);
app.enableVersioning({ type: VersioningType.URI }); // -> /v1/...

@Controller({ path: 'issues', version: '1' })
export class IssuesV1Controller {}
```

Lưu ý: NestJS còn hỗ trợ `VersioningType.HEADER`, `VersioningType.MEDIA_TYPE` và `VersioningType.CUSTOM`, chọn type khi gọi `enableVersioning`.

Ví dụ thực tế: Trong Jira Clone tôi prefix `/api` (qua `setGlobalPrefix`) và có thể bật `enableVersioning` của NestJS để gắn `version` per-controller khi cần tách v2 mà không đụng đến v1.
</details>

<details>
<summary><b>24.5 So sánh offset pagination và cursor pagination — khi nào dùng cái nào?</b></summary>

Offset (`LIMIT 20 OFFSET 100`) đơn giản, cho phép nhảy trang bất kỳ, hiện tổng số trang, nhưng có 2 nhược: deep offset chậm (DB vẫn phải duyệt và bỏ qua 100 dòng đầu), và bị "page drift" khi có insert/delete giữa các lần gọi (item bị lặp hoặc bị nhảy qua). Cursor pagination dùng một mốc ổn định (thường là cột đã index như `id` hoặc `(createdAt, id)`) làm con trỏ `WHERE id < cursor ORDER BY id DESC` — ổn định với dữ liệu thay đổi và nhanh vì tận dụng index thay vì quét-và-bỏ-qua; đánh đổi là chỉ đi tiến/lùi tuần tự, không nhảy tùy ý trang.

```ts
// Cursor pagination voi Prisma
const issues = await prisma.issue.findMany({
  take: 20,
  ...(cursor && { skip: 1, cursor: { id: cursor } }),
  orderBy: { id: 'desc' },
});
const nextCursor = issues.length === 20 ? issues.at(-1)?.id : null;
```

Lưu ý: cột dùng làm cursor phải duy nhất và có thứ tự ổn định. Nếu sắp xếp theo `createdAt` có thể trùng, hãy kết hợp thêm `id` làm tie-breaker (`orderBy: [{ createdAt: 'desc' }, { id: 'desc' }]`) để tránh bỏ sót/lặp.

Ví dụ thực tế: Jira Clone dùng cursor pagination cho danh sách issue vì board được cập nhật liên tục — cursor tránh việc issue mới tạo làm lệch trang khi user scroll.
</details>

<details>
<summary><b>24.6 Response envelope { message, data } và error format nhất quán — bạn thiết kế ra sao trong NestJS?</b></summary>

Một envelope đồng nhất giúp client xử lý predictable: success trả `{ message, data }`, lỗi trả cấu trúc cố định (`statusCode`, `message`, `error`, optional `code`). Trong NestJS tôi dùng **Interceptor** bọc mọi response thành công và **ExceptionFilter** chuẩn hóa lỗi, nhờ đó controller chỉ cần `return data`. Nên tránh bọc metadata thừa khi không dùng đến. Với pagination, thêm block `meta: { nextCursor, hasMore }` bên cạnh `data`.

```ts
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, { message: string; data: T }> {
  intercept(ctx: ExecutionContext, next: CallHandler<T>): Observable<{ message: string; data: T }> {
    return next.handle().pipe(
      map((data) => ({ message: 'success', data })),
    );
  }
}
```

Lưu ý: Interceptor bọc response **không** chạy khi một exception được throw (luồng đi thẳng sang ExceptionFilter), nên success-shape và error-shape được xử lý ở hai nơi tách biệt — đúng trong mọi trường hợp.

Ví dụ thực tế: Trong Jira Clone, AllExceptionsFilter chuẩn hóa mọi lỗi về cùng shape và đồng thời đẩy 5xx lên Sentry + structured log (đã sanitize PII), nhờ đó FE chỉ xử lý một định dạng error duy nhất.
</details>

<details>
<summary><b>24.7 HATEOAS là gì và content negotiation hoạt động thế nào? Bạn có áp dụng HATEOAS không?</b></summary>

HATEOAS (level 3 trong Richardson Maturity Model) là việc response chứa các `links` chỉ dẫn hành động tiếp theo (`self`, `next`, `transitions`), giúp client khám phá API từ trạng thái hiện tại mà không hardcode URL. Thực tế phần lớn REST API (gồm Jira Clone) dùng ở level 2 (resource + HTTP verbs dùng chuẩn) vì HATEOAS đầy đủ tốn chi phí và FE thường đã biết routes — tôi chỉ nhúng link khi thật sự cần (vd pagination `next`). Content negotiation là việc client và server thỏa thuận định dạng/ngôn ngữ: client gửi mong muốn qua `Accept` (`application/json` vs `application/xml`) và `Accept-Language`, server chọn và phản hồi `Content-Type` tương ứng (trả `406 Not Acceptable` nếu không đáp ứng được).

```http
GET /issues/42
Accept: application/json
Accept-Language: vi
--- response ---
Content-Type: application/json
{ "data": { ... }, "links": { "self": "/issues/42", "transitions": "/issues/42/transitions" } }
```

Lưu ý phân biệt: `Accept`/`Accept-Language` là **request** header (client gửi), `Content-Type` là header mô tả body của **cả request lẫn response**; ở phía validation request, NestJS kiểm `Content-Type: application/json` của body gửi lên.

Ví dụ thực tế: Jira Clone dùng JSON cố định nên content negotiation chủ yếu ở tầng validate `Content-Type` của request; HATEOAS tôi chỉ áp một phần qua `nextCursor` trong meta pagination chứ không full hypermedia.
</details>

<details>
<summary><b>24.8 Rate limiting và caching trong REST API — bạn thiết kế ra sao và dùng header nào?</b></summary>

Rate limiting bảo vệ API khỏi lạm dụng/DoS và đảm bảo công bằng. Khi vượt ngưỡng, trả `429 Too Many Requests` kèm header `Retry-After` (số giây chờ đợi) và thường thêm `X-RateLimit-Limit/Remaining/Reset` để client tự điều tiết. Thuật toán phổ biến: token bucket, sliding window. Trong NestJS dùng `@nestjs/throttler` (`ThrottlerModule`), nên lưu counter ở Redis khi chạy nhiều instance để đếm chung.

Caching giảm tải và độ trễ: HTTP caching qua `Cache-Control` (`max-age`, `no-cache`, `private/public`) và conditional request với `ETag`/`If-None-Match` — nếu resource không đổi server trả `304 Not Modified` (không gửi lại body). Ở tầng server, dùng Redis cache cho dữ liệu đọc nhiều/ghi ít, kèm chiến lược invalidation rõ ràng (vd xóa key khi resource thay đổi) để tránh stale.

```http
GET /issues/42
If-None-Match: "v3-abc"
--- response ---
304 Not Modified            # body rong, dung lai ban cache cua client
# hoac 200 OK + ETag: "v4-def" neu da thay doi
```

Ví dụ thực tế: Jira Clone giới hạn endpoint đăng nhập chặt hơn (chống brute-force) bằng ThrottlerGuard riêng, và cache các truy vấn read-heavy (vd metadata project) ở Redis với TTL ngắn + invalidate khi có mutation.
</details>

<details>
<summary><b>24.9 Phân biệt PUT và PATCH, và làm sao xử lý cập nhật đồng thời (concurrent update / lost update)?</b></summary>

PUT thay thế **toàn bộ** resource (client gửi đầy đủ representation, field thiếu sẽ bị coi như set về null/default tùy thiết kế) và idempotent. PATCH cập nhật **một phần** (chỉ gửi field cần đổi); PATCH không nhất thiết idempotent (vd patch dạng JSON Patch với thao tác `add` vào mảng). Trong thực tế REST API thường dùng PATCH cho update vì client ít khi muốn gửi lại toàn bộ object.

Vấn đề **lost update**: hai client đọc cùng phiên bản rồi ghi đè lên nhau. Giải pháp phổ biến là **optimistic concurrency control**: dùng cột `version` (hoặc `updatedAt`/`ETag`) làm điều kiện khi ghi. Nếu version không khớp -> trả `409 Conflict` (hoặc `412 Precondition Failed` khi dùng header `If-Match`).

```ts
// Optimistic locking voi Prisma: chi update khi version con khop
const result = await prisma.issue.updateMany({
  where: { id, version: clientVersion },
  data: { title, version: { increment: 1 } },
});
if (result.count === 0) {
  // ai do da update truoc -> version lech
  throw new ConflictException('Issue da bi thay doi, vui long tai lai');
}
```

Ví dụ thực tế: Trong Jira Clone, khi hai user cùng sửa một issue, optimistic locking bằng cột `version` giúp phát hiện xung đột và báo client reload thay vì âm thầm ghi đè công việc của người kia.
</details>

---

## 25. Scheduled and Background Jobs

<details>
<summary><b>25.1 @nestjs/schedule hoạt động thế nào? Phân biệt @Cron, @Interval, @Timeout và vai trò của ScheduleModule.forRoot()</b></summary>

`ScheduleModule.forRoot()` khởi tạo `SchedulerRegistry` và kích hoạt orchestrator quét các provider để tìm các handler gắn decorator; các job này được đăng ký vào registry và bắt đầu chạy khi `onApplicationBootstrap`. Điểm cần chính xác: ba decorator KHÔNG dùng chung một cơ chế định thời.

- `@Cron` chạy theo biểu thức cron và được triển khai bằng lớp `CronJob` của package `cron` (hỗ trợ 6 trường có giây và `timeZone`), KHÔNG phải `setInterval`.
- `@Interval` chạy lặp lại mỗi N mili-giây, bên dưới là `setInterval`.
- `@Timeout` chạy một lần sau N mili-giây, bên dưới là `setTimeout`.

Cả ba đều chạy trong cùng event loop của tiến trình Node, nên handler phải non-blocking và PHẢI tự `try/catch`: với `@Interval`/`@Timeout` một exception không bắt sẽ thành unhandled rejection, còn với `@Cron` package `cron` sẽ nuốt lỗi và bỏ qua lần chạy đó — cả hai đều không mong muốn.

```ts
@Injectable()
export class LogRetentionJob {
  @Cron('0 0 3 * * *', { name: 'log-retention', timeZone: 'Asia/Ho_Chi_Minh' })
  async handle() {
    try {
      await this.logService.deleteOlderThan(30);
    } catch (err) {
      this.logger.error('log-retention failed', err);
    }
  }
}
```

Ví dụ thực tế: trong Jira Clone tôi dùng `@Cron` cho retention dọn dẹp structured log cũ chạy lúc 3h sáng theo giờ Việt Nam.
</details>

<details>
<summary><b>25.2 Vì sao background job phải idempotent và bạn đảm bảo idempotency như thế nào?</b></summary>

Job có thể chạy lại do retry sau lỗi, do nhiều instance/worker cùng nhận, hoặc do crash giữa chừng rồi job được đưa lại vào queue. Nếu không idempotent sẽ gây double-charge, gửi email trùng, hoặc sai dữ liệu. Cách phổ biến: dùng idempotency key lưu trong DB với ràng buộc `unique`, và tận dụng chính ràng buộc đó làm điểm đồng bộ "check-then-act" thay vì đọc rồi ghi (đọc-rồi-ghi vẫn bị race giữa hai lần chạy song song).

Với tác vụ có side effect bên ngoài (gửi email, gọi thanh toán): nên chia hai bước — ghi bản ghi trạng thái trước (PENDING) trong transaction, gọi dịch vụ ngoài, rồi cập nhật SENT. Lưu ý gọi dịch vụ ngoài KHÔNG nằm trong DB transaction (transaction không rollback được email đã gửi).

```ts
async sendOtp(userId: string, key: string) {
  // rang buoc unique(userId, key): lan thu 2 se nem loi -> ta hieu la "da gui"
  try {
    await this.prisma.otpSend.create({ data: { userId, key, status: 'PENDING' } });
  } catch (e) {
    if (isUniqueViolation(e)) return; // da co ban ghi -> bo qua, tranh gui trung
    throw e;
  }
  await this.resend.send(/* ... */);
  await this.prisma.otpSend.update({ where: { userId_key: { userId, key } }, data: { status: 'SENT' } });
}
```

Ví dụ thực tế: khi sinh issue key `PROJECT-N`, tôi dùng counter có ràng buộc unique trên `(projectId, number)` nên retry không tạo trùng key.
</details>

<details>
<summary><b>25.3 Khi nào dùng queue-based processing (Bull/BullMQ) thay vì @Cron? So sánh</b></summary>

`@Cron` hợp với công việc định kỳ theo lịch (retention, báo cáo hàng ngày) và đơn giản, chạy trong tiến trình ứng dụng. Queue (BullMQ trên Redis) hợp với công việc phát sinh theo sự kiện, cần xử lý nặng/chậm (gửi email, resize ảnh, gọi API bên thứ ba) mà không nên block request HTTP. Queue cho bạn: concurrency control, retry/backoff sẵn có, rate limiting, ưu tiên job, theo dõi trạng thái job, và độ bền (job nằm trong Redis nên không mất khi app restart) — trong khi `@Cron` chỉ là trigger theo thời gian, chạy in-process và mất lượt nếu app đang down.

Thực tế thường kết hợp cả hai: dùng `@Cron` (hoặc BullMQ repeatable/job scheduler) để enqueue job theo lịch, rồi worker xử lý thực sự.

```ts
// Producer: tra response ngay, day viec nang vao queue
await this.emailQueue.add('welcome', { userId }, {
  attempts: 3,
  backoff: { type: 'exponential', delay: 2000 },
  removeOnComplete: true,
  removeOnFail: 1000, // giu lai mot so job loi de dieu tra
});
```

Ví dụ thực tế: trong Jira Clone, gửi email/OTP qua Resend nên đẩy vào queue để endpoint đăng ký trả về nhanh, tránh tăng latency và tránh lỗi mạng làm fail cả request đăng ký.
</details>

<details>
<summary><b>25.4 Khi deploy nhiều instance, làm sao chạy cron an toàn để job không chạy trùng? (distributed lock)</b></summary>

Mỗi instance đều có scheduler riêng, nên cùng một `@Cron` sẽ chạy trên tất cả instance cùng lúc gây double-run. Các giải pháp:

1) Distributed lock với Redis `SET key value NX PX <ttl>`: chỉ instance giành được khóa mới chạy; TTL tránh deadlock khi instance chết giữa chừng.
2) Leader election (chỉ leader chạy cron).
3) Tốt nhất: chuyển sang BullMQ repeatable job / job scheduler — Redis đảm bảo mỗi lần đáo hạn chỉ tạo MỘT delayed job, và job đó chỉ được MỘT worker pick up, nên không cần lock thủ công.

Lưu ý khi tự làm lock: (a) TTL phải đủ dài hơn thời gian chạy job tối đa; (b) job vẫn nên idempotent để phòng khóa hết hạn giữa chừng; (c) khi release phải kiểm tra đúng chủ sở hữu MỘT CÁCH NGUYÊN TỬ bằng Lua script (so sánh value rồi `DEL`), nếu không sẽ vô tình xóa khóa của instance khác.

```ts
@Cron('0 0 * * *')
async nightly() {
  // ioredis: thu tu dung la value, PX, ttl, NX
  const ok = await this.redis.set('lock:nightly', this.instanceId, 'PX', 60000, 'NX');
  if (ok !== 'OK') return; // instance khac da gianh khoa
  try {
    await this.runJob();
  } finally {
    // release nguyen tu: chi xoa neu van la chu so huu
    await this.redis.eval(
      "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end",
      1, 'lock:nightly', this.instanceId,
    );
  }
}
```

Ví dụ thực tế: cron retention log trong Jira Clone nếu scale ra nhiều pod thì phải thêm Redis lock (hoặc chuyển sang BullMQ scheduler) để không quét/xóa trùng trên cùng dataset.
</details>

<details>
<summary><b>25.5 Retry và backoff trong xử lý job: bạn cấu hình thế nào và lưu ý gì?</b></summary>

Nên retry với exponential backoff (ví dụ 1s, 2s, 4s, 8s...) có thêm jitter để tránh thundering herd, và giới hạn số lần thử (`attempts`). Chỉ retry lỗi có thể phục hồi (timeout, 503, lỗi mạng); lỗi nghiệp vụ/không thể phục hồi (validation 400, 404) thì nên dừng lại ngay — BullMQ có `UnrecoverableError` để fail luôn không retry tiếp. Sau khi hết lượt, BullMQ chuyển job vào tập `failed` (KHÔNG phải một "DLQ" có sẵn — DLQ là pattern bạn tự xây, ví dụ listener `failed` đẩy job sang một queue riêng để xử lý tay). Retry chỉ an toàn khi job idempotent.

```ts
await queue.add('sync-issue', payload, {
  attempts: 5,
  backoff: { type: 'exponential', delay: 1000 }, // ~1s,2s,4s,8s,16s
});

// Trong processor: loi khong the phuc hoi thi khong retry
// throw new UnrecoverableError('email format invalid');
```

Lưu ý: backoff `exponential` mặc định của BullMQ không tự thêm jitter; muốn jitter thì dùng custom backoff strategy. Ví dụ thực tế: khi gọi Resend gửi OTP bị lỗi mạng/timeout, tôi cấu hình retry có backoff; còn lỗi email sai định dạng thì ném `UnrecoverableError` và trả lỗi người dùng ngay.
</details>

<details>
<summary><b>25.6 Graceful shutdown cho background job/worker: tại sao cần và làm thế nào trong NestJS?</b></summary>

Khi deploy/scale, container nhận SIGTERM; nếu tắt đột ngột thì job đang chạy có thể bị cắt giữa chừng gây dữ liệu không nhất quán, hoặc job bị đánh dấu lỗi/treo. Graceful shutdown nghĩa là: ngừng nhận job mới, chờ job hiện tại hoàn tất trong một deadline, rồi đóng kết nối DB/Redis.

Trong NestJS: gọi `app.enableShutdownHooks()` trong `main.ts` để Nest lắng nghe tín hiệu hệ thống, và triển khai `OnModuleDestroy`/`beforeApplicationShutdown` để dọn dẹp. Với BullMQ gọi `worker.close()` — nó sẽ chờ job đang chạy xong (theo `worker` settings) trước khi trả về. Kết hợp với `terminationGracePeriodSeconds` của Kubernetes (phải đủ lớn hơn deadline xử lý job, nếu không K8s gửi SIGKILL cắt ngang).

```ts
@Injectable()
export class EmailWorker implements OnModuleDestroy {
  async onModuleDestroy() {
    await this.worker.close(); // ngung nhan job moi, cho job hien tai xong
  }
}
// main.ts: app.enableShutdownHooks();
```

Ví dụ thực tế: API Jira Clone bật shutdown hooks để khi rolling deploy, request và job đang xử lý được hoàn tất trước khi pod bị xóa.
</details>

<details>
<summary><b>25.7 Vì sao nên tách worker process ra khỏi API process? Đánh đổi gì?</b></summary>

Tách worker giúp cô lập tài nguyên: job nặng (CPU/IO) không làm tăng latency hay chiếm event loop của API phục vụ request người dùng. Nó còn cho phép scale độc lập (nhiều worker khi queue sâu, nhiều API khi traffic cao), triển khai/restart riêng, và giới hạn blast radius khi worker crash. Đánh đổi: phức tạp vận hành hơn — thêm deployment, cần một message broker (Redis) làm ranh giới, khó debug/trace end-to-end hơn (nên có correlation id giữa request và job).

Thường dùng chung codebase nhưng khởi động bằng entrypoint khác. Worker dùng `createApplicationContext` (không mở HTTP server) và nạp một module chỉ chứa provider/worker, không import controller.

```ts
// main-worker.ts: chi nap WorkerModule, khong mo HTTP server
const app = await NestFactory.createApplicationContext(WorkerModule);
app.enableShutdownHooks();
```

Ví dụ thực tế: nếu Jira Clone tăng lượng gửi email/notification, tôi sẽ tách worker chạy BullMQ riêng để API giữ p99 latency thấp và có thể scale riêng theo độ sâu của queue.
</details>

<details>
<summary><b>25.8 Làm sao quan sát (observability) và cảnh báo cho background job? Đo gì và cảnh báo khi nào?</b></summary>

Job chạy nền, không có user ngồi chờ, nên nếu không đo đạc thì lỗi âm thầm rất lâu mới bị phát hiện. Cần đo và cảnh báo theo nhiều góc:

- Metrics: độ sâu queue (waiting/active/delayed), số job `failed`, thời gian xử lý (p50/p95), processing rate, và độ trễ (job age) — nếu queue tăng không giảm thì worker đang không theo kịp.
- Logging: mỗi job nên có correlation/trace id để nối từ request sang job; log kết quả và exception.
- Alerting: cảnh báo khi số `failed` vượt ngưỡng, khi delayed/waiting tăng liên tục, khi job chạy quá lâu (stalled), hoặc khi cron không chạy đúng lịch (dùng heartbeat/dead-man-switch như Healthchecks.io — nếu hết hạn mà cron chưa "ping" thì báo).
- Dashboard vận hành: Bull Board / Arena để xem và retry job thủ công.

Ví dụ: trong Jira Clone tôi đặt alert khi queue email có job stalled hoặc số failed tăng đột biến, và dùng correlation id để truy từ request đăng ký đến job gửi OTP tương ứng.
</details>

---
## 26. Worked Examples for Core Concepts

<details>
<summary><b>26.1 Decorator và metadata reflection trong NestJS hoạt động thế nào? Vì sao cần `reflect-metadata` và `emitDecoratorMetadata`?</b></summary>

Decorator (`@Controller`, `@Get`, `@UseGuards`) là **syntactic sugar** của JavaScript — chúng **chuyển đổi class/method/parameter thành metadata** qua API Reflection. TypeScript (với `emitDecoratorMetadata: true`) tự phát khí loại tham số (`reflect-metadata` thư viện) vào runtime; NestJS đọc metadata này để:

1. Biết tham số nào là `@Body()`, `@Param()`, `@Query()`, và kiểu của chúng
2. Giải quyết Dependency Injection (biết class nào cần inject gì)
3. Lấy metadata decorator tự tạo (ví dụ `@Roles('admin')`) để guard/filter sử dụng

Nhờ vậy, NestJS "thông minh" — không cần khai báo function type signature thủ công như Express. Nếu thiếu `emitDecoratorMetadata` hoặc `reflect-metadata` → metadata trống → DI lỗi `Can't resolve dependencies`.

```ts
// 1. tsconfig.json phải có
{
  "compilerOptions": {
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true
  }
}

// 2. main.ts phải import (một lần đủ cho cả app)
import 'reflect-metadata';

// 3. Custom decorator ghi nhãn
export const IsOwner = () => SetMetadata('isOwner', true);

// 4. Guard đọc lại nhãn
@Injectable()
export class OwnerGuard implements CanActivate {
  constructor(private reflector: Reflector) {}
  canActivate(ctx: ExecutionContext): boolean {
    const isOwner = this.reflector.get<boolean>('isOwner', ctx.getHandler());
    if (!isOwner) return true; // decorator không có → cho qua
    // Nếu có -> kiểm tra user là chủ tài nguyên
    const { user } = ctx.switchToHttp().getRequest();
    const postId = ctx.switchToHttp().getRequest().params.id;
    return user.id === postId; // giả sử logic xác định chủ sở hữu
  }
}
```

Ví dụ thực tế: Jira Clone dùng `@Roles('ADMIN')` trên endpoint xoá issue → `RolesGuard` đọc metadata để kiểm tra role trước khi handler chạy. Nếu Reflection bị hỏng → guard không biết yêu cầu role là gì → bỏ qua kiểm tra → **lỗ hổng bảo mật**.
</details>

<details>
<summary><b>26.2 Active Record vs Data Mapper — khi nào dùng cái nào? Ảnh hưởng đến DI và testing thế nào?</b></summary>

**Active Record**: entity tự **lo việc lưu** — `user.save()`, entity chứa cả business logic và persistence. Ưu: gọn gàng, ít boilerplate. Nhược: entity nặng, khó test (phải mock toàn bộ entity), tight coupling.

**Data Mapper** (Repository pattern): entity chỉ là **data thuần**, `Repository` lo việc persist. Ưu: entity nhẹ, logic riêng biệt, dễ mock trong test, phù hợp DI.

NestJS khuyến khích **Data Mapper** vì:
1. **DI mạnh**: repository inject vào service, dễ mock bằng test double
2. **Clean Architecture**: tách logic từ model
3. **Tái sử dụng**: cùng repository cho nhiều service

```ts
// ❌ Active Record (không hợp NestJS)
class User extends BaseEntity {
  name: string;
  async save() { await User.update(...) }
  async findById(id) { return await User.findOne(id) }
}
const user = new User();
await user.save(); // entity tự persist

// ✅ Data Mapper (NestJS pattern)
class User {
  id: number;
  name: string;
  // Không có method save/find
}

@Injectable()
export class UsersService {
  constructor(
    @InjectRepository(User)
    private userRepo: Repository<User>,
  ) {}
  
  async createUser(dto: CreateUserDto): Promise<User> {
    const user = this.userRepo.create(dto);
    return await this.userRepo.save(user);
  }
}

// Unit test — dễ mock repository
const mockUserRepo = {
  create: jest.fn().mockReturnValue({ id: 1, name: 'Alice' }),
  save: jest.fn().mockResolvedValue({ id: 1, name: 'Alice' }),
};

const service = new UsersService(mockUserRepo as any);
// -> service không biết repository thật là gì, test độc lập
```

Ví dụ thực tế: Jira Clone, khi tạo issue, `IssuesService` inject `IssueRepository` → trong unit test, mock repo để test logic phát sinh issue key (`PROJECT-N`) mà không phải lắng nghe DB thật.
</details>

<details>
<summary><b>26.3 Async/await và Promise.then() khác gì? Tại sao async/await dễ hiểu hơn khi handle error?</b></summary>

**Promise.then()** là **callback-based** — chạy code tiếp khi Promise resolve, chain bằng `.then(...).catch(...)`. Khó đọc khi logic phức tạp ("callback hell" hay "pyramid of doom").

**async/await** là **syntactic sugar** trên Promise — viết code như là **synchronous** nhưng thực tế nó suspend/resume. Chẳng hạn `await x` tạm dừng function cho đến khi Promise resolve, rồi tiếp tục dòng sau.

Khác biệt lớn: **error handling**.

```ts
// Promise.then() — error handling phân tán
usersService.findOne(id)
  .then(user => postsService.findByAuthor(user.id))
  .then(posts => res.json(posts))
  .catch(err => res.status(500).json(err));
// Khi nào error từ findOne, khi nào từ findByAuthor?
// Cần kiểm tra err.stack hoặc tự wrapper để biết

// async/await — error handling tập trung, rõ ràng
try {
  const user = await usersService.findOne(id);  // nếu throw → jump vào catch
  const posts = await postsService.findByAuthor(user.id);
  res.json(posts);
} catch (err) {
  if (err instanceof NotFoundException) {
    res.status(404).json(err);
  } else if (err instanceof ValidationException) {
    res.status(400).json(err);
  } else {
    res.status(500).json(err);
  }
}
```

Kỹ thuật: `async function` **luôn return Promise** — có `await` hay không cũng là Promise. Khi `await x` và x reject → exception "ném ra" để catch bắt; nếu không catch → Promise reject.

```ts
// Controller
@Get(':id')
async getPost(@Param('id') id: string) {
  // implicit return Promise<Post>
  const post = await this.postsService.findOne(id); // nếu không tìm -> throw NotFoundException
  return post; // Nest tự wrap thành resolved Promise
}
// Nếu exception → NestJS's exception filter bắt → trả 404
```

Ví dụ thực tế: Jira Clone, khi tạo issue và gán người chủ trì (await userService.findOne(assigneeId)), nếu user không tồn tại → throw NotFoundException → catch ở service hoặc exception filter → 404 sạch sẽ.
</details>

<details>
<summary><b>26.4 Request lifecycle trong NestJS — thứ tự Guard → Pipe → Interceptor → Handler → Filter chạy như thế nào? Ảnh hưởng tới ký tự nào?</b></summary>

Thứ tự **chặt chẽ** (miễn là không exception):

**Middleware → Guard → Interceptor (before) → Pipe → Handler → Interceptor (after) → Filter**

Mỗi layer có mục đích riêng:
1. **Middleware** (Express): chạy **sớm nhất**, không biết route/handler nào → dùng cho CORS, logging chung
2. **Guard**: kiểm tra **authorization** — nếu `canActivate()` return false → 403, **không chạy handler**
3. **Interceptor (before)**: biến đổi request trước handler
4. **Pipe**: **validate & transform data** — nếu throw → 400, **không chạy handler**
5. **Handler**: logic thực tế
6. **Interceptor (after)**: bọc response trả về
7. **Exception Filter**: bắt exception từ handler/interceptor/pipe

```ts
@Controller('posts')
@UseGuards(JwtAuthGuard, RolesGuard) // chạy lần lượt
@UseInterceptors(LoggingInterceptor)  // wraps handler
export class PostsController {
  @Post()
  @HttpCode(201)
  @Roles('ADMIN')
  createPost(
    @Body(new ValidationPipe({ whitelist: true })) // Pipe chạy ở đây
    dto: CreatePostDto,
    @CurrentUser() user,
  ) {
    console.log('handler chạy'); // lúc này user đã xác thực (Guard qua)
    // dto đã validate & transform (Pipe qua)
    return this.postsService.create(dto, user);
  }
}

// Thứ tự chạy:
// 1. Request vào
// 2. JwtAuthGuard -> verify token, set request.user
// 3. RolesGuard -> kiểm tra user.role === 'ADMIN'
// 4. LoggingInterceptor (before) -> log request
// 5. ValidationPipe (cho @Body) -> check DTO constraints
// 6. createPost handler
// 7. LoggingInterceptor (after) -> log response + thời gian
// 8. Response trả về (hay Exception Filter bắt nếu có exception)
```

Ví dụ **bẫy**: Nếu Guard return false → **Pipe không chạy**, handler không chạy → response 403 sạch. Điều này tốt — không đánh giá dữ liệu của user không được phép.

Ví dụ thực tế: Jira Clone, endpoint `PATCH /issues/:id` — JwtAuthGuard check token, RolesGuard check workspace role (LEAD/ADMIN), OwnershipGuard check chủ sở hữu issue. Nếu Guard nào fail → 403, không bao giờ tạo issue mới. Pipe validate dto. Handler cập nhật issue. Interceptor log audit trail.
</details>

<details>
<summary><b>26.5 JWT Refresh Token Rotation — vì sao cần? Cơ chế cấp lại token thế nào mà khó bị CSRF?</b></summary>

**Access Token** (ngắn hạn, 15 phút): chứa payload (`sub`, `role`), client gửi mỗi request qua header `Authorization: Bearer`. **Refresh Token** (dài hạn, 7 ngày): chỉ để cấp lại access token, lưu ở client dạng httpOnly cookie (không JS access được → chống XSS).

Cơ chế:
1. Login → server phát access token + refresh token (httpOnly cookie)
2. Client dùng access token gọi API
3. Access token hết hạn → client gửi refresh token qua endpoint `/auth/refresh`
4. Server verify refresh token + **phát cặp access/refresh token MỚI** (rotation)
5. Cookie cũ replace bằng cookie mới

**Rotation** có nghĩa: mỗi lần dùng refresh token, server cấp cặp token mới và **vô hiệu hóa token cũ**. Nếu attacker lấy được 1 token → chỉ dùng được 1 lần, request tiếp theo cần token mới → phát hiện được nếu refresh từ device lạ.

```ts
// login endpoint
@Post('login')
async login(@Body() dto: LoginDto, @Res() res: Response) {
  const user = await this.authService.validateUser(dto);
  const { accessToken, refreshToken } = await this.authService.generateTokens(user);
  
  // refresh token -> httpOnly cookie
  res.cookie('refreshToken', refreshToken, {
    httpOnly: true,
    secure: true,  // HTTPS only
    sameSite: 'strict', // chống CSRF
    maxAge: 7 * 24 * 60 * 60 * 1000, // 7 ngày
  });
  
  return res.json({ accessToken }); // access token -> response body
}

// refresh endpoint
@Post('refresh')
async refresh(
  @Req() req: Request,
  @Res() res: Response,
) {
  const oldRefreshToken = req.cookies.refreshToken;
  const user = await this.authService.verifyRefreshToken(oldRefreshToken);
  
  // Rotation: phát cặp mới + vô hiệu hóa cũ
  const { accessToken, refreshToken: newRefreshToken } = 
    await this.authService.generateTokens(user, oldRefreshToken);
  
  res.cookie('refreshToken', newRefreshToken, {
    httpOnly: true,
    secure: true,
    sameSite: 'strict',
    maxAge: 7 * 24 * 60 * 60 * 1000,
  });
  
  return res.json({ accessToken });
}

// Service
@Injectable()
export class AuthService {
  async generateTokens(
    user: User,
    oldRefreshToken?: string,
  ) {
    // Nếu oldRefreshToken cung cấp -> vô hiệu hóa nó
    if (oldRefreshToken) {
      await this.tokenRepository.update(
        { token: oldRefreshToken },
        { revokedAt: new Date() },
      );
    }
    
    const accessToken = this.jwtService.sign(
      { sub: user.id, email: user.email, role: user.role },
      { expiresIn: '15m' },
    );
    
    const refreshToken = this.jwtService.sign(
      { sub: user.id, version: user.tokenVersion }, // version để phát hiện rotation
      { expiresIn: '7d' },
    );
    
    // Lưu refresh token (để kiểm tra khi refresh lại)
    await this.tokenRepository.save({
      token: refreshToken,
      userId: user.id,
      expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000),
    });
    
    return { accessToken, refreshToken };
  }
}
```

Ví dụ thực tế: Jira Clone, user login → cấp access token + refresh token (httpOnly). Mỗi request API dùng access token. Sau 15 phút, browser tự gọi `/auth/refresh` (có cookie) → server phát access token mới + refresh token mới (rotate). Nếu một attacker chộp được refresh token từ request/response → refresh một lần → token cũ invalid. Request lần sau với token cũ → 401, phát hiện intrusion.
</details>

<details>
<summary><b>26.6 Transactional operations & atomic writes — vì sao cần khi phát hành issue key (PROJECT-N)? Concurrent requests có vấn đề gì?</b></summary>

**Transaction** = tập hợp các thay đổi DB được **đảm bảo atomic** — tất cả commit hoặc tất cả rollback. Dùng khi **nhiều bước DB phụ thuộc vào nhau** và nếu 1 bước fail → toàn bộ fail.

**Vấn đề concurrent** với issue key: giả sử logic "lấy số count issue, cộng 1, save". Nếu 2 request cùng lúc:
- Request A: read count = 5
- Request B: read count = 5 (A chưa write)
- A: write count = 6, issue key = PROJECT-6
- B: write count = 6, issue key = PROJECT-6 ← **TRÙNG!**

Khắc phục: **transaction + lock** hoặc **row counter increment atomic**.

```ts
// ❌ Không atomic (race condition)
async createIssue(dto: CreateIssueDto, projectId: string) {
  const project = await this.projectRepo.findOne(projectId);
  const nextNumber = (project.issueCounter ?? 0) + 1; // Step 1: read
  const issueKey = `${project.key}-${nextNumber}`;
  
  const issue = this.issueRepo.create({
    ...dto,
    key: issueKey,
  });
  await this.issueRepo.save(issue); // Step 2: write
  await this.projectRepo.update(projectId, { issueCounter: nextNumber }); // Step 3: write
  // Nếu 2 request cùng lúc ở Step 1 -> trùng nextNumber
}

// ✅ Atomic (transaction)
async createIssue(dto: CreateIssueDto, projectId: string) {
  return await this.dataSource.transaction(async (manager) => {
    // Lock project row để đọc-ghi atomic
    const project = await manager.findOne(
      Project,
      { where: { id: projectId } },
      { lock: { mode: 'pessimistic_write' } }, // Khóa ghi — request khác chờ
    );
    
    const nextNumber = (project.issueCounter ?? 0) + 1;
    const issueKey = `${project.key}-${nextNumber}`;
    
    const issue = manager.create(Issue, {
      ...dto,
      key: issueKey,
      projectId,
    });
    await manager.save(issue);
    
    project.issueCounter = nextNumber;
    await manager.save(project);
    
    // Nếu có exception -> toàn bộ rollback (issue + project counter đều lùi lại)
    return issue;
  });
}

// Hoặc dùng database sequence (tốt hơn)
// Postgres: CREATE SEQUENCE projects_issues_seq;
// Issue entity: @Column() key: string = `${project.key}-${nextval('projects_issues_seq')}`
```

Loại lock:
- **pessimistic_write**: chặn cứng — request thứ 2 chờ request thứ 1 xong mới đọc
- **optimistic**: không chặn, nhưng kiểm tra version column — nếu version đổi → conflict, rollback

```ts
// Optimistic lock — phù hợp high-concurrency
@Entity()
export class Issue {
  @PrimaryGeneratedColumn('uuid')
  id: string;
  
  @Column()
  key: string;
  
  @Version() // TypeORM magic: tăng 1 mỗi save
  version: number;
}

// Service
async createIssue(dto) {
  return await this.dataSource.transaction(async (manager) => {
    const project = await manager.findOne(Project, projectId);
    const nextNumber = (project.issueCounter ?? 0) + 1;
    project.issueCounter = nextNumber;
    
    try {
      await manager.save(project); // save thất bại nếu version đổi
    } catch (err) {
      if (err.message.includes('version')) {
        throw new ConflictException('Issue counter changed, retry');
      }
    }
    // Continue...
  });
}
```

Ví dụ thực tế: Jira Clone, khi tạo issue ở project "BACKEND", phải đảm bảo issue key là PROJECT-N (không trùng). Nếu 100 request cùng lúc tạo issue → transaction + lock giữ thứ tự → PROJECT-1, PROJECT-2, ..., PROJECT-100 (không trùng). Nếu không transaction → chaos.
</details>

<details>
<summary><b>26.7 Many-to-Many relationship với join table — khi nào cần explicit join table (vs implicit)? Dữ liệu lưu ở đâu?</b></summary>

**Implicit join table** (TypeORM tự tạo): dùng khi **không cần dữ liệu thêm** trong bảng nối. VD: post có nhiều tag, tag thuộc nhiều post → bảng `posts_tags` chỉ chứa `postId, tagId`.

**Explicit join table** (tạo entity riêng): dùng khi **join table có dữ liệu riêng**. VD: trong Jira Clone, issue có nhiều user (assignees, watchers, subscribers), và mỗi liên kết cần lưu `assignedAt`, `role` (PRIMARY, SECONDARY), `email_notification_enabled`... → tạo entity `IssueAssignee` riêng.

```ts
// ❌ Implicit (không đủ dữ liệu)
@Entity()
export class Post {
  @ManyToMany(() => Tag, (t) => t.posts, { cascade: true })
  @JoinTable()
  tags: Tag[];
}

// ✅ Explicit (dữ liệu phong phú)
// 1. Join entity
@Entity()
export class IssueAssignee {
  @PrimaryGeneratedColumn('uuid')
  id: string;
  
  @ManyToOne(() => Issue, (i) => i.assignees, { onDelete: 'CASCADE' })
  @JoinColumn()
  issue: Issue;
  
  @ManyToOne(() => User)
  @JoinColumn()
  user: User;
  
  @Column({ type: 'enum', enum: AssigneeRole, default: AssigneeRole.DEVELOPER })
  role: AssigneeRole; // PRIMARY, SECONDARY
  
  @Column({ default: true })
  emailNotificationEnabled: boolean;
  
  @CreateDateColumn()
  assignedAt: Date;
}

// 2. Issue entity
@Entity()
export class Issue {
  @OneToMany(() => IssueAssignee, (ia) => ia.issue, { cascade: true })
  assignees: IssueAssignee[];
}

// 3. Sử dụng
const issue = await this.issueRepo.findOne(issueId, {
  relations: ['assignees', 'assignees.user'],
});

// Access liên kết chi tiết
issue.assignees.forEach(ia => {
  console.log(`${ia.user.name} (${ia.role}) - Email: ${ia.emailNotificationEnabled}`);
});

// Add assignee với dữ liệu
const assignee = this.issueAssigneeRepo.create({
  issue,
  user: { id: userId },
  role: AssigneeRole.PRIMARY,
  emailNotificationEnabled: true,
});
await this.issueAssigneeRepo.save(assignee);
```

**Dữ liệu lưu ở đâu**:
- Implicit: bảng `posts_tags` (2 cột: `postId`, `tagId`)
- Explicit: bảng `issue_assignee` (multiple cột: `id`, `issueId`, `userId`, `role`, `emailNotificationEnabled`, `assignedAt`)

```ts
// Query lọc theo dữ liệu join table
const primaryAssignees = await this.issueAssigneeRepo.find({
  where: {
    issue: { id: issueId },
    role: AssigneeRole.PRIMARY,
  },
  relations: ['user'],
});

// Update dữ liệu join table
await this.issueAssigneeRepo.update(
  { id: assigneeId },
  { emailNotificationEnabled: false },
);
```

Ví dụ thực tế: Jira Clone, issue "PROJ-123" được gán cho user A (PRIMARY role, email=yes) và user B (SECONDARY role, email=no). Khi user A đề cập issue → thông báo email; user B không. Dữ liệu này lưu trong `issue_assignee`, không thể lưu ở bảng `users` hay `issues` đơn lẻ → cần explicit join table.
</details>

<details>
<summary><b>26.8 Exception Filter vs Global Exception Filter (@Catch) — khi nào dùng custom filter? Thứ tự xử lý nếu có nhiều filter?</b></summary>

**Exception Filter** bắt exception ném ra từ handler/service/pipe/interceptor, định hình response. Có thể tạo **custom filter** cho exception cụ thể hoặc **global filter** cho mọi exception.

Khi nào tạo custom filter:
1. Exception loại khác nhau cần response khác nhau (vd `ValidationException` → 400 + `errors` array; `NotFoundException` → 404 + `message`)
2. Cần log exception đặc biệt (vd gửi lên Sentry, ghi audit)
3. Exception không phải `HttpException` (vd `TypeormError`) → cần chuyển thành HTTP response

```ts
// 1. Custom filter cho TypeORM errors
@Catch(QueryFailedError) // QueryFailedError = DB constraint violation
export class DatabaseExceptionFilter implements ExceptionFilter {
  constructor(
    private readonly logger: Logger,
    private readonly sentryService: SentryService,
  ) {}
  
  catch(exception: QueryFailedError, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const { code, message } = exception.driverError;
    
    // Log + Sentry
    this.logger.error('Database error', { code, message });
    this.sentryService.captureException(exception);
    
    let statusCode = 500;
    let responseBody: any = { message: 'Internal server error' };
    
    // Handle constraint violations
    if (code === '23505') { // Unique violation
      statusCode = 409;
      responseBody = { message: 'Resource already exists', error: 'CONFLICT' };
    } else if (code === '23503') { // Foreign key violation
      statusCode = 400;
      responseBody = { message: 'Referenced resource not found', error: 'BAD_REQUEST' };
    }
    
    response.status(statusCode).json(responseBody);
  }
}

// 2. Global filter cho mọi HttpException
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  constructor(private readonly logger: Logger) {}
  
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    const status = exception.getStatus();
    const responseBody = exception.getResponse();
    
    // Chuẩn hóa format
    const errorResponse = {
      statusCode: status,
      message: typeof responseBody === 'string' ? responseBody : responseBody['message'],
      error: typeof responseBody === 'string' ? null : responseBody['error'],
      timestamp: new Date().toISOString(),
    };
    
    this.logger.warn(`HTTP Exception [${status}]`, errorResponse);
    response.status(status).json(errorResponse);
  }
}

// 3. Global fallback filter (bắt mọi exception không dự tính)
@Catch() // Không có tham số = bắt mọi exception
export class GlobalExceptionFilter implements ExceptionFilter {
  constructor(
    private readonly logger: Logger,
    private readonly sentryService: SentryService,
  ) {}
  
  catch(exception: unknown, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse();
    
    this.logger.error('Unhandled exception', exception);
    this.sentryService.captureException(exception);
    
    response.status(500).json({
      statusCode: 500,
      message: 'Internal server error',
      timestamp: new Date().toISOString(),
    });
  }
}

// Đăng ký filters ở app.module.ts
providers: [
  { provide: APP_FILTER, useClass: DatabaseExceptionFilter },
  { provide: APP_FILTER, useClass: HttpExceptionFilter },
  { provide: APP_FILTER, useClass: GlobalExceptionFilter },
],
```

**Thứ tự xử lý** (nếu có nhiều filter):
- Filter được đăng ký **từ trước → sau** (đơn giản hóa: đăng ký trước → chạy trước)
- Exception match loại nào → filter đó bắt + xử lý. **Nếu filter không throw lại → không filter khác chạy**, response đã gửi xong.
- Vì vậy, `@Catch()` (toàn thể) phải **sau cùng** để là fallback.

```ts
// Thứ tự chạy
// 1. DatabaseExceptionFilter (@Catch(QueryFailedError))
// 2. HttpExceptionFilter (@Catch(HttpException))
// 3. GlobalExceptionFilter (@Catch()) <- fallback

// Ví dụ:
// - Throw QueryFailedError → DatabaseExceptionFilter bắt → 409 → dừng
// - Throw BadRequestException → HttpExceptionFilter bắt → 400 → dừng
// - Throw Error không quen → GlobalExceptionFilter bắt → 500 → dừng
```

Ví dụ thực tế: Jira Clone, khi tạo issue, nếu violate unique constraint (vd issue key trùng) → DatabaseExceptionFilter bắt → 409 Conflict + message rõ. Nếu workspace không tồn tại → NotFoundException → HttpExceptionFilter → 404. Nếu lỗi bất ngờ (vd out of memory) → GlobalExceptionFilter → 500 + Sentry alert.
</details>

---

*Cập nhật lần cuối: bổ sung §14–§26 (Prisma, Advanced Auth, Web Security, Observability, Advanced NestJS, Transactions & Concurrency, Advanced Node, TypeScript, Testing, System Design, API/REST, Scheduled Jobs, Worked Examples) — bám sát dự án Jira Clone. File này được bổ sung sau mỗi section mới.*
---

## 27. GraphQL và Realtime trong NestJS

<details>
<summary><b>27.1 So sánh code-first và schema-first trong @nestjs/graphql? Bạn chọn cách nào và vì sao?</b></summary>

`@nestjs/graphql` hỗ trợ hai cách xây dựng schema:

- **Code-first**: bạn viết TypeScript class kèm decorator (`@ObjectType`, `@Field`, `@InputType`...), và package sẽ tự **generate SDL** (`schema.gql`) từ metadata. Single source of truth là code TypeScript.
- **Schema-first**: bạn viết file `.graphql` (SDL) trước, rồi `@nestjs/graphql` dùng `ts-morph` để **generate ra TypeScript types/interfaces** mà resolver phải implement.

| Tiêu chí | Code-first | Schema-first |
|---|---|---|
| Source of truth | TypeScript class | File `.graphql` |
| Type safety | Tốt, type sinh thẳng từ class | Phải generate type rồi implement đúng |
| Hợp với team | BE thuần TypeScript | Team có cả FE/contract-first, nhiều ngôn ngữ |
| Rủi ro | SDL có thể "lệch" mong đợi nếu quên decorator | Phải đồng bộ tay giữa SDL và resolver |

```ts
// Code-first: schema sinh từ class
@ObjectType()
export class Issue {
  @Field(() => ID)
  id: string;

  @Field()
  title: string;

  @Field(() => IssueStatus) // enum đã đăng ký bằng registerEnumType
  status: IssueStatus;
}

// Cấu hình bật code-first (autoSchemaFile)
GraphQLModule.forRoot<ApolloDriverConfig>({
  driver: ApolloDriver,
  autoSchemaFile: join(process.cwd(), 'src/schema.gql'), // bật code-first
  sortSchema: true,
});
```

Trade-off: tôi mặc định chọn **code-first** cho dự án NestJS thuần TypeScript vì tránh trùng lặp định nghĩa (chỉ viết class một lần) và type an toàn end-to-end. Chỉ chọn schema-first khi contract SDL phải được nhiều team/ngôn ngữ chia sẻ và thống nhất trước (contract-first).

Ví dụ thực tế: trong Jira Clone tôi dùng code-first — entity và GraphQL type dùng chung decorator nên thêm một field chỉ cần `@Field()` là schema tự cập nhật.
</details>

<details>
<summary><b>27.2 Resolver trong GraphQL là gì? Phân biệt @Query, @Mutation, @ResolveField và @Args/@Parent?</b></summary>

**Resolver** là class (đánh dấu `@Resolver(() => Type)`) chứa các hàm "phân giải" dữ liệu cho schema — tương tự controller trong REST nhưng map theo field của schema thay vì theo route.

- `@Query`: khai báo một entry point đọc dữ liệu (tương đương GET).
- `@Mutation`: khai báo entry point thay đổi dữ liệu (tương đương POST/PATCH/DELETE).
- `@ResolveField`: phân giải **một field con** của một type — chạy khi client thực sự chọn field đó. Đây là chỗ resolve quan hệ lồng nhau (ví dụ `Issue.assignee`).
- `@Args`: lấy argument của query/mutation; `@Parent`: lấy object cha đang được phân giải (dùng trong `@ResolveField`).

```ts
@Resolver(() => Issue)
export class IssueResolver {
  constructor(
    private readonly issueService: IssueService,
    private readonly userService: UserService,
  ) {}

  @Query(() => [Issue])
  issues(@Args('projectId') projectId: string) {
    return this.issueService.findByProject(projectId);
  }

  @Mutation(() => Issue)
  createIssue(@Args('input') input: CreateIssueInput) {
    return this.issueService.create(input);
  }

  // Chỉ chạy khi client query field "assignee" bên trong issue
  @ResolveField(() => User, { nullable: true })
  assignee(@Parent() issue: Issue) {
    if (!issue.assigneeId) return null;
    return this.userService.findById(issue.assigneeId);
  }
}
```

Điểm cần nhớ: `@ResolveField` **lazy** — không truy vấn `assignee` nếu client không yêu cầu, đây là một lợi thế của GraphQL (chỉ lấy đúng dữ liệu cần). Nhưng nếu query trả về danh sách thì chính cơ chế này lại dễ sinh vấn đề N+1 (xem 27.3).

Ví dụ thực tế: trong Jira Clone, `IssueResolver` có `@ResolveField` cho `assignee`, `comments`, `subtasks` — client muốn vẽ board chỉ cần `id`, `title`, `status` thì không tốn query phụ cho assignee.
</details>

<details>
<summary><b>27.3 Vấn đề N+1 trong GraphQL phát sinh thế nào và DataLoader giải quyết ra sao?</b></summary>

Vấn đề **N+1** xảy ra khi query trả về một danh sách N item và mỗi item có một `@ResolveField` tự truy vấn DB. Lấy 1 query danh sách (1) + N query con cho từng item = **N+1 query**.

```graphql
query {
  issues {        # 1 query lấy 50 issue
    title
    assignee {    # mỗi issue gọi userService.findById -> 50 query nữa => 51 query
      name
    }
  }
}
```

**DataLoader** (thư viện `dataloader` của Facebook) giải quyết bằng hai cơ chế:
1. **Batching**: gom tất cả `.load(id)` được gọi trong cùng một tick của event loop thành **một** lời gọi `batchFn([id1, id2, ...])` → bạn chạy `WHERE id IN (...)` một lần.
2. **Caching theo request**: cùng một key trong một request chỉ load một lần.

```ts
// Loader phải tạo MỚI cho mỗi request (cache không được chia sẻ giữa user)
export function createUserLoader(userService: UserService) {
  return new DataLoader<string, User>(async (ids: readonly string[]) => {
    const users = await userService.findByIds([...ids]); // 1 query: WHERE id IN (...)
    const map = new Map(users.map((u) => [u.id, u]));
    // PHẢI trả về đúng thứ tự key đầu vào
    return ids.map((id) => map.get(id) ?? null);
  });
}

@ResolveField(() => User, { nullable: true })
assignee(@Parent() issue: Issue, @Context() ctx: { userLoader: DataLoader<string, User> }) {
  if (!issue.assigneeId) return null;
  return ctx.userLoader.load(issue.assigneeId); // gom batch -> 1 query
}
```

Hai lưu ý dễ sai: (1) DataLoader phải được tạo **per-request** (đặt trong `context` factory của GraphQLModule) để cache không rò rỉ giữa các user; (2) `batchFn` PHẢI trả về mảng cùng độ dài và **đúng thứ tự** với mảng key đầu vào, nếu không sẽ map sai dữ liệu.

Ví dụ thực tế: board Jira có 100 issue, mỗi issue có assignee → không DataLoader là 101 query; bật DataLoader còn 2 query (1 issue + 1 user IN).
</details>

<details>
<summary><b>27.4 GraphQL Subscription hoạt động thế nào trong NestJS và khi nào nên dùng?</b></summary>

**Subscription** là loại operation thứ ba (sau query/mutation) cho phép **server đẩy dữ liệu** cho client khi có sự kiện, chạy trên **WebSocket** (giao thức `graphql-ws`). Trong code-first, bạn khai báo `@Subscription` và dùng một `PubSub` engine để publish/subscribe sự kiện.

```ts
const pubSub = new PubSub();

@Resolver(() => Comment)
export class CommentResolver {
  @Mutation(() => Comment)
  async addComment(@Args('input') input: AddCommentInput) {
    const comment = await this.commentService.create(input);
    // Phát sự kiện cho mọi subscriber đang nghe
    await pubSub.publish('commentAdded', { commentAdded: comment });
    return comment;
  }

  @Subscription(() => Comment, {
    // Lọc: chỉ đẩy comment thuộc đúng issue mà client subscribe
    filter: (payload, variables) => payload.commentAdded.issueId === variables.issueId,
  })
  commentAdded(@Args('issueId') issueId: string) {
    return pubSub.asyncIterator('commentAdded');
  }
}
```

Khi nào dùng: cần real-time đẩy theo schema GraphQL (live comment, thông báo, cập nhật trạng thái) mà vẫn muốn client chọn đúng field như query thường.

Cảnh báo production: `PubSub` mặc định chạy **in-memory** nên chỉ phát trong một process — khi scale nhiều instance, subscriber ở instance A sẽ không nhận sự kiện publish ở instance B. Phải thay bằng backend phân tán như `graphql-redis-subscriptions` (`RedisPubSub`) để các instance chia sẻ kênh qua Redis (tương tự lý do dùng Redis adapter cho WebSocket ở 27.7).
</details>

<details>
<summary><b>27.5 @WebSocketGateway và @SubscribeMessage trong NestJS hoạt động ra sao? Vòng đời kết nối?</b></summary>

NestJS có hỗ trợ WebSocket riêng (độc lập với GraphQL subscription), mặc định dùng **Socket.IO**.

- `@WebSocketGateway(port?, options)`: đánh dấu class là gateway — điểm nhận kết nối WebSocket. Có thể đặt `namespace`, `cors`.
- `@WebSocketServer()`: inject instance server (Socket.IO `Server`) để broadcast.
- `@SubscribeMessage('event')`: handler cho một message/event do client emit lên.
- Lifecycle hooks: `OnGatewayInit` (`afterInit`), `OnGatewayConnection` (`handleConnection`), `OnGatewayDisconnect` (`handleDisconnect`).

```ts
@WebSocketGateway({ namespace: 'board', cors: { origin: '*' } })
export class BoardGateway
  implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer() server: Server;

  handleConnection(client: Socket) {
    // Có thể xác thực token ở đây (lấy từ handshake.auth)
    console.log('connected', client.id);
  }

  handleDisconnect(client: Socket) {
    console.log('disconnected', client.id);
  }

  @SubscribeMessage('joinIssue')
  onJoin(@ConnectedSocket() client: Socket, @MessageBody() issueId: string) {
    client.join(`issue:${issueId}`); // room theo issue
  }

  // Gọi từ service khi có cập nhật -> đẩy cho mọi client trong room
  emitIssueUpdated(issueId: string, payload: unknown) {
    this.server.to(`issue:${issueId}`).emit('issueUpdated', payload);
  }
}
```

Điểm hữu ích: **room** của Socket.IO (`client.join` / `server.to(room).emit`) cho phép phát có chọn lọc — chỉ những client quan tâm tới một issue/board nhận event, thay vì broadcast toàn bộ.

Ví dụ thực tế: trong Jira Clone, khi ai đó kéo issue sang cột khác, service gọi `BoardGateway.emitIssueUpdated` → mọi người đang mở đúng board (room) thấy card di chuyển real-time.
</details>

<details>
<summary><b>27.6 Authentication cho WebSocket Gateway khác gì với HTTP? Triển khai thế nào?</b></summary>

WebSocket khác HTTP ở chỗ: chỉ có **một lần handshake** lúc kết nối, sau đó là kênh hai chiều giữ mở; không có header/guard chạy lại trên từng message như REST. Vì vậy chiến lược thường là:

1. Xác thực **một lần lúc handshake** (đọc token từ `handshake.auth.token` hoặc query), nếu sai thì `disconnect`.
2. Gắn user đã giải mã vào `client.data` để các handler dùng lại.

```ts
handleConnection(client: Socket) {
  try {
    const token = client.handshake.auth?.token;
    const payload = this.jwtService.verify(token); // ném lỗi nếu sai/het han
    client.data.user = payload; // dùng lại cho các @SubscribeMessage sau
  } catch {
    client.disconnect(true); // từ chối kết nối
  }
}
```

Có thể dùng `@UseGuards(WsAuthGuard)` cho từng `@SubscribeMessage`, nhưng phải nhớ: với WebSocket, `ExecutionContext` là `context.switchToWs()` chứ không phải `switchToHttp()`, và một exception trong gateway nên ném `WsException` (không phải `HttpException`) để client nhận đúng dạng lỗi qua sự kiện `exception`.

```ts
@Injectable()
export class WsAuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const client: Socket = context.switchToWs().getClient();
    if (!client.data.user) throw new WsException('Unauthorized');
    return true;
  }
}
```

Lưu ý bảo mật: với token hết hạn giữa phiên kết nối dài, handshake một lần không tự phát hiện — nếu cần chặt chẽ, nên kiểm tra lại token/quyền định kỳ hoặc trên các action nhạy cảm, và có cơ chế server chủ động `disconnect` khi token bị thu hồi.
</details>

<details>
<summary><b>27.7 Vì sao WebSocket "vỡ" khi scale nhiều instance và Redis adapter giải quyết thế nào?</b></summary>

Mỗi instance Socket.IO chỉ giữ kết nối và room **trong bộ nhớ của chính nó**. Khi chạy nhiều instance sau load balancer:

- Client A kết nối instance #1, client B kết nối instance #2.
- Khi instance #1 gọi `server.to('issue:123').emit(...)`, nó chỉ đẩy cho các client đang nối tới #1 → client B ở #2 **không nhận được**.

Đây là vấn đề broadcast xuyên-instance, không phải vấn đề sticky session đơn thuần.

**Redis adapter** (`@socket.io/redis-adapter`) giải quyết bằng cách cho các instance **publish/subscribe sự kiện qua Redis Pub/Sub**: instance #1 publish lên Redis, mọi instance (kể cả #2) nhận và đẩy tiếp cho client của mình → broadcast/room hoạt động xuyên suốt cluster.

```ts
// main.ts
const pubClient = createClient({ url: process.env.REDIS_URL });
const subClient = pubClient.duplicate();
await Promise.all([pubClient.connect(), subClient.connect()]);

// Adapter tùy biến để dùng với @WebSocketGateway
export class RedisIoAdapter extends IoAdapter {
  private adapterConstructor: ReturnType<typeof createAdapter>;
  async connectToRedis() {
    this.adapterConstructor = createAdapter(pubClient, subClient);
  }
  createIOServer(port: number, options?: ServerOptions) {
    const server = super.createIOServer(port, options);
    server.adapter(this.adapterConstructor);
    return server;
  }
}

const adapter = new RedisIoAdapter(app);
await adapter.connectToRedis();
app.useWebSocketAdapter(adapter);
```

Hai điểm cần nhớ thêm: (1) nếu transport rơi về long-polling, vẫn cần **sticky session** ở load balancer để các request HTTP của cùng một client về đúng instance; (2) Redis adapter chỉ lo **truyền sự kiện giữa các instance**, không lưu lại message — client offline rồi quay lại không tự nhận message cũ (cần cơ chế lưu/replay riêng nếu cần).

Ví dụ thực tế: Jira Clone chạy 3 pod trên Kubernetes; không có Redis adapter thì hai người mở cùng board nhưng rơi vào hai pod sẽ không thấy cập nhật của nhau — bật adapter là hết.
</details>

<details>
<summary><b>27.8 Khi nào chọn REST, GraphQL hay WebSocket? Tiêu chí quyết định?</b></summary>

Ba công cụ giải ba bài toán khác nhau; thực tế một hệ thống thường dùng cả ba.

| Tiêu chí | REST | GraphQL | WebSocket |
|---|---|---|---|
| Mô hình | Request/response, resource-based | Request/response, query có chọn field | Hai chiều, server đẩy |
| Hợp với | CRUD, public API, cache HTTP đơn giản | Client cần lấy chính xác dữ liệu, nhiều quan hệ lồng nhau, tránh over/under-fetching | Real-time: chat, notification, live board, collaborative editing |
| Caching | Dễ (HTTP cache, CDN theo URL) | Khó hơn (thường POST tới 1 endpoint) | Không phải mô hình cache |
| Rủi ro | Over-fetch/under-fetch, nhiều round-trip | N+1, query phức tạp tốn tài nguyên, khó rate-limit theo endpoint | Quản lý kết nối, scale (cần Redis adapter), auth khác kiểu |

Hướng quyết định thực dụng:
- **REST** khi API công khai, đơn giản, cần tận dụng HTTP caching/CDN và status code rõ ràng.
- **GraphQL** khi client (đặc biệt mobile/đa màn hình) cần lấy đúng tập field, dữ liệu có nhiều quan hệ và muốn giảm số round-trip — đổi lại phải xử lý N+1 và giới hạn độ phức tạp query.
- **WebSocket** khi cần đẩy real-time hoặc giao tiếp hai chiều có độ trễ thấp; nhưng nếu chỉ cần cập nhật thưa, có thể dùng SSE hoặc polling cho đơn giản.

Ví dụ thực tế: Jira Clone dùng REST/GraphQL cho CRUD issue và truy vấn board, còn WebSocket (Socket.IO) chỉ cho phần đồng bộ real-time khi kéo-thả issue và comment trực tiếp.
</details>

<details>
<summary><b>27.9 Bảo mật một GraphQL API ở production gồm những gì? (depth limit, query complexity, introspection)</b></summary>

GraphQL mở ra bề mặt tấn công mà REST không có: client tự ghép query nên một query lồng sâu hoặc vòng lặp quan hệ có thể bắt server làm việc rất nặng (DoS). Các lớp phòng thủ chính:

1. **Depth limit**: giới hạn độ sâu lồng nhau của query để chặn query đệ quy kiểu `user -> posts -> author -> posts -> ...`.
2. **Query complexity**: gán "chi phí" cho từng field và từ chối query vượt ngưỡng tổng — chính xác hơn depth vì tính cả số lượng field/list.
3. **Disable introspection ở production**: tắt khả năng client tải toàn bộ schema (introspection), giảm lộ cấu trúc API cho kẻ tấn công. Thường tắt luôn GraphQL Playground/sandbox ở prod.
4. Bổ sung: **rate limiting**, **timeout/pagination bắt buộc** trên list field, và **persisted queries** (chỉ cho phép các query đã được allowlist) cho client nội bộ.

```ts
GraphQLModule.forRoot<ApolloDriverConfig>({
  driver: ApolloDriver,
  autoSchemaFile: true,
  introspection: process.env.NODE_ENV !== 'production', // tắt ở prod
  playground: false,
  validationRules: [depthLimit(7)], // graphql-depth-limit: chặn lồng quá sâu
});
```

```ts
// Query complexity bằng plugin (graphql-query-complexity)
@Field(() => [Comment], { complexity: 10 }) // gán chi phí field nặng
comments: Comment[];

// plugin tính tổng complexity của query, vượt maximumComplexity -> reject
```

Lưu ý: tắt introspection chỉ là "giảm thông tin", không phải biện pháp bảo mật thay cho **authorization** ở từng resolver/field — vẫn phải kiểm tra quyền (guard, field-level auth) vì kẻ tấn công có thể đoán field. Đừng để lỗi resolver lộ stack trace; nên chuẩn hóa error formatting ở prod.

Ví dụ thực tế: ở Jira Clone, prod tôi đặt `introspection=false`, `playground=false`, depth limit 7 và complexity ngưỡng cố định để một query lồng comment-trong-issue-trong-project không thể quét cả DB.
</details>

<details>
<summary><b>27.10 (Hành vi) Kể về một lần bạn xử lý sự cố hiệu năng/độ ổn định liên quan tới GraphQL hoặc WebSocket?</b></summary>

Trả lời theo cấu trúc **STAR**:

- **Situation**: Trên Jira Clone, sau khi thêm tính năng board real-time, người dùng phản ánh hai người mở cùng một board nhưng không thấy cập nhật của nhau; đồng thời API GraphQL load board thỉnh thoảng chậm vọt lên vài giây khi project có nhiều issue.
- **Task**: Tôi cần (1) làm cho cập nhật real-time hoạt động xuyên các instance sau khi đã scale lên nhiều pod, và (2) hạ thời gian phản hồi của query board.
- **Action**: Với real-time, tôi xác định nguyên nhân là Socket.IO lưu room in-memory từng instance nên broadcast không xuyên pod; tôi tích hợp `@socket.io/redis-adapter` qua `useWebSocketAdapter` để các pod publish/subscribe qua Redis, và bật sticky session ở load balancer cho fallback long-polling. Với GraphQL chậm, tôi bật query logging và phát hiện N+1: mỗi issue gọi riêng một query lấy `assignee` và `comments`; tôi đưa DataLoader per-request vào `context` để gom batch thành `WHERE id IN (...)`. Tôi cũng siết depth limit và complexity để chặn query lồng quá sâu.
- **Result**: Cập nhật board hiển thị real-time ổn định trên cả 3 pod; số query cho một board từ hơn 100 giảm còn 2-3, thời gian phản hồi p95 giảm rõ rệt. Bài học của tôi là khi dùng GraphQL/WebSocket phải tính tới scale (Redis adapter) và N+1 (DataLoader) ngay từ thiết kế, không để thành sự cố production.
</details>

---

*Cập nhật lần cuối: bổ sung §27 (GraphQL và Realtime trong NestJS — code-first vs schema-first, resolver/@ResolveField, N+1 & DataLoader, subscription, WebSocket Gateway/Socket.IO, Redis adapter, REST vs GraphQL vs WebSocket, bảo mật GraphQL) — bám sát dự án Jira Clone. File này được bổ sung sau mỗi section mới.*
---

## 28. Tư duy và Giải quyết vấn đề (Backend)

<details>
<summary><b>28.1 Khi một API trên production đột nhiên chậm/lỗi mà không deploy gì mới, bạn tiếp cận điều tra như thế nào?</b></summary>

Nguyên tắc: đừng đoán mò, hãy đi từ **triệu chứng → dữ liệu quan sát → giả thuyết → kiểm chứng**. Một backend "observable" tốt phải có 3 trụ: **logs** (có cấu trúc), **metrics** (số liệu tổng hợp) và **traces** (distributed tracing). Quy trình thực dụng:

1. **Xác định phạm vi (scope)**: lỗi xảy ra toàn bộ hay chỉ một endpoint? Một region? Một nhóm user? Nhìn dashboard metrics trước (RED method: **R**ate, **E**rrors, **D**uration).
2. **Dựng timeline**: cái gì thay đổi quanh thời điểm sự cố? Không deploy code không có nghĩa là không có gì đổi — có thể là traffic tăng, một feature flag bật, dữ liệu một bảng phình to, cron job chạy, hay cloud provider degrade.
3. **Khoanh vùng tầng**: app, DB, cache (Redis), queue (BullMQ), hay dependency bên ngoài? Trace sẽ chỉ ra span nào ngốn thời gian.
4. **Xác nhận giả thuyết bằng dữ liệu**, sửa, rồi viết postmortem (blameless).

Các thủ phạm "kinh điển" không-cần-deploy: connection pool cạn (DB hoặc Redis), một bảng lớn dần làm query mất index hiệu lực, N+1 chỉ lộ khi data tăng, memory leak tích lũy, lock/deadlock, hoặc một downstream service chậm làm nghẽn cả pool.

```ts
// Bắt slow request ngay tại tầng app để có số liệu duration + correlation id
@Injectable()
export class TimingInterceptor implements NestInterceptor {
  private readonly logger = new Logger('Timing');
  intercept(ctx: ExecutionContext, next: CallHandler) {
    const req = ctx.switchToHttp().getRequest();
    const start = process.hrtime.bigint();
    return next.handle().pipe(
      tap(() => {
        const ms = Number(process.hrtime.bigint() - start) / 1e6;
        if (ms > 500) {
          this.logger.warn({
            msg: 'slow_request',
            method: req.method,
            path: req.route?.path,
            durationMs: Math.round(ms),
            correlationId: req.headers['x-correlation-id'],
          });
        }
      }),
    );
  }
}
```

Ví dụ thực tế: trong Jira Clone, endpoint `GET /boards/:id/issues` chậm dần theo thời gian. Trace cho thấy 95% thời gian nằm ở DB; bật log query (Prisma `log: ['query']`) lộ ra một loạt query con — đó là N+1 khi load `assignee` cho từng issue. Fix bằng `include` thay vì lặp.

</details>

<details>
<summary><b>28.2 Correlation ID / request ID dùng để làm gì, và bạn truyền nó xuyên suốt một request NestJS như thế nào?</b></summary>

**Correlation ID** (hay request ID, trace ID) là một định danh duy nhất gắn vào mỗi request đầu vào, được **truyền qua mọi log, mọi service, mọi job** mà request đó sinh ra. Mục đích: khi điều tra một sự cố cụ thể của một user, bạn `grep` đúng một ID là gom được toàn bộ chuỗi xử lý — thay vì lội qua hàng triệu dòng log rời rạc. Đây là điều kiện sống còn trong hệ phân tán/microservice.

Cách làm trong NestJS: dùng **middleware** sinh/đọc ID từ header (`X-Correlation-Id`), lưu vào **AsyncLocalStorage** để mọi nơi trong cùng request truy cập được mà không cần truyền tham số thủ công, rồi inject vào logger.

```ts
// als.ts
export const als = new AsyncLocalStorage<{ correlationId: string }>();

// middleware
export function correlationMiddleware(req, res, next) {
  const id = req.headers['x-correlation-id'] ?? randomUUID();
  res.setHeader('x-correlation-id', id);
  als.run({ correlationId: id }, () => next());
}

// logger tự động đính kèm id
function log(level: string, msg: object) {
  const store = als.getStore();
  console.log(JSON.stringify({ level, ...msg, correlationId: store?.correlationId }));
}
```

Lưu ý quan trọng: khi đẩy việc sang **BullMQ**, phải nhét `correlationId` vào job data để worker tiếp tục log cùng ID — nếu không, chuỗi log sẽ "đứt" ở ranh giới async/queue. Với chuẩn **OpenTelemetry**, khái niệm tương ứng là `traceId` truyền qua header `traceparent` (W3C Trace Context), và các thư viện auto-instrumentation sẽ tự nối span giữa các service.

</details>

<details>
<summary><b>28.3 Một query bị chậm. Quy trình bạn dùng để chẩn đoán và tối ưu nó là gì?</b></summary>

Quy trình chuẩn: **đo → đọc plan → sửa → đo lại**.

1. **Bật query logging** để biết chính xác SQL nào chậm và tần suất (Prisma: `log: ['query']`; TypeORM: `logging: true`, `maxQueryExecutionTime`).
2. **Chạy `EXPLAIN ANALYZE`** trên Postgres — đây là công cụ quyết định. Đọc xem có **Seq Scan** trên bảng lớn không, ước lượng rows (`estimated`) có lệch nhiều với thực tế (`actual`) không (lệch lớn = thống kê cũ, cần `ANALYZE`), và đâu là node tốn thời gian nhất.
3. **Sửa**, theo thứ tự ưu tiên: thêm/chỉnh **index** đúng cột (gồm composite index theo thứ tự cột trong `WHERE`/`ORDER BY`), viết lại query (tránh function trên cột làm vô hiệu index), giảm dữ liệu trả về, hoặc denormalize/cache nếu cần.

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM issues
WHERE project_id = 42 AND status = 'OPEN'
ORDER BY created_at DESC
LIMIT 20;
-- Trước: Seq Scan on issues (actual time=120ms) -> rất chậm
-- Sau khi thêm index phù hợp:
CREATE INDEX idx_issues_project_status_created
  ON issues (project_id, status, created_at DESC);
-- Index Scan, actual time < 2ms
```

| Triệu chứng trong plan | Nguyên nhân thường gặp | Cách xử lý |
|---|---|---|
| Seq Scan trên bảng lớn | Thiếu index, hoặc `WHERE` dùng function trên cột | Thêm index; viết lại điều kiện sargable |
| estimated rows ≠ actual rows | Thống kê lỗi thời | `ANALYZE table` |
| Nested loop với nhiều vòng | JOIN thiếu index ở cột khóa | Index cột FK |
| Sort tốn thời gian | `ORDER BY` không khớp index | Index có cột sort, đúng chiều |

Bẫy thường gặp: index không phải lúc nào cũng tốt — nó làm chậm `INSERT/UPDATE` và tốn dung lượng; trên bảng nhỏ planner có thể cố tình Seq Scan vì rẻ hơn. Đừng thêm index bừa, hãy đo.

</details>

<details>
<summary><b>28.4 N+1 query là gì, vì sao nó "ẩn" cho tới khi data lớn, và cách phát hiện/khắc phục với Prisma, TypeORM và GraphQL?</b></summary>

**N+1** là khi bạn chạy 1 query để lấy danh sách N bản ghi, rồi chạy thêm N query con để lấy quan hệ của từng bản ghi → tổng cộng N+1 round-trip tới DB. Trên môi trường dev với 5 record nó vẫn "nhanh", nên dễ lọt qua review; lên production với 5000 record nó thành 5001 query và sập độ trễ. Đây là lý do nó ẩn — vấn đề tỉ lệ với kích thước data, không lộ ở dữ liệu seed nhỏ.

Phát hiện: bật query log và đếm số query trên một request; nếu thấy cùng một mẫu query lặp lại nhiều lần → gần như chắc chắn N+1.

```ts
// XẤU: N+1 — mỗi issue lại query author một lần
const issues = await prisma.issue.findMany({ where: { projectId } });
for (const i of issues) {
  i.author = await prisma.user.findUnique({ where: { id: i.authorId } }); // N query
}

// TỐT (Prisma): một query duy nhất với include (eager load)
const issues = await prisma.issue.findMany({
  where: { projectId },
  include: { author: true },
});
```

- **TypeORM**: dùng `relations: ['author']` hoặc `leftJoinAndSelect` trong QueryBuilder thay vì lazy access trong vòng lặp.
- **GraphQL** (`@nestjs/graphql`): N+1 sinh ra ở **field resolver** chạy cho từng phần tử của list. Giải pháp chuẩn là **DataLoader** — gom các key trong cùng một tick rồi batch thành 1 query `WHERE id IN (...)`, đồng thời cache trong vòng đời request.

```ts
const userLoader = new DataLoader<string, User>(async (ids) => {
  const users = await prisma.user.findMany({ where: { id: { in: [...ids] } } });
  const map = new Map(users.map((u) => [u.id, u]));
  return ids.map((id) => map.get(id)); // giữ đúng thứ tự key
});
```

Trade-off: eager load mọi thứ cũng có thể over-fetch (kéo cột/quan hệ không dùng tới). Nguyên tắc: chỉ `include`/`select` đúng cái cần.

</details>

<details>
<summary><b>28.5 Khi đứng trước một quyết định kiến trúc, bạn cân nhắc trade-off theo khung tư duy nào?</b></summary>

Không có kiến trúc "đúng tuyệt đối", chỉ có kiến trúc **phù hợp với ràng buộc hiện tại**. Khung tư duy tôi dùng:

1. **Làm rõ yêu cầu thật**: functional (làm được gì) và non-functional (throughput, latency mục tiêu, độ sẵn sàng, tính nhất quán, dung lượng dữ liệu, ngân sách, quy mô team). Phần lớn quyết định kiến trúc bị chi phối bởi non-functional.
2. **Liệt kê 2-3 phương án** thay vì cưới ngay phương án đầu.
3. **Đánh giá theo các trục trade-off**: độ phức tạp vận hành, chi phí, khả năng mở rộng, tốc độ ship, khả năng đảo ngược (reversibility).
4. **Ưu tiên quyết định dễ đảo ngược thì làm nhanh; quyết định khó đảo (one-way door) thì cẩn trọng** (theo cách phân loại của Amazon). Chọn DB chính, định dạng API public, sơ đồ phân chia service là khó đảo.
5. **Ghi lại bằng ADR (Architecture Decision Record)**: bối cảnh, phương án, quyết định, hệ quả — để 6 tháng sau team hiểu *vì sao*.

Một định luật nền tảng cần nhớ: **CAP theorem** (khi network partition, phải chọn giữa Consistency và Availability) và rộng hơn là **PACELC** (kể cả khi không partition, vẫn có trade-off Latency vs Consistency). Mọi "magic" như cache, replica, queue đều mua hiệu năng/độ sẵn sàng bằng cái giá là tính nhất quán hoặc độ phức tạp.

Anti-pattern cần tránh: **resume-driven development** (chọn Kafka/microservice/k8s vì nó "ngầu" chứ không vì bài toán cần), và **premature optimization** cho quy mô chưa tồn tại.

Ví dụ thực tế: với Blog/CMS API ở giai đoạn đầu (vài nghìn user), một **modular monolith** NestJS + Postgres + Redis cache là đủ; tách microservice lúc này chỉ tăng độ phức tạp vận hành mà không giải quyết vấn đề nào có thật.

</details>

<details>
<summary><b>28.6 Chọn SQL hay NoSQL như thế nào? Tiêu chí và cạm bẫy.</b></summary>

Quyết định nên dựa trên **hình dạng dữ liệu, mẫu truy vấn và yêu cầu nhất quán**, không phải theo trend.

| Tiêu chí | Nghiêng về SQL (Postgres/MySQL) | Nghiêng về NoSQL (MongoDB, DynamoDB, Cassandra...) |
|---|---|---|
| Quan hệ giữa entity | Nhiều quan hệ, cần JOIN | Dữ liệu dạng document tự chứa, ít JOIN |
| Giao dịch | Cần ACID, transaction nhiều bảng | Chấp nhận eventual consistency |
| Schema | Ổn định, cần ràng buộc toàn vẹn | Linh hoạt, thay đổi thường xuyên |
| Mẫu truy vấn | Đa dạng, ad-hoc, aggregation phức tạp | Biết trước access pattern, key-based |
| Quy mô ghi | Vertical + read replica thường đủ | Cần horizontal scale ghi cực lớn |

Điểm thường bị hiểu sai:
- **"NoSQL nhanh hơn"** không đúng tổng quát — nó nhanh *với đúng access pattern đã thiết kế cho nó*, và đánh đổi bằng việc khó query linh hoạt, dễ duplicate dữ liệu.
- **Postgres ngày nay đã rất linh hoạt**: có kiểu `JSONB` (index được, query được) để xử lý phần dữ liệu schema-less, `tsvector` cho full-text search. Nhiều bài toán tưởng "cần NoSQL" thực ra một cột `JSONB` trong Postgres là đủ.
- Lựa chọn an toàn cho **đa số app CRUD/giao dịch**: bắt đầu với **Postgres**. Chỉ chuyển/bổ sung NoSQL khi có vấn đề cụ thể (ví dụ: lưu event log khổng lồ, time-series, hoặc cache document).

```sql
-- Postgres vừa quan hệ vừa schema-less: cột JSONB cho metadata tùy biến
CREATE TABLE issue (
  id           bigint PRIMARY KEY,
  project_id   bigint REFERENCES project(id),  -- vẫn giữ tính toàn vẹn quan hệ
  custom_fields jsonb                            -- linh hoạt như NoSQL
);
CREATE INDEX idx_issue_custom ON issue USING gin (custom_fields);
```

Ví dụ thực tế: Jira Clone cần transaction (đổi status + ghi history + cập nhật sprint phải nguyên tử) và nhiều quan hệ (project → board → issue → comment) → Postgres + Prisma là lựa chọn đúng; `custom_fields` của issue thì để JSONB.

</details>

<details>
<summary><b>28.7 Hãy mô tả cách thực hiện một data migration lớn với zero-downtime (ví dụ: đổi tên cột, tách bảng, đổi kiểu dữ liệu).</b></summary>

Migration lớn nguy hiểm vì code cũ và schema mới phải **cùng tồn tại** trong lúc deploy rolling. Mẫu chuẩn là **expand → migrate/backfill → contract** (còn gọi là parallel change):

1. **Expand (mở rộng, tương thích ngược)**: thêm cấu trúc mới mà *không* xóa cái cũ. Ví dụ thêm cột mới `full_name`, để cột cũ `name` nguyên vẹn. Migration này không phá code đang chạy.
2. **Double-write**: deploy code ghi vào *cả* cấu trúc cũ và mới (hoặc dùng trigger DB). Giờ mọi ghi mới đều nhất quán hai bên.
3. **Backfill**: chạy job nền điền dữ liệu cũ sang cấu trúc mới theo **batch** (ví dụ 1000 dòng/lần, có nghỉ giữa batch) để không khóa bảng/quá tải DB. Backfill phải **idempotent** và có thể resume.
4. **Switch read**: deploy code đọc từ cấu trúc mới, xác minh số liệu khớp.
5. **Contract (thu hẹp)**: sau khi chắc chắn ổn định, ngừng ghi cũ rồi mới `DROP` cột/bảng cũ ở một migration sau.

```sql
-- Bước 1: Expand
ALTER TABLE users ADD COLUMN full_name varchar(255);

-- Bước 3: Backfill theo batch (chạy lặp tới khi hết)
UPDATE users SET full_name = first_name || ' ' || last_name
WHERE full_name IS NULL
  AND id IN (SELECT id FROM users WHERE full_name IS NULL LIMIT 1000);

-- Bước 5: Contract (deploy sau, khi không còn ai đọc cột cũ)
ALTER TABLE users DROP COLUMN first_name, DROP COLUMN last_name;
```

Lưu ý kỹ thuật trên Postgres:
- `ADD COLUMN` có `DEFAULT` hằng số là tức thì từ PG 11+, nhưng tránh default tính toán/volatile vì có thể rewrite bảng.
- Tạo index trên bảng lớn phải dùng `CREATE INDEX CONCURRENTLY` (không khóa ghi); lưu ý nó không chạy trong transaction nên cần xử lý riêng (Prisma migrate cần `--create-only` rồi sửa SQL thủ công).
- `ALTER COLUMN TYPE` thường rewrite + khóa bảng → thay bằng pattern cột-mới + backfill.
- Luôn có **plan rollback** và chạy thử trên bản sao production có dữ liệu thật về kích thước.

</details>

<details>
<summary><b>28.8 Khi nào nên tách một microservice ra khỏi monolith, và khi nào KHÔNG nên?</b></summary>

Tư duy gốc: **microservice là giải pháp cho vấn đề tổ chức/quy mô, không phải mục tiêu tự thân**. Nó đánh đổi sự đơn giản (1 deploy, 1 transaction, gọi hàm in-process) lấy khả năng scale và phát triển độc lập — kèm theo *thuế phân tán*: network không tin cậy, eventual consistency, distributed tracing, deployment phức tạp.

Lý do **chính đáng** để tách:
- Một phần hệ thống có **profile mở rộng khác biệt** (ví dụ module xử lý ảnh/video ngốn CPU cần scale riêng với phần CRUD nhẹ).
- **Ranh giới team rõ ràng** muốn deploy độc lập (Conway's Law) khi tổ chức đã đủ lớn.
- Cần **cô lập lỗi/bảo mật** (ví dụ payment) hoặc khác biệt về stack/yêu cầu tuân thủ.

Khi **KHÔNG nên** (giữ monolith, tốt nhất là **modular monolith**):
- Team nhỏ, sản phẩm còn đang tìm product-market fit, domain boundary chưa rõ — tách sớm sẽ phải sửa biên giới service liên tục, rất đắt.
- Nhu cầu chỉ là "code sạch hơn" — cái này giải quyết bằng module hóa trong cùng codebase, không cần network ở giữa.

Lộ trình khôn ngoan: **Monolith → Modular Monolith → tách dần service khi đau thật**. Trong NestJS, modular monolith rất tự nhiên nhờ hệ thống **module** với biên giới rõ; khi cần tách, NestJS hỗ trợ `@nestjs/microservices` (transport TCP/Redis/NATS/RabbitMQ/Kafka). Bước trung gian an toàn: tách các tác vụ async nặng ra **worker BullMQ** riêng — đã có lợi ích scale độc lập mà chưa phải chẻ nhỏ data store.

Bẫy lớn nhất: **distributed monolith** — tách service nhưng chúng vẫn gọi đồng bộ chằng chịt và share database, nhận đủ nhược điểm của cả hai mô hình.

</details>

<details>
<summary><b>28.9 Bạn nhận một yêu cầu (ticket) mơ hồ, ví dụ "thêm export báo cáo cho admin". Bạn làm gì trước khi viết code?</b></summary>

Viết code dựa trên một yêu cầu mơ hồ là cách nhanh nhất để làm sai và phải làm lại. Tôi **làm rõ trước, code sau**, bằng cách đặt câu hỏi có cấu trúc:

1. **Làm rõ "what" và "why"**: ai dùng, để giải quyết vấn đề gì? Hiểu mục tiêu kinh doanh giúp tránh build sai trọng tâm. ("Export" để họ tự phân tích trong Excel hay để gửi cho khách?)
2. **Đào các yêu cầu ẩn (non-functional)**:
   - **Định dạng & phạm vi**: CSV/XLSX/PDF? Bao nhiêu cột? Lọc theo gì?
   - **Khối lượng**: 100 dòng hay 10 triệu dòng? Đây là câu hỏi quyết định kiến trúc — vài trăm dòng thì xuất đồng bộ ngay trong request; hàng triệu dòng thì phải đẩy job BullMQ, **stream** ra file, lưu storage rồi gửi link/email (tránh giữ HTTP request mở quá lâu và tránh ngốn RAM).
   - **Quyền hạn**: "admin" là role nào, có phân quyền theo tổ chức/tenant không?
   - **Dữ liệu nhạy cảm / PII**: có cần che, audit log ai đã export không?
3. **Xác định edge case**: không có dữ liệu? Timezone? Số lượng đồng thời?
4. **Chốt acceptance criteria** rồi mới ước lượng. Nếu yêu cầu vẫn quá rộng, đề xuất chia phase (MVP trước).

Một mẹo giao tiếp: thay vì hỏi mở "anh muốn gì", hãy **đề xuất giả định cụ thể để họ xác nhận**: "Em hiểu là export CSV các issue đã đóng trong tháng, lọc theo project, chạy nền và gửi link tải — đúng không ạ?" Câu hỏi đóng kiểu này lấy được quyết định nhanh hơn nhiều.

Ví dụ thực tế: với Jira Clone, "export report" hóa ra là báo cáo velocity theo sprint — nếu không hỏi, dễ build nhầm thành export raw toàn bộ issue.

</details>

<details>
<summary><b>28.10 Làm sao bạn ước lượng (estimate) và chia nhỏ một task lớn, ví dụ "xây dựng hệ thống notification"?</b></summary>

Ước lượng kém thường đến từ task quá to và mơ hồ. Nguyên tắc: **chia nhỏ tới mức mỗi phần có thể ship độc lập và ước lượng tự tin** (lý tưởng dưới 1-2 ngày/phần). Cách tôi làm:

1. **Vẽ ra các thành phần dọc theo luồng dữ liệu** thay vì theo tầng kỹ thuật. Với "notification system": (a) sự kiện kích hoạt (issue assigned, comment...), (b) cơ chế fan-out/queue, (c) các kênh gửi (in-app, email, push), (d) lưu trữ & trạng thái read/unread, (e) tùy chọn preference của user, (f) real-time đẩy về client.
2. **Xác định MVP vertical slice**: làm trọn vẹn *một* luồng end-to-end trước (ví dụ: chỉ in-app notification cho sự kiện "assigned", có queue + Socket.IO + đánh dấu đã đọc). Có thứ chạy được sớm tốt hơn nhiều mảnh dở dang.
3. **Đánh dấu unknown/spike**: phần nào chưa rõ kỹ thuật (ví dụ tích hợp APNs/FCM) thì tạo một **spike** ngắn để nghiên cứu trước, không ước lượng bừa.
4. **Ước lượng theo dải, cộng buffer cho rủi ro** (testing, code review, edge case, deploy), đừng đưa con số đơn lẻ.

```
Notification System  (ước lượng thô)
├─ 1. Event emitter trong domain (EventEmitter2)      ~0.5d
├─ 2. Queue + worker xử lý (BullMQ + Redis)            ~1d
├─ 3. Kênh in-app: lưu DB + API list/mark-read        ~1d
├─ 4. Real-time push (Socket.IO gateway + room/user)  ~1d
├─ 5. Kênh email (provider + template + retry)        ~1.5d  ← có rủi ro tích hợp
├─ 6. User preferences (bật/tắt từng loại)            ~1d
└─ 7. Test + tài liệu + observability                 ~1d
```

Kỹ thuật bổ trợ khi nhiều bất định: kích thước tương đối (story points) thay vì giờ tuyệt đối, và tham chiếu task tương tự đã làm. Quan trọng nhất là **cập nhật ước lượng khi biết thêm** — estimate là dự báo, không phải lời hứa cứng.

</details>

<details>
<summary><b>28.11 Bạn được giao maintain một codebase backend lớn, lạ, không có tài liệu. Bạn đọc hiểu nó như thế nào?</b></summary>

Không đọc tuần tự từng file — đó là cách lãng phí thời gian nhất với codebase lớn. Tôi đi từ **ngoài vào trong, từ luồng thực tới chi tiết**:

1. **Chạy được nó trước đã**: dựng môi trường local theo README/Docker, gọi vài endpoint. Hiểu cách build/test/run cho ta bản đồ thực tế hơn mọi tài liệu.
2. **Tìm các điểm vào (entry points)**: với NestJS là `main.ts`, `AppModule`, danh sách `controllers` và `imports`. Cây module chính là sơ đồ kiến trúc của ứng dụng.
3. **Trace một luồng end-to-end** cho một tính năng quan trọng: từ controller → service → repository/DB → response. Đặt breakpoint hoặc đọc log để xem dữ liệu chảy qua đâu. Hiểu sâu một luồng hơn là lướt qua mười luồng.
4. **Đọc schema DB** (Prisma schema hoặc TypeORM entity) — mô hình dữ liệu thường tiết lộ domain rõ hơn cả code, vì code thay đổi nhưng cấu trúc dữ liệu cốt lõi thì ổn định.
5. **Dùng test như tài liệu sống**: test mô tả hành vi mong đợi và cách dùng API nội bộ. Đọc test của một module để hiểu hợp đồng của nó.
6. **Khảo sát lịch sử git**: `git log`/`git blame` trên file đang sửa cho biết *vì sao* code thành ra thế này, ai là người nên hỏi.
7. **Hỏi đúng người, đúng câu**: sau khi đã tự tìm hiểu, hỏi câu cụ thể ("module billing tại sao có hai service trùng tên?") thay vì câu chung chung.

Nguyên tắc bao trùm: **học theo nhu cầu (just-in-time)** — chỉ đào sâu phần liên quan tới task hiện tại, vẽ sơ đồ/ghi chú lại khi hiểu được một mảnh, và để CI/test làm lưới an toàn khi bắt đầu sửa. Một thay đổi nhỏ-an-toàn đầu tiên (sửa lỗi nhỏ, thêm log) giúp xác nhận mình đã hiểu vòng đời dev của dự án.

</details>

<details>
<summary><b>28.12 [Hành vi] Kể về một lần bạn xử lý một sự cố production nghiêm trọng. Bạn đã làm gì? (STAR)</b></summary>

Câu hỏi hành vi muốn thấy bạn xử lý áp lực, điều tra có phương pháp, và rút ra bài học. Trả lời theo **STAR**:

- **Situation (Bối cảnh)**: "Trên Jira Clone, một sáng API gần như đứng — p99 latency vọt từ 200ms lên hơn 8s, người dùng báo không load nổi board. Không có deploy nào trong 12 giờ trước đó nên không thể đổ lỗi cho code mới."

- **Task (Nhiệm vụ)**: "Là người trực, tôi phải khôi phục dịch vụ nhanh nhất có thể, đồng thời tìm nguyên nhân gốc để nó không tái diễn."

- **Action (Hành động)**: "Tôi mở dashboard metrics trước, thấy CPU DB bình thường nhưng số **active connection chạm trần pool**. Distributed trace cho thấy thời gian dồn ở bước chờ lấy connection, không phải chạy query. Đọc log có correlation id, tôi lần ra một endpoint report mới được một khách hàng lớn gọi liên tục; nó mở transaction dài và chạy một query thiếu index nên giữ connection rất lâu, làm cạn pool và nghẽn toàn bộ request khác. Hành động tức thời: tôi tạm rate-limit endpoint đó để giải phóng pool và phục hồi dịch vụ. Sau đó thêm index phù hợp (xác nhận bằng `EXPLAIN ANALYZE`), tách phần report nặng sang job BullMQ chạy nền, và đặt statement timeout để một query lỗi không thể giữ connection vô hạn."

- **Result (Kết quả)**: "Dịch vụ hồi phục trong khoảng 20 phút sau khi rate-limit; sau khi thêm index và chuyển sang job nền, endpoint report giảm từ 8s xuống dưới 300ms và không còn ảnh hưởng request khác. Tôi viết postmortem blameless, và bổ sung alert trên *connection pool utilization* — chính cảnh báo còn thiếu đã khiến chúng tôi phát hiện trễ. Bài học lớn nhất: một query thiếu index ở một endpoint phụ vẫn có thể kéo sập cả hệ thống qua tài nguyên dùng chung như connection pool."

</details>
---

## 29. Kỹ năng mềm và Câu hỏi hành vi (Backend)

<details>
<summary><b>29.1 Phương pháp STAR là gì và vì sao nên dùng nó để trả lời câu hỏi hành vi? Cho một khung mẫu áp dụng cho tình huống backend.</b></summary>

**STAR** là khung trả lời câu hỏi hành vi (behavioral) gồm 4 phần:

- **S**ituation (Tình huống): bối cảnh ngắn gọn — hệ thống nào, vai trò của bạn, quy mô.
- **T**ask (Nhiệm vụ): trách nhiệm cụ thể của bạn, mục tiêu cần đạt.
- **A**ction (Hành động): những gì **chính bạn** đã làm (dùng "tôi", không phải "chúng tôi" chung chung), kèm lý do kỹ thuật.
- **R**esult (Kết quả): kết quả đo được (latency giảm bao nhiêu %, downtime ngắn lại, số bug giảm) và bài học rút ra.

Vì sao dùng STAR: câu hỏi hành vi đánh giá **cách bạn ra quyết định và hợp tác**, không phải kiến thức lý thuyết. Trả lời lan man, kể chuyện vòng vo, hoặc nói "chúng tôi" che mất phần đóng góp cá nhân là lỗi thường gặp. STAR ép câu trả lời súc tích, có bằng chứng (con số), và làm nổi bật vai trò của bạn.

| Phần | Cần có | Lỗi thường gặp |
|------|--------|----------------|
| Situation | 1-2 câu bối cảnh | Kể quá dài, lạc đề |
| Task | Mục tiêu rõ ràng | Mơ hồ "cần làm cho tốt hơn" |
| Action | Dùng "tôi", có lý do kỹ thuật | Dùng "chúng tôi", bỏ qua lý do |
| Result | Số liệu đo được + bài học | Không có kết quả định lượng |

```text
Mẫu khung cho backend:
S: "Hệ thống Jira Clone (NestJS + Prisma + Postgres) phục vụ ~500 user nội bộ.
    Endpoint /issues bị chậm khi board có nhiều issue."
T: "Tôi được giao giảm p95 latency của endpoint này từ 1.2s xuống dưới 300ms."
A: "Tôi profiling bằng pino log + Prisma query log, phát hiện N+1 khi load assignees.
    Tôi đổi sang include có chọn lọc, thêm composite index (projectId, status),
    và paginate bằng cursor thay vì offset."
R: "p95 giảm còn 240ms. Tôi viết lại thành checklist review để team tránh N+1 sau này."
```

Ví dụ thực tế: khi phỏng vấn về Jira Clone, mọi câu "kể về lúc bạn..." nên ráp vào đúng 4 ô S-T-A-R rồi mới nói, tránh kể chuyện theo dòng thời gian dài dòng.

</details>

<details>
<summary><b>29.2 Kể về một lần hệ thống gặp downtime / bạn bị gọi on-call lúc đêm. Bạn đã xử lý ra sao? (STAR)</b></summary>

**Situation**: Trong dự án Blog/CMS API (NestJS + Prisma + Postgres + BullMQ), khoảng 2 giờ sáng API bắt đầu trả 503 hàng loạt. Alert từ health-check (`/health` qua `@nestjs/terminus`) bắn vào Slack, tôi là người trực on-call tuần đó.

**Task**: Khôi phục dịch vụ nhanh nhất có thể (giảm MTTR — mean time to recovery), đồng thời không gây mất dữ liệu, rồi tìm nguyên nhân gốc.

**Action**:
- Đầu tiên tôi **giảm thiểu trước, điều tra sau** (mitigate first). Xem dashboard: connection pool của Prisma cạn (`Timed out fetching a connection from the pool`).
- Nguyên nhân tức thời: một background worker BullMQ mở transaction dài, giữ connection không nhả. Tôi tạm **pause queue** đó và **rolling restart** các pod để giải phóng pool → API trả 200 lại trong ~8 phút.
- Sau khi ổn định, tôi đọc log (pino, có `requestId` correlation) và thấy job xử lý từng record một trong một transaction bao trùm cả vòng lặp `await` gọi HTTP bên ngoài → giữ connection rất lâu.
- Tôi mở incident channel, ghi timeline, và communicate trạng thái cho team mỗi 15 phút.

**Result**: Downtime thực tế ~10 phút. Hôm sau tôi sửa root cause: tách lời gọi HTTP ra ngoài transaction, đặt `connection_limit` hợp lý, và thêm timeout cho transaction. Tôi viết **post-mortem không đổ lỗi (blameless)** và thêm alert "pool utilization > 80%" để cảnh báo sớm. Sau đó không tái diễn.

Bài học nhấn mạnh khi trả lời: ưu tiên **mitigation trước root-cause**, communicate liên tục, và biến sự cố thành cải tiến hệ thống (alert + post-mortem) thay vì chỉ vá lỗi.

</details>

<details>
<summary><b>29.3 Bạn ưu tiên trả nợ kỹ thuật (tech debt) hay làm tính năng mới như thế nào? Lấy ví dụ một quyết định cụ thể. (STAR)</b></summary>

**Situation**: Trong Jira Clone, module `notifications` được viết vội ban đầu: gửi email/Socket.IO **đồng bộ** ngay trong request handler, không qua queue. Codebase đang chịu áp lực ra nhiều tính năng cho sprint.

**Task**: Tôi cần cân giữa việc trả nợ kỹ thuật (tách notification ra BullMQ) và yêu cầu liên tục thêm loại notification mới từ PM.

**Action**:
- Tôi **không tranh luận cảm tính** mà quy nợ kỹ thuật về **tác động kinh doanh**: notification đồng bộ làm p95 của các endpoint mutation tăng và thỉnh thoảng request fail khi SMTP chậm.
- Tôi đề xuất nguyên tắc: tech debt nào đang **gây đau định kỳ** (recurring pain) hoặc **chặn tính năng tương lai** thì ưu tiên; tech debt "xấu nhưng vô hại" thì để lại và ghi vào backlog có nhãn.
- Tôi đề xuất "Boy Scout Rule" cho phần còn lại: ai chạm vào module nào thì dọn nhẹ chỗ đó. Với notification, tôi xin một ngày refactor đưa vào BullMQ (producer trong request, consumer ở worker) — vừa trả nợ vừa **mở đường** cho các loại notification mới mà PM muốn.

**Result**: Sau refactor, các endpoint mutation nhanh và ổn định hơn, và việc thêm notification mới giờ chỉ là thêm một job type — đúng thứ PM cần. Tôi chốt được nguyên tắc với team: ưu tiên debt theo **rủi ro × tần suất đau**, và gắn việc trả nợ vào chính tính năng đang làm để không phải xin "sprint dọn dẹp" riêng.

</details>

<details>
<summary><b>29.4 Kể về một lần bạn bất đồng với Frontend / PM về API contract. Bạn giải quyết thế nào? (STAR)</b></summary>

**Situation**: Ở Jira Clone, Frontend muốn endpoint `GET /issues` trả về **toàn bộ** issue kèm comments, assignees, attachments lồng nhau trong một response để render board trong một lần gọi. Tôi (backend) lo ngại payload phình to và N+1 query.

**Task**: Thống nhất một API contract vừa đáp ứng nhu cầu UI vừa giữ hiệu năng và khả năng mở rộng.

**Action**:
- Tôi tránh nói "không" cụt lủn. Tôi giải thích bằng **dữ liệu**: với board lớn, response sẽ vài MB và làm chậm cả mobile.
- Tôi đề xuất giải pháp dung hòa: `GET /issues` trả danh sách **gọn** (chỉ field cần cho card) + cursor pagination; chi tiết nặng lấy qua `GET /issues/:id`. Đồng thời tôi đề xuất nếu UI thật sự cần dữ liệu lồng linh hoạt thì dùng **GraphQL** (`@nestjs/graphql`) để Frontend tự chọn field, tránh over-fetch.
- Tôi viết contract ra **OpenAPI/Swagger** (qua `@nestjs/swagger`) làm "nguồn chân lý chung", và ngồi 30 phút với FE + PM để duyệt từng field. Quyết định và lý do được ghi lại (một dạng lightweight ADR).

**Result**: Chốt được contract: list gọn + cursor, detail riêng. p95 của board view ổn định, FE vẫn render đủ. Quan trọng hơn, chúng tôi lập thói quen **review API contract trước khi code**, tránh phải đập đi làm lại. Điểm nhấn: bất đồng kỹ thuật giải quyết tốt nhất bằng dữ liệu + đề xuất phương án thay thế, không phải tranh thắng-thua.

</details>

<details>
<summary><b>29.5 Làm sao bạn giải thích một vấn đề kỹ thuật phức tạp cho người không chuyên (PM, sales, khách hàng)?</b></summary>

Nguyên tắc: **bắt đầu từ tác động (impact), không bắt đầu từ cơ chế**. Người không chuyên quan tâm "điều này ảnh hưởng gì đến user / tiền / thời gian", không quan tâm "connection pool" hay "index". Sau đó mới dùng **phép so sánh đời thường (analogy)**.

Các bước:
1. Nói kết luận và tác động trước ("Tính năng export sẽ chậm 5 giây với file lớn, nên tôi muốn chuyển sang xử lý nền và gửi email khi xong").
2. Dùng analogy thay vì thuật ngữ.
3. Chỉ đi sâu kỹ thuật **nếu họ hỏi**; luôn cho họ một quyết định cần chốt (trade-off rõ ràng).

| Khái niệm kỹ thuật | Cách nói cho người không chuyên |
|--------------------|----------------------------------|
| Background job / BullMQ | "Giống gọi món rồi nhận số thứ tự — bạn không phải đứng chờ ở quầy, có món sẽ báo." |
| Database index | "Như mục lục cuốn sách — không có nó thì phải lật từng trang để tìm." |
| Rate limiting | "Giống bảo vệ cửa quán chỉ cho vào X người một lúc để quán không vỡ trận." |
| Cache (Redis) | "Như để sẵn câu trả lời hay được hỏi trên bàn, khỏi vào kho lấy mỗi lần." |
| Eventual consistency | "Số like có thể chậm vài giây mới khớp — như tin nhắn 'đã gửi' rồi mới 'đã nhận'." |

```text
❌ "Endpoint bị N+1 nên p95 latency tăng do round-trip tới Postgres."
✅ "Mỗi khi mở board, hệ thống hỏi database thừa hàng trăm lần thay vì một lần,
    nên màn hình tải chậm. Tôi sửa để hỏi gọn lại — user sẽ thấy nhanh hơn rõ rệt."
```

Ví dụ thực tế: khi báo cáo cho PM về việc chuyển export báo cáo Jira sang BullMQ, tôi nói "user bấm xong làm việc khác, hệ thống gửi link tải khi sẵn sàng" thay vì giải thích queue/worker — PM duyệt ngay vì hiểu lợi ích.

</details>

<details>
<summary><b>29.6 Văn hóa code review của bạn ra sao? Bạn cho review thế nào để vừa giữ chất lượng vừa không làm tổn thương đồng đội?</b></summary>

Triết lý: code review để **bảo vệ codebase và chia sẻ kiến thức**, không phải để phán xét người viết. Tôi tách rõ **góp ý bắt buộc** và **góp ý tùy chọn** để người nhận biết đâu là blocker.

Nguyên tắc khi tôi review:
- **Comment vào code, không vào người**: "Hàm này có thể N+1 khi list rỗng" thay vì "Bạn lại quên N+1".
- **Dùng prefix rõ mức độ**: `nit:` (vặt, tùy bạn), `suggestion:` (gợi ý), `question:` (tôi chưa hiểu), `blocking:` (phải sửa mới merge). Điều này tránh việc một comment cosmetic bị hiểu là chặn merge.
- **Khen cái tốt**, không chỉ bới lỗi — giúp người mới tự tin và lan tỏa pattern tốt.
- **PR nhỏ**: tôi khuyến khích PR < 400 dòng; PR khổng lồ review hời hợt và dễ lọt bug.
- Bất đồng kéo dài thì chuyển sang gọi 5 phút thay vì cãi nhau trong comment.

| Loại comment | Ý nghĩa | Có chặn merge? |
|--------------|---------|----------------|
| `nit:` | Sở thích cá nhân, format | Không |
| `suggestion:` | Có thể tốt hơn | Không (tùy author) |
| `question:` | Cần làm rõ | Có thể |
| `blocking:` | Bug / bảo mật / sai logic | Có |

```text
✅ "blocking: chỗ này thiếu ownership check — user A có thể sửa issue của workspace B.
    Nên thêm guard kiểm tra membership trước khi update."
✅ "nit: tên biến `d` hơi khó đọc, đổi thành `dueDate` cho rõ nhé (không cần sửa nếu gấp)."
❌ "Code này tệ, sao lại viết thế này."
```

Ví dụ thực tế: ở Jira Clone tôi đặt CODEOWNERS cho thư mục `auth/` để mọi thay đổi bảo mật buộc phải có người am hiểu review, và tôi luôn để lại ít nhất một comment khen pattern tốt trong mỗi PR để giữ không khí review tích cực.

</details>

<details>
<summary><b>29.7 Kể về một lần bạn mentor / giúp một đồng nghiệp junior. Bạn tiếp cận thế nào? (STAR)</b></summary>

**Situation**: Một bạn junior mới vào team Jira Clone gặp khó với Dependency Injection và cấu trúc module của NestJS — bạn ấy hay khai báo provider sai scope và bị lỗi `Can't resolve dependencies`.

**Task**: Giúp bạn tự tin làm việc độc lập với NestJS, không chỉ "sửa hộ" cho xong.

**Action**:
- Tôi **không sửa code thay** mà pair-programming: cùng đọc lỗi `Can't resolve dependencies of ...`, giải thích chuỗi providers/imports trong message, để bạn tự tìm chỗ thiếu `exports` trong module.
- Tôi dạy theo nguyên tắc "cho cần câu": chỉ cho bạn cách đọc DI graph và đọc tài liệu chính thức thay vì đưa đáp án.
- Tôi giao **task vừa sức tăng dần** (một CRUD endpoint nhỏ trước, rồi đến guard/interceptor), review kỹ và giải thích "vì sao" trong từng comment.
- Đặt lịch 1-1 ngắn hằng tuần để bạn hỏi thoải mái mà không ngại làm phiền.

**Result**: Sau ~3 tuần bạn tự làm được module hoàn chỉnh có test và tự debug DI. Quan trọng hơn, bạn bắt đầu review PR cho người khác. Tôi rút ra: mentor tốt là **tăng tính tự chủ** của người được mentor, đo bằng việc họ cần mình ít đi, chứ không phải mình giải quyết nhanh đến đâu.

</details>

<details>
<summary><b>29.8 Bạn xử lý áp lực deadline gấp như thế nào, đặc biệt khi yêu cầu vượt thời gian thực tế? (STAR)</b></summary>

**Situation**: Sát ngày demo cho khách của Jira Clone, PM muốn thêm tính năng "real-time activity feed" (Socket.IO) chỉ trong 3 ngày, trong khi vẫn còn các bug ưu tiên cao chưa xong.

**Task**: Giao được giá trị đúng hạn mà không hy sinh chất lượng đến mức gây nợ kỹ thuật nguy hiểm hay hỏng demo.

**Action**:
- Tôi **không im lặng nhận hết rồi vỡ deadline**. Tôi làm rõ scope và trao đổi trade-off với PM: "Trong 3 ngày, tôi giao được feed real-time cho event tạo/chuyển trạng thái issue; phần filter nâng cao và lịch sử lâu dài làm sau."
- Tôi **chia nhỏ** và làm phần rủi ro cao trước (kết nối Socket.IO + auth handshake), để phát hiện vấn đề sớm.
- Tôi cắt phạm vi an toàn: dùng giải pháp đủ tốt (broadcast theo room workspace) thay vì tối ưu sớm, và **ghi rõ chỗ nào là tạm thời** để trả nợ sau demo.
- Giao tiếp trạng thái hằng ngày để PM không bất ngờ.

**Result**: Demo chạy đúng phần đã cam kết, khách hài lòng. Sau demo tôi quay lại hoàn thiện phần đã cắt. Bài học nhấn mạnh: dưới áp lực, đòn bẩy mạnh nhất là **đàm phán scope minh bạch** (chia must-have / nice-to-have) chứ không phải cố làm thêm giờ rồi đánh đổi chất lượng âm thầm.

</details>

<details>
<summary><b>29.9 "Ownership" với bạn nghĩa là gì? Kể một lần bạn nhận trách nhiệm về một vấn đề ngoài phạm vi công việc trực tiếp. (STAR)</b></summary>

**Situation**: Ở Blog/CMS API, tôi nhận thấy CI thỉnh thoảng pass nhưng production lại lỗi do thiếu migration — đây là vấn đề "process", không thuộc task feature nào tôi đang làm.

**Task**: Không phải việc ai cụ thể được giao, nhưng nó gây rủi ro chung. Ownership nghĩa là **thấy vấn đề thì nhận, không chờ được phân công** và theo tới khi giải quyết xong.

**Action**:
- Tôi điều tra và thấy migration (Prisma) không được apply tự động trong pipeline deploy.
- Tôi đề xuất và tự làm: thêm bước `prisma migrate deploy` vào CD, và thêm check trong `/health` (qua terminus) để báo nếu schema lệch so với migration.
- Tôi không chỉ vá cho mình mà viết lại tài liệu quy trình deploy và chia sẻ trong team để mọi người cùng tránh.

**Result**: Hết hẳn lỗi "quên migration" trên production. Ownership với tôi không phải "code của tôi chạy là xong", mà là **chịu trách nhiệm tới khi vấn đề ở trong tay user được giải quyết** — kể cả phần monitoring, tài liệu, và phòng ngừa tái diễn, dù nó nằm ngoài JIRA ticket được giao.

</details>

<details>
<summary><b>29.10 Bạn học một công nghệ / thư viện backend mới như thế nào khi dự án yêu cầu? Cho một ví dụ cụ thể.</b></summary>

Cách tiếp cận của tôi có thứ tự, ưu tiên **hiểu mô hình tư duy (mental model) trước khi học API chi tiết**:

1. **Đọc "Why" trước "How"**: đọc phần motivation/concepts trong docs chính thức để hiểu thư viện giải quyết vấn đề gì và nó khác lựa chọn cũ ra sao. Tài liệu chính thức ưu tiên hơn blog.
2. **Dựng spike nhỏ tách biệt**: làm một POC tối thiểu ngoài codebase chính để thử nhanh, không sợ làm bẩn dự án.
3. **Đọc source / type definitions** khi docs chưa đủ — TypeScript types thường là tài liệu chính xác nhất.
4. **Đối chiếu với cái đã biết**: ánh xạ khái niệm mới sang cái cũ để học nhanh.
5. **Viết test + ghi chú lại** để kiến thức bám và chia sẻ được cho team.

| Bước | Mục tiêu | Ví dụ với BullMQ |
|------|----------|------------------|
| Why | Hiểu vấn đề nó giải | "Tách việc nặng khỏi request, retry, rate-limit" |
| Spike | POC tối thiểu | 1 producer + 1 worker in-memory Redis, gửi 1 job |
| Source/types | Chính xác API | Đọc type `JobsOptions`, `WorkerOptions` |
| So sánh | Học bằng analogy | So với `Bull` cũ và `setTimeout` thủ công |
| Test + note | Bám kiến thức | Test job retry với `attempts` + `backoff` |

Ví dụ thực tế: khi cần thêm queue cho Jira Clone, tôi từng dùng `Bull` (cũ) nhưng dự án yêu cầu **BullMQ**. Thay vì lao vào copy code mẫu, tôi đọc lý do BullMQ tách `Queue`/`Worker`/`QueueEvents` và dựa trên Redis streams. Tôi dựng spike thử `attempts` + `backoff: { type: 'exponential' }`, đọc type để hiểu `removeOnComplete`, rồi mới tích hợp vào dự án. Nhờ học mô hình trước, tôi tránh được lỗi kinh điển là chạy producer và worker trong cùng tiến trình rồi tự chặn event loop.

</details>

<details>
<summary><b>29.11 Kể về một lần bạn nhận sai và sửa một quyết định kỹ thuật của chính mình. (STAR)</b></summary>

**Situation**: Trong Jira Clone, ban đầu tôi quyết định cache danh sách issue của board trong Redis với TTL cố định 60s để giảm tải Postgres.

**Task**: Giảm tải DB nhưng vẫn giữ dữ liệu board "đủ tươi" cho trải nghiệm cộng tác thời gian thực.

**Action**:
- Sau khi release, QA và user phàn nàn: kéo thả issue xong, người khác phải chờ tới 60s mới thấy — phá vỡ cảm giác real-time.
- Tôi **thừa nhận quyết định cache theo TTL là sai cho use-case cộng tác** thay vì bảo vệ nó. Tôi không đổ lỗi cho yêu cầu mơ hồ.
- Tôi đổi chiến lược: dùng **cache invalidation theo event** (khi có mutation thì xóa/cập nhật key cache của board đó) và phát update qua Socket.IO, thay vì dựa vào TTL dài. Tôi viết lại với key per-board và invalidation trong cùng flow update.

**Result**: Board cập nhật gần như tức thì mà tải DB vẫn thấp. Tôi chia sẻ với team rằng "cache theo TTL hợp với dữ liệu ít đổi/đọc nhiều, không hợp với dữ liệu cộng tác real-time". Điểm nhấn khi trả lời: sẵn sàng đảo quyết định của chính mình dựa trên bằng chứng là dấu hiệu của senior, không phải điểm yếu.

</details>

<details>
<summary><b>29.12 Khi nhận một bug production khẩn cấp mà bạn không phải người viết code, bạn xử lý quy trình thế nào? (STAR)</b></summary>

**Situation**: Production của Blog/CMS API trả 500 ở endpoint đăng bài; code do một đồng nghiệp đã nghỉ phép viết, tôi không quen module đó.

**Task**: Khôi phục dịch vụ và sửa đúng nguyên nhân, dù không phải "owner" của code.

**Action**:
- **Reproduce trước khi sửa**: tôi tái hiện lỗi ở staging với payload tương tự để chắc chắn hiểu đúng, không vá mò.
- **Khoanh vùng bằng dữ liệu**: dùng log có `requestId` (pino) và stack trace → thấy lỗi là validation (`class-validator`) cho phép `null` đi qua, gây Prisma `notNull` constraint vỡ.
- **Mitigation an toàn trước**: cân nhắc rollback về bản trước nếu sửa lâu; ở đây fix nhỏ nên tôi sửa nhanh: thêm rule `@IsNotEmpty()` đúng field và xử lý lỗi DB thành 400 thân thiện.
- Viết **test tái hiện bug** trước khi vá, để chắc bug không quay lại (regression test).

**Result**: Khôi phục trong ~15 phút với test che lưng. Tôi để lại comment trong PR giải thích nguyên nhân để người owner quay lại nắm được. Bài học: khi đụng code lạ, ưu tiên reproduce + đọc log + test regression, và đừng để "không phải code của tôi" làm cớ chậm trễ — đó cũng là một biểu hiện của ownership.

</details>

---

## 30. MongoDB và Mongoose (NoSQL)

<details>
<summary><b>30.1 Khi nào bạn chọn MongoDB thay vì PostgreSQL? Cho ví dụ document model vs relational model.</b></summary>

Lựa chọn không phải "NoSQL tốt hơn SQL" mà là **mô hình dữ liệu nào khớp với access pattern**. MongoDB lưu **document** (BSON, gần như JSON), schema linh hoạt, scale ngang bằng sharding tốt. PostgreSQL là **relational**, mạnh ở ràng buộc toàn vẹn (FK, constraint), JOIN nhiều bảng, và transaction ACID chặt.

| Tiêu chí | MongoDB (document) | PostgreSQL (relational) |
|---|---|---|
| Hình dạng dữ liệu | Lồng nhau, tự chứa (self-contained) | Chuẩn hoá thành nhiều bảng quan hệ |
| Schema | Linh hoạt, đa hình (polymorphic) | Cố định, có ràng buộc mạnh |
| Truy vấn chủ đạo | Đọc cả "aggregate" theo một khoá | JOIN nhiều thực thể, query ad-hoc |
| Quan hệ many-to-many phức tạp | Cồng kềnh (phải $lookup) | Tự nhiên với bảng nối |
| Toàn vẹn dữ liệu | Ứng dụng tự lo phần lớn | DB cưỡng chế bằng FK/constraint |

```js
// MongoDB: một bài viết là MỘT document tự chứa
{
  _id: ObjectId("..."),
  title: "NoSQL vs SQL",
  author: { id: "u1", name: "Apollo" }, // embed thông tin hiển thị
  tags: ["mongodb", "nestjs"],
  comments: [{ user: "u2", text: "Hay!", at: ISODate("...") }]
}
```

Quy tắc thực dụng: chọn MongoDB khi dữ liệu đọc/ghi **theo một aggregate root** (ví dụ giỏ hàng, log sự kiện, CMS content, catalog sản phẩm đa thuộc tính), khi schema thay đổi thường xuyên, hoặc cần write throughput cao + scale ngang. Chọn PostgreSQL khi quan hệ nhiều-nhiều phức tạp, cần báo cáo JOIN tuỳ biến, và toàn vẹn giao dịch là bắt buộc (tài chính, đặt vé).

Ví dụ thực tế: dự án chính Jira Clone tôi dùng Postgres + Prisma vì Issue/Project/User/Comment đan xen nhiều quan hệ và cần ràng buộc chặt. Nhưng một service log activity hoặc một Blog/CMS lưu nội dung dạng block linh hoạt thì MongoDB hợp hơn vì mỗi bài là một document tự chứa.
</details>

<details>
<summary><b>30.2 EMBED và REFERENCE khác nhau thế nào? Khi nào denormalize bằng cách embed, khi nào tách reference?</b></summary>

Đây là quyết định thiết kế quan trọng nhất trong MongoDB.

- **Embed (nhúng)**: đặt dữ liệu con ngay bên trong document cha. Đọc một lần lấy đủ, không cần JOIN. Phù hợp quan hệ **"contains" / one-to-few**, dữ liệu con luôn được truy cập cùng cha và ít sống độc lập.
- **Reference (tham chiếu)**: lưu `ObjectId` trỏ tới document khác (giống FK), rồi `populate` khi cần. Phù hợp **one-to-many lớn / many-to-many**, dữ liệu con cần truy vấn độc lập hoặc được chia sẻ.

```js
// EMBED: địa chỉ luôn đi cùng user, ít, không query riêng -> nhúng
{ _id: 1, name: "Apollo", addresses: [{ city: "HCM", street: "..." }] }

// REFERENCE: một post có thể có hàng nghìn comment -> tách + trỏ ObjectId
// posts:    { _id: "p1", title: "..." }
// comments: { _id: "c1", postId: "p1", text: "..." }
```

Tiêu chí quyết định (rule of thumb):
1. **Quan hệ một-vài (one-to-few) và đọc cùng nhau** → embed.
2. **Quan hệ một-rất-nhiều / unbounded** (comment, log) → reference, tránh document phình to (giới hạn document 16MB).
3. **Dữ liệu con được chia sẻ / cần cập nhật một chỗ** (ví dụ thông tin user) → reference để tránh phải sửa nhiều bản sao.
4. **Đọc nhiều hơn ghi và chấp nhận lệch nhẹ** → có thể **denormalize**: nhúng một bản sao vài field hiển thị (tên, avatar) để khỏi $lookup, đánh đổi việc phải đồng bộ khi dữ liệu gốc đổi.

Trade-off của denormalization: tăng tốc đọc nhưng phải tự lo **đồng bộ** (đổi tên user → cập nhật mọi document đã nhúng tên), và rủi ro dữ liệu lệch (stale). Hỏi mẹo hay gặp: "embed hay reference cho comment?" — đáp án: reference nếu comment nhiều/không giới hạn; chỉ embed khi chắc chắn ít (one-to-few).

Ví dụ thực tế: Blog/CMS API tôi nhúng `author: { id, name, avatar }` vào post (denormalize) để render danh sách bài không cần $lookup, nhưng comment thì tách riêng vì có thể rất nhiều.
</details>

<details>
<summary><b>30.3 Định nghĩa Schema và Model trong Mongoose như thế nào? Phân biệt Schema, Model và Document.</b></summary>

Mongoose là ODM (Object Document Mapper) đặt một lớp **schema + validation + type** lên trên MongoDB vốn schemaless.

- **Schema**: định nghĩa hình dạng, kiểu, validation, index, default, virtual của document.
- **Model**: là constructor compile từ schema, gắn với một **collection**; là interface để CRUD (`User.find()`, `new User(...)`).
- **Document**: một instance cụ thể trả về từ model (`const u = await User.findById(...)`), có method như `.save()`, `.populate()`.

```ts
import { Schema, model, Types } from 'mongoose';

const postSchema = new Schema(
  {
    title: { type: String, required: true, trim: true },
    slug: { type: String, required: true, unique: true },
    status: { type: String, enum: ['draft', 'published'], default: 'draft' },
    author: { type: Types.ObjectId, ref: 'User', required: true }, // reference
    tags: [{ type: String }],
  },
  { timestamps: true }, // tự thêm createdAt / updatedAt
);

const Post = model('Post', postSchema); // collection mặc định: "posts"
```

Điểm cần nhớ: `ref: 'User'` là khai báo cho `populate` biết trỏ tới model nào; `timestamps: true` rất hay dùng; tên collection mặc định là số nhiều, viết thường của tên model.
</details>

<details>
<summary><b>30.4 Tích hợp Mongoose vào NestJS với @nestjs/mongoose như thế nào (decorator schema, forFeature, inject model)?</b></summary>

NestJS có package `@nestjs/mongoose` cho phép định nghĩa schema bằng **decorator** (`@Schema`, `@Prop`) và inject model qua DI.

```ts
// post.schema.ts
@Schema({ timestamps: true })
export class Post {
  @Prop({ required: true, trim: true })
  title: string;

  @Prop({ required: true, unique: true })
  slug: string;

  @Prop({ type: MongooseSchema.Types.ObjectId, ref: 'User' })
  author: Types.ObjectId;
}
export type PostDocument = HydratedDocument<Post>;
export const PostSchema = SchemaFactory.createForClass(Post);
```

```ts
// post.module.ts — đăng ký model trong phạm vi module
@Module({
  imports: [MongooseModule.forFeature([{ name: Post.name, schema: PostSchema }])],
  providers: [PostService],
})
export class PostModule {}

// post.service.ts — inject model bằng @InjectModel
@Injectable()
export class PostService {
  constructor(@InjectModel(Post.name) private postModel: Model<PostDocument>) {}

  create(dto: CreatePostDto) {
    return this.postModel.create(dto);
  }
}
```

Kết nối toàn cục dùng `MongooseModule.forRoot(uri)` (hoặc `forRootAsync` để lấy URI từ `ConfigService`) trong `AppModule`. Lưu ý: `forFeature` đăng ký model theo từng feature module để inject đúng phạm vi; dùng `HydratedDocument<Post>` cho type của document. Mô hình này khớp triết lý module hoá của NestJS như cách `@nestjs/typeorm` hay Prisma làm.
</details>

<details>
<summary><b>30.5 Có những loại index nào trong MongoDB (single, compound, unique, text, TTL) và đánh sao cho đúng?</b></summary>

Index là yếu tố quyết định hiệu năng đọc, giống bên SQL. Các loại chính:

- **Single field**: index một field.
- **Compound**: index nhiều field theo thứ tự — tuân **rule ESR** (Equality → Sort → Range) khi sắp thứ tự field.
- **Unique**: cưỡng chế không trùng (ví dụ email, slug).
- **Text**: index full-text cho `$text` search.
- **TTL**: tự xoá document sau N giây — hợp cho session, OTP, log tạm.
- **Partial / sparse**: index có điều kiện, tiết kiệm.

```ts
// Khai báo index trên schema Mongoose
postSchema.index({ slug: 1 }, { unique: true });
postSchema.index({ status: 1, createdAt: -1 }); // compound: lọc status rồi sort theo ngày
postSchema.index({ title: 'text', body: 'text' }); // text search
// TTL: document tự xoá 3600s sau createdAt
sessionSchema.index({ createdAt: 1 }, { expireAfterSeconds: 3600 });
```

Compound index theo ESR rất hay bị hỏi: với query `find({ status: 'published' }).sort({ createdAt: -1 })`, index `{ status: 1, createdAt: -1 }` phục vụ cả lọc và sort, tránh in-memory sort. Lưu ý thực chiến:
- Index tăng tốc đọc nhưng **làm chậm ghi** và tốn RAM/đĩa → đừng đánh bừa.
- Dùng `.explain('executionStats')` để kiểm tra có dùng index (`IXSCAN`) hay phải quét toàn bộ (`COLLSCAN`).
- `unique` index không bỏ qua được trùng cũ — phải dọn data trước khi tạo.

Ví dụ thực tế: Blog/CMS API tôi đặt unique index trên `slug`, compound `{ status, publishedAt }` cho trang danh sách, và TTL trên collection lưu refresh-token tạm.
</details>

<details>
<summary><b>30.6 Aggregation pipeline là gì? Giải thích $match, $group, $lookup, $project qua một ví dụ.</b></summary>

**Aggregation pipeline** là chuỗi các **stage** mà document đi qua tuần tự, mỗi stage biến đổi rồi truyền sang stage sau — tương tự ống Unix pipe. Đây là công cụ cho báo cáo/thống kê thay cho việc kéo data về app rồi tính tay.

- `$match`: lọc document (như `WHERE`) — **đặt sớm nhất** để giảm dữ liệu và tận dụng index.
- `$group`: gom nhóm và tính toán (`$sum`, `$avg`, `$count`) — như `GROUP BY`.
- `$lookup`: JOIN sang collection khác (left outer join).
- `$project`: chọn/đổi hình dạng field xuất ra (như `SELECT`).

```ts
// Đếm số post đã publish theo từng author, kèm tên author
const result = await postModel.aggregate([
  { $match: { status: 'published' } },                 // lọc trước
  { $group: { _id: '$author', total: { $sum: 1 } } },  // gom theo author
  {
    $lookup: {                                         // join sang users
      from: 'users',
      localField: '_id',
      foreignField: '_id',
      as: 'authorDoc',
    },
  },
  { $project: { _id: 0, authorId: '$_id', total: 1, name: { $arrayElemAt: ['$authorDoc.name', 0] } } },
]);
```

Điểm cần nhớ: thứ tự stage quan trọng cho hiệu năng — `$match`/`$project` sớm để giảm khối lượng trước khi `$group`/`$lookup`. `$lookup` không có index phía `from` sẽ rất chậm. Aggregation mạnh nhưng cũng là nơi dễ viết query nặng; với báo cáo lớn nên cân nhắc precompute/materialized view.
</details>

<details>
<summary><b>30.7 populate() hoạt động thế nào và làm sao tránh N+1 với nó?</b></summary>

`populate()` thay thế field reference (`ObjectId`) bằng document thật từ collection được `ref`, mô phỏng JOIN ở tầng ứng dụng. **MongoDB không tự JOIN** khi find — Mongoose thực hiện việc này bằng truy vấn phụ.

```ts
// 1 query lấy posts, rồi 1 query phụ gom các authorId: { _id: { $in: [...] } }
const posts = await postModel
  .find({ status: 'published' })
  .populate('author', 'name avatar') // chỉ lấy name, avatar
  .populate({ path: 'comments', options: { limit: 5 } });
```

Về N+1: bản thân `populate` trên một list **đã batch** — Mongoose gom tất cả id thành một query `$in`, nên `find + populate` thường chỉ 1 + 1 query, không phải N+1. Vấn đề N+1 thật sự xuất hiện khi bạn **lặp thủ công** và populate/find trong vòng lặp:

```ts
// ANTI-PATTERN: N+1
for (const post of posts) {
  post.author = await userModel.findById(post.author); // mỗi vòng 1 query
}
```

Cách tránh: dùng `populate` (đã batch), hoặc gom id rồi `find({ _id: { $in: ids } })` một lần, hoặc dùng `$lookup` trong aggregation để DB tự gộp. Lưu ý: populate quá sâu/nhiều nhánh có thể tốn nhiều round-trip; với truy vấn nặng và cần biến đổi, `$lookup` trong aggregation thường hiệu quả hơn.

Liên hệ: vấn đề N+1 này tương tự như trong GraphQL resolver (xem §27) hay ORM SQL — gốc rễ là "1 query danh sách + N query con".
</details>

<details>
<summary><b>30.8 lean() là gì và tại sao nó tăng tốc đọc đáng kể? Đánh đổi khi dùng lean()?</b></summary>

Mặc định Mongoose "hydrate" kết quả find thành **Mongoose Document** đầy đủ — kèm getter/setter, change tracking, virtual, và các method như `.save()`. Việc này tốn CPU và RAM. `.lean()` bảo Mongoose trả về **POJO** (plain JavaScript object) thuần, bỏ qua bước hydrate.

```ts
// Trả Document đầy đủ (nặng) — dùng khi cần .save()/virtual
const post = await postModel.findById(id);

// Trả object thuần, nhanh & nhẹ hơn nhiều — dùng cho read-only API
const posts = await postModel.find().lean();
```

Lợi ích: nhanh hơn rõ rệt (thường vài lần) và tốn ít bộ nhớ hơn khi trả danh sách lớn → lý tưởng cho endpoint **chỉ đọc** (list, search, report) trả thẳng ra JSON.

Đánh đổi (vì sao không lean mọi nơi):
- Mất method instance (`.save()`, `.populate()` sau đó hạn chế), không có **virtual** và **getter/setter**.
- `pre`/`post` hook kiểu document không chạy đúng như mong đợi.
- Không có change tracking → không dùng để update qua `.save()`.

Quy tắc: `lean()` cho **đường đọc**, bỏ `lean()` khi cần biến đổi/lưu lại document. Hay bị hỏi: "API list bị chậm, tối ưu thế nào?" — câu trả lời gồm index hợp lý + `.lean()` + chỉ `select` field cần + phân trang.

Ví dụ thực tế: trang danh sách bài viết của Blog/CMS tôi dùng `.find().select('title slug author').lean()` để giảm tải, còn endpoint sửa bài thì lấy document đầy đủ để `.save()`.
</details>

<details>
<summary><b>30.9 Mongoose middleware (hook) là gì? Cho ví dụ pre('save') và lưu ý quan trọng.</b></summary>

**Middleware / hook** là hàm chạy tự động quanh một thao tác (save, validate, remove, query, aggregate). Hai loại quan trọng: `pre` (trước) và `post` (sau). Dùng cho hash mật khẩu, sinh slug, audit, làm sạch dữ liệu.

```ts
userSchema.pre('save', async function (next) {
  // 'this' là document đang lưu
  if (!this.isModified('password')) return next(); // chỉ hash khi đổi password
  this.password = await bcrypt.hash(this.password, 10);
  next();
});

postSchema.pre('save', function (next) {
  if (this.isModified('title')) this.slug = slugify(this.title);
  next();
});
```

Lưu ý quan trọng (hay là bẫy phỏng vấn):
1. Hook kiểu **document** (`save`, `validate`) có `this` là document; hook kiểu **query** (`findOneAndUpdate`, `updateMany`) có `this` là **query object**, KHÔNG phải document → muốn truy field phải `this.getUpdate()`. Vì vậy `pre('save')` **không** chạy khi bạn dùng `findOneAndUpdate` hay `updateOne` — đây là lỗi rất phổ biến khi logic hash/slug không kích hoạt.
2. Dùng `this.isModified(field)` để tránh xử lý lại không cần thiết (ví dụ hash lại password mỗi lần save).
3. Trong NestJS `@nestjs/mongoose`, gắn hook qua factory: `MongooseModule.forFeatureAsync` với `useFactory` rồi `schema.pre('save', ...)` trước khi return.

Ví dụ thực tế: User schema của tôi hash password bằng `pre('save')`, nhưng tôi luôn cảnh giác: nếu admin reset password bằng `findOneAndUpdate` thì hook không chạy — phải tự hash trong service hoặc dùng `.save()`.
</details>

<details>
<summary><b>30.10 Transaction đa document trong MongoDB hoạt động ra sao và yêu cầu hạ tầng gì?</b></summary>

Một thao tác trên **một document** trong MongoDB vốn đã atomic. Khi cần atomic **trên nhiều document/nhiều collection** (ví dụ trừ tiền ví A và cộng ví B), cần **multi-document transaction** (từ MongoDB 4.0+). Yêu cầu bắt buộc: cụm phải chạy ở dạng **replica set** (hoặc sharded cluster) — standalone không hỗ trợ transaction.

```ts
const session = await connection.startSession();
try {
  session.startTransaction();
  await accountModel.updateOne({ _id: a }, { $inc: { balance: -100 } }, { session });
  await accountModel.updateOne({ _id: b }, { $inc: { balance: +100 } }, { session });
  await session.commitTransaction(); // tất cả hoặc không gì cả
} catch (err) {
  await session.abortTransaction(); // rollback
  throw err;
} finally {
  session.endSession();
}
```

Trong NestJS, inject `@InjectConnection()` để lấy `connection` rồi `startSession()`, và truyền `{ session }` vào mọi thao tác trong transaction.

Lưu ý/trade-off:
- Transaction có chi phí (lock, write conflict) → triết lý MongoDB là **thiết kế schema để hạn chế cần transaction**: nhúng dữ liệu liên quan vào cùng một document thì cập nhật một document là atomic sẵn, khỏi cần multi-document transaction.
- Local dev: muốn test transaction phải dựng replica set (ví dụ `mongod --replSet rs0` rồi `rs.initiate()`, hoặc dùng `run-rs`/Atlas). Đây là lý do nhiều bug "transaction không chạy" do dev đang chạy standalone.

Ví dụ thực tế: nếu Blog/CMS cần "publish bài + ghi audit log + tăng counter" cùng lúc, tôi cân nhắc gộp vào một document hoặc dùng transaction trên replica set; còn nếu lệch nhẹ chấp nhận được thì xử lý bất đồng bộ qua queue.
</details>

<details>
<summary><b>30.11 Vì MongoDB schemaless, làm sao đảm bảo schema validation và consistency dữ liệu?</b></summary>

MongoDB không cưỡng chế schema ở tầng storage, nên trách nhiệm validation chuyển sang **ứng dụng** và (tuỳ chọn) tầng DB. Có ba lớp phòng thủ:

1. **Mongoose schema validation**: `required`, `enum`, `min/max`, `match` (regex), `validate` custom. Chạy trước khi save.
2. **DTO + class-validator ở tầng NestJS**: validate **input từ client** ngay ở controller (ValidationPipe) trước khi chạm tới service/DB — bắt lỗi sớm và trả 400 rõ ràng.
3. **MongoDB JSON Schema validation** (`$jsonSchema` ở mức collection): chốt chặn cuối ngay trong DB, ngăn cả ghi từ script ngoài Mongoose.

```ts
// Mongoose-level
const orderSchema = new Schema({
  status: { type: String, enum: ['pending', 'paid', 'shipped'], required: true },
  total: { type: Number, min: 0, required: true },
  email: { type: String, match: /.+@.+/ },
});
```

```js
// DB-level (chốt cuối, áp ngay cả khi ghi không qua app)
db.createCollection("orders", {
  validator: { $jsonSchema: {
    required: ["status", "total"],
    properties: { total: { bsonType: "number", minimum: 0 } }
  }}
});
```

Về consistency: cần ý thức MongoDB mặc định **eventual consistency** khi đọc từ secondary. Dùng `writeConcern` (ví dụ `w: 'majority'`) để đảm bảo ghi bền vững qua nhiều node, và `readConcern`/đọc từ primary khi cần read-after-write. Khác với SQL nơi FK/constraint do DB lo, ở Mongo bạn phải chủ động chọn mức bảo đảm.

Ví dụ thực tế: trong NestJS tôi luôn validate hai lớp — DTO `class-validator` ở controller cho input ngoài, và Mongoose schema cho ràng buộc model — để không phụ thuộc hoàn toàn vào một tầng.
</details>

<details>
<summary><b>30.12 ObjectId là gì và những điểm cần lưu ý khi dùng làm khoá chính so với UUID/auto-increment?</b></summary>

`_id` mặc định trong MongoDB là **ObjectId** — 12 byte gồm: 4 byte timestamp (giây) + 5 byte random (machine/process) + 3 byte counter tăng dần. Mỗi document tự có `_id` unique, sinh ở phía client driver nên không cần round-trip để lấy id.

```ts
import { Types } from 'mongoose';
const id = new Types.ObjectId();
console.log(id.getTimestamp()); // suy ra thời điểm tạo từ ObjectId
// So sánh phải đúng kiểu: string '...' KHÁC ObjectId('...')
postModel.find({ author: new Types.ObjectId(authorId) }); // ép kiểu khi cần
```

Điểm cần lưu ý:
- ObjectId **gần như tăng dần theo thời gian** (4 byte đầu là timestamp) → sort theo `_id` xấp xỉ sort theo thời gian tạo, và chèn không gây phân mảnh index nặng (khác UUIDv4 ngẫu nhiên hoàn toàn).
- **Đoán được thời điểm tạo và xấp xỉ thứ tự** → không nên lộ ra ngoài nếu cần che thông tin; cân nhắc UUID khi muốn id không suy ra được metadata.
- Bẫy thường gặp: so sánh `ObjectId` với **string** không khớp; query/`populate` sai do quên ép kiểu. Khi nhận id từ request (string) phải validate và ép sang `ObjectId`.
- Có thể tự đặt `_id` kiểu khác (string/UUID) nếu cần, nhưng mất lợi thế thời gian + tăng kích thước index.

So sánh nhanh: ObjectId (mặc định Mongo, gọn 12 byte, sortable theo thời gian) vs UUID (chuẩn, không lộ thông tin, ngẫu nhiên gây index lớn/phân mảnh) vs auto-increment (như SQL, khó scale ngang/sharding vì cần điểm sinh số tập trung).
</details>

---

## 31. Git Workflow và Cộng tác

<details>
<summary><b>31.1 So sánh GitFlow, GitHub Flow và Trunk-Based Development? Khi nào chọn mô hình nào?</b></summary>

Ba mô hình branching phổ biến, khác nhau ở số lượng long-lived branch và tần suất tích hợp về nhánh chính.

| Tiêu chí | GitFlow | GitHub Flow | Trunk-Based |
|---|---|---|---|
| Long-lived branch | `main`, `develop`, + `release/*`, `hotfix/*` | `main` | `main` (`master`) |
| Feature branch | có, merge vào `develop` | có, merge vào `main` | có nhưng **rất ngắn** (giờ–ngày) |
| Tần suất tích hợp | thấp, theo release | trung bình, mỗi PR | cao, nhiều lần/ngày |
| Hợp với | release đóng gói, nhiều version song song (desktop, mobile) | web app deploy liên tục | team CI/CD trưởng thành, deploy nhiều lần/ngày |
| Rủi ro | nặng nề, merge `develop`→`main` dễ conflict lớn | đơn giản, ít kiểm soát release | cần test tự động mạnh + feature flag |

- **GitFlow**: nhiều nhánh, phù hợp sản phẩm phát hành theo version có hỗ trợ song song. Ngày nay bị xem là nặng nề với web service deploy liên tục.
- **GitHub Flow**: chỉ `main` + feature branch ngắn, mỗi PR merge xong là deploy được. Đơn giản, hợp đa số web app.
- **Trunk-Based**: branch sống cực ngắn, tích hợp liên tục về `main`, code chưa hoàn thiện ẩn sau **feature flag**. Giảm tối đa merge conflict ("merge hell"), nhưng đòi hỏi test tự động và CI mạnh.

```bash
# GitHub Flow tiêu biểu: branch ngắn từ main, PR, merge, xoá
git switch -c feat/issue-comments
# ... commit nhỏ ...
git push -u origin feat/issue-comments
# mở PR -> review -> merge -> xoá branch
```

Trade-off: tôi chọn dựa trên chu kỳ release, không theo "trend". Service deploy hằng ngày thì GitHub Flow / Trunk-Based; sản phẩm đóng gói nhiều version thì cân nhắc GitFlow.

Ví dụ thực tế: dự án Jira Clone (NestJS API) deploy liên tục nên tôi dùng GitHub Flow — mỗi feature một branch ngắn, PR vào `main`, CI chạy test rồi auto-deploy staging.
</details>

<details>
<summary><b>31.2 Phân biệt merge, rebase và squash merge? Khi nào dùng cái nào?</b></summary>

Cả ba đều để đưa thay đổi từ branch này vào branch khác, nhưng tạo ra lịch sử (history) rất khác nhau.

- **Merge commit** (`git merge --no-ff`): giữ nguyên toàn bộ commit của feature branch và tạo thêm một "merge commit". Lịch sử trung thực nhưng đồ thị rẽ nhánh nhiều, khó đọc.
- **Rebase** (`git rebase main`): "phát lại" các commit của feature lên đỉnh `main`, tạo lịch sử **thẳng tuyến tính**. Nhưng nó **viết lại commit hash** → không bao giờ rebase nhánh đã được người khác pull về.
- **Squash merge**: gộp toàn bộ commit của PR thành **một commit duy nhất** trên `main`. `main` rất sạch (1 PR = 1 commit), dễ revert cả tính năng, nhưng mất lịch sử commit chi tiết bên trong.

```bash
# Cập nhật feature branch theo main bằng rebase (lịch sử thẳng)
git switch feat/auth
git fetch origin
git rebase origin/main
# nếu đã push trước đó, phải force-push AN TOÀN:
git push --force-with-lease
```

| Mục đích | Nên dùng |
|---|---|
| Cập nhật feature branch theo `main` | rebase (cá nhân, chưa share) |
| Đưa PR đã review vào `main`, sử merge sạch | squash merge |
| Giữ trọn lịch sử nhánh release/audit | merge commit `--no-ff` |

Trade-off: quy ước phổ biến của tôi là **rebase để cập nhật branch riêng**, **squash merge khi gộp PR** để `main` mỗi dòng là một PR có ý nghĩa.

Ví dụ thực tế: trên Jira Clone, repo cấu hình GitHub "Squash and merge" mặc định; tiêu đề squash commit lấy theo tiêu đề PR theo chuẩn Conventional Commits.
</details>

<details>
<summary><b>31.3 Quy tắc vàng của rebase là gì, và vì sao force-push phải dùng --force-with-lease?</b></summary>

**Golden rule of rebasing**: không bao giờ rebase (hay nói rộng hơn là viết lại lịch sử) trên branch mà người khác đang dựa vào — điển hình là các nhánh chung như `main`/`develop`. Lý do: rebase tạo ra commit mới (hash mới); nếu người khác đã pull các commit cũ, lịch sử của họ và của bạn sẽ phân kỳ, gây ra merge rối loạn và mất commit.

Khi rebase nhánh **của riêng bạn** mà đã push lên remote, bạn phải force-push vì lịch sử local đã khác remote. Nhưng đừng dùng `--force` "mù":

- `git push --force`: ghi đè remote vô điều kiện → nếu đồng nghiệp vừa push thêm commit lên cùng branch (ví dụ sửa giúp review), bạn **xoá mất** commit của họ.
- `git push --force-with-lease`: chỉ ghi đè **nếu remote vẫn đúng như bạn thấy lần fetch gần nhất**. Nếu ai đó vừa push, lệnh bị từ chối → an toàn.

```bash
git rebase -i origin/main        # dọn dẹp commit nội bộ branch
git push --force-with-lease      # an toàn: từ chối nếu remote đã đổi
```

Điểm cần nhớ: trên branch chung không bao giờ force-push; trên branch riêng luôn dùng `--force-with-lease` thay cho `--force`.

Ví dụ thực tế: trong Jira Clone, nhánh `main` được GitHub đặt **branch protection** cấm force-push; mọi dọn dẹp lịch sử chỉ làm trên feature branch trước khi mở PR.
</details>

<details>
<summary><b>31.4 Quy trình giải quyết merge conflict ra sao? Conflict thường phát sinh từ đâu?</b></summary>

Conflict xảy ra khi Git không tự gộp được vì **hai phía cùng sửa một vùng dòng** (hoặc một bên xoá file, một bên sửa). Git đánh dấu vùng xung đột bằng marker `<<<<<<<`, `=======`, `>>>>>>>`.

Quy trình xử lý an toàn:

1. Cập nhật base mới nhất (`git fetch`), rebase hoặc merge `main` vào branch.
2. Git báo conflict → mở file, chọn/hoà hai phía, **xoá hết marker**.
3. `git add <file>` để đánh dấu đã giải quyết.
4. `git rebase --continue` (nếu rebase) hoặc `git commit` (nếu merge).
5. **Chạy lại test/build** vì gộp tay rất dễ tạo code không compile.

```bash
git switch feat/board
git fetch origin
git rebase origin/main
# CONFLICT (content): src/issue/issue.service.ts
git status                       # xem file nào đang conflict
# ... sửa file, xoá marker <<<<<<< ======= >>>>>>> ...
git add src/issue/issue.service.ts
git rebase --continue
# Lỡ tay rối quá:
git rebase --abort               # quay về trạng thái trước rebase
```

Mẹo giảm conflict: branch ngắn + merge thường xuyên về `main`, chia file/module rõ ràng, dùng `git rerere` để Git nhớ cách giải quyết lặp lại. Với file lock đặc thù (ví dụ `pnpm-lock.yaml`) thì tốt nhất lấy một phía rồi `pnpm install` lại để tự sinh, đừng gộp tay.

Ví dụ thực tế: trên Blog/CMS API, conflict hay gặp ở `pnpm-lock.yaml` và migration Prisma — với lock file tôi `git checkout --theirs` rồi cài lại; với migration tôi đặt tên file timestamp riêng để hai migration không đụng nhau.
</details>

<details>
<summary><b>31.5 Conventional Commits là gì và nó liên kết với Semantic Versioning như thế nào?</b></summary>

**Conventional Commits** là quy ước đặt tên commit theo cấu trúc `type(scope): subject`. Các `type` phổ biến: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`, `build`, `ci`. Lợi ích: lịch sử đọc được, **tự sinh CHANGELOG**, và **tự suy ra version** theo SemVer.

**Semantic Versioning** dạng `MAJOR.MINOR.PATCH`:

- `fix:` → tăng **PATCH** (1.2.3 → 1.2.4)
- `feat:` → tăng **MINOR** (1.2.3 → 1.3.0)
- Có `BREAKING CHANGE:` trong footer hoặc dấu `!` (ví dụ `feat!:`) → tăng **MAJOR** (1.2.3 → 2.0.0)

```bash
# feat: tăng MINOR
feat(issue): add bulk status update endpoint

# fix: tăng PATCH
fix(auth): reject expired refresh token

# Breaking change: tăng MAJOR
feat(api)!: rename /tasks to /issues

BREAKING CHANGE: client phải đổi path /tasks thành /issues
```

Công cụ như `semantic-release` hoặc `standard-version` đọc các commit này để tự bump version, tạo git tag và CHANGELOG trong CI — không cần sửa `package.json` bằng tay.

Ví dụ thực tế: Jira Clone API dùng `commitlint` để ép chuẩn Conventional Commits, và CI dùng `semantic-release` để mỗi lần merge vào `main` tự tag version + publish changelog.
</details>

<details>
<summary><b>31.6 Code review nên tập trung vào những gì? Vì sao PR nhỏ lại tốt hơn PR lớn?</b></summary>

Review **không phải** để bắt lỗi format (đó là việc của linter/Prettier tự động). Reviewer nên tập trung vào những thứ máy không kiểm được:

- **Tính đúng đắn (correctness)**: logic, edge case, xử lý lỗi, race condition.
- **Bảo mật**: SQL/NoSQL injection, thiếu authz, lộ secret, log dữ liệu nhạy cảm.
- **Thiết kế/khả năng bảo trì**: đặt đúng layer (controller/service/repository), tránh trùng lặp, đặt tên rõ ràng.
- **Test**: có test cho path chính và edge case không.
- **Hợp đồng API**: breaking change, backward compatibility.

Vì sao **PR nhỏ tốt hơn**:

| PR nhỏ (< ~300 dòng) | PR lớn |
|---|---|
| Reviewer đọc kỹ, bắt được bug | Reviewer "LGTM" cho qua vì ngợp |
| Merge nhanh, ít conflict | Treo lâu, rebase liên tục |
| Dễ revert một thay đổi | Revert kéo theo nhiều thứ |
| Phản hồi nhanh, vòng lặp ngắn | Phản hồi chậm, dễ tranh cãi lớn |

Nguyên tắc thực dụng: một PR nên làm **một việc**. Nếu vừa refactor vừa thêm feature, tách thành hai PR. Comment review nên cụ thể và đề xuất hướng sửa, phân biệt "phải sửa" với "nit/tuỳ chọn".

Ví dụ thực tế: trên Jira Clone tôi tách "thêm endpoint comment" và "refactor IssueService" thành hai PR riêng — reviewer duyệt phần endpoint trong vài phút thay vì phải đọc một diff 800 dòng lẫn lộn.
</details>

<details>
<summary><b>31.7 Phân biệt git revert và git reset? Trên branch chung phải dùng cái nào?</b></summary>

Cả hai đều "huỷ" thay đổi nhưng theo cách hoàn toàn khác:

- **`git revert <commit>`**: tạo một **commit mới** đảo ngược nội dung của commit cũ. Lịch sử được **giữ nguyên và bổ sung** → an toàn cho branch đã được chia sẻ.
- **`git reset <commit>`**: **dời con trỏ branch** về commit cũ, **xoá** các commit phía sau khỏi lịch sử branch → viết lại lịch sử, **nguy hiểm trên branch chung**.

`git reset` có ba chế độ:

| Mode | HEAD | Staging (index) | Working dir |
|---|---|---|---|
| `--soft` | dời | giữ thay đổi (đã staged) | giữ |
| `--mixed` (mặc định) | dời | bỏ stage | giữ |
| `--hard` | dời | xoá | **xoá** (mất code chưa commit) |

```bash
# An toàn trên main/develop: đảo ngược một commit đã đẩy lên
git revert a1b2c3d            # tạo commit "Revert ..."

# Chỉ dùng trên branch RIÊNG chưa share: bỏ commit cuối nhưng giữ code để sửa
git reset --soft HEAD~1
```

Quy tắc: **branch chung (`main`, branch người khác đang dùng) → luôn `git revert`**; `reset --hard` chỉ dùng trên branch riêng và khi chắc chắn, vì nó có thể mất công việc chưa commit (`git reflog` đôi khi cứu được commit, nhưng working dir bị `--hard` thì mất hẳn).

Ví dụ thực tế: trên Jira Clone, một commit lên `main` làm hỏng migration; tôi `git revert` để rollback an toàn rồi mở PR fix riêng, thay vì reset lịch sử nhánh chung.
</details>

<details>
<summary><b>31.8 git cherry-pick dùng để làm gì? Có rủi ro gì cần lưu ý?</b></summary>

`git cherry-pick <commit>` lấy **một (hoặc vài) commit cụ thể** từ branch này và áp dụng (như commit mới) sang branch hiện tại — không cần merge cả branch.

Trường hợp dùng điển hình:

- **Hotfix**: vá lỗi gấp đã fix trên `main` cần đưa nhanh sang nhánh `release/1.4` đang chạy production.
- Lấy đúng một commit hữu ích từ một branch chưa sẵn sàng merge toàn bộ.

```bash
git switch release/1.4
git cherry-pick a1b2c3d              # đưa đúng commit fix sang
git cherry-pick a1b2c3d..e5f6g7h     # một dải commit
```

Rủi ro / lưu ý:

- Commit được cherry-pick có **hash mới** → cùng một thay đổi giờ tồn tại ở hai nơi với hash khác nhau. Khi sau này merge hai branch lại, có thể gây **duplicate commit** hoặc conflict.
- Cherry-pick lẻ dễ thiếu commit phụ thuộc (commit trước nó), khiến code không chạy.
- Lạm dụng cherry-pick thay cho merge/rebase làm lịch sử khó truy vết. Thường ưu tiên fix trên `main` rồi để quy trình release tự đưa xuống; cherry-pick chỉ cho hotfix gấp.

Ví dụ thực tế: trên Blog/CMS API, một bug rò rỉ token được fix trên `main`; production đang ở nhánh `release/2.1`, tôi cherry-pick đúng commit fix sang để hotfix mà không phải kéo theo các feature chưa release.
</details>

<details>
<summary><b>31.9 .gitignore dùng làm gì, và lỡ commit secret rồi thì xử lý thế nào?</b></summary>

`.gitignore` liệt kê các đường dẫn mà Git **không track** — tránh commit thứ không nên: `node_modules`, build output (`dist`), log, và đặc biệt là **secret** (`.env`).

```gitignore
node_modules/
dist/
*.log
.env
.env.*
!.env.example      # vẫn commit file mẫu (không chứa giá trị thật)
coverage/
```

Lưu ý quan trọng: `.gitignore` **chỉ tác dụng với file chưa được track**. Nếu file đã từng commit, thêm vào `.gitignore` không gỡ nó ra; phải:

```bash
git rm --cached .env       # ngừng track nhưng giữ file trên đĩa
git commit -m "chore: stop tracking .env"
```

Nếu secret **đã bị push lên remote**, coi như **đã lộ**:

1. **Xoay (rotate) ngay secret** đó — đổi key/password/token. Đây là việc quan trọng nhất, vì lịch sử Git có thể đã bị clone.
2. Xoá khỏi toàn bộ lịch sử bằng `git filter-repo` (hoặc BFG Repo-Cleaner) rồi force-push — nhưng không thay thế cho việc rotate.

Phòng ngừa: dùng `.env.example` cho cấu trúc, đọc secret từ biến môi trường / secret manager, và bật secret-scanning trong CI.

Ví dụ thực tế: trong Blog/CMS API tôi commit `.env.example` (chỉ liệt kê `DATABASE_URL=`, `JWT_SECRET=` rỗng), còn `.env` thật được ignore; NestJS đọc qua `@nestjs/config` từ biến môi trường khi deploy.
</details>

<details>
<summary><b>31.10 husky + lint-staged để làm gì? Tại sao chỉ chạy lint trên file đã staged?</b></summary>

**husky** quản lý **Git hooks** (script chạy tự động tại các sự kiện như `pre-commit`, `commit-msg`). **lint-staged** chạy lệnh (lint/format) **chỉ trên các file đang staged** thay vì toàn repo.

Vì sao chỉ chạy trên file staged: lint/format cả repo mỗi commit thì **chậm** và còn vô tình sửa file không liên quan đến commit. lint-staged giữ pre-commit nhanh và phạm vi gọn — chỉ kiểm thứ bạn sắp commit.

```jsonc
// package.json
{
  "lint-staged": {
    "*.ts": ["eslint --fix", "prettier --write"]
  }
}
```

```bash
# .husky/pre-commit  — chặn commit nếu lint fail
npx lint-staged
```

```bash
# .husky/commit-msg — ép Conventional Commits (nối với commitlint)
npx --no-install commitlint --edit "$1"
```

Lưu ý: hook chỉ chạy ở **máy local** và có thể bị bỏ qua bằng `git commit --no-verify`, nên đây là "hàng rào nhanh" chứ **không thay thế CI**. CI vẫn phải chạy lint/test đầy đủ như nguồn chân lý.

Ví dụ thực tế: Jira Clone (pnpm) dùng husky chạy `lint-staged` (ESLint + Prettier) ở `pre-commit` và `commitlint` ở `commit-msg`, còn pipeline CI vẫn chạy `pnpm lint` + `pnpm test` toàn bộ để không phụ thuộc hook local.
</details>

<details>
<summary><b>31.11 git bisect giúp tìm commit gây lỗi như thế nào?</b></summary>

`git bisect` dùng **binary search** trên lịch sử commit để tìm chính xác commit đầu tiên gây ra bug. Thay vì xem từng commit (O(n)), nó chia đôi liên tục → chỉ cần khoảng `log2(n)` lần kiểm tra (ví dụ 1000 commit chỉ cần ~10 bước).

Cách dùng thủ công:

```bash
git bisect start
git bisect bad                 # commit hiện tại đang LỖI
git bisect good v1.4.0         # commit/tag biết là CÒN TỐT
# Git checkout commit giữa -> bạn test:
#   nếu lỗi:  git bisect bad
#   nếu ổn:   git bisect good
# ... lặp lại, Git thu hẹp dần ...
git bisect reset               # xong, quay về branch ban đầu
```

Tự động hoá hoàn toàn bằng một script trả exit code (0 = good, khác 0 = bad):

```bash
# script chạy đúng test tái hiện bug; exit 0 nếu pass
git bisect start HEAD v1.4.0
git bisect run pnpm jest src/issue/issue.service.spec.ts
git bisect reset
```

Điều kiện để dùng tốt: bug phải **tái hiện được bằng một lệnh/test rõ ràng**, và lịch sử mỗi commit nên build/chạy được (đây là một lý do nữa để commit nhỏ, atomic).

Ví dụ thực tế: trên Jira Clone, một regression khiến endpoint `GET /issues` trả sai phân trang sau ~40 commit; tôi viết một test tái hiện rồi `git bisect run pnpm jest` để Git tự khoanh đúng commit đổi logic `skip/take` chỉ trong vài bước.
</details>

---

## 32. CI/CD và Triển khai (Docker, Cloud)

<details>
<summary><b>32.1 Phân biệt CI, Continuous Delivery và Continuous Deployment? Ranh giới giữa ba khái niệm này nằm ở đâu?</b></summary>

Ba khái niệm này thường bị gộp chung thành "CI/CD" nhưng chúng mô tả các mức độ tự động hóa khác nhau:

- **CI (Continuous Integration)**: mỗi commit/PR được merge thường xuyên vào nhánh chung, và mỗi lần như vậy pipeline tự động chạy `lint` + `test` + `build` để phát hiện lỗi sớm. Mục tiêu là *integrate liên tục*, tránh "big bang merge".
- **Continuous Delivery**: mở rộng CI — sau khi build xong, artifact (Docker image) luôn ở trạng thái **sẵn sàng deploy**, nhưng việc đẩy lên production cần **một bước bấm tay** (manual approval).
- **Continuous Deployment**: mọi commit qua được toàn bộ pipeline sẽ **tự động lên production**, không có bước duyệt thủ công.

| Giai đoạn | CI | Continuous Delivery | Continuous Deployment |
|---|---|---|---|
| Lint + test + build tự động | Có | Có | Có |
| Tạo artifact sẵn sàng deploy | Không bắt buộc | Có | Có |
| Deploy lên prod | Thủ công | Bấm nút duyệt | Tự động hoàn toàn |

```yaml
# Continuous Delivery: deploy có cổng duyệt (environment có required reviewers)
deploy-prod:
  needs: [build]
  environment: production   # GitHub sẽ chặn chờ approval ở environment này
  runs-on: ubuntu-latest
  steps:
    - run: ./scripts/deploy.sh
```

Trade-off: Continuous Deployment cho tốc độ tối đa nhưng đòi hỏi test coverage cao, feature flag, monitoring và rollback tốt. Đa số team thực tế dừng ở **Continuous Delivery** cho production (tự động tới staging, duyệt tay lên prod) để giữ kiểm soát.

Ví dụ thực tế: trong Jira Clone tôi để pipeline tự động deploy lên `staging` (Continuous Deployment cho staging) nhưng job `deploy-prod` gắn `environment: production` có required reviewer — đây là Continuous Delivery cho prod.
</details>

<details>
<summary><b>32.2 Một pipeline CI cho service NestJS nên gồm những giai đoạn nào và thứ tự ra sao? Vì sao thứ tự đó quan trọng?</b></summary>

Nguyên tắc cốt lõi: **fail fast** — đặt các bước rẻ và nhanh lên trước để phản hồi sớm, không lãng phí phút runner cho những commit chắc chắn hỏng.

Thứ tự điển hình cho NestJS:

1. **Install** (`pnpm install --frozen-lockfile`): cài đúng theo lockfile, fail nếu lockfile lệch.
2. **Lint + format check** (`eslint`, `prettier --check`): rẻ nhất, bắt lỗi style/quy ước.
3. **Type check / build** (`tsc --noEmit` hoặc `nest build`): bắt lỗi type trước khi chạy test.
4. **Unit test** (`jest`): nhanh, không cần hạ tầng ngoài.
5. **Integration / e2e test** (`jest --config test/jest-e2e.json`): cần DB (Postgres) qua service container.
6. **Build artifact** (Docker image): chỉ chạy khi mọi bước trên xanh.
7. **Deploy**: ở nhánh được phép (thường `main`).

```yaml
jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: pnpm }
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint
      - run: pnpm exec tsc --noEmit
      - run: pnpm test --ci --coverage
```

Vì sao thứ tự quan trọng: nếu để build Docker image (chậm, vài phút) trước lint (vài giây), một lỗi format nhỏ vẫn tốn cả phút build mới báo. Đặt rẻ-trước-đắt-sau giúp dev nhận feedback trong giây.

Bẫy thường gặp: bỏ `--frozen-lockfile` khiến CI tự cập nhật lockfile, dẫn tới "chạy được trên CI nhưng khác máy dev". Luôn dùng install ở chế độ frozen/ci trong pipeline.
</details>

<details>
<summary><b>32.3 Trong GitHub Actions, phân biệt workflow, job và step? Job chạy song song hay tuần tự, và làm sao tạo phụ thuộc?</b></summary>

Cấu trúc phân cấp của GitHub Actions:

- **Workflow**: một file `.yml` trong `.github/workflows/`, kích hoạt bởi event (`on: push`, `pull_request`, `schedule`...).
- **Job**: một đơn vị chạy trên **một runner riêng** (máy ảo sạch). Các job **mặc định chạy song song**.
- **Step**: một lệnh (`run`) hoặc một action (`uses`) bên trong job, chạy **tuần tự** và **chia sẻ cùng filesystem** trong job đó.

Điểm mấu chốt: job không chia sẻ filesystem với nhau (mỗi job là VM mới). Muốn job B chạy sau job A, dùng `needs`; muốn truyền dữ liệu (như build artifact) giữa job thì phải `upload-artifact`/`download-artifact`.

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pnpm test

  build:
    needs: test          # build chỉ chạy SAU khi test pass
    runs-on: ubuntu-latest
    steps:
      - run: docker build -t app .

  deploy:
    needs: build         # tuần tự: test -> build -> deploy
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: ./deploy.sh
```

Lưu ý: vì mỗi job là VM mới, nếu `build` job tạo file mà `deploy` job cần thì phải dùng artifact. Với Docker, cách phổ biến là `build` job push image lên registry, `deploy` job pull về — registry đóng vai trò "kênh truyền" giữa job.
</details>

<details>
<summary><b>32.4 Caching trong GitHub Actions hoạt động thế nào? Cache dependency (pnpm/npm) và Docker layer khác nhau ra sao?</b></summary>

Mỗi job chạy trên VM sạch nên mặc định phải cài lại dependency từ đầu — rất chậm. Cache giúp lưu lại thư mục giữa các lần chạy, key dựa trên hash của lockfile.

**Cache dependency**: cách đơn giản nhất là dùng tham số `cache` của `setup-node` (nó tự cache store của pnpm/npm/yarn theo lockfile).

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: pnpm                  # tự động cache & restore theo pnpm-lock.yaml
- run: pnpm install --frozen-lockfile
```

Hoặc cache thủ công với `actions/cache`, key gồm hash lockfile + `restore-keys` để fallback:

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.local/share/pnpm/store
    key: pnpm-${{ hashFiles('pnpm-lock.yaml') }}
    restore-keys: pnpm-
```

**Cache Docker layer**: khác hẳn — đây là cache các layer của image để build lần sau không build lại layer không đổi. Với `docker/build-push-action` thường dùng GitHub Actions cache backend:

```yaml
- uses: docker/build-push-action@v6
  with:
    push: true
    tags: registry/app:latest
    cache-from: type=gha          # đọc layer cache từ GitHub
    cache-to: type=gha,mode=max   # ghi lại toàn bộ layer
```

Điểm cần nhớ: key đúng là yếu tố quyết định. Key nên gắn với hash lockfile — nếu key không đổi khi dependency đổi thì cache sẽ "stale" (cài thiếu package mới). `restore-keys` cho phép tái dùng một phần cache cũ khi không trúng key tuyệt đối.
</details>

<details>
<summary><b>32.5 Matrix strategy trong GitHub Actions dùng để làm gì? Cho ví dụ test NestJS trên nhiều phiên bản Node và xử lý khi một biến thể được phép fail.</b></summary>

**Matrix** cho phép sinh ra nhiều job song song từ một định nghĩa, mỗi job ứng với một tổ hợp giá trị — rất hợp để test trên nhiều phiên bản Node, nhiều OS, hoặc nhiều phiên bản DB.

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      fail-fast: false              # một biến thể fail KHÔNG hủy các biến thể khác
      matrix:
        os: [ubuntu-latest]
        node: [18, 20, 22]
        include:
          - os: ubuntu-latest
            node: 23                # thêm tổ hợp lẻ
        # exclude: loại bỏ tổ hợp không cần
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '${{ matrix.node }}', cache: pnpm }
      - run: pnpm install --frozen-lockfile
      - run: pnpm test
    continue-on-error: ${{ matrix.node == 23 }}  # cho phép Node 23 (mới) fail mà job vẫn xanh
```

Các tham số quan trọng:
- `fail-fast: false`: mặc định khi một biến thể fail thì GitHub hủy hết — đặt `false` để thấy *tất cả* biến thể nào hỏng.
- `include`/`exclude`: thêm/bớt tổ hợp lẻ ngoài tích Descartes.
- `continue-on-error`: cho một biến thể "thử nghiệm" được fail mà không làm hỏng cả workflow.

Ví dụ thực tế: với một thư viện chia sẻ (npm package nội bộ) tôi test trên Node 18/20/22 vì khách hàng dùng nhiều phiên bản; còn service Jira Clone deploy chỉ pin một Node version đúng bằng version production nên không cần matrix Node — chỉ matrix khi thực sự cần đảm bảo tương thích đa môi trường.
</details>

<details>
<summary><b>32.6 Quản lý secret trong GitHub Actions như thế nào? Phân biệt repository secret, environment secret và variable; cảnh báo bảo mật cần lưu ý.</b></summary>

GitHub Actions cung cấp các cấp lưu trữ:

- **Repository secrets** (`secrets.X`): mã hóa, dùng chung cho mọi workflow của repo.
- **Environment secrets**: gắn với một `environment` (vd `production`), chỉ job khai báo `environment: production` mới đọc được — kết hợp được với **required reviewers** và **wait timer**.
- **Variables** (`vars.X`): giá trị **không nhạy cảm**, hiện rõ trong log (vd region, tên service).

```yaml
deploy:
  environment: production         # mở khóa environment secrets + cổng duyệt
  runs-on: ubuntu-latest
  steps:
    - run: ./deploy.sh
      env:
        AWS_REGION: ${{ vars.AWS_REGION }}            # variable: không nhạy cảm
        DB_PASSWORD: ${{ secrets.DB_PASSWORD }}       # secret: được mask trong log
```

Cảnh báo bảo mật:
1. Secret được **tự động mask** trong log nếu in nguyên văn, nhưng nếu bạn biến đổi nó (base64, cắt chuỗi) thì mask không còn hiệu lực — rò rỉ.
2. Với `pull_request` từ fork, secret **không được cấp** (chống PR độc hại đánh cắp secret). Dùng `pull_request_target` rất nguy hiểm vì nó chạy với secret — chỉ dùng khi thật sự hiểu.
3. Ưu tiên **OIDC** thay vì lưu long-lived cloud key: workflow lấy token tạm thời từ AWS/GCP, không cần lưu access key tĩnh trong secret.

```yaml
permissions:
  id-token: write        # cần cho OIDC
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123:role/gha-deploy   # AssumeRole qua OIDC, không cần secret key
    aws-region: ap-southeast-1
```

Ví dụ thực tế: trong Jira Clone tôi để `DATABASE_URL`/`JWT_SECRET` là environment secret riêng cho `staging` và `production` (giá trị khác nhau), còn tên cluster/region là `vars` để đọc thoải mái trong log khi debug.
</details>

<details>
<summary><b>32.7 Viết một multi-stage Dockerfile cho NestJS sao cho image nhỏ, build nhanh và chạy non-root. Giải thích từng stage.</b></summary>

Multi-stage build tách quá trình build (cần devDependencies, TypeScript compiler) khỏi runtime (chỉ cần code đã compile + production deps). Image cuối **không chứa** source TypeScript, devDependencies hay toolchain → nhỏ và ít bề mặt tấn công.

```dockerfile
# ---- Stage 1: deps (cache layer cho dependency) ----
FROM node:20-alpine AS deps
WORKDIR /app
RUN corepack enable
COPY package.json pnpm-lock.yaml ./
RUN pnpm install --frozen-lockfile          # layer này chỉ rebuild khi lockfile đổi

# ---- Stage 2: build ----
FROM node:20-alpine AS build
WORKDIR /app
RUN corepack enable
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN pnpm build                              # nest build -> dist/
RUN pnpm prune --prod                       # bỏ devDependencies, giữ prod deps

# ---- Stage 3: runtime (image nhỏ, non-root) ----
FROM node:20-alpine AS runtime
WORKDIR /app
ENV NODE_ENV=production
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY --from=build /app/package.json ./
USER node                                   # chạy non-root (user 'node' có sẵn trong image)
EXPOSE 3000
CMD ["node", "dist/main.js"]
```

Các kỹ thuật quan trọng:
- **COPY lockfile trước, source sau**: tận dụng Docker layer cache — đổi code không làm rebuild layer `pnpm install`.
- **`pnpm prune --prod`**: loại devDependencies trước khi copy sang runtime.
- **`node:20-alpine`**: base nhỏ (~50MB). Cân nhắc `distroless` cho nhỏ và an toàn hơn nữa.
- **`USER node`**: container không chạy root, giảm rủi ro nếu bị chiếm quyền (defense in depth).
- **`.dockerignore`** (`node_modules`, `dist`, `.git`, `.env`): tránh copy rác vào build context.

Bẫy thường gặp: quên `.dockerignore` khiến `node_modules` của máy host (build cho OS khác) lọt vào image → image phình to và sai binary native (vd `bcrypt`).
</details>

<details>
<summary><b>32.8 So sánh deploy Next.js lên Vercel với tự đóng gói Docker rồi deploy lên cloud. Khi nào chọn cái nào?</b></summary>

**Vercel** là nền tảng được xây riêng cho Next.js: nó hiểu mô hình SSR/SSG/ISR/Edge của Next, tự map từng page sang serverless/edge function, build và CDN hoàn toàn tự động chỉ qua một `git push`.

```bash
# Deploy thủ công bằng CLI (thường chỉ cần kết nối Git là auto-deploy)
vercel --prod
# Mỗi PR có một Preview Deployment (URL riêng) để review
```

| Tiêu chí | Vercel | Docker + Cloud (Cloud Run/ECS) |
|---|---|---|
| Setup | Gần như zero-config | Tự viết Dockerfile + IaC |
| Tính năng Next (ISR, Edge, Image Opt) | Hỗ trợ native | Phải tự lo / dùng standalone output |
| Kiểm soát hạ tầng | Thấp | Cao (network, VPC, sidecar) |
| Long-running / WebSocket | Hạn chế (serverless timeout) | Tốt |
| Chi phí ở quy mô lớn | Có thể đắt | Kiểm soát tốt hơn |

Khi nào chọn:
- **Vercel**: dự án Next.js thuần frontend/BFF, cần preview deploy theo PR, không có process chạy nền dài, ưu tiên tốc độ ship.
- **Docker + cloud**: cần đặt chung VPC với backend, cần WebSocket/long-running, cần kiểm soát chi phí/hạ tầng, hoặc đã có K8s/ECS.

Lưu ý kỹ thuật: nếu muốn Docker hóa Next.js, bật `output: 'standalone'` trong `next.config.js` để Next gom đúng dependency tối thiểu vào `.next/standalone`, giúp Docker image nhỏ hơn nhiều.

Ví dụ thực tế: frontend Jira Clone (Next.js) tôi để Vercel cho preview-per-PR tiện QA; NestJS API thì Docker hóa và deploy cloud vì cần WebSocket realtime cho board — đúng cái Vercel serverless không hợp.
</details>

<details>
<summary><b>32.9 Trình bày cách deploy một container NestJS lên cloud (ví dụ Cloud Run hoặc ECS). Những yêu cầu hạ tầng nào service phải tuân thủ để chạy được?</b></summary>

Mô hình chung của các nền tảng container (Cloud Run, ECS Fargate, Render): bạn cung cấp một image trong registry, nền tảng chạy nó như stateless container, tự scale theo traffic.

Quy trình điển hình (Cloud Run):

```bash
# 1) Build & push image (thường chạy trong CI)
docker build -t asia-docker.pkg.dev/PROJECT/repo/api:$GIT_SHA .
docker push asia-docker.pkg.dev/PROJECT/repo/api:$GIT_SHA

# 2) Deploy revision mới (Cloud Run quản lý rolling + traffic)
gcloud run deploy api \
  --image asia-docker.pkg.dev/PROJECT/repo/api:$GIT_SHA \
  --region asia-southeast1 \
  --port 3000 \
  --set-env-vars NODE_ENV=production \
  --set-secrets DATABASE_URL=db-url:latest   # đọc secret từ Secret Manager
```

Yêu cầu service NestJS phải đáp ứng để chạy tốt trên các nền tảng này:
1. **Listen đúng `PORT`**: nền tảng inject biến `PORT` (Cloud Run dùng 8080); phải `app.listen(process.env.PORT ?? 3000)` thay vì hardcode.
2. **Stateless**: không lưu session/upload trên local disk (container có thể bị giết bất cứ lúc nào); session → Redis, file → object storage (S3/GCS).
3. **Bind `0.0.0.0`**: `app.listen(port, '0.0.0.0')` để nhận traffic từ ngoài container.
4. **Graceful shutdown**: bật `app.enableShutdownHooks()` để đóng kết nối DB/queue khi nhận SIGTERM (xem 32.12).
5. **Health endpoint**: cung cấp `/health` cho liveness/readiness probe.
6. **Cấu hình qua env/secret**, không hardcode.

So sánh ngắn: **Cloud Run** đơn giản nhất (scale-to-zero, trả tiền theo request); **ECS Fargate** hợp khi đã ở AWS, cần kiểm soát network/VPC; **Render** dễ dùng cho team nhỏ, ít cấu hình. Tất cả đều yêu cầu container stateless như trên.
</details>

<details>
<summary><b>32.10 Quản lý biến môi trường và secret theo từng môi trường (dev/staging/prod) như thế nào cho an toàn? `.env` file đóng vai trò gì và không nên làm gì?</b></summary>

Nguyên tắc 12-factor: **config tách khỏi code**, cấp qua environment, **không bao giờ commit secret** vào Git.

Phân lớp theo môi trường:
- **Local dev**: `.env` (đã `.gitignore`) + một `.env.example` commit để chỉ rõ những key cần có (không có giá trị thật).
- **CI**: GitHub environment secrets riêng cho từng env.
- **Production/staging**: secret manager của cloud (AWS Secrets Manager, GCP Secret Manager, Vault) — không để secret nằm phẳng trong env var nếu tránh được.

Trong NestJS dùng `@nestjs/config` với validation để fail sớm khi thiếu/sai biến:

```ts
ConfigModule.forRoot({
  isGlobal: true,
  validationSchema: Joi.object({
    NODE_ENV: Joi.string().valid('development', 'staging', 'production').required(),
    DATABASE_URL: Joi.string().required(),
    JWT_SECRET: Joi.string().min(32).required(),
  }),
  // KHÔNG load .env ở production: ở prod biến đến từ container env/secret manager
  ignoreEnvFile: process.env.NODE_ENV === 'production',
});
```

Những điều **không** nên làm:
- Không commit `.env` thật (chỉ commit `.env.example`); nếu lỡ commit secret, coi như đã lộ → **rotate** ngay, không chỉ xóa file.
- Không in toàn bộ `process.env` ra log.
- Không dùng chung một secret cho mọi môi trường (prod phải có `JWT_SECRET`, `DATABASE_URL` riêng).
- Không nhúng secret vào Docker image (qua `ENV` hay `ARG`) — nó nằm lại trong layer history; inject lúc runtime.

Ví dụ thực tế: Jira Clone có `JWT_SECRET` khác nhau giữa staging/prod (lưu ở Secret Manager), `validationSchema` chặn khởi động nếu thiếu — tránh service "chạy được nhưng auth hỏng" do quên set biến.
</details>

<details>
<summary><b>32.11 Chạy database migration trong pipeline sao cho an toàn? Vì sao không nên dùng `synchronize: true` và cách xử lý migration phá vỡ (breaking) không downtime?</b></summary>

Migration là thao tác **trạng thái** (không idempotent như deploy code), nên cần kỷ luật riêng.

Nguyên tắc cơ bản:
1. **Tắt auto-sync ở mọi môi trường thật**: TypeORM `synchronize: true` (hay Prisma `db push`) so sánh entity với DB rồi tự đổi schema — nó có thể **drop cột/bảng và mất dữ liệu**. Chỉ dùng cho prototype local. Production luôn dùng migration file được version control.
2. **Chạy migration như một bước riêng trước khi switch traffic**, không nhồi vào lúc app khởi động trên nhiều instance (sẽ có race khi N instance cùng chạy migration).

```yaml
# Job migrate chạy TRƯỚC deploy, một lần duy nhất
migrate:
  needs: build
  environment: production
  runs-on: ubuntu-latest
  steps:
    - run: pnpm typeorm migration:run    # hoặc: prisma migrate deploy
      env:
        DATABASE_URL: ${{ secrets.DATABASE_URL }}

deploy:
  needs: migrate      # chỉ deploy app sau khi migrate xong
  runs-on: ubuntu-latest
  steps: [ ... ]
```

**Migration phá vỡ không downtime** — dùng kỹ thuật **expand/contract (parallel change)**: tách thay đổi thành nhiều bước tương thích ngược thay vì một bước phá vỡ. Ví dụ đổi tên cột `name` → `full_name`:
1. **Expand**: thêm cột `full_name`, code ghi cả hai cột, đọc ưu tiên `full_name` (cột cũ vẫn còn → cả version cũ lẫn mới đều chạy).
2. **Backfill**: copy dữ liệu `name` → `full_name`.
3. **Contract**: sau khi mọi instance đã lên version mới, migration sau mới `DROP COLUMN name`.

Các điểm an toàn khác: migration phải **forward-only và đảo ngược được ý niệm** (có kế hoạch rollback), tránh `ALTER TABLE` khóa bảng lâu trên bảng lớn (Postgres: thêm cột có DEFAULT trên bảng khổng lồ, tạo index nên dùng `CREATE INDEX CONCURRENTLY`), và backup/snapshot trước migration rủi ro cao.

Ví dụ thực tế: trong Jira Clone tôi tách `prisma migrate deploy` thành job riêng `needs: build` rồi mới deploy app; khi đổi tên cột tôi đi expand/contract để pod cũ và mới chạy song song không lỗi.
</details>

<details>
<summary><b>32.12 So sánh các chiến lược deploy zero-downtime: rolling, blue-green và canary. Mỗi cái phù hợp khi nào? Vai trò của graceful shutdown?</b></summary>

Cả ba đều nhằm cập nhật mà không cắt service, nhưng đánh đổi khác nhau về tài nguyên và mức độ kiểm soát rủi ro:

| Chiến lược | Cách hoạt động | Ưu | Nhược |
|---|---|---|---|
| **Rolling** | Thay dần từng instance (vài pod một lúc) | Không cần gấp đôi tài nguyên | Trong lúc deploy tồn tại cả 2 version → cần tương thích ngược |
| **Blue-green** | Dựng full môi trường mới (green) song song, switch traffic 100% một nhát | Switch/rollback tức thì | Tốn gấp đôi tài nguyên tạm thời |
| **Canary** | Đẩy traffic dần (5% → 25% → 100%) sang version mới, theo dõi metric | Giới hạn bán kính ảnh hưởng, phát hiện lỗi sớm | Cần routing theo % + monitoring tốt |

Chọn khi nào:
- **Rolling**: mặc định cho hầu hết service stateless (K8s Deployment, Cloud Run đều làm sẵn).
- **Blue-green**: khi cần switch dứt khoát và rollback tức thì (release nhạy cảm), chấp nhận tốn tài nguyên.
- **Canary**: khi thay đổi rủi ro cao và có metric/alert tốt để tự đánh giá phiên bản mới.

**Graceful shutdown** là điều kiện tiên quyết cho mọi chiến lược trên: khi instance cũ bị thay, orchestrator gửi `SIGTERM`. Service phải ngừng nhận request mới, xử lý nốt request đang chạy, đóng DB/queue rồi mới thoát — nếu không, request đang dở sẽ bị cắt giữa chừng (lỗi 502).

```ts
// main.ts — bật shutdown hook để NestJS bắt SIGTERM/SIGINT
const app = await NestFactory.create(AppModule);
app.enableShutdownHooks();

// Provider có thể giải phóng tài nguyên khi tắt
@Injectable()
export class QueueService implements OnApplicationShutdown {
  async onApplicationShutdown(signal: string) {
    await this.worker.close();   // chờ job đang chạy xong rồi đóng worker
  }
}
```

Lưu ý chung: dù dùng chiến lược nào, vì luôn có khoảnh khắc hai version cùng chạy nên **mọi thay đổi phải tương thích ngược** (đặc biệt là DB — xem expand/contract ở 32.11).
</details>

<details>
<summary><b>32.13 Health check và smoke test khác nhau ra sao? Phân biệt liveness và readiness probe, và viết một health endpoint trong NestJS.</b></summary>

Hai loại kiểm tra phục vụ hai mục đích khác nhau:
- **Health check**: endpoint nội bộ để orchestrator quyết định instance có sống/sẵn sàng không.
- **Smoke test**: một tập nhỏ kịch bản nghiệp vụ chạy **sau deploy** để xác nhận hệ thống thực sự hoạt động (login được, tạo được issue...), thường gọi qua HTTP từ ngoài.

**Liveness vs Readiness** (rất hay bị nhầm):
- **Liveness**: "process còn sống không?" Nếu fail → orchestrator **restart** container. Phải nhẹ, không phụ thuộc DB (nếu liveness fail vì DB chết, restart vô ích còn gây restart loop).
- **Readiness**: "đã sẵn sàng nhận traffic chưa?" Nếu fail → **gỡ khỏi load balancer** nhưng không restart. Đây là nơi check dependency (DB, Redis). Quan trọng cho rolling: instance mới chưa kết nối DB xong thì readiness fail → LB chưa gửi traffic.

NestJS dùng `@nestjs/terminus`:

```ts
@Controller('health')
export class HealthController {
  constructor(
    private health: HealthCheckService,
    private db: TypeOrmHealthIndicator,
  ) {}

  @Get('live')                         // liveness: nhẹ, chỉ báo còn sống
  @HealthCheck()
  live() {
    return this.health.check([]);
  }

  @Get('ready')                        // readiness: check DB sẵn sàng
  @HealthCheck()
  ready() {
    return this.health.check([
      () => this.db.pingCheck('database', { timeout: 1500 }),
    ]);
  }
}
```

Smoke test sau deploy trong pipeline:

```yaml
- name: Smoke test
  run: |
    curl -fsS https://staging.api/health/ready          # readiness phải 200
    curl -fsS -X POST https://staging.api/auth/login \
      -d '{"email":"smoke@test.io","password":"..."}' -H 'Content-Type: application/json'
```

Nếu smoke test fail thì pipeline coi như deploy hỏng và kích hoạt rollback. Ví dụ thực tế: Jira Clone có `/health/ready` ping Postgres + Redis cho readiness probe của Cloud Run, và một smoke test login + tạo issue sau deploy staging — nếu fail thì không promote lên prod.
</details>

<details>
<summary><b>32.14 Khi một bản deploy gây lỗi production, cơ chế rollback nên hoạt động thế nào? Tại sao "rollback DB migration" lại nguy hiểm và làm gì để hạn chế?</b></summary>

Rollback tốt nghĩa là quay về **một artifact đã biết là tốt** một cách nhanh và xác định.

Rollback ở tầng ứng dụng:
- **Bất biến image theo digest/SHA**: mỗi build tạo image tag bằng git SHA (không dùng `latest` cho prod). Rollback = redeploy tag cũ.

```bash
# Cloud Run: trỏ 100% traffic về revision tốt trước đó — gần như tức thì
gcloud run services update-traffic api --to-revisions=api-00042-abc=100
# Kubernetes:
kubectl rollout undo deployment/api
```

- **Blue-green** cho rollback nhanh nhất (switch ngược về môi trường cũ); **canary** thì chỉ cần dừng tăng % và rút traffic.

Vì sao **rollback DB migration nguy hiểm**: code có thể quay lui sạch, nhưng schema/dữ liệu thì không. Một migration `DROP COLUMN` đã chạy thì rollback code không khôi phục được dữ liệu trong cột đó. Migration "down" thường được viết qua loa và hiếm khi test, nên chạy ngược trên production rất rủi ro mất dữ liệu.

Cách hạn chế:
1. **Tách deploy code và migration** — và làm migration **tương thích ngược** (expand/contract) để rollback *code* không cần rollback *schema*. Nếu cột mới chỉ được thêm (không drop), version cũ vẫn chạy bình thường khi rollback.
2. **Không drop ngay** — bước contract (drop cột/bảng cũ) chỉ thực hiện ở một deploy sau, khi đã chắc chắn không cần rollback về version cũ nữa.
3. **Backup/snapshot trước migration rủi ro cao**; với thao tác phá hủy, ưu tiên "soft" (đổi tên cột thành `_deprecated`) thay vì DROP cứng.
4. **Feature flag**: tắt tính năng lỗi tức thì mà không cần redeploy/rollback cả service.

Ví dụ thực tế: trong Jira Clone tôi pin image theo git SHA nên rollback chỉ là `update-traffic` về revision cũ trong vài giây; nhờ migration luôn additive (expand/contract) nên rollback code không bao giờ phải rollback schema — đây là lý do tách hai việc này.
</details>

<details>
<summary><b>32.15 Mô tả một pipeline CI/CD GitHub Actions hoàn chỉnh cho service NestJS từ commit đến production, ghép các mảnh đã nói ở trên thành một bức tranh tổng thể.</b></summary>

Dưới đây là khung tổng hợp đầy đủ một quy trình thực tế, thể hiện cách các giai đoạn móc nối:

```yaml
name: CI/CD
on:
  push: { branches: [main] }
  pull_request:

jobs:
  # 1) CI: lint + type + test (chạy cho cả PR) — fail fast
  ci:
    runs-on: ubuntu-latest
    services:
      postgres:                       # service container cho e2e
        image: postgres:16
        env: { POSTGRES_PASSWORD: test }
        ports: ['5432:5432']
        options: >-
          --health-cmd pg_isready --health-interval 10s --health-retries 5
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: pnpm }
      - run: pnpm install --frozen-lockfile
      - run: pnpm lint && pnpm exec tsc --noEmit
      - run: pnpm test --ci --coverage
      - run: pnpm test:e2e
        env: { DATABASE_URL: postgres://postgres:test@localhost:5432/test }

  # 2) Build & push image (chỉ trên main, sau khi ci xanh)
  build:
    needs: ci
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    permissions: { id-token: write, contents: read }   # OIDC, không lưu key
    steps:
      - uses: actions/checkout@v4
      - uses: docker/build-push-action@v6
        with:
          push: true
          tags: registry/api:${{ github.sha }}          # tag bất biến theo SHA
          cache-from: type=gha
          cache-to: type=gha,mode=max

  # 3) Migration chạy MỘT lần trước deploy
  migrate:
    needs: build
    environment: production           # cổng duyệt + environment secrets
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: pnpm prisma migrate deploy
        env: { DATABASE_URL: ${{ secrets.DATABASE_URL }} }

  # 4) Deploy rolling + smoke test, rollback nếu smoke fail
  deploy:
    needs: migrate
    environment: production
    runs-on: ubuntu-latest
    steps:
      - run: gcloud run deploy api --image registry/api:${{ github.sha }} --region asia-southeast1
      - name: Smoke test
        run: curl -fsS https://api.example.com/health/ready
      - name: Rollback on failure
        if: failure()
        run: gcloud run services update-traffic api --to-revisions=LATEST=0
```

Bức tranh tổng thể, đọc theo trình tự nhân quả:
1. **`ci`** chạy cho cả PR — fail fast bằng lint → type → unit → e2e (dùng Postgres service container). Đây là cổng chất lượng.
2. **`build`** chỉ chạy trên `main`, tạo image tag theo **git SHA bất biến**, dùng OIDC để không lưu cloud key, cache layer qua GHA.
3. **`migrate`** chạy **một lần** trước deploy (tránh race đa instance), gắn `environment: production` để có approval; migration đã thiết kế tương thích ngược (expand/contract).
4. **`deploy`** rolling lên Cloud Run, **smoke test** readiness; `if: failure()` rút traffic về revision cũ → **rollback** tự động.

Các sợi chỉ xuyên suốt: tách rõ từng giai đoạn bằng `needs`; secret theo environment; image bất biến giúp rollback xác định; health/smoke là cổng cuối; DB migration luôn additive để rollback code không cần rollback schema. Đây chính là cách tôi tổ chức pipeline cho Jira Clone.
</details>

---

## 33. Tích hợp Third-party và Khả năng chống chịu

<details>
<summary><b>33.1 Khi gọi một API bên ngoài, những lớp bảo vệ tối thiểu nào bạn luôn áp dụng và vì sao?</b></summary>

Mọi cuộc gọi mạng ra ngoài đều có thể chậm, lỗi, hoặc treo vô hạn. Một client "trần" không có gì bảo vệ sẽ kéo theo cả service của bạn chết (cascading failure). Bốn lớp tối thiểu:

- **Timeout**: bắt buộc. Không có timeout thì một request treo sẽ giữ socket/connection và làm cạn pool. Nên có cả **connect timeout** và **total/request timeout**.
- **Retry có kiểm soát**: chỉ retry với lỗi tạm thời (network error, 502/503/504, 429), và chỉ với request **idempotent** (GET, PUT, DELETE, hoặc POST có idempotency key).
- **Circuit breaker**: khi provider lỗi liên tục thì ngừng gọi một thời gian để khỏi "đập" vào hệ thống đang chết.
- **Bulkhead / giới hạn concurrency**: cô lập tài nguyên cho từng provider để một provider chậm không nuốt hết worker.

```ts
// NestJS HttpModule (axios) cấu hình timeout mặc định
HttpModule.register({
  timeout: 5000,        // total timeout 5s
  maxRedirects: 2,
});

// Hoặc với fetch/undici + AbortController
async function callExternal(url: string) {
  const ac = new AbortController();
  const t = setTimeout(() => ac.abort(), 5000);
  try {
    const res = await fetch(url, { signal: ac.signal });
    if (!res.ok) throw new Error(`Upstream ${res.status}`);
    return await res.json();
  } finally {
    clearTimeout(t); // luôn dọn timer, tránh leak
  }
}
```

Lỗi thường gặp: đặt timeout quá dài (30s) khiến request treo vẫn giữ tài nguyên rất lâu; hoặc không có timeout vì "thư viện chắc có sẵn" — thực tế nhiều client mặc định là vô hạn.

Ví dụ thực tế: trong Jira Clone, mọi call sang dịch vụ email/notification đều bọc qua một `ResilientHttpService` đặt timeout 5s và đo thời gian phản hồi bằng pino.
</details>

<details>
<summary><b>33.2 Giải thích retry với exponential backoff và jitter. Vì sao jitter lại quan trọng?</b></summary>

**Retry** đơn thuần (thử lại ngay lập tức) rất nguy hiểm: nếu provider đang quá tải, hàng loạt client thử lại cùng lúc sẽ "đánh" tiếp làm nó càng chết — gọi là **retry storm**.

- **Exponential backoff**: thời gian chờ tăng theo hàm mũ: 1s, 2s, 4s, 8s... (`base * 2^attempt`), giúp giảm áp lực dần.
- **Jitter**: thêm yếu tố ngẫu nhiên vào thời gian chờ. Nếu không có jitter, tất cả client retry tại đúng các mốc 1s/2s/4s nên vẫn ập vào cùng lúc (thundering herd). Jitter trải đều các lần thử ra theo thời gian.

```ts
// Full jitter (kiểu AWS khuyến nghị)
async function retryWithBackoff<T>(
  fn: () => Promise<T>,
  { retries = 3, base = 200, cap = 5000 } = {},
): Promise<T> {
  let attempt = 0;
  for (;;) {
    try {
      return await fn();
    } catch (err) {
      if (attempt >= retries || !isRetryable(err)) throw err;
      const exp = Math.min(cap, base * 2 ** attempt);
      const delay = Math.random() * exp; // full jitter: random trong [0, exp]
      await new Promise((r) => setTimeout(r, delay));
      attempt++;
    }
  }
}
```

| Chiến lược | Hành vi | Vấn đề |
|---|---|---|
| Retry ngay | thử lại tức thì | retry storm, làm provider chết hẳn |
| Exponential, không jitter | 1s/2s/4s đồng loạt | thundering herd, đồng bộ hóa lỗi |
| Exponential + full jitter | random trong [0, exp] | trải đều, được khuyến nghị |

Lưu ý: tổng số lần retry và `cap` phải gói gọn trong timeout tổng của request gọi đến bạn (ví dụ HTTP request từ client), nếu không client phía trên đã timeout từ lâu mà bạn vẫn retry vô ích.
</details>

<details>
<summary><b>33.3 Idempotency là gì và vì sao bắt buộc khi retry các thao tác gây side-effect (như thanh toán)?</b></summary>

**Idempotent** nghĩa là gọi cùng một thao tác nhiều lần cho ra cùng một kết quả như gọi một lần — không tạo ra side-effect lặp. Vấn đề cốt lõi: khi bạn gọi `POST /charges` rồi bị **timeout**, bạn không biết provider đã xử lý hay chưa. Nếu retry mù quáng, có thể **trừ tiền khách hai lần**.

Giải pháp chuẩn: gửi một **Idempotency-Key** (thường là UUID) trong header. Provider lưu key đó; nếu thấy key đã xử lý thì trả về kết quả cũ thay vì thực thi lại.

```ts
import { randomUUID } from 'crypto';

async function charge(amount: number, orderId: string) {
  // Key gắn với nghiệp vụ (orderId) -> retry bao nhiêu lần cũng cùng key
  const idempotencyKey = `charge:${orderId}`;
  return stripe.paymentIntents.create(
    { amount, currency: 'usd' },
    { idempotencyKey }, // Stripe SDK dùng lại kết quả nếu key trùng
  );
}
```

Phía **server của bạn nhận request** cũng nên hỗ trợ idempotency cho các POST nhạy cảm: lưu `idempotency_key` (unique) cùng response đã trả; request thứ hai cùng key thì trả lại response cũ.

```sql
CREATE TABLE idempotency_keys (
  key         TEXT PRIMARY KEY,
  response    JSONB,
  created_at  TIMESTAMPTZ DEFAULT now()
);
```

Điểm cần nhớ: key phải sinh từ phía gọi và **ổn định qua các lần retry** (đừng `randomUUID()` mới mỗi lần retry, vì như vậy mỗi lần là một thao tác mới). Nên gắn key với một định danh nghiệp vụ (orderId, requestId).

Ví dụ thực tế: trong Jira Clone, endpoint tạo invoice gửi idempotency-key = `invoice:{projectId}:{period}` để job chạy lại không tạo trùng hóa đơn.
</details>

<details>
<summary><b>33.4 Circuit breaker hoạt động thế nào và vì sao retry không thôi là chưa đủ?</b></summary>

Retry giải quyết lỗi **thoáng qua** (một request lỗi lẻ tẻ). Nhưng khi provider **chết hẳn** trong vài phút, retry chỉ làm mọi thứ tệ hơn: mỗi request đợi timeout 5s rồi retry, làm cạn thread/connection và lỗi lan ngược vào hệ thống của bạn.

**Circuit breaker** (cầu dao) theo dõi tỉ lệ lỗi và có 3 trạng thái:

- **Closed**: bình thường, cho request đi qua. Đếm lỗi.
- **Open**: khi lỗi vượt ngưỡng, "ngắt mạch" — **fail fast** ngay lập tức không gọi provider trong khoảng `resetTimeout`. Tránh phí tài nguyên chờ timeout.
- **Half-open**: hết `resetTimeout`, cho một vài request "thăm dò" đi qua. Nếu thành công → đóng lại (Closed); nếu vẫn lỗi → mở lại (Open).

```ts
import CircuitBreaker from 'opossum';

const breaker = new CircuitBreaker(callPaymentProvider, {
  timeout: 4000,                 // coi như lỗi nếu quá 4s
  errorThresholdPercentage: 50,  // >50% lỗi thì Open
  resetTimeout: 30_000,          // 30s sau thử half-open
});

breaker.fallback(() => ({ status: 'queued', note: 'provider down' }));

breaker.on('open', () => logger.warn('Payment breaker OPEN'));

await breaker.fire(orderId); // fail fast khi đang Open
```

So sánh ngắn: **timeout** giới hạn 1 request; **retry** xử lý lỗi lẻ tẻ; **circuit breaker** bảo vệ khi lỗi diện rộng/kéo dài. Ba thứ bổ trợ nhau, không thay thế nhau.

Lỗi thường gặp: đặt `errorThresholdPercentage` quá thấp khiến breaker mở liên tục vì vài lỗi vặt; hoặc dùng chung một breaker cho nhiều provider khác nhau (nên mỗi provider/endpoint một breaker).
</details>

<details>
<summary><b>33.5 Cách nhận webhook ĐÚNG: trình tự xử lý nào bạn áp dụng để vừa an toàn vừa không mất event?</b></summary>

Webhook là endpoint công khai do provider gọi tới. Một handler tốt phải xử lý đúng theo trình tự:

1. **Verify chữ ký** trước khi làm bất cứ gì — chống giả mạo (xem 33.6). Cần **raw body** để tính HMAC, nên phải tắt JSON parser cho route này.
2. **Idempotency**: dùng `event.id` của provider làm khóa; nếu đã xử lý thì bỏ qua. Provider thường gửi lặp khi không nhận được 2xx.
3. **Trả 2xx thật nhanh**, rồi **xử lý nặng async** qua queue. Nếu bạn xử lý đồng bộ và chậm, provider sẽ timeout và gửi lại → trùng event, hoặc tệ hơn là disable webhook.
4. Phân biệt rõ: lỗi do request sai (signature hỏng) → trả 4xx (đừng để provider retry vô ích); lỗi do mình (DB down) → trả 5xx để provider retry sau.

```ts
@Controller('webhooks')
export class WebhookController {
  constructor(private readonly queue: Queue) {}

  @Post('stripe')
  @HttpCode(200)
  async handle(@Req() req: RawBodyRequest<Request>, @Headers('stripe-signature') sig: string) {
    let event: Stripe.Event;
    try {
      // verify dùng raw body, KHÔNG dùng body đã parse
      event = stripe.webhooks.constructEvent(req.rawBody, sig, process.env.STRIPE_WEBHOOK_SECRET);
    } catch {
      throw new BadRequestException('Invalid signature'); // 400, provider không cần retry
    }

    // Idempotency: chèn event.id; nếu trùng thì đã xử lý rồi -> bỏ qua
    const inserted = await this.markSeen(event.id);
    if (inserted) {
      await this.queue.add('process-stripe-event', { id: event.id, type: event.type });
    }
    return { received: true }; // trả 2xx ngay, xử lý nặng để worker làm
  }
}
```

Để có `req.rawBody` trong NestJS: bật `rawBody: true` khi `NestFactory.create(AppModule, { rawBody: true })` và đảm bảo body parser không "ăn" mất raw bytes của route webhook.

Ví dụ thực tế: trong Blog/CMS API, webhook thanh toán chỉ ghi event vào bảng `webhook_events` rồi đẩy BullMQ job; toàn bộ logic kích hoạt gói premium chạy trong worker.
</details>

<details>
<summary><b>33.6 Verify chữ ký webhook bằng HMAC như thế nào, và vì sao phải dùng so sánh "timing-safe"?</b></summary>

Provider ký payload bằng một **shared secret** dùng HMAC (thường SHA-256) và gửi chữ ký trong header. Bạn tính lại HMAC trên **raw body** với cùng secret rồi so sánh. Nếu khớp → chắc chắn payload do provider gửi và chưa bị sửa.

Hai điểm bắt buộc:

- So sánh bằng **`crypto.timingSafeEqual`**, KHÔNG dùng `===`. So sánh chuỗi thường thoát ra ở ký tự khác đầu tiên, để lộ thời gian → kẻ tấn công có thể đoán dần chữ ký (timing attack).
- Kiểm tra **timestamp** (nếu provider gửi kèm) để chống **replay** — từ chối event quá cũ (ví dụ > 5 phút).

```ts
import { createHmac, timingSafeEqual } from 'crypto';

function verifySignature(rawBody: Buffer, signature: string, secret: string): boolean {
  const expected = createHmac('sha256', secret).update(rawBody).digest('hex');
  const a = Buffer.from(expected);
  const b = Buffer.from(signature);
  // timingSafeEqual yêu cầu cùng độ dài, nếu khác coi như fail
  return a.length === b.length && timingSafeEqual(a, b);
}
```

Lỗi thường gặp:

- Tính HMAC trên **body đã JSON.parse rồi stringify lại** — thứ tự key/khoảng trắng đổi nên chữ ký không bao giờ khớp. Phải dùng raw bytes y hệt provider đã ký.
- Lộ secret trong log hoặc commit vào repo (xem 33.8).
- Không kiểm tra timestamp → kẻ tấn công bắt lại một webhook hợp lệ cũ và phát lại.
</details>

<details>
<summary><b>33.7 Provider trả 429 (rate limit). Bạn xử lý ra sao và Retry-After đóng vai trò gì?</b></summary>

`429 Too Many Requests` nghĩa là bạn gọi quá hạn mức của provider. Cách xử lý:

- **Tôn trọng header `Retry-After`**: provider nói rõ phải chờ bao lâu (số giây, hoặc một mốc HTTP-date). Backoff tự tính của bạn nên **nhường** cho giá trị này — đừng thử lại sớm hơn.
- Nếu không có `Retry-After`, dùng exponential backoff + jitter (33.2).
- **Chủ động giới hạn tốc độ phía mình** (client-side rate limiting / token bucket) để không chạm trần ngay từ đầu — tốt hơn là cứ đâm vào rồi mới lùi.
- Với khối lượng lớn: gom vào **queue có concurrency giới hạn** để rải đều thay vì bùng nổ.

```ts
async function callWithRateLimit(fn: () => Promise<Response>) {
  for (let attempt = 0; attempt < 5; attempt++) {
    const res = await fn();
    if (res.status !== 429) return res;

    const ra = res.headers.get('retry-after');
    // Retry-After có thể là số giây hoặc HTTP-date
    const waitMs = ra
      ? (Number.isNaN(Number(ra)) ? new Date(ra).getTime() - Date.now() : Number(ra) * 1000)
      : Math.min(30_000, 500 * 2 ** attempt) * Math.random();

    await new Promise((r) => setTimeout(r, Math.max(0, waitMs)));
  }
  throw new Error('Rate limited: hết số lần thử');
}
```

| Tín hiệu | Phản ứng đúng |
|---|---|
| 429 + Retry-After: 30 | chờ đúng 30s rồi thử lại |
| 429 không kèm header | exponential backoff + jitter |
| 429 lặp lại liên tục | bật circuit breaker, giảm concurrency |

Ví dụ thực tế: khi gửi email hàng loạt qua **Resend**, tôi đặt BullMQ rate limiter (`limiter: { max, duration }`) để giữ dưới hạn mức của provider, và worker vẫn đọc `Retry-After` khi gặp 429 để lùi đúng thời gian.
</details>

<details>
<summary><b>33.8 Lưu và xoay (rotate) API key/secret an toàn như thế nào trong một service NestJS?</b></summary>

Nguyên tắc cốt lõi: **secret không bao giờ nằm trong source code hay log**, và phải xoay được mà không downtime.

Cách lưu:

- Không hardcode, không commit. Dùng biến môi trường, nạp qua `@nestjs/config`. File `.env` chỉ cho local và phải nằm trong `.gitignore`.
- Production: dùng **secret manager** (AWS Secrets Manager, GCP Secret Manager, Vault) hoặc secret của orchestrator (Kubernetes Secret). Mã hóa at-rest, kiểm soát truy cập (IAM).
- Validate sự tồn tại của secret lúc khởi động (fail fast) bằng schema validation của config — thiếu secret thì app không boot, hơn là chết giữa chừng khi gọi provider.

```ts
// validation lúc boot: thiếu key thì app không start
ConfigModule.forRoot({
  validationSchema: Joi.object({
    STRIPE_SECRET_KEY: Joi.string().required(),
    STRIPE_WEBHOOK_SECRET: Joi.string().required(),
  }),
});
```

Xoay (rotation) không downtime — chấp nhận **2 key cùng lúc** trong giai đoạn chuyển:

1. Tạo key mới ở provider, đẩy vào secret manager (giữ cả key cũ).
2. Service đọc cả hai; verify webhook thử lần lượt cả `WEBHOOK_SECRET_CURRENT` và `WEBHOOK_SECRET_OLD`.
3. Sau khi mọi instance đã nạp key mới và lưu lượng dùng key mới, **revoke** key cũ.

Lỗi thường gặp: in `Authorization` header hoặc cả config object ra log; nhúng secret vào URL (lưu vào access log); để một key duy nhất nên không thể xoay mà không downtime. Nên có **redaction** trong logger (pino `redact: ['req.headers.authorization', 'config.stripeSecretKey']`).
</details>

<details>
<summary><b>33.9 Outbox pattern là gì và nó giải quyết vấn đề gì khi tích hợp với hệ thống bên ngoài/queue?</b></summary>

Vấn đề **dual write**: bạn vừa muốn ghi DB vừa muốn publish event (gọi provider / đẩy queue). Hai thao tác này nằm ở hai hệ thống khác nhau nên không thể bọc trong một transaction. Có hai cách hỏng:

- Commit DB xong rồi publish, nhưng publish lỗi → **mất event** (DB nói "đã tạo order" nhưng không ai biết để gửi email/charge).
- Publish trước rồi DB lỗi → **event ma** cho thứ chưa tồn tại.

**Outbox pattern**: ghi event vào một bảng `outbox` **trong cùng transaction** với thay đổi nghiệp vụ. Sau đó một tiến trình riêng (relay/poller hoặc CDC) đọc bảng outbox và publish ra queue/provider, đánh dấu đã gửi. Vì ghi nghiệp vụ và ghi outbox cùng atomic, **không bao giờ mất event**.

```ts
// Ghi nghiệp vụ + outbox trong CÙNG transaction (Prisma)
await prisma.$transaction(async (tx) => {
  const order = await tx.order.create({ data: dto });
  await tx.outbox.create({
    data: {
      id: randomUUID(),
      type: 'order.created',
      payload: order,         // sẽ publish sau
      status: 'PENDING',
    },
  });
});

// Relay chạy nền: đọc PENDING, publish, đánh dấu SENT (at-least-once)
const events = await prisma.outbox.findMany({ where: { status: 'PENDING' }, take: 100 });
for (const e of events) {
  await queue.add(e.type, e.payload, { jobId: e.id }); // jobId chống trùng
  await prisma.outbox.update({ where: { id: e.id }, data: { status: 'SENT' } });
}
```

Lưu ý quan trọng: outbox đảm bảo **at-least-once** (event có thể publish trùng nếu relay crash giữa publish và update). Vì vậy **consumer phải idempotent** (33.3). Đây là lý do outbox và idempotency luôn đi cặp với nhau.

Ví dụ thực tế: trong Jira Clone, tạo issue và event `issue.assigned` ghi chung transaction vào outbox; một BullMQ producer poll outbox để bắn notification — kể cả khi Redis tạm chết lúc tạo issue thì event vẫn không mất.
</details>

<details>
<summary><b>33.10 Fallback và graceful degradation: khi một provider chết, service của bạn cư xử ra sao?</b></summary>

**Graceful degradation** nghĩa là khi một dependency không quan trọng (non-critical) chết, hệ thống **giảm chức năng** chứ không sập hoàn toàn. Quyết định fallback phụ thuộc tính chất nghiệp vụ:

- **Non-critical** (gợi ý, thumbnail, gửi email thông báo): trả về giá trị mặc định / bỏ qua / cache cũ, vẫn cho user dùng tính năng chính.
- **Critical nhưng có thể trì hoãn** (gửi email, ghi analytics): **enqueue** để xử lý sau khi provider sống lại — không chặn luồng chính.
- **Critical và không thể giả** (thanh toán): không được "giả vờ thành công". Phải báo lỗi rõ ràng cho user và đảm bảo không tạo dữ liệu sai.

```ts
async function getRecommendations(userId: string): Promise<Item[]> {
  try {
    return await breaker.fire(userId);       // gọi recommender service
  } catch {
    logger.warn('Recommender down, dùng fallback');
    return this.cache.get(`reco:${userId}`)  // 1) cache cũ
        ?? await this.popularItems();        // 2) danh sách phổ biến (degrade)
  }
}
```

| Loại dependency | Khi chết | Chiến lược |
|---|---|---|
| Gợi ý/thumbnail | non-critical | default value / cache / bỏ qua |
| Email/notification | trì hoãn được | enqueue, retry sau |
| Lưu trữ file (S3) | critical đọc | serve cache/CDN; ghi thì queue |
| Thanh toán | critical | fail rõ ràng, không giả thành công |

Kết hợp với circuit breaker: `breaker.fallback(...)` chính là nơi cài graceful degradation, để khi breaker Open thì trả ngay giá trị thay thế thay vì lỗi.

Ví dụ thực tế: trong Blog/CMS API, khi **S3** lỗi lúc upload ảnh bìa, bài viết vẫn lưu được (ảnh để trống + job retry upload sau), thay vì chặn toàn bộ thao tác tạo bài.
</details>

<details>
<summary><b>33.11 Khi nào dùng SDK chính thức của provider, khi nào tự gọi raw HTTP client?</b></summary>

**SDK** là client do provider phát hành (ví dụ `stripe`, `@aws-sdk/client-s3`, `resend`); **raw HTTP** là tự gọi qua `fetch`/`axios`/`undici`.

| Tiêu chí | SDK chính thức | Raw HTTP |
|---|---|---|
| Tốc độ phát triển | Nhanh, có sẵn types, helper (vd verify webhook) | Phải tự viết nhiều |
| Tính năng có sẵn | Retry, idempotency, pagination, auth refresh | Tự cài hết |
| Kiểm soát | Bị "che" hành vi (timeout, retry mặc định) | Toàn quyền |
| Phụ thuộc & bundle | Thêm dependency, dễ outdated/CVE | Gọn, ít dependency |
| Endpoint mới/beta | Có thể chưa hỗ trợ | Gọi được ngay |

Nguyên tắc chọn:

- **Mặc định dùng SDK** cho các provider phức tạp/nhạy cảm (thanh toán như Stripe, AWS) — họ đã cài sẵn idempotency, retry, helper verify chữ ký webhook đúng chuẩn, tự viết lại dễ sai về bảo mật.
- **Dùng raw HTTP** khi: API rất đơn giản (một REST endpoint), SDK quá nặng/không có cho ngôn ngữ, hoặc bạn cần kiểm soát chính xác timeout/retry/circuit breaker mà SDK che mất.

```ts
// Dù dùng SDK, vẫn nên ghi đè timeout/retry để hợp với policy của mình
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  timeout: 5000,
  maxNetworkRetries: 2,   // idempotency được SDK tự xử lý cho các call an toàn
});
```

Điểm cần nhớ: dù chọn SDK, đừng tin mặc định một cách mù quáng — vẫn kiểm tra và cấu hình lại timeout/retry, và bọc trong circuit breaker/bulkhead của bạn. SDK lo phần protocol; resilience policy là trách nhiệm của bạn.

Ví dụ thực tế: trong Jira Clone dùng `stripe` SDK (vì verify webhook và idempotency quá quan trọng để tự làm), nhưng gọi một internal microservice nhỏ thì dùng `HttpService` (axios) thẳng vì chỉ cần một endpoint.
</details>
