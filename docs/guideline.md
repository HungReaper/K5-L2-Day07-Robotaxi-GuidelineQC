# TRAFFIC OBJECT ANNOTATION GUIDELINE
**Version:** 1.1  
**Project:** Traffic Object Detection  
**Mục đích:** Thống nhất quy tắc gán nhãn đối tượng giao thông, thuộc tính và cách xử lý các trường hợp khó.

---

# 1. DANH SÁCH LỚP (CLASS LIST)

Dự án sử dụng 8 lớp đối tượng:

| STT | Class | Định nghĩa |
|---|---|---|
| 1 | `car` | Ô tô con, xe du lịch cá nhân. |
| 2 | `truck` | Xe tải, phương tiện chuyên chở hàng hóa. |
| 3 | `van` | Xe van, xe tải nhẹ hoặc xe chở hàng có khoang kín. |
| 4 | `bus` | Xe buýt, xe khách có sức chứa nhiều hành khách. |
| 5 | `motorcycle` | Xe máy, xe mô tô hai bánh có động cơ. |
| 6 | `bicycle` | Xe đạp hai bánh. |
| 7 | `three-wheeled vehicle` | Phương tiện giao thông có cấu tạo ba bánh. |
| 8 | `person` | Người đi bộ, người điều khiển hoặc ngồi trên phương tiện. |

**Lưu ý:**
- Không sử dụng class `non-motorized vehicle`.
- Không tự ý thêm class mới ngoài danh sách đã thống nhất.
- Mỗi đối tượng chỉ được gán một class phù hợp nhất.

# 2. THUỘC TÍNH (ATTRIBUTES)

## 2.1. Occluded Level – Mức độ che khuất

| Giá trị | Tỷ lệ bị che | Mô tả |
|---|---|---|
| `0` | Dưới 10% | Nhìn thấy gần như toàn bộ đối tượng. |
| `1` | 10% – 35% | Bị che khuất nhẹ bởi vật cản. |
| `2` | Trên 35% – 80% | Bị che khuất nhiều nhưng vẫn nhận dạng được. |
| `3` | Trên 80% | Chỉ nhìn thấy một phần nhỏ, vẫn có đủ căn cứ nhận dạng. |

**Quy tắc:**
- Đánh giá phần diện tích đối tượng thực sự bị vật thể khác che khuất.
- Không tính phần nằm ngoài khung hình vào tỷ lệ occlusion.
- Nếu đối tượng bị che nhưng vẫn nhận dạng được thì vẫn gán nhãn.
- Nếu không đủ căn cứ xác định đối tượng thì không tự suy đoán để gán nhãn.

## 2.2. Truncated – Đối tượng bị cắt mép ảnh

| Giá trị | Điều kiện | Mô tả |
|---|---|---|
| `no` | Không bị cắt | Toàn bộ đối tượng nằm trong khung hình. |
| `yes` | Bị cắt mép | Một phần đối tượng vượt ra ngoài một trong bốn cạnh ảnh. |

**Quy tắc:**
- Đối tượng bị ảnh cắt mất một phần thì chọn `yes`.
- Đối tượng nằm sát mép nhưng vẫn nhìn thấy đầy đủ thì chọn `no`.
- Không nhầm lẫn giữa bị che khuất và bị cắt mép ảnh.

## 2.3. Can Review – Đánh dấu cần xem xét

| Giá trị | Ý nghĩa |
|---|---|
| `true` | Trường hợp khó, chưa chắc chắn, cần đưa ra Calibration. |
| `false` | Trường hợp rõ ràng, có thể gán nhãn bình thường. |

Đánh dấu `true` khi:
- Không chắc chắn giữa hai class.
- Đối tượng bị nhòe hoặc che khuất nghiêm trọng.
- Không xác định được ranh giới đối tượng.
- Xuất hiện phương tiện có cấu tạo đặc biệt.
- Có trường hợp chưa được quy định trong guideline.

# 3. QUY TẮC GÁN NHÃN CƠ BẢN

