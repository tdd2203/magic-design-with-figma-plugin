# Code: đọc số liệu để bàn giao

Dán nguyên hàm vào `execute_figma_code`, đổi id ở dòng cuối. Hàm chỉ đọc, không sửa file. Code đã
kiểm kiểu với `@figma/plugin-typings`.

## `handoffData(id)`

Trả về:

- `tokens`: biến đang gắn trên màn, kèm tên trong code (nếu có), giá trị theo từng mode, và thuộc
  tính dùng nó (`fills`, `itemSpacing`…). Không tính phần ruột của instance.
- `styles`: text style, color style, effect style đang dùng.
- `components`: component nào, bao nhiêu instance, những tổ hợp variant và props đang dùng.
- `interactions`: tương tác prototype (trigger, đi tới đâu, chuyển động, thời lượng, easing).
- `notes`: annotation đã có trên layer.

```js
async function handoffData(id) {
  const root = await figma.getNodeByIdAsync(id)
  if (!root || !('findAll' in root) || root.type === 'PAGE' || root.type === 'DOCUMENT') throw new Error('Không thấy frame ' + id)
  const hex = (c) => '#' + [c.r, c.g, c.b].map((v) => Math.round(v * 255).toString(16).padStart(2, '0')).join('').toUpperCase()
  const insideInstance = (n) => {
    for (let p = n.parent; p && p !== root; p = p.parent) if (p.type === 'INSTANCE') return true
    return false
  }
  const varProps = new Map()
  const styleIds = new Set()
  const comps = new Map()
  const interactions = []
  const notes = []
  for (const n of [root, ...root.findAll(() => true)]) {
    if (insideInstance(n)) continue // phần ruột của component: dev lấy từ component, không cần đặc tả lại
    for (const [prop, bound] of Object.entries(n.boundVariables ?? {})) {
      for (const a of Array.isArray(bound) ? bound : [bound]) {
        if (a && typeof a === 'object' && 'id' in a && typeof a.id === 'string') {
          varProps.set(a.id, (varProps.get(a.id) ?? new Set()).add(prop))
        }
      }
    }
    if (n.type === 'TEXT' && typeof n.textStyleId === 'string' && n.textStyleId) styleIds.add(n.textStyleId)
    if ('fillStyleId' in n && typeof n.fillStyleId === 'string' && n.fillStyleId) styleIds.add(n.fillStyleId)
    if ('effectStyleId' in n && n.effectStyleId) styleIds.add(n.effectStyleId)
    if (n.type === 'INSTANCE') {
      const main = await n.getMainComponentAsync()
      const owner = main && main.parent && main.parent.type === 'COMPONENT_SET' ? main.parent : main
      const name = owner ? owner.name : '(mất main component)'
      const entry = comps.get(name) ?? { count: 0, variants: new Set() }
      entry.count++
      const props = Object.entries(n.componentProperties).map(([k, v]) => `${k.split('#')[0]}=${v.value}`)
      if (entry.variants.size < 8) entry.variants.add(props.join(', '))
      comps.set(name, entry)
    }
    if ('reactions' in n) {
      for (const r of n.reactions) {
        for (const a of r.actions ?? (r.action ? [r.action] : [])) {
          let what = a.type
          if (a.type === 'NODE') {
            const dest = a.destinationId ? await figma.getNodeByIdAsync(a.destinationId) : null
            const t = a.transition
            what = `${a.navigation} → ${dest ? dest.name : '?'}` +
              (t ? ` (${t.type} ${Math.round(t.duration * 1000)}ms ${t.easing.type})` : ' (không chuyển động)')
          } else if (a.type === 'URL') what = `mở link ${a.url}`
          if (interactions.length < 40) interactions.push(`${n.name}: ${r.trigger ? r.trigger.type : '?'} · ${what}`)
        }
      }
    }
    if ('annotations' in n) {
      for (const a of n.annotations) if (notes.length < 40) notes.push(`${n.name}: ${a.labelMarkdown || a.label || '(chỉ có thuộc tính đo)'}`)
    }
  }
  const tokens = []
  for (const [vid, props] of varProps) {
    const v = await figma.variables.getVariableByIdAsync(vid)
    if (!v) continue
    const col = await figma.variables.getVariableCollectionByIdAsync(v.variableCollectionId)
    const values = {}
    for (const m of col ? col.modes : []) {
      const val = v.valuesByMode[m.modeId]
      if (val === undefined) continue
      if (typeof val === 'object' && 'type' in val && val.type === 'VARIABLE_ALIAS') {
        const target = await figma.variables.getVariableByIdAsync(val.id)
        values[m.name] = '→ ' + (target ? target.name : val.id)
      } else if (typeof val === 'object' && 'r' in val) values[m.name] = hex(val)
      else values[m.name] = typeof val === 'object' ? JSON.stringify(val) : String(val)
    }
    tokens.push({ name: v.name, code: v.codeSyntax.WEB || null, values, usedFor: [...props] })
  }
  const styles = []
  for (const sid of styleIds) {
    const s = await figma.getStyleByIdAsync(sid)
    if (!s) continue
    if (s.type === 'TEXT') {
      const lh = s.lineHeight.unit === 'AUTO' ? 'auto' : s.lineHeight.unit === 'PIXELS' ? String(s.lineHeight.value) : s.lineHeight.value + '%'
      styles.push({ name: s.name, value: `${s.fontName.family} ${s.fontName.style} ${s.fontSize}/${lh}` })
    } else styles.push({ name: s.name, value: s.type })
  }
  return {
    frame: { id: root.id, name: root.name, size: 'width' in root ? `${Math.round(root.width)}×${Math.round(root.height)}` : null },
    tokens,
    styles,
    components: [...comps].map(([name, c]) => ({ name, count: c.count, variants: [...c.variants] })),
    interactions,
    notes,
  }
}
return await handoffData('12:34') // id của frame cần bàn giao
```

Màn rất lớn (kết quả bị cắt ở 30.000 ký tự): chạy riêng cho từng section của màn.
