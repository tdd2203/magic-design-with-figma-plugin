# Pattern màn hình và thư viện quyết định

Hầu hết app chỉ cần một hoặc hai khung xương. **Chọn rồi dùng nhất quán**; đừng sáng tạo bố cục cho
từng màn hình. Một quy ước hơi dở áp dụng khắp nơi tốt hơn năm quy ước tối ưu cục bộ — mỗi ngoại lệ
là một lần người dùng phải học lại. Thứ tự ưu tiên: nhất quán **với chính sản phẩm** → với nền tảng
→ với thói quen chung (nút đóng góc phải trên, logo về trang chủ).

Mọi con số là điểm bắt đầu. Làm khác khi giải thích được bằng **lợi ích cho người dùng** (không phải
"trông đẹp hơn"), làm khác **nhất quán** ở mọi nơi, và **ghi lại** lý do trên trang Cover.

## Mục lục

1. Năm khung xương
2. Nút
3. Form
4. Bảng dữ liệu
5. Điều hướng
6. Popover, modal, drawer hay trang riêng
7. Tìm kiếm và lọc
8. Trạng thái trống
9. Hành vi không vẽ được → ghi chú cho dev
10. Bộ dữ liệu bẩn

---

## 1. Năm khung xương

| Khung | Dùng cho | Luật chính | Dựng trong Figma |
|---|---|---|---|
| **Danh sách + chi tiết** | Email, CRM, đơn hàng, ticket, học sinh | Chọn một mục **không** làm mất danh sách · chi tiết có link riêng · dưới 1024px tách thành hai màn hình có nút Back | Frame 1440 auto layout ngang: Nav 240 · List 320–400 · Detail FILL. Vẽ thêm bản 768 hoặc 390 cho hai màn hình tách |
| **Dashboard** | Tổng quan, màn hình đầu sau đăng nhập | **Tối đa 4 chỉ số lớn** hàng đầu · mỗi widget trả lời một câu hỏi viết ra được, bấm vào ra dữ liệu chi tiết · là màn hình để **đọc**, không phải để **làm** | Hàng KPI: auto layout ngang, 4 thẻ FILL. Widget: grid hoặc wrap, gap 16–24. Rộng tối đa 1440 |
| **Wizard nhiều bước** | Onboarding, quy trình có thứ tự bắt buộc | Hiện "Bước 2/4" · lùi lại không mất dữ liệu · **5–7 trường mỗi bước** · bước cuối xem lại toàn bộ. Không dùng cho việc hằng ngày, không dùng khi làm bước nào trước cũng được | Mỗi bước một frame; component Stepper có `State` = Done · Current · Upcoming |
| **Canvas** | Thời khoá biểu, sơ đồ, lịch, kéo thả | Công cụ ở cạnh · thuộc tính của đối tượng đang chọn ở panel phải · có hoàn tác/làm lại · tự lưu, hiện "Đã lưu lúc 10:32" | Toolbar và panel phải là component; ghi chú cho dev phần hoàn tác, phím zoom/pan |
| **Trang form** | Nhập liệu, cấu hình, hồ sơ | Một cột 480–640px · nhóm có tiêu đề · thanh lưu dính đáy nếu form dài hơn một màn hình · tự lưu nháp nếu quá 10 trường | Cột form là auto layout dọc, `maxWidth` 640. Thanh lưu: con cuối của frame màn hình, `layoutPositioning = 'ABSOLUTE'`, `constraints.vertical = 'MAX'`; `numberOfFixedChildren = 1` trên frame màn hình để prototype giữ thanh khi cuộn |

Phép thử cho dashboard: *nhìn 10 giây xong, người dùng sẽ làm gì khác đi?* Không trả lời được thì
widget đó chưa cần.

Dựng **màn hình khó nhất trước** — bảng nhiều cột nhất, form dài nhất, trang phân quyền. Component
sống được ở đó thì chỗ khác tự ổn.

Khung danh sách + chi tiết, đặt vào chỗ trống bên phải nội dung hiện có:

