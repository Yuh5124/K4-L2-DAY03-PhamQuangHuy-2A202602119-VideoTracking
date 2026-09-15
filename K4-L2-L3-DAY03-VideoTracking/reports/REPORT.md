# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Phạm Quang Huy`
Ngày: `15/9/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | 	CVAT Community (local, http://localhost:8080) |
| Thời gian gán `clip_02` (warm-up) | `60` phút |
| Thời gian gán `clip_01` | `180` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `33` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Xác định frame bắt đầu của track 5. Chiếc xe này xuất hiện dần từ vùng mờ, không có frame nào rõ ràng là "bắt đầu"`
2. `Một vật tĩnh cỡ xe ở giữa khung hình. Quanh toạ độ (544, 243) có vật kích thước ~100×58 px trông giống xe nhưng không di chuyển `
3. `Đoạn khó của reference track 6`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `có track nào nhấp nháy hay đổi số không? Kết quả cuối cho thấy IDSW = 0, nên nhiều khả năng lượt này PASS — ghi rõ bạn đã kiểm gì`
- Lượt 2: `frame đầu/cuối của từng track: Lượt này lẽ ra phải bắt được lỗi track 5 bắt đầu sớm 4 frame và kết thúc sớm 4 frame. Không bắt được vì tôi kiểm "bbox có treo không" chứ chưa kiểm "bbox có bắt đầu đúng lúc không".`
- Lượt 3: `giữa hai keyframe: Lượt này lẽ ra phải bắt được 6 chỗ bbox trôi (track 5 ở frame 82, 88, 89, 90, 133; track 8 ở frame 168), nơi IoU tụt còn 0.55–0.58`

Kiểm chéo với: `làm cá nhân, không có peer reviewer. Đã hỏi Lab Coach về cách xử lý mục này`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `Không có. Bước tools/lock_pre_gold.py bị bỏ sót trước khi nhận teaching reference. Đã báo Lab` |
| Thời điểm khóa | `17h` |
| Số row / frame / track trước khi mở reference | `565 bbox · 190 frame · 8 track   ` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold |0.821 | 0.805|0.838 |0.876 | 0.972|0.944 | 0.862|12 |20 |0 |
| Sau rework | ko rework | | | | | | | | | |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): có — IDF1 0.972 · MOTA 0.944 · MOTP 0.862.

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
|Bbox treo |75–78 (4 frame) |track 5 |Track bắt đầu sớm 4 frame so với reference. Xem câu 5 mục 5: đây không phải lỗi đơn lẻ mà là cả track bị dời sớm 4 frame. Sửa: dời keyframe đầu sang frame 79, kéo dài keyframe cuối tới frame 139. |
|Bbox trôi |82, 88, 89, 90, 133 |track 5 | IoU tụt còn 0.551–0.580, tất cả nằm giữa hai keyframe cách xa nhau ở đoạn xe đổi hướng. Sửa: thêm keyframe quanh frame 88–90 và 133.|
|Bbox trôi |168 |track 8 | 	IoU 0.581. Cùng nguyên nhân. Sửa: thêm keyframe quanh frame 168.|

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt · control bytetrack.yaml · treatment configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7] (COCO car, bus, truck)` |
| device | `GPU device=0, persist=true, 190 frame` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold |0.821 |	0.805 |0.838 |0.876 |0.972 |0.944 | 0.862|12 |20 | 0|
| ByteTrack control vs gold |0.709 |0.649 |0.776 |0.846 | 0.875|0.749 | 0.823|88| 54|2 |
| BoT-SORT + ReID vs gold |0.763 |0.711 |0.820 |0.872 | 0.900|0.792 |0.860 | 91|26 | 2|
| ReID vs bạn |0.730|0.674|0.793|0.845|0.901|0.791|0.825|95|22|1

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`Trong bản nhãn của tôi, MOTA (0.944) thấp hơn IDF1 (0.972) — ngược với trường hợp kinh điển mà câu hỏi nêu.

Nguyên nhân là IDSW = 0. Không có lần đổi ID nào, nên toàn bộ điểm trừ của MOTA đến từ FP và FN:

MOTA = 1 − (FP + FN + IDSW) / GT = 1 − (12 + 20 + 0) / 573 = 0.9442
IDF1 = 2·IDTP / (2·IDTP + IDFP + IDFN) = 2·553 / (2·553 + 12 + 20) = 0.9719

Hai chỉ số chịu đúng cùng 32 lỗi, nhưng IDF1 có IDTP nhân đôi ở mẫu số nên cùng lượng lỗi bị pha loãng hơn. Khi identity hoàn hảo, hai chỉ số đo cùng một thứ — coverage và geometry — với hai trọng số khác nhau, và IDF1 luôn cao hơn.

