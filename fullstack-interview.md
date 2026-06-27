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

Cách tiếp cận hiệu quả nhất là thiết kế theo hướng "contract-first": thống nhất hợp đồng dữ liệu trước, rồi mới triển khai song song FE và BE. Thứ tự thực tế thường là:

1. **Hiểu yêu cầu nghiệp vụ và xác định domain model**: Feature này thao tác trên thực thể (entity) nào, các trạng thái và quy tắc nghiệp vụ ra sao.
2. **Thiết kế DB schema trước UI**: Vì DB là nơi khó migrate nhất. Xác định bảng, quan hệ, index, ràng buộc (unique, foreign key). Ví dụ feature "comment trên bài viết" cần bảng `comments` với `post_id`, `author_id`, `parent_id` (cho nested), index trên `post_id`.
3. **Định nghĩa API contract**: Endpoint, method, request/response shape, error shape, pagination. Đây là điểm FE và BE "bắt tay". Nên viết bằng OpenAPI hoặc Zod schema dùng chung.
4. **Triển khai song song**: BE viết theo contract (có thể mock trước), FE dùng mock/codegen client để không bị block.
5. **Tích hợp và kiểm thử e2e**: Ghép thật, test happy path + edge case (lỗi mạng, validate fail, race condition).

Một nguyên tắc quan trọng: **mọi business rule phải nằm ở BE** (FE chỉ làm UX nhanh), và DB schema không nên phản chiếu 1-1 lên API response (tách DTO khỏi entity).

```ts
// Bước 3: contract dùng chung (shared/contracts/comment.ts)
import { z } from 'zod';

export const CreateCommentDto = z.object({
  postId: z.string().uuid(),
  body: z.string().min(1).max(2000),
  parentId: z.string().uuid().optional(),
});

export const CommentResponse = z.object({
  id: z.string().uuid(),
  body: z.string(),
  authorId: z.string().uuid(),
  createdAt: z.string().datetime(),
});

export type CreateCommentDto = z.infer<typeof CreateCommentDto>;
export type CommentResponse = z.infer<typeof CommentResponse>;
```

Khi nào dùng cách nào: với team có cả FE và BE, contract-first giúp parallel hóa công việc. Với prototype 1 người, đôi khi code BE trước rồi suy ra type cho FE là đủ nhanh.

</details>

---

### 1.2. Có những cách nào để chia sẻ kiểu dữ liệu (type) giữa FE và BE? So sánh OpenAPI + codegen, Zod schema dùng chung, và tRPC.

<details>
<summary>👉 Xem câu trả lời</summary>

Có ba hướng phổ biến, đánh đổi giữa độ linh hoạt, độ chặt chẽ và mức độ ràng buộc công nghệ:

| Cách | Nguồn sự thật (source of truth) | Ưu điểm | Nhược điểm | Khi nào dùng |
|------|--------------------------------|---------|-----------|--------------|
| **OpenAPI + codegen** | File OpenAPI/Swagger | Chuẩn ngôn ngữ-độc lập, FE/BE/mobile/khách bên ngoài đều dùng được; có công cụ sinh client, mock, docs | Phải maintain spec; codegen đôi khi sinh type cồng kềnh; runtime không tự validate trừ khi thêm | API công khai, đa client, đa ngôn ngữ |
| **Zod schema dùng chung** | Zod schema trong package chung (monorepo) | Vừa là type tĩnh (`z.infer`) vừa validate runtime; một nguồn sự thật duy nhất | Buộc cả hai phía dùng TypeScript + Zod; cần monorepo hoặc publish package | Fullstack TS, monorepo, cần validate cả hai đầu |
| **tRPC** | Type của router BE | Type-safe đầu cuối không cần codegen; autocomplete xuyên FE-BE; refactor an toàn | Ràng buộc TS hai đầu; khó cho client không phải TS; không có HTTP contract rõ ràng cho bên thứ ba | Monorepo TS, app nội bộ, ưu tiên tốc độ phát triển |

```ts
// tRPC: BE định nghĩa router, FE tự suy ra type
// server/router.ts
export const appRouter = router({
  getPost: publicProcedure
    .input(z.object({ id: z.string().uuid() }))
    .query(({ input }) => postService.findById(input.id)),
});
export type AppRouter = typeof appRouter;

// client.ts (FE) - không cần codegen, type chảy thẳng từ BE
const post = await trpc.getPost.query({ id }); // post được suy type đầy đủ
```

Lưu ý quan trọng: NestJS không native với tRPC (tRPC sinh ra từ hệ Next.js/Express thuần). Với stack **Next.js + NestJS**, lựa chọn thực tế nhất là **OpenAPI + codegen** (NestJS có `@nestjs/swagger` sinh spec tự động từ decorator) hoặc **Zod schema dùng chung trong monorepo**. tRPC ít phù hợp vì NestJS đã có cơ chế controller/DI riêng.

</details>

---

### 1.3. Trình bày luồng authentication end-to-end khi Next.js gọi NestJS với JWT lưu trong httpOnly cookie. Tại sao httpOnly cookie an toàn hơn localStorage?

<details>
<summary>👉 Xem câu trả lời</summary>

Luồng tổng quát:

1. User submit form login trên Next.js → gọi `POST /auth/login` tới NestJS (hoặc qua Next.js Route Handler làm proxy).
2. NestJS xác thực, tạo **access token** (sống ngắn, ví dụ 15 phút) và **refresh token** (sống dài, ví dụ 7 ngày).
3. NestJS set cả hai vào response qua `Set-Cookie` với cờ `HttpOnly`, `Secure`, `SameSite`.
4. Các request sau, trình duyệt tự đính kèm cookie. NestJS dùng guard đọc cookie, verify JWT.
5. Khi access token hết hạn → BE trả `401` → FE gọi `/auth/refresh` (gửi refresh token trong cookie) để lấy access token mới.

```ts
// NestJS: set cookie khi login
res.cookie('access_token', accessToken, {
  httpOnly: true,        // JS không đọc được -> chống XSS đánh cắp token
  secure: true,          // chỉ gửi qua HTTPS
  sameSite: 'lax',       // hoặc 'strict'/'none' tùy kiến trúc cross-site
  maxAge: 15 * 60 * 1000,
  path: '/',
});
```

**Tại sao httpOnly cookie an toàn hơn localStorage:**

- `localStorage` truy cập được bằng JavaScript → nếu có lỗ hổng **XSS**, kẻ tấn công chạy `localStorage.getItem('token')` và đánh cắp ngay.
- `httpOnly` cookie **không đọc được bằng JS** → kịch bản XSS đánh cắp token bị chặn.

Đánh đổi: cookie tự gửi kèm mọi request nên mở ra nguy cơ **CSRF**, cần biện pháp chống riêng (xem câu sau). Tóm lại: localStorage dễ bị XSS, cookie dễ bị CSRF nhưng CSRF dễ phòng thủ hơn và XSS-đánh-cắp-token là rủi ro nặng hơn, nên httpOnly cookie thường được ưu tiên.

Lưu ý SSR: với Next.js, dùng cookie giúp Server Component / Route Handler đọc được phiên đăng nhập ngay khi render server-side, điều mà localStorage (chỉ tồn tại ở client) không làm được.

</details>

---

### 1.4. Khi dùng JWT trong cookie, làm sao chống CSRF? Giải thích vai trò của SameSite và khi nào nó vẫn chưa đủ.

<details>
<summary>👉 Xem câu trả lời</summary>

CSRF (Cross-Site Request Forgery) xảy ra vì trình duyệt **tự động đính kèm cookie** vào mọi request tới domain đó, kể cả request do trang web độc hại khởi tạo. Các lớp phòng thủ:

**1. Cờ `SameSite` trên cookie** — lớp phòng thủ đầu tiên:

| Giá trị | Hành vi | Phù hợp |
|---------|---------|---------|
| `Strict` | Cookie KHÔNG gửi với bất kỳ request cross-site nào | Bảo mật cao nhất, nhưng click link từ ngoài vào sẽ mất phiên |
| `Lax` | Cookie gửi với navigation top-level GET, KHÔNG gửi với POST cross-site | Mặc định tốt cho hầu hết app |
| `None` | Cookie luôn gửi (bắt buộc kèm `Secure`) | Khi FE và BE khác site và cần cookie cross-site |

**2. CSRF token (double-submit cookie hoặc synchronizer token)** — cần khi `SameSite=None`:

```ts
// Double-submit: BE set một cookie csrf không-httpOnly + FE gửi lại trong header
// BE kiểm tra header khớp cookie
const csrfCookie = req.cookies['csrf_token'];
const csrfHeader = req.headers['x-csrf-token'];
if (csrfCookie !== csrfHeader) throw new ForbiddenException('CSRF mismatch');
```

**SameSite chưa đủ khi nào:**

- Khi FE và BE ở **khác site** (ví dụ `app.example.com` gọi `api.other.com`) buộc phải dùng `SameSite=None`, lúc đó SameSite mất tác dụng chống CSRF → phải có CSRF token hoặc kiểm tra `Origin`/`Referer` header.
- Subdomain cùng site (`app.example.com` và `api.example.com`) được coi là same-site nên `Lax`/`Strict` vẫn hoạt động — đây là lý do nhiều team đặt FE và API cùng một registrable domain.
- Một số trình duyệt cũ không hỗ trợ SameSite → cần defense-in-depth.

