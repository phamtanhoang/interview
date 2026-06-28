# 🤝 Bộ câu hỏi phỏng vấn: Hành vi, Văn hóa & Cách làm việc (Behavioral)

> Gom các câu **behavioral / culture / soft-skill / ways-of-working** vốn nằm rải ở `frontend-`, `backend-`, `fullstack-` và `ai-interview.md` về **một chỗ** cho dễ ôn.
> Tổ chức **theo chủ đề** (không theo stack); các câu gần trùng giữa FE/BE đã được **gộp** lại, giữ cả ví dụ Frontend lẫn Backend để bạn linh hoạt theo vòng phỏng vấn.
> Câu hành vi nên trả lời theo cấu trúc **STAR** (Situation → Task → Action → Result) và gắn với **ví dụ thật** từ dự án của bạn (vd Jira Clone).

## 📚 Mục lục

1. [Cách làm việc và Tư duy sản phẩm](#1-cách-làm-việc-và-tư-duy-sản-phẩm)
2. [Cộng tác và Giao tiếp](#2-cộng-tác-và-giao-tiếp)
3. [Ownership, Sự cố và Sai lầm](#3-ownership-sự-cố-và-sai-lầm)
4. [Ưu tiên và Áp lực](#4-ưu-tiên-và-áp-lực)
5. [Học hỏi và Phát triển](#5-học-hỏi-và-phát-triển)
6. [Làm việc với AI](#6-làm-việc-với-ai)

---

## 1. Cách làm việc và Tư duy sản phẩm

### 1.1. "MVP-first" và "bias toward shipping" nghĩa là gì? Khi nhận một feature mới, bạn ưu tiên giao một bản chạy được nhanh như thế nào?

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

### 1.2. "Đủ tốt để ship" (good enough to ship) là tới đâu? Bạn cân bằng tốc độ và chất lượng thế nào?

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

### 1.3. Over-engineering là gì? Áp dụng YAGNI và KISS trong code thực tế ra sao?

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

### 1.4. Bạn ước lượng (estimate) và chia nhỏ một feature lớn thành các task như thế nào?

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

### 1.5. Khi gặp một vấn đề kỹ thuật lạ, quy trình tự tháo gỡ (self-sufficient) của bạn ra sao trước khi hỏi người khác?

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

## 2. Cộng tác và Giao tiếp

### 2.1. Bạn cộng tác với Tech Manager và đồng đội ra sao? Khi nào thì cần communicate sớm và hỏi đúng câu hỏi?

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

### 2.2. Kể về một lần bạn bất đồng kỹ thuật với designer / frontend / backend / PM. Bạn giải quyết xung đột đó ra sao?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

**📌 Câu trả lời mẫu:**

"Có lần ở Jira Clone, team Frontend muốn endpoint `GET /issues` trả về toàn bộ issue kèm comments, assignees, attachments lồng nhau cho tiện, còn mình thì lo payload phình lên vài MB và dính N+1 query xuống Postgres. Thay vì nói 'không' cụt lủn, mình quay về dữ liệu để thuyết phục: mình đo và chỉ ra payload nặng làm chậm cả mobile, rồi đưa ra phương án dung hòa thay vì áp đặt. Mình đề xuất list trả về gọn kèm cursor pagination, còn dữ liệu nặng thì tách sang `GET /issues/:id`; mình cũng viết contract ra OpenAPI/Swagger làm nguồn chân lý và ngồi duyệt từng field với FE và PM trước khi code. Cuối cùng cả hai chốt list gọn + cursor, detail riêng, p95 ổn định mà FE vẫn đủ data. Với mình, mấu chốt là giải quyết bất đồng bằng dữ liệu và đưa nhiều lựa chọn để cùng quy về mục tiêu chung, chứ không biến nó thành cuộc tranh thắng-thua; sau vụ đó team mình giữ luôn thói quen review API contract trước khi code."

---

- Nhà tuyển dụng muốn thấy bạn xử lý bất đồng dựa trên **dữ liệu** và **mục tiêu chung**, không phải cái tôi: dùng bằng chứng (demo, số đo, contract), đưa nhiều phương án để bên kia chọn, quy về mục tiêu chung.

**Ví dụ — bất đồng với designer (animation nặng):**
- **S:** Designer làm animation phức tạp cho list 200+ item bằng layout animation.
- **A:** Không nói thẳng "không làm được". Quay demo độ giật trên thiết bị thật; đề xuất: chỉ animate khi vào viewport, tôn trọng `prefers-reduced-motion`, dùng `transform` thay vì animate layout; mời designer tinh chỉnh cùng.
- **R:** Giữ ~90% cảm giác thiết kế mà đạt 60fps; sau đó designer chủ động hỏi giới hạn hiệu năng từ đầu.

**Ví dụ — bất đồng về API contract (FE ↔ BE):**
- **S:** FE muốn `GET /issues` trả **toàn bộ** issue kèm comments/assignees/attachments lồng nhau; tôi lo payload phình + N+1.
- **A:** Không nói "không" cụt; giải thích bằng **dữ liệu** (payload vài MB chậm cả mobile). Đề xuất dung hòa: list **gọn** + cursor, chi tiết nặng qua `GET /issues/:id`; nếu cần linh hoạt thì GraphQL cho FE tự chọn field. Viết contract ra **OpenAPI/Swagger** làm nguồn chân lý, duyệt từng field với FE+PM trước khi code (lightweight ADR).
- **R:** Chốt list gọn + cursor, detail riêng; p95 ổn định, FE đủ data. Lập thói quen **review contract trước khi code**.

- **Nhấn:** giải quyết bằng dữ liệu + phương án thay thế, không tranh thắng-thua.

</details>

---

### 2.3. Làm sao bạn giải thích một vấn đề / quyết định kỹ thuật phức tạp cho người không chuyên (PM, sales, khách hàng, sếp)?

<details>
<summary>👉 Xem câu trả lời</summary>

- Nguyên tắc cốt lõi: **nói về tác động (impact), không nói cơ chế (mechanism)** — họ quan tâm ảnh hưởng đến người dùng/tiền/thời gian, không phải `useMemo` hay "connection pool".
- Dùng **phép so sánh đời thường (analogy)** thay vì thuật ngữ; chỉ đi sâu kỹ thuật **nếu họ hỏi**.
- Quy sang **ngôn ngữ kinh doanh**; luôn đưa **đánh đổi dạng lựa chọn** để họ chốt; trực quan hóa bằng số/demo/biểu đồ.

| Khái niệm kỹ thuật | Cách nói cho người không chuyên |
|---|---|
| Background job / queue (BullMQ) | "Gọi món rồi nhận số thứ tự — có món sẽ báo, khỏi đứng chờ." |
| Database index | "Như mục lục cuốn sách — không có thì phải lật từng trang." |
| Cache (Redis) | "Để sẵn câu trả lời hay hỏi trên bàn, khỏi vào kho lấy." |
| Hydration | "Ảnh in sẵn để thấy ngay, rồi JS chạy nền thổi hồn cho nút bấm." |

Ví dụ quy kỹ thuật về kinh doanh:

> ❌ "Endpoint bị N+1 nên p95 latency tăng do round-trip tới Postgres."
>
> ✅ "Mỗi lần mở board, hệ thống hỏi database thừa hàng trăm lần → màn hình tải chậm. Tôi sửa để hỏi gọn lại, user sẽ thấy nhanh hơn rõ rệt."

> ❌ "App đang CSR nên FCP/LCP tệ, ta nên dùng SSG."
>
> ✅ "Người dùng nhìn màn hình trắng ~4s, Google chấm tốc độ thấp ảnh hưởng thứ hạng. Có kỹ thuật tạo sẵn trang để thấy nội dung gần như tức thì; tôi cần ~3 ngày."

**Bẫy:** dùng jargon khiến họ gật gù nhưng quyết sai. Kiểm tra lại bằng "anh/chị thấy rõ không?".

</details>

---

### 2.4. Kể về một lần bạn nhận feedback khó nghe về code/cách làm của mình. Bạn xử lý ra sao? Và bạn đưa feedback cho người khác thế nào?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

**📌 Câu trả lời mẫu:**

"Mình nhớ một lần bị một senior nhận xét khá thẳng trong code review: PR của mình gom quá nhiều thứ, vừa thêm component mới, vừa refactor, lại vừa đổi state management, nên rất khó review và rủi ro cao. Thú thật lúc đầu mình hơi tự ái, nhưng mình tự nhắc nhiệm vụ thật là ra code đúng chứ không phải bảo vệ cái tôi, nên mình tách feedback ra khỏi bản thân: anh ấy đang nói về *code*, không phải về *mình*. Mình cảm ơn, hỏi lại kỳ vọng và quy ước PR của team, rồi tách thành ba PR nhỏ độc lập, và từ đó tập thói quen mỗi PR chỉ làm một việc kèm mô tả rõ ràng. Kết quả là các PR sau merge nhanh hơn hẳn, và vài tháng sau chính mình lại đưa lời khuyên tương tự cho một bạn junior. Ngược lại, khi đưa feedback cho người khác, mình luôn phê bình code chứ không phê bình người, phân biệt rõ 'phải sửa' với 'gợi ý' bằng cách đánh dấu *nit*, hỏi thay vì ra lệnh khi chưa chắc, và khen phần làm tốt — vì với mình review là kênh học hai chiều, không phải đấu trường."

---

- Câu này đo độ trưởng thành cảm xúc và growth mindset. Đừng chọn feedback "giả vờ tiêu cực" kiểu "tôi quá cầu toàn"; chọn feedback thật cho thấy bạn đã thay đổi. Tách cái tôi khỏi công việc: feedback nhắm vào *code*, không nhắm vào *tôi*.
- **S:** Code review: senior nhận xét PR của tôi gom quá nhiều (component mới + refactor + đổi state management), khó review, rủi ro cao.
- **T:** Ban đầu hơi tự ái, nhưng tự nhắc nhiệm vụ thật là ra code đúng, không phải bảo vệ cái tôi.
- **A:** Cảm ơn, hỏi lại kỳ vọng/quy ước PR của team; tách thành 3 PR nhỏ độc lập; tập thói quen mỗi PR làm một việc + mô tả rõ.
- **R:** PR sau merge nhanh hơn; vài tháng sau tôi đưa lời khuyên tương tự cho junior.

**Khi ĐƯA feedback cho người khác:** phê bình code không phê bình người; phân biệt "phải sửa" (bug) với "đề xuất" (đánh dấu *nit*); hỏi thay vì ra lệnh khi chưa chắc ("kịch bản nào dẫn tới race condition?"); khen phần làm tốt. Code review là kênh nâng chất lượng và học hai chiều, không phải đấu trường thắng-thua.

</details>

---

### 2.5. Kể về một lần bạn mentor một đồng nghiệp ít kinh nghiệm hơn, hoặc giúp đỡ cả team. Kết quả ra sao?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

**📌 Câu trả lời mẫu:**

"Mình từng kèm một bạn junior liên tục dính bug stale closure và re-render thừa với `useEffect`, thậm chí có lần lọt cả lên production. Mình không muốn chỉ sửa hộ từng bug vì làm vậy bạn ấy không lớn lên được, nên mục tiêu của mình là giúp bạn hiểu *nguyên lý*. Mình ngồi pair programming, vẽ ra cách closure 'chụp' state ở mỗi lần render và khi nào thì cần khai báo dependency array, rồi để bạn ấy tự sửa và mình chỉ review lại. Để tạo đòn bẩy cho cả team chứ không dừng ở một người, mình làm thêm một buổi share nội bộ 30 phút về các bẫy `useEffect` và đề xuất bật rule `react-hooks/exhaustive-deps` để ESLint tự bắt. Sau khoảng một tháng số bug loại này giảm rõ, bạn ấy nhận được task khó hơn, và buổi share cộng lint rule giúp cả team tránh lặp lại lỗi cũ. Với mình, mentor tốt là dạy cách câu cá chứ không cho con cá, và biến kinh nghiệm cá nhân thành tài liệu hoặc công cụ để nhân rộng — chứ không phải kiểu 'mình giỏi nên sửa hết'."

---

- Câu này (cấp Senior) đánh giá khả năng nâng tầm người khác (force multiplier) và tư duy lâu dài: dạy *cách câu cá* chứ không *cho con cá*.
- **S:** Junior liên tục gặp bug stale closure và re-render thừa với `useEffect`, gây bug production.
- **T:** Tôi muốn bạn ấy hiểu *nguyên lý*, không chỉ sửa hộ từng bug.
- **A:** Pair programming, vẽ cách closure "chụp" state mỗi lần render và khi nào cần dependency array; để bạn ấy tự sửa rồi tôi review; làm buổi share nội bộ 30 phút về bẫy `useEffect` + đề xuất bật `react-hooks/exhaustive-deps`.
- **R:** Sau ~1 tháng bug giảm rõ, bạn ấy nhận task khó hơn; buổi share + ESLint rule giúp cả team tránh lặp lỗi.
- **Nhấn:** kiên nhẫn, dạy nguyên lý thay vì sửa hộ, tạo *đòn bẩy* cho cả team (tài liệu, lint rule, share). Tránh kiểu "tôi giỏi nên sửa hết".

</details>

---

## 3. Ownership, Sự cố và Sai lầm

### 3.1. "Own một feature end-to-end" nghĩa là gì? Trách nhiệm của bạn trải dài tới đâu?

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

### 3.2. Kể về một lần bạn xử lý một sự cố production nghiêm trọng. Bạn đã làm gì? (STAR)

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

**📌 Câu trả lời mẫu:**

"Một sáng ở Jira Clone, API gần như đứng hình — p99 latency từ khoảng 200ms vọt lên hơn 8 giây, mà 12 tiếng trước đó không hề có deploy nào. Nguyên tắc của mình lúc xử lý sự cố là giảm thiệt hại trước, tìm root cause sau, và điều tra có phương pháp thay vì đoán mò. Nhìn metrics mình thấy CPU database vẫn bình thường nhưng active connection chạm trần pool, còn trace thì cho thấy thời gian dồn hết ở bước chờ lấy connection; lần theo correlation id trong log thì ra một endpoint report mới đang bị một khách lớn gọi liên tục, transaction dài cộng query thiếu index giữ connection quá lâu nên cạn pool. Việc đầu tiên mình làm là rate-limit ngay endpoint đó để giải phóng pool, hệ thống hồi phục sau khoảng 20 phút; rồi mình mới xử lý gốc rễ — thêm index và xác nhận bằng `EXPLAIN ANALYZE`, tách phần report nặng sang job BullMQ, và đặt statement timeout. Report sau đó từ 8 giây xuống dưới 300ms. Cuối cùng tụi mình làm postmortem blameless và thêm alert cho connection pool utilization để lần sau phát hiện sớm hơn — với mình quan trọng là biến sự cố thành cải tiến hệ thống chứ không phải đi tìm người để đổ lỗi."

---

- Muốn thấy: bình tĩnh, **ưu tiên giảm thiệt hại (mitigation) TRƯỚC root cause**, điều tra có phương pháp (monitoring/log/trace), rút bài học và biến sự cố thành cải tiến hệ thống. Đừng đổ lỗi.

**Ví dụ — Frontend (UI hỏng trên prod):**
- **S:** Deploy chiều thứ Sáu khiến trang thanh toán trắng màn hình trên Safari.
- **A:** *Rollback* ngay để dừng thiệt hại → Sentry thấy `Cannot read properties of undefined` (field API `null`, optional chaining sai chỗ) → thêm guard + `ErrorBoundary` bọc luồng thanh toán + viết test cho ca `null`.
- **R:** UI khôi phục ~8 phút nhờ rollback, fix thật deploy sau ~1 tiếng. Postmortem không đổ lỗi; thêm `ErrorBoundary` cho luồng quan trọng.

**Ví dụ — Backend (API latency vọt, cạn connection pool):**
- **S:** Jira Clone một sáng API gần đứng — p99 latency từ 200ms vọt >8s, không deploy nào trong 12h trước.
- **A:** Metrics thấy CPU DB bình thường nhưng **active connection chạm trần pool**; trace cho thấy thời gian dồn ở bước **chờ lấy connection**. Log theo correlation id lần ra một endpoint report mới bị khách lớn gọi liên tục — transaction dài + query thiếu index giữ connection lâu, cạn pool. Tức thời **rate-limit endpoint** đó để giải phóng pool; sau đó **thêm index** (xác nhận `EXPLAIN ANALYZE`), tách report nặng sang **job BullMQ**, đặt **statement timeout**.
- **R:** Hồi phục ~20 phút sau rate-limit; report từ 8s xuống <300ms. **Postmortem blameless**, thêm **alert connection pool utilization**.

- **Nhấn:** mitigation (rollback/feature flag/rate-limit) trước root cause; dùng monitoring (Sentry/metrics/trace); thêm test/alert hồi quy.

</details>

---

### 3.3. Mô tả một lần bạn mắc lỗi đáng kể (hoặc nhận sai một quyết định kỹ thuật của chính mình) và đã sửa ra sao. (STAR)

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

**📌 Câu trả lời mẫu:**

"Trong Jira Clone, để giảm tải Postgres mình từng cache danh sách issue của mỗi board trong Redis với TTL cố định 60 giây. Nghe thì hợp lý, nhưng sau release user phàn nàn rằng kéo thả issue xong thì người khác phải chờ tới 60 giây mới thấy thay đổi — tức là mình đã phá vỡ tính real-time vốn là cốt lõi của sản phẩm cộng tác. Thay vì cố bảo vệ quyết định của mình, mình thừa nhận thẳng rằng TTL cố định là sai cho use-case này, vì dữ liệu board thay đổi theo event chứ không theo thời gian. Mình đổi sang cache invalidation theo event: key cache để per-board, và mỗi khi có mutation thì mình xóa hoặc cập nhật đúng key của board đó ngay trong cùng flow update, đồng thời push thay đổi qua Socket.IO để các client thấy gần như tức thì. Kết quả là board cập nhật gần như real-time mà tải DB vẫn thấp như mong muốn. Bài học mình rút ra là sẵn sàng đảo lại quyết định của chính mình khi có bằng chứng — và đó là điều mình nghĩ một engineer trưởng thành nên làm thay vì cố chấp đúng."

---

- Câu này kiểm tra trung thực, ownership, khả năng học. "Tôi chưa từng mắc lỗi" = thiếu tự nhận thức. Chọn lỗi thật, nhận trách nhiệm sớm, sửa cả **triệu chứng lẫn quy trình**, rút bài học cụ thể.

**Ví dụ — mắc lỗi giả định môi trường (Frontend):**
- **S:** Đẩy thay đổi lưu state lên `localStorage` mà không test kỹ chế độ riêng tư của Safari (nơi `localStorage` bị giới hạn) → một nhóm user crash app khi mở.
- **A:** Báo ngay team/stakeholder thay vì giấu, không đổ lỗi trình duyệt; bọc storage trong try/catch + fallback in-memory + test ca storage không khả dụng; thêm kiểm thử cross-browser (cả chế độ riêng tư) vào checklist QA.
- **R:** Xử lý trong ngày; checklist mới bắt thêm vài vấn đề tương thích trước prod. Bài học: giả định về môi trường người dùng là nguồn bug nguy hiểm nhất của frontend.

**Ví dụ — đảo quyết định của chính mình (Backend):**
- **S:** Trong Jira Clone, tôi cache danh sách issue của board trong Redis với TTL cố định 60s để giảm tải Postgres.
- **A:** Sau release, user phàn nàn kéo thả xong người khác chờ tới 60s mới thấy — phá real-time. Tôi **thừa nhận TTL sai cho use-case cộng tác**, không bảo vệ. Đổi sang **cache invalidation theo event** (mutation thì xóa/cập nhật key cache board đó) + push qua Socket.IO; key per-board, invalidate trong cùng flow update.
- **R:** Board cập nhật gần tức thì, tải DB vẫn thấp.

- **Nhấn:** chủ động báo + nhận trách nhiệm sớm; sẵn sàng đảo quyết định của chính mình theo bằng chứng (dấu hiệu senior); sửa cả quy trình, không chỉ triệu chứng.

</details>

---

## 4. Ưu tiên và Áp lực

### 4.1. Khi backlog vừa có technical debt vừa có tính năng mới mà sếp/khách hàng đang hối, bạn ưu tiên thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Câu này kiểm tra tư duy trade-off và giao tiếp — quyết định dựa trên **tác động kinh doanh & rủi ro**, không phải sở thích.
- **Không phải mọi tech debt đều ngang nhau:** nợ đang "chảy máu" (gây bug, chậm mọi feature, rủi ro bảo mật) ưu tiên cao; nợ "khó chịu nhưng không cản trở" hoãn được.
- **Lượng hóa chi phí:** thay vì "code xấu", nói "mỗi feature đụng module này tốn thêm 2-3 ngày và hay sinh bug".
- **"Đóng thuế" (boy scout rule):** dành ~15-20% mỗi sprint trả nợ, hoặc refactor từng phần trên đường làm feature; để stakeholder quyết định có thông tin (trình bày rõ đánh đổi).

| Loại | Ví dụ | Hành động |
| --- | --- | --- |
| Nguy hiểm | State rối đẻ bug, dependency có CVE | Xử lý gần như ngay |
| Cản trở | Thiếu test luồng core, prop drilling sâu | Trả dần khi làm feature liên quan |
| Thẩm mỹ | Tên biến chưa đẹp, CSS trùng | Hoãn, gom lúc rảnh |

**Ví dụ thực tế (Jira Clone):** module `notifications` gửi email/Socket.IO **đồng bộ** trong request handler làm p95 mutation tăng và fail khi SMTP chậm — nợ "đang đau định kỳ" + chặn tính năng mới. Refactor sang **BullMQ** vừa trả nợ vừa mở đường cho job type mới → thêm notification mới chỉ là thêm 1 job type.

**Bẫy:** trả lời cực đoan ("luôn ưu tiên feature" = thiếu trách nhiệm dài hạn; "luôn refactor trước" = thiếu nhạy bén kinh doanh). Câu tốt nằm ở giữa, theo **rủi ro × tần suất đau**.

</details>

---

### 4.2. Hãy kể về một lần bạn phải làm việc dưới áp lực deadline rất gắt. Bạn xử lý thế nào để vừa kịp tiến độ vừa không bung chất lượng?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

**📌 Câu trả lời mẫu:**

"Mình từng phải launch một landing page chiến dịch trong vỏn vẹn 3 ngày trước một sự kiện báo chí, mà scope ban đầu rõ ràng là to hơn thời gian có. Cách mình xử lý không phải là cày xuyên đêm làm cho hết, mà là ưu tiên thông minh và báo rủi ro sớm. Ngay ngày đầu mình ngồi với PM phân loại theo MoSCoW: form đăng ký và phần hero là Must, còn animation phụ với A/B test xếp vào Could và sẵn sàng cắt; mình cũng cảnh báo stakeholder ngay từ đầu về rủi ro để họ chủ động quyết định cắt gì thay vì để đến phút chót. Để chạy nhanh mà không bung chất lượng, mình tái dùng component có sẵn từ design system, dùng feature flag để có thể bật phần chưa kịp hoàn thiện sau khi launch, và vẫn giữ test cho hai luồng quan trọng nhất là đăng ký và thanh toán. Kết quả là page live đúng hạn, luồng chính ổn định, không có sự cố trong sự kiện, còn phần Could thì bổ sung dần sau qua feature flag. Với mình, dưới áp lực thì cắt *scope* có chủ đích luôn an toàn hơn cắt chất lượng, và điều quan trọng nhất là báo rủi ro đủ sớm để cả team cùng ra quyết định."

---

- Câu này đo quản lý áp lực, ưu tiên việc, giao tiếp sớm. Nhà tuyển dụng *không* muốn nghe "tôi cày xuyên đêm làm hết"; muốn thấy ưu tiên thông minh và báo rủi ro kịp thời.
- **S:** Phải launch landing page chiến dịch trong 3 ngày trước sự kiện báo chí, scope lớn hơn thời gian.
- **T:** Live đúng hạn với chất lượng chấp nhận được cho luồng quan trọng nhất, thay vì làm hết dở dang.
- **A:** Cùng PM phân loại **MoSCoW** (form đăng ký + hero = Must; animation phụ, A/B test = Could, cắt); tái dùng component design system; dùng **feature flag** để bật phần chưa xong sau launch; cảnh báo stakeholder ngay ngày đầu về rủi ro để họ quyết cắt gì.
- **R:** Launch đúng hạn, luồng chính ổn và có test cho đăng ký/thanh toán; phần "Could" bổ sung dần qua feature flag; không sự cố trong sự kiện.
- **Nhấn:** cắt *scope* có chủ đích thay vì cắt chất lượng; báo rủi ro *sớm*; tái sử dụng + feature flag; giữ test cho phần quan trọng. Tránh khoe hy sinh sức khỏe.

</details>

---

## 5. Học hỏi và Phát triển

### 5.1. Bạn cập nhật kiến thức trong một hệ sinh thái thay đổi nhanh (React/Next.js, Node/NestJS) bằng cách nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Câu này kiểm tra khả năng tự học **và sự cân bằng** — không chạy theo mọi thứ mới (shiny object syndrome); biết khi nào *nên* và *chưa nên* áp dụng.
- **Nguồn chính thống:** blog/changelog của framework (React, Next.js, NestJS, Prisma, TanStack) — đọc release notes.
- **Theo dõi người tạo công cụ** và **RFC / GitHub Discussions** để hiểu *tại sao* một API ra đời (vd RSC, React Compiler).
- **Học bằng cách làm:** side project nhỏ thử công nghệ mới trước khi đề xuất vào dự án thật. Newsletter/podcast để nắm bức tranh lớn ít tốn thời gian.

| Tiêu chí | Câu hỏi |
| --- | --- |
| Độ chín | Stable hay còn experimental/canary? |
| Vấn đề thật | Giải quyết vấn đề ta gặp, hay chỉ "trông hay"? |
| Chi phí chuyển đổi | Team học được không? Migration tốn bao nhiêu? |
| Cộng đồng | Được bảo trì tích cực, dễ tuyển người biết không? |

**Bẫy:** "tôi luôn dùng phiên bản/công nghệ mới nhất" khiến nhà tuyển dụng lo bạn đưa rủi ro vào dự án. Cập nhật để *biết và đánh giá*, rồi áp dụng có chọn lọc.

</details>

---

## 6. Làm việc với AI

### 6.1. "Làm việc AI-native nhưng có phán đoán" nghĩa là gì? Khi nào bạn tin output của AI và khi nào bắt buộc verify?

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

### 6.2. Hãy mô tả quy trình hằng ngày của bạn khi dùng AI (đặc biệt là Claude Code) trong công việc. AI xen vào những bước nào, và bạn giữ vai trò gì ở mỗi bước?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

**📌 Câu trả lời mẫu:**

"Mình làm backend NestJS + Prisma cho Jira Clone, mỗi ngày là một mớ task thêm tính năng, sửa bug, refactor hay viết test, và trước đây phần lớn thời gian trôi vào gõ boilerplate với đi mò code. Triết lý của mình là để AI gánh phần cơ học và khám phá code, còn mình dồn sức vào thiết kế và quyết định — nhưng vẫn hiểu và chịu trách nhiệm cho mọi thứ ship ra. Cụ thể mình gắn AI vào từng bước với đúng công cụ: `CLAUDE.md` để nạp ngữ cảnh đầu phiên và `/clear` khi đổi task, subagent Explore để tìm code mà không làm phình context, plan mode để duyệt hướng trước khi cho nó sửa, permission mode mặc định để nó hỏi trước khi thực thi, và hook PostToolUse để tự động lint và chạy test sau mỗi thay đổi. Bước cuối cùng luôn là mình đọc toàn bộ diff như đang review một PR thật, và với phần nhạy cảm như auth hay quyền thì mình chạy thêm skill code-review hoặc security-review. Nhờ vậy thời gian gõ tay giảm rõ rệt, tìm code nhanh hơn, nhưng chất lượng không giảm vì hook và bước review là chốt chặn — và quan trọng nhất là mình vẫn giải thích được mọi thay đổi trong PR, tức là mình vẫn là người cầm lái chứ không phải copy mù."

---

- **S:** Làm backend NestJS + Prisma (Jira Clone), mỗi ngày nhận task thêm tính năng/sửa bug/refactor/test; trước đây tốn nhiều thời gian gõ boilerplate và tìm code.
- **T:** Muốn AI gánh phần cơ học và khám phá code, để dồn sức vào thiết kế và quyết định — nhưng vẫn hiểu và chịu trách nhiệm mọi thứ ship.
- **A:** Gắn AI vào từng bước với đúng công cụ: `CLAUDE.md` nạp ngữ cảnh đầu phiên (`/clear` khi đổi task); subagent Explore để tìm code mà không phình context; plan mode duyệt hướng trước khi sửa; permission mode mặc định (hỏi trước) khi thực thi; hook PostToolUse tự lint/test; cuối cùng đọc toàn bộ diff như review PR (dùng skill code-review/security-review cho phần nhạy cảm).
- **R:** Gõ tay giảm rõ rệt, tìm code nhanh hơn, chất lượng không giảm vì hook + review là chốt chặn; vẫn giải thích được mọi thay đổi trong PR.

Lưu ý: nêu cụ thể công cụ cho từng bước để cho thấy "đúng việc dùng đúng công cụ", tránh kiểu "tôi hỏi AI rồi copy". Luôn nhấn mạnh mình là người cầm lái.

</details>

---

### 6.3. Kể về một task cụ thể mà AI giúp bạn tăng tốc đáng kể. Việc đó nếu làm tay sẽ tốn bao lâu, và bạn đã đảm bảo chất lượng không bị hy sinh như thế nào?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

**📌 Câu trả lời mẫu:**

"Một task mà AI giúp mình tăng tốc rõ nhất là khi cần thêm validation và DTO nhất quán cho khoảng 12 endpoint kèm test — làm tay thì tốn cỡ 2-3 ngày vì lặp đi lặp lại và rất dễ sai sót do mệt. Mục tiêu của mình là rút ngắn thời gian nhưng không hạ chất lượng, nghĩa là phải khớp convention và có test bao phủ các luồng quan trọng. Mình mô tả rõ pattern trong prompt, cho AI đọc ví dụ đúng convention từ `CLAUDE.md`, rồi để nó sinh DTO, validation và test theo từng endpoint, cố ý chia nhỏ để mỗi diff ngắn và dễ review. Mình bật hook tự chạy test để chặn merge code đỏ, và quan trọng là *tự tay viết phần edge case nghiệp vụ* như ràng buộc quyền hay trạng thái hợp lệ, vì đó đúng là chỗ AI hay đoán sai. Kết quả là 2-3 ngày rút còn khoảng nửa ngày, test phủ luồng chính, và mình vẫn đọc hết diff để giải thích được mọi thay đổi. Bài học của mình là AI tăng tốc phần 'typing', còn phần 'thinking' về edge case vẫn là việc của mình — và đó mới là nơi tạo ra giá trị thật."

---

- **S:** Cần thêm validation + DTO nhất quán cho ~12 endpoint kèm test; làm tay mất khoảng 2-3 ngày vì lặp lại và dễ sai do mệt.
- **T:** Rút ngắn thời gian nhưng không hạ chất lượng: khớp convention và có test bao phủ các luồng quan trọng.
- **A:** Mô tả rõ pattern trong prompt, cho AI đọc ví dụ đúng convention từ `CLAUDE.md`; sinh DTO + validation + test theo từng endpoint, chia nhỏ để mỗi diff ngắn dễ review; bật hook tự chạy test; **tự viết phần edge case nghiệp vụ** (ràng buộc quyền, trạng thái hợp lệ) vì AI dễ đoán sai.
- **R:** 2-3 ngày rút còn ~nửa ngày; test bao phủ luồng chính, hook chặn merge code đỏ test; đọc hết diff và giải thích được mọi thay đổi.

Bài học: AI tăng tốc phần "typing", phần "thinking" về edge case vẫn là của mình — đó là nơi giá trị thật. Hãy gắn con số và mô tả cơ chế giữ chất lượng (diff nhỏ, hook test, tự viết edge case).

</details>

---

### 6.4. Kể về một lần AI dẫn bạn đi sai hướng — ví dụ nó bịa ra API, gợi ý một giải pháp sai tinh vi, hoặc tạo code chạy được nhưng sai bản chất. Bạn phát hiện và xử lý ra sao?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

**📌 Câu trả lời mẫu:**

"Có lần mình nhờ AI viết phân trang với Prisma; code trông rất hợp lý, chạy ngon trên dữ liệu nhỏ và pass cả test ban đầu, nhưng nó dùng offset đơn giản và còn gợi ý một option cấu hình nghe rất đúng mà thực ra không tồn tại. Có hai cờ đỏ khiến mình dừng lại: mình không tự tin giải thích được behavior khi dữ liệu lớn lên, và cái option kia thì 'nghe đúng' nhưng mình chưa từng thấy trong tài liệu. Mình áp quy tắc cá nhân là 'không hiểu thì không merge', nên đối chiếu thẳng với tài liệu chính thống của Prisma — và đúng là option đó là hallucination, đồng thời mình cũng nhận ra offset sẽ chậm dần khi dữ liệu lớn. Sau đó mình yêu cầu AI giải thích lựa chọn và đề xuất hướng cursor-based, rồi tự kiểm chứng lại lần nữa trước khi áp dụng. Cuối cùng mình thay bằng cursor-based pagination dùng đúng API thật và viết test cho cả trường hợp dữ liệu lớn. Bài học là với API lạ thì phải kiểm chứng bằng tài liệu, không kiểm chứng được thì chưa đủ tư cách ship, và phải test ở quy mô thật chứ không chỉ happy path. Mình không đổ lỗi cho AI — thông điệp đúng vẫn là 'trust but verify'."

---

- **S:** Nhờ AI viết phân trang với Prisma; code trông hợp lý, chạy được trên dữ liệu nhỏ, pass test ban đầu — nhưng dùng offset đơn giản và gợi ý một option cấu hình mơ hồ là không tồn tại.
- **T:** Xác định giải pháp có đúng và bền không trước khi đưa vào production, nhất là khi dữ liệu lớn.
- **A:** Hai cờ đỏ khiến dừng lại: không giải thích chắc chắn được behavior khi dữ liệu lớn, và option "nghe đúng" nhưng chưa từng thấy trong tài liệu. Áp quy tắc "không hiểu thì không merge"; đối chiếu tài liệu chính thống Prisma → option đó là hallucination; nhận ra offset chậm dần khi dữ liệu lớn; yêu cầu AI giải thích lựa chọn và đề xuất cursor-based, rồi tự kiểm chứng lại.
- **R:** Thay bằng cursor-based pagination đúng API thật, có test cho dữ liệu lớn. Bài học: kiểm chứng API lạ bằng tài liệu; không kiểm chứng được thì chưa đủ tư cách ship; test trên quy mô thực, không chỉ happy path.

Lưu ý: chọn sự cố THẬT, kể rõ dấu hiệu nghi ngờ và cách kiểm chứng. Đừng đổ lỗi AI; thông điệp đúng là "trust but verify".

</details>

---

## ✅ Checklist ôn nhanh

- [ ] Chuẩn bị sẵn **4-5 câu chuyện thật** theo **STAR**, mỗi câu xoay được nhiều hướng hỏi (sự cố, xung đột, sai lầm, deadline, mentor).
- [ ] Sự cố production: luôn nói **mitigation trước root cause**, rồi postmortem + test/alert hồi quy.
- [ ] Bất đồng/feedback: giải quyết bằng **dữ liệu + mục tiêu chung**, tách cái tôi khỏi công việc.
- [ ] Tech debt / deadline: ưu tiên theo **rủi ro × tác động**, cắt **scope** chứ không cắt chất lượng, báo rủi ro **sớm**.
- [ ] Ways of working: MVP-first, YAGNI/KISS, ước lượng theo vertical slice, tự tháo gỡ có time-box.
- [ ] Làm việc với AI: **AI-native nhưng có phán đoán** — mình chịu trách nhiệm cuối, luôn verify phần nhạy cảm.

---

> 📌 *Điểm cộng lớn nhất ở câu hành vi: kể chuyện THẬT, cụ thể, có số liệu, nhận trách nhiệm và rút bài học — thể hiện bạn vừa có kỹ năng vừa có **judgment** và tinh thần đồng đội.* 🚀
