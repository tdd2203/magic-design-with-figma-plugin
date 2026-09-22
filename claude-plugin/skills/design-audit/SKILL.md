---
name: design-audit
description: Kiểm và phê bình thiết kế trên file Figma đang nối qua plugin magic-design-with-figma — chạy tool audit_design (contrast trên nền đã trộn, vùng bấm, nấc khoảng cách, màu và chữ chưa gắn biến/style, bảng màu light/dark), soi bằng screenshot phần máy không đo được, chấm heuristic và báo cáo bằng tiếng Việt. Dùng khi người dùng nói "audit", "kiểm thiết kế", "review" hay "critique màn này", "đo contrast", "có đạt WCAG/a11y không", "thiết kế ổn chưa", hoặc trước khi báo xong một màn hình dựng bằng các tool Figma của plugin này.
---

# Kiểm và phê bình thiết kế trong Figma

Skill này trả lời một câu: **thiết kế này đã dùng được chưa**, bằng số đo trước, bằng mắt sau.
Phần kiểm **chỉ đọc**: không sửa gì trong file. Sửa là một bước riêng, chỉ làm khi người dùng đồng
ý, và sửa xong phải đo lại.

Mọi phát hiện được xếp theo **bốn câu hỏi độc lập**. Giao diện hỏng thường hỏng đúng ở câu người
làm không hỏi: màu đẹp mà nút quá nhỏ, khoảng cách đều mà chữ bị cắt.

| Nhóm | Câu hỏi | Hỏng thì trông thế nào |
|---|---|---|
| 1 · Đọc được không | Chữ và ranh giới có nhìn ra không? | Chữ nhạt, ô nhập vô hình, chữ trên ảnh không đọc nổi |
| 2 · Dùng được không | Bấm trúng được không? | Icon 16px không vùng bấm, nút mobile 32px |
| 3 · Hiểu được không | Nội dung có thật, đủ và có nghĩa không? | `undefined`, `NaN`, lorem ipsum, chữ bị khung cắt mất |
| 4 · Có theo hệ thiết kế không | Có dùng đúng biến, style, nấc của file không? | Mã màu rời, khoảng cách 13px, chữ không style |

## Thứ tự ưu tiên: tri thức, không phải luật

Số và quy ước trong skill này là **mặc định để bắt đầu**. Khi có mâu thuẫn: **biến, style và
component của chính file Figma thắng**, sau đó tới **DESIGN.md của dự án** nếu người dùng có, cuối
cùng mới tới mặc định ở đây. Riêng các ngưỡng WCAG là chuẩn quốc tế: dự án được đặt khắt hơn (AAA),
còn nới ra thì phải có lý do viết thành chữ (xem *Kỷ luật báo cáo*).

## Khi nào dùng

- Người dùng hỏi "kiểm", "audit", "review", "critique", "đo contrast", "đạt a11y chưa", "ổn chưa".
- Vừa dựng hoặc sửa xong một màn hình, trước khi báo "xong".
- Vừa đổi biến màu, thêm mode Dark, đổi thang chữ hay khoảng cách.
- Nhận một bảng màu từ ngoài vào file: chạy scope `palette` trước khi dùng.

**Không** dùng skill này để dựng màn mới: việc đó bắt đầu từ `design-workflow`. Khi sửa cần dựng lại
cả một phần (bảng màu, thang chữ, bộ trạng thái của component), chuyển sang các skill thiết kế cùng
plugin: `color-system`, `layout-type`, `components-states`.

## Quy trình

1. **Xác định phạm vi.** `get_context` để biết trang, selection. Hỏi lại nếu không rõ người dùng muốn
   kiểm frame nào. Trang lớn thì kiểm **từng frame một** (kết quả tool bị cắt ở 30.000 ký tự).
2. **Đo bằng máy.** `audit_design` với `scope: "design"` cho frame. Nếu file có biến màu local, chạy
   thêm `scope: "palette"`: nó đo cặp token ở **mọi mode**, kể cả mode chưa frame nào hiện.
3. **Xác minh trước khi tin.** Với mỗi FAIL, `screenshot` đúng node đó (`inspect_nodes` nếu cần
   thêm thuộc tính). Số đo mà mắt thấy vô lý thì báo cả hai, đừng lặng lẽ bỏ.
4. **Soi bằng mắt phần máy không đo** (bảy phép thử ở dưới), bằng `screenshot`.
5. **Phê bình** khi người dùng hỏi "ổn chưa", "review", "critique": chấm 10 heuristic, kiểm tải
   nhận thức, kiểm vẻ "AI chung chung". Chi tiết: [reference/critique.md](reference/critique.md).
6. **Báo cáo** theo mẫu: [reference/output-template.md](reference/output-template.md). In đủ bốn
   nhóm và danh sách "máy không đo được".
