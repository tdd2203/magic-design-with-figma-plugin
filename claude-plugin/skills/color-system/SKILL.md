---
name: color-system
description: Chọn và dựng bảng màu thành Figma Variables (ramp OKLCH, collection Primitives → Semantic có mode Light/Dark, gắn biến vào fill/stroke, kiểm contrast và dark mode) bằng các tool của magic-design-with-figma như execute_figma_code, audit_design và screenshot. Dùng khi người dùng muốn tạo hoặc sửa bảng màu, color token, màu thương hiệu, dark mode, chuyển màu viết cứng sang biến, hoặc hỏi "màu thế này ổn chưa" trên file Figma đang mở qua plugin Magic Design with Figma.
---

# Hệ màu trong Figma

Skill này biến câu "màu thương hiệu là #…" thành một **hệ màu sống trong file**: ramp đo được, token
đặt theo vai trò, Light/Dark đổi bằng một lần chuyển mode, và mọi layer gắn biến chứ không mang mã hex.
Thứ bàn giao là **bảng token trong Variables**, không phải ba mã hex. Bàn giao ba mã hex thì người
làm tiếp sẽ tự chế thêm ba mươi mã nữa.

## Tri thức, không phải luật

Khi các nguồn nói khác nhau, theo thứ tự:

1. **Variables, styles, components đã có trong file.** Mở rộng hệ đó, không dựng hệ song song.
2. **DESIGN.md của dự án**, nếu người dùng có: theo màu và tên token trong đó.
3. **Mặc định trong skill này.**

Riêng ngưỡng contrast là phép đo WCAG, không phải gu. File có sẵn mà trượt ngưỡng thì báo cho người
dùng kèm số đo, đừng im lặng làm theo.

## Sáu ý cốt lõi

- **90/7/3.** Khoảng 90% diện tích là neutral (nền, thẻ, viền, chữ), 7% primary (nút, link, mục
  đang chọn), 3% semantic và accent. Hệ quả: **chỗ nào có màu là chỗ bấm được.** Tô màu vùng không
  tương tác là nói dối mắt người dùng.
- **Dựng bằng OKLCH, bàn giao RGB 0–1.** Trong OKLCH, cùng L là sáng ngang nhau thật; HSL thì không.
  Mọi số hue trong skill này là **hue OKLCH** (xanh dương `#2563EB`: HSL 221°, OKLCH 263°).
- **Lightness quyết định thứ bậc và khả năng đọc.** Chỉ được chỉnh một thứ thì chỉnh L.
- **Neutral trước, primary theo chức năng, semantic cố định nghĩa, tối đa một accent.**
- **Hai lớp token.** Primitives giữ giá trị, Semantic giữ vai trò và có mode Light/Dark. Layer chỉ
  gắn biến Semantic.
- **Đo, đừng tin bảng.** Mọi bảng số ở đây là điểm xuất phát. Đo lại sau mỗi lần đổi.

**Register.** Giao diện sản phẩm (app, dashboard, công cụ) giữ đúng 90/7/3: không tô màu tiêu đề,
không nền màu ở vùng lớn. Trang thương hiệu (landing, campaign) được phép có hero nền màu và tiêu đề
có màu, nhưng màu vẫn phải cùng hệ với sản phẩm bên trong.

## Quy trình

**0. Đọc file trước khi chọn màu.** Gọi `get_context`, rồi một `execute_figma_code` chỉ đọc để liệt kê
collection, mode, số biến màu và paint style ([tokens-and-modes.md §11](reference/tokens-and-modes.md)).
File đã có hệ thì báo lại cho người dùng và mở rộng hệ đó. Chỉ dựng mới khi file chưa có gì.

**1. Lấy đầu vào.** Gồm màu thương hiệu (hex), register, có cần dark mode không, loại sản phẩm. Không
có màu thương hiệu thì gợi ý từ [palettes.md](reference/palettes.md). Chọn sơ đồ phối màu theo bảng ở
[color-theory.md §3](reference/color-theory.md). Phân vân thì chọn **đơn sắc cộng một accent**.

