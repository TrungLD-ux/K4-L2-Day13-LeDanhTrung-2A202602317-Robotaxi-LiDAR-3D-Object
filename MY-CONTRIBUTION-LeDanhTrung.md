# Đóng góp cá nhân — Lê Danh Trung

- Họ tên: **Lê Danh Trung**
- MSSV: **2A202602317**
- Nhóm: **K4-DAY13-Nhom-LeDanhTrung**

## Công việc thực hiện

- Kiểm tra output trực tiếp do thành viên nhóm chạy.
- Đọc `summary.csv`, prediction JSON và ảnh Side của A/B/C.
- Phân tích ảnh hưởng của delta và pillar.
- Đối chiếu lỗi toàn batch và lỗi cục bộ một cuboid.
- Tổng hợp `TEAMMATES.md`, `PRE-LABEL-REPORT.md` và hồ sơ nộp GitHub.

## Điều rút ra

Delta được áp dụng trước inference nên có thể thay đổi toàn bộ tập prediction, không chỉ dịch z của các hộp có sẵn. Pillar size thay đổi cách point cloud được gom thành biểu diễn đầu vào. Lỗi z đồng loạt cần kiểm pipeline; lỗi riêng một hộp cần kiểm theo đối tượng.

## Vai trò trong lượt chạy nhóm

Phạm Văn Thân trực tiếp vận hành runner. Lê Danh Trung kiểm tra output và tổng hợp báo cáo. Nguyễn An Thái kiểm provenance và các ca QC. Toàn nhóm sử dụng trạng thái `executed-by-group`.
