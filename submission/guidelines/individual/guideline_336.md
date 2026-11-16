# Guideline gán nhãn — v0.1 — 036

- Phiên bản: v0.1 (bản cá nhân)
- Ngày: 2026-11-16
- Người viết: 036 / Người duyệt: nhóm (Pha 2)

## 1. Mục tiêu
Nhãn dùng để train detector 2D phát hiện người và xe trên ảnh đường phố (cho robotaxi).
Sai nguy hiểm nhất là **bỏ sót người thật** (nhất là người bị che, bị cắt mép). Sau đó là bỏ sót xe thật.
Gán nhầm tượng, đồ chơi hay ảnh in là người/xe cũng không tốt, vì xe sẽ phanh vô cớ.

## 2. Lớp và thuộc tính
### Lớp
| Lớp | Định nghĩa | Ví dụ | Dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con, SUV, taxi, không có thùng hàng riêng | img_015, img_023, img_019 (xe diễu hành có biển số) | Ô tô đồ chơi img_038 → không gán |
| `truck` | Xe có cabin tách riêng thùng hàng, **tính cả pickup** | img_001, img_002, img_007, img_036, img_037 | Tượng pickup img_027 → không gán |
| `van` | Xe thân liền một khối kín, cỡ nhỏ (kiểu Hiace) | img_005, img_016 | Pickup có nắp thùng img_037 → `truck` |
| `motorcycle` | Xe máy, xe ga, xe điện 2 bánh (sidecar gộp chung box) | img_026, img_022, img_029 | Xe đạp, tuk-tuk → ngoài taxonomy |
| `person` | Người thật đi bộ hoặc ngồi trên xe hở | img_014, img_032 | Tượng img_017, ma-nơ-canh img_033, người trên biển quảng cáo img_018 → không gán |
| `ignore_region` | Vùng không gán từng vật (đám đông dày, vật nhòe quá, xe ngoài taxonomy) | img_021, img_030 | Không dùng cho tượng/đồ chơi |

**Vật không thuộc lớp nào:**
- Xe thật nhưng ngoài taxonomy (xe buýt, tuk-tuk, xích lô, xe đạp): vẽ `ignore_region`, người ngồi trên xe vẫn gán `person`.
- Vật không thật (tượng, đồ chơi, ảnh in, ảnh phản chiếu): không vẽ gì.

### Thuộc tính
- `occluded_level` (chỉ tính phần bị vật khác che):
  - `0`: bị che < 10%
  - `1`: 10–40%
  - `2`: 40–80%
  - `3`: > 80% hoặc không đoán được
  - Phân vân giữa hai mức thì chọn mức cao hơn.
- `truncated`: `yes` nếu bị mép ảnh cắt mất khoảng ≥ 10% vật, còn lại là `no`.
- `can_review`: tick khi phân vân lớp, không chắc mép box, hoặc gặp ca HỎI ở mục 4.

## 3. Quy tắc vẽ box
- **R-01:** Box ôm sát phần nhìn thấy, không đoán phần bị che. Không tính gương, ăng-ten.
- **R-02:** Vật bị che vẫn gán nếu còn nhận ra được và box cao ≥ 20 px.
- **R-03 (người đi xe máy):** mỗi người một box `person`, xe một box `motorcycle`. Một xe chở 2 người là 3 box. Người ngồi trên xe đang đỗ cũng làm vậy.
- **R-04:** Vật bị cắt mép thì box dừng ở mép ảnh, đặt `truncated=yes`.
- **R-05:** Box cao < 20 px thì bỏ. Nhiều vật nhỏ sát nhau thì gộp thành một `ignore_region`.
- **R-06 (hàng trên xe):**
  - Hàng nằm trong thân xe thì tính vào box xe, kể cả xe con trên xe chở xe ở img_002.
  - Hàng chìa ra ngoài thân xe (ống dài ở img_029) thì không tính.
  - Đồ người cầm hoặc gánh không tính vào box người.
- **R-07:** Người chỉ nhìn thấy qua kính ô tô kín thì không gán.
- **R-08:** Tượng, ma-nơ-canh, đồ chơi, ảnh in, ảnh phản chiếu, xe của người chụp (ego): không gán.
- **R-09 (đám đông):** Tách được từng vật thì gán từng vật. Từ 5 vật trở lên chồng nhau không tách được thì vẽ một `ignore_region`.
- **R-10:** Bị nhòe nhưng vẫn thấy viền thì gán bình thường. Nhòe đến mức không thấy viền thì vẽ `ignore_region`.
- **R-11:** Ảnh không có vật nào vẫn phải có trong file COCO.

## 4. Ca khó
| Ảnh | Tình huống | Rule | Quyết định | Lý do |
|---|---|---|---|---|
| img_026 | Đôi nam nữ trên xe máy | R-03 | 2 `person` + 1 `motorcycle` | Tách box người và box xe. Thử lại trên img_010 vẫn đúng |
| img_002 | Xe chở 9 xe con | R-06 | 1 box `truck` gồm cả các xe được chở | Cả khối di chuyển như một vật. **Hỏi reviewer** |
| img_017 | Tượng người cưỡi ngựa | R-08, R-11 | Không có box | Tượng không phải người thật. Ảnh vẫn có trong COCO |
| img_004 | 2 tuk-tuk có tài xế | R-07, R-08 | Tuk-tuk vẽ `ignore_region`, tài xế gán `person` | Tuk-tuk ngoài taxonomy, xe hở nên thấy được tài xế |
| img_021 | Ngã tư kín xe máy | R-09 | Tiền cảnh gán từng cặp người/xe, phía sau vẽ `ignore_region` | Phía sau quá dày, không tách được. **Hỏi reviewer** chỗ ranh giới |

## 5. QC và phiên bản
- Ngưỡng lấy theo `taxonomy.json`:
  - Ghép box khi IoU ≥ 0.5
  - IAA IoU ≥ 0.85
  - Đồng thuận lớp ≥ 95%
  - Box cao ít nhất 20 px
  - Tỉ lệ ảnh lỗi ≤ 2%
- Ca mơ hồ: lúc gán thì tick `can_review`, đến Pha 5 người điều phối chốt.
- Muốn đem ra họp: có nên thêm lớp `bus` / xe 3 bánh không? Xe được chở ở img_002 có gán `car` riêng không?

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0 | Bản gốc của giảng viên | — |
| v0.1-036 | Thêm định nghĩa lớp, ngưỡng thuộc tính, R-01 đến R-11, bảng ca khó | img_002, img_004, img_017, img_021, img_026 |
