# Guideline gán nhãn — v0.341 (bản cá nhân, Pha 1) — thành viên 341

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện người và phương tiện thật trên ảnh 2D** (camera giao thông, xe tự lái / robotaxi).

Sai ở đâu thì nguy hiểm:
- **Gán box cho vật không thật** (tượng, ma-nơ-canh, đồ chơi, hình in, phản chiếu): mô hình học báo động giả, xe phanh gấp vì "người" trên billboard.
- **Bỏ sót vật thật** (người đi bộ, xe máy, xe ba bánh): nguy hiểm nhất vì là va chạm.
- **Box mỗi người vẽ một kiểu** (xe máy có gồm người không?): mô hình không hội tụ, IAA thấp.

Phép thử chung cho lớp và thuộc tính: *"mô hình có phải đoán điều này không?"* Nếu phải đoán phần không nhìn thấy trong ảnh thì quy tắc phải nói rõ có đoán hay không.

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa (một câu) | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô con 4 bánh chở người (sedan, hatchback, SUV, wagon) đang là **xe thật**. | img_002 (xe trên rơ-moóc là hàng, xem R-07), img_023 (Honda đen), img_024 (Pacer đen), img_040 (taxi) | Pickup (là `truck`); xe đồ chơi / mô hình (img_038); xe giả tưởng nhưng chạy thật, xem R-05 |
| `truck` | Xe có **thùng hàng riêng hoặc khoang chở hàng hở**: xe tải, đầu kéo, **pickup**. | img_002 (đầu kéo), img_007 (Isuzu), img_012, img_037 (pickup), img_036 (xe tải xanh), img_001 (pickup trắng) | Xe thùng kín cỡ nhỏ (là `van`); pickup bằng đá / tượng (img_027) |
| `van` | Xe thùng kín cỡ nhỏ chở người hoặc hàng, **thân liền một khối**. | img_005 (van trắng ở nền), img_016 (van trắng) | Minibus (xem R-06); xe tải nhỏ có thùng rời |
| `motorcycle` | Xe hai bánh có động cơ (xe máy, mô tô, xe có sidecar). **Chỉ phần xe**, không tính người ngồi trên. | img_006, img_008, img_021, img_022 (có sidecar), img_029 | Xe đạp, cyclo (không động cơ, xem R-06); xe máy kim loại điêu khắc (img_011, **[HỎI]**) |
| `person` | Người thật (đi bộ, ngồi trên xe, đứng bán hàng). | img_014, img_034 (người đi bộ rõ), img_003, img_026 (người ngồi xe) | Ma-nơ-canh (img_033), tượng (img_017), người in trên billboard (img_018), chuột bông (img_019) |
| `ignore_region` | Vùng **có vật thật nhưng không tách / không nhìn được từng cái** (đám đông, bãi xe dày, vùng nhòe). Bỏ sót ở đây không tính lỗi. | img_032 (đám đông ở trạm), img_021 (xa), img_030 (bãi xe), img_019 (đám khán giả) | Vật không thật (không phải ignore, xem R-01); vật xa nhưng tách được |

**Vật không thuộc lớp nào:** nếu là vật **thật** mà taxonomy v0 không có lớp (bus lớn, xe đạp, cyclo, xe ba bánh) thì ghi ảnh vào `image_issues.csv` và **không** nhét tạm vào lớp sai trừ khi có quy tắc ở R-06. Nếu là vật **không thật** thì không gán, không vẽ ignore (R-01).

### Thuộc tính

**`occluded_level`** (ước lượng theo tỉ lệ diện tích vật **bị che**, so với toàn vật nếu không bị che; người gán ước lượng bằng mắt, ngưỡng để tránh mỗi người một kiểu):

| Mức | Nghĩa | Ngưỡng |
|---|---|---|
| `0` | Thấy rõ | Che < 10% |
| `1` | Che ít | 10% – 40% |
| `2` | Che nhiều | > 40% – 80% |
| `3` | Gần như bị che hết, chỉ còn gợi ý | > 80% nhưng vẫn nhận ra được là vật gì (nếu không nhận ra thì không vẽ box) |

Nguồn che chỉ tính **vật khác** đè lên (người, xe, hàng rào, cột). Vật bị **cắt mép ảnh** thì dùng `truncated`, không tính vào `occluded_level`.

**`truncated`**: `yes` khi phần vật nằm ngoài khung ảnh ≥ 10% ước lượng (ví dụ img_015 hai xe ở mép, img_040 gương và người bị cắt, img_005 nhiều người bị cắt dưới). Còn lại `no`. Box vẽ sát mép ảnh, không vẽ ra ngoài.

**`can_review`**: tick khi một trong các điều kiện: (a) ca ngoài taxonomy mà tôi vẫn phải chọn lớp tạm, (b) chưa chắc là vật thật hay không, (c) không chắc mức `occluded_level` giữa hai bậc liền kề. Không tick tràn lan: tick quá nhiều làm mất ý nghĩa.