Thực tế nên: cùng site → `SameSite=Lax` + kiểm tra `Origin`. Khác site → `SameSite=None; Secure` + CSRF token.

</details>

---

### 1.5. Thiết kế cơ chế refresh token end-to-end an toàn. Refresh token rotation là gì và tại sao cần nó?

<details>
<summary>👉 Xem câu trả lời</summary>

Mô hình hai token: **access token** sống ngắn (vài phút) để hạn chế thiệt hại nếu lộ, **refresh token** sống dài để đổi lấy access token mới mà không bắt user đăng nhập lại.

**Refresh token rotation**: mỗi lần dùng refresh token để lấy access token mới, BE đồng thời cấp **refresh token mới** và **vô hiệu hóa cái cũ**. Lý do: nếu refresh token bị đánh cắp, kẻ tấn công và user thật sẽ "tranh nhau" dùng cùng một token. Khi BE thấy một refresh token đã bị xoay (đã dùng rồi mà lại được dùng tiếp) → phát hiện **reuse** → thu hồi toàn bộ chuỗi token của phiên đó (buộc đăng nhập lại). Đây là cơ chế phát hiện trộm token.

```ts
// NestJS: refresh có rotation + phát hiện reuse
async refresh(oldToken: string, res: Response) {
  const record = await this.tokenRepo.findByHash(hash(oldToken));
  if (!record) throw new UnauthorizedException();

  if (record.usedAt) {
    // Token đã dùng rồi mà lại đến -> nghi ngờ bị đánh cắp
    await this.tokenRepo.revokeFamily(record.familyId);
    throw new UnauthorizedException('Token reuse detected');
  }

  await this.tokenRepo.markUsed(record.id);
  const newRefresh = this.issueRefresh(record.familyId, record.userId);
  const newAccess = this.issueAccess(record.userId);

  res.cookie('refresh_token', newRefresh, { httpOnly: true, secure: true, sameSite: 'lax', path: '/auth/refresh' });
  res.cookie('access_token', newAccess, { httpOnly: true, secure: true, sameSite: 'lax' });
}
```

Các điểm thực hành tốt:

- Lưu refresh token ở DB dưới dạng **hash** (không lưu plaintext), gắn `familyId` để revoke cả chuỗi.
- Giới hạn `path` của refresh cookie (ví dụ `/auth/refresh`) để nó không bị gửi kèm mọi request.
- Phía FE: khi nhận `401`, dùng một **single-flight** (chỉ một request refresh chạy, các request khác chờ) để tránh nhiều tab cùng refresh gây race và làm rotation hỏng.

</details>

---

### 1.6. Hãy thiết kế luồng upload file end-to-end dùng presigned URL / S3. Tại sao nên upload trực tiếp lên S3 thay vì qua server?

<details>
<summary>👉 Xem câu trả lời</summary>

Luồng presigned URL gồm ba bên (FE, BE, S3):

1. FE gọi BE: "Tôi muốn upload file tên X, type Y, size Z".
2. BE **validate** (loại file, kích thước tối đa, quyền) rồi tạo **presigned PUT URL** có thời hạn ngắn, trả về cho FE.
3. FE `PUT` file **trực tiếp lên S3** bằng URL đó, theo dõi progress.
4. Sau khi xong, FE báo BE "đã upload object key này" → BE lưu metadata vào DB, có thể kích hoạt xử lý nền (tạo thumbnail, quét virus).

```ts
// NestJS: tạo presigned URL
import { getSignedUrl } from '@aws-sdk/s3-request-presigner';
import { PutObjectCommand } from '@aws-sdk/client-s3';

async createUploadUrl(dto: { filename: string; contentType: string; size: number }) {
  if (size > 10 * 1024 * 1024) throw new BadRequestException('File quá lớn');
  if (!['image/png', 'image/jpeg'].includes(dto.contentType))
    throw new BadRequestException('Loại file không hỗ trợ');

  const key = `uploads/${crypto.randomUUID()}-${dto.filename}`;
  const command = new PutObjectCommand({
    Bucket: this.bucket,
    Key: key,
    ContentType: dto.contentType,
  });
  const url = await getSignedUrl(this.s3, command, { expiresIn: 300 }); // 5 phút
  return { url, key };
}
```

```ts
// FE: upload trực tiếp + progress bằng XHR
function upload(file: File, url: string, onProgress: (p: number) => void) {
  return new Promise<void>((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open('PUT', url);
    xhr.setRequestHeader('Content-Type', file.type);
    xhr.upload.onprogress = (e) => onProgress(Math.round((e.loaded / e.total) * 100));
    xhr.onload = () => (xhr.status < 300 ? resolve() : reject(new Error('Upload thất bại')));
    xhr.onerror = () => reject(new Error('Lỗi mạng'));
    xhr.send(file);
  });
}
```

**Tại sao upload thẳng lên S3 thay vì qua server:**

- File không chiếm băng thông và bộ nhớ của BE; BE không thành nút thắt cổ chai khi nhiều người upload đồng thời.
- Giảm chi phí và độ trễ (S3 có hạ tầng tối ưu cho việc này).
- BE vẫn giữ quyền kiểm soát qua presigned URL (chỉ key/type/size được phép, có thời hạn).

Lưu ý bảo mật: dù FE gửi `contentType`, vẫn nên **xác minh lại file thực tế phía server/qua S3 event** (kiểm tra magic bytes, quét virus) vì FE có thể nói dối. Có thể dùng `fetch` + `ReadableStream` thay XHR, nhưng XHR vẫn là cách đơn giản nhất để có progress upload.

</details>

---

### 1.7. Optimistic UI là gì? Hãy minh họa cách làm optimistic update kèm rollback khi server trả lỗi, dùng React Query.

<details>
<summary>👉 Xem câu trả lời</summary>

**Optimistic UI**: cập nhật giao diện ngay lập tức theo kết quả kỳ vọng **trước khi** server xác nhận, để cảm giác phản hồi tức thì. Nếu server lỗi, ta **rollback** về trạng thái cũ. Phù hợp với hành động khả năng thành công cao và ít rủi ro (like, toggle, thêm comment); không phù hợp với thao tác cần độ chính xác cao (thanh toán).

Với React Query (`useMutation`), pattern chuẩn dùng `onMutate` / `onError` / `onSettled`:

```tsx
const queryClient = useQueryClient();

const likeMutation = useMutation({
  mutationFn: (postId: string) => api.likePost(postId),

  onMutate: async (postId) => {
    // 1. Hủy các refetch đang chạy để không ghi đè cập nhật lạc quan
    await queryClient.cancelQueries({ queryKey: ['post', postId] });

    // 2. Lưu snapshot để rollback
    const previous = queryClient.getQueryData(['post', postId]);

    // 3. Cập nhật cache ngay lập tức
    queryClient.setQueryData(['post', postId], (old: any) => ({
      ...old,
      liked: true,
      likeCount: old.likeCount + 1,
    }));

    return { previous }; // context cho onError
  },

  onError: (_err, postId, context) => {
    // 4. Server lỗi -> trả lại snapshot cũ
    if (context?.previous) {
      queryClient.setQueryData(['post', postId], context.previous);
    }
    toast.error('Không thể like, vui lòng thử lại');
  },

  onSettled: (_data, _err, postId) => {
    // 5. Đồng bộ lại với server (dù thành công hay thất bại)
    queryClient.invalidateQueries({ queryKey: ['post', postId] });
  },
});
```

Các điểm dễ sai thường gặp:

- Quên `cancelQueries` → một refetch đang bay về sau có thể ghi đè cập nhật lạc quan.
- Quên lưu snapshot → không rollback được.
- Quên `invalidateQueries` ở `onSettled` → UI và server lệch nhau (ví dụ likeCount thực tế khác).
- Áp optimistic cho thao tác cần xác nhận server (thanh toán, chuyển tiền) → rất nguy hiểm, không nên.

</details>

---

### 1.8. Tại sao cần một error shape thống nhất giữa FE và BE? Hãy đề xuất một cấu trúc lỗi chuẩn và cách FE tiêu thụ nó.

<details>
<summary>👉 Xem câu trả lời</summary>

Nếu mỗi endpoint trả lỗi theo định dạng khác nhau, FE phải viết logic xử lý lỗi rời rạc, dễ sót case, khó hiển thị thông báo nhất quán và khó map lỗi field vào form. Một error shape thống nhất giúp FE có **một chỗ duy nhất** parse lỗi.

Cấu trúc đề xuất (gần với RFC 7807 Problem Details, có mở rộng cho field errors):

```ts
// shared/error.ts
interface ApiError {
  code: string;              // mã máy đọc được, ví dụ 'VALIDATION_FAILED'
  message: string;           // thông điệp con người đọc (có thể i18n)
  statusCode: number;        // HTTP status
  traceId?: string;          // để trace log giữa FE-BE
  fieldErrors?: Record<string, string[]>; // lỗi theo từng field cho form
}
```

```ts
// NestJS: exception filter chuẩn hóa mọi lỗi về cùng shape
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost) {
    const res = host.switchToHttp().getResponse<Response>();
    const status = exception instanceof HttpException ? exception.getStatus() : 500;
    const body: ApiError = {
      code: exception instanceof HttpException ? exception.name : 'INTERNAL_ERROR',
      message: exception instanceof HttpException ? exception.message : 'Lỗi hệ thống',
      statusCode: status,
      traceId: res.req.headers['x-trace-id'] as string,
    };
    res.status(status).json(body);
  }
}
```

