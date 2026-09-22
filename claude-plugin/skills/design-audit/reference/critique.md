# Phê bình thiết kế: soi bằng mắt và chấm điểm

Máy đo được contrast, kích thước, nấc. Nó không biết mắt người dùng rơi vào đâu, màn có dạy được
bước đầu tiên không, hay giao diện có trông như dựng theo khuôn. Phần đó làm bằng `screenshot` và
bằng lập luận, theo một khung cố định để lần nào cũng soi đủ.

Mọi nhận xét phải **chỉ vào chỗ cụ thể**: tên layer, id, và điều nhìn thấy trên ảnh chụp. "Trông chưa
ổn" chưa phải phát hiện.

## 0. Thứ tự làm

1. `screenshot` cả frame. Trả lời **ba câu hỏi quan sát** (mục 1) **trước** khi đọc kỹ kết quả
   `audit_design`, vì danh sách số đo dễ kéo mắt về đúng những gì máy đã thấy.
2. Đọc kết quả `audit_design`: FAIL, WARN, SKIP.
3. Làm bảy phép thử bằng mắt (mục 2).
4. Nếu người dùng muốn phê bình: chấm 10 heuristic (mục 3), kiểm tải nhận thức (mục 4), kiểm vẻ
   "AI chung chung" (mục 5).
5. Viết báo cáo theo [output-template.md](output-template.md).

## 1. Ba câu hỏi quan sát

1. Mắt rơi vào đâu **đầu tiên**?
2. Nút nào là **việc chính** của màn này?
3. Nếu là người dùng, sẽ bấm **gì tiếp theo**?

Ba câu trả lời khớp với ý đồ của màn thì thứ bậc đang làm việc. Lệch thì có vấn đề thật, và biết
luôn nó nằm ở đâu. Khi người dùng góp ý mơ hồ ("thấy sao sao"), hỏi lại họ đúng ba câu này để lấy
quan sát thay vì ý kiến.

## 2. Bảy phép thử bằng mắt

### 2.1. Thang xám
- **Đạt khi:** bỏ hết màu mà vẫn thấy nút chính, chữ phụ, mục đang được chọn.
- **Trượt thì sửa thứ bậc** (cỡ, weight, vị trí, khoảng trắng), đừng sửa màu. Màu không được là thứ
  duy nhất tạo thứ bậc.
- **Cách chỉ đọc:** so độ sáng. Đọc fill của nút chính, nút phụ và nền, rồi tính `lum` bằng hàm ở
  [rules.md](rules.md) §6.2. Nút chính khác nút phụ chủ yếu ở hue mà độ sáng gần bằng nhau thì khi
  mất màu hai nút trông như một.
- **Cách nhìn tận mắt** (thêm layer tạm vào file nên **hỏi người dùng trước**): nhân bản frame ra chỗ
  trống cạnh nội dung trên trang hiện tại, phủ một lớp xám blend `SATURATION`, chụp, rồi xoá bản tạm.

```js
const src = await figma.getNodeByIdAsync('FRAME_ID')
if (!src || src.type === 'PAGE' || src.type === 'DOCUMENT') throw new Error('Sai id')
const right = Math.max(0, ...figma.currentPage.children.map((n) => n.x + n.width))
const wrap = figma.createFrame()
wrap.name = 'Kiểm thang xám (tạm)'
wrap.fills = [{ type: 'SOLID', color: { r: 1, g: 1, b: 1 } }] // nền trắng cho chỗ trong suốt của bản sao
wrap.resize(src.width, src.height)
wrap.x = right + 200
const copy = src.clone()
wrap.appendChild(copy)
copy.x = 0
copy.y = 0
const veil = figma.createRectangle()
wrap.appendChild(veil)
veil.resize(src.width, src.height)
veil.fills = [{ type: 'SOLID', color: { r: 0.5, g: 0.5, b: 0.5 } }]
veil.blendMode = 'SATURATION' // giữ độ sáng, bỏ sắc độ của mọi lớp bên dưới
return { wrap: wrap.id }
```

`screenshot` với `node_id` là `wrap`, xem xong thì xoá **đúng bản tạm vừa tạo**:

```js
const wrap = await figma.getNodeByIdAsync('WRAP_ID')
if (!wrap || wrap.type !== 'FRAME' || !wrap.name.startsWith('Kiểm thang xám')) throw new Error('Không phải bản tạm')
wrap.remove()
return 'đã xoá bản tạm'
```

