# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` khớp từng dòng ở đây.

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `traffic_light` | rectangle | class | N/A | N/A | false | Đối tượng đầu đèn vật lý duy nhất cần nhận diện và khoanh vùng trên ảnh |
| `state` | N/A | attribute | `__undefined__`, `red`, `yellow`, `green`, `off`, `unknown` | `__undefined__` | false | Trạng thái phát sáng của bóng đèn. Bắt buộc chọn, không gán mặc định để tránh thiên kiến |
| `relevance` | N/A | attribute | `__undefined__`, `ego_lane`, `cross_traffic`, `pedestrian`, `uncertain` | `__undefined__` | false | Mức độ điều khiển làn xe chủ. Đây là thuộc tính tối quan trọng tránh critical failure |
| `direction` | N/A | attribute | `__undefined__`, `straight`, `left`, `right`, `general` | `__undefined__` | false | Hướng lưu thông chỉ định của đèn (mũi tên rẽ hay bóng tròn chung) |
| `needs_review` | N/A | attribute | checkbox (`false` / `true`) | `false` | false | Cờ đánh dấu để chuyển cấp thẩm định (escalation) khi annotator không chắc chắn |
| `image_escalate` | tag | tag | N/A | N/A | false | Gán cấp ảnh khi chất lượng ảnh quá kém, chóa lóa toàn bộ không thể quan sát |

## Class hay attribute

- **Tại sao chỉ có 1 class `traffic_light`:**
  Về mặt thị giác và kiến trúc mạng nơ-ron (Object Detection), mọi đầu đèn đều có chung cấu trúc vật lý là một hộp đèn chữ nhật (`housing`). Việc tách riêng thành nhiều class (như `red_light`, `green_light`) sẽ gây bùng nổ số lượng class và phân mảnh dữ liệu huấn luyện. Do đó, gom vào một class duy nhất và quản lý các trạng thái chức năng bằng **Attributes** là chuẩn mực tối ưu cho bài toán xe tự hành.
- **Tại sao đặt default là `__undefined__`:**
  Nếu đặt default là `green` hoặc `ego_lane`, khi người vẽ quên chọn hoặc vội vàng bấm lưu, hệ thống sẽ tự động gán đèn thành màu xanh hoặc đèn điều khiển xe mình mà không có cảnh báo. Giá trị `__undefined__` buộc người vẽ phải tự tay tương tác và chọn đúng giá trị, đồng thời cho phép code kiểm tra lỗi dữ liệu chưa gán trong khâu QA.

## CVAT

- **Phiên bản CVAT** (`make cvat-status`): `v2.74.1` (hoặc `v2.76.0`)
- **Tên task calibration**: `sudo-lite-calib-v1`
- **Guide của task đã dán `02_guideline.md`?**: có (đã sao chép toàn bộ nội dung dán vào ô Description/Guide khi tạo task)
- **Nhóm dùng Track hay Shape, vì sao:** Dùng **Shape** (Rectangle). Mặc dù có dữ liệu chuỗi LISA, việc gán nhãn từng frame bằng Shape mode giúp định vị chính xác vị trí hộp đèn mà không bị trôi dạt (drift) do chuyển động xóc nảy của xe khi tiến vào ngã tư.

## Setup test

Thành viên **Vũ Minh Kiệt** (CVAT & Data Owner) đã tạo task và cùng **Lê Nguyễn Hà My** (QA Owner) mở task thử nghiệm trên máy local để kiểm tra:
- Tạo mới một bounding box chữ nhật trên ảnh `BDD07` ôm khít 2 đầu đèn vỏ vàng New York.
- Chuyển sang tab Objects, kiểm tra đầy đủ 3 dropdown (`state`, `relevance`, `direction`) và 1 checkbox (`needs_review`).
- Kiểm tra tính năng Setup tag `image_escalate` hoạt động tốt.
- Ghi nhận: Thao tác mượt mà, phím tắt `N` lặp lại công cụ vẽ chính xác, nhãn hiển thị màu sắc tương phản rõ ràng (#F9A825).
