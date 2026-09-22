---
name: a11y-audit
description: 'Kiểm trợ năng (accessibility, a11y) của một màn hình trong file Figma đang mở theo WCAG 2.2 mức AA: độ tương phản, vùng bấm, nhãn ô nhập, tên cho nút chỉ có icon, alt cho ảnh, thứ tự focus, lỗi có chữ đi kèm. Máy đo phần đo được, phần còn lại soi từng mục, báo cáo theo mã WCAG, và nếu người dùng muốn thì ghi chú trợ năng lên layer cho dev. Dùng khi người dùng nói "kiểm a11y", "kiểm trợ năng", "đo contrast màn này", "có đạt WCAG không", "người khiếm thị dùng được không" với file Figma nối qua plugin Magic Design with Figma. Không dùng cho ảnh chụp hay link Figma không mở qua plugin.'
argument-hint: "<tên frame, để trống thì dùng layer đang chọn>"
---

# Kiểm trợ năng

Trả lời câu hỏi: người nhìn kém, người mù màu, người chỉ dùng bàn phím hay trình đọc màn hình có
dùng được màn này không.

## Cách dùng

```
/a11y-audit               kiểm frame đang chọn
/a11y-audit Thanh toán    kiểm frame "Thanh toán"
```

## Các bước

1. `get_context`, xác định frame. Không rõ là frame nào thì hỏi.
2. **Máy đo.** Chạy `audit_design` với `node_id` của frame. File có biến màu local thì chạy thêm
   `scope: "palette"`. Trong kết quả, giữ các luật có ở bảng *Máy đo được*; luật khác (khoảng cách,
   biến) thuộc `design-audit`.
3. **Tìm chỗ cần tên.** Chạy `a11yTargets` trong [reference/code.md](reference/code.md): nó trả về
   danh sách ảnh, nút chỉ có icon và ô nhập, kèm ghi chú đã có.
4. **Soi bằng mắt** (`screenshot`) các mục trong bảng *Phải soi*.
5. **Báo cáo** theo mẫu. Mỗi phát hiện có chỗ (tên layer + id), mã WCAG, mức nặng nhẹ và cách sửa.
6. **Hỏi rồi mới làm tiếp.** Hai việc người dùng có thể chọn:
   - sửa lỗi đo được (contrast, vùng bấm): cách sửa ở skill `design-audit`, đo lại sau khi sửa;
   - ghi chú trợ năng lên layer cho dev (thứ tự focus, tên đọc lên, alt) bằng `addA11yNotes`. Ghi
     chú nằm trong nhóm annotation tên "Tiếp cận" (nghĩa là trợ năng), cùng tên nhóm mà các skill khác
     của plugin đang dùng.

## Máy đo được (`audit_design`)

| Mã luật | WCAG 2.2 |
|---|---|
| `tuong-phan-chu`, `bang-mau-tuong-phan` | 1.4.3 Tương phản chữ: 4,5:1, chữ lớn 3:1 |
| `ranh-gioi-o-nhap`, `ky-hieu-mo` | 1.4.11 Tương phản của viền ô nhập, icon, ký hiệu mang nghĩa: 3:1 |
| `vung-bam-nho` | 2.5.8 Vùng bấm tối thiểu 24×24 |
| `vung-bam-mobile` | 2.5.5 Vùng bấm 44×44 (mức AAA, chỉ là khuyến nghị) |
| `chu-khong-do-duoc` (SKIP) | 1.4.3 Chữ trên ảnh hoặc gradient: máy không đo được, phải soi |
| `mu-mau-trung-nhau` (scope `palette`) | Gợi ý, liên quan 1.4.1: hai màu trạng thái trông như nhau với người mù màu |

## Phải soi (máy không đo)