### 2.2. Nheo mắt, làm mờ
- `screenshot` frame với `scale` nhỏ (khoảng 0.2–0.3) để có ảnh cỡ thumbnail. Chi tiết mất đi, chỉ
  còn khối.
- **Đạt khi:** còn đúng **một** điểm nhấn mạnh nhất, và thấy được 2–4 khối lớn tách nhau rõ.
- **Trượt khi:** mọi khối nặng như nhau, hoặc có vài điểm tranh nhau. Sửa bằng cách giảm bớt, không
  thêm: hạ weight, bỏ bóng, bỏ nền của thứ phụ.

### 2.3. Chữ trên ảnh, gradient, kính mờ
- Mọi SKIP `chu-khong-do-duoc` đều phải soi. `screenshot` vùng chữ với `scale` 2, nhìn chỗ **tệ
  nhất** của nền, không phải chỗ đẹp nhất.
- Ảnh do người dùng tải lên sẽ thay đổi. Đừng dựa vào "ảnh này đủ tối". Cần một lớp phủ cố định
  (tối 40–60%) hoặc một mặt nền đặc sau chữ.
- Gradient: đo chữ với từng stop ([rules.md](rules.md) §6.6), lấy số thấp nhất.

### 2.4. Mù màu
- Scope `palette` cho số (`mu-mau-trung-nhau`). Phần mắt phải làm: liệt kê **mọi chỗ chỉ có màu mang
  nghĩa**:
  - chấm trạng thái;
  - số tăng/giảm tô đỏ/xanh mà không có dấu +/−;
  - ô nhập lỗi chỉ đổi viền đỏ;
  - dấu * bắt buộc chỉ bằng màu;
  - các đường trong biểu đồ;
  - link chỉ khác màu chữ.
- Mỗi chỗ cần một **tín hiệu thứ hai**: icon, chữ, hình dạng, vị trí, gạch chân. Link trong đoạn văn
  cần gạch chân, hoặc khác chữ quanh nó ≥ 3:1.
- Kiểm biến thể lỗi của ô nhập: có dòng chữ báo lỗi và icon, không chỉ viền đỏ.

### 2.5. Đủ trạng thái
Liệt kê biến thể của mọi component set (chỉ đọc):

```js
await figma.loadAllPagesAsync() // cần khi tìm trên mọi trang
const sets = figma.root.findAllWithCriteria({ types: ['COMPONENT_SET'] })
return sets.slice(0, 60).map((s) => ({ id: s.id, name: s.name,
  variants: Object.fromEntries(Object.entries(s.componentPropertyDefinitions)
    .filter(([, d]) => d.type === 'VARIANT').map(([k, d]) => [k, d.variantOptions])) }))
```

Đối chiếu với bộ tối thiểu:

| Thành phần | Trạng thái cần có | Soi gì trên từng biến thể |
|---|---|---|
| Nút | default · hover · pressed · focus · disabled · loading | Hover lệch ít nhất một nấc màu, **không** bằng opacity. Pressed lệch ≥ 2 nấc trên mobile (không có hover). Focus: vòng 2px cách mép 2px, ≥ 3:1 với nền quanh nó (có khe thì cùng màu nút vẫn được, dính sát thì phải khác màu nút). Disabled vẫn nhận ra là nút và có cách nói vì sao (tooltip). Loading giữ nguyên bề rộng, có spinner trong nút và chữ kiểu "Đang lưu…" |
| Ô nhập | default · hover · focus · disabled, thêm error · success | Error có chữ báo lỗi ngay dưới ô (chuyện gì + làm gì tiếp), không chỉ viền đỏ |
| Màn hình | có dữ liệu · trống · đang tải · một phần · lỗi · mất kết nối/dữ liệu cũ · không có quyền | Màn trống có một câu nói chỗ này để làm gì, nút chính, và một đường thoát (hướng dẫn, dữ liệu mẫu). Đang tải dùng skeleton đúng hình khi biết trước hình dạng. Không có spinner che cả màn |

Trạng thái "có dữ liệu" phải thử ở **ba cỡ**: 1–2 dòng, cỡ vừa, và quá tải. Thiếu trạng thái thì ghi
tên component/màn và trạng thái thiếu. Đó là phát hiện, không phải góp ý.

