# Kiểm màu: 8 bước, các bẫy khi đo, danh sách cấm

Không nguồn nào thay được phép đo trên màn hình thật. Bảng token đạt chưa có nghĩa là màn hình đạt:
badge có nền trong suốt, chữ bị giảm opacity, lớp chồng lên nhau đều làm số thật khác số trên giấy.

## 0. Hai lần `audit_design`

`audit_design` **chỉ đọc**, không bao giờ sửa file. Chạy theo thứ tự này:

1. **`audit_design {scope: "palette"}`** trên các biến màu cục bộ, theo từng mode. Rẻ, và sửa ở gốc thì
   mọi màn hình được sửa theo. Các rule: `bang-mau-tuong-phan` (cặp chữ/nền và chữ-trên-nút/nút dưới
   4,5) · `hue-semantic-gan-primary` (semantic cách primary dưới 60°) · `mu-mau-trung-nhau` (WARN,
   ΔE-OK < 0,1 khi mô phỏng mù màu) · `den-trang-thuan-token` (mode Dark có nền `#000` hoặc chữ `#FFF`).
2. **`audit_design {node_id, scope: "design"}`** trên màn hình thật, ở **cả hai mode**: bản Light và bản
   sao đặt mode Dark ([tokens-and-modes.md §6](tokens-and-modes.md)). Các rule liên quan tới màu:
   `tuong-phan-chu` (đo trên nền **đã trộn**) · `ky-hieu-mo` · `chu-khong-do-duoc` (SKIP) ·
   `chu-mo-bang-opacity` · `ranh-gioi-o-nhap` · `nen-den-thuan` · `chu-trang-thuan` · `mau-khong-bien` ·
   `gradient-chu` · `kinh-mo`.

Mỗi rule đo gì, ngưỡng, nguồn, cách sửa, và cách đọc `summary`, bốn nhóm, `notMeasured`: skill
`design-audit` (SKILL.md và reference/rules.md). Ba điều không được quên khi báo cáo màu:
- **`SKIP` nghĩa là "không đo được", không bao giờ là "đạt".** Kiểm bằng `screenshot` hoặc nói rõ với
  người dùng.
- Chép nguyên `notMeasured` vào báo cáo.
- Nêu ngưỡng đã dùng kèm nguồn (kết quả có trả về). Đừng tự đặt ngưỡng khác.

## 1. Tám bước kiểm, theo thứ tự

Bước 1–4 loại lỗi chết người: trượt thì làm lại bảng màu. Bước 5–8 là tinh chỉnh: trượt thì chỉnh ramp.

| # | Kiểm | Đạt khi | Làm trên Figma | Trượt thì |
|---|---|---|---|---|
| 1 | **Contrast bằng số**: mọi cặp chữ/nền *thật sự xuất hiện* | 100% cặp qua AA: chữ thường 4,5 · chữ lớn 3 · viền ô nhập, icon, thành phần giao diện 3 | `audit_design` palette rồi design, ở cả hai mode; bảng số §2 | Làm lại bảng màu |
| 2 | **Thang xám** | Vẫn thấy đâu là nút chính, chữ phụ, vùng được chọn | Bản sao tạm + lớp `SATURATION`, `screenshot` (§2) | Sửa layout, không sửa màu |
| 3 | **Mù màu**: deuteranopia, protanopia, tritanopia | Success và danger vẫn phân biệt được | `mu-mau-trung-nhau` + kiểm tín hiệu thứ hai bằng `inspect_nodes` | Thêm icon, hình, chữ; không đổi màu |
| 4 | **Dark mode** | Nền không đen thuần, chữ không trắng thuần, primary sáng lên, không dựa bóng đổ | Bản sao mode Dark, `screenshot`, `audit_design` design | Làm lại ánh xạ token |
| 5 | **Nội dung thật**: header, sidebar, bảng 20 dòng, 3 loại nút, form đang báo lỗi, empty state, modal | Nhìn 5 phút không mệt, mắt tự tìm tới nút chính | Dựng một màn thử bằng biến Semantic; `noi-dung-gia`, `ro-ri-gia-tri` | Hạ chroma primary |
| 6 | **Đủ trạng thái**: nút có default · hover · pressed · focus · disabled · loading; ô nhập có default · hover · focus · disabled, thêm error · success | Che nhãn vẫn phân biệt được; hover lệch ≥ 1 nấc, **không** bằng opacity | `inspect_nodes` trên component set, đối chiếu biến của từng biến thể | Tinh chỉnh ramp |
| 7 | **Điều kiện thực tế**: điện thoại ngoài nắng, laptop giảm sáng 30%, màn rẻ | Ranh giới giữa các vùng vẫn thấy | Tính ΔL giữa các bề mặt cạnh nhau (§2) | Tăng chênh lệch hoặc thêm viền |
| 8 | **Độ bền** | Còn ≥ 1 hue dự phòng cho trạng thái hay gói sản phẩm mới | Liệt kê hue đang dùng, tìm khoảng trống ≥ 60° | Gom bớt màu đang dùng |

