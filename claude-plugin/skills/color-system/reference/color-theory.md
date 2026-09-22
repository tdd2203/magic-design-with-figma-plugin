# Lý thuyết màu cho giao diện

Tầng tri thức: đây là cách nghĩ để chọn màu đúng, không phải luật. Hệ màu có sẵn trong file và
DESIGN.md của dự án được ưu tiên hơn các mặc định ở đây (xem SKILL.md).

## 1. Giao diện khác poster ở ba chỗ

1. **Người dùng ngồi trong đó hàng giờ, không chỉ nhìn 2 giây.** Vì vậy khoảng 90% diện tích là
   neutral (nền, thẻ, viền, chữ), 7% primary (nút, link, tab đang chọn), 3% semantic và accent. Hệ quả:
   **chỗ nào có màu là chỗ bấm được.** Tô màu một vùng không tương tác là nói dối mắt người dùng.
2. **Bảng màu là một hệ, không phải ba mã hex.** Mỗi màu cần nhiều nấc cho nền, viền, hover, chữ,
   disabled, focus. Cộng lại khoảng 40–60 giá trị. Trong Figma, đó là 60–70 biến Primitives và khoảng
   25 biến Semantic.
3. **Màu phải sống được ở nhiều môi trường:** màn OLED, màn văn phòng rẻ, điện thoại ngoài nắng, dark
   mode, chế độ tương phản cao, người mù màu. Nên bảng màu phải **đo**, không chỉ **cảm**.

## 2. Ba thuộc tính màu, và vì sao dựng bằng OKLCH

- **Hue (sắc):** màu gì. Quyết định cảm xúc và ý nghĩa.
- **Chroma (độ bão hoà):** tinh khiết hay pha xám. Quyết định sang hay loè loẹt.
- **Lightness (độ sáng):** quyết định **thứ bậc** và **khả năng đọc**. Chỉ được chỉnh một thứ thì chỉnh cái này.

HSL có một khuyết tật: L của nó không khớp độ sáng mắt thấy. `hsl(60 100% 50%)` là vàng chói còn
`hsl(240 100% 50%)` là xanh tối, dù cùng L = 50%. Kéo L đều trong HSL nên ra một ramp nhảy không đều.
**OKLCH** đồng nhất về cảm nhận: cùng L thì sáng ngang nhau thật.

**Dựng bằng OKLCH, bàn giao RGB 0–1.** Figma lưu màu dạng `{ r, g, b }` trong khoảng 0–1 (sRGB).
`figma.util.rgb()` đọc được hex, `rgb()`, `hsl()`, `lab()` nhưng **không đọc `oklch()`**, nên cần
helper ở §7 để đổi qua lại.

Mọi số hue trong skill là **hue OKLCH**, lệch khá xa hue HSL: `#2563EB` có hue HSL 221° nhưng hue
OKLCH 263°. Đừng lấy số hue từ color picker HSL rồi so với các khoảng hue ở đây.

## 3. Chọn sơ đồ phối màu

| Tình huống | Sơ đồ | Lưu ý |
|---|---|---|
| Chưa biết bắt đầu từ đâu | Đơn sắc + 1 accent bổ túc | Rủi ro thấp nhất |
| Cần CTA nổi tối đa | Bổ túc (hai hue đối diện) | Một màu chiếm 85–95%. Chia 50–50 thì rung mắt |
| Cần tương phản nhưng dịu | Bổ túc chia đôi | Lựa chọn an toàn nhất khi cần nổi mà không gắt |
| Nội dung dài, đọc lâu | Đơn sắc, hoặc tương đồng (2–3 hue cách nhau 30–60°) | Tương đồng khó làm CTA nổi |
| Nhiều nhóm dữ liệu ngang hàng | Bộ ba, hoặc bảng màu phân loại riêng cho biểu đồ | Khó kiểm soát; bộ bốn gần như luôn là quá nhiều |
| Thương hiệu có sẵn 1 màu | Đơn sắc từ màu đó + neutral ám hue đó | Trường hợp phổ biến nhất |
| Dashboard phức tạp | Neutral thống trị + 1 primary + semantic | Màu chỉ ở cột trạng thái, không tô cả bảng |

Màu cho biểu đồ là một bảng riêng (phân loại, tuần tự, phân kỳ), không lấy luôn semantic. Đỏ trong
biểu đồ sẽ bị đọc thành "lỗi".

## 4. Ba nhóm màu

