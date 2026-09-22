# Sổ luật của `audit_design`

Mỗi luật ghi: **đo gì**, **ngưỡng**, **nguồn**, **sửa thế nào trên canvas**, và **bẫy** (lúc số đo dễ
hiểu sai). Đọc sổ này khi cần giải thích một phát hiện, hoặc trước khi sửa.

## 0. Mức và nhóm do kết quả quyết

- `severity` và `group` in trong kết quả của tool là **mức thật**. Sổ này xếp luật theo nhóm tool đang
  gán để dễ tra, còn "mức thường gặp" chỉ để đoán trước. Hai bên lệch nhau thì tin kết quả.
- Quy tắc chung: luật có nguồn **WCAG** trượt thì ra **FAIL**; luật từ **tri thức thiết kế** (quy ước,
  không phải chuẩn) ra **WARN**; chỗ **không đo được** ra **SKIP**. Cố định theo đặc tả của tool:
  `ky-hieu-mo` luôn là WARN, `chu-khong-do-duoc` luôn là SKIP, `mu-mau-trung-nhau` luôn là WARN.
- `where` chỉ ra layer. Dùng `inspect_nodes` và `screenshot` trên id đó để xem tận mắt trước khi sửa.

## 1. Ngưỡng và nguồn

| Đại lượng | Ngưỡng | Nguồn | Loại |
|---|---|---|---|
| Chữ thường | ≥ 4,5:1 | WCAG 2.2 SC 1.4.3 (AA) | chuẩn |
| Chữ lớn | ≥ 3:1 | WCAG 2.2 SC 1.4.3 | chuẩn |
| Thế nào là chữ lớn | ≥ 24px, hoặc ≥ 18,66px khi weight ≥ 700 | WCAG định nghĩa 18pt / 14pt đậm; 1pt = 1,333px. Chữ 18–23px weight thường **vẫn cần 4,5** | chuẩn |
| Thành phần không phải chữ: viền ô nhập, icon mang nghĩa, vòng focus | ≥ 3:1 với nền kề nó | WCAG 2.2 SC 1.4.11 (AA) | chuẩn |
| Link trong đoạn văn không gạch chân | khác màu chữ quanh nó ≥ 3:1 | WCAG SC 1.4.1, kỹ thuật G183 (soi bằng mắt, tool không đo) | chuẩn |
| Vùng bấm | ≥ 24×24px | WCAG 2.2 SC 2.5.8 (AA) | chuẩn |
| Vùng bấm nên dùng | desktop 32px · mobile 44px (iOS 44pt, Android 48dp) | SC 2.5.5 (AAA) = 44; tri thức thiết kế | tri thức |
| Khoảng cách auto layout | bội số của 4 | tri thức thiết kế | tri thức |
| Opacity của layer chữ | ≥ 0,7 | mốc do tool đặt; tri thức phối màu: không làm nhạt chữ bằng opacity | tự đặt |
| Hue semantic so với primary | cách ≥ 60° (hue OKLCH) | tri thức phối màu | tri thức |
| Cặp màu dưới mô phỏng mù màu | ΔE-OK ≥ 0,1 | **tự đặt**, không có chuẩn quốc tế; chỉ ra WARN | tự đặt |
| Nền tối, chữ sáng | nền không #000 (khoảng #0F172A–#18181B) · chữ không #FFFFFF | tri thức phối màu | tri thức |
| Độ dài dòng của đoạn chữ từ hai dòng | ≤ 80 ký tự mỗi dòng (nên 65–75) | WCAG 2.2 SC 1.4.8 (AAA); tri thức thiết kế | tri thức |
| Line height của chữ cỡ thân (≤ 18px) từ hai dòng | ≥ 1,3 lần cỡ chữ (nên 1,4–1,5) | tri thức thiết kế (thang 12/16, 14/20, 16/24); SC 1.4.8 (AAA) khuyên 1,5 | tri thức |
| Căn lề đoạn văn | căn giữa tối đa 2 dòng · không căn đều hai bên | tri thức thiết kế; SC 1.4.8 (AAA) cho căn đều | tri thức |
| Viết hoa toàn bộ | tối đa 4 tiếng · nhãn in hoa ≤ 20px giãn chữ ≥ 2% (nên 5–10%) | tri thức thiết kế | tri thức |
| Thang chữ trong một màn | ≤ 2 họ font · ≤ 3 độ đậm · ≤ 7 cỡ · không cỡ lẻ | tri thức thiết kế | tri thức |

Công thức đều là công thức công bố: tỉ lệ tương phản và độ chói tương đối theo WCAG 2.x; trộn lớp bán
trong suốt kiểu source-over; OKLab/OKLCH của Björn Ottosson; mô phỏng mù màu theo ma trận
Machado–Oliveira–Fernandes (2009).

