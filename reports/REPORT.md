# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên: Nhữ Đình Chiến
Ngày: 15/09/2026

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT (Computer Vision Annotation Tool)` |
| Thời gian gán `clip_02` (warm-up) | `30` phút |
| Thời gian gán `clip_01` | `60` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `6` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe bị che khuất một phần (occlusion)** bởi xe khác hoặc vệt che khuất trong cảnh: Xử lý bằng cách thu hẹp bounding box ôm sát vùng nhìn thấy được (visible area) và duy trì Track ID nếu thời gian bị che dưới ngưỡng 25 frames.
2. **Bounding box bị trôi/chệch (interpolation drift)** ở các frame giữa hai keyframe khi xe tăng/giảm tốc độ hoặc chuyển làn: Xử lý bằng cách đặt bổ sung các keyframe trung gian tại các mốc xe thay đổi vận tốc hoặc hướng đi.
3. **Xe xuất hiện chớm ranh giới khung hình hoặc dần rời khỏi khung hình (edge clipping)**: Xử lý bằng cách bám sát ranh giới viền ảnh, bắt đầu track ngay khi xác định rõ là vehicle và ngắt track ngay khi xe rời hẳn khung hình để tránh ghost box.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- **Lượt 1:** Bắt được tính nhất quán ID của từng xe từ đầu đến cuối clip, đảm bảo 1 xe giữ đúng 1 ID và không bị nhảy ID (ID switch) hay dùng lại ID cũ cho xe mới.
- **Lượt 2:** Bắt được điểm bắt đầu (entry) và điểm kết thúc (exit) của từng track, phát hiện và xóa 2 ghost boxes ở cuối track ID 4 (frame 150-151) và đầu track ID 5 (frame 76-78).
- **Lượt 3:** Bắt được độ ôm sát của bounding box ở các frame giữa 2 keyframe, chỉnh sửa 1 loose box ở track ID 6 (frame 104) bị chệch viền sang xe bên cạnh.

Kiểm chéo với: Bạn đồng học (Pair Reviewer). Chi tiết ở [`reports/review_partner.md`](file:///c:/Users/Admin/Desktop/K4-L2-DAY03-NhuDinhChien-2A202602130-VideoTracking/reports/review_partner.md).
Số lỗi bạn tìm được trong bản của bạn ấy: `2`. Số lỗi bạn ấy tìm được trong bản của bạn: `3`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Ca phân vân khi xe ở sát rìa khung hình chỉ còn thấy một phần rất nhỏ (<10% diện tích). Luật trong [`GUIDELINE_MINI.md`](file:///c:/Users/Admin/Desktop/K4-L2-DAY03-NhuDinhChien-2A202602130-VideoTracking/GUIDELINE_MINI.md) còn thiếu quy định về tỷ lệ diện tích/kích thước tối thiểu (ví dụ: visible area >= 10% hoặc bbox >= 15x15 pixel) để quyết định bắt đầu/kết thúc track.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `c28fc512967266f3afb2b0246e83fe9c284b21e51ee24db5d7928ae7c815b165` |
| Thời điểm khóa | `15/09/2026 15:30:00+07:00` |
| Số row / frame / track trước khi mở reference | `613 / 190 / 8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.7725 | 0.7510 | 0.8012 | 0.8520 | 0.9210 | 0.8850 | 0.8410 | 58 | 12 | 1 |
| Sau rework | 0.8062 | 0.7936 | 0.8207 | 0.8675 | 0.9646 | 0.9267 | 0.8541 | 41 | 1 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Ghost box | 149 - 151 | 4 | Xóa box thừa 2 frame sau khi xe ID 4 đã ra khỏi rìa phải khung hình |
| Ghost box | 76 - 78 | 5 | Cắt 3 frame vẽ sớm trước khi xe ID 5 thực sự xuất hiện rõ |
| Loose box | 104 | 6 | Điều chỉnh viền bounding box ôm sát thân xe ID 6, loại bỏ phần đè sang xe đi bên |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ [`outputs/model_run_config.json`](file:///c:/Users/Admin/Desktop/K4-L2-DAY03-NhuDinhChien-2A202602130-VideoTracking/outputs/model_run_config.json):

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `Python 3.13.15 / Ultralytics 8.4.145 / PyTorch 2.11.0+cu128 / LAP 0.5.13` |
| weights / hai tracker | `yolo26n.pt / ByteTrack (bytetrack.yaml) & BoT-SORT + ReID (botsort-reid.yaml)` |
| conf / IoU / imgsz / classes | `conf: 0.25 / IoU: 0.70 / imgsz: 960 / classes: [2, 5, 7]` |
| device | `GPU 0` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.8062 | 0.7936 | 0.8207 | 0.8675 | 0.9646 | 0.9267 | 0.8541 | 41 | 1 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.7394 | 0.6771 | 0.8131 | 0.8664 | 0.8825 | 0.7651 | 0.8511 | 83 | 61 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

Trong bản gán nhãn của bạn vs Gold, MOTA (0.9267) thấp hơn IDF1 (0.9646). Tuy nhiên ở mô hình ByteTrack control, MOTA (0.7487) lại thấp hơn hẳn IDF1 (0.8746). 
Nếu trường hợp MOTA cao mà IDF1 thấp xảy ra, điều đó phản ánh rằng Detector phát hiện vật thể ở từng frame đơn lẻ rất tốt (ít FP/FN), nhưng Tracker liên kết đối tượng qua thời gian lại rất kém (xảy ra nhiều lỗi đứt gãy track, nhảy ID khi bị che khuất hoặc khi xe tạm rời khung hình). 
MOTA không phạt nặng lỗi ID vì công thức `MOTA = 1 - (FP + FN + IDSW) / GT_boxes`. Do tổng số box chuẩn trong video rất lớn (573 boxes), mỗi lần nhảy ID chỉ bị trừ đúng 1 đơn vị IDSW (ảnh hưởng < 0.5% tới MOTA). Ngược lại, IDF1 đo lường sự khớp chuỗi ID dài hạn dựa trên F1-score của các ID được gán tương ứng, nên khi 1 track bị chia cắt thành nhiều ID ngắn thì IDF1 sẽ sụt giảm rất nặng.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

So sánh với Gold, BoT-SORT + ReID vượt trội ByteTrack ở mọi chỉ số liên kết: `IDF1` tăng từ 0.8746 lên 0.9001 (+0.0255), `AssA` tăng từ 0.7761 lên 0.8204 (+0.0443), số `IDSW` giữ ở mức 2.
Xét chuỗi frame 85 - 115 khi xe ID 5 và ID 6 bị che khuất bán phần / lướt qua khu vực nhiễu: ByteTrack bị đứt track khiến FN tăng vọt (54 FN ở ByteTrack so với 26 FN ở ReID) và đứt gãy track ID 5 sang ID 32. Trong khi đó BoT-SORT + ReID nhờ trích xuất vector đặc trưng ngoại hình đã tìm lại và duy trì đúng track ID khi xe xuất hiện lại, nâng tỷ lệ coverage từ 0.77 lên 0.79.
Cần lưu ý rõ: Sự cải thiện này KHÔNG cô lập hoàn toàn tác động nhân quả (causal effect) của riêng module ReID, vì ByteTrack và BoT-SORT còn khác nhau ở mô hình chuyển động Kalman Filter, cơ chế bù chuyển động camera (GMC - Global Motion Compensation) và thuật toán ghép cặp Hungarian Matching.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

Về độ chính xác phát hiện: `DetA` của BoT-SORT + ReID đạt 0.7110 (cao hơn 0.6487 của ByteTrack).
Số lượng `FP` của hai mô hình tương đương nhau (88 FP ở ByteTrack vs 91 FP ở ReID) do dùng chung detector `yolo26n.pt` với threshold conf = 0.25. Tuy nhiên, số lượng `FN` của ReID giảm hơn 50% (từ 54 FN xuống 26 FN) nhờ tracker tái phát hiện và giữ vết tốt hơn.
Lỗi còn lại chủ yếu nằm ở **Detector** (`yolo26n.pt` phiên bản nano). Mức FP vẫn cao (~90 FP trên 190 frames do detector nhận diện nhầm vật thể tĩnh bên đường thành xe, thể hiện ở các Ghost tracks 7, 27, 38). Ngoài ra conf = 0.25 vừa làm sót xe xa (FN) vừa bắt nhầm nhiễu (FP).

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Tại chuỗi Frame 16 - 116 (dài 43 frames), mô hình ReID tạo ra Ghost Track **ID 7** (`eval_reid_vs_gold.json` chẩn đoán: `pred_track 7`, lý do: `"không khớp track tham chiếu nào"`).
Bạn (người gán nhãn) xác định chính xác đây là vệt bóng cây / chướng ngại vật cố định bên đường không phải xe bốn bánh nên KHÔNG gán nhãn. Ngược lại mô hình ReID bị detector nhận diện nhầm và tracker duy trì nhầm track này liên tục 43 frame.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Tại Frame 104 - 107 (Track ID 6):
Mô hình ReID báo ID switch tại Frame 107 (chuyển ID 24 -> 28 -> 31) và Loose box ở Frame 104 (IoU 0.522). Khi kiểm tra lại annotation ban đầu ở đoạn này, xe ID 6 bị xe đi bên cạnh vượt qua. Ban đầu bạn vẽ box kéo hơi chệch sang thân xe bên cạnh. Mô hình ReID với feature extractor nhận diện sự thay đổi ngoại hình đột ngột (ReID feature distance spike) tại frame 107 dẫn đến nhảy ID, qua đó giúp bạn phát hiện và sửa lại viền bounding box ở frame 104-106 cho ôm sát thân xe.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- **Sửa trong [`GUIDELINE_MINI.md`](file:///c:/Users/Admin/Desktop/K4-L2-DAY03-NhuDinhChien-2A202602130-VideoTracking/GUIDELINE_MINI.md):**
  1. Bổ sung định lượng rõ ràng: Bắt đầu gán nhãn khi xe xuất hiện có diện tích visible >= 10% hoặc size >= 15x15 pixel.
  2. Bổ sung luật kiểm tra interpolation drift: Đặt khoảng cách tối đa giữa 2 keyframe liên tiếp không quá 15 frames ở các đoạn đường xe di chuyển không đều.
  3. Làm rõ quy tắc giữ ID khi occlusion ngắn (<25 frames) kèm ví dụ hình ảnh minh họa.

- **Đổi trong quy trình làm việc:**
  1. Áp dụng quy trình Auto-annotation: Chạy pre-annotation bằng mô hình BoT-SORT + ReID trước để tạo khung sẵn, sau đó human-in-the-loop chỉ việc chỉnh sửa (tốc độ gán sẽ nhanh gấp 3 lần, giảm từ 60 phút xuống ~20 phút/clip).
  2. Thực hiện quy trình QC 3 lượt (Identity, Endpoint, Geometry) ngay sau khi hoàn thành từng clip thay vì dồn vào cuối buổi.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
