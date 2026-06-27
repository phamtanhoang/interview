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
14. [Kỹ năng mềm và Câu hỏi hành vi](#14-kỹ-năng-mềm-và-câu-hỏi-hành-vi)

---
## 1. JavaScript và TypeScript cốt lõi cho React

### 1.1. Hãy giải thích event loop trong JavaScript. Sự khác nhau giữa microtask và macrotask là gì, và điều đó ảnh hưởng thế nào tới thứ tự chạy code trong một component React?

<details>
<summary>👉 Xem câu trả lời</summary>

JavaScript chạy đơn luồng (single-threaded). Event loop là cơ chế điều phối: nó liên tục kiểm tra call stack, và khi stack rỗng thì lấy việc từ các hàng đợi (queue) ra chạy. Có hai loại hàng đợi chính:

- Macrotask queue: `setTimeout`, `setInterval`, `setImmediate`, I/O, sự kiện DOM (event handler), `MessageChannel`.
- Microtask queue: `Promise.then/catch/finally`, `queueMicrotask`, `await` (phần code sau `await`), `MutationObserver`.

Quy tắc vàng: sau mỗi macrotask, event loop sẽ rút cạn TOÀN BỘ microtask queue rồi mới render và lấy macrotask tiếp theo. Vì vậy microtask luôn chạy trước macrotask kế tiếp.

```js
console.log('1: sync');

setTimeout(() => console.log('2: macrotask'), 0);

Promise.resolve().then(() => console.log('3: microtask'));

console.log('4: sync');

// Thứ tự in ra: 1 -> 4 -> 3 -> 2
// Code đồng bộ chạy hết trước, rồi microtask (3), cuối cùng mới tới macrotask (2)
```

Liên hệ React: từ React 18, các cập nhật state được batch (gộp) lại. Trong một event handler, nhiều lần gọi `setState` được gộp và chỉ re-render một lần. Hiểu microtask cũng giúp giải thích vì sao `await` trong một hàm khiến phần code phía sau bị "hoãn" tới sau khi stack rỗng — đôi khi gây ra hành vi state cũ (stale).

Bẫy thường gặp: tin rằng `setTimeout(fn, 0)` chạy "ngay lập tức". Thực tế nó luôn xếp sau toàn bộ microtask đang chờ.

</details>

---

### 1.2. Closure là gì? Cho một ví dụ closure gây bug trong React và cách khắc phục.

<details>
<summary>👉 Xem câu trả lời</summary>

Closure là khả năng một hàm "ghi nhớ" và truy cập các biến trong phạm vi (scope) nơi nó được khai báo, kể cả khi hàm đó được gọi ở nơi khác và scope gốc đã kết thúc thực thi.

Trong React, bug closure phổ biến nhất là "stale closure" (closure ôm giá trị cũ) bên trong `useEffect` hoặc `setInterval`.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      // count ở đây bị "đóng băng" bằng giá trị lúc effect chạy lần đầu = 0
      // nên luôn set thành 0 + 1 = 1, không bao giờ tăng tiếp
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
  }, []); // mảng deps rỗng => closure chỉ tạo một lần với count = 0

  return <p>{count}</p>;
}
```

Cách khắc phục: dùng dạng cập nhật hàm (functional update) để không phụ thuộc giá trị bị đóng băng:

```jsx
useEffect(() => {
  const id = setInterval(() => {
    setCount(prev => prev + 1); // prev luôn là giá trị mới nhất
  }, 1000);
  return () => clearInterval(id);
}, []);
```

Ứng dụng tích cực của closure: tạo hàm khởi tạo có state riêng (factory), memoization, hoặc giữ tham chiếu ổn định trong custom hook. Bẫy: khai báo thiếu dependency trong `useEffect`/`useCallback` thường tạo stale closure — eslint-plugin-react-hooks (rule `exhaustive-deps`) chính là để cảnh báo việc này.

</details>

---

### 1.3. `this` hoạt động thế nào trong JavaScript, và tại sao arrow function lại được ưa dùng trong React?

<details>
<summary>👉 Xem câu trả lời</summary>

Với hàm thường (`function`), `this` được xác định lúc GỌI hàm (call-site), không phải lúc khai báo:

- Gọi như method `obj.fn()` → `this` là `obj`.
- Gọi độc lập `fn()` → `this` là `undefined` (strict mode) hoặc `window` (non-strict).
- Gọi qua `call`/`apply`/`bind` → `this` là giá trị truyền vào.
- Dùng làm constructor với `new` → `this` là instance mới.

Arrow function KHÔNG có `this` riêng. Nó lấy `this` theo lexical scope (scope bao quanh nơi nó được định nghĩa), nên không bị mất `this` khi truyền đi làm callback.

```jsx
// Class component xưa: phải bind thủ công vì handleClick truyền vào onClick sẽ mất this
class Old extends React.Component {
  handleClick() {
    console.log(this); // undefined nếu không bind
  }
  // Cách 1: bind trong constructor   this.handleClick = this.handleClick.bind(this)
  // Cách 2 (phổ biến hơn): class field arrow function
  handleClickArrow = () => {
    console.log(this); // luôn trỏ đúng instance nhờ lexical this
  };
  render() {
    return <button onClick={this.handleClickArrow}>Click</button>;
  }
}
```

Trong code function component hiện đại, vấn đề `this` gần như biến mất vì ta không còn class. Tuy vậy hiểu `this` vẫn cần khi đọc code cũ, khi viết thư viện, hoặc khi xử lý `this` trong các callback của API bên ngoài.

Bẫy thường gặp: dùng arrow function làm method trên object literal rồi mong `this` trỏ tới object đó — không, arrow lấy `this` từ scope ngoài.

</details>

---

### 1.4. Phân biệt chạy nhiều tác vụ async TUẦN TỰ và SONG SONG với `async/await`. Khi nào dùng cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Điểm mấu chốt: `await` sẽ DỪNG hàm cho tới khi Promise xong. Nếu bạn `await` từng cái một, các tác vụ chạy tuần tự (chờ nhau). Nếu bạn khởi tạo tất cả Promise trước rồi mới `await Promise.all`, chúng chạy song song.

```ts
// TUẦN TỰ: chậm — tổng thời gian = tổng các request (vd 300ms + 300ms = 600ms)
async function sequential() {
  const user = await fetchUser();     // chờ xong mới chạy tiếp
  const posts = await fetchPosts();   // chỉ bắt đầu sau khi user xong
  return { user, posts };
}

// SONG SONG: nhanh — tổng thời gian = request lâu nhất (~300ms)
async function parallel() {
  const [user, posts] = await Promise.all([
    fetchUser(),   // cả hai khởi chạy cùng lúc
    fetchPosts(),
  ]);
  return { user, posts };
}
```

Khi nào dùng tuần tự: khi request sau PHỤ THUỘC kết quả request trước (ví dụ phải có `userId` rồi mới fetch đơn hàng của user đó).

Khi nào dùng song song: khi các tác vụ ĐỘC LẬP nhau — nên chạy song song để giảm thời gian chờ.

Liên hệ React/Next.js: trong Server Component hoặc `loader` của React Router, gom các fetch độc lập vào `Promise.all` giúp tránh "waterfall" (thác nước request nối tiếp gây chậm trang). Bẫy: dùng `Promise.all` cho tác vụ phụ thuộc nhau sẽ lỗi vì dữ liệu chưa sẵn sàng.

</details>

---

### 1.5. So sánh `Promise.all`, `Promise.allSettled`, `Promise.race` và `Promise.any`. Mỗi cái dùng trong tình huống nào?

<details>
<summary>👉 Xem câu trả lời</summary>

| Phương thức | Resolve khi nào | Reject khi nào | Kết quả trả về |
|---|---|---|---|
| `Promise.all` | TẤT CẢ resolve | NGAY KHI một cái reject (fail-fast) | Mảng các giá trị |
| `Promise.allSettled` | TẤT CẢ đã "settled" (xong dù thành/bại) | Không bao giờ reject | Mảng `{status, value/reason}` |
| `Promise.race` | Cái ĐẦU TIÊN settled (resolve hoặc reject) | Nếu cái đầu tiên settled là reject | Giá trị/lỗi của cái xong đầu tiên |
| `Promise.any` | Cái ĐẦU TIÊN resolve thành công | Khi TẤT CẢ đều reject (`AggregateError`) | Giá trị của cái resolve đầu tiên |

```ts
// all: cần tất cả, hỏng một cái là hỏng cả lô (ví dụ tải 3 phần dữ liệu bắt buộc của trang)
const [a, b, c] = await Promise.all([f1(), f2(), f3()]);

// allSettled: muốn biết kết quả của TỪNG cái, không muốn một lỗi làm sập cả nhóm
const results = await Promise.allSettled([f1(), f2(), f3()]);
results.forEach(r => {
  if (r.status === 'fulfilled') console.log(r.value);
  else console.warn(r.reason);
});

// race: làm timeout — đua request thật với một timer
const data = await Promise.race([
  fetchData(),
  new Promise((_, reject) => setTimeout(() => reject(new Error('timeout')), 5000)),
]);

// any: nhiều nguồn dự phòng (mirror), lấy nguồn nào trả về sớm nhất và thành công
const fastest = await Promise.any([fetchFromCDN1(), fetchFromCDN2()]);
```

Bẫy thường gặp: dùng `Promise.all` cho dashboard nhiều widget — chỉ một widget lỗi là cả dashboard trắng; nên dùng `allSettled` để render từng phần độc lập.

</details>

---

### 1.6. Tính bất biến (immutability) trong React là gì? Phân biệt shallow copy và deep copy.

<details>
<summary>👉 Xem câu trả lời</summary>

React phát hiện thay đổi state bằng so sánh tham chiếu (reference equality, `Object.is`). Nếu bạn sửa trực tiếp (mutate) object/array cũ, tham chiếu không đổi → React tưởng không có gì thay đổi → không re-render. Vì vậy phải tạo object/array MỚI thay vì sửa cái cũ — đó là immutability.

- Shallow copy (sao chép nông): chỉ sao chép cấp một; các object/array lồng bên trong vẫn dùng CHUNG tham chiếu. Dùng `{...obj}`, `[...arr]`, `Object.assign`, `Array.prototype.slice`.
- Deep copy (sao chép sâu): sao chép toàn bộ cây lồng nhau, không chia sẻ tham chiếu nào. Dùng `structuredClone(obj)` (API hiện đại), hoặc thư viện như lodash `cloneDeep`.

```ts
const state = { user: { name: 'An' }, tags: ['react'] };

// SHALLOW: state.user vẫn là cùng một object
const shallow = { ...state };
shallow.user.name = 'Binh';
console.log(state.user.name); // 'Binh' — đã vô tình sửa state gốc!

// DEEP: hoàn toàn tách biệt
const deep = structuredClone(state);
deep.user.name = 'Cuong';
console.log(state.user.name); // 'Binh' — state gốc không bị ảnh hưởng
```

Cập nhật state lồng nhau trong React đúng cách:

```ts
setUser(prev => ({ ...prev, address: { ...prev.address, city: 'HCM' } }));
```

Lưu ý: deep copy tốn chi phí, không cần thiết cho phần lớn cập nhật state phẳng. Thực tế nên giữ state nông và phẳng, hoặc dùng Immer (qua Redux Toolkit) để viết code "mutate" nhưng vẫn sinh ra bản bất biến. Bẫy: dùng `JSON.parse(JSON.stringify(obj))` để deep copy sẽ làm mất `Date`, `Map`, `Set`, `undefined`, hàm — `structuredClone` xử lý tốt hơn (trừ hàm).

</details>

---

### 1.7. Giải thích spread, rest và destructuring. Cho ví dụ chúng được dùng thế nào trong code React hằng ngày.

<details>
<summary>👉 Xem câu trả lời</summary>

Ba cú pháp này nhìn giống nhau (`...`) nhưng vai trò khác nhau:

- Destructuring: tách giá trị từ object/array ra biến.
- Spread: "trải" phần tử của một iterable/object ra (dùng khi tạo mảng/object/gọi hàm).
- Rest: gom phần còn lại vào một biến (dùng trong tham số hàm hoặc khi destructuring).

```tsx
// Destructuring props + đặt giá trị mặc định
function Button({ label, variant = 'primary', ...rest }: ButtonProps) {
  // rest: gom mọi prop còn lại (onClick, disabled, aria-*...) để chuyển tiếp
  return <button className={variant} {...rest}>{label}</button>; // spread props
}

// Destructuring kết quả của hook
const [count, setCount] = useState(0);
const { data, isLoading } = useQuery({ queryKey: ['user'], queryFn: fetchUser });

// Spread để cập nhật state bất biến
setForm(prev => ({ ...prev, email: 'a@b.com' }));

// Rest params trong hàm tiện ích
function logAll(...args: unknown[]) { console.log(args); }
```

Mẫu "props passthrough" (`{...rest}`) rất hữu ích khi xây component tái sử dụng: bạn tách ra vài prop cần xử lý riêng, còn lại chuyển thẳng xuống phần tử DOM.

Bẫy thường gặp: spread chỉ tạo shallow copy (xem câu trên), nên sửa object lồng bên trong vẫn ảnh hưởng bản gốc. Ngoài ra, thứ tự khi spread quan trọng: `{ ...defaults, ...overrides }` thì `overrides` thắng.

</details>

---

### 1.8. ES Modules là gì? Phân biệt named export với default export, và `import` tĩnh với `dynamic import()`.

<details>
<summary>👉 Xem câu trả lời</summary>

ES Modules (ESM) là hệ thống module chuẩn của JavaScript dùng `import`/`export`. Khác với CommonJS (`require`), ESM phân tích phụ thuộc tĩnh lúc biên dịch, cho phép tree-shaking (loại bỏ code không dùng).

```ts
// Named export: nhiều thứ một file, import phải đúng tên
export function add(a, b) { return a + b; }
export const PI = 3.14;
import { add, PI } from './math';

// Default export: một thứ "chính" mỗi file, import đặt tên tùy ý
export default function Button() {}
import MyButton from './Button';
```

`import` tĩnh được giải quyết lúc bundle/biên dịch, luôn nằm đầu file. `import()` động trả về một Promise và chỉ tải module KHI cần lúc chạy → tạo ra một "chunk" riêng, giúp code-splitting.

```tsx
// React: lazy-load component để giảm bundle ban đầu
import { lazy, Suspense } from 'react';
const HeavyChart = lazy(() => import('./HeavyChart')); // dynamic import

function Dashboard() {
  return (
    <Suspense fallback={<Spinner />}>
      <HeavyChart />
    </Suspense>
  );
}
```

Trong Next.js có `next/dynamic` làm điều tương tự, kèm tùy chọn `ssr: false` để bỏ render phía server cho component chỉ chạy ở client (ví dụ component dùng `window`).

Bẫy: trong file không phải module (script thường) thì không có `import`; default export khó tree-shake và khó refactor tên hơn named export — nhiều team ưu tiên named export.

</details>

---

### 1.9. Trong TypeScript, khi nào dùng `interface` và khi nào dùng `type`?

<details>
<summary>👉 Xem câu trả lời</summary>

Cả hai đều mô tả hình dạng (shape) dữ liệu và phần lớn thay thế được cho nhau khi định nghĩa object. Khác biệt chính:

| Tiêu chí | `interface` | `type` |
|---|---|---|
| Mô tả object/class shape | Có | Có |
| Union / intersection / tuple | Không trực tiếp | Có (`A | B`, `A & B`, `[a, b]`) |
| Mở rộng | `extends` | `&` (intersection) |
| Declaration merging (khai báo trùng tên gộp lại) | Có | Không |
| Primitive alias, mapped/conditional type | Không | Có |

```ts
// interface: hợp khi mô tả props/đối tượng, dễ extends, hỗ trợ merging
interface ButtonProps {
  label: string;
  onClick: () => void;
}
interface IconButtonProps extends ButtonProps {
  icon: string;
}

// type: hợp khi cần union, tuple, hoặc kiểu suy ra phức tạp
type Status = 'idle' | 'loading' | 'success' | 'error';
type Point = [number, number];
type Nullable<T> = T | null;
```

Quy tắc thực dụng nhiều team React áp dụng: dùng `interface` cho props của component và các object công khai (vì dễ extends và merging khi viết thư viện); dùng `type` cho union, tuple, và các kiểu tính toán bằng generics/utility types. Quan trọng nhất là nhất quán trong codebase.

Bẫy: declaration merging của `interface` đôi khi gây bất ngờ — hai interface cùng tên ở hai nơi sẽ tự gộp; `type` thì báo lỗi trùng tên (an toàn hơn về mặt này).

</details>

---

### 1.10. Generics trong TypeScript là gì? Cho một ví dụ generic hữu ích trong ngữ cảnh React.

<details>
<summary>👉 Xem câu trả lời</summary>

Generics cho phép viết hàm/kiểu/component hoạt động với NHIỀU kiểu khác nhau mà vẫn giữ được thông tin kiểu (type-safe), thay vì phải dùng `any`. Hiểu nôm na: generic là "tham số kiểu" được điền vào lúc sử dụng.

```ts
// Không generic: trả về any, mất type
function firstAny(arr: any[]): any { return arr[0]; }

// Generic: T được suy ra theo input, output giữ đúng kiểu
function first<T>(arr: T[]): T | undefined { return arr[0]; }
const n = first([1, 2, 3]);        // n: number | undefined
const s = first(['a', 'b']);       // s: string | undefined
```

Ví dụ trong React — một custom hook fetch dữ liệu generic:

```tsx
function useFetch<T>(url: string): { data: T | null; loading: boolean } {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  useEffect(() => {
    fetch(url)
      .then(r => r.json() as Promise<T>)
      .then(d => { setData(d); setLoading(false); });
  }, [url]);
  return { data, loading };
}

// Khi dùng, truyền kiểu cụ thể -> data được gõ kiểu đầy đủ
const { data } = useFetch<User>('/api/user'); // data: User | null
```

Có thể giới hạn generic bằng `extends` (constraint): `function getId<T extends { id: string }>(item: T)`. Component generic cũng hữu ích cho danh sách tái sử dụng: `<List<User> items={users} renderItem={...} />`.

Bẫy: lạm dụng generic làm signature khó đọc; hoặc truyền kiểu sai vào `.json() as Promise<T>` vì `as` chỉ ép kiểu chứ không kiểm tra runtime — nên xác thực bằng Zod nếu dữ liệu không tin cậy.

</details>

---

### 1.11. Utility types `Partial`, `Pick`, `Omit`, `Record` làm gì? Cho ví dụ thực tế khi gõ kiểu props.

<details>
<summary>👉 Xem câu trả lời</summary>

Utility types là các kiểu dựng sẵn giúp biến đổi một kiểu có sẵn thành kiểu mới, tránh lặp lại.

| Utility | Tác dụng |
|---|---|
| `Partial<T>` | Biến mọi thuộc tính thành tùy chọn (optional) |
| `Required<T>` | Ngược lại, biến mọi thuộc tính thành bắt buộc |
| `Pick<T, K>` | Lấy ra MỘT SỐ thuộc tính `K` từ `T` |
| `Omit<T, K>` | Bỏ ĐI một số thuộc tính `K` khỏi `T` |
| `Record<K, V>` | Tạo object có key kiểu `K`, value kiểu `V` |

```ts
interface User {
  id: string;
  name: string;
  email: string;
  age: number;
}

// Partial: dùng cho hàm update chỉ truyền vài field
function updateUser(id: string, patch: Partial<User>) {}
updateUser('1', { name: 'An' }); // hợp lệ, không cần đủ field

// Pick: props chỉ cần một phần của User
type UserCardProps = Pick<User, 'name' | 'email'>;

// Omit: tạo type "tạo mới" bỏ field do server sinh
type CreateUserInput = Omit<User, 'id'>;

// Record: map id -> User (dạng lookup table phổ biến trong Redux/Zustand)
type UsersById = Record<string, User>;
const map: UsersById = { '1': { id: '1', name: 'An', email: 'a@b.com', age: 20 } };
```

Ví dụ rất hay gặp khi mở rộng props của element DOM:

```tsx
// Lấy mọi prop của <button> trừ những prop ta tự định nghĩa lại
type Props = Omit<React.ComponentProps<'button'>, 'onClick'> & {
  onClick: (id: string) => void;
};
```

Bẫy: `Omit` không cảnh báo nếu bạn omit một key không tồn tại (trừ khi cấu hình chặt); và `Pick`/`Omit` chỉ làm ở cấp một, không đệ quy vào object lồng nhau.

</details>

---

### 1.12. Phân biệt `any`, `unknown` và `never` trong TypeScript. Khi nào dùng cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- `any`: tắt kiểm tra kiểu cho giá trị đó — gán đi đâu cũng được, gọi gì cũng được. Tiện nhưng nguy hiểm vì mất an toàn kiểu; nên tránh.
- `unknown`: "top type" an toàn. Nhận được mọi giá trị NHƯNG bạn phải thu hẹp kiểu (narrow) bằng kiểm tra trước khi dùng. Là lựa chọn an toàn thay cho `any`.
- `never`: "bottom type" — kiểu của thứ KHÔNG bao giờ tồn tại (hàm luôn throw, hoặc nhánh không thể xảy ra). Không gán được giá trị nào vào `never`.

```ts
// any: không cảnh báo gì, dễ sinh lỗi runtime
let a: any = 'hi';
a.foo.bar(); // compiler im lặng, runtime nổ

// unknown: bắt buộc kiểm tra trước khi dùng
function handle(value: unknown) {
  // value.length;          // LỖI biên dịch — chưa biết kiểu
  if (typeof value === 'string') {
    console.log(value.length); // OK sau khi narrow
  }
}

// never: dùng cho exhaustiveness check (bảo đảm xử lý hết mọi nhánh union)
type Status = 'idle' | 'loading' | 'done';
function render(s: Status) {
  switch (s) {
    case 'idle': return '...';
    case 'loading': return 'Đang tải';
    case 'done': return 'Xong';
    default:
      const _exhaustive: never = s; // nếu thêm status mới mà quên xử lý -> lỗi biên dịch
      return _exhaustive;
  }
}
```

Liên hệ React: API trả JSON nên gõ `unknown` rồi xác thực bằng Zod thay vì `any`; `catch (e)` trong TS hiện đại mặc định là `unknown`, buộc bạn kiểm tra `e instanceof Error` trước khi đọc `e.message`. Bẫy: dùng `any` cho props "cho nhanh" sẽ lan ra cả cây component và làm mất tác dụng của TypeScript.

</details>

---

### 1.13. Bạn gõ kiểu cho props của một component và cho giá trị trả về của custom hook như thế nào? Có những lưu ý gì?

<details>
<summary>👉 Xem câu trả lời</summary>

Gõ kiểu props: định nghĩa một `interface`/`type` cho props và truyền vào component. Với function component hiện đại, nên gõ trực tiếp tham số props thay vì dùng `React.FC` (vì `React.FC` từng ép kiểu `children` ngầm và gây vài bất tiện).

```tsx
interface CardProps {
  title: string;
  onClose?: () => void;             // optional
  variant?: 'sm' | 'md' | 'lg';     // union literal
  children: React.ReactNode;         // gõ children rõ ràng
}

function Card({ title, onClose, variant = 'md', children }: CardProps) {
  return <section className={variant}>{title}{children}</section>;
}
```

Gõ kiểu custom hook: hook chỉ là hàm, nên gõ kiểu tham số và kiểu trả về như hàm bình thường. Với hook trả về tuple (giống `useState`), dùng `as const` để TypeScript giữ đúng kiểu tuple thay vì suy ra mảng union.

```ts
function useToggle(initial = false) {
  const [on, setOn] = useState(initial);
  const toggle = useCallback(() => setOn(p => !p), []);
  return [on, toggle] as const; // kiểu: readonly [boolean, () => void]
}

const [isOpen, toggleOpen] = useToggle(); // isOpen: boolean, toggleOpen: () => void
```

Một số lưu ý:
- Gõ event đúng kiểu: `(e: React.ChangeEvent<HTMLInputElement>) => void`, `React.MouseEvent<HTMLButtonElement>`.
- Mở rộng prop của element gốc: `React.ComponentProps<'input'>` hoặc `ButtonHTMLAttributes<HTMLButtonElement>` để nhận đủ `className`, `aria-*`, `onClick`...
- Generic component (như câu 1.10) cho danh sách tái sử dụng.

Bẫy: gõ kiểu trả về của hook là object thì không cần `as const`; nhưng tuple thì rất cần, nếu không bạn nhận `(boolean | (() => void))[]` và destructuring mất kiểu. Tránh `any` cho props của thư viện ngoài — dùng `unknown` rồi narrow, hoặc tận dụng kiểu mà thư viện đã export.

</details>
## 2. React căn bản

### 2.1. JSX là gì và nó được biên dịch (compile) ra cái gì?

<details>
<summary>👉 Xem câu trả lời</summary>

JSX không phải HTML và cũng không phải một phần của JavaScript chuẩn. Nó chỉ là "đường cú pháp" (syntactic sugar) mà Babel (hoặc SWC trong Next.js) biên dịch thành các lời gọi hàm tạo phần tử React. Trình duyệt không bao giờ hiểu JSX trực tiếp.

Với "automatic runtime" (mặc định từ React 17 trở đi, không cần `import React` chỉ để dùng JSX), mỗi thẻ JSX được biên dịch thành lời gọi `_jsx`/`_jsxs` từ `react/jsx-runtime`. Với "classic runtime" cũ thì nó biên dịch thành `React.createElement(...)`.

```jsx
// Code bạn viết:
const el = <button className="btn" onClick={handleClick}>Lưu</button>;

// Babel (classic runtime) biên dịch ra:
const el = React.createElement(
  "button",
  { className: "btn", onClick: handleClick },
  "Lưu"
);

// Babel (automatic runtime, React 17+) biên dịch ra:
import { jsx as _jsx } from "react/jsx-runtime";
const el = _jsx("button", { className: "btn", onClick: handleClick, children: "Lưu" });
```

Kết quả của `createElement`/`_jsx` là một **React element** — chỉ là một object JavaScript thuần (kiểu `{ type, props, key, ... }`) mô tả những gì cần render, chứ chưa phải DOM thật.

Vài điều suy ra từ đây và hay bị hỏi tiếp:
- Đó là lý do `className` (không phải `class`) và `htmlFor` (không phải `for`), vì các props này map sang thuộc tính của object/DOM, và `class`/`for` là từ khóa trong JS.
- Đó là lý do component phải viết **hoa chữ đầu**: `<Button />` biên dịch thành `_jsx(Button, ...)` (tham chiếu biến), còn `<button />` thành `_jsx("button", ...)` (chuỗi, thẻ HTML thường).
- Mỗi block `{...}` trong JSX là một biểu thức JavaScript, nên bạn nhúng được giá trị nhưng không nhúng được câu lệnh (if/for) trực tiếp.

</details>

---

### 2.2. Virtual DOM là gì? Nó giúp giải quyết vấn đề gì?

<details>
<summary>👉 Xem câu trả lời</summary>

Virtual DOM là một bản biểu diễn nhẹ của UI dưới dạng cây các object JavaScript (chính là cây React element ở câu trên), được giữ trong bộ nhớ. Thao tác trực tiếp lên DOM thật rất tốn kém (đụng tới layout, reflow, repaint). Ý tưởng của Virtual DOM là:

1. Khi state/props đổi, React dựng một cây Virtual DOM mới.
2. React so sánh (diff) cây mới với cây cũ.
3. React tính ra **tập thay đổi tối thiểu** và chỉ áp các thay đổi đó lên DOM thật.

Điểm cần nói rõ trong phỏng vấn để tránh hiểu lầm phổ biến: Virtual DOM **không tự động làm app nhanh hơn DOM thật**. Một thao tác DOM thủ công tối ưu thường nhanh hơn. Giá trị thật của Virtual DOM là **mô hình lập trình khai báo (declarative)**: bạn mô tả "UI nên trông thế nào với state này", còn React lo phần đồng bộ DOM. Nó giúp gom (batch) cập nhật và tránh việc bạn phải thao tác DOM thủ công đầy lỗi.

```jsx
// Khai báo: bạn không nói "tìm thẻ span rồi đổi textContent"
// Bạn chỉ mô tả UI theo state, React tự tìm ra phần khác biệt và cập nhật.
function Counter({ count }) {
  return <span>Số lần click: {count}</span>;
}
```

Lưu ý: từ phiên bản kiến trúc Fiber (React 16+), "Virtual DOM" được hiện thực qua cấu trúc Fiber cho phép cập nhật có thể bị gián đoạn/ưu tiên (interruptible), phục vụ tính năng concurrent.

</details>

---

### 2.3. Reconciliation và thuật toán diffing của React hoạt động thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**Reconciliation** là quá trình React đối chiếu cây element mới với cây cũ để quyết định cập nhật gì trên DOM. So sánh hai cây một cách tổng quát có độ phức tạp O(n³), không khả thi. React dùng thuật toán heuristic gần O(n) dựa trên hai giả định:

1. **Hai element khác `type` thì coi như khác hoàn toàn.** Nếu `type` ở vị trí đó đổi (vd `<div>` thành `<span>`, hoặc `<Profile>` thành `<Settings>`), React **hủy toàn bộ cây con cũ** (unmount, state bị mất) và dựng cây mới. Nếu `type` giống nhau, React giữ node DOM và chỉ cập nhật các attribute/props đã đổi.

2. **`key` giúp React nhận diện phần tử nào ổn định giữa các lần render** trong một danh sách (xem câu về key).

```jsx
// type đổi -> toàn bộ cây con bên trong bị dựng lại, state mất:
{isEditing ? <EditForm /> : <ReadView />}

// type giữ nguyên (đều là <input>) -> React tái dùng DOM, chỉ cập nhật prop value:
<input value={value} onChange={onChange} />
```

Một bẫy hay gặp: nếu bạn để cùng một `type` ở cùng vị trí nhưng kỳ vọng nó "reset", state sẽ **không** reset vì React tái dùng node. Khi đó dùng `key` khác nhau để ép React coi đó là instance khác:

```jsx
// Khi userId đổi, ta muốn form reset hoàn toàn -> đổi key để ép remount:
<UserForm key={userId} userId={userId} />
```

Lưu ý: từ React 18, reconciliation có thể chạy ở chế độ concurrent (render có thể bị tạm dừng và làm lại), nhưng các quy tắc heuristic về type/key vẫn giữ nguyên.

</details>

---

### 2.4. Tại sao React cần `key` khi render danh sách, và vì sao không nên dùng index làm key?

<details>
<summary>👉 Xem câu trả lời</summary>

`key` cho React một định danh **ổn định** để biết phần tử nào trong danh sách tương ứng giữa lần render cũ và mới. Nhờ đó React tái dùng đúng node DOM và đúng state của component, thay vì hủy/tạo lại tốn kém hay gán nhầm state.

`key` chỉ cần **ổn định, duy nhất giữa các phần tử anh em (siblings)** — thường là id từ dữ liệu. Nó không cần unique toàn cục và không được đọc bằng props (React không truyền `key` xuống component con như một prop).

**Tại sao không dùng index?** Khi danh sách thay đổi thứ tự, bị chèn/xóa ở giữa, index của mỗi phần tử bị "dịch", nên React map sai phần tử cũ với phần tử mới. Hậu quả: state nội bộ (vd nội dung đang gõ trong input) hoặc DOM bị gán nhầm sang dòng khác.

```jsx
// SAI: thêm item vào đầu danh sách sẽ làm các giá trị input bị lệch dòng
{todos.map((todo, index) => (
  <li key={index}>
    <input defaultValue={todo.text} />
  </li>
))}

// ĐÚNG: dùng id ổn định từ dữ liệu
{todos.map((todo) => (
  <li key={todo.id}>
    <input defaultValue={todo.text} />
  </li>
))}
```

Khi nào index "tạm chấp nhận được": danh sách **tĩnh**, không sắp xếp lại, không thêm/xóa ở giữa, và các phần tử không có state nội bộ. Còn lại thì luôn ưu tiên id thật. Tránh dùng `Math.random()` làm key vì nó đổi mỗi lần render, khiến React remount toàn bộ list mỗi lần.

</details>

---

### 2.5. Phân biệt `props` và `state`.

<details>
<summary>👉 Xem câu trả lời</summary>

| | `props` | `state` |
|---|---|---|
| Sở hữu bởi | Component cha truyền xuống | Bản thân component quản lý |
| Có thể thay đổi? | Bất biến (read-only) với component nhận | Thay đổi được qua hàm setter (`setState`/`useState`) |
| Ai cập nhật | Cha render lại với props mới | Chính component (qua sự kiện, effect...) |
| Vai trò | Cấu hình/dữ liệu đầu vào | Dữ liệu nội bộ thay đổi theo thời gian |
| Kích hoạt re-render | Khi cha truyền props mới | Khi gọi setter làm state đổi |

Cách nhớ ngắn: **props giống tham số của một hàm** (bên ngoài truyền vào, bên trong không tự sửa), còn **state giống biến cục bộ "ghi nhớ được" của hàm đó qua các lần render**.

```jsx
function Counter({ step }) {        // step là props (do cha truyền, không tự sửa)
  const [count, setCount] = useState(0); // count là state (component tự quản)
  return (
    <button onClick={() => setCount(count + step)}>
      {count}
    </button>
  );
}

// Cha:
<Counter step={5} />
```

Bẫy hay gặp: cố gắng "sửa props" bên trong component con — đó là sai mô hình. Nếu con cần thay đổi dữ liệu của cha, cha truyền xuống một callback (vd `onChange`) để con báo lên; cha mới là nơi cập nhật state.

</details>

---

### 2.6. Controlled component và uncontrolled component khác nhau thế nào? Cho ví dụ.

<details>
<summary>👉 Xem câu trả lời</summary>

Điểm khác biệt là **ai giữ "nguồn sự thật" (source of truth) cho giá trị của input**.

- **Controlled component**: React state giữ giá trị. Input nhận `value` từ state và mỗi lần gõ gọi `onChange` để cập nhật state. UI luôn phản ánh state.
- **Uncontrolled component**: DOM tự giữ giá trị. React không kiểm soát từng lần gõ; ta đọc giá trị khi cần qua `ref` (hoặc `defaultValue` để set giá trị khởi tạo).

```jsx
// Controlled: state là nguồn sự thật
function ControlledInput() {
  const [name, setName] = useState("");
  return (
    <input
      value={name}
      onChange={(e) => setName(e.target.value)}
    />
  );
}

// Uncontrolled: DOM giữ giá trị, đọc qua ref khi submit
function UncontrolledInput() {
  const inputRef = useRef(null);
  const handleSubmit = () => {
    console.log(inputRef.current.value); // đọc tại thời điểm cần
  };
  return (
    <>
      <input ref={inputRef} defaultValue="" />
      <button onClick={handleSubmit}>Gửi</button>
    </>
  );
}
```

**Khi nào dùng cái nào?**
- Controlled khi cần validate tức thì, disable nút theo nội dung, format khi gõ, hoặc đồng bộ nhiều field — tức là UI cần phản ứng theo giá trị. Đây là lựa chọn mặc định trong React.
- Uncontrolled khi form đơn giản, cần tích hợp DOM/thư viện ngoài, hoặc khi muốn tránh re-render mỗi phím gõ vì lý do hiệu năng. Thư viện như **React Hook Form** dùng cách tiếp cận uncontrolled (qua ref) để giảm re-render, đó là một thế mạnh của nó so với form controlled thủ công.

Bẫy: nếu truyền `value` mà quên `onChange`, input thành read-only và React cảnh báo. Đừng để một input "lúc controlled lúc uncontrolled" (vd `value={x}` với `x` lúc `undefined` lúc có giá trị) — React sẽ cảnh báo và hành vi khó lường.

</details>

---

### 2.7. "Luồng dữ liệu một chiều" (one-way / unidirectional data flow) trong React nghĩa là gì?

<details>
<summary>👉 Xem câu trả lời</summary>

Dữ liệu trong React chảy **một chiều từ trên xuống**: từ component cha xuống con qua props. Con không sửa trực tiếp dữ liệu của cha. Khi con cần thay đổi gì đó, nó gọi **callback do cha truyền xuống**, và cha mới là nơi cập nhật state; state mới lại chảy xuống qua props. Nhờ đó luồng dữ liệu dễ dự đoán và dễ debug — khi UI sai, bạn lần ngược lên trên để tìm nơi giữ state.

Điều này khác với two-way binding kiểu một số framework cũ, nơi view và model tự đồng bộ hai chiều. Ngay cả "two-way binding" trong React (như input controlled) thực chất vẫn là một chiều: state -> `value` đi xuống, sự kiện -> `setState` đi lên, chỉ là ta tự nối lại thủ công.

```jsx
function Parent() {
  const [query, setQuery] = useState("");
  return (
    <>
      <SearchBox value={query} onChange={setQuery} />  {/* xuống: value; lên: onChange */}
      <Results query={query} />
    </>
  );
}

function SearchBox({ value, onChange }) {
  // không tự giữ "sự thật"; báo lên cha qua onChange
  return <input value={value} onChange={(e) => onChange(e.target.value)} />;
}
```

Khi nhiều component cần chia sẻ cùng một state, ta "nâng state lên" (lifting state up) tới tổ tiên chung gần nhất; nếu truyền props qua quá nhiều tầng (prop drilling) thì dùng Context hoặc thư viện state như Redux Toolkit/Zustand.

</details>

---

### 2.8. Các giai đoạn vòng đời (lifecycle) của class component ánh xạ sang hooks như thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Hooks không có "lifecycle method" tách biệt; thay vào đó `useEffect` mô tả **đồng bộ hóa với hệ thống bên ngoài** theo dependency. Bảng ánh xạ tương đương phổ biến:

| Class component | Tương đương với hooks |
|---|---|
| `componentDidMount` | `useEffect(() => { ... }, [])` |
| `componentDidUpdate` | `useEffect(() => { ... }, [deps])` (chạy khi deps đổi) |
| `componentWillUnmount` | hàm cleanup `return () => { ... }` trong `useEffect` |
| `constructor` (khởi tạo state) | `useState(initial)` / `useReducer` |
| `getDerivedStateFromProps` | tính trực tiếp khi render, hoặc đổi `key` để reset |
| `shouldComponentUpdate` | `React.memo` + `useMemo`/`useCallback` |

```jsx
useEffect(() => {
  const id = setInterval(tick, 1000);   // ~ componentDidMount
  return () => clearInterval(id);        // ~ componentWillUnmount (cleanup)
}, []); // mảng rỗng -> chỉ chạy sau lần mount đầu

useEffect(() => {
  document.title = `Bạn có ${count} thông báo`; // ~ componentDidUpdate khi count đổi
}, [count]);
```

Lưu ý quan trọng về tư duy: với hooks, đừng nghĩ theo "chạy ở giai đoạn nào", hãy nghĩ "effect này đồng bộ cái gì với state/props nào". Cùng một logic class có thể tách thành **nhiều `useEffect`** theo mối quan tâm, thay vì gom hết vào `componentDidMount`. Ngoài ra trong React 18 StrictMode (dev), effect bị chạy mount-unmount-mount để giúp phát hiện cleanup thiếu — nên cleanup phải đúng.

</details>

---

### 2.9. `Fragment` (`<></>`) là gì và dùng để làm gì?

<details>
<summary>👉 Xem câu trả lời</summary>

Một component chỉ trả về được **một node gốc**. `Fragment` cho phép nhóm nhiều phần tử con mà **không thêm node DOM thừa** (không tạo `<div>` bao ngoài). Điều này quan trọng với layout dựa trên flex/grid, hoặc khi cấu trúc HTML hợp lệ không cho phép thẻ bọc (vd `<td>` trong `<tr>`, `<option>` trong `<select>`).

```jsx
// Không Fragment: bị buộc phải bọc 1 div thừa, có thể phá vỡ layout
function Cells() {
  return (
    <>
      <td>Tên</td>
      <td>Email</td>
    </>
  );
}
```

Có hai dạng:
- Cú pháp rút gọn `<>...</>`: ngắn gọn nhưng **không nhận key/prop**.
- Dạng đầy đủ `<React.Fragment key={...}>...</React.Fragment>`: dùng khi render danh sách mà mỗi phần tử cần bọc nhiều node và cần `key`.

```jsx
{items.map((item) => (
  <React.Fragment key={item.id}>
    <dt>{item.term}</dt>
    <dd>{item.desc}</dd>
  </React.Fragment>
))}
```

</details>

---

### 2.10. Tại sao không được mutate (sửa trực tiếp) state trong React?

<details>
<summary>👉 Xem câu trả lời</summary>

React quyết định có cần re-render hay không bằng cách so sánh **tham chiếu (reference)** của state cũ và mới (so sánh nông — `Object.is`). Nếu bạn sửa trực tiếp object/array hiện có, tham chiếu **không đổi**, nên React nghĩ "không có gì thay đổi" và **không re-render** — UI bị lỗi không cập nhật. Ngoài ra mutate trực tiếp còn phá các tối ưu dựa trên so sánh nông (`React.memo`, dependency của `useMemo`/`useEffect`) và làm state khó dự đoán.

Quy tắc: luôn tạo **object/array mới** (immutable update).

```jsx
// SAI: mutate trực tiếp -> tham chiếu không đổi -> không re-render
const handleAdd = () => {
  todos.push(newTodo);     // mutate mảng cũ
  setTodos(todos);         // cùng reference -> React bỏ qua
};

// ĐÚNG: tạo mảng/đối tượng mới
setTodos((prev) => [...prev, newTodo]);                 // thêm
setTodos((prev) => prev.filter((t) => t.id !== id));    // xóa
setUser((prev) => ({ ...prev, name: "An" }));           // sửa 1 field

// Lồng nhau sâu thì dùng Immer (Redux Toolkit dùng sẵn Immer):
setState(produce((draft) => { draft.a.b.c = 1; }));
```

Bẫy đi kèm: khi cập nhật dựa trên giá trị trước đó (nhất là gọi nhiều lần liên tiếp), hãy dùng **dạng hàm cập nhật** `setCount(prev => prev + 1)` thay vì `setCount(count + 1)`, vì `count` trong closure có thể là giá trị cũ (stale).

</details>

---

### 2.11. Synthetic event trong React là gì? Nó khác native event ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

`SyntheticEvent` là lớp bọc (wrapper) đa trình duyệt mà React tạo quanh native DOM event, để hành vi sự kiện đồng nhất giữa các trình duyệt và có API thống nhất (`e.target`, `e.preventDefault()`, `e.stopPropagation()`...). Bạn vẫn truy cập native event qua `e.nativeEvent` khi cần.

Cơ chế quan trọng: React **không gắn listener lên từng node**. Nó dùng **event delegation** — gắn listener ở gốc. Từ **React 17 trở đi, listener gắn vào container gốc của app** (`root`), chứ không phải `document` như React 16. Điều này quan trọng khi tích hợp nhiều ứng dụng React trên cùng trang.

```jsx
function Form() {
  const handleSubmit = (e) => {
    e.preventDefault();      // chặn reload trang
    e.stopPropagation();     // chặn nổi bọt lên cha
    console.log(e.type, e.nativeEvent); // truy cập native event khi cần
  };
  return <form onSubmit={handleSubmit}>...</form>;
}
```

Vài lưu ý/bẫy:
- Tên handler dùng camelCase: `onClick`, `onChange`, `onSubmit` (khác HTML `onclick`).
- **Trong React 17+, không còn event pooling** (việc `e` bị "tái sử dụng" rồi xóa thuộc tính sau handler như React 16). Nếu bạn còn nhớ phải gọi `e.persist()` — đó là chuyện của React 16, nay không cần nữa.
- `onChange` của React thực chất bắn ở mỗi lần gõ (giống `input` của DOM), không phải khi blur như native `change`.

</details>

---

### 2.12. Cách render danh sách và render có điều kiện trong React?

<details>
<summary>👉 Xem câu trả lời</summary>

**Render danh sách**: dùng `Array.prototype.map` để biến mảng dữ liệu thành mảng element, mỗi phần tử cần `key` ổn định.

```jsx
<ul>
  {users.map((user) => (
    <li key={user.id}>{user.name}</li>
  ))}
</ul>
```

**Render có điều kiện**: vì JSX nhận biểu thức (không nhận `if`), ta dùng:

```jsx
// 1) Toán tử ba ngôi - khi có 2 nhánh
{isLoading ? <Spinner /> : <Content data={data} />}

// 2) Toán tử && - hiển thị hoặc không
{error && <ErrorBanner message={error} />}

// 3) Tách biến/hàm ở trên return cho logic phức tạp
let body;
if (status === "loading") body = <Spinner />;
else if (status === "error") body = <ErrorBanner />;
else body = <List items={items} />;
return <div>{body}</div>;
```

Bẫy kinh điển với `&&`: nếu vế trái là **số 0**, React sẽ render ra `0` trên màn hình (vì `0` là falsy nhưng vẫn là giá trị hợp lệ để render), còn `false`/`null`/`undefined` thì không render gì.

```jsx
// SAI: khi items.length === 0 -> in ra "0"
{items.length && <List items={items} />}

// ĐÚNG: ép về boolean rõ ràng
{items.length > 0 && <List items={items} />}
```

</details>

---

### 2.13. `children` là gì và component composition (kết hợp component) giải quyết vấn đề gì?

<details>
<summary>👉 Xem câu trả lời</summary>

`children` là một prop đặc biệt chứa **nội dung nằm giữa thẻ mở và thẻ đóng** của component. Nhờ nó, một component có thể bọc và hiển thị nội dung tùy ý mà không cần biết trước nội dung đó là gì — đây là nền tảng của **composition**.

```jsx
function Card({ title, children }) {
  return (
    <section className="card">
      <h3>{title}</h3>
      <div className="card-body">{children}</div>
    </section>
  );
}

// Dùng:
<Card title="Hồ sơ">
  <Avatar />
  <p>Một số thông tin...</p>
</Card>
```

**Composition giải quyết vấn đề gì?** React khuyến nghị **ưu tiên composition hơn kế thừa (inheritance)** để tái sử dụng. Thay vì tạo nhiều biến thể component bằng kế thừa, ta lắp ghép component nhỏ lại với nhau. Composition cũng là cách tránh prop drilling và tránh việc component cha phải biết chi tiết con: cha chỉ "chừa chỗ" qua `children` (hoặc qua props là JSX, gọi là slot).

```jsx
// "Slot" qua props là JSX - linh hoạt hơn 1 children duy nhất:
function Layout({ sidebar, content }) {
  return (
    <div className="layout">
      <aside>{sidebar}</aside>
      <main>{content}</main>
    </div>
  );
}

<Layout sidebar={<Nav />} content={<Dashboard />} />
```

Ý nghĩa thực tế: composition giúp tạo các component dùng chung như `Modal`, `Card`, `Layout`, `Button`... trong design system (vd kết hợp với Storybook), nơi "khung" cố định nhưng "ruột" do nơi dùng quyết định.

</details>
## 3. React Hooks

### 3.1. `useState` hoạt động như thế nào? Khi nào nên dùng "functional update" và "lazy initialization"?

<details>
<summary>👉 Xem câu trả lời</summary>

`useState` trả về một cặp `[state, setState]` và giữ giá trị xuyên suốt các lần render. Có hai kỹ thuật quan trọng:

- **Functional update** (`setState(prev => ...)`): dùng khi giá trị mới phụ thuộc vào giá trị cũ. Vì state trong một closure có thể bị "cũ" (stale), truyền hàm bảo đảm React luôn đưa cho bạn giá trị mới nhất. Đặc biệt cần thiết khi cập nhật nhiều lần trong cùng một sự kiện, hoặc trong `setInterval`/event handler đăng ký một lần.
- **Lazy initialization** (`useState(() => expensiveCompute())`): truyền một HÀM thay vì giá trị. Hàm khởi tạo chỉ chạy ở lần render ĐẦU TIÊN, tránh tính toán nặng lặp lại ở mỗi lần render.

```tsx
// Bẫy: cập nhật theo giá trị cũ -> chỉ tăng 1 dù gọi 3 lần
function brokenIncrement() {
  setCount(count + 1);
  setCount(count + 1); // vẫn dựa trên `count` cũ
  setCount(count + 1);
}

// Đúng: functional update -> tăng đủ 3
function correctIncrement() {
  setCount(prev => prev + 1);
  setCount(prev => prev + 1);
  setCount(prev => prev + 1);
}

// Lazy init: đọc localStorage chỉ 1 lần lúc mount
const [token, setToken] = useState(() => localStorage.getItem('token') ?? '');
```

**Bẫy thường gặp:** viết `useState(expensiveCompute())` (gọi hàm ngay) sẽ chạy `expensiveCompute` ở MỌI lần render — React chỉ bỏ qua kết quả từ lần thứ hai trở đi chứ không bỏ qua việc gọi hàm. Phải truyền `useState(expensiveCompute)` hoặc `useState(() => expensiveCompute())`.

</details>

---

### 3.2. Mảng dependency của `useEffect` có ý nghĩa gì? Phân biệt 3 trường hợp: không truyền, mảng rỗng `[]`, và có dependency.

<details>
<summary>👉 Xem câu trả lời</summary>

Mảng dependency cho React biết khi nào cần chạy lại effect: sau mỗi lần render, React so sánh từng phần tử bằng `Object.is`; nếu có thay đổi thì chạy cleanup của lần trước rồi chạy lại effect.

| Cách viết | Khi nào effect chạy |
| --- | --- |
| `useEffect(fn)` (bỏ mảng) | Sau MỖI lần render |
| `useEffect(fn, [])` | Chỉ một lần sau lần mount (và cleanup khi unmount) |
| `useEffect(fn, [a, b])` | Sau mount, và mỗi khi `a` hoặc `b` đổi giá trị |

```tsx
// Đăng ký listener một lần, dọn dẹp khi unmount
useEffect(() => {
  const onResize = () => setWidth(window.innerWidth);
  window.addEventListener('resize', onResize);
  return () => window.removeEventListener('resize', onResize);
}, []);

// Chạy lại khi userId đổi
useEffect(() => {
  fetchUser(userId);
}, [userId]);
```

**Bẫy thường gặp:** cố tình bỏ bớt dependency để "effect chỉ chạy 1 lần" sẽ tạo stale closure — effect đọc giá trị cũ. Quy tắc: liệt kê ĐẦY ĐỦ mọi biến reactive mà effect dùng (đúng theo `eslint-plugin-react-hooks`). Nếu một object/function bị tạo mới mỗi render lọt vào dependency, effect sẽ chạy vô tận — khi đó dùng `useCallback`/`useMemo` hoặc functional update để loại nó khỏi dependency.

</details>

---

### 3.3. Cleanup function trong `useEffect` để làm gì? Cho ví dụ thực tế.

<details>
<summary>👉 Xem câu trả lời</summary>

Hàm `return` từ effect là cleanup. Nó chạy trước khi effect kế tiếp chạy lại VÀ khi component unmount. Mục đích là hủy đăng ký, dọn dẹp tài nguyên để tránh memory leak và side effect chồng chéo (ví dụ nhiều listener cộng dồn, timer chạy sau khi component biến mất).

```tsx
// Subscription: hủy subscribe khi roomId đổi hoặc unmount
useEffect(() => {
  const conn = createConnection(roomId);
  conn.connect();
  return () => conn.disconnect(); // dọn dẹp kết nối cũ
}, [roomId]);

// setInterval: clear để không bị nhiều interval cùng chạy
useEffect(() => {
  const id = setInterval(() => tick(), 1000);
  return () => clearInterval(id);
}, []);
```

Thứ tự khi `roomId` đổi: cleanup kết nối cũ -> chạy effect tạo kết nối mới. Nhờ vậy luôn chỉ có một kết nối active.

**Bẫy thường gặp:** quên cleanup khi đăng ký event/subscription/timer khiến mỗi lần render lại cộng thêm một handler, gây lỗi gọi nhiều lần và rò rỉ bộ nhớ.

</details>

---

### 3.4. Trong React StrictMode (dev), tại sao effect chạy 2 lần? Điều này nói lên vấn đề gì về code của bạn?

<details>
<summary>👉 Xem câu trả lời</summary>

Ở chế độ development với `<StrictMode>`, React cố ý mount component, chạy effect, rồi chạy cleanup, rồi chạy effect lại (mount -> cleanup -> mount). Mục đích là phát hiện sớm các effect KHÔNG idempotent / thiếu cleanup. Nó KHÔNG xảy ra ở production build.

Nếu effect của bạn bị lỗi khi chạy 2 lần (ví dụ gọi API tạo bản ghi 2 lần, đếm tăng gấp đôi, listener nhân đôi), đó là dấu hiệu effect thiếu cleanup hoặc không an toàn khi chạy lại — chứ không phải lỗi của React.

```tsx
// Bị nhân đôi ở StrictMode vì không cleanup
useEffect(() => {
  const conn = createConnection();
  conn.connect();
  // thiếu return cleanup -> StrictMode để lộ ra 2 kết nối
}, []);

// Sửa: thêm cleanup -> mount/cleanup/mount vẫn để lại đúng 1 kết nối
useEffect(() => {
  const conn = createConnection();
  conn.connect();
  return () => conn.disconnect();
}, []);
```

**Lưu ý Next.js:** App Router bật StrictMode mặc định ở dev, nên hiện tượng chạy 2 lần rất hay gặp khi mới chuyển sang. Cách xử lý đúng là viết effect idempotent + có cleanup, không phải tắt StrictMode.

</details>

---

### 3.5. Race condition khi fetch dữ liệu trong `useEffect` là gì? Làm sao để xử lý?

<details>
<summary>👉 Xem câu trả lời</summary>

Khi dependency đổi nhanh (ví dụ người dùng gõ search liên tục, hoặc đổi `userId` nhanh), nhiều request được gửi đi. Chúng có thể trả về KHÔNG đúng thứ tự — response của request cũ về sau lại ghi đè kết quả của request mới, làm UI hiển thị dữ liệu sai. Đó là race condition.

Cách xử lý phổ biến: dùng cờ `ignore` trong cleanup, hoặc `AbortController` để hủy request cũ.

```tsx
useEffect(() => {
  let ignore = false;
  const controller = new AbortController();

  (async () => {
    const res = await fetch(`/api/users/${userId}`, { signal: controller.signal });
    const data = await res.json();
    if (!ignore) setUser(data); // chỉ set nếu effect này còn "hợp lệ"
  })();

  return () => {
    ignore = true;       // bỏ qua kết quả của request cũ
    controller.abort();  // hủy luôn request mạng
  };
}, [userId]);
```

**Thực tế:** trong dự án thực, nên dùng **TanStack Query** (`useQuery({ queryKey: ['user', userId], queryFn })`). Nó tự xử lý race condition, caching, dedupe và loại bỏ response cũ — gần như không bao giờ phải tự viết logic fetch trong `useEffect`.

</details>

---

### 3.6. `useRef` dùng để làm gì? Nó khác `useState` ở điểm nào?

<details>
<summary>👉 Xem câu trả lời</summary>

`useRef` trả về một object `{ current: ... }` ổn định xuyên suốt các lần render. Hai công dụng chính:

1. **Tham chiếu DOM node** (truyền vào thuộc tính `ref`).
2. **Lưu một giá trị mutable** không liên quan đến render (ví dụ id của timer, giá trị trước đó, biến đếm).

Khác biệt cốt lõi với `useState`: thay đổi `ref.current` KHÔNG kích hoạt re-render, còn `setState` thì có. Ref cũng được cập nhật ngay lập tức (đồng bộ), trong khi state thì áp dụng ở lần render kế tiếp.

| | `useState` | `useRef` |
| --- | --- | --- |
| Thay đổi gây re-render | Có | Không |
| Cập nhật | Bất đồng bộ (theo render) | Ngay lập tức, mutable |
| Dùng cho | Dữ liệu hiển thị trên UI | DOM, giá trị "ngầm" không cần render |

```tsx
function SearchBox() {
  const inputRef = useRef<HTMLInputElement>(null);
  const renderCount = useRef(0);

  useEffect(() => {
    inputRef.current?.focus(); // truy cập DOM
  }, []);

  renderCount.current++; // đếm số lần render mà KHÔNG gây re-render

  return <input ref={inputRef} />;
}
```

**Bẫy:** đừng đọc/ghi `ref.current` trong lúc render để tạo logic hiển thị — vì thay đổi nó không trigger render, UI sẽ không đồng bộ. Dữ liệu cần hiện trên màn hình phải nằm ở state.

</details>

---

### 3.7. `useMemo` và `useCallback` khác nhau như thế nào? Khi nào thực sự nên dùng, và tại sao lạm dụng lại có hại?

<details>
<summary>👉 Xem câu trả lời</summary>

Cả hai đều memoize dựa trên dependency array:

- `useMemo(() => compute(a, b), [a, b])` — ghi nhớ KẾT QUẢ của một phép tính.
- `useCallback(fn, [a, b])` — ghi nhớ chính HÀM đó (tương đương `useMemo(() => fn, deps)`).

```tsx
// useMemo: tránh tính lại danh sách đã lọc nặng nề
const filtered = useMemo(
  () => hugeList.filter(item => item.name.includes(query)),
  [hugeList, query]
);

// useCallback: giữ tham chiếu hàm ổn định cho component con đã React.memo
const handleSelect = useCallback((id: string) => {
  setSelected(id);
}, []);
```

**Khi nào nên dùng:**
- `useMemo`: phép tính thực sự tốn kém, hoặc cần giữ tham chiếu object/array ổn định để truyền làm dependency / prop cho component con đã `memo`.
- `useCallback`: hàm được truyền xuống component con bọc `React.memo`, hoặc dùng làm dependency của một hook khác.

**Tại sao lạm dụng có hại:**
- Bản thân memoization tốn chi phí: lưu cache, so sánh dependency mỗi render. Với phép tính rẻ, chi phí này còn LỚN hơn cái nó tiết kiệm.
- Làm code rối, khó đọc, dễ sai dependency.
- Nếu component con KHÔNG `React.memo` thì `useCallback` vô nghĩa, vì con vẫn re-render.

**Lưu ý:** React 19 + React Compiler có thể tự động memoize, giảm nhu cầu viết tay `useMemo`/`useCallback`.

</details>

---

### 3.8. Khi nào nên chọn `useReducer` thay vì `useState`?

<details>
<summary>👉 Xem câu trả lời</summary>

`useReducer` phù hợp khi:
- State phức tạp, gồm nhiều trường liên quan nhau, hoặc state tiếp theo phụ thuộc state trước theo logic rõ ràng.
- Có nhiều cách chuyển trạng thái (nhiều action) — gom logic vào một reducer giúp dễ test, dễ đọc và tập trung.
- Cần truyền `dispatch` xuống sâu (dispatch ổn định, không đổi tham chiếu nên tốt cho `useCallback`/context).

`useState` phù hợp cho state đơn giản, độc lập (boolean, string, số).

```tsx
type State = { count: number; step: number };
type Action = { type: 'inc' } | { type: 'dec' } | { type: 'setStep'; step: number };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'inc': return { ...state, count: state.count + state.step };
    case 'dec': return { ...state, count: state.count - state.step };
    case 'setStep': return { ...state, step: action.step };
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0, step: 1 });
  return (
    <>
      <span>{state.count}</span>
      <button onClick={() => dispatch({ type: 'inc' })}>+</button>
    </>
  );
}
```

**Quy tắc ngón tay cái:** nếu nhiều `setState` luôn được gọi cùng nhau, hoặc logic cập nhật bắt đầu lan ra nhiều chỗ và khó suy luận, hãy gom thành một reducer. Reducer là hàm thuần (pure) nên rất dễ unit test mà không cần render component.

</details>

---

### 3.9. `useContext` giải quyết vấn đề gì? Nhược điểm về hiệu năng của nó là gì?

<details>
<summary>👉 Xem câu trả lời</summary>

`useContext` cho phép đọc một giá trị context được cung cấp bởi `Provider` ở phía trên cây component, tránh "prop drilling" (truyền prop qua nhiều tầng trung gian). Hay dùng cho theme, ngôn ngữ, thông tin user đã đăng nhập.

```tsx
const ThemeContext = createContext<'light' | 'dark'>('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}

function ThemedButton() {
  const theme = useContext(ThemeContext); // không cần truyền prop qua Toolbar
  return <button className={theme}>Click</button>;
}
```

**Nhược điểm hiệu năng:** khi giá trị `value` của Provider thay đổi, MỌI component đang `useContext` context đó sẽ re-render — không có cơ chế chọn lọc (selector) như Redux/Zustand. Vì vậy:
- Đừng đặt giá trị đổi liên tục (ví dụ vị trí chuột) vào context dùng chung diện rộng.
- Nếu `value` là object/array tạo mới mỗi render, hãy bọc `useMemo` để tránh re-render thừa.
- State server-state (dữ liệu fetch) nên dùng TanStack Query; global client-state phức tạp nên dùng Zustand/Redux Toolkit thay vì nhồi hết vào một context lớn.

</details>

---

### 3.10. Custom hook là gì? Hãy viết một hook `useDebounce`.

<details>
<summary>👉 Xem câu trả lời</summary>

Custom hook là một hàm bắt đầu bằng `use`, dùng các hook khác để đóng gói và tái sử dụng logic có state (stateful logic) giữa nhiều component. Nó chia sẻ LOGIC, không chia sẻ state — mỗi component gọi hook sẽ có state riêng.

`useDebounce` trả về một giá trị chỉ cập nhật sau khi giá trị nguồn ngừng thay đổi trong khoảng thời gian `delay` — rất hợp cho ô tìm kiếm.

```tsx
function useDebounce<T>(value: T, delay = 500): T {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id); // hủy timer cũ khi value đổi -> debounce
  }, [value, delay]);

  return debounced;
}

// Sử dụng: chỉ gọi API khi người dùng ngừng gõ 500ms
function Search() {
  const [text, setText] = useState('');
  const debounced = useDebounce(text, 500);

  useEffect(() => {
    if (debounced) fetchResults(debounced);
  }, [debounced]);

  return <input value={text} onChange={e => setText(e.target.value)} />;
}
```

Điểm cốt lõi của debounce nằm ở cleanup `clearTimeout`: mỗi lần `value` đổi, timer cũ bị hủy, chỉ lần gõ cuối cùng sau khoảng lặng mới thực sự cập nhật.

</details>

---

### 3.11. Hãy viết các custom hook `usePrevious` và `useToggle`.

<details>
<summary>👉 Xem câu trả lời</summary>

`usePrevious` trả về giá trị của lần render TRƯỚC — tận dụng việc `useEffect` chạy SAU render, nên `ref.current` còn giữ giá trị cũ trong lúc render hiện tại.

```tsx
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();
  useEffect(() => {
    ref.current = value; // cập nhật SAU khi render -> lần render sau mới thấy
  }, [value]);
  return ref.current; // trong render hiện tại, đây là giá trị lần trước
}

// Dùng: so sánh count cũ và mới
const prevCount = usePrevious(count);
```

`useToggle` đóng gói trạng thái boolean cùng hàm đảo và hàm set tường minh:

```tsx
function useToggle(initial = false) {
  const [value, setValue] = useState(initial);
  const toggle = useCallback(() => setValue(v => !v), []); // functional update
  return [value, toggle, setValue] as const; // as const để giữ kiểu tuple
}

// Dùng cho modal
const [isOpen, toggleOpen, setOpen] = useToggle(false);
```

**Lưu ý:** trả về `as const` để TypeScript suy ra tuple `[boolean, () => void, ...]` thay vì union array; và dùng functional update trong `toggle` để hàm ổn định, không phụ thuộc giá trị hiện tại.

</details>

---

### 3.12. Rules of Hooks gồm những quy tắc nào? Tại sao bắt buộc phải tuân theo?

<details>
<summary>👉 Xem câu trả lời</summary>

Hai quy tắc chính:

1. **Chỉ gọi hook ở cấp cao nhất** (top level) — KHÔNG gọi trong `if`, vòng lặp, hàm lồng nhau, hay sau một câu `return` sớm.
2. **Chỉ gọi hook từ trong React function component hoặc từ custom hook khác** — không gọi từ hàm JavaScript thường.

**Tại sao:** React không dựa vào tên để nhận diện hook, mà dựa vào THỨ TỰ gọi hook (một danh sách theo index) giữa các lần render. Nếu bạn gọi hook có điều kiện, thứ tự giữa lần render này và lần sau sẽ lệch nhau, khiến React gán nhầm state của hook này cho hook khác — gây bug khó lường.

```tsx
// SAI: hook nằm trong điều kiện -> thứ tự đổi giữa các render
function Bad({ show }: { show: boolean }) {
  if (show) {
    const [name, setName] = useState(''); // vi phạm
  }
  // ...
}

// ĐÚNG: hook luôn ở top level, điều kiện nằm BÊN TRONG hook hoặc ở JSX
function Good({ show }: { show: boolean }) {
  const [name, setName] = useState('');
  useEffect(() => {
    if (!show) return;
    // logic có điều kiện đặt bên trong
  }, [show]);
  return show ? <input value={name} /> : null;
}
```

Dùng `eslint-plugin-react-hooks` để tự động phát hiện vi phạm.

</details>

---

### 3.13. `useLayoutEffect` khác `useEffect` ở điểm nào? Khi nào bắt buộc dùng `useLayoutEffect`?

<details>
<summary>👉 Xem câu trả lời</summary>

Khác biệt nằm ở THỜI ĐIỂM chạy so với việc trình duyệt vẽ (paint):

| | `useEffect` | `useLayoutEffect` |
| --- | --- | --- |
| Thời điểm chạy | Bất đồng bộ, SAU khi trình duyệt paint | Đồng bộ, SAU khi DOM thay đổi nhưng TRƯỚC khi paint |
| Có chặn paint không | Không | Có (chặn cho tới khi chạy xong) |
| Dùng cho | Đa số side effect (fetch, subscribe, log) | Đo đạc layout DOM rồi thay đổi DOM trước khi user thấy |

Dùng `useLayoutEffect` khi bạn cần ĐỌC kích thước/vị trí DOM rồi điều chỉnh ngay, để tránh hiện tượng nhấp nháy (flicker). Ví dụ tính vị trí tooltip dựa trên kích thước phần tử.

```tsx
function Tooltip({ targetRef }) {
  const tipRef = useRef<HTMLDivElement>(null);
  const [pos, setPos] = useState(0);

  useLayoutEffect(() => {
    const rect = tipRef.current!.getBoundingClientRect();
    setPos(/* tính từ rect */); // cập nhật trước khi paint -> không nhấp nháy
  }, []);

  return <div ref={tipRef} style={{ top: pos }} />;
}
```

**Lưu ý:** ưu tiên `useEffect` vì nó không chặn paint; chỉ chuyển sang `useLayoutEffect` khi thấy nhấp nháy. Trong Next.js SSR, `useLayoutEffect` không chạy trên server và sẽ cảnh báo — nếu logic chỉ chạy ở client thì cân nhắc `useEffect` hoặc kỹ thuật isomorphic layout effect.

</details>

---

### 3.14. `useTransition` và `useDeferredValue` dùng để làm gì? Khác nhau ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

Cả hai thuộc tính năng concurrent của React, đánh dấu một cập nhật là "không khẩn cấp" (non-urgent) để giữ UI luôn phản hồi mượt.

- **`useTransition`**: cho bạn bọc một hành động cập nhật state vào `startTransition`. React giữ UI cũ phản hồi và đánh dấu cập nhật đó có thể bị gián đoạn. Trả về cờ `isPending` để hiện trạng thái loading. Dùng khi bạn KIỂM SOÁT chỗ gọi `setState`.

- **`useDeferredValue`**: nhận một giá trị và trả về phiên bản "trễ" của nó; React sẽ render với giá trị cũ trước, rồi cập nhật sang giá trị mới ở một lần render nền. Dùng khi bạn chỉ nhận được GIÁ TRỊ (ví dụ từ prop) mà không kiểm soát được nơi nó được set.

```tsx
// useTransition: gõ ô input mượt, danh sách lọc nặng được hạ ưu tiên
function Search() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState<string[]>([]);
  const [isPending, startTransition] = useTransition();

  function onChange(e: React.ChangeEvent<HTMLInputElement>) {
    setQuery(e.target.value);              // cập nhật khẩn cấp -> input mượt
    startTransition(() => {
      setResults(filterHugeList(e.target.value)); // hạ ưu tiên
    });
  }

  return (
    <>
      <input value={query} onChange={onChange} />
      {isPending && <span>Đang lọc...</span>}
    </>
  );
}

// useDeferredValue: chỉ có giá trị query, không kiểm soát nơi set
function List({ query }: { query: string }) {
  const deferredQuery = useDeferredValue(query);
  const items = useMemo(() => filterHugeList(deferredQuery), [deferredQuery]);
  return <ul>{items.map(i => <li key={i}>{i}</li>)}</ul>;
}
```

**Tóm tắt:** `useTransition` bọc HÀNH ĐỘNG (và cho `isPending`); `useDeferredValue` bọc GIÁ TRỊ. Cả hai không "tăng tốc" việc tính toán mà chỉ thay đổi ĐỘ ƯU TIÊN để phần khẩn cấp (gõ phím) không bị chặn.

</details>

---

### 3.15. `useId` giải quyết vấn đề gì, đặc biệt trong ngữ cảnh SSR của Next.js?

<details>
<summary>👉 Xem câu trả lời</summary>

`useId` sinh ra một id duy nhất, ổn định và KHỚP giữa server và client. Vấn đề nó giải quyết: nếu bạn tự sinh id ngẫu nhiên (ví dụ `Math.random()`) để gắn `htmlFor`/`aria-*`, thì id sinh trên server (SSR) sẽ khác id sinh trên client, gây lỗi **hydration mismatch**. `useId` đảm bảo cả hai phía cho ra cùng một chuỗi.

```tsx
function PasswordField() {
  const id = useId();
  return (
    <>
      <label htmlFor={id}>Mật khẩu</label>
      <input id={id} type="password" aria-describedby={`${id}-hint`} />
      <p id={`${id}-hint`}>Tối thiểu 8 ký tự</p>
    </>
  );
}
```

**Lưu ý quan trọng:**
- `useId` dùng để tạo id cho thuộc tính accessibility/liên kết label–input, KHÔNG dùng làm `key` trong list.
- Một lần gọi có thể tạo nhiều id liên quan bằng cách nối hậu tố (như `${id}-hint`) thay vì gọi `useId` nhiều lần.
- Rất hữu ích trong Next.js App Router (mặc định SSR) và khi viết component thư viện dùng lại.

</details>

---

### 3.16. `useSyncExternalStore` dùng để làm gì? Tại sao nó quan trọng với SSR?

<details>
<summary>👉 Xem câu trả lời</summary>

`useSyncExternalStore` là hook để đăng ký (subscribe) một external store (nguồn dữ liệu nằm NGOÀI React, ví dụ một store tự viết, `localStorage`, hay API trình duyệt như `navigator.onLine`) một cách an toàn với concurrent rendering. Nó nhận `subscribe`, `getSnapshot` (đọc giá trị phía client) và một `getServerSnapshot` tùy chọn (đọc giá trị phía server cho SSR).

Lý do quan trọng: với concurrent rendering, nếu đọc dữ liệu ngoài bằng cách thông thường có thể bị "tearing" (các phần UI thấy giá trị không nhất quán). `useSyncExternalStore` đảm bảo mọi component đọc cùng một snapshot nhất quán. Đây cũng là API mà các thư viện như Zustand, Redux dùng bên dưới.

```tsx
// Theo dõi trạng thái online/offline an toàn cho SSR
function useOnlineStatus() {
  return useSyncExternalStore(
    (callback) => {
      window.addEventListener('online', callback);
      window.addEventListener('offline', callback);
      return () => {
        window.removeEventListener('online', callback);
        window.removeEventListener('offline', callback);
      };
    },
    () => navigator.onLine,   // getSnapshot (client)
    () => true                // getServerSnapshot (SSR) -> tránh hydration mismatch
  );
}
```

**Lưu ý SSR/Next.js:** phải cung cấp `getServerSnapshot` trả về giá trị mặc định an toàn; nếu không, server và client lệch nhau gây hydration mismatch. Đa số trường hợp dùng app thông thường bạn không gọi hook này trực tiếp — nó dành cho tác giả thư viện hoặc khi tích hợp với store/API ngoài React.

</details>

---

### 3.17. `useImperativeHandle` kết hợp `forwardRef` để làm gì? Cho ví dụ.

<details>
<summary>👉 Xem câu trả lời</summary>

Mặc định bạn không thể gắn `ref` lên một function component để gọi trực tiếp. `forwardRef` (React <= 18; trong React 19 `ref` có thể truyền như prop) cho phép component nhận `ref` từ cha. `useImperativeHandle` cho phép component con TÙY BIẾN object mà ref đó trỏ tới — tức là chủ động "lộ" ra một API imperative (như `focus()`, `scrollToTop()`) thay vì lộ toàn bộ DOM node.

Dùng khi cần điều khiển mệnh lệnh từ cha (focus input, reset form, play/pause video) mà vẫn ẩn chi tiết bên trong.

```tsx
type InputHandle = { focus: () => void; clear: () => void };

const FancyInput = forwardRef<InputHandle, { placeholder?: string }>(
  (props, ref) => {
    const inputRef = useRef<HTMLInputElement>(null);

    useImperativeHandle(ref, () => ({
      focus: () => inputRef.current?.focus(),
      clear: () => { if (inputRef.current) inputRef.current.value = ''; },
    }), []); // chỉ lộ ra focus & clear, không lộ toàn bộ DOM

    return <input ref={inputRef} placeholder={props.placeholder} />;
  }
);

// Cha gọi imperative
function Form() {
  const inputRef = useRef<InputHandle>(null);
  return (
    <>
      <FancyInput ref={inputRef} />
      <button onClick={() => inputRef.current?.focus()}>Focus</button>
    </>
  );
}
```

**Bẫy/Lưu ý:** đây là cách tiếp cận imperative, đi ngược tinh thần khai báo (declarative) của React, nên chỉ dùng khi thực sự cần (focus, scroll, media control). Nếu có thể giải quyết bằng prop/state thì ưu tiên cách đó.

</details>
## 4. Tối ưu hiệu năng React

### 4.1. Những nguyên nhân phổ biến gây re-render thừa trong React là gì?

<details>
<summary>👉 Xem câu trả lời</summary>

Một component re-render khi: (1) state của chính nó thay đổi, (2) props thay đổi, (3) component cha re-render (mặc định React render lại toàn bộ cây con), (4) giá trị Context mà nó subscribe thay đổi. Các nguyên nhân gây re-render *thừa* (không cần thiết) thường gặp:

- **Component cha re-render kéo theo cả cây con** dù props của con không đổi về mặt giá trị.
- **Truyền object/array/hàm inline** xuống props: mỗi lần render tạo reference mới, phá vỡ so sánh tham chiếu của `React.memo`.
- **Context value là object mới mỗi render**: mọi consumer re-render dù chỉ một trường thay đổi.
- **State đặt quá cao** (lifted up) khiến một thay đổi nhỏ render lại nhánh lớn.
- **Tạo component mới ngay trong render** (định nghĩa hàm component lồng nhau) làm React mất danh tính component và remount.

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  // Mỗi lần count đổi, Parent render lại -> style và onClick là object/hàm MỚI
  // -> Child sẽ re-render dù logic của nó không liên quan tới count
  return (
    <>
      <button onClick={() => setCount((c) => c + 1)}>{count}</button>
      <Child style={{ margin: 8 }} onClick={() => console.log('hi')} />
    </>
  );
}
```

Bẫy thường gặp: nghĩ rằng "re-render = chậm". Re-render bản thân nó rẻ; vấn đề là khi cây con lớn hoặc mỗi node làm việc nặng (tính toán, DOM lớn). Hãy đo bằng Profiler trước khi tối ưu.

</details>

---

### 4.2. `React.memo` hoạt động thế nào, khi nào nó thực sự hiệu quả và khi nào vô dụng (thậm chí có hại)?

<details>
<summary>👉 Xem câu trả lời</summary>

`React.memo` là một HOC bọc component, bỏ qua re-render nếu props không thay đổi theo **so sánh nông (shallow comparison)**. Nó chỉ chặn được re-render gây ra bởi *cha re-render*, không liên quan tới state nội bộ hay Context của chính component đó.

**Hiệu quả khi:**
- Component render thường xuyên với *cùng props*.
- Component đủ "nặng" (cây con lớn, tính toán đáng kể).
- Props thực sự ổn định về reference (primitive, hoặc object/hàm đã được `useMemo`/`useCallback`).

**Vô dụng / có hại khi:**
- Props thay đổi gần như mỗi lần render (object/hàm inline) -> so sánh luôn fail, vừa tốn chi phí so sánh vừa không chặn được gì.
- Component rất nhẹ -> chi phí so sánh props lớn hơn chi phí render lại.
- Component có `children` là JSX inline -> `children` là object mới mỗi render -> memo vô hiệu.

```jsx
const Row = React.memo(function Row({ item, onSelect }) {
  return <li onClick={() => onSelect(item.id)}>{item.name}</li>;
});

// HỮU ÍCH: onSelect ổn định nhờ useCallback, item ổn định nhờ list không đổi
function List({ items }) {
  const onSelect = useCallback((id) => console.log(id), []);
  return <ul>{items.map((i) => <Row key={i.id} item={i} onSelect={onSelect} />)}</ul>;
}

// VÔ DỤNG: truyền hàm inline -> reference mới mỗi render -> memo không bao giờ ăn
// <Row item={i} onSelect={(id) => console.log(id)} />
```

So sánh tùy biến qua tham số thứ hai `areEqual(prevProps, nextProps)` chỉ nên dùng khi bạn hiểu rõ; deep compare thủ công thường lợi bất cập hại.

</details>

---

### 4.3. Khi nào nên dùng `useMemo` và `useCallback`? Phân biệt hai hook này và những hiểu lầm thường gặp.

<details>
<summary>👉 Xem câu trả lời</summary>

- `useMemo(fn, deps)`: ghi nhớ **giá trị trả về** của `fn`, chỉ tính lại khi `deps` đổi.
- `useCallback(fn, deps)`: ghi nhớ **chính hàm** đó (tương đương `useMemo(() => fn, deps)`).

**Nên dùng khi:**
1. Phép tính thực sự tốn kém (sort/filter/transform tập dữ liệu lớn) — `useMemo`.
2. Giữ ổn định reference của object/hàm khi truyền xuống **component đã memo** hoặc làm **dependency** của hook khác (`useEffect`, `useMemo`) — `useCallback`/`useMemo`.

```jsx
function Table({ rows, query }) {
  // Hợp lý: lọc 10k dòng mỗi render là lãng phí
  const filtered = useMemo(
    () => rows.filter((r) => r.name.includes(query)),
    [rows, query],
  );

  // Hợp lý: handler dùng làm prop cho child đã memo
  const handleClick = useCallback((id) => console.log(id), []);

  return <MemoizedGrid data={filtered} onClick={handleClick} />;
}
```

**Hiểu lầm / bẫy:**
- Memo hóa *mọi thứ* "cho chắc": bản thân memo có chi phí (lưu trữ, so sánh deps) và làm code khó đọc. Nếu giá trị rẻ và không truyền xuống component memo, đừng dùng.
- Quên/sai mảng `deps` -> giá trị cũ (stale closure) hoặc tính lại vô ích.
- `useCallback` *không* chặn re-render nếu component nhận hàm đó chưa được bọc `React.memo`.
- React Compiler (React 19) tự động memo hóa, giảm nhu cầu viết tay; nhưng khi chưa bật, vẫn cần hiểu nguyên lý.

</details>

---

### 4.4. Bạn cần render một danh sách 10.000+ dòng và app bị giật. Bạn xử lý thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Vấn đề là render và giữ hàng chục nghìn DOM node trong cây làm trình duyệt nghẽn (layout, paint, bộ nhớ). Giải pháp chính là **virtualization (windowing)**: chỉ render những item đang nằm trong viewport (cộng một buffer overscan), phần còn lại thay bằng khoảng trống có chiều cao tính sẵn.

Thư viện phổ biến: `react-window`, `react-virtualized`, và `@tanstack/react-virtual` (linh hoạt, headless, hỗ trợ kích thước động và cả grid).

```tsx
import { useVirtualizer } from '@tanstack/react-virtual';

function BigList({ items }: { items: string[] }) {
  const parentRef = useRef<HTMLDivElement>(null);
  const rowVirtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 40, // chiều cao mỗi dòng (px)
    overscan: 8,
  });

  return (
    <div ref={parentRef} style={{ height: 600, overflow: 'auto' }}>
      <div style={{ height: rowVirtualizer.getTotalSize(), position: 'relative' }}>
        {rowVirtualizer.getVirtualItems().map((v) => (
          <div
            key={v.key}
            style={{
              position: 'absolute', top: 0, left: 0, width: '100%',
              height: v.size, transform: `translateY(${v.start}px)`,
            }}
          >
            {items[v.index]}
          </div>
        ))}
      </div>
    </div>
  );
}
```

Bổ sung: kết hợp **phân trang/infinite scroll** (TanStack Query `useInfiniteQuery`) để không tải hết dữ liệu một lần; memo hóa từng dòng; tránh inline style/handler trong dòng. Bẫy: quên đặt chiều cao cho container cuộn (virtualization cần biết viewport), hoặc dùng index làm `key` khi list có thể sắp xếp lại.

</details>

---

### 4.5. Code splitting trong React/Next.js: giải thích `React.lazy`, `Suspense`, và sự khác biệt với `next/dynamic`.

<details>
<summary>👉 Xem câu trả lời</summary>

**Code splitting** chia bundle thành các chunk nhỏ tải theo nhu cầu, giảm JavaScript ban đầu (cải thiện TTI/LCP). Trong React thuần, `React.lazy` nhận một hàm trả về `import()` động và phải được bọc trong `<Suspense>` với `fallback`:

```tsx
import { lazy, Suspense } from 'react';

const Chart = lazy(() => import('./Chart')); // tách thành chunk riêng

function Dashboard() {
  return (
    <Suspense fallback={<Spinner />}>
      <Chart />
    </Suspense>
  );
}
```

`Suspense` cho React tạm "treo" cây con trong khi chunk (hoặc dữ liệu) đang tải và hiển thị fallback.

Trong **Next.js** dùng `next/dynamic` (bọc sẵn `React.lazy` + `Suspense`) với các tùy chọn đặc thù SSR:

```tsx
import dynamic from 'next/dynamic';

// Tắt SSR cho component chỉ chạy ở client (ví dụ dùng window)
const Map = dynamic(() => import('./Map'), {
  ssr: false,
  loading: () => <p>Đang tải bản đồ...</p>,
});
```

Khác biệt chính: `next/dynamic` hỗ trợ `ssr: false` và `loading`, tích hợp với pipeline render của Next. Trong App Router, Server Components vốn không gửi JS xuống client nên một phần "code splitting" diễn ra tự nhiên; `next/dynamic` chủ yếu dùng cho Client Components.

Chiến lược thường gặp: split theo **route** (mỗi trang một chunk) và theo **component nặng/ít dùng** (modal, biểu đồ, editor). Bẫy: split quá nhỏ gây nhiều request lẻ; đặt `Suspense` sai vị trí làm cả màn hình nhấp nháy fallback.

</details>

---

### 4.6. Tại sao `key` ổn định lại quan trọng cho hiệu năng và tính đúng đắn? Tại sao không nên dùng index làm key?

<details>
<summary>👉 Xem câu trả lời</summary>

React dùng `key` trong quá trình **reconciliation** để khớp phần tử cũ với phần tử mới giữa các lần render, từ đó quyết định node nào được giữ lại, di chuyển, thêm hay xóa. Key ổn định và duy nhất giúp React tái sử dụng DOM/instance thay vì hủy và tạo lại — vừa nhanh hơn vừa **giữ đúng state nội bộ** của từng item.

Dùng **index làm key** gây lỗi khi danh sách thay đổi thứ tự, chèn hoặc xóa ở giữa: index gắn với *vị trí* chứ không gắn với *dữ liệu*, nên React khớp nhầm, dẫn tới state/uncontrolled input "nhảy" sai chỗ.

```jsx
// SAI: gõ vào ô input rồi xóa item đầu -> giá trị input bị gán nhầm sang dòng khác
{todos.map((t, i) => <TodoItem key={i} todo={t} />)}

// ĐÚNG: dùng id ổn định gắn với dữ liệu
{todos.map((t) => <TodoItem key={t.id} todo={t} />)}
```

| Tình huống | Index làm key | ID ổn định |
|---|---|---|
| Danh sách tĩnh, không đổi thứ tự | Chấp nhận được | Tốt |
| Có chèn/xóa/sắp xếp | Lỗi state + render thừa | Đúng |
| Item có state nội bộ (input, animation) | Sai lệch | Đúng |

Bẫy: đừng dùng `Math.random()` làm key — key đổi mỗi render khiến React remount toàn bộ, phá hủy hiệu năng và state.

</details>

---

### 4.7. "Colocate state" (đặt state gần nơi dùng) và "lift state up" (nâng state lên) ảnh hưởng tới hiệu năng ra sao? Khi nào chọn cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Phạm vi re-render phụ thuộc vào *nơi state sống*. Khi một state thay đổi, component sở hữu nó và toàn bộ cây con render lại. Vì vậy:

- **Colocate** (đặt state càng gần component dùng nó càng tốt): chỉ một nhánh nhỏ re-render. Tốt cho state cục bộ như nội dung input, trạng thái mở/đóng của một dropdown.
- **Lift up** (nâng state lên tổ tiên chung): cần khi nhiều component anh em phải chia sẻ/đồng bộ state. Nhưng nâng quá cao khiến mỗi thay đổi nhỏ render lại nhánh lớn không liên quan.

```jsx
// VẤN ĐỀ: search state ở App khiến cả <HeavyTree/> render lại mỗi lần gõ phím
function App() {
  const [q, setQ] = useState('');
  return (<><SearchBox value={q} onChange={setQ} /><HeavyTree /></>);
}

// CẢI THIỆN: colocate state vào riêng nhánh tìm kiếm
function App() {
  return (<><SearchSection /><HeavyTree /></>);
}
function SearchSection() {
  const [q, setQ] = useState(''); // chỉ SearchSection render khi gõ
  return <SearchBox value={q} onChange={setQ} />;
}
```

Một kỹ thuật bổ trợ khi buộc phải share: **"lift content, not state"** — truyền phần JSX nặng xuống qua `children` để nó không bị render lại khi state cha đổi (vì `children` đã được tạo từ component trên nữa và giữ nguyên reference).

Quy tắc: bắt đầu bằng colocate, chỉ nâng lên khi thực sự cần chia sẻ; sau đó cân nhắc tách state-container ra để cô lập vùng re-render.

</details>

---

### 4.8. Context API gây re-render hàng loạt thế nào, và làm sao tối ưu nó?

<details>
<summary>👉 Xem câu trả lời</summary>

Mọi component gọi `useContext` sẽ re-render khi **giá trị context (`value`) thay đổi reference** — không có so sánh chọn lọc theo từng trường. Hai lỗi phổ biến:

1. **`value` là object mới mỗi render**: dù dữ liệu không đổi, reference mới làm *tất cả* consumer re-render.
2. **Gộp nhiều thứ vào một Context**: một trường đổi kéo theo mọi consumer của các trường khác cũng render.

```jsx
// SAI: { user, setUser } là object mới mỗi lần Provider render
<AuthContext.Provider value={{ user, setUser }}>{children}</AuthContext.Provider>

// ĐÚNG: memo hóa value
const value = useMemo(() => ({ user, setUser }), [user]);
<AuthContext.Provider value={value}>{children}</AuthContext.Provider>
```

**Các cách tối ưu:**
- **Memo hóa `value`** bằng `useMemo`.
- **Tách Context**: chia state ít đổi và hay đổi thành các provider riêng; hoặc tách phần *giá trị* và phần *dispatcher* (setter/dispatch thường ổn định) thành hai context để consumer chỉ dùng dispatch không bị render khi state đổi.
- **Tách provider thành component riêng** và truyền `children` xuống để children không render lại khi state nội bộ của provider đổi.
- Với state phức tạp/đọc-ghi nhiều, cân nhắc thư viện chuyên dụng (Zustand, Redux Toolkit, Jotai) hỗ trợ **selector** để chỉ subscribe phần dữ liệu cần.

Bẫy: lạm dụng một "MegaContext" cho toàn app là nguyên nhân số 1 gây re-render lan rộng khó debug.

</details>

---

### 4.9. Bạn dùng React DevTools Profiler thế nào để tìm và sửa vấn đề hiệu năng?

<details>
<summary>👉 Xem câu trả lời</summary>

Profiler (tab trong React DevTools) ghi lại các commit và cho biết **component nào render**, **render bao lâu**, và **vì sao render**.

Quy trình thực tế:
1. Bật **"Record why each component rendered"** trong settings của Profiler.
2. Bấm record, thực hiện thao tác bị giật, dừng record.
3. Xem **flame chart** (cây render của một commit, độ rộng = thời gian) và **ranked chart** (xếp hạng component tốn thời gian nhất).
4. Chọn một component để xem lý do render: *props changed*, *hooks changed*, *parent rendered*, *context changed*.
5. Nhận diện re-render thừa (component render dù không cần) -> áp dụng `React.memo`, ổn định props bằng `useMemo`/`useCallback`, colocate/tách context, hoặc virtualization.

```jsx
// Ngoài DevTools, có thể chèn <Profiler> để log số liệu trong code:
import { Profiler } from 'react';

<Profiler id="List" onRender={(id, phase, actualDuration) => {
  console.log(id, phase, actualDuration); // phase: 'mount' | 'update'
}}>
  <List />
</Profiler>
```

Nguyên tắc: **đo trước, tối ưu sau**. Đừng tối ưu mò; dùng Profiler để xác nhận đúng "điểm nóng", và đo lại sau khi sửa để chứng minh cải thiện. Lưu ý chạy ở chế độ production build khi đánh giá con số cuối (development chậm hơn nhiều).

</details>

---

### 4.10. Concurrent rendering trong React 18 là gì? `useTransition` và `useDeferredValue` giúp gì cho hiệu năng cảm nhận được?

<details>
<summary>👉 Xem câu trả lời</summary>

**Concurrent rendering** là khả năng React 18 cho phép render bị **gián đoạn, tạm dừng và nối lại** thay vì luôn render đồng bộ một mạch. Nhờ đó React ưu tiên cập nhật khẩn (gõ phím, click) trước, hoãn cập nhật nặng để giao diện không bị "đơ". Đây là cơ chế nền tảng, được kích hoạt khi dùng `createRoot` và các API như `useTransition`, `useDeferredValue`, Suspense.

- `useTransition`: đánh dấu một cập nhật state là **không khẩn cấp (transition)**. React xử lý nó ở mức ưu tiên thấp, giữ input phản hồi mượt; trả về cờ `isPending` để hiển thị trạng thái.
- `useDeferredValue`: tạo phiên bản **trễ** của một giá trị; phần UI nặng dùng giá trị trễ, còn input dùng giá trị tức thì.

```tsx
function Search() {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();
  const [results, setResults] = useState<string[]>([]);

  function onChange(e: React.ChangeEvent<HTMLInputElement>) {
    setQuery(e.target.value);               // cập nhật khẩn: ô input mượt
    startTransition(() => {
      setResults(filterHuge(e.target.value)); // cập nhật nặng: ưu tiên thấp
    });
  }

  return (
    <>
      <input value={query} onChange={onChange} />
      {isPending && <span>Đang lọc...</span>}
      <ResultList items={results} />
    </>
  );
}
```

Lưu ý: đây là tối ưu **hiệu năng cảm nhận (perceived performance)** — nó không làm phép tính chạy nhanh hơn, chỉ giữ phần ưu tiên cao luôn phản hồi. Bẫy: dùng transition cho cập nhật vốn đã rẻ là thừa; và transition không thay thế cho việc giảm khối lượng tính toán thực (vẫn nên kết hợp memo/virtualization).

</details>

---

### 4.11. Automatic batching ở React 18 là gì? Khác gì so với React 17?

<details>
<summary>👉 Xem câu trả lời</summary>

**Batching** là việc React gom nhiều lần `setState` vào **một lần render** duy nhất thay vì render sau mỗi setState, giảm số lần re-render.

- **React 17 và trước**: chỉ batch các update bên trong **event handler của React**. Các update trong `setTimeout`, Promise (`.then`), native event hay async callback **không** được batch — mỗi setState gây một render riêng.
- **React 18 (với `createRoot`)**: **automatic batching** — gom update ở *mọi nơi*, kể cả trong promise, `setTimeout`, async/await, native handler.

```jsx
function handleClick() {
  // Cả React 17 lẫn 18: 1 lần render (đã batch trong event handler)
  setCount((c) => c + 1);
  setFlag((f) => !f);
}

async function handleAsync() {
  await fetchData();
  // React 17: 2 lần render. React 18: 1 lần render (automatic batching)
  setCount((c) => c + 1);
  setFlag((f) => !f);
}
```

Nếu cần **thoát batching** (buộc React render và cập nhật DOM ngay giữa chừng) có thể dùng `flushSync` từ `react-dom` — nhưng dùng hiếm vì nó làm mất lợi ích gộp render.

```jsx
import { flushSync } from 'react-dom';
flushSync(() => setCount((c) => c + 1)); // render & commit ngay
```

Bẫy nâng cấp: code cũ vô tình dựa vào việc render xảy ra ngay sau mỗi setState trong async có thể đổi hành vi sau khi lên React 18.

</details>

---

### 4.12. Tại sao "tránh object/hàm inline" lại quan trọng, và khi nào điều đó thực sự đáng quan tâm?

<details>
<summary>👉 Xem câu trả lời</summary>

Mỗi lần render, một object/array/arrow function viết inline tạo ra một **reference mới**. Reference mới này:
- Phá vỡ so sánh nông của `React.memo` (props "thay đổi" dù giá trị logic không đổi).
- Làm `useEffect`/`useMemo`/`useCallback` nhận dependency "mới" -> chạy lại không cần thiết.

```jsx
// Hàm/object inline -> reference mới mỗi render
<MemoChild onClick={() => doSomething(id)} config={{ size: 'lg' }} />

// Ổn định reference khi cần truyền xuống component đã memo:
const onClick = useCallback(() => doSomething(id), [id]);
const config = useMemo(() => ({ size: 'lg' }), []);
<MemoChild onClick={onClick} config={config} />
```

**Quan trọng khi:** truyền xuống component đã `React.memo`, hoặc làm dependency của hook, hoặc trong vòng lặp render nhiều item.

**Không đáng lo khi:** component nhận nó *không* được memo và rẻ — vì component đó vốn đã re-render theo cha, ổn định reference chẳng giúp gì mà còn làm code rườm rà. Tạo một object/hàm nhỏ về mặt chi phí GC là cực rẻ; chi phí thật nằm ở re-render dây chuyền mà nó có thể kích hoạt.

Bẫy: tối ưu sớm — bọc mọi handler bằng `useCallback` ngay cả khi không truyền cho component memo. Điều này thêm độ phức tạp mà không đem lại lợi ích đo được. React Compiler (React 19) khi bật sẽ tự lo phần lớn việc này.

</details>

---

### 4.13. Kể về một lần bạn chẩn đoán và khắc phục một vấn đề hiệu năng React trong dự án thực tế.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Đây là câu hỏi hành vi — hãy kể một câu chuyện cụ thể, có số liệu, theo cấu trúc STAR.

- **Situation (Bối cảnh):** "Trong dự án Jira Clone, màn hình board kéo-thả bị giật khi sprint có vài trăm issue. Người dùng phản ánh nhập liệu vào ô tìm kiếm bị trễ rõ rệt."
- **Task (Nhiệm vụ):** "Tôi nhận trách nhiệm giảm độ trễ tương tác xuống mức mượt (dưới ~50ms cho mỗi lần gõ phím) mà không phải viết lại toàn bộ màn hình."
- **Action (Hành động):**
  - "Tôi mở React DevTools Profiler, bật 'record why each component rendered', và thấy mỗi lần gõ phím khiến *toàn bộ* danh sách cột và card re-render."
  - "Nguyên nhân: state tìm kiếm và filter đặt ở component cha cao nhất, cộng với một Context truyền object `value` mới mỗi render. Card lại nhận handler inline."
  - "Tôi colocate state tìm kiếm xuống một component riêng, memo hóa `value` của Context và tách phần dispatch ra context riêng, bọc `Card` bằng `React.memo` với `onClick` ổn định qua `useCallback`."
  - "Với danh sách dài, tôi áp dụng virtualization bằng `@tanstack/react-virtual` và bọc cập nhật lọc trong `useTransition`."
- **Result (Kết quả):** "Số component render mỗi lần gõ phím giảm từ hàng trăm xuống vài chục; thời gian commit theo Profiler giảm đáng kể và giao diện hết giật. Tôi cũng ghi lại checklist tối ưu để cả team dùng lại."

Lời khuyên: nhấn mạnh bạn **đo trước khi sửa** (Profiler), sửa đúng nguyên nhân gốc, rồi **đo lại để xác nhận**. Tránh kể kiểu "tôi bọc useMemo khắp nơi" — nhà tuyển dụng tìm tư duy có phương pháp, không phải tối ưu mò.

</details>
## 5. Quản lý State và Data Fetching

### 5.1. Context API thực sự dùng để làm gì, và đâu là những giới hạn khiến nó không thể thay thế một thư viện state management?

<details>
<summary>👉 Xem câu trả lời</summary>

Context API là cơ chế **truyền dữ liệu xuống cây component mà không cần prop drilling** (truyền prop qua nhiều tầng trung gian). Nó là một công cụ **dependency injection**, KHÔNG phải một công cụ tối ưu hiệu năng.

Giới hạn quan trọng nhất: **mọi component đang `useContext` sẽ re-render khi giá trị `value` của Provider thay đổi reference**, bất kể chúng có dùng tới phần dữ liệu đã đổi hay không. Context không có cơ chế selector để "đăng ký nghe một phần state". Vì vậy, đặt state thay đổi liên tục (vd ô input, vị trí chuột) vào một Context lớn sẽ gây re-render hàng loạt.

Bẫy phổ biến: tạo object `value` mới ngay trong render khiến reference luôn đổi.

```tsx
// ❌ Tệ: mỗi lần ThemeProvider render, value là object mới -> mọi consumer re-render
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// ✅ Tốt hơn: memo hóa value
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const value = useMemo(() => ({ theme, setTheme }), [theme]);
  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}
```

Khi nào dùng Context: dữ liệu **ít thay đổi**, mang tính "toàn cục đọc nhiều" như theme, locale, user đã đăng nhập, theme provider của styled-components. Khi state thay đổi thường xuyên hoặc có logic phức tạp, dùng Zustand/Redux/Jotai (chúng có selector và chỉ re-render component thật sự cần).

Một mẹo giảm re-render: **tách Context** thành Context cho giá trị và Context cho hàm dispatch (hàm dispatch ổn định nên consumer của nó gần như không re-render).

</details>

---

### 5.2. Phân biệt "server state" và "client state". Vì sao việc tách bạch hai loại này lại quan trọng khi chọn công cụ?

<details>
<summary>👉 Xem câu trả lời</summary>

| Tiêu chí | Client state | Server state |
|---|---|---|
| Sở hữu | Do client tạo và nắm giữ hoàn toàn | "Mượn" từ server, server mới là nguồn sự thật |
| Tính tươi mới | Luôn đúng (đồng bộ ngay) | Có thể **bị cũ (stale)** vì người khác đổi dữ liệu |
| Bất đồng bộ | Đồng bộ, tức thời | Bất đồng bộ, có loading/error |
| Ví dụ | Modal đang mở, tab đang chọn, giá trị form chưa submit | Danh sách bài viết, hồ sơ user, đơn hàng từ API |

Tách bạch quan trọng vì **chúng cần công cụ khác nhau**:

- Client state hợp với `useState`/`useReducer`/Zustand/Jotai/Redux: đơn giản, đồng bộ.
- Server state có những bài toán riêng mà thư viện client state không giải được: caching, dedupe request trùng, làm mới nền (background refetch), retry, biết khi nào dữ liệu cũ, đồng bộ giữa nhiều tab. Đây chính là lý do **TanStack Query / SWR / RTK Query** ra đời.

Sai lầm kinh điển: nhét dữ liệu API vào Redux rồi tự viết action `fetchUsersStart/Success/Failure`, tự quản loading, tự cache. Việc này tốn rất nhiều boilerplate và thường vẫn thiếu (không dedupe, không stale-while-revalidate). Một thư viện server state chuyên dụng làm điều đó tốt hơn nhiều với ít code hơn.

```tsx
// Server state: dùng TanStack Query, không nhồi vào store client
const { data: posts } = useQuery({ queryKey: ['posts'], queryFn: fetchPosts });

// Client state: useState/Zustand là đủ
const [isSidebarOpen, setSidebarOpen] = useState(false);
```

</details>

---

### 5.3. Redux Toolkit giải quyết những "đau đầu" gì của Redux truyền thống? Giải thích `createSlice` và immutable update bằng Immer.

<details>
<summary>👉 Xem câu trả lời</summary>

Redux gốc có quá nhiều boilerplate: tự định nghĩa action type (chuỗi hằng), action creator, reducer với `switch`, tự cấu hình store + middleware. Redux Toolkit (RTK) là cách viết Redux **được khuyến nghị chính thức**, gói gọn các pattern phổ biến.

`createSlice` tự sinh ra **action creators và action types** từ tên các reducer, nên không cần khai báo thủ công. Bên trong reducer, RTK dùng **Immer** nên bạn có thể "viết như mutate" trên một bản nháp (draft), Immer sẽ tạo ra state mới bất biến phía sau. Lưu ý: chỉ được mutate `state` (draft) bên trong reducer của createSlice/createReducer — viết kiểu mutate ở nơi khác vẫn sai.

```ts
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

interface CartState { items: { id: string; qty: number }[] }

const cartSlice = createSlice({
  name: 'cart',
  initialState: { items: [] } as CartState,
  reducers: {
    addItem(state, action: PayloadAction<string>) {
      // "viết như mutate" nhờ Immer, thực tế tạo state mới bất biến
      const found = state.items.find((i) => i.id === action.payload);
      if (found) found.qty += 1;
      else state.items.push({ id: action.payload, qty: 1 });
    },
    clear(state) {
      state.items = []; // an toàn nhờ Immer
    },
  },
});

export const { addItem, clear } = cartSlice.actions; // action tự sinh
export default cartSlice.reducer;
```

`configureStore` cũng tự bật Redux DevTools và các middleware mặc định (vd kiểm tra serializable, kiểm tra mutate ngoài ý muốn) trong môi trường dev.

</details>

---

### 5.4. RTK Query là gì? Nó khác `createAsyncThunk` ở điểm nào, và khi nào nên chọn RTK Query thay vì TanStack Query?

<details>
<summary>👉 Xem câu trả lời</summary>

**RTK Query** là lớp data-fetching/caching được tích hợp sẵn trong Redux Toolkit. Bạn định nghĩa các endpoint, RTK Query tự sinh ra **React hooks** (`useGetPostsQuery`, `useAddPostMutation`...), tự quản caching, dedupe, loading/error, invalidation qua hệ thống **tag**, và lưu cache ngay trong Redux store.

Khác với `createAsyncThunk`: thunk chỉ là cách viết một async action; bạn vẫn phải tự viết reducer để lưu data, tự quản loading, tự cache thủ công. RTK Query làm hết những việc đó.

```ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['Post'],
  endpoints: (builder) => ({
    getPosts: builder.query<Post[], void>({
      query: () => 'posts',
      providesTags: ['Post'],
    }),
    addPost: builder.mutation<Post, NewPost>({
      query: (body) => ({ url: 'posts', method: 'POST', body }),
      invalidatesTags: ['Post'], // tự refetch getPosts sau khi thêm
    }),
  }),
});

export const { useGetPostsQuery, useAddPostMutation } = api;
```

Khi nào chọn cái nào:
- **RTK Query**: dự án **đã dùng Redux** cho client state, muốn một hệ sinh thái thống nhất, hoặc thích cách định nghĩa endpoint tập trung.
- **TanStack Query**: không muốn dùng Redux, muốn thư viện server state độc lập, linh hoạt hơn về cache key và các tính năng nâng cao (infinite query, optimistic mượt mà, devtools mạnh). Đây thường là lựa chọn mặc định ngày nay cho React/Next.js khi không gắn với Redux.

Lưu ý: cả hai làm chung một nhóm việc (server state). **Đừng dùng cả hai cùng lúc** cho cùng một mục đích.

</details>

---

### 5.5. Zustand hoạt động thế nào? Vì sao nó tránh được vấn đề re-render của Context, và `selector` quan trọng ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

Zustand là thư viện state nhỏ gọn, dựa trên hook và **không cần Provider bao quanh cây component**. Store là một hook; component đăng ký nghe state qua một **selector** và chỉ re-render khi **phần state mà selector trả về** thay đổi (so sánh theo `Object.is` mặc định). Đây là điểm khác biệt mấu chốt so với Context: Context khiến mọi consumer re-render khi value đổi, còn Zustand cho phép đăng ký chọn lọc.

```ts
import { create } from 'zustand';

interface BearStore {
  bears: number;
  fish: number;
  addBear: () => void;
}

const useStore = create<BearStore>((set) => ({
  bears: 0,
  fish: 0,
  addBear: () => set((s) => ({ bears: s.bears + 1 })),
}));

// ✅ Chỉ re-render khi bears đổi, fish đổi không ảnh hưởng
const bears = useStore((s) => s.bears);
const addBear = useStore((s) => s.addBear);
```

Bẫy thường gặp: trả về một **object mới** trong selector khiến luôn khác reference -> re-render mỗi lần.

```ts
// ❌ Object mới mỗi render -> luôn re-render
const { bears, fish } = useStore((s) => ({ bears: s.bears, fish: s.fish }));

// ✅ Dùng useShallow để so sánh nông
import { useShallow } from 'zustand/react/shallow';
const { bears, fish } = useStore(useShallow((s) => ({ bears: s.bears, fish: s.fish })));
```

Zustand hợp cho **client state toàn cục** vừa và nhỏ: nhẹ, ít boilerplate, không Provider. Nhưng nó **không phải công cụ server state** — vẫn nên dùng TanStack Query cho dữ liệu API.

</details>

---

### 5.6. Jotai và Recoil tiếp cận state theo mô hình "atom". Mô hình này khác gì so với store tập trung kiểu Redux/Zustand?

<details>
<summary>👉 Xem câu trả lời</summary>

Redux/Zustand theo mô hình **top-down / store tập trung**: có một (vài) store lớn, component đọc ra một lát cắt. Jotai/Recoil theo mô hình **bottom-up / atomic**: state được chia thành nhiều **atom** nhỏ độc lập; component đăng ký đúng atom nó cần. Mô hình này gần với `useState` nhưng có thể chia sẻ giữa các component và **kết hợp (derive)** với nhau.

```ts
import { atom, useAtom } from 'jotai';

const countAtom = atom(0);
// derived atom: tự tính lại khi countAtom đổi, KHÔNG lưu trùng dữ liệu
const doubledAtom = atom((get) => get(countAtom) * 2);

function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const [doubled] = useAtom(doubledAtom);
  return <button onClick={() => setCount((c) => c + 1)}>{count} -> {doubled}</button>;
}
```

Ưu điểm atomic: chỉ component dùng atom đó mới re-render (giải bài toán re-render dư thừa rất tự nhiên), và **derived atom** giúp tính state phái sinh mà không lưu trùng. Jotai rất nhẹ, API gần với `useState` nên dễ học; Recoil là dự án của Facebook nhưng hiện gần như không còn được phát triển tích cực, nên thực tế **ưu tiên Jotai** hơn cho dự án mới.

Khi nào chọn atomic: khi state phân mảnh, nhiều mảnh nhỏ liên quan nhau, hoặc bạn muốn cách tư duy "state như những ô tính toán" thay vì một store lớn. Với app có ít state toàn cục đơn giản, Zustand thường gọn hơn.

</details>

---

### 5.7. Trong TanStack Query, hãy giải thích `staleTime` và `gcTime` (trước đây là `cacheTime`). Chúng kiểm soát điều gì?

<details>
<summary>👉 Xem câu trả lời</summary>

Đây là hai khái niệm dễ nhầm nhưng kiểm soát hai thứ hoàn toàn khác nhau:

- **`staleTime`**: khoảng thời gian dữ liệu được coi là **còn tươi (fresh)**. Trong khoảng này, query sẽ **không tự refetch** (kể cả khi component mount lại, focus cửa sổ...). Mặc định là `0`, nghĩa là dữ liệu bị coi là stale ngay lập tức -> rất hay refetch nền.
- **`gcTime`** (garbage collection time, mặc định 5 phút): khi một query **không còn component nào dùng** (inactive), dữ liệu của nó được giữ trong cache thêm bằng `gcTime` rồi mới bị dọn. Nó quyết định dữ liệu cũ còn sống bao lâu để dùng lại tức thì khi quay lại.

```tsx
const { data } = useQuery({
  queryKey: ['posts'],
  queryFn: fetchPosts,
  staleTime: 60_000,   // trong 1 phút coi là tươi, không refetch nền
  gcTime: 5 * 60_000,  // sau khi không dùng, giữ cache thêm 5 phút
});
```

Tư duy thực dụng: dữ liệu ít đổi (danh mục, hồ sơ) đặt `staleTime` cao để tránh refetch thừa; dữ liệu thay đổi nhanh (giá realtime) để `staleTime` thấp. `gcTime` chỉ ảnh hưởng tới chuyện "khi quay lại, có dữ liệu cũ để hiển thị ngay trong lúc fetch lại hay không".

Bẫy: nhầm `staleTime` với `gcTime` rồi than phiền "đặt cacheTime rồi mà vẫn refetch hoài" — refetch nền do `staleTime`, không phải `gcTime`.

</details>

---

### 5.8. `queryKey` đóng vai trò gì trong TanStack Query? Giải thích cơ chế invalidation và vì sao nó tốt hơn việc tự refetch thủ công.

<details>
<summary>👉 Xem câu trả lời</summary>

`queryKey` là **danh tính (identity) của một query trong cache**, đồng thời là **dependency**: khi key đổi, query tự fetch lại với tham số mới. Key là một mảng và nên chứa mọi tham số ảnh hưởng tới kết quả.

```tsx
// Key gồm tham số -> đổi page hay filter là tự fetch lại đúng dữ liệu
useQuery({
  queryKey: ['posts', { page, status }],
  queryFn: () => fetchPosts({ page, status }),
});
```

**Invalidation**: sau khi dữ liệu thay đổi (vd vừa tạo bài viết), ta đánh dấu các query liên quan là stale để chúng refetch:

```tsx
const queryClient = useQueryClient();

const mutation = useMutation({
  mutationFn: createPost,
  onSuccess: () => {
    // mọi query có key bắt đầu bằng ['posts'] sẽ được làm mới
    queryClient.invalidateQueries({ queryKey: ['posts'] });
  },
});
```

Vì sao tốt hơn tự refetch thủ công:
- Invalidation hoạt động theo **prefix matching**: `['posts']` khớp cả `['posts', { page: 1 }]`, `['posts', { page: 2 }]`... nên không cần biết hết các biến thể đang cache.
- Query Client biết query nào đang **active** để refetch ngay, query inactive sẽ refetch khi được dùng lại.
- Tránh được việc bạn phải tự gọi lại đúng hàm fetch ở đúng chỗ — vốn dễ sót và khó bảo trì khi nhiều component dùng chung dữ liệu.

</details>

---

### 5.9. Optimistic update là gì? Hãy mô tả luồng đúng để làm optimistic update an toàn (kể cả khi lỗi) trong TanStack Query.

<details>
<summary>👉 Xem câu trả lời</summary>

**Optimistic update** là cập nhật UI **ngay lập tức** theo kết quả mong đợi trước khi server phản hồi, để cảm giác tức thời (vd nút "like"). Nếu server lỗi, ta phải **rollback** về trạng thái cũ.

Luồng an toàn dùng `onMutate / onError / onSettled`:

```tsx
const mutation = useMutation({
  mutationFn: toggleLike,
  // 1. Chạy trước khi gọi server
  onMutate: async (postId) => {
    // Hủy refetch đang chạy để nó không ghi đè update lạc quan
    await queryClient.cancelQueries({ queryKey: ['post', postId] });
    // Lưu lại snapshot để rollback nếu lỗi
    const previous = queryClient.getQueryData(['post', postId]);
    // Cập nhật cache ngay
    queryClient.setQueryData(['post', postId], (old: Post) => ({
      ...old,
      liked: !old.liked,
    }));
    return { previous }; // context truyền sang onError
  },
  // 2. Nếu lỗi -> rollback bằng snapshot
  onError: (_err, postId, context) => {
    if (context?.previous) {
      queryClient.setQueryData(['post', postId], context.previous);
    }
  },
  // 3. Dù thành công hay lỗi -> đồng bộ lại với server
  onSettled: (_data, _err, postId) => {
    queryClient.invalidateQueries({ queryKey: ['post', postId] });
  },
});
```

Ba điểm dễ quên mà người phỏng vấn hay hỏi:
1. **`cancelQueries`** ở `onMutate` để một refetch đang bay không ghi đè lên giá trị lạc quan.
2. **Lưu snapshot** và trả về qua `context` để `onError` rollback chính xác.
3. **`onSettled` invalidate** để cuối cùng UI vẫn khớp với nguồn sự thật của server.

</details>

---

### 5.10. Infinite query khác gì với phân trang thông thường trong TanStack Query? Khi nào dùng `useInfiniteQuery`?

<details>
<summary>👉 Xem câu trả lời</summary>

Phân trang thường (`useQuery` với key chứa `page`) thay thế dữ liệu trang này bằng trang khác — hợp với UI có nút "Trang trước/Trang sau". `useInfiniteQuery` **tích lũy nhiều trang vào một cấu trúc `pages`**, hợp với "Tải thêm" hoặc cuộn vô hạn (infinite scroll).

Điểm cốt lõi là `getNextPageParam`: từ trang cuối đã tải, hàm này trả về tham số cho trang kế tiếp (cursor hoặc số trang). Trả về `undefined` nghĩa là hết dữ liệu.

```tsx
const {
  data,
  fetchNextPage,
  hasNextPage,
  isFetchingNextPage,
} = useInfiniteQuery({
  queryKey: ['posts'],
  queryFn: ({ pageParam }) => fetchPosts({ cursor: pageParam }),
  initialPageParam: null as string | null,
  getNextPageParam: (lastPage) => lastPage.nextCursor ?? undefined,
});

// data.pages là mảng các trang -> gộp lại để render
const allPosts = data?.pages.flatMap((p) => p.items) ?? [];
```

Khi nào dùng: feed mạng xã hội, danh sách dài tải dần khi cuộn, "Load more". Thường kết hợp với `IntersectionObserver` để gọi `fetchNextPage()` khi người dùng cuộn tới cuối.

Bẫy: quên `flatMap` các `pages` rồi render nhầm cấu trúc; hoặc đặt `getNextPageParam` sai khiến `hasNextPage` luôn `true` -> tải vô tận.

</details>

---

### 5.11. SWR là gì và nó khác TanStack Query ở điểm nào? Khi nào bạn cân nhắc SWR?

<details>
<summary>👉 Xem câu trả lời</summary>

**SWR** (do Vercel phát triển) là thư viện data-fetching dựa trên chiến lược **stale-while-revalidate**: trả về dữ liệu cũ trong cache ngay (hiển thị nhanh), đồng thời refetch nền để cập nhật. Tên thư viện chính là tên chiến lược đó.

So sánh nhanh:

| Tiêu chí | SWR | TanStack Query |
|---|---|---|
| Tác giả | Vercel (gắn liền hệ sinh thái Next.js) | TanStack |
| API | Rất tối giản, `useSWR(key, fetcher)` | Phong phú hơn, nhiều option |
| Tính năng nâng cao | Đủ dùng (mutate, infinite) | Mạnh hơn: devtools, mutation API chi tiết, optimistic rõ ràng, query cancellation |
| Triết lý | Nhẹ, tối giản | Đầy đủ, "đồ nghề" cho ca phức tạp |

```tsx
import useSWR from 'swr';

const fetcher = (url: string) => fetch(url).then((r) => r.json());

function Profile() {
  const { data, error, isLoading } = useSWR('/api/user', fetcher);
  if (isLoading) return <Spinner />;
  if (error) return <ErrorBox />;
  return <span>{data.name}</span>;
}
```

Khi nào cân nhắc SWR: dự án **Next.js** muốn thứ tối giản, ít cấu hình, fetch đọc là chính. Khi cần mutation phức tạp, optimistic update tinh vi, infinite query nhiều biến thể, devtools mạnh — TanStack Query thường thoải mái hơn. Cả hai đều tốt; điểm chung quan trọng là **cả hai đều coi đây là server state**, đừng tự cache thủ công trong Redux nữa.

</details>

---

### 5.12. "Chuẩn hóa dữ liệu" (data normalization) là gì? Khi nào cần và khi nào không?

<details>
<summary>👉 Xem câu trả lời</summary>

**Chuẩn hóa** là lưu dữ liệu dạng phẳng, mỗi thực thể chỉ tồn tại **một bản duy nhất**, tham chiếu nhau bằng `id` — giống bảng trong cơ sở dữ liệu quan hệ. Trái ngược là dữ liệu **lồng nhau (nested/denormalized)** dễ bị trùng lặp một thực thể ở nhiều nơi.

```ts
// ❌ Lồng nhau: cùng một user xuất hiện ở nhiều post -> sửa tên phải sửa nhiều chỗ
const posts = [
  { id: 'p1', title: 'A', author: { id: 'u1', name: 'An' } },
  { id: 'p2', title: 'B', author: { id: 'u1', name: 'An' } },
];

// ✅ Chuẩn hóa: mỗi thực thể một bản, tham chiếu bằng id
const state = {
  posts: { p1: { id: 'p1', title: 'A', authorId: 'u1' }, p2: { id: 'p2', title: 'B', authorId: 'u1' } },
  users: { u1: { id: 'u1', name: 'An' } },
};
```

Khi nào cần: store **client tập trung** (Redux) với dữ liệu quan hệ, hay cập nhật một thực thể và muốn mọi nơi tham chiếu nó cùng cập nhật. Redux Toolkit có `createEntityAdapter` hỗ trợ sẵn dạng `{ ids: [], entities: {} }`.

Khi nào KHÔNG cần: nếu bạn dùng **TanStack Query / RTK Query**, cache vốn được tổ chức theo `queryKey` và invalidation, nên thường **không cần tự chuẩn hóa** — bạn chỉ cần invalidate đúng query để mọi chỗ lấy lại dữ liệu mới. Tự chuẩn hóa thủ công khi không thực sự cần chỉ làm tăng độ phức tạp.

</details>

---

### 5.13. "Derived state" là gì, và vì sao việc lưu state phái sinh vào `useState` thường là một anti-pattern?

<details>
<summary>👉 Xem câu trả lời</summary>

**Derived state** (state phái sinh) là dữ liệu **có thể tính ra được** từ state hoặc props đã có. Nguyên tắc: **đừng lưu thứ tính được**. Nếu lưu vào `useState` riêng, bạn tạo ra **nguồn sự thật trùng lặp**, dễ lệch nhau và sinh bug khi quên đồng bộ.

```tsx
// ❌ Anti-pattern: lưu filteredItems vào state, phải tự đồng bộ qua useEffect
const [items, setItems] = useState<Item[]>([]);
const [query, setQuery] = useState('');
const [filtered, setFiltered] = useState<Item[]>([]);
useEffect(() => {
  setFiltered(items.filter((i) => i.name.includes(query)));
}, [items, query]); // dễ quên dependency, render thừa một nhịp

// ✅ Tính trực tiếp khi render
const filtered = items.filter((i) => i.name.includes(query));

// ✅ Nếu phép tính nặng, memo hóa
const filtered = useMemo(
  () => items.filter((i) => i.name.includes(query)),
  [items, query],
);
```

Tư duy: chỉ đưa vào state những gì **thực sự là nguồn sự thật tối thiểu**. Mọi thứ suy ra được thì tính trong render (memo hóa nếu tốn kém). Điều này tránh được cả một lớp bug "state không khớp nhau" và những `useEffect` đồng bộ thừa thãi.

Ngoại lệ hiếm: đôi khi cần lưu "giá trị trước đó" hoặc snapshot, nhưng đó là chủ ý rõ ràng, không phải derived state thông thường.

</details>

---

### 5.14. Vì sao đôi khi nên lưu state trên URL thay vì trong React state? Trong Next.js bạn làm điều này thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

State liên quan tới **"trạng thái có thể chia sẻ / khôi phục được"** — bộ lọc, từ khóa tìm kiếm, số trang, tab đang chọn, sắp xếp — nên đặt trên URL (query string). Lợi ích:

- **Chia sẻ và bookmark được**: dán link là người khác thấy đúng kết quả lọc.
- **Sống sót qua reload**: F5 không mất trạng thái.
- **Nút back/forward của trình duyệt hoạt động đúng**.
- **Một nguồn sự thật duy nhất** thay vì đồng bộ giữa state và URL.

Trong Next.js App Router, dùng `useSearchParams` để đọc và `useRouter` để cập nhật:

```tsx
'use client';
import { useRouter, useSearchParams, usePathname } from 'next/navigation';

function SearchBox() {
  const router = useRouter();
  const pathname = usePathname();
  const searchParams = useSearchParams();
  const q = searchParams.get('q') ?? '';

  const onChange = (value: string) => {
    const params = new URLSearchParams(searchParams);
    if (value) params.set('q', value);
    else params.delete('q');
    router.replace(`${pathname}?${params.toString()}`); // replace để không spam history
  };

  return <input value={q} onChange={(e) => onChange(e.target.value)} />;
}
```

Với React Router (SPA), dùng `useSearchParams` tương tự. Một mẹo hay là **kết hợp URL state với `queryKey` của TanStack Query**: đọc filter từ URL rồi đưa vào `queryKey` — đổi URL là tự fetch lại đúng dữ liệu.

Bẫy: nhồi mọi thứ vào URL kể cả state tạm thời nhạy cảm (vd token, dữ liệu form chưa submit) hoặc dùng `push` thay `replace` cho thay đổi gõ phím khiến lịch sử trình duyệt đầy rác.

</details>

---

### 5.15. Hãy đưa ra "cây quyết định" của bạn: gặp một nhu cầu state, bạn chọn công cụ nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Câu hỏi này kiểm tra **tư duy kiến trúc**, không có đáp án duy nhất. Một khung lập luận tốt:

1. **Đây là server state hay client state?**
   - Server state (dữ liệu từ API): dùng **TanStack Query** (hoặc SWR, hoặc RTK Query nếu đã có Redux). Không tự cache thủ công.
   - Client state: đi tiếp các bước dưới.

2. **State có thể suy ra được không?** Nếu có -> **derived state**, tính trong render (memo nếu nặng), không lưu.

3. **State có nên ở trên URL không?** Filter/search/page/tab -> đặt vào **URL** (useSearchParams).

4. **Phạm vi state tới đâu?**
   - Chỉ trong một component -> `useState` / `useReducer`.
   - Vài component gần nhau -> nâng state lên (lift) hoặc truyền props.
   - Toàn cục nhưng ít đổi (theme, user) -> **Context** (đã memo value).
   - Toàn cục thay đổi thường xuyên, cần selector -> **Zustand** (nhẹ) hoặc **Jotai** (atomic).
   - App lớn, nhiều logic, đã có hệ sinh thái Redux, cần devtools/time-travel, middleware -> **Redux Toolkit**.

```text
Có phải dữ liệu từ server?  -> TanStack Query / SWR / RTK Query
Tính được từ state khác?    -> derived (useMemo)
Nên chia sẻ qua URL?        -> useSearchParams
Cục bộ?                     -> useState / useReducer
Toàn cục ít đổi?            -> Context (memo value)
Toàn cục hay đổi, cần selector? -> Zustand / Jotai
App lớn, cần devtools/middleware? -> Redux Toolkit
```

Thông điệp gửi người phỏng vấn: **bắt đầu từ công cụ đơn giản nhất** (`useState`), chỉ nâng cấp khi có lý do rõ ràng (re-render thừa, prop drilling sâu, cần chia sẻ rộng), và **luôn tách server state khỏi client state**.

</details>

---

### 5.16. Kể về một lần bạn refactor cách quản lý state trong một dự án. Vấn đề ban đầu là gì và bạn cải thiện ra sao?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Câu hỏi hành vi — dùng cấu trúc STAR (Situation, Task, Action, Result). Đây là một mẫu trả lời bạn có thể điều chỉnh theo trải nghiệm thật:

- **Situation (Bối cảnh)**: "Dự án trang quản trị (admin dashboard) dùng Redux cho mọi thứ, kể cả dữ liệu API. Mỗi màn hình phải tự viết action `fetchStart/Success/Failure`, tự quản loading và cache. Code phình to, và có bug dữ liệu cũ vì hai màn hình cùng dùng một danh sách nhưng không đồng bộ khi một bên cập nhật."

- **Task (Nhiệm vụ)**: "Tôi được giao giảm boilerplate và xử lý triệt để vấn đề dữ liệu lệch nhau giữa các màn hình, đồng thời không phá vỡ tính năng đang chạy."

- **Action (Hành động)**: "Tôi đề xuất tách bạch **server state** và **client state**. Tôi đưa toàn bộ dữ liệu API sang **TanStack Query**, dùng `queryKey` thống nhất và `invalidateQueries` sau mỗi mutation để mọi màn hình tự làm mới. Phần client state còn lại thật sự nhỏ (modal, bước wizard) nên tôi chuyển sang **Zustand** với selector để tránh re-render thừa, và bỏ bớt Redux. Tôi làm theo từng module, có test cho luồng quan trọng trước khi gỡ code cũ."

- **Result (Kết quả)**: "Lượng code quản lý data giảm đáng kể, bug dữ liệu cũ biến mất vì chỉ còn một cơ chế invalidation, và trải nghiệm nhanh hơn nhờ caching và background refetch. Quan trọng hơn, đồng đội về sau thêm màn hình mới nhanh hơn vì pattern đã rõ ràng."

Mẹo: nhấn vào **lý do kỹ thuật** (tách server/client state) và **kết quả đo được** (ít code hơn, hết bug đồng bộ, nhanh hơn), thay vì chỉ kể "đổi thư viện".

</details>
## 6. React Ecosystem và Thư viện

### 6.1. So sánh React Router v6 với v5. Những thay đổi quan trọng nào bạn cần chú ý?

<details>
<summary>👉 Xem câu trả lời</summary>

React Router v6 viết lại nhiều API so với v5. Những điểm khác biệt quan trọng nhất:

| | v5 | v6 |
|---|---|---|
| Khai báo route | `<Switch>` chọn route đầu tiên match | `<Routes>` chọn route **phù hợp nhất** (ranking) |
| Render component | `component`/`render`/`children` props | luôn dùng `element={<Comp />}` |
| Lồng route | `<Route path>` tự ghép thủ công | hỗ trợ **nested routes** + `<Outlet />` |
| Điều hướng bằng code | `useHistory()` (`history.push`) | `useNavigate()` (`navigate('/path')`) |
| Path matching | cần `exact` | mặc định match chính xác, không cần `exact` |
| Relative path | đường dẫn tuyệt đối | route con dùng **relative path** so với cha |

```tsx
import { Routes, Route, Outlet } from 'react-router-dom';

function App() {
  return (
    <Routes>
      <Route path="/" element={<Layout />}>
        <Route index element={<Home />} />
        <Route path="users/:id" element={<UserDetail />} />
        <Route path="*" element={<NotFound />} />
      </Route>
    </Routes>
  );
}

// Layout là route cha — chỗ <Outlet /> sẽ render route con
function Layout() {
  return (
    <div>
      <Navbar />
      <Outlet />
    </div>
  );
}
```

**Bẫy thường gặp:** dùng `path="users/:id"` trong nested route nhưng quên đặt `<Outlet />` ở component cha → route con không hiển thị gì cả. Một lỗi khác là đặt `path="/users"` (có dấu `/` đầu) cho route con khiến nó thành đường dẫn tuyệt đối thay vì lồng vào cha.

</details>

---

### 6.2. Làm sao đọc dynamic params và query string trong React Router? Phân biệt `useParams`, `useSearchParams`, `useLocation`.

<details>
<summary>👉 Xem câu trả lời</summary>

- **`useParams()`** — đọc các tham số động khai báo bằng `:` trong path (vd `/users/:id`). Trả về object, giá trị luôn là **string**.
- **`useSearchParams()`** — đọc/ghi query string (`?page=2&sort=asc`). Trả về cặp `[searchParams, setSearchParams]` tương tự `useState`, trong đó `searchParams` là một `URLSearchParams`.
- **`useLocation()`** — trả về object location hiện tại (`pathname`, `search`, `hash`, `state`). Hữu ích để theo dõi URL hoặc đọc `state` truyền qua `navigate`.

```tsx
import { useParams, useSearchParams, useLocation } from 'react-router-dom';

function ProductList() {
  const { category } = useParams();        // /products/:category
  const [searchParams, setSearchParams] = useSearchParams();
  const location = useLocation();

  const page = Number(searchParams.get('page') ?? 1);

  const nextPage = () => {
    // setSearchParams nhận object hoặc URLSearchParams
    setSearchParams({ page: String(page + 1) });
  };

  console.log(location.pathname); // "/products/shoes"
  return <button onClick={nextPage}>Trang {page}</button>;
}
```

**Bẫy thường gặp:** quên rằng params và search params đều là **string** — phải `Number(...)` thủ công khi cần số. Ngoài ra `setSearchParams` sẽ **ghi đè toàn bộ** query string; nếu muốn giữ các param khác phải merge thủ công từ `searchParams` cũ.

</details>

---

### 6.3. `loader` và `action` trong React Router (data router) là gì? Chúng giải quyết vấn đề gì so với fetch trong `useEffect`?

<details>
<summary>👉 Xem câu trả lời</summary>

Từ React Router v6.4 trở đi (data router với `createBrowserRouter`), router hỗ trợ **`loader`** và **`action`** gắn trực tiếp vào route:

- **`loader`**: hàm chạy **trước khi** route render để nạp dữ liệu. Dữ liệu lấy bằng `useLoaderData()`. Vì loader chạy song song với việc khớp route, ta tránh được "fetch waterfall" và **render khi đã có data** (không còn cảnh loading nhấp nháy như fetch trong `useEffect`).
- **`action`**: hàm xử lý mutation (submit form). Khi `<Form method="post">` submit, router gọi action, sau đó **tự động revalidate** các loader liên quan để dữ liệu trên UI luôn mới.

```tsx
import { createBrowserRouter, useLoaderData, Form, redirect } from 'react-router-dom';

const router = createBrowserRouter([
  {
    path: '/users/:id',
    element: <UserPage />,
    loader: async ({ params }) => {
      const res = await fetch(`/api/users/${params.id}`);
      if (!res.ok) throw new Response('Not Found', { status: 404 });
      return res.json(); // dữ liệu này -> useLoaderData()
    },
    action: async ({ request, params }) => {
      const form = await request.formData();
      await fetch(`/api/users/${params.id}`, {
        method: 'PUT',
        body: JSON.stringify(Object.fromEntries(form)),
      });
      return redirect(`/users/${params.id}`);
    },
  },
]);

function UserPage() {
  const user = useLoaderData() as User; // data đã sẵn sàng khi render
  return (
    <Form method="post">
      <input name="name" defaultValue={user.name} />
      <button type="submit">Lưu</button>
    </Form>
  );
}
```

**Khi nào dùng:** loader/action hợp khi bạn dùng React Router làm "framework data layer" cho SPA. Nếu dự án đã dùng **TanStack Query** hoặc là **Next.js** (đã có cơ chế data fetching riêng) thì thường không cần loader/action — chọn một mô hình nhất quán, đừng trộn lẫn.

</details>

---

### 6.4. Cách triển khai một Protected Route (yêu cầu đăng nhập) trong React Router?

<details>
<summary>👉 Xem câu trả lời</summary>

Ý tưởng: tạo một component "bảo vệ" — nếu chưa đăng nhập thì `<Navigate>` về trang login (kèm lưu vị trí muốn vào), nếu đã đăng nhập thì render `<Outlet />` cho route con.

```tsx
import { Navigate, Outlet, useLocation } from 'react-router-dom';

function RequireAuth() {
  const { user } = useAuth();
  const location = useLocation();

  if (!user) {
    // replace để người dùng bấm Back không quay lại trang protected
    // state lưu nơi định vào -> sau khi login điều hướng lại
    return <Navigate to="/login" replace state={{ from: location }} />;
  }
  return <Outlet />;
}

// Khai báo route
<Routes>
  <Route element={<RequireAuth />}>
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/settings" element={<Settings />} />
  </Route>
  <Route path="/login" element={<Login />} />
</Routes>;

// Trong Login, sau khi đăng nhập thành công:
const navigate = useNavigate();
const location = useLocation();
const from = location.state?.from?.pathname ?? '/';
navigate(from, { replace: true });
```

**Lưu ý quan trọng về bảo mật:** protected route chỉ là **bảo vệ phía client** (UX) — bất kỳ ai cũng có thể gọi API trực tiếp. Việc phân quyền thật sự **phải** nằm ở backend. Với data router, bạn cũng có thể chặn ngay trong `loader` bằng cách `throw redirect('/login')` nếu chưa đăng nhập, giúp chặn trước cả khi component render.

</details>

---

### 6.5. Tại sao React Hook Form có hiệu năng tốt hơn so với cách quản lý form bằng state thông thường?

<details>
<summary>👉 Xem câu trả lời</summary>

Cách form truyền thống là "controlled component": mỗi input gắn `value` + `onChange` vào state, nên **mỗi lần gõ phím là một lần `setState` → re-render toàn bộ form**. Form lớn (nhiều field) sẽ chậm và lag.

React Hook Form (RHF) dùng cách tiếp cận **uncontrolled** dựa trên `ref`:

- Dùng `register` để gắn input vào RHF qua `ref` (DOM giữ giá trị), thay vì đưa giá trị vào React state.
- Vì không setState mỗi lần gõ → **gõ phím không gây re-render** component. RHF chỉ re-render khi cần (vd khi validate, khi trạng thái submit thay đổi).
- Form **isolate** việc cập nhật, nên hiệu năng gần như không phụ thuộc số lượng field.

```tsx
import { useForm } from 'react-hook-form';

function LoginForm() {
  const { register, handleSubmit, formState: { errors } } = useForm();

  const onSubmit = (data) => console.log(data);

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {/* register trả về ref + name + onChange/onBlur, spread vào input */}
      <input {...register('email', { required: 'Bắt buộc nhập email' })} />
      {errors.email && <span>{errors.email.message}</span>}

      <input type="password" {...register('password')} />
      <button>Đăng nhập</button>
    </form>
  );
}
```

**Tóm tắt:** ít re-render hơn → nhanh hơn; ít code boilerplate hơn; tích hợp validation sẵn. Đánh đổi: với input không phải DOM gốc (component thư viện UI) phải dùng `Controller`/`control` (xem câu sau).

</details>

---

### 6.6. Phân biệt `register` và `control` (Controller) trong React Hook Form. Khi nào dùng cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

| | `register` | `Controller` / `control` |
|---|---|---|
| Cơ chế | uncontrolled, gắn qua `ref` | controlled, RHF tự đăng ký & cập nhật |
| Dùng cho | input HTML gốc (`input`, `select`, `textarea`) | component UI không expose `ref`/`value` chuẩn (MUI Select, react-select, Antd DatePicker...) |
| Hiệu năng | tối ưu nhất (không re-render khi gõ) | re-render field khi giá trị đổi (chỉ field đó) |
| Cách dùng | `{...register('name')}` | bọc `<Controller render={...} />` |

```tsx
import { useForm, Controller } from 'react-hook-form';
import Select from 'react-select';

function ProfileForm() {
  const { register, control, handleSubmit } = useForm();

  return (
    <form onSubmit={handleSubmit(console.log)}>
      {/* input gốc -> register */}
      <input {...register('username')} />

      {/* component custom không nhận ref chuẩn -> Controller */}
      <Controller
        name="country"
        control={control}
        rules={{ required: true }}
        render={({ field }) => <Select {...field} options={options} />}
      />
    </form>
  );
}
```

**Quy tắc:** ưu tiên `register` cho input gốc để tận dụng hiệu năng; chỉ dùng `Controller` khi component bên thứ ba không hoạt động với `ref` (cần `value` + `onChange` được kiểm soát). Bẫy thường gặp: nhồi mọi thứ vào `Controller` dù không cần, làm mất ưu thế hiệu năng của RHF.

</details>

---

### 6.7. Tích hợp Zod (hoặc Yup) với React Hook Form như thế nào? Ưu điểm của schema validation?

<details>
<summary>👉 Xem câu trả lời</summary>

RHF có sẵn validation cơ bản (`required`, `min`, `pattern`...) nhưng với form phức tạp ta nên tách validation ra một **schema** dùng Zod/Yup, kết nối qua `@hookform/resolvers`.

Ưu điểm lớn nhất của Zod là **type inference**: định nghĩa schema một lần, suy ra luôn TypeScript type (`z.infer`) — không phải khai báo type hai lần, và schema có thể tái dùng cho validate ở cả client lẫn server.

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email('Email không hợp lệ'),
  password: z.string().min(8, 'Tối thiểu 8 ký tự'),
  confirm: z.string(),
}).refine((d) => d.password === d.confirm, {
  message: 'Mật khẩu nhập lại không khớp',
  path: ['confirm'], // gắn lỗi vào field confirm
});

type FormValues = z.infer<typeof schema>; // type tự suy ra từ schema

function SignUp() {
  const { register, handleSubmit, formState: { errors } } =
    useForm<FormValues>({ resolver: zodResolver(schema) });

  return (
    <form onSubmit={handleSubmit(console.log)}>
      <input {...register('email')} />
      <p>{errors.email?.message}</p>
      {/* ... */}
    </form>
  );
}
```

**So với Yup:** Zod được thiết kế ưu tiên TypeScript (inference tốt hơn), không cần thư viện type bổ sung; Yup ra đời sớm hơn, phổ biến rộng nhưng inference yếu hơn. Cả hai đều validate được logic chéo giữa các field (như khớp mật khẩu) thông qua `refine` (Zod) hoặc `test`/`when` (Yup).

</details>

---

### 6.8. So sánh các phương án styling: CSS Modules, styled-components/Emotion (CSS-in-JS), và Tailwind CSS. Trade-off của mỗi cách?

<details>
<summary>👉 Xem câu trả lời</summary>

| Tiêu chí | CSS Modules | styled-components / Emotion | Tailwind CSS |
|---|---|---|---|
| Cách viết | file `.module.css`, class scoped | viết CSS trong JS, tạo component | utility class ngay trên JSX |
| Scope | tự động (class hash hóa) | tự động (theo component) | toàn cục nhưng class utility cố định |
| Dynamic theo props | khó (phải toggle class) | mạnh (template literal theo props) | qua biến class / `cva` |
| Hiệu năng runtime | không (CSS tĩnh build sẵn) | **có runtime cost** (trừ bản zero-runtime) | không (CSS sinh lúc build, purge) |
| RSC (Next.js App Router) | hỗ trợ tốt | nhiều thư viện CSS-in-JS gặp khó với Server Components | hỗ trợ tốt |
| Đường cong học | thấp | trung bình | phải nhớ tên utility |

```tsx
// CSS Modules
import styles from './Button.module.css';
<button className={styles.primary}>Lưu</button>

// styled-components — style theo props
const Button = styled.button<{ $primary?: boolean }>`
  background: ${(p) => (p.$primary ? '#2563eb' : '#e5e7eb')};
`;

// Tailwind — utility ngay trên JSX
<button className="bg-blue-600 text-white px-4 py-2 rounded">Lưu</button>
```

**Khi nào dùng cái nào:**
- **Tailwind**: prototyping nhanh, design system thống nhất, hợp Next.js App Router (không runtime cost). Nhược điểm: JSX dài, cần làm quen.
- **CSS-in-JS (styled-components/Emotion)**: cần style động mạnh theo props/theme, đóng gói component. Nhược điểm: runtime overhead, vướng víu với React Server Components — nhiều team Next.js mới đang dịch chuyển khỏi nó.
- **CSS Modules**: muốn CSS thuần, scope an toàn, không phụ thuộc runtime — lựa chọn "an toàn" và nhẹ.

</details>

---

### 6.9. `clsx`/`classnames` và `cva` (class-variance-authority) dùng để làm gì? Vì sao cần chúng khi dùng Tailwind?

<details>
<summary>👉 Xem câu trả lời</summary>

Khi dùng Tailwind, việc ghép class **có điều kiện** rất phổ biến, và viết bằng template string dễ lỗi (thừa khoảng trắng, khó đọc).

- **`clsx`** (hoặc `classnames`): nối class có điều kiện gọn gàng. Nhận string, object `{ class: boolean }`, array — bỏ qua giá trị falsy.
- **`cva`** (class-variance-authority): định nghĩa các **variant** của một component (size, intent...) một cách có cấu trúc, suy ra luôn type cho props. Rất hợp để xây UI component có nhiều biến thể (chính shadcn/ui dùng cva).
- Thường kết hợp thêm **`tailwind-merge`** (`twMerge`) để xử lý xung đột class Tailwind: ví dụ `px-2` và `px-4` cùng xuất hiện thì giữ cái sau.

```tsx
import { clsx } from 'clsx';
import { cva, type VariantProps } from 'class-variance-authority';
import { twMerge } from 'tailwind-merge';

// clsx: class có điều kiện
<button className={clsx('btn', { 'btn--active': isActive, 'opacity-50': disabled })} />;

// cva: định nghĩa variant có cấu trúc
const button = cva('rounded font-medium', {
  variants: {
    intent: { primary: 'bg-blue-600 text-white', ghost: 'bg-transparent' },
    size: { sm: 'px-2 py-1 text-sm', lg: 'px-4 py-2 text-base' },
  },
  defaultVariants: { intent: 'primary', size: 'sm' },
});

type ButtonProps = VariantProps<typeof button>;
function Button({ intent, size, className }: ButtonProps & { className?: string }) {
  // twMerge để override an toàn khi class trùng nhóm
  return <button className={twMerge(button({ intent, size }), className)} />;
}
```

**Tóm lại:** `clsx` cho ghép class có điều kiện đơn giản; `cva` cho component có hệ thống variant; `tailwind-merge` để dedup/override class trùng. Bộ ba này là nền tảng cách shadcn/ui tổ chức component.

</details>

---

### 6.10. Framer Motion (nay là Motion) dùng để làm gì? Cách tạo animation cơ bản, và `AnimatePresence` giải quyết vấn đề gì?

<details>
<summary>👉 Xem câu trả lời</summary>

Framer Motion là thư viện animation khai báo cho React. Thay vì viết CSS keyframe thủ công, ta mô tả trạng thái và để thư viện lo phần chuyển động (spring, tween) và tính toán hiệu năng.

Khái niệm cốt lõi:
- **`motion.*`**: thành phần có thể animate (`motion.div`, `motion.button`...).
- **`initial` / `animate` / `exit`**: trạng thái lúc mount, lúc hiển thị, và lúc unmount.
- **`transition`**: cấu hình kiểu chuyển động (duration, type spring...).
- **`variants`**: gom các trạng thái có tên, hỗ trợ `staggerChildren` để animate con tuần tự.
- **`AnimatePresence`**: cho phép animate khi component **bị gỡ khỏi cây** (unmount) — điều mà React không hỗ trợ sẵn, vì bình thường component biến mất ngay lập tức không kịp chạy exit animation.

```tsx
import { motion, AnimatePresence } from 'framer-motion';

function Toast({ show, message }: { show: boolean; message: string }) {
  return (
    <AnimatePresence>
      {show && (
        <motion.div
          initial={{ opacity: 0, y: 20 }}
          animate={{ opacity: 1, y: 0 }}
          exit={{ opacity: 0, y: 20 }}     // chạy được nhờ AnimatePresence
          transition={{ type: 'spring', stiffness: 300 }}
        >
          {message}
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

**Bẫy thường gặp:** quên bọc `AnimatePresence` → `exit` animation không bao giờ chạy (component biến mất tức thì). Khi render danh sách trong `AnimatePresence`, mỗi item **phải có `key` ổn định** thì Motion mới theo dõi được item nào bị gỡ.

</details>

---

### 6.11. So sánh thư viện component MUI, Radix UI và shadcn/ui. Triết lý "headless" và "copy-paste component" của shadcn khác gì?

<details>
<summary>👉 Xem câu trả lời</summary>

| | MUI (Material UI) | Radix UI | shadcn/ui |
|---|---|---|---|
| Loại | component đầy đủ + style sẵn | **headless** (chỉ logic + accessibility, không style) | bộ component build sẵn trên Radix + Tailwind |
| Phân phối | cài qua npm, dùng như dependency | cài qua npm (primitives) | **copy code vào repo của bạn** (qua CLI), không phải dependency |
| Tùy biến style | qua theme/`sx`, đôi khi gò bó | tự do hoàn toàn (bạn tự style) | sửa trực tiếp code đã copy |
| Quyền kiểm soát | thấp (chạy theo design system của MUI) | cao | cao nhất (bạn sở hữu code) |

- **Headless** nghĩa là thư viện lo phần **khó**: hành vi, quản lý focus, accessibility (ARIA, bàn phím), nhưng **không áp đặt giao diện** — bạn tự style. Radix là ví dụ điển hình (Dialog, Dropdown, Popover... đều accessible sẵn).
- **shadcn/ui không phải một package npm** mà là cách phân phối: bạn dùng CLI để **copy mã nguồn component vào dự án**. Vì bạn sở hữu code, bạn sửa thoải mái, không bị khóa version, không lo breaking change từ dependency. Nó dùng Radix (logic/a11y) + Tailwind + cva (variant) bên dưới.

```tsx
// shadcn/ui: component được copy vào src/components/ui — bạn toàn quyền sửa
import { Button } from '@/components/ui/button';
<Button variant="destructive" size="sm">Xóa</Button>;

// Radix headless: tự lo style, thư viện chỉ lo hành vi + a11y
import * as Dialog from '@radix-ui/react-dialog';
<Dialog.Root>
  <Dialog.Trigger>Mở</Dialog.Trigger>
  <Dialog.Portal>
    <Dialog.Overlay className="..." />
    <Dialog.Content className="...">Nội dung</Dialog.Content>
  </Dialog.Portal>
</Dialog.Root>;
```

**Khi nào dùng cái nào:** MUI hợp khi cần ra sản phẩm nhanh, chấp nhận giao diện Material và ít tùy biến sâu. Radix/shadcn hợp khi cần **design system riêng**, kiểm soát hoàn toàn UI và muốn accessibility chuẩn mà không tự viết lại logic phức tạp.

</details>

---

### 6.12. Storybook dùng để làm gì? Nó đem lại lợi ích gì cho quy trình phát triển frontend?

<details>
<summary>👉 Xem câu trả lời</summary>

Storybook là công cụ phát triển và tài liệu hóa **component một cách độc lập** (isolation) — chạy riêng từng component ở mọi trạng thái, không cần khởi động cả ứng dụng hay điều hướng tới đúng màn hình mới thấy được component.

Mỗi "story" mô tả một component ở một trạng thái cụ thể (loading, error, empty, có data...). Lợi ích:
- **Phát triển isolation**: dựng UI không phụ thuộc backend/state thật.
- **Tài liệu sống** cho design system: dev và designer cùng xem được mọi biến thể.
- **Visual testing / regression**: kết hợp Chromatic để bắt thay đổi giao diện ngoài ý muốn.
- **Test tương tác & accessibility**: addon a11y, play function mô phỏng tương tác người dùng.

```tsx
// Button.stories.tsx (Component Story Format - CSF 3)
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

const meta: Meta<typeof Button> = {
  component: Button,
  title: 'UI/Button',
};
export default meta;

type Story = StoryObj<typeof Button>;

export const Primary: Story = { args: { variant: 'primary', children: 'Lưu' } };
export const Disabled: Story = { args: { disabled: true, children: 'Lưu' } };
export const Loading: Story = { args: { loading: true } };
```

**Tóm lại:** Storybook đặc biệt giá trị cho team xây **component library / design system**, giúp tách biệt việc phát triển UI khỏi business logic, và làm cầu nối giữa designer - developer - QA.

</details>

---

### 6.13. Triển khai đa ngôn ngữ (i18n) trong React/Next.js với react-i18next như thế nào? Những điểm cần lưu ý?

<details>
<summary>👉 Xem câu trả lời</summary>

`react-i18next` (xây trên `i18next`) là giải pháp i18n phổ biến cho React. Nó quản lý các bộ resource dịch theo namespace, đổi ngôn ngữ runtime, và cung cấp hook `useTranslation`.

```tsx
import { useTranslation } from 'react-i18next';

function Greeting({ name }: { name: string }) {
  const { t, i18n } = useTranslation();

  return (
    <div>
      <p>{t('greeting', { name })}</p>          {/* "Xin chào, {{name}}" */}
      <p>{t('cart.items', { count })}</p>        {/* plural tự động theo count */}
      <button onClick={() => i18n.changeLanguage('en')}>EN</button>
    </div>
  );
}

// resource (vi.json):  { "greeting": "Xin chào, {{name}}", "cart": { "items_one": "{{count}} món", "items_other": "{{count}} món" } }
```

Những điểm cần lưu ý:
- **Interpolation & pluralization**: dùng `{{var}}` và quy tắc số nhiều (`_one`/`_other`) thay vì ghép chuỗi thủ công.
- **Namespace + lazy load**: tách file dịch theo trang/feature và nạp theo nhu cầu để giảm bundle.
- **Định dạng số/ngày/tiền tệ**: nên dùng `Intl` (NumberFormat, DateTimeFormat) theo locale, không hardcode.
- **Với Next.js App Router**: `react-i18next` chạy tốt phía client; cho i18n ở **Server Components** thường dùng `next-intl` hoặc cấu hình i18next phía server, vì hook chỉ chạy ở client component. Cần thiết kế chiến lược locale routing (vd `/en/...`, `/vi/...`).
- **Tránh** nhồi mọi chuỗi vào một file khổng lồ và **đừng** dịch bằng cách nối chuỗi (sai ngữ pháp ở nhiều ngôn ngữ).

</details>

---

### 6.14. TanStack Table giải quyết vấn đề gì? Vì sao nói nó là thư viện "headless"?

<details>
<summary>👉 Xem câu trả lời</summary>

TanStack Table (trước đây là React Table) là thư viện **headless** để xây bảng dữ liệu phức tạp: sorting, filtering, pagination, grouping, row selection, virtualization... Nó lo toàn bộ **logic và state** của bảng nhưng **không render ra markup hay style nào** — bạn tự render `<table>`/`<div>` và tự style (Tailwind, CSS...).

Vì sao "headless" lại có lợi:
- Tự do hoàn toàn về giao diện, không bị áp layout (khác với data grid "full" như AG Grid hay MUI DataGrid).
- Nhẹ, không kéo theo CSS thừa; framework-agnostic (có bản cho Vue, Svelte, Solid).

```tsx
import {
  useReactTable, getCoreRowModel, getSortedRowModel, flexRender,
  type ColumnDef,
} from '@tanstack/react-table';

const columns: ColumnDef<User>[] = [
  { accessorKey: 'name', header: 'Tên' },
  { accessorKey: 'email', header: 'Email' },
];

function UsersTable({ data }: { data: User[] }) {
  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
    getSortedRowModel: getSortedRowModel(), // bật sorting
  });

  return (
    <table>
      <thead>
        {table.getHeaderGroups().map((hg) => (
          <tr key={hg.id}>
            {hg.headers.map((h) => (
              <th key={h.id} onClick={h.column.getToggleSortingHandler()}>
                {flexRender(h.column.columnDef.header, h.getContext())}
              </th>
            ))}
          </tr>
        ))}
      </thead>
      <tbody>
        {table.getRowModel().rows.map((row) => (
          <tr key={row.id}>
            {row.getVisibleCells().map((cell) => (
              <td key={cell.id}>{flexRender(cell.column.columnDef.cell, cell.getContext())}</td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

**Khi nào dùng:** chọn TanStack Table khi cần bảng tùy biến giao diện cao và kiểm soát logic. Với bảng cực lớn nên kết hợp **TanStack Virtual** để chỉ render hàng trong viewport. Nếu cần một data grid "đóng gói sẵn UI + tính năng enterprise" mà không muốn tự render thì AG Grid / MUI DataGrid phù hợp hơn.

</details>

---

### 6.15. Khi chọn một thư viện cho dự án (vd state management hay form), bạn dựa trên những tiêu chí nào để quyết định?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Đây là câu hành vi đánh giá tư duy kỹ thuật khi đưa quyết định. Trả lời theo STAR:

- **Situation**: nêu một bối cảnh cụ thể, ví dụ team cần chọn giải pháp quản lý server state cho dự án dashboard đang phình to, hiện đang fetch trong `useEffect` rải rác và khó cache.
- **Task**: vai trò của bạn là đánh giá và đề xuất thư viện (vd TanStack Query vs Redux Toolkit Query vs tự viết), thuyết phục team.
- **Action** — nêu các tiêu chí đánh giá có hệ thống:
  1. **Bài toán có thật sự cần thư viện không** — tránh thêm dependency khi vài dòng code tự xử lý được.
  2. **Mức độ phù hợp** với vấn đề (server state khác hẳn client state).
  3. **Sức khỏe dự án**: số lượng maintainer, tần suất release, số issue tồn đọng, lần commit gần nhất.
  4. **Cộng đồng & tài liệu**: dễ tìm giải pháp, dễ tuyển/onboard người mới.
  5. **Bundle size** và ảnh hưởng tới hiệu năng/Core Web Vitals.
  6. **TypeScript support** và DX.
  7. **Tính tương thích** với stack hiện tại (vd React Server Components nếu dùng Next.js App Router).
  8. **Chi phí migration & lock-in**: rút lui có dễ không.
- **Result + Bài học**: nêu kết quả (chọn được giải pháp giảm code fetch, có cache/retry sẵn, team onboard nhanh) và bài học: quyết định dựa trên **dữ liệu và trade-off rõ ràng**, làm POC nhỏ trước khi cam kết toàn dự án thay vì chọn theo "thư viện đang hot".

👉 Điểm ăn: thể hiện bạn cân nhắc **trade-off**, ưu tiên giải pháp đơn giản nhất giải quyết được bài toán, và biết đánh giá sức khỏe/độ rủi ro của dependency thay vì chạy theo trend.

</details>
## 7. Next.js và Cơ chế Rendering

### 7.1. Hãy phân biệt CSR, SSR, SSG và ISR. Khi nào bạn chọn mỗi chiến lược?

<details>
<summary>👉 Xem câu trả lời</summary>

Đây là bốn chiến lược rendering chính. Điểm khác biệt cốt lõi là **HTML được tạo ra ở đâu và khi nào**.

| Chiến lược | HTML tạo ra khi nào | Nơi tạo | Dữ liệu | Khi nào dùng |
|---|---|---|---|---|
| **CSR** (Client-Side Rendering) | Trong trình duyệt, sau khi tải JS | Client | Fetch sau khi mount | Dashboard sau đăng nhập, trang nội bộ không cần SEO |
| **SSR** (Server-Side Rendering) | Mỗi request | Server | Lấy mới mỗi lần | Dữ liệu thay đổi liên tục, cá nhân hóa (giỏ hàng, feed) |
| **SSG** (Static Site Generation) | Tại thời điểm build | Build server | Cố định lúc build | Blog, docs, landing page ít thay đổi |
| **ISR** (Incremental Static Regeneration) | Build + tạo lại nền theo chu kỳ | Build + server | Cố định nhưng tự làm mới | Trang sản phẩm e-commerce, tin tức (cần SEO + dữ liệu khá mới) |

ISR là dạng lai: bạn được tốc độ và CDN của trang tĩnh, nhưng trang sẽ được **tạo lại ở nền (regenerate)** sau khoảng `revalidate` giây, nên dữ liệu không bao giờ "cũ vĩnh viễn".

```tsx
// App Router — SSG (mặc định khi không có dynamic data)
export default async function Page() {
  const data = await fetch('https://api.example.com/posts'); // cache mặc định -> tĩnh
  return <List items={await data.json()} />;
}

// App Router — ISR: tạo lại sau mỗi 60s
export const revalidate = 60;

// App Router — SSR thuần: không cache, render mỗi request
export const dynamic = 'force-dynamic';
// hoặc fetch(url, { cache: 'no-store' })
```

**Cách chọn nhanh:** SEO + dữ liệu hiếm thay đổi → SSG; SEO + dữ liệu thay đổi nhưng chấp nhận trễ vài giây → ISR; cần dữ liệu thời gian thực hoặc cá nhân hóa từng người → SSR; không cần SEO, nội dung phía sau auth → CSR.

**Bẫy thường gặp:** nhiều người nghĩ SSR luôn "tốt hơn" CSR. Thực tế SSR tốn tài nguyên server mỗi request và TTFB chậm hơn trang tĩnh; nếu không cần SEO/cá nhân hóa thì SSG/ISR (CDN) thường tối ưu hơn nhiều.

</details>

---

### 7.2. App Router khác Pages Router ở những điểm cốt lõi nào? Có nên migrate dự án cũ không?

<details>
<summary>👉 Xem câu trả lời</summary>

App Router (thư mục `app/`, ổn định từ Next 13.4) là kiến trúc mới dựa trên **React Server Components**, còn Pages Router (`pages/`) là kiến trúc truyền thống dựa trên data-fetching functions.

| Khía cạnh | Pages Router (`pages/`) | App Router (`app/`) |
|---|---|---|
| Mô hình component | Toàn bộ là Client Component | Server Component mặc định |
| Data fetching | `getServerSideProps`, `getStaticProps`, `getInitialProps` | `async`/`await` trực tiếp trong component, `fetch` mở rộng |
| Layout chung | `_app.tsx` + manual | `layout.tsx` lồng nhau, giữ state khi điều hướng |
| Loading/Error | Tự xử lý | File quy ước: `loading.tsx`, `error.tsx` |
| Streaming/Suspense | Hạn chế | Hỗ trợ native (Suspense, streaming SSR) |
| API | `pages/api/*` | Route Handlers `app/**/route.ts` |
| Mutation | API route + fetch thủ công | Server Actions |

```tsx
// Pages Router
export async function getServerSideProps() {
  const data = await getData();
  return { props: { data } };
}
export default function Page({ data }) { /* ... */ }

// App Router — tương đương, gọn hơn
export default async function Page() {
  const data = await getData(); // chạy trên server
  return <View data={data} />;
}
```

**Có nên migrate không?** Hai router có thể **cùng tồn tại trong một dự án** (incremental adoption) — Next sẽ ưu tiên `app/` cho route trùng. Nên migrate dần (route mới viết trong `app/`) thay vì viết lại toàn bộ. Tuy nhiên App Router có learning curve cao hơn (RSC, ranh giới server/client, caching phức tạp), nên với dự án nhỏ và ổn định, Pages Router vẫn là lựa chọn hợp lệ.

</details>

---

### 7.3. Server Components và Client Components khác nhau thế nào? Chỉ thị `'use client'` thực sự làm gì?

<details>
<summary>👉 Xem câu trả lời</summary>

Trong App Router, **mọi component mặc định là Server Component (RSC)** — chỉ chạy trên server, không bao giờ gửi JS của nó xuống client. `'use client'` đánh dấu một file (và toàn bộ cây import từ nó) là Client Component — được hydrate và chạy trên trình duyệt.

| | Server Component | Client Component (`'use client'`) |
|---|---|---|
| Nơi chạy | Chỉ server (lúc render) | Server (SSR ban đầu) + Client (hydrate) |
| Truy cập DB/secret | Có (an toàn) | Không nên (lộ ra bundle) |
| Dùng `useState`/`useEffect` | Không | Có |
| Event handler (`onClick`...) | Không | Có |
| Dùng hooks browser (`window`) | Không | Có |
| Gửi JS xuống client | Không | Có (tăng bundle) |

```tsx
// app/page.tsx — Server Component (mặc định)
import db from '@/lib/db';
import LikeButton from './LikeButton';

export default async function Page() {
  const post = await db.post.findFirst(); // OK: chạy trên server
  return (
    <article>
      <h1>{post.title}</h1>
      <LikeButton postId={post.id} /> {/* phần tương tác tách riêng */}
    </article>
  );
}
```

```tsx
// app/LikeButton.tsx — Client Component
'use client';
import { useState } from 'react';

export default function LikeButton({ postId }: { postId: string }) {
  const [liked, setLiked] = useState(false);
  return <button onClick={() => setLiked(!liked)}>{liked ? '❤️' : '🤍'}</button>;
}
```

**Điểm quan trọng / bẫy:**
- `'use client'` **không** nghĩa là "render hoàn toàn ở client". Client Component vẫn được SSR lần đầu để có HTML, rồi mới hydrate.
- Một khi đánh dấu `'use client'`, mọi thứ component đó **import** cũng thành client. Nên đặt `'use client'` càng "sâu" trong cây (lá) càng tốt để giữ bundle nhỏ.
- Bạn **không thể** import Server Component vào trong Client Component bằng `import`, nhưng có thể truyền nó qua props `children` (composition pattern).
- Props truyền từ Server → Client Component phải **serializable** (không truyền được function, class instance, Date phức tạp...).

</details>

---

### 7.4. Hydration là gì? Hydration mismatch xảy ra vì sao và làm sao tránh?

<details>
<summary>👉 Xem câu trả lời</summary>

**Hydration** là quá trình React "gắn" logic JS vào HTML tĩnh do server gửi xuống: nó dựng lại virtual DOM, attach event listeners và làm trang trở nên tương tác — mà không vẽ lại DOM. Để làm được, React **giả định cây HTML do client render lần đầu phải khớp y hệt HTML server đã sinh**.

**Hydration mismatch** xảy ra khi HTML server và render đầu tiên ở client khác nhau. React sẽ cảnh báo (và trong các trường hợp nặng có thể vứt bỏ HTML server, render lại từ đầu → chớp màn hình, mất hiệu năng).

Nguyên nhân phổ biến:
- Dùng giá trị **không ổn định giữa server và client**: `Date.now()`, `Math.random()`, `new Date().toLocaleString()` (khác timezone/locale server vs client).
- Đọc `window`/`localStorage` ngay trong render (chỉ có ở client).
- Theme/dark-mode đọc từ localStorage khiến server render "light" còn client render "dark".
- HTML không hợp lệ do trình duyệt tự sửa: ví dụ `<div>` lồng trong `<p>`, hoặc `<table>` thiếu `<tbody>`.

```tsx
// ❌ SAI — gây mismatch: server và client cho thời gian khác nhau
function Clock() {
  return <span>{new Date().toLocaleTimeString()}</span>;
}

// ✅ ĐÚNG — render giá trị "an toàn" ở server, cập nhật sau khi mount
'use client';
import { useState, useEffect } from 'react';

function Clock() {
  const [time, setTime] = useState<string | null>(null);
  useEffect(() => {
    setTime(new Date().toLocaleTimeString()); // chỉ chạy ở client
  }, []);
  return <span>{time ?? '—'}</span>; // server và client đều render '—' lần đầu
}
```

**Cách tránh:** giữ render lần đầu thuần khiết và không phụ thuộc môi trường; hoãn dữ liệu chỉ-có-ở-client sang `useEffect`; với widget hoàn toàn không thể SSR, dùng `next/dynamic` với `ssr: false`. Trong trường hợp cần thiết và biết rõ một node sẽ khác, có thể dùng `suppressHydrationWarning` (ví dụ timestamp), nhưng đây là giải pháp cuối cùng chứ không phải để "giấu" lỗi.

</details>

---

### 7.5. Trong App Router, `fetch` được mở rộng thế nào với `cache`, `no-store` và `revalidate`?

<details>
<summary>👉 Xem câu trả lời</summary>

Next.js bọc (patch) hàm `fetch` gốc để thêm khả năng caching và revalidation. Hành vi được điều khiển qua option thứ hai:

```tsx
// 1) Cache vĩnh viễn (mặc định trong nhiều trường hợp) -> dữ liệu tĩnh (SSG)
await fetch('https://api/posts', { cache: 'force-cache' });

// 2) Không cache -> render động mỗi request (SSR)
await fetch('https://api/cart', { cache: 'no-store' });

// 3) Time-based revalidation (ISR): cache, làm mới sau 60s
await fetch('https://api/products', { next: { revalidate: 60 } });

// 4) Tag-based revalidation: gắn tag để chủ động làm mới khi có thay đổi
await fetch('https://api/products', { next: { tags: ['products'] } });
```

Với cách số 4, ở Server Action hoặc Route Handler bạn có thể chủ động xóa cache:

```tsx
import { revalidateTag, revalidatePath } from 'next/cache';

revalidateTag('products');        // làm mới mọi fetch gắn tag 'products'
revalidatePath('/products/[id]'); // làm mới theo đường dẫn
```

**Lưu ý về phiên bản:** mặc định caching thay đổi đáng kể giữa các bản Next.

- Next 13/14: `fetch` mặc định **được cache** (`force-cache`), và route mặc định cố gắng render tĩnh.
- Next 15: mặc định đổi sang **không cache** (`no-store`) cho `fetch`, và `GET` Route Handler không còn cache mặc định — bạn phải opt-in caching một cách tường minh.

Vì thế trong phỏng vấn, nên nói rõ "tùy phiên bản" và nhấn mạnh nguyên tắc: dùng `revalidate` cho dữ liệu tolerant-stale, `no-store` cho dữ liệu nhạy thời gian/cá nhân hóa, và tag để invalidate có chủ đích sau mutation.

</details>

---

### 7.6. `generateStaticParams` dùng để làm gì? Cho ví dụ với route động.

<details>
<summary>👉 Xem câu trả lời</summary>

`generateStaticParams` (App Router, thay cho `getStaticPaths` của Pages Router) báo cho Next biết **những giá trị param nào cần render tĩnh tại thời điểm build** đối với một dynamic route như `app/blog/[slug]/page.tsx`.

```tsx
// app/blog/[slug]/page.tsx

export async function generateStaticParams() {
  const posts = await fetch('https://api/posts').then((r) => r.json());
  // Trả về mảng các param -> mỗi phần tử là một trang tĩnh
  return posts.map((post: { slug: string }) => ({ slug: post.slug }));
}

export default async function Page({ params }: { params: { slug: string } }) {
  const post = await fetch(`https://api/posts/${params.slug}`).then((r) => r.json());
  return <article>{post.title}</article>;
}
```

**Hành vi với param không nằm trong danh sách** được điều khiển bởi `dynamicParams` (mặc định `true`):

```tsx
export const dynamicParams = true;  // (mặc định) slug lạ -> render on-demand rồi cache (giống ISR)
// export const dynamicParams = false; // slug lạ -> trả về 404
```

Kết hợp với `export const revalidate = 3600;` bạn có pattern ISR cổ điển: build sẵn các trang phổ biến, các trang còn lại sinh khi có người truy cập lần đầu, và toàn bộ tự làm mới theo chu kỳ.

**Bẫy:** `generateStaticParams` chạy lúc build (và on-demand), nên nếu danh sách quá lớn (hàng trăm nghìn trang) sẽ làm build rất chậm — khi đó nên chỉ generate các trang quan trọng và để phần còn lại dùng `dynamicParams: true`.

</details>

---

### 7.7. Server Actions là gì? Cho ví dụ form submit và nêu các lưu ý bảo mật.

<details>
<summary>👉 Xem câu trả lời</summary>

**Server Actions** là các hàm async chạy **trên server** nhưng được gọi trực tiếp từ component (kể cả client) mà không cần tự viết API endpoint. Chúng được đánh dấu bằng chỉ thị `'use server'` và thường dùng cho **mutation** (tạo/sửa/xóa dữ liệu, submit form).

```tsx
// app/new-post/page.tsx — Server Component dùng action ngay trong form
import { redirect } from 'next/navigation';
import { revalidatePath } from 'next/cache';
import db from '@/lib/db';

async function createPost(formData: FormData) {
  'use server'; // đánh dấu đây là Server Action
  const title = formData.get('title') as string;
  if (!title?.trim()) throw new Error('Title required');

  await db.post.create({ data: { title } });
  revalidatePath('/posts'); // làm mới danh sách
  redirect('/posts');
}

export default function NewPost() {
  return (
    <form action={createPost}>
      <input name="title" />
      <button type="submit">Create</button>
    </form>
  );
}
```

Ở Client Component, ta thường kết hợp `useFormStatus` (trạng thái pending) hoặc hook `useActionState` (React 19) để xử lý loading và lỗi.

**Lưu ý bảo mật — cực kỳ quan trọng:**
- Server Action là một **public HTTP endpoint** ẩn. Bất kỳ ai cũng có thể gọi nó với input tùy ý → **luôn validate (vd Zod) và kiểm tra quyền (authz) bên trong action**, đừng tin frontend đã chặn.
- Đừng dựa vào việc "nút bị disable" hay "form bị ẩn" để bảo vệ; phải kiểm tra session/role ngay trong action.
- Tránh truyền dữ liệu nhạy cảm/closure không mong muốn vào action vì chúng được tuần tự hóa.

**Khi nào dùng:** mutation gắn liền với UI (form, like, toggle). **Khi nào dùng Route Handler thay thế:** cần một API công khai, có versioning, gọi từ client bên ngoài (mobile app, webhook), hoặc cần kiểm soát HTTP method/headers chi tiết.

</details>

---

### 7.8. Giải thích hệ thống file routing đặc biệt trong App Router: `layout`, `loading`, `error`, `template`.

<details>
<summary>👉 Xem câu trả lời</summary>

App Router dùng **các file quy ước (convention)** trong mỗi thư mục route để định nghĩa UI bao quanh `page.tsx`:

| File | Vai trò | Đặc điểm |
|---|---|---|
| `layout.tsx` | Khung dùng chung (header, sidebar) | **Giữ nguyên state** khi điều hướng giữa các trang con; không re-render |
| `template.tsx` | Giống layout nhưng **tạo instance mới mỗi lần điều hướng** | Reset state/effect mỗi route change (vd animation vào trang) |
| `loading.tsx` | UI fallback khi route đang tải | Tự bọc `page` trong `<Suspense>` |
| `error.tsx` | Error boundary cho route segment | **Phải là Client Component**; nhận `error` + `reset()` |
| `not-found.tsx` | UI khi gọi `notFound()` hoặc 404 | |
| `global-error.tsx` | Error boundary cấp root | Bắt cả lỗi trong root layout |

```tsx
// app/dashboard/layout.tsx — giữ sidebar khi chuyển trang con
export default function Layout({ children }: { children: React.ReactNode }) {
  return (
    <div className="flex">
      <Sidebar />
      <main>{children}</main>
    </div>
  );
}

// app/dashboard/loading.tsx — hiện ngay khi page đang fetch
export default function Loading() {
  return <Spinner />;
}

// app/dashboard/error.tsx — bắt lỗi runtime trong segment
'use client';
export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div>
      <p>Đã có lỗi: {error.message}</p>
      <button onClick={reset}>Thử lại</button>
    </div>
  );
}
```

**Phân biệt hay bị hỏi — `layout` vs `template`:** `layout` được dùng lại (mount một lần, giữ state/scroll), còn `template` remount mỗi lần điều hướng → dùng template khi bạn cần effect chạy lại hoặc animation enter mỗi khi vào trang.

**Lưu ý:** `error.tsx` không bắt được lỗi xảy ra trong chính `layout.tsx` cùng cấp (vì error boundary nằm bên trong layout đó) — để bắt lỗi layout root cần `global-error.tsx`.

</details>

---

### 7.9. Route groups, parallel routes và intercepting routes là gì? Mỗi loại giải quyết vấn đề gì?

<details>
<summary>👉 Xem câu trả lời</summary>

Đây là các pattern routing nâng cao của App Router, dùng cú pháp thư mục đặc biệt:

**1. Route Groups — `(folder)`**
Nhóm các route để tổ chức code hoặc dùng layout riêng **mà không thêm segment vào URL**.
```
app/
  (marketing)/         <- không xuất hiện trong URL
    layout.tsx         <- layout riêng cho marketing
    about/page.tsx     -> /about
  (shop)/
    layout.tsx         <- layout riêng cho shop
    cart/page.tsx      -> /cart
```

**2. Parallel Routes — `@slot`**
Render **nhiều page độc lập trong cùng một layout**, mỗi slot có loading/error riêng. Hữu ích cho dashboard nhiều khu vực.
```
app/
  layout.tsx           // nhận props { children, team, analytics }
  @team/page.tsx
  @analytics/page.tsx
  page.tsx
```
```tsx
export default function Layout({ children, team, analytics }) {
  return <>{children}{team}{analytics}</>; // hai slot render song song
}
```

**3. Intercepting Routes — `(.)`, `(..)`, `(...)`**
"Chặn" một route để hiển thị nó trong ngữ cảnh hiện tại (vd modal) khi điều hướng từ trong app, nhưng vẫn cho ra trang đầy đủ khi tải URL trực tiếp. Kinh điển là pattern **modal ảnh** (như Instagram, Dribbble).
```
app/
  feed/page.tsx
  photo/[id]/page.tsx          // URL trực tiếp -> trang full
  feed/@modal/(.)photo/[id]/page.tsx  // điều hướng từ feed -> mở modal
```

| Pattern | Vấn đề giải quyết | Cú pháp |
|---|---|---|
| Route group | Tách layout/tổ chức không đổi URL | `(name)` |
| Parallel route | Nhiều vùng nội dung song song trong 1 layout | `@slot` |
| Intercepting | Modal/preview giữ context + URL chia sẻ được | `(.)`, `(..)`, `(...)` |

Intercepting thường đi kèm parallel route (`@modal`) để khi đóng modal vẫn quay lại trang nền mượt mà.

</details>

---

### 7.10. Middleware trong Next.js chạy ở đâu và dùng để làm gì? Có giới hạn nào?

<details>
<summary>👉 Xem câu trả lời</summary>

`middleware.ts` (đặt ở gốc dự án) chạy **trước khi request tới được route**, trên **Edge Runtime**. Nó cho phép can thiệp request/response: rewrite, redirect, set header/cookie, kiểm tra auth, A/B testing, i18n routing.

```ts
// middleware.ts
import { NextResponse, type NextRequest } from 'next/server';

export function middleware(req: NextRequest) {
  const token = req.cookies.get('session')?.value;

  // Chặn route được bảo vệ nếu chưa đăng nhập
  if (!token && req.nextUrl.pathname.startsWith('/dashboard')) {
    const url = req.nextUrl.clone();
    url.pathname = '/login';
    return NextResponse.redirect(url);
  }
  return NextResponse.next();
}

// Chỉ chạy middleware cho các path khớp matcher (tối ưu hiệu năng)
export const config = {
  matcher: ['/dashboard/:path*', '/account/:path*'],
};
```

**Giới hạn quan trọng (hay bị hỏi):**
- Chạy trên **Edge Runtime**, không phải Node.js đầy đủ → **không** dùng được nhiều Node API (`fs`, các thư viện native, một số DB driver), không truy cập trực tiếp DB nặng.
- Phải nhẹ và nhanh vì chạy trên **mọi request khớp matcher** → không làm tác vụ tốn thời gian (gọi DB phức tạp, xử lý nặng).
- Không nên dùng middleware làm nơi authz duy nhất; với data-sensitive vẫn cần kiểm tra ở layer data/Server Action. (Trong các bản Next gần đây, Next còn khuyến nghị Data Access Layer cho authz thay vì chỉ dựa middleware.)
- Một dự án về cơ bản dùng **một** file middleware ở gốc (định tuyến bên trong bằng điều kiện/matcher).

</details>

---

### 7.11. `generateMetadata` hoạt động thế nào để làm SEO động? Khác `metadata` tĩnh ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

App Router quản lý thẻ `<head>` qua **Metadata API**, thay vì component `<Head>` của Pages Router. Có hai cách:

```tsx
// 1) Metadata tĩnh — biết trước lúc build, đặt ngay trong page/layout
export const metadata = {
  title: 'Trang chủ',
  description: 'Mô tả cố định',
};
```

```tsx
// 2) generateMetadata — metadata động phụ thuộc params/dữ liệu
import type { Metadata } from 'next';

export async function generateMetadata(
  { params }: { params: { slug: string } }
): Promise<Metadata> {
  const post = await fetch(`https://api/posts/${params.slug}`).then((r) => r.json());
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      images: [post.coverImage], // OG image cho social share
    },
    alternates: { canonical: `https://site.com/blog/${params.slug}` },
  };
}
```

**Điểm cốt lõi cho SEO:** vì metadata được sinh **trên server trước khi gửi HTML**, crawler/bot nhận được `<title>`, `<meta description>`, OG tags ngay trong HTML ban đầu — đây là lợi thế lớn so với SPA thuần CSR (nơi head ban đầu trống và bot có thể không thấy nội dung).

**Tối ưu thực tế:** Next **dedupe** các `fetch` giống nhau giữa `generateMetadata` và `page`, nên bạn có thể fetch cùng dữ liệu ở cả hai mà không gọi mạng hai lần (với fetch được cache). Ngoài ra App Router còn có `app/sitemap.ts`, `app/robots.ts`, và file `opengraph-image.tsx` để sinh OG image động.

**Bẫy:** `metadata` và `generateMetadata` chỉ hoạt động trong **Server Component**; bạn không thể export chúng từ file có `'use client'`.

</details>

---

### 7.12. `next/image` và `next/font` giúp tối ưu hiệu năng/Core Web Vitals như thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Đây là hai component/tiện ích built-in nhắm thẳng vào **Core Web Vitals** (LCP và CLS).

**`next/image`** tự động hóa các best practice về ảnh:
- **Resize & định dạng hiện đại** (WebP/AVIF) theo thiết bị; chỉ gửi đúng kích thước cần.
- **Lazy loading** mặc định (ảnh ngoài viewport chưa tải).
- **Chống CLS** bằng việc giữ chỗ theo `width`/`height` (hoặc `fill`) → layout không nhảy khi ảnh tải xong.
- `priority` cho ảnh LCP (vd hero) để preload, không lazy-load.
- `placeholder="blur"` để có hiệu ứng mờ trong khi tải.

```tsx
import Image from 'next/image';

<Image
  src="/hero.jpg"
  alt="Banner"
  width={1200}
  height={600}
  priority           // ảnh LCP: tải sớm, không lazy
  placeholder="blur"
/>
```

**`next/font`** tối ưu font để tránh layout shift và request chậm:
- **Self-host tự động** (kể cả Google Fonts) → không request sang domain ngoài, không lộ thông tin, loại bỏ vòng request thêm.
- **Không gây layout shift** nhờ tính toán fallback metrics (`size-adjust`), giảm CLS do FOUT/FOIT.
- Subset chỉ ký tự cần dùng.

```tsx
import { Inter } from 'next/font/google';
const inter = Inter({ subsets: ['latin'], display: 'swap' });

export default function RootLayout({ children }) {
  return <html className={inter.className}><body>{children}</body></html>;
}
```

**Bẫy thường gặp với `next/image`:** quên đặt `width/height` (hoặc `fill` + container có position) → vẫn bị CLS; dùng domain ngoài mà chưa cấu hình `images.remotePatterns` trong `next.config.js` → ảnh lỗi; lạm dụng `priority` cho mọi ảnh → mất tác dụng lazy-load.

</details>

---

### 7.13. Streaming SSR với Suspense hoạt động ra sao trong App Router? Nó cải thiện gì so với SSR truyền thống?

<details>
<summary>👉 Xem câu trả lời</summary>

SSR truyền thống là **all-or-nothing**: server phải fetch xong toàn bộ dữ liệu rồi mới gửi HTML → nếu một phần dữ liệu chậm, **cả trang** bị chặn (TTFB cao, người dùng nhìn màn hình trắng).

**Streaming SSR** chia trang thành nhiều mảnh và gửi HTML **dần dần** ngay khi từng phần sẵn sàng. Trong App Router, ranh giới streaming chính là `<Suspense>` (và `loading.tsx` thực chất là một Suspense bao quanh page).

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react';

export default function Page() {
  return (
    <section>
      <Header />                 {/* gửi ngay lập tức */}
      <Suspense fallback={<SkeletonStats />}>
        <SlowStats />            {/* fetch chậm -> stream sau, không chặn Header */}
      </Suspense>
      <Suspense fallback={<SkeletonFeed />}>
        <SlowFeed />
      </Suspense>
    </section>
  );
}

async function SlowStats() {
  const data = await fetch('https://api/stats').then((r) => r.json()); // chậm
  return <Stats data={data} />;
}
```

**Lợi ích:**
- **TTFB thấp hơn / FCP nhanh hơn:** phần shell (header, layout) hiển thị ngay với skeleton, không chờ phần chậm.
- **Tránh waterfall chặn UI:** mỗi Suspense boundary tải độc lập, song song.
- Cải thiện cảm nhận về tốc độ (perceived performance) đáng kể.

**Lưu ý:** để streaming có ích, nên đặt ranh giới Suspense **quanh phần chậm cụ thể**, không bọc cả trang (bọc cả trang thì lại quay về all-or-nothing). Khi stream, mã trạng thái HTTP đã gửi (200) nên việc xử lý lỗi chuyển sang error boundary của segment thay vì đổi status code.

</details>

---

### 7.14. Route Handlers (`route.ts`) là gì? Khi nào dùng thay cho Server Actions?

<details>
<summary>👉 Xem câu trả lời</summary>

**Route Handlers** là cách tạo **API endpoint** trong App Router, thay cho `pages/api/*`. Đặt file `route.ts` trong thư mục `app/`, export các hàm theo tên HTTP method (`GET`, `POST`, `PUT`, `DELETE`, `PATCH`...). Chúng dùng Web `Request`/`Response` chuẩn.

```ts
// app/api/posts/route.ts
import { NextResponse } from 'next/server';

export async function GET(req: Request) {
  const { searchParams } = new URL(req.url);
  const q = searchParams.get('q');
  const posts = await db.post.findMany({ where: { title: { contains: q ?? '' } } });
  return NextResponse.json(posts);
}

export async function POST(req: Request) {
  const body = await req.json();
  const created = await db.post.create({ data: body });
  return NextResponse.json(created, { status: 201 });
}
```

```ts
// app/api/posts/[id]/route.ts — route động
export async function DELETE(_req: Request, { params }: { params: { id: string } }) {
  await db.post.delete({ where: { id: params.id } });
  return new Response(null, { status: 204 });
}
```

**Route Handlers vs Server Actions:**

| | Route Handler | Server Action |
|---|---|---|
| Mục đích | API công khai (REST-like) | Mutation gắn với UI/form |
| Cách gọi | `fetch`/HTTP từ bất kỳ client nào | Gọi như function từ component |
| Kiểm soát HTTP | Đầy đủ (method, status, headers) | Trừu tượng hóa |
| Hợp với | Mobile app, webhook (Stripe...), third-party, cron | Form submit, like/toggle nội bộ |
| Caching | `GET` có thể opt-in cache | Không cache (luôn động) |

**Khi nào dùng Route Handler:** cần endpoint cho client bên ngoài (mobile/native), nhận **webhook** từ dịch vụ thứ ba, build REST/GraphQL public API, hoặc cần stream/trả file với kiểm soát header chi tiết. **Khi nào dùng Server Action:** mutation phát sinh từ chính UI Next của bạn (đơn giản, type-safe, ít boilerplate).

**Lưu ý phiên bản:** từ Next 15, `GET` Route Handler **không còn cache mặc định** — muốn cache phải khai báo (`export const dynamic = 'force-static'` hoặc tùy chọn revalidate).

</details>

---

### 7.15. Hãy mô tả các tầng caching của Next.js (App Router) và cách invalidate chúng.

<details>
<summary>👉 Xem câu trả lời</summary>

App Router có **nhiều tầng cache** chồng lên nhau — đây là chủ đề khó và hay gây nhầm. Có 4 tầng chính:

| Tầng cache | Nằm ở | Cache cái gì | Thời gian | Cách invalidate |
|---|---|---|---|---|
| **Request Memoization** | Server, trong 1 request | Kết quả `fetch` trùng nhau (cùng URL/options) | Trong vòng đời 1 request render | Tự hết sau request |
| **Data Cache** | Server (persistent, qua nhiều request/deploy) | Kết quả `fetch` / dữ liệu | Đến khi revalidate | `revalidateTag`, `revalidatePath`, `revalidate` time |
| **Full Route Cache** | Server (build-time) | HTML + RSC payload của route tĩnh | Đến khi revalidate/redeploy | `revalidatePath`, redeploy, dữ liệu động |
| **Router Cache** | **Client** (bộ nhớ trình duyệt) | RSC payload các route đã ghé, prefetch | Phiên/khoảng ngắn | `router.refresh()`, `revalidatePath` (qua action), điều hướng |

```tsx
// Sau mutation, invalidate đúng tầng:
import { revalidateTag, revalidatePath } from 'next/cache';

async function updateProduct(id: string) {
  'use server';
  await db.product.update(/* ... */);
  revalidateTag('products');     // -> xóa Data Cache theo tag
  revalidatePath('/products');   // -> xóa Full Route + đẩy refresh Router Cache
}
```

```tsx
// Phía client, làm mới Router Cache thủ công
'use client';
import { useRouter } from 'next/navigation';
const router = useRouter();
router.refresh(); // lấy lại RSC payload mới từ server, giữ nguyên client state
```

**Sai lầm kinh điển:** dữ liệu đã đổi trong DB nhưng UI vẫn cũ → thường do **Data Cache** hoặc **Router Cache** chưa được invalidate sau mutation. Quy tắc: sau khi ghi dữ liệu, luôn `revalidateTag`/`revalidatePath` cho phần liên quan; nếu là tương tác client thuần, dùng `router.refresh()`.

**Lưu ý phiên bản:** Next 15 thay đổi mặc định theo hướng "ít cache hơn" (fetch và GET handler không cache mặc định, Router Cache với `staleTimes` ngắn hơn cho dynamic page), nhằm giảm các bug "dữ liệu cũ" mà Next 13/14 hay gặp. Khi trả lời nên nêu rõ bạn ý thức được khác biệt này giữa các phiên bản.

</details>
## 8. HTML CSS và Trình duyệt

### 8.1. Hãy giải thích Box Model trong CSS. Sự khác nhau giữa `content-box` và `border-box` là gì, và vì sao trong dự án React/Next.js người ta hay đặt `box-sizing: border-box` cho toàn bộ?

<details>
<summary>👉 Xem câu trả lời</summary>

Box Model mô tả mỗi phần tử như một hộp gồm 4 lớp từ trong ra ngoài: `content` (nội dung) → `padding` (đệm) → `border` (viền) → `margin` (lề). Thuộc tính `box-sizing` quyết định `width`/`height` bạn khai báo áp dụng cho phần nào:

- `content-box` (mặc định): `width` chỉ tính cho phần content. Chiều rộng thực tế phần tử chiếm = `width + padding + border`. Rất khó tính toán layout.
- `border-box`: `width` đã bao gồm cả `padding` và `border`. Đặt `width: 200px` thì phần tử luôn rộng đúng 200px dù bạn thêm padding hay border bao nhiêu.

Trong dự án React/Next.js, ta thường reset toàn cục để layout dễ đoán, đặc biệt khi ghép nhiều component có padding/border khác nhau:

```css
*,
*::before,
*::after {
  box-sizing: border-box;
}
```

Bẫy thường gặp: với `content-box`, một `<input width: 100%>` có thêm `padding` sẽ tràn ra ngoài container cha và gây cuộn ngang. Lưu ý `margin` không bao giờ nằm trong `box-sizing` — nó luôn cộng thêm ở ngoài.

</details>

---

### 8.2. Khi nào bạn chọn Flexbox và khi nào chọn CSS Grid? Cho ví dụ cụ thể trong một component React.

<details>
<summary>👉 Xem câu trả lời</summary>

Nguyên tắc gọn: **Flexbox cho bố cục một chiều** (một hàng HOẶC một cột), **Grid cho bố cục hai chiều** (cả hàng và cột cùng lúc). Flexbox thiên về phân phối không gian theo nội dung; Grid thiên về định nghĩa cấu trúc trước rồi đặt nội dung vào.

| Tiêu chí | Flexbox | Grid |
|---|---|---|
| Số chiều | 1 chiều (row/column) | 2 chiều (rows + columns) |
| Tư duy | Content-first (theo nội dung) | Layout-first (theo khung) |
| Hợp với | Navbar, toolbar, danh sách nút, căn giữa | Dashboard, card gallery, layout trang |
| Khoảng cách | `gap` (hỗ trợ tốt) | `gap` (sinh ra cho Grid) |

Ví dụ — một thanh `Toolbar` (Flexbox, một chiều):

```tsx
function Toolbar() {
  return (
    <div style={{ display: 'flex', alignItems: 'center', gap: 8 }}>
      <Logo />
      <SearchBox style={{ flex: 1 }} /> {/* chiếm hết khoảng trống còn lại */}
      <UserMenu />
    </div>
  );
}
```

Ví dụ — một lưới card responsive (Grid, hai chiều, tự xuống dòng):

```tsx
function CardGrid({ items }: { items: Item[] }) {
  return (
    <div
      style={{
        display: 'grid',
        gap: 16,
        gridTemplateColumns: 'repeat(auto-fill, minmax(240px, 1fr))',
      }}
    >
      {items.map((it) => (
        <Card key={it.id} {...it} />
      ))}
    </div>
  );
}
```

Thực tế hai cái thường kết hợp: Grid dựng khung tổng trang, Flexbox sắp xếp nội dung bên trong từng ô.

</details>

---

### 8.3. Phân biệt các giá trị của `position`: `static`, `relative`, `absolute`, `fixed`, `sticky`. `position: sticky` hoạt động dựa trên điều kiện nào?

<details>
<summary>👉 Xem câu trả lời</summary>

| Giá trị | Hành vi | Tham chiếu định vị |
|---|---|---|
| `static` | Mặc định, theo luồng tài liệu, bỏ qua `top/left/...` | — |
| `relative` | Vẫn giữ chỗ trong luồng, dịch chuyển so với vị trí gốc; tạo "mốc" cho con `absolute` | Chính nó |
| `absolute` | Tách khỏi luồng, không chiếm chỗ | Tổ tiên gần nhất có `position` khác `static` (hoặc viewport) |
| `fixed` | Tách khỏi luồng, dính theo viewport khi cuộn | Viewport |
| `sticky` | Lai giữa `relative` và `fixed`: bình thường như `relative`, dính lại khi cuộn chạm ngưỡng | Container cuộn cha gần nhất |

`position: sticky` cần đủ điều kiện để hoạt động:
1. Phải có ít nhất một trong `top/bottom/left/right` để xác định ngưỡng dính.
2. Phần tử chỉ dính trong phạm vi container cha của nó — khi cuộn quá khỏi cha, nó trượt đi theo.
3. Cha (hoặc tổ tiên) **không được** đặt `overflow: hidden/auto/scroll` — đây là bẫy phổ biến nhất khiến sticky "không chạy".

Ví dụ header dính trong React:

```tsx
function StickyHeader() {
  return (
    <header
      style={{ position: 'sticky', top: 0, zIndex: 10, background: '#fff' }}
    >
      Tiêu đề luôn ở trên cùng khi cuộn
    </header>
  );
}
```

Mẹo Next.js: nếu bạn bọc nội dung trong một wrapper có `overflow-x-hidden` (hay dùng để chặn cuộn ngang trên mobile), header `sticky` bên trong sẽ không dính — cần đặt `overflow` đúng cấp.

</details>

---

### 8.4. CSS Specificity được tính như thế nào? Vì sao nên tránh `!important`, và trong dự án dùng Tailwind/styled-components thì specificity ảnh hưởng ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

Specificity quyết định selector nào thắng khi nhiều rule cùng nhắm vào một phần tử. Tính theo bộ ba `(a, b, c)`, so sánh từ trái sang:
- `a`: số lượng ID selector (`#id`)
- `b`: số lượng class, attribute, pseudo-class (`.class`, `[type]`, `:hover`)
- `c`: số lượng element và pseudo-element (`div`, `::before`)

Inline style (`style="..."`) cao hơn mọi selector. `!important` ghi đè lên cả inline. Khi specificity bằng nhau, rule khai báo **sau** trong nguồn sẽ thắng.

```css
/* (0,0,1) */   p            { color: black; }
/* (0,1,1) */   p.intro      { color: blue; }   /* thắng */
/* (1,0,0) */   #main        { color: green; }  /* thắng cả hai trên */
```

Vì sao tránh `!important`: nó phá vỡ thứ tự tự nhiên, khiến khó ghi đè về sau (chỉ một `!important` khác mới thắng được), tạo "cuộc chạy đua important" và làm CSS khó bảo trì. Cách lành mạnh hơn: tăng specificity hợp lý hoặc sắp xếp lại cấu trúc.

Trong hệ React:
- **Tailwind**: các class utility có specificity rất thấp (đều là single class `(0,1,0)`), nên thứ tự trong file CSS đã build quyết định — đôi khi `flex` và `block` "đá" nhau; dùng `tailwind-merge` (qua `clsx`/`cn`) để class sau ghi đè đúng class trước thay vì dựa vào `!important`.
- **styled-components**: tự sinh class hash, mỗi component một class nên ít xung đột; khi cần ghi đè component con, styled-components dùng kỹ thuật nhân đôi class `&&` để tăng specificity một cách có kiểm soát thay vì `!important`.

</details>

---

### 8.5. Semantic HTML là gì và vì sao nó quan trọng cho accessibility? Cho ví dụ tái cấu trúc một component React từ "div soup" sang semantic.

<details>
<summary>👉 Xem câu trả lời</summary>

Semantic HTML là việc dùng thẻ mang đúng ý nghĩa nội dung (`<header>`, `<nav>`, `<main>`, `<article>`, `<button>`, `<label>`...) thay vì dùng `<div>`/`<span>` cho mọi thứ. Lợi ích:
- **Accessibility**: trình đọc màn hình (screen reader) dựa vào vai trò ngầm (implicit ARIA role) để điều hướng theo landmark, đọc đúng cấu trúc; người dùng dùng bàn phím tab qua được `<button>`/`<a>` mà không cần code thêm.
- **SEO** và khả năng đọc của code.
- Hành vi miễn phí: `<button>` có sẵn focus, Enter/Space, `<a href>` điều hướng được.

```tsx
// ❌ Div soup: mất ngữ nghĩa, không truy cập được bằng bàn phím
function BadCard() {
  return (
    <div className="card">
      <div className="title">Bài viết</div>
      <div className="link" onClick={open}>Đọc thêm</div>
    </div>
  );
}

// ✅ Semantic: đúng vai trò, accessible mặc định
function GoodCard() {
  return (
    <article className="card">
      <h2 className="title">Bài viết</h2>
      <a href="/post/1" className="link">Đọc thêm</a>
    </article>
  );
}
```

Bẫy điển hình trong React: dùng `<div onClick>` làm nút bấm. Nó không focus được bằng Tab, không phản hồi phím Enter/Space, không được screen reader thông báo là nút. Nếu bắt buộc dùng phần tử không phải button, phải thêm `role="button"`, `tabIndex={0}` và xử lý `onKeyDown` — nhiều việc hơn là chỉ dùng `<button>`. Luôn gắn `<label htmlFor>` với input, dùng `alt` cho ảnh có nghĩa và `alt=""` cho ảnh trang trí.

</details>

---

### 8.6. Bạn xây dựng responsive như thế nào? So sánh `media query` với `container query`, và giải thích vì sao nên ưu tiên `rem`/`em` cùng tư duy mobile-first.

<details>
<summary>👉 Xem câu trả lời</summary>

**Mobile-first**: viết style nền cho màn hình nhỏ trước, rồi dùng `min-width` để bổ sung cho màn lớn. Lý do: style cơ sở gọn hơn, tránh ghi đè ngược, và phù hợp việc đa số người dùng vào từ mobile.

```css
/* base: mobile */
.grid { display: grid; grid-template-columns: 1fr; }
/* tablet trở lên */
@media (min-width: 768px) {
  .grid { grid-template-columns: repeat(2, 1fr); }
}
```

**Đơn vị**: `rem` tính theo `font-size` của `<html>` (gốc), nên nhất quán toàn trang và tôn trọng cài đặt cỡ chữ của người dùng (quan trọng cho accessibility). `em` tính theo `font-size` của chính phần tử/cha, dễ "dồn" (compounding) khi lồng nhau. Dùng `px` cho viền mảnh, `rem` cho khoảng cách/typography, `em` khi muốn co giãn theo cỡ chữ cục bộ (ví dụ padding của nút theo cỡ chữ nút).

**Media query vs Container query**:

| | Media query | Container query |
|---|---|---|
| Đo theo | Kích thước viewport | Kích thước container cha |
| Vấn đề giải quyết | Layout toàn trang | Component tái dùng ở nhiều ngữ cảnh |
| Cú pháp | `@media (min-width: ...)` | `container-type` + `@container (...)` |

Container query cực hợp với component-based UI như React: cùng một `<Card>` đặt trong sidebar hẹp hay vùng content rộng sẽ tự đổi layout dựa trên không gian thực nó có, thay vì đoán theo viewport.

```css
.card-wrapper { container-type: inline-size; }
@container (min-width: 400px) {
  .card { display: grid; grid-template-columns: 120px 1fr; }
}
```

Lưu ý: Tailwind hỗ trợ container query qua plugin/biến thể `@container`; còn breakpoint `sm/md/lg` mặc định của Tailwind dựa trên `min-width` viewport (mobile-first).

</details>

---

### 8.7. Critical Rendering Path là gì? Trình duyệt đi qua những bước nào từ HTML/CSS tới khi hiển thị pixel, và tại sao CSS được coi là "render-blocking"?

<details>
<summary>👉 Xem câu trả lời</summary>

Critical Rendering Path (CRP) là chuỗi bước trình duyệt thực hiện để biến HTML, CSS, JS thành pixel trên màn hình. Các bước chính:

1. **Parse HTML → DOM**: dựng cây DOM.
2. **Parse CSS → CSSOM**: dựng cây CSSOM. CSS là render-blocking vì trình duyệt không muốn vẽ nội dung rồi lại nhảy style (FOUC) — nó chờ CSSOM hoàn tất trước khi render.
3. **Render Tree**: ghép DOM + CSSOM, bỏ các node `display: none`.
4. **Layout (Reflow)**: tính vị trí và kích thước hình học của từng node.
5. **Paint**: tô màu, text, ảnh, viền thành các layer.
6. **Composite**: ghép các layer lại để ra khung hình cuối.

Vì sao tối ưu CRP quan trọng: nó ảnh hưởng trực tiếp tới First Contentful Paint và Largest Contentful Paint. JS đồng bộ (`<script>` không có `defer/async`) là parser-blocking — chặn cả việc dựng DOM.

Trong Next.js, các kỹ thuật giảm CRP gồm: SSR/SSG để gửi HTML có sẵn nội dung; tự động code-splitting; `next/font` để tránh chặn render bởi font và giảm layout shift; `next/script` với chiến lược `afterInteractive`/`lazyOnload`; inline critical CSS. Trên Pages Router, JS dùng `defer`; CSS quan trọng được tối ưu sẵn.

```tsx
// next/script: tải script không chặn render ban đầu
import Script from 'next/script';

<Script src="https://example.com/analytics.js" strategy="lazyOnload" />;
```

</details>

---

### 8.8. Phân biệt Reflow (Layout) và Repaint. Loại nào tốn kém hơn, và bạn làm gì trong React để giảm thiểu reflow khi animation?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Reflow (Layout)**: trình duyệt tính lại hình học — vị trí, kích thước của phần tử (và thường kéo theo các phần tử khác). Kích hoạt bởi đổi `width`, `height`, `top`, `margin`, thêm/xóa DOM, đổi font-size, đọc các thuộc tính như `offsetWidth`...
- **Repaint**: vẽ lại pixel mà không đổi layout — đổi `color`, `background`, `visibility`, `box-shadow`.

**Reflow tốn kém hơn** vì nó luôn kéo theo repaint, và có thể lan ra cả cây. Repaint thì không nhất thiết gây reflow.

Tối ưu khi animation: chỉ animate **`transform` và `opacity`** vì chúng được xử lý ở giai đoạn composite trên GPU, **không gây reflow hay repaint**. Tránh animate `left/top/width/height`.

```tsx
// ❌ animate `left` → reflow mỗi frame, giật
.box { transition: left 0.3s; }

// ✅ animate `transform` → chạy trên compositor, mượt
.box { transition: transform 0.3s; }
```

Một bẫy nữa là **layout thrashing**: đọc một thuộc tính layout (vd `el.offsetHeight`) ngay sau khi ghi style trong vòng lặp khiến trình duyệt phải reflow đồng bộ liên tục. Trong React nên gom thay đổi DOM, đọc đo đạc trong `useLayoutEffect` một lần, hoặc dùng thư viện như Framer Motion — nó animate qua `transform`/`opacity` và dùng `requestAnimationFrame` để tối ưu sẵn:

```tsx
import { motion } from 'framer-motion';

<motion.div animate={{ x: 100, opacity: 1 }} />; // x = translateX (transform)
```

</details>

---

### 8.9. Stacking context là gì? Vì sao đôi khi `z-index: 9999` vẫn không đưa được phần tử lên trên, và điều này liên quan gì tới modal/dropdown trong React?

<details>
<summary>👉 Xem câu trả lời</summary>

Stacking context là một "ngữ cảnh xếp chồng" độc lập trên trục z. `z-index` chỉ so sánh **giữa các phần tử trong cùng một stacking context**. Một phần tử dù có `z-index: 9999` nhưng nằm trong một stacking context cha có thứ hạng thấp thì vẫn bị che bởi context cha khác cao hơn.

Một số thứ tạo stacking context mới:
- `position` khác `static` **kèm** `z-index` là số (không phải `auto`).
- `opacity` < 1.
- `transform`, `filter`, `perspective`, `will-change`, `backdrop-filter` khác giá trị mặc định.
- `position: fixed`/`sticky`.
- Phần tử con của Flex/Grid có `z-index` khác `auto`.

```tsx
// Bẫy: cha A có opacity tạo stacking context riêng
// .modal z-index 9999 bị "nhốt" bên trong A, không vượt được .overlay
.parentA { opacity: 0.99; }       /* tạo stacking context */
.parentA .modal { z-index: 9999; }
.overlay { position: fixed; z-index: 10; } /* vẫn che được .modal */
```

Đây chính là lý do dropdown/tooltip/modal trong React hay bị một phần tử cha (có `transform`, `overflow`, hoặc `opacity`) "cắt" hoặc che mất. Giải pháp chuẩn là **Portal**: render phần tử lên thẳng `document.body`, thoát khỏi stacking context và `overflow` của cha:

```tsx
import { createPortal } from 'react-dom';

function Modal({ children }: { children: React.ReactNode }) {
  return createPortal(
    <div className="overlay">{children}</div>,
    document.body,
  );
}
```

Lưu ý Next.js (App Router với RSC): Portal là API client-side nên component dùng `createPortal` phải là Client Component (`'use client'`) và thường cần kiểm tra đã mount để tránh lệch hydration. Các thư viện như Radix UI đã xử lý sẵn portal cho dropdown/dialog.

</details>

---

### 8.10. Bạn triển khai theming và dark mode bằng CSS Variables như thế nào? Vì sao biến CSS hợp với React/Next.js hơn là dùng nhiều class điều kiện, và làm sao tránh nhấp nháy (FOUC) khi tải trang?

<details>
<summary>👉 Xem câu trả lời</summary>

CSS Custom Properties (biến CSS) khai báo bằng `--ten` và đọc bằng `var(--ten)`. Chúng **kế thừa** và có thể ghi đè theo scope (vd `:root` vs `[data-theme="dark"]`), nên rất hợp cho theming: đổi giá trị biến ở một nơi là toàn bộ UI đổi theo mà không phải render lại component.

```css
:root {
  --bg: #ffffff;
  --text: #111111;
}
[data-theme='dark'] {
  --bg: #0a0a0a;
  --text: #f5f5f5;
}
body {
  background: var(--bg);
  color: var(--text);
}
```

```tsx
// Đổi theme = đổi 1 attribute, không cần truyền prop xuống mọi component
function ThemeToggle() {
  const toggle = () => {
    const next =
      document.documentElement.dataset.theme === 'dark' ? 'light' : 'dark';
    document.documentElement.dataset.theme = next;
    localStorage.setItem('theme', next);
  };
  return <button onClick={toggle}>Đổi giao diện</button>;
}
```

Vì sao hơn class điều kiện: tránh việc rải `className={isDark ? 'a' : 'b'}` khắp nơi, biến CSS chỉ cần một điểm điều khiển (attribute trên `<html>`), runtime-themeable (đổi mượt qua transition), và Tailwind cũng dùng cách này qua `darkMode: 'class'` ánh xạ token sang biến CSS.

**Tránh FOUC/nhấp nháy** (vấn đề lớn với SSR như Next.js): vì HTML server render trước khi JS chạy, nếu set theme trong `useEffect` thì khung hình đầu sẽ là light rồi nháy sang dark. Giải pháp là chạy một script đồng bộ trong `<head>` đọc `localStorage` và đặt attribute **trước khi** trang paint:

```tsx
// Trong app: chèn script blocking nhỏ ở head (hoặc dùng thư viện next-themes)
<script
  dangerouslySetInnerHTML={{
    __html: `(function(){var t=localStorage.getItem('theme')||'light';document.documentElement.dataset.theme=t;})();`,
  }}
/>
```

Thực tế trong Next.js người ta thường dùng `next-themes`: nó tự chèn script chống nháy, hỗ trợ `system` theme và đặt `suppressHydrationWarning` để tránh cảnh báo lệch hydration trên thẻ `<html>`.

</details>

---

### 8.11. (Tình huống) Một đồng nghiệp báo rằng trang bị "cuộn ngang lạ" trên mobile và một tooltip bị che mất. Hãy kể về một lần bạn debug vấn đề CSS/layout khó và cách bạn tiếp cận.

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

**Situation (Bối cảnh)**: Trong một dự án Next.js, sau khi release một trang landing, QA báo trên mobile xuất hiện cuộn ngang không mong muốn và một tooltip trong bảng dữ liệu bị cắt mất nửa.

**Task (Nhiệm vụ)**: Tôi cần tìm nguyên nhân gốc của cả hai và sửa mà không phá layout các trang khác.

**Action (Hành động)**:
- Với cuộn ngang: tôi mở DevTools, dùng thủ thuật thêm `* { outline: 1px solid red; }` để khoanh vùng, rồi soi phần tử tràn. Hóa ra một phần tử dùng `width: 100vw` bên trong container có padding — `100vw` không trừ thanh cuộn và padding cha. Tôi đổi sang `width: 100%` và rà soát các chỗ dùng đơn vị viewport. Tôi cũng kiểm tra `box-sizing` đã là `border-box` toàn cục để padding không làm tràn.
- Với tooltip bị che: tôi nhận ra `z-index` cao vẫn vô dụng. Kiểm tra cây cha thì thấy một wrapper có `transform` (do animation) tạo stacking context, "nhốt" tooltip. Tôi chuyển tooltip sang render bằng `createPortal` lên `document.body` để thoát khỏi stacking context và `overflow: hidden` của bảng.

**Result (Kết quả)**: Cuộn ngang biến mất trên mọi thiết bị, tooltip hiển thị đầy đủ. Tôi rút ra checklist nội bộ: cẩn trọng với `vw`, kiểm tra stacking context khi `z-index` "không nghe lời", và ưu tiên Portal cho mọi overlay. Tôi chia sẻ lại trong buổi review để cả nhóm tránh lặp lại.

</details>

---

### 8.12. `gap`, `margin` collapsing và căn giữa: hãy giải thích margin collapsing là gì và nêu các cách căn giữa một phần tử cả chiều ngang lẫn dọc trong CSS hiện đại.

<details>
<summary>👉 Xem câu trả lời</summary>

**Margin collapsing**: các margin theo chiều dọc (top/bottom) của các phần tử block kề nhau hoặc lồng nhau có thể "gộp" lại thành một margin lớn nhất thay vì cộng dồn. Ví dụ một `<p>` margin-bottom 20px nằm trên `<p>` margin-top 16px → khoảng cách thực là 20px (max), không phải 36px.

Đặc điểm cần nhớ:
- Chỉ xảy ra với **margin dọc**, không xảy ra với margin ngang.
- **Không** xảy ra trong Flex/Grid container — đây là lý do nữa nên dùng `gap` của Flex/Grid để kiểm soát khoảng cách thay vì margin.
- Có thể chặn bằng `padding`, `border`, hoặc tạo block formatting context (vd `overflow: hidden`, `display: flow-root`).

**Các cách căn giữa cả hai chiều**:

```css
/* 1. Flexbox - phổ biến nhất */
.parent { display: flex; align-items: center; justify-content: center; }

/* 2. Grid - ngắn gọn nhất */
.parent { display: grid; place-items: center; }

/* 3. Absolute + transform - khi cần thoát luồng */
.child {
  position: absolute;
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
}

/* 4. margin auto - chỉ căn ngang cho block có width cố định */
.child { width: 200px; margin-inline: auto; }
```

Trong React, cách 1 và 2 được dùng nhiều nhất vì rõ ràng và không cần biết kích thước con. Tailwind ánh xạ thẳng: `flex items-center justify-center` hoặc `grid place-items-center`. Nên ưu tiên `gap` thay cho margin giữa các item con để tránh margin collapsing khó đoán.

</details>
## 9. Web Networking và Bảo mật Frontend

### 9.1. Khi bạn gõ một URL vào trình duyệt và nhấn Enter, điều gì xảy ra? Là frontend developer, bạn quan tâm tới những bước nào nhất?

<details>
<summary>👉 Xem câu trả lời</summary>

Tóm tắt luồng từ lúc nhấn Enter tới khi thấy giao diện:

1. **Phân giải DNS**: trình duyệt tra cứu IP của domain (kiểm tra cache trình duyệt → OS → router → DNS resolver).
2. **Thiết lập kết nối**: TCP handshake (3 bước), rồi TLS handshake nếu là HTTPS (trao đổi chứng chỉ, thỏa thuận khóa).
3. **Gửi HTTP request**: trình duyệt gửi request (kèm cookie cho domain đó, header `Accept`, v.v.).
4. **Server xử lý và trả response**: trả về HTML (với Next.js có thể là HTML đã render sẵn từ SSR/SSG).
5. **Trình duyệt parse HTML**: dựng DOM tree, gặp CSS thì dựng CSSOM, gặp `<script>` thì tải/chạy JS (có thể chặn parse nếu không có `defer`/`async`).
6. **Render**: kết hợp DOM + CSSOM thành render tree → layout (reflow) → paint → composite.
7. **Với SPA/Next.js**: sau khi HTML hiển thị, JS bundle tải về và **hydration** gắn event listener vào DOM tĩnh để trang trở nên tương tác.

Là FE developer, các bước đáng quan tâm nhất:

| Bước | Tại sao FE quan tâm |
|------|---------------------|
| Tải HTML/JS/CSS | Bundle size, code-splitting, ảnh hưởng tới TTFB/FCP/LCP |
| Render & Layout | Tránh layout thrashing, reflow tốn kém |
| Hydration (Next.js) | Tốn thời gian, gây "hydration mismatch" nếu HTML server khác client |
| DNS/TLS | `<link rel="preconnect">`, `dns-prefetch` để tối ưu kết nối tới API/CDN |

```html
<!-- Tối ưu kết nối sớm tới domain API/CDN -->
<link rel="preconnect" href="https://api.example.com" />
<link rel="dns-prefetch" href="https://cdn.example.com" />
```

**Bẫy thường gặp**: nói rằng JS luôn chặn render. Thực tế `<script defer>` và ES module (`type="module"`) tải song song và chạy sau khi parse xong; chỉ `<script>` đồng bộ không có thuộc tính mới chặn parser.

</details>

---

### 9.2. Phân biệt các HTTP method phổ biến và ý nghĩa của tính idempotent. Hãy giải thích vài status code mà frontend cần xử lý khác nhau.

<details>
<summary>👉 Xem câu trả lời</summary>

**HTTP method**:

| Method | Mục đích | Idempotent? | An toàn (safe)? |
|--------|----------|-------------|-----------------|
| GET | Đọc dữ liệu | Có | Có (không đổi state) |
| POST | Tạo mới / hành động | Không | Không |
| PUT | Thay thế toàn bộ resource | Có | Không |
| PATCH | Cập nhật một phần | Thường không | Không |
| DELETE | Xóa | Có | Không |

**Idempotent** = gọi nhiều lần cho cùng kết quả phía server như gọi một lần. Quan trọng với FE khi làm **retry**: được phép retry GET/PUT/DELETE, nhưng retry POST có thể tạo trùng dữ liệu (ví dụ tạo 2 đơn hàng) — cần `idempotency-key` nếu muốn retry an toàn.

**Status code FE hay xử lý**:

```ts
async function handleResponse(res: Response) {
  if (res.ok) return res.json();            // 2xx
  switch (res.status) {
    case 401: redirectToLogin(); break;     // Chưa/đã hết hạn xác thực
    case 403: showForbidden(); break;       // Đã đăng nhập nhưng không đủ quyền
    case 404: showNotFound(); break;
    case 422: return showValidationErrors(await res.json()); // Lỗi validate
    case 429: return retryWithBackoff();     // Bị rate limit
    default:
      if (res.status >= 500) showServerError(); // Lỗi server -> có thể retry
  }
}
```

Các nhóm cần nhớ: `2xx` thành công, `3xx` redirect, `4xx` lỗi do client (đừng retry vô ích trừ 408/429), `5xx` lỗi server (thường nên retry với backoff).

**Bẫy**: nhầm `401 Unauthorized` (chưa xác thực) với `403 Forbidden` (đã xác thực nhưng không có quyền). Và phân biệt `422` (dữ liệu hợp lệ về cú pháp nhưng sai nghiệp vụ) với `400` (request sai định dạng).

</details>

---

### 9.3. CORS là gì? Vì sao bạn lại gặp lỗi CORS, và frontend có thể "tự sửa" nó không?

<details>
<summary>👉 Xem câu trả lời</summary>

**CORS (Cross-Origin Resource Sharing)** là cơ chế dùng HTTP header để **server** cho phép trình duyệt đọc response từ một origin khác. Nó được trình duyệt thực thi vì chính sách **Same-Origin Policy** (origin = scheme + host + port).

Bạn gặp lỗi khi frontend ở `http://localhost:3000` gọi API ở `https://api.example.com` mà server **không trả** header `Access-Control-Allow-Origin` phù hợp. Trình duyệt vẫn gửi request và server vẫn xử lý, nhưng trình duyệt **chặn không cho JS đọc response**.

Điểm mấu chốt cần nói trong phỏng vấn: **CORS là vấn đề cấu hình ở server, frontend không thể tự sửa bằng code JS**. Đổi header request từ FE không giải quyết được.

```http
# Server phải trả về:
Access-Control-Allow-Origin: http://localhost:3000
Access-Control-Allow-Credentials: true   # nếu gửi cookie
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
```

**Cách xử lý phía FE (workaround hợp lệ)**:

```ts
// next.config.js — dùng rewrites/proxy để request trở thành same-origin
module.exports = {
  async rewrites() {
    return [{ source: '/api/:path*', destination: 'https://api.example.com/:path*' }];
  },
};
```

Vì request đi qua server Next.js (same-origin với trình duyệt), không còn bị CORS chặn. Khi dev cũng có thể dùng `proxy` của dev server.

**Bẫy**: `*` cho `Access-Control-Allow-Origin` **không hoạt động** khi gửi credentials (cookie). Phải trả về origin cụ thể.

</details>

---

### 9.4. Preflight request là gì? Khi nào trình duyệt gửi nó, và làm sao để tránh preflight không cần thiết?

<details>
<summary>👉 Xem câu trả lời</summary>

**Preflight** là một request `OPTIONS` mà trình duyệt **tự động gửi trước** request thật, để hỏi server: "Tôi sắp gửi method/header này, anh có cho phép không?". Server phản hồi bằng các header `Access-Control-Allow-*`.

Trình duyệt gửi preflight khi request **không** thuộc loại "simple request". Một request là simple khi:
- Method là `GET`, `HEAD`, hoặc `POST`, **và**
- Chỉ dùng các header được phép sẵn, **và**
- `Content-Type` là `application/x-www-form-urlencoded`, `multipart/form-data`, hoặc `text/plain`.

Vì vậy, request rất phổ biến sau đây **sẽ kích hoạt preflight**:

```ts
fetch('https://api.example.com/users', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',   // -> không phải simple -> preflight
    Authorization: 'Bearer ...',          // header tùy chỉnh -> preflight
  },
  body: JSON.stringify(data),
});
```

Luồng: `OPTIONS /users` (preflight) → server `204` + cho phép → trình duyệt mới gửi `POST /users` thật.

**Tối ưu**: server đặt `Access-Control-Max-Age` để cache kết quả preflight, tránh gửi `OPTIONS` lặp lại:

```http
Access-Control-Max-Age: 86400
```

**Bẫy**: Khi debug Network tab, nhiều người hoảng vì thấy request `OPTIONS` "lỗi" hoặc nghĩ API gọi 2 lần. Đó là hành vi bình thường của preflight. Nếu preflight thất bại (server không trả allow header đúng), request thật sẽ không bao giờ được gửi.

</details>

---

### 9.5. Hãy giải thích cơ chế caching HTTP qua `Cache-Control`, `ETag` và `stale-while-revalidate`. Chúng liên hệ thế nào với TanStack Query?

<details>
<summary>👉 Xem câu trả lời</summary>

**`Cache-Control`** điều khiển ai được cache và bao lâu:

```http
Cache-Control: max-age=3600          # cache 1 giờ, dùng lại không hỏi server
Cache-Control: no-cache              # phải revalidate với server trước khi dùng
Cache-Control: no-store              # không bao giờ cache (dữ liệu nhạy cảm)
Cache-Control: private, max-age=600  # chỉ trình duyệt cache, không CDN
```

**`ETag`** là "vân tay" của nội dung. Lần sau trình duyệt gửi kèm `If-None-Match`; nếu chưa đổi, server trả `304 Not Modified` (không gửi lại body → tiết kiệm băng thông):

```http
# Response lần 1
ETag: "abc123"
# Request lần 2
If-None-Match: "abc123"
# Response: 304 Not Modified (body rỗng)
```

**`stale-while-revalidate`**: cho phép dùng ngay dữ liệu cũ (stale) để hiển thị tức thì, đồng thời ngầm fetch bản mới ở background:

```http
Cache-Control: max-age=60, stale-while-revalidate=300
# 0-60s: fresh; 60-360s: trả bản cũ + fetch mới ngầm; >360s: bắt buộc fetch mới
```

**Liên hệ TanStack Query**: triết lý y hệt `stale-while-revalidate` nhưng ở tầng JS/in-memory:

```tsx
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  staleTime: 60_000,   // 60s coi là fresh, không refetch
  gcTime: 300_000,     // giữ cache 5 phút sau khi không còn observer
});
```

`staleTime` tương ứng `max-age` (bao lâu coi là tươi), `gcTime` (trước đây là `cacheTime`) tương ứng thời gian giữ trong bộ nhớ. Khi data stale, TanStack Query vẫn hiển thị bản cũ ngay rồi refetch ngầm — chính là stale-while-revalidate ở phía client.

**Bẫy**: nhầm `no-cache` (vẫn cache nhưng phải revalidate) với `no-store` (cấm cache hoàn toàn). Và `staleTime` mặc định của TanStack Query là `0`, nghĩa là mọi data bị coi là stale ngay lập tức — nhiều người ngạc nhiên vì sao query refetch liên tục khi mount lại.

</details>

---

### 9.6. So sánh cookie, localStorage và sessionStorage. Khi nào dùng cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

| Tiêu chí | Cookie | localStorage | sessionStorage |
|----------|--------|--------------|----------------|
| Dung lượng | ~4KB | ~5-10MB | ~5-10MB |
| Tự gửi kèm request | Có (mọi request tới domain) | Không | Không |
| Truy cập từ JS | Có (trừ `httpOnly`) | Có | Có |
| Thời gian sống | Tùy `Expires`/`Max-Age` | Vĩnh viễn tới khi xóa | Hết khi đóng tab |
| Chia sẻ giữa tab | Có | Có | Không (riêng từng tab) |
| Gửi tới server | Tự động | Phải tự gắn vào header | Phải tự gắn vào header |

**Khi nào dùng cái nào**:
- **Cookie (đặc biệt `httpOnly`)**: lưu token xác thực, session — vì tự gửi kèm request và JS không đọc được (chống XSS).
- **localStorage**: dữ liệu không nhạy cảm cần giữ lâu — theme, ngôn ngữ, draft, "đã xem onboarding".
- **sessionStorage**: dữ liệu chỉ cần trong một phiên/một tab — bước của một form wizard, state tạm.

```ts
// localStorage cho preference UI
localStorage.setItem('theme', 'dark');
const theme = localStorage.getItem('theme');

// sessionStorage tự xóa khi đóng tab
sessionStorage.setItem('wizardStep', '2');
```

**Bẫy trong Next.js (SSR)**: `localStorage`/`sessionStorage` chỉ tồn tại trên trình duyệt. Truy cập trong lúc render server (hoặc trong component render đầu) sẽ lỗi `localStorage is not defined`. Cần đọc trong `useEffect` hoặc kiểm tra `typeof window !== 'undefined'`.

```tsx
useEffect(() => {
  const saved = localStorage.getItem('theme'); // an toàn, chỉ chạy trên client
  if (saved) setTheme(saved);
}, []);
```

</details>

---

### 9.7. Nên lưu JWT ở đâu cho an toàn: httpOnly cookie hay localStorage? Hãy phân tích đánh đổi.

<details>
<summary>👉 Xem câu trả lời</summary>

Đây là câu hỏi "kinh điển" và câu trả lời tốt là **phân tích đánh đổi giữa XSS và CSRF**, không phải chọn cứng một bên.

| | localStorage | httpOnly cookie |
|--|--------------|-----------------|
| Bị đọc bởi JS độc (XSS) | **Có** — script đánh cắp token dễ dàng | **Không** — JS không đọc được |
| Tự gửi kèm request | Không (tự gắn header `Authorization`) | Có |
| Rủi ro CSRF | Không (vì không tự gửi) | **Có** — cần phòng chống thêm |
| Hợp với SSR/Next.js | Khó (server không đọc được) | Tốt (server đọc cookie khi SSR) |

**Quan điểm được khuyến nghị**: lưu trong **`httpOnly` + `Secure` + `SameSite` cookie**. Lý do: XSS nguy hiểm hơn nhiều — nếu bị XSS thì localStorage coi như mất token hoàn toàn. Còn rủi ro CSRF của cookie có thể giảm mạnh bằng `SameSite=Lax/Strict` và CSRF token.

```http
Set-Cookie: token=eyJ...; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=900
```

Mô hình thực tế thường gặp: **access token ngắn hạn** + **refresh token trong httpOnly cookie**:
- Access token (sống ngắn, 5-15 phút): có thể giữ trong bộ nhớ JS (biến/Context), mất khi reload thì xin lại.
- Refresh token: httpOnly cookie, dùng để cấp access token mới qua endpoint `/refresh`.

```tsx
// Access token giữ trong memory, không persist -> giảm bề mặt tấn công
let accessToken: string | null = null;
export const setToken = (t: string) => { accessToken = t; };

fetch('/api/refresh', { credentials: 'include' }) // refresh cookie tự gửi
  .then(r => r.json())
  .then(d => setToken(d.accessToken));
```

**Bẫy**: nói "localStorage an toàn vì JS của tôi tự quản lý" — sai, bất kỳ script bên thứ ba hoặc lỗ hổng XSS nào cũng đọc được localStorage. Và nếu dùng cookie phải nhớ `Secure` (chỉ gửi qua HTTPS) + `SameSite`.

</details>

---

### 9.8. So sánh WebSocket, Server-Sent Events (SSE) và polling. Khi nào chọn cái nào trong một ứng dụng React?

<details>
<summary>👉 Xem câu trả lời</summary>

| | Polling | SSE | WebSocket |
|--|---------|-----|-----------|
| Hướng dữ liệu | Client hỏi liên tục | Server → client (1 chiều) | 2 chiều, full-duplex |
| Giao thức | HTTP request lặp lại | HTTP (giữ kết nối) | TCP nâng cấp từ HTTP |
| Tự reconnect | Tự code | Có sẵn (built-in) | Tự code |
| Độ phức tạp | Thấp | Trung bình | Cao |
| Hợp với | Dữ liệu đổi chậm | Feed/notification, log stream | Chat, game, collaborative editing |

**Polling** (đơn giản nhất): hỏi server theo chu kỳ. TanStack Query làm việc này dễ:

```tsx
useQuery({
  queryKey: ['status'],
  queryFn: fetchStatus,
  refetchInterval: 5000, // polling mỗi 5s
});
```

**SSE**: một chiều server → client, tự reconnect, dùng `EventSource`. Tốt cho thông báo, cập nhật giá, stream token của AI:

```tsx
useEffect(() => {
  const es = new EventSource('/api/notifications');
  es.onmessage = (e) => setNotifications(prev => [...prev, JSON.parse(e.data)]);
  return () => es.close();
}, []);
```

**WebSocket**: hai chiều, độ trễ thấp, dùng khi cả client lẫn server cần đẩy dữ liệu liên tục (chat, hiển thị con trỏ realtime):

```tsx
useEffect(() => {
  const ws = new WebSocket('wss://api.example.com/chat');
  ws.onmessage = (e) => addMessage(JSON.parse(e.data));
  return () => ws.close();
}, []);
```

**Cách chọn**: bắt đầu từ đơn giản nhất đủ dùng. Cần realtime một chiều → SSE (nhẹ, tự reconnect, chạy trên HTTP). Cần hai chiều độ trễ thấp → WebSocket. Dữ liệu đổi không thường xuyên → polling đủ.

**Bẫy**: quên cleanup (`es.close()`/`ws.close()`) trong `useEffect` → rò rỉ kết nối khi component unmount, hoặc tạo nhiều kết nối trùng khi re-render. SSE giới hạn ~6 kết nối đồng thời mỗi domain trên HTTP/1.1 (HTTP/2 thì không bị giới hạn này).

</details>

---

### 9.9. API trả về `429 Too Many Requests`. Bạn xử lý rate limit ở frontend như thế nào? Giải thích exponential backoff.

<details>
<summary>👉 Xem câu trả lời</summary>

**Exponential backoff**: khi request thất bại do quá tải/rate limit, thay vì retry ngay, ta **tăng dần thời gian chờ theo cấp số nhân** (1s, 2s, 4s, 8s...) để giảm áp lực lên server. Thêm **jitter** (ngẫu nhiên hóa) để tránh nhiều client cùng retry đồng loạt ("thundering herd").

Quy tắc quan trọng: nếu response có header **`Retry-After`**, phải tôn trọng nó thay vì tự tính.

```ts
async function fetchWithBackoff(url: string, maxRetries = 4): Promise<Response> {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    const res = await fetch(url);
    if (res.status !== 429 && res.status < 500) return res;
    if (attempt === maxRetries) return res;

    // Ưu tiên Retry-After do server chỉ định
    const retryAfter = res.headers.get('Retry-After');
    const baseDelay = retryAfter
      ? Number(retryAfter) * 1000
      : Math.min(1000 * 2 ** attempt, 30_000); // 1s,2s,4s,8s... tối đa 30s
    const jitter = Math.random() * 300;
    await new Promise(r => setTimeout(r, baseDelay + jitter));
  }
  throw new Error('Max retries exceeded');
}
```

**TanStack Query** có sẵn cơ chế retry với backoff:

```tsx
useQuery({
  queryKey: ['data'],
  queryFn: fetchData,
  retry: (failureCount, error) => failureCount < 3 && error.status !== 404,
  retryDelay: (attempt) => Math.min(1000 * 2 ** attempt, 30_000), // exponential
});
```

**Phòng ngừa từ đầu (đừng để bị 429)**: debounce/throttle input tìm kiếm, dedupe request trùng (TanStack Query tự dedupe theo `queryKey`), tránh gọi API trong vòng lặp render.

**Bẫy**: retry vô hạn hoặc retry cả lỗi `4xx` không thể phục hồi (`400`, `404`) — vô nghĩa và làm nặng thêm. Chỉ retry `429` và `5xx`. Và đừng quên giới hạn số lần retry tối đa.

</details>

---

### 9.10. XSS là gì? React bảo vệ bạn khỏi XSS như thế nào, và khi nào React KHÔNG bảo vệ bạn?

<details>
<summary>👉 Xem câu trả lời</summary>

**XSS (Cross-Site Scripting)**: kẻ tấn công chèn mã JS độc vào trang để chạy trong trình duyệt nạn nhân, từ đó đánh cắp cookie/token, mạo danh người dùng.

**React bảo vệ mặc định**: mọi giá trị nhúng qua `{}` trong JSX đều được **escape tự động**. Nên dù chuỗi chứa thẻ HTML, nó hiển thị dưới dạng text chứ không được thực thi:

```tsx
const userInput = '<img src=x onerror="alert(document.cookie)" />';
return <div>{userInput}</div>; // An toàn: hiển thị nguyên văn chuỗi, KHÔNG chạy
```

**Khi nào React KHÔNG bảo vệ**:

1. **`dangerouslySetInnerHTML`** — bỏ qua escape, render HTML thô:

```tsx
// NGUY HIỂM nếu html đến từ người dùng
<div dangerouslySetInnerHTML={{ __html: userHtml }} />

// AN TOÀN: sanitize trước bằng DOMPurify
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userHtml) }} />
```

2. **Inject vào thuộc tính nhạy cảm**, đặc biệt `href`/`src` với `javascript:`:

```tsx
// NGUY HIỂM: userUrl = "javascript:alert(document.cookie)"
<a href={userUrl}>Link</a>
// Cần validate scheme là http/https trước khi render
```

3. **Truyền dữ liệu vào API thao tác DOM trực tiếp** (`ref.current.innerHTML = ...`), hoặc dùng thư viện render HTML không an toàn.

**Phòng thủ tầng cuối — Content Security Policy (CSP)**: header hạn chế nguồn script được phép chạy, giảm thiệt hại ngay cả khi có lỗ hổng:

```http
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none'
```

Trong Next.js có thể đặt CSP qua `headers()` trong `next.config.js` hoặc middleware (lưu ý cần cấu hình nonce cho inline script).

**Bẫy**: nghĩ "dùng React là miễn nhiễm XSS". Sai — React chỉ an toàn cho nội dung qua `{}`. `dangerouslySetInnerHTML`, URL `javascript:`, và thao tác DOM thủ công đều là cửa ngõ. Luôn sanitize HTML do người dùng cung cấp.

</details>

---

### 9.11. CSRF là gì và khác XSS thế nào? Là frontend developer, bạn góp phần phòng chống CSRF ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

**CSRF (Cross-Site Request Forgery)**: kẻ tấn công lừa trình duyệt nạn nhân **gửi request tới site đã đăng nhập** mà nạn nhân không hay biết, lợi dụng việc **cookie tự động gửi kèm**. Ví dụ: nạn nhân đang đăng nhập bank.com, truy cập site độc có form ẩn tự submit `POST bank.com/transfer` — cookie session vẫn được gửi kèm.

**Khác biệt cốt lõi với XSS**:

| | XSS | CSRF |
|--|-----|------|
| Bản chất | Chạy mã JS độc trong trang nạn nhân | Lừa trình duyệt gửi request hợp lệ |
| Khai thác | Lỗ hổng escape/sanitize | Cơ chế tự gửi cookie |
| Cần JS chạy trên site? | Có | Không cần |
| Phòng chống chính | Escape + sanitize + CSP | SameSite cookie + CSRF token |

**Phòng chống (FE phối hợp BE)**:

1. **`SameSite` cookie** — biện pháp mạnh nhất hiện nay. `Lax` (mặc định trình duyệt hiện đại) chặn cookie trong request cross-site dạng POST/AJAX; `Strict` chặt hơn nữa:

```http
Set-Cookie: session=...; HttpOnly; Secure; SameSite=Lax
```

2. **CSRF token** (Synchronizer Token / Double Submit): server cấp token, FE đọc và gắn vào header mỗi request thay đổi state:

```ts
// FE đọc token (ví dụ từ cookie không-httpOnly hoặc meta tag) và gắn vào header
const csrf = getCsrfToken();
fetch('/api/transfer', {
  method: 'POST',
  credentials: 'include',
  headers: { 'X-CSRF-Token': csrf, 'Content-Type': 'application/json' },
  body: JSON.stringify({ amount: 100 }),
});
```

Token bí mật này site độc không đọc được (Same-Origin Policy), nên không thể giả mạo request hợp lệ.

**Bẫy**: nếu token lưu trong **localStorage và gắn header `Authorization`** thay vì cookie, thì về cơ bản **miễn nhiễm CSRF** (vì không có gì tự gửi) — nhưng lại đánh đổi rủi ro XSS. Đây chính là lý do câu chuyện lưu JWT (câu 9.7) luôn gắn với CSRF/XSS. Đừng tự tin rằng `SameSite=Lax` chặn được mọi thứ — một số request GET vẫn lọt, nên các thao tác đổi state không bao giờ nên dùng GET.

</details>

---

### 9.12. Nhìn từ góc độ frontend, GraphQL khác REST như thế nào? Khi nào bạn chọn cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

| Tiêu chí | REST | GraphQL |
|----------|------|---------|
| Endpoint | Nhiều, theo resource | Một endpoint `/graphql` |
| Lấy dữ liệu | Server quyết định shape | Client tự khai báo field cần |
| Over/under-fetching | Hay gặp | Lấy đúng field cần |
| Nhiều resource | Thường phải gọi nhiều request | Một query gộp được |
| Caching HTTP | Tận dụng cache trình duyệt/CDN dễ | Khó hơn (thường POST), cần cache ở tầng client |
| Status code | Dùng status HTTP | Thường trả `200` kèm mảng `errors` |

**Lợi ích GraphQL nhìn từ FE**:
- Tránh **over-fetching** (REST trả thừa field) và **under-fetching** (phải gọi nhiều endpoint rồi ghép).
- Một query lấy được dữ liệu lồng nhau từ nhiều resource:

```graphql
query {
  user(id: "1") {
    name
    posts(last: 5) { title comments { text } }
  }
}
```

**Lợi ích REST nhìn từ FE**:
- Đơn giản, tận dụng **HTTP caching/CDN/ETag** sẵn có (vì GET có URL rõ ràng).
- Dễ debug bằng Network tab, status code rõ nghĩa.
- Không cần thêm tầng client phức tạp.

**Cách chọn**: REST hợp với CRUD đơn giản, cần CDN cache mạnh, hoặc API public ổn định. GraphQL hợp khi UI cần nhiều shape dữ liệu khác nhau, dữ liệu lồng nhiều cấp, nhiều client (web/mobile) với nhu cầu field khác nhau, muốn giảm round-trip.

```tsx
// GraphQL với TanStack Query (không nhất thiết phải Apollo)
useQuery({
  queryKey: ['user', id],
  queryFn: () => graphqlRequest(USER_QUERY, { id }),
});
```

**Bẫy cần biết về GraphQL phía FE**:
- Lỗi thường trả về với status `200` kèm field `errors`, nên không thể chỉ dựa vào `res.ok` — phải kiểm tra body.
- HTTP caching khó hơn (request thường là POST), nên thường dựa vào normalized cache của Apollo Client / urql.
- Có rủi ro query quá nặng/lồng sâu (server cần giới hạn độ phức tạp). FE cũng nên cẩn thận với fragment trùng lặp.

</details>
## 10. System Design cho Frontend

### 10.1. Bạn sẽ tổ chức kiến trúc thư mục cho một ứng dụng React lớn (100+ trang, nhiều team) như thế nào? Vì sao chọn feature-based thay vì gom theo loại file?

<details>
<summary>👉 Xem câu trả lời</summary>

Với ứng dụng lớn, cách gom file theo **kỹ thuật** (tất cả `components/`, tất cả `hooks/`, tất cả `services/`) sẽ sụp đổ: mỗi lần làm một tính năng bạn phải nhảy qua 5-6 thư mục, và không ai biết xoá file nào khi gỡ tính năng. Giải pháp là **feature-based (vertical slice)**: nhóm theo nghiệp vụ, mỗi feature tự chứa component, hook, API, state, type của riêng nó.

Một cấu trúc phổ biến (lấy cảm hứng từ *bulletproof-react*):

```
src/
  app/                 # routing, providers, layout gốc
  features/
    auth/
      components/
      api/             # hàm gọi API + hook query/mutation
      hooks/
      stores/          # state cục bộ của feature
      types.ts
      index.ts         # public API của feature (chỉ export thứ cần ra ngoài)
    billing/
    projects/
  components/          # UI dùng chung (Button, Modal) — KHÔNG chứa nghiệp vụ
  lib/                 # cấu hình axios, query client, dayjs...
  hooks/               # hook hạ tầng dùng chung (useDebounce...)
  config/  utils/  types/
```

Nguyên tắc cốt lõi để quy mô không loạn:

- **Luồng phụ thuộc một chiều**: `shared (components/lib/hooks)` ← `features` ← `app`. Feature được dùng shared, app được dùng feature, nhưng **feature không import chéo feature** (vd `billing` không `import` thẳng vào trong `auth/`). Nếu cần chia sẻ, đẩy phần dùng chung lên `shared` hoặc giao tiếp qua `app`.
- **Public API qua `index.ts`**: bên ngoài chỉ import `features/auth`, không chọc thẳng `features/auth/components/LoginForm/internal-helper`. Có thể ép bằng ESLint (`no-restricted-imports`) hoặc Nx module boundaries.
- **Ownership rõ ràng**: mỗi folder feature map vào một team qua `CODEOWNERS`, giảm xung đột merge và review chéo.

| Tiêu chí | Type-based | Feature-based |
|---|---|---|
| Tìm code 1 tính năng | Rải rác nhiều folder | Một chỗ |
| Xoá tính năng | Phải dò khắp nơi | Xoá 1 folder |
| Co-location (test, style) | Khó | Dễ |
| Quy mô 100+ trang | Khó duy trì | Tốt |

**Bẫy thường gặp**: tạo một thư mục `shared/` hoặc `utils/` khổng lồ thành "bãi rác" — mọi thứ không biết để đâu thì nhét vào, dần thành cục coupling ngầm. Quy ước: chỉ lên `shared` khi thực sự có **2+ feature** dùng chung.

</details>

---

### 10.2. Khi xây một component library / design system dùng chung cho nhiều team, bạn quan tâm những vấn đề kỹ thuật nào? Trình bày từ API component, versioning đến tài liệu.

<details>
<summary>👉 Xem câu trả lời</summary>

Component library thành công không chỉ là "viết Button đẹp" mà là một **sản phẩm có người dùng là các dev khác**. Các trục cần lo:

**1. Thiết kế API component** — ưu tiên dễ dùng và khó dùng sai:
- Dựa trên **design tokens** thay vì giá trị cứng (`color="danger"` chứ không phải `color="#e11d48"`).
- Component **không có style cố định về layout/margin ngoài** (để nơi dùng tự lo spacing), tránh "magic margin".
- Hỗ trợ **`asChild` / polymorphic** (kiểu Radix) hoặc `as` prop để đổi thẻ HTML mà giữ style.
- **Forward ref** và **spread các thuộc tính HTML còn lại** để không chặn người dùng truyền `aria-*`, `data-*`, `onClick`.

```tsx
type ButtonProps = React.ComponentPropsWithoutRef<'button'> & {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
};

export const Button = React.forwardRef<HTMLButtonElement, ButtonProps>(
  ({ variant = 'primary', size = 'md', className, ...rest }, ref) => (
    <button
      ref={ref}
      className={cn(buttonVariants({ variant, size }), className)}
      {...rest}  // cho phép aria-*, data-*, type, onClick... đi qua
    />
  ),
);
Button.displayName = 'Button';
```

**2. Versioning & phân phối**:
- Publish lên npm (registry nội bộ) theo **semver**. Đổi prop bắt buộc / xoá variant = **breaking (major)**.
- Dùng **changesets** để quản lý changelog và bump version trong monorepo.
- Cung cấp **codemod** cho breaking change lớn để team consumer migrate tự động.

**3. Tài liệu & độ tin cậy**:
- **Storybook** làm tài liệu sống: mỗi component có stories cho các trạng thái (loading, error, disabled, RTL).
- **Visual regression** (Chromatic / Playwright snapshot) để mọi thay đổi style đều được duyệt bằng mắt.
- Test accessibility tự động (`jest-axe`, addon a11y của Storybook).

**Bẫy thường gặp**:
- **Quá linh hoạt** (nhận cả `style` lẫn `className` lẫn 20 props) → mỗi team dùng một kiểu, design system mất tác dụng thống nhất.
- **Quá cứng** → team consumer không tuỳ biến được, họ fork hoặc tự viết lại → library bị bỏ.
- Phụ thuộc lẫn nhau giữa "component" và "app shell" (vd component tự gọi API hoặc đọc router) — component dùng chung **phải là presentational**, không biết về data fetching.

</details>

---

### 10.3. Thiết kế frontend cho ứng dụng chịu tải 1 triệu+ user/ngày: bạn dùng những kỹ thuật nào về CDN, cache và rendering để giảm tải server và tăng tốc?

<details>
<summary>👉 Xem câu trả lời</summary>

Ý tưởng trung tâm: **đẩy càng nhiều thứ ra càng gần user càng tốt**, và **render càng sớm càng tốt** để mỗi request không phải đụng tới origin server. Theo lớp:

**1. Tĩnh hoá + CDN**:
- Asset bất biến (JS/CSS bundle có hash trong tên) đặt `Cache-Control: public, max-age=31536000, immutable` → trình duyệt và CDN cache vĩnh viễn; đổi nội dung = đổi hash = cache busting tự nhiên.
- HTML thì cache ngắn hoặc dùng **stale-while-revalidate** để vừa nhanh vừa cập nhật.

**2. Chọn chiến lược render (Next.js)** theo từng loại trang:

| Trang | Chiến lược | Lý do |
|---|---|---|
| Landing, blog, docs | **SSG / ISR** | Build sẵn, phục vụ từ CDN, gần như không chạm server |
| Trang sản phẩm thay đổi vừa phải | **ISR** (`revalidate`) | Tĩnh nhưng tự làm mới sau N giây |
| Dashboard cá nhân hoá | **SSR / streaming** hoặc CSR | Dữ liệu theo user, không cache chung được |

ISR cực mạnh cho quy mô lớn: 1 triệu lượt xem một trang sản phẩm chỉ tạo ra **một** lần regenerate mỗi `revalidate` giây, phần còn lại serve từ cache CDN.

```tsx
// Next.js App Router — ISR: phục vụ tĩnh, tự làm mới mỗi 60s
export const revalidate = 60;

export default async function ProductPage({ params }) {
  const product = await getProduct(params.id); // chỉ chạy khi cache hết hạn
  return <ProductView product={product} />;
}
```

**3. Cache nhiều tầng**:
- **Browser cache** (asset bất biến) → **CDN edge cache** (HTML/ISR) → **Data cache** (Next fetch cache / Redis) → DB. Mỗi tầng chặn bớt request xuống tầng dưới.
- Tách dữ liệu **per-user** (không cache hoặc cache ở client với TanStack Query) khỏi dữ liệu **dùng chung** (cache mạnh ở edge).

**4. Tối ưu payload để chịu băng thông**:
- Code splitting theo route, lazy-load phần dưới màn hình.
- `next/image` cho ảnh responsive + định dạng hiện đại (AVIF/WebP) qua CDN ảnh.
- Brotli/gzip, HTTP/2-3, preconnect tới origin API.

**Bẫy thường gặp**: cá nhân hoá toàn trang (header chào tên user) khiến **không cache được gì cả**. Cách xử lý: render khung tĩnh ở edge, chèn phần cá nhân hoá bằng client component / streaming, để phần lớn HTML vẫn cache chung được.

</details>

---

### 10.4. Micro-frontend là gì, khi nào nên dùng và khi nào KHÔNG? So sánh các cách triển khai (Module Federation, route-based, iframe).

<details>
<summary>👉 Xem câu trả lời</summary>

**Micro-frontend (MFE)** là kiến trúc chia một ứng dụng web lớn thành nhiều phần độc lập, mỗi phần do một team sở hữu và **deploy riêng**, rồi ghép lại lúc chạy hoặc lúc build. Mục tiêu chính là **độc lập về tổ chức và deploy**, không phải hiệu năng.

**Khi NÊN dùng**:
- Nhiều team lớn (chục+ dev) cần release độc lập, không muốn chờ nhau và chờ một CI monolith khổng lồ.
- Các phần nghiệp vụ tách bạch (vd `Checkout`, `Search`, `Account` của một sàn TMĐT).
- Có nhu cầu mix tech/version (đang migrate dần khỏi framework cũ).

**Khi KHÔNG nên**:
- Team nhỏ (< 15-20 dev) → chi phí hạ tầng, chia sẻ dependency, debug cross-app **lớn hơn lợi ích**. Một monorepo modular thường đủ.
- Yêu cầu hiệu năng cực cao (MFE dễ tải trùng React, tăng bundle).

**So sánh cách triển khai**:

| Cách | Cơ chế | Ưu | Nhược |
|---|---|---|---|
| **Module Federation** (Webpack 5 / Vite plugin) | Tải code remote lúc runtime, **chia sẻ dependency** (React singleton) | Tích hợp mượt, chung 1 DOM, chia sẻ lib | Phải đồng bộ version shared dep, cấu hình phức tạp |
| **Route-based / app shell** | Mỗi MFE là một app riêng, ghép qua reverse proxy theo path | Đơn giản, cô lập tốt | Chuyển route giữa MFE thường full reload |
| **iframe** | Nhúng app khác qua `<iframe>` | Cô lập tuyệt đối (CSS/JS) | Khó chia sẻ state, routing/SEO/UX kém |

Trong hệ sinh thái Next.js, có thể dùng **Multi-Zones** (mỗi zone là một Next app phục vụ một nhánh path) — một dạng MFE route-based "chính chủ".

**Bẫy thường gặp**:
- **Tải trùng React/lib** → bundle phình, lỗi "multiple React instances" (hook crash). Phải cấu hình shared singleton cẩn thận.
- **Chia sẻ state cross-MFE** dễ thành coupling ngầm; nên giao tiếp qua sự kiện/URL/contract rõ ràng, không share Redux store toàn cục.
- Đồng bộ **design system** giữa các MFE — cần một component library chung, nếu không UI vênh nhau.

</details>

---

### 10.5. Monorepo cho frontend (Turborepo / Nx): giải quyết vấn đề gì? Cấu trúc ra sao và build/CI được tối ưu thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

**Monorepo** là một repo chứa nhiều app + nhiều package dùng chung. Nó giải quyết các đau đầu của multi-repo: chia sẻ code (design system, util, type) phải publish/bump version liên tục; thay đổi xuyên repo phải nhiều PR; phiên bản lệch nhau.

**Cấu trúc điển hình** (pnpm workspaces + Turborepo):

```
apps/
  web/            # Next.js app khách hàng
  admin/          # Next.js app nội bộ
packages/
  ui/             # design system dùng chung
  config/         # eslint, tsconfig, tailwind preset dùng chung
  api-client/     # SDK gọi API + type
  utils/
turbo.json
pnpm-workspace.yaml
```

Lợi ích chính:
- **Atomic change**: sửa `packages/ui` và cập nhật `apps/web` trong **một PR**, một lần review — không cần publish npm trung gian.
- **Type-safe end-to-end**: app import trực tiếp type từ package nội bộ, đổi type là báo lỗi compile ngay ở app dùng.
- **Chuẩn hoá tooling** qua `packages/config`.

**Tối ưu build/CI (điểm Turborepo/Nx tỏa sáng)**:
- **Task graph + chỉ chạy thứ bị ảnh hưởng**: dựa trên đồ thị phụ thuộc, chỉ build/test package đã đổi và những thứ phụ thuộc nó (`turbo run build --filter=...[origin/main]`, Nx `affected`).
- **Caching**: hash input (source + deps + config); nếu không đổi thì **dùng lại output** từ cache, không chạy lại.
- **Remote cache**: chia sẻ cache giữa các máy CI và máy dev → người khác/CI hưởng kết quả build bạn đã chạy.

```jsonc
// turbo.json
{
  "tasks": {
    "build": {
      "dependsOn": ["^build"],     // build dependency trước
      "outputs": [".next/**", "dist/**"]
    },
    "test": { "dependsOn": ["^build"] },
    "lint": {},
    "dev": { "cache": false, "persistent": true }
  }
}
```

| | Turborepo | Nx |
|---|---|---|
| Triết lý | Nhẹ, "thêm vào" repo có sẵn | Toàn diện, nhiều generator/plugin |
| Module boundaries | Tự lo (ESLint) | Có sẵn lint rule ràng buộc tag |
| Đường cong học | Thoải hơn | Dốc hơn, nhiều convention |

**Bẫy thường gặp**: cấu hình `outputs`/`inputs` sai khiến cache không hit (build lại hoài) hoặc cache "bẩn" (dùng lại output cũ). Và monorepo **không tự** tạo ranh giới module — vẫn cần ESLint boundaries để package không import lung tung.

</details>

---

### 10.6. Design tokens là gì và vai trò của chúng trong một design system ở quy mô lớn (nhiều brand, nhiều theme, dark mode)?

<details>
<summary>👉 Xem câu trả lời</summary>

**Design tokens** là các quyết định thiết kế được lưu dưới dạng **biến có tên, độc lập nền tảng** (màu, khoảng cách, font, bo góc, shadow...). Thay vì hardcode `#1e40af`, ta tham chiếu `color.action.primary`. Chúng là **nguồn chân lý duy nhất** nối giữa Figma và code.

Cách tổ chức tốt là chia **tầng token** (token layering):

```
1. Primitive (global)  : blue-500 = #3b82f6, space-4 = 16px   → giá trị thô
2. Semantic (alias)     : color.action.primary = {blue-500}    → ý nghĩa
3. Component            : button.bg = {color.action.primary}   → ngữ cảnh cụ thể
```

Lớp **semantic** chính là chìa khoá cho multi-theme: component chỉ dùng token semantic, còn việc đổi theme = đổi ánh xạ semantic → primitive.

```css
/* Theme bằng CSS variables — đổi token theo data-theme, component không đổi */
:root {
  --color-action-primary: #3b82f6; /* light: blue-500 */
  --color-bg: #ffffff;
}
[data-theme='dark'] {
  --color-action-primary: #60a5fa; /* dark: blue-400 */
  --color-bg: #0a0a0a;
}
```
```tsx
// Button luôn dùng token semantic, không quan tâm đang ở theme/brand nào
<button style={{ background: 'var(--color-action-primary)' }}>Lưu</button>
```

Lợi ích ở quy mô lớn:
- **Đổi dark mode / brand B** chỉ cần đổi tập giá trị token, không sửa hàng trăm component.
- **Đồng bộ Figma ↔ code**: dùng chuẩn **W3C Design Tokens** (file `tokens.json`) + **Style Dictionary** để build ra CSS vars, JS object, Tailwind config, iOS/Android cùng lúc → "1 nguồn, nhiều nền tảng".

| Cách | Tác động khi rebrand | Đa nền tảng |
|---|---|---|
| Hardcode giá trị | Sửa khắp nơi, dễ sót | Không |
| Chỉ primitive token | Đỡ hơn nhưng vẫn rối khi đổi nghĩa | Một phần |
| Primitive + semantic | Đổi 1 lớp ánh xạ | Tốt |

**Bẫy thường gặp**: bỏ qua lớp semantic, component dùng thẳng `blue-500` → khi dark mode cần màu khác, lại phải sửa từng component. Và đặt tên token theo **giá trị** (`color-blue`) thay vì **ý nghĩa** (`color-action-primary`) khiến token mất tác dụng khi brand đổi màu chủ đạo.

</details>

---

### 10.7. Làm sao đảm bảo accessibility (a11y) ở quy mô lớn — không phải chỉ một trang mà toàn bộ ứng dụng, qua nhiều team và theo thời gian?

<details>
<summary>👉 Xem câu trả lời</summary>

A11y ở quy mô lớn là bài toán **quy trình + tự động hoá**, không phải "audit một lần rồi quên". Chiến lược nhiều lớp:

**1. Bắt từ gốc — component library accessible sẵn**:
- Component dùng chung (Modal, Dropdown, Tabs...) đã đúng a11y một lần thì **mọi nơi dùng đều được hưởng**. Nên build trên **headless lib** đã lo focus trap, ARIA, keyboard nav như **Radix UI / React Aria**.
- Đây là đòn bẩy lớn nhất: sửa 1 lần ở library, hàng trăm trang đúng theo.

**2. Tự động hoá trong CI để chặn hồi quy**:
- **`jest-axe` / `@axe-core/playwright`** trong test → fail build nếu vi phạm WCAG cơ bản.
- ESLint plugin **`jsx-a11y`** bắt lỗi tĩnh (thiếu `alt`, `onClick` trên div không có role/keyboard...).
- Storybook addon a11y để dev thấy lỗi ngay khi code component.

```tsx
import { axe, toHaveNoViolations } from 'jest-axe';
expect.extend(toHaveNoViolations);

it('Form không vi phạm a11y', async () => {
  const { container } = render(<SignupForm />);
  expect(await axe(container)).toHaveNoViolations();
});
```

**3. Những thứ máy không bắt được — cần quy trình con người**:
- Tool tự động chỉ phát hiện được ~30-50% lỗi. Thứ tự logic của screen reader, ngữ cảnh `aria-label`, độ hợp lý của focus order → cần **test bằng bàn phím** (Tab/Shift+Tab/Esc) và **screen reader** (NVDA/VoiceOver) trong định kỳ QA.
- Đưa a11y vào **Definition of Done** và checklist PR review.

**4. Đặc thù SPA/React cần chú ý**:
- **Route change** không reload trang → screen reader không thông báo. Phải quản lý focus thủ công (focus vào heading trang mới) và dùng **live region** (`aria-live`) thông báo điều hướng.
- Quản lý focus khi mở/đóng modal, toast, async loading.

**Bẫy thường gặp**: lạm dụng ARIA ("ARIA thừa còn tệ hơn không có") — thêm `role`/`aria-*` sai làm screen reader hiểu nhầm; ưu tiên **HTML ngữ nghĩa** (`<button>`, `<nav>`, `<label>`) trước. Và chỉ kiểm a11y bằng tool tự động rồi tưởng đã "đạt chuẩn".

</details>

---

### 10.8. Thiết kế chiến lược xử lý lỗi toàn cục cho một ứng dụng React/Next.js: error boundary, lỗi network qua interceptor, và báo cáo lỗi (observability).

<details>
<summary>👉 Xem câu trả lời</summary>

Cần phân biệt **hai loại lỗi** vì chúng được xử lý ở hai nơi khác nhau:

| Loại lỗi | Ví dụ | Bắt ở đâu |
|---|---|---|
| Lỗi **render** (đồng bộ trong React) | đọc `user.name` khi `user` null | **Error Boundary** |
| Lỗi **bất đồng bộ** (network, promise) | API trả 500, mất mạng | **try/catch / interceptor / state của query** |

Error boundary **không** bắt lỗi trong event handler, async, hay SSR — đó là lý do cần lớp thứ hai.

**1. Error Boundary phân tầng**:
- **Root boundary** (Next App Router: `app/global-error.tsx`, `app/error.tsx`) → chặn crash trắng màn hình, hiện UI fallback thân thiện + nút "Thử lại".
- **Boundary cục bộ** quanh widget rủi ro (biểu đồ, embed bên thứ ba) → một widget hỏng không sập cả trang.

```tsx
// Next.js App Router: error.tsx là error boundary cho segment đó
'use client';
export default function Error({ error, reset }: { error: Error; reset: () => void }) {
  useEffect(() => { reportError(error); }, [error]); // gửi về Sentry
  return (
    <div role="alert">
      <p>Đã có lỗi xảy ra. Vui lòng thử lại.</p>
      <button onClick={reset}>Thử lại</button>
    </div>
  );
}
```

**2. Lỗi network qua interceptor (tập trung)**:
- Một **axios interceptor** (hoặc wrapper fetch) xử lý chéo toàn app: 401 → refresh token / đẩy về login; 403 → trang cấm; 5xx → toast chung + log; retry các lỗi tạm thời.

```ts
api.interceptors.response.use(
  (res) => res,
  (error) => {
    const status = error.response?.status;
    if (status === 401) redirectToLogin();
    else if (status >= 500) { toast.error('Lỗi hệ thống, thử lại sau.'); reportError(error); }
    return Promise.reject(error); // vẫn ném tiếp để TanStack Query xử lý cục bộ
  },
);
```

- Kết hợp **TanStack Query**: `onError` toàn cục ở `QueryCache`/`MutationCache` cho thông báo chung, còn lỗi đặc thù thì xử lý cục bộ trong component (hiện inline cạnh form).

**3. Observability**:
- Tích hợp **Sentry** (hoặc tương đương) bắt lỗi runtime + Promise rejection chưa xử lý, kèm **source map** để stack trace đọc được, **release tag** để biết lỗi xuất hiện ở bản nào, và **breadcrumb/user context** để tái hiện.

**Bẫy thường gặp**: nuốt lỗi im lặng (`catch (e) {}`) → user thấy app "đơ" mà không ai biết; hoặc bắn toast lỗi trùng lặp khi nhiều query cùng fail. Cần một cơ chế thông báo lỗi **gộp/dedupe** và phân biệt lỗi nên hiện cho user với lỗi chỉ nên log.

</details>

---

### 10.9. "Performance budget" là gì? Bạn đặt budget thế nào và làm sao ép buộc nó trong quy trình phát triển/CI để không bị "trôi" theo thời gian?

<details>
<summary>👉 Xem câu trả lời</summary>

**Performance budget** là các **ngưỡng định lượng** đặt ra cho hiệu năng (kích thước bundle, thời gian tải, chỉ số trải nghiệm) mà sản phẩm cam kết không vượt qua. Mục đích là biến hiệu năng thành thứ **đo được và bắt buộc**, chứ không phải "thấy hơi chậm thì sửa".

**Đặt budget theo 3 nhóm**:

| Nhóm | Ví dụ ngưỡng |
|---|---|
| **Số lượng/kích thước** | JS đầu trang ≤ 170KB (gzip); tổng ảnh ≤ 1MB |
| **Chỉ số thời gian (Core Web Vitals)** | LCP < 2.5s, INP < 200ms, CLS < 0.1 |
| **Theo route** | Mỗi route mới chỉ được thêm tối đa N KB |

Cách chọn con số: dựa trên thiết bị/mạng của **người dùng thật** (vd mobile 4G tầm trung), tham chiếu Core Web Vitals của Google, không phải máy dev xịn.

**Ép buộc trong CI (quan trọng nhất — budget không có "răng" thì sẽ trôi)**:
- **Bundle size check**: `size-limit` hoặc `bundlesize` chạy trong CI, **fail PR** nếu vượt ngưỡng.
- **Lighthouse CI**: chạy Lighthouse tự động trên PR, assert ngưỡng (`assertions`) cho LCP/CLS/JS size.
- **Theo dõi xu hướng**: dashboard (vd của Next.js bundle analyzer, hoặc Calibre/SpeedCurve) để thấy đồ thị tăng dần.

```jsonc
// .size-limit.json — chặn PR nếu bundle vượt ngưỡng
[
  { "name": "App shell JS", "path": ".next/static/**/*.js", "limit": "170 KB" }
]
```

**Kỹ thuật giữ trong budget** (khi bị vượt): code splitting theo route + `dynamic()`/`React.lazy`, tree-shaking, thay lib nặng (moment → dayjs), `next/dynamic` cho component nặng, lazy ảnh, phân tích bằng `@next/bundle-analyzer` để biết "ai" làm phình.

**Bẫy thường gặp**:
- Chỉ đo trên máy dev mạnh + mạng nhanh → budget ảo. Phải đo điều kiện thực tế (throttle).
- Budget chỉ là "hướng dẫn tham khảo" không gắn vào CI → 6 tháng sau bundle gấp đôi mà không ai hay (death by a thousand cuts — mỗi PR thêm vài KB).
- Quên budget cho **bên thứ ba** (analytics, ads, chat widget) — thường đây mới là thứ làm chậm nặng nhất.

</details>

---

### 10.10. Khi thiết kế kiến trúc quản lý state cho một ứng dụng lớn, bạn phân loại state thế nào và chọn công cụ nào cho từng loại? (server state, client/UI state, form, URL)

<details>
<summary>👉 Xem câu trả lời</summary>

Sai lầm kinh điển là **nhét tất cả vào một Redux store**, kể cả dữ liệu từ server. Kiến trúc state tốt bắt đầu bằng việc **phân loại** rồi mới chọn công cụ phù hợp — vì mỗi loại state có vòng đời và nhu cầu khác nhau.

| Loại state | Đặc tính | Công cụ phù hợp |
|---|---|---|
| **Server state** (dữ liệu từ API) | Bất đồng bộ, có thể cũ, cần cache/refetch/dedupe | **TanStack Query** / RTK Query |
| **Client/UI state toàn cục** (theme, sidebar, auth client) | Đồng bộ, do client sở hữu | **Zustand** / Redux Toolkit / Context |
| **Form state** | Cục bộ, đổi liên tục, cần validate | **React Hook Form** + **Zod** |
| **URL state** (filter, tab, phân trang) | Cần share link, back/forward, reload giữ nguyên | **search params** / router |
| **State cục bộ component** | Chỉ một component dùng | `useState` / `useReducer` |

**Nguyên tắc quyết định**:
- **Server state KHÔNG nên nằm trong global store.** Nó không phải "state của bạn", nó là **cache** của dữ liệu thuộc server. Dùng TanStack Query để được caching, stale-while-revalidate, dedupe, retry, invalidation miễn phí — thay vì tự viết `loading/error/data` + `useEffect` đầy bug.

```tsx
// Server state: để TanStack Query lo cache/refetch/dedupe
const { data, isLoading, error } = useQuery({
  queryKey: ['project', id],
  queryFn: () => api.getProject(id),
  staleTime: 60_000,
});
```

- **URL là một dạng store bị quên lãng.** Filter/tab/trang nên ở URL để link chia sẻ được và reload không mất → `useSearchParams` (Next) thay vì `useState`.
- **Client state toàn cục thật sự ít hơn bạn nghĩ.** Sau khi tách server state ra TanStack Query và URL state ra router, phần còn lại (theme, trạng thái UI, auth) thường nhỏ → **Zustand** đủ và nhẹ; chỉ chọn **Redux Toolkit** khi cần devtools mạnh, middleware phức tạp, hoặc logic nghiệp vụ phía client lớn.

**So sánh nhanh client-state lib**:

| | Zustand | Redux Toolkit | Context API |
|---|---|---|---|
| Boilerplate | Rất ít | Vừa (đã giảm nhiều so với Redux cũ) | Ít |
| Re-render | Selector tinh, mượt | Selector + memo | Dễ re-render cả cây |
| Hợp khi | State global vừa/nhỏ | App lớn, cần middleware/devtools | Giá trị ít đổi (theme, locale) |

**Bẫy thường gặp**:
- Đồng bộ server data vào global store rồi tự quản lý "stale" → tái phát minh TanStack Query một cách lỗi.
- Dùng **Context cho state đổi liên tục** → mọi consumer re-render. Context hợp cho giá trị ổn định (theme/locale/auth), không hợp cho dữ liệu thay đổi nhanh.
- Đặt state lên global khi chỉ một component dùng → tăng coupling vô ích; ưu tiên đặt state **càng gần nơi dùng càng tốt**.

</details>
## 11. Testing Frontend

### 11.1. Hãy giải thích "kim tự tháp test" (test pyramid) so với "testing trophy". Là một frontend developer làm React, bạn nghiêng về mô hình nào và tại sao?

<details>
<summary>👉 Xem câu trả lời</summary>

**Test pyramid** (Mike Cohn) chia test theo tầng từ dưới lên: nhiều **unit test** ở đáy (nhanh, rẻ), ít hơn là **integration test** ở giữa, và rất ít **E2E test** ở đỉnh (chậm, đắt, dễ "flaky"). Triết lý: càng lên cao test càng chậm và giòn nên hãy giữ số lượng ít.

**Testing trophy** (Kent C. Dodds, tác giả React Testing Library) điều chỉnh lại cho frontend hiện đại, gồm 4 tầng theo chiều cao: `Static` (TypeScript, ESLint) → `Unit` → `Integration` (phình to nhất) → `E2E`. Điểm khác biệt cốt lõi: **integration test được ưu tiên nhiều nhất** vì nó cho "return on investment" cao nhất - test nhiều component phối hợp với nhau giống cách người dùng thật tương tác, mà vẫn đủ nhanh.

| Tiêu chí | Pyramid | Trophy |
|---|---|---|
| Tầng phình to nhất | Unit | Integration |
| Có tầng Static không | Không | Có (TS, lint) |
| Phù hợp nhất với | Backend, logic thuần | UI component |

```tsx
// Unit: test logic thuần, không render gì
expect(formatPrice(1000)).toBe('1.000 đ');

// Integration (ưu tiên với React): render component + tương tác như user
render(<CheckoutForm />);
await userEvent.type(screen.getByLabelText(/email/i), 'a@b.com');
await userEvent.click(screen.getByRole('button', { name: /thanh toán/i }));
expect(await screen.findByText(/đặt hàng thành công/i)).toBeInTheDocument();
```

Với React, tôi nghiêng về **testing trophy**: viết nhiều integration test render component thật + giả lập tương tác người dùng, vì test một hàm `add()` không nói lên được UI có hoạt động không. **Bẫy thường gặp**: viết quá nhiều unit test cho từng component nhỏ lẻ (test implementation detail) dẫn tới test vỡ liên tục khi refactor dù hành vi không đổi.

</details>

---

### 11.2. Triết lý cốt lõi của React Testing Library (RTL) là gì? Tại sao RTL khuyên ta tránh test "implementation detail"?

<details>
<summary>👉 Xem câu trả lời</summary>

Triết lý nổi tiếng của RTL được Kent C. Dodds tóm gọn:

> "The more your tests resemble the way your software is used, the more confidence they can give you."
> (Test càng giống cách phần mềm được sử dụng thật, càng cho bạn nhiều sự tự tin.)

Nghĩa là test nên tương tác với component **giống như người dùng cuối**: tìm phần tử qua text/role/label hiển thị, click vào nút, gõ vào input - chứ không thò tay vào state nội bộ, không gọi trực tiếp instance method, không kiểm tra props.

**Implementation detail** là những thứ người dùng không bao giờ thấy: tên state, có dùng `useReducer` hay `useState`, class CSS nội bộ, cấu trúc cây component... Test những thứ này gây ra hai vấn đề:

- **False negative**: bạn refactor (đổi `useState` thành `useReducer`) mà hành vi không đổi, nhưng test vỡ → tốn công sửa test vô ích.
- **False positive**: test pass nhưng app thực tế hỏng, vì test không phản ánh trải nghiệm người dùng.

```tsx
// ❌ Test implementation detail (với enzyme cũ) - dễ vỡ
expect(wrapper.state('count')).toBe(1);
expect(wrapper.find('Counter').prop('value')).toBe(1);

// ✅ RTL: test cái user thấy
render(<Counter />);
await userEvent.click(screen.getByRole('button', { name: /tăng/i }));
expect(screen.getByText('Số đếm: 1')).toBeInTheDocument();
```

**Lưu ý**: RTL cố tình KHÔNG cung cấp API để đọc state hay props. Nếu bạn thấy mình "cần" làm vậy, thường là dấu hiệu bạn đang test sai tầng (nên test qua hành vi đầu ra).

</details>

---

### 11.3. RTL khuyến nghị một thứ tự ưu tiên khi chọn query (getByRole, getByLabelText, getByText, getByTestId...). Hãy trình bày thứ tự đó và lý do.

<details>
<summary>👉 Xem câu trả lời</summary>

RTL chia query thành 3 nhóm theo mức độ "giống người dùng / hỗ trợ accessibility", nên ưu tiên dùng nhóm trên trước:

**1. Accessible to everyone (ưu tiên cao nhất):**
- `getByRole` - tốt nhất, vì phản ánh cả accessibility tree mà screen reader dùng. `getByRole('button', { name: /gửi/i })`.
- `getByLabelText` - lý tưởng cho form field (liên kết label ↔ input).
- `getByPlaceholderText`
- `getByText` - cho nội dung không tương tác (đoạn văn, div).
- `getByDisplayValue` - cho input đã có giá trị.

**2. Semantic queries:**
- `getByAltText` (ảnh), `getByTitle`.

**3. Test ID (lựa chọn cuối cùng):**
- `getByTestId` - chỉ dùng khi không cách nào khác (text động, không có role/label phù hợp).

```tsx
// ✅ Tốt nhất: role + accessible name
screen.getByRole('button', { name: /đăng nhập/i });
screen.getByRole('textbox', { name: /email/i });

// ✅ Ổn: form field
screen.getByLabelText(/mật khẩu/i);

// ⚠️ Chỉ khi bí: data-testid
screen.getByTestId('user-avatar');
```

**Lý do thứ tự này:** ưu tiên `getByRole` ép bạn viết HTML có ngữ nghĩa và accessible (đúng thẻ `<button>`, `<label>` gắn `htmlFor`...). Test tốt và accessibility tốt thường đi cùng nhau.

**Bẫy thường gặp:**
- Lạm dụng `getByTestId` khiến test không kiểm chứng được trải nghiệm thật.
- Nhầm lẫn biến thể query: `getBy*` (báo lỗi nếu không có), `queryBy*` (trả `null`, dùng để khẳđịnh **không tồn tại**), `findBy*` (trả Promise, dùng cho phần tử **xuất hiện bất đồng bộ**).

</details>

---

### 11.4. Jest và Vitest khác nhau ở điểm gì? Khi nào bạn chọn cái nào trong dự án React/Next.js?

<details>
<summary>👉 Xem câu trả lời</summary>

Cả hai đều là test runner + assertion + mocking, API gần như tương thích nhau (`describe`, `it`, `expect`). Khác biệt chính nằm ở kiến trúc và hệ sinh thái build.

| Tiêu chí | Jest | Vitest |
|---|---|---|
| Ra đời / chủ quản | Lâu đời, Meta | Mới, team Vite |
| Transform code | Babel / ts-jest | esbuild (rất nhanh) |
| ESM | Hỗ trợ nhưng cấu hình rườm rà | Hỗ trợ native, mượt |
| TypeScript | Cần `ts-jest`/babel | Chạy thẳng qua esbuild |
| Watch mode | Nhanh | Rất nhanh (HMR cho test) |
| Tích hợp config | Riêng | Dùng chung `vite.config.ts` |
| Hợp nhất với | Next.js, CRA cũ | Vite, dự án Vite-based |

**Khi nào chọn cái nào:**
- Dự án **Next.js**: Jest vẫn là lựa chọn mặc định được Next.js hỗ trợ chính thức (`next/jest` tự cấu hình transform, alias). Đa số dự án Next dùng Jest.
- Dự án dùng **Vite** (React + Vite, không Next): Vitest gần như là lựa chọn hiển nhiên vì dùng chung pipeline build, cấu hình tối thiểu, chạy nhanh hơn nhiều.
- Migrate Jest → Vitest tương đối dễ vì API tương đồng, nhưng cần đổi `jest.fn()` thành `vi.fn()`, `jest.mock` thành `vi.mock`.

```ts
// Vitest: cần khai báo globals hoặc import
import { describe, it, expect, vi } from 'vitest';

vi.mock('./api'); // tương đương jest.mock('./api')

// Jest với next/jest (jest.config.js)
const nextJest = require('next/jest');
const createJestConfig = nextJest({ dir: './' });
module.exports = createJestConfig({ testEnvironment: 'jsdom' });
```

**Bẫy:** đừng quên cấu hình `testEnvironment: 'jsdom'` (Jest) hoặc `environment: 'jsdom'` (Vitest), vì mặc định là `node` và sẽ không có `document`/`window` để render component.

</details>

---

### 11.5. MSW (Mock Service Worker) là gì và tại sao nó được ưa chuộng hơn mock `fetch`/`axios` trực tiếp khi test các component gọi API?

<details>
<summary>👉 Xem câu trả lời</summary>

**MSW** là thư viện mock API ở tầng **network**: nó chặn (intercept) request HTTP thực sự bằng Service Worker (trên trình duyệt) hoặc bằng cách patch lớp request (trong Node/test), rồi trả về response do bạn định nghĩa qua các "handler". Component của bạn vẫn gọi `fetch`/`axios` y như thật, không biết là đang bị mock.

**Tại sao tốt hơn mock `fetch`/`axios` trực tiếp:**
- **Không gắn chặt với implementation:** bạn mock ở tầng mạng, nên dù đổi từ `axios` sang `fetch` hay sang TanStack Query, test không cần sửa. Mock `jest.mock('axios')` thì gắn chặt vào thư viện cụ thể.
- **Dùng lại được:** cùng một bộ handler dùng cho test, cho Storybook, và cho dev (mock backend chưa xong).
- **Sát thực tế hơn:** request đi qua đúng vòng đời (serialize body, header, status code), test gần với hành vi production.

```ts
// handlers.ts
import { http, HttpResponse } from 'msw';

export const handlers = [
  http.get('/api/users', () => {
    return HttpResponse.json([{ id: 1, name: 'An' }]);
  }),
];

// setup test (Node)
import { setupServer } from 'msw/node';
const server = setupServer(...handlers);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers()); // reset override sau mỗi test
afterAll(() => server.close());
```

```ts
// Trong 1 test, override để giả lập lỗi 500
server.use(
  http.get('/api/users', () => new HttpResponse(null, { status: 500 }))
);
```

**Bẫy thường gặp:**
- Quên gọi `server.resetHandlers()` trong `afterEach` → handler override của test này "rò rỉ" sang test khác.
- Quên cài polyfill `fetch`/`TextEncoder` trong môi trường Node cũ.

</details>

---

### 11.6. Hãy phân biệt "integration test" với "unit test" trong React và cho một ví dụ integration test có ý nghĩa.

<details>
<summary>👉 Xem câu trả lời</summary>

- **Unit test**: kiểm thử một đơn vị **cô lập** - thường là một hàm thuần (utility, reducer, custom hook tính toán) - mock hết mọi phụ thuộc.
- **Integration test**: kiểm thử **nhiều phần phối hợp với nhau** - ví dụ một form gồm nhiều input + nút submit + gọi API (mock qua MSW) + hiển thị kết quả - giống một luồng người dùng thật. Đây là loại test cho giá trị cao nhất trong React.

Ranh giới không cứng nhắc; điểm mấu chốt là integration test render component thật và tương tác qua hành vi, thay vì kiểm tra từng mảnh nhỏ riêng lẻ.

```tsx
// Integration test: form đăng nhập + API (MSW) + chuyển trạng thái
test('đăng nhập thành công thì hiển thị lời chào', async () => {
  server.use(
    http.post('/api/login', () =>
      HttpResponse.json({ token: 'abc', name: 'An' })
    )
  );

  render(<LoginForm />);

  await userEvent.type(screen.getByLabelText(/email/i), 'an@example.com');
  await userEvent.type(screen.getByLabelText(/mật khẩu/i), '123456');
  await userEvent.click(screen.getByRole('button', { name: /đăng nhập/i }));

  // chờ kết quả bất đồng bộ
  expect(await screen.findByText(/xin chào, An/i)).toBeInTheDocument();
});
```

Test này bao phủ: render form, validate input, gửi request, xử lý response, cập nhật UI - tất cả trong một luồng, gần với cách user dùng thật. **Bẫy:** nếu bạn tách thành 5 unit test ("input nhận giá trị", "nút gọi handler"...) bạn test được code nhưng không chắc cả luồng có chạy.

</details>

---

### 11.7. E2E test là gì? So sánh Cypress và Playwright, và cho biết khi nào nên dùng E2E thay vì integration test với RTL.

<details>
<summary>👉 Xem câu trả lời</summary>

**E2E (end-to-end) test** chạy ứng dụng thật trong **trình duyệt thật**, với backend thật (hoặc gần thật), mô phỏng toàn bộ hành trình người dùng từ đầu đến cuối (đăng nhập → thêm sản phẩm → thanh toán). RTL chỉ render component trong jsdom (môi trường DOM giả lập trong Node), không có trình duyệt thật, không có routing/network thật ở mức trình duyệt.

| Tiêu chí | Cypress | Playwright |
|---|---|---|
| Chủ quản | Cypress.io | Microsoft |
| Trình duyệt | Chrome, Firefox, Edge, WebKit | Chromium, Firefox, WebKit |
| Ngôn ngữ | JS/TS | JS/TS, Python, Java, C# |
| Multi-tab / multi-origin | Hạn chế | Hỗ trợ tốt |
| Chạy song song | Cần Cypress Cloud (trả phí) | Có sẵn, miễn phí |
| Auto-wait | Có | Có (mạnh) |
| Mô hình chạy | Trong cùng event loop trình duyệt | Điều khiển qua DevTools protocol |

**Khi nào dùng E2E thay vì integration:**
- Khi cần kiểm chứng **luồng quan trọng nhất** (critical path) end-to-end: đăng ký, thanh toán, checkout - những thứ mà nếu hỏng là mất tiền.
- Khi cần kiểm thử tích hợp thật giữa frontend ↔ backend ↔ routing ↔ redirect, hành vi trình duyệt thật (cookie, localStorage, navigation).

Nhưng E2E **chậm và dễ flaky**, nên giữ số lượng ít (theo cả pyramid lẫn trophy). Phần lớn logic UI nên test bằng integration test RTL.

```ts
// Playwright
import { test, expect } from '@playwright/test';

test('mua hàng end-to-end', async ({ page }) => {
  await page.goto('/');
  await page.getByRole('link', { name: 'iPhone' }).click();
  await page.getByRole('button', { name: 'Thêm vào giỏ' }).click();
  await page.getByRole('link', { name: 'Thanh toán' }).click();
  await expect(page.getByText('Đặt hàng thành công')).toBeVisible();
});
```

**Bẫy:** E2E dùng selector dựa trên text/role (như RTL) sẽ bền hơn nhiều so với selector dựa vào class CSS hay XPath dài; tránh `cy.wait(2000)` cứng nhắc, hãy dùng auto-wait/assert.

</details>

---

### 11.8. Làm thế nào để test một custom hook trong React? Cho ví dụ với `renderHook`.

<details>
<summary>👉 Xem câu trả lời</summary>

Hook không thể gọi ngoài component (vi phạm Rules of Hooks). RTL cung cấp `renderHook` (từ `@testing-library/react`, trước đây tách ở `@testing-library/react-hooks`) để render hook trong một component ẩn và trả về `result.current` chứa giá trị hook trả ra. Mọi thay đổi state phải bọc trong `act` (hoặc dùng `userEvent`/`waitFor` đã tự bọc act).

```tsx
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

test('useCounter tăng giá trị', () => {
  const { result } = renderHook(() => useCounter(0));

  expect(result.current.count).toBe(0);

  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(1);
});
```

Với hook bất đồng bộ (vd hook fetch data), dùng `waitFor` để chờ và mock API qua MSW:

```tsx
test('useUser trả dữ liệu sau khi fetch', async () => {
  const { result } = renderHook(() => useUser(1));

  expect(result.current.isLoading).toBe(true);

  await waitFor(() => expect(result.current.isLoading).toBe(false));
  expect(result.current.data?.name).toBe('An');
});
```

Nếu hook cần Context/Provider (vd dùng React Query, Redux), truyền `wrapper`:

```tsx
const wrapper = ({ children }: PropsWithChildren) => (
  <QueryClientProvider client={new QueryClient()}>{children}</QueryClientProvider>
);
const { result } = renderHook(() => useTodos(), { wrapper });
```

**Bẫy thường gặp:**
- Quên bọc `act` khi gọi hàm làm thay đổi state → cảnh báo "not wrapped in act(...)".
- Lưu ý: nhiều khi tốt hơn là test hook **gián tiếp qua một component dùng nó** (đúng tinh thần RTL), chỉ dùng `renderHook` cho hook tái sử dụng / phức tạp.

</details>

---

### 11.9. Khi test một component có trạng thái bất đồng bộ (loading → success / error), bạn xử lý thế nào? Vì sao `findBy*` và `waitFor` quan trọng?

<details>
<summary>👉 Xem câu trả lời</summary>

Component async đi qua nhiều trạng thái theo thời gian: `loading` → `success` (hiển thị data) hoặc `error`. Test phải **chờ** state cập nhật vì cập nhật xảy ra sau khi promise resolve, qua nhiều vòng event loop. Không thể assert đồng bộ ngay sau render.

Hai công cụ chờ chính của RTL:
- **`findBy*`**: kết hợp `getBy*` + `waitFor`, trả về Promise, retry cho tới khi phần tử xuất hiện (hoặc timeout). Dùng để chờ một phần tử **xuất hiện**.
- **`waitFor`**: chờ tới khi một callback assertion không còn ném lỗi. Linh hoạt hơn, dùng cho điều kiện tùy ý (vd chờ phần tử **biến mất** qua `waitForElementToBeRemoved`).

```tsx
test('hiển thị 3 trạng thái: loading → data', async () => {
  server.use(
    http.get('/api/posts', () =>
      HttpResponse.json([{ id: 1, title: 'Bài 1' }])
    )
  );

  render(<PostList />);

  // 1. loading hiện ngay (đồng bộ)
  expect(screen.getByText(/đang tải/i)).toBeInTheDocument();

  // 2. chờ data xuất hiện bất đồng bộ
  expect(await screen.findByText('Bài 1')).toBeInTheDocument();

  // 3. loading đã biến mất
  expect(screen.queryByText(/đang tải/i)).not.toBeInTheDocument();
});

test('hiển thị lỗi khi API trả 500', async () => {
  server.use(
    http.get('/api/posts', () => new HttpResponse(null, { status: 500 }))
  );

  render(<PostList />);
  expect(await screen.findByText(/đã có lỗi xảy ra/i)).toBeInTheDocument();
});
```

**Bẫy thường gặp:**
- Quên `await` trước `findBy*` → test pass/fail sai lệch, hoặc warning act.
- Dùng `getBy*` cho phần tử async (chưa kịp render) → lỗi "không tìm thấy". Dùng `findBy*`.
- Dùng `queryBy*` để khẳng định **không tồn tại** (vd loading đã mất); `getBy*` sẽ ném lỗi thay vì trả `null`.
- Đặt nhiều assertion nặng trong callback `waitFor`; nên giữ callback gọn (một assertion) vì nó chạy lặp nhiều lần.

</details>

---

### 11.10. Snapshot testing là gì? Nêu ưu, nhược điểm và khi nào nên (hoặc không nên) dùng.

<details>
<summary>👉 Xem câu trả lời</summary>

**Snapshot test** chụp lại "ảnh" output được serialize của một component (cây DOM/JSX) vào file `.snap`, rồi ở các lần chạy sau so sánh output mới với snapshot đã lưu. Nếu khác, test fail và yêu cầu bạn xem lại rồi cập nhật (`--update` / `u`).

```tsx
import { render } from '@testing-library/react';

test('Button khớp snapshot', () => {
  const { container } = render(<Button variant="primary">Lưu</Button>);
  expect(container).toMatchSnapshot();
});
```

**Ưu điểm:**
- Viết rất nhanh, bắt được thay đổi UI ngoài ý muốn (regression).
- Tốt cho output ổn định, ít thay đổi: serialize cấu hình, error message, component thuần presentational nhỏ.

**Nhược điểm:**
- **Dễ bị "rubber-stamp"**: khi snapshot fail, lập trình viên có thói quen bấm update bừa mà không thực sự đọc diff → mất hết ý nghĩa.
- **Giòn (brittle)**: thay đổi nhỏ về markup/class làm vỡ snapshot dù hành vi không đổi → nhiễu.
- **Snapshot lớn khó review**: file `.snap` hàng trăm dòng, không ai đọc kỹ trong code review.
- Không diễn đạt **ý định** test (test nên nói rõ "tôi mong nút có chữ Lưu"), snapshot chỉ nói "giống lần trước".

**Khi nào dùng / tránh:**
- Nên dùng cho output nhỏ, ổn định, và dùng **inline snapshot** (`toMatchInlineSnapshot`) để diff nằm ngay trong test, dễ review.
- Tránh dùng cho component lớn/động hoặc thay cho assertion có chủ đích. Ưu tiên assertion rõ ràng kiểu `expect(screen.getByRole('button')).toHaveTextContent('Lưu')` vì nó thể hiện đúng kỳ vọng.

</details>

---

### 11.11. Bạn cấu hình môi trường test cho một dự án Next.js như thế nào (jsdom, setup file, mock những gì)? Những thứ đặc thù Next nào hay phải mock?

<details>
<summary>👉 Xem câu trả lời</summary>

Với Next.js, cách chính thống là dùng `next/jest` để Jest hiểu được cấu hình build của Next (SWC transform, path alias, biến môi trường, xử lý CSS module).

```js
// jest.config.js
const nextJest = require('next/jest');
const createJestConfig = nextJest({ dir: './' });

module.exports = createJestConfig({
  testEnvironment: 'jest-environment-jsdom',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
});
```

```ts
// jest.setup.ts
import '@testing-library/jest-dom'; // thêm matcher: toBeInTheDocument, toHaveTextContent...
```

**Những thứ đặc thù Next hay phải mock / lưu ý:**
- **Router**: component dùng `useRouter`. App Router dùng `next/navigation` (`useRouter`, `usePathname`, `useSearchParams`) - cần mock vì jsdom không có router thật.
- **`next/image`**: đôi khi cần mock để tránh xử lý tối ưu ảnh.
- **`next/link`**: thường chạy ổn trong jsdom, nhưng navigation thật thì không (đó là việc của E2E).
- **Server Components (App Router)**: component async server không render được trong jsdom theo cách thông thường - phần này thường để E2E (Playwright) hoặc test riêng logic. Client Component (`'use client'`) thì test bình thường bằng RTL.

```tsx
// mock useRouter của App Router
jest.mock('next/navigation', () => ({
  useRouter: () => ({ push: jest.fn(), replace: jest.fn() }),
  usePathname: () => '/blog',
  useSearchParams: () => new URLSearchParams(),
}));
```

**Bẫy:** đừng cố unit-test Server Component bằng RTL/jsdom (đặc biệt khi nó `async` và fetch data) - giới hạn của môi trường jsdom hiện tại; hãy test phần đó qua E2E hoặc tách logic data ra hàm thuần để unit test.

</details>

---

### 11.12. Hãy kể về một lần bạn xử lý test "flaky" (lúc pass lúc fail) hoặc thiết lập chiến lược test cho một dự án frontend. (Câu hỏi hành vi)

<details>
<summary>👉 Goi y tra loi (STAR)</summary>

Đây là câu hỏi hành vi - hãy trả lời theo cấu trúc **STAR** (Situation - Task - Action - Result). Dưới đây là một khung mẫu, bạn nên thay bằng trải nghiệm thật của mình.

**Situation (Tình huống):**
"Ở dự án React/Next.js trước, pipeline CI thường xuyên đỏ ngẫu nhiên: cùng một bộ test, chạy lại là pass. Khoảng 10-15% lần build E2E bị fail không rõ lý do, khiến cả team mất niềm tin vào CI và có thói quen bấm 're-run' thay vì đọc lỗi."

**Task (Nhiệm vụ):**
"Tôi nhận trách nhiệm điều tra và làm ổn định bộ test để CI đáng tin cậy trở lại, đồng thời thiết lập quy ước test cho team."

**Action (Hành động):**
- Tôi rà các test flaky và phát hiện ba nguyên nhân chính: (1) dùng `setTimeout`/`cy.wait(2000)` cứng thay vì chờ theo điều kiện; (2) thiếu `await` trước các `findBy*` trong RTL nên có race condition; (3) MSW handler bị 'rò rỉ' giữa các test do quên `resetHandlers()`.
- Tôi thay các chờ cứng bằng auto-wait của Playwright và `findBy*`/`waitFor` của RTL; thêm `afterEach(() => server.resetHandlers())`; bật ESLint rule `testing-library/await-async-queries` để bắt thiếu `await` ngay khi viết.
- Tôi soạn một tài liệu quy ước ngắn: ưu tiên `getByRole`, không test implementation detail, dùng MSW cho mọi mock API, và áp dụng testing trophy (nhiều integration, ít E2E chỉ cho critical path).

**Result (Kết quả):**
"Tỉ lệ flaky của E2E giảm từ ~15% xuống dưới 1%; thời gian một vòng CI giảm vì bỏ được các `wait` cứng; team lấy lại niềm tin vào CI và bắt đầu chặn merge khi test đỏ. Quan trọng hơn, quy ước test giúp người mới viết test đúng hướng ngay từ đầu."

**Mẹo trả lời:** nhấn mạnh tư duy chẩn đoán nguyên nhân gốc (không chỉ 're-run'), việc bạn dùng công cụ đúng (auto-wait thay vì sleep cứng), và tác động đo lường được (con số % flaky, thời gian CI).

</details>
## 12. Bài tập Coding (Live Coding)

### 12.1. Viết hook `useDebounce` để trì hoãn một giá trị (ví dụ: ô tìm kiếm). Giải thích vì sao cần `debounce` và sự khác biệt với `throttle`.

<details>
<summary>👉 Xem câu trả lời</summary>

`debounce` chỉ cập nhật giá trị sau khi người dùng *ngừng* thay đổi nó trong một khoảng thời gian (`delay`). Rất hợp với ô tìm kiếm: ta không muốn gọi API sau mỗi ký tự, mà chỉ gọi khi người dùng ngừng gõ. Điểm mấu chốt là phải `clearTimeout` ở hàm cleanup của `useEffect` để mỗi lần `value` đổi thì timer cũ bị huỷ.

Khác biệt với `throttle`: `debounce` "dồn" các lần gọi liên tiếp thành một lần ở *cuối*; `throttle` đảm bảo hàm chạy *tối đa một lần mỗi khoảng thời gian* (phù hợp với `scroll`, `resize`).

```tsx
import { useEffect, useState } from "react";

function useDebounce<T>(value: T, delay = 500): T {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    // Mỗi lần value/delay đổi, huỷ timer cũ -> chỉ lần cuối "sống sót"
    return () => clearTimeout(id);
  }, [value, delay]);

  return debounced;
}

// Cách dùng
function Search() {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebounce(query, 400);

  useEffect(() => {
    if (!debouncedQuery) return;
    fetch(`/api/search?q=${debouncedQuery}`); // chỉ chạy khi ngừng gõ 400ms
  }, [debouncedQuery]);

  return <input value={query} onChange={(e) => setQuery(e.target.value)} />;
}
```

Bẫy thường gặp: quên `clearTimeout` (mọi timer đều chạy, mất tác dụng debounce); để `delay` vào `useState` thay vì `useEffect`; gọi API trực tiếp trong `onChange` thay vì lắng nghe giá trị đã debounce.

</details>

---

### 12.2. Viết hàm `deepClone(obj)` để sao chép sâu một object **không dùng thư viện**. Vì sao `structuredClone` hay `JSON.parse(JSON.stringify(...))` chưa đủ trong mọi trường hợp?

<details>
<summary>👉 Xem câu trả lời</summary>

Sao chép sâu (deep clone) tạo bản sao hoàn toàn độc lập, sửa bản sao không ảnh hưởng bản gốc. Cần xử lý: object/array lồng nhau, `Date`, `Map`, `Set`, và đặc biệt là **vòng tham chiếu (circular reference)** — dùng một `WeakMap` để nhớ các object đã clone.

```ts
function deepClone<T>(value: T, seen = new WeakMap()): T {
  // Kiểu nguyên thuỷ (primitive) hoặc null -> trả về luôn
  if (value === null || typeof value !== "object") return value;

  // Xử lý vòng tham chiếu
  if (seen.has(value as object)) return seen.get(value as object);

  if (value instanceof Date) return new Date(value.getTime()) as T;
  if (value instanceof RegExp) return new RegExp(value.source, value.flags) as T;

  if (value instanceof Map) {
    const result = new Map();
    seen.set(value as object, result);
    value.forEach((v, k) => result.set(deepClone(k, seen), deepClone(v, seen)));
    return result as T;
  }

  if (value instanceof Set) {
    const result = new Set();
    seen.set(value as object, result);
    value.forEach((v) => result.add(deepClone(v, seen)));
    return result as T;
  }

  const result: any = Array.isArray(value) ? [] : {};
  seen.set(value as object, result);
  for (const key of Object.keys(value as object)) {
    result[key] = deepClone((value as any)[key], seen);
  }
  return result;
}
```

So sánh các cách:

| Cách | Ưu điểm | Hạn chế |
|------|---------|---------|
| `JSON.parse(JSON.stringify(x))` | Ngắn gọn | Mất `Date` (-> string), `undefined`, `function`, `Map`/`Set`; lỗi với circular reference |
| `structuredClone(x)` | Hỗ trợ `Date`/`Map`/`Set`/circular | Không clone được `function`, không sao chép prototype tuỳ biến; tương đối mới |
| Tự viết đệ quy | Kiểm soát hoàn toàn | Phải tự xử lý nhiều kiểu, dễ thiếu sót |

Trong React, deep clone hay cần khi update state lồng sâu mà muốn giữ tính bất biến (immutability) — nhưng thực tế nên ưu tiên cập nhật bất biến từng tầng hoặc dùng Immer.

</details>

---

### 12.3. Viết hàm `flatten` làm phẳng mảng lồng nhau ở độ sâu bất kỳ, và hàm `groupBy(array, keyFn)` gom nhóm phần tử.

<details>
<summary>👉 Xem câu trả lời</summary>

`flatten` có thể giải bằng đệ quy hoặc `reduce`. Bản gốc đã có `Array.prototype.flat(Infinity)`, nhưng phỏng vấn thường yêu cầu tự viết. `groupBy` trả về một object map từ key -> mảng phần tử (tương tự `Object.groupBy` mới).

```ts
// flatten: làm phẳng mọi độ sâu
function flatten<T>(arr: any[]): T[] {
  return arr.reduce<T[]>((acc, item) => {
    return acc.concat(Array.isArray(item) ? flatten<T>(item) : item);
  }, []);
}

flatten([1, [2, [3, [4]], 5]]); // [1, 2, 3, 4, 5]

// flatten có kiểm soát độ sâu
function flattenDepth(arr: any[], depth = 1): any[] {
  if (depth < 1) return arr.slice();
  return arr.reduce((acc, item) => {
    return acc.concat(Array.isArray(item) ? flattenDepth(item, depth - 1) : item);
  }, []);
}

// groupBy
function groupBy<T, K extends string | number>(
  arr: T[],
  keyFn: (item: T) => K
): Record<K, T[]> {
  return arr.reduce((acc, item) => {
    const key = keyFn(item);
    (acc[key] ||= []).push(item);
    return acc;
  }, {} as Record<K, T[]>);
}

const users = [
  { name: "An", role: "admin" },
  { name: "Bình", role: "user" },
  { name: "Cường", role: "admin" },
];
groupBy(users, (u) => u.role);
// { admin: [An, Cường], user: [Bình] }
```

Bẫy thường gặp: với `flatten` đệ quy trên mảng cực sâu có thể tràn stack — khi đó nên dùng vòng lặp với stack thủ công. Với `groupBy`, nhớ `keyFn` trả về primitive (string/number) vì object không làm key của plain object được.

</details>

---

### 12.4. Viết component `Dropdown` hỗ trợ **tìm kiếm (search)** và **chọn nhiều (multi-select)**. Cần lưu ý gì về accessibility và đóng menu khi click ra ngoài?

<details>
<summary>👉 Xem câu trả lời</summary>

Một `Dropdown` multi-select + search cần: state mở/đóng, ô input lọc options, danh sách chọn (mảng `selected`), toggle từng item, và xử lý click ra ngoài để đóng. Đây là bài kinh điển kiểm tra khả năng quản lý state cục bộ + DOM events.

```tsx
import { useState, useRef, useEffect, useMemo } from "react";

type Option = { value: string; label: string };

function MultiSelectDropdown({ options }: { options: Option[] }) {
  const [open, setOpen] = useState(false);
  const [search, setSearch] = useState("");
  const [selected, setSelected] = useState<string[]>([]);
  const ref = useRef<HTMLDivElement>(null);

  // Đóng khi click ra ngoài
  useEffect(() => {
    function handler(e: MouseEvent) {
      if (ref.current && !ref.current.contains(e.target as Node)) {
        setOpen(false);
      }
    }
    document.addEventListener("mousedown", handler);
    return () => document.removeEventListener("mousedown", handler);
  }, []);

  const filtered = useMemo(
    () => options.filter((o) => o.label.toLowerCase().includes(search.toLowerCase())),
    [options, search]
  );

  const toggle = (value: string) =>
    setSelected((prev) =>
      prev.includes(value) ? prev.filter((v) => v !== value) : [...prev, value]
    );

  return (
    <div ref={ref} style={{ position: "relative" }}>
      <button onClick={() => setOpen((o) => !o)} aria-haspopup="listbox" aria-expanded={open}>
        {selected.length ? `Đã chọn ${selected.length}` : "Chọn..."}
      </button>

      {open && (
        <div role="listbox" aria-multiselectable="true">
          <input
            autoFocus
            value={search}
            onChange={(e) => setSearch(e.target.value)}
            placeholder="Tìm kiếm..."
          />
          {filtered.map((o) => (
            <label key={o.value} role="option" aria-selected={selected.includes(o.value)}>
              <input
                type="checkbox"
                checked={selected.includes(o.value)}
                onChange={() => toggle(o.value)}
              />
              {o.label}
            </label>
          ))}
          {filtered.length === 0 && <div>Không có kết quả</div>}
        </div>
      )}
    </div>
  );
}
```

Lưu ý: dùng `mousedown` (không phải `click`) để bắt click-outside chính xác hơn; thêm `role`/`aria-*` cho accessibility; điều hướng bàn phím (mũi tên + Enter) là điểm cộng nếu kịp. Để dùng lại tốt hơn nên biến thành controlled component qua props `value`/`onChange`.

</details>

---

### 12.5. Viết hook `useFetch(url)` trả về `{ data, loading, error }` và **huỷ request khi unmount hoặc url đổi** bằng `AbortController`.

<details>
<summary>👉 Xem câu trả lời</summary>

Một `useFetch` tốt phải: reset trạng thái khi `url` đổi, xử lý lỗi, và quan trọng nhất là **huỷ request cũ** để tránh race condition (response cũ về sau lại ghi đè data mới) và cảnh báo setState sau unmount.

```tsx
import { useState, useEffect } from "react";

function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    const controller = new AbortController();
    setLoading(true);
    setError(null);

    fetch(url, { signal: controller.signal })
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then((json: T) => setData(json))
      .catch((err) => {
        // Bỏ qua lỗi do chính ta abort
        if (err.name !== "AbortError") setError(err);
      })
      .finally(() => {
        // Tránh setLoading nếu đã abort (request bị thay thế)
        if (!controller.signal.aborted) setLoading(false);
      });

    return () => controller.abort(); // huỷ khi url đổi / unmount
  }, [url]);

  return { data, loading, error };
}
```

Bẫy thường gặp: không phân biệt `AbortError` với lỗi thật -> hiển thị lỗi sai; quên reset `error`/`loading` khi `url` đổi; race condition khi không abort. Trong dự án thật nên dùng **TanStack Query** thay vì tự viết, vì nó lo sẵn cache, dedupe, retry, stale-while-revalidate — `useFetch` chủ yếu để chứng minh hiểu bản chất.

</details>

---

### 12.6. Viết hai hook tiện ích nhỏ: `usePrevious(value)` lấy giá trị trước đó, và `useLocalStorage(key, initial)` đồng bộ state với `localStorage`.

<details>
<summary>👉 Xem câu trả lời</summary>

`usePrevious` dùng `useRef` để lưu giá trị qua các lần render — ref không gây re-render và được cập nhật *sau* khi render xong (trong `useEffect`), nên lúc render hiện tại nó vẫn giữ giá trị cũ.

`useLocalStorage` đọc giá trị khởi tạo lười (lazy initializer) để không đọc localStorage mỗi lần render, và ghi lại khi value đổi.

```tsx
import { useEffect, useRef, useState } from "react";

function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();
  useEffect(() => {
    ref.current = value; // chạy sau render -> ref giữ giá trị "lần trước"
  }, [value]);
  return ref.current;
}

function useLocalStorage<T>(key: string, initial: T) {
  const [value, setValue] = useState<T>(() => {
    try {
      const raw = localStorage.getItem(key);
      return raw ? (JSON.parse(raw) as T) : initial;
    } catch {
      return initial;
    }
  });

  useEffect(() => {
    try {
      localStorage.setItem(key, JSON.stringify(value));
    } catch {
      /* quota đầy hoặc storage bị chặn */
    }
  }, [key, value]);

  return [value, setValue] as const;
}
```

Lưu ý với Next.js (SSR): `localStorage` không tồn tại trên server -> lazy initializer chạy trên server sẽ lỗi. Cần khởi tạo bằng `initial` rồi đọc localStorage trong `useEffect`, hoặc chỉ render component đó ở client để tránh lỗi **hydration mismatch**. `usePrevious` ở lần render đầu trả về `undefined` — cần xử lý trường hợp này.

</details>

---

### 12.7. Triển khai hàm `throttle(fn, wait)`. Khác `debounce` ở chỗ nào và khi nào nên dùng?

<details>
<summary>👉 Xem câu trả lời</summary>

`throttle` đảm bảo `fn` chạy **tối đa một lần trong mỗi khoảng `wait`**, bất kể được gọi bao nhiêu lần. Dùng cho sự kiện bắn liên tục như `scroll`, `mousemove`, `resize` — nơi ta muốn cập nhật đều đặn nhưng không quá dày.

```ts
function throttle<T extends (...args: any[]) => void>(fn: T, wait: number) {
  let lastTime = 0;
  let timer: ReturnType<typeof setTimeout> | null = null;

  return function (this: any, ...args: Parameters<T>) {
    const now = Date.now();
    const remaining = wait - (now - lastTime);

    if (remaining <= 0) {
      // Đã đủ thời gian -> chạy ngay (leading edge)
      if (timer) { clearTimeout(timer); timer = null; }
      lastTime = now;
      fn.apply(this, args);
    } else if (!timer) {
      // Lên lịch cho lần cuối (trailing edge)
      timer = setTimeout(() => {
        lastTime = Date.now();
        timer = null;
        fn.apply(this, args);
      }, remaining);
    }
  };
}

// Cách dùng
const onScroll = throttle(() => console.log(window.scrollY), 200);
window.addEventListener("scroll", onScroll);
```

So sánh nhanh:

| | `debounce` | `throttle` |
|---|---|---|
| Khi nào chạy | Sau khi *ngừng* gọi `delay`ms | Đều đặn mỗi `wait`ms |
| Tình huống | Ô search, validate input | Scroll, resize, mousemove, kéo thả |
| Số lần chạy khi gọi liên tục | 1 lần (cuối) | Nhiều lần, giãn đều |

Trong React, khi tạo hàm throttle/debounce phải bọc trong `useMemo`/`useRef` để **không tạo lại mỗi render** (nếu không, mỗi render lại reset closure và mất tác dụng).

</details>

---

### 12.8. Triển khai hàm `memoize(fn)` để cache kết quả theo tham số. Liên hệ với `useMemo`/`React.memo`.

<details>
<summary>👉 Xem câu trả lời</summary>

`memoize` lưu kết quả của `fn` theo tham số: nếu gọi lại với cùng tham số thì trả cache thay vì tính lại. Bản đơn giản dùng tham số đầu (hoặc `JSON.stringify(args)`) làm key.

```ts
function memoize<Args extends any[], R>(fn: (...args: Args) => R) {
  const cache = new Map<string, R>();
  return (...args: Args): R => {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key)!;
    const result = fn(...args);
    cache.set(key, result);
    return result;
  };
}

const slowSquare = (n: number) => {
  for (let i = 0; i < 1e7; i++) {} // giả lập tính nặng
  return n * n;
};
const fastSquare = memoize(slowSquare);
fastSquare(5); // tính
fastSquare(5); // lấy từ cache, tức thì
```

Liên hệ React:

| Công cụ | Cache gì | Cơ chế so sánh |
|---|---|---|
| `memoize` (tự viết) | Kết quả hàm theo tham số | Key tuỳ ta định nghĩa |
| `useMemo(fn, deps)` | Giá trị tính toán trong render | So sánh `deps` (`Object.is`) |
| `React.memo(Component)` | Bỏ qua re-render component | So nông props |
| `useCallback(fn, deps)` | Tham chiếu hàm | So sánh `deps` |

Bẫy: `JSON.stringify` không xử lý được `function`/`undefined`/thứ tự key object, và tốn chi phí; với object làm tham số nên cân nhắc `WeakMap`. Cache không giới hạn có thể rò rỉ bộ nhớ -> với production cần LRU. Trong React, `useMemo` chỉ cache *một* giá trị (deps đổi là mất), khác với `memoize` cache nhiều key.

</details>

---

### 12.9. Triển khai hàm `curry(fn)` cho phép gọi từng phần tham số. Currying hữu ích thế nào trong code React?

<details>
<summary>👉 Xem câu trả lời</summary>

Currying biến hàm nhiều tham số `f(a, b, c)` thành chuỗi hàm có thể gọi từng phần: `f(a)(b)(c)` hoặc `f(a, b)(c)`. Bản generic dựa vào `fn.length` (số tham số khai báo) để biết khi nào đủ.

```ts
function curry(fn: (...args: any[]) => any) {
  return function curried(this: any, ...args: any[]) {
    if (args.length >= fn.length) {
      return fn.apply(this, args); // đủ tham số -> gọi thật
    }
    // Chưa đủ -> trả hàm nhận tiếp phần còn lại
    return (...next: any[]) => curried.apply(this, [...args, ...next]);
  };
}

const sum = (a: number, b: number, c: number) => a + b + c;
const curried = curry(sum);
curried(1)(2)(3); // 6
curried(1, 2)(3); // 6
curried(1)(2, 3); // 6
```

Trong React, currying hay dùng tạo handler theo tham số mà vẫn gọn:

```tsx
// thay vì onClick={() => handleSelect(id)} lặp lại
const handleSelect = (id: string) => (e: React.MouseEvent) => {
  console.log("chọn", id);
};
// dùng
items.map((it) => <button key={it.id} onClick={handleSelect(it.id)}>{it.name}</button>);
```

Bẫy: `fn.length` không đếm tham số có giá trị mặc định hoặc rest param -> curry generic sẽ sai với các hàm đó. Trong vòng lặp render, dạng `handleSelect(id)` tạo hàm mới mỗi render — chấp nhận được trừ khi truyền xuống component đã `memo` (khi đó phá vỡ tối ưu).

</details>

---

### 12.10. Viết hook `useInfiniteScroll` dùng `IntersectionObserver` để tải thêm dữ liệu khi cuộn tới cuối danh sách.

<details>
<summary>👉 Xem câu trả lời</summary>

`IntersectionObserver` báo khi một phần tử "sentinel" (chốt) lọt vào viewport. Đặt sentinel ở cuối danh sách: khi nó hiện ra -> gọi `loadMore`. Ưu điểm so với lắng nghe `scroll`: không cần tính toán vị trí thủ công, hiệu năng tốt hơn (chạy ngoài luồng chính), tự throttle.

Mẹo quan trọng: dùng **callback ref** thay vì `useRef` thuần để observer gắn lại đúng lúc node xuất hiện/đổi.

```tsx
import { useCallback, useRef } from "react";

function useInfiniteScroll(
  loadMore: () => void,
  { hasMore, loading }: { hasMore: boolean; loading: boolean }
) {
  const observer = useRef<IntersectionObserver | null>(null);

  const sentinelRef = useCallback(
    (node: HTMLElement | null) => {
      if (loading) return;             // đang tải thì khoan observe
      observer.current?.disconnect();  // ngắt observer cũ
      if (!node) return;

      observer.current = new IntersectionObserver((entries) => {
        if (entries[0].isIntersecting && hasMore) {
          loadMore();
        }
      });
      observer.current.observe(node);
    },
    [loading, hasMore, loadMore]
  );

  return sentinelRef;
}

// Cách dùng
function List({ items, fetchNext, hasMore, loading }: any) {
  const sentinelRef = useInfiniteScroll(fetchNext, { hasMore, loading });
  return (
    <div>
      {items.map((it: any) => <div key={it.id}>{it.name}</div>)}
      {hasMore && <div ref={sentinelRef}>Đang tải...</div>}
    </div>
  );
}
```

Bẫy: quên `disconnect` observer cũ -> rò rỉ và gọi `loadMore` nhiều lần; không chặn khi `loading` -> gọi trùng; truyền `loadMore` không ổn định khiến observer tạo lại liên tục (nên bọc `useCallback` ở phía cha). Với Next.js/TanStack Query, kết hợp `useInfiniteQuery` để lo phần data, hook này chỉ lo phần kích hoạt.

</details>

---

### 12.11. Xây dựng component `Tabs` tái sử dụng (controlled hoặc uncontrolled). Làm sao để API gọn và dễ mở rộng?

<details>
<summary>👉 Xem câu trả lời</summary>

Mẫu tốt là **compound components** kết hợp Context: `<Tabs>` giữ state tab đang chọn và chia sẻ qua Context, các con `<Tab>` / `<TabPanel>` tự đọc Context. Cách này cho API khai báo tự nhiên, dễ tuỳ biến bố cục, không phải truyền props lằng nhằng.

```tsx
import { createContext, useContext, useState, ReactNode } from "react";

type TabsCtx = { active: string; setActive: (v: string) => void };
const TabsContext = createContext<TabsCtx | null>(null);

function useTabs() {
  const ctx = useContext(TabsContext);
  if (!ctx) throw new Error("Tab phải nằm trong <Tabs>");
  return ctx;
}

function Tabs({ defaultValue, children }: { defaultValue: string; children: ReactNode }) {
  const [active, setActive] = useState(defaultValue);
  return (
    <TabsContext.Provider value={{ active, setActive }}>
      <div>{children}</div>
    </TabsContext.Provider>
  );
}

function Tab({ value, children }: { value: string; children: ReactNode }) {
  const { active, setActive } = useTabs();
  const selected = active === value;
  return (
    <button role="tab" aria-selected={selected} onClick={() => setActive(value)}>
      {children}
    </button>
  );
}

function TabPanel({ value, children }: { value: string; children: ReactNode }) {
  const { active } = useTabs();
  if (active !== value) return null;
  return <div role="tabpanel">{children}</div>;
}

Tabs.Tab = Tab;
Tabs.Panel = TabPanel;

// Cách dùng
<Tabs defaultValue="a">
  <div role="tablist">
    <Tabs.Tab value="a">Tab A</Tabs.Tab>
    <Tabs.Tab value="b">Tab B</Tabs.Tab>
  </div>
  <Tabs.Panel value="a">Nội dung A</Tabs.Panel>
  <Tabs.Panel value="b">Nội dung B</Tabs.Panel>
</Tabs>
```

Mở rộng: hỗ trợ controlled bằng cách nhận thêm props `value`/`onChange` (nếu có thì dùng, không thì tự quản uncontrolled). Accessibility: `role="tablist"/"tab"/"tabpanel"`, `aria-selected`, điều hướng mũi tên. Trong production có thể dùng Radix UI / Headless UI để có sẵn a11y + bàn phím.

</details>

---

### 12.12. Xây dựng component `Accordion` cho phép cấu hình mở một mục (single) hoặc nhiều mục (multiple). Cần chú ý gì về animation và accessibility?

<details>
<summary>👉 Xem câu trả lời</summary>

`Accordion` quản lý danh sách item đóng/mở. Tham số `type` quyết định "single" (mở 1, click cái khác thì đóng cái cũ) hay "multiple" (mở nhiều cùng lúc — dùng mảng/Set các id đang mở).

```tsx
import { useState, ReactNode } from "react";

type AccordionProps = {
  type?: "single" | "multiple";
  items: { id: string; title: string; content: ReactNode }[];
};

function Accordion({ type = "single", items }: AccordionProps) {
  const [openIds, setOpenIds] = useState<string[]>([]);

  const toggle = (id: string) => {
    setOpenIds((prev) => {
      const isOpen = prev.includes(id);
      if (type === "single") return isOpen ? [] : [id];
      return isOpen ? prev.filter((x) => x !== id) : [...prev, id];
    });
  };

  return (
    <div>
      {items.map((item) => {
        const expanded = openIds.includes(item.id);
        return (
          <div key={item.id}>
            <button
              aria-expanded={expanded}
              aria-controls={`panel-${item.id}`}
              onClick={() => toggle(item.id)}
            >
              {item.title}
            </button>
            <div
              id={`panel-${item.id}`}
              role="region"
              hidden={!expanded}
            >
              {item.content}
            </div>
          </div>
        );
      })}
    </div>
  );
}
```

Về animation: `height: auto` không thể transition bằng CSS thuần. Cách phổ biến là dùng `grid-template-rows: 0fr -> 1fr`, hoặc `max-height`, hoặc đo chiều cao bằng JS; mượt nhất thì dùng **Framer Motion** (`AnimatePresence` + `animate={{ height }}`).

Accessibility: nút dùng `aria-expanded` và `aria-controls` trỏ tới panel; panel nên có `role="region"`. Dùng `hidden` (không phải chỉ ẩn bằng CSS) để screen reader bỏ qua nội dung đóng. Lưu ý đừng unmount nội dung nếu cần animation đóng (sẽ mất transition) — khi đó dùng `AnimatePresence` của Framer Motion.

</details>
## 13. Tư duy và Giải quyết vấn đề

### 13.1. Bạn tiếp cận một bug khó tái hiện (heisenbug) như thế nào, ví dụ "đôi khi danh sách bị render trùng item"?

<details>
<summary>👉 Xem câu trả lời</summary>

Bug khó tái hiện thường đến từ trạng thái phụ thuộc thời gian (race condition), dữ liệu không xác định (thứ tự response từ server), hoặc state rò rỉ giữa các lần render. Quy trình tôi hay dùng:

1. **Thu hẹp điều kiện tái hiện**: ghi lại môi trường (trình duyệt, network chậm/nhanh, dữ liệu cụ thể). Bật Network throttling để ép race condition lộ ra.
2. **Tăng tính quan sát (observability)**: thêm log có timestamp, hoặc dùng React DevTools Profiler để xem component nào re-render bất thường.
3. **Cô lập biến**: tạo một bản tái hiện tối thiểu (minimal reproduction) loại bỏ dần các phần không liên quan.
4. **Hình thành giả thuyết rồi kiểm chứng từng cái một**, không sửa lung tung.

Một nguyên nhân kinh điển trong React là **dùng index làm `key`** khiến danh sách bị render sai khi sắp xếp/thêm/xóa; hoặc **race condition trong fetch** khi response cũ về sau response mới.

```tsx
// BUG: response cũ có thể ghi đè response mới -> hiển thị dữ liệu sai/trùng
useEffect(() => {
  fetch(`/api/items?q=${query}`)
    .then((r) => r.json())
    .then((data) => setItems(data)); // không hủy request cũ
}, [query]);

// FIX: dùng cờ huỷ (cleanup) hoặc AbortController
useEffect(() => {
  const controller = new AbortController();
  fetch(`/api/items?q=${query}`, { signal: controller.signal })
    .then((r) => r.json())
    .then((data) => setItems(data))
    .catch((e) => {
      if (e.name !== 'AbortError') throw e;
    });
  return () => controller.abort(); // huỷ request cũ khi query đổi
}, [query]);
```

**Bẫy thường gặp**: thêm `console.log` ở chỗ sai rồi kết luận vội; sửa triệu chứng (ép re-render) thay vì sửa nguyên nhân gốc; quên rằng React Strict Mode ở dev chạy effect 2 lần, làm tưởng có bug trong khi production thì không (hoặc ngược lại, che giấu bug cleanup).

</details>

---

### 13.2. Khi đứng trước một quyết định kỹ thuật (ví dụ chọn cách quản lý state), bạn cân nhắc trade-off ra sao và làm sao để quyết định không bị "cảm tính"?

<details>
<summary>👉 Xem câu trả lời</summary>

Tôi cố gắng biến quyết định thành một bài toán có tiêu chí rõ ràng thay vì theo sở thích cá nhân. Khung suy nghĩ:

1. **Xác định bài toán thật sự là gì** (ví dụ: "đồng bộ server state" khác với "quản lý UI state nội bộ"). Rất nhiều tranh cãi sai vì nhầm hai loại state này.
2. **Liệt kê các phương án khả thi** và **tiêu chí đánh giá**: độ phức tạp, đường cong học, hiệu năng, khả năng test, bảo trì lâu dài, kích thước bundle, mức độ phù hợp với team.
3. **Đánh giá theo từng tiêu chí**, ghi rõ trade-off thay vì chỉ kết luận.
4. **Cân nhắc chi phí đảo ngược (reversibility)**: quyết định dễ đảo thì làm nhanh, quyết định khó đảo (kiến trúc, chọn framework) thì cân nhắc kỹ.

Ví dụ so sánh khi chọn cách quản lý state:

| Nhu cầu | Lựa chọn phù hợp | Lý do |
|---|---|---|
| Dữ liệu từ server (cache, refetch, loading) | TanStack Query | Sinh ra để xử lý server state, có cache/invalidation sẵn |
| State UI cục bộ một component | `useState` / `useReducer` | Đơn giản nhất, không thêm phụ thuộc |
| State global nhẹ, ít boilerplate | Zustand | API gọn, không cần Provider lồng nhau |
| State global phức tạp, cần devtools/middleware/team lớn | Redux Toolkit | Chuẩn hóa, dễ trace, có hệ sinh thái lớn |

**Bẫy thường gặp**: nhét toàn bộ server state vào Redux rồi tự viết lại logic cache/loading (trong khi TanStack Query làm sẵn); chọn công nghệ vì "hot" chứ không vì bài toán; over-engineer cho team 2 người như thể team 50 người.

</details>

---

### 13.3. Bạn được giao maintain một codebase React lớn mà chưa từng đụng tới. Bạn làm gì trong vài ngày đầu để hiểu nó?

<details>
<summary>👉 Xem câu trả lời</summary>

Tôi đọc codebase theo hướng "từ ngoài vào trong" và "theo luồng dữ liệu" thay vì đọc tuần tự từng file:

1. **Chạy được app trước đã**: đọc `README`, `package.json` (scripts, dependencies để đoán hệ công nghệ: Next.js? React Router? Redux hay Zustand?).
2. **Tìm điểm vào (entry point)**: với Next.js là cấu trúc `app/` hoặc `pages/`; với SPA là `main.tsx`/`index.tsx` rồi lần theo router để biết các trang chính.
3. **Đi theo một luồng người dùng cụ thể** (ví dụ login): UI -> gọi API -> cập nhật state -> render. Đây là cách nhanh nhất để hiểu kiến trúc thật.
4. **Đọc cấu trúc thư mục và quy ước đặt tên** để hiểu ranh giới module.
5. **Dùng công cụ thay vì đọc thủ công**: `git log`/`git blame` để biết phần nào hay đổi (thường là phần quan trọng/dễ vỡ), tìm theo từ khóa, dựa vào type của TypeScript để lần theo.
6. **Đọc test** (nếu có): test mô tả hành vi mong đợi tốt hơn cả comment.

```tsx
// Mẹo: với Next.js App Router, nhìn cấu trúc thư mục là hiểu ngay routing + layout
// app/
//   layout.tsx            -> layout gốc (provider, theme)
//   (marketing)/page.tsx  -> route group, không ảnh hưởng URL
//   dashboard/
//     layout.tsx          -> layout lồng cho dashboard
//     page.tsx            -> /dashboard
//     [id]/page.tsx       -> /dashboard/:id (dynamic segment)
```

**Bẫy thường gặp**: cố đọc hết mọi file ngay từ đầu (quá tải và vô ích); refactor sớm khi chưa hiểu vì sao code được viết như vậy (Chesterton's Fence — đừng phá hàng rào khi chưa biết vì sao nó ở đó); bỏ qua việc hỏi đồng đội — một câu hỏi đúng người tiết kiệm cả ngày đọc code.

</details>

---

### 13.4. Khi nào KHÔNG nên tối ưu sớm (premature optimization)? Cho một ví dụ cụ thể trong React.

<details>
<summary>👉 Xem câu trả lời</summary>

Nguyên tắc: **đo trước, tối ưu sau**. Tối ưu sớm khi chưa có bằng chứng về vấn đề hiệu năng thường làm code phức tạp hơn, khó đọc hơn mà không mang lại lợi ích thực tế, thậm chí gây bug. Chỉ tối ưu khi: (1) có số đo cụ thể (Profiler, Lighthouse, FPS thấp), và (2) phần đó thực sự nằm trên đường tới hạn (hot path).

Ví dụ kinh điển trong React là rải `useMemo`/`useCallback`/`React.memo` khắp nơi "cho chắc":

```tsx
// TỐI ƯU SỚM KHÔNG CẦN THIẾT: phép tính rẻ, bọc useMemo còn tốn hơn
const fullName = useMemo(() => `${first} ${last}`, [first, last]);
// Chỉ là nối chuỗi — chi phí so sánh dependencies + giữ cache còn lớn hơn việc tính lại.

// useCallback vô nghĩa nếu hàm không được truyền xuống component đã memo hoá
const handleClick = useCallback(() => setOpen(true), []); // truyền cho <button> thường -> thừa
```

Khi nào tối ưu này MỚI hợp lý: hàm/giá trị được truyền xuống một component con bọc `React.memo`, hoặc dùng làm dependency của một effect/hook khác, hoặc phép tính thật sự nặng (lọc/sắp xếp danh sách lớn). Lưu ý React Compiler (React 19) tự động hóa phần lớn việc memo hóa này, càng làm việc memo thủ công bừa bãi trở nên dư thừa.

**Bẫy thường gặp**: nhầm "code trông tối ưu" với "code chạy nhanh hơn"; bỏ công sức tối ưu một phần chạy 1 lần/phút trong khi bỏ qua render list 10.000 dòng không virtualize; tối ưu làm mất khả năng đọc khiến đồng đội gây bug về sau. Câu nói gốc của Knuth: "premature optimization is the root of all evil" — nhưng phần thường bị quên là "in 97% of cases".

</details>

---

### 13.5. Bạn dựa vào những tiêu chí nào để chọn một thư viện cho dự án (ví dụ chọn thư viện form, hoặc thư viện UI)?

<details>
<summary>👉 Xem câu trả lời</summary>

Một thư viện là một khoản nợ dài hạn, nên tôi đánh giá nó như một quyết định kiến trúc, không chỉ nhìn số sao GitHub. Các tiêu chí:

1. **Phù hợp bài toán & API**: nó giải đúng vấn đề mình có không, API có dễ dùng không.
2. **Sức khỏe dự án (maintenance)**: tần suất commit, issue được trả lời, lịch sử release, có bị bỏ rơi không.
3. **Cộng đồng & tài liệu**: nhiều người dùng nghĩa là dễ tìm giải pháp khi gặp lỗi; tài liệu tốt giảm chi phí học.
4. **Kích thước bundle & tác động hiệu năng**: tra trên bundlephobia, có tree-shakeable không — đặc biệt quan trọng với client bundle.
5. **Tính tương thích**: hỗ trợ TypeScript tốt? Chạy được với Server Components / SSR của Next.js không (nhiều thư viện cũ thiếu `'use client'` hoặc đụng `window` khi SSR)?
6. **Chi phí rời bỏ (lock-in)**: nếu sau này muốn thay thì khó tới mức nào.

Ví dụ thực tế khi chọn thư viện form, tôi thường nghiêng về **React Hook Form** vì dùng uncontrolled input nên ít re-render, tích hợp tốt với **Zod** qua resolver để validate type-safe:

```tsx
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';

const schema = z.object({ email: z.string().email(), age: z.coerce.number().min(18) });
type FormValues = z.infer<typeof schema>; // suy ra type từ schema -> một nguồn sự thật

const { register, handleSubmit, formState } = useForm<FormValues>({
  resolver: zodResolver(schema),
});
```

**Bẫy thường gặp**: chọn theo độ "trend" mà không kiểm tra còn được maintain không; kéo về một thư viện nặng chỉ để dùng 1 hàm nhỏ (có thể tự viết); bỏ qua việc kiểm tra tương thích SSR/RSC rồi vỡ khi build Next.js.

</details>

---

### 13.6. PM/designer đưa cho bạn một yêu cầu mơ hồ kiểu "làm cái trang dashboard cho đẹp và nhanh". Bạn xử lý thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Yêu cầu mơ hồ là chuyện thường ngày, kỹ năng quan trọng là **biến cái mơ hồ thành cái đo được** trước khi viết một dòng code. Tôi làm theo hướng:

1. **Hỏi để làm rõ mục tiêu kinh doanh/người dùng**: "Đẹp" với ai và để làm gì? "Nhanh" nghĩa là LCP dưới bao nhiêu giây, hay cảm giác mượt khi tương tác?
2. **Biến mong muốn thành tiêu chí có thể kiểm chứng (acceptance criteria)**: ví dụ "đẹp" -> theo design system đã có; "nhanh" -> LCP < 2.5s, không có layout shift (CLS gần 0).
3. **Làm rõ phạm vi và độ ưu tiên**: dữ liệu lấy từ đâu, cần real-time không, có bao nhiêu loại người dùng, thiết bị mục tiêu (mobile?).
4. **Đề xuất giải pháp tối giản trước (MVP)** rồi xác nhận lại bằng wireframe/prototype để tránh hiểu sai sau khi đã code xong.
5. **Ghi lại thống nhất** (dù chỉ vài dòng) để sau này không cãi nhau "anh đâu có nói vậy".

Đây là dạng câu hỏi hành vi, nên có thể trả lời theo cấu trúc dưới đây.

</details>

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

- **Situation (Tình huống)**: "Ở dự án trước, PM giao tôi làm trang dashboard với mô tả rất ngắn là 'cần trực quan và load nhanh', không có spec chi tiết."
- **Task (Nhiệm vụ)**: "Việc của tôi là vừa làm rõ yêu cầu vừa giao đúng thứ người dùng cần, tránh làm xong rồi phải đập đi làm lại."
- **Action (Hành động)**: "Tôi đặt một buổi 30 phút với PM và designer, hỏi 3 nhóm câu: ai dùng và để ra quyết định gì, 'nhanh' đo bằng chỉ số nào, dữ liệu cập nhật theo thời gian thực hay theo phút. Tôi chốt tiêu chí cụ thể: LCP < 2.5s, dùng đúng design system, ưu tiên 3 widget quan trọng nhất. Sau đó tôi dựng nhanh một prototype để xác nhận trước khi code thật."
- **Result (Kết quả)**: "Nhờ chốt tiêu chí từ đầu, bản đầu tiên được duyệt gần như ngay, tránh được một vòng làm lại. Trang đạt LCP khoảng 1.8s nhờ ưu tiên render widget chính và lazy-load phần còn lại."

</details>

---

### 13.7. Một task được ước lượng "2 ngày" hóa ra mất 5 ngày. Bạn ước lượng thời gian một task frontend như thế nào để chính xác hơn và xử lý khi sắp trễ ra sao?

<details>
<summary>👉 Xem câu trả lời</summary>

Ước lượng sai thường vì chỉ tính thời gian code "trường hợp lý tưởng" mà bỏ quên các phần ẩn. Cách tôi cải thiện:

1. **Chia nhỏ task tới mức từng phần đo được** — task càng nhỏ ước lượng càng chính xác. Task lớn hơn nửa ngày nên được tách ra.
2. **Tính cả công việc ẩn (hidden work)**: edge case, trạng thái loading/empty/error, responsive, accessibility, viết test, code review, sửa sau review, tích hợp API thật (thường khác mock).
3. **Nhận diện điểm bất định (unknowns)**: nếu task có phần chưa từng làm, tách riêng một "spike" (nghiên cứu có giới hạn thời gian) để giảm rủi ro ước lượng.
4. **Đưa ra dải thay vì một con số**: "2-4 ngày, phụ thuộc việc API có sẵn không" trung thực hơn "2 ngày".
5. **Khi phát hiện sắp trễ, báo SỚM**, kèm lựa chọn: giảm phạm vi, cắt phần phụ, hay xin thêm thời gian — đừng im lặng tới phút chót.

```text
Ví dụ phân rã task "Làm trang danh sách user":
- UI bảng + phân trang ............... 0.5 ngày
- Tích hợp API + loading/error/empty . 0.5 ngày
- Tìm kiếm + lọc (debounce, sync URL) . 0.5 ngày   <- thường bị quên
- Responsive + a11y .................. 0.5 ngày   <- thường bị quên
- Test + sửa sau review .............. 0.5 ngày   <- thường bị quên
=> Thực tế ~2.5 ngày, không phải "1 ngày code bảng"
```

**Bẫy thường gặp**: ước lượng theo "thời gian gõ code" chứ không phải "thời gian đến khi merge được"; quên buffer cho review và sửa lỗi; cam kết con số cứng cho phần còn nhiều unknown; và tệ nhất là giấu việc trễ tới sát deadline.

</details>

---

### 13.8. Khi gặp một bài toán lớn (ví dụ "xây dựng tính năng editor giàu định dạng có cộng tác real-time"), bạn dùng kỹ thuật gì để chia nhỏ nó?

<details>
<summary>👉 Xem câu trả lời</summary>

Bài toán lớn đáng sợ vì mơ hồ; chia nhỏ là cách biến nó thành chuỗi bước có thể làm và kiểm chứng. Các kỹ thuật:

1. **Chia theo lát cắt dọc (vertical slice)** thay vì lát ngang: làm một luồng end-to-end nhỏ nhưng chạy được (gõ chữ -> lưu -> hiển thị), rồi mở rộng. Tránh làm xong toàn bộ UI mà chưa có gì hoạt động.
2. **Tách theo rủi ro/độ bất định**: phần khó và mới nhất (đồng bộ real-time) làm prototype/spike trước để gỡ rủi ro sớm, đừng để cuối mới phát hiện không khả thi.
3. **Tách theo ranh giới kỹ thuật**: tầng render (editor), tầng dữ liệu (mô hình tài liệu), tầng đồng bộ (transport/CRDT). Mỗi tầng có thể test độc lập.
4. **Xác định MVP và những gì cắt được**: bản đầu chỉ cần bold/italic + lưu, chưa cần cộng tác; cộng tác là pha sau.
5. **Vẽ phụ thuộc giữa các phần** để biết thứ tự làm và phần nào song song được (chia việc cho team).

```text
"Editor cộng tác real-time" tách thành các lát cắt:
Pha 0 (spike): thử CRDT/OT trên prototype tối thiểu -> xác nhận khả thi
Pha 1: editor cơ bản 1 người (bold/italic), lưu/đọc tài liệu        <- chạy được end-to-end
Pha 2: lịch sử undo/redo, autosave
Pha 3: hiển thị con trỏ + nhiều người cùng sửa (real-time)
Pha 4: xử lý xung đột, offline, quyền chỉnh sửa
```

**Bẫy thường gặp**: bắt đầu từ phần dễ/vui (UI) thay vì phần rủi ro cao; chia ngang khiến không có gì demo được suốt nhiều tuần; ôm trọn cả bài toán trong một PR khổng lồ khó review; không xác định MVP nên làm cả những thứ chưa ai cần.

</details>

---

### 13.9. Bạn cân bằng giữa trải nghiệm người dùng (UX), hiệu năng và trải nghiệm lập trình (DX) như thế nào khi chúng mâu thuẫn?

<details>
<summary>👉 Xem câu trả lời</summary>

Ba yếu tố này thường kéo về ba hướng, và nguyên tắc của tôi là **người dùng cuối được ưu tiên cao nhất**, nhưng không vì thế mà hy sinh khả năng bảo trì tới mức không ai dám đụng vào code. Cách tôi cân bằng:

1. **Mặc định ưu tiên UX**: người dùng không quan tâm code đẹp hay không; họ quan tâm app có nhanh, mượt, đúng kỳ vọng không.
2. **Hiệu năng phục vụ UX, không phải mục tiêu tự thân**: tối ưu phần ảnh hưởng cảm nhận người dùng (LCP, INP, không giật khi cuộn) chứ không phải vi tối ưu vô hình.
3. **DX là khoản đầu tư dài hạn**: code dễ đọc giúp ít bug hơn và ship nhanh hơn về sau — tức là gián tiếp tốt cho UX. Chỉ khi DX và UX thật sự xung đột thì UX thắng, nhưng phải ghi rõ lý do (để người sau hiểu vì sao code "xấu").
4. **Quyết định dựa trên dữ liệu**: đo trước, đừng đoán.

Ví dụ tình huống xung đột thường gặp giữa UX và hiệu năng:

| Lựa chọn | Lợi UX | Cái giá |
|---|---|---|
| Optimistic update (cập nhật UI ngay trước khi server trả lời) | Cảm giác tức thì, mượt | DX phức tạp hơn: phải xử lý rollback khi lỗi |
| Hiển thị skeleton thay vì spinner | Cảm giác nhanh hơn, giảm CLS | Tốn công làm và bảo trì nhiều biến thể skeleton |
| Virtualize danh sách lớn | Cuộn mượt, không treo trình duyệt | Code khó hơn, phá Ctrl+F của trình duyệt, khó a11y |

```tsx
// Ví dụ optimistic update với TanStack Query: ưu tiên UX (phản hồi tức thì),
// chấp nhận DX phức tạp hơn vì phải tự rollback khi lỗi
useMutation({
  mutationFn: toggleLike,
  onMutate: async (postId) => {
    await queryClient.cancelQueries({ queryKey: ['post', postId] });
    const prev = queryClient.getQueryData(['post', postId]);
    queryClient.setQueryData(['post', postId], (old: any) => ({ ...old, liked: !old.liked }));
    return { prev }; // lưu lại để rollback
  },
  onError: (_e, postId, ctx) => {
    queryClient.setQueryData(['post', postId], ctx?.prev); // hoàn tác khi lỗi
  },
  onSettled: (_d, _e, postId) => queryClient.invalidateQueries({ queryKey: ['post', postId] }),
});
```

**Bẫy thường gặp**: hy sinh UX để code "sạch" theo ý mình (người dùng chịu thiệt); ngược lại, viết code không ai bảo trì nổi nhân danh "vì người dùng"; vi tối ưu hiệu năng những thứ người dùng không cảm nhận được trong khi bỏ qua INP/LCP thực tế.

</details>
## 14. Kỹ năng mềm và Câu hỏi hành vi

### 14.1. Cấu trúc STAR là gì và tại sao nên dùng nó khi trả lời câu hỏi hành vi?

<details>
<summary>👉 Xem câu trả lời</summary>

STAR là một khung trả lời giúp câu chuyện của bạn rõ ràng, có trọng tâm và chứng minh được tác động thực tế thay vì nói chung chung. Nhà tuyển dụng dùng câu hỏi hành vi vì hành vi trong quá khứ là chỉ báo tốt nhất cho hành vi tương lai.

- **S (Situation - Tình huống):** Bối cảnh ngắn gọn. Bạn đang làm dự án gì, vai trò gì.
- **T (Task - Nhiệm vụ):** Vấn đề/mục tiêu cụ thể bạn phải giải quyết.
- **A (Action - Hành động):** Những việc *bạn* đã làm (dùng "tôi", không phải "chúng tôi"). Đây là phần quan trọng nhất, chiếm phần lớn câu trả lời.
- **R (Result - Kết quả):** Kết quả đo lường được, ưu tiên con số, và bài học rút ra.

Ví dụ áp dụng nhanh trong ngữ cảnh frontend:

| Phần | Nội dung mẫu |
| --- | --- |
| Situation | "Dashboard React của chúng tôi mất hơn 6 giây để load lần đầu, khách hàng phàn nàn." |
| Task | "Tôi được giao tối ưu Largest Contentful Paint xuống dưới 2.5 giây." |
| Action | "Tôi dùng `React.lazy` + `Suspense` để code-split route, chuyển ảnh sang `next/image`, và memo hóa các bảng nặng." |
| Result | "LCP giảm còn 1.8 giây, bounce rate giảm 18%." |

**Bẫy thường gặp:** kể lể quá dài phần Situation, quên phần Result (không có con số), hoặc dùng "chúng tôi" suốt khiến người phỏng vấn không biết *bạn* đã làm gì. Hãy chuẩn bị sẵn 4-5 câu chuyện STAR có thể tái sử dụng cho nhiều câu hỏi.

</details>

---

### 14.2. Hãy kể về một lần bạn gây ra (hoặc phát hiện) một bug production làm hỏng giao diện người dùng. Bạn đã xử lý như thế nào?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Câu này kiểm tra khả năng giữ bình tĩnh, ưu tiên giảm thiệt hại trước khi tìm nguyên nhân gốc, và tinh thần trách nhiệm. Đừng đổ lỗi; hãy nhấn mạnh quy trình.

- **Situation:** "Sau một lần deploy lúc chiều thứ Sáu, trang thanh toán bị trắng màn hình (white screen) trên Safari. Người dùng không hoàn tất được đơn hàng."
- **Task:** "Là người trực release, tôi cần khôi phục giao diện càng nhanh càng tốt rồi mới điều tra nguyên nhân."
- **Action:** "Việc đầu tiên tôi làm là *rollback* về bản trước để dừng thiệt hại — ưu tiên người dùng trước, debug sau. Sau đó tôi mở Sentry, thấy lỗi `Cannot read properties of undefined`. Tôi tái hiện được trên Safari: một field mới từ API có thể là `null`, nhưng code dùng optional chaining sai chỗ. Tôi thêm guard và một `ErrorBoundary` bọc luồng thanh toán để lần sau lỗi component không làm trắng cả app. Tôi cũng viết một test case cho đúng trường hợp `null` đó."
- **Result:** "Giao diện khôi phục trong khoảng 8 phút nhờ rollback, fix thật được deploy sau 1 tiếng. Tôi viết một postmortem ngắn không đổ lỗi, và đề xuất thêm `ErrorBoundary` cho các luồng quan trọng — sau đó nó cứu chúng tôi thêm vài lần nữa."

**Điểm nhấn nên thể hiện:** mitigation (rollback / feature flag) đi trước root cause; dùng error monitoring (Sentry/LogRocket); thêm test hồi quy; và biến sự cố thành cải tiến hệ thống (`ErrorBoundary`, postmortem) thay vì chỉ vá lỗi.

```tsx
// Bài học cụ thể: bọc luồng quan trọng bằng ErrorBoundary để 1 component lỗi
// không làm sập toàn bộ cây React (white screen).
<ErrorBoundary fallback={<CheckoutFallback />}>
  <CheckoutFlow />
</ErrorBoundary>
```

</details>

---

### 14.3. Khi backlog vừa có technical debt vừa có tính năng mới mà sếp/khách hàng đang hối, bạn ưu tiên thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Đây là câu kiểm tra tư duy đánh đổi (trade-off) và khả năng giao tiếp với người không chuyên kỹ thuật, không phải kiểm tra xem bạn có "sùng bái code sạch" hay không. Câu trả lời tốt cho thấy bạn ra quyết định dựa trên **tác động kinh doanh và rủi ro**, không phải sở thích cá nhân.

Khung lập luận nên dùng:

- **Không phải mọi tech debt đều ngang nhau.** Phân loại: nợ đang *chảy máu* (gây bug, làm chậm mọi feature, rủi ro bảo mật) thì ưu tiên cao; nợ "khó chịu nhưng không cản trở" thì có thể hoãn.
- **Lượng hóa chi phí.** Thay vì nói "code này xấu", hãy nói "mỗi feature đụng vào module này tốn thêm 2-3 ngày và hay sinh bug". Ngôn ngữ tác động dễ thuyết phục stakeholder hơn.
- **Chiến lược "đóng thuế" (boy scout rule).** Dành ~15-20% mỗi sprint cho việc trả nợ, hoặc refactor *từng phần* ngay trên đường làm feature mới đụng tới vùng đó.
- **Để stakeholder quyết định có thông tin.** Trình bày rõ: "Làm feature này trước thì ra mắt sớm 1 tuần, nhưng sẽ tăng rủi ro X. Trả nợ trước thì chậm 1 tuần nhưng các feature sau nhanh hơn." Người ra quyết định cuối là họ, việc của bạn là cho họ dữ liệu.

| Loại | Ví dụ frontend | Hành động |
| --- | --- | --- |
| Nợ nguy hiểm | State management rối khiến mỗi feature mới đẻ ra bug, dependency có CVE | Ưu tiên xử lý gần như ngay |
| Nợ cản trở | Không có test cho luồng core, prop drilling sâu | Trả dần trong khi làm feature liên quan |
| Nợ thẩm mỹ | Tên biến chưa đẹp, CSS hơi trùng lặp | Hoãn, gom vào lúc rảnh |

**Bẫy thường gặp:** trả lời cực đoan kiểu "luôn ưu tiên feature mới vì kinh doanh là trên hết" (cho thấy thiếu trách nhiệm dài hạn) hoặc "luôn refactor cho sạch trước" (cho thấy thiếu nhạy bén kinh doanh). Câu trả lời tốt nằm ở giữa: *tùy mức độ rủi ro và tác động*.

</details>

---

### 14.4. Hãy kể về một lần bạn bất đồng với designer hoặc backend developer. Bạn giải quyết xung đột đó ra sao?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Nhà tuyển dụng muốn thấy bạn xử lý bất đồng dựa trên dữ liệu và mục tiêu chung, không phải cái tôi. Frontend là vị trí "ở giữa" designer và backend nên kỹ năng này cực kỳ quan trọng.

Ví dụ xung đột với designer:

- **Situation:** "Designer thiết kế một animation trang trí phức tạp cho danh sách 200+ item, dùng Framer Motion với layout animation cho từng item."
- **Task:** "Tôi nhận ra nó gây giật (jank) nặng trên điện thoại tầm trung, nhưng tôi tôn trọng ý đồ thẩm mỹ của bạn ấy."
- **Action:** "Tôi không nói thẳng 'cái này không làm được'. Thay vào đó tôi quay màn hình demo độ giật trên thiết bị thật, và đưa ra phương án thay thế: chỉ animate khi item vào viewport, tôn trọng `prefers-reduced-motion`, và dùng transform thay vì animate layout. Tôi mời designer ngồi xem cùng để cùng tinh chỉnh."
- **Result:** "Chúng tôi giữ được 90% cảm giác thiết kế nhưng đạt 60fps. Quan trọng hơn, sau đó designer chủ động hỏi tôi về giới hạn hiệu năng ngay từ giai đoạn thiết kế."

Ví dụ xung đột với backend (về hình dạng API):

- **Situation:** "Backend trả về một mảng phẳng nhưng UI cần dữ liệu dạng cây để render menu nhiều cấp."
- **Action:** "Tôi đề xuất phương án và để backend chọn: hoặc backend trả về dạng lồng nhau (đỡ việc cho frontend, đúng hơn về mặt domain), hoặc tôi tự build cây phía client nếu họ quá tải. Tôi viết ra một contract (kiểu TypeScript / OpenAPI) để hai bên thống nhất trước khi code."
- **Result:** "Chúng tôi chọn backend trả dạng cây vì tái dùng được cho mobile. Việc thống nhất contract trước giúp giảm hẳn các vòng đi-về sửa API."

**Điểm nhấn:** dùng *bằng chứng* (demo, số đo, contract) thay vì tranh luận cảm tính; luôn quy về *mục tiêu chung* (trải nghiệm người dùng, tiến độ); và đề xuất nhiều phương án để bên kia có quyền lựa chọn.

</details>

---

### 14.5. Làm sao bạn giải thích một quyết định kỹ thuật phức tạp cho người không chuyên (PM, khách hàng, sếp)?

<details>
<summary>👉 Xem câu trả lời</summary>

Kỹ năng này phân biệt một dev "chỉ code" với một dev có thể ảnh hưởng đến quyết định. Nguyên tắc cốt lõi: **nói về tác động (impact), không nói về cơ chế (mechanism)**. Người không chuyên quan tâm "việc này ảnh hưởng gì đến người dùng, tiền bạc, thời gian", không quan tâm "chúng ta dùng `useMemo` hay không".

Các kỹ thuật cụ thể:

- **Dùng phép ẩn dụ (analogy).** Ví dụ giải thích về hydration trong Next.js: "Trang giống như một bức ảnh in sẵn gửi tới người dùng để họ thấy ngay; sau đó JavaScript chạy nền để 'thổi hồn' làm các nút bấm hoạt động được. Khoảng giữa đó là lúc nhìn thì thấy nhưng bấm chưa ăn."
- **Quy đổi sang ngôn ngữ kinh doanh.** Thay vì "trang chưa được code-split" hãy nói "trang tải chậm 3 giây, mỗi giây chậm làm mất khoảng X% người mua hàng".
- **Đưa ra đánh đổi dưới dạng lựa chọn.** "Phương án A nhanh ra mắt nhưng tốn chi phí bảo trì về sau; phương án B ngược lại. Tôi đề xuất A vì lý do..." — cho họ thấy bạn cân nhắc chứ không áp đặt.
- **Trực quan hóa.** Một biểu đồ, một bản demo, một con số "trước/sau" thường thắng cả ngàn lời giải thích.

Ví dụ thực tế: thuyết phục PM cho thời gian chuyển từ Client-Side Rendering sang SSR/SSG bằng Next.js.

> ❌ "App đang CSR nên TTFB ổn nhưng FCP và LCP tệ, ta nên dùng `getStaticProps` để pre-render."
>
> ✅ "Hiện tại người dùng nhìn màn hình trắng khoảng 4 giây và Google đánh giá tốc độ trang của ta thấp, ảnh hưởng thứ hạng tìm kiếm. Có một kỹ thuật cho phép tạo sẵn trang để người dùng thấy nội dung gần như tức thì và Google index tốt hơn. Tôi cần khoảng 3 ngày, đổi lại tốc độ và SEO cải thiện rõ rệt."

**Bẫy thường gặp:** dùng thuật ngữ (jargon) khiến người nghe gật gù nhưng không hiểu rồi ra quyết định sai; hoặc đi quá sâu vào chi tiết triển khai. Hãy kiểm tra lại bằng cách hỏi "anh/chị thấy phần này có rõ không?".

</details>

---

### 14.6. Kể về một lần bạn nhận được feedback tiêu cực (ví dụ trong code review hoặc đánh giá hiệu suất). Bạn phản ứng thế nào?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Câu này đo độ trưởng thành về cảm xúc (emotional maturity) và khả năng học hỏi (growth mindset). Đừng chọn một feedback "giả vờ tiêu cực" kiểu "tôi quá cầu toàn". Hãy chọn feedback thật, cho thấy bạn đã thực sự thay đổi.

- **Situation:** "Trong một code review, một senior nhận xét PR của tôi gom quá nhiều thay đổi (component mới + refactor + đổi state management) khiến review rất khó và rủi ro cao."
- **Task:** "Phản ứng đầu tiên trong đầu tôi là hơi tự ái vì tôi đã bỏ nhiều công. Nhưng tôi tự nhắc rằng feedback nhắm vào *code*, không nhắm vào *tôi*."
- **Action:** "Tôi cảm ơn, hỏi lại cho rõ kỳ vọng, rồi tách PR đó thành ba PR nhỏ độc lập. Tôi cũng chủ động hỏi senior đó về quy ước PR của team. Từ đó tôi tập thói quen mỗi PR chỉ làm một việc và viết mô tả rõ ràng."
- **Result:** "Các PR sau của tôi được merge nhanh hơn hẳn vì dễ review. Vài tháng sau, chính tôi lại đưa ra lời khuyên tương tự cho một bạn junior — feedback đã trở thành tiêu chuẩn của tôi."

**Điểm nhấn nên thể hiện:** thừa nhận cảm xúc ban đầu một cách trung thực (cho thấy bạn là người thật, không phòng thủ), tách biệt cái tôi khỏi công việc, biến feedback thành hành động cụ thể và thay đổi lâu dài. Tránh đổ lỗi cho người đưa feedback hoặc nói rằng họ "hiểu sai".

</details>

---

### 14.7. Bạn cập nhật kiến thức trong một hệ sinh thái thay đổi nhanh như React/Next.js bằng cách nào?

<details>
<summary>👉 Xem câu trả lời</summary>

Câu này kiểm tra khả năng tự học và sự nhạy bén với xu hướng — nhưng đồng thời cũng kiểm tra **sự cân bằng**: bạn không nên là người chạy theo mọi công nghệ mới (shiny object syndrome). Câu trả lời tốt cho thấy bạn vừa cập nhật, vừa biết đánh giá khi nào *nên* và *chưa nên* áp dụng.

Nguồn và cách học hiệu quả (chọn vài cái bạn thật sự dùng, đừng đọc thuộc lòng):

- **Nguồn chính thống:** blog và changelog chính thức của React, Next.js (vercel.com/blog), TanStack. Đọc release notes là cách nhanh nhất nắm được hướng đi.
- **Theo dõi người tạo ra công cụ:** ví dụ Dan Abramov, đội ngũ Vercel, Tanner Linsley (TanStack), trên X/GitHub.
- **RFC và GitHub Discussions:** đọc RFC giúp hiểu *tại sao* một API ra đời, không chỉ *cách dùng*. Ví dụ hiểu động cơ đằng sau React Server Components hay React Compiler.
- **Học bằng cách làm:** dựng một side project nhỏ để thử công nghệ mới (ví dụ thử App Router của Next.js, hay thử React Server Components) trước khi đề xuất đưa vào dự án thật.
- **Newsletter/cộng đồng:** các bản tin tổng hợp hàng tuần, podcast khi đi lại, để không bỏ lỡ bức tranh lớn mà không tốn quá nhiều thời gian.

Quan trọng nhất là **bộ lọc đánh giá** khi quyết định có dùng công nghệ mới hay không:

| Tiêu chí | Câu hỏi cần đặt ra |
| --- | --- |
| Độ chín (maturity) | Đã stable chưa hay còn experimental/canary? (vd RSC mất khá lâu mới ổn định) |
| Vấn đề thật | Nó giải quyết vấn đề ta đang gặp, hay chỉ "trông hay"? |
| Chi phí chuyển đổi | Team có học được không? Migration tốn bao nhiêu? |
| Cộng đồng & bảo trì | Có được duy trì tích cực, nhiều người dùng, dễ tuyển người biết không? |

**Bẫy thường gặp:** trả lời kiểu "tôi luôn dùng phiên bản mới nhất / công nghệ hot nhất" — điều này khiến nhà tuyển dụng lo bạn sẽ đưa rủi ro vào dự án. Hãy cho thấy bạn cập nhật để *biết và đánh giá*, rồi mới áp dụng có chọn lọc.

</details>

---

### 14.8. Kể về một lần bạn mentor một đồng nghiệp ít kinh nghiệm hơn, hoặc giúp đỡ cả team. Kết quả ra sao?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Câu này (thường hỏi ở cấp Senior) đánh giá khả năng nâng tầm người khác (force multiplier) và tư duy lâu dài. Trọng tâm là bạn dạy *cách câu cá* chứ không chỉ *cho con cá*.

- **Situation:** "Một bạn junior trong team liên tục gặp bug về stale closure và component re-render thừa khi dùng `useEffect`, dẫn đến vài bug khó chịu trên production."
- **Task:** "Tôi muốn giúp bạn ấy hiểu *nguyên lý* chứ không chỉ sửa hộ từng bug, để bạn ấy tự đứng vững."
- **Action:** "Tôi pair programming với bạn ấy một buổi, vẽ ra cách closure 'chụp' lại giá trị state ở mỗi lần render, và khi nào cần dependency array. Sau đó tôi không sửa code hộ nữa mà để bạn ấy tự sửa rồi tôi review. Tôi cũng làm một buổi chia sẻ nội bộ 30 phút về 'các bẫy thường gặp của `useEffect`' cho cả team và đề xuất bật rule `react-hooks/exhaustive-deps` trong ESLint."
- **Result:** "Sau khoảng một tháng, số bug liên quan giảm rõ và bạn ấy tự tin nhận task khó hơn. Buổi chia sẻ + ESLint rule giúp cả team tránh lặp lại lỗi này, đó là tác động lan tỏa mà tôi tự hào nhất."

**Điểm nhấn nên thể hiện:** kiên nhẫn, dạy nguyên lý thay vì sửa hộ, tạo *đòn bẩy* cho cả team (tài liệu, lint rule, buổi share) chứ không chỉ giúp một người, và đo bằng kết quả thực tế. Tránh kể kiểu "tôi giỏi nên tôi sửa hết cho mọi người" — đó là chống mentor.

</details>

---

### 14.9. Hãy kể về một lần bạn phải làm việc dưới áp lực deadline rất gắt. Bạn xử lý thế nào để vừa kịp tiến độ vừa không bung chất lượng?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Câu này đo khả năng quản lý áp lực, ưu tiên việc, và giao tiếp sớm. Điều nhà tuyển dụng *không* muốn nghe là "tôi cày xuyên đêm và làm hết". Họ muốn thấy bạn ưu tiên thông minh và giao tiếp kịp thời về rủi ro.

- **Situation:** "Chúng tôi phải kịp launch một landing page chiến dịch trong 3 ngày trước sự kiện báo chí, nhưng phạm vi yêu cầu lớn hơn thời gian cho phép."
- **Task:** "Mục tiêu là live đúng hạn với chất lượng chấp nhận được cho luồng quan trọng nhất, thay vì làm hết mọi thứ một cách dở dang."
- **Action:** "Đầu tiên tôi cùng PM phân loại tính năng theo MoSCoW (Must / Should / Could). Form đăng ký và phần hero là Must; animation phụ và A/B test là Could — chúng tôi cắt khỏi bản đầu. Tôi tái sử dụng component có sẵn trong design system thay vì làm mới, và dùng feature flag để có thể bật phần chưa hoàn thiện sau ngày launch mà không cần deploy lại. Tôi cũng cảnh báo sớm cho stakeholder ngay ngày đầu rằng nếu giữ nguyên scope thì sẽ trễ — để họ quyết cắt gì."
- **Result:** "Chúng tôi launch đúng hạn với luồng chính chạy ổn và có test cho phần thanh toán/đăng ký. Các phần 'Could' được bổ sung dần trong tuần sau qua feature flag. Không có sự cố nào trong sự kiện."

**Điểm nhấn nên thể hiện:** cắt phạm vi (scope) một cách có chủ đích thay vì cắt chất lượng; giao tiếp rủi ro *sớm* chứ không phải đến hạn mới báo; tái sử dụng và feature flag để giảm rủi ro; vẫn giữ test cho phần quan trọng. Tránh nói rằng bạn hy sinh giấc ngủ và sức khỏe — điều đó cho thấy quản lý kém chứ không phải đáng khen.

</details>

---

### 14.10. Mô tả một lần bạn mắc một lỗi đáng kể trong công việc. Bạn đã làm gì sau đó?

<details>
<summary>👉 Gợi ý trả lời (STAR)</summary>

Đây là câu kiểm tra tính trung thực, tinh thần trách nhiệm (ownership) và khả năng học hỏi. Người phỏng vấn cảnh giác với ứng viên nói "tôi chưa từng mắc lỗi đáng kể nào" — nó cho thấy thiếu tự nhận thức. Hãy chọn một lỗi thật, nhận trách nhiệm rõ ràng, và tập trung vào bài học.

- **Situation:** "Tôi đẩy một thay đổi vào cách lưu state lên localStorage mà không kiểm thử kỹ trên tab ẩn danh / chế độ riêng tư của Safari, nơi `localStorage` bị giới hạn."
- **Task:** "Hệ quả là một nhóm người dùng bị crash app ngay khi mở. Tôi cần thừa nhận và khắc phục."
- **Action:** "Tôi báo ngay cho team và stakeholder thay vì giấu, không đổ lỗi cho trình duyệt. Tôi bọc các lời gọi storage trong try/catch và thêm fallback in-memory, rồi viết test cho trường hợp storage không khả dụng. Sau đó tôi đề xuất thêm bước kiểm thử cross-browser (kể cả chế độ riêng tư) vào checklist QA trước release."
- **Result:** "Lỗi được xử lý trong cùng ngày. Quan trọng hơn, checklist mới giúp team bắt được vài vấn đề tương thích trình duyệt khác trước khi lên production. Tôi học được rằng giả định về môi trường người dùng là một trong những nguồn bug nguy hiểm nhất của frontend."

**Điểm nhấn nên thể hiện:** chủ động báo lỗi và nhận trách nhiệm sớm; sửa cả triệu chứng lẫn quy trình để không tái diễn; rút ra bài học cụ thể. Tránh chọn lỗi quá nhỏ (cho thấy né tránh) hoặc đổ lỗi cho hoàn cảnh/người khác.

</details>
---

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
