# Chuyển động — thời lượng, easing, prototype Figma

Chuyển động có một việc: **cho người dùng biết cái gì vừa đổi và nó đến từ đâu.** Hiệu ứng không
làm việc đó thì bỏ: trong app dùng hằng ngày, mỗi hiệu ứng lặp lại hàng trăm lần, và lần thứ một
trăm nó chỉ còn là thời gian chờ.

Mọi số là mặc định. `DESIGN.md` hoặc file đã có quy ước chuyển động → theo đó.

---

## 1. Thời lượng

| Loại | Thời lượng | Ví dụ | `duration` trong Figma |
|---|---|---|---|
| Vi mô | 100–150ms | Hover, đổi màu, vòng focus | 0.1–0.15 |
| Nhỏ | 150–200ms | Tooltip, dropdown, checkbox | 0.15–0.2 |
| Vừa | 200–300ms | Modal, drawer, accordion | 0.2–0.3 |
| Lớn | 300–400ms | Chuyển trang, panel toàn màn hình | 0.3–0.4 |

- **Trên 400ms là chậm** với app dùng hằng ngày. Phần tử càng lớn, đi càng xa thì càng lâu, trong
  khung trên.
- Cặp mở–đóng: lượt đóng ngắn hơn lượt mở, cỡ 75% (modal mở 0.2 thì đóng 0.15). Lúc bấm đóng,
  người dùng đã xong việc với nó.
- `duration` của transition tính bằng **giây**. `timeout` của `AFTER_TIMEOUT` và `delay` tính bằng
  **mili giây**. Nhầm đơn vị là lỗi hay gặp nhất khi viết prototype bằng code.

## 2. Easing

| Chuyển động | Easing | Figma `easing` |
|---|---|---|
| Xuất hiện, đi vào | ease-out — nhanh lúc đầu, dừng êm | `{ type: 'EASE_OUT' }` |
| Biến mất, đi ra | ease-in — chậm lúc đầu, rời nhanh | `{ type: 'EASE_IN' }` |
| Di chuyển trong màn hình | ease-in-out | `{ type: 'EASE_IN_AND_OUT' }` |
| Spinner, thanh tiến trình | linear | `{ type: 'LINEAR' }` |
| **Không biết chọn** | **200ms ease-out** | `{ type: 'EASE_OUT' }`, `duration: 0.2` |

- Cần ease-out dứt khoát hơn mặc định: `CUSTOM_CUBIC_BEZIER` với
  `easingFunctionCubicBezier: { x1: 0.22, y1: 1, x2: 0.36, y2: 1 }` — lao nhanh rồi dừng rất êm.
- **Không nảy, không đàn hồi:** không dùng `BOUNCY`, `EASE_IN_BACK`, `EASE_OUT_BACK`,
  `EASE_IN_AND_OUT_BACK` (vượt quá đích rồi bật lại), không `CUSTOM_SPRING` có nảy. Các spring
  preset khác (`GENTLE`, `QUICK`, `SLOW`) chỉ dùng khi người dùng muốn và đã xem thử không vượt đích.

## 3. Chuyển động phải giải thích được

- **Đến từ đâu, về đâu:** dropdown mọc từ nút mở nó · drawer chi tiết trượt từ phải, drawer điều
  hướng từ trái · toast vào từ góc cố định của nó · modal hiện tại chỗ, nền mờ dần phía sau.
- **Chỉ động cái đang đổi:** phản hồi khi bấm, mở–đóng lớp phủ, đổi trạng thái, chuyển màn. Màn
  hình mở ra thì nội dung có mặt ngay, không lần lượt hiện từng khối.
- **Không bao giờ chặn thao tác:** không xếp chuỗi `AFTER_TIMEOUT` bắt người dùng đợi trước khi
  bấm được.
- **Giảm chuyển động:** prototype chỉ thể hiện một phiên bản. Ghi trong bàn giao chuyển động nào
  là cần thiết và phương án cho người bật giảm chuyển động ở hệ điều hành: hiện tức thì
  (`transition: null`) hoặc `DISSOLVE` ngắn thay cho trượt, phóng.

## 4. Ánh xạ sang prototype Figma

