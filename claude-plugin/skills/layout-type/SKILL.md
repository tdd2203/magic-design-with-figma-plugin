---
name: layout-type
description: 'Quyết và sửa chữ, khoảng cách, bố cục và kích thước trong file Figma đang mở: thang chữ và text style, khoảng cách theo lưới 4, auto layout, layout grid, cỡ nút và ô nhập, vùng bấm, bo góc, đổ bóng, khung web/iOS/Android, chuyển động prototype. Dùng khi người dùng hỏi về cỡ chữ, khoảng cách, căn lề, lưới hay kích thước màn hình trên file Figma nối qua plugin Magic Design with Figma. Dựng cả một màn thì bắt đầu từ design-workflow.'
argument-hint: "<việc cần làm với chữ, khoảng cách hay bố cục>"
---

# Bố cục và chữ trên canvas Figma

Skill này biến các thang đo — khoảng cách, chữ, cỡ điều khiển, vùng bấm, bo góc, độ nổi, chuyển
động — thành **thuộc tính Figma cụ thể**, dựng bằng `execute_figma_code`, kiểm bằng `screenshot` và
`audit_design`. Màu và contrast → skill `color-system`; component và trạng thái →
`components-states`; brief, quy trình → `design-workflow`; spec bàn giao → `handoff-spec`; audit đầy đủ → `design-audit`.

**Tri thức, không phải luật.** Khi các nguồn nói khác nhau, thứ tự ưu tiên là:

1. **Hệ thống có sẵn trong file** — variables, text/effect/grid styles, components của người dùng. Luôn thắng.
2. **`DESIGN.md` của dự án**, nếu người dùng có.
3. **Mặc định trong skill này** — chỉ dùng cho phần hai nguồn trên chưa nói tới.

Làm khác mặc định được, khi đủ ba điều: giải thích bằng **lợi ích cho người dùng** (không phải
"trông đẹp hơn") · áp **nhất quán**, không lẻ tẻ một màn hình · **ghi lại** lý do (mô tả của frame
hoặc `DESIGN.md`) để sau này không ai "sửa lại cho đúng".

## 1. Chọn register trước con số đầu tiên

| | Product / app | Brand / đọc |
|---|---|---|
| Ví dụ | Dashboard, CRM, form nghiệp vụ, công cụ nội bộ | Landing, blog, tài liệu, trang giới thiệu |
| Chữ thân | **14/20** | **16/24** |
| Nút, ô nhập | 36–40px | 36–40px; 44–48px cho mobile, người dùng phổ thông |
| Độ đậm | 2: 400 + 500–600 | 2–3: thêm 700 cho tiêu đề lớn |
| Tỉ lệ giữa hai nấc chữ | 1,125–1,2 | 1,25–1,5 |
| Khoảng trắng | Tạo thứ bậc, không trang trí | Rộng rãi, nhịp chặt–thoáng tương phản |
| Chuyển động | 100–400ms (trên 400 là chậm), chỉ để báo trạng thái | Hero được phép có một điểm nhấn động; phản hồi thao tác vẫn theo thang 100–400ms |

Người dùng lớn tuổi, dùng chủ yếu trên mobile, hoặc nhìn màn hình từ xa → chữ thân 16px dù là
product. Chưa rõ register → hỏi một câu; có bảng hoặc form thì mặc định là product.

## 2. Quy trình trên canvas

