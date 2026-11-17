# Guideline gán nhãn thống nhất — v1.0-agreed — Nhóm T005

- Nhóm: **T005**
- Phiên bản: **v1.0-agreed** (Bản đồng thuận chính thức của nhóm sau phiên họp hiệu chuẩn Pha 2)
- Ngày hiệu lực: **16/11/2026**
- Thành viên biên soạn & biểu quyết:
  1. Nguyễn Đức Cường (2A202603040 / Mã: 040)
  2. Hoàng Nhật Anh (2A202603183 / Mã: 183)
  3. Nguyễn Bá Duy (2A202603192 / Mã: 192 — Điều phối viên)
  4. Đỗ Huy Kiên (2A202603216 / Mã: 216)
  5. Phạm Mạnh Hùng (2A202603240 / Mã: 240)
- Phạm vi áp dụng: **Toàn bộ 40/40 ảnh** (`img_001.jpg` – `img_040.jpg`) phục vụ lượt gán nhãn độc lập (Pha 3) và đối soát QC (Pha 4–6).
- Công cụ: **CVAT (Computer Vision Annotation Tool)**, định dạng xuất dữ liệu COCO 1.0.

---

## 1. Mục tiêu và Ý nghĩa An toàn

Tập dữ liệu này được xây dựng nhằm chuẩn hóa bộ nhãn huấn luyện mô hình thị giác máy tính 2D Object Detection (YOLO, Faster R-CNN, DETR) phục vụ hệ thống lái xe tự động (Autonomous Driving - AD) và giám sát giao thông thông minh (ITS) trong môi trường giao thông đô thị hỗn hợp châu Á.

### Ý nghĩa an toàn kỹ thuật:
1. **Bảo vệ đối tượng dễ tổn thương (Vulnerable Road Users - VRU):** Bỏ sót người đi bộ (`person`), người điều khiển xe máy (`motorcycle`), xe đạp (`bicycle`) hoặc xe xích lô (`cyclo`) là lỗi nghiêm trọng mức độ tối cao (Critical Hazard), dẫn đến nguy cơ tai nạn đâm va trực tiếp đe dọa tính mạng con người.
2. **Ước lượng ranh giới động học và khoảng cách phanh an toàn:**
   - Phân biệt rõ xe con (`car`), xe tải (`truck`), xe van (`van`), xe buýt (`bus`), xe 3 bánh có động cơ (`tuk_tuk`), xe đạp (`bicycle`) và xe xích lô (`cyclo`) giúp bộ lập quỹ đạo (Motion Planner) tính toán chính xác quán tính dừng, điểm mù phương tiện và không gian lách vượt an toàn.
   - Nhầm lẫn xe tải/buýt thành xe con sẽ khiến xe tự lái tính thiếu khoảng cách an toàn, dễ gây va chạm tạt đầu hoặc lọt vào điểm mù xe lớn.
3. **Chống kích hoạt phanh khẩn cấp ảo (Phantom Braking Mitigation):**
   - Không gán nhãn cho các ảo ảnh quang học (hình phản chiếu trong gương chiếu hậu, kính cửa sổ, mặt đường ướt) và các vật mô phỏng (tượng đá, hình in quảng cáo, búp bê, mô hình xe đồ chơi). Gán nhãn các vật này sẽ khiến xe tự lái nhận diện vật cản ảo ngay trước đầu xe và phanh khẩn cấp vô cớ khi đang di chuyển tốc độ cao, dẫn đến tai nạn dồn toa từ phía sau.

---

## 2. Lớp đối tượng và Thuộc tính

### 2.1. Taxonomy chính thức (10 lớp đối tượng)

Sau phiên họp Pha 2, nhóm T005 thống nhất bổ sung 4 lớp phương tiện đặc thù (`bus`, `tuk_tuk`, `bicycle`, `cyclo`) vào bộ 6 lớp cơ sở của v0, tạo thành taxonomy 10 lớp hoàn chỉnh được quản lý tại `submission/taxonomy.json`:

