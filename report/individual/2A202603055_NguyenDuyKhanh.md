# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Duy Khánh
- MSSV: 2A202603055
- Email: duykhanh91.hr@gmail.com/duykhanh19.hr@gmail.com
- Lớp: K5 – L2A (2A-LAB-E402)
- Link repo bài làm: https://github.com/henrykhanh-ai/K5-L2A-DAY02-NGUYENDUYKHANH-2A202603055-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1rAuJnnCZxJn_uxMuAnLlgMKWZIv2nONP

## 2. Schema và mini guideline

Chi tiết ở [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Schema áp dụng đúng README: `car=0, truck=1, bus=2, van=3` (trong CVAT tương ứng nhãn thứ 1–4).

Các ca mơ hồ đã quyết định:

1. **Xe cứu hộ công an có cẩu** (drive_014, 019, 034, 038, 046): lần gán đầu không nhất quán (lúc `van`, lúc `car`), sau self-QC thống nhất là `truck` vì đây là xe chuyên dụng trên khung xe tải; `car` loại trừ xe tải.
2. **Minivan thân hình hộp** (drive_021, 024, 027, 031, 043): ban đầu bị gán `car`, sửa thành `van` theo định nghĩa "minivan chở hàng hoặc chở người". Xe mui thấp có ca-pô và đuôi rõ vẫn là `car`.
3. **Xe ba bánh chở hàng** (drive_024): không gán vì nằm ngoài bốn lớp của schema.
4. **Xe ben, xe tải thùng, xe bồn** (drive_008, 009, 033, 037, 047): sửa từ `van`/`car` sang `truck` ("xe tải chở hàng"), nhất quán với xe ben ở drive_040.

Cần giảng viên làm rõ:

- Nhãn gold (cả `dataset/labels/eval/` và gold train) đang dùng mapping **1 = van, 2 = truck, 3 = bus**, lệch với `data.yaml`/README (1 = truck, 2 = bus, 3 = van). Dữ kiện ở mục 4 và 5.
- Gold không gán xe ở xa cuối đường và xe đỗ ven đường; README không nêu quy tắc này.

## 3. Annotation và self-QC

- Số file nhãn train: 50/50, công cụ CVAT, export định dạng Ultralytics YOLO Detection 1.0. Tổng 1204 box (car 1065, bus 91, van 34, truck 14).
- Kết quả checker và lỗi đã xử lý: `check_yolo_labels.py` đạt 50/50 ở mọi lần export. Đã kiểm tra `data.yaml` của bản export đúng thứ tự `car, truck, bus, van`.
- Danh sách ảnh đã rà bằng mắt (vẽ box lên ảnh và phóng to từng xe nghi ngờ): drive_008, drive_009, drive_014, drive_019, drive_021, drive_024, drive_027, drive_031, drive_033, drive_034, drive_037, drive_038, drive_043, drive_046, drive_047, drive_048.
- Lỗi phát hiện khi rà và cách sửa (vòng self-QC, trước khi nhận gold):
  - Bản 1 chỉ có 3 `truck`: nhiều xe tải bị gán `van` hoặc `car`. Đã đổi xe ben, xe thùng, xe bồn, xe cứu hộ sang `truck` (3 → 14 box).
  - Nhiều minivan bị gán `car`: đổi sang `van` ở drive_021, 027, 031, 043 và van đỏ ở drive_038 (nhất quán với drive_014).
  - Xóa box xe ba bánh ở drive_024; ở drive_024 đổi lại 2 box xe con bị gán nhầm `van` về `car`.
  - Box xe nhỏ được chỉnh sát lại (một số box ban đầu dư khoảng trống trên nóc và đầu xe).
- Commit khóa nhãn: `f04246b`. Bộ nhãn được chốt từ bản export CVAT lúc 10:14 ngày 10/11, **trước khi nhận gold**; do sự cố Git trên máy (mục 6), commit lên GitHub lúc 10:34. Nhãn không chỉnh sửa sau khi nhận gold.

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 483 | Không rework |
| FP | 721 | |
| FN | 170 | |
| Precision | 0.4012 | |
| Recall | 0.7397 | |
| F1 | 0.5202 | |
| mean_iou_of_matches | 0.8508 | |
| passed | `false` theo ngưỡng mặc định trong script (0.80/0.70); **đạt** theo ngưỡng BTC thông báo ngày 10/11 (F1 > 0.5, IoU > 0.8) | |

Kết quả: [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json).

Kiểm tra thêm (mở lại ảnh, so box của mình với gold, không sửa gold):

- **Lệch mapping lớp:** khi ghép box không xét lớp ở IoU ≥ 0.5, `truck` của tôi khớp gold ID 2 (13 box), `bus` khớp gold ID 3 (51 box), `van` khớp gold ID 1 (21 box). Các box này đúng vị trí nhưng bị tính FP + FN vì khác ID.
- **Khác quy ước độ phủ:** trung bình gold 13 box/ảnh, của tôi 24 box/ảnh. Gold bỏ qua xe ở xa cuối đường và xe đỗ ven đường (ví dụ dãy xe đỗ bên trái drive_049: gold 6 box, tôi 29 box; drive_002: gold 13, tôi 45). Recall khi không xét lớp là khoảng 0.92, cho thấy gần như không bỏ sót xe gold.
- **Box sát:** mean IoU 0.85 (mức "Xuất sắc" theo rubric).

Không rework vì đã đạt ngưỡng BTC. Nếu rework, hướng xử lý là bổ sung quy tắc vùng gán nhãn vào guideline (chỉ gán xe trên đường chính gần camera) và hỏi giảng viên về mapping lớp; không đổi ID chỉ để khớp gold khi chưa có xác nhận.

## 5. Kết quả YOLO

- Runtime: Google Colab, GPU (`device: 0`), tổng thời gian train khoảng 85 giây. Ultralytics: bản mới nhất cài bằng `pip install -U ultralytics` trong notebook; Colab clone đúng commit `f04246b`.
- Tham số: giữ nguyên notebook (`yolo26n.pt`, `epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`).
- Best checkpoint: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt` (mAP50 cao nhất ở epoch 1).

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.411 |
| Recall | 0.230 |
| mAP50 | 0.272 |
| mAP50–95 | 0.202 |

Recall từng lớp, tính từ [confusion matrix](assets/yolo26_runs/vehicle_eval/confusion_matrix.png) (cột "True" là nhãn gold eval):

| Lớp (ID theo data.yaml) | Box gold | Dự đoán đúng lớp | Recall |
|---|---:|---:|---:|
| car | 94 | 84 | 0.89 |
| truck | 5 | 1 (4 bị đoán `car`) | 0.20 |
| bus | 2 | 0 | 0.00 |
| van | 12 | 0 (11 bị đoán `bus`) | 0.00 |

Ảnh đánh giá: [assets/eval_predictions/visuals/](assets/eval_predictions/visuals/). Log train: [results.csv](assets/yolo26_runs/vehicle_annotation/results.csv), [results.png](assets/yolo26_runs/vehicle_annotation/results.png). Chỉ số eval: [assets/yolo26_runs/vehicle_eval/](assets/yolo26_runs/vehicle_eval/).

Ví dụ cụ thể: ở [drive_052](assets/eval_predictions/visuals/drive_052.jpg), xe buýt hai tầng giữa ảnh được model dự đoán `bus` (đúng với mắt người), nhưng gold eval ghi xe này ID 3 (`van`), nên bị tính sai. Tương tự xe buýt xanh lá ở [drive_053](assets/eval_predictions/visuals/drive_053.jpg) (dự đoán `bus` 0.8).

Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   `bus` và `van` cùng recall 0. Minh họa: drive_052 và drive_053, xe buýt được dự đoán `bus` nhưng gold ghi `van`. Confusion matrix cho thấy 11/12 box gold "van" bị dự đoán "bus". Recall 0 ở đây phần lớn do gold eval dùng mapping 1 = van, 2 = truck, 3 = bus, không hẳn do model không nhận ra xe.

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   mAP50 chỉ yêu cầu IoU ≥ 0.5; mAP50–95 lấy trung bình trên các ngưỡng 0.50 đến 0.95. Nếu mAP50 cao mà mAP50–95 thấp thì model tìm đúng vật thể nhưng box lỏng hoặc lệch: cần kiểm tra các cạnh box có ôm sát thân xe không, có lấn bóng đổ/nền không, và độ sát của nhãn train. Ở bài này 0.202 / 0.272 ≈ 0.74, hai chỉ số không chênh nhiều, nên box khi đã khớp là khá sát (phù hợp mean IoU nhãn 0.85). Vấn đề chính là phân lớp và độ phủ.

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ?**
   - Lệch mapping giữa schema README (nhãn train) và gold: dữ kiện là confusion matrix (gold "van" → dự đoán "bus" 11 lần) và kết quả ghép box ở mục 4.
   - Ranh giới van/car: 4 box gold ID 1 (thực chất là minivan) bị dự đoán `car`. Đây là ca mơ hồ số 2 trong guideline; dữ liệu train có ít `van` (34 box) so với `car` (1065).
   - Mất cân bằng lớp: `car` chiếm khoảng 88% box train, `truck` chỉ 14 box, nên model thiên về `car`.
   - Precision thấp một phần vì nhãn train gán cả xe nhỏ ở xa mà gold eval bỏ qua, nên các dự đoán này bị tính FP.
   - mAP50 cao nhất ở epoch 1 (0.272) rồi giảm còn khoảng 0.246 ở epoch 50: trọng số pretrained đã nhận diện tốt `car`, còn fine-tune trên nhãn khác mapping với eval không cải thiện điểm eval.

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng?**
   Trong notebook, 10 ảnh eval được dùng làm validation sau mỗi epoch để chọn `best.pt` và để tính chỉ số cuối. Nếu đưa vào train, model đã "thấy đáp án" (rò rỉ dữ liệu), chỉ số bị thổi phồng và không còn đo được khả năng tổng quát hóa. Vì eval đã được dùng để chọn checkpoint, nó cũng không phải test set độc lập; kết quả chỉ mang tính tham khảo.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CVAT, 09/11 tối | Phím tắt vẽ lại hiện "Press undefined to draw again", phím N không hoạt động | Settings → Shortcuts gán lại phím; tạm thời bấm nút Shape | Gán nhãn tiếp được |
| Colab, 09/11 khoảng 23:30 | "Không gán được phần phụ trợ"; Colab báo sự cố cấp runtime toàn hệ thống | Đợi, chạy lại sáng 10/11 | Chạy thành công trên GPU sáng 10/11 |
| Gold QC | F1 0.52 thấp do lệch mapping lớp và khác quy ước độ phủ | Ghép box không xét lớp để tách nguyên nhân | Đạt ngưỡng BTC; cần giảng viên xác nhận mapping |

## 7. Tự đánh giá theo rubric

| Tiêu chí | Tự đánh giá |
|---|---|
| Hoàn thành và định dạng YOLO (10) | Đã có: 50/50 file, checker đạt |
| Đầy đủ vật thể (20) | Đã có: recall không xét lớp ≈ 0.92; gán dư xe ở xa so với quy ước gold |
| Độ bám sát box (20) | Đã có: mean IoU 0.85 |
| Phân lớp nhất quán (15) | Đã sửa qua self-QC; còn lệch ID so với gold do mapping, chờ giảng viên xác nhận |
| Mini guideline (10) | Đã có: định nghĩa 4 lớp, quy tắc box, 4 ca mơ hồ có lý do |
| Gold-set QC (15) | Đã có JSON; đạt ngưỡng BTC; phân tích nguyên nhân FP/FN |
| Pipeline YOLO và phân tích (10) | Đã có: Colab chạy được, giữ đúng split, có metrics, ảnh eval và nhận xét |

## 8. Công cụ AI và tài liệu tham khảo

- **Claude (Anthropic)**, dùng ở các bước: hướng dẫn dùng CVAT; QC tự động bộ nhãn (vẽ box lên ảnh, thống kê, phóng to xe nghi ngờ để đối chiếu schema); hỗ trợ diễn đạt guideline và báo cáo này.
- Phần tôi tự làm và tự kiểm tra: gán toàn bộ 1204 box trên CVAT; quyết định lớp cho từng ca mơ hồ; tự sửa nhãn trên CVAT sau mỗi vòng QC; tự chạy notebook Colab/evaluate; đọc lại và xác nhận các số liệu trong báo cáo.
- Tài liệu ngoài repo: tài liệu CVAT về định dạng Ultralytics YOLO (docs.cvat.ai), tài liệu Ultralytics về tích hợp CVAT (docs.ultralytics.com).
