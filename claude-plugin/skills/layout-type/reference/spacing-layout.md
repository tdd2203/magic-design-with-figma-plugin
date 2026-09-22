# Khoảng cách và bố cục — thang 4, auto layout, lưới

Khoảng cách là thứ đầu tiên nên kiểm khi màn hình "thấy sai sai" mà màu, chữ đều đúng thang. Hai
bệnh hay gặp nhất: mọi khoảng bằng nhau nên không còn nhóm, hoặc mỗi chỗ một số nên không còn hệ.

Mọi số là mặc định. File có spacing variables → dùng đúng thang đó.

---

## 1. Thang khoảng cách

- Mọi khoảng cách là **bội số của 4**: `4 8 12 16 20 24 32 40 48 64 80 96`.
- Vì sao cơ số 4 mà không 8: với cơ số 8, khoảng icon ↔ chữ (4) và nấc 12 không có chỗ đứng.
- Áp cho mọi thứ: `itemSpacing`, padding, chiều cao điều khiển, bề rộng cột, line height.
- Thang dựng thành variables một lần (§11) rồi gắn vào node — đổi thang ở một chỗ, và
  `audit_design` không còn báo `lech-nac-khoang-cach`.

## 2. Gần nhau là cùng nhóm

Mắt ghép các thứ đứng gần nhau thành một nhóm. Khoảng cách **nói** cái gì thuộc về cái gì:

| Quan hệ | Khoảng |
|---|---|
| Nhãn ↔ ô của nó (icon ↔ chữ bên cạnh: 4–8) | 8 |
| Ô ↔ ô trong một nhóm | 16 |
| Nhóm ↔ nhóm | 32 |
| Vùng ↔ vùng của trang | 48–64 |

- **Khoảng trong nhóm luôn nhỏ hơn khoảng giữa nhóm, tỉ lệ tối thiểu 1:2.** Nhãn cách ô bằng ô
  cách ô → mắt ghép nhãn với ô phía trên.
- Trong Figma, quy tắc này **chính là cấu trúc auto layout lồng nhau**: mỗi cấp nhóm là một frame
  auto layout với `itemSpacing` riêng (field 8 → nhóm 16 → form 32). Đừng dựng một frame phẳng
  rồi chèn khoảng bằng spacer.

## 3. Nhịp, thứ bậc và phép thử nheo mắt

- **Nhịp** là các cỡ khoảng cách đi thành bậc: 8–16 trong nhóm → 32 giữa nhóm → 48–64 giữa vùng. Đọc
  từ trên xuống, mắt gặp khoảng nhỏ là biết còn cùng nhóm, gặp khoảng lớn là biết sang chuyện khác.
  Một màn hình chỉ dùng một cỡ khoảng cách thì không truyền được điều đó.
- **Thứ quan trọng được nhiều chỗ trống quanh nó hơn** — đây là cách tạo thứ bậc không tốn thêm
  màu hay khung. Tiêu đề: khoảng phía trên lớn hơn phía dưới để nó bám vào phần nó mở đầu.
