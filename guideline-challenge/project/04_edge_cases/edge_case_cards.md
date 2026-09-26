# Edge-case library

Tối thiểu **8 card**, khuyến nghị 10–12. Một edge case tốt là case mà hai annotator hợp lý có thể làm khác nhau nếu
guideline chưa rõ. Tám ảnh dễ có label rõ ràng không được tính là edge-case library.

Cần có đủ độ đa dạng: occlusion / truncation / small-far · ambiguous semantics · conflicting road elements · **một case
critical-risk** · **một case guideline cho phép escalation**.

File này là kho nội bộ của nhóm, **không gửi cho peer**. Card dùng ảnh example/calibration thì chép rule + ví dụ sang
`02_guideline.md` (mục 7 và 9) để peer đọc được. Card về ảnh blind chỉ nằm ở đây, và decision của nó phải có trong
`gold_decisions.csv` trước `make freeze`.

`make status` đếm số dòng `CASE ID:` đã điền (đã thay placeholder).

---

CASE ID: EC01
Sample: BDD25
Scene: Giao lộ nhiều làn lúc chạng vạng (dawn/dusk), xe đang ở làn đi thẳng.
Observation: Trên giá long môn có 3 đầu đèn: 1 đầu đèn mũi tên rẽ trái đang ĐỎ (`state=red`, `pictogram=arrow_left`), 2 đầu đèn đi thẳng đang XANH (`state=green`, `pictogram=circle`).
Decision: LABEL
Expected: Vẽ 3 bounding box riêng biệt: Box 1 (mũi tên trái) gán `state=red`, `relevance=not_relevant`, `pictogram=arrow_left`; Box 2 & 3 (đi thẳng) gán `state=green`, `relevance=relevant`, `pictogram=circle`.
Rationale: Lỗi gán nhầm đèn rẽ trái thành `relevant` sẽ khiến hệ thống tự hành phanh gấp nguy hiểm; ngược lại gán đèn xanh đi thẳng vào làn rẽ trái sẽ gây vượt đèn đỏ cắt ngang dòng xe đối diện (Critical Failure).
Common mistake: Annotator chỉ vẽ 1 box to bao trùm toàn bộ giá đèn, hoặc gán nhầm tất cả các đèn thành `relevance=relevant`.
Diversity: critical, conflict

---

CASE ID: EC02
Sample: BDD18
Scene: Ngã tư đô thị ban đêm, camera bị lóa đèn.
Observation: Đèn tín hiệu ban đêm phát ra quầng sáng đỏ rực (halo/glare) tỏa rộng gấp đôi kích thước phần cứng của đầu đèn.
Decision: LABEL
Expected: Vẽ bounding box chữ nhật ôm sát mép vỏ kim loại (housing) nhìn thấy của đầu đèn, không bao gồm quầng sáng lóa; `state=red`, `relevance=relevant`, `pictogram=circle`, `needs_review=false`. Dung sai biên ≤ 3px.
Rationale: Module Perception cần vị trí và kích thước vật lý chuẩn xác của đèn để tính toán khoảng cách 3D (Depth estimation). Box bị phình to sẽ làm thuật toán ước tính xe đang ở gần đèn hơn thực tế.
Common mistake: Kéo box to ra ngoài viền vỏ kim loại để ôm trọn toàn bộ quầng sáng phát quang.
Diversity: critical, low_visibility, geometry

---

CASE ID: EC03
Sample: BDD10
Scene: Đường phố ban ngày, có hàng cây rậm rạp và xe tải đỗ bên lề.
Observation: Đầu đèn bên phải bị cành cây che khuất khoảng 40% phần thân vỏ, nhưng bóng đèn tròn màu vàng vẫn nhìn thấy rõ ràng.
Decision: LABEL
Expected: Vẽ box ôm phần vỏ đèn nhìn thấy và ước lượng nhẹ phần bị che lấp không quá 10px; gán `state=yellow`, `relevance=relevant`, `pictogram=circle`, bật checkbox `needs_review=true`.
Rationale: Đèn bị che một phần nhưng tín hiệu vẫn có hiệu lực pháp lý điều khiển xe. Việc bật `needs_review=true` giúp hệ thống QA và model nhận diện dữ liệu có độ tự tin trung bình.
Common mistake: Bỏ qua không vẽ vì thấy bị che, hoặc vẽ box kéo quá rộng bao trùm cả cành cây.
Diversity: occlusion, escalation

