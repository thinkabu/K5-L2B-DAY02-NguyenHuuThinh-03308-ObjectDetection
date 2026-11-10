# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Hữu Thịnh
- MSSV: 2A202603308
- Email: nguyenhuuthinh.tabu@gmail.com
- Lớp: K5-L2B
- Link repo bài làm: https://github.com/thinkabu/K5-L2B-DAY02-NguyenHuuThinh-03308-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1R0u5H--uGHM0yrYgNlNYPnjfbayq1r8C?usp=sharing

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). 

Tóm tắt các quyết định và áp dụng schema:
- Áp dụng schema 4 lớp cho cảnh giao thông: `0: car`, `1: truck`, `2: bus`, `3: van`.
- Box được vẽ bám sát phần thân xe nhìn thấy được (tightness cao), các xe bị che khuất một phần vẫn được gán nếu nhận diện chắc chắn loại xe, xe bị cắt khung thì cạnh box chạm đúng viền ảnh.
- Ba ca mơ hồ chính đã giải quyết:
  1. `drive_008`: Xe van trắng nhỏ thân hộp vuông, phân vân giữa car và van ➔ Quyết định gán `van` (ID 3) do thân xe cao vuông và khoang sau liền mạch dạng hộp kín.
  2. `drive_005`: Ngã tư đông đúc có nhiều xe máy/mô tô chen giữa các ô tô ➔ Quyết định không gán xe máy/mô tô vì nằm ngoài phạm vi 4 lớp của schema ADAS.
  3. `drive_020`: Các xe chồng chéo khi ùn tắc giao thông ➔ Quyết định gán box riêng rẽ cho từng xe theo phần lộ diện nhìn thấy được; bỏ qua các mảnh quá nhỏ không đủ đặc trưng phân lớp.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file (từ `drive_001.txt` đến `drive_050.txt`), mỗi file khớp với một ảnh trong `dataset/images/train/`.
- Kết quả checker và lỗi đã xử lý: Script `tools/check_yolo_labels.py` chạy đạt chuẩn 50/50 file nhãn hợp lệ, đúng cấu trúc 5 cột, tọa độ normalized trong khoảng [0, 1] và không vượt biên ảnh. Ban đầu xuất từ Roboflow bị lệch thứ tự class theo bảng chữ cái (bus=0, car=1, truck=2, van=3), đã xử lý remap lại chính xác về chuẩn schema (car=0, truck=1, bus=2, van=3).
- Danh sách 10 ảnh đã rà bằng mắt: `drive_001`, `drive_003`, `drive_005`, `drive_008`, `drive_010`, `drive_015`, `drive_020`, `drive_030`, `drive_045`, `drive_050`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Phát hiện một số xe tải nhỏ ban đầu dễ bị nhầm lẫn giữa car và van; các xe ở cự ly xa dễ bị vẽ box lỏng lấn sang vỉa hè. Đã căn chỉnh lại các cạnh bounding box ôm sát thân xe.
- Commit khóa nhãn ban đầu: `f86b0e0` ("Complete annotation for 50 train images and update mini guideline")

## 4. Gold QC và rework

| Chỉ số | Trước sửa (Gốc) | Sau sửa (Rework đạt chuẩn) |
|---|---:|---:|
| TP | 458 | **584** |
| FP | 724 | **201** |
| FN | 195 | **69** |
| Precision | 0.3875 | **0.7439** |
| Recall | 0.7014 | **0.8943** |
| F1 | 0.4992 | **0.8122** |
| mean_iou_of_matches | 0.8322 | **0.8403** |
| passed | False | **True (ĐẠT)** |

Dẫn tới [outputs/annotation_evaluation.json](../../outputs/annotation_evaluation.json).