Không phải ngưỡng, chỉ là lời khuyên: **APCA** để tham khảo (Lc 75 cho đoạn văn thân bài), chưa phải
chuẩn pháp lý. Đoạn đọc lâu nên nhắm 12–16:1, đừng đẩy tới 21:1. Mobile nên nhắm 5:1 vì màn hình
hay bị xem ngoài nắng.

**Đổi một ngưỡng** (ví dụ dự án y tế đòi AAA 7:1, hoặc DESIGN.md chọn nấc 8): ghi `luật · giá trị mới ·
lý do · ai quyết` vào DESIGN.md (hoặc vào báo cáo nếu dự án chưa có), rồi in dòng đó trong mọi báo cáo sau.

## 2. Nhóm 1 · Đọc được không

### `tuong-phan-chu`
- **Đo gì:** tỉ lệ giữa màu chữ và **nền đã trộn** ngay dưới chữ: fill của các lớp cha, **cộng các lớp
  anh em nằm dưới và che lên chữ**, tính cả opacity của paint lẫn opacity của layer.
- **Ngưỡng:** 4,5:1; chữ lớn 3:1. **Nguồn:** WCAG 2.2 SC 1.4.3. **Mức thường gặp:** FAIL.
- **Sửa:** đổi fill chữ sang biến **cùng vai trò** mà đạt (chữ phụ vẫn lấy biến chữ phụ, chỉ là nấc đậm
  hơn), hoặc đổi nền. Chữ trên nút đo với nền nút: dùng cặp biến "chữ trên action" và "action". Chữ
  placeholder và chữ muted **vẫn là chữ**, vẫn cần 4,5. Sửa ở biến thì chạy thêm `scope: "palette"` để
  chắc mode kia không trượt.
- **Bẫy:** nền tint bán trong suốt (badge tô màu semantic ở khoảng 12%) làm số tụt 0,3–0,7 so với đo
  trên màu gốc, nên luôn đo trên màu đã trộn. Chữ của nút disabled và logo được WCAG miễn: tool **tự bỏ qua**
  mọi layer nằm trong layer có tên chứa `disabled` / `State=Disabled` / `vô hiệu` (ghi lý do trong `notMeasured`),
  nên đặt tên biến thể disabled cho đúng. Nút disabled vẫn phải nhận ra được là nút. Logo thì tool không tự biết:
  miễn có ghi lý do.

### `ky-hieu-mo`
- **Đo gì:** layer chữ chỉ có ký hiệu (·, •, →, ›, ×…) mà dưới ngưỡng. **Mức:** luôn WARN.
- **Sửa:** ký hiệu trang trí thuần thì để nguyên, ghi là trang trí. Ký hiệu **mang nghĩa** (× để đóng,
  → để đi tiếp, • báo trạng thái) là thành phần không phải chữ: đưa lên ≥ 3:1, hoặc thay bằng icon có nhãn.

### `chu-khong-do-duoc`
- **Đo gì:** chữ nằm trên fill ảnh, gradient hay lớp blur. Máy không đoán màu nền nên báo **SKIP**.
- **Soi bằng mắt:** `screenshot` vùng chữ, nhìn chỗ **tệ nhất** của nền. Với gradient: đọc
  `gradientStops` của lớp nền (đoạn code 6.6), đo chữ với từng stop bằng hàm ở 6.2, lấy số thấp nhất.
- **Sửa:** đặt chữ lên một mặt nền đặc; hoặc thêm lớp phủ tối 40–60% giữa ảnh và chữ (rectangle fill
  đen, opacity 0.4–0.6, hoặc gradient phủ ở phía có chữ); hoặc dời chữ ra khỏi ảnh.

### `chu-mo-bang-opacity`
- **Đo gì:** layer chữ có opacity < 0,7. **Nguồn:** tri thức phối màu. **Mức thường gặp:** WARN.
- **Vì sao:** opacity làm màu chữ phụ thuộc vào thứ nằm dưới, nên cùng một chữ đạt trên nền này mà
  trượt trên nền kia. Hover bằng opacity cũng hỏng vì cùng lý do.
- **Sửa:** `opacity = 1`, gắn fill vào biến chữ phụ hoặc muted đạt 4,5 (đoạn 6.3).

### `ranh-gioi-o-nhap`
- **Đo gì:** layer trông như ô nhập (input, select, ô tìm…): viền **hoặc** nền của nó so với nền quanh ≥ 3:1.
- **Nguồn:** WCAG 2.2 SC 1.4.11. **Mức thường gặp:** FAIL.
- **Sửa:** stroke 1px gắn biến "viền ô nhập" (neutral khoảng nấc 500, đạt 3:1 trên mọi nền nó nằm).
  Viền trang trí (đường chia, viền thẻ) **không** bắt buộc 3:1. Chỉ viền giúp nhận ra ô nhập mới bắt buộc.
