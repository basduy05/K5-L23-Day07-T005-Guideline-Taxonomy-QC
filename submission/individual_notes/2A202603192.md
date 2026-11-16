# Ghi chú cá nhân — 2A202603192 - Nguyễn Bá Duy

- Thành viên: Nguyễn Bá Duy (MSHV: 2A202603192, Mã thành viên: 192 / Nhóm T005)
- Vai trò trong nhóm: Điều phối viên (Leader), phụ trách chia ảnh, thiết lập pipeline CVAT và tổng hợp báo cáo QC.

---

## 1. Pha 1: Ba ca khó tôi tìm ra trong 40 ảnh

| Ảnh | v0 không trả lời được gì | Quyết định ban đầu của tôi |
|---|---|---|
| `img_020.jpg` | Guideline v0 không đề cập đến hiện tượng ảnh phản chiếu trong gương chiếu hậu (rearview mirror) của xe buồng lái. | **Không gán box** cho bất kỳ ô tô hay xe máy nào nhìn thấy trong mặt gương phản chiếu. Nếu gán, xe tự lái sẽ phát hiện vật cản ảo ngay phía trước xe chủ, gây hiện tượng phanh khẩn cấp vô căn cứ (Phantom Braking). Khung gương khoanh `ignore_region` hoặc bỏ qua. |
| `img_029.jpg` | Guideline v0 không nói rõ ranh giới bounding box khi xe máy chở khối hàng hóa khổng lồ vượt gấp 3-4 lần kích thước xe và người lái gần như bị che lấp. | Box `motorcycle` bám sát khung sườn và bánh xe cơ học; thanh kim loại và bao xốp là hàng hóa rời, không nới rộng box xe theo hàng rời. Gán 1 box `person` cho người lái (`occluded_level = 2` hoặc `3`). |
| `img_038.jpg` | Guideline v0 định nghĩa `car` là ô tô con, `person` là người, nhưng không có điều khoản loại trừ mô hình đồ chơi thu nhỏ hoặc búp bê trưng bày ở sạp hàng. | **Không gán** các mô hình xe đồ chơi và búp bê thành `car`/`person`. Giữ nguyên làm nền âm tính để mô hình học cách phân biệt vật thật và vật giả. |
| `img_019.jpg` | Guideline v0 không có quy định xử lý khi có hàng chục người chen chúc, che lấp và dính sát nhau sau hàng rào chắn bảo vệ. | **Ca khó (Đám đông nhiều người dính nhau):** Bôi toàn bộ vùng nhiều người dính nhau là `ignore_region`, không vẽ box người riêng lẻ cho đám đông chen chúc. Xe diễu hành 4 bánh gán `car` (`can_review = true`), người lái xe thật trong cabin gán `person` (`occluded_level = 2`). |

---

## 2. Pha 2: Chỗ tôi bất đồng với nhóm và Quyết định bổ sung Taxonomy v1.0
- **Bổ sung 4 lớp phương tiện chuyên biệt (`bus`, `tuk_tuk`, `bicycle`, `cyclo`) vào Taxonomy:**
  - *Vấn đề xe buýt (`img_032`, `img_010`):* Bổ sung lớp **`bus`** để tách bạch phương tiện công cộng cỡ lớn với xe con và xe tải.
  - *Vấn đề xe Tuk-tuk / Auto Rickshaw (`img_004`, `img_035`, `img_040`):* Chuẩn hóa thành lớp **`tuk_tuk`** cho xe 3 bánh cơ giới gắn động cơ.
  - *Vấn đề xe Xích lô (`img_006`, `img_009`, `img_023`):* Ban đầu tôi đề xuất gộp chung vào xe đạp. Sau khi thảo luận sâu với Hoàng Nhật Anh, nhóm nhận thấy xích lô 3 bánh đạp chân có kích thước cồng kềnh, cấu trúc khoang khách phía trước khác biệt hoàn toàn với xe đạp 2 bánh. Do đó, nhóm quyết định tách riêng thành 2 lớp độc lập: **`bicycle`** (xe đạp 2 bánh) và **`cyclo`** (xe xích lô 3 bánh đạp chân).
  - *Kết quả thống nhất:* Cả nhóm thông qua bổ sung 4 lớp `bus`, `tuk_tuk`, `bicycle`, `cyclo` vào `taxonomy.json` phiên bản `v1.0` (tổng 10 lớp) và đồng bộ trực tiếp lên hệ thống CVAT Local / Cloud.

---

## 3. Pha 3: Ảnh tôi đã gán
- Số ảnh được giao: 24 ảnh (thuộc tập 40 ảnh của nhóm T005, theo phân bổ `assignment.csv`).
- Số box dự kiến đã vẽ: ~200 box trên 24 ảnh.
- Số lần bật cờ `can_review`: 3 lần (tại các ảnh có xe độ diễu hành `img_019`, xe chở rau củ `img_012`, và xe máy chở hàng quá tải `img_029`).
- Ca lưỡng lự nhất: `img_019.jpg` (xe ô tô diễu hành hình Danger Mouse và đám đông dính sát nhau sau rào chắn). Quyết định gán `car` vì là phương tiện cơ giới 4 bánh có người lái lưu thông công cộng. Đám đông khán giả chen chúc dính sát nhau sau rào chắn được xử lý bằng cách bôi toàn bộ vùng nhiều người là `ignore_region`, tránh vẽ hàng chục box người chồng lấn gây tranh cãi.

---

## 4. Pha 4–5: Tôi lệch ở đâu
- Trong quá trình đối chiếu IAA: Một số lệch phổ biến xảy ra ở ranh giới vẽ box người ngồi trên xe máy/xe đạp (phần gót chân có bao gồm bàn đạp xe hay không).
- Quyết định calibration quan trọng nhất: Thống nhất quy tắc vẽ box tách biệt giữa người lái và phương tiện 2-3 bánh (2 box chồng lấn: box người lấy từ đỉnh đầu/nón đến bàn chân, box xe lấy từ ghi-đông đến mặt tiếp đất của bánh xe). Đối với xe buýt `bus`, không vẽ box cho hành khách nhìn qua cửa sổ kính.

---

## 5. Dùng AI
- Công cụ sử dụng: Antigravity Assistant (Gemini/Claude).
- Mục đích sử dụng: Hỗ trợ viết script tự động hóa chia task CVAT, kiểm tra tính toàn vẹn của dữ liệu và rà soát các trường hợp biên an toàn (Edge Cases) trong 40 ảnh giao thông.
- Cách kiểm tra kết quả: Kiểm chứng thủ công từng ảnh thực tế trong `data/images/`, chạy `python scripts/check_submission.py` và chạy bộ kiểm thử `python -m unittest discover -s tests -v` để đảm bảo 20/20 test đạt OK.

---

## 6. Bài học
- Một điều tôi sẽ làm khác khi viết guideline lần sau: Luôn định nghĩa bảng phân loại lớp phương tiện đặc thù địa phương (như bus, tuk-tuk, xích lô) ngay từ đầu để tránh phải điều chỉnh lại taxonomy giữa chừng.