| Lớp | Định nghĩa cốt lõi và Dấu hiệu phân biệt | Ví dụ trong bộ 40 ảnh | Phản ví dụ / Dễ nhầm lẫn |
|:---|:---|:---|:---|
| `car` | Ô tô con 4 bánh có kết cấu thân vỏ chở người nhỏ (dưới 9 chỗ): sedan, hatchback, crossover, SUV, wagon, taxi, xe diễu hành 4 bánh có người lái. | Xe sedan `img_023`; wagon đen `img_024`; taxi `img_040`; xe diễu hành Danger Mouse `img_019`. | Xe bán tải (`truck`), xe van chở hàng kín (`van`), xe buýt (`bus`), xe tuk-tuk 3 bánh (`tuk_tuk`), mô hình ô tô đồ chơi `img_038`. |
| `truck` | Xe cơ giới chuyên dụng chở hàng hóa, có cabin và thùng/sàn chở hàng tách biệt về kết cấu (thùng hở, thùng kín, rơ-moóc hoặc sàn chở xe). Bao gồm xe bán tải (pickup truck). | Xe tải nhẹ `img_001`, `img_007`, `img_036`; xe bán tải chở hàng `img_012`; pickup có nắp thùng `img_037`; xe chuyên dụng chở ô tô `img_002`. | Xe van khoang liền cabin (`van`), xe ô tô con chở người (`car`), tượng đài xe bán tải `img_027` (nền âm tính). |
| `van` | Xe ô tô thân thùng kín cỡ nhỏ đến vừa (khoảng 9–16 chỗ hoặc chở hàng kín), khoang lái và khoang hàng gắn liền thành một khối khí động học; không có thùng hàng độc lập. | Xe thân hộp phía sau xe máy `img_005`; xe van trắng `img_016`. | Xe tải nhỏ cabin rời `img_036` là `truck`; xe bán tải lắp nắp thùng cao `img_037` vẫn là `truck`; xe khách lớn là `bus`. |
| `bus` | Phương tiện cơ giới chở khách công cộng cỡ lớn hoặc vừa (trên 16 chỗ), xe buýt hai tầng, xe khách đường dài liên tỉnh. | Xe khách thân dài giữa cảnh `img_009`; xe buýt nội đô `img_031`, `img_032`, `img_037`, `img_040`. | Xe van 9–16 chỗ (`van`), xe ô tô con 7 chỗ (`car`), xe tải chở hàng (`truck`). |
| `motorcycle` | Xe máy, mô tô 2–3 bánh có gắn động cơ đốt trong hoặc động cơ điện. Bao gồm cả xe tay ga (scooter) và tổ hợp mô tô gắn thùng phụ bên hông (sidecar cố định). | Scooter `img_010`, `img_026`; mô tô sidecar 3 bánh `img_022`; xe máy chở hàng `img_029`. | Xe đạp `bicycle`, xe xích lô `cyclo`, xe tuk-tuk chuyên dụng `tuk_tuk`, các tác phẩm mô phỏng xe bằng kim loại `img_011`. |
| `tuk_tuk` | Xe 3 bánh cơ giới gắn động cơ (auto-rickshaw, xe lam, tuk-tuk), có cụm cabin và khoang chở khách hoặc hàng nhẹ trên khung xe 3 bánh chuyên dụng. | Tuk-tuk Thái Lan `img_035`; auto-rickshaw Ấn Độ `img_004`, `img_016`, `img_040`. | Mô tô 2 bánh thông thường (`motorcycle`); mô tô gắn sidecar `img_022` (vẫn là `motorcycle`); xe xích lô đạp chân (`cyclo`). |
| `bicycle` | Phương tiện thô sơ 2 bánh vận hành bằng bàn đạp sử dụng sức người (hoặc trợ lực điện nhẹ): xe đạp thể thao, xe đạp mini, xe đạp cào cào. | Xe đạp `img_006`, `img_029`, `img_039`. | Xe máy có động cơ (`motorcycle`), xe xích lô 3 bánh (`cyclo`), xe đẩy tay hàng rong (ngoài taxonomy). |
| `cyclo` | Xe xích lô 3 bánh dùng bàn đạp sức người, có ghế/khoang hành khách phía trước và vị trí người đạp phía sau. | Xích lô du lịch tại Huế `img_006`, xích lô chở khách ở trung tâm `img_023`, `img_009`, `img_034`. | Xe đạp 2 bánh (`bicycle`), xe ba bánh có động cơ (`tuk_tuk`). |
| `person` | Con người sống thật nhìn thấy trực tiếp ở mọi tư thế: đi bộ, đứng, ngồi, lái xe, đạp xe, bốc dỡ hàng hóa. | Người bán hàng `img_003`; người đi bộ `img_014`; người điều khiển và hành khách trên xe `img_026`, `img_040`. | Tượng điêu khắc `img_017`, chân dung trên billboard `img_018`, ma-nơ-canh `img_033`, búp bê đồ chơi `img_038`, ảnh phản chiếu gương. |
| `ignore_region` | Hình chữ nhật xác định vùng không gian không gán chi tiết từng đối tượng do quá xa, mờ nhòe, hoặc tập hợp đông đúc dưới ngưỡng phát hiện ($< 20\text{ px}$), hoặc đám đông dày đặc nhiều người dính sát nhau khó tách biệt ranh giới (`img_019`). Bỏ sót trong vùng này không tính là lỗi. | Cụm người/xe ở hậu cảnh xa xôi `img_021`, đám đông xếp hàng ở trạm buýt ngoài cự ly `img_032`, đám đông khán giả chen chúc dính sát nhau sau hàng rào chắn `img_019`. | Không phủ lên tượng, đồ chơi, hình in đã biết là vật âm tính; không phủ lên toàn ảnh để trốn tránh việc vẽ box. |

---

### 2.2. Thuộc tính đo lường định lượng

Mỗi bounding box thuộc 9 lớp đối tượng thực bắt buộc phải điền đủ 3 thuộc tính sau (`ignore_region` không có thuộc tính):