- **Cách tool tìm viền:** đo stroke/fill của chính layer; layer đó không vẽ gì (khung bọc) thì đo layer con
  phủ ≥ 90% diện tích nó. Không thấy viền hay nền nào → **WARN** ("có thể không có ranh giới"), không phải FAIL.
  Viền gradient/ảnh thì bỏ qua. Ô nhập ở trạng thái disabled được miễn (WCAG 1.4.11).
- **Bẫy:** tool nhận ô nhập chủ yếu qua tên layer (Input, Field, Select, Search…). Ô nhập đặt tên lạ
  có thể lọt, nên soi thêm bằng mắt; đặt tên layer theo vai trò cũng giúp tool đo đúng.
  Vòng focus (stroke 2px cách mép 2px) cũng cần ≥ 3:1 với nền quanh nó. Có khe 2px thì vòng cùng màu
  nút vẫn được; vòng dính sát mép thì phải khác màu nút. Tool không đo nó, phải soi biến thể `State=Focus`.

### `nen-den-thuan`
- **Đo gì:** frame lớn fill #000000 thuần. **Nguồn:** tri thức phối màu. **Mức thường gặp:** WARN.
- **Sửa:** nền tối khoảng #0F172A–#18181B, gắn biến nền trang ở mode Dark. Phân lớp bằng **độ sáng
  nền** (bề mặt càng nổi càng sáng), không dựa vào bóng đổ vì bóng gần như vô hình trên nền tối.
- **Bẫy:** trình phát video, ảnh tràn viền: miễn có lý do.

### `chu-trang-thuan`
- **Đo gì:** chữ #FFFFFF thuần trên nền tối. **Mức thường gặp:** WARN.
- **Sửa:** dùng nấc neutral sáng nhất (50) ám nhẹ hue primary, gắn biến chữ chính ở mode Dark.
  Trắng thuần trên nền tối chênh quá gắt, đọc lâu mỏi mắt (đoạn đọc lâu chỉ cần 12–16:1); nấc 50 vẫn
  dư sức đạt 4,5:1.

### `dong-qua-dai`
- **Đo gì:** đoạn chữ từ hai dòng có trung bình quá 80 ký tự mỗi dòng (tổng ký tự chia số dòng).
  **Nguồn:** WCAG 2.2 SC 1.4.8 (AAA); tri thức thiết kế khuyên 65–75. **Mức thường gặp:** WARN.
- **Sửa:** hộp chữ rộng khoảng 40–45 lần cỡ chữ (16px ≈ 640–720px): đoạn văn `FILL` trong cột có
  `maxWidth`, hoặc đặt `maxWidth` thẳng trên chữ khi nó là con của auto layout (đoạn 6.7).
- **Bẫy:** bảng, mã, dữ liệu xếp cột không phải văn để đọc: miễn có lý do. Chữ một dòng không bị chấm.

### `khoang-dong-chat`
- **Đo gì:** chữ cỡ thân (≤ 18px) từ hai dòng có line height dưới 1,3 lần cỡ chữ. Line height `AUTO`
  tính ≈ 1,2 vì phần lớn font giao diện cho ra khoảng đó. **Nguồn:** tri thức thiết kế (thang 12/16,
  14/20, 16/24); SC 1.4.8 (AAA) khuyên 1,5. **Mức thường gặp:** WARN.
- **Sửa:** line height theo px, bội số của 4: 12 → 16, 14 → 20, 16 → 24, 18 → 28 (đoạn 6.7). Tốt nhất là
  áp text style có sẵn của file.
- **Bẫy:** tiếng Việt chồng dấu ("Ẩ", "Ỗ") cần dòng rộng hơn chữ Latin; dòng chật thì dấu chạm dòng trên,
  hoặc bị cắt khi frame `clipsContent` (`tran-chu`).

### `can-le-kho-doc`
- **Đo gì:** chữ căn giữa từ ba dòng, hoặc căn đều hai bên (`JUSTIFIED`) từ hai dòng.
  **Nguồn:** tri thức thiết kế; SC 1.4.8 (AAA) cho căn đều. **Mức thường gặp:** WARN.
- **Sửa:** đoạn văn `textAlignHorizontal = 'LEFT'`. Tiêu đề căn giữa thì giữ tối đa hai dòng: rút câu hoặc
  nới bề rộng; `textWrapStyle = 'BALANCE'` cho các dòng dài gần bằng nhau.
- **Bẫy:** hộp chữ cố định cao hơn nội dung (kéo cao để căn giữa theo chiều dọc) không bị đếm thành nhiều
  dòng: tool đếm dòng theo nét chữ thật.