Về vế thứ hai, bằng chứng nằm ngay trong thí nghiệm này. ByteTrack control tách 3 track của reference: track 4 (95 frame) chia cho ID [15, 14], track 5 (60 frame) chia cho [32, 23], track 7 (85 frame) chia cho [52, 71]. Nhưng MOTA chỉ ghi nhận 2 điểm ID switch trên 573 bbox — đóng góp 0.35% vào điểm trừ. Trong khi AssA tụt xuống 0.776 và IDF1 còn 0.875.

Lý do: MOTA đếm ID switch theo sự kiện — mỗi lần nhảy ID tính đúng một lỗi, bất kể sau đó bao nhiêu frame mang sai danh tính. IDF1 và AssA đếm theo thời lượng — chúng ghép track với track trên toàn video, nên một cú switch giữa track 95 frame làm hỏng gần nửa quãng đời và bị phạt tương ứng.

Hệ quả cho bài toán xe tự lái: tầng Prediction cần vận tốc, mà vận tốc chỉ tính được khi biết vị trí của cùng một xe ở hai thời điểm. MOTA cao không bảo chứng điều đó; IDF1 và AssA mới là chỉ số phải nhìn.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`	ByteTrack	ReID	Chênh
IDF1	0.875	0.900	+0.025
AssA	0.776	0.820	+0.044
IDSW	2	2	0
HOTA	0.709	0.763	+0.054

Treatment tốt hơn ở mọi chỉ số identity, nhưng theo cách cần đọc kỹ: AssA tăng 0.044 trong khi IDSW không đổi. ReID không làm giảm số lần đổi ID — nó cải thiện bằng cách khác.

Hai điểm switch cũng nằm ở chỗ khác nhau:

	Frame	Track reference	Chuyển ID
ByteTrack	59	4	14 → 15
ByteTrack	94	5	23 → 32
ReID	87	5	17 → 18
ReID	113	6	24 → 31

Một nhận xét về bản chất các "switch" này: nhìn trường frames_each trong JSON, cả bốn đều là ID sơ sinh tồn tại 1–2 frame rồi nhường chỗ cho ID chính, không phải hoán đổi giữa chừng. Ví dụ ByteTrack track 4: ID 14 chỉ khớp 2 frame, ID 15 khớp 90 frame. ReID track 5: ID 17 khớp 1 frame, ID 18 khớp 51 frame. Đây là hiện tượng track khởi tạo chưa ổn định, không phải hai xe bị tráo danh tính.

Frame sequence dẫn chứng — reference track 8, frame 136–170:

ByteTrack gán ID 54, chạy frame 136–167, sinh 32 bbox, nhưng chỉ 20/33 frame (61%) khớp reference. Tám bbox trong đoạn 152–167 có IoU chỉ 0.525–0.579.
BoT-SORT + ReID gán ID 34, chạy frame 136–170, sinh 35 bbox, và reference track 8 không xuất hiện trong danh sách thiếu đoạn của ReID.

Nghĩa là trên cùng một chiếc xe, cùng một detector, treatment theo được dài hơn 3 frame và bám sát hơn hẳn. Riêng track này đóng góp phần lớn vào việc FN giảm từ 54 xuống 26.

Cơ chế hợp lý: BoT-SORT giữ track sống qua các đoạn detection yếu và hiệu chỉnh box tốt hơn. Con số ủng hộ: treatment xuất 638 bbox so với 607 của control, cùng 16 track — tức mỗi track kéo dài hơn chứ không sinh thêm track.

Lưu ý phương pháp: đây là system comparison, không phải causal ablation của ReID. ByteTrack và BoT-SORT là hai implementation association khác nhau; BoT-SORT thay đổi cả cách xử lý Kalman và quản lý vòng đời track chứ không chỉ thêm appearance cue. Chênh lệch 0.044 ở AssA không chứng minh riêng ReID gây ra thay đổi. Muốn cô lập, phải chạy BoT-SORT cùng cấu hình ở with_reid: false rồi so với with_reid: true — điều lab này không yêu cầu.

Kết luận đúng phạm vi: BoT-SORT + ReID treatment cho identity metrics tốt hơn ByteTrack control trên clip này.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`	ByteTrack	ReID	Chênh
DetA	0.649	0.711	+0.062
FN	54	26	−28 (−52%)
FP	88	91	+3
LocA	0.846	0.872	+0.026

Detector input hoàn toàn giống nhau giữa hai run — cùng yolo26n.pt, conf=0.25, iou=0.70, imgsz=960, classes=[2,5,7], cùng thứ tự frame, cùng persist=true, khóa trong outputs/model_run_config.json.