1. **`occluded_level` (Mức độ che khuất):**
   Đo bằng tỷ lệ diện tích $r$ của hình dạng đối tượng bị vật thể khác che khuất trên **phần cơ thể/thân xe lẽ ra phải nhìn thấy được trong ảnh**.
   - `0`: $r = 0\%$ — Không có bất kỳ phần nào bị vật khác che khuất (kể cả khi vật bị mép ảnh cắt).
   - `1`: $0\% < r \le 30\%$ — Bị che khuất nhẹ (cột đèn, cành cây, mép xe khác).
   - `2`: $30\% < r \le 70\%$ — Bị che khuất trung bình đến nhiều (vẫn nhận diện được đặc trưng chính như bánh xe, đầu xe, nón bảo hiểm).
   - `3`: $70\% < r < 100\%$ — Bị che khuất nặng hoặc không thể ước lượng diện tích che một cách đáng tin cậy.
   > *Quy tắc kiểm ranh giới:* $0\% \rightarrow `0`$; $20\%, 30\% \rightarrow `1`$; $31\%, 70\% \rightarrow `2`$; $71\% \rightarrow `3`$. Vật bị che $100\%$ hoàn toàn không được suy diễn để vẽ box.

2. **`truncated` (Bị cắt bởi mép ảnh):**
   - `no`: Toàn bộ ranh giới vật thể nằm trọn vẹn bên trong khung hình.
   - `yes`: Vật thể bị cắt mất một phần cơ thể/vỏ xe tại bất kỳ mép viền nào của ảnh ($\ge 5\text{ px}$).

3. **`can_review` (Đánh dấu cần xem xét lại):**
   - `false`: Trường hợp rõ ràng, tuân thủ đúng quy tắc chuẩn.
   - `true`: Bật cờ khi gặp trường hợp mơ hồ (chưa chắc chắn về lớp, ranh giới thật/giả, ca đặc thù chưa có tiền lệ) để đưa ra hội chẩn hiệu chuẩn (Calibration).

---

## 3. Quy tắc vẽ Bounding Box (Quy chuẩn kỹ thuật)

### 3.1. Nguyên tắc biên nhìn thấy thực tế (Visible Boundary Principle)
- Bounding box là hình chữ nhật song song trục tọa độ ảnh, ôm sát các pixel **thực sự nhìn thấy được** của vật thể.
- Tuyệt đối không mở rộng box để suy diễn phần bị che khuất hoặc phần đã nằm ngoài khung ảnh (không đoán vị trí chân người sau bàn, không đoán vị trí bánh xe bị che).
- Tọa độ chuẩn COCO: `[x_min, y_min, width, height]`.

### 3.2. Quy tắc phân định Thân xe cơ học vs Hàng hóa rời
- **Bộ phận cố định của xe:** Box phương tiện ôm trọn thân vỏ kim loại, khung sườn, bánh xe, tay lái, gương chiếu hậu và thùng hàng cố định.
- **Hàng hóa rời cồng kềnh:** Nông sản, bao tải lưới (`img_012`), thanh kim loại dài (`img_029`), bọc xốp chở quá tải (`img_029`, `img_035`) là **hàng rời**, không dùng làm điểm cực trị để nới rộng box xe. Box xe bám sát kết cấu cơ học của phương tiện nhằm bảo toàn tỷ lệ khung hình (aspect ratio) chuẩn của mô hình thị giác.

### 3.3. Quy tắc ghép đôi Người và Phương tiện 2/3 bánh (Coupling & Overlap)
- Đối với xe máy (`motorcycle`), xe đạp (`bicycle`), xe xích lô (`cyclo`), hoặc tuk-tuk hở (`tuk_tuk`):
  - Người và xe là **hai đối tượng độc lập**, box chấp nhận chồng lấn lên nhau.
  - Một xe chở $N$ người nhìn thấy rõ: Vẽ **1 box xe** bao toàn bộ kết cấu cơ học xe + **$N$ box `person`** riêng biệt cho từng người (từ đỉnh nón bảo hiểm đến bàn chân).
  - Đối với xe xích lô (`cyclo`): Vẽ 1 box `cyclo` cho toàn bộ khung xe xích lô + 1 box `person` cho người đạp phía sau + 1 box `person` cho mỗi hành khách ngồi ở khoang trước.

### 3.4. Quy tắc Xe buýt (`bus`) và Xe buồng lái kín
- Box `bus` ôm sát toàn bộ thân vỏ xe buýt nhìn thấy được.
- Hành khách ngồi bên trong cửa sổ kính xe buýt/ô tô con: **Không gán box `person` riêng lẻ** để tránh gây nhiễu bộ đếm hành khách. Chỉ gán `person` cho tài xế hoặc hành khách đang thò nửa người ra ngoài cửa sổ hoặc bước xuống cửa xe.

