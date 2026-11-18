# Guideline gán nhãn — v1.1 (sau calibration) — Robotaxi

> **BẢN NHÁP chờ nhóm duyệt.** Nội dung đổi so với v1.0 nằm ở các mục đánh dấu **[v1.1]** và bảng lịch sử cuối file. Xóa dòng này sau khi họp chốt.

- Phiên bản: v1.1 (calibration sau IAA Pha 4), kế thừa v1.0
- Ngày: 2026-11-17
- Căn cứ: `submission/qc/iaa_summary.md` (882 cặp box ghép, IoU TB 0.811 < 0.85, đồng thuận lớp 98.0%, `occluded_level` khớp 52.2%), `calibration_candidates.csv` (1624 ca trên 35 ảnh), `calibration_decisions.csv` (1624 ca + 3 ca cùng sai R001–R003)
- Người điều phối chốt: 231 / Người kiểm lại: … (điền mã)
- Taxonomy: `submission/taxonomy_v1.1.json` (v1.1). Giữ nguyên 9 lớp, 3 thuộc tính và mục `rules`; chỉ sửa định nghĩa `car`, `van`, `truck` theo R-07, R-13, R-16. Bản gán độc lập vẫn theo `submission/taxonomy.json` v1.0. Thay đổi v1.1 là **semantic** (đổi cách chọn vật, vẽ box, chọn thuộc tính) nên phải kiểm lại cả lô.

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
| `car` | Ô tô con chở người: sedan, hatchback, SUV, wagon, taxi; **[v1.1]** gồm cả ô tô con nằm trên xe chở xe (R-07) | img_015, img_023, img_040, img_002 (8 xe trên sàn) | Ô tô đồ chơi, mô hình (img_038, xe vàng trên nóc img_024) → không gán. **[v1.1]** Xe diễu hành / xe giả tưởng (img_019) → không gán `car` (R-13) |
| `van` | Xe thân liền một khối kín, cỡ nhỏ (kiểu Hiace); **[v1.1]** gồm cả pickup có nắp thùng cao kín, nhìn liền khối (R-16) | img_005, img_016, img_037 | Pickup thùng hở → `truck`. Xe từ ~20 chỗ → `bus` |
| `truck` | Xe có cabin tách thùng hàng, gồm pickup **thùng hở**, xe tải, đầu kéo, xe chở xe | img_001, img_002 (đầu kéo + sàn), img_007, img_036 | Tượng pickup img_027 → không gán. **[v1.1]** Pickup có nắp thùng cao kín → `van` |
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
  - **[v1.1] Cách đo (R-15):** xem quy tắc R-15 ở mục 3. Ước theo **phần thân bị che**, không ước theo cảm giác "che nhiều/ít".
- `truncated`: `yes` khi ≥ 10% vật nằm ngoài mép ảnh, còn lại `no`. Thuộc tính này độc lập với `occluded_level`.
- `can_review`: tick khi phân vân lớp, không chắc mép box, hoặc gặp ca có ghi "can_review" trong bảng mục 4. Đã tick vẫn phải chọn một quyết định cụ thể.

## 3. Quy tắc vẽ box
- **R-01 (vật thật):** chỉ gán vật thật. Tượng, ma-nơ-canh, đồ chơi, mô hình, ảnh in, tranh tường, linh vật: không vẽ gì, kể cả `ignore_region`.
- **R-02 (ôm sát):** box ôm sát phần nhìn thấy, lệch tối đa ~2 px. Không đoán phần bị che.
  - Có tính: gương chiếu hậu, mũ bảo hiểm, đồ mặc trên người.
  - Không tính: ăng-ten, bóng đổ, đồ người cầm hoặc gánh.
  - **[v1.1]** `person`: từ đỉnh mũ/đầu tới điểm thấp nhất **nhìn thấy** của chân hoặc bàn chân; người ngồi xe máy thì đáy box dừng ở chỗ chân khuất sau xe. `motorcycle`: từ mép bánh trước tới mép bánh sau, gồm gương và chắn bùn; không kéo box theo chân người lái chìa ra ngoài. Hai người gán lệch nhau > 5 px ở một cạnh là phải vẽ lại (427 ca `iou_thap`, chủ yếu `person` và `motorcycle`).
