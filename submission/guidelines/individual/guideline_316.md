# Guideline gán nhãn — v0 sửa đổi (cá nhân) — 316

- Phiên bản: v0 sửa đổi (bản cá nhân, chưa phải v1.0 của nhóm)
- Ngày: 2026-11-16
- Loại nhãn: bounding box 2D (hộp chữ nhật) trên ảnh, gán bằng CVAT, xuất COCO 1.0
- Phạm vi: 40 ảnh `data/images/img_001.jpg … img_040.jpg` (cạnh dài tối đa 1024 px)
- Người viết / duyệt: 316 (tự viết, chưa duyệt)
- Taxonomy: thuộc tính và ngưỡng trùng `submission/taxonomy.json` hiện tại. Ba lớp `bus`, `tuk_tuk`, `bicycle` là **đề xuất của tôi**, chưa có trong taxonomy; nhóm quyết ở Pha 2. Nếu không thêm: `bus` gộp vào `van`/`truck`, `tuk_tuk` và `bicycle` theo quyết định nhóm.

> Cách đọc: mục 1–2 cho biết **gán cái gì**, mục 3 cho biết **vẽ thế nào**, mục 4 là **ca khó có ảnh**, mục 5 là **QC và phiên bản**.
> Mã `R-xx` là quy tắc được dẫn lại trong ghi chú, phiếu QC và `calibration_decisions.csv`.

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện và đánh giá **mô hình phát hiện vật thể 2D** cho camera của xe robotaxi chạy trong đô thị đông xe máy (Việt Nam, Thái Lan và các thành phố tương tự).

Hai kiểu sai có hậu quả khác nhau, nên guideline phải chặn cả hai:

| Kiểu sai của nhãn | Mô hình học được gì | Hậu quả với robotaxi |
|---|---|---|
| **Bỏ sót** người / xe máy / xe thật | coi họ là nền | không phanh, không tránh |
| **Gán thừa** tượng, ảnh in, phản chiếu, đồ chơi | thấy "người" ở nơi không có người | phanh gấp vô cớ, kẹt xe |
| Box lỏng / lệch | vị trí và kích cỡ sai | đánh giá khoảng cách, dự đoán quỹ đạo sai |
| Lớp lẫn lộn (xe máy ↔ xe đạp, car ↔ truck) | hành vi dự đoán sai (xe máy luồn lách, xe tải phanh dài) | dự đoán quỹ đạo sai |

**Phép thử chung cho mọi quyết định** (dùng khi guideline chưa nói tới ca đó):

> *"Mô hình phát hiện của xe có phải tránh / nhường / dự đoán chuyển động của vật này không, và vật này có **thật sự có mặt** trong cảnh không?"*
> Có cả hai → gán. Là hình ảnh / bản sao / vật trang trí → không gán. Là vật thật nhưng không gán được từng cái → `ignore_region`.

---

## 2. Lớp và thuộc tính

### 2.1 Bảng lớp (mỗi vật thuộc **đúng một** lớp)

| Lớp | Định nghĩa (một câu) | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Người thật, có mặt ở bất cứ tư thế nào: đi bộ, đứng, ngồi, đang lái hay đang ngồi sau xe; kể cả trẻ em. | img_014 (hai người qua đường), img_034, img_005 (người ngồi xe máy), img_001 (khách quán) | Tượng người (img_017), ma-nơ-canh (img_033), ảnh in trên biển quảng cáo (img_018, img_030), người chỉ thấy qua phản chiếu (img_020), linh vật/vật trang trí dạng tượng (img_011) |
| `motorcycle` | Xe **có động cơ**, hai bánh (xe số, xe ga, mô tô), kể cả xe gắn thùng bên (sidecar). Không gồm người ngồi. | img_006, img_008, img_026, img_022 (có sidecar) | Xe đạp (→ `bicycle`), xe ba bánh có mái (→ `tuk_tuk`), xe chỉ là tượng / mô hình (img_011) |
| `bicycle` | Xe đạp, xe đạp thồ, **xích lô / cyclo đạp chân** (2–3 bánh, chạy bằng sức người). Không gồm người đạp, người ngồi. | img_006 (xe đạp bên phải), img_003 (xe đạp có giỏ), img_009, img_023, img_034, img_039 (cyclo) | Xe máy nhìn giống xe đạp (có bình xăng, không bàn đạp → `motorcycle`); xe không rõ bàn đạp: chọn `motorcycle` và tick `can_review` |
| `tuk_tuk` | Xe **ba bánh có động cơ, có mái hoặc khung** chở khách / hàng (tuk-tuk, auto-rickshaw, xe lam, xe lôi máy). | img_004, img_016, img_035, img_040 (nền) | Cyclo đạp chân (→ `bicycle`); xe máy kéo thùng nhỏ không có cabin: `motorcycle` + `can_review` |
| `car` | Ô tô con chở người, thân kín, tối đa khoảng 9 chỗ: sedan, hatchback, SUV, wagon, taxi, minivan gia đình. | img_015, img_023 (Honda đen), img_024 (Pacer wagon), img_040 (taxi), img_019 (xe giả tưởng chạy thật, R-12) | Xe mô hình / xe tượng / bia mộ hình xe (img_027, img_038, xe vàng trên nóc Pacer img_024) |
| `van` | Xe thùng kín cỡ nhỏ–vừa, cabin liền thân, cửa trượt hoặc hai cánh sau, thường chở nhóm người hoặc hàng nhẹ (kiểu Hiace, Transit, xe bán tải kín). | img_005 (xe xám phía sau), img_016 (xe trắng bên phải) | Xe chở ≥ 20 chỗ, dài (→ `bus`); xe 4 cửa gầm thấp (→ `car`) |
| `truck` | Xe có **thùng hàng riêng**: bán tải (pickup), tải nhỏ/vừa/nặng, đầu kéo + rơ-moóc, xe chở xe, xe thùng kín cỡ tải. | img_001, img_012, img_037 (pickup), img_036 (tải xanh), img_007 (Isuzu), img_002 (xe chở xe) | Van (cabin liền thùng, cỡ nhỏ); pickup mô hình / bia mộ (img_027) |
| `bus` | Xe buýt, xe khách, minibus cỡ lớn (≥ ~20 chỗ), có hàng cửa sổ dài hai bên. | img_009 (xe khách kem), img_031 (buýt đêm), img_037 (buýt xanh, nền), img_040 (buýt nền) | Van chở người cỡ nhỏ (→ `van`) |
| `ignore_region` | Hộp bao quanh **vật thật mà không thể / không cần gán từng cái** (quá nhỏ, quá dày, quá mờ, bị che gần hết). Bỏ sót bên trong vùng này không tính là lỗi. | cụm xe máy xa trong img_021, bãi xe dày img_030, vật nhỏ trong img_013 | Tượng, ảnh in, phản chiếu, đồ chơi (đó là **không vẽ gì**, không phải ignore — R-10, R-11) |