```ts
// FE: một hàm tiêu thụ lỗi dùng chung
async function handleApiError(err: ApiError) {
  if (err.code === 'VALIDATION_FAILED' && err.fieldErrors) {
    // map vào react-hook-form
    Object.entries(err.fieldErrors).forEach(([field, msgs]) =>
      form.setError(field as any, { message: msgs[0] }),
    );
    return;
  }
  if (err.statusCode === 401) return redirectToLogin();
  toast.error(err.message);
}
```

Nguyên tắc quan trọng:

- Dùng **`code` (mã ổn định)** để FE phân nhánh logic, KHÔNG dựa vào `message` (vì message có thể đổi/đa ngôn ngữ).
- Đính kèm `traceId` để khi user báo lỗi, ta tra log BE đúng request đó.
- Tách lỗi validate theo field (`fieldErrors`) để gắn vào form, tách khỏi lỗi hệ thống chung.

</details>

---

### 1.9. So sánh offset pagination và cursor pagination. Tại sao cursor tốt hơn cho dữ liệu thay đổi liên tục, và FE-BE cần đồng bộ những gì?

<details>
<summary>👉 Xem câu trả lời</summary>

| Tiêu chí | Offset (LIMIT/OFFSET) | Cursor (keyset) |
|----------|----------------------|-----------------|
| Cách hoạt động | `OFFSET 40 LIMIT 20` | Lấy bản ghi sau một con trỏ, ví dụ `WHERE created_at < $cursor` |
| Nhảy tới trang bất kỳ | Được (trang 5, 10...) | Không (chỉ next/prev tuần tự) |
| Hiệu năng khi offset lớn | Chậm dần (DB vẫn quét và bỏ qua N dòng) | Ổn định (dùng index trên cột cursor) |
| Dữ liệu thay đổi liên tục | Bị **trùng/sót item** khi có insert/delete giữa các lần tải | Ổn định, không trùng/sót |

**Tại sao cursor tốt hơn cho dữ liệu thay đổi liên tục:** với offset, nếu có ai chèn 1 bản ghi mới ở đầu danh sách giữa lúc bạn tải trang 1 rồi trang 2, thì item cuối của trang 1 sẽ bị đẩy xuống và **xuất hiện lại** ở đầu trang 2 (duplicate); hoặc khi xóa thì bị **nhảy cóc** mất item. Cursor neo vào giá trị thực của một dòng (ví dụ `created_at` + `id` để phá hòa) nên không bị lệch khi tập dữ liệu dịch chuyển.

```ts
// BE: cursor pagination, dùng (createdAt, id) để ổn định khi trùng createdAt
async listPosts(cursor?: string, limit = 20) {
  const items = await this.prisma.post.findMany({
    take: limit + 1, // lấy dư 1 để biết có trang sau không
    ...(cursor && { cursor: { id: cursor }, skip: 1 }),
    orderBy: [{ createdAt: 'desc' }, { id: 'desc' }],
  });
  const hasNext = items.length > limit;
  const data = hasNext ? items.slice(0, limit) : items;
  return { data, nextCursor: hasNext ? data[data.length - 1].id : null };
}
```

**FE-BE cần đồng bộ những gì:**

- **Hình dạng response**: thống nhất `{ data, nextCursor, hasNext }` để FE không phải đoán.
- **Cách hiểu cursor**: cursor là opaque (FE coi như chuỗi mờ, không tự suy diễn) — BE có quyền đổi cách encode mà không vỡ FE.
- **Thứ tự sắp xếp phải nhất quán** giữa các lần gọi, nếu không cursor mất nghĩa.

FE dùng `useInfiniteQuery` của React Query khớp rất tự nhiên với mô hình này: `getNextPageParam: (last) => last.nextCursor`.

</details>

---

### 1.10. Realtime end-to-end: khi nào dùng WebSocket, khi nào dùng SSE? Mô tả luồng và những điểm cần xử lý ở cả FE lẫn BE.

<details>
<summary>👉 Xem câu trả lời</summary>

| Tiêu chí | WebSocket | SSE (Server-Sent Events) |
|----------|-----------|--------------------------|
| Chiều dữ liệu | Hai chiều (full-duplex) | Một chiều (server → client) |
| Giao thức | `ws://`/`wss://` (nâng cấp từ HTTP) | HTTP thường (`text/event-stream`) |
| Tự reconnect | Tự code (hoặc thư viện như socket.io) | Trình duyệt tự reconnect sẵn |
| Phù hợp | Chat, game, collab, cần client gửi nhiều | Notification, feed, progress, stock ticker |
| Hạ tầng | Cần proxy/load balancer hỗ trợ ws | Chạy trên HTTP/2 dễ scale, qua được nhiều proxy |

Nguyên tắc chọn: **cần client gửi liên tục → WebSocket; chỉ server đẩy xuống → SSE** (đơn giản hơn, tận dụng auto-reconnect của trình duyệt).

```ts
// BE NestJS: WebSocket gateway
@WebSocketGateway({ cors: true })
export class ChatGateway {
  @SubscribeMessage('message')
  handleMessage(@MessageBody() data: { room: string; text: string }, @ConnectedSocket() client: Socket) {
    this.server.to(data.room).emit('message', { text: data.text, at: Date.now() });
  }
}
```

```ts
// FE: SSE đơn giản cho notification
const es = new EventSource('/api/notifications', { withCredentials: true });
es.onmessage = (e) => addNotification(JSON.parse(e.data));
es.onerror = () => {/* trình duyệt tự reconnect; có thể hiện trạng thái offline */};
```

**Điểm cần xử lý ở cả hai phía:**

- **Authentication**: WebSocket không gửi header tùy ý dễ như HTTP; thường truyền token qua cookie (nếu same-site) hoặc query param/handshake auth. SSE đi qua HTTP nên cookie hoạt động bình thường.
- **Reconnect & resume**: khi mất kết nối rồi nối lại, cần cơ chế lấy lại các message bị lỡ (SSE có `Last-Event-ID`; WebSocket tự thiết kế sequence/ack).
- **Backpressure & scale**: khi nhiều instance BE, cần một bộ pub/sub (Redis adapter) để broadcast tới đúng client đang nối ở instance khác.
- **Cleanup phía FE**: hủy subscription khi component unmount để tránh leak; throttle khi message dồn dập.

</details>

---

### 1.11. Tại sao không được đặt business logic quan trọng ở phía client? Cho ví dụ một lỗi thực tế và cách phòng tránh khi đồng bộ validation FE-BE.

<details>
<summary>👉 Xem câu trả lời</summary>

Mọi thứ chạy trên client (giá tiền, quyền hạn, kiểm tra điều kiện nghiệp vụ) đều có thể bị **sửa, bỏ qua hoặc giả mạo** vì người dùng kiểm soát hoàn toàn trình duyệt: họ có thể mở DevTools, sửa request, gọi API trực tiếp bằng `curl`/Postman, bỏ qua hoàn toàn UI. Do đó FE chỉ làm validation để **cải thiện trải nghiệm (UX)**, còn BE phải là **nguồn sự thật bắt buộc (authoritative)**.

Ví dụ lỗi thực tế:

```ts
// SAI: chỉ kiểm tra phía client
// FE
if (user.role === 'admin') showDeleteButton();
// BE không kiểm tra lại -> kẻ tấn công gọi thẳng:
//   DELETE /posts/123  (không qua UI) -> xóa được dù không phải admin!
```

```ts
// SAI: tính giảm giá ở client rồi gửi giá đã giảm lên server
const finalPrice = price * 0.9; // user sửa thành price * 0.0 trong DevTools
await api.checkout({ amount: finalPrice }); // BE tin tưởng -> mua giá 0đ
```

Cách phòng tránh:

- **Luôn enforce ở BE**: kiểm tra quyền trong guard, tính toán giá/giảm giá ở server từ dữ liệu gốc trong DB, không nhận giá từ client.
- **Đồng bộ validation bằng schema dùng chung** (Zod) để FE và BE dùng cùng luật, nhưng hiểu rằng validation FE chỉ là "fail fast cho UX", BE vẫn validate lại độc lập.
- **Không gửi dữ liệu nhạy cảm xuống client** để rồi nhận lại (như giá, role hiệu lực). Gửi định danh (id, mã coupon) và để BE tự tra.

```ts
// ĐÚNG: schema dùng chung, nhưng BE validate lại + enforce nghiệp vụ
// FE: validate nhanh để hiện lỗi form
const result = CheckoutSchema.safeParse(formData);

// BE: validate LẠI + tính giá từ DB, kiểm tra quyền
const dto = CheckoutSchema.parse(req.body);      // không tin client
const coupon = await couponRepo.findValid(dto.couponCode); // tự tra
const amount = computePrice(items, coupon);      // tính ở server
await authz.ensure(user, 'checkout');            // enforce quyền
```

Tư duy cốt lõi: "FE validation là cánh cửa lịch sự, BE validation là ổ khóa". Không bao giờ tin client.

</details>

---

### 1.12. API versioning: khi nào và làm thế nào để version một API mà không làm vỡ client FE đang chạy? So sánh các chiến lược.

<details>
<summary>👉 Xem câu trả lời</summary>

