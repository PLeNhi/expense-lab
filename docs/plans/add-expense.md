# Plan: Tính năng "Thêm khoản chi"

## Context

Expense Lab hiện là project mới khởi tạo (Vite + React + TS + Vitest), chưa có bất kỳ tính năng nghiệp vụ nào — chỉ có boilerplate demo của Vite trong `App.tsx`. Đây là lần đầu triển khai theo spec `docs/specs/add-expense.md`: form thêm khoản chi + 2 nơi hiển thị danh sách (trang chủ và trang `/expenses`), theo đúng quy ước tách logic (`src/lib/`) khỏi UI (`src/components/`, `src/pages/`) trong CLAUDE.md. Nhiều điểm trong spec bỏ ngỏ (message lỗi, format hiển thị, cấu trúc thư mục pages) đã được hỏi và chốt trực tiếp với user — plan dưới đây phản ánh đúng các quyết định đó, không có phần nào là tự đoán.

## Quyết định đã chốt (áp dụng thẳng, không cần hỏi lại)

| Chủ đề | Quyết định |
|---|---|
| Router | `react-router-dom@7.18.4` (mới nhất theo `npm view`, cập nhật 2026-09-15), dùng `BrowserRouter` |
| Routes | `/` = form thêm + list bên dưới; `/expenses` = trang list riêng, đầy đủ |
| List component | Dùng chung 1 `ExpenseList` đầy đủ tính năng cho cả 2 nơi |
| Amount input | `<input type="number" step="any">`, nhập số trần, KHÔNG format khi gõ; cho phép thập phân (không bắt số nguyên) |
| Date hợp lệ | `[hôm nay − 7 ngày, hôm nay + 7 ngày]`, hai đầu inclusive |
| localStorage trống | Hiển thị `"Chưa có khoản chi nào"` |
| Sort header | Bấm lần đầu vào cột → desc; bấm lại cùng cột → đảo asc/desc. Mặc định ban đầu: desc theo `createdAt` |
| Sort theo category | Sort theo **nhãn hiển thị** tiếng Việt, không theo mã |
| Date hiển thị trong list | Format `DD/MM/YYYY` (lưu trữ vẫn `YYYY-MM-DD`) |
| Vị trí page component | Tạo `src/pages/` riêng (tách khỏi `src/components/` thuần UI) |
| Content lỗi (rỗng hoặc >100 ký tự) | `"Nội dung chi tiêu phải từ 1 đến 100 ký tự"` |
| Category lỗi | `"Vui lòng chọn danh mục"` |
| Date ngoài khoảng hợp lệ | `"Ngày chi tiêu chỉ được chọn trong khoảng 7 ngày trước đến 7 ngày sau hôm nay"` |
| Note quá 10.000 ký tự | `"Ghi chú không được vượt quá 10.000 ký tự"` — áp dụng **luôn**, kể cả khi note không bắt buộc |
| Amount rỗng/NaN | `"Vui lòng nhập số tiền"` |
| Amount < 1.000 | `"Số tiền tối thiểu cần nhập là 1.000 đồng"` (theo tiêu chí nghiệm thu) |
| Amount > 100.000.000.000 | `"Số tiền tối đa có thể nhập là 100.000.000.000 đồng"` (theo tiêu chí nghiệm thu) |
| Date tương lai, note rỗng | `"Nhập lý do chi tiêu trong tương lai để còn nhớ!"` (theo tiêu chí nghiệm thu) |
| localStorage hỏng | `"Không còn dữ liệu nào tồn tại nữa, hãy làm lẹ 1 cái Database để lưu đi"` (theo tiêu chí nghiệm thu) |
| localStorage hỏng — phục hồi | Không để người dùng kẹt vĩnh viễn. Kèm nút `"Xóa dữ liệu hỏng và bắt đầu lại"` cạnh message lỗi; bấm vào thì ghi đè storage thành mảng rỗng, quay về trạng thái "Chưa có khoản chi nào", cho thêm khoản chi bình thường. Không tính là tính năng "xóa khoản chi" (mục Ngoài phạm vi của spec) vì dữ liệu đang hỏng không đọc được thành khoản chi hợp lệ nào để mà sửa/xóa từng cái — đây là thao tác reset storage, không phải CRUD trên 1 bản ghi. |
| Tên file types | `src/lib/expense-types.ts` → đổi thành `src/lib/ExpenseTypes.ts` (PascalCase) để đúng luật đặt tên "Types/Interfaces: PascalCase" trong CLAUDE.md — file này chỉ chứa `type`/`interface`, không phải utility/helper function nên không áp dụng luật kebab-case |

