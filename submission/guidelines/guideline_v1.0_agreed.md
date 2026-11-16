# Guideline gán nhãn — v1.0 (nhóm thống nhất) — Robotaxi

- Phiên bản: v1.0 (khóa cho lượt gán độc lập)
- Ngày: 2026-11-16
- Gộp từ 5 bản cá nhân: 336, 231, 316, 323, 341
- Người điều phối chốt: … / Người kiểm lại: … (điền mã)
- Taxonomy: `submission/taxonomy.json` v1.0, mục `rules` giữ nguyên

## 1. Mục tiêu
Nhãn dùng để train detector 2D phát hiện người và phương tiện trên ảnh camera đường phố cho robotaxi.

Sai nguy hiểm theo thứ tự:
1. Bỏ sót người thật.
2. Bỏ sót xe thật.
3. Gán cho vật không thật (tượng, ảnh in, phản chiếu, đồ chơi), làm xe phanh vô cớ.
4. Nhầm lớp giữa các loại xe.

**Phép thử khi guideline chưa nói tới:** vật này có thật sự có mặt trong cảnh không, và xe có phải tránh nó không? Có cả hai thì gán. Là hình ảnh hoặc bản sao thì không gán. Thật nhưng không tách được từng cái thì vẽ `ignore_region`.

## 2. Lớp và thuộc tính
### Lớp (9 lớp, mỗi vật đúng một lớp)
| Lớp | Định nghĩa | Ví dụ | Dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con chở người: sedan, hatchback, SUV, wagon, taxi | img_015, img_023, img_040, img_019 (xe diễu hành chạy thật) | Ô tô đồ chơi, mô hình (img_038, xe vàng trên nóc img_024) → không gán |
| `van` | Xe thân liền một khối kín, cỡ nhỏ (kiểu Hiace) | img_005, img_016 | Pickup có nắp thùng → `truck`. Xe từ ~20 chỗ → `bus` |
| `truck` | Xe có cabin tách thùng hàng, **gồm pickup**, xe tải, đầu kéo, xe chở xe | img_001, img_002, img_007, img_036, img_037 | Tượng pickup img_027 → không gán |
| `bus` | Xe buýt, xe khách, minibus từ ~20 chỗ, có hàng cửa sổ dài | img_009, img_031, img_037 (nền) | Van nhỏ → `van` |
| `motorcycle` | Xe 2 bánh có động cơ, sidecar gộp chung box | img_026, img_022, img_029 | Xe đạp → `bicycle` |
| `three_wheeler` | Xe 3 bánh có động cơ: tuk-tuk, xe lam, auto-rickshaw | img_004, img_016, img_035 | Xích lô (đạp chân) → `bicycle` |
| `bicycle` | Xe chạy bằng sức người, 2 hoặc 3 bánh, gồm cả xích lô | img_003, img_006, img_023 | Xe đạp điện có bàn đạp: chọn `motorcycle` và tick `can_review` |
| `person` | Người thật: đi bộ, đứng, ngồi, lái hoặc ngồi trên xe hở | img_014, img_026, img_032 | Tượng img_017, ma-nơ-canh img_033, người trên biển quảng cáo img_018 |
| `ignore_region` | Vùng có vật thật nhưng không gán từng cái được (quá nhỏ, quá dày, quá nhòe) | img_021, img_030 | Không dùng cho vật không thật |

**Vật ngoài các lớp trên** (cọc giao thông, xe đẩy, gánh hàng, thú vật, tượng): không vẽ gì.

### Thuộc tính (không áp dụng cho `ignore_region`)
- `occluded_level`: phần trăm bị **vật khác** che.
  - `0`: < 10%
  - `1`: 10–40%
  - `2`: 40–80%
  - `3`: > 80% nhưng vẫn nhận ra lớp
  - Phân vân giữa hai mức thì chọn mức cao hơn.
  - **Không tính** những thứ sau là che: lưới hoặc hàng rào thưa nhìn xuyên được, nhòe chuyển động, phần người bị chính xe mình ngồi che.
  - **Có tính**: người lái che xe máy của họ.