---

CASE ID: EC04
Sample: BDD02
Scene: Ngã tư người đi bộ qua đường (Crosswalk) tại khu dân cư đông đúc.
Observation: Trên cột tín hiệu góc vỉa hè có đầu đèn nhỏ hiển thị biểu tượng người đi bộ màu đỏ/xanh (Pedestrian signal head).
Decision: LABEL
Expected: Vẽ box ôm đầu đèn người đi bộ; gán `state=red` (hoặc green tùy bóng sáng), `relevance=not_relevant`, `pictogram=other`, `needs_review=false`.
Rationale: Downstream planning chỉ dùng đèn có `relevance=relevant` để ra lệnh phanh/ga cho xe ego. Đèn người đi bộ cần được nhận diện để hiểu hành vi người đi bộ nhưng KHÔNG ĐƯỢC dùng làm tín hiệu điều khiển trực tiếp cho xe.
Common mistake: Bỏ qua không vẽ dẫn tới false negative, hoặc gán `relevance=relevant` làm xe dừng sai luật khi đèn người đi bộ đỏ nhưng đèn phương tiện đang xanh.
Diversity: ambiguity, conflict

---

CASE ID: EC05
Sample: BDD04
Scene: Giao lộ phức tạp, xe đang dừng chờ tại vạch dừng làn thẳng.
Observation: Cột đèn giao thông của tuyến đường cắt ngang (vuông góc với xe) quay mặt hơi chếch về phía camera, bóng đèn đỏ đang sáng.
Decision: LABEL
Expected: Vẽ box ôm đầu đèn; gán `state=red`, `relevance=not_relevant`, `pictogram=circle`, `needs_review=false`.
Rationale: Tránh nhầm lẫn đèn của hướng giao cắt thành đèn của xe mình. Đèn này điều khiển dòng xe vuông góc, xe ego tuyệt đối không được tuân theo đèn này.
Common mistake: Thấy đèn đỏ sáng rõ liền gán `relevance=relevant` khiến xe đứng im không di chuyển dù đèn trên làn của mình đã chuyển xanh.
Diversity: conflict, ambiguity

---

CASE ID: EC06
Sample: BDD12
Scene: Đường phố góc nhìn xa, xe cách giao lộ hơn 150 mét.
Observation: Có 2 cụm đèn tín hiệu ở phía chân trời, chiều cao trên ảnh chỉ đạt khoảng 8–10 pixel, chỉ thấy một chấm màu mờ nhạt không rõ hình vỏ đèn.
Decision: IGNORE
Expected: Không tạo bounding box cho các đầu đèn có chiều cao < 15 pixel.
Rationale: Quy chuẩn kỹ thuật (cut-off threshold): Đèn dưới 15px không đủ pixel để trích xuất đặc trưng tin cậy, nếu ép annotator vẽ sẽ sinh ra hàng loạt box nhiễu, làm giảm độ chính xác IoU và khiến model học đặc trưng ảo.
Common mistake: Cố gắng zoom cực đại 500% rồi chấm 1 box 6x8 px không thể kiểm chứng.
Diversity: small_far, ambiguity

---

