# Hướng dẫn cấu trúc file Excel cho trợ lý quản lý tài chính cá nhân

Tài liệu này định nghĩa các sheet và trường dữ liệu cần thiết trong file Excel để MCP server đọc/ghi đúng cách.

---

## 1. Sheet `Transactions` (giao dịch) — bắt buộc

Đây là sheet chính, mỗi dòng là một giao dịch.

| Cột | Tên trường | Kiểu dữ liệu | Bắt buộc | Ghi chú |
|---|---|---|---|---|
| A | `id` | Số nguyên (tự tăng) | Có | Khóa duy nhất để update/delete chính xác |
| B | `date` | Ngày (YYYY-MM-DD) | Có | Ngày phát sinh giao dịch |
| C | `type` | Text: `income` / `expense` | Có | Phân biệt thu vs chi |
| D | `amount` | Số | Có | Luôn dương; dấu +/- suy ra từ `type` |
| E | `category` | Text | Có | Ví dụ: Ăn uống, Di chuyển, Lương, Giải trí... |
| F | `subcategory` | Text | Không | Chi tiết hơn, ví dụ: "Ăn uống > Cà phê" |
| G | `note` | Text | Không | Mô tả tự do, dùng để suy luận category tự động |
| H | `payment_method` | Text | Không | Tiền mặt / Thẻ / Chuyển khoản / Ví điện tử |
| I | `account` | Text | Không | Tên tài khoản/ nguồn tiền (nếu quản lý nhiều tài khoản) |
| J | `tags` | Text (phân cách bởi dấu phẩy) | Không | Gắn nhãn linh hoạt, ví dụ: "du lịch,gia đình" |
| K | `is_recurring` | Boolean (TRUE/FALSE) | Không | Giao dịch định kỳ (tiền nhà, internet...) |
| L | `created_at` | Datetime | Không | Thời điểm ghi vào hệ thống (tự động, để audit) |

**Quy tắc:**
- `id` tăng dần, không được trùng — dùng làm khóa cho `update_transaction`/`delete_transaction`.
- `amount` luôn là số dương; không nhập số âm để tránh sai logic tổng hợp.
- Nếu `category` để trống, tool có thể tự gợi ý dựa trên `note`.

---

## 2. Sheet `Categories` (danh mục) — khuyến nghị

Giúp chuẩn hóa category, tránh gõ tự do gây sai chính tả/trùng lặp.

| Cột | Tên trường | Kiểu dữ liệu | Ghi chú |
|---|---|---|---|
| A | `category_name` | Text | Tên category chuẩn |
| B | `type` | `income` / `expense` | Category này thuộc nhóm thu hay chi |
| C | `parent_category` | Text | Nếu có phân cấp (category cha) |
| D | `icon_color` | Text | Tùy chọn, phục vụ hiển thị nếu sau này làm dashboard |

---

## 3. Sheet `Budget` (ngân sách) — khuyến nghị

Dùng cho tool `budget_check`.

| Cột | Tên trường | Kiểu dữ liệu | Ghi chú |
|---|---|---|---|
| A | `category` | Text | Phải khớp với `category` ở sheet Transactions |
| B | `period` | Text: `monthly` / `yearly` | Chu kỳ áp dụng hạn mức |
| C | `limit_amount` | Số | Hạn mức chi tiêu cho category đó |
| D | `start_date` | Ngày | Áp dụng từ ngày nào (nếu hạn mức thay đổi theo thời gian) |
| E | `note` | Text | Ghi chú thêm |

---

## 4. Sheet `Accounts` (tài khoản/nguồn tiền) — tùy chọn

Nếu người dùng quản lý nhiều nguồn tiền (tiền mặt, ngân hàng, ví điện tử).

| Cột | Tên trường | Kiểu dữ liệu | Ghi chú |
|---|---|---|---|
| A | `account_name` | Text | Ví dụ: "Vietcombank", "Tiền mặt", "Momo" |
| B | `account_type` | Text | Ngân hàng / Tiền mặt / Ví điện tử / Thẻ tín dụng |
| C | `initial_balance` | Số | Số dư ban đầu khi bắt đầu theo dõi |
| D | `currency` | Text | Mặc định VND |

---

## 5. Sheet `Report` (báo cáo tổng hợp) — do hệ thống tự sinh

Sheet này **không** để người dùng nhập tay — chỉ do tool `write_report_sheet` ghi ra, tránh việc phân tích ghi đè dữ liệu gốc.

| Cột | Tên trường | Ghi chú |
|---|---|---|
| A | `period` | Tháng/năm của báo cáo |
| B | `category` | Category được tổng hợp |
| C | `total_amount` | Tổng tiền trong kỳ |
| D | `percent_of_total` | % trên tổng chi tiêu kỳ đó |
| E | `budget_status` | ok / gần vượt / vượt (nếu có sheet Budget) |
| F | `generated_at` | Thời điểm sinh báo cáo |

---

## 6. Quy ước đặt tên & định dạng chung

- **Ngày:** luôn dùng format `YYYY-MM-DD` để tránh lỗi khác biệt định dạng theo vùng (dd/mm vs mm/dd).
- **Số tiền:** không chứa dấu phẩy phân cách nghìn trong ô dữ liệu gốc (chỉ format hiển thị, giá trị thực là number thuần).
- **Text tự do** (`note`, `tags`): tránh ký tự đặc biệt gây lỗi khi đọc bằng thư viện Excel.
- **Header row:** luôn nằm ở dòng 1, không để trống dòng nào phía trên.
- **Không merge cells** trong vùng dữ liệu (Excel merge cell dễ gây lỗi khi đọc bằng pandas/openpyxl).
- Mỗi sheet nên có tên cố định (`Transactions`, `Categories`, `Budget`, `Accounts`, `Report`) để tool gọi đúng sheet mà không cần hỏi lại người dùng.

---

## 7. Trường tối thiểu nếu muốn làm đơn giản (MVP)

Nếu chưa muốn làm đầy đủ ngay, chỉ cần sheet `Transactions` với 5 cột bắt buộc:

```
date | type | amount | category | note
```

Các sheet `Categories`, `Budget`, `Accounts` có thể bổ sung dần khi cần tính năng nâng cao (kiểm tra ngân sách, quản lý đa tài khoản).
