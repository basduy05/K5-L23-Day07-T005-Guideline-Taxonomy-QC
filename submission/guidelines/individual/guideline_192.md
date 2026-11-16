# Guideline gán nhãn — v0.4-individual — 2A202603192 - Nguyễn Bá Duy

- Phiên bản: v0.4 (Bản cá nhân cập nhật chuẩn hóa đầy đủ 10 lớp theo thống nhất nhóm T005)
- Ngày: 2026-11-16
- Người viết: Nguyễn Bá Duy (MSSV: 2A202603192, Mã thành viên: 192 / Điều phối viên nhóm T005)
- Nhiệm vụ: Đề xuất quy chuẩn gán nhãn cá nhân sau khi nghiên cứu 24 ảnh được phân công trong bộ 40 ảnh cảnh giao thông đường phố châu Á, chuẩn hóa hệ thống 10 lớp hoàn chỉnh (`car`, `truck`, `van`, `bus`, `motorcycle`, `tuk_tuk`, `bicycle`, `cyclo`, `person`, `ignore_region`).

---

## 1. Mục tiêu và Ý nghĩa An toàn
- Tập dữ liệu này được xây dựng để huấn luyện mô hình thị giác máy tính phát hiện đối tượng (Object Detection 2D) cho hệ thống lái xe tự động (Autonomous Driving - AD) và giám sát giao thông thông minh (ITS) tại môi trường giao thông hỗn hợp châu Á.
- Hệ quả an toàn trong vận hành xe tự lái:
  - **Bỏ sót người (`person`), xe đạp (`bicycle`), xích lô (`cyclo`) hoặc xe máy (`motorcycle`):** Là lỗi nghiêm trọng nhất (Critical Safety Hazard). Hệ thống không phát hiện được yếu tố tổn thương (Vulnerable Road Users - VRU), dẫn đến nguy cơ đâm va trực diện gây tử vong.
  - **Tách bạch phương tiện công cộng (`bus`), phương tiện 3 bánh (`tuk_tuk`, `cyclo`) và xe thô sơ (`bicycle`):** 
    - Lớp `bus` (xe buýt, xe khách) có kích thước lớn, điểm mù rộng và thường xuyên tấp lề đón trả khách.
    - Lớp `tuk_tuk` phân biệt chính xác xe 3 bánh cơ giới gắn động cơ với ô tô con 4 bánh (`car`) và mô tô 2 bánh (`motorcycle`).
    - Lớp `cyclo` tách bạch xe xích lô 3 bánh đạp chân (kích thước cồng kềnh, có khoang khách phía trước) với xe đạp 2 bánh (`bicycle`).
  - **Gán nhãn sai cho ảo ảnh quang học (hình phản chiếu gương, tượng điêu khắc, đồ chơi, ma-nơ-canh):** Khiến xe tự lái phanh gấp vô cớ (Phantom Braking) khi di chuyển tốc độ cao, dẫn đến nguy cơ tai nạn liên hoàn từ các xe phía sau.

---

## 2. Lớp đối tượng và Thuộc tính đo lường

### 2.1 Định nghĩa 10 lớp đối tượng trong Taxonomy v1.0