Cần versioning khi phải thực hiện **breaking change** (đổi tên/xóa field, đổi kiểu, đổi ngữ nghĩa, đổi response shape) mà vẫn còn client cũ (app mobile chưa cập nhật, tab đang mở, cache) đang gọi API. Mục tiêu: client cũ tiếp tục chạy trong khi client mới dùng phiên bản mới.

| Chiến lược | Ví dụ | Ưu | Nhược |
|-----------|-------|----|-------|
| **URI versioning** | `/v1/posts`, `/v2/posts` | Rõ ràng, dễ cache/route/debug | "Ô nhiễm" URL; v thực ra không phải tài nguyên |
| **Header versioning** | `Accept: application/vnd.app.v2+json` | URL sạch, đúng tinh thần REST | Khó test bằng tay/curl, dễ bị bỏ sót |
| **Query param** | `/posts?version=2` | Đơn giản | Dễ quên, lẫn với filter |

NestJS hỗ trợ sẵn cả ba qua `enableVersioning`:

```ts
// main.ts
app.enableVersioning({ type: VersioningType.URI, defaultVersion: '1' });

// controller
@Controller('posts')
export class PostsController {
  @Version('1')
  @Get()
  findAllV1() { /* shape cũ */ }

  @Version('2')
  @Get()
  findAllV2() { /* shape mới */ }
}
```

Nguyên tắc giảm thiểu việc phải version:

- **Ưu tiên thay đổi non-breaking**: thêm field mới (client cũ bỏ qua được), không xóa/đổi field đang dùng. Đây gọi là tiến hóa tương thích ngược.
- **Chỉ version khi thực sự breaking.** Mỗi version là gánh nặng bảo trì.
- **Có chính sách deprecate rõ ràng**: trả header `Deprecation`/`Sunset`, thông báo timeline, đo lượng traffic còn dùng version cũ trước khi xóa.
- Phối hợp với FE: FE và BE thống nhất version đang dùng trong API client (codegen theo version cụ thể) để khi BE ra `/v2`, FE chủ động migrate chứ không vỡ ngầm.

Thực tế phổ biến nhất cho web app nội bộ là **URI versioning** vì dễ debug và route nhất.

</details>

## 2. Domain: Analytics, Telemetry và Feature Flags

### 2.1. Vì sao thu thập event analytics ở phía server (server-side) thường đáng tin hơn phía client (client-side)? Khi nào vẫn bắt buộc phải dùng client-side?

<details>
<summary>👉 Xem câu trả lời</summary>

Client-side tracking chạy trong browser/app của người dùng nên dễ bị **mất hoặc sai lệch** vì: ad-blocker và trình duyệt chặn script analytics, người dùng đóng tab trước khi request kịp gửi, mạng chập chờn, hoặc kẻ xấu giả mạo (spoof) event. Server-side tracking phát sinh từ chính backend khi xử lý request thật, nên khó bị chặn, khó giả mạo, và dữ liệu nhất quán hơn (dùng cùng nguồn sự thật với DB).

So sánh nhanh:

| Tiêu chí | Client-side | Server-side |
| --- | --- | --- |
| Bị ad-blocker chặn | Có | Không |
| Độ tin cậy/chống giả mạo | Thấp | Cao |
| Bắt được hành vi UI (hover, scroll, click) | Tốt | Không thấy được |
| Bắt được kết quả nghiệp vụ (payment success, order created) | Không chắc chắn | Chính xác |
| Lộ thông tin nhạy cảm (API key, logic) | Rủi ro cao | An toàn |

**Khi nào vẫn cần client-side:** đo các tương tác chỉ tồn tại ở UI mà server không nhìn thấy — ví dụ scroll depth, thời gian xem một component, A/B test render giao diện, rage-click. Thực tế ở team trưởng thành thường dùng **hybrid**: client-side cho UX/engagement, server-side cho các "money/business event" quan trọng.

Ví dụ phát một event tin cậy từ NestJS sau khi nghiệp vụ thực sự thành công:

```ts
@Post('orders')
async createOrder(@Body() dto: CreateOrderDto, @Req() req) {
  const order = await this.orderService.create(dto);
  // Event phát SAU khi ghi DB thành công => phản ánh sự thật
  this.analytics.track({
    name: 'order_created',
    userId: req.user.id,
    props: { orderId: order.id, amount: order.amount, currency: order.currency },
    occurredAt: new Date(),
  });
  return order;
}
```

Một event "order_created" gửi từ client có thể bị giả mạo hoặc gửi cả khi thanh toán fail; gửi từ server thì chỉ phát khi DB đã commit.

</details>

---

### 2.2. Thiết kế một event schema tốt cần những gì? Cách đặt tên, property và đặc biệt là versioning khi schema thay đổi?

<details>
<summary>👉 Xem câu trả lời</summary>

Một event schema tốt giúp dữ liệu **nhất quán, query được và tiến hoá được** mà không phá báo cáo cũ.

**Đặt tên (naming):**
- Thống nhất một convention, phổ biến là `object_action` ở thì quá khứ: `order_created`, `video_played`, `level_completed`. Tránh tên mơ hồ như `click` hay `event1`.
- Dùng `snake_case` (hoặc một chuẩn duy nhất), tránh trộn lẫn camelCase/kebab.

**Property:**
- Mỗi property có kiểu cố định và ý nghĩa cố định: `amount` luôn là number, không lúc string lúc number.
- Tách rõ thuộc tính của event (`order_id`, `amount`) và thuộc tính của ngữ cảnh chung (user, device, app version) — context thường đính kèm tự động.
- Tránh nhồi PII vào property (xem câu về GDPR).

**Versioning** là phần dễ bị bỏ quên. Khi cần thay đổi:
- **Thay đổi tương thích ngược (thêm property optional):** không cần version mới.
- **Thay đổi phá vỡ (đổi kiểu, đổi ý nghĩa, bỏ field):** nên thêm trường `version` (hoặc `schema_version`) hoặc tạo event mới (`checkout_completed_v2`). Không bao giờ "tái sử dụng" tên cũ với ý nghĩa mới — pipeline và dashboard cũ sẽ hiểu sai dữ liệu.

Ví dụ định nghĩa schema bằng zod để validate trước khi ingest:

```ts
import { z } from 'zod';

const OrderCreatedV1 = z.object({
  name: z.literal('order_created'),
  version: z.literal(1),
  occurredAt: z.string().datetime(),
  userId: z.string(),
  props: z.object({
    orderId: z.string(),
    amount: z.number().nonnegative(),
    currency: z.enum(['USD', 'VND', 'EUR']),
  }),
});
```

Khi đổi `amount` từ "đơn vị đồng" sang "đơn vị cent" (thay đổi ý nghĩa, kiểu vẫn number) -> bắt buộc bump version 2, vì dữ liệu cũ và mới không thể trộn trong cùng một phép tính tổng doanh thu.

</details>

---

### 2.3. "Idempotent event ingestion" là gì? Trong mô hình at-least-once delivery, làm sao chống event bị đếm trùng (dedupe)?

<details>
<summary>👉 Xem câu trả lời</summary>

Phần lớn hệ thống messaging/HTTP retry đảm bảo **at-least-once** (giao ít nhất một lần) chứ không phải exactly-once — nghĩa là một event có thể đến **2 lần trở lên** (client retry vì timeout, queue redeliver vì consumer crash trước khi ack). Nếu cứ thấy event là cộng vào báo cáo thì số liệu sẽ phồng lên.

**Idempotent ingestion** = xử lý cùng một event nhiều lần cho ra cùng kết quả như xử lý đúng một lần. Cách làm chuẩn:

1. **Mỗi event mang một `event_id` duy nhất** (UUID sinh tại điểm phát, không phải tại điểm nhận — để retry vẫn giữ nguyên id).
2. Lưu các `event_id` đã xử lý vào một store có ràng buộc unique (DB unique index, hoặc Redis SET với TTL).
3. Khi nhận event: thử insert id; nếu trùng -> đã xử lý rồi -> bỏ qua (drop), không xử lý lại.

```ts
async ingest(event: AnalyticsEvent) {
  try {
    // unique index trên event_id; INSERT trùng sẽ ném lỗi
    await this.repo.insert({ eventId: event.eventId, payload: event });
  } catch (e) {
    if (this.isUniqueViolation(e)) {
      return; // duplicate -> idempotent drop, KHÔNG đếm lần nữa
    }
    throw e; // lỗi khác -> để retry
  }
  await this.process(event);
}
```

Với throughput rất cao, dùng Redis làm "seen set":

```ts
const isNew = await redis.set(`evt:${event.eventId}`, '1', 'NX', 'EX', 86400);
if (!isNew) return; // đã thấy trong 24h qua -> bỏ
```

**Lưu ý:** dedupe cần một **cửa sổ thời gian** (TTL) hợp lý — nếu nhớ id mãi mãi thì store phình to; nếu cửa sổ quá ngắn, một bản trùng đến muộn (sau TTL) vẫn lọt. Chọn TTL lớn hơn khoảng thời gian retry tối đa của hệ thống.

</details>

---

### 2.4. Feature flag là gì? Giải thích rollout theo phần trăm (%), kill switch và targeting. Triển khai an toàn ở cả frontend (React/Next.js) lẫn backend (NestJS) ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

Feature flag (feature toggle) là cơ chế **bật/tắt hoặc điều khiển một tính năng tại runtime mà không cần deploy lại**. Tách "deploy code" khỏi "release tính năng".

