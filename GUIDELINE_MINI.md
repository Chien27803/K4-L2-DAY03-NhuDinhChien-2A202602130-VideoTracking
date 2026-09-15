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
| Xe con, SUV, taxi, xe bán tải (pickup) | Người đi bộ |
| Van, minivan | Xe đạp, xe đẩy tay |
| Xe buýt, minibus | **Xe máy / mô tô / xe gắn máy 2-3 bánh** |
| Xe tải, xe container, xe đầu kéo | Xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

**Bổ sung của nhóm:** Bỏ qua xe mô tô, xe gắn máy 2-3 bánh và người đi bộ ngay cả khi di chuyển song song hoặc ở cùng làn đường với xe bốn bánh.

---

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Bảo đảm tính liên tục của đối tượng (tracklet continuity) theo chuẩn MOT |
| Xe bị che lâu hơn ngưỡng trên | Ngắt track cũ, tạo ID track mới khi xe xuất hiện lại | Tránh gán sai ID (ID switch) khi không đủ bằng chứng Re-identification |
| Xe rời khung hình rồi quay lại | Mặc định: **tạo track ID mới** | Đảm bảo tính nhất quán thực nghiệm, không đưa ra giả định vô căn cứ |
| Hai xe cắt nhau / chồng lên nhau (crossing/occlusion) | Xe bị che duy trì ID, bbox thu gọn ôm phần visible; xe đằng trước duy trì ID bình thường | Tránh nhảy ID (ID switch) giữa hai đối tượng cắt nhau trong khu vực giao cắt |

---

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa viền ảnh, tuyệt đối không suy đoán phần thân xe nằm ngoài ảnh |
| Xe bị xe khác che một phần | Bbox ôm sát phần **nhìn thấy được** (visible region), không bao phủ vùng bị che |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng diện tích: **>= 15x15 pixel** hoặc diện tích visible >= 10% |
| Xe đang đỗ, không di chuyển | Giữ nguyên bounding box cố định qua các frame trung gian, duy trì duy nhất 1 Track ID |
| Keyframe đặt dày ở đâu | Đặt keyframe dày (mỗi 2-5 frame) ở đoạn xe đổi hướng, tăng/giảm tốc độ, chuyển làn hoặc bắt đầu/kết thúc occlusion |

---

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- **Clip / frame / ID:** `clip_01` / Frame 85–95 / ID 5
- **Tình huống:** Xe ID 5 di chuyển qua khu vực bị xe lớn đi trước che khuất mất 80% phần đầu xe.
- **Quyết định:** Giữ nguyên Track ID 5, thu hẹp bounding box ôm sát 20% đuôi xe còn nhìn thấy được (visible region).
- **Lý do:** Thời gian bị che dưới 25 frame (10 frame) và phần đuôi xe vẫn nhận dạng rõ ràng thuộc về xe ID 5.

### Ca 2
- **Clip / frame / ID:** `clip_01` / Frame 149–151 / ID 4
- **Tình huống:** Xe ID 4 di chuyển sát mép phải khung hình, ở frame 150 chỉ còn hở 5% viền gương chiếu hậu.
- **Quyết định:** Kết thúc track ở Frame 149.
- **Lý do:** Tránh tạo ghost box (box treo) ngoài khung hình khi vật thể không còn đủ đặc trưng nhận diện tối thiểu (visible < 10%).

### Ca 3
- **Clip / frame / ID:** `clip_01` / Frame 104–107 / ID 6
- **Tình huống:** Xe ID 6 chạy song song và bị chèn góc quay bởi xe bên cạnh, khiến nội suy tự động đè viền box sang xe đi bên.
- **Quyết định:** Bổ sung 2 keyframe liên tiếp ở frame 104 và 106 để chỉnh lại viền bbox chuẩn xác.
- **Lý do:** Loại bỏ lỗi loose box (bbox lỏng lẻo) chệch sang thân xe khác, nâng chỉ số IoU từ 0.52 lên 0.85.

---

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- **Bổ sung ngưỡng diện tích tối thiểu ở rìa khung hình:** Quy định rõ chỉ bắt đầu tạo track hoặc kết thúc track khi diện tích hiển thị nhìn thấy được (visible area) của xe đạt tối thiểu **>= 10%** diện tích xe hoặc bounding box đạt từ **15x15 pixel** trở lên.
- **Bổ sung quy tắc kiểm tra Interpolation Drift:** Đặt khoảng cách tối đa giữa 2 keyframe liên tiếp không quá **15 frames** ở những đoạn đường xe di chuyển với tốc độ không đều hoặc chuyển hướng nghiêng.
- **Làm rõ quy tắc Occlusion ngắn:** Giữ nguyên Track ID cho các trường hợp xe bị che khuất tạm thời dưới **25 frames** (tương đương 2 giây ở tốc độ 12.5 fps), miễn là vị trí và hướng di chuyển khi xuất hiện lại khớp với vận tốc ban đầu của xe.
