# Kích thước và bề mặt — điều khiển, vùng bấm, bo góc, độ nổi, mật độ

Mọi số là mặc định. File có component, radius variables, effect styles → dùng của file.

---

## 1. Chiều cao điều khiển

| Mức | Nút, ô nhập, select | Dùng khi |
|---|---|---|
| Dày | 28–32px | Bảng, toolbar, người dùng chuyên nghiệp |
| Mặc định | 36–40px | Phần lớn giao diện desktop |
| Thoáng | 44–48px | Mobile, form quan trọng, người dùng phổ thông |

- **Nút, ô nhập, select nằm cùng hàng phải cùng chiều cao.** Lệch 4px giữa nút và ô cạnh nhau là
  lỗi nhìn thấy ngay. `inspect_nodes` trả `height` của từng con trong hàng — so là thấy.
- Trong Figma: mỗi component điều khiển có biến thể `Size=S / M / L` với chiều cao **cố định**
  32 / 40 / 48 (`resize` rồi giữ `layoutSizingVertical = 'FIXED'`) và `counterAxisAlignItems = 'CENTER'`.
  Padding ngang 12 / 16 / 20, **padding dọc 0** (công thức dựng cả set: skill `components-states`).
- Vì sao không dựng chiều cao bằng padding dọc: nhãn 14/20 cộng padding 10 mỗi bên mới ra 40 —
  10 lệch nấc 4, và chỉ cần đổi font là chiều cao trượt. Chiều cao cố định + căn giữa thì luôn đúng.
- Dòng bảng theo cùng thang: 32 / 40 / 48 (§6).

## 2. Vùng bấm

| Bối cảnh | Tối thiểu | Nên dùng | Mốc |
|---|---|---|---|
| Web desktop | 24×24 | **32×32** | WCAG 2.2 — 2.5.8 (AA) |
| Web trên mobile | 44×44 | 44–48 | WCAG 2.5.5 (AAA) |
| iOS | 44×44pt | 44pt | Apple HIG |
| Android | 48×48dp | 48dp | Material |

- **Hai vùng bấm cách nhau ≥ 8px.** Nút phá huỷ cách nút chính ≥ 24px.
- **Vùng bấm được lớn hơn phần nhìn thấy:** icon 16px vẫn có vùng bấm 40px (desktop) hay 44–48
  (mobile) nhờ frame bọc ngoài. Bắt buộc với nút chỉ có icon — kèm tooltip nói nó làm gì.
- Link nằm giữa đoạn văn được WCAG miễn ngưỡng 24px; link đứng riêng (menu, danh sách) thì không.
- `audit_design` báo `vung-bam-nho` (dưới 24×24) và `vung-bam-mobile` (dưới 44 trong frame rộng
  ≤ 480px). Sửa bằng frame bọc, không phóng to icon.

```js
// Nút chỉ có icon: vùng bấm 40×40, icon 16 nằm giữa
const icon = await figma.getNodeByIdAsync('ICON_COMPONENT_ID')
if (!icon || icon.type !== 'COMPONENT') throw new Error('Cần id component icon 16px')
const button = figma.createComponent()
button.name = 'IconButton/Close'
button.layoutMode = 'HORIZONTAL'
button.primaryAxisAlignItems = 'CENTER'
button.counterAxisAlignItems = 'CENTER'
button.resize(40, 40) // mobile: 44 (iOS) hoặc 48 (Android)
button.layoutSizingHorizontal = 'FIXED'
button.layoutSizingVertical = 'FIXED'
button.cornerRadius = 6
button.fills = [] // vùng bấm trong suốt; nền chỉ hiện ở biến thể hover
button.appendChild(icon.createInstance())
button.description = 'Chỉ có icon — luôn kèm tooltip "Đóng"'
return { id: button.id }
```

## 3. Bo góc

| Bán kính | Dùng cho |
|---|---|
| 0–2px | Bảng, dòng danh sách |
| 4–6px | Nút, ô nhập, tag, checkbox |
| 8–12px | Thẻ, modal, panel |
| 999px | Pill, avatar (Figma tự giới hạn ở nửa chiều cao) |

- **Phần tử lồng trong bo nhỏ hơn phần tử bao ngoài:** `bán kính trong = bán kính ngoài − padding`.
  Thẻ 12, padding 4 → ảnh bên trong 8. Khi padding ≥ bán kính ngoài (thẻ 12, padding 16), phép
  trừ ra 0 hoặc số âm → phần tử trong lấy bán kính theo vai trò của nó (ô nhập vẫn 6).
