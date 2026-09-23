# Magic Design with Figma

Thiết kế trong Figma bằng Claude Code: bạn chat với Claude trong Claude Code (đăng nhập tài khoản Claude, **không cần API key**), còn Claude đọc file, dựng layer và tự chụp ảnh kiểm tra ngay trên canvas.

Repo này chỉ chứa bản đã build để cài đặt. Gồm hai plugin nói chuyện với nhau qua `localhost`:

- **Plugin Claude Code** (`claude-plugin/`): một MCP server nhỏ mà Claude Code tự khởi động, cung cấp tool Figma và các skill thiết kế cho Claude.
- **Plugin Figma** (`manifest.json`, `dist/`): panel trong Figma: nút "Kết nối", chọn thư mục dự án cho Claude, và nhật ký các bước Claude làm trong đoạn chat đang chạy.

## Cần có

- [Claude Code](https://claude.com/claude-code), đăng nhập tài khoản Claude
- [Figma Desktop](https://www.figma.com/downloads/) (plugin phát triển không chạy trên bản trình duyệt)
- [Node.js](https://nodejs.org) 18 trở lên

## Cài đặt

1. **Plugin Claude Code.** Cài một lần, theo một trong hai cách:

   - **App Claude:** vào **Customize → Plugins**, bấm **+** → **Add marketplace**, dán `tdd2203/magic-design-with-figma-plugin`, rồi cài **Magic Design with Figma**.
   - **Terminal:**

     ```bash
     claude plugin marketplace add tdd2203/magic-design-with-figma-plugin
     claude plugin install magic-design-with-figma@magic-design-with-figma
     ```

     Đã thêm marketplace trong app thì chỉ cần chạy lệnh thứ hai.

   Tab Code của app Claude không chạy lệnh `/plugin` (báo "isn't available in this environment"), nên trong app hãy cài qua Customize.

2. **Plugin Figma.** Tải repo này về máy: bấm **Code → Download ZIP** rồi giải nén vào một thư mục cố định, hoặc:

   ```bash
   git clone https://github.com/tdd2203/magic-design-with-figma-plugin.git
   ```

   Trong Figma Desktop: **Plugins → Development → Import plugin from manifest…**, chọn file `manifest.json` trong thư mục vừa tải.

3. Mở một phiên Claude Code mới trong thư mục dự án (trong app Claude, hoặc chạy `claude` trong Terminal), mở plugin **Magic Design with Figma** trong Figma rồi bấm **Kết nối**.
4. **Ghép cặp (một lần mỗi máy):** panel hiện một mã 6 số. Gõ `/magic-design-with-figma:pair 482913` trong Claude Code (nút "Copy lệnh" chép yêu cầu ghép cặp dùng được cho cả hai ứng dụng), hoặc chỉ cần nói mã với Claude.
5. **Chọn thư mục:** panel liệt kê các thư mục đang mở Claude Code, thư mục có khả năng đúng nhất nằm đầu kèm nhãn gợi ý. Bấm một thư mục là xong; lần sau mở lại file này, thư mục đó có nhãn **Dự án đang làm** (nhãn **Giống tên file** chỉ là đoán theo tên); mở nhiều phiên Claude Code trong cùng thư mục thì phiên nào ở đó cũng sửa được.
6. Nhờ Claude, ví dụ: "Thiết kế màn hình đăng nhập mobile trong Figma".

Claude Code sẽ hỏi quyền ở lần đầu mỗi tool được gọi. Chọn "always allow" nếu không muốn bị hỏi lại.

## Dùng với Codex

Codex dùng chung MCP server và plugin Figma với Claude Code. Đăng ký một lần trong Terminal
(thay đường dẫn bằng nơi bạn lưu repo và thư mục bạn muốn thiết kế):

```bash
codex mcp add magic-design-figma --env MAGIC_DESIGN_CLIENT=codex --env 'MAGIC_DESIGN_PROJECT_DIR=/duong/dan/du-an' -- node /duong/dan/repo/claude-plugin/server/index.cjs
```

Cấu hình này cố định thư mục dự án của kết nối; đăng ký lại khi muốn đổi thư mục.
Sau khi cập nhật, mở lại plugin Figma và khởi động lại các phiên Claude/Codex đang dùng cầu nối
để tất cả nạp bản mới. Mở lại Codex để nạp MCP vừa đăng ký.
Trong Figma, bấm **Kết nối**, ghép cặp bằng cách gửi “Ghép cặp Figma bằng mã 123456”
cho AI nếu panel yêu cầu, rồi chọn **Codex · tên thư mục**. Sau đó giao yêu cầu thiết kế ngay trong Codex.

Danh sách phân biệt **Claude Code** và **Codex**, kể cả khi cùng mở một thư mục.
Chỉ kết nối được chọn có thể sửa Figma; **Đổi kết nối** để chuyển ứng dụng.
Khi kết nối đóng, ứng dụng khác cùng thư mục không tự nhận quyền chỉnh sửa.
Các phiên cũ chưa có nhãn được coi là Claude Code.

Ví dụ: “Đọc file Figma đang mở, thiết kế màn hình đăng nhập mobile 390×844,
chụp ảnh và kiểm tra thiết kế trước khi báo hoàn tất.”
Các hướng dẫn thiết kế dùng chung nằm trong `claude-plugin/skills/`; MCP cũng gửi
hướng dẫn thao tác Figma cho cả hai ứng dụng.

## Cập nhật

- **Plugin Claude Code:** mặc định Claude Code không tự cập nhật plugin từ marketplace bên thứ ba. Cập nhật bằng một trong hai cách, rồi mở phiên Claude Code mới để chạy bản mới:
  - **App Claude:** vào **Customize → Plugins**, bấm **Refresh marketplace**, rồi cập nhật **Magic Design with Figma**.
  - **Terminal** (nút "Copy lệnh cập nhật" trong panel chép sẵn):

    ```bash
    claude plugin marketplace update magic-design-with-figma
    claude plugin update magic-design-with-figma@magic-design-with-figma
    ```

  Muốn tự cập nhật: chạy `claude` trong Terminal, gõ `/plugin` → **Marketplaces** → chọn `magic-design-with-figma` → **Enable auto-update**.
- **Plugin Figma:** tải bản mới đè lên đúng thư mục cũ (hoặc `git pull` nếu đã clone). Figma đọc lại file ở lần mở plugin tiếp theo, không cần import lại.

Mỗi bản phát hành có tag theo số phiên bản (ví dụ `v0.0.4`); số phiên bản đang dùng hiện trong panel Figma.

## Claude làm được gì

- `get_context`, `inspect_nodes`: đọc file, trang, selection và cấu trúc layer.
- `execute_figma_code`: dựng và sửa layer bằng Figma Plugin API. Mỗi lần chạy có sửa file là một bước undo riêng.
- `screenshot`: chụp node để Claude tự kiểm tra.
- `audit_design`: đo contrast, vùng bấm, khoảng cách, màu và chữ chưa gắn biến… mà không sửa gì.

### Lệnh

Plugin kèm 11 skill tiếng Việt, mỗi skill là một lệnh. Gõ `/` rồi gõ tên lệnh (ví dụ `/handoff-spec Đăng nhập`), hoặc cứ nói bình thường ("bàn giao màn đăng nhập cho dev"), Claude sẽ tự chọn lệnh hợp. Mọi lệnh làm việc trên file Figma đang mở qua plugin, không cần kết nối dịch vụ nào khác.

**Dựng**

| Lệnh | Làm gì |
|---|---|
| `/design-workflow` | Thiết kế hoặc làm lại một màn hình, luồng, landing page. Bắt đầu từ đây |
| `/color-system` | Bảng màu thành Variables, có sáng và tối |
| `/layout-type` | Chữ, khoảng cách, lưới, kích thước, khung web/iOS/Android |
| `/components-states` | Component đủ trạng thái (hover, focus, disabled, loading, lỗi) |

**Kiểm và bàn giao**

| Lệnh | Làm gì |
|---|---|
| `/design-audit` | Chấm một màn: đo lỗi rồi nhận xét thứ bậc, bố cục, độ dễ dùng |
| `/a11y-audit` | Kiểm trợ năng theo WCAG 2.2 AA, ghi chú trợ năng lên layer cho dev |
| `/ux-writing` | Soát chữ của một màn, đề xuất câu tốt hơn, thay thẳng vào Figma |
| `/design-system-audit` | Kiểm biến, style, component của cả file; viết tài liệu component |
| `/handoff-spec` | Viết spec bàn giao cho dev từ số liệu thật trong file |

**Nghiên cứu người dùng**

| Lệnh | Làm gì |
|---|---|
| `/usability-test` | Lên kế hoạch cho người dùng thật thử prototype: nhiệm vụ, kịch bản, bảng ghi chép |
| `/research-board` | Tổng hợp ghi chép thành chủ đề và insight, bày lên board FigJam |

## Bảo mật

- Cầu nối chỉ nghe trên `127.0.0.1`/`::1` (cổng 3777), không mở ra mạng LAN.
- Panel Figma phải **ghép cặp** bằng mã 6 số nộp từ Claude Code thì mới nhận lệnh; trang web khác không giả làm panel hay phiên Claude Code được.
- Code do Claude viết chạy trong sandbox của plugin Figma: chỉ chạm được vào file đang mở, và mọi thay đổi đều hoàn tác được.