```js
const right = Math.max(0, ...figma.currentPage.children.map((n) => n.x + n.width))
const screen = figma.createFrame()
screen.name = 'Học sinh — danh sách + chi tiết'
screen.resize(1440, 900)
screen.layoutMode = 'HORIZONTAL'
screen.primaryAxisSizingMode = 'FIXED'
screen.counterAxisSizingMode = 'FIXED'
screen.x = right + 120
const column = (name = '', width = 0) => { // width 0 = FILL phần còn lại
  const f = figma.createFrame()
  f.name = name
  f.layoutMode = 'VERTICAL'
  f.resize(width || 100, 900)
  screen.appendChild(f)
  f.layoutSizingVertical = 'FILL'
  f.layoutSizingHorizontal = width ? 'FIXED' : 'FILL'
  return f.id
}
return { screenId: screen.id, columns: [column('Nav', 240), column('List', 360), column('Detail')] }
```

Tách các cột bằng khoảng trắng → đường kẻ mảnh → nền nhạt, theo thứ tự đó. Bóng đổ chỉ cho thứ thật
sự nổi lên trên (dropdown, modal, toast).

## 2. Nút

| Cấp | Hình | Mỗi màn hình |
|---|---|---|
| Chính (Primary) | Nền màu đặc | **Đúng 1** |
| Phụ (Secondary) | Viền, nền trong | Nên ≤ 3 |
| Thứ ba (Tertiary) | Chỉ chữ | Hành động ít quan trọng |
| Phá huỷ (Destructive) | Nền hoặc chữ đỏ | Chỉ khi cần, **không đặt cạnh nút chính** |

- Hai hành động thật sự ngang hàng → cả hai làm nút viền, bỏ nút đặc.
- **Nhãn = động từ + danh từ**, 1–3 từ: "Lưu thay đổi", "Gửi báo cáo", "Xoá 12 học sinh". Người
  dùng đọc nút, không đọc đoạn văn phía trên.
- Vị trí: form trên trang → nút chính bên trái; modal → nút chính bên phải; nút phá huỷ cách nút
  chính ≥ 24px. **Chọn một quy ước và dùng khắp sản phẩm** quan trọng hơn chọn "đúng".
- Nút chỉ có icon: bắt buộc tooltip và tên cho trình đọc màn hình; chỉ dùng icon nghĩa phổ quát.
  Không ai đoán được icon nào nghĩa là "chốt sổ".

## 3. Form

| Quy tắc | Vì sao |
|---|---|
| **Một cột**; hai cột chỉ cho cặp ngắn đi liền (thành phố / mã bưu điện, ngày / giờ) | Mắt quét dọc nhanh hơn zigzag |
| Bề rộng ô phản ánh nội dung | Ô mã bưu điện hẹp hơn ô địa chỉ là gợi ý ngầm rất mạnh |
| **Nhãn phía trên ô** | Quét nhanh nhất, dịch sang tiếng khác không vỡ layout, mobile không phải đổi |
| Placeholder chỉ là **ví dụ định dạng** ("vd: 0912 345 678"), không thay nhãn | Gõ vào là placeholder biến mất |
| Đa số ô bắt buộc → đánh dấu ô **tuỳ chọn** | Ít nhiễu hơn |
| Hướng dẫn và lỗi nằm **dưới** ô, chữ nhỏ | Đọc đúng lúc cần |

Khoảng cách: nhãn ↔ ô của nó 8 · ô ↔ ô trong nhóm 16 · nhóm ↔ nhóm 32. Khoảng trong nhóm luôn nhỏ
hơn khoảng giữa nhóm, tối thiểu gấp đôi — nhãn cách ô bằng ô cách ô là mắt ghép nhầm.

**Bớt trường** là cách tối ưu form hiệu quả nhất. Hỏi từng trường: hệ thống tự biết được không (ngày
tạo, người tạo, mã tự sinh)? đoán đúng mặc định 80% được không (ngày là hôm nay, lớp là lớp vừa thao
tác)? hỏi sau được không? Trên canvas, vẽ ô đã **điền sẵn giá trị mặc định**, không để trống.

