# Thông tin nhóm

- Tên nhóm: Robotaxi
- Mã thành viên (ba chữ số): 231, 316, 323, 336, 341
- Người điều phối: 231
- Người review bản cuối: 316
- Commit chốt bài (điền ở kênh nộp sau khi commit):

## Kết quả tóm tắt (điền cuối bài)

- Lượt gán hoàn tất / được giao: 120/120 (K=3, 24 ảnh/người)
- IAA trước calibration (kèm mẫu số): IoU TB 0.811, đồng thuận lớp 98.0% trên 882 cặp box ghép → chưa đạt (ngưỡng IoU 0.85)
- Số ảnh phải calibration: 35 / 40 bị cờ; v1.1 là semantic nên rà lại cả 40 / 40
- Vòng QC và số ảnh đã duyệt / cả lô: vòng 2, 40/40 (vòng 1: 40/40, Rework)
- Tỉ lệ lỗi: vòng 2 là 0.0% theo ảnh (0/40), 0.0% theo đối tượng (0/573); vòng 1 là 17.5% (7/40), 1.2% (7/578)
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: Release (theo qc_report.py vòng 2; giới hạn: R-15, R-11 chưa đo lại, ảnh đông chỉ xem tổng thể, chưa có người ngoài nhóm kiểm)
