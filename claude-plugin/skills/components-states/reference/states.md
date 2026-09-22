# Trạng thái của component

Một nút vẽ đẹp ở trạng thái Default mới là **một phần sáu** của nút. Dev gặp trạng thái nào chưa
vẽ thì sẽ tự nghĩ ra — và mỗi người nghĩ một kiểu.

Code trong file này là thân của một lần gọi `execute_figma_code`. Công thức dựng cả set Button ở
[components.md §4](components.md#4-dựng-một-component-set-từ-đầu--button).

## Mục lục

1. Bộ trạng thái mặc định
2. Màu theo nấc ramp — hover, pressed
3. Focus
4. Disabled
5. Loading
6. Error và Success của ô nhập
7. Trạng thái của các thành phần khác
8. Chuyển động giữa các trạng thái
9. Ma trận trạng thái — điền trước khi vẽ

---

## 1. Bộ trạng thái mặc định

| Trạng thái | Nói với người dùng | Trên canvas |
|---|---|---|
| **Default** | "Chỗ này bấm được" | Màu nấc chính của ramp (thường 600) |
| **Hover** | "Chuột đang trỏ vào đây" — màn cảm ứng không có | Lệch ≥ 1 nấc |
| **Pressed** | "Đã nhận cú bấm" | Lệch thêm 1 nấc; mobile lệch ≥ 2 nấc so với Default |
| **Focus** | "Phím Tab đang dừng ở đây" | Vòng 2px, cách mép 2px |
| **Disabled** | "Chưa dùng được, vì …" | Token neutral + câu nói vì sao, đặt cạnh |
| **Loading** | "Đang làm, đừng bấm lại" | Spinner + chữ tiến trình, giữ bề rộng |
| **Error** *(ô nhập)* | "Giá trị này chưa dùng được" — hiện sau khi rời ô hoặc bấm gửi | Viền danger + câu dưới ô |
| **Success** *(ô nhập)* | "Giá trị này đã được xác nhận" — **chỉ khi điều đó có ích** | Viền success + câu dưới ô |

| Thành phần | Trục `State` |
|---|---|
| Button, icon button | Default · Hover · Pressed · Focus · Disabled · Loading |
| Input, select, textarea | Default · Hover · Focus · Disabled · Error · Success |
| Checkbox, radio, switch | `Checked` (Off · On · Mixed cho checkbox) × Default · Hover · Focus · Disabled |
| Table row | Default · Hover · Selected · Focus |
| Nav item, tab | Default · Hover · Current · Focus |
| Tag, chip lọc | Default · Hover · Selected · Focus · Disabled |
| Link | Default · Hover · Focus (· Visited nếu là nội dung đọc) |

**Hover phục vụ chuột, Focus phục vụ bàn phím**; trên màn cảm ứng chỉ còn Pressed báo đã chạm.
Gộp Hover và Focus thành một hình là bỏ rơi một trong hai nhóm: mỗi trạng thái cần một variant
riêng, nhìn khác nhau.

Toast, banner, badge có trục **`Tone`** (Info · Success · Warning · Danger) — đó là nghĩa, không
phải trạng thái; đừng trộn vào trục `State`.

## 2. Màu theo nấc ramp — hover, pressed

Nấc ramp và vai trò token do skill `color-system` dựng. Nếu file chưa có, công thức Button dùng các
hex dự phòng dưới đây (neutral slate + primary blue — đã đo, chữ trên nền đều ≥ 4,5:1).

| Kind | Default | Hover | Pressed | Chữ | Disabled |
|---|---|---|---|---|---|
| Primary | `action` #2563EB | `action-hover` #1D4ED8 | `action-active` #1E40AF | `text-on-action` #FFFFFF (5,17 → 8,72) | nền `bg-disabled` · chữ `text-disabled` (file chưa có: `bg-subtle` · `text-secondary`) |
| Secondary | `bg-surface` + viền `border-input` | `bg-subtle` #F1F5F9 | `bg-subtle-active` #E2E8F0 | `text-primary` #0F172A | như trên, bỏ viền |
| Tertiary | trong suốt | `action-subtle-bg` #EFF6FF | `action-subtle-bg-active` #DBEAFE | `action-subtle-text` #1D4ED8 (≥ 5,4) | chữ như Primary, không nền |
| Destructive | `danger` #DC2626 | `danger-hover` #B91C1C | `danger-active` #991B1B | `text-on-action` (4,83 → 8,31) | như Primary |

- **Vai trò còn thiếu.** Bộ vai trò mặc định của skill `color-system` có `action`, `action-hover`,
  `action-active`, `action-subtle-bg`, `bg-subtle`, `danger`, nhưng nấc nhấn của nền phụ và nấc
  hover/nhấn của danger chỉ là vai trò **thêm** khi dựng bộ nút. Công thức Button báo chúng trong
  `missingTokens`. Thêm vào collection vai trò có sẵn của file, mỗi biến là **alias** tới primitive,
  theo đúng quy ước tên của file. Với hệ của `color-system` (Light / Dark): `color/bg/subtle-active` →
  neutral-200 / neutral-700 · `color/action/subtle-bg-active` → primary-100 / primary-800 ·
  `color/danger/hover` → danger-800 / danger-300 · `color/danger/active` → danger-900 / danger-200
  (đoạn thêm vào script: skill `color-system`, reference/tokens-and-modes.md §3). Rồi chạy lại để gắn
  biến. Hỏi người dùng trước khi thêm biến vào file của họ.
- **Hover lệch ít nhất một nấc** trên ramp: 600 → 700; nút phụ nền trắng → neutral-100 → neutral-200.
- **Không bao giờ hover bằng opacity.** Hạ opacity cả nút là kéo chữ nhạt theo, contrast tụt.
  `audit_design` tính cả opacity của layer và của cha khi đo chữ (`tuong-phan-chu`), nên sẽ bắt
  được; chữ có opacity dưới 0,7 còn bị cảnh báo riêng (`chu-mo-bang-opacity`).
- **Pressed** lệch thêm một nấc (800). Mobile không có hover, nên pressed phải lệch **≥ 2 nấc** so
  với Default — lệch một nấc dưới ngón tay gần như không thấy.
- **Dark mode**: primary sáng lên khoảng hai nấc (Default 400, Hover 300, Pressed 200) và chữ trên
  nút chuyển sang tối. Mode tối là việc của biến, không phải variant riêng: gắn biến đúng thì
  `setExplicitVariableModeForCollection` trên một khung là xem được cả set ở mode tối.
- **Ba phép thử** trên screenshot của cả set:
  1. **Che nhãn** — các ô trong một hàng vẫn phân biệt được với nhau?
  2. **Thang xám** — chỉ nhìn độ sáng, Default, Hover, Pressed, Disabled còn khác nhau?
  3. **Không chỉ bằng hue** — Error và Success của ô nhập khác nhau cả ở chữ, không chỉ ở màu viền.

## 3. Focus

Vòng focus cho người dùng bàn phím biết mình đang ở đâu. Nó phải **luôn nhìn thấy** (WCAG 2.4.7)
và là thành phần không phải chữ, nên cần **≥ 3:1 với màu nền quanh nó** (WCAG 1.4.11).

| Thuộc tính | Mặc định |
|---|---|
| Độ dày | 2px |
| Khoảng cách tới mép | 2px — vòng không chạm vào nền nút |
| Bo góc | bán kính của nút + 4 (bán kính mép ngoài của vòng) |
| Màu | biến `focus-ring`; dự phòng #2563EB — 5,17:1 trên trắng, 4,94:1 trên #F8FAFC |
| Áp cho | mọi thứ focus được: button, link, input, checkbox, tab, dòng bảng — **cùng một kiểu** |

- Nhờ khe 2px, vòng chỉ cần tương phản với **nền trang**, không cần tương phản với màu nút. Không có
  khe (vòng dính sát mép) thì vòng **phải khác màu nút** — nếu không, nút primary lúc focus trông
  như không có vòng.
- Ô nhập ở Focus: viền đổi sang `action` **và** có vòng như trên.
- Ghi chú cho dev: vòng chỉ hiện khi điều khiển bằng bàn phím, không hiện khi bấm chuột.

Thêm vòng vào một variant đã có (ví dụ `State=Focus`, hoặc khung `Field` trong ô nhập):

```js
const node = await figma.getNodeByIdAsync(NODE_ID)
if (node?.type !== 'COMPONENT' && node?.type !== 'FRAME') throw new Error('NODE_ID phải là variant hoặc frame, không phải instance')
const radius = typeof node.cornerRadius === 'number' ? node.cornerRadius : 6
const token = (await figma.variables.getLocalVariablesAsync('COLOR')).find((v) => /focus[-/ ]?ring$/i.test(v.name))
const base = figma.util.solidPaint('#2563EB')
const ring = figma.createFrame()
ring.name = 'Focus ring'
ring.fills = []
ring.strokes = [token ? figma.variables.setBoundVariableForPaint(base, 'color', token) : base]
ring.strokeWeight = 2
ring.strokeAlign = 'INSIDE'
ring.cornerRadius = radius + 4
node.appendChild(ring)
if (node.layoutMode !== 'NONE') ring.layoutPositioning = 'ABSOLUTE'
ring.resize(node.width + 8, node.height + 8)
ring.x = -4
ring.y = -4
ring.constraints = { horizontal: 'STRETCH', vertical: 'STRETCH' }
node.clipsContent = false
return ring.id
```

Vòng thò ra ngoài 4px: mọi khung bao ngoài cũng phải `clipsContent = false` (component chứa
`Field`, component set), nếu không vòng bị cắt. Chỉ đặt vòng ở variant Focus — `audit_design` bỏ
qua layer ẩn, nhưng layer thừa ở mười variant khác là rác cho người sau.

## 4. Disabled

- **Đổi token, không hạ opacity**, bỏ màu action. Mất màu là tín hiệu rõ nhất vì trong hệ màu này
  **có màu nghĩa là bấm được**.
- **Token:** file có vai trò disabled riêng (`bg/disabled`, `text/disabled`, như hệ của skill
  `color-system` dựng sẵn) thì dùng chúng; công thức Button tự chọn khi tìm thấy. File chưa có thì
  dùng nền `bg-subtle`, chữ `text-secondary`.
- WCAG miễn yêu cầu contrast cho điều khiển đang vô hiệu, nhưng nhãn vẫn phải đọc được — người dùng
  cần biết đó là nút gì để biết mình đang thiếu gì. `audit_design` không biết layer nào đang vô hiệu:
  `text/disabled` của `color-system` (khoảng 2,3:1 ở Light) sẽ bị FAIL `tuong-phan-chu`. Ghi trong
  báo cáo rằng FAIL ở variant Disabled là cố ý (WCAG 1.4.3 miễn cho điều khiển vô hiệu), kèm lý do.
  Cách dự phòng `text-secondary` trên `bg-subtle` (#475569 trên #F1F5F9, 6,92) thì không bị FAIL.
- Ô nhập vô hiệu **giữ viền `border-input`** (≥ 3:1): còn phải nhận ra đó là một ô.
- **Nói vì sao, ngay cạnh**: "Cần chọn ít nhất một học sinh", "Chỉ quản trị viên sửa được mục này".
  Trên trang Components: đặt một instance Tooltip hoặc dòng helper cạnh variant Disabled. Trên màn
  hình mẫu: hiện câu đó dưới nút hoặc dưới ô.
- Thường tốt hơn vô hiệu: **cho bấm rồi báo cụ thể** thiếu gì, ngay tại chỗ thiếu. Vô hiệu một nút
  mà không nói vì sao là bắt người dùng đoán.

## 5. Loading

- **Spinner 16px** thế chỗ icon đầu nút; chữ đổi thành **động từ đang diễn ra** khớp hành động:
  Lưu → "Đang lưu…", Gửi → "Đang gửi…", Xoá 12 mục → "Đang xoá…". Không viết "Loading…" hay "Vui lòng chờ".
- **Giữ nguyên bề rộng** của trạng thái Default — nút co lại làm cả hàng nhảy. Trong set, Loading
  có property chữ riêng (`Loading label`); trên màn hình, khoá bề rộng instance như
  [components.md §5](components.md#5-dùng-instance).
- Ghi chú cho dev: đổi trạng thái nút **ngay** khi bấm; khoá bấm lặp tới khi xong; spinner cho vùng
  lớn chỉ hiện sau khoảng 0,3 giây. Ngưỡng thời gian chờ đầy đủ và frame cần vẽ cho từng mốc: skill
  `design-workflow`, reference/screen-states.md §3.
- Vùng nội dung đang tải: **skeleton đúng kích thước thật** cho thứ biết trước hình dạng (bảng, thẻ,
  danh sách); spinner chỉ cho vùng nhỏ hoặc kết quả không biết hình dạng. Không bao giờ spinner che
  toàn màn hình.

## 6. Error và Success của ô nhập

**Anatomy** (từ trên xuống, auto layout dọc, khoảng cách 8):

| Layer | Nội dung | Mặc định |
|---|---|---|
| `Label` | Tên trường, luôn hiện, **phía trên ô** | 14/20, weight 500, `text-primary` |
| `Field` | Khung ô: cao 40, padding ngang 12, bo 6 | nền `bg-surface`, viền 1px `border-input` (≥ 3:1) |
| `Value` | Giá trị, hoặc placeholder là **ví dụ định dạng** | 14/20; placeholder dùng `text-muted` (vẫn ≥ 4,5:1) |
| `Helper` | Hướng dẫn ngắn, **dưới ô**; ở Error thì thành câu lỗi | 12/16, `text-secondary` |

| State | Viền `Field` | `Helper` |
|---|---|---|
| Default | `border-input` #64748B (4,76:1 trên trắng) | hướng dẫn, `text-secondary` |
| Hover | đậm hơn một nấc, #475569 | như Default |
| Focus | `action` + vòng focus (§3) | như Default |
| Disabled | giữ `border-input`, nền `bg-disabled` (file chưa có: `bg-subtle`) | câu nói vì sao |
| Error | `danger` #DC2626 | câu lỗi, `danger` (4,83 trên trắng · 4,62 trên #F8FAFC) |
| Success | `success` #15803D | câu xác nhận, `success` (5,02 trên trắng) |

- **Màu không là tín hiệu duy nhất.** Câu dưới ô đã nói bằng chữ; thêm icon từ bộ icon của file qua
  INSTANCE_SWAP nếu có. Đổi mỗi màu viền là chưa đủ.
- Câu lỗi theo công thức **chuyện gì + vì sao → làm gì tiếp**, 1–2 dòng, không đổ lỗi:
  "Tên này đã có người dùng. Thử thêm số, vd: nguyen.an2." — chi tiết ở [ux-copy.md](ux-copy.md).
- **Success chỉ khi có ích**: tên đăng nhập còn trống, mã giảm giá hợp lệ. Không tô xanh mọi ô đúng
  — người dùng đang nhập, không cần được khen từng ô.
- Lỗi xuất hiện **lúc nào** là hành vi, không vẽ được → ghi chú cho dev: đang gõ chỉ đếm ký tự,
  không hiện đỏ · rời ô thì kiểm định dạng · bấm gửi thì kiểm liên trường, cuộn tới ô sai đầu tiên.
- Đa số ô bắt buộc → đánh dấu ô **tuỳ chọn** "(không bắt buộc)", đỡ nhiễu hơn rải dấu sao.

## 7. Trạng thái của các thành phần khác

| Thành phần | Điểm cần nhớ |
|---|---|
| **Checkbox, radio** | Ô 16–20px nhưng **cả hàng kèm nhãn là vùng bấm** (≥ 24, mobile ≥ 44): component `Checkbox` gồm cả nhãn, cao ≥ 24 — `audit_design` đo vùng bấm trên chính layer tên Checkbox. Checked đổi cả hình (dấu tick, chấm), không chỉ màu. Checkbox có thêm `Mixed` cho "chọn một phần" |
| **Switch** | Vị trí núm là tín hiệu chính, màu là phụ. Nhãn nói điều đang bật ("Gửi email nhắc"), không phải "Bật/Tắt" |
| **Table row** | Hover đổi nền `bg-subtle` — **bắt buộc**, giúp mắt bám dòng. Selected: nền `action-subtle-bg` + checkbox On. Focus: vòng như §3 hoặc thanh trái 2px |
| **Nav item, tab** | Current phải rõ bằng **nền hoặc thanh chỉ báo** (tab: gạch dưới 2px), không chỉ đổi màu chữ |
| **Link** | Trong đoạn văn: gạch chân. Chỉ khác màu thì màu link phải chênh ≥ 3:1 với chữ thường quanh nó **và** có gạch chân khi Hover/Focus. Hover: đổi nấc màu hoặc dày gạch chân |
| **Icon button** | Vùng bấm 40 cho icon 16–20 (tối thiểu 32 trên desktop; mobile 44, Android 48). **Bắt buộc** tooltip + tên cho trình đọc màn hình (ghi chú cho dev). Chỉ dùng icon nghĩa phổ quát: xoá, sửa, đóng, tìm, cài đặt, thêm |
| **Tag, chip lọc** | Selected có dấu tick hoặc nền đặc; chip có nút xoá thì nút xoá là vùng bấm riêng ≥ 24 |

## 8. Chuyển động giữa các trạng thái

- Hover, đổi màu, vòng focus: **100–150ms**, ease-out. Tooltip, dropdown, checkbox: 150–200ms.
- Trong prototype: `SMART_ANIMATE`, `EASE_OUT`; hover `duration: 0.12`, nhấn `0.1` (đơn vị giây) —
  [components.md §9](components.md#9-prototype-component-tương-tác). Bảng ánh xạ cho mọi tình huống:
  skill `layout-type`, reference/motion.md §4.
- Không nảy, không lắc. Chuyển động không bao giờ chặn thao tác kế tiếp.

## 9. Ma trận trạng thái — điền trước khi vẽ

Viết bảng này (trong câu trả lời, hoặc ngay trên khung `Doc / <Tên>`) **trước** lần gọi dựng đầu
tiên. Ô nào chưa trả lời được là chỗ dev sẽ phải đoán.

| State | Nền | Chữ | Viền | Khác | Token còn thiếu |
|---|---|---|---|---|---|
| Default | | | | | |
| Hover | | | | | |
| Pressed | | | | | |
| Focus | | | | vòng 2px / cách 2px | |
| Disabled | | | | câu vì sao: … | |
| Loading | | | | spinner, "Đang …", giữ bề rộng | |

Ví dụ đã điền — Button, Kind=Primary, product register:

| State | Nền | Chữ | Viền | Khác |
|---|---|---|---|---|
| Default | `action` | `text-on-action`, 14/20 500 | — | cao 40, padding 16, bo 6 |
| Hover | `action-hover` | như trên | — | +1 nấc |
| Pressed | `action-active` | như trên | — | +2 nấc so với Default |
| Focus | `action` | như trên | — | vòng `focus-ring` 2px, cách 2px, bo 10 |
| Disabled | `bg-disabled` | `text-disabled` | — | tooltip "Cần chọn ít nhất một học sinh" |
| Loading | `action` | như Default | — | spinner 16, "Đang lưu…", giữ bề rộng |