### Neutral (~90%): 10–12 nấc

Đây là xương sống: nền trang, nền thẻ, viền, toàn bộ chữ. **Không để xám thuần.** Ám chroma khoảng
0,005–0,015 về **hue của primary** thì cả giao diện dính lại với nhau. Đừng ám "ấm cho thân thiện" hay
"lạnh cho kỹ thuật" theo phản xạ. Nền kem hay cát chỉ đúng khi chính hue thương hiệu là màu ấm (ví dụ
F&B tông nâu đất). Primary teal mà nền ám vàng là tự mâu thuẫn.

### Primary (~7%): 9–11 nấc

Đây là màu của hành động, với ba điều kiện khác hẳn khi làm poster:
- Chữ trắng trên nó đạt ≥ 4,5:1 (hoặc chữ tối, nếu cố ý chọn nút sáng).
- Có một nấc đủ sáng để thay thế trong dark mode.
- Lặp 20 lần trên một màn hình mà không mệt.

Màu sáng rực (vàng, mint, cam nhạt) là **primary tồi** dù đẹp trên poster. Thương hiệu bắt buộc dùng
thì đặt nó làm accent, lấy nấc đậm hơn cùng hue làm primary. **Màu logo dùng cho logo, primary dùng cho
nút**; cùng hue được, khác nấc cũng được.

Màu thương hiệu gần như xám (chroma < 0,03) thì không có hue để dựng ramp. Khi đó hỏi người dùng một
màu hành động, hoặc làm hệ đơn sắc với nút chính là neutral-900.

### Semantic + accent (~3%)

| Vai trò | Hue OKLCH gần đúng | Dùng cho |
|---|---|---|
| Success | lục 140–155 | Lưu xong, đang hoạt động |
| Warning | hổ phách 60–85 | Cần chú ý, sắp hết hạn |
| Danger | đỏ 25–35 | Lỗi, xoá, không hoàn tác được |
| Info | xanh dương 240–265 | Ghi chú, gợi ý |

- Semantic mang nghĩa cố định, **không dùng để trang trí**. Badge "Mới" hay "Miễn phí" không phải
  success.
- Mỗi semantic nên cách primary **≥ 60°** (khoảng cách tính vòng tròn: 350° và 20° cách nhau 30°).
  Không cách được thì phải có **tín hiệu thứ hai** (icon, chữ, hình dạng), và ghi lý do vào
  `description` của biến để người sau hiểu. Các ca hay gặp:
  - *Primary teal hoặc xanh lục:* đẩy success ra 60° thì nó thành vàng-lục, màu gợi cảm giác bệnh. Nên
    giữ success ở lục, dùng màu này chỉ cho trạng thái thật, và luôn kèm icon hoặc chữ.
  - *Primary xanh dương:* info rơi đúng vùng primary. Hoặc cho info dùng chính ramp primary và luôn có
    icon "i", hoặc bỏ màu info riêng, vì info là semantic nhẹ nhất.
  - *Primary đỏ hoặc cam:* danger và warning đều gần. Nên chọn primary khác, hoặc giữ danger và bắt
    buộc có icon, chữ "Lỗi" hay "Xoá" đi kèm.
- **Accent: tối đa một**, dùng cho đúng một việc. Viết được câu "accent này dùng cho X ở Y" thì mới giữ.
- Nền pastel hay nền tint chỉ làm nền. Chữ đặt lên đó lấy nấc 700–900 **của chính hue nền**, không lấy
  neutral: neutral trên nền có màu vừa hụt contrast vừa lệch tông với khối chứa nó.

## 5. Thứ tự sáu bước

1. **Neutral trước, màu thương hiệu sau.** Neutral chiếm 90% nên nó quyết định khí chất. Trong Figma, có
   thể dựng bản nháp với `color/action/*` tạm trỏ về neutral-900. Nếu trông tử tế thì layout đúng hướng;
   nếu phải có màu mới nhìn được thì layout đang sai. Khi ổn, chỉ cần đổi alias sang primary, layer
   không phải sửa gì.
2. **Chọn primary theo chức năng.** Nó có nổi trên nền trang không? Chữ trắng có đạt 4,5 không? Dark
   mode dùng nấc nào? Trượt câu thứ hai thì lấy nấc đậm hơn, đừng cố giữ đúng màu logo.