| Lớp | Định nghĩa cốt lõi | Ví dụ hợp lệ trong ảnh giao thông | Phản ví dụ / Không gán |
|:---|:---|:---|:---|
| `car` | Phương tiện cơ giới 4 bánh có động cơ, kết cấu thân vỏ kín hoặc bán hở dùng chở người hoặc chở khách cỡ nhỏ (dưới 9 chỗ). | Sedan, SUV, Crossover, Hatchback, Taxi 4 bánh, wagon (`img_024`), xe diễu hành Dangermouse có người lái (`img_019`). | Xe bán tải chở hàng (`truck`), xe van chở hàng kín (`van`), xe buýt chở khách (`bus`), xe tuk-tuk 3 bánh (`tuk_tuk`). |
| `truck` | Phương tiện cơ giới chuyên dụng chở hàng hóa, có thùng hàng riêng biệt (thùng hở, thùng lửng, rơ-moóc hoặc sàn phẳng). | Xe tải thùng, xe ben chở đất, xe bồn, xe đầu kéo container, xe chuyên dụng chở xe con (`img_002`), xe bán tải (pickup truck, `img_012`, `img_036`, `img_037`). | Xe van chở hàng kín khoang liền cabin (`van`), xe ô tô con (`car`), xe buýt (`bus`), xe kéo tay thô sơ (không gán). |
| `van` | Xe ô tô thân thùng kín cỡ nhỏ đến vừa (khoảng 9-16 chỗ hoặc chở hàng kín), khoang lái và khoang chở hàng gắn liền thành một khối khí động học. | Xe chở hàng kín Ford Transit van, Mercedes Sprinter thùng kín, Suzuki Blind Van. | Xe bán tải thùng hở (`truck`), xe bán tải nắp thùng rời (`truck`), SUV chở gia đình (`car`), xe buýt cỡ lớn (`bus`). |
| `bus` | Phương tiện cơ giới chở khách công cộng cỡ vừa và lớn (trên 16 chỗ), xe buýt hai tầng, xe khách đường dài. | Xe buýt nội đô tuyến đường phố (`img_032`), xe khách liên tỉnh (`img_009`), xe buýt đón trả khách (`img_031`, `img_037`, `img_040`). | Xe van 9-16 chỗ chở hàng kín (`van`), xe ô tô con chở khách 7 chỗ (`car`), xe tải (`truck`). |
| `motorcycle` | Phương tiện cơ giới 2-3 bánh chạy động cơ đốt trong hoặc động cơ điện. | Xe máy số, xe tay ga (scooter), xe mô tô phân khối lớn, xe ba bánh có gắn thùng phụ bên hông (sidecar, `img_022`). | Xe đạp (`bicycle`), xe xích lô (`cyclo`), xe tuk-tuk 3 bánh (`tuk_tuk`). |
| `tuk_tuk` | Phương tiện cơ giới 3 bánh gắn động cơ đốt trong hoặc động cơ điện, có kết cấu cabin/mái che dùng chở khách hoặc chở hàng nhẹ. | Xe Tuk-tuk Thái Lan (`img_035`), xe Auto Rickshaw Ấn Độ (`img_004`, `img_040`), xe lam 3 bánh. | Xe máy 2 bánh thông thường (`motorcycle`), xe ô tô con 4 bánh (`car`), xe buýt (`bus`), xe xích lô đạp chân (`cyclo`). |
| `bicycle` | Phương tiện thô sơ 2 bánh vận hành bằng bàn đạp sử dụng sức người (hoặc trợ lực điện nhẹ). | Xe đạp thể thao, xe đạp thông thường (`img_006`, `img_029`, `img_039`). | Xe máy chạy xăng (`motorcycle`), xe xích lô 3 bánh (`cyclo`), xe đẩy tay hàng rong (không gán). |
| `cyclo` | Xe xích lô 3 bánh vận hành bằng bàn đạp sức người, có ghế/khoang hành khách phía trước và vị trí người đạp phía sau. | Xích lô du lịch chở khách tại Huế (`img_006`), xích lô chở khách ở trung tâm (`img_023`, `img_009`, `img_034`). | Xe đạp 2 bánh (`bicycle`), xe ba bánh có động cơ (`tuk_tuk`). |
| `person` | Cơ thể con người sống ở mọi tư thế: đi bộ, đứng, ngồi trên phương tiện, lái xe, đạp xe, bốc dỡ hàng hóa. | Người đi bộ qua đường (`img_014`), người lái xe máy, người ngồi sau xe máy (`img_029`), người đạp xích lô và khách ngồi xích lô (`img_006`, `img_023`), hành khách tại trạm xe buýt (`img_032`). | Ma-nơ-canh trong tủ kính (`img_033`), tượng đài điêu khắc (`img_017`, `img_027`), hình người in trên biển quảng cáo (`img_018`), búp bê đồ chơi (`img_038`). |
| `ignore_region` | Vùng không gian trong ảnh không gán chi tiết từng đối tượng do quá xa, mờ nhòe, hoặc tập hợp đông đúc dưới ngưỡng phát hiện, hoặc đám đông dày đặc nhiều người dính sát nhau (`img_019`); không tính lỗi phạt khi bỏ sót trong vùng này. | Cụm người đi bộ ở hậu cảnh xa xôi có chiều cao $< 20\text{ px}$, xe bị che khuất $> 80\%$, đám đông khán giả chen chúc nhiều người dính nhau sau hàng rào chắn (`img_019`), tủ trưng bày đồ chơi phức tạp. | Bất kỳ phương tiện hay con người nào nhìn thấy rõ ranh giới và có chiều cao $\ge 20\text{ px}$. |