Hành vi → ghi chú cho dev: không tự xoá dữ liệu người dùng đã nhập kể cả khi sai định dạng · cho dán,
kể cả ô mật khẩu · tự định dạng số điện thoại, tiền, ngày khi gõ nhưng nhận mọi cách gõ · form đơn
giản Enter là gửi, form nhiều dòng Ctrl/Cmd+Enter.

## 4. Bảng dữ liệu

Thành phần quan trọng nhất của app nghiệp vụ, và hay bị làm ẩu nhất.

| Hạng mục | Mặc định |
|---|---|
| Căn | Cột đầu là định danh (tên, mã), ghim trái khi cuộn ngang · **số căn phải** · chữ căn trái · **không căn giữa**, trừ icon trạng thái đơn lẻ · tiêu đề cột căn theo nội dung cột |
| Số | Chữ số đều bề rộng (tabular). Plugin API không bật được — bật trong text style bằng tay, hoặc ghi chú cho dev |
| Cột | Hiện 5–9 cột; còn lại bật tắt qua menu "Cột hiển thị" |
| Dòng | 32 / 40 / 48 theo mật độ (collection `Density` ba mode Compact · Default · Comfortable, skill `layout-type` reference/sizing-surfaces.md §6; gói Figma không cho thêm mode thì làm trục variant `Density`) · chỉ kẻ **ngang** mảnh · sọc xen kẽ chỉ khi rộng hơn 8 cột · **hover dòng đổi nền — bắt buộc** |
| Ô dài | Cắt bằng "…", không xuống dòng làm vỡ chiều cao; hiện đủ khi hover (ghi chú) |
| Hành động trên dòng | 1–3 hành động → icon hiện khi hover (mobile luôn hiện) · từ 4 → menu "…" · **xoá không bao giờ là icon đầu tiên**, nằm trong menu |
| Sắp xếp | Bấm tiêu đề: tăng → giảm → mặc định, có mũi tên chỉ chiều |
| Chọn nhiều | Checkbox cột đầu · checkbox tiêu đề chỉ chọn **trang hiện tại** và nói rõ: "Đã chọn 25 dòng trên trang này. Chọn tất cả 1.243 dòng?" · thanh hành động hàng loạt nói số lượng: "Xoá 25 mục" |
| Phân trang | Mặc định phân trang (người dùng cần định vị, quay lại, gửi link) · 25 hoặc 50 dòng, cho chọn 10/25/50/100 · cuộn vô hạn chỉ cho nội dung duyệt liên tục (feed, thư viện ảnh) |
| Chi tiết tạo khác biệt | Luôn hiện tổng số ("1.243 học sinh") · **ba loại bảng trống, ba câu khác nhau** (§8) · nút xuất Excel/CSV — với app nghiệp vụ ở Việt Nam gần như luôn cần |

Ô chữ co giãn có cắt "…" và ô số căn phải, trong một component `Table row` auto layout ngang:

```js
const row = await figma.getNodeByIdAsync(NODE_ID)
if (row?.type !== 'COMPONENT' && row?.type !== 'FRAME') throw new Error('NODE_ID phải là frame/component auto layout ngang')
await figma.loadFontAsync({ family: 'Inter', style: 'Regular' })
const cell = (chars = '', width = 0, right = false) => { // width 0 = FILL
  const t = figma.createText()
  t.fontName = { family: 'Inter', style: 'Regular' }
  t.fontSize = 14
  t.lineHeight = { unit: 'PIXELS', value: 20 }
  t.characters = chars
  row.appendChild(t)
  t.layoutSizingHorizontal = width ? 'FIXED' : 'FILL'
  if (width) t.resize(width, t.height)
  t.textAutoResize = 'HEIGHT'
  t.textAlignHorizontal = right ? 'RIGHT' : 'LEFT'
  t.textTruncation = 'ENDING' // dài thì cắt "…", không xuống dòng
  t.maxLines = 1
  return t.id
}
return [cell('ĐẶNG THỊ NGỌC ẨN'), cell('10A1', 96), cell('1.234.567', 120, true)]
```

## 5. Điều hướng

