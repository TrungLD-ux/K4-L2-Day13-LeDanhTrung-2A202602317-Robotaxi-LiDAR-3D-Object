# Báo cáo thực hành PointPillars — Day 13

> **Tuyên bố thực hiện:** Nhóm gồm ba thành viên đã trực tiếp hoàn thành quy trình Student runner. Một thành viên vận hành runner; hai thành viên còn lại kiểm tra output, phân tích A/B/C và các ca QC. `smoke.json` ghi nhận trạng thái `passed`.

## 1. Nhóm và provenance

- Mã nhóm: **K4-DAY13-Nhom-LeDanhTrung**
- Thành viên: xem `TEAMMATES.md`.
- Trạng thái: **`executed-by-group`**.
- Người trực tiếp vận hành runner: **Phạm Văn Thân, MSSV 2A202602322**.
- Người tổng hợp báo cáo: **Lê Danh Trung, MSSV 2A202602317**.
- Người kiểm provenance và QC: **Nguyễn An Thái, MSSV 2A202602159**.
- Nguồn output: output trực tiếp của nhóm.
- Thời gian thực thi ghi trong `smoke.json`: **2026-10-01 07:57:41–07:58:31 UTC**, tương ứng **14:57:41–14:58:31 giờ Việt Nam (UTC+7)**.
- Môi trường thực thi được ghi trong output: Windows 11; Docker runtime `linux/amd64`.
- Giới hạn container: 4 CPU, 4 GB RAM.
- Image tag: `day13-pointpillars:lc-20261001-amd64`.
- Image ID: `sha256:e03983bd922ec29890bf547db8de408402efd82583680b62e671c20da2fd2c82`.
- Repo revision: `0831856d921609312d42c7582c366e5a311bb7b1`.
- Working tree dirty: `true` theo `smoke.json`.
- Frame: `demo.pcd`, `frame_id = demo`.
- Dataset/model dataset: KITTI.
- Input SHA256: `3b5ea3da13e2b19149cab6a8d521c2ca55f2df93f026b5a3f8c273ce70645d60`.
- Checkpoint: `/opt/PointPillars/pretrained/epoch_160.pth`.
- Checkpoint SHA256: `482dfcf63b932cc5ccf012b4bbdad52aa51aa33becf87d0a39d61c39b377b5b1`.
- Phạm vi: front-window, không dùng `--full-scene`.
- Score threshold: `0.3`.
- `z_ground = 0.075 m`, do pipeline ước lượng từ PCD.

### Ghi chú thiết bị

Laptop Acer Aspire 7 đời 2022 của Lê Danh Trung được sử dụng để tổng hợp báo cáo và nộp GitHub. Môi trường runner được báo cáo theo metadata của output là Windows 11 với Docker `linux/amd64`. Báo cáo không suy đoán CPU, GPU hoặc RAM khi chưa có mã cấu hình máy chính xác.

### Giả định kênh thứ tư

PCD Student đã bỏ reflectance thật; trường RGB bằng 0 chỉ là placeholder. Pipeline sử dụng kênh hằng của adapter, vì vậy đây không phải intensity LiDAR được phục hồi và không phải benchmark KITTI tiêu chuẩn.

## 2. Mục tiêu và thiết kế thí nghiệm

Ba lượt dùng cùng PCD, checkpoint, score threshold và ROI:

- **A:** `delta = 0 m`, pillar XY `0.16 m`.
- **B:** `delta = 1.73 m`, pillar XY `0.16 m`.
- **C:** `delta = 1.73 m`, pillar XY `0.32 m`.

A/B chỉ thay đổi delta. B/C chỉ thay đổi kích thước pillar. Không dùng A/C để quy kết một nguyên nhân riêng vì hai biến thay đổi đồng thời.

## 3. Kết quả A/B/C