### 2.2 Thuộc tính đo lường định lượng

1. **`occluded_level` (Mức độ che khuất bởi vật thể khác, không tính cắt mép ảnh):**
   - `0`: $r = 0\%$ — Nhìn thấy toàn bộ hình dạng vật thể, không bị vật khác che.
   - `1`: $0\% < r \le 30\%$ — Bị che khuất nhẹ (cột đèn, cành cây, mép xe khác).
   - `2`: $30\% < r \le 70\%$ — Bị che khuất trung bình/nhiều nhưng vẫn nhận dạng rõ ràng các bộ phận cốt lõi.
   - `3`: $70\% < r < 100\%$ — Bị che khuất nặng hoặc không thể ước lượng chính xác tỷ lệ diện tích bị che.
2. **`truncated` (Bị cắt bởi mép biên ảnh):**
   - `no`: Toàn bộ ranh giới vật thể nằm trọn vẹn trong khung hình ảnh.
   - `yes`: Vật thể bị cắt cụt bởi mép viền ảnh $\ge 5\text{ px}$.
3. **`can_review` (Đánh dấu trường hợp nghi ngờ/tranh chấp):**
   - `true`: Bật cờ khi gặp trường hợp mơ hồ (chưa chắc lớp, ranh giới thật/giả, ca đặc thù) để đưa ra hội chẩn nhóm (Calibration).
   - `false`: Trường hợp rõ ràng, tuân thủ đúng quy tắc.

---

## 3. Quy tắc vẽ Bounding Box (Quy chuẩn kỹ thuật)

1. **Nguyên tắc biên nhìn thấy thực tế (Visible Boundary Principle):**
   - Bounding box phải ôm sát các pixel thực sự nhìn thấy được của vật thể. Không tự suy diễn mở rộng box vào phần không nhìn thấy.
2. **Quy tắc phân tách Thân xe cơ học vs Hàng hóa rời:**
   - **Thân xe và bộ phận cố định:** Box phương tiện (`car`, `truck`, `bus`, `motorcycle`, `tuk_tuk`, `bicycle`, `cyclo`) ôm phần thân vỏ cơ học, khung sườn, bánh xe, tay lái, gương chiếu hậu và thùng xe cố định.
   - **Hàng hóa rời cồng kềnh:** Nông sản, thanh sắt, bọc xốp, hành lý cồng kềnh chở trên xe máy (`img_029`), trên thùng xe bán tải (`img_012`) hoặc trên nóc xe tuk-tuk (`img_035`) là **hàng rời**, không dùng làm điểm cực trị để nới rộng box xe.
3. **Quy tắc ghép đôi Người - Phương tiện 2/3 bánh (Coupling & Overlap):**
   - Đối với xe máy (`motorcycle`), xe đạp (`bicycle`), xe xích lô (`cyclo`), hoặc xe tuk-tuk hở cabin (`tuk_tuk`):
     - Vẽ **1 box `person`**: Ôm trọn vẹn người từ đỉnh đầu (hoặc đỉnh nón bảo hiểm) đến mép gót chân/bàn đạp chân.
     - Vẽ **1 box phương tiện (`motorcycle`, `bicycle`, `cyclo`, hoặc `tuk_tuk`)**: Ôm toàn bộ kết cấu cơ học của xe. Hai box này chấp nhận chồng lấn hình học lên nhau.
   - Xe xích lô (`cyclo`): 1 box `cyclo` cho khung xe + 1 box `person` cho người đạp + 1 box `person` cho mỗi hành khách.
4. **Quy tắc cho Xe buýt (`bus`):**
   - Box `bus` ôm sát toàn bộ thân vỏ xe buýt nhìn thấy được.
   - Hành khách ngồi bên trong cửa sổ kính xe buýt: **Không gán box `person` riêng lẻ**. Chỉ gán `person` cho tài xế hoặc hành khách đang thò nửa người ra ngoài cửa sổ hoặc bước xuống cửa xe.
5. **Quy tắc Xe chuyên chở xe (`img_002`):**
   - 1 box lớn `truck` bao toàn bộ tổ hợp xe chuyên chở (đầu kéo + sàn chở).
   - Mỗi ô tô con thật đặt trên các tầng sàn chở: Gán từng box `car` riêng biệt (`occluded_level = 1` hoặc `2`).