3. **Dựng ramp** (§6).
4. **Thêm semantic, kiểm xung đột.** Đặt primary và bốn semantic cạnh nhau, không ghi tên, rồi hỏi người
   dùng màu nào là "lỗi". Trên canvas, đó là một hàng 5 ô vuông gắn biến.
5. **Tối đa một accent**, kèm câu mô tả chỗ dùng.
6. **Gán token theo vai trò** ([tokens-and-modes.md](tokens-and-modes.md)). Trong component **không có
   mã hex**, chỉ có biến Semantic.

## 6. Dựng ramp

Nếu giữ nguyên H và C rồi chỉ kéo L, nấc sáng sẽ bạc, nấc tối xỉn như mực, nấc giữa chói. Có ba chỉnh sửa:

- **Chroma hình chuông:** thấp ở hai đầu, cao nhất quanh nấc 500–600.
- **Dịch hue tổng cộng 6–12°** từ đầu sáng sang đầu tối. Nấc sáng nghiêng về phía ấm (vàng, hue
  khoảng 60–90), nấc tối nghiêng về phía lạnh (xanh dương, hue khoảng 260). Dấu `+` hay `−` **không**
  tự mang nghĩa ấm hay lạnh; chiều dịch tuỳ hue gốc:

  | Hue gốc | Nấc sáng | Nấc tối | `shift` trong helper |
  |---|---|---|---|
  | ~110–270 (lục, teal, xanh dương) | giảm hue (về lục-vàng / cyan) | tăng hue (về xanh dương / chàm) | dương, +6…+12 |
  | ~270–360 và 0–110 (tím, hồng, đỏ, cam, vàng) | tăng hue (về hồng / vàng) | giảm hue (về đỏ / xanh) | âm, −6…−12 |

- **Bước L không đều:** nhỏ ở vùng sáng (mắt phân biệt kém), lớn hơn ở vùng tối.

Bảng điểm khởi đầu (OKLCH). Sau đó tinh chỉnh bằng mắt và **đo lại**:

| Nấc | L màu | C màu | L neutral | C neutral | Vai trò điển hình |
|---|---|---|---|---|---|
| 50 | 0,97 | 0,02 | 0,985 | 0,004 | Nền tag, vùng nhấn rất nhẹ · nền trang (neutral) |
| 100 | 0,94 | 0,04 | 0,965 | 0,006 | Hover hàng bảng, alert nhẹ · nền phụ (neutral) |
| 200 | 0,89 | 0,07 | 0,92 | 0,008 | Viền vùng nhấn · viền trang trí (neutral) |
| 300 | 0,82 | 0,11 | 0,87 | 0,010 | Viền input nhạt, disabled · chữ phụ dark (neutral) |
| 400 | 0,72 | 0,15 | 0,71 | 0,012 | Icon phụ · **primary trong dark mode** |
| 500 | 0,64 | 0,18 | 0,55 | 0,014 | Ranh giới · **viền ô nhập, chữ muted** (neutral) |
| 600 | 0,56 | 0,19 | 0,45 | 0,014 | **Nút chính, link**, nếu chữ trắng đạt 4,5 · chữ phụ (neutral) |
| 700 | 0,48 | 0,17 | 0,37 | 0,012 | Hover nút · chữ màu trên nền tint |
| 800 | 0,40 | 0,14 | 0,28 | 0,010 | Active / pressed · bề mặt dark (neutral) |
| 900 | 0,32 | 0,11 | 0,21 | 0,008 | Chữ màu trên nền nhạt cùng hue · **nền trang dark** (neutral) |
| 950 | 0,22 | 0,07 | 0,15 | 0,006 | Chữ đậm nhất · chữ trên nút dark (neutral) |

**Vì sao neutral có cột L riêng.** Neutral phải gánh hai việc mà màu chính không phải gánh. Đầu tối
của nó là nền dark mode (khoảng `#0F172A`–`#18181B`, tức L ≈ 0,21). Còn nấc 500–600 của nó là chữ
muted và chữ phụ, nên phải đủ đậm để đạt 4,5 trên nền trang. Nếu dùng chung cột L màu, neutral-500
(L 0,64) chỉ đạt 3,2 trên nền trang: đem làm chữ là trượt.

**Nấc 600 không phải lúc nào cũng đạt.** Đo chữ trắng trên nấc 600 ở L 0,56, C 0,19:

| Hue | Chữ trắng / 600 | Trên nền subtle (neutral-100) | Hạ L tới khi cả hai ≥ 4,5 |
|---|---|---|---|
| Xanh dương 262 | 4,81 | 4,34 | L 0,55 → 5,02 / 4,54 |
| Teal 185 | 4,42 | 4,00 | L 0,53 → 5,03 / 4,54 |
| Xanh lục 150 | 4,34 | 3,92 | L 0,52 → 5,16 / 4,66 |
| Cam 45 | 4,97 | 4,49 | L 0,55 → 5,18 / 4,68 |
| Tím 290 | 5,03 | 4,54 | Giữ nguyên |

Hàm `fitStep` ở §7 làm việc này tự động: mỗi lần hạ 0,01, tối đa 0,06. Hạ hết 0,06 vẫn trượt thì để
vai trò hành động lùi một nấc (default 700, hover 800, active 900) thay vì ép nấc 600.

**Chroma của thương hiệu.** Cột C màu hợp với màu bão hoà vừa phải. Thương hiệu trầm thì nhân cả cột
với `k = C thương hiệu / 0,19` để màu trầm vẫn trầm. Màu nào vượt gamut sRGB thì helper tự hạ chroma,
giữ nguyên L và H, nên các nấc sáng của hue teal và cyan sẽ có chroma thật thấp hơn bảng. Đó là
đúng, không phải lỗi.

## 7. Helper JS: OKLCH ⇄ RGB và contrast

Chỉ là phép tính thuần, chạy được trong `execute_figma_code` (và ở bất cứ đâu có JS). Mỗi lần gọi
`execute_figma_code` là một phạm vi độc lập, nên phải **dán khối này lên đầu** mọi đoạn code cần nó.
Helper cố ý viết ở cú pháp ES2017 (không `?.`, `??`, object spread) để dán vào đâu cũng chạy; đây là
lựa chọn thận trọng, không phải điều kiện của `execute_figma_code`.

```js
// ── Helper màu: OKLCH ⇄ RGB 0–1 của Figma, contrast WCAG 2 (toán OKLab của Björn Ottosson) ──
const lin = (c) => (c <= 0.04045 ? c / 12.92 : ((c + 0.055) / 1.055) ** 2.4)
const enc = (c) => (c <= 0.0031308 ? 12.92 * c : 1.055 * c ** (1 / 2.4) - 0.055)
function lchToLinear(L, C, H) {
  const a = C * Math.cos((H * Math.PI) / 180), b = C * Math.sin((H * Math.PI) / 180)
  const l = (L + 0.3963377774 * a + 0.2158037573 * b) ** 3
  const m = (L - 0.1055613458 * a - 0.0638541728 * b) ** 3
  const s = (L - 0.0894841775 * a - 1.291485548 * b) ** 3
  return [4.0767416621 * l - 3.3077115913 * m + 0.2309699292 * s,
    -1.2684380046 * l + 2.6097574011 * m - 0.3413193965 * s,
    -0.0041960863 * l - 0.7034186147 * m + 1.707614701 * s]
}
function oklch(L, C, H) { // trả {r,g,b} 0–1; ngoài gamut sRGB thì hạ chroma, giữ L và H
  const fits = (c) => lchToLinear(L, c, H).every((v) => v >= -1e-4 && v <= 1 + 1e-4)
  if (!fits(C)) { let lo = 0, hi = C; for (let i = 0; i < 20; i++) { const mid = (lo + hi) / 2; if (fits(mid)) lo = mid; else hi = mid } C = lo }
  const [r, g, b] = lchToLinear(L, C, H).map((v) => Math.min(1, Math.max(0, enc(v))))
  return { r, g, b }
}
function toOklch(rgb) { // {r,g,b} 0–1 → {L, C, H}
  const [R, G, B] = [rgb.r, rgb.g, rgb.b].map(lin)
  const l = Math.cbrt(0.4122214708 * R + 0.5363325363 * G + 0.0514459929 * B)
  const m = Math.cbrt(0.2119034982 * R + 0.6806995451 * G + 0.1073969566 * B)
  const s = Math.cbrt(0.0883024619 * R + 0.2817188376 * G + 0.6299787005 * B)
  const a = 1.9779984951 * l - 2.428592205 * m + 0.4505937099 * s
  const b = 0.0259040371 * l + 0.7827717662 * m - 0.808675766 * s
  return { L: 0.2104542553 * l + 0.793617785 * m - 0.0040720468 * s, C: Math.hypot(a, b), H: ((Math.atan2(b, a) * 180) / Math.PI + 360) % 360 }
}
const lum = (c) => 0.2126 * lin(c.r) + 0.7152 * lin(c.g) + 0.0722 * lin(c.b)
const contrast = (x, y) => { const p = lum(x), q = lum(y); return (Math.max(p, q) + 0.05) / (Math.min(p, q) + 0.05) }
const toHex = (c) => '#' + [c.r, c.g, c.b].map((v) => Math.round(v * 255).toString(16).padStart(2, '0')).join('').toUpperCase()
const hueGap = (a, b) => { const d = Math.abs(a - b) % 360; return Math.min(d, 360 - d) }

// ── Ramp 11 nấc (bảng §6) ──
const STEPS = [50, 100, 200, 300, 400, 500, 600, 700, 800, 900, 950]
const L_COLOR = [0.97, 0.94, 0.89, 0.82, 0.72, 0.64, 0.56, 0.48, 0.4, 0.32, 0.22]
const C_COLOR = [0.02, 0.04, 0.07, 0.11, 0.15, 0.18, 0.19, 0.17, 0.14, 0.11, 0.07]
const L_NEUTRAL = [0.985, 0.965, 0.92, 0.87, 0.71, 0.55, 0.45, 0.37, 0.28, 0.21, 0.15]
const C_NEUTRAL = [0.004, 0.006, 0.008, 0.01, 0.012, 0.014, 0.014, 0.012, 0.01, 0.008, 0.006]
// shift: tổng độ dịch hue từ nấc 50 tới 950, có dấu (bảng chiều dịch §6); k: hệ số chroma (màu trầm < 1)
function ramp(H, Ls, Cs, shift = 0, k = 1) {
  const out = {}
  STEPS.forEach((s, i) => { out[s] = oklch(Ls[i], Cs[i] * k, H + shift * (i / (STEPS.length - 1) - 0.5)) })
  return out
}
// Hạ L một nấc (mỗi lần 0,01, tối đa 0,06) tới khi mọi cặp [nền, ngưỡng] đạt; không đạt thì trả null
function fitStep(L, C, H, pairs) {
  for (let d = 0; d <= 0.0601; d += 0.01) {
    const c = oklch(L - d, C, H)
    if (pairs.every(([bg, min]) => contrast(c, bg) >= min)) return c
  }
  return null
}
```

