# Hiểu brief trước khi vẽ

Mọi con số ở đây là mặc định để bắt đầu; thứ tự ưu tiên giữa file Figma, `DESIGN.md` và mặc định nằm
ở [../SKILL.md](../SKILL.md). Có lý do cụ thể thì làm khác và ghi lại (mục 7).

---

## 1. Thuận tiện là gì

**Thuận tiện không phải đơn giản.** Đơn giản là thuộc tính của giao diện; thuận tiện là thuộc tính của
việc *làm xong*. Giấu tính năng cho gọn thì người dùng thành thạo phải bấm thêm mỗi ngày.

Thuận tiện = làm xong việc với **ít tải nhận thức, ít thao tác, ít nỗi sợ sai**. Ba vế cân nhau.

| Loại tải nhận thức | Là gì | Xử lý |
|---|---|---|
| Nội tại | Độ khó vốn có của công việc | Không cắt được |
| Ngoại lai | Do giao diện gây ra: phải nhớ mã, phải tự tính tổng | **Cắt tối đa**: đây là việc của người thiết kế |
| Hữu ích | Giúp người dùng tự hiểu: tổng tiền cập nhật khi gõ | Giữ, có khi tăng |

**Bảy nguồn bất tiện**, nặng nhất trước:

1. Mất việc đang làm.
2. Không biết chuyện gì đang xảy ra.
3. Phải nhớ thay vì được nhắc.
4. Làm lại việc máy làm được.
5. Không hoàn tác được.
6. Thông báo lỗi vô dụng.
7. Giao diện đổi chỗ khi tải xong.

Chỉ sửa được ba thì sửa 1, 2, 5. Trên canvas, ba cái đó hiện thành: trạng thái lưu nháp, trạng thái
loading và error có vẽ, nút Hoàn tác trong toast.

**Đo, đừng tranh luận.** Với app dùng hằng ngày, chỉ số quan trọng nhất là thời gian làm xong **khi đã
thành thạo**: người dùng chỉ mới một lần, nhưng thành thạo suốt nhiều năm. Trên thiết kế, đếm được ngay
số cú bấm cho việc chính bằng cách đi theo luồng giữa các frame.

---

## 2. Bốn tầng: đặc tả trước khi vẽ

```
Tầng 4 · Bề mặt   │ layout, chữ, màu, khoảng cách      ← người dùng cảm từ trên xuống
Tầng 3 · Luồng    │ điều hướng, thứ tự màn hình
──────────────────┼─────────────────────────────────
Tầng 2 · Mô hình  │ đối tượng, quan hệ, từ vựng
Tầng 1 · Nhiệm vụ │ người dùng đến đây để làm gì       ← người làm đi từ dưới lên
```

Khi ai đó nói "giao diện rối", phần lớn lỗi nằm ở tầng 1–2. **Tô lại tầng 4 không sửa được.** Chưa rõ
tầng 1–3 thì hỏi, hoặc ghi giả định, trước khi dựng frame đầu tiên.

### 2.1. Tầng 1, nhiệm vụ: tần suất × hoàn tác

Viết việc bằng **động từ của người dùng**, không bằng tên tính năng:

| Tên tính năng | Việc thật |
|---|---|
| Quản lý đơn hàng | Cuối ngày dò các đơn chưa giao để gọi lại cho khách |
| Module báo cáo | Sáng thứ Hai gửi trưởng nhóm số liệu tuần trước |

Gắn mỗi việc **tần suất** và **có hoàn tác được không**, rồi đặt theo ma trận:

| | Hoàn tác được | Không hoàn tác được |
|---|---|---|
| **Hằng ngày** | Bỏ hết ma sát. Nằm ở màn hình đầu. Có phím tắt | Màn hình đầu, tách xa nút hay dùng. Có hoàn tác |
| **Hằng tuần** | Cách màn hình đầu một cú bấm | Xác nhận nhẹ |
| **Hiếm khi** | Menu phụ cũng được | Nằm trong Cài đặt. Xác nhận bằng cách gõ lại tên |

- **Tần suất quyết định vị trí.** Sai thì người dùng mệt mỗi ngày.
- **Mức nguy hiểm quyết định ma sát.** Sai thì người dùng mất dữ liệu.

Xong tầng 1 khi trả lời được: *việc làm nhiều nhất là gì, và nó mất bao nhiêu cú bấm?*

### 2.2. Tầng 2, mô hình: đối tượng và bảng từ vựng

Giao diện phản ánh **mô hình trong đầu người dùng**, không phải cấu trúc database. Database lưu từng bản
ghi "tiết học", giáo viên nghĩ theo "tuần của tôi": lộ cấu trúc bản ghi ra là bắt người dùng dịch ngược
mỗi lần dùng.

1. Liệt kê **danh từ** (đối tượng) và **động từ** (hành động trên đối tượng).
2. Mỗi đối tượng chính: có màn danh sách không, màn chi tiết không, tạo/sửa/xoá ở đâu.
3. Lập bảng từ vựng ba cột. Chữ trên mọi frame lấy từ cột cuối.