**2. Chọn primary theo chức năng, không theo poster.** Primary phải qua ba câu hỏi:
- Chữ trắng trên nó đạt 4,5:1 không? (Hoặc chữ tối, nếu cố ý làm nút sáng.)
- Có nấc đủ sáng để thay nó trong dark mode không?
- Lặp 20 lần trên một màn hình có mệt mắt không?

Màu rực và sáng (vàng, mint, cam nhạt) là primary tồi dù đẹp. Nếu thương hiệu bắt buộc, cho nó làm
accent và lấy nấc đậm hơn cùng hue làm primary. **Màu logo dùng cho logo, primary dùng cho nút.**

**3. Neutral, semantic, accent.**
- Neutral 10–12 nấc (script dựng 11), ám chroma khoảng 0,005–0,015 về **hue của primary**. Không để
  xám thuần, và không ám "ấm cho thân thiện" theo phản xạ.
- Semantic: success 140–155, warning 60–85, danger 25–35, info 240–265. Mỗi màu cách primary
  **≥ 60°**. Không cách được thì bắt buộc có tín hiệu thứ hai (icon, chữ), xem
  [color-theory.md §4](reference/color-theory.md).
- Accent: tối đa một, và phải viết được câu "accent này dùng cho X ở Y".

**4. Dựng Variables.** Chạy script ở [tokens-and-modes.md §4](reference/tokens-and-modes.md) trong
**một** lần `execute_figma_code`. Script tạo collection `Primitives` (một mode `Value`, 11 nấc cho mỗi
nhóm neutral, primary, success, warning, danger, info) và collection `Semantic` (mode `Light`/`Dark`,
mọi giá trị là alias trỏ về Primitives). Nó tự hạ L nấc 600 để chữ trắng và link trên nền subtle đạt
4,5, rồi trả về `semanticNearPrimary`: những semantic nằm gần primary dưới 60°, cần xử lý ở bước 3.

**5. Gắn biến.** Component và màn hình chỉ gắn biến **Semantic**:

```js
const sem = (await figma.variables.getLocalVariableCollectionsAsync()).find((c) => c.name === 'Semantic')
if (!sem) throw new Error('Chưa có collection Semantic')
const vars = (await figma.variables.getLocalVariablesAsync('COLOR')).filter((v) => v.variableCollectionId === sem.id)
const token = (name) => { const v = vars.find((x) => x.name === name); if (!v) throw new Error('Thiếu biến ' + name); return v }
const paint = (name) => figma.variables.setBoundVariableForPaint({ type: 'SOLID', color: { r: 0, g: 0, b: 0 } }, 'color', token(name))
const card = await figma.getNodeByIdAsync('12:34') // id thật từ inspect_nodes
card.fills = [paint('color/bg/surface')]
card.strokes = [paint('color/border/default')]
```

Màu viết cứng sẵn có trong file thì chuyển sang biến theo
[tokens-and-modes.md §8](reference/tokens-and-modes.md). Chỉ tự gắn những màu khớp chính xác; màu
còn mơ hồ thì hỏi người dùng.

**6. Trang tài liệu và bản Dark.** Tạo bảng swatch gắn biến Primitives
([tokens-and-modes.md §7](reference/tokens-and-modes.md)). Nhân bản một màn hình mẫu, đặt mode Dark
cho bản sao bằng `setExplicitVariableModeForCollection`
([tokens-and-modes.md §6](reference/tokens-and-modes.md)), rồi `screenshot` cả hai bản cạnh nhau.

**7. Kiểm.** Chạy theo thứ tự ở [checks.md](reference/checks.md):
1. `audit_design {scope: "palette"}` trên các biến màu, sửa hết FAIL.
2. `audit_design {node_id, scope: "design"}` trên bảng mẫu và màn hình thật, **ở cả hai mode**.
3. Làm nốt 8 bước kiểm. Trong đó có những bước máy không làm được: thang xám, nội dung thật, đủ trạng
   thái, điều kiện thực tế.

