---
name: components-states
description: Dựng component tái sử dụng và đủ trạng thái trực tiếp trên canvas Figma bằng các tool của plugin magic-design-with-figma (execute_figma_code, inspect_nodes, screenshot, audit_design) — main component, variant Property=Value, component property TEXT/BOOLEAN/INSTANCE_SWAP, trạng thái hover/pressed/focus/disabled/loading/error/success, trang Components có lưới tài liệu. Dùng khi người dùng muốn tạo, sửa hoặc kiểm button, input, select, checkbox, table row, nav, tabs, modal, drawer, toast, empty state, hoặc chọn khung màn hình (list + detail, dashboard, wizard, form, canvas) và viết câu chữ giao diện trong file Figma đang mở qua plugin này; dựng cả một màn hình hay một luồng thì bắt đầu từ skill design-workflow.
---

# Component và trạng thái trong Figma

Một component tốt là vẽ một lần, dùng trăm lần, và **đủ trạng thái để dev không phải đoán**. Skill
này gồm bốn phần: cơ chế component của Figma, bộ trạng thái, thư viện quyết định cho các pattern
màn hình, và câu chữ đặt trên canvas.

**Tri thức, không phải luật.** Thứ tự ưu tiên: variables, styles và component **có sẵn trong file**
thắng → `DESIGN.md` của dự án nếu người dùng có → mặc định trong skill này. Mặc định chỉ để bắt đầu
khi chưa có gì. Có lý do cụ thể thì làm khác, và ghi lý do lên canvas (description của component
hoặc trang Cover).

## Khi nào dùng

- Tạo mới hoặc mở rộng một component: button, icon button, input, select, checkbox, radio, switch,
  tag, table row, nav item, tab, modal, drawer, toast, banner, empty state.
- Thêm hoặc sửa variant, component property, trạng thái; dọn một component set lộn xộn.
- Chọn khung màn hình và quyết định pattern (nút nào đặc, form mấy cột, modal hay drawer…).
- Viết câu chữ trên giao diện: nhãn nút, thông báo lỗi, trạng thái trống, ghi chú thời gian chờ.

Việc dựng bảng màu và biến màu thuộc skill `color-system`; thang chữ, spacing và kích thước thuộc
`layout-type`; quy trình cả màn hình thuộc `design-workflow`; đọc kết quả `audit_design` và phê bình
thuộc `design-audit`. Skill này dùng kết quả của chúng.

## Quy trình — mỗi bước khoảng một lần gọi tool

