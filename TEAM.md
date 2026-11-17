# Thông tin nhóm

- Tên nhóm: T005
- Mã thành viên (ba chữ số): 040, 183, 192, 216, 240 (tương ứng 101, 102, 103, 104, 105)
- Người điều phối: Nguyễn Bá Duy (192 / 2A202603192)
- Người review bản cuối: Nguyễn Đức Cường (040 / 2A202603040)
- Commit chốt bài (điền ở kênh nộp sau khi commit):
- Repo bài nộp (URL): https://github.com/basduy05/K5-L23-Day07-T005-Guideline-Taxonomy-QC
- Số thành viên: 5  → số người gán mỗi ảnh (K): 3

| STT | Mã (3 số) | MSHV | Họ và tên | Lớp | Phụ trách thêm |
|---|---|---|---|---|---|
| 1 | 040 | 2A202603040 | NGUYỄN ĐỨC CƯỜNG | L2/L3 | Review bản cuối, viết ca khó guideline |
| 2 | 183 | 2A202603183 | HOÀNG NHẬT ANH | L2/L3 | Quản lý tài liệu, kiểm thử script |
| 3 | 192 | 2A202603192 | NGUYỄN BÁ DUY | L2/L3 | Điều phối viên (Leader), CVAT setup, chia task |
| 4 | 216 | 2A202603216 | ĐỖ HUY KIÊN | L2/L3 | Hỗ trợ QC, kiểm tra định dạng COCO |
| 5 | 240 | 2A202603240 | PHẠM MẠNH HÙNG | L2/L3 | Hiệu chuẩn nhãn, rà soát ca lệch |

## Kết quả tóm tắt (điền cuối bài)

- Lượt gán hoàn tất / được giao: 120 / 120 (40 ảnh x 3 lượt)
- IAA trước calibration (kèm mẫu số): IoU TB = 0.957, Cùng lớp = 96.4% (mẫu số: 1209 cặp box ghép)
- Số ảnh phải calibration: 40 / 40
- Vòng QC và số ảnh đã duyệt / cả lô: Vòng 1, đã duyệt 40 / 40 ảnh
- Tỉ lệ lỗi: 0.0% theo ảnh (0/40), 0.0% theo đối tượng (0/589)
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: **RELEASE** — Cả 40 ảnh đạt chuẩn kiểm định QC vòng 1 (0 lỗi / 40 ảnh, 0 lỗi / 589 đối tượng), giải quyết triệt để 1409 ca lệch theo quy tắc thống nhất.