### 2.6. Nội dung thật, nội dung bẩn
- Dữ liệu bẩn: tên 200 ký tự không dấu cách · tên 1 ký tự · emoji · ô trống · chữ hoa tiếng Việt chồng
  dấu ("ĐẶNG THỊ NGỌC ẨN", "Nguyễn Thị Hường") · số âm, 0, 1.234.567.890 · ngày 1900 và 2099 · 0 dòng,
  1 dòng, 5.000 dòng.
- Chỉ đọc: tìm layer chữ dễ vỡ, tức hộp chữ cố định (`textAutoResize = 'NONE'`) hoặc line-height sát
  hơn khoảng 1,2 lần cỡ chữ (dấu chồng tầng dễ bị cắt):

```js
const root = await figma.getNodeByIdAsync('FRAME_ID')
if (!root || !('findAllWithCriteria' in root)) throw new Error('Cần id của một frame')
const risky = []
for (const t of root.findAllWithCriteria({ types: ['TEXT'] })) {
  const size = t.fontSize === figma.mixed ? null : t.fontSize
  const lh = t.lineHeight === figma.mixed ? null : t.lineHeight
  const tight = size && lh && ((lh.unit === 'PIXELS' && lh.value < size * 1.2) || (lh.unit === 'PERCENT' && lh.value < 120))
  if (t.textAutoResize === 'NONE' || tight) risky.push({ id: t.id, name: t.name, resize: t.textAutoResize, size, lineHeight: lh })
}
return risky.slice(0, 60)
```

- Muốn thấy tận mắt thì nhân bản frame (hỏi trước), đổ dữ liệu bẩn vào bản sao, chụp. Chỉ kịp thử
  hai thứ thì thử **trống** và **quá tải**, vì người dùng thật sống lâu nhất ở hai chỗ đó.

### 2.7. Nhịp thị giác
- **Gần xa:** khoảng trong nhóm ≤ một nửa khoảng giữa nhóm (nhãn ↔ ô 8 · ô ↔ ô 16 · nhóm ↔ nhóm 32).
  Nhãn cách ô bằng ô cách ô thì mắt ghép nhầm.
- **Thẳng hàng:** các khối chung một mép trái. Lệch 1–2px là lỗi, không phải phong cách (xem `x` bằng
  `inspect_nodes`).
- **Cùng hàng cùng cao:** nút, ô nhập, select nằm một hàng thì cao bằng nhau.
- **Thang chữ theo register:**
  - app/sản phẩm: chữ thân 14/20, điều khiển cao 36–40, hai weight (400 và 500–600);
  - trang đọc, thương hiệu: chữ thân 16/24.
  - Không căn giữa đoạn quá hai dòng. Số trong bảng căn phải, dùng số đều bề rộng.
- **Bo góc theo vai trò:** 0–2 cho bảng và dòng · 4–6 cho điều khiển · 8–12 cho thẻ, modal · 999 cho
  pill. Lồng trong nhỏ hơn bao ngoài: trong = ngoài − padding.
- **Tách bề mặt theo thứ tự:** khoảng trắng → đường kẻ mảnh → nền khác nhẹ → bóng. Bóng chỉ dành cho
  thứ thật sự nổi lên (dropdown, modal, toast).
- **Màu 90/7/3:** khoảng 90% neutral, 7% primary, 3% semantic và accent. Chỗ có màu là chỗ bấm được.
  Mỗi màn đúng **một** nút nền đặc.

## 3. Chấm 10 heuristic của Nielsen (0–4, tổng /40)

Thang điểm dùng chung cho cả mười tiêu chí:

| Điểm | Nghĩa |
|---|---|
| 0 | Trên các frame đã xem, chưa chỉ ra được chỗ nào đáp ứng tiêu chí |
| 1 | Chỉ một hai chỗ lẻ đáp ứng; luồng chính vẫn hổng |
| 2 | Luồng chính ổn; trạng thái phụ (lỗi, trống, đang tải) hổng |
| 3 | Luồng chính và trạng thái phụ đều ổn; còn dưới ba lỗ nhỏ, gọi tên được từng lỗ |
| 4 | Soi hết các frame không thấy lỗ; mỗi ý có layer cụ thể làm bằng chứng |

- Chỉ cho 4 khi chỉ ra được bằng chứng. Điểm 2–3 là chuyện thường, không phải lời chê.
- Frame hiện có không đủ để chấm một tiêu chí (ví dụ chưa có màn lỗi nào) thì ghi **"chưa chấm được"**.
  Đừng cho 2 cho có, và tính tổng trên số tiêu chí đã chấm.

