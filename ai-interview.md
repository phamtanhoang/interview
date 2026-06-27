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

Tôi xem AI (đặc biệt là agentic coding như Claude Code) là một pair-programmer cấp cao, không phải thứ thay thế tư duy của tôi. Nó cực giỏi việc tạo ra văn bản code đúng cú pháp với tốc độ rất nhanh, nhưng nó không chịu trách nhiệm về sản phẩm — tôi mới là người chịu trách nhiệm. Vai trò của tôi chuyển từ "người gõ từng dòng" sang "người định hướng, ra quyết định kiến trúc, và kiểm soát chất lượng".

Tư duy đúng có thể tóm tắt bằng cách phân tách rõ trách nhiệm:

| Khía cạnh | AI làm tốt | Con người vẫn phải giữ |
| --- | --- | --- |
| Tốc độ tạo code | Rất nhanh, đặc biệt boilerplate | Quyết định có nên viết phần đó không |
| Phạm vi | Local, theo prompt | Bức tranh tổng thể, ràng buộc nghiệp vụ |
| Kiến thức | Rộng nhưng có thể lỗi thời/sai | Ngữ cảnh dự án, lý do (the "why") |
| Trách nhiệm | Không có | Toàn bộ — review, an toàn, vận hành |

Vì sao quan điểm này quan trọng: nếu coi AI là "người thay thế", lập trình viên sẽ ship code mà không hiểu, dẫn tới nợ kỹ thuật và lỗ hổng bảo mật mà không ai phát hiện được. Nếu coi AI là "trợ lý có siêu năng lực", tôi tận dụng tốc độ của nó nhưng vẫn giữ vai trò "engineer ra quyết định" — đó mới là nơi tạo ra giá trị thật.

Một cách diễn đạt tôi hay dùng: AI làm tăng tốc phần "typing", nhưng phần "thinking" (thiết kế, đánh đổi, đọc hiểu hệ thống) vẫn là của tôi. Bẫy thường gặp là nhầm tốc độ gõ code với tốc độ giải quyết vấn đề — hai thứ này khác nhau.

</details>

---

### 1.2. Phân biệt "AI-assisted coding" và "vibe coding". Khi nào mỗi cách là phù hợp?

<details>
<summary>👉 Xem câu trả lời</summary>

Hai cách dùng AI khác nhau ở mức độ lập trình viên hiểu và sở hữu code đầu ra:

| Tiêu chí | AI-assisted coding | Vibe coding |
| --- | --- | --- |
| Người lái | Lập trình viên chủ động, AI hỗ trợ | Để AI tự sinh, chỉ "cảm nhận" kết quả |
| Đọc code đầu ra | Đọc kỹ, review từng phần | Hiếm khi đọc, chỉ thử chạy |
| Hiểu code | Hiểu rõ, có thể giải thích lại | Thường không hiểu chi tiết |
| Mục tiêu | Code đi vào production, bền vững | Prototype nhanh, thử ý tưởng |
| Rủi ro | Thấp nếu review tốt | Cao: nợ kỹ thuật, lỗ hổng ẩn |

"Vibe coding" là kiểu mô tả mong muốn rồi chấp nhận bất cứ thứ gì AI sinh ra mà không soát kỹ — "code chạy là được". Nó hợp cho prototype dùng một lần, hackathon, thử nghiệm ý tưởng, hoặc khám phá một API lạ để học. Nhưng nguy hiểm khi đưa vào code base thật.

"AI-assisted coding" là tôi vẫn cầm lái: tôi quyết định kiến trúc, tách việc cho AI làm từng phần, đọc và review đầu ra, yêu cầu sửa, rồi mới merge. Đây là cách phù hợp cho code production.

```text
Khi nào dùng cái nào:
- Prototype/POC dùng một lần, sẽ vứt đi  -> vibe coding chấp nhận được
- Học một thư viện mới trong môi trường nháp -> vibe coding ổn
- Tính năng đi vào production            -> AI-assisted (đọc + review bắt buộc)
- Code đụng tới bảo mật/tiền/dữ liệu     -> AI-assisted + review cực kỹ
```

Bẫy lớn nhất: vibe coding một prototype rồi vô tình để nó "trườn" thẳng vào production mà không qua bước review. Ranh giới giữa hai cách không nằm ở công cụ, mà nằm ở việc tôi có thật sự hiểu và sở hữu code hay không.

</details>

---

### 1.3. Khi nào bạn NÊN dùng AI trong một task, và khi nào bạn quyết định KHÔNG dùng (hoặc dùng rất hạn chế)?

<details>
<summary>👉 Xem câu trả lời</summary>

Tôi quyết định dựa trên hai trục: (1) mức độ rủi ro/độ phức tạp ngữ cảnh của task, và (2) việc tôi có khả năng kiểm chứng đầu ra hay không. AI tỏa sáng khi task lặp lại, có khuôn mẫu rõ, và tôi đủ kiến thức để xác minh kết quả.

NÊN dùng AI:
- Boilerplate, code lặp khuôn (CRUD, DTO, controller, mapper).
- Refactor cơ học (đổi tên xuyên file, tách hàm, chuyển callback sang async/await).
- Viết test cho code đã hiểu rõ, sinh dữ liệu mẫu.
- Viết/cập nhật tài liệu, comment, commit message.
- Khám phá code base lạ — dùng Explore subagent để tìm "logic X nằm ở đâu".
- Debug khi tôi cần một "đôi mắt thứ hai" gợi giả thuyết.
- Học khái niệm mới, dịch ý tưởng từ ngôn ngữ này sang ngôn ngữ khác.

NÊN hạn chế / KHÔNG dùng (hoặc giám sát rất chặt):
- Quyết định kiến trúc cốt lõi và đánh đổi dài hạn — đây là việc của con người.
- Code chạm tới bảo mật, mật mã, xác thực, thanh toán — AI có thể tạo ra mẫu sai tinh vi.
- Logic nghiệp vụ phức tạp mà tôi chưa hiểu — nếu tôi không kiểm chứng được, tôi không nên ship.
- Khi ngữ cảnh quá lớn/đặc thù mà tôi không thể truyền đủ cho AI, nó dễ "đoán" sai.

```text
Quy tắc lọc nhanh trước khi giao việc cho AI:
1. Mình có hiểu vấn đề đủ để REVIEW đầu ra không?
   - Không -> tự làm trước, hoặc dùng AI để HỌC, đừng để nó QUYẾT.
2. Sai thì hậu quả tới đâu (bảo mật/tiền/dữ liệu)?
   - Nghiêm trọng -> AI hỗ trợ, nhưng review gấp đôi.
3. Task có khuôn mẫu rõ và lặp lại không?
   - Có -> giao mạnh cho AI, đây là điểm mạnh nhất của nó.
```

Bẫy: dùng AI cho đúng việc mình KHÔNG biết cách kiểm chứng. Khi đó bạn không thể phân biệt câu trả lời đúng với câu trả lời "nghe có vẻ đúng".

</details>

---

### 1.4. Hãy liệt kê những loại công việc AI giúp ích nhiều nhất trong vòng đời phát triển, và với mỗi loại, AI giúp bằng cách nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Tôi nhóm theo các giai đoạn quen thuộc của một task. Với agentic coding như Claude Code, điểm mạnh là nó tự đọc file, chạy lệnh và lặp lại — nên nó làm tốt cả những việc cần "đi vòng quanh" code base.

| Loại việc | AI giúp bằng cách nào | Lưu ý / tôi vẫn kiểm soát |
| --- | --- | --- |
| Boilerplate | Sinh CRUD, DTO, cấu hình, scaffold module | Kiểm tra có khớp convention dự án (CLAUDE.md) |
| Refactor | Đổi tên xuyên file, tách hàm, dọn code | Phải có test bao phủ trước khi để AI sửa rộng |
| Test | Sinh unit/integration test, liệt kê edge case | Tôi quyết case nào đáng test, đọc kỹ assertion |
| Tài liệu | Viết README, JSDoc, ADR, commit message | Đảm bảo tài liệu khớp hành vi thật |
| Học | Giải thích khái niệm, so sánh cách tiếp cận | Đối chiếu với tài liệu chính thống |
| Debug | Đề xuất giả thuyết, đọc stack trace, đọc log | Tôi xác nhận nguyên nhân gốc, không sửa mò |
| Khám phá | Subagent Explore tìm "code X ở đâu" | Đọc lại để hiểu, không tin tuyệt đối |

Một ví dụ thực tế trong dự án NestJS của tôi: khi thêm một resource mới, tôi mô tả schema và để Claude Code sinh module + service + controller + DTO theo đúng convention đã ghi trong `CLAUDE.md`, sau đó tôi review từng phần, bổ sung logic nghiệp vụ và viết test cho những đường đi quan trọng. Phần "gõ tay" được rút ngắn rất nhiều, còn phần "suy nghĩ" vẫn là của tôi.

```text
Quy tắc: AI làm phần 80% có khuôn mẫu (mechanical),
con người tập trung vào 20% quyết định và cạm bẫy nghiệp vụ.
```

Bẫy: kỳ vọng AI tự biết convention của dự án. Nó chỉ biết nếu bạn cung cấp ngữ cảnh (qua `CLAUDE.md`, ví dụ mẫu trong code base, hoặc mô tả rõ trong prompt).

</details>

---

### 1.5. "Giữ quyền sở hữu và hiểu rõ code mình ship" nghĩa là gì khi dùng AI? Làm sao bạn đảm bảo điều đó trong thực tế?

<details>
<summary>👉 Xem câu trả lời</summary>

"Sở hữu code" nghĩa là: dù AI viết ra, tôi vẫn phải giải thích được từng quyết định trong đó, sửa được khi nó hỏng lúc 2 giờ sáng, và chịu trách nhiệm trước team. Nếu tôi không thể giải thích một đoạn code trong PR của mình, thì đoạn đó chưa sẵn sàng để ship. Quy tắc của tôi rất đơn giản: "không hiểu thì không merge".

Cách tôi đảm bảo điều này trong thực tế:

- Đọc toàn bộ diff trước khi merge, đúng như khi review PR của đồng nghiệp. AI là một "đồng nghiệp" cần được review.
- Chia task nhỏ để diff đủ ngắn mà đọc hiểu được. PR khổng lồ do AI sinh ra là cờ đỏ.
- Dùng plan mode của Claude Code: yêu cầu nó trình bày kế hoạch trước khi sửa code, để tôi duyệt hướng đi trước khi có một dòng code nào được viết.
- Yêu cầu AI giải thích "vì sao chọn cách này", rồi tôi đánh giá lập luận đó.
- Tự viết hoặc tự duyệt kỹ test — test là cách tôi chứng minh mình hiểu hành vi mong đợi.
- Với phần lạ, tôi nhờ AI giải thích lại cho tôi, rồi tự kiểm chứng bằng tài liệu.

```text
Checklist trước khi merge code có AI tham gia:
[ ] Tôi đọc hết diff chưa?
[ ] Tôi giải thích được từng thay đổi không?
[ ] Có test cho đường đi quan trọng chưa?
[ ] Code có khớp convention dự án (CLAUDE.md) không?
[ ] Có chỗ nào "magic" mà tôi không hiểu không?  -> dừng lại, làm rõ
```

Bẫy thường gặp: AI tạo code chạy đúng trên happy path nhưng xử lý sai edge case hoặc dùng một API đã deprecated. Nếu chỉ "chạy được là merge", bạn sẽ phát hiện ra khi đã quá muộn. Quyền sở hữu thể hiện đúng nhất ở lúc sự cố xảy ra — người hiểu code là người cứu được hệ thống.

</details>

---

### 1.6. Rủi ro của việc phụ thuộc quá mức vào AI là gì? Bạn nhận biết và phòng tránh ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

Phụ thuộc quá mức (over-reliance) là khi lập trình viên không còn khả năng làm việc hiệu quả nếu thiếu AI, hoặc tệ hơn, mất khả năng đánh giá đúng/sai của đầu ra. Các rủi ro chính:

- Teo kỹ năng (skill atrophy): lâu ngày không tự đọc/viết những phần nền tảng, mất "trực giác" để phát hiện code có mùi.
- Mất kiến thức ngữ cảnh: AI không nắm lý do lịch sử của hệ thống; nếu tôi cũng không nắm, không ai trong team thật sự hiểu.
- Tin mù (automation bias): xu hướng tin đầu ra của máy ngay cả khi nó sai một cách tinh vi.
- Code "nghe có vẻ đúng": AI có thể bịa (hallucinate) API hoặc tạo ra mẫu sai trông rất thuyết phục.
- Đồng nhất hóa giải pháp: luôn nhận cách làm "trung bình phổ biến", bỏ lỡ giải pháp tối ưu cho ngữ cảnh riêng.

Dấu hiệu nhận biết tôi đang phụ thuộc quá mức: tôi merge code mà không đọc; tôi không thể debug nếu AI offline; tôi copy đề xuất mà không tự hỏi "tại sao".

Cách phòng tránh:

```text
- Giữ thói quen tự làm những phần cốt lõi/khó để duy trì kỹ năng.
- Luôn review như review đồng nghiệp; không bao giờ merge thứ mình không hiểu.
- Kiểm chứng API/khái niệm AI đưa ra bằng tài liệu chính thống (đặc biệt khi nghi ngờ hallucination).
- Để máy lo phần kỷ luật lặp lại bằng HOOK (vd PostToolUse chạy lint/test, hook chặn lệnh nguy hiểm),
  thay vì "nhờ AI nhớ" — vì hành vi tự động phải do harness đảm bảo, không dựa vào trí nhớ của AI.
- Đặt permission mode hợp lý (mặc định hỏi trước; chỉ acceptEdits khi đã tin tưởng phạm vi).
```

Một điểm tôi nhấn mạnh về tư duy hệ thống: nếu muốn "mỗi lần sửa file phải tự động format/lint", đó phải là một hook trong `settings.json` chứ không phải lời nhắc AI — vì AI có thể quên, còn hook thì harness luôn chạy. Phân biệt được "việc cần tất định (deterministic)" và "việc cần suy luận" chính là dấu hiệu của người dùng AI trưởng thành.

</details>

---

### 1.7. AI ảnh hưởng thế nào tới năng suất VÀ chất lượng? Năng suất tăng có luôn đồng nghĩa chất lượng tăng không?

<details>
<summary>👉 Xem câu trả lời</summary>

Hai thứ này không tự động đi cùng nhau. AI gần như chắc chắn làm tăng tốc độ tạo code (throughput), nhưng chất lượng phụ thuộc vào quy trình kiểm soát quanh nó. Nếu bỏ qua review, năng suất "tăng" chỉ là đẩy nhanh việc tạo ra nợ kỹ thuật và bug.

Tôi phân biệt rõ:

| | Năng suất (tốc độ) | Chất lượng (độ bền vững) |
| --- | --- | --- |
| AI tác động trực tiếp | Tăng mạnh phần gõ code, scaffold | Trung tính — tùy quy trình của bạn |
| Rủi ro nếu lạm dụng | Tạo PR khổng lồ khó review | Bug ẩn, nợ kỹ thuật, lỗ hổng bảo mật |
| Yếu tố quyết định | Mô tả task rõ, ngữ cảnh tốt | Review, test, hook tự động, kiến trúc |

Năng suất thật sự không phải "số dòng code/giờ" mà là "số tính năng đúng, bền, ship được/đơn vị thời gian". Nếu tôi sinh ra gấp ba lượng code nhưng tốn gấp năm thời gian sửa bug về sau, đó là năng suất ÂM.

Cách tôi để AI tăng cả hai cùng lúc:

```text
- Tốc độ: giao boilerplate/refactor/test cho AI; dùng subagent chạy SONG SONG
  cho các nhánh tìm kiếm độc lập; dùng prompt caching để giảm chi phí & độ trễ khi gọi API.
- Chất lượng: bắt buộc review diff; có test bao phủ; dùng hook tự động lint/test;
  dùng skill chuyên biệt như code-review / security-review cho các phần nhạy cảm.
```

Một quan sát thực tế: AI làm giảm chi phí tạo ra "lượng" code, nên nút thắt chuyển dịch sang khâu review và hiểu code. Vì vậy người dùng AI giỏi sẽ đầu tư mạnh vào tự động hóa kiểm soát chất lượng (test, hook, CI) để khâu review không trở thành điểm nghẽn mới. Tăng tốc độ tạo code mà không tăng năng lực review tương ứng là công thức dẫn tới khủng hoảng chất lượng.

Bẫy lớn nhất: đo năng suất bằng cảm giác "code ra nhanh". Hãy đo bằng kết quả cuối: tính năng đúng, ít bug, dễ bảo trì.

</details>

---

### 1.8. Hãy kể về một lần bạn dùng AI như một "siêu năng lực" để hoàn thành việc mà nếu không có AI sẽ tốn hơn rất nhiều — nhưng bạn vẫn giữ quyền kiểm soát chất lượng.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Đây là câu hỏi hành vi — hãy kể một câu chuyện cụ thể, có quy trình rõ ràng, theo cấu trúc STAR. Dưới đây là một mẫu bạn có thể điều chỉnh theo trải nghiệm thật:

- **Situation (Bối cảnh):** "Trong dự án Jira Clone (NestJS + Prisma), tôi cần thêm một lớp audit log cho hàng loạt resource hiện có, đồng thời bổ sung test cho phần đó. Làm tay sẽ rất lặp lại và tốn nhiều ngày."
- **Task (Nhiệm vụ):** "Tôi muốn rút ngắn thời gian xuống còn một phần nhỏ, nhưng tuyệt đối không hạ chất lượng — code phải khớp convention, có test, và tôi phải hiểu hết để chịu trách nhiệm."
- **Action (Hành động):**
  - "Tôi viết rõ convention và kiến trúc vào `CLAUDE.md` để Claude Code tự nạp ngữ cảnh, tránh phải nhắc lại mỗi lần."
  - "Tôi dùng plan mode yêu cầu AI trình bày kế hoạch trước; tôi duyệt hướng đi rồi mới cho sửa code, nên không bị 'lạc' giữa chừng."
  - "Tôi chia nhỏ thành từng resource để mỗi diff đủ ngắn mà review kỹ; với phần tìm hiểu code cũ tôi dùng subagent Explore để định vị nhanh các chỗ cần chạm."
  - "Tôi cấu hình một hook PostToolUse tự chạy lint và test mỗi khi file thay đổi — kỷ luật chất lượng do harness đảm bảo, không phụ thuộc việc AI có nhớ hay không."
  - "Phần logic nhạy cảm, tôi tự viết hoặc đọc cực kỹ; tôi từ chối merge bất cứ đoạn nào mình không giải thích được."
- **Result (Kết quả):** "Khối lượng việc lặp lại hoàn thành nhanh hơn nhiều lần so với làm tay, test bao phủ các đường đi chính, và quan trọng nhất là tôi vẫn hiểu và bảo vệ được toàn bộ thay đổi trong code review. Tôi rút ra một quy trình tái dùng: ngữ cảnh tốt (CLAUDE.md) + plan mode + diff nhỏ + hook tự động + review nghiêm."

Lời khuyên: nhấn mạnh bạn dùng AI để **khuếch đại** năng lực, không phải để **thay thế** trách nhiệm. Hãy nêu cụ thể cơ chế giữ chất lượng (plan mode, hook, review, test). Tránh kể kiểu "tôi để AI tự làm hết rồi nó chạy" — nhà tuyển dụng tìm người kiểm soát được công cụ, không phải người bị công cụ dẫn dắt.

</details>
## 2. Prompting và Brainstorming với AI

### 2.1. Một prompt hiệu quả cho công cụ agentic coding (như Claude Code) gồm những thành phần nào? Hãy phân biệt một prompt "yếu" và một prompt "mạnh".

<details>
<summary>👉 Xem câu trả lời</summary>

Một prompt mạnh thường có đủ bốn yếu tố: (1) mục tiêu rõ ràng — bạn muốn đạt được gì; (2) ngữ cảnh — file liên quan, thông báo lỗi, kiến trúc; (3) ràng buộc — quy ước code, thư viện được phép dùng, điều cần tránh; (4) định dạng đầu ra mong muốn — ví dụ "chỉ sửa file này", "trả về diff", "không thêm dependency mới".

Khác biệt cốt lõi: prompt yếu mô tả triệu chứng một cách mơ hồ và để AI tự đoán; prompt mạnh thu hẹp không gian đoán bằng ngữ cảnh và ràng buộc cụ thể.

| Prompt yếu | Prompt mạnh |
|---|---|
| "Sửa cái form login đi" | "Trong `src/features/auth/LoginForm.tsx`, nút submit không bị disable khi đang gọi API nên user bấm nhiều lần. Hãy disable nút khi `isPending` của mutation đang chạy. Dùng React Query đã có sẵn, không thêm state thủ công." |
| "Code này chậm, tối ưu giúp" | "Hàm `buildReport()` trong `report.service.ts` chạy ~4s với 10k bản ghi. Tôi nghi do query N+1 trong vòng lặp. Hãy phân tích và đề xuất cách gom query, giữ nguyên chữ ký hàm." |

```text
# Khung prompt mạnh có thể tái dùng
Mục tiêu: <kết quả mong muốn>
Ngữ cảnh: <file / lỗi / hành vi hiện tại>
Ràng buộc: <quy ước, thư viện, điều cấm>
Đầu ra: <diff / chỉ file X / kèm test>
```

Liên hệ Claude Code: vì đây là công cụ agentic (tự đọc file, chạy lệnh, tìm code), bạn không cần dán toàn bộ file vào prompt — chỉ cần chỉ đúng đường dẫn và mô tả mục tiêu, agent sẽ tự đọc. Nhưng bạn vẫn phải nêu rõ ràng buộc, vì agent sẽ chủ động hành động theo những gì bạn ngầm cho phép.

Bẫy thường gặp: prompt yếu khiến AI tự "sáng tác" yêu cầu — thêm dependency, đổi kiến trúc, sửa file ngoài phạm vi — vì bạn không nói rõ giới hạn.

</details>

---

### 2.2. Cung cấp ngữ cảnh tốt nghĩa là gì? Trong Claude Code, có những cách nào để đưa ngữ cảnh vào ngoài việc gõ thẳng vào prompt?

<details>
<summary>👉 Xem câu trả lời</summary>

Ngữ cảnh tốt = đủ thông tin để AI hiểu vấn đề mà không phải đoán, nhưng không nhiễu (không dán cả nghìn dòng không liên quan). Ba loại ngữ cảnh quan trọng nhất: file/đoạn code liên quan, thông báo lỗi nguyên văn (stack trace, log), và mục tiêu/kết quả mong muốn.

Trong Claude Code có nhiều kênh đưa ngữ cảnh, không chỉ qua prompt:

- Chỉ đường dẫn file để agent tự đọc (vì là agentic coding, nó tự `read`/`grep`).
- `CLAUDE.md`: file ghi quy ước dự án, lệnh hay dùng, kiến trúc — được tự nạp vào context mỗi phiên. Đây là nơi đặt ngữ cảnh BỀN VỮNG, lặp lại nhiều lần.
- Memory: ghi nhớ sự kiện/sở thích qua nhiều phiên (ví dụ "dự án dùng pnpm, Postgres ở cổng 5433").
- Dán nguyên văn lỗi/log vào prompt cho vấn đề tức thời.
- MCP server: kết nối nguồn dữ liệu ngoài (GitHub, database, Linear...) để agent lấy ngữ cảnh trực tiếp từ hệ thống thật.

```text
# Ngữ cảnh nghèo (AI phải đoán)
"Test bị fail, fix giúp"

# Ngữ cảnh tốt (dán lỗi nguyên văn + chỉ chỗ)
"Chạy `pnpm test auth.spec.ts` báo:
  ● LoginForm › disables button when pending
    expect(received).toBeDisabled()
    Received element is not disabled
File test: src/features/auth/__tests__/auth.spec.ts:42
Component: src/features/auth/LoginForm.tsx
Mong muốn: nút phải disabled khi mutation đang chạy."
```

Nguyên tắc phân tầng: thông tin dùng MỘT LẦN → đưa vào prompt; thông tin lặp lại MỌI PHIÊN (quy ước, kiến trúc) → đưa vào `CLAUDE.md`; sở thích bền vững qua nhiều phiên → Memory.

Bẫy thường gặp: dán quá nhiều ("context dump") khiến tín hiệu bị loãng và AI tập trung sai chỗ; hoặc ngược lại, mô tả lỗi bằng lời thay vì dán stack trace nguyên văn, làm mất manh mối quan trọng.

</details>

---

### 2.3. Vì sao nên chia nhỏ một vấn đề lớn khi làm việc với AI thay vì yêu cầu "làm hết một lượt"? Minh hoạ bằng một tính năng cụ thể.

<details>
<summary>👉 Xem câu trả lời</summary>

Yêu cầu một việc lớn nguyên khối ("xây cả hệ thống thanh toán") thường cho kết quả tệ: AI phải tự đưa ra hàng loạt quyết định ngầm, khó kiểm tra, và nếu sai một chỗ thì cả khối phải làm lại. Chia nhỏ giúp: mỗi bước có phạm vi rõ, dễ review, dễ sửa, và bạn giữ được quyền kiểm soát hướng đi.

Ví dụ tính năng "đăng nhập bằng email + xác thực 2 lớp (2FA)" có thể chia thành các bước tuần tự:

```text
1. Thiết kế schema DB cho user + bảng lưu mã OTP/secret 2FA.
2. Endpoint đăng nhập bước 1 (email + password) -> trả token tạm.
3. Sinh và gửi mã OTP (hoặc tích hợp TOTP).
4. Endpoint xác thực OTP -> đổi token tạm lấy token thật.
5. Viết test cho luồng thành công + các luồng lỗi (mã sai, hết hạn).
6. Xử lý rate limit chống brute-force.
```

Mỗi bước là một prompt riêng, review xong mới sang bước sau.

Liên hệ Claude Code: với việc lớn nên dùng Plan mode để AI lập kế hoạch (chính là việc chia nhỏ) và trình bày trước khi sửa code; bạn duyệt kế hoạch rồi mới cho thực thi. Với việc rất lớn cần chạy nhiều bước tất định, có thể dùng Workflows (điều phối nhiều agent bằng script). Còn các nhánh điều tra song song, độc lập có thể giao cho Subagents để cô lập context.

Lợi ích phụ về context: mỗi bước nhỏ giữ context window gọn, AI ít bị "trôi" mục tiêu hơn so với một cuộc hội thoại khổng lồ ôm cả chục yêu cầu.

Bẫy thường gặp: chia nhỏ nhưng các mảnh phụ thuộc nhau chéo chằng chịt; nên xác định ranh giới (interface giữa các bước) trước, để mỗi bước thực sự độc lập kiểm thử được.

</details>

---

### 2.4. Hãy mô tả vòng lặp "lập và tinh chỉnh prompt" (iterate). Khi kết quả đầu tiên chưa đúng, bạn làm gì để hội thoại tiếp theo tốt hơn thay vì chỉ lặp lại?

<details>
<summary>👉 Xem câu trả lời</summary>

Prompting hiếm khi đúng ngay lần đầu; nó là vòng lặp: ra prompt → đọc kết quả → chẩn đoán chỗ lệch → tinh chỉnh ngữ cảnh/ràng buộc → ra lại. Điều quan trọng là mỗi vòng phải THÊM thông tin hoặc THU HẸP yêu cầu, chứ không phải gõ lại y nguyên và mong kết quả khác.

Khi kết quả chưa đúng, có vài chiến thuật hiệu quả:

- Chỉ ra cụ thể chỗ sai và VÌ SAO sai, thay vì "vẫn chưa đúng": "Cách này tạo thêm `useState`, nhưng tôi muốn dùng state của React Query đã có."
- Bổ sung ràng buộc bị thiếu mà bạn vừa nhận ra: "Đừng đổi chữ ký public của hàm."
- Đưa phản ví dụ: "Khi input rỗng nó vẫn gọi API — hãy chặn trường hợp đó."
- Khi hội thoại đã đi quá xa hoặc lạc hướng, đôi khi tốt hơn là bắt đầu lại với một prompt gọn, đầy đủ ngữ cảnh đã học được (trong Claude Code có thể `/clear` để dọn context cũ rồi mở lại sạch sẽ).

```text
# Vòng 1
"Viết hàm debounce cho ô search."

# Kết quả: hàm chạy nhưng không huỷ timer khi unmount, và không generic.

# Vòng 2 (tinh chỉnh — thêm ràng buộc + chỉ đúng chỗ thiếu)
"Tốt, nhưng: (1) hãy làm thành custom hook useDebouncedValue<T> generic;
 (2) phải cleanup timer trong useEffect khi giá trị đổi hoặc component unmount;
 (3) thêm tham số delay mặc định 300ms. Giữ nguyên cách dùng phía gọi càng đơn giản càng tốt."
```

Mẹo nâng cao: nếu cùng một loại tinh chỉnh lặp lại nhiều lần ("luôn dùng React Query, luôn cleanup effect"), hãy đưa nó vào `CLAUDE.md` để không phải nhắc lại mỗi phiên.

Bẫy thường gặp: "lái mù" — cứ nhấn lại mà không nói rõ sai ở đâu; hoặc thêm quá nhiều yêu cầu mới cùng lúc khiến AI sửa được cái này lại hỏng cái kia. Nên tinh chỉnh từng khía cạnh một khi vấn đề khó.

</details>

---

### 2.5. Few-shot prompting là gì và khi nào nó đặc biệt hữu ích trong công việc lập trình hằng ngày?

<details>
<summary>👉 Xem câu trả lời</summary>

Few-shot prompting là đưa cho AI MỘT VÀI ví dụ mẫu (cặp input → output mong muốn) ngay trong prompt, để nó suy ra khuôn mẫu và áp dụng cho trường hợp mới. Ngược lại với zero-shot (không ví dụ, chỉ mô tả). Ví dụ giúp AI khớp đúng PHONG CÁCH và ĐỊNH DẠNG mà lời mô tả khó diễn đạt hết.

Đặc biệt hữu ích khi: bạn cần đầu ra theo một quy ước riêng (đặt tên, cấu trúc), khi chuyển đổi dữ liệu lặp lại, hoặc khi mô tả bằng lời thì dài dòng mà một ví dụ nói thay được.

```ts
// Few-shot: dạy AI cách viết test theo đúng phong cách của team
"Hãy viết test cho hàm `formatCurrency` theo đúng phong cách ví dụ sau:

Ví dụ — test cho formatDate:
describe('formatDate', () => {
  it('định dạng ngày hợp lệ', () => {
    expect(formatDate('2024-01-05')).toBe('05/01/2024');
  });
  it('trả về chuỗi rỗng khi input không hợp lệ', () => {
    expect(formatDate('xxx')).toBe('');
  });
});

Bây giờ viết test tương tự cho formatCurrency(amount, currency)."
```

Trong agentic coding, một dạng few-shot tự nhiên là chỉ AI tham chiếu một file đã có làm mẫu: "Tạo `UserService` theo đúng cấu trúc của `ProductService.ts`." Agent tự đọc file mẫu rồi mô phỏng quy ước — vừa là few-shot vừa tận dụng ngữ cảnh sẵn có trong repo.

Bẫy thường gặp: ví dụ mâu thuẫn nhau hoặc ví dụ chất lượng kém — AI sẽ học theo cả lỗi trong ví dụ. Ví dụ phải đại diện đúng cho thứ bạn muốn.

</details>

---

### 2.6. Chain-of-thought (yêu cầu AI suy luận từng bước) là gì? Khi nào nên dùng, và khi nào nó không phù hợp?

<details>
<summary>👉 Xem câu trả lời</summary>

Chain-of-thought (CoT) là kỹ thuật yêu cầu AI trình bày các bước suy luận trước khi đưa ra kết luận, thay vì "nhảy" thẳng tới đáp án. Với các bài toán nhiều bước (debug logic phức tạp, phân tích kiến trúc, lần theo dữ liệu chạy qua nhiều lớp), việc suy luận tường minh thường cho kết quả chính xác hơn vì AI không bỏ qua bước trung gian.

Khi nên dùng:
- Debug một bug khó, cần lần theo luồng dữ liệu/trạng thái.
- So sánh nhiều phương án có nhiều tiêu chí đánh đổi.
- Bài toán có ràng buộc logic (thuật toán, điều kiện biên).

```text
# Prompt khuyến khích suy luận từng bước trước khi sửa
"Component này re-render vô hạn. Trước khi sửa, hãy:
 1. Liệt kê mọi nguồn có thể gây re-render (state, props, effect deps).
 2. Lần theo: effect nào set state nào, và vì sao deps khiến nó chạy lại.
 3. Chỉ ra nguyên nhân gốc.
 Sau đó mới đề xuất bản sửa tối thiểu."
```

Liên hệ Claude Code: Plan mode mang tinh thần CoT ở cấp quy trình — AI lập kế hoạch và lý giải trước khi đụng vào code, cho bạn cơ hội bắt sai sót ở khâu suy luận thay vì sau khi đã sửa. Ngoài ra, bản thân model Claude cũng có khả năng suy luận sâu hơn cho các bài khó, nên việc yêu cầu trình bày từng bước thường khai thác tốt năng lực này.

Khi KHÔNG phù hợp: tác vụ đơn giản, một bước (đổi tên biến, format) — bắt suy luận dài chỉ tốn thời gian và token, làm chậm phản hồi. Với việc nhỏ, đi thẳng vào kết quả tốt hơn.

Bẫy thường gặp: nhầm "trình bày dài" với "suy luận đúng" — CoT làm lộ ra lập luận để BẠN kiểm tra, chứ bản thân nó không bảo đảm đúng; vẫn phải đọc và phản biện các bước.

</details>

---

### 2.7. Brainstorming kỹ thuật cùng AI khác gì với việc nhờ AI viết code? Bạn dùng AI thế nào để đề xuất nhiều phương án và so sánh trade-off?

<details>
<summary>👉 Xem câu trả lời</summary>

Brainstorming là dùng AI ở giai đoạn KHÁM PHÁ — khi chưa biết nên đi hướng nào — để mở rộng không gian phương án và phơi bày đánh đổi. Khác với prompt viết code (đã chốt hướng, cần triển khai), brainstorming cố tình giữ câu hỏi mở và yêu cầu nhiều lựa chọn.

Cách làm hiệu quả: yêu cầu nhiều phương án kèm bảng so sánh theo các tiêu chí bạn quan tâm (độ phức tạp, hiệu năng, chi phí vận hành, khả năng mở rộng), và yêu cầu nêu rõ giả định.

```text
"Tôi cần đồng bộ trạng thái real-time cho bảng Kanban nhiều người dùng.
Hãy đề xuất 3-4 phương án (vd WebSocket tự quản, dịch vụ pub/sub managed,
polling, CRDT...). Với mỗi phương án, lập bảng so sánh theo:
- Độ phức tạp triển khai
- Độ trễ
- Chi phí hạ tầng
- Xử lý conflict khi sửa đồng thời
Nêu rõ giả định, và khuyến nghị một phương án cho team 3 người, quy mô vừa."
```

Kết quả nên là một bảng trade-off:

| Phương án | Độ phức tạp | Độ trễ | Xử lý conflict |
|---|---|---|---|
| Polling định kỳ | Thấp | Cao | Đơn giản, dễ ghi đè |
| WebSocket tự quản | Trung bình | Thấp | Tự xử lý, dễ sai |
| Dịch vụ pub/sub managed | Thấp-TB | Thấp | Phụ thuộc dịch vụ |
| CRDT | Cao | Thấp | Tự hội tụ, học khó |

