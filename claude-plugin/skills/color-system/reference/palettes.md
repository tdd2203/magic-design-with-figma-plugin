# Bảng màu khởi đầu theo loại sản phẩm

Đây là **điểm xuất phát, không phải đáp án**. Chỉ dùng khi file và DESIGN.md của dự án chưa có màu
thương hiệu. Có màu thương hiệu rồi thì dựng từ màu đó ([tokens-and-modes.md §4](tokens-and-modes.md)).
Chọn xong một hàng vẫn phải chạy đủ 8 bước kiểm ([checks.md](checks.md)). Chép bảng màu mà không kiểm
lại là một trong các lỗi bị cấm.

Mọi số contrast dưới đây được đo lại bằng helper ở [color-theory.md §7](color-theory.md), chuẩn
WCAG 2. Cột "Nút" là chữ trắng trên nút; cột "Chữ phụ" là chữ phụ trên nền trang của chính hàng đó.

## Bảng theo loại dự án

| Loại dự án | Neutral | Nền trang | Chữ phụ | Nút (chữ trắng) | Cạm bẫy chính |
|---|---|---|---|---|---|
| SaaS / dashboard | Lạnh, ám xanh | `#F8FAFC` | `#475569` (7,24) | `#2563EB` (5,17) | Sidebar nền đậm; bảng dữ liệu nhiều màu. Màu chỉ ở cột trạng thái |
| Fintech | Lạnh, rất ít chroma | `#F7F8FA` | `#475467` (7,23) | `#175CD3` (5,99) | Màu tăng/giảm là quy ước văn hoá (Trung Quốc, Đài Loan, Nhật: đỏ là tăng). Cho người dùng đổi; luôn kèm `+`/`−`. Chữ "tăng" dùng `#027A48` (5,09) |
| E-commerce | Gần trắng thuần: nền ám làm ảnh sản phẩm lệch màu | `#FFFFFF` | `#6B7280` (4,83) | `#C2410C` (5,18) | Mỗi màn hình một nút màu; đỏ giá khuyến mãi phải khác đỏ báo lỗi |
| Giáo dục | Ám theo hue primary | ám teal rất nhẹ | `#57534E` (7,63 trên trắng) | `#0F766E` (5,47) | Pastel chỉ làm nền, chữ lấy nấc 700–900 cùng hue; công cụ cho giáo viên thiết kế như SaaS |
| Y tế | Lạnh, sáng | `#F7FAFC` | `#4B5565` (7,19) | `#155E75` (7,27) | Đỏ **chỉ** cho nguy cấp; tránh lục-vàng; nhắm AAA 7:1 |
| Landing page | Tuỳ hero | `#FFFFFF`, hero `#0F172A` | — | `#C2410C` (5,18) | Ba CTA cùng màu thì coi như không có CTA; màu landing phải cùng hệ với sản phẩm bên trong |
| Blog / đọc dài | Không trắng thuần, không đen thuần | `#FDFCFA` | `#57534E` (7,44) | link `#1D4ED8` (6,54) · đã xem `#6D28D9` (6,93) | Contrast **quá cao** cũng mỏi: chữ thân bài nhắm 12–16:1 (`#1F1D1A` = 16,4); không tô màu tiêu đề |
| Luxury | Trầm, ít chroma | `#FAFAF9` | `#78716C` (4,59) | gần như không có | Vàng kim `#B8A06A` chỉ 2,43, chỉ dùng cho đường kẻ trang trí. Làm chữ thì phải là `#836C37` (4,83) |
| F&B | Kem ấm, **đúng ở đây** vì hue thương hiệu ấm | `#FDF8F0` | `#7A6A5F` (4,90) | `#B45309` (5,02) | Xanh dương làm giảm cảm giác ngon; thẻ chứa ảnh món giữ nền trắng |
| Trẻ em | Trắng | `#FFFFFF` | — | `#4F46E5` (6,29) | Màu tươi chỉ ở khối lớn bo tròn, không ở chữ nhỏ; chữ to không được miễn contrast |
| Dịch vụ công | Trung tính | `#FFFFFF` | — | `#1E40AF` (8,72) | AAA cho toàn bộ chữ; không có accent trang trí; phải đọc được ở chế độ tương phản cao của hệ điều hành |
| Game / streaming | Tối sâu, mặc định dark | `#0B0B14` | `#A0A0B8` (7,66) | `#7350F3` (5,04) | Neon chỉ ở diện tích nhỏ; glow không đặt lên chữ; hạ chroma 10–15% so với bản light |
| Bất động sản / du lịch | Ấm trung tính | `#FCFBF9` | `#6B6459` (5,65) | `#1E3A5F` (11,50) | Chữ đè ảnh cần lớp phủ tối 40–60%, đừng tin "ảnh này chắc đủ tối" |
| Mạng xã hội | Xám nhạt, giao diện gần đơn sắc | `#F6F7F8` | `#65676B` (5,28) | `#026AE3` (5,03) | Mọi màu trong khung giao diện đều giành chỗ với ảnh người dùng; primary chỉ ở nút và link |