CASE ID: EC07
Sample: BDD11
Scene: Giao lộ đang bảo trì hoặc mất điện vào ban ngày.
Observation: Cụm đèn tín hiệu giao thông không sáng bóng nào (toàn bộ 3 mắt đen ngòm), hoặc đèn vàng nhấp nháy cảnh báo giảm tốc độ.
Decision: LABEL
Expected: Nếu đèn tắt hoàn toàn: vẽ box ôm vỏ, gán `state=off`, `relevance=relevant`, `pictogram=circle`. Nếu nhấp nháy vàng: gán `state=yellow`, `relevance=relevant`, `pictogram=circle`.
Rationale: Xe tự hành cần biết có cột đèn nhưng đang không hoạt động (`state=off`) để chuyển sang chế độ tuân thủ biển báo phụ hoặc quy tắc nhường đường ngã tư không đèn.
Common mistake: Nghĩ đèn tắt là không cần vẽ nên bỏ qua (IGNORE), hoặc đoán mò thành `state=unknown`.
Diversity: ambiguity, escalation

---

CASE ID: EC08
Sample: BDD17
Scene: Trời mưa lớn (Rainy daytime), gạt nước chưa quét kịp làm nước đọng thành vệt lượn sóng trên kính chắn gió.
Observation: Đầu đèn tín hiệu phía trước bị khúc xạ qua giọt nước mưa làm méo mó hình dạng, biên vỏ đèn bị nhòe và màu sắc pha tạp giữa vàng và đỏ.
Decision: ESCALATE
Expected: Vẽ box bao quanh vùng vỏ đèn nhận diện được; gán `state=unknown`, `relevance=relevant`, `pictogram=circle`, bật checkbox `needs_review=true`.
Rationale: Khi bằng chứng thị giác bị biến dạng nghiêm trọng bởi thời tiết, guideline yêu cầu không suy đoán chủ quan; phải gắn cờ `unknown` + `needs_review` để thuật toán dung hợp cảm biến (sensor fusion - radar/lidar/bản đồ HD) xử lý thay vì tin vào camera đơn lẻ.
Common mistake: Tự suy đoán trạng thái đèn theo kinh nghiệm cá nhân rồi gán liều `state=red` hoặc `state=yellow`.
Diversity: low_visibility, escalation

---

CASE ID: EC09
Sample: LISA15
Scene: Clip video xe tiếp cận giao lộ, góc quay đổi dần qua các frame.
Observation: Một cột đèn có cụm 2 đầu đèn gắn sát nhau theo phương thẳng đứng (1 đầu đèn tròn chính, 1 đầu đèn phụ mũi tên rẽ phải bên dưới).
Decision: LABEL
Expected: Vẽ 2 bounding box (Track) riêng biệt cho 2 đầu đèn; không vẽ 1 box gộp cả cụm. Gán riêng từng `state` và `pictogram` cho từng box.
Rationale: Mỗi đầu đèn vật lý có chu kỳ tín hiệu độc lập. Gom 2 đầu đèn vào chung 1 box sẽ phá vỡ định nghĩa instance và khiến hệ thống không thể phân tách luồng điều khiển rẽ phải và đi thẳng.
Common mistake: Vẽ 1 box to bao bọc cả 2 đầu đèn trên cùng một giá treo.
Diversity: conflict, geometry

---

CASE ID: EC10
Sample: BDD24
Scene: Bão tuyết phủ dày đặc (Snowy daytime), sương tuyết và bùn bám kín góc camera.
Observation: Toàn bộ góc trên bên phải của ảnh bị tuyết che lấp hoàn toàn, không thể xác định có đèn giao thông hay không dù ngã tư có biển báo.
Decision: ESCALATE
Expected: Không vẽ box mò vào vùng tuyết; gán nhãn tag toàn cảnh `image_escalate` cho frame ảnh.
Rationale: Tránh tạo ra các annotation "ma" (hallucinated labels) dựa trên phán đoán ngẫu nhiên. Tag ảnh cảnh báo pipeline huấn luyện loại bỏ frame lỗi khỏi tập train.
Common mistake: Nhìn thấy cột mờ mờ trong tuyết rồi tự tưởng tượng ra đầu đèn và chấm box.
Diversity: low_visibility, escalation