## 3. Quy tắc vẽ box

Mã `R-xx` để dẫn lại ở bảng ca khó và khi calibration.

**R-01 — Chỉ gán vật thật.** Tượng, ma-nơ-canh, đồ chơi, mô hình, hình in, tranh tường, billboard, phản chiếu **không gán lớp**, không vẽ ignore. Phép thử: nếu mô hình xe tự lái nhận là "người / xe" thì xe có phanh nhầm không? Có → không gán.

**R-02 — Người và xe máy: hai lớp, hai box, không gộp.**
- Mỗi người (kể cả hành khách) có **một box `person`** ôm phần người **nhìn thấy** (đầu, thân, chân nếu thấy).
- Xe máy có **một box `motorcycle`** ôm thân xe (bánh trước tới bánh sau, tay lái, yên); **không** kéo box giãn ra cho người ngồi. Hai box được **phép chồng nhau**.
- Cả hai người ngồi trên một xe: 2 box `person` + 1 box `motorcycle` (img_026: 2 + 1; img_008: người mặc áo phản quang là `person`).
- Xe máy không người, đỗ: vẫn `motorcycle` (img_008, img_030).
- Người đội mũ bảo hiểm: mũ thuộc người, vào box.

**R-03 — Vật mang theo / hàng hóa.**
- Đồ đeo, xách, đội sát người (balo, túi, nón, mũ): **vào** box người.
- Vật mang **nhô ra xa thân** hoặc dài hơn thân người / xe (gánh hàng img_003, ống dài img_029, hàng chất cao trên xe ba bánh img_035): **không** tính vào box. Box ôm phần người / xe, bỏ phần hàng nhô ra.
- Hàng chất trong thùng xe (img_012 thùng xe pickup chất đồ): box `truck` ôm cả thùng xe gồm hàng (hàng thuộc khoang xe).

**R-04 — Che khuất.** **Không đoán phần bị che.** Box chỉ ôm phần nhìn thấy, phần bị che không được mở rộng box. Gán `occluded_level` theo ngưỡng ở trên.

**R-05 — Xe lạ / giả tưởng nhưng chạy thật.** Phương tiện thật có động cơ, người lái, biển số (img_019 xe Dangermouse) được gán lớp gần nhất (`car`) và tick `can_review`. Người ngồi trong vẫn là `person`. Đồ chơi / nhân vật bông ngồi cạnh (chuột trắng) **không** gán (R-01).

**R-06 — Vật thật ngoài taxonomy v0** (tạm xử lý, chờ nhóm chốt ở Pha 2).
| Vật | Ảnh | Quyết định ban đầu |
|---|---|---|
| Xe 3 bánh có động cơ (tuk-tuk, auto rickshaw) | img_004, img_016, img_035, img_040 | **[HỎI]** Tạm `car` + `can_review`. Lý do: kích thước, có cabin, có người ngồi trong. Nếu nhóm thêm lớp mới thì sửa lại. |
| Bus / minibus | img_009, img_031, img_037, img_040 | Minibus cỡ nhỏ (img_009) tạm `van` + `can_review`. Bus lớn: ghi `image_issues.csv`, **không** gán lớp khác. |
| Xe đạp, cyclo (không động cơ) | img_003, img_006, img_009, img_023, img_029, img_034, img_039 | Không gán phương tiện. **Người** ngồi / đạp vẫn là `person`. Ghi `image_issues.csv`. |

**R-07 — Xe chở xe (xe tải chở ô tô).** Img_002: **một box `truck`** ôm đầu kéo + rơ-moóc; ô tô nằm trên rơ-moóc là **hàng**, không có box `car` riêng. Lý do: mô hình thấy một khối phương tiện đang di chuyển, không phải 5 ô tô chạy riêng. **[HỎI]** — nhóm có thể thấy mỗi ô tô cần box riêng.

**R-08 — Xe đặt lên xe (xe giả hoặc xe thật chở trên mui).** Img_024: xe vàng trên mui xe wagon đen. Xe đen là `car`. Xe vàng là vật trang trí / mô hình trên mui, **không** gán (R-01). **[HỎI]**.

**R-09 — Phản chiếu.** Mặt đường ướt (img_001, 013, 015, 031), kính cửa hàng (img_033, 038), gương chiếu hậu (img_020): hình phản chiếu **không gán**. Gương ego-vehicle chỉ là một phần xe chủ (R-10).

**R-10 — Xe chủ (ego).** Phần xe của chính camera (img_020 cửa, gương; có thể thấy ở mép ảnh): không gán.

