# Tổ chức file và bàn giao

Mục tiêu: người dựng giao diện mở file ra là hiểu đủ, không phải hỏi lại. Muốn vậy cần ba thứ: file
sắp xếp dễ tìm, thiết kế qua checklist, và ghi chú nằm ngay cạnh thứ nó nói tới.

---

## 1. Tổ chức file

| Page | Chứa gì |
|---|---|
| `Cover` | Một frame bìa: tên sản phẩm, trạng thái (Đang làm / Chờ duyệt / Sẵn sàng bàn giao), người phụ trách, ngày cập nhật. Đặt làm ảnh thu nhỏ của file |
| `Foundations` | Bảng hiển thị variables và styles: màu theo vai trò, thang chữ, spacing, bo góc, hiệu ứng |
| `Components` | Main component và component set, mỗi nhóm một section, kèm mô tả cách dùng |
| `Screens` | Màn hình theo luồng: mỗi luồng một section, màn theo thứ tự từ trái sang phải, trạng thái xếp dọc dưới mỗi màn |

- File đã có cách tổ chức riêng thì theo nó. Không tạo page mới trong file đang có nhiều page, không
  chuyển nội dung của người dùng sang page khác khi chưa hỏi.
- Bản cũ: hỏi trước khi xoá; muốn giữ thì chuyển vào page `Archive`.
- Trước khi bàn giao, dọn phần do chính mình tạo: không còn `Frame 12`, `Group 5`, layer ẩn thừa, group
  lồng vô nghĩa. Layer ẩn của người dùng thì để nguyên.

Tạo các page chuẩn trong file mới (file mới chỉ có một page trống, và đó là page hiện tại):

```js
const wanted = ['Cover', 'Foundations', 'Components', 'Screens']
const pages = figma.root.children
if (pages.length === 1 && pages[0].children.length === 0) pages[0].name = 'Cover' // file mới: đổi tên page trống
for (const name of wanted) {
  if (!figma.root.children.some((p) => p.name === name)) figma.createPage().name = name
}
const screens = figma.root.children.find((p) => p.name === 'Screens')
if (!screens) throw new Error('Không tạo được page Screens')
await figma.setCurrentPageAsync(screens) // phải chuyển page trước khi dựng trên page đó
return { pages: figma.root.children.map((p) => p.name) }
```

Gom các màn của một luồng vào một section (các frame phải nằm trực tiếp trên page hiện tại):

```js
const FRAME_IDS = ['12:34', '12:35']
const frames = []
for (const id of FRAME_IDS) {
  const n = await figma.getNodeByIdAsync(id)
  if (n && n.type === 'FRAME' && n.parent === figma.currentPage) frames.push(n)
}
if (!frames.length) throw new Error('Không có khung nào nằm trực tiếp trên page hiện tại')
const pad = 80
const left = Math.min(...frames.map((f) => f.x))
const top = Math.min(...frames.map((f) => f.y))
const right = Math.max(...frames.map((f) => f.x + f.width))
const bottom = Math.max(...frames.map((f) => f.y + f.height))
const section = figma.createSection()
section.name = 'Luồng đặt hàng'
section.x = left - pad
section.y = top - pad
section.resizeWithoutConstraints(right - left + pad * 2, bottom - top + pad * 2)
for (const f of frames) {
  const [x, y] = [f.x - section.x, f.y - section.y] // toạ độ trong section tính từ góc section
  section.appendChild(f)
  f.x = x
  f.y = y
}
return { sectionId: section.id }
```

Đặt frame bìa làm ảnh thu nhỏ của file: `await figma.setFileThumbnailNodeAsync(coverFrame)`.

---

## 2. Checklist bàn giao

Mục ghi `(audit: …)` được `audit_design` đo; mục ghi `(ghi chú)` là thứ không vẽ được, phải viết thành
annotation cho dev (mục 3); còn lại tự xét bằng screenshot. Thông báo và trạng thái:
[screen-states.md](screen-states.md) mục 5. Màu: skill `color-system`.

**Đặc tả**
- [ ] Việc làm nhiều nhất mất bao nhiêu cú bấm? Bớt được một không?
- [ ] Chữ trên giao diện trùng từ người dùng thật nói (bảng từ vựng, [brief.md](brief.md) mục 2.2)?
- [ ] Có ô nào hệ thống tự biết (ngày tạo, người tạo, mã tự sinh) mà vẫn bắt nhập?
- [ ] Trạng thái đáng nhớ (tab, bộ lọc, sắp xếp, trang) có địa chỉ riêng; quay lại còn nguyên bộ lọc và vị trí cuộn? (ghi chú)

