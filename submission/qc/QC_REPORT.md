# Báo cáo QC — Robotaxi

> Báo cáo điền theo kết quả vòng QC 1 (Rework) và vòng QC 2 (Release theo `qc_report.py`). Mọi số lấy từ `iaa_summary.md`, `qc_result.json` và `qc_review.csv`.

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và QC nhãn box 2D (người, phương tiện) cho detector camera robotaxi. Lô chỉ có 40 ảnh đường phố, chủ yếu đô thị châu Á, ban ngày, nên không đại diện cho đêm, mưa hay đường cao tốc. Chỉ dùng cho luyện tập quy trình, không dùng huấn luyện sản phẩm.
- Nguồn và phép sử dụng dữ liệu: Theo `data/ATTRIBUTION.csv` (giữ nguyên ghi công từng ảnh). Không thêm ảnh hay thông tin cá nhân.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json`; SHA-256 của `labels_calibrated.json` là `4cd74e880edeec983c9f9b193291a0733dfe2e87e9f96511335ef2137441c91d` (577 box, 40 ảnh)
- Guideline và taxonomy khi gán / khi bàn giao: Gán theo `guideline_v1.0_agreed.md` và `taxonomy.json` v1.0. Bàn giao theo `guideline_v1.1_calibrated.md`, thay đổi `semantic` (xem `submission/guideline_change.json`). Taxonomy giữ v1.0, mục `rules` không đổi.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 120/120 (5 người 231, 316, 323, 336, 341; K=3 mỗi ảnh; mỗi người 24 ảnh). `validate_labels.py --all` sạch, chỉ có 3 cảnh báo vật nhỏ trong file 323 (img_021, img_031, img_039). Số liệu dưới đây tính trên bản nhãn đã khóa ở commit 28aeb62 (231 đã export lại trước khi khóa IAA).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: Mẫu số **882 cặp box ghép** (IoU ≥ 0.5, ghép 1-1, cộng trên 10 cặp người gán × ảnh chung = 120 lượt cặp-ảnh). IoU TB **0.811** (ngưỡng 0.85, **chưa đạt**). Đồng thuận lớp **98.0%** (ngưỡng 95%, đạt). Cùng `occluded_level` 52.2%, cùng `truncated` 90.9%. Chỉ 2/10 cặp đạt (231|316 0.872, 316|323 0.852). Nguồn: `submission/qc/iaa_summary.md`.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 334/330 box chưa ghép (phía A/B), 0 box loại do ignore, 75 ca `bat_dong_ignore`. Tổng 1624 ca trên 35/40 ảnh: `bo_sot_thua` 645, `sai_thuoc_tinh` 457, `iou_thap` 427, `bat_dong_ignore` 75, `sai_lop` 18, `co_can_review` 2. Tập trung ở ảnh đông xe máy: img_021 (211 ca), img_019 (150), img_032 (133), img_003 (98). Số box (không tính ignore) mỗi người chênh nhiều: 323 có 311, 231 có 276, 341 có 265, 316 có 184, 336 có 178.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: **img_002**: cả 3 người được giao (231, 336, 341) đều vẽ 1 `truck` + 8 `car` cho xe chở ô tô, trái R-07 v1.0 (một box `truck`, không gán `car` riêng). IAA không cờ ca này vì 3 người giống nhau. Phát hiện bằng cách đếm lớp theo ảnh trong từng file `labels/<mã>.json` rồi so với bảng ca khó mục 5 của guideline. Các ảnh rỗng theo R-14 (img_017, img_020, img_027, img_033, img_038) thì đã kiểm: 0 box ở mọi người. Xem ảnh chồng nhãn còn thấy thêm hai ca: **img_019**, cả 323, 336, 341 đều không gán xe diễu hành màu vàng (R-13 ghi là `car`), ca R002; **img_037**, đa số sai, 336 và 341 gán `van` cho pickup có nắp thùng, chỉ 316 đúng `truck`, ca R003. Như vậy bỏ phiếu đa số 2/3 không thay được việc đọc ảnh. Chưa đối chiếu với ca tham chiếu của coach vì repo chưa có file đó.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: `submission/qc/calibration_decisions.csv` có 1627 dòng (1624 ca IAA và 3 ca cùng sai/đa số sai: R001 img_002, R002 img_019, R003 img_037). Mỗi dòng đã có đề xuất `quyet_dinh` và rule: R-17 556, R-02 427, R-15 422, R-11 159, R-05 35, R-16 15, R-06 5, R-03 4, R-13 2, R-10 1, R-07 1. Sau QC vòng 1 thêm 7 ca Q01–Q07 (người chốt 316), nên bảng có 1634 dòng. R001–R003 do 231 chốt; 1624 ca còn lại giữ theo bản đồng thuận (người chốt 231). Cột `phieu_sua` ghi rõ phần chưa chấm lại bằng mắt, `kiem_lai` ghi 316 xem overlay ở mức tổng thể.
- Tác động v1.1 lên nhãn và ảnh: Thay đổi `semantic` (R-11 viết lại, thêm R-15, R-16, R-17; R-07 chốt ở R001: xe chở xe gán 1 `truck` và mỗi xe nhìn rõ một `car`) nên phạm vi là **cả lô 40 ảnh** (các dòng `pham_vi=lo`). Không đổi taxonomy nên không cần mapping lớp. COCO sau sửa: `submission/qc/labels_calibrated_b2af32795fd1cdf2.json`. Nhãn sau sửa không qua CVAT (mất dữ liệu CVAT): sinh bằng script từ `consensus_draft.json`, bỏ box person 960x288 ở img_019 (R002), tắt `can_review`, rồi áp 7 sửa Q01–Q07. R-15 (`occluded_level`) và R-11 (`ignore_region`) chỉ giữ giá trị đa số của bản đồng thuận, chưa đo lại theo mốc. Một box ở img_005 trong `consensus_draft.json` vượt biên ảnh 0.01 px do làm tròn; đã hạ chiều cao 0.01 px và cập nhật hash trong `iaa_summary.json`, `calibration_context.json`.
- Tham chiếu bằng chứng khi Rework: Vòng 1 (Rework, `reviews/round-01/`): 7/40 ảnh lỗi (17.5%), 7/578 box. Vùng sửa: img_016 (#194 bus→truck), img_019 (#244 thừa), img_022 (#341 thừa), img_023 (#342 three_wheeler→bicycle), img_024 (#364 mô hình xe), img_035 (#528 trùng), img_037 (#549 bus thừa). Vòng 2 (`reviews/round-02/`) duyệt lại sau khi sửa.

## 4. Kết quả duyệt

- Vòng QC: 2
- Người review và ngày: 316, 2026-11-18
- Đã duyệt / cả lô: 40/40
- Ảnh lỗi / ảnh đã duyệt: vòng 2 là 0/40 = 0.0%; vòng 1 là 7/40 = 17.5%
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: vòng 2 là 0/573 = 0.0%; vòng 1 là 7/578 = 1.2%
- Vật bỏ sót, ảnh và vùng: không có (0 dòng `bo_sot` ở cả hai vòng)
- Ca khó / phần còn lại, lỗi lặp và ví dụ: vòng 1: 4 lỗi `thua` (img_022, img_024, img_035, img_037), 2 `sai_lop` (img_016, img_023), 1 `sai_vi_tri_kich_thuoc` (img_019); các lỗi lặp là box thừa và nhầm lớp xe. Ảnh đông (003, 006, 008, 019, 021, 030, 032, 034) chỉ được xem tổng thể, không kiểm từng box.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy chặn được: box cao < 20 px (cảnh báo vật nhỏ của `validate_labels.py` ở img_021, img_031, img_039) và độ lệch giữa người gán (IoU, lớp, thuộc tính). Cần người đọc ảnh: img_002, cả 3 người cùng vẽ `car` trên xe chở xe trái R-07. Script không cờ vì 3 người đồng thuận; chỉ người đối chiếu guideline với ảnh mới thấy.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Theo `qc_report.py --round 2` (0/40 ảnh lỗi; vòng 1 giữ nguyên làm bằng chứng). Lý do: sau khi sửa 7 lỗi vòng 1 không còn lỗi ghi nhận. Phạm vi: luyện tập quy trình QC, không dùng huấn luyện sản phẩm. Giới hạn: người duyệt (316) không độc lập hoàn toàn với người sửa nhãn và có công cụ AI hỗ trợ; R-15 và R-11 chưa đo lại; ảnh đông chưa kiểm từng box.
- Nếu còn việc: Một người ngoài 316 xem lại 8 ảnh đông và img_022 (hai box motorcycle, đã xóa một cái; cần xác nhận bằng mắt); chấm lại `occluded_level` theo R-15; chưa có kiểm ngược bởi người ngoài nhóm.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: R001 → img_002 → R-07 → `labels_calibrated.json` (1 `truck` + 8 `car`); Q07 → img_037 → R-16 → bỏ box `bus` thừa. Chưa có người ngoài nhóm kiểm.