**R-11 — Vật nhỏ, xa.** Box cao < **20 px** (theo `rules.min_box_height_px`) không gán từng vật. Nếu là cụm khó tách → R-12. Nếu là vật lẻ rất nhỏ → bỏ qua, không vẽ ignore (ví dụ người xa ở img_025).

**R-12 — Đám đông / bãi xe dày.**
- Chỉ vẽ `ignore_region` khi **≥ 8 người hoặc xe chồng lấn nhau** và **không tách được** từng vật bằng mắt (img_032 trạm xe, img_021 dòng xe máy xa, img_030 bãi xe, img_019 khán giả sau hàng rào).
- Vật **ở gần, rõ** trong cùng ảnh vẫn vẽ box riêng. Ignore region chỉ phủ **vùng không tách được**, không phủ lên vật đã có box.
- Vật mờ do chuyển động (img_014 xe máy ở nền) → `ignore_region`, người rõ ở trước vẫn là `person`.

**R-13 — Box ôm sát.** Box ôm sát **phần nhìn thấy**, sai số cho phép 2–3 px; không cố ý chừa lề. Không vẽ box qua một mép khác của vật. Bóng đổ và phản chiếu dưới xe không tính.

**R-14 — Ảnh không có vật hợp lệ.** Ảnh không có vật nào cần box (img_017, img_020, img_027, img_033, img_038 nếu tất cả bị loại theo R-01) vẫn phải có mặt trong COCO với danh sách box rỗng.

## 4. Ca khó

| Ảnh     | Tình huống                                                               | Rule ID          | Quyết định (của 341)                                                                                        |
| ------- | ------------------------------------------------------------------------ | ---------------- | ----------------------------------------------------------------------------------------------------------- |
| img_002 | Xe chở xe: đầu kéo + 5 ô tô trên rơ-moóc                                 | R-07             | 1 box `truck` ôm cả đoàn, không box từng ô tô                                                               |
| img_003 | Người gánh hàng; xe đạp; hàng xóm xe máy ở nền                           | R-03, R-06, R-12 | `person` ôm người, bỏ phần gánh nhô ra; xe đạp không gán; xe máy nền xa ≥ 8 chiếc → `ignore_region`         |
| img_004 | Hai tuk-tuk đỗ cạnh nhau, tài xế ngồi trong, cọc tiêu                    | R-06, R-02       | 2 box `car` + `can_review`; mỗi tài xế 1 box `person`; cọc tiêu không gán                                   |
| img_005 | Nhiều người đội mũ, chồng nhau, cắt mép; van trắng ở nền                 | R-02, R-04       | Mỗi người một `person`, `occluded_level` 1–3 tùy người, `truncated=yes` ở mép dưới; van = `van`             |
| img_006 | Nhiều xe máy và người đi từ sau, xe đạp                                  | R-02, R-06       | Mỗi xe máy 1 box, mỗi người 1 box; xe đạp không gán                                                         |
| img_007 | Pickup (Hilux) phía sau, xe tải Isuzu phía trước, trong kho tối          | R-04             | Pickup `truck` (che 1), Isuzu `truck`                                                                       |
| img_008 | Hàng xe máy đỗ và người mặc áo phản quang số 7, 4                        | R-02, R-12       | Mỗi xe `motorcycle`; xe nền dày không tách → `ignore_region`; người mặc áo là `person`                      |
| img_009 | Minibus giữa phố; xe máy nhỏ ở xa                                        | R-06, R-11       | `van` + `can_review`; xe máy xa cao < 20 px thì bỏ                                                          |
| img_011 | Nhân vật Predator hóa trang (tượng / hóa trang?), xe máy kim loại tự chế | R-01, R-05       | Predator: `person` nếu là người mặc đồ, `can_review`; xe kim loại: `motorcycle` nếu chạy được, `can_review` |
| img_012 | Pickup chất hàng + ô tô ở nền                                            | R-03, R-04       | `truck` ôm cả thùng + hàng; ô tô xám `car` (che 1)                                                          |
| img_014 | Hai người băng đường rõ, xe nhòe chuyển động ở sau                       | R-12, R-04       | 2 `person` (che 0); vùng nhòe → `ignore_region`                                                             |
| img_015 | Xe đỏ, xe bạc bị cắt mép, phản chiếu mặt đường ướt                       | R-09, truncated  | `car` `truncated=yes` cho từng xe; phản chiếu không gán                                                     |
| img_017 | Tượng người cưỡi ngựa, nón cảnh sát trên đầu                             | R-01             | Không gán                                                                                                   |
| img_018 | Billboard in mặt người; xe đỗ sau hàng rào                               | R-01, R-04       | Mặt billboard không gán; xe `car` `occluded_level` 1–2                                                      |
| img_019 | Xe đồ chơi Dangermouse chạy thật, chuột bông, đám đông sau hàng rào      | R-05, R-01, R-12 | `car` + `can_review`; người lái `person`; chuột bông không gán; khán giả → `ignore_region`                  |
| img_020 | Ảnh trong gương chiếu hậu: xe và cầu vồng phản chiếu                     | R-09, R-10       | Không gán; ảnh trả về danh sách rỗng                                                                        |
| img_022 | Xe có sidecar, ảnh cũ trắng đen, xe nhỏ                                  | R-02, R-11       | 1 `motorcycle` ôm cả xe + sidecar; người trên xe 1–2 `person` nếu cao ≥ 20 px                               |
| img_024 | Xe vàng đặt trên mui xe wagon đen                                        | R-08, R-01       | Xe đen `car`; xe vàng không gán                                                                             |
| img_026 | Cặp đôi trên xe máy; tranh tường có tàu                                  | R-02, R-01       | 1 `motorcycle` + 2 `person`; tranh tường không gán                                                          |
| img_027 | Mộ hình xe pickup bằng đá đen                                            | R-01             | Không gán                                                                                                   |
| img_028 | Pickup cổ thật; tượng ngựa trắng trong thùng                             | R-01, R-03       | Pickup `truck`; ngựa tượng không gán (không tính ngựa vào box nếu nhô ra khỏi thùng)                        |
| img_029 | Xe máy và xe đạp chở ống dài nhô ra                                      | R-03, R-02, R-06 | Mỗi người 1 `person`; 1 `motorcycle`; xe đạp không gán; ống không tính vào box                              |
| img_030 | Bãi xe máy chồng chất, người bảo vệ đứng                                 | R-12, R-04       | Xe gần rõ vẽ `motorcycle`; vùng dày → `ignore_region`; `person` bảo vệ 1 box                                |
| img_032 | Đám đông ở trạm xe buýt, ô tô gần                                        | R-12             | Ô tô xanh gần `car` (`truncated`); cả đám đông → `ignore_region`                                            |
| img_033 | Ma-nơ-canh sau kính, phản chiếu xe                                       | R-01, R-09       | Không gán                                                                                                   |
| img_035 | Xe ba bánh chở hàng khổng lồ, nhiều xe khác                              | R-03, R-06       | Mỗi xe ba bánh `car` + `can_review`, box không gồm hàng nhô quá thân; xe máy, ô tô khác vẽ như thường       |
| img_037 | Pickup có nắp thùng, bus ở nền, người đi bộ                              | R-06             | Pickup `truck`; bus lớn bỏ + ghi issue; người đi bộ `person`                                                |
| img_038 | Tủ trưng bày đồ chơi, xe mô hình                                         | R-01             | Không gán                                                                                                   |
| img_040 | Taxi xanh, người đội mũ che phần lớn, tuk-tuk, bus                       | R-06, R-04       | Taxi `car` (che 2, `truncated`); người `person`; tuk-tuk `car` `can_review`; bus bỏ                         |

