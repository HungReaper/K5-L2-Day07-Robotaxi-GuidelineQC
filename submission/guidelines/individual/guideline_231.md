# Guideline gán nhãn cá nhân — v0

- **Người viết:** Trịnh Đức Hùng — `2A202603177`
- **Phiên bản:** v0 — bản đề xuất cá nhân, chưa được nhóm thống nhất
- **Loại nhãn:** bounding box trên ảnh đường phố
- **Công cụ / định dạng:** CVAT / COCO 1.0
- **Lưu ý:** Các tình huống ở mục 4 cần đối chiếu với ảnh; chưa gắn mã ảnh để người viết tự xác nhận sau khi xem.

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện người và phương tiện trong cảnh đường phố, ví dụ cho camera giao thông hoặc xe tự lái. Bỏ sót người, nhận nhầm hình in/phản chiếu thành người hoặc xe, và phân loại sai phương tiện đều có thể khiến mô hình nhận diện sai chủ thể giao thông. Guideline này là đề xuất cá nhân, chưa thay thế quyết định thống nhất của nhóm.

## 2. Lớp và thuộc tính

Mỗi người hoặc phương tiện được gán một lớp theo taxonomy hiện tại. `ignore_region` là vùng bỏ qua, không phải lớp của một vật thể. Bảng dưới phân biệt lớp có sẵn ở taxonomy v0 với lớp chỉ là đề xuất; các lớp đề xuất chưa dùng trong CVAT trước khi nhóm thống nhất và cập nhật taxonomy.

### 2.1 Lớp

| Lớp             | Trạng thái                                  | Định nghĩa đề xuất                                                                      | Ví dụ                                          | Phản ví dụ / ranh giới                                                                                                                                                                             |
| --------------- | ------------------------------------------- | --------------------------------------------------------------------------------------- | ---------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `car`           | Có trong taxonomy v0                        | Ô tô con chở người, gồm sedan, hatchback và SUV.                                        | Ô tô con nhìn từ trước, sau hoặc bên.          | Không dùng cho xe tải hoặc van nhận diện được.                                                                                                                                                     |
| `truck`         | Có trong taxonomy v0                        | Xe tải chở hàng, có khoang hoặc sàn chở hàng riêng.                                     | Xe tải thùng, xe tải sàn.                      | Không dùng cho ô tô con; xe bán tải cần review nếu khó phân lớp.                                                                                                                                   |
| `van`           | Có trong taxonomy v0                        | Xe thân kín dạng hộp dùng chở người hoặc hàng.                                          | Van giao hàng thân kín.                        | Không phân biệt chỉ bằng kích thước; nếu ranh giới với `truck` không rõ, bật `can_review`.                                                                                                         |
| `motorcycle`    | Có trong taxonomy v0                        | Phương tiện hai bánh có động cơ, gồm xe máy và xe tay ga.                               | Xe máy đang chạy hoặc đỗ.                      | Không dùng cho xe đạp hoặc phương tiện ba bánh.                                                                                                                                                    |
| `person`        | Có trong taxonomy v0                        | Người thật trong cảnh, đi bộ, đứng, lái xe hoặc ngồi trên xe.                           | Người đi bộ, người lái, hành khách.            | Không dùng cho tượng, ma-nơ-canh, hình người in hoặc ảnh phản chiếu.                                                                                                                               |
| `three_wheeler` | Đề xuất bổ sung — chưa có trong taxonomy v0 | Phương tiện có ba bánh, không thuộc lớp xe khác.                                        | Xe ba bánh, tuk-tuk.                           | Chưa dùng để gán nhãn cho tới khi nhóm chốt và cập nhật taxonomy.                                                                                                                                  |
| `other_vehicle` | Đề xuất bổ sung — chưa có trong taxonomy v0 | Phương tiện thật thuộc phạm vi nhưng không phù hợp lớp xe cụ thể nào.                   | Xe buýt, xe đạp, xe do động vật kéo (đề xuất). | Chưa dùng để gán nhãn cho tới khi nhóm chốt và cập nhật taxonomy; không gán nhãn con vật kéo.                                                                                                      |
| `ignore_region` | Có trong taxonomy v0                        | Box vùng bỏ qua cho đối tượng quá nhỏ theo ngưỡng lab; không phải box của từng vật thể. | Vùng có đối tượng mục tiêu cao dưới 20 px.     | Không phủ lên đối tượng đủ điều kiện gán riêng; cách gom các vật nhỏ gần nhau cần nhóm chốt. Không dùng thay cho lớp chưa rõ; trường hợp khác như đám đông/ảnh mờ chỉ áp dụng nếu nhóm thống nhất. |

