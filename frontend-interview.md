# 🎯 Bộ câu hỏi phỏng vấn Frontend (React / Next.js)

> Tài liệu ôn tập cho vị trí **Frontend Developer**, trọng tâm **React / ReactJS, Next.js** và hệ sinh thái (React Router, TanStack Query, Redux Toolkit, Zustand, React Hook Form, Tailwind, Testing Library...).
> Câu hỏi được nhóm theo chủ đề; mỗi đáp án nằm trong khối **collapse** (`<details>`) — bấm để xem lời giải + ví dụ.
>
> 💡 *Mẹo ôn tập:* Tự trả lời trước khi mở đáp án, rồi diễn đạt lại bằng lời của bạn — phỏng vấn kiểm tra khả năng **giải thích**, không phải đọc thuộc.

## 📚 Mục lục

1. [TypeScript và Immutability trong React](#1-typescript-và-immutability-trong-react)
2. [React căn bản](#2-react-căn-bản)
3. [React Hooks](#3-react-hooks)
4. [Tối ưu hiệu năng React](#4-tối-ưu-hiệu-năng-react)
5. [Quản lý State và Data Fetching](#5-quản-lý-state-và-data-fetching)
6. [React Ecosystem và Thư viện](#6-react-ecosystem-và-thư-viện)
7. [Next.js và Cơ chế Rendering](#7-nextjs-và-cơ-chế-rendering)
8. [HTML CSS và Trình duyệt](#8-html-css-và-trình-duyệt)
9. [Web Networking và Bảo mật Frontend](#9-web-networking-và-bảo-mật-frontend)
10. [System Design cho Frontend](#10-system-design-cho-frontend)
11. [Testing Frontend](#11-testing-frontend)
12. [Bài tập Coding (Live Coding)](#12-bài-tập-coding-live-coding)
13. [Tư duy và Giải quyết vấn đề](#13-tư-duy-và-giải-quyết-vấn-đề)

---

## 1. TypeScript và Immutability trong React

### 1.1. Tính bất biến (immutability) trong React là gì? Phân biệt shallow copy và deep copy.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Immutability nghĩa là khi cập nhật state mình không sửa trực tiếp object/array cũ mà tạo ra một bản mới. Lý do là React phát hiện thay đổi bằng cách so sánh tham chiếu chứ không so sánh từng giá trị bên trong — nếu mình mutate trực tiếp thì tham chiếu vẫn y nguyên, React tưởng không có gì đổi nên không re-render, UI bị sai. Về copy thì có hai mức: shallow copy như spread `{...obj}` hay `[...arr]` chỉ tạo mới ở cấp ngoài cùng, còn object lồng bên trong vẫn dùng chung tham chiếu cũ, nên sửa cái lồng là dính cả bản gốc; deep copy như `structuredClone` thì sao chép toàn bộ cây nên tách biệt hẳn. Trong thực tế mình ưu tiên giữ state nông và phẳng để chỉ cần shallow copy ở đúng cấp cần đổi, vì deep copy tốn chi phí; còn khi state lồng sâu thì mình dùng Immer như trong Redux Toolkit cho gọn. Một bẫy mình tránh là `JSON.parse(JSON.stringify())` để deep copy, vì nó làm mất `Date`, `Map`, `Set`, `undefined` và hàm."

---

- React phát hiện thay đổi bằng so sánh tham chiếu (`Object.is`). Mutate trực tiếp → tham chiếu không đổi → không re-render. Vì vậy phải tạo object/array MỚI.
- Shallow copy: chỉ sao chép cấp một, object/array lồng vẫn dùng chung tham chiếu (`{...obj}`, `[...arr]`, `Object.assign`, `slice`).
- Deep copy: sao chép toàn bộ cây, không chia sẻ tham chiếu (`structuredClone`, lodash `cloneDeep`).
- Lưu ý: deep copy tốn chi phí; nên giữ state nông/phẳng hoặc dùng Immer (Redux Toolkit). Bẫy: `JSON.parse(JSON.stringify())` mất `Date`, `Map`, `Set`, `undefined`, hàm.

```ts
const state = { user: { name: 'An' } };
const shallow = { ...state };
shallow.user.name = 'Binh';       // state.user.name cũng đổi thành 'Binh'!
const deep = structuredClone(state); // tách biệt hoàn toàn
```

</details>

---

### 1.2. Bạn gõ kiểu cho props của một component và cho giá trị trả về của custom hook như thế nào? Có những lưu ý gì?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Với props thì mình định nghĩa một `interface` hoặc `type` riêng rồi gõ thẳng vào tham số props của function component, chứ mình tránh `React.FC` vì nó tự ép thêm `children` vào dù component không nhận children. Còn custom hook bản chất chỉ là một hàm thường, nên mình gõ tham số và kiểu trả về như hàm bình thường thôi. Điểm cần lưu ý nhất là khi hook trả về tuple kiểu như `useState` thì phải thêm `as const`, không thì TypeScript sẽ suy ra thành mảng union và lúc destructuring sẽ mất kiểu; còn nếu trả về object thì không cần. Ngoài ra mình hay gõ event đúng kiểu như `React.ChangeEvent<HTMLInputElement>`, và khi muốn component nhận luôn các prop gốc của thẻ HTML thì dùng `React.ComponentProps<'input'>` để khỏi liệt kê tay. Nói chung mục tiêu là để autocomplete và type-check bắt lỗi giúp mình, nên mình tránh `any` kể cả khi tích hợp thư viện ngoài."

---

- Props: định nghĩa `interface`/`type` rồi gõ trực tiếp tham số props (tránh `React.FC` vì ép `children` ngầm).
- Custom hook là hàm → gõ tham số và kiểu trả về như hàm thường. Hook trả tuple (giống `useState`) cần `as const` để giữ đúng kiểu tuple.
- Lưu ý: gõ event đúng kiểu (`React.ChangeEvent<HTMLInputElement>`); mở rộng prop gốc qua `React.ComponentProps<'input'>`; generic component cho danh sách.
- Bẫy: trả object không cần `as const`, nhưng tuple thì rất cần (không thì destructuring mất kiểu). Tránh `any` cho thư viện ngoài.

```tsx
interface CardProps { title: string; onClose?: () => void; children: React.ReactNode; }

function useToggle(initial = false) {
  const [on, setOn] = useState(initial);
  const toggle = useCallback(() => setOn(p => !p), []);
  return [on, toggle] as const; // readonly [boolean, () => void]
}
```

</details>

## 2. React căn bản

### 2.1. JSX là gì và nó được biên dịch (compile) ra cái gì?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"JSX là cú pháp giống HTML viết ngay trong JavaScript, nhưng nó chỉ là 'syntactic sugar' chứ trình duyệt không hiểu trực tiếp. Lúc build, Babel hoặc SWC sẽ biên dịch nó thành lời gọi hàm — với React mới là `_jsx` từ `react/jsx-runtime` (nên từ React 17 trở đi không cần `import React` nữa), còn cách cũ là `React.createElement`. Kết quả trả về không phải DOM thật mà là một React element, tức là một object JS thuần mô tả UI gồm `type`, `props`, `key`. Hiểu được điều này thì giải thích được nhiều quy tắc: mình viết `className` thay vì `class` vì `class` là từ khóa JS; component phải viết hoa chữ đầu vì chữ thường thì JSX hiểu là thẻ HTML dạng chuỗi còn chữ hoa thì hiểu là tham chiếu tới biến component; và trong dấu `{}` chỉ nhúng được biểu thức chứ không nhúng được `if` hay `for`. Tóm lại JSX chỉ là cách viết dễ đọc, còn bản chất vẫn là gọi hàm tạo object."

---

- JSX là "syntactic sugar", không phải HTML cũng không phải JS chuẩn; trình duyệt không hiểu trực tiếp.
- Babel/SWC biên dịch JSX thành lời gọi hàm: `React.createElement(...)` (classic runtime) hoặc `_jsx`/`_jsxs` từ `react/jsx-runtime` (automatic runtime, mặc định React 17+, không cần `import React`).
- Kết quả là **React element** — một object JS thuần `{ type, props, key, ... }` mô tả UI, chưa phải DOM thật.
- Suy ra: dùng `className`/`htmlFor` (vì `class`/`for` là từ khóa JS); component phải **viết hoa chữ đầu** (`<Button />` → tham chiếu biến, `<button />` → chuỗi thẻ HTML); mỗi `{...}` là một biểu thức (không nhúng được `if`/`for`).

```jsx
// Babel (automatic runtime) biên dịch:
const el = _jsx("button", { className: "btn", onClick: handleClick, children: "Lưu" });
```

</details>

---

### 2.2. Virtual DOM là gì? Nó giúp giải quyết vấn đề gì?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Virtual DOM là một bản sao nhẹ của UI dưới dạng cây object JavaScript được React giữ trong bộ nhớ. Khi state hoặc props đổi, React dựng một cây mới rồi so sánh (diff) với cây cũ, từ đó tính ra tập thay đổi nhỏ nhất cần áp lên DOM thật, tránh việc cập nhật bừa gây reflow/repaint thừa. Có một hiểu lầm hay bị hỏi là Virtual DOM giúp app nhanh hơn — thực ra một DOM thật được tối ưu thủ công vẫn có thể nhanh hơn; giá trị thật của Virtual DOM nằm ở chỗ nó cho mình lập trình theo kiểu khai báo, nghĩa là mình chỉ mô tả 'UI trông như thế nào ứng với state này', còn việc đồng bộ với DOM và gom các cập nhật lại để React lo. Nhờ vậy code dễ đọc, dễ bảo trì hơn nhiều so với tự tay thao tác DOM. Từ React 16 với kiến trúc Fiber, quá trình render này còn có thể bị gián đoạn và ưu tiên để phục vụ concurrent rendering."

---

- Virtual DOM là bản biểu diễn nhẹ của UI dưới dạng cây object JS (cây React element) giữ trong bộ nhớ.
- Cơ chế: state/props đổi → dựng cây mới → diff với cây cũ → áp **tập thay đổi tối thiểu** lên DOM thật (tránh reflow/repaint thừa).
- Bẫy hay hỏi: Virtual DOM **không tự làm app nhanh hơn** DOM thật tối ưu thủ công. Giá trị thật là **mô hình khai báo (declarative)**: ta mô tả UI theo state, React lo đồng bộ DOM và gom (batch) cập nhật.
- Từ React 16+ (kiến trúc Fiber), cập nhật có thể bị gián đoạn/ưu tiên (interruptible) để phục vụ concurrent.

</details>

---

### 2.3. Reconciliation và thuật toán diffing của React hoạt động thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Reconciliation là quá trình React đối chiếu cây element mới với cây cũ để quyết định cần cập nhật gì lên DOM. Nếu so sánh hai cây một cách tổng quát thì độ phức tạp là O(n³) — quá đắt — nên React dùng vài heuristic để đưa về xấp xỉ tuyến tính. Giả định quan trọng nhất là: nếu `type` của một node đổi thì React coi như khác hoàn toàn, hủy luôn cả cây con cũ kèm state rồi dựng lại từ đầu; còn nếu cùng `type` ở cùng vị trí thì nó giữ lại node DOM và chỉ cập nhật những prop đã đổi. Giả định thứ hai là dùng `key` để nhận diện phần tử ổn định trong danh sách khi thứ tự thay đổi. Một điểm hay gây bất ngờ là cùng type ở cùng vị trí thì state không bị reset vì React tái dùng node đó; nên khi mình muốn ép một component reset hẳn — ví dụ form đổi sang user khác cần xóa hết nội dung cũ — thì mình đổi `key` để buộc nó remount."

---

- **Reconciliation**: quá trình đối chiếu cây element mới với cũ để quyết định cập nhật DOM. So sánh tổng quát là O(n³) nên React dùng heuristic gần O(n).
- Giả định 1: **khác `type` → coi như khác hoàn toàn** → hủy cả cây con cũ (state mất), dựng mới. Cùng `type` → giữ node DOM, chỉ cập nhật props đã đổi.
- Giả định 2: **`key`** giúp nhận diện phần tử ổn định trong danh sách.
- Bẫy: cùng `type` ở cùng vị trí thì state **không** reset (React tái dùng node). Muốn ép reset/remount → đổi `key`.

```jsx
<UserForm key={userId} userId={userId} /> // đổi userId -> remount, form reset
```

</details>

---

### 2.4. Controlled component và uncontrolled component khác nhau thế nào? Cho ví dụ.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Khác biệt cốt lõi giữa hai loại là ai giữ nguồn sự thật cho giá trị của input. Controlled component thì React state giữ giá trị: input nhận `value` từ state và mỗi lần gõ lại gọi `onChange` để cập nhật state. Đây là cách mặc định mình dùng khi cần validate ngay khi gõ, format giá trị, hoặc đồng bộ nhiều field với nhau. Uncontrolled component thì để DOM tự giữ giá trị, mình chỉ đọc qua `ref` khi cần (và khởi tạo bằng `defaultValue`); cách này hợp với form đơn giản, khi tích hợp thư viện DOM ngoài, hoặc khi muốn tránh re-render mỗi lần gõ phím — React Hook Form đi theo hướng này nên rất nhanh với form lớn. Bẫy hay gặp là truyền `value` mà quên `onChange` khiến input thành read-only và React cảnh báo; và phải tránh để một input lúc controlled lúc uncontrolled, tức `value` lúc thì `undefined` lúc thì có giá trị."

---

- Khác biệt cốt lõi: **ai giữ "nguồn sự thật" cho giá trị input**.
- **Controlled**: React state giữ giá trị; input nhận `value` từ state, mỗi lần gõ gọi `onChange`. Đây là mặc định; dùng khi cần validate tức thì, format khi gõ, đồng bộ field.
- **Uncontrolled**: DOM tự giữ giá trị, đọc qua `ref` khi cần (dùng `defaultValue` để khởi tạo). Hợp form đơn giản, tích hợp DOM/thư viện ngoài, tránh re-render mỗi phím (React Hook Form theo hướng này).
- Bẫy: truyền `value` mà quên `onChange` → input read-only + cảnh báo. Đừng để input "lúc controlled lúc uncontrolled" (`value` lúc `undefined` lúc có).

```jsx
// Controlled
<input value={name} onChange={(e) => setName(e.target.value)} />
// Uncontrolled
<input ref={inputRef} defaultValue="" /> // đọc inputRef.current.value khi submit
```

</details>

---

### 2.5. Các giai đoạn vòng đời (lifecycle) của class component ánh xạ sang hooks như thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Với hooks thì không còn các lifecycle method tách rời như class nữa; thay vào đó `useEffect` mô tả việc đồng bộ component với hệ thống bên ngoài theo dependency. Ánh xạ thì đại khái: `componentDidMount` tương ứng `useEffect` với mảng deps rỗng `[]`, `componentDidUpdate` là `useEffect` có deps, `componentWillUnmount` là hàm cleanup `return` bên trong effect, còn `shouldComponentUpdate` thì gần với `React.memo` kết hợp `useMemo`/`useCallback`. Nhưng điều mình muốn nhấn mạnh là tư duy đã đổi: thay vì nghĩ 'code này chạy ở giai đoạn nào', mình nghĩ 'effect này đồng bộ cái gì với state/props nào', và tách thành nhiều `useEffect` theo từng mối quan tâm thay vì nhồi hết vào một chỗ như `componentDidMount` ngày xưa. Một lưu ý thực tế là StrictMode ở dev của React 18 cố tình chạy mount-unmount-mount để lộ ra những effect thiếu cleanup, nên mình luôn viết cleanup cho đúng."

---

- Hooks không có lifecycle method riêng; `useEffect` mô tả **đồng bộ với hệ thống ngoài** theo dependency.

| Class | Hooks |
|---|---|
| `componentDidMount` | `useEffect(() => {...}, [])` |
| `componentDidUpdate` | `useEffect(() => {...}, [deps])` |
| `componentWillUnmount` | cleanup `return () => {...}` trong `useEffect` |
| `shouldComponentUpdate` | `React.memo` + `useMemo`/`useCallback` |

- Tư duy: không nghĩ "chạy giai đoạn nào" mà "effect này đồng bộ cái gì với state/props nào"; tách thành **nhiều `useEffect`** theo mối quan tâm.
- React 18 StrictMode (dev) chạy effect mount-unmount-mount để lộ cleanup thiếu → cleanup phải đúng.

```jsx
useEffect(() => {
  const id = setInterval(tick, 1000);
  return () => clearInterval(id); // cleanup
}, []);
```

</details>

---

### 2.6. Tại sao không được mutate (sửa trực tiếp) state trong React?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Lý do là React quyết định có re-render hay không bằng cách so sánh tham chiếu, tức so sánh nông qua `Object.is`. Nếu mình mutate trực tiếp như `state.items.push(...)` thì tham chiếu của object/array vẫn giữ nguyên, React thấy 'cũ vẫn bằng mới' nên không re-render, dẫn đến UI không cập nhật dù dữ liệu thực tế đã đổi. Mutate còn phá luôn các tối ưu dựa trên so sánh nông như `React.memo` hay dependency của `useMemo`/`useEffect`, làm chúng hoạt động sai. Vì vậy quy tắc là luôn tạo object/array mới khi cập nhật — spread cho cấp nông, còn lồng sâu thì mình dùng Immer (Redux Toolkit có sẵn) cho đỡ cực. Một bẫy liên quan là khi cập nhật dựa trên giá trị trước đó và gọi nhiều lần trong cùng một event, mình phải dùng dạng hàm `setCount(prev => prev + 1)` để tránh đọc phải giá trị cũ bị kẹt trong closure."

---

- React quyết định re-render bằng **so sánh tham chiếu** (so sánh nông, `Object.is`). Mutate trực tiếp → tham chiếu không đổi → React nghĩ "không đổi" → **không re-render**, UI lỗi.
- Còn phá các tối ưu dựa trên so sánh nông (`React.memo`, deps của `useMemo`/`useEffect`).
- Quy tắc: luôn tạo **object/array mới** (immutable update). Lồng sâu dùng Immer (Redux Toolkit dùng sẵn).
- Bẫy: cập nhật dựa trên giá trị trước (gọi liên tiếp) → dùng **dạng hàm** `setCount(prev => prev + 1)` để tránh giá trị cũ (stale) trong closure.

```jsx
setTodos((prev) => [...prev, newTodo]);            // thêm
setUser((prev) => ({ ...prev, name: "An" }));      // sửa 1 field
```

</details>

---

### 2.7. Synthetic event trong React là gì? Nó khác native event ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"SyntheticEvent là một lớp bọc của React quanh native event của trình duyệt, mục đích là chuẩn hóa hành vi để API đồng nhất giữa các trình duyệt — mình dùng `e.target`, `e.preventDefault()`, `e.stopPropagation()` như bình thường, còn nếu cần event gốc thì truy cập qua `e.nativeEvent`. Điểm hay ở chỗ React không gắn listener lên từng node mà dùng event delegation, gắn một listener ở gốc rồi phân phối; từ React 17 trở đi listener này gắn vào container gốc của app chứ không phải `document` như React 16 — điều này quan trọng khi mình chạy nhiều app React trên cùng một trang để chúng không đạp lên nhau. Tên handler thì viết camelCase như `onClick`, `onChange`. Hai điểm cập nhật cần nhớ: từ React 17 không còn event pooling nên không cần gọi `e.persist()` nữa; và `onChange` của React bắn mỗi lần gõ giống sự kiện `input` của DOM, chứ không đợi blur như `change` native."

---

- `SyntheticEvent` là wrapper đa trình duyệt quanh native event → hành vi đồng nhất, API thống nhất (`e.target`, `e.preventDefault()`, `e.stopPropagation()`); native event truy cập qua `e.nativeEvent`.
- React **không gắn listener lên từng node** mà dùng **event delegation** gắn ở gốc. Từ **React 17+, listener gắn vào container gốc của app** (không phải `document` như React 16) → quan trọng khi chạy nhiều app React trên một trang.
- Tên handler camelCase: `onClick`, `onChange`, `onSubmit`.
- **React 17+ không còn event pooling** → không cần `e.persist()` nữa (đó là chuyện React 16).
- `onChange` của React bắn mỗi lần gõ (giống `input` của DOM), không phải khi blur như native `change`.

</details>

---

### 2.8. `children` là gì và component composition (kết hợp component) giải quyết vấn đề gì?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"`children` là một prop đặc biệt, chứa phần nội dung nằm giữa thẻ mở và thẻ đóng của component. Nhờ nó mà một component có thể bọc và hiển thị nội dung tùy ý mà không cần biết trước nội dung đó là gì — đây chính là nền tảng của composition. React khuyến nghị ưu tiên composition hơn kế thừa: thay vì tạo nhiều biến thể component bằng inheritance, mình lắp ghép các component nhỏ lại với nhau. Lợi ích lớn là tránh được prop drilling và component cha không cần biết chi tiết bên trong con, vì cha chỉ 'chừa chỗ' qua `children` hoặc qua các prop nhận JSX kiểu slot. Trong thực tế mình dùng cách này rất nhiều khi xây design system — những thứ như `Modal`, `Card`, `Layout`, `Button` đều có phần khung cố định còn phần ruột thì nơi sử dụng tự quyết định, nên cùng một component dùng lại được ở rất nhiều chỗ mà không phải sửa."

---

- `children` là prop đặc biệt chứa **nội dung giữa thẻ mở và đóng**; cho phép component bọc và hiển thị nội dung tùy ý mà không cần biết trước — nền tảng của **composition**.
- React khuyến nghị **ưu tiên composition hơn kế thừa**: lắp ghép component nhỏ thay vì tạo biến thể bằng inheritance.
- Composition giúp tránh prop drilling và cha không cần biết chi tiết con: cha "chừa chỗ" qua `children` hoặc props là JSX ("slot").
- Thực tế: tạo component dùng chung như `Modal`, `Card`, `Layout`, `Button` trong design system — "khung" cố định, "ruột" do nơi dùng quyết định.

```jsx
function Layout({ sidebar, content }) {
  return <div className="layout"><aside>{sidebar}</aside><main>{content}</main></div>;
}
<Layout sidebar={<Nav />} content={<Dashboard />} />
```

</details>

## 3. React Hooks

### 3.1. `useState` hoạt động như thế nào? Khi nào nên dùng "functional update" và "lazy initialization"?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"useState cho mình một cặp state và hàm setState, và giá trị này được React giữ lại qua mỗi lần render chứ không bị tạo lại. Có hai điểm mình hay nhấn. Thứ nhất là functional update, tức gọi setState(prev => ...): mình dùng khi giá trị mới phụ thuộc vào giá trị cũ, ví dụ tăng count nhiều lần trong cùng một event hoặc cập nhật trong một interval đăng ký một lần. Nếu viết setCount(count + 1) hai lần liên tiếp thì cả hai đều đọc cùng một count cũ nên chỉ tăng một, đó chính là bẫy stale state. Thứ hai là lazy initialization: khi giá trị khởi tạo phải tính nặng hay đọc localStorage, mình truyền một HÀM vào useState(() => ...) để nó chỉ chạy đúng một lần ở render đầu; nếu viết useState(expensiveCompute()) thì hàm đó chạy lại mỗi render dù kết quả bị bỏ đi. Nói ngắn gọn, functional update để tránh giá trị cũ, còn lazy init để tránh tính lại vô ích."

---

- `useState` trả về cặp `[state, setState]`, giữ giá trị xuyên suốt các lần render.
- **Functional update** (`setState(prev => ...)`): dùng khi giá trị mới phụ thuộc giá trị cũ; tránh stale state, cần khi cập nhật nhiều lần trong một event hoặc trong `setInterval`/handler đăng ký một lần.
- **Lazy init** (`useState(() => expensiveCompute())`): truyền HÀM, chỉ chạy ở lần render đầu tiên, tránh tính toán nặng lặp lại.
- Bẫy: `useState(expensiveCompute())` gọi hàm ở MỌI render. Phải truyền `useState(expensiveCompute)`.

```tsx
// Sai: cùng `count` cũ -> chỉ tăng 1
setCount(count + 1); setCount(count + 1);
// Đúng: functional update -> tăng đủ
setCount(prev => prev + 1); setCount(prev => prev + 1);
// Lazy init: đọc localStorage 1 lần
const [token] = useState(() => localStorage.getItem('token') ?? '');
```

</details>

---

### 3.2. Mảng dependency của `useEffect` có ý nghĩa gì? Phân biệt 3 trường hợp: không truyền, mảng rỗng `[]`, và có dependency.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Mảng dependency là cách mình nói cho React biết effect này phụ thuộc vào những giá trị nào. Sau mỗi render React so sánh các dependency bằng Object.is, nếu có cái nào đổi thì nó chạy cleanup của lần trước rồi chạy lại effect. Có ba trường hợp: không truyền mảng thì effect chạy sau MỖI render, rất dễ vô tình gây vòng lặp; truyền mảng rỗng [] thì chỉ chạy một lần sau mount và cleanup khi unmount, hợp với việc đăng ký một lần như mở kết nối; còn truyền [a, b] thì chạy sau mount và mỗi khi a hoặc b đổi, đây là trường hợp thường dùng nhất. Bẫy lớn nhất là cố tình bỏ bớt dependency cho đỡ chạy lại, nó tạo stale closure đọc nhầm giá trị cũ. Ngược lại nếu một object hay function tạo mới mỗi render mà lọt vào deps thì effect chạy liên tục, lúc đó mình bọc bằng useMemo hoặc useCallback. Quy tắc của mình là liệt kê đầy đủ và để eslint-plugin-react-hooks bắt lỗi."

---

- React so sánh dependency bằng `Object.is` sau mỗi render; nếu đổi thì chạy cleanup rồi chạy lại effect.
- Bẫy: bỏ bớt dependency tạo stale closure; nếu object/function tạo mới mỗi render lọt vào deps thì effect chạy vô tận — dùng `useCallback`/`useMemo`.
- Quy tắc: liệt kê đầy đủ mọi biến reactive (theo `eslint-plugin-react-hooks`).

| Cách viết | Khi nào chạy |
| --- | --- |
| `useEffect(fn)` | Sau MỖI render |
| `useEffect(fn, [])` | Một lần sau mount + cleanup khi unmount |
| `useEffect(fn, [a, b])` | Sau mount, và khi `a`/`b` đổi |

</details>

---

### 3.3. Cleanup function trong `useEffect` để làm gì? Cho ví dụ thực tế.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Cleanup là hàm mà mình return ra từ trong useEffect, và React sẽ gọi nó TRƯỚC khi chạy effect lần kế tiếp và cả khi component unmount. Mục đích là dọn dẹp những gì effect đã tạo ra: hủy đăng ký listener, đóng kết nối, clear timer, để tránh memory leak và tránh chồng chéo side effect. Điểm hay là thứ tự khi dependency đổi luôn là cleanup cũ chạy trước rồi mới tới effect mới, nhờ vậy mình luôn chỉ có đúng một thứ đang active, ví dụ đổi roomId thì nó tự ngắt phòng cũ rồi mới nối phòng mới. Bẫy kinh điển là quên cleanup, mỗi lần effect chạy lại cộng thêm một listener hoặc một kết nối, dần dần callback bị gọi nhiều lần và rò rỉ bộ nhớ. Mình hình dung đơn giản: effect mở cái gì thì cleanup phải đóng đúng cái đó."

---

- Hàm `return` từ effect là cleanup; chạy TRƯỚC effect kế tiếp VÀ khi unmount.
- Mục đích: hủy đăng ký, dọn tài nguyên, tránh memory leak và side effect chồng chéo.
- Thứ tự khi deps đổi: cleanup cũ -> chạy effect mới (luôn chỉ 1 kết nối active).
- Bẫy: quên cleanup khiến mỗi render cộng thêm handler -> gọi nhiều lần, rò rỉ bộ nhớ.

```tsx
useEffect(() => {
  const conn = createConnection(roomId);
  conn.connect();
  return () => conn.disconnect(); // dọn kết nối cũ
}, [roomId]);
```

</details>

---

### 3.4. Trong React StrictMode (dev), tại sao effect chạy 2 lần? Điều này nói lên vấn đề gì về code của bạn?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Trong StrictMode ở môi trường dev, React cố ý chạy effect theo trình tự mount rồi cleanup rồi mount lại một lần nữa. Đây là một cái bẫy có chủ đích để lộ ra những effect không idempotent hoặc thiếu cleanup, và nó KHÔNG xảy ra ở production. Cho nên nếu effect của mình chạy hai lần mà sinh lỗi, ví dụ tạo trùng bản ghi hay listener bị nhân đôi, thì đó là dấu hiệu code mình thiếu cleanup chứ không phải React bị lỗi. Cách xử lý đúng là viết effect sao cho chạy lại vẫn an toàn và bổ sung cleanup tương ứng, ví dụ mở kết nối thì return hàm disconnect, như vậy dù mount-cleanup-mount thì cuối cùng vẫn chỉ còn đúng một kết nối. Mình không bao giờ tắt StrictMode để giấu lỗi, vì Next.js App Router bật nó mặc định ở dev và nó thực sự giúp mình phát hiện bug sớm."

---

- `<StrictMode>` ở dev cố ý chạy: mount -> cleanup -> mount, để phát hiện effect không idempotent / thiếu cleanup. KHÔNG xảy ra ở production.
- Nếu effect lỗi khi chạy 2 lần (tạo bản ghi 2 lần, listener nhân đôi) -> dấu hiệu thiếu cleanup, không phải lỗi React.
- Next.js App Router bật StrictMode mặc định ở dev. Cách đúng: viết effect idempotent + cleanup, không tắt StrictMode.

```tsx
// Sửa: thêm cleanup -> mount/cleanup/mount vẫn còn 1 kết nối
useEffect(() => {
  const conn = createConnection();
  conn.connect();
  return () => conn.disconnect();
}, []);
```

</details>

---

### 3.5. Race condition khi fetch dữ liệu trong `useEffect` là gì? Làm sao để xử lý?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Race condition xảy ra khi mình fetch trong useEffect mà dependency đổi nhanh, ví dụ người dùng gõ ô search liên tục hoặc đổi userId nhanh. Mỗi lần đổi là bắn một request, nhưng các response không nhất thiết về đúng thứ tự đã gửi, nên một response cũ về muộn có thể ghi đè lên kết quả mới và làm UI hiển thị sai dữ liệu. Cách xử lý phổ biến là dùng một cờ ignore đặt trong cleanup: khi deps đổi, cleanup set ignore = true, và trước khi setState mình kiểm tra nếu ignore thì bỏ qua, nhờ vậy response cũ không còn ghi vào state nữa. Tốt hơn nữa là kết hợp AbortController để hủy hẳn request cũ, đỡ tốn mạng. Còn trong thực tế dự án mình ưu tiên dùng TanStack Query với useQuery, vì nó tự lo race condition, caching, dedupe luôn nên mình không phải tự tay viết những đoạn này."

---

- Khi deps đổi nhanh (gõ search, đổi `userId`), nhiều request trả về không đúng thứ tự; response cũ ghi đè kết quả mới -> UI sai.
- Xử lý: cờ `ignore` trong cleanup, và/hoặc `AbortController` để hủy request cũ.
- Thực tế nên dùng **TanStack Query** (`useQuery`): tự xử lý race, caching, dedupe.

```tsx
useEffect(() => {
  let ignore = false;
  const c = new AbortController();
  (async () => {
    const data = await (await fetch(url, { signal: c.signal })).json();
    if (!ignore) setUser(data);
  })();
  return () => { ignore = true; c.abort(); };
}, [userId]);
```

</details>

---

### 3.6. `useRef` dùng để làm gì? Nó khác `useState` ở điểm nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"useRef trả về một object dạng { current: ... } và cái object này ổn định, không đổi qua các lần render. Mình dùng nó cho hai việc chính: một là tham chiếu tới DOM node, ví dụ để focus một input; hai là lưu một giá trị mutable nhưng KHÔNG liên quan tới việc hiển thị, kiểu id của timer, instance của một thư viện, hay giá trị của lần trước. Khác biệt cốt lõi với useState là khi mình đổi ref.current thì React không re-render và giá trị cập nhật ngay lập tức, còn setState thì luôn kéo theo re-render và cập nhật mang tính bất đồng bộ, phải đợi render kế tiếp mới thấy. Bẫy hay gặp là dùng ref.current cho dữ liệu cần hiển thị ra UI: vì nó không trigger render nên màn hình sẽ không cập nhật, dữ liệu nào cần hiện lên thì phải để ở state."

---

- Trả về object `{ current: ... }` ổn định qua các render. Dùng để: (1) tham chiếu DOM node, (2) lưu giá trị mutable không liên quan render (id timer, giá trị trước).
- Khác cốt lõi: đổi `ref.current` KHÔNG re-render và cập nhật ngay (đồng bộ); `setState` thì re-render và bất đồng bộ.
- Bẫy: đừng dùng `ref.current` cho logic hiển thị — dữ liệu cần hiện UI phải ở state.

| | `useState` | `useRef` |
| --- | --- | --- |
| Re-render khi đổi | Có | Không |
| Cập nhật | Bất đồng bộ | Ngay lập tức |
| Dùng cho | Dữ liệu UI | DOM, giá trị "ngầm" |

</details>

---

### 3.7. Khi nào nên chọn `useReducer` thay vì `useState`?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Mình chọn useReducer khi state bắt đầu phức tạp: nhiều trường liên quan tới nhau, có nhiều cách chuyển trạng thái khác nhau, hoặc khi mình muốn truyền một dispatch xuống sâu mà giữ được tham chiếu ổn định, không phải truyền cả đống setState. Còn state đơn giản, độc lập như một boolean hay một chuỗi thì useState là đủ và gọn hơn. Quy tắc thực dụng của mình là khi thấy nhiều setState lúc nào cũng phải gọi chung với nhau, hoặc logic cập nhật state đang rải ra nhiều chỗ, thì gom hết vào một reducer để tập trung logic về một nơi. Một lợi điểm nữa là reducer là hàm thuần, nhận state và action rồi trả state mới, nên rất dễ unit test mà không cần render component. Về bản chất useReducer không mạnh hơn useState, nó chỉ giúp tổ chức logic chuyển trạng thái rõ ràng và dễ bảo trì hơn khi state phức tạp."

---

- Dùng `useReducer` khi: state phức tạp nhiều trường liên quan, nhiều cách chuyển trạng thái (nhiều action), hoặc cần truyền `dispatch` (ổn định tham chiếu) xuống sâu.
- Dùng `useState` cho state đơn giản, độc lập.
- Quy tắc: nhiều `setState` luôn gọi cùng nhau hoặc logic lan ra nhiều chỗ -> gom thành reducer. Reducer là hàm thuần nên dễ unit test.

```tsx
function reducer(s, a) {
  switch (a.type) {
    case 'inc': return { ...s, count: s.count + s.step };
    case 'setStep': return { ...s, step: a.step };
  }
}
const [state, dispatch] = useReducer(reducer, { count: 0, step: 1 });
```

</details>

---

### 3.8. Custom hook là gì? Hãy viết một hook `useDebounce`.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Custom hook đơn giản là một hàm tên bắt đầu bằng use, bên trong gọi các hook khác để đóng gói lại một mảng logic stateful và tái sử dụng được. Điểm quan trọng cần nói rõ là custom hook chia sẻ LOGIC chứ không chia sẻ state: hai component cùng dùng một hook thì mỗi component có state riêng biệt, không dùng chung. Với useDebounce, ý tưởng là trả về một giá trị chỉ cập nhật sau khi giá trị nguồn ngừng đổi trong khoảng delay, rất hợp cho ô search để khỏi gọi API mỗi phím gõ. Cách làm là mỗi lần value đổi thì set một timeout để cập nhật giá trị debounced, và mấu chốt nằm ở cleanup: trước khi chạy effect lần sau, clearTimeout hủy bỏ timer cũ. Nhờ vậy chừng nào người dùng còn gõ liên tục thì timer cứ bị reset, chỉ khi họ dừng đủ lâu giá trị mới thực sự được cập nhật."

---

- Custom hook: hàm bắt đầu bằng `use`, dùng hook khác để đóng gói và tái sử dụng stateful logic. Chia sẻ LOGIC, không chia sẻ state (mỗi component có state riêng).
- `useDebounce` trả về giá trị chỉ cập nhật sau khi nguồn ngừng đổi trong `delay`; cốt lõi nằm ở `clearTimeout` trong cleanup.

```tsx
function useDebounce<T>(value: T, delay = 500): T {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id); // hủy timer cũ khi value đổi
  }, [value, delay]);
  return debounced;
}
```

</details>

---

### 3.9. Hãy viết các custom hook `usePrevious` và `useToggle`.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Đây là hai custom hook nhỏ mà mình hay dùng. usePrevious dùng để lấy giá trị của lần render trước. Mẹo nằm ở chỗ useEffect chạy SAU khi render xong, nên trong thân render hiện tại ref.current vẫn còn giữ giá trị cũ, mình return nó ra trước rồi mới để effect ghi giá trị mới vào ref cho lần sau dùng. useToggle thì đóng gói một boolean kèm hàm đảo trạng thái và hàm set, rất tiện cho mở đóng modal hay sidebar. Mình dùng functional update v => !v trong useCallback để hàm toggle ổn định tham chiếu, và return mảng kết thúc bằng as const để TypeScript suy ra đúng kiểu tuple, nhờ vậy khi destructuring ra thì từng phần tử vẫn giữ đúng kiểu thay vì bị gộp thành union. Cả hai hook này minh họa rõ tư duy tách logic tái dùng ra khỏi component."

---

- `usePrevious`: tận dụng `useEffect` chạy SAU render, nên `ref.current` còn giữ giá trị cũ trong render hiện tại.
- `useToggle`: đóng gói boolean + hàm đảo + set; trả về `as const` để TypeScript suy ra tuple; dùng functional update để hàm ổn định.

```tsx
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();
  useEffect(() => { ref.current = value; }, [value]);
  return ref.current; // giá trị lần trước
}

function useToggle(initial = false) {
  const [value, setValue] = useState(initial);
  const toggle = useCallback(() => setValue(v => !v), []);
  return [value, toggle, setValue] as const;
}
```

</details>

---

### 3.10. Rules of Hooks gồm những quy tắc nào? Tại sao bắt buộc phải tuân theo?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Rules of Hooks có hai quy tắc chính. Thứ nhất, chỉ gọi hook ở top level của component, tức là không gọi trong if, trong vòng lặp, trong hàm lồng hay sau một câu return sớm. Thứ hai, chỉ gọi hook từ React function component hoặc từ custom hook, chứ không gọi từ một hàm JavaScript thường. Lý do bản chất rất quan trọng: React không nhận diện hook theo tên mà theo THỨ TỰ gọi, nó dựa vào index của lần gọi để biết state nào ứng với hook nào. Nếu mình gọi hook có điều kiện thì giữa các lần render thứ tự sẽ lệch, ví dụ render này có gọi useState còn render kia thì không, dẫn tới React gán nhầm state của hook này sang hook khác và app chạy sai một cách rất khó debug. Cho nên thay vì đặt hook trong điều kiện, mình luôn để hook ở top level rồi đặt điều kiện bên trong thân nó. Mình cũng bật eslint-plugin-react-hooks để tự động bắt các vi phạm này."

---

- Chỉ gọi hook ở **top level** — không trong `if`, vòng lặp, hàm lồng, hay sau `return` sớm.
- Chỉ gọi hook từ **React function component hoặc custom hook**, không từ hàm JS thường.
- Tại sao: React nhận diện hook theo THỨ TỰ gọi (theo index), không theo tên. Gọi có điều kiện làm lệch thứ tự giữa các render -> gán nhầm state.
- Dùng `eslint-plugin-react-hooks` để bắt vi phạm.

```tsx
// Đúng: hook ở top level, điều kiện đặt bên trong
useEffect(() => { if (!show) return; /* ... */ }, [show]);
```

</details>

---

### 3.11. `useLayoutEffect` khác `useEffect` ở điểm nào? Khi nào bắt buộc dùng `useLayoutEffect`?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Cả hai đều chạy effect sau khi DOM được cập nhật, khác nhau ở thời điểm so với lúc trình duyệt vẽ ra màn hình. useEffect chạy bất đồng bộ, SAU khi trình duyệt đã paint, nên không chặn việc hiển thị; còn useLayoutEffect chạy đồng bộ TRƯỚC khi paint, tức React sẽ giữ lại màn hình cho đến khi effect chạy xong. Vì vậy mặc định mình luôn dùng useEffect, chỉ chuyển sang useLayoutEffect khi cần ĐỌC kích thước hoặc vị trí DOM rồi điều chỉnh layout ngay, ví dụ tính toán vị trí tooltip hay đo chiều cao một phần tử để căn chỉnh — nếu dùng useEffect ở đây người dùng sẽ thấy phần tử nhảy/nhấp nháy một nhịp vì nó đã paint rồi mới sửa. Đánh đổi là useLayoutEffect chặn paint nên lạm dụng sẽ làm chậm hiển thị, và trong Next.js SSR nó không chạy trên server, React còn cảnh báo, nên với code dùng chung mình thường tránh hoặc dùng useEffect cho an toàn."

---

- Khác ở thời điểm chạy so với paint của trình duyệt.
- Dùng `useLayoutEffect` khi cần ĐỌC kích thước/vị trí DOM rồi điều chỉnh ngay, tránh nhấp nháy (ví dụ tính vị trí tooltip).
- Ưu tiên `useEffect` (không chặn paint); chỉ chuyển khi thấy flicker. Trong Next.js SSR, `useLayoutEffect` không chạy trên server và sẽ cảnh báo.

| | `useEffect` | `useLayoutEffect` |
| --- | --- | --- |
| Thời điểm | Bất đồng bộ, SAU paint | Đồng bộ, TRƯỚC paint |
| Chặn paint | Không | Có |

</details>

---

### 3.12. `useId` giải quyết vấn đề gì, đặc biệt trong ngữ cảnh SSR của Next.js?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"useId sinh ra một id duy nhất và ổn định, quan trọng nhất là nó KHỚP nhau giữa phía server và phía client. Vấn đề nó giải quyết là khi mình cần id để liên kết label với input (htmlFor) hoặc cho các thuộc tính aria — nếu mình tự sinh id ngẫu nhiên kiểu Math.random() thì server render ra một chuỗi, client hydrate lại sinh ra chuỗi khác, dẫn tới hydration mismatch và React phàn nàn. useId đảm bảo cả hai phía cho ra cùng một chuỗi nên không còn lệch. Trong Next.js, đặc biệt App Router vốn SSR mặc định, hoặc khi mình viết component dùng chung trong design system, đây là cách chuẩn để tạo id cho accessibility. Lưu ý là nó chỉ dùng cho việc liên kết label–input chứ KHÔNG dùng làm key cho list, và nếu cần nhiều id liên quan thì nên nối hậu tố như `${id}-hint` thay vì gọi useId nhiều lần."

---

- Sinh id duy nhất, ổn định, KHỚP giữa server và client.
- Giải quyết: id ngẫu nhiên (`Math.random()`) cho `htmlFor`/`aria-*` sẽ khác nhau giữa SSR và client -> **hydration mismatch**. `useId` đảm bảo cùng chuỗi hai phía.
- Lưu ý: dùng cho accessibility/liên kết label–input, KHÔNG dùng làm `key` trong list; nối hậu tố (`${id}-hint`) thay vì gọi nhiều lần; hữu ích trong Next.js App Router và component thư viện.

```tsx
const id = useId();
<label htmlFor={id}>Mật khẩu</label>
<input id={id} aria-describedby={`${id}-hint`} />
```

</details>

---

### 3.13. `useImperativeHandle` kết hợp `forwardRef` để làm gì? Cho ví dụ.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Mặc định ref chỉ gắn được vào DOM element, không truyền thẳng vào function component được; forwardRef chính là cách để một function component nhận ref từ cha (ở React 19 thì ref đã có thể truyền như một prop bình thường nên không cần forwardRef nữa). Còn useImperativeHandle cho phép con TÙY BIẾN cái object mà ref trỏ tới — thay vì lộ nguyên DOM node ra ngoài, con chỉ expose đúng những hành động mệnh lệnh cần thiết như focus(), scrollToTop() hay play()/pause(). Mình dùng cặp này khi thực sự cần điều khiển kiểu imperative từ cha, ví dụ cha muốn focus vào một input tùy biến hoặc reset/scroll một component con. Đánh đổi là cách làm này đi ngược tinh thần declarative của React, nên mình coi nó là giải pháp cuối: nếu việc đó giải quyết được bằng props hoặc state thì luôn ưu tiên props/state, chỉ rơi xuống imperative handle khi không còn cách nào tự nhiên hơn."

---

- `forwardRef` (React <= 18; React 19 truyền `ref` như prop) cho function component nhận `ref` từ cha.
- `useImperativeHandle` cho con TÙY BIẾN object mà ref trỏ tới — lộ API imperative (`focus()`, `scrollToTop()`) thay vì toàn bộ DOM node.
- Dùng khi cần điều khiển mệnh lệnh (focus, reset form, play/pause video).
- Bẫy: đi ngược tinh thần declarative, chỉ dùng khi thực sự cần; nếu giải quyết được bằng prop/state thì ưu tiên.

```tsx
const FancyInput = forwardRef<InputHandle, Props>((props, ref) => {
  const inputRef = useRef<HTMLInputElement>(null);
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current?.focus(),
  }), []); // chỉ lộ focus, không lộ toàn bộ DOM
  return <input ref={inputRef} />;
});
```

</details>

## 4. Tối ưu hiệu năng React

### 4.1. Những nguyên nhân phổ biến gây re-render thừa trong React là gì?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Một component sẽ re-render khi state của nó đổi, props đổi, khi component CHA re-render (vì mặc định React render lại cả cây con), hoặc khi Context mà nó subscribe thay đổi. Re-render thừa hay phổ biến nhất đến từ vài nguyên nhân: truyền object, array hay hàm inline xuống con — mỗi lần render là một reference mới nên React.memo vô hiệu; Context có value là object mới mỗi render khiến toàn bộ consumer render theo; và đặt state quá cao trong cây làm một thay đổi nhỏ kéo theo cả nhánh lớn render lại. Điều quan trọng mình luôn nhấn là bản thân re-render khá rẻ — nó chỉ thật sự gây giật khi cây con lớn hoặc node nặng. Nên trước khi tối ưu mình luôn ĐO bằng React Profiler để biết chỗ nào thực sự là vấn đề, chứ không rải memo khắp nơi cho yên tâm."

---

- Component re-render khi: state đổi, props đổi, **cha re-render** (mặc định render cả cây con), hoặc Context nó subscribe đổi.
- **Truyền object/array/hàm inline**: mỗi render tạo reference mới, phá `React.memo`.
- **Context `value` là object mới mỗi render**: mọi consumer re-render.
- **State đặt quá cao**: một thay đổi nhỏ render lại nhánh lớn.
- Lưu ý: re-render bản thân nó rẻ; chỉ chậm khi cây con lớn/node nặng. **Đo bằng Profiler trước khi tối ưu.**

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  // style và onClick là object/hàm MỚI mỗi render -> Child re-render thừa
  return (
    <>
      <button onClick={() => setCount((c) => c + 1)}>{count}</button>
      <Child style={{ margin: 8 }} onClick={() => console.log('hi')} />
    </>
  );
}
```

</details>

---

### 4.2. `React.memo` hoạt động thế nào, khi nào nó thực sự hiệu quả và khi nào vô dụng (thậm chí có hại)?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"React.memo là một HOC bọc quanh component, nó so sánh NÔNG props giữa lần render cũ và mới, nếu props không đổi thì bỏ qua việc render lại. Điểm cần hiểu rõ là nó chỉ chặn re-render gây ra BỞI cha render lại, chứ không liên quan gì tới việc state nội bộ hay Context mà component đó tự subscribe. Nó thực sự hiệu quả khi component bị render thường xuyên với cùng props, đủ nặng để đáng tối ưu, và props phải ổn định reference — tức là primitive hoặc đã được giữ ổn định bằng useMemo/useCallback. Ngược lại nó vô dụng, thậm chí có hại, khi props gần như đổi mỗi render do truyền inline (so sánh luôn fail nhưng vẫn tốn chi phí so sánh), khi component quá nhẹ, hoặc khi children là JSX inline vì đó là object mới mỗi lần. Nên mình chỉ dùng React.memo có chủ đích, đi kèm với việc ổn định props, chứ không bọc bừa."

---

- HOC bọc component, bỏ qua re-render nếu props không đổi theo **so sánh nông**. Chỉ chặn re-render do *cha re-render*, không liên quan state/Context nội bộ.
- **Hiệu quả khi:** render thường xuyên với *cùng props*, component đủ nặng, props ổn định reference (primitive hoặc đã `useMemo`/`useCallback`).
- **Vô dụng/có hại khi:** props đổi gần như mỗi render (inline) -> so sánh luôn fail; component quá nhẹ; `children` là JSX inline (object mới mỗi render).

```jsx
const Row = React.memo(({ item, onSelect }) =>
  <li onClick={() => onSelect(item.id)}>{item.name}</li>);

// HỮU ÍCH: onSelect ổn định nhờ useCallback
const onSelect = useCallback((id) => console.log(id), []);
// VÔ DỤNG: <Row onSelect={(id) => ...} /> -> reference mới mỗi render
```

</details>

---

### 4.3. Khi nào nên dùng `useMemo` và `useCallback`? Phân biệt hai hook này và những hiểu lầm thường gặp.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Hai hook này bản chất giống nhau, chỉ khác cái được ghi nhớ: useMemo ghi nhớ GIÁ TRỊ trả về của một phép tính, còn useCallback ghi nhớ chính HÀM — thực ra useCallback(fn, deps) chỉ là viết tắt của useMemo(() => fn, deps). Mình dùng useMemo khi có phép tính tốn kém như sort hay filter một tập lớn, để khỏi tính lại mỗi render. Còn useCallback hoặc useMemo dùng để giữ ổn định reference khi truyền xuống một component đã được React.memo, hoặc khi giá trị đó là dependency của một hook khác như useEffect — để tránh effect chạy lại liên tục. Hiểu lầm thường gặp là memo hóa mọi thứ cho chắc: bản thân memo cũng có chi phí lưu và so sánh, deps sai còn gây stale closure, và useCallback hoàn toàn vô dụng nếu component nhận nó chưa được bọc React.memo. Mình cũng hay nhắc thêm là React Compiler từ React 19 sẽ tự lo phần lớn việc này, nên xu hướng là bớt memo thủ công đi."

---

- `useMemo(fn, deps)`: ghi nhớ **giá trị trả về**; `useCallback(fn, deps)`: ghi nhớ **chính hàm** (= `useMemo(() => fn, deps)`).
- **Nên dùng khi:** (1) phép tính tốn kém (sort/filter tập lớn) — `useMemo`; (2) giữ ổn định reference khi truyền xuống **component đã memo** hoặc làm **dependency** hook khác — `useCallback`/`useMemo`.
- **Bẫy:** memo hóa mọi thứ "cho chắc" (bản thân memo có chi phí); sai `deps` -> stale closure; `useCallback` vô dụng nếu component nhận chưa bọc `React.memo`; React Compiler (React 19) tự lo phần lớn việc này.

```jsx
const filtered = useMemo(() => rows.filter(r => r.name.includes(query)), [rows, query]);
const handleClick = useCallback((id) => console.log(id), []);
```

</details>

---

### 4.4. Bạn cần render một danh sách 10.000+ dòng và app bị giật. Bạn xử lý thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Vấn đề ở đây là render hàng chục nghìn DOM node cùng lúc làm nghẽn layout, paint và ngốn bộ nhớ, dù người dùng chỉ nhìn thấy chừng chục dòng trong viewport. Giải pháp chính mình dùng là virtualization, hay còn gọi windowing: chỉ render những item đang nằm trong vùng nhìn thấy cộng thêm một ít buffer overscan, còn lại chỉ là một khoảng trống có chiều cao tính sẵn để thanh cuộn vẫn đúng. Mình thường dùng @tanstack/react-virtual vì nó headless và hỗ trợ cả kích thước động lẫn grid, ngoài ra còn react-window cho trường hợp đơn giản. Bổ trợ thêm thì mình kết hợp phân trang hoặc infinite scroll với useInfiniteQuery để không kéo hết dữ liệu một lần, memo từng dòng và tránh truyền style/handler inline. Bẫy hay gặp là quên đặt chiều cao cố định cho container cuộn nên virtualizer tính sai, và dùng index làm key khi danh sách có thể sắp xếp lại — phải dùng id ổn định."

---

- Vấn đề: hàng chục nghìn DOM node làm nghẽn layout/paint/bộ nhớ.
- Giải pháp chính: **virtualization (windowing)** — chỉ render item trong viewport (cộng buffer overscan), phần còn lại là khoảng trống có chiều cao tính sẵn.
- Thư viện: `react-window`, `react-virtualized`, `@tanstack/react-virtual` (headless, hỗ trợ kích thước động + grid).
- Bổ sung: kết hợp phân trang/infinite scroll (`useInfiniteQuery`); memo từng dòng; tránh inline style/handler.
- Bẫy: quên đặt chiều cao container cuộn; dùng index làm `key` khi list sắp xếp lại được.

```tsx
const rowVirtualizer = useVirtualizer({
  count: items.length,
  getScrollElement: () => parentRef.current,
  estimateSize: () => 40,
  overscan: 8,
});
// render getVirtualItems(), mỗi item position:absolute + transform translateY(v.start)
```

</details>

---

### 4.5. Code splitting trong React/Next.js: giải thích `React.lazy`, `Suspense`, và sự khác biệt với `next/dynamic`.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Code splitting là chia bundle JavaScript thành nhiều chunk nhỏ và chỉ tải khi cần, để giảm lượng JS phải tải ban đầu, nhờ đó cải thiện TTI và LCP. React.lazy nhận một hàm import động, tách phần đó thành chunk riêng và phải bọc trong Suspense với một fallback — Suspense sẽ 'treo' cây con và hiển thị fallback trong lúc chunk đang tải. next/dynamic của Next.js về cơ bản là wrapper bọc sẵn lazy và Suspense, nhưng tiện hơn ở chỗ cho thêm tùy chọn ssr: false để component chỉ chạy phía client, và prop loading để khai báo fallback. Một điểm đặc thù là trong App Router, Server Components vốn không gửi JS xuống client nên next/dynamic chủ yếu dùng cho Client Components. Chiến lược của mình là split theo route và theo những component nặng/ít dùng như modal, chart hay editor; tránh split quá nhỏ gây nhiều request lẻ, và đặt Suspense ở đúng tầm để không làm cả màn hình nhấp nháy."

---

- **Code splitting:** chia bundle thành chunk tải theo nhu cầu, giảm JS ban đầu (cải thiện TTI/LCP).
- `React.lazy(() => import(...))`: tách chunk riêng, phải bọc trong `<Suspense fallback>`. `Suspense` "treo" cây con khi chunk/dữ liệu đang tải.
- `next/dynamic`: bọc sẵn lazy + Suspense, thêm tùy chọn `ssr: false` và `loading`; trong App Router, Server Components vốn không gửi JS nên `next/dynamic` chủ yếu cho Client Components.
- Chiến lược: split theo **route** và theo **component nặng/ít dùng** (modal, chart, editor). Bẫy: split quá nhỏ -> nhiều request lẻ; đặt `Suspense` sai vị trí làm cả màn hình nhấp nháy.

```tsx
const Chart = lazy(() => import('./Chart'));
const Map = dynamic(() => import('./Map'), { ssr: false, loading: () => <p>...</p> });
```

</details>

---

### 4.6. Tại sao `key` ổn định lại quan trọng cho hiệu năng và tính đúng đắn? Tại sao không nên dùng index làm key?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Trong quá trình reconciliation, React dùng key để khớp phần tử cũ và mới trong một danh sách, từ đó quyết định node nào được giữ lại, node nào di chuyển, thêm hay xóa. Một key ổn định và duy nhất giúp React tái sử dụng đúng DOM node và instance thay vì hủy đi tạo lại — vừa nhanh hơn vừa GIỮ ĐÚNG state nội bộ của từng item, ví dụ giá trị đang gõ trong input của một dòng. Vấn đề khi dùng index làm key là index gắn với VỊ TRÍ chứ không gắn với DỮ LIỆU, nên khi mình chèn, xóa hoặc sắp xếp lại danh sách, các item bị 'lệch' key, dẫn tới state và input nhảy sai chỗ, kèm theo render thừa. Với danh sách hoàn toàn tĩnh thì index tạm chấp nhận được, nhưng cứ có thay đổi thứ tự hay item có state riêng là phải dùng id ổn định từ dữ liệu. Và tuyệt đối tránh Math.random() làm key vì nó tạo key mới mỗi render khiến cả danh sách remount."

---

- React dùng `key` trong **reconciliation** để khớp phần tử cũ-mới, quyết định node nào giữ/di chuyển/thêm/xóa.
- Key ổn định, duy nhất giúp tái sử dụng DOM/instance thay vì hủy-tạo lại — nhanh hơn và **giữ đúng state nội bộ**.
- **Index làm key** gây lỗi khi đổi thứ tự/chèn/xóa giữa danh sách: index gắn với *vị trí* không gắn *dữ liệu* -> state/input nhảy sai chỗ.
- Bẫy: đừng dùng `Math.random()` làm key -> remount toàn bộ mỗi render.

```jsx
{todos.map((t, i) => <TodoItem key={i} todo={t} />)}   // SAI
{todos.map((t) => <TodoItem key={t.id} todo={t} />)}   // ĐÚNG
```

| Tình huống | Index | ID ổn định |
|---|---|---|
| Danh sách tĩnh | Chấp nhận được | Tốt |
| Có chèn/xóa/sắp xếp | Lỗi state + render thừa | Đúng |
| Item có state nội bộ | Sai lệch | Đúng |

</details>

---

### 4.7. "Colocate state" (đặt state gần nơi dùng) và "lift state up" (nâng state lên) ảnh hưởng tới hiệu năng ra sao? Khi nào chọn cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Phạm vi re-render phụ thuộc vào NƠI state sống: khi state đổi thì component sở hữu nó cùng toàn bộ cây con sẽ render lại. Colocate là đặt state gần đúng nơi dùng, nhờ vậy chỉ một nhánh nhỏ render lại — rất hợp với state cục bộ như nội dung một ô input hay trạng thái mở/đóng dropdown. Lift state up là nâng state lên cha chung, cần khi nhiều component anh em phải chia sẻ cùng một state; nhưng nếu nâng quá cao thì mỗi thay đổi nhỏ lại kéo cả một nhánh lớn render theo, gây chậm. Một kỹ thuật bổ trợ mình hay dùng là 'lift content, not state': truyền phần JSX nặng xuống qua children, vì children là prop được tạo ở cha cao hơn nên không bị render lại khi state ở cha trung gian đổi. Nguyên tắc của mình là mặc định cứ colocate cho gọn, chỉ nâng state lên khi thực sự cần chia sẻ, để giữ phạm vi re-render càng hẹp càng tốt."

---

- Khi state đổi, component sở hữu nó + cả cây con re-render -> phạm vi re-render phụ thuộc *nơi state sống*.
- **Colocate** (đặt gần nơi dùng): chỉ một nhánh nhỏ re-render. Tốt cho state cục bộ (nội dung input, mở/đóng dropdown).
- **Lift up**: cần khi nhiều anh em chia sẻ state; nhưng nâng quá cao -> mỗi thay đổi nhỏ render lại nhánh lớn.
- Kỹ thuật bổ trợ: **"lift content, not state"** — truyền JSX nặng qua `children` để không bị render lại khi state cha đổi.
- Quy tắc: bắt đầu colocate, chỉ nâng khi thực sự cần chia sẻ.

```jsx
// CẢI THIỆN: colocate q vào SearchSection -> chỉ nó render khi gõ
function App() { return <><SearchSection /><HeavyTree /></>; }
function SearchSection() { const [q, setQ] = useState(''); return <SearchBox value={q} onChange={setQ} />; }
```

</details>

---

### 4.8. Context API gây re-render hàng loạt thế nào, và làm sao tối ưu nó?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Bản chất là Context không có cơ chế chọn lọc theo trường: mọi component gọi useContext sẽ re-render bất cứ khi nào cái value của Provider đổi reference, chứ không phải khi phần dữ liệu nó thực sự dùng thay đổi. Hai cái bẫy mình hay gặp là tạo object value mới ngay trong render khiến tất cả consumer render lại, và gộp quá nhiều thứ vào một MegaContext nên chỉ một field đổi cũng kéo cả app render. Cách mình tối ưu là memo hóa value bằng useMemo, tách Context ra theo mức độ thay đổi hoặc tách riêng value và dispatch (vì hàm dispatch thường ổn định, không cần re-render những ai chỉ cần nó). Mình cũng tách Provider thành component riêng nhận children để phần con không bị render lại theo. Còn nếu state đổi liên tục và phức tạp thì mình không cố ép Context nữa mà dùng Zustand hoặc Redux Toolkit vì chúng có selector, đăng ký chọn lọc đúng phần state cần. Tóm lại Context tốt cho dữ liệu ít đổi như theme, user, locale, còn state nóng thì nên dùng thư viện có selector."

---

- Mọi `useContext` re-render khi **`value` đổi reference** — không có so sánh chọn lọc theo trường.
- Hai lỗi phổ biến: (1) `value` là object mới mỗi render -> *tất cả* consumer render; (2) gộp nhiều thứ vào một Context -> một trường đổi kéo theo mọi consumer.
- **Tối ưu:** memo hóa `value` bằng `useMemo`; **tách Context** (state hay đổi/ít đổi, hoặc tách *value* và *dispatcher*); tách provider thành component riêng truyền `children`; state phức tạp dùng Zustand/Redux Toolkit/Jotai có **selector**.
- Bẫy: "MegaContext" cho toàn app là nguyên nhân số 1 gây re-render lan rộng.

```jsx
const value = useMemo(() => ({ user, setUser }), [user]);
<AuthContext.Provider value={value}>{children}</AuthContext.Provider>
```

</details>

---

### 4.9. Bạn dùng React DevTools Profiler thế nào để tìm và sửa vấn đề hiệu năng?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"React DevTools Profiler là công cụ để mình đo thật chứ không đoán: nó ghi lại từng commit, cho biết component nào render, render mất bao lâu và quan trọng nhất là vì sao nó render. Quy trình của mình là bật tùy chọn 'record why each component rendered', record đúng thao tác đang bị giật rồi dừng lại, sau đó nhìn flame chart để thấy cây render và ranked chart để biết component nào tốn thời gian nhất. Khi click vào một component, Profiler nói rõ lý do là props đổi, hooks đổi, parent render hay context đổi, từ đó mình biết sửa đúng chỗ chứ không rải tối ưu lung tung. Tùy nguyên nhân mà mình áp giải pháp tương ứng: ổn định props rồi bọc React.memo, colocate hoặc tách context, hay virtualization cho list dài. Nguyên tắc mình luôn giữ là đo trước rồi mới tối ưu, sửa xong đo lại để xác nhận, và con số cuối cùng phải đánh giá trên production build vì dev build chậm hơn nhiều và có thêm chi phí của chính DevTools."

---

- Profiler ghi lại các commit: **component nào render**, **bao lâu**, **vì sao**.
- Quy trình: (1) bật "Record why each component rendered"; (2) record thao tác bị giật rồi dừng; (3) xem **flame chart** + **ranked chart**; (4) chọn component xem lý do (*props/hooks changed, parent rendered, context changed*); (5) sửa: `React.memo`, ổn định props, colocate/tách context, virtualization.
- Nguyên tắc: **đo trước, tối ưu sau**; đo lại để xác nhận. Đánh giá con số cuối ở **production build**.

```jsx
<Profiler id="List" onRender={(id, phase, actualDuration) => console.log(id, phase, actualDuration)}>
  <List />
</Profiler>
```

</details>

---

### 4.10. Concurrent rendering trong React 18 là gì? `useTransition` và `useDeferredValue` giúp gì cho hiệu năng cảm nhận được?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Concurrent rendering ở React 18 nghĩa là việc render không còn là một khối chạy liền một mạch nữa, mà có thể bị gián đoạn, tạm dừng rồi nối lại; nhờ vậy React ưu tiên những cập nhật khẩn như gõ phím hay click trước, và hoãn những cập nhật nặng lại để không làm nghẽn tương tác. Nó được bật khi mình dùng createRoot. useTransition cho phép mình đánh dấu một cập nhật là không khẩn cấp, ví dụ lọc một danh sách lớn, và nó trả về cờ isPending để hiển thị trạng thái đang xử lý; còn useDeferredValue thì tạo một phiên bản trễ của giá trị, để phần UI nặng dùng giá trị trễ trong khi ô input vẫn cập nhật tức thì theo giá trị mới. Điểm cốt lõi mình hay nhấn là cả hai chỉ cải thiện hiệu năng cảm nhận, tức là làm app phản hồi mượt hơn về cảm giác chứ không làm phép tính chạy nhanh hơn. Nên mình chỉ dùng khi cập nhật thực sự nặng, và vẫn kết hợp với memo hay virtualization để giảm khối lượng tính toán thật sự."

---

- **Concurrent rendering:** React 18 cho phép render bị **gián đoạn, tạm dừng, nối lại**; ưu tiên cập nhật khẩn (gõ, click) trước, hoãn cập nhật nặng. Kích hoạt qua `createRoot`.
- `useTransition`: đánh dấu cập nhật **không khẩn cấp**, xử lý ưu tiên thấp; trả về cờ `isPending`.
- `useDeferredValue`: tạo phiên bản **trễ** của giá trị; UI nặng dùng giá trị trễ, input dùng giá trị tức thì.
- Lưu ý: đây là tối ưu **hiệu năng cảm nhận** — không làm phép tính nhanh hơn. Bẫy: dùng cho cập nhật vốn rẻ là thừa; vẫn nên kết hợp memo/virtualization.

```tsx
const [isPending, startTransition] = useTransition();
setQuery(e.target.value);                         // khẩn: input mượt
startTransition(() => setResults(filterHuge(e.target.value))); // ưu tiên thấp
```

</details>

---

### 4.11. Automatic batching ở React 18 là gì? Khác gì so với React 17?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Batching là việc React gom nhiều lần gọi setState lại thành một lần render duy nhất, thay vì render lại sau mỗi setState, để giảm số lần re-render. Ở React 17 thì batching chỉ xảy ra bên trong event handler của React thôi; còn nếu mình gọi setState trong setTimeout, trong Promise hay trong native event thì mỗi setState lại gây một lần render riêng. React 18 với createRoot mang lại automatic batching ở mọi nơi, kể cả async/await, setTimeout hay promise, nên ví dụ sau một lần await fetch mà gọi hai setState thì giờ chỉ render một lần thay vì hai. Khi nào cần thoát batching để DOM cập nhật ngay lập tức thì có flushSync từ react-dom, nhưng cái này hiếm dùng vì nó ép render và commit ngay nên mất lợi ích gom. Một lưu ý khi nâng cấp là code cũ nào dựa vào việc render ngay sau mỗi setState trong async có thể bị đổi hành vi, nên cần để ý."

---

- **Batching:** gom nhiều `setState` vào **một lần render**, giảm số re-render.
- **React 17:** chỉ batch trong **event handler của React**; trong `setTimeout`/Promise/native event thì mỗi setState một render.
- **React 18 (`createRoot`):** **automatic batching** ở *mọi nơi* (promise, `setTimeout`, async/await, native handler).
- Thoát batching: `flushSync` từ `react-dom` (render & commit ngay, dùng hiếm).
- Bẫy nâng cấp: code cũ dựa vào render ngay sau mỗi setState trong async có thể đổi hành vi.

```jsx
async function handleAsync() {
  await fetchData();
  setCount(c => c + 1); setFlag(f => !f); // R17: 2 render, R18: 1 render
}
```

</details>

---

### 4.12. Kể về một lần bạn chẩn đoán và khắc phục một vấn đề hiệu năng React trong dự án thực tế.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

**📌 Câu trả lời mẫu:**

"Ở dự án Jira Clone, board kéo-thả bắt đầu giật rõ khi một sprint có vài trăm issue, và gõ vào ô tìm kiếm thì trễ thấy được. Việc đầu tiên mình làm là đo chứ không đoán: mở React Profiler, bật 'record why each component rendered', và phát hiện mỗi lần gõ một ký tự là toàn bộ các cột và card đều re-render. Truy ra gốc, hóa ra state tìm kiếm đặt ở component cha cao nhất, lại truyền qua Context với một object value mới mỗi render, cộng với Card nhận handler inline nên React.memo cũng vô dụng. Mình xử lý đúng gốc: colocate state tìm kiếm xuống gần nơi dùng, memo hóa value và tách Context dispatch riêng, bọc Card bằng React.memo với onClick qua useCallback; với danh sách dài thì thêm virtualization bằng @tanstack/react-virtual và bọc phần lọc trong useTransition để giữ ô input mượt. Kết quả là số component render mỗi lần gõ giảm từ hàng trăm xuống còn vài chục, thời gian commit giảm hẳn và board hết giật. Bài học mình luôn nhấn là đo trước khi sửa, sửa đúng nguyên nhân rồi đo lại để xác nhận, chứ không rải useMemo khắp nơi cho yên tâm."

---

Câu hỏi hành vi — kể câu chuyện cụ thể, có số liệu, theo STAR:

- **Situation:** "Dự án Jira Clone, board kéo-thả bị giật khi sprint có vài trăm issue; gõ vào ô tìm kiếm trễ rõ rệt."
- **Task:** "Giảm độ trễ tương tác xuống mượt (< ~50ms/lần gõ) mà không viết lại toàn màn hình."
- **Action:**
  - "Mở Profiler, bật 'record why each component rendered' -> mỗi lần gõ làm *toàn bộ* cột và card re-render."
  - "Nguyên nhân: state tìm kiếm/filter ở cha cao nhất + Context truyền `value` mới mỗi render + Card nhận handler inline."
  - "Colocate state tìm kiếm, memo hóa `value` và tách dispatch ra context riêng, bọc `Card` bằng `React.memo` với `onClick` qua `useCallback`."
  - "Danh sách dài: virtualization bằng `@tanstack/react-virtual` + bọc lọc trong `useTransition`."
- **Result:** "Số component render mỗi lần gõ giảm từ hàng trăm xuống vài chục; thời gian commit giảm đáng kể, hết giật. Ghi lại checklist cho team."

Lời khuyên: nhấn mạnh **đo trước khi sửa**, sửa đúng gốc, **đo lại xác nhận**. Tránh kể "tôi bọc useMemo khắp nơi".

</details>

## 5. Quản lý State và Data Fetching

### 5.1. Context API thực sự dùng để làm gì, và đâu là những giới hạn khiến nó không thể thay thế một thư viện state management?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Mình xem Context API về bản chất là một công cụ dependency injection: nó giúp truyền dữ liệu xuyên qua nhiều tầng component xuống nơi cần mà không phải khoan props qua từng cấp. Điều quan trọng cần nói rõ là nó không phải công cụ tối ưu hiệu năng và không thay thế được một thư viện state management. Giới hạn lớn nhất là Context không có selector: mọi consumer sẽ re-render khi value đổi reference, kể cả khi nó chỉ dùng một phần nhỏ trong đó không liên quan tới thay đổi đó. Một cái bẫy rất phổ biến là tạo object value mới ngay trong render khiến reference luôn đổi, mình khắc phục bằng useMemo. Vì những lý do đó, Context hợp với dữ liệu ít đổi và đọc nhiều như theme, locale hay thông tin user; còn state đổi thường xuyên hoặc logic phức tạp thì mình chọn Zustand, Redux Toolkit hay Jotai vì chúng cho đăng ký chọn lọc bằng selector. Một mẹo nhỏ để giảm re-render là tách Context giá trị và Context dispatch ra riêng."

---

- Context API là công cụ **dependency injection**: truyền dữ liệu xuống cây component, tránh prop drilling. **Không** phải công cụ tối ưu hiệu năng.
- Giới hạn lớn nhất: mọi consumer (`useContext`) **re-render khi `value` đổi reference**, dù không dùng phần dữ liệu đã đổi. Context **không có selector**.
- Bẫy: tạo object `value` mới ngay trong render -> reference luôn đổi. Khắc phục bằng `useMemo`.
- Hợp với dữ liệu **ít đổi, đọc nhiều** (theme, locale, user). State đổi thường xuyên/logic phức tạp -> Zustand/Redux/Jotai (có selector).
- Mẹo giảm re-render: **tách Context** thành Context giá trị và Context dispatch.

```tsx
// ❌ value là object mới mỗi render -> mọi consumer re-render
<ThemeContext.Provider value={{ theme, setTheme }}>
// ✅ memo hóa
const value = useMemo(() => ({ theme, setTheme }), [theme]);
<ThemeContext.Provider value={value}>
```

</details>

---

### 5.2. Phân biệt "server state" và "client state". Vì sao việc tách bạch hai loại này lại quan trọng khi chọn công cụ?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Mình phân biệt hai loại này theo ai sở hữu nguồn sự thật của dữ liệu. Client state là do client sở hữu, luôn đồng bộ và luôn đúng, ví dụ modal đang mở hay đóng, tab đang chọn, nội dung form chưa submit. Còn server state thực ra chỉ là bản sao mình mượn từ server, nó bất đồng bộ, có thể bị cũ đi vì server đã đổi mà client chưa biết, và luôn kèm theo trạng thái loading, error, ví dụ danh sách bài viết hay hồ sơ user. Việc tách bạch quan trọng vì hai loại này cần công cụ khác nhau: client state thì useState hay Zustand là đủ, còn server state cần caching, dedupe request, refetch nền, retry và biết khi nào dữ liệu stale, nên mình dùng TanStack Query, SWR hoặc RTK Query. Sai lầm kinh điển mình hay thấy là nhét data API vào Redux rồi tự viết fetchStart, fetchSuccess, fetchFailure, vừa nhiều boilerplate mà vẫn thiếu những thứ như dedupe hay stale-while-revalidate mà các thư viện server state làm sẵn."

---

- **Client state**: do client sở hữu, đồng bộ, luôn đúng (modal, tab, form chưa submit).
- **Server state**: "mượn" từ server, bất đồng bộ, có thể **bị cũ (stale)**, có loading/error (danh sách bài viết, hồ sơ user).
- Tách bạch quan trọng vì cần **công cụ khác nhau**: client state hợp `useState`/Zustand; server state cần caching, dedupe, background refetch, retry, biết khi nào stale -> dùng **TanStack Query/SWR/RTK Query**.
- Sai lầm kinh điển: nhét data API vào Redux rồi tự viết `fetchStart/Success/Failure` -> nhiều boilerplate mà vẫn thiếu (không dedupe, không stale-while-revalidate).

```tsx
const { data: posts } = useQuery({ queryKey: ['posts'], queryFn: fetchPosts }); // server
const [isOpen, setOpen] = useState(false); // client
```

</details>

---

### 5.3. Redux Toolkit giải quyết những "đau đầu" gì của Redux truyền thống? Giải thích `createSlice` và immutable update bằng Immer.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Redux truyền thống có nhiều boilerplate: mình phải tự khai báo action type, viết action creator, viết reducer với switch case dài, rồi cấu hình store thủ công. Redux Toolkit là cách viết Redux được khuyến nghị chính thức để giải quyết đúng những đau đầu đó. createSlice cho mình khai báo tên slice và các reducer, rồi nó tự sinh ra action creators và action types tương ứng, nên mình không phải viết tay nữa. Điểm hay nhất là RTK dùng Immer bên dưới, nên trong reducer mình có thể viết như đang mutate trực tiếp, ví dụ state.items.push, nhưng thực chất Immer theo dõi thay đổi trên một draft và tạo ra state mới bất biến phía sau, vừa đỡ phải spread lồng nhau vừa giữ đúng tính immutable. Lưu ý là chỉ được mutate cái draft bên trong reducer của createSlice hay createReducer thôi, ra ngoài thì không. Ngoài ra configureStore tự bật Redux DevTools và các middleware mặc định như serializable check và mutation check ở môi trường dev, giúp bắt lỗi sớm."

---

- Redux gốc nhiều boilerplate: tự viết action type, action creator, reducer `switch`, cấu hình store. RTK là cách viết Redux **được khuyến nghị chính thức**.
- `createSlice` **tự sinh action creators + action types** từ tên reducer.
- RTK dùng **Immer** nên có thể "viết như mutate" trên draft, Immer tạo state mới bất biến phía sau. Lưu ý: chỉ mutate được draft **bên trong** reducer của createSlice/createReducer.
- `configureStore` tự bật DevTools và middleware mặc định (serializable check, mutate check) ở dev.

```ts
const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [] } as CartState,
  reducers: {
    addItem(state, action: PayloadAction<string>) {
      const f = state.items.find((i) => i.id === action.payload);
      if (f) f.qty += 1; else state.items.push({ id: action.payload, qty: 1 });
    },
  },
});
export const { addItem } = cartSlice.actions; // tự sinh
```

</details>

---

### 5.4. RTK Query là gì? Nó khác `createAsyncThunk` ở điểm nào, và khi nào nên chọn RTK Query thay vì TanStack Query?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"RTK Query là lớp data-fetching và caching được tích hợp sẵn trong Redux Toolkit. Mình chỉ cần định nghĩa các endpoint, nó tự sinh ra hooks như useGetPostsQuery, tự lo caching, dedupe, trạng thái loading và error, và quản lý làm mới dữ liệu qua hệ thống tag để invalidate; cache thì nằm trong chính Redux store. So với createAsyncThunk thì khác hẳn về mức độ: thunk chỉ là một async action, mình vẫn phải tự viết reducer để lưu data, tự quản loading và cache, trong khi RTK Query làm hết những việc đó. Về lựa chọn, mình dùng RTK Query khi dự án đã dùng Redux rồi và muốn giữ một hệ sinh thái thống nhất; còn nếu không dùng Redux thì mình chọn TanStack Query vì nó là thư viện server state độc lập, linh hoạt hơn với infinite query, optimistic update, devtools mạnh, và ngày nay thường là lựa chọn mặc định. Điểm cần nhớ là cả hai cùng giải quyết server state, nên không nên dùng song song cả hai cho cùng một mục đích."

---

- **RTK Query**: lớp data-fetching/caching tích hợp trong RTK. Định nghĩa endpoint -> tự sinh **hooks** (`useGetPostsQuery`...), tự quản caching/dedupe/loading/error, invalidation qua **tag**, cache lưu trong Redux store.
- Khác `createAsyncThunk`: thunk chỉ là async action, vẫn phải tự viết reducer lưu data, tự quản loading/cache. RTK Query làm hết.
- **Chọn RTK Query**: dự án **đã dùng Redux**, muốn hệ sinh thái thống nhất.
- **Chọn TanStack Query**: không dùng Redux, muốn thư viện server state độc lập, linh hoạt hơn (infinite query, optimistic, devtools mạnh) — thường là mặc định ngày nay.
- Lưu ý: cả hai cùng làm server state -> **đừng dùng cả hai** cho cùng mục đích.

```ts
addPost: builder.mutation<Post, NewPost>({
  query: (body) => ({ url: 'posts', method: 'POST', body }),
  invalidatesTags: ['Post'], // tự refetch getPosts
}),
```

</details>

---

### 5.5. Zustand hoạt động thế nào? Vì sao nó tránh được vấn đề re-render của Context, và `selector` quan trọng ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Zustand biến store thành một hook và không cần Provider bao quanh app. Component đăng ký vào store thông qua một selector, và nó chỉ re-render khi đúng phần state mà selector trả về thay đổi, so sánh theo Object.is. Đây chính là lý do nó tránh được vấn đề của Context: Context render lại mọi consumer khi value đổi, còn Zustand cho phép đăng ký chọn lọc nên chỉ ai dùng đúng phần đó mới render lại. Vì thế selector rất quan trọng, nó vừa là cách lấy dữ liệu vừa là cách giới hạn phạm vi re-render. Có một cái bẫy là nếu selector trả về một object mới, ví dụ gom nhiều field vào một object, thì reference luôn khác nhau nên component sẽ render lại mỗi lần state bất kỳ đổi; mình khắc phục bằng useShallow để so sánh nông. Zustand hợp với client state toàn cục cỡ vừa và nhỏ vì nhẹ và ít boilerplate, nhưng nó không phải công cụ cho server state, cái đó mình vẫn để TanStack Query lo."

---

- Store là một **hook**, **không cần Provider**. Component đăng ký qua **selector**, chỉ re-render khi phần state selector trả về đổi (so theo `Object.is`).
- Khác Context: Context re-render mọi consumer khi value đổi; Zustand cho đăng ký **chọn lọc**.
- Bẫy: selector trả về **object mới** -> luôn khác reference -> re-render mỗi lần. Khắc phục bằng `useShallow`.
- Hợp **client state toàn cục** vừa/nhỏ: nhẹ, ít boilerplate. **Không** phải công cụ server state.

```ts
const bears = useStore((s) => s.bears); // ✅ chỉ re-render khi bears đổi
// ❌ const { bears, fish } = useStore((s) => ({ bears: s.bears, fish: s.fish }));
const { bears, fish } = useStore(useShallow((s) => ({ bears: s.bears, fish: s.fish }))); // ✅
```

</details>

---

### 5.6. Trong TanStack Query, hãy giải thích `staleTime` và `gcTime` (trước đây là `cacheTime`). Chúng kiểm soát điều gì?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Hai cái này điều khiển hai chuyện khác nhau nên rất hay bị nhầm. staleTime là khoảng thời gian dữ liệu còn được coi là tươi; trong khoảng đó TanStack Query sẽ không tự refetch, kể cả khi component remount hay cửa sổ được focus lại, nó dùng luôn dữ liệu trong cache. Mặc định staleTime là 0, nghĩa là dữ liệu stale ngay lập tức nên hay thấy refetch nền liên tục. Còn gcTime, trước đây gọi là cacheTime, mặc định 5 phút, là khoảng thời gian giữ cache lại sau khi query không còn component nào dùng nữa; hết khoảng đó cache mới bị dọn. Nó quyết định khi mình quay lại màn hình cũ thì có sẵn dữ liệu cũ để hiển thị ngay hay phải fetch từ đầu. Cách mình dùng thực tế là dữ liệu ít đổi thì đặt staleTime cao để đỡ gọi lại, dữ liệu đổi nhanh thì staleTime thấp. Cái bẫy kinh điển là nhầm hai khái niệm rồi than 'đặt cacheTime mà sao vẫn refetch hoài', trong khi việc refetch nền là do staleTime quyết định chứ không phải gcTime."

---

- **`staleTime`**: thời gian dữ liệu còn **tươi (fresh)**; trong khoảng này **không tự refetch** (kể cả remount/focus). Mặc định `0` -> stale ngay, hay refetch nền.
- **`gcTime`** (mặc định 5 phút): khi query **không còn component dùng** (inactive), cache được giữ thêm `gcTime` rồi mới dọn. Quyết định có dữ liệu cũ để hiển thị ngay khi quay lại không.
- Thực dụng: dữ liệu ít đổi -> `staleTime` cao; dữ liệu đổi nhanh -> `staleTime` thấp.
- Bẫy: nhầm hai khái niệm rồi than "đặt cacheTime mà vẫn refetch hoài" — refetch nền do `staleTime`, không phải `gcTime`.

```tsx
useQuery({
  queryKey: ['posts'], queryFn: fetchPosts,
  staleTime: 60_000,    // 1 phút coi là tươi
  gcTime: 5 * 60_000,   // giữ cache 5 phút sau khi không dùng
});
```

</details>

---

### 5.7. `queryKey` đóng vai trò gì trong TanStack Query? Giải thích cơ chế invalidation và vì sao nó tốt hơn việc tự refetch thủ công.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Với TanStack Query thì queryKey vừa là danh tính của một query trong cache, vừa là dependency của nó. Hai query có cùng key thì dùng chung một ô cache, còn khi key đổi thì Query tự hiểu là dữ liệu khác và fetch lại, nên mình luôn nhét mọi tham số ảnh hưởng tới kết quả vào key, ví dụ ['posts', { page, status }]. Invalidation là sau khi mình mutate xong, mình đánh dấu những query liên quan là stale để chúng tự refetch, bằng invalidateQueries({ queryKey: ['posts'] }). Cái hay là nó khớp theo prefix, nên gọi ['posts'] là quét luôn mọi biến thể có phân trang hay filter bên trong, và Query Client biết query nào đang active để refetch ngay query đó. So với tự refetch thủ công, cách này tránh được việc mình phải nhớ gọi đúng hàm fetch ở đúng chỗ sau mỗi mutation, vốn rất dễ sót và khó bảo trì khi app lớn dần. Nói ngắn gọn thì mình chỉ cần khai báo 'dữ liệu này phụ thuộc cái gì' qua key, còn lại để thư viện lo việc đồng bộ."

---

- `queryKey` là **danh tính của query trong cache**, đồng thời là **dependency**: key đổi -> tự fetch lại. Là mảng, nên chứa mọi tham số ảnh hưởng kết quả.
- **Invalidation**: sau mutation, đánh dấu query liên quan là stale để chúng refetch (`invalidateQueries`).
- Tốt hơn refetch thủ công vì: hoạt động theo **prefix matching** (`['posts']` khớp cả `['posts', {page:1}]`...); Query Client biết query nào **active** để refetch ngay; tránh tự gọi lại đúng hàm fetch ở đúng chỗ (dễ sót, khó bảo trì).

```tsx
useQuery({ queryKey: ['posts', { page, status }], queryFn: () => fetchPosts({ page, status }) });
// sau mutation:
queryClient.invalidateQueries({ queryKey: ['posts'] }); // khớp mọi biến thể prefix
```

</details>

---

### 5.8. Optimistic update là gì? Hãy mô tả luồng đúng để làm optimistic update an toàn (kể cả khi lỗi) trong TanStack Query.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Optimistic update là mình cập nhật UI ngay theo kết quả mình kỳ vọng, trước cả khi server trả lời, để cảm giác phản hồi tức thì, kiểu bấm like là tim đỏ liền chứ không chờ mạng. Đổi lại là nếu request lỗi thì mình phải rollback về trạng thái cũ. Luồng an toàn trong TanStack Query mình làm qua ba callback của useMutation. Trong onMutate, việc đầu tiên là cancelQueries để chặn các refetch đang bay về ghi đè lên thay đổi lạc quan của mình, rồi mình chụp một snapshot dữ liệu hiện tại, dùng setQueryData để sửa cache ngay, và return cái snapshot đó qua context. Nếu lỗi thì onError lấy snapshot trong context để khôi phục lại. Cuối cùng onSettled luôn gọi invalidateQueries dù thành công hay thất bại, để đồng bộ lại với server cho chắc chắn đúng. Điểm dễ quên nhất là bước cancelQueries và việc lưu snapshot, thiếu hai cái đó là dễ dính bug nhấp nháy hoặc trạng thái sai khi có request chạy song song."

---

- **Optimistic update**: cập nhật UI **ngay** theo kết quả mong đợi trước khi server phản hồi (vd nút like); lỗi thì **rollback**.
- Luồng an toàn với `onMutate / onError / onSettled`:
  1. `onMutate`: **`cancelQueries`** (chặn refetch đang bay ghi đè), **lưu snapshot**, `setQueryData` cập nhật cache ngay, return `{ previous }` qua context.
  2. `onError`: rollback bằng snapshot trong context.
  3. `onSettled`: **`invalidateQueries`** để đồng bộ lại với server.

```tsx
useMutation({
  mutationFn: toggleLike,
  onMutate: async (id) => {
    await queryClient.cancelQueries({ queryKey: ['post', id] });
    const previous = queryClient.getQueryData(['post', id]);
    queryClient.setQueryData(['post', id], (o: Post) => ({ ...o, liked: !o.liked }));
    return { previous };
  },
  onError: (_e, id, ctx) => ctx?.previous && queryClient.setQueryData(['post', id], ctx.previous),
  onSettled: (_d, _e, id) => queryClient.invalidateQueries({ queryKey: ['post', id] }),
});
```

</details>

---

### 5.9. Infinite query khác gì với phân trang thông thường trong TanStack Query? Khi nào dùng `useInfiniteQuery`?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Khác biệt cốt lõi nằm ở cách dữ liệu được hiển thị. Phân trang thường thì mình bỏ page vào queryKey, mỗi trang là một query riêng và trang mới thay thế trang cũ trên màn hình, hợp với UI có nút Trang trước Trang sau. Còn useInfiniteQuery thì tích lũy các trang lại vào mảng pages, dữ liệu cộng dồn chứ không thay thế, nên hợp với kiểu Tải thêm hoặc cuộn vô hạn như feed mạng xã hội. Trái tim của nó là getNextPageParam, từ trang cuối mình tính ra param cho trang kế, có thể là cursor hoặc số trang, và trả undefined khi hết để hasNextPage thành false. Mình thường ghép với IntersectionObserver, khi item cuối vào viewport thì gọi fetchNextPage. Hai cái bẫy hay gặp là quên flatMap để gộp các trang lại khi render, và getNextPageParam viết sai khiến hasNextPage luôn true rồi tải vô tận."

---

- Phân trang thường (`useQuery` với key chứa `page`): **thay** trang này bằng trang khác — hợp UI "Trang trước/sau".
- `useInfiniteQuery`: **tích lũy nhiều trang vào `pages`** — hợp "Tải thêm"/cuộn vô hạn.
- Cốt lõi là `getNextPageParam`: từ trang cuối trả về param trang kế (cursor/số trang); trả `undefined` = hết.
- Dùng cho: feed mạng xã hội, list dài tải dần; thường kết hợp `IntersectionObserver` gọi `fetchNextPage()`.
- Bẫy: quên `flatMap` các `pages`; `getNextPageParam` sai khiến `hasNextPage` luôn `true` -> tải vô tận.

```tsx
const { data, fetchNextPage, hasNextPage } = useInfiniteQuery({
  queryKey: ['posts'],
  queryFn: ({ pageParam }) => fetchPosts({ cursor: pageParam }),
  initialPageParam: null as string | null,
  getNextPageParam: (last) => last.nextCursor ?? undefined,
});
const allPosts = data?.pages.flatMap((p) => p.items) ?? [];
```

</details>

---

### 5.10. SWR là gì và nó khác TanStack Query ở điểm nào? Khi nào bạn cân nhắc SWR?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"SWR là thư viện data fetching của Vercel, tên nó chính là chiến lược nó dùng: stale-while-revalidate, tức là trả ngay cache cũ cho người dùng thấy liền rồi âm thầm refetch ở nền để cập nhật. Về bản chất nó và TanStack Query cùng giải quyết một bài toán là server state, nhưng triết lý khác nhau. SWR tối giản, dùng cực gọn kiểu useSWR(key, fetcher), hợp khi app thiên về đọc dữ liệu là chính. TanStack Query thì mạnh và đầy đủ hơn nhiều: devtools tốt, API mutation chi tiết, optimistic update rõ ràng, query cancellation, infinite query. Nên mình cân nhắc SWR khi dự án Next.js muốn nhẹ nhàng ít cấu hình; còn khi cần mutation phức tạp, optimistic, hoặc cần debug nhiều thì mình chọn TanStack Query. Điểm chung quan trọng nhất là cả hai đều là công cụ cho server state, nên đừng tự cache dữ liệu API thủ công trong Redux nữa."

---

- **SWR** (Vercel): thư viện data-fetching theo chiến lược **stale-while-revalidate** — trả cache cũ ngay rồi refetch nền cập nhật. Tên = tên chiến lược.
- So với TanStack Query: SWR **tối giản hơn** (`useSWR(key, fetcher)`); TanStack **mạnh hơn** (devtools, mutation API chi tiết, optimistic rõ ràng, query cancellation).
- Cân nhắc SWR: dự án **Next.js**, muốn tối giản, ít cấu hình, fetch đọc là chính. Mutation/optimistic phức tạp, devtools mạnh -> TanStack Query.
- Điểm chung: **cả hai đều là server state** — đừng tự cache thủ công trong Redux.

```tsx
const { data, error, isLoading } = useSWR('/api/user', fetcher);
```

</details>

---

### 5.11. "Chuẩn hóa dữ liệu" (data normalization) là gì? Khi nào cần và khi nào không?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Chuẩn hóa dữ liệu là lưu state theo kiểu phẳng giống bảng quan hệ: mỗi thực thể chỉ có đúng một bản duy nhất, và các nơi khác tham chiếu tới nó qua id thay vì lồng cả object vào. Ngược lại với nó là lưu lồng nhau, dễ dẫn tới một thực thể bị nhân bản ở nhiều chỗ, sửa một nơi quên nơi kia là lệch dữ liệu. Mình cần chuẩn hóa khi có một store client tập trung như Redux chứa dữ liệu quan hệ, và muốn cập nhật một thực thể thì mọi nơi tham chiếu nó cùng đổi theo; RTK có sẵn createEntityAdapter cho dạng ids và entities này. Nhưng khi đã dùng TanStack Query hay RTK Query thì thường không cần chuẩn hóa, vì cache đã tổ chức theo queryKey và mình chỉ cần invalidate đúng query là xong. Tự chuẩn hóa khi không cần thiết chỉ làm tăng độ phức tạp vô ích, nên mình luôn cân nhắc theo bối cảnh chứ không mặc định làm."

---

- **Chuẩn hóa**: lưu phẳng, mỗi thực thể **một bản duy nhất**, tham chiếu bằng `id` (giống bảng quan hệ). Trái ngược là **lồng nhau (nested)** dễ trùng lặp một thực thể nhiều nơi.
- **Cần** khi: store **client tập trung** (Redux) với dữ liệu quan hệ, cập nhật một thực thể muốn mọi nơi tham chiếu cùng đổi. RTK có `createEntityAdapter` (`{ ids, entities }`).
- **Không cần** khi: dùng **TanStack/RTK Query** — cache theo `queryKey` + invalidation, chỉ cần invalidate đúng query. Tự chuẩn hóa khi không cần chỉ tăng độ phức tạp.

```ts
// ✅ Chuẩn hóa: tham chiếu bằng id
const state = {
  posts: { p1: { id: 'p1', title: 'A', authorId: 'u1' } },
  users: { u1: { id: 'u1', name: 'An' } },
};
```

</details>

---

### 5.12. "Derived state" là gì, và vì sao việc lưu state phái sinh vào `useState` thường là một anti-pattern?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Derived state là dữ liệu mà mình hoàn toàn tính ra được từ state hoặc props đã có sẵn, ví dụ danh sách đã lọc tính từ items và query. Nguyên tắc của mình là đừng lưu thứ tính được. Nếu mình nhét cái filtered đó vào một useState riêng thì sinh ra hai nguồn sự thật cho cùng một dữ liệu, rồi phải dùng useEffect để đồng bộ chúng, mà cách này rất dễ quên dependency, dễ lệch nhau, và còn render thừa một nhịp vì effect chạy sau render. Cách đúng đơn giản hơn nhiều: cứ tính trực tiếp ngay trong lúc render, và nếu phép tính nặng thật thì bọc useMemo cho khỏi tính lại. Mình chỉ đưa vào state đúng cái là nguồn sự thật tối thiểu, còn lại suy ra. Ngoại lệ hiếm hoi là khi cần giữ lại giá trị trước đó hay một snapshot có chủ ý, nhưng đó là trường hợp đặc biệt chứ không phải mặc định."

---

- **Derived state**: dữ liệu **tính ra được** từ state/props đã có. Nguyên tắc: **đừng lưu thứ tính được**.
- Lưu vào `useState` riêng -> **nguồn sự thật trùng lặp**, dễ lệch nhau, phải đồng bộ qua `useEffect` (dễ quên dependency, render thừa một nhịp).
- Cách đúng: tính **trực tiếp khi render**; memo hóa nếu phép tính nặng.
- Chỉ đưa vào state thứ **thực sự là nguồn sự thật tối thiểu**. Ngoại lệ hiếm: lưu "giá trị trước đó"/snapshot (chủ ý rõ ràng).

```tsx
// ❌ lưu filtered + useEffect đồng bộ
// ✅ tính khi render
const filtered = items.filter((i) => i.name.includes(query));
// ✅ memo nếu nặng
const filtered = useMemo(() => items.filter((i) => i.name.includes(query)), [items, query]);
```

</details>

---

### 5.13. Vì sao đôi khi nên lưu state trên URL thay vì trong React state? Trong Next.js bạn làm điều này thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Mình hay đặt lên URL những state mà người dùng cần chia sẻ hoặc khôi phục lại, ví dụ bộ lọc, từ khóa tìm kiếm, số trang, tab đang chọn, kiểu sắp xếp. Lợi ích là người ta copy link gửi cho nhau hoặc bookmark thì mở lại đúng trạng thái đó, F5 không mất, nút Back Forward của trình duyệt hoạt động đúng, và quan trọng là chỉ còn một nguồn sự thật thay vì phải tự đồng bộ giữa React state với URL. Trong Next.js App Router mình đọc bằng useSearchParams và cập nhật bằng useRouter, và mình ưu tiên router.replace thay vì push để không làm rác lịch sử khi người dùng gõ liên tục. Một mẹo rất hay là ghép URL state vào queryKey của TanStack Query, vì khi URL đổi thì key đổi và dữ liệu tự fetch lại luôn. Còn bẫy cần tránh là đừng nhồi state tạm hay dữ liệu nhạy cảm như token, form chưa submit vào URL."

---

- State **chia sẻ/khôi phục được** (filter, search, page, tab, sort) nên đặt trên URL (query string).
- Lợi ích: **chia sẻ/bookmark được**; **sống sót qua reload** (F5); **back/forward đúng**; **một nguồn sự thật** thay vì đồng bộ state ↔ URL.
- Next.js App Router: `useSearchParams` đọc, `useRouter` cập nhật; dùng `router.replace` để không spam history.
- Mẹo: kết hợp URL state với `queryKey` TanStack Query -> đổi URL là tự fetch lại.
- Bẫy: nhồi state tạm/nhạy cảm (token, form chưa submit) vào URL; dùng `push` thay `replace` khi gõ phím -> lịch sử đầy rác.

```tsx
const params = new URLSearchParams(searchParams);
value ? params.set('q', value) : params.delete('q');
router.replace(`${pathname}?${params.toString()}`);
```

</details>

---

### 5.14. Hãy đưa ra "cây quyết định" của bạn: gặp một nhu cầu state, bạn chọn công cụ nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Khi gặp một nhu cầu state mình đi qua một chuỗi câu hỏi chứ không chọn công cụ theo thói quen. Câu hỏi đầu tiên là dữ liệu này từ server hay là client state; nếu là server state thì mình dùng TanStack Query, hoặc SWR, hoặc RTK Query nếu dự án đã có Redux, và tuyệt đối không tự cache thủ công. Tiếp theo là nó có suy ra được từ thứ khác không, nếu có thì đó là derived state, mình tính ngay trong render và memo nếu nặng. Rồi mình hỏi nó có nên nằm trên URL không, kiểu filter, search, page, tab thì đưa lên URL qua useSearchParams. Cuối cùng mới xét phạm vi: chỉ một component thì useState hoặc useReducer; vài component gần nhau thì lift lên rồi truyền props; toàn cục mà ít đổi như theme thì Context có memo value; toàn cục hay đổi và cần selector thì Zustand hoặc Jotai; app lớn cần devtools và middleware thì Redux Toolkit. Tinh thần chung là bắt đầu từ công cụ đơn giản nhất rồi chỉ nâng cấp khi có lý do rõ ràng, và luôn tách server state khỏi client state."

---

Khung lập luận (không có đáp án duy nhất):
- **Server hay client state?** Server -> **TanStack Query** (hoặc SWR/RTK Query nếu đã có Redux), không tự cache.
- **Suy ra được không?** -> **derived state**, tính trong render (memo nếu nặng).
- **Nên ở URL không?** Filter/search/page/tab -> **URL** (`useSearchParams`).
- **Phạm vi?** 1 component -> `useState`/`useReducer`; vài component gần -> lift/props; toàn cục ít đổi -> **Context** (memo value); toàn cục hay đổi cần selector -> **Zustand/Jotai**; app lớn cần devtools/middleware -> **Redux Toolkit**.

Thông điệp: **bắt đầu từ công cụ đơn giản nhất** (`useState`), chỉ nâng cấp khi có lý do rõ (re-render thừa, prop drilling sâu), và **luôn tách server state khỏi client state**.

```text
Dữ liệu từ server?      -> TanStack Query / SWR / RTK Query
Tính được?              -> derived (useMemo)
Chia sẻ qua URL?        -> useSearchParams
Cục bộ?                 -> useState / useReducer
Toàn cục ít đổi?        -> Context (memo value)
Toàn cục hay đổi+selector? -> Zustand / Jotai
App lớn cần devtools?   -> Redux Toolkit
```

</details>

---

### 5.15. Kể về một lần bạn refactor cách quản lý state trong một dự án. Vấn đề ban đầu là gì và bạn cải thiện ra sao?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

**📌 Câu trả lời mẫu:**

"Mình từng refactor state cho một admin dashboard mà ban đầu nhét mọi thứ vào Redux, kể cả data lấy từ API. Mỗi màn lại tự viết fetchStart/Success/Failure, tự quản loading và cache, nên code phình to và đặc biệt hay dính bug dữ liệu cũ: hai màn dùng chung một list nhưng không màn nào tự đồng bộ với màn kia. Mục tiêu của mình là giảm boilerplate và xử lý triệt để chuyện dữ liệu lệch nhau, nhưng không được phá tính năng đang chạy. Cách tiếp cận là tách bạch server state và client state: phần data API mình chuyển sang TanStack Query với queryKey thống nhất, và sau mỗi mutation thì gọi invalidateQueries để mọi màn liên quan tự làm mới; phần client state nhỏ còn lại như modal hay wizard thì mình đưa sang Zustand dùng selector và bỏ hẳn Redux. Mình làm từng module một, viết test cho luồng quan trọng trước khi gỡ code cũ để an toàn. Kết quả là lượng code quản lý data giảm đáng kể, hết hẳn bug dữ liệu cũ vì giờ chỉ còn một cơ chế invalidation duy nhất, UX cũng nhanh hơn nhờ caching và background refetch, và đồng đội thêm màn mới nhanh hơn vì pattern đã rõ ràng. Điều mình muốn nhấn là quyết định dựa trên lý do kỹ thuật là tách server/client state, chứ không phải chỉ đổi thư viện cho mới."

---

Câu hỏi hành vi — dùng cấu trúc STAR. Mẫu trả lời điều chỉnh theo trải nghiệm thật:

- **Situation**: admin dashboard dùng Redux cho mọi thứ kể cả data API; mỗi màn tự viết `fetchStart/Success/Failure`, tự quản loading/cache. Code phình to, bug dữ liệu cũ vì hai màn dùng chung list nhưng không đồng bộ.
- **Task**: giảm boilerplate, xử lý triệt để dữ liệu lệch nhau, không phá tính năng đang chạy.
- **Action**: tách **server state** và **client state**. Đưa data API sang **TanStack Query**, dùng `queryKey` thống nhất + `invalidateQueries` sau mutation để mọi màn tự làm mới. Client state nhỏ còn lại (modal, wizard) chuyển sang **Zustand** với selector, bỏ Redux. Làm từng module, có test cho luồng quan trọng trước khi gỡ code cũ.
- **Result**: code quản lý data giảm đáng kể, hết bug dữ liệu cũ (chỉ còn một cơ chế invalidation), UX nhanh hơn nhờ caching/background refetch, đồng đội thêm màn mới nhanh hơn vì pattern rõ ràng.

Mẹo: nhấn vào **lý do kỹ thuật** (tách server/client state) và **kết quả đo được** (ít code, hết bug đồng bộ, nhanh hơn), thay vì chỉ "đổi thư viện".

</details>

## 6. React Ecosystem và Thư viện

### 6.1. `loader` và `action` trong React Router (data router) là gì? Chúng giải quyết vấn đề gì so với fetch trong `useEffect`?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Từ React Router v6.4 trở đi, với createBrowserRouter mình có thể gắn loader và action thẳng vào từng route. Loader chạy trước khi component render để nạp sẵn dữ liệu, rồi mình lấy ra bằng useLoaderData; cái này giải quyết được hai vấn đề của fetch trong useEffect: tránh fetch waterfall vì data nạp song song với việc chuyển route, và component chỉ render khi đã có dữ liệu nên không còn cảnh nhấp nháy loading rồi mới hiện. Action thì xử lý mutation, thường đi với Form method post, và sau khi action chạy xong router tự revalidate lại các loader liên quan, nên mình không phải tự đi refetch. Về bản chất đây là cách React Router đẩy phần lấy dữ liệu ra khỏi component và đưa vào tầng routing. Lưu ý là cái này hợp khi mình dùng React Router làm data layer cho SPA; còn nếu đã có TanStack Query hoặc đang dùng Next.js thì thường không cần, quan trọng là chọn một mô hình nhất quán chứ đừng trộn nhiều cơ chế."

---

- Từ v6.4 (`createBrowserRouter`), router gắn `loader`/`action` trực tiếp vào route.
- **`loader`**: chạy **trước khi** render để nạp data, lấy bằng `useLoaderData()` → tránh fetch waterfall, render khi đã có data (hết loading nhấp nháy).
- **`action`**: xử lý mutation (`<Form method="post">`), sau đó **tự revalidate** loader liên quan.

```tsx
loader: async ({ params }) => {
  const res = await fetch(`/api/users/${params.id}`);
  if (!res.ok) throw new Response('Not Found', { status: 404 });
  return res.json();           // -> useLoaderData()
},
```

- **Lưu ý:** hợp khi dùng RR làm data layer của SPA. Nếu đã có **TanStack Query** hoặc **Next.js** thì thường không cần — chọn một mô hình nhất quán.

</details>

---

### 6.2. Cách triển khai một Protected Route (yêu cầu đăng nhập) trong React Router?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Cách mình làm protected route là tạo một component bảo vệ đứng giữa: nếu chưa đăng nhập thì điều hướng về trang login bằng Navigate với replace, đồng thời lưu lại vị trí người dùng đang muốn vào qua state from để sau khi login xong quay lại đúng chỗ; còn nếu đã đăng nhập thì render Outlet để cho đi tiếp vào các route con. Mình dùng replace chứ không push để khi bấm Back người dùng không bị quay lại trang protected đang chặn. Điểm cực kỳ quan trọng phải nói rõ là kiểu chặn này chỉ là bảo vệ phía client cho mục đích UX thôi, phân quyền thật bắt buộc phải nằm ở backend, vì người ta hoàn toàn có thể bỏ qua client để gọi thẳng API. Nếu dùng data router thì mình có thể chặn sớm hơn nữa bằng cách throw redirect về login ngay trong loader, để route thậm chí không render khi chưa đủ quyền."

---

- Tạo component bảo vệ: chưa login → `<Navigate to="/login" replace>` (lưu vị trí muốn vào qua `state`), đã login → render `<Outlet />`.
- `replace` để bấm Back không quay lại trang protected; sau login điều hướng về `from`.

```tsx
function RequireAuth() {
  const { user } = useAuth();
  const location = useLocation();
  if (!user) return <Navigate to="/login" replace state={{ from: location }} />;
  return <Outlet />;
}
```

- **Bảo mật:** protected route chỉ là bảo vệ phía client (UX); phân quyền thật **phải** ở backend. Với data router có thể `throw redirect('/login')` ngay trong `loader`.

</details>

---

### 6.3. Tại sao React Hook Form có hiệu năng tốt hơn so với cách quản lý form bằng state thông thường?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Vấn đề của form controlled thông thường là mỗi input phải gắn value và onChange vào React state, nên cứ gõ một ký tự là một lần setState, mà setState thì re-render lại toàn bộ form. Form vài field thì không sao, nhưng form lớn vài chục field thì gõ bắt đầu thấy khựng. React Hook Form đi theo hướng uncontrolled: nó dùng register để gắn input vào DOM qua ref và đọc giá trị trực tiếp từ DOM, nên khi mình gõ thì React không hề re-render, hiệu năng gần như không phụ thuộc số lượng field. Bản chất là RHF tránh việc biến mỗi phím gõ thành một lần cập nhật state của React. Đổi lại, vì dựa trên ref nên với mấy component UI thư viện không expose ref chuẩn như MUI Select hay react-select thì mình phải dùng Controller để bọc lại, lúc đó field đó mới controlled. Mình thường dùng RHF cho mọi form thật sự, đặc biệt form dài hoặc cần validate, còn useState chỉ để cho một hai input lặt vặt."

---

- Form controlled truyền thống: mỗi input gắn `value`+`onChange` vào state → **mỗi lần gõ là một `setState` → re-render toàn form** (form lớn lag).
- RHF dùng **uncontrolled** qua `ref`: `register` gắn input vào DOM, không setState khi gõ → **gõ phím không re-render**, hiệu năng gần như độc lập số field.

```tsx
const { register, handleSubmit, formState: { errors } } = useForm();
<form onSubmit={handleSubmit(onSubmit)}>
  <input {...register('email', { required: 'Bắt buộc nhập email' })} />
  {errors.email && <span>{errors.email.message}</span>}
</form>
```

- Đổi lại: input từ component UI thư viện (không nhận `ref` chuẩn) phải dùng `Controller` (xem câu sau).

</details>

---

### 6.4. Phân biệt `register` và `control` (Controller) trong React Hook Form. Khi nào dùng cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"register và Controller là hai cách để đưa một input vào React Hook Form, khác nhau ở cơ chế kiểm soát. register là cách uncontrolled, RHF gắn ref vào input HTML gốc và đọc giá trị từ DOM, nên gõ không re-render, đây là cách tối ưu nhất và nên là mặc định cho input, textarea, select gốc. Controller thì ngược lại, nó tạo một cầu nối controlled, RHF tự quản value và onChange rồi đưa xuống cho component qua render prop. Mình chỉ dùng Controller khi cần tích hợp component bên thứ ba không nhận ref một cách chuẩn, ví dụ MUI Select, react-select, date picker, những thứ chỉ làm việc qua value và onChange. Điểm cần nhớ là Controller chỉ re-render đúng field đó chứ không phải cả form, nhưng nếu lạm dụng nhồi mọi thứ vào Controller thì mình mất luôn ưu thế hiệu năng vốn là lý do chọn RHF. Nên quy tắc của mình là ưu tiên register, chỉ rút Controller ra khi thật sự cần."

---

| | `register` | `Controller`/`control` |
|---|---|---|
| Cơ chế | uncontrolled qua `ref` | controlled, RHF tự quản |
| Dùng cho | input HTML gốc | component UI không expose `ref` (MUI Select, react-select...) |
| Hiệu năng | tối ưu (không re-render khi gõ) | re-render chỉ field đó |

```tsx
<input {...register('username')} />
<Controller name="country" control={control}
  render={({ field }) => <Select {...field} options={options} />} />
```

- **Quy tắc:** ưu tiên `register` cho input gốc; chỉ dùng `Controller` khi component bên thứ ba cần `value`+`onChange` kiểm soát. Bẫy: nhồi mọi thứ vào `Controller` làm mất ưu thế hiệu năng.

</details>

---

### 6.5. Tích hợp Zod (hoặc Yup) với React Hook Form như thế nào? Ưu điểm của schema validation?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Ý tưởng là tách phần luật validate ra thành một schema riêng bằng Zod hoặc Yup, rồi nối vào React Hook Form qua package @hookform/resolvers, cụ thể là zodResolver. Khi đó RHF không tự viết rule rải rác trong từng register nữa mà giao toàn bộ việc validate cho schema, code gọn và một chỗ duy nhất để nhìn. Ưu điểm lớn nhất của Zod với mình là type inference: mình định nghĩa schema một lần rồi dùng z.infer để suy ra luôn TypeScript type của form, không phải khai báo type hai lần và không sợ schema với type lệch nhau. Cùng một schema đó mình còn tái dùng được ở phía server để validate request, nên client và server dùng chung một nguồn sự thật. Với mấy luật chéo như mật khẩu phải khớp confirm thì mình dùng refine của Zod để gắn lỗi vào đúng field. So với Yup thì Yup phổ biến và quen thuộc hơn nhưng inference yếu hơn, còn Zod thì sinh ra cho TypeScript nên giờ mình mặc định chọn Zod."

---

- Với form phức tạp, tách validation ra **schema** Zod/Yup, nối qua `@hookform/resolvers` (`zodResolver`).
- Ưu điểm lớn nhất của Zod: **type inference** (`z.infer`) — định nghĩa schema một lần, suy ra TypeScript type; tái dùng được cả client lẫn server.
- Validate logic chéo (khớp mật khẩu) qua `refine` (Zod) / `test`/`when` (Yup).

```tsx
const schema = z.object({
  password: z.string().min(8), confirm: z.string(),
}).refine((d) => d.password === d.confirm, { path: ['confirm'] });
type FormValues = z.infer<typeof schema>;
useForm<FormValues>({ resolver: zodResolver(schema) });
```

- **So với Yup:** Zod ưu tiên TypeScript (inference tốt hơn, không cần type bổ sung); Yup phổ biến rộng nhưng inference yếu hơn.

</details>

---

### 6.6. So sánh các phương án styling: CSS Modules, styled-components/Emotion (CSS-in-JS), và Tailwind CSS. Trade-off của mỗi cách?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Ba cách này khác nhau chủ yếu ở chỗ style sống ở đâu và phải trả giá runtime hay không. CSS Modules là viết CSS thuần trong file .module.css rồi import vào, Next tự scope tên class nên không lo đụng tên toàn cục, gần như không có chi phí runtime, nhẹ và an toàn, hợp khi mình muốn viết CSS bình thường. CSS-in-JS như styled-components hay Emotion thì mạnh ở chỗ style động theo props rất tự nhiên, nhưng đa số bản có chi phí runtime để sinh class lúc chạy, và nhiều thư viện loại này vướng với React Server Components trong App Router. Tailwind thì viết utility class thẳng trên JSX, style được purge lúc build nên không có runtime cost, làm việc tốt với App Router và rất nhanh để dựng giao diện cùng design system, đổi lại JSX nhìn hơi rậm. Thực tế mình hay chọn Tailwind cho dự án Next App Router vì hợp RSC và đi nhanh, CSS Modules khi muốn CSS thuần gọn nhẹ, còn CSS-in-JS chỉ khi thật sự cần style động phức tạp và chấp nhận đánh đổi."

---

| Tiêu chí | CSS Modules | CSS-in-JS | Tailwind |
|---|---|---|---|
| Cách viết | `.module.css` scoped | CSS trong JS | utility class trên JSX |
| Dynamic theo props | khó | mạnh (theo props) | qua `cva`/biến class |
| Runtime cost | không | **có** (trừ bản zero-runtime) | không (purge lúc build) |
| RSC (App Router) | tốt | nhiều lib gặp khó | tốt |

```tsx
<button className={styles.primary}>Lưu</button>            // CSS Modules
const Button = styled.button<{ $primary?: boolean }>`...`; // styled
<button className="bg-blue-600 px-4 py-2 rounded">Lưu</button> // Tailwind
```

- **Chọn:** Tailwind cho prototyping nhanh + design system + Next App Router; CSS-in-JS khi cần style động mạnh (nhưng runtime + vướng RSC); CSS Modules khi muốn CSS thuần, scope an toàn, nhẹ.

</details>

---

### 6.7. So sánh thư viện component MUI, Radix UI và shadcn/ui. Triết lý "headless" và "copy-paste component" của shadcn khác gì?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Ba cái này nằm ở ba mức kiểm soát giao diện khác nhau. MUI là bộ component đầy đủ có sẵn cả logic lẫn style theo Material Design, ra sản phẩm cực nhanh nhưng đổi lại mình bị bó theo giao diện của họ, muốn tùy biến sâu thì khá vất. Radix UI thì theo triết lý headless, nghĩa là nó chỉ lo phần khó và dễ sai nhất là hành vi, focus management, ARIA và điều hướng bàn phím, còn giao diện thì không áp đặt gì cả, mình tự style hoàn toàn. shadcn/ui thì đặc biệt ở chỗ nó không phải một npm package mà là bộ component dựng sẵn trên Radix cộng Tailwind, và mình dùng CLI để copy thẳng mã nguồn vào repo của mình. Điều đó nghĩa là mình sở hữu code đó, sửa thoải mái, không bị khóa version và không lo breaking change khi thư viện update. Cách mình chọn là MUI khi cần ra sản phẩm gấp và chấp nhận Material, còn Radix hoặc shadcn khi cần một design system riêng và muốn kiểm soát UI hoàn toàn, đây cũng là lựa chọn mình thích cho các dự án có thương hiệu riêng."

---

| | MUI | Radix UI | shadcn/ui |
|---|---|---|---|
| Loại | component + style sẵn | **headless** (logic+a11y, không style) | build sẵn trên Radix + Tailwind |
| Phân phối | npm dependency | npm primitives | **copy code vào repo** qua CLI |
| Kiểm soát | thấp | cao | cao nhất (bạn sở hữu code) |

- **Headless**: thư viện lo phần khó (hành vi, focus, ARIA/bàn phím) nhưng **không áp đặt giao diện** — bạn tự style (Radix).
- **shadcn/ui không phải npm package**: copy mã nguồn vào dự án → sửa thoải mái, không khóa version, không lo breaking change. Dùng Radix + Tailwind + cva bên dưới.

```tsx
<Button variant="destructive" size="sm">Xóa</Button>; // shadcn (code của bạn)
<Dialog.Root>...</Dialog.Root>;                       // Radix headless
```

- **Chọn:** MUI để ra sản phẩm nhanh (chấp nhận Material); Radix/shadcn khi cần design system riêng + kiểm soát UI hoàn toàn.

</details>

---

### 6.8. Khi chọn một thư viện cho dự án (vd state management hay form), bạn dựa trên những tiêu chí nào để quyết định?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

**📌 Câu trả lời mẫu:**

"Khi chọn thư viện mình đi theo một bộ tiêu chí có hệ thống chứ không chạy theo trend. Đầu tiên là hỏi liệu có thật sự cần thư viện không, vì thêm một dependency là thêm chi phí bảo trì và rủi ro; nhiều bài toán dùng đồ có sẵn của React là đủ. Tiếp theo mình xét mức độ phù hợp với bài toán, ví dụ ở dự án Jira Clone khi cần quản lý server state của board và issue thì mình chọn TanStack Query thay vì nhồi vào Redux, vì server state cần caching, dedupe và invalidation mà client state không cần. Sau đó mình nhìn sức khỏe dự án: maintainer còn active không, tần suất release, issue tồn đọng, commit gần nhất; rồi đến cộng đồng và tài liệu, bundle size, và chất lượng TypeScript/DX. Cuối cùng là độ tương thích với stack hiện tại và chi phí migration cũng như nguy cơ lock-in. Với những lựa chọn lớn mình thường làm một POC nhỏ để có dữ liệu thật trước khi cam kết, rồi trình bày trade-off cho team để cả nhóm cùng quyết. Tinh thần chung là ưu tiên giải pháp đơn giản nhất giải quyết được bài toán, và quyết định dựa trên dữ liệu chứ không phải cảm tính."

---

- **Situation**: team cần chọn giải pháp quản lý server state cho dashboard đang phình to, đang fetch rải rác trong `useEffect`, khó cache.
- **Task**: đánh giá và đề xuất thư viện (vd TanStack Query vs RTK Query vs tự viết), thuyết phục team.
- **Action** — tiêu chí có hệ thống:
  1. Có **thật sự cần** thư viện không (tránh dependency thừa).
  2. Mức độ **phù hợp** với bài toán (server state khác client state).
  3. **Sức khỏe dự án**: maintainer, tần suất release, issue tồn đọng, commit gần nhất.
  4. **Cộng đồng & tài liệu**, **bundle size**, **TypeScript/DX**.
  5. **Tương thích stack** (RSC nếu dùng Next App Router) + chi phí **migration/lock-in**.
- **Result + Bài học**: chọn được giải pháp giảm code fetch, có cache/retry sẵn, onboard nhanh; quyết định dựa trên **dữ liệu + trade-off**, làm POC nhỏ trước khi cam kết.

👉 Điểm ăn: thể hiện cân nhắc **trade-off**, ưu tiên giải pháp đơn giản nhất, biết đánh giá sức khỏe/rủi ro dependency thay vì chạy theo trend.

</details>

## 7. Next.js và Cơ chế Rendering

### 7.1. Hãy phân biệt CSR, SSR, SSG và ISR. Khi nào bạn chọn mỗi chiến lược?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Bốn chiến lược này khác nhau ở câu hỏi HTML được tạo ra ở đâu và vào lúc nào. CSR là render ngay trong trình duyệt sau khi tải JS, hợp cho mấy màn hình sau đăng nhập như dashboard, nơi không cần SEO và dữ liệu riêng từng người. SSR là render trên server theo từng request, hợp khi cần dữ liệu thời gian thực hoặc cá nhân hóa như giỏ hàng hay feed. SSG là render sẵn lúc build, HTML tĩnh, hợp blog, docs, landing ít thay đổi và muốn nhanh nhất qua CDN. ISR là dung hòa giữa SSG và SSR: vẫn phục vụ trang tĩnh nhưng sau một khoảng revalidate thì Next tự dựng lại ở nền, hợp e-commerce hay tin tức cần vừa nhanh vừa SEO mà dữ liệu không cần tươi tuyệt đối. Một điều mình hay nhấn là SSR không phải luôn tốt hơn, vì nó tốn server mỗi request và TTFB chậm hơn trang tĩnh; nếu không cần SEO hay cá nhân hóa thì SSG hoặc ISR phục vụ qua CDN sẽ tối ưu hơn nhiều. Mình chọn theo từng trang chứ không áp một kiểu cho cả app."

---

- Khác biệt cốt lõi: **HTML được tạo ra ở đâu và khi nào**.
- **CSR**: render trong trình duyệt sau khi tải JS → dashboard sau login, không cần SEO.
- **SSR**: render mỗi request trên server → dữ liệu thời gian thực, cá nhân hóa (giỏ hàng, feed).
- **SSG**: render lúc build, dữ liệu cố định → blog, docs, landing ít đổi.
- **ISR**: SSG + tự tạo lại ở nền sau `revalidate` giây → e-commerce, tin tức (SEO + dữ liệu khá mới).

```tsx
export const revalidate = 60;            // ISR: tạo lại sau 60s
export const dynamic = 'force-dynamic';  // SSR thuần, render mỗi request
```

**Bẫy:** SSR không phải lúc nào cũng "tốt hơn" — tốn server mỗi request, TTFB chậm hơn trang tĩnh; không cần SEO/cá nhân hóa thì SSG/ISR (CDN) tối ưu hơn.

</details>

---

### 7.2. App Router khác Pages Router ở những điểm cốt lõi nào? Có nên migrate dự án cũ không?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Khác biệt cốt lõi là App Router xây trên React Server Components nên component mặc định chạy ở server và không gửi JS xuống client, còn Pages Router thì mọi thứ về bản chất là Client Component và data lấy qua getServerSideProps hoặc getStaticProps. Với App Router mình fetch dữ liệu bằng async await ngay trong component, có layout lồng nhau giữ được state khi điều hướng, có file quy ước như loading.tsx và error.tsx, dùng được Suspense và streaming một cách tự nhiên, và mutation thì dùng Server Actions thay cho việc tự viết API rồi fetch. Về câu có nên migrate hay không, điểm quan trọng là hai router cùng tồn tại được trong một dự án, nên mình không phải viết lại toàn bộ; cách hợp lý là migrate dần, route mới thì viết trong app còn route cũ để nguyên. App Router có learning curve cao hơn vì phải hiểu RSC, ranh giới server client và cơ chế caching, nên với một dự án nhỏ đang chạy ổn định thì giữ Pages Router vẫn hoàn toàn hợp lý chứ không nhất thiết phải đổi."

---

- **App Router** (`app/`, ổn định từ 13.4): dựa trên **React Server Components**, Server Component mặc định.
- **Pages Router** (`pages/`): toàn bộ là Client Component, data qua `getServerSideProps`/`getStaticProps`.
- App Router: data bằng `async/await` ngay trong component, layout lồng nhau giữ state, file quy ước `loading.tsx`/`error.tsx`, native Suspense/streaming, Server Actions cho mutation.

| | Pages | App |
|---|---|---|
| Data | `getServerSideProps`... | `async` component |
| API | `pages/api/*` | Route Handlers `route.ts` |
| Mutation | API + fetch | Server Actions |

**Có nên migrate?** Hai router **cùng tồn tại** (incremental adoption) — nên migrate dần (route mới viết trong `app/`). App Router learning curve cao hơn (RSC, ranh giới server/client, caching); dự án nhỏ ổn định thì Pages Router vẫn hợp lệ.

</details>

---

### 7.3. Server Components và Client Components khác nhau thế nào? Chỉ thị `'use client'` thực sự làm gì?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Trong App Router thì mặc định mọi component là Server Component, tức là chỉ chạy trên server và không gửi JS xuống trình duyệt, nhờ vậy mình truy cập database hay dùng secret an toàn ngay trong component, nhưng đổi lại không dùng được useState, useEffect, event handler hay window. Client Component thì ngược lại, dùng được hook, sự kiện và browser API, nhưng phải gửi JS xuống nên làm tăng bundle. Chỉ thị use client đặt ở đầu file là cách mình đánh dấu file đó và cả nhánh import từ nó là Client Component, để nó được hydrate và chạy trên trình duyệt. Có vài điểm dễ hiểu nhầm mình hay lưu ý: use client không có nghĩa là render hoàn toàn ở client, lần đầu nó vẫn được SSR rồi mới hydrate; nên đặt use client càng gần lá càng tốt để giữ bundle nhỏ; không import một Server Component vào trong Client Component mà phải truyền qua children theo kiểu composition; và props truyền từ server xuống client phải serializable, không truyền được function hay class instance. Bản chất là mình muốn giữ phần lớn cây ở server và chỉ biến những chỗ thật sự cần tương tác thành client."

---

- Mặc định **mọi component là Server Component (RSC)** — chỉ chạy trên server, không gửi JS xuống client.
- `'use client'` đánh dấu file (và cả cây import từ nó) là Client Component → được hydrate, chạy trên trình duyệt.
- Server Component: truy cập DB/secret an toàn, KHÔNG dùng `useState`/`useEffect`/event handler/`window`.
- Client Component: dùng được hook, event, browser API nhưng gửi JS → tăng bundle.

**Điểm quan trọng / bẫy:**
- `'use client'` **không** nghĩa là render hoàn toàn ở client — vẫn được SSR lần đầu rồi mới hydrate.
- Đặt `'use client'` càng "sâu" (lá) càng tốt để giữ bundle nhỏ.
- Không `import` Server Component vào Client Component; chỉ truyền qua props `children` (composition).
- Props Server → Client phải **serializable** (không truyền function, class instance...).

</details>

---

### 7.4. Hydration là gì? Hydration mismatch xảy ra vì sao và làm sao tránh?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Hydration là bước React gắn JS vào phần HTML tĩnh mà server đã trả về: nó dựng lại Virtual DOM và attach các event listener để trang trở nên tương tác, mà không phải vẽ lại DOM từ đầu. Để làm được điều đó, React giả định cây HTML mà client render ở lần đầu phải khớp y hệt với HTML mà server đã gửi. Hydration mismatch là khi hai cái đó lệch nhau, lúc đó React cảnh báo, và nặng thì nó vứt HTML của server đi render lại từ client, gây chớp màn hình và mất hiệu năng SEO. Nguyên nhân thường là render giá trị không ổn định giữa hai phía như Date.now hay Math.random, lệch locale hoặc timezone, hoặc đọc window và localStorage ngay lúc render, ví dụ kinh điển là đọc theme từ localStorage. Cách mình tránh là giữ lần render đầu thuần khiết, để dữ liệu chỉ có ở client thì hoãn vào useEffect cập nhật sau khi mount; widget nào không SSR được thì mình dùng next/dynamic với ssr false; còn suppressHydrationWarning mình chỉ coi là giải pháp cuối cùng cho mấy trường hợp như hiển thị thời gian, chứ không dùng để giấu lỗi thật."

---

- **Hydration**: React "gắn" JS vào HTML tĩnh từ server (dựng VDOM, attach listener) để trang tương tác mà không vẽ lại DOM.
- React giả định cây HTML client render lần đầu phải **khớp y hệt** HTML server.
- **Mismatch**: HTML server ≠ render đầu ở client → cảnh báo, nặng thì vứt HTML server render lại (chớp màn hình, mất hiệu năng).
- Nguyên nhân: giá trị không ổn định (`Date.now()`, `Math.random()`, locale/timezone), đọc `window`/`localStorage` khi render, theme từ localStorage, HTML không hợp lệ (`<div>` trong `<p>`).

```tsx
// ✅ render giá trị an toàn ở server, cập nhật sau mount
function Clock() {
  const [time, setTime] = useState<string | null>(null);
  useEffect(() => setTime(new Date().toLocaleTimeString()), []);
  return <span>{time ?? '—'}</span>;
}
```

**Cách tránh:** render lần đầu thuần khiết, hoãn dữ liệu chỉ-có-ở-client sang `useEffect`; widget không SSR được dùng `next/dynamic` với `ssr: false`; `suppressHydrationWarning` chỉ là giải pháp cuối.

</details>

---

### 7.5. Trong App Router, `fetch` được mở rộng thế nào với `cache`, `no-store` và `revalidate`?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Trong App Router thì Next patch lại hàm fetch gốc để thêm khả năng caching và revalidation, mình điều khiển qua option thứ hai. Nếu để cache force-cache thì kết quả được cache vĩnh viễn và trang coi như tĩnh kiểu SSG; để no-store thì không cache, render động lại mỗi request giống SSR; còn next revalidate 60 thì là kiểu ISR, vẫn cache nhưng tự làm mới ở nền sau 60 giây. Ngoài ra mình hay gắn next tags cho fetch để sau này chủ động xóa cache có chủ đích bằng revalidateTag, hoặc revalidatePath, ngay trong Server Action sau khi mutation. Một lưu ý quan trọng là hành vi mặc định đổi theo phiên bản: Next 13 và 14 mặc định cache fetch và cố gắng làm trang tĩnh, nhưng từ Next 15 thì mặc định không cache nữa, kể cả GET Route Handler, nên mình phải chủ động opt-in nếu muốn cache. Nguyên tắc chọn của mình là dùng revalidate cho dữ liệu chấp nhận hơi cũ một chút, no-store cho dữ liệu nhạy thời gian hoặc cá nhân hóa, và dùng tag để invalidate đúng phần liên quan sau khi ghi dữ liệu."

---

- Next patch `fetch` gốc để thêm caching/revalidation, điều khiển qua option thứ hai.

```tsx
fetch(url, { cache: 'force-cache' });     // cache vĩnh viễn -> tĩnh (SSG)
fetch(url, { cache: 'no-store' });        // không cache -> động mỗi request (SSR)
fetch(url, { next: { revalidate: 60 } }); // ISR: cache, làm mới sau 60s
fetch(url, { next: { tags: ['products'] } }); // gắn tag để chủ động invalidate
```

- Chủ động xóa cache trong Server Action/Route Handler: `revalidateTag('products')`, `revalidatePath('/products/[id]')`.

**Lưu ý phiên bản:**
- Next 13/14: `fetch` mặc định **được cache** (`force-cache`), route cố gắng tĩnh.
- Next 15: mặc định **không cache** (`no-store`), `GET` Route Handler không cache mặc định — phải opt-in.

**Nguyên tắc:** `revalidate` cho dữ liệu tolerant-stale, `no-store` cho dữ liệu nhạy thời gian/cá nhân hóa, tag để invalidate có chủ đích sau mutation.

</details>

---

### 7.6. `generateStaticParams` dùng để làm gì? Cho ví dụ với route động.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"`generateStaticParams` là cách mình nói cho Next biết với một dynamic route như `app/blog/[slug]/page.tsx` thì những giá trị param nào cần được render sẵn thành HTML tĩnh ngay lúc build. Mình trả về một mảng các object param, mỗi phần tử tương ứng một trang được build trước, nên người dùng tải vào là có ngay, rất tốt cho SEO và tốc độ. Nó thay cho `getStaticPaths` của Pages Router, và kết hợp với `export const revalidate` cùng `dynamicParams` thì mình có đúng pattern ISR: build sẵn các trang phổ biến, các slug lạ thì render on-demand rồi cache lại, và tự làm mới theo chu kỳ. Bẫy lớn nhất là nếu mình generate cả vài trăm nghìn slug lúc build thì build sẽ cực chậm, nên mình chỉ build trước những trang quan trọng và để `dynamicParams: true` lo phần còn lại. Khi muốn slug lạ trả 404 thay vì render thì mình đặt `dynamicParams: false`."

---

- `generateStaticParams` (thay `getStaticPaths`) báo cho Next biết **những param nào cần render tĩnh lúc build** cho dynamic route như `app/blog/[slug]/page.tsx`.
- Trả về mảng param → mỗi phần tử là một trang tĩnh.

```tsx
export async function generateStaticParams() {
  const posts = await fetch('https://api/posts').then(r => r.json());
  return posts.map((p: { slug: string }) => ({ slug: p.slug }));
}
```

- `dynamicParams` (mặc định `true`): param lạ → render on-demand rồi cache (giống ISR); đặt `false` → trả 404.
- Kết hợp `export const revalidate = 3600` → pattern ISR cổ điển: build trang phổ biến, phần còn lại sinh khi truy cập, tự làm mới theo chu kỳ.

**Bẫy:** chạy lúc build → danh sách quá lớn (hàng trăm nghìn trang) làm build rất chậm; nên chỉ generate trang quan trọng, phần còn lại để `dynamicParams: true`.

</details>

---

### 7.7. Server Actions là gì? Cho ví dụ form submit và nêu các lưu ý bảo mật.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Server Action là một hàm async chạy hẳn trên server nhưng mình gọi nó trực tiếp từ component, kể cả Client Component, mà không phải tự dựng API endpoint riêng. Mình đánh dấu nó bằng `'use server'` và chủ yếu dùng cho mutation như tạo, sửa, xóa hay submit form, ví dụ gắn thẳng vào `<form action={createPost}>`. Điểm mình luôn nhấn mạnh khi phỏng vấn là về bản chất Server Action vẫn là một HTTP endpoint công khai bị ẩn đi, nên không bao giờ được tin frontend: phải validate input bằng Zod và kiểm tra quyền ngay bên trong action, đừng dựa vào việc disable nút hay ẩn form. Ở client mình thường kết hợp `useFormStatus` để bắt trạng thái pending hoặc `useActionState` của React 19 để xử lý loading và lỗi gọn gàng. Mình chọn Server Action khi mutation gắn liền với UI nội bộ vì nó type-safe và ít boilerplate; còn khi cần API công khai cho mobile, webhook hay bên thứ ba thì mình dùng Route Handler."

---

- Hàm async chạy **trên server** nhưng gọi trực tiếp từ component (kể cả client) mà không cần tự viết API endpoint.
- Đánh dấu bằng `'use server'`, thường dùng cho **mutation** (tạo/sửa/xóa, submit form).

```tsx
async function createPost(formData: FormData) {
  'use server';
  const title = formData.get('title') as string;
  if (!title?.trim()) throw new Error('Title required');
  await db.post.create({ data: { title } });
  revalidatePath('/posts');
  redirect('/posts');
}
// <form action={createPost}>...
```

- Ở Client Component kết hợp `useFormStatus` (pending) hoặc `useActionState` (React 19) để xử lý loading/lỗi.

**Bảo mật — cực kỳ quan trọng:**
- Server Action là **public HTTP endpoint** ẩn → luôn validate (Zod) và kiểm tra quyền (authz) **bên trong action**, đừng tin frontend.
- Đừng dựa vào "nút disable"/"form ẩn"; kiểm tra session/role ngay trong action.
- Tránh truyền dữ liệu nhạy cảm/closure vào action vì chúng bị tuần tự hóa.

**Khi nào dùng:** mutation gắn UI (form, like, toggle). Dùng Route Handler khi cần API công khai/versioning/gọi từ client ngoài (mobile, webhook) hoặc kiểm soát HTTP chi tiết.

</details>

---

### 7.8. Giải thích hệ thống file routing đặc biệt trong App Router: `layout`, `loading`, `error`, `template`.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"App Router dùng các file đặt tên theo quy ước trong mỗi thư mục route để định nghĩa phần UI bao quanh `page.tsx`, nhờ vậy mình tách rõ từng concern. `layout.tsx` là khung dùng chung, nó mount một lần và giữ nguyên state cùng scroll khi mình điều hướng giữa các trang con nên không bị re-render. `template.tsx` trông giống layout nhưng tạo instance mới mỗi lần điều hướng, mình dùng khi muốn effect chạy lại hoặc cần animation enter mỗi lần vào trang. `loading.tsx` thực chất là Next tự bọc page của mình trong `<Suspense>` để hiển thị fallback trong lúc tải, còn `error.tsx` là error boundary cho segment, bắt buộc là Client Component và nhận vào `error` cùng hàm `reset()` để thử lại. Một bẫy hay bị hỏi là `error.tsx` không bắt được lỗi trong chính `layout.tsx` cùng cấp với nó, vì boundary nằm bên trong layout đó, nên muốn bắt lỗi ở root layout thì mình phải dùng `global-error.tsx`."

---

App Router dùng **file quy ước** trong mỗi thư mục route để định nghĩa UI bao quanh `page.tsx`:

| File | Vai trò |
|---|---|
| `layout.tsx` | Khung dùng chung, **giữ state** khi điều hướng, không re-render |
| `template.tsx` | Như layout nhưng **tạo instance mới mỗi điều hướng** (reset state/effect) |
| `loading.tsx` | Fallback khi tải; tự bọc page trong `<Suspense>` |
| `error.tsx` | Error boundary segment; **phải là Client Component**, nhận `error` + `reset()` |
| `not-found.tsx` | UI cho `notFound()`/404 |
| `global-error.tsx` | Error boundary cấp root, bắt cả lỗi root layout |

**Hay bị hỏi — `layout` vs `template`:** layout mount một lần, giữ state/scroll; template remount mỗi điều hướng → dùng khi cần effect chạy lại hoặc animation enter.

**Lưu ý:** `error.tsx` không bắt được lỗi trong chính `layout.tsx` cùng cấp (boundary nằm trong layout đó) — cần `global-error.tsx` cho lỗi root layout.

</details>

---

### 7.9. Route groups, parallel routes và intercepting routes là gì? Mỗi loại giải quyết vấn đề gì?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Ba pattern này giải quyết ba vấn đề tổ chức route khác nhau. Route Groups dùng cú pháp `(folder)` để mình nhóm code hoặc gắn layout riêng cho một nhóm trang mà không thêm segment nào vào URL, ví dụ `app/(shop)/cart` vẫn ra đường dẫn `/cart`. Parallel Routes dùng `@slot` cho phép mình render nhiều page độc lập cùng lúc trong một layout, mỗi slot có loading và error riêng, rất hợp với dashboard có nhiều khu vực tải song song. Intercepting Routes dùng `(.)`, `(..)`, `(...)` để chặn một route và hiển thị nó trong context hiện tại, kinh điển nhất là modal ảnh kiểu Instagram: bấm vào ảnh từ feed thì mở modal, nhưng nếu vào thẳng URL đó thì vẫn ra trang full. Thực tế intercepting thường đi chung với parallel route qua một slot `@modal` để khi đóng modal thì quay về trang nền mượt mà mà không mất state."

---

| Pattern | Cú pháp | Giải quyết |
|---|---|---|
| **Route Groups** | `(folder)` | Tổ chức code / layout riêng **không thêm segment vào URL** |
| **Parallel Routes** | `@slot` | Render **nhiều page độc lập trong cùng layout**, mỗi slot có loading/error riêng (dashboard) |
| **Intercepting Routes** | `(.)`, `(..)`, `(...)` | "Chặn" route hiện trong context hiện tại (modal); URL trực tiếp vẫn ra trang full |

- Route Groups: `app/(shop)/cart/page.tsx` → `/cart`, layout riêng cho nhóm.
- Parallel: layout nhận props `{ children, team, analytics }`, render các slot song song.
- Intercepting: kinh điển là **modal ảnh** (Instagram) — `feed/@modal/(.)photo/[id]/page.tsx` mở modal khi điều hướng từ feed.

Intercepting thường đi kèm parallel route (`@modal`) để đóng modal quay lại trang nền mượt mà.

</details>

---

### 7.10. Middleware trong Next.js chạy ở đâu và dùng để làm gì? Có giới hạn nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Middleware là file `middleware.ts` ở gốc dự án, nó chạy trước khi request đi tới route và chạy trên Edge Runtime, tức là sát người dùng và rất nhanh. Mình dùng nó cho những việc nhẹ ở tầng cổng vào như redirect, rewrite, set header hay cookie, chặn auth sơ bộ, A/B testing hay xử lý i18n; thường mình khai báo thêm `matcher` để nó chỉ chạy đúng những path cần thiết thay vì mọi request. Giới hạn quan trọng nhất là vì chạy trên Edge nên mình không dùng được nhiều Node API như `fs` hay một số DB driver, và vì nó chạy trên mọi request khớp matcher nên phải cực gọn, không làm tác vụ tốn thời gian. Mình cũng luôn nói rõ là không nên coi middleware là nơi authz duy nhất; kiểm tra phân quyền với dữ liệu nhạy cảm vẫn phải làm ở data layer hoặc Server Action, đúng như Next khuyến nghị dùng Data Access Layer. Và về cơ bản chỉ có một file middleware ở gốc thôi."

---

- `middleware.ts` (gốc dự án) chạy **trước khi request tới route**, trên **Edge Runtime**.
- Dùng: rewrite, redirect, set header/cookie, kiểm tra auth, A/B testing, i18n.

```ts
export function middleware(req: NextRequest) {
  const token = req.cookies.get('session')?.value;
  if (!token && req.nextUrl.pathname.startsWith('/dashboard'))
    return NextResponse.redirect(new URL('/login', req.url));
  return NextResponse.next();
}
export const config = { matcher: ['/dashboard/:path*'] }; // tối ưu, chỉ chạy path khớp
```

**Giới hạn (hay bị hỏi):**
- Edge Runtime → **không** dùng nhiều Node API (`fs`, native lib, một số DB driver), không truy cập DB nặng.
- Chạy trên **mọi request khớp matcher** → phải nhẹ/nhanh, không tác vụ tốn thời gian.
- Không nên là nơi authz duy nhất; data-sensitive vẫn kiểm tra ở data layer/Server Action (Next khuyến nghị Data Access Layer).
- Về cơ bản chỉ **một** file middleware ở gốc.

</details>

---

### 7.11. `generateMetadata` hoạt động thế nào để làm SEO động? Khác `metadata` tĩnh ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Trong App Router mình quản lý thẻ `<head>` qua Metadata API thay cho `<Head>` của Pages Router. Khi dữ liệu biết trước lúc build thì mình export một object tĩnh `export const metadata`, còn khi title hay ảnh OG phụ thuộc vào params hoặc dữ liệu fetch về, ví dụ trang chi tiết bài viết, thì mình dùng `generateMetadata`, một hàm async nhận params, fetch dữ liệu rồi trả về object Metadata. Điểm cốt lõi cho SEO là metadata được sinh trên server trước khi gửi HTML đi, nên crawler nhận được `<title>` và OG tags ngay trong HTML ban đầu, đây là lợi thế lớn so với SPA client-side render gửi head rỗng rồi mới fill bằng JS. Một điểm tối ưu mình hay nhắc là Next dedupe các fetch giống nhau, nên nếu `generateMetadata` và `page` cùng fetch một API thì nó chỉ gọi mạng một lần. Bẫy cần nhớ là `generateMetadata` chỉ chạy trong Server Component, không export được từ file có `'use client'`."

---

- App Router quản lý `<head>` qua **Metadata API** (thay `<Head>` của Pages Router).
- **Tĩnh**: `export const metadata = { title, description }` — biết trước lúc build.
- **Động**: `generateMetadata` — phụ thuộc params/dữ liệu.

```tsx
export async function generateMetadata({ params }): Promise<Metadata> {
  const post = await fetch(`https://api/posts/${params.slug}`).then(r => r.json());
  return { title: post.title, openGraph: { images: [post.coverImage] } };
}
```

- **SEO:** metadata sinh **trên server trước khi gửi HTML** → crawler nhận `<title>`/OG ngay trong HTML ban đầu (lợi thế so với SPA CSR head trống).
- **Tối ưu:** Next **dedupe** fetch giống nhau giữa `generateMetadata` và `page` → không gọi mạng hai lần. Có thêm `sitemap.ts`, `robots.ts`, `opengraph-image.tsx`.

**Bẫy:** chỉ hoạt động trong **Server Component** — không export được từ file `'use client'`.

</details>

---

### 7.12. `next/image` và `next/font` giúp tối ưu hiệu năng/Core Web Vitals như thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Cả hai component này đều nhắm thẳng vào Core Web Vitals, đặc biệt là LCP và CLS. `next/image` tự resize ảnh và phục vụ định dạng hiện đại như WebP hay AVIF theo từng thiết bị, lazy load mặc định, và quan trọng nhất là chống layout shift bằng cách giữ chỗ theo `width`/`height` hoặc `fill`; ngoài ra mình dùng `priority` cho ảnh hero để preload phần tử LCP và `placeholder=\"blur\"` cho hiệu ứng mờ lúc tải. `next/font` thì tự động self-host font kể cả Google Fonts nên không phải gọi sang domain ngoài, và nó dùng fallback metrics với `size-adjust` để khớp kích thước font dự phòng, nhờ vậy gần như không gây layout shift do FOUT hay FOIT, đồng thời subset bớt ký tự không dùng. Bẫy thực chiến với `next/image` là quên `width/height` hoặc dùng `fill` mà container không có position thì vẫn bị CLS, ảnh từ domain ngoài chưa khai báo `images.remotePatterns` thì lỗi, và lạm dụng `priority` cho mọi ảnh thì mất luôn tác dụng lazy-load."

---

Cả hai nhắm thẳng vào **Core Web Vitals** (LCP, CLS).

**`next/image`:**
- Resize + định dạng hiện đại (WebP/AVIF) theo thiết bị; lazy loading mặc định.
- **Chống CLS** bằng giữ chỗ theo `width`/`height` (hoặc `fill`).
- `priority` cho ảnh LCP (hero) để preload; `placeholder="blur"` hiệu ứng mờ.

```tsx
<Image src="/hero.jpg" alt="Banner" width={1200} height={600} priority />
```

**`next/font`:**
- **Self-host tự động** (kể cả Google Fonts) → không request domain ngoài.
- **Không layout shift** nhờ fallback metrics (`size-adjust`), giảm CLS do FOUT/FOIT; subset ký tự.

**Bẫy `next/image`:** quên `width/height` (hoặc `fill` + container position) → vẫn CLS; domain ngoài chưa cấu hình `images.remotePatterns` → ảnh lỗi; lạm dụng `priority` cho mọi ảnh → mất tác dụng lazy-load.

</details>

---

### 7.13. Streaming SSR với Suspense hoạt động ra sao trong App Router? Nó cải thiện gì so với SSR truyền thống?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"SSR truyền thống mang tính all-or-nothing: server phải fetch xong toàn bộ dữ liệu của trang rồi mới gửi HTML đi, nên chỉ cần một phần chậm là chặn cả trang, dẫn tới TTFB cao và người dùng nhìn màn hình trắng. Streaming SSR thì chia trang thành nhiều mảnh và gửi HTML xuống dần dần khi từng phần sẵn sàng. Ranh giới để stream chính là `<Suspense>`, và thật ra `loading.tsx` cũng chỉ là một Suspense bao quanh page mà thôi: phần shell như header gửi xuống ngay, còn phần chậm như thống kê thì hiện skeleton trước rồi stream nội dung thật sau. Lợi ích là FCP nhanh hơn, người dùng thấy giao diện sớm, các phần chậm tải độc lập và song song nên tránh waterfall chặn UI, cải thiện perceived performance rõ rệt. Lưu ý thực chiến là mình phải đặt Suspense quanh đúng phần chậm cụ thể, chứ bọc cả trang thì lại quay về all-or-nothing; và vì khi stream thì status 200 đã gửi đi rồi, nên lỗi phải xử lý qua error boundary của segment thay vì đổi status code."

---

- SSR truyền thống là **all-or-nothing**: fetch xong toàn bộ mới gửi HTML → một phần chậm chặn cả trang (TTFB cao, màn hình trắng).
- **Streaming SSR**: chia trang thành mảnh, gửi HTML **dần dần** khi từng phần sẵn sàng. Ranh giới streaming là `<Suspense>` (và `loading.tsx` thực chất là Suspense bao quanh page).

```tsx
<Header />                                  {/* gửi ngay */}
<Suspense fallback={<SkeletonStats />}>
  <SlowStats />                             {/* fetch chậm -> stream sau */}
</Suspense>
```

**Lợi ích:**
- TTFB thấp / FCP nhanh hơn: shell hiển thị ngay với skeleton.
- Tránh waterfall chặn UI: mỗi Suspense tải độc lập, song song.
- Cải thiện perceived performance.

**Lưu ý:** đặt Suspense **quanh phần chậm cụ thể**, không bọc cả trang (lại thành all-or-nothing). Khi stream, status 200 đã gửi → lỗi xử lý qua error boundary segment thay vì đổi status code.

</details>

---

### 7.14. Route Handlers (`route.ts`) là gì? Khi nào dùng thay cho Server Actions?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Route Handlers là cách tạo API endpoint trong App Router, thay cho `pages/api/*` của Pages Router. Mình tạo file `route.ts` và export các hàm đặt tên theo HTTP method như `GET`, `POST`, dùng chuẩn Web `Request`/`Response`, nên nó giống một REST endpoint thực thụ. Khác biệt cốt lõi so với Server Action là Route Handler dành cho client gọi qua HTTP từ bất kỳ đâu, còn Server Action thì mình gọi như một function ngay trong component. Vì vậy mình chọn Route Handler khi cần endpoint cho client ngoài như mobile app, nhận webhook, public API, cron, hay khi cần kiểm soát chi tiết header và stream file; còn mutation gắn liền với form và UI nội bộ thì mình ưu tiên Server Action vì type-safe và ít boilerplate hơn. Một lưu ý phiên bản quan trọng là từ Next 15, `GET` Route Handler không còn cache mặc định nữa, muốn cache thì phải khai báo rõ qua `dynamic = 'force-static'` hoặc revalidate."

---

- Cách tạo **API endpoint** trong App Router (thay `pages/api/*`): file `route.ts`, export hàm theo tên HTTP method (`GET`, `POST`...), dùng Web `Request`/`Response` chuẩn.

```ts
export async function GET(req: Request) {
  const q = new URL(req.url).searchParams.get('q');
  return NextResponse.json(await db.post.findMany({ where: { title: { contains: q ?? '' } } }));
}
export async function POST(req: Request) {
  return NextResponse.json(await db.post.create({ data: await req.json() }), { status: 201 });
}
```

| | Route Handler | Server Action |
|---|---|---|
| Mục đích | API công khai (REST) | Mutation gắn UI/form |
| Gọi | `fetch`/HTTP từ client bất kỳ | Như function từ component |
| Hợp với | Mobile, webhook, third-party, cron | Form, like/toggle nội bộ |

**Dùng Route Handler:** endpoint cho client ngoài, nhận webhook, public API, stream/file với kiểm soát header. **Dùng Server Action:** mutation từ chính UI Next (type-safe, ít boilerplate).

**Lưu ý phiên bản:** từ Next 15, `GET` Route Handler **không còn cache mặc định** — muốn cache phải khai báo (`dynamic = 'force-static'` hoặc revalidate).

</details>

---

### 7.15. Hãy mô tả các tầng caching của Next.js (App Router) và cách invalidate chúng.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"App Router có bốn tầng cache chồng lên nhau và đây là chủ đề rất dễ nhầm. Tầng đầu là Request Memoization, nằm trên server trong phạm vi một request, gộp các `fetch` trùng nhau và tự hết khi request kết thúc. Tầng thứ hai là Data Cache, lưu kết quả fetch lâu dài trên server, mình invalidate bằng `revalidateTag`, `revalidatePath` hoặc theo thời gian. Tầng thứ ba là Full Route Cache, cache HTML và RSC payload của route tĩnh từ lúc build, invalidate qua `revalidatePath` hoặc redeploy. Tầng cuối là Router Cache nằm ở phía client trong trình duyệt, cache RSC payload của các route đã ghé hoặc prefetch, làm mới bằng `router.refresh()` hoặc khi điều hướng. Sai lầm kinh điển là DB đã đổi mà UI vẫn cũ, gần như luôn do Data Cache hoặc Router Cache chưa được invalidate sau mutation, nên quy tắc của mình là cứ ghi xong thì `revalidateTag`/`revalidatePath`, còn tương tác client thuần thì `router.refresh()`. Cũng nên nhớ Next 15 mặc định cache ít hơn để giảm bớt bug dữ liệu cũ vốn rất phổ biến ở Next 13 và 14."

---

App Router có **4 tầng cache** chồng lên nhau (chủ đề khó, hay nhầm):

| Tầng | Nằm ở | Cache gì | Invalidate |
|---|---|---|---|
| **Request Memoization** | Server, trong 1 request | `fetch` trùng nhau | Tự hết sau request |
| **Data Cache** | Server (persistent) | Kết quả `fetch`/dữ liệu | `revalidateTag`, `revalidatePath`, time |
| **Full Route Cache** | Server (build-time) | HTML + RSC payload route tĩnh | `revalidatePath`, redeploy |
| **Router Cache** | **Client** (trình duyệt) | RSC payload route đã ghé/prefetch | `router.refresh()`, điều hướng |

```tsx
async function updateProduct(id: string) {
  'use server';
  await db.product.update(/* ... */);
  revalidateTag('products');   // xóa Data Cache theo tag
  revalidatePath('/products'); // xóa Full Route + refresh Router Cache
}
// Phía client: router.refresh() lấy RSC payload mới, giữ client state
```

**Sai lầm kinh điển:** DB đã đổi nhưng UI cũ → thường do **Data Cache**/**Router Cache** chưa invalidate sau mutation. Quy tắc: sau khi ghi luôn `revalidateTag`/`revalidatePath`; tương tác client thuần dùng `router.refresh()`.

**Lưu ý phiên bản:** Next 15 mặc định "ít cache hơn" (fetch và GET handler không cache, `staleTimes` ngắn hơn) để giảm bug "dữ liệu cũ" của Next 13/14.

</details>

## 8. HTML CSS và Trình duyệt

### 8.1. Phân biệt các giá trị của `position`: `static`, `relative`, `absolute`, `fixed`, `sticky`. `position: sticky` hoạt động dựa trên điều kiện nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Bản chất `position` quyết định một phần tử được định vị theo cái gì. `static` là mặc định, đi theo luồng bình thường và bỏ qua `top/left`. `relative` thì vẫn giữ chỗ trong luồng nhưng cho phép mình dịch nó đi, và quan trọng là nó tạo mốc để con `absolute` bám vào. `absolute` tách hẳn khỏi luồng và định vị theo tổ tiên gần nhất có `position` khác `static`, còn `fixed` thì bám theo viewport nên dính cố định khi cuộn. `sticky` là dạng lai: nó nằm bình thường như `relative` cho tới khi cuộn chạm ngưỡng mình đặt thì dính lại như `fixed`, nhưng chỉ trong phạm vi container cha. Mình hay dùng `sticky` cho header hoặc thanh filter. Cái bẫy mà mình gặp nhiều nhất là `sticky` không chạy, và gần như luôn vì một tổ tiên nào đó có `overflow: hidden/auto/scroll`, hoặc mình quên đặt `top`; nên khi nó im lặng không hoạt động thì mình kiểm tra hai thứ đó trước."

---

- `static`: mặc định, theo luồng, bỏ qua `top/left`.
- `relative`: giữ chỗ trong luồng, dịch so với vị trí gốc; tạo mốc cho con `absolute`.
- `absolute`: tách luồng, định vị theo tổ tiên gần nhất có `position` khác `static`.
- `fixed`: tách luồng, dính theo viewport khi cuộn.
- `sticky`: lai relative/fixed, dính lại khi cuộn chạm ngưỡng, trong phạm vi container cha.

`sticky` cần: (1) có ít nhất một `top/bottom/left/right`; (2) chỉ dính trong cha; (3) cha/tổ tiên **không** được `overflow: hidden/auto/scroll` — bẫy phổ biến nhất (vd wrapper `overflow-x-hidden` chống cuộn ngang mobile làm sticky không chạy).

```tsx
<header style={{ position: 'sticky', top: 0, zIndex: 10, background: '#fff' }}>...</header>
```

</details>

---

### 8.2. Semantic HTML là gì và vì sao nó quan trọng cho accessibility? Cho ví dụ tái cấu trúc một component React từ "div soup" sang semantic.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Semantic HTML nghĩa là mình dùng đúng thẻ mang đúng ý nghĩa, ví dụ `<button>`, `<nav>`, `<main>`, `<article>`, `<label>`, thay vì nhét tất cả vào `<div>` và `<span>`. Lý do quan trọng nhất là accessibility: screen reader dựa vào role ngầm của các thẻ này để cho người dùng nhảy nhanh giữa các landmark, và những thẻ như `<button>` hay `<a>` đã có sẵn focus, tab được, bấm Enter hoặc Space được mà mình không phải viết thêm dòng nào. Ngoài ra còn lợi cho SEO và code dễ đọc hơn. Cái bẫy kinh điển trong React là làm nút bằng `<div onClick>`: trông thì vẫn click được bằng chuột, nhưng không focus được, không phản hồi bàn phím, và screen reader không biết đó là nút. Nên khi review code mình hay bắt đúng chỗ này, và nguyên tắc của mình là cái gì là nút thì dùng `<button>`, cái gì điều hướng thì dùng `<a href>`, đừng tự chế lại hành vi mà trình duyệt cho sẵn miễn phí."

---

- Semantic HTML: dùng thẻ đúng nghĩa (`<header>`, `<nav>`, `<main>`, `<article>`, `<button>`, `<label>`) thay vì `<div>`/`<span>` cho mọi thứ.
- **Accessibility**: screen reader dựa vào implicit ARIA role để điều hướng landmark; tab qua được `<button>`/`<a>` mà không cần code thêm. Lợi thêm: SEO, code dễ đọc, hành vi miễn phí (`<button>` có focus + Enter/Space).
- Bẫy React: `<div onClick>` làm nút — không focus được, không phản hồi Enter/Space, screen reader không báo là nút. Nếu buộc dùng phải thêm `role="button"`, `tabIndex={0}`, `onKeyDown`. Luôn `<label htmlFor>`, `alt` cho ảnh có nghĩa và `alt=""` cho ảnh trang trí.

```tsx
// ❌ <div className="link" onClick={open}>Đọc thêm</div>
// ✅
<article className="card">
  <h2>Bài viết</h2>
  <a href="/post/1">Đọc thêm</a>
</article>
```

</details>

---

### 8.3. Bạn xây dựng responsive như thế nào? So sánh `media query` với `container query`, và giải thích vì sao nên ưu tiên `rem`/`em` cùng tư duy mobile-first.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Cách mình làm responsive là mobile-first: viết style nền cho màn nhỏ trước rồi dùng `min-width` để bổ sung cho màn lớn, như vậy CSS cơ sở gọn và mình ít phải ghi đè ngược. Về đơn vị, mình ưu tiên `rem` cho spacing và typography vì nó tính theo `font-size` của `<html>`, tức là tôn trọng cài đặt cỡ chữ của người dùng, ai chỉnh chữ to lên thì cả layout giãn theo nên tốt cho accessibility; `em` thì tính theo phần tử cha nên hợp khi mình muốn một thứ co giãn theo cỡ chữ cục bộ. Điểm mình hay nhấn trong phỏng vấn là khác biệt giữa media query và container query: media query đo theo viewport nên hợp layout toàn trang, còn container query đo theo container cha nên cùng một component `<Card>` đặt ở sidebar hẹp hay vùng content rộng sẽ tự đổi bố cục mà không cần biết viewport bao nhiêu. Vì thế với design system hay component tái dùng mình nghiêng về container query, còn layout tổng thể vẫn dùng media query."

---

- **Mobile-first**: style nền cho màn nhỏ trước, rồi `min-width` bổ sung cho màn lớn → cơ sở gọn, tránh ghi đè ngược.
- **Đơn vị**: `rem` theo `font-size` của `<html>` (nhất quán, tôn trọng cài đặt cỡ chữ người dùng); `em` theo phần tử/cha (dễ dồn khi lồng). Dùng `px` cho viền mảnh, `rem` cho spacing/typography, `em` khi muốn co theo cỡ chữ cục bộ.
- **Media query**: đo theo viewport, hợp layout toàn trang. **Container query**: đo theo container cha, hợp component tái dùng (cùng `<Card>` đặt sidebar hẹp hay content rộng tự đổi layout). Cú pháp: `container-type` + `@container`.
- Tailwind: hỗ trợ container query qua plugin `@container`; breakpoint `sm/md/lg` mặc định dựa `min-width` viewport.

```css
@media (min-width: 768px) { .grid { grid-template-columns: repeat(2, 1fr); } }

.card-wrapper { container-type: inline-size; }
@container (min-width: 400px) { .card { display: grid; grid-template-columns: 120px 1fr; } }
```

</details>

---

### 8.4. Phân biệt Reflow (Layout) và Repaint. Loại nào tốn kém hơn, và bạn làm gì trong React để giảm thiểu reflow khi animation?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Reflow, hay layout, là khi trình duyệt phải tính lại hình học của trang, tức là vị trí và kích thước các phần tử; còn repaint chỉ là vẽ lại pixel khi màu sắc hay bóng đổi mà layout không đổi. Reflow tốn kém hơn nhiều vì nó thường kéo theo cả repaint và có thể lan ra cả cây phần tử, trong khi repaint thì cục bộ hơn. Vì vậy khi animation, nguyên tắc của mình là chỉ animate `transform` và `opacity`, vì hai cái này được xử lý ở tầng composite trên GPU, không gây reflow cũng không repaint; tránh animate `left`, `top`, `width`, `height`. Trong React còn một bẫy nữa là layout thrashing: nếu trong một vòng lặp mình vừa ghi style vừa đọc thuộc tính layout như `offsetWidth` thì trình duyệt buộc phải reflow đồng bộ liên tục, rất giật. Cách mình xử lý là gom các thao tác đọc và ghi DOM lại, đo một lần trong `useLayoutEffect`, hoặc đơn giản là dùng thư viện như Framer Motion vốn đã animate qua transform/opacity và đồng bộ với requestAnimationFrame."

---

- **Reflow (Layout)**: tính lại hình học (vị trí, kích thước), thường kéo theo cả cây. Kích hoạt bởi đổi `width/height/top/margin`, thêm/xóa DOM, đọc `offsetWidth`...
- **Repaint**: vẽ lại pixel không đổi layout — `color`, `background`, `visibility`, `box-shadow`.
- **Reflow tốn kém hơn** vì luôn kéo theo repaint và có thể lan ra cả cây.
- Animation chỉ nên dùng **`transform` và `opacity`** — xử lý ở composite trên GPU, không reflow/repaint. Tránh `left/top/width/height`.
- Bẫy **layout thrashing**: đọc thuộc tính layout ngay sau khi ghi style trong vòng lặp → reflow đồng bộ liên tục. Gom thay đổi DOM, đo trong `useLayoutEffect` một lần, hoặc dùng Framer Motion (animate qua transform/opacity + rAF).

```css
.box { transition: transform 0.3s; } /* ✅ thay vì transition: left */
```

</details>

---

### 8.5. Stacking context là gì? Vì sao đôi khi `z-index: 9999` vẫn không đưa được phần tử lên trên, và điều này liên quan gì tới modal/dropdown trong React?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Stacking context là một ngữ cảnh xếp chồng độc lập trên trục z, và mấu chốt là `z-index` chỉ so sánh giữa các phần tử trong cùng một stacking context với nhau. Đó là lý do đặt `z-index: 9999` đôi khi vẫn vô dụng: nếu phần tử đó nằm trong một context cha mà bản thân cha lại có thứ hạng thấp so với một context khác, thì cả cụm con dù `z-index` to đến đâu cũng bị đè. Điều đáng nhớ là rất nhiều thứ tạo ra stacking context mới một cách âm thầm, không chỉ `position` cộng `z-index`, mà cả `opacity` nhỏ hơn 1, `transform`, `filter`, `will-change`. Chính vì thế dropdown, tooltip hay modal hay bị một cha có `transform` cho animation hoặc `overflow: hidden` cắt hoặc che mất. Cách mình giải quyết gọn nhất là dùng Portal, render thẳng phần tử lên `document.body` để nó thoát khỏi stacking context và overflow của cha. Trong Next.js App Router thì `createPortal` là client-side nên mình nhớ thêm `'use client'` và kiểm tra mounted để tránh lệch hydration, hoặc đơn giản dùng Radix UI vì nó đã xử lý portal sẵn."

---

- Stacking context: ngữ cảnh xếp chồng độc lập trên trục z. `z-index` chỉ so sánh **trong cùng một stacking context** — phần tử `z-index: 9999` nằm trong context cha thứ hạng thấp vẫn bị context khác cao hơn che.
- Thứ tạo stacking context mới: `position` khác static + `z-index` là số; `opacity < 1`; `transform`, `filter`, `will-change`, `backdrop-filter`; `position: fixed/sticky`; con Flex/Grid có `z-index`.
- Đây là lý do dropdown/tooltip/modal hay bị cha (có `transform`, `overflow`, `opacity`) cắt/che.
- Giải pháp: **Portal** render lên thẳng `document.body`, thoát stacking context + overflow của cha.
- Next.js (App Router/RSC): `createPortal` là client-side → cần `'use client'` + kiểm tra mounted tránh lệch hydration. Radix UI xử lý sẵn portal.

```tsx
import { createPortal } from 'react-dom';
function Modal({ children }) {
  return createPortal(<div className="overlay">{children}</div>, document.body);
}
```

</details>

---

### 8.6. Bạn triển khai theming và dark mode bằng CSS Variables như thế nào? Vì sao biến CSS hợp với React/Next.js hơn là dùng nhiều class điều kiện, và làm sao tránh nhấp nháy (FOUC) khi tải trang?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Cách mình làm theming là dựa trên CSS Variables: khai báo các biến như `--bg`, `--text` ở `:root` cho theme sáng, rồi ghi đè cùng các biến đó trong `[data-theme='dark']`, và các component chỉ việc đọc `var(--bg)`. Như vậy mình chỉ cần đổi một attribute trên thẻ `<html>` là cả UI đổi màu, mà không component nào phải re-render vì việc đổi màu xảy ra hoàn toàn ở tầng CSS. Cái này hợp với React/Next.js hơn là rải `className={isDark ? ... : ...}` khắp nơi, vì với class điều kiện mình phải nhớ xử lý ở từng chỗ và rất dễ sót; còn với biến CSS thì chỉ có một điểm điều khiển duy nhất và theme có thể đổi runtime. Tailwind cũng đi theo hướng này qua `darkMode: 'class'`. Vấn đề lớn nhất là FOUC, tức nháy sáng rồi mới sang tối, xảy ra khi mình set theme trong `useEffect` vì lúc đó trang đã paint xong rồi. Cách đúng là chèn một script đồng bộ trong `<head>` đọc `localStorage` và đặt attribute trước khi trang paint; thực tế mình hay dùng luôn `next-themes` vì nó đã làm sẵn script chống nháy, hỗ trợ chế độ `system` và đặt `suppressHydrationWarning` giúp mình."

---

- CSS Variables: khai báo `--ten`, đọc `var(--ten)`; kế thừa và ghi đè theo scope (`:root` vs `[data-theme="dark"]`). Đổi giá trị một nơi là toàn UI đổi theo, không render lại component.
- Hơn class điều kiện: không rải `className={isDark?...}` khắp nơi, chỉ một điểm điều khiển (attribute trên `<html>`), runtime-themeable. Tailwind dùng cách này qua `darkMode: 'class'`.
- **Tránh FOUC** (vấn đề với SSR): set theme trong `useEffect` sẽ nháy light→dark. Giải pháp: script đồng bộ trong `<head>` đọc `localStorage` và đặt attribute **trước khi** paint. Thực tế dùng `next-themes` (tự chèn script chống nháy, hỗ trợ `system`, đặt `suppressHydrationWarning`).

```css
:root { --bg: #fff; --text: #111; }
[data-theme='dark'] { --bg: #0a0a0a; --text: #f5f5f5; }
body { background: var(--bg); color: var(--text); }
```

</details>

---

### 8.7. (Tình huống) Một đồng nghiệp báo rằng trang bị "cuộn ngang lạ" trên mobile và một tooltip bị che mất. Hãy kể về một lần bạn debug vấn đề CSS/layout khó và cách bạn tiếp cận.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

**📌 Câu trả lời mẫu:**

"Mình từng gặp đúng kiểu tình huống này trên một dự án Next.js: sau khi release landing page, QA báo mobile bị cuộn ngang lạ và một tooltip trong bảng dữ liệu bị cắt mất nửa. Mình tách thành hai vấn đề và xử lý từng cái theo hướng tìm gốc. Với cuộn ngang, mình dùng mẹo khoanh vùng bằng cách thêm tạm `* { outline: 1px solid red }` để nhìn ra phần tử nào tràn ra ngoài, và phát hiện một block đặt `width: 100vw` mà không trừ thanh cuộn và padding của cha; mình đổi sang `100%`, rà lại các đơn vị viewport và xác nhận `box-sizing: border-box` đang áp toàn cục. Với tooltip thì điểm mấu chốt là `z-index` cao bao nhiêu cũng vô dụng, vì wrapper có `transform` cho animation đã tạo ra một stacking context riêng nhốt tooltip lại; mình giải quyết bằng cách render tooltip qua `createPortal` lên thẳng `document.body`. Kết quả là hết cuộn ngang và tooltip hiển thị đủ. Sau đó mình rút ra checklist chia sẻ cho nhóm: cẩn thận với `vw` trên mobile, khi `z-index` không nghe lời thì kiểm tra ngay stacking context, và ưu tiên Portal cho mọi overlay."

---

- **Situation**: Dự án Next.js, sau release landing page, QA báo mobile bị cuộn ngang lạ và tooltip trong bảng dữ liệu bị cắt mất nửa.
- **Task**: Tìm nguyên nhân gốc cả hai và sửa mà không phá layout các trang khác.
- **Action**: Cuộn ngang — dùng `* { outline: 1px solid red; }` khoanh vùng, phát hiện phần tử `width: 100vw` (không trừ thanh cuộn/padding cha) → đổi `100%`, rà các đơn vị viewport, xác nhận `box-sizing: border-box` toàn cục. Tooltip — `z-index` cao vô dụng vì wrapper có `transform` (animation) tạo stacking context; chuyển tooltip sang `createPortal` lên `document.body`.
- **Result**: Hết cuộn ngang, tooltip hiển thị đủ. Rút checklist: cẩn trọng `vw`, kiểm tra stacking context khi `z-index` "không nghe lời", ưu tiên Portal cho overlay; chia sẻ lại cho cả nhóm.

</details>

---

## 9. Web Networking và Bảo mật Frontend

### 9.1. Khi bạn gõ một URL vào trình duyệt và nhấn Enter, điều gì xảy ra? Là frontend developer, bạn quan tâm tới những bước nào nhất?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Tóm tắt thì khi gõ URL và Enter, trình duyệt phân giải DNS để tìm IP, bắt tay TCP rồi TLS nếu là HTTPS, gửi HTTP request kèm cookie, server trả về HTML, sau đó trình duyệt parse HTML dựng DOM, parse CSS dựng CSSOM, ghép thành render tree rồi layout, paint và composite ra màn hình. Với một SPA hay app Next.js thì sau khi HTML hiển thị, bundle JS mới tải về và chạy hydration để gắn event listener, lúc đó trang mới thực sự tương tác được. Là frontend developer thì những bước mình quan tâm nhất là phía sau khi nhận HTML: bundle size và code-splitting vì nó ảnh hưởng trực tiếp tới FCP, LCP và thời gian tới lúc tương tác được; chi phí hydration và rủi ro hydration mismatch; và việc kết nối sớm tới API hay CDN bằng `preconnect`, `dns-prefetch`. Một bẫy mình hay được hỏi là nhiều người nói JS luôn chặn render, thực ra `<script defer>` và `type=\"module\"` tải song song và chỉ chạy sau khi parse xong, chỉ có script đồng bộ mới chặn parser."

---

- **Luồng chính**: phân giải DNS → TCP + TLS handshake (nếu HTTPS) → gửi HTTP request (kèm cookie) → server trả HTML → trình duyệt parse dựng DOM/CSSOM → render tree → layout → paint → composite.
- **Với SPA/Next.js**: sau khi HTML hiển thị, JS bundle tải về và **hydration** gắn event listener để trang tương tác được.
- **FE quan tâm nhất**: bundle size/code-splitting (TTFB/FCP/LCP), tránh reflow/layout thrashing, chi phí và "hydration mismatch", tối ưu kết nối sớm tới API/CDN.

```html
<link rel="preconnect" href="https://api.example.com" />
<link rel="dns-prefetch" href="https://cdn.example.com" />
```

**Bẫy**: nói JS luôn chặn render. Thực tế `<script defer>` và `type="module"` tải song song, chạy sau khi parse xong; chỉ `<script>` đồng bộ mới chặn parser.

</details>

---

### 9.2. Phân biệt các HTTP method phổ biến và ý nghĩa của tính idempotent. Hãy giải thích vài status code mà frontend cần xử lý khác nhau.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Các method chính là GET để đọc, POST để tạo, PUT để thay toàn bộ một resource, PATCH để sửa một phần, và DELETE để xóa. Khái niệm idempotent nghĩa là gọi nhiều lần cũng cho ra cùng kết quả như gọi một lần; GET, PUT, DELETE là idempotent còn POST thì không. Cái này rất quan trọng khi retry: nếu mạng chập chờn thì retry một GET hay PUT là an toàn, nhưng retry POST có thể tạo bản ghi trùng, nên với POST quan trọng mình thường dùng `idempotency-key` để server nhận diện và chống trùng. Về status code, ở phía frontend mình xử lý khác nhau khá rõ: `401` là chưa hoặc hết xác thực nên đẩy về trang login, `403` là đã đăng nhập nhưng không đủ quyền nên báo cấm chứ không bắt login lại, `422` là lỗi validate nên mình hiển thị lỗi theo từng field, `429` là bị rate limit nên mình retry với backoff, còn `5xx` là lỗi server thì có thể retry. Hai chỗ mình hay thấy người nhầm là lẫn `401` với `403`, và lẫn `422` đúng cú pháp nhưng sai nghiệp vụ với `400` sai định dạng request."

---

- **Method**: GET (đọc, idempotent + safe), POST (tạo, không idempotent), PUT (thay toàn bộ, idempotent), PATCH (sửa một phần, thường không idempotent), DELETE (xóa, idempotent).
- **Idempotent** = gọi nhiều lần cho cùng kết quả như một lần → quan trọng khi **retry**: GET/PUT/DELETE an toàn, retry POST có thể tạo trùng (cần `idempotency-key`).
- **Status code FE xử lý khác nhau**: `401` (chưa/hết xác thực → login), `403` (không đủ quyền), `404`, `422` (lỗi validate), `429` (rate limit → backoff), `5xx` (lỗi server → có thể retry).
- **Nhóm**: `2xx` ok, `3xx` redirect, `4xx` lỗi client (đừng retry trừ 408/429), `5xx` lỗi server (retry với backoff).

```ts
switch (res.status) {
  case 401: redirectToLogin(); break;
  case 403: showForbidden(); break;
  case 422: return showValidationErrors(await res.json());
  case 429: return retryWithBackoff();
}
```

**Bẫy**: nhầm `401` (chưa xác thực) với `403` (đã xác thực, thiếu quyền); và `422` (đúng cú pháp, sai nghiệp vụ) với `400` (sai định dạng request).

</details>

---

### 9.3. CORS là gì? Vì sao bạn lại gặp lỗi CORS, và frontend có thể "tự sửa" nó không?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"CORS là cơ chế dùng HTTP header để server cho phép trình duyệt đọc response từ một origin khác, trong đó origin là bộ ba scheme cộng host cộng port; nó tồn tại vì mặc định trình duyệt áp Same-Origin Policy. Mình gặp lỗi CORS khi app chạy ở `localhost:3000` gọi sang một API khác origin mà server không trả về header `Access-Control-Allow-Origin` phù hợp. Điểm quan trọng mà nhiều người hiểu sai là: request thật ra vẫn được gửi đi và server vẫn xử lý, chỉ là trình duyệt chặn không cho JS đọc response. Vì vậy CORS là cấu hình ở phía server, frontend không thể tự sửa bằng cách đổi header request, làm vậy vô tác dụng. Cái mà frontend làm được một cách hợp lệ là dùng proxy hoặc rewrites, ví dụ rewrites trong Next.js, để biến request thành same-origin rồi server proxy gọi tiếp tới API. Một bẫy nữa là khi gửi kèm credentials như cookie thì `Access-Control-Allow-Origin: *` không hoạt động, server bắt buộc phải trả về origin cụ thể."

---

- **CORS** là cơ chế dùng HTTP header để **server** cho phép trình duyệt đọc response từ origin khác (origin = scheme + host + port), do Same-Origin Policy.
- Gặp lỗi khi FE (`localhost:3000`) gọi API khác origin mà server không trả `Access-Control-Allow-Origin` phù hợp: request vẫn được gửi và xử lý, nhưng trình duyệt **chặn JS đọc response**.
- **Điểm mấu chốt**: CORS là cấu hình ở **server**, FE không tự sửa bằng code JS. Đổi header request từ FE vô tác dụng.
- **Workaround hợp lệ phía FE**: dùng proxy/rewrites (Next.js) để request thành same-origin.

```js
// next.config.js
async rewrites() {
  return [{ source: '/api/:path*', destination: 'https://api.example.com/:path*' }];
}
```

**Bẫy**: `Access-Control-Allow-Origin: *` **không hoạt động** khi gửi credentials (cookie) — phải trả origin cụ thể.

</details>

---

### 9.4. Preflight request là gì? Khi nào trình duyệt gửi nó, và làm sao để tránh preflight không cần thiết?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Preflight là một request `OPTIONS` mà trình duyệt tự gửi đi trước request thật, để hỏi server xem có cho phép method và header mà mình định dùng không; server trả lại các header `Access-Control-Allow-*` để xác nhận. Trình duyệt chỉ gửi preflight khi request không phải dạng simple. Simple nghĩa là method là GET, HEAD hoặc POST, chỉ dùng các header được phép sẵn, và `Content-Type` thuộc nhóm `form-urlencoded`, `multipart/form-data` hoặc `text/plain`. Trong thực tế thì hầu hết API hiện đại đều bị preflight, vì chỉ cần gửi `Content-Type: application/json` hoặc kèm header `Authorization` tùy chỉnh là đã thoát khỏi simple rồi; luồng khi đó là `OPTIONS` đi trước, server trả `204` cho phép, rồi request thật mới đi. Để tránh preflight lặp lại không cần thiết, server có thể đặt `Access-Control-Max-Age` để trình duyệt cache kết quả preflight trong một khoảng thời gian. Bẫy mình hay nhắc là nhìn thấy `OPTIONS` trong tab Network rồi tưởng API bị gọi hai lần hay bị lỗi, thực ra đó là bình thường; nhưng nếu preflight thất bại thì request thật sẽ không bao giờ được gửi."

---

- **Preflight** là request `OPTIONS` trình duyệt **tự gửi trước** request thật để hỏi server có cho phép method/header không; server trả các header `Access-Control-Allow-*`.
- Gửi khi request **không phải "simple"**. Simple = method `GET`/`HEAD`/`POST`, chỉ header cho phép sẵn, và `Content-Type` là `form-urlencoded`/`multipart/form-data`/`text/plain`.
- Vì vậy `Content-Type: application/json` hoặc header `Authorization` tùy chỉnh → **kích hoạt preflight**. Luồng: `OPTIONS` → server `204` cho phép → request thật.
- **Tối ưu**: server đặt `Access-Control-Max-Age` để cache kết quả preflight, tránh lặp `OPTIONS`.

```http
Access-Control-Max-Age: 86400
```

**Bẫy**: thấy `OPTIONS` trong Network tab và tưởng API gọi 2 lần hoặc "lỗi" — đó là bình thường. Nếu preflight thất bại, request thật không bao giờ được gửi.

</details>

---

### 9.5. Hãy giải thích cơ chế caching HTTP qua `Cache-Control`, `ETag` và `stale-while-revalidate`. Chúng liên hệ thế nào với TanStack Query?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Mình hiểu caching HTTP qua ba mảnh ghép. Cache-Control quyết định trình duyệt được dùng lại response trong bao lâu mà không cần hỏi server: max-age là dùng thẳng, no-cache vẫn lưu nhưng phải revalidate trước khi dùng, còn no-store là cấm lưu hoàn toàn. ETag là một chuỗi vân tay của nội dung; lần sau client gửi kèm If-None-Match, nếu nội dung chưa đổi server trả 304 với body rỗng nên tiết kiệm băng thông mà vẫn xác nhận dữ liệu còn đúng. stale-while-revalidate là tư duy mình thích nhất: hiển thị ngay bản cũ cho người dùng thấy tức thì, đồng thời fetch bản mới ở background rồi cập nhật. Điểm hay là TanStack Query bê nguyên triết lý này lên tầng JS — staleTime gần như max-age, gcTime gần như thời gian giữ trong bộ nhớ, và data stale vẫn hiện ngay rồi refetch ngầm. Bẫy hay gặp là nhầm no-cache với no-store, và staleTime mặc định là 0 nên mọi data stale ngay khiến app refetch liên tục mỗi lần mount lại."

---

- **`Cache-Control`**: `max-age` (dùng lại không hỏi), `no-cache` (phải revalidate trước khi dùng), `no-store` (cấm cache), `private` (chỉ trình duyệt, không CDN).
- **`ETag`** là "vân tay" nội dung; lần sau gửi kèm `If-None-Match`, chưa đổi thì server trả `304 Not Modified` (body rỗng → tiết kiệm băng thông).
- **`stale-while-revalidate`**: dùng ngay bản cũ để hiển thị tức thì, đồng thời fetch bản mới ở background.
- **TanStack Query**: triết lý y hệt nhưng ở tầng JS — `staleTime` ≈ `max-age`, `gcTime` (cũ là `cacheTime`) ≈ thời gian giữ trong bộ nhớ; data stale vẫn hiển thị ngay rồi refetch ngầm.

```tsx
useQuery({ queryKey: ['todos'], queryFn: fetchTodos,
  staleTime: 60_000, gcTime: 300_000 });
```

**Bẫy**: nhầm `no-cache` (vẫn cache nhưng phải revalidate) với `no-store` (cấm hoàn toàn). `staleTime` mặc định của TanStack Query là `0` → mọi data stale ngay, gây refetch liên tục khi mount lại.

</details>

---

### 9.6. So sánh cookie, localStorage và sessionStorage. Khi nào dùng cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Ba cơ chế lưu trữ này khác nhau ở ba điểm chính: dung lượng, có tự gửi lên server không, và vòng đời. Cookie chỉ khoảng 4KB nhưng đặc biệt ở chỗ nó tự đính kèm vào mọi request tới domain và có thể đặt httpOnly để JS không đọc được, nên hợp với token/session. localStorage thì lớn hơn nhiều (5-10MB), JS đọc được, không tự gửi lên server, sống vĩnh viễn tới khi xóa và chia sẻ giữa các tab — mình dùng cho thứ không nhạy cảm như theme, ngôn ngữ, hay draft. sessionStorage giống localStorage nhưng riêng từng tab và mất khi đóng tab, hợp state tạm như form wizard nhiều bước. Quy tắc của mình: cái gì nhạy cảm và cần gửi server thì dùng cookie httpOnly, cái gì giữ lâu mà không nhạy cảm thì localStorage, cái gì chỉ sống trong một phiên thì sessionStorage. Một bẫy quan trọng khi làm Next.js là localStorage và sessionStorage chỉ tồn tại trên trình duyệt, nên truy cập lúc render server sẽ lỗi 'localStorage is not defined' — mình luôn đọc chúng trong useEffect hoặc check typeof window trước."

---

- **Cookie**: ~4KB, tự gửi kèm mọi request tới domain, có thể `httpOnly` (JS không đọc), sống theo `Expires`/`Max-Age`.
- **localStorage**: ~5-10MB, không tự gửi server, JS đọc được, sống vĩnh viễn tới khi xóa, chia sẻ giữa tab.
- **sessionStorage**: như localStorage nhưng riêng từng tab và hết khi đóng tab.
- **Khi nào dùng**: cookie `httpOnly` cho token/session (chống XSS, tự gửi kèm); localStorage cho dữ liệu không nhạy cảm giữ lâu (theme, ngôn ngữ, draft); sessionStorage cho state tạm trong một phiên/tab (form wizard).

```ts
localStorage.setItem('theme', 'dark');
sessionStorage.setItem('wizardStep', '2');
```

**Bẫy SSR (Next.js)**: `localStorage`/`sessionStorage` chỉ có trên trình duyệt — truy cập khi render server gây `localStorage is not defined`. Đọc trong `useEffect` hoặc kiểm tra `typeof window !== 'undefined'`.

</details>

---

### 9.7. Nên lưu JWT ở đâu cho an toàn: httpOnly cookie hay localStorage? Hãy phân tích đánh đổi.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Mình nghĩ câu này không nên chọn cứng một bên, mà phải phân tích đánh đổi giữa rủi ro XSS và CSRF. Nếu để JWT trong localStorage thì tiện cho client đọc và không lo CSRF vì nó không tự gửi, nhưng bất kỳ script độc hay lỗ hổng XSS nào cũng đọc được token ngay lập tức. Ngược lại, httpOnly cookie thì JS hoàn toàn không đọc được nên an toàn trước XSS, đổi lại nó tự gửi kèm request nên có rủi ro CSRF, mình giảm bằng SameSite cộng CSRF token. Vì XSS thường nguy hiểm hơn, mình thiên về httpOnly cookie kèm Secure và SameSite. Mô hình thực chiến mình hay dùng là access token ngắn hạn giữ trong memory (mất khi reload nhưng phơi nhiễm tối thiểu) và refresh token đặt trong httpOnly cookie để gọi endpoint /refresh lấy token mới. Câu trả lời mình tránh là 'localStorage an toàn vì JS của tôi tự quản lý' — đó là hiểu sai, vì điểm yếu nằm ở chính khả năng JS đọc được nó."

---

- Câu trả lời tốt là **phân tích đánh đổi XSS vs CSRF**, không chọn cứng một bên.
- **localStorage**: bị JS độc (XSS) đọc dễ dàng, nhưng không tự gửi nên miễn nhiễm CSRF; khó dùng với SSR.
- **httpOnly cookie**: JS không đọc được (an toàn trước XSS), nhưng tự gửi → có rủi ro CSRF (giảm bằng `SameSite` + CSRF token); hợp SSR.
- **Khuyến nghị**: dùng `httpOnly` + `Secure` + `SameSite` cookie (XSS nguy hiểm hơn). Mô hình thực tế: **access token ngắn hạn giữ trong memory** + **refresh token trong httpOnly cookie** gọi `/refresh`.

```http
Set-Cookie: token=eyJ...; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=900
```

**Bẫy**: nói "localStorage an toàn vì JS của tôi tự quản lý" — sai, bất kỳ script bên thứ ba/lỗ hổng XSS nào cũng đọc được. Dùng cookie phải nhớ `Secure` + `SameSite`.

</details>

---

### 9.8. So sánh WebSocket, Server-Sent Events (SSE) và polling. Khi nào chọn cái nào trong một ứng dụng React?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Mình so ba cái theo hướng truyền dữ liệu và độ phức tạp. Polling là client tự hỏi server theo chu kỳ qua HTTP lặp lại, đơn giản nhất, hợp dữ liệu đổi chậm — trong React mình chỉ cần đặt refetchInterval của TanStack Query. SSE là một chiều từ server xuống client, chạy trên HTTP thường và tự reconnect nhờ EventSource, rất hợp với notification, stream giá, log, hay stream token của AI. WebSocket là hai chiều full-duplex, độ trễ thấp nhưng phải tự code reconnect, hợp với chat, game, hay collaborative editing. Triết lý chọn của mình là bắt đầu từ cái đơn giản nhất đủ dùng: cần realtime một chiều thì SSE, cần hai chiều độ trễ thấp thì WebSocket, còn đổi không thường xuyên thì cứ polling cho nhẹ. Bẫy lớn nhất trong React là quên cleanup — phải gọi es.close() hoặc ws.close() trong useEffect, nếu không mỗi lần re-render lại mở thêm kết nối gây trùng và rò rỉ."

---

- **Polling**: client hỏi theo chu kỳ qua HTTP lặp lại, đơn giản nhất, hợp dữ liệu đổi chậm (TanStack Query `refetchInterval`).
- **SSE**: một chiều server → client, chạy trên HTTP, **tự reconnect** (`EventSource`); hợp notification, stream giá/log, stream token AI.
- **WebSocket**: hai chiều full-duplex, độ trễ thấp, phải tự code reconnect; hợp chat, game, collaborative editing.
- **Cách chọn**: bắt đầu từ đơn giản nhất đủ dùng → realtime một chiều dùng SSE, hai chiều độ trễ thấp dùng WebSocket, đổi không thường xuyên dùng polling.

```tsx
const es = new EventSource('/api/notifications');
es.onmessage = (e) => addNotification(JSON.parse(e.data));
return () => es.close();
```

**Bẫy**: quên cleanup (`es.close()`/`ws.close()`) trong `useEffect` → rò rỉ/trùng kết nối khi re-render. SSE giới hạn ~6 kết nối/domain trên HTTP/1.1 (HTTP/2 không bị).

</details>

---

### 9.9. API trả về `429 Too Many Requests`. Bạn xử lý rate limit ở frontend như thế nào? Giải thích exponential backoff.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Khi gặp 429 thì cốt lõi là đừng dập server liên tục mà phải lùi lại có chiến lược. Exponential backoff nghĩa là mỗi lần retry mình tăng thời gian chờ theo cấp số nhân — 1s, 2s, 4s, 8s — để giảm áp lực lên server đang quá tải. Mình luôn thêm jitter, tức ngẫu nhiên hóa khoảng chờ một chút, để tránh tình huống nhiều client cùng retry đúng một thời điểm gây 'thundering herd'. Nếu response có header Retry-After thì phải tôn trọng con số đó thay vì tự tính, vì server đang nói rõ khi nào được quay lại. Quan trọng không kém là phòng từ đầu: debounce/throttle input, dedupe request trùng (TanStack Query tự dedupe theo queryKey), và không gọi API trong vòng lặp render. Bẫy mình tránh là retry vô hạn hoặc retry cả lỗi 4xx không phục hồi được như 400 hay 404 — vô nghĩa và chỉ làm nặng thêm; mình chỉ retry 429 và 5xx với số lần có giới hạn."

---

- **Exponential backoff**: khi thất bại do rate limit, tăng dần thời gian chờ theo cấp số nhân (1s, 2s, 4s, 8s...) để giảm áp lực server.
- Thêm **jitter** (ngẫu nhiên hóa) để tránh nhiều client retry đồng loạt ("thundering herd").
- Nếu response có header **`Retry-After`**, phải tôn trọng nó thay vì tự tính.
- **Phòng từ đầu**: debounce/throttle input, dedupe request trùng (TanStack Query tự dedupe theo `queryKey`), tránh gọi API trong vòng lặp render.

```tsx
useQuery({ queryKey: ['data'], queryFn: fetchData,
  retry: (count, err) => count < 3 && err.status !== 404,
  retryDelay: (attempt) => Math.min(1000 * 2 ** attempt, 30_000) });
```

**Bẫy**: retry vô hạn hoặc retry cả lỗi `4xx` không phục hồi được (`400`, `404`) — vô nghĩa, làm nặng thêm. Chỉ retry `429` và `5xx`, và giới hạn số lần retry.

</details>

---

### 9.10. XSS là gì? React bảo vệ bạn khỏi XSS như thế nào, và khi nào React KHÔNG bảo vệ bạn?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"XSS là khi kẻ tấn công chèn được JS độc chạy ngay trong trình duyệt nạn nhân, từ đó đánh cắp cookie, token hay mạo danh người dùng. React bảo vệ mình khá tốt theo mặc định: mọi giá trị nhúng qua dấu ngoặc nhọn trong JSX đều được escape tự động, nên một chuỗi chứa thẻ script sẽ hiển thị thành text chứ không thực thi. Nhưng React không phải tấm khiên tuyệt đối — nó hết bảo vệ khi mình dùng dangerouslySetInnerHTML để render HTML thô, khi nhét chuỗi javascript: vào href/src, hoặc khi thao tác DOM trực tiếp như gán innerHTML qua ref. Trong những trường hợp đó mình phải sanitize đầu vào, ví dụ chạy HTML của người dùng qua DOMPurify trước khi render, và validate scheme URL phải là http/https. Tầng phòng thủ cuối mình thích là Content Security Policy để giới hạn nguồn script được phép chạy. Bẫy hay gặp là tâm lý 'dùng React là miễn nhiễm XSS' — thực ra React chỉ an toàn cho nội dung đi qua ngoặc nhọn thôi."

---

- **XSS**: kẻ tấn công chèn JS độc chạy trong trình duyệt nạn nhân để đánh cắp cookie/token, mạo danh.
- **React bảo vệ mặc định**: mọi giá trị nhúng qua `{}` trong JSX được **escape tự động** → chuỗi chứa thẻ HTML hiển thị dưới dạng text, không thực thi.
- **React KHÔNG bảo vệ khi**: dùng `dangerouslySetInnerHTML` (render HTML thô), inject vào `href`/`src` với `javascript:`, hoặc thao tác DOM trực tiếp (`ref.current.innerHTML = ...`).
- **Phòng thủ tầng cuối**: Content Security Policy (CSP) hạn chế nguồn script được chạy.

```tsx
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userHtml) }} />
```

**Bẫy**: nghĩ "dùng React là miễn nhiễm XSS" — sai, React chỉ an toàn cho nội dung qua `{}`. Luôn sanitize HTML do người dùng cung cấp và validate scheme URL là http/https.

</details>

---

### 9.11. CSRF là gì và khác XSS thế nào? Là frontend developer, bạn góp phần phòng chống CSRF ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"CSRF là lừa trình duyệt của nạn nhân gửi một request tới site mà họ đang đăng nhập mà họ không hề hay biết, lợi dụng đúng đặc điểm là cookie tự động gửi kèm — ví dụ một form ẩn tự submit lệnh chuyển tiền tới bank.com. Điểm khác cốt lõi với XSS là: XSS chạy mã JS độc ngay trong trang của mình do lỗ hổng escape/sanitize, còn CSRF không cần chạy JS trên site của mình, nó chỉ lừa gửi một request trông hợp lệ nhờ cơ chế cookie tự gửi. Là frontend, cách góp phần phòng chống mạnh nhất là dùng SameSite cookie ở mức Lax hoặc Strict, vì nó chặn cookie gửi kèm khi request đến từ site khác. Thêm một lớp nữa là CSRF token gắn vào header cho mọi request đổi state — site độc không đọc được token này vì Same-Origin Policy. Một lưu ý mình nhớ là đừng tin SameSite=Lax chặn được tất cả vì vài request GET vẫn lọt, nên thao tác đổi state thì không bao giờ làm qua GET."

---

- **CSRF**: lừa trình duyệt nạn nhân **gửi request tới site đã đăng nhập** mà họ không hay, lợi dụng việc **cookie tự gửi kèm** (ví dụ: form ẩn tự submit `POST bank.com/transfer`).
- **Khác XSS**: XSS chạy mã JS độc trong trang (lỗ hổng escape/sanitize); CSRF lừa gửi request hợp lệ (cơ chế tự gửi cookie), không cần JS chạy trên site.
- **Phòng chống**: `SameSite` cookie (`Lax`/`Strict`) là biện pháp mạnh nhất; thêm **CSRF token** gắn vào header mỗi request đổi state — site độc không đọc được token (Same-Origin Policy).

```ts
fetch('/api/transfer', { method: 'POST', credentials: 'include',
  headers: { 'X-CSRF-Token': getCsrfToken() }, body: JSON.stringify({ amount: 100 }) });
```

**Bẫy**: token trong localStorage + header `Authorization` thì miễn nhiễm CSRF nhưng đánh đổi rủi ro XSS (xem 9.7). Đừng tin `SameSite=Lax` chặn mọi thứ — vài request GET vẫn lọt, nên thao tác đổi state không bao giờ dùng GET.

</details>

---

### 9.12. Nhìn từ góc độ frontend, GraphQL khác REST như thế nào? Khi nào bạn chọn cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Nhìn từ phía frontend, khác biệt lớn nhất là ai quyết định shape dữ liệu. Với REST, mình gọi nhiều endpoint theo từng resource và server quyết định trả về cái gì, nên hay bị over-fetching (lấy dư field) hoặc under-fetching (phải gọi thêm nhiều lần), nhưng bù lại tận dụng HTTP caching, CDN và ETag rất dễ, status code cũng rõ nghĩa. Với GraphQL thì chỉ một endpoint /graphql và client tự khai báo đúng những field cần, một query có thể lấy luôn dữ liệu lồng từ nhiều resource nên giảm round-trip và tránh over/under-fetching — nhưng caching HTTP khó hơn nhiều vì thường đi qua POST. Mình chọn REST cho CRUD đơn giản, cần cache CDN mạnh, API public ổn định và dễ debug. Mình chọn GraphQL khi UI cần nhiều shape dữ liệu khác nhau, dữ liệu lồng sâu, hoặc nhiều client web/mobile có nhu cầu field khác nhau. Bẫy phía FE mình luôn nhắc: GraphQL hay trả lỗi dưới status 200 kèm field errors, nên không thể chỉ dựa res.ok mà phải kiểm tra body, và caching thường phải dựa vào normalized cache của Apollo hay urql."

---

- **REST**: nhiều endpoint theo resource, server quyết định shape, hay over/under-fetching, nhưng tận dụng HTTP caching/CDN/ETag dễ và status code rõ nghĩa.
- **GraphQL**: một endpoint `/graphql`, client tự khai báo field cần → tránh over/under-fetching, một query lấy được dữ liệu lồng từ nhiều resource; nhưng caching HTTP khó (thường POST).
- **Chọn REST**: CRUD đơn giản, cần CDN cache mạnh, API public ổn định, dễ debug.
- **Chọn GraphQL**: UI cần nhiều shape dữ liệu, dữ liệu lồng nhiều cấp, nhiều client (web/mobile) nhu cầu field khác nhau, muốn giảm round-trip.

```graphql
query { user(id: "1") { name posts(last: 5) { title comments { text } } } }
```

**Bẫy GraphQL phía FE**: lỗi thường trả `200` kèm field `errors` → không thể chỉ dựa `res.ok`, phải kiểm tra body. Caching khó (POST) nên dựa vào normalized cache của Apollo/urql. Rủi ro query quá nặng/lồng sâu — server cần giới hạn độ phức tạp.

</details>

## 10. System Design cho Frontend

### 10.1. Bạn sẽ tổ chức kiến trúc thư mục cho một ứng dụng React lớn (100+ trang, nhiều team) như thế nào? Vì sao chọn feature-based thay vì gom theo loại file?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Với app lớn nhiều team mình tổ chức theo feature-based, hay còn gọi là vertical slice, chứ không gom theo loại file. Lý do là cách gom theo kỹ thuật — kiểu một folder components, một folder hooks, một folder services — sẽ sụp đổ ở quy mô lớn: một tính năng bị rải khắp nhiều folder, và khi cần gỡ tính năng đó đi thì không ai biết chắc phải xóa file nào. Feature-based thì mỗi feature tự chứa component, hook, api, store, type của riêng nó, nên mở một thư mục là thấy trọn vẹn nghiệp vụ, thêm hay xóa tính năng đều gọn. Bên cạnh đó mình vẫn giữ một tầng shared cho UI dùng chung, lib và hook hạ tầng. Nguyên tắc quan trọng là luồng phụ thuộc một chiều: shared được dùng bởi features, features được dùng bởi app, và feature không import chéo feature — cần dùng chung thì đẩy lên shared. Mình ép kỷ luật này bằng public API qua index.ts và ESLint no-restricted-imports hoặc Nx boundaries, và map ownership từng team qua CODEOWNERS. Bẫy mình tránh là để shared hay utils phình to thành bãi rác gây coupling ngầm — chỉ đẩy lên shared khi có từ hai feature trở lên thật sự dùng chung."

---

- Gom theo **kỹ thuật** (`components/`, `hooks/`, `services/`) sụp đổ ở quy mô lớn: một tính năng rải rác nhiều folder, khó biết xoá file nào khi gỡ tính năng.
- **Feature-based (vertical slice)**: nhóm theo nghiệp vụ, mỗi feature tự chứa component/hook/api/store/type; ngoài ra có `components/` (UI dùng chung), `lib/`, `hooks/` hạ tầng.
- **Luồng phụ thuộc một chiều**: `shared` ← `features` ← `app`; **feature không import chéo feature** (cần chung thì đẩy lên shared).
- **Public API qua `index.ts`** (ép bằng ESLint `no-restricted-imports` / Nx boundaries) và **ownership** map team qua `CODEOWNERS`.

| Tiêu chí | Type-based | Feature-based |
|---|---|---|
| Tìm/xoá 1 tính năng | Rải rác | Một chỗ |
| Co-location, quy mô lớn | Khó | Tốt |

**Bẫy**: `shared/` hoặc `utils/` khổng lồ thành "bãi rác" → coupling ngầm; chỉ lên shared khi **2+ feature** dùng chung.

</details>

---

### 10.2. Khi xây một component library / design system dùng chung cho nhiều team, bạn quan tâm những vấn đề kỹ thuật nào? Trình bày từ API component, versioning đến tài liệu.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Khi xây design system dùng chung, mình quan tâm ba nhóm vấn đề: thiết kế API, versioning và tài liệu/độ tin cậy. Về API component, mình muốn nó dựa trên design tokens — ví dụ truyền variant='danger' thay vì một mã màu hex — và component không tự gán margin hay layout cố định để nơi dùng kiểm soát bố cục; mình luôn forward ref và spread các prop HTML còn lại để aria-*, data-*, onClick đi qua được, đảm bảo tính linh hoạt và accessibility. Về versioning, mình publish lên npm theo semver, coi việc đổi prop bắt buộc hay xóa variant là breaking change major, dùng changesets để quản changelog tự động và cung cấp codemod cho những breaking lớn để team migrate đỡ đau. Về tài liệu và tin cậy, mình dùng Storybook để có story cho mọi trạng thái, chạy visual regression bằng Chromatic hay Playwright để bắt thay đổi giao diện ngoài ý muốn, và test a11y tự động với jest-axe. Bẫy lớn nhất là cân bằng độ linh hoạt: nhận quá nhiều prop tùy biến thì mất tính thống nhất, còn quá cứng thì team sẽ tự fork ra dùng riêng; và component dùng chung phải thuần presentational, không tự gọi API hay đọc router."

---

- **API component**: dựa trên **design tokens** (`color="danger"` không phải hex), không gán margin/layout cố định, hỗ trợ `asChild`/polymorphic, **forward ref** + spread props HTML còn lại (cho `aria-*`, `data-*`, `onClick` đi qua).
- **Versioning**: publish npm theo **semver** (đổi prop bắt buộc / xoá variant = major), dùng **changesets** quản changelog, cung cấp **codemod** cho breaking lớn.
- **Tài liệu & tin cậy**: **Storybook** (stories cho mọi trạng thái), **visual regression** (Chromatic/Playwright), test a11y tự động (`jest-axe`).

```tsx
type ButtonProps = React.ComponentPropsWithoutRef<'button'> & {
  variant?: 'primary' | 'secondary' | 'danger';
};
export const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant = 'primary', className, ...rest }, ref) => (
    <button ref={ref} className={cn(buttonVariants({ variant }), className)} {...rest} />
  ),
);
```

**Bẫy**: quá linh hoạt (nhận cả `style`/`className`/20 props) → mất thống nhất; quá cứng → team fork; component dùng chung **phải presentational**, không tự gọi API / đọc router.

</details>

---

### 10.3. Thiết kế frontend cho ứng dụng chịu tải 1 triệu+ user/ngày: bạn dùng những kỹ thuật nào về CDN, cache và rendering để giảm tải server và tăng tốc?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Triết lý của mình là đẩy càng nhiều việc ra biên (edge/CDN) càng tốt để server gốc làm ít nhất. Đầu tiên mình tách rõ hai loại nội dung: phần tĩnh dùng chung và phần cá nhân hoá theo user. Phần tĩnh thì stative hoá rồi cache ở CDN — asset có hash cho cache vĩnh viễn (immutable), còn HTML thì cache ngắn kèm stale-while-revalidate để vừa nhanh vừa tươi. Mình chọn rendering theo từng trang chứ không một kiểu cho tất cả: landing/blog dùng SSG/ISR để 1 triệu lượt xem chỉ regenerate một lần mỗi 60s rồi serve hết từ CDN, còn dashboard cá nhân hoá thì SSR streaming hoặc CSR. Quan trọng là cache nhiều tầng browser → CDN → Redis → DB, mỗi tầng chặn bớt một phần tải. Cái bẫy lớn nhất là cá nhân hoá toàn trang kiểu 'Chào anh Nam' làm không cache được gì — mình giải quyết bằng cách render khung tĩnh ở edge rồi chèn phần cá nhân hoá bằng client component, giữ được cache cho 99% nội dung."

---

- **Tĩnh hoá + CDN**: asset bất biến (có hash) đặt `Cache-Control: ...immutable` cache vĩnh viễn; HTML cache ngắn / `stale-while-revalidate`.
- **Chọn chiến lược render** theo trang: landing/blog → **SSG/ISR**; trang đổi vừa phải → **ISR** (`revalidate`); dashboard cá nhân hoá → **SSR/streaming hoặc CSR**.
- **Cache nhiều tầng**: browser → CDN edge → data cache (Redis) → DB; tách dữ liệu per-user khỏi dữ liệu dùng chung.
- **Tối ưu payload**: code splitting theo route, `next/image` (AVIF/WebP), Brotli/gzip, HTTP/2-3, preconnect.

```tsx
// ISR: 1 triệu lượt xem chỉ regenerate 1 lần mỗi 60s, còn lại serve từ CDN
export const revalidate = 60;
```

**Bẫy**: cá nhân hoá toàn trang (chào tên user) → không cache được gì; render khung tĩnh ở edge, chèn phần cá nhân hoá bằng client component/streaming.

</details>

---

### 10.4. Monorepo cho frontend (Turborepo / Nx): giải quyết vấn đề gì? Cấu trúc ra sao và build/CI được tối ưu thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Monorepo giải quyết bài toán chia sẻ code giữa nhiều app mà không phải publish package lên npm rồi bump version liên tục — ví dụ sửa design system rồi cập nhật web app trong cùng một PR (atomic change), không sợ version lệch. Cấu trúc điển hình mình dùng là pnpm workspaces với apps/ chứa các app (web, admin) và packages/ chứa thứ dùng chung (ui, config, api-client, utils). Cái lợi lớn nữa là type-safe end-to-end và chuẩn hoá tooling qua một package config chung. Về CI thì điểm cốt lõi của Turborepo/Nx là chạy 'affected' — dựa trên task graph để chỉ build những gì thực sự bị ảnh hưởng — cộng với caching theo hash của input và remote cache chia sẻ giữa CI và máy dev, nên build nhanh hơn rất nhiều. Khác biệt là Turborepo nhẹ, thêm vào repo có sẵn còn Nx toàn diện hơn với plugin và module boundaries built-in. Bẫy hay gặp là cấu hình outputs/inputs sai gây cache miss build hoài, và monorepo không tự tạo ranh giới giữa các module nên vẫn cần ESLint boundaries để chống import lung tung."

---

- **Giải quyết**: chia sẻ code (design system/util/type) không cần publish/bump liên tục; thay đổi xuyên repo trong 1 PR; tránh version lệch.
- **Cấu trúc** (pnpm workspaces + Turborepo): `apps/` (web, admin) + `packages/` (ui, config, api-client, utils) + `turbo.json`.
- **Lợi ích**: **atomic change** (sửa `ui` + `web` 1 PR), **type-safe end-to-end**, chuẩn hoá tooling qua `packages/config`.
- **Tối ưu CI**: chạy **affected** (chỉ build thứ bị ảnh hưởng theo task graph), **caching** theo hash input, **remote cache** chia sẻ giữa CI/dev.

```jsonc
// turbo.json
{ "tasks": {
  "build": { "dependsOn": ["^build"], "outputs": [".next/**", "dist/**"] },
  "dev": { "cache": false, "persistent": true }
}}
```

| | Turborepo | Nx |
|---|---|---|
| Triết lý | Nhẹ, thêm vào repo | Toàn diện, nhiều plugin |
| Module boundaries | Tự lo (ESLint) | Có sẵn lint rule theo tag |

**Bẫy**: cấu hình `outputs`/`inputs` sai → cache miss (build hoài) hoặc cache "bẩn"; monorepo không tự tạo ranh giới module, vẫn cần ESLint boundaries.

</details>

---

### 10.5. Làm sao đảm bảo accessibility (a11y) ở quy mô lớn — không phải chỉ một trang mà toàn bộ ứng dụng, qua nhiều team và theo thời gian?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Ở quy mô lớn thì không thể đi sửa a11y từng trang một, nên mình tập trung vào điểm có đòn bẩy lớn nhất là các component dùng chung. Mình làm Modal, Dropdown, Tooltip đúng a11y một lần — thường build trên headless lib như Radix hoặc React Aria vì nó lo sẵn focus trap, ARIA, keyboard navigation — thì mọi nơi dùng đều được hưởng. Tiếp theo là tự động hoá trong CI: ESLint jsx-a11y bắt lỗi tĩnh, jest-axe hoặc axe-core/playwright fail build nếu vi phạm WCAG, để a11y không bị trôi theo thời gian. Nhưng tool chỉ bắt được khoảng 30-50% vấn đề thôi, nên vẫn cần con người test bằng bàn phím và screen reader định kỳ, và đưa a11y vào Definition of Done. Một điểm đặc thù của SPA là route change không reload trang nên mình phải tự quản focus — focus vào heading khi đổi route — và dùng aria-live region để screen reader thông báo. Bẫy lớn nhất là lạm dụng ARIA: 'ARIA thừa còn tệ hơn không có', nên mình luôn ưu tiên HTML ngữ nghĩa như button, nav trước đã."

---

- **Bắt từ gốc**: component dùng chung (Modal, Dropdown) đúng a11y một lần → mọi nơi hưởng; build trên **headless lib** (Radix / React Aria) lo sẵn focus trap, ARIA, keyboard. Đòn bẩy lớn nhất.
- **Tự động hoá trong CI**: `jest-axe` / `@axe-core/playwright` fail build nếu vi phạm WCAG; ESLint `jsx-a11y` bắt lỗi tĩnh; Storybook addon a11y.
- **Quy trình con người** (tool chỉ bắt ~30-50%): test bằng bàn phím (Tab/Esc) + screen reader (NVDA/VoiceOver) định kỳ; đưa a11y vào **Definition of Done**.
- **Đặc thù SPA**: route change không reload → quản lý focus thủ công (focus heading) + **live region** (`aria-live`); quản focus khi mở/đóng modal/toast.

```tsx
it('Form không vi phạm a11y', async () => {
  const { container } = render(<SignupForm />);
  expect(await axe(container)).toHaveNoViolations();
});
```

**Bẫy**: lạm dụng ARIA ("ARIA thừa còn tệ hơn không có") → ưu tiên HTML ngữ nghĩa (`<button>`, `<nav>`); đừng tưởng chỉ tool tự động là "đạt chuẩn".

</details>

---

### 10.6. Thiết kế chiến lược xử lý lỗi toàn cục cho một ứng dụng React/Next.js: error boundary, lỗi network qua interceptor, và báo cáo lỗi (observability).

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Đầu tiên mình phân biệt hai loại lỗi vì cách bắt hoàn toàn khác nhau. Lỗi render đồng bộ thì dùng Error Boundary, còn lỗi bất đồng bộ như network hay promise thì Error Boundary không bắt được — phải xử lý qua try/catch, interceptor hoặc state của query. Error Boundary mình đặt phân tầng: ở root như global-error/error để không bao giờ bị màn hình trắng, và boundary cục bộ quanh các widget rủi ro để một widget hỏng không kéo sập cả trang. Lỗi network thì mình tập trung vào một interceptor: 401 thì refresh token hoặc về login, 403 thì trang cấm, 5xx thì toast báo lỗi rồi log lại — nhưng vẫn reject để TanStack Query xử lý cục bộ ở chỗ gọi. Cuối cùng là observability với Sentry để bắt runtime error và unhandled rejection, kèm source map, release tag và breadcrumb để debug được trên production. Bẫy mình luôn tránh là nuốt lỗi im lặng kiểu catch rỗng — app đơ mà không ai biết — và phải dedupe toast khi nhiều query cùng fail, đồng thời phân biệt lỗi nào hiện cho user còn lỗi nào chỉ log."

---

- Phân biệt **hai loại lỗi**: lỗi **render** (đồng bộ) → **Error Boundary**; lỗi **bất đồng bộ** (network/promise) → try/catch / interceptor / state của query. Error boundary KHÔNG bắt lỗi event handler, async, SSR.
- **Error Boundary phân tầng**: root (`app/global-error.tsx`, `app/error.tsx`) chặn crash trắng màn hình; boundary cục bộ quanh widget rủi ro để 1 widget hỏng không sập cả trang.
- **Network qua interceptor tập trung**: 401 → refresh/login, 403 → trang cấm, 5xx → toast + log + retry; vẫn `reject` để TanStack Query xử lý cục bộ.
- **Observability**: **Sentry** bắt runtime error + unhandled rejection, kèm **source map**, **release tag**, breadcrumb/user context.

```ts
api.interceptors.response.use((res) => res, (error) => {
  const status = error.response?.status;
  if (status === 401) redirectToLogin();
  else if (status >= 500) { toast.error('Lỗi hệ thống.'); reportError(error); }
  return Promise.reject(error);
});
```

**Bẫy**: nuốt lỗi im lặng (`catch (e) {}`) → app "đơ" mà không ai biết; toast lỗi trùng lặp khi nhiều query cùng fail → cần gộp/dedupe và phân biệt lỗi hiện cho user vs chỉ log.

</details>

---

### 10.7. "Performance budget" là gì? Bạn đặt budget thế nào và làm sao ép buộc nó trong quy trình phát triển/CI để không bị "trôi" theo thời gian?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Performance budget là các ngưỡng định lượng cho hiệu năng — bundle size, thời gian tải, Core Web Vitals — mà sản phẩm cam kết không được vượt. Bản chất là biến hiệu năng từ thứ mơ hồ 'thấy hơi chậm' thành thứ đo được và bắt buộc. Mình thường đặt theo ba nhóm: kích thước (ví dụ JS đầu trang dưới 170KB gzip), chỉ số thời gian (LCP dưới 2.5s, INP dưới 200ms, CLS dưới 0.1), và theo route. Quan trọng là chọn số dựa trên thiết bị và mạng của người dùng thật — mobile 4G chứ không phải máy dev xịn. Phần then chốt nhất là ép buộc trong CI: dùng size-limit để fail PR khi vượt ngưỡng, Lighthouse CI assert các chỉ số, vì nếu budget không gắn CI thì mỗi PR thêm vài KB, sáu tháng sau bundle gấp đôi mà không ai để ý. Để giữ trong budget thì code splitting với dynamic import, tree-shaking, thay lib nặng như moment bằng dayjs. Bẫy hay quên là budget cho bên thứ ba — analytics, ads, chat widget — thường đó mới là thứ chậm nhất."

---

- **Performance budget**: các **ngưỡng định lượng** cho hiệu năng (bundle size, thời gian tải, Core Web Vitals) mà sản phẩm cam kết không vượt → biến hiệu năng thành thứ **đo được và bắt buộc**.
- **Đặt theo 3 nhóm**: kích thước (JS đầu trang ≤ 170KB gzip), chỉ số thời gian (LCP < 2.5s, INP < 200ms, CLS < 0.1), theo route (mỗi route thêm tối đa N KB). Chọn số dựa trên **thiết bị/mạng người dùng thật** (mobile 4G), không phải máy dev.
- **Ép buộc trong CI** (quan trọng nhất): `size-limit`/`bundlesize` fail PR khi vượt; **Lighthouse CI** assert ngưỡng; dashboard theo dõi xu hướng.
- **Giữ trong budget**: code splitting + `dynamic()`, tree-shaking, thay lib nặng (moment → dayjs), lazy ảnh, `@next/bundle-analyzer`.

```jsonc
// .size-limit.json
[ { "name": "App shell JS", "path": ".next/static/**/*.js", "limit": "170 KB" } ]
```

**Bẫy**: chỉ đo trên máy dev mạnh → budget ảo; budget không gắn CI → 6 tháng sau bundle gấp đôi (mỗi PR thêm vài KB); quên budget cho **bên thứ ba** (analytics, ads, chat) — thường là thứ chậm nhất.

</details>

---

### 10.8. Khi thiết kế kiến trúc quản lý state cho một ứng dụng lớn, bạn phân loại state thế nào và chọn công cụ nào cho từng loại? (server state, client/UI state, form, URL)

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Sai lầm kinh điển là nhét tất cả vào một Redux store, kể cả dữ liệu từ server. Mình phân loại state trước rồi mới chọn công cụ. Server state — dữ liệu từ API — thực chất là cache của server chứ không phải state của mình, nên mình để TanStack Query lo caching, dedupe, retry, invalidation thay vì tự viết loading/error/data đầy bug và không nhét nó vào global store. Client/UI state toàn cục như theme hay sidebar thì dùng Zustand là đủ, chỉ dùng Redux Toolkit khi cần devtools hay middleware mạnh. Form thì React Hook Form cộng Zod để validate. Một loại hay bị quên là URL state — filter, tab, phân trang nên để trên URL qua search params để share link được và reload không mất. Còn lại state cục bộ thì cứ useState/useReducer, đặt càng gần nơi dùng càng tốt. Điểm mấu chốt là sau khi tách server state và URL ra thì phần client state toàn cục còn lại nhỏ hơn nhiều so với mình tưởng. Bẫy là dùng Context cho state đổi liên tục khiến mọi consumer re-render — Context chỉ hợp giá trị ổn định như theme/locale."

---

- Sai lầm kinh điển: nhét tất cả vào một Redux store, kể cả dữ liệu server. Phải **phân loại** trước rồi chọn công cụ.

| Loại state | Công cụ phù hợp |
|---|---|
| **Server state** (API) | **TanStack Query** / RTK Query |
| **Client/UI toàn cục** (theme, sidebar) | **Zustand** / Redux Toolkit / Context |
| **Form** | **React Hook Form** + **Zod** |
| **URL** (filter, tab, phân trang) | **search params** / router |
| **Cục bộ component** | `useState` / `useReducer` |

- **Server state KHÔNG nằm trong global store** — nó là **cache** của dữ liệu server; TanStack Query lo caching/dedupe/retry/invalidation thay vì tự viết `loading/error/data` đầy bug.
- **URL là store bị quên**: filter/tab/trang nên ở URL (`useSearchParams`) để share link và reload không mất.
- **Client state toàn cục ít hơn bạn nghĩ**: sau khi tách server + URL ra, phần còn lại nhỏ → **Zustand** đủ; chọn Redux Toolkit khi cần devtools/middleware mạnh.

**Bẫy**: đồng bộ server data vào global store rồi tự quản "stale" (tái phát minh TanStack Query); dùng **Context cho state đổi liên tục** → mọi consumer re-render (Context hợp giá trị ổn định: theme/locale); đặt state global khi chỉ 1 component dùng → đặt **càng gần nơi dùng càng tốt**.

</details>

## 11. Testing Frontend

### 11.1. Hãy giải thích "kim tự tháp test" (test pyramid) so với "testing trophy". Là một frontend developer làm React, bạn nghiêng về mô hình nào và tại sao?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Test pyramid của Mike Cohn là mô hình cổ điển: rất nhiều unit test ở đáy vì nhanh và rẻ, ít integration ở giữa, và rất ít E2E ở đỉnh vì chậm và giòn. Testing trophy của Kent C. Dodds thì đảo lại trọng tâm: bốn tầng Static (TypeScript, lint) → Unit → Integration → E2E, trong đó tầng Integration phình to nhất vì nó cho ROI cao nhất — tức là cân bằng tốt nhất giữa độ tin cậy và tốc độ. Là frontend làm React, mình nghiêng hẳn về trophy. Lý do là một component hiếm khi chạy độc lập, giá trị thực nằm ở chỗ nhiều component ráp lại và phản ứng đúng với tương tác của user. Nên mình viết nhiều integration test kiểu render component thật rồi giả lập user click, gõ, và kiểm tra kết quả họ thấy trên màn hình. Cái bẫy mình tránh là viết quá nhiều unit test cho từng component nhỏ và test vào implementation detail — loại đó vỡ liên tục mỗi lần refactor dù hành vi không hề đổi, tốn công mà không tăng độ tin cậy."

---

- **Test pyramid** (Mike Cohn): nhiều **unit** ở đáy (nhanh, rẻ), ít **integration** ở giữa, rất ít **E2E** ở đỉnh (chậm, giòn).
- **Testing trophy** (Kent C. Dodds): 4 tầng `Static` (TS, lint) → `Unit` → **Integration** (phình to nhất) → `E2E`. Ưu tiên integration vì ROI cao nhất.
- Pyramid hợp backend/logic thuần; trophy hợp UI component.
- Với React tôi chọn **trophy**: nhiều integration test render component thật + giả lập tương tác user.
- **Bẫy**: viết quá nhiều unit test cho từng component nhỏ (test implementation detail) → vỡ liên tục khi refactor dù hành vi không đổi.

```tsx
// Integration (ưu tiên với React): render thật + tương tác như user
render(<CheckoutForm />);
await userEvent.click(screen.getByRole('button', { name: /thanh toán/i }));
expect(await screen.findByText(/đặt hàng thành công/i)).toBeInTheDocument();
```

</details>

---

### 11.2. Triết lý cốt lõi của React Testing Library (RTL) là gì? Tại sao RTL khuyên ta tránh test "implementation detail"?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Triết lý cốt lõi của RTL nằm gọn trong một câu của Kent C. Dodds: test càng giống cách phần mềm được dùng thật thì càng cho mình sự tự tin. Nghĩa là test phải tương tác như người dùng cuối — tìm element qua text, role, label rồi click, gõ — chứ không thò tay vào state, props hay instance method bên trong. Lý do tránh implementation detail là vì nó gây ra hai vấn đề ngược nhau. Một là false negative: mình refactor đổi từ useState sang useReducer, hay đổi tên một biến state, hành vi y hệt nhưng test vỡ — test trở thành gánh nặng. Hai là false positive: test pass nhưng app thực ra hỏng, vì mình kiểm tra biến nội bộ đúng chứ không kiểm tra cái user thực sự thấy. RTL cố tình không cho API đọc state hay props — nếu mình thấy 'cần' đọc state để test thì đó là dấu hiệu đang test sai tầng. Test bám vào hành vi quan sát được thì sống sót qua refactor và mới thật sự bảo vệ được sản phẩm."

---

- Triết lý: "The more your tests resemble the way your software is used, the more confidence they can give you" — test càng giống cách dùng thật, càng tự tin.
- Tương tác như **người dùng cuối**: tìm qua text/role/label, click, gõ — không thò vào state/props/instance method.
- **Implementation detail** (tên state, `useReducer` vs `useState`, class CSS) gây: **false negative** (refactor là vỡ test) và **false positive** (test pass nhưng app hỏng).
- RTL cố tình không có API đọc state/props — nếu "cần" tức là đang test sai tầng.

```tsx
// ❌ test implementation detail (enzyme): expect(wrapper.state('count')).toBe(1);
// ✅ RTL: test cái user thấy
await userEvent.click(screen.getByRole('button', { name: /tăng/i }));
expect(screen.getByText('Số đếm: 1')).toBeInTheDocument();
```

</details>

---

### 11.3. RTL khuyến nghị một thứ tự ưu tiên khi chọn query (getByRole, getByLabelText, getByText, getByTestId...). Hãy trình bày thứ tự đó và lý do.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"RTL chia query thành ba nhóm theo thứ tự ưu tiên. Nhóm một là thứ ai cũng truy cập được, ưu tiên cao nhất: getByRole tốt nhất vì nó phản ánh đúng accessibility tree, rồi đến getByLabelText cho form field, getByPlaceholderText, getByText, getByDisplayValue. Nhóm hai là semantic như getByAltText, getByTitle. Nhóm ba là getByTestId, chỉ dùng khi hết cách. Lý do của thứ tự này rất hay: ưu tiên getByRole ép mình viết HTML có ngữ nghĩa và accessible — dùng đúng button, heading, đặt label cho input — nên test tốt và a11y tốt tự nhiên đi cùng nhau, một mũi tên trúng hai đích. Còn testId là 'cửa sau', nó không đảm bảo gì về trải nghiệm thật của user nên mình để cuối. Một điểm mình hay nhấn thêm là phân biệt biến thể: getBy ném lỗi nếu không tìm thấy, queryBy trả null và dùng để khẳđịnh element KHÔNG tồn tại, còn findBy trả Promise dùng cho element xuất hiện bất đồng bộ."

---

- **1. Accessible to everyone**: `getByRole` (tốt nhất, phản ánh a11y tree) → `getByLabelText` (form field) → `getByPlaceholderText` → `getByText` → `getByDisplayValue`.
- **2. Semantic**: `getByAltText`, `getByTitle`.
- **3. Test ID (cuối cùng)**: `getByTestId` — chỉ khi không còn cách nào.
- **Lý do**: ưu tiên `getByRole` ép viết HTML có ngữ nghĩa + accessible; test tốt và a11y tốt đi cùng nhau.
- **Bẫy**: lạm dụng `getByTestId`; nhầm biến thể — `getBy*` (lỗi nếu không có), `queryBy*` (trả `null`, khẳng định **không tồn tại**), `findBy*` (Promise, cho phần tử **bất đồng bộ**).

```tsx
screen.getByRole('button', { name: /đăng nhập/i }); // tốt nhất
screen.getByLabelText(/mật khẩu/i);                  // ổn
screen.getByTestId('user-avatar');                   // chỉ khi bí
```

</details>

---

### 11.4. Jest và Vitest khác nhau ở điểm gì? Khi nào bạn chọn cái nào trong dự án React/Next.js?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Cả Jest và Vitest đều là bộ test runner kèm assertion và mock, và API gần như tương thích nhau — describe, it, expect dùng y hệt — nên khác biệt chính nằm ở kiến trúc build chứ không phải cách viết test. Jest lâu đời của Meta, transform code qua Babel hoặc ts-jest, và xử lý ESM hơi rườm rà; điểm cộng lớn là Next.js có next/jest hỗ trợ chính thức nên tích hợp mượt. Vitest thì mới hơn, từ team Vite, transform bằng esbuild nên chạy rất nhanh, hỗ trợ ESM và TypeScript native, và dùng chung luôn vite.config nên dự án Vite-based gần như không cần cấu hình thêm. Trong dự án React/Next.js mình thường theo mặc định của framework: Next.js thì dùng Jest với next/jest, còn app dựng trên Vite thì Vitest cho nhanh và đỡ cấu hình. Migrate giữa hai cái cũng dễ, chủ yếu đổi jest.fn thành vi.fn, jest.mock thành vi.mock. Bẫy hay quên nhất là phải đặt testEnvironment hoặc environment thành jsdom, vì mặc định là node sẽ không có document và window để test component."

---

- Cả hai đều là test runner + assertion + mock, API gần như tương thích (`describe`, `it`, `expect`). Khác ở kiến trúc/build.
- **Jest**: lâu đời (Meta), transform Babel/ts-jest, ESM rườm rà; hợp **Next.js** (có `next/jest` chính thức).
- **Vitest**: mới (team Vite), transform esbuild rất nhanh, ESM/TS native, dùng chung `vite.config.ts`; hợp dự án **Vite-based**.
- Migrate Jest → Vitest khá dễ: đổi `jest.fn()`→`vi.fn()`, `jest.mock`→`vi.mock`.
- **Bẫy**: nhớ đặt `testEnvironment/environment: 'jsdom'` (mặc định là `node`, không có `document`/`window`).

```ts
import { describe, it, expect, vi } from 'vitest';
vi.mock('./api'); // ≈ jest.mock('./api')
```

</details>

---

### 11.5. MSW (Mock Service Worker) là gì và tại sao nó được ưa chuộng hơn mock `fetch`/`axios` trực tiếp khi test các component gọi API?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"MSW mock API ở tầng network chứ không phải tầng code. Thay vì viết jest.mock('axios') để giả lập thư viện gọi HTTP, mình để MSW chặn ngay request HTTP và trả về response theo handler mình định nghĩa, nên component vẫn gọi fetch hay axios y như thật, đi qua đúng vòng đời request gồm body, header, status. Cái hay nhất là test không bị gắn chặt vào implementation: sau này mình đổi từ axios sang fetch hay sang TanStack Query thì test không cần sửa gì, vì nó chỉ quan tâm tới request HTTP đi ra. Một lợi ích thực chiến nữa là cùng một bộ handler mình dùng lại được cho cả test, Storybook lẫn dev mode, đỡ trùng lặp. Mình hay dùng MSW khi test component gọi API, và cần lưu ý phải gọi resetHandlers trong afterEach để handler của test này không rò rỉ sang test khác, nếu không sẽ rất khó debug khi test pass riêng nhưng fail khi chạy chung."

---

- **MSW** mock API ở tầng **network**: chặn request HTTP qua Service Worker (browser) hoặc patch request layer (Node/test), trả response theo "handler". Component vẫn gọi `fetch`/`axios` như thật.
- **Không gắn chặt implementation**: đổi `axios`→`fetch`/TanStack Query không cần sửa test (khác `jest.mock('axios')`).
- **Dùng lại**: cùng handler cho test, Storybook, dev.
- **Sát thật**: đi qua đúng vòng đời request (body, header, status).
- **Bẫy**: quên `server.resetHandlers()` trong `afterEach` → handler "rò rỉ" sang test khác; thiếu polyfill `fetch`/`TextEncoder` ở Node cũ.

```ts
const server = setupServer(
  http.get('/api/users', () => HttpResponse.json([{ id: 1, name: 'An' }]))
);
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());
// override lỗi: server.use(http.get('/api/users', () => new HttpResponse(null, { status: 500 })));
```

</details>

---

### 11.6. E2E test là gì? So sánh Cypress và Playwright, và cho biết khi nào nên dùng E2E thay vì integration test với RTL.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"E2E là test chạy app thật trong trình duyệt thật với backend gần như thật, mô phỏng cả hành trình người dùng như đăng nhập rồi thêm vào giỏ rồi thanh toán, kiểm cả routing, cookie và navigation. Còn integration test với RTL chỉ render component trong jsdom, không có browser hay network thật. Giữa hai công cụ E2E thì Cypress chạy trong event loop của trình duyệt nên dùng quen tay nhưng hạn chế multi-tab và chạy song song phải trả tiền cho Cloud; còn Playwright của Microsoft điều khiển qua DevTools protocol, hỗ trợ multi-tab và multi-origin tốt, chạy song song miễn phí, auto-wait rất mạnh nên ít flaky hơn, nên gần đây mình ưu tiên Playwright. Về khi nào dùng E2E thì mình chỉ để cho critical path như đăng ký, checkout, thanh toán, vì E2E chậm và dễ flaky nên giữ càng ít càng tốt. Phần lớn logic UI mình test bằng integration với RTL vì nó nhanh, ổn định và đủ tự tin; đây chính là tư duy testing trophy, dồn nhiều vào integration và để E2E ở đỉnh chóp."

---

- **E2E** chạy app thật trong **trình duyệt thật**, backend gần thật, mô phỏng cả hành trình (đăng nhập → giỏ → thanh toán). RTL chỉ render trong jsdom, không có browser/routing/network thật.
- **Cypress** (Cypress.io): JS/TS, chạy trong event loop trình duyệt, multi-tab hạn chế, song song cần Cloud trả phí.
- **Playwright** (Microsoft): đa ngôn ngữ, điều khiển qua DevTools protocol, multi-tab/origin tốt, song song miễn phí sẵn có, auto-wait mạnh.
- **Dùng E2E khi**: critical path (đăng ký, thanh toán, checkout); kiểm thử tích hợp thật FE↔BE↔routing↔cookie/localStorage/navigation.
- E2E **chậm, dễ flaky** → giữ ít; phần lớn logic UI test bằng integration RTL.
- **Bẫy**: selector theo text/role bền hơn class/XPath; tránh `cy.wait(2000)` cứng, dùng auto-wait.

```ts
test('mua hàng end-to-end', async ({ page }) => {
  await page.goto('/');
  await page.getByRole('button', { name: 'Thêm vào giỏ' }).click();
  await expect(page.getByText('Đặt hàng thành công')).toBeVisible();
});
```

</details>

---

### 11.7. Làm thế nào để test một custom hook trong React? Cho ví dụ với `renderHook`.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Hook không gọi trực tiếp ngoài component được vì sẽ vi phạm rules of hooks, nên mình dùng renderHook từ @testing-library/react, nó render hook trong một component ẩn rồi trả về result.current để mình đọc giá trị. Khi mình gọi hàm làm đổi state thì phải bọc trong act, nếu không React sẽ cảnh báo 'not wrapped in act', còn nếu hook async thì dùng waitFor để chờ và mock API qua MSW; hook cần Context như TanStack Query hay Redux thì truyền thêm wrapper là Provider. Quan điểm thực chiến của mình là phần lớn trường hợp nên test hook gián tiếp qua chính component dùng nó, vì cuối cùng cái mình quan tâm là hành vi người dùng thấy, chứ không phải nội bộ hook. Mình chỉ dùng renderHook khi đó là hook tái sử dụng nhiều hoặc logic phức tạp, đáng tách ra test riêng cho rõ ràng."

---

- Hook không gọi được ngoài component → dùng `renderHook` (từ `@testing-library/react`) render hook trong component ẩn, trả `result.current`.
- Thay đổi state phải bọc trong `act` (hoặc dùng `userEvent`/`waitFor` đã tự bọc).
- Hook async: dùng `waitFor` chờ, mock API qua MSW. Hook cần Context: truyền `wrapper`.
- **Bẫy**: quên `act` → cảnh báo "not wrapped in act(...)"; thường tốt hơn là test hook **gián tiếp qua component dùng nó**, chỉ `renderHook` cho hook tái sử dụng/phức tạp.

```tsx
const { result } = renderHook(() => useCounter(0));
expect(result.current.count).toBe(0);
act(() => result.current.increment());
expect(result.current.count).toBe(1);

// có Provider:
const wrapper = ({ children }) => (
  <QueryClientProvider client={new QueryClient()}>{children}</QueryClientProvider>
);
renderHook(() => useTodos(), { wrapper });
```

</details>

---

### 11.8. Khi test một component có trạng thái bất đồng bộ (loading → success / error), bạn xử lý thế nào? Vì sao `findBy*` và `waitFor` quan trọng?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Component async thường đi qua ba trạng thái: loading rồi tới success hoặc error, và việc chuyển trạng thái xảy ra sau khi promise resolve, tức là ở một microtask sau. Nên nếu mình render xong rồi assert đồng bộ ngay thì data chưa kịp về, test sẽ fail; bắt buộc phải chờ. findBy thực chất là getBy cộng waitFor, nó trả về Promise và retry liên tục cho tới khi phần tử xuất hiện, nên mình dùng nó để khẳng định 'cái gì đó sẽ hiện ra'. Còn waitFor linh hoạt hơn, nó chờ tới khi callback assertion hết ném lỗi, hợp khi mình muốn chờ một phần tử biến mất hoặc chờ một điều kiện tổng quát. Lỗi hay gặp nhất là quên await trước findBy, hoặc dùng getBy cho phần tử chưa render xong khiến nó throw ngay. Mình cũng giữ callback trong waitFor thật gọn, chỉ một assertion, vì nó chạy lặp nhiều lần nên để nặng hoặc nhiều side-effect sẽ chậm và khó debug."

---

- Component async đi qua `loading` → `success`/`error`; cập nhật xảy ra sau khi promise resolve nên phải **chờ**, không assert đồng bộ ngay sau render.
- **`findBy*`** = `getBy*` + `waitFor`, trả Promise, retry tới khi phần tử **xuất hiện**.
- **`waitFor`**: chờ tới khi callback assertion hết ném lỗi; linh hoạt (vd chờ phần tử **biến mất** qua `waitForElementToBeRemoved`).
- **Bẫy**: quên `await` trước `findBy*`; dùng `getBy*` cho phần tử async (chưa render); dùng `queryBy*` để khẳng định **không tồn tại**; giữ callback `waitFor` gọn (1 assertion) vì chạy lặp nhiều lần.

```tsx
render(<PostList />);
expect(screen.getByText(/đang tải/i)).toBeInTheDocument();      // loading đồng bộ
expect(await screen.findByText('Bài 1')).toBeInTheDocument();   // chờ data
expect(screen.queryByText(/đang tải/i)).not.toBeInTheDocument(); // loading đã mất
```

</details>

---

### 11.9. Snapshot testing là gì? Nêu ưu, nhược điểm và khi nào nên (hoặc không nên) dùng.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Snapshot testing là mình chụp lại output đã serialize của component vào một file .snap, lần chạy sau Jest so sánh output mới với cái đã lưu, khác là fail và đề nghị mình cập nhật snapshot. Ưu điểm là viết rất nhanh và bắt regression UI tốt với những output ổn định như config, error message hay component presentational nhỏ. Nhưng nhược điểm khá lớn: nó dễ bị 'rubber-stamp', nghĩa là test fail thì người ta cứ bấm update cho qua mà không thực sự đọc diff, lúc đó test mất hết giá trị; ngoài ra nó giòn, chỉ đổi chút markup hay class name là vỡ dù hành vi không đổi, và snapshot lớn thì gần như không review nổi trong PR. Quan điểm của mình là chỉ dùng snapshot cho output nhỏ và ổn định, và ưu tiên inline snapshot để diff nằm ngay trong file test cho dễ đọc. Còn với component lớn hay động thì mình tránh, thay vào đó viết assertion có chủ đích như kiểm tra đúng text hoặc đúng role, vì nó diễn đạt rõ ý định test hơn nhiều."

---

- **Snapshot test** chụp output serialize của component vào file `.snap`, lần sau so sánh; khác thì fail và yêu cầu cập nhật (`--update`/`u`).
- **Ưu**: viết nhanh, bắt regression UI; tốt cho output ổn định (config, error message, component presentational nhỏ).
- **Nhược**: dễ "rubber-stamp" (update bừa không đọc diff); giòn (đổi markup/class là vỡ dù hành vi không đổi); snapshot lớn khó review; không diễn đạt **ý định** test.
- **Dùng**: output nhỏ/ổn định, ưu tiên **inline snapshot** (`toMatchInlineSnapshot`) để diff nằm trong test.
- **Tránh**: component lớn/động; ưu tiên assertion có chủ đích như `expect(...).toHaveTextContent('Lưu')`.

```tsx
const { container } = render(<Button variant="primary">Lưu</Button>);
expect(container).toMatchSnapshot();
```

</details>

---

### 11.10. Bạn cấu hình môi trường test cho một dự án Next.js như thế nào (jsdom, setup file, mock những gì)? Những thứ đặc thù Next nào hay phải mock?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Với Next.js mình dùng next/jest để Jest hiểu được build của Next, tức là dùng SWC transform, hiểu path alias, env và CSS module mà mình không phải tự cấu hình Babel; rồi đặt testEnvironment là jsdom và thêm một setup file trong setupFilesAfterEnv để import @testing-library/jest-dom cho có các matcher như toBeInTheDocument. Thứ đặc thù Next hay phải mock nhất là các hook điều hướng từ next/navigation như useRouter, usePathname, useSearchParams, vì jsdom không có router thật nên gọi vào sẽ lỗi; đôi khi mình cũng mock next/image, còn next/link thì thường chạy ổn. Điểm quan trọng cần nói rõ là Server Components, loại async và fetch trực tiếp, không render được trong jsdom, nên mình hoặc để cho E2E kiểm, hoặc tách phần logic ra hàm thuần để unit test riêng. Client Component có 'use client' thì test bình thường với RTL như component React thường."

---

- Dùng `next/jest` để Jest hiểu build của Next (SWC transform, path alias, env, CSS module); đặt `testEnvironment: 'jsdom'` + `setupFilesAfterEnv` import `@testing-library/jest-dom`.
- **Hay phải mock/lưu ý**: `useRouter`/`usePathname`/`useSearchParams` từ `next/navigation` (jsdom không có router); đôi khi `next/image`; `next/link` thường chạy ổn (navigation thật để E2E).
- **Server Components** (async, fetch) không render được trong jsdom → để E2E hoặc tách logic ra hàm thuần; Client Component (`'use client'`) test bình thường bằng RTL.

```tsx
jest.mock('next/navigation', () => ({
  useRouter: () => ({ push: jest.fn(), replace: jest.fn() }),
  usePathname: () => '/blog',
  useSearchParams: () => new URLSearchParams(),
}));
```

</details>

---

### 11.11. Hãy kể về một lần bạn xử lý test "flaky" (lúc pass lúc fail) hoặc thiết lập chiến lược test cho một dự án frontend. (Câu hỏi hành vi)

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

**📌 Câu trả lời mẫu:**

"Ở một dự án React/Next, CI của tụi mình hay đỏ ngẫu nhiên, khoảng 10-15% build E2E fail không rõ lý do, đến mức cả team mất niềm tin và quen tay bấm 're-run' cho qua. Mình nhận xử lý vì test mà không đáng tin thì coi như không có test. Thay vì re-run, mình ngồi tìm nguyên nhân gốc và lọc ra ba thủ phạm chính: nhiều chỗ chờ cứng bằng setTimeout hoặc cy.wait(2000) nên phụ thuộc tốc độ máy, có chỗ thiếu await trước findBy* gây race condition, và MSW handler bị rò rỉ giữa các test vì quên resetHandlers. Mình thay chờ cứng bằng auto-wait của Playwright cùng findBy*/waitFor, thêm afterEach(resetHandlers) để cô lập test, và bật ESLint rule testing-library/await-async-queries để chặn lỗi thiếu await ngay từ đầu. Song song đó mình soạn một bộ quy ước test cho team: ưu tiên getByRole, không test implementation detail, dùng MSW cho mọi mock, và theo mô hình testing trophy. Kết quả là tỉ lệ flaky E2E từ khoảng 15% xuống dưới 1%, vòng CI nhanh và đáng tin hơn nên team mới dám chặn merge khi test đỏ, và người mới cũng viết test đúng hướng ngay. Điều mình muốn nhấn là chẩn đoán đúng nguyên nhân gốc và dùng công cụ đúng như auto-wait thay vì sleep, với tác động đo được rõ ràng."

---

Trả lời theo **STAR**; thay bằng trải nghiệm thật của bạn.

- **Situation**: CI dự án React/Next thường đỏ ngẫu nhiên, ~10-15% build E2E fail không rõ lý do, team mất niềm tin và quen bấm 're-run'.
- **Task**: điều tra, làm ổn định bộ test để CI đáng tin, đồng thời lập quy ước test cho team.
- **Action**: tìm ra 3 nguyên nhân — chờ cứng `setTimeout`/`cy.wait(2000)`, thiếu `await` trước `findBy*` (race condition), MSW handler rò rỉ do quên `resetHandlers()`. Thay bằng auto-wait Playwright + `findBy*`/`waitFor`, thêm `afterEach(resetHandlers)`, bật ESLint `testing-library/await-async-queries`; soạn quy ước (ưu tiên `getByRole`, không test implementation detail, MSW cho mọi mock, testing trophy).
- **Result**: flaky E2E từ ~15% xuống dưới 1%, vòng CI nhanh hơn, team chặn merge khi test đỏ, người mới viết test đúng hướng.
- **Mẹo**: nhấn tư duy chẩn đoán nguyên nhân gốc, dùng công cụ đúng (auto-wait thay sleep), tác động đo được (% flaky, thời gian CI).

</details>

## 12. Bài tập Coding (Live Coding)

### 12.1. Viết hook `useDebounce` để trì hoãn một giá trị (ví dụ: ô tìm kiếm). Giải thích vì sao cần `debounce` và sự khác biệt với `throttle`.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"useDebounce trả về một giá trị chỉ được cập nhật sau khi người dùng ngừng gõ trong một khoảng delay, ví dụ 500ms. Mục đích là với ô search, mình không muốn gọi API ngay mỗi ký tự, mà chờ người ta gõ xong mới gọi một lần, đỡ spam server và tránh race condition giữa các response. Mấu chốt nằm ở cleanup của useEffect: mỗi lần value đổi mình set một timer mới, nhưng cleanup sẽ clearTimeout cái timer cũ, nên chỉ lần gõ cuối cùng mới 'sống sót' và thực sự set giá trị debounced. Khác biệt với throttle là debounce dồn về một lần ở cuối khi đã ngừng, còn throttle thì chạy đều đặn tối đa một lần mỗi khoảng, hợp với sự kiện liên tục như scroll hay resize. Lỗi hay gặp là quên clearTimeout khiến timer chồng nhau, hoặc gọi API thẳng trong onChange thay vì lắng nghe giá trị đã debounce, lúc đó coi như debounce vô tác dụng."

---

- `debounce` chỉ cập nhật giá trị sau khi người dùng *ngừng* gõ trong `delay`ms; hợp với ô search (không gọi API mỗi ký tự).
- Mấu chốt: `clearTimeout` trong cleanup của `useEffect` -> mỗi lần `value` đổi thì timer cũ bị huỷ, chỉ lần cuối "sống sót".
- Khác `throttle`: debounce "dồn" về 1 lần ở *cuối*; throttle chạy *tối đa 1 lần mỗi khoảng* (hợp `scroll`/`resize`).
- Bẫy: quên `clearTimeout`; gọi API thẳng trong `onChange` thay vì lắng nghe giá trị đã debounce.

```tsx
function useDebounce<T>(value: T, delay = 500): T {
  const [debounced, setDebounced] = useState(value);
  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id); // huỷ timer cũ
  }, [value, delay]);
  return debounced;
}
```

</details>

---

### 12.2. Viết hàm `deepClone(obj)` để sao chép sâu một object **không dùng thư viện**. Vì sao `structuredClone` hay `JSON.parse(JSON.stringify(...))` chưa đủ trong mọi trường hợp?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"deepClone là tạo một bản sao hoàn toàn độc lập, sửa bản sao không ảnh hưởng bản gốc ở mọi tầng lồng nhau. Khi tự viết, mình đệ quy qua object và array, xử lý riêng các kiểu đặc biệt như Date, Map, Set, và quan trọng nhất là dùng một WeakMap để nhớ những object đã clone nhằm xử lý vòng tham chiếu, nếu không sẽ đệ quy vô hạn và tràn stack. Lý do hai cách nhanh chưa đủ: JSON.parse(JSON.stringify) tuy ngắn nhưng làm mất Date, undefined, function, Map, Set và sẽ ném lỗi khi gặp circular reference; còn structuredClone tốt hơn nhiều vì hỗ trợ Date, Map, Set và cả circular, nhưng nó không clone được function và không giữ prototype tùy biến của class. Trong React thì mình ít khi deep clone để update state, vì cách đúng là cập nhật bất biến từng tầng hoặc dùng Immer cho gọn; deepClone chủ yếu để chứng minh mình hiểu bản chất tham chiếu và các edge case này."

---

- Deep clone tạo bản sao hoàn toàn độc lập; cần xử lý object/array lồng nhau, `Date`, `Map`, `Set`, và **vòng tham chiếu** (dùng `WeakMap` nhớ object đã clone).
- `JSON.parse(JSON.stringify(x))`: ngắn nhưng mất `Date`/`undefined`/`function`/`Map`/`Set` và lỗi với circular.
- `structuredClone(x)`: hỗ trợ `Date`/`Map`/`Set`/circular nhưng không clone `function`, không giữ prototype tuỳ biến.
- Trong React: deep clone hay cần khi update state lồng sâu, nhưng nên ưu tiên cập nhật bất biến từng tầng hoặc dùng Immer.

```ts
function deepClone<T>(value: T, seen = new WeakMap()): T {
  if (value === null || typeof value !== "object") return value;
  if (seen.has(value as object)) return seen.get(value as object); // circular
  if (value instanceof Date) return new Date(value.getTime()) as T;
  const result: any = Array.isArray(value) ? [] : {};
  seen.set(value as object, result);
  for (const k of Object.keys(value as object))
    result[k] = deepClone((value as any)[k], seen);
  return result;
}
```

</details>

---

### 12.3. Viết hàm `flatten` làm phẳng mảng lồng nhau ở độ sâu bất kỳ, và hàm `groupBy(array, keyFn)` gom nhóm phần tử.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"flatten là làm phẳng một mảng lồng nhau ở độ sâu bất kỳ thành mảng một chiều. Cách tự nhiên nhất là đệ quy kết hợp reduce: duyệt từng phần tử, nếu nó là mảng thì gọi flatten đệ quy rồi concat, không thì nối thẳng vào. JavaScript có sẵn arr.flat(Infinity) làm đúng việc này, nhưng phỏng vấn hay yêu cầu tự viết để xem mình hiểu đệ quy. groupBy thì mình cũng dùng reduce, trả về một object map từ key sang mảng phần tử, với key lấy từ một hàm keyFn người dùng truyền vào; ý tưởng giống Object.groupBy mới có trong chuẩn. Hai chỗ cần lưu ý: với mảng cực sâu thì đệ quy của flatten có thể tràn stack nên trong môi trường thật mình chuyển sang dùng stack thủ công; và keyFn trong groupBy phải trả về primitive như string hoặc number vì nó được dùng làm key của object."

---

- `flatten`: đệ quy + `reduce`, nối phần tử (nếu là mảng thì đệ quy tiếp). Bản gốc có `flat(Infinity)` nhưng phỏng vấn hay yêu cầu tự viết.
- `groupBy`: `reduce` trả object map từ key -> mảng phần tử (giống `Object.groupBy`).
- Bẫy: `flatten` đệ quy trên mảng cực sâu có thể tràn stack (dùng stack thủ công); `keyFn` phải trả primitive (string/number).

```ts
function flatten<T>(arr: any[]): T[] {
  return arr.reduce<T[]>((acc, x) =>
    acc.concat(Array.isArray(x) ? flatten<T>(x) : x), []);
}

function groupBy<T, K extends string | number>(arr: T[], keyFn: (i: T) => K) {
  return arr.reduce((acc, item) => {
    (acc[keyFn(item)] ||= []).push(item);
    return acc;
  }, {} as Record<K, T[]>);
}
```

</details>

---

### 12.4. Viết component `Dropdown` hỗ trợ **tìm kiếm (search)** và **chọn nhiều (multi-select)**. Cần lưu ý gì về accessibility và đóng menu khi click ra ngoài?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Dropdown này cần vài state cốt lõi: trạng thái mở đóng menu, ô input để lọc options, một mảng selected lưu các item đã chọn, hàm toggle để thêm hoặc bỏ một item, và xử lý click ra ngoài để đóng menu. Phần click-outside mình nghe sự kiện mousedown trên document chứ không nghe click, vì mousedown chính xác hơn và tránh trường hợp menu đóng trước khi xử lý chọn xong; trong handler mình kiểm tra nếu target không nằm trong ref của dropdown thì setOpen false, và nhớ removeEventListener trong cleanup để khỏi rò rỉ. Về accessibility, đây là điểm dễ ghi điểm: mình gắn role listbox cho menu và role option cho từng item, thêm aria-haspopup, aria-expanded, aria-multiselectable, và aria-selected trên item đang chọn; nếu làm thêm điều hướng bàn phím bằng mũi tên và Enter thì càng tốt. Cuối cùng để tái sử dụng, mình hay biến nó thành controlled component qua props value và onChange thay vì giữ selected nội bộ, để cha kiểm soát dữ liệu."

---

- Cần: state mở/đóng, ô input lọc options, mảng `selected`, toggle từng item, xử lý click ra ngoài để đóng.
- Click-outside: nghe `mousedown` (chính xác hơn `click`) trên `document`, kiểm tra `ref.current.contains(e.target)`.
- Accessibility: `role="listbox"/"option"`, `aria-haspopup`, `aria-expanded`, `aria-multiselectable`, `aria-selected`; điều hướng bàn phím (mũi tên + Enter) là điểm cộng.
- Tái sử dụng tốt hơn: biến thành controlled qua props `value`/`onChange`.

```tsx
useEffect(() => {
  const handler = (e: MouseEvent) => {
    if (ref.current && !ref.current.contains(e.target as Node)) setOpen(false);
  };
  document.addEventListener("mousedown", handler);
  return () => document.removeEventListener("mousedown", handler);
}, []);

const toggle = (v: string) => setSelected(p =>
  p.includes(v) ? p.filter(x => x !== v) : [...p, v]);
```

</details>

---

### 12.5. Viết hook `useFetch(url)` trả về `{ data, loading, error }` và **huỷ request khi unmount hoặc url đổi** bằng `AbortController`.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Bản chất của useFetch là gói logic gọi API vào một effect chạy lại mỗi khi url đổi, trả về data, loading, error để component dùng. Điểm quan trọng nhất là phải huỷ request cũ bằng AbortController trong hàm cleanup của useEffect, vì nếu không sẽ dính hai vấn đề: race condition (response cũ về sau response mới rồi ghi đè data sai), và setState sau khi component đã unmount. Khi gọi controller.abort() thì fetch ném ra AbortError, mình bắt và bỏ qua nó chứ không hiển thị như lỗi thật, vì đó là mình chủ động huỷ. Trong thực tế mình gần như luôn dùng TanStack Query thay vì tự viết, vì nó lo sẵn cache, dedupe, retry, refetch; còn useFetch chủ yếu để chứng minh là mình hiểu cơ chế bên dưới."

---

- Phải reset trạng thái khi `url` đổi, xử lý lỗi, và **huỷ request cũ** bằng `AbortController` -> tránh race condition và setState sau unmount.
- Bỏ qua `AbortError` (lỗi do chính ta abort), không hiển thị như lỗi thật.
- Trong dự án thật nên dùng **TanStack Query** (lo cache, dedupe, retry, SWR); `useFetch` chủ yếu để chứng minh hiểu bản chất.

```tsx
useEffect(() => {
  const controller = new AbortController();
  setLoading(true); setError(null);
  fetch(url, { signal: controller.signal })
    .then(r => { if (!r.ok) throw new Error(`HTTP ${r.status}`); return r.json(); })
    .then(setData)
    .catch(e => { if (e.name !== "AbortError") setError(e); })
    .finally(() => { if (!controller.signal.aborted) setLoading(false); });
  return () => controller.abort(); // huỷ khi url đổi / unmount
}, [url]);
```

</details>

---

### 12.6. Viết hai hook tiện ích nhỏ: `usePrevious(value)` lấy giá trị trước đó, và `useLocalStorage(key, initial)` đồng bộ state với `localStorage`.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Hai hook này nhỏ nhưng test khá tốt độ hiểu của mình về timing trong React. usePrevious dựa trên một mẹo: mình lưu giá trị vào useRef và chỉ cập nhật ref bên trong useEffect — mà effect chạy SAU khi render xong, nên trong lúc render hiện tại ref vẫn còn giữ giá trị của lần trước; lần đầu thì trả undefined. Còn useLocalStorage thì mình dùng lazy initializer để đọc localStorage đúng một lần lúc khởi tạo, rồi ghi ngược lại trong useEffect khi value đổi, và bọc try/catch để phòng quota đầy hay bị chặn. Cái bẫy lớn nhất là SSR trong Next.js: trên server không có localStorage, nên nếu đọc ngay lúc render sẽ lỗi hoặc gây hydration mismatch — mình khởi tạo bằng initial rồi mới đọc localStorage trong useEffect ở phía client."

---

- `usePrevious`: lưu giá trị trong `useRef`, cập nhật ref trong `useEffect` (chạy *sau* render) nên lúc render hiện tại ref vẫn giữ giá trị cũ; lần đầu trả `undefined`.
- `useLocalStorage`: dùng lazy initializer đọc localStorage 1 lần, ghi lại trong `useEffect` khi value đổi; bọc try/catch (quota/bị chặn).
- Bẫy SSR (Next.js): `localStorage` không có trên server -> khởi tạo bằng `initial` rồi đọc trong `useEffect`, hoặc chỉ render ở client để tránh **hydration mismatch**.

```tsx
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();
  useEffect(() => { ref.current = value; }, [value]);
  return ref.current;
}
```

</details>

---

### 12.7. Triển khai hàm `throttle(fn, wait)`. Khác `debounce` ở chỗ nào và khi nào nên dùng?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Throttle nghĩa là dù hàm bị gọi liên tục bao nhiêu lần, nó chỉ thực thi tối đa một lần trong mỗi khoảng wait mili-giây — chạy đều đặn, giãn cách nhau. Điểm khác cốt lõi với debounce: debounce đợi tới khi người dùng NGỪNG thao tác mới chạy một lần cuối, còn throttle thì chạy đều trong suốt quá trình. Vì vậy throttle hợp với scroll, mousemove, resize — những sự kiện bắn liên tục mà mình muốn phản hồi mượt theo thời gian thực; còn debounce hợp với ô search, autosave — chờ người dùng gõ xong mới gọi API. Một lưu ý khi dùng trong React là phải bọc hàm throttle bằng useMemo hoặc useRef, nếu không mỗi lần render sẽ tạo lại closure mới và biến đếm thời gian bị reset, làm throttle mất tác dụng."

---

- `throttle`: `fn` chạy **tối đa 1 lần mỗi `wait`ms** dù gọi bao nhiêu lần; hợp `scroll`/`mousemove`/`resize`.
- Debounce chạy *sau khi ngừng* (1 lần cuối); throttle chạy *đều đặn* (nhiều lần, giãn đều).
- Trong React: bọc hàm throttle/debounce trong `useMemo`/`useRef` để **không tạo lại mỗi render** (nếu không sẽ reset closure, mất tác dụng).

```ts
function throttle<T extends (...a: any[]) => void>(fn: T, wait: number) {
  let last = 0, timer: ReturnType<typeof setTimeout> | null = null;
  return function (this: any, ...args: Parameters<T>) {
    const now = Date.now(), remaining = wait - (now - last);
    if (remaining <= 0) {            // leading edge
      if (timer) { clearTimeout(timer); timer = null; }
      last = now; fn.apply(this, args);
    } else if (!timer) {             // trailing edge
      timer = setTimeout(() => { last = Date.now(); timer = null; fn.apply(this, args); }, remaining);
    }
  };
}
```

</details>

---

### 12.8. Triển khai hàm `memoize(fn)` để cache kết quả theo tham số. Liên hệ với `useMemo`/`React.memo`.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Memoize là kỹ thuật cache kết quả của một hàm theo tham số đầu vào: lần đầu gọi thì tính và lưu vào một Map với key là tham số, lần sau gọi cùng tham số thì trả thẳng kết quả đã cache, khỏi tính lại — chỉ hợp với hàm thuần (cùng input ra cùng output). Liên hệ sang React thì useMemo cũng là memoize nhưng cache đúng một giá trị của lần render và so deps bằng Object.is; React.memo thì memoize cả component để bỏ qua re-render khi props không đổi; useCallback thì cache tham chiếu của hàm. Cái bẫy hay gặp là dùng JSON.stringify(args) làm key — nó không xử lý được function hay undefined, phụ thuộc thứ tự key, và tốn chi phí với object lớn. Ngoài ra cache mà không giới hạn thì rò rỉ bộ nhớ, nên ở production thường phải dùng LRU hoặc WeakMap."

---

- `memoize`: cache kết quả theo tham số (key = tham số đầu hoặc `JSON.stringify(args)`); gọi lại cùng tham số thì trả cache.
- Liên hệ React: `useMemo(fn, deps)` cache 1 giá trị render (so `deps` bằng `Object.is`); `React.memo` bỏ qua re-render (so nông props); `useCallback` cache tham chiếu hàm.
- Bẫy: `JSON.stringify` không xử lý `function`/`undefined`/thứ tự key, tốn chi phí (object nên cân nhắc `WeakMap`); cache không giới hạn -> rò rỉ bộ nhớ, production cần LRU.

```ts
function memoize<A extends any[], R>(fn: (...a: A) => R) {
  const cache = new Map<string, R>();
  return (...args: A): R => {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key)!;
    const r = fn(...args); cache.set(key, r); return r;
  };
}
```

</details>

---

### 12.9. Viết hook `useInfiniteScroll` dùng `IntersectionObserver` để tải thêm dữ liệu khi cuộn tới cuối danh sách.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Ý tưởng là đặt một phần tử 'sentinel' (thẻ trống) ở cuối danh sách, rồi dùng IntersectionObserver để theo dõi: khi sentinel lọt vào viewport tức là người dùng đã cuộn tới cuối, mình gọi loadMore để tải thêm. Cách này hơn hẳn việc nghe sự kiện scroll rồi tự tính toạ độ — IntersectionObserver chạy ngoài luồng chính, tự throttle, hiệu năng tốt và đỡ phải tính scrollTop thủ công. Một mẹo quan trọng là dùng callback ref thay vì useRef thuần, để observer gắn lại đúng thời điểm node sentinel xuất hiện hoặc thay đổi. Các bẫy cần tránh: quên disconnect observer cũ sẽ gây rò rỉ và gọi loadMore trùng; không chặn lại khi đang loading hoặc hết dữ liệu; và loadMore từ component cha phải bọc useCallback cho ổn định, nếu không effect chạy lại liên tục."

---

- Đặt phần tử "sentinel" ở cuối danh sách; `IntersectionObserver` báo khi nó vào viewport -> gọi `loadMore`. Ưu điểm so với nghe `scroll`: không tính vị trí thủ công, hiệu năng tốt, tự throttle.
- Mẹo: dùng **callback ref** (không phải `useRef` thuần) để observer gắn lại đúng lúc node xuất hiện/đổi.
- Bẫy: quên `disconnect` observer cũ (rò rỉ, gọi trùng); không chặn khi `loading`; `loadMore` không ổn định -> bọc `useCallback` ở cha.

```tsx
const sentinelRef = useCallback((node: HTMLElement | null) => {
  if (loading) return;
  observer.current?.disconnect();
  if (!node) return;
  observer.current = new IntersectionObserver(([e]) => {
    if (e.isIntersecting && hasMore) loadMore();
  });
  observer.current.observe(node);
}, [loading, hasMore, loadMore]);
```

</details>

---

### 12.10. Xây dựng component `Tabs` tái sử dụng (controlled hoặc uncontrolled). Làm sao để API gọn và dễ mở rộng?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Mình thường dựng Tabs theo mẫu compound components kết hợp Context: component Tabs cha giữ state tab đang chọn và chia sẻ qua Context, còn Tab và TabPanel tự đọc Context để biết mình có đang active không. Cách này làm API khai báo rất tự nhiên — người dùng chỉ lồng <Tabs><Tab/><Tab/></Tabs> mà không phải truyền hàng loạt props lằng nhằng xuyên nhiều tầng. Để dễ mở rộng, mình cho component chạy được cả uncontrolled (tự quản state nội bộ) lẫn controlled (nhận thêm value/onChange — nếu cha truyền vào thì dùng, không thì tự quản); đây là pattern chuẩn để component vừa dùng nhanh vừa kiểm soát được khi cần. Quan trọng nữa là accessibility: gắn đúng role tablist/tab/tabpanel, aria-selected và hỗ trợ điều hướng phím mũi tên; còn trong dự án thật thì mình hay tựa vào Radix UI hoặc Headless UI để khỏi tự lo phần a11y phức tạp."

---

- Mẫu tốt: **compound components** + Context — `<Tabs>` giữ state tab đang chọn và chia sẻ qua Context; `<Tab>`/`<TabPanel>` tự đọc Context. API khai báo tự nhiên, không truyền props lằng nhằng.
- Mở rộng controlled: nhận thêm `value`/`onChange` (có thì dùng, không thì tự quản uncontrolled).
- Accessibility: `role="tablist"/"tab"/"tabpanel"`, `aria-selected`, điều hướng mũi tên; production có thể dùng Radix UI / Headless UI.

```tsx
const TabsContext = createContext<{ active: string; setActive: (v: string) => void } | null>(null);

function Tab({ value, children }) {
  const { active, setActive } = useTabs();
  return <button role="tab" aria-selected={active === value}
    onClick={() => setActive(value)}>{children}</button>;
}
Tabs.Tab = Tab; Tabs.Panel = TabPanel;
```

</details>

---

## 13. Tư duy và Giải quyết vấn đề

### 13.1. Bạn tiếp cận một bug khó tái hiện (heisenbug) như thế nào, ví dụ "đôi khi danh sách bị render trùng item"?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Với bug khó tái hiện, nguyên tắc của mình là không sửa lung tung theo cảm tính mà đi theo phương pháp. Đầu tiên mình cố thu hẹp điều kiện để ép bug lộ ra — ví dụ render trùng item thường do race condition, nên mình bật Network throttling để làm response về chậm và không theo thứ tự, từ đó tái hiện được. Tiếp theo là tăng observability: thêm log có timestamp, dùng React DevTools Profiler xem component nào re-render bất thường. Rồi mình cô lập từng biến, dựng một minimal reproduction, đặt giả thuyết và kiểm chứng từng cái một. Với bài render trùng item thì hai thủ phạm kinh điển là dùng index làm key, và response cũ về sau response mới rồi ghi đè — cái sau mình fix bằng AbortController huỷ request cũ khi query đổi. Điều mình luôn nhắc là sửa nguyên nhân gốc chứ không sửa triệu chứng, và nhớ Strict Mode ở dev chạy effect hai lần để khỏi tưởng nhầm là bug."

---

- Nguyên nhân hay gặp: race condition, thứ tự response không xác định, state rò rỉ giữa các lần render.
- **Thu hẹp điều kiện**: ghi lại môi trường, bật Network throttling để ép race condition lộ ra.
- **Tăng observability**: log có timestamp, dùng React DevTools Profiler xem component re-render bất thường.
- **Cô lập biến**: dựng minimal reproduction, hình thành giả thuyết rồi kiểm chứng từng cái — không sửa lung tung.
- Hai thủ phạm kinh điển trong React: dùng index làm `key`; race condition khi response cũ về sau response mới.

```tsx
// FIX race condition: dùng AbortController huỷ request cũ khi query đổi
useEffect(() => {
  const controller = new AbortController();
  fetch(`/api/items?q=${query}`, { signal: controller.signal })
    .then((r) => r.json())
    .then((data) => setItems(data))
    .catch((e) => { if (e.name !== 'AbortError') throw e; });
  return () => controller.abort();
}, [query]);
```

**Bẫy**: sửa triệu chứng thay vì nguyên nhân gốc; quên Strict Mode (dev) chạy effect 2 lần làm tưởng có bug.

</details>

---

### 13.2. Khi đứng trước một quyết định kỹ thuật (ví dụ chọn cách quản lý state), bạn cân nhắc trade-off ra sao và làm sao để quyết định không bị "cảm tính"?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Để quyết định không bị cảm tính, mình bắt đầu bằng việc xác định đúng bài toán thật sự là gì, vì rất nhiều tranh cãi về state management thực chất là nhầm lẫn giữa 'đồng bộ server state' và 'quản lý UI state nội bộ' — hai thứ hoàn toàn khác nhau. Sau đó mình liệt kê các phương án và một bộ tiêu chí rõ ràng: độ phức tạp, đường cong học, hiệu năng, khả năng test, bảo trì, bundle size, và độ phù hợp với team. Mình chấm điểm từng phương án theo từng tiêu chí và ghi rõ trade-off chứ không kết luận chung chung kiểu 'cái này tốt hơn'. Một yếu tố mình rất coi trọng là chi phí đảo ngược — chọn framework hay kiến trúc khó sửa thì phải cân nhắc kỹ hơn nhiều so với chọn một thư viện nhỏ. Ví dụ cụ thể: server state mình dùng TanStack Query, UI state cục bộ thì useState/useReducer, global nhẹ thì Zustand, còn dự án lớn cần trace rõ thì Redux Toolkit. Bẫy hay gặp là nhét server state vào Redux rồi tự viết lại cache, hoặc chọn vì nó đang 'hot'."

---

- **Xác định bài toán thật**: "đồng bộ server state" khác "quản lý UI state nội bộ" — nhiều tranh cãi sai vì nhầm hai loại.
- **Liệt kê phương án + tiêu chí**: độ phức tạp, đường cong học, hiệu năng, khả năng test, bảo trì, bundle size, phù hợp team.
- **Đánh giá theo từng tiêu chí**, ghi rõ trade-off thay vì kết luận chung chung.
- **Cân nhắc chi phí đảo ngược**: quyết định khó đảo (kiến trúc, framework) thì cân nhắc kỹ.

| Nhu cầu | Lựa chọn | Lý do |
|---|---|---|
| Dữ liệu server (cache, refetch) | TanStack Query | Có cache/invalidation sẵn |
| UI state cục bộ | `useState`/`useReducer` | Đơn giản, không thêm phụ thuộc |
| Global nhẹ, ít boilerplate | Zustand | API gọn, không cần Provider |
| Global phức tạp, team lớn | Redux Toolkit | Chuẩn hóa, dễ trace, hệ sinh thái lớn |

**Bẫy**: nhét server state vào Redux rồi tự viết lại cache/loading; chọn vì "hot"; over-engineer cho team nhỏ.

</details>

---

### 13.3. Bạn được giao maintain một codebase React lớn mà chưa từng đụng tới. Bạn làm gì trong vài ngày đầu để hiểu nó?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Việc đầu tiên mình làm là chạy được app lên đã, vì không chạy được thì mọi thứ chỉ là đoán. Mình đọc README và package.json để nắm scripts và dependencies, từ đó đoán ra hệ công nghệ. Tiếp theo mình tìm entry point — Next.js thì là thư mục app/ hoặc pages/, còn SPA thì là main.tsx rồi lần theo router. Nhưng cách hiệu quả nhất để hiểu kiến trúc thật là đi trọn một luồng người dùng, ví dụ luồng login từ UI gọi API tới state rồi render ra sao — đi một lượt như vậy mình hiểu được cách các tầng nối với nhau hơn là đọc rời rạc. Mình cũng tận dụng git log và git blame để biết phần nào hay bị sửa, vì những chỗ đó thường là phần quan trọng hoặc dễ vỡ; và đọc test vì test mô tả hành vi mong đợi chính xác hơn comment. Bẫy cần tránh là cố đọc hết mọi file ngay từ đầu, và refactor sớm khi chưa hiểu vì sao code lại viết như vậy — kiểu Chesterton's Fence."

---

- **Chạy được app trước**: đọc `README`, `package.json` (scripts, dependencies) để đoán hệ công nghệ.
- **Tìm entry point**: Next.js là `app/`/`pages/`; SPA là `main.tsx` rồi lần theo router.
- **Đi theo một luồng người dùng** (ví dụ login: UI → API → state → render) — cách nhanh nhất hiểu kiến trúc thật.
- **Dùng công cụ**: `git log`/`blame` biết phần hay đổi (thường quan trọng/dễ vỡ); dựa vào type TS để lần theo.
- **Đọc test**: mô tả hành vi mong đợi tốt hơn comment.

```text
// Next.js App Router: nhìn cấu trúc thư mục là hiểu routing + layout
app/layout.tsx            -> layout gốc (provider, theme)
app/dashboard/[id]/page.tsx -> /dashboard/:id (dynamic segment)
```

**Bẫy**: cố đọc hết mọi file ngay; refactor sớm khi chưa hiểu vì sao code viết vậy (Chesterton's Fence); ngại hỏi đồng đội.

</details>

---

### 13.4. Khi nào KHÔNG nên tối ưu sớm (premature optimization)? Cho một ví dụ cụ thể trong React.

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Nguyên tắc của mình là đo trước, tối ưu sau — không nên tối ưu khi chưa có bằng chứng là chỗ đó thật sự chậm, vì tối ưu mù làm code phức tạp, khó đọc, dễ sinh bug mà chẳng nhanh hơn. Mình chỉ tối ưu khi thoả hai điều: có số đo cụ thể (Profiler, Lighthouse, FPS tụt), và chỗ đó nằm trên hot path tức là chạy nhiều. Lỗi kinh điển trong React là rải useMemo, useCallback, React.memo khắp nơi cho chắc; ví dụ bọc useMemo cho một phép nối chuỗi fullName thì vô nghĩa, vì chi phí so sánh deps còn lớn hơn cả việc tính lại; hay useCallback một handler mà chẳng truyền xuống component nào được memo thì cũng thừa. Chúng chỉ đáng dùng khi truyền xuống component bọc React.memo, làm dependency cho hook khác, hoặc tính toán thật sự nặng. Với React Compiler ở React 19 tự động hoá memo, thì việc memo thủ công bừa bãi càng trở nên dư thừa."

---

- Nguyên tắc: **đo trước, tối ưu sau**. Tối ưu chưa có bằng chứng làm code phức tạp, khó đọc, dễ bug mà vô ích.
- Chỉ tối ưu khi: (1) có số đo cụ thể (Profiler, Lighthouse, FPS thấp), và (2) nằm trên hot path.
- Lỗi kinh điển trong React: rải `useMemo`/`useCallback`/`React.memo` khắp nơi "cho chắc".

```tsx
// THỪA: phép tính rẻ, chi phí so sánh deps còn lớn hơn tính lại
const fullName = useMemo(() => `${first} ${last}`, [first, last]);
// THỪA: useCallback vô nghĩa nếu không truyền xuống component đã memo
const handleClick = useCallback(() => setOpen(true), []);
```

- Chỉ hợp lý khi: truyền xuống component bọc `React.memo`, làm dependency của hook khác, hoặc tính toán thật nặng.
- React Compiler (React 19) tự động hóa memo, càng làm memo thủ công bừa bãi thành dư thừa.

**Bẫy**: nhầm "trông tối ưu" với "chạy nhanh"; tối ưu chỗ chạy 1 lần/phút mà bỏ list 10.000 dòng không virtualize.

</details>

---

### 13.5. Bạn dựa vào những tiêu chí nào để chọn một thư viện cho dự án (ví dụ chọn thư viện form, hoặc thư viện UI)?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Khi chọn thư viện mình không chạy theo trend mà cân nhắc vài yếu tố cốt lõi. Đầu tiên là nó có giải đúng bài toán của mình không và API có dễ dùng không, vì một thư viện mạnh nhưng API rối thì cả team sẽ khổ. Thứ hai là sức khỏe dự án: commit có đều không, issue có người trả lời không, release gần đây ra sao, để tránh chọn phải thứ sắp bị bỏ rơi. Tiếp theo mình nhìn bundle size và khả năng tree-shake, vì frontend rất nhạy với JS tải về; mình không muốn kéo cả thư viện nặng chỉ để xài một hàm. Với dự án Next.js thì mình bắt buộc kiểm tra tính tương thích RSC/SSR: có hỗ trợ TypeScript không, có đụng tới window khi render trên server không, có cần 'use client' không. Ví dụ chọn thư viện form mình ưu tiên React Hook Form vì nó uncontrolled nên ít re-render, ghép với Zod để validate type-safe từ một nguồn schema duy nhất. Cuối cùng mình luôn nghĩ tới chi phí rời bỏ: nếu mai này phải bỏ thì gỡ ra có khó không."

---

- **Phù hợp bài toán & API**: giải đúng vấn đề mình có, API dễ dùng.
- **Sức khỏe dự án**: tần suất commit, issue được trả lời, lịch sử release, có bị bỏ rơi không.
- **Cộng đồng & tài liệu**: nhiều người dùng → dễ tìm giải pháp; tài liệu tốt → giảm chi phí học.
- **Bundle size**: tra bundlephobia, có tree-shakeable không.
- **Tương thích**: hỗ trợ TypeScript? Chạy với RSC/SSR Next.js không (thiếu `'use client'` hay đụng `window`)?
- **Chi phí rời bỏ (lock-in)**.

```tsx
// React Hook Form (ít re-render) + Zod resolver -> validate type-safe, một nguồn sự thật
const schema = z.object({ email: z.string().email(), age: z.coerce.number().min(18) });
type FormValues = z.infer<typeof schema>;
const { register, handleSubmit } = useForm<FormValues>({ resolver: zodResolver(schema) });
```

**Bẫy**: chọn theo trend không kiểm tra maintain; kéo thư viện nặng chỉ để dùng 1 hàm; quên kiểm tra SSR/RSC.

</details>

---

### 13.6. Một task được ước lượng "2 ngày" hóa ra mất 5 ngày. Bạn ước lượng thời gian một task frontend như thế nào để chính xác hơn và xử lý khi sắp trễ ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Ước lượng sai thường vì mình chỉ tính thời gian gõ code trong điều kiện lý tưởng mà quên hết phần ẩn. Cách mình làm chính xác hơn là chia task tới mức đo được, cái gì lớn hơn nửa ngày là mình tách nhỏ ra, vì task càng to thì sai số càng lớn. Khi tách mình cố ý liệt kê những thứ hay quên: loading/empty/error state, responsive, accessibility, viết test, review và sửa sau review, rồi tích hợp với API thật chứ không phải mock. Ví dụ một 'trang danh sách user' nghe thì như một ngày code bảng, nhưng cộng đủ tìm kiếm có debounce, đồng bộ URL, responsive và test thì thực tế là hai ngày rưỡi. Với phần mình chưa từng làm thì mình tách riêng một spike có giới hạn thời gian để gỡ unknown trước. Mình cũng quen đưa một dải thay vì một con số, kiểu '2 đến 4 ngày tùy API đã sẵn chưa', vì như vậy trung thực hơn. Và quan trọng nhất, nếu thấy sắp trễ thì mình báo sớm kèm phương án, ví dụ cắt phần phụ hay giảm scope, chứ không giấu tới sát deadline."

---

Sai vì chỉ tính thời gian code "lý tưởng" mà bỏ phần ẩn. Cách cải thiện:

- **Chia nhỏ tới mức đo được**: task lớn hơn nửa ngày nên tách ra.
- **Tính công việc ẩn**: edge case, loading/empty/error, responsive, a11y, test, review, sửa sau review, tích hợp API thật.
- **Nhận diện unknowns**: phần chưa từng làm thì tách một "spike" (nghiên cứu có giới hạn thời gian).
- **Đưa dải thay vì một số**: "2-4 ngày tùy API có sẵn" trung thực hơn "2 ngày".
- **Sắp trễ thì báo SỚM**, kèm lựa chọn: giảm phạm vi, cắt phần phụ, hay xin thêm thời gian.

```text
Phân rã "Trang danh sách user":
- UI bảng + phân trang ............... 0.5 ngày
- Tích hợp API + loading/error/empty . 0.5 ngày
- Tìm kiếm + lọc (debounce, sync URL) . 0.5 ngày   <- hay quên
- Responsive + a11y .................. 0.5 ngày   <- hay quên
- Test + sửa sau review .............. 0.5 ngày   <- hay quên
=> ~2.5 ngày, không phải "1 ngày code bảng"
```

**Bẫy**: ước lượng "thời gian gõ code" thay vì "thời gian đến khi merge"; quên buffer; giấu việc trễ tới sát deadline.

</details>

---

### 13.7. Khi gặp một bài toán lớn (ví dụ "xây dựng tính năng editor giàu định dạng có cộng tác real-time"), bạn dùng kỹ thuật gì để chia nhỏ nó?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Với bài toán lớn mình tránh chia ngang theo tầng kỹ thuật rồi cuối cùng mới ráp lại, vì như vậy rất lâu mới có thứ chạy được để demo. Mình ưu tiên lát cắt dọc: làm một luồng end-to-end nhỏ nhưng thật sự chạy, ví dụ với editor thì gõ được rồi lưu rồi hiển thị lại, sau đó mới mở rộng dần. Song song với đó mình tách theo rủi ro, tức là phần khó và mới nhất thường là đồng bộ real-time thì mình làm một prototype hay spike trước để xác nhận khả thi, gỡ rủi ro sớm thay vì để nó nổ lúc sát deadline. Mình cũng phân tách theo ranh giới kỹ thuật rõ ràng: tầng render, tầng mô hình dữ liệu của tài liệu, và tầng đồng bộ qua transport hoặc CRDT, để mỗi phần test được độc lập. Trước khi code mình xác định đâu là MVP và đâu là cái cắt được; ví dụ bản đầu chỉ cần bold/italic và lưu, còn cộng tác nhiều người là pha sau. Cuối cùng mình vẽ sơ đồ phụ thuộc để biết thứ tự làm và phần nào chạy song song được, và chia PR vừa phải để review được."

---

- **Lát cắt dọc (vertical slice)**: làm một luồng end-to-end nhỏ nhưng chạy được (gõ → lưu → hiển thị), rồi mở rộng.
- **Tách theo rủi ro**: phần khó/mới nhất (đồng bộ real-time) làm prototype/spike trước để gỡ rủi ro sớm.
- **Tách theo ranh giới kỹ thuật**: tầng render, tầng dữ liệu (mô hình tài liệu), tầng đồng bộ (transport/CRDT) — test độc lập.
- **Xác định MVP và cái cắt được**: bản đầu chỉ cần bold/italic + lưu; cộng tác là pha sau.
- **Vẽ phụ thuộc** để biết thứ tự làm và phần nào song song được.

```text
"Editor cộng tác real-time" tách thành:
Pha 0 (spike): thử CRDT/OT trên prototype -> xác nhận khả thi
Pha 1: editor 1 người (bold/italic), lưu/đọc  <- chạy được end-to-end
Pha 2: undo/redo, autosave
Pha 3: con trỏ + nhiều người cùng sửa (real-time)
Pha 4: xung đột, offline, quyền chỉnh sửa
```

**Bẫy**: bắt đầu từ phần dễ/vui (UI) thay vì phần rủi ro cao; chia ngang nên không demo được lâu; PR khổng lồ khó review.

</details>

---

### 13.8. Bạn cân bằng giữa trải nghiệm người dùng (UX), hiệu năng và trải nghiệm lập trình (DX) như thế nào khi chúng mâu thuẫn?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Mặc định của mình là UX thắng, vì người dùng chỉ quan tâm app nhanh, mượt và đúng kỳ vọng chứ không quan tâm code mình đẹp tới đâu. Hiệu năng mình coi là công cụ phục vụ UX, nên mình tập trung tối ưu cái ảnh hưởng tới cảm nhận như LCP, INP, không giật khi cuộn, chứ không vi tối ưu những thứ người dùng chẳng bao giờ thấy. DX thì mình xem là khoản đầu tư dài hạn: code dễ đọc thì ít bug và ship nhanh hơn, nên về lâu dài nó cũng phục vụ UX; chỉ khi xung đột thật sự thì mình hy sinh DX nhưng ghi rõ lý do để người sau hiểu. Ví dụ điển hình là optimistic update: nó cho người dùng cảm giác tức thì khi bấm like, đổi lại mình phải viết thêm logic rollback khi lỗi, tức là DX phức tạp hơn nhưng mình chấp nhận vì lợi UX rõ ràng. Điểm mấu chốt là mình quyết định dựa trên đo đạc chứ không đoán, đo trước rồi mới chọn đánh đổi."

---

- **Mặc định ưu tiên UX**: người dùng quan tâm app nhanh, mượt, đúng kỳ vọng — không phải code đẹp.
- **Hiệu năng phục vụ UX**: tối ưu phần ảnh hưởng cảm nhận (LCP, INP, không giật khi cuộn), không vi tối ưu vô hình.
- **DX là đầu tư dài hạn**: code dễ đọc → ít bug, ship nhanh → gián tiếp tốt cho UX. Khi xung đột thật thì UX thắng, nhưng ghi rõ lý do.
- **Quyết định dựa trên dữ liệu**: đo trước, đừng đoán.

| Lựa chọn | Lợi UX | Cái giá |
|---|---|---|
| Optimistic update | Cảm giác tức thì | Phải xử lý rollback khi lỗi |
| Skeleton thay spinner | Cảm giác nhanh, giảm CLS | Tốn công bảo trì nhiều biến thể |
| Virtualize list lớn | Cuộn mượt | Code khó, phá Ctrl+F, khó a11y |

```tsx
// Optimistic update (TanStack Query): ưu tiên UX, chấp nhận DX phức tạp vì phải rollback
useMutation({
  mutationFn: toggleLike,
  onMutate: async (postId) => {
    await queryClient.cancelQueries({ queryKey: ['post', postId] });
    const prev = queryClient.getQueryData(['post', postId]);
    queryClient.setQueryData(['post', postId], (old: any) => ({ ...old, liked: !old.liked }));
    return { prev };
  },
  onError: (_e, postId, ctx) => queryClient.setQueryData(['post', postId], ctx?.prev),
  onSettled: (_d, _e, postId) => queryClient.invalidateQueries({ queryKey: ['post', postId] }),
});
```

**Bẫy**: hy sinh UX để code "sạch" theo ý mình; viết code không ai bảo trì nổi nhân danh "vì người dùng"; vi tối ưu thứ người dùng không cảm nhận trong khi bỏ qua INP/LCP thực tế.

</details>

## ✅ Checklist ôn nhanh trước phỏng vấn

- [ ] Giải thích trôi chảy: event loop, closure, `this`, tính bất biến của state.
- [ ] Hooks: dependency array & cleanup của `useEffect`, `useMemo`/`useCallback`, viết được custom hook.
- [ ] Render & tối ưu: reconciliation, `key`, `React.memo`, virtualization cho danh sách lớn.
- [ ] State: phân biệt server state vs client state; thành thạo TanStack Query (React Query).
- [ ] Next.js: CSR/SSR/SSG/ISR, hydration mismatch, Server vs Client Components, Server Actions.
- [ ] Thư viện: React Router, React Hook Form + Zod, styling (Tailwind/CSS-in-JS).
- [ ] Performance: code splitting, lazy load, Core Web Vitals.
- [ ] System design: feature-based, component library, scale bằng CDN/cache.
- [ ] Coding: tự tin viết debounce, deepClone, Dropdown, custom hook mà không cần tra.
- [ ] Behavioral: chuẩn bị 4–5 câu chuyện theo cấu trúc **STAR**.

---

> 📌 *Hãy gắn câu trả lời với ví dụ từ chính dự án của bạn — câu trả lời gắn với kinh nghiệm thật luôn ấn tượng hơn lý thuyết suông.*
> Chúc bạn phỏng vấn thành công! 🚀