## Cấu hình nền tảng cần sửa trước

- `vite.config.ts`: đổi import `defineConfig` từ `'vite'` sang `'vitest/config'`, thêm `test: { environment: 'jsdom', setupFiles: ['./src/test/setup.ts'] }`.
- Tạo `src/test/setup.ts`: `import '@testing-library/jest-dom/vitest';`
- `npm install react-router-dom@7.18.4`
- Lưu ý tsconfig: `erasableSyntaxOnly: true` → KHÔNG dùng `enum` cho `ExpenseCategory`, dùng union string literal. `verbatimModuleSyntax: true` → import chỉ dùng làm type phải viết `import type { ... }`.

## File sẽ tạo/sửa

**`src/lib/`** (thuần logic, không import React; test cạnh file):
- `ExpenseTypes.ts` — types (không có test file)
- `expense-categories.ts` + test
- `format-date.ts` + test
- `format-amount.ts` + test
- `validate-expense.ts` + test
- `create-expense.ts` + test
- `expense-storage.ts` + test
- `sort-expenses.ts` + test

**`src/hooks/`**:
- `useExpenses.ts` + test

**`src/components/`** (UI thuần, chỉ gọi lib):
- `ExpenseForm.tsx` + test
- `ExpenseList.tsx` + test

**`src/pages/`** (page-level, ghép component + hook):
- `HomePage.tsx` + test — route `/`
- `ExpensesPage.tsx` + test — route `/expenses`

**Sửa**:
- `vite.config.ts`, `src/App.tsx` (viết lại hoàn toàn: `BrowserRouter` + `Routes` + nav), `package.json`/`package-lock.json` (thêm dependency)

**Không đụng tới** (ngoài phạm vi task): `src/assets/*`, `src/App.css`, `src/index.css` giữ nguyên trừ khi build lỗi vì boilerplate cũ.

## Kiểu dữ liệu (`src/lib/ExpenseTypes.ts`)

```ts
export type ExpenseCategory =
  | 'rent' | 'food' | 'love' | 'sport' | 'utilities'
  | 'transport' | 'buy' | 'debt' | 'other';

export interface Expense {
  id: string;
  content: string;
  category: ExpenseCategory;
  date: string;        // YYYY-MM-DD
  amount: number;
  note?: string;        // undefined nếu không nhập
  createdAt: string;    // ISO string
}

export interface ExpenseFormValues {
  content: string;
  category: ExpenseCategory | '';
  date: string;
  amount: string;       // raw string từ input
  note: string;
}

export type ExpenseFormErrors = Partial<Record<keyof ExpenseFormValues, string>>;
export type SortField = 'content' | 'category' | 'date' | 'amount' | 'note' | 'createdAt';
export type SortDirection = 'asc' | 'desc';
```

`src/lib/expense-categories.ts`: `EXPENSE_CATEGORIES` (mảng 9 mã), `CATEGORY_LABELS` (map mã→nhãn, giữ nguyên chính tả spec kể cả chữ thường "sinh hoạt - mua sắm", "trả nợ"), `isValidCategory(value: string): value is ExpenseCategory`.

## Hàm `src/lib/` — chữ ký và hành vi