- **R-03 (người trên xe):** người và xe tách box. Mỗi người một `person` (người lái, người ngồi sau, trẻ em), xe một box riêng. Xe máy chở 2 người là 3 box. Người dắt xe hoặc ngồi trên xe đỗ cũng làm như vậy.
- **R-04 (che khuất):** vẫn gán nếu nhận ra lớp và box cao ≥ 20 px. Vật bị chia làm hai mảnh vẫn chỉ vẽ một box.
- **R-05 (cắt mép):** box dừng ở mép ảnh, đặt `truncated=yes`.
- **R-06 (vật nhỏ):** box cao < 20 px thì không gán. Từ 3 vật nhỏ sát nhau trở lên thì gộp thành một `ignore_region`. Một vật nhỏ đứng lẻ thì bỏ qua.
- **R-07 (hàng trên xe):**
  - Hàng nằm trong khung xe thì tính vào box xe, kể cả khi chất cao (img_012, img_035).
  - **[v1.1 — đổi, ca R001]** Xe chở xe (img_002): một box `truck` cho đầu kéo và sàn chở, **cộng một box `car` riêng cho mỗi xe được chở nhìn rõ** (cao ≥ 20 px, theo R-04/R-06). Box `truck` không cần ôm hết các xe trên sàn.
    - v1.0 quy định chỉ 1 box `truck`. Ở img_002 cả 3 người được giao (231, 336, 341) đều vẽ thêm 8 `car`; nhóm chốt theo cách làm này vì mỗi xe trên sàn là một vật thật nhìn thấy rõ, detector cần học được hình dạng ô tô ở mọi vị trí.
    - Phạm vi: rà lại cả lô cho mọi xe chở xe / xe chở hàng là phương tiện.
  - Hàng chìa ra ngoài thân xe quá ½ chiều dài xe (ống ở img_029): phần chìa ra không tính vào box.
- **R-08 (người trong xe):** người chỉ thấy qua kính của xe kín thì không gán. Người trên xe hở (tuk-tuk, xích lô, xe mui trần) thì gán nếu thấy được từ đầu và vai trở lên.
- **R-09 (phản chiếu):** ảnh trong gương, kính, mặt đường ướt thì không gán (img_020, img_031, img_033).
- **R-10 (xe chủ):** phần xe của người chụp thì không gán (img_020).
- **R-11 (đám đông) [v1.1 viết lại]:** v1.0 để mỗi người tự quyết "tách được hay không", dẫn tới 645 ca bỏ sót/thừa và 75 ca bất đồng vùng ignore, dồn vào img_021, img_019, img_032, img_003. Từ v1.1, chia theo **chiều cao box**:
  1. **Tiền cảnh: vật cao ≥ 40 px** thì luôn gán từng cái (người và xe tách box theo R-03), kể cả khi bị che tới mức 3. Không được dùng `ignore_region` cho vật này.
  2. **Vùng giữa: vật cao 20–40 px** gán từng cái nếu thấy được ≥ 50% thân. Nếu ≥ 5 vật như vậy chồng nhau mà không thấy ≥ 50% thân từng cái thì gom vào `ignore_region`.
  3. **Hậu cảnh: vật cao < 20 px** không gán (R-06). Từ 3 vật trở lên thì gom vào `ignore_region`.
  4. Mỗi khối đám đông vẽ **một** `ignore_region` hình chữ nhật ôm cả khối, mép trên theo đỉnh đầu cao nhất, mép dưới theo vật thấp nhất trong khối. Không chia khối thành nhiều vùng nhỏ.
  5. `ignore_region` **không được phủ** lên box đã gán. Box nào có ≥ 50% diện tích nằm trong vùng ignore thì xóa box đó hoặc thu hẹp vùng.
- **R-12 (nhòe / tối):** còn thấy viền và nhận ra lớp thì gán bình thường. Chỉ còn vệt mờ thì vẽ `ignore_region` (img_014).
- **R-13 (xe diễu hành / xe giả tưởng) [v1.1 — đổi, ca R002]:** xe diễu hành, xe trang trí hoặc xe giả tưởng (img_019) **không gán** `car` hay lớp xe nào khác, không vẽ `ignore_region`. Người thật lái hoặc ngồi trên xe vẫn gán `person` theo R-08. Không vẽ một box `person` phủ cả đám đông (dùng R-11).
  - v1.0 xếp loại xe này theo chức năng (img_019 → `car`). Cả 3 người được giao (323, 336, 341) đều không gán xe này; nhóm chốt giữ cách làm đó vì đây là xe mẫu chạy cho sự kiện, không chạy thật ngoài đường, nên không gắn lớp `car`.
- **R-14 (ảnh rỗng):** ảnh không có box nào vẫn phải có trong COCO (img_017, img_020, img_027, img_033, img_038).
- **R-15 (đo `occluded_level`) [v1.1 mới]:** `occluded_level` chỉ khớp 52.2% giữa các cặp (422 ca). Cách đo:
  1. Hình dung box của **cả vật** nếu không bị che, rồi ước phần trăm diện tích bị **vật khác** che.
  2. Mốc cho `person` đứng/đi: thấy đủ người → `0`; mất bàn chân hoặc một tay → `1`; chỉ thấy từ hông trở lên → `2`; chỉ thấy đầu và vai → `3`.
  3. Người ngồi trên xe máy: phần chân bị chính xe mình che **không tính**. Chỉ tính phần bị người/xe khác che.
  4. `motorcycle` có người lái: người lái che thân xe → tối thiểu `1`; thấy ≤ 1 bánh xe → `2`; chỉ thấy đầu xe/gương → `3`.
  5. `car`/`truck`/`bus` bị xe khác che: tính theo tỉ lệ chiều dài thân xe bị che (≤ 1/10 → `0`, ≤ 2/5 → `1`, ≤ 4/5 → `2`, còn lại `3`).
  6. Phân vân giữa hai mức thì vẫn chọn mức cao hơn (giữ quy tắc v1.0).
