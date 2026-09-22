# Component trong Figma — cơ chế và cách dựng

Mọi đoạn code dưới đây là **thân của một lần gọi `execute_figma_code`** (JavaScript, `figma` có sẵn,
dùng `await` thoải mái). Mỗi lần gọi là một bước undo, tối đa 60 giây, kết quả trả về tối đa
30.000 ký tự — nên trả **id và con số**, đừng trả cả cây node. Chữ in hoa như `SET_ID` là chỗ điền
id thật lấy từ lần gọi trước hoặc từ `inspect_nodes`.

## Mục lục

1. Main component và instance
2. Tìm component có sẵn trước khi vẽ
3. Chọn loại property
4. Dựng một component set từ đầu — Button
5. Dùng instance
6. Instance lồng nhau, override và expose
7. Cấu trúc file và trang Components
8. Kiểm mọi variant
9. Prototype: component tương tác
10. Ghi chú cho dev trên canvas

---

## 1. Main component và instance

| | Main component (`COMPONENT`) | Instance (`INSTANCE`) |
|---|---|---|
| Là gì | Bản gốc, nằm trên trang Components | Bản dùng trên màn hình |
| Đổi cấu trúc (thêm, bớt layer) | Được — lan ra mọi instance | **Không** — chỉ override |
| Override được | — | Chữ, fill, stroke, visible, effect, property, swap instance lồng |

- **Sửa main là sửa mọi nơi** trừ chỗ đã override. Trước khi sửa main đang dùng, đếm
  `(await comp.getInstancesAsync()).length` và báo người dùng con số đó.
- **Không detach** (`detachInstance()`) để sửa nhanh: mất liên kết, lần sau sửa main không lan tới.
  Cần khác đi → thêm variant hoặc property.
- Đổi sang component khác: `instance.swapComponent(other)` **giữ** override;
  gán `instance.mainComponent = other` **xoá** override. `instance.removeOverrides()` đưa về như main.
- File dùng dynamic page: đọc main bằng `await instance.getMainComponentAsync()`; `mainComponent`
  chỉ ghi được, không đọc được.

## 2. Tìm component có sẵn trước khi vẽ

```js
await figma.loadAllPagesAsync() // file rất lớn: tách thành nhiều lần gọi, mỗi lần vài trang
const out = []
for (const page of figma.root.children) {
  for (const n of page.findAllWithCriteria({ types: ['COMPONENT_SET', 'COMPONENT'] })) {
    if (n.parent?.type === 'COMPONENT_SET') continue // variant: đã tính trong set
    const props = Object.entries(n.componentPropertyDefinitions).map(
      ([k, d]) => `${k}: ${d.type}${d.variantOptions ? ' = ' + d.variantOptions.join('|') : ''}`,
    )
    out.push({ page: page.name, id: n.id, name: n.name, type: n.type, props })
  }
}
return { total: out.length, items: out.slice(0, 80) } // giữ kết quả dưới 30.000 ký tự
```

Đọc kết quả trước khi quyết: tên property và giá trị file đang dùng (`Type` hay `Kind`, `Hover` hay
`Hovered`) là quy ước phải theo. Chỉ đọc `componentPropertyDefinitions` trên set hoặc component
đứng riêng — gọi trên một variant sẽ lỗi.

## 3. Chọn loại property

| Muốn đổi | Loại | Nối vào | Ví dụ |
|---|---|---|---|
| Hình, màu, kích thước theo nhóm cố định | `VARIANT` | tên variant | `Kind`, `Size`, `State`, `Checked` |
| Hiện hoặc ẩn một layer | `BOOLEAN` | `visible` | `Show icon`, `Show helper` |
| Chữ | `TEXT` | `characters` | `Label`, `Helper text` |
| Đổi layer con thành component khác | `INSTANCE_SWAP` | `mainComponent` | `Icon` |

- Số variant là **tích các trục**: Kind 4 × Size 3 × State 6 = 72 — vẫn quản được. Vượt khoảng 100
  thì tách set (icon button tách khỏi button, input có icon tách khỏi input thường).
- **Không làm variant cho thứ chỉ khác chữ hoặc khác icon** — đó là việc của TEXT và INSTANCE_SWAP.
- Tên property và giá trị: tiếng Anh, viết hoa chữ đầu, khớp tên prop phía code nếu đã biết.
- Trạng thái là **một trục** `State`, không phải nhiều boolean (`isHover`, `isDisabled`) — boolean
  cho phép những tổ hợp vô nghĩa như vừa Disabled vừa Loading.