Vậy mà DetA tăng 0.062, FN giảm hơn một nửa, và LocA cũng tăng 0.026.

Ba con số này không thể thay đổi vì detector, bởi detector là hằng số. Chúng thay đổi vì quản lý track: BoT-SORT giữ track sống lâu hơn qua các đoạn detection yếu (giải thích FN và DetA) và hiệu chỉnh box tốt hơn (giải thích LocA). Đây là bài học về cách đọc chỉ số — chữ "Det" trong DetA không chỉ đo detector, nó đo phần detection của cả hệ thống tracking, và phần đó chịu ảnh hưởng của bước association.

Lỗi còn lại chủ yếu là detector, không phải association. Ba bằng chứng:

FP gần như đứng yên (88 → 91) và vẫn rất cao — 91 bbox thừa trên tổng 573 của reference. Association chỉ nối những box detector đưa ra, nó không loại bỏ được box sai.
Các track ma giống hệt nhau ở cả hai tracker. ByteTrack ID 10 (frame 17–116) và ReID ID 7 (frame 16–116) là cùng một vật thể; ByteTrack ID 41 và ReID ID 27 cùng chạy frame 106–121. Cùng một ảo ảnh xuất hiện ở hai tracker khác nhau ⇒ nguồn gốc nằm ở detector.
Hình học lệch. ByteTrack có 30 bbox với IoU 0.51–0.59; ReID giảm còn 9 nhưng vẫn còn. Phần dư này là detector khoanh chưa khít.

Để giảm FP phải can thiệp ở tầng detector — fine-tune trên dữ liệu domain hoặc nâng ngưỡng conf — không phải đổi tracker.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Track ma ở tâm ảnh: ReID ID 7 (frame 16–116) và ByteTrack ID 10 (frame 17–116).

Phân tích toạ độ trực tiếp từ outputs/model_reid_clip_01.txt:

Thuộc tính	Giá trị
Tâm bbox	(544, 243)
Kích thước	96×53 đến 106×60 px
Biên độ dịch chuyển của tâm qua toàn bộ 100 frame	4 px ngang, 6 px dọc
Thời lượng	43 bbox trải từ frame 16 đến 116 (≈ 8 giây @ 12.5 fps)

Một chiếc xe đang lưu thông, dù chậm đến mấy, cũng không thể chỉ nhích 4 pixel trong 8 giây. Đây là vật tĩnh cỡ xe bị detector bắt nhầm — đúng dạng cảnh báo trong notebook: "bbox tồn tại hàng chục frame mà không nhúc nhích — nhìn track đứng im là nhận ra ngay. Đó là model sai, không phải bạn thiếu."

Tôi không gán vật này, và teaching reference cũng không — bằng chứng là track ma này không khớp track reference nào trong cả hai file eval. Cả hai tracker sai ở đúng một chỗ.

Hai ví dụ phụ củng cố kết luận:

ReID ID 27 (frame 106–121) và ID 38 (frame 158–178): cùng tâm (10, 291), kích thước 20×20 px, sát mép trái ảnh, tâm dịch 0–2 px. Cùng một vật tĩnh bé bị bắt hai lần ở hai khoảng thời gian.
ReID ID 28 (frame 107): bbox 74×20 px — tỉ lệ cạnh gần 4:1, dẹt bất thường, không phải hình dạng xe bốn bánh nhìn từ camera CCTV.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Track 5 của tôi bị dời sớm 4 frame trên toàn bộ quãng đời.

Phát hiện này không nhìn thấy được từ một file đơn lẻ — phải ghép bốn nguồn:

Nguồn	Nội dung
eval_vs_gold.json	Track 5 của tôi có bbox ở frame 75–78 trước khi reference track 5 xuất hiện
eval_reid_vs_me.json	Model ID 18 còn bbox ở frame 136–139 sau khi track 5 của tôi đã kết thúc
eval_reid_vs_gold.json	Model ID 18 không thừa frame nào so với reference
model_reid_clip_01.txt	Model ID 18 chạy từ frame 87 đến 139

Ghép lại: reference track 5 sống khoảng frame 79–139; track 5 của tôi sống khoảng frame 75–135. Độ dài gần như bằng nhau — 61 frame của tôi so với 60 frame của reference — nhưng toàn bộ bị dời sớm 4 frame ở cả hai đầu.

Đây là loại lỗi mà một mình eval_vs_gold.json không chỉ ra được: nó chỉ báo "bbox treo 4 frame ở đầu" và không nói gì về đuôi, vì 4 frame thiếu ở cuối chưa đủ kích hoạt cảnh báo thiếu đoạn. Chính việc model duy trì track đến frame 139 — và việc so với reference cho thấy model không sai ở đó — mới lộ ra rằng đuôi track của tôi mới là cái ngắn.