Các kiểu chính:
- **Kill switch (boolean):** bật/tắt cứng một tính năng. Khi tính năng mới gây sự cố, tắt ngay không cần rollback deploy.
- **Rollout theo %:** mở dần cho 5% -> 25% -> 100% người dùng để giảm rủi ro (canary). Quan trọng: việc gán user vào nhóm phải **ổn định (sticky)** — cùng một user luôn nhận cùng kết quả, nếu không UI sẽ "nhấp nháy". Cách làm: hash `userId` thành số 0-99 và so với ngưỡng.
- **Targeting:** bật theo thuộc tính — quốc gia, gói trả phí, beta tester, internal staff.

```ts
// Rollout % sticky: cùng userId luôn cho cùng kết quả
import { createHash } from 'crypto';

function isEnabled(flag: string, userId: string, percentage: number): boolean {
  const h = createHash('sha256').update(`${flag}:${userId}`).digest();
  const bucket = h.readUInt32BE(0) % 100; // 0..99
  return bucket < percentage;
}
```

**Backend (NestJS):** đánh giá flag ở server là nguồn sự thật, đặc biệt cho các tính năng nhạy cảm (không để client tự quyết quyền truy cập). Có thể bọc bằng một guard/service đọc flag từ config service hoặc nhà cung cấp như LaunchDarkly/Unleash.

**Frontend (React/Next.js):** rủi ro lớn nhất là **flicker** và **hydration mismatch**. Nếu client render mặc định OFF rồi mới bật ON sau khi fetch flag, người dùng thấy giao diện nháy. Cách xử lý: với Next.js, đánh giá flag **ở server (Server Component / `getServerSideProps` / middleware)** rồi truyền kết quả xuống để render nhất quán ngay từ HTML đầu tiên.

```tsx
// Next.js App Router - đánh giá ở server, không flicker
export default async function Page() {
  const userId = await getUserId();
  const newCheckout = isEnabled('new_checkout', userId, 25);
  return newCheckout ? <NewCheckout /> : <OldCheckout />;
}
```

**Nguyên tắc vận hành:** flag phải có giá trị mặc định an toàn (nếu service flag down thì fallback về trạng thái cũ/OFF), và phải được dọn dẹp (technical debt) sau khi tính năng đã 100% ổn định — flag tồn đọng lâu ngày làm code khó hiểu và khó test.

</details>

---

### 2.5. Cách thiết kế và đo lường một A/B test cơ bản? Giải thích metric, sample size và statistical significance (ý nghĩa thống kê) một cách thực tế.

<details>
<summary>👉 Xem câu trả lời</summary>

A/B test = chia ngẫu nhiên người dùng thành nhóm **Control (A)** và **Variant (B)**, mỗi nhóm thấy một phiên bản, rồi so sánh một **metric mục tiêu** để xem thay đổi có thực sự tốt hơn hay chỉ là may rủi.

Các thành phần cần làm đúng:

1. **Chọn một primary metric rõ ràng** trước khi chạy (ví dụ conversion rate = số mua / số xem). Không "đào bới" hàng chục metric rồi chọn cái nào đẹp — đó là p-hacking. Nên kèm vài **guardrail metric** (ví dụ tỉ lệ lỗi, thời gian load) để chắc rằng variant không làm hỏng thứ khác.
2. **Phân nhóm ngẫu nhiên và sticky:** dùng cùng kỹ thuật hash `userId` như feature flag, đảm bảo một user luôn ở một nhánh.
3. **Sample size & thời gian chạy:** cần đủ lượng người dùng để phát hiện được chênh lệch mong đợi. Kết thúc test quá sớm (peeking) dễ ra kết luận sai. Thường chạy đủ một bội số của chu kỳ tuần để tránh hiệu ứng ngày trong tuần.
4. **Statistical significance:** thường dùng ngưỡng p-value < 0.05, nghĩa là **xác suất chênh lệch quan sát được chỉ do ngẫu nhiên là dưới 5%**. p-value nhỏ -> tin rằng khác biệt là thật. Lưu ý: significance (có thật không) khác với **effect size** (lớn cỡ nào) — chênh lệch 0.1% có thể "significant" với mẫu khổng lồ nhưng vô nghĩa về kinh doanh.

So sánh hai cách kết thúc sai và đúng:

| Tình huống | Vấn đề |
| --- | --- |
| Dừng ngay khi B vượt A vài giờ đầu | "Peeking" -> dương tính giả, kết quả không lặp lại |
| Đổi metric mục tiêu giữa chừng vì A đang thắng | Thiên lệch chủ quan, kết luận không đáng tin |
| Định trước metric + sample size, chạy đủ thời gian, rồi mới đọc kết quả | Đúng chuẩn |

Việc tính p-value/significance nên dùng thư viện thống kê hoặc công cụ chuyên dụng (ví dụ scipy, hoặc nền tảng experimentation), engineer không nên tự "ước lượng bằng mắt".

</details>

---

### 2.6. So sánh batch và stream processing trong một data pipeline analytics nhẹ. Vai trò của queue (Kafka/SQS/RabbitMQ) ở đâu? Khi nào chọn cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Một pipeline analytics điển hình: **producer (app phát event) -> queue/buffer -> processor -> data store (warehouse) -> dashboard**.

**Queue (Kafka, SQS, RabbitMQ)** đứng giữa producer và processor để **tách rời (decouple)** và **hấp thụ tải đỉnh (buffering)**. Lợi ích:
- App backend chỉ cần `enqueue` event rất nhanh rồi trả response — không chờ ghi warehouse (giảm latency cho user).
- Khi traffic tăng đột biến, queue giữ event lại, processor xử lý theo nhịp của nó (back-pressure), tránh sập DB.
- Nếu processor chết, event vẫn nằm trong queue -> không mất dữ liệu (kết hợp at-least-once + dedupe ở câu 2.3).

**Batch vs Stream:**

| Tiêu chí | Batch | Stream |
| --- | --- | --- |
| Cách xử lý | Gom theo lô, chạy định kỳ (mỗi giờ/ngày) | Xử lý từng event gần real-time |
| Latency | Cao (phút/giờ) | Thấp (giây) |
| Độ phức tạp vận hành | Đơn giản hơn | Phức tạp hơn (state, ordering) |
| Chi phí | Thường rẻ hơn | Thường cao hơn |
| Hợp với | Báo cáo doanh thu cuối ngày, tổng hợp lịch sử | Cảnh báo gian lận, live dashboard, kill switch theo lỗi |

**Khi nào chọn cái nào:** nếu chỉ cần báo cáo "ngày hôm qua bao nhiêu order" thì batch là đủ và rẻ. Nếu cần phản ứng tức thì (phát hiện crash spike, theo dõi live concurrent players) thì stream. Nhiều hệ thống dùng **cả hai (lambda architecture)**: stream cho dashboard real-time, batch để tính lại chính xác và sửa sai.

Ví dụ producer nhẹ trong NestJS chỉ đẩy vào queue:

```ts
async track(event: AnalyticsEvent) {
  // Không ghi thẳng DB trong request handler -> đẩy vào queue
  await this.queue.add('analytics_event', event, {
    attempts: 5,
    backoff: { type: 'exponential', delay: 1000 },
  });
}
```

Với "pipeline nhẹ" cho dự án nhỏ, không cần Kafka ngay — một queue như BullMQ (Redis) hoặc SQS đã đủ để decouple và retry; chỉ nâng lên Kafka khi cần throughput rất lớn và replay log.

</details>

---

### 2.7. Làm sao tránh lộ PII trong log/analytics và tôn trọng consent theo GDPR? Vì sao không nên log dữ liệu cá nhân, và xử lý "right to be forgotten" thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**PII (Personally Identifiable Information)** là dữ liệu nhận dạng được cá nhân: email, số điện thoại, địa chỉ, IP, payment info, đôi khi cả `userId` thật. Log và analytics là nơi PII hay vô tình rò rỉ nhất (log nguyên cả request body, đính email vào event property).

Vì sao nguy hiểm: log thường được lưu lâu, sao chép sang nhiều hệ thống (monitoring, third-party analytics), nhiều người truy cập -> bề mặt rò rỉ rộng. GDPR yêu cầu xử lý dữ liệu cá nhân phải có **cơ sở pháp lý (consent)**, **giới hạn mục đích**, **giảm thiểu dữ liệu (data minimization)** và cho người dùng quyền xoá.

Các biện pháp thực tế:

1. **Không log PII ngay từ đầu (data minimization).** Redact/mask ở tầng logger trước khi ghi.

```ts
const REDACT = ['password', 'email', 'phone', 'token', 'authorization', 'card'];
function sanitize(obj: Record<string, any>) {
  for (const k of Object.keys(obj)) {
    if (REDACT.includes(k.toLowerCase())) obj[k] = '[REDACTED]';
  }
  return obj;
}
logger.info('order_created', sanitize({ ...payload }));
```

2. **Pseudonymisation:** trong analytics dùng id giả/hash thay vì email thật, để vẫn join được mà không phơi bày danh tính.
3. **Consent gating:** chỉ bắn analytics/tracking khi người dùng đã đồng ý. Trên frontend, không load script tracking trước khi có consent.

```ts
if (consent.analytics) {
  analytics.track('page_view', { path });
}
```

4. **Right to be forgotten / right to access:** phải có khả năng **xoá hoặc ẩn danh** dữ liệu của một người khi họ yêu cầu. Vì vậy nên gắn dữ liệu với một `userId` để có thể tìm và xoá theo cascade, thay vì rải PII khắp các free-text log không thể truy vết.
5. **Data retention:** đặt TTL cho log/event, không giữ vô thời hạn.