| Lượt | Delta | Pillar XY | Số hộp | mean_z | Phân bố lớp |
|---|---:|---:|---:|---:|---|
| A | 0 m | 0.16 m | 1 | 0.330 m | 1 `vehicles` |
| B | 1.73 m | 0.16 m | 13 | 1.034 m | 10 `vehicles`, 1 `two-wheels`, 2 `pedestrian` |
| C | 1.73 m | 0.32 m | 6 | 1.091 m | 6 `pedestrian` |

`mean_z` là trung bình tọa độ z tâm hộp, không phải thước đo chất lượng.

### Lượt A

- Prediction SHA256: `02b1ba0ab8c2054852b74221a0ed31053d7447cded56f1e0fe1f784eb0d44880`.
- Một hộp `vehicles`, score `0.322`, tâm xấp xỉ `(13.154, -0.451, 0.330)` m.
- Kích thước xấp xỉ `3.620 × 1.523 × 1.459` m.
- Đáy hộp xấp xỉ `0.330 - 1.459/2 = -0.399 m`, thấp hơn đường tham chiếu z = 0.

### Lượt B

- Prediction SHA256: `16f30b0833ec5978320d71a102bf54df825d76c8d8264e563a7b7b40bee4cf61`.
- 13 hộp, score từ `0.318` đến `0.933`.
- Hai xe score cao nhất ở gần `(8.094, 1.208, 0.921)` và `(14.766, -1.081, 0.900)` m.
- Vùng x khoảng 8–11 m có nhiều hộp chồng nhau trên ảnh Side, có thể do các đối tượng khác y bị chiếu lên cùng mặt phẳng x-z.

### Lượt C

- Prediction SHA256: `326bbb947b7571f98e391fe63886c14ee62d74d2d2a283eb6105da0f75f791ad`.
- Sáu prediction đều là `pedestrian`; không còn `vehicles` hoặc `two-wheels`.
- Một số hộp C nằm gần vùng có xe/xe hai bánh trong B, ví dụ hộp C gần `(9.109, 0.404)` và hộp B gần `(8.094, 1.208)`.

## 4. Phân tích A/B

Phép chuyển đổi được mô tả bởi:

`z_model = z_source - z_ground - delta`

`z_source = z_model + z_ground + delta`

Thay delta trước inference khác với dịch output sau inference. Delta làm thay đổi tọa độ z của toàn bộ điểm trước khi gom pillar và trước khi model dự đoán. Vì vậy, model nhận input khác và có thể thay đổi số hộp, lớp, vị trí, yaw và score. Kết quả A/B minh họa điều đó: số hộp tăng từ 1 lên 13 và phân bố lớp thay đổi, chứ không chỉ có z của một tập hộp cố định dịch đều 1.73 m.

Chưa thể nói B “đúng hơn” chỉ vì có nhiều hộp hơn. Không có ground truth được duyệt để xác định precision, recall hoặc chất lượng từng cuboid.

## 5. Phân tích B/C

B/C giữ nguyên delta 1.73 m và chỉ đổi pillar từ 0.16 m lên 0.32 m. Số hộp giảm từ 13 xuống 6; phân bố lớp đổi từ 10 xe, 1 xe hai bánh, 2 người thành 6 người.

Pillar 0.32 m gom điểm trong ô x-y lớn hơn, làm thay đổi độ phân giải không gian và biểu diễn đầu vào. Checkpoint được huấn luyện với cấu hình gốc 0.16 m, nên C chỉ là thí nghiệm kiểm phản ứng của pipeline khi thay biểu diễn, không chứng minh 0.32 m tốt hoặc xấu trong mọi trường hợp.

Không đủ bằng chứng để kết luận B hay C tốt hơn chỉ dựa trên số hộp, score hoặc mean_z. Cần Top/Front/camera và reference đã duyệt.

## 6. Giới hạn ROI và ảnh Side

Runner chỉ xử lý front-window của checkpoint. Vật nằm ngoài ROI không xuất hiện trong output là giới hạn phạm vi, không tự động là false negative.