Liên hệ Claude Code: giai đoạn brainstorm hợp với Plan mode (chưa sửa code, chỉ bàn phương án), và có thể giao cho Subagent kiểu Explore/Plan để fan-out khảo sát codebase hiện tại trước khi đề xuất, mà không làm rối context chính.

Bẫy thường gặp: chốt phương án đầu tiên AI đưa ra mà không ép nó liệt kê lựa chọn khác — bạn mất đi giá trị lớn nhất của brainstorming là thấy được các đánh đổi. Luôn yêu cầu "ít nhất N phương án" và "nêu nhược điểm của lựa chọn bạn khuyến nghị".

</details>

---

### 2.8. Vì sao nên để AI đóng vai "người phản biện" (devil's advocate / red-team) cho thiết kế của bạn? Cho ví dụ cách ra prompt.

<details>
<summary>👉 Xem câu trả lời</summary>

AI mặc định có xu hướng "đồng thuận" và xác nhận ý bạn. Để khai thác đúng giá trị, hãy chủ động giao cho nó vai phản biện: tìm lỗ hổng, trường hợp biên, giả định sai, rủi ro vận hành trong THIẾT KẾ của bạn. Điều này biến AI thành một người review thiết kế giúp bạn thấy điểm mù trước khi viết code.

```text
"Đây là thiết kế của tôi cho hệ thống upload file:
 client gọi API lấy presigned URL -> upload thẳng lên S3 ->
 gọi API confirm để lưu metadata.

Hãy đóng vai một senior khó tính review thiết kế này. Đừng khen.
Tập trung tìm:
- Trường hợp biên và lỗi (client upload xong nhưng không gọi confirm thì sao?)
- Rủi ro bảo mật (presigned URL bị lạm dụng, upload file quá lớn?)
- Vấn đề vận hành (file mồ côi, dọn rác, retry)
Liệt kê từng vấn đề kèm mức độ nghiêm trọng và đề xuất giảm thiểu."
```

Một biến thể mạnh là yêu cầu AI tự sinh ra các kịch bản thất bại: "Liệt kê 5 cách thiết kế này có thể sập trong production." Hoặc dùng hai vai: một vai bảo vệ thiết kế, một vai tấn công nó, rồi bạn làm trọng tài.

Liên hệ Claude Code: tinh thần này được "đóng gói" thành các Skill chuyên biệt như code-review hay security-review — gọi qua slash command để chạy đúng một quy trình phản biện/rà soát có hệ thống, thay vì bạn phải tự soạn prompt phản biện mỗi lần.

Bẫy thường gặp: phản biện chung chung ("nên cẩn thận bảo mật"). Để hữu ích, hãy ép nó cụ thể: chỉ ra đúng dòng/đúng kịch bản, kèm mức nghiêm trọng và cách khắc phục. Và nhớ: AI phản biện cũng có thể sai hoặc thổi phồng — bạn vẫn là người quyết định.

</details>

---

### 2.9. Khi AI hiểu sai ý bạn (đi nhầm hướng, sửa nhầm file, "ảo giác" ra một API không tồn tại), bạn xử lý thế nào để kéo nó về đúng hướng?

<details>
<summary>👉 Xem câu trả lời</summary>

Trước tiên phải nhận ra vì sao AI hiểu sai — thường do prompt thiếu ràng buộc, ngữ cảnh mơ hồ, hoặc context đã bị "trôi" sau hội thoại quá dài. Cách xử lý theo mức độ:

- Hiểu lệch nhẹ: nêu chính xác chỗ sai và bổ sung ràng buộc bị thiếu, không cần làm lại từ đầu. "Bạn sửa đúng logic nhưng nhầm file — phải sửa `v2/handler.ts` chứ không phải `v1/handler.ts`."
- AI "ảo giác" API/hàm không tồn tại: yêu cầu nó kiểm chứng bằng nguồn thật thay vì trí nhớ — "Hãy grep trong repo / đọc file định nghĩa để xác nhận hàm này tồn tại trước khi dùng." Trong agentic coding, lợi thế là AI có thể tự đọc code và chạy lệnh để TỰ kiểm chứng, nên hãy hướng nó làm vậy.
- Đi lạc xa, sửa lung tung: dừng lại, hoàn tác (revert) thay đổi, dọn context (`/clear`) và mở lại với một prompt gọn, đầy đủ ràng buộc đã rút ra.

```text
# Khi AI bịa API
"`queryClient.invalidateAll()` không tồn tại trong React Query.
Trước khi viết tiếp, hãy mở tài liệu / file types của @tanstack/react-query
và dùng đúng API có thật (vd invalidateQueries). Đừng đoán tên hàm."

# Khi AI sửa ngoài phạm vi
"Hãy hoàn nguyên mọi thay đổi ngoài file LoginForm.tsx.
Chỉ được sửa đúng file đó. Liệt kê lại các file bạn vừa đụng vào."
```

Phòng bệnh hơn chữa bệnh:
- Dùng Plan mode để duyệt hướng đi TRƯỚC khi AI sửa code — bắt hiểu sai sớm.
- Đưa quy ước/ranh giới vào `CLAUDE.md` để không phải nhắc lại.
- Để permission mode mặc định (hỏi trước khi hành động) cho việc rủi ro, thay vì bật bypass.
- Với hành vi "tự động mỗi lần" (vd luôn chạy lint sau khi sửa), dùng Hook chứ đừng trông cậy AI tự nhớ — harness chạy hook một cách tất định.

Bẫy thường gặp: thấy AI lạc hướng nhưng vẫn cố "vá" trong cùng cuộc hội thoại đã rối; nhiều khi `/clear` rồi bắt đầu lại với ngữ cảnh sạch nhanh và sạch hơn nhiều. Và luôn review diff — không merge code mình chưa hiểu.

</details>

---

### 2.10. Prompt để VIẾT CODE và prompt để bàn THIẾT KẾ / Ý TƯỞNG khác nhau ra sao? Bạn điều chỉnh cách viết prompt thế nào cho mỗi loại?

<details>
<summary>👉 Xem câu trả lời</summary>

Hai loại có MỤC TIÊU và TIÊU CHÍ "tốt" khác nhau, nên cách viết prompt cũng khác:

| Tiêu chí | Prompt viết code | Prompt thiết kế / ý tưởng |
|---|---|---|
| Mục tiêu | Sản phẩm chạy được, đúng spec | Khám phá phương án, thấy trade-off |
| Mức cụ thể | Rất cụ thể: file, hàm, ràng buộc kỹ thuật | Mở hơn: bối cảnh, mục tiêu, ràng buộc nghiệp vụ |
| Đầu ra mong muốn | Code / diff / test | Danh sách phương án, bảng so sánh, khuyến nghị |
| Tính hội tụ | Hội tụ về MỘT lời giải | Phân kỳ ra NHIỀU lựa chọn rồi mới chốt |
| Cách đánh giá | Chạy được, qua test, đúng quy ước | Lập luận vững, đánh đổi rõ, hợp ràng buộc |

```text
# Prompt VIẾT CODE: cụ thể, đóng, có ràng buộc kỹ thuật
"Trong `cart.service.ts`, thêm hàm `applyCoupon(cartId, code)`:
 - Validate coupon còn hạn và chưa dùng (bảng coupons).
 - Trả về tổng tiền sau giảm.
 - Ném `BadRequestException` nếu coupon không hợp lệ.
 - Viết kèm unit test. Không đổi chữ ký các hàm khác."

# Prompt THIẾT KẾ: mở, đặt bối cảnh + ràng buộc nghiệp vụ, xin nhiều phương án
"Chúng tôi muốn thêm hệ thống coupon. Quy mô ~50k đơn/tháng, team nhỏ.
 Hãy đề xuất các cách mô hình hoá coupon (phần trăm, cố định, theo sản phẩm,
 dùng-một-lần...) và đánh đổi giữa linh hoạt và độ phức tạp.
 Chưa cần code, hãy bàn mô hình dữ liệu và rủi ro trước."
```

Quy tắc thực dụng: thiết kế đi TRƯỚC, code đi SAU. Bắt đầu mở (brainstorm/Plan mode) để chốt hướng, rồi chuyển sang prompt viết code thật cụ thể để triển khai. Trong Claude Code, đây đúng là luồng Plan mode (bàn và duyệt kế hoạch) rồi mới chuyển sang chế độ sửa code.

Bẫy thường gặp: viết prompt thiết kế quá cụ thể (đã vô tình áp đặt một lời giải) làm AI không khám phá phương án khác; hoặc ngược lại, viết prompt code quá mở khiến AI phải tự đoán hàng loạt quyết định kỹ thuật và đi chệch.

</details>
## 3. Claude Code Tổng quan

### 3.1. Claude Code là gì? Nó khác gì với một chatbot lập trình thông thường?

<details>
<summary>👉 Xem câu trả lời</summary>

Claude Code là công cụ "agentic coding" của Anthropic, chạy trên các model Claude (Opus/Sonnet/Haiku/Fable). Điểm khác biệt cốt lõi so với một chatbot thông thường (kiểu hỏi-đáp, dán code vào ô chat) là Claude Code hoạt động như một **agent**: nó không chỉ trả lời bằng văn bản, mà còn tự chủ động **hành động** trong môi trường thật của bạn — đọc/sửa file, chạy lệnh shell, tìm kiếm trong codebase, gọi các tool.

So sánh nhanh:

| Tiêu chí | Chatbot lập trình thường | Claude Code (agentic) |
|---|---|---|
| Tương tác | Bạn dán code, nó trả về text | Nó tự đọc/sửa file trong dự án |
| Hành động | Không thực thi gì | Chạy lệnh, sửa file, gọi tool |
| Vòng lặp | Một lượt hỏi-đáp | Vòng lặp agentic nhiều bước tới khi xong việc |
| Ngữ cảnh | Chỉ những gì bạn dán vào | Tự khám phá codebase, đọc `CLAUDE.md` |

Ví dụ thực tế: thay vì hỏi "viết giúp tôi hàm validate email" rồi tự copy-paste, bạn nói "thêm validation cho field email trong `auth.service.ts` và viết test". Claude Code sẽ tự tìm file, đọc context xung quanh, sửa code, có thể chạy test để kiểm chứng — tất cả trong một phiên làm việc.

Điểm cần nắm khi phỏng vấn: nhấn mạnh từ khóa "agentic" — tự chủ, có vòng lặp hành động, làm việc trực tiếp trên codebase, chứ không chỉ sinh text.

</details>

---

### 3.2. Claude Code có những dạng (form factor) nào? Khi nào nên dùng dạng nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Claude Code không chỉ là một công cụ CLI; Anthropic cung cấp nhiều dạng để phù hợp với nhiều quy trình làm việc khác nhau:

- **CLI trong terminal**: dạng phổ biến nhất, chạy ngay trong terminal. Phù hợp với người quen làm việc trên dòng lệnh, dễ tích hợp vào script và CI.
- **Extension cho IDE**: có cho **VS Code** và **JetBrains**. Tích hợp ngay trong editor, tiện cho việc xem diff, theo dõi thay đổi file trực quan.
- **App desktop**: bản ứng dụng riêng cho **Mac và Windows**.
- **Web**: truy cập qua **claude.ai/code**, không cần cài đặt cục bộ.

Bảng gợi ý chọn dạng:

| Tình huống | Dạng nên dùng |
|---|---|
| Làm việc nhiều với dòng lệnh, script, server | CLI terminal |
| Muốn xem diff/thay đổi ngay trong editor | Extension VS Code / JetBrains |
| Muốn trải nghiệm app riêng, gọn gàng | App desktop (Mac/Windows) |
| Không muốn cài đặt, dùng nhanh qua trình duyệt | Web (claude.ai/code) |

Lưu ý: dù dạng nào thì bản chất agentic vẫn giống nhau — vẫn là vòng lặp đọc/sửa file, chạy lệnh, gọi tool. Khác biệt chủ yếu là giao diện và cách tích hợp vào quy trình của bạn.

</details>

---

### 3.3. "Vòng lặp agentic" (agentic loop) của Claude Code hoạt động như thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Vòng lặp agentic là cách Claude Code tự chủ hoàn thành một nhiệm vụ qua nhiều bước, thay vì chỉ trả lời một lần. Mỗi vòng, model sẽ quyết định hành động tiếp theo dựa trên kết quả của hành động trước:

1. Đọc yêu cầu và ngữ cảnh hiện có (bao gồm `CLAUDE.md`, file đã đọc...).
2. Quyết định một hành động: đọc file, sửa file, chạy lệnh shell, tìm kiếm code, hoặc gọi một tool.
3. Thực thi hành động đó (xin phép nếu cần theo permission mode).
4. Nhận lại kết quả (nội dung file, output của lệnh, kết quả tìm kiếm).
5. Đánh giá: đã xong chưa? Nếu chưa, quay lại bước 2 với ngữ cảnh đã cập nhật.
6. Lặp lại tới khi hoàn thành nhiệm vụ.

```text
[Yêu cầu] -> đọc context -> chọn hành động -> thực thi (tool) -> nhận kết quả
                  ^                                                   |
                  |__________________ chưa xong ______________________|
                                                                      |
                                                                  [Hoàn thành]
```

Ví dụ cụ thể với yêu cầu "sửa bug ở endpoint login":
- Tìm kiếm trong codebase chỗ xử lý login (grep/search).
- Đọc file controller và service liên quan.
- Phát hiện lỗi, sửa file.
- Chạy lệnh test để kiểm chứng.
- Nếu test fail, đọc lỗi, sửa tiếp — vòng lặp tiếp tục.

Điểm quan trọng: chính nhờ vòng lặp này mà Claude Code "tự xoay sở" được những việc phức tạp, nhiều bước. Đây là khác biệt cốt lõi so với một lệnh hỏi-đáp đơn lẻ.

</details>

---

### 3.4. Permission mode là gì? Có những chế độ nào và dùng khi nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Vì Claude Code tự thực thi hành động (sửa file, chạy lệnh), nó cần một cơ chế kiểm soát để bạn không vô tình cho nó làm điều nguy hiểm. Đó là **permission mode** — quy định khi nào agent cần **xin phép** trước khi hành động.

Các chế độ:

- **Mặc định (hỏi trước)**: trước mỗi hành động có tác động (sửa file, chạy lệnh), Claude Code dừng lại và hỏi bạn duyệt. An toàn nhất, phù hợp khi mới làm quen hoặc làm trên code quan trọng.
- **`acceptEdits`**: tự động chấp nhận các thao tác sửa file, không hỏi từng lần. Tăng tốc khi bạn tin tưởng hướng đi của agent.
- **`plan`**: xem mục câu hỏi về plan mode bên dưới — agent chỉ lập kế hoạch, không thực sự sửa code.
- **`bypassPermissions`**: bỏ qua việc hỏi phép, agent hành động tự do. Mạnh nhưng rủi ro — chỉ dùng trong môi trường cô lập, được kiểm soát (ví dụ sandbox).

Bảng so sánh nhanh:

| Chế độ | Hành vi | Khi nào dùng |
|---|---|---|
| Mặc định | Hỏi trước mỗi hành động | Code quan trọng, cần kiểm soát chặt |
| `acceptEdits` | Tự duyệt sửa file | Tin tưởng, muốn nhanh hơn |
| `plan` | Chỉ lập kế hoạch, không sửa | Muốn xem chiến lược trước khi làm |
| `bypassPermissions` | Không hỏi phép | Sandbox/CI cô lập, làm tự động |

Bẫy thường gặp: bật `bypassPermissions` trên máy thật với codebase quan trọng. Quyền tự do hành động đi kèm rủi ro mất mát dữ liệu nếu agent chạy nhầm lệnh. Nguyên tắc: chế độ càng "thoáng" thì môi trường càng phải cô lập.

</details>

---

### 3.5. Plan mode là gì? Tại sao nó hữu ích trước khi để agent sửa code?

<details>
<summary>👉 Xem câu trả lời</summary>

**Plan mode** là chế độ trong đó Claude Code **lập kế hoạch và trình bày** những gì nó định làm, **trước khi** thực sự sửa code. Nói cách khác, agent khám phá codebase, suy nghĩ về cách tiếp cận, rồi trình ra một bản kế hoạch để bạn duyệt — chứ chưa đụng tới file nào.

Lợi ích:

- **Kiểm soát hướng đi**: với việc lớn hoặc phức tạp, bạn xem agent định làm gì trước, tránh việc nó "lao đầu" sửa hàng loạt file theo hướng sai.
- **Phát hiện hiểu nhầm sớm**: nếu agent hiểu sai yêu cầu, bạn nhận ra ngay ở bước kế hoạch thay vì sau khi đã sửa nhầm.
- **An toàn**: không có thay đổi nào lên codebase cho tới khi bạn đồng ý.

Quy trình điển hình:

```text
1. Bật plan mode
2. Giao nhiệm vụ: "Refactor module thanh toán để tách logic tính phí"
3. Agent đọc code, khảo sát, trình bày kế hoạch:
   - Tạo file fee-calculator.service.ts
   - Di chuyển hàm calculateFee ra khỏi payment.service.ts
   - Cập nhật các chỗ gọi
   - Viết test cho service mới
4. Bạn xem -> chỉnh sửa kế hoạch nếu cần -> phê duyệt
5. Agent mới bắt đầu thực thi
```

Khi nào dùng: việc lớn, đụng nhiều file, hoặc việc mà sai một bước sẽ khó sửa. Với việc nhỏ và rõ ràng (sửa một dòng), plan mode có thể là thừa.

Liên hệ: plan mode cũng là một trong các permission mode — nó kiểm soát hành vi bằng cách "khóa" giai đoạn sửa code lại cho tới khi bạn duyệt kế hoạch.

</details>

---

### 3.6. So sánh Claude Code (agentic) với autocomplete kiểu Copilot (gợi ý từng dòng). Khác nhau cơ bản ở đâu?

<details>
<summary>👉 Xem câu trả lời</summary>

Đây là một câu hỏi rất hay để thể hiện bạn hiểu bản chất "agentic". Hai cách tiếp cận giải quyết hai bài toán khác nhau:

| Tiêu chí | Autocomplete (kiểu Copilot) | Agentic coding (Claude Code) |
|---|---|---|
| Đơn vị làm việc | Gợi ý từng dòng / từng đoạn khi bạn gõ | Hoàn thành cả một nhiệm vụ nhiều bước |
| Ai điều khiển | Bạn gõ, nó gợi ý phần tiếp theo | Bạn giao việc, nó tự lên kế hoạch và hành động |
| Phạm vi | Chủ yếu file/dòng đang gõ | Cả codebase: tìm, đọc, sửa nhiều file |
| Hành động | Chỉ chèn text gợi ý | Chạy lệnh, sửa file, gọi tool, chạy test |
| Vòng lặp | Một gợi ý cho một vị trí con trỏ | Vòng lặp agentic tới khi xong việc |

Bản chất:
- **Autocomplete** là "trợ lý gõ phím" — tăng tốc lúc bạn đang viết code, nhưng *bạn* vẫn là người dẫn dắt từng bước.
- **Agentic** là "đồng nghiệp" nhận một nhiệm vụ — ví dụ "thêm pagination cho API danh sách user" — rồi tự khám phá, sửa nhiều chỗ, kiểm chứng.

Ví dụ minh họa sự khác biệt:
- Copilot: bạn gõ `function getUserById(` và nó gợi ý phần thân hàm.
- Claude Code: bạn nói "thêm endpoint lấy user theo id, có xử lý lỗi 404 và viết test", nó tự tạo route, controller, service, test, và có thể chạy test để xác nhận.

Lưu ý khi trả lời: hai cái không loại trừ nhau — nhiều người dùng cả hai. Autocomplete cho lúc gõ chi tiết, agentic cho việc lớn cần tự chủ. Điều phỏng vấn muốn nghe là bạn hiểu *khi nào* dùng cái nào.

</details>

---

### 3.7. Slash command là gì trong Claude Code? Cho ví dụ vài lệnh built-in và ý nghĩa.

<details>
<summary>👉 Xem câu trả lời</summary>

**Slash command** là các lệnh có dạng `/tên` mà bạn gõ trực tiếp trong Claude Code để kích hoạt một hành động hoặc quy trình cụ thể. Có hai loại: lệnh **built-in** (có sẵn) và lệnh **tùy biến** (do bạn hoặc nhóm tự định nghĩa).

Một số lệnh built-in thường gặp:

| Lệnh | Ý nghĩa |
|---|---|
| `/init` | Khởi tạo file `CLAUDE.md` mô tả/ghi rule cho dự án |
| `/review` | Review một pull request / thay đổi |
| `/config` | Cấu hình (model, theme...) |
| `/clear` | Xóa ngữ cảnh phiên hiện tại để bắt đầu lại sạch |

Ví dụ luồng dùng thực tế:

```text
/init          -> Claude quét dự án, tạo CLAUDE.md ghi conventions, lệnh hay dùng, kiến trúc
/clear         -> bắt đầu một nhiệm vụ mới với context sạch, tránh nhiễu từ việc trước
/review        -> review thay đổi đang chờ trước khi commit
```

Slash command tùy biến: bạn có thể định nghĩa lệnh riêng cho các quy trình lặp đi lặp lại (ví dụ một lệnh để chạy quy trình review bảo mật theo chuẩn của nhóm). Đây cũng chính là cách Skills được gọi ra — một skill được kích hoạt qua slash command.

Bẫy thường gặp: quên `/clear` giữa các nhiệm vụ không liên quan, khiến context cũ làm "nhiễu" và agent có thể đưa ra quyết định dựa trên thông tin không còn phù hợp.

</details>

---

### 3.8. Việc chọn model trong Claude Code có ý nghĩa gì với hiệu quả công việc?

<details>
<summary>👉 Xem câu trả lời</summary>

Claude Code chạy trên các model Claude khác nhau (Opus, Sonnet, Haiku, Fable), mỗi model có sự đánh đổi giữa **tốc độ**, **chi phí** và **độ thông minh**. Việc chọn đúng model chính là một đòn bẩy về hiệu quả.

Tư duy đánh đổi:

| Ưu tiên | Hướng chọn |
|---|---|
| Tốc độ, chi phí thấp, việc đơn giản | Model nhẹ/nhanh hơn (kiểu Haiku) |
| Cân bằng tốc độ và chất lượng | Model tầm trung (kiểu Sonnet) |
| Việc khó, suy luận sâu, agentic dài hơi | Model mạnh nhất (Opus/Fable) |

Ý nghĩa thực tế với một "lập trình viên biết tận dụng AI":
- Việc lặp lại, đơn giản (đổi tên biến, sửa lỗi nhỏ) — ưu tiên nhanh và rẻ.
- Việc phức tạp (refactor lớn, debug khó, thiết kế kiến trúc) — chấp nhận chậm và tốn hơn để đổi lấy chất lượng và độ chính xác.

Bạn có thể đổi model qua `/config` hoặc cấu hình trong `settings.json`.

Lưu ý quan trọng để tránh "bịa": Claude Code có nhiều dòng model với tốc độ/chi phí khác nhau, và bạn chọn theo bài toán. Điều phỏng vấn muốn thấy là *tư duy đánh đổi* — không phải mặc định luôn dùng model mạnh nhất cho mọi việc (lãng phí), cũng không dùng model yếu cho việc khó (kết quả kém). Chọn model là một kỹ năng tối ưu chi phí/chất lượng có chủ đích.

Ví dụ minh họa tư duy chọn model:

```text
Đổi tên biến trong 1 file          -> model nhẹ/nhanh (Haiku)
Viết một service mới, có test       -> model cân bằng (Sonnet)
Refactor kiến trúc nhiều module     -> model mạnh nhất (Opus/Fable)
```

</details>

---

### 3.9. Hãy kể về một lần bạn dùng Claude Code (hoặc công cụ agentic) để xử lý một việc mà cách làm thủ công sẽ rất tốn thời gian.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Đây là câu hỏi hành vi (behavioral) — hãy trả lời theo cấu trúc STAR để thể hiện bạn thật sự coi AI là "siêu năng lực".

- **Situation (Bối cảnh)**: Mô tả ngắn gọn tình huống. Ví dụ: "Dự án backend NestJS của tôi cần đổi convention đặt tên cho toàn bộ DTO và cập nhật mọi chỗ import liên quan — rải rác trên hơn 40 file."

- **Task (Nhiệm vụ)**: Nêu rõ mục tiêu và ràng buộc. Ví dụ: "Tôi cần đổi tên nhất quán, cập nhật import, không làm hỏng test, và hoàn thành trong buổi chiều để kịp release."

- **Action (Hành động)**: Đây là phần quan trọng nhất — thể hiện cách bạn tận dụng agentic coding một cách *có chiến lược*, không chỉ "nhờ AI làm hộ":
  - Bật **plan mode** để Claude Code khảo sát codebase và trình bày kế hoạch đổi tên trước, tôi duyệt để chắc chắn nó không bỏ sót chỗ nào.
  - Để agent chạy **vòng lặp agentic**: tìm các chỗ dùng, sửa file, rồi tự chạy test để kiểm chứng.
  - Dùng **permission mode** hợp lý: ban đầu hỏi trước để xem agent làm đúng hướng, sau đó chuyển `acceptEdits` để tăng tốc khi đã tin tưởng.
  - Dùng `CLAUDE.md` để agent nắm convention dự án, đảm bảo code sinh ra đúng phong cách.

- **Result (Kết quả)**: Định lượng nếu được. Ví dụ: "Việc dự kiến mất nửa ngày làm tay được hoàn thành trong khoảng một giờ, toàn bộ test pass, và quan trọng là không sót chỗ import nào — điều mà làm thủ công rất dễ bỏ lỡ. Tôi vẫn review kỹ diff cuối cùng trước khi commit."

Mẹo gây ấn tượng: nhấn mạnh rằng bạn không "phó mặc" cho AI — bạn dùng plan mode để kiểm soát, dùng permission mode để cân bằng tốc độ và an toàn, và luôn review kết quả. Đó là dấu hiệu của một người dùng AI như công cụ tăng năng suất một cách *có trách nhiệm và có kỹ năng*.

</details>
## 4. Claude Code Skills và Slash Commands

### 4.1. Skill trong Claude Code là gì? Nó khác gì so với một slash command thông thường, và mối quan hệ giữa hai khái niệm này ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

Skill là một "kỹ năng" tái sử dụng được trong Claude Code: nó đóng gói một quy trình chuyên biệt (ví dụ code review, deep research, security review) dưới dạng tập hợp hướng dẫn cộng tài nguyên đi kèm. Một skill được định nghĩa trong file `SKILL.md` có phần frontmatter (`name`, `description`) và phần thân là nội dung hướng dẫn chi tiết.

Slash command là cơ chế *gọi* (invoke): bạn gõ `/tên` để kích hoạt một hành động. Mối quan hệ giữa hai khái niệm:

- Slash command là cánh cửa để gọi nhiều thứ: lệnh built-in (`/init`, `/review`, `/config`, `/clear`), lệnh tùy biến do bạn tự tạo, và cả skill.
- Skill được "phơi bày" ra như một slash command — khi cài một skill tên `deep-research`, bạn gọi nó bằng `/deep-research`.

Nói cách khác: không phải mọi slash command đều là skill, nhưng nhiều skill được gọi qua slash command. Skill nhấn mạnh phần *nội dung và quy trình* (cái gì sẽ chạy), còn slash command nhấn mạnh phần *kích hoạt* (gõ gì để chạy).

| Tiêu chí | Slash command | Skill |
|---|---|---|
| Bản chất | Cách gọi (`/tên`) | Quy trình đóng gói lại để tái dùng |
| Định nghĩa ở | Tùy loại; built-in có sẵn | File `SKILL.md` + frontmatter |
| Mục tiêu chính | Kích hoạt nhanh một hành động | Cung cấp hướng dẫn/quy trình chuyên biệt |
| Ví dụ | `/clear`, `/init` | `/code-review`, `/security-review` |

Liên hệ với người làm React/Next.js: bạn có thể đóng gói quy trình "review một PR frontend" (kiểm tra accessibility, re-render thừa, ranh giới Server/Client Component) thành một skill và gọi lại bằng một slash command thay vì gõ lại đoạn prompt dài mỗi lần.

Bẫy thường gặp: nghĩ rằng "skill" và "slash command" là hai thứ tách biệt hoàn toàn. Thực tế chúng giao nhau — slash command là *giao diện gọi*, skill là *thứ được gọi*.

</details>

---

### 4.2. File `SKILL.md` gồm những phần nào? Vai trò của frontmatter `name` và `description` là gì, và tại sao `description` lại quan trọng đến vậy?

<details>
<summary>👉 Xem câu trả lời</summary>

Một skill là một thư mục chứa file `SKILL.md`. File này gồm hai phần:

1. Phần frontmatter ở đầu file (định dạng giống YAML) với các trường bắt buộc: `name` (tên skill) và `description` (mô tả ngắn skill làm gì và khi nào nên dùng).
2. Phần thân (body): nội dung hướng dẫn đầy đủ — các bước cần làm, quy ước, tài nguyên tham chiếu.

```markdown
---
name: pr-review-frontend
description: Review một Pull Request frontend React/Next.js — kiểm tra
  re-render thừa, accessibility, ranh giới Server/Client Component, và
  quản lý state. Dùng khi người dùng muốn review code frontend trước khi merge.
---

# Quy trình review PR frontend

## Bước 1: Đọc diff
...

## Bước 2: Kiểm tra checklist
- Có dùng `useMemo`/`useCallback` đúng chỗ không?
- Component có "use client" thừa không?
...
```

Vai trò của `description`: đây là phần *luôn nằm trong context* theo cơ chế "progressive disclosure" (tiết lộ dần). Claude đọc `description` để quyết định *khi nào* skill này phù hợp với tác vụ; chỉ khi tác vụ thực sự cần, Claude mới đọc toàn bộ phần thân `SKILL.md`. Nhờ vậy context nền giữ được gọn, chi tiết được nạp theo nhu cầu.

Vì lý do đó `description` phải nói rõ cả "skill làm gì" lẫn "khi nào nên dùng". Một mô tả mơ hồ như "review code" khiến skill khó được kích hoạt đúng lúc; mô tả tốt nêu rõ ngữ cảnh kích hoạt (ví dụ "dùng khi review PR frontend trước khi merge").

Bẫy thường gặp:
- Viết `description` chỉ mô tả *cái gì* mà quên *khi nào* → skill không tự kích hoạt đúng thời điểm.
- Nhồi toàn bộ hướng dẫn dài vào `description` thay vì để trong body → làm phình context nền, mất ý nghĩa của progressive disclosure.

</details>

---

### 4.3. Skill built-in khác skill tùy biến (custom) ở điểm nào? Cho ví dụ và nói rõ khi nào nên tự viết skill.

<details>
<summary>👉 Xem câu trả lời</summary>

Có thể phân loại skill theo nguồn gốc:

- Skill built-in: do Anthropic cung cấp sẵn trong Claude Code, dùng được ngay mà không cần tự định nghĩa. Ví dụ: `code-review`, `deep-research`, `security-review`.
- Skill tùy biến (custom): do bạn tự viết bằng cách tạo file `SKILL.md`. Có thể đặt ở cấp user (`~/.claude`), cấp project (`.claude`), hoặc đến từ một plugin.

| Loại | Nguồn | Đặt ở đâu | Khi dùng |
|---|---|---|---|
| Built-in | Anthropic cung cấp | Có sẵn | Quy trình phổ biến (review, research, security) |
| Custom (user) | Bạn tự viết | `~/.claude` | Quy ước cá nhân, dùng xuyên nhiều dự án |
| Custom (project) | Bạn tự viết | `.claude` | Quy trình riêng của dự án, chia sẻ với team |
| Custom (plugin) | Đóng gói trong plugin | Theo plugin | Phân phối lại cho nhiều người/nhiều máy |

Khi nào nên tự viết skill: khi bạn thấy mình lặp đi lặp lại cùng một đoạn prompt dài hoặc cùng một quy trình nhiều bước. Ví dụ với người làm React/Next.js:

- Skill "scaffold một feature module": tạo folder, file component, file test, file barrel export theo đúng cấu trúc dự án.
- Skill "kiểm tra hiệu năng render": chạy checklist về `memo`, `useMemo`, key trong list, tránh re-render thừa.
- Skill "kiểm tra Server/Client boundary trong Next.js App Router": đảm bảo không lộ secret xuống client, `"use client"` chỉ đặt nơi cần.

Đóng gói thành skill có hai lợi ích so với việc dán prompt mỗi lần: nhất quán (mọi lần đều chạy đúng quy trình) và chia sẻ được (đặt ở `.claude` của project để cả team dùng chung).

Bẫy thường gặp: viết một skill quá rộng ("làm mọi thứ về frontend"). Skill nên có phạm vi rõ ràng, một mục tiêu — như một hàm thuần thay vì một "God object".

</details>

---

### 4.4. Skill khác với một prompt thông thường (gõ trực tiếp vào chat) ở những điểm nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Cùng là "hướng dẫn cho AI", nhưng skill và prompt thông thường khác nhau ở cách lưu trữ, tái sử dụng và nạp vào context:

| Tiêu chí | Prompt gõ trực tiếp | Skill |
|---|---|---|
| Tái sử dụng | Một lần; muốn lại phải gõ lại | Đóng gói, gọi lại bằng `/tên` bất kỳ lúc nào |
| Lưu trữ | Chỉ tồn tại trong cuộc hội thoại | Lưu thành file `SKILL.md` trên đĩa |
| Cách nạp context | Toàn bộ prompt vào context ngay | Progressive disclosure: chỉ `description` thường trú, body nạp khi cần |
| Kích hoạt | Thủ công, đọc và chạy ngay | Claude tự cân nhắc qua `description`, hoặc bạn gọi qua slash command |
| Chia sẻ | Khó (copy-paste cho nhau) | Đặt ở `.claude` của project hoặc đóng gói plugin |
| Nhất quán | Dễ sai khác mỗi lần gõ | Cố định, mọi lần chạy giống nhau |

Điểm mấu chốt là progressive disclosure và tái sử dụng. Prompt thông thường: nếu bạn dán một đoạn 300 dòng vào chat thì cả 300 dòng nằm trong context ngay lập tức, và lần sau muốn dùng lại bạn phải dán lại. Skill: chỉ phần `description` (vài dòng) thường trú để Claude biết "có công cụ này"; toàn bộ hướng dẫn chi tiết chỉ được đọc khi tác vụ thực sự liên quan.

Một điểm quan trọng cần phân biệt: skill (và prompt) là *hướng dẫn cho AI làm khi được gọi*. Nếu bạn muốn một hành vi *tự động xảy ra mỗi khi có sự kiện X* (ví dụ tự chạy ESLint/Prettier sau mỗi lần sửa file) thì đó KHÔNG phải việc của skill — đó là việc của hook (cấu hình trong `settings.json`), vì harness mới là cái tự chạy lệnh theo sự kiện, chứ không phải dựa vào việc AI "nhớ" gọi skill.

Bẫy thường gặp: kỳ vọng skill "tự động chạy" theo lịch hay theo sự kiện. Skill được kích hoạt khi tác vụ phù hợp (hoặc khi bạn gọi), không phải là cron hay trigger sự kiện.

</details>

---

### 4.5. Slash command built-in và slash command tùy biến khác nhau thế nào? Kể vài lệnh built-in hay dùng và công dụng của chúng.

<details>
<summary>👉 Xem câu trả lời</summary>

Slash command là lệnh dạng `/tên`. Có hai nhóm:

- Built-in: lệnh có sẵn trong Claude Code. Ví dụ:
  - `/init`: khởi tạo file `CLAUDE.md` ghi tài liệu/ngữ cảnh của codebase.
  - `/review`: review một Pull Request.
  - `/config`: xem và chỉnh cấu hình (ví dụ theme, model).
  - `/clear`: xóa context của cuộc hội thoại hiện tại để bắt đầu lại sạch sẽ.
