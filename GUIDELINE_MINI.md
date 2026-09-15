# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Nhữ Đình Chiến
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): Bỏ qua xe mô tô, xe gắn máy 2 bánh và người đi bộ ngay cả khi di chuyển cùng làn xe 4 bánh.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Bảo đảm tính liên tục đối tượng theo chuẩn MOT |
| Xe bị che lâu hơn ngưỡng trên | Ngắt track cũ, tạo ID track mới khi xuất hiện lại | Tránh gán sai ID (ID switch) khi không đủ bằng chứng ReID |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Đảm bảo tính nhất quán không giả định vô căn cứ |
| Hai xe cắt nhau / chồng lên nhau | Xe bị che duy trì ID, box thu gọn ôm phần visible; xe đằng trước duy trì ID bình thường | Tránh nhảy ID giữa hai đối tượng cắt nhau |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** (visible region) |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng diện tích >= 15x15 pixel |
| Xe đang đỗ, không di chuyển | Giữ nguyên bounding box tĩnh qua các frame, duy trì ID |
| Keyframe đặt dày ở đâu | Đặt dày ở đoạn xe đổi hướng, tăng/giảm tốc độ, hoặc bắt đầu bị che khuất |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01` / Frame 85-95 / ID 5
- Tình huống: Xe ID 5 di chuyển qua khu vực bị xe lớn đi trước che mất 80% phần đầu.
- Quyết định: Giữ nguyên Track ID 5, thu hẹp bbox ôm sát 20% đuôi xe còn nhìn thấy được.
- Lý do: Xe bị che dưới 25 frame và phần đuôi xe vẫn nhận dạng rõ ràng thuộc xe ID 5.

### Ca 2
- Clip / frame / ID: `clip_01` / Frame 149-151 / ID 4
- Tình huống: Xe ID 4 đã ra sát mép phải khung hình, chỉ còn hở 5% viền gương.
- Quyết định: Kết thúc track ở Frame 149.
- Lý do: Tránh tạo ghost box treo ngoài khung hình khi vật thể không còn đủ đặc trưng nhận diện.

### Ca 3
- Clip / frame / ID: `clip_01` / Frame 104-107 / ID 6
- Tình huống: Xe ID 6 chạy song song và bị chèn góc quay bởi một xe khác.
- Quyết định: Bổ sung 2 keyframe liên tiếp ở frame 104 và 106 điều chỉnh lại viền bbox.
- Lý do: Loại bỏ loose box chệch sang thân xe bên cạnh, tăng IoU từ 0.52 lên 0.85.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Quy định rõ hơn ngưỡng diện tích visible tối thiểu (>= 10%) để bắt đầu/kết thúc track ở rìa ảnh.
- Bổ sung luật kiểm tra interpolation drift giữa các keyframe cách xa nhau quá 15 frames.