- `truncated`: `yes` khi ≥ 10% vật nằm ngoài mép ảnh, còn lại `no`. Thuộc tính này độc lập với `occluded_level`.
- `can_review`: tick khi phân vân lớp, không chắc mép box, hoặc gặp ca có ghi "can_review" trong bảng mục 4. Đã tick vẫn phải chọn một quyết định cụ thể.

## 3. Quy tắc vẽ box
- **R-01 (vật thật):** chỉ gán vật thật. Tượng, ma-nơ-canh, đồ chơi, mô hình, ảnh in, tranh tường, linh vật: không vẽ gì, kể cả `ignore_region`.
- **R-02 (ôm sát):** box ôm sát phần nhìn thấy, lệch tối đa ~2 px. Không đoán phần bị che.
  - Có tính: gương chiếu hậu, mũ bảo hiểm, đồ mặc trên người.
  - Không tính: ăng-ten, bóng đổ, đồ người cầm hoặc gánh.
- **R-03 (người trên xe):** người và xe tách box. Mỗi người một `person` (người lái, người ngồi sau, trẻ em), xe một box riêng. Xe máy chở 2 người là 3 box. Người dắt xe hoặc ngồi trên xe đỗ cũng làm như vậy.
- **R-04 (che khuất):** vẫn gán nếu nhận ra lớp và box cao ≥ 20 px. Vật bị chia làm hai mảnh vẫn chỉ vẽ một box.
- **R-05 (cắt mép):** box dừng ở mép ảnh, đặt `truncated=yes`.
- **R-06 (vật nhỏ):** box cao < 20 px thì không gán. Từ 3 vật nhỏ sát nhau trở lên thì gộp thành một `ignore_region`. Một vật nhỏ đứng lẻ thì bỏ qua.
- **R-07 (hàng trên xe):**
  - Hàng nằm trong khung xe thì tính vào box xe, kể cả khi chất cao (img_012, img_035).
  - Xe chở xe (img_002): một box `truck` gồm cả các xe được chở, không gán `car` riêng.
  - Hàng chìa ra ngoài thân xe quá ½ chiều dài xe (ống ở img_029): phần chìa ra không tính vào box.
- **R-08 (người trong xe):** người chỉ thấy qua kính của xe kín thì không gán. Người trên xe hở (tuk-tuk, xích lô, xe mui trần) thì gán nếu thấy được từ đầu và vai trở lên.
- **R-09 (phản chiếu):** ảnh trong gương, kính, mặt đường ướt thì không gán (img_020, img_031, img_033).
- **R-10 (xe chủ):** phần xe của người chụp thì không gán (img_020).
- **R-11 (đám đông):** tách được thì gán từng vật, ưu tiên vật ở gần. Từ 5 vật cùng lớp trở lên chồng nhau, không tách được, thì vẽ một `ignore_region` ôm khối đó. Không phủ `ignore_region` lên vật đã có box.
- **R-12 (nhòe / tối):** còn thấy viền và nhận ra lớp thì gán bình thường. Chỉ còn vệt mờ thì vẽ `ignore_region` (img_014).
- **R-13 (xe lạ nhưng chạy thật):** xe diễu hành hoặc xe giả tưởng có người lái thì xếp theo chức năng (img_019 → `car`, tick `can_review`).
- **R-14 (ảnh rỗng):** ảnh không có box nào vẫn phải có trong COCO (img_017, img_020, img_027, img_033, img_038).

## 4. Bất đồng giữa 5 bản và quyết định nhóm
| # | Vấn đề | Các bản cá nhân | Nhóm chốt | Lý do |
|---|---|---|---|---|
| 1 | Xe ngoài taxonomy v0 (buýt, tuk-tuk, xe đạp) | 316, 323: thêm lớp. 231: thêm `three_wheeler`, `other_vehicle`. 336: vẽ `ignore_region`. 341: tạm gán `car`/`van` | Thêm `bus`, `three_wheeler`, `bicycle` | 3/5 bản muốn thêm lớp. Ép tuk-tuk thành `car` sẽ làm bẩn lớp `car` |
| 2 | Xích lô | 316: `bicycle`. 336: ignore. 341: không gán | `bicycle` | Đạp bằng sức người, cùng nhóm với xe đạp |
| 3 | Đồ chơi, mô hình xe | 323: vẫn gán theo lớp. 336, 231, 316, 341: không gán | Không gán (R-01) | 4/5 bản đồng ý. Đồ chơi không phải vật cản thật |
| 4 | Ngưỡng `occluded_level` | 231: 30/70%. 323: 35/80%. 336, 341, 316: khoảng 10/40/80% | 10 / 40 / 80% | Theo số đông, dễ ước bằng mắt |
| 5 | Ngưỡng `truncated` | 316: 5%. 323, 231: cứ bị cắt là `yes`. 336, 341: 10% | 10% | Tránh tick `yes` khi vật chỉ chạm mép một chút |
| 6 | Xe chở xe img_002 | 336, 316, 341: 1 box `truck`. 231, 323: không nói | 1 box `truck` | Cả khối di chuyển như một vật cản |
| 7 | Hàng chất cao img_035 | 336, 316: tính vào box. 341: bỏ phần hàng | Tính vào box (R-07) | Hàng nằm trong khung xe và đi cùng xe |
| 8 | Ngưỡng đám đông | 336: ≥ 5 vật. 341: ≥ 8 vật. 316: không tách được thì gộp | ≥ 5 vật và không tách được | Ảnh img_032 không có khối nào tới 8 vật mà vẫn khó tách |
| 9 | Mô tô trang trí img_011 | 316: không gán. 336, 341: `motorcycle` + `can_review` | `motorcycle` + `can_review` | Có thể là xe thật. Đem ra calibration |
| 10 | Gương ở đáy ảnh img_040 | 336: không gán. 316: `motorcycle` truncated | `motorcycle` truncated, `can_review` | Gương thuộc xe máy của người trong ảnh, không phải xe chủ |
| 11 | Hàng rào thưa che xe img_018 | 336: tối đa mức 1. 316: không tính là che | Không tính là che | Đơn giản hơn, ít lệch giữa các người gán |

## 5. Ca khó
| Ảnh | Tình huống | Rule | Quyết định |
|---|---|---|---|
| img_026 | Đôi nam nữ trên xe máy | R-03 | 2 `person` + 1 `motorcycle` |
| img_002 | Xe chở 9 xe con | R-07 | 1 `truck`, không gán `car` riêng |
| img_004 | 2 tuk-tuk có tài xế | R-03, R-08 | 2 `three_wheeler` + 2 `person` |
| img_017 | Tượng người cưỡi ngựa | R-01, R-14 | Không có box |
| img_021 | Ngã tư kín xe máy | R-11 | Tiền cảnh gán từng cặp người/xe, phía sau vẽ `ignore_region` |
| img_024 | Mô hình xe trên nóc xe thật | R-01, R-07 | Gán `car` cho xe thật, mô hình không gán riêng |
| img_029 | Xe máy chở ống dài | R-07 | `person` + `motorcycle`, không tính phần ống chìa ra |

## 6. QC và phiên bản
- Ngưỡng QC (giữ nguyên trong `taxonomy.json`):
  - Ghép box khi IoU ≥ 0.5
  - IAA IoU ≥ 0.85
  - Đồng thuận lớp ≥ 95%
  - IoU ≥ 0.85 cho box ≥ 50 px, ≥ 0.7 cho box nhỏ hơn
  - Box cao ít nhất 20 px
  - Tỉ lệ ảnh lỗi ≤ 2%
- Ca mơ hồ: lúc gán thì tick `can_review` hoặc ghi `image_issues.csv`, không xem nhãn của người khác. Người điều phối chốt ở Pha 5.
- Thay đổi làm đổi lớp, box hoặc thuộc tính là `semantic` và phải kiểm lại cả lô. Chỉ sửa câu chữ là `editorial`.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0 | Bản gốc của giảng viên | — |
| v1.0 | Gộp 5 bản cá nhân; thêm `bus`, `three_wheeler`, `bicycle`; R-01 đến R-14 | Bảng bất đồng ở mục 4 |
