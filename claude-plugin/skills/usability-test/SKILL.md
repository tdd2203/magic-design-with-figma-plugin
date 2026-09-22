---
name: usability-test
description: 'Lên kế hoạch cho người dùng thật thử prototype trong file Figma đang mở: đọc các luồng prototype, tìm màn cụt, rồi viết mục tiêu, nhiệm vụ, kịch bản buổi test, câu hỏi và bảng ghi chép. Dùng khi người dùng nói "test prototype với người dùng", "usability test", "kịch bản test prototype", "lên kế hoạch test" cho thiết kế trong file Figma nối qua plugin Magic Design with Figma.'
argument-hint: "<điều muốn biết, ví dụ: người mới có tự đặt hàng được không>"
---

# Thử prototype với người dùng

Chuẩn bị một buổi cho người dùng thật bấm thử prototype Figma, để biết thiết kế vấp ở đâu trước khi
dev làm.

## Cách dùng

```
/usability-test                                      lập kế hoạch cho các luồng prototype trên page
/usability-test người mới có tự đặt hàng được không  lập kế hoạch quanh một câu hỏi
```

## Các bước

1. **Hỏi điều muốn biết**, nếu người dùng chưa nói: sau buổi test sẽ quyết định gì (ví dụ "giữ hay
   bỏ bước xác nhận"). Không có quyết định nào phụ thuộc vào kết quả thì chưa cần test.
2. **Đọc prototype.** `get_context`, rồi chạy `prototypeMap` trong [reference/code.md](reference/code.md):
   nó trả về từng luồng (flow), các màn theo thứ tự đi tới, và màn cụt (không đi tiếp, không quay
   lại được). `screenshot` màn đầu của mỗi luồng.
3. **Sửa prototype trước khi test.** Màn cụt, nút bấm không đi đâu, dữ liệu giả (lorem, "abc") làm
   người test vấp vì prototype chứ không phải vì thiết kế. Liệt kê cho người dùng. Màn kết thúc thật
   (như "Đặt hàng thành công") thì không tính là lỗi. Page chưa có flow nào thì nhờ người dùng đặt
   điểm bắt đầu flow trong Figma (tab Prototype).
4. **Viết kế hoạch** theo mẫu:
   - 3–5 nhiệm vụ. Mỗi nhiệm vụ là một tình huống thật và **không lộ tên nút**: "Bạn muốn gửi quà
     cho mẹ trước thứ Sáu", không phải "Bấm Thêm vào giỏ".
   - Thế nào là xong: tới màn đích nào trong prototype, trong khoảng bao lâu.
   - 5 người đúng đối tượng là đủ để thấy phần lớn chỗ vấp của một luồng.
5. **Viết kịch bản** cho buổi 45 phút: làm quen 5' → hỏi bối cảnh 10' → làm nhiệm vụ 20' → cảm nhận
   5' → kết 5'.
6. Người dùng muốn thì lưu thành `research/<tên>-ke-hoach.md` trong thư mục dự án. Sau buổi test,
   dán ghi chép vào `/research-board` để tổng hợp.

## Mẫu kế hoạch

```markdown
## Kế hoạch test: <tên>
Muốn biết: … · Quyết định sau test: …
Prototype: page <tên>, luồng "<flow>" (5 màn) · Cần sửa trước: …
Người tham gia: 5 người, <tiêu chí chọn>

### Nhiệm vụ
| # | Tình huống đọc cho người test | Xong khi | Ghi lại |
|---|---|---|---|
| 1 | … | tới màn "Đặt hàng thành công" | làm được không, mất bao lâu, vấp ở đâu |

### Kịch bản
**Làm quen (5')**: "Hôm nay chúng tôi thử thiết kế, không thử bạn. Bạn cứ nói to điều mình đang nghĩ."
**Bối cảnh (10')**: 2–3 câu về cách họ đang làm việc này hôm nay.
**Nhiệm vụ (20')**: đọc tình huống rồi im lặng quan sát. Họ dừng lại thì hỏi "Bạn đang tìm gì?".
**Cảm nhận (5')**: phần nào dễ nhất, phần nào khó nhất.
**Kết (5')**: cảm ơn, hỏi còn điều gì muốn nói.

### Ghi chép mỗi nhiệm vụ
| Người | Xong? | Thời gian | Vấp ở đâu | Câu nói đáng nhớ |
|---|---|---|---|---|
```

## Lưu ý

- Hỏi mở, không gợi ý: "Bạn nghĩ nút này làm gì?", không phải "Nút này dễ hiểu đúng không?".
- Hỏi về việc họ đã làm, không hỏi họ sẽ làm gì trong tương lai.
- Gọi người tham gia bằng mã P1, P2… Không ghi số điện thoại, email hay tên thật vào file Figma.
- Gửi link prototype và cấp quyền xem là việc người dùng tự làm trong Figma. Plugin không đổi quyền
  chia sẻ.
