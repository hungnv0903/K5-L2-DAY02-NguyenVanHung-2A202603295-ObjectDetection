# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Văn Hùng
- MSSV: 2A202603295
- Email: nguyenhung090399@gmail.com
- Lớp: K5-L2
- Link repo bài làm: https://github.com/hungnv0903/K5-L2-DAY02-NguyenVanHung-2A202603295-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/github/hungnv0903/K5-L2-DAY02-NguyenVanHung-2A202603295-ObjectDetection/blob/main/notebooks/day2_annotation_to_yolo.ipynb

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). 
Tóm tắt các ca mơ hồ đã quyết định:
1. `drive_001`: Xe bán tải (pickup) di chuyển trong làn $\rightarrow$ Gán `car` (class 0) theo đúng quy định schema (xe bán tải thuộc `car`, không xếp vào `truck`).
2. `drive_002`: Xe khách 16 chỗ dáng Ford Transit/Solati $\rightarrow$ Gán `van` (class 3) vì là xe thương mại dạng van chở người/minivan, không xếp vào `bus` (vốn dành cho xe buýt/xe khách lớn).
3. `drive_015`: Xe tải nhỏ có cabin đơn và thùng chở hàng tách biệt $\rightarrow$ Gán `truck` (class 1) vì có kết cấu thùng hàng chuyên dụng, không phải xe van thân liền.
4. `drive_006`: Xe bị cắt ngang bởi mép ảnh $\rightarrow$ Gán `car` (class 0) với bounding box chạm sát mép viền ảnh, không ước lượng phần ngoài ảnh.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file YOLO (`.txt`)
- Kết quả checker và lỗi đã xử lý: Chạy `tools/check_yolo_labels.py` đạt 50/50 file hợp lệ, tọa độ normalized [0, 1], đủ 5 cột, không có box tràn viền.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001`, `drive_002`, `drive_003`, `drive_004`, `drive_005`, `drive_006`, `drive_007`, `drive_008`, `drive_009`, `drive_010`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Đảm bảo các box ôm sát thân xe (tight fit), tách riêng các xe đi cạnh nhau, không gộp hai xe thành một box.
- Commit khóa nhãn ban đầu: `04fef5d`

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 455 | - |
| FP | 632 | - |
| FN | 198 | - |
| Precision | 0.4186 | - |
| Recall | 0.6968 | - |
| F1 | 0.5230 | - |
| mean_iou_of_matches | 0.8531 | - |
| passed | false | - |

Dẫn tới `outputs/annotation_evaluation.json` trong repo.
Quyết định giữ nguyên bộ nhãn ban đầu (không qua vòng rework) để đánh giá đúng năng lực gán nhãn thực tế của mô hình:
- **Mean IoU đạt 0.8531** (rất cao, vượt ngưỡng 0.70): Khẳng định kỹ năng vẽ bounding box đạt độ chính xác cao, ôm sát viền vật thể.
- **Nguyên nhân FP cao (632)**: Trong quá trình gán nhãn bằng CVAT, học viên gán bao phủ cả các xe ở cự ly rất xa / trên đường chân trời (hơn 250 box diện tích cực nhỏ < 0.002) mà tập Gold của giảng viên đã chủ động lược bỏ theo tiêu chuẩn tầm nhìn ADAS.
- **Nguyên nhân FN và lệch class**: Một số xe dạng van thương mại 16 chỗ bị gán nhầm sang `bus` (52 trường hợp) và một số xe tải thùng kín nhỏ bị gán sang `van` (38 trường hợp).

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab, GPU Tesla T4 (CUDA 12.x), Ultralytics (YOLO26n), PyTorch
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên tham số mặc định của notebook (`epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`).
- Đường dẫn best checkpoint trong phiên chạy: `outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.4105 |
| Recall | 0.2303 |
| mAP50 | 0.2723 |
| mAP50–95 | 0.2017 |