### `viet-hoa`
- **Đo gì:** chữ viết hoa toàn bộ (bằng `textCase = 'UPPER'` hoặc gõ hoa sẵn) quá 4 tiếng; hoặc nhãn in
  hoa ≤ 20px giãn chữ dưới 2%. Một cụm gõ hoa đứng một mình (USD, PDF) được coi là viết tắt, bỏ qua.
  **Nguồn:** tri thức thiết kế: chữ hoa mất hình dáng từ nên đọc chậm; khoảng chữ mặc định của font
  canh cho chữ thường. **Mức thường gặp:** WARN.
- **Sửa:** câu dài để chữ thường. Nhãn ngắn: nội dung gốc chữ thường, `textCase = 'UPPER'`,
  `letterSpacing = { unit: 'PERCENT', value: 6 }` (5–10%). Load font trước khi đổi.
- **Bẫy:** tiếng Việt đếm theo tiếng: "đăng nhập" là 2 tiếng. Logo, mã sản phẩm: miễn có lý do.

### `bang-mau-tuong-phan` (scope `palette`)
- **Đo gì:** cặp biến chữ/nền và chữ-trên-action/action, **từng mode**, dưới 4,5:1.
- **Nguồn:** WCAG 2.2 SC 1.4.3. **Mức thường gặp:** FAIL.
- **Sửa:** đổi giá trị biến ở đúng mode đó: light lấy nấc đậm hơn, dark lấy nấc sáng hơn. Nút chữ trắng
  không phải hue nào cũng đạt ở nấc 600, lùi một nấc. Đo lại, đừng tin bảng màu "đã kiểm sẵn".
- **Cách ghép cặp:** tool đọc vai trò từ **tên biến** rồi đo **mọi** cặp: mọi biến chữ (`text`, `text/primary`,
  `text/secondary`, `text/muted`…) với mọi biến nền (`bg`, `bg/page`, `bg/surface`…), và mọi biến chữ-trên-action
  với mọi biến action/primary. Biến bị ẩn khỏi picker (`scopes = []`, thường là ramp Primitives) **không** bị
  chấm, vì người thiết kế không áp chúng trực tiếp; file không có biến nào hiện thì tool chấm tất cả.
- **Bẫy:** biến đặt tên theo vai trò (text, bg/background, action/primary, success, danger…) thì ghép đúng.
  Tên lạ thì không ghép được: đo tay (6.2).

### `vai-tro-mau` (scope `palette`)
- **Đo gì:** không phải lỗi thiết kế mà là **giới hạn đo**: file chưa có biến màu nào, hoặc không tên biến
  nào cho thấy vai trò. **Mức:** SKIP, nghĩa là bảng màu **chưa được đo**, không phải đạt.
- **Sửa:** đặt tên biến theo vai trò (`color/text/primary`, `color/bg/page`, `color/action`,
  `color/text/on-action`, `color/success`…) như skill `color-system` hướng dẫn, rồi chạy lại.

### `den-trang-thuan-token` (scope `palette`)
- **Đo gì:** mode Dark có biến nền = #000 hoặc biến chữ = #FFF. **Mức thường gặp:** WARN.
- **Sửa:** như `nen-den-thuan` và `chu-trang-thuan`, nhưng sửa ở biến, mọi frame sẽ theo.

### `pham-vi`
- **Đo gì:** giới hạn đo: phạm vi quá lớn, tool dừng sau 5.000 layer. **Mức:** SKIP; phần còn lại **chưa đo**.
- **Sửa:** chạy lại với `node_id` của từng frame/màn thay vì cả trang.

## 3. Nhóm 2 · Dùng được không

### `vung-bam-nho`
- **Đo gì:** layer trông như nút, link hay icon bấm được mà nhỏ hơn 24×24. Tool nhận chúng chủ yếu qua
  tên layer (Button, Link, Tab, Checkbox, Close…), nên icon bấm được mà đặt tên lạ phải soi bằng mắt.
- **Nguồn:** WCAG 2.2 SC 2.5.8 (AA). **Mức thường gặp:** FAIL.
- **Bẫy:** SC 2.5.8 có ngoại lệ. Target nhỏ vẫn đạt nếu vòng tròn đường kính 24px đặt giữa nó không chạm
  target khác, cũng không chạm vòng tròn của một target nhỏ khác. Link nằm trong đoạn văn cũng được miễn.
  Tool có thể chưa tính các ngoại lệ này, nên soi `screenshot` trước khi kết luận.
- **Sửa:** nút có auto layout thì tăng padding hoặc đặt `minHeight`/`minWidth`. Icon trần thì bọc trong
  frame vùng bấm 40px, tối thiểu 32px trên desktop (đoạn 6.5). Vùng bấm lớn hơn phần nhìn thấy là bình
  thường: icon 16px trong frame 40px. Hai vùng bấm kề nhau nên cách ≥ 8px.