- `format-date.ts`:
  - `toDateInputValue(date: Date): string` — `YYYY-MM-DD` theo giờ máy local (không dùng `toISOString()`).
  - `addDays(date: Date, days: number): Date`
  - `formatDateDisplay(dateStr: string): string` — `YYYY-MM-DD` → `DD/MM/YYYY`, dùng khi render list.
- `format-amount.ts`: `formatAmount(amount: number): string` → `"1.500.000 ₫"`.
- `validate-expense.ts` — mỗi hàm `throw new Error('<message>')` nếu sai, không throw = hợp lệ:
  - `validateContent(content: string): void`
  - `validateCategory(category: string): void`
  - `validateAmount(amount: number): void`
  - `validateDate(date: string, referenceDate?: Date): void`
  - `validateNote(note: string, date: string, referenceDate?: Date): void`
  - `validateExpenseForm(values: ExpenseFormValues, referenceDate?: Date): ExpenseFormErrors` — hàm tổng hợp, bọc try/catch quanh từng validator field để trả về **map lỗi theo field** (phục vụ yêu cầu hiển thị lỗi dưới từng input), không dừng ở lỗi đầu tiên. Đây là **nguồn sự thật duy nhất** cho rule validate — mọi nơi cần validate (form UI lẫn `createExpense`) đều phải gọi qua hàm này, không tự gọi lại từng `validateX` riêng lẻ để tránh 2 luồng logic lệch nhau khi sửa rule.
  - `FIELD_VALIDATION_ORDER: (keyof ExpenseFormValues)[] = ['content', 'category', 'date', 'amount', 'note']` — thứ tự ưu tiên field, dùng để chọn 1 message duy nhất khi cần throw (xem `createExpense` bên dưới).
- `create-expense.ts`: `createExpense(values: ExpenseFormValues, referenceDate?: Date): Expense` — gọi `validateExpenseForm(values, referenceDate)`; nếu map lỗi khác rỗng, throw `Error(errors[field])` với `field` là phần tử đầu tiên trong `FIELD_VALIDATION_ORDER` có mặt trong map lỗi (giữ đúng hành vi cũ: throw 1 message duy nhất, ưu tiên content → category → date → amount → note khi nhiều field sai cùng lúc). **Không** gọi lại `validateContent`/`validateCategory`/... trực tiếp — tránh 2 đường validate tách biệt (form dùng `validateExpenseForm`, `createExpense` tự validate lại theo thứ tự riêng) có thể lệch nhau khi sau này sửa rule ở 1 chỗ mà quên chỗ kia. Nếu hợp lệ, trả `Expense` với `id: crypto.randomUUID()`, `createdAt: (referenceDate ?? new Date()).toISOString()`, `content`/`note` đã trim, `note` là `undefined` nếu rỗng.
- `expense-storage.ts` (key `'expenses'`):
  - `readExpenses(): Expense[]` — không có key → `[]`; JSON lỗi/không phải mảng/phần tử sai shape → throw message lỗi hỏng dữ liệu.
  - `saveExpenses(expenses: Expense[]): void`
  - `appendExpense(expense: Expense): Expense[]` — đọc, nối, lưu, trả mảng mới; propagate lỗi hỏng dữ liệu từ `readExpenses`.
  - `clearExpenses(): void` — ghi đè storage thành `[]` (gọi `saveExpenses([])`) mà KHÔNG đọc lại trước, vì mục đích là phục hồi khi dữ liệu đang hỏng (đọc lại sẽ throw lại). Dùng cho nút "Xóa dữ liệu hỏng và bắt đầu lại".
- `sort-expenses.ts`: `sortExpenses(expenses: Expense[], field: SortField, direction: SortDirection): Expense[]` — hàm thuần, không mutate; `amount` so sánh số học; `date`/`createdAt` so sánh chuỗi ISO; `content`/`note` dùng `localeCompare('vi')` (note rỗng/undefined = `''`); `category` sort theo `CATEGORY_LABELS[code]`.