1. Chỉ gán nhãn các đối tượng thuộc 8 class đã quy định.
2. Mỗi đối tượng là một vùng gán nhãn độc lập.
3. Không gộp người và phương tiện thành một đối tượng.
4. Không gán nhãn phần nền, mặt đường hoặc vật thể không thuộc đối tượng.
5. Đối tượng bị che khuất vẫn được gán nhãn nếu còn đủ căn cứ nhận dạng.
6. Không tự vẽ hoặc suy đoán những phần đối tượng không nhìn thấy.
7. Không gán nhãn các hình ảnh chỉ là hình biểu diễn, hình phản chiếu hoặc vật thể mô phỏng người thật theo các quy định ở phần 4.

# 4. CÁC TRƯỜNG HỢP KHÓ VÀ CÁCH XỬ LÝ (EDGE CASES)

## 4.1. Xe di chuyển nhanh, hình ảnh bị nhòe (Motion Blur)

**Trường hợp:** Xe di chuyển với tốc độ cao khiến hình ảnh xuất hiện vệt nhòe, không nhìn rõ hình dạng hoặc đặc điểm của phương tiện.

**Cách xử lý:**

- Nếu xe bị nhòe quá mức, không thể xác định đó là loại phương tiện nào thì KHÔNG gán nhãn.
- Nếu vẫn nhận ra được class và xác định được vị trí, hình dáng đối tượng thì VẪN gán nhãn.
- Nếu đối tượng còn một phần bị vật thể khác che khuất thì đánh giá `occluded_level` theo tỷ lệ bị che thực tế.
- Nếu chỉ bị nhòe do chuyển động thì không tự động tăng `occluded_level`, vì nhòe chuyển động không đồng nghĩa với che khuất.
- Nếu không chắc chắn, đánh dấu `can_review = true`.

**Ví dụ:**
- Xe chỉ còn một vệt màu, không xác định được loại xe: Không gán.
- Xe bị nhòe nhưng vẫn nhận ra là ô tô: Gán `car`.
- Xe bị xe khác che một phần, phần còn lại bị nhòe nhưng vẫn nhận dạng được: Gán nhãn và đánh giá mức che khuất thực tế.

## 4.2. Tượng đá, tượng người và Manocanh

**Trường hợp:** Trong ảnh xuất hiện tượng đá, tượng trang trí, mô hình người hoặc manocanh có hình dáng giống người thật.

**Cách xử lý:**
- KHÔNG gán nhãn class `person`.
- Không phân biệt tượng có kích thước lớn hay nhỏ.
- Không gán nhãn manocanh trong cửa hàng, trung tâm thương mại hoặc khu vực trưng bày.

**Nguyên tắc:** Chỉ gán nhãn người thật, không gán các vật thể mô phỏng hình dáng con người.

## 4.3. Hình ảnh người trên Billboard, Poster hoặc Biển quảng cáo

**Trường hợp:** Xuất hiện hình người trên billboard, áp phích, poster, banner hoặc biển quảng cáo.

**Cách xử lý:**
- KHÔNG gán nhãn `person`.
- Không gán nhãn người trong ảnh quảng cáo dù hình ảnh rõ nét, có kích thước lớn hoặc giống người thật.
- Không gán nhãn hình người trên màn hình LED, màn hình điện tử hoặc bảng quảng cáo.

**Nguyên tắc:** Chỉ gán nhãn người thật xuất hiện trực tiếp trong môi trường thực tế.

## 4.4. Hình ảnh người hoặc phương tiện trong gương

**Trường hợp:** Người hoặc phương tiện xuất hiện trong gương chiếu hậu, gương trang trí, gương cửa hàng hoặc bề mặt phản chiếu.

**Cách xử lý:**
- KHÔNG gán nhãn đối tượng xuất hiện trong hình ảnh phản chiếu.
- Không gán nhãn `person` trong gương chiếu hậu.
- Không gán nhãn xe xuất hiện trong gương nếu đó chỉ là hình ảnh phản chiếu.

**Nguyên tắc:** Chỉ gán nhãn đối tượng thực tế xuất hiện trực tiếp trong khung cảnh, không gán nhãn hình ảnh phản chiếu của chúng.

## 4.5. Đồ chơi mô phỏng phương tiện giao thông

