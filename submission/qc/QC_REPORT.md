# Báo cáo QC — Nhóm T005

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và kiểm soát chất lượng (QC) dữ liệu bounding box 2D cho bài toán phát hiện đối tượng giao thông đường phố; giới hạn sử dụng trong nghiên cứu và huấn luyện mô hình thị giác máy tính với 40 ảnh phức tạp.
- Nguồn và phép sử dụng dữ liệu: Xem ATTRIBUTION.csv, ảnh thu thập từ Wikimedia Commons với giấy phép CC BY-SA 2.0 / CC BY 2.0.
- Manifest và checksum nhãn cuối: batch_manifest.json (SHA-256: 14ef207718eb444d256a2ba9aa5585fff10738167b68b9fb9cb9b71a116e5566).
- Guideline và taxonomy khi gán / khi bàn giao: Taxonomy v1.0 gồm 10 lớp; Guideline v1.0 thống nhất nhóm khi gán độc lập và Guideline v1.1 calibrated khi bàn giao (bổ sung quy tắc ca khó img_019).

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 120/120 (40 ảnh x 3 lượt gán độc lập bởi 5 thành viên).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 1209 cặp box ghép, IoU TB 0.957, đồng thuận lớp 96.4% (mẫu số: 1209 cặp box ghép).
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 205 box chưa ghép của A, 243 box chưa ghép của B; 0 bỏ qua do ignore; 125 ca bất đồng vùng ignore.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Ca img_019.jpg (xe diễu hành Danger Mouse và đám đông dính sát nhau), ban đầu có người vẽ box cá nhân lẻ tẻ đè lên nhau, sau đối chiếu chốt bôi trọn cụm đông người bằng ignore_region.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 1409 ca được phân loại và xử lý theo các quy tắc R-01 đến R-09, ghi trong calibration_decisions.csv, xác nhận bởi Reviewer 040.
- Tác động v1.1 lên nhãn và ảnh: Chuẩn hóa toàn bộ 40 ảnh về bản đồng thuận không còn cờ can_review, khoanh ignore_region cho đám đông chen chúc.
- Tham chiếu bằng chứng khi Rework: Không áp dụng vòng Rework (Vòng 1 đạt chuẩn Release).

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 040 (Nguyễn Đức Cường), 2026-11-17
- Đã duyệt / cả lô: 40/40 ảnh
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0%
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/589 = 0.0%
- Vật bỏ sót, ảnh và vùng: Không có (0 vật bỏ sót)
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Ca khó img_019.jpg, img_021.jpg, img_029.jpg, img_038.jpg đều tuân thủ đúng quy tắc; không có lỗi lặp.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Lỗi máy chặn được: toạ độ bbox vượt mép ảnh và lớp ngoài taxonomy; lỗi cần người đọc: phân biệt xe đồ chơi thu nhỏ img_038 với xe thật.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Chất lượng đạt chuẩn 100%, tỷ lệ lỗi 0.0% (< ngưỡng 2.0%), độ đồng thuận IAA 96.4% (IoU 0.957); bàn giao cho giảng viên và nhóm nghiên cứu mô hình phát hiện vật cản tự hành.
- Nếu còn việc: Không còn việc tồn đọng, toàn bộ dữ liệu đã đồng bộ.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca C001 (img_001.jpg, R-02, labels_calibrated.json), Ca img_019.jpg (R-08, vùng đám đông ignore_region, labels_calibrated.json).
