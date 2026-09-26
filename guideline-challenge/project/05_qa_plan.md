# QA plan + quality gates

## Flow

Quy trình kiểm soát chất lượng dữ liệu của nhóm:
`Guideline → Calibration → Production → Self-QC → Review → Rework → Quality Gate`.

- **Ai review, review bao nhiêu:**
  QA Owner (**Lê Nguyễn Hà My**) cùng Lead Annotator (**Hoàng Kim Thiên**) thực hiện review độc lập.
  - Review 100% đối với các frame rủi ro cao (ban đêm `BDD18`, `BDD26`, chạng vạng `BDD25`, ngã tư dàn đèn phức tạp `LISA`).
  - Review 50% ngẫu nhiên đối với các frame ban ngày thông thường.
  - Review 100% các object có cắm cờ `needs_review = true` hoặc ảnh có tag `image_escalate`.
- **Chọn sample theo rule nào:**
  Áp dụng lấy mẫu phân tầng theo mức độ rủi ro (Risk-based Stratified Sampling):
  1. Ưu tiên toàn bộ các frame mang tag `critical`, `edge`, `ambiguity`, `low_visibility`.
  2. 100% sản phẩm của annotator mới hoặc annotator có tỷ lệ bất đồng cao ở vòng calibration.
  3. Lấy mẫu ngẫu nhiên 20% các frame còn lại sau khi annotator đã hoàn tất khâu Self-QC.
- **Issue được ghi ở đâu, đóng thế nào:**
  - Issue được tạo trực tiếp trên CVAT task qua tính năng Review & Issue tracking gắn vào từng bounding box.
  - Tổng hợp danh mục lỗi và nguyên nhân vào bảng theo dõi chất lượng của nhóm.
  - Đóng issue: Annotator sửa lỗi (rework) trực tiếp trên frame, reviewer kiểm tra lại nếu đạt chuẩn sẽ chuyển trạng thái issue thành `Resolved`.
- **Khi phát hiện guideline gap thì update và version ra sao:**
  - Khi phát hiện tình huống mới chưa có quy tắc rõ ràng: Tạm dừng gán nhãn các đối tượng tương tự, QA Owner cùng Spec Owner thảo luận nhanh 5 phút để chốt quy tắc xử lý.
  - Cập nhật quy tắc mới vào `02_guideline.md`, tăng version (`v1` → `v2` → `v3`).
  - Ghi nhận chi tiết nguyên nhân và bằng chứng vào `08_revision_log.md`.
  - Thông báo toàn nhóm và dán lại nội dung guideline mới vào phần Description/Guide của task CVAT.

## Defect severity

Bảng phân loại mức độ nghiêm trọng của lỗi gán nhãn cho bài toán đèn giao thông xe tự hành:

| Severity | Định nghĩa cho project này | Ví dụ | Action mặc định |
|---|---|---|---|
| Critical | Lỗi sai lệch nghiêm trọng dẫn đến việc xe tự hành ra quyết định di chuyển sai lầm gây tai nạn đâm va trực diện tại giao lộ | Gán nhầm đèn đỏ đi thẳng (`ego_lane`, `red`) thành đèn xanh của làn rẽ phụ (`cross_traffic`, `green`); Bỏ sót hoàn toàn đầu đèn đỏ đang điều khiển làn xe chủ | Reject toàn bộ batch; Dừng việc để đào tạo lại annotator; Kiểm tra 100% toàn bộ task của người đó |
| Major | Lỗi phân loại sai thuộc tính trạng thái/hướng hoặc vẽ box sai lệch đáng kể ảnh hưởng đến độ tin cậy của hệ thống nhận thức | Nhầm giữa `off` và `unknown`; Bỏ sót đèn người đi bộ; Box vẽ bao trùm quầng sáng chóa (bloom/flare) lệch > 5px so với vỏ đèn | Trả về yêu cầu Rework ngay lập tức trên các frame bị lỗi; Tăng tỷ lệ kiểm tra ngẫu nhiên thêm 30% |
| Minor | Sai số hình học nhỏ về vị trí viền vỏ đèn không làm thay đổi ngữ nghĩa hoặc quyết định lái xe | Box lệch viền vỏ đèn từ 3px đến 5px nhưng không bao trùm quầng sáng; Box hơi rộng ở phần nắp che nắng (visor) | Reviewer tự chỉnh sửa trực tiếp (Quick-fix); Nhắc nhở annotator rút kinh nghiệm ở batch sau |
| Question | Tình huống mập mờ, bị che khuất hoặc chóa sáng nặng mà người vẽ đã chủ động gắn cờ để xin ý kiến | Đèn bị cành cây/xe tải che khuất 50% diện tích vỏ; Ngã tư 5 nhánh không rõ đèn thuộc làn nào | QA Owner cùng nhóm trưởng phân tích ảnh gốc, đưa ra quyết định chốt hoặc gán `uncertain`/`image_escalate`, đưa vào `edge_case_cards.md` |