| Tên trong code | Người dùng nói | Chữ trên giao diện |
|---|---|---|
| `class_session` | tiết | Tiết |
| `teaching_log` | sổ đầu bài | Sổ đầu bài |
| `entity` | — | Không bao giờ hiện |

Ba lỗi hay gặp: đặt tên theo bảng database · đổi từ quen thành từ "chuyên nghiệp hơn" · một khái niệm
mang hai tên ở hai màn hình.

### 2.3. Tầng 3, luồng: ba cấp điều hướng

Mọi màn hình trả lời được trong hai giây: *tôi đang ở đâu, làm được gì ở đây, tiếp theo là gì.*

| Cấp | Vị trí | Nội dung | Tối đa |
|---|---|---|---|
| Toàn cục | Sidebar hoặc topbar | Khu vực chính của sản phẩm | 7 (trần 9) |
| Cục bộ | Tab trong trang | Các mặt của cùng một đối tượng | 5 |
| Ngữ cảnh | Trên từng dòng, thẻ | Hành động trên một đối tượng | 3 hiện ra + menu `…` |

**Cần tới cấp thứ tư là dấu hiệu mô hình tầng 2 sai.** Quay lại tầng 2.

Trạng thái đáng nhớ (tab đang chọn, bộ lọc, sắp xếp, trang đang xem) cần có địa chỉ riêng để gửi cho
nhau và quay lại được: vẽ nó thành frame riêng khi nó đổi giao diện đáng kể, và ghi vào ghi chú bàn
giao. Form dài: vẽ dấu hiệu tự lưu nháp ("Đã lưu nháp lúc 10:32").

---

## 3. Ba nguyên tắc

### 3.1. Nhận ra dễ hơn nhớ lại

| Bắt nhớ | Cho nhận ra |
|---|---|
| Ô nhập mã khách hàng | Ô tìm có gợi ý, hiện cả tên lẫn mã |
| "Nhập theo định dạng YYYY-MM-DD" | Chọn ngày, hoặc tự định dạng khi gõ |
| Bước 3 không thấy lại lựa chọn ở bước 1 | Tóm tắt các bước trước luôn hiện |
| Menu chính giấu sau icon hamburger trên desktop | Nav luôn hiện |

### 3.2. Nhất quán quan trọng hơn hoàn hảo

Một quy ước hơi dở áp dụng khắp nơi tốt hơn năm quy ước tối ưu cục bộ: mỗi ngoại lệ là một lần học
lại. Thứ tự ưu tiên: nhất quán **với chính sản phẩm** (component và style đã có trong file) → **với nền
tảng** (web, iOS, Android) → **với thế giới** (nút đóng góc trên bên phải, logo về trang chủ).

### 3.3. Hoàn tác tốt hơn xác nhận

"Bạn có chắc không?" đẩy trách nhiệm sang người dùng, và sau vài lần ai cũng bấm bừa. Hỏi xác nhận chỉ
khi không hoàn tác được. Mỗi mức kéo theo một frame cần vẽ: toast có nút Hoàn tác, hộp xác nhận, hoặc
hộp bắt gõ lại tên đối tượng. Bảng bốn mức và cách viết hộp xác nhận: skill `components-states`,
reference/ux-copy.md §5.

---

## 4. Register và câu bối cảnh

**Nhận ra register** từ lời nhờ và từ file:

- *Product*: lời nhờ nói tới quản lý, theo dõi, nhập liệu, phân quyền, cấu hình; file đã có sidebar,
  bảng, form. Tiền lệ là một tính năng: dùng mẫu người dùng đã gặp ở app khác (sidebar, tab, bảng,
  command palette), đừng chế control mới cho lạ mắt.
- *Brand*: lời nhờ nói tới giới thiệu, ra mắt, chiến dịch, kể chuyện thương hiệu; file có hero, khối
  chữ lớn, ảnh. Trang loại này cần nét riêng: xin người dùng hai, ba cái tên có thật họ muốn học theo
  và học điểm nào. Một cái tên thật dễ bàn hơn những chữ như "sang", "trẻ trung", "tinh tế".
- Không rõ thì hỏi đúng một câu: "Người dùng đến màn này để làm việc, hay để đọc và được thuyết phục?"

**Câu bối cảnh** tả một lần dùng thật: ai, cầm máy gì, ở chỗ nào, xung quanh ra sao, có gấp không. Câu
tốt là câu mà người khác đọc xong tự chọn được nền, mật độ, cỡ chữ giống mình:

| Câu bối cảnh | Kéo theo |
|---|---|
| Kế toán đối soát cuối tháng trên màn 27 inch trong văn phòng sáng đèn, ngồi ba tiếng liền | Nền sáng, mật độ dày, chữ 14px, bảng là trung tâm |
| Nhân viên kho quét mã bằng điện thoại cũ ngoài bãi nắng, tay đeo găng, lúc nào cũng vội | Nền sáng, contrast cao, vùng bấm 48px, chữ từ 16px, rất ít chữ |
| Khách xem lịch chiếu trong sảnh rạp tối, vài phút trước giờ vào | Nền tối (không `#000`), độ sáng thấp, chữ vừa đủ lớn |