- Tùy biến: lệnh bạn tự định nghĩa, hoặc đến từ skill/plugin. Khi cài một skill, nó thường xuất hiện như một slash command tùy biến để gọi.

```text
# Một số tình huống dùng lệnh built-in khi làm dự án Next.js

/init      -> sinh CLAUDE.md mô tả kiến trúc, lệnh build/test, quy ước code
/clear     -> dọn context khi chuyển sang tác vụ mới, tránh "nhiễu" ngữ cảnh cũ
/review    -> review PR trước khi merge
/config    -> đổi model dùng (vd dùng Opus cho việc khó, Haiku cho việc nhẹ)
```

Phân biệt cốt lõi: built-in là "pin sẵn trong hộp", tùy biến là "bạn tự lắp thêm". Cả hai gọi giống nhau bằng cú pháp `/tên`, nhưng built-in được Anthropic định nghĩa và bảo trì, còn tùy biến do bạn (hoặc plugin) kiểm soát nội dung.

Mẹo thực dụng: `/clear` rất hữu ích giữa các tác vụ độc lập — context tích lũy lâu có thể làm câu trả lời "trôi dạt" theo ngữ cảnh cũ không còn liên quan; dọn sạch giúp Claude tập trung vào việc mới.

Bẫy thường gặp: nhầm slash command với lệnh shell. `/review` là lệnh của Claude Code, không phải lệnh chạy trong terminal hệ điều hành.

</details>

---

### 4.6. Skill có thể tổ chức theo những cấp nào (user / project)? Khi cùng tên thì cấp nào nên ưu tiên, và vì sao tách cấp lại quan trọng với một team?

<details>
<summary>👉 Xem câu trả lời</summary>

Skill tùy biến có thể tổ chức theo phân cấp giống cách Claude Code phân cấp `CLAUDE.md`:

- Cấp user: đặt trong `~/.claude` — áp dụng cho cá nhân bạn, dùng xuyên suốt mọi dự án trên máy.
- Cấp project: đặt trong `.claude` của repo — áp dụng riêng cho dự án đó, và được commit vào Git để cả team dùng chung.
- Ngoài ra skill còn có thể đến từ built-in (Anthropic cung cấp) hoặc từ plugin (đóng gói phân phối).

| Cấp | Vị trí | Phạm vi ảnh hưởng | Có chia sẻ với team không |
|---|---|---|---|
| Built-in | Có sẵn | Mọi nơi | Có (mặc định) |
| User | `~/.claude` | Mọi dự án của bạn | Không (riêng máy bạn) |
| Project | `.claude` | Chỉ dự án đó | Có (commit vào Git) |
| Plugin | Theo plugin | Tùy plugin cài | Có (qua plugin) |

Ý nghĩa của việc tách cấp với một team React/Next.js:

- Quy ước riêng của dự án (cấu trúc thư mục, thư viện UI dùng, cách viết test) nên đặt ở cấp project `.claude` và commit vào Git — như vậy mọi thành viên mở repo lên là có ngay cùng một bộ skill, không ai phải tự cấu hình lại.
- Thói quen cá nhân (ví dụ skill ghi chú riêng của bạn) nên đặt ở cấp user `~/.claude` để không "ép" lên team.

Về ưu tiên khi cùng tên: cơ chế phân cấp nói chung cho phép cấp gần với ngữ cảnh hơn (project, thư mục con) ghi đè/ưu tiên hơn cấp xa (user, enterprise) — giống cách `CLAUDE.md` hoạt động. Nếu không chắc thứ tự chính xác trong từng phiên bản, hãy hiểu nguyên tắc tổng quát: cấu hình theo dự án có tính cục bộ và thường thắng cấu hình cá nhân/toàn cục cho phạm vi của dự án đó.

Bẫy thường gặp: viết một skill chứa quy ước riêng của dự án nhưng lại đặt ở cấp user → người khác trong team không có skill đó, dẫn đến code không nhất quán. Quy tắc: cái gì thuộc về dự án thì để ở `.claude` của dự án và commit lại.

</details>

---

### 4.7. Plugin là gì trong bối cảnh skill, và nó giúp gì cho việc chia sẻ/phân phối skill trong tổ chức?

<details>
<summary>👉 Xem câu trả lời</summary>

Plugin là một cách đóng gói và phân phối lại các năng lực mở rộng cho Claude Code — trong đó có skill. Thay vì mỗi người tự tạo từng file `SKILL.md`, một plugin gom các skill (và có thể kèm cấu hình liên quan) thành một đơn vị cài đặt được, để chia sẻ cho nhiều người/nhiều máy.

So sánh ba con đường đưa một skill đến tay người dùng:

| Cách | Phạm vi phân phối | Phù hợp khi |
|---|---|---|
| Skill cấp project (`.claude`) | Trong một repo, qua Git | Quy trình riêng của một dự án |
| Skill cấp user (`~/.claude`) | Một máy/cá nhân | Thói quen cá nhân |
| Plugin | Nhiều người, nhiều dự án | Muốn phân phối lại như một "gói" dùng chung |

Liên hệ thực tế: giả sử công ty bạn có một bộ quy chuẩn frontend (lint rules, quy trình review accessibility, cách viết component) muốn phổ biến cho mọi team. Đóng gói các skill tương ứng thành một plugin sẽ giúp mọi người cài một lần là có đủ bộ skill, thay vì copy-paste thủ công các file `SKILL.md` vào từng repo.

Nói gọn về quan hệ giữa các khái niệm:
- `SKILL.md` định nghĩa một skill.
- Skill được gọi qua slash command.
- Plugin là một trong các nguồn cung cấp skill (bên cạnh built-in, user, project).

Lưu ý: chi tiết quy trình tạo/cài đặt plugin có thể khác nhau theo phiên bản; ý chính cần nắm là plugin đóng vai trò "kênh phân phối" để tái sử dụng skill ở quy mô tổ chức, không chỉ trong một repo đơn lẻ.

Bẫy thường gặp: nhầm plugin với MCP server. MCP (Model Context Protocol) là chuẩn kết nối Claude tới *tool/dữ liệu bên ngoài* (GitHub, database, Figma...). Plugin là cách *đóng gói và phân phối năng lực của Claude Code* (như skill). Hai thứ phục vụ mục đích khác nhau.

</details>

---

### 4.8. Hãy kể về một lần bạn dùng skill/slash command (hoặc đóng gói quy trình lặp lại thành công cụ tái dùng) để tăng tốc một quy trình lặp đi lặp lại trong nhóm.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Đây là câu hỏi hành vi — hãy kể một câu chuyện cụ thể theo cấu trúc STAR (Situation, Task, Action, Result), ưu tiên có số liệu. Dưới đây là một mẫu bạn có thể điều chỉnh theo trải nghiệm thật:

- Situation (Bối cảnh): "Nhóm frontend của tôi (4 người) làm một ứng dụng Next.js. Mỗi lần review PR, chúng tôi đều phải nhắc đi nhắc lại cùng một checklist: kiểm tra re-render thừa, ranh giới Server/Client Component, accessibility cơ bản, và không để lộ secret xuống client. Việc này tốn thời gian và mỗi người review một kiểu, không nhất quán."

- Task (Nhiệm vụ): "Tôi muốn chuẩn hóa quy trình review để vừa nhanh hơn vừa đồng đều giữa các reviewer, đồng thời giảm số bug lặp lại lọt qua review."

- Action (Hành động): "Tôi viết một skill tùy biến đặt ở cấp project trong `.claude`, định nghĩa trong `SKILL.md` với `description` nói rõ 'dùng khi review PR frontend trước khi merge'. Phần thân chứa checklist chi tiết và các bước cụ thể. Tôi commit nó vào Git để cả team có ngay. Từ đó mỗi reviewer chỉ cần gõ một slash command thay vì gõ lại đoạn prompt dài. Với những việc cần *tự động luôn xảy ra* như chạy lint/format sau khi sửa file, tôi tách riêng dùng hook trong `settings.json` chứ không nhồi vào skill, vì đó là việc của harness chứ không nên trông cậy AI tự nhớ."

- Result (Kết quả): "Sau khi áp dụng, thời gian review trung bình mỗi PR giảm rõ rệt và quan trọng hơn là review đồng đều giữa các thành viên — cùng một checklist, không phụ thuộc người review là ai. Số bug về re-render thừa và rò rỉ context client/server lọt qua review giảm hẳn. Tôi cũng định hướng nếu mở rộng cho nhiều team thì sẽ đóng gói các skill này thành plugin để phân phối lại."

Lời khuyên khi trả lời thật: chọn một câu chuyện bạn thực sự trải qua, nêu được con số (thời gian tiết kiệm, số bug giảm, số người dùng skill), và làm nổi bật tư duy phân biệt đúng công cụ — skill cho quy trình tái dùng, hook cho hành vi tự động theo sự kiện, plugin cho phân phối quy mô lớn.

</details>
## 5. Claude Code Agents và Subagents

### 5.1. Subagent trong Claude Code là gì, và vì sao việc "cô lập context" (context isolation) lại quan trọng?

<details>
<summary>👉 Xem câu trả lời</summary>

Subagent là một agent con do Claude Code tạo ra để xử lý một phần việc, và điểm cốt lõi là nó chạy trong một CONTEXT WINDOW RIÊNG, tách biệt với cuộc hội thoại chính. Agent chính giao việc, subagent tự đọc/sửa file, chạy lệnh, gọi tool trong vùng context của riêng mình, rồi chỉ trả về một bản tóm tắt kết quả cho agent cha.

Vì sao cô lập context lại quan trọng:

- Context window là hữu hạn. Nếu agent chính tự đọc 30 file để tìm một thứ, toàn bộ nội dung 30 file đó sẽ "ngốn" context của nó và làm loãng các thông tin quan trọng khác.
- Khi giao cho subagent, các bước trung gian ồn ào (nội dung file, log lệnh, kết quả tìm kiếm) nằm trong context của subagent. Agent cha chỉ nhận lại phần kết luận tinh gọn.
- Nhờ đó cuộc hội thoại chính giữ được "tín hiệu cao, nhiễu thấp" và đi xa hơn mà không bị đầy context.

```text
Agent CHÍNH (context của bạn)
  │  giao việc: "Tìm xem hàm validateToken được gọi ở đâu"
  ▼
Subagent (CONTEXT RIÊNG)
  - grep toàn repo, đọc 12 file, lần theo import...   ← ồn ào, nằm trong context con
  ▼
  trả về: "Được gọi ở 3 nơi: auth.guard.ts:21, ws.ts:88, jobs.ts:5"  ← chỉ kết luận về agent cha
```

Có thể ví von: subagent giống như giao một việc nghiên cứu cho đồng nghiệp — bạn không cần đọc hết đống tài liệu họ lật qua, bạn chỉ cần bản tóm tắt cuối cùng. Bẫy thường gặp: tưởng subagent "thấy" được toàn bộ lịch sử hội thoại chính — không, nó chỉ biết những gì agent cha mô tả trong lời giao việc, nên prompt giao việc phải đủ đầy đủ và độc lập.

</details>

---

### 5.2. Claude Code có những loại agent nào sẵn có? Phân biệt general-purpose, Explore và Plan.

<details>
<summary>👉 Xem câu trả lời</summary>

Claude Code cung cấp một số agent type dựng sẵn, mỗi loại tối ưu cho một mục đích, và bạn còn có thể tự định nghĩa agent tùy biến.

| Agent type | Mục đích | Khi nào dùng |
|---|---|---|
| general-purpose | Agent "vạn năng", có đủ tool để vừa tìm kiếm vừa sửa file vừa chạy lệnh | Việc phức tạp, nhiều bước, cần cả đọc lẫn ghi |
| Explore | Chuyên tìm kiếm, đọc hiểu codebase (thường thiên về read-only) | Trả lời "code này nằm ở đâu / hoạt động thế nào" mà không sửa gì |
| Plan | Chuyên lập kế hoạch, thiết kế giải pháp trước khi viết code | Cần một đề án/kiến trúc trước khi bắt tay sửa |

Khác biệt mấu chốt:

- Explore mạnh về fan-out tìm kiếm (tỏa ra nhiều hướng để định vị thông tin), tiêu tốn ít, kết quả là "bản đồ" của codebase. Thường được giao việc đọc nhiều file song song mà không làm bẩn context chính.
- Plan tập trung suy luận để ra một kế hoạch — gần với tinh thần "plan mode" của Claude Code: nghĩ và trình bày trước, chưa đụng vào code.
- general-purpose là lựa chọn mặc định khi việc cần làm nhiều thứ kết hợp và không gọn vào một vai chuyên biệt.

```text
"Repo này xử lý refresh token ở đâu?"      -> Explore (chỉ tìm, không sửa)
"Thiết kế cách thêm rate-limit cho API"    -> Plan   (ra kế hoạch, chưa code)
"Sửa toàn bộ module auth theo kế hoạch X"  -> general-purpose (đọc + sửa + chạy test)
```

Bẫy thường gặp: dùng general-purpose cho một việc chỉ-đọc đơn giản — vừa chậm vừa có rủi ro nó tự ý sửa file. Nếu chỉ cần tra cứu, Explore phù hợp và an toàn hơn.

</details>

---

### 5.3. Khi nào nên giao việc cho subagent thay vì để agent chính tự làm?

<details>
<summary>👉 Xem câu trả lời</summary>

Nên cân nhắc subagent khi việc rơi vào một trong các tình huống sau:

- Việc tốn nhiều context để khám phá nhưng kết quả lại gọn. Ví dụ: "lùng" cả repo để tìm chỗ định nghĩa một symbol — quá trình đọc rất nhiều, nhưng câu trả lời chỉ là vài dòng. Giao cho subagent để giữ context chính sạch.
- Việc có thể chia thành nhiều nhánh ĐỘC LẬP, chạy được song song (fan-out). Ví dụ: tìm đồng thời cách 3 module khác nhau xử lý lỗi.
- Việc cần một "vai" chuyên biệt với bộ tool hoặc chỉ dẫn riêng (vd review bảo mật, deep research).
- Việc dài/phức tạp mà bạn muốn cô lập để nếu nó "đi lạc" thì không làm hỏng dòng suy luận chính.

```text
Câu hỏi tự kiểm tra trước khi giao subagent:
  1. Việc này có "đọc nhiều, trả về ít" không?         -> nếu có, hợp với subagent
  2. Có chia được thành các nhánh độc lập song song?    -> nếu có, fan-out nhiều subagent
  3. Kết quả có cần độc lập với context hiện tại không?  -> nếu có, cô lập là điểm cộng
  Nếu cả 3 đều "không" và việc nhỏ/tuần tự -> làm trực tiếp, đừng tạo subagent
```

Ví dụ thực tế trong dự án NestJS + Prisma: trước khi refactor service `auth`, bạn giao một Explore subagent đi tìm "tất cả nơi import `AuthService` và gọi `validateUser`". Nó trả về danh sách vị trí; agent chính dùng danh sách đó để lên kế hoạch sửa — mà context chính không hề bị nhồi nội dung của 15 file.

Bẫy thường gặp: lạm dụng subagent cho mọi thứ. Một thao tác đọc một file rồi sửa một dòng thì tự làm nhanh hơn nhiều so với chi phí khởi tạo + giao việc + chờ subagent tóm tắt.

</details>

---

### 5.4. Chạy song song nhiều subagent (fan-out) hoạt động ra sao, và có lợi gì?

<details>
<summary>👉 Xem câu trả lời</summary>

Fan-out là kỹ thuật agent chính khởi tạo NHIỀU subagent cùng lúc, mỗi cái lo một nhánh việc độc lập, rồi gom kết quả lại. Vì mỗi subagent có context riêng và việc của chúng không phụ thuộc nhau, chúng chạy song song được.

Lợi ích:

- Giảm thời gian thực: nhiều nhánh tìm kiếm/phân tích diễn ra đồng thời thay vì nối tiếp.
- Cô lập context theo từng nhánh: kết quả của nhánh A không làm loãng nhánh B, và không nhánh nào làm đầy context chính.
- Phù hợp tự nhiên cho các tác vụ "bản đồ hóa" codebase: tìm cùng lúc nhiều thứ ở nhiều nơi.

```text
                 Agent CHÍNH
        ┌────────────┼────────────┐
        ▼            ▼            ▼
   Subagent A   Subagent B   Subagent C        (3 context riêng, chạy SONG SONG)
   "tìm chỗ      "tìm cách    "liệt kê test
    dùng DTO X"   handle lỗi"   cho module Y"
        └────────────┼────────────┘
                     ▼
        Agent chính gộp 3 bản tóm tắt -> ra quyết định
```

Lưu ý điều kiện song song: chỉ song song được khi các nhánh ĐỘC LẬP. Nếu nhánh B cần kết quả của nhánh A (vd "tìm userId rồi mới tìm đơn hàng của user đó"), thì phải làm tuần tự — giống nguyên tắc `Promise.all` chỉ dùng cho tác vụ độc lập.

Bẫy thường gặp: fan-out nhiều subagent để rồi tất cả cùng ghi vào một file — vừa khó dự đoán vừa dễ xung đột. Fan-out hợp nhất với các việc CHỈ ĐỌC / phân tích độc lập; còn việc ghi/sửa có phụ thuộc nhau thì nên tuần tự hoặc do agent chính điều phối cẩn thận.

</details>

---

### 5.5. Tự định nghĩa một subagent trong `.claude/agents` như thế nào? Có những gì cần khai báo?

<details>
<summary>👉 Xem câu trả lời</summary>

Bạn có thể tạo agent tùy biến cho dự án bằng cách đặt một file định nghĩa agent trong thư mục `.claude/agents` của repo (hoặc ở cấp user `~/.claude/agents` nếu muốn dùng cho mọi dự án). File này mô tả "vai" của agent: tên, mô tả khi nào nên dùng nó, và chỉ dẫn hệ thống (system prompt) định hình hành vi.

Ý tưởng chính là một agent tùy biến = một subagent có context riêng + một bộ chỉ dẫn chuyên biệt, có thể giới hạn tool và (tùy cấu hình) chỉ định model. Nhờ vậy bạn đóng gói một quy trình lặp lại (vd "agent review bảo mật", "agent viết test") để Claude tự gọi khi gặp việc phù hợp.

```markdown
---
name: test-writer
description: Dùng khi cần viết hoặc bổ sung unit test cho một service NestJS.
  Chủ động gọi sau khi thêm logic mới chưa có test.
---

Bạn là một agent chuyên viết test cho dự án NestJS + Prisma.
Quy tắc:
- Dùng Jest, đặt file *.spec.ts cạnh file nguồn.
- Mock PrismaService, không gọi DB thật.
- Bao phủ cả nhánh thành công lẫn nhánh lỗi (ví dụ NotFoundException).
- Sau khi viết xong, chạy test và báo lại kết quả ngắn gọn.
```

Phân biệt cấp độ (gợi nhớ giống phân cấp của CLAUDE.md):

- Cấp project: `.claude/agents` — đi theo repo, chia sẻ cho cả team.
- Cấp user: `~/.claude/agents` — riêng cho máy của bạn, dùng xuyên dự án.
- Ngoài ra còn các agent dựng sẵn (general-purpose, Explore, Plan) và agent đến từ plugin.

Bẫy thường gặp: viết `description` mơ hồ khiến Claude không biết khi nào nên gọi agent này. Mô tả nên nêu rõ "dùng khi…" và tình huống kích hoạt, vì đó chính là tín hiệu để agent chính quyết định ủy thác. Lưu ý: agent tùy biến chỉ định hình HÀNH VI; còn việc "tự động chạy mỗi khi có sự kiện X" (vd lint sau mỗi lần sửa file) là việc của hook, không phải của agent.

</details>

---

### 5.6. Subagent khác gì với Skill và với CLAUDE.md? Khi nào dùng cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Cả ba đều giúp Claude Code làm việc tốt hơn nhưng giải quyết các vấn đề khác nhau:

| Cơ chế | Bản chất | Giải quyết vấn đề gì |
|---|---|---|
| Subagent / Agent | Một tiến trình con có CONTEXT RIÊNG, có thể chạy song song | Cô lập context, fan-out việc lớn/độc lập |
| Skill | "Kỹ năng" tái sử dụng, định nghĩa trong `SKILL.md`, gọi qua slash command | Đóng gói một QUY TRÌNH chuyên biệt (vd code-review, deep-research) |
| CLAUDE.md | File ngữ cảnh/rule được tự nạp vào context | Cung cấp quy ước, kiến trúc, lệnh hay dùng cho dự án |

Cách phân biệt nhanh:

- Cần cô lập một khối việc tốn context, hoặc làm song song nhiều nhánh -> dùng subagent/agent.
- Cần một quy trình có thể gọi đi gọi lại theo tên -> đóng gói thành Skill (frontmatter `name`, `description`; có loại built-in, cấp user `~/.claude`, cấp project `.claude`, hoặc từ plugin).
- Cần Claude luôn nhớ "quy ước code của dự án", "lệnh build/test", "kiến trúc thư mục" -> ghi vào CLAUDE.md.

```text
Tình huống: "Mỗi lần làm việc với repo này, hãy luôn dùng pnpm và đặt DTO trong /dto"
  -> CLAUDE.md (ngữ cảnh nền, tự nạp)

Tình huống: "Tôi muốn một quy trình review bảo mật gọi bằng /security-review"
  -> Skill (SKILL.md)

Tình huống: "Tìm song song xem 3 module xử lý lỗi thế nào, đừng làm đầy context của tôi"
  -> fan-out nhiều Explore subagent
```

Lưu ý: chúng có thể kết hợp. Một subagent vẫn đọc CLAUDE.md để biết quy ước dự án, và một Skill có thể được thực thi bên trong một subagent. Bẫy thường gặp: nhầm subagent với CLAUDE.md — CLAUDE.md không tạo context riêng, nó chỉ thêm thông tin vào context hiện có.

</details>

---

### 5.7. Subagent quan hệ thế nào với Workflows và với MCP? Phân biệt điều phối "agent tự quyết" và điều phối "tất định".

<details>
<summary>👉 Xem câu trả lời</summary>

Điều phối (orchestration) là cách bạn phối hợp nhiều agent/bước để hoàn thành một việc lớn. Có hai phong cách, và subagent xuất hiện ở cả hai:

- Điều phối kiểu "agent tự quyết": agent chính tự QUYẾT ĐỊNH lúc chạy là có nên tạo subagent không, tạo bao nhiêu, giao việc gì. Linh hoạt, hợp với việc mơ hồ, khó đặc tả trước.
- Workflows (điều phối tất định): bạn LẬP TRÌNH bằng script để điều phối nhiều agent một cách tất định (deterministic) — bước nào chạy trước, fan-out ra sao, gom kết quả thế nào đều cố định. Hợp với việc lớn, lặp lại, cần độ tin cậy và tái lập cao.

MCP (Model Context Protocol) thì khác cấp độ: nó là chuẩn mở để KẾT NỐI AI với tool/dữ liệu bên ngoài qua "MCP server" (expose tools, resources, prompts). MCP không phải là agent; nó MỞ RỘNG khả năng mà agent (kể cả subagent) có thể dùng — ví dụ thêm MCP server cho GitHub, database, Linear, Figma, Notion để subagent có thêm tool truy vấn các hệ thống đó.

```text
Agent tự quyết (runtime quyết định):
  Agent chính --(thấy cần)--> sinh 0..n subagent tùy tình huống

Workflow (tất định, viết bằng script):
  Bước 1: fan-out 5 subagent phân tích 5 module   (cố định)
  Bước 2: gom kết quả                              (cố định)
  Bước 3: 1 subagent tổng hợp báo cáo              (cố định)

MCP: mặt cắt ngang — cấp thêm TOOL cho agent/subagent
  (GitHub / database / Linear / Figma / Notion ...)
```

Liên hệ Claude API: ở phía API, các khái niệm tương đương cũng tồn tại — model id (claude-opus, claude-sonnet, claude-haiku, claude-fable), tool use/function calling, structured output, và hỗ trợ agents cũng như MCP — nên kiến trúc nhiều agent bạn dựng trong Claude Code có thể ánh xạ sang khi gọi qua API.

Bẫy thường gặp: chọn workflow tất định cho một việc bản chất là mơ hồ (mỗi lần một khác) sẽ gãy ngay khi thực tế lệch khỏi script; ngược lại, để agent "tự quyết" cho một quy trình production cần tái lập chính xác thì khó kiểm soát. Chọn phong cách theo độ xác định của bài toán.

</details>

---

### 5.8. Subagent có những ưu/nhược điểm gì, và khi nào KHÔNG nên dùng subagent?

<details>
<summary>👉 Xem câu trả lời</summary>

Ưu điểm:

- Cô lập context: giữ cuộc hội thoại chính sạch, đi được xa hơn mà không đầy context.
- Song song hóa: fan-out nhiều nhánh độc lập để giảm thời gian.
- Chuyên môn hóa: mỗi agent một vai, có bộ tool/chỉ dẫn riêng (vd review, research).
- Hạn chế rủi ro: nếu một nhánh đi lạc, nó không kéo theo cả dòng suy luận chính.

Nhược điểm / chi phí:

- Chi phí khởi tạo và giao việc: tạo subagent + viết prompt giao việc + chờ tóm tắt không miễn phí.
- Mất ngữ cảnh ngầm: subagent KHÔNG thấy lịch sử hội thoại; nếu giao việc thiếu thông tin, nó sẽ làm sai hoặc hiểu lệch.
- Khó phối hợp khi có phụ thuộc: nhiều subagent cùng ghi/sửa dễ xung đột, khó suy luận về thứ tự.
- "Tóm tắt làm mất chi tiết": bạn chỉ nhận lại bản tóm tắt, đôi khi mất chi tiết mà agent cha cần.

Khi nào KHÔNG nên dùng subagent:

| Tình huống | Vì sao tự làm tốt hơn |
|---|---|
| Việc nhỏ, 1-2 thao tác (đọc 1 file, sửa 1 dòng) | Chi phí ủy thác > giá trị; làm trực tiếp nhanh hơn |
| Việc cần BÁM SÁT ngữ cảnh hội thoại đang diễn ra | Subagent không thấy lịch sử, dễ lệch |
| Các bước PHỤ THUỘC nhau chặt (output bước trước là input bước sau) | Không song song được; điều phối phức tạp không cần thiết |
| Cần hành vi "tự động mỗi khi có sự kiện" (vd format sau khi lưu) | Đó là việc của HOOK, không phải subagent |

```text
NÊN giao subagent:  "đọc-nhiều-trả-ít" + độc lập + cô lập context có lợi
KHÔNG nên:          việc nhỏ / phụ thuộc ngữ cảnh / phụ thuộc tuần tự / cần tự-động-theo-sự-kiện
```

Bẫy thường gặp: dùng subagent với hy vọng nó "tự động hóa" một hành vi lặp lại theo sự kiện. Hành vi kiểu "luôn làm X mỗi khi Y xảy ra" phải dùng hook (PreToolUse, PostToolUse, Stop...) cấu hình trong settings.json — vì hook được harness tự chạy theo sự kiện, còn subagent chỉ chạy khi được giao việc.

</details>
## 6. Claude Code Rules CLAUDE.md và Memory

### 6.1. File `CLAUDE.md` dùng để làm gì, và tại sao nó hiệu quả hơn việc nhắc lại ngữ cảnh trong từng prompt?

<details>
<summary>👉 Xem câu trả lời</summary>

`CLAUDE.md` là file "rule" / ngữ cảnh dự án mà Claude Code **tự động nạp vào context** ở đầu mỗi phiên (không cần bạn dán lại). Mục tiêu là cung cấp cho agent những thông tin bền vững về dự án: coding conventions, các lệnh hay dùng (build/test/lint), kiến trúc tổng quan, và những điều "đừng làm". Nhờ vậy agent hành xử nhất quán mà không phải hỏi lại hay đoán mò.

Lý do nó tốt hơn việc nhắc trong từng prompt:
- **Không bị quên giữa phiên**: context window có giới hạn; nếu bạn chỉ nói "dùng pnpm chứ đừng dùng npm" ở tin nhắn đầu, sau nhiều bước agent có thể trôi mất chi tiết đó. Rule trong `CLAUDE.md` luôn hiện diện.
- **Tái sử dụng qua nhiều phiên và nhiều người**: commit `CLAUDE.md` vào repo thì cả team (và mọi phiên Claude Code) đều dùng chung một bộ quy ước.
- **Giảm prompt lặp đi lặp lại**: bạn không phải gõ lại "chạy test bằng `pnpm test`" mỗi lần.

Ví dụ một `CLAUDE.md` gọn cho dự án NestJS:

```md
# Dự án: Jira Clone API (NestJS + Prisma)

## Lệnh hay dùng
- Cài dependency: `pnpm install`
- Chạy dev: `pnpm start:dev`
- Test: `pnpm test` (đừng dùng `npm`)
- Migrate DB: `pnpm prisma migrate dev`

## Quy ước code
- Dùng dependency injection, KHÔNG khởi tạo service bằng `new`.
- DTO validate bằng class-validator; mọi endpoint phải có DTO.
- Trả lỗi qua HttpException, không trả object lỗi tự chế.

## Kiến trúc
- Tầng: controller -> service -> repository (Prisma).
- Business logic nằm ở service, KHÔNG ở controller.
```

**Bẫy thường gặp**: nhồi `CLAUDE.md` quá dài (cả nghìn dòng) làm tốn context và loãng tín hiệu; tốt nhất viết ngắn gọn, chỉ giữ thứ thật sự định hướng được hành vi. Một lỗi khác là viết rule chung chung kiểu "viết code sạch" — vô nghĩa với agent; hãy nêu quy tắc cụ thể, kiểm chứng được.

</details>

---

### 6.2. Làm sao để viết rule tốt trong `CLAUDE.md`? Nêu nguyên tắc và những loại thông tin nên đưa vào.

<details>
<summary>👉 Xem câu trả lời</summary>

Một `CLAUDE.md` tốt nên ngắn, cụ thể, và "actionable" (agent đọc xong biết phải làm gì). Các nhóm nội dung đáng đưa vào:

1. **Lệnh hay dùng**: build, run, test, lint, migrate. Đây là thứ agent cần nhất để tự kiểm chứng công việc.
2. **Coding conventions**: quy ước đặt tên, style (đã có Prettier/ESLint thì nói rõ "tuân theo config sẵn có"), pattern bắt buộc (DI, repository pattern...).
3. **Kiến trúc**: sơ đồ tầng, module quan trọng nằm ở đâu, luồng dữ liệu chính. Giúp agent định hướng khi sửa code mà không phải đọc cả repo.
4. **Điều CẤM / cảnh báo**: ví dụ "đừng sửa file migration đã commit", "đừng commit `.env`", "đừng gọi API thật trong test".

Nguyên tắc viết:
- **Cụ thể, kiểm chứng được**: "dùng `pnpm` chứ không `npm`" tốt hơn "quản lý package cẩn thận".
- **Ngắn gọn, có cấu trúc**: dùng heading và bullet, để agent (và người) quét nhanh.
- **Dùng nhấn mạnh cho rule quan trọng**: viết IN HOA hoặc dùng từ như "LUÔN/KHÔNG BAO GIỜ" cho ràng buộc cứng.
- **Cập nhật khi thực tế thay đổi**: rule lỗi thời còn nguy hiểm hơn không có rule.

So sánh rule kém vs rule tốt:

| Rule kém (mơ hồ) | Rule tốt (cụ thể, hành động được) |
|---|---|
| "Viết test đầy đủ" | "Mọi service mới phải có unit test; chạy `pnpm test` trước khi báo xong" |
| "Code phải sạch" | "Tuân theo ESLint config có sẵn; không thêm `// eslint-disable` nếu chưa giải thích" |
| "Cẩn thận với DB" | "KHÔNG sửa file migration đã tồn tại; luôn tạo migration mới bằng `prisma migrate dev`" |

**Mẹo thực tế**: chạy slash command built-in `/init` để Claude Code quét codebase và sinh bản nháp `CLAUDE.md` ban đầu, rồi bạn tinh chỉnh thay vì viết từ con số 0.

</details>

---

### 6.3. `CLAUDE.md` có phân cấp như thế nào (enterprise / user / project / thư mục con) và chúng kết hợp ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

`CLAUDE.md` (và file rule nói chung) có nhiều **cấp**, được nạp và hợp nhất lại để tạo ngữ cảnh cuối cùng. Theo thứ tự phạm vi từ rộng đến hẹp:

1. **Enterprise**: rule do tổ chức đặt ra, áp cho mọi dev trong công ty (ví dụ chính sách bảo mật, không gọi service nội bộ trái phép).
2. **User** (`~/.claude/CLAUDE.md`): sở thích cá nhân của bạn, áp cho **mọi dự án** bạn làm. Ví dụ "trả lời ngắn gọn", "ưu tiên giải thích bằng tiếng Việt". File này nằm ở máy bạn, **không commit** vào repo dự án.
3. **Project** (`CLAUDE.md` ở gốc repo): quy ước riêng của dự án, **commit vào git** để cả team dùng chung.
4. **Thư mục con**: `CLAUDE.md` đặt trong một thư mục con áp ngữ cảnh hẹp hơn cho phần code ở đó (ví dụ rule riêng cho thư mục `packages/payment`).

Cách kết hợp: các cấp được **gộp lại** chứ không loại trừ nhau; cấp hẹp/cụ thể hơn thường bổ sung hoặc tinh chỉnh cho cấp rộng hơn.

```text
Enterprise rule  (toàn công ty)
        │  + nạp chồng
User ~/.claude/CLAUDE.md  (mọi dự án của bạn)
        │  +
Project ./CLAUDE.md  (cả team, commit vào git)
        │  +
Sub-dir ./packages/payment/CLAUDE.md  (chỉ thư mục đó)
        ▼
   Ngữ cảnh cuối cùng nạp vào agent
```

**Khi nào dùng cái nào**: sở thích cá nhân (giọng văn, ngôn ngữ trả lời) -> cấp user; quy ước dự án (lệnh build, kiến trúc) -> cấp project và commit; ràng buộc đặc thù một module -> cấp thư mục con.

**Bẫy thường gặp**: nhét sở thích cá nhân vào `CLAUDE.md` của project rồi commit, khiến đồng đội bị áp đặt thói quen của riêng bạn. Sở thích cá nhân nên ở cấp user.

</details>

---

### 6.4. Memory khác `CLAUDE.md` ở điểm nào? Khi nào nên dùng Memory, khi nào nên dùng `CLAUDE.md`?

<details>
<summary>👉 Xem câu trả lời</summary>

Cả hai đều giúp Claude Code "nhớ" thông tin qua thời gian, nhưng bản chất và cách quản lý khác nhau:

- **`CLAUDE.md`**: là file rule **do bạn chủ động viết và kiểm soát**, thường nằm trong repo, dùng để định hình hành vi agent (conventions, lệnh, kiến trúc). Nó là "hợp đồng" rõ ràng, review được qua git.
- **Memory**: cơ chế **ghi nhớ sự kiện / sở thích bền vững qua nhiều phiên làm việc**. Phù hợp cho những điều phát sinh trong lúc làm việc mà bạn muốn agent nhớ về sau, ví dụ "dự án này dùng Postgres ở cổng 5433 vì 5432 đã bị PG bản máy chiếm", hay "user tự gõ code, mình chỉ hướng dẫn".

Khác biệt cốt lõi:

| Tiêu chí | `CLAUDE.md` | Memory |
|---|---|---|
| Bản chất | File rule/ngữ cảnh do người viết | Ghi nhớ sự kiện/sở thích bền vững |
| Phạm vi | Theo dự án (hoặc user/enterprise) | Bền vững qua nhiều phiên |
| Cách quản lý | Sửa file thủ công, review qua git | Tích lũy/cập nhật trong quá trình dùng |
| Mục đích chính | Định hình hành vi, quy ước cứng | Nhớ ngữ cảnh, bối cảnh, sở thích |

**Khi nào dùng cái nào**:
- Quy ước cứng, cần cả team thấy và review -> `CLAUDE.md` (commit vào repo).
- Bối cảnh/sở thích cá nhân tích lũy dần, không muốn tự tay biên tập file -> Memory.

**Lưu ý quan trọng**: cả `CLAUDE.md` lẫn Memory đều là **ngữ cảnh thụ động** — chúng *mô tả* cho agent biết, nhưng KHÔNG đảm bảo "tự động chạy lệnh X mỗi khi Y". Nếu bạn cần hành vi tự động chắc chắn (ví dụ luôn format sau khi sửa file), đó là việc của **hook**, không phải Memory hay rule (xem câu 6.6).

</details>

---

### 6.5. "Tránh rule mâu thuẫn" nghĩa là gì? Điều gì xảy ra khi rule xung đột và làm sao phòng tránh?

<details>
<summary>👉 Xem câu trả lời</summary>

Vì rule đến từ nhiều cấp (enterprise, user, project, thư mục con) và còn cộng với Memory, rất dễ xảy ra **mâu thuẫn** — hai chỉ dẫn ngược nhau. Khi đó agent phải "đoán" nên theo cái nào, dẫn đến hành vi không nhất quán, khó dự đoán, và bạn mất niềm tin vào nó.

Ví dụ xung đột điển hình:

```md
# Trong ~/.claude/CLAUDE.md (user)
- Luôn dùng npm để cài package.

# Trong ./CLAUDE.md (project)
- Dự án này dùng pnpm; KHÔNG dùng npm (sẽ phá lockfile).
```

Agent thấy hai chỉ dẫn ngược nhau về package manager. Dù cấp project thường thắng, việc để tồn tại mâu thuẫn vẫn là rủi ro.

Cách phòng tránh:
1. **Phân tầng đúng vai trò**: sở thích cá nhân để ở user; quy ước dự án để ở project. Đừng đặt thứ mang tính dự án vào file user và ngược lại.
2. **Một nguồn sự thật cho mỗi quyết định**: package manager, cách chạy test... chỉ nên được quy định ở một nơi.
3. **Rule cụ thể chiến thắng rule chung**: viết rule hẹp/cụ thể cho thư mục con thay vì viết rule mâu thuẫn ở gốc.
4. **Định kỳ dọn dẹp**: rà soát `CLAUDE.md` và Memory để bỏ rule lỗi thời (ví dụ đã chuyển từ Jest sang Vitest nhưng rule cũ vẫn ghi Jest).
5. **Diễn đạt rõ ràng, không nhập nhằng**: "ưu tiên X trừ khi Y" rõ hơn là hai câu rule rời rạc cùng nói về X.

**Bẫy thường gặp**: copy `CLAUDE.md` từ dự án cũ sang dự án mới mà quên sửa, để lại lệnh/kiến trúc không còn đúng; hoặc Memory ghi một sở thích cũ ("dùng REST") trong khi dự án mới đã chuyển sang GraphQL — agent bị kéo theo hai hướng.

</details>

---

### 6.6. Phân biệt project settings và user settings (`settings.json` vs `settings.local.json`). Cái gì nên ở đâu, và vì sao "tự động làm X" phải dùng hook chứ không phải rule?

<details>
<summary>👉 Xem câu trả lời</summary>

Bên cạnh file rule dạng văn bản (`CLAUDE.md`), Claude Code còn có **file cấu hình `settings.json`** quy định permissions, biến môi trường, hooks và model. Cũng có phân cấp tương tự:

- **User settings** (`~/.claude/settings.json`): cấu hình cá nhân, áp cho mọi dự án của bạn (ví dụ model mặc định, permission cá nhân).
- **Project settings** (`.claude/settings.json` trong repo): cấu hình dùng chung cho cả team, **commit vào git** (ví dụ cho phép chạy `pnpm test` mà không hỏi, hook format toàn dự án).
- **Project local settings** (`.claude/settings.local.json`): ghi đè ở mức cá nhân **cho riêng dự án này**, thường **không commit** (đưa vào `.gitignore`) — dùng cho thứ riêng tư hoặc khác biệt của máy bạn (ví dụ đường dẫn local, một permission bạn tự bật).

Phân biệt với `CLAUDE.md`: `CLAUDE.md` là **ngữ cảnh ngôn ngữ tự nhiên** (agent đọc và hiểu), còn `settings.json` là **cấu hình harness** (chương trình Claude Code thực thi tất định). Quyền hạn, hook, model — thuộc `settings.json`. Quy ước code, kiến trúc — thuộc `CLAUDE.md`.

Vì sao "tự động làm X mỗi khi Y" phải dùng **hook** chứ không phải rule:
- Rule trong `CLAUDE.md` chỉ *nhờ* AI nhớ làm; AI có thể quên, bỏ qua, hoặc làm không nhất quán — đó là hành vi xác suất.
- **Hook** là lệnh shell do **harness tự chạy** khi có sự kiện (`PostToolUse`, `PreToolUse`, `Stop`, `UserPromptSubmit`, `SessionStart`...). Nó chạy **tất định**, không phụ thuộc agent "có nhớ hay không".

Ví dụ: bạn muốn "mỗi khi sửa file `.ts` thì tự chạy Prettier". Viết rule trong `CLAUDE.md` thì agent đôi khi quên. Đúng cách là cấu hình hook `PostToolUse`:

```jsonc
// .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          // Harness truyền thông tin sự kiện (gồm đường dẫn file) dưới dạng JSON
          // qua stdin; ở đây dùng jq để lấy đường dẫn rồi format.
          { "type": "command", "command": "jq -r '.tool_input.file_path' | xargs pnpm prettier --write" }
        ]
      }
    ]
  }
}
```

**Khi nào dùng cái nào**:
- "Quy ước, gợi ý, ngữ cảnh" mà agent nên cân nhắc -> `CLAUDE.md`.
- "Bắt buộc tự động xảy ra mỗi lần" (format, lint, chặn lệnh nguy hiểm, gửi thông báo) -> hook trong `settings.json`.
- "Cho phép/cấm một loại lệnh/tool" -> permissions trong `settings.json`.

**Bẫy thường gặp**: viết "luôn chạy lint sau khi sửa code" trong `CLAUDE.md` rồi ngạc nhiên khi agent thỉnh thoảng bỏ qua — vì đó là rule, không phải hook. Một bẫy khác là commit `settings.local.json` chứa thông tin máy cá nhân vào repo, gây nhiễu cho đồng đội.

</details>

---

### 6.7. Hãy đưa một ví dụ `CLAUDE.md` thực tế "đáng giá" cho dự án backend, và giải thích vì sao mỗi mục giúp ích cho agent.

<details>
<summary>👉 Xem câu trả lời</summary>

Một `CLAUDE.md` "đáng giá" là file mà nếu thiếu nó, agent sẽ hỏi lại hoặc làm sai. Dưới đây là mẫu cho một API NestJS + Prisma, kèm lý do từng mục:

```md
# Jira Clone API

## Tech stack
- NestJS, Prisma (PostgreSQL), pnpm.

## Lệnh hay dùng
- Dev: `pnpm start:dev`
- Test: `pnpm test`  | Test 1 file: `pnpm test -- path/to/spec.ts`
- Lint: `pnpm lint`
- Migration: `pnpm prisma migrate dev --name <ten>`

## Database (LƯU Ý)
- Postgres chạy qua Docker ở cổng 5433 (cổng 5432 đã bị PG bản máy chiếm).
- Connection string đọc từ `.env`; KHÔNG hardcode.

## Quy ước
- Tầng: controller -> service -> repository. Business logic ở service.
- Mọi endpoint phải có DTO validate bằng class-validator.
- Lỗi trả qua HttpException với mã status đúng ngữ nghĩa.
- KHÔNG sửa migration đã commit; luôn tạo migration mới.

## Trước khi báo "xong"
- Chạy `pnpm lint` và `pnpm test`, đảm bảo pass.
```

Vì sao mỗi mục giúp ích:
- **Tech stack + lệnh**: agent biết ngay cách build/test/migrate để **tự kiểm chứng** việc nó làm, thay vì đoán hoặc dùng sai công cụ (npm thay vì pnpm).
- **Mục Database**: chứa "tri thức ngầm" mà chỉ người trong dự án mới biết (cổng 5433). Không ghi ra thì agent rất dễ kết nối nhầm 5432 rồi báo lỗi khó hiểu.
- **Quy ước kiến trúc**: ép agent đặt logic đúng tầng, giữ codebase nhất quán khi nó tự thêm code mới.
- **Điều CẤM (migration)**: ngăn lỗi nghiêm trọng, khó đảo ngược.
- **"Trước khi báo xong"**: định nghĩa rõ "thế nào là hoàn thành", giúp agent tự verify thay vì dừng sớm.

**Bẫy thường gặp**: viết những thứ agent tự suy ra được từ code (kiểu "đây là dự án TypeScript") — vô ích, chỉ tốn context. Hãy tập trung vào **tri thức ngầm** (quyết định, ràng buộc, lệnh đặc thù) mà agent không thể đoán bằng cách đọc file. Và nhớ: phần "tự động chạy lint mỗi lần" nếu muốn chắc chắn thì chuyển sang hook (xem 6.6); ở đây nó chỉ là nhắc nhở.

</details>
## 7. Claude Code Hooks và Tự động hóa

### 7.1. Hook trong Claude Code là gì? Nó hoạt động theo cơ chế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Hook là một lệnh shell mà chính harness của Claude Code (chứ không phải model AI) tự động chạy khi xảy ra một sự kiện (event) trong vòng đời của phiên làm việc. Bạn cấu hình hook trong `settings.json`, gắn nó với một loại sự kiện, và mỗi khi sự kiện đó xảy ra, harness sẽ thực thi lệnh đó một cách tất định (deterministic) — luôn luôn chạy, không phụ thuộc vào việc model "có nhớ" hay "có muốn" hay không.

Điểm mấu chốt cần nắm: hook do **harness** chạy, không phải do model. Đây là một sự khác biệt lớn về độ tin cậy. Model có thể quên, có thể diễn giải sai chỉ dẫn; còn hook là một quy tắc cứng — đã cấu hình là chạy.

Các loại sự kiện (event) phổ biến:

| Event | Kích hoạt khi |
|---|---|
| `PreToolUse` | Trước khi một tool được chạy (vd trước khi sửa file, chạy lệnh shell) |
| `PostToolUse` | Sau khi một tool chạy xong |
| `UserPromptSubmit` | Khi người dùng gửi một prompt |
| `Stop` | Khi Claude kết thúc lượt trả lời (agent dừng) |
| `SessionStart` | Khi một phiên làm việc bắt đầu |

```jsonc
// .claude/settings.json — chạy prettier sau mỗi lần Claude ghi/sửa file
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          // Harness truyền chi tiết tool call (gồm đường dẫn file) cho hook qua stdin dạng JSON.
          // Script đọc stdin để lấy file vừa sửa rồi format.
          { "type": "command", "command": ".claude/hooks/format.sh" }
        ]
      }
    ]
  }
}
```

Liên hệ thực tế (dự án React/Next.js): một hook `PostToolUse` chạy `eslint --fix` mỗi khi Claude sửa file `.tsx` đảm bảo code Claude sinh ra luôn theo đúng coding convention của team, không cần nhắc lại trong mỗi prompt.

Bẫy thường gặp: nghĩ rằng hook là "một loại lệnh trong chat". Không — hook không phải `/command`; nó là cấu hình hệ thống, chạy ngầm khi event xảy ra.

</details>

---

### 7.2. Tại sao một hành vi "tự động mỗi khi X" PHẢI dùng hook, chứ không phải nhờ AI ghi nhớ?

<details>
<summary>👉 Xem câu trả lời</summary>

Đây là một trong những hiểu lầm quan trọng nhất khi mới dùng Claude Code. Khi bạn muốn "luôn luôn format code sau khi sửa", "luôn chạy test trước khi commit", hay "luôn gửi thông báo khi Claude làm xong", bạn có hai lựa chọn:

1. **Nhờ AI ghi nhớ** (qua `CLAUDE.md`, Memory, hoặc nhắc trong prompt) — đây là một chỉ dẫn mềm. Model *có thể* làm theo, nhưng cũng có thể quên, bỏ qua khi context dài, hoặc diễn giải khác đi. Nó **không đảm bảo** (non-deterministic).
2. **Dùng hook** — harness *luôn luôn* chạy lệnh đó khi event xảy ra. Đây là đảm bảo cứng, tất định (deterministic).

Quy tắc vàng: bất cứ khi nào yêu cầu có dạng "mỗi khi X thì luôn làm Y", "trước/sau mỗi lần X", "tự động Y" — đó là tín hiệu phải dùng hook. Ghi nhớ/preference chỉ phù hợp cho sở thích hay ngữ cảnh (vd "tôi thích dùng pnpm"), không phù hợp cho hành vi bắt buộc lặp lại.

```text
Yêu cầu: "Mỗi lần sửa file TypeScript thì luôn chạy lint."

❌ SAI: thêm vào CLAUDE.md dòng "Hãy luôn chạy eslint sau khi sửa .ts"
        -> phụ thuộc model có nhớ làm hay không -> không đảm bảo.

✅ ĐÚNG: cấu hình hook PostToolUse với matcher "Edit|Write" chạy eslint
        -> harness luôn chạy, kể cả khi model không nghĩ tới.
```

Lý do sâu xa: model là một thành phần xác suất (probabilistic). Bạn không thể xây dựng một quy trình kỹ thuật đáng tin cậy trên một thành phần xác suất nếu kết quả phải chắc chắn 100%. Hook tách phần "đảm bảo" ra khỏi model và đặt nó vào harness — vốn là phần tất định.

Bẫy thường gặp: viết "QUAN TRỌNG: BẠN PHẢI luôn..." trong `CLAUDE.md` rồi nghĩ đã chắc chắn. Câu chữ mạnh không biến một chỉ dẫn mềm thành đảm bảo cứng — chỉ hook mới làm được.

</details>

---

### 7.3. Hãy cho vài ví dụ thực tế về việc dùng hook. Mỗi event hợp với loại tác vụ nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Hook mạnh nhất khi gắn đúng tác vụ với đúng event. Một số mẫu thường dùng:

| Tác vụ | Event phù hợp | Vì sao |
|---|---|---|
| Format/lint tự động (prettier, eslint) | `PostToolUse` (matcher `Edit\|Write`) | Chạy ngay sau khi file vừa được sửa (hook đọc đường dẫn file từ JSON stdin) |
| Chặn lệnh nguy hiểm (vd `rm -rf`, sửa file `.env`) | `PreToolUse` | Chạy *trước* khi tool thực thi nên có thể từ chối |
| Gửi thông báo (desktop notification, Slack) khi xong việc | `Stop` | Báo cho người dùng biết agent đã trả lời xong |
| Nạp ngữ cảnh đầu phiên (vd in ra branch git, trạng thái) | `SessionStart` | Chạy một lần khi mở phiên |
| Chèn nhắc nhở/kiểm tra mỗi prompt | `UserPromptSubmit` | Chạy mỗi khi người dùng gửi yêu cầu |

```jsonc
// Ví dụ 1: format tự động sau khi sửa file (PostToolUse)
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          // Script đọc JSON từ stdin để biết file nào vừa được sửa, rồi format.
          { "type": "command", "command": ".claude/hooks/format.sh" }
        ]
      }
    ]
  }
}
```

```bash
# .claude/hooks/format.sh — đọc đường dẫn file từ JSON stdin rồi chạy prettier
#!/usr/bin/env bash
# Harness gửi chi tiết tool call (JSON) qua stdin; ta lấy trường file_path.
input=$(cat)
file=$(echo "$input" | jq -r '.tool_input.file_path // empty')
[ -n "$file" ] && prettier --write "$file" 2>/dev/null || true
exit 0
```

```jsonc
// Ví dụ 2: chặn lệnh nguy hiểm trước khi chạy (PreToolUse)
// Hook trả về exit code khác 0 để báo harness CHẶN tool đó lại.
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          { "type": "command", "command": "scripts/guard-dangerous-commands.sh" }
        ]
      }
    ]
  }
}
```

```bash
# scripts/guard-dangerous-commands.sh — chặn rm -rf phá hủy
#!/usr/bin/env bash
# Harness truyền chi tiết tool call qua stdin (JSON) cho hook đọc.
input=$(cat)
if echo "$input" | grep -Eq 'rm[[:space:]]+-rf?[[:space:]]+/'; then
  echo "Bị chặn: lệnh xóa đệ quy nguy hiểm." >&2
  exit 2   # exit khác 0 -> chặn không cho tool chạy
fi
exit 0
```

Liên hệ React/Next.js: cặp hook hữu ích nhất cho dự án frontend là `PostToolUse` chạy `eslint --fix` + `prettier`, và (với team dùng TypeScript) một hook chạy `tsc --noEmit` để bắt lỗi kiểu ngay khi Claude vừa sửa code.

Bẫy thường gặp: cho hook `PostToolUse` chạy lệnh quá nặng (vd build cả dự án, chạy toàn bộ test suite) sau *mỗi* lần sửa file — làm mỗi thao tác chậm chạp. Nên giới hạn phạm vi (chỉ lint file vừa đổi) và để kiểm tra nặng cho event ít xảy ra hơn (như `Stop`).

</details>

---

### 7.4. Hook được cấu hình ở đâu? Cấu trúc cấu hình ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

Hook được khai báo trong file `settings.json` (hoặc `settings.local.json` cho cấu hình riêng máy của bạn, không commit vào repo). Cấu trúc gồm khóa `hooks`, bên trong là từng tên event, mỗi event chứa một mảng các "matcher group" (nhóm khớp), và mỗi nhóm chứa danh sách lệnh sẽ chạy.

`settings.json` không chỉ giữ hook — nó còn cấu hình `permissions` (quyền), biến môi trường (`env`), `model`. Hook là một phần trong đó.

```jsonc
// .claude/settings.json
{
  "hooks": {
    "<TênEvent>": [
      {
        "matcher": "<mẫu khớp tool>",   // vd "Edit|Write", "Bash"; bỏ qua nếu event không gắn tool
        "hooks": [
          { "type": "command", "command": "<lệnh shell sẽ chạy>" }
        ]
      }
    ]
  }
}
```

`matcher` dùng để lọc theo loại tool (chỉ áp dụng cho event gắn với tool như `PreToolUse`/`PostToolUse`). Ví dụ `"Edit|Write"` nghĩa là chỉ chạy hook khi tool là Edit hoặc Write. Với event không gắn tool (như `Stop`, `SessionStart`), bạn không cần `matcher`.

Phân cấp cấu hình (giống `CLAUDE.md`): có nhiều tầng `settings.json` — cấp enterprise, cấp user (`~/.claude/settings.json`), và cấp project (`.claude/settings.json`). Hook định nghĩa ở các tầng được áp dụng phối hợp; tầng project thường dùng cho hook riêng của dự án (lint/format theo convention của team), tầng user cho hook cá nhân (vd thông báo).

So sánh `settings.json` vs `settings.local.json`:

| File | Mục đích | Có commit không? |
|---|---|---|
| `.claude/settings.json` | Cấu hình chung của project, chia sẻ cho cả team | Có |
| `.claude/settings.local.json` | Cấu hình riêng máy bạn (đè lên project) | Không (thường vào `.gitignore`) |

Bẫy thường gặp: đặt hook chứa đường dẫn tuyệt đối riêng máy bạn (vd `/Users/an/...`) vào `settings.json` của project rồi commit — đồng đội sẽ bị lỗi. Cấu hình riêng máy nên đặt ở `settings.local.json`.

</details>

---

### 7.5. Phân biệt Hook với Skill và với rule trong CLAUDE.md. Khi nào dùng cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Ba thứ này dễ bị nhầm vì đều liên quan tới "điều khiển hành vi", nhưng bản chất rất khác nhau:

| Tiêu chí | Hook | Skill | Rule trong CLAUDE.md |
|---|---|---|---|
| Ai thực thi? | Harness (hệ thống) | Model (AI) thực hiện quy trình | Model đọc rồi tự diễn giải |
| Tất định? | Có (deterministic) | Không (model tự quyết cách làm) | Không (chỉ là chỉ dẫn mềm) |
| Kích hoạt thế nào? | Tự động khi có event | Người dùng/Claude gọi qua `/command` | Tự nạp vào context |
| Dùng cho gì? | "Tự động mỗi khi X" — format, chặn lệnh, thông báo | Quy trình chuyên biệt tái dùng (code-review, security-review) | Quy ước, ngữ cảnh dự án, kiến trúc |

Cách phân biệt nhanh bằng câu hỏi:

- "Việc này có PHẢI luôn xảy ra một cách tự động và đảm bảo không?" → **Hook**. Vd format sau mỗi lần sửa file.
- "Đây có phải một quy trình/bộ kỹ năng mà tôi muốn gọi ra khi cần không?" → **Skill**. Vd `/security-review` chạy quy trình rà soát bảo mật.
- "Đây có phải kiến thức/quy ước nền mà model nên biết khi làm việc không?" → **Rule trong CLAUDE.md**. Vd "dự án dùng pnpm, kiến trúc theo feature folder".

```text
Tình huống: "Sau khi sửa code, luôn chạy prettier."
  -> HOOK (PostToolUse). Phải đảm bảo, model không được quên.

Tình huống: "Tôi muốn một quy trình review code theo checklist của team, gọi khi muốn."
  -> SKILL (định nghĩa trong SKILL.md, gọi bằng /code-review).

Tình huống: "Dự án này viết bằng Next.js App Router, dùng Tailwind, import theo alias @/."
  -> RULE trong CLAUDE.md. Là ngữ cảnh để model hiểu, không phải lệnh bắt buộc.
```

Điểm cốt lõi để trả lời phỏng vấn: Skill và rule đều dựa vào model *quyết định và thực hiện* — nên chúng linh hoạt nhưng không đảm bảo. Hook bỏ qua model hoàn toàn cho phần thực thi — nên nó kém linh hoạt nhưng đảm bảo tuyệt đối. Chọn công cụ theo việc bạn cần "linh hoạt thông minh" hay "đảm bảo chắc chắn".

Bẫy thường gặp: dùng Skill hoặc rule cho việc đáng ra phải là hook (vd nhờ một skill "luôn format"), rồi ngạc nhiên khi đôi lúc nó không chạy — vì model không phải lúc nào cũng gọi skill đó.

</details>

---

### 7.6. Khi dùng hook, cần lưu ý gì về an toàn (safety)?

<details>
<summary>👉 Xem câu trả lời</summary>

Hook là lệnh shell chạy tự động với quyền của bạn, không qua bước hỏi phép của permission mode — nên đây vừa là sức mạnh vừa là rủi ro. Một số lưu ý an toàn quan trọng:

1. **Hook chạy với quyền của bạn, tự động.** Khác với tool thông thường (mặc định Claude xin phép trước khi chạy lệnh), hook do harness tự chạy. Một hook viết ẩu (vd có `rm` sai chỗ) sẽ thực thi mà không hỏi. Hãy viết và kiểm thử hook cẩn thận như viết script production.

2. **Cẩn trọng với input không tin cậy.** Harness truyền dữ liệu (đường dẫn file, nội dung tool call) cho hook qua stdin dạng JSON (và một vài biến môi trường như `CLAUDE_PROJECT_DIR`). Nếu hook nội suy thẳng dữ liệu này vào lệnh shell, có nguy cơ command injection. Luôn trích dẫn biến (`"$file"`) và tránh `eval`.

3. **Hook là một con dao hai lưỡi cho bảo mật — và đó cũng là công dụng tốt.** Chính vì hook `PreToolUse` chạy *trước* và có thể *chặn* tool, nó là cơ chế lý tưởng để dựng rào chắn: chặn `rm -rf /`, chặn sửa file nhạy cảm (`.env`, secrets), chặn `git push --force`. Đây là cách dùng hook đúng đắn nhất về mặt an toàn.

4. **Đừng commit secret vào hook.** Không nhúng token/API key trực tiếp trong `command`. Dùng biến môi trường (cấu hình ở `env` trong settings, hoặc lấy từ secret manager).

5. **Phạm vi tin cậy của file cấu hình.** `settings.json` của project nằm trong repo — nếu bạn mở một repo lạ, hook trong đó có thể chạy lệnh tùy ý. Cảnh giác với hook đến từ nguồn không tin cậy, đúng như cảnh giác với script trong repo lạ.

```bash
# file lấy an toàn từ JSON stdin: file=$(echo "$input" | jq -r '.tool_input.file_path')

# AN TOÀN: trích dẫn biến, không eval, fail mềm để không chặn quy trình ngoài ý muốn
prettier --write "$file" 2>/dev/null || true

# NGUY HIỂM: nội suy thô + eval -> command injection nếu tên file độc hại
eval prettier --write $file
```

Bẫy thường gặp: viết hook `PostToolUse` rất "mạnh tay" (vd tự `git add -A && git commit`) khiến mọi thay đổi của Claude bị commit tự động — mất kiểm soát lịch sử git. Nguyên tắc: hook nên làm những việc an toàn, có thể đảo ngược (format, lint, thông báo); những hành động khó đảo ngược (commit, push, xóa) nên để người dùng chủ động, hoặc nếu dùng hook thì phải là `PreToolUse` để *chặn*, chứ không phải tự *thực hiện*.

</details>

---

### 7.7. Hãy kể về một lần bạn dùng hook (hoặc tự động hóa) để giải quyết một vấn đề lặp đi lặp lại trong quy trình làm việc.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Đây là câu hỏi hành vi (behavioral) — trả lời theo cấu trúc STAR (Situation, Task, Action, Result). Dưới đây là một khung gợi ý; bạn nên thay bằng trải nghiệm thật của mình.

**Situation (Bối cảnh):**
"Trong một dự án Next.js + TypeScript, team tôi áp chuẩn format bằng Prettier và lint bằng ESLint rất chặt. Khi dùng Claude Code để sửa code, AI sinh ra code đúng logic nhưng đôi khi không khớp 100% style của team, dẫn tới CI fail ở bước lint, hoặc tôi phải sửa thủ công lại — lặp đi lặp lại, rất mất thời gian."

**Task (Nhiệm vụ):**
"Tôi cần đảm bảo rằng MỌI file Claude sửa đều tự động được format và lint-fix ngay tại chỗ, một cách chắc chắn, mà không phải nhắc AI trong từng prompt và không phụ thuộc vào việc nó có nhớ hay không."

**Action (Hành động):**
"Tôi nhận ra đây là dạng yêu cầu 'tự động mỗi khi X' nên phải dùng hook chứ không phải ghi vào CLAUDE.md. Tôi cấu hình một hook `PostToolUse` trong `.claude/settings.json` của project, với matcher `Edit|Write`, gọi một script đọc đường dẫn file vừa sửa từ JSON stdin rồi chạy `prettier --write` và `eslint --fix` trên đúng file đó. Tôi để hook fail mềm (`|| true`) để một file lỗi lint không chặn cả quy trình. Vì cấu hình nằm ở cấp project và được commit, cả team cùng hưởng lợi. Phần thông báo riêng (báo khi Claude làm xong) tôi đặt ở hook `Stop` trong `settings.local.json` của riêng tôi để không ảnh hưởng người khác."

**Result (Kết quả):**
"Sau khi áp dụng, số lần CI fail vì lỗi format/lint từ code do AI sinh ra giảm gần như về 0. Tôi không còn phải nhắc AI 'nhớ format nhé' hay sửa style thủ công nữa. Quan trọng hơn, tôi rút ra nguyên tắc rõ ràng cho cả team: việc gì cần đảm bảo tất định thì đặt vào hook, việc gì là ngữ cảnh/quy ước thì để CLAUDE.md, việc gì là quy trình gọi-khi-cần thì làm thành Skill — và điều đó giúp chúng tôi dùng Claude Code có kỷ luật hơn nhiều."

Lưu ý khi trả lời: nhấn mạnh được *lý do chọn hook thay vì nhờ AI nhớ* (tính tất định) là điểm cộng lớn, vì nó cho thấy bạn hiểu đúng bản chất công cụ chứ không chỉ dùng theo kiểu thử-sai.

</details>
## 8. MCP Model Context Protocol

### 8.1. MCP (Model Context Protocol) là gì và vì sao nó quan trọng trong việc dùng AI để phát triển phần mềm?

<details>
<summary>👉 Xem câu trả lời</summary>

MCP (Model Context Protocol) là một chuẩn mở (open standard) do Anthropic giới thiệu, dùng để kết nối các ứng dụng AI với tool và dữ liệu bên ngoài một cách thống nhất. Bạn có thể hình dung MCP giống như "cổng USB-C cho AI": thay vì mỗi công cụ AI phải viết một bộ tích hợp riêng cho từng dịch vụ (GitHub, database, Jira...), tất cả nói chung một giao thức.

Vì sao quan trọng:

- **Giải quyết bài toán M×N tích hợp:** Trước MCP, nếu có M ứng dụng AI và N nguồn dữ liệu/tool thì cần tới M×N bộ tích hợp riêng. Với MCP, mỗi bên chỉ cần nói chuẩn MCP một lần (M+N), rồi cắm vào nhau tự do.
- **Tái sử dụng:** Một MCP server viết cho database có thể dùng được trong Claude Code, trong app desktop, hay bất kỳ MCP client nào khác.
- **Mở rộng năng lực agent:** Model như Claude vốn chỉ "biết" những gì có trong context. MCP cho phép agent thực sự đọc issue trên Linear, truy vấn database, hay lấy design từ Figma — biến nó từ "chatbot trả lời" thành "agent hành động trên hệ thống thật".

Liên hệ với Claude Code: khi bạn thêm một MCP server (ví dụ GitHub), Claude Code có thêm các tool mới để gọi trong vòng lặp agentic — đọc/sửa file, chạy lệnh, và giờ thêm cả "tạo pull request", "đọc issue"... tất cả qua cùng một cơ chế.

```text
KHÔNG có MCP: mỗi AI app tự viết tích hợp riêng -> M × N bộ code
   Claude Code ──┬── tích hợp GitHub riêng
                 ├── tích hợp Postgres riêng
                 └── tích hợp Linear riêng

CÓ MCP: chuẩn chung -> M + N
   Claude Code ──┐
   App desktop ──┼── (chuẩn MCP) ──┬── MCP server GitHub
   IDE khác    ──┘                 ├── MCP server Postgres
                                   └── MCP server Linear
```

Bẫy thường gặp: nghĩ MCP là một "tính năng của Claude" — không phải. MCP là một chuẩn mở, độc lập với model; Claude Code chỉ là một trong nhiều client biết nói chuẩn này.

</details>

---

### 8.2. Phân biệt MCP server và MCP client. Trong bối cảnh Claude Code, đâu là client, đâu là server?

<details>
<summary>👉 Xem câu trả lời</summary>

MCP theo mô hình client–server:

- **MCP server:** là bên *cung cấp* năng lực — nó expose ra các tool, resource và prompt để client gọi. Server thường bọc quanh một hệ thống cụ thể: một server cho GitHub, một server cho database, một server cho Figma... Server biết cách thực sự gọi API GitHub hay chạy SQL, và trả kết quả về theo chuẩn MCP.
- **MCP client:** là bên *sử dụng* năng lực — nó là ứng dụng AI nhúng model, kết nối tới một hoặc nhiều server, lấy danh sách tool và để model quyết định khi nào gọi tool nào.

Trong Claude Code:

- **Claude Code đóng vai client.** Nó kết nối tới các MCP server bạn đã cấu hình, gom các tool mà những server đó expose vào "hộp công cụ" của agent, rồi model Claude tự chọn gọi khi cần.
- **Server là các tiến trình/dịch vụ riêng** — có thể chạy local (một process trên máy bạn, giao tiếp qua stdio) hoặc remote (một dịch vụ qua HTTP/SSE).

```text
        ┌──────────────── Claude Code (MCP CLIENT) ────────────────┐
        │  model Claude + vòng lặp agentic + permission            │
        └───┬───────────────────┬───────────────────┬─────────────┘
            │ (chuẩn MCP)        │                   │
            ▼                    ▼                   ▼
   MCP server GitHub     MCP server Postgres   MCP server Figma
   (gọi GitHub API)      (chạy SQL)            (đọc design)
```

Một client có thể nối nhiều server cùng lúc; một server cũng có thể phục vụ nhiều client khác nhau. Điểm cần nhớ: model không trực tiếp "biết" GitHub — nó chỉ thấy các tool do server khai báo, và client là cầu nối truyền lời gọi tool xuống server rồi lấy kết quả về.

</details>

---

### 8.3. Trong MCP có ba loại "khả năng" mà server có thể expose: tools, resources và prompts. Giải thích sự khác nhau và khi nào dùng cái nào.

<details>
<summary>👉 Xem câu trả lời</summary>

MCP server có thể expose ba loại primitive, mỗi loại phục vụ một mục đích khác nhau:

| Primitive | Là gì | Ai điều khiển | Ví dụ |
|-----------|-------|---------------|-------|
| **Tools** | Hành động/hàm mà model có thể *gọi để làm việc gì đó* (thường có tác dụng phụ). | Model quyết định gọi khi cần (model-controlled). | `create_issue`, `run_query`, `send_message` |
| **Resources** | Dữ liệu/nội dung mà client có thể *đọc* để đưa vào context (chỉ đọc, như endpoint GET). | Ứng dụng/người dùng chọn nạp (app-controlled). | nội dung file, một bản ghi DB, một trang tài liệu |
| **Prompts** | Mẫu prompt/quy trình dựng sẵn, thường gắn với slash command hoặc lựa chọn của người dùng. | Người dùng kích hoạt (user-controlled). | "tóm tắt PR này", "review theo checklist X" |

Cách phân biệt nhanh:

- Nếu bạn muốn agent **thực hiện hành động** (ghi, gọi API, thay đổi trạng thái) → đó là **tool**.
- Nếu bạn muốn **cung cấp ngữ cảnh để đọc** mà không gây tác dụng phụ → đó là **resource**.
- Nếu bạn muốn **gói một quy trình/lời nhắc tái sử dụng** cho người dùng chọn → đó là **prompt**.

```text
Tools     -> "Hãy LÀM việc này"      (model gọi)      vd: create_issue(...)
Resources -> "Đây là dữ liệu để ĐỌC" (app nạp)        vd: read file://report.md
Prompts   -> "Đây là quy trình sẵn"  (user kích hoạt) vd: /summarize-pr
```

Lưu ý thực tế: trong nhiều client (gồm Claude Code), **tools** là phần được tận dụng nhiều nhất vì khớp tự nhiên với cơ chế tool use / function calling của model. Resources và prompts hữu ích nhưng mức độ hỗ trợ có thể khác nhau giữa các client — nên khi viết server, đừng giả định mọi client đều dùng đủ ba loại như nhau.

</details>

---

### 8.4. Hãy kể vài MCP server phổ biến và mô tả cách thêm một MCP server vào Claude Code.

<details>
<summary>👉 Xem câu trả lời</summary>

Một số MCP server phổ biến (mỗi cái bọc quanh một hệ thống thật):

- **GitHub** — đọc/tạo issue, pull request, duyệt repo.
- **Database** (ví dụ Postgres) — kiểm tra schema, chạy truy vấn.
- **Linear / Atlassian (Jira)** — đọc và cập nhật task, ticket.
- **Figma** — lấy thông tin design, component, layout.
- **Notion** — đọc/ghi tài liệu, trang, database nội dung.

