# Code: đọc các luồng prototype

Dán nguyên hàm vào `execute_figma_code`. Hàm chỉ đọc, không sửa file. Code đã kiểm kiểu với
`@figma/plugin-typings`.

## `prototypeMap()`

Đọc page hiện tại. Mỗi màn là một frame nằm ngay dưới page hoặc dưới section. Hàm đi theo các tương
tác prototype chuyển sang màn khác hoặc mở overlay trên cùng page. Tương tác chỉ đổi variant (hover,
bấm trong component) hay cuộn tới một chỗ thì không tính. Hàm trả về:

- `flows`: mỗi điểm bắt đầu flow có tên flow, các màn theo thứ tự đi tới (tối đa 60), và `deadEnds`
  là các màn không đi tiếp được và cũng không có nút quay lại.
- `screensWithLinks`: số màn có dính tới prototype.

`flows` rỗng nghĩa là page chưa có điểm bắt đầu flow nào.

```js
async function prototypeMap() {
  const page = figma.currentPage
  const screenOf = (n) => {
    let cur = n
    while (cur.parent && cur.parent.type !== 'PAGE' && cur.parent.type !== 'SECTION') cur = cur.parent
    return cur.type === 'PAGE' || cur.type === 'DOCUMENT' ? null : cur
  }
  const graph = new Map()
  const node = (s) => {
    const g = graph.get(s.id) ?? { name: s.name, to: new Set(), back: false }
    graph.set(s.id, g)
    return g
  }
  const onThisPage = (n) => {
    let p = n
    while (p && p.type !== 'PAGE') p = p.parent
    return p === page
  }
  for (const n of page.findAll((x) => 'reactions' in x && x.reactions.length > 0)) {
    const from = screenOf(n)
    // Tương tác nằm trong main component (đổi variant khi hover, bấm) không phải chuyển màn
    if (!from || from.type === 'COMPONENT' || from.type === 'COMPONENT_SET' || !('reactions' in n)) continue
    const g = node(from)
    for (const r of n.reactions) {
      for (const a of r.actions ?? (r.action ? [r.action] : [])) {
        if (a.type === 'BACK' || a.type === 'CLOSE') g.back = true
        if (a.type === 'NODE' && a.destinationId && a.navigation !== 'CHANGE_TO' && a.navigation !== 'SCROLL_TO') {
          const dest = await figma.getNodeByIdAsync(a.destinationId)
          const to = dest && onThisPage(dest) ? screenOf(dest) : null
          if (to && to.id !== from.id) { g.to.add(to.id); node(to) }
        }
      }
    }
  }
  const flows = []
  for (const start of page.flowStartingPoints) {
    const order = []
    const queue = [start.nodeId]
    while (queue.length && order.length < 60) {
      const id = queue.shift()
      if (order.includes(id)) continue
      order.push(id)
      for (const next of graph.get(id)?.to ?? []) queue.push(next)
    }
    const nameOf = async (id) => graph.get(id)?.name ?? (await figma.getNodeByIdAsync(id))?.name ?? id
    const screens = []
    for (const id of order) screens.push(await nameOf(id))
    const deadEnds = []
    for (const id of order) {
      const g = graph.get(id)
      if (!g || (g.to.size === 0 && !g.back)) deadEnds.push(await nameOf(id))
    }
    flows.push({ flow: start.name, screens, deadEnds })
  }
  return { page: page.name, flows, screensWithLinks: graph.size }
}
return await prototypeMap()
```

Màn cụt không phải lúc nào cũng là lỗi: màn cuối của luồng (đặt hàng xong, gửi thành công) không
cần đi tiếp. Hỏi người dùng nếu không chắc.
