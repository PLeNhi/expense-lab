# Expense Lab

App ghi chi tiêu cá nhân. Vite + React + TypeScript, test bằng Vitest.

## Lệnh

- Dev: `npm run dev`
- Test toàn bộ: `npm test` · Test 1 file: `npx vitest run <path>`
- Typecheck: `npx tsc --noEmit` · Lint: `npm run lint`

## Cấu trúc

- `src/lib/` — logic thuần (tính toán, validate), KHÔNG import React
- `src/components/` — UI, chỉ gọi hàm từ `src/lib/`, không chứa logic nghiệp vụ
- Test đặt cạnh file: `foo.ts` → `foo.test.ts`

## Quy ước

- Tiền: số nguyên, đơn vị đồng. KHÔNG dùng float
- Đặt tên file theo quy ước
  - PascalCase: Dùng cho Component, Class, Interface, Type. Tên file component phải trùng khớp với tên component bên trong.
  - camelCase: Dùng cho Function, Hook, Variable, Utility/Helper file.
  - Custom Hooks (React): Dùng camelCase và bắt đầu bằng tiền tố use.
  - kebab-case: Dùng cho Folder, CSS/SCSS file.
  - UPPERCASE: Dùng cho Constants (Hằng số) và biến môi trường.
  - Utility / Helper functions: Dùng kebab-case phản ánh đúng chức năng
  - Types / Interfaces: Dùng PascalCase kèm hậu tố hoặc nằm chung
- Lỗi:
  - Hàm trong `src/lib/` validate input, sai thì `throw new Error('<message tiếng Việt, cụ thể>')`
  - Component bắt lỗi và hiển thị message cho người dùng, không nuốt lỗi
  - Không để lại `console.log` trong code đã commit
- Không thêm thư viện khi chưa hỏi. Khi đề xuất, phải kèm:
  - Lý do không tự viết được bằng code hiện có
  - Kết quả `npm view <tên> version time.modified` (bản mới nhất và ngày cập nhật)

## Quy trình

- Task sửa từ 2 file trở lên: lập plan trước, chờ tôi duyệt rồi mới code
- Viết test trước, chạy xác nhận fail, rồi mới implement
- KHÔNG sửa, xóa hay skip test để cho pass
- Chỉ sửa file liên quan task; muốn sửa file khác thì hỏi trước
- Không thêm thư viện khi chưa hỏi
- Báo "xong" phải kèm output thật của test và typecheck

## Gotchas

- <để trống, thêm dần khi Claude mắc lỗi lặp lại>
