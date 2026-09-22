# Công thức Figma Plugin API

Các công thức được viết bằng TypeScript, kiểm kiểu với `@figma/plugin-typings` (để không gọi sai tên API),
rồi chuyển sang **JavaScript dán được ngay** vào `execute_figma_code`. Cách dùng: dán hàm (và các hàm phụ
nó gọi) vào code, rồi gọi nó ở dòng cuối. Hai điều cần nhớ:

1. **Mỗi lần gọi là một phạm vi riêng.** Hàm phụ (`hex`, `solid`, `stack`, `text`, `freeSpotOnPage`)
   phải dán lại trong đúng lần gọi cần nó, rồi gọi hàm và trả kết quả gọn: `return await loginScreen()`.
2. **Giá trị và tên trong ví dụ chỉ để minh hoạ API** (Inter, `#2563EB`, bo góc 10 và 16, chữ 15px,
   tiêu đề 28 Bold, padding nút 20/12, chữ tiếng Anh, variant `Variant=Primary`, biến `color/surface` và
   `space/md`). Việc thật lấy variables, styles, components của file; file chưa có thì theo mặc định của
   register trong [../SKILL.md](../SKILL.md) và các skill `color-system`, `layout-type`,
   `components-states` (variant `Kind=Primary, Size=M, State=Default`, biến `color/bg/surface`, `space/16`).

| # | Công thức | Hàm |
|---|---|---|
| 1 | Đọc hệ thiết kế của file trước khi vẽ | `readDesignSystem` |
| 2 | Tìm chỗ trống bên phải nội dung có sẵn | `freeSpotOnPage` |
| 3 | Màu: hex → RGB 0–1 của Figma | `hex`, `solid` |
| 4 | Khung auto layout | `stack` |
| 5 | Chữ, có load font (và lùi về font dự phòng khi thiếu) | `text` |
| 6 | Sửa chữ có sẵn: load mọi font nó dùng trước | `editText` |
| 7 | Khung màn hình đặt ở chỗ trống, thẻ bên trong giãn hết bề ngang | `loginScreen` |
| 8 | Component set nút: variants và thuộc tính chữ | `buttonComponent` |
| 9 | Dùng component: instance và ghi đè thuộc tính | `useButton` |
| 10 | Variables: token màu hai mode Light/Dark gắn vào fill, token khoảng cách gắn vào gap | `tokens` |
| 11 | Styles: tạo text style rồi áp (setter async vì file dùng dynamic page loading) | `textStyle` |
| 12 | Lưới thẻ tự xuống dòng và fill gradient tuyến tính | `cardGrid` |
| 13 | Min/max width trong auto layout, badge định vị tuyệt đối | `responsiveBits` |
| 14 | Font có trên máy (chọn họ font có thật trước khi thiết kế) | `fonts` |
| 15 | Auto layout dạng lưới (layoutMode 'GRID'): cột cố định, khoảng cách, đặt con theo ô | `dashboardGrid` |

---

## 1. Đọc hệ thiết kế của file trước khi vẽ

Gọi đầu tiên trong mọi việc thiết kế; file lớn thì lọc theo tiền tố tên trước khi `return` để không vượt 30.000 ký tự.

```js
async function readDesignSystem() {
  const [paintStyles, textStyles, effectStyles] = await Promise.all([
    figma.getLocalPaintStylesAsync(),
    figma.getLocalTextStylesAsync(),
    figma.getLocalEffectStylesAsync(),
  ]);
  const collections = await figma.variables.getLocalVariableCollectionsAsync();
  const variables = await figma.variables.getLocalVariablesAsync();
  // Chỉ lấy component trên page hiện tại; muốn tìm mọi page thì gọi figma.loadAllPagesAsync() trước.
  const components = figma.currentPage.findAllWithCriteria({ types: ['COMPONENT', 'COMPONENT_SET'] });
  return {
    paintStyles: paintStyles.map((s) => ({ id: s.id, name: s.name })),
    textStyles: textStyles.map((s) => ({ id: s.id, name: s.name, font: s.fontName, size: s.fontSize })),
    effectStyles: effectStyles.map((s) => ({ id: s.id, name: s.name })),
    collections: collections.map((c) => ({ id: c.id, name: c.name, modes: c.modes.map((m) => m.name) })),
    variables: variables.map((v) => ({ id: v.id, name: v.name, type: v.resolvedType })),
    components: components.map((c) => ({ id: c.id, name: c.name, type: c.type })),
  };
}
```

