# Peer review — Day 3

Reviewer chỉ ghi finding; tác giả tự sửa bài của mình và điền closure.

| Trường | Giá trị |
| --- | --- |
| Author | Nhữ Đình Chiến |
| Reviewer | Bạn đồng học (Pair Reviewer) |
| Pair ID | PAIR-DAY03-01 |
| CVAT version | 2.15.0 |
| Thời điểm review | 15/09/2026 14:00 |

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất, dùng `needs-review`; không ép tác giả sửa theo cảm tính.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 149 | 150 | 4 | Ghost box | Box còn kéo dài 2 frame sau khi xe đã ra khỏi lề phải | Cắt bỏ box ở frame 150-151 | fixed |
| 2 | 76 | 77 | 5 | Ghost box | Box bắt đầu vẽ sớm 3 frame trước khi xe đi vào khung | Bắt đầu track từ frame 79 | fixed |
| 3 | 104 | 105 | 6 | Loose box | Bounding box bị chệch sang phần xe bên cạnh | Co gọn box ôm đúng phần xe 6 | fixed |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | clip_01 có đủ 8 tracks vehicle |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | Không phát hiện ID switch sai |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | Frame 85-95 xe ID 5 giữ nguyên ID |
| Entry/exit đúng; không box treo sau khi xe rời khung | PASS | Đã sửa tại frame 149-151 (ID 4) |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS | Đã chỉnh loose box frame 104 (ID 6) |
| Frame giữa hai keyframe không bị interpolation drift | PASS | IoU đạt > 0.85 trên toàn chuỗi |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | Đã kiểm tra format CSV gt.txt |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | Đã điền closure = fixed cho tất cả |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | 8 tracks khớp timeline |
| 2 — endpoint/scope | ĐÃ SỬA | Đã sửa endpoint ID 4 và ID 5 |
| 3 — geometry/interpolation | ĐÃ SỬA | Đã sửa geometry ID 6 |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: Cần kết thúc track chính xác tại frame xe hoàn toàn rời khỏi khung hình, áp dụng Rule Endpoint trong `GUIDELINE_MINI.md`.
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): Không có (tất cả các finding đều hợp lý và đã được sửa).
3. Một rule cần Lab Coach làm rõ (nếu có): Quy định về tỷ lệ diện tích tối thiểu khi xe vừa xuất hiện ở rìa ảnh.