- **R-16 (phân biệt lớp xe) [v1.1 mới]:** 18 ca `sai_lop` chủ yếu là car/van/truck.
  - SUV, MPV 7 chỗ, crossover → `car`, kể cả khi thân cao.
  - `van`: thân hộp liền khối, mui cao, đầu xe ngắn (Hiace, Transit, xe cứu thương cỡ nhỏ).
  - Có thùng hàng **hở** tách cabin (gồm pickup thùng hở) → `truck`.
  - **[v1.1 — đổi, ca R003]** Pickup có nắp thùng cao, kín, nhìn liền một khối với cabin (img_037) → `van`. v1.0 xếp loại này vào `truck`; ở img_037 hai trong ba người (336, 341) gán `van`, nhóm chốt theo số đông vì nhìn từ camera xe này giống thân van liền khối hơn xe tải thùng hở.
  - Box rộng ngang ôm một xe nhiều cửa sổ là `bus`, không bao giờ là `person` (ca img_009).
  - Một box trùng cả người lẫn xe máy là lỗi R-03: tách thành hai box.
- **R-17 (phủ đủ vật) [v1.1 mới]:** mọi vật thật thuộc taxonomy, cao ≥ 20 px và không nằm trong `ignore_region` đều phải có box. Trước khi lưu, rà ảnh theo lưới trái → phải, trên → dưới. Ca chỉ 1 người vẽ được giữ lại nếu vật thỏa R-17 (kiểm trên ảnh gốc, không bỏ phiếu).

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
| img_002 | Xe chở 9 xe con | R-07 | v1.0: 1 `truck`, không gán `car` riêng. **v1.1:** 1 `truck` + 1 `car` cho mỗi xe nhìn rõ trên sàn (8 xe) |
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
| v1.1 | **semantic**: viết lại R-11 (ngưỡng 40/20 px cho đám đông, một vùng ignore mỗi khối); thêm R-15 (mốc đo `occluded_level`), R-16 (phân biệt car/van/truck/bus), R-17 (phủ đủ vật); làm rõ R-02 (mép box person/motorcycle); **đổi R-07** (xe chở xe: 1 `truck` + `car` riêng, R001), **đổi R-13** (xe diễu hành không gán, R002), **đổi định nghĩa `van`/`truck`** (pickup có nắp thùng kín → `van`, R003). Taxonomy lên v1.1 (`submission/taxonomy_v1.1.json`, chỉ đổi định nghĩa, không đổi lớp/thuộc tính) | IAA Pha 4 chưa đạt: IoU TB 0.811, `occluded_level` khớp 52.2%, 645 ca bỏ sót/thừa, 75 ca bất đồng ignore, 18 ca sai lớp; ca cùng sai img_002, img_019; ca đa số sai img_037 |

### Bất đồng phát hiện ở Pha 4 và cách v1.1 xử lý [v1.1]
| Loại ca (số ca) | Nguyên nhân chính | Cả nhóm lệch hay một người lệch | Quy tắc v1.1 |
|---|---|---|---|
| `bo_sot_thua` (645) | Đám đông xe máy, người hậu cảnh: 323 vẽ 311 box, 336 chỉ vẽ 178 box trên cùng 24 ảnh | Cả nhóm: v1.0 không có ngưỡng rõ khi nào gán từng cái | R-11, R-17, R-06 |
| `sai_thuoc_tinh` (457; 422 là `occluded_level`, 35 là `truncated`) | Ước phần trăm che bằng cảm giác | Cả nhóm | R-15, R-05 |
| `iou_thap` (427) | Mép box person (chân) và motorcycle (gương, chân người lái) | Cả nhóm | R-02 |
| `bat_dong_ignore` (75) | Người chia khối thành nhiều vùng nhỏ, người khác vẽ từng box | Cả nhóm | R-11 |
| `sai_lop` (18) | car/van/truck; person↔motorcycle; 231 gán person cho xe buýt img_009 | Phần lớn là một người lệch | R-16, R-03 |
| Cả nhóm làm khác v1.0, IAA không cờ (img_002) | 3/3 người vẽ 8 `car` trên xe chở xe, v1.0 R-07 chỉ cho 1 `truck` | Quy tắc v1.0 không hợp với cách nhóm thực sự gán | Đổi R-07 theo cách làm của nhóm (R001) |
| Cả nhóm làm khác v1.0, IAA không cờ (img_019) | 3/3 người không gán xe diễu hành, v1.0 R-13 ghi `car` | Xe mẫu chạy cho sự kiện, không chạy thật ngoài đường | Đổi R-13: không gán xe giả tưởng (R002) |
| Lệch lớp (img_037) | 336, 341 gán `van`, 316 gán `truck` (đúng v1.0) cho pickup có nắp thùng | Định nghĩa `van`/`truck` v1.0 chưa khớp cách nhìn của đa số | Đổi định nghĩa: pickup nắp thùng kín → `van` (R003); 316 sửa nhãn |