1. **Đọc trước khi vẽ.** `get_context` → một lần `execute_figma_code` liệt kê text styles, effect
   styles, grid styles, variables số và font đang dùng
   ([spacing-layout.md §10](reference/spacing-layout.md#10-đọc-thang-có-sẵn-trong-file)).
   `inspect_nodes` trả cấu trúc kèm thuộc tính chữ (fontWeight, lineHeight, letterSpacing, textCase,
   textAutoResize), tên text style và tên variable đã gắn. Gom thang chữ của cả màn thì dùng code
   ([typography.md §10](reference/typography.md#10-gom-thang-chữ-của-cả-màn)).
2. **Chốt thang.** File đã có thang → dùng đúng thang đó, kể cả khi lệch mặc định ở đây. Chưa có →
   tạo spacing/radius variables và bộ text style **trước** khi dựng màn hình, rồi nói với người dùng
   đã thêm gì vào file.
3. **Dựng từ ngoài vào trong bằng auto layout:** khung màn hình → vùng → nhóm → phần tử. Mỗi lần
   `execute_figma_code` một vùng, trả về id để lần sau nối tiếp.
4. **Đổ dữ liệu bẩn** ngay sau khi dựng khung, trước khi chỉnh đẹp: tiếng Việt có dấu chồng ("ĐẶNG
   THỊ NGỌC ẨN", "Nguyễn Thị Hường"), tên 200 ký tự không dấu cách, tên 1 ký tự, số âm · 0 ·
   1.234.567.890, danh sách 0 · 1 · 5.000 dòng.
5. **Kiểm:** `screenshot` (thêm một lần ở `scale` nhỏ để nheo mắt — §7) → `audit_design` trên frame
   vừa dựng → sửa FAIL, cân nhắc WARN. **SKIP nghĩa là máy không đo được, không phải đạt** — tự
   nhìn lại bằng screenshot.

## 3. Thang mặc định — một trang

```
KHOẢNG CÁCH   bội số 4: 4 8 12 16 20 24 32 40 48 64 80 96
GẦN = CÙNG NHÓM  nhãn↔ô 8 · ô↔ô 16 · nhóm↔nhóm 32 · khoảng trong nhóm ≤ ½ khoảng giữa nhóm
BỀ RỘNG TỐI ĐA   văn bản đọc 640–720 (65–75 ký tự) · form 480–640 · dashboard 1440 · bảng không giới hạn
CHỮ           12/16 · 14/20 app · 16/24 đọc · 18–20/28 · 24/32 · 30–32/40
              400 + 500–600 · 1 sans + 1 mono · số trong bảng dùng tabular · không căn giữa quá 2 dòng
ĐIỀU KHIỂN    dày 28–32 · mặc định 36–40 · thoáng 44–48 · cùng hàng thì cùng cao
VÙNG BẤM      sàn 24 · desktop nên 32 · mobile 44pt (iOS) / 48dp (Android) · hai vùng cách nhau ≥ 8
BO GÓC        0–2 bảng, dòng · 4–6 nút, ô, tag · 8–12 thẻ, modal · 999 pill, avatar · trong = ngoài − padding
TÁCH BỀ MẶT   khoảng trắng → đường kẻ mảnh → nền nhạt khác màu → bóng (bóng chỉ cho thứ nổi thật)
CHUYỂN ĐỘNG   100–150 · 150–200 · 200–300 · 300–400ms · vào ease-out · ra ease-in
              không biết chọn: 200ms ease-out · không nảy, không đàn hồi
CHỮ LỚN       (ngưỡng contrast 3:1) 24px, hoặc 18,66px khi weight ≥ 700 — còn lại cần 4,5:1
```

## 4. Từ quy tắc sang thuộc tính Figma

| Quy tắc | Thuộc tính / API | Đọc thêm |
|---|---|---|
| Khoảng giữa phần tử | `itemSpacing` · `counterAxisSpacing` (khi wrap) · `gridRowGap` / `gridColumnGap` | [spacing-layout.md](reference/spacing-layout.md) |
| Khoảng đệm | `paddingTop` / `Right` / `Bottom` / `Left` | |
| Thang khoảng cách, bo góc | FLOAT variable + `setBoundVariable('itemSpacing' \| 'paddingLeft' \| 'cornerRadius', v)` | |
| Hug / Fill / Fixed | `layoutSizingHorizontal` / `layoutSizingVertical` | |
| Bề rộng đọc, form | `maxWidth` trên con của auto layout | |
| Cột trang | `layoutGrids` COLUMNS 12 / 8 / 4 · `createGridStyle()` | |
| Thang chữ | `createTextStyle()` · `lineHeight: { unit: 'PIXELS' }` · `setTextStyleIdAsync` | [typography.md](reference/typography.md) |
| Đoạn văn trong cột | `layoutSizingHorizontal = 'FILL'` + `textAutoResize = 'HEIGHT'` | |
| Nhãn viết hoa | `textCase = 'UPPER'` + `letterSpacing: { unit: 'PERCENT', value: 6 }` | |
| Cùng hàng cùng cao | chiều cao cố định theo biến thể Size + `counterAxisAlignItems = 'CENTER'` | [sizing-surfaces.md](reference/sizing-surfaces.md) |
| Vùng bấm | frame bọc icon 40 desktop (tối thiểu 32) · 44 iOS, web mobile · 48 Android, icon nằm giữa | |
| Độ nổi | `createEffectStyle()` theo mức · `setEffectStyleIdAsync` | |
| Mật độ | collection có mode Compact / Default / Comfortable | |
| Khung nền tảng | cỡ frame + khung giữ chỗ cho vùng an toàn | [platforms.md](reference/platforms.md) |
| Chuyển động | `setReactionsAsync` · `SMART_ANIMATE` / `DISSOLVE` / `MOVE_IN` · `EASE_OUT` | [motion.md](reference/motion.md) |

## 5. Khung dựng mẫu — một nhóm form

Nhóm lồng nhau chính là quy tắc "gần = cùng nhóm": field (gap 8) → nhóm (gap 16) → form (gap 32).

```js
const R = { family: 'Inter', style: 'Regular' }, M = { family: 'Inter', style: 'Medium' }
await Promise.all([figma.loadFontAsync(R), figma.loadFontAsync(M)])
const stack = (name = '', gap = 0) => {
  const f = figma.createFrame()
  f.name = name; f.layoutMode = 'VERTICAL'; f.itemSpacing = gap; f.fills = []
  f.layoutSizingHorizontal = 'HUG'; f.layoutSizingVertical = 'HUG'
  return f
}
const form = stack('Form', 32)              // nhóm ↔ nhóm
form.resize(560, 100)                       // form 480–640: resize trước...
form.layoutSizingHorizontal = 'FIXED'       // ...rồi đặt rõ cả hai chiều
form.layoutSizingVertical = 'HUG'
const group = stack('Nhóm · Liên hệ', 16)   // ô ↔ ô
form.appendChild(group); group.layoutSizingHorizontal = 'FILL'
for (const name of ['Họ và tên', 'Số điện thoại']) {
  const field = stack(`Field · ${name}`, 8) // nhãn ↔ ô
  group.appendChild(field); field.layoutSizingHorizontal = 'FILL'
  const label = figma.createText()
  label.fontName = M; label.fontSize = 14; label.characters = name
  label.lineHeight = { unit: 'PIXELS', value: 20 }
  const input = figma.createFrame()
  input.name = 'Input'; input.layoutMode = 'HORIZONTAL'; input.counterAxisAlignItems = 'CENTER'
  input.paddingLeft = input.paddingRight = 12; input.cornerRadius = 6
  input.fills = [{ type: 'SOLID', color: { r: 1, g: 1, b: 1 } }]
  input.strokes = [{ type: 'SOLID', color: { r: 0.39, g: 0.45, b: 0.55 } }] // viền ô nhập ≥ 3:1
  input.resize(200, 40)                     // cao 40 cố định, không dùng padding dọc lẻ nấc
  field.appendChild(label); field.appendChild(input)
  input.layoutSizingHorizontal = 'FILL'; input.layoutSizingVertical = 'FIXED'
}
figma.currentPage.selection = [form]
figma.viewport.scrollAndZoomIntoView([form])
return { formId: form.id }
```

Trong file thật: thay số và màu bằng variables/styles của file, đặt form vào chỗ trống trên page.

## 6. Bẫy Plugin API hay gặp khi làm bố cục và chữ

- `'FILL'` chỉ đặt được **sau** `appendChild` vào cha có auto layout; `'HUG'` chỉ cho frame auto layout và text.
- Gọi `resize()` trước, rồi đặt rõ `layoutSizingHorizontal`/`Vertical` cho cả hai chiều — đừng đoán resize để lại chế độ nào.
- Load font trước khi đổi `fontName`, `characters`, `fontSize`, `lineHeight`, `letterSpacing`, `textAutoResize`, `textCase`.
- `lineHeight` và `letterSpacing` kiểu `PERCENT` tính theo **cỡ chữ**: 150 = 1,5 lần; 6 = 0,06em.
- Style trong file dynamic page: dùng `setTextStyleIdAsync`, `setEffectStyleIdAsync`, `setGridStyleIdAsync`, không gán `textStyleId` trực tiếp.
- `fills`, `strokes`, `effects`, `layoutGrids` là mảng chỉ đọc: gán mảng mới, không sửa phần tử.
- Text nhiều style trả `figma.mixed` — đọc bằng `getStyledTextSegments([...])`.
- `openTypeFeatures` chỉ đọc: **không bật được số tabular bằng code** ([typography.md §7](reference/typography.md#7-số-trong-bảng-và-dữ-liệu)).
- Transition prototype tính `duration` bằng **giây** (0.2); `AFTER_TIMEOUT` và `delay` tính bằng **mili giây**.

## 7. Kiểm trước khi báo xong

**Máy đo được** — chạy `audit_design` trên frame vừa dựng:

| Rule | Bắt gì | Sửa |
|---|---|---|
| `lech-nac-khoang-cach` | gap/padding không chia hết cho 4 | Về nấc gần nhất, hoặc gắn spacing variable |
| `vung-bam-nho` · `vung-bam-mobile` | nút, link, icon dưới 24×24; dưới 44 trong frame ≤ 480px | Frame bọc vùng bấm ([sizing-surfaces.md §2](reference/sizing-surfaces.md#2-vùng-bấm)) |
| `tran-chu` | chữ bị frame cha cắt mất | `textAutoResize = 'HEIGHT'`, bỏ chiều cao cố định, hoặc cắt có chủ đích bằng `maxLines` |
| `chu-khong-style` | text không gắn text style dù file có | `setTextStyleIdAsync` với style đúng vai trò |
| `tuong-phan-chu` | chữ dưới 4,5:1 (3:1 cho chữ lớn) | Đổi màu theo token của file; đừng chỉ phóng to chữ cho qua ngưỡng |
| `ranh-gioi-o-nhap` | viền hoặc nền ô nhập dưới 3:1 so với nền | `strokes` bằng token viền ô nhập của file ([sizing-surfaces.md §4](reference/sizing-surfaces.md#4-tách-bề-mặt)) |

**Máy không đo được** — tự nhìn bằng `screenshot`:

- [ ] Nheo mắt: chụp ở `scale` 0,25 — mắt rơi vào đâu trước? nút nào là việc chính? các nhóm có tách nhau?
- [ ] Khoảng trong nhóm nhỏ hơn khoảng giữa nhóm, tỉ lệ ≥ 1:2?
- [ ] Nút, ô nhập, select cùng hàng cùng chiều cao? Mọi thứ thẳng hàng theo một vài mép chung?
- [ ] Đoạn văn không rộng quá 75 ký tự; không đoạn nào hơn 2 dòng bị căn giữa?
- [ ] Số trong bảng căn phải, cột số không nhấp nhô?
- [ ] Chữ dài, dấu chồng, 0 dòng và rất nhiều dòng — bố cục vẫn đứng?

## 8. Đọc sâu

- [reference/typography.md](reference/typography.md) — thang chữ, độ đậm, độ dài dòng, chữ trên nền tối, số tabular, cơ chế text và text style trong Figma.
- [reference/spacing-layout.md](reference/spacing-layout.md) — thang 4, gần = cùng nhóm, nhịp, Gestalt, auto layout, grid auto layout, constraints, layout grid, spacing variables.
- [reference/sizing-surfaces.md](reference/sizing-surfaces.md) — chiều cao điều khiển, vùng bấm, bo góc, tách bề mặt, thang độ nổi, mật độ, icon, ảnh.
- [reference/platforms.md](reference/platforms.md) — pt · dp · px, khung web, iOS, Android, vùng an toàn, thanh hệ thống.
- [reference/motion.md](reference/motion.md) — thời lượng, easing, ánh xạ sang prototype Figma.