### 3.5. Quy tắc Xe chuyên dụng chở xe (`img_002`)
- 1 box lớn `truck` bao toàn bộ tổ hợp xe chuyên chở (đầu kéo + sàn chở/rơ-moóc).
- Mỗi ô tô con thật đặt trên các tầng sàn chở: Gán từng box `car` riêng biệt theo phần nhìn thấy (`occluded_level = 1` hoặc `2` do khung sàn che).

### 3.6. Quy tắc Xe chủ (Ego-vehicle) và Phản chiếu gương/kính (`img_020`)
- Gương chiếu hậu và mép cửa sổ của chính xe gắn camera chụp (xe chủ) bị loại bỏ.
- Xe cộ và người nhìn thấy qua mặt gương phản chiếu hoặc mặt đường ướt là ảnh ảo $\rightarrow$ **tuyệt đối không gán**.
- Nhìn xuyên qua cửa kính trong suốt thấy xe/người thật phía sau vẫn gán bình thường.

### 3.7. Quy tắc Vật mô phỏng (Tượng, Đồ chơi, Hình in quảng cáo, Ma-nơ-canh)
- Tượng điêu khắc (`img_017`, `img_027`), tranh ảnh chân dung (`img_018`), ma-nơ-canh (`img_033`), mô hình ô tô đồ chơi và búp bê (`img_038`): **Tuyệt đối không gán** thành người hoặc xe.
- Giữ nguyên các vật này làm nền âm tính (hard negative background), **không phủ `ignore_region` lên chúng** để bộ phát hiện học cách loại bỏ vật giả.
- Người thật hóa trang (cosplay Predator `img_011`): Xác định là người thật điều khiển hoặc cử động thì gán `person` (bật `can_review=true`).
- Trường hợp tượng/mô hình mascot gắn trên xe (như tượng chuột Danger Mouse trong `img_019`): Chỉ gán `person` cho người lái xe thật nhìn thấy trong buồng lái; nhân vật chuột mô phỏng không có căn cứ người thật thì không tự gán `person`.

### 3.8. Quy tắc Kích thước tối thiểu và Vùng bỏ qua (Ignore Region)
- Chiều cao hình chiếu $h = y_{max} - y_{min}$ trên ảnh gốc:
  - $h \ge 20\text{ px}$: Bắt buộc gán box đối tượng riêng lẻ.
  - $h < 20\text{ px}$: Không gán box đối tượng riêng; gom cụm vật nhỏ ở hậu cảnh xa vào 1 box `ignore_region` gọn gàng.
- Không dùng `ignore_region` bao trùm lên toàn bộ ảnh hoặc che phủ các đối tượng lớn rõ ràng.
- **Xử lý ca khó Đám đông nhiều người dính sát nhau (`img_019.jpg`):**
  - Khi xuất hiện tình huống đám đông tập hợp đông đúc, nhiều người đứng chen chúc, đè lấn và dính sát vào nhau sau hàng rào chắn bảo vệ (như trong `img_019.jpg`), việc vẽ tách biệt từng box người riêng lẻ sẽ dẫn đến tranh chấp ranh giới hình học gay gắt, độ che khuất không thể xác định chính xác và gây sụt giảm mạnh chỉ số IoU giữa các annotator.
  - **Cách xử lý chuẩn hóa:** Bôi/khoanh toàn bộ vùng tập trung nhiều người dính sát nhau này bằng nhãn **`ignore_region`**. Tuyệt đối không gán từng box `person` đơn lẻ cho các cá nhân nằm trong vùng đám đông dính nhau này. Lỗi bỏ sót bên trong `ignore_region` không bị tính là lỗi QC. Đối tượng foreground chính (xe diễu hành `car` và người lái thật `person` trong cabin) vẫn phải được gán box chuẩn xác.

---

## 4. Bảng quyết định chuẩn hóa 40/40 ảnh thực tế của nhóm T005