7. **Sửa khi được đồng ý**: FAIL trước, rồi WARN. Sửa xong **chạy lại `audit_design` cùng phạm vi**
   và báo trước → sau ("tuong-phan-chu: 3 → 0"). "Đã sửa" chưa phải bằng chứng; "đo lại còn 0" mới phải.

## Tool `audit_design`

Chỉ đọc, không sửa file. Input:

| Tham số | Giá trị | Mặc định |
|---|---|---|
| `node_id` | id một frame/node | bỏ trống: selection hiện tại, không có selection thì các frame cấp cao nhất của trang hiện tại |
| `scope` | `"design"` hoặc `"palette"` | `"design"` |

- `design`: kiểm node theo thứ tự ưu tiên ở trên (node_id → selection → frame cấp cao nhất).
- `palette`: kiểm các **biến màu local** của file, theo từng mode. Biến từ thư viện ngoài không
  phải local nên không được đo. Ghi điều đó vào "máy không đo được".

Output là JSON:

- `summary`: số đếm `{ FAIL, WARN, SKIP }`.
- Phát hiện xếp theo bốn nhóm `1 · Đọc được không` · `2 · Dùng được không` · `3 · Hiểu được không` ·
  `4 · Có theo hệ thiết kế không`. Mỗi phát hiện có `{ rule, group, severity, where, message, fix, count, samples }`.
- Ngưỡng đã dùng, **kèm nguồn của từng số**.
- `notMeasured`: danh sách thứ không máy nào phán được. Luôn chuyển nguyên cho người dùng.

Mã luật (giải thích từng luật, ngưỡng, nguồn và cách sửa trên canvas: [reference/rules.md](reference/rules.md)):

| Nhóm | scope `design` | scope `palette` |
|---|---|---|
| 1 · Đọc được không | `tuong-phan-chu` · `ky-hieu-mo` · `chu-khong-do-duoc` · `chu-mo-bang-opacity` · `ranh-gioi-o-nhap` · `nen-den-thuan` · `chu-trang-thuan` | `bang-mau-tuong-phan` · `den-trang-thuan-token` |
| 2 · Dùng được không | `vung-bam-nho` · `vung-bam-mobile` | |
| 3 · Hiểu được không | `ro-ri-gia-tri` · `noi-dung-gia` · `tran-chu` | |
| 4 · Có theo hệ thiết kế không | `lech-nac-khoang-cach` · `mau-khong-bien` · `chu-khong-style` · `gradient-chu` · `kinh-mo` | `hue-semantic-gan-primary` · `mu-mau-trung-nhau` |

Nhóm ở bảng này là nhóm tool đang gán; khi báo cáo, luôn theo trường `group` trong kết quả.
Hai mã luật SKIP báo **giới hạn đo**, không phải lỗi: `pham-vi` (phạm vi quá lớn, đo từng frame) và `vai-tro-mau`
(không nhận ra vai trò biến màu, bảng màu chưa được đo).

## Đọc FAIL · WARN · SKIP

| Mức | Nghĩa | Làm gì |
|---|---|---|
| **FAIL** | Đã đo, trượt một ngưỡng có nguồn (thường là WCAG) | Sửa trước mọi thứ khác |
| **WARN** | Nghi vấn: máy thiếu ngữ cảnh để chắc, hoặc luật là quy ước chứ không phải chuẩn | Người quyết: sửa, hoặc miễn **có lý do viết ra** |
| **SKIP** | **Không đo được** (chữ trên ảnh, gradient, blur…) | Soi bằng mắt. SKIP **không bao giờ** là "đạt" |

`summary` là `FAIL 0 · WARN 0 · SKIP 5` thì câu đúng là "máy không thấy lỗi, còn 5 chỗ chưa đo được",
không phải "đạt hết".

## Kỷ luật báo cáo

1. **In đủ bốn nhóm, kể cả nhóm trống.** Nhóm trống ghi "đã đo, không thấy gì trong phạm vi này",
   khác hẳn với việc không ai đo.
2. **Mỗi phát hiện có ba thứ:** chỗ (tên layer + id), **số đo** (4,1:1; 20×20px; gap 13px), và cách sửa
   cụ thể trên canvas. Không có số thì không phải phát hiện, chỉ là cảm giác.
3. **Ngưỡng có nguồn.** In dòng ngưỡng kèm nguồn: WCAG 2.2, hoặc tri thức thiết kế đi kèm skill. Không
   tự nghĩ ra ngưỡng giữa chừng.
4. **Đổi ngưỡng hay miễn một luật phải có lý do viết thành chữ** (ai quyết, vì sao, áp ở đâu). Ghi
   vào DESIGN.md nếu dự án có, và luôn in trong báo cáo. Phát hiện được miễn **vẫn in ra**, kèm lý do.
5. **SKIP là việc còn nợ**, không phải điểm cộng.
6. **Luôn chuyển `notMeasured`** cho người dùng, cộng thêm những gì chính mình không kiểm được.

