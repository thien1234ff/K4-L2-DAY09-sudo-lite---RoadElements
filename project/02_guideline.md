# Annotation guideline — Trạng thái & tính liên quan của Đèn giao thông tại nút giao phức tạp

**Version:** v2

## 1. Objective + scope

- **Mục tiêu:** Nhận diện toàn bộ các đầu đèn tín hiệu giao thông, xác định trạng thái màu sắc (`state`), mức độ điều khiển phương tiện đối với xe chủ (`relevance`), và hướng điều khiển (`direction`) phục vụ hệ thống điều khiển xe tự hành an toàn qua nút giao.
- **Trong scope (bắt buộc label):**
  - Mọi đầu đèn tín hiệu giao thông (bao gồm đèn dành cho phương tiện giao thông và đèn dành cho người đi bộ) có mặt thấu kính/bóng đèn quay về hướng xe chủ đang quan sát.
  - Kích thước phần vỏ hộp đèn nhìn thấy tối thiểu từ $8 \times 8\text{ px}$ trở lên.
- **Ngoài scope (bỏ qua / ignore):**
  - Đèn quay lưng (chỉ thấy nắp lưng hộp đèn hoặc kết cấu kim loại mặt sau).
  - Đèn quay ngang 90 độ phục vụ luồng giao thông cắt ngang không chiếu về phía camera.
  - Đèn tín hiệu đường sắt, đèn công trường thi công di động đặt tạm trên mặt đất, đèn cảnh báo nhấp nháy 1 bóng độc lập.
  - Cột đèn, tay xà đòn ngang, biển báo phụ gắn tách rời ngoài vỏ đèn.
  - Đèn quá xa hoặc quá mờ có kích thước $< 8 \times 8\text{ px}$.
  - Hình ảnh đèn phản chiếu trên nắp ca-pô, kính xe phía trước hoặc mặt đường ướt.

## 2. Annotation unit

- **Đơn vị gắn nhãn:** Từng đầu đèn độc lập (Single traffic light head/housing). 
- **Quy tắc tách instance:** Mỗi hộp đèn vật lý chứa một cụm bóng đèn (ví dụ: hộp 3 bóng Đỏ-Vàng-Xanh, hoặc hộp 1 bóng mũi tên rẽ phụ) là một instance chữ nhật (`rectangle`) riêng biệt. Tuyệt đối **không** gom nhiều đầu đèn treo cạnh nhau thành một box lớn duy nhất.
- **Chế độ:** Ảnh tĩnh độc lập (`Shape` mode).

## 3. Geometry rule

- **Công cụ:** Draw new rectangle (phím tắt trên CVAT).
- **Quy tắc bao khít vỏ đèn (Tight visible bounding box):**
  - Bounding box phải ôm khít đường biên ngoài của phần vỏ cứng kim loại/nhựa (`housing`) nhìn thấy được của đầu đèn.
  - Bao gồm cả phần nắp che che nắng (visors/hoods) gắn liền trên đỉnh các bóng đèn nếu có.
- **Quy tắc quầng sáng (NO bloom / glare):**
  - Tuyệt đối **không** kéo box bao phủ quầng sáng chóa, tia sáng tán xạ quang học (lens flare/bloom) phát ra từ bóng đèn, đặc biệt trong ảnh ban đêm hoặc thời tiết mưa ẩm. Chỉ ôm sát phần vỏ cứng vật lý.
- **Dung sai hình học (Geometry tolerance):** Lệch không quá $\le 3\text{ px}$ ở mỗi cạnh so với mép vỏ đèn thực tế.

## 4. Taxonomy

### Class duy nhất
- `traffic_light` (Shape: `rectangle`)

### Attributes bắt buộc
Mỗi box `traffic_light` phải được gán đầy đủ các thuộc tính sau (giá trị mặc định ban đầu là `__undefined__`, người vẽ bắt buộc phải chọn):

1. **`state`** (Trạng thái tín hiệu bóng đèn):
   - `red`: Đèn đỏ đang sáng (bóng tròn hoặc mũi tên đỏ).
   - `yellow`: Đèn vàng đang sáng (bóng tròn hoặc mũi tên vàng).
   - `green`: Đèn xanh đang sáng (bóng tròn hoặc mũi tên xanh).
   - `off`: Đầu đèn đang tắt hoàn toàn (không có bóng nào phát sáng).
   - `unknown`: Không thể xác định màu do chóa lóa sáng, ngược sáng quá nặng, hoặc bị bóng lá che khuất tim đèn.