Ý nghĩa: đây không phải lỗi vẽ ẩu mà là lỗi hệ thống về ngưỡng. Tôi nhận diện xe sớm hơn reference 4 frame ở đầu và mất dấu sớm 4 frame ở cuối — nhất quán ở cả hai đầu, tức tiêu chí "xe xác định được" của tôi lệch so với reference một lượng cố định. Luật cần định lượng lại, không phải nhắc nhau cẩn thận hơn.

Một trường hợp tương tự, yếu hơn: model ID 31 còn bbox ở frame 153–156 sau khi track 7 của tôi kết thúc, và cũng không thừa so với reference — cùng dạng lỗi kết thúc sớm 4 frame.

Cách sửa đã viết vào GUIDELINE_MINI.md: thay tiêu chí cảm tính bằng tiêu chí đo được (nhìn thấy thân xe và ít nhất một đặc trưng xác định, cạnh ngắn bbox ≥ 12 px), cộng thêm bước bắt buộc tua kiểm frame trước frame đầu và frame sau frame cuối của mỗi track.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Sửa gì trong GUIDELINE_MINI.md:

Định lượng ngưỡng bắt đầu và kết thúc track. Đây là nguồn gốc của lỗi lớn nhất trong nhãn của tôi — track 5 lệch 4 frame ở cả hai đầu. Luật cũ ("frame đầu tiên xác định được là xe bốn bánh") phụ thuộc hoàn toàn vào cảm nhận, nên hai người khác nhau sẽ cho hai kết quả lệch nhau một cách hệ thống. Luật mới: bắt đầu track từ frame đầu tiên nhìn thấy đủ thân xe và ít nhất một đặc trưng xác định (bánh, đèn, kính); cạnh ngắn bbox ≥ 12 px.
Thêm luật mật độ keyframe. Cả 6 chỗ bbox trôi đều nằm giữa hai keyframe cách xa nhau ở đoạn xe đổi hướng. Luật mới: tối đa 10 frame giữa hai keyframe khi xe rẽ, đổi làn, phanh, tăng tốc, hoặc khi kích thước bbox đổi quá 20%; 20–30 frame khi xe đi thẳng đều.
Thêm luật loại vật tĩnh. Cả hai model đều tạo track ma từ vật đứng im cỡ xe. Người gán nhãn cũng có thể mắc lỗi này. Luật mới: nếu tâm bbox dịch dưới 10 px qua 20 frame liên tiếp, dừng lại kiểm — đó có thể là vật tĩnh, không phải xe đỗ.

Đổi gì trong quy trình làm việc:

Đưa lock_pre_gold.py thành bước tự động ngay sau export. Lần này tôi bỏ sót nó, và đó là lỗi quy trình chứ không phải lỗi hiểu bài. Nếu gán 10 clip, sẽ viết một script chạy tuần tự export → check_mot_labels → lock_pre_gold để không có chỗ cho việc quên.
Đổi cách làm lượt 2 của tự kiểm. Hiện tôi kiểm "bbox có treo sau khi xe rời khung không", nên bắt được đuôi thừa nhưng không bắt được đuôi thiếu và đầu thừa. Phải kiểm cả hai chiều: tua về frame trước frame đầu và frame sau frame cuối của từng track, xác nhận xe thật sự chưa xuất hiện / đã rời khung.
Dùng model làm bước rà soát cuối, không phải bước tạo nhãn. Giá trị lớn nhất của model trong thí nghiệm này là thu hẹp vùng cần kiểm: bảng bất đồng chỉ ra 8 frame đáng soi (52, 54, 106, 107, 111, 118, 139, 170) thay vì phải rà cả 190 frame. Nhưng phải làm đúng thứ tự — gán độc lập trước, khóa bằng chứng, rồi mới đối chiếu.
Nhìn track ma như tín hiệu về dữ liệu, không chỉ về model. Vật tĩnh ở toạ độ (544, 243) khiến cả hai tracker sai suốt 100 frame. Nếu gán 10 clip trong cùng bối cảnh camera, vật đó sẽ xuất hiện lại. Đáng ghi vào guideline ngay từ clip đầu tiên để người sau không phải tự phát hiện lại.`

## 7. Tệp đã nộp

- [X] `annotations/clip_01/gt.txt`
- [X] `annotations/clip_02/gt.txt`
- [X] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [X] `GUIDELINE_MINI.md` đã điền
- [X] `outputs/eval_vs_gold.json`
- [X] `outputs/model_bytetrack_clip_01.txt`
- [X] `outputs/model_reid_clip_01.txt`
- [X] `outputs/model_run_config.json`
- [X] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [X] `reports/review_partner.md`
- [X] `reports/REPORT.md` (file này)
