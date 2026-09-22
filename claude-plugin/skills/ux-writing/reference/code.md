# Code: đọc và thay chữ

Dán nguyên hàm vào `execute_figma_code`, đổi tham số ở dòng cuối. Code đã kiểm kiểu với
`@figma/plugin-typings`.

## `listTexts(id)`: đọc chữ (chỉ đọc)

Trả về tối đa 300 layer chữ đang hiện. Mỗi layer có `id`, tên layer, tên layer cha, chữ, `where`
(`màn hình`, `instance: …` hay `main component: …`), `prop` (tên property TEXT nếu chữ nối vào
component), `maxWidth` (bề rộng cố định; `null` là layer tự giãn theo chữ) và `mixedStyle` (layer có
nhiều kiểu chữ, ví dụ một chữ đậm hay một link màu riêng). Layer nằm trong nhánh đang ẩn bị bỏ qua.

```js
async function listTexts(id) {
  const root = await figma.getNodeByIdAsync(id)
  if (!root || !('findAllWithCriteria' in root) || root.type === 'DOCUMENT') throw new Error('Không thấy frame ' + id)
  const shown = (n) => {
    for (let p = n; p && p !== root; p = p.parent) if ('visible' in p && !p.visible) return false
    return true
  }
  const out = []
  for (const t of root.findAllWithCriteria({ types: ['TEXT'] })) {
    if (!t.characters.trim() || !shown(t)) continue
    let where = 'màn hình'
    for (let p = t.parent; p && p !== root; p = p.parent) {
      if (p.type === 'INSTANCE') { where = 'instance: ' + p.name; break }
      if (p.type === 'COMPONENT') { where = 'main component: ' + p.name; break }
    }
    out.push({
      id: t.id,
      layer: t.name,
      parent: t.parent ? t.parent.name : '',
      text: t.characters,
      where,
      prop: t.componentPropertyReferences ? t.componentPropertyReferences.characters || null : null,
      maxWidth: t.textAutoResize === 'WIDTH_AND_HEIGHT' ? null : Math.round(t.width),
      mixedStyle: t.getStyledTextSegments(['fontName', 'fills', 'fontSize', 'textDecoration']).length > 1,
    })
    if (out.length >= 300) break
  }
  return out
}
return await listTexts('12:34') // id của frame
```

## `applyTexts(edits)`: thay chữ đã được đồng ý

Load mọi font của layer trước khi đổi. Chữ nối vào property TEXT thì đổi qua `setProperties` của
instance. Layer có nhiều kiểu chữ bị bỏ qua (báo lỗi), trừ khi mục đó có `flatten: true`. Mục nào lỗi
thì báo lỗi riêng mục đó, các mục sau vẫn chạy. Một lần gọi là một bước Undo.

```js
async function applyTexts(edits) {
  const done = []
  for (const e of edits) {
    try {
      const n = await figma.getNodeByIdAsync(e.id)
      if (!n || n.type !== 'TEXT') throw new Error('không phải layer chữ')
      const segments = n.getStyledTextSegments(['fontName', 'fills', 'fontSize', 'textDecoration'])
      // Đổi chữ làm cả layer theo kiểu của ký tự đầu: chữ đậm, link màu riêng sẽ mất
      if (segments.length > 1 && !e.flatten) throw new Error('layer có nhiều kiểu chữ; sửa tay, hoặc gửi lại với flatten: true')
      const fonts = segments.length ? segments.map((s) => s.fontName) : [n.fontName]
      await Promise.all(fonts.map((f) => figma.loadFontAsync(f)))
      const prop = n.componentPropertyReferences ? n.componentPropertyReferences.characters : undefined
      let owner = n.parent
      while (owner && owner.type !== 'INSTANCE') owner = owner.parent
      if (prop && owner && owner.type === 'INSTANCE') owner.setProperties({ [prop]: e.text }) // chữ nối vào property TEXT
      else n.characters = e.text
      done.push({ id: e.id, ok: true })
    } catch (err) {
      done.push({ id: e.id, error: err instanceof Error ? err.message : String(err) }) // mục lỗi không chặn các mục sau
    }
  }
  return done
}
return await applyTexts([
  { id: '12:40', text: 'Lưu thay đổi' },
  { id: '12:52', text: 'Chưa có đơn hàng nào. Đơn mới sẽ hiện ở đây.' },
])
```