Ảnh Side chiếu toàn bộ điểm lên mặt x-z, nên các vật khác y có thể chồng nhau. Side hữu ích cho kiểm z, chiều cao và đáy hộp, nhưng không đủ để kết luận tâm y, chiều rộng, yaw hoặc đầu/đuôi xe. Đường z = 0 trên plot chỉ là tham chiếu, không phải mặt đường cục bộ tại mọi vị trí.

## 7. Khả năng import JSON

Không JSON nào trong A/B/C đủ cơ sở để import vào Robotaxi. Chúng là prediction thô trên frame KITTI demo, khác frame và dataset. Các file QC cũng có `training_only: true`, không phải ground truth và không được import vào CVAT.

Trước khi dùng prediction làm pre-label thực tế cần xác nhận đúng frame, schema, hệ tọa độ, quy ước center-z/yaw, ROI, class và đối chiếu bằng point cloud cùng camera.

## 8. Ca QC có kiểm soát

| Ca | Hộp lệch z | Lượng lệch | Trường khác | Quyết định |
|---|---:|---:|---|---|
| `case-correct` | 0/13 | 0 m | Giữ nguyên prediction B | Dùng đối chiếu transform, không coi là ground truth |
| `case-batch-z` | 13/13 | -1.805 m | Class, x, y, kích thước, yaw và score không đổi | Dừng batch, kiểm pipeline |
| `case-one-box-z` | 1/13 | -1.805 m ở hộp đầu tiên | 12 hộp còn lại giữ nguyên | Kiểm riêng cuboid |

`height_offset = delta + z_ground = 1.73 + 0.075 = 1.805 m`.

### Batch-z

Tất cả 13 hộp bị trừ cùng 1.805 m. Đây là dấu hiệu lỗi hệ thống phù hợp với việc quên chuyển ngược từ hệ model về hệ nguồn. Hành động đúng là dừng sửa thủ công, kiểm transform và tạo lại prediction.

### One-box-z

Chỉ hộp đầu tiên bị lệch, còn 12 hộp giữ nguyên. Chưa đủ cơ sở kết luận pipeline hỏng. Cần kiểm riêng hộp bằng Top, Side, Front và camera.

## 9. Trạng thái runner của nhóm

`smoke.json` có `status: passed`. Năm bước đều thành công:

| Bước | Thời gian |
|---|---:|
| docker-load | 34.165 s |
| run-A | 5.310 s |
| run-B | 5.130 s |
| run-C | 3.540 s |
| qc-cases | 2.002 s |

Tổng thời gian từ đầu docker-load đến cuối QC xấp xỉ 50.27 giây.

## 10. Nhận xét cá nhân

### 10.1. Lê Danh Trung, MSSV 2A202602317

- **Vai trò:** Tổng hợp báo cáo, kiểm `summary.csv`, đối chiếu JSON và phân tích các ca QC từ bộ kết quả được cung cấp.
- **Quan sát:** A có 1 hộp score 0.322; B có 13 hộp với ba class; C có 6 hộp và tất cả là `pedestrian`. Tôi không dùng số hộp để kết luận cấu hình tốt hơn vì chưa có ground truth được duyệt.
- **Diễn giải z:** Delta được áp dụng lên point cloud trước inference, nên thay delta có thể làm thay đổi toàn bộ tập prediction. Sau inference, pipeline cộng lại `z_ground + delta` để trả hộp về hệ nguồn.
- **Quyết định QC:** Với `case-batch-z`, 13/13 hộp cùng lệch -1.805 m nên phải dừng batch và kiểm pipeline. Với `case-one-box-z`, chỉ 1/13 hộp lệch nên kiểm riêng hộp đó.
- **Điều chưa chắc:** Side không thể hiện y và yaw đầy đủ; chưa có camera hoặc reference để kết luận từng cuboid đúng hay sai.
- **Tuyên bố thực hành:** Tôi trực tiếp kiểm tra output nhóm, phân tích số liệu A/B/C và tổng hợp báo cáo; thành viên Phạm Văn Thân trực tiếp vận hành runner.