**Vật không thuộc lớp nào** (nón giao thông, biển báo, ô, xe đẩy, gánh hàng, thú vật, tượng thú, xe lăn, xe nâng, vật dụng bàn ghế…): **không vẽ gì**. Không tạo lớp mới ngoài taxonomy; không dùng `ignore_region` cho chúng (R-21).

### 2.2 Phân lớp xe — cây quyết định nhanh (R-17)

1. Có động cơ **và** hai bánh? → `motorcycle`. Chạy bằng sức người, hai hoặc ba bánh (gồm cyclo)? → `bicycle`.
2. Ba bánh có động cơ, có mái / khung chở người? → `tuk_tuk`.
3. Bốn bánh trở lên, có **thùng hàng riêng** (thùng hở, bệ hàng, thùng rời)? → `truck`.
4. Thân kín liền khối, ≥ ~20 chỗ, hàng cửa sổ dài? → `bus`.
5. Thân kín liền khối, kiểu Hiace / Transit, cửa hông trượt, nóc cao? → `van`.
6. Còn lại, ô tô chở người gầm thấp (kể cả SUV, wagon, minivan gia đình)? → `car`.
7. Vẫn phân vân giữa hai lớp: chọn lớp theo **chức năng thực tế** ở bước trên, tick `can_review`, ghi ảnh và lý do vào ghi chú. Không để trống, không vẽ hai box.

### 2.3 Thuộc tính (đúng `submission/taxonomy.json`)

Áp dụng cho mọi lớp **trừ** `ignore_region` (không cần thuộc tính).

**`occluded_level`** — vật bị **vật khác** che (không phải mép ảnh). Ước lượng tỉ lệ diện tích bị che so với toàn bộ vật nằm trong khung ảnh.

| Mức | Tỉ lệ bị che | Phần nhìn thấy | Dấu hiệu nhận ra |
|---|---|---|---|
| `0` | ≤ 10% | ≥ 90% | thấy gần như đủ |
| `1` | > 10% đến 40% | 60–< 90% | thiếu một góc / một phần thân |
| `2` | > 40% đến 75% | 25–< 60% | chỉ còn nửa vật hoặc ít hơn |
| `3` | > 75% | 15–< 25% | chỉ còn một mảnh, vẫn xác định được lớp |

- Dưới **15%** nhìn thấy, hoặc không chắc lớp: không vẽ box, đưa vào `ignore_region` (R-19).
- Che bởi vật **mảnh, xuyên thấu** (lưới, hàng rào sắt, cành cây nhỏ, kính trong): **không** tính vào mức che; nếu vì thế mà nhìn mờ, tick `can_review` (R-05).
- Người ngồi trên xe: phần thân dưới bị **chính xe họ ngồi** che không tính là bị che.
- Cờ `occluded` sẵn của CVAT (phím `Q`) **bỏ qua**, chỉ dùng `occluded_level`.

**`truncated`** — vật bị **mép ảnh** cắt.

| Giá trị | Khi nào |
|---|---|
| `yes` | ước ≥ 5% vật nằm ngoài khung ảnh, hoặc vật chạm mép và rõ ràng còn tiếp tục ra ngoài |
| `no` | còn lại (mặc định) |

Truncated và occluded **độc lập**: một vật có thể vừa bị cắt mép vừa bị che (R-06).

**`can_review`** (checkbox) — tick khi không chắc về lớp, ranh giới ca, hoặc phần che. Tick là **cho phép** nhưng **không thay** việc phải chọn một quyết định cụ thể (R-20).