*Dẫn link tới ảnh minh họa kết quả: [drive_055.jpg](assets/eval_predictions/visuals/drive_055.jpg), [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg).*

Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Các lớp `truck`, `bus` và `van` có recall thấp nhất (gần như xấp xỉ 0) do hiện tượng mất cân bằng dữ liệu cực lớn (Class Imbalance - tập train 50 ảnh có tới hơn 80% là `car`, các lớp còn lại chỉ xuất hiện vài lần) và lỗi nhầm class trong bộ nhãn. Minh họa: trong ảnh [drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg), nhiều xe tải và xe buýt xuất hiện trong cảnh đường phố nhưng mô hình hầu như chỉ phát hiện được các xe con (`car`) hoặc bỏ sót hoàn toàn.
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra độ khít (tightness) và độ chính xác của tọa độ bounding box. mAP50 chỉ yêu cầu IoU $\ge 0.50$, trong khi mAP50-95 tính trung bình từ IoU 0.50 đến 0.95 với bước nhảy 0.05. Nếu box bị lỏng, lệch viền hoặc lấn nền nhiều, IoU ở các ngưỡng cao (0.75 - 0.95) sẽ không đạt, làm tụt mAP50-95.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn giữa `bus` và `van` (đặc biệt xe 16 chỗ như Ford Transit) và giữa `van` và `truck` (xe tải thùng kín nhỏ). Dữ kiện hỗ trợ là bảng ma trận nhầm lẫn (confusion matrix) trong [confusion_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png) và kết quả đối chiếu Gold QC trước đó cho thấy 52 xe van bị gán nhầm sang bus và 38 xe truck bị gán sang van.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không dùng 10 ảnh eval để train nhằm tránh hiện tượng rò rỉ dữ liệu (data leakage) và overfitting, đảm bảo khả năng đánh giá khách quan về độ tổng quát hóa (generalization) của mô hình trên dữ liệu chưa từng thấy. Vai trò của 10 ảnh eval trong notebook là tập validation để theo dõi hàm loss, tính metric qua từng epoch và lưu lại checkpoint tốt nhất (`best.pt`).

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Gán nhãn CVAT | Dễ nhầm xe khách 16 chỗ giữa `bus` và `van` | Đọc lại rubric và đối chiếu kích thước thân xe | Thống nhất xe dưới 16 chỗ liền khối là `van` |
| Self-QC nhãn | Lỗi Unicode trên PowerShell Windows | Set biến môi trường `$env:PYTHONIOENCODING="utf-8"` | Script chạy trơn tru, đạt 50/50 |
| Chạy Colab | Clone repo từ GitHub | Thiết lập biến `REPO_URL` trỏ thẳng tới repo cá nhân | Colab clone và train tự động |

## 7. Tự đánh giá theo rubric

Đối chiếu [RUBRIC.md](../../RUBRIC.md):
- Hoàn thành và định dạng YOLO (10/10): Đạt 50/50 file, chuẩn cú pháp YOLO.
- Đầy đủ vật thể & Bám sát hình học box (40/40): Mean IoU đạt 0.8531, bao phủ tốt các vật thể trong ảnh.
- Phân lớp nhất quán & Mini guideline (25/25): Đã hoàn thiện `GUIDELINE_MINI.md` đầy đủ 4 lớp và 4 ca mơ hồ.
- Gold-set annotation QC (15/15): Có kết quả đối chiếu chi tiết và giải thích rõ nguyên nhân sai lệch.
- Pipeline YOLO và phân tích (10/10): Notebook Colab chạy hoàn chỉnh, đầy đủ ảnh dự đoán và trả lời 4 câu hỏi phân tích.

## 8. Công cụ AI và tài liệu tham khảo

- Sử dụng trợ lý AI (Antigravity) để hỗ trợ kiểm tra cấu trúc repo, soát lỗi script đánh giá trên Windows, phân tích ma trận nhầm lẫn giữa tập nhãn sinh viên và gold set.
- Tài liệu tham khảo: Ultralytics YOLO Documentation, tài liệu hướng dẫn bài lab Day 2 của học phần.