## Metrics

| Metric | Cách tính | Vì sao phù hợp với bài toán |
|---|---|---|
| **Decision Accuracy (DA)** | `(Số quyết định đúng / Tổng số quyết định kiểm tra) * 100%` | Đánh giá tổng quát độ chính xác của nhãn class, trạng thái `state`, tính liên quan `relevance` và hướng `direction` |
| **Critical Defect Escape Rate (CDER)** | `(Số lỗi Critical phát hiện sau Self-QC / Tổng số đối tượng) * 100%` | Chỉ số sống còn đo lường độ an toàn tuyệt đối cho hệ thống xe tự hành (bắt buộc = 0%) |
| **Geometry IoU & Boundary Error** | IoU trung bình so với viền vỏ cứng; Tỷ lệ box có sai số mỗi cạnh $\le 3\text{px}$ | Đảm bảo kích thước và vị trí box phản ánh đúng vỏ đèn thực tế, không bị méo mó do quầng sáng |

Metric high-risk tách riêng:
- **Chỉ tiêu Critical Defect:** Bắt buộc tuyệt đối **0 lỗi Critical** (`Critical Error Count = 0`). Chỉ cần phát hiện 1 lỗi Critical lọt qua khâu Self-QC, toàn bộ task sẽ bị giữ lại tại Quality Gate.

## Quality gate

Threshold kiểm định chất lượng trước khi bàn giao và nộp bài:

```text
PASS if:
  - Critical Defect Escape Rate = 0% (tuyệt đối không có lỗi Critical).
  - Decision Accuracy (DA) >= 92%.
  - Geometry Boundary Compliance: >= 95% số bounding box có độ lệch biên <= 3px.
  - 100% các cờ needs_review và Question đã được giải quyết dứt điểm.
  - 0% thuộc tính còn sót giá trị mặc định __undefined__.

REWORK if:
  - Xuất hiện lỗi Major trong khoảng từ 1 đến 2 lỗi trên mỗi 10 frame, HOẶC
  - Decision Accuracy nằm trong khoảng 85% <= DA < 92%, HOẶC
  - Có bounding box vẽ bao trùm quầng sáng chóa ban đêm.
  -> Action: Annotator phải thực hiện sửa chữa (rework) và nộp lại trong vòng 15 phút.

REJECT / ESCALATE if:
  - Xuất hiện >= 1 lỗi Critical (nhầm đèn đỏ xe mình thành đèn xanh làn rẽ), HOẶC
  - Decision Accuracy < 85%, HOẶC
  - Còn từ 3 box trở lên chưa gán thuộc tính (bỏ quên __undefined__).
  -> Action: Hủy kết quả kiểm duyệt của batch, coaching trực tiếp 10 phút, gán nhãn lại toàn bộ.
```

Trade-off:
Nhóm đặt mục tiêu **An toàn tính mạng là tối thượng (Zero-tolerance với Critical Defect)**. Do đó, chấp nhận chi phí thời gian cao hơn để annotator zoom lớn ảnh (200% - 300%) phân tách kỹ viền vỏ kim loại trong điều kiện đêm chóa sáng, đồng thời đối chiếu vạch kẻ làn đường để xác định chính xác `ego_lane`. Đối với các sai lệch Minor vài pixel viền nắp che nắng, nhóm chấp nhận nới lỏng dung sai để đảm bảo tốc độ và năng suất hoàn thành đúng timeline 240 phút.