Ba cấp điều hướng: toàn cục tối đa 7 mục (trần 9) · tab trong trang tối đa 5 · hành động trên một
dòng 3 cái rồi menu "…". Cần tới cấp thứ tư là mô hình đối tượng đang sai. Lý do và cách đặc tả luồng:
skill `design-workflow`, reference/brief.md §2.3.

| | Sidebar dọc | Topbar ngang |
|---|---|---|
| Số mục | 5–15 | 3–7 |
| Thêm mục về sau | Dễ | Khó — hết chỗ ngang |
| Chiếm | Chiều ngang (quý với bảng) | Chiều dọc (quý với form) |

- App nghiệp vụ mặc định: **sidebar dọc thu gọn được**, 240–280px khi mở, 56–64px khi thu gọn
  (chỉ icon + tooltip). Trong Figma: một set `Sidebar` với trục `Mode` = Expanded · Collapsed;
  `Nav item` là component riêng có `State` = Default · Hover · Current · Focus.
- Mục đang ở: **nền hoặc thanh chỉ báo**, không chỉ đổi màu chữ.
- Tối đa hai cấp trong sidebar; cấp ba thành tab trong trang. Breadcrumb khi sâu hơn hai cấp, cấp
  nào cũng bấm được.
- Logo về màn hình chính. **Không dùng hamburger trên desktop.**
- Sắp mục theo **tần suất dùng**, không theo bảng chữ cái hay sơ đồ phòng ban. Quá 12 mục thì nhóm
  có tiêu đề. Cài đặt, Trợ giúp, Hồ sơ ở cuối, tách khỏi nhóm chính.

## 6. Popover, modal, drawer hay trang riêng

```
Cần nhập hoặc xem bao nhiêu?
├─ Một lựa chọn nhanh từ danh sách ngắn                       → POPOVER / DROPDOWN
├─ Dưới 3 trường, không rẽ nhánh, cần giữ ngữ cảnh phía sau   → MODAL (400–560)
├─ Xem chi tiết một mục, có thể cuộn, giữ danh sách phía sau  → DRAWER phải (400–560)
├─ Việc dài, nhiều bước, cần link riêng                       → TRANG RIÊNG
└─ Chỉ báo tin, không cần quyết định                          → TOAST / BANNER
```

**Chọn trang riêng nếu bất kỳ điều nào đúng:** cần gửi link · quá 5 trường · có rẽ nhánh (chọn A
thì hiện ô X) · có thể cần mở nhiều cái cùng lúc · nội dung dài hơn một màn hình.

| | Modal | Drawer |
|---|---|---|
| Bề rộng | 400 (xác nhận) · 560 (form ngắn) · 720 (nội dung phức tạp), không rộng hơn | 400–560 |
| Hướng | Giữa màn hình | Từ phải cho chi tiết, từ trái cho điều hướng |
| Figma | Set `Modal` trục `Size` = S · M · L; header và hàng nút cố định, thân cuộn | Set `Drawer`; danh sách phía sau vẫn thấy — đó là lý do chọn drawer |
| Luật | Không bao giờ modal trong modal · bo 8–12 · có bóng vì nó nổi lên thật | Bấm ↑ ↓ chuyển sang mục kế ngay trong drawer (ghi chú) |

Ghi chú cho dev trên component Modal: Escape đóng · bấm ra ngoài chỉ đóng khi **chưa có thay đổi**,
nhập dở thì hỏi · mở ra focus vào ô đầu · focus bị nhốt trong modal · đóng thì focus về nút đã mở nó.
Hộp xác nhận: focus mặc định vào nút **an toàn**, nhãn nút nói việc sẽ xảy ra.

Trong prototype, mở modal bằng action `navigation: 'OVERLAY'` từ nút; chuyển động 200–300ms ease-out.

## 7. Tìm kiếm và lọc

| Số bản ghi | Cần |
|---|---|
| < 20 | Không cần gì |
| 20–100 | Ô tìm đơn giản |
| 100–1.000 | Tìm + 2–4 bộ lọc |
| > 1.000 | Tìm + bộ lọc + lưu bộ lọc thành preset có tên |

