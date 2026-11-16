# Ghi chú cá nhân — 323

> File ghi chú cá nhân của thành viên có mã số **323**. Ghi chép trung thực quá trình làm bài qua các pha (ngắn gọn, khoảng một trang).

---

## 1. Pha 1: ba ca khó tôi tìm ra trong 40 ảnh

| Ảnh | v0 không trả lời được gì | Quyết định ban đầu của tôi |
|---|---|---|
| **`img_018.jpg`** | Guideline v0 chỉ ghi `person: Người` và yêu cầu "gán box cho mọi người và phương tiện", nhưng **không phân biệt người thật với hình ảnh người phẳng in trên biển quảng cáo (billboard/poster)** bên đường. Hai người gán sẽ bất đồng: một người thấy mặt người thì vẽ box, một người coi là biển báo nên bỏ qua. | **Không gán nhãn `person`** cho hình người trên billboard. Lý do: Mục tiêu mô hình là nhận diện đối tượng vật lý tham gia giao thông để hỗ trợ xe tự lái/camera an toàn; nếu gán box cho ảnh poster sẽ gây ra lỗi báo động giả (False Positive), khiến xe tự lái có thể phanh khẩn cấp không đáng có. |
| **`img_005.jpg`** | Guideline v0 ghi `person: Người đi bộ hoặc người ngồi trên xe` nhưng hoàn toàn bỏ ngỏ quy tắc: **(1)** Gán 1 box gộp chung cả người và xe hay tách thành 2 box riêng? **(2)** Ranh giới box của người vẽ ôm sát phần nhìn thấy hay ước lượng cả phần chân bị che? **(3)** Thuộc tính `occluded_level` xác định ra sao khi người và xe che lẫn nhau? | **Tách thành 2 box độc lập**: 1 box cho `person` và 1 box cho `motorcycle`. Box của `person` ôm sát phần cơ thể thực tế nhìn thấy (từ đỉnh mũ bảo hiểm đến phần hông/đùi trên), chọn `occluded_level = 2` (bị thân xe che > 35%); box của `motorcycle` ôm trọn xe, chọn `occluded_level = 1` (bị người lái che phần yên xe ~10-35%). Tuyệt đối không gộp chung vào 1 box. |
| **`img_020.jpg`** | Guideline v0 **không có bất kỳ quy tắc nào về hiện tượng phản xạ quang học (ảnh phản chiếu qua gương chiếu hậu, kính tòa nhà, vũng nước mưa)**. Trong ảnh có hình ảnh xe và cảnh vật phản chiếu trong gương xe ô tô, dẫn đến sự nhập nhằng giữa vật thể thực và ảnh ảo. | **Không gán box** cho đối tượng phản chiếu trong gương. Lý do: Đối tượng trong gương là ảnh ảo 2D trên bề mặt quang học, không phải là chướng ngại vật vật lý đang hiện diện trên mặt đường cần tránh va chạm. |

*(Một ca dự phòng khác được ghi nhận thêm: **`img_004.jpg`** (xe tuk-tuk 3 bánh) và **`img_006.jpg`** (xe xích lô) – taxonomy của v0 chỉ có `car`, `truck`, `van`, `motorcycle`, không có lớp cho xe 3 bánh hay xe đẩy thô sơ, nên quyết định ban đầu là tick `can_review = true` để đưa ra họp nhóm).*

---

## 2. Pha 2: chỗ tôi bất đồng với nhóm
- Chỗ nào bản của tôi khác bản của bạn? Cuối cùng nhóm chốt gì, tôi đồng ý / chưa đồng ý vì sao?
*(Sẽ hoàn thiện sau cuộc họp thống nhất Guideline v1.0 của nhóm)*

---

## 3. Pha 3: ảnh tôi đã gán
- Số ảnh được giao: … Số box đã vẽ: … Số lần tôi tick `can_review`: …
- Ca nào tôi lưỡng lự nhất khi gán (ảnh, vì sao)?
*(Sẽ hoàn thiện sau khi hoàn thành gán nhãn độc lập trên CVAT)*

---

## 4. Pha 4–5: tôi lệch ở đâu
- Trong `calibration_candidates.csv`, các ca có tên tôi: tôi lệch vì guideline mơ hồ hay vì tôi chưa nắm quy tắc?
- Quyết định calibration nào tôi thấy quan trọng nhất?
*(Sẽ hoàn thiện sau phiên Calibration của nhóm)*

---

## 5. Dùng AI
- Công cụ nào, cho việc gì, bạn kiểm tra kết quả thế nào? (Ghi "không dùng" nếu không dùng.)
  - Sử dụng trợ lý AI để tra cứu các quy chuẩn gán nhãn quốc tế (KITTI, COCO, Cityscapes) nhằm chuyển đổi định tính của thuộc tính `occluded_level` (mức che khuất) sang định lượng theo ngưỡng tỷ lệ diện tích che chắn (<10%, 10-35%, >35-80%, >80%).
  - Kiểm tra kết quả: Tự rà soát trực quan lại trên toàn bộ 40 ảnh mẫu của bài lab để đảm bảo các ngưỡng định lượng áp dụng thực tế được và phù hợp với độ phân giải của tập dữ liệu.

---

## 6. Bài học
- Một điều tôi sẽ làm khác khi viết guideline lần sau:
  - Cần quy định rõ ràng ngay từ đầu các ca biên (edge cases) liên quan đến hình ảnh phi thực thể (biển quảng cáo, tượng điêu khắc, manocanh, gương phản chiếu) và quy tắc tách box khi người ngồi trên phương tiện để tránh mất thời gian tranh cãi khi đo độ đồng thuận (IAA).

