# Trạng thái màn hình, thông báo và dữ liệu bẩn

Một màn hình là **bảy** thiết kế, không phải một. File này nói vẽ gì cho từng trạng thái, vẽ gì cho
từng mốc thời gian chờ, và thử thiết kế bằng dữ liệu xấu trước khi làm đẹp. Trạng thái của từng control
(hover, focus, error của ô nhập…) và cách viết câu thông báo nằm ở skill `components-states`.

---

## 1. Bảy trạng thái

| # | Trạng thái | Câu hỏi kiểm tra | Hay bị quên | Vẽ gì trên canvas |
|---|---|---|---|---|
| 1 | **Ideal**: có dữ liệu | Thiết kế chính | Không | Thử ở ba cỡ dữ liệu (mục 2) |
| 2 | **Empty**: chưa có gì | Người mới mở lần đầu thấy gì? | **Rất cao** | Một câu nói chỗ này để làm gì · nút chính · đường thoát |
| 3 | **Loading**: đang xử lý | Họ biết hệ thống chưa treo chứ? | Trung bình | Skeleton đúng hình nội dung thật (mục 3) |
| 4 | **Partial**: có một phần | Tải được 8/10 khối thì hiện gì? | **Rất cao** | Phần có dữ liệu hiện bình thường; khối hỏng báo ngay tại chỗ, có nút Thử lại |
| 5 | **Error**: hỏng | Hỏng ở đâu, ai sửa được? | Thấp | Câu ba phần (mục 4) + nút hành động; dữ liệu người dùng vẫn còn |
| 6 | **Offline / Stale** | Có cho làm việc tiếp không? | **Rất cao** | Banner "Đang offline, thay đổi sẽ đồng bộ khi có mạng" hoặc "Cập nhật lúc 10:32"; nút cần mạng thì vô hiệu và nói vì sao |
| 7 | **No permission** | Thấy được mà không được sửa thì hiện gì? | Cao | Xem mục 1.2 |

Component đứng riêng hay landing tĩnh không cần đủ bảy: vẽ cái áp dụng, và ghi trong báo cáo cái nào
bỏ qua, vì sao. Chỉ kịp làm kỹ hai trạng thái thì làm **Empty** và **quá tải**: hai chỗ người dùng thật
sống lâu nhất.

### 1.1. Empty là cơ hội bị bỏ phí nhiều nhất

Màn hình trắng là thất bại. Đó là chỗ dạy bước đầu tiên và cho biết sản phẩm dùng để làm gì. Đủ ba thứ:

1. **Một câu nói chỗ này để làm gì**, không phải "Chưa có dữ liệu".
2. **Nút hành động chính**: đúng nút sẽ dùng khi đã có dữ liệu.
3. **Đường thoát**: hướng dẫn ngắn, dữ liệu mẫu, hoặc nhập từ Excel.

Danh sách và bảng có **ba kiểu trống, ba thông điệp khác nhau**: chưa có dữ liệu (nút tạo mới, nhập
từ Excel) · lọc không ra (nút xoá bộ lọc, gợi ý bỏ bộ lọc nào) · lỗi tải (nút Thử lại, nói dữ liệu vẫn
an toàn). Câu mẫu và cách dựng thành component set `Empty state` với trục `Kind`: skill
`components-states`, reference/patterns.md §8.

### 1.2. Không có quyền

Tách theo câu hỏi: **người dùng có được biết đối tượng này tồn tại không?**

- Được biết (cùng tổ chức, chỉ thiếu vai trò): chế độ chỉ xem, nút sửa vô hiệu kèm lý do; hoặc nói rõ
  cần quyền gì và hỏi ai ("Cần quyền Quản lý kho. Liên hệ chị Lan, quản trị viên.").
- Không được biết: màn "Không tìm thấy", giống hệt màn của đối tượng không tồn tại. Với người không có
  quyền, "không tồn tại" và "không được phép" phải trông như nhau. Nghi ngờ thì chọn cách này.