Cách thêm vào Claude Code (ý tưởng chung — chi tiết cú pháp có thể thay đổi theo phiên bản, nên tra tài liệu hiện hành):

- **Bằng CLI:** dùng lệnh `claude mcp add` để đăng ký server. Với server local, bạn chỉ ra lệnh khởi chạy tiến trình server (giao tiếp qua stdio); với server remote, bạn chỉ ra URL của dịch vụ (HTTP/SSE).
- **Bằng file cấu hình:** khai báo server trong cấu hình (ví dụ trong settings / file cấu hình MCP của dự án), kèm lệnh chạy, tham số và biến môi trường như token xác thực.
- **Phạm vi (scope):** server có thể được cấu hình ở cấp user (dùng cho mọi dự án của bạn) hay cấp project (chia sẻ với cả team qua file trong repo).

```bash
# Ý tưởng: đăng ký một MCP server local chạy bằng một lệnh khởi chạy
claude mcp add ten-server -- <lenh-chay-server> [tham-so...]

# Sau khi thêm, kiểm tra danh sách server đã cấu hình
claude mcp list

# Trong phiên Claude Code, có thể xem trạng thái MCP qua slash command
/mcp
```

Sau khi server kết nối thành công, các tool nó expose sẽ xuất hiện trong hộp công cụ của agent (thường có tiền tố theo tên server để tránh trùng), và Claude tự gọi khi phù hợp — vẫn tuân theo permission mode (mặc định hỏi trước khi chạy).

Bẫy thường gặp: thêm server nhưng quên cung cấp token/biến môi trường xác thực, dẫn tới server kết nối được nhưng mọi lời gọi tool đều lỗi 401/403. Luôn kiểm tra trạng thái bằng `/mcp` hoặc `claude mcp list` trước khi nghĩ rằng "AI không chịu làm".

</details>

---

### 8.5. Khi cấp cho agent quyền truy cập hệ thống thật qua MCP, bạn lo ngại điều gì về bảo mật và xác thực? Làm sao giảm thiểu rủi ro?

<details>
<summary>👉 Xem câu trả lời</summary>

Cho agent kết nối hệ thống thật là con dao hai lưỡi: tiện lợi đi kèm bề mặt tấn công lớn hơn. Các mối lo chính:

- **Xác thực (authentication):** MCP server thường cần credential để gọi hệ thống đích — API token, OAuth, connection string database. Nếu lộ token (commit nhầm vào repo, log ra console) thì kẻ xấu có toàn quyền như chính bạn.
- **Phân quyền quá rộng (over-broad scope):** Cấp token có quyền ghi/xóa trong khi tác vụ chỉ cần đọc. Một lệnh sai của agent (hoặc do prompt injection) có thể xóa dữ liệu thật.
- **Prompt injection qua dữ liệu:** Nội dung agent đọc về (issue, trang web, bản ghi DB) có thể chứa chỉ thị độc hại khiến model gọi tool ngoài ý muốn — đây là rủi ro đặc thù của agent có tool.
- **Server không tin cậy:** Cài một MCP server lạ chẳng khác gì chạy phần mềm của bên thứ ba trên máy bạn với quyền truy cập dữ liệu.

Cách giảm thiểu:

- **Nguyên tắc đặc quyền tối thiểu (least privilege):** cấp token chỉ-đọc khi có thể; giới hạn scope OAuth; dùng database user quyền hạn chế.
- **Quản lý secret đúng cách:** đặt token trong biến môi trường / secret store, không hardcode; thêm vào `.gitignore`; ưu tiên OAuth có thể thu hồi thay vì token tĩnh sống mãi.
- **Tận dụng permission mode của Claude Code:** giữ chế độ hỏi trước cho tool có tác dụng phụ; **tránh `bypassPermissions` với MCP đụng tới hệ thống production.** Có thể dùng hook (PreToolUse) để chặn/kiểm duyệt lời gọi tool nguy hiểm một cách tự động (xác định, không phụ thuộc model nhớ).
- **Chỉ dùng server tin cậy** và đọc kỹ nó expose tool gì; cô lập bằng môi trường staging/sandbox khi thử nghiệm.

```text
Phân lớp phòng thủ khi dùng MCP:
  1) Token least-privilege (đọc-only nếu đủ)   <- giới hạn thiệt hại tối đa
  2) Secret trong env, không vào git           <- tránh rò rỉ credential
  3) Permission mode hỏi trước cho tool ghi     <- con người duyệt hành động
  4) Hook PreToolUse chặn lệnh/đối số nguy hiểm  <- chốt chặn tự động
  5) Chỉ cài server tin cậy, ưu tiên môi trường staging
```

Điểm nhấn: với hành vi "luôn chặn X" hay "luôn kiểm tra trước khi gọi tool", phải dùng **hook** (harness tự chạy, xác định) chứ không thể trông cậy model "tự nhớ" — model có thể bị injection hoặc đơn giản là quên.

</details>

---

### 8.6. Phân biệt MCP với Skill và Plugin trong Claude Code. Khi nào nên dùng cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Ba khái niệm này dễ lẫn vì đều "mở rộng" Claude Code, nhưng giải quyết bài toán khác nhau:

| | Bản chất | Giải quyết gì | Định nghĩa ở đâu |
|---|----------|---------------|------------------|
| **MCP** | Chuẩn mở kết nối AI với **tool/dữ liệu bên ngoài** qua server. | Cho agent *năng lực mới* để chạm hệ thống thật (GitHub, DB, Figma...). | MCP server (process/dịch vụ riêng), khai báo trong cấu hình. |
| **Skill** | Một "kỹ năng" tái sử dụng — gói **quy trình/kiến thức chuyên biệt** cho Claude. | Đóng gói *cách làm* một việc lặp lại (vd code-review, deep-research). | File `SKILL.md` có frontmatter (name, description); cấp user/project/built-in/plugin; gọi qua slash command. |
| **Plugin** | Gói phân phối **đóng gói nhiều thứ lại** — có thể chứa skill, slash command, agent, và cả cấu hình MCP server. | *Phân phối & cài đặt* một bộ năng lực cho team. | Gói plugin (đóng gói và chia sẻ). |

Cách chọn:

- Cần agent **truy cập một hệ thống bên ngoài** (đọc Jira, query DB, lấy design Figma)? → **MCP server**. Đây là chuyện "kết nối dữ liệu/tool", không phải chuyện "dạy quy trình".
- Cần agent **làm theo một quy trình chuyên biệt** dùng lại nhiều lần (review bảo mật, nghiên cứu sâu)? → **Skill**. Nó định hình *cách Claude làm*, dùng các tool sẵn có.
- Cần **đóng gói và chia sẻ** một bộ skill + command + (có thể cả) cấu hình MCP cho cả nhóm? → **Plugin**.

```text
MCP    = "kết nối ra ngoài"  (năng lực: chạm tool/dữ liệu thật)
Skill  = "biết cách làm"     (quy trình tái sử dụng, gọi qua slash command)
Plugin = "đóng gói để phát"  (bọc skill/command/agent/MCP để cài & chia sẻ)
```

Điểm hay ho: chúng bổ trợ nhau, không loại trừ. Ví dụ một plugin "Jira workflow" có thể vừa kèm cấu hình **MCP server Atlassian** (để chạm Jira thật) vừa kèm một **skill** mô tả quy trình "phân rã epic thành sub-task". Bẫy thường gặp: cố nhét logic kết nối hệ thống vào skill bằng cách bảo Claude "chạy curl" — kém ổn định và khó tái sử dụng; nếu là tích hợp hệ thống, dùng MCP server đúng nghĩa.

</details>

---

### 8.7. Kể về một lần bạn dùng (hoặc sẽ thiết kế việc dùng) MCP để biến AI thành "siêu năng lực" trong quy trình làm việc của team.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Câu này kiểm tra khả năng nhìn AI như một phần của hệ thống kỹ thuật (chứ không chỉ "hỏi đáp"), tư duy về tự động hóa quy trình, và ý thức bảo mật khi cho agent chạm hệ thống thật. Hãy nhấn mạnh giá trị mang lại và cách bạn kiểm soát rủi ro.

- **Situation:** "Team tôi tốn nhiều thời gian thủ công khi nhận một ticket: phải mở Jira đọc mô tả, đối chiếu code trong repo, rồi viết tay phần mô tả pull request. Bối cảnh nằm rải rác ở nhiều công cụ nên ai cũng phải nhảy qua lại."
- **Task:** "Tôi muốn rút ngắn vòng từ ticket đến PR draft, để mỗi developer tập trung vào phần suy nghĩ thực sự thay vì sao chép thông tin giữa các tab."
- **Action:** "Tôi cấu hình Claude Code (đóng vai MCP client) kết nối tới MCP server cho Atlassian/Jira và GitHub. Khi bắt đầu một task, agent đọc trực tiếp ticket qua tool của server Jira, đối chiếu với code trong repo, rồi soạn mô tả PR có dẫn chiếu đúng issue. Về bảo mật, tôi dùng token least-privilege (chỉ phạm vi cần thiết), để credential trong biến môi trường không commit vào repo, giữ permission mode hỏi trước cho các tool có tác dụng phụ (tạo/sửa PR), và thêm một hook PreToolUse để chặn các thao tác ghi ngoài ý muốn. Tôi cũng phân biệt rõ: phần *kết nối hệ thống* dùng MCP, còn *quy trình review* tôi đóng gói thành một skill riêng để cả team dùng lại."
- **Result:** "Thời gian từ 'nhận ticket' đến 'có PR draft kèm ngữ cảnh' giảm đáng kể, và mô tả PR đồng nhất hơn vì luôn bám theo nội dung ticket thật. Quan trọng là không lần nào agent thực hiện hành động ghi mà thiếu sự duyệt của con người, nhờ permission mode và hook. Cả nhóm bắt đầu coi đây là quy trình chuẩn."

**Điểm nhấn nên thể hiện:** hiểu vai trò client–server (Claude Code là client, Jira/GitHub là server); ghép đúng công cụ cho đúng việc (MCP cho kết nối, skill cho quy trình); và đặc biệt là kỷ luật bảo mật (least privilege, secret trong env, permission mode, hook) — cho thấy bạn trao quyền cho AI một cách *có kiểm soát*, không liều lĩnh.

</details>
## 9. Workflows và Điều phối Multi-Agent

### 9.1. Khi nào nên chuyển từ một agent đơn lẻ sang điều phối multi-agent (workflow)? Dấu hiệu nào cho thấy bài toán cần fan-out hoặc cô lập context?

<details>
<summary>👉 Xem câu trả lời</summary>

Mặc định nên giữ một agent đơn lẻ — nó đơn giản, dễ debug, ít tốn token hơn. Chỉ chuyển sang điều phối multi-agent (workflow) khi có những dấu hiệu rõ ràng sau:

- **Cần chạy song song (fan-out)**: nhiều việc độc lập, không phụ thuộc nhau, mà nếu làm tuần tự sẽ rất chậm. Ví dụ: tìm kiếm 5 chủ đề khác nhau cùng lúc, review một thay đổi từ nhiều góc độ độc lập.
- **Cần cô lập context**: một việc con sẽ "ngốn" rất nhiều token (đọc hàng chục file, log dài) mà phần lớn dữ liệu trung gian đó không cần quay về context chính. Tách ra subagent giúp context chính gọn gàng, chỉ nhận lại kết quả tóm tắt.
- **Việc lớn, nhiều giai đoạn rõ ràng**: có thể chia thành các bước tách bạch (research → design → implement → verify), mỗi bước phù hợp với một vai trò/agent riêng.
- **Cần độ tin cậy cao qua kiểm chứng chéo**: một agent sinh kết quả, một agent khác (context độc lập) kiểm tra — giảm thiên kiến tự xác nhận.

Bảng quyết định nhanh:

| Tình huống | Một agent | Multi-agent / workflow |
|---|---|---|
| Việc nhỏ, tuần tự, ngữ cảnh gọn | ✅ | ❌ (thừa) |
| Nhiều việc độc lập cần làm cùng lúc | ❌ (chậm) | ✅ (fan-out) |
| Một việc con đọc rất nhiều file/log | Context phình to | ✅ (cô lập context) |
| Cần kiểm chứng chéo, giảm sai sót | Khó tự bắt lỗi mình | ✅ (verify/judge) |

Bẫy thường gặp: vội vàng dựng multi-agent cho việc đơn giản. Mỗi subagent là một context riêng và thêm chi phí token điều phối — nếu việc tuần tự và nhỏ, một agent vẫn là lựa chọn tối ưu. Nguyên tắc trong phỏng vấn: **bắt đầu từ đơn giản nhất, chỉ nâng cấp lên workflow khi bài toán thực sự cần.**

</details>

---

### 9.2. Có những mẫu (pattern) điều phối multi-agent phổ biến nào? Mô tả ngắn gọn từng mẫu và khi nào dùng.

<details>
<summary>👉 Xem câu trả lời</summary>

Có bốn mẫu hay gặp khi điều phối nhiều agent. Mỗi mẫu giải quyết một dạng bài toán khác nhau:

- **Fan-out song song (parallel fan-out)**: orchestrator chia việc thành nhiều việc con độc lập, giao cho nhiều subagent chạy đồng thời, rồi gom kết quả lại. Dùng khi các việc con không phụ thuộc nhau.
- **Pipeline (đường ống tuần tự)**: kết quả của agent này là đầu vào của agent kế tiếp, theo các giai đoạn rõ ràng. Dùng khi có thứ tự phụ thuộc (research → draft → review).
- **Adversarial verify (kiểm chứng đối kháng)**: một agent tạo ra kết quả, một agent khác (context độc lập) cố tìm lỗi/phản biện. Dùng khi cần độ chính xác cao và muốn giảm thiên kiến tự xác nhận.
- **Judge panel (hội đồng chấm điểm)**: nhiều agent cùng đánh giá/chấm một kết quả theo tiêu chí, rồi tổng hợp (bỏ phiếu, lấy đồng thuận). Dùng khi việc đánh giá mang tính chủ quan hoặc dễ nhiễu, cần nhiều góc nhìn.

Bảng tóm tắt:

| Mẫu | Cấu trúc | Khi nào dùng |
|---|---|---|
| Fan-out song song | 1 → N chạy đồng thời → gom | Việc con độc lập, muốn nhanh |
| Pipeline | A → B → C tuần tự | Có thứ tự phụ thuộc rõ ràng |
| Adversarial verify | Tạo ↔ phản biện | Cần chính xác, giảm tự xác nhận |
| Judge panel | N chấm điểm → tổng hợp | Đánh giá chủ quan, cần đồng thuận |

```text
Fan-out:                 Pipeline:
   ┌─ subagent A ─┐         A ──> B ──> C
1 ─┼─ subagent B ─┼─> gom
   └─ subagent C ─┘      Judge panel:
                            ┌─ judge 1 ─┐
Adversarial verify:      X ─┼─ judge 2 ─┼─> tổng hợp
   Tạo ─> Phản biện         └─ judge 3 ─┘
    ^         |
    └─ sửa ───┘
```

Lưu ý: các mẫu có thể kết hợp. Ví dụ một pipeline mà ở bước review lại dùng judge panel; hoặc fan-out để research rồi pipeline để tổng hợp. Trong Claude Code, fan-out và cô lập context thường được thực hiện qua **subagents** (mỗi agent con có context riêng), còn điều phối phức tạp/tất định thì dùng **workflow** (script).

</details>

---

### 9.3. Chi phí token là yếu tố cần cân nhắc thế nào khi thiết kế workflow multi-agent? Vì sao multi-agent có thể đắt hơn nhiều so với một agent?

<details>
<summary>👉 Xem câu trả lời</summary>

Multi-agent đánh đổi **token (chi phí) lấy tốc độ và chất lượng**. Hiểu rõ nguồn phát sinh token là yếu tố quyết định khi thiết kế:

- **Mỗi subagent có context riêng**: nó phải được nạp lại system prompt, hướng dẫn, và phần ngữ cảnh cần thiết. N subagent nghĩa là N lần "khởi động" context — cộng dồn rất nhanh.
- **Lặp lại đầu vào ở nhiều agent**: trong fan-out hay judge panel, cùng một tài liệu/dữ liệu có thể bị gửi tới nhiều agent → nhân token đầu vào lên nhiều lần.
- **Vòng lặp agentic dài**: mỗi subagent tự chạy nhiều bước (đọc file, gọi tool) → token tích lũy trong từng agent.
- **Chi phí tổng hợp**: orchestrator phải đọc lại kết quả từ tất cả subagent để gom — đây cũng là token.

Các đòn bẩy giảm chi phí:

| Kỹ thuật | Tác dụng |
|---|---|
| Prompt caching | Phần đầu vào chung (tài liệu, system prompt) được cache, các lần sau chỉ ~0.1× giá |
| Cô lập context | Dữ liệu trung gian khổng lồ ở lại subagent, chỉ tóm tắt mới về context chính |
| Chọn model rẻ cho việc đơn giản | Dùng model nhẹ (kiểu Haiku) cho subagent làm việc đơn giản, model mạnh cho orchestrator |
| `effort` thấp cho subagent | Hạ độ "suy nghĩ" với việc dễ để tiết kiệm token |

Ví dụ minh họa về cân nhắc: một judge panel 5 agent cùng đọc một báo cáo 50K token. Nếu không cache, đó là ~250K token đầu vào chỉ để chấm điểm. Nếu báo cáo là phần prefix chung và được cache, 4 agent sau chỉ trả ~0.1× cho phần đó. Đây là lý do trong fan-out song song, kỹ thuật phổ biến là: gửi 1 request trước, đợi nó bắt đầu stream (ghi cache xong), rồi mới bắn N−1 request còn lại để chúng đọc từ cache.

Điểm cần nắm khi phỏng vấn: multi-agent **không miễn phí**. Trước khi dựng, hãy tự hỏi: giá trị (nhanh hơn / chính xác hơn) có xứng với chi phí token tăng thêm không? Đó là tư duy của người tối ưu có chủ đích.

</details>

---

### 9.4. Khi nào KHÔNG nên dùng workflow multi-agent? Những trường hợp một agent đơn lẻ vẫn là lựa chọn tốt hơn.

<details>
<summary>👉 Xem câu trả lời</summary>

Đây là câu hỏi quan trọng để thể hiện bạn không "lạm dụng" công cụ. Multi-agent là con dao hai lưỡi — nhiều trường hợp một agent đơn lẻ vẫn tốt hơn:

- **Việc nhỏ, một lần, tuần tự**: ví dụ "sửa một bug ở một file" — chia thành nhiều agent chỉ làm phức tạp hóa, thêm chi phí và độ trễ điều phối mà không có lợi ích.
- **Các bước phụ thuộc chặt chẽ, không song song được**: nếu bước B bắt buộc cần kết quả đầy đủ của bước A, "song song hóa" không giúp gì — chỉ là một pipeline mà bạn có thể để một agent tự chạy.
- **Khi chi phí token vượt giá trị**: nếu bài toán không đủ lớn/khó để biện minh cho token nhân lên, một agent là đủ.
- **Khi context không quá lớn**: nếu toàn bộ ngữ cảnh vẫn vừa và gọn, không cần cô lập context bằng subagent.
- **Khi cần đơn giản để debug/bảo trì**: nhiều agent = nhiều điểm có thể sai, khó truy vết hơn. Với việc cần độ minh bạch cao, một agent dễ kiểm soát hơn.

Bảng tự kiểm tra trước khi dựng workflow:

| Câu hỏi | Nếu "Không" → |
|---|---|
| Việc có thực sự nhiều bước, khó đặc tả trước không? | Giữ một agent |
| Giá trị có xứng chi phí token & độ trễ tăng thêm không? | Giữ một agent |
| Có việc con nào thật sự độc lập, song song được không? | Không cần fan-out |
| Lỗi có thể phát hiện & khôi phục được không? | Cẩn trọng với agent tự chủ cao |

Ví dụ thực tế về "đừng lạm dụng": muốn đổi tên một biến trong ba file. Một số người sẽ định dựng "agent tìm + agent sửa + agent verify". Thực ra một agent với một vòng lặp đơn giản (grep → edit → chạy test) là quá đủ — dựng multi-agent ở đây là over-engineering.

Nguyên tắc trong phỏng vấn: **luôn bắt đầu ở tầng đơn giản nhất đáp ứng được yêu cầu.** Single agent xử lý phần lớn các trường hợp; chỉ leo lên multi-agent/workflow khi bài toán thật sự đòi hỏi.

</details>

---

### 9.5. Cho một ví dụ workflow cụ thể cho "review nhiều chiều" và một cho "nghiên cứu (deep research)". Mô tả cách điều phối các agent.

<details>
<summary>👉 Xem câu trả lời</summary>

Hai ví dụ kinh điển cho thấy multi-agent phát huy giá trị:

**Ví dụ 1 — Review nhiều chiều (judge panel + fan-out):**

Thay vì một agent review tất cả mọi thứ (dễ bỏ sót do phải để ý quá nhiều khía cạnh), ta fan-out review thành nhiều agent chuyên biệt, mỗi agent một góc nhìn độc lập:

```text
                  ┌─ agent: tìm bug logic / correctness ─┐
Diff / PR ────────┼─ agent: kiểm tra bảo mật            ─┼──> orchestrator
                  ├─ agent: review hiệu năng/đơn giản hóa┤    tổng hợp + xếp
                  └─ agent: kiểm tra style/convention    ┘    hạng findings
```

- Mỗi agent có context riêng, tập trung sâu vào một khía cạnh → recall cao hơn.
- Orchestrator gom các findings, loại trùng lặp, xếp hạng theo mức độ nghiêm trọng.
- Có thể thêm một bước **verify** (adversarial): một agent khác kiểm tra lại các findings để lọc báo động giả.

**Ví dụ 2 — Deep research (fan-out → verify → tổng hợp):**

```text
1. Phân rã câu hỏi thành các nhánh tìm kiếm con
2. Fan-out: nhiều subagent tìm kiếm song song trên các nguồn khác nhau
3. Mỗi subagent đọc nguồn, trích dẫn, trả về phát hiện (context cô lập)
4. Adversarial verify: agent kiểm chứng các tuyên bố so với nguồn (chống bịa)
5. Tổng hợp: orchestrator viết báo cáo có trích dẫn từ các phát hiện đã kiểm chứng
```

- Bước fan-out tận dụng song song để nhanh và bao phủ rộng nguồn.
- Bước cô lập context giữ cho mỗi subagent xử lý nguồn dài mà không làm phình context chính.
- Bước adversarial verify là điểm mấu chốt: tách việc "tạo" khỏi việc "kiểm" giúp giảm thiên kiến tự xác nhận và bắt được tuyên bố thiếu căn cứ.

Liên hệ thực tế: trong Claude Code, mẫu deep research này chính là tinh thần của skill `deep-research` (fan-out tìm kiếm, đối chiếu nguồn để xác minh, tổng hợp báo cáo có trích dẫn); còn review nhiều chiều ăn khớp với tư duy của `/code-review` / `security-review`. Các agent con thường là **subagents** (vai trò Explore để tìm kiếm), được điều phối bởi một orchestrator hoặc một workflow script.

</details>

---

### 9.6. Vì sao điều phối tất định (deterministic, bằng script) lại hữu ích cho việc lớn? Khác gì với việc để một agent tự quyết mọi thứ?

<details>
<summary>👉 Xem câu trả lời</summary>

**Workflow** là điều phối nhiều agent một cách **tất định** — luồng được lập trình sẵn bằng script, thay vì để model tự ý quyết định trình tự. Đây là điểm khác biệt cốt lõi giữa "workflow" và "agent tự chủ".

So sánh:

| Tiêu chí | Agent tự chủ (model tự quyết) | Workflow tất định (script điều phối) |
|---|---|---|
| Quyết định luồng | Model chọn bước tiếp theo mỗi vòng | Lập trình viên định nghĩa các bước cố định |
| Khả năng tái lập | Mỗi lần chạy có thể khác nhau | Luồng giống nhau, dễ tái lập |
| Kiểm soát | Linh hoạt nhưng khó đoán | Chặt chẽ, đoán trước được |
| Phù hợp | Việc mở, khó đặc tả trước | Việc lớn, lặp lại, có quy trình rõ |

Vì sao tất định hữu ích cho việc lớn:

- **Tái lập được (reproducibility)**: cùng đầu vào → cùng luồng xử lý. Quan trọng cho pipeline production, CI, hay việc chạy định kỳ.
- **Kiểm soát chi phí & độ trễ**: biết trước sẽ gọi bao nhiêu agent, mỗi bước làm gì → ước lượng được token và thời gian.
- **Dễ debug & audit**: khi có lỗi, bạn biết chính xác bước nào sai vì luồng cố định, không phải "đoán" model đã làm gì.
- **Đặt rào chắn (guardrails) đúng chỗ**: chèn các bước kiểm tra, phê duyệt, hoặc gating ở những điểm xác định trong luồng.

Ví dụ minh họa: một quy trình "build tài liệu hằng đêm" gồm: thu thập dữ liệu → fan-out tóm tắt từng phần → tổng hợp → kiểm chứng → xuất file. Nếu để một agent tự quyết mỗi đêm, kết quả và chi phí có thể dao động khó lường. Nếu đóng gói thành workflow script, mỗi đêm chạy giống hệt, dễ giám sát, dễ sửa khi một bước hỏng.

```text
Agent tự chủ:        "Đây là mục tiêu, tự xoay sở" — linh hoạt, khó đoán
Workflow tất định:   bước1 -> bước2 -> (fan-out) -> tổng hợp -> verify
                     (các bước do script điều phối, model làm phần "thông minh" trong mỗi bước)
```

Điểm cần nắm khi phỏng vấn: tất định và tự chủ không loại trừ nhau. Workflow **đóng khung** luồng tổng thể (tất định), còn trong mỗi bước, model vẫn làm phần việc "thông minh" của nó. Tư duy đúng: dùng workflow tất định cho khung quy trình lớn cần độ tin cậy, để agent tự chủ cho những việc mở, sáng tạo, khó đặc tả trước.

</details>
## 10. Claude API và Xây dựng tính năng AI

### 10.1. Các model Claude (Opus / Sonnet / Haiku / Fable) khác nhau ra sao, và bạn chọn model theo tiêu chí gì?

<details>
<summary>👉 Xem câu trả lời</summary>

Họ model Claude được phân tầng theo cân bằng giữa **trí thông minh — tốc độ — chi phí**. Mỗi model có một `model id` chính xác (string) mà bạn truyền vào tham số `model` khi gọi API; phải dùng đúng string, không được tự bịa hay thêm hậu tố ngày tháng.

Bảng so sánh các model hiện hành (giá tính theo USD trên 1 triệu token):

| Model | Model ID | Context | Input $/1M | Output $/1M | Dùng khi |
|---|---|---|---|---|---|
| Claude Fable 5 | `claude-fable-5` | 1M | $10 | $50 | Model mạnh nhất, cho reasoning khó và agentic dài hơi |
| Claude Opus 4.8 | `claude-opus-4-8` | 1M | $5 | $25 | Mặc định cho phần lớn việc cần "thông minh" |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 1M | $3 | $15 | Cân bằng tốc độ / trí tuệ cho production volume cao |
| Claude Haiku 4.5 | `claude-haiku-4-5` | 200K | $1 | $5 | Tác vụ đơn giản, nhạy về tốc độ (phân loại, trích xuất ngắn) |

Tiêu chí chọn model:

- **Độ khó tác vụ**: classification, tóm tắt ngắn, trích xuất đơn giản → Haiku; reasoning nhiều bước, coding, agentic → Opus 4.8 hoặc Fable 5.
- **Chi phí và volume**: workload production lớn, lặp lại nhiều → cân nhắc Sonnet hoặc Haiku để giảm chi phí. Lưu ý: việc "hạ cấp model để tiết kiệm" là quyết định của business, không nên tự ý làm.
- **Context window**: nếu phải nhồi nhiều dữ liệu (codebase lớn, nhiều tài liệu) thì cần model có context lớn (1M token với Opus/Sonnet/Fable).
- **Latency**: tác vụ interactive (chat real-time) ưu tiên model nhanh (Haiku/Sonnet); tác vụ batch/nền có thể dùng model mạnh hơn.

Một mẹo hay trong kiến trúc thực tế: **dùng nhiều model trong cùng một hệ thống** — Haiku cho các sub-task rẻ (ví dụ tìm kiếm, phân loại), Opus cho bước reasoning chính. Đây là pattern "model routing" giúp tối ưu chi phí.

Bẫy thường gặp: dùng `model id` sai (ví dụ `claude-sonnet-4.6` thay vì `claude-sonnet-4-6`, hoặc thêm hậu tố ngày) → lỗi `404 not_found_error`.

</details>

---

### 10.2. Tool use (function calling) là gì? Mô tả vòng đời một lượt gọi tool.

<details>
<summary>👉 Xem câu trả lời</summary>

**Tool use** (hay function calling) là cơ chế để model gọi ra các "công cụ" mà bạn định nghĩa — không phải model tự thực thi, mà nó **yêu cầu** bạn chạy hàm rồi trả kết quả về. Đây là nền tảng của mọi agent và workflow.

Một tool được định nghĩa bằng `name`, `description`, và `input_schema` (JSON Schema):

```ts
const tools = [{
  name: "get_weather",
  description: "Lấy thời tiết hiện tại của một thành phố. Gọi khi người dùng hỏi về thời tiết.",
  input_schema: {
    type: "object",
    properties: {
      location: { type: "string", description: "Tên thành phố, vd: Hà Nội" },
    },
    required: ["location"],
  },
}];
```

Vòng đời một lượt (đây chính là "vòng lặp agentic" ở cấp API):

1. Bạn gửi request kèm `tools` + câu hỏi của user.
2. Model trả về `stop_reason: "tool_use"` cùng một block `tool_use` chứa `name`, `input` (đã parse), và `id`.
3. **Bạn** (harness) thực thi hàm thật với `input` đó.
4. Bạn gửi lại kết quả trong một message `role: "user"` với block `tool_result` (kèm đúng `tool_use_id` khớp với block ở bước 2).
5. Model dùng kết quả để trả lời tiếp, hoặc gọi tiếp tool khác. Lặp lại tới khi `stop_reason: "end_turn"`.

```python
while True:
    resp = client.messages.create(model="claude-opus-4-8", max_tokens=16000,
                                   tools=tools, messages=messages)
    if resp.stop_reason == "end_turn":
        break
    messages.append({"role": "assistant", "content": resp.content})
    results = []
    for b in resp.content:
        if b.type == "tool_use":
            out = run_tool(b.name, b.input)        # bạn chạy hàm thật
            results.append({"type": "tool_result",
                            "tool_use_id": b.id, "content": out})
    messages.append({"role": "user", "content": results})
```

`tool_choice` điều khiển hành vi: `auto` (model tự quyết, mặc định), `any` (buộc dùng ít nhất một tool), `{type: "tool", name: ...}` (buộc dùng đúng tool đó), `none` (cấm dùng tool).

Các bẫy thường gặp:
- **Thiếu `tool_result` cho một `tool_use_id`** → API báo lỗi 400. Mỗi block `tool_use` phải có đúng một `tool_result` tương ứng.
- **Raw-string-match trên input đã serialize**: phải parse bằng `JSON.parse()`/`json.loads()`, vì model có thể escape Unicode/forward-slash khác nhau.
- **`description` mô tả sờ sài** → model gọi sai hoặc không gọi. Nên mô tả rõ "*khi nào* gọi", không chỉ "*làm gì*".

SDK còn có **tool runner** (beta) tự lo vòng lặp này cho bạn — bạn chỉ cần khai báo hàm; dùng vòng lặp thủ công khi cần kiểm soát chi tiết (logging, human-in-the-loop, duyệt trước khi chạy).

</details>

---

### 10.3. Prompt caching là gì? Nó giúp giảm chi phí và độ trễ như thế nào, và cái gì dễ làm hỏng cache?

<details>
<summary>👉 Xem câu trả lời</summary>

**Prompt caching** cho phép cache phần đầu (prefix) ổn định của prompt để các request sau không phải xử lý lại — tiết kiệm tới ~90% chi phí cho phần được cache và giảm đáng kể độ trễ.

Nguyên tắc bất biến cốt lõi: **caching là so khớp prefix (prefix match)**. Cache key dựa trên đúng từng byte của prompt đã render tới điểm `cache_control`. Chỉ cần một byte thay đổi ở vị trí N là cache của mọi thứ từ vị trí N trở đi bị vô hiệu. Thứ tự render là: `tools` → `system` → `messages`.

Đặt `cache_control` ở cuối phần ổn định:

```python
response = client.messages.create(
    model="claude-opus-4-8",
    max_tokens=16000,
    system=[{
        "type": "text",
        "text": tai_lieu_lon,                       # nội dung dùng lại nhiều lần
        "cache_control": {"type": "ephemeral"},      # TTL mặc định 5 phút; "1h" cho 1 giờ
    }],
    messages=[{"role": "user", "content": "Tóm tắt các ý chính"}],
)
```

Kiểm chứng cache có "ăn" không bằng trường `usage`:

| Trường | Ý nghĩa |
|---|---|
| `cache_creation_input_tokens` | Token ghi vào cache (trả phí ~1.25x) |
| `cache_read_input_tokens` | Token đọc từ cache (trả phí ~0.1x) |
| `input_tokens` | Token tính full giá (không cache) |

Kinh tế học: đọc cache ~0.1x giá gốc, ghi cache ~1.25x (TTL 5 phút) hoặc 2x (TTL 1 giờ). Với TTL 5 phút, chỉ cần 2 request là hòa vốn.

Những thứ **âm thầm phá cache** (silent invalidators) — cần soi kỹ khi review:
- `datetime.now()` / `Date.now()` / UUID nhét vào system prompt → prefix đổi mỗi request.
- `json.dumps()` không `sort_keys=True`, hoặc duyệt `set` → byte serialize khác nhau.
- Nhét session/user id vào system prompt → mỗi user một prefix, không share được.
- **Đổi danh sách tool hoặc đổi model giữa chừng** → tool render ở vị trí 0, đổi tool phá toàn bộ cache; cache cũng gắn theo model.

Dấu hiệu nhận biết: nếu `cache_read_input_tokens` luôn = 0 dù các request có prefix giống hệt, chắc chắn có một silent invalidator đang hoạt động — diff byte prompt giữa hai request để tìm.

</details>

---

### 10.4. Khi nào dùng RAG, khi nào dùng long context, khi nào fine-tune?

<details>
<summary>👉 Xem câu trả lời</summary>

Ba cách "đưa kiến thức riêng" cho model, mỗi cách hợp với một bài toán khác nhau:

| Cách tiếp cận | Cơ chế | Hợp khi |
|---|---|---|
| **Long context** | Nhồi thẳng tài liệu vào prompt | Tập dữ liệu vừa phải, ổn định, dùng lại nhiều lần (kết hợp prompt caching) |
| **RAG** (Retrieval-Augmented Generation) | Tìm các đoạn liên quan rồi mới nhồi vào prompt | Kho dữ liệu lớn, thay đổi thường xuyên, mỗi câu hỏi chỉ cần một phần nhỏ |
| **Fine-tune** | Huấn luyện lại trọng số model | Cần dạy *phong cách/định dạng/hành vi*, không phải dạy *sự kiện* |

Cách tư duy chọn:

- **Long context**: với context window 1M token (Opus/Sonnet/Fable), bạn có thể nhồi cả một cuốn tài liệu vào. Đơn giản nhất, không cần hạ tầng. Kết hợp **prompt caching** thì chi phí lặp lại rất rẻ. Hợp khi dữ liệu không quá lớn và không đổi liên tục — ví dụ: hỏi đáp trên một hợp đồng, một file tài liệu.
- **RAG**: khi kho dữ liệu quá lớn để nhồi hết (hàng nghìn tài liệu) hoặc đổi liên tục (knowledge base nội bộ cập nhật hằng ngày). Bạn dùng **embeddings** để tìm các đoạn liên quan nhất với câu hỏi, rồi chỉ đưa các đoạn đó vào prompt. Vừa tiết kiệm token vừa cập nhật được dữ liệu mới mà không phải train lại.
- **Fine-tune**: ít dùng nhất, và **không phải** để nhồi kiến thức (RAG/long context làm việc đó tốt hơn và rẻ hơn). Fine-tune hợp khi bạn cần model bám một *phong cách trả lời* hoặc *định dạng output* rất đặc thù mà prompt không đủ, hoặc cần giảm latency/độ dài prompt cho một tác vụ lặp đi lặp lại ổn định.

Quy tắc thực dụng: **bắt đầu bằng long context + prompt caching** (đơn giản nhất). Khi dữ liệu vượt context hoặc đổi liên tục → chuyển sang **RAG**. Chỉ fine-tune khi đã thử hai cách kia mà vẫn thiếu về hành vi/phong cách. Cũng có thể kết hợp: RAG + long context (tìm đoạn liên quan rồi nhồi cả block lớn vào context có cache).

Bẫy hay gặp: dùng fine-tune để "nhét kiến thức công ty" — vừa tốn kém vừa khó cập nhật; dữ liệu mới lại phải train lại. RAG mới là câu trả lời cho bài toán "kiến thức thay đổi".

</details>

---

### 10.5. Xây một agent bằng Claude API như thế nào? Vòng lặp tool hoạt động ra sao và khi nào *nên* xây agent?

<details>
<summary>👉 Xem câu trả lời</summary>

Một **agent** ở cấp API về bản chất là một vòng lặp tool use mà *model tự quyết quỹ đạo của nó*: nó gọi tool, đọc kết quả, suy nghĩ, gọi tool tiếp, cho tới khi xong việc. Khác với **workflow** (bạn lập trình tất định các bước, model chỉ làm một mắt xích), agent để model tự lái.

Vòng lặp agent thủ công (cho phép kiểm soát chi tiết: duyệt tool, logging, dừng có điều kiện):

```python
messages = [{"role": "user", "content": yeu_cau}]
while True:
    resp = client.messages.create(model="claude-opus-4-8", max_tokens=16000,
                                   tools=tools, messages=messages)
    if resp.stop_reason == "end_turn":
        break
    if resp.stop_reason == "pause_turn":           # server tool đạt giới hạn vòng lặp
        messages.append({"role": "assistant", "content": resp.content})
        continue                                    # gửi lại để server chạy tiếp
    messages.append({"role": "assistant", "content": resp.content})
    tool_results = []
    for b in resp.content:
        if b.type == "tool_use":
            tool_results.append({"type": "tool_result",
                                 "tool_use_id": b.id,
                                 "content": execute_tool(b.name, b.input)})
    messages.append({"role": "user", "content": tool_results})
```

API là **stateless** — mỗi lần gọi bạn phải gửi lại toàn bộ lịch sử hội thoại (`messages`). Vì thế phải luôn append nguyên `resp.content` (giữ cả block `tool_use`, `thinking`) để giữ tính nhất quán.

Trước khi quyết định xây agent, kiểm tra 4 tiêu chí:
- **Độ phức tạp**: tác vụ nhiều bước, khó đặc tả trước hết? (ví dụ "biến design doc thành PR" vs "trích tiêu đề PDF")
- **Giá trị**: kết quả có đáng với chi phí + độ trễ cao hơn không?
- **Khả thi**: model có giỏi loại tác vụ này không?
- **Chi phí của lỗi**: lỗi có bắt và phục hồi được không (test, review, rollback)?

Nếu trả lời "không" cho bất kỳ tiêu chí nào → dùng tầng đơn giản hơn (một lần gọi API, hoặc workflow tất định). Nguyên tắc vàng: **bắt đầu đơn giản**, chỉ leo lên agent khi tác vụ thật sự cần khám phá mở do model tự lái.

Mẹo cho agent dài hơi: bật **adaptive thinking** (`thinking: {type: "adaptive"}`) để model tự quyết suy nghĩ bao nhiêu; dùng tham số `effort` (`low`/`medium`/`high`/`max`) để cân chi phí — `high`/`xhigh` cho coding/agentic, `low` cho sub-task đơn giản.

</details>

---

### 10.6. Streaming là gì và tại sao gần như luôn nên dùng cho output dài?

<details>
<summary>👉 Xem câu trả lời</summary>

**Streaming** trả về phản hồi theo từng phần (token/chunk) qua **Server-Sent Events (SSE)** thay vì chờ toàn bộ rồi mới trả. Lợi ích kép: (1) hiển thị chữ ngay cho người dùng (UX tốt cho chat), và (2) **tránh timeout HTTP** với request có output lớn — đây là lý do *kỹ thuật* quan trọng.

```python
with client.messages.stream(
    model="claude-opus-4-8",
    max_tokens=64000,
    messages=[{"role": "user", "content": "Viết một câu chuyện"}],
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
    final = stream.get_final_message()    # lấy message hoàn chỉnh sau khi stream xong
```

Các loại event trong stream (cần nắm khi xử lý thủ công):

| Event | Khi nào bắn |
|---|---|
| `message_start` | Một lần ở đầu, chứa metadata |
| `content_block_start` | Khi một block (text/thinking/tool_use) bắt đầu |
| `content_block_delta` | Mỗi chunk nội dung |
| `content_block_stop` | Khi block kết thúc |
| `message_delta` | Chứa `stop_reason` và `usage` |
| `message_stop` | Một lần ở cuối |

Vì sao "gần như luôn dùng" cho output dài: SDK sẽ **báo lỗi `ValueError`** nếu bạn gọi non-streaming với `max_tokens` ước tính vượt ~10 phút xử lý (kết nối idle sẽ rớt). Các model như Opus 4.8 hỗ trợ tới 128K output token, nhưng **bắt buộc streaming** để tránh timeout. Quy tắc đặt `max_tokens`: ~16000 cho non-streaming, ~64000 cho streaming.

Mẹo hay: ngay cả khi không cần hiển thị real-time, vẫn nên stream rồi gọi `.get_final_message()` / `.finalMessage()` để có message hoàn chỉnh mà vẫn được bảo vệ khỏi timeout. Đừng tự bọc các event `.on()` trong `new Promise()` — dùng helper của SDK.

Một lưu ý về thinking khi stream: trên Opus 4.8/4.7/Fable 5, mặc định `display` là `"omitted"` (block thinking rỗng), nên người dùng sẽ thấy một khoảng "đứng im" trước khi text bắt đầu. Nếu muốn hiển thị tiến trình suy nghĩ, đặt `thinking: {type: "adaptive", display: "summarized"}`.

</details>

---

### 10.7. Structured output là gì? Làm sao buộc model trả về JSON đúng schema?

<details>
<summary>👉 Xem câu trả lời</summary>

**Structured output** ràng buộc phản hồi của model theo một JSON Schema cụ thể, đảm bảo output luôn parse được — cực hữu ích khi tích hợp vào pipeline cần dữ liệu có cấu trúc (trích xuất, phân loại, gọi DB).

Có hai cơ chế:

1. **JSON output** qua `output_config.format` — ràng buộc *định dạng phản hồi* của model.
2. **Strict tool use** (`strict: true`) — đảm bảo *tham số tool* khớp schema.

Cách khuyên dùng trong SDK là `messages.parse()` với một model schema (Pydantic ở Python, Zod ở TS) — nó tự validate response cho bạn:

```python
from pydantic import BaseModel

class ContactInfo(BaseModel):
    name: str
    email: str
    plan: str
    demo_requested: bool

resp = client.messages.parse(
    model="claude-opus-4-8",
    max_tokens=16000,
    messages=[{"role": "user",
               "content": "Trích: Jane (jane@co.com) muốn gói Enterprise, có yêu cầu demo."}],
    output_format=ContactInfo,
)
contact = resp.parsed_output      # đã là instance ContactInfo đã validate
print(contact.name)               # "Jane"
```

Hoặc dùng raw schema trực tiếp:

```python
output_config={
    "format": {
        "type": "json_schema",
        "schema": {
            "type": "object",
            "properties": {"name": {"type": "string"}, "email": {"type": "string"}},
            "required": ["name", "email"],
            "additionalProperties": False,
        },
    }
}
```

Những điều cần lưu ý:
- **`output_config.format`** là tham số chuẩn hiện hành; tham số cũ `output_format` ở cấp `messages.create()` đã bị deprecated.
- Lần đầu dùng một schema mới có **độ trễ biên dịch** một lần; các lần sau dùng cache 24 giờ.
- Một số ràng buộc JSON Schema **không hỗ trợ**: schema đệ quy, `minimum`/`maximum`, `minLength`/`maxLength`. `additionalProperties` phải đặt `false`.
- Không tương thích với **citations** (lỗi 400) và với **prefill**.
- Nếu `stop_reason: "refusal"` (model từ chối) hoặc `"max_tokens"` (cụt giữa chừng), output có thể *không* khớp schema — phải kiểm tra `stop_reason` trước.

So với cách cũ là "prefill" (nhét sẵn `{"name": "` vào lượt assistant): trên các model 4.6/4.7/4.8 và Fable 5, prefill ở lượt assistant cuối **trả về lỗi 400** — structured output chính là cách thay thế đúng.

</details>

---

### 10.8. Cửa sổ ngữ cảnh (context window) và đếm token: tại sao quan trọng và đếm thế nào cho đúng?

<details>
<summary>👉 Xem câu trả lời</summary>

**Context window** là tổng số token tối đa cho một request — bao gồm cả input (system + tools + toàn bộ lịch sử messages) lẫn output. Các model Opus/Sonnet/Fable hiện có cửa sổ **1M token**, Haiku 4.5 là 200K. Vượt cửa sổ → model trả `stop_reason: "model_context_window_exceeded"` (khác với `"max_tokens"` là chạm trần *output* bạn đặt).

Vì sao quan trọng:
- **Chi phí** tính theo token (input rẻ hơn output nhiều — ví dụ Opus 4.8: $5 input vs $25 output trên 1M).
- **Hội thoại dài / agent dài hơi** sẽ phình lịch sử và có thể vượt cửa sổ.
- Phải ước lượng được token để đặt `max_tokens` hợp lý và dự toán chi phí.

Đếm token cho đúng: dùng endpoint **`count_tokens`** (`POST /v1/messages/count_tokens`), **không** dùng `tiktoken`.

```python
resp = client.messages.count_tokens(
    model="claude-opus-4-8",
    messages=messages,
    system=system,
)
print(resp.input_tokens)
```

Bẫy cực phổ biến: **dùng `tiktoken`** (tokenizer của OpenAI) để ước lượng token Claude — nó đếm thiếu ~15–20% với text thường, và sai nhiều hơn với code hay tiếng không phải Anh. Token là **đặc thù theo model**, nên phải truyền đúng `model` khi đếm.

Khi hội thoại quá dài, có vài chiến lược quản lý context:
- **Compaction** (beta): API tự tóm tắt phần đầu khi gần chạm trần. Quan trọng: phải append cả `response.content` (gồm block `compaction`) ngược lại, không chỉ phần text.
- **Context editing**: dọn các tool result / thinking cũ đã hết liên quan.
- **Prompt caching**: không giảm token nhưng giảm *chi phí* xử lý lại phần prefix ổn định.

Mẹo thực tế: gọi `count_tokens` *trước* khi gửi để ước lượng chi phí và phát hiện sớm prompt quá dài. Tuyệt đối không "cắt bớt" (truncate) input một cách âm thầm khi vượt cửa sổ — thay vào đó báo cho người dùng và bàn phương án (chunking, tóm tắt, RAG).

</details>

---

### 10.9. Embeddings là gì và vai trò của nó trong RAG?

<details>
<summary>👉 Xem câu trả lời</summary>

**Embeddings** là cách biểu diễn một đoạn văn bản thành một **vector số** (mảng số thực) sao cho các đoạn có *ý nghĩa gần nhau* thì vector cũng gần nhau trong không gian. Nhờ đó ta có thể tìm kiếm theo *ngữ nghĩa* (semantic search) thay vì khớp từ khóa.

Vai trò trong **RAG**: đây là bước "Retrieval" (truy hồi).

Pipeline RAG điển hình:

| Bước | Việc làm |
|---|---|
| 1. Indexing (offline) | Chia kho tài liệu thành chunk → tạo embedding cho từng chunk → lưu vào vector database |
| 2. Query | Người dùng hỏi → tạo embedding cho câu hỏi |
| 3. Retrieval | Tìm các chunk có vector gần nhất với vector câu hỏi (vd cosine similarity) |
| 4. Augment + Generate | Nhồi các chunk liên quan vào prompt → gọi model để trả lời |

Vì sao cần embeddings cho RAG: khi kho dữ liệu quá lớn để nhồi hết vào context, ta chỉ muốn đưa *các đoạn liên quan nhất* với câu hỏi. Tìm kiếm bằng từ khóa thuần (keyword match) bỏ sót khi câu hỏi và tài liệu dùng từ khác nhau nhưng cùng ý ("ô tô" vs "xe hơi"); semantic search qua embeddings bắt được sự tương đồng về *nghĩa*.

Lưu ý thực tế:
- Embedding và model sinh text (generation) là **hai bước riêng**: bạn dùng một model embedding để tạo vector, rồi mới dùng Claude để sinh câu trả lời từ các đoạn truy hồi được.
- Embeddings tự nó không "trả lời" — nó chỉ phục vụ *tìm kiếm*. Phần sinh câu trả lời vẫn là model ngôn ngữ.
- Chất lượng RAG phụ thuộc nhiều vào cách **chunking** (chia đoạn) và chất lượng retrieval, không chỉ ở model. Nếu retrieval đưa nhầm đoạn, model dù mạnh cũng trả lời sai (rác vào, rác ra).

Bẫy hay gặp: tưởng RAG "thông minh" sẵn — thực ra phần lớn lỗi RAG nằm ở khâu retrieval (chunk quá to/quá nhỏ, embedding không hợp domain), chứ không phải ở model sinh text.

</details>

---

### 10.10. Đánh giá (eval) tính năng AI: làm sao đo chất lượng, và "LLM-as-judge" là gì?

<details>
<summary>👉 Xem câu trả lời</summary>

Khi xây tính năng AI, output không tất định (cùng input có thể ra output khác nhau), nên bạn **không thể test bằng assert đúng/sai tuyệt đối** như code thường. Cần một bộ **eval** (đánh giá) để đo chất lượng một cách có hệ thống — đặc biệt khi nâng cấp model hoặc đổi prompt.

Các loại eval, từ rẻ tới đắt:

| Loại | Cơ chế | Hợp khi |
|---|---|---|
| **Exact / rule-based** | So khớp chính xác hoặc theo luật (regex, có chứa keyword) | Output có đáp án rõ: phân loại, trích xuất field, có/không |
| **Programmatic** | Chạy code kiểm tra (vd code sinh ra có compile/pass test không) | Tác vụ code, output JSON đúng schema |
| **LLM-as-judge** | Dùng một model khác để *chấm điểm* output theo rubric | Tác vụ mở: chất lượng văn bản, độ hữu ích, tính chính xác lập luận |
| **Human eval** | Người chấm | Chuẩn vàng, nhưng đắt và chậm — dùng để hiệu chỉnh các loại trên |

**LLM-as-judge** là kỹ thuật dùng chính một LLM làm "giám khảo": bạn cho nó output cần đánh giá + một **rubric** (tiêu chí rõ ràng), nó chấm điểm/xếp loại. Hữu ích cho các tác vụ mở mà không có đáp án "đúng" duy nhất.

```python
prompt = f"""Bạn là giám khảo. Chấm câu trả lời theo rubric.
Câu hỏi: {cau_hoi}
Câu trả lời: {cau_tra_loi}
Rubric: trả về JSON {{"diem": 1-5, "ly_do": "..."}} — 5 là chính xác và đầy đủ."""
resp = client.messages.create(model="claude-opus-4-8", max_tokens=1024,
                              output_config={"format": {"type": "json_schema",
                                                        "schema": rubric_schema}},
                              messages=[{"role": "user", "content": prompt}])
```

Nguyên tắc viết rubric tốt: dùng tiêu chí **rõ ràng và chấm được độc lập** ("CSV có cột `price` kiểu số"), tránh tiêu chí mơ hồ ("trông ổn"). Tiêu chí mơ hồ làm điểm chấm nhiễu.

Lưu ý/bẫy:
- LLM-as-judge **không thay thế hoàn toàn** human eval — nên hiệu chỉnh giám khảo bằng một tập mẫu đã có người chấm để biết nó đáng tin tới đâu.
- Nên **tách bước chấm khỏi bước sinh**: dùng context window riêng (giám khảo không thấy "quá trình suy nghĩ" của model tạo output), kết quả khách quan hơn.
- Eval phải chạy trên một **tập cố định** để so sánh được giữa các phiên bản — đổi prompt/model thì chạy lại cùng tập để biết tốt lên hay xấu đi.

Trong hệ Claude API/agent, khái niệm này còn xuất hiện ở dạng "outcome/rubric" — bạn nêu "done" trông thế nào, hệ thống lặp sinh → chấm → sửa cho tới khi đạt rubric.

</details>

---

### 10.11. Guardrails và an toàn (safety): khi xây tính năng AI cho người dùng, bạn xử lý từ chối, prompt injection và rò rỉ bí mật ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

**Guardrails** là các lớp bảo vệ quanh tính năng AI để đảm bảo an toàn, đúng phạm vi, và không bị lạm dụng. Có vài khía cạnh chính:

**1. Xử lý `stop_reason: "refusal"`.** Model có thể từ chối vì lý do an toàn — API trả về HTTP 200 (không phải lỗi) với `stop_reason: "refusal"` và một object `stop_details` chứa category. Code phải **kiểm tra `stop_reason` trước khi đọc `content`**, nếu không sẽ lỗi index khi `content` rỗng:

```python
resp = client.messages.create(model="claude-opus-4-8", max_tokens=1024, messages=msgs)
if resp.stop_reason == "refusal":
    handle_refusal(resp.stop_details)        # đừng đọc resp.content[0] một cách mù quáng
else:
    print(resp.content[0].text)
```

Với Fable 5, có thêm cơ chế **fallbacks** (opt-in): nếu request bị từ chối, API tự chạy lại trên một model dự phòng (vd `claude-opus-4-8`) trong cùng lượt gọi — bạn nên bật mặc định cho code Fable 5.

**2. Chống prompt injection.** Đây là rủi ro lớn nhất khi cho phép dữ liệu *không tin cậy* (nội dung web, file người dùng, kết quả tool) vào prompt — kẻ tấn công nhét "lệnh" ẩn để chiếm quyền điều khiển agent. Cách phòng:
- **Phân biệt rõ kênh chỉ thị (instruction) và kênh dữ liệu (data)**: lệnh của bạn nên ở `system`, dữ liệu không tin cậy chỉ là *nội dung để xử lý*. Có kênh `role: "system"` giữa hội thoại (beta) là kênh operator không giả mạo được — an toàn hơn nhồi lệnh vào text user.
- **Promote hành động nguy hiểm thành tool có gate**: hành động khó đảo ngược (xóa dữ liệu, gửi mail, gọi API ngoài) nên là tool riêng để harness có thể chặn/duyệt, thay vì để model chạy `bash` tùy ý.

**3. Không rò rỉ bí mật.** Tuyệt đối **không nhét API key, token, mật khẩu vào system prompt hay message** — chúng bị lưu trong lịch sử hội thoại và compaction. Giữ secret ở phía host: khi agent cần gọi API có auth, để *orchestrator* của bạn (đang giữ key) thực thi qua một custom tool, container/sandbox không bao giờ thấy key.

**4. Phân quyền và xác nhận (human-in-the-loop).** Với các thao tác có side-effect, dùng vòng lặp tool thủ công để **duyệt trước khi chạy** (permission gate), thay vì để tool runner tự động thực thi mọi thứ. Validate input trong chính hàm tool.

**5. Xử lý lỗi và rate limit** cũng là một phần của guardrails vận hành: bắt typed exception (`RateLimitError`, `APIStatusError`...), retry với backoff (SDK tự retry 429/5xx mặc định), log `request_id` khi báo lỗi.

Bẫy hay gặp: cho dữ liệu web/file người dùng vào prompt mà coi như đáng tin → bị prompt injection; hoặc đọc `content[0]` mà quên kiểm tra `stop_reason` → crash khi model từ chối; hoặc nhét secret vào prompt như một "cách nhanh" → rò rỉ vĩnh viễn trong lịch sử session.

</details>
## 11. Best Practices và Siêu năng lực với AI

### 11.1. Quy trình review code do AI sinh ra của bạn gồm những bước nào? Vì sao không thể chỉ "chạy thử thấy được là merge"?

<details>
<summary>👉 Xem câu trả lời</summary>

Tôi review code do AI sinh ra y hệt như review PR của một đồng nghiệp — thậm chí kỹ hơn, vì AI tự tin một cách đồng đều kể cả khi sai. Nguyên tắc xương sống: đọc kỹ, chạy test, và hiểu trước khi merge. "Chạy thử thấy được" chỉ chứng minh happy path hoạt động, không chứng minh code đúng ở edge case, không có lỗ hổng bảo mật, không tạo nợ kỹ thuật.

Quy trình ba lớp của tôi:

| Lớp | Làm gì | Mục tiêu |
| --- | --- | --- |
| Đọc | Đọc toàn bộ diff từng dòng | Hiểu từng thay đổi, phát hiện magic code |
| Chạy | Chạy test + lint + chạy app thật nếu cần | Xác nhận hành vi đúng, không chỉ compile |
| Hiểu | Tự giải thích lại "vì sao code này đúng" | Đảm bảo tôi sở hữu được code, debug được sau này |

```text
Checklist review code AI:
[ ] Diff có đủ ngắn để đọc hết không? (PR khổng lồ = cờ đỏ)
[ ] Mình giải thích được TỪNG thay đổi chưa?
[ ] Edge case: null, mảng rỗng, input sai, lỗi mạng — đã xử lý chưa?
[ ] Có dùng API nào deprecated / bịa (hallucinate) không?
[ ] Có test cho đường đi quan trọng chưa?
[ ] Code khớp convention dự án (CLAUDE.md) chưa?
[ ] Có chỗ "magic" mình không hiểu không? -> dừng, làm rõ
```

Vì sao "chạy được là merge" nguy hiểm: AI thường tạo code đúng trên happy path nhưng bỏ qua xử lý lỗi, hoặc dùng một mẫu (pattern) sai tinh vi trông rất thuyết phục. Test thủ công bằng tay chỉ đi vào vài đường phổ biến, không phủ được các nhánh hiếm. Nút thắt khi dùng AI không còn là "viết code" mà là "review code" — nên đầu tư vào khâu này chính là nơi tạo ra chất lượng. Bẫy lớn nhất là tưởng tốc độ sinh code bằng tốc độ giải quyết vấn đề; phần review và hiểu code vẫn là trách nhiệm của con người.

</details>

---

### 11.2. "Không tin tưởng mù quáng — luôn verify" nghĩa là gì trong thực tế? Bạn verify đầu ra của AI bằng cách nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Tin mù (automation bias) là xu hướng tin đầu ra của máy chỉ vì nó được trình bày tự tin và trôi chảy. AI có thể **hallucinate** (bịa) một API không tồn tại, một tham số sai, hoặc một mẫu code lỗi thời mà nhìn rất "đúng kiểu". Verify nghĩa là tôi luôn coi đầu ra của AI là một *giả thuyết cần kiểm chứng*, không phải sự thật.

Các cách verify cụ thể, theo loại đầu ra:

| Loại đầu ra của AI | Cách verify |
| --- | --- |
| Code | Chạy test, chạy lint, chạy app thật, đọc diff |
| API / hàm thư viện | Đối chiếu tài liệu chính thống (nghi ngờ = tra ngay) |
| Giải thích khái niệm | So với nguồn tin cậy, không tin một mình AI |
| Khẳng định "code này an toàn" | Tự kiểm tra logic, dùng skill security-review |
| Kết quả tìm kiếm codebase | Đọc lại file gốc, không tin tóm tắt 100% |

```text
Cờ đỏ buộc tôi phải verify gấp đôi:
- AI dùng một hàm/flag mà mình chưa từng thấy -> tra docs
- AI nói "chuẩn rồi", "an toàn rồi" mà không kèm bằng chứng
- Code đụng bảo mật / tiền / dữ liệu
- AI tự sửa nhiều file một lúc trên vùng mình không hiểu rõ
```

Một điểm quan trọng về kỷ luật verify: tôi để **máy** lo phần kiểm chứng lặp lại bằng tự động hóa — ví dụ hook `PostToolUse` tự chạy test/lint sau mỗi lần sửa file, hoặc CI chặn merge nếu test fail. Như vậy verify không phụ thuộc vào việc tôi (hay AI) có nhớ hay không. Bẫy nguy hiểm nhất là dùng AI cho đúng việc mình KHÔNG có khả năng kiểm chứng — khi đó tôi không thể phân biệt câu trả lời đúng với câu trả lời "nghe có vẻ đúng".

</details>

---

### 11.3. Làm sao bạn dùng AI để đọc hiểu và làm quen một codebase lạ thật nhanh — mà không rơi vào bẫy hiểu sai?

<details>
<summary>👉 Xem câu trả lời</summary>

Đây là một trong những "siêu năng lực" rõ rệt nhất của agentic coding: thay vì lần mò hàng giờ, tôi giao cho AI việc khảo sát và định vị, rồi tự đọc lại những chỗ trọng yếu để thật sự hiểu. Với Claude Code, tôi tận dụng **subagent Explore** chuyên tìm kiếm, và có thể chạy **nhiều subagent song song** để fan-out các câu hỏi độc lập, mỗi cái có context riêng nên không làm "loãng" phiên chính.

Quy trình của tôi:

```text
1. Hỏi tổng quan kiến trúc:
   "Mô tả cấu trúc dự án, các module chính và luồng dữ liệu từ request tới DB."
2. Định vị logic cụ thể (dùng Explore subagent):
   "Logic xác thực JWT nằm ở file nào? Liệt kê các chỗ gọi tới nó."
3. Truy vết một luồng end-to-end:
   "Khi user gọi POST /orders, đi qua những file/hàm nào theo thứ tự?"
4. Đọc lại CHÍNH những file AI chỉ ra -> tự xác nhận.
```

Cách tránh bẫy hiểu sai (rất quan trọng):

| Việc nên làm | Vì sao |
| --- | --- |
| Đọc lại file gốc AI trỏ tới | AI có thể tóm tắt sai hoặc bỏ sót nhánh |
| Hỏi "trỏ tôi tới dòng/file cụ thể" | Có địa chỉ rõ để tự kiểm chứng, tránh AI bịa |
| Kiểm chứng bằng cách chạy thử | Đọc hiểu khác với hành vi runtime |
| Ghi lại hiểu biết vào CLAUDE.md | Tích lũy ngữ cảnh cho lần sau, cho cả team |

Một ví dụ thực tế: khi nhận một dự án NestJS chưa quen, tôi hỏi Claude Code truy vết luồng từ controller xuống service rồi tới Prisma, để nó liệt kê các file theo thứ tự gọi. Tôi dùng kết quả đó như "tấm bản đồ", rồi tự mở từng file đọc kỹ ở chỗ logic nghiệp vụ.

Bẫy: tin tóm tắt của AI mà không bao giờ mở file gốc. Tóm tắt là điểm khởi đầu để biết "đọc cái gì trước", không phải thay thế cho việc đọc. AI tăng tốc việc *định vị*, nhưng việc *hiểu* vẫn cần mắt tôi.

</details>

---

### 11.4. Làm sao để refactor an toàn với sự hỗ trợ của AI? Điều kiện tiên quyết là gì?

<details>
<summary>👉 Xem câu trả lời</summary>

Refactor là một trong những việc AI làm tốt nhất (đổi tên xuyên file, tách hàm, gom logic trùng lặp), nhưng cũng dễ gây hỏng diện rộng nếu làm ẩu. Điều kiện tiên quyết tuyệt đối của tôi: **phải có test bao phủ trước khi cho AI sửa rộng**. Refactor theo định nghĩa là "thay đổi cấu trúc mà KHÔNG đổi hành vi" — chỉ có test mới chứng minh được hành vi không đổi.

Quy trình refactor an toàn:

```text
1. Đảm bảo có test xanh (green) bao phủ vùng sắp đổi.
   - Nếu chưa có -> viết test trước (có thể nhờ AI sinh, mình duyệt).
2. Bật plan mode: yêu cầu AI trình bày kế hoạch refactor trước.
   - Duyệt hướng đi, đảm bảo không bỏ sót chỗ.
3. Chia nhỏ: mỗi bước refactor là một thay đổi gọn, diff đọc được.
4. Sau mỗi bước -> chạy lại test (lý tưởng là hook tự chạy).
5. Commit nhỏ sau mỗi bước xanh -> dễ rollback nếu bước sau hỏng.
```

| Việc | An toàn để giao AI | Cần thận trọng |
| --- | --- | --- |
| Đổi tên biến/hàm xuyên file | Rất tốt | Nếu thiếu test, dễ sót chỗ |
| Tách hàm, gom code trùng | Tốt | Kiểm tra không đổi ngữ nghĩa |
| Đổi chữ ký hàm (signature) | Được | Phải cập nhật mọi caller + test |
| Refactor đụng concurrency/giao dịch | Hạn chế | Logic tinh vi, review cực kỹ |

Một điểm tận dụng AI: vì AI có thể chạy test trong vòng lặp agentic, tôi để nó tự "refactor -> chạy test -> nếu đỏ thì sửa tiếp", còn tôi giám sát kế hoạch và review diff cuối. Kết hợp với **commit nhỏ**, tôi luôn có điểm an toàn để quay lại.

Bẫy lớn nhất: cho AI refactor một vùng không có test rồi tin nó "không đổi hành vi". Không có test thì không có gì chứng minh điều đó — bạn chỉ đang đổi code và hy vọng. Bẫy thứ hai: gộp refactor và thêm tính năng vào cùng một thay đổi, khiến không phân biệt được lỗi đến từ đâu.

</details>

---

### 11.5. Bạn dùng AI để viết test như thế nào? Có rủi ro gì khi để AI tự sinh test?

<details>
<summary>👉 Xem câu trả lời</summary>

Viết test cùng AI rất hiệu quả: AI giỏi sinh bộ khung test, liệt kê edge case mà tôi có thể quên, tạo dữ liệu mẫu và mock. Nhưng tôi không "khoán trắng" khâu này, vì test là nơi tôi *chứng minh mình hiểu hành vi mong đợi* — nếu test do AI sinh ra mà tôi không hiểu, thì test đó vô giá trị.

Cách tôi phối hợp:

```text
- Tôi quyết định CÁI GÌ đáng test (đường đi quan trọng, edge case nghiệp vụ).
- AI sinh khung test + gợi ý các case tôi có thể bỏ sót.
- Tôi đọc kỹ TỪNG assertion: nó kiểm tra đúng cái cần kiểm tra chưa?
- Tôi bổ sung case nghiệp vụ mà chỉ tôi biết (AI không nắm "the why").
```

Các rủi ro khi để AI tự sinh test:

| Rủi ro | Mô tả | Cách phòng |
| --- | --- | --- |
| Test tautology | Test "luôn pass", không kiểm gì thật | Đọc kỹ assertion, thử cố tình làm fail |
| Test ngược lại bug | Sinh test khớp với code SAI thay vì với yêu cầu đúng | Viết test từ yêu cầu, không từ code |
| Phủ giả (fake coverage) | Cao % coverage nhưng không kiểm đúng nhánh | Nhìn vào nhánh logic, không nhìn con số |
| Mock quá đà | Mock hết, test không còn ý nghĩa | Giữ test gần hành vi thật |

Một mẹo verify hay dùng: sau khi AI sinh test, tôi cố tình **đưa một bug nhỏ vào code** xem test có đỏ không. Test không bắt được bug là test vô dụng. Đây cũng là tinh thần TDD: test phải thất bại trước, rồi mới làm cho nó xanh.

Bẫy nguy hiểm nhất: AI nhìn vào code hiện tại (có thể đang sai) rồi sinh test "khớp" với hành vi sai đó — kết quả là test xanh nhưng đang bảo vệ một bug. Vì vậy test nên xuất phát từ *yêu cầu/spec*, không phải từ *code hiện có*.

</details>

---

### 11.6. Trình bày cách bạn làm TDD (Test-Driven Development) cùng AI. Vì sao thứ tự "test trước" lại đặc biệt giá trị khi dùng AI?

<details>
<summary>👉 Xem câu trả lời</summary>

TDD theo vòng red-green-refactor: viết test thất bại trước (red), viết code tối thiểu cho test xanh (green), rồi dọn dẹp (refactor). Khi làm cùng AI, thứ tự "test trước" đặc biệt quý vì nó biến test thành một **đặc tả (spec) khách quan** ràng buộc đầu ra của AI — thay vì để AI tự "đoán" tôi muốn gì.

Quy trình TDD với AI:

```text
1. RED: Tôi (hoặc cùng AI) viết test mô tả hành vi mong muốn.
   - Chạy -> test FAIL (chứng minh test thật sự kiểm cái gì đó).
2. GREEN: Giao AI viết code tối thiểu để test xanh.
   - AI chạy test trong vòng lặp agentic tới khi xanh.
3. REFACTOR: Cho AI dọn code, test vẫn phải xanh.
4. Commit nhỏ sau mỗi vòng xanh.
```

Vì sao thứ tự test-trước đặc biệt giá trị với AI:

| Lợi ích | Giải thích |
| --- | --- |
| Spec rõ ràng cho AI | Test định nghĩa "đúng là gì", AI có mục tiêu khách quan để nhắm |
| Chống test-khớp-bug | Test viết TRƯỚC khi có code nên không thể "khớp" với code sai |
| Tiêu chí dừng tự động | AI biết khi nào xong: khi test xanh, không lan man |
| Hợp với vòng lặp agentic | AI tự sửa-chạy-sửa tới khi đạt, tôi chỉ giám sát |
| Buộc tôi nghĩ trước | Tôi phải định nghĩa hành vi mong muốn — giữ quyền thiết kế |

Một điểm mạnh khi kết hợp agentic coding: vì AI chạy được test, vòng "code -> chạy test -> đỏ -> sửa" diễn ra tự động, AI tự lặp tới khi xanh. Việc của tôi chuyển sang viết test chuẩn và review diff. Test trở thành cái "rọ" giữ AI không đi chệch yêu cầu.

Bẫy: để AI viết cả test lẫn code cùng lúc rồi cả hai đều "khớp" với hiểu lầm của AI về yêu cầu. Vì vậy tôi giữ quyền định nghĩa test (ít nhất là các case quan trọng) — đó là nơi tôi cài đặt ý định thật sự của mình vào quy trình.

</details>

---

### 11.7. AI sinh tài liệu — bạn dùng nó cho việc gì, và đảm bảo tài liệu không "lệch" với code thật bằng cách nào?

<details>
<summary>👉 Xem câu trả lời</summary>

