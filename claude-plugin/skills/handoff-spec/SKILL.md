---
name: handoff-spec
description: 'Viết tài liệu bàn giao cho dev từ một màn hình trong file Figma đang mở: bố cục, token màu và chữ, component và props, trạng thái, tương tác, co giãn, trường hợp biên, trợ năng. Số lấy thẳng từ file, không đoán. Dùng khi người dùng nói "bàn giao", "handoff", "viết spec cho dev", "màn này sẵn sàng cho dev chưa" với file Figma nối qua plugin Magic Design with Figma.'
argument-hint: "<tên frame, để trống thì dùng layer đang chọn>"
---

# Bàn giao cho dev

Đọc một màn hình trong Figma rồi viết spec để dev dựng đúng mà không phải hỏi lại.

## Cách dùng

```
/handoff-spec                              bàn giao frame đang chọn
/handoff-spec Đăng nhập                    tìm frame tên "Đăng nhập"
/handoff-spec Đăng nhập, React + Tailwind  thêm công nghệ để ghi chú cho hợp code
```

## Các bước

1. **Tìm frame.** `get_context`. Có tên thì tìm frame trùng tên trên page, không có thì dùng layer
   đang chọn. Không rõ là frame nào thì hỏi.
2. **Đọc số liệu.** `inspect_nodes` (depth 3) để lấy bố cục. Chạy `handoffData` trong
   [reference/code.md](reference/code.md) để lấy token, style, component, tương tác prototype và ghi
   chú có sẵn. `screenshot` frame để nhìn.
3. **Tìm các bản đi kèm.** Tìm trên page các frame cùng tên: `… / Empty`, `… / Loading`,
   `… / Error`, `… · Mobile`. Thiếu trạng thái hay thiếu bản mobile thì ghi vào mục "Còn thiếu".
   Đừng tự vẽ ra.
4. **Đo trước khi giao.** Chạy `audit_design` với `node_id` của frame. Còn FAIL thì báo người dùng
   trước. Spec vẫn viết được, nhưng phải ghi rõ lỗi đang còn.
5. **Viết spec** theo mẫu dưới. Ghi **tên token** trước, giá trị sau (`color/bg/surface · #FFFFFF`).
   Chỗ nào file không nói thì ghi "chưa quyết" và hỏi, đừng tự điền.
6. **Hỏi nơi giao.** Mặc định trả spec trong chat. Nếu người dùng muốn:
   - lưu thành file `handoff/<tên-màn>.md` trong thư mục dự án;
   - gắn ghi chú lên từng layer (annotation, dev đọc trong Dev Mode) và đánh dấu **Ready for dev**:
     làm theo [../design-workflow/reference/handoff.md](../design-workflow/reference/handoff.md)
     mục 3 và 4. Chỉ đánh dấu Ready for dev khi người dùng bảo.

## Mẫu spec

```markdown
## Bàn giao: <tên màn> (<id>)
Frame 1440×900 · page Screens · audit_design: 0 FAIL, 2 WARN (lý do ở cuối)

### Tổng quan
Màn này để làm gì, ai dùng, việc nào làm nhiều nhất.

### Bố cục
Khung, cột, khoảng cách chính (theo token). Phần nào giãn, phần nào cố định.

### Token đang dùng
| Token | Light / Dark | Dùng cho |
|---|---|---|

### Component
| Component | Variant và props | Ghi chú |
|---|---|---|

### Trạng thái và tương tác
| Thành phần | Trạng thái hoặc thao tác | Kết quả |
|---|---|---|
Chuyển động: trigger, thời lượng, easing (lấy từ prototype).

### Co giãn
| Bề rộng | Thay đổi |
|---|---|

### Nội dung và trường hợp biên
Giới hạn ký tự, cắt chữ, khi trống, đang tải, lỗi, dữ liệu rất dài.

### Trợ năng
Thứ tự focus, tên cho nút chỉ có icon, alt cho ảnh, phím tắt.

### Còn thiếu / chưa quyết
```

## Lưu ý

- Chỉ ghi thứ đọc được từ file hoặc người dùng nói. Tự suy ra thì ghi rõ là "đề xuất".
- Không đặc tả lại phần ruột của instance, vì dev lấy phần đó từ component. Chỉ ghi variant và
  props đang dùng.
- Màu và chữ viết cứng (chưa gắn biến, chưa gắn style) là chỗ dev phải đoán. `audit_design` liệt kê
  chúng (`mau-khong-bien`, `chu-khong-style`): đưa vào mục "Còn thiếu".
- Người dùng cho biết tên token trong code thì khai cho biến, để Dev Mode hiện đúng tên (xem
  handoff.md mục 3).
- Muốn kiểm kỹ phần trợ năng trước khi giao thì dùng skill `a11y-audit`.