### `vung-bam-mobile`
- **Đo gì:** như trên, nhưng < 44 trong frame rộng ≤ 480px (màn mobile).
- **Nguồn:** SC 2.5.5 (AAA) = 44; Apple HIG 44pt; Material 48dp. **Mức thường gặp:** WARN.
- **Sửa:** vùng bấm 44×44 (Android 48); nút và ô nhập mobile cao 44–48px.

## 4. Nhóm 3 · Hiểu được không

### `ro-ri-gia-tri`
- **Đo gì:** chữ hiện `undefined`, `NaN`, `[object Object]`, `Invalid Date`, `null`.
- **Nguồn:** tri thức thiết kế (checklist bàn giao). **Mức thường gặp:** WARN.
- **Sửa:** thay bằng dữ liệu thật. Nếu muốn thể hiện "không có dữ liệu" thì thiết kế hẳn: "—", "Chưa có",
  hoặc một trạng thái trống, và ghi chú cho dev biết giá trị thiếu thì hiện gì.
- **Bẫy:** màn cho lập trình viên (log, JSON) có thể hiện `null` thật. Miễn có lý do.

### `noi-dung-gia`
- **Đo gì:** lorem ipsum và chữ giữ chỗ. **Mức thường gặp:** WARN.
- **Sửa:** viết nội dung thật bằng ngôn ngữ của người dùng. Nhãn nút là động từ + danh từ ("Lưu thay đổi").
  Rồi thử thêm dữ liệu bẩn: tên 200 ký tự, số âm, 0 dòng. Lorem giấu mất độ dài thật, và không có dấu
  tiếng Việt chồng tầng để lộ chỗ line-height quá sát.

### `tran-chu`
- **Đo gì:** chữ bị một lớp tổ tiên có `clipsContent` cắt mất. **Mức thường gặp:** WARN.
- **Sửa:** cho chữ tự cao (`textAutoResize = 'HEIGHT'`) trong một cha auto layout hug; hoặc **thiết kế**
  việc cắt chữ cho có chủ đích: `textTruncation = 'ENDING'` kèm `maxLines`, và một biến thể hiện đủ chữ
  (tooltip). Tên tiếng Việt chữ hoa chồng dấu ("ĐẶNG THỊ NGỌC ẨN") hay bị cắt khi line-height quá sát.

## 5. Nhóm 4 · Có theo hệ thiết kế không

### `lech-nac-khoang-cach`
- **Đo gì:** gap hoặc padding của auto layout không chia hết cho 4: `itemSpacing` (bỏ qua khi phân bố
  `SPACE_BETWEEN`), `counterAxisSpacing` khi wrap, `paddingTop`… và gap của lưới.
- **Nguồn:** tri thức thiết kế. Thang mặc định 4 · 8 · 12 · 16 · 20 · 24 · 32 · 40 · 48 · 64 · 80 · 96.
  **Mức thường gặp:** WARN.
- **Sửa:** file có biến khoảng cách thì gắn biến (`frame.setBoundVariable('itemSpacing', variable)`, tương tự
  cho `paddingTop`…). Không có thì nắn về nấc gần nhất (đoạn 6.4, chạy xem trước rồi mới áp). Nhân tiện
  soi quan hệ gần xa: khoảng trong nhóm ≤ một nửa khoảng giữa nhóm.
- **Bẫy:** DESIGN.md chọn nấc khác (8, hay 2 cho vùng icon dày) thì theo DESIGN.md. Instance lấy padding
  từ main component, sửa ở đó.

### `mau-khong-bien`
- **Đo gì:** màu solid không gắn biến hay paint style, trong khi file **có** biến hoặc style màu.
- **Nguồn:** tri thức phối màu: thứ bàn giao là bảng token, không phải mã hex. **Mức thường gặp:** WARN.
- **Sửa:** tìm biến cùng vai trò (6.2 cho biết biến nào đạt contrast trên nền đó) rồi gắn (6.3). Chọn theo
  **vai trò**, không theo mã hex gần nhất.
- **Bẫy:** màu trong hình minh hoạ, logo, ảnh không cần biến. Miễn có lý do.

### `chu-khong-style`
- **Đo gì:** layer chữ không gắn text style, trong khi file có text style. **Mức thường gặp:** WARN.
- **Sửa:** liệt kê style cùng cỡ (6.3b) rồi áp. Không style nào khớp nghĩa là chữ lệch thang, hoặc thang
  thiếu một nấc. Hỏi người dùng nên thêm style hay đưa chữ về nấc có sẵn.