## 4. Dựng một component set từ đầu — Button

Ba lần gọi: dựng variant và gộp set → thêm property và icon → đưa lên trang Components kèm tài liệu.

### 4.1. Gọi 1 — 4 Kind × 6 State, gộp set, xếp lưới

Hàm `paint(role)` tìm biến màu theo **đuôi tên** (`color/action/hover` → `action-hover`,
`color/action/default` → `action`) và gắn biến vào fill; không có biến thì dùng hex dự phòng và ghi
lại vai trò còn thiếu. Bốn vai trò `bg-subtle-active`, `action-subtle-bg-active`, `danger-hover`,
`danger-active` là vai trò thêm, thường chưa có trong bảng token — cách thêm ở [states.md §2](states.md#2-màu-theo-nấc-ramp--hover-pressed).
Variant Disabled dùng `bg-disabled` và `text-disabled` khi file có, không thì `bg-subtle` và
`text-secondary` ([states.md §4](states.md#4-disabled)).

```js
const STATES = ['Default', 'Hover', 'Pressed', 'Focus', 'Disabled', 'Loading']
const KINDS = { // nền [Default, Hover, Pressed] · chữ · viền ('' = không có)
  Primary: { bg: ['action', 'action-hover', 'action-active'], fg: 'text-on-action', stroke: '' },
  Secondary: { bg: ['bg-surface', 'bg-subtle', 'bg-subtle-active'], fg: 'text-primary', stroke: 'border-input' },
  Tertiary: { bg: ['', 'action-subtle-bg', 'action-subtle-bg-active'], fg: 'action-subtle-text', stroke: '' },
  Destructive: { bg: ['danger', 'danger-hover', 'danger-active'], fg: 'text-on-action', stroke: '' },
}
const FALLBACK = new Map(Object.entries({
  'action': '#2563EB', 'action-hover': '#1D4ED8', 'action-active': '#1E40AF', 'text-on-action': '#FFFFFF',
  'bg-surface': '#FFFFFF', 'bg-subtle': '#F1F5F9', 'bg-subtle-active': '#E2E8F0', 'border-input': '#64748B',
  'text-primary': '#0F172A', 'text-secondary': '#475569', 'action-subtle-bg': '#EFF6FF',
  'action-subtle-bg-active': '#DBEAFE', 'action-subtle-text': '#1D4ED8', 'danger': '#DC2626',
  'danger-hover': '#B91C1C', 'danger-active': '#991B1B', 'focus-ring': '#2563EB',
}))
const vars = await figma.variables.getLocalVariablesAsync('COLOR')
const tails = (name = '') => { // 'color/action/hover' → [..., 'action-hover', 'hover']; '/default' bị bỏ
  const parts = name.toLowerCase().split('/').map((p) => p.trim().replace(/[\s_.]+/g, '-'))
  return parts.map((_, i) => parts.slice(i).join('-').replace(/-default$/, ''))
}
const missing = new Set()
const paint = (role = '') => {
  const p = figma.util.solidPaint(FALLBACK.get(role) ?? '#FF00FF')
  const v = vars.find((x) => tails(x.name).includes(role))
  if (!v) { missing.add(role); return p }
  return figma.variables.setBoundVariableForPaint(p, 'color', v)
}
const has = (role = '') => vars.some((x) => tails(x.name).includes(role))
const OFF_BG = has('bg-disabled') ? 'bg-disabled' : 'bg-subtle' // file có vai trò disabled riêng thì dùng
const OFF_FG = has('text-disabled') ? 'text-disabled' : 'text-secondary'
const labelStyle = (await figma.getLocalTextStylesAsync()).find((s) => s.fontSize === 14 && /medium|semi/i.test(s.fontName.style))
const font = labelStyle ? labelStyle.fontName : { family: 'Inter', style: 'Medium' }
await figma.loadFontAsync(font)

const variants = []
for (const [kind, look] of Object.entries(KINDS)) {
  for (const state of STATES) {
    const c = figma.createComponent()
    c.name = `Kind=${kind}, State=${state}`
    c.layoutMode = 'HORIZONTAL'
    c.resize(120, 40) // cao 40: mặc định product
    c.primaryAxisSizingMode = 'AUTO' // rộng theo chữ
    c.counterAxisSizingMode = 'FIXED'
    c.primaryAxisAlignItems = 'CENTER'
    c.counterAxisAlignItems = 'CENTER'
    c.minWidth = 96
    c.paddingLeft = c.paddingRight = 16
    c.itemSpacing = 8
    c.cornerRadius = 6
    const off = state === 'Disabled'
    const step = state === 'Hover' ? 1 : state === 'Pressed' ? 2 : 0
    const bg = off ? (kind === 'Tertiary' ? '' : OFF_BG) : look.bg[step]
    c.fills = bg ? [paint(bg)] : []
    if (look.stroke && !off) { c.strokes = [paint(look.stroke)]; c.strokeWeight = 1; c.strokeAlign = 'INSIDE' }

    const label = figma.createText()
    label.name = 'Label'
    label.fontName = font
    label.fontSize = 14
    label.lineHeight = { unit: 'PIXELS', value: 20 }
    label.characters = state === 'Loading' ? 'Đang lưu…' : 'Lưu thay đổi'
    label.fills = [paint(off ? OFF_FG : look.fg)]
    c.appendChild(label)
    if (labelStyle) await label.setTextStyleIdAsync(labelStyle.id)

    if (state === 'Loading') {
      const spin = figma.createEllipse()
      spin.name = 'Spinner'
      spin.resize(16, 16)
      spin.arcData = { startingAngle: 0, endingAngle: Math.PI * 1.5, innerRadius: 0.75 }
      spin.fills = [paint(look.fg)]
      c.insertChild(0, spin)
    }
    if (state === 'Focus') {
      const ring = figma.createFrame() // vòng 2px, cách mép 2px
      ring.name = 'Focus ring'
      ring.fills = []
      ring.strokes = [paint('focus-ring')]
      ring.strokeWeight = 2
      ring.strokeAlign = 'INSIDE'
      ring.cornerRadius = 6 + 4
      c.appendChild(ring)
      ring.layoutPositioning = 'ABSOLUTE'
      ring.resize(c.width + 8, c.height + 8)
      ring.x = -4
      ring.y = -4
      ring.constraints = { horizontal: 'STRETCH', vertical: 'STRETCH' }
      c.clipsContent = false
    }
    variants.push(c)
  }
}

const set = figma.combineAsVariants(variants, figma.currentPage)
set.name = 'Button'
set.layoutMode = 'NONE' // xếp lưới bằng toạ độ
const kinds = Object.keys(KINDS)
const colW = Math.max(...variants.map((v) => v.width)) + 32
for (const v of variants) { // hàng = Kind, cột = State; Primary/Default ở góc trên trái
  const p = v.variantProperties
  if (!p) continue
  v.x = 32 + STATES.indexOf(p.State) * colW
  v.y = 32 + kinds.indexOf(p.Kind) * 72
}
set.resizeWithoutConstraints(STATES.length * colW + 32, kinds.length * 72 + 24)
set.x = Math.max(0, ...figma.currentPage.children.filter((n) => n !== set).map((n) => n.x + n.width)) + 120
set.y = 0
figma.currentPage.selection = [set]
figma.viewport.scrollAndZoomIntoView([set])
return { setId: set.id, variants: variants.length, missingTokens: [...missing] }
```

- Thêm trục `Size`: bọc thêm một vòng lặp, S cao 32 / padding 12, M cao 40 / padding 16, L cao 48 /
  padding 20; tên thành `Kind=…, Size=…, State=…`, hàng của lưới là Kind × Size.
- `missingTokens` khác rỗng → nói với người dùng; biến màu nên được thêm bằng skill `color-system`
  rồi gắn lại, thay vì để hex dự phòng nằm lâu trong file.
- Xong gọi `screenshot` với `node_id` là set.

### 4.2. Gọi 2 — property Label, Loading label, Show icon, Icon

```js
const set = await figma.getNodeByIdAsync(SET_ID)
if (set?.type !== 'COMPONENT_SET') throw new Error('SET_ID phải là id của một component set')
// Icon: dùng icon của file nếu có (đổi tên cần tìm); không có thì tạo một component giữ chỗ 16×16
let icon = figma.currentPage.findAllWithCriteria({ types: ['COMPONENT'] }).find((n) => n.name === 'Icon/Placeholder')
if (!icon) {
  icon = figma.createComponent()
  icon.name = 'Icon/Placeholder'
  icon.resize(16, 16)
  icon.fills = []
  const box = figma.createRectangle()
  box.name = 'Shape'
  box.resize(12, 12)
  box.x = box.y = 2
  box.cornerRadius = 2
  box.fills = []
  box.strokes = [figma.util.solidPaint('#475569')]
  box.strokeWeight = 1.5
  icon.appendChild(box)
  icon.x = set.x
  icon.y = set.y - 48
}
const labelKey = set.addComponentProperty('Label', 'TEXT', 'Lưu thay đổi')
const loadingKey = set.addComponentProperty('Loading label', 'TEXT', 'Đang lưu…')
const showIconKey = set.addComponentProperty('Show icon', 'BOOLEAN', false)
const iconKey = set.addComponentProperty('Icon', 'INSTANCE_SWAP', icon.id)

for (const v of set.findAllWithCriteria({ types: ['COMPONENT'] })) { // mỗi variant
  const label = v.findAllWithCriteria({ types: ['TEXT'] }).find((n) => n.name === 'Label')
  if (!label || label.fontName === figma.mixed) continue
  await figma.loadFontAsync(label.fontName)
  const loading = v.variantProperties?.State === 'Loading'
  label.componentPropertyReferences = { characters: loading ? loadingKey : labelKey }
  if (loading) continue // Loading dùng chỗ của icon cho spinner
  const inst = icon.createInstance()
  inst.name = 'Icon'
  v.insertChild(0, inst)
  const shape = inst.findOne((n) => n.name === 'Shape')
  if (shape && 'strokes' in shape && label.fills !== figma.mixed) shape.strokes = label.fills // cùng màu chữ, giữ cả biến
  inst.componentPropertyReferences = { visible: showIconKey, mainComponent: iconKey }
}
return { labelKey, loadingKey, showIconKey, iconKey }
```

- Property thêm trên **set** là của mọi variant; phần **nối** (`componentPropertyReferences`) phải
  làm trong từng variant — bỏ sót một variant là instance ở trạng thái đó không đổi chữ được.
- Muốn giới hạn icon được chọn: `set.editComponentProperty(iconKey, { preferredValues: icons.map((c) => ({ type: 'COMPONENT', key: c.key })) })`.
- Thêm một trục variant vào set có sẵn: `set.addComponentProperty('Size', 'VARIANT', 'M')` — mọi
  variant hiện có nhận giá trị `M`, sau đó nhân bản và đổi tên cho S, L.

### 4.3. Gọi 3 — đưa lên trang Components kèm tài liệu

Khung tài liệu là một frame thường (không auto layout) để nhãn hàng và cột đặt theo toạ độ của
variant. Hỏi người dùng trước khi thêm trang mới vào file của họ.

```js
const set = await figma.getNodeByIdAsync(SET_ID)
if (set?.type !== 'COMPONENT_SET') throw new Error('SET_ID phải là id của một component set')
let page = figma.root.children.find((p) => p.name === 'Components')
if (!page) { page = figma.createPage(); page.name = 'Components' }
await figma.setCurrentPageAsync(page)
await figma.loadFontAsync({ family: 'Inter', style: 'Semi Bold' })
await figma.loadFontAsync({ family: 'Inter', style: 'Regular' })
const text = (chars = '', size = 12, style = 'Regular', x = 0, y = 0, width = 0) => {
  const t = figma.createText()
  t.fontName = { family: 'Inter', style }
  t.fontSize = size
  t.characters = chars
  if (width) { t.resize(width, t.height); t.textAutoResize = 'HEIGHT' }
  t.x = x
  t.y = y
  return t
}
const doc = figma.createFrame()
doc.name = 'Doc / Button'
doc.y = Math.max(0, ...page.children.filter((n) => n !== doc).map((n) => n.y + n.height + 160))
doc.appendChild(text('Button', 24, 'Semi Bold', 40, 40))
doc.appendChild(text('Một hành động bấm được. Mỗi màn hình đúng một nút Primary; nhãn là động từ + danh từ. ' +
  'Disabled phải kèm câu nói vì sao; Loading giữ nguyên bề rộng.', 14, 'Regular', 40, 80, 640))
const X0 = 160
const Y0 = 180
doc.appendChild(set)
set.x = X0
set.y = Y0
for (const v of set.findAllWithCriteria({ types: ['COMPONENT'] })) {
  const p = v.variantProperties
  if (!p) continue
  if (p.Kind === 'Primary') doc.appendChild(text(p.State, 12, 'Regular', X0 + v.x, Y0 - 24))
  if (p.State === 'Default') doc.appendChild(text(p.Kind, 12, 'Regular', 40, Y0 + v.y + 12))
}
doc.resize(X0 + set.width + 40, Y0 + set.height + 40)
// Icon giữ chỗ (giá trị mặc định của INSTANCE_SWAP) đi theo sang trang Components
const swap = Object.values(set.componentPropertyDefinitions).find((d) => d.type === 'INSTANCE_SWAP')
const icon = swap ? await figma.getNodeByIdAsync(String(swap.defaultValue)) : null
if (icon?.type === 'COMPONENT' && icon.parent?.type === 'PAGE' && icon.parent !== page) {
  page.appendChild(icon)
  icon.x = doc.x - 120
  icon.y = doc.y
}
figma.currentPage.selection = [doc]
figma.viewport.scrollAndZoomIntoView([doc])
return { docId: doc.id }
```

Khung tài liệu cần thêm, khi đã có dữ liệu thật: bảng property (tên · loại · giá trị · khi nào
dùng), một hàng "Nên / Không nên" có instance minh hoạ, và ghi chú cho dev (§10). Chữ tài liệu
nên gắn text style và biến màu của file, như mọi chữ khác.

## 5. Dùng instance

```js
const set = await figma.getNodeByIdAsync(SET_ID)
const parent = await figma.getNodeByIdAsync(PARENT_ID) // khung auto layout của màn hình
if (set?.type !== 'COMPONENT_SET') throw new Error('SET_ID phải là id của một component set')
if (parent?.type !== 'FRAME') throw new Error('PARENT_ID phải là một frame')
const defs = set.componentPropertyDefinitions
const key = (name = '') => { // 'Label' → 'Label#12:3'
  const found = Object.keys(defs).find((id) => id.split('#')[0] === name)
  if (!found) throw new Error(`Set chưa có property ${name}`)
  return found
}
const main = set.findAllWithCriteria({ types: ['COMPONENT'] })
  .find((c) => c.variantProperties?.Kind === 'Primary' && c.variantProperties?.State === 'Default')
if (!main) throw new Error('Thiếu variant Kind=Primary, State=Default')

const save = main.createInstance()
parent.appendChild(save)
for (const t of save.findAllWithCriteria({ types: ['TEXT'] })) {
  if (t.fontName !== figma.mixed) await figma.loadFontAsync(t.fontName)
}
save.setProperties({ [key('Label')]: 'Gửi báo cáo' })

// Minh hoạ lúc đang gửi: đo bề rộng ở Default rồi khoá lại, để nút không co giãn
const sending = save.clone()
parent.appendChild(sending)
const width = sending.width
sending.setProperties({ State: 'Loading', [key('Loading label')]: 'Đang gửi…' })
sending.layoutSizingHorizontal = 'FIXED'
sending.resize(width, sending.height)
return { ids: [save.id, sending.id] }
```

- `audit_design` nhận ra vùng bấm **qua tên layer**. Đọc lại `name` của instance vừa tạo: phải là
  `Button` (hoặc tên có chữ button); nếu nó mang tên variant (`Kind=…, State=…`) thì đặt lại `name`.
- Đổi trạng thái bằng `setProperties({ State: '…' })`, không đổi fill bằng tay trên instance.

## 6. Instance lồng nhau, override và expose

Component lớn thường chứa instance của component nhỏ: ô nhập chứa icon và nút xoá, dòng bảng chứa
checkbox, modal chứa button.

- Override instance lồng từ instance ngoài: tìm theo tên rồi `setProperties` trên chính nó —
  `row.findOne((n) => n.type === 'INSTANCE' && n.name === 'Checkbox').setProperties({ Checked: 'On' })`.
- **Expose** để người dùng chỉnh instance lồng ngay ở panel của instance ngoài. Đặt trong main
  component, trên instance lồng **không nằm trong một instance khác** (instance "sơ cấp" của main):

```js
const input = await figma.getNodeByIdAsync(INPUT_SET_ID)
if (input?.type !== 'COMPONENT_SET') throw new Error('INPUT_SET_ID phải là id của một component set')
for (const v of input.findAllWithCriteria({ types: ['COMPONENT'] })) {
  const clear = v.findAllWithCriteria({ types: ['INSTANCE'] }).find((n) => n.name === 'Clear button')
  if (clear) clear.isExposedInstance = true
}
return 'ok'
// Sau đó, ở một instance của Input trên màn hình:
// const [clearButton] = field.exposedInstances
// clearButton.setProperties({ State: 'Hover' })
```

- Đổi icon lồng: dùng property INSTANCE_SWAP (§4.2), không swap tay trong từng instance.
- Đặt tên instance lồng theo vai trò (`Clear button`, `Leading icon`) — tên là cách tìm lại nó,
  và là thứ `audit_design` đọc.

## 7. Cấu trúc file và trang Components

Bốn page `Cover` · `Foundations` · `Components` · `Screens`, cách xếp màn theo luồng và frame bìa: skill
`design-workflow`, reference/handoff.md §1. Riêng trang **Components**:

- Mỗi component một khung `Doc / <Tên>`, xếp từ nhỏ đến lớn: icon → button → input, select →
  checkbox, radio, switch → tag, badge → table row → nav, tab → toast, banner → modal, drawer → empty state.
- Mỗi khung `Doc / <Tên>`: tiêu đề · một câu công dụng và khi nào **không** dùng · lưới đủ variant
  có nhãn hàng, cột · bảng property · ví dụ nên/không nên · ghi chú cho dev.
- File đã có cấu trúc riêng → theo file, chỉ đề xuất. Thêm hoặc đổi tên trang là thay đổi lớn với
  người dùng: hỏi trước.
- Tạo trang: `figma.createPage()`; chuyển trang: `await figma.setCurrentPageAsync(page)` — người
  dùng sẽ thấy màn hình của họ đổi trang, nói trước cho họ biết.

## 8. Kiểm mọi variant

Kiểm màn hình chỉ đo được những variant **đang được đặt** lên màn hình. Variant Hover hay Disabled
chưa ai dùng vẫn có thể sai contrast, thiếu property, lệch kích thước. Nên kiểm trên chính set:

```js
const set = await figma.getNodeByIdAsync(SET_ID)
if (set?.type !== 'COMPONENT_SET') throw new Error('SET_ID phải là id của một component set')
const variants = set.findAllWithCriteria({ types: ['COMPONENT'] })
const defs = set.componentPropertyDefinitions
const axes = Object.keys(defs).filter((k) => defs[k].type === 'VARIANT')
let combos = [''] // mọi tổ hợp giá trị của các trục VARIANT
for (const k of axes) combos = combos.flatMap((c) => (defs[k].variantOptions ?? []).map((o) => (c ? c + '|' : '') + o))
const have = new Set(variants.map((v) => axes.map((k) => v.variantProperties?.[k]).join('|')))
const expected = ['Default', 'Hover', 'Pressed', 'Focus', 'Disabled', 'Loading'] // ô nhập: Default, Hover, Focus, Disabled, Error, Success
return {
  total: variants.length,
  missingCombos: combos.filter((c) => !have.has(c)),
  missingStates: expected.filter((s) => !(defs.State?.variantOptions ?? []).includes(s)),
  under24: variants.filter((v) => Math.min(v.width, v.height) < 24).map((v) => v.name),
  under44ForMobile: variants.filter((v) => v.height < 44).length,
  labelNotWired: variants
    .filter((v) => v.findAllWithCriteria({ types: ['TEXT'] }).some((t) => t.name === 'Label' && !t.componentPropertyReferences?.characters))
    .map((v) => v.name),
}
```

Rồi:

1. `audit_design` với `node_id` = id của set — đo contrast chữ ở **mọi** variant (tính cả opacity
   của layer và của cha), khoảng cách lệch nấc 4, màu chưa gắn biến, chữ chưa có text style, chữ
   tràn khung cắt.
2. `screenshot` cả set ở `scale` 2: nhìn từng cột, che nhãn trong đầu — các ô có còn khác nhau?
3. `under44ForMobile` > 0 mà component dùng cho mobile → cần Size L (44–48), hoặc ghi rõ component
   chỉ dành cho desktop.
4. Ô nhập: nếu `ranh-gioi-o-nhap` báo FAIL trên một khung bao **không có viền và nền** (component
   set `Input`, hay instance `Input` chứa Label, Field, Helper) thì đó là đo nhầm chỗ. Phép đo có
   nghĩa nằm ở layer `Field` bên trong — đọc dòng của `Field`, và ghi rõ điều này trong báo cáo.

`audit_design` bỏ qua layer ẩn (vòng focus ở variant khác không bị đo nhầm) và chỉ đo vùng bấm
trên layer có tên giống nút — variant tên `Kind=…, State=…` không được đo, nên cột `under24` ở
trên là phần bù.

## 9. Prototype: component tương tác

Cho Default → Hover khi rê chuột và Hover → Pressed khi nhấn, để prototype tự đổi trạng thái:

```js
const set = await figma.getNodeByIdAsync(SET_ID)
if (set?.type !== 'COMPONENT_SET') throw new Error('SET_ID phải là id của một component set')
const variants = set.findAllWithCriteria({ types: ['COMPONENT'] })
const pick = (kind = '', state = '') => {
  const v = variants.find((c) => c.variantProperties?.Kind === kind && c.variantProperties?.State === state)
  if (!v) throw new Error(`Thiếu variant Kind=${kind}, State=${state}`)
  return v
}
for (const kind of ['Primary', 'Secondary', 'Tertiary', 'Destructive']) {
  const [rest, hover, pressed] = ['Default', 'Hover', 'Pressed'].map((s) => pick(kind, s))
  await rest.setReactionsAsync([{ trigger: { type: 'ON_HOVER' }, actions: [{ type: 'NODE', destinationId: hover.id,
    navigation: 'CHANGE_TO', transition: { type: 'SMART_ANIMATE', easing: { type: 'EASE_OUT' }, duration: 0.12 } }] }])
  await hover.setReactionsAsync([{ trigger: { type: 'ON_PRESS' }, actions: [{ type: 'NODE', destinationId: pressed.id,
    navigation: 'CHANGE_TO', transition: { type: 'SMART_ANIMATE', easing: { type: 'EASE_OUT' }, duration: 0.1 } }] }])
}
return 'ok'
```

- `ON_HOVER` là "While hovering", `ON_PRESS` là "While pressing" — nhả ra thì tự quay về.
- Hover 120ms, nhấn 100ms, ease-out: đổi màu và trạng thái nằm trong nhóm vi mô 100–150ms. Không dùng
  kiểu nảy. Bảng ánh xạ đầy đủ (overlay, drawer, chuyển trang): skill `layout-type`, reference/motion.md §4.
- Focus bằng bàn phím không mô phỏng được trong prototype: variant Focus chỉ để trình bày, kèm ghi chú cho dev.

## 10. Ghi chú cho dev trên canvas

Những hành vi không vẽ được — nhốt focus trong modal, Escape để đóng, focus chỉ hiện khi dùng bàn
phím, khoá bấm lặp khi đang tải, URL giữ bộ lọc, debounce ô tìm, tên cho trình đọc màn hình — ghi
thành chữ, đặt đúng chỗ. Hai lớp, dùng cả hai:

```js
const node = await figma.getNodeByIdAsync(SET_ID)
if (!node || !('annotations' in node)) throw new Error('Node này không nhận annotation')
node.annotations = [...node.annotations, { // giữ ghi chú đã có, chỉ thêm
  labelMarkdown: '**Focus** chỉ hiện khi dùng bàn phím. **Loading**: khoá bấm lặp, giữ bề rộng. ' +
    '**Icon button**: bắt buộc tooltip và tên cho trình đọc màn hình.',
}]
return 'ok'
```

- **Annotation** hiện trong Dev Mode, gắn đúng node — dev đọc khi đo.
- **Khung ghi chú nhìn thấy được** (`Dev note / …`, chữ 12/16, nền nhạt) đặt **cạnh** màn hình hoặc
  trong khung `Doc / <Tên>`, không đặt bên trong frame màn hình — người review không mở Dev Mode vẫn
  đọc được, và `audit_design` không đo nhầm ghi chú như một phần giao diện.