AI rất giỏi việc tạo tài liệu vì đây là việc lặp lại, có khuôn mẫu, và tốn thời gian nếu làm tay. Tôi dùng nó để viết README, docstring/JSDoc, ADR (Architecture Decision Record), commit message, và mô tả PR. Nhưng tài liệu chỉ có giá trị khi nó **khớp với hành vi thật của code** — nên tôi luôn kiểm chứng phần này.

Các loại tài liệu tôi giao AI và rủi ro lệch:

| Loại tài liệu | AI giúp | Rủi ro lệch |
| --- | --- | --- |
| README / hướng dẫn dùng | Sinh nhanh, cấu trúc tốt | Mô tả tính năng không còn tồn tại |
| Docstring / JSDoc | Mô tả tham số, giá trị trả về | Tham số đã đổi nhưng doc cũ |
| Commit / PR description | Tóm tắt thay đổi mạch lạc | Tóm tắt sai phạm vi thay đổi |
| ADR | Ghi lại lý do quyết định | Lý do AI "đoán" chứ không phải lý do thật |

Cách đảm bảo không lệch:

```text
- Sinh tài liệu TỪ code/diff hiện tại, không từ trí nhớ/mô tả mơ hồ.
- Đọc lại: lệnh cài đặt có chạy đúng không? Ví dụ code có chạy không?
- Với commit/PR: AI tóm tắt diff, nhưng TÔI duyệt vì tôi biết "vì sao".
- Coi tài liệu như code: review, cập nhật cùng PR, không để nó "trôi".
- Dùng /init để Claude Code sinh CLAUDE.md, nhưng tôi soát lại convention.
```

Một lưu ý quan trọng về "the why": AI mô tả tốt code làm GÌ (the what), nhưng lý do lịch sử / đánh đổi (the why) là thứ chỉ con người biết. Với ADR và commit message, tôi để AI lo phần văn phong còn tôi bổ sung lý do thật.

Bẫy thường gặp: tài liệu đẹp nhưng mô tả một API đã đổi tham số — người đọc tin tài liệu rồi gặp lỗi. Tài liệu lệch còn tệ hơn không có tài liệu, vì nó tạo niềm tin sai. Cách phòng tốt nhất: viết tài liệu càng gần code càng tốt (docstring trong code), cập nhật cùng PR, và lý tưởng là có kiểm tra tự động (ví dụ doctest, hoặc CI kiểm ví dụ trong README).

</details>

---

### 11.8. Quy trình debug cùng AI của bạn ra sao? Làm sao tránh "sửa mò" và che giấu nguyên nhân gốc?

<details>
<summary>👉 Xem câu trả lời</summary>

AI là một "đôi mắt thứ hai" tuyệt vời khi debug: nó đọc stack trace, đọc log, đề xuất giả thuyết, và với agentic coding nó còn tự thêm log, chạy lại để thu thập bằng chứng. Nhưng tôi luôn yêu cầu xác định **nguyên nhân gốc (root cause)** trước khi sửa, để tránh tình trạng AI vá triệu chứng mà bệnh vẫn còn.

Quy trình của tôi:

```text
1. Cung cấp đủ bằng chứng: thông báo lỗi, stack trace, cách tái hiện.
2. Yêu cầu AI ĐỀ XUẤT GIẢ THUYẾT, không sửa ngay:
   "Liệt kê các nguyên nhân có thể, xếp theo khả năng, và cách kiểm chứng từng cái."
3. Kiểm chứng giả thuyết bằng dữ liệu (log, test tái hiện bug), không đoán.
4. Khi đã xác định root cause -> sửa ĐÚNG chỗ đó.
5. Viết test tái hiện bug (regression test) -> đảm bảo bug không quay lại.
```

So sánh sửa mò và sửa đúng:

| | Sửa mò (triệu chứng) | Sửa đúng (nguyên nhân gốc) |
| --- | --- | --- |
| Cách làm | Thêm try/catch, gán giá trị mặc định để hết lỗi | Hiểu vì sao lỗi xảy ra rồi sửa gốc |
| Hệ quả | Bug ẩn đi, xuất hiện lại chỗ khác | Bug biến mất thật, có test bảo vệ |
| Dấu hiệu | "Thử cái này xem", đổi đại tới khi hết đỏ | Giải thích được chuỗi nhân quả |

Một kỹ thuật hay với AI khi bí: yêu cầu nó giải thích "vì sao đoạn code này CHẠY ĐÚNG" (rubber-duck ngược) — quá trình đó thường lộ ra giả định sai. Hoặc yêu cầu nó tạo một test tối thiểu tái hiện bug; có test tái hiện rồi thì việc sửa trở nên xác định.

Bẫy nguy hiểm: AI (và cả con người) dễ rơi vào "đổi đại cho hết đỏ" — đặc biệt khi AI tự tin đề xuất một bản vá nhanh. Tôi luôn hỏi "bản vá này sửa triệu chứng hay nguyên nhân?". Một bẫy nữa với agentic: trước khi để AI chạy lệnh thay đổi trạng thái hệ thống (restart, xóa, sửa config) để "thử", hãy chắc bằng chứng thực sự ủng hộ hành động đó — đừng để nó pattern-match một lỗi quen thuộc rồi hành động trên giả định.

</details>

---

### 11.9. Khi làm việc với AI coding, bạn xử lý vấn đề bảo mật/bí mật như thế nào? Vì sao không bao giờ được dán secret/khóa cho AI?

<details>
<summary>👉 Xem câu trả lời</summary>

Nguyên tắc cứng của tôi: **không bao giờ dán secret, khóa API, mật khẩu, token, chuỗi kết nối DB thật** vào prompt, vào code mẫu, hay vào bất cứ đâu AI xử lý. Lý do: những nội dung này có thể bị lưu lại trong lịch sử phiên, log, hoặc context — một khi đã lộ ra, coi như khóa đó cần bị thu hồi. Secret đặt sai chỗ là rủi ro vĩnh viễn, không phải tạm thời.

Cách tôi xử lý:

```text
- Secret luôn nằm trong biến môi trường / file .env (đã .gitignore), KHÔNG trong code.
- Khi cần minh hoạ, dùng placeholder: process.env.API_KEY, "<your-key-here>".
- Không dán dữ liệu thật (PII, dữ liệu khách hàng) vào prompt; dùng dữ liệu giả.
- Dùng HOOK PreToolUse để CHẶN các lệnh nguy hiểm hoặc lệnh in lộ secret.
- Cấu hình permissions trong settings.json để giới hạn vùng AI được chạm.
- Với phần nhạy cảm (auth, mật mã, thanh toán) -> dùng skill security-review.
```

Vì sao AI dễ tạo ra rủi ro bảo mật:

| Rủi ro | Mô tả |
| --- | --- |
| Lưu vết secret | Khóa dán vào prompt có thể tồn tại trong context/log |
| Mẫu sai tinh vi | AI sinh code auth/crypto trông đúng nhưng có lỗ hổng |
| Hardcode | AI vô tình nhúng khóa thẳng vào code mẫu |
| API deprecated | Dùng hàm mã hóa cũ/yếu vì học từ dữ liệu lỗi thời |

Một điểm tư duy hệ thống: việc "chặn lệnh nguy hiểm" hay "không cho commit file .env" nên là **hook tất định** trong harness, chứ không phải nhắc AI nhớ — vì AI có thể quên, còn hook thì luôn chạy. Đây là sự phân biệt giữa kiểm soát bằng cơ chế và kiểm soát bằng niềm tin.

Bẫy thường gặp: nghĩ "chỉ dán tạm để AI debug rồi xoá" — nhưng nội dung đã đi qua context thì không lấy lại được. Quy tắc đơn giản: nếu một chuỗi mà bị lộ sẽ phải thu hồi/đổi, thì chuỗi đó không bao giờ được xuất hiện trước AI. Code đụng bảo mật là loại tôi review kỹ gấp đôi và ưu tiên tự viết hoặc đọc cực kỹ.

</details>

---

### 11.10. Quản lý ngữ cảnh (context) cho AI quan trọng thế nào? Bạn dùng CLAUDE.md và tham chiếu file ra sao để AI làm việc hiệu quả?

<details>
<summary>👉 Xem câu trả lời</summary>

Chất lượng đầu ra của AI tỉ lệ thuận với chất lượng ngữ cảnh đầu vào. AI không tự biết convention, kiến trúc, hay "luật" của dự án — nếu tôi không cung cấp, nó sẽ "đoán" theo trung bình phổ biến và thường lệch. Quản lý ngữ cảnh tốt chính là đòn bẩy lớn nhất để AI sinh code đúng phong cách ngay từ đầu, đỡ phải sửa nhiều vòng.

Các công cụ ngữ cảnh tôi dùng trong Claude Code:

| Công cụ | Dùng cho | Đặc điểm |
| --- | --- | --- |
| `CLAUDE.md` | Convention, lệnh hay dùng, kiến trúc, "luật" dự án | Tự nạp vào context; có phân cấp (user, project, thư mục con) |
| Tham chiếu file cụ thể | Trỏ AI tới đúng file/đoạn liên quan | Tránh AI mò cả codebase, giảm nhiễu |
| Memory | Ghi nhớ sở thích/sự kiện bền qua nhiều phiên | Khác với context tạm của một phiên |
| `/clear` | Xóa context khi chuyển sang việc không liên quan | Tránh "nhiễu chéo" giữa các nhiệm vụ |

```text
Một CLAUDE.md tốt thường ghi:
- Convention đặt tên, cấu trúc thư mục, mẫu code chuẩn của dự án.
- Lệnh hay dùng: chạy test, lint, build, migrate DB.
- Kiến trúc tổng thể và các ràng buộc nghiệp vụ quan trọng.
- Những điều NÊN / KHÔNG NÊN khi sửa code trong dự án này.
```

Nguyên tắc quản lý ngữ cảnh của tôi:
- Cung cấp đủ và đúng: trỏ thẳng tới file liên quan thay vì để AI tự tìm khắp nơi.
- Giữ context sạch: `/clear` giữa các nhiệm vụ khác nhau để tránh quyết định dựa trên thông tin cũ.
- Đưa cả "the why": ràng buộc nghiệp vụ, lý do — những thứ AI không thể tự suy ra.
- Tách "ghi nhớ bền" (memory, CLAUDE.md) khỏi "ngữ cảnh phiên" (những gì đang làm).

Một lưu ý về phân biệt cơ chế: nếu muốn một hành vi xảy ra TỰ ĐỘNG mỗi lần (vd luôn format sau khi sửa file), đó phải là **hook** chứ không phải một dòng nhắc trong CLAUDE.md — vì CLAUDE.md là ngữ cảnh để AI tham khảo, AI có thể không tuân thủ, còn hook thì harness luôn thực thi.

Bẫy thường gặp: kỳ vọng AI "tự biết" convention dự án; hoặc nhồi quá nhiều thứ không liên quan vào context khiến AI bị nhiễu. Ngữ cảnh tốt là *đủ và đúng trọng tâm*, không phải *càng nhiều càng tốt*.

</details>

---

### 11.11. Vì sao "commit nhỏ" lại đặc biệt quan trọng khi làm việc với AI? Nó giúp gì cho việc kiểm soát chất lượng?

<details>
<summary>👉 Xem câu trả lời</summary>

Commit nhỏ (small, focused commits) luôn là good practice, nhưng khi làm với AI nó trở nên *cực kỳ* quan trọng vì AI có thể sinh ra lượng thay đổi rất lớn trong thời gian ngắn. Commit nhỏ giúp tôi giữ được khả năng đọc hiểu, review, và quan trọng nhất là **luôn có điểm an toàn để quay lại** khi AI đi sai hướng.

Lợi ích của commit nhỏ khi dùng AI:

| Lợi ích | Giải thích |
| --- | --- |
| Diff review được | Diff nhỏ thì đọc hết và hiểu được; PR khổng lồ thì không |
| Điểm rollback an toàn | AI hỏng một bước -> revert về commit xanh gần nhất |
| Cô lập nguyên nhân | Mỗi commit một thay đổi -> biết lỗi đến từ đâu |
| Lịch sử kể chuyện | Đọc git log hiểu được tiến trình, không phải một cục lớn |
| Hợp với vòng lặp agentic | Sau mỗi bước xanh -> commit -> bước tiếp an toàn |

```text
Nhịp làm việc với AI:
[giao việc nhỏ] -> [AI sửa] -> [chạy test/lint] -> [review diff] -> [commit nhỏ]
       ^                                                                  |
       |____________________ việc tiếp theo ______________________________|

Nếu một bước hỏng -> revert về commit gần nhất, không mất công cả buổi.
```

Một quan sát thực tế: AI dễ tạo "PR khổng lồ" gộp nhiều thứ — đó là cờ đỏ vì không ai review nổi, và nếu một phần sai thì khó tách. Tôi chủ động chia task thành các đơn vị đủ nhỏ để mỗi commit là một thay đổi mạch lạc, có thể giải thích trong một câu.

Commit nhỏ cũng cộng hưởng với refactor an toàn và TDD: mỗi vòng red-green-refactor kết thúc bằng một commit xanh, tạo thành chuỗi các điểm an toàn liên tục.

Bẫy thường gặp: để AI chạy tự do hàng loạt thay đổi rồi commit một lần khổng lồ ở cuối — nếu có lỗi, bạn phải đọc lại toàn bộ và rất khó xác định lỗi nằm ở thay đổi nào. Commit nhỏ là cách rẻ nhất để mua lấy khả năng "lùi lại an toàn" khi làm việc với một công cụ tốc độ cao như AI.

</details>

---

### 11.12. Khi AI đi sai hướng (lạc đề, càng sửa càng hỏng), bạn xử lý thế nào? Khi nào reset, khi nào định hướng lại?

<details>
<summary>👉 Xem câu trả lời</summary>

AI đi sai hướng là chuyện bình thường, không phải thất bại — kỹ năng quan trọng là *nhận ra sớm* và *xử lý dứt khoát* thay vì để nó càng sửa càng đào sâu cái hố. Tôi có hai lựa chọn chính: định hướng lại (steer) hoặc reset (làm lại sạch), tùy mức độ lệch.

Dấu hiệu AI đang đi sai hướng:

```text
- Cùng một lỗi sửa vài lần vẫn không qua, mỗi lần lại đổi cách khác.
- Thay đổi lan rộng ra ngoài phạm vi yêu cầu ban đầu.
- AI thêm code/abstraction mà mình không yêu cầu (over-engineering).
- Context đã đầy thông tin sai/lạc đề từ những bước trước.
- Mình bắt đầu không hiểu nó đang làm gì nữa.
```

Khi nào dùng cách nào:

| Tình huống | Cách xử lý |
| --- | --- |
| Lệch nhẹ, hiểu sai một điểm | Định hướng lại: nói rõ chỗ sai, cung cấp ràng buộc cụ thể |
| Context đã nhiễm thông tin sai | `/clear` rồi bắt đầu lại với prompt và ngữ cảnh sạch hơn |
| Code đã hỏng lan rộng | Revert về commit xanh gần nhất rồi giao lại việc |
| Cứ loanh quanh không thoát | Dừng, tự làm phần khó, dùng AI cho phần còn lại |

Cách reset/định hướng lại hiệu quả:

```text
- Revert về commit nhỏ xanh gần nhất (đây là lý do commit nhỏ quan trọng).
- /clear để bỏ context cũ đã nhiễu, tránh AI "ám" theo hướng sai.
- Viết lại prompt rõ hơn: nêu rõ phạm vi, ràng buộc, "đừng làm gì".
- Quay lại plan mode: bắt AI trình bày kế hoạch trước khi sửa lại.
- Chia nhỏ hơn nữa: vấn đề có thể quá lớn cho một lượt.
```

Một quan sát: cố "vá" một phiên đã lạc đề bằng cách thêm prompt thường kém hiệu quả hơn là reset sạch — vì context cũ vẫn kéo AI về hướng sai. Tôi không ngại bỏ một phiên và bắt đầu lại với mô tả tốt hơn; thời gian bỏ ra để reset thường ít hơn nhiều so với gỡ rối một mớ hỗn độn.

Bẫy lớn nhất: tâm lý "tiếc công" — đã để AI làm lâu rồi nên cố cứu vớt thay vì revert. Với agentic coding, chi phí làm lại thấp (AI nhanh), nên đừng ngần ngại quay về điểm an toàn. Đây chính là lúc commit nhỏ, plan mode, và `/clear` phát huy giá trị: chúng cho tôi quyền kiểm soát để luôn "lái" được con thuyền, kể cả khi nó đã chệch hướng.

</details>
## 12. Tư duy và Đánh giá kết quả AI

### 12.1. Làm thế nào để bạn phát hiện một đoạn code do AI sinh ra bị "hallucination" (bịa ra thứ không tồn tại)?

<details>
<summary>👉 Xem câu trả lời</summary>

Hallucination trong code thường không phải lỗi cú pháp — code trông rất hợp lý, chạy thử có khi vẫn qua được — mà là việc AI bịa ra một thứ không tồn tại hoặc dùng sai: tên hàm/API không có thật, tham số sai, package không tồn tại trên registry, hành vi thư viện bị "tưởng tượng" theo kiểu "lẽ ra nó nên như vậy".

Một số dấu hiệu và cách tôi soát:

| Dạng hallucination | Dấu hiệu nhận biết | Cách kiểm chứng |
| --- | --- | --- |
| API/method không tồn tại | Tên nghe "quá tiện", quá hợp lý | Tra tài liệu chính thức, để IDE/TypeScript báo lỗi |
| Tham số/option bịa | Option giải đúng vấn đề một cách thần kỳ | Đọc type definition, đọc source thư viện |
| Package ma | Import lạ, chưa từng thấy | Kiểm tra trên npm/PyPI, xem số download |
| Hành vi sai của thư viện | Giả định mặc định không đúng | Viết một test nhỏ kiểm chứng thực tế |

Quy tắc của tôi: AI sinh ra `claim`, còn việc `verify` là của tôi. Những lớp phòng thủ rẻ và nhanh nhất nên đặt sát code: TypeScript/biên dịch để bắt API sai, linter, và test. Với agentic coding như Claude Code, tôi tận dụng chính vòng lặp agentic — yêu cầu agent chạy test/biên dịch để tự kiểm chứng, vì lúc đó hallucination sẽ lộ ra dưới dạng lỗi compile hoặc test fail.

```ts
// AI gợi ý — nghe rất "hợp lý" nhưng cần nghi ngờ:
const user = await prisma.user.findUniqueOrThrow({
  where: { email },
  // ⚠ AI có thể bịa option như "include: { ...}, cache: true"
  // -> kiểm chứng bằng type của Prisma, đừng tin vì nó "nghe đúng"
});

// Cách verify rẻ nhất: để compiler/type bắt lỗi ngay
// Nếu option không tồn tại, TS sẽ báo đỏ -> hallucination lộ diện
```

Một bẫy đặc biệt nguy hiểm là "slopsquatting": kẻ xấu đăng ký sẵn các tên package mà AI hay bịa ra, để khi ai đó cài theo gợi ý của AI thì dính mã độc. Vì vậy với mọi dependency mới do AI đề xuất, tôi luôn kiểm tra nó có thật, đúng tên, đủ uy tín trước khi cài.

</details>

---

### 12.2. Bạn đánh giá độ tin cậy của một output do AI tạo ra dựa trên những yếu tố nào? Có phải mọi output đều cần soát kỹ như nhau?

<details>
<summary>👉 Xem câu trả lời</summary>

Không — soát kỹ như nhau cho mọi thứ là lãng phí. Tôi điều chỉnh mức độ soi xét theo **rủi ro của vùng code** và **khả năng kiểm chứng**. Đây gần như một ma trận risk-based review.

Các yếu tố tôi cân nhắc:
- **Mức độ ảnh hưởng (blast radius)**: code đụng tới tiền, bảo mật, xóa dữ liệu, migration → soi cực kỹ. Code throwaway, script một lần → soi nhẹ.
- **Khả năng kiểm chứng tự động**: nếu có test/type/biên dịch bắt lỗi giúp, tôi tin hơn; nếu là logic nghiệp vụ tinh tế mà không test nào bắt được, tôi phải tự đọc.
- **Độ mới của kiến thức**: code dùng framework phổ biến, ổn định (đã có lâu trước thời điểm huấn luyện) đáng tin hơn nhiều so với API mới ra gần đây.
- **Độ "đặc thù dự án"**: convention, ràng buộc nghiệp vụ riêng — AI không thể biết nếu không có ngữ cảnh (`CLAUDE.md`), nên phần này dễ sai.
- **Độ phức tạp và độ dài**: diff càng lớn càng dễ giấu lỗi; tôi chủ động chia nhỏ để mỗi lần review đủ ngắn.

| Vùng code | Rủi ro | Mức soát |
| --- | --- | --- |
| Boilerplate CRUD, DTO | Thấp | Lướt + để test/type chặn |
| Logic nghiệp vụ cốt lõi | Cao | Đọc từng dòng, tự suy luận lại |
| Auth / phân quyền / thanh toán | Rất cao | Review cực kỹ + test + có thể nhờ người khác soát |
| Migration / xóa dữ liệu | Rất cao | Đọc kỹ, chạy thử trên môi trường nháp |
| Script throwaway | Thấp | Chạy thử là đủ |

Nguyên tắc gốc: độ tin cậy không phải thuộc tính của AI, mà là kết quả của quy trình kiểm chứng tôi áp lên output. Bẫy thường gặp là để "code trông sạch sẽ, tự tin" đánh lừa — AI luôn diễn đạt trôi chảy kể cả khi sai, nên giọng văn tự tin không phải bằng chứng đúng.

</details>

---

### 12.3. Vì sao AI đặc biệt hay sai với các thư viện/API mới ra sau thời điểm huấn luyện (training cutoff)? Bạn xử lý thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Model được huấn luyện trên dữ liệu tới một mốc thời gian nhất định (training cutoff). Mọi thứ ra đời hoặc thay đổi sau mốc đó, model về cơ bản "không biết". Nhưng vấn đề tệ hơn "không biết" là: model vẫn cố trả lời một cách tự tin, và nó lấp khoảng trống bằng cách suy diễn từ phiên bản cũ hoặc từ các thư viện tương tự. Kết quả là code dùng API đã `deprecated`, sai signature của bản mới, hoặc bịa hẳn ra API.

Các tình huống điển hình:
- Framework vừa ra major version mới (breaking changes) sau cutoff → AI viết theo cú pháp bản cũ.
- Thư viện vừa đổi tên hàm/đổi cách cấu hình → AI dùng tên cũ.
- SDK/dịch vụ mới hoàn toàn → AI bịa API "nghe hợp lý".

Cách tôi xử lý — cốt lõi là **bơm ngữ cảnh mới vào** thay vì tin trí nhớ của model:

```text
Chiến lược xử lý kiến thức lỗi thời:
1. Cung cấp tài liệu mới ngay trong ngữ cảnh
   - Dán đoạn doc/changelog của phiên bản đang dùng vào prompt
   - Dùng công cụ đọc web để lấy doc mới nhất khi cần
2. Dùng MCP server kết nối nguồn tài liệu/cập nhật (nếu có)
3. Ghim phiên bản và quy ước vào CLAUDE.md
   - Ví dụ: "Dự án dùng <thư viện> v<X>, KHÔNG dùng API cũ <...>"
4. Để compiler/type của chính phiên bản đó làm trọng tài
   - Cài đúng version -> type definitions sẽ bắt API sai
5. Hỏi thẳng: "Kiến thức của bạn có thể cũ; hãy kiểm tra doc trước khi khẳng định"
```

Một mẹo thực tế: nếu nghi ngờ một API mới, tôi yêu cầu agent đọc trực tiếp file type definition trong `node_modules` của dự án — đó là "sự thật mặt đất" (ground truth) đúng với phiên bản đang cài, thay vì dựa vào trí nhớ của model. Với Claude Code, việc để agent tự biên dịch và sửa theo lỗi compiler là cách hiệu quả để hội tụ về API đúng.

Bẫy cần tránh: hỏi "phiên bản mới nhất là gì?" rồi tin câu trả lời — model không có cách biết "mới nhất" thật sự nếu không tra cứu; con số nó đưa ra có thể là phỏng đoán.

</details>

---

### 12.4. Ai chịu trách nhiệm cuối cùng cho một đoạn code do AI viết và đã được ship lên production? Quan điểm này ảnh hưởng thế nào đến cách bạn làm việc?

<details>
<summary>👉 Xem câu trả lời</summary>

Câu trả lời thẳng và không có vùng xám: **tôi — lập trình viên đặt commit/mở PR — chịu trách nhiệm hoàn toàn.** "AI viết đoạn này" không bao giờ là lời bào chữa hợp lệ khi có sự cố. Về mặt nghề nghiệp và đạo đức, ký tên vào commit nghĩa là tôi đứng ra bảo chứng cho từng dòng, bất kể nó được gõ bằng tay hay do agent sinh.

Quan điểm này định hình cách làm việc theo một nguyên tắc cứng: **tôi không merge thứ tôi không hiểu và không giải thích được.** Cụ thể:
- Mọi đoạn code do AI sinh đều phải qua tay tôi như thể một junior submit PR cho tôi review — không tin mặc định.
- Nếu một đoạn "chạy được nhưng tôi không hiểu vì sao", tôi coi đó là cờ đỏ: hoặc tôi đào tới khi hiểu, hoặc tôi viết lại theo cách mình kiểm soát được.
- Tôi giữ các cơ chế chất lượng độc lập với AI: test, type, code review của đồng đội, hook tự động lint/test (do harness chạy, không phụ thuộc AI có "nhớ" hay không).

```text
Phân định trách nhiệm:
  AI       -> tạo ra văn bản code (a draft)
  Tôi      -> review, kiểm chứng, ra quyết định merge
  Tôi      -> chịu trách nhiệm khi production lỗi
  AI       -> KHÔNG chịu trách nhiệm gì cả
```

Một cách diễn đạt tôi hay dùng trong phỏng vấn: AI là người **đề xuất**, tôi là người **quyết định và bảo chứng**. Năng suất tăng nhờ AI là thật, nhưng nó không chuyển giao trách nhiệm — chỉ chuyển công việc gõ phím. Bẫy văn hóa nguy hiểm nhất ở một đội dùng AI là tâm lý "AI làm mà", làm loãng cảm giác sở hữu (ownership); tôi chủ động chống lại điều đó bằng kỷ luật review nghiêm.

</details>

---

### 12.5. Bạn cân bằng giữa tốc độ mà AI mang lại và nhu cầu hiểu sâu vấn đề như thế nào? Làm sao để không bị "kỹ năng teo lại"?

<details>
<summary>👉 Xem câu trả lời</summary>

Đây là một đánh đổi thật, không phải lý thuyết. AI cho tốc độ gõ code rất lớn, nhưng nếu tôi chỉ chấp nhận output mà không hiểu, về dài hạn kỹ năng tự suy luận và kiến thức nền sẽ teo lại (skill atrophy) — và đúng lúc khẩn cấp (production cháy, AI bí) tôi sẽ không tự xoay xở được.

Cách tôi cân bằng dựa trên việc phân loại task theo mục tiêu:

| Loại công việc | Ưu tiên | Cách dùng AI |
| --- | --- | --- |
| Boilerplate lặp lại, đã hiểu rõ | Tốc độ | Để AI làm nhanh, lướt review |
| Vùng cốt lõi / quyết định kiến trúc | Hiểu sâu | Tự suy nghĩ trước, dùng AI để phản biện |
| Học công nghệ/lĩnh vực mới | Hiểu sâu | Dùng AI như gia sư: hỏi "vì sao", không chỉ "code gì" |
| Debug lỗi lạ | Cân bằng | Nhờ AI gợi ý hướng, tự xác nhận nguyên nhân gốc |

Một số nguyên tắc cụ thể:
- **Đầu tư hiểu sâu vào nơi tạo ra giá trị**: kiến trúc, mô hình dữ liệu, ranh giới hệ thống, đánh đổi. Đây là phần AI không thay được, và là phần làm nên một engineer.
- **Dùng plan mode như một bước "ép mình suy nghĩ"**: đọc kế hoạch của agent, tôi phải hiểu và phê duyệt — điều này buộc tôi nắm được cách tiếp cận trước khi code chạy.
- **Dùng AI để học, không chỉ để xong việc**: yêu cầu nó giải thích vì sao chọn cách này, đánh đổi là gì, có cách nào khác không. Biến nó thành công cụ tăng hiểu biết.
- **Định kỳ "tập tay không"**: với những kỹ năng cốt lõi, thỉnh thoảng tôi tự làm để giữ phong độ.

Cách diễn đạt then chốt: AI tăng tốc phần **typing**, nhưng phần **thinking** vẫn là của tôi và tôi cố tình không giao nó đi. Bẫy lớn nhất là nhầm "tốc độ gõ code" với "tốc độ giải quyết vấn đề" — ship nhanh một giải pháp mình không hiểu thực ra là tạo nợ kỹ thuật, không phải năng suất thật.

</details>

---

### 12.6. Bạn đo lường tác động thực sự của AI lên năng suất như thế nào? Cẩn thận với những chỉ số nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Câu hỏi này dễ trả lời sai vì cám dỗ dùng các chỉ số "kêu to mà rỗng". Năng suất kỹ thuật thật sự là **giá trị giao được một cách bền vững**, không phải lượng code sinh ra.

Những chỉ số dễ gây hiểu lầm (vanity metrics) cần cảnh giác:
- **Số dòng code / số commit**: AI làm tăng vọt mà chẳng nói gì về giá trị; nhiều code hơn thường là nợ nhiều hơn.
- **% gợi ý AI được chấp nhận**: chấp nhận không có nghĩa là đúng hay tốt.
- **Tốc độ ra PR đầu tiên**: nhanh nhưng nếu kéo theo nhiều vòng sửa thì không có lợi.

Những tín hiệu tôi tin hơn — gần với các chỉ số kiểu DORA và chất lượng dài hạn:
- **Lead time**: từ lúc bắt đầu tới lúc lên production, tính cả thời gian review và sửa.
- **Change failure rate**: tỷ lệ thay đổi gây lỗi/rollback — nếu AI làm cái này tăng thì tốc độ là ảo.
- **Tỷ lệ rework / số vòng review**: code AI có hay bị trả về sửa nhiều lần không.
- **Bug thoát ra production** và **nợ kỹ thuật tích lũy** theo thời gian.
- **Cảm nhận của đội**: thời gian dành cho việc giá trị cao (thiết kế) so với việc nhàm chán.

```text
Cách tiếp cận đo lường đúng:
- So sánh có đối chứng: cùng loại task, có/không AI -> nhìn lead time + chất lượng
- Đo theo OUTCOME (giá trị giao được), không theo OUTPUT (lượng code)
- Nhìn cả hệ thống: tốc độ + ổn định, không hy sinh cái này cho cái kia
- Định tính bổ sung định lượng: hỏi đội họ thấy AI giúp ở đâu, hại ở đâu
```

Bẫy kinh điển là tối ưu cái dễ đo (dòng code) thay vì cái quan trọng (giá trị). Tôi cũng cảnh giác với hiệu ứng "cảm giác nhanh hơn" — vài nghiên cứu cho thấy lập trình viên đôi khi *cảm thấy* AI giúp họ nhanh hơn ngay cả khi đo thực tế không nhanh hơn. Vì vậy tôi ưu tiên số liệu khách quan và outcome hơn cảm nhận.

</details>

---

### 12.7. Khi dùng AI để sinh code, bạn xử lý các vấn đề đạo đức, bản quyền và license của code đầu ra như thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Code do AI sinh không tự nhiên "sạch" về mặt pháp lý chỉ vì máy tạo ra. Có vài rủi ro thật cần quản lý:

- **License lây nhiễm (copyleft)**: nếu AI tái tạo gần như nguyên văn một đoạn code có license ràng buộc như GPL, đưa vào sản phẩm thương mại closed-source có thể vi phạm. Rủi ro cao nhất với những đoạn dài, đặc thù, "nguyên khối".
- **Bản quyền / ghi nguồn**: code huấn luyện đến từ nhiều nguồn; phần thuộc về ai và trách nhiệm ghi nhận vẫn là vùng đang tranh luận về pháp lý.
- **Dependency do AI đề xuất**: mỗi thư viện kéo theo license riêng — phải kiểm tra tương thích với license dự án (ví dụ tránh GPL trong sản phẩm đóng).

Cách tôi giảm rủi ro trong thực tế:

| Rủi ro | Biện pháp |
| --- | --- |
| Tái tạo nguyên văn code có license | Cảnh giác với khối lớn, đặc thù; viết lại theo cách của mình |
| License của dependency mới | Kiểm tra license trước khi cài; dùng công cụ quét license/SCA |
| Lỗ hổng/đạo văn ẩn | Chạy SCA và quét bảo mật trong CI |
| Quy định nội bộ công ty | Tuân thủ chính sách dùng AI của tổ chức (tool nào được duyệt) |

```text
Nguyên tắc thực hành:
- Coi code AI sinh giống code lấy từ nguồn ngoài: phải qua kiểm tra license
- Đoạn càng dài + càng "giống nguyên bản" -> càng nên viết lại
- Tự động hóa: tích hợp license-check / SCA vào pipeline (CI), có thể qua hook
- Đọc và tuân thủ điều khoản của chính nhà cung cấp AI về quyền dùng output
```

Quan điểm tôi nêu trong phỏng vấn: trách nhiệm pháp lý cuối cùng thuộc về đội và tổ chức ship sản phẩm, không phải về công cụ AI. Vì vậy tôi đối xử với output của AI bằng cùng sự cẩn trọng như khi copy code từ internet — không dán nguyên si những đoạn lớn không rõ nguồn gốc, và để công cụ tự động (SCA, license scanner) làm lớp chặn khách quan thay vì dựa vào trí nhớ con người.

</details>

---

### 12.8. Bạn xử lý dữ liệu nhạy cảm và yêu cầu tuân thủ (compliance) như thế nào khi làm việc với công cụ AI và Claude API?

<details>
<summary>👉 Xem câu trả lời</summary>

Nguyên tắc nền tảng: **những gì tôi đưa vào prompt/ngữ cảnh là dữ liệu rời khỏi tầm kiểm soát của tôi.** Vì vậy tôi xử lý dữ liệu nhạy cảm với AI cũng nghiêm như khi gửi ra bất kỳ dịch vụ bên thứ ba nào, theo nguyên tắc tối thiểu hóa dữ liệu (data minimization).

Các loại dữ liệu cần cẩn trọng: secrets (API key, mật khẩu, token), PII của người dùng, dữ liệu khách hàng, code/thông tin kinh doanh thuộc sở hữu công ty, và dữ liệu nằm trong phạm vi quy định như GDPR, HIPAA, PCI-DSS.

Cách tôi làm việc an toàn:

```text
Quy tắc xử lý dữ liệu nhạy cảm với AI:
1. KHÔNG dán secrets/credentials vào prompt
   - Dùng biến môi trường (.env), secret manager; AI chỉ thấy tên biến
2. Tối thiểu hóa: chỉ đưa đúng ngữ cảnh cần thiết, không "dán cả database"
3. Ẩn danh / che dữ liệu (mask, redact) khi cần ví dụ với PII thật
4. Cấu hình harness để giảm rủi ro
   - Permission mode hỏi trước; KHÔNG bừa bãi bypassPermissions
   - Hook PreToolUse có thể chặn lệnh đọc/gửi file nhạy cảm
   - Giới hạn phạm vi thư mục agent được phép chạm
5. Cẩn thận với MCP server bên ngoài: nó mở rộng cả bề mặt rò rỉ dữ liệu
6. Tuân thủ chính sách tổ chức và yêu cầu pháp lý/khu vực lưu trữ
```

