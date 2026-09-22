# Câu chữ trên canvas

Chữ trên giao diện là một phần của thiết kế, không phải thứ điền sau. Vẽ bằng **câu thật** ngay từ
đầu: lorem ipsum giấu mất độ dài thật của câu, và `audit_design` báo nó là `noi-dung-gia`.

## Mục lục

1. Ba câu hỏi
2. Công thức một thông báo
3. Ba trục: ai sửa, có chặn không, dữ liệu ra sao
4. Thang gián đoạn — chọn kênh
5. Phòng ngừa trước, báo lỗi sau cùng
6. Nhãn nút và link
7. Thời gian chờ — ngưỡng thành ghi chú
8. Câu chữ trong form
9. Trống, không có quyền, không làm được
10. Bảng từ vựng
11. Núm vặn giọng văn
12. Đưa câu chữ vào Figma
13. Không dùng

---

## 1. Ba câu hỏi

Mỗi khi có chuyện xảy ra, người dùng hỏi đúng theo thứ tự:

```
1. Chuyện gì vừa xảy ra?
2. Tại sao?
3. Giờ tôi làm gì?        ← quan trọng nhất, hay bị quên nhất
```

Viết ba câu trả lời ra trước, rồi mới gọt thành câu chữ. **Không trả lời được câu 2 hoặc câu 3 thì
không phải vấn đề câu chữ** — hoặc chưa hiểu lỗi, hoặc sản phẩm thiếu một đường thoát. Báo lại cho
người dùng thay vì cố viết cho hay.

## 2. Công thức một thông báo

```
[chuyện gì] + [vì sao] → [làm gì tiếp]  (+ [trấn an], nếu dữ liệu không an toàn)
```

| Quy tắc | Không nên | Nên |
|---|---|---|
| Chủ ngữ rõ khi hệ thống hỏng | "Lưu thất bại" | "Chúng tôi chưa lưu được thay đổi" |
| Không đùa khi người ta đang gặp chuyện | "Oops!" | "Chúng tôi xin lỗi vì sự cố này" |
| Bước tiếp là **nút**, không phải chữ | "…vui lòng thử lại" | Nút **Thử lại** |
| Ngắn | Ba dòng giải thích | 1–2 dòng |
| Không đổ lỗi | "Bạn đã nhập sai" | "Email cần có dấu @" |

Câu dài quá hai dòng thường là dấu hiệu chưa hiểu rõ lỗi. Người ta sợ mất công sức hơn sợ lỗi:
*"Bản nháp của bạn vẫn còn"* đáng giá hơn cả đoạn giải thích nguyên nhân.

Xung đột dữ liệu hoặc thao tác bị từ chối sau khi đã hiện kết quả trước: nói rõ, không âm thầm quay
lại — *"Thay đổi vừa rồi chưa lưu được vì có người khác đã sửa mục này lúc 10:32. Đã khôi phục về
bản mới nhất."*

## 3. Ba trục: ai sửa, có chặn không, dữ liệu ra sao

Gán ba thuộc tính cho mỗi tình huống; chúng tự quyết giọng văn, kênh và nội dung.

| Trục | Giá trị → cách viết |
|---|---|
| **A — Ai sửa được?** | Người dùng → chỉ rõ chỗ sai và cách sửa, trung tính · Người có quyền cao hơn → nói cần liên hệ ai · Hệ thống → xin lỗi, nhận trách nhiệm, cho mã tham chiếu · Bên ngoài (mạng, dịch vụ khác) → gợi ý thử lại, nói rõ dữ liệu có an toàn không |
| **B — Có chặn công việc không?** | Chặn → kênh nặng hơn, bắt buộc có nút hành động · Không chặn → kênh nhẹ |
| **C — Dữ liệu ra sao?** | An toàn · Có rủi ro · Đã mất → quyết định có cần câu trấn an không |

```
Người dùng + Chặn + An toàn
  → inline, chỉ chỗ sai, không xin lỗi
  → "Email cần có dấu @. Sửa lại rồi bấm Tiếp tục."

Hệ thống + Chặn + Có rủi ro
  → banner hoặc modal, xin lỗi, trấn an, mã tham chiếu, đường thoát
  → "Chúng tôi chưa lưu được thay đổi. Nội dung vẫn đang giữ trên máy này — đừng đóng tab.
     Thử lại sau vài giây. Mã: #A7K2"

Bên ngoài + Không chặn + An toàn
  → toast nhẹ hoặc chỉ dấu trạng thái
  → "Đang offline — thay đổi sẽ đồng bộ khi có mạng."
```