## 2. Cách làm từng bước

### Bước 1: bảng số contrast từ biến Semantic

`audit_design` palette đã báo cặp nào trượt. Khi cần **bảng số đầy đủ** cho báo cáo, chạy đoạn sau
(dán helper ở [color-theory.md §7](color-theory.md) lên trước). Đoạn này chỉ đọc. Tên biến theo hệ dựng
ở [tokens-and-modes.md §4](tokens-and-modes.md); file dùng tên khác thì sửa `PAIRS`.

```js
const sem = (await figma.variables.getLocalVariableCollectionsAsync()).find((c) => c.name === 'Semantic')
if (!sem) throw new Error('Chưa có collection Semantic')
async function resolve(value) { // đi theo alias tới giá trị thật (mode mặc định của collection đích)
  while (typeof value === 'object' && 'type' in value && value.type === 'VARIABLE_ALIAS') {
    const v = await figma.variables.getVariableByIdAsync(value.id)
    if (!v) return null
    const c = await figma.variables.getVariableCollectionByIdAsync(v.variableCollectionId)
    if (!c) return null
    value = v.valuesByMode[c.defaultModeId]
  }
  return value
}
const color = {} // 'Light text/muted' → {r,g,b}
for (const id of sem.variableIds) {
  const v = await figma.variables.getVariableByIdAsync(id)
  if (!v || v.resolvedType !== 'COLOR') continue
  for (const m of sem.modes) {
    const val = await resolve(v.valuesByMode[m.modeId])
    if (val && typeof val === 'object' && 'r' in val) color[m.name + ' ' + v.name.slice(6)] = val
  }
}
const PAIRS = [ // [chữ hoặc viền, nền, ngưỡng]
  ['text/primary', 'bg/page', 4.5], ['text/primary', 'bg/surface', 4.5], ['text/secondary', 'bg/subtle', 4.5],
  ['text/muted', 'bg/page', 4.5], ['text/muted', 'bg/surface', 4.5], ['text/on-action', 'action/default', 4.5],
  ['text/on-action', 'action/hover', 4.5], ['action/default', 'bg/subtle', 4.5], ['action/subtle-text', 'action/subtle-bg', 4.5],
  ['border/input', 'bg/subtle', 3], ['border/input', 'bg/surface', 3], ['focus-ring', 'bg/page', 3],
]
for (const s of ['success', 'warning', 'danger', 'info']) PAIRS.push([s + '/default', 'bg/subtle', 4.5], [s + '/default', s + '/subtle-bg', 4.5])
const rows = []
for (const m of sem.modes) {
  for (const [fg, bg, min] of PAIRS) {
    const a = color[m.name + ' ' + fg], b = color[m.name + ' ' + bg]
    if (!a || !b) { rows.push(m.name + ' · thiếu biến ' + fg + ' hoặc ' + bg); continue }
    const r = contrast(a, b)
    rows.push(m.name + ' · ' + fg + ' / ' + bg + ' = ' + r.toFixed(2) + (r < min ? '  TRƯỢT (cần ' + min + ')' : ''))
  }
  const page = color[m.name + ' bg/page'], surface = color[m.name + ' bg/surface']
  if (!page || !surface) continue
  const dL = Math.abs(toOklch(page).L - toOklch(surface).L)
  rows.push(m.name + ' · ΔL page/surface = ' + dL.toFixed(3) + (dL < 0.04 ? '  (thẻ cần viền)' : ''))
}
return rows
```

Số này là số **trên giấy**. Số thật trên màn hình lấy từ `audit_design {scope: "design"}`, vì nó tính cả
opacity và lớp chồng.

### Bước 2: thang xám trên bản sao tạm

Không sửa frame gốc. Nhân bản vào một khung tạm ở chỗ trống, phủ lên một lớp xám blend mode `SATURATION`
(giữ độ sáng từng điểm, bỏ sắc độ), `screenshot`, rồi xoá khung tạm. Việc này thêm layer tạm vào file nên
hỏi người dùng trước. Code tạo và xoá khung tạm: skill `design-audit`, reference/critique.md §2.1.