So sánh sai/đúng:

| Sai | Đúng |
| --- | --- |
| `logger.info(JSON.stringify(req.body))` | Sanitize rồi mới log, chỉ giữ field cần |
| Event chứa `email`, `phone` | Chỉ chứa `userId` đã hash, để join phân tích |
| Track ngay khi load trang | Track sau khi người dùng đồng ý cookie/consent |
| Giữ log mãi mãi | Đặt retention policy + cơ chế xoá theo userId |

</details>

---

### 2.8. Trong bối cảnh game tools/publishing, hãy thiết kế telemetry để đo hành vi người chơi và dựng một funnel (ví dụ onboarding/level progression). Cần lưu ý gì đặc thù?

<details>
<summary>👉 Xem câu trả lời</summary>

Telemetry người chơi (player telemetry) là các event mô tả hành trình chơi: cài đặt, mở game, hoàn thành tutorial, qua level, mua hàng, churn. Mục tiêu là **dựng funnel** để thấy người chơi rơi rớt ở đâu và tối ưu chỗ đó.

**Funnel** là chuỗi bước có thứ tự; mỗi bước đo "bao nhiêu % người ở bước trước đi tiếp được". Ví dụ onboarding:

```text
install -> first_open -> tutorial_started -> tutorial_completed -> level_1_completed -> first_purchase
   100%       82%             70%                  55%                    40%                6%
```

Nhìn vào đây thấy ngay tụt mạnh ở `tutorial_completed` (70% -> 55%) -> tutorial quá khó/dài, cần sửa.

Thiết kế event cho game (áp dụng schema ở câu 2.2):

```ts
// Mỗi bước funnel = một event tên rõ ràng, có thứ tự
analytics.track('level_completed', {
  level: 3,
  durationSec: 142,
  attempts: 4,
  result: 'win',
  // KHÔNG đính tên/email; chỉ playerId đã hash
});
```

Lưu ý đặc thù game/publishing:

1. **Offline & buffering:** game mobile hay mất mạng giữa chừng. Client cần **buffer event local rồi gửi batch khi có mạng**, kèm `occurredAt` (thời điểm thật) tách khỏi `receivedAt` (thời điểm server nhận) — nếu không, biểu đồ theo thời gian sẽ sai.
2. **Idempotency/dedupe (câu 2.3):** vì gửi lại sau khi offline rất dễ trùng, mỗi event phải có `event_id` để dedupe.
3. **Server-side cho money/anti-cheat (câu 2.1):** event "first_purchase", "currency_spent" phải xác nhận ở server, không tin client (chống gian lận đếm doanh thu/đồ ảo).
4. **Session & cohort:** gắn `session_id` để phân tích một phiên chơi; phân tích theo **cohort** (nhóm cài cùng tuần) để đo retention D1/D7/D30 — chỉ số sống còn của game.
5. **Privacy (câu 2.7):** nhiều game có người chơi nhỏ tuổi -> ràng buộc COPPA/GDPR chặt hơn, càng phải tránh PII và tôn trọng consent.

Funnel + cohort retention là hai báo cáo "ăn tiền" nhất mà publishing team thường yêu cầu, nên telemetry cần được thiết kế ngay từ đầu để trả lời được hai câu đó.

</details>

## 3. Cách làm việc và Tư duy sản phẩm

### 3.1. "MVP-first" và "bias toward shipping" nghĩa là gì? Khi nhận một feature mới, bạn ưu tiên giao một bản chạy được nhanh như thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

MVP-first nghĩa là chia feature thành phiên bản nhỏ nhất tạo ra giá trị thật cho người dùng, ship nó trước, rồi mới lặp lại để hoàn thiện. "Bias toward shipping" là thiên hướng đưa code ra production sớm thay vì đánh bóng vô tận trong nhánh local — vì code chỉ tạo giá trị khi người dùng dùng được, và feedback thật từ production đáng giá hơn mọi phỏng đoán.

Quy trình thực tế khi nhận một feature, ví dụ "thêm lọc task theo assignee" trong một Jira clone:

1. Xác định luồng giá trị cốt lõi: người dùng chọn một assignee và thấy danh sách task được lọc. Đó là MVP.
2. Cắt phần không cần cho lần ship đầu: lọc nhiều assignee cùng lúc, lưu bộ lọc, share URL bộ lọc — để vào lần sau.
3. Ship dọc (vertical slice): làm xuyên suốt từ DB query -> API -> UI cho đúng một luồng, thay vì làm xong toàn bộ backend rồi mới đụng frontend.

```ts
// MVP: lọc theo MOT assignee, đủ chạy và giao giá trị ngay
@Get('tasks')
findTasks(@Query('assigneeId') assigneeId?: string) {
  return this.prisma.task.findMany({
    where: assigneeId ? { assigneeId } : {},
  });
}

// Phiên bản sau (KHONG làm ngay lần đầu): nhiều assignee, lưu filter, phân trang
// @Get('tasks')
// findTasks(@Query() q: TaskFilterDto) { ... }
```

So sánh tư duy:

| Tư duy "đánh bóng trước" | Tư duy MVP-first |
|---|---|
| Làm hết mọi case rồi mới ship | Ship luồng chính, bổ sung edge case theo nhu cầu thật |
| Feedback đến muộn, rework lớn | Feedback đến sớm, điều chỉnh rẻ |
| Rủi ro làm sai thứ không ai cần | Validate giá trị sớm |

Lưu ý: MVP-first KHONG có nghĩa là làm ẩu. Bản MVP vẫn phải đúng, an toàn (auth, validation input), và không phá luồng hiện có. "Nhỏ" là về phạm vi tính năng, không phải về chất lượng nền tảng.

</details>

---

### 3.2. "Đủ tốt để ship" (good enough to ship) là tới đâu? Bạn cân bằng tốc độ và chất lượng thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

"Đủ tốt để ship" không phải một con số cố định mà phụ thuộc vào rủi ro và độ đảo ngược của thay đổi. Nguyên tắc tôi dùng: phân loại quyết định theo mức độ "có thể quay đầu được không".

- Thay đổi dễ đảo ngược + rủi ro thấp (đổi label nút, thêm filter UI): ngưỡng "đủ tốt" thấp, ship nhanh, sửa sau nếu cần.
- Thay đổi khó đảo ngược + rủi ro cao (schema migration, đổi API public, xử lý thanh toán): ngưỡng cao, cần test kỹ, review kỹ, có thể feature flag.

Một checklist tối thiểu của tôi cho "đủ tốt để ship" với một feature backend:

```
[x] Đúng luồng chính (happy path) + các edge case có khả năng xảy ra cao
[x] Validation input + xử lý lỗi không làm crash / không lộ thông tin nhạy cảm
[x] Auth/authorization đúng (không ship lỗ hổng quyền)
[x] Có test cho logic nghiệp vụ quan trọng
[x] Không phá vỡ chức năng cũ (regression)
[ ] Tối ưu performance khi chưa có bằng chứng là vấn đề  <- để sau (YAGNI)
[ ] Hỗ trợ mọi edge case hiếm gặp  <- để sau, ship trước
```

Cách cân bằng: tôi tách "phải có trước khi ship" (đúng, an toàn, không regression) khỏi "có thì tốt" (đánh bóng, tối ưu sớm, mở rộng giả định). Nếu thấy mình đang trau chuốt phần "có thì tốt" trong khi luồng chính đã chạy ổn, đó là tín hiệu nên ship rồi lặp lại.

Quan trọng: khi tôi cố ý ship một thứ "đủ tốt" mà còn nợ kỹ thuật, tôi ghi rõ nó ra (comment `// TODO`, ticket tech-debt, hoặc note trong PR) để team thấy được đánh đổi, thay vì giấu đi. Tốc độ có chủ đích khác hẳn với ẩu.

</details>

---

### 3.3. Over-engineering là gì? Áp dụng YAGNI và KISS trong code thực tế ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

Over-engineering là xây dựng sự linh hoạt / trừu tượng cho những nhu cầu chưa tồn tại — tạo ra độ phức tạp phải trả phí ngay nhưng lợi ích chỉ là giả định.