Hai người tranh nhau một câu thông báo → đừng tranh câu chữ, hỏi *ba trục của nó là gì*. Thường
hoá ra hai người đang phân loại khác nhau.

## 4. Thang gián đoạn — chọn kênh

Mỗi lần ngắt người dùng là tiêu một khoản chú ý có hạn. Chọn kênh **nhẹ nhất còn đủ tác dụng**.

```
Im lặng → Inline → Toast → Banner → Modal → Toàn trang
 rẻ nhất                                      đắt nhất
```

| Kênh | Dùng khi | Ghi trên canvas |
|---|---|---|
| **Im lặng** | Thành công đúng như kỳ vọng — không phải thành công nào cũng cần báo | — |
| **Inline** | Thông tin gắn với một phần tử cụ thể (một ô, một dòng) | Vẽ ngay dưới phần tử đó |
| **Toast** | Đã xong / đã hỏng, không cản bước tiếp | Thành công tự tắt ~4 giây · cảnh báo ~6 giây · **lỗi không tự tắt**, có nút đóng · có nút Hoàn tác thì giữ 8–10 giây · tối đa 3 cái, **cùng một góc** trên toàn sản phẩm |
| **Banner** | Trạng thái kéo dài, ảnh hưởng cả màn hình (offline, sắp hết hạn) | Đầu vùng nội dung, có nút hành động nếu có việc để làm |
| **Modal** | Sắp mất dữ liệu, hoặc không đi tiếp được nếu chưa quyết | Chỉ khi chặn việc hoặc dữ liệu không an toàn |
| **Toàn trang** | Cả chức năng không dùng được | Có đường về trang chính |

Dấu hiệu tiêu hoang: người dùng bấm "OK" theo phản xạ mà không đọc. Khi đó modal đã mất tác dụng, và
lúc thật sự cần cảnh báo thì không còn công cụ nào.

Toast không đọc được thời gian trên hình — ghi lên component:

```js
const toast = await figma.getNodeByIdAsync(NODE_ID)
if (!toast || !('annotations' in toast)) throw new Error('Node này không nhận annotation')
toast.annotations = [...toast.annotations, { // giữ ghi chú đã có, chỉ thêm
  labelMarkdown: '**Thời gian**: thành công tự tắt ~4 giây, cảnh báo ~6 giây, **lỗi không tự tắt**. ' +
    'Có nút Hoàn tác: giữ 8–10 giây. Tối đa 3 toast, luôn ở cùng một góc.',
}]
return 'ok'
```

## 5. Phòng ngừa trước, báo lỗi sau cùng

Trước khi gọt câu báo lỗi, leo thang từ trên xuống; chỉ khi không bậc nào làm được mới tới bậc cuối.

| Bậc | Cách làm — ví dụ với ô ngày tháng |
|---|---|
| 1. **Ngăn** | Không cho chọn ngày không hợp lệ |
| 2. **Tự sửa** | Gõ `20-9-26` → tự hiểu 20/09/2026 |
| 3. **Cảnh báo** | Báo khi rời ô, trước lúc gửi |
| 4. **Hoàn tác** | Làm luôn, cho 8–10 giây bấm Hoàn tác |
| 5. **Báo lỗi** | Giải thích sau khi đã hỏng |

**Hoàn tác tốt hơn xác nhận.** "Bạn có chắc không?" đẩy trách nhiệm sang người dùng, và sau lần thứ
ba ai cũng bấm bừa. Khi không hoàn tác được thì xác nhận đúng mức:

| Hành động | Cách xử lý |
|---|---|
| Hoàn tác dễ | Làm ngay, toast có nút Hoàn tác |
| Hoàn tác tốn công | Làm ngay, toast Hoàn tác lâu hơn (~15 giây) + thùng rác |
| Không hoàn tác được, phạm vi nhỏ | Hộp xác nhận |
| Không hoàn tác được, phạm vi lớn | Bắt gõ lại tên đối tượng |