6. **Quy tắc Xe chủ (Ego-vehicle) và Phản chiếu gương (`img_020`):**
   - Gương chiếu hậu và mép cửa xe chủ bị loại bỏ.
   - Xe cộ và người nhìn thấy qua mặt gương phản chiếu là ảnh ảo $\rightarrow$ **tuyệt đối không gán**.
7. **Quy tắc Vật mô phỏng (Tượng, Đồ chơi, Hình in quảng cáo, Ma-nơ-canh):**
   - Giữ nguyên làm nền âm tính (negative background), **tuyệt đối không gán** thành người hoặc xe thật, và không phủ `ignore_region` lên chúng.
8. **Ngưỡng kích thước tối thiểu (Minimum Size Threshold):**
   - Chiều cao hình chiếu $< 20\text{ px}$: Không gán box đối tượng riêng lẻ; gom cụm vật nhỏ ở hậu cảnh xa vào 1 box `ignore_region`.
9. **Quy tắc Ca khó Đám đông nhiều người dính sát nhau (`img_019`):**
   - Khi có đám đông tập hợp đông đúc, nhiều người đứng chen chúc, đè lấn dính sát vào nhau sau hàng rào chắn bảo vệ: **Bôi toàn bộ vùng nhiều người dính nhau là `ignore_region`**.
   - Không vẽ box `person` đơn lẻ cho các cá nhân trong đám đông dính sát nhau này để tránh tranh cãi ranh giới hình học và sai lệch chỉ số QC. Các đối tượng foreground chính (xe diễu hành `car` và người lái thật `person` trong cabin) vẫn vẽ box chuẩn xác.

---

## 4. Phân tích chi tiết các ca khó thực tế trong 24 ảnh được phân công

| Mã ảnh | Hiện tượng quan sát | Đề xuất quyết định | Cơ sở lý luận & Ý nghĩa an toàn |
|:---|:---|:---|:---|
| `img_032.jpg` | Trạm dừng xe buýt nội đô đông đúc người chờ và xe buýt lớn ghé trạm. | Gán 1 box `bus` cho thân xe buýt. Gán từng box `person` cho người chờ xe có chiều cao $\ge 20\text{ px}$. Đám đông chen chúc ở quá xa $< 20\text{ px}$ khoanh vào `ignore_region`. | Xe buýt là phương tiện công cộng cỡ lớn cần cảnh báo vùng đệm an toàn; người chờ xe ở mép đường là đối tượng VRU có nguy cơ ngã vào làn xe. |
| `img_035.jpg` | Xe Tuk-tuk / Auto Rickshaw 3 bánh chở hàng cồng kềnh trên nóc và đông hành khách. | Gán 1 box `tuk_tuk` ôm sát khung sườn và cabin xe (không nới theo kiện hàng trên nóc). Gán box `person` cho tài xế và từng hành khách nhìn thấy rõ $\ge 20\text{ px}$ (`occluded_level = 1` hoặc `2`). | Tuk-tuk là phương tiện cơ giới 3 bánh đặc thù có động cơ; hàng hóa rời không kéo rộng box xe. |
| `img_006.jpg` | Xe xích lô du lịch chở khách nước ngoài tại Huế. | Gán 1 box `cyclo` cho toàn bộ khung xe xích lô 3 bánh đạp chân. Gán 2 box `person`: 1 cho người đạp xích lô phía sau, 1 cho du khách ngồi phía trước. | Xích lô 3 bánh đạp chân được định danh bằng lớp riêng `cyclo`. Hai người trên xe được theo dõi chuyển động riêng biệt. |
| `img_012.jpg` | Xe bán tải chở đầy các bao tải rau củ quả dạng lưới xanh di chuyển trên đường Hà Nội. | Gán 1 box `truck` ôm toàn bộ cabin và khung thùng xe bán tải (rau củ là hàng rời, không nới theo ngọn rau; `occluded_level = 0`, `truncated = no`). | Thùng hàng hở phía sau đạt chuẩn phân loại `truck`. Bám sát biên cơ học giúp mô hình nhận diện đúng hình dáng xe bán tải. |
| `img_020.jpg` | Hình ảnh nhìn từ trong buồng lái xe, quan sát thấy các xe ô tô và xe máy phản chiếu trong gương chiếu hậu (rearview mirror). | **Không gán box** cho các phương tiện và người nhìn thấy trong mặt gương phản chiếu. Khung gương và mép cửa xe chủ (ego-vehicle) không gán hoặc khoanh `ignore_region`. | Hình ảnh phản chiếu trong gương là vật thể ảo. Gán nhãn xe ở đây sẽ khiến hệ thống tự lái nhận diện có vật cản ngay trước mặt xe chủ, kích hoạt phanh khẩn cấp vô lý (Phantom Braking). |
| `img_029.jpg` | Xe máy chở thanh kim loại dài vượt xa kích thước xe và khối hàng xốp khổng lồ. | Gán 1 box `motorcycle` bám sát khung xe máy và bánh xe (không kéo theo thanh kim loại dài). Gán 1 box `person` cho người lái xe (`occluded_level = 2` hoặc `3`). | Phân tách rõ hàng rời và khung xe giúp bảo toàn đặc trưng hình học của phương tiện trong mạng nơ-ron. |
| `img_019.jpg` | Xe ô tô diễu hành Danger Mouse màu vàng có người lái và đám đông khán giả chen chúc dính sát nhau sau rào sắt. | **Ca khó (Đám đông nhiều người dính nhau):**<br>- Gán 1 box `car` bao quát toàn bộ thân xe diễu hành 4 bánh (`can_review = true`).<br>- Gán 1 box `person` cho người điều khiển xe thật nhìn thấy trong cabin (`occluded_level = 2`).<br>- **Bôi toàn bộ vùng nhiều người dính nhau** phía sau hàng rào thành `ignore_region`, không vẽ box người riêng lẻ cho đám đông chen chúc. | Phương tiện cơ giới 4 bánh có người lái lưu thông trên đường mang rủi ro va chạm của `car`. Khoanh toàn bộ đám đông khán giả dính sát nhau sau rào chắn thành `ignore_region` để loại bỏ tranh chấp ranh giới cá nhân bị che khuất nặng và không bị phạt lỗi QC bỏ sót. |
| `img_022.jpg` | Xe mô tô cổ 3 bánh gắn thùng phụ chở khách bên hông (Sidecar). | Gán 1 box `motorcycle` bao quát cả thân mô tô và thùng phụ bên cạnh. Gán 2 box `person` riêng biệt: 1 cho người điều khiển mô tô và 1 cho hành khách ngồi trong thùng phụ. | Thùng phụ gắn cố định vào xe máy tạo thành một phương tiện cơ giới 3 bánh; 2 người ngồi độc lập cần được theo dõi chuyển động riêng biệt. |
| `img_038.jpg` | Quầy hàng lưu niệm trưng bày nhiều mô hình xe ô tô thu nhỏ (toy cars) và búp bê đồ chơi. | **Tuyệt đối không gán** các mô hình xe đồ chơi thành `car` và búp bê thành `person`. Giữ nguyên làm nền âm tính. | Đồ chơi thu nhỏ không phải là người tham gia giao thông và không gây nguy hiểm va chạm cơ giới. |

