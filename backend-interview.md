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
30. [MongoDB và Mongoose (NoSQL)](#30-mongodb-và-mongoose-nosql)
31. [Git Workflow và Cộng tác](#31-git-workflow-và-cộng-tác)
32. [CI/CD và Triển khai (Docker, Cloud)](#32-cicd-và-triển-khai-docker-cloud)
33. [Tích hợp Third-party và Khả năng chống chịu](#33-tích-hợp-third-party-và-khả-năng-chống-chịu)
34. [SQL Thuần và Thiết kế CSDL](#34-sql-thuần-và-thiết-kế-csdl)

---

## 1. Node.js Core & Event Loop

<details>
<summary><b>1.1 Node.js là gì? Khác gì với JavaScript chạy trên trình duyệt?</b></summary>

- **Runtime** chạy JavaScript ngoài trình duyệt, xây trên engine **V8** + thư viện **libuv** (event loop & thread pool cho I/O) + **bindings** (JS ↔ C++).
- Node có API hệ thống (`fs`, `http`, `process`, `Buffer`) thay cho `window`, `document`, `DOM`.
- Mô hình: **single-threaded, non-blocking I/O, event-driven** → hợp ứng dụng I/O-bound (API, realtime).

</details>

<details>
<summary><b>1.2 Node single-threaded mà sao xử lý được hàng nghìn request đồng thời?</b></summary>

- JavaScript chạy trên **một thread chính**.
- Các tác vụ **I/O** (DB, API, file) được giao cho OS / thread pool của **libuv** xử lý nền, **không chặn** thread chính.
- I/O xong → callback đẩy vào hàng đợi, **Event Loop** chạy khi thread chính rảnh.
- Ví dụ: đầu bếp bỏ mì vào nồi rồi phục vụ khách khác thay vì đứng đợi.

</details>

<details>
<summary><b>1.3 Thứ tự chạy: process.nextTick vs Promise.then vs setTimeout vs setImmediate?</b></summary>

- **Đồng bộ chạy hết trước** → **microtask** (`process.nextTick` ưu tiên cao nhất → rồi `Promise`) → **macrotask** (`setTimeout`, `setImmediate`, I/O).
- Sau **mỗi** macrotask, dọn sạch toàn bộ microtask rồi mới sang macrotask kế.

```js
console.log('A');                                // đồng bộ
setTimeout(() => console.log('E'), 0);           // macrotask
setImmediate(() => console.log('F'));            // macrotask
Promise.resolve().then(() => console.log('D'));  // microtask
process.nextTick(() => console.log('C'));        // microtask ưu tiên cao
console.log('B');                                // đồng bộ
// In ra: A, B, C, D, rồi E/F (E/F không xác định ở main module)
```

</details>

<details>
<summary><b>1.4 Khi nào KHÔNG nên dùng Node.js?</b></summary>

- Với tác vụ **CPU-bound** nặng (xử lý ảnh/video, mã hoá, tính toán lớn).
- Lý do: chúng **chặn event loop** (thread duy nhất) → cả server đứng.
- Giải pháp: Worker Threads, child process, hàng đợi job, hoặc ngôn ngữ khác cho phần nặng.

</details>

---

## 2. NestJS Architecture & Dependency Injection

<details>
<summary><b>2.1 Dependency Injection (DI) là gì? Lợi ích?</b></summary>

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

- **Controller**: xử lý HTTP & routing — "mỏng", chỉ nhận request, trả response.
- **Service (Provider)**: chứa **business logic** — "dày".
- Nguyên tắc: *Controller mỏng, Service dày* → tách bạch, dễ test, dễ tái dùng.
</details>

<details>
<summary><b>2.3 Module trong NestJS để làm gì?</b></summary>

- Là "chiếc hộp" gom nhóm các thành phần liên quan.
- Gồm: `controllers`, `providers`, `imports` (mượn module khác), `exports` (cho module khác dùng provider của mình).
- App Nest = tập hợp nhiều module.
</details>

<details>
<summary><b>2.5 Module encapsulation: làm sao dùng provider của module khác?</b></summary>

- Provider chỉ "sống" trong module khai báo nó.
- Muốn module B dùng provider của A: **A `exports`** provider, **B `imports`** module A.
- Ví dụ: `UsersModule` exports `UsersService` để `AuthModule` import & dùng.
</details>

<details>
<summary><b>2.6 Scope mặc định của provider là gì?</b></summary>

- Mặc định **Singleton** (`DEFAULT`): tạo **một lần**, dùng chung toàn app.
- `REQUEST`: mỗi request một instance.
- `TRANSIENT`: mỗi nơi inject một instance.
</details>

---

## 3. Configuration & Environment

<details>
<summary><b>3.1 Quản lý biến môi trường/secret trong NestJS thế nào?</b></summary>

- Dùng `@nestjs/config` (`ConfigModule`) đọc file `.env`, truy cập qua `ConfigService.get('KEY')`.
- `.env` phải được **gitignore**, không hardcode secret vào code.
- Commit `.env.example` (chỉ tên biến, không giá trị) làm mẫu cho team.

</details>

<details>
<summary><b>3.2 <code>forRoot</code> vs <code>forRootAsync</code> khác gì?</b></summary>

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

- Tự động `CREATE/ALTER` bảng cho khớp entity **mỗi lần khởi động**.
- Tiện khi dev, nhưng 🔴 **KHÔNG** dùng production: có thể xoá cột → **mất dữ liệu thật**.
- Production dùng **migrations** (script đổi schema có kiểm soát, review & rollback được).

</details>

<details>
<summary><b>3.4 Biến môi trường được đọc khi nào? Sửa <code>.env</code> lúc app đang chạy có ăn không?</b></summary>

- Đọc **một lần** lúc tiến trình khởi động (`ConfigModule.forRoot()` khi bootstrap).
- Sửa `.env` lúc app đang chạy **KHÔNG** có hiệu lực.
- `nest start --watch` chỉ theo dõi file `.ts`, không reload `.env` → phải **restart** app.

</details>

---

## 4. Database, TypeORM & Entities

<details>
<summary><b>4.2 Active Record vs Data Mapper? NestJS dùng cái nào?</b></summary>

- **Active Record**: entity tự lưu (`user.save()`), logic nằm trên model.
- **Data Mapper**: `Repository` riêng lo lưu (`repo.save(user)`), entity là data thuần.
- TypeORM hỗ trợ cả hai; NestJS dùng **Data Mapper / Repository pattern** vì hợp **DI**, tách logic, dễ test.

</details>

<details>
<summary><b>4.3 UUID vs auto-increment (serial) cho khóa chính?</b></summary>

- **UUID**: duy nhất toàn cục, khó đoán (chống dò ID), hợp hệ phân tán. Nhược: 16 byte, không tuần tự → phân mảnh index (`uuidv7` khắc phục).
- **Serial**: nhỏ gọn, tuần tự, nhanh. Nhược: dễ đoán, khó merge giữa nhiều DB.

</details>

<details>
<summary><b>4.4 <code>forRoot</code> vs <code>forFeature</code> trong TypeOrmModule?</b></summary>

- `forRootAsync` (ở AppModule): cấu hình **kết nối** DB, 1 lần, global.
- `forFeature([Entity])` (ở module con): đăng ký **repository** cho entity cụ thể.
- Nhờ `forFeature` mới inject được `Repository<Entity>` trong service module đó.

</details>

<details>
<summary><b>4.5 Vì sao đặt <code>select: false</code> cho cột password?</b></summary>

- Mặc định các query `SELECT` (`find`, `findOne`) **không trả** cột password → tránh rò rỉ hash.
- Khi cần (login): dùng QueryBuilder `.addSelect('user.password')`.
- **Lưu ý**: `@Exclude` + `ClassSerializerInterceptor` là **lớp phòng thủ thứ hai** cho trường hợp chủ động `addSelect` rồi lỡ trả object ra response.

</details>

<details>
<summary><b>4.7 Cột enum trong Postgres có gì đặc biệt?</b></summary>

- Postgres tạo một **kiểu enum riêng** (vd `users_role_enum`).
- Thêm/bớt giá trị enum sau này thường phải qua **migration** (không dễ như varchar).

</details>

<details>
<summary><b>4.9 <code>@InjectRepository</code> hoạt động được nhờ đâu?</b></summary>

- Nhờ `TypeOrmModule.forFeature([Entity])` đã đăng ký **repository token** cho entity vào DI container.
- Thiếu `forFeature` → lỗi `Nest can't resolve dependencies`.

</details>

<details>
<summary><b>4.10 <code>create()</code> vs <code>save()</code> trong Repository?</b></summary>

- `repo.create(data)`: chỉ tạo **instance trong bộ nhớ** (áp default), chưa đụng DB.
- `repo.save(entity)`: thực sự ghi DB — `INSERT`, hoặc `UPDATE` nếu có id.

</details>

<details>
<summary><b>4.11 <code>preload</code> làm gì? <code>remove</code> vs <code>delete</code>?</b></summary>

- `preload({ id, ...dto })`: tải bản ghi theo id rồi **merge** dữ liệu mới, trả entity đã merge (hoặc `undefined`) → tiện PATCH + check 404.
- `remove(entity)`: cần entity đã load, chạy lifecycle hook.
- `delete(criteria)`: xoá thẳng theo điều kiện/id, nhanh hơn, không cần load.

</details>

<details>
<summary><b>4.12 Soft delete là gì?</b></summary>

- Không xoá hẳn dòng, chỉ set cột `deletedAt` (`@DeleteDateColumn` + `softRemove`/`softDelete`).
- Giữ được lịch sử, khôi phục được.
- Query mặc định tự bỏ qua bản ghi đã soft-delete.

</details>

---

## 5. DTO, Validation & Pipes

<details>
<summary><b>5.1 DTO khác Entity thế nào?</b></summary>

- **DTO** (Data Transfer Object): mô tả hình dạng + luật của dữ liệu **vào/ra** — hợp đồng API.
- **Entity**: ánh xạ **bảng DB**.
- Tách biệt để API và DB không phụ thuộc cứng: đổi DB không vỡ API và ngược lại.

</details>

<details>
<summary><b>5.2 Pipe trong NestJS là gì? Chạy lúc nào?</b></summary>

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

- `whitelist`: **âm thầm xoá** field không khai báo trong DTO.
- `forbidNonWhitelisted`: gửi field lạ → **báo lỗi 400**.
- `transform`: chuyển payload thô thành **instance DTO** + ép kiểu (vd `"5"` → `5`).

</details>

<details>
<summary><b>5.4 <code>PartialType</code> dùng để làm gì?</b></summary>

- Kế thừa **toàn bộ luật validation** nhưng biến mọi field thành **optional**.
- Hợp cho `PATCH` (cập nhật một phần).
- Giúp **DRY**, không lặp lại `@IsEmail`, `@MinLength`...

Ví dụ: `PartialType(CreateUserDto)`.

</details>

<details>
<summary><b>5.6 Vì sao truyền id sai định dạng lại ra 500 thay vì 404?</b></summary>

- Param sai kiểu cột (chuỗi không phải UUID vào cột `uuid`) → truy vấn **vỡ ngay ở tầng DB** (`invalid input syntax for type uuid`) → 500.
- Lỗi xảy ra **trước** logic "không tìm thấy → 404".
- Khắc phục: `ParseUUIDPipe` chặn param sai → **400** trước khi chạm DB.

</details>

---

## 6. REST API & Controllers

<details>
<summary><b>6.2 PUT vs PATCH?</b></summary>

- **PUT**: thay **toàn bộ** resource (gửi đủ mọi field).
- **PATCH**: cập nhật **một phần** (chỉ field cần đổi).
- Trong Nest thường dùng PATCH + `PartialType` cho DTO.

</details>

<details>
<summary><b>6.4 Nested route (route lồng nhau) dùng khi nào?</b></summary>

- Khi một tài nguyên **phụ thuộc** tài nguyên khác (vd comment thuộc post).
- URL phản ánh quan hệ theo chuẩn REST; lấy id cha qua `@Param('postId')`.
- Ví dụ: `@Controller('posts/:postId/comments')` -> `POST /posts/<id>/comments` thêm comment vào bài đó.

</details>

---

## 7. Authentication & Authorization

<details>
<summary><b>7.1 Authentication vs Authorization?</b></summary>

- **Authentication (AuthN)**: "bạn **là ai**?" — xác minh danh tính (đăng nhập email/mật khẩu).
- **Authorization (AuthZ)**: "bạn **được phép** làm gì?" — kiểm tra quyền (vd chỉ admin xoá bài).
- Luôn AuthN trước, AuthZ sau.

</details>

<details>
<summary><b>7.2 Vì sao hash mật khẩu? bcrypt khác MD5/SHA thế nào?</b></summary>

- Hash để DB lộ thì mật khẩu không "trần trụi"; hash là **một chiều**, không đảo ngược.
- bcrypt **có salt** (chống rainbow table) và **chậm có chủ đích** (cost factor → chống brute-force).
- MD5/SHA quá nhanh → không hợp cho mật khẩu.
- Lựa chọn khác: `argon2`.

</details>

<details>
<summary><b>7.5 JWT gồm mấy phần? Vì sao gọi là stateless?</b></summary>

- 3 phần ngăn bởi `.`: **Header** (thuật toán) . **Payload** (`sub`, `role`, `exp`...) . **Signature** (chữ ký bằng secret).
- Server chỉ cần **verify chữ ký** bằng secret là biết user là ai.
- Không cần lưu session ⇒ stateless.

</details>

<details>
<summary><b>7.6 Nên để gì trong JWT payload?</b></summary>

- Chỉ dữ liệu định danh tối thiểu: `sub` (id user — chuẩn JWT), `role`, có thể `email`.
- **Không** để dữ liệu nhạy cảm (mật khẩu, token khác).
- Payload chỉ **base64** → ai cũng giải mã đọc được; chỉ chữ ký mới được bảo vệ.

</details>

<details>
<summary><b>7.7 Vì sao message lỗi đăng nhập nên mập mờ?</b></summary>

- Trả "Email hoặc mật khẩu không đúng" — không nói rõ cái nào sai.
- Tránh **lộ email nào tồn tại** trong hệ thống (chống dò tài khoản / user enumeration).

</details>

<details>
<summary><b>7.8 Guard là gì? Chạy lúc nào trong request lifecycle?</b></summary>

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
<summary><b>7.9 <code>AuthGuard('jwt')</code> kết nối với <code>JwtStrategy</code> thế nào?</b></summary>

- Kết nối qua **tên strategy** `'jwt'` (mặc định của `passport-jwt`).
- Guard kích hoạt → Passport chạy strategy: lấy token từ header `Authorization: Bearer`, verify chữ ký + hạn, gọi `validate(payload)`.
- Giá trị `validate` trả về được gắn vào **`request.user`**.

</details>

<details>
<summary><b>7.11 Guard khác Middleware thế nào?</b></summary>

- Guard biết **ngữ cảnh Nest** (`ExecutionContext` — handler/class, metadata) → hợp phân quyền theo route/role.
- Middleware là hàm Express thuần, chạy sớm hơn, không biết handler nào sẽ xử lý.

</details>

<details>
<summary><b>7.12 JWT hết hạn thì sao?</b></summary>

- Với `ignoreExpiration: false`, Passport tự trả **401** khi token quá hạn (`exp`).
- Client phải đăng nhập lại.
- Hoặc dùng **refresh token** (access token ngắn hạn + refresh token dài hạn) để cấp lại mà không bắt login.

</details>

<details>
<summary><b>7.14 RBAC là gì?</b></summary>

- **Role-Based Access Control** — kiểm soát truy cập dựa trên **vai trò** (admin/author/reader...).
- Mỗi route khai báo role được phép; user có role phù hợp mới qua.
- Là hình thức **Authorization** phổ biến nhất.

</details>

<details>
<summary><b>7.15 <code>@Roles</code> + <code>RolesGuard</code> phối hợp thế nào?</b></summary>

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
<summary><b>7.17 Thứ tự nhiều Guard trong <code>@UseGuards(A, B)</code>?</b></summary>

- Chạy **lần lượt** trái sang phải: A trước, B sau.
- Với auth phải để `JwtAuthGuard` **trước** `RolesGuard`.
- Vì `RolesGuard` cần `request.user` (do JwtAuthGuard/strategy gắn) mới biết role để kiểm tra.

</details>

<details>
<summary><b>7.18 Vì sao không nên cho user tự đăng ký role <code>admin</code>?</b></summary>

- Đó là lỗ hổng **privilege escalation** — ai cũng tự thành admin.
- Role nên do **hệ thống** quyết định (mặc định `reader`); admin tạo qua **seed/migration** hoặc do admin khác cấp.
- Không để `role` trong DTO đăng ký công khai.

</details>

<details>
<summary><b>7.19 RBAC vs ABAC (ownership-based)?</b></summary>

- **RBAC** xét **vai trò** (admin/author/reader).
- **ABAC / ownership-based** xét **thuộc tính của tài nguyên** — vd "chỉ tác giả mới sửa bài đó" (`post.author.id === user.id`).
- ABAC linh hoạt hơn cho luật phụ thuộc ngữ cảnh; thường kết hợp cả hai (admin vượt quyền + chủ sở hữu).

</details>

<details>
<summary><b>7.20 401 Unauthorized vs 403 Forbidden?</b></summary>

- **401** = **chưa xác thực** (chưa đăng nhập / token sai/thiếu/hết hạn).
- **403** = **đã xác thực nhưng không đủ quyền** (sai role, không phải chủ tài nguyên).
- Nhớ: 401 về *danh tính*, 403 về *quyền hạn*.

</details>

<details>
<summary><b>7.21 Nên đặt kiểm tra ownership ở Guard hay Service?</b></summary>

- **Guard**: tách bạch & tái sử dụng tốt, nhưng phải **tự load tài nguyên** (query DB) trong guard để biết chủ sở hữu.
- **Service**: tiện khi handler đã load sẵn tài nguyên (vd `findOne` rồi so `author.id`); ít lặp query.
- Không có đáp án tuyệt đối — nêu được **trade-off** (tái sử dụng vs đơn giản, số lần query) là điểm cộng vòng Senior.

</details>

---

## 8. Docker & Debugging thực chiến

<details>
<summary><b>8.1 Ý nghĩa <code>5433:5432</code> trong Docker? Hai service cùng đòi 1 cổng thì sao?</b></summary>

- Định dạng **`HOST:CONTAINER`**: trái là cổng trên máy host, phải là cổng trong container.
- Hai service cùng giành 1 cổng host → **xung đột**, một bên không bind được.
- Cách xử lý: đổi cổng host (vd `5433:5432`) hoặc tắt service kia.

> Ví dụ: máy có PostgreSQL native chiếm 5432 → container map ra `5433` để khỏi đụng.

</details>

<details>
<summary><b>8.2 Làm sao biết tiến trình nào đang chiếm một cổng?</b></summary>

- Windows: `Get-NetTCPConnection -LocalPort 5432` hoặc `netstat -ano | findstr :5432` → lấy PID → tra process.
- Linux/Mac: `lsof -i :5432`.

</details>

<details>
<summary><b>8.3 Dữ liệu trong Docker nằm ở đâu? <code>down</code> vs <code>down -v</code>?</b></summary>

- Dữ liệu nằm trong **named volume** (xoá/tạo lại container vẫn giữ state).
- `docker compose down`: xoá container + network, **giữ volume** (dữ liệu còn).
- `docker compose down -v`: xoá **cả volume** → **mất sạch dữ liệu**.
- Cẩn thận `-v` và tránh xoá nhầm volume của project khác.

</details>

<details>
<summary><b>8.4 Cách đọc lỗi để khoanh vùng (debug mindset)?</b></summary>

- `ECONNREFUSED` = không tới được server (mạng/cổng).
- `password authentication failed (28P01)` = tới được nhưng sai thông tin đăng nhập.
- **Cô lập biến số** (test trực tiếp DB, parse config, check env) rồi suy ra phần còn lại.
- Đây là tư duy debug mà nhà tuyển dụng Mid/Senior muốn thấy.

</details>

---

## 9. TypeORM Relations

<details>
<summary><b>9.1 One-to-Many / Many-to-One: khóa ngoại (FK) nằm ở bảng nào?</b></summary>

- FK luôn ở phía **"nhiều" (Many)**.
- Lý do: một cha có nhiều con → không thể nhét nhiều id vào 1 dòng cha; mỗi con chỉ 1 cha → để FK ở con là hợp lý.

Ví dụ: User (1) ─< Post (N) → bảng `posts` giữ cột `authorId` trỏ về `users.id`.

</details>

<details>
<summary><b>9.2 <code>@ManyToOne</code> vs <code>@OneToMany</code> — cái nào tạo cột trong bảng?</b></summary>

- Chỉ **`@ManyToOne`** (phía Many) sinh cột FK thật.
- **`@OneToMany`** không tạo cột nào — chỉ là property "ảo" để truy cập ngược (vd `user.posts`).

```ts
@ManyToOne(() => User, (u) => u.posts, { onDelete: 'CASCADE' })
author: User; // posts có cột authorId

@OneToMany(() => Post, (p) => p.author)
posts: Post[]; // users KHÔNG có cột mới
```

</details>

<details>
<summary><b>9.3 Vì sao quan hệ dùng arrow function <code>() => User</code>?</b></summary>

- Tham chiếu kiểu kiểu **"lười" (lazy)**, trì hoãn việc đọc kiểu tới runtime.
- Tránh lỗi **circular dependency** khi hai entity import lẫn nhau (User cần Post, Post cần User), nên thứ tự load không gây `undefined`.

</details>

<details>
<summary><b>9.4 <code>onDelete</code> có những lựa chọn nào?</b></summary>

- `CASCADE`: xoá cha → xoá kèm con.
- `SET NULL`: set FK về null (con "mồ côi").
- `RESTRICT` / `NO ACTION`: cấm xoá cha nếu còn con tham chiếu.

</details>

<details>
<summary><b>9.5 Eager vs Lazy loading? Vấn đề N+1 là gì? <i>(sẽ học kỹ ở bài truy vấn quan hệ)</i></b></summary>

- **Eager**: tự động load quan hệ kèm mỗi query. **Lazy**: chỉ load khi truy cập (trả Promise).
- Mặc định TypeORM không load quan hệ trừ khi `eager: true`, `relations: ['author']` hoặc `leftJoinAndSelect`.
- **N+1**: lấy N bài rồi lặp query tác giả từng bài → 1 + N query → chậm.
- Khắc phục: JOIN (`leftJoinAndSelect`) hoặc `relations` để lấy 1 lần.

</details>

<details>
<summary><b>9.6 Làm sao load quan hệ khi truy vấn?</b></summary>

- (1) option `relations: ['author']` trong `find`/`findOne`.
- (2) QueryBuilder `.leftJoinAndSelect('post.author', 'author')`.
- (3) `eager: true` trên decorator (luôn tự load — dùng thận trọng).
- (1) và (2) đều dịch ra JOIN → 1 query, tránh N+1.

```ts
this.postRepository.find({ relations: ['author'] });
```

</details>

<details>
<summary><b>9.8 (Security) Vì sao <code>authorId</code> phải lấy từ JWT, không từ body client gửi?</b></summary>

- Chống **mạo danh**: nếu client tự gửi `authorId`, họ có thể tạo bài "nhân danh" người khác.
- Mọi dữ liệu **định danh chủ sở hữu** phải lấy từ **token đã xác thực** (`@CurrentUser()`), không tin body.
- Vì vậy `authorId` **không** nằm trong DTO.

</details>

<details>
<summary><b>9.10 Lọc & sắp xếp theo quan hệ trong TypeORM?</b></summary>

- Lọc theo quan hệ: `where: { post: { id: postId } }` (dịch ra `WHERE postId = ...`).
- Sắp xếp: `order: { createdAt: 'DESC' }`.
- Kết hợp `relations` để kèm dữ liệu liên quan.

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

- Ở một **bảng nối (join table)** chứa 2 khóa ngoại, mỗi dòng là một cặp liên kết.
- Không thể nhét FK vào 2 bảng gốc vì cả hai phía đều "nhiều". TypeORM tự tạo bảng nối.

Ví dụ: `posts_tags(postId, tagId)`.

</details>

<details>
<summary><b>9.12 <code>@ManyToMany</code> + <code>@JoinTable</code> — đặt ở đâu?</b></summary>

- `@ManyToMany` đặt ở **cả hai** entity.
- `@JoinTable` chỉ ở **MỘT phía** — **owning side** (bên sở hữu bảng nối); phía kia là inverse side.

```ts
@ManyToMany(() => Tag, (t) => t.posts, { cascade: true }) // owning side
@JoinTable({ name: 'posts_tags' })
tags: Tag[];

@ManyToMany(() => Post, (p) => p.tags) // inverse side
posts: Post[];
```

</details>

<details>
<summary><b>9.13 <code>cascade: true</code> để làm gì? Tránh tạo bản ghi trùng (find-or-create)?</b></summary>

- `cascade: true`: lưu cha kèm entity liên quan **mới** → TypeORM tự lưu chúng + tạo liên kết (đỡ save thủ công).
- Tránh trùng: đặt cột định danh (`name`) là `unique` + pattern **find-or-create** — tìm theo name, có thì tái dùng, chưa thì tạo mới.

</details>

---

## 10. Interceptors & Serialization

<details>
<summary><b>10.1 Interceptor là gì? Chạy lúc nào trong request lifecycle?</b></summary>

- Thành phần **bọc quanh handler**, chạy **cả trước và sau** handler.
- Dùng **RxJS** biến đổi luồng response: loại field, bọc dữ liệu, logging, đo thời gian, cache.
- Vị trí: `Guard → Interceptor(trước) → Pipe → Handler → Interceptor(sau) → Filter`.

</details>

<details>
<summary><b>10.2 <code>ClassSerializerInterceptor</code> + <code>@Exclude()</code> làm gì?</b></summary>

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

- `select: false` (TypeORM): chặn ở tầng **truy vấn DB**, cột không load trừ khi `addSelect`.
- `@Exclude()` (class-transformer): chặn ở tầng **serialize response**, không lộ ra ngoài dù field đang trong bộ nhớ.
- Dùng cả hai = bảo vệ **2 lớp**.

</details>

<details>
<summary><b>10.4 Khi nào <code>@Exclude()</code> KHÔNG có tác dụng?</b></summary>

- Khi handler trả về **plain object** (vd `return { ...user }`) thay vì **instance class entity**.
- `ClassSerializerInterceptor` cần instance để transform; plain object không mang metadata `@Exclude`.

</details>

---

## 11. Pagination, Filtering & QueryBuilder

<details>
<summary><b>11.1 Phân trang trong TypeORM thế nào?</b></summary>

- Dùng `skip((page-1)*limit).take(limit)` để phân trang.
- `getManyAndCount()` trả `[data, total]`.
- Tính `totalPages = Math.ceil(total/limit)`.
- Response chuẩn: `{ data, total, page, limit, totalPages }`.

```ts
const [data, total] = await qb.skip((page-1)*limit).take(limit).getManyAndCount();
```

</details>

<details>
<summary><b>11.2 <code>skip</code>/<code>take</code> khác <code>offset</code>/<code>limit</code>?</b></summary>

- `skip`/`take`: phân trang ở **cấp entity**.
- Khi join quan hệ to-many, TypeORM tự dùng subquery để lấy đúng số entity, tránh nhân dòng do JOIN.
- `offset`/`limit`: SQL thô, áp lên kết quả join → dễ sai số khi join to-many.

</details>

<details>
<summary><b>11.3 Offset pagination vs Cursor pagination?</b></summary>

- **Offset** (`skip/take` theo page): đơn giản, nhảy trang tuỳ ý.
- Nhược offset: chậm khi offset lớn (DB vẫn quét qua), dễ lệch/trùng khi dữ liệu thêm/xoá giữa chừng.
- **Cursor** (dựa `id`/`createdAt` của bản ghi cuối): ổn định, nhanh với dữ liệu lớn/real-time; nhược: không nhảy trang bất kỳ.
- Feed/mạng xã hội thường dùng cursor.

</details>

<details>
<summary><b>11.5 Query param luôn là string — validate kiểu number/boolean thế nào?</b></summary>

- Dùng `@Type(() => Number)` (class-transformer) + `transform: true` trong ValidationPipe để **ép kiểu** trước khi `@IsInt`/`@Min` kiểm tra.
- Boolean dạng query: `@IsBooleanString()` (nhận `'true'`/`'false'`) rồi tự so `=== 'true'`.

</details>

---

## 12. Exception Filters & Error Handling

<details>
<summary><b>12.1 Exception Filter là gì? Chạy lúc nào?</b></summary>

- Thành phần **bắt exception** ném ra trong app và định hình response lỗi.
- Nằm ở **cuối** request lifecycle, kích hoạt khi có lỗi.
- Dùng để **chuẩn hoá format lỗi** + **log tập trung**.

</details>

<details>
<summary><b>12.3 <code>ArgumentsHost</code> là gì?</b></summary>

- Đối tượng truy cập **ngữ cảnh thực thi trung lập** (HTTP / WebSocket / RPC).
- Nhờ vậy filter/guard/interceptor dùng được trên nhiều loại transport.
- Ví dụ HTTP: `host.switchToHttp().getRequest()/getResponse()`.

</details>

<details>
<summary><b>12.4 Cấu trúc <code>HttpException</code>?</b></summary>

- `getStatus()`: mã HTTP; `getResponse()`: string hoặc object `{ statusCode, message, error }`.
- Lỗi validation: `message` là **mảng**.
- Các exception dựng sẵn (`NotFoundException`...) đều kế thừa `HttpException`.
- **Lưu ý:** lỗi không phải HttpException → nên coi là **500**.

</details>

<details>
<summary><b>12.5 Filter toàn cục: <code>useGlobalFilters</code> vs <code>APP_FILTER</code>?</b></summary>

- `app.useGlobalFilters(new MyFilter())`: đơn giản nhưng **không inject** được dependency.
- Provider **`APP_FILTER`** (trong module): Nest quản lý qua DI → **inject được service** (vd gửi lỗi lên Sentry/logger ngoài).
- Tương tự có `APP_GUARD`, `APP_INTERCEPTOR`, `APP_PIPE`.

</details>

---

## 13. Testing (Jest)

<details>
<summary><b>13.1 Unit vs Integration vs E2E test?</b></summary>

- **Unit**: test 1 đơn vị (vd service) **cô lập**, mock hết dependency → nhanh, ổn định.
- **Integration**: nhiều thành phần ghép, một phần thật (vd service + DB test).
- **E2E**: test cả luồng qua HTTP như user thật (supertest + app + DB test).
- Tháp test: nhiều unit, ít e2e.

</details>

<details>
<summary><b>13.3 <code>Test.createTestingModule</code> + <code>getRepositoryToken</code>?</b></summary>

- `Test.createTestingModule({ providers: [...] }).compile()`: dựng DI container riêng cho test.
- Cung cấp repository giả qua `{ provide: getRepositoryToken(Entity), useValue: mockRepo }`.
- Đúng token mà `@InjectRepository(Entity)` dùng → service nhận bản giả.

```ts
const module = await Test.createTestingModule({
  providers: [UsersService, { provide: getRepositoryToken(User), useValue: mockRepo }],
}).compile();
```

</details>

<details>
<summary><b>13.6 E2E test trong NestJS setup thế nào?</b></summary>

- `Test.createTestingModule({ imports: [AppModule] }).compile()` → `createNestApplication()` → `app.init()`.
- Dùng **supertest**: `request(app.getHttpServer()).post(...).send(...).expect(201)`.
- `app.init()` (không `listen()`) dựng app trong bộ nhớ, không chiếm cổng.
- Nhớ `afterAll(() => app.close())`.

</details>

<details>
<summary><b>13.7 Vì sao global pipe/filter không tự hoạt động trong E2E test?</b></summary>

- **`main.ts` không chạy** trong môi trường test — `createNestApplication()` chỉ dựng app từ AppModule.
- Mọi thứ đăng ký global ở `main.ts` (`useGlobalPipes`/`useGlobalFilters`/`useGlobalInterceptors`) phải **tự áp lại** trong setup test.
- Đây là bẫy hay gặp.

</details>

<details>
<summary><b>13.8 Chiến lược database cho E2E test?</b></summary>

- Dùng **DB test riêng** (vd `blog_test`, hoặc SQLite in-memory) tách khỏi dev/prod.
- **Dọn dữ liệu** giữa các lần chạy: truncate bảng, hoặc **transaction rollback** mỗi test, hoặc dùng khoá duy nhất (vd email theo timestamp).
- Mục tiêu: test **ổn định**, không phụ thuộc dữ liệu cũ.

</details>

---

## 14. Prisma (ORM)

<details>
<summary><b>14.1 So sánh Prisma vs TypeORM: ưu/nhược điểm, khi nào chọn Prisma?</b></summary>

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
<summary><b>14.4 `$transaction` dạng interactive và dạng array khác nhau thế nào, khi nào dùng cái nào?</b></summary>

- Array (`$transaction([...])`): nhận mảng Prisma promise, chạy trong cùng transaction (BEGIN/COMMIT, rollback nếu lỗi), theo thứ tự — nhưng KHÔNG dùng được kết quả bước trước cho bước sau. Hợp cho thao tác độc lập.
- Interactive (`$transaction(async (tx) => {...})`): callback với client `tx`, cho phép logic điều kiện, đọc rồi ghi dựa trên kết quả trung gian.
- Lưu ý interactive: connection bị giữ suốt callback → giữ transaction NGẮN, tránh I/O ngoài (HTTP, queue) bên trong. Cấu hình được `timeout`, `maxWait`, `isolationLevel`.

```ts
await prisma.$transaction(async (tx) => {
  const p = await tx.project.update({
    where: { id }, data: { issueCounter: { increment: 1 } },
    select: { key: true, issueCounter: true },
  });
  return tx.issue.create({ data: { key: `${p.key}-${p.issueCounter}`, title } });
});
```

</details>

<details>
<summary><b>14.6 Connection pool trong Prisma hoạt động sao và PG driver adapter để làm gì?</b></summary>

- Prisma quản lý pool nội bộ. Query engine (v6 trở về trước): default `num_physical_cpus * 2 + 1`, chỉnh qua `connection_limit` trên `DATABASE_URL`. (v7 + driver adapter: theo default driver, ví dụ `pg` `max = 10`.)
- Serverless/nhiều instance: mỗi instance mở pool riêng → dễ cạn `max_connections` của PostgreSQL.
- Khắc phục: dùng pooler ngoài (PgBouncer, Prisma Accelerate) và hạ `connection_limit` (thường `=1` khi qua pooler chế độ transaction).
- Driver adapter (`@prisma/adapter-pg`): dùng driver JS native thay query engine binary → hợp edge runtime, dễ tái dùng pool/pooler có sẵn.

```ts
import { PrismaPg } from '@prisma/adapter-pg';
const adapter = new PrismaPg({ connectionString: process.env.DATABASE_URL });
const prisma = new PrismaClient({ adapter });
```

</details>

<details>
<summary><b>14.7 Khi nào dùng `$queryRaw` và làm sao tránh SQL injection?</b></summary>

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

<details>
<summary><b>14.8 Enum trong Prisma được map xuống PostgreSQL và sinh type ra sao?</b></summary>

- `enum` trong `schema.prisma` → tạo native enum type trong PostgreSQL (`CREATE TYPE`) + sinh TypeScript enum trong Client → kiểm tra ở cả DB lẫn compile time.
- Migration: thêm giá trị enum là an toàn, NHƯNG `ALTER TYPE ... ADD VALUE` không chạy trong transaction block → Prisma tách ra migration riêng.
- Đổi/xóa giá trị enum thường cần migration thủ công vì ảnh hưởng data hiện có (backfill/cast).
- Enum ràng buộc trường trạng thái/role gọn và type-safe hơn string tùy ý.

```prisma
enum IssueStatus { TODO IN_PROGRESS DONE }

model Issue {
  id     String      @id @default(cuid())
  status IssueStatus @default(TODO)
}
```

</details>

---

## 15. Advanced Authentication

<details>
<summary><b>15.1 Phân biệt access token và refresh token? Tại sao cần cả hai thay vì một token sống lâu?</b></summary>

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
<summary><b>15.2 Refresh token ROTATION là gì? Nó chống replay attack như thế nào?</b></summary>

- **Rotation**: mỗi lần refresh, server cấp refresh token MỚI và invalidate token cũ -> mỗi token chỉ dùng một lần.
- Chống **replay**: attacker dùng lại token đã bị xoay sẽ fail vì token cũ đã invalid.
- Nhận một refresh token **đã used** = dấu hiệu **token reuse** -> thu hồi toàn bộ **token family** của user, buộc đăng nhập lại.
- Nguyên tắc: lưu token theo **family/chain** để phát hiện reuse.
- **Bẫy**: đánh dấu `usedAt` cũ + cấp token mới phải nằm trong **một transaction** để tránh race khi refresh song song.

```ts
async refresh(oldToken: string, res: Response) {
  return this.prisma.$transaction(async (tx) => {
    const record = await this.findValidToken(tx, oldToken);
    if (!record) throw new UnauthorizedException();
    if (record.usedAt || record.revokedAt) { // reuse -> thu hoi ca family
      await tx.refreshToken.updateMany({ where: { familyId: record.familyId, revokedAt: null }, data: { revokedAt: new Date() } });
      throw new UnauthorizedException('Token reuse detected');
    }
    await tx.refreshToken.update({ where: { id: record.id }, data: { usedAt: new Date() } });
    return this.issueTokens(tx, record.userId, record.familyId, res);
  });
}
```

</details>

<details>
<summary><b>15.3 Lưu token ở httpOnly cookie vs localStorage khác nhau thế nào? Trade-off XSS và CSRF?</b></summary>

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
<summary><b>15.4 JWT (stateless) vs session (stateful) khác nhau ra sao? Khi nào chọn cái nào?</b></summary>

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
<summary><b>15.5 Mô tả flow OTP / email verification an toàn? Cần lưu OTP thế nào?</b></summary>

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
<summary><b>15.6 Với JWT stateless, làm sao implement logout và thu hồi token thực sự?</b></summary>

- JWT thuần: 'logout' ở client chỉ xóa cookie, token vẫn hợp lệ đến khi hết hạn -> chưa thu hồi thật.
- **Hướng 1**: thu hồi **refresh token** trong DB (revoke) -> lần refresh sau fail; access token đoản hạn nên cửa sổ rủi ro nhỏ.
- **Hướng 2**: **blocklist** `jti` trong Redis (TTL = thời gian còn lại) để chặn cả access token còn hạn.
- **Bẫy**: hướng 2 khiến mỗi request lookup Redis -> mất tính stateless; chỉ dùng khi cần revoke tức thì. Logout-all-devices = revoke toàn bộ family.

```ts
async logout(userId: string, jti: string, exp: number, res: Response) {
  await this.prisma.refreshToken.updateMany({ where: { userId, revokedAt: null }, data: { revokedAt: new Date() } });
  const ttl = exp - Math.floor(Date.now() / 1000);
  if (ttl > 0) await this.redis.set(`bl:${jti}`, '1', 'EX', ttl); // tuy chon
  res.clearCookie('access_token'); res.clearCookie('refresh_token');
}
```

</details>

---

## 16. Web Security

<details>
<summary><b>16.1 Helmet làm gì và những security header nào quan trọng nhất?</b></summary>

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
<summary><b>16.7 Quản lý secret/env an toàn và masking PII trong log như thế nào?</b></summary>

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
<summary><b>16.8 Kể vài lỗ hổng OWASP Top 10 phổ biến và cách bạn phòng chống trong NestJS?</b></summary>

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

<details>
<summary><b>16.10 [BỔ SUNG] Bảo mật dependency/supply-chain trong dự án Node/NestJS như thế nào?</b></summary>

- Phần lớn code chạy đến từ `node_modules` → dependency là bề mặt tấn công lớn (OWASP A06, A08).
- **Quét định kỳ**: `pnpm audit`/`npm audit`, tích hợp Dependabot/Renovate/Snyk vào CI để cảnh báo CVE và tự mở PR.
- **Khóa phiên bản**: commit lockfile, cài bằng `--frozen-lockfile` trong CI → build tái lập.
- **Giảm bề mặt**: hạn chế dependency thừa, cảnh giác typosquatting, xem postinstall script lạ; tách secret CI/registry khỏi log, dùng `provenance` khi publish.

```bash
pnpm audit --audit-level=high   # fail CI nếu có CVE high+
pnpm install --frozen-lockfile  # build tái lập
```

</details>

---

## 17. Observability and Logging

<details>
<summary><b>17.1 Tại sao nên dùng structured logging (JSON) thay vì log dạng text thuần? Lợi ích là gì?</b></summary>

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

- **INFO**: sự kiện bình thường, đáng quan tâm nghiệp vụ (user login, tạo issue, cron xong).
- **WARN**: bất thường nhưng vẫn xử lý được (gần chạm rate limit, retry thành công, gọi deprecated API).
- **ERROR**: thất bại thật sự cần hành động (exception bất ngờ, 5xx, rớt kết nối DB).
- Quy tắc quan trọng: **400/401/403/404 KHÔNG phải ERROR** — lỗi client, kỳ vọng xảy ra; chỉ log WARN/INFO để tránh nhiễu và alert fatigue.
- Còn có **DEBUG/TRACE** (chi tiết kỹ thuật, tắt ở prod) và **FATAL** (process không thể tiếp tục).

Ví dụ: Exception Filter map `statusCode >= 500` -> ERROR, 4xx -> WARN, nên dashboard ERROR chỉ chứa lỗi server thật.

</details>

<details>
<summary><b>17.3 Kiến trúc log request: tại sao dùng Interceptor cho success và Exception Filter cho lỗi?</b></summary>

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
<summary><b>17.4 Tại sao logging không nên block request? Buffered/batched write giải quyết điều này như thế nào?</b></summary>

- Ghi I/O đồng bộ (`fs.writeFileSync`) hoặc `await` HTTP call tới log service ngay trong request -> thêm latency mỗi response.
- **Sync I/O** (`*Sync`) còn chặn event loop của Node, giảm throughput toàn service. Async I/O không chặn loop nhưng `await` vẫn kéo dài phản hồi.
- Giải pháp: **buffer trong memory** rồi **flush theo batch** (theo số lượng hoặc thời gian) bằng ghi async ở background. Request chỉ push log vào buffer rồi trả về ngay.
- Đánh đổi: có thể mất vài log cuối nếu crash trước flush -> nên flush khi shutdown (`onApplicationShutdown`/`beforeExit`).
- Thực tế: **Pino** đã xử lý sẵn (async, worker thread qua `pino.transport`); tự viết buffer phải cảnh giác mất log khi crash và race condition.

```ts
write(entry) {
  this.buffer.push(entry);            // non-blocking
  if (this.buffer.length >= 100) void this.flush();
}
@Cron('*/5 * * * * *')                // flush moi 5s
async flush() {
  if (this.flushing || !this.buffer.length) return;
  this.flushing = true;
  const batch = this.buffer.splice(0);
  try { await this.sink.writeBatch(batch); }
  catch (e) { this.buffer.unshift(...batch); }  // tra lai neu loi
  finally { this.flushing = false; }
}
```

</details>

<details>
<summary><b>17.5 Correlation ID / Request ID là gì và tại sao nó quan trọng trong hệ thống?</b></summary>

- Một định danh duy nhất (vd UUID) gắn cho mỗi request khi vào hệ thống, rồi đính vào **tất cả** log sinh ra trong request đó.
- Cho phép **truy vết xuyên suốt** một request (controller -> service -> DB) chỉ bằng lọc theo ID — quan trọng khi nhiều request đồng thời trộn lẫn trong log.
- Microservice: truyền qua header (`X-Request-Id`) giữa các service để trace toàn hệ thống.
- Thường dùng middleware + `AsyncLocalStorage` để lưu ID theo context, không phải truyền tay.
- Bẫy: phải gọi `als.run(...)` sao cho **toàn bộ phần còn lại của request** chạy trong callback đó (gọi `next()` bên trong `als.run()`); đọc lại qua `als.getStore()`.

```ts
export const als = new AsyncLocalStorage<{ requestId: string }>();
export function requestIdMiddleware(req, res, next) {
  const id = req.headers['x-request-id'] ?? randomUUID();
  req.requestId = id;
  res.setHeader('x-request-id', id);
  als.run({ requestId: id }, () => next());
}
```

</details>

<details>
<summary><b>17.6 Tại sao chỉ gửi Sentry cho lỗi 5xx và chỉ ở môi trường production?</b></summary>

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
<summary><b>17.7 Cron retention xóa log cũ hoạt động thế nào và tại sao cần nó?</b></summary>

- Log tăng liên tục -> đầy disk/DB, chậm query. Retention là job định kỳ (`@Cron`) **xóa log cũ hơn ngưỡng giữ lại** (vd 30 ngày).
- Xóa khối lớn nên làm **theo batch** (vd 10k dòng/vòng, lặp đến hết) để tránh khóa bảng lâu, transaction quá to, phình WAL.
- Cần index trên `createdAt`/`timestamp` để query xóa nhanh.
- Quy mô lớn: **table partitioning** + `DROP PARTITION` tối ưu nhất — gần như tức thời, không sinh bloat/vacuum nặng như `DELETE`.
- Lưu ý: `prisma.deleteMany` xóa một lần tất cả bản ghi thỏa điều kiện -> bảng rất lớn nên tự chia batch.

```ts
@Cron(CronExpression.EVERY_DAY_AT_MIDNIGHT)
async purgeOldLogs() {
  const cutoff = new Date(Date.now() - 30 * 24 * 60 * 60 * 1000);
  for (;;) {
    const old = await this.prisma.log.findMany({
      where: { createdAt: { lt: cutoff } },
      select: { id: true }, take: 10_000,
    });
    if (old.length === 0) break;
    await this.prisma.log.deleteMany({
      where: { id: { in: old.map((l) => l.id) } },
    });
  }
}
```

</details>

<details>
<summary><b>17.8 Health check endpoint dùng để làm gì và nên kiểm tra những gì?</b></summary>

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
<summary><b>18.1 Global providers (APP_GUARD/APP_FILTER/APP_INTERCEPTOR) khác gì so với useGlobalGuards()? Tại sao nên dùng token này? Nhiều APP_GUARD có chạy đúng thứ tự không?</b></summary>

- `useGlobalGuards()`: instance tạo thủ công **ngoài DI container** nên không inject được dependency (`Reflector`, service...).
- Token `APP_GUARD`/`APP_FILTER`/`APP_INTERCEPTOR`: Nest tạo qua DI nên **inject được** Reflector/service/config; một module có thể cung cấp nhiều provider cho cùng token.
- Bẫy: nhiều `APP_GUARD` **không đảm bảo thứ tự** (resolve từ DI). Chỉ `@UseGuards(A, B)` và `useGlobalGuards(a, b)` mới chạy đúng thứ tự.
- Cần thứ tự (auth trước authz): gộp logic vào một guard, hoặc đặt auth ở `APP_GUARD` còn `RolesGuard` bind ở controller/handler.

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

</details>

<details>
<summary><b>18.2 Mô tả pattern global JwtAuthGuard + @Public(). Làm sao một route opt-out khỏi xác thực?</b></summary>

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
<summary><b>18.3 Custom param decorator (@CurrentUser) hoạt động thế nào? Khác với class/method decorator ở chỗ nào?</b></summary>

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
<summary><b>18.4 Phân biệt useClass, useValue, useFactory, useExisting. Khi nào dùng useFactory với inject?</b></summary>

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
<summary><b>18.5 Injection scope DEFAULT/REQUEST/TRANSIENT khác nhau ra sao? Cái giá của REQUEST scope?</b></summary>

- `DEFAULT` (singleton): một instance dùng chung, khởi tạo một lần, hiệu năng tốt nhất.
- `REQUEST`: instance mới mỗi request, inject được `REQUEST`; nhưng **bubble up** — provider phụ thuộc nó cũng thành request-scoped, gây overhead, giảm throughput.
- `TRANSIENT`: mỗi consumer inject nhận instance riêng.
- Tránh REQUEST scope ở provider dùng rộng; thay bằng `AsyncLocalStorage` để lưu context theo request mà vẫn giữ singleton.

```ts
@Injectable({ scope: Scope.REQUEST })
export class RequestContextService {
  constructor(@Inject(REQUEST) private req: Request) {}
}
```

</details>

<details>
<summary><b>18.6 Các lifecycle hooks onModuleInit và onApplicationShutdown dùng để làm gì? Làm sao bật shutdown hooks?</b></summary>

- `onModuleInit`: chạy sau khi dependency của module khởi tạo xong — khởi tạo kết nối, seed data, warm cache (luôn chạy).
- `onApplicationShutdown(signal)`: chạy khi app nhận tín hiệu dừng (SIGTERM/SIGINT) — đóng DB, flush buffer, graceful shutdown.
- Shutdown hook chỉ kích hoạt khi gọi `app.enableShutdownHooks()`.
- Thứ tự: init theo dependency trước; shutdown chạy ngược lại với init.

```ts
@Injectable()
export class LogService implements OnApplicationShutdown {
  async onApplicationShutdown(signal?: string) {
    await this.flushBuffer(); // ghi not log con trong buffer
  }
}
// main.ts
app.enableShutdownHooks();
```

</details>

<details>
<summary><b>18.8 Dynamic module là gì? Phân biệt forRoot và forRootAsync.</b></summary>

- Dynamic module: trả cấu hình động qua static method, truyền options và đăng ký provider/exports lúc runtime (nền tảng của `ConfigModule`, `JwtModule`).
- `forRoot(options)`: cấu hình **đồng bộ**, giá trị có sẵn ngay.
- `forRootAsync(options)`: cấu hình **bất đồng bộ**, dùng `useFactory` + `inject` lấy options từ provider khác (vd `ConfigService`) — hữu ích khi config phụ thuộc env/provider load trước.

```ts
@Module({})
export class MailModule {
  static forRootAsync(opts: MailAsyncOptions): DynamicModule {
    return {
      module: MailModule,
      imports: opts.imports ?? [], // vi du ConfigModule
      providers: [
        { provide: MAIL_OPTIONS, useFactory: opts.useFactory, inject: opts.inject ?? [] },
      ],
      exports: [MAIL_OPTIONS],
    };
  }
}
```

</details>

<details>
<summary><b>18.9 Tại sao nên validate biến môi trường (.env) và làm sao làm trong NestJS?</b></summary>

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
<summary><b>19.4 Phân biệt optimistic lock (version) và pessimistic lock (SELECT FOR UPDATE)? Khi nào dùng cái nào?</b></summary>

- Optimistic: không khóa trước; thêm cột `version`, update kèm `WHERE version = X` và set `version = version + 1`. rowCount = 0 → conflict → retry/báo lỗi (HTTP 409). Tốt khi tranh chấp THẤP (throughput cao).
- Pessimistic: `SELECT ... FOR UPDATE` khóa row khi đọc, giữ đến hết transaction; request khác phải chờ. Tốt khi tranh chấp CAO, đổi lại giảm concurrency + rủi ro deadlock.
- Tóm: optimistic = "cứ làm, đụng độ thì retry"; pessimistic = "khóa trước cho chắc".

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

</details>

<details>
<summary><b>19.5 Nêu các isolation level của PostgreSQL và các vấn đề chúng phòng tránh.</b></summary>

- Read Committed (mặc định): mỗi statement thấy data đã commit lúc nó bắt đầu; tránh dirty read, vẫn có non-repeatable + phantom read.
- Repeatable Read: snapshot cố định suốt transaction; tránh dirty + non-repeatable + (ở Postgres) phantom read; vẫn còn write-skew, có thể lỗi serialization phải retry.
- Serializable: mạnh nhất (SSI), chống mọi anomaly kể cả write-skew; đắt hiệu năng, dễ lỗi `could not serialize access` phải retry.
- Postgres không có Read Uncommitted thật (hành xử như Read Committed).
- Quy tắc: dùng level thấp nhất mà vẫn đúng; chỉ nâng Serializable khi có anomaly thật + luôn có retry.

```ts
await prisma.$transaction(
  async (tx) => { /* ... */ },
  { isolationLevel: Prisma.TransactionIsolationLevel.Serializable },
);
```

</details>

<details>
<summary><b>19.6 Index là gì? Phân biệt B-tree, composite, unique và partial index?</b></summary>

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
<summary><b>19.7 Vấn đề N+1 query là gì và xử lý trong Prisma như thế nào?</b></summary>

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
<summary><b>19.8 Cascade delete và soft delete khác nhau thế nào? Khi nào dùng soft delete?</b></summary>

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

<details>
<summary><b>19.9 Deadlock là gì, vì sao xảy ra và làm sao tránh trong ứng dụng NestJS/Prisma?</b></summary>

- Deadlock: hai+ transaction giữ lock và chờ lock của nhau theo vòng tròn (T1 khóa A chờ B, T2 khóa B chờ A).
- Postgres tự phát hiện và hủy một transaction với lỗi `deadlock detected` (40P01); transaction còn lại chạy tiếp.
- Nguyên nhân: update nhiều row theo THỨ TỰ KHÁC NHAU, giữ transaction mở lâu (I/O chậm), lock escalation.
- Tránh/giảm: (1) khóa/update row theo MỘT thứ tự nhất quán (sort theo id); (2) giữ transaction NGẮN, không gọi API/mail bên trong; (3) đặt `timeout`/`lock_timeout`; (4) RETRY với backoff khi gặp 40P01/40001 (lỗi tạm thời).

```ts
async function withRetry<T>(fn: () => Promise<T>, tries = 3): Promise<T> {
  for (let i = 0; ; i++) {
    try { return await fn(); }
    catch (e: any) {
      const code = e?.code; // P2034 (Prisma) hoac 40P01/40001
      if (i < tries - 1 && (code === 'P2034' || code === '40P01' || code === '40001')) continue;
      throw e;
    }
  }
}
```

</details>

---

## 20. Advanced Node.js

<details>
<summary><b>20.1 Giải thích các phase của event loop (timers, poll, check) và sự khác biệt giữa microtask và macrotask?</b></summary>

- Mỗi tick chạy tuần tự các phase: **timers** (`setTimeout`/`setInterval` hết hạn) → pending callbacks → idle/prepare → **poll** (I/O) → **check** (`setImmediate`) → close callbacks. Callback trong các phase này là **macrotask**.
- **Microtask** không thuộc phase nào: có 2 hàng đợi ưu tiên cao hơn — `process.nextTick` (cao nhất) và Promise (`.then`, `queueMicrotask`).
- Sau MỖI macrotask, Node drain hết `nextTick` rồi hết microtask, RỒI mới sang phase tiếp theo → microtask luôn chạy trước macrotask kế.
- Bẫy: sinh microtask/nextTick vô hạn sẽ làm đói (starve) event loop.

```js
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
Promise.resolve().then(() => console.log('promise')); // microtask: chay truoc
// Thu tu timeout vs immediate o top-level la KHONG xac dinh
// (chi trong I/O callback thi immediate moi luon truoc timeout)
```

</details>

<details>
<summary><b>20.2 process.nextTick starvation là gì và tại sao nó nguy hiểm hơn setImmediate?</b></summary>

- `process.nextTick` có hàng đợi RIÊNG, drain trước cả microtask Promise và trước khi sang phase kế (ưu tiên còn cao hơn microtask).
- Đệ quy gọi `nextTick` → hàng đợi không bao giờ rỗng → event loop bị "đói": I/O, timer, microtask đều không chạy, server treo.
- `setImmediate` an toàn hơn: chạy ở phase **check** mỗi vòng, nên giữa các vòng phase **poll** vẫn xử lý được I/O.
- Dùng `nextTick` cho hoãn rất ngắn (đảm bảo API luôn async); việc lặp/tính nặng nên dùng `setImmediate` hoặc worker_threads.

```js
function loop() { process.nextTick(loop); }      // treo server
function safeLoop() { setImmediate(safeLoop); }   // KHONG chan I/O
```

</details>

<details>
<summary><b>20.3 Backpressure trong Streams là gì và cách xử lý đúng?</b></summary>

- Backpressure: bên ghi (writable) tiêu thụ chậm hơn bên đọc đẩy vào → buffer dồn lại, tốn RAM, có thể OOM.
- Cơ chế đúng: `writable.write()` trả về `false` nghĩa là buffer vượt `highWaterMark` → ngừng ghi (vd `readable.pause()`), đợi sự kiện `'drain'` rồi tiếp.
- Thực tế nên dùng `pipe()`/`pipeline()` để tự điều phối. Ưu tiên `pipeline()` vì tự destroy + báo lỗi khi có stream lỗi (`pipe()` không làm → dễ rò file descriptor).

```js
const { pipeline } = require('node:stream/promises');
await pipeline(
  fs.createReadStream('big.csv'),
  parseTransform,
  fs.createWriteStream('out.json'),
); // tu dong backpressure + cleanup khi error
```

</details>

<details>
<summary><b>20.4 So sánh worker_threads, cluster và child_process — khi nào dùng cái nào cho tác vụ CPU-bound?</b></summary>

- **cluster**: fork nhiều process Node giống nhau, chia sẻ cổng listen; mỗi process có heap riêng (không share memory) → scale I/O-bound theo số core.
- **child_process** (`spawn`/`exec`/`fork`): tạo tiến trình con, chạy được chương trình khác (vd `ffmpeg`), giao tiếp qua stdio/IPC → chạy tool ngoài hoặc cách ly process.
- **worker_threads**: tạo THREAD trong cùng process, nhẹ hơn fork, share dữ liệu nhị phân qua `SharedArrayBuffer` → tốt nhất cho CPU-bound thuần JS (hash, nén, tính nặng) vì không chặn main loop.

```js
const { Worker } = require('node:worker_threads');
const w = new Worker('./heavy-hash.js', { workerData: payload });
w.once('message', result => res.json(result)); // main loop van phuc vu request khac
```

</details>

<details>
<summary><b>20.5 Các nguyên nhân rò rỉ bộ nhớ thường gặp trong Node (closure, event listener) và cách phát hiện?</b></summary>

- Nguyên nhân phổ biến: **event listener** đăng ký lặp không `off`/`removeListener` (kèm `MaxListenersExceededWarning`), **closure** giữ tham chiếu object lớn, **cache/`Map` toàn cục** chỉ thêm không xóa.
- Dấu hiệu: RSS/heap tăng dần, không giảm cả sau GC (`--expose-gc` để ép `global.gc()` khi test).
- Phát hiện: chụp nhiều heap snapshot (Chrome DevTools `--inspect`) so sánh retained size; log `process.memoryUsage()`; `clinic doctor`/`heapprofiler`.
- Khắc phục: dùng LRU có giới hạn (`lru-cache`) thay cache vô hạn; hủy listener/timer trong `onModuleDestroy` của NestJS.

```js
const v8 = require('node:v8');
v8.writeHeapSnapshot(`heap-${Date.now()}.heapsnapshot`); // so sanh 2 file
```

</details>

<details>
<summary><b>20.6 Làm sao detect event loop bị block trong production?</b></summary>

- Đo tin cậy bằng **event loop delay/lag**: `perf_hooks.monitorEventLoopDelay()` (histogram nanosecond) hoặc đo lệch giữa `setInterval` dự kiến và thực tế.
- Lag cao (vd p99 > 100ms) thường do code đồng bộ nặng: `JSON.parse`/`stringify` payload khổng lồ, vòng lặp lớn, crypto sync.
- Tìm hàm gây nghẽn bằng `--prof` / `clinic flame` (flamegraph).
- Gắn metric vào logging + cảnh báo (Sentry/Prometheus) → triệu chứng sớm trước khi latency tăng và 5xx.

```js
const { monitorEventLoopDelay } = require('node:perf_hooks');
const h = monitorEventLoopDelay({ resolution: 20 }); h.enable();
setInterval(() => { console.log('p99 lag(ms):', (h.percentile(99)/1e6).toFixed(2)); h.reset(); }, 5000).unref();
```

</details>

<details>
<summary><b>20.7 Buffer là gì, khác String thế nào, và lưu ý bảo mật khi cấp phát?</b></summary>

- `Buffer` là vùng nhớ nhị phân thuần, cấp phát NGOÀI V8 heap (off-heap), không tính trong `--max-old-space-size`; dùng cho dữ liệu raw (file, socket, crypto).
- String JS là UTF-16, immutable, không biểu diễn byte tùy ý an toàn.
- Bảo mật: ưu tiên `Buffer.alloc(size)` (zero-fill) thay vì `allocUnsafe` (nhanh hơn nhưng vùng nhớ chưa xóa, có thể lộ dữ liệu cũ).
- So sánh secret dùng `crypto.timingSafeEqual` (chống timing attack); hàm ném lỗi nếu khác độ dài → kiểm tra length trước.

```js
const a = Buffer.from(tokenFromCookie, 'utf8');
const b = Buffer.from(storedToken, 'utf8');
const ok = a.length === b.length && crypto.timingSafeEqual(a, b);
```

</details>

<details>
<summary><b>20.8 Làm thế nào để scale một ứng dụng Node trên 1 máy (cluster/PM2) và lưu ý khi có state?</b></summary>

- Logic JS chạy 1 thread → 1 process dùng 1 core. Dùng hết core bằng **cluster** hoặc **PM2** cluster (`pm2 start app.js -i max`); kernel/PM2 phân phối kết nối.
- Instance KHÔNG share memory → mọi state phải đẩy ra ngoài: session/refresh token, rate-limit counter, cache đều lưu Redis.
- Cron (`@Cron`) phải chỉ chạy 1 instance (leader election / distributed lock Redis / worker riêng) tránh chạy trùng.
- WebSocket/Socket.IO cần adapter (vd Redis adapter) để broadcast giữa các worker.

```bash
pm2 start dist/main.js -i max --name jira-api  # 1 process moi core, auto restart
```

</details>

<details>
<summary><b>20.9 Cơ chế xử lý lỗi (error handling) trong Node async khác gì giữa callback, Promise và EventEmitter?</b></summary>

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

<details>
<summary><b>20.10 AsyncLocalStorage là gì và vì sao hữu ích cho request context / tracing trong NestJS?</b></summary>

- `AsyncLocalStorage` (`node:async_hooks`) lưu kho dữ liệu gắn với MỘT chuỗi async (vd vòng đời 1 request) mà không phải truyền tham số xuyên suốt.
- Dữ liệu được bảo toàn qua ranh giới async (`await`, callback) nhờ `async_hooks`; mỗi request có "store" riêng, không lẫn.
- Ứng dụng: gắn `requestId`/`traceId`, user, tenant vào store → logger đọc ra cho mỗi dòng log có context (tracing/correlation).
- Lưu ý: có chi phí nhỏ; NestJS thường dùng qua `nestjs-cls`. Đừng dùng thay DI — chỉ cho cross-cutting context.

```js
const als = new AsyncLocalStorage();
app.use((req, res, next) => als.run(new Map([['requestId', crypto.randomUUID()]]), () => next()));
function log(msg) { console.log(`[${als.getStore()?.get('requestId')}] ${msg}`); }
```

</details>

---

## 21. TypeScript for Backend

<details>
<summary><b>21.1 Generics là gì và khi nào bạn dùng chúng trong code backend?</b></summary>

- Generics cho phép viết hàm/class/type chạy trên nhiều kiểu mà vẫn giữ type safety, thay vì `any`.
- Compiler tự suy luận (infer) kiểu tại chỗ gọi → giữ autocomplete và bắt lỗi lúc compile.
- Backend thường dùng cho wrapper tái sử dụng: repository, response DTO, hàm tiện ích.
- Ràng buộc bằng `extends` (vd `<T extends { id: string }>`) để giới hạn kiểu đầu vào; `keyof T` đảm bảo field hợp lệ.

```ts
interface ApiResponse<T> { data: T; meta: { total: number }; }
function paginate<T extends { id: string }>(items: T[]): ApiResponse<T[]> {
  return { data: items, meta: { total: items.length } };
}
```

</details>

<details>
<summary><b>21.3 type và interface khác nhau thế nào? Khi nào chọn cái nào?</b></summary>

- `interface`: mô tả shape object/class, `extends`, hỗ trợ declaration merging (khai báo trùng tên gộp lại).
- `type`: alias tổng quát hơn — biểu diễn được union, tuple, primitive, conditional/mapped type; mở rộng qua intersection `&`.
- Quy tắc: `interface` cho contract object ổn định; `type` cho union/intersection/kiểu phức tạp. Shape object thuần thì gần như tương đương.
- Lưu ý: interface dễ bị merge ngoài ý muốn → public API/augmentation ưu tiên interface; type nội bộ cần chắc không mở rộng thì an toàn hơn.

```ts
type IssueStatus = 'TODO' | 'IN_PROGRESS' | 'DONE'; // interface khong lam duoc
type WithTimestamps<T> = T & { createdAt: Date; updatedAt: Date };
```

</details>

<details>
<summary><b>21.4 Discriminated union là gì và vì sao nó hữu ích khi narrowing?</b></summary>

- Union của nhiều object cùng chia một field literal làm 'tag' (vd `type`/`kind`/`status`).
- Khi switch/if theo tag, TS tự narrow về đúng nhánh và biết chính xác các field còn lại.
- Biến các trạng thái loại trừ lẫn nhau thành kiểu an toàn; kết hợp `never` để check exhaustiveness lúc compile.

```ts
type Result =
  | { status: 'success'; data: string }
  | { status: 'error'; message: string };
// switch (r.status): case 'success' biet co .data, 'error' biet co .message
// default: const _e: never = r; -> them nhanh moi se bao loi
```

</details>

<details>
<summary><b>21.5 Type guard và narrowing hoạt động ra sao? Cho ví dụ custom type guard.</b></summary>

- Narrowing: TS thu hẹp kiểu trong một nhánh dựa trên kiểm tra runtime (`typeof`, `instanceof`, `in`, so sánh literal).
- Custom type guard: hàm trả `param is Type` (type predicate); nếu trả `true`, compiler coi tham số là kiểu đó trong nhánh sau.
- Quan trọng khi xử lý `unknown` từ input ngoài (request body, JSON parse) để chuyển an toàn sang kiểu đã biết.
- Lưu ý: predicate là lời hứa với compiler — logic sai sẽ tạo lỗ hổng; phải kiểm tra cả sự tồn tại lẫn kiểu từng field. Phức tạp thì dùng Zod/class-validator.

```ts
function isAuthUser(x: unknown): x is AuthUser {
  return typeof x === 'object' && x !== null &&
    'id' in x && typeof (x as any).id === 'string';
}
```

</details>

<details>
<summary><b>21.6 Decorators + reflect-metadata liên quan thế nào đến DI của NestJS?</b></summary>

- Decorator là hàm gắn vào class/method/property/parameter để thêm metadata hoặc sửa hành vi; cần bật `experimentalDecorators` + `emitDecoratorMetadata`.
- `emitDecoratorMetadata` phát ra metadata kiểu (vd `design:paramtypes`) qua `reflect-metadata`.
- NestJS đọc metadata này: `@Injectable()` + constructor có param kiểu `UserService` → container biết dependency cần resolve (DI dựa trên type).
- `@Inject(TOKEN)` khi cần token tùy chỉnh (interface, primitive, string/symbol token).
- Lưu ý: param decorator của constructor có `propertyKey` là `undefined`; nó chỉ lưu metadata, KHÔNG tự inject — container/guard mới resolve.

```ts
@Injectable()
class IssuesController { constructor(private users: UserService) {} }
Reflect.getMetadata('design:paramtypes', IssuesController); // [UserService]
```

</details>

<details>
<summary><b>21.7 satisfies và as const dùng để làm gì? Khác với type annotation thường?</b></summary>

- `as const`: literal thành readonly và narrow về kiểu hẹp nhất (vd `'TODO'` thay vì `string`); hợp cho config/hằng số.
- `satisfies` (TS 4.9+): kiểm tra expression thỏa mãn kiểu nhưng KHÔNG làm rộng kiểu suy diễn về kiểu đó.
- Annotation `: T` ép biến về đúng `T`, có thể mất thông tin literal/key cụ thể; `satisfies` vừa check ràng buộc vừa giữ kiểu cụ thể nhất cho autocomplete.

```ts
type Config = Record<string, { port: number }>;
const b = { db: { port: 5432 } } satisfies Config;
b.db.port;  // OK, autocomplete chi biet key 'db'
const ROLES = ['OWNER', 'MEMBER'] as const; // readonly tuple
```

</details>

<details>
<summary><b>21.8 Phân biệt unknown và any. Vì sao nên ưu tiên unknown?</b></summary>

- `any` tắt hoàn toàn kiểm tra kiểu và lan ra mọi nơi → mất an toàn.
- `unknown` là 'top type' an toàn: nhận mọi giá trị nhưng KHÔNG cho thao tác cho đến khi narrow (type guard, `typeof`, `in`, ép kiểu có kiểm soát).
- Dữ liệu ngoài (request body, response API, `JSON.parse`) nên dùng `unknown` rồi validate — buộc xử lý trường hợp sai kiểu thay vì âm thầm bỏ qua.

```ts
function parse(json: string): unknown { return JSON.parse(json); }
const data = parse('{}');
if (typeof data === 'object' && data !== null && 'id' in data) { /* OK sau narrow */ }
```

</details>

<details>
<summary><b>21.9 enum và union literal: nên dùng loại nào trong backend?</b></summary>

- `enum` có mặt lúc runtime: TS phát ra object thật (numeric enum có reverse mapping), tốn code và có điểm bất ngờ.
- Union literal (`'TODO' | 'DONE'`) chỉ tồn tại lúc compile: zero runtime cost, narrow tốt, dễ đồng bộ với DB.
- Xu hướng: ưu tiên union literal hoặc `as const` object; chỉ dùng `enum` khi có lý do rõ ràng. `const enum` inline nhưng vướng `isolatedModules`/bundler nên thường tránh.

```ts
type Status = 'TODO' | 'IN_PROGRESS' | 'DONE';
const Role = { OWNER: 'OWNER', MEMBER: 'MEMBER' } as const;
type Role = typeof Role[keyof typeof Role]; // 'OWNER' | 'MEMBER'
```

</details>

---

## 22. Advanced Testing

<details>
<summary><b>22.1 Test pyramid là gì? Tại sao nên có nhiều unit test hơn e2e test?</b></summary>

- Mô tả tỷ lệ lý tưởng: nhiều **unit test** ở đáy (nhanh, cách ly, rẻ), ít hơn **integration** ở giữa, rất ít **e2e** ở đỉnh (chậm, dễ flaky, đắt).
- Unit test cho feedback gần như tức thì và chỉ ra chính xác nơi lỗi.
- E2e chạy qua nhiều lớp (HTTP, DB, network) nên chậm và dễ flaky.
- Anti-pattern: "ice cream cone" (quá nhiều e2e, ít unit) khiến CI chậm, khó debug.

Ví dụ: logic sinh issue key `PROJECT-N` test kỹ bằng unit (mock Prisma); luồng login + httpOnly cookie chỉ cần vài e2e.

</details>

<details>
<summary><b>22.2 Phân biệt mock, spy, stub và fake. Khi nào dùng loại nào?</b></summary>

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
<summary><b>22.3 Làm sao test một Guard, Interceptor và Pipe trong NestJS?</b></summary>

- **Unit test** như class thường: tự tạo `ExecutionContext`/`ArgumentMetadata` giả, không quay cả HTTP.
- **Guard**: gọi trực tiếp `canActivate(context)` với context mock chứa `request.user`, assert kết quả.
- **Pipe**: gọi `transform(value, metadata)`, assert kết quả/lỗi.
- **Interceptor**: mock `CallHandler.handle()` trả observable rồi assert phần transform (dùng `firstValueFrom`).
- Lưu ý RBAC: thường guard **trả `false`** và Nest tự ném `ForbiddenException`; guard chỉ tự `throw` khi bạn chủ động viết. Test đúng hành vi guard thực sự làm.

```ts
// Guard tu throw khi thieu role
jest.spyOn(reflector, 'getAllAndOverride').mockReturnValue(['LEAD']);
expect(() => guard.canActivate(ctx)).toThrow(ForbiddenException);
```

</details>

<details>
<summary><b>22.4 Làm sao cô lập DB giữa các e2e test? So sánh truncate, transaction rollback và in-memory.</b></summary>

- Mục tiêu: mỗi test bắt đầu từ trạng thái sạch, không rò rỉ dữ liệu.
- **Truncate** sau mỗi test: đơn giản, dùng schema thật, nhưng chậm hơn; dùng `TRUNCATE ... RESTART IDENTITY CASCADE` để xóa dữ liệu liên quan + reset auto-increment.
- **Transaction rollback**: nhanh và sạch, nhưng **khó khi code tự mở transaction** (Prisma `$transaction` lồng nhau) hoặc nhiều connection (HTTP khác connection test).
- **In-memory (SQLite)**: nhanh nhưng lệch behavior so với Postgres (kiểu dữ liệu, constraint, isolation), dễ "green local, red prod".
- Ưu tiên: **Testcontainers Postgres + truncate** để sát production.

</details>

<details>
<summary><b>22.5 Code coverage đo cái gì và giới hạn của nó là gì? 100% coverage có nghĩa là không còn bug?</b></summary>

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
<summary><b>22.6 Integration test khác unit test và e2e test thế nào? Khi nào nên viết integration test?</b></summary>

- **Unit**: kiểm tra đơn vị cách ly (mock toàn bộ phụ thuộc).
- **Integration**: nhiều thành phần thật cùng làm việc (service + Prisma + DB thật), thường bỏ qua tầng HTTP/controller.
- **E2e**: chạy toàn bộ qua HTTP như người dùng thật.
- Integration đặc biệt giá trị khi rủi ro nằm ở **ranh giới** (ORM mapping, SQL sinh ra, transaction, constraint) — nơi mock che giấu lỗi.

Ví dụ: integration test repo sinh issue key với DB thật để chắc `$transaction` + unique constraint chặn được trùng key khi 2 request song song (mock Prisma không bao giờ bắt được).

</details>

<details>
<summary><b>22.7 Flaky test là gì, nguyên nhân phổ biến và cách phòng tránh?</b></summary>

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
<summary><b>22.8 Làm thế nào test code bất đồng bộ đúng cách? AAA pattern là gì?</b></summary>

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
<summary><b>22.9 Làm sao test Prisma trong NestJS: mock Prisma hay dùng DB thật? Công cụ nào hỗ trợ?</b></summary>

- **Mock/stub PrismaService** (unit): cách ly, nhanh, không cần DB; dùng `jest-mock-extended` (`mockDeep<PrismaClient>()`) để mock sâu method lồng nhau. Hợp khi test logic nghiệp vụ.
- **DB thật** (integration/e2e): dùng **Testcontainers** spin Postgres thật, chạy migration; bắt được lỗi mock che giấu (unique constraint, cascade, isolation, kiểu dữ liệu).
- Nguyên tắc: mock khi kiểm **logic**, DB thật khi kiểm **tương tác DB**. Assert "đã gọi create đúng tham số" KHÔNG xác nhận DB chấp nhận dữ liệu.

```ts
const prisma = mockDeep<PrismaService>();
prisma.issue.findUnique.mockResolvedValue(null);
```

</details>

<details>
<summary><b>22.10 Test data builder / factory là gì và tại sao quan trọng cho test đạt chất lượng?</b></summary>

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

## 23. System Design and Scaling

<details>
<summary><b>23.1 Modular monolith vs microservices: khi nào nên tách service? Đánh đổi là gì?</b></summary>

- Mặc định bắt đầu bằng **modular monolith**: deploy đơn giản, transaction xuyên module dễ (một `$transaction`), không có network latency / distributed tracing.
- Chỉ tách **microservices** khi có lý do thật: module cần scale độc lập, team lớn cần deploy độc lập, hoặc yêu cầu công nghệ khác biệt.
- Đánh đổi khi tách: mất ACID xuyên service (phải dùng saga/eventual consistency), phức tạp vận hành (service discovery, tracing, network failure), overhead serialize qua network.
- Nguyên tắc: đặt ranh giới module rõ trong monolith trước (theo bounded context, mỗi module sở hữu data riêng, giao tiếp qua interface) -> tách sau chỉ là "kéo module ra".

Ví dụ: Jira Clone là modular monolith (Auth, Workspace, Project, Issue); nếu module gửi email/OTP thành bottleneck thì tách riêng notification service tiêu thụ từ queue.

</details>

<details>
<summary><b>23.2 Scale ngang app stateless: vì sao không được lưu state in-memory? Ảnh hưởng gì đến thiết kế?</b></summary>

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
<summary><b>23.3 Caching với Redis: giải thích cache-aside và các chiến lược invalidation?</b></summary>

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
<summary><b>23.4 Message queue (Bull/RabbitMQ): khi nào dùng cho tác vụ nặng/async và lợi ích gì?</b></summary>

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
<summary><b>23.5 Multi-tenancy: các mô hình isolation dữ liệu? Anh đang áp dụng kiểu nào?</b></summary>

- (1) **Shared DB, shared schema**: mỗi row có `tenantId`, isolation bằng WHERE; rẻ, dễ scale, nhưng quên filter -> lộ data chéo.
- (2) **Shared DB, schema-per-tenant**: mỗi tenant một schema; isolation tốt hơn nhưng migration phức tạp (chạy trên mọi schema).
- (3) **Database-per-tenant**: isolation mạnh nhất, hợp enterprise/compliance, nhưng tốn kém và khó vận hành ở quy mô lớn.
- SaaS thường chọn (1). An toàn hơn: bật **RLS** ở Postgres và/hoặc guard bắt buộc mọi query có tenant scope (tốt nhất cả hai).

```ts
// Luon scope theo workspace, KHONG query "tran"
const issues = await prisma.issue.findMany({
  where: { project: { workspaceId: ctx.workspaceId } },
});
```

Lưu ý: RBAC kiểm soát "được làm gì", tenant scope kiểm soát "thấy data của ai" — hai thứ khác nhau. Jira Clone dùng shared schema với `workspaceId`.

</details>

<details>
<summary><b>23.6 Idempotency: tại sao cần và triển khai idempotency key cho API thế nào?</b></summary>

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

<details>
<summary><b>23.7 Database read replica: dùng để scale đọc thế nào? Cần lưu ý gì về replication lag?</b></summary>

- Khi tải **đọc** lớn hơn ghi nhiều: dùng **read replica**: primary nhận mọi write, replica đồng bộ từ primary và phục vụ read. Định tuyến write -> primary, read -> replica.
- Vấn đề cốt lõi: **replication lag** — replica chậm hơn primary, gây **read-your-write inconsistency** (vừa tạo issue, đọc list từ replica chưa kịp sync -> không thấy).
- Giải pháp: thao tác vừa-ghi-rồi-đọc-ngay thì đọc từ primary (read-after-write); chỗ không nhạy cảm chấp nhận eventual consistency.
- Lưu ý: PgBouncer chỉ là connection pooler, **không** tự định tuyến read/write. Replica còn dùng cho query phân tích nặng và standby HA.

```ts
import { readReplicas } from '@prisma/extension-read-replicas';
const prisma = new PrismaClient().$extends(readReplicas({ url: process.env.REPLICA_URL! }));
await prisma.issue.findMany();                    // -> replica
await prisma.$primary().issue.create({ data });   // ep tren primary
```

</details>

<details>
<summary><b>23.8 Rate limiting phân tán: tại sao throttler in-memory hỏng khi scale, và fix thế nào?</b></summary>

- @nestjs/throttler mặc định lưu counter **in-memory** từng instance. Scale ngang -> giới hạn bị nhân theo số instance (100 req/phút x 4 instance ~ 400), và counter mất khi restart.
- Fix: dùng **shared store**, phổ biến là **Redis** (vd `@nest-lab/throttler-storage-redis`) — atomic `INCR` + `EXPIRE` nên mọi instance cộng chung một counter.
- Thuật toán hay dùng: **sliding window** hoặc **token bucket**, thường viết bằng Lua script để gộp lệnh Redis thành một bước atomic.
- Nên rate limit theo nhiều khóa (per-IP, per-user, per-endpoint), chặn sớm ở edge/gateway; lấy đúng client IP sau proxy (X-Forwarded-For / `trust proxy`).

```ts
ThrottlerModule.forRootAsync({
  useFactory: () => ({
    throttlers: [{ limit: 100, ttl: 60000 }],
    storage: new ThrottlerStorageRedisService(redis), // shared counter
  }),
});
```

</details>

---

## 24. API and REST Design

<details>
<summary><b>24.1 Trình bày các REST conventions bạn tuân thủ khi đặt tên resource và thiết kế endpoint?</b></summary>

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
<summary><b>24.2 Bạn chọn HTTP status code như thế nào cho các case: tạo mới, validation fail, chưa đăng nhập, không đủ quyền, conflict?</b></summary>

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
<summary><b>24.3 Idempotency là gì? GET, POST, PUT, DELETE — cái nào idempotent, cái nào safe, và xử lý POST tạo trùng thế nào?</b></summary>

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
<summary><b>24.4 Các chiến lược API versioning? Bạn chọn cái nào và tại sao?</b></summary>

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
<summary><b>24.5 So sánh offset pagination và cursor pagination — khi nào dùng cái nào?</b></summary>

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
<summary><b>24.6 Response envelope { message, data } và error format nhất quán — bạn thiết kế ra sao trong NestJS?</b></summary>

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
<summary><b>24.8 Rate limiting và caching trong REST API — bạn thiết kế ra sao và dùng header nào?</b></summary>

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
<summary><b>24.9 Phân biệt PUT và PATCH, và làm sao xử lý cập nhật đồng thời (concurrent update / lost update)?</b></summary>

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

## 25. Scheduled and Background Jobs

<details>
<summary><b>25.1 @nestjs/schedule hoạt động thế nào? Phân biệt @Cron, @Interval, @Timeout và vai trò của ScheduleModule.forRoot()</b></summary>

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
<summary><b>25.2 Vì sao background job phải idempotent và bạn đảm bảo idempotency như thế nào?</b></summary>

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
<summary><b>25.3 Khi nào dùng queue-based processing (Bull/BullMQ) thay vì @Cron? So sánh</b></summary>

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
<summary><b>25.4 Khi deploy nhiều instance, làm sao chạy cron an toàn để job không chạy trùng? (distributed lock)</b></summary>

- Mỗi instance có scheduler riêng nên cùng `@Cron` chạy trên tất cả instance gây double-run.
- Distributed lock Redis `SET key value NX PX <ttl>`: chỉ instance giành khóa mới chạy; TTL tránh deadlock khi instance chết.
- Leader election (chỉ leader chạy cron).
- Tốt nhất: BullMQ repeatable/job scheduler — Redis đảm bảo mỗi lần đáo hạn tạo MỘT delayed job, chỉ MỘT worker pick up, không cần lock thủ công.
- Lưu ý lock tự làm: TTL > thời gian chạy tối đa; job vẫn nên idempotent; release phải kiểm tra đúng chủ sở hữu NGUYÊN TỬ bằng Lua (so value rồi `DEL`).

```ts
const ok = await this.redis.set('lock:nightly', this.instanceId, 'PX', 60000, 'NX');
if (ok !== 'OK') return; // instance khac da gianh khoa
try { await this.runJob(); }
finally {
  await this.redis.eval(
    "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end",
    1, 'lock:nightly', this.instanceId);
}
```

</details>

<details>
<summary><b>25.5 Retry và backoff trong xử lý job: bạn cấu hình thế nào và lưu ý gì?</b></summary>

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

<details>
<summary><b>25.6 Graceful shutdown cho background job/worker: tại sao cần và làm thế nào trong NestJS?</b></summary>

- Khi deploy/scale, container nhận SIGTERM; tắt đột ngột thì job đang chạy bị cắt giữa chừng gây dữ liệu không nhất quán hoặc job treo.
- Graceful shutdown: ngừng nhận job mới, chờ job hiện tại xong trong deadline, rồi đóng kết nối DB/Redis.
- NestJS: gọi `app.enableShutdownHooks()` trong `main.ts`, triển khai `OnModuleDestroy`/`beforeApplicationShutdown` để dọn dẹp; BullMQ gọi `worker.close()` (chờ job đang chạy xong).
- Kết hợp `terminationGracePeriodSeconds` của K8s phải lớn hơn deadline, nếu không K8s gửi SIGKILL cắt ngang.

```ts
@Injectable()
export class EmailWorker implements OnModuleDestroy {
  async onModuleDestroy() { await this.worker.close(); } // cho job hien tai xong
}
// main.ts: app.enableShutdownHooks();
```

</details>

<details>
<summary><b>25.7 Vì sao nên tách worker process ra khỏi API process? Đánh đổi gì?</b></summary>

- Cô lập tài nguyên: job nặng (CPU/IO) không làm tăng latency hay chiếm event loop của API.
- Scale độc lập (nhiều worker khi queue sâu, nhiều API khi traffic cao), deploy/restart riêng, giới hạn blast radius khi crash.
- Đánh đổi: phức tạp vận hành hơn — thêm deployment, cần broker (Redis), khó trace end-to-end (nên có correlation id).
- Thường dùng chung codebase nhưng entrypoint khác: worker dùng `createApplicationContext` (không mở HTTP), nạp module chỉ chứa provider/worker.

```ts
// main-worker.ts: chi nap WorkerModule, khong mo HTTP server
const app = await NestFactory.createApplicationContext(WorkerModule);
app.enableShutdownHooks();
```

</details>

<details>
<summary><b>25.8 Làm sao quan sát (observability) và cảnh báo cho background job? Đo gì và cảnh báo khi nào?</b></summary>

- Metrics: độ sâu queue (waiting/active/delayed), số job `failed`, thời gian xử lý (p50/p95), processing rate, job age — queue tăng không giảm là worker không theo kịp.
- Logging: mỗi job có correlation/trace id để nối request sang job; log kết quả và exception.
- Alerting: khi `failed` vượt ngưỡng, delayed/waiting tăng liên tục, job stalled, hoặc cron không chạy đúng lịch (heartbeat/dead-man-switch như Healthchecks.io).
- Dashboard: Bull Board / Arena để xem và retry job thủ công.

</details>

---

## 26. Worked Examples for Core Concepts

<details>
<summary><b>26.4 Request lifecycle trong NestJS — thứ tự Guard → Pipe → Interceptor → Handler → Filter chạy như thế nào? Ảnh hưởng tới ký tự nào?</b></summary>

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
<summary><b>26.5 JWT Refresh Token Rotation — vì sao cần? Cơ chế cấp lại token thế nào mà khó bị CSRF?</b></summary>

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
<summary><b>26.6 Transactional operations & atomic writes — vì sao cần khi phát hành issue key (PROJECT-N)? Concurrent requests có vấn đề gì?</b></summary>

- **Transaction** = nhóm thay đổi DB atomic: tất cả commit hoặc tất cả rollback. Cần khi nhiều bước phụ thuộc nhau.
- **Race condition** với issue key: 2 request cùng read counter = 5 → cả hai ghi PROJECT-6 → **trùng**.
- Khắc phục: **transaction + lock** (hoặc DB sequence / atomic increment).
- **pessimistic_write**: chặn cứng, request sau chờ. **optimistic** (`@Version`): không chặn, version đổi → conflict → retry; hợp high-concurrency.

```ts
async createIssue(dto: CreateIssueDto, projectId: string) {
  return this.dataSource.transaction(async (manager) => {
    const project = await manager.findOne(Project,
      { where: { id: projectId } },
      { lock: { mode: 'pessimistic_write' } }); // khóa ghi → request khác chờ
    const next = (project.issueCounter ?? 0) + 1;
    const issue = manager.create(Issue, { ...dto, key: `${project.key}-${next}`, projectId });
    await manager.save(issue);
    project.issueCounter = next;
    await manager.save(project); // exception → rollback cả issue lẫn counter
    return issue;
  });
}
// Tốt hơn: Postgres SEQUENCE → nextval() đảm bảo không trùng
```

</details>

<details>
<summary><b>26.8 Exception Filter vs Global Exception Filter (@Catch) — khi nào dùng custom filter? Thứ tự xử lý nếu có nhiều filter?</b></summary>

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

*Cập nhật lần cuối: bổ sung §14–§26 (Prisma, Advanced Auth, Web Security, Observability, Advanced NestJS, Transactions & Concurrency, Advanced Node, TypeScript, Testing, System Design, API/REST, Scheduled Jobs, Worked Examples) — bám sát dự án Jira Clone. File này được bổ sung sau mỗi section mới.*
---

## 27. GraphQL và Realtime trong NestJS

<details>
<summary><b>27.1 So sánh code-first và schema-first trong @nestjs/graphql? Bạn chọn cách nào và vì sao?</b></summary>

- **Code-first**: viết TypeScript class kèm decorator (`@ObjectType`, `@Field`...), package tự generate SDL (`schema.gql`). Source of truth là code.
- **Schema-first**: viết file `.graphql` (SDL) trước, generate ra TS types mà resolver phải implement. Source of truth là file SDL.
- Trade-off: code-first an toàn type end-to-end, không trùng định nghĩa → mặc định cho dự án NestJS thuần TS; schema-first hợp khi contract phải chia sẻ giữa nhiều team/ngôn ngữ (contract-first).
- Bẫy: code-first dễ "lệch" SDL nếu quên decorator; schema-first phải đồng bộ tay giữa SDL và resolver.

```ts
@ObjectType()
export class Issue {
  @Field(() => ID) id: string;
  @Field() title: string;
}
// Bật code-first:
GraphQLModule.forRoot({ driver: ApolloDriver, autoSchemaFile: 'src/schema.gql' });
```

</details>

<details>
<summary><b>27.2 Resolver trong GraphQL là gì? Phân biệt @Query, @Mutation, @ResolveField và @Args/@Parent?</b></summary>

- **Resolver** (`@Resolver(() => Type)`): class "phân giải" dữ liệu cho schema, giống controller nhưng map theo field thay vì route.
- `@Query` = entry point đọc (như GET); `@Mutation` = entry point ghi (POST/PATCH/DELETE).
- `@ResolveField` = phân giải **một field con**, chỉ chạy khi client thực sự chọn field đó (resolve quan hệ lồng nhau, ví dụ `Issue.assignee`).
- `@Args` lấy argument; `@Parent` lấy object cha (dùng trong `@ResolveField`).
- Lưu ý: `@ResolveField` **lazy** (chỉ query khi client cần) → lợi thế, nhưng với list dễ sinh N+1 (xem 27.3).

```ts
@ResolveField(() => User, { nullable: true })
assignee(@Parent() issue: Issue) {
  return issue.assigneeId ? this.userService.findById(issue.assigneeId) : null;
}
```

</details>

<details>
<summary><b>27.3 Vấn đề N+1 trong GraphQL phát sinh thế nào và DataLoader giải quyết ra sao?</b></summary>

- **N+1**: query trả về N item, mỗi item có `@ResolveField` tự truy vấn DB → 1 query list + N query con = N+1 query.
- **DataLoader** giải quyết bằng: (1) **Batching** — gom các `.load(id)` trong cùng tick thành 1 `batchFn` → chạy `WHERE id IN (...)` một lần; (2) **Cache theo request** — cùng key chỉ load 1 lần.
- Bẫy 1: loader phải tạo **per-request** (trong `context` factory) để cache không rò rỉ giữa user.
- Bẫy 2: `batchFn` PHẢI trả mảng cùng độ dài và **đúng thứ tự** với key đầu vào, nếu không map sai dữ liệu.

```ts
return new DataLoader<string, User>(async (ids) => {
  const users = await userService.findByIds([...ids]); // 1 query IN (...)
  const map = new Map(users.map((u) => [u.id, u]));
  return ids.map((id) => map.get(id) ?? null); // đúng thứ tự key
});
```

</details>

<details>
<summary><b>27.4 GraphQL Subscription hoạt động thế nào trong NestJS và khi nào nên dùng?</b></summary>

- **Subscription** = operation thứ ba (sau query/mutation), cho **server đẩy dữ liệu** khi có sự kiện, chạy trên WebSocket (`graphql-ws`).
- Code-first: khai báo `@Subscription` + dùng `PubSub` để publish/subscribe; có `filter` để chỉ đẩy đúng subscriber.
- Khi dùng: cần real-time theo schema GraphQL (live comment, notification, cập nhật trạng thái) mà client vẫn chọn đúng field.
- Cảnh báo: `PubSub` mặc định **in-memory**, chỉ phát trong 1 process → scale nhiều instance phải thay bằng `graphql-redis-subscriptions` (`RedisPubSub`).

```ts
@Subscription(() => Comment, {
  filter: (payload, vars) => payload.commentAdded.issueId === vars.issueId,
})
commentAdded(@Args('issueId') issueId: string) {
  return pubSub.asyncIterator('commentAdded');
}
```

</details>

<details>
<summary><b>27.5 @WebSocketGateway và @SubscribeMessage trong NestJS hoạt động ra sao? Vòng đời kết nối?</b></summary>

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
<summary><b>27.6 Authentication cho WebSocket Gateway khác gì với HTTP? Triển khai thế nào?</b></summary>

- WebSocket chỉ có **một lần handshake** lúc kết nối, sau đó kênh hai chiều giữ mở — không có guard chạy lại mỗi message như REST.
- Chiến lược: (1) xác thực **một lần lúc handshake** (token từ `handshake.auth`), sai thì `disconnect`; (2) gắn user vào `client.data` để các handler dùng lại.
- Guard cho từng message: dùng `context.switchToWs()` (không phải `switchToHttp()`), và ném `WsException` (không phải `HttpException`).
- Bẫy bảo mật: token hết hạn giữa phiên dài không tự phát hiện → cần kiểm tra lại định kỳ / trên action nhạy cảm và chủ động `disconnect` khi token bị thu hồi.

```ts
handleConnection(client: Socket) {
  try {
    client.data.user = this.jwtService.verify(client.handshake.auth?.token);
  } catch { client.disconnect(true); }
}
```

</details>

<details>
<summary><b>27.7 Vì sao WebSocket "vỡ" khi scale nhiều instance và Redis adapter giải quyết thế nào?</b></summary>

- Mỗi instance Socket.IO giữ kết nối/room **trong bộ nhớ riêng**. Client A nối instance #1, client B nối #2 → #1 emit chỉ tới client của #1, B ở #2 **không nhận**.
- Đây là vấn đề broadcast xuyên-instance, không phải sticky session đơn thuần.
- **Redis adapter** (`@socket.io/redis-adapter`): các instance publish/subscribe qua Redis Pub/Sub → #1 publish, mọi instance nhận và đẩy tiếp → room/broadcast hoạt động xuyên cluster.
- Lưu ý thêm: (1) nếu rơi về long-polling vẫn cần **sticky session** ở LB; (2) Redis adapter chỉ truyền sự kiện, không lưu message — client offline rồi quay lại không tự nhận message cũ.

```ts
const adapter = new RedisIoAdapter(app);
await adapter.connectToRedis(); // createAdapter(pubClient, subClient)
app.useWebSocketAdapter(adapter);
```

</details>

<details>
<summary><b>27.8 Khi nào chọn REST, GraphQL hay WebSocket? Tiêu chí quyết định?</b></summary>

- **REST**: CRUD, public API đơn giản, cần tận dụng HTTP cache/CDN và status code rõ; rủi ro over/under-fetch, nhiều round-trip.
- **GraphQL**: client (mobile/đa màn hình) cần lấy đúng tập field, dữ liệu nhiều quan hệ lồng nhau, giảm round-trip; đổi lại phải xử lý N+1, giới hạn complexity, khó cache.
- **WebSocket**: real-time/hai chiều độ trễ thấp (chat, notification, live board); rủi ro quản lý kết nối, scale (Redis adapter), auth khác kiểu.
- Thực tế dùng cả ba; cập nhật thưa có thể dùng SSE/polling cho đơn giản.

| | REST | GraphQL | WebSocket |
|---|---|---|---|
| Mô hình | Req/res, resource | Req/res, chọn field | Hai chiều, server đẩy |
| Caching | Dễ (CDN) | Khó (1 endpoint) | Không áp dụng |

</details>

<details>
<summary><b>27.9 Bảo mật một GraphQL API ở production gồm những gì? (depth limit, query complexity, introspection)</b></summary>

- Client tự ghép query → query lồng sâu/vòng lặp có thể gây DoS. Các lớp phòng thủ:
- **Depth limit**: giới hạn độ sâu lồng nhau, chặn query đệ quy `user -> posts -> author -> ...`.
- **Query complexity**: gán chi phí cho từng field, reject query vượt ngưỡng tổng (chính xác hơn depth vì tính cả số field/list).
- **Tắt introspection + Playground ở prod**: giảm lộ cấu trúc schema.
- Bổ sung: rate limiting, timeout/pagination bắt buộc trên list, persisted queries (allowlist).
- Lưu ý: tắt introspection chỉ "giảm thông tin", **không thay** authorization ở từng resolver/field; đừng để lộ stack trace ở prod.

```ts
GraphQLModule.forRoot({
  introspection: process.env.NODE_ENV !== 'production',
  playground: false,
  validationRules: [depthLimit(7)],
});
```

</details>

<details>
<summary><b>27.10 (Hành vi) Kể về một lần bạn xử lý sự cố hiệu năng/độ ổn định liên quan tới GraphQL hoặc WebSocket?</b></summary>

Trả lời theo cấu trúc **STAR**:

- **Situation**: Jira Clone sau khi thêm board real-time, hai người mở cùng board không thấy cập nhật của nhau; query GraphQL load board thỉnh thoảng chậm vài giây khi nhiều issue.
- **Task**: (1) làm real-time hoạt động xuyên các pod sau khi scale, (2) hạ thời gian phản hồi query board.
- **Action**: Real-time — xác định Socket.IO lưu room in-memory nên không broadcast xuyên pod → tích hợp `@socket.io/redis-adapter` + sticky session cho long-polling. GraphQL chậm — phát hiện N+1 (mỗi issue query riêng `assignee`/`comments`) → đưa DataLoader per-request vào `context`; siết depth limit và complexity.
- **Result**: Real-time ổn định trên 3 pod; query/board từ >100 còn 2-3, p95 giảm rõ. Bài học: dùng GraphQL/WebSocket phải tính scale (Redis adapter) và N+1 (DataLoader) ngay từ thiết kế.

</details>

---

*Cập nhật lần cuối: bổ sung §27 (GraphQL và Realtime trong NestJS — code-first vs schema-first, resolver/@ResolveField, N+1 & DataLoader, subscription, WebSocket Gateway/Socket.IO, Redis adapter, REST vs GraphQL vs WebSocket, bảo mật GraphQL) — bám sát dự án Jira Clone. File này được bổ sung sau mỗi section mới.*
---

## 28. Tư duy và Giải quyết vấn đề (Backend)

<details>
<summary><b>28.1 Khi một API trên production đột nhiên chậm/lỗi mà không deploy gì mới, bạn tiếp cận điều tra như thế nào?</b></summary>

- Nguyên tắc: đừng đoán mò — đi từ **triệu chứng → dữ liệu → giả thuyết → kiểm chứng**, dựa trên 3 trụ observability: **logs, metrics, traces**.
- Quy trình: (1) xác định **scope** (toàn bộ hay một endpoint/region/user, nhìn RED: Rate/Errors/Duration); (2) dựng **timeline** (không deploy ≠ không đổi: traffic tăng, feature flag, bảng phình to, cron, cloud degrade); (3) khoanh **tầng** (app/DB/cache/queue/dependency) bằng trace; (4) xác nhận bằng dữ liệu rồi viết postmortem blameless.
- Thủ phạm "không-cần-deploy": **connection pool cạn**, bảng lớn dần làm mất hiệu lực index, N+1 lộ khi data tăng, memory leak, lock/deadlock, downstream chậm nghẽn pool.

```ts
// Bắt slow request tại tầng app để có duration + correlation id
tap(() => {
  const ms = Number(process.hrtime.bigint() - start) / 1e6;
  if (ms > 500) this.logger.warn({ msg: 'slow_request', path: req.route?.path, durationMs: Math.round(ms) });
})
```

Ví dụ: `GET /boards/:id/issues` chậm dần — trace cho thấy 95% thời gian ở DB, log query (Prisma `log:['query']`) lộ N+1 khi load `assignee`; fix bằng `include`.

</details>

<details>
<summary><b>28.2 Correlation ID / request ID dùng để làm gì, và bạn truyền nó xuyên suốt một request NestJS như thế nào?</b></summary>

- **Correlation ID** là định danh duy nhất gắn vào mỗi request, truyền qua **mọi log/service/job** mà request đó sinh ra → `grep` một ID là gom được toàn chuỗi xử lý. Sống còn trong hệ phân tán.
- Cách làm NestJS: **middleware** sinh/đọc ID từ header `X-Correlation-Id`, lưu vào **AsyncLocalStorage** để mọi nơi truy cập không cần truyền tham số, rồi inject vào logger.

```ts
export const als = new AsyncLocalStorage<{ correlationId: string }>();
export function correlationMiddleware(req, res, next) {
  const id = req.headers['x-correlation-id'] ?? randomUUID();
  res.setHeader('x-correlation-id', id);
  als.run({ correlationId: id }, () => next());
}
```

- **Lưu ý**: khi đẩy việc sang **BullMQ**, phải nhét `correlationId` vào job data, nếu không chuỗi log "đứt" ở ranh giới async/queue.
- Chuẩn **OpenTelemetry**: tương ứng là `traceId` qua header `traceparent` (W3C Trace Context), auto-instrumentation tự nối span.

</details>

<details>
<summary><b>28.4 N+1 query là gì, vì sao nó "ẩn" cho tới khi data lớn, và cách phát hiện/khắc phục với Prisma, TypeORM và GraphQL?</b></summary>

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
<summary><b>28.5 Khi đứng trước một quyết định kiến trúc, bạn cân nhắc trade-off theo khung tư duy nào?</b></summary>

- Không có kiến trúc "đúng tuyệt đối", chỉ có cái **phù hợp ràng buộc hiện tại**.
- Khung: (1) làm rõ yêu cầu **functional + non-functional** (throughput, latency, availability, consistency, ngân sách, quy mô team — phần lớn quyết định bị non-functional chi phối); (2) liệt kê **2-3 phương án**; (3) đánh giá theo trục: phức tạp vận hành, chi phí, scale, tốc độ ship, **khả năng đảo ngược**; (4) quyết định dễ đảo thì làm nhanh, **one-way door** (DB chính, API public, phân chia service) thì cẩn trọng; (5) ghi lại bằng **ADR**.
- Định luật nền tảng: **CAP** (khi partition, chọn C hay A) và **PACELC** (kể cả không partition, vẫn trade Latency vs Consistency). Mọi cache/replica/queue đều mua hiệu năng bằng giá nhất quán/phức tạp.
- **Anti-pattern**: resume-driven development (chọn Kafka/k8s vì "ngầu") và premature optimization cho quy mô chưa có.
- Ví dụ: Blog/CMS giai đoạn đầu (vài nghìn user) → **modular monolith** NestJS + Postgres + Redis là đủ; tách microservice chỉ thêm phức tạp.

</details>

<details>
<summary><b>28.7 Hãy mô tả cách thực hiện một data migration lớn với zero-downtime (ví dụ: đổi tên cột, tách bảng, đổi kiểu dữ liệu).</b></summary>

- Nguy hiểm vì code cũ và schema mới phải **cùng tồn tại** lúc deploy rolling. Mẫu chuẩn: **expand → migrate/backfill → contract** (parallel change).
- (1) **Expand**: thêm cấu trúc mới, không xóa cũ (tương thích ngược); (2) **Double-write**: code ghi cả cũ lẫn mới (hoặc trigger DB); (3) **Backfill**: job nền điền dữ liệu theo **batch** (vd 1000 dòng/lần, có nghỉ), phải **idempotent + resume được**; (4) **Switch read**: đọc từ cấu trúc mới, xác minh khớp; (5) **Contract**: ngừng ghi cũ rồi `DROP` ở migration sau.

```sql
ALTER TABLE users ADD COLUMN full_name varchar(255);                    -- Expand
UPDATE users SET full_name = first_name || ' ' || last_name             -- Backfill batch
WHERE full_name IS NULL AND id IN (SELECT id FROM users WHERE full_name IS NULL LIMIT 1000);
ALTER TABLE users DROP COLUMN first_name, DROP COLUMN last_name;         -- Contract (deploy sau)
```

- **Lưu ý Postgres**: `ADD COLUMN DEFAULT` hằng số là tức thì từ PG 11+ (tránh default volatile vì rewrite bảng); index bảng lớn dùng **`CREATE INDEX CONCURRENTLY`** (không khóa ghi, không chạy trong transaction → Prisma cần `--create-only`); `ALTER COLUMN TYPE` thường rewrite + khóa → thay bằng cột-mới + backfill.
- Luôn có **plan rollback** và chạy thử trên bản sao production cỡ thật.

</details>

<details>
<summary><b>28.8 Khi nào nên tách một microservice ra khỏi monolith, và khi nào KHÔNG nên?</b></summary>

- **Microservice là giải pháp cho vấn đề tổ chức/quy mô, không phải mục tiêu**. Đánh đổi sự đơn giản (1 deploy, 1 transaction, gọi in-process) lấy scale/phát triển độc lập — kèm *thuế phân tán* (network không tin cậy, eventual consistency, tracing, deploy phức tạp).
- **Nên tách khi**: một phần có **profile mở rộng khác biệt** (vd xử lý ảnh/video ngốn CPU); **ranh giới team rõ** muốn deploy độc lập (Conway's Law); cần **cô lập lỗi/bảo mật** (vd payment) hoặc khác stack/tuân thủ.
- **KHÔNG nên** (giữ **modular monolith**): team nhỏ, chưa có product-market fit, domain boundary chưa rõ (tách sớm phải sửa biên giới liên tục); nhu cầu chỉ là "code sạch hơn" (giải bằng module hóa).
- Lộ trình: **Monolith → Modular Monolith → tách dần khi đau thật**. NestJS có `@nestjs/microservices` (TCP/Redis/NATS/RabbitMQ/Kafka); bước trung gian an toàn là tách tác vụ async nặng ra **worker BullMQ**.
- **Bẫy lớn nhất**: **distributed monolith** — tách service nhưng gọi đồng bộ chằng chịt và share database, nhận đủ nhược điểm cả hai.

</details>

<details>
<summary><b>28.9 Bạn nhận một yêu cầu (ticket) mơ hồ, ví dụ "thêm export báo cáo cho admin". Bạn làm gì trước khi viết code?</b></summary>

- Nguyên tắc: **làm rõ trước, code sau** — đặt câu hỏi có cấu trúc.
- (1) Làm rõ **"what" & "why"**: ai dùng, giải quyết vấn đề gì (export để tự phân tích Excel hay gửi khách?); (2) đào **non-functional**: định dạng & phạm vi (CSV/XLSX/PDF, cột, lọc), **khối lượng** (vài trăm dòng → xuất đồng bộ; hàng triệu → job BullMQ + **stream** ra file + lưu storage rồi gửi link, tránh giữ HTTP mở lâu & ngốn RAM), quyền hạn (role/tenant), PII/audit; (3) xác định **edge case** (rỗng, timezone, đồng thời); (4) chốt **acceptance criteria** rồi ước lượng, đề xuất chia phase nếu rộng.
- **Mẹo giao tiếp**: thay vì hỏi mở, **đề xuất giả định cụ thể để xác nhận**: "Em hiểu là export CSV issue đã đóng trong tháng, lọc theo project, chạy nền gửi link — đúng không ạ?" (câu đóng lấy quyết định nhanh hơn).
- Ví dụ: Jira Clone "export report" hóa ra là báo cáo velocity theo sprint — không hỏi dễ build nhầm thành export raw toàn bộ issue.

</details>

<details>
<summary><b>28.11 Bạn được giao maintain một codebase backend lớn, lạ, không có tài liệu. Bạn đọc hiểu nó như thế nào?</b></summary>

- Không đọc tuần tự từng file — đi từ **ngoài vào trong, từ luồng thực tới chi tiết**.
- (1) **Chạy được nó trước** (dựng local, gọi vài endpoint — hiểu build/test/run rõ hơn mọi tài liệu); (2) tìm **entry points** (NestJS: `main.ts`, `AppModule`, `controllers`, `imports` — cây module là sơ đồ kiến trúc); (3) **trace một luồng end-to-end** controller → service → repo → response (breakpoint/log); (4) đọc **schema DB** (Prisma/entity — mô hình dữ liệu tiết lộ domain rõ hơn code); (5) dùng **test như tài liệu sống** (mô tả hành vi & cách dùng API); (6) khảo sát **git log/blame** để biết *vì sao* và ai nên hỏi; (7) **hỏi đúng người câu cụ thể** sau khi đã tự tìm hiểu.
- Nguyên tắc bao trùm: **học just-in-time** (chỉ đào sâu phần liên quan task), ghi chú khi hiểu một mảnh, để CI/test làm lưới an toàn; một thay đổi nhỏ-an-toàn đầu tiên giúp xác nhận đã hiểu vòng đời dev.

</details>

## 30. MongoDB và Mongoose (NoSQL)

<details>
<summary><b>30.1 Khi nào bạn chọn MongoDB thay vì PostgreSQL? Cho ví dụ document model vs relational model.</b></summary>

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

<details>
<summary><b>30.2 EMBED và REFERENCE khác nhau thế nào? Khi nào denormalize bằng cách embed, khi nào tách reference?</b></summary>

- **Embed (nhúng)**: đặt dữ liệu con trong document cha, đọc một lần đủ. Hợp **one-to-few**, con luôn đi cùng cha.
- **Reference (tham chiếu)**: lưu `ObjectId` trỏ tới document khác (giống FK), rồi `populate`. Hợp **one-to-many lớn / many-to-many** hoặc dữ liệu được chia sẻ.
- Quy tắc: one-to-few + đọc cùng → embed; unbounded (comment, log) → reference (tránh phình to, giới hạn document 16MB); dữ liệu chia sẻ/cập nhật một chỗ → reference.
- **Denormalize**: nhúng bản sao vài field hiển thị (tên, avatar) để khỏi `$lookup`; đánh đổi là phải tự **đồng bộ** khi gốc đổi và rủi ro dữ liệu lệch (stale).

```js
// EMBED: địa chỉ ít, không query riêng -> nhúng
{ _id: 1, name: "Apollo", addresses: [{ city: "HCM" }] }
// REFERENCE: post có hàng nghìn comment -> tách + trỏ ObjectId
// comments: { _id: "c1", postId: "p1", text: "..." }
```

Bẫy hay gặp: "embed hay reference cho comment?" → reference vì comment nhiều/không giới hạn.

</details>

<details>
<summary><b>30.3 Định nghĩa Schema và Model trong Mongoose như thế nào? Phân biệt Schema, Model và Document.</b></summary>

- Mongoose là **ODM** đặt lớp schema + validation + type lên MongoDB vốn schemaless.
- **Schema**: định nghĩa hình dạng, kiểu, validation, index, default, virtual.
- **Model**: constructor compile từ schema, gắn với một **collection**, là interface CRUD (`User.find()`, `new User()`).
- **Document**: instance cụ thể trả về từ model, có method như `.save()`, `.populate()`.

```ts
const postSchema = new Schema({
  title: { type: String, required: true, trim: true },
  slug: { type: String, required: true, unique: true },
  status: { type: String, enum: ['draft', 'published'], default: 'draft' },
  author: { type: Types.ObjectId, ref: 'User', required: true }, // reference
}, { timestamps: true });                                         // tự thêm createdAt/updatedAt
const Post = model('Post', postSchema);                           // collection: "posts"
```

Nhớ: `ref` để `populate` biết trỏ model nào; tên collection mặc định là số nhiều, viết thường.

</details>

<details>
<summary><b>30.5 Có những loại index nào trong MongoDB (single, compound, unique, text, TTL) và đánh sao cho đúng?</b></summary>

- **Single**: một field. **Compound**: nhiều field, theo **rule ESR** (Equality → Sort → Range).
- **Unique**: cấm trùng (email, slug). **Text**: full-text cho `$text`. **TTL**: tự xoá sau N giây (session, OTP, log). **Partial/sparse**: index có điều kiện.
- Index tăng tốc đọc nhưng **làm chậm ghi** và tốn RAM/đĩa → đừng đánh bừa.
- Dùng `.explain('executionStats')` để kiểm tra `IXSCAN` (dùng index) hay `COLLSCAN` (quét toàn bộ); `unique` không bỏ qua trùng cũ — phải dọn data trước.

```ts
postSchema.index({ slug: 1 }, { unique: true });
postSchema.index({ status: 1, createdAt: -1 });                        // compound theo ESR
postSchema.index({ title: 'text', body: 'text' });                     // text search
sessionSchema.index({ createdAt: 1 }, { expireAfterSeconds: 3600 });   // TTL
```

ESR hay bị hỏi: query `find({status}).sort({createdAt:-1})` → index `{status:1, createdAt:-1}` phục vụ cả lọc và sort, tránh in-memory sort.

</details>

<details>
<summary><b>30.6 Aggregation pipeline là gì? Giải thích $match, $group, $lookup, $project qua một ví dụ.</b></summary>

- Chuỗi **stage** mà document đi qua tuần tự, mỗi stage biến đổi rồi truyền sang stage sau (như Unix pipe). Dùng cho báo cáo/thống kê.
- `$match`: lọc (như `WHERE`), **đặt sớm nhất** để giảm dữ liệu + tận dụng index.
- `$group`: gom nhóm + tính (`$sum`, `$avg`) như `GROUP BY`. `$lookup`: JOIN sang collection khác. `$project`: chọn/đổi field xuất ra (như `SELECT`).
- Thứ tự stage quyết định hiệu năng; `$lookup` không có index phía `from` sẽ rất chậm.

```ts
postModel.aggregate([
  { $match: { status: 'published' } },                          // lọc trước
  { $group: { _id: '$author', total: { $sum: 1 } } },           // gom theo author
  { $lookup: { from: 'users', localField: '_id', foreignField: '_id', as: 'authorDoc' } },
  { $project: { _id: 0, total: 1, name: { $arrayElemAt: ['$authorDoc.name', 0] } } },
]);
```

</details>

<details>
<summary><b>30.7 populate() hoạt động thế nào và làm sao tránh N+1 với nó?</b></summary>

- `populate()` thay field reference (`ObjectId`) bằng document thật từ collection được `ref` — mô phỏng JOIN ở tầng ứng dụng (MongoDB không tự JOIN khi find).
- Trên một list, `populate` **đã batch**: Mongoose gom tất cả id thành một query `$in` → `find + populate` thường chỉ 1+1 query, KHÔNG phải N+1.
- N+1 thật sự xuất hiện khi **lặp thủ công** và find/populate trong vòng lặp.
- Tránh: dùng `populate`, hoặc gom id `find({ _id: { $in: ids } })` một lần, hoặc `$lookup` trong aggregation.

```ts
// TỐT: 1 query + 1 query phụ ($in)
postModel.find().populate('author', 'name avatar');

// ANTI-PATTERN N+1: mỗi vòng 1 query
for (const post of posts) post.author = await userModel.findById(post.author);
```

</details>

<details>
<summary><b>30.8 lean() là gì và tại sao nó tăng tốc đọc đáng kể? Đánh đổi khi dùng lean()?</b></summary>

- Mặc định Mongoose "hydrate" kết quả thành **Document** đầy đủ (getter/setter, change tracking, virtual, `.save()`) → tốn CPU/RAM.
- `.lean()` trả về **POJO** thuần, bỏ bước hydrate → nhanh hơn vài lần và nhẹ hơn, lý tưởng cho endpoint **chỉ đọc** (list, search, report).
- Đánh đổi: mất method instance (`.save()`), không có virtual/getter/setter, hook document không chạy đúng, không change tracking.
- Quy tắc: `lean()` cho **đường đọc**; bỏ `lean()` khi cần biến đổi/lưu document.

```ts
const post = await postModel.findById(id);          // Document đầy đủ (để .save())
const posts = await postModel.find().lean();        // POJO, nhanh & nhẹ
```

Hỏi mẹo "API list chậm, tối ưu sao?" → index hợp lý + `.lean()` + `select` field cần + phân trang.

</details>

<details>
<summary><b>30.9 Mongoose middleware (hook) là gì? Cho ví dụ pre('save') và lưu ý quan trọng.</b></summary>

- **Hook** là hàm chạy tự động quanh thao tác (save, validate, query, aggregate); hai loại: `pre` (trước) và `post` (sau). Dùng để hash password, sinh slug, audit.
- **Bẫy lớn**: hook **document** (`save`, `validate`) có `this` là document; hook **query** (`findOneAndUpdate`, `updateMany`) có `this` là **query object** → `pre('save')` KHÔNG chạy khi dùng `findOneAndUpdate`/`updateOne` (lỗi hash/slug không kích hoạt rất phổ biến).
- Dùng `this.isModified(field)` để tránh xử lý lại không cần thiết.
- Trong NestJS: gắn hook qua `forFeatureAsync` + `useFactory`, gọi `schema.pre(...)` trước khi return.

```ts
userSchema.pre('save', async function (next) {
  if (!this.isModified('password')) return next();        // chỉ hash khi đổi
  this.password = await bcrypt.hash(this.password, 10);
  next();
});
```

</details>

<details>
<summary><b>30.10 Transaction đa document trong MongoDB hoạt động ra sao và yêu cầu hạ tầng gì?</b></summary>

- Thao tác trên **một document** vốn đã atomic. Cần atomic **nhiều document/collection** → **multi-document transaction** (MongoDB 4.0+).
- **Yêu cầu bắt buộc**: cụm chạy ở dạng **replica set** (hoặc sharded cluster); standalone KHÔNG hỗ trợ.
- Trong NestJS: `@InjectConnection()` lấy `connection`, `startSession()`, truyền `{ session }` vào mọi thao tác.
- Trade-off: transaction có chi phí (lock, write conflict) → triết lý Mongo là **thiết kế schema (nhúng) để hạn chế cần transaction**; local dev phải dựng replica set (`mongod --replSet rs0` + `rs.initiate()`).

```ts
const session = await connection.startSession();
try {
  session.startTransaction();
  await accountModel.updateOne({ _id: a }, { $inc: { balance: -100 } }, { session });
  await accountModel.updateOne({ _id: b }, { $inc: { balance: +100 } }, { session });
  await session.commitTransaction();        // tất cả hoặc không gì cả
} catch (e) { await session.abortTransaction(); throw e; }
finally { session.endSession(); }
```

</details>

<details>
<summary><b>30.12 ObjectId là gì và những điểm cần lưu ý khi dùng làm khoá chính so với UUID/auto-increment?</b></summary>

- `_id` mặc định là **ObjectId** — 12 byte: 4 byte timestamp + 5 byte random (machine/process) + 3 byte counter. Sinh ở client driver nên không cần round-trip lấy id.
- **Gần như tăng dần theo thời gian** → sort `_id` xấp xỉ sort theo thời gian tạo, chèn ít phân mảnh index (khác UUIDv4 ngẫu nhiên).
- Nhược: **đoán được thời điểm/thứ tự tạo** → cân nhắc UUID khi muốn id không lộ metadata.
- Bẫy: so sánh `ObjectId` với **string** không khớp → khi nhận id từ request (string) phải validate và ép sang `ObjectId`.

```ts
const id = new Types.ObjectId();
id.getTimestamp();                                      // suy ra thời điểm tạo
postModel.find({ author: new Types.ObjectId(authorId) }); // ép kiểu khi cần
```

So sánh: ObjectId (gọn 12 byte, sortable) vs UUID (không lộ thông tin, ngẫu nhiên gây index lớn) vs auto-increment (khó scale ngang vì cần điểm sinh số tập trung).

</details>

---

## 31. Git Workflow và Cộng tác

<details>
<summary><b>31.1 So sánh GitFlow, GitHub Flow và Trunk-Based Development? Khi nào chọn mô hình nào?</b></summary>

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
<summary><b>31.2 Phân biệt merge, rebase và squash merge? Khi nào dùng cái nào?</b></summary>

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
<summary><b>31.4 Quy trình giải quyết merge conflict ra sao? Conflict thường phát sinh từ đâu?</b></summary>

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
<summary><b>31.5 Conventional Commits là gì và nó liên kết với Semantic Versioning như thế nào?</b></summary>

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
<summary><b>31.6 Code review nên tập trung vào những gì? Vì sao PR nhỏ lại tốt hơn PR lớn?</b></summary>

- Format để cho linter/Prettier; reviewer tập trung thứ máy không kiểm được:
  - **Correctness**: logic, edge case, xử lý lỗi, race condition.
  - **Bảo mật**: injection, thiếu authz, lộ secret, log dữ liệu nhạy cảm.
  - **Thiết kế/bảo trì**: đúng layer, tránh trùng lặp, đặt tên rõ.
  - **Test** và **hợp đồng API** (breaking change).
- **PR nhỏ (<~300 dòng)**: review kỹ bắt được bug, merge nhanh, ít conflict, dễ revert; PR lớn dễ bị "LGTM" cho qua, treo lâu.
- Nguyên tắc: 1 PR làm **một việc**; tách refactor và feature riêng.

</details>

<details>
<summary><b>31.7 Phân biệt git revert và git reset? Trên branch chung phải dùng cái nào?</b></summary>

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
<summary><b>31.9 .gitignore dùng làm gì, và lỡ commit secret rồi thì xử lý thế nào?</b></summary>

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

<details>
<summary><b>31.11 git bisect giúp tìm commit gây lỗi như thế nào?</b></summary>

- `git bisect` dùng **binary search** trên lịch sử để tìm commit đầu tiên gây bug: chỉ ~`log2(n)` lần kiểm tra (1000 commit → ~10 bước).
- Đánh dấu commit `bad` (đang lỗi) và `good` (còn tốt); Git checkout commit giữa, bạn test rồi báo `good`/`bad`, lặp đến khi thu hẹp; xong `git bisect reset`.
- Tự động bằng `git bisect run <script>` (exit 0 = good, khác 0 = bad).
- Điều kiện: bug phải tái hiện được bằng một lệnh/test rõ ràng; mỗi commit nên build/chạy được (lý do để commit nhỏ, atomic).

```bash
git bisect start HEAD v1.4.0
git bisect run pnpm jest src/issue/issue.service.spec.ts
git bisect reset
```

</details>

---

## 32. CI/CD và Triển khai (Docker, Cloud)

<details>
<summary><b>32.1 Phân biệt CI, Continuous Delivery và Continuous Deployment? Ranh giới giữa ba khái niệm này nằm ở đâu?</b></summary>

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
<summary><b>32.2 Một pipeline CI cho service NestJS nên gồm những giai đoạn nào và thứ tự ra sao? Vì sao thứ tự đó quan trọng?</b></summary>

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
<summary><b>32.6 Quản lý secret trong GitHub Actions như thế nào? Phân biệt repository secret, environment secret và variable; cảnh báo bảo mật cần lưu ý.</b></summary>

- **Repository secrets** (`secrets.X`): mã hóa, dùng chung mọi workflow của repo.
- **Environment secrets**: gắn với một `environment` (vd `production`), kết hợp được required reviewers + wait timer.
- **Variables** (`vars.X`): giá trị **không nhạy cảm**, hiện rõ trong log (region, tên service).
- Cảnh báo: secret tự mask trong log nhưng **biến đổi nó** (base64, cắt chuỗi) thì mất mask → rò rỉ; PR từ fork không được cấp secret (`pull_request_target` rất nguy hiểm); ưu tiên **OIDC** lấy token tạm thay vì lưu cloud key tĩnh.

```yaml
permissions: { id-token: write }   # cho OIDC
- uses: aws-actions/configure-aws-credentials@v4
  with: { role-to-assume: arn:...:role/gha-deploy }  # AssumeRole, không cần key
```

</details>

<details>
<summary><b>32.7 Viết một multi-stage Dockerfile cho NestJS sao cho image nhỏ, build nhanh và chạy non-root. Giải thích từng stage.</b></summary>

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
<summary><b>32.9 Trình bày cách deploy một container NestJS lên cloud (ví dụ Cloud Run hoặc ECS). Những yêu cầu hạ tầng nào service phải tuân thủ để chạy được?</b></summary>

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
<summary><b>32.10 Quản lý biến môi trường và secret theo từng môi trường (dev/staging/prod) như thế nào cho an toàn? `.env` file đóng vai trò gì và không nên làm gì?</b></summary>

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

<details>
<summary><b>32.11 Chạy database migration trong pipeline sao cho an toàn? Vì sao không nên dùng `synchronize: true` và cách xử lý migration phá vỡ (breaking) không downtime?</b></summary>

- Migration là thao tác **trạng thái** (không idempotent) → cần kỷ luật riêng.
- **Tắt auto-sync** ở mọi môi trường thật: `synchronize: true` (TypeORM) / `db push` (Prisma) tự đổi schema → có thể **drop cột/bảng, mất dữ liệu**; prod dùng migration file version control.
- Chạy migration **một bước riêng trước switch traffic** (không nhồi lúc app khởi động trên N instance → race).
- **Không downtime** dùng **expand/contract**: (1) Expand — thêm cột mới, code ghi cả hai, đọc ưu tiên cột mới; (2) Backfill dữ liệu; (3) Contract — sau khi mọi instance lên version mới mới `DROP` cột cũ.
- Khác: tránh `ALTER TABLE` khóa bảng lâu trên bảng lớn (`CREATE INDEX CONCURRENTLY`), backup trước migration rủi ro cao.

```yaml
migrate:
  needs: build
  steps:
    - run: pnpm prisma migrate deploy
deploy:
  needs: migrate      # deploy app sau khi migrate xong
```

</details>

<details>
<summary><b>32.12 So sánh các chiến lược deploy zero-downtime: rolling, blue-green và canary. Mỗi cái phù hợp khi nào? Vai trò của graceful shutdown?</b></summary>

- **Rolling**: thay dần từng instance; không cần gấp đôi tài nguyên nhưng tồn tại cả 2 version → cần tương thích ngược. Mặc định cho service stateless.
- **Blue-green**: dựng full môi trường mới song song, switch 100% một nhát; switch/rollback tức thì nhưng tốn gấp đôi tài nguyên. Hợp release nhạy cảm.
- **Canary**: đẩy traffic dần (5%→25%→100%) + theo dõi metric; giới hạn bán kính ảnh hưởng nhưng cần routing % + monitoring tốt. Hợp thay đổi rủi ro cao.
- **Graceful shutdown** là tiên quyết: nhận `SIGTERM` → ngừng nhận request mới, xử lý nốt request đang chạy, đóng DB/queue rồi thoát (không thì lỗi 502). Mọi thay đổi phải tương thích ngược vì luôn có lúc 2 version cùng chạy.

```ts
app.enableShutdownHooks();   // NestJS bắt SIGTERM/SIGINT

@Injectable()
export class QueueService implements OnApplicationShutdown {
  async onApplicationShutdown() { await this.worker.close(); }
}
```

</details>

<details>
<summary><b>32.13 Health check và smoke test khác nhau ra sao? Phân biệt liveness và readiness probe, và viết một health endpoint trong NestJS.</b></summary>

- **Health check**: endpoint nội bộ để orchestrator biết instance sống/sẵn sàng. **Smoke test**: tập nhỏ kịch bản nghiệp vụ chạy **sau deploy** (login, tạo issue...) qua HTTP từ ngoài.
- **Liveness**: "process còn sống?" Fail → **restart**. Phải nhẹ, **không phụ thuộc DB** (tránh restart loop).
- **Readiness**: "sẵn sàng nhận traffic?" Fail → **gỡ khỏi LB** không restart. Nơi check dependency (DB, Redis).
- NestJS dùng `@nestjs/terminus`; nếu smoke test fail → coi như deploy hỏng, kích hoạt rollback.

```ts
@Get('live') @HealthCheck()
live() { return this.health.check([]); }          // nhẹ

@Get('ready') @HealthCheck()
ready() { return this.health.check([
  () => this.db.pingCheck('database', { timeout: 1500 }),
]); }
```

</details>

<details>
<summary><b>32.14 Khi một bản deploy gây lỗi production, cơ chế rollback nên hoạt động thế nào? Tại sao "rollback DB migration" lại nguy hiểm và làm gì để hạn chế?</b></summary>

- Rollback tốt = quay về **artifact đã biết là tốt** nhanh và xác định: tag image theo **git SHA** (không dùng `latest` cho prod), rollback = redeploy tag cũ. Blue-green/canary rollback nhanh nhất.
- **Rollback DB migration nguy hiểm**: code quay lui được nhưng schema/dữ liệu thì không — `DROP COLUMN` đã chạy thì rollback code không khôi phục dữ liệu; migration "down" thường viết qua loa, hiếm khi test.
- Hạn chế: (1) **Tách deploy code/migration** + migration tương thích ngược (expand/contract) để rollback code không cần rollback schema; (2) **Không drop ngay** — contract ở deploy sau; (3) backup trước thao tác phá hủy, ưu tiên "soft" (đổi tên `_deprecated`) thay DROP cứng; (4) **Feature flag** để tắt tính năng lỗi tức thì.

```bash
gcloud run services update-traffic api --to-revisions=api-00042-abc=100
kubectl rollout undo deployment/api
```

</details>

---

## 33. Tích hợp Third-party và Khả năng chống chịu

<details>
<summary><b>33.1 Khi gọi một API bên ngoài, những lớp bảo vệ tối thiểu nào bạn luôn áp dụng và vì sao?</b></summary>

- Mọi call mạng ra ngoài đều có thể chậm/lỗi/treo; client "trần" sẽ kéo cả service chết (cascading failure).
- **Timeout** (bắt buộc): cả connect timeout và total timeout, tránh treo giữ cạn pool connection.
- **Retry có kiểm soát**: chỉ với lỗi tạm thời (network, 502/503/504, 429) và request idempotent.
- **Circuit breaker**: provider lỗi liên tục thì ngừng gọi một thời gian.
- **Bulkhead / giới hạn concurrency**: cô lập tài nguyên cho từng provider.
- Bẫy: timeout quá dài (30s) vẫn giữ tài nguyên lâu; nhiều client mặc định timeout vô hạn.

```ts
// fetch + AbortController
const ac = new AbortController();
const t = setTimeout(() => ac.abort(), 5000);
try {
  const res = await fetch(url, { signal: ac.signal });
  if (!res.ok) throw new Error(`Upstream ${res.status}`);
  return await res.json();
} finally {
  clearTimeout(t); // luôn dọn timer, tránh leak
}
```

</details>

<details>
<summary><b>33.2 Giải thích retry với exponential backoff và jitter. Vì sao jitter lại quan trọng?</b></summary>

- Retry tức thì rất nguy hiểm: provider quá tải mà hàng loạt client thử lại cùng lúc → **retry storm**.
- **Exponential backoff**: thời gian chờ tăng theo hàm mũ (`base * 2^attempt`): 1s, 2s, 4s... giảm áp lực dần.
- **Jitter**: thêm ngẫu nhiên vào thời gian chờ; không có jitter thì mọi client retry đúng các mốc 1s/2s/4s → **thundering herd**. Jitter trải đều các lần thử.
- Lưu ý: tổng retry và `cap` phải gói trong timeout tổng của request gọi đến bạn, kẻo client phía trên đã timeout mà vẫn retry vô ích.

```ts
// Full jitter (AWS khuyến nghị)
const exp = Math.min(cap, base * 2 ** attempt);
const delay = Math.random() * exp; // random trong [0, exp]
await new Promise((r) => setTimeout(r, delay));
```

</details>

<details>
<summary><b>33.3 Idempotency là gì và vì sao bắt buộc khi retry các thao tác gây side-effect (như thanh toán)?</b></summary>

- **Idempotent**: gọi nhiều lần cho cùng kết quả như gọi một lần, không tạo side-effect lặp.
- Vấn đề: `POST /charges` bị **timeout** thì không biết provider đã xử lý chưa; retry mù → trừ tiền 2 lần.
- Giải pháp: gửi **Idempotency-Key** (UUID) trong header; provider lưu key, trùng thì trả kết quả cũ.
- Server của bạn cũng nên hỗ trợ idempotency cho POST nhạy cảm (lưu key unique + response đã trả).
- Mấu chốt: key phải **ổn định qua các lần retry** — gắn với định danh nghiệp vụ (orderId), đừng `randomUUID()` mới mỗi lần.

```ts
const idempotencyKey = `charge:${orderId}`; // gắn nghiệp vụ -> ổn định khi retry
return stripe.paymentIntents.create({ amount, currency: 'usd' }, { idempotencyKey });
```

</details>

<details>
<summary><b>33.4 Circuit breaker hoạt động thế nào và vì sao retry không thôi là chưa đủ?</b></summary>

- Retry chỉ trị lỗi **thoáng qua**; khi provider chết hẳn vài phút, mỗi request đợi timeout rồi retry → cạn thread/connection, lỗi lan ngược vào bạn.
- **Circuit breaker** theo dõi tỉ lệ lỗi, có 3 trạng thái:
  - **Closed**: bình thường, cho qua và đếm lỗi.
  - **Open**: lỗi vượt ngưỡng → **fail fast** ngay, không gọi provider trong `resetTimeout`.
  - **Half-open**: hết `resetTimeout`, cho vài request thăm dò; thành công → Closed, lỗi → Open lại.
- Ba lớp bổ trợ: timeout (1 request) + retry (lỗi lẻ) + breaker (lỗi diện rộng/kéo dài).
- Bẫy: ngưỡng quá thấp → breaker mở liên tục; dùng chung 1 breaker cho nhiều provider (nên mỗi provider một breaker).

```ts
const breaker = new CircuitBreaker(callPaymentProvider, {
  timeout: 4000, errorThresholdPercentage: 50, resetTimeout: 30_000,
});
breaker.fallback(() => ({ status: 'queued' }));
await breaker.fire(orderId); // fail fast khi Open
```

</details>

<details>
<summary><b>33.5 Cách nhận webhook ĐÚNG: trình tự xử lý nào bạn áp dụng để vừa an toàn vừa không mất event?</b></summary>

- **Verify chữ ký trước tiên** (chống giả mạo): cần **raw body** để tính HMAC → tắt JSON parser cho route này.
- **Idempotency**: dùng `event.id` của provider làm khóa; đã xử lý thì bỏ qua (provider hay gửi lặp khi không nhận 2xx).
- **Trả 2xx thật nhanh** rồi **xử lý nặng async qua queue**; xử lý đồng bộ chậm → provider timeout, gửi lại hoặc disable webhook.
- Phân biệt lỗi: signature hỏng → **4xx** (provider không cần retry); lỗi của mình (DB down) → **5xx** để provider retry.

```ts
@Post('stripe') @HttpCode(200)
async handle(@Req() req: RawBodyRequest<Request>, @Headers('stripe-signature') sig: string) {
  let event: Stripe.Event;
  try { event = stripe.webhooks.constructEvent(req.rawBody, sig, secret); }
  catch { throw new BadRequestException('Invalid signature'); } // 400
  if (await this.markSeen(event.id))                            // idempotency
    await this.queue.add('process-stripe-event', { id: event.id });
  return { received: true };                                    // 2xx ngay
}
```

NestJS: bật `rawBody: true` khi `NestFactory.create` để có `req.rawBody`.

</details>

<details>
<summary><b>33.6 Verify chữ ký webhook bằng HMAC như thế nào, và vì sao phải dùng so sánh "timing-safe"?</b></summary>

- Provider ký payload bằng **shared secret** qua HMAC (thường SHA-256). Bạn tính lại HMAC trên **raw body** với cùng secret rồi so sánh; khớp → payload đúng provider và chưa bị sửa.
- So sánh bằng **`crypto.timingSafeEqual`**, KHÔNG dùng `===`: so sánh thường thoát ở ký tự khác đầu tiên, lộ thời gian → **timing attack** đoán dần chữ ký.
- Kiểm tra **timestamp** để chống **replay** — từ chối event quá cũ (vd > 5 phút).
- Bẫy: tính HMAC trên body đã `JSON.parse` rồi stringify lại (đổi thứ tự key/khoảng trắng → không khớp); phải dùng raw bytes y hệt.

```ts
const expected = createHmac('sha256', secret).update(rawBody).digest('hex');
const a = Buffer.from(expected), b = Buffer.from(signature);
return a.length === b.length && timingSafeEqual(a, b);
```

</details>

<details>
<summary><b>33.7 Provider trả 429 (rate limit). Bạn xử lý ra sao và Retry-After đóng vai trò gì?</b></summary>

- `429` = gọi quá hạn mức provider.
- **Tôn trọng header `Retry-After`** (số giây hoặc HTTP-date): nói rõ phải chờ bao lâu; backoff của bạn nên nhường giá trị này.
- Không có `Retry-After` → exponential backoff + jitter.
- **Chủ động giới hạn tốc độ phía mình** (client-side rate limit / token bucket) để không chạm trần ngay.
- Khối lượng lớn → gom vào **queue có concurrency giới hạn** để rải đều.

| Tín hiệu | Phản ứng đúng |
|---|---|
| 429 + Retry-After: 30 | chờ đúng 30s rồi thử lại |
| 429 không kèm header | exponential backoff + jitter |
| 429 lặp liên tục | bật circuit breaker, giảm concurrency |

</details>

<details>
<summary><b>33.9 Outbox pattern là gì và nó giải quyết vấn đề gì khi tích hợp với hệ thống bên ngoài/queue?</b></summary>

- Vấn đề **dual write**: ghi DB và publish event ở hai hệ thống khác nhau, không bọc chung transaction được.
  - Commit DB rồi publish lỗi → **mất event**.
  - Publish trước rồi DB lỗi → **event ma**.
- **Outbox pattern**: ghi event vào bảng `outbox` **trong cùng transaction** với thay đổi nghiệp vụ; một relay/poller (hoặc CDC) đọc outbox và publish ra queue, đánh dấu đã gửi → không bao giờ mất event.
- Lưu ý: outbox là **at-least-once** (có thể publish trùng nếu relay crash giữa publish và update) → **consumer phải idempotent** (33.3). Outbox và idempotency luôn đi cặp.

```ts
await prisma.$transaction(async (tx) => {
  const order = await tx.order.create({ data: dto });
  await tx.outbox.create({ data: { type: 'order.created', payload: order, status: 'PENDING' } });
});
// relay nền: đọc PENDING -> publish (jobId chống trùng) -> đánh dấu SENT
```

</details>

<details>
<summary><b>33.10 Fallback và graceful degradation: khi một provider chết, service của bạn cư xử ra sao?</b></summary>

- **Graceful degradation**: dependency non-critical chết thì **giảm chức năng**, không sập hoàn toàn. Chiến lược tùy tính chất nghiệp vụ:
  - **Non-critical** (gợi ý, thumbnail): default value / cache cũ / bỏ qua, vẫn dùng tính năng chính.
  - **Critical nhưng trì hoãn được** (email, analytics): **enqueue** xử lý sau, không chặn luồng chính.
  - **Critical không thể giả** (thanh toán): không "giả thành công"; báo lỗi rõ ràng, không tạo dữ liệu sai.
- Kết hợp circuit breaker: `breaker.fallback(...)` chính là nơi cài graceful degradation.

```ts
try {
  return await breaker.fire(userId);
} catch {
  return this.cache.get(`reco:${userId}`) ?? await this.popularItems(); // cache cũ -> degrade
}
```

</details>

---

## 34. SQL Thuần và Thiết kế CSDL

> **Cơ bản → Nâng cao.** Phần này tập trung kỹ năng *viết và đọc SQL thuần* (query + thiết kế bảng) — phần bạn còn yếu — và **bổ trợ** cho [Phần 19](#19-database-transactions-and-concurrency) (đã lo transaction, lock, isolation, deadlock). Ví dụ dùng schema Jira Clone trên **PostgreSQL**: `project(id, key, name)`, `issue(id, project_id, assignee_id, status, priority, story_points, sprint_id, parent_id, created_at, deleted_at)`, `users(id, name, email)`, `comment(id, issue_id, author_id, created_at)`, `sprint(id, project_id, name)`.

### 🟢 Cơ bản

<details>
<summary><b>34.1 SQL là gì? Phân biệt nhóm lệnh DDL, DML, DCL, TCL?</b></summary>

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
<summary><b>34.2 Thứ tự thực thi LOGIC của một câu SELECT là gì? Vì sao không alias được ở WHERE?</b></summary>

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
<summary><b>34.3 Các loại JOIN (INNER, LEFT, RIGHT, FULL, CROSS) khác nhau thế nào?</b></summary>

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
<summary><b>34.4 GROUP BY và các hàm tổng hợp (COUNT/SUM/AVG/MIN/MAX) hoạt động ra sao?</b></summary>

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
<summary><b>34.5 WHERE và HAVING khác nhau ở đâu?</b></summary>

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
<summary><b>34.6 NULL trong SQL có gì đặc biệt? Các bẫy thường gặp?</b></summary>

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
<summary><b>34.7 Phân biệt PRIMARY KEY, FOREIGN KEY và UNIQUE constraint?</b></summary>

- **PRIMARY KEY**: định danh duy nhất mỗi dòng — **duy nhất + NOT NULL**, mỗi bảng chỉ một (có thể gồm nhiều cột = composite key).
- **FOREIGN KEY**: ràng buộc tham chiếu sang khóa của bảng khác, đảm bảo **toàn vẹn tham chiếu** (không trỏ tới id không tồn tại); kèm hành vi `ON DELETE CASCADE/SET NULL/RESTRICT` (xem 19.8).
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
<summary><b>34.8 Subquery: phân biệt scalar, IN, EXISTS và correlated subquery?</b></summary>

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
<summary><b>34.9 Vì sao LEFT JOIN rồi COUNT(*) bị sai, mà COUNT(cột) lại đúng?</b></summary>

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
<summary><b>34.10 UNION vs UNION ALL? Khi nào dùng cái nào?</b></summary>

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
<summary><b>34.11 Dùng CASE WHEN để "pivot" / conditional aggregation thế nào?</b></summary>

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
<summary><b>34.12 Chuẩn hóa (1NF, 2NF, 3NF) là gì? Khi nào cố ý denormalize?</b></summary>

- **1NF**: mỗi ô là **giá trị nguyên tử**, không nhóm lặp/mảng nhồi trong 1 cột (vd không nhét "label1,label2" vào một cột text → tách bảng `issue_label`).
- **2NF**: đạt 1NF + **không phụ thuộc một phần** vào *một phần* khóa chính ghép (mọi cột phải phụ thuộc **toàn bộ** khóa).
- **3NF**: đạt 2NF + **không phụ thuộc bắc cầu** (cột non-key không phụ thuộc cột non-key khác — vd đừng lưu `project_name` trong bảng `issue`, chỉ giữ `project_id`).
- Mục tiêu: giảm trùng lặp & sai lệch dữ liệu. **Denormalize** (cố ý lặp dữ liệu) chỉ khi cần **tốc độ đọc** cao và chấp nhận tự đồng bộ (vd cache `comment_count`, xem 30.2) — đánh đổi đúng-đắn lấy hiệu năng.

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
<summary><b>34.13 Index giúp gì? "Sargable" là gì? Vì sao nhiều WHERE không dùng được index?</b></summary>

- Index (B-tree) cho phép DB **tìm nhanh** thay vì quét toàn bảng (chi tiết loại index ở 19.6).
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
<summary><b>34.14 Phân trang OFFSET vs keyset (cursor) — cái nào tốt cho danh sách issue lớn?</b></summary>

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
<summary><b>34.15 Window function là gì? Khác GROUP BY ra sao? Phân biệt ROW_NUMBER/RANK/DENSE_RANK?</b></summary>

- Window function tính toán **trên một "cửa sổ" các dòng liên quan** nhưng **không gộp dòng lại** — mỗi dòng vẫn giữ nguyên, có thêm cột tính toán.
- Khác `GROUP BY` (làm **co** N dòng thành 1): window **giữ đủ N dòng**.
- `OVER (PARTITION BY ... ORDER BY ...)` định nghĩa cửa sổ; ranking:
  - `ROW_NUMBER()`: số thứ tự **duy nhất** 1,2,3...
  - `RANK()`: đồng hạng thì **nhảy cóc** (1,1,3).
  - `DENSE_RANK()`: đồng hạng **không nhảy** (1,1,2).

```sql
-- Xep hang issue trong moi project theo story_points giam dan
SELECT id, project_id, story_points,
  ROW_NUMBER() OVER (PARTITION BY project_id ORDER BY story_points DESC) AS rn
FROM issue;
```

</details>

<details>
<summary><b>34.16 "Greatest-N-per-group": lấy issue MỚI NHẤT của mỗi project như thế nào?</b></summary>

- Bài toán kinh điển: mỗi nhóm lấy 1 (hoặc N) dòng top theo một tiêu chí — `GROUP BY` thuần **không làm được** (nó mất các cột chi tiết).
- **Cách 1 — window**: đánh `ROW_NUMBER()` trong mỗi partition rồi lọc `= 1` (tổng quát, lấy top-N dễ).
- **Cách 2 — `DISTINCT ON` (Postgres)**: ngắn gọn cho top-1.
- Tránh kiểu subquery `MAX(created_at)` rồi join lại — dễ sai khi trùng thời điểm.

```sql
-- Cach 1: window
SELECT * FROM (
  SELECT i.*, ROW_NUMBER() OVER (PARTITION BY project_id ORDER BY created_at DESC) rn
  FROM issue i
) t WHERE rn = 1;

-- Cach 2: DISTINCT ON (Postgres)
SELECT DISTINCT ON (project_id) *
FROM issue ORDER BY project_id, created_at DESC;
```

</details>

<details>
<summary><b>34.17 Running total và LAG/LEAD: tính tổng dồn và so sánh với dòng trước?</b></summary>

- **Running total** (chạy tổng): `SUM(x) OVER (ORDER BY ...)` cộng dồn theo thứ tự — vd velocity tích lũy theo sprint.
- **`LAG(col, n)` / `LEAD(col, n)`**: lấy giá trị của dòng **trước/sau** trong cửa sổ → tính delta, tăng trưởng mà **không cần self-join**.
- Mặc định frame của `SUM ... ORDER BY` là "từ đầu partition đến dòng hiện tại" (running), khác với không có `ORDER BY` (tính cả partition).

```sql
-- So issue dong moi sprint + tong don + chenh lech so voi sprint truoc
SELECT sprint_id, COUNT(*) AS done,
  SUM(COUNT(*)) OVER (ORDER BY sprint_id) AS cong_don,
  COUNT(*) - LAG(COUNT(*)) OVER (ORDER BY sprint_id) AS chenh_lech
FROM issue WHERE status = 'DONE'
GROUP BY sprint_id;
```

</details>

<details>
<summary><b>34.18 CTE (WITH) là gì? Recursive CTE để duyệt cây subtask/epic ra sao?</b></summary>

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
<summary><b>34.19 Đọc EXPLAIN ANALYZE: phân biệt Seq Scan vs Index Scan, estimated vs actual?</b></summary>

- `EXPLAIN` cho **kế hoạch dự kiến** (cost ước lượng); `EXPLAIN (ANALYZE, BUFFERS)` **chạy thật** và cho thời gian + số dòng thực + lượng buffer đọc.
- **Seq Scan** (quét toàn bảng): ổn với bảng nhỏ, nhưng trên bảng lớn + có điều kiện lọc chọn lọc → dấu hiệu **thiếu index**. **Index Scan / Index Only Scan**: đang dùng index (thường tốt).
- So **estimated vs actual rows**: lệch lớn ⇒ **thống kê cũ**, chạy `ANALYZE` để planner ước lượng đúng.
- Quy trình tối ưu chuẩn (bật query log → EXPLAIN ANALYZE → thêm index/viết lại sargable) xem chi tiết ở [28.x](#28-tư-duy-và-giải-quyết-vấn-đề-backend).

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM issue WHERE project_id = 1 ORDER BY created_at DESC LIMIT 20;
-- Soi: co "Seq Scan on issue"? actual rows >> estimated? -> them index (project_id, created_at DESC)
```

</details>

<details>
<summary><b>34.20 SQL vs NoSQL: chọn dựa trên tiêu chí nào?</b></summary>

- **SQL (quan hệ)**: schema chặt, **JOIN** mạnh, **transaction ACID** đa bảng, truy vấn linh hoạt ad-hoc → hợp dữ liệu **quan hệ chặt, tính nhất quán cao** (đa số nghiệp vụ như Jira: project–issue–user–comment).
- **NoSQL** (document/key-value/wide-column): schema linh hoạt, scale ngang dễ, đọc/ghi theo **một aggregate** rất nhanh → hợp dữ liệu **phi quan hệ, throughput lớn, hình dạng đa dạng** (log, event, catalog).
- Đừng chọn theo "hot tech": phần lớn app khởi đầu **nên dùng Postgres** (có cả `jsonb` cho phần linh hoạt); chỉ thêm NoSQL khi có nhu cầu rõ (scale/đặc thù truy cập). Chi tiết MongoDB ở [Phần 30](#30-mongodb-và-mongoose-nosql).
- Câu trả lời "ăn điểm": nói về **mô hình truy cập (access pattern)** và **tính nhất quán cần có**, không nói chung chung "NoSQL nhanh hơn".

</details>

<details>
<summary><b>34.21 Một query danh sách issue rất chậm — bạn debug và tối ưu theo các bước nào?</b></summary>

- **Đo trước, đoán sau**: bật query log (Prisma `log:['query']` / TypeORM `maxQueryExecutionTime`) tìm đúng query chậm, rồi `EXPLAIN ANALYZE`.
- **Đọc plan**: Seq Scan trên bảng lớn? actual ≫ estimated (thống kê cũ)? node nào tốn thời gian nhất?
- **Sửa theo ưu tiên**: (1) thêm/chỉnh **index** composite theo đúng thứ tự `WHERE` rồi `ORDER BY`; (2) viết lại điều kiện **sargable** (bỏ hàm bọc cột); (3) **giảm dữ liệu** trả về (chỉ `SELECT` cột cần, phân trang **keyset**); (4) xử lý **N+1** bằng JOIN/`include` (xem 19.7); (5) cuối cùng mới tính cache/denormalize.
- Đo lại sau mỗi thay đổi để xác nhận đúng nguyên nhân.

```sql
-- Vd: query loc + sort -> index phu hop
CREATE INDEX idx_issue_proj_created ON issue (project_id, created_at DESC)
  WHERE deleted_at IS NULL;   -- partial: bo qua issue da xoa mem
```

</details>

<details>
<summary><b>34.22 Vài "gotcha" SQL hay bị hỏi để bẫy?</b></summary>

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