Hộp xác nhận: tiêu đề nói việc sắp xảy ra, thân nói hậu quả, hai nút nói đúng việc của chúng —
**"Xoá 12 học sinh" / "Giữ lại"**, không "OK" / "Huỷ". Ghi chú cho dev: focus mặc định vào nút an toàn.

## 6. Nhãn nút và link

- **Động từ + danh từ**, 1–3 từ: "Lưu thay đổi", "Gửi báo cáo", "Tạo lớp". Hành động hàng loạt nói
  số lượng: "Xoá 25 mục".
- Không "OK", "Đồng ý", "Xác nhận", "Gửi đi" chung chung — đọc riêng cái nút phải biết nó làm gì.
- Nhãn nút trong trạng thái Loading: động từ đang diễn ra — "Đang lưu…", "Đang gửi…".
- Chữ của link tự nói nó dẫn tới đâu, kể cả khi bị đọc riêng ra (trình đọc màn hình hay liệt kê
  các link liền nhau): "Tải bảng điểm học kỳ 1" thay cho "tại đây".
- Icon button không có chữ → tooltip viết như nhãn nút ("Xoá dòng"), và cùng câu đó làm tên cho trình
  đọc màn hình (ghi chú cho dev).

## 7. Thời gian chờ — ngưỡng thành ghi chú

Thời gian không vẽ được. Vẽ **hình của từng mốc** (nút đang tải, spinner tại chỗ, skeleton, thanh tiến
trình) và ghi ngưỡng lên màn hình hoặc component. Năm mốc thời gian (dưới 0,1 giây · dưới 1 giây ·
1–3 giây · 3–10 giây · trên 10 giây), việc bắt buộc ở từng mốc và frame cần vẽ: skill
`design-workflow`, reference/screen-states.md §3. Phần câu chữ:

- Chữ mô tả bằng ngôn ngữ nghiệp vụ: không "Đang truy vấn" mà "Đang tải danh sách đơn hàng — 3/5
  trang". Việc lâu thì báo trước độ dài: "Xuất file thường mất khoảng một phút".
- Không bao giờ để màn hình đứng im quá 10 giây không tín hiệu — người dùng sẽ tải lại trang và mất
  cả tác vụ.

## 8. Câu chữ trong form

| Phần | Viết thế nào |
|---|---|
| Nhãn | Danh từ ngắn, luôn hiện, phía trên ô: "Số điện thoại" |
| Placeholder | Chỉ là ví dụ định dạng: "vd: 0912 345 678" — không thay nhãn, không chứa hướng dẫn quan trọng |
| Hướng dẫn | Dưới ô, một câu, chỉ khi trường không hiển nhiên — kể cả **vì sao** hỏi: "Dùng để gửi mã xác nhận" |
| Tuỳ chọn | Đa số ô bắt buộc thì ghi "(không bắt buộc)" cạnh nhãn ô tuỳ chọn |
| Lỗi | Thay chỗ hướng dẫn, theo công thức §2: "Số điện thoại cần đủ 10 chữ số. Kiểm tra lại rồi bấm Lưu." |

Thời điểm kiểm tra — ghi chú cho dev trên component Input:

| Lúc nào | Kiểm gì | Hiển thị |
|---|---|---|
| Đang gõ | Chỉ giới hạn ký tự, độ dài | Đếm ký tự. **Không** hiện lỗi đỏ |
| Rời ô | Định dạng, giá trị hợp lệ | Inline ngay dưới ô |
| Bấm gửi | Ràng buộc giữa các trường, kiểm phía máy chủ | Inline từng ô + cuộn tới ô sai đầu tiên; nhiều lỗi thì tóm tắt ở đầu form |

Sai kinh điển: hiện lỗi đỏ từ ký tự đầu tiên — người dùng thấy bị mắng khi chưa làm gì sai.

## 9. Trống, không có quyền, không làm được