| Mã ảnh | Tình huống quan sát | Quyết định chuẩn hóa của Nhóm T005 | Cơ sở lý luận & Ý nghĩa an toàn |
|:---|:---|:---|:---|
| [img_001](../../data/images/img_001.jpg) | Xe tải nhỏ và nhiều người ngồi, chân/thân bị bàn và xe che. | Xe gán `truck`; người gán từng `person` $\ge 20\text{ px}$; `truncated=yes` cho người bị cắt cạnh dưới ảnh. | Xe tải nhỏ cabin rời là `truck`; không suy đoán phần thân người sau bàn. |
| [img_002](../../data/images/img_002.jpg) | Xe chuyên chở mang nhiều ô tô con trên giá 2 tầng. | 1 box lớn `truck` cho xe chuyên chở; từng xe con thật trên sàn gán `car` (`occluded_level=1` hoặc `2`). | Xe được chở là chướng ngại vật khối hộp cần nhận diện thể tích va chạm. |
| [img_003](../../data/images/img_003.jpg) | Người bán hàng mang đòn gánh hoa quả, xe đạp và xe máy. | Gán `person`, `bicycle`, `motorcycle` riêng; đòn gánh và thúng hoa quả là hàng rời, không mở rộng box người. | Hàng hóa rời không tính vào kích thước cơ thể người. |
| [img_004](../../data/images/img_004.jpg) | Hai xe auto-rickshaw 3 bánh có người ngồi bên trong. | Gán từng xe là `tuk_tuk`; người nhìn thấy rõ $\ge 20\text{ px}$ gán từng box `person`. | Phân lớp chuẩn xác xe 3 bánh cơ giới chở khách đặc thù châu Á. |
| [img_005](../../data/images/img_005.jpg) | Người đi xe máy sát nhau, xe thân hộp phía sau. | Người và xe máy gán riêng (1 `person` + 1 `motorcycle`); xe thân hộp phía sau gán `van`. | Xe thân hộp liền khối là `van`; phân tách người và phương tiện. |
| [img_006](../../data/images/img_006.jpg) | Xe xích lô du lịch chở khách tại Huế, xe đạp và xe máy. | Xích lô gán `cyclo`; 2 `person` cho người đạp và du khách; xe đạp dùng `bicycle`, xe máy dùng `motorcycle`. | Xích lô 3 bánh đạp chân có lớp riêng `cyclo`. |
| [img_007](../../data/images/img_007.jpg) | Xe tải thùng kín và pickup trong kho tối. | Xe có thùng tách cabin gán `truck`; xe máy gán `motorcycle`; thiếu sáng không tự động tăng mức che. | Cấu trúc thùng tách rời cabin là `truck`; bóng tối không đồng nghĩa bị che. |
| [img_008](../../data/images/img_008.jpg) | Bãi xe máy đỗ dày, người in trên mặt sau áo. | Gán từng `motorcycle` đỗ và từng `person` thật; không gán hình chân dung in trên áo. | Xe đỗ vẫn là chướng ngại vật; hình in trên áo là nền âm tính. |
| [img_009](../../data/images/img_009.jpg) | Xe khách thân dài giữa cảnh, xe xích lô và người. | Xe khách gán `bus`; xe xích lô gán `cyclo` + `person`; người và xe máy xung quanh theo lớp riêng. | Xe khách lớn quy chuẩn vào `bus` để cảnh báo vùng an toàn đón trả khách. |
| [img_010](../../data/images/img_010.jpg) | Người lái scooter và xe đẩy chở hoa quả bán rong. | Gán 1 `person` + 1 `motorcycle`; xe đẩy hoa quả bằng tay ngoài taxonomy (không gán xe), chỉ gán `person`. | Xe đẩy tay không có động cơ, không thuộc phương tiện cơ giới. |
| [img_011](../../data/images/img_011.jpg) | Nhân vật Predator tiền cảnh và tạo hình xe kim loại. | Predator là người thật hóa trang $\rightarrow$ gán `person` (`can_review=true`); các mô hình xe kim loại không có động cơ $\rightarrow$ không gán xe. | Không gán xe mô phỏng không có chức năng giao thông. |
| [img_012](../../data/images/img_012.jpg) | Xe bán tải chở rau củ đầy thùng sát camera; SUV bên cạnh. | SUV là `car`; xe bán tải là `truck` (box ôm cabin và thành thùng xe, không nới theo ngọn rau); loại bỏ ego nếu có. | Rau củ là hàng rời; box xe bám sát kết cấu cơ học thùng xe. |
| [img_013](../../data/images/img_013.jpg) | Đường ướt ban đêm, người và xe phản chiếu trên mặt đường. | Chỉ gán người và xe thật; tuyệt đối không gán bóng phản chiếu và vệt đèn trên mặt đường ướt. | Tránh tạo ảo ảnh khiến xe tự lái nhận diện vật cản dưới gầm xe. |
| [img_014](../../data/images/img_014.jpg) | Người đi bộ băng qua đường giữa dòng xe máy. | Gán `person` cho người đi bộ (`occluded_level=1` hoặc `2`); gán `motorcycle` và `person` cho các xe máy. | Bảo vệ người đi bộ qua đường (VRU) là ưu tiên an toàn cao nhất. |
| [img_015](../../data/images/img_015.jpg) | Ô tô bị cắt mép hai bên ảnh trong trời mưa nhòe. | Xe hai bên gán `car` theo phần nhìn thấy (`truncated=yes`); vệt nước mưa và nhòe không kéo rộng box. | Cận cảnh thấy một phần xe vẫn gán nếu đủ đặc trưng nhận diện. |
| [img_016](../../data/images/img_016.jpg) | Xe tuk-tuk, xe máy, van và người bị cắt ở các cạnh ảnh. | Gán tách biệt `tuk_tuk`, `motorcycle`, `van`, `person`; đánh dấu `truncated=yes` cho vật chạm mép ảnh. | Định danh chính xác từng phương tiện trong giao thông hỗn hợp. |
| [img_017](../../data/images/img_017.jpg) | Tượng đài công tước Wellington cưỡi ngựa đội nón giao thông. | Không gán `person` cho tượng người; tượng ngựa và nón chóp ngoài taxonomy. Ảnh không có annotation. | Tượng điêu khắc là nền âm tính, tuyệt đối không gán nhãn người thật. |
| [img_018](../../data/images/img_018.jpg) | Ảnh chân dung trên billboard và ô tô sau hàng rào kim loại. | Không gán chân dung billboard; gán `car` cho ô tô sau hàng rào (`occluded_level=1` hoặc `2` tùy mức che). | Ảnh in quảng cáo không phải người; hàng rào kim loại là vật thể che khuất. |
| [img_019](../../data/images/img_019.jpg) | Xe diễu hành Danger Mouse màu vàng có người lái và đám đông khán giả dính sát nhau sau hàng rào chắn. | **Ca khó (Đám đông nhiều người dính nhau):**<br>1. Gán 1 `car` bao quát thân xe diễu hành 4 bánh (`can_review=true`).<br>2. Gán 1 `person` cho người lái xe thật nhìn thấy trong cabin (`occluded_level=2`).<br>3. Bôi toàn bộ vùng đám đông nhiều người dính sát nhau phía sau hàng rào chắn là `ignore_region` (không gán từng box `person` riêng lẻ cho cụm người chen chúc). | Phương tiện cơ giới 4 bánh có người lái lưu thông trên đường mang rủi ro va chạm của `car`. Khoanh toàn bộ đám đông khán giả dính sát nhau sau rào chắn thành `ignore_region` để tránh tranh chấp ranh giới từng người bị che khuất và loại bỏ nhiễu cho mô hình thị giác. |
| [img_020](../../data/images/img_020.jpg) | Gương chiếu hậu buồng lái nhìn thấy xe cộ phía sau. | Loại bỏ xe phản chiếu trong gương và phần xe chủ (gương/khung cửa). Ảnh không có annotation hợp lệ. | Hình ảnh trong gương là ảnh ảo; gán nhãn sẽ kích hoạt phanh khẩn cấp vô cớ. |
| [img_021](../../data/images/img_021.jpg) | Dòng xe máy cực đông tại giao lộ, xe xa chỉ lộ một phần. | Gán riêng từng người và xe máy ở cự ly gần; cụm xe/người ở quá xa $< 20\text{ px}$ khoanh `ignore_region`. | Không gom cả đám đông vào một box; xử lý vật siêu nhỏ bằng ignore. |
| [img_022](../../data/images/img_022.jpg) | Mô tô cổ gắn thùng phụ chở khách bên hông (Sidecar). | 1 box `motorcycle` bao trọn cụm mô tô và thùng phụ; gán 2 box `person` (1 người lái, 1 người trong sidecar). | Thùng phụ gắn cố định vào mô tô tạo thành một phương tiện 3 bánh thống nhất. |
| [img_023](../../data/images/img_023.jpg) | Xích lô du lịch ở trung tâm và ô tô đen bên trái. | Gán 1 `cyclo` cho khung xe xích lô; 1 `person` cho người đạp; ô tô đen gán `car`. | Phân định rõ xích lô 3 bánh `cyclo` và ô tô con `car`. |
| [img_024](../../data/images/img_024.jpg) | Xe wagon đen thật trong khu trưng bày, mô hình vàng trên nóc. | Wagon đen gán `car`; vật mô hình nhỏ màu vàng trên nóc loại bỏ (không gán); các xe thật xung quanh gán `car`. | Mô hình trưng bày thu nhỏ không phải là xe thật được chuyên chở. |
| [img_025](../../data/images/img_025.jpg) | Người đi bộ ở xa, xe máy gắn giỏ phụ bên hông. | Gán riêng `person` và `motorcycle`; giỏ phụ gắn cố định thuộc khung xe máy; kiểm tra ngưỡng 20 px ở xa. | Bộ phận cơ học gắn cố định thuộc về phương tiện; đo vật xa cẩn thận. |
| [img_026](../../data/images/img_026.jpg) | Hai người ngồi trên xe máy trong hầm, tranh tường phía sau. | Gán 1 `motorcycle`, 2 `person` riêng biệt cho 2 người; kiểm tra cắt mép chân; tranh tường không gán. | Tách bạch từng người ngồi trên xe máy; tranh vẽ là nền âm tính. |
| [img_027](../../data/images/img_027.jpg) | Đài tưởng niệm tạc hình xe bán tải bằng đá nguyên khối. | Không gán `truck`/`car` cho tượng xe bán tải; giữ nguyên làm nền âm tính. Ảnh không có annotation. | Tác phẩm điêu khắc không có chức năng giao thông, không gán phương tiện. |
| [img_028](../../data/images/img_028.jpg) | Xe bán tải thật chở tượng ngựa trang trí trên thùng xe. | Gán `truck` cho xe bán tải (box bám thân xe, không nới theo đầu ngựa); ô tô đỏ gán `car`; tượng ngựa ngoài taxonomy. | Xe thật chở vật giả vẫn là xe thật; hàng hóa rời không làm nới rộng box xe. |
| [img_029](../../data/images/img_029.jpg) | Xe máy chở thanh kim loại dài vượt xa kích thước xe và bó xốp. | Gán `motorcycle` bám sát khung sườn và bánh xe (không kéo theo thanh kim loại); gán `person`; xe đạp gán `bicycle`. | Hàng hóa rời không kéo dài box xe để tránh méo mó tỷ lệ hình học mạng nơ-ron. |
| [img_030](../../data/images/img_030.jpg) | Bãi xe máy dày đặc ban đêm, poster chân dung phía trên. | Gán từng `motorcycle` còn phân biệt được; cụm đỗ sâu không tách được khoanh `ignore_region`; không gán poster. | Phân loại chính xác xe đỗ; tranh ảnh quảng cáo là nền âm tính. |
| [img_031](../../data/images/img_031.jpg) | Xe buýt nội đô, ô tô và xe máy di chuyển ban đêm. | Gán `bus` cho thân xe buýt; ô tô gán `car`; người gán `person`; loại bỏ vệt đèn phản chiếu trên đường. | Xe buýt lớn cần theo dõi quỹ đạo chuyên biệt; loại bỏ phản chiếu ánh đèn. |
| [img_032](../../data/images/img_032.jpg) | Trạm dừng xe buýt đông đúc người chờ và xe buýt ghé trạm. | Gán `bus` cho xe buýt; người chờ xe $\ge 20\text{ px}$ gán từng `person`; cụm chen chúc quá xa $< 20\text{ px}$ khoanh `ignore_region`. | Điểm dừng xe buýt là khu vực rủi ro cao tập trung nhiều đối tượng VRU. |
| [img_033](../../data/images/img_033.jpg) | Ma-nơ-canh trong tủ kính thời trang, xe phản chiếu trên kính. | Không gán ma-nơ-canh thành `person`; không gán xe phản chiếu; giữ làm nền âm tính. Ảnh không có annotation. | Ma-nơ-canh và ảnh phản chiếu kính là các trường hợp kiểm tra nhãn thừa. |
| [img_034](../../data/images/img_034.jpg) | Dòng giao thông hỗn hợp có người đi bộ và xe đạp. | Người đi bộ gán `person`; xe đạp gán `bicycle`; xe máy gán `motorcycle`; kiểm tra che và cắt mép độc lập. | Không gộp người đi ngang vào xe phía sau; phân loại đúng xe đạp. |
| [img_035](../../data/images/img_035.jpg) | Auto-rickshaw chở hàng quá tải trên nóc và đông hành khách. | Gán `tuk_tuk` ôm sát khung vỏ xe (không nới theo kiện hàng); gán `person` cho tài xế và hành khách nhìn thấy. | Tuk-tuk chở hàng quá tải không đổi lớp; hàng rời không kéo rộng box xe. |
| [img_036](../../data/images/img_036.jpg) | Xe tải nhỏ CMC Veryca có thùng lửng phủ bạt trên đường. | Gán `truck` bao gồm cabin và thùng xe cố định. | Thùng xe tách biệt cabin lái là tiêu chí cốt lõi của `truck`, không phụ thuộc kích thước. |
| [img_037](../../data/images/img_037.jpg) | Xe bán tải gắn nắp thùng cao (canopy), xe buýt phía sau. | Bán tải nắp thùng cao vẫn là `truck`; xe buýt gán `bus`; người đi lại gán `person`. | Phụ kiện nắp thùng không biến xe bán tải thành xe van. |
| [img_038](../../data/images/img_038.jpg) | Quầy hàng lưu niệm trưng bày nhiều mô hình xe ô tô và búp bê. | Tuyệt đối không gán xe đồ chơi và búp bê thành `car`/`person`. Giữ nguyên làm nền âm tính. | Mô hình thu nhỏ không phải người tham gia giao thông và không gây rủi ro va chạm. |
| [img_039](../../data/images/img_039.jpg) | Phố có người, xe đạp, xe máy đỗ/chạy và ô tô phía xa. | Gán riêng `person`, `bicycle`, `motorcycle`, `car` đủ điều kiện $\ge 20\text{ px}$; xe đỗ không người vẫn gán box xe. | Phương tiện đỗ trên lòng đường vẫn là chướng ngại vật vật lý cần nhận diện. |
| [img_040](../../data/images/img_040.jpg) | Người đội mũ kín mặt ở tiền cảnh, taxi, tuk-tuk, SUV, buýt phía sau. | Người là `person`; taxi/SUV là `car`; tuk-tuk là `tuk_tuk`; xe buýt là `bus`; xe máy lộ ít gán nếu $\ge 20\text{ px}$. | Nhận diện người không phụ thuộc vào việc có nhìn thấy khuôn mặt hay không. |

