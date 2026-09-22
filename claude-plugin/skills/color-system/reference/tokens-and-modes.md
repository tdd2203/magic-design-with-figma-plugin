# Token theo vai trò, Figma Variables và mode

Thứ bàn giao là **bảng token**, không phải bảng màu. Component chỉ biết "đây là nền thẻ", "đây là chữ
phụ"; mã màu thật nằm ở một chỗ duy nhất. Đổi thương hiệu hay thêm dark mode thì đổi ở đó, không phải
đi sửa từng layer.

## 1. Hai lớp, và vì sao

- **Primitives**: giá trị thô, đặt tên theo nhóm và nấc (`primary/600`, `neutral/100`). Không mang nghĩa
  sử dụng. Chỉ có một mode.
- **Semantic**: vai trò (`color/action/default`, `color/text/muted`). Mỗi biến là một **alias** trỏ về
  một biến Primitives, và mỗi mode (Light, Dark) trỏ về một nấc khác nhau.

Dark mode **chỉ định nghĩa lại lớp Semantic**; Primitives giữ nguyên. Layer chỉ gắn biến Semantic, nên
đổi mode trên một frame là cả frame đổi theo.

## 2. Ánh xạ sang Figma

| | Primitives | Semantic |
|---|---|---|
| Collection | `Primitives` | `Semantic` |
| Mode | `Value` (một mode) | `Light` (mode mặc định), `Dark` |
| Giá trị | `{ r, g, b }` 0–1, tính từ OKLCH | `figma.variables.createVariableAlias(primitive)` |
| Tên | `neutral/50` … `neutral/950`, `primary/…`, `success/…`, `warning/…`, `danger/…`, `info/…`, `base/white` | `color/<nhóm>/<vai trò>` |
| `scopes` | `[]`: ẩn khỏi picker | theo vai trò (bảng dưới) |

**Tên.** Dấu `/` tạo nhóm trong bảng Variables (`color/text/muted` nằm trong nhóm color › text). Nên
dùng chữ thường, số, gạch ngang. Tránh dấu chấm và ngoặc nhọn, vì chúng dễ vỡ khi xuất token ra code.
Các tên vai trò ở §3 còn là thứ để `audit_design {scope: "palette"}` nhận ra cặp chữ/nền và cặp
primary/semantic. Tên lạ quá thì công cụ có thể trả `SKIP`, và `SKIP` không phải là đạt.

**Scope** chỉ lọc danh sách trong picker của Figma, không cấm gắn bằng Plugin API. Có scope đúng thì
người dùng không vô tình tô nền bằng màu chữ.

| Loại vai trò | `scopes` |
|---|---|
| Nền (`color/bg/*`, `*/subtle-bg`) | `['FRAME_FILL', 'SHAPE_FILL']` |
| Chữ (`color/text/*`, `action/subtle-text`) | `['TEXT_FILL', 'SHAPE_FILL']` (`SHAPE_FILL` để icon vector dùng chung màu chữ) |
| Viền (`color/border/*`) | `['STROKE_COLOR']` |
| Hành động và trạng thái (`action/default`, `hover`, `active`, `<status>/default`) | `['ALL_FILLS', 'STROKE_COLOR']` |
| Focus ring | `['STROKE_COLOR', 'EFFECT_COLOR']` |
| Mọi biến Primitives | `[]` |

`ALL_FILLS` đã bao gồm frame, shape và text, nên đừng ghép nó với `FRAME_FILL` hay `TEXT_FILL`.

