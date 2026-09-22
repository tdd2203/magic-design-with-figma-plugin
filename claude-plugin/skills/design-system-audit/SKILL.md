---
name: design-system-audit
description: 'Kiểm hệ thiết kế của cả file Figma đang mở: biến màu và khoảng cách, text style, component (đặt tên, trùng lặp, thiếu trạng thái, thiếu mô tả), và bao nhiêu phần trên màn hình đã dùng biến thay cho giá trị viết cứng. Cũng viết tài liệu cho một component. Dùng khi người dùng nói "kiểm design system", "hệ thiết kế ổn chưa", "viết tài liệu cho component" với file Figma nối qua plugin Magic Design with Figma.'
argument-hint: "[tài liệu <tên component>]"
---

# Kiểm hệ thiết kế

Nhìn cả file chứ không nhìn một màn: biến, style và component có gọn, đủ và được dùng thật không.

## Cách dùng

```
/design-system-audit                      kiểm cả file, báo cáo và việc nên làm
/design-system-audit tài liệu Button      viết mô tả cho component Button, gắn vào Figma
```

Thêm component mới cho hợp hệ thì dùng skill `components-states`: nó đọc hệ của file trước khi dựng.

## Kiểm (mặc định)

1. `get_context`, rồi chạy `systemInventory(false)` trong [reference/code.md](reference/code.md).
   Biến, style và component luôn được kê khai trên cả file. Tham số chỉ quyết độ phủ đếm ở đâu:
   `false` là page hiện tại, `true` là mọi page. File rất lớn mà lần gọi quá 60 giây thì báo người
   dùng và đếm từng page.
2. **Xét bốn mặt:**
   - **Đặt tên.** Một kiểu phân cách cho cả file (`color/bg/page`). Tên theo vai trò, không theo
     màu (`color/danger`, không phải `color/red`). Variant theo `Property=Value`.
   - **Token.** Màu ngữ nghĩa trỏ về màu gốc (alias), không lặp lại mã hex. Đủ mode Light và Dark
     nếu sản phẩm cần. `sameValue` là các biến màu trùng mã màu: xem xét gộp nếu chúng cùng vai trò,
     còn khác vai trò (chữ phụ và viền trùng màu) thì cứ để riêng.
   - **Component.** Set có đủ trạng thái theo bảng *Bộ trạng thái* của skill `components-states`
     (nút có 6 trạng thái, ô nhập có thêm Error và Success) và có mô tả. `maybeDetached` là frame
     trùng tên component, có thể là instance đã bị detach.
   - **Độ phủ.** Tỉ lệ `boundFills / solidFills` (màu) và `styledTexts / texts` (chữ). Dưới 80%
     là dev sẽ phải đoán nhiều.
3. **Báo cáo** theo mẫu. Việc nên làm xếp theo tác động.
4. **Sửa là việc riêng, hỏi trước.** Đổi tên, gộp biến, thêm trạng thái: làm bằng skill
   `color-system`, `layout-type`, `components-states`. Đổi tên biến hay component ảnh hưởng mọi chỗ
   đang dùng, kể cả file khác dùng thư viện này: nói rõ điều đó với người dùng.

## Tài liệu cho component

1. Tìm component set theo tên, `inspect_nodes` và `screenshot` nó.
2. Viết mô tả ngắn: dùng khi nào, variant, props, trạng thái, trợ năng, nên và không nên.
3. Người dùng đồng ý thì chạy `documentComponent(id, markdown)`: mô tả được gắn vào component, hiện
   ở panel bên phải và trong Dev Mode. Muốn có bảng tài liệu trên trang Components thì dùng skill
   `components-states` (bước 6).

## Mẫu báo cáo

```markdown
## Hệ thiết kế: <tên file>
3 collection · 124 biến · 18 text style · 22 component set
Độ phủ (page Screens): màu 71% · chữ 88%

### Đặt tên
### Token
### Component
| Component | Thiếu | Ghi chú |
|---|---|---|
### Việc nên làm
1. **Gắn biến cho 29% màu còn viết cứng**: chạy `audit_design` từng màn để ra danh sách.
```

## Lưu ý

- Phần kiểm chỉ đọc. Mọi thay đổi đều hỏi trước.
- Biến, style và component lấy từ thư viện ngoài không phải local nên không được đếm ở đây. Ghi rõ
  điều này trong báo cáo.
- File lớn: `systemInventory` dừng ở 20.000 node (`stoppedEarly: true`). Khi đó kiểm từng page.
- Kiểm lỗi của một màn cụ thể thì dùng skill `design-audit`.
