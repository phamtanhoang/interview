# 🧱 Bộ câu hỏi phỏng vấn: JavaScript · TypeScript · Node.js (Nền tảng)

> Gom các câu hỏi NỀN TẢNG về **JavaScript, TypeScript, Node.js core** — vốn trùng nhau ở `frontend-interview.md` (§1) và `backend-interview.md` (§1 Node, §21 TS) — về một chỗ; đã **gộp câu trùng** và **bỏ framing React/NestJS**.
> Đây là phần ngôn ngữ & runtime, KHÔNG gắn framework: câu đặc thù React vẫn ở file frontend, đặc thù NestJS/Prisma vẫn ở file backend.
> Mỗi đáp án nằm trong khối collapse (`<details>`) — bấm để xem.

## 📚 Mục lục

1. [JavaScript](#1-javascript)
2. [TypeScript](#2-typescript)
3. [Node.js core](#3-nodejs-core)

---

## 1. JavaScript

### 1.1. Event loop, microtask vs macrotask, và thứ tự chạy (nextTick/Promise/setTimeout/setImmediate)?

<details>
<summary>👉 Xem câu trả lời</summary>

**📌 Câu trả lời mẫu:**

"Node chạy trên một thread duy nhất, nên nó không tự làm việc nặng mà giao cho hệ thống lo, rồi đăng ký một callback để chạy khi xong. Event loop là vòng lặp liên tục lấy các callback đã sẵn sàng ra chạy — nhờ vậy một thread vẫn phục vụ được nhiều request mà không bị đứng khi chờ I/O.

Callback chia hai loại: **macrotask** (`setTimeout`, `setImmediate`, I/O callback) — mỗi lượt làm một cái; và **microtask** (`Promise.then`, code sau `await`, `queueMicrotask`) — việc vặt phải làm xong NGAY. Quy tắc vàng: chạy xong một macrotask thì **vét sạch toàn bộ microtask** rồi mới sang macrotask kế, nên `Promise` luôn chạy trước `setTimeout`. Riêng `process.nextTick` (Node) ưu tiên cao nhất, chen trước cả Promise.

Tóm lại thứ tự là: **đồng bộ → nextTick → Promise → setTimeout**. Điều quan trọng thực tế: đừng chạy việc CPU nặng trên thread chính vì sẽ **block** cả event loop — đẩy sang `worker_threads` hoặc làm async."

> 🍜 Mẹo nhớ: **macrotask** = "bưng 1 món lớn ra" (mỗi lượt 1 cái); **microtask** = "việc vặt phải dọn xong trước khi bưng món tiếp"; **`nextTick`** = "sếp gọi gấp, chen lên đầu".

---

**Micro vs macro task:**

- JS đơn luồng; event loop kiểm tra call stack, khi rỗng thì lấy việc từ các hàng đợi ra chạy.
- **Macrotask**: `setTimeout`, `setInterval`, `setImmediate`, I/O callback, sự kiện DOM, `MessageChannel`.
- **Microtask**: `Promise.then/catch/finally`, code sau `await`, `queueMicrotask`, `MutationObserver`; riêng Node thêm `process.nextTick` (ưu tiên trên cả Promise).
- Quy tắc vàng: sau MỖI macrotask, rút cạn TOÀN BỘ microtask rồi mới sang macrotask kế → microtask luôn chạy trước.
- Bẫy: tưởng `setTimeout(fn, 0)` chạy ngay — thực ra luôn xếp sau mọi microtask đang chờ; sinh microtask/`nextTick` vô hạn sẽ làm đói (starve) event loop.

**Thứ tự ưu tiên:** đồng bộ chạy hết → microtask (`process.nextTick` trước → rồi `Promise`) → macrotask.

**Các phase của event loop (Node):** mỗi tick chạy tuần tự: **timers** (`setTimeout`/`setInterval` hết hạn) → pending callbacks → idle/prepare → **poll** (I/O) → **check** (`setImmediate`) → close callbacks. Callback trong các phase này đều là macrotask; microtask (`nextTick` rồi Promise) được drain sau MỖI callback, không thuộc phase nào.

```js
console.log('A');                                // đồng bộ
setTimeout(() => console.log('E'), 0);           // macrotask
setImmediate(() => console.log('F'));            // macrotask (Node, phase check)
Promise.resolve().then(() => console.log('D'));  // microtask
process.nextTick(() => console.log('C'));        // microtask ưu tiên cao nhất (Node)
console.log('B');                                // đồng bộ
// In ra: A, B, C, D, rồi E/F
// E/F không xác định ở top-level; chỉ trong I/O callback thì F (immediate) mới luôn trước E (timeout)
```

</details>

---

### 1.2. Closure là gì? Cho ví dụ và một bug thường gặp (stale closure)?

<details>
<summary>👉 Xem câu trả lời</summary>

- Closure: hàm "ghi nhớ" và tiếp tục truy cập được các biến ở scope nơi nó được khai báo (lexical scope), kể cả khi hàm ngoài đã `return` và scope gốc tưởng như đã kết thúc.
- Ứng dụng tích cực: factory giữ state riêng (đếm, cache), data privacy (mô phỏng biến private), memoization, currying, giữ tham chiếu ổn định cho callback.
- Bug phổ biến — **stale closure**: callback "đóng băng" (capture) giá trị tại thời điểm tạo; nếu biến đó thay đổi sau này, callback vẫn dùng giá trị cũ. Hay gặp trong `setInterval`/`setTimeout`/event handler chạy lâu dài.
- Cách khắc phục: dùng **functional update** thay vì đọc biến bị đóng băng (truyền hàm nhận giá trị mới nhất), hoặc dùng tham chiếu có thể thay đổi (object/`let` được cập nhật) thay vì capture một snapshot.

```js
// Bug kinh điển: var bị hoisting, mọi callback chia sẻ CÙNG một biến i
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // in 3, 3, 3 (stale: i đã = 3)
}
// Fix: let tạo binding MỚI mỗi vòng lặp
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // in 0, 1, 2
}
```

```js
// Stale closure trong setInterval: count bị đóng băng = 0
function makeCounter() {
  let count = 0;
  return {
    get: () => count,                      // closure đọc biến mới nhất qua tham chiếu
    inc: () => { count += 1; },             // closure GHI vào cùng biến count
  };
}
const c = makeCounter();
c.inc(); c.inc();
console.log(c.get()); // 2 — closure chia sẻ cùng biến, không phải snapshot
```

</details>

---

### 1.3. `this` trong JavaScript hoạt động thế nào? Vì sao arrow function khác hàm thường?

<details>
<summary>👉 Xem câu trả lời</summary>

- Với **hàm thường**, `this` KHÔNG cố định lúc định nghĩa mà được xác định lúc GỌI (call-site):
  - `obj.fn()` → `this` là `obj` (object đứng trước dấu chấm).
  - `fn()` gọi trần → `undefined` ở strict mode, hoặc `globalThis`/`window` ở non-strict.
  - `fn.call(x)` / `fn.apply(x)` / `fn.bind(x)` → `this` chính là giá trị truyền vào (`bind` tạo hàm mới gắn cứng `this`).
  - `new Fn()` → `this` là instance mới vừa tạo.
- **Arrow function** KHÔNG có `this` riêng: nó lấy `this` theo **lexical scope** (scope bao quanh lúc viết code), nên `call/apply/bind` không đổi được `this` của arrow. Lợi ích: truyền làm callback (ví dụ `setTimeout`, event handler) không bị "mất `this`".
- Hệ quả thực tế: bug kinh điển là tách method ra biến rồi gọi (`const f = obj.fn; f()`) làm mất ngữ cảnh → dùng `bind` hoặc bọc arrow để giữ `this`.
- **Bẫy ngược lại**: dùng arrow làm method trên object literal rồi mong `this` trỏ chính object đó — KHÔNG, arrow lấy `this` từ scope ngoài (thường là module/global), không phải object.

```js
const counter = {
  n: 0,
  incNormal() { this.n++; },        // this = counter khi gọi counter.incNormal()
  incArrow: () => { this.n++; },    // this KHÔNG phải counter -> sai
};

class Timer {
  s = 0;
  tick = () => { this.s++; };       // lexical this -> luôn đúng instance dù làm callback
}
setTimeout(new Timer().tick, 1000); // không cần bind
```

| Cách gọi | `this` (hàm thường) |
|---|---|
| `obj.fn()` | `obj` |
| `fn()` | `undefined` (strict) / global |
| `fn.call/apply/bind(x)` | `x` |
| `new Fn()` | instance mới |
| arrow | lexical (bỏ qua call-site) |

</details>

---

### 1.4. Phân biệt chạy nhiều tác vụ async TUẦN TỰ và SONG SONG. Khi nào dùng cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- `await` DỪNG hàm cho tới khi Promise settle. `await` lần lượt từng Promise → chạy TUẦN TỰ (cái sau chờ cái trước xong). Khởi tạo TẤT CẢ Promise trước rồi `await Promise.all([...])` → chạy SONG SONG (cùng khởi động, chờ chung).
- Dùng tuần tự khi tác vụ sau PHỤ THUỘC kết quả tác vụ trước (vd cần `userId` từ bước 1 mới fetch đơn hàng ở bước 2).
- Dùng song song khi các tác vụ ĐỘC LẬP → tổng thời gian xấp xỉ tác vụ chậm nhất thay vì tổng các tác vụ, tránh hiện tượng "waterfall".
- Lưu ý: `await` trong vòng lặp `for` là tuần tự (đôi khi cố ý để giới hạn tải/đúng thứ tự). Bẫy: `forEach` KHÔNG chờ `await`; còn `Promise.all` fail-fast — một Promise reject là cả nhóm reject (cần chịu lỗi từng phần thì dùng `Promise.allSettled`).

```ts
// Tuần tự: ~600ms (mỗi cái 300ms, chờ nhau)
const user = await fetchUser();
const posts = await fetchPosts();

// Song song: ~300ms (khởi động cùng lúc)
const [user, posts] = await Promise.all([fetchUser(), fetchPosts()]);
```

</details>

---

### 1.5. So sánh `Promise.all`, `Promise.allSettled`, `Promise.race`, `Promise.any`?

<details>
<summary>👉 Xem câu trả lời</summary>

Cả bốn đều nhận một iterable các promise và trả về MỘT promise mới, khác nhau ở điều kiện settle:

- `Promise.all`: resolve khi TẤT CẢ resolve (trả mảng kết quả theo đúng thứ tự); reject NGAY khi một cái fail (fail-fast). Dùng khi cần đủ mọi phần dữ liệu, thiếu một cái là vô nghĩa.
- `Promise.allSettled`: chờ TẤT CẢ settled, KHÔNG bao giờ reject; trả mảng `{status, value}` (fulfilled) hoặc `{status, reason}` (rejected). Dùng khi muốn biết kết quả từng cái mà không để một lỗi làm sập cả nhóm.
- `Promise.race`: lấy cái settled ĐẦU TIÊN, dù resolve hay reject. Dùng làm timeout hoặc lấy nguồn phản hồi nhanh nhất.
- `Promise.any`: lấy cái resolve ĐẦU TIÊN; chỉ reject khi TẤT CẢ fail, với lỗi gộp `AggregateError`. Dùng nhiều nguồn dự phòng (đủ một cái thành công là được).

| Method | Resolve khi | Reject khi |
|---|---|---|
| `all` | tất cả resolve | một cái fail (fail-fast) |
| `allSettled` | tất cả settled | không bao giờ |
| `race` | cái đầu tiên settle | cái đầu tiên settle là reject |
| `any` | cái đầu tiên resolve | tất cả fail (`AggregateError`) |

Lưu ý: tất cả promise vẫn được khởi chạy SONG SONG ngay khi tạo; method chỉ quyết định cách CHỜ, không quyết định cách chạy. Bẫy thường gặp: dùng `Promise.all` cho một loạt tác vụ độc lập (ví dụ ghi log nhiều nguồn) — một cái fail là mất hết kết quả đã có; nên dùng `allSettled`.

```ts
// timeout bằng race: chậm quá 5s thì coi như fail
const data = await Promise.race([
  fetchData(),
  new Promise((_, rej) => setTimeout(() => rej(new Error('timeout')), 5000)),
]);
```

</details>

---

### 1.6. Tiến hoá callback → Promise → async/await: callback hell và xử lý lỗi mỗi kiểu?

<details>
<summary>👉 Xem câu trả lời</summary>

Đây là ba thế hệ xử lý code bất đồng bộ (asynchronous) trong JavaScript/Node. Bản chất cả ba giải quyết cùng một việc, chỉ khác về độ dễ đọc và cách bắt lỗi.

- **Callback**: truyền một hàm vào để "gọi lại" khi tác vụ xong; quy ước **error-first** `(err, data)` — lỗi luôn nằm ở tham số đầu, kiểm qua `err`.
- **Callback hell**: callback lồng nhau nhiều tầng (pyramid of doom) → code thụt sâu hình kim tự tháp, lặp check lỗi ở mỗi tầng, khó đọc và khó bảo trì.
- **Promise**: đối tượng đại diện cho kết quả tương lai (`pending → fulfilled / rejected`); nối tiếp **phẳng** bằng `.then()`, gom lỗi tập trung bằng `.catch()`.
- **async/await**: syntactic sugar trên Promise → viết bất đồng bộ trông như đồng bộ; bắt lỗi bằng `try/catch` quen thuộc. (`await` chỉ dùng được trong hàm `async`.)
- **Xử lý lỗi theo từng kiểu**: callback → check `err`; Promise → `.catch()`; async/await → `try/catch`.

```js
// 1) Callback (error-first) → dễ thành "callback hell"
getUser(id, (err, user) => {
  if (err) return handle(err);
  getOrders(user.id, (err, orders) => {   // lồng tầng, lặp check lỗi
    if (err) return handle(err);
    // ...
  });
});

// 2) Promise → phẳng, lỗi gom 1 chỗ
getUser(id)
  .then(user => getOrders(user.id))
  .then(orders => /* ... */)
  .catch(handle);

// 3) async/await → như đồng bộ, dùng try/catch
try {
  const user = await getUser(id);
  const orders = await getOrders(user.id);
} catch (err) {
  handle(err);
}
```

</details>

---

### 1.7. Đồng bộ vs bất đồng bộ; blocking vs non-blocking I/O là gì?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Synchronous**: code chạy tuần tự, dòng sau phải đợi dòng trước xong mới chạy. **Asynchronous**: khởi động một tác vụ rồi đi làm việc khác, kết quả về sau qua callback/Promise/`async-await`.
- **Blocking I/O**: thao tác I/O (đọc file, query DB) chặn luôn thread tới khi hoàn tất, vd `fs.readFileSync`. **Non-blocking I/O**: giao việc cho tầng nền (ở Node là libuv) rồi trả quyền điều khiển lại ngay, kết quả nhận sau qua callback, vd `fs.readFile`.
- Lưu ý: sync/async nói về *cách viết và nhận kết quả*, còn blocking/non-blocking nói về *thread có bị giữ hay không*. Chúng thường đi cùng nhau nhưng không đồng nghĩa.
- Node chạy **single-threaded** trên event loop: một thao tác blocking sẽ làm **đứng cả tiến trình**, không phục vụ được việc nào khác → vì vậy ưu tiên **non-blocking**.
- Non-blocking tận dụng thời gian chờ I/O để xử lý việc khác → rất hợp tải **I/O-bound** (API, DB, network).

```js
// Blocking — đứng đợi đọc xong file mới in "B"
const data = fs.readFileSync('a.txt'); // chặn thread
console.log('B');

// Non-blocking — in "B" ngay, callback chạy sau khi đọc xong
fs.readFile('a.txt', (err, data) => { /* dùng data ở đây */ });
console.log('B'); // chạy trước callback
```

</details>

---

### 1.8. CommonJS (require) vs ES Modules (import) khác nhau thế nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- **CommonJS (CJS)**: `require()` + `module.exports` — kiểu module cũ, **mặc định** của Node. Nạp **đồng bộ, tại runtime** nên có thể `require` bên trong hàm hoặc theo điều kiện `if`.
- **ES Modules (ESM)**: `import` + `export` — chuẩn chính thức của ngôn ngữ. `import` là **tĩnh, phân tích lúc compile**, phải đặt ở **top-level** → nhờ vậy hỗ trợ **tree-shaking** và `import()` động (async).
- **Cách runtime phân biệt**: trong `package.json` có `"type": "module"` thì mọi file `.js` được hiểu là **ESM**; không khai báo (hoặc `"commonjs"`) thì `.js` là **CJS**. Ép từng file: đuôi `.mjs` **luôn** là ESM, `.cjs` **luôn** là CJS — không phụ thuộc `"type"`.
- **Khác biệt thực dụng**: file ESM **không có** sẵn `require`, `module.exports`, `__dirname`, `__filename`; ngược lại file CJS không dùng được `import` tĩnh.
- **Không trộn tuỳ tiện**: trộn lẫn dễ dính lỗi `require is not defined` hoặc `Cannot use import statement outside a module`. ESM có thể `import` một package CJS, nhưng chiều ngược lại (CJS muốn nạp ESM) phải dùng `import()` động vì ESM nạp bất đồng bộ.

| | CommonJS | ES Modules |
|---|---|---|
| Cú pháp | `require` / `module.exports` | `import` / `export` |
| Thời điểm nạp | runtime, **đồng bộ** | compile-time, **tĩnh** |
| Vị trí | bất kỳ (kể cả trong hàm) | **top-level** |
| Tree-shaking | không | có |
| Bật bằng | mặc định / `.cjs` | `"type":"module"` / `.mjs` |

```js
// CommonJS (mặc định)
const fs = require('fs');
module.exports = { foo };

// ES Modules (cần "type": "module" hoặc đuôi .mjs)
import fs from 'fs';
export const foo = () => {};
```

</details>

---

## 2. TypeScript

### 2.1. Generics trong TypeScript là gì và khi nào dùng?

<details>
<summary>👉 Xem câu trả lời</summary>

- Generics cho phép viết hàm/class/type chạy được với NHIỀU kiểu mà vẫn giữ type-safe, thay vì dùng `any`. Hiểu nôm na: "tham số kiểu" (`T`) được điền lúc gọi.
- Compiler tự suy luận (infer) `T` từ input → output giữ đúng kiểu, vẫn có autocomplete và bắt lỗi lúc compile.
- Khi nào dùng: các wrapper/tiện ích tái sử dụng trên nhiều kiểu — `first<T>`, hàm xử lý collection, cấu trúc bao đóng dữ liệu (`Container<T>`, `Result<T>`)... Nếu một hàm chỉ làm việc với đúng một kiểu cụ thể thì KHÔNG cần generic.
- Ràng buộc bằng `extends` (constraint) để giới hạn kiểu đầu vào: `<T extends { id: string }>`; `keyof T` đảm bảo chỉ truy cập field hợp lệ.
- Bẫy: lạm dụng làm signature khó đọc; generic chỉ kiểm tra lúc compile — ép kiểu kiểu `as T` (vd `JSON.parse` trả `any`/`unknown`) KHÔNG xác thực runtime, dữ liệu không tin cậy nên validate (Zod, type guard).

```ts
function first<T>(arr: T[]): T | undefined { return arr[0]; }
const n = first([1, 2, 3]); // n: number | undefined

interface ApiResponse<T> { data: T; meta: { total: number }; }
function paginate<T extends { id: string }>(items: T[]): ApiResponse<T[]> {
  return { data: items, meta: { total: items.length } };
}
```

</details>

---

### 2.2. `interface` và `type` khác nhau thế nào? Khi nào chọn cái nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- Cả hai đều mô tả shape của dữ liệu và phần lớn thay thế được cho nhau khi dùng với object thuần.
- `interface`: mô tả shape object/class; mở rộng bằng `extends`; hỗ trợ **declaration merging** (nhiều khai báo trùng tên được gộp lại).
- `type`: alias tổng quát hơn — biểu diễn được **union, intersection, tuple, primitive alias, mapped/conditional type**; mở rộng bằng intersection `&`.
- Quy tắc thực dụng: dùng `interface` cho contract object ổn định (public API, nơi cần augmentation); dùng `type` cho union/tuple/kiểu tính toán mà `interface` không làm được. Quan trọng nhất là **nhất quán** trong codebase.
- Bẫy: declaration merging của `interface` có thể gây gộp ngoài ý muốn; `type` trùng tên báo lỗi ngay → an toàn hơn khi cần chắc kiểu không bị mở rộng.

```ts
interface User { id: string; name: string; }
interface User { email: string; }        // merge -> User có cả 3 field
interface Admin extends User { role: string; }

type Status = 'idle' | 'loading' | 'done';        // interface không làm được
type Point = [number, number];                    // tuple
type WithTimestamps<T> = T & { createdAt: Date };  // intersection + generic
```

</details>

---

### 2.3. Phân biệt `any`, `unknown` và `never`. Vì sao nên ưu tiên `unknown` hơn `any`?

<details>
<summary>👉 Xem câu trả lời</summary>

- `any`: tắt hoàn toàn kiểm tra kiểu — gán được mọi thứ, thao tác gì cũng "hợp lệ" trên compiler, và còn **lan ra** (gán `any` cho biến khác làm biến đó cũng mất kiểm tra). Tiện nhưng mất an toàn → nên tránh.
- `unknown`: "top type" an toàn — cũng nhận mọi giá trị như `any`, nhưng **không cho thao tác gì** cho đến khi narrow (`typeof`, `in`, `instanceof`, type guard, ép kiểu có kiểm soát). Đây là thay thế an toàn cho `any`.
- `never`: "bottom type" — kiểu của thứ **không bao giờ tồn tại**: hàm luôn `throw`/vòng lặp vô hạn, hoặc nhánh bất khả thi. Không gán được giá trị nào cho `never`. Dùng chính cho exhaustiveness check.
- Vì sao ưu tiên `unknown`: dữ liệu đến từ ngoài (`JSON.parse`, payload mạng, input người dùng) chưa chắc đúng kiểu. Gõ `unknown` **buộc** phải validate/narrow trước khi dùng, biến lỗi runtime tiềm ẩn thành lỗi biên dịch; còn `any` âm thầm bỏ qua và để bug lọt xuống runtime.

```ts
function parse(json: string): unknown { return JSON.parse(json); }
const data = parse('{}');
if (typeof data === 'object' && data !== null && 'id' in data) {
  // OK sau narrow; nếu để any thì truy cập data.id luôn mà không ai cảnh báo
}

type Status = 'idle' | 'done';
function render(s: Status): string {
  switch (s) {
    case 'idle': return '...';
    case 'done': return 'OK';
    default: const _exhaustive: never = s; return _exhaustive; // quên 1 nhánh -> lỗi biên dịch
  }
}
```

</details>

---

### 2.4. Utility types `Partial`, `Pick`, `Omit`, `Record` làm gì? Ví dụ thực tế?

<details>
<summary>👉 Xem câu trả lời</summary>

- `Partial<T>`: biến mọi thuộc tính của `T` thành optional — hợp cho hàm update/patch nhận object một phần. `Required<T>` là chiều ngược lại.
- `Pick<T, K>`: lấy ra một tập thuộc tính `K` từ `T`. `Omit<T, K>`: tạo type mới bằng cách bỏ đi tập thuộc tính `K` (vd type "tạo mới" bỏ `id` do DB tự sinh).
- `Record<K, V>`: object có key thuộc kiểu `K`, value thuộc kiểu `V` — dùng cho lookup table / map tra cứu (vd index theo `id`).
- Lợi ích chung: derive type từ một "source of truth" duy nhất, không khai báo lặp; khi `T` đổi thì các type dẫn xuất tự cập nhật.
- Bẫy: `Partial`, `Pick`, `Omit` chỉ tác động ở cấp một (không đệ quy/recursive) — nested object vẫn giữ nguyên kiểu cũ.

```ts
interface User { id: string; name: string; email: string; }

function updateUser(id: string, patch: Partial<User>) {} // { name?, email?, ... }
type CreateUserInput = Omit<User, 'id'>;                  // { name; email }
type UserPreview     = Pick<User, 'id' | 'name'>;         // { id; name }
type UsersById       = Record<string, User>;              // map id -> User
```

</details>

---

### 2.5. Type guard và narrowing hoạt động ra sao? Cho ví dụ custom type guard?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Narrowing**: trong một nhánh code, TypeScript tự thu hẹp kiểu của biến dựa trên kiểm tra runtime — `typeof`, `instanceof`, `in`, so sánh literal, hoặc check `=== null`/truthiness. Compiler dùng kết quả check để biết chắc kiểu trong nhánh đó.
- **Custom type guard**: hàm trả về `param is Type` (type predicate). Nếu hàm trả `true`, compiler coi `param` là `Type` trong nhánh sau. Đây là cách tự định nghĩa narrowing khi các check built-in không đủ.
- Đặc biệt hữu ích khi xử lý `unknown` từ nguồn ngoài (JSON parse, dữ liệu I/O, payload mạng) để chuyển an toàn sang kiểu đã biết, thay vì ép `as`.
- **Lưu ý**: predicate là một "lời hứa" với compiler — logic kiểm tra sai sẽ tạo lỗ hổng kiểu mà compiler vẫn tin. Phải kiểm tra cả sự tồn tại lẫn kiểu của từng field. Khi shape phức tạp, nên dùng thư viện validate runtime (vd Zod) để sinh guard tự động.

```ts
interface User { id: string; name: string; }

function isUser(x: unknown): x is User {
  return typeof x === 'object' && x !== null &&
    'id' in x && typeof (x as Record<string, unknown>).id === 'string' &&
    'name' in x && typeof (x as Record<string, unknown>).name === 'string';
}

const data: unknown = JSON.parse(input);
if (isUser(data)) {
  data.name; // OK: đã narrow về User
}
```

</details>

---

### 2.6. `satisfies` và `as const` dùng để làm gì? Khác type annotation thường?

<details>
<summary>👉 Xem câu trả lời</summary>

- **`as const`**: ép literal thành `readonly` và narrow về kiểu hẹp nhất — `'TODO'` thay vì `string`, mảng thành tuple `readonly [...]`. Hợp cho hằng số/config cần giữ nguyên giá trị cụ thể.
- **`satisfies` (TS 4.9+)**: kiểm tra expression có thỏa mãn một kiểu hay không, NHƯNG không làm rộng (widen) kiểu suy diễn về kiểu đó — biến vẫn giữ kiểu cụ thể nhất.
- **Annotation `: T`**: ép biến mang đúng kiểu `T`, nên có thể mất thông tin literal/key cụ thể (vd suy ra `Record<string, ...>` làm mất danh sách key thật, autocomplete kém).
- Tóm lại: `: T` chỉ ràng buộc; `satisfies` vừa ràng buộc vừa giữ kiểu hẹp; `as const` khóa giá trị bất biến và narrow tối đa. Thường kết hợp `as const` + `satisfies` cho config.

```ts
type Config = Record<string, { port: number }>;

// Annotation: hợp lệ nhưng b.db bị mất, key bị widen về string
const a: Config = { db: { port: 5432 } };

// satisfies: vẫn check ràng buộc, mà giữ key 'db' cho autocomplete
const b = { db: { port: 5432 } } satisfies Config;
b.db.port; // OK, TS biết chính xác key 'db'

const ROLES = ['OWNER', 'MEMBER'] as const; // readonly ['OWNER', 'MEMBER']
type Role = typeof ROLES[number];           // 'OWNER' | 'MEMBER'
```

</details>

---

### 2.7. `enum` và union literal: nên dùng loại nào?

<details>
<summary>👉 Xem câu trả lời</summary>

- `enum` **tồn tại lúc runtime**: TS phát ra một object thật (numeric enum còn có reverse mapping), nên tốn code sinh ra và có vài điểm gây bất ngờ.
- Union literal (`'TODO' | 'DONE'`) chỉ tồn tại lúc **compile-time**: zero runtime cost, narrow rất tốt, dễ đồng bộ với giá trị string ở nguồn ngoài (DB, JSON, API).
- Xu hướng hiện nay: ưu tiên **union literal** hoặc **`as const` object**; chỉ dùng `enum` khi có lý do rõ ràng. `const enum` được inline nên nhẹ, nhưng vướng `isolatedModules`/bundler nên thường tránh.

```ts
type Status = 'TODO' | 'IN_PROGRESS' | 'DONE';

const Role = { OWNER: 'OWNER', MEMBER: 'MEMBER' } as const;
type Role = typeof Role[keyof typeof Role]; // 'OWNER' | 'MEMBER'
```

</details>

---

## 3. Node.js core

### 3.1. Node.js là gì? Khác gì với JavaScript chạy trên trình duyệt?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Node.js là một runtime** để chạy JavaScript ngoài trình duyệt (server, CLI, tooling), xây trên engine **V8** (biên dịch JS) + thư viện **libuv** (event loop & thread pool cho I/O) + **bindings** nối JS ↔ C/C++.
- **JavaScript là ngôn ngữ**, còn **trình duyệt và Node.js là 2 môi trường thực thi (host) khác nhau**: cùng cú pháp JS nhưng cung cấp API/đối tượng toàn cục khác nhau.
- **Mô hình thực thi của Node**: single-threaded, non-blocking I/O, event-driven → hợp ứng dụng **I/O-bound** (API, DB, realtime); không hợp tác vụ CPU-bound nặng (chặn event loop).
- Khác biệt cốt lõi nằm ở **API môi trường** chứ không phải bản thân ngôn ngữ:

| Khía cạnh | Browser | Node.js |
|---|---|---|
| Global object | `window` / `globalThis` | `global` / `globalThis` |
| API đặc trưng | `document`, `DOM`, `fetch`, `localStorage` | `fs`, `http`, `process`, `Buffer`, `path` |
| Module mặc định | ESM (`import`) | CommonJS (`require`) + ESM |
| Truy cập | Bị **sandbox**, không đụng filesystem trực tiếp | Truy cập **hệ thống**: file, network, env, OS |
| Mục đích | UI/tương tác phía client | Server, tooling, script |

```js
// Chỉ chạy trong Node — không có trong browser
const fs = require('fs');
const data = fs.readFileSync('a.txt', 'utf8');
console.log(process.env.NODE_ENV);

// Chỉ chạy trong browser — không có trong Node
document.querySelector('#app').textContent = 'Hello';
```

- Tóm lại: **JS giống nhau, nhưng `window`/`document` là của browser; `fs`/`process`/`Buffer` là của Node.** Code chỉ chạy được ở đúng môi trường cung cấp API tương ứng.

</details>

---

### 3.2. Node single-threaded mà sao xử lý được hàng nghìn request đồng thời?

<details>
<summary>👉 Xem câu trả lời</summary>

- **JavaScript chạy trên một thread chính** — mọi code JS của bạn đều chạy tuần tự trên thread đó.
- Nhưng các tác vụ **I/O** (đọc file, query DB, gọi API mạng) **không tự tay** thread chính làm: chúng được giao xuống cho **OS** hoặc **thread pool của libuv** xử lý nền → thread chính **không bị chặn**, đi phục vụ việc khác ngay.
- Khi I/O xong, **callback được đẩy vào hàng đợi**; **Event Loop** lấy ra chạy mỗi khi thread chính rảnh.
- Nhờ vậy "single-threaded" chỉ áp cho phần chạy JS; phần chờ I/O chạy song song dưới nền → một thread vẫn ôm được hàng nghìn kết nối đang chờ.
- Mấu chốt: mô hình này hợp **I/O-bound** (web/API/DB). Với **CPU-bound** nặng (mã hoá, xử lý ảnh/video) thì code chạy ngay trên thread chính → **chặn Event Loop**, cả tiến trình đứng. Khi đó cần `Worker Threads` / `child_process` / hàng đợi job.

Ví dụ "đầu bếp": bỏ mì vào nồi (giao I/O cho libuv) rồi đi phục vụ khách khác, mì chín thì quay lại bưng ra (callback) — thay vì đứng nhìn nồi cho tới lúc chín.

```js
// I/O non-blocking: giao việc rồi trả quyền điều khiển ngay
fs.readFile('a.txt', (err, data) => { /* callback chạy SAU khi đọc xong */ });
console.log('B'); // in NGAY, không đợi file → thread chính rảnh phục vụ việc khác
```

</details>

---

### 3.3. Khi nào KHÔNG nên dùng Node.js?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Tác vụ CPU-bound nặng**: xử lý ảnh/video, mã hoá (encryption/hashing nặng), nén dữ liệu, tính toán khoa học, dựng PDF lớn, vòng lặp tính số khổng lồ.
- **Lý do cốt lõi**: JS chạy trên **một thread duy nhất** (single event loop). Một tác vụ tính toán nặng **chiếm dụng (block) event loop** → mọi request/tác vụ khác phải xếp hàng chờ → cả tiến trình "đứng hình", latency tăng vọt. Node mạnh ở **I/O-bound** (chờ DB, network, file) chứ không phải ở việc "cày" CPU.
- **Phân biệt**: I/O-bound = phần lớn thời gian là *chờ* (libuv lo nền, event loop rảnh) → Node rất hợp. CPU-bound = phần lớn thời gian là *tính* (giữ chặt thread) → Node yếu.
- **Cách xử lý nếu vẫn dùng Node**:
  - `worker_threads`: chạy phần tính nặng trên thread riêng trong cùng process → main loop vẫn phục vụ request khác (tốt nhất cho CPU-bound thuần JS).
  - `child_process` / `cluster`: tách ra process riêng hoặc gọi tool ngoài (vd `ffmpeg`).
  - **Job queue** (đẩy việc nặng ra worker xử lý bất đồng bộ) hoặc viết phần nóng bằng ngôn ngữ khác (Rust/Go/C++ qua native addon, microservice).

| Loại tải | Node phù hợp? |
|---|---|
| API/CRUD, realtime, proxy, BFF (I/O-bound) | ✅ Rất tốt |
| Tính toán nặng kéo dài (CPU-bound) | ❌ Tránh / phải offload |

```js
// ❌ CPU-bound chặn event loop — cả tiến trình đứng tới khi tính xong
function fib(n) { return n < 2 ? n : fib(n - 1) + fib(n - 2); }
app.get('/calc', (req, res) => res.send(fib(45))); // request khác phải chờ

// ✅ Offload sang worker thread — main loop vẫn phục vụ request khác
const { Worker } = require('node:worker_threads');
const w = new Worker('./fib-worker.js', { workerData: 45 });
w.once('message', r => res.send(r));
```

</details>

---

### 3.4. `package.json`: dependencies vs devDependencies, lock file, semver `^ ~`?

<details>
<summary>👉 Xem câu trả lời</summary>

- **`package.json`**: file khai báo (manifest) của một project Node — ghi `name`, `version`, các `scripts` (vd `build`, `test`, `start`) và danh sách thư viện mà project phụ thuộc.
- **`dependencies`**: thứ cần để **chạy ở runtime/production** (vd thư viện HTTP, ORM client...).
- **`devDependencies`**: chỉ cần lúc **phát triển hoặc build** — `typescript`, `jest`, `eslint`, bundler... Khi deploy production có thể bỏ qua nhóm này (`npm ci --omit=dev`) để image nhẹ hơn.
- **Lock file** (`package-lock.json` / `pnpm-lock.yaml` / `yarn.lock`): khoá **chính xác version** của mọi package, kể cả **package con (transitive)**, để mọi máy và CI cài ra **cùng một cây dependency** → tránh cảnh "máy mình chạy, máy người khác lỗi". Nhớ commit lock file vào Git.
- **Semver** dạng `MAJOR.MINOR.PATCH`:
  - `^1.2.3` → cập nhật trong cùng **MAJOR** (lấy được `1.9.0`, **không** lên `2.0.0`).
  - `~1.2.3` → chặt hơn, chỉ cập nhật **PATCH** (lấy được `1.2.9`, **không** lên `1.3.0`).
- Range (`^`/`~`) chỉ là giới hạn cho phép; nhờ có lock file nên cài lại vẫn ra **đúng version cũ**, chỉ khi chủ động `npm update`/`npm install <pkg>@latest` mới nhảy version.

| Ký hiệu | Ví dụ | Cho phép cập nhật | Chặn ở |
|---|---|---|---|
| `^` (caret) | `^1.2.3` | MINOR + PATCH | `< 2.0.0` |
| `~` (tilde) | `~1.2.3` | chỉ PATCH | `< 1.3.0` |
| (cố định) | `1.2.3` | không | đúng `1.2.3` |

> Lưu ý: với `0.x` thì `^0.2.3` được coi là không ổn định nên chỉ cho cập nhật PATCH (`< 0.3.0`), vì version `0.x` chưa cam kết tương thích.

</details>

---

### 3.5. EventEmitter và lập trình hướng sự kiện trong Node là gì?

<details>
<summary>👉 Xem câu trả lời</summary>

- **Event-driven**: một bên **phát** sự kiện (`emit`), các bên khác **lắng nghe** và phản ứng (`on`) — thay vì gọi hàm trực tiếp lẫn nhau → giảm coupling.
- `EventEmitter` là core class của Node (module `events`); nhiều object built-in kế thừa nó (vd `http.Server`, các `stream`).
- `on(event, fn)` đăng ký listener; `emit(event, ...args)` gọi **đồng bộ** mọi listener của event đó theo **đúng thứ tự đăng ký**.
- `once(event, fn)`: chạy 1 lần rồi tự gỡ; `off`/`removeListener`: gỡ thủ công.
- **Khi nào dùng**: tách side-effect (notification, logging, audit) khỏi luồng business logic chính; giao tiếp nội bộ giữa các module.
- **Rò rỉ listener**: cứ `on` thêm handler mà quên gỡ → số listener tăng dần, Node cảnh báo `MaxListenersExceededWarning` (mặc định >10/event). Khắc phục bằng `once` hoặc `off`/`removeListener`.
- **Lưu ý error**: sự kiện `'error'` mà không có listener nào → Node **ném và crash** process; nên luôn lắng `'error'`.

```js
const { EventEmitter } = require('events');
const bus = new EventEmitter();

bus.on('order.created', (order) => console.log('Notify:', order.id));
bus.once('order.created', (order) => console.log('Log 1 lần:', order.id));

bus.emit('order.created', { id: 'A-101' });
// Notify: A-101
// Log 1 lần: A-101

// emit lần 2: 'once' đã tự gỡ, chỉ còn 'on' chạy
bus.emit('order.created', { id: 'A-102' }); // Notify: A-102
```

</details>

---

## ✅ Checklist ôn nhanh

- [ ] **Async JS**: event loop, micro vs macrotask, thứ tự `đồng bộ → nextTick → Promise → setTimeout`; tuần tự vs song song (`Promise.all`).
- [ ] **JS core**: closure (stale closure), `this` & arrow function, callback → Promise → async/await.
- [ ] **Module & I/O**: CommonJS vs ES Modules, blocking vs non-blocking.
- [ ] **TypeScript**: generics, `type` vs `interface`, `unknown/any/never`, utility types, type guard, `satisfies`/`as const`, enum vs union.
- [ ] **Node core**: single-thread + event loop, khi nào KHÔNG dùng Node, `package.json`/semver, EventEmitter.

---

> 📌 *Nền tảng JS/TS/Node vững giúp bạn trả lời chắc các câu "vì sao", và là gốc để hiểu sâu cả React (frontend) lẫn NestJS (backend).* 🚀