### 1.3. Dựng các trạng thái trên canvas

Nhân bản frame Ideal rồi chỉ đổi phần khác nhau, để phần còn lại giống hệt. Xếp các trạng thái dọc bên
dưới Ideal, tên `<Màn hình> / <Trạng thái>`. Phần dùng lại ở nhiều trạng thái (banner offline, toast,
khối lỗi, empty) nên là component. Một lần gọi `execute_figma_code`:

```js
const ideal = await figma.getNodeByIdAsync('12:34') // id của frame "… / Ideal"
if (!ideal || ideal.type !== 'FRAME' || !ideal.parent) throw new Error('Không thấy khung Ideal')
const parent = ideal.parent
const base = ideal.name.replace(/\s*\/\s*Ideal$/, '')
const states = ['Empty', 'Loading', 'Partial', 'Error', 'Offline', 'No permission']
const gap = 80
// Vùng sẽ đặt sáu bản sao ngay dưới Ideal phải còn trống
const top = ideal.y + ideal.height + gap
const bottom = ideal.y + (states.length + 1) * (ideal.height + gap)
const blocked = parent.children.some(
  (n) => n.id !== ideal.id && n.x < ideal.x + ideal.width && ideal.x < n.x + n.width && n.y < bottom && top < n.y + n.height,
)
if (blocked) throw new Error('Phía dưới khung Ideal đã có layer khác: chọn chỗ khác trước khi nhân bản')
return states.map((state, i) => {
  const copy = ideal.clone()
  parent.appendChild(copy) // gắn chắc vào đúng page/section của Ideal
  copy.name = `${base} / ${state}`
  copy.x = ideal.x
  copy.y = ideal.y + (i + 1) * (ideal.height + gap)
  return { state, id: copy.id }
})
```

Ideal nằm trong section thì nới section cho vừa sau khi nhân bản (`section.resizeWithoutConstraints(w, h)`).
Sau đó sửa từng frame trong một lần gọi riêng, chụp lại từng cái.

---

## 2. Ba cỡ dữ liệu và dữ liệu bẩn

**Ideal phải thử ở ba cỡ**, không chỉ cỡ đẹp trong bản vẽ:

- **1–2 dòng**: bố cục không được trống hoác.
- **Cỡ vừa**: thiết kế chính.
- **Quá tải**: rất nhiều dòng, tên 200 ký tự không dấu cách. Trên canvas không vẽ hết 5.000 dòng: vẽ
  đủ dòng để tràn khung, thêm tổng số ("1.243 đơn hàng") và thanh phân trang, tiêu đề bảng dính.

Đổ dữ liệu bẩn **ngay sau khi dựng khung**, trước khi làm đẹp. Phần lớn lỗi lộ ra trong mười phút đầu.

| Loại | Giá trị |
|---|---|
| Chuỗi | 200 ký tự không dấu cách · 1 ký tự · ký tự đặc biệt và emoji · ô trống |
| Tiếng Việt | Chữ hoa chồng hai dấu phía trên, dấu nặng phía dưới: "ĐẶNG THỊ NGỌC ẨN", "Nguyễn Thị Hường" |
| Số | Âm · 0 · rất lớn (1.234.567.890) |
| Ngày | Rất cũ (1900) · rất xa (2099) |
| Số lượng | 0 dòng · 1 dòng · rất nhiều dòng |

- **Giá trị rỗng**: vẽ cái giao diện sẽ hiện thay thế ("—", "Chưa có số điện thoại"). Không bao giờ để
  chữ `null`, `undefined`, `NaN`, `Invalid Date`, `[object Object]` trên canvas (`audit_design` bắt
  lỗi này bằng rule `ro-ri-gia-tri`).
- **Dấu tiếng Việt bị cắt** khi line height quá chật, hoặc khi khung cha bật `clipsContent` với chiều cao
  cố định. Chụp vùng đó với `scale` 3 để nhìn; rule `tran-chu` bắt chữ tràn ra ngoài khung đang cắt.