---

## 5. Mẫu brief và cách hỏi

Brief gọn, 5–8 dòng, gửi trong chat trước khi dựng việc lớn:

```
Việc chính:    ai, làm gì, bao lâu một lần
Register:      product | brand. Bối cảnh: <một câu>
Nền tảng:      desktop 1440 | mobile (web 390, iOS 393×852, Android 360×800) | cả hai
Màn hình:      danh sách màn + trạng thái cần vẽ
Nội dung thật: dữ liệu mẫu; cỡ dữ liệu 0 / vài / rất nhiều
Dùng lại:      variables, text styles, components có sẵn trong file
Mức hoàn thiện: phác xám | hoàn chỉnh | có prototype
Không muốn:    điều mà thấy là biết sai
```

- Tối đa **3 câu hỏi một lượt**, chọn câu làm đổi thiết kế nhiều nhất: nền tảng, việc chính, register,
  dữ liệu thật.
- Lời nhờ và file đã gần như trả lời thì **nói cách hiểu của mình** thành một câu để người dùng gật
  hoặc sửa ("Mình hiểu đây là app nội bộ, dùng trên desktop, đúng không?"). Bày năm phương án ra là bắt
  người dùng làm thay việc thiết kế.
- Câu nào tự trả lời được bằng một giả định hợp lý thì không hỏi: ghi giả định vào brief, người dùng thấy
  sai sẽ sửa.

---

## 6. Quy trình mười bước

| # | Bước | Làm bằng |
|---|---|---|
| 1 | Hiểu việc, không hiểu tính năng: người dùng đang làm việc đó bằng gì (Excel, giấy, phần mềm cũ) | Hỏi trong chat |
| 2 | Bảng tần suất × hoàn tác (mục 2.1) | Brief |
| 3 | Đối tượng + bảng từ vựng (mục 2.2) | Brief |
| 4 | Phác luồng: ô vuông và mũi tên, chưa đẹp (mục 2.3) | Frame xám trên canvas, hoặc danh sách trong chat |
| 5 | Chọn khung xương | Skill `components-states` |
| 6 | **Dựng màn khó nhất trước**: bảng nhiều cột nhất, trang phân quyền, form dài nhất. Sống được ở đó thì chỗ khác tự ổn | `execute_figma_code` + `screenshot` |
| 7 | Đổ dữ liệu bẩn | [screen-states.md](screen-states.md) |
| 8 | Đủ bảy trạng thái cho các màn chính | [screen-states.md](screen-states.md) |
| 9 | Kiểm tính khả thi ngay từ bước 4: dùng component có sẵn, tránh hiệu ứng khó dựng. Thiết kế không dựng kịp là thiết kế tồi | Hỏi người dùng khi nghi |
| 10 | Checklist bàn giao + `audit_design` | [handoff.md](handoff.md), skill `design-audit` |

Sau khi ra mắt: đo lại số cú bấm và thời gian làm xong. Bước hay bị bỏ nhất, và là bước duy nhất cho
biết đúng hay sai.

**Góp ý mơ hồ** ("thấy sao sao ấy"): đừng hỏi "muốn sửa chỗ nào". Hỏi ba câu để lấy **quan sát** thay vì
ý kiến:

1. Nhìn vào cái gì đầu tiên?
2. Nút nào là việc chính ở màn hình này?
3. Nếu là người dùng, sẽ bấm gì tiếp theo?

Họ nhìn đúng chỗ đã định thì "sao sao" chỉ là thói quen. Nhìn nhầm thì có vấn đề thật, và biết chính
xác nó ở đâu. Tự hỏi ba câu này với screenshot của mình trước khi báo cáo.

---

## 7. Khi nào được phá mặc định

Đủ cả ba điều kiện:

1. **Giải thích được bằng lợi ích cho người dùng**, không bằng thẩm mỹ. "Hai nút chính vì hai việc
   ngang hàng, người dùng chọn 50-50": được. "Trông đẹp hơn": không.
2. **Phá nhất quán, không phá lẻ tẻ.** Phá ở đúng một màn hình là tệ nhất trong mọi lựa chọn.
3. **Ghi lại**: phá cái gì, vì sao, ở đâu. Ghi thành annotation hoặc khung ghi chú cạnh frame
   ([handoff.md](handoff.md)); dự án có `DESIGN.md` thì đề xuất với người dùng thêm một dòng vào đó.
   Không ghi thì sáu tháng sau có người "sửa lại cho đúng".

| Mặc định | Thường phá đúng khi |
|---|---|
| Mật độ mặc định | Người dùng là chuyên gia nhập liệu: chọn mức dày nhất |
| Phân trang thay cuộn vô hạn | Nội dung kiểu feed, người ta duyệt chứ không tìm |
| Một nút chính mỗi màn | Hai việc thật sự ngang hàng, tần suất ngang nhau |
| Nhãn phía trên ô nhập | Form cấu hình rất dài: nhãn bên trái tiết kiệm chiều dọc đáng kể |
| Chữ thân 14px cho app | Người lớn tuổi, màn hình xa, hoặc bề mặt để đọc |
