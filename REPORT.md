# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Ngô Lê Đức Anh`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `20` phút |
| Thời gian gán `clip_01` | `40` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `71` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Xe ra khỏi. Cách xử lý là khi xe rời khung, em dùng Outside đúng frame thay vì xóa track.  `
2. `Các xe giao nhau/crossing, dễ nhầm ID. Cách xử lý là em xem chậm từng frame quanh điểm giao nhau, đối chiếu vị trí và chuyển động của từng xe để giữ đúng ID, tránh ID switch hoặc tách một xe thành nhiều track.`
3. `Xe đi vào khung hình. Cách xử lý là khi xe xuất hiện, em kiểm tra frame vào để đảm bảo track được bắt đầu đúng vị trí.`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `phát hiện các lỗi về ID, đặc biệt là ID bị đổi hoặc một xe bị gán nhiều ID.`
- Lượt 2: `phát hiện lỗi ở frame đầu/cuối, như xe xuất hiện/mất khỏi khung hình sai thời điểm hoặc xử lý Outside chưa đúng.`
- Lượt 3: `phát hiện lỗi ở frame giữa, chủ yếu là mất track, đứt track hoặc bbox lệch khi xe bị che/khuất hay giao cắt.`

Kiểm chéo với: `lab coach`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `1`. Số lỗi bạn ấy tìm được trong bản của bạn: `1`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Hai người khác nhau ở cách xử lý xe rời khung hình và xuất hiện lại: cần quy định rõ frame kết thúc track bằng Outside và khi quay lại có giữ ID hay tạo track mới. Guideline hiện đã ghi mặc định tạo track mới nhưng chưa quy định rõ frame áp dụng Outside và cách xử lý khi xe chỉ khuất/rời khung trong thời gian ngắn.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `7dfab91901d66359d6f4ccb25b0f684e7dda96619d61e472c9ffb5b018f560d3` |
| Thời điểm khóa | `2026-09-15T14:51:46.214240+00:00` |
| Số row / frame / track trước khi mở reference | `row: 568, frame: 190, số track: 8` |

|                  |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP | FP | FN | IDSW |
| ---------------- | ----: | ----: | ----: | ----: | ----: | ----: | ----: | -: | -: | ---: |
| **Bản pre-gold** | 0.812 | 0.802 | 0.822 | 0.880 | 0.962 | 0.925 | 0.869 | 19 | 24 |    0 |
| **Sau rework**   | 0.815 | 0.803 | 0.827 | 0.885 | 0.960 | 0.921 | 0.874 | 13 | 32 |    0 |


Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Tách track | 85 | 6 → 29/39 | Kiểm tra lại track 6 và giữ cùng ID cho xe, không để bị tách thành ID 39 |
| Bbox lệch | 139 | 3 | Chỉnh lại bbox cho bám sát xe tại frame 139 |
| Bbox treo/thừa | 55–61 | 9 | Kiểm tra lại đoạn trước khi xe xuất hiện và loại/sửa bbox thừa |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `"python": "3.13.15", "ultralytics": "8.4.145", "torch": "2.11.0+cu128", "lap": "0.5.13",` |
| weights / hai tracker | `"weights": "yolo26n.pt"; ByteTrack: bytetrack.yaml; BoT-SORT + ReID: /content/Day3-Lab/configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `conf=0.25 , IoU=0.7 , imgsz=960 , classes=[2, 5, 7]` |
| device | `GPU` |

| So sánh                   |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP | FP | FN | IDSW |
| ------------------------- | ----: | ----: | ----: | ----: | ----: | ----: | ----: | -: | -: | ---: |
| bạn vs gold               | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 |  0 |  0 |    0 |
| ByteTrack control vs gold | 0.742 | 0.679 | 0.816 | 0.872 | 0.883 | 0.766 | 0.852 | 85 | 46 |    2 |
| BoT-SORT + ReID vs gold   | 0.827 | 0.771 | 0.888 | 0.922 | 0.910 | 0.810 | 0.917 | 89 | 19 |    0 |
| ReID vs bạn               | 0.827 | 0.771 | 0.888 | 0.922 | 0.910 | 0.810 | 0.917 | 89 | 19 |    0 |


## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA = 0.921, thấp hơn IDF1 = 0.960 ở bản sau rework. MOTA chủ yếu phản ánh lỗi phát hiện và coverage như FP, FN, IDSW; vì vậy có thể MOTA vẫn cao dù còn lỗi nhận dạng ID. IDF1 tập trung hơn vào việc duy trì đúng danh tính giữa các frame nên nhạy hơn với lỗi association/identity.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`BoT-SORT + ReID có IDF1 = 0.910, AssA = 0.888 và IDSW = 0, tốt hơn ByteTrack với IDF1 = 0.883, AssA = 0.816 và IDSW = 2. Điều này cho thấy treatment duy trì association tốt hơn trên clip này. Tuy nhiên không thể kết luận riêng tác động nhân quả của ReID vì ByteTrack và BoT-SORT là hai tracker implementation khác nhau.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`ByteTrack có DetA = 0.679, FP = 85, FN = 46; BoT-SORT + ReID có DetA = 0.771, FP = 89, FN = 19. Treatment giảm FN và tăng DetA rõ rệt nhưng FP tăng nhẹ. Vì IDSW của treatment = 0 và IDF1/AssA cao hơn, phần lỗi còn lại có vẻ thiên về detection/coverage hơn là association.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`frame 16/116, ID 7, ReID box nhầm vào một gian hàng chứ không phải phương tiện.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Frame 139, ID 3: ReID có loose box với IoU = 0.547 so với annotation. Tuy nhiên đây chỉ là dấu hiệu model và annotation không khớp, chưa đủ để kết luận annotation sai. Cần xem trực tiếp frame 139; nếu bbox của annotation bám đúng xe thì lỗi thuộc model.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Bổ sung rõ quy tắc cho frame đầu/cuối của track, đặc biệt khi xe ra/vào khung hình; quy định rõ trường hợp xe bị che dưới/trên 25 frame và trường hợp rời khung rồi quay lại. Khi gán nhãn, sẽ xem kỹ các frame đầu/cuối và các đoạn crossing/occlusion, sau đó tự kiểm riêng ID, đầu/cuối track và frame giữa trước khi kiểm chéo.`

## 7. Tệp đã nộp

- [ ] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền
- [ ] `outputs/eval_vs_gold.json`
- [ ] `outputs/model_bytetrack_clip_01.txt`
- [ ] `outputs/model_reid_clip_01.txt`
- [ ] `outputs/model_run_config.json`
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [ ] `reports/REPORT.md` (file này)