---

## 5. Quy trình Kiểm soát Chất lượng (QC) và Lịch sử Phiên bản

### 5.1 Tiêu chí nghiệm thu QC
- **Độ trùng khớp hình học (Geometric IoU):** IoU giữa các annotator độc lập đối với cùng một đối tượng phải đạt $\ge 0.85$ (với vật thể cao $\ge 50\text{ px}$) và $\ge 0.70$ (với vật thể cao từ $20\text{ px}$ đến $< 50\text{ px}$).
- **Đồng thuận phân lớp (Classification Agreement):** Tỉ lệ đồng thuận nhãn lớp đối với các box bắt cặp $\ge 95\%$.
- **Cơ chế xử lý tranh chấp:**
  - Bật thuộc tính `can_review = true` trên box hoặc ghi vào `image_issues.csv` khi phát hiện các trường hợp mơ hồ để đưa ra phiên hiệu chuẩn (Calibration) của nhóm.

### 5.2 Lịch sử phiên bản
- **`v0.1` (2026-11-16):** Bản khởi tạo độc lập của Nguyễn Bá Duy dựa trên taxonomy 6 lớp cơ bản.
- **`v0.2` (2026-11-16):** Bổ sung 2 lớp `tuk_tuk` và `bicycle`.
- **`v0.3` (2026-11-16):** Bổ sung lớp `bus` và chuẩn hóa logic hàng hóa rời.
- **`v0.4` (2026-11-16):** Bổ sung lớp `cyclo` (xích lô 3 bánh đạp chân), hoàn thiện hệ thống 10 lớp chính thức đồng bộ toàn diện với nhóm T005 và CVAT Local.