**Lớp ngoài taxonomy:** Taxonomy v0 hiện chưa có `three_wheeler` hay `other_vehicle`. Xe buýt, xe đạp, xe kéo và loại phương tiện khác là các đề xuất cần nhóm xác nhận; không ép chúng vào lớp gần giống hoặc thêm nhãn CVAT trước khi taxonomy được cập nhật. Nếu đối tượng mơ hồ, ghi review thay vì tự tạo lớp.

### 2.2 Thuộc tính

| Thuộc tính       | Giá trị            | Quy tắc áp dụng                                                                                                                                                                                                                          |
| ---------------- | ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `occluded_level` | `0`, `1`, `2`, `3` | Ước lượng tỷ lệ diện tích vật bị vật khác che: `0` = không bị che; `1` = trên 0% đến 30% (kể cả 30%); `2` = trên 30% đến 70% (kể cả 70%); `3` = trên 70% nhưng vẫn nhận diện được lớp. Nếu không xác định được lớp thì không đoán, ghi review. |
| `truncated`      | `no`, `yes`        | `yes` khi vật bị biên ảnh cắt và có dấu hiệu phần vật tiếp tục ngoài ảnh; còn lại `no`.                                                                                                                                                  |
| `can_review`     | tick / không       | Tick khi còn nghi ngờ về lớp, ranh giới box, mức che hoặc quy tắc; ghi lý do ngắn để người review tìm được.                                                                                                                              |

Ngưỡng che là quy ước đề xuất để nhóm xem xét. Ước lượng phần bị che so với toàn vật dự kiến; không đoán phần khuất khi thiếu dấu hiệu.

## 3. Quy tắc vẽ box

1. **Ôm sát:** Box theo ranh giới phần vật nhìn thấy; không gộp nhiều người/xe, không kéo box qua vật che hoặc ra ngoài khung ảnh.
2. **Che khuất:** Gán riêng từng người/xe nếu nhận diện được; box chỉ bao phần nhìn thấy và chọn `occluded_level` theo bảng trên. Nếu không nhận diện chắc lớp, bật `can_review`, không đoán.
3. **Cắt mép:** Box theo phần còn trong ảnh; đặt `truncated=yes` khi vật bị biên ảnh cắt. Không suy đoán phần nằm ngoài khung.
4. **Phản chiếu, ảnh in, tượng/ma-nơ-canh, đồ chơi:** Không gán chúng thành người/xe thật. Không tạo box riêng cho phản chiếu trên kính, gương hoặc mặt đường. Nếu không phân biệt được ảnh phản chiếu với vật thật, ghi review.
5. **Người trên xe:** Nếu nhìn thấy, tạo box riêng cho người và cho xe. Không gộp người lái/hành khách vào box xe; người bị che một phần vẫn gán nếu nhận diện được.
6. **Vật nhỏ:** Theo ngưỡng lab, vật cao dưới `20 px` không gán box riêng; dùng box `ignore_region` để đánh dấu vùng bỏ qua, không tạo box cho từng vật nhỏ. Vật cao từ `20 px` trở lên gán riêng nếu nhận diện được.
7. **Đám đông, nhòe hoặc tối:** Tách từng người/xe khi nhìn thấy ranh giới; không gộp cả đám thành một đối tượng. Chỉ dùng `ignore_region` cho vật quá nhỏ theo ngưỡng lab; chỉ mở rộng dùng cho đám đông/ảnh mờ nếu nhóm thống nhất. Nếu đã vẽ box nhưng còn nghi ngờ, tick `can_review`; nếu chưa thể vẽ box cho đối tượng, ghi ảnh, người gán, vùng và lý do vào `submission/qc/image_issues.csv` theo các cột `image,annotator,region,note` (sau khi khóa nhãn).
8. **Ngoài taxonomy:** Không gán nhầm sang lớp gần nhất; ghi tình huống để nhóm chốt lớp hoặc phạm vi trước khi khóa taxonomy.