| WCAG 2.2 | Hỏi gì trên Figma |
|---|---|
| 1.1.1 Nội dung không phải chữ | Ảnh mang nghĩa có alt chưa? Ảnh trang trí đã ghi "trang trí" chưa? |
| 1.3.1 Thông tin và cấu trúc | Tiêu đề, nhóm, bảng có rõ cấu trúc, hay chỉ khác nhau ở cỡ chữ? |
| 1.4.1 Dùng màu | Lỗi, trạng thái, link có thêm icon, chữ hay gạch chân, không chỉ đổi màu? |
| 1.4.4, 1.4.10 Phóng chữ, bề rộng hẹp | Có bản 320px chưa? Phóng chữ 200% có vỡ bố cục không? |
| 2.1.1 Bàn phím | Thao tác kéo thả, hover mới hiện có cách làm bằng bàn phím không? |
| 2.4.3 Thứ tự focus | Thứ tự Tab có đi theo thứ tự đọc không? |
| 2.4.7 Focus nhìn thấy được | Component có variant Focus với vòng focus rõ không? |
| 2.4.11 Focus không bị che | Phần tử đang focus có bị header dính, thanh dưới hay cookie banner che hết không? |
| 2.5.7 Kéo thả | Thao tác kéo (sắp xếp, thanh trượt) có cách làm bằng một cú bấm không (nút lên/xuống, ô nhập số)? |
| 3.3.1 Báo lỗi | Lỗi có câu nói rõ lỗi gì, hay chỉ có viền đỏ? |
| 3.3.2 Nhãn | Ô nhập có nhãn thật, hay chỉ có placeholder? |
| 3.3.8 Đăng nhập dễ | Màn đăng nhập cho dán mật khẩu, dùng trình quản lý mật khẩu, không bắt giải đố hay gõ lại mã từ ảnh? |
| 4.1.2 Tên, vai trò, giá trị | Nút chỉ có icon đã có tên đọc lên chưa ("Đóng", "Xoá đơn hàng")? |

**Mức nặng nhẹ:** **Nặng** là chặn hẳn một nhóm người (chữ không đọc được, nút icon không tên, ô
nhập không nhãn). **Vừa** là làm khó (focus không rõ, lỗi chỉ báo bằng màu). **Nhẹ** là nên sửa
(vùng bấm trên mobile dưới 44). FAIL của máy thường là Nặng hoặc Vừa, WARN thường là Vừa hoặc Nhẹ;
SKIP chưa có mức cho tới khi soi xong.

## Mẫu báo cáo

```markdown
## Trợ năng: <tên màn> (<id>) · WCAG 2.2 AA
Máy đo: 2 FAIL · 1 WARN · 3 SKIP. Soi tay: 4 mục.

| # | Chỗ | WCAG | Mức | Vấn đề | Sửa |
|---|---|---|---|---|---|
| 1 | Button/Close (12:40) | 4.1.2 | Nặng | Nút chỉ có icon, chưa có tên | Ghi chú: đọc là "Đóng" |

### Thứ tự focus đề xuất
1. Email → 2. Mật khẩu → 3. Quên mật khẩu → 4. Đăng nhập

### Bàn phím
| Thành phần | Tab | Enter / Space | Esc | Mũi tên |
|---|---|---|---|---|

### Chưa kiểm được
Trình đọc màn hình thật, phóng 200%, chuyển động: cần thử trên bản chạy thật.
```

## Lưu ý

- Kiểm trên Figma mới là nửa đầu. Bàn phím và trình đọc màn hình phải thử lại trên sản phẩm chạy
  thật. Luôn ghi điều này ở cuối báo cáo.
- Layer disabled được miễn tương phản (WCAG cho phép), nhưng vẫn phải nhìn thấy là nó có ở đó.
- `audit_design` và `a11yTargets` nhận ra nút và ô nhập qua **tên layer**. Layer tên `Frame 12` sẽ bị
  bỏ sót, nên đổi tên theo vai trò trước khi kiểm (`Button/Close`, `Input/Email`).
- SKIP là "chưa đo được", không phải "đạt".
- Muốn chấm cả thiết kế (thứ bậc, bố cục, độ dễ dùng) thì dùng skill `design-audit`.