- **Chọn cách chữ dài xử lý**, rồi đặt đúng thuộc tính:

| Chữ | Khi dài | Thuộc tính Figma |
|---|---|---|
| Ô trong bảng, tên trên thẻ | Cắt một dòng bằng "…", hiện đủ khi hover (ghi chú bàn giao) | `layoutSizingHorizontal = 'FILL'`, `textTruncation = 'ENDING'`, `maxLines = 1` |
| Tiêu đề | Xuống dòng; chặn ở số dòng khung chứa được | `textTruncation = 'ENDING'`, `maxLines` (ví dụ 2) |
| Mô tả, đoạn văn | Tự cao theo nội dung | `layoutSizingHorizontal = 'FILL'`, `textAutoResize = 'HEIGHT'` |
| Nhãn nút, tab | Không cắt; nhãn phải ngắn | Khung cha `HUG`, đặt `minWidth` nếu cần |

Nhân bản một màn rồi đổ dữ liệu bẩn theo **tên layer chữ** (vì vậy layer chữ nên đặt tên theo vai trò):

```js
const src = await figma.getNodeByIdAsync('12:34')
if (!src || src.type !== 'FRAME' || !src.parent) throw new Error('Không thấy khung màn hình')
const right = Math.max(...src.parent.children.map((n) => n.x + n.width)) // mép phải của mọi thứ đang có
const copy = src.clone()
src.parent.appendChild(copy)
copy.name = src.name.replace(/\s*\/[^/]*$/, '') + ' / Dữ liệu bẩn'
copy.x = right + 120 // đặt ngoài cùng bên phải, không đè lên màn kế tiếp của luồng
copy.y = src.y

// Tên layer chữ → giá trị bẩn. Đổi khoá theo đúng tên layer trong màn hình.
const dirty = new Map([
  ['Customer name', 'ĐẶNG THỊ NGỌC ẨN'],
  ['Order title', 'ĐơnHàngGiaoLạiChoKháchỞChiNhánhThủĐức'.repeat(6).slice(0, 200)],
  ['Amount', '-1.234.567.890 ₫'],
  ['Date', '01/01/1900'],
  ['Note', 'A'],
])
const skipped = []
for (const t of copy.findAllWithCriteria({ types: ['TEXT'] })) {
  const value = dirty.get(t.name)
  if (value === undefined) continue
  try {
    const fonts = t.getStyledTextSegments(['fontName']).map((s) => s.fontName)
    await Promise.all(fonts.map((f) => figma.loadFontAsync(f)))
    t.characters = value
  } catch (e) {
    skipped.push(t.name) // ví dụ chữ gắn component property: đổi bằng instance.setProperties
  }
}
return { dirtyId: copy.id, skipped }
```

Chụp bản bẩn, chạy `audit_design` trên nó, sửa bản gốc (không sửa bản bẩn), rồi đổ lại. Xong thì hỏi
người dùng có giữ frame bẩn trong file không.

---

## 3. Chờ: ngưỡng thời gian

| Thời gian | Bắt buộc có | Frame cần vẽ |
|---|---|---|
| < 0,1 giây | Không cần gì | — |
| < 1 giây | Đổi trạng thái nút | Nút ở trạng thái loading |
| 1–3 giây | Spinner tại chỗ | Spinner trong vùng đang tải |
| 3–10 giây | Skeleton + chữ nói **đang làm gì** | Frame Loading |
| > 10 giây | Tiến trình có ý nghĩa + nút Huỷ + cho làm việc khác | Thanh tiến trình, nút Huỷ |

- Spinner và skeleton chỉ hiện sau khoảng **0,3 giây**: việc xong sớm hơn thì không hiện gì, vì loader nháy
  lên rồi tắt còn khó chịu hơn chờ. Đổi trạng thái nút thì vẫn đổi ngay. Ghi điều này vào ghi chú bàn giao.
