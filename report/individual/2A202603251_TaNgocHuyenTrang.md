# Báo cáo thực hành Day 3 — Image Segmentation

- **Mã học viên theo lớp:** 2A202603251
- **Họ và tên:** Tạ Ngọc Huyền Trang
- **Ngày / CVAT local:** 12/11/2026 / CVAT Local (localhost:8080)
- **Công cụ đã dùng:** CVAT, Brush, Polygon

---

## 1. Bài đã nộp

Đã hoàn thành xuất dữ liệu cho toàn bộ 9 task (14 ảnh) theo định dạng yêu cầu của bài Lab Day 03. Sau khi khắc phục lỗi cấu hình class tại sáu checkpoint, cả 9 file ZIP đều đạt kiểm tra cấu trúc bằng `scripts/inspect_submissions.py`.

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| :--- | :--- | :---: | :---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | | **14 / 14** | **100** |

**Định dạng xuất:** Segmentation mask 1.1 cho Easy, CP3, CP4, CP6; COCO 1.0 cho Medium, Hard, CP1, CP2, CP5.

---

## 2. Một quyết định trước khi dùng gợi ý

- **Trước khi dùng gợi ý:** Tôi đã tự vẽ một object ở bài Medium. Sau đó, tôi xóa bản vẽ ban đầu khi chạy lại gợi ý nên bản xuất hiện tại không giữ riêng kết quả tự vẽ đó.
- **Đối tượng đối chiếu trong bản hiện tại:** Ảnh `000000181542.jpg`, người phụ nữ mặc áo dài sáng màu đứng gần giữa khung hình, phía trước các xe máy; class `person`. Đây là đối tượng được kiểm trong bản xuất, không xác nhận là object tự vẽ đầu tiên.
- **Quy tắc chọn biên:** Giữ phần cơ thể và trang phục nhìn thấy của người trong cùng mask; không lấy thêm xe máy hoặc nền đường phía sau.
- **Gợi ý đã dùng:** SAM hỗ trợ tạo mask. Tôi dùng quy tắc phần nhìn thấy để cân nhắc biên, thay vì chỉ dựa vào vùng gợi ý.

---

## 3. Một lỗi tôi tìm thấy và sửa

- **Task/ảnh/vùng:** `medium_instance`, ảnh `000000181542.jpg`, chiếc xe bán tải phía trên, gần giữa và lệch phải ảnh, phía sau người phụ nữ mặc áo dài.
- **Loại lỗi:** Sai class: bản xuất đang gán `car`, trong khi reference của bài quy định là `truck`.
- **Bằng chứng:** Mask bám khá sát chiếc xe nhưng tên class chưa đúng. Đối chiếu cùng vật thể với reference cho IoU 92,23%; sai class vẫn làm mất cặp ghép đúng.
- **Cách sửa cần thực hiện:** Đổi nhãn chiếc xe từ `car` sang `truck`, giữ phần mask phù hợp, kiểm lại rồi Save và xuất lại `medium_instance.zip` theo COCO 1.0.
- **Trạng thái bản nộp hiện tại:** ZIP đã đạt kiểm tra cấu trúc, nhưng lỗi class này vẫn còn trong bản đã kiểm. Chưa có bản xuất mới để xác nhận đã sửa xong.

---

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh / Vị trí | Hai cách hiểu có thể | Quy tắc / Chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `000000181542.jpg`, chiếc xe bán tải phía sau người phụ nữ mặc áo dài. | Gán `car` vì cabin giống xe con, hoặc `truck` theo phân loại của bài. | Reference xác định chiếc xe này là `truck`; bản xuất đang là `car`. | Cần sửa thành `truck`; hỏi coach để thống nhất quy ước cho xe bán tải ở các ảnh khác. |
| `000000373353.jpg`, xe trong hàng đỗ bên phải, phía sau người đi bộ qua đường. | Giữ mask hiện tại, hoặc kiểm lại biên xe và phần bị người che. | Mask `car` có IoU 48,21%, chưa đạt ngưỡng ghép 50%. Chỉ vẽ phần xe nhìn thấy. | Cần phóng to kiểm biên; không nối mask xuyên qua người đi bộ. |
| `000000458325.jpg`, người nhỏ ở cuối phố, gần giữa ảnh và lệch trái người trượt ván. | Bỏ qua vì nhỏ, hoặc bổ sung một instance `person` nếu xác định được phần nhìn thấy. | Reference có người tại vùng này, nhưng bản xuất chưa có mask `person` chồng lấp. | Cần kiểm lại và bổ sung phần người nhìn thấy; hỏi coach nếu biên quá mơ hồ, không đoán phần bị che. |

---

**Kết quả tự đánh giá:** Ba tier chính đạt tổng 42,2/82 điểm theo kết quả scorer đã ghi nhận (Easy: 17,8/20; Medium: 13,6/32; Hard: 10,8/30). Đây là điểm tự đánh giá, không phải điểm chính thức của coach. Chưa có kết quả tự đánh giá sáu checkpoint.
