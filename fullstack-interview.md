# 🔗 Bộ câu hỏi phỏng vấn Fullstack (FE ↔ BE & Ways of Working)

> Tài liệu bổ trợ cho vai trò **Fullstack Developer** (React/Next.js + NestJS/Node.js) — tập trung vào phần **xuyên suốt** mà file frontend/backend riêng lẻ chưa bao quát: tích hợp **end-to-end**, **analytics/telemetry**, và **cách làm việc** (MVP-first, AI-native).
> Mỗi đáp án nằm trong khối **collapse** (`<details>`) — bấm để xem lời giải + ví dụ.

## 📚 Mục lục

1. [Fullstack End-to-end (FE và BE)](#1-fullstack-end-to-end-fe-và-be)
2. [Domain: Analytics, Telemetry và Feature Flags](#2-domain-analytics-telemetry-và-feature-flags)

---


## 1. Fullstack End-to-end (FE và BE)

### 1.1. Hãy mô tả quy trình thiết kế một feature từ đầu đến cuối (UI → API → DB). Bạn bắt đầu từ đâu và các bước theo thứ tự nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Mình tiếp cận theo hướng contract-first: trước tiên hiểu kỹ nghiệp vụ và domain model, rồi thiết kế DB schema vì đây là phần khó migrate nhất về sau. Tiếp theo mình chốt API contract gồm endpoint, request/response và error shape, thường viết bằng OpenAPI hoặc Zod để FE và BE dùng chung một nguồn sự thật. Khi đã có hợp đồng rõ ràng thì FE và BE làm song song, FE dùng mock nên không bị block chờ BE. Cuối cùng mình tích hợp và test end-to-end cả happy path lẫn edge case. Nguyên tắc xuyên suốt là mọi business rule phải nằm ở BE và tách DTO khỏi entity DB."

---

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

**📌 Câu trả lời mẫu:**

"Có ba cách chính. OpenAPI cộng codegen thì source là file spec, ngôn ngữ độc lập và hợp cho API công khai nhiều client, nhưng phải maintain spec và không tự validate runtime. Zod schema dùng chung thì một schema vừa cho type tĩnh vừa validate runtime, đúng một nguồn sự thật, nhưng buộc cả hai đầu dùng TypeScript trong monorepo. tRPC thì type-safe đầu cuối không cần codegen, nhưng cũng buộc TS hai đầu và không có HTTP contract rõ cho bên thứ ba. Với stack Next.js cộng NestJS thì mình thường chọn OpenAPI qua nest/swagger hoặc Zod dùng chung, vì NestJS không native với tRPC."

---

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

**📌 Câu trả lời mẫu:**

"Khi login, Next.js gọi sang NestJS, BE cấp một access token ngắn hạn khoảng 15 phút và một refresh token dài hạn, rồi set cả hai vào cookie với cờ HttpOnly, Secure và SameSite. Các request sau trình duyệt tự đính kèm cookie nên guard chỉ việc verify JWT; khi access hết hạn thì BE trả 401 và FE gọi sang refresh để lấy token mới. Lý do httpOnly an toàn hơn localStorage là localStorage đọc được bằng JavaScript nên chỉ cần dính một lỗ hổng XSS là token bị đánh cắp ngay, còn httpOnly cookie thì JS không đọc được nên chặn được hướng tấn công đó. Đánh đổi là cookie mở ra rủi ro CSRF, nhưng CSRF dễ phòng hơn nhiều. Một lợi ích nữa là SSR có thể đọc được phiên ngay khi render ở server."

---

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

**📌 Câu trả lời mẫu:**

"CSRF xảy ra vì trình duyệt tự đính kèm cookie kể cả khi request đến từ một trang độc hại, nên kẻ tấn công có thể mượn phiên đăng nhập của user. Lớp phòng thủ đầu tiên là cờ SameSite: mình hay đặt Lax làm mặc định vì nó vẫn gửi cookie với điều hướng GET top-level nhưng chặn POST cross-site. Vấn đề là khi FE và BE nằm khác site thì buộc phải dùng SameSite None, lúc đó cờ này mất tác dụng nên mình bổ sung CSRF token kiểu double-submit hoặc kiểm tra Origin/Referer. Cần lưu ý subdomain cùng registrable domain vẫn được coi là same-site, và trình duyệt cũ có thể không hỗ trợ SameSite, nên mình không bao giờ chỉ dựa vào một mình nó."

---

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

**📌 Câu trả lời mẫu:**

"Mình dùng mô hình hai token: access ngắn để hạn chế thiệt hại nếu lộ, và refresh dài để đổi access mới mà không bắt user đăng nhập lại. Refresh token rotation nghĩa là mỗi lần dùng refresh thì BE cấp một cái mới và vô hiệu hóa cái cũ ngay. Cần rotation vì nếu refresh bị trộm, kẻ tấn công và user thật sẽ tranh nhau dùng; khi BE thấy một token đã đánh dấu used lại được dùng tiếp thì đó là dấu hiệu reuse, mình sẽ thu hồi cả family token đó và buộc đăng nhập lại. Về kỹ thuật mình lưu refresh dưới dạng hash kèm familyId để revoke theo chuỗi, giới hạn path cookie về đúng endpoint refresh, và phía FE dùng single-flight để tránh nhiều tab cùng refresh gây race."

---

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

**📌 Câu trả lời mẫu:**

"Luồng là FE xin BE quyền upload một file với tên, type và size cụ thể; BE validate loại, kích thước và quyền rồi tạo một presigned PUT URL hạn ngắn trả về. FE dùng URL đó PUT thẳng lên S3 và theo dõi progress, xong thì báo lại object key để BE lưu metadata và xử lý nền như tạo thumbnail hay quét virus. Mình ưu tiên upload thẳng lên S3 vì nó không chiếm băng thông và bộ nhớ của BE, BE không trở thành bottleneck với file lớn, mà vẫn kiểm soát được qua presigned URL. Một điểm cần cẩn thận là FE có thể khai gian contentType, nên file thật vẫn phải được xác minh ở phía server hoặc qua S3 event, ví dụ kiểm magic bytes và quét virus."

---

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

**📌 Câu trả lời mẫu:**

"Optimistic UI là cập nhật giao diện ngay theo kết quả mình kỳ vọng trước khi server xác nhận, để app cảm giác nhanh và mượt; nếu server trả lỗi thì rollback lại trạng thái cũ. Mình chỉ dùng cho hành động khả năng thành công cao và ít rủi ro như like, toggle hay comment, chứ không dùng cho thao tác cần chính xác tuyệt đối như thanh toán. Với React Query thì pattern là trong onMutate mình cancelQueries, snapshot lại dữ liệu hiện tại rồi setQueryData; onError thì khôi phục từ snapshot; onSettled thì invalidateQueries để đồng bộ lại với server. Lỗi hay gặp là quên cancelQueries khiến refetch ghi đè update, quên snapshot nên không rollback được, hoặc quên invalidate khiến UI lệch với server."

---

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

**📌 Câu trả lời mẫu:**

"Nếu mỗi endpoint trả lỗi một kiểu thì FE phải xử lý rời rạc, dễ sót trường hợp, khó hiển thị nhất quán và khó map lỗi vào từng field của form. Một error shape thống nhất cho phép FE có đúng một chỗ để parse và xử lý mọi lỗi. Mình thường đề xuất cấu trúc gần RFC 7807 gồm code, message, statusCode, traceId và fieldErrors; phía BE dùng một exception filter để chuẩn hóa mọi lỗi về cùng shape này. Nguyên tắc quan trọng là FE phân nhánh theo code ổn định chứ không dựa vào message vì message có thể đổi hoặc đa ngôn ngữ, dùng traceId để tra log, và tách fieldErrors riêng để gắn vào form."

---

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

**📌 Câu trả lời mẫu:**

"Offset pagination dùng kiểu OFFSET và LIMIT, cho phép nhảy tới trang bất kỳ nhưng khi offset lớn thì DB phải quét và bỏ qua nhiều dòng nên chậm dần. Cursor pagination thì neo vào giá trị thực của một dòng, ví dụ created_at kèm id để phá hòa, nên hiệu năng ổn định nhờ dùng index nhưng chỉ đi next/prev được. Với dữ liệu thay đổi liên tục thì cursor tốt hơn vì offset hay bị trùng hoặc sót item khi có insert/delete xen vào giữa lúc phân trang, còn cursor neo vào một dòng cụ thể nên không bị lệch. FE và BE cần đồng bộ response shape kiểu data, nextCursor và hasNext; cursor phải là opaque để FE không tự suy diễn; và thứ tự sắp xếp phải nhất quán giữa các lần gọi. Phía FE thì cursor khớp rất tự nhiên với useInfiniteQuery."

---

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

**📌 Câu trả lời mẫu:**

"Mình chọn dựa trên chiều giao tiếp: nếu client cần gửi dữ liệu liên tục và cần hai chiều như chat, game hay collab thì dùng WebSocket; còn nếu chỉ server đẩy xuống một chiều như notification, feed hay progress thì SSE đơn giản hơn và tận dụng được auto-reconnect của trình duyệt. Về luồng, cả hai đều cần xử lý auth, với WS thì truyền token qua handshake hoặc cookie, còn SSE dùng cookie bình thường. Hai phía đều phải lo reconnect và resume: SSE có sẵn Last-Event-ID, còn WS thì mình tự thiết kế sequence hoặc ack. Khi scale nhiều instance thì cần một lớp pub/sub như Redis adapter để broadcast, và phía FE phải nhớ cleanup, hủy subscription khi unmount và throttle khi sự kiện dồn dập."

---

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

**📌 Câu trả lời mẫu:**

"Mình luôn xem client là môi trường không tin được, vì user có thể mở DevTools, sửa request hoặc gọi thẳng API bằng curl/Postman, nên mọi logic quan trọng như giá, quyền hay điều kiện nghiệp vụ đều phải nằm ở backend. FE validation với mình chỉ để cải thiện UX, còn BE mới là nguồn sự thật bắt buộc. Mình nhớ một case kinh điển: tính giảm giá ở client rồi gửi giá đã giảm lên server — chỉ cần user sửa hệ số thành 0 là mua được 0 đồng. Cách phòng là BE tự tính giá từ DB và tự enforce quyền, FE chỉ gửi id hoặc mã coupon chứ không gửi số tiền. Mình thường dùng schema chung như Zod để đồng bộ luật cho đỡ lệch, nhưng BE vẫn phải validate lại độc lập, vì với mình FE validation là cánh cửa lịch sự còn BE validation mới là ổ khóa."

---

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

**📌 Câu trả lời mẫu:**

"Mình chỉ version API khi buộc phải làm breaking change như xóa field, đổi tên, đổi kiểu hay đổi ngữ nghĩa, trong khi vẫn còn client cũ đang chạy ngoài thực tế. Mục tiêu là client cũ không vỡ còn client mới dùng được phiên bản mới. Về chiến lược, mình hay dùng URI versioning kiểu /v1/posts vì rõ ràng, dễ route, dễ cache và dễ debug, dù nó hơi làm bẩn URL; ngoài ra có header versioning sạch URL nhưng khó test tay, và query param thì đơn giản nhưng dễ quên. Nguyên tắc của mình là ưu tiên thay đổi non-breaking — chỉ thêm field thì không cần version — và chỉ bump khi thực sự breaking. Quan trọng nữa là phải có chính sách deprecate rõ ràng với header Deprecation/Sunset, đo traffic phiên bản cũ và phối hợp với team FE để họ chủ động migrate trước khi mình tắt."

---

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

**📌 Câu trả lời mẫu:**

"Theo kinh nghiệm của mình, event client-side rất dễ mất hoặc sai lệch: bị ad-blocker chặn, user đóng tab trước khi event kịp gửi, mạng chập chờn, và dễ bị spoof. Trong khi đó server-side được phát ngay khi backend xử lý request thật nên khó chặn, khó giả mạo và nhất quán với DB vì cùng một nguồn sự thật. Vì vậy mình luôn ưu tiên server-side cho các 'money/business event' quan trọng như order_created, và phát event sau khi DB commit để chắc chắn không track nhầm khi giao dịch fail. Nhưng client-side vẫn bắt buộc với những tương tác chỉ tồn tại trên UI mà server không thấy được, như scroll depth, thời gian xem một component, rage-click hay A/B test ở tầng render. Thực tế mình theo hướng hybrid: client lo phần UX/engagement, server lo phần business quan trọng."

---

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

**📌 Câu trả lời mẫu:**

"Với event schema, mình bắt đầu từ naming nhất quán theo kiểu object_action ở thì quá khứ như order_created, dùng snake_case và tránh tên mơ hồ kiểu 'click' hay 'event1' vì sau này không ai hiểu nổi. Về property, mình giữ kiểu cố định — amount luôn là number — và tách rõ thuộc tính riêng của event như order_id khỏi context chung như user hay device, đồng thời tránh nhồi PII vào trong đó. Phần versioning là thứ mình cẩn thận nhất: thêm property optional thì tương thích ngược, không cần version mới; nhưng nếu đổi kiểu, đổi ý nghĩa hoặc bỏ field thì phải thêm trường version hoặc tạo hẳn event mới. Cái bẫy lớn nhất là tái dùng tên cũ với ý nghĩa mới — ví dụ đổi amount từ đồng sang cent — vì dashboard và pipeline cũ sẽ tính tổng sai mà không hề báo lỗi."

---

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

**📌 Câu trả lời mẫu:**

"At-least-once nghĩa là một event có thể đến từ hai lần trở lên — client retry vì timeout, hoặc queue redeliver khi consumer crash trước khi kịp ack — và hệ quả là số liệu bị phồng lên. Idempotent ingestion là khi mình xử lý cùng một event nhiều lần nhưng kết quả vẫn giống như xử lý đúng một lần. Cách mình làm là mỗi event mang một event_id duy nhất sinh ngay tại điểm phát, để khi retry thì id vẫn giữ nguyên; sau đó mình lưu các id đã xử lý vào một store có ràng buộc unique như DB index hoặc Redis SET, nếu trùng thì drop. Điểm cần lưu ý là cửa sổ TTL của store dedupe: nhớ mãi thì store phình to, mà quá ngắn thì bản trùng đến muộn vẫn lọt qua, nên mình luôn chọn TTL lớn hơn thời gian retry tối đa."

---

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

**📌 Câu trả lời mẫu:**

"Feature flag với mình là cách bật/tắt hoặc điều khiển một tính năng ngay tại runtime mà không cần deploy lại, nhờ đó tách được việc deploy code khỏi việc release tính năng. Rollout theo phần trăm là mở dần 5% rồi 25% rồi 100% kiểu canary, và điểm mấu chốt là phải sticky — mình hash userId để cùng một user luôn nhận cùng kết quả, nếu không UI sẽ nhấp nháy. Kill switch là tắt cứng một tính năng lỗi ngay lập tức mà không phải rollback cả bản deploy, còn targeting là bật theo thuộc tính như quốc gia, gói trả phí hay beta tester. Ở backend NestJS mình đánh giá flag tại server vì đó là nguồn sự thật cho tính năng nhạy cảm, bọc qua guard hoặc service; còn ở Next.js mình đánh giá flag ở phía server như Server Component hay middleware rồi truyền xuống để tránh flicker và hydration mismatch. Cuối cùng, flag luôn phải có default an toàn — service down thì fallback về OFF — và phải dọn sau khi tính năng đã ổn định để tránh nợ kỹ thuật."

---

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

**📌 Câu trả lời mẫu:**

"A/B test là mình chia ngẫu nhiên user thành nhóm Control và Variant rồi so một metric mục tiêu để xem thay đổi thật sự tốt hơn hay chỉ là may rủi. Mình luôn chốt một primary metric như conversion rate trước khi chạy, kèm vài guardrail metric như tỉ lệ lỗi hay load time, và tuyệt đối tránh đào bới nhiều metric rồi chọn cái đẹp vì đó là p-hacking. Phân nhóm thì sticky bằng hash userId để một người luôn nằm một nhánh. Về sample size, mình ước lượng đủ lớn để phát hiện chênh lệch và chạy đủ bội số tuần để tránh hiệu ứng ngày, đồng thời không dừng sớm khi B vừa vượt A vì peeking dễ cho dương tính giả. Significance thì p-value dưới 0.05 nghĩa là xác suất chênh lệch chỉ do ngẫu nhiên dưới 5%, nhưng mình luôn phân biệt nó với effect size — một chênh lệch 0.1% có thể significant với mẫu khổng lồ nhưng chẳng có ý nghĩa kinh doanh. Phần tính toán mình để cho thư viện như scipy hoặc nền tảng experimentation lo, không ước lượng bằng mắt."

---

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

**📌 Câu trả lời mẫu:**

"Một pipeline analytics nhẹ với mình thường là producer đẩy vào queue, processor xử lý rồi nạp vào warehouse và lên dashboard. Vai trò của queue như Kafka, SQS hay RabbitMQ là decouple producer với processor: app chỉ cần enqueue rồi trả response thật nhanh, queue hấp thụ tải đỉnh giúp back-pressure, và nếu processor chết thì event vẫn nằm trong queue chờ xử lý lại — tất nhiên kết hợp với at-least-once nên phải có dedupe. Về batch và stream, batch gom theo lô nên latency cao nhưng đơn giản và rẻ, hợp với báo cáo doanh thu cuối ngày; còn stream xử lý từng event gần real-time, latency thấp nhưng phức tạp và đắt hơn, hợp với cảnh báo gian lận hay live dashboard. Cách mình chọn rất thực tế: nếu chỉ cần biết hôm qua có bao nhiêu order thì batch, còn cần phản ứng tức thì thì stream. Với dự án nhỏ mình chưa vội dùng Kafka, BullMQ trên Redis hoặc SQS đã đủ decouple và retry, chỉ nâng lên Kafka khi cần throughput lớn và khả năng replay log."

---

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

**📌 Câu trả lời mẫu:**

"PII là dữ liệu nhận dạng cá nhân như email, số điện thoại, địa chỉ, IP, thông tin thanh toán và đôi khi cả userId thật, và log với analytics chính là nơi hay rò rỉ nhất. Lý do nguy hiểm là log lưu rất lâu, được sao chép sang nhiều hệ thống và nhiều người truy cập, nên bề mặt rò rỉ rất rộng — trong khi GDPR yêu cầu consent, giới hạn mục đích và data minimization. Cách mình làm là không log PII ngay từ đầu, redact hoặc mask ngay tại tầng logger, và dùng id hash thay cho email thật để vẫn join được dữ liệu mà không lộ danh tính. Về consent, mình chỉ track khi user đồng ý và frontend không load script tracking trước khi có consent. Còn 'right to be forgotten' thì mình luôn gắn dữ liệu với một userId để có thể tìm và xóa theo cascade, tránh rải PII lung tung vào free-text log, đồng thời đặt TTL retention để không giữ dữ liệu vô thời hạn."

---

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

**📌 Câu trả lời mẫu:**

"Telemetry là ghi lại các sự kiện mô tả hành trình người dùng (install → mở app → xong tutorial → qua level → mua hàng) để mình dựng được funnel — một chuỗi bước có thứ tự mà ở mỗi bước mình đo tỉ lệ đi tiếp, nhờ đó thấy người dùng rơi rớt mạnh nhất ở đâu để tập trung sửa. Vài điểm đặc thù mình luôn chú ý: client mobile hay mất mạng nên phải buffer event ở local rồi gửi theo batch, và phải tách `occurredAt` (lúc sự kiện thật xảy ra) khỏi `receivedAt` (lúc server nhận) để số liệu không lệch. Vì gửi lại sau offline dễ trùng nên mỗi event cần một `event_id` để server dedupe. Những event liên quan tiền hay chống gian lận thì phải xác nhận ở server, không tin client. Cuối cùng mình gắn `session_id` và phân tích theo cohort để đo retention kiểu D1/D7/D30, và rất cẩn thận về privacy — tránh PII, tôn trọng consent (có người dùng nhỏ tuổi thì COPPA/GDPR chặt hơn)."

---

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

---

> 📌 *Vai trò fullstack đánh giá cao người nhìn được bức tranh **end-to-end**: từ UI → API → DB → deploy, và biết **ship nhanh** mà vẫn kiểm soát chất lượng.*
> Hãy gắn câu trả lời với một feature thật bạn từng làm trọn vẹn cả hai đầu. 🚀