## Ngưỡng chính

| Đo gì | Ngưỡng | Nguồn |
|---|---|---|
| Chữ thường trên nền | ≥ 4,5:1 | WCAG 2.2 SC 1.4.3 (AA) |
| Chữ lớn: ≥ 24px, hoặc ≥ 18,66px khi weight ≥ 700 | ≥ 3:1 | WCAG 2.2 SC 1.4.3 (18pt / 14pt đậm) |
| Viền ô nhập, icon mang nghĩa, vòng focus | ≥ 3:1 | WCAG 2.2 SC 1.4.11 (AA) |
| Vùng bấm | sàn 24×24 · desktop nên 32 · mobile 44 (Android 48dp) | SC 2.5.8 (AA) · SC 2.5.5 (AAA) · tri thức thiết kế |
| Khoảng cách auto layout | bội số của 4 | tri thức thiết kế |
| Hue semantic cách primary | ≥ 60° (OKLCH) | tri thức phối màu |
| Cặp màu dưới mô phỏng mù màu | ΔE-OK ≥ 0,1, chỉ ra WARN | ngưỡng tự đặt, không có chuẩn quốc tế |

APCA chỉ để tham khảo, không dùng để kết luận đạt hay trượt. Bảng đủ: [reference/rules.md](reference/rules.md).

## Máy không đo được: soi bằng `screenshot`

Cách làm từng phép thử: [reference/critique.md](reference/critique.md).

1. **Thang xám:** bỏ màu đi, vẫn tìm ra nút chính, chữ phụ, mục đang chọn không?
2. **Nheo mắt / làm mờ:** ảnh nhỏ cỡ thumbnail, còn thấy thứ bậc và điểm nhìn đầu tiên không?
3. **Chữ trên ảnh, gradient, kính mờ:** soi chỗ tệ nhất của nền, không soi chỗ đẹp nhất.
4. **Mù màu:** chỗ nào chỉ có màu mang nghĩa (đỏ/xanh, chấm trạng thái, viền lỗi) thì cần thêm icon hoặc chữ.
5. **Đủ trạng thái:** nút có default · hover · pressed · focus · disabled · loading; ô nhập có default ·
   hover · focus · disabled, thêm error · success; màn hình đủ bảy trạng thái (có dữ liệu, trống, đang
   tải, một phần, lỗi, mất kết nối, không có quyền).
6. **Nội dung thật, nội dung bẩn:** tên dài, dấu tiếng Việt chồng tầng, số âm, 0 dòng và 5.000 dòng.
7. **Nhịp thị giác:** khoảng trong nhóm nhỏ hơn khoảng giữa nhóm, thẳng hàng, bo góc và cỡ chữ theo thang.

## Sửa trong Figma: nguyên tắc

- **Hỏi trước khi sửa.** Mỗi lần `execute_figma_code` là một bước undo, nên gom mỗi loại lỗi vào một lần gọi.
- **Layer nằm trong instance thì sửa ở main component** (`getMainComponentAsync()`), và nói cho người
  dùng biết việc đó đổi mọi instance. Sửa override lẻ chỉ khi người dùng muốn đúng một chỗ khác đi.
- **Gắn biến theo vai trò, không theo mã hex gần nhất.** Chữ phụ lấy biến chữ phụ. Chọn "màu gần nhất"
  có thể gắn nhầm một màu semantic.
- **Không lách luật:** không phóng chữ lên 24px chỉ để hưởng ngưỡng 3:1; không miễn SKIP mà chưa soi.
- Công thức contrast, đoạn code đọc thuộc tính chữ, gắn biến, áp text style, nắn khoảng cách, nới
  vùng bấm: [reference/rules.md](reference/rules.md).

## Khi tool không chạy được

- Plugin chưa nối hay chưa ghép cặp: chuyển nguyên hướng dẫn của tool cho người dùng.
- Không có `audit_design` (plugin cũ): nói rõ với người dùng. Có thể đo tay vài cặp chữ/nền quan trọng
  bằng công thức ở [reference/rules.md](reference/rules.md), còn lại ghi là **chưa đo**. Không được đoán
  rồi báo đạt.
- Kết quả bị cắt (`…[truncated`): thu hẹp phạm vi, kiểm từng frame.

## Tài liệu kèm

- [reference/rules.md](reference/rules.md): từng mã luật (đo gì, ngưỡng, nguồn, sửa thế nào), bảng
  ngưỡng đầy đủ, đoạn code đo và sửa.
- [reference/critique.md](reference/critique.md): bảy phép thử bằng mắt, chấm 10 heuristic (0–4, /40),
  checklist tải nhận thức, kiểm vẻ "AI chung chung".
- [reference/output-template.md](reference/output-template.md): mẫu báo cáo tiếng Việt, mẫu trước → sau,
  khi nào được nói "xong".
