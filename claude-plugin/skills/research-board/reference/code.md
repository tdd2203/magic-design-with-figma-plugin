# Code: đọc và bày sticky trên FigJam

Dán nguyên hàm vào `execute_figma_code`, đổi tham số ở dòng cuối. Code đã kiểm kiểu với
`@figma/plugin-typings`.

## `readStickies()`: đọc sticky có sẵn (chỉ đọc)

Trả về tối đa 500 sticky của page hiện tại. Mỗi sticky có `id`, chữ, tên người tạo, và section chứa
nó (nếu có). File Figma thiết kế không có sticky nên kết quả là mảng rỗng.

```js
function readStickies() {
  return figma.currentPage.findAllWithCriteria({ types: ['STICKY'] }).slice(0, 500).map((s) => ({
    id: s.id,
    text: s.text.characters,
    author: s.authorName,
    section: s.parent && s.parent.type === 'SECTION' ? s.parent.name : null,
  }))
}
return readStickies()
```

## `buildBoard(title, themes)`: bày kết quả lên board

Chỉ chạy khi người dùng đồng ý, và chỉ chạy trên FigJam. Mỗi chủ đề là một section: sticky insight
to ở đầu, các quan sát xếp 3 cột bên dưới, mỗi chủ đề một màu. Board được đặt bên phải nội dung có
sẵn, rồi màn hình cuộn tới đó. `title` rỗng thì không tạo sticky tiêu đề.

```js
async function buildBoard(title, themes) {
  if (figma.editorType !== 'figjam') throw new Error('Cần file FigJam: mở một board FigJam, bật plugin rồi chạy lại')
  const page = figma.currentPage
  const STICKY = 240, GAP = 24, COLS = 3, PAD = 40
  const colors = [
    { r: 1, g: 0.85, b: 0.4 }, { r: 0.56, g: 0.85, b: 0.63 }, { r: 0.55, g: 0.76, b: 1 },
    { r: 1, g: 0.66, b: 0.62 }, { r: 0.8, g: 0.68, b: 1 }, { r: 1, g: 0.75, b: 0.5 },
  ]
  const sticky = async (text, color, wide) => {
    const s = figma.createSticky()
    await figma.loadFontAsync(s.text.fontName)
    s.text.characters = text
    s.isWideWidth = wide
    if (color) s.fills = [{ type: 'SOLID', color }]
    return s
  }
  // Đặt board ở chỗ trống bên phải nội dung có sẵn
  let x = page.children.reduce((max, n) => Math.max(max, n.x + n.width), 0) + 200
  const top = page.children.length ? Math.min(...page.children.map((n) => n.y)) : 0
  const made = []
  const y = top + STICKY + 80 // chừa chỗ sticky tiêu đề, để các lần gọi sau thẳng hàng với lần đầu
  if (title) { // chia board thành nhiều lần gọi thì chỉ lần đầu có tiêu đề
    const heading = await sticky(title, null, true)
    heading.x = x
    heading.y = top
    made.push(heading.id)
  }
  for (const [i, theme] of themes.entries()) {
    const section = figma.createSection()
    section.name = theme.name
    const rows = Math.ceil(theme.notes.length / COLS) + 1
    const width = PAD * 2 + COLS * STICKY + (COLS - 1) * GAP
    const height = PAD * 2 + rows * STICKY + (rows - 1) * GAP + 40
    section.x = x
    section.y = y
    section.resizeWithoutConstraints(width, height)
    const insight = await sticky('Insight: ' + theme.insight, null, true)
    section.appendChild(insight)
    insight.x = PAD
    insight.y = PAD + 40
    for (const [j, note] of theme.notes.entries()) {
      const s = await sticky(note, colors[i % colors.length], false)
      section.appendChild(s)
      s.x = PAD + (j % COLS) * (STICKY + GAP)
      s.y = PAD + 40 + STICKY + GAP + Math.floor(j / COLS) * (STICKY + GAP)
    }
    made.push(section.id)
    x += width + 80
  }
  const nodes = []
  for (const id of made) { const n = await figma.getNodeByIdAsync(id); if (n && n.type !== 'PAGE' && n.type !== 'DOCUMENT') nodes.push(n) }
  figma.viewport.scrollAndZoomIntoView(nodes)
  return { ids: made }
}
return await buildBoard('Tổng hợp test đặt hàng · 6 người · 12–14/9', [
  {
    name: '1. Không biết đơn đã gửi chưa (5/6)',
    insight: 'Sau khi bấm Đặt hàng, người dùng cần thấy ngay đơn đã được nhận',
    notes: ['"Tôi bấm rồi mà không thấy gì hết" (P2)', 'P4 bấm Đặt hàng hai lần', 'P5 mở email để kiểm tra'],
  },
])
```

Sau khi dựng, `screenshot` một section để xem chữ có vừa sticky không. Câu quá dài thì FigJam tự thu
nhỏ chữ; nên rút câu lại thay vì để chữ bé.
