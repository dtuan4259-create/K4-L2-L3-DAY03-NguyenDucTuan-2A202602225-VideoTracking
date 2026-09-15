# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Nguyễn Đức Tuấn`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `30` phút |
| Thời gian gán `clip_01` | `60` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `9` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Các xe bị che khuất/chồng lấn trong một số đoạn, đặc biệt ở các track 4, 5 và 7. Tôi giữ ID theo cùng một đối tượng xuyên suốt, ưu tiên đối chiếu frame trước/sau vùng che thay vì tạo ID mới chỉ vì bbox tạm thời khó nhìn.`

2. `Các xe ở rìa khung hoặc chỉ xuất hiện một phần, như track 6 và 8. Tôi kiểm tra thêm frame kế cận để xác định xe còn ở trong khung hay đã rời khung, tránh kéo bbox quá sớm hoặc quá muộn.`

3. `Kích thước/vị trí bbox thay đổi mạnh khi xe tiến gần hoặc đi ra khỏi khung. Tôi rà lại frame đầu/cuối và các frame giữa, đồng thời giữ bbox bám sát phần xe nhìn thấy thay vì cố suy diễn phần bị che.`


## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `Kiểm tra tính liên tục của ID; nhãn của tôi không có IDSW khi đối chiếu với gold (IDSW = 0).`
- Lượt 2: `Rà frame đầu/cuối phát hiện một lỗi biên của track 4: bbox của ID4 còn tồn tại ở frame 149–151 sau khi track tham chiếu 4 đã rời khung.`
- Lượt 3: `Rà các frame giữa cho thấy một số bbox có IoU sát ngưỡng ở track 5/6/8; các frame tiêu biểu cần xem lại là 91–93 (ID5), 101–102 (ID6) và 168 (ID8).`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `...` |
| Thời điểm khóa | `...` |
| Số row / frame / track trước khi mở reference | `568 row / 190 frame / 8 track trong annotation hiện có; chưa đủ evidence để xác nhận đây chính xác là snapshot pre-gold.` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | | | | | | | | | | |
| Sau rework | | | | | | | | | | |

Số liệu annotation hiện có khi đối chiếu với gold: `HOTA=0.8023, DetA=0.7905, AssA=0.8163, LocA=0.8520, IDF1=0.9781, MOTA=0.9564, MOTP=0.8347, FP=10, FN=15, IDSW=0.`

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Track kéo dài sau khi đối tượng rời khung | 149–151 | 4 | `Cắt track ID4 tại frame cuối thực sự còn xuất hiện; không giữ bbox sau khi xe đã rời khung.` |
| Bbox lỏng | 91–93 | 5 | `Rà lại biên bbox theo hình xe; đây là nhóm frame có IoU thấp 0.531–0.586 khi so với gold.` |
| Bbox lỏng | 101–102 | 6 | `Rà lại vị trí/kích thước bbox ở vùng xe bị che/khó quan sát; IoU khoảng 0.598–0.597 khi so với gold.` |

`Lưu ý: file eval_vs_gold hiện có xác nhận các lỗi/điểm cần rework ở trên, nhưng chưa có một file annotation sau rework riêng để xác nhận rằng mọi thay đổi đã được lưu vào một phiên bản mới.`

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml / botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7]` |
| device | `0` |
| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.8023 | 0.7905 | 0.8163 | 0.8520 | 0.9781 | 0.9564 | 0.8347 | 10 | 15 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.7456 | 0.6833 | 0.8176 | 0.8595 | 0.8939 | 0.7746 | 0.8422 | 99 | 29 | 0 |

**## 5. Phân tích — năm câu hỏi**

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA của tôi (0.9564) thấp hơn IDF1 (0.9781). Điều này cho thấy bài toán annotation của tôi nhìn chung rất ổn cả về phát hiện và duy trì identity. Nói chung, khi MOTA cao nhưng IDF1 thấp thì có thể hiểu là số lỗi FP/FN và phát hiện tổng thể vẫn khá tốt nhưng identity association còn kém. MOTA chủ yếu tổng hợp FN, FP và ID switch, nên không phản ánh chi tiết độ nhất quán identity qua toàn bộ quỹ đạo như IDF1; vì vậy có thể có lỗi ID nhưng MOTA vẫn khá cao.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`BoT-SORT + ReID có IDF1=0.9001 cao hơn ByteTrack=0.8746 (+0.0255), AssA=0.8204 cao hơn 0.7761 (+0.0443), nhưng IDSW đều bằng 2 nên không giảm số IDSW trong clip này. Một sequence tiêu biểu là khoảng frame 85–94 quanh GT track 5: ByteTrack bị ID switch tại frame 94 (23 -> 32), còn ReID bị ID switch sớm hơn tại frame 87 (17 -> 18). Như vậy ReID không cải thiện số IDSW ở sequence này, nhưng điểm AssA/IDF1 tổng thể vẫn cao hơn. Đây là system comparison, chưa thể kết luận chênh lệch hoàn toàn do ReID vì hai tracker implementation khác nhau.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`So với ByteTrack, ReID tăng DetA từ 0.6487 lên 0.7110, giảm FN mạnh từ 54 xuống 26, nhưng FP tăng nhẹ từ 88 lên 91. Vì số FN giảm 28 trong khi FP chỉ tăng 3, treatment giúp bắt được nhiều target hơn và chất lượng detection/coverage tốt hơn. Association vẫn còn lỗi vì IDSW vẫn là 2 và có các track bị fragment. Vì vậy phần lỗi còn lại là hỗn hợp, nhưng sau treatment vấn đề detection được cải thiện rõ hơn; association vẫn là điểm cần soi ở các vùng giao nhau/occlusion.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Khoảng frame 87–88, ReID có thêm pred track 7; diagnostics của eval_reid_vs_me ghi đây là ghost track vì không khớp track tham chiếu nào, kéo dài tổng cộng 43 frame. Trong cùng vùng, nhãn của tôi không có track tương ứng. Vì vậy đây là ví dụ model tạo thêm một track không được annotation tham chiếu xác nhận; không nên sửa annotation chỉ để khớp model.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Frame 149–151, ID4 là chỗ cần xem lại annotation: eval_vs_gold ghi rõ pred track 4 còn bbox sau khi track tham chiếu 4 đã rời khung. Đây là một lỗi biên của annotation cần cắt ở điểm kết thúc hợp lý. Ngoài ra, frame 91–93 của ID5 và 101–102 của ID6 có IoU thấp với gold, nên cũng cần rà lại hình học bbox. Evidence hiện tại cho thấy lỗi này nằm ở annotation của tôi, không phải lý do để sửa theo model một cách máy móc.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Tôi sẽ bổ sung rõ ba quy tắc: (1) tiêu chí kết thúc track khi đối tượng thực sự rời khung, đặc biệt ở mép trái/phải; (2) cách xử lý xe bị che một phần: giữ cùng ID nếu vẫn còn bằng chứng liên tục, không tạo ID mới chỉ vì bbox tạm thời mất/nhỏ; (3) khi bbox chỉ còn một phần ở mép khung, ưu tiên kiểm tra frame trước và sau để xác định frame đầu/cuối. Về quy trình, tôi sẽ luôn tua ba lượt riêng cho ID, frame đầu/cuối và frame giữa; sau đó mới xem model như một công cụ diagnostic để tìm điểm đáng nghi, không dùng model làm reference để quyết định annotation.`

**## 7. Tệp đã nộp**

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