Mười tiêu chí theo thứ tự gốc của Nielsen: visibility of system status · match between system and the
real world · user control and freedom · consistency and standards · error prevention · recognition
rather than recall · flexibility and efficiency of use · aesthetic and minimalist design · help users
recognize, diagnose, and recover from errors · help and documentation.

| # | Tiêu chí | Soi gì trên canvas |
|---|---|---|
| 1 | Cho thấy hệ thống đang làm gì | Biến thể loading của nút, skeleton, "Đã lưu lúc 10:32", bước 2/5, mục điều hướng đang chọn có nền hoặc vạch (không chỉ đổi màu chữ) |
| 2 | Nói ngôn ngữ của người dùng | Từ trên giao diện là từ người dùng nói, không phải tên bảng dữ liệu. Ngày, số, tiền theo kiểu Việt Nam (1.234.567 ₫). Icon chỉ dùng loại ai cũng hiểu |
| 3 | Người dùng làm chủ, luôn có đường lùi | Nút Huỷ và Quay lại trong luồng nhiều bước, X đóng modal, toast có Hoàn tác sau khi xoá |
| 4 | Nhất quán và theo quy ước | Cùng một việc dùng cùng một component (instance của cùng main component), vị trí nút chính giữ một quy ước, một khái niệm một tên |
| 5 | Ngăn lỗi trước khi xảy ra | Chọn thay vì gõ, gợi ý định dạng, giá trị mặc định hợp lý, nút phá huỷ cách nút chính ≥ 24px và không là icon đầu tiên trên dòng |
| 6 | Cho nhận ra thay vì bắt nhớ | Tóm tắt các bước trước, ô tìm có gợi ý, điều hướng luôn hiện (không hamburger trên desktop) |
| 7 | Nhanh cho người đã quen | Hành động hàng loạt có số lượng ("Xoá 25 mục"), gợi ý phím tắt, bộ lọc lưu được, lựa chọn mật độ |
| 8 | Tối giản, đẹp có mục đích | Mỗi phần tử có việc để làm, một điểm nhấn, không trang trí thừa (mục 5) |
| 9 | Giúp hiểu và gỡ lỗi | Câu báo lỗi có chuyện gì + vì sao + làm gì tiếp; giữ nguyên dữ liệu đã nhập; không "Oops"; lỗi không tự biến mất |
| 10 | Trợ giúp đúng chỗ | Màn trống dạy bước đầu, gợi ý nhỏ dưới ô nhập, tooltip cho nút chỉ có icon |

Xếp loại theo tổng (hoặc theo % khi có tiêu chí chưa chấm):

| Tổng /40 | % | Xếp loại | Nghĩa |
|---|---|---|---|
| 33–40 | ≥ 82% | Vững | Chỉ còn đánh bóng |
| 25–32 | 62–81% | Khá | Nền tốt; sửa các tiêu chí ≤ 2 điểm trước khi giao |
| 17–24 | 42–61% | Cần sửa | Người dùng sẽ vấp ở luồng chính |
| 0–16 | < 42% | Làm lại từ gốc | Vấn đề nằm ở nhiệm vụ và luồng, sơn lại bề mặt không cứu được |

Mỗi vấn đề tìm ra gắn một mức ưu tiên:

- **Chặn:** người dùng dừng lại ở đây, việc chính không tới đích.
- **Nặng:** vẫn tới đích nhưng phải trả giá: bấm nhầm, mất dữ liệu, hoặc mất thời gian thấy rõ.
- **Vừa:** vướng víu, nhưng người dùng tự tìm được cách khác để làm.
- **Nhẹ:** không đổi kết quả của việc; sửa để bề mặt gọn và đều hơn.

Một FAIL của `audit_design` nằm trên luồng chính thì ít nhất là **Nặng**.

## 4. Tải nhận thức

Ba loại tải: **nội tại** (độ khó vốn có của việc, không cắt được) · **ngoại lai** (do giao diện gây
ra, như phải nhớ mã hay tự cộng tổng: cắt tối đa) · **hữu ích** (giúp người dùng hiểu, như tổng tiền
cập nhật khi gõ: giữ lại). Checklist dưới đây chỉ nhắm vào tải ngoại lai.