**Mô tả và code syntax.** Ghi lý do ngoại lệ vào `variable.description` (ví dụ "success chỉ cách primary
34°, luôn kèm icon"). `variable.setVariableCodeSyntax('WEB', 'var(--text-muted)')` giúp dev thấy đúng tên
token trong Dev Mode. Việc này tuỳ chọn, nhưng nên làm khi file sẽ được bàn giao.

**Giới hạn gói.** Số mode mỗi collection tuỳ gói Figma; gói thấp nhất có thể chỉ cho một mode. Khi đó
`addMode` báo lỗi dạng `Limited to N modes`. Script ở §4 tạo `Semantic` trước để dừng sớm khi gặp lỗi
này. Gặp lỗi thì báo người dùng và hỏi họ muốn làm gì, đừng tự dựng một collection "Dark" riêng để lách.

## 3. Bảng vai trò

Số trong cột đo là contrast **thấp nhất** trên các nền mà token đó thật sự nằm lên (Light: page ·
surface · subtle; Dark: page · surface). Đo trên hệ mặc định do script §4 dựng từ `#2563EB`. Đổi màu
thương hiệu thì số đổi, **đo lại**.

| Biến | Light | Dark | Đo L / D | Dùng ở đâu |
|---|---|---|---|---|
| `color/bg/page` | neutral-50 | **neutral-900** | | Nền trang |
| `color/bg/surface` | base/white | neutral-800 | | Thẻ, modal, bảng |
| `color/bg/subtle` | neutral-100 | neutral-800 | | Vùng phụ, hover hàng |
| `color/bg/disabled` | neutral-100 | neutral-800 | | Nền control vô hiệu |
| `color/border/default` | neutral-200 | neutral-700 | | Viền trang trí, đường chia. Không bắt buộc 3:1 |
| `color/border/input` | **neutral-500** | **neutral-500** | 4,38 / 3,01 | Viền ô nhập, select. Là thứ duy nhất để nhận ra ô, nên **≥ 3:1** (WCAG 1.4.11) |
| `color/text/primary` | neutral-900 | neutral-50 | 16,0 / 14,0 | Chữ chính |
| `color/text/secondary` | neutral-600 | **neutral-300** | 6,7 / 9,8 | Chữ phụ |
| `color/text/muted` | **neutral-500** | **neutral-400** | 4,6 trên page / 5,7 | Placeholder, gợi ý. Vẫn là chữ nên cần 4,5. Trên `bg/subtle` chỉ 4,38 → ở đó dùng `text/secondary` |
| `color/text/disabled` | neutral-400 | neutral-500 | 2,3 / 3,0 | Chữ của control vô hiệu. WCAG miễn contrast, nhưng phải kèm lý do vì sao vô hiệu |
| `color/text/on-action` | base/white | neutral-950 | 5,0 / 7,8 | Chữ trên nút (đo trên `action/default`) |
| `color/action/default` | primary-600 | primary-400 | 4,55 / 5,8 | Nút chính, link |
| `color/action/hover` | primary-700 | primary-300 | 6,8 / 11,2 (chữ trên nút) | Lệch ≥ 1 nấc |
| `color/action/active` | primary-800 | primary-200 | 9,5 / 14,1 (chữ trên nút) | Nhấn. Lệch ≥ 2 nấc so với default, đủ cho mobile |
| `color/action/subtle-bg` | primary-50 | primary-900 | | Nút phụ, tag, mục đang chọn |
| `color/action/subtle-text` | primary-700 | primary-200 | 6,2 / 9,3 | Chữ trên `subtle-bg` |
| `color/focus-ring` | primary-600 | primary-400 | 4,5 / 5,8 | Vòng 2px **cách nút 2px**. Không có khoảng cách thì nút primary đang focus trông như không có vòng |
| `color/<status>/default` | `<status>`-700 | `<status>`-400 | ≥ 5,5 / ≥ 5,5 | Icon, chữ, viền của success · warning · danger · info |
| `color/<status>/subtle-bg` | `<status>`-50 | `<status>`-900 | chữ trên nó ≥ 5,6 / ≥ 5,0 | Nền badge, alert |

Các chỗ **in đậm** cố ý khác lối nghĩ "đảo ngược light":
- Nền dark là neutral-900 (`#16181C`), không phải neutral-950 (`#0A0B0E`, gần đen thuần).
- `text/secondary` dark lên neutral-300 để còn khác `text/muted`.
- `border/input` có vai trò riêng, vì viền trang trí neutral-200 chỉ khoảng 1,2–1,3:1 và ô nhập gần như vô hình.
- Các nền tint ở Dark (`action/subtle-bg`, `<status>/subtle-bg`) dùng nấc 900, không phải 950: nấc 950
  (L 0,22) gần trùng nền trang Dark (contrast chỉ khoảng 1,02:1), tint sẽ không thấy.

Trạng thái dùng nấc 700 ở Light vì nấc 600 của lục chỉ đạt khoảng 4,1 trên nền trang. Nấc 600 chỉ dùng
được khi đã đo ≥ 4,5 trên mọi nền nó nằm lên. Nút danger đặc dùng `danger/default` làm nền và
`text/on-action` làm chữ (đo được 7,1 / 7,4).

**Nếu script báo `actionStep: 700`** (nấc 600 không đạt kể cả khi đã hạ L), các vai trò hành động
dịch xuống: default 700, hover 800, active 900.

**Vai trò thêm cho bộ nút đầy đủ.** Nút phụ, nút thứ ba (chỉ chữ) và nút phá huỷ (skill
**components-states**) cần thêm bốn vai trò mà bảng trên chưa có. Dựng khi làm bộ nút, sau khi hỏi
người dùng: dán đoạn dưới vào script §4, ngay sau vòng lặp thêm `<status>/default`, rồi chạy lại
(script upsert, không tạo trùng; kết quả báo `semantic: 29` thay vì 25).

```js
Object.assign(ROLES, {
  'color/bg/subtle-active': ['neutral/200', 'neutral/700', BG], // nút phụ lúc nhấn
  'color/action/subtle-bg-active': ['primary/100', 'primary/800', BG], // nút thứ ba lúc nhấn
  'color/danger/hover': ['danger/800', 'danger/300', ANY], // lệch 1 nấc so với danger/default
  'color/danger/active': ['danger/900', 'danger/200', ANY], // lệch 2 nấc
})
```

Trên hệ mặc định dựng từ `#2563EB`: `text/on-action` trên `danger/hover` và `danger/active` đạt 9,9 và
13,3 (Light), 10,9 và 13,9 (Dark); `action/subtle-text` trên `subtle-bg-active` đạt 5,7 (Light) và 6,8
(Dark). Đổi màu thương hiệu hay hue semantic thì đo lại.

**Register đọc lâu** (blog, tài liệu): contrast quá cao cũng mỏi mắt. Chữ thân bài nên ở mức 12–16:1,
đừng nhắm 21. Khi đó cho `text/primary` Light trỏ neutral-800 (khoảng 14:1).

**Ở Light, `bg/page` và `bg/surface` chỉ chênh khoảng 1,5% độ sáng.** Thẻ đặt trên nền trang cần
`border/default` hoặc bóng nhẹ, nếu không sẽ không thấy ranh giới.

## 4. Script dựng: một lần `execute_figma_code`

Dán khối helper và ramp ở [color-theory.md §7](color-theory.md) lên đầu, sửa `BRAND` (và `HUES` nếu
cần), rồi chạy. Script **upsert**: chạy lại sau khi lỗi giữa chừng sẽ cập nhật biến cũ chứ không tạo
bản trùng. Chỉ chạy nguyên script trên file **chưa có hệ màu**. File đã có hệ thì xem §11.

```js
// <dán khối helper + ramp của color-theory.md §7 vào đây>
const BRAND = '#2563EB' // màu thương hiệu (hoặc màu hành động đã chốt với người dùng)
const PIN_600 = '' // điền lại mã BRAND nếu muốn giữ đúng mã ở nấc 600 (vẫn phải qua phép đo)
const HUES = { success: 145, warning: 75, danger: 28, info: 250 } // hue OKLCH, chỉnh theo color-theory.md §4
const WHITE = { r: 1, g: 1, b: 1 }
const b = toOklch(figma.util.rgb(BRAND))
if (b.C < 0.03) throw new Error('Màu thương hiệu gần xám (C ' + b.C.toFixed(3) + '): hỏi người dùng màu hành động trước')
const H = b.H, K = Math.min(1, b.C / 0.19), SHIFT = H > 110 && H < 270 ? 8 : -8
const fam = { neutral: ramp(H, L_NEUTRAL, C_NEUTRAL), primary: ramp(H, L_COLOR, C_COLOR, SHIFT, K), base: { white: WHITE } }
for (const s of Object.keys(HUES)) fam[s] = ramp(HUES[s], L_COLOR, C_COLOR)
// Nấc 600 phải chở chữ trắng VÀ làm link trên bg/subtle (≥ 4,5); không đạt thì vai trò hành động lùi một nấc
const TESTS = [[WHITE, 4.5], [fam.neutral[100], 4.5]]
let p600 = fitStep(0.56, 0.19 * K, H + SHIFT * 0.1, TESTS)
if (PIN_600) { const pin = figma.util.rgb(PIN_600); p600 = TESTS.every(([bg, min]) => contrast(pin, bg) >= min) ? pin : null }
if (p600) fam.primary[600] = p600
const [act, hov, prs] = p600 ? [600, 700, 800] : [700, 800, 900]

const cols = await figma.variables.getLocalVariableCollectionsAsync()
const vars = await figma.variables.getLocalVariablesAsync('COLOR')
function collection(name, modes) {
  let c = cols.find((x) => x.name === name), fresh = false
  if (!c) { c = figma.variables.createVariableCollection(name); c.renameMode(c.modes[0].modeId, modes[0]); fresh = true }
  try { for (const m of modes) if (!c.modes.some((x) => x.name === m)) c.addMode(m) } catch (e) {
    if (fresh) c.remove()
    throw new Error('Không thêm được mode cho "' + name + '" (gói Figma giới hạn số mode?): ' + String(e))
  }
  return c
}
const mode = (c, name) => { const m = c.modes.find((x) => x.name === name); if (!m) throw new Error('Thiếu mode ' + name); return m.modeId }
function upsert(c, name) {
  let v = vars.find((x) => x.variableCollectionId === c.id && x.name === name)
  if (!v) { v = figma.variables.createVariable(name, c, 'COLOR'); vars.push(v) }
  return v
}
const sem = collection('Semantic', ['Light', 'Dark']) // tạo trước: gói giới hạn mode thì dừng khi chưa tạo gì
const prim = collection('Primitives', ['Value'])

const P = {}
for (const f of Object.keys(fam)) for (const step of Object.keys(fam[f])) {
  const v = upsert(prim, f + '/' + step)
  v.setValueForMode(mode(prim, 'Value'), fam[f][step])
  v.scopes = [] // ẩn khỏi picker: người thiết kế chỉ chọn biến Semantic
  P[v.name] = v
}
const TEXT = ['TEXT_FILL', 'SHAPE_FILL'], BG = ['FRAME_FILL', 'SHAPE_FILL'], LINE = ['STROKE_COLOR'], ANY = ['ALL_FILLS', 'STROKE_COLOR']
const ROLES = { // tên: [Light, Dark, scopes]
  'color/bg/page': ['neutral/50', 'neutral/900', BG],
  'color/bg/surface': ['base/white', 'neutral/800', BG],
  'color/bg/subtle': ['neutral/100', 'neutral/800', BG],
  'color/bg/disabled': ['neutral/100', 'neutral/800', BG],
  'color/border/default': ['neutral/200', 'neutral/700', LINE],
  'color/border/input': ['neutral/500', 'neutral/500', LINE],
  'color/text/primary': ['neutral/900', 'neutral/50', TEXT],
  'color/text/secondary': ['neutral/600', 'neutral/300', TEXT],
  'color/text/muted': ['neutral/500', 'neutral/400', TEXT],
  'color/text/disabled': ['neutral/400', 'neutral/500', TEXT],
  'color/text/on-action': ['base/white', 'neutral/950', TEXT],
  'color/action/default': ['primary/' + act, 'primary/400', ANY],
  'color/action/hover': ['primary/' + hov, 'primary/300', ANY],
  'color/action/active': ['primary/' + prs, 'primary/200', ANY],
  'color/action/subtle-bg': ['primary/50', 'primary/900', BG],
  'color/action/subtle-text': ['primary/700', 'primary/200', TEXT],
  'color/focus-ring': ['primary/' + act, 'primary/400', ['STROKE_COLOR', 'EFFECT_COLOR']],
}
for (const s of Object.keys(HUES)) {
  ROLES['color/' + s + '/default'] = [s + '/700', s + '/400', ANY]
  ROLES['color/' + s + '/subtle-bg'] = [s + '/50', s + '/900', BG]
}
for (const name of Object.keys(ROLES)) {
  const [light, dark, scopes] = ROLES[name]
  const v = upsert(sem, name)
  v.setValueForMode(mode(sem, 'Light'), figma.variables.createVariableAlias(P[light]))
  v.setValueForMode(mode(sem, 'Dark'), figma.variables.createVariableAlias(P[dark]))
  v.scopes = scopes
  v.setVariableCodeSyntax('WEB', 'var(--' + name.split('/').slice(1).join('-') + ')') // tuỳ chọn, cho dev
}
return {
  brand: { L: +b.L.toFixed(3), C: +b.C.toFixed(3), H: Math.round(H) }, actionStep: act, primary600: toHex(fam.primary[600]),
  primitives: Object.keys(P).length, semantic: Object.keys(ROLES).length,
  semanticNearPrimary: Object.keys(HUES).filter((s) => hueGap(HUES[s], H) < 60),
}
```

Kết quả với `#2563EB`: 67 biến Primitives, 25 biến Semantic, `actionStep: 600`, nấc 600 được hạ
xuống L 0,55 (`#3468DE`), và `semanticNearPrimary: ['info']`. Cách xử lý info ở
[color-theory.md §4](color-theory.md).

Thêm accent: tạo ramp `accent/…` trong Primitives, rồi **một** biến Semantic đặt tên theo chỗ dùng
(`color/accent/highlight`), không đặt tên chung chung. Sau đó chạy lại `audit_design {scope: "palette"}`.

## 5. Gắn biến

```js
const sem = (await figma.variables.getLocalVariableCollectionsAsync()).find((c) => c.name === 'Semantic')
if (!sem) throw new Error('Chưa có collection Semantic')
const vars = (await figma.variables.getLocalVariablesAsync('COLOR')).filter((v) => v.variableCollectionId === sem.id)
const token = (name) => { const v = vars.find((x) => x.name === name); if (!v) throw new Error('Thiếu biến ' + name); return v }
const paint = (name) => figma.variables.setBoundVariableForPaint({ type: 'SOLID', color: { r: 0, g: 0, b: 0 } }, 'color', token(name))
const card = await figma.getNodeByIdAsync('12:34') // id thật từ inspect_nodes
card.fills = [paint('color/bg/surface')]
card.strokes = [paint('color/border/default')]
const title = card.findAllWithCriteria({ types: ['TEXT'] })[0]
if (title) title.fills = [paint('color/text/primary')]
```

- `setBoundVariableForPaint` trả về **bản sao** của paint đã gắn biến. Mảng `fills`/`strokes` là chỉ
  đọc, nên phải gán mảng mới. Muốn giữ `opacity` hay `blendMode` của paint cũ thì truyền chính paint cũ
  vào thay cho object mẫu.
- Chữ nhiều màu trong một text node: dùng `text.setRangeFills(start, end, [paint(...)])`.
- Bóng và focus ring làm bằng effect: `figma.variables.setBoundVariableForEffect(effect, 'color', token('color/focus-ring'))`
  rồi gán `node.effects = [...]`.
- Component: gắn biến trong **main component**, không override từng instance. Biến thể trạng thái
  (hover, pressed, focus, disabled, loading, error, success) dựng theo skill **components-states**,
  mỗi trạng thái trỏ về đúng vai trò `action/hover`, `action/active`…
- Kiểm một paint đã gắn biến chưa: `paint.boundVariables && paint.boundVariables.color` (có `id` của biến).

## 6. Xem và kiểm mode

Mode được **kế thừa** theo cây layer. Page có mode mặc định (Light, mode đầu tiên của collection).
Một frame đặt mode riêng thì mọi con của nó theo mode đó, trừ khi một frame con lại đặt mode khác.

```js
// Nhân bản một frame cấp trên cùng và cho bản sao chạy mode Dark, đặt vào chỗ trống bên phải
const src = await figma.getNodeByIdAsync('12:34')
const sem = (await figma.variables.getLocalVariableCollectionsAsync()).find((c) => c.name === 'Semantic')
const dark = sem && sem.modes.find((m) => m.name === 'Dark')
if (!sem || !dark) throw new Error('Chưa có collection Semantic với mode Dark')
const right = Math.max(...figma.currentPage.children.map((n) => n.x + n.width))
const copy = src.clone()
figma.currentPage.appendChild(copy)
copy.name = src.name + ' · Dark'
copy.x = right + 120
copy.y = src.y
copy.setExplicitVariableModeForCollection(sem, dark.modeId)
const bg = (await figma.variables.getLocalVariablesAsync('COLOR')).find((v) => v.variableCollectionId === sem.id && v.name === 'color/bg/page')
return { id: copy.id, bgPage: bg ? bg.resolveForConsumer(copy).value : null }
```

- Sau đó `screenshot` bản Light và bản Dark, rồi chạy `audit_design {node_id, scope: "design"}` trên
  **từng bản**.
- `variable.resolveForConsumer(node)` cho biết giá trị thật mà biến nhận **tại node đó**. Dùng nó khi
  nghi một frame con đang đè mode, hoặc khi hai bản Light/Dark cho số đo giống hệt nhau.
- `node.explicitVariableModes` là mode đặt trực tiếp trên node; `node.resolvedVariableModes` là mode
  thực tế sau kế thừa.
- Bỏ mode riêng: `node.clearExplicitVariableModeForCollection(sem)`.
- Bản Dark là một deliverable thật, nên giữ lại nếu người dùng muốn. Chỉ bản sao **tạm** dùng để kiểm
  (thang xám) mới phải xoá ngay.

## 7. Trang tài liệu: bảng swatch

Bảng swatch gắn biến Primitives. Biến Primitives có `scopes = []`, nhưng Plugin API vẫn gắn được.

```js
const prim = (await figma.variables.getLocalVariableCollectionsAsync()).find((c) => c.name === 'Primitives')
if (!prim) throw new Error('Chưa có collection Primitives')
const vars = (await figma.variables.getLocalVariablesAsync('COLOR')).filter((v) => v.variableCollectionId === prim.id)
await figma.loadFontAsync({ family: 'Inter', style: 'Medium' })
const right = Math.max(0, ...figma.currentPage.children.map((n) => n.x + n.width))
const board = figma.createFrame()
board.name = 'Color system · Primitives'
board.layoutMode = 'VERTICAL'
board.itemSpacing = 16
board.paddingTop = board.paddingBottom = board.paddingLeft = board.paddingRight = 32
board.primaryAxisSizingMode = 'AUTO'
board.counterAxisSizingMode = 'AUTO'
board.x = right + 120
for (const fam of ['neutral', 'primary', 'success', 'warning', 'danger', 'info']) {
  const row = figma.createFrame()
  row.name = fam
  row.layoutMode = 'HORIZONTAL'
  row.itemSpacing = 8
  row.counterAxisAlignItems = 'CENTER'
  row.primaryAxisSizingMode = 'AUTO'
  row.counterAxisSizingMode = 'AUTO'
  row.fills = []
  const label = figma.createText()
  label.fontName = { family: 'Inter', style: 'Medium' }
  label.characters = fam
  label.fontSize = 14
  label.resize(88, label.height)
  label.textAutoResize = 'HEIGHT'
  row.appendChild(label)
  for (const v of vars.filter((x) => x.name.indexOf(fam + '/') === 0)) {
    const cell = figma.createRectangle()
    cell.name = v.name
    cell.resize(64, 48)
    cell.cornerRadius = 6
    cell.fills = [figma.variables.setBoundVariableForPaint({ type: 'SOLID', color: { r: 0, g: 0, b: 0 } }, 'color', v)]
    row.appendChild(cell)
  }
  board.appendChild(row)
}
figma.currentPage.selection = [board]
figma.viewport.scrollAndZoomIntoView([board])
return { id: board.id }
```

Font khác Inter thì đổi `family` sau khi đã kiểm font có trong file. Nền của `board` nên gắn
`color/bg/surface` và chữ gắn `color/text/primary`, để nhân bản sang Dark cũng xem được. Kèm theo nên có
một thẻ mẫu dùng toàn biến Semantic: tiêu đề, chữ phụ, ô nhập, nút chính, nút phụ, badge bốn trạng
thái. Đặt bản Light và bản Dark của thẻ này cạnh nhau (§6).

## 8. Chuyển màu viết cứng sang biến

`audit_design` báo `mau-khong-bien` khi file đã có biến mà layer vẫn tô màu viết cứng. Làm hai bước.

**Bước 1: quét, chỉ đọc.** Đếm các màu đặc chưa gắn biến và chưa dùng style, theo loại (chữ, nền, viền).
Không đi vào instance; sửa ở main component.

```js
// cần toHex trong helper (color-theory.md §7)
const root = await figma.getNodeByIdAsync('12:34')
const count = {}
const note = (p, kind) => {
  if (p.type !== 'SOLID' || p.visible === false || (p.boundVariables && p.boundVariables.color)) return
  const key = kind + ' ' + toHex(p.color)
  count[key] = (count[key] || 0) + 1
}
const walk = (n) => {
  if (n.type === 'INSTANCE') return
  if ('fills' in n && n.fills !== figma.mixed && !n.fillStyleId) n.fills.forEach((p) => note(p, n.type === 'TEXT' ? 'text' : 'fill'))
  if ('strokes' in n && !n.strokeStyleId) n.strokes.forEach((p) => note(p, 'stroke'))
  if ('children' in n) n.children.forEach(walk)
}
walk(root)
return count
```

**Bước 2: gắn những màu khớp chính xác.** Chỉ gắn khi giá trị Light của **đúng một** biến Semantic
(có scope hợp với loại layer) trùng mã màu. Trùng nhiều biến thì trả về `ambiguous` để hỏi người dùng.
Màu gần giống nhưng không khớp thì **không tự làm tròn**; đưa danh sách cho người dùng chọn vai trò.

```js
// cần toHex trong helper (color-theory.md §7)
const sem = (await figma.variables.getLocalVariableCollectionsAsync()).find((c) => c.name === 'Semantic')
const lightMode = sem && sem.modes.find((m) => m.name === 'Light')
if (!sem || !lightMode) throw new Error('Chưa có collection Semantic với mode Light')
const light = lightMode.modeId
const vars = (await figma.variables.getLocalVariablesAsync('COLOR')).filter((v) => v.variableCollectionId === sem.id)
const SCOPE = { text: ['TEXT_FILL', 'ALL_FILLS'], fill: ['FRAME_FILL', 'SHAPE_FILL', 'ALL_FILLS'], stroke: ['STROKE_COLOR'] }
const byKey = {}
for (const v of vars) {
  let val = v.valuesByMode[light]
  if (typeof val === 'object' && 'type' in val && val.type === 'VARIABLE_ALIAS') {
    const t = await figma.variables.getVariableByIdAsync(val.id)
    if (!t) continue
    const c = await figma.variables.getVariableCollectionByIdAsync(t.variableCollectionId)
    if (!c) continue
    val = t.valuesByMode[c.defaultModeId]
  }
  if (typeof val !== 'object' || !('r' in val)) continue // chỉ màu đặc
  for (const kind of Object.keys(SCOPE)) if (v.scopes.some((s) => SCOPE[kind].indexOf(s) >= 0)) {
    const key = kind + ' ' + toHex(val)
    byKey[key] = (byKey[key] || []).concat(v)
  }
}
const done = {}, ambiguous = {}
const fix = (paints, kind) => paints.map((p) => {
  if (p.type !== 'SOLID' || (p.boundVariables && p.boundVariables.color)) return p
  const key = kind + ' ' + toHex(p.color), hits = byKey[key] || []
  if (hits.length !== 1) { if (hits.length > 1) ambiguous[key] = hits.map((v) => v.name); return p }
  done[key + ' → ' + hits[0].name] = (done[key + ' → ' + hits[0].name] || 0) + 1
  return figma.variables.setBoundVariableForPaint(p, 'color', hits[0])
})
const walk = (n) => {
  if (n.type === 'INSTANCE') return
  if ('fills' in n && n.fills !== figma.mixed && !n.fillStyleId) n.fills = fix(n.fills, n.type === 'TEXT' ? 'text' : 'fill')
  if ('strokes' in n && !n.strokeStyleId) n.strokes = fix(n.strokes, 'stroke')
  if ('children' in n) n.children.forEach(walk)
}
walk(await figma.getNodeByIdAsync('12:34'))
return { done, ambiguous }
```

Mỗi lần chạy là một bước undo. Làm từng frame một và chụp lại sau khi gắn. Gắn đúng thì màn hình trông
y như cũ ở Light; khác đi là đã gắn nhầm vai trò.

## 9. Variables hay paint styles

| | Variables (COLOR) | Paint styles |
|---|---|---|
| Mode Light/Dark | Có | Không |
| Alias hai lớp | Có | Không (style có thể **bọc** một biến) |
| Gradient, ảnh, nhiều lớp paint, blend mode | Không, chỉ màu đặc | Có |
| Scope trong picker | Có | Không |

- File mới: dùng **Variables** cho mọi màu đặc.
- File đang dùng paint styles: giữ styles (theo thứ tự ưu tiên). Có thể cho style bọc biến để được cả
  hai: `style.paints = [figma.variables.setBoundVariableForPaint(paint, 'color', v)]`.
- Gradient và lớp phủ lên ảnh thì dùng paint style. Chữ đè ảnh cần lớp phủ tối 40–60%, đừng tin "ảnh
  này chắc đủ tối". `audit_design` trả `SKIP` (`chu-khong-do-duoc`) cho chữ trên ảnh hoặc gradient,
  nên phải chụp màn hình và nhìn.
- Gán style cho node dùng setter async: `await node.setFillStyleIdAsync(style.id)`.

## 10. Dark mode chi tiết

Ba luật:
1. Nền **không** `#000000`. Nền trang nằm khoảng `#0F172A`–`#18181B`; hệ mặc định dùng neutral-900
   `#16181C`. Đen thuần làm chữ sáng loé và viền rung.
2. Chữ sáng **không** `#FFFFFF` thuần mà là neutral-50. Chữ sáng trên nền tối vốn trông đậm hơn thật,
   thêm trắng thuần thì càng chói.
3. Primary **sáng lên khoảng 2 nấc** (600 → 400) và **bớt chroma**. Trong bảng ramp, nấc 400 đã có
   chroma 0,15 so với 0,19 của nấc 600; vẫn chói thì nhân thêm hệ số cho riêng nấc đó.

Phân tầng bằng độ sáng: page (neutral-900) → surface (neutral-800). Popover hay modal cần nổi hơn nữa
thì thêm `color/bg/raised` trỏ neutral-700, và **đo lại mọi chữ đặt trên nó**: `text/muted` sẽ tụt dưới
4,5. Bóng đổ trên nền tối gần như vô hình, nên đừng dựa vào nó để tách lớp.

Semantic ở Dark dùng nấc 400 (sáng hơn), nền tint dùng nấc 900. Đo lại chữ trên tint vì hai nấc này gần
nhau hơn ở Light.

## 11. Khi file đã có hệ màu

Đọc trước, chỉ đọc:

```js
const cols = await figma.variables.getLocalVariableCollectionsAsync()
const colorVars = await figma.variables.getLocalVariablesAsync('COLOR')
const paintStyles = await figma.getLocalPaintStylesAsync()
return {
  collections: cols.map((c) => ({
    name: c.name,
    modes: c.modes.map((m) => m.name),
    colors: colorVars.filter((v) => v.variableCollectionId === c.id).length,
    sample: colorVars.filter((v) => v.variableCollectionId === c.id).slice(0, 12).map((v) => v.name),
  })),
  paintStyleCount: paintStyles.length,
  paintStyles: paintStyles.slice(0, 40).map((s) => s.name),
}
```

Sau đó:
- Lập bảng "vai trò ở §3 → biến có sẵn". Vai trò nào thiếu (hay thiếu nhất là `border/input`,
  `text/muted` đạt 4,5, `focus-ring`, mode Dark) thì **đề xuất thêm vào collection có sẵn**, theo đúng
  quy ước đặt tên của file, không theo tên ở đây.
- Không đổi tên, không xoá biến của người dùng khi chưa hỏi. Đổi tên làm vỡ liên kết ở file khác dùng
  thư viện này.
- Biến gắn từ thư viện team (`variable.remote === true`) là hệ của team: dùng nó, đừng tạo bản local
  song song.
- Chạy `audit_design {scope: "palette"}` trên hệ có sẵn và báo kết quả **trước** khi đề xuất sửa.
