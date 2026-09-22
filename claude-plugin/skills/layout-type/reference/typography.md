# Chữ — thang, text style và cơ chế text của Figma

Trên màn hình app, hầu hết những gì người dùng cần biết đều nằm trong chữ. Việc của thang chữ là
giúp họ **đọc nhanh, biết cái gì quan trọng hơn, và gặp cùng một vai trò thì thấy cùng một kiểu
chữ**. Chữ "có cá tính" mà đọc chậm là đổi nhầm.

Mọi số dưới đây là mặc định. File đã có text styles → dùng styles đó; dự án có `DESIGN.md` → số
trong đó thắng.

---

## 1. Thang chữ mặc định

| Cỡ / line height | Dùng cho | Tên style gợi ý |
|---|---|---|
| 12 / 16 | Nhãn phụ, chú thích, badge | `Caption` |
| **14 / 20** | **Mặc định của app**: bảng, form, nav, nút | `Body/Default` |
| 16 / 24 | Văn bản đọc, mô tả dài — **mặc định của trang đọc, brand** | `Body/Reading` |
| 18–20 / 28 | Tiêu đề thẻ, tiêu đề modal | `Title/Section` |
| 24 / 32 | Tiêu đề trang | `Title/Page` |
| 30–32 / 40 | Số liệu lớn trong dashboard | `Display/Metric` |

- **14px cho app** vì app cần mật độ và người dùng ngồi gần màn hình mỗi ngày. Người lớn tuổi,
  mobile, màn hình xa → 16px.
- Chữ nhỏ cần line height tương đối lớn (1,4–1,5 lần cỡ chữ), chữ lớn cần nhỏ hơn (1,2–1,3).
- Mọi line height trong bảng đều là bội số của 4 → dòng chữ khớp luôn lưới khoảng cách 4. Giữ tính
  chất này khi thêm nấc.
- Mỗi vai trò đúng **một** style. Cùng là "tiêu đề thẻ" mà chỗ 18 chỗ 20 là lỗi, không phải biến tấu.

## 2. Thêm nấc bằng tỉ lệ

Cần nấc ngoài bảng (landing, hero, trang tài liệu) → nhân cỡ chữ thân với một tỉ lệ cố định rồi
làm tròn về số chẵn gần nhất:

| Register | Tỉ lệ | Ví dụ từ chữ thân | Làm tròn |
|---|---|---|---|
| Product | 1,125–1,2 | 14 → 16,8 → 20,2 → 24,2 → 29 | 14 · 16 · 20 · 24 · 30 |
| Brand | 1,25–1,5 | 16 → 20 → 25 → 31,3 → 39 | 16 · 20 · 24 · 32 · 40 |

- Product có nhiều nấc sát nhau vì giao diện dày; brand ít nấc hơn nhưng mỗi bước nhảy rõ.
- Hai cỡ chỉ cách nhau 1px (13 cạnh 14, 15 cạnh 16) không tạo được thứ bậc: người xem thấy hai chữ
  lệch nhau chứ không thấy cái nào quan trọng hơn. Muốn khác thì nhảy hẳn một nấc. Thang sáu nấc ở
  §1 đã đủ cho phần lớn sản phẩm; thêm nấc thứ bảy phải có vai trò cụ thể.

## 3. Độ đậm và họ font

- **Hai độ đậm:** 400 cho chữ thường, 500–600 cho tiêu đề, nhãn, nút. 700 trở lên để dành cho tiêu
  đề lớn ở trang brand. Ở cỡ 12–14, 500 đứng cạnh 400 khó nhận ra — nhãn cần nổi thì lấy 600.
- **Tối đa hai họ font:** một sans + một mono (cho mã, số liệu kỹ thuật). Thứ bậc trong app đến từ
  cỡ và độ đậm của **một** họ sans là đủ. Trang brand muốn thêm họ thứ hai (serif cho tiêu đề chẳng
  hạn) thì hai họ phải khác hẳn nhau về cấu trúc; hai sans gần giống nhau chỉ trông như lỗi.
- **Font là quyết định của sản phẩm**, ghi trong `DESIGN.md` hoặc text styles của file. Chưa có →
  hỏi người dùng hoặc dùng font họ đang dùng ở các file khác, đừng tự đổi font vì thấy "nhạt".
- **Tiếng Việt là điều kiện bắt buộc.** Font phải đủ dấu: ă â đ ê ô ơ ư và dấu chồng (ẩ, ặ, ỗ). Nhiều
  font display thiếu glyph tiếng Việt — Figma thay bằng font khác, chữ lệch nét. Thử bằng chuỗi
  "ĐẶNG THỊ NGỌC ẨN · Nguyễn Thị Hường" trước khi chốt.
- Tên style khác nhau giữa các họ font (`Semi Bold` hay `SemiBold`). Lấy đúng tên từ
  `figma.listAvailableFontsAsync()`, đừng đoán.

## 4. Thứ bậc bằng tương phản

