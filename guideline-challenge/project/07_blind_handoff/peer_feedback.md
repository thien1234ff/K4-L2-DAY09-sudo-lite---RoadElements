# Peer feedback + owner response

Phần 1 do **nhóm peer** trả lời (gửi kèm file export). Phần 2 do **nhóm owner** điền.

- **Nhóm peer:** Group 2 (Traffic Light Annotators)
- **Người label blind:** Peer Annotator (Blind Test Reviewer)

## 1. Peer trả lời

1. **Rule nào rõ nhất / giúp quyết định nhanh nhất?**
   - Mỗi hộp đèn là 1 box, không gom cụm (§2, §10.2). Trên cần vươn của LISA vẽ 3 box riêng ngay lập tức, khớp nhau giữa các frame.
   - Rule v2: "đầu đèn cạnh biển rẽ/quay đầu trái là `cross_traffic`" (§4). Giúp gán `red / cross_traffic / left` trong vài giây nhờ có mốc thị giác cụ thể là tấm biển phụ.
   - Loại trừ ảnh phản chiếu trên kính / nắp ca-pô (§1): Ở BDD26 vệt xanh phản chiếu trên kính lái được bỏ qua ngay.

2. **Rule nào mơ hồ hoặc phải tự suy diễn?**
   - `relevance = ego_lane` khi camera đứng sát vạch dừng và không thấy vạch phân làn dưới bánh xe (ở chuỗi LISA).
   - Đèn người đi bộ: Thiếu trạng thái cho số đếm ngược trắng/walk, và `direction` không có giá trị phù hợp (phải gán bừa `general`).
   - Vẽ vỏ ban đêm khi vỏ chìm hoàn toàn vào nền đen (ở BDD26), zoom 400% vẫn không thấy vỏ buộc phải vi phạm rule No-bloom để vẽ ôm quầng sáng.

3. **Sample nào khiến guideline "vỡ"?**
   - **LISA20 / 25 / 28:** Nhiều đầu tròn cùng màu trên giàn, nhưng không thấy vạch làn xe chủ dẫn đến suy đoán ngược về `ego_lane` vs `cross_traffic`.
   - **BDD26:** Vỏ đèn hoàn toàn vô hình trong đêm, đèn người đi bộ bị lấn át bởi quầng sáng xanh.
   - **BDD18:** Đèn đếm ngược người đi bộ không có trạng thái mô tả tương ứng.

4. **Attribute / default nào trong CVAT dễ gây thao tác sai?**
   - `needs_review` mặc định là `false` nên annotator quên tích khi gặp ca mơ hồ (0/20 box được cắm cờ).
   - `__undefined__` vẫn nằm trong dropdown list của CVAT và không chặn Save tự động.
   - Thứ tự giá trị `direction` để `general` sau `straight` dễ gây nhầm lẫn.

5. **Một thay đổi cụ thể giúp annotator mới ít hỏi hơn?**
   - Thêm vào §7 "Bảng quyết định nhanh" cho 3 tình huống vỡ: Khi không thấy vạch làn xe chủ; khi ban đêm vỏ vô hình; và quy chuẩn cho đèn người đi bộ.
   - Đổi `needs_review` từ checkbox sang dropdown bắt buộc chọn để không bị bỏ quên.

## 2. Owner phân loại

Owner không tranh luận để bảo vệ guideline. Mỗi feedback và mỗi decision peer làm sai được xếp vào một hướng xử lý.

| Feedback / decision sai | Nguyên nhân (guideline gap / data ambiguity / execution error) | Xử lý (accept + revise / reject with evidence / add escalation rule) | Bằng chứng |
|---|---|---|---|
| LISA20/25: Gán nhầm relevance giữa đèn cần vươn và đèn cột xa | Guideline gap: Chưa có quy tắc khi xe đứng sát vạch dừng không nhìn thấy vạch kẻ đường dưới bánh | Accept + Revise: Bổ sung "Quy tắc vị trí xe sát vạch dừng" vào §4 và §7 của Guideline v3 | LISA20 peer gán 2 đèn cần vươn là cross_traffic thay vì ego_lane |
| BDD26: Vẽ box bọc quầng chóa sáng bloom 39x53 px | Guideline gap: Chưa hướng dẫn fallback khi vỏ đèn ban đêm hoàn toàn vô hình | Accept + Revise: Cho phép vẽ bám sát tim bóng phát sáng thực tế khi vỏ chìm vào nền tối, cấm lấy bloom | BDD26 peer export box kích thước 39x53 px trùm toàn bộ ánh sáng phát tán |
| LISA28: Không gán uncertain và needs_review=true | Execution error & CVAT UI: Checkbox mặc định false khiến annotator bỏ qua cờ kiểm tra | Add escalation rule: Bắt buộc gán uncertain + needs_review khi góc chụp xiên > 45 độ | LISA28 peer có 0/5 box đánh dấu needs_review dù thừa nhận góc chụp khó |
| BDD18: Đèn người đi bộ số đếm ngược trắng gán off | Guideline gap: Taxonomy chưa định nghĩa rõ quy ước cho tín hiệu đi bộ đếm ngược | Accept + Revise: Quy ước đèn đếm ngược / chữ trắng thuộc cụm người đi bộ gán state = green, direction = general | BDD18 peer phải gán state = off cho số đếm ngược đang sáng |
