# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- **Họ tên:** Nguyễn Trọng Hùng
- **MSSV:** 2A202603147
- **Email:** ngtronghung004@gmail.com
- **Lớp:** K5-L2
- **Link repo bài làm:** https://github.com/ngtronghung004-droid/K5-L2-Day02-NguyenTrongHung-2A202603147-ObjectDetection
- **Link Colab đã chạy:** [day2_annotation_to_yolo.ipynb trên Colab](https://colab.research.google.com/github/ngtronghung004-droid/K5-L2-Day02-NguyenTrongHung-2A202603147-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb)

## 2. Schema và mini guideline

Chi tiết định nghĩa lớp và quy tắc gán nhãn xem tại [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).

**Tóm tắt cách áp dụng schema và ba ca mơ hồ đã quyết định:**
- **Cách áp dụng schema 4 lớp (`0: car`, `1: truck`, `2: bus`, `3: van`):** Phân loại dựa trên **kiểu dáng cấu trúc vật lý của phương tiện**, không phụ thuộc vào trạng thái đang di chuyển hay đang đỗ. Các xe bán tải (pickup) và SUV/sedan thuộc `car`; xe có cabin tách rời thùng hàng lớn phía sau thuộc `truck`; xe khách/xe buýt thân dài nhiều ô kính thuộc `bus`; xe thân liền khối dạng hộp cỡ trung (minivan/MPV chở hàng hoặc chở người) thuộc `van`.
- **Ba ca mơ hồ tiêu biểu:**
  1. **`drive_008`:** Xe hình hộp lớn ở làn phải (`x ≈ 0.77, y ≈ 0.34`) và xe chở khách thân dài ở giữa đường (`x ≈ 0.46, y ≈ 0.22`). Quyết định gán xe ở làn phải là `van (3)` vì thân liền khối ngắn, và xe ở giữa đường là `bus (2)` vì có dãy cửa sổ dài của xe buýt chở khách.
  2. **`drive_024`:** Hai xe lớn ở làn phải (`x ≈ 0.78, y ≈ 0.49` và `x ≈ 0.83, y ≈ 0.40`) đi cạnh xe hộp ở làn trái (`x ≈ 0.14, y ≈ 0.29`). Quyết định gán hai xe làn phải là `truck (1)` do thấy rõ thùng hàng vuông tách biệt với cabin, còn xe làn trái là `van (3)`.
  3. **`drive_028`:** Có 3 xe con rõ nét (`car`), nhưng ở sát đường chân trời có một cụm nhiều chấm phương tiện rất nhỏ và mờ chồng lấp nhau. Quyết định chỉ gán 3 xe `car` rõ nét và **bỏ qua cụm xe quá nhỏ/mơ hồ ở chân trời** theo đúng quy tắc không tự đoán khi không đủ đặc trưng nhận diện lớp.

## 3. Annotation và self-QC

- **Commit khóa nhãn ban đầu (CP4):** `d853f23`
- **Số file nhãn train:** `50/50` file (`.txt`) trong `dataset/labels/train/`, khớp 1-1 với 50 ảnh `drive_001.jpg` – `drive_050.jpg`.
- **Kết quả checker và lỗi đã xử lý:**
  - Chạy `python tools/check_yolo_labels.py` đạt `50/50` file hợp lệ (`ĐẠT cấu trúc: 50/50 file nhãn hợp lệ`).
  - Xử lý lỗi cấu trúc thư mục bị lồng khi giải nén từ công cụ gán nhãn (`labels/train/` nằm trong thư mục con) bằng cách đưa 50 file `.txt` về trực tiếp `dataset/labels/train/`.
- **Danh sách 12 ảnh đã rà bằng mắt (Self-QC):**
  `drive_001`, `drive_005`, `drive_008`, `drive_012`, `drive_019`, `drive_024`, `drive_026`, `drive_028`, `drive_031`, `drive_038`, `drive_042`, `drive_050`.
- **Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:**
  - Phát hiện thứ tự ID lớp xuất ra từ công cụ ban đầu bị lệch thứ tự ở 3 lớp `1, 2, 3` (`bus`, `van`, `truck` bị xoay vòng so với `data.yaml`) và một số xe ở quá xa phía chân trời bị gán dư. Đã chuẩn hóa lại ánh xạ ID lớp theo đúng `0: car, 1: truck, 2: bus, 3: van` và rà lại độ khít cạnh box.

## 4. Gold QC và rework

Kết quả đối chiếu với `gold_train_labels` (chi tiết lưu tại [annotation_evaluation.json](../../outputs/annotation_evaluation.json)):

| Chỉ số | Trước sửa (`ok1a`) | Sau sửa (Rework) |
|---|---:|---:|
| TP | 354 | **574** |
| FP | 253 | **39** |
| FN | 299 | **79** |
| Precision | 0.5832 | **0.9364** |
| Recall | 0.5421 | **0.8790** |
| F1 | 0.5619 | **0.9068** |
| mean_iou_of_matches | 0.8045 | **0.8153** |
| passed | `false` | **`true`** |

**Nhận xét và lý do Rework:**
- Ở bản trước khi sửa, mặc dù độ bám sát vật thể rất tốt (`mean_iou_of_matches = 0.8045`), chỉ số `F1 = 0.5619` chưa đạt ngưỡng `0.80` do hai nguyên nhân:
  1. **Lệch ánh xạ `class_id` giữa các lớp `truck (1)`, `bus (2)`, `van (3)`:** Nhiều box ở các ảnh `drive_008`, `drive_009`, `drive_024`, `drive_027`, `drive_042` có vị trí khớp sát với Gold (`IoU > 0.80`) nhưng bị lệch mã lớp nên bị tính đồng thời thành 1 FP và 1 FN.
  2. **Thừa box nhỏ ở xa và thiếu một số xe trung cảnh:** Ở các ảnh `drive_001`, `drive_002`, `drive_028`, `drive_031`, `drive_049` có các box gán vào xe quá nhỏ ở xa (`area < 0.008`), đồng thời bản rút gọn `ok1a` bị xóa nhầm một số box xe hợp lệ ở `drive_005`, `drive_019`, `drive_026`, `drive_041`.
- Sau khi chuẩn hóa lại `class_id`, khôi phục các box xe hợp lệ đã vẽ và loại bỏ các box nhiễu quá nhỏ ở chân trời, kết quả đạt **`F1 = 0.9068`** và **`Mean IoU = 0.8153`** (đạt mức **Xuất sắc** theo `RUBRIC.md`).

## 5. Kết quả YOLO

- **Runtime, GPU/CPU, phiên bản Ultralytics và Torch:** Google Colab (T4 GPU), Python 3, PyTorch, Ultralytics YOLO (`yolo26n.pt`).
- **Các tham số thay đổi so với notebook nếu có và lý do:** Giữ nguyên toàn bộ siêu tham số mặc định của notebook (`epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`, `conf=0.25`).
- **Đường dẫn best checkpoint trong phiên chạy:** `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị (từ `model_eval_metrics.json`) |
|---|---:|
| Precision | **0.5267** (`0.5266685401367479`) |
| Recall | **0.6082** (`0.6081803631365598`) |
| mAP50 | **0.5678** (`0.5678433582036945`) |
| mAP50–95 | **0.3549** (`0.35492517781967103`) |

**Dẫn chứng hình ảnh đánh giá và dự đoán trên tập `eval`:**
- Thư mục ảnh dự đoán 10 ảnh `eval`: `assets/eval_predictions/visuals/` (ví dụ: [drive_051.jpg](assets/eval_predictions/visuals/drive_051.jpg), [drive_055.jpg](assets/eval_predictions/visuals/drive_055.jpg), [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg)).
- Thư mục log huấn luyện và biểu đồ đánh giá: [results.csv](assets/yolo26_runs/vehicle_annotation/results.csv), [results.png](assets/yolo26_runs/vehicle_annotation/results.png), [confusion_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png), [confusion_matrix_normalized.png](assets/yolo26_runs/vehicle_eval/confusion_matrix_normalized.png) và [BoxPR_curve.png](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png).
- **Phân tích ảnh cụ thể trên tập `eval`:** Trên các ảnh như [drive_055.jpg](assets/eval_predictions/visuals/drive_055.jpg) và [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg), mô hình đạt `mAP50 = 0.5678` và `mAP50–95 = 0.3549` (checkpoint tốt nhất đạt tại epoch 34), phát hiện tốt và vẽ box sát cho các xe thuộc lớp `car` ở tiền cảnh và trung cảnh với độ tự tin cao; tuy nhiên với các phương tiện thuộc lớp ít mẫu (`truck`, `bus`, `van`) hoặc các xe nhỏ bị che khuất một phần ở phía xa trong [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg), mô hình dễ bỏ sót (FN) hoặc nhầm sang lớp `car`.

**Trả lời bốn câu hỏi cuối notebook:**

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Các lớp thiểu số (`van`, `bus`, `truck`) có Recall thấp hơn hẳn so với lớp `car` (thường `van` hoặc `bus` có Recall thấp nhất trên tập `eval`). Nguyên nhân chính là **mất cân bằng lớp (class imbalance)** nghiêm trọng trong tập 50 ảnh train (lớp `car` chiếm đa số tuyệt đối với hơn 450 box, trong khi `bus`, `truck`, `van` chỉ có vài chục mẫu mỗi lớp). Ví dụ minh họa trên tập eval như ảnh `drive_056.jpg` hoặc `drive_058.jpg`, các xe `car` được bắt gần như đầy đủ nhưng xe lớn/van ở xa bị bỏ sót hoặc nhận nhầm thành `car`.
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra **độ khít/bám sát (tightness) của các cạnh bounding box** trong dữ liệu gán nhãn và dự đoán. `mAP50` chỉ yêu cầu `IoU >= 0.50` là đã tính đúng (box hơi lỏng hoặc lệch nhẹ vẫn đạt), trong khi `mAP50–95` lấy trung bình ở các ngưỡng khắt khe tới `IoU = 0.75, 0.85, 0.95`. Nếu `mAP50` cao mà `mAP50–95` thấp, chứng tỏ mô hình định vị đúng vùng có xe nhưng biên box còn lỏng (lấy dư nền/bóng đổ) hoặc cắt thiếu phần thân xe.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn giữa **`van` ↔ `bus`** (xe minibus/van thân hộp), **`van` ↔ `car`** (SUV/MPV 7 chỗ cỡ lớn với minivan) và **`truck` ↔ `van`** (xe tải thùng kín nhỏ với xe van chở hàng). Dữ kiện hỗ trợ: ngay trong bước Gold QC (Mục 4) và ma trận nhầm lẫn (`confusion_matrix.png`), các ca lệch lớp tập trung chủ yếu ở nhóm phương tiện thân hộp nhìn từ góc camera CCTV trên cao chiếu xuống (`drive_008`, `drive_024`, `drive_042`), nơi ranh giới thị giác giữa minivan (`van`), minibus (`bus`) và xe tải thùng nhỏ (`truck`) rất gần nhau nếu không quy định rõ tiêu chí phân biệt trong guideline.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không được đưa 10 ảnh `eval` vào tập `train` để tránh **rò rỉ dữ liệu (data leakage)**: nếu mô hình đã học trực tiếp trên các ảnh này thì điểm mAP đo lại trên chính chúng sẽ cao giả tạo (overfitting) và không phản ánh khả năng tổng quát hóa trên cảnh mới.
   - Trong notebook này, 10 ảnh `eval` đóng vai trò là **tập validation (`val`)** được đánh giá sau mỗi epoch để theo dõi quá trình hội tụ và **lựa chọn checkpoint tốt nhất (`best.pt`)**, sau đó dùng chính `best.pt` để xuất dự đoán trực quan. Vì đã tham gia vào việc chọn `best.pt`, tập `eval` này là tập validation chứ không phải một tập test độc lập hoàn toàn.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CP3 — Kiểm tra cấu trúc `check_yolo_labels.py` | Báo thiếu 50 file nhãn dù đã giải nén vào `dataset/labels/train/`. | Kiểm tra cây thư mục, phát hiện file nén tạo thêm thư mục con `ok1a/labels/train/`. Đưa 50 file `.txt` ra trực tiếp `dataset/labels/train/`. | Đạt `50/50` file hợp lệ. |
| CP5 — Chấm Gold QC lần đầu | `Mean IoU` đạt cao (`0.8045`) nhưng `F1` chỉ đạt `0.5619 < 0.80`. | Phân tích từng cặp box khớp vị trí (`IoU >= 0.50`), phát hiện thứ tự `class_id` của `truck/bus/van` bị xoay vòng (`1->2, 2->3, 3->1`) và dư các box nhỏ ở chân trời. Tiến hành rework chuẩn hóa ID lớp, khôi phục box bị xóa nhầm và lọc box nhiễu. | Đạt `F1 = 0.9068`, `Mean IoU = 0.8153` (`passed = true`). |

## 7. Tự đánh giá theo rubric

- **Hoàn thành và định dạng YOLO (10/10):** Đủ 50 file `.txt` đúng basename, 5 cột chuẩn hóa, vượt qua `check_yolo_labels.py`.
- **Đầy đủ vật thể & Độ bám sát box (40/40):** `Recall = 0.8790`, `Precision = 0.9364`, `Mean IoU = 0.8153 >= 0.80` (mức Xuất sắc).
- **Phân lớp nhất quán & Mini guideline (25/25):** Đã hoàn thiện [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md) với định nghĩa 4 lớp, quy tắc vẽ box và 4 ca mơ hồ có lý do chi tiết.
- **Gold-set annotation QC (15/15):** Có đầy đủ [annotation_evaluation.json](../../outputs/annotation_evaluation.json), đạt `F1 = 0.9068 >= 0.80` và phân tích rõ số liệu trước/sau rework.
- **Pipeline YOLO và phân tích (10/10):** Giữ đúng split 50 train / 10 eval, trả lời đầy đủ 4 câu hỏi phân tích.

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ sử dụng:** Công cụ gán nhãn xuất định dạng YOLO, Python scripts (`tools/check_yolo_labels.py`, `tools/evaluate_annotations.py`), Google Colab (Ultralytics YOLO), và trợ lý AI hỗ trợ kiểm tra lỗi cấu trúc thư mục, phân tích nguyên nhân lệch `class_id` trong bước Gold QC và rà soát trình bày báo cáo.
- **Phần đã tự kiểm tra:** Đã tự đối chiếu lại các file nhãn trong `dataset/labels/train/`, kết quả `outputs/annotation_evaluation.json` và kiểm tra trực quan các ảnh dự đoán.
- **Tài liệu tham khảo:** Tài liệu hướng dẫn trong repo bài lab (`README.md`, `GUIDE.md`, `RULES.md`, `RUBRIC.md`, `SUBMISSION.md`).
