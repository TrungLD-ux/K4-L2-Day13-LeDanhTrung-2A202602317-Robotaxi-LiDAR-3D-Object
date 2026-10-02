# Thành viên nhóm Day 13

- Mã nhóm: **K4-DAY13-Nhom-LeDanhTrung**
- Số thành viên: **3**
- Trạng thái thực hiện: **`executed-by-group`**
- Hình thức: Một thành viên trực tiếp vận hành runner; các thành viên còn lại kiểm tra output, phân tích A/B/C, đối chiếu các ca QC và hoàn thiện báo cáo.

| Họ và tên | MSSV | Vai trò lượt A | Vai trò lượt B | Vai trò lượt C |
|---|---|---|---|---|
| Lê Danh Trung | 2A202602317 | Kiểm JSON và cấu hình delta/pillar | Phân tích `summary.csv`, phân bố lớp | Phân tích ảnh Side, tổng hợp báo cáo và hồ sơ GitHub |
| Phạm Văn Thân | 2A202602322 | Trực tiếp vận hành runner, kiểm Docker và `smoke.json` | Kiểm tiến trình inference và output sinh ra | Kiểm đủ file A/B/C và tạo các ca QC |
| Nguyễn An Thái | 2A202602159 | Kiểm provenance và cấu hình đầu vào | Đối chiếu ảnh Side với JSON | Phân tích lỗi batch-z, one-box-z và rà kết luận |

## Phân công và trách nhiệm

### Lê Danh Trung

- Tổng hợp hồ sơ nộp GitHub.
- Đọc `summary.csv`, JSON và ảnh Side của ba lượt.
- Viết phần phân tích A/B, B/C, giới hạn ROI và quyết định QC.

### Phạm Văn Thân

- Trực tiếp vận hành Student runner của nhóm.
- Kiểm Docker Linux/amd64, trạng thái `smoke.json` và tính đầy đủ của output.
- Giữ nguyên output do runner sinh ra, không chỉnh sửa các file bằng tay.

### Nguyễn An Thái

- Kiểm tra provenance, input, checkpoint và cấu hình thí nghiệm.
- Đối chiếu các ca `case-correct`, `case-batch-z`, `case-one-box-z`.
- Rà soát kết luận để tránh coi prediction là ground truth.