- **Ô tìm** ở trên cùng bên trái vùng nội dung, không giấu sau icon. Placeholder nói tìm được gì:
  "Tìm theo tên, mã học sinh, lớp". Có nút xoá nhanh. Hiện số kết quả: "12 kết quả cho 'nguyễn'".
- **Bộ lọc** áp ngay, không cần nút "Áp dụng" trừ khi truy vấn nặng · bộ lọc đang áp hiện thành chip
  có nút xoá · từ hai bộ lọc có "Xoá tất cả".
- **Không có kết quả**: nói rõ điều kiện nào không ra gì · nút xoá bộ lọc · gợi ý bỏ bộ lọc nào hoặc
  kiểm chính tả.
- Ghi chú cho dev trên ô tìm: không phân biệt hoa thường · **không phân biệt dấu** — gõ `nguyen` phải
  ra "Nguyễn", gõ `dat` phải ra "Đạt" (chữ đ không tự tách dấu khi chuẩn hoá Unicode, phải đổi
  riêng) · chờ khoảng 300ms sau lần gõ cuối mới tìm · phím `/` hoặc Ctrl/Cmd+K nhảy vào ô · bộ lọc
  và từ khoá nằm trong URL.

## 8. Trạng thái trống

Màn hình trắng là thất bại; trạng thái trống là chỗ dạy người dùng bước đầu tiên. Đủ ba thứ:

1. **Một câu nói chỗ này để làm gì** — không phải "Chưa có dữ liệu".
2. **Nút hành động chính** — đúng nút sẽ dùng khi đã có dữ liệu, không phải nút riêng cho lúc trống.
3. **Một đường thoát**: hướng dẫn, dữ liệu mẫu, hoặc nhập từ Excel.

Bảng có **ba kiểu trống, ba câu khác nhau** — làm set `Empty state` với trục `Kind`:

| Kind | Câu | Nút | Đường thoát |
|---|---|---|---|
| No data | "Lớp 10A1 chưa có học sinh. Thêm học sinh để bắt đầu điểm danh và nhập điểm." | Thêm học sinh | Nhập từ Excel |
| Filtered out | "Không có học sinh nào khớp 'Lớp 10A1' và 'Vắng trên 3 buổi'." | Xoá bộ lọc | Bỏ bộ lọc "Vắng trên 3 buổi" |
| Load failed | "Chưa tải được danh sách học sinh vì mất kết nối tới máy chủ. Dữ liệu của bạn không bị ảnh hưởng." | Thử lại | Mã hỗ trợ: #A7K2 |