- Một họ bán kính cho cả sản phẩm. Gắn `cornerRadius` vào variable `radius/*`
  ([spacing-layout.md §11](spacing-layout.md#11-dựng-và-gắn-spacing-variables)).
- Bo riêng từng góc: `topLeftRadius`, `topRightRadius`, `bottomLeftRadius`, `bottomRightRadius`
  — ví dụ bottom sheet chỉ bo hai góc trên.

## 4. Tách bề mặt

Thử theo thứ tự, dừng ở bước đầu tiên đủ tác dụng:

| # | Cách | Trong Figma |
|---|---|---|
| 1 | Khoảng trắng | `itemSpacing` giữa hai vùng lớn hơn |
| 2 | Đường kẻ mảnh | `strokes` 1px màu border của file, `strokeAlign = 'INSIDE'`. Kẻ một cạnh: `strokeBottomWeight = 1`, ba cạnh còn lại 0 |
| 3 | Nền nhạt khác màu | fill bằng token nền phụ (subtle) |
| 4 | Bóng đổ | effect style mức 2 trở lên (§5) |

- **Bóng chỉ cho thứ thật sự nổi lên trên thứ khác** — dropdown, popover, modal, toast. Thẻ nằm
  phẳng trên nền không cần bóng.
- Đường kẻ chia vùng là trang trí, không bắt buộc 3:1. **Viền ô nhập thì bắt buộc ≥ 3:1** — nó là
  thứ duy nhất cho biết ô nằm ở đâu (`audit_design` báo `ranh-gioi-o-nhap`).
- **Dark mode:** bóng gần như vô hình. Tách lớp bằng **độ sáng nền** — bề mặt càng nổi càng sáng.
- Bảng: chỉ kẻ ngang, không kẻ dọc; đổi nền dòng khi hover.

## 5. Thang độ nổi — effect styles

Một thang, khai một lần thành effect styles, không đặt bóng tuỳ hứng:

| Mức | Dùng cho | Bóng khởi đầu (nền sáng) |
|---|---|---|
| 0 · Phẳng | Nội dung, thẻ trên nền | Không |
| 1 · Dính | Header dính, cột ghim, thanh công cụ | Kẻ dưới 1px, hoặc y 1 · blur 2 · 6% |
| 2 · Nổi | Dropdown, popover, gợi ý, tooltip | y 4 · blur 12 · 10% + y 1 · blur 3 · 6% |
| 3 · Modal | Modal, drawer — kèm lớp phủ tối phía sau | y 12 · blur 32 · 16% + y 2 · blur 6 · 8% |
| 4 · Thông báo | Toast | y 8 · blur 24 · 14% |

Màu bóng lấy nấc neutral tối nhất của file thay vì đen thuần. Thứ tự chồng lớp trong Figma là thứ
tự trong `children`: phần tử đứng sau nằm trên (hiện ở trên cùng trong bảng Layers). Toast và
tooltip luôn ở trên cùng.

```js
const ink = { r: 0.06, g: 0.09, b: 0.16 } // thay bằng neutral tối nhất của file
const levels = [ // mỗi lớp bóng: [y, blur, độ đậm]
  { name: 'Elevation/1 Sticky', layers: [[1, 2, 0.06]] },
  { name: 'Elevation/2 Floating', layers: [[4, 12, 0.1], [1, 3, 0.06]] },
  { name: 'Elevation/3 Modal', layers: [[12, 32, 0.16], [2, 6, 0.08]] },
  { name: 'Elevation/4 Toast', layers: [[8, 24, 0.14]] },
]
const existing = (await figma.getLocalEffectStylesAsync()).map((s) => s.name)
const made = []
for (const { name, layers } of levels) {
  if (existing.includes(name)) continue
  const s = figma.createEffectStyle()
  s.name = name
  s.effects = layers.map(([y, blur, alpha]) => ({
    type: 'DROP_SHADOW', color: { r: ink.r, g: ink.g, b: ink.b, a: alpha }, offset: { x: 0, y },
    radius: blur, spread: 0, visible: true, blendMode: 'NORMAL',
  }))
  made.push({ id: s.id, name })
}
return made // gắn: await node.setEffectStyleIdAsync(id)
```

## 6. Mật độ

App dày hơn website: khoảng trắng để tạo thứ bậc, không để trang trí. App dùng hằng ngày nên cho
người dùng chọn mật độ.

| | Dày | Mặc định | Thoáng |
|---|---|---|---|
| Dòng bảng | 32px | 40px | 48px |
| Ô nhập | 28px | 36px | 44px |
| Padding thẻ | 12px | 16px | 24px |
| Cỡ chữ | 13px | 14px | 15px |

**Dày mà không rối:** căn thẳng hàng nghiêm ngặt · ít đường kẻ · ít màu · chênh lệch cỡ và độ đậm
rõ hơn bình thường.

Trong Figma, mật độ là **một collection có ba mode** — đổi mode trên khung là cả màn hình đổi theo:

```js
const col = figma.variables.createVariableCollection('Density')
const compact = col.modes[0].modeId
col.renameMode(compact, 'Compact')
const normal = col.addMode('Default')
const comfy = col.addMode('Comfortable')
const make = (name = '', values = [0, 0, 0]) => {
  const v = figma.variables.createVariable(name, col, 'FLOAT')
  v.setValueForMode(compact, values[0])
  v.setValueForMode(normal, values[1])
  v.setValueForMode(comfy, values[2])
  return v
}
const rowHeight = make('density/row-height', [32, 40, 48])
rowHeight.scopes = ['WIDTH_HEIGHT']
const cardPadding = make('density/card-padding', [12, 16, 24])
cardPadding.scopes = ['GAP']
const bodySize = make('density/body-size', [13, 14, 15])
bodySize.scopes = ['FONT_SIZE']
// Gắn: row.setBoundVariable('height', rowHeight) · card.setBoundVariable('paddingLeft', cardPadding)
//      textStyle.setBoundVariable('fontSize', bodySize)
// Xem thử: screen.setExplicitVariableModeForCollection(col, comfy)
return { collection: col.id }
```

`addMode` lỗi khi gói Figma của người dùng giới hạn số mode — báo cho người dùng và dựng ba frame
mẫu cho ba mức thay vì cố tạo mode.

## 7. Icon

- Ba cỡ: **16 · 20 · 24**, mỗi icon là component vuông đúng cỡ đó, hình vẽ nằm giữa khung — thay
  icon không làm lệch hàng.
- **Nét đều trong cả bộ:** cùng cỡ thì cùng độ dày nét, cùng kiểu đầu nét và góc. Trộn hai bộ icon
  là lỗi dễ thấy nhất sau lỗi căn hàng.
- Icon cạnh chữ: gap 4–8, `counterAxisAlignItems = 'CENTER'`; icon 16 đi với chữ 14, icon 20 đi
  với chữ 16.
- Màu icon theo màu chữ đứng cạnh nó. Icon đứng một mình và mang nghĩa (không trang trí) cần
  contrast ≥ 3:1 với nền.
- Trong component có icon đổi được: thuộc tính `INSTANCE_SWAP`, không tạo biến thể cho từng icon.
- Icon không đối xứng (tam giác, mũi tên) căn đúng tâm khung vẫn có thể trông lệch vì phần "nặng"
  của hình dồn về một phía. Chỉ chỉnh vị trí hình vẽ bên trong component icon (1px) khi
  `screenshot` phóng to cho thấy lệch rõ; không chỉnh theo phỏng đoán.

## 8. Ảnh

| Tỉ lệ | Dùng cho |
|---|---|
| 1:1 | Avatar, thumbnail, lưới sản phẩm |
| 4:3 · 3:2 | Thẻ sản phẩm, ảnh bài viết |
| 16:9 | Video, hero, ảnh bìa |
| 4:5 · 3:4 | Ảnh dọc trên mobile, feed |

- Sandbox **không có mạng**: `figma.createImageAsync(url)` sẽ lỗi. Ảnh thật đi qua tool `place_image`
  (file trên máy hoặc URL): vẽ khung đúng tỉ lệ trước rồi truyền `node_id` để ảnh lấp khung, `scale_mode`
  `FILL` để cắt vừa. Chưa có ảnh thì để placeholder (khung nền neutral, tên nói nội dung và tỉ lệ) hoặc
  dùng lại image fill có sẵn trong file của người dùng.
- Ảnh trong thẻ auto layout: `FILL` bề ngang + `lockAspectRatio()` để chiều cao đi theo tỉ lệ.
- **Chữ đè lên ảnh** cần lớp phủ tối 40–60% hoặc dải gradient tối sau chữ; đừng tin "ảnh này chắc
  đủ tối". `audit_design` trả `chu-khong-do-duoc` (SKIP) cho trường hợp này — phải tự nhìn bằng
  `screenshot`.

```js
const card = await figma.getNodeByIdAsync('CARD_ID')
if (!card || card.type !== 'FRAME' || card.layoutMode === 'NONE') throw new Error('Cần thẻ có auto layout')
const img = figma.createFrame()
img.name = 'Ảnh · Sản phẩm 4:3'
img.resize(400, 300)
img.fills = [{ type: 'SOLID', color: { r: 0.93, g: 0.94, b: 0.96 } }]
img.cornerRadius = 8
card.insertChild(0, img)
img.layoutSizingHorizontal = 'FILL'
img.lockAspectRatio() // giữ 4:3 khi thẻ đổi bề ngang
return { id: img.id }
```