## Component/Hook/Page

- `useExpenses()` (`src/hooks/useExpenses.ts`): load expenses từ storage khi mount (bắt lỗi hỏng dữ liệu vào `error` state), `addExpense(values)` gọi `createExpense` + `appendExpense`, throw lỗi ra ngoài để component tự bắt (không nuốt lỗi); `clearCorruptedData()` gọi `clearExpenses()` rồi `setExpenses([])` + `setError(null)`.
- `ExpenseForm` (props: `onAddExpense`): state `values`/`errors`/`submitError`; submit → `validateExpenseForm`, có lỗi thì hiển thị dưới từng input và dừng; hợp lệ thì gọi `onAddExpense`, bọc try/catch để hiển thị `submitError` nếu storage hỏng (form giữ nguyên input khi lỗi, chỉ reset khi thành công). `date` mặc định = hôm nay (`toDateInputValue`). `amount` dùng `type="number" step="any"`. `category` dùng `<select>` render từ `EXPENSE_CATEGORIES`/`CATEGORY_LABELS`.
- `ExpenseList` (props: `expenses`): state `sortField`/`sortDirection` (mặc định `createdAt`/`desc`), dùng `sortExpenses`; `expenses.length === 0` → hiển thị `"Chưa có khoản chi nào"`; render 5 cột (content, category label, date `DD/MM/YYYY`, amount đã format, note) đều có nút sort ở header.
- `HomePage` (route `/`): dùng `useExpenses`; nếu `error` → hiển thị message + nút "Xóa dữ liệu hỏng và bắt đầu lại" (`onClick={clearCorruptedData}`), không render `ExpenseForm`/`ExpenseList`; nếu không lỗi → `ExpenseForm` → `ExpenseList` bên dưới.
- `ExpensesPage` (route `/expenses`): dùng `useExpenses`; nếu `error` → message + nút phục hồi (cùng hành vi như trên); nếu không lỗi → `ExpenseList`.
- `App.tsx`: `BrowserRouter` + `nav` (link `/` và `/expenses`) + `Routes` (`/` → `HomePage`, `/expenses` → `ExpensesPage`), xoá toàn bộ boilerplate cũ.

## Test case theo hàm (TDD: viết test → chạy fail → implement → chạy pass)

- **format-date**: format đúng `YYYY-MM-DD`/`DD/MM/YYYY` theo local time (không qua UTC), zero-pad tháng/ngày <10, `addDays` cộng/trừ đúng kể cả rollover qua tháng.
- **format-amount**: `1500000→"1.500.000 ₫"`, `1000→"1.000 ₫"`, `100000000000→"100.000.000.000 ₫"`, `0→"0 ₫"`.
- **expense-categories**: đúng 9 mã, đúng 9 nhãn (khớp chính tả spec), `isValidCategory` đúng/sai.
- **validate-expense**:
  - Amount: `999`→throw min message; `1000`→ok (biên); `100000000000`→ok (biên); `100000000001`→throw max message; rỗng/NaN→throw `"Vui lòng nhập số tiền"`; có phần thập phân (vd `1500000.5`)→ok (không lỗi).
  - Content: 1 ký tự→ok; 100 ký tự→ok; rỗng/chỉ khoảng trắng/101 ký tự→throw message content.
  - Category: mỗi 1/9 mã→ok; mã lạ/rỗng→throw message category.
  - Date (dùng `referenceDate` cố định, không phụ thuộc đồng hồ máy): hôm nay→ok; hôm nay−7→ok (biên); hôm nay−8→throw; hôm nay+7→ok (biên); hôm nay+8→throw; rỗng→throw. Tất cả throw dùng đúng message date-range đã chốt.
  - Note: ngày mai + note rỗng/chỉ khoảng trắng→throw message tương lai; ngày mai + note có nội dung→ok; hôm nay/hôm qua + note rỗng→ok; note đúng 10.000 ký tự→ok (biên); 10.001 ký tự→throw message note-length (áp dụng cả khi note không bắt buộc).
  - `validateExpenseForm`: hợp lệ toàn bộ→`{}`; 1 field sai→đúng 1 key; **nhiều field sai cùng lúc** (vd content rỗng + amount=999)→trả về đủ cả 2 key lỗi.