### 2.4 Ngưỡng QC dùng trong lab (không đổi, nằm trong `taxonomy.json › rules`)

| Tham số | Giá trị |
|---|---|
| `min_box_height_px` | 20 (box cao < 20 px không được là vật thường) |
| `iou_thr_large` / `large_px` | 0.85 với box cao ≥ 50 px |
| `iou_thr_small` | 0.7 với box cao < 50 px |
| `iaa_iou` / `iaa_class` | IoU TB ≥ 0.85; cùng lớp ≥ 95% (chỉ trên cặp ghép 1–1) |
| `match_iou` | 0.5 (ngưỡng ghép hai box về cùng một vật) |
| `error_rate_img_max` | 2% ảnh có lỗi (trên ảnh đã duyệt) |

---

## 3. Quy tắc vẽ box

Mọi số đo "px" tính trên **ảnh gốc trong `data/images/`** (kích thước thật của file), không tính theo độ zoom của CVAT.

### R-01. Chỉ gán vật thật đang có mặt trong cảnh
Vật phải hiện diện vật lý. Hình ảnh của vật (ảnh in, tranh, màn hình), bản sao (tượng, mô hình, đồ chơi, ma-nơ-canh), và ảnh phản chiếu **không** là vật. Chi tiết ở R-10, R-11.

### R-02. Một vật — một box — một lớp
Không vẽ hai box cho cùng một vật; không vẽ box "ôm nhóm" cho nhiều vật tách rời được. Vật bị chia đôi bởi vật che (xe bị cột chắn giữa) vẫn là **một** box nếu rõ ràng là một vật.

### R-03. Độ sát của box
- Box chạm đến điểm ngoài cùng của phần **nhìn thấy**: mép trên cùng, dưới cùng, trái, phải. Lề thừa ≤ **2 px** hoặc ≤ 2% cạnh box (lấy số lớn hơn).
- **Gồm:** bánh, gương chiếu hậu, thùng hàng gắn liền, đèn, biển số, mũ bảo hiểm, tóc, bàn tay, bàn chân.
- **Không gồm:** ăng-ten mảnh, dây điện, bóng đổ, vệt phản chiếu dưới xe, cờ, đồ vật người đang cầm hoặc mang (túi, ô, gánh hàng, cần câu).
- Box vẽ trên phần **nhìn thấy**, **không** suy đoán phần bị che (kể cả khi đoán được hình dạng đầy đủ).
- Vật bị cắt mép ảnh: box dừng đúng ở mép (tọa độ 0 hoặc rộng/cao ảnh).

### R-04. Người và xe tách thành hai box
Một người ngồi trên xe máy / xe đạp / cyclo / tuk-tuk hở là **hai vật**:
- `person` — phần thân người nhìn thấy (mũ bảo hiểm → bàn chân, kể cả phần chân chồng lên xe).
- Lớp xe — phần thân xe nhìn thấy (không cố cắt phần người đè lên; phần chồng nhau tự nhiên nằm trong cả hai box).
- Mỗi người ngồi (lái, sau, trước, trẻ em) có **box riêng**. Xe máy chở 2–3 người = 1 box xe + 2–3 box `person`.
- Người **dắt** xe, **đứng cạnh** xe: hai box tách như vật độc lập.
- Mức che: người bị xe mình ngồi che **không tính** (xem 2.3). Người bị người khác / vật khác che tính bình thường.

### R-05. Vật bị che
- Gán nếu nhìn thấy ≥ 15% và xác định được lớp; ghi `occluded_level` theo bảng 2.3.
- Che bởi vật xuyên thấu / mảnh: xem 2.3, không đẩy mức lên.
- Vật chỉ nhìn thấy qua cửa kính: nhìn rõ → gán như thường; kính phản chiếu mạnh hoặc mờ → R-15.

### R-06. Vật bị cắt mép
`truncated=yes` theo bảng 2.3 và box dừng ở mép. Mép ảnh chỉ cắt một phần nhỏ (< 5%) thì giữ `no`. Một vật vừa cắt mép vừa bị che: tick **cả hai**.

### R-07. Vật nhỏ (ngưỡng 20 px)
- Đo **chiều cao** box (px, ảnh gốc). **< 20 px** → **không** vẽ box thường; gộp các vật nhỏ liền kề vào một `ignore_region` (R-08).
- **≥ 20 px** → gán bình thường, kể cả khi ở xa.
- Vật cao 18–22 px (sát ngưỡng): đo bằng công cụ đo của CVAT trên đúng ảnh gốc; không chắc, ưu tiên `ignore_region` và tick `can_review`.
- Box nhỏ tính IoU với ngưỡng 0.7 (cao < 50 px), nên đặt cạnh box **chính xác tới từng điểm ảnh**.

### R-08. `ignore_region` — dùng khi nào, vẽ thế nào
Dùng cho vật **thật** mà mô hình không cần / không thể học từng cái:
1. Vật cao < 20 px (R-07).
2. Cụm vật dày, chồng chéo không tách được (R-09).
3. Vật nhìn thấy < 15% hoặc không chắc lớp (R-19).
4. Vật nhòe chuyển động nặng, thiếu sáng nặng, không phân biệt được lớp (R-18).

