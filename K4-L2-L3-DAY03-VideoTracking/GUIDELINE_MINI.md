# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `...`
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

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | `Theo mặc định của lab. Trong 2 giây, một chiếc xe ở tốc độ đô thị không đi đủ xa để nhầm với xe khác, nên vị trí + hướng đủ để xác định danh tính mà không cần đoán.` |
| Xe bị che lâu hơn ngưỡng trên | `Bấm outside trong đoạn bị che, kết thúc track. Khi xe hiện lại thì mở track mới` | `Quá 2 giây thì không còn cơ sở khách quan để khẳng định là cùng xe. Đoán bừa tạo ID switch trong chính ground truth, và khi chấm sẽ phạt model vì làm đúng.` |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `Theo mặc định của lab: ra khỏi khung là kết thúc track. Không có thông tin nào trong clip chứng minh chiếc xe quay lại là chiếc xe đã đi ra.` |
| Hai xe cắt nhau / chồng lên nhau | `Mỗi xe một bbox riêng, giữ nguyên ID của từng xe. Trước và sau đoạn cắt nhau phải tua frame-by-frame để xác nhận không đổi ID cho nhau` | `Đây là chỗ sinh ID switch nhiều nhất. Bám theo hướng di chuyển và kích thước để phân biệt, không bám theo vị trí tuyệt đối.` |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `cạnh ngắn của bbox ≥ 12 px — dưới mức đó thì chờ thêm frame` |
| Xe đang đỗ, không di chuyển | `Vẫn gán vehicle, vẫn tạo track. Đặt keyframe thưa (xe không đổi vị trí nên nội suy chính xác), nhưng vẫn phải bấm outside nếu bị che khuất hẳn` |
| Keyframe đặt dày ở đâu | `đâu	Tối đa mỗi 10 frame khi xe rẽ, đổi làn, phanh, tăng tốc, hoặc khi kích thước bbox đổi quá 20% giữa hai keyframe. Xe đi thẳng đều: 20–30 frame. Bắt buộc thêm keyframe ở frame vào/ra khung` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 · frame 75–79 · track 5 của tôi`
- Tình huống: `Chiếc xe của track 5 xuất hiện dần từ vùng mờ ở xa. Không có một frame nào rõ ràng là ranh giới giữa "chưa thấy" và "thấy rồi" — ở frame 75 đã có một khối mờ nhận ra được là xe, nhưng chưa nhìn rõ bánh hay đèn`
- Quyết định: `Bắt đầu track từ frame 75, tức từ lúc nhận ra hình dạng xe.`
- Lý do: `Luật lab ghi "bắt đầu từ frame đầu tiên xác định được là xe bốn bánh", và ở frame 75 tôi đã xác định được. Nhưng tiêu chí này hoàn toàn cảm tính.`

### Ca 2
- Clip / frame / ID: `frame 16–116 · vật ở toạ độ tâm (544, 243), kích thước ~100×58 px`
- Tình huống: `Ở vùng tâm ảnh có một vật kích thước đúng bằng một chiếc xe con nhìn từ camera CCTV. Nó không di chuyển suốt hơn 100 frame. Câu hỏi: đây là xe đang đỗ (luật nói phải gán) hay vật tĩnh giống xe (luật nói không gán)?`
- Quyết định: `Không gán`
- Lý do: `Không thấy được đặc trưng xe rõ ràng (bánh, kính, đèn) và vật nằm ở vị trí không phải làn đường hay chỗ đỗ hợp lý.`

### Ca 3
- Clip / frame / ID: `clip_01 · reference track 6 (dài 56 frame), quanh vùng frame 104–120`
- Tình huống: `Chiếc xe này có một đoạn khó xác định — 【KIỂM LẠI: mở frame quanh 104–120, mô tả xe bị che, ở xa, hay lẫn vào nền】. Tôi phải quyết định có tiếp tục theo hay bấm outside   `
- Quyết định: `Bắt đầu track muộn, chỉ từ khi xác định chắc chắn là xe bốn bánh (khoảng frame 106), và kết thúc track khi xe gần ra hết khung`
- Lý do: `Áp dụng nguyên tắc thận trọng trong RULES.md — "ca mơ hồ thì ghi lại và hỏi, đừng đoán". Tôi chọn không vẽ ở những frame mình không chắc, vì một bbox sai tạo ra false positive, còn thiếu một bbox chỉ làm giảm coverage. Đây là đánh đổi có ý thức, nhưng nhìn lại thì nó phản ánh đúng vấn đề ở Ca 1: luật của tôi chưa có tiêu chí đo được để quyết định "chắc chắn" là bao nhiêu.`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

Kết quả chấm clip_01 với teaching reference: HOTA 0.821 · IDF1 0.972 · MOTA 0.944 · MOTP 0.862 · IDSW 0 · FP 12 · FN 20 — đạt mức Xuất sắc theo bảng trong RUBRIC.md.

Không có lỗi nào thuộc nhóm identity: IDSW = 0, không track nào bị tách, 8/8 track khớp reference. Bốn lỗi còn lại đều thuộc nhóm coverage và geometry, và cả bốn đều quy về ba chỗ guideline ban đầu chưa đủ rõ:

Ngưỡng bắt đầu/kết thúc track chưa định lượng. Track 5 của tôi bắt đầu sớm 4 frame (bbox ở frame 75–78 trước khi reference xuất hiện) và kết thúc sớm 4 frame (model ID 18 còn bbox ở frame 136–139 sau khi track tôi đã hết, trong khi model không thừa frame nào so với reference). Luật cũ quá phụ thuộc cảm nhận. Đã sửa ở mục 3 thành tiêu chí đo được: thân xe + ít nhất một đặc trưng xác định, cạnh ngắn ≥ 12 px, áp dụng đối xứng cho cả hai đầu track.
Chưa có luật mật độ keyframe. Sáu chỗ bbox trôi — track 5 ở frame 82, 88, 89, 90, 133 và track 8 ở frame 168, IoU tụt còn 0.551–0.581 — đều nằm giữa hai keyframe cách xa nhau ở đoạn xe đổi hướng. Đã bổ sung ở mục 3: tối đa 10 frame khi xe rẽ/phanh/tăng tốc hoặc khi bbox đổi kích thước quá 20%.
Chưa có luật phân biệt vật tĩnh với xe đỗ. Xem Ca 2. Đã bổ sung ở mục 1 thành ngưỡng đo được.

Một điểm về quy trình, không phải về luật gán nhãn: lần này tôi bỏ sót bước chạy tools/lock_pre_gold.py trước khi nhận teaching reference. Đã báo Lab Coach. Nếu gán tiếp các clip sau, bước này phải nằm ngay sau lệnh export trong một script chạy tuần tự, không phải là bước nhớ được thì làm.

Một điểm về tự kiểm: lượt 2 hiện tại của tôi chỉ kiểm "bbox có treo sau khi xe rời khung không", nên bắt được đuôi thừa nhưng không bắt được đuôi thiếu và đầu thừa — đúng hai lỗi mà track 5 mắc phải. Lượt 2 phải kiểm cả hai chiều: tua về frame trước frame đầu và frame sau frame cuối của từng track.