**Trường hợp:** Xuất hiện ô tô đồ chơi, xe máy đồ chơi, xe đạp mô hình hoặc các phương tiện mô phỏng có hình dáng tương ứng với các class trong dự án.

**Cách xử lý:**
- VẪN GÁN NHÃN nếu đồ chơi có thể xác định rõ thuộc một trong các class đã quy định.
- Ô tô đồ chơi: `car`.
- Xe tải đồ chơi: `truck`.
- Xe máy đồ chơi: `motorcycle`.
- Xe đạp đồ chơi: `bicycle`.
- Không tạo class mới dành riêng cho đồ chơi.

**Nguyên tắc:** Đồ chơi mô phỏng phương tiện vẫn được gán theo class tương ứng nếu nhận dạng được.

## 4.6. Người bị che khuất bởi phương tiện hoặc vật thể khác

**Trường hợp:** Người đi bộ bị ô tô, xe máy, cây cối hoặc vật thể khác che khuất.

**Cách xử lý:**
- Nếu vẫn nhận dạng được người thật thì gán nhãn `person`.
- Đánh giá `occluded_level` theo tỷ lệ diện tích bị che.
- Không vẽ thêm phần cơ thể bị che khuất.
- Nếu chỉ còn một phần rất nhỏ và không đủ căn cứ xác định là người thì không gán nhãn.

## 4.7. Phương tiện bị cắt khỏi khung hình

**Trường hợp:** Ô tô, xe máy hoặc phương tiện khác chỉ xuất hiện một phần ở cạnh ảnh.

**Cách xử lý:**
- Nếu vẫn xác định được class thì vẫn gán nhãn.
- Bounding box chỉ bao phủ phần đối tượng xuất hiện trong ảnh.
- Đánh dấu `truncated = yes`.
- Nếu đồng thời bị vật thể khác che khuất thì đánh giá thêm `occluded_level`.

## 4.8. Đối tượng quá nhỏ hoặc ở khoảng cách xa

**Trường hợp:** Phương tiện hoặc người xuất hiện ở xa, kích thước rất nhỏ.

**Cách xử lý:**
- Nếu vẫn nhận dạng được class và xác định được vị trí thì gán nhãn.
- Nếu quá nhỏ, không thể xác định chính xác đối tượng thì không gán nhãn.
- Không suy đoán class dựa trên vị trí hoặc ngữ cảnh đơn thuần.
- Đánh dấu `can_review = true` nếu trường hợp chưa rõ ràng.

## 4.9. Người và phương tiện xuất hiện cùng nhau

**Trường hợp:** Người ngồi trên xe máy, người dắt xe đạp hoặc người đứng cạnh phương tiện.

**Cách xử lý:**
- Gán `person` cho người.
- Gán class tương ứng cho phương tiện.
- Mỗi đối tượng có bounding box riêng.
- Không gộp người và phương tiện thành một bounding box.

# 5. BẢNG TỔNG HỢP CÁC TRƯỜNG HỢP KHÓ

| Trường hợp | Có gán nhãn? | Cách xử lý |
|---|---|---|
| Xe chạy nhanh bị nhòe, không nhận dạng được | Không | Bỏ qua, không suy đoán class. |
| Xe bị nhòe nhưng vẫn nhận dạng được | Có | Gán class tương ứng. |
| Xe bị che khuất nhưng vẫn nhận dạng được | Có | Gán nhãn và đánh giá occluded. |
| Tượng đá hình người | Không | Không gán `person`. |
| Manocanh | Không | Không gán `person`. |
| Người trên billboard/poster | Không | Không gán `person`. |
| Người trong gương | Không | Không gán đối tượng phản chiếu. |
| Xe trong gương | Không | Không gán đối tượng phản chiếu. |
| Ô tô đồ chơi | Có | Gán `car`. |
| Xe máy đồ chơi | Có | Gán `motorcycle`. |
| Phương tiện bị cắt mép ảnh | Có, nếu nhận dạng được | `truncated = yes`. |
| Người bị phương tiện che một phần | Có, nếu nhận dạng được | Đánh giá `occluded_level`. |
| Phương tiện quá nhỏ, không xác định được | Không | Không suy đoán class. |