Nút trong empty state là **instance của Button** (expose để đổi nhãn — [components.md §6](components.md#6-instance-lồng-nhau-override-và-expose)),
không vẽ lại nút.

## 9. Hành vi không vẽ được → ghi chú cho dev

Canvas chỉ vẽ được hình. Những thứ dưới đây ghi thành annotation trên node liên quan, kèm khung
`Dev note / …` nhìn thấy được ([components.md §10](components.md#10-ghi-chú-cho-dev-trên-canvas)).

| Hành vi | Ghi trên |
|---|---|
| Vòng focus chỉ hiện khi dùng bàn phím; Tab đi đúng thứ tự thị giác | Mọi set có variant Focus; màn hình |
| Nhốt focus trong modal, Escape đóng, trả focus về nút đã mở | Modal, drawer |
| Tên cho trình đọc màn hình (icon button, ảnh có nghĩa), mô tả thay thế cho ảnh | Icon button, ảnh |
| Bộ lọc, sắp xếp, trang, tab đang chọn nằm trong URL; Back làm đúng; quay lại còn bộ lọc và vị trí cuộn | Màn hình danh sách |
| Chờ ~300ms sau lần gõ cuối mới tìm; tìm không phân biệt dấu | Ô tìm |
| Đổi trạng thái nút ngay khi bấm; khoá bấm lặp; loader vùng lớn hiện sau ~0,3 giây | Button, vùng tải |
| Thời điểm kiểm lỗi form ([ux-copy.md §8](ux-copy.md#8-câu-chữ-trong-form)) | Input |
| Tự lưu nháp khi quá 10 trường; cảnh báo khi rời trang giữa chừng | Trang form |
| Hoàn tác/làm lại ≥ 20 bước; "Đã lưu lúc HH:mm" | Canvas |
| Tiêu đề bảng dính khi dài hơn 15 dòng; phân trang phía server; nhớ lựa chọn "Cột hiển thị" của từng người | Bảng |
| Nhớ trạng thái thu gọn sidebar | Sidebar |
| Gợi ý phím tắt nào hiện trên giao diện thì phím đó phải chạy | Mọi chỗ hiện gợi ý |
| Màn hình nhập liệu hằng ngày: Tab sang ô kế, Enter xuống dòng kế, dán nhiều dòng từ Excel | Màn hình nhập liệu |
| Tôn trọng thiết lập giảm chuyển động của hệ điều hành; zoom 200% không vỡ layout | Trang Cover (áp cho cả sản phẩm) |

## 10. Bộ dữ liệu bẩn

Đổ vào **ngay sau khi dựng khung**, trước khi làm đẹp. Bộ giá trị đầy đủ (chuỗi, tiếng Việt chồng dấu,
số, ngày, số lượng) và cách đổ vào cả màn hình: skill `design-workflow`, reference/screen-states.md §2.
Ô trống thì component phải hiện một thứ có chủ đích ("—", "Chưa có"), **không bao giờ** chữ `null`,
`undefined`, `NaN`.

Nhân bản một instance có property TEXT thành một bảng thử, đặt riêng ngoài màn hình của người dùng:

```js
const src = await figma.getNodeByIdAsync(INSTANCE_ID)
if (src?.type !== 'INSTANCE') throw new Error('INSTANCE_ID phải là một instance')
const props = src.componentProperties
const key = Object.keys(props).find((k) => props[k].type === 'TEXT')
if (!key) throw new Error('Instance này không có property TEXT')
for (const t of src.findAllWithCriteria({ types: ['TEXT'] })) {
  const fonts = t.fontName === figma.mixed ? t.getStyledTextSegments(['fontName']).map((s) => s.fontName) : [t.fontName]
  for (const f of fonts) await figma.loadFontAsync(f)
}
const DIRTY = ['ĐẶNG THỊ NGỌC ẨN', 'Nguyễn Thị Hường', 'A', 'Trường'.repeat(34).slice(0, 200),
  'Café ☕ & <Đơn #12> "VIP"', '', '-1.234.567.890', '0', '01/01/1900', '31/12/2099']
const board = figma.createFrame()
board.name = `Dữ liệu bẩn — ${src.name}`
board.layoutMode = 'VERTICAL'
board.itemSpacing = 16
board.paddingTop = board.paddingBottom = board.paddingLeft = board.paddingRight = 24
board.primaryAxisSizingMode = 'AUTO'
board.counterAxisSizingMode = 'AUTO'
board.x = Math.max(0, ...figma.currentPage.children.filter((n) => n !== board).map((n) => n.x + n.width)) + 120
const failed = []
for (const value of DIRTY) {
  const copy = src.clone()
  board.appendChild(copy)
  try { copy.setProperties({ [key]: value }) } catch (e) { failed.push(value); copy.remove() }
}
return { boardId: board.id, failed }
```

Rồi `screenshot` bảng thử và `audit_design` với `node_id` là bảng thử (`tran-chu` bắt chữ tràn khỏi
khung cắt, kể cả dấu tiếng Việt bị xén). Mỗi dòng phải có câu trả lời thiết kế: xuống dòng, cắt "…",
hay giới hạn bề rộng (`maxWidth` + `textTruncation`) — rồi vẽ đúng câu trả lời đó vào component. Font
phải có đủ glyph tiếng Việt: nhìn screenshot xem dấu có bị vẽ bằng font khác không; còn
`hasMissingFont` là `true` nghĩa là file dùng một font máy này không có — báo người dùng, đừng tự đổi.
Xong thì hỏi người dùng giữ hay xoá bảng thử.