**Tương tác**
- [ ] Mọi nút có đủ sáu trạng thái, ô nhập có thêm error và success; nút vô hiệu nói vì sao?
- [ ] Hành động phá huỷ cách nút hay dùng ≥ 24px, và không là icon đầu tiên trên dòng?
- [ ] Hành động hàng loạt nói số lượng ("Xoá 25 mục")?
- [ ] Gợi ý phím tắt nào hiện trên giao diện cũng phải chạy thật? (ghi chú)
- [ ] Mỗi màn hình đúng một nút nền đặc?

**Bề mặt**
- [ ] Gap và padding là bội số của 4 (audit: `lech-nac-khoang-cach`)
- [ ] Khoảng trong nhóm nhỏ hơn khoảng giữa nhóm, tỉ lệ ≥ 1:2?
- [ ] Nút, ô nhập, select cùng hàng cùng chiều cao?
- [ ] Trong bảng: số căn phải, chữ căn trái, không căn giữa?
- [ ] Lớp nổi (dropdown, modal, toast, tooltip) nằm trên cùng theo đúng thứ tự; chỉ lớp nổi mới có bóng?
- [ ] Màu gắn variable hoặc style, chữ gắn text style (audit: `mau-khong-bien`, `chu-khong-style`)

**Tiếp cận**
- [ ] Contrast chữ trên nền thật (audit: `tuong-phan-chu`); viền ô nhập ≥ 3:1 (audit: `ranh-gioi-o-nhap`)
- [ ] Mọi control có trạng thái focus nhìn thấy được (vòng 2px, cách 2px, ≥ 3:1 với nền)?
- [ ] Mọi ô nhập có nhãn thật, không chỉ placeholder?
- [ ] Vùng bấm ≥ 24px, trên mobile 44px (audit: `vung-bam-nho`, `vung-bam-mobile`)
- [ ] Thứ tự focus, alt text, dùng được hoàn toàn bằng bàn phím, zoom 200% không vỡ bố cục (ghi chú)

**Dữ liệu**
- [ ] Đã đổ dữ liệu bẩn: 0 dòng, 1 dòng, rất nhiều dòng, tên 200 ký tự, "ĐẶNG THỊ NGỌC ẨN"?
- [ ] Ba loại bảng trống có ba thông điệp khác nhau?
- [ ] Tìm kiếm không phân biệt dấu: gõ `nguyen` ra "Nguyễn", gõ `dat` ra "Đạt" (chữ `đ` không tự tách dấu khi chuẩn hoá Unicode, dev phải đổi riêng) (ghi chú)
- [ ] Không còn `null`, `undefined`, `NaN`, lorem ipsum (audit: `ro-ri-gia-tri`, `noi-dung-gia`)

**Tốc độ cảm nhận**
- [ ] API chậm 3 giây: frame Loading có nói đang làm gì?
- [ ] Skeleton cùng kích thước nội dung thật, giao diện không nhảy khi dữ liệu về?
- [ ] Ô tìm kiếm chờ khoảng 300ms sau lần gõ cuối mới tìm, hiện số kết quả? (ghi chú)

---

## 3. Ghi chú bàn giao trên canvas

**Ghi gì**, theo thứ tự hay bị hỏi nhất:

1. **Thứ tự đọc và thứ tự focus**: đánh số trên từng control ("Focus 1: Email", "Focus 2: Mật khẩu").
2. **Alt text**: ảnh mang nghĩa thì mô tả nội dung; ảnh, icon trang trí thì ghi "trang trí, trình đọc
   màn hình bỏ qua"; nút chỉ có icon thì ghi nhãn đọc lên ("Đóng", "Xoá đơn hàng").
3. **Tương tác**: trigger, kết quả, thời lượng và easing chuyển động (số liệu ở skill `layout-type`);
   phím tắt; điều gì xảy ra khi bấm ra ngoài modal.
4. **Co giãn**: cái gì giãn theo chiều ngang, `minWidth`/`maxWidth`, bố cục đổi thế nào ở bề rộng nhỏ.
5. **Giới hạn nội dung**: tối đa bao nhiêu ký tự, cắt bằng "…" hay xuống dòng, hover có hiện đủ không.
6. **Thông báo**: kênh (inline, toast, banner, modal) và thời lượng toast.
7. **Định dạng dữ liệu**: số `1.234.567 ₫`, ngày `dd/MM/yyyy`, giờ 24h, số trong bảng dùng chữ số đều bề rộng.
8. **Quyết định khác mặc định** và lý do ([brief.md](brief.md) mục 7).