- Thứ bậc chắc nhất khi **cỡ, độ đậm và khoảng trắng cùng nói một điều**. Ví dụ tiêu đề nhóm:
  một nấc thang to hơn chữ thân, 600 thay vì 400, khoảng phía trên (32) gấp đôi khoảng phía dưới
  (16) để nó dính vào nội dung nó mở đầu. Làm đủ ba việc đó thì không cần tô màu tiêu đề.
- Chênh 1–2px không tạo thứ bậc; muốn khác thì nhảy ít nhất một nấc thang.
- Chữ phụ khác chữ chính bằng **màu chữ phụ** (token text-secondary của file), không bằng thêm một
  cỡ chữ lạ.
- **Không dùng màu thay cho thứ bậc.** Tô màu tiêu đề trong giao diện product làm mắt tưởng chỗ đó
  bấm được.

## 5. Căn lề, độ dài dòng, viết hoa

- **Căn trái** cho đoạn văn. Không căn giữa đoạn quá hai dòng. Không `JUSTIFIED` — tạo khoảng
  trắng loang giữa các từ.
- **Độ dài dòng 65–75 ký tự** → bề rộng hộp chữ khoảng **40–45 lần cỡ chữ**: 16px ≈ 640–720px,
  14px ≈ 560–630px. Trong Figma: text `FILL` trong cột có `maxWidth`, hoặc đặt `maxWidth` thẳng
  trên text khi nó là con của auto layout.
- **Không viết hoa toàn bộ quá hai từ.** Nhãn ngắn viết hoa (eyebrow, tiêu đề cột nhỏ) thì dùng
  `textCase = 'UPPER'` — nội dung gốc giữ chữ thường, sửa lại không phải gõ lại. Khoảng chữ mặc
  định của font được canh cho chữ thường; dòng toàn chữ in hoa trông dính lại, nên mở thêm
  `letterSpacing` khoảng 5–10%.
- Tiêu đề nhiều dòng: `textWrapStyle = 'BALANCE'` để các dòng dài gần bằng nhau, không còn một
  chữ mồ côi ở dòng cuối.
- Chữ rất lớn (từ 32px) có thể siết `letterSpacing` âm nhẹ, −1% đến −2%. Chữ thân để 0.

## 6. Chữ sáng trên nền tối

Cùng một text style, đặt chữ sáng lên nền tối sẽ trông nhẹ nét hơn và các dòng như khít lại. Style
cho dark mode nên chỉnh cả ba thông số, không chỉ một:

| Bù | Mức | Trong Figma |
|---|---|---|
| Line height | Rộng thêm 5–10 điểm phần trăm (150% → 155–160%) | `lineHeight` |
| Khoảng chữ | Thêm 1–2% | `letterSpacing: { unit: 'PERCENT', value: 1 }` |
| Độ đậm chữ thân | Tăng một bậc nếu vẫn thấy mảnh (400 → 500) | style riêng cho dark, hoặc variable `fontWeight` theo mode |

Chữ sáng không dùng `#FFFFFF` thuần — `audit_design` báo `chu-trang-thuan`.

## 7. Số trong bảng và dữ liệu

- **Cột số dùng số tabular** (mọi chữ số rộng bằng nhau) — không thì cột nhấp nhô, mắt so hàng khó.
  Font mono đã đều bề rộng sẵn.
- **Plugin API không bật được tabular:** `openTypeFeatures` chỉ đọc. Cách làm:
  1. Tạo (hoặc tìm) text style `Data/Number`.
  2. Nhờ người dùng bật tabular figures cho style đó trong phần Type settings của Figma — một lần.
  3. Gắn style đó cho mọi ô số bằng `setTextStyleIdAsync`.
  4. Kiểm: `node.openTypeFeatures.TNUM === true`.
- Số căn phải (`textAlignHorizontal = 'RIGHT'`), chữ căn trái, tiêu đề cột căn theo nội dung cột.
- Số liệu lớn trên dashboard (30–32/40) vẫn cần nhãn đi kèm ở cỡ 12–14.

## 8. Cơ chế text trong Figma