2. **`relevance`** (Tính liên quan tới làn xe chủ - Ego-relevance):
   - `ego_lane`: Đèn đang trực tiếp điều khiển làn đường mà xe chủ đang di chuyển (đèn treo thẳng trên nóc làn, hoặc cột đèn điều khiển xe đi thẳng/làn hiện tại). *Quy tắc bổ sung v2:* Trên đường một chiều có nhiều làn (như `BDD02`), nếu các đầu đèn ở cột trái và cột phải cùng đồng bộ một trạng thái tín hiệu cho cả mặt đường thì tất cả đều là `ego_lane`.
   - `cross_traffic`: Đèn điều khiển xe ở luồng cắt ngang, xe ngược chiều, hoặc làn rẽ phụ đã có dải phân cách/vạch rẽ riêng biệt không thuộc hướng đi của xe chủ. *Quy tắc bổ sung v2:* Trên giàn đèn ngang ngã tư (như `LISA04`, `LISA07`), đầu đèn nào nằm cạnh biển phụ chỉ dẫn rẽ/quay đầu trái thì gán `cross_traffic` đối với xe đang đi thẳng.
   - `pedestrian`: Đèn tín hiệu dành riêng cho người đi bộ (có hình người đi bộ hoặc đèn kích thước nhỏ treo tầm thấp trên vỉa hè).
   - `uncertain`: Nút giao quá phức tạp, mất vạch kẻ đường, góc chụp nghiêng không thể khẳng định chắc chắn đèn thuộc làn nào.

3. **`direction`** (Hướng lưu thông chỉ định):
   - `straight`: Đèn có ký hiệu mũi tên đi thẳng.
   - `left`: Đèn có ký hiệu mũi tên rẽ trái (hoặc đầu đèn cạnh biển báo rẽ trái chuyên biệt).
   - `right`: Đèn có ký hiệu mũi tên rẽ phải.
   - `general`: Đèn bóng tròn tiêu chuẩn điều khiển luồng phương tiện chung.

4. **`needs_review`** (Cờ thẩm định - Checkbox):
   - `false` (mặc định): Tự tin với quyết định.
   - `true`: Đánh dấu cần Senior Reviewer / QA kiểm tra lại (dùng khi gặp trường hợp mập mờ, tranh chấp làn).

### Tag cấp ảnh
- `image_escalate` (Type: `tag`): Gán cho cả ảnh khi điều kiện thời tiết/chất lượng ảnh quá tệ (nhòe mờ toàn bộ, chóa lóa toàn bộ khung hình không thể định vị giao lộ).

## 5. Inclusion / exclusion

- **Bắt buộc gắn nhãn (Inclusion):**
  - Mọi đầu đèn có mặt đèn quay về phía xe trong phạm vi quan sát rõ (kích thước $\ge 8\times 8\text{ px}$).
  - Đèn cho người đi bộ quay về hướng xe chủ.
  - Đèn gắn trên cột phụ hoặc treo trên dây cáp ngang giữa ngã tư.
- **Bỏ qua không vẽ (Exclusion):**
  - Đèn chỉ quay mặt sau/mặt bên (lưng đèn màu đen/xám).
  - Đèn sau xe khác (đèn hậu ô tô, đèn phanh).
  - Đèn quá nhỏ dưới 8 pixel hoặc đèn mờ tịt ở hậu cảnh xa xôi.

## 6. Visibility / occlusion

- **Bị che khuất một phần (Partial Occlusion):**
  - Nếu đầu đèn bị cành cây, dây điện, cột biển báo hoặc xe tải che khuất $\le 50\%$ diện tích vỏ: Kéo bounding box bao trùm toàn bộ phần vỏ cứng nhìn thấy được (Visible bounding box).
  - Nếu bị che khuất $> 50\%$ diện tích hoặc tim bóng đèn bị che hoàn toàn không đoán được hình dạng vỏ: Bỏ qua (Ignore).
- **Bị cắt mép ảnh (Truncation):**
  - Nếu đầu đèn bị mép ảnh cắt ngang nhưng phần vỏ nằm trong ảnh vẫn $\ge 8\times 8\text{ px}$: Kéo box sát mép biên của khung hình.
- **Ảnh ban đêm & Chóa đèn (Nighttime & Glare):**
  - Khi bóng đèn phát sáng tạo quầng sáng tỏa tròn rực rỡ, hãy zoom to ảnh (150% - 300%) để tìm viền vỏ kim loại/nhựa hình chữ nhật phía sau quầng sáng. Vẽ bám sát viền vỏ đó.