- [ ] Đếm số lựa chọn nhìn thấy cùng lúc ở mỗi chỗ phải quyết định. Quá 4 thì đặt sẵn mặc định, gom
      thành nhóm có tên, hoặc đẩy phần ít dùng vào menu (Cowan 2001: trí nhớ làm việc ôm được khoảng 4 cụm).
- [ ] Mỗi màn có một việc chính và đúng một nút nền đặc.
- [ ] Thứ liên quan nằm gần nhau hoặc chung một nền; khoảng trong nhóm ≤ một nửa khoảng giữa nhóm.
- [ ] Không bắt nhớ từ màn trước: lựa chọn ở bước 1 vẫn thấy ở bước 3.
- [ ] Phức tạp mở dần: tuỳ chọn nâng cao nằm sau một cú bấm, không bày hết ra.
- [ ] Điều hướng trong giới hạn: toàn cục ≤ 7 mục (trần 9) · tab trong trang ≤ 5 · hành động trên một
      dòng 3 cái rồi menu "…". Cần tới cấp thứ tư là mô hình đối tượng đang sai.
- [ ] Chữ là chữ của người dùng, và một khái niệm chỉ có một tên trên mọi màn.
- [ ] Khối lượng hợp khung: dashboard ≤ 4 chỉ số lớn; wizard 5–7 trường mỗi bước; form quá 5 trường
      hoặc có rẽ nhánh thì ra trang riêng, không nhét vào modal.

Không cộng điểm. Mỗi mục trượt là một vấn đề, ghi kèm chỗ và cách sửa. Trượt từ ba mục trở lên thì
đề nghị xem lại bố cục và luồng của màn, không chỉ vá từng chỗ.

## 5. Vẻ "AI chung chung"

Đây là phép thử gu có lý lẽ, không phải luật. Giao diện thương hiệu (landing, chiến dịch) được biểu
cảm hơn giao diện sản phẩm, nhưng biểu cảm phải có lý do.

Dấu hiệu, và cách thấy trên canvas:

1. **Gradient mặc định không gắn với thương hiệu:** tím sang xanh, hồng sang cam trên hero, nút, thẻ.
   `inspect_nodes` thấy fill `GRADIENT_*` ở khắp nơi. `gradient-chu` bắt riêng trường hợp chữ.
2. **Lưới thẻ đúc khuôn:** một hàng thẻ cùng cỡ, bên trong xếp y hệt nhau, dù nội dung vốn không ngang
   hàng hay không cùng độ quan trọng. Danh sách dữ liệu thật (sản phẩm, học sinh) dùng chung một
   component là đúng; chỉ tính là dấu hiệu khi các thẻ chở những ý khác loại.
3. **Căn giữa tất cả:** hero, tiêu đề mục, đoạn nhiều dòng, cả form. `textAlign: CENTER` và
   `counterAxisAlignItems: CENTER` ở hầu hết các frame.
4. **Trang trí không có việc:** đốm màu mờ, quầng sáng, kính mờ (`kinh-mo`), bóng trên mọi thẻ, viền màu
   một cạnh chỉ để "có điểm nhấn", emoji thay icon.
5. **Thẻ lồng thẻ:** mục nào cũng bọc trong một khung có nền, viền và bóng.
6. **Một kiểu cho mọi thứ:** cùng bo góc lớn, cùng bóng, cùng độ đậm, nên không còn thứ bậc (phép nheo mắt trượt).
7. **Con số khoe không nguồn:** "10x", "99,9%" cỡ khổng lồ, không nói đo cái gì.
8. **Câu chữ dùng cho sản phẩm nào cũng được:** khẩu hiệu không nhắc tới việc cụ thể nào của người dùng.
9. **Màu và font theo phản xạ ngành:** chọn vì "ngành này hay dùng", không vì thương hiệu hay người dùng.

**Phép thử đoán ngành:** che logo và tên sản phẩm. Nếu người lạ đoán ra ngành chỉ nhờ bộ màu và font
"ai cũng dùng" của ngành đó, hoặc đổi tên sang một đối thủ mà màn vẫn đúng nguyên, thì thiết kế chưa
có cá tính.

**Kết luận:**
- 0 dấu hiệu rõ: có chủ đích.
- 1–2: vài chỗ theo phản xạ, sửa từng chỗ.
- Từ 3: trông như mẫu dựng tự động. Đề nghị xem lại hướng thị giác từ đầu (skill `design-workflow`)
  trước khi đánh bóng.