`SKIP` nghĩa là **không đo được**, không phải "đạt". Mục `notMeasured` phải được kiểm bằng mắt hoặc
chuyển sang người dùng.

**8. Báo cáo.** Nêu những gì đã tạo (collection, mode, số biến), contrast của các cặp chính ở cả hai
mode, kết quả audit (tính cả SKIP), những chỗ lệch mặc định kèm lý do, và việc người dùng cần tự kiểm.

## Vai trò Semantic, tóm tắt

Đầy đủ giá trị Light/Dark, số đo và scope ở [tokens-and-modes.md §3](reference/tokens-and-modes.md),
kể cả bốn vai trò thêm khi dựng bộ nút đầy đủ (`bg/subtle-active`, `action/subtle-bg-active`,
`danger/hover`, `danger/active`).

| Nhóm | Biến |
|---|---|
| Nền | `color/bg/page` · `surface` · `subtle` · `disabled` |
| Viền | `color/border/default` (trang trí) · `color/border/input` (**≥ 3:1**, WCAG 1.4.11) |
| Chữ | `color/text/primary` · `secondary` · `muted` (vẫn là chữ, cần 4,5) · `disabled` · `on-action` |
| Hành động | `color/action/default` · `hover` · `active` · `subtle-bg` · `subtle-text` |
| Focus | `color/focus-ring`: vòng 2px, cách nút 2px, ≥ 3:1 với nền |
| Trạng thái | `color/success\|warning\|danger\|info/default` và `/subtle-bg` |

Ngưỡng: chữ thường 4,5:1; chữ lớn (**24px**, hoặc **18,66px khi weight ≥ 700**) 3:1; viền ô nhập,
icon và thành phần giao diện 3:1. Đo trên **màu đã trộn** nếu nền hoặc lớp có độ trong suốt.

## Dark mode, ba luật

1. Nền **không** `#000000`, mà trong khoảng `#0F172A`–`#18181B`. Đen thuần làm chữ sáng loé và viền rung.
2. Chữ sáng **không** `#FFFFFF`, mà dùng neutral-50.
3. Primary **sáng lên khoảng 2 nấc** (600 → 400) và bớt chroma, vì trên nền tối màu trông bão hoà hơn thật.

Bóng đổ không có tác dụng trên nền tối. Phân tầng bằng **độ sáng nền**: bề mặt càng nổi càng sáng.

## Cấm, bản rút gọn

Dùng màu thay cho thứ bậc · nền màu ở vùng lớn (trừ hero trang thương hiệu) · hai accent · mã hex viết
cứng trong component · đen thuần hoặc trắng thuần · hover bằng opacity · semantic dùng để trang trí ·
chép bảng màu của sản phẩm khác mà không chạy lại 8 bước kiểm. Bản đầy đủ ở
[checks.md §4](reference/checks.md).

## Tài liệu trong skill

- [reference/color-theory.md](reference/color-theory.md): OKLCH, chọn sơ đồ, ba nhóm màu, dựng ramp
  (bảng L/C, chiều dịch hue, nấc 600 đo thật), helper JS OKLCH ⇄ RGB và contrast.
- [reference/tokens-and-modes.md](reference/tokens-and-modes.md): hai lớp token trong Figma Variables,
  bảng vai trò, script dựng, scope, gắn biến, chuyển mode, chuyển màu viết cứng, Variables hay paint style.
- [reference/palettes.md](reference/palettes.md): bảng màu khởi đầu cho 14 loại sản phẩm (đã đo lại),
  cộng phần riêng cho mobile.
- [reference/checks.md](reference/checks.md): 8 bước kiểm làm trên Figma, cách đọc `audit_design`,
  các bẫy khi đo, danh sách cấm, mẫu báo cáo.

Skill liên quan trong plugin: **design-workflow** (điều phối cả màn hình), **components-states**
(biến thể trạng thái dùng các token này), **layout-type** (chữ, khoảng cách), **design-audit**
(chi tiết về `audit_design`).