## 7. Ambiguity / escalation

Quy tắc xử lý khi bằng chứng không rõ ràng:
1. **Thấy vỏ đèn nhưng bóng đèn bị chóa trắng xóa không rõ màu:** Gán `state = unknown`, tích `needs_review = true`.
2. **Nhiều đầu đèn treo ngang trên giá long môn không rõ đèn nào của làn mình:** 
   - Nếu có biển chỉ dẫn làn đi kèm: Đối chiếu hướng làn xe đang đứng với biển chỉ dẫn.
   - Nếu không có biển và vạch phân làn bị mờ: Gán `relevance = uncertain`, tích `needs_review = true`.
3. **Ảnh hỏng/chóa sáng toàn bộ:** Đặt nhãn Tag `image_escalate` cho ảnh ở công cụ Setup tag.

## 8. Temporal rule

- **Quy tắc thời gian:** Nhiệm vụ này thực hiện theo phương thức **ảnh tĩnh độc lập (Frame-by-frame annotation)**.
- Đối với chuỗi frame liên tiếp trong dữ liệu LISA (`LISA01` – `LISA30`):
  - Vẽ từng frame bằng chế độ `Shape: rectangle`. Không tạo track tự động kéo dài giữa các frame để tránh trôi lệch vị trí bounding box khi xe di chuyển xóc nảy.
  - Khi xe tiến gần đến ngã tư qua từng frame, kích thước bounding box phải được phóng to tương ứng theo mép vỏ đèn.

## 9. Examples

Dưới đây là các ví dụ minh họa chuẩn từ tập ảnh `example` và `calibration`:

| sample_id | Thấy gì trên ảnh | Expected output | Rule áp dụng |
|---|---|---|---|
| `BDD07` | Ban ngày đường phố, 2 đầu đèn vỏ vàng New York treo ngang làn xe | Vẽ 2 box `traffic_light`, `state = green`, `relevance = ego_lane`, `direction = general` | Tách riêng từng đầu đèn; ôm sát vỏ vàng kim loại |
| `LISA01` | Ban ngày ngã tư rộng, giàn đèn đỏ 3 đầu đèn treo trên cần vươn | Vẽ 3 box riêng biệt ôm khít từng hộp đèn, không gom cụm | Quy tắc instance đơn lẻ; ôm sát viền vỏ |
| `LISA04` | Xe tiến gần ngã tư, thấy rõ đèn mũi tên rẽ trái và đèn đi thẳng | 1 box rẽ trái (`direction = left`), 2 box đi thẳng (`direction = general`/`straight`) | Phân định hướng di chuyển theo mũi tên thấu kính |
| `BDD25` | Chạng vạng hoàng hôn, đèn xanh sáng nhẹ ở tim đèn trên đại lộ | Box ôm sát thân vỏ đèn đen, không vẽ tràn ra quầng sáng chóa | Quy tắc Geometry: No bloom/glare |
| `BDD02` | Góc ngã tư đô thị, đèn giao thông treo trên cột và cần vươn | Vẽ các box cho đèn quay về hướng xe, bỏ qua đèn quay góc 90 độ | Exclusion rule: Chỉ vẽ đèn chiếu về hướng xe chủ |

## 10. Common mistakes

1. **Lỗi nghiêm trọng nhất (Critical Failure):** Gán đèn đỏ của hướng đi thẳng thành đèn xanh của làn rẽ phụ (hoặc gán `cross_traffic` nhầm thành `ego_lane`). *Cách tránh: Luôn nhìn xuống vạch kẻ đường dưới bánh xe để định vị làn xe mình đang chạy trước khi gán relevance*.
2. **Lỗi gom cụm (Clustering error):** Vẽ 1 khung to bao trọn cả giá treo chứa 3 đầu đèn. *Cách tránh: Mỗi vỏ hộp đèn chữ nhật là 1 box riêng*.
3. **Lỗi ôm quầng sáng (Bloom error):** Vẽ box hình vuông to đùng bao trọn vệt sáng chóa ban đêm thay vì hình chữ nhật đứng của thân vỏ đèn. *Cách tránh: Luôn zoom ảnh để tìm mép nắp che nắng hoặc cạnh vỏ*.
4. **Lỗi quên chọn thuộc tính:** Để nguyên giá trị `__undefined__`. *Cách tránh: Luôn kiểm tra sidebar Objects trước khi bấm Save (Ctrl+S)*.