## Những mã quen tay nhưng trượt

Các mã này hay được đề xuất cho đúng các loại dự án ở trên. Đo lại thì chúng trượt ngưỡng. Cột giữa là
chữ trắng trên mã đó, trừ dòng ghi khác:

| Mã | Đo lại | Thay bằng |
|---|---|---|
| `#EA580C` (cam e-commerce) | 3,56 | `#C2410C` (5,18) |
| `#F97316` (cam landing) | 2,80 | `#C2410C` (5,18) |
| `#0D9488` (teal giáo dục) | 3,74 | `#0F766E` (5,47) |
| `#039855` (chữ "tăng" fintech) | 3,51, đo khi nó làm chữ trên nền `#F7F8FA` | `#027A48` (5,09) |
| `#7C5CFF` (tím game) | 4,35 | `#7350F3` (5,04) |
| `#1877F2` (xanh mạng xã hội) | 4,23 | `#026AE3` (5,03) |
| `#0E7490` (y tế) | 5,36: đạt AA nhưng trượt AAA mà ngành này nên nhắm | `#155E75` (7,27) |
| `#1D4ED8` (dịch vụ công) | 6,70: trượt AAA | `#1E40AF` (8,72) |

Chữ 18–23px **thường** (không đậm) vẫn cần 4,5. Chỉ chữ từ 24px, hoặc từ 18,66px khi weight ≥ 700,
mới được hạ xuống 3:1.

## Ghi chú khi đo lại

- **Game / streaming:** `#0B0B14` (L ≈ 0,155) sâu hơn khoảng nền dark khuyên dùng (`#0F172A`–`#18181B`,
  L ≈ 0,21). Chấp nhận được khi màn hình chủ yếu là ảnh và video. Nếu có nhiều chữ đọc lâu thì nâng nền
  lên neutral-900 của hệ mặc định. Nút `#7350F3` trên nền này đạt 3,89, đủ 3:1 cho thành phần giao diện.
- **Luxury:** chữ phụ `#78716C` đạt 4,80 trên trắng nhưng chỉ 4,59 trên `#FAFAF9`. Vẫn đạt, nhưng sát
  ngưỡng; đặt lên nền tint thì phải đo lại.
- **Nền ấm (F&B, BĐS, blog):** chỉ đúng khi hue thương hiệu ấm. Primary lạnh mà nền kem là tự mâu thuẫn.

## Cộng thêm cho mobile (mọi loại)

- Chữ thường nhắm **5:1** thay vì 4,5, vì màn hình hay bị dùng ngoài nắng.
- Dark mode là **bắt buộc**, vì hệ điều hành có thể tự chuyển sang tối mà người dùng không để ý.
- Không có hover, nên trạng thái pressed phải lệch **≥ 2 nấc** ramp so với default. Hệ mặc định đã làm
  vậy: 600 → 800 ở Light, 400 → 200 ở Dark.

## Từ một hàng trong bảng sang Variables

1. `BRAND` = mã nút của hàng đó. Mã đã đạt cả chữ trắng lẫn nền subtle thì đặt `PIN_600` bằng chính mã
   đó để giữ nguyên ([tokens-and-modes.md §4](tokens-and-modes.md)).
2. Neutral tự ám theo hue của `BRAND`. Riêng hàng có neutral khác hue nút (luxury, F&B, blog) thì sau khi
   dựng, sửa giá trị các primitive `neutral/*` cho đúng tông, rồi đo lại cả bảng vai trò.
3. Muốn đúng mã nền trang của hàng đó (ví dụ `#FDF8F0`) thì sửa giá trị primitive `neutral/50`. Không
   sửa trực tiếp biến Semantic.
4. Chạy `audit_design {scope: "palette"}`, rồi làm 8 bước kiểm.