1. **Đọc file trước.** `get_context`, rồi `execute_figma_code` liệt kê component set, variables,
   text styles trên mọi trang ([components.md §2](reference/components.md#2-tìm-component-có-sẵn-trước-khi-vẽ)).
   Có component gần đúng → dùng hoặc mở rộng nó, **không vẽ bản thứ hai**. Main component đang có
   instance (`getInstancesAsync()`) hoặc cần thêm trang mới → hỏi người dùng trước khi sửa.
2. **Chọn register** (bảng số mặc định bên dưới): product/app hay brand/đọc.
3. **Viết ma trận trước khi vẽ**, gửi ngắn gọn cho người dùng: anatomy (tên layer), property
   (VARIANT · BOOLEAN · TEXT · INSTANCE_SWAP), danh sách trạng thái kèm token màu
   ([states.md §9](reference/states.md#9-ma-trận-trạng-thái--điền-trước-khi-vẽ)).
4. **Dựng đủ variant và gộp set** — auto layout, gắn biến, `combineAsVariants`, xếp lưới
   hàng = Kind/Size, cột = State ([components.md §4](reference/components.md#4-dựng-một-component-set-từ-đầu--button)).
5. **Thêm component property** và nối `componentPropertyReferences` cho **mọi** variant.
6. **Tài liệu trên trang Components**: tiêu đề, một câu công dụng, nhãn hàng/cột, ghi chú cho dev.
7. **Kiểm mọi variant**, không chỉ instance đang nằm trên màn hình: script kiểm tổ hợp thiếu và
   kích thước ([components.md §8](reference/components.md#8-kiểm-mọi-variant)), `audit_design`
   với `node_id` là component set, `screenshot` cả set, rồi đổ dữ liệu bẩn vào một instance.
8. **Báo cáo**: đã tạo gì (id), token nào còn thiếu (đang dùng hex dự phòng), FAIL/WARN còn lại
   kèm lý do, và những gì máy không đo được (`notMeasured`).

Sau mỗi bước có thay đổi nhìn thấy được: `screenshot` rồi sửa chỗ lệch trước khi đi tiếp.

## Quy ước tối thiểu

- **Tên set** là tên thành phần: `Button`, `Input`, `Table row`. **Tên variant** là
  `Property=Value` nối bằng dấu phẩy: `Kind=Primary, Size=M, State=Hover`. File đã có quy ước
  khác (`Type`, `Variant`, `Status`…) → theo file.
- **Tên layer theo vai trò**: `Label`, `Icon`, `Field`, `Helper`, `Spinner`, `Focus ring`.
  `audit_design` nhận ra vùng bấm và ô nhập **qua tên layer** (button, link, tab, chip, checkbox,
  radio, switch, toggle, close… và input, field, select, dropdown, text area…). Đặt tên sai là
  máy bỏ sót.
- **Chọn loại property**: thứ đổi hình hoặc màu theo nhóm cố định → VARIANT; bật tắt một layer →
  BOOLEAN; chữ → TEXT; đổi icon hoặc layer con → INSTANCE_SWAP. Không làm variant cho thứ chỉ khác chữ.
- **Không detach instance** để sửa nhanh. Cần khác đi → thêm variant hoặc property ở main.
- Màu gắn **biến theo vai trò** (`action`, `action-hover`, `text-on-action`, `border-input`,
  `focus-ring`…), không gắn hex. File chưa có biến → dùng hex dự phòng và báo danh sách thiếu.

## Bộ trạng thái — tóm tắt

| Thành phần | Trạng thái |
|---|---|
| Button, icon button, link | Default · Hover · Pressed · Focus · Disabled · Loading |
| Input, select, textarea | Default · Hover · Focus · Disabled · **Error** · **Success** |
| Checkbox, radio, switch | trục `Checked` × Default · Hover · Focus · Disabled |
| Table row · nav item · tab | Default · Hover · Selected/Current · Focus |

- **Hover lệch ít nhất một nấc ramp** (600 → 700). **Không bao giờ hover bằng opacity** — opacity
  kéo cả chữ nhạt theo và contrast tụt.
- **Pressed** lệch thêm một nấc (800). Mobile không có hover → pressed lệch **≥ 2 nấc** so với default.
- **Focus**: vòng 2px, cách mép 2px, đạt ≥ 3:1 với nền quanh nó. Một kiểu vòng cho mọi thành phần.
- **Disabled**: đổi token (`bg/disabled`, `text/disabled` khi file có), không hạ opacity; **nói vì sao
  ngay cạnh** (helper text hoặc tooltip).
- **Loading**: spinner trong nút, chữ tiến trình ("Đang lưu…"), **giữ nguyên bề rộng**.
- **Error/Success** của ô nhập: viền đổi màu **và** có icon + câu dưới ô — màu không là tín hiệu duy nhất.

Chi tiết, công thức Figma và bảng token: [reference/states.md](reference/states.md).

## Số mặc định theo register

| | Product / app (mặc định) | Brand / đọc |
|---|---|---|
| Chữ thân | 14/20 | 16/24 |
| Nhãn nút, nhãn ô | 14/20, weight 500–600 | 16/24, weight 500–600 |
| Chiều cao control | 36–40 (dày: 28–32 cho bảng, toolbar) | 36–40 (mobile, người dùng phổ thông: 44–48) |
| Padding ngang nút | 16 (12 khi cao 32) | 16 (20 khi cao 48) |
| Độ đậm | 2: 400 + 500 hoặc 600 | 2–3: thêm 700 cho tiêu đề lớn |
| Bo góc | control 4–6 · thẻ, modal 8–12 · pill 999 | theo thương hiệu |
| Icon | 16, đi với chữ 14 | 20, đi với chữ 16 |
| Vùng bấm | sàn 24 · nên 32 trên desktop · 44 trên mobile (48dp Android) | như product |
| Khoảng cách | bội số 4 · icon↔chữ 4–8 · nhãn↔ô 8 · ô↔ô 16 · nhóm↔nhóm 32 | như product |

Nút, ô nhập, select **nằm cùng hàng thì cùng chiều cao**. Icon 16px vẫn có vùng bấm 40px nhờ padding
(tối thiểu 32 trên desktop, 44 trên mobile).

## Pattern và câu chữ — những quyết định hay dùng nhất

- **Đúng một nút nền đặc mỗi màn hình.** Nhãn nút = **động từ + danh từ**: "Lưu thay đổi", "Xoá 12 học sinh".
- **Form một cột**, nhãn phía trên ô, placeholder chỉ là ví dụ định dạng, lỗi nằm dưới ô.
- **Bảng**: số căn phải, chữ căn trái, không căn giữa; dòng 32/40/48 theo mật độ; hover dòng bắt buộc.
- **Sidebar** 240–280 khi mở, 56–64 khi thu gọn. **Modal** 400 / 560 / 720. **Drawer** 400–560.
- **Trạng thái trống** = một câu nói chỗ này để làm gì + đúng nút hành động chính + một đường thoát.
- **Thông báo** = chuyện gì + vì sao → làm gì tiếp. Chọn kênh nhẹ nhất còn đủ tác dụng:
  inline → toast → banner → modal.
- Hành vi chỉ có khi chạy thật (nhốt focus trong modal, Escape để đóng, URL giữ bộ lọc, chờ gõ xong
  mới tìm, tên cho trình đọc màn hình) **không vẽ được** → ghi thành ghi chú cho dev trên canvas.

Đầy đủ: [reference/patterns.md](reference/patterns.md) · [reference/ux-copy.md](reference/ux-copy.md).

## Bẫy Plugin API hay gặp

- `setProperties` cần **tên đầy đủ có hậu tố** cho TEXT/BOOLEAN/INSTANCE_SWAP (`Label#12:3`).
  Đọc tên từ `componentPropertyDefinitions`, đừng đoán. VARIANT dùng tên trần (`State`).
- `componentPropertyReferences` đặt trên **layer bên trong từng variant**, không đặt trên set.
- INSTANCE_SWAP nhận **id của một COMPONENT** làm giá trị mặc định (không phải id của set);
  `preferredValues` dùng `key` của component.
- `combineAsVariants` chỉ nhận ComponentNode; mỗi tổ hợp giá trị phải là duy nhất.
  `defaultVariant` là variant nằm **trên cùng bên trái** — xếp Default của Kind chính vào góc đó.
- Nạp font trước khi tạo chữ, trước khi nối property TEXT và trước khi `setProperties` đổi chữ.
- `layoutSizingHorizontal = 'FILL'` chỉ đặt sau khi đã `appendChild` vào khung auto layout.
- Dynamic page: đọc main component bằng `await instance.getMainComponentAsync()`.
- Vòng focus nằm ngoài mép nút → component phải `clipsContent = false`.
- Plugin API **chỉ đọc** được `openTypeFeatures`: số dạng bảng (tabular) phải bật trong text style
  bằng tay — nhờ người dùng, hoặc ghi chú cho dev.
- `audit_design` bỏ qua layer ẩn và chỉ đo vùng bấm trên layer có tên giống nút → kích thước từng
  variant phải kiểm bằng script. SKIP nghĩa là **chưa đo được**, không phải đạt.

## Checklist trước khi báo xong

- [ ] Đủ trạng thái theo bảng trên; che nhãn đi vẫn phân biệt được từng ô trong lưới?
- [ ] Hover và pressed đi theo nấc ramp, không có opacity trên chữ hay trên cả nút?
- [ ] Vòng focus 2px cách mép 2px, ≥ 3:1 với nền; giống nhau ở mọi component?
- [ ] Variant Disabled có câu giải thích đặt cạnh; Loading giữ bề rộng?
- [ ] Mọi variant đã nối property (Label, Icon…); không còn tổ hợp thiếu trong set?
- [ ] `audit_design` trên component set: không FAIL; WARN còn lại có lý do?
- [ ] Mọi màu và khoảng cách gắn biến hoặc style, khoảng cách là bội số của 4?
- [ ] Đã đổ dữ liệu bẩn (tên 200 ký tự, "ĐẶNG THỊ NGỌC ẨN", ô trống, số âm): không tràn, không cắt dấu?
- [ ] Không còn lorem ipsum, `undefined`, `NaN`, `null` trên canvas?
- [ ] Hành vi không vẽ được đã thành ghi chú cho dev?

## Tài liệu chi tiết

- [reference/components.md](reference/components.md) — main component và instance, chọn property,
  công thức dựng Button, instance lồng nhau, trang Components, kiểm mọi variant, prototype.
- [reference/states.md](reference/states.md) — từng trạng thái, token theo nấc ramp, vòng focus,
  disabled, loading, error/success, ma trận trạng thái.
- [reference/patterns.md](reference/patterns.md) — năm khung xương, nút, form, bảng, điều hướng,
  overlay, tìm kiếm, trạng thái trống, ghi chú cho dev, bảng thử dữ liệu bẩn cho component.
- [reference/ux-copy.md](reference/ux-copy.md) — công thức thông báo, ba trục, thang gián đoạn và thời
  lượng toast, phòng ngừa và xác nhận, nhãn nút, câu chữ form và lúc kiểm lỗi, giọng văn.

Trạng thái cả màn hình (bảy trạng thái), ngưỡng thời gian chờ, bộ dữ liệu bẩn, bảng từ vựng: skill
`design-workflow`.