---

## 5. Quy trình Kiểm soát Chất lượng (QC) và Lịch sử Phiên bản

### 5.1. Tiêu chí nghiệm thu định lượng (Khóa theo `taxonomy.json -> rules`)

Toàn bộ các ngưỡng kỹ thuật kiểm soát chất lượng được áp dụng tự động bởi bộ script kiểm tra của nhóm và không được sửa đổi:

| Chỉ số | Ngưỡng yêu cầu | Ý nghĩa & Cách đo |
|:---|:---:|:---|
| **Chiều cao box tối thiểu** | $\ge 20\text{ px}$ | Mọi đối tượng có chiều cao $< 20\text{ px}$ không gán riêng, gom vào `ignore_region`. |
| **Ngưỡng ghép box (IoU match)** | $\ge 0.50$ | Hai box của 2 annotator khác nhau được coi là cùng gán một đối tượng khi $\text{IoU} \ge 0.50$. |
| **Độ trùng khớp box lớn** | $\text{IoU} \ge 0.85$ | Áp dụng cho các box có chiều cao trung bình $\ge 50\text{ px}$. |
| **Độ trùng khớp box nhỏ** | $\text{IoU} \ge 0.70$ | Áp dụng cho các box có chiều cao từ $20\text{ px}$ đến $< 50\text{ px}$. |
| **Chỉ số đồng thuận IAA toàn lô** | $\overline{\text{IoU}} \ge 0.85$ & $\ge 95\%$ class agreement | Tính trên toàn bộ các cặp box bắt cặp được giữa các annotator độc lập. |
| **Tỷ lệ lỗi theo ảnh tối đa để Release** | $\le 2.0\%$ ($0/40$ ảnh lỗi) | $1/40$ ảnh lỗi tương đương $2.5\%$ (vượt ngưỡng $2\%$). Do đó, lô 40 ảnh phải đạt $0$ ảnh lỗi để Release. |

