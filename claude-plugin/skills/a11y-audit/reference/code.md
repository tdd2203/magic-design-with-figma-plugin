# Code: tìm chỗ cần tên và ghi chú trợ năng

Dán nguyên hàm vào `execute_figma_code`, đổi tham số ở dòng cuối. Code đã kiểm kiểu với
`@figma/plugin-typings`.

## `a11yTargets(id)`: tìm chỗ cần tên (chỉ đọc)

Trả về tối đa 150 mục, chia ba nhóm:

- `images`: layer có ảnh. Cần alt, hoặc ghi là ảnh trang trí.
- `iconOnly`: nút, link, tab… không có chữ nào bên trong. Cần tên đọc lên.
- `inputs`: ô nhập, kèm tối đa 3 dòng chữ bên trong. Chữ đó là nhãn hay chỉ là placeholder thì xem
  `screenshot` mới biết.

Mỗi mục kèm `notes`, là các annotation đã có, để khỏi ghi chú trùng. Nhánh đang ẩn bị bỏ qua. Nút
và ô nhập được nhận ra qua tên layer, giống cách `audit_design` làm.

```js
async function a11yTargets(id) {
  const root = await figma.getNodeByIdAsync(id)
  if (!root || !('findAll' in root) || root.type === 'PAGE' || root.type === 'DOCUMENT') throw new Error('Không thấy frame ' + id)
  // Cùng cách nhận ra nút và ô nhập theo tên layer như audit_design
  const TARGET = /\b(button|btn|link|icon[- ]?button|tab|chip|toggle|switch|checkbox|radio|close|cta|nut)\b|nút/i
  const INPUT = /\b(input|text ?field|field|text ?box|text ?area|search ?(bar|field|box)|select|dropdown|combo ?box|o nhap)\b|ô nhập/i
  const shown = (n, top) => {
    for (let p = n; p && p !== top; p = p.parent) if ('visible' in p && !p.visible) return false
    return true
  }
  const textOf = (n) =>
    'findAllWithCriteria' in n
      ? n.findAllWithCriteria({ types: ['TEXT'] }).filter((t) => shown(t, n)).map((t) => t.characters.trim()).filter(Boolean)
      : []
  const noteOf = (n) => ('annotations' in n ? n.annotations.map((a) => a.labelMarkdown || a.label || '').filter(Boolean) : [])
  const images = [], iconOnly = [], inputs = []
  // Đi từ trên xuống, bỏ nhánh đang ẩn. Một nút thật thì không đi sâu thêm (icon bên trong không đếm lại);
  // khung tên giống nút nhưng chứa nút khác bên trong (Tab bar, Button group) thì đi tiếp vào trong.
  const walk = (n, inInput) => {
    if (!n.visible || images.length + iconOnly.length + inputs.length >= 150) return
    if ('fills' in n && Array.isArray(n.fills) && n.fills.some((p) => p.type === 'IMAGE' && p.visible !== false)) {
      images.push({ id: n.id, name: n.name, notes: noteOf(n) })
    }
    if (n.type === 'TEXT') return
    if (INPUT.test(n.name)) {
      if (!inInput) inputs.push({ id: n.id, name: n.name, texts: textOf(n).slice(0, 3), notes: noteOf(n) })
      inInput = true // vẫn đi vào trong để tìm nút xoá, nút hiện mật khẩu…
    } else if (TARGET.test(n.name)) {
      const holdsTargets = 'findOne' in n && !!n.findOne((c) => c.type !== 'TEXT' && TARGET.test(c.name) && shown(c, n))
      if (!holdsTargets) {
        if (textOf(n).length === 0) iconOnly.push({ id: n.id, name: n.name, notes: noteOf(n) })
        return
      }
    }
    if ('children' in n) for (const c of n.children) walk(c, inInput)
  }
  for (const c of root.children) walk(c, false)
  return { images, iconOnly, inputs }
}
return await a11yTargets('12:34') // id của frame
```

## `addA11yNotes(notes)`: ghi chú trợ năng lên layer

Chỉ chạy khi người dùng đồng ý. Ghi chú nằm trong category "Tiếp cận", dev đọc được trong Dev Mode.
Hàm thêm ghi chú vào sau các ghi chú cũ, không xoá của người khác. Viết mỗi ghi chú một ý, bắt đầu
bằng loại:

- `**Focus 3**: sau ô Mật khẩu, trước nút Đăng nhập`
- `**Tên đọc lên**: "Đóng hộp thoại"`
- `**Alt**: "Biểu đồ doanh thu 6 tháng, tăng đều từ 120 lên 180 triệu"`
- `**Alt**: trang trí, trình đọc màn hình bỏ qua`
- `**Bàn phím**: Esc đóng hộp thoại, focus quay về nút đã mở nó`

```js
async function addA11yNotes(notes) {
  const categories = await figma.annotations.getAnnotationCategoriesAsync()
  const category =
    categories.find((c) => c.label === 'Tiếp cận') ??
    (await figma.annotations.addAnnotationCategoryAsync({ label: 'Tiếp cận', color: 'green' }))
  // Ghi chú cũ đọc ra có thể có cả label lẫn labelMarkdown; khi gán lại chỉ giữ một
  const keep = (a) => {
    const { label, labelMarkdown, ...rest } = a
    return labelMarkdown ? { ...rest, labelMarkdown } : label ? { ...rest, label } : rest
  }
  const result = []
  for (const note of notes) {
    try {
      const n = await figma.getNodeByIdAsync(note.id)
      if (!n || !('annotations' in n)) throw new Error('không gắn ghi chú được lên layer này')
      n.annotations = [...n.annotations.map(keep), { labelMarkdown: note.text, categoryId: category.id }] // giữ ghi chú cũ
      result.push({ id: note.id, ok: true })
    } catch (err) {
      result.push({ id: note.id, error: err instanceof Error ? err.message : String(err) }) // mục lỗi không chặn các mục sau
    }
  }
  return result
}
return await addA11yNotes([
  { id: '12:40', text: '**Tên đọc lên**: "Đóng hộp thoại"' },
  { id: '12:41', text: '**Focus 1**: ô Email' },
])
```

Layer không nhận annotation (ví dụ layer nằm sâu trong instance) thì hàm báo lỗi cho mục đó. Khi đó
gắn ghi chú lên instance chứa nó, hoặc đặt khung ghi chú cạnh màn hình như
[../../design-workflow/reference/handoff.md](../../design-workflow/reference/handoff.md) mục 3.
