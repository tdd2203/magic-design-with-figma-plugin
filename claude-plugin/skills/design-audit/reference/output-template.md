# Mẫu báo cáo

Ba mẫu: **kiểm** (sau `audit_design`), **đã sửa** (trước → sau), **phê bình** (khi người dùng hỏi
"ổn chưa", "review"). Viết tiếng Việt; giữ tên tool, tên thuộc tính Figma và thuật ngữ như auto
layout, variable, fill, instance bằng tiếng Anh.

## Khi nào được nói "xong"

Chỉ khi đủ cả bốn điều:

1. `audit_design` trên mọi frame vừa đổi (và `scope: "palette"` nếu đã đổi biến màu) có **FAIL = 0**.
2. Mọi WARN **đã sửa**, hoặc **đã miễn kèm lý do** viết ra.
3. Mọi SKIP **đã soi bằng mắt**, và kết quả soi có trong báo cáo.
4. Danh sách "máy không đo được" đã đưa cho người dùng.

Thiếu điều nào thì câu đúng là "chưa xong, còn …", kèm đúng thứ còn thiếu.

## Quy tắc câu chữ

- **Nói số, không nói tính từ:** "3,1:1 (cần 4,5:1)", không "hơi nhạt". Số viết kiểu Việt: 4,5:1 · 18,66px.
- **Mỗi phát hiện ba dòng:** chỗ (tên layer + id) · số đo · cách sửa trên canvas.
- **Không làm nhẹ FAIL:** "trượt WCAG 1.4.3", không "có thể cân nhắc".
- **Không nói "đạt hết" khi còn SKIP.** Câu đúng: "máy không thấy lỗi; còn 3 chỗ chưa đo được, đã soi bằng mắt: …".
- **Tối đa 5 chỗ cụ thể cho mỗi phát hiện**, phần còn lại ghi "và 12 chỗ nữa".
- **Nhóm trống vẫn in**, ghi "đã đo, không thấy gì trong phạm vi này".
- Báo cáo dài: đưa dòng **Tổng** và **ba việc nên làm ngay** lên đầu.

## Mẫu 1: báo cáo kiểm

```markdown
## Kiểm thiết kế · <tên frame> (<id>)

**Phạm vi:** scope `design` · <node_id | selection | các frame cấp cao của trang "<tên trang>"> · <n> frame · mode đang hiển thị: <Light | Dark>
**Ngưỡng:** chữ 4,5:1 · chữ lớn (≥ 24px, hoặc ≥ 18,66px khi weight ≥ 700) 3:1 · viền ô nhập 3:1 (WCAG 2.2 SC 1.4.3, 1.4.11) · vùng bấm 24px (SC 2.5.8), mobile 44px (SC 2.5.5 AAA, Apple HIG; Android 48dp) · khoảng cách bội số 4 (tri thức thiết kế)
**Tổng:** FAIL <n> · WARN <n> · SKIP <n> · đã miễn <n>

### 1 · Đọc được không
- **FAIL · tuong-phan-chu ×3** · "Button/Primary › Label" (12:345)
  Đo: 3,7:1, cần 4,5:1 (chữ #FFFFFF 14px Medium trên nền #3B82F6)
  Sửa: gắn fill của nút vào biến action nấc 600 (#2563EB, đạt 5,2:1); chữ giữ biến "chữ trên action"
  Cùng lỗi: "Badge › Mới" (12:400) 2,9:1 · "Tag › Beta" (12:410) 3,4:1

### 2 · Dùng được không
- Đã đo, không thấy gì trong phạm vi này.

### 3 · Hiểu được không
- **WARN · noi-dung-gia ×2** · "Card › Description" (14:20)
  Đo: chữ "Lorem ipsum dolor sit amet…"
  Sửa: thay bằng mô tả thật, rồi thử bản 200 ký tự để xem thẻ có vỡ không

### 4 · Có theo hệ thiết kế không
- **WARN · lech-nac-khoang-cach ×7** · frame "Form" (15:2)
  Đo: itemSpacing 13 · paddingLeft 10 · paddingRight 18 …
  Sửa: nắn về 12 · 12 · 16 (đã xem trước danh sách, chờ đồng ý)

### Chưa đo được (SKIP): đã soi bằng mắt
- chu-khong-do-duoc · "Hero › Title" (9:12) trên ảnh: chỗ tệ nhất là góc phải, chữ trắng trên mây sáng, không đọc được. Đề xuất lớp phủ tối 50%.

### Máy không đo được
- <chép nguyên từng mục `notMeasured` của tool>
- <cộng thêm phần mình chưa kiểm: ví dụ "chưa có frame mode Dark", "biến từ thư viện ngoài không phải local nên chưa đo", "trạng thái hover/focus không có trong frame này">

### Miễn trừ và ngưỡng đã đổi
- `mau-khong-bien` ở "Illustration/*": màu của hình minh hoạ, không thuộc hệ token. Người quyết: <tên>, <ngày>.

### Nên làm tiếp
1. Sửa 3 FAIL contrast: một lần `execute_figma_code`, gắn biến. Cần xác nhận trước khi sửa.
2. Thêm lớp phủ cho chữ trên ảnh hero.
3. Nắn 7 khoảng cách lệch nấc.
```