| Thuộc tính | Giá trị | Dùng khi |
|---|---|---|
| `fontName` | `{ family, style }` | Load bằng `figma.loadFontAsync` trước mọi thay đổi |
| `lineHeight` | `{ unit: 'PIXELS', value: 20 }` | **Mặc định cho style** — khớp lưới 4, không phụ thuộc font |
| | `{ unit: 'PERCENT', value: 150 }` | Tính theo cỡ chữ; tiện khi một style dùng nhiều cỡ |
| | `{ unit: 'AUTO' }` | Tránh trong hệ thống — mỗi font một khác |
| `letterSpacing` | `{ unit: 'PERCENT', value: 6 }` | 6 = 0,06em. Chữ thân 0 |
| `textAutoResize` | `'WIDTH_AND_HEIGHT'` | Nhãn, chữ trong nút — co theo nội dung, không xuống dòng |
| | `'HEIGHT'` | Đoạn văn — bề rộng do cột quyết (`FILL`), chiều cao tự giãn |
| | `'NONE'` | Hộp cố định — dễ tràn, chỉ dùng khi có lý do |
| `textTruncation` + `maxLines` | `'ENDING'`, `1` | Ô bảng, tiêu đề một dòng cắt bằng `…` thay vì làm vỡ chiều cao |
| `paragraphSpacing` | 8–16 | Khoảng giữa các đoạn trong một text node. Đã có `paragraphSpacing` thì để `paragraphIndent = 0` — hai cách tách đoạn chồng lên nhau là thừa |
| `textCase` | `'UPPER'` · `'TITLE'` · `'SMALL_CAPS'` | Đổi hiển thị, giữ nội dung gốc |
| `textAlignHorizontal` | `'LEFT'` · `'RIGHT'` · `'CENTER'` | Số trong bảng `RIGHT` |
| `textWrapStyle` | `'BALANCE'` | Tiêu đề nhiều dòng |

- **Chữ trong auto layout:** nhãn → `layoutSizingHorizontal = 'HUG'`; đoạn văn → `'FILL'` rồi
  `textAutoResize = 'HEIGHT'`. Hộp chữ cố định chiều cao trong frame `clipsContent` là nguồn chính
  của lỗi `tran-chu`.
- **Dấu chồng bị cắt:** line height quá chật (dưới ~1,2 ở chữ nhỏ) cộng frame cắt nội dung làm mất
  phần trên của "Ẩ", "Ỗ". Luôn thử với tên in hoa có dấu.

## 9. Tạo và dùng text style

Tìm style có sẵn trước; chỉ tạo phần thiếu. Ví dụ bộ style cho register product:

```js
const scale = [
  { name: 'Caption', size: 12, lh: 16, style: 'Regular' },
  { name: 'Body/Default', size: 14, lh: 20, style: 'Regular' },
  { name: 'Body/Strong', size: 14, lh: 20, style: 'Semi Bold' },
  { name: 'Body/Reading', size: 16, lh: 24, style: 'Regular' },
  { name: 'Title/Section', size: 18, lh: 28, style: 'Semi Bold' },
  { name: 'Title/Page', size: 24, lh: 32, style: 'Semi Bold' },
  { name: 'Display/Metric', size: 32, lh: 40, style: 'Semi Bold' },
]
const existing = await figma.getLocalTextStylesAsync()
const made = []
for (const { name, size, lh, style } of scale) {
  if (existing.some((s) => s.name === name)) continue
  const fontName = { family: 'Inter', style }
  await figma.loadFontAsync(fontName)
  const s = figma.createTextStyle()
  s.name = name
  s.fontName = fontName
  s.fontSize = size
  s.lineHeight = { unit: 'PIXELS', value: lh }
  s.letterSpacing = { unit: 'PERCENT', value: 0 }
  made.push({ id: s.id, name })
}
return made
```

Gắn style cho text: `await text.setTextStyleIdAsync(styleId)`. File dùng variable cho cỡ chữ →
gắn luôn vào style: `s.setBoundVariable('fontSize', sizeVariable)` (tương tự `lineHeight`,
`letterSpacing`, `fontWeight`), để đổi thang ở một chỗ.

## 10. Đọc thuộc tính chữ mà inspect_nodes không trả

`inspect_nodes` không trả lineHeight, letterSpacing, fontWeight, style id, variable đã gắn. Đọc
bằng code, chỉ đọc, gom theo tổ hợp để kết quả gọn:

```js
const root = await figma.getNodeByIdAsync('FRAME_ID')
if (!root || !('findAllWithCriteria' in root)) throw new Error('Cần id của một frame')
const groups = new Map()
for (const t of root.findAllWithCriteria({ types: ['TEXT'] })) {
  for (const seg of t.getStyledTextSegments(['fontSize', 'fontName', 'fontWeight', 'lineHeight', 'letterSpacing', 'textStyleId'])) {
    const lh = seg.lineHeight.unit === 'AUTO' ? 'AUTO' : `${seg.lineHeight.value}${seg.lineHeight.unit === 'PIXELS' ? 'px' : '%'}`
    const key = `${seg.fontName.family} ${seg.fontWeight} ${seg.fontSize}/${lh} ls ${seg.letterSpacing.value}${seg.letterSpacing.unit === 'PERCENT' ? '%' : 'px'}`
    if (!groups.has(key)) groups.set(key, { count: 0, withStyle: 0, sample: t.id })
    const g = groups.get(key)
    g.count++
    if (seg.textStyleId) g.withStyle++
  }
}
return Object.fromEntries(groups)
```

Đọc kết quả: nhiều tổ hợp chỉ xuất hiện 1–2 lần → thang đang bị phá lẻ tẻ; `withStyle` nhỏ hơn
`count` → chữ chưa gắn style (khớp với rule `chu-khong-style`).