### 5.2. Năm mã lỗi chuẩn trong phiếu QC (`qc_review.csv`)
1. `bo_sot`: Bỏ sót đối tượng hợp lệ trong ảnh (để trống `object_id`, ghi rõ mô tả vùng `region`).
2. `thua`: Gán nhãn cho vật không hợp lệ (tượng, đồ chơi, hình in, phản chiếu gương).
3. `sai_lop`: Nhầm lẫn phân loại giữa các lớp trong taxonomy (ví dụ: nhầm `truck` thành `van`, `car` thành `tuk_tuk`, `bicycle` thành `cyclo`).
4. `sai_vi_tri_kich_thuoc`: Box vẽ không ôm sát biên nhìn thấy, box quá rộng do kéo theo hàng rời, hoặc lấn ra ngoài ảnh.
5. `sai_thuoc_tinh`: Sai sót trong việc chọn giá trị `occluded_level` hoặc `truncated`.
*Ảnh không có lỗi nào:* Ghi 1 dòng với `error_type = khong`.

### 5.3. Cây quyết định bàn giao (Hold $\rightarrow$ Rework $\rightarrow$ Release)
1. **HOLD (Tạm giữ):** Guideline còn điểm mơ hồ chưa thống nhất gây ảnh hưởng đến nhãn, hoặc bộ dữ liệu có lỗi cấu trúc, thiếu file nộp. Cần tổ chức họp hiệu chuẩn trước khi xét tiếp.
2. **REWORK (Làm lại):** Tỷ lệ ảnh lỗi $> 2\%$, IAA chưa đạt chuẩn ($\overline{\text{IoU}} < 0.85$ hoặc đồng thuận lớp $< 95\%$), hoặc xuất hiện lỗi lặp lại mang tính hệ thống (một loại lỗi chiếm $\ge 50\%$ tổng số lỗi và có $\ge 3$ ca vi phạm).
3. **RELEASE (Nghiệm thu bàn giao):** Kiểm tra toàn bộ 40/40 ảnh, đạt tỷ lệ lỗi $\le 2\%$, IAA đạt chuẩn, 0 lỗi hệ thống, hồ sơ bàn giao đầy đủ bằng chứng đối soát.

### 5.4. Lịch sử phiên bản
- **`v0` (15/11/2026):** Bản khởi tạo sơ khởi của đề bài, 6 lớp cơ bản, thiếu định nghĩa ca khó và ngưỡng đo.
- **`v0.1` (16/11/2026):** Bản đề xuất cá nhân độc lập của các thành viên (Cường, Anh, Duy, Kiên, Hùng).
- **`v1.0-agreed` (16/11/2026):** Bản đồng thuận chính thức của Nhóm T005:
  - Bổ sung 4 lớp `bus`, `tuk_tuk`, `bicycle`, `cyclo` hoàn thiện taxonomy 10 lớp tương thích tuyệt đối với CVAT Local/Cloud.
  - Chuẩn hóa quy tắc hàng rời không kéo rộng box xe, quy tắc không gán hành khách trong kính xe buýt.
  - Định lượng hóa ngưỡng che khuất theo các mốc chuẩn: $0\%$, $1-30\%$, $31-70\%$, $>70\%$.
  - Chốt quyết định đồng thuận cho toàn bộ 40/40 ảnh cảnh giao thông châu Á.