### 10.2. Phạm Văn Thân, MSSV 2A202602322

- **Vai trò:** Trực tiếp vận hành Student runner, kiểm Docker, `smoke.json` và tính đầy đủ của output A/B/C.
- **Quan sát:** Runner hoàn thành đủ `docker-load`, `run-A`, `run-B`, `run-C` và `qc-cases`; trạng thái cuối là `passed`. B có 13 hộp, nhiều hơn A và C, nhưng số lượng prediction không phải thước đo accuracy.
- **Diễn giải z:** Delta được trừ khỏi point cloud trước inference và được cộng lại cùng `z_ground` khi đưa hộp về hệ nguồn. Vì vậy, thay delta có thể làm thay đổi tập prediction.
- **Quyết định QC:** Với `case-batch-z`, 13/13 hộp cùng lệch -1.805 m nên dừng batch và kiểm phép chuyển hệ, không sửa thủ công từng hộp.
- **Điều chưa chắc:** Không có ground truth hay camera trong gói Student nên chưa thể đánh giá precision/recall.
- **Tuyên bố thực hành:** Tôi là thành viên trực tiếp vận hành runner và giữ nguyên output do chương trình sinh ra.

### 10.3. Nguyễn An Thái, MSSV 2A202602159

- **Vai trò:** Kiểm provenance, đối chiếu ba ca QC và rà soát kết luận của nhóm.
- **Quan sát:** `case-batch-z` làm 13/13 hộp giảm đúng 1.805 m; `case-one-box-z` chỉ làm hộp đầu tiên giảm 1.805 m, trong khi 12 hộp còn lại giữ nguyên.
- **Diễn giải z:** Nếu thiếu phép cộng ngược `z_ground + delta`, lỗi sẽ xuất hiện đồng loạt. Một hộp sai riêng không đủ để kết luận transform của toàn pipeline sai.
- **Quyết định QC:** Dừng và kiểm pipeline đối với lỗi đồng loạt; kiểm riêng cuboid đối với lỗi cục bộ.
- **Điều chưa chắc:** Mặt đường cục bộ có thể khác `z_ground` ước lượng từ toàn scan, do đó cần nhiều góc nhìn khi đánh giá một hộp riêng.
- **Tuyên bố thực hành:** Tôi kiểm tra output, provenance và logic QC của lượt chạy nhóm.

## 11. Hạn chế

- Chỉ có một frame KITTI demo.
- Không có ground truth/camera trong bundle để chấm accuracy.
- Không giữ reflectance LiDAR thật.
- `z_ground` là ước lượng toàn scan.
- C dùng checkpoint không được train riêng cho pillar 0.32 m.
- CPU NMS là phương án thay thế kernel CUDA.
- Có thể khác prediction hash giữa kiến trúc; không yêu cầu khớp bitwise.

## 12. Kết luận

A/B cho thấy thay delta trước inference có thể thay đổi toàn bộ tập prediction. B/C cho thấy thay kích thước pillar có thể làm thay đổi mạnh số lượng và phân bố class. Các kết quả này không đủ để tuyên bố cấu hình nào chính xác hơn nếu thiếu ground truth và kiểm tra đa góc nhìn.

Ba ca QC chứng minh nguyên tắc xử lý quan trọng: lỗi đồng loạt trên toàn batch cần dừng và kiểm pipeline; lỗi riêng một cuboid cần được kiểm theo đối tượng. Không dùng prediction KITTI hoặc các ca `training_only` để import vào CVAT Robotaxi.

## 13. LC ghi nhận riêng

- Quyền dùng PCD/image và đúng ca:
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung:
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT:
- Nhận xét từng thành viên và quyết định dừng pipeline:
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do:
