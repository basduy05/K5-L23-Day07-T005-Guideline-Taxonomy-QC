# Ghi chú cá nhân — 2A202603240 - Phạm Mạnh Hùng

- Thành viên: Phạm Mạnh Hùng (MSHV: 2A202603240, Mã thành viên: 240 / Nhóm T005)
- Vai trò trong nhóm: Phụ trách hiệu chuẩn nhãn, rà soát ca lệch (Calibration & Error Review).

---

## 1. Pha 1: Ba ca khó tôi tìm ra trong 24 ảnh được phân công

| Ảnh | v0 không trả lời được gì | Quyết định ban đầu của tôi |
|---|---|---|
| `img_002.jpg` | Guideline v0 không quy định cách xử lý xe tải chuyên dụng chở ô tô (Car carrier) 2 tầng: box `truck` có bao gồm các xe con trên rơ-moóc không, và các xe con đó có được gán nhãn `car` độc lập không. | Gán 1 box `truck` bao trọn cả đầu kéo, rơ-moóc và toàn bộ các xe con được chở trên giá; **tuyệt đối không gán nhãn `car` riêng** cho các xe trên rơ-moóc vì chúng là hàng hóa tĩnh, không di chuyển độc lập và không tiếp đất. |
| `img_027.jpg` | Guideline v0 định nghĩa `truck` là xe tải chở hàng, nhưng không có điều khoản loại trừ đối với bia mộ hoặc tượng đài điêu khắc tạc hình xe tải bằng đá. | **Tuyệt đối không gán box `truck`** cho bia mộ đá tạc hình xe bán tải trong nghĩa trang; đây là vật thể tĩnh vĩnh cửu, không phải phương tiện giao thông. Gán nhãn sẽ gây ô nhiễm dữ liệu nghiêm trọng cho mạng nơ-ron nhận thức. |
| `img_033.jpg` | Guideline v0 định nghĩa `person` là người, `car` là ô tô con, nhưng không xử lý trường hợp tủ kính thời trang chứa ma-nơ-canh nhựa, poster in ảnh người mẫu 2D và bóng phản chiếu xe ô tô qua gương kính. | **Không gán bất kỳ box nào** cho ma-nơ-canh, ảnh in người mẫu hay xe phản chiếu qua gương; khoanh toàn bộ tủ kính vào `ignore_region` để ngăn chặn hiện tượng phanh ma (Phantom Braking) cho xe tự lái. |

---

## 2. Pha 2: Chỗ tôi bất đồng với nhóm
- **Vấn đề phân loại xe tải chuyên dụng chở xe (`img_002.jpg`):**
  - Một số bạn đề xuất nên gán nhãn `car` cho từng chiếc xe con nằm trên giàn xe tải.
  - Quan điểm của tôi: Các xe con trên rơ-moóc hoàn toàn là hàng hóa tĩnh. Nếu bộ phát hiện 2D gán `car`, hệ thống ước lượng 3D và lập quỹ đạo (Motion Planning) sẽ bị xung đột nghiêm trọng do các xe này không có bánh tiếp đất và di chuyển cùng vận tốc với xe tải. Cả nhóm đã thống nhất theo quan điểm này và đưa vào R-04.
- **Vấn đề người ngồi trong cabin xe đóng kín (`img_005.jpg`):**
  - Có ý kiến cho rằng cứ nhìn thấy người qua kính chắn gió/cửa sổ thì phải gán `person`.
  - Quan điểm của tôi: Chỉ gán `person` cho người tham gia giao thông hở mui, xe máy, xích lô, tuk-tuk hoặc ngồi trên thùng hở. Người trong cabin đóng kín đã được thân xe bảo vệ; gán `person` sẽ gây nhầm lẫn cho hệ thống phát hiện VRU. Nhóm đã đồng ý 100%.

---

## 3. Pha 3: Ảnh tôi đã gán
- Số ảnh được giao: 24 ảnh (thuộc Task 51 / Job 51 trên hệ thống CVAT, theo phân công `assignment.csv`).
- Số box dự kiến đã vẽ: ~190 box trên 24 ảnh.
- Số lần tôi tick `can_review`: 3 lần (`img_014.jpg` cho vệt xe máy nhòe chuyển động cao; `img_019.jpg` cho xe diễu hành hình dáng độc lạ; `img_037.jpg` cho xe bán tải gắn nắp thùng kín cao Master 75).
- Ca tôi lưỡng lự nhất: `img_037.jpg` (xe bán tải cứu hộ GO Transit lắp nắp thùng cao kín Master 75 Truck Cap). Ban đầu lưỡng lự giữa `van` và `truck`. Tuy nhiên, vì khung gầm là xe bán tải hạng nặng chuyên dụng với cabin tách rời thùng xe, tôi quyết định xếp vào `truck` và vẽ box bao cả nắp thùng cao.

---

## 4. Pha 4–5: Tôi lệch ở đâu
- Trong quá trình đối chiếu IAA: Lệch chủ yếu phát sinh ở đường bao đáy của các xe ô tô trên đường ướt (`img_015.jpg` - giữa việc dừng lại ở mặt lốp tiếp đất hay kéo tràn xuống bóng phản chiếu) và ranh giới box túi xách/ba lô của người đi bộ (`img_014.jpg`, `img_034.jpg`).
- Quyết định calibration tôi thấy quan trọng nhất: Thống nhất nguyên tắc tiếp xúc mặt đất của lốp xe (không lấy bóng phản chiếu) và tính gộp phụ kiện (ba lô, túi xách, mũ bảo hiểm) vào bounding box `person`.

---

## 5. Dùng AI
- Công cụ sử dụng: Antigravity Assistant.
- Mục đích sử dụng: Hỗ trợ rà soát các ca biên phức tạp (Edge Cases) trong 24 ảnh của Task 51 / Job 51, kiểm tra định dạng và cấu trúc guideline cá nhân theo đúng Rubric và Checkpoints.
- Cách kiểm tra kết quả: Tự mình mở và quan sát chi tiết từng ảnh gốc trong thư mục `work/2A202603240/images/`, đối chiếu với mã quy tắc và chạy các bài kiểm tra xác thực định dạng nhãn.

---

## 6. Bài học
- Một điều tôi sẽ làm khác khi viết guideline lần sau: Luôn xác lập các ví dụ phản chứng (Negative Examples) và quy tắc loại trừ ảo ảnh quang học/vật thể tĩnh (tượng, đồ chơi, ma-nơ-canh, ảnh phản chiếu) ngay từ trang đầu tiên để ngăn ngừa hoàn toàn các lỗi nhận diện tai hại cho hệ thống xe tự lái.