Cách vẽ:
- Một hộp bao **sát** cụm, không bao trùm các box đã gán (tránh phủ lên vật đã gán).
- Không dùng `ignore_region` để "bỏ qua cho đỡ phải vẽ" hoặc cho vật **giả** (tượng, đồ chơi, ảnh in — R-10).
- Không có thuộc tính.
- Một người vẽ `ignore_region` phủ một box mà người kia vẽ box riêng → ca bất đồng, đưa vào calibration.

### R-09. Đám đông và hàng xe dày
- Người / xe **tách được** (thấy ≥ 30% thân, cao ≥ 20 px, ranh giới rõ) → box riêng, có `occluded_level`.
- Phần còn lại là **khối** không tách được (hàng chục người chen nhau, xe máy chồng chất) → **một** `ignore_region` phủ khối; **không** cố đoán từng người.
- Thứ tự ưu tiên khi vẽ: vật ở **gần camera** trước, vật ở xa nhét vào ignore.
- Ví dụ: img_032 (hàng người chờ xe buýt): box từng người tách được ở hàng đầu, phần dày phía sau → ignore; img_021 (dòng xe máy): vẽ các xe gần, cụm xa nhỏ → ignore; img_030 (bãi xe): vẽ vài xe rõ phía trước, khối chồng chéo phía sau → ignore.

### R-10. Vật không thật → **không vẽ gì**
Không vẽ box, không vẽ ignore cho:
| Loại | Ví dụ ảnh |
|---|---|
| Tượng người / thú (kể cả tượng cưỡi ngựa) | img_017, ngựa trong thùng pickup img_028 |
| Ma-nơ-canh, hình nộm | img_033 |
| Đồ chơi, mô hình xe thu nhỏ (kể cả cả tủ kính) | img_038 |
| Xe mô hình đặt trên xe thật | xe vàng trên nóc Pacer img_024 |
| Bia mộ / đài tưởng niệm hình xe | img_027 |
| Chân dung in trên biển quảng cáo, tranh tường, bảng hiệu | img_018, img_030 (áp phích), img_026 (tranh tường) |
| Điêu khắc trang trí, tượng nhân vật hóa trang dạng tượng | img_011 (Predator) |

Nghi ngờ vật là thật hay giả: áp **phép thử mục 1**. Vẫn không chắc → vẽ theo cách phổ biến nhất (không vẽ) và ghi vào `image_issues.csv` / tick `can_review` nếu đã vẽ.

### R-11. Phản chiếu → không vẽ
Vật thấy qua gương, kính cửa sổ, mặt đường ướt, mặt nước **không** là vật.
- img_020: xe và cảnh trong gương chiếu hậu → không vẽ; phần xe chủ (khung gương) cũng không vẽ (R-14).
- img_013, img_031, img_015: ánh phản chiếu đèn trên mặt đường ướt → không vẽ.
- img_033: bóng người và xe trên kính tủ → không vẽ.
- Vật **thật** đứng sau kính nhìn rõ, ví dụ người sau cửa kính cửa hàng: gán như thường nếu nhìn rõ.

### R-12. Xe giả tưởng / xe diễu hành
Một vật là xe **thật** nếu nó là phương tiện chạy được trên đường với người điều khiển. Hình dạng không bình thường (xe hoạt hình, xe diễu hành) vẫn xếp theo **chức năng**.
- img_019 (xe Danger Mouse): xe thật, tư thế lái, đang chạy trong đoàn diễu hành → `car`, người lái → `person`. Chuột trắng bên cạnh là **hình nộm / linh vật** → không vẽ. Tick `can_review` ở xe để nhóm kiểm lại.
- Mô hình đồ chơi (xe nhỏ không người) → R-10.

### R-13. Xe chở xe, hàng hóa, rơ-moóc
- **Xe chở xe** (img_002): một box `truck` bao toàn bộ cụm (đầu kéo + khung chở). Các xe **đang được chở** là hàng hóa → **không** vẽ box riêng.
- Hàng xếp trên / trong xe (thùng hộp, bao tải, rau củ trong thùng pickup img_012, bao hàng ngất ngưởng trên tuk-tuk img_035): **gồm trong box xe**.
- Hàng dài mảnh nhô ra khỏi thân xe (ống, sào, thanh — img_029): chỉ gồm phần nhô ra **≤ ½ chiều dài thân xe**; phần nhô xa hơn bỏ khỏi box (không kéo box sang vật khác).
- Rơ-moóc đang nối với đầu kéo: **một box** cho cả bộ. Rơ-moóc đã tháo, đứng riêng: box riêng `truck`.
- Người ngồi / đứng trên thùng hàng: `person` riêng (R-04).

### R-14. Xe chủ (ego) và phần của máy ảnh / xe chụp
Phần xe của người chụp (nắp capo, khung cửa kính, gương chụp từ trong xe, kính chắn gió của xe chụp) **không** gán. Img_020: khung gương và cửa xe chụp → không gán. Img_040: nắp capo xanh-đỏ ở cạnh dưới có thể là xe taxi **khác** (đang nằm trong cảnh) → gán `car`; người đội mũ bảo hiểm đen đứng gần camera và gương chiếu hậu dưới cùng của **xe máy** họ đang ngồi → `person` + `motorcycle` (cả hai `truncated=yes`).

