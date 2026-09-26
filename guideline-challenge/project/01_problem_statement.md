# Problem statement + downstream contract

## Bài toán

Gắn nhãn trạng thái và mức độ điều khiển phương tiện (Ego-relevance) của đèn tín hiệu giao thông tại ngã tư có nhiều đầu đèn và nhánh rẽ.

## Downstream contract

1. **Downstream task / model / user là ai?**
   Module Nhận thức (Perception: Traffic Light Detection & Classification) phối hợp với Module Lập kế hoạch hành vi (Behavior Planning) của hệ thống xe tự hành (Autonomous Vehicle cấp độ L3/L4), phục vụ việc ra quyết định dừng, đi tiếp hoặc rẽ an toàn tại các nút giao phức tạp.

2. **Output annotation nào thực sự cần?**
   - **Geometry:** Bounding box hình chữ nhật (`rectangle`) ôm khít phần vỏ cứng nhìn thấy (`housing`) của từng đầu đèn.
   - **Class:** Nhãn duy nhất `traffic_light`.
   - **Attributes bắt buộc:**
     - `state`: `red`, `yellow`, `green`, `off`, `unknown`.
     - `relevance`: `ego_lane` (điều khiển trực tiếp làn xe mình đang đi), `cross_traffic` (hướng cắt ngang hoặc xe ngược chiều), `pedestrian` (đèn dành riêng cho người đi bộ), `uncertain` (không đủ căn cứ phân làn).
     - `direction`: `straight` (đèn mũi tên/ký hiệu đi thẳng), `left` (rẽ trái), `right` (rẽ phải), `general` (đèn tròn chung).
     - `needs_review`: checkbox đánh dấu cờ cần thẩm định (`true`/`false`).
   - **Tag cấp ảnh:** `image_escalate` dùng khi ảnh mất thông tin nghiêm trọng.

3. **Failure nào gây hậu quả lớn nhất? (Critical Failure)**
   Gán nhầm đèn đỏ của làn xe mình đang chạy (`relevance = ego_lane`, `state = red`) thành đèn xanh của làn rẽ phụ/hướng cắt ngang (`cross_traffic` hoặc turn lane), hoặc gán nhầm đèn đang điều khiển xe mình thành không liên quan. Hậu quả: Xe tự hành hiểu sai tín hiệu được phép đi, vượt đèn đỏ tại giao lộ đông đúc và dẫn đến tai nạn đâm va trực diện nghiêm trọng.

4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?**
   - Trên cấp Object: Khi đèn bị chóa lóa quang học nặng, bị cành cây/xe tải che khuất, hoặc góc ngã tư quá phức tạp không xác định được làn: Người vẽ gán `relevance = uncertain` (hoặc `state = unknown`), đồng thời tích chọn `needs_review = true`.
   - Trên cấp Image: Nếu toàn cảnh nút giao bị che mờ hoặc chóa sáng toàn bộ không thể xác định luồng giao thông, người vẽ gắn nhãn tag `image_escalate` cho ảnh.
   - Các trường hợp này được xuất ra trong file thẩm định để Lead Annotator / Senior QA xử lý và bổ sung vào thư viện edge cases.

## Scope

- **Trong scope (bắt buộc label):**
  Mọi đầu đèn tín hiệu giao thông (dành cho phương tiện và người đi bộ) có mặt đèn (thấu kính/bóng đèn) quay về phía xe chủ và có kích thước tối thiểu từ 8x8 pixel trở lên trong tầm quan sát.
- **Ngoài scope (ignore):**
  - Đèn quay lưng hoàn toàn về phía xe (chỉ thấy nắp lưng màu đen/xám không phát tín hiệu về hướng mình).
  - Đèn tín hiệu đường sắt, đèn cảnh báo công trường di động đặt tạm dưới mặt đất, đèn cảnh báo nguy hiểm một bóng nhấp nháy độc lập.
  - Cột đèn, cần treo kim loại xà ngang, và các biển báo phụ gắn tách rời bên ngoài vỏ đèn.
  - Đèn quá xa hoặc quá mờ có kích thước nhỏ hơn 8x8 pixel không thể nhận diện được hình dạng cấu trúc vỏ đèn.
- **Geometry tolerance:**
  Bounding box hình chữ nhật phải ôm sát phần vỏ cứng nhìn thấy (`housing`) của đầu đèn. Sai số cho phép lệch $\le 3\text{ px}$ mỗi cạnh. Tuyệt đối không vẽ bao trùm quầng sáng tán xạ (glare/bloom/flare) phát ra từ bóng đèn.

## Output chấm được

Mọi quyết định đều được thể hiện tường minh và đọc được từ file export XML của CVAT:
- **LABEL:** Khung chữ nhật `traffic_light` mang đủ các thuộc tính `state`, `relevance`, `direction`.
- **IGNORE:** Các đèn ngoài scope (quay lưng, quá nhỏ < 8px) hoàn toàn không có bounding box.
- **UNKNOWN:** Thuộc tính `state = unknown`.
- **ESCALATE:** Thuộc tính `relevance = uncertain` hoặc checkbox `needs_review = true` trên object, hoặc tag ảnh `image_escalate`.
- **GEOMETRY:** Tọa độ $(xtl, ytl, xbr, ybr)$ của bounding box khớp với vỏ đèn trong dung sai $\le 3\text{ px}$.

## Dữ liệu và giới hạn

- **Nguồn ảnh:** Dữ liệu sẵn có trong repo:
  - `data/bdd100k/`: 12 ảnh có đèn tín hiệu giao thông bao gồm ban ngày, chạng vạng (`BDD25`), ban đêm (`BDD18`, `BDD26`), và trời mưa (`BDD17`).
  - `data/lisa/`: 30 frame liên tiếp (`LISA01` – `LISA30`) ghi lại cảnh xe tiếp cận ngã tư có dàn đèn treo ngang phức tạp.
- **Giới hạn đã biết:** 
  - Dữ liệu LISA là chuỗi ảnh liên tiếp từ một video clip đơn lẻ, do đó khi chọn ảnh cho tập `blind` phải cách xa các frame trong tập `example`/`calibration` tối thiểu > 2 frame để chống rò rỉ thông tin ngữ cảnh thời gian (temporal leakage).
  - Dữ liệu BDD100k chụp ban đêm có hiện tượng chóa lóa (flare) mạnh, đòi hỏi annotator tuân thủ nghiêm ngặt quy tắc bám sát vỏ cứng thay vì quầng sáng.
