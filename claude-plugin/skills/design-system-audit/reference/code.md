# Code: đọc hệ thiết kế của file

Dán nguyên hàm vào `execute_figma_code`, đổi tham số ở dòng cuối. Code đã kiểm kiểu với
`@figma/plugin-typings`.

## `systemInventory(allPages)`: kê khai cả hệ (chỉ đọc)

Trả về:

- `collections`: mỗi collection biến có tên, các mode, số biến, số biến là alias, tối đa 150 tên biến.
- `sameValue`: các nhóm biến màu cùng collection có cùng mã màu ở mọi mode (không tính alias).
- `textStyles`, `paintStyles`, `effectStyles`: style local.
- `components`: mỗi component set (và component đứng riêng) có page, số variant, các giá trị của
  từng trục variant, các property khác, đã có mô tả chưa.
- `coverage`: độ phủ trên màn hình (bỏ qua phần ruột component và instance): số fill màu đặc và số
  đã gắn biến hoặc style, số layer chữ và số đã có text style, số instance, và `maybeDetached`.

```js
async function systemInventory(allPages) {
  const isAlias = (x) => typeof x === 'object' && 'type' in x && x.type === 'VARIABLE_ALIAS'
  const cols = await figma.variables.getLocalVariableCollectionsAsync()
  const vars = await figma.variables.getLocalVariablesAsync()
  const sameValue = new Map()
  for (const v of vars) {
    // Chỉ so màu có giá trị thật. Alias trỏ cùng một màu gốc là đúng cách, số trùng nhau (space/16 = radius/16) cũng không sao
    if (v.resolvedType !== 'COLOR' || Object.values(v.valuesByMode).some(isAlias)) continue
    const key = v.variableCollectionId + '|' + JSON.stringify(v.valuesByMode)
    sameValue.set(key, [...(sameValue.get(key) ?? []), v.name])
  }
  const collections = cols.map((c) => {
    const own = vars.filter((v) => v.variableCollectionId === c.id)
    return {
      name: c.name,
      modes: c.modes.map((m) => m.name),
      count: own.length,
      aliasCount: own.filter((v) => Object.values(v.valuesByMode).some(isAlias)).length,
      names: own.slice(0, 150).map((v) => v.name),
    }
  })
  const textStyles = (await figma.getLocalTextStylesAsync()).map((s) => `${s.name} · ${s.fontName.family} ${s.fontName.style} ${s.fontSize}`)
  const paintStyles = (await figma.getLocalPaintStylesAsync()).map((s) => s.name)
  const effectStyles = (await figma.getLocalEffectStylesAsync()).map((s) => s.name)

  await figma.loadAllPagesAsync()
  const pageOf = (n) => {
    let p = n
    while (p && p.type !== 'PAGE') p = p.parent
    return p ? p.name : '?'
  }
  const components = []
  for (const s of figma.root.findAllWithCriteria({ types: ['COMPONENT_SET'] })) {
    let variants = {}
    let props = []
    try {
      const defs = Object.entries(s.componentPropertyDefinitions)
      variants = Object.fromEntries(defs.filter(([, d]) => d.type === 'VARIANT').map(([k, d]) => [k, d.variantOptions || []]))
      props = defs.filter(([, d]) => d.type !== 'VARIANT').map(([k, d]) => `${k.split('#')[0]} (${d.type})`)
    } catch (e) {
      props = ['LỖI: set có variant trùng tên hoặc hỏng, Figma không đọc được property']
    }
    components.push({ name: s.name, page: pageOf(s), variantCount: s.children.length, variants, props, hasDescription: !!s.description })
  }
  const loose = figma.root.findAllWithCriteria({ types: ['COMPONENT'] }).filter((c) => !c.parent || c.parent.type !== 'COMPONENT_SET')
  for (const c of loose) components.push({ name: c.name, page: pageOf(c), variantCount: 1, variants: {}, props: [], hasDescription: !!c.description })

  // Độ phủ trên màn hình: bỏ qua phần ruột component và instance, vì chúng đi theo component
  const compNames = new Set(components.map((c) => c.name))
  const cover = { solidFills: 0, boundFills: 0, texts: 0, styledTexts: 0, instances: 0, maybeDetached: [] }
  const pages = allPages ? figma.root.children : [figma.currentPage]
  let seen = 0
  const walk = (n) => {
    if (++seen > 20000) return
    if (n.type === 'COMPONENT' || n.type === 'COMPONENT_SET') return
    if (n.type === 'INSTANCE') { cover.instances++; return }
    if (n.type === 'FRAME' && compNames.has(n.name) && cover.maybeDetached.length < 20) cover.maybeDetached.push(`${n.name} (${n.id})`)
    if ('fills' in n && Array.isArray(n.fills)) {
      for (const p of n.fills) {
        if (p.type !== 'SOLID' || p.visible === false) continue
        cover.solidFills++
        if ((p.boundVariables && p.boundVariables.color) || ('fillStyleId' in n && n.fillStyleId)) cover.boundFills++
      }
    }
    if (n.type === 'TEXT') { cover.texts++; if (typeof n.textStyleId === 'string' && n.textStyleId) cover.styledTexts++ }
    if ('children' in n) for (const c of n.children) walk(c)
  }
  for (const page of pages) for (const c of page.children) walk(c)
  return {
    collections,
    sameValue: [...sameValue.values()].filter((names) => names.length > 1).slice(0, 30),
    textStyles,
    paintStyles,
    effectStyles,
    components,
    coverage: { pages: pages.map((p) => p.name), nodesSeen: Math.min(seen, 20000), stoppedEarly: seen > 20000, ...cover },
  }
}
return await systemInventory(false) // true: đếm độ phủ trên mọi page
```

Kết quả quá dài (bị cắt ở 30.000 ký tự): trả từng phần, ví dụ `return (await systemInventory(false)).components`.

## `documentComponent(id, markdown)`: gắn mô tả vào component

Chỉ chạy khi người dùng đồng ý. Hàm ghi đè mô tả cũ, nên đọc mô tả cũ trước và giữ lại phần còn đúng:
`return (await figma.getNodeByIdAsync('20:1')).descriptionMarkdown` (`inspect_nodes` không trả mô tả).

```js
async function documentComponent(id, markdown) {
  const node = await figma.getNodeByIdAsync(id)
  if (!node || (node.type !== 'COMPONENT_SET' && node.type !== 'COMPONENT')) throw new Error('Không phải component hay component set')
  node.descriptionMarkdown = markdown // hiện ở panel bên phải và trong Dev Mode
  return { id: node.id, name: node.name }
}
return await documentComponent('20:1', [
  '**Dùng khi** cần một hành động chính trên màn. Mỗi màn chỉ một nút Primary.',
  '**Variant**: Kind (Primary, Secondary, Ghost) × Size (S, M, L) × State.',
  '**Trợ năng**: nút chỉ có icon phải có tên đọc lên; vòng focus 2px.',
].join('\n\n'))
```
