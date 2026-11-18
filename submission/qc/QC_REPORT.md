# Báo cáo QC — Robotaxi

> **BẢN NHÁP.** Mục 1–3 đã điền từ artifact do script sinh. Các chỗ `<<...>>` chỉ điền **sau khi** nhóm thực sự sửa nhãn, chạy `build_final.py` và duyệt vòng QC. Không điền số chưa đo.

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và QC nhãn box 2D (người, phương tiện) cho detector camera robotaxi. Lô chỉ có 40 ảnh đường phố, chủ yếu đô thị châu Á, ban ngày, nên không đại diện cho đêm, mưa hay đường cao tốc. Chỉ dùng cho luyện tập quy trình, không dùng huấn luyện sản phẩm.
- Nguồn và phép sử dụng dữ liệu: Theo `data/ATTRIBUTION.csv` (giữ nguyên ghi công từng ảnh). Không thêm ảnh hay thông tin cá nhân.
- Manifest và checksum nhãn cuối: <<batch_manifest.json và SHA-256 của labels_calibrated.json, sau khi chạy build_final.py>>
- Guideline và taxonomy khi gán / khi bàn giao: Gán theo `guideline_v1.0_agreed.md` và `taxonomy.json` v1.0. Bàn giao theo `guideline_v1.1_calibrated.md`, thay đổi `semantic` (xem `submission/guideline_change.json`). Taxonomy giữ v1.0, mục `rules` không đổi.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 120/120 (5 người 231, 316, 323, 336, 341; K=3 mỗi ảnh; mỗi người 24 ảnh). `validate_labels.py --all` sạch, chỉ có 3 cảnh báo vật nhỏ trong file 323 (img_021, img_031, img_039). Số liệu dưới đây tính trên bản nhãn đã khóa ở commit 28aeb62 (231 đã export lại trước khi khóa IAA).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: Mẫu số **882 cặp box ghép** (IoU ≥ 0.5, ghép 1-1, cộng trên 10 cặp người gán × ảnh chung = 120 lượt cặp-ảnh). IoU TB **0.811** (ngưỡng 0.85, **chưa đạt**). Đồng thuận lớp **98.0%** (ngưỡng 95%, đạt). Cùng `occluded_level` 52.2%, cùng `truncated` 90.9%. Chỉ 2/10 cặp đạt (231|316 0.872, 316|323 0.852). Nguồn: `submission/qc/iaa_summary.md`.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 334/330 box chưa ghép (phía A/B), 0 box loại do ignore, 75 ca `bat_dong_ignore`. Tổng 1624 ca trên 35/40 ảnh: `bo_sot_thua` 645, `sai_thuoc_tinh` 457, `iou_thap` 427, `bat_dong_ignore` 75, `sai_lop` 18, `co_can_review` 2. Tập trung ở ảnh đông xe máy: img_021 (211 ca), img_019 (150), img_032 (133), img_003 (98). Số box (không tính ignore) mỗi người chênh nhiều: 323 có 311, 231 có 276, 341 có 265, 316 có 184, 336 có 178.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: **img_002**: cả 3 người được giao (231, 336, 341) đều vẽ 1 `truck` + 8 `car` cho xe chở ô tô, trái R-07 v1.0 (một box `truck`, không gán `car` riêng). IAA không cờ ca này vì 3 người giống nhau. Phát hiện bằng cách đếm lớp theo ảnh trong từng file `labels/<mã>.json` rồi so với bảng ca khó mục 5 của guideline. Các ảnh rỗng theo R-14 (img_017, img_020, img_027, img_033, img_038) thì đã kiểm: 0 box ở mọi người. Xem ảnh chồng nhãn còn thấy thêm hai ca: **img_019**, cả 323, 336, 341 đều không gán xe diễu hành màu vàng (R-13 ghi là `car`), ca R002; **img_037**, đa số sai, 336 và 341 gán `van` cho pickup có nắp thùng, chỉ 316 đúng `truck`, ca R003. Như vậy bỏ phiếu đa số 2/3 không thay được việc đọc ảnh. <<bổ sung kết quả đối chiếu ca tham chiếu coach cung cấp>>

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: `submission/qc/calibration_decisions.csv` có 1627 dòng (1624 ca IAA và 3 ca cùng sai/đa số sai: R001 img_002, R002 img_019, R003 img_037). Mỗi dòng đã có đề xuất `quyet_dinh` và rule: R-17 556, R-02 427, R-15 422, R-11 159, R-05 35, R-16 15, R-06 5, R-03 4, R-13 2, R-10 1, R-07 1. <<nhóm chốt từng dòng: nguoi_chot, phieu_sua, kiem_lai>>
- Tác động v1.1 lên nhãn và ảnh: Thay đổi `semantic` (R-11 viết lại, thêm R-15, R-16, R-17; R-07 chờ chốt) nên phạm vi là **cả lô 40 ảnh** (các dòng `pham_vi=lo`). Không đổi taxonomy nên không cần mapping lớp. <<đường dẫn labels_calibrated_<phiên-bản>.json do prepare_calibration.py in ra>>
- Tham chiếu bằng chứng khi Rework: <<nếu có vòng 2: vòng cũ, vùng sửa, vòng mới>>

## 4. Kết quả duyệt

- Vòng QC: <<1>>
- Người review và ngày: <<mã, ngày>>
- Đã duyệt / cả lô: <<…/40>>
- Ảnh lỗi / ảnh đã duyệt: <<theo qc_result.json>>
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: <<theo qc_result.json>>
- Vật bỏ sót, ảnh và vùng: <<từ qc_review.csv, error_type=bo_sot>>
- Ca khó / phần còn lại, lỗi lặp và ví dụ: <<dẫn qc_review.csv của vòng>>
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy chặn được: box cao < 20 px (cảnh báo vật nhỏ của `validate_labels.py` ở img_021, img_031, img_039) và độ lệch giữa người gán (IoU, lớp, thuộc tính). Cần người đọc ảnh: img_002, cả 3 người cùng vẽ `car` trên xe chở xe trái R-07. Script không cờ vì 3 người đồng thuận; chỉ người đối chiếu guideline với ảnh mới thấy.

## 5. Quyết định bàn giao

- Quyết định: <<Release hoặc Hold hoặc Rework, theo qc_report.py và người kiểm xác nhận>>
- Lý do, người nhận và phạm vi sử dụng: <<>>
- Nếu còn việc: <<việc, người xử lý, bằng chứng cần kiểm lại>>
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: <<ví dụ: R001 → img_002 → R-07 → labels_calibrated.json; kết quả do người ngoài nhóm kiểm>>