Hỏi: nếu che hết màu, nút chính có còn là thứ nổi nhất không? Chữ phụ có lùi lại không? Mục đang chọn
có còn nhận ra không? Nếu phải nhờ màu mới phân biệt được thì đó là lỗi **thứ bậc**: sửa bằng cỡ chữ,
độ đậm, khoảng cách, nền, đừng sửa bằng màu. Làm bước này sớm, ngay trên bản nháp đầu tiên, thì rẻ nhất.

### Bước 3: mù màu

- `mu-mau-trung-nhau` chỉ là WARN: hai màu gần nhau khi mô phỏng chưa chắc là lỗi, **nếu** có tín hiệu
  thứ hai.
- Dùng `inspect_nodes` trên badge, alert, ô nhập lỗi để kiểm: có icon không? Có chữ nói trạng thái
  không? Trạng thái lỗi của ô nhập có dòng chữ giải thích không?
- Sửa bằng cách thêm hình hoặc chữ. Đừng đổi màu semantic để "né" mù màu, vì nghĩa của màu sẽ mất.

### Bước 4: dark mode

Nhân bản màn hình, đặt mode Dark ([tokens-and-modes.md §6](tokens-and-modes.md)), chụp cả hai bản cạnh
nhau, rồi chạy `audit_design` design trên bản Dark. Soát thêm bằng mắt:
- Thẻ có tách khỏi nền nhờ độ sáng không? Bóng đổ trên nền tối gần như vô hình.
- Primary có loé không? Nếu có, hạ chroma nấc 400.
- Ảnh và logo có viền trắng hay nền trắng lộ ra không?
- Có layer nào đang tô màu viết cứng, nên không đổi theo mode, không? (`mau-khong-bien`)

### Bước 5: nội dung thật

Màn thử phải có đủ: header, sidebar, bảng 20 dòng, ba loại nút (chính, phụ, thứ ba), một form đang báo
lỗi, một empty state, một modal. Dùng nội dung thật (tên người, số tiền, ngày tháng), không lorem
ipsum. `audit_design` bắt `noi-dung-gia` và `ro-ri-gia-tri`. Nhìn 5 phút: mắt phải tự tìm tới nút
chính. Màn hình mệt và loè loẹt thì hạ chroma primary (hệ số `K` trong script) chứ đừng đổi hue.

### Bước 6: đủ trạng thái

`inspect_nodes` trên component set để lấy danh sách biến thể, rồi `execute_figma_code` đọc
`boundVariables` của fill và stroke từng biến thể:
- Hover trỏ `action/hover`, pressed trỏ `action/active`. Lệch ≥ 1 nấc; trên mobile pressed lệch ≥ 2
  nấc vì không có hover.
- **Không** làm hover hay pressed bằng `opacity` của layer: nó làm cả chữ nhạt đi.
- Focus: vòng 2px bằng `color/focus-ring`, **cách nút 2px**, ≥ 3:1 với nền xung quanh. Focus phải là
  một vòng tách khỏi nút, không chỉ là nút đổi sang màu đậm hơn. Vòng cùng màu primary vẫn được, vì
  khoảng hở 2px tách nó khỏi nút.
- Disabled dùng `bg/disabled` và `text/disabled`, không dùng opacity, và phải có chỗ nói **vì sao** vô
  hiệu.
- Loading giữ nguyên bề rộng nút.
- Ô nhập có error (viền `danger/default` + icon + chữ) và success.
- Cách dựng các biến thể: xem skill **components-states**.

### Bước 7: điều kiện thực tế

Figma không mô phỏng được nắng hay màn rẻ, nên dùng số thay cho mắt:
- Hai bề mặt cạnh nhau chênh dưới khoảng 4% độ sáng (ΔL OKLCH < 0,04) thì ngoài nắng coi như một mặt.
  Ở Light, `bg/page` và `bg/surface` của hệ mặc định chỉ chênh 0,015, nên thẻ bắt buộc có
  `border/default` hoặc bóng.
- Mobile: chữ thường nhắm 5:1.
- Nếu người dùng có điện thoại, gợi ý họ mở bản prototype ngoài trời. Đây là mục `notMeasured`.

### Bước 8: độ bền

Liệt kê hue đang dùng (primary, bốn semantic, accent) và tìm khoảng trống ≥ 60° cho một trạng thái hay
một gói sản phẩm mới. Hết chỗ thì gom bớt: có thật cần accent không? Info có thể dùng ramp primary không?

## 3. Các bẫy khi đo trong Figma

1. **Mode kế thừa.** Frame con đặt mode riêng sẽ đè mode của cha. Bản sao giữ nguyên mode của bản gốc.
   Trước khi đo, kiểm `node.resolvedVariableModes`, hoặc `variable.resolveForConsumer(node)` cho một
   biến cụ thể.