- YAGNI (You Aren't Gonna Need It): đừng thêm khả năng cho tới khi thật sự cần.
- KISS (Keep It Simple, Stupid): chọn giải pháp đơn giản nhất giải quyết được vấn đề hiện tại.

Ví dụ over-engineering điển hình ở backend — tạo abstraction layer cho một thứ chỉ có một implementation:

```ts
// OVER-ENGINEERED: factory + interface + strategy cho 1 provider email duy nhất
interface IEmailProvider { send(to: string, body: string): Promise<void>; }
class SendgridProvider implements IEmailProvider { /* ... */ }
class EmailProviderFactory {
  static create(type: 'sendgrid' | 'ses' | 'mailgun'): IEmailProvider {
    // chỉ mới có sendgrid, phần còn lại là phỏng đoán
  }
}

// KISS: làm thẳng, refactor khi THỰC SỰ cần provider thứ hai
@Injectable()
class EmailService {
  async send(to: string, body: string) {
    await this.sendgrid.send({ to, html: body });
  }
}
```

Ví dụ ở frontend — không cần đưa state vào global store (Redux/Zustand) khi nó chỉ là state cục bộ của một component:

```tsx
// Over-engineered: state mở/đóng của 1 modal đẩy lên Redux
// KISS: dùng useState ngay tại component
const [open, setOpen] = useState(false);
```

Khi nào trừu tượng hóa là HỢP LÝ chứ không phải over-engineering? Quy tắc "rule of three": chấp nhận lặp lại 1-2 lần; tới lần thứ 3 khi pattern đã rõ thì mới tách abstraction. Trừu tượng hóa đúng dựa trên pattern đã quan sát được, over-engineering dựa trên tương lai tưởng tượng.

Bẫy thường gặp: nhầm "đơn giản" với "ngắn". Code dùng quá nhiều mẹo cho ngắn (one-liner khó đọc) cũng vi phạm KISS. Đơn giản là dễ hiểu và dễ sửa, không phải ít dòng nhất.

</details>

---

### 3.4. Khi gặp một vấn đề kỹ thuật lạ, quy trình tự tháo gỡ (self-sufficient) của bạn ra sao trước khi hỏi người khác?

<details>
<summary>👉 Xem câu trả lời</summary>

Self-sufficient không có nghĩa là không bao giờ hỏi, mà là tôn trọng thời gian của team: tự khai thác hết các kênh thông tin trong một khung thời gian hợp lý, rồi mới hỏi với một câu hỏi đã được nén lại đủ ngữ cảnh.

Quy trình của tôi, đại khái theo thứ tự:

1. Đọc kỹ error message và stack trace — phần lớn lỗi tự nói ra nguyên nhân. Đọc dòng đầu tiên có file của chính mình trong stack.
2. Tái hiện tối thiểu (minimal reproduction): cô lập vấn đề để loại biến số.
3. Đọc docs / source chính thức của thư viện đang dùng (không phải blog cũ). Đôi khi đọc thẳng source code trong `node_modules` là nhanh nhất.
4. Dùng AI (Claude/Copilot) để tăng tốc tạo giả thuyết, giải thích error, gợi ý hướng — nhưng tôi verify lại (xem câu 3.8).
5. Thử nghiệm có kiểm soát: thay đổi MỘT biến mỗi lần, dùng log/debugger để thu hẹp.
6. Tìm trên codebase nội bộ xem đã có ai giải quyết pattern tương tự chưa (`git log`, search trong repo).

```bash
# Vài thao tác cô lập / tự kiểm tra hay dùng
node -e "console.log(require('some-lib/package.json').version)"  # đúng version chưa?
git log --oneline -- path/to/file   # ai từng đụng file này, vì sao
git bisect start                    # tìm commit gây regression
```

Tôi đặt một "time-box": ví dụ 30-60 phút cho lỗi không chặn (non-blocking). Nếu vượt time-box mà chưa ra, tôi escalate — vì lúc đó cái giá của việc kẹt một mình lớn hơn cái giá của việc làm phiền người khác. Ngược lại, nếu vấn đề đang CHẶN cả team (block release), tôi escalate gần như ngay lập tức, không cố tỏ ra tự lực.

Điều then chốt: khi hỏi, tôi đưa kèm "tôi đã thử X, Y, Z và kết quả là...", chứ không hỏi "cái này lỗi, fix sao?". Câu hỏi tốt thể hiện rằng mình đã tự tháo gỡ trước.

</details>

---

### 3.5. Bạn ước lượng (estimate) và chia nhỏ một feature lớn thành các task như thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Nguyên tắc của tôi: chia nhỏ tới mức mỗi task có thể hoàn thành trong khoảng nửa ngày đến một ngày, có định nghĩa "xong" rõ ràng, và lý tưởng là mỗi task tạo ra một thay đổi có thể ship/merge độc lập (vertical slice).

Quy trình:

1. Phân rã theo luồng giá trị, không theo tầng kỹ thuật. Tôi tránh chia kiểu "task 1: làm hết DB, task 2: làm hết API, task 3: làm hết UI" vì không có gì merge được cho tới cuối. Thay vào đó chia theo từng luồng người dùng đi xuyên các tầng.
2. Đánh dấu phụ thuộc và rủi ro/ẩn số. Task nào có nhiều điều chưa biết thì ước lượng kém tin cậy nhất.
3. Spike cho phần mơ hồ: dành một task nhỏ time-box để nghiên cứu/thử nghiệm trước khi ước lượng phần phụ thuộc vào nó.
4. Ước lượng và cộng buffer cho ẩn số, không cộng buffer cho thứ đã rõ.

Ví dụ chia nhỏ feature "bình luận trên task":

```
Feature: Comment trên task
├─ [Slice 1] Tạo comment + hiển thị list (happy path)
│   - schema + migration bảng comment        (~0.5d)
│   - API POST/GET comment + test            (~0.5d)
│   - UI form + list cơ bản                  (~0.5d)
├─ [Slice 2] Sửa/Xóa comment                 (~1d)
├─ [Slice 3] Mention @user + notify          (~1.5d, có ẩn số -> spike trước)
└─ [Slice 4] Đánh bóng: loading/empty/error  (~0.5d)
```

Về độ chính xác ước lượng: tôi nói rõ mức độ tự tin thay vì đưa một con số giả vờ chắc chắn. Ví dụ: "Slice 1-2 tôi khá chắc ~2.5 ngày; Slice 3 có ẩn số về hệ thống notify, tôi cần spike nửa ngày rồi mới ước lượng được." Ước lượng là dự báo có sai số, không phải lời hứa.

Một thói quen quan trọng: cập nhật lại ước lượng sớm khi phát hiện mình sai. Nếu đến giữa task thấy sẽ trễ, tôi báo NGAY chứ không đợi tới hạn (xem câu 3.6) — communicate sớm giúp Tech Manager còn xoay xở.

</details>

---

### 3.6. Bạn cộng tác với Tech Manager và đồng đội ra sao? Khi nào thì cần communicate sớm và hỏi đúng câu hỏi?

<details>
<summary>👉 Xem câu trả lời</summary>

Triết lý của tôi: communicate sớm và chủ động luôn rẻ hơn việc để bất ngờ nổ ra muộn. Một thông tin "tôi sẽ trễ 2 ngày" nói hôm nay giá trị hơn nhiều so với nói vào đúng hạn.

Khi nào tôi chủ động liên lạc ngay (không đợi standup):

- Khi phát hiện sẽ trễ deadline, hoặc scope lớn hơn dự kiến.
- Khi gặp blocker mà mình không tự gỡ được trong time-box (xem 3.4).
- Khi requirement mơ hồ hoặc mâu thuẫn — hỏi để làm rõ TRƯỚC khi code, vì code sai do hiểu sai yêu cầu là loại rework đắt nhất.
- Khi phát hiện một quyết định của mình ảnh hưởng tới người khác (đổi API contract, đổi schema dùng chung).

Cách hỏi "đúng câu hỏi" — tôi cố biến câu hỏi mở thành câu hỏi đóng kèm phương án và đánh đổi:

```
KÉM:   "Em nên làm phần search thế nào?"  (đẩy toàn bộ việc suy nghĩ sang người khác)

TỐT:   "Phần search em đang phân vân 2 hướng:
        (A) filter ở DB bằng SQL ilike — đơn giản, nhưng chậm khi data lớn.
        (B) tích hợp full-text search — mạnh hơn nhưng tốn hạ tầng.
        Hiện data ~vài nghìn record nên em nghiêng về (A) cho lần này,
        đúng hướng không ạ?"
```

Câu hỏi tốt cho người được hỏi thấy: bối cảnh, các lựa chọn, đánh đổi, và đề xuất của mình. Họ chỉ cần xác nhận hoặc chỉnh, thay vì phải tự đào từ đầu.

Với đồng đội: tôi viết PR description rõ ràng (mục tiêu, cách tiếp cận, phần cần chú ý khi review), tôn trọng quy ước chung của team, và chủ động sync khi công việc của tôi chạm vào phần của người khác. Cộng tác tốt là làm cho người khác dễ làm việc với mình.

</details>

---

### 3.7. Kể về một lần bạn nhận feedback khó nghe về code/cách làm của mình. Bạn xử lý ra sao? Và bạn đưa feedback cho người khác thế nào?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Đây là câu hành vi, trả lời theo STAR. Một mẫu (hãy thay bằng câu chuyện thật của bạn):

- Situation: Trong một dự án (ví dụ một API NestJS), tôi mở PR cho một module xử lý nghiệp vụ. Reviewer (một senior) để lại comment khá thẳng rằng cách tôi xử lý transaction có thể gây race condition và phần logic nhét hết vào controller là sai tầng kiến trúc.
- Task: Phản ứng đầu tiên của tôi là hơi tự ái vì đã bỏ nhiều công. Nhưng nhiệm vụ thật sự của tôi là ra code đúng, không phải bảo vệ cái tôi.
- Action:
  - Tôi tách cảm xúc khỏi nội dung: feedback nói về code, không phải về con người tôi.
  - Tôi hỏi lại để hiểu rõ thay vì phản bác ngay: "Anh có thể chỉ giúp em kịch bản cụ thể nào dẫn tới race condition không?" — và đúng là có một kịch bản tôi đã bỏ sót.
  - Tôi refactor: chuyển logic xuống service layer, bọc trong transaction của Prisma, và thêm test cho kịch bản tranh chấp đó.
  - Tôi cảm ơn reviewer công khai trong PR.
- Result: PR được merge, tôi tránh được một bug khó tái hiện trên production, và quan trọng hơn là tôi nhớ luôn pattern này cho các lần sau.

Phần "đưa feedback cho người khác" — nguyên tắc của tôi:

- Phê bình code, không phê bình người: "đoạn này có thể null khi...", chứ không phải "sao lại viết thế này".
- Phân biệt rõ "phải sửa" (bug, lỗ hổng) với "đề xuất / sở thích" — tôi đánh dấu nit (nitpick) cho loại sau để người nhận biết cái nào không chặn merge.
- Hỏi thay vì ra lệnh khi chưa chắc: "Mình có cân nhắc dùng X ở đây không? Vì...". Để mở khả năng là mình đang thiếu ngữ cảnh.
- Khen phần làm tốt, không chỉ soi lỗi. Feedback cân bằng dễ được tiếp nhận hơn.

Thông điệp cốt lõi: tôi xem code review là cơ chế nâng chất lượng chung và là kênh học hỏi hai chiều, không phải đấu trường thắng-thua.

</details>

---

### 3.8. "Làm việc AI-native nhưng có phán đoán" nghĩa là gì? Khi nào bạn tin output của AI và khi nào bắt buộc verify?

<details>
<summary>👉 Xem câu trả lời</summary>

AI-native nghĩa là mặc định dùng AI (Claude, Copilot...) như một công cụ tăng tốc trong luồng làm việc hàng ngày: scaffold code, viết test, giải thích error, refactor, khám phá API lạ. "Có phán đoán" nghĩa là tôi vẫn là người chịu trách nhiệm cuối cùng cho code — AI không bao giờ là cái cớ cho một bug. Tôi xem AI như một junior cộng sự rất nhanh nhưng đôi khi tự tin sai (hallucinate).

Nguyên tắc verify của tôi dựa trên rủi ro và độ dễ kiểm chứng:

| Trường hợp | Mức tin cậy | Hành động |
|---|---|---|
| Boilerplate, scaffold, đổi cú pháp | Cao | Đọc lướt + để test/compiler bắt lỗi |
| Code có logic nghiệp vụ | Trung bình | Đọc kỹ từng dòng, tự viết/kiểm test |
| API/hàm thư viện AI "nhớ" ra | Thấp | Đối chiếu docs chính thức — AI hay bịa tên hàm/tham số |
| Bảo mật, auth, query tiền/quyền | Rất thấp | Tự suy luận + review người + test kỹ |
| Migration/lệnh phá hủy dữ liệu | Rất thấp | Không bao giờ chạy mù, đọc và hiểu từng dòng |

Các dấu hiệu bắt buộc tôi dừng và verify:

- AI gọi một hàm/tham số mà tôi chưa từng thấy -> kiểm tra docs ngay, rất có thể là hallucination.
- Code "trông đúng" nhưng tôi không giải thích được vì sao nó đúng -> chưa được merge cho tới khi tôi hiểu.
- Output liên quan đến security, dữ liệu thật, hoặc thứ khó đảo ngược.

```ts
// AI gợi ý — nhìn hợp lý nhưng tôi PHẢI verify:
// 1. Prisma có method findUniqueOrThrow đúng tên này không? -> check docs (có thật)
// 2. Logic phân quyền: AI bỏ qua case user xem task của project mình KHONG thuộc?
const task = await this.prisma.task.findUniqueOrThrow({ where: { id } });
// -> tôi tự thêm kiểm tra quyền, KHONG tin AI đã lo phần security
if (!userCanAccessProject(user, task.projectId)) throw new ForbiddenException();
```

Ranh giới rõ ràng: tôi để AI tăng tốc phần cơ học và phần khám phá, nhưng tôi không bao giờ outsource phán đoán kỹ thuật và trách nhiệm. Code vào PR của tôi là code tôi đã hiểu và đứng tên, dù ai (hay cái gì) viết ra bản nháp đầu tiên.

</details>

---

### 3.9. "Own một feature end-to-end" nghĩa là gì? Trách nhiệm của bạn trải dài tới đâu?

<details>
<summary>👉 Xem câu trả lời</summary>

Own end-to-end nghĩa là tôi chịu trách nhiệm cho feature từ lúc hiểu yêu cầu cho tới khi nó chạy ổn định trên production và sau đó — không phải chỉ "viết xong code rồi quăng qua tường" cho QA hay người khác.

Trong một dự án fullstack, vòng đời một feature tôi own gồm:

1. Làm rõ yêu cầu: hỏi để hiểu vấn đề người dùng thật sự, không chỉ nhận spec mặt chữ.
2. Thiết kế: chọn cách tiếp cận, cân nhắc đánh đổi, sync nếu chạm phần dùng chung.
3. Triển khai xuyên tầng: DB/schema -> API backend (NestJS) -> UI frontend (React/Next.js). Một người, một luồng xuyên suốt giúp tránh "khoảng hở" giữa FE và BE.
4. Chất lượng: tự viết test, tự kiểm thử các edge case, xử lý loading/empty/error ở UI.
5. Đưa ra production: PR, review, theo dõi quá trình deploy.
6. Sau khi ship: quan sát log/metrics/error tracking xem feature có hoạt động đúng với người dùng thật không, sửa nếu có sự cố.
7. Đóng vòng: cập nhật docs nếu cần, để lại đủ thông tin để người sau bảo trì được.

```
Own end-to-end = chịu trách nhiệm cả chuỗi này:
Hiểu yêu cầu -> Thiết kế -> Code (DB/BE/FE) -> Test -> Ship -> Theo dõi prod -> Bảo trì
                                                                        ▲
                                  KHONG dừng ở "code đã chạy trên máy em"
```

Một biểu hiện cụ thể của ownership: khi feature của tôi gây lỗi production lúc 2 giờ chiều, tôi không nói "phần đó qua bên kia rồi"; tôi là người đầu tiên nhảy vào điều tra. Ownership đo bằng việc bạn nhận trách nhiệm khi có sự cố, không chỉ nhận công khi mọi thứ êm.

Lưu ý cân bằng: own không phải là làm tất cả một mình và ôm khư khư. Tôi vẫn nhờ review, vẫn hỏi chuyên gia về phần ngoài chuyên môn (ví dụ devops, design). Own nghĩa là tôi đảm bảo feature về đích, kể cả khi phải kéo người khác vào — chứ không phải từ chối giúp đỡ.

</details>

---

### 3.10. Bạn hình dung lộ trình phát triển của mình thế nào: từ own một feature, tới lead một project, rồi mentor người khác?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Câu này hỏi về định hướng và growth mindset — có thể trả lời bằng STAR dựa trên một trải nghiệm có thật, hoặc trình bày lộ trình kèm ví dụ cụ thể. Một khung tham khảo:

- Situation: Tôi bắt đầu ở vai trò own từng feature riêng lẻ — chịu trách nhiệm trọn vẹn một mảnh chức năng end-to-end.
- Task: Mục tiêu phát triển của tôi là mở rộng phạm vi ảnh hưởng: từ "làm tốt việc của mình" sang "làm cho cả nhóm làm tốt hơn".
- Action — ba nấc tôi nhìn thấy và những gì cần thay đổi ở mỗi nấc:
  - Own feature: thành thạo kỹ thuật, ship đáng tin cậy, giao tiếp tốt với manager. Đây là nền tảng, không có nó thì không lên được nấc sau.
  - Lead một project/epic: chuyển trọng tâm từ "code của tôi" sang "kết quả của cả mảng". Việc mới gồm: chia nhỏ và phân công công việc, định nghĩa kiến trúc/quy ước chung, gỡ blocker cho người khác, ra quyết định đánh đổi ở quy mô lớn hơn, và communicate với stakeholder. Ở đây tôi học cách buông bớt việc tự tay code để tạo đòn bẩy qua người khác.
  - Mentor: nhân rộng năng lực bằng cách nâng người khác — review code mang tính dạy chứ không chỉ bắt lỗi, pair programming, viết tài liệu/chuẩn chung, tạo môi trường an toàn để người mới dám hỏi và dám sai. Thước đo thành công đổi từ "tôi giải được vấn đề" sang "team tôi tự giải được vấn đề khi không có tôi".
- Result: Mục tiêu cuối là tăng dần "bán kính ảnh hưởng" — đóng góp của tôi không còn giới hạn ở số dòng code mình viết, mà ở chất lượng và tốc độ của cả nhóm.

Một điểm tôi muốn nhấn mạnh để tránh hiểu lầm: lên cao hơn không có nghĩa là rời bỏ kỹ thuật. Tôi vẫn muốn giữ tay nghề (đọc code, đủ sâu để ra quyết định kiến trúc đúng), chỉ là dùng nó để tạo đòn bẩy thay vì chỉ để tự sản xuất. Và growth không tuyến tính cứng — ngay từ khi còn own feature tôi đã có thể bắt đầu mentor nhẹ (giúp người mới onboard) và lead nhẹ (dẫn dắt một slice nhỏ). Tôi tích lũy kỹ năng của nấc sau từ sớm, thay vì đợi đến khi được "phong chức".

</details>

---

> 📌 *Vai trò fullstack đánh giá cao người nhìn được bức tranh **end-to-end**: từ UI → API → DB → deploy, và biết **ship nhanh** mà vẫn kiểm soát chất lượng.*
> Hãy gắn câu trả lời với một feature thật bạn từng làm trọn vẹn cả hai đầu. 🚀