### `thang-chu-roi`
- **Đo gì:** trong mỗi layer gốc được đo (thường là một màn): quá 2 họ font, quá 3 độ đậm, quá 7 cỡ chữ,
  hoặc cỡ chữ lẻ như 15,5px. Font icon (tên có "icon", "symbol"…) không tính. **Nguồn:** tri thức thiết kế:
  một sans + một mono, 2–3 độ đậm, thang 6–7 nấc. **Mức thường gặp:** WARN.
- **Sửa:** gom về text style của file (6.3b), mỗi vai trò đúng một style. Cỡ lẻ thường do kéo giãn layer
  chữ bằng công cụ Scale: đặt lại cỡ đúng nấc rồi áp style.
- **Bẫy:** trang brand được thêm một họ serif cho tiêu đề và độ đậm 700, ghi lý do vào DESIGN.md. Đo cả
  trang thì mỗi màn là một gốc riêng; chọn nhiều màn cùng lúc cũng vậy.

### `gradient-chu`
- **Đo gì:** layer chữ fill gradient. Contrast không đo ổn định, và thường chỉ là trang trí.
  **Mức thường gặp:** WARN.
- **Sửa:** fill đặc từ biến. Muốn nhấn thì dùng weight, cỡ chữ, khoảng trắng.

### `kinh-mo`
- **Đo gì:** effect `BACKGROUND_BLUR` (kính mờ). Chữ trên đó có contrast đổi theo thứ nằm phía sau, và
  hiệu ứng này nặng khi dựng thật. **Mức thường gặp:** WARN.
- **Sửa:** mặt nền đặc. Nếu thương hiệu cần kính mờ thì chỉ dùng cho bề mặt trang trí; chữ đặt trên một
  lớp nền đặc; ghi lý do miễn.

### `hue-semantic-gan-primary` (scope `palette`)
- **Đo gì:** hue OKLCH của success, warning, danger, info cách hue primary dưới 60°.
- **Nguồn:** tri thức phối màu. **Mức thường gặp:** WARN.
- **Sửa:** kéo hue semantic về vùng quen: success 140–155 · warning 60–85 · danger 25–35 · info 240–265.
  Không kéo được (primary xanh lá thì success cũng xanh lá) thì giữ, nhưng **luôn** đi kèm tín hiệu thứ hai
  (icon, chữ), chỉ dùng semantic cho trạng thái thật, và ghi lý do.

### `mu-mau-trung-nhau` (scope `palette`)
- **Đo gì:** success với danger, primary với success, sau khi mô phỏng protanopia, deuteranopia,
  tritanopia, cách nhau ΔE-OK < 0,1. **Mức:** luôn WARN (ngưỡng tự đặt).
- **Sửa:** thêm tín hiệu thứ hai: icon ✓/!, chữ "Lỗi", hình dạng, vị trí. Chỉ đổi màu thì chưa đủ.
  Nếu vẫn đổi màu thì đổi **độ sáng** để hai màu còn khác nhau khi mất sắc.

## 6. Đoạn code đo và sửa

Tất cả là thân hàm cho `execute_figma_code`. Thay `'…_ID'` bằng id thật. Đoạn **đọc** chạy thoải mái.
Đoạn **sửa** chỉ chạy khi người dùng đồng ý; mỗi lần là một bước undo.

### 6.1. Đọc từng đoạn của một layer chữ

`inspect_nodes` trả fontWeight, lineHeight, letterSpacing, textAutoResize, tên text style và biến
gắn cho cả layer; chữ trộn nhiều kiểu thì có thêm `segments` (cỡ, font, weight, màu). Cần line
height, letter spacing hay style riêng của **từng đoạn** thì đọc bằng code:

```js
const node = await figma.getNodeByIdAsync('TEXT_ID')
if (!node || node.type !== 'TEXT') throw new Error('Không phải layer chữ')
const segs = node.getStyledTextSegments(['fontSize', 'fontWeight', 'lineHeight', 'letterSpacing', 'textStyleId', 'fillStyleId'])
return {
  opacity: node.opacity,
  textAutoResize: node.textAutoResize,
  boundFill: node.boundVariables?.fills?.map((a) => a.id) ?? [],
  segments: segs.slice(0, 10).map((s) => ({ text: s.characters.slice(0, 40), size: s.fontSize, weight: s.fontWeight,
    lineHeight: s.lineHeight, letterSpacing: s.letterSpacing, textStyleId: s.textStyleId, fillStyleId: s.fillStyleId })),
}
```

### 6.2. Đo contrast, và tìm biến màu đạt trên một nền

Dùng để đo tay khi tool không đo được, hoặc để chọn biến trước khi sửa. `resolveForConsumer` trả giá
trị của biến theo đúng mode của frame chứa node.

