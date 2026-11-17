# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 5
- Lượt gán đủ **120/120**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 882 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 120 | 882 | 0.811 | 98.0% | 52.2% | 90.9% | 334/330 | 0 | CHƯA ĐẠT |
| 231|316 | 13 | 88 | 0.872 | 100.0% | 53.4% | 95.5% | 27/10 | 0 | ĐẠT |
| 231|323 | 12 | 149 | 0.815 | 100.0% | 34.2% | 89.3% | 32/37 | 0 | CHƯA ĐẠT |
| 231|336 | 12 | 86 | 0.778 | 100.0% | 67.4% | 88.4% | 63/11 | 0 | CHƯA ĐẠT |
| 231|341 | 11 | 86 | 0.820 | 100.0% | 59.3% | 94.2% | 21/29 | 0 | CHƯA ĐẠT |
| 316|323 | 13 | 98 | 0.852 | 100.0% | 45.9% | 88.8% | 17/42 | 0 | ĐẠT |
| 316|336 | 11 | 68 | 0.799 | 97.1% | 55.9% | 94.1% | 18/13 | 0 | CHƯA ĐẠT |
| 316|341 | 11 | 64 | 0.840 | 93.8% | 67.2% | 100.0% | 5/35 | 0 | CHƯA ĐẠT |
| 323|336 | 11 | 63 | 0.767 | 93.7% | 50.8% | 84.1% | 102/17 | 0 | CHƯA ĐẠT |
| 323|341 | 12 | 91 | 0.811 | 97.8% | 38.5% | 83.5% | 40/48 | 0 | CHƯA ĐẠT |
| 336|341 | 14 | 89 | 0.741 | 93.3% | 67.4% | 94.4% | 9/88 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 1624 ({'bo_sot_thua': 645, 'iou_thap': 427, 'sai_thuoc_tinh': 457, 'bat_dong_ignore': 75, 'co_can_review': 2, 'sai_lop': 18}), trên 35 ảnh cần calibration.