- **Trạng thái trống**: một câu nói chỗ này để làm gì + nút hành động chính + một đường thoát
  (hướng dẫn, dữ liệu mẫu, nhập từ Excel). Ba ví dụ cho bảng ở
  [patterns.md §8](patterns.md#8-trạng-thái-trống).
- **Không có quyền**: nói cần quyền gì và hỏi ai — "Bạn cần quyền Giáo vụ để sửa điểm. Liên hệ quản
  trị viên của trường." Khi nào hiện câu này, khi nào hiện trang không tìm thấy: skill
  `design-workflow`, reference/screen-states.md §1.2.
- **Hệ thống không làm được việc được yêu cầu**: "Không thực hiện được" là câu tệ nhất — nó đóng mọi
  cánh cửa. Thay bằng: nói rõ cái gì đang cản ("Hai điều kiện A và B đang mâu thuẫn") → đưa kết quả
  gần đúng nếu có (bản 90% kèm danh sách điểm chưa đạt) → luôn có ít nhất một nút dẫn tới nơi sửa được.

## 10. Bảng từ vựng

Một khái niệm chỉ có **một tên** trong toàn sản phẩm. Lúc "mục", lúc "bản ghi", lúc "item" là nguồn
gốc cảm giác "app thiếu chuyên nghiệp" mà người dùng không chỉ ra được. Tương tự cho động từ: chọn
"Xoá" thì không nơi nào dùng "Gỡ" hay "Loại bỏ" cho cùng việc.

Lập bảng từ vựng ba cột (tên trong code · người dùng nói · chữ trên giao diện) khi bắt đầu, đặt trên
trang Cover; mẫu bảng ở skill `design-workflow`, reference/brief.md §2.2. Thuật ngữ của mô hình dữ liệu
không được lọt lên giao diện.

## 11. Núm vặn giọng văn

Quyết một lần cho cả sản phẩm, ghi trên trang Cover hoặc `DESIGN.md`. Chưa biết → hỏi người dùng.

| Núm | Cần quyết |
|---|---|
| Xưng hô | "bạn" · "anh/chị" · không xưng hô |
| Mức trang trọng | Thân mật ↔ nghiêm túc; có dùng emoji không |
| Mức xin lỗi khi hệ thống hỏng | Tài chính, y tế cần trang trọng hơn nhiều |
| Ngưỡng chờ | App nội bộ chịu chờ lâu hơn app đại chúng |
| Bảng từ vựng | Từ dùng / không dùng của nghiệp vụ |

## 12. Đưa câu chữ vào Figma

- Chữ thay đổi theo ngữ cảnh → **property TEXT** của component (`Label`, `Helper text`, `Title`),
  không override tay trong từng lớp con.
- Độ dài thật: đặt câu dài nhất có thể xảy ra vào ít nhất một instance. Giao diện có thể được dịch →
  chừa khoảng 30% bề rộng cho chữ dài ra; nhãn nút để hug theo chữ, không đặt bề rộng cố định.
- Số liệu đặt thành cặp **nhãn — giá trị** (hai layer chữ, hoặc nhãn rồi đến số), đừng kẹp số vào
  giữa một câu: bản dịch chỉ phải đổi nhãn, không phải xếp lại cả câu.
- Thời gian, điều kiện, hành vi → annotation (§4), không nhét vào câu hiển thị.
- `audit_design` bắt được: `ro-ri-gia-tri` (`undefined`, `NaN`, `[object Object]`, `Invalid Date`,
  `null` hiện ra như chữ) và `noi-dung-gia` (lorem ipsum). Không đo được: câu có dễ hiểu không —
  máy xếp nó vào `notMeasured`, nên phải tự đọc lại.

## 13. Không dùng

- "Oops", "Rất tiếc!", đùa khi báo lỗi.
- "Có lỗi xảy ra" **một mình**. Lỗi hệ thống không có "vì sao" để nói thì vẫn phải có bước tiếp và mã
  tham chiếu: "Có lỗi xảy ra phía chúng tôi. Thử lại sau vài giây. Mã hỗ trợ: #A7K2" — và "Thử lại" là nút.
- "OK", "Đồng ý", "Xác nhận" làm nhãn nút hành động.
- Stack trace, tên bảng, đường dẫn máy chủ, câu truy vấn, thông báo thô của thư viện. Với người không
  có quyền, "không tồn tại" và "không được phép" phải trông giống hệt nhau.
- Thuật ngữ kỹ thuật hoặc tiếng Anh lọt lên giao diện tiếng Việt khi đã có từ người dùng quen.
- Câu chỉ nhắc lại điều đã thấy trên màn hình (thân hộp thoại chép tiêu đề, dòng hướng dẫn chép nhãn
  nút). Mỗi dòng chữ phải mang thêm một thông tin; không có thì bỏ dòng đó.
