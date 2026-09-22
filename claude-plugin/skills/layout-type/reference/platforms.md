# Nền tảng — đơn vị, khung web, iOS, Android

Phần này là quy ước nền tảng (Apple HIG, Material, WCAG), không phải gu. Số liệu thiết bị thay đổi
theo từng đời máy: khi người dùng nêu máy cụ thể, dùng đúng máy đó; file có sẵn khung mẫu hay UI
kit của nền tảng → dùng của file.

---

## 1. Đơn vị: px · pt · dp

**Thiết kế ở 1x:** 1 đơn vị Figma = 1 CSS px (web) = 1 pt (iOS) = 1 dp (Android). Khung là kích
thước **logic** (393×852), không phải điểm ảnh vật lý (1179×2556).

| Nền tảng | Đơn vị | Chữ | Xuất tài nguyên |
|---|---|---|---|
| Web | CSS px | px (trình duyệt cho phóng to) | 1x, 2x cho màn mật độ cao |
| iOS | pt | pt, co giãn theo Dynamic Type | @2x, @3x |
| Android | dp | **sp** — co giãn theo cỡ chữ người dùng chọn | mdpi 1x · hdpi 1,5x · xhdpi 2x · xxhdpi 3x · xxxhdpi 4x |

- Android: `px = dp × dpi / 160`.
- Người dùng phóng to chữ trên cả ba nền tảng (WCAG 1.4.4: tới 200% không mất nội dung). Thiết kế
  cho chữ **giãn được**: text `textAutoResize = 'HEIGHT'`, không đặt chiều cao cố định cho khối
  chứa chữ, thử lại màn hình chính với chữ to hơn khoảng 1,3 và 2 lần.

## 2. Web

| Khung | Đại diện | Ghi chú |
|---|---|---|
| **1440** | Desktop phổ biến | Khung chính cho app nghiệp vụ và landing |
| 1280 | Laptop | Kiểm bảng nhiều cột, sidebar |
| 1024 | Tablet ngang, laptop nhỏ | Danh sách + chi tiết dưới mức này tách hai màn |
| 768 | Tablet dọc | Sidebar thu gọn hoặc chuyển thành drawer |
| **390** | Mobile web | Vùng bấm 44, chữ thân 16 |