## 4. Ca khó cần đối chiếu ảnh

**Bảng dưới là các tình huống cần rà, chưa khẳng định ảnh nào chứa chúng. Tự điền mã `img_0xx` sau khi xem và xác nhận ảnh; nội dung “chưa chốt” cần đưa ra họp nhóm.**

| Ảnh                         | Tình huống                                            | Quyết định dự thảo                                                                                          | Lý do / trạng thái                                                                      |
| --------------------------- | ----------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `img_004`                   | Xe buýt, xe đạp, xe ba bánh/tuk-tuk hoặc xe kéo       | Chưa chốt lớp; đề xuất nhóm quyết định trong taxonomy. Không mặc định dùng `other_vehicle` cho cả bốn loại. | Taxonomy v0 chưa có các lớp này; `three_wheeler` và `other_vehicle` đều mới là đề xuất. |
| `img_017`, `img_033`, `038` | Tượng, ma-nơ-canh, đồ chơi hoặc hình người/xe in      | Dự thảo: không gán là người/xe thật.                                                                        | Tránh dạy mô hình nhầm vật trưng bày/hình ảnh với chủ thể thật.                         |
| `img_020`                   | Người/xe trong gương, kính hoặc phản chiếu trên đường | Dự thảo: không tạo box cho ảnh phản chiếu; chỉ gán vật thể thật nhìn thấy trực tiếp.                        | Tránh gán ảnh phản chiếu thành đối tượng độc lập.                                       |
| `img_021`                   | Đối tượng nhỏ, đám đông, ảnh nhòe/tối                 | Dưới `20 px`: `ignore_region`; còn lại chỉ gán riêng khi đủ rõ, nếu không ghi review.                       | Không bịa số lượng hoặc ranh giới không quan sát được.                                  |

## 5. QC và phiên bản

### QC đề xuất

- Soát đủ ảnh được giao; kiểm tra box ôm sát phần thấy được, không vượt khung, lớp/thuộc tính hợp lệ; rà box thừa trên phản chiếu/hình in.
- Theo ngưỡng lab: box cao `<50 px` dùng IoU `0.70`; box cao `≥50 px` dùng `0.85`.
- Ngưỡng IAA tham chiếu: IoU trung bình `≥0.85`, cùng lớp `≥95%`. Tỷ lệ ảnh có lỗi tối đa `2%` mới xem xét Release; báo đủ tử số/mẫu số. Không ghi kết quả QC trước khi thực sự chạy/kiểm tra.
- Ca chưa rõ ghi `can_review` hoặc vấn đề cấp ảnh; quyết định cuối thuộc buổi họp nhóm. Không tự sửa taxonomy chung trong giai đoạn guideline cá nhân.

### Lịch sử phiên bản

| Phiên bản | Thay đổi                                                                                                   | Lý do / trạng thái                                                                         |
| --------- | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| v0        | Bản cá nhân đề xuất, bổ sung định nghĩa lớp, ngưỡng thuộc tính, quy tắc box và danh sách ca cần đối chiếu. | Chưa được nhóm thống nhất; mã ảnh và ca cụ thể cần người viết tự xác nhận sau khi xem ảnh. |