2. **Nền phải cộng dồn.** Opacity của paint × opacity của layer × các lớp nằm dưới. Badge có nền tint
   trong suốt làm contrast tụt khoảng 0,3–0,7 điểm so với đo trên màu gốc. Ưu tiên nền tint **đặc**
   (`<status>/subtle-bg`) thay vì màu có alpha. `audit_design` design đo trên màu đã trộn;
   `audit_design` palette thì không nhìn thấy alpha đặt trên layer.
3. **Ngưỡng chữ lớn tính bằng px thật:** 24px, hoặc 18,66px khi weight ≥ 700. "Semi Bold" (600) chưa đủ
   đậm. `inspect_nodes` không trả weight; đọc `textNode.fontWeight` bằng `execute_figma_code`
   (kết quả là `figma.mixed` nếu chữ trộn nhiều weight).
4. **Viền ô nhập là phép đo riêng.** Quét chữ không bắt được nó. `ranh-gioi-o-nhap` chỉ đo những layer
   mà nó nhận ra là ô nhập. Đặt tên layer rõ ràng (`Input`, `Select`, `Textarea`) để công cụ và người đọc
   file đều nhận ra. Ô nào nghi bị bỏ sót thì đo tay: màu viền (hoặc nền ô) với nền bên ngoài, ngưỡng 3:1.
5. **Chữ trên ảnh, gradient, blur** luôn là `SKIP`. Chụp màn hình và nhìn; cần thì thêm lớp phủ tối
   40–60% (paint style, xem [tokens-and-modes.md §9](tokens-and-modes.md)).
6. **APCA chỉ để tham khảo.** Nếu dùng: đoạn văn thân bài cần Lc 75; Lc 60 là cho chữ nội dung không
   phải thân bài. WCAG 2 vẫn là mốc đo và là thứ `audit_design` dùng.

## 4. Danh sách cấm, một trang

```
TỶ LỆ       90% neutral · 7% primary · 3% semantic + accent — chỗ có màu là chỗ bấm được
THỨ TỰ      neutral → primary → semantic → ≤ 1 accent → ramp 11 nấc → token vai trò
RAMP        OKLCH · chroma hình chuông · dịch hue 6–12° theo hue gốc · đo lại nấc nút
CONTRAST    chữ thường 4,5 · chữ lớn (24px / 18,66px khi weight ≥ 700) 3 · viền ô nhập, icon 3
            đọc lâu nhắm 12–16, đừng 21 · đo trên màu ĐÃ TRỘN nếu có trong suốt
DARK        nền không #000 · chữ không #FFF · primary sáng lên ~2 nấc · không dựa bóng đổ
FIGMA       layer chỉ gắn biến Semantic · Primitives ẩn khỏi picker · sửa màu ở main component,
            không override từng instance · mỗi trạng thái là một biến thể trỏ đúng vai trò

CẤM         màu thay cho thứ bậc · nền màu ở vùng lớn (trừ hero trang thương hiệu) · hai accent
            mã hex viết cứng trong component · đen thuần / trắng thuần · hover bằng opacity
            semantic dùng trang trí · chép bảng màu sản phẩm khác mà không chạy lại 8 bước
            dải màu dày dọc mép trái/phải thẻ để báo trạng thái (thay bằng badge có icon + chữ,
            hoặc tô nền tint cả thẻ)
            chữ tô gradient · chữ đặt trên kính mờ · gradient chỉ để "cho có màu" (kiểu tím chuyển xanh)
            chữ neutral trên nền tint có màu (lấy nấc 700–900 của chính hue nền)
```

## 5. Mẫu báo cáo

```
Hệ màu · <tên file>
Đã tạo:        Primitives (67 biến, 1 mode) · Semantic (25 biến, Light/Dark)
Primary:       #3468DE (OKLCH 0,55 / 0,19 / 264°): nấc 600, hạ L 0,01 để link đạt 4,5 trên nền subtle
Contrast chính (Light / Dark):
               chữ chính 16,0 / 14,0 · chữ muted 4,6 / 5,7 · chữ trên nút 5,0 / 7,8 · viền ô nhập 4,4 / 3,0
audit_design:  palette 0 FAIL · 1 WARN · 0 SKIP | design Light 0 FAIL | design Dark 0 FAIL · 2 SKIP (chữ trên ảnh)
Lệch mặc định: info nằm trong vùng hue của primary → info dùng ramp primary, luôn kèm icon "i"
Cần bạn xem:   ảnh thang xám (đính kèm) · 2 chỗ chữ trên ảnh · mục notMeasured: <chép nguyên danh sách>
```