# 6. CÁC LỖI CẦN TRÁNH

1. Gán nhãn hình người trên billboard hoặc poster.
2. Gán nhãn tượng đá hoặc manocanh thành `person`.
3. Gán nhãn hình ảnh phản chiếu trong gương.
4. Bỏ qua xe bị nhòe nhưng vẫn nhận dạng được.
5. Cố gắng gán class cho xe không thể nhận dạng.
6. Nhầm lẫn giữa motion blur và occlusion.
7. Đánh dấu `truncated = yes` khi đối tượng chỉ nằm gần mép ảnh.
8. Gộp người và phương tiện vào cùng một bounding box.
9. Tự suy đoán phần đối tượng bị che hoặc nằm ngoài ảnh.
10. Tự ý thêm class hoặc thay đổi thuộc tính.

# 7. QUY TRÌNH CALIBRATION

Khi gặp trường hợp khó hoặc chưa có quy định cụ thể:

1. Annotator phát hiện trường hợp chưa rõ.
2. Đánh dấu `can_review = true`.
3. Lưu lại ảnh và thông tin trường hợp cần xem xét.
4. Đưa vào danh sách Calibration.
5. Các thành viên trao đổi và thống nhất cách xử lý.
6. Người phụ trách xác nhận quy tắc.
7. Cập nhật guideline nếu cần thiết.
8. Áp dụng thống nhất cho các ảnh tiếp theo.

Không tự ý tạo quy tắc mới khi chưa có sự thống nhất của người phụ trách dự án.

# 8. CHECKLIST TRƯỚC KHI HOÀN THÀNH

- [ ] Đã gán đủ các đối tượng thuộc phạm vi dự án.
- [ ] Đã sử dụng đúng 8 class.
- [ ] Không gán nhãn tượng đá hoặc manocanh.
- [ ] Không gán nhãn người trên billboard, poster.
- [ ] Không gán nhãn hình ảnh phản chiếu trong gương.
- [ ] Đã gán nhãn đồ chơi mô phỏng phương tiện khi nhận dạng được.
- [ ] Không bỏ sót xe bị nhòe nhưng vẫn nhận dạng được.
- [ ] Không gán nhãn đối tượng không đủ căn cứ nhận dạng.
- [ ] Đã đánh giá `occluded_level` chính xác.
- [ ] Đã đánh giá `truncated` chính xác.
- [ ] Đã đánh dấu `can_review` cho các trường hợp khó.
- [ ] Không gộp nhiều đối tượng vào cùng một bounding box.

# 9. NGUYÊN TẮC CHUNG

**ACCURACY – CONSISTENCY – REVIEW**

- **Accuracy:** Đảm bảo đối tượng được gán đúng class và đúng vùng.
- **Consistency:** Áp dụng cùng một quy tắc cho toàn bộ dataset.
- **Review:** Chủ động đánh dấu các trường hợp khó để đưa ra Calibration.

**Quy tắc quan trọng nhất:**

Chỉ gán nhãn khi có đủ căn cứ nhận dạng đối tượng. Không suy đoán đối tượng từ những hình ảnh không rõ ràng. Đối tượng bị nhòe hoặc che khuất nhưng vẫn nhận dạng được thì tiếp tục gán nhãn theo quy định. Các hình ảnh mô phỏng, phản chiếu hoặc không thuộc đối tượng thực tế phải được xử lý theo quy tắc riêng trong guideline này.

Mọi thay đổi về class, thuộc tính và quy tắc gán nhãn cần được người phụ trách dự án xác nhận trước khi áp dụng.
:::Một lưu ý quan trọng khi áp dụng trên CVAT
Có một điểm mình muốn nhấn mạnh để tránh sai dữ liệu:
- Nhòe (Blur): Không đồng nghĩa với bị che khuất (occluded). Nếu xe nhòe nhưng vẫn nhận dạng được thì vẫn gán nhãn; nếu không nhận dạng được thì bỏ qua.
- Occluded: Chỉ dùng khi đối tượng thực sự bị một vật thể khác che.
- Truncated: Dùng khi đối tượng bị khung hình cắt mất một phần.