- **create-expense**: input hợp lệ→`Expense` đúng field, `id` dạng UUID, `createdAt` đúng `referenceDate.toISOString()`; content/note đã trim; note rỗng→`undefined`; 1 field sai→throw đúng message tương ứng field đó (khớp message của `validateExpenseForm`); **nhiều field sai cùng lúc** (vd content rỗng + amount=999)→throw đúng message của field đứng trước theo `FIELD_VALIDATION_ORDER` (content), không phải message của amount; test bổ sung xác nhận `createExpense` không throw ra message khác với message mà `validateExpenseForm` trả về cho cùng input (đảm bảo 2 hàm không lệch rule — mock/spy `validateExpenseForm` để xác nhận `createExpense` thực sự gọi qua nó thay vì tự validate lại).
- **expense-storage** (jsdom localStorage thật, `beforeEach` clear): không có key→`[]`; `'[]'`→`[]`; JSON hợp lệ 2 phần tử→trả đúng; `'not-json{'`→throw message hỏng dữ liệu; không phải mảng→throw; phần tử sai shape/category lạ→throw; `saveExpenses`→ghi đúng JSON; `appendExpense` rỗng/có sẵn dữ liệu/khi storage hỏng (propagate throw); `clearExpenses()` khi storage đang hỏng (`'not-json{'`) → không throw, ghi thành công, sau đó `readExpenses()` trả `[]`.
- **sort-expenses**: mảng rỗng→`[]`; sort đúng theo từng field (createdAt/amount/date/content/note/category) cả asc/desc; category sort theo label; note undefined coi như `''`; không mutate input gốc; toggle hướng khi gọi lại cùng field (logic toggle nằm ở component, nhưng hàm sort bản thân phải thuần theo tham số truyền vào).
- **ExpenseForm** (RTL): date mặc định = hôm nay; amount=999 submit→hiện đúng message dưới input amount, `onAddExpense` không được gọi; date=mai+note rỗng submit→hiện đúng message dưới note; amount=100000000001→hiện đúng message max; input hợp lệ→`onAddExpense` gọi đúng 1 lần, form reset; `onAddExpense` throw (mô phỏng storage hỏng)→hiện lỗi, form KHÔNG reset.
- **ExpenseList** (RTL): rỗng→`"Chưa có khoản chi nào"`; mặc định sort desc theo createdAt; bấm header amount→sort desc rồi bấm lại→asc; amount/date hiển thị đã format; category hiển thị nhãn.
- **HomePage/ExpensesPage** (integration, localStorage thật, clear mỗi test): rỗng→đúng UI; thêm hợp lệ→xuất hiện ngay trong list (HomePage không điều hướng); storage hỏng→hiện đúng message lỗi + nút "Xóa dữ liệu hỏng và bắt đầu lại", không crash, không render form/list; bấm nút phục hồi→hết lỗi, hiện "Chưa có khoản chi nào", HomePage cho nhập lại được ngay (không phải reload trang).
- **App** (smoke test routing, khuyến nghị): mặc định ở `/` thấy form; click link `/expenses`→chỉ thấy list.

## Mốc commit (chia làm 2 lần commit riêng)

Lý do chia: tổng cộng ~14 file code + 13 file test, vượt xa ngưỡng ~400 dòng/commit. Mỗi mốc verify độc lập (test + typecheck + lint) trước khi commit, rồi mới sang mốc tiếp theo.

