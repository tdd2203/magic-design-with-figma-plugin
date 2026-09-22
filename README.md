# Magic Design with Figma

Thiết kế trong Figma bằng Claude Code: bạn chat với Claude trong Claude Code (đăng nhập tài khoản Claude, **không cần API key**), còn Claude đọc file, dựng layer và tự chụp ảnh kiểm tra ngay trên canvas.

Repo này chỉ chứa bản đã build để cài đặt. Gồm hai plugin nói chuyện với nhau qua `localhost`:

- **Plugin Claude Code** (`claude-plugin/`): một MCP server nhỏ mà Claude Code tự khởi động, cung cấp tool Figma và các skill thiết kế cho Claude.
- **Plugin Figma** (`manifest.json`, `dist/`): panel trong Figma: nút "Kết nối", chọn thư mục dự án cho Claude, và nhật ký các bước Claude làm.

## Cần có

- [Claude Code](https://claude.com/claude-code), đăng nhập tài khoản Claude
- [Figma Desktop](https://www.figma.com/downloads/) (plugin phát triển không chạy trên bản trình duyệt)
- [Node.js](https://nodejs.org) 18 trở lên

## Cài đặt

1. **Plugin Claude Code.** Trong Claude Code, chạy:

   ```
   /plugin marketplace add tdd2203/magic-design-with-figma-plugin
   /plugin install magic-design-with-figma@magic-design-with-figma
   ```

2. **Plugin Figma.** Tải repo này về máy: bấm **Code → Download ZIP** rồi giải nén vào một thư mục cố định, hoặc:

   ```bash
   git clone https://github.com/tdd2203/magic-design-with-figma-plugin.git
   ```

   Trong Figma Desktop: **Plugins → Development → Import plugin from manifest…**, chọn file `manifest.json` trong thư mục vừa tải.

3. Mở `claude` trong thư mục dự án, mở plugin **Magic Design with Figma** trong Figma rồi bấm **Kết nối**.
4. **Ghép cặp (một lần mỗi máy):** panel hiện một mã 6 số. Gõ `/magic-design-with-figma:pair 482913` trong Claude Code (nút "Copy lệnh" trong panel chép sẵn), hoặc chỉ cần nói mã với Claude.
5. **Chọn thư mục:** panel liệt kê các thư mục đang chạy `claude`, thư mục có khả năng đúng nhất nằm đầu kèm nhãn gợi ý. Bấm một thư mục là xong.
6. Nhờ Claude, ví dụ: "Thiết kế màn hình đăng nhập mobile trong Figma".

Claude Code sẽ hỏi quyền ở lần đầu mỗi tool được gọi. Chọn "always allow" nếu không muốn bị hỏi lại.

## Cập nhật

- **Plugin Claude Code:** mặc định Claude Code không tự cập nhật plugin từ marketplace bên thứ ba. Bật bằng cách chạy `/plugin` → **Marketplaces** → chọn `magic-design-with-figma` → **Enable auto-update**. Khi có bản mới, khởi động lại Claude Code để MCP server chạy bản mới.
- **Plugin Figma:** tải bản mới đè lên đúng thư mục cũ (hoặc `git pull` nếu đã clone). Figma đọc lại file ở lần mở plugin tiếp theo, không cần import lại.

Mỗi bản phát hành có tag theo số phiên bản (ví dụ `v0.0.4`); số phiên bản đang dùng hiện trong panel Figma.

## Claude làm được gì

- `get_context`, `inspect_nodes`: đọc file, trang, selection và cấu trúc layer.
- `execute_figma_code`: dựng và sửa layer bằng Figma Plugin API. Mỗi lần chạy là một bước undo riêng.
- `screenshot`: chụp node để Claude tự kiểm tra.
- `audit_design`: đo contrast, vùng bấm, khoảng cách, màu và chữ chưa gắn biến… mà không sửa gì.

Kèm 5 skill thiết kế (tiếng Việt): `design-workflow`, `color-system`, `layout-type`, `components-states`, `design-audit`. Claude chỉ nạp khi cần.

## Bảo mật

- Cầu nối chỉ nghe trên `127.0.0.1`/`::1` (cổng 3777), không mở ra mạng LAN.
- Panel Figma phải **ghép cặp** bằng mã 6 số nộp từ Claude Code thì mới nhận lệnh; trang web khác không giả làm panel hay phiên Claude Code được.
- Code do Claude viết chạy trong sandbox của plugin Figma: chỉ chạm được vào file đang mở, và mọi thay đổi đều hoàn tác được.