## 5. QC và phiên bản

- **Ngưỡng áp dụng** (từ `submission/taxonomy.json`, không đổi): box cao tối thiểu 20 px; IoU ghép cặp 0.5; IoU đồng thuận 0.85 (vật lớn ≥ 50 px) / 0.7 (vật nhỏ); đồng thuận lớp 0.95; tỉ lệ ảnh lỗi tối đa 2%.
- **Ai quyết ca mơ hồ:** tick `can_review`, người điều phối chốt cùng nhóm ở Pha 2 và Pha 5; không tự đổi quyết định khi đã khóa v1.0.
- **Quy tắc tăng phiên bản:** đổi cách chọn vật, lớp, box, thuộc tính → `semantic` → kiểm lại cả lô; đính chính câu chữ → `editorial`.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản giảng viên | — |
| v0.341 | Thêm R-01 … R-14, định nghĩa lớp, ngưỡng `occluded_level` và `truncated`, 29 ca khó | v0 thiếu: người trên xe máy, che khuất, vật không thật, vật ngoài taxonomy, xe chở xe |

## Phụ lục: chỗ v0 làm hai người hiểu khác nhau

1. "Gán box cho mọi người và phương tiện": người ngồi xe máy là một box hay hai? Có tính hành khách? (R-02)
2. "Box ôm sát vật": sát phần thấy hay phần suy ra? Hàng nhô ra có tính không? (R-03, R-04)
3. "Ô tô con", "xe tải", "xe thùng kín cỡ nhỏ": pickup, tuk-tuk, minibus, bus thuộc đâu? (R-06)
4. `occluded_level` 1, 2, 3 chỉ có từ ("ít", "nhiều", "không rõ") nên mỗi người đặt mốc khác nhau: đã thêm ngưỡng %.
5. `ignore_region`: v0 không nói vật bao nhiêu thì thành "vùng", và tượng / hình in có thuộc vùng này không (R-01, R-12).
6. Không có quy tắc cho vật không thật, phản chiếu, xe chủ, vật nhỏ.