```js
const K = figma.util.rgb('#000000') // giá trị mặc định của tham số, chỉ để rõ kiểu
const lin = (v = 0) => (v <= 0.04045 ? v / 12.92 : ((v + 0.055) / 1.055) ** 2.4)
const lum = (c = K) => 0.2126 * lin(c.r) + 0.7152 * lin(c.g) + 0.0722 * lin(c.b)
const ratio = (a = K, b = K) => (Math.max(lum(a), lum(b)) + 0.05) / (Math.min(lum(a), lum(b)) + 0.05)
// lớp bán trong suốt `top` (alpha 0–1) phủ lên `under`, kiểu source-over
const mix = (top = K, alpha = 1, under = K) => ({ r: top.r * alpha + under.r * (1 - alpha), g: top.g * alpha + under.g * (1 - alpha), b: top.b * alpha + under.b * (1 - alpha) })

const backdrop = figma.util.rgb('#1E293B') // nền ĐÃ TRỘN (lớp bán trong suốt thì mix trước)
const need = 4.5 // 3 cho chữ lớn, viền ô nhập, icon mang nghĩa
const node = await figma.getNodeByIdAsync('TEXT_ID')
if (!node || node.type === 'PAGE' || node.type === 'DOCUMENT') throw new Error('Sai id')
const pass = []
for (const v of await figma.variables.getLocalVariablesAsync('COLOR')) {
  const { value } = v.resolveForConsumer(node)
  if (typeof value !== 'object' || !('r' in value) || ('a' in value && value.a < 1)) continue
  const r = ratio(value, backdrop)
  if (r >= need) pass.push({ name: v.name, id: v.id, ratio: Math.round(r * 100) / 100 })
}
return pass.slice(0, 40)
```

Danh sách này cho biết **biến nào đạt**. Chọn biến theo vai trò, không theo số cao nhất.

### 6.3. Gắn fill vào biến

```js
const node = await figma.getNodeByIdAsync('TEXT_ID')
const variable = await figma.variables.getVariableByIdAsync('VARIABLE_ID')
if (!node || node.type !== 'TEXT' || !variable) throw new Error('Sai id')
await Promise.all(node.getStyledTextSegments(['fontName']).map((s) => figma.loadFontAsync(s.fontName)))
node.fills = [figma.variables.setBoundVariableForPaint({ type: 'SOLID', color: { r: 0, g: 0, b: 0 } }, 'color', variable)]
return { id: node.id, fill: variable.name }
```

Với frame hay shape thì bỏ dòng tải font. Với viền ô nhập thì gán vào `strokes` thay vì `fills`. Sửa
`chu-mo-bang-opacity` thì thêm `node.opacity = 1` trong cùng lần gọi.

### 6.3b. Liệt kê và áp text style

```js
const node = await figma.getNodeByIdAsync('TEXT_ID')
if (!node || node.type !== 'TEXT') throw new Error('Không phải layer chữ')
const styles = await figma.getLocalTextStylesAsync()
return styles.filter((s) => s.fontSize === node.fontSize).map((s) => ({ id: s.id, name: s.name, font: s.fontName, lineHeight: s.lineHeight }))
```

```js
const node = await figma.getNodeByIdAsync('TEXT_ID')
const style = await figma.getStyleByIdAsync('STYLE_ID')
if (!node || node.type !== 'TEXT' || !style || style.type !== 'TEXT') throw new Error('Sai id')
await figma.loadFontAsync(style.fontName)
await node.setTextStyleIdAsync(style.id)
return { id: node.id, style: style.name }
```

### 6.4. Nắn khoảng cách về bội số của 4

Chạy với `APPLY = false` để xem trước danh sách, rồi hỏi người dùng. Bỏ qua instance (sửa ở main
component) và giá trị đã gắn biến; xét đúng những thuộc tính `lech-nac-khoang-cach` đo.

```js
const APPLY = false
const root = await figma.getNodeByIdAsync('FRAME_ID')
if (!root || !('findAll' in root)) throw new Error('Cần id của một frame')
const out = []
for (const f of [root, ...root.findAll((n) => 'layoutMode' in n)]) {
  if (!('layoutMode' in f) || f.layoutMode === 'NONE') continue
  let inInstance = f.type === 'INSTANCE'
  for (let p = f.parent; p && !inInstance; p = p.parent) inInstance = p.type === 'INSTANCE'
  if (inInstance) continue
  const values = { paddingTop: f.paddingTop, paddingRight: f.paddingRight, paddingBottom: f.paddingBottom, paddingLeft: f.paddingLeft }
  if (f.layoutMode === 'GRID') Object.assign(values, { gridRowGap: f.gridRowGap, gridColumnGap: f.gridColumnGap })
  else if (f.primaryAxisAlignItems !== 'SPACE_BETWEEN') Object.assign(values, { itemSpacing: f.itemSpacing })
  if (f.layoutWrap === 'WRAP' && f.counterAxisSpacing !== null) Object.assign(values, { counterAxisSpacing: f.counterAxisSpacing })
  const bound = f.boundVariables ?? {}
  for (const [k, v] of Object.entries(values)) {
    if (v % 4 === 0 || k in bound) continue
    const next = Math.round(v / 4) * 4
    out.push(`${f.name} (${f.id}) · ${k}: ${v} → ${next}`)
    if (APPLY) Object.assign(f, { [k]: next })
  }
}
return out.slice(0, 100)
```