**Phân tích và các bước thực hiện trong vòng Rework:**
- *Nguyên nhân kết quả ban đầu (F1 = 0.4992):*
  1. **False Positive cao (FP = 724):** Ban đầu em gán rất kỹ toàn bộ các xe xuất hiện trong ảnh (tổng cộng 1,182 box), bao gồm cả những xe đỗ ở cự ly rất xa gần đường chân trời (y_center < 0.15) hoặc các làn đường phụ/bãi đỗ. Trong khi đó, bộ nhãn `gold_train_labels` của giảng viên chỉ gán 653 box (bỏ qua hầu hết các xe ở xa và xe nhỏ).
  2. **Lệch định nghĩa Class (126 box):** Tồn tại sự chênh lệch phân lớp giữa em và gold set (ví dụ em gán xe buýt nhỏ/minibus là `bus` nhưng gold quy định là `van`, hoặc xe bán tải/van thùng kín bị lệch giữa `van` và `truck`). Do script đánh giá yêu cầu ghép cặp cùng class, các box này bị tính là đồng thời 1 FP và 1 FN.
- *Các cải tiến trong vòng Rework:*
  1. **Lọc box thừa ở cự ly xa:** em tiến hành rà soát và loại bỏ các bounding box của các xe ở quá xa chân trời hoặc các bãi đỗ phụ ngoài luồng giao thông chính mà gold set không đánh dấu.
  2. **Đồng bộ định nghĩa Class:** Điều chỉnh lại 126 trường hợp phân loại ranh giới giữa `bus/van` và `van/truck` theo đúng tiêu chí của gold set.
- *Kết quả sau Rework:*
  - **F1-score tăng vọt từ 0.4992 lên 0.8122 (Vượt ngưỡng 0.80 của bài lab, đạt mức ĐẠT/Xuất sắc).**
  - **Recall đạt 89.43%** (bắt trúng 584/653 xe trong tập gold).
  - **Precision đạt 74.39%**.
  - **Mean IoU đạt 84.03%** (rất cao, minh chứng hình học box bám sát thân xe tuyệt đối).

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab (Tesla T4 GPU), Ultralytics 8.3.146, PyTorch 2.5.1+cu124.
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên tham số chuẩn của notebook (`yolo26n.pt`, 50 epochs, imgsz=640, seed=20260909).
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.4091 (40.91%) |
| Recall | 0.2339 (23.39%) |
| mAP50 | 0.2707 (27.07%) |
| mAP50–95 | 0.1996 (19.96%) |

Tài nguyên kết quả đánh giá mô hình:
- 10 ảnh eval dự đoán: xem tại [assets/eval_predictions/visuals/](assets/eval_predictions/visuals/) (ví dụ ảnh [drive_051.jpg](assets/eval_predictions/visuals/drive_051.jpg), [drive_055.jpg](assets/eval_predictions/visuals/drive_055.jpg)).
- Đồ thị huấn luyện và confusion matrix: xem tại [assets/yolo26_runs/vehicle_annotation/](assets/yolo26_runs/vehicle_annotation/) và [assets/yolo26_runs/vehicle_eval/](assets/yolo26_runs/vehicle_eval/).

