# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: Vũ Trung Hiếu
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

Bổ sung của nhóm (nếu có): chỉ gán xe thật trong cảnh; không gán vật thể tĩnh hoặc hình xe xuất hiện trong biển quảng cáo/gương.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (2 giây ở 12.5 fps) | một xe vẫn là cùng identity; kiểm tra vị trí, hướng đi và kích thước trước/sau che |
| Xe bị che lâu hơn ngưỡng trên | tạo track mới khi không thể nối chắc với ID cũ sau 25 frame | tránh nối nhầm hai xe giống nhau chỉ vì cùng xuất hiện gần vị trí cũ |
| Xe rời khung hình rồi quay lại | **track mới** | ra khỏi khung là kết thúc track; không giả định đó vẫn là xe cũ |
| Hai xe cắt nhau / chồng lên nhau | giữ ID theo quỹ đạo trước giao cắt; đối chiếu hướng đi, vị trí và đặc điểm xe, không đổi ID chỉ vì bbox chồng nhau | ưu tiên continuity của từng xe; đặt keyframe dày quanh đoạn giao cắt để tránh interpolation đổi xe |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; không dùng ngưỡng pixel cứng, nhưng phải thấy đủ hình dáng chuyển động/biên xe để phân biệt với người, xe máy hoặc vật thể nền |
| Xe đang đỗ, không di chuyển | vẫn giữ track trong toàn bộ thời gian xe còn nhìn thấy; chỉ bấm `outside` khi xe thực sự rời khung |
| Keyframe đặt dày ở đâu | đặt dày quanh lúc vào/ra khung, rẽ, phanh, che khuất, giao cắt và khi bbox đổi kích thước nhanh; đoạn đi thẳng đều có thể đặt thưa hơn |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 105 / ID 6`
- Tình huống: bbox của xe ở đoạn chuyển động/che khuất bị lệch hình học; đây là một trong các điểm IoU thấp nhất khi so với gold (`IoU 0.522`).
- Quyết định: giữ nguyên ID 6, chỉ ôm phần xe nhìn thấy và đặt keyframe dày hơn quanh đoạn này.
- Lý do: lỗi hình học không làm thay đổi identity; tách ID sẽ tạo fragmentation không cần thiết.

### Ca 2
- Clip / frame / ID: `clip_01 / frame 88 / ID 5`
- Tình huống: xe gần hoặc chồng với xe khác, bbox có nguy cơ trôi khi nội suy; IoU tại frame này chỉ khoảng `0.524`.
- Quyết định: tiếp tục dùng ID 5, kiểm tra frame trước/sau và chỉnh bbox theo phần nhìn thấy.
- Lý do: quỹ đạo liên tục trước và sau giao cắt đáng tin hơn việc cấp ID mới chỉ vì hai bbox gần nhau.

### Ca 3
- Clip / frame / ID: `clip_01 / frame 136 / ID 8`
- Tình huống: một xe mới đi vào cảnh và chỉ xác định rõ từ frame 136; cần quyết định thời điểm bắt đầu track.
- Quyết định: bắt đầu ID 8 ở frame 136, không vẽ ngược về các frame trước khi xe chưa đủ rõ.
- Lý do: biên track phải là frame đầu tiên nhận diện chắc đó là xe bốn bánh, tránh bbox thừa ở nền.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Khi xe nhỏ hoặc mờ, không dùng kích thước bbox làm tiêu chí duy nhất; chỉ bắt đầu khi có đủ bằng chứng hình dáng xe bốn bánh và ghi lại frame bắt đầu.
- Khi xe bị che hoặc hai xe giao cắt, phải đặt keyframe dày và kiểm tra giữa hai keyframe; nếu xe đã ra khỏi khung thì kết thúc track, lần xuất hiện sau luôn dùng ID mới.
- Sau khi chấm gold, cần kiểm tra riêng các biên track lệch như ID 4 bắt đầu ở frame 60 thay vì gold frame 54, ID 6 bắt đầu ở frame 105 thay vì gold frame 101, và các bbox hình học quanh frame 84, 88, 105, 110. Các điểm này là finding để xem lại, không tự ý đổi ID nếu chưa có evidence hình ảnh.
