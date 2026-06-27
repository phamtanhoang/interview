# 🔗 Bộ câu hỏi phỏng vấn Fullstack (FE ↔ BE & Ways of Working)

> Tài liệu bổ trợ cho vai trò **Fullstack Developer** (React/Next.js + NestJS/Node.js) — tập trung vào phần **xuyên suốt** mà file frontend/backend riêng lẻ chưa bao quát: tích hợp **end-to-end**, **analytics/telemetry**, và **cách làm việc** (MVP-first, AI-native).
> Mỗi đáp án nằm trong khối **collapse** (`<details>`) — bấm để xem lời giải + ví dụ.

## 📚 Mục lục

1. [Fullstack End-to-end (FE và BE)](#1-fullstack-end-to-end-fe-và-be)
2. [Domain: Analytics, Telemetry và Feature Flags](#2-domain-analytics-telemetry-và-feature-flags)
3. [Cách làm việc và Tư duy sản phẩm](#3-cách-làm-việc-và-tư-duy-sản-phẩm)

---


## 1. Fullstack End-to-end (FE và BE)

### 1.1. Hãy mô tả quy trình thiết kế một feature từ đầu đến cuối (UI → API → DB). Bạn bắt đầu từ đâu và các bước theo thứ tự nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Tiếp cận "contract-first": chốt hợp đồng dữ liệu trước, rồi làm song song FE/BE.

- **Hiểu nghiệp vụ + domain model**: entity nào, trạng thái và quy tắc ra sao.
- **Thiết kế DB schema trước** (khó migrate nhất): bảng, quan hệ, index, ràng buộc (unique, FK).
- **Định nghĩa API contract**: endpoint, method, request/response/error shape, pagination — viết bằng OpenAPI hoặc Zod dùng chung.
- **Triển khai song song**: BE theo contract (mock trước), FE dùng mock/codegen để không bị block.
- **Tích hợp + test e2e**: happy path + edge case (lỗi mạng, validate fail, race).
- Nguyên tắc: mọi business rule nằm ở BE; tách DTO khỏi entity (không map 1-1 DB lên response).

```ts
// Contract dùng chung (shared/contracts/comment.ts)
export const CreateCommentDto = z.object({
  postId: z.string().uuid(),
  body: z.string().min(1).max(2000),
  parentId: z.string().uuid().optional(),
});
export type CreateCommentDto = z.infer<typeof CreateCommentDto>;
```

</details>

---

### 1.2. Có những cách nào để chia sẻ kiểu dữ liệu (type) giữa FE và BE? So sánh OpenAPI + codegen, Zod schema dùng chung, và tRPC.

<details>
<summary>👉 Xem câu trả lời</summary>

- **OpenAPI + codegen**: source = file spec; chuẩn ngôn ngữ-độc lập, đa client; nhưng phải maintain spec, không tự validate runtime. Dùng cho API công khai, đa client/ngôn ngữ.
- **Zod dùng chung**: source = schema trong package chung; vừa type tĩnh (`z.infer`) vừa validate runtime, một nguồn sự thật; nhưng buộc cả hai dùng TS + monorepo. Dùng cho fullstack TS.
- **tRPC**: source = type router BE; type-safe đầu cuối không cần codegen; nhưng buộc TS hai đầu, không có HTTP contract rõ cho bên thứ ba. Dùng cho monorepo TS nội bộ.
- Lưu ý: NestJS không native với tRPC → với **Next.js + NestJS** nên chọn **OpenAPI + codegen** (`@nestjs/swagger`) hoặc **Zod dùng chung**.

```ts
// tRPC: BE định nghĩa router, FE tự suy type, không cần codegen
const post = await trpc.getPost.query({ id }); // post có type đầy đủ
```

</details>

---

### 1.3. Trình bày luồng authentication end-to-end khi Next.js gọi NestJS với JWT lưu trong httpOnly cookie. Tại sao httpOnly cookie an toàn hơn localStorage?

<details>
<summary>👉 Xem câu trả lời</summary>

- Login → `POST /auth/login` tới NestJS; BE tạo **access token** (ngắn, ~15 phút) + **refresh token** (dài, ~7 ngày).
- BE set cả hai vào `Set-Cookie` với `HttpOnly`, `Secure`, `SameSite`; request sau trình duyệt tự đính kèm, guard verify JWT.
- Access hết hạn → `401` → FE gọi `/auth/refresh` lấy token mới.
- **httpOnly an toàn hơn localStorage**: localStorage đọc được bằng JS → XSS lấy token ngay; httpOnly cookie JS không đọc được → chặn XSS đánh cắp token.
- Đánh đổi: cookie mở ra nguy cơ **CSRF** (cần biện pháp riêng), nhưng CSRF dễ phòng hơn. Cookie còn giúp SSR đọc phiên khi render server-side.

```ts
res.cookie('access_token', accessToken, {
  httpOnly: true, secure: true, sameSite: 'lax', maxAge: 15 * 60 * 1000, path: '/',
});
```

</details>

---

### 1.4. Khi dùng JWT trong cookie, làm sao chống CSRF? Giải thích vai trò của SameSite và khi nào nó vẫn chưa đủ.

<details>
<summary>👉 Xem câu trả lời</summary>

CSRF xảy ra vì trình duyệt tự đính kèm cookie kể cả với request từ trang độc hại.

- **Cờ `SameSite`** (lớp đầu): `Strict` (không gửi cross-site), `Lax` (gửi với top-level GET, không với POST cross-site — mặc định tốt), `None` (luôn gửi, bắt buộc kèm `Secure`).
- **CSRF token** (double-submit / synchronizer): cần khi `SameSite=None`.
- **SameSite chưa đủ khi**: FE/BE khác site buộc dùng `None` → mất tác dụng → cần CSRF token hoặc kiểm tra `Origin`/`Referer`; subdomain cùng site vẫn same-site (nên đặt chung registrable domain); trình duyệt cũ không hỗ trợ.
- Thực tế: cùng site → `Lax` + kiểm `Origin`; khác site → `None; Secure` + CSRF token.

```ts
// Double-submit: header phải khớp cookie csrf (cookie này không-httpOnly)
if (req.cookies['csrf_token'] !== req.headers['x-csrf-token'])
  throw new ForbiddenException('CSRF mismatch');
```

</details>

---

### 1.5. Thiết kế cơ chế refresh token end-to-end an toàn. Refresh token rotation là gì và tại sao cần nó?

<details>
<summary>👉 Xem câu trả lời</summary>

- Mô hình hai token: access ngắn (hạn chế thiệt hại nếu lộ), refresh dài (đổi access mới không cần đăng nhập lại).
- **Rotation**: mỗi lần dùng refresh → cấp refresh mới + vô hiệu cái cũ.
- **Tại sao cần**: nếu refresh bị trộm, kẻ tấn công và user "tranh nhau" dùng. BE thấy token đã dùng lại được dùng tiếp → phát hiện **reuse** → thu hồi cả chuỗi (family) → buộc đăng nhập lại.
- Best practice: lưu refresh dưới dạng **hash** + `familyId` để revoke chuỗi; giới hạn `path` cookie (vd `/auth/refresh`); FE dùng **single-flight** khi `401` để tránh nhiều tab refresh race.

```ts
if (record.usedAt) {                         // đã dùng mà lại đến -> nghi bị trộm
  await this.tokenRepo.revokeFamily(record.familyId);
  throw new UnauthorizedException('Token reuse detected');
}
await this.tokenRepo.markUsed(record.id);    // rồi cấp refresh + access mới
```

</details>

---

### 1.6. Hãy thiết kế luồng upload file end-to-end dùng presigned URL / S3. Tại sao nên upload trực tiếp lên S3 thay vì qua server?

<details>
<summary>👉 Xem câu trả lời</summary>

- FE xin BE: muốn upload file tên X, type Y, size Z.
- BE **validate** (loại/kích thước/quyền) → tạo **presigned PUT URL** hạn ngắn → trả FE.
- FE `PUT` **thẳng lên S3** bằng URL đó, theo dõi progress.
- Xong, FE báo BE object key → BE lưu metadata, có thể xử lý nền (thumbnail, quét virus).
- **Tại sao upload thẳng S3**: không chiếm băng thông/bộ nhớ BE, BE không thành bottleneck; rẻ và nhanh hơn; BE vẫn kiểm soát qua presigned URL.
- Bảo mật: FE có thể nói dối `contentType` → vẫn phải xác minh file thật phía server/qua S3 event (magic bytes, quét virus).

```ts
const command = new PutObjectCommand({ Bucket, Key: key, ContentType: dto.contentType });
const url = await getSignedUrl(this.s3, command, { expiresIn: 300 }); // 5 phút
```

</details>

---

### 1.7. Optimistic UI là gì? Hãy minh họa cách làm optimistic update kèm rollback khi server trả lỗi, dùng React Query.

<details>
<summary>👉 Xem câu trả lời</summary>

- **Optimistic UI**: cập nhật giao diện ngay theo kết quả kỳ vọng trước khi server xác nhận; lỗi thì **rollback**.
- Phù hợp hành động khả năng thành công cao, ít rủi ro (like, toggle, comment); KHÔNG dùng cho thao tác cần chính xác (thanh toán).
- React Query pattern: `onMutate` (cancelQueries + snapshot + setQueryData) → `onError` (khôi phục snapshot) → `onSettled` (invalidateQueries).
- Lỗi hay gặp: quên `cancelQueries` (refetch ghi đè), quên snapshot (không rollback được), quên `invalidateQueries` (UI lệch server).

```tsx
onMutate: async (postId) => {
  await queryClient.cancelQueries({ queryKey: ['post', postId] });
  const previous = queryClient.getQueryData(['post', postId]);
  queryClient.setQueryData(['post', postId], (o: any) => ({ ...o, liked: true, likeCount: o.likeCount + 1 }));
  return { previous };
},
onError: (_e, postId, ctx) => ctx?.previous && queryClient.setQueryData(['post', postId], ctx.previous),
onSettled: (_d, _e, postId) => queryClient.invalidateQueries({ queryKey: ['post', postId] }),
```

</details>

---

### 1.8. Tại sao cần một error shape thống nhất giữa FE và BE? Hãy đề xuất một cấu trúc lỗi chuẩn và cách FE tiêu thụ nó.

<details>
<summary>👉 Xem câu trả lời</summary>

- Mỗi endpoint trả lỗi khác nhau → FE xử lý rời rạc, dễ sót, khó hiển thị nhất quán, khó map field vào form. Shape thống nhất → FE có **một chỗ duy nhất** parse lỗi.
- Cấu trúc đề xuất (gần RFC 7807): `code`, `message`, `statusCode`, `traceId?`, `fieldErrors?`.
- BE: dùng exception filter chuẩn hóa mọi lỗi về cùng shape.
- Nguyên tắc: FE phân nhánh theo **`code` ổn định** (không dựa `message` vì đổi/đa ngôn ngữ); đính `traceId` để tra log; tách `fieldErrors` để gắn vào form.

```ts
interface ApiError {
  code: string;        // 'VALIDATION_FAILED'
  message: string;
  statusCode: number;
  traceId?: string;
  fieldErrors?: Record<string, string[]>;
}
```

</details>

---

### 1.9. So sánh offset pagination và cursor pagination. Tại sao cursor tốt hơn cho dữ liệu thay đổi liên tục, và FE-BE cần đồng bộ những gì?

<details>
<summary>👉 Xem câu trả lời</summary>

| Tiêu chí | Offset | Cursor (keyset) |
|----------|--------|-----------------|
| Cách làm | `OFFSET 40 LIMIT 20` | `WHERE created_at < $cursor` |
| Nhảy trang bất kỳ | Được | Không (chỉ next/prev) |
| Offset lớn | Chậm dần (quét và bỏ N dòng) | Ổn định (dùng index) |
| Dữ liệu thay đổi | Trùng/sót item khi có insert/delete | Ổn định |

- **Cursor tốt hơn khi dữ liệu động**: offset bị duplicate (item bị đẩy xuống xuất hiện lại trang sau) hoặc nhảy cóc mất item; cursor neo vào giá trị thực một dòng (`created_at` + `id` để phá hòa) nên không lệch.
- **FE-BE cần đồng bộ**: response shape `{ data, nextCursor, hasNext }`; cursor là **opaque** (FE không tự suy diễn); thứ tự sắp xếp nhất quán giữa các lần gọi. FE khớp tốt với `useInfiniteQuery`.

```ts
take: limit + 1,                                   // lấy dư 1 để biết có trang sau
...(cursor && { cursor: { id: cursor }, skip: 1 }),
orderBy: [{ createdAt: 'desc' }, { id: 'desc' }],
```

</details>

---

### 1.10. Realtime end-to-end: khi nào dùng WebSocket, khi nào dùng SSE? Mô tả luồng và những điểm cần xử lý ở cả FE lẫn BE.

<details>
<summary>👉 Xem câu trả lời</summary>

| Tiêu chí | WebSocket | SSE |
|----------|-----------|-----|
| Chiều | Hai chiều | Một chiều (server→client) |
| Giao thức | `ws/wss` | HTTP (`text/event-stream`) |
| Reconnect | Tự code/socket.io | Trình duyệt tự lo |
| Phù hợp | Chat, game, collab | Notification, feed, progress |

- Chọn: **client gửi liên tục → WebSocket; chỉ server đẩy → SSE** (đơn giản, tận dụng auto-reconnect).
- Cần xử lý hai phía: **Auth** (WS truyền token qua cookie/query/handshake; SSE dùng cookie bình thường); **reconnect & resume** (SSE có `Last-Event-ID`; WS tự thiết kế sequence/ack); **scale** cần pub/sub (Redis adapter) để broadcast đa instance; **cleanup FE** hủy subscription khi unmount, throttle khi dồn dập.

```ts
const es = new EventSource('/api/notifications', { withCredentials: true });
es.onmessage = (e) => addNotification(JSON.parse(e.data));
```

</details>

---

### 1.11. Tại sao không được đặt business logic quan trọng ở phía client? Cho ví dụ một lỗi thực tế và cách phòng tránh khi đồng bộ validation FE-BE.

<details>
<summary>👉 Xem câu trả lời</summary>

- Mọi thứ chạy ở client (giá, quyền, điều kiện nghiệp vụ) đều có thể bị **sửa/bỏ qua/giả mạo**: user mở DevTools, sửa request, gọi API thẳng bằng `curl`/Postman.
- FE validation chỉ để **cải thiện UX**; BE là **nguồn sự thật bắt buộc**.
- Phòng tránh: luôn enforce ở BE (kiểm quyền trong guard, tính giá từ DB); đồng bộ luật bằng schema dùng chung (Zod) nhưng BE vẫn validate lại độc lập; không gửi dữ liệu nhạy cảm xuống rồi nhận lại — gửi id/mã, BE tự tra.
- Tư duy: "FE validation là cánh cửa lịch sự, BE validation là ổ khóa".

```ts
// SAI: tính giảm giá ở client rồi gửi giá đã giảm lên server
const finalPrice = price * 0.9; // user sửa thành price * 0.0 -> BE tin -> mua 0đ
// ĐÚNG: BE tính giá từ DB + enforce quyền
const amount = computePrice(items, await couponRepo.findValid(dto.couponCode));
```

</details>

---

### 1.12. API versioning: khi nào và làm thế nào để version một API mà không làm vỡ client FE đang chạy? So sánh các chiến lược.

<details>
<summary>👉 Xem câu trả lời</summary>

- Cần version khi phải làm **breaking change** (xóa/đổi tên field, đổi kiểu/ngữ nghĩa/shape) mà còn client cũ đang chạy. Mục tiêu: client cũ vẫn chạy, client mới dùng phiên bản mới.
- **URI** (`/v1/posts`): rõ ràng, dễ cache/route/debug; nhưng "ô nhiễm" URL.
- **Header** (`Accept: ...vnd.app.v2+json`): URL sạch, đúng REST; nhưng khó test tay, dễ sót.
- **Query param** (`?version=2`): đơn giản; nhưng dễ quên, lẫn filter.
- Nguyên tắc: ưu tiên thay đổi non-breaking (chỉ thêm field); chỉ version khi thực sự breaking; có chính sách deprecate (header `Deprecation`/`Sunset`, đo traffic); phối hợp FE để chủ động migrate. Web nội bộ → **URI versioning** phổ biến nhất.

```ts
app.enableVersioning({ type: VersioningType.URI, defaultVersion: '1' });
// @Version('1') findAllV1() ...   @Version('2') findAllV2() ...
```

</details>

## 2. Domain: Analytics, Telemetry và Feature Flags

### 2.1. Vì sao thu thập event analytics ở phía server (server-side) thường đáng tin hơn phía client (client-side)? Khi nào vẫn bắt buộc phải dùng client-side?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Client-side dễ mất/sai lệch:** bị ad-blocker chặn, user đóng tab trước khi gửi, mạng chập chờn, dễ bị spoof.
- **Server-side đáng tin hơn:** phát từ backend khi xử lý request thật -> khó chặn, khó giả mạo, nhất quán với DB (cùng nguồn sự thật).
- **Vẫn cần client-side** cho tương tác chỉ có ở UI: scroll depth, thời gian xem component, A/B test render, rage-click.
- Team trưởng thành thường dùng **hybrid:** client cho UX/engagement, server cho "money/business event" quan trọng.

```ts
@Post('orders')
async createOrder(@Body() dto, @Req() req) {
  const order = await this.orderService.create(dto);
  // Phát SAU khi DB commit => phản ánh sự thật, không gửi khi fail
  this.analytics.track({ name: 'order_created', userId: req.user.id, props: { orderId: order.id, amount: order.amount } });
  return order;
}
```

</details>

---

### 2.2. Thiết kế một event schema tốt cần những gì? Cách đặt tên, property và đặc biệt là versioning khi schema thay đổi?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Naming:** thống nhất convention `object_action` thì quá khứ (`order_created`); dùng `snake_case`; tránh tên mơ hồ (`click`, `event1`).
- **Property:** kiểu cố định (`amount` luôn number); tách thuộc tính event (`order_id`) khỏi context chung (user, device); tránh nhồi PII.
- **Versioning:** thêm property optional -> tương thích ngược, không cần version mới. Đổi kiểu/ý nghĩa/bỏ field -> thêm `version` hoặc tạo event mới (`checkout_completed_v2`).
- **Bẫy:** không bao giờ tái dùng tên cũ với ý nghĩa mới — dashboard/pipeline cũ sẽ hiểu sai.

```ts
const OrderCreatedV1 = z.object({
  name: z.literal('order_created'),
  version: z.literal(1),
  props: z.object({ orderId: z.string(), amount: z.number().nonnegative() }),
});
```

Ví dụ: đổi `amount` từ "đồng" sang "cent" (kiểu vẫn number nhưng ý nghĩa đổi) -> bắt buộc bump version vì dữ liệu cũ/mới không trộn được khi tính tổng.

</details>

---

### 2.3. "Idempotent event ingestion" là gì? Trong mô hình at-least-once delivery, làm sao chống event bị đếm trùng (dedupe)?

<details>
<summary>👉 Xem câu trả lời</summary>

- **At-least-once** nghĩa là một event có thể đến >= 2 lần (client retry vì timeout, queue redeliver khi consumer crash trước ack) -> số liệu phồng.
- **Idempotent ingestion:** xử lý cùng event nhiều lần cho kết quả như xử lý một lần.
- Cách làm: mỗi event mang `event_id` duy nhất (sinh tại điểm phát để retry giữ nguyên) -> lưu id đã xử lý vào store unique (DB index / Redis SET TTL) -> nếu trùng thì drop.
- **Bẫy:** dedupe cần cửa sổ TTL hợp lý — nhớ mãi thì store phình, quá ngắn thì bản trùng đến muộn vẫn lọt. Chọn TTL > thời gian retry tối đa.

```ts
async ingest(event) {
  try {
    await this.repo.insert({ eventId: event.eventId, payload: event }); // unique index
  } catch (e) {
    if (this.isUniqueViolation(e)) return; // duplicate -> drop, không đếm lại
    throw e;
  }
  await this.process(event);
}
```

Throughput cao: `redis.set(\`evt:${id}\`, '1', 'NX', 'EX', 86400)` — trả null nếu đã thấy.

</details>

---

### 2.4. Feature flag là gì? Giải thích rollout theo phần trăm (%), kill switch và targeting. Triển khai an toàn ở cả frontend (React/Next.js) lẫn backend (NestJS) ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Feature flag:** bật/tắt/điều khiển tính năng tại runtime không cần deploy lại -> tách "deploy code" khỏi "release tính năng".
- **Kill switch:** tắt cứng tính năng lỗi ngay, không cần rollback deploy.
- **Rollout %:** mở dần 5% -> 25% -> 100% (canary); phải **sticky** (hash `userId` để cùng user luôn cùng kết quả) nếu không UI nhấp nháy.
- **Targeting:** bật theo thuộc tính (quốc gia, gói trả phí, beta tester).
- **Backend (NestJS):** đánh giá ở server là nguồn sự thật cho tính năng nhạy cảm; bọc bằng guard/service đọc flag.
- **Frontend (Next.js):** rủi ro **flicker/hydration mismatch** -> đánh giá flag ở server (Server Component/middleware) rồi truyền xuống.
- **Vận hành:** flag phải có default an toàn (service down -> fallback OFF) và được dọn dẹp sau khi ổn định.

```ts
function isEnabled(flag, userId, percentage) {
  const h = createHash('sha256').update(`${flag}:${userId}`).digest();
  return (h.readUInt32BE(0) % 100) < percentage; // bucket 0..99, sticky
}
```

</details>

---

### 2.5. Cách thiết kế và đo lường một A/B test cơ bản? Giải thích metric, sample size và statistical significance (ý nghĩa thống kê) một cách thực tế.

<details>
<summary>👉 Xem câu trả lời</summary>

- **A/B test:** chia ngẫu nhiên user thành Control (A) và Variant (B), so sánh một metric mục tiêu xem thay đổi thật sự tốt hơn hay may rủi.
- **Primary metric:** chọn rõ trước khi chạy (vd conversion rate); tránh đào bới nhiều metric rồi chọn cái đẹp (p-hacking); kèm guardrail metric (tỉ lệ lỗi, load time).
- **Phân nhóm sticky:** hash `userId` để một user luôn ở một nhánh.
- **Sample size & thời gian:** đủ lớn để phát hiện chênh lệch; dừng sớm (peeking) -> kết luận sai; chạy đủ bội số tuần tránh hiệu ứng ngày.
- **Significance:** p-value < 0.05 = xác suất chênh lệch chỉ do ngẫu nhiên < 5%. Khác với **effect size** (lớn cỡ nào) — chênh 0.1% có thể significant với mẫu khổng lồ nhưng vô nghĩa kinh doanh.

| Tình huống | Vấn đề |
| --- | --- |
| Dừng khi B vừa vượt A | Peeking -> dương tính giả |
| Đổi metric giữa chừng | Thiên lệch, không đáng tin |
| Định trước metric + sample size, chạy đủ | Đúng chuẩn |

Việc tính p-value nên dùng thư viện/công cụ (scipy, nền tảng experimentation), không "ước lượng bằng mắt".

</details>

---

### 2.6. So sánh batch và stream processing trong một data pipeline analytics nhẹ. Vai trò của queue (Kafka/SQS/RabbitMQ) ở đâu? Khi nào chọn cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Pipeline:** producer -> queue/buffer -> processor -> warehouse -> dashboard.
- **Queue (Kafka/SQS/RabbitMQ)** decouple producer/processor: app chỉ enqueue rồi trả response nhanh; hấp thụ tải đỉnh (back-pressure); processor chết thì event vẫn trong queue (kết hợp at-least-once + dedupe).
- **Batch:** gom theo lô, latency cao, đơn giản, rẻ -> báo cáo doanh thu cuối ngày, tổng hợp lịch sử.
- **Stream:** xử lý từng event gần real-time, latency thấp, phức tạp hơn, đắt hơn -> cảnh báo gian lận, live dashboard.
- **Chọn:** chỉ cần "hôm qua bao nhiêu order" -> batch. Cần phản ứng tức thì -> stream. Nhiều hệ thống dùng cả hai (lambda architecture).
- Dự án nhỏ chưa cần Kafka ngay; BullMQ (Redis)/SQS đủ decouple + retry, nâng Kafka khi cần throughput lớn và replay log.

```ts
async track(event) {
  await this.queue.add('analytics_event', event, {
    attempts: 5, backoff: { type: 'exponential', delay: 1000 },
  });
}
```

</details>

---

### 2.7. Làm sao tránh lộ PII trong log/analytics và tôn trọng consent theo GDPR? Vì sao không nên log dữ liệu cá nhân, và xử lý "right to be forgotten" thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- **PII** = dữ liệu nhận dạng cá nhân: email, phone, địa chỉ, IP, payment, đôi khi `userId` thật. Log/analytics là nơi hay rò rỉ nhất.
- **Vì sao nguy hiểm:** log lưu lâu, sao chép sang nhiều hệ thống, nhiều người truy cập -> bề mặt rò rỉ rộng. GDPR yêu cầu consent, giới hạn mục đích, data minimization, quyền xoá.
- **Data minimization:** không log PII từ đầu, redact/mask ở tầng logger.
- **Pseudonymisation:** dùng id hash thay email thật, vẫn join được.
- **Consent gating:** chỉ track khi user đồng ý; frontend không load script tracking trước consent.
- **Right to be forgotten:** gắn dữ liệu với `userId` để tìm/xoá theo cascade, không rải PII khắp free-text log.
- **Data retention:** đặt TTL, không giữ vô thời hạn.

```ts
const REDACT = ['password', 'email', 'phone', 'token', 'authorization', 'card'];
function sanitize(obj) {
  for (const k of Object.keys(obj))
    if (REDACT.includes(k.toLowerCase())) obj[k] = '[REDACTED]';
  return obj;
}
```

</details>

---

### 2.8. Trong bối cảnh game tools/publishing, hãy thiết kế telemetry để đo hành vi người chơi và dựng một funnel (ví dụ onboarding/level progression). Cần lưu ý gì đặc thù?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Telemetry người chơi:** event mô tả hành trình chơi (install, open, tutorial, level, purchase, churn) để dựng **funnel** thấy người chơi rơi rớt ở đâu.
- **Funnel:** chuỗi bước có thứ tự, mỗi bước đo % người đi tiếp -> nhìn ra chỗ tụt mạnh để sửa.
- **Offline & buffering:** game mobile hay mất mạng -> buffer event local, gửi batch khi có mạng; tách `occurredAt` (thời điểm thật) khỏi `receivedAt`.
- **Idempotency:** gửi lại sau offline dễ trùng -> mỗi event có `event_id` để dedupe.
- **Server-side cho money/anti-cheat:** `first_purchase`, `currency_spent` xác nhận ở server, không tin client.
- **Session & cohort:** gắn `session_id`; phân tích theo cohort để đo retention D1/D7/D30.
- **Privacy:** game có người chơi nhỏ tuổi -> COPPA/GDPR chặt hơn, tránh PII, tôn trọng consent.

```text
install -> first_open -> tutorial_completed -> level_1_completed -> first_purchase
  100%       82%              55%                    40%                 6%
```

</details>

## 3. Cách làm việc và Tư duy sản phẩm

### 3.1. "MVP-first" và "bias toward shipping" nghĩa là gì? Khi nhận một feature mới, bạn ưu tiên giao một bản chạy được nhanh như thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- MVP-first: chia feature thành phiên bản nhỏ nhất tạo giá trị thật, ship trước rồi lặp lại để hoàn thiện.
- Bias toward shipping: thiên hướng đưa code ra production sớm — code chỉ tạo giá trị khi dùng được, feedback thật quý hơn phỏng đoán.
- Quy trình: (1) xác định luồng giá trị cốt lõi = MVP; (2) cắt phần chưa cần cho lần đầu; (3) ship dọc (vertical slice) DB -> API -> UI cho một luồng.
- MVP-first KHÔNG phải làm ẩu: bản MVP vẫn phải đúng, an toàn (auth, validation), không phá luồng cũ. "Nhỏ" là về phạm vi, không phải chất lượng.

```ts
// MVP: lọc theo MOT assignee, đủ giao giá trị ngay
@Get('tasks')
findTasks(@Query('assigneeId') assigneeId?: string) {
  return this.prisma.task.findMany({ where: assigneeId ? { assigneeId } : {} });
}
// Sau: nhiều assignee, lưu filter, phân trang (KHONG làm lần đầu)
```

</details>

---

### 3.2. "Đủ tốt để ship" (good enough to ship) là tới đâu? Bạn cân bằng tốc độ và chất lượng thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Không phải con số cố định; phụ thuộc rủi ro và độ đảo ngược của thay đổi.
- Dễ đảo ngược + rủi ro thấp (đổi label, thêm filter UI): ngưỡng thấp, ship nhanh, sửa sau.
- Khó đảo ngược + rủi ro cao (migration, đổi API public, thanh toán): ngưỡng cao, test/review kỹ, cân nhắc feature flag.
- Cách cân bằng: tách "phải có trước ship" (đúng, an toàn, không regression) khỏi "có thì tốt" (đánh bóng, tối ưu sớm). Đang trau chuốt phần "có thì tốt" mà luồng chính đã ổn = tín hiệu nên ship.
- Khi cố ý ship còn nợ kỹ thuật: ghi rõ (`// TODO`, ticket, note PR) để team thấy đánh đổi. Tốc độ có chủ đích khác hẳn ẩu.

```
[x] Happy path + edge case khả năng cao   [x] Validation + xử lý lỗi an toàn
[x] Auth/authz đúng   [x] Test logic quan trọng   [x] Không regression
[ ] Tối ưu perf chưa có bằng chứng (YAGNI)   [ ] Edge case hiếm  <- để sau
```

</details>

---

### 3.3. Over-engineering là gì? Áp dụng YAGNI và KISS trong code thực tế ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

- Over-engineering: xây trừu tượng/linh hoạt cho nhu cầu chưa tồn tại — trả phí phức tạp ngay, lợi ích chỉ là giả định.
- YAGNI: đừng thêm khả năng cho tới khi thật sự cần. KISS: chọn giải pháp đơn giản nhất giải quyết vấn đề hiện tại.
- "Rule of three": chấp nhận lặp 1-2 lần; tới lần 3 khi pattern đã rõ mới tách abstraction. Trừu tượng đúng dựa trên pattern đã thấy, không phải tương lai tưởng tượng.
- Bẫy: nhầm "đơn giản" với "ngắn". One-liner khó đọc cũng vi phạm KISS. Đơn giản = dễ hiểu, dễ sửa.

```ts
// OVER: factory + interface cho 1 provider email duy nhất (phần còn lại là phỏng đoán)
// KISS: làm thẳng, refactor khi THỰC SỰ cần provider thứ hai
@Injectable() class EmailService {
  async send(to: string, body: string) { await this.sendgrid.send({ to, html: body }); }
}
// FE: state mở/đóng 1 modal -> useState tại component, không đẩy lên Redux
```

</details>

---

### 3.4. Khi gặp một vấn đề kỹ thuật lạ, quy trình tự tháo gỡ (self-sufficient) của bạn ra sao trước khi hỏi người khác?

<details>
<summary>👉 Xem câu trả lời</summary>

- Self-sufficient không phải không hỏi, mà tự khai thác hết các kênh trong khung thời gian hợp lý rồi mới hỏi với câu hỏi đã nén đủ ngữ cảnh.
- Quy trình: (1) đọc kỹ error + stack trace; (2) tái hiện tối thiểu để cô lập; (3) đọc docs/source chính thức (kể cả `node_modules`); (4) dùng AI tạo giả thuyết nhưng verify; (5) thử nghiệm đổi MỘT biến mỗi lần; (6) tìm trong codebase nội bộ pattern tương tự.
- Time-box: ví dụ 30-60 phút cho lỗi non-blocking; vượt thì escalate. Nếu CHẶN cả team thì escalate gần như ngay.
- Khi hỏi: kèm "tôi đã thử X, Y, Z và kết quả là...", không hỏi cộc lốc "lỗi này fix sao?".

```bash
node -e "console.log(require('some-lib/package.json').version)"  # đúng version chưa?
git log --oneline -- path/to/file   # ai từng đụng file này
git bisect start                    # tìm commit gây regression
```

</details>

---

### 3.5. Bạn ước lượng (estimate) và chia nhỏ một feature lớn thành các task như thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Mỗi task xong trong ~0.5-1 ngày, có định nghĩa "xong" rõ, lý tưởng là ship/merge độc lập (vertical slice).
- Phân rã theo luồng giá trị, KHÔNG theo tầng kỹ thuật (tránh "task DB / task API / task UI" vì không merge được tới cuối).
- Đánh dấu phụ thuộc + ẩn số; spike time-box cho phần mơ hồ trước khi ước lượng; chỉ cộng buffer cho ẩn số.
- Nói rõ mức tự tin thay vì con số giả vờ chắc; cập nhật ước lượng NGAY khi phát hiện sai. Ước lượng là dự báo có sai số, không phải lời hứa.

```
Feature: Comment trên task
├─ [Slice 1] Tạo + list (schema ~0.5d, API+test ~0.5d, UI ~0.5d)
├─ [Slice 2] Sửa/Xóa (~1d)
├─ [Slice 3] Mention @user + notify (~1.5d, có ẩn số -> spike trước)
└─ [Slice 4] Đánh bóng loading/empty/error (~0.5d)
```

</details>

---

### 3.6. Bạn cộng tác với Tech Manager và đồng đội ra sao? Khi nào thì cần communicate sớm và hỏi đúng câu hỏi?

<details>
<summary>👉 Xem câu trả lời</summary>

- Communicate sớm rẻ hơn để bất ngờ nổ ra muộn; "sẽ trễ 2 ngày" nói hôm nay quý hơn nói đúng hạn.
- Liên lạc ngay (không đợi standup) khi: sẽ trễ/scope phình; gặp blocker quá time-box; requirement mơ hồ/mâu thuẫn (hỏi TRƯỚC khi code); quyết định của mình ảnh hưởng người khác (đổi API contract/schema chung).
- Hỏi đúng: biến câu hỏi mở thành câu hỏi đóng kèm phương án + đánh đổi + đề xuất; người được hỏi chỉ cần xác nhận/chỉnh.
- Với đồng đội: PR description rõ (mục tiêu, cách tiếp cận, phần cần chú ý), tôn trọng quy ước, sync khi chạm phần người khác.

```
KÉM:  "Em nên làm search thế nào?"
TỐT:  "Search em phân vân: (A) SQL ilike — đơn giản, chậm khi data lớn;
       (B) full-text search — mạnh hơn, tốn hạ tầng. Data ~vài nghìn record
       nên em nghiêng (A) lần này, đúng hướng không ạ?"
```

</details>

---

### 3.7. Kể về một lần bạn nhận feedback khó nghe về code/cách làm của mình. Bạn xử lý ra sao? Và bạn đưa feedback cho người khác thế nào?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Câu hành vi, trả lời theo STAR (thay bằng câu chuyện thật):

- Situation: Mở PR cho một module NestJS; reviewer senior comment thẳng rằng cách xử lý transaction có thể gây race condition và nhét logic vào controller là sai tầng.
- Task: Hơi tự ái, nhưng nhiệm vụ thật là ra code đúng, không phải bảo vệ cái tôi.
- Action: tách cảm xúc khỏi nội dung (feedback nói về code); hỏi lại để hiểu thay vì phản bác ("kịch bản nào dẫn tới race condition?"); refactor logic xuống service, bọc transaction Prisma, thêm test; cảm ơn reviewer công khai.
- Result: PR merge, tránh được bug khó tái hiện trên prod, nhớ pattern cho lần sau.

Khi đưa feedback cho người khác: phê bình code không phê bình người; phân biệt "phải sửa" (bug) với "đề xuất" (đánh dấu nit); hỏi thay vì ra lệnh khi chưa chắc; khen phần làm tốt. Code review là kênh nâng chất lượng và học hai chiều, không phải đấu trường thắng-thua.

</details>

---

### 3.8. "Làm việc AI-native nhưng có phán đoán" nghĩa là gì? Khi nào bạn tin output của AI và khi nào bắt buộc verify?

<details>
<summary>👉 Xem câu trả lời</summary>

- AI-native: mặc định dùng AI tăng tốc (scaffold, test, giải thích error, refactor, khám phá API lạ).
- Có phán đoán: tôi vẫn chịu trách nhiệm cuối cho code — AI không phải cái cớ cho bug. Xem AI như junior rất nhanh nhưng đôi khi hallucinate.
- Dấu hiệu bắt buộc verify: AI gọi hàm/tham số lạ (check docs ngay); code "trông đúng" mà không giải thích được vì sao; output liên quan security, dữ liệu thật, thứ khó đảo ngược.
- Ranh giới: để AI tăng tốc phần cơ học và khám phá, KHÔNG outsource phán đoán kỹ thuật và trách nhiệm.

| Trường hợp | Tin cậy | Hành động |
|---|---|---|
| Boilerplate, đổi cú pháp | Cao | Đọc lướt, để test/compiler bắt |
| Logic nghiệp vụ | Trung bình | Đọc kỹ, tự viết test |
| API thư viện AI "nhớ" | Thấp | Đối chiếu docs (hay bịa tên hàm) |
| Bảo mật/auth/tiền/quyền, migration phá dữ liệu | Rất thấp | Tự suy luận + review người + test kỹ |

</details>

---

### 3.9. "Own một feature end-to-end" nghĩa là gì? Trách nhiệm của bạn trải dài tới đâu?

<details>
<summary>👉 Xem câu trả lời</summary>

- Chịu trách nhiệm feature từ lúc hiểu yêu cầu tới khi chạy ổn trên production và sau đó — không "viết xong rồi quăng qua tường".
- Vòng đời: làm rõ yêu cầu -> thiết kế (cân nhắc đánh đổi, sync phần chung) -> triển khai xuyên tầng DB/BE/FE -> chất lượng (test, edge case, loading/empty/error) -> đưa lên prod (PR, review, deploy) -> theo dõi log/metrics sau ship -> đóng vòng (docs, để lại đủ thông tin bảo trì).
- Biểu hiện ownership: feature gây lỗi prod thì mình là người đầu tiên nhảy vào điều tra, không nói "phần đó qua bên kia rồi". Đo bằng việc nhận trách nhiệm khi có sự cố, không chỉ nhận công.
- Cân bằng: own không phải ôm tất cả một mình — vẫn nhờ review, hỏi chuyên gia (devops, design). Own = đảm bảo feature về đích, kể cả khi kéo người khác vào.

```
Hiểu yêu cầu -> Thiết kế -> Code(DB/BE/FE) -> Test -> Ship -> Theo dõi prod -> Bảo trì
(KHONG dừng ở "code đã chạy trên máy em")
```

</details>

---

> 📌 *Vai trò fullstack đánh giá cao người nhìn được bức tranh **end-to-end**: từ UI → API → DB → deploy, và biết **ship nhanh** mà vẫn kiểm soát chất lượng.*
> Hãy gắn câu trả lời với một feature thật bạn từng làm trọn vẹn cả hai đầu. 🚀