- Dựng khung chính trước (1440 hoặc 1280), rồi kiểm 1024 và 390 — các khung này là **điểm ngắt**
  giao cho dev. Cột và lề cho từng khung:
  [spacing-layout.md §9](spacing-layout.md#9-layout-grid--cột).
- Chiều cao khung web không quan trọng bằng bề ngang; dùng 900–1024 để xem "màn hình đầu tiên",
  nội dung dài thì khung dài.
- Mobile web: thanh địa chỉ và thanh công cụ của trình duyệt chiếm một phần chiều cao — đừng đặt
  nút quan trọng sát đáy khung.
- Màn cảm ứng **không có hover**: trạng thái pressed phải khác default rõ (lệch ≥ 2 nấc màu), và
  không giấu hành động sau hover.

## 3. iOS

| Khung (pt) | Máy | Vùng an toàn dọc (trên / dưới) |
|---|---|---|
| 375×667 | iPhone SE (đời 2, 3) — dùng để thử màn nhỏ | 20 / 0 |
| 390×844 | iPhone 12, 13, 14 | 47 / 34 |
| **393×852** | iPhone 14 Pro, 15, 15 Pro, 16 | 59 / 34 |
| 402×874 | iPhone 16 Pro | 62 / 34 |
| 430×932 | iPhone 14 Pro Max, 15 Plus, 15 Pro Max, 16 Plus | 59 / 34 |
| 440×956 | iPhone 16 Pro Max | 62 / 34 |

- **Vùng an toàn**: không đặt nội dung bấm được vào dải trên (status bar, Dynamic Island hoặc tai
  thỏ) và dải dưới 34pt (thanh home indicator). Nền và ảnh được tràn ra đó.
- Thanh hệ thống quen thuộc: navigation bar 44pt (tiêu đề lớn cao hơn), tab bar 49pt **cộng** 34pt
  home indicator phía dưới.
- **Vùng bấm 44×44pt.** Lề ngang 16pt, 20pt trên máy lớn.
- Chữ hệ thống: Body 17pt, Headline 17pt semibold, Subheadline 15, Footnote 13, Caption 12,
  Large Title 34. Không dưới 11pt.
- Font hệ thống là SF Pro. Kiểm bằng `figma.listAvailableFontsAsync()`; máy người dùng không có thì
  hỏi, hoặc dùng tạm Inter và ghi rõ trong tên frame.

## 4. Android

| Khung (dp) | Đại diện | Lớp cửa sổ |
|---|---|---|
| **360×800** | Máy phổ thông, màn nhỏ | Compact |
| 412×915 | Máy màn lớn (dòng Pixel, flagship) | Compact |

**Lớp cửa sổ theo bề ngang (Material):** Compact < 600dp · Medium 600–839 · Expanded 840–1199 ·
Large 1200–1599 · Extra-large ≥ 1600. Lớp phụ thuộc **cửa sổ**, không phụ thuộc loại máy (máy gập,
chia đôi màn hình).

- Status bar khoảng 24dp, cao hơn trên máy có camera đục lỗ. Thanh điều hướng ba nút 48dp; điều
  hướng bằng cử chỉ chỉ chừa một dải mỏng ở đáy.
- Top app bar 64dp, navigation bar dưới 80dp (Material 3).
- **Vùng bấm 48×48dp**, hai vùng cách nhau ≥ 8dp. Lề ngang 16dp ở Compact, 24dp từ Medium.
- Thang chữ Material 3 khớp thang mặc định của skill: Body Large 16/24 · Body Medium 14/20 ·
  Body Small 12/16 · Title Large 22/28 · Headline Small 24/32. Font hệ thống là Roboto.

## 5. Dựng khung mobile

Vùng hệ thống là **khung giữ chỗ có tên** (hoặc instance status bar từ UI kit của file), không phải
padding — số 59, 47, 34 không nằm trên thang 4, và khung có tên nói rõ vì sao có khoảng đó.

```js
const spec = { name: 'iPhone 15 · 393×852', w: 393, h: 852, top: 59, bottom: 34 }
const nodes = figma.currentPage.children
const x = nodes.length ? Math.max(...nodes.map((n) => n.x + n.width)) + 120 : 0
const frame = (name = '') => {
  const f = figma.createFrame()
  f.name = name
  f.fills = []
  return f
}
const screen = frame(spec.name)
screen.layoutMode = 'VERTICAL'
screen.resize(spec.w, spec.h)
screen.layoutSizingHorizontal = 'FIXED'
screen.layoutSizingVertical = 'FIXED'
screen.fills = [{ type: 'SOLID', color: { r: 0.98, g: 0.98, b: 0.99 } }]
screen.clipsContent = true
screen.x = x
screen.y = 0
const top = frame('System · Status bar')
top.resize(spec.w, spec.top)
const content = frame('Content')
content.layoutMode = 'VERTICAL'
content.itemSpacing = 16
content.paddingLeft = content.paddingRight = 16
content.paddingTop = 8
const bottom = frame('System · Home indicator')
bottom.resize(spec.w, spec.bottom)
for (const n of [top, content, bottom]) screen.appendChild(n)
for (const n of [top, content, bottom]) n.layoutSizingHorizontal = 'FILL'
content.layoutSizingVertical = 'FILL' // nội dung chiếm phần còn lại
figma.currentPage.selection = [screen]
figma.viewport.scrollAndZoomIntoView([screen])
return { screen: screen.id, content: content.id }
```

Android 412×915: `top` khoảng 24–32, `bottom` 48 khi dùng thanh ba nút. Web mobile 390×844: bỏ
hai khung hệ thống.

## 6. Bảng tra nhanh

| | Web desktop | Web mobile | iOS | Android |
|---|---|---|---|---|
| Khung chính | 1440 / 1280 | 390 | 393×852 | 360×800 · 412×915 |
| Vùng bấm | sàn 24, nên 32 | 44 | 44pt | 48dp |
| Chữ thân | 14 (app) · 16 (đọc) | 16 | 17pt | 14–16sp |
| Lề ngang | theo layout grid | 16 | 16–20pt | 16dp · 24dp |
| Hover | Có | Không | Không | Không |
