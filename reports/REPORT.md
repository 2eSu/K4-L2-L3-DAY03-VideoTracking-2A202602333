# Báo cáo Ngày 3 — Tracking Annotation


- Họ tên / nhóm: `Vũ Trung Hiếu`
- Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `20` phút |
| Thời gian gán `clip_01` | `40` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `Chưa ghi lại` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe bị che khuất tạm thời bởi xe khác hoặc vật cản. Tôi tua chậm các frame trước/sau khi bị che, giữ nguyên ID cũ khi xe xuất hiện lại và đặt keyframe dày hơn quanh đoạn che khuất.
2. Hai xe đi gần nhau hoặc giao cắt khiến dễ nhầm ID. Tôi đối chiếu hướng di chuyển, vị trí trước khi giao cắt và kích thước/đặc điểm xe để duy trì đúng identity cho từng track.
3. Xe vào hoặc rời khỏi khung hình, đặc biệt khi chỉ xuất hiện một phần. Tôi đặt điểm bắt đầu/kết thúc đúng frame xe thực sự xuất hiện/rời khung và dùng Outside để kết thúc track, tránh tạo box ở các frame không còn xe.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Có 8 ID trong `clip_01`; đối chiếu với gold không có ID switch, không có track bị tách và không có track thừa. Đây là kết quả của `outputs/eval_vs_gold.json`, không phải bằng chứng thay thế cho việc xem video bằng mắt.
- Lượt 2: Các biên cần chú ý là track 4 bắt đầu ở frame 60 (gold: 54), track 5 ở frame 80 (gold: 79), track 6 ở frame 105 (gold: 101), track 7 ở frame 107 (gold: 106), và track 8 ở frame 136. Không có bbox treo theo chẩn đoán; một số điểm kết thúc cũng lệch nhẹ so với gold, như track 4 kết thúc ở 150 (gold: 148) và track 5 ở 139 (gold: 138).
- Lượt 3: Cần soi lại hình học quanh frame 84, 88 của ID 5 và frame 105, 110 của ID 6. Đây là các bbox có IoU thấp nhất trong chẩn đoán (`0.522` đến `0.543`), dù vẫn giữ đúng identity.


## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `971bee1bcdd70e48e51b1140c30bc1048877ebc2de6a680d7df56af0f2a9704d` |
| Thời điểm khóa | `2026-09-15T09:06:30.140783+00:00` |
| Số row / frame / track trước khi mở reference | `566 / 190 / 8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.8512 | 0.8388 | 0.8654 | 0.8896 | 0.9693 | 0.9389 | 0.8824 | 14 | 21 | 0 |
| Sau rework | 0.8512 | 0.8388 | 0.8654 | 0.8896 | 0.9693 | 0.9389 | 0.8824 | 14 | 21 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Không có rework được ghi nhận | - | - | Snapshot pre-gold và `annotations/clip_01/gt.txt` có cùng SHA-256, số row và metrics. |
| | | | |
| | | | |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.9 / 8.4.145 / 2.14.0+cpu / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml / configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7]` |
| device | `cpu` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.8512 | 0.8388 | 0.8654 | 0.8896 | 0.9693 | 0.9389 | 0.8824 | 14 | 21 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.8063 | 0.7536 | 0.8629 | 0.9077 | 0.9086 | 0.8092 | 0.8992 | 89 | 17 | 2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi thấp hơn IDF1 một chút (`0.9389` so với `0.9693`). MOTA phạt FP, FN và IDSW trên tổng số ground-truth object-frame; IDF1 tập trung vào độ chính xác của liên kết identity sau matching. Vì vậy MOTA cao nhưng IDF1 thấp có thể xảy ra khi detector bắt đúng vị trí và số lượng xe nhưng tracker đổi ID hoặc tách track nhiều lần. MOTA không phạt nặng lỗi ID vì IDSW chỉ là một thành phần cộng trong công thức MOTA, còn IDF1 trực tiếp tính IDTP/IDFP/IDFN.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

BoT-SORT + ReID tốt hơn ByteTrack trên clip này: IDF1 tăng từ `0.8750` lên `0.9003`, AssA từ `0.7758` lên `0.8197`, còn IDSW giữ nguyên ở `2`. ByteTrack có switch tại frame 59 (gold ID 4: model ID 14 -> 15) và frame 94 (gold ID 5: ID 23 -> 32). ReID cũng chưa giải quyết hoàn toàn identity: có switch tại frame 87 (gold ID 5: ID 17 -> 18) và frame 113 (gold ID 6: ID 24 -> 31). Trong chuỗi khoảng frame 85-94, ReID giữ được một đoạn track dài hơn nhưng vẫn tách ID 5 quanh frame 87; treatment tốt hơn tổng thể nhưng không tốt hơn ở mọi frame. Đây là system comparison, không cô lập causal effect của ReID vì ByteTrack và BoT-SORT là hai tracker implementation khác nhau.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng từ `0.6492` lên `0.7110`, FN giảm mạnh từ `54` xuống `26`, cho thấy treatment bắt được nhiều object-frame hơn. Tuy nhiên FP tăng nhẹ từ `88` lên `91`, nên treatment cũng tạo thêm bbox không khớp. Phần lỗi còn lại là cả detector lẫn association: detector thể hiện qua FP/FN và các bbox lệch; association thể hiện qua 2 IDSW và việc gold ID 5, 6, 7 vẫn bị chia thành nhiều track ReID. ReID không thể sửa một detection mà YOLO không tạo ra.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Frame 55-59, ReID có ID 7 nhưng ID này không khớp track nào trong gold; nó là một track thừa kéo dài tới frame 116. Nhãn tay không tạo bbox tương ứng ở đoạn này, nên ở finding này annotation đúng hơn và ReID là false positive. Evidence chấm trực tiếp ghi `ID 7: không khớp track tham chiếu nào (frame 16-116, 43 frame)`.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

ReID không cung cấp đủ evidence để buộc sửa annotation. Ở frame 87, 88 và 94, ReID tách gold ID 5 thành các ID model khác nhau; ở frame 113, gold ID 6 cũng bị đổi ID. Ngoài ra ReID có 16 track so với 8 track của annotation, `91 FP`, `26 FN` và `2 IDSW`. Vì vậy các bất đồng đó phù hợp hơn với lỗi detector/association của model, không phải lý do để đổi ID annotation. Các bbox annotation có IoU thấp quanh frame 84, 88, 105 và 110 vẫn nên được xem lại hình học độc lập.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Tôi sẽ sửa `GUIDELINE_MINI.md` thành luật có thể kiểm tra được: giữ ID khi che dưới 25 frame ở 12.5 fps, track mới nếu xe đã ra khỏi khung, và ghi ngưỡng tối thiểu để xác định xe nhỏ/mờ. Tôi cũng sẽ thêm ví dụ frame thật cho entry/exit và crossing, quy định bbox chỉ ôm phần nhìn thấy, cùng quy tắc đặt keyframe dày quanh occlusion/rẽ/ra khung.

Với 10 clip tiếp theo, tôi sẽ lưu frame đầu/cuối của từng track ngay khi gán, chạy ba lượt tua ngay sau mỗi clip, kiểm tra MOT trước khi khóa pre-gold, và lưu reviewer findings theo mẫu với `frame + ID + closure`. Tôi sẽ khóa snapshot trước khi mở gold/model, sau đó chạy đủ bốn phép đánh giá để không nhầm model output với ground truth.

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
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