| Tình huống | Trigger | Navigation | Transition | Giây |
|---|---|---|---|---|
| Hover nút | `ON_HOVER` | `CHANGE_TO` biến thể Hover | `SMART_ANIMATE` · `EASE_OUT` | 0.12 |
| Nhấn | `ON_PRESS` | `CHANGE_TO` biến thể Pressed | `SMART_ANIMATE` · `EASE_OUT` | 0.1 |
| Checkbox, toggle | `ON_CLICK` | `CHANGE_TO` | `SMART_ANIMATE` · `EASE_OUT` | 0.15 |
| Dropdown, popover, tooltip | `ON_CLICK` / `ON_HOVER` | `OVERLAY` | `DISSOLVE` · `EASE_OUT` | 0.15 |
| Modal | `ON_CLICK` | `OVERLAY` | `DISSOLVE` · `EASE_OUT` | 0.2 |
| Drawer bên phải | `ON_CLICK` | `OVERLAY` | `MOVE_IN`, `direction: 'LEFT'` · `EASE_OUT` | 0.25 |
| Bottom sheet (mobile) | `ON_CLICK` | `OVERLAY` | `MOVE_IN`, `direction: 'TOP'` · `EASE_OUT` | 0.25–0.3 |
| Chuyển trang desktop | `ON_CLICK` | `NAVIGATE` | `DISSOLVE` hoặc `SMART_ANIMATE` · `EASE_IN_AND_OUT` | 0.3 |
| Đi sâu một cấp (mobile) | `ON_CLICK` | `NAVIGATE` | `PUSH`, `direction: 'LEFT'` · `EASE_IN_AND_OUT` | 0.3 |
| Toast tự tắt | `AFTER_TIMEOUT` 4000 (thành công) · 6000 (cảnh báo) | action `CLOSE` | — | — |

- `direction` là **hướng chuyển động**: `'LEFT'` = trượt sang trái, tức đi vào từ mép phải. Dễ nhầm
  — nhờ người dùng bấm Play kiểm lại.
- Toast báo lỗi **không tự tắt**; có nút đóng.
- `SMART_ANIMATE` ghép lớp theo **tên và cấu trúc**: hai biến thể phải cùng tên lớp, cùng cây lớp,
  không thì nó chỉ mờ chéo.
- Vị trí overlay (giữa, bên phải, bám nút), nền tối phía sau, "bấm ra ngoài để đóng" là thuộc tính
  **chỉ đọc** với Plugin API — nhắc người dùng đặt trong bảng Prototype của frame overlay.
- Toast có nút Hoàn tác giữ 8000–10000ms.

## 5. Mẫu code

Hover và nhấn cho component set nút (Default → Hover khi rê chuột 0.12, Hover → Pressed khi nhấn 0.1,
`SMART_ANIMATE` · `EASE_OUT`, cho mọi `Kind` trong set): skill `components-states`,
reference/components.md §9.

Mở drawer bên phải bằng overlay:

```js
const trigger = await figma.getNodeByIdAsync('BUTTON_ID')
const drawer = await figma.getNodeByIdAsync('DRAWER_FRAME_ID') // frame cấp trên cùng, rộng 400–560
if (!trigger || !('setReactionsAsync' in trigger) || !drawer) throw new Error('Sai id')
await trigger.setReactionsAsync([{
  trigger: { type: 'ON_CLICK' },
  actions: [{
    type: 'NODE', destinationId: drawer.id, navigation: 'OVERLAY',
    transition: {
      type: 'MOVE_IN', direction: 'LEFT', matchLayers: false, duration: 0.25,
      easing: { type: 'CUSTOM_CUBIC_BEZIER', easingFunctionCubicBezier: { x1: 0.22, y1: 1, x2: 0.36, y2: 1 } },
    },
  }],
}])
return 'Nhờ người dùng đặt overlay position = bên phải trong bảng Prototype'
```

## 6. Kiểm

`audit_design` không đo chuyển động và `screenshot` không chụp được nó. Đọc lại các reaction trên
page, chỉ đọc, tìm chỗ quá chậm hoặc có nảy:

```js
const bouncy = ['BOUNCY', 'EASE_IN_BACK', 'EASE_OUT_BACK', 'EASE_IN_AND_OUT_BACK', 'CUSTOM_SPRING']
const out = []
for (const n of figma.currentPage.findAll((x) => 'reactions' in x && x.reactions.length > 0)) {
  if (!('reactions' in n)) continue
  for (const r of n.reactions) {
    for (const a of r.actions || []) {
      if (a.type !== 'NODE' || !a.transition) continue
      const t = a.transition
      if (t.duration > 0.4 || bouncy.includes(t.easing.type)) {
        out.push({ id: n.id, name: n.name, type: t.type, ms: Math.round(t.duration * 1000), easing: t.easing.type })
      }
    }
  }
}
return out.length ? out : 'Không có transition quá 400ms hay có nảy'
```

Còn lại cần người: nhờ người dùng bấm Play, thử hover, nhấn, mở–đóng overlay, và xác nhận không
chỗ nào phải chờ animation xong mới bấm được.