Kiểm helper bằng một điểm đã biết: `toOklch(figma.util.rgb('#2563EB'))` phải ra khoảng
`{ L: 0.546, C: 0.215, H: 262.9 }`, và `contrast` của nó với trắng là 5,17.

Ví dụ xem trước ramp từ một màu thương hiệu mà **chưa tạo gì trong file** (dán helper lên trước):

```js
const brand = toOklch(figma.util.rgb('#0D9488'))
const shift = brand.H > 110 && brand.H < 270 ? 8 : -8
const p = ramp(brand.H, L_COLOR, C_COLOR, shift, Math.min(1, brand.C / 0.19))
const white = { r: 1, g: 1, b: 1 }
return STEPS.map((s) => s + ' ' + toHex(p[s]) + ' trắng ' + contrast(p[s], white).toFixed(2))
```

Trình bày kết quả cho người dùng dưới dạng bảng và nói rõ nấc nào chở được chữ trắng. Chỉ khi người
dùng đồng ý mới dựng Variables.

## 8. Màu không bao giờ là tín hiệu duy nhất

Khoảng 8% nam giới có dạng mù màu đỏ-lục (protanopia, deuteranopia). Tritanopia (lam-vàng) hiếm hơn
nhiều. WCAG 1.4.1 yêu cầu thông tin không được truyền **chỉ** bằng màu:
- Trạng thái lỗi của ô nhập có viền đỏ **và** icon **và** dòng chữ nói lỗi gì.
- Tăng/giảm trong bảng số có dấu `+`/`−` hoặc mũi tên, không chỉ xanh/đỏ.
- Link nằm giữa đoạn văn có gạch chân, hoặc chênh ≥ 3:1 với chữ xung quanh **và** có thêm dấu hiệu khi
  hover hoặc focus.
- Biểu đồ phân biệt series bằng cả nhãn hay hình dạng điểm, không chỉ bằng màu.

`audit_design {scope: "palette"}` mô phỏng ba dạng mù màu trên biến màu (rule `mu-mau-trung-nhau`). Nó
chỉ nói hai màu **có thể** trùng nhau; tín hiệu thứ hai thì vẫn phải kiểm trên canvas.