---

## 2. Tìm chỗ trống bên phải nội dung có sẵn

Trả toạ độ để đặt frame mới không đè lên việc của người dùng; chỉ xét các con trực tiếp của page hiện tại.

```js
function freeSpotOnPage(gap = 120) {
  const nodes = figma.currentPage.children;
  if (!nodes.length)
    return { x: 0, y: 0 };
  const right = Math.max(...nodes.map((n) => n.x + n.width));
  const top = Math.min(...nodes.map((n) => n.y));
  return { x: Math.round(right + gap), y: Math.round(top) };
}
```

---

## 3. Màu: hex → RGB 0–1 của Figma

Hàm phụ cho các công thức khác; màu dùng thật nên gắn variable (công thức 10) thay vì để hex trần.

```js
function hex(value) {
  const v = value.replace('#', '');
  const n = parseInt(v.length === 3 ? v.split('').map((c) => c + c).join('') : v, 16);
  return { r: ((n >> 16) & 255) / 255, g: ((n >> 8) & 255) / 255, b: (n & 255) / 255 };
}
const solid = (value, opacity = 1) => ({ type: 'SOLID', color: hex(value), opacity });
```

---

## 4. Khung auto layout

Khung ôm nội dung; `padding` là một số hoặc mảng [trên, phải, dưới, trái]; gap và padding giữ bội số 4.

```js
function stack(name, direction, gap, padding = 0) {
  const frame = figma.createFrame();
  frame.name = name;
  frame.layoutMode = direction;
  frame.itemSpacing = gap;
  const [t, r, b, l] = typeof padding === 'number' ? [padding, padding, padding, padding] : padding;
  frame.paddingTop = t;
  frame.paddingRight = r;
  frame.paddingBottom = b;
  frame.paddingLeft = l;
  frame.primaryAxisSizingMode = 'AUTO'; // ôm nội dung theo hướng xếp
  frame.counterAxisSizingMode = 'AUTO';
  frame.fills = []; // trong suốt, trừ khi khung này là một bề mặt
  return frame;
}
```

---

## 5. Chữ, có load font (và lùi về font dự phòng khi thiếu)

File có text style thì áp style (công thức 11) thay vì đặt cỡ và line height tay như ở đây.

```js
async function text(characters, size, style = 'Regular', family = 'Inter', color = '#111827') {
  let fontName = { family, style };
  try {
    await figma.loadFontAsync(fontName);
  }
  catch {
    fontName = { family: 'Inter', style: 'Regular' };
    await figma.loadFontAsync(fontName);
  }
  const node = figma.createText();
  node.fontName = fontName;
  node.fontSize = size;
  node.characters = characters;
  node.lineHeight = { unit: 'PERCENT', value: size >= 24 ? 120 : 150 };
  node.fills = [solid(color)];
  return node;
}
```

---

## 6. Sửa chữ có sẵn: load mọi font nó dùng trước

Dùng cho mọi layer chữ có sẵn, kể cả layer trộn nhiều font; thiếu bước này thì gán `characters` sẽ lỗi.

```js
async function editText(node, characters) {
  const fonts = node.getStyledTextSegments(['fontName']).map((s) => s.fontName);
  await Promise.all(fonts.map((f) => figma.loadFontAsync(f)));
  node.characters = characters;
}
```

---

## 7. Khung màn hình đặt ở chỗ trống, thẻ bên trong giãn hết bề ngang

Ví dụ ghép các hàm phụ; dán kèm `hex`, `solid`, `stack`, `text`, `freeSpotOnPage` trong cùng lần gọi.

