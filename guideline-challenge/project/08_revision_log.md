# Revision log

Guideline v1 = bản nháp đầu; v2 = sau calibration nội bộ; v3 = sau blind handoff. Mỗi lần tăng `Version` trong
`02_guideline.md`, thêm một hoặc nhiều dòng vào bảng: đổi gì và vì sao, kèm bằng chứng (sample_id, dòng
calibration report, câu hỏi trong clarification log, feedback của peer).

Cột Version ghi dạng `v1`, `v2`, `v3` — `make status` tìm dòng bảng có `v2` và dòng có `v3`.

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| `v2` | Làm rõ quy tắc phân làn cho đường một chiều và giàn đèn có biển rẽ phụ; bổ sung checklist kiểm tra thuộc tính __undefined__ | Giải quyết 3 bất đồng lớn phát hiện trong đợt calibration nội bộ giữa 3 annotator (thien, kiet, minh) | `BDD02`, `LISA04`, `BDD07` (file `06_calibration_report.csv`) |
| `v3` | Bổ sung quy tắc khi xe đứng sát vạch dừng không thấy vạch phân làn; quy tắc fallback vẽ tim bóng khi vỏ chìm vào nền tối ban đêm; quy chuẩn đèn người đi bộ đếm ngược | Khắc phục các lỗi gán nhãn và mơ hồ do nhóm peer annotator phản hồi sau đợt blind handoff | `LISA20`, `BDD26`, `BDD18` (file `peer_feedback.md`, `transfer_score.csv`) |