**Ghi ở đâu**: ưu tiên annotation gốc của Figma (dev đọc trong Dev Mode, nằm đúng trên layer). Không gắn
được thì đặt khung ghi chú cạnh màn hình. Một lần gọi làm cả hai đường:

```js
const screen = await figma.getNodeByIdAsync('12:34')
if (!screen || screen.type !== 'FRAME' || !screen.parent) throw new Error('Không thấy khung màn hình')
const target = screen.findOne((n) => n.name === 'Button/Primary')
const text =
  '**Lưu thay đổi**: Enter cũng gửi. Đang lưu → spinner trong nút, giữ bề rộng. Lỗi → toast không tự tắt.'

try {
  if (!target || !('annotations' in target)) throw new Error('Không gắn annotation được lên layer này')
  const categories = await figma.annotations.getAnnotationCategoriesAsync()
  const category =
    categories.find((c) => c.label === 'Tương tác') ??
    (await figma.annotations.addAnnotationCategoryAsync({ label: 'Tương tác', color: 'blue' }))
  target.annotations = [
    ...target.annotations,
    { labelMarkdown: text, categoryId: category.id, properties: [{ type: 'fills' }, { type: 'cornerRadius' }] },
  ]
} catch (e) {
  // Dự phòng: khung ghi chú đặt cạnh màn hình, cùng page/section
  await figma.loadFontAsync({ family: 'Inter', style: 'Regular' })
  const note = figma.createFrame()
  screen.parent.appendChild(note)
  note.name = `Ghi chú / ${screen.name}`
  note.resize(320, 100)
  note.layoutMode = 'VERTICAL'
  note.primaryAxisSizingMode = 'AUTO'
  note.counterAxisSizingMode = 'FIXED'
  note.paddingLeft = note.paddingRight = note.paddingTop = note.paddingBottom = 16
  note.cornerRadius = 8
  note.fills = [{ type: 'SOLID', color: { r: 1, g: 0.97, b: 0.8 } }]
  const line = figma.createText() // font mặc định Inter Regular, đã load ở trên
  line.characters = text.replace(/\*\*/g, '')
  line.fontSize = 14
  note.appendChild(line)
  line.layoutSizingHorizontal = 'FILL'
  // Bên phải màn hình nếu trống; không thì đặt phía trên, để không đè lên màn kế tiếp của luồng
  const x = screen.x + screen.width + 40
  const free = !screen.parent.children.some(
    (n) => n !== note && n.x < x + note.width && x < n.x + n.width && n.y < screen.y + note.height && screen.y < n.y + n.height,
  )
  note.x = free ? x : screen.x
  note.y = free ? screen.y : screen.y - note.height - 40
}
return { ok: true }
```

- `annotations` được gán nguyên mảng: muốn thêm thì trải mảng cũ ra trước, như trên, để không xoá ghi
  chú của người khác.
- `properties` cho Dev Mode hiện giá trị đo được của layer (`fills`, `cornerRadius`, `itemSpacing`,
  `padding`, `fontSize`…). Chỉ khai thuộc tính mà layer đó thật sự có.
- Dùng vài category cố định cho cả file: Tương tác, Tiếp cận, Nội dung, Co giãn, Quyết định.
- Người dùng cho biết tên token trong code thì khai cho biến, để Dev Mode hiện đúng tên:
  `variable.setVariableCodeSyntax('WEB', 'var(--color-bg-page)')`.

---

## 4. Đánh dấu sẵn sàng bàn giao

Khi `audit_design` đã hết FAIL, checklist đã qua và **người dùng bảo bàn giao**:

```js
const frame = await figma.getNodeByIdAsync('12:34')
if (frame && frame.type === 'FRAME') frame.devStatus = { type: 'READY_FOR_DEV', description: 'Đã qua audit_design, còn 2 WARN có lý do' }
```

`devStatus` chỉ đặt được trên frame hoặc section nằm ngay dưới page hoặc dưới section, và không đặt lồng
bên trong một node đã có `devStatus`. Cả luồng sẵn sàng thì đánh dấu section thay cho từng frame. Cập nhật
trạng thái trên frame bìa cho khớp.