### 6.5. Bọc icon trong vùng bấm

```js
const SIZE = 40 // sàn 24 · desktop tối thiểu 32, mặc định 40 · mobile 44 · Android 48
const icon = await figma.getNodeByIdAsync('ICON_ID')
if (!icon || icon.type === 'PAGE' || icon.type === 'DOCUMENT') throw new Error('Sai id')
const parent = icon.parent
if (!parent || !('insertChild' in parent)) throw new Error('Không có layer cha')
const hit = figma.createFrame()
hit.name = `${icon.name} · vùng bấm`
hit.fills = []
hit.layoutMode = 'HORIZONTAL'
hit.primaryAxisAlignItems = 'CENTER'
hit.counterAxisAlignItems = 'CENTER'
hit.resize(Math.max(SIZE, icon.width), Math.max(SIZE, icon.height))
hit.primaryAxisSizingMode = 'FIXED'
hit.counterAxisSizingMode = 'FIXED'
parent.insertChild(parent.children.indexOf(icon), hit)
if (!('layoutMode' in parent) || parent.layoutMode === 'NONE') {
  hit.x = icon.x - (hit.width - icon.width) / 2
  hit.y = icon.y - (hit.height - icon.height) / 2
}
hit.appendChild(icon)
return { hit: hit.id }
```

Icon nằm trong instance thì làm việc này ở main component.

### 6.6. Đọc màu các stop của gradient nền

```js
const bg = await figma.getNodeByIdAsync('BACKDROP_ID')
if (!bg || !('fills' in bg) || bg.fills === figma.mixed) throw new Error('Sai id')
const g = bg.fills.find((p) => 'gradientStops' in p)
return g && 'gradientStops' in g ? g.gradientStops.map((s) => s.color) : null
```

Đưa từng màu vào `ratio` ở 6.2 (stop có alpha < 1 thì `mix` với nền bên dưới trước). Số thấp nhất là số của chữ.

### 6.7. Nắn một đoạn văn: line height, căn lề, bề rộng

Chạy với `APPLY = false` để xem trước, rồi hỏi người dùng. Chữ đã gắn text style thì sửa ở style (hoặc
áp style khác) thay vì sửa từng layer.

```js
const APPLY = false
const node = await figma.getNodeByIdAsync('TEXT_ID')
if (!node || node.type !== 'TEXT') throw new Error('Không phải layer chữ')
if (node.fontSize === figma.mixed) throw new Error('Chữ trộn nhiều cỡ: sửa từng đoạn hoặc áp text style')
const size = node.fontSize
const plan = {
  lineHeight: { unit: 'PIXELS', value: Math.round((size * 1.45) / 4) * 4 }, // 12→16 · 14→20 · 16→24 · 18→28
  textAlignHorizontal: 'LEFT',
  maxWidth: Math.round(size * 45), // khoảng 75 ký tự mỗi dòng
}
if (!APPLY) return { id: node.id, now: { lineHeight: node.lineHeight, align: node.textAlignHorizontal, width: node.width }, plan }
await Promise.all(node.getRangeAllFontNames(0, node.characters.length).map((f) => figma.loadFontAsync(f)))
node.lineHeight = plan.lineHeight
node.textAlignHorizontal = plan.textAlignHorizontal
if (node.width > plan.maxWidth) {
  const inAutoLayout = node.parent && 'layoutMode' in node.parent && node.parent.layoutMode !== 'NONE'
  if (inAutoLayout && node.layoutSizingHorizontal === 'FILL') node.maxWidth = plan.maxWidth
  else {
    node.textAutoResize = 'HEIGHT'
    node.resize(plan.maxWidth, node.height)
  }
}
return { id: node.id, lineHeight: node.lineHeight, width: node.width }
```

Nhãn in hoa ngắn (`viet-hoa`): sau khi load font như trên, `node.textCase = 'UPPER'` và
`node.letterSpacing = { unit: 'PERCENT', value: 6 }`. Nội dung đã gõ hoa sẵn thì chỉ cần thêm khoảng chữ.
