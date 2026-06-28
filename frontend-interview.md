# 🎯 Bộ câu hỏi phỏng vấn Frontend (React / Next.js)

> Tài liệu ôn tập cho vị trí **Frontend Developer**, trọng tâm **React / ReactJS, Next.js** và hệ sinh thái (React Router, TanStack Query, Redux Toolkit, Zustand, React Hook Form, Tailwind, Testing Library...).
> Câu hỏi được nhóm theo chủ đề; mỗi đáp án nằm trong khối **collapse** (`<details>`) — bấm để xem lời giải + ví dụ.
>
> 💡 *Mẹo ôn tập:* Tự trả lời trước khi mở đáp án, rồi diễn đạt lại bằng lời của bạn — phỏng vấn kiểm tra khả năng **giải thích**, không phải đọc thuộc.

## 📚 Mục lục

1. [JavaScript và TypeScript cốt lõi cho React](#1-javascript-và-typescript-cốt-lõi-cho-react)
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

## 1. JavaScript và TypeScript cốt lõi cho React

### 1.1. Hãy giải thích event loop trong JavaScript. Sự khác nhau giữa microtask và macrotask là gì, và điều đó ảnh hưởng thế nào tới thứ tự chạy code trong một component React?

<details>
<summary>👉 Xem câu trả lời</summary>

- JS đơn luồng; event loop kiểm tra call stack, khi rỗng thì lấy việc từ các hàng đợi ra chạy.
- Macrotask: `setTimeout`, `setInterval`, I/O, sự kiện DOM, `MessageChannel`.
- Microtask: `Promise.then/catch/finally`, `queueMicrotask`, code sau `await`, `MutationObserver`.
- Quy tắc vàng: sau mỗi macrotask, rút cạn TOÀN BỘ microtask rồi mới render và lấy macrotask kế tiếp → microtask luôn chạy trước.
- Liên hệ React 18: nhiều `setState` trong một handler được batch, chỉ re-render một lần.
- Bẫy: tưởng `setTimeout(fn, 0)` chạy ngay — thực ra luôn xếp sau mọi microtask đang chờ.

```js
console.log('1: sync');
setTimeout(() => console.log('2: macrotask'), 0);
Promise.resolve().then(() => console.log('3: microtask'));
console.log('4: sync');
// Thứ tự: 1 -> 4 -> 3 -> 2
```

</details>

---

### 1.2. Closure là gì? Cho một ví dụ closure gây bug trong React và cách khắc phục.

<details>
<summary>👉 Xem câu trả lời</summary>

- Closure: hàm "ghi nhớ" và truy cập biến ở scope nơi nó được khai báo, kể cả khi scope gốc đã kết thúc.
- Bug phổ biến trong React: "stale closure" — closure ôm giá trị cũ trong `useEffect`/`setInterval` (thường do thiếu dependency).
- Khắc phục: dùng functional update (`setCount(prev => prev + 1)`) để không phụ thuộc giá trị bị đóng băng.
- Tích cực: factory có state riêng, memoization, giữ tham chiếu ổn định. Rule `exhaustive-deps` cảnh báo việc này.

```jsx
// SAI: count bị đóng băng = 0, mãi set thành 1
useEffect(() => {
  const id = setInterval(() => setCount(count + 1), 1000);
  return () => clearInterval(id);
}, []);

// ĐÚNG: prev luôn mới nhất
useEffect(() => {
  const id = setInterval(() => setCount(prev => prev + 1), 1000);
  return () => clearInterval(id);
}, []);
```

</details>

---

### 1.3. `this` hoạt động thế nào trong JavaScript, và tại sao arrow function lại được ưa dùng trong React?

<details>
<summary>👉 Xem câu trả lời</summary>

- Hàm thường: `this` xác định lúc GỌI (call-site): `obj.fn()` → `obj`; `fn()` → `undefined`/`window`; `call/apply/bind` → giá trị truyền vào; `new` → instance mới.
- Arrow function: KHÔNG có `this` riêng, lấy theo lexical scope → không mất `this` khi truyền làm callback.
- Trong React: class component xưa phải bind thủ công hoặc dùng class field arrow; function component hiện đại gần như không còn vấn đề `this`.
- Bẫy: dùng arrow làm method trên object literal rồi mong `this` trỏ object — không, arrow lấy `this` từ scope ngoài.

```jsx
class Old extends React.Component {
  handleClick = () => { console.log(this); }; // luôn đúng instance nhờ lexical this
  render() { return <button onClick={this.handleClick}>Click</button>; }
}
```

</details>

---

### 1.4. Phân biệt chạy nhiều tác vụ async TUẦN TỰ và SONG SONG với `async/await`. Khi nào dùng cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- `await` DỪNG hàm tới khi Promise xong. `await` từng cái → tuần tự (chờ nhau). Khởi tạo hết Promise rồi `await Promise.all` → song song.
- Tuần tự: khi request sau PHỤ THUỘC kết quả trước (cần `userId` mới fetch đơn hàng).
- Song song: khi các tác vụ ĐỘC LẬP → giảm thời gian chờ.
- Liên hệ React/Next.js: gom fetch độc lập vào `Promise.all` tránh "waterfall". Bẫy: dùng `Promise.all` cho tác vụ phụ thuộc nhau sẽ lỗi.

```ts
// Tuần tự: ~600ms
const user = await fetchUser();
const posts = await fetchPosts();

// Song song: ~300ms
const [user, posts] = await Promise.all([fetchUser(), fetchPosts()]);
```

</details>

---

### 1.5. So sánh `Promise.all`, `Promise.allSettled`, `Promise.race` và `Promise.any`. Mỗi cái dùng trong tình huống nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- `Promise.all`: resolve khi TẤT CẢ resolve; reject ngay khi một cái fail (fail-fast). Dùng khi cần đủ mọi phần dữ liệu.
- `Promise.allSettled`: chờ TẤT CẢ settled, không bao giờ reject; trả `{status, value/reason}`. Dùng khi muốn biết kết quả từng cái, không để một lỗi sập cả nhóm.
- `Promise.race`: lấy cái settled ĐẦU TIÊN (resolve hoặc reject). Dùng làm timeout.
- `Promise.any`: lấy cái resolve ĐẦU TIÊN; reject khi tất cả fail (`AggregateError`). Dùng nhiều nguồn dự phòng.
- Bẫy: dùng `Promise.all` cho dashboard nhiều widget — một widget lỗi là cả trang trắng; nên dùng `allSettled`.

```ts
const data = await Promise.race([
  fetchData(),
  new Promise((_, rej) => setTimeout(() => rej(new Error('timeout')), 5000)),
]);
```

</details>

---

### 1.6. Tính bất biến (immutability) trong React là gì? Phân biệt shallow copy và deep copy.

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 1.9. Trong TypeScript, khi nào dùng `interface` và khi nào dùng `type`?

<details>
<summary>👉 Xem câu trả lời</summary>

- Cả hai mô tả shape dữ liệu, phần lớn thay thế được cho nhau với object.
- `interface`: hợp cho props/object công khai; mở rộng bằng `extends`; hỗ trợ declaration merging (gộp khai báo trùng tên).
- `type`: làm được union, intersection, tuple, primitive alias, mapped/conditional type; mở rộng bằng `&`.
- Quy tắc thực dụng: `interface` cho props, `type` cho union/tuple/kiểu tính toán — quan trọng nhất là nhất quán. Bẫy: declaration merging của `interface` đôi khi gây bất ngờ; `type` trùng tên báo lỗi (an toàn hơn).

```ts
interface ButtonProps { label: string; onClick: () => void; }
interface IconButtonProps extends ButtonProps { icon: string; }
type Status = 'idle' | 'loading' | 'success' | 'error';
type Point = [number, number];
```

</details>

---

### 1.10. Generics trong TypeScript là gì? Cho một ví dụ generic hữu ích trong ngữ cảnh React.

<details>
<summary>👉 Xem câu trả lời</summary>

- Generics cho phép viết hàm/kiểu/component dùng được với NHIỀU kiểu mà vẫn type-safe, thay vì `any`. Hiểu nôm na: "tham số kiểu" được điền lúc dùng.
- `T` được suy ra theo input, output giữ đúng kiểu.
- Có thể giới hạn bằng `extends` (constraint): `<T extends { id: string }>`. Component generic hữu ích cho danh sách tái sử dụng.
- Bẫy: lạm dụng làm signature khó đọc; `as Promise<T>` chỉ ép kiểu chứ không kiểm tra runtime → xác thực bằng Zod nếu dữ liệu không tin cậy.

```ts
function first<T>(arr: T[]): T | undefined { return arr[0]; }
const n = first([1, 2, 3]);   // number | undefined

// custom hook generic
function useFetch<T>(url: string): { data: T | null } { /* ... */ }
const { data } = useFetch<User>('/api/user'); // data: User | null
```

</details>

---

### 1.11. Utility types `Partial`, `Pick`, `Omit`, `Record` làm gì? Cho ví dụ thực tế khi gõ kiểu props.

<details>
<summary>👉 Xem câu trả lời</summary>

- `Partial<T>`: mọi thuộc tính thành optional (dùng cho hàm update). `Required<T>`: ngược lại.
- `Pick<T, K>`: lấy ra một số thuộc tính. `Omit<T, K>`: bỏ đi một số thuộc tính (vd type "tạo mới" bỏ `id`).
- `Record<K, V>`: object có key kiểu `K`, value kiểu `V` (lookup table trong Redux/Zustand).
- Bẫy: `Pick`/`Omit` chỉ làm ở cấp một, không đệ quy.

```ts
function updateUser(id: string, patch: Partial<User>) {}
type CreateUserInput = Omit<User, 'id'>;
type UsersById = Record<string, User>;

// mở rộng prop element DOM
type Props = Omit<React.ComponentProps<'button'>, 'onClick'> & {
  onClick: (id: string) => void;
};
```

</details>

---

### 1.12. Phân biệt `any`, `unknown` và `never` trong TypeScript. Khi nào dùng cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- `any`: tắt kiểm tra kiểu — tiện nhưng mất an toàn, nên tránh.
- `unknown`: "top type" an toàn — nhận mọi giá trị nhưng phải narrow trước khi dùng; thay thế an toàn cho `any`.
- `never`: "bottom type" — kiểu của thứ không bao giờ tồn tại (hàm luôn throw, nhánh bất khả thi); dùng cho exhaustiveness check.
- Liên hệ React: gõ JSON trả về là `unknown` rồi xác thực Zod; `catch (e)` mặc định `unknown`, phải kiểm `e instanceof Error`. Bẫy: `any` cho props lan ra cả cây component.

```ts
function handle(value: unknown) {
  if (typeof value === 'string') console.log(value.length); // OK sau narrow
}
function render(s: Status) {
  switch (s) {
    case 'idle': return '...';
    default: const _exhaustive: never = s; return _exhaustive; // quên nhánh -> lỗi biên dịch
  }
}
```

</details>

---

### 1.13. Bạn gõ kiểu cho props của một component và cho giá trị trả về của custom hook như thế nào? Có những lưu ý gì?

<details>
<summary>👉 Xem câu trả lời</summary>

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

- Virtual DOM là bản biểu diễn nhẹ của UI dưới dạng cây object JS (cây React element) giữ trong bộ nhớ.
- Cơ chế: state/props đổi → dựng cây mới → diff với cây cũ → áp **tập thay đổi tối thiểu** lên DOM thật (tránh reflow/repaint thừa).
- Bẫy hay hỏi: Virtual DOM **không tự làm app nhanh hơn** DOM thật tối ưu thủ công. Giá trị thật là **mô hình khai báo (declarative)**: ta mô tả UI theo state, React lo đồng bộ DOM và gom (batch) cập nhật.
- Từ React 16+ (kiến trúc Fiber), cập nhật có thể bị gián đoạn/ưu tiên (interruptible) để phục vụ concurrent.

</details>

---

### 2.3. Reconciliation và thuật toán diffing của React hoạt động thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Reconciliation**: quá trình đối chiếu cây element mới với cũ để quyết định cập nhật DOM. So sánh tổng quát là O(n³) nên React dùng heuristic gần O(n).
- Giả định 1: **khác `type` → coi như khác hoàn toàn** → hủy cả cây con cũ (state mất), dựng mới. Cùng `type` → giữ node DOM, chỉ cập nhật props đã đổi.
- Giả định 2: **`key`** giúp nhận diện phần tử ổn định trong danh sách.
- Bẫy: cùng `type` ở cùng vị trí thì state **không** reset (React tái dùng node). Muốn ép reset/remount → đổi `key`.

```jsx
<UserForm key={userId} userId={userId} /> // đổi userId -> remount, form reset
```

</details>

---

### 2.6. Controlled component và uncontrolled component khác nhau thế nào? Cho ví dụ.

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 2.8. Các giai đoạn vòng đời (lifecycle) của class component ánh xạ sang hooks như thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 2.10. Tại sao không được mutate (sửa trực tiếp) state trong React?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 2.11. Synthetic event trong React là gì? Nó khác native event ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

- `SyntheticEvent` là wrapper đa trình duyệt quanh native event → hành vi đồng nhất, API thống nhất (`e.target`, `e.preventDefault()`, `e.stopPropagation()`); native event truy cập qua `e.nativeEvent`.
- React **không gắn listener lên từng node** mà dùng **event delegation** gắn ở gốc. Từ **React 17+, listener gắn vào container gốc của app** (không phải `document` như React 16) → quan trọng khi chạy nhiều app React trên một trang.
- Tên handler camelCase: `onClick`, `onChange`, `onSubmit`.
- **React 17+ không còn event pooling** → không cần `e.persist()` nữa (đó là chuyện React 16).
- `onChange` của React bắn mỗi lần gõ (giống `input` của DOM), không phải khi blur như native `change`.

</details>

---

### 2.13. `children` là gì và component composition (kết hợp component) giải quyết vấn đề gì?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 3.8. Khi nào nên chọn `useReducer` thay vì `useState`?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 3.10. Custom hook là gì? Hãy viết một hook `useDebounce`.

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 3.11. Hãy viết các custom hook `usePrevious` và `useToggle`.

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 3.12. Rules of Hooks gồm những quy tắc nào? Tại sao bắt buộc phải tuân theo?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 3.13. `useLayoutEffect` khác `useEffect` ở điểm nào? Khi nào bắt buộc dùng `useLayoutEffect`?

<details>
<summary>👉 Xem câu trả lời</summary>

- Khác ở thời điểm chạy so với paint của trình duyệt.
- Dùng `useLayoutEffect` khi cần ĐỌC kích thước/vị trí DOM rồi điều chỉnh ngay, tránh nhấp nháy (ví dụ tính vị trí tooltip).
- Ưu tiên `useEffect` (không chặn paint); chỉ chuyển khi thấy flicker. Trong Next.js SSR, `useLayoutEffect` không chạy trên server và sẽ cảnh báo.

| | `useEffect` | `useLayoutEffect` |
| --- | --- | --- |
| Thời điểm | Bất đồng bộ, SAU paint | Đồng bộ, TRƯỚC paint |
| Chặn paint | Không | Có |

</details>

---

### 3.15. `useId` giải quyết vấn đề gì, đặc biệt trong ngữ cảnh SSR của Next.js?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 3.17. `useImperativeHandle` kết hợp `forwardRef` để làm gì? Cho ví dụ.

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 4.13. Kể về một lần bạn chẩn đoán và khắc phục một vấn đề hiệu năng React trong dự án thực tế.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

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

### 5.7. Trong TanStack Query, hãy giải thích `staleTime` và `gcTime` (trước đây là `cacheTime`). Chúng kiểm soát điều gì?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 5.8. `queryKey` đóng vai trò gì trong TanStack Query? Giải thích cơ chế invalidation và vì sao nó tốt hơn việc tự refetch thủ công.

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 5.9. Optimistic update là gì? Hãy mô tả luồng đúng để làm optimistic update an toàn (kể cả khi lỗi) trong TanStack Query.

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 5.10. Infinite query khác gì với phân trang thông thường trong TanStack Query? Khi nào dùng `useInfiniteQuery`?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 5.11. SWR là gì và nó khác TanStack Query ở điểm nào? Khi nào bạn cân nhắc SWR?

<details>
<summary>👉 Xem câu trả lời</summary>

- **SWR** (Vercel): thư viện data-fetching theo chiến lược **stale-while-revalidate** — trả cache cũ ngay rồi refetch nền cập nhật. Tên = tên chiến lược.
- So với TanStack Query: SWR **tối giản hơn** (`useSWR(key, fetcher)`); TanStack **mạnh hơn** (devtools, mutation API chi tiết, optimistic rõ ràng, query cancellation).
- Cân nhắc SWR: dự án **Next.js**, muốn tối giản, ít cấu hình, fetch đọc là chính. Mutation/optimistic phức tạp, devtools mạnh -> TanStack Query.
- Điểm chung: **cả hai đều là server state** — đừng tự cache thủ công trong Redux.

```tsx
const { data, error, isLoading } = useSWR('/api/user', fetcher);
```

</details>

---

### 5.12. "Chuẩn hóa dữ liệu" (data normalization) là gì? Khi nào cần và khi nào không?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 5.13. "Derived state" là gì, và vì sao việc lưu state phái sinh vào `useState` thường là một anti-pattern?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 5.14. Vì sao đôi khi nên lưu state trên URL thay vì trong React state? Trong Next.js bạn làm điều này thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 5.15. Hãy đưa ra "cây quyết định" của bạn: gặp một nhu cầu state, bạn chọn công cụ nào?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 5.16. Kể về một lần bạn refactor cách quản lý state trong một dự án. Vấn đề ban đầu là gì và bạn cải thiện ra sao?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Câu hỏi hành vi — dùng cấu trúc STAR. Mẫu trả lời điều chỉnh theo trải nghiệm thật:

- **Situation**: admin dashboard dùng Redux cho mọi thứ kể cả data API; mỗi màn tự viết `fetchStart/Success/Failure`, tự quản loading/cache. Code phình to, bug dữ liệu cũ vì hai màn dùng chung list nhưng không đồng bộ.
- **Task**: giảm boilerplate, xử lý triệt để dữ liệu lệch nhau, không phá tính năng đang chạy.
- **Action**: tách **server state** và **client state**. Đưa data API sang **TanStack Query**, dùng `queryKey` thống nhất + `invalidateQueries` sau mutation để mọi màn tự làm mới. Client state nhỏ còn lại (modal, wizard) chuyển sang **Zustand** với selector, bỏ Redux. Làm từng module, có test cho luồng quan trọng trước khi gỡ code cũ.
- **Result**: code quản lý data giảm đáng kể, hết bug dữ liệu cũ (chỉ còn một cơ chế invalidation), UX nhanh hơn nhờ caching/background refetch, đồng đội thêm màn mới nhanh hơn vì pattern rõ ràng.

Mẹo: nhấn vào **lý do kỹ thuật** (tách server/client state) và **kết quả đo được** (ít code, hết bug đồng bộ, nhanh hơn), thay vì chỉ "đổi thư viện".

</details>

## 6. React Ecosystem và Thư viện

### 6.3. `loader` và `action` trong React Router (data router) là gì? Chúng giải quyết vấn đề gì so với fetch trong `useEffect`?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 6.4. Cách triển khai một Protected Route (yêu cầu đăng nhập) trong React Router?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 6.5. Tại sao React Hook Form có hiệu năng tốt hơn so với cách quản lý form bằng state thông thường?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 6.6. Phân biệt `register` và `control` (Controller) trong React Hook Form. Khi nào dùng cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 6.7. Tích hợp Zod (hoặc Yup) với React Hook Form như thế nào? Ưu điểm của schema validation?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 6.8. So sánh các phương án styling: CSS Modules, styled-components/Emotion (CSS-in-JS), và Tailwind CSS. Trade-off của mỗi cách?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 6.11. So sánh thư viện component MUI, Radix UI và shadcn/ui. Triết lý "headless" và "copy-paste component" của shadcn khác gì?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 6.15. Khi chọn một thư viện cho dự án (vd state management hay form), bạn dựa trên những tiêu chí nào để quyết định?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

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

### 8.3. Phân biệt các giá trị của `position`: `static`, `relative`, `absolute`, `fixed`, `sticky`. `position: sticky` hoạt động dựa trên điều kiện nào?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 8.5. Semantic HTML là gì và vì sao nó quan trọng cho accessibility? Cho ví dụ tái cấu trúc một component React từ "div soup" sang semantic.

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 8.6. Bạn xây dựng responsive như thế nào? So sánh `media query` với `container query`, và giải thích vì sao nên ưu tiên `rem`/`em` cùng tư duy mobile-first.

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 8.8. Phân biệt Reflow (Layout) và Repaint. Loại nào tốn kém hơn, và bạn làm gì trong React để giảm thiểu reflow khi animation?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 8.9. Stacking context là gì? Vì sao đôi khi `z-index: 9999` vẫn không đưa được phần tử lên trên, và điều này liên quan gì tới modal/dropdown trong React?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 8.10. Bạn triển khai theming và dark mode bằng CSS Variables như thế nào? Vì sao biến CSS hợp với React/Next.js hơn là dùng nhiều class điều kiện, và làm sao tránh nhấp nháy (FOUC) khi tải trang?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 8.11. (Tình huống) Một đồng nghiệp báo rằng trang bị "cuộn ngang lạ" trên mobile và một tooltip bị che mất. Hãy kể về một lần bạn debug vấn đề CSS/layout khó và cách bạn tiếp cận.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

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

### 10.5. Monorepo cho frontend (Turborepo / Nx): giải quyết vấn đề gì? Cấu trúc ra sao và build/CI được tối ưu thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 10.7. Làm sao đảm bảo accessibility (a11y) ở quy mô lớn — không phải chỉ một trang mà toàn bộ ứng dụng, qua nhiều team và theo thời gian?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 10.8. Thiết kế chiến lược xử lý lỗi toàn cục cho một ứng dụng React/Next.js: error boundary, lỗi network qua interceptor, và báo cáo lỗi (observability).

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 10.9. "Performance budget" là gì? Bạn đặt budget thế nào và làm sao ép buộc nó trong quy trình phát triển/CI để không bị "trôi" theo thời gian?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 10.10. Khi thiết kế kiến trúc quản lý state cho một ứng dụng lớn, bạn phân loại state thế nào và chọn công cụ nào cho từng loại? (server state, client/UI state, form, URL)

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 11.7. E2E test là gì? So sánh Cypress và Playwright, và cho biết khi nào nên dùng E2E thay vì integration test với RTL.

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 11.8. Làm thế nào để test một custom hook trong React? Cho ví dụ với `renderHook`.

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 11.9. Khi test một component có trạng thái bất đồng bộ (loading → success / error), bạn xử lý thế nào? Vì sao `findBy*` và `waitFor` quan trọng?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 11.10. Snapshot testing là gì? Nêu ưu, nhược điểm và khi nào nên (hoặc không nên) dùng.

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 11.11. Bạn cấu hình môi trường test cho một dự án Next.js như thế nào (jsdom, setup file, mock những gì)? Những thứ đặc thù Next nào hay phải mock?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 11.12. Hãy kể về một lần bạn xử lý test "flaky" (lúc pass lúc fail) hoặc thiết lập chiến lược test cho một dự án frontend. (Câu hỏi hành vi)

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

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

### 12.10. Viết hook `useInfiniteScroll` dùng `IntersectionObserver` để tải thêm dữ liệu khi cuộn tới cuối danh sách.

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 12.11. Xây dựng component `Tabs` tái sử dụng (controlled hoặc uncontrolled). Làm sao để API gọn và dễ mở rộng?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 13.7. Một task được ước lượng "2 ngày" hóa ra mất 5 ngày. Bạn ước lượng thời gian một task frontend như thế nào để chính xác hơn và xử lý khi sắp trễ ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 13.8. Khi gặp một bài toán lớn (ví dụ "xây dựng tính năng editor giàu định dạng có cộng tác real-time"), bạn dùng kỹ thuật gì để chia nhỏ nó?

<details>
<summary>👉 Xem câu trả lời</summary>

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

### 13.9. Bạn cân bằng giữa trải nghiệm người dùng (UX), hiệu năng và trải nghiệm lập trình (DX) như thế nào khi chúng mâu thuẫn?

<details>
<summary>👉 Xem câu trả lời</summary>

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