Về phía **Claude API** trong sản phẩm tự xây, có vài điểm tuân thủ cần lưu ý ở mức tổng quát: chọn đúng phương án triển khai phù hợp với yêu cầu lưu trú dữ liệu và compliance của tổ chức; xem xét điều khoản về cách dữ liệu được xử lý và lưu giữ; và ở tầng ứng dụng, tự mình lọc/ẩn danh PII trước khi gửi đi, ghi log audit có kiểm soát, và không log nguyên văn dữ liệu nhạy cảm. (Chi tiết chính sách cụ thể tôi sẽ tra tài liệu chính thức thay vì khẳng định chắc chắn.)

Bẫy thường gặp: vô tình để agent đọc file `.env` hoặc paste log production chứa PII vào prompt. Đây chính là lý do hành vi "luôn chặn X" nên đặt ở **hook/permission** (do harness cưỡng chế) chứ không trông cậy vào việc con người hay AI "nhớ đừng làm" — kiểm soát bằng cơ chế, không bằng kỷ luật ý chí.

</details>
## 13. Câu hỏi hành vi về ứng dụng AI

### 13.1. Hãy mô tả quy trình hằng ngày của bạn khi dùng AI (đặc biệt là Claude Code) trong công việc phát triển phần mềm. AI xen vào những bước nào, và bạn giữ vai trò gì ở mỗi bước?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Đây là câu hỏi hành vi — nhà tuyển dụng muốn thấy AI đã trở thành một phần *có hệ thống* trong quy trình của bạn, chứ không phải dùng tùy hứng. Hãy kể theo cấu trúc STAR và nhấn mạnh bạn luôn là người cầm lái. Dưới đây là một mẫu bạn có thể điều chỉnh theo trải nghiệm thật:

- **Situation (Bối cảnh):** "Tôi làm backend cho một dự án NestJS + Prisma (Jira Clone). Mỗi ngày tôi nhận vài task từ ticket: thêm tính năng, sửa bug, refactor, viết test. Trước đây phần lớn thời gian là gõ tay boilerplate và đi tìm code trong một codebase lớn."

- **Task (Nhiệm vụ):** "Tôi muốn AI gánh phần cơ học và phần khám phá code, để tôi dồn năng lượng vào thiết kế và ra quyết định — nhưng vẫn phải hiểu và chịu trách nhiệm mọi thứ mình ship."

- **Action (Hành động):** "Quy trình một ngày của tôi thường là:
  - **Đầu phiên:** tôi để Claude Code tự nạp ngữ cảnh từ `CLAUDE.md` (convention, kiến trúc, lệnh build/test). Nếu chuyển sang task mới hẳn, tôi gõ `/clear` để dọn context cũ, tránh nhiễu.
  - **Khám phá:** với codebase lạ, tôi dùng subagent Explore để fan-out tìm 'logic X nằm ở đâu' mà không làm phình context chính.
  - **Lập kế hoạch:** với task không tầm thường, tôi bật plan mode — bắt AI trình bày kế hoạch trước khi sửa một dòng code nào, để tôi duyệt hướng đi.
  - **Thực thi:** tôi giao phần boilerplate/refactor cho AI ở permission mode mặc định (hỏi trước), chỉ chuyển sang acceptEdits cho phạm vi đã tin tưởng.
  - **Kỷ luật chất lượng:** tôi cấu hình hook PostToolUse tự chạy lint/test mỗi khi file thay đổi — việc tất định này do harness lo, không phụ thuộc AI nhớ.
  - **Review:** tôi đọc toàn bộ diff như review PR của đồng nghiệp; phần nhạy cảm tôi dùng skill code-review / security-review."

- **Result (Kết quả):** "Phần gõ tay rút ngắn rõ rệt, tôi tìm code nhanh hơn nhiều, và chất lượng không giảm vì hook + review luôn là chốt chặn. Quan trọng nhất: tôi vẫn giải thích được từng thay đổi trong PR của mình."

Lời khuyên khi trả lời thật: nêu cụ thể từng công cụ vào từng bước (CLAUDE.md, plan mode, subagent, hook, skill) để cho thấy bạn hiểu *đúng việc dùng đúng công cụ*, không phải chỉ 'chat với AI'. Tránh mô tả mơ hồ kiểu 'tôi hỏi AI rồi copy'.

</details>

---

### 13.2. Kể về một task cụ thể mà AI giúp bạn tăng tốc đáng kể. Việc đó nếu làm tay sẽ tốn bao lâu, và bạn đã đảm bảo chất lượng không bị hy sinh như thế nào?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Đây là câu hỏi hành vi kinh điển — hãy chọn một câu chuyện có **con số** (thời gian tiết kiệm, số file, số test) và thể hiện rõ bạn vẫn kiểm soát chất lượng. Mẫu tham khảo:

- **Situation (Bối cảnh):** "Trong dự án, tôi cần thêm một lớp validation và DTO nhất quán cho khoảng 12 endpoint hiện có, kèm test cho từng cái. Làm tay theo kinh nghiệm sẽ mất khoảng 2-3 ngày vì lặp lại và dễ sai sót do mệt."

- **Task (Nhiệm vụ):** "Rút ngắn thời gian xuống còn một phần nhỏ, nhưng tuyệt đối không hạ chất lượng — code phải khớp convention dự án và có test bao phủ các đường đi quan trọng."

- **Action (Hành động):** "Tôi mô tả rõ khuôn mẫu (pattern) trong prompt và để Claude Code đọc một ví dụ mẫu đã đúng convention từ `CLAUDE.md`. Tôi giao cho AI sinh DTO + validation + test theo từng endpoint, chia nhỏ để mỗi diff đủ ngắn mà tôi review kỹ. Tôi bật hook tự chạy test sau mỗi thay đổi để bắt lỗi ngay. Với các edge case nghiệp vụ (ví dụ ràng buộc về quyền, trạng thái hợp lệ), tôi tự bổ sung và tự viết assertion, vì đó là phần AI dễ đoán sai."

- **Result (Kết quả):** "Khối việc 2-3 ngày rút xuống còn khoảng nửa ngày. Test bao phủ các luồng chính, hook đảm bảo không có lần nào tôi merge code đỏ test. Quan trọng nhất, tôi đọc hết diff và giải thích được mọi thay đổi khi review. Tôi nhận ra điểm cốt lõi: AI tăng tốc phần 'typing', còn phần 'thinking' về edge case vẫn là của tôi — và đó là nơi giá trị thật nằm."

Lời khuyên: đừng chỉ nói 'AI làm nhanh hơn'. Hãy gắn con số và mô tả *cơ chế giữ chất lượng* (diff nhỏ, hook test, tự viết phần edge case). Nhà tuyển dụng tìm người dùng AI để khuếch đại năng lực chứ không buông trách nhiệm.

</details>

---

### 13.3. Kể về một lần AI dẫn bạn đi sai hướng — ví dụ nó bịa ra API, gợi ý một giải pháp sai tinh vi, hoặc tạo code chạy được nhưng sai bản chất. Bạn phát hiện và xử lý ra sao?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Đây là câu hỏi hành vi kiểm tra **tư duy phản biện** và khả năng không tin mù AI (automation bias). Câu trả lời tốt cho thấy bạn có cơ chế phát hiện sai và một quy trình xử lý bình tĩnh. Mẫu tham khảo:

- **Situation (Bối cảnh):** "Tôi nhờ AI viết một đoạn xử lý phân trang với Prisma. Code trông rất hợp lý, chạy được trên dữ liệu nhỏ và pass test ban đầu của tôi. Nhưng AI dùng một cách offset đơn giản và còn gợi ý một option cấu hình mà tôi mơ hồ là không tồn tại đúng như nó mô tả."

- **Task (Nhiệm vụ):** "Tôi cần xác định liệu giải pháp này có đúng và bền vững không, trước khi cho nó vào production — đặc biệt khi dữ liệu lớn lên."

- **Action (Hành động):** "Có hai cờ đỏ khiến tôi dừng lại: một là tôi không thể giải thích chắc chắn behavior khi dữ liệu lớn (hiệu năng offset), hai là cái option kia 'nghe có vẻ đúng' nhưng tôi chưa từng thấy trong tài liệu. Tôi áp dụng quy tắc 'không hiểu thì không merge'. Tôi đối chiếu trực tiếp với tài liệu chính thống của Prisma — phát hiện option đó là hallucination, AI bịa ra một API không tồn tại. Tôi cũng nhận ra offset phân trang sẽ chậm dần với dữ liệu lớn. Tôi yêu cầu AI giải thích 'vì sao chọn cách này' và đề xuất phương án cursor-based, rồi tự kiểm chứng lại bằng tài liệu trước khi áp dụng."

- **Result (Kết quả):** "Tôi thay bằng cursor-based pagination đúng API thật, có test cho trường hợp dữ liệu lớn. Bài học tôi rút ra và áp dụng từ đó: (1) luôn kiểm chứng API lạ bằng tài liệu chính thống, đừng tin vì nó 'trông thuyết phục'; (2) nếu tôi không kiểm chứng được đầu ra thì tôi chưa đủ tư cách ship nó; (3) test trên dữ liệu thực tế quy mô lớn, không chỉ happy path."

Lời khuyên: chọn một sự cố THẬT, kể rõ *dấu hiệu* khiến bạn nghi ngờ và *cách kiểm chứng* (đối chiếu tài liệu, đọc lại logic, test edge case). Đừng đổ lỗi cho AI — điểm cộng nằm ở việc bạn coi review là trách nhiệm của mình. Tránh kết kiểu 'từ đó tôi không tin AI nữa'; thông điệp đúng là 'tôi tin nhưng luôn kiểm chứng' (trust but verify).

</details>

---

### 13.4. Hãy kể về một lần bạn thuyết phục đồng nghiệp hoặc cả team áp dụng AI (hoặc một quy trình dùng AI) khi họ còn hoài nghi. Bạn đã làm thế nào?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Đây là câu hỏi hành vi về **ảnh hưởng và dẫn dắt thay đổi**. Câu trả lời tốt cho thấy bạn thuyết phục bằng bằng chứng và đồng cảm với lo ngại của người khác, không áp đặt. Mẫu tham khảo:

- **Situation (Bối cảnh):** "Team tôi có vài bạn hoài nghi AI: lo nó sinh code rác, lo lộ thông tin, và cho rằng 'tự viết còn nhanh hơn đi sửa code AI'. Có cả nỗi lo ngầm rằng dùng AI là 'gian lận' hoặc làm teo kỹ năng."

- **Task (Nhiệm vụ):** "Tôi muốn team thử một quy trình dùng AI an toàn, có kiểm soát, để chứng minh nó tăng tốc thật mà không hạ chất lượng — nhưng không ép buộc ai."

- **Action (Hành động):** "Tôi không thuyết giảng. Tôi chọn một task lặp lại ai cũng ghét (sinh boilerplate + test) và làm một bản demo nhỏ: cùng task đó, đo thời gian làm tay so với dùng Claude Code có review nghiêm. Tôi lắng nghe và trả lời thẳng từng lo ngại: với 'sợ code rác', tôi cho thấy quy trình review diff + hook tự lint/test làm chốt chặn; với 'sợ lộ thông tin', tôi nói rõ về permission mode (mặc định hỏi trước) và hook chặn lệnh nguy hiểm; với 'sợ teo kỹ năng', tôi chia sẻ nguyên tắc 'AI làm phần cơ học, người vẫn quyết kiến trúc và edge case'. Sau đó tôi đề xuất một quy trình tối thiểu để cả team thống nhất: viết `CLAUDE.md` chung cho dự án, đóng gói skill code-review để mọi người dùng cùng một checklist, và bắt buộc review diff trước khi merge."

- **Result (Kết quả):** "Nhờ demo có số liệu và việc tôi giải quyết được đúng lo ngại của họ, team đồng ý thử nghiệm trên các task boilerplate trước. Khi thấy thời gian giảm mà bug không tăng (nhờ hook + review), những bạn hoài nghi nhất dần dùng nhiều hơn. Quan trọng là cả team có chung một quy trình an toàn, không ai dùng AI theo kiểu vibe coding lén lút."

Lời khuyên: nhấn mạnh bạn thuyết phục bằng **bằng chứng cụ thể** (demo, số liệu) và **lắng nghe lo ngại thật** thay vì áp đặt. Cho thấy bạn đề xuất *quy trình có kiểm soát* (review, hook, permission) chứ không chỉ 'cứ dùng đi'. Đó mới là dẫn dắt thay đổi một cách trưởng thành.

</details>

---

### 13.5. Khi dùng AI để đi nhanh, làm sao bạn cân bằng giữa TỐC ĐỘ và CHẤT LƯỢNG? Kể một tình huống bạn phải đánh đổi và lý do bạn chọn như vậy.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Đây là câu hỏi hành vi về **phán đoán kỹ thuật (engineering judgment)**. Câu trả lời tốt cho thấy bạn không tôn thờ tốc độ một cách mù quáng mà điều chỉnh mức độ kiểm soát theo rủi ro của task. Mẫu tham khảo:

- **Situation (Bối cảnh):** "Sát deadline demo, tôi cần đồng thời hai thứ: một màn hình prototype để khách hàng xem (dùng một lần, có thể vứt), và một endpoint xử lý thanh toán đi vào production thật."

- **Task (Nhiệm vụ):** "Tôi phải phân bổ mức độ 'tin AI' khác nhau cho hai phần này, vì hậu quả nếu sai rất khác nhau."

- **Action (Hành động):** "Tôi áp dụng nguyên tắc: mức kiểm soát tỉ lệ thuận với rủi ro.
  - Với prototype demo: tôi cho phép kiểu gần như vibe coding — để AI sinh nhanh, tôi chỉ thử chạy, vì nó sẽ bị vứt đi và không chạm dữ liệu thật. Ở đây tốc độ thắng.
  - Với endpoint thanh toán: tôi đi cực chậm và kỹ. Tôi dùng plan mode để duyệt hướng trước, tự viết phần logic nhạy cảm, đọc từng dòng diff, chạy skill security-review, và viết test cho cả edge case (số tiền âm, race condition, idempotency). Ở đây chất lượng tuyệt đối thắng tốc độ.
  Điểm mấu chốt là tôi tách bạch ranh giới: tuyệt đối không để code prototype 'trườn' thẳng vào production mà không qua review."

- **Result (Kết quả):** "Prototype xong rất nhanh, kịp demo. Endpoint thanh toán thì chậm hơn nhưng an toàn — security-review bắt được một chỗ thiếu kiểm tra idempotency mà nếu vội tôi đã bỏ sót. Bài học: 'năng suất' không phải số dòng code/giờ, mà là số tính năng đúng và bền/đơn vị thời gian. Đi nhanh chỗ rủi ro thấp, đi chậm và chắc chỗ rủi ro cao."

Lời khuyên: thể hiện bạn coi cân bằng này là một *quyết định có chủ đích theo rủi ro*, không phải lúc nào cũng 'càng nhanh càng tốt'. Nêu được một lần bạn cố ý đi chậm để giữ chất lượng là điểm cộng lớn — nó cho thấy sự trưởng thành.

</details>

---

### 13.6. Bạn đã từng hướng dẫn người khác (đồng nghiệp mới, junior) dùng AI hiệu quả chưa? Bạn dạy họ những nguyên tắc gì để họ không rơi vào bẫy lạm dụng?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Đây là câu hỏi hành vi về **mentoring và truyền đạt văn hóa kỹ thuật**. Câu trả lời tốt cho thấy bạn không chỉ dùng AI giỏi mà còn giúp người khác dùng *có trách nhiệm*. Mẫu tham khảo:

- **Situation (Bối cảnh):** "Một bạn junior mới vào team dùng AI rất hăng nhưng theo kiểu nguy hiểm: copy nguyên đầu ra vào PR mà không đọc, PR khổng lồ không ai review nổi, và khi bị hỏi 'tại sao chỗ này làm vậy' thì bạn ấy không giải thích được."

- **Task (Nhiệm vụ):** "Tôi cần giúp bạn ấy giữ được tốc độ mà AI mang lại, nhưng tránh bẫy phụ thuộc quá mức và mất quyền sở hữu code."

- **Action (Hành động):** "Tôi pair với bạn ấy vài buổi và truyền đạt mấy nguyên tắc cốt lõi:
  - **'Không hiểu thì không merge'** — AI là một đồng nghiệp cần được review, phải đọc hết diff như review PR của người thật.
  - **Chia task nhỏ** để diff đủ ngắn mà đọc hiểu; PR khổng lồ do AI sinh là cờ đỏ.
  - **Kiểm chứng API lạ bằng tài liệu chính thống** vì AI có thể hallucinate một cách thuyết phục.
  - **Dùng đúng công cụ:** plan mode cho task khó để duyệt hướng trước; hook cho việc tự động lặp lại (lint/test) thay vì 'nhờ AI nhớ'; subagent Explore để tìm code.
  - **Phân biệt khi nào dùng/không dùng AI:** giao mạnh phần boilerplate có khuôn mẫu; tự làm và học phần nền tảng để không teo kỹ năng; cực kỳ thận trọng với code chạm bảo mật/tiền/dữ liệu.
  Tôi cũng nhấn mạnh cách *đánh giá đầu ra*, vì kỹ năng quan trọng nhất thời AI không phải là viết prompt mà là review được kết quả."

- **Result (Kết quả):** "Sau vài tuần, bạn ấy bắt đầu chia PR nhỏ hơn, đọc diff trước khi mở PR, và tự bắt được vài lỗi AI bịa. Chất lượng PR tăng rõ và bạn ấy giải thích được code của mình trong review. Tôi rút ra rằng dạy người khác dùng AI hiệu quả chủ yếu là dạy *kỷ luật và tư duy phản biện*, không phải mẹo prompt."

Lời khuyên: nhấn mạnh bạn dạy *nguyên tắc và tư duy* (sở hữu code, review, kiểm chứng, đúng công cụ), không chỉ thao tác. Một mẫu STAR có chuyển biến rõ ở người được mentor sẽ rất thuyết phục. Tránh kể kiểu 'tôi chỉ họ vài câu lệnh' — chiều sâu nằm ở văn hóa chất lượng bạn truyền lại.

</details>

---

### 13.7. Nhiều người lo AI sẽ thay thế lập trình viên. Quan điểm của bạn là gì? Bạn định vị bản thân thế nào trong bối cảnh đó?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Đây là câu hỏi hành vi về **quan điểm và sự tự định vị** — không có 'câu chuyện sự cố' bắt buộc, nhưng bạn vẫn nên neo quan điểm vào trải nghiệm thật để thuyết phục. Có thể dùng STAR linh hoạt (lấy bối cảnh nghề nghiệp làm Situation). Mẫu tham khảo:

- **Situation (Bối cảnh):** "Tôi đã trải qua giai đoạn AI thay đổi cách tôi làm việc mỗi ngày: nó gánh phần lớn việc gõ code cơ học. Điều đó khiến tôi tự hỏi nghiêm túc giá trị của một lập trình viên giờ nằm ở đâu."

- **Task (Nhiệm vụ):** "Tôi cần một quan điểm rõ ràng để định hướng phát triển bản thân, thay vì hoặc hoảng sợ hoặc phủ nhận."

- **Action (Hành động):** "Quan điểm của tôi: AI thay thế *một số tác vụ*, không thay thế *vai trò kỹ sư*. Nó cực giỏi tạo ra văn bản code, nhưng nó không sở hữu trách nhiệm, không hiểu ngữ cảnh nghiệp vụ sâu xa, không ra quyết định đánh đổi kiến trúc dài hạn, và không chịu trách nhiệm khi hệ thống sập lúc 2 giờ sáng. Nút thắt của phần mềm đang dịch chuyển: chi phí *tạo* code giảm mạnh, nên giá trị dồn về *quyết định viết gì, review, hiểu hệ thống và chịu trách nhiệm*. Vì vậy tôi chủ động định vị mình ở phía 'engineer ra quyết định': tôi đầu tư vào tư duy thiết kế, đọc-hiểu hệ thống, đánh giá đầu ra (review), và dùng AI như một bộ khuếch đại năng lực. Tôi cũng giữ thói quen tự làm phần nền tảng để không teo kỹ năng."

- **Result (Kết quả):** "Cách định vị này biến AI từ mối đe dọa thành đòn bẩy: tôi làm được nhiều hơn, ở tầng cao hơn, mà vẫn giữ trách nhiệm. Tôi tin người bị thay thế không phải là 'lập trình viên', mà là lối làm việc chỉ biết gõ code mà không tư duy. Người biết dùng AI như siêu năng lực — nhưng vẫn sở hữu và hiểu code mình ship — sẽ có giá trị hơn bao giờ hết."

Lời khuyên: tránh hai thái cực — vừa không nên hoảng loạn ('AI sẽ lấy mất việc'), vừa không nên phủ nhận ('AI chỉ là đồ chơi'). Quan điểm chín chắn nhất: AI dịch chuyển *nội dung công việc*, người kỹ sư biết thích nghi và giữ trách nhiệm sẽ mạnh hơn. Gắn vào việc bạn đang *chủ động* nâng kỹ năng tầng cao (thiết kế, review, ra quyết định) sẽ rất ghi điểm.

</details>

---

## 14. Cursor, Copilot và Lựa chọn công cụ AI

### 14.1. GitHub Copilot là gì và đâu là thế mạnh chính của nó?

<details>
<summary>👉 Xem câu trả lời</summary>

GitHub Copilot là trợ lý AI lập trình của GitHub, tích hợp trực tiếp vào IDE (VS Code, JetBrains, Visual Studio, Neovim). Ba tính năng cốt lõi:

- **Inline suggestion (autocomplete)**: gợi ý code ngay khi bạn đang gõ, dạng "ghost text" mờ, nhấn `Tab` để chấp nhận. Đây là điểm mạnh nhất.
- **Copilot Chat**: hỏi đáp ngay trong IDE (giải thích code, sửa bug, viết test) mà không cần rời màn hình.
- **Chế độ agent (Copilot agent/Workspace)**: gần đây bổ sung khả năng làm task nhiều bước, sửa nhiều file.

Thế mạnh: gợi ý **nhanh, mượt, ngay tại con trỏ**, ít làm gián đoạn dòng suy nghĩ, "ít rời tay khỏi bàn phím". Rất hợp với viết code lặp lại, boilerplate, hàm nhỏ.

Ví dụ inline suggestion: bạn gõ comment hoặc tên hàm, Copilot điền phần thân:

```ts
// tính tổng các số chẵn trong mảng
function sumEven(nums: number[]): number {
  // ↓ phần dưới là Copilot tự gợi ý
  return nums.filter((n) => n % 2 === 0).reduce((a, b) => a + b, 0);
}
```

Khi nào dùng: lúc bạn đã biết rõ mình muốn viết gì và chỉ cần AI gõ giúp cho nhanh.

</details>

---

### 14.2. Cursor là gì, và những tính năng nào tạo nên thế mạnh của nó?

<details>
<summary>👉 Xem câu trả lời</summary>

Cursor là một **IDE "AI-first"** (fork từ VS Code), nghĩa là AI được tích hợp sâu vào toàn bộ trải nghiệm chỉnh sửa code, chứ không chỉ là một plugin gắn thêm. Các tính năng chính:

| Tính năng | Mô tả |
|-----------|-------|
| **Tab** | Autocomplete đa dòng mạnh, dự đoán cả chuỗi thay đổi tiếp theo (không chỉ một dòng) |
| **Inline edit (`Cmd/Ctrl-K`)** | Bôi đen code rồi ra lệnh sửa bằng ngôn ngữ tự nhiên |
| **Chat** | Hỏi đáp có ngữ cảnh codebase |
| **Composer / Agent** | Sửa nhiều file cùng lúc, chạy lệnh, làm feature đa bước |
| **Index codebase** | Đánh chỉ mục toàn dự án để AI có context rộng |
| **@-mention** | Tham chiếu file/docs/web cụ thể vào prompt |
| **Đa model** | Chọn Claude, GPT... tùy tác vụ |

Thế mạnh: **trải nghiệm IDE tích hợp sâu cộng với context toàn dự án**. Nhờ index codebase và @-mention, Cursor "hiểu" dự án của bạn rộng hơn so với một autocomplete đơn thuần.

Ví dụ inline edit: bôi đen một hàm rồi nhấn `Ctrl-K` và gõ:

```text
Refactor hàm này dùng async/await thay vì .then(), và thêm try/catch
```

Cursor sẽ sửa tại chỗ và hiển thị diff để bạn duyệt.

Khi nào dùng: muốn một môi trường code thường ngày có AI mạnh xuyên suốt, đặc biệt khi cần AI hiểu nhiều file trong dự án.

</details>

---

### 14.3. Claude Code là gì và thế mạnh của nó nằm ở đâu?

<details>
<summary>👉 Xem câu trả lời</summary>

Claude Code là công cụ lập trình **agentic** của Anthropic. Khác với autocomplete, nó hoạt động theo một **vòng lặp agentic**: tự đọc/sửa file, tự chạy lệnh, tự dùng tool, quan sát kết quả rồi lặp lại cho tới khi hoàn thành tác vụ. Có mặt ở nhiều nơi: CLI (terminal-native), extension VS Code/JetBrains, app desktop và web.

Các khả năng đặc trưng:

- **Permission mode**: kiểm soát quyền (hỏi trước khi chạy lệnh/sửa file) để an toàn.
- **Plan mode**: lập kế hoạch trước khi hành động.
- **Slash command, Skills, Subagents**: đóng gói quy trình, chuyên môn hóa, chia việc cho nhiều agent con.
- **Hooks**: tự động chạy script ở các mốc sự kiện (ví dụ format/lint sau khi sửa file).
- **MCP (Model Context Protocol)**: kết nối tool/dữ liệu ngoài.
- **Workflows**: tự động hóa quy trình nhiều bước.

Thế mạnh: **tác vụ đa bước/tự chủ, terminal-native, và khả năng tùy biến + tự động hóa sâu** (hợp với CI/CD, kịch bản lặp lại).

Ví dụ chạy ở CLI:

```bash
# giao một task đa bước, Claude Code tự đọc file, sửa, chạy test
claude "Thêm validation cho endpoint POST /users và viết test, chạy test cho tới khi pass"
```

Khi nào dùng: việc lớn nhiều bước, tự động hóa quy trình, hoặc thao tác mạnh ở terminal.

</details>

---

### 14.4. So sánh ba công cụ: autocomplete vs AI-IDE vs agentic — khác nhau cơ bản ở đâu?

<details>
<summary>👉 Xem câu trả lời</summary>

Ba công cụ đại diện cho ba "tầng" hỗ trợ AI khác nhau, từ hẹp tới rộng về mức tự chủ:

| Tiêu chí | GitHub Copilot | Cursor | Claude Code |
|----------|----------------|--------|-------------|
| **Bản chất** | Trợ lý AI trong IDE | IDE AI-first (fork VS Code) | Agentic coding tool |
| **Tương tác chính** | Inline suggestion khi gõ | Tab + inline edit + Composer | Vòng lặp agentic tự chủ |
| **Context** | Quanh con trỏ + file mở | Index toàn codebase, @-mention | Tự khám phá file/repo, MCP |
| **Mức tự chủ** | Thấp (bạn gõ, nó gợi ý) | Trung bình–cao (Agent sửa nhiều file) | Cao (tự đọc/sửa/chạy lệnh) |
| **Nơi sống** | Trong IDE | Là chính IDE | CLI + IDE + desktop/web |
| **Hợp nhất với** | Gõ code từng dòng nhanh | Code thường ngày + context dự án | Task lớn, tự động hóa, CI |

Cách hình dung ngắn gọn:

- **Copilot** ≈ autocomplete/assistant thông minh **trong** IDE — bạn vẫn là người lái, nó gợi ý.
- **Cursor** ≈ một **AI-IDE** có agent và context toàn dự án — vừa code tay vừa nhờ AI làm cụm việc.
- **Claude Code** ≈ **agentic**, mạnh cho task lớn và tự động hóa — bạn giao việc, nó tự thực thi nhiều bước.

Lưu ý quan trọng: ba thứ này **không loại trừ nhau**. Nhiều dev dùng kết hợp (xem câu về workflow thực chiến).

</details>

---

### 14.5. Với từng loại công việc cụ thể, nên chọn công cụ nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Nguyên tắc: chọn theo **quy mô và tính chất** của việc, chứ không phải "công cụ nào tốt nhất". Bảng quyết định:

| Loại công việc | Công cụ phù hợp | Vì sao |
|----------------|-----------------|--------|
| Gõ từng dòng/hàm nhanh, boilerplate | Copilot hoặc Cursor **Tab** | Autocomplete tại con trỏ, ít gián đoạn |
| Sửa một đoạn theo yêu cầu | Cursor inline edit (`Ctrl-K`) | Sửa tại chỗ, xem diff ngay |
| Sửa nhiều file / một feature | Cursor **Composer** hoặc **Claude Code** | Cần context nhiều file + nhiều bước |
| Tự động hóa quy trình / CI / hooks | **Claude Code** | Hooks, Skills, Workflows, terminal-native |
| Hiểu/khám phá codebase lạ | Cursor (index) hoặc Claude Code | Context rộng, tự đọc repo |
| Hỏi nhanh giải thích code | Copilot Chat / Cursor Chat | Có sẵn trong IDE |

Ví dụ cách diễn đạt khi phỏng vấn:

```text
- "Refactor một module gồm 6 file" → mình dùng Cursor Composer hoặc Claude Code
  vì cần AI thấy được context liên file và thực hiện nhiều bước có kiểm soát.
- "Viết nốt một hàm util mình đã hình dung rõ" → Copilot/Cursor Tab cho nhanh.
- "Chạy migration + sinh test + chạy test trong pipeline" → Claude Code với
  workflow/hooks vì nó terminal-native và tự lặp tới khi pass.
```

Câu trả lời cho thấy bạn hiểu **dùng đúng tool cho đúng tầng việc**, đó là điều nhà tuyển dụng muốn nghe.

</details>

---

### 14.6. Mô tả một AI-native workflow thực chiến: bạn kết hợp các công cụ như thế nào trong một ngày làm việc?

<details>
<summary>👉 Xem câu trả lời</summary>

Điểm mấu chốt: **không có công cụ "thắng tất cả"**, mà ta phối hợp theo từng giai đoạn của công việc. Một workflow kết hợp tiêu biểu:

1. **Khám phá & lập kế hoạch**: dùng agentic tool (ví dụ Claude Code **plan mode**) hoặc Chat của IDE để hiểu yêu cầu, đọc codebase và phác kế hoạch trước khi sửa.
2. **Viết code thường ngày**: ở trong IDE, dùng **autocomplete** (Copilot/Cursor Tab) cho tốc độ gõ và **inline edit** cho các sửa nhỏ.
3. **Cụm thay đổi nhiều file**: giao cho **Cursor Composer** hoặc **Claude Code** làm, rồi **review diff** kỹ trước khi nhận.
4. **Tự động hóa & chất lượng**: dùng **Claude Code hooks/skills** để tự format, lint, chạy test sau mỗi thay đổi; tích hợp vào CI cho task lặp lại.
5. **Luôn có vòng người duyệt**: dù dùng tool nào, dev vẫn đọc diff, chạy test, chịu trách nhiệm code cuối.

```text
Hiểu việc       → Claude Code plan mode / Chat
Gõ nhanh        → Copilot / Cursor Tab
Sửa cụm/feature → Cursor Composer / Claude Code agent
Tự động hóa     → Claude Code hooks + workflows + CI
Review          → con người (luôn luôn)
```

Thông điệp khi phỏng vấn: bạn coi AI là **bộ khuếch đại năng suất có kiểm soát**, biết chỗ nào để AI làm chủ động và chỗ nào người phải nắm.

</details>

---

### 14.7. Khi nào bạn nên tự gõ code thay vì để AI làm?

<details>
<summary>👉 Xem câu trả lời</summary>

AI tăng tốc nhưng không phải lúc nào cũng nên giao phó. Tự gõ (hoặc kiểm soát rất sát) khi:

- **Logic nghiệp vụ cốt lõi / nhạy cảm**: tính tiền, phân quyền, bảo mật, dữ liệu tài chính — sai một li đi một dặm, cần bạn nắm chắc từng dòng.
- **Code bạn chưa hiểu rõ vấn đề**: nếu bản thân chưa hiểu bài toán, để AI viết hộ dễ tạo ra code "trông đúng" nhưng sai bản chất, và bạn không phát hiện được.
- **Đoạn quá ngắn / quá đặc thù**: nhiều khi gõ tay nhanh hơn là viết prompt rồi sửa lại gợi ý.
- **Khi đang học**: tự viết để hiểu sâu; nếu phụ thuộc autocomplete quá sớm sẽ không nắm được nền tảng.
- **Khi AI đi vòng lặp sai**: nếu AI sửa hoài không đúng, dừng lại tự làm thường rẻ hơn là tiếp tục "thúc" nó.

Quan trọng: **bạn luôn chịu trách nhiệm review** kể cả khi AI viết. Hiểu được code AI sinh ra là điều kiện bắt buộc trước khi merge.

```text
Để AI làm:   boilerplate, glue code, test mẫu, refactor cơ học, code mình đã hiểu rõ
Tự kiểm soát: business logic cốt lõi, bảo mật/phân quyền, thuật toán tinh tế,
              phần mình đang học, hoặc khi AI lặp sai nhiều lần
```

Câu này thể hiện sự **chín chắn về kỹ thuật**: dùng AI có chọn lọc, không mù quáng.

</details>

---

### 14.8. Khi dùng công cụ AI ở công ty, cần lưu ý gì về bảo mật và chi phí?

<details>
<summary>👉 Xem câu trả lời</summary>

Đây là góc nhìn "trưởng nhóm/doanh nghiệp", không chỉ kỹ thuật cá nhân. Các điểm cần cân nhắc:

**Bảo mật & dữ liệu:**
- **Code có gửi lên cloud không?** Hầu hết các tool gửi ngữ cảnh code lên server của nhà cung cấp để xử lý. Cần đọc kỹ chính sách dữ liệu.
- **Zero-retention / không dùng để train**: với code nhạy cảm, ưu tiên gói/cấu hình cam kết **không lưu giữ** và **không dùng dữ liệu để huấn luyện**.
- **Không để lộ secret**: tránh để API key, mật khẩu, token nằm trong file mà AI đọc/gửi lên; dùng biến môi trường, file `.env` được ignore, và quét secret.
- **Gói enterprise**: thường có kiểm soát truy cập, audit log, ranh giới dữ liệu rõ ràng hơn bản cá nhân.

**Chi phí:**
- Tính theo **license theo ghế** (per-seat) và/hoặc **theo lượng dùng** (token/usage) tùy công cụ và mô hình.
- Cân nhắc ROI: thời gian tiết kiệm so với chi phí license; chọn model phù hợp (model mạnh đắt hơn — không phải việc nào cũng cần).

**Tuân thủ & quy trình:**
- Tôn trọng chính sách license/bản quyền của code do AI sinh ra.
- Thiết lập **permission/guardrail** (ví dụ Claude Code permission mode) để AI không tự ý chạy lệnh nguy hiểm.

```text
Checklist trước khi dùng AI tool ở công ty:
[ ] Chính sách dữ liệu: code có rời máy không? lưu giữ bao lâu?
[ ] Có cam kết zero-retention / không train trên dữ liệu của ta?
[ ] Secret được loại khỏi context (.env ignore, secret scanning)?
[ ] Dùng gói enterprise (SSO, audit log, kiểm soát truy cập)?
[ ] Mô hình chi phí: per-seat hay theo usage? đã ước tính ROI?
[ ] Có guardrail/permission để giới hạn hành động của agent?
```

Trả lời được câu này cho thấy bạn nghĩ tới **rủi ro tổ chức**, không chỉ "tool nào xịn".

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