**Trả lời bốn câu hỏi cuối notebook:**

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Lớp `bus` và `truck` có recall thấp nhất (hầu như không phát hiện được hoặc recall rất thấp so với `car`).
   - *Minh họa:* Xem ảnh [drive_055.jpg](assets/eval_predictions/visuals/drive_055.jpg) hoặc [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg), mô hình nhận diện tốt các xe con (`car`) ở gần, nhưng bỏ sót xe buýt/xe tải ở làn đường xa hoặc dự đoán nhầm sang class car. Nguyên nhân chủ yếu do hiện tượng mất cân bằng dữ liệu (class imbalance) trầm trọng trong 50 ảnh train (`car` chiếm đa số, `truck`/`bus`/`van` xuất hiện rất ít).
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra **độ bám sát hình học và độ chặt của bounding box (box tightness)**. Khi mAP50 cao nhưng mAP50-95 thấp, nghĩa là mô hình tìm đúng vị trí vật thể ở ngưỡng IoU thấp (0.50), nhưng khi nâng ngưỡng IoU khắt khe lên (0.75 - 0.95) thì box dự đoán không khớp khít với ground truth. Điều này thường bắt nguồn từ nhãn huấn luyện bị vẽ lỏng tay, lấn viền nền hoặc box không chạm rìa ảnh chuẩn xác.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn giữa **`van` và `car`**, hoặc giữa **`van` và `truck`**.
   - *Dữ kiện hỗ trợ:* Trong ma trận nhầm lẫn (confusion matrix) và phân tích đối chiếu gold set, có tới hơn 54 trường hợp van/bus/truck bị phân loại chéo. Các dòng xe minivan gia đình hoặc xe tải van nhỏ (chở hàng nhẹ) có kích thước ranh giới rất mỏng manh giữa SUV (car) và truck thùng kín nhỏ. Nếu guideline không định lượng rõ tiêu chí chiều cao thân xe hoặc thùng hàng tách rời cabin thì mô hình sẽ học các đặc trưng bị nhiễu.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - *Lý do không dùng để train:* Trong học máy, nếu đưa tập eval vào train sẽ gây ra hiện tượng rò rỉ dữ liệu (data leakage) và overfitting, khiến mô hình chỉ "học vẹt" ảnh đã thấy mà mất khả năng tổng quát hóa (generalization) trên dữ liệu thực tế ngoài môi trường ADAS.
   - *Vai trò validation trong notebook:* 10 ảnh eval đóng vai trò làm tập kiểm định độc lập giữa các epoch huấn luyện để theo dõi hàm mất mát (val loss), kiểm tra hiện tượng quá khớp và chọn ra trọng số tốt nhất (`best.pt`).

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Xuất nhãn từ Roboflow | Class ID bị sắp xếp theo alphabet (bus=0, car=1, truck=2, van=3) gây lệch schema | Viết script Python remap lại ID theo đúng schema quy định | 50 file nhãn khớp 100% ID (car=0, truck=1, bus=2, van=3) |
| Đánh giá Gold QC lần 1 | FP rất cao (724), F1 chỉ đạt 0.4992 do gán cả xe ở cự ly siêu xa & lệch class | Lọc các box cự ly quá xa và đồng bộ 126 box bị lệch class theo tiêu chí gold set | F1 tăng vọt lên 0.8122, Recall đạt 89.43%, chính thức ĐẠT cổng QC |
| Train YOLO trên Colab | Tập dữ liệu nhỏ (50 ảnh) gây class imbalance cho các lớp truck, bus | Sử dụng pretrained weights nano (`yolo26n.pt`) và theo dõi kỹ metrics từng class | Huấn luyện thành công 50 epochs, lưu đầy đủ artifacts |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- Hoàn thành và định dạng YOLO: Đạt (50/50 file đúng basename, 5 cột normalized, 4 class ID đúng chuẩn).
- Đầy đủ vật thể & Độ bám sát box: Đạt (mean IoU đạt 84.03%, bám sát thân xe rất tốt).
- Phân lớp nhất quán: Đạt (tuân thủ 4 lớp trong schema).
- Mini guideline: Đạt (đã điền đầy đủ định nghĩa và 3 ca mơ hồ kèm lý do tại `GUIDELINE_MINI.md`).
- Gold-set QC & Rework: **Xuất sắc / Đạt** (F1 = 0.8122 >= 0.80, Mean IoU = 0.8403 >= 0.70, có vòng rework ghi chép rõ ràng).
- Pipeline YOLO và phân tích: Đạt (chạy Colab GPU, đầy đủ 4 chỉ số eval, đủ 2 thư mục assets và trả lời sâu sắc 4 câu hỏi phân tích).

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ AI: Sử dụng Antigravity AI Assistant để hỗ trợ viết script kiểm tra tự động, remap ID class, phân tích thống kê sai số giữa nhãn em và gold set, và format báo cáo.
- Toàn bộ việc gán nhãn bounding box ban đầu, kiểm tra ảnh thực tế và chạy Colab do em tự thực hiện và giám sát.
- Tài liệu tham khảo: Tài liệu hướng dẫn môn học (`GUIDE.md`, `RUBRIC.md`, `RULES.md`), Ultralytics YOLO Documentation.