### R-15. Người trong xe
- Người trong **xe kín** (cabin có kính chắn): thấy rõ mặt/thân trên qua kính → **không** gán (không phải đối tượng tách biệt có mặt ngoài đường); tránh tranh luận nên **không gán** cho mọi trường hợp qua kính.
- Người trong xe **hở / cabin mở** (tuk-tuk không kính, thùng pickup, xích lô, xe mui trần): gán nếu thấy ≥ đầu + vai.
- Người ngồi sau kính cửa hàng, quán: nhìn rõ → gán (không phải xe).

### R-16. Xe nhiều phần
- Xe máy + sidecar (img_022): **một** `motorcycle` bao cả sidecar; người lái → `person`.
- Xe đạp thồ gắn thùng, xe ba gác đạp chở hàng: `bicycle` một box bao thùng.
- Xe ba bánh có động cơ (kể cả có thùng sau) có mái / khung chở người → `tuk_tuk`; xe máy kéo thùng không có cabin → `motorcycle` (`can_review`).

### R-17. Ranh giới giữa các lớp xe
Theo cây quyết định mục 2.2. Ví dụ đã chốt:
| Phương tiện | Lớp |
|---|---|
| Pickup thùng hở, pickup có nắp thùng (img_037), tải nhỏ có bạt (img_036) | `truck` |
| Xe Hiace xám phía sau trong img_005, xe trắng trong img_016 | `van` |
| Xe khách cỡ vừa (img_009), buýt thành phố (img_031, img_037, img_040) | `bus` |
| Taxi sedan, Honda đen (img_023), SUV | `car` |
| Cyclo (img_006, 009, 023, 034, 039) | `bicycle` (+ `person` người đạp, người ngồi) |
| Tuk-tuk xanh, auto-rickshaw vàng | `tuk_tuk` |

### R-18. Nhòe chuyển động, thiếu sáng, ngược sáng
- Nhận ra được lớp **và** ranh giới → gán, nếu mờ tick `can_review`.
- Chỉ còn vệt mờ, không xác định lớp → `ignore_region` (img_014: các xe máy chạy nhòe phía sau hai người đi bộ).
- Ảnh đêm, ngược sáng (img_013, img_015, img_031): chỉ gán vật nhìn rõ hình dạng; vật chỉ là đốm đèn → không gán (không có hình dạng để vẽ).

### R-19. Ngưỡng nhìn thấy tối thiểu
Nhìn thấy < 15% vật hoặc không chắc lớp → không vẽ box thường, gộp vào `ignore_region` (R-08). 15–< 25% nhìn thấy và chắc lớp → `occluded_level=3`.

### R-20. Chưa chắc thì làm gì
1. Chọn **một** quyết định cụ thể theo guideline (lớp, box, mức che).
2. Tick `can_review` trên box đó.
3. Nếu không vẽ được box: ghi vào `submission/qc/image_issues.csv` (`image, annotator, region, note`) **sau khi khóa nhãn**.
4. **Không xem nhãn của người khác** để quyết định; ca mơ hồ chỉ chốt ở buổi calibration.

### R-21. Ngoài taxonomy
Vật không có lớp phù hợp (nón giao thông, biển báo, xe đẩy, gánh hàng, thú vật, xe lăn) → **không vẽ gì**; người bán hàng rong vẫn là `person`, còn gánh hàng / xe đẩy của họ thì không gán. Không thêm lớp tạm, không dùng `ignore_region` (vì đó không phải vật bị nhỏ/dày mà là vật nằm ngoài phạm vi). Nếu ca này xuất hiện lặp lại và ảnh hưởng đến mục tiêu → đề xuất ở calibration, đổi taxonomy theo `semantic` change.

### R-22. Xe đỗ, xe dừng
Xe đỗ, xe tắt máy, xe ở trong nhà kho đều **được gán** (img_007, img_008, img_030, img_036, img_037). Robotaxi vẫn phải thấy xe đỗ để tránh / đánh giá cửa mở.

### R-23. Mặc định và thứ tự thao tác
1. Chọn lớp, 2. vẽ box, 3. đặt `occluded_level`, 4. `truncated`, 5. `can_review` nếu cần.
Mặc định: `occluded_level=0`, `truncated=no`, `can_review=false`. `ignore_region` không điền thuộc tính.

---

## 4. Ca khó (có ảnh, rule, quyết định, lý do)

