# Edge-case library

Thư viện 8 tình huống khó và điển hình cho bài toán Gắn nhãn trạng thái và mức độ điều khiển phương tiện của Đèn giao thông tại nút giao phức tạp.

---

CASE ID: CASE-01
Sample: BDD26
Scene: Ban đêm đường phố, đèn xanh sáng chói lóa trên cột bên trái
Observation: Đèn phát sáng mạnh tạo quầng sáng tán xạ (flare/bloom) hình tròn tỏa rộng, lấn át viền vỏ kim loại
Decision: LABEL
Expected: Bounding box chữ nhật ôm sát thân vỏ nhựa/kim loại đen nhìn thấy, tuyệt đối không bao quầng sáng chóa; state = green, relevance = ego_lane, direction = general
Rationale: Giúp mô hình nhận diện học được tọa độ vật lý chính xác của nguồn sáng thay vì bị đánh lừa bởi quầng tán xạ quang học của thấu kính camera
Common mistake: Kéo box vuông to bao trùm toàn bộ vệt sáng phát quang
Diversity: low_visibility, bloom, geometry

---

CASE ID: CASE-02
Sample: LISA25
Scene: Ban ngày xe tiếp cận ngã tư có giàn đèn treo ngang gồm 3 đầu đèn
Observation: Đầu đèn bên trái điều khiển làn rẽ/quay đầu, hai đầu đèn ở giữa và bên phải điều khiển luồng đi thẳng
Decision: LABEL
Expected: Đèn bên trái: relevance = cross_traffic, direction = left, state = red; Hai đèn thẳng: relevance = ego_lane, direction = general, state = red
Rationale: Ngăn chặn Critical Failure: nếu gán nhầm đèn rẽ trái thành ego_lane, khi đèn rẽ chuyển xanh mà đèn đi thẳng vẫn đỏ, xe tự hành sẽ phóng qua giao lộ gây tai nạn thảm khốc
Common mistake: Gán toàn bộ cả 3 đầu đèn thành ego_lane vì thấy treo chung trên một cần xà ngang
Diversity: critical, conflict

---

CASE ID: CASE-03
Sample: LISA28
Scene: Xe tiến sát góc giao lộ, góc nhìn camera bị nghiêng và khuất tầm vạch kẻ đường
Observation: Đầu đèn nằm ở rìa góc quan sát, khó phân định luồng xe điều khiển trực diện
Decision: ESCALATE
Expected: relevance = uncertain, needs_review = true, direction = general
Rationale: Khi bằng chứng phân làn không rõ ràng, việc cắm cờ escalation giúp dữ liệu được chuyển lên cấp Senior QA thẩm định thay vì đoán mò gây nhiễu dữ liệu huấn luyện
Common mistake: Tự ý phỏng đoán gán ego_lane mà không cắm cờ review
Diversity: ambiguity, escalation

---

CASE ID: CASE-04
Sample: BDD18
Scene: Ban đêm ngã tư lớn, có nhiều phương tiện và cột đèn bên phải
Observation: Trên cột bên phải có đèn tín hiệu dành cho người đi bộ ở tầm thấp (hình bàn tay/người đi bộ)
Decision: LABEL
Expected: relevance = pedestrian, state = red, direction = general
Rationale: Hệ thống xe tự hành cần phân biệt rạch ròi giữa đèn điều khiển xe cơ giới và đèn cho người đi bộ để tránh nhầm tín hiệu dừng của người đi bộ thành lệnh dừng xe trên làn
Common mistake: Bỏ qua không vẽ vì nghĩ chỉ gắn nhãn đèn ô tô, hoặc gán nhầm thành ego_lane
Diversity: pedestrian, small_far

---

CASE ID: CASE-05
Sample: BDD07
Scene: Ban ngày đường đô thị, hai đầu đèn treo ngang qua làn xe có vỏ sơn màu vàng New York
Observation: Vỏ hộp đèn có màu vàng sáng (thay vì màu đen/xám thông thường), cả hai đèn đang sáng màu xanh
Decision: LABEL
Expected: 2 box riêng biệt ôm khít từng vỏ hộp đèn màu vàng; state = green, relevance = ego_lane, direction = general
Rationale: Đảm bảo mô hình nhận thức bao quát được sự đa dạng về màu sắc thân vỏ đèn theo quy chuẩn từng địa phương (New York, California)
Common mistake: Vẽ 1 box gộp cả 2 đầu đèn vào nhau hoặc tưởng vỏ vàng là đèn cảnh báo
Diversity: color_variation, normal

---

CASE ID: CASE-06
Sample: LISA04
Scene: Ban ngày ngã tư rộng, trên cần vươn có biển báo phụ hình mũi tên quay đầu gắn cạnh đầu đèn bên trái
Observation: Đầu đèn ngoài cùng bên trái được bố trí liền kề biển chỉ dẫn phụ làn quay đầu
Decision: LABEL
Expected: Box chỉ ôm khít vỏ đèn, không bao biển báo phụ; direction = left, relevance = cross_traffic (đối với xe đang đi thẳng)
Rationale: Nhận biết chức năng rẽ riêng biệt thông qua biển phụ báo hiệu làn rẽ/quay đầu
Common mistake: Kéo box trùm cả biển báo phụ hình vuông gắn cạnh đèn
Diversity: turn_arrow, auxiliary_sign

---

CASE ID: CASE-07
Sample: BDD02
Scene: Phố đô thị một chiều nhiều làn, đèn treo ở cả cột bên trái và cột bên phải
Observation: Hai bên đường đều có cột đèn giao thông quay về cùng một hướng, mặt đường không có dải phân cách cứng
Decision: LABEL
Expected: Toàn bộ các đèn cùng hướng điều khiển luồng đi thẳng đều gán relevance = ego_lane
Rationale: Trên đường một chiều, đèn treo góc trái đóng vai trò nhắc lại tín hiệu cho các làn bên trái, vẫn trực tiếp điều khiển hành vi của xe chủ
Common mistake: Gán đèn cột bên trái thành cross_traffic do nhầm với đường hai chiều
Diversity: one_way_multilane, semantic_conflict

---

CASE ID: CASE-08
Sample: BDD25
Scene: Hoàng hôn chạng vạng, đại lộ nhiều tòa nhà cao tầng ngược sáng
Observation: Đèn xanh sáng ở khoảng cách trung bình, bầu trời sáng nhẹ nhưng mặt đường và các tòa nhà đổ bóng tối làm chìm viền vỏ đèn
Decision: LABEL
Expected: Box bám sát khối chữ nhật đen của vỏ đèn qua độ tương phản bóng tối; state = green, relevance = ego_lane
Rationale: Rèn luyện kỹ năng nhận diện cấu trúc vật lý của đèn trong điều kiện ánh sáng chạng vạng chuyển tiếp giữa ngày và đêm
Common mistake: Không nhìn thấy viền vỏ nên bỏ sót không gán nhãn
Diversity: low_visibility, silhouette
