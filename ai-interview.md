# 🤖 Bộ câu hỏi phỏng vấn: Kỹ năng ứng dụng AI (Claude Code & AI-assisted Development)

> Tài liệu ôn tập **gọn**, chỉ giữ những câu **thực sự hay được hỏi** khi vị trí fullstack/backend có nhắc tới kỹ năng dùng AI coding tools.
> Trọng tâm: tư duy/phán đoán khi dùng AI, prompting cơ bản, review & bảo mật output, và câu chuyện hành vi (STAR) — KHÔNG đi sâu vào MCP/Workflows/Claude API nội bộ vì hiếm khi được hỏi cho vai trò này.
> Mỗi đáp án nằm trong khối **collapse** (`<details>`) — bấm để xem lời giải + ví dụ.
>
> ⚠️ *Lưu ý:* Hệ sinh thái công cụ AI thay đổi rất nhanh. Hãy kiểm chứng lại các tính năng cụ thể với tài liệu chính thức của Claude Code / Anthropic ngay trước buổi phỏng vấn.

## 📚 Mục lục

1. [Tư duy và vai trò của AI](#1-tư-duy-và-vai-trò-của-ai)
2. [Prompting và dùng Claude Code thực tế](#2-prompting-và-dùng-claude-code-thực-tế)
3. [Review, bảo mật và đánh giá output AI](#3-review-bảo-mật-và-đánh-giá-output-ai)
4. [Lựa chọn công cụ AI](#4-lựa-chọn-công-cụ-ai)

---

## 1. Tư duy và vai trò của AI

### 1.1. Bạn nhìn nhận vai trò của AI trong lập trình như thế nào — nó là công cụ thay thế lập trình viên, hay là một dạng "trợ lý"? Vì sao quan điểm này lại quan trọng với chất lượng sản phẩm?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Với mình, AI là một trợ lý cấp cao chứ không phải thứ thay thế lập trình viên. Nó giúp mình gõ code và làm boilerplate nhanh hơn rất nhiều, nhưng phần thiết kế, cân nhắc đánh đổi và ra quyết định kiến trúc vẫn là của mình. Mình hay nói là AI tăng tốc phần 'typing', còn phần 'thinking' thì con người vẫn phải giữ. Quan điểm này quan trọng vì nếu coi AI như người thay thế, mình sẽ ship những đoạn code mà chính mình không hiểu, dẫn đến nợ kỹ thuật và lỗ hổng bảo mật. Cuối cùng AI không chịu trách nhiệm cho sản phẩm, mình mới là người chịu, nên mình luôn giữ vai trò người định hướng và kiểm soát chất lượng."

---

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

### 1.2. "Giữ quyền sở hữu và hiểu rõ code mình ship" nghĩa là gì khi dùng AI? Làm sao bạn đảm bảo điều đó trong thực tế?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Sở hữu code với mình nghĩa là dù AI viết ra, mình vẫn giải thích được từng quyết định, sửa được khi nó hỏng và dám đứng ra chịu trách nhiệm trước team. Quy tắc của mình rất đơn giản: không hiểu thì không merge. Để làm được, mình đọc hết diff trước khi merge giống như đang review PR của một đồng nghiệp, và mình chia task đủ nhỏ để diff luôn ngắn, dễ đọc. Mình cũng hay dùng plan mode để duyệt hướng đi trước khi AI đụng vào code, và yêu cầu nó giải thích vì sao chọn cách đó. Quyền sở hữu thể hiện rõ nhất lúc có sự cố, khi đó người thật sự hiểu code mới là người cứu được hệ thống."

---

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

### 1.3. Rủi ro của việc phụ thuộc quá mức vào AI là gì? Bạn nhận biết và phòng tránh ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Rủi ro lớn nhất mình thấy là teo kỹ năng và tin mù vào AI, kiểu nhận đề xuất mà không hỏi tại sao, hoặc merge code mà không đọc. Lâu dần mình mất luôn cảm giác về hệ thống và không debug nổi khi AI sai. Để phòng tránh, mình vẫn tự tay làm những phần cốt lõi và khó để giữ tay nghề, đồng thời luôn kiểm chứng API hay khái niệm lạ bằng tài liệu chính thống thay vì tin ngay. Một điểm mình rất chú trọng là phân biệt việc tất định với việc cần suy luận: những thứ lặp đi lặp lại như format hay lint thì mình để hook trong settings.json lo, vì AI có thể quên còn harness thì luôn chạy. Còn dấu hiệu cảnh báo phụ thuộc quá mức thì mình tự soi: nếu thấy mình copy code mà không hiểu là phải dừng lại ngay."

---

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

## 2. Prompting và dùng Claude Code thực tế

### 2.1. Một prompt hiệu quả cho công cụ agentic coding (như Claude Code) gồm những thành phần nào? Hãy phân biệt một prompt "yếu" và một prompt "mạnh".

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Theo mình một prompt mạnh cần đủ bốn thứ: mục tiêu rõ, ngữ cảnh như file hoặc lỗi cụ thể, ràng buộc về thư viện và quy ước, và định dạng đầu ra mong muốn. Prompt yếu thì chỉ mô tả triệu chứng mơ hồ rồi để AI tự đoán, còn prompt mạnh thu hẹp không gian đoán bằng ngữ cảnh và ràng buộc cụ thể. Với Claude Code thì hay ở chỗ nó là agentic, mình không cần dán cả file mà chỉ cần chỉ đường dẫn và nêu mục tiêu, agent sẽ tự đọc. Nhưng đúng vì nó chủ động hành động nên mình càng phải nêu rõ ràng buộc, ví dụ cấm thêm dependency hay cấm đụng file ngoài phạm vi. Nếu prompt yếu, AI dễ tự sáng tác yêu cầu, đổi luôn cả kiến trúc mà mình không ngờ tới."

---

- Prompt mạnh có đủ 4 yếu tố: mục tiêu rõ, ngữ cảnh (file/lỗi/kiến trúc), ràng buộc (quy ước, thư viện, điều cấm), định dạng đầu ra (diff / chỉ sửa file X / kèm test).
- Prompt yếu mô tả triệu chứng mơ hồ, để AI tự đoán; prompt mạnh thu hẹp không gian đoán bằng ngữ cảnh + ràng buộc cụ thể.
- Claude Code là agentic: không cần dán cả file, chỉ cần chỉ đường dẫn và nêu mục tiêu — agent tự đọc. Nhưng vẫn phải nêu rõ ràng buộc vì agent sẽ chủ động hành động.
- Bẫy: prompt yếu khiến AI tự "sáng tác" yêu cầu — thêm dependency, đổi kiến trúc, sửa file ngoài phạm vi.

| Prompt yếu | Prompt mạnh |
|---|---|
| "Sửa cái form login đi" | "Trong `LoginForm.tsx`, nút submit không disable khi gọi API. Hãy disable khi `isPending` của mutation chạy. Dùng React Query sẵn có, không thêm state thủ công." |

</details>

---

### 2.2. Thư mục `.claude/` trong một dự án gồm những gì? Mỗi phần có tác dụng gì?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Thư mục .claude là nơi cấu hình Claude Code cho dự án, đặt trong dự án thì áp riêng cho dự án đó, còn đặt ở home thì áp toàn cục. Quan trọng nhất là CLAUDE.md, nó như bộ nhớ của dự án, nạp vào context mỗi phiên để dạy AI về quy ước, cấu trúc và lệnh hay dùng. Tiếp theo là settings.json lo phần harness như permissions, env và khai báo hooks. Còn lại thì mình phân biệt theo cách kích hoạt: commands là slash command mình chủ động gọi, skills là AI tự nạp khi thấy task phù hợp, agents là ủy thác cho một AI con chạy với context riêng, và hooks là script tự động chạy theo sự kiện. Cái mình nhấn mạnh là hooks tất định, không phụ thuộc AI có nhớ hay không, nên những việc kỷ luật lặp lại mình giao cho hooks."

---

- `.claude/` là nơi cấu hình Claude Code cho dự án. Đặt ở thư mục dự án → áp cho dự án đó; đặt ở `~/.claude/` (home) → áp toàn cục mọi dự án.

| Thành phần | Tác dụng |
|---|---|
| `CLAUDE.md` | **Bộ nhớ/chỉ dẫn dự án**, nạp vào context mỗi phiên: quy ước, cấu trúc codebase, lệnh hay dùng, điều phải tuân thủ. File quan trọng nhất để "dạy" AI về dự án. |
| `settings.json` | **Cấu hình harness**: permissions (cho phép/chặn lệnh), env vars, khai báo hooks, model mặc định. |
| `agents/` | Định nghĩa **subagent** tùy chỉnh — mỗi file mô tả 1 agent chuyên biệt (vai trò, tool, model) chạy với context riêng. |
| `commands/` | **Slash command** tùy chỉnh — mỗi file `.md` là một lệnh `/<tên>` đóng gói prompt/quy trình lặp lại. |
| `skills/` | **Skill** — folder có `SKILL.md`, nạp hướng dẫn chuyên môn *theo nhu cầu* (AI tự đọc khi task liên quan). |
| `hooks/` | Script chạy **tự động** theo sự kiện (trước/sau tool, khi dừng…); phần kích hoạt khai báo trong `settings.json`. |
| `rules/` | Thường là **quy ước riêng của dự án** (coding standard) được `CLAUDE.md` trỏ tới — không phải mục built-in chuẩn. |

- Phân biệt nhanh: **commands** = mình chủ động gọi (`/lệnh`); **skills** = AI tự nạp khi thấy phù hợp; **agents** = ủy thác cho một AI con chạy riêng; **hooks** = tự động tất định, không phụ thuộc AI nhớ.

</details>

---

## 3. Review, bảo mật và đánh giá output AI

### 3.1. Quy trình review code do AI sinh ra của bạn gồm những bước nào? Vì sao không thể chỉ "chạy thử thấy được là merge"?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Mình review code AI sinh ra y như review PR của đồng nghiệp, thậm chí kỹ hơn, vì AI luôn tự tin đều đặn kể cả lúc nó sai. Quy trình của mình là đọc kỹ diff từng dòng, chạy test, lint và thử app thật, rồi tự giải thích được vì sao đoạn code này đúng trước khi merge. Lý do không thể chỉ chạy thử thấy được là merge: chạy được chỉ chứng minh happy path, nó không nói gì về edge case, lỗ hổng bảo mật hay nợ kỹ thuật. Mình hay nói là khi dùng AI thì nút thắt dịch chuyển từ viết code sang review code, nên mình dành nhiều công sức nhất cho khâu này. Bẫy lớn nhất là tưởng nhầm tốc độ sinh code bằng tốc độ giải quyết vấn đề."

---

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

### 3.2. Khi làm việc với AI coding, bạn xử lý vấn đề bảo mật/bí mật như thế nào? Vì sao không bao giờ được dán secret/khóa cho AI?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Nguyên tắc cứng của mình là không bao giờ dán secret, khóa API, mật khẩu hay connection string thật cho AI, vì chúng có thể bị lưu trong context, log hoặc lịch sử, mà đã lộ thì coi như phải thu hồi và đổi mới hết. Trong thực tế thì secret luôn nằm trong .env đã gitignore, còn khi nói chuyện với AI mình chỉ dùng placeholder kiểu process.env.API_KEY, và tuyệt đối không đưa PII hay dữ liệu thật, cần test thì dùng dữ liệu giả. Mình cũng dựng cơ chế tất định như hook PreToolUse để chặn lệnh nguy hiểm hay in lộ secret, và cấu hình permissions trong settings.json. Mình không tin vào kiểu 'dán tạm rồi xóa', vì một khi chuỗi đã đi qua context thì không lấy lại được. Với mình, bất kỳ chuỗi nào lộ là phải thu hồi thì không bao giờ được xuất hiện trước AI."

---

- Nguyên tắc cứng: **không bao giờ dán secret/khóa API/mật khẩu/token/connection string thật** cho AI — chúng có thể bị lưu trong context/log/lịch sử; đã lộ là phải thu hồi (rủi ro vĩnh viễn).
- Cách làm: secret nằm trong `.env` (đã .gitignore), minh hoạ bằng placeholder (`process.env.API_KEY`); không dán PII/dữ liệu thật; dùng dữ liệu giả.
- Cơ chế tất định: hook `PreToolUse` chặn lệnh nguy hiểm/in lộ secret; cấu hình `permissions` trong settings.json; phần auth/crypto/thanh toán dùng skill security-review.
- Rủi ro: lưu vết secret, mẫu auth/crypto sai tinh vi, hardcode khóa, dùng API mã hóa cũ/yếu.
- Bẫy: "dán tạm rồi xoá" — đã qua context thì không lấy lại được. Quy tắc: chuỗi nào lộ phải thu hồi thì không bao giờ xuất hiện trước AI.

</details>

---

### 3.3. Làm thế nào để bạn phát hiện một đoạn code do AI sinh ra bị "hallucination" (bịa ra thứ không tồn tại)?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Hallucination thường không phải lỗi cú pháp, mà là code trông rất hợp lý nhưng AI bịa ra thứ không tồn tại, ví dụ một hàm hay tham số không có thật, một package ma, hoặc hiểu sai hành vi thư viện. Cách mình phát hiện là coi mỗi thứ AI sinh ra là một claim, còn verify là việc của mình. Mình đặt sẵn các lớp phòng thủ rẻ và sát code như TypeScript, linter và test, hallucination sẽ lộ ngay dưới dạng lỗi compile hoặc test fail. Với Claude Code thì mình yêu cầu agent tự chạy test và biên dịch, cái gì bịa ra là tự nó phơi ra. Một thứ mình rất cảnh giác là slopsquatting, kẻ xấu đăng ký sẵn những tên package mà AI hay bịa để cài mã độc, nên mỗi dependency mới mình đều kiểm tra là nó có thật, đúng tên và đủ uy tín."

---

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

### 3.4. Ai chịu trách nhiệm cuối cùng cho một đoạn code do AI viết và đã được ship lên production? Quan điểm này ảnh hưởng thế nào đến cách bạn làm việc?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Câu này mình trả lời rất thẳng: người đặt commit và mở PR chịu trách nhiệm hoàn toàn, tức là mình, chứ 'AI viết đoạn này' không bao giờ là lời bào chữa. Quan điểm đó định hình toàn bộ cách mình làm việc: code AI sinh ra phải qua tay mình như review PR của một junior, và mình không merge thứ mình không hiểu hay không giải thích được. Nếu gặp tình huống chạy được nhưng không hiểu vì sao, mình coi đó là cờ đỏ, hoặc đào tới khi hiểu hoặc viết lại theo cách mình kiểm soát. Mình cũng giữ các cơ chế chất lượng độc lập với AI như test, type, review của đồng đội và hook lint do harness chạy. Tóm lại, AI là người đề xuất, còn mình là người quyết định và bảo chứng, nó chuyển công việc gõ phím chứ không chuyển trách nhiệm."

---

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

## 4. Lựa chọn công cụ AI

### 4.1. Với từng loại công việc cụ thể, nên chọn công cụ nào (Copilot / Cursor / Claude Code)?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Mình chọn công cụ theo quy mô và tính chất của việc, chứ không đi tìm tool nào tốt nhất. Khi chỉ gõ từng dòng hay boilerplate thì autocomplete kiểu Copilot hoặc Cursor Tab là nhanh nhất, còn sửa một đoạn theo yêu cầu cụ thể thì mình dùng inline edit. Khi việc trải ra nhiều file hoặc cả một feature, mình chuyển sang Claude Code hoặc Cursor Composer vì chúng làm việc ở tầng cao hơn. Riêng phần tự động hóa, CI hay hooks thì mình ưu tiên Claude Code vì nó mạnh ở agentic và cấu hình harness. Thông điệp mình muốn truyền tải là biết dùng đúng tool cho đúng tầng việc, đó mới là điều quan trọng chứ không phải trung thành với một công cụ duy nhất."

---

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

## ✅ Checklist ôn nhanh

- [ ] Nêu rõ: AI là **trợ lý** giúp tăng tốc, nhưng **bạn** chịu trách nhiệm cuối cùng cho code đã ship.
- [ ] Viết prompt tốt: rõ ràng, đủ ngữ cảnh, nêu ràng buộc, chia nhỏ vấn đề, lặp & tinh chỉnh.
- [ ] Biết `.claude/` có gì: `CLAUDE.md` (ngữ cảnh), `settings.json` (cấu hình + hook), `agents`/`commands`/`skills`/`hooks`.
- [ ] Luôn **review & verify** code AI sinh ra; cảnh giác hallucination, nhất là API/thư viện mới.
- [ ] Không dán secret/khóa nhạy cảm vào prompt; lưu ý license & dữ liệu nhạy cảm.
- [ ] Chuẩn bị 3–4 câu chuyện thực tế (theo **STAR**) về việc dùng AI trong công việc.

---

> 📌 *Điểm cộng lớn nhất khi phỏng vấn về AI: thể hiện bạn dùng AI có **phán đoán** — biết khi nào tin, khi nào kiểm chứng, và luôn hiểu code mình giao đi.* 🚀