```js
async function loginScreen() {
  const spot = freeSpotOnPage();
  const screen = stack('Login', 'VERTICAL', 24, [64, 24, 32, 24]);
  screen.resize(390, 844);
  screen.primaryAxisSizingMode = 'FIXED';
  screen.counterAxisSizingMode = 'FIXED';
  screen.fills = [solid('#F9FAFB')];
  screen.x = spot.x;
  screen.y = spot.y;
  const title = await text('Welcome back', 28, 'Bold');
  screen.appendChild(title);
  const card = stack('Card', 'VERTICAL', 16, 24);
  card.fills = [solid('#FFFFFF')];
  card.cornerRadius = 16;
  card.effects = [
    { type: 'DROP_SHADOW', color: { r: 0, g: 0, b: 0, a: 0.08 }, offset: { x: 0, y: 4 }, radius: 16, spread: 0, visible: true, blendMode: 'NORMAL' },
  ];
  screen.appendChild(card);
  card.layoutSizingHorizontal = 'FILL'; // chỉ hợp lệ sau khi đã append vào cha có auto layout
  figma.currentPage.selection = [screen];
  figma.viewport.scrollAndZoomIntoView([screen]);
  return { id: screen.id };
}
```

---

## 8. Component set nút: variants và thuộc tính chữ

Mẫu tạo variant và TEXT property; bộ trạng thái đầy đủ của nút xem skill `components-states`.

```js
async function buttonComponent() {
  const make = async (variant, bg, fg) => {
    const c = figma.createComponent();
    c.name = `Variant=${variant}`;
    c.layoutMode = 'HORIZONTAL';
    c.primaryAxisSizingMode = 'AUTO';
    c.counterAxisSizingMode = 'AUTO';
    c.primaryAxisAlignItems = 'CENTER';
    c.counterAxisAlignItems = 'CENTER';
    c.paddingLeft = c.paddingRight = 20;
    c.paddingTop = c.paddingBottom = 12;
    c.cornerRadius = 10;
    c.fills = [solid(bg)];
    const label = await text('Button', 15, 'Semi Bold', 'Inter', fg);
    c.appendChild(label);
    return c;
  };
  const variants = [await make('Primary', '#2563EB', '#FFFFFF'), await make('Secondary', '#E5E7EB', '#111827')];
  const set = figma.combineAsVariants(variants, figma.currentPage);
  set.name = 'Button';
  set.layoutMode = 'HORIZONTAL';
  set.itemSpacing = 16;
  set.paddingLeft = set.paddingRight = set.paddingTop = set.paddingBottom = 16;
  // Đưa nhãn thành component property dùng chung cho mọi variant
  const key = set.addComponentProperty('Label', 'TEXT', 'Button');
  for (const variant of set.children) {
    const label = variant.findOne((n) => n.type === 'TEXT');
    label.componentPropertyReferences = { characters: key };
  }
  return { id: set.id, labelKey: key };
}
```

---

## 9. Dùng component: instance và ghi đè thuộc tính

`labelKey` là key trả về từ công thức 8 (có dạng `Label#…`); tên variant phải khớp đúng chuỗi `Variant=Primary`.

```js
async function useButton(setId, labelKey, parent) {
  const set = (await figma.getNodeByIdAsync(setId));
  const primary = set.children.find((c) => c.name === 'Variant=Primary');
  const instance = primary.createInstance();
  parent.appendChild(instance);
  instance.setProperties({ [labelKey]: 'Sign in' });
  instance.layoutSizingHorizontal = 'FILL';
  return instance.id;
}
```

---

## 10. Variables: token màu hai mode Light/Dark gắn vào fill, token khoảng cách gắn vào gap

Chỉ minh hoạ API; cách chia collection (Primitives, Semantic) và đặt tên theo vai trò xem skill `color-system`.

```js
async function tokens(frame) {
  const collection = figma.variables.createVariableCollection('Tokens');
  const light = collection.modes[0].modeId;
  collection.renameMode(light, 'Light');
  const dark = collection.addMode('Dark');
  const surface = figma.variables.createVariable('color/surface', collection, 'COLOR');
  surface.setValueForMode(light, { r: 1, g: 1, b: 1, a: 1 });
  surface.setValueForMode(dark, { r: 0.07, g: 0.09, b: 0.15, a: 1 });
  const gap = figma.variables.createVariable('space/md', collection, 'FLOAT');
  gap.setValueForMode(light, 16);
  gap.setValueForMode(dark, 16);
  const paint = figma.variables.setBoundVariableForPaint(solid('#FFFFFF'), 'color', surface);
  frame.fills = [paint];
  frame.setBoundVariable('itemSpacing', gap);
  frame.setExplicitVariableModeForCollection(collection, dark); // xem thử mode Dark trên riêng frame này
  return { collection: collection.id };
}
```

