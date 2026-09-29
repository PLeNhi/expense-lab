# Spec: Thêm khoản chi

## Trường dữ liệu

| Trường    | Kiểu                | Bắt buộc             | Ràng buộc                                                 |
| --------- | ------------------- | -------------------- | --------------------------------------------------------- |
| id        | uuid                | có                   | crypto.randomUUID()                                       |
| content   | string              | Có                   | trim, 1–100 ký tự                                         |
| category  | mã danh mục         | Có                   | 1 trong 9 mã bên dưới                                     |
| date      | YYYY-MM-DD          | Có                   | mặc định hôm nay (giờ máy); chỉ được chọn trong khoảng hôm nay − 7 ngày đến hôm nay + 7 ngày (cả 2 đầu đều hợp lệ) |
| amount    | số thập phân (đồng) | Có                   | 1.000 ≤ x ≤ 100.000.000.000; cho phép nhập phần thập phân (không bắt buộc số nguyên) |
| note      | string              | Khi date > hôm nay   | trim, tối đa 10.000 ký tự                                 |
| createdAt | date                | có (hệ thống tự tạo) | 2026-09-29T13:27:00.000Z                                  |

## Danh mục

| Mã        | Nhãn                 |
| --------- | -------------------- |
| rent      | Tiền nhà             |
| food      | Ăn uống              |
| love      | Thăm người yêu       |
| sport     | Thể thao             |
| utilities | Điện nước - internet |
| transport | Đi lại               |
| buy       | sinh hoạt - mua sắm  |
| debt      | trả nợ               |
| other     | Khác                 |

## Lưu trữ

- localStorage, key `expenses`
- Dữ liệu hỏng: Hiển thị lỗi thông báo cho người dùng — text chính xác: "Không còn dữ liệu nào tồn tại nữa, hãy làm lẹ 1 cái Database để lưu đi". Kèm nút "Xóa dữ liệu hỏng và bắt đầu lại" cạnh message lỗi — bấm vào thì xóa sạch dữ liệu hỏng trong storage, quay về trạng thái trống ("Chưa có khoản chi nào"), cho phép thêm khoản chi lại ngay mà không cần tự xóa localStorage bằng tay. Không hiển thị form/list khi đang ở trạng thái lỗi này.
- localStorage trống: Hiển thị "Chưa có khoản chi nào"

## Hành vi form

- Validate khi: Nếu nhấn nút Lưu sẽ kiểm tra Validate và hiển thị lỗi nếu vi phạm ở dưới từng input của nó và thể hiện rule của input đó.
- Sau khi lưu: Sau khi lưu thành công form reset, sau đó cho phép nhập thêm.

## Tiêu chí nghiệm thu

- Nhập amount = 999 → lỗi "Số tiền tối thiểu cần nhập là 1.000 đồng"
- Chọn ngày mai, bỏ trống note → lỗi "Nhập lý do chi tiêu trong tương lai để còn nhớ!"
- Nhập amount = 100.000.000.001 → lỗi "Số tiền tối đa có thể nhập là 100.000.000.000 đồng"
- localStorage hỏng -> thông báo ra màn hình " Không còn dữ liệu nào tồn tại nữa, hãy làm lẹ 1 cái Database để lưu đi"
- localStorage hỏng, bấm nút "Xóa dữ liệu hỏng và bắt đầu lại" → hết lỗi, hiện "Chưa có khoản chi nào", thêm khoản chi mới được ngay
- Content rỗng hoặc > 100 ký tự → lỗi "Nội dung chi tiêu phải từ 1 đến 100 ký tự"
- Category chưa chọn hoặc không hợp lệ → lỗi "Vui lòng chọn danh mục"
- Date ngoài khoảng hôm nay ± 7 ngày → lỗi "Ngày chi tiêu chỉ được chọn trong khoảng 7 ngày trước đến 7 ngày sau hôm nay"
- Note > 10.000 ký tự → lỗi "Ghi chú không được vượt quá 10.000 ký tự" (áp dụng luôn, kể cả khi note không bắt buộc)
- Amount để trống hoặc không phải số → lỗi "Vui lòng nhập số tiền"

## Trong phạm vi

- Danh sách nằm trong phạm vi của lần này: Sẽ tồn tại ở 2 nơi, 1 là trang list riêng, 2 là bên dưới form thêm để user thấy mình đã thêm những gì. Dùng chung 1 component list, tính năng giống hệt nhau ở cả 2 nơi.
- Cài thêm `react-router-dom@7.18.4` cho việc chuyển trang (`BrowserRouter`). Route `/` = trang form thêm + list bên dưới; route `/expenses` = trang list riêng.
- List sẽ hiển thị content, category, date (định dạng DD/MM/YYYY), amount (1.500.000 ₫), note. Sắp xếp theo mới nhất đến cũ nhất theo createdAt và thêm nút sort ở header cho tất cả các trường trên. Default là chỉ sort theo createdAt.
- Hành vi nút sort ở header: bấm lần đầu vào 1 cột → sort giảm dần (desc) theo cột đó; bấm lại cùng cột → đảo chiều asc/desc. Cột category sort theo nhãn tiếng Việt hiển thị, không theo mã.

## Ngoài phạm vi

- Sửa, xóa khoản chi (từng bản ghi riêng lẻ)
- Thống kê

Lưu ý: nút "Xóa dữ liệu hỏng và bắt đầu lại" ở mục Lưu trữ KHÔNG thuộc phạm vi "sửa, xóa khoản chi" bị loại trừ ở trên — đây là thao tác reset toàn bộ storage khi dữ liệu đã hỏng, không đọc được thành khoản chi hợp lệ nào để mà sửa/xóa từng cái.