## Mẫu 2: đã sửa (trước → sau)

```markdown
## Đã sửa · <tên frame> (<id>)

| Luật | Trước | Sau |
|---|---|---|
| tuong-phan-chu | FAIL ×3 | 0 |
| lech-nac-khoang-cach | WARN ×7 | 0 |
| mau-khong-bien | WARN ×5 | WARN ×1 (đã miễn: màu minh hoạ) |

**Đã làm:** <n> lần `execute_figma_code`, mỗi lần một bước undo: <tóm tắt từng lần>
**Đo lại:** `audit_design` cùng phạm vi → FAIL 0 · WARN 1 (đã miễn) · SKIP 1 (đã soi)
**Còn lại:** <việc chưa làm, và vì sao>
**Máy không đo được:** <nhắc lại danh sách, hoặc "không đổi so với báo cáo trước">
```

Không có số "sau" từ một lần đo lại thật thì không dùng mẫu này. Viết "đã sửa, chưa đo lại" và đo lại.

## Mẫu 3: phê bình

```markdown
## Phê bình · <tên màn> (<id>)

**Ấn tượng đầu:** mắt rơi vào <…> · việc chính có vẻ là <…> · người dùng sẽ bấm <…> tiếp theo
<khớp / lệch với ý đồ của màn, lệch ở đâu>

**Heuristic: <n>/40 · <Vững | Khá | Cần sửa | Làm lại từ gốc>** <(chấm <k>/10 tiêu chí, quy đổi <x>%) nếu có tiêu chí chưa chấm được>

| # | Tiêu chí | Điểm | Bằng chứng trên canvas |
|---|---|---|---|
| 1 | Cho thấy hệ thống đang làm gì | 2 | Nút "Lưu" (21:4) không có biến thể loading |
| … | … | … | … |

**Tải nhận thức:** trượt <n>/8 mục. <mục trượt · chỗ · cách sửa>
**Vẻ AI chung chung:** <n> dấu hiệu. <dấu hiệu · chỗ>. Kết luận: <có chủ đích | vài chỗ theo phản xạ | trông như mẫu dựng tự động>

**Làm tốt** (2–3 điều, nói rõ vì sao nó tốt):
- …

**Vấn đề ưu tiên** (3–5, nặng nhất trước):
1. **[Nặng]** <vấn đề> · <chỗ, id> · <vì sao hại người dùng> · Sửa: <việc cụ thể trên canvas>
2. …

**Số đo đi kèm:** FAIL <n> · WARN <n> · SKIP <n> (chi tiết ở báo cáo kiểm)
**Máy không đo được:** <danh sách>

**Muốn sửa phần nào trước?** <2–3 lựa chọn cụ thể gắn với vấn đề ở trên, ví dụ "chỉ 3 lỗi contrast", "thêm đủ trạng thái cho nút", "làm lại lưới thẻ">
```