| Ảnh | Tình huống | Rule | Quyết định | Lý do |
|---|---|---|---|---|
| img_001 | Pickup trắng (thùng hở), khách ngồi quán phía sau, phụ nữ xăm hình đứng sát, mép trái bị cắt | R-04, R-06, R-17 | `truck` (kèm hàng ống thép trên thùng), mỗi khách `person` riêng với `occluded_level` theo mức che bởi bàn/xe; người phụ nữ `truncated=yes` nếu bị cắt | Pickup có thùng hàng riêng → `truck` |
| img_002 | Xe chở xe, 6–8 ô tô nằm trên khung | R-13 | 1 box `truck` bao cả khung chở + các xe; **không** box riêng cho xe được chở | Xe được chở là hàng hóa; người thấp ở cuối khung nếu ≥ 20 px thì gán `person` |
| img_003 | Người bán rong gánh (nón lá), xe đạp có giỏ, người dắt xe, xe máy nền | R-03, R-04, R-21 | `person` ôm người, không gồm gánh hàng; `bicycle` có giỏ gồm giỏ; xe máy nền theo kích cỡ | Gánh hàng không thuộc lớp |
| img_004 | Hai tuk-tuk xanh đỗ, tài xế ngồi trong khung hở, nón giao thông | R-15, R-21 | 2 `tuk_tuk`; tài xế thấy rõ ≥ đầu+vai → `person`; nón giao thông không gán | Tuk-tuk là lớp riêng; cabin hở |
| img_005 | 4 người chen trên xe máy, xe van xám phía sau, bị che nhiều | R-04, R-05 | `person` từng người (`occluded_level` theo che bởi người khác), `motorcycle` phần nhìn thấy, `van` (`occluded_level=1–2`) | Che nặng nhưng chắc lớp |
| img_006 | Cyclo, xe máy, xe đạp, taxi trắng, đường phố đông | R-04, R-09 | Xe gần gán đầy đủ, người nhỏ xa < 20 px → `ignore_region` | R-07 |
| img_007 | Nhà kho tối, xe Isuzu + pickup đèn sáng, vũng nước phản chiếu | R-11, R-22 | `truck` × 2; nước phản chiếu không vẽ | Xe đỗ vẫn gán |
| img_008 | Chợ đêm: hàng xe máy đỗ, người mặc áo phản quang ngồi xe | R-04, R-09, R-22 | Xe máy rõ → box; xe chồng chéo phía sau → `ignore_region`; người ngồi xe → `person` | Phép thử mục 1 |
| img_009 | Xe khách kem cỡ vừa, ngã tư Huế | R-17 | `bus` | ≥ 20 chỗ, hàng cửa sổ dài |
| img_010 | Người bán rau, xe máy, xe đẩy | R-04, R-21 | `person`, `motorcycle`; xe đẩy hàng rau không gán | Xe đẩy ngoài taxonomy |
| img_011 | Tượng Predator (hóa trang dạng tượng), xe moto trang trí kim loại, người xa | R-10, R-07 | Không vẽ Predator; moto trang trí dạng điêu khắc → không vẽ (`can_review` nếu có vẽ); người xa cao ≥ 20 px → `person`, < 20 px → `ignore_region` | Điêu khắc không phải xe thật |
| img_012 | Pickup thùng đầy hàng, xe Mazda xám phía xa | R-13, R-17 | `truck` gồm hàng; Mazda → `car` | Hàng gắn liền gộp box |
| img_013 | Đêm mưa, đường ướt phản chiếu, người xa, xe xa | R-11, R-07, R-18 | Không vẽ phản chiếu; vật < 20 px → ignore; vật đủ rõ ≥ 20 px → gán | Vật nhỏ, ánh sáng |
| img_014 | Hai người sắc nét, xe máy nhòe chuyển động | R-18 | 2 `person` rõ; vệt nhòe không xác định → `ignore_region` | Mất ranh giới |
| img_015 | Đêm mưa, hai xe cận cảnh bị cắt mép, xe xa nhòe | R-06, R-11, R-18 | `car` đỏ (trái) và `car` bạc (phải) `truncated=yes`; xe xa rõ hình → gán; phản chiếu mặt đường không vẽ | Cắt mép + phản chiếu |
| img_016 | Tuk-tuk (xe xanh lá, xe bạc trái), van trắng, Vespa + người, phụ nữ áo đỏ cắt mép | R-04, R-06, R-17 | `tuk_tuk`, `van`, `motorcycle`, `person` mỗi vật một box; người áo đỏ `truncated=yes` | Cảnh đủ lớp |
| img_017 | **Tượng** Công tước Wellington cưỡi ngựa, nón giao thông trên đầu, mòng biển | R-10 | **Không vẽ gì** | Tượng không phải người thật; ca kinh điển "IAA cao nhưng cùng sai" |
| img_018 | Biển quảng cáo có chân dung, xe đỗ sau hàng rào | R-10, R-05 | Không vẽ chân dung; xe sau hàng rào sắt → gán `car`, không tăng mức che vì hàng rào | Ảnh in; hàng rào xuyên thấu |
| img_019 | Xe Danger Mouse, đám đông phía sau hàng rào, linh vật chuột | R-12, R-09, R-10 | `car` (xe thật, `can_review`); người lái `person`; chuột không vẽ; khán giả dày → box từng người tách rõ, còn lại `ignore_region` | Xe giả tưởng nhưng thật |
| img_020 | Gương chiếu hậu: xe và cầu vồng trong gương, khung xe chụp | R-11, R-14 | **Không vẽ gì** | Phản chiếu + xe chủ |
| img_021 | Ngã tư Sài Gòn: dòng xe máy dày, người đội mũ bảo hiểm gần | R-09, R-04 | Xe + người gần → box; cụm xe xa → 1–2 `ignore_region` | Đám đông phương tiện |
| img_022 | Ảnh đen trắng cũ: xe máy gắn sidecar, người lái, đường rừng | R-16, R-07 | `motorcycle` một box gồm sidecar; người lái `person` nếu cao ≥ 20 px | Một vật nhiều phần |
| img_023 | Cyclo nhìn từ phía sau, Honda đen, xe máy xa | R-04, R-17 | `bicycle` (cyclo), `person` người đạp (hoặc khách nếu thấy), `car`, `motorcycle` nếu ≥ 20 px | Cyclo là xe đạp |
| img_024 | Pacer wagon đen, **xe vàng mô hình trên nóc**, xe cổ sau | R-10, R-17 | `car` Pacer; xe vàng không vẽ; xe cổ khác: `car` nếu đủ nhìn thấy | Mô hình |
| img_025 | Đại lộ bãi biển, người đi bộ xa, xe ba bánh | R-07, R-09 | Người ≥ 20 px → `person`; cụm nhỏ → `ignore_region`; xe ba bánh có cabin → `tuk_tuk` | Vật nhỏ |
| img_026 | Cặp đôi trên xe máy trong hầm, tranh tường cảng, mép dưới cắt xe | R-04, R-06, R-10 | `motorcycle` `truncated=yes`, 2 `person`; tranh tường (tàu, cảng) không vẽ | Hai người một xe |
| img_027 | **Bia mộ hình pickup** bằng đá | R-10 | **Không vẽ gì** | Không phải xe thật |
| img_028 | Pickup cổ, **tượng ngựa trắng** trong thùng | R-10, R-13 | `truck` (gồm thùng); tượng ngựa không vẽ; xe SUV đỏ xa → `car` | Tượng; pickup thật |
| img_029 | Xe máy chở ống dài nhô ra, người đi xe đạp bên cạnh | R-13 | `motorcycle` + `person`; phần ống nhô quá ½ thân xe không vào box; xe đạp → `bicycle` + `person` | Hàng dài mảnh |
| img_030 | Bãi xe máy dày đặc, áp phích chân dung, người mặc đồ xanh | R-09, R-10 | Xe rõ ở trước → box; khối chồng chéo → `ignore_region`; áp phích không vẽ | Đám đông xe |
| img_031 | Phố đêm Đài Bắc, xe, buýt, người đi bộ, nhiều phản chiếu | R-11, R-17, R-18 | `car`, `bus`, `person` vật rõ; phản chiếu đèn không vẽ | Ảnh đêm |
| img_032 | Hàng người chờ xe buýt, nón giao thông, Honda xanh trái cắt mép | R-09, R-06, R-21 | `person` rõ → box; khối dày → `ignore_region`; `car` xanh `truncated=yes`; nón không vẽ | Đám đông người |
| img_033 | Ma-nơ-canh, tủ kính, phản chiếu | R-10, R-11 | **Không vẽ gì** | Ma-nơ-canh + phản chiếu |
| img_034 | Người đi bộ băng đường, xe đen, xe máy | R-04 | `person` băng đường; `car` đen; xe máy xa theo R-07 | Cảnh đường thường |
| img_035 | Tuk-tuk vàng chất hàng cao, xe máy, `car` trắng | R-13, R-04, R-17 | `tuk_tuk` bao cả bao hàng cao; `motorcycle` + `person`; `car` trắng; nhiều xe chồng nhau → `occluded_level` | Hàng gắn liền gộp box |
| img_036 | Xe tải nhỏ xanh có bạt, xe máy đỗ hàng dài | R-17, R-22 | `truck`; xe máy xa theo R-07/R-09 | Thùng riêng |
| img_037 | Pickup trắng có nắp thùng, buýt xanh nền, người xa | R-17 | `truck` (gồm nắp thùng), `bus`, `person` nếu ≥ 20 px | Nắp thùng là phần của xe |
| img_038 | Tủ kính đồ chơi: xe mô hình, vịt cao su, chuột bông | R-10 | **Không vẽ gì** | Đồ chơi / mô hình |
| img_039 | Phố Huế xa, vài xe máy, người, biển hiệu | R-07, R-09 | Chỉ gán vật ≥ 20 px; vật nhỏ xa → `ignore_region` | Vật nhỏ |
| img_040 | Taxi xanh-đỏ, tuk-tuk, buýt, người đội mũ bảo hiểm gần camera | R-04, R-06, R-14 | `car` (taxi), `tuk_tuk`, `bus`; người cận cảnh `person` + `motorcycle` `truncated=yes` | Cận cảnh cắt mép |