- Cần nhấn mạnh hơn nữa thì thêm cỡ và độ đậm ([typography.md §4](typography.md#4-thứ-bậc-bằng-tương-phản));
  màu nền và đường viền là bước sau cùng.
- **Phép thử nheo mắt:** `screenshot` với `scale` 0,25 (hình nhỏ, chi tiết nhoè đi). Trả lời:
  1. Nhìn vào cái gì đầu tiên?
  2. Nút nào là việc chính ở màn hình này?
  3. Các nhóm có tách nhau rõ không?

  Câu trả lời trùng ý định → ổn. Không trùng → sửa bố cục (khoảng cách, cỡ, vị trí) trước khi
  nghĩ tới màu.

## 4. Gestalt và bố cục

| Nguyên lý | Nghĩa | Trong Figma |
|---|---|---|
| Gần nhau | Gần = cùng nhóm | `itemSpacing` theo cấp (§2) |
| Giống nhau | Cùng vai trò thì cùng hình thức | Cùng component, cùng text style |
| Vùng chung | Cùng nền, cùng khung = cùng nhóm | Frame có fill hoặc stroke — dùng sau khi khoảng trắng không đủ |
| Liên tục | Mắt đi theo đường thẳng hàng | Ít mép căn chung; `counterAxisAlignItems` nhất quán |
| Nền–hình | Thứ nổi phải tách khỏi nền | Lớp phủ tối sau modal, độ nổi ([sizing-surfaces.md §5](sizing-surfaces.md#5-thang-độ-nổi--effect-styles)) |

- **Thẳng hàng theo ít mép:** mỗi thứ lệch vài px so với mép chung là một tín hiệu nhiễu. Trong
  auto layout, dùng chung padding cho cả cột thay vì chỉnh từng con.
- **Một điểm nhấn mỗi màn hình:** đúng một nút nền đặc, một con số lớn, hoặc một tiêu đề chính.
- **Đường quét mắt:** trang nhiều chữ và dashboard đọc theo hình chữ F (trên trái trước, dọc xuống
  bên trái); trang ít chữ như landing đọc theo hình chữ Z. Đặt thứ quan trọng trên đường đó.
- **Thẻ (frame có nền, viền hoặc bóng) phải có lý do.** Nhóm đã tách được bằng khoảng cách và mép
  căn chung thì thêm khung chỉ thêm đường nét. Dùng thẻ khi mỗi khối là một đối tượng riêng mà
  người dùng mở, kéo, chọn được. Trong thẻ cần chia nhỏ nữa thì dùng khoảng cách hoặc đường kẻ,
  không đặt thêm thẻ bên trong.

## 5. Bề rộng tối đa

| Nội dung | Bề rộng tối đa |
|---|---|
| Văn bản đọc dài | 640–720px (65–75 ký tự mỗi dòng) |
| Form | 480–640px, một cột |
| Bảng dữ liệu | Không giới hạn |
| Dashboard | 1440px |

Trong Figma: khung trang là auto layout dọc với `counterAxisAlignItems = 'CENTER'`; cột nội dung
bên trong `layoutSizingHorizontal = 'FILL'` + `maxWidth = 720`. Khung trang rộng ra thì cột dừng
ở 720 và nằm giữa.

Bố cục danh sách + chi tiết quen thuộc: nav 240 · danh sách 320–400 · chi tiết phần còn lại.

## 6. Auto layout

| Việc | Thuộc tính | Giá trị |
|---|---|---|
| Hướng | `layoutMode` | `'HORIZONTAL'` · `'VERTICAL'` · `'GRID'` (§7) |
| Khoảng giữa con | `itemSpacing` | Từ thang; âm được (avatar chồng nhau) |
| Đệm | `paddingTop` · `paddingRight` · `paddingBottom` · `paddingLeft` | Từ thang |
| Căn trục chính | `primaryAxisAlignItems` | `'MIN'` · `'CENTER'` · `'MAX'` · `'SPACE_BETWEEN'` |
| Căn trục phụ | `counterAxisAlignItems` | `'MIN'` · `'CENTER'` · `'MAX'` · `'BASELINE'` |
| Co giãn | `layoutSizingHorizontal` / `layoutSizingVertical` | `'FIXED'` · `'HUG'` · `'FILL'` |
| Xuống dòng | `layoutWrap = 'WRAP'` + `counterAxisSpacing` | Chỉ với `HORIZONTAL` |
| Giới hạn | `minWidth` · `maxWidth` · `minHeight` · `maxHeight` | Frame auto layout và con trực tiếp của nó |
| Nằm ngoài dòng chảy | `layoutPositioning = 'ABSOLUTE'` + `constraints` | Badge góc, nút đóng |
| Viền tính vào kích thước | `strokesIncludedInLayout` | `true` khi viền dày phải nằm trong khung |
| Cắt phần tràn | `clipsContent` | Khung màn hình, ô ảnh — không bật cho nhóm chứa chữ |

**Chọn co giãn:**

- **Hug** — thứ co theo nội dung: nút, tag, badge, nhãn.
- **Fill** — thứ giãn theo khung chứa: ô nhập trong form, thẻ trong cột, đoạn văn.
- **Fixed** — khung màn hình, cột có bề rộng cố định (sidebar 240), điều khiển có chiều cao chuẩn.

**Mẹo:**

- Hàng nhãn–giá trị khác cỡ chữ → `counterAxisAlignItems = 'BASELINE'` để chân chữ thẳng hàng.
- Thanh công cụ (tiêu đề trái, hành động phải) → `primaryAxisAlignItems = 'SPACE_BETWEEN'`.
- Absolute là ngoại lệ: dùng nhiều thì auto layout mất tác dụng.

```js
// Chip xuống dòng khi hết chỗ + badge absolute ở góc thẻ
const card = await figma.getNodeByIdAsync('CARD_ID')
const badge = await figma.getNodeByIdAsync('BADGE_ID')
if (!card || card.type !== 'FRAME' || card.layoutMode === 'NONE') throw new Error('Cần một thẻ có auto layout')
if (!badge || badge.type !== 'FRAME') throw new Error('Cần id của badge')
const chips = figma.createFrame()
chips.name = 'Chips'
chips.layoutMode = 'HORIZONTAL'
chips.layoutWrap = 'WRAP'
chips.itemSpacing = 8
chips.counterAxisSpacing = 8
chips.fills = []
card.appendChild(chips)
chips.layoutSizingHorizontal = 'FILL'
chips.layoutSizingVertical = 'HUG'
card.appendChild(badge)
badge.layoutPositioning = 'ABSOLUTE'
badge.constraints = { horizontal: 'MAX', vertical: 'MIN' }
badge.x = card.width - badge.width - 8
badge.y = 8
return { chips: chips.id }
```

## 7. Grid auto layout

`layoutMode = 'GRID'` cho thứ cần thẳng **cả hàng lẫn cột**: dashboard, lưới thẻ, bảng giá. Danh
sách chảy một chiều (chip, tag) thì `HORIZONTAL` + `WRAP` là đủ.

```js
const grid = figma.createFrame()
grid.name = 'Dashboard'
grid.layoutMode = 'GRID'
grid.gridColumnCount = 3
grid.gridRowCount = 2
grid.gridColumnGap = 16
grid.gridRowGap = 16
grid.gridColumnSizes[0].type = 'FIXED' // cột đầu cố định, các cột còn lại FLEX chia phần còn lại
grid.gridColumnSizes[0].value = 280
grid.resize(1200, 560)
grid.layoutSizingHorizontal = 'FIXED'
grid.layoutSizingVertical = 'FIXED'
grid.fills = []
const widgets = []
for (let i = 0; i < 5; i++) {
  const w = figma.createFrame()
  w.name = `Widget ${i + 1}`
  w.cornerRadius = 12
  widgets.push(w)
}
widgets.forEach((w, i) => {
  grid.appendChildAt(w, Math.floor(i / 3), i % 3)
  w.layoutSizingHorizontal = 'FILL' // lấp đầy ô
  w.layoutSizingVertical = 'FILL'
})
widgets[4].gridColumnSpan = 2 // widget cuối chiếm hai cột
return { grid: grid.id }
```

- `gridAutoTracks = 'ROWS'` để hàng tự thêm khi thêm con (không đặt `gridRowCount` nữa).
- Code báo lỗi ở `layoutMode = 'GRID'` (bản Figma cũ) → dựng bằng auto layout dọc chứa các hàng ngang.

## 8. Constraints — frame không auto layout

Con của frame **không** auto layout (và con `ABSOLUTE`) bám mép theo `constraints`:

| Muốn | `constraints` |
|---|---|
| Header giãn theo bề ngang, dính trên | `{ horizontal: 'STRETCH', vertical: 'MIN' }` |
| Nút nổi góc dưới phải | `{ horizontal: 'MAX', vertical: 'MAX' }` |
| Hộp giữa màn hình | `{ horizontal: 'CENTER', vertical: 'CENTER' }` |
| Hình nền co theo tỉ lệ | `{ horizontal: 'SCALE', vertical: 'SCALE' }` |

Thử bằng cách `resize` **bản sao** của khung (1440 → 1024 → 390) rồi `screenshot` — chỗ nào vỡ
là chỗ thiếu constraint hoặc nên chuyển sang auto layout.

## 9. Layout grid — cột

Layout grid chỉ là **đường gióng**: nó không tự đặt vị trí, con trong auto layout không bám vào
nó. Dùng để suy ra bề rộng cột (sidebar = 3 cột, nội dung = 9 cột) và để kiểm căn thẳng.

| Khung | Cột | Gutter | Lề |
|---|---|---|---|
| 1440 | 12 | 24 | 80 |
| 1280 | 12 | 24 | 64 |
| 1024 | 12 | 24 | 32 |
| 768 | 8 | 16 | 32 |
| 390 | 4 | 16 | 16 |

Điểm khởi đầu phổ biến, không phải chuẩn — file có grid style thì theo file. Bề rộng cột ra số lẻ
là bình thường; thứ cần nằm trên thang 4 là gutter và lề.

```js
const frame = await figma.getNodeByIdAsync('FRAME_ID')
if (!frame || frame.type !== 'FRAME') throw new Error('Cần một frame')
const name = 'Grid/Desktop 1440'
let style = (await figma.getLocalGridStylesAsync()).find((s) => s.name === name)
if (!style) {
  style = figma.createGridStyle()
  style.name = name
  style.layoutGrids = [
    { pattern: 'COLUMNS', alignment: 'STRETCH', count: 12, gutterSize: 24, offset: 80,
      visible: true, color: { r: 1, g: 0, b: 0, a: 0.08 } },
    { pattern: 'GRID', sectionSize: 4, visible: false, color: { r: 0, g: 0, b: 1, a: 0.06 } }, // lưới 4 để kiểm căn
  ]
}
await frame.setGridStyleIdAsync(style.id) // khung dùng một lần: gán thẳng frame.layoutGrids = [...]
return { style: style.id }
```

## 10. Đọc thang có sẵn trong file

Chạy trước khi dựng, chỉ đọc:

```js
const [text, effect, grid] = await Promise.all([
  figma.getLocalTextStylesAsync(),
  figma.getLocalEffectStylesAsync(),
  figma.getLocalGridStylesAsync(),
])
const collections = await figma.variables.getLocalVariableCollectionsAsync()
const numbers = await figma.variables.getLocalVariablesAsync('FLOAT')
return {
  textStyles: text.map((s) => ({ name: s.name, font: s.fontName, size: s.fontSize, lineHeight: s.lineHeight })),
  effectStyles: effect.map((s) => s.name),
  gridStyles: grid.map((s) => s.name),
  collections: collections.map((c) => ({ name: c.name, modes: c.modes.map((m) => m.name) })),
  numbers: numbers.map((v) => {
    const c = collections.find((col) => col.id === v.variableCollectionId)
    return { name: v.name, scopes: v.scopes, value: c ? v.valuesByMode[c.defaultModeId] : null }
  }),
}
```

Variables và styles lấy từ thư viện team không hiện trong danh sách local. Thấy node đã gắn
variable lạ → đọc `node.boundVariables` rồi `figma.variables.getVariableByIdAsync(id)`.

## 11. Dựng và gắn spacing variables

Chỉ tạo khi file chưa có thang. Tên theo giá trị (`space/16`) dễ đọc cho lớp nguyên thuỷ; file đã
đặt tên kiểu khác (`space/md`) thì theo file.

```js
const locals = await figma.variables.getLocalVariablesAsync('FLOAT')
if (locals.some((v) => v.name.startsWith('space/'))) return 'File đã có thang space/ — dùng lại'
const col = figma.variables.createVariableCollection('Spacing')
const mode = col.modes[0].modeId
const space = new Map()
for (const n of [4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96]) {
  const v = figma.variables.createVariable(`space/${n}`, col, 'FLOAT')
  v.setValueForMode(mode, n)
  v.scopes = ['GAP'] // hiện ở ô gap và padding của auto layout
  space.set(n, v)
}
const radius = new Map()
// sm: nút, ô nhập, tag (4–6) · md: popover, thẻ nhỏ · lg: thẻ, modal (8–12) · full: pill, avatar
for (const { name, value } of [{ name: 'sm', value: 6 }, { name: 'md', value: 8 }, { name: 'lg', value: 12 }, { name: 'full', value: 999 }]) {
  const v = figma.variables.createVariable(`radius/${name}`, col, 'FLOAT')
  v.setValueForMode(mode, value)
  v.scopes = ['CORNER_RADIUS']
  radius.set(name, v)
}
const card = await figma.getNodeByIdAsync('CARD_ID')
if (card && card.type === 'FRAME') {
  card.setBoundVariable('itemSpacing', space.get(16))
  card.setBoundVariable('paddingTop', space.get(24))
  card.setBoundVariable('paddingBottom', space.get(24))
  card.setBoundVariable('paddingLeft', space.get(24))
  card.setBoundVariable('paddingRight', space.get(24))
  card.setBoundVariable('cornerRadius', radius.get('lg'))
}
return { collection: col.id }
```

## 12. Sửa khoảng cách lệch nấc

`audit_design` báo `lech-nac-khoang-cach` → file có spacing variable thì gắn biến (§11); không có thì
làm tròn về bội số 4 gần nhất. **Bỏ qua** giá trị đã gắn variable (đó là quyết định của file) và lớp
nằm trong instance (sửa ở main component, không sửa từng instance). Đoạn code nắn khoảng cách, chạy
xem trước rồi mới áp, xét đúng những thuộc tính rule này đo (bỏ `itemSpacing` khi `SPACE_BETWEEN`,
tính `counterAxisSpacing` khi wrap, gap của GRID): skill `design-audit`, reference/rules.md §6.4. Chạy
trên đúng frame được báo, rồi `screenshot` lại để chắc không có gì nhảy chỗ.