---

## 11. Styles: tạo text style rồi áp (setter async vì file dùng dynamic page loading)

Load font của style trước khi tạo hay áp; dùng `setTextStyleIdAsync`, `setFillStyleIdAsync`, không gán thẳng `textStyleId`.

```js
async function textStyle(node) {
  await figma.loadFontAsync({ family: 'Inter', style: 'Bold' });
  const style = figma.createTextStyle();
  style.name = 'Heading/H1';
  style.fontName = { family: 'Inter', style: 'Bold' };
  style.fontSize = 32;
  style.lineHeight = { unit: 'PIXELS', value: 40 };
  await node.setTextStyleIdAsync(style.id);
  const fill = figma.createPaintStyle();
  fill.name = 'Brand/Primary';
  fill.paints = [solid('#2563EB')];
  await node.setFillStyleIdAsync(fill.id);
}
```

---

## 12. Lưới thẻ tự xuống dòng và fill gradient tuyến tính

`layoutWrap = 'WRAP'` chỉ chạy với khung HORIZONTAL; `counterAxisSpacing` là khoảng giữa các hàng.

```js
function cardGrid(count) {
  const grid = stack('Grid', 'HORIZONTAL', 16, 0);
  grid.layoutWrap = 'WRAP';
  grid.counterAxisSpacing = 16;
  grid.resize(1200, 100);
  grid.primaryAxisSizingMode = 'FIXED';
  for (let i = 0; i < count; i++) {
    const card = figma.createFrame();
    card.name = `Card ${i + 1}`;
    card.resize(376, 240);
    card.cornerRadius = 12;
    card.fills = [
      {
        type: 'GRADIENT_LINEAR',
        gradientTransform: [
          [1, 0, 0],
          [0, 1, 0],
        ],
        gradientStops: [
          { position: 0, color: { r: 0.93, g: 0.95, b: 1, a: 1 } },
          { position: 1, color: { r: 0.85, g: 0.89, b: 1, a: 1 } },
        ],
      },
    ];
    grid.appendChild(card);
  }
  return grid.id;
}
```

---

## 13. Min/max width trong auto layout, badge định vị tuyệt đối

`frame` phải là khung auto layout; badge `ABSOLUTE` thoát khỏi luồng xếp và bám góc phải trên nhờ `constraints`.

```js
function responsiveBits(frame, badge) {
  frame.minWidth = 320;
  frame.maxWidth = 640;
  frame.appendChild(badge);
  badge.layoutPositioning = 'ABSOLUTE';
  badge.constraints = { horizontal: 'MAX', vertical: 'MIN' };
  badge.x = frame.width - badge.width - 8;
  badge.y = 8;
}
```

---

## 14. Font có trên máy (chọn họ font có thật trước khi thiết kế)

Chọn xong thì chụp thử chữ "ĐẶNG THỊ NGỌC ẨN" để chắc font có đủ dấu tiếng Việt.

```js
async function fonts() {
  const all = await figma.listAvailableFontsAsync();
  const families = [...new Set(all.map((f) => f.fontName.family))];
  return families.slice(0, 50);
}
```

---

## 15. Auto layout dạng lưới (layoutMode 'GRID'): cột cố định, khoảng cách, đặt con theo ô

Hợp cho dashboard; `appendChildAt(node, hàng, cột)` đặt từng thẻ vào đúng ô.

```js
function dashboardGrid(cards) {
  const grid = figma.createFrame();
  grid.name = 'Dashboard grid';
  grid.layoutMode = 'GRID';
  grid.gridColumnCount = 3;
  grid.gridRowCount = Math.max(1, Math.ceil(cards.length / 3));
  grid.gridColumnGap = 16;
  grid.gridRowGap = 16;
  grid.gridColumnSizes[0].type = 'FIXED'; // cột đầu cố định, các cột còn lại chia phần còn lại (FLEX)
  grid.gridColumnSizes[0].value = 280;
  grid.resize(1200, 600);
  cards.forEach((card, i) => grid.appendChildAt(card, Math.floor(i / 3), i % 3));
  return grid.id;
}
```