> Ảnh ngoài bảng này vẫn áp dụng bảng 2.1 và R-01…R-23. Gặp tình huống chưa có ở đây thì mở mục 5 và làm theo quy trình ca mơ hồ.

---

## 5. QC, ca mơ hồ và phiên bản

### 5.1 Ngưỡng và cách đếm
- IoU ghép 1–1, `match_iou=0.5`; đạt khi IoU ≥ 0.85 (box cao ≥ 50 px) hoặc ≥ 0.7 (box nhỏ hơn).
- IAA: IoU TB ≥ 0.85 và cùng lớp ≥ 95% trên các **cặp ghép được**; luôn ghi dạng `tử số/mẫu số`.
- Lỗi theo **ảnh**: ảnh có ≥ 1 lỗi / ảnh đã duyệt; **≤ 2%** mới được Release. Với lô 40 ảnh, 1 ảnh lỗi (2,5%) đã vượt ngưỡng. Báo thêm lỗi theo **đối tượng** (annotation lỗi / annotation duyệt).
- Release yêu cầu **duyệt toàn bộ lô**; vòng lấy mẫu chỉ là demo → luôn **Hold** khi bàn giao.
- Giữ IAA **trước** calibration trong hồ sơ.

### 5.2 Năm loại lỗi trong phiếu QC
| `error_type` | Nghĩa | Rule hay dẫn |
|---|---|---|
| `bo_sot` | vật thật không có box (kể cả vật ≥ 20 px không nằm trong `ignore_region`) | R-07, R-08, R-09 |
| `thua` | box trên tượng, ảnh in, phản chiếu, đồ chơi | R-10, R-11 |
| `sai_lop` | gán sai lớp (van ↔ truck, bicycle ↔ motorcycle, bus ↔ van) | R-17 |
| `sai_vi_tri_kich_thuoc` | box lỏng, thiếu phần nhìn thấy, bao thêm đồ nhô | R-03, R-13 |
| `sai_thuoc_tinh` | `occluded_level` / `truncated` sai mức | 2.3 |

