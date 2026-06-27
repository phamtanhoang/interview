# 🤖 Bộ câu hỏi phỏng vấn: Kỹ năng ứng dụng AI (Claude Code & AI-assisted Development)

> Tài liệu ôn tập cho các câu hỏi phỏng vấn về **kỹ năng tận dụng AI trong lập trình** — đặc biệt là **Claude Code** (agentic coding) và **Claude API**.
> Nội dung: tư duy ứng dụng AI, prompting & brainstorming, skills, subagents, rules/CLAUDE.md, hooks, MCP, workflows, best practices và câu hỏi hành vi.
> Mỗi đáp án nằm trong khối **collapse** (`<details>`) — bấm để xem lời giải + ví dụ.
>
> ⚠️ *Lưu ý:* Hệ sinh thái công cụ AI thay đổi rất nhanh. Hãy kiểm chứng lại các tính năng cụ thể với tài liệu chính thức của Claude Code / Anthropic ngay trước buổi phỏng vấn.

## 📚 Mục lục

1. [Tư duy ứng dụng AI trong lập trình](#1-tư-duy-ứng-dụng-ai-trong-lập-trình)
2. [Prompting và Brainstorming với AI](#2-prompting-và-brainstorming-với-ai)
3. [Claude Code Tổng quan](#3-claude-code-tổng-quan)
4. [Claude Code Skills và Slash Commands](#4-claude-code-skills-và-slash-commands)
5. [Claude Code Agents và Subagents](#5-claude-code-agents-và-subagents)
6. [Claude Code Rules CLAUDE.md và Memory](#6-claude-code-rules-claudemd-và-memory)
7. [Claude Code Hooks và Tự động hóa](#7-claude-code-hooks-và-tự-động-hóa)
8. [MCP Model Context Protocol](#8-mcp-model-context-protocol)
9. [Workflows và Điều phối Multi-Agent](#9-workflows-và-điều-phối-multi-agent)
10. [Claude API và Xây dựng tính năng AI](#10-claude-api-và-xây-dựng-tính-năng-ai)
11. [Best Practices và Siêu năng lực với AI](#11-best-practices-và-siêu-năng-lực-với-ai)
12. [Tư duy và Đánh giá kết quả AI](#12-tư-duy-và-đánh-giá-kết-quả-ai)
13. [Câu hỏi hành vi về ứng dụng AI](#13-câu-hỏi-hành-vi-về-ứng-dụng-ai)
14. [Cursor, Copilot và Lựa chọn công cụ AI](#14-cursor-copilot-và-lựa-chọn-công-cụ-ai)

---

## 1. Tư duy ứng dụng AI trong lập trình

### 1.1. Bạn nhìn nhận vai trò của AI trong lập trình như thế nào — nó là công cụ thay thế lập trình viên, hay là một dạng "trợ lý"? Vì sao quan điểm này lại quan trọng với chất lượng sản phẩm?

<details>
<summary>👉 Xem câu trả lời</summary>

- AI (nhất là agentic coding như Claude Code) là pair-programmer cấp cao, không thay thế tư duy: nó tạo code nhanh nhưng không chịu trách nhiệm sản phẩm.
- Vai trò của tôi chuyển từ "gõ từng dòng" sang "định hướng, ra quyết định kiến trúc, kiểm soát chất lượng".
- AI tăng tốc phần "typing"; phần "thinking" (thiết kế, đánh đổi, đọc hiểu hệ thống) vẫn là của tôi.
- Quan điểm này quan trọng: coi AI là "người thay thế" -> ship code không hiểu -> nợ kỹ thuật và lỗ hổng bảo mật.

| Khía cạnh | AI làm tốt | Con người giữ |
| --- | --- | --- |
| Tốc độ code | Rất nhanh, boilerplate | Có nên viết phần đó không |
| Kiến thức | Rộng nhưng có thể sai | Ngữ cảnh, lý do (the "why") |
| Trách nhiệm | Không có | Toàn bộ: review, an toàn, vận hành |

Bẫy: nhầm tốc độ gõ code với tốc độ giải quyết vấn đề.

</details>

---

### 1.2. Phân biệt "AI-assisted coding" và "vibe coding". Khi nào mỗi cách là phù hợp?

<details>
<summary>👉 Xem câu trả lời</summary>

- Khác nhau ở mức độ lập trình viên hiểu và sở hữu code đầu ra.
- Vibe coding: mô tả mong muốn rồi chấp nhận thứ AI sinh ra mà không soát kỹ ("chạy là được") — hợp prototype dùng một lần, hackathon, học API lạ.
- AI-assisted: tôi cầm lái — quyết kiến trúc, giao việc, đọc/review diff, yêu cầu sửa rồi mới merge — dùng cho code production.
- Ranh giới không nằm ở công cụ mà ở việc tôi có thật sự hiểu và sở hữu code không.

| Tiêu chí | AI-assisted | Vibe coding |
| --- | --- | --- |
| Đọc/hiểu code | Review kỹ, hiểu rõ | Hiếm đọc, không hiểu |
| Mục tiêu | Production, bền vững | Prototype nhanh |
| Rủi ro | Thấp nếu review tốt | Cao: nợ kỹ thuật, lỗ hổng ẩn |

Bẫy: vibe coding một prototype rồi để nó "trườn" thẳng vào production không qua review.

</details>

---

### 1.3. Khi nào bạn NÊN dùng AI trong một task, và khi nào bạn quyết định KHÔNG dùng (hoặc dùng rất hạn chế)?

<details>
<summary>👉 Xem câu trả lời</summary>

- Quyết theo 2 trục: mức rủi ro/độ phức tạp ngữ cảnh, và tôi có kiểm chứng được đầu ra không.
- NÊN dùng: boilerplate (CRUD, DTO, controller), refactor cơ học, viết test cho code đã hiểu, tài liệu/commit message, khám phá code base lạ (Explore subagent), debug như "đôi mắt thứ hai", học khái niệm mới.
- HẠN CHẾ/KHÔNG dùng: quyết định kiến trúc cốt lõi, code bảo mật/xác thực/thanh toán, logic nghiệp vụ tôi chưa hiểu, khi ngữ cảnh quá lớn không truyền đủ cho AI.

```text
Quy tắc lọc nhanh:
1. Mình có hiểu đủ để REVIEW không? Không -> tự làm hoặc dùng AI để HỌC, đừng để nó QUYẾT.
2. Sai thì hậu quả tới đâu? Nghiêm trọng -> review gấp đôi.
3. Có khuôn mẫu lặp lại không? Có -> giao mạnh cho AI.
```

Bẫy: dùng AI cho đúng việc mình KHÔNG biết cách kiểm chứng — không phân biệt được "đúng" với "nghe có vẻ đúng".

</details>

---

### 1.4. Hãy liệt kê những loại công việc AI giúp ích nhiều nhất trong vòng đời phát triển, và với mỗi loại, AI giúp bằng cách nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Agentic coding mạnh vì tự đọc file, chạy lệnh, lặp lại — làm tốt cả việc cần "đi vòng quanh" code base.
- Nguyên tắc: AI làm 80% có khuôn mẫu (mechanical), con người giữ 20% quyết định và cạm bẫy nghiệp vụ.

| Loại việc | AI giúp | Tôi vẫn kiểm soát |
| --- | --- | --- |
| Boilerplate | Sinh CRUD, DTO, scaffold | Khớp convention (CLAUDE.md) |
| Refactor | Đổi tên xuyên file, tách hàm | Phải có test trước |
| Test | Sinh test, liệt kê edge case | Quyết case nào đáng test |
| Debug | Đề xuất giả thuyết, đọc log | Xác nhận nguyên nhân gốc |
| Khám phá | Subagent Explore tìm "code X ở đâu" | Đọc lại để hiểu |

Ví dụ: thêm resource NestJS — mô tả schema, để Claude Code sinh module/service/controller/DTO theo `CLAUDE.md`, rồi tôi review và bổ sung logic + test.

Bẫy: kỳ vọng AI tự biết convention dự án — nó chỉ biết khi bạn cung cấp ngữ cảnh (CLAUDE.md, ví dụ mẫu, prompt rõ).

</details>

---

### 1.5. "Giữ quyền sở hữu và hiểu rõ code mình ship" nghĩa là gì khi dùng AI? Làm sao bạn đảm bảo điều đó trong thực tế?

<details>
<summary>👉 Xem câu trả lời</summary>

- Sở hữu code = dù AI viết, tôi giải thích được từng quyết định, sửa được khi nó hỏng, chịu trách nhiệm trước team. Quy tắc: "không hiểu thì không merge".
- Đọc toàn bộ diff trước khi merge như review PR đồng nghiệp; AI là "đồng nghiệp" cần review.
- Chia task nhỏ để diff đủ ngắn; dùng plan mode để duyệt hướng đi trước khi sửa code.
- Yêu cầu AI giải thích "vì sao chọn cách này"; tự viết/duyệt kỹ test để chứng minh mình hiểu hành vi.

```text
Checklist trước khi merge:
[ ] Đọc hết diff? Giải thích được từng thay đổi?
[ ] Có test cho đường đi quan trọng?
[ ] Khớp convention (CLAUDE.md)?
[ ] Có chỗ "magic" không hiểu? -> dừng lại, làm rõ
```

Bẫy: code chạy đúng happy path nhưng sai edge case hoặc dùng API deprecated. Quyền sở hữu thể hiện rõ nhất lúc sự cố — người hiểu code là người cứu được hệ thống.

</details>

---

### 1.6. Rủi ro của việc phụ thuộc quá mức vào AI là gì? Bạn nhận biết và phòng tránh ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

- Rủi ro chính: teo kỹ năng (skill atrophy), mất kiến thức ngữ cảnh hệ thống, tin mù (automation bias), code "nghe có vẻ đúng" do hallucinate API, đồng nhất hóa giải pháp (luôn nhận cách "trung bình phổ biến").
- Dấu hiệu phụ thuộc quá mức: merge code mà không đọc; không debug được khi AI offline; copy đề xuất mà không hỏi "tại sao".

```text
Phòng tránh:
- Giữ thói quen tự làm phần cốt lõi/khó để duy trì kỹ năng.
- Luôn review như review đồng nghiệp; không merge thứ không hiểu.
- Kiểm chứng API/khái niệm bằng tài liệu chính thống.
- Để máy lo kỷ luật lặp lại bằng HOOK (PostToolUse lint/test) thay vì "nhờ AI nhớ".
- Đặt permission mode hợp lý (mặc định hỏi; chỉ acceptEdits khi tin tưởng phạm vi).
```

Điểm nhấn: việc tự động (vd "sửa file -> format/lint") phải là hook trong `settings.json`, không phải lời nhắc AI — AI có thể quên, harness luôn chạy. Phân biệt "việc tất định" và "việc cần suy luận" là dấu hiệu người dùng AI trưởng thành.

</details>

---

### 1.7. AI ảnh hưởng thế nào tới năng suất VÀ chất lượng? Năng suất tăng có luôn đồng nghĩa chất lượng tăng không?

<details>
<summary>👉 Xem câu trả lời</summary>

- Hai thứ không tự đi cùng nhau: AI gần như chắc chắn tăng throughput, nhưng chất lượng phụ thuộc quy trình kiểm soát quanh nó.
- Bỏ qua review -> "năng suất tăng" chỉ là đẩy nhanh việc tạo nợ kỹ thuật và bug.
- Năng suất thật = "tính năng đúng, bền, ship được/đơn vị thời gian", không phải "số dòng code/giờ". Sinh gấp 3 code mà tốn gấp 5 thời gian sửa bug là năng suất ÂM.

| | Năng suất | Chất lượng |
| --- | --- | --- |
| AI tác động | Tăng mạnh gõ code, scaffold | Trung tính, tùy quy trình |
| Rủi ro lạm dụng | PR khổng lồ khó review | Bug ẩn, nợ kỹ thuật, lỗ hổng |
| Yếu tố quyết định | Mô tả task rõ, ngữ cảnh tốt | Review, test, hook, kiến trúc |

- Nút thắt dịch sang khâu review/hiểu code -> đầu tư mạnh vào tự động hóa kiểm soát (test, hook, CI, skill code-review/security-review).

Bẫy: đo năng suất bằng cảm giác "code ra nhanh" — hãy đo bằng kết quả cuối: tính năng đúng, ít bug, dễ bảo trì.

</details>

---

### 1.8. Hãy kể về một lần bạn dùng AI như một "siêu năng lực" để hoàn thành việc mà nếu không có AI sẽ tốn hơn rất nhiều — nhưng bạn vẫn giữ quyền kiểm soát chất lượng.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Câu hỏi hành vi — kể một câu chuyện cụ thể theo cấu trúc STAR (mẫu để điều chỉnh theo trải nghiệm thật):

- **Situation:** "Dự án Jira Clone (NestJS + Prisma), cần thêm lớp audit log cho hàng loạt resource hiện có kèm test — làm tay rất lặp lại, tốn nhiều ngày."
- **Task:** "Rút ngắn thời gian nhưng không hạ chất lượng — code phải khớp convention, có test, và tôi phải hiểu hết để chịu trách nhiệm."
- **Action:**
  - "Viết convention/kiến trúc vào `CLAUDE.md` để Claude Code tự nạp ngữ cảnh."
  - "Dùng plan mode duyệt kế hoạch trước khi cho sửa code."
  - "Chia nhỏ theo từng resource để diff ngắn, review kỹ; dùng subagent Explore định vị code cũ."
  - "Cấu hình hook PostToolUse tự chạy lint/test mỗi khi file đổi — kỷ luật do harness đảm bảo."
  - "Phần nhạy cảm tự viết/đọc cực kỹ; từ chối merge đoạn không giải thích được."
- **Result:** "Việc lặp lại xong nhanh hơn nhiều lần, test bao phủ đường đi chính, tôi vẫn bảo vệ được toàn bộ thay đổi trong review. Quy trình tái dùng: CLAUDE.md + plan mode + diff nhỏ + hook tự động + review nghiêm."

Lời khuyên: nhấn mạnh dùng AI để **khuếch đại** năng lực, không **thay thế** trách nhiệm; nêu cụ thể cơ chế giữ chất lượng. Tránh kể "tôi để AI tự làm hết rồi nó chạy".

</details>

## 2. Prompting và Brainstorming với AI

### 2.1. Một prompt hiệu quả cho công cụ agentic coding (như Claude Code) gồm những thành phần nào? Hãy phân biệt một prompt "yếu" và một prompt "mạnh".

<details>
<summary>👉 Xem câu trả lời</summary>

- Prompt mạnh có đủ 4 yếu tố: mục tiêu rõ, ngữ cảnh (file/lỗi/kiến trúc), ràng buộc (quy ước, thư viện, điều cấm), định dạng đầu ra (diff / chỉ sửa file X / kèm test).
- Prompt yếu mô tả triệu chứng mơ hồ, để AI tự đoán; prompt mạnh thu hẹp không gian đoán bằng ngữ cảnh + ràng buộc cụ thể.
- Claude Code là agentic: không cần dán cả file, chỉ cần chỉ đường dẫn và nêu mục tiêu — agent tự đọc. Nhưng vẫn phải nêu rõ ràng buộc vì agent sẽ chủ động hành động.
- Bẫy: prompt yếu khiến AI tự "sáng tác" yêu cầu — thêm dependency, đổi kiến trúc, sửa file ngoài phạm vi.

| Prompt yếu | Prompt mạnh |
|---|---|
| "Sửa cái form login đi" | "Trong `LoginForm.tsx`, nút submit không disable khi gọi API. Hãy disable khi `isPending` của mutation chạy. Dùng React Query sẵn có, không thêm state thủ công." |

</details>

---

### 2.2. Cung cấp ngữ cảnh tốt nghĩa là gì? Trong Claude Code, có những cách nào để đưa ngữ cảnh vào ngoài việc gõ thẳng vào prompt?

<details>
<summary>👉 Xem câu trả lời</summary>

- Ngữ cảnh tốt = đủ để AI hiểu mà không phải đoán, nhưng không nhiễu. Ba loại quan trọng: file/code liên quan, lỗi nguyên văn (stack trace, log), mục tiêu mong muốn.
- Các kênh đưa ngữ cảnh trong Claude Code: chỉ đường dẫn file (agent tự `read`/`grep`); `CLAUDE.md` cho ngữ cảnh BỀN VỮNG (quy ước, kiến trúc, lệnh hay dùng); Memory cho sở thích qua nhiều phiên; dán lỗi/log cho vấn đề tức thời; MCP server kết nối nguồn ngoài (GitHub, DB, Linear).
- Nguyên tắc phân tầng: dùng MỘT LẦN → prompt; lặp MỌI PHIÊN → `CLAUDE.md`; sở thích bền vững → Memory.
- Bẫy: dán quá nhiều ("context dump") làm loãng tín hiệu; hoặc tả lỗi bằng lời thay vì dán stack trace nguyên văn.

```text
# Ngữ cảnh tốt: dán lỗi + chỉ chỗ
"Chạy `pnpm test auth.spec.ts` báo: toBeDisabled() — element not disabled.
File: auth.spec.ts:42 | Component: LoginForm.tsx
Mong muốn: nút disabled khi mutation chạy."
```

</details>

---

### 2.3. Vì sao nên chia nhỏ một vấn đề lớn khi làm việc với AI thay vì yêu cầu "làm hết một lượt"? Minh hoạ bằng một tính năng cụ thể.

<details>
<summary>👉 Xem câu trả lời</summary>

- Việc lớn nguyên khối → AI tự quyết định ngầm hàng loạt, khó kiểm tra, sai một chỗ phải làm lại cả khối.
- Chia nhỏ: mỗi bước phạm vi rõ, dễ review/sửa, giữ context window gọn nên AI ít "trôi" mục tiêu, bạn giữ quyền kiểm soát hướng đi.
- Claude Code: việc lớn dùng Plan mode (AI lập kế hoạch = chia nhỏ, bạn duyệt rồi mới thực thi); việc rất lớn nhiều bước tất định dùng Workflows; nhánh điều tra song song giao Subagents để cô lập context.
- Bẫy: chia nhỏ nhưng các mảnh phụ thuộc chéo chằng chịt — nên xác định ranh giới (interface) trước để mỗi bước kiểm thử độc lập được.

```text
# Tính năng login + 2FA, chia tuần tự:
1. Schema DB user + bảng OTP/secret 2FA
2. Endpoint login bước 1 -> token tạm
3. Sinh/gửi OTP (hoặc TOTP)
4. Endpoint xác thực OTP -> đổi lấy token thật
5. Test luồng thành công + lỗi (sai/hết hạn)
6. Rate limit chống brute-force
```

</details>

---

### 2.4. Hãy mô tả vòng lặp "lập và tinh chỉnh prompt" (iterate). Khi kết quả đầu tiên chưa đúng, bạn làm gì để hội thoại tiếp theo tốt hơn thay vì chỉ lặp lại?

<details>
<summary>👉 Xem câu trả lời</summary>

- Vòng lặp: ra prompt → đọc kết quả → chẩn đoán chỗ lệch → tinh chỉnh → ra lại. Mỗi vòng phải THÊM thông tin hoặc THU HẸP yêu cầu, không gõ lại y nguyên.
- Chiến thuật: chỉ rõ chỗ sai và VÌ SAO sai (không nói "vẫn chưa đúng"); bổ sung ràng buộc vừa nhận ra; đưa phản ví dụ; nếu lạc quá xa thì `/clear` rồi mở lại bằng prompt gọn, đầy đủ ngữ cảnh đã học.
- Mẹo: nếu một loại tinh chỉnh lặp lại nhiều lần ("luôn dùng React Query") → đưa vào `CLAUDE.md`.
- Bẫy: "lái mù" — nhấn lại mà không nói rõ sai ở đâu; hoặc thêm quá nhiều yêu cầu mới cùng lúc khiến sửa chỗ này hỏng chỗ kia. Việc khó nên tinh chỉnh từng khía cạnh.

```text
# Vòng 2 (thêm ràng buộc + chỉ đúng chỗ thiếu)
"Tốt, nhưng: (1) làm thành hook useDebouncedValue<T> generic;
 (2) cleanup timer trong useEffect khi đổi/unmount; (3) delay mặc định 300ms."
```

</details>

---

### 2.5. Few-shot prompting là gì và khi nào nó đặc biệt hữu ích trong công việc lập trình hằng ngày?

<details>
<summary>👉 Xem câu trả lời</summary>

- Few-shot: đưa MỘT VÀI ví dụ mẫu (input → output) ngay trong prompt để AI suy ra khuôn mẫu rồi áp dụng. Ngược với zero-shot (chỉ mô tả). Ví dụ giúp AI khớp đúng PHONG CÁCH và ĐỊNH DẠNG mà lời mô tả khó diễn đạt.
- Hữu ích khi: cần đầu ra theo quy ước riêng (đặt tên, cấu trúc), chuyển đổi dữ liệu lặp lại, hoặc khi một ví dụ nói thay được nhiều lời.
- Trong agentic coding, dạng few-shot tự nhiên là chỉ AI tham chiếu file mẫu sẵn có: "Tạo `UserService` theo cấu trúc `ProductService.ts`" — vừa là few-shot vừa tận dụng ngữ cảnh repo.
- Bẫy: ví dụ mâu thuẫn hoặc chất lượng kém — AI học theo cả lỗi. Ví dụ phải đại diện đúng thứ bạn muốn.

```text
"Viết test cho formatCurrency theo phong cách ví dụ:
describe('formatDate', () => { it('định dạng hợp lệ', ...); it('input lỗi -> chuỗi rỗng', ...); });"
```

</details>

---

### 2.6. Chain-of-thought (yêu cầu AI suy luận từng bước) là gì? Khi nào nên dùng, và khi nào nó không phù hợp?

<details>
<summary>👉 Xem câu trả lời</summary>

- CoT: yêu cầu AI trình bày các bước suy luận trước khi kết luận, thay vì nhảy thẳng tới đáp án. Bài nhiều bước thường chính xác hơn vì AI không bỏ qua bước trung gian.
- Nên dùng: debug bug khó cần lần luồng dữ liệu/trạng thái; so sánh nhiều phương án nhiều tiêu chí; bài có ràng buộc logic (thuật toán, điều kiện biên).
- Claude Code: Plan mode mang tinh thần CoT ở cấp quy trình — AI lập kế hoạch và lý giải trước khi đụng code, giúp bắt sai sót ở khâu suy luận.
- KHÔNG phù hợp: tác vụ đơn giản một bước (đổi tên biến, format) — tốn thời gian/token, làm chậm.
- Bẫy: nhầm "trình bày dài" với "suy luận đúng" — CoT chỉ lộ lập luận để BẠN kiểm tra, không bảo đảm đúng; vẫn phải phản biện.

```text
"Component re-render vô hạn. Trước khi sửa: (1) liệt kê mọi nguồn gây re-render;
 (2) lần theo effect nào set state nào và vì sao; (3) chỉ nguyên nhân gốc. Rồi mới sửa tối thiểu."
```

</details>

---

### 2.7. Brainstorming kỹ thuật cùng AI khác gì với việc nhờ AI viết code? Bạn dùng AI thế nào để đề xuất nhiều phương án và so sánh trade-off?

<details>
<summary>👉 Xem câu trả lời</summary>

- Brainstorming dùng AI ở giai đoạn KHÁM PHÁ (chưa biết hướng) để mở rộng phương án và phơi bày đánh đổi; cố tình giữ câu hỏi mở. Khác prompt viết code (đã chốt hướng, cần triển khai).
- Cách làm: yêu cầu NHIỀU phương án kèm bảng so sánh theo tiêu chí bạn quan tâm (độ phức tạp, hiệu năng, chi phí, mở rộng), buộc AI nêu rõ giả định và khuyến nghị.
- Claude Code: brainstorm hợp Plan mode (chưa sửa code); có thể giao Subagent Explore/Plan để fan-out khảo sát codebase mà không làm rối context chính.
- Bẫy: chốt ngay phương án đầu tiên mà không ép liệt kê lựa chọn khác — mất giá trị lớn nhất là thấy đánh đổi. Luôn yêu cầu "ít nhất N phương án" và "nêu nhược điểm của lựa chọn khuyến nghị".

| Phương án | Độ phức tạp | Độ trễ | Xử lý conflict |
|---|---|---|---|
| Polling | Thấp | Cao | Dễ ghi đè |
| WebSocket tự quản | TB | Thấp | Tự xử lý, dễ sai |
| Pub/sub managed | Thấp-TB | Thấp | Phụ thuộc dịch vụ |
| CRDT | Cao | Thấp | Tự hội tụ, học khó |

</details>

---

### 2.8. Vì sao nên để AI đóng vai "người phản biện" (devil's advocate / red-team) cho thiết kế của bạn? Cho ví dụ cách ra prompt.

<details>
<summary>👉 Xem câu trả lời</summary>

- AI mặc định hay "đồng thuận", xác nhận ý bạn. Hãy chủ động giao vai phản biện: tìm lỗ hổng, trường hợp biên, giả định sai, rủi ro vận hành trong THIẾT KẾ của bạn — biến AI thành người review giúp thấy điểm mù trước khi viết code.
- Biến thể mạnh: yêu cầu AI sinh kịch bản thất bại ("liệt kê 5 cách thiết kế này sập trong production"); hoặc dùng hai vai (bảo vệ / tấn công) rồi bạn làm trọng tài.
- Claude Code: tinh thần này được đóng gói thành Skill như code-review, security-review — gọi qua slash command để chạy quy trình rà soát có hệ thống.
- Bẫy: phản biện chung chung ("nên cẩn thận bảo mật"). Ép cụ thể: đúng dòng/kịch bản, kèm mức nghiêm trọng + cách khắc phục. Và AI phản biện cũng có thể sai/thổi phồng — bạn vẫn quyết định.

```text
"Đóng vai senior khó tính review thiết kế upload (presigned URL -> S3 -> confirm). Đừng khen.
Tìm: trường hợp biên (upload xong không confirm?), rủi ro bảo mật, file mồ côi/dọn rác.
Liệt kê từng vấn đề kèm mức nghiêm trọng + cách giảm thiểu."
```

</details>

---

### 2.9. Khi AI hiểu sai ý bạn (đi nhầm hướng, sửa nhầm file, "ảo giác" ra một API không tồn tại), bạn xử lý thế nào để kéo nó về đúng hướng?

<details>
<summary>👉 Xem câu trả lời</summary>

- Nhận ra vì sao sai: thường do prompt thiếu ràng buộc, ngữ cảnh mơ hồ, hoặc context đã "trôi" sau hội thoại quá dài.
- Xử lý theo mức độ: lệch nhẹ → nêu chính xác chỗ sai + bổ sung ràng buộc, không làm lại; "ảo giác" API → bắt AI tự kiểm chứng bằng nguồn thật (grep repo, đọc file types) thay vì trí nhớ; lạc xa → revert, `/clear`, mở lại bằng prompt gọn đủ ràng buộc.
- Phòng bệnh: Plan mode duyệt hướng trước khi sửa; đưa quy ước vào `CLAUDE.md`; để permission mode hỏi trước cho việc rủi ro; hành vi "tự động mỗi lần" (vd lint sau khi sửa) dùng Hook (harness chạy tất định) chứ đừng trông cậy AI tự nhớ.
- Bẫy: cố "vá" trong cuộc hội thoại đã rối; nhiều khi `/clear` rồi bắt đầu lại nhanh và sạch hơn. Luôn review diff, không merge code chưa hiểu.

```text
"`queryClient.invalidateAll()` không tồn tại. Mở types @tanstack/react-query
và dùng API có thật (invalidateQueries). Đừng đoán tên hàm."
```

</details>

---

### 2.10. Prompt để VIẾT CODE và prompt để bàn THIẾT KẾ / Ý TƯỞNG khác nhau ra sao? Bạn điều chỉnh cách viết prompt thế nào cho mỗi loại?

<details>
<summary>👉 Xem câu trả lời</summary>

- Hai loại có mục tiêu và tiêu chí "tốt" khác nhau nên cách viết prompt khác nhau (xem bảng).
- Quy tắc thực dụng: thiết kế đi TRƯỚC, code đi SAU — bắt đầu mở (brainstorm/Plan mode) để chốt hướng, rồi chuyển sang prompt code thật cụ thể. Trong Claude Code đúng là luồng Plan mode rồi mới sang chế độ sửa code.
- Bẫy: prompt thiết kế quá cụ thể (vô tình áp đặt một lời giải) làm AI không khám phá phương án khác; hoặc prompt code quá mở khiến AI phải tự đoán hàng loạt quyết định kỹ thuật và đi chệch.

| Tiêu chí | Prompt viết code | Prompt thiết kế |
|---|---|---|
| Mục tiêu | Chạy được, đúng spec | Khám phá phương án, thấy trade-off |
| Mức cụ thể | Rất cụ thể: file, hàm, ràng buộc | Mở: bối cảnh, mục tiêu, ràng buộc nghiệp vụ |
| Đầu ra | Code / diff / test | Danh sách phương án, bảng so sánh, khuyến nghị |
| Tính hội tụ | Hội tụ MỘT lời giải | Phân kỳ NHIỀU lựa chọn rồi chốt |

</details>

## 3. Claude Code Tổng quan

### 3.1. Claude Code là gì? Nó khác gì với một chatbot lập trình thông thường?

<details>
<summary>👉 Xem câu trả lời</summary>

- Claude Code là công cụ "agentic coding" của Anthropic, chạy trên các model Claude (Opus/Sonnet/Haiku/Fable).
- Khác biệt cốt lõi: hoạt động như một **agent** — không chỉ trả text mà tự **hành động** trong môi trường thật: đọc/sửa file, chạy lệnh shell, tìm trong codebase, gọi tool.
- Có **vòng lặp agentic** nhiều bước tới khi xong việc, tự khám phá ngữ cảnh (đọc `CLAUDE.md`), thay vì chỉ một lượt hỏi-đáp.

| Tiêu chí | Chatbot thường | Claude Code (agentic) |
|---|---|---|
| Tương tác | Dán code, trả về text | Tự đọc/sửa file trong dự án |
| Hành động | Không thực thi | Chạy lệnh, sửa file, gọi tool |
| Ngữ cảnh | Chỉ những gì bạn dán | Tự khám phá codebase, đọc `CLAUDE.md` |

Lưu ý phỏng vấn: nhấn từ khóa "agentic" — tự chủ, có vòng lặp hành động, làm trực tiếp trên codebase.

</details>

---

### 3.2. Claude Code có những dạng (form factor) nào? Khi nào nên dùng dạng nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- **CLI terminal**: phổ biến nhất; hợp người quen dòng lệnh, dễ tích hợp script/CI.
- **Extension IDE** (VS Code, JetBrains): tích hợp trong editor, tiện xem diff trực quan.
- **App desktop**: bản riêng cho Mac và Windows.
- **Web** (claude.ai/code): dùng nhanh qua trình duyệt, không cần cài đặt.
- Lưu ý: bản chất agentic giống nhau ở mọi dạng; khác biệt chủ yếu là giao diện và cách tích hợp.

</details>

---

### 3.3. "Vòng lặp agentic" (agentic loop) của Claude Code hoạt động như thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Là cách agent tự hoàn thành nhiệm vụ qua nhiều bước, mỗi vòng quyết định hành động dựa trên kết quả bước trước.
- Các bước: đọc yêu cầu + ngữ cảnh → chọn hành động (đọc/sửa file, chạy lệnh, tìm code, gọi tool) → thực thi (xin phép nếu cần) → nhận kết quả → đánh giá xong chưa → nếu chưa thì lặp lại.
- Chính vòng lặp này giúp agent "tự xoay sở" việc phức tạp nhiều bước — khác biệt cốt lõi với hỏi-đáp đơn lẻ.

```text
[Yêu cầu] -> đọc context -> chọn hành động -> thực thi (tool) -> nhận kết quả
                  ^________________ chưa xong _________________|
                                                          [Hoàn thành]
```

</details>

---

### 3.4. Permission mode là gì? Có những chế độ nào và dùng khi nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Permission mode** quy định khi nào agent cần **xin phép** trước khi hành động (vì nó tự sửa file, chạy lệnh).
- **Mặc định (hỏi trước)**: hỏi duyệt trước mỗi hành động có tác động — an toàn nhất.
- **`acceptEdits`**: tự duyệt sửa file, không hỏi từng lần — nhanh khi đã tin tưởng.
- **`plan`**: chỉ lập kế hoạch, không sửa code (xem câu 3.5).
- **`bypassPermissions`**: bỏ qua hỏi phép — mạnh nhưng rủi ro, chỉ dùng trong sandbox/CI cô lập.
- Nguyên tắc: chế độ càng "thoáng" thì môi trường càng phải cô lập (tránh mất dữ liệu nếu agent chạy nhầm lệnh).

</details>

---

### 3.5. Plan mode là gì? Tại sao nó hữu ích trước khi để agent sửa code?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Plan mode**: agent khám phá codebase, **trình bày kế hoạch** định làm, **trước khi** thực sự sửa file — chưa đụng file nào tới khi bạn duyệt.
- Lợi ích: kiểm soát hướng đi (tránh sửa hàng loạt sai hướng), phát hiện hiểu nhầm sớm, an toàn (không thay đổi tới khi đồng ý).
- Khi nào dùng: việc lớn, đụng nhiều file, hoặc sai một bước khó sửa. Việc nhỏ rõ ràng thì có thể thừa.
- Liên hệ: plan mode cũng là một permission mode — "khóa" giai đoạn sửa code cho tới khi duyệt kế hoạch.

</details>

---

### 3.6. So sánh Claude Code (agentic) với autocomplete kiểu Copilot (gợi ý từng dòng). Khác nhau cơ bản ở đâu?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Autocomplete (Copilot)** là "trợ lý gõ phím": gợi ý từng dòng/đoạn, *bạn* dẫn dắt từng bước, phạm vi chủ yếu file đang gõ.
- **Agentic (Claude Code)** là "đồng nghiệp": nhận cả nhiệm vụ nhiều bước, tự lên kế hoạch, tìm/sửa nhiều file, chạy lệnh + test, lặp tới khi xong.
- Hai cái không loại trừ nhau — điều phỏng vấn muốn nghe là bạn hiểu *khi nào* dùng cái nào (autocomplete lúc gõ chi tiết, agentic cho việc lớn cần tự chủ).

| Tiêu chí | Autocomplete | Agentic |
|---|---|---|
| Đơn vị | Từng dòng/đoạn | Cả nhiệm vụ nhiều bước |
| Phạm vi | File/dòng đang gõ | Cả codebase, nhiều file |
| Hành động | Chèn text gợi ý | Chạy lệnh, sửa file, chạy test |

</details>

---

### 3.7. Slash command là gì trong Claude Code? Cho ví dụ vài lệnh built-in và ý nghĩa.

<details>
<summary>👉 Xem câu trả lời</summary>

- **Slash command** là lệnh dạng `/tên` gõ trực tiếp để kích hoạt hành động/quy trình. Có loại **built-in** và **tùy biến**.
- Vài lệnh built-in: `/init` (tạo `CLAUDE.md` ghi rule dự án), `/review` (review PR/thay đổi), `/config` (cấu hình model, theme...), `/clear` (xóa ngữ cảnh phiên, bắt đầu sạch).
- Lệnh tùy biến cho quy trình lặp lại; đây cũng là cách Skills được gọi ra.
- Bẫy: quên `/clear` giữa các nhiệm vụ không liên quan → context cũ gây nhiễu, agent quyết định sai.

</details>

---

### 3.8. Việc chọn model trong Claude Code có ý nghĩa gì với hiệu quả công việc?

<details>
<summary>👉 Xem câu trả lời</summary>

- Mỗi model (Opus, Sonnet, Haiku, Fable) có đánh đổi giữa **tốc độ**, **chi phí** và **độ thông minh** — chọn đúng model là đòn bẩy hiệu quả.
- Việc đơn giản, lặp lại → model nhẹ/nhanh, rẻ (kiểu Haiku). Việc khó, refactor lớn, debug, agentic dài hơi → model mạnh nhất (Opus/Fable). Cân bằng → tầm trung (Sonnet).
- Đổi qua `/config` hoặc cấu hình trong `settings.json`.
- Điều phỏng vấn muốn thấy là *tư duy đánh đổi*: không mặc định dùng model mạnh nhất cho mọi việc (lãng phí), cũng không dùng model yếu cho việc khó (kết quả kém).

```text
Đổi tên biến 1 file          -> model nhẹ/nhanh (Haiku)
Viết service mới + test      -> model cân bằng (Sonnet)
Refactor nhiều module        -> model mạnh nhất (Opus/Fable)
```

</details>

---

### 3.9. Hãy kể về một lần bạn dùng Claude Code (hoặc công cụ agentic) để xử lý một việc mà cách làm thủ công sẽ rất tốn thời gian.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Câu hỏi hành vi — trả lời theo cấu trúc STAR để thể hiện coi AI là "siêu năng lực".

- **Situation (Bối cảnh)**: tình huống ngắn gọn. Ví dụ: đổi convention đặt tên toàn bộ DTO trong dự án NestJS, rải rác trên hơn 40 file.
- **Task (Nhiệm vụ)**: mục tiêu + ràng buộc. Ví dụ: đổi tên nhất quán, cập nhật import, không hỏng test, xong trong buổi chiều để kịp release.
- **Action (Hành động)** — phần quan trọng nhất, thể hiện dùng agentic *có chiến lược*:
  - Bật **plan mode** để agent khảo sát và trình kế hoạch trước, tôi duyệt để không bỏ sót.
  - Để agent chạy **vòng lặp agentic**: tìm chỗ dùng, sửa file, tự chạy test kiểm chứng.
  - Dùng **permission mode** hợp lý: đầu hỏi trước, sau chuyển `acceptEdits` khi đã tin tưởng.
  - Dùng `CLAUDE.md` để agent nắm convention dự án.
- **Result (Kết quả)**: định lượng nếu được. Ví dụ: nửa ngày làm tay rút còn ~1 giờ, test pass hết, không sót import; vẫn review kỹ diff trước khi commit.

Mẹo gây ấn tượng: nhấn rằng bạn không "phó mặc" cho AI — dùng plan mode để kiểm soát, permission mode để cân bằng tốc độ/an toàn, và luôn review. Đó là dấu hiệu dùng AI *có trách nhiệm và có kỹ năng*.

</details>

## 4. Claude Code Skills và Slash Commands

### 4.1. Skill trong Claude Code là gì? Nó khác gì so với một slash command thông thường, và mối quan hệ giữa hai khái niệm này ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

- Skill: "kỹ năng" tái sử dụng, đóng gói một quy trình chuyên biệt (code review, deep research...), định nghĩa trong file `SKILL.md` (frontmatter + phần thân hướng dẫn).
- Slash command: cơ chế *gọi* — gõ `/tên` để kích hoạt một hành động.
- Quan hệ: slash command là *giao diện gọi* nhiều thứ (built-in, lệnh tùy biến, và skill); skill là *thứ được gọi*. Cài skill `deep-research` → gọi bằng `/deep-research`.
- Không phải mọi slash command đều là skill, nhưng nhiều skill được phơi bày qua slash command.

| Tiêu chí | Slash command | Skill |
|---|---|---|
| Bản chất | Cách gọi (`/tên`) | Quy trình đóng gói tái dùng |
| Định nghĩa | Built-in/tùy loại | File `SKILL.md` |
| Ví dụ | `/clear`, `/init` | `/code-review` |

Bẫy: nghĩ skill và slash command là hai thứ tách biệt hoàn toàn — thực ra chúng giao nhau.

</details>

---

### 4.2. File `SKILL.md` gồm những phần nào? Vai trò của frontmatter `name` và `description` là gì, và tại sao `description` lại quan trọng đến vậy?

<details>
<summary>👉 Xem câu trả lời</summary>

- `SKILL.md` gồm: frontmatter (bắt buộc `name`, `description`) và phần thân (hướng dẫn chi tiết, các bước, quy ước).
- `description` luôn nằm trong context theo cơ chế "progressive disclosure": Claude đọc nó để quyết định *khi nào* dùng skill; chỉ khi cần mới nạp toàn bộ body → context nền gọn.
- Vì vậy `description` phải nói rõ cả "làm gì" lẫn "khi nào dùng".

```markdown
---
name: pr-review-frontend
description: Review PR frontend React/Next.js (re-render thừa, a11y,
  ranh giới Server/Client). Dùng khi review code FE trước khi merge.
---
# Quy trình review PR frontend
...
```

Bẫy: mô tả chỉ nêu "cái gì" mà quên "khi nào" → skill không tự kích hoạt đúng lúc; hoặc nhồi hết hướng dẫn vào `description` → phình context, mất ý nghĩa progressive disclosure.

</details>

---

### 4.3. Skill built-in khác skill tùy biến (custom) ở điểm nào? Cho ví dụ và nói rõ khi nào nên tự viết skill.

<details>
<summary>👉 Xem câu trả lời</summary>

- Built-in: Anthropic cung cấp sẵn, dùng ngay (`code-review`, `deep-research`, `security-review`).
- Custom: bạn tự viết `SKILL.md`, đặt ở cấp user (`~/.claude`), project (`.claude`), hoặc qua plugin.
- Khi nào tự viết: khi lặp lại cùng một prompt dài / quy trình nhiều bước (vd "scaffold feature module", "kiểm tra Server/Client boundary").
- Lợi ích đóng gói: nhất quán + chia sẻ được (đặt ở `.claude` cho cả team).

| Loại | Đặt ở đâu | Khi dùng |
|---|---|---|
| Built-in | Có sẵn | Quy trình phổ biến |
| Custom (user) | `~/.claude` | Quy ước cá nhân |
| Custom (project) | `.claude` | Quy trình riêng dự án, share team |

Bẫy: viết skill quá rộng ("làm mọi thứ về frontend") — skill nên có phạm vi rõ, một mục tiêu (như hàm thuần, không phải "God object").

</details>

---

### 4.4. Skill khác với một prompt thông thường (gõ trực tiếp vào chat) ở những điểm nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Tái sử dụng: prompt phải gõ lại mỗi lần; skill gọi lại bằng `/tên`.
- Lưu trữ: prompt chỉ tồn tại trong hội thoại; skill lưu thành file trên đĩa.
- Nạp context: prompt vào context toàn bộ ngay; skill dùng progressive disclosure (chỉ `description` thường trú, body nạp khi cần).
- Chia sẻ & nhất quán: skill commit vào `.claude`, mọi lần chạy giống nhau.

| Tiêu chí | Prompt gõ trực tiếp | Skill |
|---|---|---|
| Tái dùng | Gõ lại mỗi lần | Gọi `/tên` |
| Context | Vào hết ngay | `description` thường trú, body khi cần |

Lưu ý quan trọng: skill/prompt là *hướng dẫn khi được gọi*. Muốn hành vi *tự động theo sự kiện* (vd chạy ESLint/Prettier sau khi sửa file) → dùng **hook** (`settings.json`), vì harness mới tự chạy, không trông cậy AI "nhớ" gọi skill.

Bẫy: kỳ vọng skill "tự động chạy" theo lịch/sự kiện — skill không phải cron hay trigger.

</details>

---

### 4.5. Slash command built-in và slash command tùy biến khác nhau thế nào? Kể vài lệnh built-in hay dùng và công dụng của chúng.

<details>
<summary>👉 Xem câu trả lời</summary>

- Built-in: lệnh có sẵn, do Anthropic định nghĩa/bảo trì.
- Tùy biến: bạn tự định nghĩa, hoặc đến từ skill/plugin (cài skill → xuất hiện như slash command).
- Cả hai gọi giống nhau (`/tên`); khác ở ai kiểm soát nội dung.

```text
/init    -> sinh CLAUDE.md (kiến trúc, lệnh build/test, quy ước)
/clear   -> dọn context khi chuyển tác vụ, tránh "nhiễu"
/review  -> review PR trước khi merge
/config  -> đổi cấu hình (vd model: Opus việc khó, Haiku việc nhẹ)
```

Mẹo: `/clear` hữu ích giữa các tác vụ độc lập — context cũ tích lũy làm câu trả lời "trôi dạt".

Bẫy: nhầm slash command với lệnh shell — `/review` là lệnh của Claude Code, không chạy trong terminal OS.

</details>

---

### 4.6. Skill có thể tổ chức theo những cấp nào (user / project)? Khi cùng tên thì cấp nào nên ưu tiên, và vì sao tách cấp lại quan trọng với một team?

<details>
<summary>👉 Xem câu trả lời</summary>

- Cấp user (`~/.claude`): áp dụng cá nhân, xuyên mọi dự án; không share team.
- Cấp project (`.claude`): riêng dự án, commit vào Git → cả team dùng chung.
- Ngoài ra còn built-in và plugin.
- Ưu tiên khi cùng tên: cấp gần ngữ cảnh hơn (project) ghi đè/ưu tiên cấp xa (user/enterprise) — giống cách `CLAUDE.md` hoạt động.

| Cấp | Vị trí | Share team? |
|---|---|---|
| User | `~/.claude` | Không |
| Project | `.claude` | Có (commit Git) |

Ý nghĩa với team: quy ước riêng dự án để ở `.claude` (commit) → mọi người mở repo là có ngay, không cấu hình lại; thói quen cá nhân để ở `~/.claude`.

Bẫy: để quy ước riêng dự án ở cấp user → người khác trong team không có → code không nhất quán.

</details>

---

### 4.7. Plugin là gì trong bối cảnh skill, và nó giúp gì cho việc chia sẻ/phân phối skill trong tổ chức?

<details>
<summary>👉 Xem câu trả lời</summary>

- Plugin: cách đóng gói và phân phối lại năng lực mở rộng (gồm skill) cho Claude Code — gom nhiều skill thành một đơn vị cài đặt được, dùng cho nhiều người/nhiều máy.
- Quan hệ: `SKILL.md` định nghĩa skill → skill gọi qua slash command → plugin là một nguồn cung cấp skill (cạnh built-in, user, project).
- Ví dụ: công ty có bộ quy chuẩn frontend → đóng gói thành plugin để mọi team cài một lần, thay vì copy-paste `SKILL.md` vào từng repo.

| Cách | Phạm vi | Phù hợp khi |
|---|---|---|
| Project (`.claude`) | Một repo, qua Git | Quy trình riêng dự án |
| User (`~/.claude`) | Một máy/cá nhân | Thói quen cá nhân |
| Plugin | Nhiều người/dự án | Phân phối như "gói" dùng chung |

Bẫy: nhầm plugin với MCP server. MCP là chuẩn kết nối Claude tới *tool/dữ liệu ngoài* (GitHub, DB, Figma); plugin là cách *đóng gói/phân phối năng lực Claude Code* (như skill).

</details>

---

### 4.8. Hãy kể về một lần bạn dùng skill/slash command (hoặc đóng gói quy trình lặp lại thành công cụ tái dùng) để tăng tốc một quy trình lặp đi lặp lại trong nhóm.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Câu hỏi hành vi — kể theo STAR, ưu tiên có số liệu. Mẫu để điều chỉnh theo trải nghiệm thật:

- **Situation**: Nhóm FE 4 người làm app Next.js. Mỗi lần review PR phải nhắc lại cùng checklist (re-render thừa, ranh giới Server/Client, a11y, không lộ secret) — tốn thời gian, mỗi người review một kiểu, không nhất quán.
- **Task**: Chuẩn hóa quy trình review để nhanh hơn, đồng đều giữa reviewer, giảm bug lặp lại lọt qua.
- **Action**: Viết skill tùy biến ở cấp project (`.claude`), `description` nêu rõ "dùng khi review PR FE trước merge", body chứa checklist chi tiết; commit vào Git. Reviewer chỉ gõ một slash command. Việc *tự động theo sự kiện* (lint/format sau sửa file) tách riêng dùng **hook** trong `settings.json`, không nhồi vào skill.
- **Result**: Thời gian review/PR giảm rõ, review đồng đều giữa các thành viên; bug re-render thừa và rò rỉ client/server giảm hẳn. Mở rộng nhiều team thì đóng gói thành plugin.

Lời khuyên: chọn chuyện thật, nêu con số (thời gian tiết kiệm, bug giảm), và làm nổi tư duy chọn đúng công cụ — skill cho quy trình tái dùng, hook cho hành vi tự động theo sự kiện, plugin cho phân phối quy mô lớn.

</details>

## 5. Claude Code Agents và Subagents

### 5.1. Subagent trong Claude Code là gì, và vì sao việc "cô lập context" (context isolation) lại quan trọng?

<details>
<summary>👉 Xem câu trả lời</summary>

- Subagent là agent con chạy trong một CONTEXT WINDOW RIÊNG, tách biệt hội thoại chính: tự đọc/sửa file, chạy lệnh, gọi tool, rồi chỉ trả về bản tóm tắt cho agent cha.
- Cô lập quan trọng vì context window là hữu hạn: các bước trung gian ồn ào (nội dung file, log, kết quả tìm kiếm) nằm trong context con, không làm loãng context chính.
- Nhờ đó hội thoại chính giữ "tín hiệu cao, nhiễu thấp" và đi xa hơn mà không bị đầy.
- Bẫy: subagent KHÔNG thấy lịch sử hội thoại chính, chỉ biết những gì agent cha mô tả khi giao việc — nên prompt giao việc phải đầy đủ, độc lập.

```text
Agent CHÍNH ──giao việc──> Subagent (CONTEXT RIÊNG)
   grep repo, đọc 12 file...   ← ồn ào, ở context con
   trả về: "Gọi ở 3 nơi: auth.guard.ts:21, ws.ts:88, jobs.ts:5"  ← chỉ kết luận
```

</details>

---

### 5.2. Claude Code có những loại agent nào sẵn có? Phân biệt general-purpose, Explore và Plan.

<details>
<summary>👉 Xem câu trả lời</summary>

- general-purpose: agent "vạn năng" đủ tool (đọc + sửa + chạy lệnh), dùng cho việc phức tạp nhiều bước.
- Explore: chuyên tìm kiếm/đọc hiểu codebase (thiên read-only), mạnh fan-out, kết quả là "bản đồ" code — dùng khi chỉ cần biết "code ở đâu / chạy thế nào".
- Plan: chuyên lập kế hoạch/thiết kế trước khi viết code (tinh thần "plan mode") — dùng khi cần đề án/kiến trúc trước.
- Bẫy: dùng general-purpose cho việc chỉ-đọc đơn giản vừa chậm vừa rủi ro nó tự sửa file; chỉ tra cứu thì Explore an toàn hơn.

```text
"Repo xử lý refresh token ở đâu?"     -> Explore (chỉ tìm)
"Thiết kế thêm rate-limit cho API"    -> Plan (ra kế hoạch)
"Sửa module auth theo kế hoạch X"     -> general-purpose (đọc + sửa + test)
```

</details>

---

### 5.3. Khi nào nên giao việc cho subagent thay vì để agent chính tự làm?

<details>
<summary>👉 Xem câu trả lời</summary>

- Việc "đọc nhiều, trả về ít": khám phá tốn nhiều context nhưng kết quả gọn (vd lùng cả repo tìm chỗ định nghĩa một symbol).
- Việc chia được thành nhiều nhánh ĐỘC LẬP, chạy song song (fan-out).
- Việc cần một "vai" chuyên biệt với tool/chỉ dẫn riêng (vd review bảo mật, deep research).
- Việc dài/phức tạp, muốn cô lập để nếu "đi lạc" không hỏng dòng suy luận chính.
- Bẫy: lạm dụng subagent. Đọc 1 file rồi sửa 1 dòng thì tự làm nhanh hơn chi phí khởi tạo + giao việc + chờ tóm tắt.

Ví dụ: trước khi refactor `AuthService`, giao Explore subagent tìm "mọi nơi import AuthService và gọi validateUser" — agent chính nhận danh sách để lên kế hoạch, context không bị nhồi 15 file.

</details>

---

### 5.4. Chạy song song nhiều subagent (fan-out) hoạt động ra sao, và có lợi gì?

<details>
<summary>👉 Xem câu trả lời</summary>

- Fan-out: agent chính khởi tạo NHIỀU subagent cùng lúc, mỗi cái lo một nhánh độc lập, rồi gom kết quả.
- Lợi ích: giảm thời gian thực (chạy đồng thời), cô lập context theo nhánh (nhánh A không loãng nhánh B), hợp với "bản đồ hóa" codebase.
- Điều kiện: chỉ song song khi các nhánh ĐỘC LẬP; nếu nhánh B cần kết quả nhánh A thì phải tuần tự (giống `Promise.all` chỉ cho tác vụ độc lập).
- Bẫy: fan-out rồi cùng ghi vào một file dễ xung đột. Fan-out hợp việc CHỈ ĐỌC / phân tích độc lập; việc ghi có phụ thuộc thì tuần tự hoặc để agent chính điều phối cẩn thận.

```text
            Agent CHÍNH
     ┌──────────┼──────────┐
  Subagent A  Subagent B  Subagent C   (3 context, SONG SONG)
     └──────────┼──────────┘
          gộp 3 tóm tắt -> quyết định
```

</details>

---

### 5.5. Tự định nghĩa một subagent trong `.claude/agents` như thế nào? Có những gì cần khai báo?

<details>
<summary>👉 Xem câu trả lời</summary>

- Đặt file định nghĩa agent trong `.claude/agents` (cấp project, theo repo) hoặc `~/.claude/agents` (cấp user, xuyên dự án).
- File khai báo "vai": `name`, `description` (khi nào nên dùng), và system prompt định hình hành vi; có thể giới hạn tool và (tùy cấu hình) chỉ định model.
- Bản chất: agent tùy biến = subagent có context riêng + bộ chỉ dẫn chuyên biệt, đóng gói một quy trình lặp lại để Claude tự gọi khi gặp việc phù hợp.
- Bẫy: `description` mơ hồ khiến Claude không biết khi nào gọi — nên nêu rõ "dùng khi…". Lưu ý: agent chỉ định hình HÀNH VI; "tự động chạy theo sự kiện" là việc của hook.

```markdown
---
name: test-writer
description: Dùng khi cần viết/bổ sung unit test cho service NestJS.
---
Bạn là agent viết test cho dự án NestJS + Prisma.
- Dùng Jest, đặt *.spec.ts cạnh file nguồn; mock PrismaService.
- Bao phủ cả nhánh thành công lẫn lỗi (vd NotFoundException).
```

</details>

---

### 5.6. Subagent khác gì với Skill và với CLAUDE.md? Khi nào dùng cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Subagent/Agent: tiến trình con có CONTEXT RIÊNG, chạy song song được — dùng khi cần cô lập context hoặc fan-out việc lớn/độc lập.
- Skill: "kỹ năng" tái sử dụng định nghĩa trong `SKILL.md`, gọi qua slash command — dùng khi cần một QUY TRÌNH gọi lại theo tên (vd code-review, deep-research).
- CLAUDE.md: file ngữ cảnh/rule tự nạp vào context — dùng để cung cấp quy ước code, lệnh build/test, kiến trúc thư mục.
- Có thể kết hợp: subagent vẫn đọc CLAUDE.md, Skill chạy được trong subagent.
- Bẫy: nhầm subagent với CLAUDE.md — CLAUDE.md KHÔNG tạo context riêng, chỉ thêm thông tin vào context hiện có.

```text
"Luôn dùng pnpm, đặt DTO trong /dto"      -> CLAUDE.md
"Quy trình gọi bằng /security-review"      -> Skill
"Tìm song song 3 module xử lý lỗi"         -> fan-out Explore subagent
```

</details>

---

### 5.7. Subagent quan hệ thế nào với Workflows và với MCP? Phân biệt điều phối "agent tự quyết" và điều phối "tất định".

<details>
<summary>👉 Xem câu trả lời</summary>

- Điều phối "agent tự quyết": agent chính tự QUYẾT ĐỊNH lúc chạy có tạo subagent không, bao nhiêu, giao gì — linh hoạt, hợp việc mơ hồ khó đặc tả trước.
- Workflows (tất định): LẬP TRÌNH bằng script để điều phối nhiều agent cố định (bước nào trước, fan-out ra sao, gom thế nào) — hợp việc lớn, lặp lại, cần tin cậy và tái lập.
- MCP (Model Context Protocol): chuẩn mở KẾT NỐI AI với tool/dữ liệu ngoài qua "MCP server" (expose tools, resources, prompts). MCP không phải agent, nó MỞ RỘNG tool cho agent/subagent (GitHub, database, Linear, Figma, Notion...).
- Liên hệ Claude API: cũng có model id (claude-opus, claude-sonnet, claude-haiku, claude-fable), tool use, structured output, hỗ trợ agents và MCP — kiến trúc đa agent ánh xạ được sang API.
- Bẫy: chọn workflow tất định cho việc bản chất mơ hồ sẽ gãy; để agent "tự quyết" cho quy trình production cần tái lập chính xác thì khó kiểm soát. Chọn theo độ xác định của bài toán.

</details>

---

### 5.8. Subagent có những ưu/nhược điểm gì, và khi nào KHÔNG nên dùng subagent?

<details>
<summary>👉 Xem câu trả lời</summary>

- Ưu: cô lập context (hội thoại chính sạch, đi xa hơn); song song hóa (fan-out giảm thời gian); chuyên môn hóa (mỗi agent một vai); hạn chế rủi ro (nhánh lạc không kéo cả dòng chính).
- Nhược/chi phí: tốn chi phí khởi tạo + giao việc + chờ tóm tắt; mất ngữ cảnh ngầm (không thấy lịch sử, giao thiếu sẽ làm lệch); khó phối hợp khi có phụ thuộc; tóm tắt làm mất chi tiết.
- KHÔNG nên dùng: việc nhỏ 1-2 thao tác; việc bám sát ngữ cảnh hội thoại; các bước phụ thuộc tuần tự; cần hành vi "tự động theo sự kiện".
- Bẫy: hành vi "luôn làm X mỗi khi Y" phải dùng hook (PreToolUse, PostToolUse, Stop...) trong settings.json — hook được harness tự chạy theo sự kiện, còn subagent chỉ chạy khi được giao việc.

```text
NÊN:    "đọc-nhiều-trả-ít" + độc lập + cô lập context có lợi
KHÔNG:  việc nhỏ / phụ thuộc ngữ cảnh / tuần tự / cần tự-động-theo-sự-kiện
```

</details>

## 6. Claude Code Rules CLAUDE.md và Memory

### 6.1. File `CLAUDE.md` dùng để làm gì, và tại sao nó hiệu quả hơn việc nhắc lại ngữ cảnh trong từng prompt?

<details>
<summary>👉 Xem câu trả lời</summary>

- `CLAUDE.md` là file "rule"/ngữ cảnh dự án mà Claude Code **tự động nạp vào context** ở đầu mỗi phiên (không cần dán lại).
- Chứa tri thức bền vững: coding conventions, lệnh hay dùng (build/test/lint), kiến trúc, điều "đừng làm" -> agent hành xử nhất quán.
- **Không bị quên giữa phiên**: rule luôn hiện diện, không trôi khỏi context như khi chỉ nói ở tin nhắn đầu.
- **Tái sử dụng**: commit vào repo thì cả team và mọi phiên dùng chung; **giảm prompt lặp lại**.
- **Bẫy**: nhồi quá dài (loãng tín hiệu, tốn context) hoặc viết rule chung chung kiểu "code sạch". Hãy ngắn, cụ thể, kiểm chứng được.

```md
# Jira Clone API (NestJS + Prisma)
## Lệnh
- Test: `pnpm test` (đừng dùng `npm`)
## Quy ước
- Dùng DI, KHÔNG khởi tạo service bằng `new`.
- Business logic ở service, KHÔNG ở controller.
```

</details>

---

### 6.2. Làm sao để viết rule tốt trong `CLAUDE.md`? Nêu nguyên tắc và những loại thông tin nên đưa vào.

<details>
<summary>👉 Xem câu trả lời</summary>

- Mục tiêu: **ngắn, cụ thể, actionable** (đọc xong biết phải làm gì).
- Nội dung nên đưa: **lệnh hay dùng** (build/run/test/lint/migrate); **coding conventions**; **kiến trúc** (tầng, module, luồng dữ liệu); **điều CẤM/cảnh báo**.
- Nguyên tắc: cụ thể kiểm chứng được ("dùng `pnpm` chứ không `npm`"); có cấu trúc heading/bullet; IN HOA/"LUÔN/KHÔNG BAO GIỜ" cho ràng buộc cứng; cập nhật khi thực tế đổi (rule lỗi thời nguy hiểm hơn không có rule).
- **Mẹo**: chạy `/init` để Claude Code sinh bản nháp rồi tinh chỉnh.

| Rule kém | Rule tốt |
|---|---|
| "Viết test đầy đủ" | "Mọi service mới phải có unit test; chạy `pnpm test` trước khi báo xong" |
| "Cẩn thận với DB" | "KHÔNG sửa migration đã tồn tại; luôn tạo mới bằng `prisma migrate dev`" |

</details>

---

### 6.3. `CLAUDE.md` có phân cấp như thế nào (enterprise / user / project / thư mục con) và chúng kết hợp ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Enterprise**: rule toàn công ty (bảo mật, chính sách chung).
- **User** (`~/.claude/CLAUDE.md`): sở thích cá nhân, áp cho mọi dự án; **không commit** vào repo.
- **Project** (`CLAUDE.md` gốc repo): quy ước dự án, **commit vào git** cho cả team.
- **Thư mục con**: ngữ cảnh hẹp cho phần code đó (ví dụ `packages/payment`).
- Cách kết hợp: các cấp được **gộp lại** (nạp chồng), không loại trừ; cấp hẹp/cụ thể hơn bổ sung hoặc tinh chỉnh cấp rộng.
- **Bẫy**: nhét sở thích cá nhân vào `CLAUDE.md` project rồi commit -> áp đặt thói quen riêng lên đồng đội. Sở thích cá nhân để ở cấp user.

</details>

---

### 6.4. Memory khác `CLAUDE.md` ở điểm nào? Khi nào nên dùng Memory, khi nào nên dùng `CLAUDE.md`?

<details>
<summary>👉 Xem câu trả lời</summary>

- **`CLAUDE.md`**: file rule **do bạn chủ động viết và kiểm soát**, thường trong repo, review qua git -> định hình hành vi (conventions, lệnh, kiến trúc).
- **Memory**: cơ chế **ghi nhớ sự kiện/sở thích bền vững qua nhiều phiên**, tích lũy trong lúc làm việc (ví dụ "dự án dùng Postgres cổng 5433").
- Khi nào dùng: quy ước cứng cần team review -> `CLAUDE.md` (commit); bối cảnh/sở thích tích lũy dần, không muốn tự biên tập -> Memory.
- **Lưu ý**: cả hai là **ngữ cảnh thụ động** — chỉ *mô tả*, KHÔNG đảm bảo "tự động chạy lệnh X mỗi khi Y". Cần hành vi tự động chắc chắn thì dùng **hook** (xem 6.6).

| Tiêu chí | `CLAUDE.md` | Memory |
|---|---|---|
| Bản chất | File rule do người viết | Ghi nhớ sự kiện/sở thích |
| Quản lý | Sửa thủ công, review qua git | Tích lũy/cập nhật khi dùng |
| Mục đích | Quy ước cứng, định hình hành vi | Nhớ ngữ cảnh, bối cảnh |

</details>

---

### 6.5. "Tránh rule mâu thuẫn" nghĩa là gì? Điều gì xảy ra khi rule xung đột và làm sao phòng tránh?

<details>
<summary>👉 Xem câu trả lời</summary>

- Rule đến từ nhiều cấp (enterprise/user/project/thư mục con) cộng Memory -> dễ **mâu thuẫn** (hai chỉ dẫn ngược nhau).
- Hậu quả: agent phải "đoán" theo cái nào -> hành vi không nhất quán, khó dự đoán, mất niềm tin.
- Phòng tránh: **phân tầng đúng vai trò** (cá nhân ở user, dự án ở project); **một nguồn sự thật** cho mỗi quyết định; rule cụ thể thắng rule chung; **định kỳ dọn dẹp** rule lỗi thời; diễn đạt rõ ("ưu tiên X trừ khi Y").
- **Bẫy**: copy `CLAUDE.md` từ dự án cũ mà quên sửa; Memory giữ sở thích cũ ("dùng REST") trong khi dự án đã chuyển GraphQL.

```md
# ~/.claude/CLAUDE.md (user): Luôn dùng npm.
# ./CLAUDE.md (project): Dự án dùng pnpm; KHÔNG npm (phá lockfile).
# -> Xung đột package manager: dù project thường thắng, vẫn là rủi ro.
```

</details>

---

### 6.6. Phân biệt project settings và user settings (`settings.json` vs `settings.local.json`). Cái gì nên ở đâu, và vì sao "tự động làm X" phải dùng hook chứ không phải rule?

<details>
<summary>👉 Xem câu trả lời</summary>

- `settings.json` quy định permissions, env, hooks, model — cũng có phân cấp:
  - **User** (`~/.claude/settings.json`): cấu hình cá nhân cho mọi dự án.
  - **Project** (`.claude/settings.json`): dùng chung cả team, **commit vào git**.
  - **Project local** (`.claude/settings.local.json`): ghi đè cá nhân cho riêng dự án này, **không commit** (gitignore).
- Phân biệt: `CLAUDE.md` là **ngữ cảnh ngôn ngữ tự nhiên** (agent đọc/hiểu); `settings.json` là **cấu hình harness** (chương trình thực thi tất định).
- **"Tự động làm X mỗi khi Y" phải dùng hook**: rule chỉ *nhờ* AI nhớ (xác suất, có thể quên); **hook** là lệnh shell do **harness tự chạy** khi có sự kiện (`PostToolUse`, `PreToolUse`, `Stop`...) -> chạy **tất định**.
- **Bẫy**: viết "luôn chạy lint sau khi sửa" trong `CLAUDE.md` rồi ngạc nhiên khi agent bỏ qua; hoặc commit `settings.local.json` chứa thông tin máy cá nhân.

```jsonc
// .claude/settings.json — tự format file vừa sửa
{ "hooks": { "PostToolUse": [ {
  "matcher": "Edit|Write",
  "hooks": [{ "type": "command",
    "command": "jq -r '.tool_input.file_path' | xargs pnpm prettier --write" }]
} ] } }
```

</details>

---

### 6.7. Hãy đưa một ví dụ `CLAUDE.md` thực tế "đáng giá" cho dự án backend, và giải thích vì sao mỗi mục giúp ích cho agent.

<details>
<summary>👉 Xem câu trả lời</summary>

- File "đáng giá" = nếu thiếu, agent sẽ hỏi lại hoặc làm sai. Tập trung vào **tri thức ngầm** (quyết định, ràng buộc, lệnh đặc thù) mà agent không đoán được khi đọc code.
- **Tech stack + lệnh**: agent biết cách build/test/migrate để **tự kiểm chứng**, không dùng sai công cụ.
- **Database**: tri thức ngầm chỉ người trong dự án biết (cổng 5433) -> tránh kết nối nhầm 5432.
- **Quy ước kiến trúc**: ép logic đúng tầng, giữ codebase nhất quán.
- **Điều CẤM (migration)**: ngăn lỗi khó đảo ngược. **"Trước khi báo xong"**: định nghĩa rõ "hoàn thành", giúp agent tự verify.
- **Bẫy**: viết thứ agent tự suy ra ("đây là dự án TypeScript") -> vô ích, tốn context. Phần "tự động chạy lint" nếu muốn chắc chắn -> chuyển sang hook (xem 6.6).

```md
# Jira Clone API
## Lệnh: Dev `pnpm start:dev` | Test `pnpm test` | Migration `pnpm prisma migrate dev`
## Database (LƯU Ý): Postgres Docker cổng 5433 (5432 bị chiếm); đọc từ `.env`, KHÔNG hardcode.
## Quy ước: controller -> service -> repository; mọi endpoint có DTO; KHÔNG sửa migration đã commit.
## Trước khi báo "xong": chạy `pnpm lint` và `pnpm test`, đảm bảo pass.
```

</details>

## 7. Claude Code Hooks và Tự động hóa

### 7.1. Hook trong Claude Code là gì? Nó hoạt động theo cơ chế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Hook là một lệnh shell do **harness** của Claude Code (không phải model AI) tự động chạy khi xảy ra một event trong vòng đời phiên làm việc.
- Cấu hình trong `settings.json`, gắn với một event; mỗi khi event đó xảy ra harness sẽ chạy lệnh một cách **tất định** (deterministic) — luôn chạy, không phụ thuộc model có nhớ hay không.
- Đây là khác biệt lớn về độ tin cậy: model có thể quên/diễn giải sai; hook là quy tắc cứng.
- Các event phổ biến: `PreToolUse` (trước khi chạy tool), `PostToolUse` (sau khi chạy tool), `UserPromptSubmit` (khi gửi prompt), `Stop` (khi agent dừng), `SessionStart` (đầu phiên).
- Bẫy: nghĩ hook là "một loại `/command` trong chat" — sai; hook là cấu hình hệ thống, chạy ngầm khi event xảy ra.

```jsonc
// .claude/settings.json — format sau mỗi lần Claude ghi/sửa file
{
  "hooks": {
    "PostToolUse": [
      { "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": ".claude/hooks/format.sh" }] }
    ]
  }
}
```

</details>

---

### 7.2. Tại sao một hành vi "tự động mỗi khi X" PHẢI dùng hook, chứ không phải nhờ AI ghi nhớ?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Nhờ AI ghi nhớ** (CLAUDE.md, Memory, prompt) là chỉ dẫn mềm: model *có thể* làm theo nhưng cũng có thể quên, bỏ qua khi context dài — **không đảm bảo** (non-deterministic).
- **Dùng hook**: harness *luôn luôn* chạy khi event xảy ra — đảm bảo cứng, tất định.
- Quy tắc vàng: yêu cầu dạng "mỗi khi X thì luôn làm Y", "trước/sau mỗi lần X", "tự động Y" → phải dùng hook. Ghi nhớ chỉ hợp cho sở thích/ngữ cảnh (vd "tôi thích pnpm").
- Lý do sâu xa: model là thành phần xác suất; không thể xây quy trình chắc chắn 100% trên nó. Hook tách phần "đảm bảo" sang harness (phần tất định).
- Bẫy: viết "QUAN TRỌNG: BẠN PHẢI luôn..." trong CLAUDE.md rồi tưởng chắc chắn — câu chữ mạnh không biến chỉ dẫn mềm thành đảm bảo cứng.

```text
Yêu cầu: "Mỗi lần sửa file TypeScript thì luôn chạy lint."
❌ SAI: thêm dòng "luôn chạy eslint sau khi sửa .ts" vào CLAUDE.md -> không đảm bảo.
✅ ĐÚNG: hook PostToolUse matcher "Edit|Write" chạy eslint -> harness luôn chạy.
```

</details>

---

### 7.3. Hãy cho vài ví dụ thực tế về việc dùng hook. Mỗi event hợp với loại tác vụ nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Format/lint tự động (prettier, eslint) → `PostToolUse` (matcher `Edit|Write`): chạy ngay sau khi file vừa sửa.
- Chặn lệnh nguy hiểm (`rm -rf`, sửa `.env`) → `PreToolUse`: chạy *trước* nên có thể từ chối.
- Gửi thông báo khi xong việc (desktop, Slack) → `Stop`.
- Nạp ngữ cảnh đầu phiên (branch git, trạng thái) → `SessionStart`.
- Chèn nhắc nhở/kiểm tra mỗi prompt → `UserPromptSubmit`.
- Bẫy: cho `PostToolUse` chạy lệnh quá nặng (build cả dự án, toàn bộ test) sau *mỗi* lần sửa → mỗi thao tác chậm. Giới hạn phạm vi (chỉ lint file vừa đổi), để kiểm tra nặng cho event ít xảy ra (như `Stop`).

```bash
# PreToolUse guard — chặn rm -rf; exit khác 0 -> harness CHẶN tool
input=$(cat)
if echo "$input" | grep -Eq 'rm[[:space:]]+-rf?[[:space:]]+/'; then
  echo "Bị chặn: lệnh xóa đệ quy nguy hiểm." >&2; exit 2
fi
exit 0
```

</details>

---

### 7.4. Hook được cấu hình ở đâu? Cấu trúc cấu hình ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

- Khai báo trong `settings.json` (hoặc `settings.local.json` cho cấu hình riêng máy, không commit).
- Cấu trúc: khóa `hooks` → từng tên event → mảng "matcher group" → danh sách lệnh sẽ chạy.
- `matcher` lọc theo loại tool (chỉ áp dụng cho event gắn tool như `PreToolUse`/`PostToolUse`); event không gắn tool (`Stop`, `SessionStart`) không cần matcher.
- Phân cấp như CLAUDE.md: enterprise / user (`~/.claude/settings.json`) / project (`.claude/settings.json`), áp dụng phối hợp. `settings.json` còn giữ `permissions`, `env`, `model`.
- Bẫy: đặt đường dẫn tuyệt đối riêng máy (vd `/Users/an/...`) vào `settings.json` project rồi commit → đồng đội lỗi. Cấu hình riêng máy đặt ở `settings.local.json`.

```jsonc
{
  "hooks": {
    "<TênEvent>": [
      { "matcher": "<mẫu khớp tool>",   // vd "Edit|Write"; bỏ nếu event không gắn tool
        "hooks": [{ "type": "command", "command": "<lệnh shell>" }] }
    ]
  }
}
```

</details>

---

### 7.5. Phân biệt Hook với Skill và với rule trong CLAUDE.md. Khi nào dùng cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Hook**: harness thực thi, tất định, tự động khi có event → "tự động mỗi khi X" (format, chặn lệnh, thông báo).
- **Skill**: model thực hiện quy trình, không tất định, gọi qua `/command` → quy trình chuyên biệt tái dùng (code-review, security-review).
- **Rule CLAUDE.md**: model đọc rồi tự diễn giải, không tất định, tự nạp vào context → quy ước/ngữ cảnh/kiến trúc dự án.
- Hỏi nhanh để chọn: "PHẢI luôn xảy ra tự động và đảm bảo?" → Hook; "Quy trình muốn gọi khi cần?" → Skill; "Kiến thức nền model nên biết?" → Rule.
- Cốt lõi: Skill/rule dựa vào model *quyết định* nên linh hoạt nhưng không đảm bảo; Hook bỏ qua model cho phần thực thi nên kém linh hoạt nhưng đảm bảo tuyệt đối.
- Bẫy: dùng Skill/rule cho việc đáng ra là hook (vd skill "luôn format") rồi ngạc nhiên khi đôi lúc nó không chạy.

</details>

---

### 7.6. Khi dùng hook, cần lưu ý gì về an toàn (safety)?

<details>
<summary>👉 Xem câu trả lời</summary>

- Hook chạy **với quyền của bạn, tự động**, không qua bước hỏi phép permission — hook viết ẩu (vd `rm` sai chỗ) sẽ thực thi mà không hỏi. Viết/kiểm thử như script production.
- **Cẩn trọng input không tin cậy**: harness truyền dữ liệu (đường dẫn file, nội dung tool call) qua stdin JSON; nội suy thẳng vào shell có nguy cơ command injection. Luôn trích dẫn biến (`"$file"`), tránh `eval`.
- **Dùng hook đúng cho bảo mật**: `PreToolUse` chạy trước và có thể *chặn* tool → lý tưởng dựng rào chắn (chặn `rm -rf /`, sửa `.env`/secrets, `git push --force`).
- **Đừng commit secret** trong `command`; dùng biến môi trường / secret manager.
- **Phạm vi tin cậy**: `settings.json` project nằm trong repo — mở repo lạ thì hook có thể chạy lệnh tùy ý. Cảnh giác hook từ nguồn không tin cậy.
- Bẫy: hook `PostToolUse` "mạnh tay" tự `git add -A && git commit` → mất kiểm soát lịch sử git. Hook nên làm việc an toàn, đảo ngược được (format, lint, thông báo); hành động khó đảo ngược (commit, push, xóa) nên dùng `PreToolUse` để *chặn*, không tự *thực hiện*.

```bash
# AN TOÀN: trích dẫn biến, không eval, fail mềm
prettier --write "$file" 2>/dev/null || true
# NGUY HIỂM: nội suy thô + eval -> command injection
eval prettier --write $file
```

</details>

---

### 7.7. Hãy kể về một lần bạn dùng hook (hoặc tự động hóa) để giải quyết một vấn đề lặp đi lặp lại trong quy trình làm việc.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Câu hỏi hành vi — trả lời theo STAR; thay bằng trải nghiệm thật của bạn.

- **Situation:** Dự án Next.js + TypeScript, team áp chuẩn Prettier/ESLint chặt. Code Claude sinh ra đúng logic nhưng đôi khi lệch style → CI fail ở bước lint, phải sửa thủ công lặp đi lặp lại.
- **Task:** Đảm bảo MỌI file Claude sửa đều tự động format + lint-fix ngay tại chỗ, chắc chắn, không phải nhắc AI từng prompt.
- **Action:** Nhận ra đây là dạng "tự động mỗi khi X" nên dùng hook chứ không ghi CLAUDE.md. Cấu hình hook `PostToolUse` (matcher `Edit|Write`) trong `.claude/settings.json` project, gọi script đọc đường dẫn file từ JSON stdin rồi chạy `prettier --write` + `eslint --fix` đúng file đó; fail mềm (`|| true`). Cấu hình ở cấp project + commit nên cả team hưởng lợi; hook `Stop` báo xong việc đặt ở `settings.local.json` riêng.
- **Result:** CI fail vì format/lint từ code AI giảm gần về 0, không còn nhắc "nhớ format" hay sửa style thủ công. Rút ra nguyên tắc cho team: cần tất định → hook; ngữ cảnh/quy ước → CLAUDE.md; quy trình gọi-khi-cần → Skill.

Lưu ý: nhấn mạnh *lý do chọn hook thay vì nhờ AI nhớ* (tính tất định) là điểm cộng lớn — cho thấy bạn hiểu đúng bản chất công cụ.

</details>

## 8. MCP Model Context Protocol

### 8.1. MCP (Model Context Protocol) là gì và vì sao nó quan trọng trong việc dùng AI để phát triển phần mềm?

<details>
<summary>👉 Xem câu trả lời</summary>

- MCP là chuẩn mở do Anthropic giới thiệu để kết nối ứng dụng AI với tool/dữ liệu bên ngoài một cách thống nhất — ví như "cổng USB-C cho AI".
- Giải quyết bài toán tích hợp M×N: thay vì mỗi AI viết tích hợp riêng cho từng dịch vụ, mỗi bên chỉ nói chuẩn MCP một lần (M+N) rồi cắm vào nhau.
- Tái sử dụng: một MCP server (vd cho database) dùng được trong Claude Code, app desktop, hay client khác.
- Mở rộng năng lực agent: biến model từ "chatbot trả lời" thành "agent hành động" trên hệ thống thật (đọc issue Linear, query DB, lấy design Figma).
- Bẫy: MCP không phải "tính năng của Claude" — nó là chuẩn mở, độc lập với model; Claude Code chỉ là một trong nhiều client.

```text
KHÔNG MCP: mỗi AI app tự viết tích hợp -> M × N bộ code
CÓ MCP:   chuẩn chung -> M + N (nhiều client ── chuẩn MCP ── nhiều server)
```

</details>

---

### 8.2. Phân biệt MCP server và MCP client. Trong bối cảnh Claude Code, đâu là client, đâu là server?

<details>
<summary>👉 Xem câu trả lời</summary>

- **MCP server:** bên *cung cấp* năng lực — expose tool/resource/prompt; bọc quanh một hệ thống cụ thể (GitHub, DB, Figma), biết cách gọi API/chạy SQL thật và trả kết quả theo chuẩn MCP.
- **MCP client:** bên *sử dụng* — ứng dụng AI nhúng model, kết nối tới nhiều server, gom tool và để model quyết định gọi tool nào khi nào.
- Trong Claude Code: **Claude Code là client**; server là các tiến trình/dịch vụ riêng, chạy local (qua stdio) hoặc remote (qua HTTP/SSE).
- Một client nối được nhiều server, một server phục vụ được nhiều client. Model không trực tiếp "biết" GitHub — chỉ thấy tool server khai báo, client làm cầu nối.

```text
Claude Code (CLIENT) ──┬── MCP server GitHub  (gọi GitHub API)
                       ├── MCP server Postgres (chạy SQL)
                       └── MCP server Figma   (đọc design)
```

</details>

---

### 8.3. Trong MCP có ba loại "khả năng" mà server có thể expose: tools, resources và prompts. Giải thích sự khác nhau và khi nào dùng cái nào.

<details>
<summary>👉 Xem câu trả lời</summary>

- **Tools:** hành động model gọi để *làm việc gì đó* (thường có tác dụng phụ), model-controlled — vd `create_issue`, `run_query`. Dùng khi cần agent thực hiện hành động.
- **Resources:** dữ liệu chỉ-đọc để nạp vào context (như GET), app-controlled — vd nội dung file, bản ghi DB. Dùng khi cần cung cấp ngữ cảnh không gây tác dụng phụ.
- **Prompts:** mẫu prompt/quy trình dựng sẵn, user-controlled (gắn slash command) — vd "tóm tắt PR này". Dùng khi gói quy trình tái sử dụng cho người dùng chọn.
- Lưu ý: nhiều client (gồm Claude Code) tận dụng **tools** nhiều nhất vì khớp tự nhiên với tool use; mức hỗ trợ resource/prompt khác nhau giữa client — đừng giả định client nào cũng dùng đủ ba loại.

```text
Tools     -> "Hãy LÀM việc này"      (model gọi)
Resources -> "Đây là dữ liệu để ĐỌC" (app nạp)
Prompts   -> "Đây là quy trình sẵn"  (user kích hoạt)
```

</details>

---

### 8.4. Hãy kể vài MCP server phổ biến và mô tả cách thêm một MCP server vào Claude Code.

<details>
<summary>👉 Xem câu trả lời</summary>

- Server phổ biến: **GitHub** (issue/PR/repo), **Postgres** (schema, query), **Linear/Atlassian Jira** (task, ticket), **Figma** (design, component), **Notion** (tài liệu, trang).
- Thêm bằng **CLI** `claude mcp add`: server local chỉ ra lệnh khởi chạy (stdio), server remote chỉ ra URL (HTTP/SSE).
- Thêm bằng **file cấu hình**: khai báo lệnh chạy, tham số, biến môi trường (token).
- **Scope:** cấp user (mọi dự án) hoặc cấp project (chia sẻ với team qua repo).
- Server kết nối xong, tool của nó xuất hiện trong hộp công cụ agent (thường có tiền tố tên server); vẫn tuân permission mode (mặc định hỏi trước).
- Bẫy: quên cấp token/biến môi trường → server nối được nhưng mọi lời gọi tool lỗi 401/403. Kiểm tra trạng thái bằng `/mcp` hoặc `claude mcp list` trước khi nghĩ "AI không chịu làm".

```bash
claude mcp add ten-server -- <lenh-chay-server> [tham-so...]
claude mcp list   # kiểm tra danh sách; trong phiên dùng /mcp
```

</details>

---

### 8.5. Khi cấp cho agent quyền truy cập hệ thống thật qua MCP, bạn lo ngại điều gì về bảo mật và xác thực? Làm sao giảm thiểu rủi ro?

<details>
<summary>👉 Xem câu trả lời</summary>

Mối lo chính:

- **Xác thực:** lộ credential (token, OAuth, connection string) do commit nhầm/log ra → kẻ xấu có toàn quyền như bạn.
- **Phân quyền quá rộng:** cấp token ghi/xóa trong khi chỉ cần đọc → một lệnh sai (hoặc prompt injection) có thể xóa dữ liệu thật.
- **Prompt injection qua dữ liệu:** nội dung agent đọc (issue, web, bản ghi DB) chứa chỉ thị độc hại khiến model gọi tool ngoài ý muốn.
- **Server không tin cậy:** cài server lạ = chạy phần mềm bên thứ ba với quyền chạm dữ liệu.

Cách giảm thiểu:

- **Least privilege:** token chỉ-đọc khi đủ, giới hạn scope OAuth, DB user quyền hạn chế.
- **Quản lý secret:** để token trong env/secret store, `.gitignore`, ưu tiên OAuth thu hồi được thay vì token tĩnh.
- **Permission mode:** giữ hỏi-trước cho tool có tác dụng phụ; tránh `bypassPermissions` với MCP đụng production; dùng **hook PreToolUse** để chặn tự động (xác định, không phụ thuộc model nhớ).
- Chỉ dùng server tin cậy, cô lập bằng staging/sandbox khi thử.

Điểm nhấn: hành vi "luôn chặn X" phải dùng **hook** (harness tự chạy, xác định) chứ không trông cậy model "tự nhớ" — model có thể bị injection hoặc quên.

</details>

---

### 8.6. Phân biệt MCP với Skill và Plugin trong Claude Code. Khi nào nên dùng cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- **MCP:** chuẩn mở kết nối AI với **tool/dữ liệu bên ngoài** qua server; cho agent *năng lực mới* chạm hệ thống thật (GitHub, DB, Figma). Định nghĩa ở MCP server riêng, khai báo trong cấu hình.
- **Skill:** "kỹ năng" tái sử dụng đóng gói **quy trình/kiến thức** (cách làm một việc lặp lại như code-review, deep-research). File `SKILL.md` có frontmatter, gọi qua slash command.
- **Plugin:** gói phân phối **đóng gói nhiều thứ** (skill + command + agent + cả cấu hình MCP); để *phân phối & cài đặt* một bộ năng lực cho team.

Cách chọn:

- Cần **truy cập hệ thống bên ngoài** (đọc Jira, query DB) → **MCP server**.
- Cần **làm theo quy trình chuyên biệt** dùng lại nhiều lần → **Skill**.
- Cần **đóng gói & chia sẻ** một bộ năng lực cho nhóm → **Plugin**.

Chúng bổ trợ nhau, không loại trừ (vd plugin "Jira workflow" kèm cả MCP server Atlassian lẫn skill phân rã epic). Bẫy: nhét logic kết nối hệ thống vào skill bằng cách bảo Claude "chạy curl" — kém ổn định; tích hợp hệ thống thì dùng MCP server đúng nghĩa.

</details>

---

### 8.7. Kể về một lần bạn dùng (hoặc sẽ thiết kế việc dùng) MCP để biến AI thành "siêu năng lực" trong quy trình làm việc của team.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Câu này kiểm tra khả năng nhìn AI như một phần hệ thống kỹ thuật, tư duy tự động hóa quy trình, và ý thức bảo mật khi cho agent chạm hệ thống thật.

- **Situation:** "Team tốn nhiều thời gian thủ công khi nhận ticket: mở Jira đọc mô tả, đối chiếu code trong repo, viết tay mô tả PR — bối cảnh rải rác nhiều công cụ."
- **Task:** "Rút ngắn vòng từ ticket đến PR draft, để developer tập trung vào suy nghĩ thực sự thay vì sao chép thông tin giữa các tab."
- **Action:** "Cấu hình Claude Code (MCP client) kết nối MCP server Atlassian/Jira và GitHub: agent đọc ticket trực tiếp, đối chiếu code, soạn mô tả PR dẫn chiếu đúng issue. Bảo mật: token least-privilege, credential trong env không commit, permission mode hỏi-trước cho tool ghi (tạo/sửa PR), thêm hook PreToolUse chặn ghi ngoài ý muốn. Phân biệt rõ: *kết nối hệ thống* dùng MCP, *quy trình review* đóng gói thành skill cho team dùng lại."
- **Result:** "Thời gian 'nhận ticket → có PR draft kèm ngữ cảnh' giảm đáng kể, mô tả PR đồng nhất hơn. Không lần nào agent ghi mà thiếu duyệt của con người, nhờ permission mode và hook."

**Điểm nhấn:** hiểu vai client–server (Claude Code = client, Jira/GitHub = server); ghép đúng công cụ (MCP cho kết nối, skill cho quy trình); kỷ luật bảo mật (least privilege, secret trong env, permission mode, hook) — trao quyền cho AI *có kiểm soát*.

</details>

## 9. Workflows và Điều phối Multi-Agent

### 9.1. Khi nào nên chuyển từ một agent đơn lẻ sang điều phối multi-agent (workflow)? Dấu hiệu nào cho thấy bài toán cần fan-out hoặc cô lập context?

<details>
<summary>👉 Xem câu trả lời</summary>

- Mặc định giữ **một agent đơn lẻ**: đơn giản, dễ debug, ít token. Chỉ lên multi-agent khi có dấu hiệu rõ ràng.
- **Fan-out song song**: nhiều việc độc lập, làm tuần tự sẽ chậm (ví dụ: tìm kiếm 5 chủ đề cùng lúc).
- **Cô lập context**: việc con ngốn nhiều token (đọc hàng chục file/log) mà không cần trả hết về context chính.
- **Việc lớn nhiều giai đoạn**: chia bước tách bạch (research → design → implement → verify).
- **Kiểm chứng chéo**: một agent tạo, agent khác (context độc lập) kiểm → giảm thiên kiến tự xác nhận.

| Tình huống | Một agent | Multi-agent |
|---|---|---|
| Việc nhỏ, tuần tự, gọn | ✅ | ❌ thừa |
| Nhiều việc độc lập | ❌ chậm | ✅ fan-out |
| Việc con đọc rất nhiều file | phình to | ✅ cô lập context |

Bẫy: vội dựng multi-agent cho việc đơn giản — mỗi subagent thêm context riêng + chi phí điều phối. Nguyên tắc: **bắt đầu đơn giản nhất, chỉ nâng cấp khi thật sự cần.**

</details>

---

### 9.2. Có những mẫu (pattern) điều phối multi-agent phổ biến nào? Mô tả ngắn gọn từng mẫu và khi nào dùng.

<details>
<summary>👉 Xem câu trả lời</summary>

- **Fan-out song song**: 1 orchestrator chia việc → N subagent chạy đồng thời → gom. Dùng khi việc con độc lập.
- **Pipeline (tuần tự)**: kết quả agent này là đầu vào agent kế (research → draft → review). Dùng khi có thứ tự phụ thuộc.
- **Adversarial verify**: một agent tạo, agent khác (context độc lập) tìm lỗi/phản biện. Dùng khi cần chính xác cao, giảm tự xác nhận.
- **Judge panel**: nhiều agent cùng chấm theo tiêu chí rồi tổng hợp (bỏ phiếu/đồng thuận). Dùng khi đánh giá chủ quan, cần nhiều góc nhìn.

| Mẫu | Cấu trúc | Khi nào dùng |
|---|---|---|
| Fan-out | 1 → N đồng thời → gom | Việc con độc lập, muốn nhanh |
| Pipeline | A → B → C | Có thứ tự phụ thuộc |
| Adversarial verify | Tạo ↔ phản biện | Cần chính xác |
| Judge panel | N chấm → tổng hợp | Đánh giá chủ quan |

Lưu ý: các mẫu kết hợp được. Trong Claude Code: fan-out/cô lập context dùng **subagents**, điều phối phức tạp/tất định dùng **workflow** (script).

</details>

---

### 9.3. Chi phí token là yếu tố cần cân nhắc thế nào khi thiết kế workflow multi-agent? Vì sao multi-agent có thể đắt hơn nhiều so với một agent?

<details>
<summary>👉 Xem câu trả lời</summary>

- Multi-agent đánh đổi **token lấy tốc độ + chất lượng**. Nguồn phát sinh token:
- **Mỗi subagent có context riêng**: nạp lại system prompt + ngữ cảnh → N agent = N lần "khởi động".
- **Lặp đầu vào**: fan-out/judge panel gửi cùng tài liệu tới nhiều agent → nhân token đầu vào.
- **Vòng lặp agentic dài** trong mỗi subagent + **chi phí tổng hợp** ở orchestrator.

| Đòn bẩy giảm chi phí | Tác dụng |
|---|---|
| Prompt caching | Đầu vào chung được cache, lần sau chỉ ~0.1× giá |
| Cô lập context | Dữ liệu trung gian ở lại subagent, chỉ tóm tắt về |
| Model rẻ cho việc dễ | Haiku cho subagent đơn giản |
| `effort` thấp | Hạ "suy nghĩ" cho việc dễ |

Ví dụ: judge panel 5 agent đọc báo cáo 50K token → ~250K token nếu không cache; cache prefix chung thì 4 agent sau chỉ ~0.1×. Mẹo fan-out: gửi 1 request trước để ghi cache, rồi mới bắn N−1 request còn lại đọc từ cache.

Điểm phỏng vấn: multi-agent **không miễn phí** — luôn tự hỏi giá trị có xứng chi phí token không.

</details>

---

### 9.4. Khi nào KHÔNG nên dùng workflow multi-agent? Những trường hợp một agent đơn lẻ vẫn là lựa chọn tốt hơn.

<details>
<summary>👉 Xem câu trả lời</summary>

- **Việc nhỏ, một lần, tuần tự** (sửa 1 bug ở 1 file): chia agent chỉ thêm chi phí + độ trễ.
- **Bước phụ thuộc chặt, không song song được**: B cần đủ kết quả A thì "song song hóa" vô ích.
- **Chi phí token vượt giá trị** / **context không quá lớn**: không cần cô lập.
- **Cần dễ debug/bảo trì**: nhiều agent = nhiều điểm sai, khó truy vết.

| Câu hỏi | Nếu "Không" → |
|---|---|
| Việc có nhiều bước, khó đặc tả trước? | Giữ một agent |
| Giá trị xứng chi phí token & độ trễ? | Giữ một agent |
| Có việc con song song được? | Không cần fan-out |

Ví dụ over-engineering: đổi tên biến trong 3 file mà dựng "agent tìm + sửa + verify" — một agent với vòng lặp grep → edit → test là quá đủ. Nguyên tắc: **luôn bắt đầu ở tầng đơn giản nhất đáp ứng yêu cầu.**

</details>

---

### 9.5. Cho một ví dụ workflow cụ thể cho "review nhiều chiều" và một cho "nghiên cứu (deep research)". Mô tả cách điều phối các agent.

<details>
<summary>👉 Xem câu trả lời</summary>

**Review nhiều chiều (fan-out + judge):** fan-out diff/PR thành nhiều agent chuyên biệt, mỗi agent một góc (correctness, bảo mật, hiệu năng, style) → context riêng → recall cao. Orchestrator gom findings, loại trùng, xếp hạng theo mức nghiêm trọng. Có thể thêm bước **verify** lọc báo động giả.

**Deep research (fan-out → verify → tổng hợp):**

```text
1. Phân rã câu hỏi thành nhánh tìm kiếm con
2. Fan-out: nhiều subagent tìm kiếm song song
3. Mỗi subagent đọc nguồn, trích dẫn, trả phát hiện (context cô lập)
4. Adversarial verify: kiểm chứng tuyên bố so với nguồn (chống bịa)
5. Tổng hợp: orchestrator viết báo cáo có trích dẫn
```

- Fan-out → nhanh + bao phủ rộng; cô lập context → subagent xử lý nguồn dài không phình context chính; verify → tách "tạo" khỏi "kiểm" để giảm thiên kiến.
- Liên hệ Claude Code: deep research ↔ skill `deep-research`; review nhiều chiều ↔ `/code-review`, `security-review`. Agent con thường là **subagents** (vai trò Explore).

</details>

---

### 9.6. Vì sao điều phối tất định (deterministic, bằng script) lại hữu ích cho việc lớn? Khác gì với việc để một agent tự quyết mọi thứ?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Workflow** = điều phối nhiều agent **tất định**, luồng lập trình sẵn bằng script thay vì để model tự quyết trình tự.
- **Tái lập được**: cùng đầu vào → cùng luồng; quan trọng cho production/CI/chạy định kỳ.
- **Kiểm soát chi phí & độ trễ**: biết trước gọi bao nhiêu agent, ước lượng token + thời gian.
- **Dễ debug & audit**: lỗi ở bước nào thấy ngay vì luồng cố định.
- **Đặt guardrails đúng chỗ**: chèn kiểm tra/phê duyệt/gating tại điểm xác định.

| Tiêu chí | Agent tự chủ | Workflow tất định |
|---|---|---|
| Quyết định luồng | Model chọn mỗi vòng | Dev định nghĩa cố định |
| Tái lập | Mỗi lần có thể khác | Dễ tái lập |
| Phù hợp | Việc mở, khó đặc tả | Việc lớn, lặp lại, quy trình rõ |

Ví dụ: "build tài liệu hằng đêm" (thu thập → fan-out tóm tắt → tổng hợp → verify → xuất) đóng gói thành workflow script thì mỗi đêm chạy giống hệt, dễ giám sát.

Điểm phỏng vấn: tất định và tự chủ **không loại trừ nhau** — workflow đóng khung luồng tổng thể, model vẫn làm phần "thông minh" trong mỗi bước.

</details>

## 10. Claude API và Xây dựng tính năng AI

### 10.1. Các model Claude (Opus / Sonnet / Haiku / Fable) khác nhau ra sao, và bạn chọn model theo tiêu chí gì?

<details>
<summary>👉 Xem câu trả lời</summary>

- Họ model phân tầng theo cân bằng **trí thông minh — tốc độ — chi phí**; truyền `model id` chính xác (string), không tự bịa hay thêm hậu tố ngày.
- Tiêu chí chọn: **độ khó tác vụ** (Haiku cho phân loại/trích xuất ngắn; Opus 4.8/Fable 5 cho reasoning, coding, agentic), **chi phí/volume** (Sonnet/Haiku cho production lớn — nhưng hạ cấp là quyết định business, không tự ý), **context window**, **latency** (interactive ưu tiên model nhanh).
- Pattern thực tế: **model routing** — dùng nhiều model trong một hệ thống (Haiku cho sub-task rẻ, Opus cho reasoning chính).
- Bẫy: dùng `model id` sai (vd `claude-sonnet-4.6` thay vì `claude-sonnet-4-6`) → lỗi `404 not_found_error`.

| Model | Model ID | Context | In/Out $/1M | Dùng khi |
|---|---|---|---|---|
| Fable 5 | `claude-fable-5` | 1M | $10/$50 | Mạnh nhất, reasoning khó, agentic dài |
| Opus 4.8 | `claude-opus-4-8` | 1M | $5/$25 | Mặc định cho việc cần "thông minh" |
| Sonnet 4.6 | `claude-sonnet-4-6` | 1M | $3/$15 | Cân bằng cho production volume cao |
| Haiku 4.5 | `claude-haiku-4-5` | 200K | $1/$5 | Tác vụ đơn giản, nhạy tốc độ |

</details>

---

### 10.2. Tool use (function calling) là gì? Mô tả vòng đời một lượt gọi tool.

<details>
<summary>👉 Xem câu trả lời</summary>

- **Tool use** là cơ chế model **yêu cầu** bạn chạy hàm rồi trả kết quả về (model không tự thực thi); nền tảng của mọi agent. Tool định nghĩa bằng `name`, `description`, `input_schema`.
- Vòng đời: (1) gửi `tools` + câu hỏi → (2) model trả `stop_reason: "tool_use"` với block chứa `name`/`input`/`id` → (3) **bạn** chạy hàm thật → (4) gửi lại `tool_result` kèm đúng `tool_use_id` trong message `role: "user"` → (5) lặp tới `end_turn`.
- `tool_choice`: `auto` (mặc định), `any` (buộc dùng ≥1 tool), `{type:"tool", name}` (buộc đúng tool), `none` (cấm).
- Bẫy: thiếu `tool_result` cho một `tool_use_id` → lỗi 400; phải `JSON.parse()` input (đừng raw-string-match); `description` sờ sài → model gọi sai (mô tả rõ *khi nào* gọi).
- SDK có **tool runner** (beta) tự lo vòng lặp; dùng vòng lặp thủ công khi cần logging/human-in-the-loop.

```python
while True:
    resp = client.messages.create(model="claude-opus-4-8", max_tokens=16000,
                                   tools=tools, messages=messages)
    if resp.stop_reason == "end_turn": break
    messages.append({"role": "assistant", "content": resp.content})
    results = [{"type": "tool_result", "tool_use_id": b.id,
                "content": run_tool(b.name, b.input)}
               for b in resp.content if b.type == "tool_use"]
    messages.append({"role": "user", "content": results})
```

</details>

---

### 10.3. Prompt caching là gì? Nó giúp giảm chi phí và độ trễ như thế nào, và cái gì dễ làm hỏng cache?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Prompt caching** cache phần prefix ổn định của prompt → tiết kiệm tới ~90% chi phí phần được cache và giảm độ trễ.
- Nguyên tắc cốt lõi: **prefix match** theo đúng từng byte; một byte đổi ở vị trí N phá cache mọi thứ từ N trở đi. Thứ tự render: `tools` → `system` → `messages`. Đặt `cache_control` ở cuối phần ổn định.
- Kinh tế: đọc cache ~0.1x giá gốc, ghi cache ~1.25x (TTL 5 phút) / 2x (TTL 1h); chỉ cần 2 request là hòa vốn.
- **Silent invalidators** (phá cache âm thầm): `datetime.now()`/UUID trong system prompt; `json.dumps()` không `sort_keys=True`; nhét session/user id vào system; đổi danh sách tool hoặc đổi model giữa chừng.
- Kiểm chứng qua `usage`: `cache_read_input_tokens` luôn = 0 dù prefix giống hệt → chắc chắn có silent invalidator (diff byte để tìm).

```python
system=[{"type": "text", "text": tai_lieu_lon,
         "cache_control": {"type": "ephemeral"}}]   # TTL 5 phút; "1h" cho 1 giờ
```

</details>

---

### 10.4. Khi nào dùng RAG, khi nào dùng long context, khi nào fine-tune?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Long context**: nhồi thẳng tài liệu vào prompt; hợp khi dữ liệu vừa phải, ổn định, dùng lại nhiều (kết hợp prompt caching). Đơn giản nhất, không cần hạ tầng.
- **RAG**: dùng **embeddings** tìm các đoạn liên quan rồi mới nhồi vào prompt; hợp khi kho dữ liệu lớn (hàng nghìn tài liệu) hoặc đổi liên tục. Tiết kiệm token và cập nhật được dữ liệu mới mà không train lại.
- **Fine-tune**: huấn luyện lại trọng số; **không** để nhồi kiến thức mà để dạy *phong cách/định dạng/hành vi* đặc thù prompt không đủ.
- Quy tắc thực dụng: **bắt đầu long context + caching** → vượt context/đổi liên tục thì **RAG** → chỉ fine-tune khi hai cách kia vẫn thiếu về hành vi. Có thể kết hợp RAG + long context.
- Bẫy: dùng fine-tune để "nhét kiến thức công ty" — tốn kém, khó cập nhật; RAG mới là câu trả lời cho "kiến thức thay đổi".

</details>

---

### 10.5. Xây một agent bằng Claude API như thế nào? Vòng lặp tool hoạt động ra sao và khi nào *nên* xây agent?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Agent** = vòng lặp tool use mà *model tự quyết quỹ đạo*: gọi tool → đọc kết quả → suy nghĩ → gọi tiếp, tới khi xong. Khác **workflow** (bạn lập trình tất định các bước).
- API **stateless** — mỗi lần gọi phải gửi lại toàn bộ `messages`; luôn append nguyên `resp.content` (giữ cả block `tool_use`, `thinking`). Xử lý `pause_turn` (server tool đạt giới hạn) bằng cách gửi lại để chạy tiếp.
- Trước khi xây agent, kiểm 4 tiêu chí: **độ phức tạp**, **giá trị** (đáng chi phí/độ trễ không), **khả thi** (model giỏi loại tác vụ này không), **chi phí của lỗi** (bắt/phục hồi được không). "Không" với bất kỳ → dùng tầng đơn giản hơn.
- Nguyên tắc vàng: **bắt đầu đơn giản**, chỉ leo lên agent khi tác vụ thật sự cần khám phá mở.
- Agent dài hơi: bật **adaptive thinking** (`thinking: {type:"adaptive"}`) + tham số `effort` (`low`→`max`; `high`/`xhigh` cho coding, `low` cho sub-task đơn giản).

</details>

---

### 10.6. Streaming là gì và tại sao gần như luôn nên dùng cho output dài?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Streaming** trả phản hồi theo từng chunk qua **Server-Sent Events (SSE)**. Lợi ích kép: (1) hiển thị chữ ngay (UX chat), (2) **tránh timeout HTTP** với output lớn (lý do kỹ thuật chính).
- SDK **báo `ValueError`** nếu gọi non-streaming với `max_tokens` ước tính vượt ~10 phút; Opus 4.8 hỗ trợ tới 128K output token nhưng **bắt buộc streaming**. Quy tắc `max_tokens`: ~16000 non-streaming, ~64000 streaming.
- Mẹo: ngay cả không cần real-time vẫn nên stream rồi gọi `.get_final_message()` để vừa có message hoàn chỉnh vừa tránh timeout. Đừng tự bọc `.on()` trong `new Promise()` — dùng helper SDK.
- Lưu ý thinking: trên Opus 4.8/4.7/Fable 5 mặc định `display: "omitted"` (thấy "đứng im" trước khi text ra); muốn hiện tiến trình đặt `thinking: {type:"adaptive", display:"summarized"}`.
- Event chính: `message_start`, `content_block_start/delta/stop`, `message_delta` (chứa `stop_reason`/`usage`), `message_stop`.

```python
with client.messages.stream(model="claude-opus-4-8", max_tokens=64000,
        messages=[{"role": "user", "content": "Viết một câu chuyện"}]) as stream:
    for text in stream.text_stream: print(text, end="", flush=True)
    final = stream.get_final_message()
```

</details>

---

### 10.7. Structured output là gì? Làm sao buộc model trả về JSON đúng schema?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Structured output** ràng buộc phản hồi theo JSON Schema, đảm bảo output luôn parse được; hữu ích cho pipeline cần dữ liệu có cấu trúc.
- Hai cơ chế: **JSON output** qua `output_config.format` (ràng buộc *định dạng phản hồi*) và **strict tool use** (`strict: true`, ràng buộc *tham số tool*). Khuyên dùng `messages.parse()` với Pydantic/Zod để tự validate.
- Lưu ý: `output_config.format` là tham số chuẩn (cũ `output_format` ở `messages.create()` đã deprecated); lần đầu một schema có **độ trễ biên dịch** (sau đó cache 24h); **không** hỗ trợ schema đệ quy, `minimum`/`maximum`, `minLength`/`maxLength`, và `additionalProperties` phải `false`; không tương thích **citations** và **prefill** (lỗi 400).
- Phải kiểm `stop_reason` trước: nếu `"refusal"` hoặc `"max_tokens"`, output có thể không khớp schema.

```python
class ContactInfo(BaseModel):
    name: str; email: str; plan: str; demo_requested: bool

resp = client.messages.parse(model="claude-opus-4-8", max_tokens=16000,
    messages=[{"role": "user", "content": "Trích: Jane (jane@co.com)..."}],
    output_format=ContactInfo)
contact = resp.parsed_output      # instance đã validate
```

</details>

---

### 10.8. Cửa sổ ngữ cảnh (context window) và đếm token: tại sao quan trọng và đếm thế nào cho đúng?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Context window** = tổng token tối đa cho một request (input: system + tools + lịch sử messages, lẫn output). Opus/Sonnet/Fable: **1M**, Haiku 4.5: 200K. Vượt → `stop_reason: "model_context_window_exceeded"` (khác `"max_tokens"` là chạm trần output).
- Quan trọng vì: **chi phí** theo token (output đắt hơn input nhiều); **hội thoại/agent dài** dễ phình và vượt cửa sổ; cần ước lượng để đặt `max_tokens` và dự toán.
- Đếm đúng: dùng endpoint **`count_tokens`** (`POST /v1/messages/count_tokens`), **không** dùng `tiktoken` (tokenizer OpenAI — thiếu ~15–20%, sai nhiều hơn với code/tiếng không phải Anh). Token đặc thù theo model → truyền đúng `model`.
- Hội thoại quá dài: **compaction** (beta, API tự tóm tắt — phải append cả block `compaction`), **context editing** (dọn tool result/thinking cũ), **prompt caching** (giảm chi phí, không giảm token).
- Mẹo: gọi `count_tokens` *trước* khi gửi; đừng âm thầm truncate input — báo người dùng và bàn chunking/tóm tắt/RAG.

</details>

---

### 10.9. Embeddings là gì và vai trò của nó trong RAG?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Embeddings** biểu diễn văn bản thành **vector số** sao cho đoạn *gần nghĩa* thì vector gần nhau → tìm kiếm theo *ngữ nghĩa* (semantic search) thay vì khớp từ khóa (bắt được "ô tô" vs "xe hơi").
- Vai trò trong **RAG**: là bước **Retrieval**. Pipeline: (1) Indexing offline — chunk tài liệu → embedding → vector DB; (2) Query — embedding câu hỏi; (3) Retrieval — tìm chunk gần nhất (cosine similarity); (4) Augment + Generate — nhồi chunk vào prompt → gọi model.
- Lưu ý: embedding và model sinh text là **hai bước riêng**; embeddings chỉ phục vụ *tìm kiếm*, không tự trả lời.
- Bẫy: tưởng RAG "thông minh" sẵn — phần lớn lỗi nằm ở **retrieval** (chunk quá to/nhỏ, embedding không hợp domain), không ở model sinh (rác vào, rác ra).

</details>

---

### 10.10. Đánh giá (eval) tính năng AI: làm sao đo chất lượng, và "LLM-as-judge" là gì?

<details>
<summary>👉 Xem câu trả lời</summary>

- Output AI không tất định → **không test bằng assert đúng/sai tuyệt đối**; cần bộ **eval** có hệ thống, nhất là khi nâng cấp model/đổi prompt.
- Các loại (rẻ → đắt): **exact/rule-based** (so khớp, regex, keyword — cho phân loại/trích xuất), **programmatic** (chạy code kiểm tra — cho code/JSON schema), **LLM-as-judge** (model khác chấm theo rubric — cho tác vụ mở), **human eval** (chuẩn vàng, đắt/chậm).
- **LLM-as-judge**: cho LLM output + **rubric** rõ ràng để chấm điểm; hợp tác vụ mở không có đáp án duy nhất.
- Nguyên tắc/bẫy: rubric **rõ ràng, chấm được độc lập** (tránh "trông ổn"); LLM-judge **không thay hoàn toàn** human (hiệu chỉnh bằng tập đã có người chấm); **tách bước chấm khỏi bước sinh** (context riêng); chạy trên **tập cố định** để so sánh giữa các phiên bản.

```python
prompt = f"""Bạn là giám khảo. Chấm theo rubric.
Câu hỏi: {cau_hoi}\nCâu trả lời: {cau_tra_loi}
Rubric: trả JSON {{"diem": 1-5, "ly_do": "..."}} — 5 là chính xác, đầy đủ."""
```

</details>

---

### 10.11. Guardrails và an toàn (safety): khi xây tính năng AI cho người dùng, bạn xử lý từ chối, prompt injection và rò rỉ bí mật ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Xử lý `refusal`**: model từ chối trả HTTP 200 với `stop_reason: "refusal"` + `stop_details`. Phải **kiểm `stop_reason` trước khi đọc `content`** (tránh lỗi index khi `content` rỗng). Fable 5 có **fallbacks** (opt-in): tự chạy lại trên model dự phòng — nên bật mặc định.
- **Chống prompt injection** (rủi ro lớn nhất khi đưa dữ liệu *không tin cậy* vào prompt): tách **kênh chỉ thị** (lệnh ở `system`/`role:"system"` beta) khỏi **kênh dữ liệu**; promote hành động nguy hiểm thành **tool có gate** để harness chặn/duyệt (thay vì `bash` tùy ý).
- **Không rò rỉ bí mật**: tuyệt đối không nhét API key/token/mật khẩu vào prompt (bị lưu trong lịch sử/compaction); giữ secret ở host, để orchestrator thực thi qua custom tool, sandbox không thấy key.
- **Human-in-the-loop**: thao tác có side-effect dùng vòng lặp tool thủ công để **duyệt trước khi chạy**; validate input trong hàm tool.
- **Lỗi & rate limit**: bắt typed exception (`RateLimitError`...), retry backoff (SDK tự retry 429/5xx), log `request_id`.

```python
if resp.stop_reason == "refusal":
    handle_refusal(resp.stop_details)   # đừng đọc resp.content[0] mù quáng
else:
    print(resp.content[0].text)
```

</details>

## 11. Best Practices và Siêu năng lực với AI

### 11.1. Quy trình review code do AI sinh ra của bạn gồm những bước nào? Vì sao không thể chỉ "chạy thử thấy được là merge"?

<details>
<summary>👉 Xem câu trả lời</summary>

- Review code AI y như review PR đồng nghiệp, thậm chí kỹ hơn: AI tự tin đều cả khi sai.
- Nguyên tắc: đọc kỹ diff từng dòng, chạy test/lint/app thật, và tự giải thích "vì sao code này đúng" trước khi merge.
- "Chạy được" chỉ chứng minh happy path, không phủ edge case, lỗ hổng bảo mật, hay nợ kỹ thuật.
- Nút thắt khi dùng AI là review chứ không phải viết code; trách nhiệm hiểu code vẫn của con người.
- Bẫy: tưởng tốc độ sinh code bằng tốc độ giải quyết vấn đề.

```text
Checklist: diff đủ ngắn để đọc hết? giải thích được TỪNG thay đổi?
edge case (null/rỗng/lỗi)? API bịa/deprecated? có test? khớp CLAUDE.md?
```

</details>

---

### 11.2. "Không tin tưởng mù quáng — luôn verify" nghĩa là gì trong thực tế? Bạn verify đầu ra của AI bằng cách nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Tin mù (automation bias): tin đầu ra chỉ vì nó trình bày tự tin; AI có thể **hallucinate** API/tham số/mẫu lỗi thời.
- Coi mọi đầu ra AI là *giả thuyết cần kiểm chứng*, verify theo loại: code → chạy test/lint/app; API → đối chiếu docs; khái niệm → so nguồn tin cậy.
- Cờ đỏ verify gấp đôi: AI nói "an toàn/chuẩn rồi" không kèm bằng chứng; code đụng bảo mật/tiền/dữ liệu; dùng hàm/flag lạ.
- Để **máy** lo kiểm chứng lặp lại: hook `PostToolUse` tự chạy test/lint, CI chặn merge khi fail.
- Bẫy nguy hiểm nhất: dùng AI cho việc mình KHÔNG đủ năng lực kiểm chứng → không phân biệt được đúng và "nghe có vẻ đúng".

</details>

---

### 11.3. Làm sao bạn dùng AI để đọc hiểu và làm quen một codebase lạ thật nhanh — mà không rơi vào bẫy hiểu sai?

<details>
<summary>👉 Xem câu trả lời</summary>

- Giao AI việc *khảo sát/định vị*, rồi tự đọc lại chỗ trọng yếu để *hiểu*; AI tăng tốc định vị, mắt mình mới hiểu được.
- Dùng **subagent Explore** chuyên tìm kiếm, chạy nhiều subagent song song cho các câu hỏi độc lập (context riêng, không loãng phiên chính).
- Quy trình: hỏi tổng quan kiến trúc → định vị logic cụ thể → truy vết một luồng end-to-end → đọc lại CHÍNH file AI chỉ ra.
- Tránh bẫy: luôn yêu cầu "trỏ tôi tới dòng/file cụ thể" và mở file gốc đọc lại; kiểm chứng bằng chạy thử; ghi hiểu biết vào CLAUDE.md.
- Bẫy: tin tóm tắt mà không bao giờ mở file gốc — tóm tắt chỉ là điểm khởi đầu, không thay việc đọc.

</details>

---

### 11.4. Làm sao để refactor an toàn với sự hỗ trợ của AI? Điều kiện tiên quyết là gì?

<details>
<summary>👉 Xem câu trả lời</summary>

- Điều kiện tiên quyết: **phải có test xanh bao phủ trước khi cho AI sửa rộng**. Refactor = đổi cấu trúc, KHÔNG đổi hành vi → chỉ test mới chứng minh được.
- Quy trình: bật plan mode duyệt kế hoạch → chia nhỏ từng bước diff đọc được → sau mỗi bước chạy lại test (hook tự chạy) → commit nhỏ để dễ rollback.
- An toàn giao AI: đổi tên xuyên file, tách hàm, gom code trùng. Thận trọng: đổi signature (cập nhật mọi caller + test), vùng concurrency/giao dịch.
- Tận dụng vòng lặp agentic: AI "refactor → chạy test → đỏ thì sửa", mình giám sát kế hoạch và review diff cuối.
- Bẫy: refactor vùng không có test rồi tin "không đổi hành vi"; gộp refactor với thêm tính năng → không biết lỗi từ đâu.

</details>

---

### 11.5. Bạn dùng AI để viết test như thế nào? Có rủi ro gì khi để AI tự sinh test?

<details>
<summary>👉 Xem câu trả lời</summary>

- AI giỏi sinh khung test, liệt kê edge case dễ quên, tạo mock/dữ liệu mẫu; nhưng không "khoán trắng" — test là nơi mình chứng minh hiểu hành vi mong đợi.
- Phối hợp: mình quyết định CÁI GÌ đáng test, AI sinh khung + gợi case bỏ sót, mình đọc kỹ từng assertion và bổ sung case nghiệp vụ ("the why").
- Rủi ro: test tautology (luôn pass), test khớp code SAI thay vì yêu cầu đúng, fake coverage (% cao nhưng sai nhánh), mock quá đà.
- Mẹo verify: cố tình đưa một bug nhỏ vào code xem test có đỏ không; test không bắt được bug là test vô dụng.
- Bẫy nguy hiểm nhất: AI sinh test "khớp" với code đang sai → test xanh nhưng bảo vệ bug. Test nên xuất phát từ *spec/yêu cầu*, không từ *code hiện có*.

</details>

---

### 11.6. Trình bày cách bạn làm TDD (Test-Driven Development) cùng AI. Vì sao thứ tự "test trước" lại đặc biệt giá trị khi dùng AI?

<details>
<summary>👉 Xem câu trả lời</summary>

- Vòng red-green-refactor: viết test fail trước → AI viết code tối thiểu cho xanh (tự lặp trong vòng agentic) → dọn code, test vẫn xanh → commit nhỏ.
- "Test trước" biến test thành **spec khách quan** ràng buộc đầu ra AI, thay vì để AI đoán ý mình.
- Giá trị với AI: cho AI mục tiêu rõ ràng; chống test-khớp-bug (test có trước code); là tiêu chí dừng tự động; hợp vòng lặp agentic; buộc mình nghĩ trước (giữ quyền thiết kế).
- Test trở thành cái "rọ" giữ AI không chệch yêu cầu; việc của mình chuyển sang viết test chuẩn và review diff.
- Bẫy: để AI viết cả test lẫn code cùng lúc → cả hai cùng "khớp" với hiểu lầm của AI. Giữ quyền định nghĩa các case quan trọng.

</details>

---

### 11.7. AI sinh tài liệu — bạn dùng nó cho việc gì, và đảm bảo tài liệu không "lệch" với code thật bằng cách nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Dùng AI cho việc lặp, có khuôn mẫu: README, docstring/JSDoc, ADR, commit message, mô tả PR.
- Đảm bảo không lệch: sinh tài liệu TỪ code/diff hiện tại (không từ trí nhớ); đọc lại (lệnh cài/ví dụ code có chạy?); coi tài liệu như code (review, cập nhật cùng PR).
- "The why": AI mô tả tốt code làm GÌ, nhưng lý do/đánh đổi chỉ con người biết → với ADR/commit mình bổ sung lý do thật, AI lo văn phong.
- Rủi ro lệch: doc mô tả tính năng/tham số đã đổi; ADR ghi lý do AI "đoán".
- Bẫy: tài liệu lệch còn tệ hơn không có vì tạo niềm tin sai. Phòng tốt nhất: viết gần code (docstring), cập nhật cùng PR, kiểm tra tự động (doctest, CI kiểm ví dụ README).

</details>

---

### 11.8. Quy trình debug cùng AI của bạn ra sao? Làm sao tránh "sửa mò" và che giấu nguyên nhân gốc?

<details>
<summary>👉 Xem câu trả lời</summary>

- AI là "đôi mắt thứ hai": đọc stack trace/log, đề xuất giả thuyết, tự thêm log chạy lại thu bằng chứng.
- Luôn xác định **root cause** trước khi sửa, tránh vá triệu chứng.
- Quy trình: cung cấp đủ bằng chứng (lỗi, stack trace, cách tái hiện) → yêu cầu AI ĐỀ XUẤT GIẢ THUYẾT xếp theo khả năng, KHÔNG sửa ngay → kiểm chứng bằng dữ liệu → sửa đúng gốc → viết regression test.
- Kỹ thuật khi bí: bắt AI giải thích "vì sao code này CHẠY ĐÚNG" (rubber-duck ngược) thường lộ giả định sai; hoặc tạo test tối thiểu tái hiện bug.
- Bẫy: "đổi đại cho hết đỏ"; trước khi để AI chạy lệnh đổi trạng thái hệ thống (restart/xóa/sửa config) để thử, phải chắc bằng chứng ủng hộ — đừng để nó pattern-match rồi hành động trên giả định.

</details>

---

### 11.9. Khi làm việc với AI coding, bạn xử lý vấn đề bảo mật/bí mật như thế nào? Vì sao không bao giờ được dán secret/khóa cho AI?

<details>
<summary>👉 Xem câu trả lời</summary>

- Nguyên tắc cứng: **không bao giờ dán secret/khóa API/mật khẩu/token/connection string thật** cho AI — chúng có thể bị lưu trong context/log/lịch sử; đã lộ là phải thu hồi (rủi ro vĩnh viễn).
- Cách làm: secret nằm trong `.env` (đã .gitignore), minh hoạ bằng placeholder (`process.env.API_KEY`); không dán PII/dữ liệu thật; dùng dữ liệu giả.
- Cơ chế tất định: hook `PreToolUse` chặn lệnh nguy hiểm/in lộ secret; cấu hình `permissions` trong settings.json; phần auth/crypto/thanh toán dùng skill security-review.
- Rủi ro: lưu vết secret, mẫu auth/crypto sai tinh vi, hardcode khóa, dùng API mã hóa cũ/yếu.
- Bẫy: "dán tạm rồi xoá" — đã qua context thì không lấy lại được. Quy tắc: chuỗi nào lộ phải thu hồi thì không bao giờ xuất hiện trước AI.

</details>

---

### 11.10. Quản lý ngữ cảnh (context) cho AI quan trọng thế nào? Bạn dùng CLAUDE.md và tham chiếu file ra sao để AI làm việc hiệu quả?

<details>
<summary>👉 Xem câu trả lời</summary>

- Chất lượng đầu ra tỉ lệ thuận với chất lượng ngữ cảnh đầu vào; không cung cấp thì AI "đoán" theo trung bình phổ biến và lệch.
- Công cụ: `CLAUDE.md` (convention, lệnh hay dùng, kiến trúc, "luật" — tự nạp, phân cấp); tham chiếu file cụ thể (giảm nhiễu); Memory (nhớ bền qua phiên); `/clear` (tránh nhiễu chéo).
- Nguyên tắc: cung cấp đủ và đúng trọng tâm; giữ context sạch; đưa cả "the why" (ràng buộc nghiệp vụ AI không tự suy ra); tách "nhớ bền" khỏi "ngữ cảnh phiên".
- Cơ chế vs niềm tin: hành vi cần TỰ ĐỘNG mỗi lần (vd format sau khi sửa) phải là **hook**, không phải dòng nhắc trong CLAUDE.md (AI có thể không tuân thủ).
- Bẫy: kỳ vọng AI "tự biết" convention; nhồi quá nhiều thứ không liên quan gây nhiễu. Ngữ cảnh tốt là *đủ và đúng*, không phải *càng nhiều càng tốt*.

</details>

---

### 11.11. Vì sao "commit nhỏ" lại đặc biệt quan trọng khi làm việc với AI? Nó giúp gì cho việc kiểm soát chất lượng?

<details>
<summary>👉 Xem câu trả lời</summary>

- AI sinh lượng thay đổi lớn trong thời gian ngắn → commit nhỏ giúp giữ khả năng đọc/review và **luôn có điểm an toàn để quay lại**.
- Lợi ích: diff nhỏ review được; điểm rollback an toàn; cô lập nguyên nhân lỗi; lịch sử kể chuyện; hợp vòng lặp agentic (mỗi bước xanh → commit → bước tiếp).
- Cộng hưởng với refactor an toàn và TDD: mỗi vòng red-green-refactor kết bằng một commit xanh → chuỗi điểm an toàn liên tục.
- "PR khổng lồ" là cờ đỏ: không ai review nổi, một phần sai thì khó tách.
- Bẫy: để AI chạy tự do rồi commit một cục khổng lồ ở cuối — có lỗi thì khó xác định nằm ở thay đổi nào.

```text
[việc nhỏ] -> [AI sửa] -> [test/lint] -> [review diff] -> [commit nhỏ] -> lặp
Bước hỏng -> revert về commit xanh gần nhất.
```

</details>

---

### 11.12. Khi AI đi sai hướng (lạc đề, càng sửa càng hỏng), bạn xử lý thế nào? Khi nào reset, khi nào định hướng lại?

<details>
<summary>👉 Xem câu trả lời</summary>

- AI đi sai là bình thường; kỹ năng là *nhận ra sớm* và *xử lý dứt khoát*. Hai lựa chọn: định hướng lại (steer) hoặc reset, tùy mức lệch.
- Dấu hiệu sai hướng: cùng lỗi sửa mãi không qua; thay đổi lan ngoài phạm vi; over-engineering; context đã nhiễm thông tin sai; mình không hiểu nó đang làm gì.
- Chọn cách: lệch nhẹ → steer (nói rõ chỗ sai + ràng buộc); context nhiễm → `/clear` rồi làm lại; code hỏng lan rộng → revert commit xanh gần nhất; loanh quanh không thoát → tự làm phần khó.
- Reset hiệu quả: revert về commit nhỏ xanh, `/clear` bỏ context cũ, viết lại prompt rõ (phạm vi/ràng buộc/"đừng làm gì"), quay lại plan mode, chia nhỏ hơn.
- Bẫy lớn nhất: tâm lý "tiếc công" cố cứu vớt thay vì revert. Chi phí làm lại với AI thấp → đừng ngại quay về điểm an toàn.

</details>

## 12. Tư duy và Đánh giá kết quả AI

### 12.1. Làm thế nào để bạn phát hiện một đoạn code do AI sinh ra bị "hallucination" (bịa ra thứ không tồn tại)?

<details>
<summary>👉 Xem câu trả lời</summary>

- Hallucination thường không phải lỗi cú pháp: code trông hợp lý nhưng AI bịa thứ không tồn tại — API/hàm không có thật, tham số sai, package ma, hành vi thư viện bị "tưởng tượng".
- Nguyên tắc: AI sinh ra `claim`, `verify` là việc của tôi. Đặt lớp phòng thủ rẻ sát code: TypeScript/biên dịch, linter, test.
- Với agentic coding (Claude Code): yêu cầu agent tự chạy test/biên dịch — hallucination sẽ lộ ra dưới dạng lỗi compile/test fail.
- Bẫy nguy hiểm: "slopsquatting" — kẻ xấu đăng ký sẵn tên package AI hay bịa để cài vào dính mã độc; luôn kiểm tra dependency mới có thật, đúng tên, đủ uy tín.

| Dạng | Cách kiểm chứng |
| --- | --- |
| API/method không tồn tại | Tra doc chính thức, để IDE/TS báo lỗi |
| Tham số bịa | Đọc type definition / source |
| Package ma | Tra npm/PyPI, xem số download |
| Hành vi sai của thư viện | Viết test nhỏ kiểm chứng |

</details>

---

### 12.2. Bạn đánh giá độ tin cậy của một output do AI tạo ra dựa trên những yếu tố nào? Có phải mọi output đều cần soát kỹ như nhau?

<details>
<summary>👉 Xem câu trả lời</summary>

- Không soát kỹ như nhau cho mọi thứ — đó là lãng phí. Tôi điều chỉnh mức soi xét theo **rủi ro (blast radius)** và **khả năng kiểm chứng** (một ma trận risk-based review).
- Yếu tố cân nhắc: mức ảnh hưởng (tiền/bảo mật/migration → soi kỹ), có test/type bắt lỗi giúp không, độ mới của kiến thức (API mới ra dễ sai), độ đặc thù dự án, độ dài diff.
- Nguyên tắc gốc: độ tin cậy không phải thuộc tính của AI mà là kết quả của quy trình kiểm chứng tôi áp lên.
- Bẫy: đừng để "code trông sạch, tự tin" đánh lừa — giọng văn trôi chảy không phải bằng chứng đúng.

| Vùng code | Mức soát |
| --- | --- |
| Boilerplate CRUD, DTO | Lướt + để test/type chặn |
| Logic nghiệp vụ cốt lõi | Đọc từng dòng, tự suy luận lại |
| Auth / phân quyền / thanh toán | Cực kỹ + test + nhờ người soát |
| Migration / xóa dữ liệu | Đọc kỹ, chạy thử trên môi trường nháp |

</details>

---

### 12.3. Vì sao AI đặc biệt hay sai với các thư viện/API mới ra sau thời điểm huấn luyện (training cutoff)? Bạn xử lý thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Model chỉ "biết" tới mốc training cutoff. Tệ hơn việc "không biết" là: nó vẫn tự tin trả lời, lấp khoảng trống bằng phiên bản cũ hoặc thư viện tương tự → dùng API `deprecated`, sai signature, hoặc bịa hẳn API.
- Cốt lõi xử lý: **bơm ngữ cảnh mới vào** thay vì tin trí nhớ model — dán doc/changelog vào prompt, dùng công cụ đọc web, ghim version + quy ước vào `CLAUDE.md`, để compiler/type của đúng version làm trọng tài.
- Mẹo: yêu cầu agent đọc thẳng type definition trong `node_modules` — đó là "ground truth" đúng version đang cài.
- Bẫy: hỏi "phiên bản mới nhất là gì?" rồi tin — model không biết thật nếu không tra cứu.

```text
Ví dụ ghim CLAUDE.md: "Dự án dùng <lib> v<X>, KHÔNG dùng API cũ <...>"
```

</details>

---

### 12.4. Ai chịu trách nhiệm cuối cùng cho một đoạn code do AI viết và đã được ship lên production? Quan điểm này ảnh hưởng thế nào đến cách bạn làm việc?

<details>
<summary>👉 Xem câu trả lời</summary>

- Thẳng thắn: **tôi — người đặt commit/mở PR — chịu trách nhiệm hoàn toàn.** "AI viết đoạn này" không bao giờ là lời bào chữa.
- Nguyên tắc cứng: **không merge thứ tôi không hiểu và không giải thích được.** Code AI sinh phải qua tay tôi như review PR của một junior.
- "Chạy được nhưng không hiểu vì sao" = cờ đỏ: hoặc đào tới khi hiểu, hoặc viết lại theo cách mình kiểm soát.
- Giữ cơ chế chất lượng độc lập với AI: test, type, review đồng đội, hook lint/test do harness chạy.
- Cách nói: AI là người **đề xuất**, tôi là người **quyết định và bảo chứng**. AI chuyển công việc gõ phím, không chuyển trách nhiệm.

```text
AI  -> tạo draft code
Tôi -> review, kiểm chứng, quyết định merge, chịu trách nhiệm khi lỗi
```

</details>

---

### 12.5. Bạn cân bằng giữa tốc độ mà AI mang lại và nhu cầu hiểu sâu vấn đề như thế nào? Làm sao để không bị "kỹ năng teo lại"?

<details>
<summary>👉 Xem câu trả lời</summary>

- Đánh đổi thật: AI cho tốc độ gõ, nhưng chỉ chấp nhận output mà không hiểu → kỹ năng teo lại (skill atrophy), lúc khẩn cấp không tự xoay xở được.
- Cách cân bằng: phân loại task — boilerplate đã hiểu rõ thì để AI làm nhanh; vùng cốt lõi/kiến trúc thì tự nghĩ trước, dùng AI để phản biện.
- Nguyên tắc: đầu tư hiểu sâu nơi tạo giá trị (kiến trúc, mô hình dữ liệu, đánh đổi); dùng plan mode để "ép mình suy nghĩ"; dùng AI như gia sư (hỏi "vì sao"); định kỳ "tập tay không".
- Cách nói then chốt: AI tăng tốc phần **typing**, phần **thinking** vẫn là của tôi. Bẫy: nhầm "tốc độ gõ code" với "tốc độ giải quyết vấn đề".

</details>

---

### 12.6. Bạn đo lường tác động thực sự của AI lên năng suất như thế nào? Cẩn thận với những chỉ số nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Năng suất thật = **giá trị giao được bền vững**, không phải lượng code sinh ra.
- Vanity metrics cần cảnh giác: số dòng code/commit (AI làm tăng vọt, thường là nợ nhiều hơn), % gợi ý AI được chấp nhận, tốc độ ra PR đầu tiên.
- Tín hiệu đáng tin (kiểu DORA + chất lượng dài hạn): lead time tới production, change failure rate, tỷ lệ rework/số vòng review, bug thoát ra production, nợ kỹ thuật, cảm nhận của đội.
- Cách đo đúng: đo theo **OUTCOME** không theo OUTPUT; so sánh có đối chứng cùng loại task; nhìn cả tốc độ + ổn định.
- Bẫy: tối ưu cái dễ đo (dòng code); cảnh giác hiệu ứng "cảm giác nhanh hơn" trong khi đo thực tế không nhanh hơn — ưu tiên số liệu khách quan.

</details>

---

### 12.7. Khi dùng AI để sinh code, bạn xử lý các vấn đề đạo đức, bản quyền và license của code đầu ra như thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Code AI sinh không tự nhiên "sạch" về pháp lý. Rủi ro: license lây nhiễm (copyleft như GPL khi tái tạo gần nguyên văn), bản quyền/ghi nguồn còn tranh luận, license của dependency AI đề xuất.
- Giảm rủi ro: đoạn càng dài + càng "giống nguyên bản" càng nên viết lại theo cách của mình; kiểm tra license dependency trước khi cài; tự động hóa SCA/license-check trong CI (có thể qua hook); đọc điều khoản của nhà cung cấp AI về quyền dùng output.
- Quan điểm: trách nhiệm pháp lý cuối cùng thuộc về đội/tổ chức ship sản phẩm, không phải công cụ AI; đối xử với output AI như code copy từ internet.

| Rủi ro | Biện pháp |
| --- | --- |
| Tái tạo nguyên văn code có license | Cảnh giác khối lớn, viết lại |
| License dependency mới | Kiểm tra trước khi cài, dùng SCA |
| Lỗ hổng/đạo văn ẩn | Chạy SCA + quét bảo mật trong CI |

</details>

---

### 12.8. Bạn xử lý dữ liệu nhạy cảm và yêu cầu tuân thủ (compliance) như thế nào khi làm việc với công cụ AI và Claude API?

<details>
<summary>👉 Xem câu trả lời</summary>

- Nguyên tắc nền: **những gì đưa vào prompt là dữ liệu rời khỏi tầm kiểm soát** — xử lý nghiêm như gửi ra dịch vụ bên thứ ba, theo data minimization.
- Dữ liệu cần cẩn trọng: secrets (API key, token), PII, dữ liệu khách hàng, code/thông tin kinh doanh, dữ liệu thuộc GDPR/HIPAA/PCI-DSS.
- Cách làm an toàn: KHÔNG dán secrets vào prompt (dùng `.env`/secret manager); chỉ đưa đúng ngữ cảnh cần; mask/redact PII; cấu hình harness (permission hỏi trước, không bừa bãi bypassPermissions, hook `PreToolUse` chặn file nhạy cảm, giới hạn phạm vi thư mục); cẩn thận MCP bên ngoài.
- Với **Claude API**: chọn phương án triển khai hợp yêu cầu lưu trú dữ liệu/compliance, tự lọc/ẩn danh PII ở tầng ứng dụng, không log nguyên văn dữ liệu nhạy cảm (chi tiết chính sách tra doc chính thức).
- Bẫy: vô tình để agent đọc `.env` hay paste log production chứa PII — kiểm soát bằng **hook/permission** (harness cưỡng chế), không bằng kỷ luật ý chí.

</details>

## 13. Câu hỏi hành vi về ứng dụng AI

### 13.1. Hãy mô tả quy trình hằng ngày của bạn khi dùng AI (đặc biệt là Claude Code) trong công việc phát triển phần mềm. AI xen vào những bước nào, và bạn giữ vai trò gì ở mỗi bước?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

- **S:** Làm backend NestJS + Prisma (Jira Clone), mỗi ngày nhận task thêm tính năng/sửa bug/refactor/test; trước đây tốn nhiều thời gian gõ boilerplate và tìm code.
- **T:** Muốn AI gánh phần cơ học và khám phá code, để dồn sức vào thiết kế và quyết định — nhưng vẫn hiểu và chịu trách nhiệm mọi thứ ship.
- **A:** Gắn AI vào từng bước với đúng công cụ: `CLAUDE.md` nạp ngữ cảnh đầu phiên (`/clear` khi đổi task); subagent Explore để tìm code mà không phình context; plan mode duyệt hướng trước khi sửa; permission mode mặc định (hỏi trước) khi thực thi; hook PostToolUse tự lint/test; cuối cùng đọc toàn bộ diff như review PR (dùng skill code-review/security-review cho phần nhạy cảm).
- **R:** Gõ tay giảm rõ rệt, tìm code nhanh hơn, chất lượng không giảm vì hook + review là chốt chặn; vẫn giải thích được mọi thay đổi trong PR.

Lưu ý: nêu cụ thể công cụ cho từng bước để cho thấy "đúng việc dùng đúng công cụ", tránh kiểu "tôi hỏi AI rồi copy". Luôn nhấn mạnh mình là người cầm lái.

</details>

---

### 13.2. Kể về một task cụ thể mà AI giúp bạn tăng tốc đáng kể. Việc đó nếu làm tay sẽ tốn bao lâu, và bạn đã đảm bảo chất lượng không bị hy sinh như thế nào?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

- **S:** Cần thêm validation + DTO nhất quán cho ~12 endpoint kèm test; làm tay mất khoảng 2-3 ngày vì lặp lại và dễ sai do mệt.
- **T:** Rút ngắn thời gian nhưng không hạ chất lượng: khớp convention và có test bao phủ các luồng quan trọng.
- **A:** Mô tả rõ pattern trong prompt, cho AI đọc ví dụ đúng convention từ `CLAUDE.md`; sinh DTO + validation + test theo từng endpoint, chia nhỏ để mỗi diff ngắn dễ review; bật hook tự chạy test; **tự viết phần edge case nghiệp vụ** (ràng buộc quyền, trạng thái hợp lệ) vì AI dễ đoán sai.
- **R:** 2-3 ngày rút còn ~nửa ngày; test bao phủ luồng chính, hook chặn merge code đỏ test; đọc hết diff và giải thích được mọi thay đổi.

Bài học: AI tăng tốc phần "typing", phần "thinking" về edge case vẫn là của mình — đó là nơi giá trị thật. Hãy gắn con số và mô tả cơ chế giữ chất lượng (diff nhỏ, hook test, tự viết edge case).

</details>

---

### 13.3. Kể về một lần AI dẫn bạn đi sai hướng — ví dụ nó bịa ra API, gợi ý một giải pháp sai tinh vi, hoặc tạo code chạy được nhưng sai bản chất. Bạn phát hiện và xử lý ra sao?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

- **S:** Nhờ AI viết phân trang với Prisma; code trông hợp lý, chạy được trên dữ liệu nhỏ, pass test ban đầu — nhưng dùng offset đơn giản và gợi ý một option cấu hình mơ hồ là không tồn tại.
- **T:** Xác định giải pháp có đúng và bền không trước khi đưa vào production, nhất là khi dữ liệu lớn.
- **A:** Hai cờ đỏ khiến dừng lại: không giải thích chắc chắn được behavior khi dữ liệu lớn, và option "nghe đúng" nhưng chưa từng thấy trong tài liệu. Áp quy tắc "không hiểu thì không merge"; đối chiếu tài liệu chính thống Prisma → option đó là hallucination; nhận ra offset chậm dần khi dữ liệu lớn; yêu cầu AI giải thích lựa chọn và đề xuất cursor-based, rồi tự kiểm chứng lại.
- **R:** Thay bằng cursor-based pagination đúng API thật, có test cho dữ liệu lớn. Bài học: kiểm chứng API lạ bằng tài liệu; không kiểm chứng được thì chưa đủ tư cách ship; test trên quy mô thực, không chỉ happy path.

Lưu ý: chọn sự cố THẬT, kể rõ dấu hiệu nghi ngờ và cách kiểm chứng. Đừng đổ lỗi AI; thông điệp đúng là "trust but verify" — không phải "từ đó hết tin AI".

</details>

---

### 13.4. Hãy kể về một lần bạn thuyết phục đồng nghiệp hoặc cả team áp dụng AI (hoặc một quy trình dùng AI) khi họ còn hoài nghi. Bạn đã làm thế nào?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

- **S:** Team có người hoài nghi AI: lo code rác, lo lộ thông tin, cho rằng "tự viết còn nhanh hơn sửa code AI", và sợ teo kỹ năng / "gian lận".
- **T:** Muốn team thử một quy trình dùng AI an toàn, có kiểm soát, chứng minh tăng tốc thật mà không hạ chất lượng — không ép buộc.
- **A:** Không thuyết giảng mà làm demo nhỏ trên task lặp lại ai cũng ghét (boilerplate + test), đo thời gian tay vs Claude Code có review. Trả lời thẳng từng lo ngại: "code rác" → review diff + hook lint/test; "lộ thông tin" → permission mode (hỏi trước) + hook chặn lệnh nguy hiểm; "teo kỹ năng" → "AI làm cơ học, người quyết kiến trúc và edge case". Đề xuất quy trình tối thiểu chung: `CLAUDE.md` chung, skill code-review dùng chung checklist, bắt buộc review diff trước merge.
- **R:** Nhờ demo có số liệu + giải quyết đúng lo ngại, team thử trên task boilerplate; thấy thời gian giảm mà bug không tăng nên dùng nhiều dần. Cả team có chung quy trình an toàn, không vibe coding lén lút.

Lưu ý: thuyết phục bằng bằng chứng cụ thể (demo, số liệu) và lắng nghe lo ngại thật, đề xuất quy trình có kiểm soát thay vì "cứ dùng đi".

</details>

---

### 13.5. Khi dùng AI để đi nhanh, làm sao bạn cân bằng giữa TỐC ĐỘ và CHẤT LƯỢNG? Kể một tình huống bạn phải đánh đổi và lý do bạn chọn như vậy.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

- **S:** Sát deadline demo, cần đồng thời: một prototype dùng một lần cho khách xem, và một endpoint xử lý thanh toán đi production.
- **T:** Phân bổ mức "tin AI" khác nhau cho hai phần vì hậu quả nếu sai rất khác nhau.
- **A:** Nguyên tắc: mức kiểm soát tỉ lệ thuận với rủi ro.
  - Prototype: gần như vibe coding, để AI sinh nhanh, chỉ thử chạy (sẽ vứt, không chạm dữ liệu thật) — tốc độ thắng.
  - Endpoint thanh toán: đi cực chậm — plan mode duyệt hướng, tự viết logic nhạy cảm, đọc từng dòng diff, chạy security-review, test edge case (số âm, race condition, idempotency) — chất lượng thắng.
  - Tách bạch ranh giới: tuyệt đối không để code prototype "trườn" vào production mà không review.
- **R:** Prototype xong kịp demo; endpoint thanh toán an toàn — security-review bắt được chỗ thiếu idempotency. Bài học: năng suất = số tính năng đúng và bền/đơn vị thời gian, không phải số dòng code/giờ.

Lưu ý: thể hiện đây là quyết định có chủ đích theo rủi ro; một lần cố ý đi chậm để giữ chất lượng là điểm cộng lớn (cho thấy sự trưởng thành).

</details>

---

### 13.6. Bạn đã từng hướng dẫn người khác (đồng nghiệp mới, junior) dùng AI hiệu quả chưa? Bạn dạy họ những nguyên tắc gì để họ không rơi vào bẫy lạm dụng?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

- **S:** Một junior dùng AI hăng nhưng nguy hiểm: copy đầu ra vào PR không đọc, PR khổng lồ không ai review nổi, bị hỏi "tại sao làm vậy" thì không giải thích được.
- **T:** Giúp giữ tốc độ AI mang lại nhưng tránh bẫy phụ thuộc quá mức và mất quyền sở hữu code.
- **A:** Pair vài buổi, truyền các nguyên tắc cốt lõi:
  - "Không hiểu thì không merge" — AI là đồng nghiệp cần review, đọc hết diff như PR người thật.
  - Chia task nhỏ để diff ngắn dễ đọc; PR khổng lồ là cờ đỏ.
  - Kiểm chứng API lạ bằng tài liệu chính thống (AI hallucinate thuyết phục).
  - Dùng đúng công cụ: plan mode cho task khó; hook cho việc lặp lại (lint/test) thay vì "nhờ AI nhớ"; subagent Explore để tìm code.
  - Phân biệt khi nào dùng/không: giao mạnh boilerplate; tự làm phần nền tảng để không teo kỹ năng; cực thận trọng với code bảo mật/tiền/dữ liệu.
- **R:** Sau vài tuần junior chia PR nhỏ, đọc diff trước khi mở PR, tự bắt được lỗi AI bịa; giải thích được code của mình. Dạy dùng AI hiệu quả chủ yếu là dạy kỷ luật và tư duy phản biện, không phải mẹo prompt.

Lưu ý: nhấn mạnh dạy nguyên tắc và tư duy (sở hữu code, review, kiểm chứng, đúng công cụ), không chỉ thao tác. Kỹ năng quan trọng nhất thời AI là review được kết quả, không phải viết prompt.

</details>

---

### 13.7. Nhiều người lo AI sẽ thay thế lập trình viên. Quan điểm của bạn là gì? Bạn định vị bản thân thế nào trong bối cảnh đó?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

- **S:** Đã trải qua giai đoạn AI thay đổi cách làm việc mỗi ngày (gánh phần lớn việc gõ code), khiến tự hỏi nghiêm túc giá trị của một lập trình viên giờ nằm ở đâu.
- **T:** Cần một quan điểm rõ ràng để định hướng phát triển bản thân, thay vì hoảng sợ hoặc phủ nhận.
- **A:** Quan điểm: AI thay thế *một số tác vụ*, không thay thế *vai trò kỹ sư*. Nó giỏi tạo văn bản code nhưng không sở hữu trách nhiệm, không hiểu ngữ cảnh nghiệp vụ sâu, không quyết đánh đổi kiến trúc dài hạn, không chịu trách nhiệm khi hệ thống sập lúc 2h sáng. Nút thắt dịch chuyển: chi phí *tạo* code giảm, giá trị dồn về *quyết định viết gì, review, hiểu hệ thống, chịu trách nhiệm*. Vì vậy chủ động định vị mình là "engineer ra quyết định": đầu tư tư duy thiết kế, đọc-hiểu hệ thống, review, dùng AI như bộ khuếch đại; vẫn tự làm phần nền tảng để không teo kỹ năng.
- **R:** AI từ mối đe dọa thành đòn bẩy: làm nhiều hơn ở tầng cao hơn mà vẫn giữ trách nhiệm. Người bị thay thế là lối làm việc chỉ gõ code mà không tư duy, không phải "lập trình viên".

Lưu ý: tránh hai thái cực (hoảng loạn / phủ nhận). Quan điểm chín: AI dịch chuyển *nội dung công việc*; gắn vào việc bạn đang *chủ động* nâng kỹ năng tầng cao (thiết kế, review, ra quyết định).

</details>

---

## 14. Cursor, Copilot và Lựa chọn công cụ AI

### 14.1. GitHub Copilot là gì và đâu là thế mạnh chính của nó?

<details>
<summary>👉 Xem câu trả lời</summary>

- Trợ lý AI lập trình của GitHub, tích hợp trực tiếp vào IDE (VS Code, JetBrains, Visual Studio, Neovim).
- Ba tính năng cốt lõi: **inline suggestion** (gợi ý "ghost text" khi gõ, nhấn `Tab` chấp nhận — mạnh nhất), **Copilot Chat** (hỏi đáp trong IDE), **chế độ agent** (làm task nhiều bước, sửa nhiều file).
- Thế mạnh: gợi ý **nhanh, mượt, ngay tại con trỏ**, ít gián đoạn dòng suy nghĩ; hợp với code lặp lại, boilerplate, hàm nhỏ.
- Khi nào dùng: đã biết rõ muốn viết gì, chỉ cần AI gõ giúp cho nhanh.

```ts
// tính tổng các số chẵn trong mảng → Copilot điền thân hàm:
return nums.filter((n) => n % 2 === 0).reduce((a, b) => a + b, 0);
```

</details>

---

### 14.2. Cursor là gì, và những tính năng nào tạo nên thế mạnh của nó?

<details>
<summary>👉 Xem câu trả lời</summary>

- **IDE "AI-first"** (fork từ VS Code): AI tích hợp sâu vào toàn bộ trải nghiệm code, không chỉ là plugin gắn thêm.
- Tính năng chính: **Tab** (autocomplete đa dòng, dự đoán cả chuỗi thay đổi), **inline edit** (`Cmd/Ctrl-K` sửa bằng ngôn ngữ tự nhiên), **Chat** có ngữ cảnh, **Composer/Agent** (sửa nhiều file, làm feature đa bước), **index codebase**, **@-mention**, **đa model**.
- Thế mạnh: **IDE tích hợp sâu + context toàn dự án** nhờ index codebase và @-mention.
- Khi nào dùng: môi trường code thường ngày có AI mạnh xuyên suốt, cần AI hiểu nhiều file.

Ví dụ inline edit: bôi đen hàm, nhấn `Ctrl-K`, gõ "Refactor dùng async/await thay .then() + thêm try/catch" → Cursor sửa tại chỗ và hiện diff để duyệt.

</details>

---

### 14.3. Claude Code là gì và thế mạnh của nó nằm ở đâu?

<details>
<summary>👉 Xem câu trả lời</summary>

- Công cụ lập trình **agentic** của Anthropic, chạy theo **vòng lặp agentic**: tự đọc/sửa file, chạy lệnh, dùng tool, quan sát kết quả rồi lặp tới khi xong. Có ở CLI, extension VS Code/JetBrains, desktop và web.
- Khả năng đặc trưng: **permission mode** (kiểm soát quyền), **plan mode**, **slash command/Skills/Subagents**, **hooks** (tự chạy script ở các mốc), **MCP** (kết nối tool/dữ liệu ngoài), **workflows**.
- Thế mạnh: **tác vụ đa bước/tự chủ, terminal-native, tùy biến + tự động hóa sâu** (hợp CI/CD, kịch bản lặp lại).
- Khi nào dùng: việc lớn nhiều bước, tự động hóa quy trình, thao tác mạnh ở terminal.

```bash
claude "Thêm validation cho POST /users và viết test, chạy test cho tới khi pass"
```

</details>

---

### 14.4. So sánh ba công cụ: autocomplete vs AI-IDE vs agentic — khác nhau cơ bản ở đâu?

<details>
<summary>👉 Xem câu trả lời</summary>

- Ba "tầng" hỗ trợ AI, từ hẹp tới rộng về mức tự chủ.

| Tiêu chí | Copilot | Cursor | Claude Code |
|----------|---------|--------|-------------|
| Bản chất | Trợ lý AI trong IDE | IDE AI-first | Agentic tool |
| Tương tác | Inline suggestion | Tab + inline edit + Composer | Vòng lặp agentic |
| Context | Quanh con trỏ + file mở | Index codebase, @-mention | Tự khám phá repo, MCP |
| Mức tự chủ | Thấp | Trung bình–cao | Cao |
| Nơi sống | Trong IDE | Là chính IDE | CLI + IDE + desktop/web |

- **Copilot** ≈ autocomplete thông minh **trong** IDE (bạn lái, nó gợi ý).
- **Cursor** ≈ **AI-IDE** có agent + context toàn dự án.
- **Claude Code** ≈ **agentic**, mạnh cho task lớn và tự động hóa.
- Lưu ý: ba thứ **không loại trừ nhau** — nhiều dev dùng kết hợp.

</details>

---

### 14.5. Với từng loại công việc cụ thể, nên chọn công cụ nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Nguyên tắc: chọn theo **quy mô và tính chất** của việc, không phải "tool nào tốt nhất".

| Loại công việc | Công cụ phù hợp |
|----------------|-----------------|
| Gõ từng dòng/hàm, boilerplate | Copilot / Cursor **Tab** |
| Sửa một đoạn theo yêu cầu | Cursor inline edit (`Ctrl-K`) |
| Sửa nhiều file / một feature | Cursor **Composer** / **Claude Code** |
| Tự động hóa / CI / hooks | **Claude Code** |
| Hiểu/khám phá codebase lạ | Cursor (index) / Claude Code |
| Hỏi nhanh giải thích code | Copilot Chat / Cursor Chat |

- Thông điệp khi phỏng vấn: hiểu **dùng đúng tool cho đúng tầng việc** — đó là điều nhà tuyển dụng muốn nghe.

</details>

---

### 14.6. Mô tả một AI-native workflow thực chiến: bạn kết hợp các công cụ như thế nào trong một ngày làm việc?

<details>
<summary>👉 Xem câu trả lời</summary>

- Mấu chốt: **không có công cụ "thắng tất cả"**, phối hợp theo từng giai đoạn.
- **Khám phá & lập kế hoạch**: agentic tool (Claude Code **plan mode**) hoặc Chat để hiểu yêu cầu, đọc codebase, phác kế hoạch.
- **Code thường ngày**: autocomplete (Copilot/Cursor Tab) cho tốc độ gõ, inline edit cho sửa nhỏ.
- **Cụm thay đổi nhiều file**: giao cho Cursor Composer / Claude Code, rồi **review diff** kỹ.
- **Tự động hóa & chất lượng**: Claude Code hooks/skills để format, lint, chạy test; tích hợp CI.
- **Luôn có vòng người duyệt**: dev đọc diff, chạy test, chịu trách nhiệm code cuối.

```text
Hiểu việc       → Claude Code plan mode / Chat
Gõ nhanh        → Copilot / Cursor Tab
Sửa cụm/feature → Cursor Composer / Claude Code agent
Tự động hóa     → Claude Code hooks + workflows + CI
Review          → con người (luôn luôn)
```

</details>

---

### 14.7. Khi nào bạn nên tự gõ code thay vì để AI làm?

<details>
<summary>👉 Xem câu trả lời</summary>

Tự gõ (hoặc kiểm soát rất sát) khi:

- **Logic nghiệp vụ cốt lõi / nhạy cảm**: tính tiền, phân quyền, bảo mật, dữ liệu tài chính — sai một li đi một dặm.
- **Chưa hiểu rõ vấn đề**: để AI viết hộ dễ ra code "trông đúng" nhưng sai bản chất mà bạn không phát hiện.
- **Đoạn quá ngắn/đặc thù**: gõ tay nhanh hơn viết prompt rồi sửa.
- **Khi đang học**: tự viết để hiểu sâu, tránh phụ thuộc autocomplete quá sớm.
- **Khi AI đi vòng lặp sai**: dừng lại tự làm thường rẻ hơn tiếp tục "thúc" nó.
- Quan trọng: **bạn luôn chịu trách nhiệm review**; hiểu được code AI sinh ra là bắt buộc trước khi merge.

```text
Để AI làm:   boilerplate, glue code, test mẫu, refactor cơ học, code đã hiểu rõ
Tự kiểm soát: business logic cốt lõi, bảo mật/phân quyền, thuật toán tinh tế,
              phần đang học, hoặc khi AI lặp sai nhiều lần
```

</details>

---

### 14.8. Khi dùng công cụ AI ở công ty, cần lưu ý gì về bảo mật và chi phí?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Bảo mật & dữ liệu**: code thường được gửi lên cloud nhà cung cấp → đọc kỹ chính sách; ưu tiên cam kết **zero-retention / không train** cho code nhạy cảm; **không để lộ secret** (dùng `.env` ignore, quét secret); **gói enterprise** có audit log, kiểm soát truy cập tốt hơn.
- **Chi phí**: tính theo **per-seat** và/hoặc **theo usage** (token); cân nhắc ROI và chọn model phù hợp (model mạnh đắt hơn — không phải việc nào cũng cần).
- **Tuân thủ & quy trình**: tôn trọng license/bản quyền code AI sinh ra; thiết lập **permission/guardrail** để agent không chạy lệnh nguy hiểm.

```text
Checklist trước khi dùng AI tool ở công ty:
[ ] Chính sách dữ liệu: code có rời máy không? lưu giữ bao lâu?
[ ] Cam kết zero-retention / không train trên dữ liệu của ta?
[ ] Secret loại khỏi context (.env ignore, secret scanning)?
[ ] Gói enterprise (SSO, audit log, kiểm soát truy cập)?
[ ] Chi phí: per-seat hay usage? đã ước tính ROI?
[ ] Có guardrail/permission giới hạn hành động của agent?
```

</details>

---

## ✅ Checklist ôn nhanh

- [ ] Nêu rõ: AI là **trợ lý** giúp tăng tốc, nhưng **bạn** chịu trách nhiệm cuối cùng cho code đã ship.
- [ ] Viết prompt tốt: rõ ràng, đủ ngữ cảnh, nêu ràng buộc, chia nhỏ vấn đề, lặp & tinh chỉnh.
- [ ] Hiểu Claude Code: vòng lặp agentic, permission mode, plan mode, slash command.
- [ ] Phân biệt: **Skill** (quy trình tái dùng), **Subagent** (context riêng/song song), **CLAUDE.md** (rule/ngữ cảnh), **Hook** (tự động hóa qua sự kiện), **MCP** (kết nối tool/dữ liệu ngoài).
- [ ] "Tự động mỗi khi X" ⇒ phải dùng **hook**, không phải nhờ AI ghi nhớ.
- [ ] Luôn **review & verify** code AI sinh ra; cảnh giác hallucination, nhất là API/thư viện mới.
- [ ] Không dán secret/khóa nhạy cảm vào prompt; lưu ý license & dữ liệu nhạy cảm.
- [ ] Chuẩn bị 3–4 câu chuyện thực tế (theo **STAR**) về việc dùng AI trong công việc.

---

> 📌 *Điểm cộng lớn nhất khi phỏng vấn về AI: thể hiện bạn dùng AI có **phán đoán** — biết khi nào tin, khi nào kiểm chứng, và luôn hiểu code mình giao đi.* 🚀