- Chữ mô tả bằng ngôn ngữ nghiệp vụ: không "Đang truy vấn" mà "Đang tải danh sách đơn hàng, 3/5 trang".
- **Skeleton** cho nội dung biết trước hình dạng (bảng, thẻ, danh sách); **spinner** chỉ cho kết quả
  không biết hình dạng, hoặc vùng rất nhỏ. **Không bao giờ spinner che toàn màn hình.**
- Vẽ skeleton: từ frame Loading đã nhân bản, thay chữ bằng thanh chữ nhật cùng kích thước (radius 4,
  màu nền phụ của file), giữ nguyên khung auto layout để kích thước đúng như thật: dữ liệu về thì giao
  diện không nhảy. Component có sẵn biến thể loading thì đổi variant thay vì vẽ.
- Hiện kết quả trước khi máy chủ xác nhận (optimistic) hợp với bật tắt, thích, xoá một dòng, đổi trạng
  thái, sắp lại thứ tự. **Không** hợp với thanh toán, tạo bản ghi cần hiện mã ngay, hành động không
  hoàn tác được. Khi bị từ chối, báo rõ và nói đã khôi phục về bản nào.

---

## 4. Thông báo trên canvas

Chọn kênh (im lặng → inline → toast → banner → modal → toàn trang), công thức câu `[chuyện gì] +
[vì sao] → [làm gì tiếp]`, ba trục ai sửa · có chặn không · dữ liệu ra sao, bậc phòng ngừa trước khi báo
lỗi, và lúc nào kiểm ô nhập: skill `components-states`, reference/ux-copy.md §1–§5 và §8. Phần ảnh hưởng
tới việc vẽ màn hình:

- Mỗi thông báo là một frame hoặc instance thật, đặt đúng chỗ nó sẽ hiện: inline ngay dưới phần tử,
  toast ở **một góc cố định** cho cả sản phẩm, banner ở đầu vùng nội dung. Thời lượng không vẽ được, nên
  ghi bằng annotation: toast thành công tự tắt sau khoảng 4 giây, cảnh báo khoảng 6 giây, **lỗi không tự
  tắt**, toast có nút Hoàn tác giữ 8–10 giây (15 giây kèm thùng rác khi khôi phục tốn công).
- Lỗi chặn việc luôn có ít nhất một nút dẫn tới chỗ sửa được. Dữ liệu đang rủi ro thì thêm câu trấn an
  ("Bản nháp của bạn vẫn còn").
- Ô nhập lỗi hiện khi rời ô hoặc khi bấm gửi, không hiện khi đang gõ: màn form cần một frame ở trạng thái
  đã bấm gửi, có lỗi dưới từng ô sai và tóm tắt ở đầu form khi nhiều lỗi.
- Giọng (xưng hô, mức trang trọng, có emoji không): hỏi một lần hoặc đọc từ `DESIGN.md`, rồi giữ nhất
  quán. Một khái niệm chỉ có một tên trong cả sản phẩm (bảng từ vựng ở [brief.md](brief.md) mục 2.2).

---

## 5. Checklist trước khi giao màn hình

- [ ] Đủ bảy trạng thái cho màn chính (hoặc đã nói rõ vì sao bỏ)? Empty có dạy bước tiếp theo?
- [ ] Ideal đã thử với 1 dòng và với dữ liệu quá tải? Đã đổ "ĐẶNG THỊ NGỌC ẨN" và chuỗi 200 ký tự?
- [ ] Danh sách có ba kiểu trống với ba thông điệp khác nhau?
- [ ] Mỗi thông báo trả lời đủ ba câu hỏi, dùng kênh nhẹ nhất còn đủ tác dụng?
- [ ] Mọi lỗi chặn việc có ít nhất một nút dẫn tới chỗ sửa được?
- [ ] Không còn `null`, `undefined`, `NaN`, lorem ipsum trên canvas (`audit_design`: `ro-ri-gia-tri`, `noi-dung-gia`)?