Phiếu `qc_review.csv`: `image, object_id, error_type, reviewer, rule_id, region, note`. Ghi `rule_id` mọi lỗi để thấy **lỗi lặp** theo quy tắc.

### 5.3 Phân biệt IAA thấp: sửa guideline hay sửa người?
| Dấu hiệu | Nguyên nhân | Xử lý |
|---|---|---|
| Cả hai người cùng lệch trên cùng loại ca | quy tắc mơ hồ | sửa guideline (tăng phiên bản) |
| Chỉ một người lệch | chưa nắm quy tắc | hướng dẫn lại, giữ guideline |
| Hai người **giống nhau** nhưng cùng sai (ví dụ cùng vẽ `person` lên tượng img_017) | cùng hiểu sai | đối chiếu ca tham chiếu của coach, sửa quy tắc |

Script **kiểm được**: tên file, lớp / thuộc tính hợp lệ, box trong ảnh, box quá nhỏ, đủ lượt gán, độ lệch giữa hai người. Script **không kiểm được**: box có ôm sát vật thật không, lớp có đúng thực tế không, vật cả nhóm cùng bỏ sót, cả nhóm cùng hiểu sai → cần mắt người đối chiếu với guideline và ảnh gốc.

### 5.4 Ai chốt ca mơ hồ, và chốt thế nào
1. Mỗi người gán độc lập theo guideline đã khóa của nhóm, chưa chắc thì `can_review` + ghi ca (R-20).
2. Họp calibration: mỗi ca bất đồng **một dòng** trong `calibration_decisions.csv` (`quyet_dinh`, `quy_tac_ghi_vao_guideline`, `phien_ban_guideline`, `nguoi_chot`, `phieu_sua`, `kiem_lai`, `pham_vi`).
3. **Người điều phối** (ghi trong `TEAM.md`) chốt khi nhóm không đồng ý; quyết định phải kèm lý do và mã quy tắc.
4. Ca là **quy tắc mới / đổi nghĩa** (đổi cách chọn vật, lớp, box, thuộc tính) → `kind=semantic` trong `guideline_change.json`, `pham_vi=lo`, phải kiểm lại cả 40 ảnh. Chỉ sửa câu chữ → `editorial`.
5. Người kiểm QC **khác** người sửa; nhóm 2 người đổi vai và ghi rõ.

### 5.5 Những lỗi dễ gặp nhất (để tự kiểm trước khi export)
1. Vẽ `person` lên tượng / ảnh in / ma-nơ-canh (R-10): img_017, 018, 033.
2. Vẽ box trên phản chiếu (R-11): img_020, 013, 031.
3. Vẽ box riêng cho xe đang được chở trên xe chở xe (R-13): img_002.
4. Quên người ngồi sau xe máy (R-04): img_005, 026.
5. Box gồm cả gánh hàng / hàng nhô dài (R-03, R-13): img_003, 029.
6. Vật < 20 px vẽ box thường (R-07): img_013, 021, 025, 039.
7. Nhầm `van` / `truck` / `bus` (R-17): img_005, 009, 037.
8. Quên `truncated=yes` ở vật chạm mép (R-06): img_015, 016, 026, 032, 040.
9. Dùng `ignore_region` thay cho "không vẽ" ở vật giả (R-08, R-10).
10. Box cách vật > 2 px (R-03).

### 5.6 Lịch sử phiên bản
| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu: 5 lớp + `ignore_region`, 3 thuộc tính, 3 quy tắc ngắn | giảng viên cung cấp |
| **v0 sửa đổi (316)** | Thêm lớp `bus`, `tuk_tuk`, `bicycle`; thêm 23 quy tắc R-01…R-23; định nghĩa ngưỡng `occluded_level` / `truncated`; chốt quy tắc vật không thật, phản chiếu, xe chở xe, người trên xe, vật nhỏ, đám đông, xe chủ | img_017, 018, 033, 038 (vật giả), img_002 (xe chở xe), img_005, 026 (người trên xe), img_021, 030, 032 (đám đông), img_004, 035, 016 (tuk-tuk), img_009, 031, 037 (bus), img_003, 006, 023 (cyclo/xe đạp) |