- **Mốc 1 — `src/lib/` (logic thuần, test độc lập, không cần React/DOM)**: bước 1 đến 8 bên dưới. Sau mốc này: `npm test` chỉ chạy test của `src/lib/` (chưa có component/hook nào), `npx tsc --noEmit` và `npm run lint` clean. Commit riêng, báo "xong Mốc 1" kèm output test/typecheck thật trước khi sang Mốc 2.
- **Mốc 2 — hook + component + page + router**: bước 9 đến 14. Phụ thuộc vào toàn bộ Mốc 1. Sau mốc này chạy lại `npm test`/`npx tsc --noEmit`/`npm run lint` cho toàn bộ project, cộng bước 15 (verify thủ công qua `npm run dev`).

## Các bước thực hiện theo thứ tự

### Mốc 1 — `src/lib/`

1. Cấu hình nền tảng: `npm install react-router-dom@7.18.4`, sửa `vite.config.ts`, tạo `src/test/setup.ts`, chạy `npm test` xác nhận cấu hình không lỗi.
2. `ExpenseTypes.ts` + `expense-categories.ts` (test trước → fail → implement → pass).
3. `format-date.ts` (test trước → fail → implement → pass).
4. `format-amount.ts` (test trước → fail → implement → pass).
5. `validate-expense.ts` (test đầy đủ theo bảng trên → fail → implement từng validator + aggregator → pass).
6. `create-expense.ts` (test → fail → implement → pass).
7. `expense-storage.ts` (test → fail → implement → pass, gồm cả `clearExpenses`).
8. `sort-expenses.ts` (test → fail → implement → pass).
   - **Chốt Mốc 1**: `npm test`, `npx tsc --noEmit`, `npm run lint` — báo output thật, chờ duyệt trước khi sang Mốc 2.

### Mốc 2 — hook + component + page + router

9. `useExpenses.ts` (test bằng `renderHook` → fail → implement → pass, gồm `clearCorruptedData`).
10. `ExpenseForm.tsx` (test → fail → implement → pass).
11. `ExpenseList.tsx` (test → fail → implement → pass).
12. `HomePage.tsx` (test → fail → implement → pass, gồm nút phục hồi khi storage hỏng).
13. `ExpensesPage.tsx` (test → fail → implement → pass, gồm nút phục hồi khi storage hỏng).
14. `App.tsx` viết lại, wiring Router, xoá boilerplate.
15. Verify cuối: `npm test` (toàn bộ pass), `npx tsc --noEmit` (clean, chú ý `import type`), `npm run lint` (clean), `grep -rn "console.log" src` (rỗng), `npm run dev` chạy thử thủ công cả 2 route, form reset sau khi lưu, sort header hoạt động đúng, nút "Xóa dữ liệu hỏng và bắt đầu lại" hoạt động đúng khi giả lập storage hỏng.

## Verification

- Mốc 1: `npm test` (chỉ có test `src/lib/`), `npx tsc --noEmit`, `npm run lint` — đều clean, output thật.
- Mốc 2: `npm test` (toàn bộ, gồm cả component/integration test) — output đầy đủ tất cả pass, đặc biệt các test khớp chính xác message lỗi trong "Quyết định đã chốt". `npx tsc --noEmit`, `npm run lint` — không lỗi.
- Chạy `npm run dev`, thử tay: nhập amount=999 → thấy đúng lỗi; chọn ngày mai bỏ trống note → thấy đúng lỗi; nhập hợp lệ → form reset, khoản chi mới hiện ngay trong list dưới form; qua `/expenses` → thấy đúng khoản chi vừa thêm, bấm nút sort từng cột hoạt động đúng (desc lần đầu, đảo chiều lần 2); giả lập storage hỏng (set trực tiếp giá trị JSON sai trong DevTools) → thấy đúng lỗi + nút phục hồi, bấm nút → hết lỗi, thêm khoản chi lại được ngay.
