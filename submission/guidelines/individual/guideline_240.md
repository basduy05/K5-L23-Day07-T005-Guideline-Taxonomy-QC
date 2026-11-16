# Guideline gán nhãn — v0.1-individual — 2A202603240 - Phạm Mạnh Hùng

- Phiên bản: v0.1 (Bản đề xuất cá nhân)
- Ngày: 2026-11-16
- Người viết: Phạm Mạnh Hùng (MSHV: 2A202603240, Mã thành viên: 240 / Nhóm T005)
- Nhiệm vụ: Xây dựng đặc tả quy chuẩn gán nhãn độc lập sau khi trực tiếp nghiên cứu và phân tích toàn diện 24 ảnh giao thông được phân công (Task 51 / Job 51 trên hệ thống CVAT: `http://localhost:8080/tasks/51/jobs/51`).

---

## 1. Mục tiêu và Ý nghĩa An toàn

Tập dữ liệu này được xây dựng để huấn luyện và đánh giá mô hình thị giác máy tính phát hiện đối tượng 2D (Object Detection 2D) phục vụ hệ thống lái xe tự động (Autonomous Driving - AD / ADAS) và hệ thống giám sát giao thông thông minh (ITS) trong môi trường giao thông hỗn hợp phức tạp tại đô thị châu Á.

Trong bối cảnh xe tự lái vận hành thời gian thực, mọi quyết định gán nhãn đều gắn liền trực tiếp với sinh mạng con người và an toàn giao thông:
- **Bỏ sót người (`person`) hoặc người điều khiển xe máy (`motorcycle` + `person`):** Là lỗi nghiêm trọng mức độ tối cao (Critical Safety Hazard / False Negative). Người đi bộ và người đi xe máy là đối tượng tham gia giao thông yếu thế (Vulnerable Road Users - VRU). Nếu hệ thống bỏ sót đối tượng này, bộ điều khiển phanh khẩn cấp tự động (AEB) sẽ không kích hoạt, dẫn đến nguy cơ đâm va trực diện gây tử vong.
- **Nhầm lẫn phân loại phương tiện (`truck` vs `car` vs `van`):** Mỗi phân lớp phương tiện có kích thước bao vật lý, khối lượng quán tính và đặc tính động học phanh hoàn toàn khác nhau. Ví dụ: nhầm xe chuyên dụng chở xe (`img_002`) hoặc xe bán tải chở hàng cồng kềnh (`img_012`) thành xe con (`car`) sẽ khiến bộ lập quỹ đạo (Motion Planner) đánh giá sai khoảng cách an toàn khi vượt hoặc chuyển làn, gây va quệt nghiêm trọng.
- **Gán nhãn sai cho ảo ảnh quang học, tượng đài, tranh ảnh và đồ chơi (False Positive / Optical Illusion):** Khi mô hình nhận diện nhầm bóng phản chiếu trong gương/kính (`img_020`, `img_033`), tượng đài điêu khắc (`img_017`, `img_027`), poster in hình người (`img_030`, `img_033`) hoặc mô hình xe đồ chơi (`img_038`) thành chướng ngại vật thực tế trên đường, xe tự hành sẽ kích hoạt hiện tượng "Phanh ma" (Phantom Braking) - phanh gấp đột ngột giữa đường trống ở tốc độ cao, dẫn đến nguy cơ tai nạn liên hoàn từ các phương tiện di chuyển phía sau.

---

## 2. Lớp đối tượng và Thuộc tính đo lường định lượng

### 2.1 Định nghĩa 6 lớp đối tượng trong Taxonomy

Hệ thống phân loại gồm 6 lớp chuẩn hóa. Mỗi đối tượng hợp lệ chỉ thuộc đúng một lớp duy nhất:

| Lớp | Định nghĩa cốt lõi | Ví dụ hợp lệ trong ảnh giao thông | Phản ví dụ / Tuyệt đối không gán |
|:---|:---|:---|:---|
| `car` | Phương tiện cơ giới 3-4 bánh có động cơ, kết cấu thân vỏ kín hoặc bán hở dùng chở người hoặc chở khách công cộng cỡ nhỏ. | Ô tô con sedan (`img_015`, `img_023`, `img_034`), SUV (`img_012`), Hatchback, Taxi, Tuk-tuk / Auto Rickshaw (`img_004`), xe diễu hành Dangermouse có động cơ lưu thông trên lòng đường (`img_019`). | Xe bán tải chở hàng (`truck`), xe van khoang kín liền khối (`van`), xe ô tô đồ chơi/mô hình thu nhỏ (`img_038`), tượng đá tạc hình xe (`img_027`), xe con đang được chở làm hàng hóa trên rơ-moóc (`img_002`). |
| `truck` | Phương tiện cơ giới chuyên dụng chở hàng hóa, có thùng hàng riêng biệt (thùng hở, thùng lửng, thùng đông lạnh, rơ-moóc hoặc sàn phẳng). | Xe tải hạng nặng, xe chuyên dụng chở ô tô (`img_002`), xe tải thùng kín giao hàng (`img_007`), xe bán tải (pickup truck, `img_012`, `img_028`, `img_037`), xe bán tải hoán cải chở khách Songthaew (`img_008`, `img_016`). | Xe van chở hàng thân kín liền khối cabin (`van`), xe du lịch 7 chỗ (`car`), xe kéo tay ba gác thô sơ đạp chân, bia mộ đá tạc hình xe tải (`img_027`). |
| `van` | Xe ô tô thân thùng kín cỡ nhỏ đến vừa, khoang lái và khoang chở hàng gắn liền thành một khối khí động học (unibody). | Xe chở khách/chở hàng thân liền Ford Transit, Toyota HiAce (`img_016`), SsangYong/Mercedes MB100 (`img_005`), Suzuki Blind Van. | Xe bán tải gắn nắp thùng cao rời phía sau (`truck`, `img_037`), SUV gia đình (`car`), xe khách/buýt 29 chỗ trở lên (`img_009`, `img_032` - ngoài taxonomy). |
| `motorcycle` | Phương tiện cơ giới 2-3 bánh chạy động cơ đốt trong hoặc động cơ điện. | Xe máy số, xe tay ga (scooter, `img_005`, `img_008`, `img_016`, `img_030`, `img_039`), xe phân khối lớn, xe mô tô gắn thùng phụ bên hông (sidecar, `img_022`), xe máy gắn thêm thùng lôi hàng (`img_025`). | Xe đạp thông thường, xe xích lô đạp chân chở khách (`img_009`, `img_023`, `img_034`), xe đẩy hàng rong bằng tay. |
| `person` | Cơ thể con người sống ở mọi tư thế: đi bộ, chạy, đứng chờ, ngồi trên xe máy, lái xe hở mui, đạp xích lô/xe đạp. | Người đi bộ qua đường (`img_014`, `img_034`), người điều khiển xe máy (`img_005`, `img_008`), người ngồi sau xe máy, người lái và hành khách trên xe Tuk-tuk/xích lô/sidecar (`img_004`, `img_022`, `img_023`), người đứng chờ xe buýt (`img_032`). | Ma-nơ-canh nhựa trưng bày tủ kính (`img_033`), tượng đài điêu khắc bằng đồng/đá (`img_017`), hình ảnh người in trên poster quảng cáo (`img_030`, `img_033`), búp bê đồ chơi (`img_038`), linh vật thú bông mascot (`img_019`), người ngồi trong cabin đóng kín của ô tô/xe van (`img_005`). |
| `ignore_region` | Vùng không gian trong ảnh không gán chi tiết từng vật do quá xa, mờ nhòe, mật độ chen chúc vượt ngưỡng nhận diện; bỏ sót trong vùng này không tính lỗi phạt. | Đám đông đi dạo ở hậu cảnh xa có chiều cao $< 20\text{ px}$ (`img_025`), cụm xe máy đậu san sát đan xen ở hậu cảnh sâu (`img_030`), sạp hàng trưng bày đồ chơi và mô hình phức tạp (`img_038`), tủ kính cửa hàng phản chiếu hỗn loạn (`img_033`). | Bất kỳ người hoặc phương tiện nào nhìn thấy rõ ranh giới và có chiều cao $\ge 20\text{ px}$. |

*Quy ước vật thể nằm ngoài taxonomy (Out-of-Taxonomy Standard):*
- Xe buýt, xe khách cỡ lớn (`img_009`, `img_032`, `img_037`): Không thuộc taxonomy $\rightarrow$ Tuyệt đối không gán nhãn `car` hay `van` để tránh làm sai lệch phân phối kích thước phương tiện; bỏ qua hoặc khoanh `ignore_region` nếu che khuất tầm nhìn lớn.
- Xe đạp, xe xích lô đạp chân: Phương tiện thô sơ không có động cơ $\rightarrow$ Không gán nhãn phương tiện; chỉ gán nhãn `person` cho người điều khiển và hành khách ngồi trên xe.
- Động vật (chó, chim bồ câu, ngựa): Không thuộc taxonomy $\rightarrow$ Không gán box.

### 2.2 Thuộc tính đo lường định lượng (Attributes)

1. **`occluded_level` (Mức độ che khuất - đo định lượng theo diện tích hình chiếu 2D):**
   - `0`: Nhìn thấy rõ $\ge 90\%$ diện tích thân xe hoặc cơ thể người, không bị che chắn đáng kể.
   - `1`: Bị che khuất nhẹ từ $10\%$ đến $35\%$ diện tích bởi vật cản khác (cột đèn, cành cây, mép phương tiện khác).
   - `2`: Bị che khuất trung bình từ $35\%$ đến $70\%$ diện tích (vẫn nhận diện được đặc trưng chính như bánh xe, đầu xe, nửa thân người).
   - `3`: Bị che khuất nặng $> 70\%$ diện tích nhưng còn đủ manh mối đặc trưng nhận dạng (mũ bảo hiểm, góc đèn pha, một phần tay lái).
2. **`truncated` (Bị cắt cụt bởi mép viền ảnh):**
   - `no`: Toàn bộ ranh giới đối tượng nằm trọn vẹn bên trong khung ảnh.
   - `yes`: Đối tượng bị cắt cụt bởi ít nhất một mép biên ảnh $\ge 5\text{ px}$ (`img_015`, `img_016`, `img_032`, `img_039`).
3. **`can_review` (Đánh dấu trường hợp nghi ngờ / đưa ra phiên hội chẩn Calibration):**
   - `false`: Nhãn rõ ràng, tự tin tuân thủ đúng quy tắc.
   - `true`: Bật cờ khi gặp trường hợp mơ hồ, ca biên tranh chấp (ví dụ: xe độ hình thù lạ `img_019`, xe chở hàng siêu cồng kềnh `img_029`, vật thể ở ranh giới 20 px) để đưa ra hội chẩn nhóm.

---

## 3. Quy chuẩn kỹ thuật vẽ Bounding Box (Rules & Standards)

- **R-01: Nguyên tắc biên nhìn thấy thực tế (Visible Boundary Principle):**
  Bounding box phải ôm sát các pixel thực sự nhìn thấy được của vật thể. Không phóng đại hay suy đoán mở rộng box vào vùng bị che khuất hoàn toàn hoặc nằm ngoài mép ảnh. Tiếp xúc đất phải lấy chính xác tại mép dưới của lốp xe tiếp xúc với mặt đường; tuyệt đối không kéo box xuống bóng đổ (shadow) hoặc bóng phản chiếu trên mặt đường ướt (`img_015`).
- **R-02: Quy tắc tách biệt Người và Xe máy (Coupling & Overlap Rule):**
  Khi có người điều khiển hoặc hành khách ngồi trên xe máy, bắt buộc vẽ **2 box riêng biệt** chồng lấn lên nhau:
  - Box `person`: Ôm trọn vẹn cơ thể người từ đỉnh đầu (hoặc đỉnh mũ bảo hiểm) đến gót chân/bàn chân đặt trên bàn đạp.
  - Box `motorcycle`: Ôm toàn bộ kết cấu cơ học của xe máy từ tay lái/gương chiếu hậu đến điểm tiếp đất của bánh xe.
- **R-03: Quy tắc hàng hóa phương tiện chở tải:**
  - Hàng hóa nằm trong thùng xe tải / xe bán tải (`img_012`, `img_028`): Box `truck` ôm cả thân xe và toàn bộ khối hàng hóa chuyên chở, vì khối hàng hóa này cấu thành thể tích cản trở vật lý của phương tiện.
  - Xe máy chở hàng hóa cồng kềnh: Box `motorcycle` ôm trọn khung xe máy và khối hàng gắn chặt trên xe; box `person` ôm phần cơ thể nhìn thấy của người lái.
- **R-04: Quy tắc xe chuyên dụng chở xe (Car Carrier Rule — `img_002`):**
  - Gán 1 box `truck` lớn bao trọn toàn bộ xe đầu kéo, rơ-moóc chuyên dụng và toàn bộ dàn xe con được chuyên chở trên các tầng giá đỡ.
  - **Tuyệt đối KHÔNG gán nhãn `car` cho các ô tô con nằm trên rơ-moóc:** Các xe này là hàng hóa tĩnh (cargo), không tiếp đất, không có động lực học độc lập. Gán `car` sẽ khiến hệ thống nhận thức 3D hiểu sai là có hàng loạt xe con đang lơ lửng trên không trung, gây nhiễu loạn module dự báo chuyển động (Motion Planning).
  - Người đứng điều khiển/kiểm tra cạnh xe: Gán box `person` độc lập.
- **R-05: Quy tắc xe gắn thùng phụ (Sidecar `img_022` / Xe lôi `img_025`):**
  - Gán 1 box `motorcycle` duy nhất bao quát toàn bộ xe máy và thùng phụ bên hông.
  - Gán các box `person` riêng biệt cho người lái mô tô và từng hành khách ngồi trong thùng phụ.
- **R-06: Quy tắc người ngồi trong cabin xe kín:**
  - Người ngồi bên trong cabin đóng kín của ô tô (`car`), xe tải (`truck`), xe van (`van`): **KHÔNG gán nhãn `person`** (kể cả nhìn thấy mờ qua kính chắn gió, `img_005`). Box phương tiện đã bảo vệ toàn diện không gian này.
  - CHỈ gán `person` cho người trên xe trong trường hợp buồng lái hở hoàn toàn, mui trần (`img_019`), xe Tuk-tuk (`img_004`), xe máy (`img_005`), xích lô (`img_023`), hoặc người ngồi trên thùng hở phía sau xe bán tải/Songthaew (`img_016`).
- **R-07: Quy tắc loại trừ vật thể ảo, tượng đài, tranh ảnh và đồ chơi (Anti-Hallucination Rule):**
  - Tượng đài nghệ thuật bằng đá/đồng (`img_017`, `img_027`): KHÔNG gán nhãn `person` hay `truck`.
  - Hình phản chiếu qua gương chiếu hậu hoặc vách kính (`img_020`, `img_033`): KHÔNG gán nhãn phương tiện hay người.
  - Ma-nơ-canh trong tủ kính thời trang (`img_033`): KHÔNG gán nhãn `person`.
  - Tranh ảnh chân dung in 2D trên poster, biển quảng cáo (`img_030`, `img_033`): KHÔNG gán nhãn `person`.
  - Mô hình ô tô đồ chơi, búp bê trong tủ kính (`img_038`): KHÔNG gán nhãn `car` hay `person`; khoanh toàn bộ tủ kính vào `ignore_region`.
- **R-08: Ngưỡng kích thước tối thiểu (Minimum Size Threshold):**
  Đối tượng có chiều cao hình chiếu $< 20\text{ px}$ không được gán box đơn lẻ. Nếu nằm trong cụm đông đúc, gom vào 1 box `ignore_region`.
- **R-09: Quy tắc vật dụng mang theo của con người:**
  Ba lô đeo sau lưng, túi xách cầm tay, nón bảo hiểm, nón lá (`img_014`, `img_034`) được tính là một phần cơ thể người và nằm trọn vẹn trong bounding box `person`.
- **R-10: Vết nhòe chuyển động (Motion Blur Rule — `img_014`):**
  Với các đối tượng di chuyển tốc độ cao bị nhòe chuyển động: nếu còn nhận diện được đường bao tương đối và kích thước $\ge 20\text{ px}$, vẽ box ôm sát vệt mờ có mật độ nhận dạng cao và gán `occluded_level = 2` hoặc `3`; nếu nhòe thành vệt sáng vô định hình thì bỏ qua hoặc đưa vào `ignore_region`, bật cờ `can_review = true`.

---

## 4. Phân tích chi tiết 24 Ca khó thực tế trong Task 51 / Job 51

Dưới đây là bảng phân tích toàn diện 24 ảnh trong nhiệm vụ được phân bổ của Phạm Mạnh Hùng (Task 51 / Job 51), chỉ rõ hiện tượng quan sát, mã quy tắc, quyết định gán nhãn và cơ sở an toàn cho xe tự lái:

| STT | Mã ảnh | Hiện tượng quan sát thực tế trong ảnh | Rule ID | Đề xuất quyết định gán nhãn | Cơ sở lý luận & Ý nghĩa an toàn cho xe tự lái |
|:---:|:---|:---|:---|:---|:---|
| 1 | `img_002.jpg` | Xe tải Renault 380 chuyên dụng chở ô tô 2 tầng, trên giá đỡ chở 7-8 xe con; có 1 người đứng cạnh đuôi xe. | R-03, R-04 | Gán 1 box `truck` bao toàn bộ đầu kéo, rơ-moóc và dàn xe con chở trên giá. **KHÔNG gán box `car` riêng** cho các xe con trên rơ-moóc. Gán 1 box `person` cho người đứng bên phải (`occluded_level = 0`). | Dàn xe con là hàng hóa tĩnh cố định, không tiếp đất và không di chuyển độc lập. Gán `car` sẽ khiến hệ thống ADAS hiểu nhầm có xe con đang bay lơ lửng, gây xung đột mô-đun 3D Bounding Box và Motion Planning. |
| 2 | `img_004.jpg` | 2 xe Tuk-tuk xanh có biển hiệu TAXI đậu trên vỉa hè ban đêm; tài xế ngồi trong cabin mở; phía trước có cọc tiêu giao thông. | R-01, R-06 | Gán 2 box `car` cho thân 2 xe Tuk-tuk. Gán 2 box `person` cho 2 tài xế ngồi bên trong buồng lái mở (`occluded_level = 1` và `2`). Cọc tiêu giao thông bỏ qua (ngoài taxonomy). | Tuk-tuk là phương tiện cơ giới 3 bánh chở khách công cộng có mui che, tương đương taxi cỡ nhỏ nên xếp vào `car`. Vì buồng lái hở, tài xế có nguy cơ va quệt trực tiếp nên bắt buộc phải gán `person`. |
| 3 | `img_005.jpg` | Dòng người lái xe máy ken đặc tại ngã tư TP.HCM; hậu cảnh có xe van SsangYong MB100; người trong cabin xe van nhìn thấy qua kính. | R-01, R-02, R-06 | Gán cặp box `person` và `motorcycle` chồng lấn cho từng người lái xe máy. Gán 1 box `van` cho xe MB100. **KHÔNG gán `person`** cho tài xế/hành khách trong khoang kín xe van. | Phân định rõ 2 box riêng cho người và xe máy để bảo vệ VRU. Người ngồi trong cabin kín đã được khung vỏ xe che chắn, không gán `person` để tránh kích hoạt phanh khẩn cấp sai mục tiêu. |
| 4 | `img_007.jpg` | Xe tải thùng Isuzu ELF và xe bán tải hoán cải thùng đông lạnh đậu trong kho hàng ban đêm Pattaya; 1 người đứng trong bóng tối. | R-01, R-03 | Gán `truck` cho xe tải thùng Isuzu (`truncated = yes`). Gán `truck` cho xe bán tải gắn thùng hàng riêng. Gán `person` cho người đứng giữa 2 xe ở hậu cảnh tối (`occluded_level = 3`). | Cả 2 xe đều có cấu trúc khoang chở hàng riêng biệt đạt chuẩn `truck`. Người đứng trong bóng tối dù bị che khuất nặng $> 70\%$ vẫn cần được nhận diện để phòng ngừa xe tự hành lùi trúng. |
| 5 | `img_008.jpg` | Chợ đêm Pattaya sầm uất; hàng chục xe máy đậu san sát; tài xế áo cam phản quang; xe Songthaew chở khách phía xa. | R-01, R-02, R-08 | Gán box `motorcycle` cho từng xe máy nhìn rõ ranh giới $\ge 20\text{ px}$. Gán cặp box `person` + `motorcycle` cho tài xế áo cam. Gán `truck` cho xe Songthaew phía xa. | Xe máy đỗ sát nhau vẫn phải gán riêng từng box để mô hình học cách phân tách cá thể (instance segmentation/detection) trong cảnh tắc nghẽn đô thị châu Á. |
| 6 | `img_009.jpg` | Ngã tư đường phố Huế; xe khách/buýt 29 chỗ (Hyundai County 75K-2944) rẽ ngang; người đi xe máy; chó chạy qua đường. | R-01, R-02 | **KHÔNG gán box** cho xe khách 29 chỗ (hoặc khoanh `ignore_region`). Không gán box cho con chó. Gán các box `person` và `motorcycle` cho người và xe máy xung quanh. | Xe khách lớn nằm ngoài taxonomy (không thuộc car/truck/van). Không được tự ý ép vào lớp khác làm méo mó đặc trưng hình học của mô hình. Động vật không thuộc taxonomy. |
| 7 | `img_012.jpg` | Xe bán tải thùng hở chất đầy bao rau củ quả, thùng carton đỗ trên phố hẹp; bên cạnh có xe SUV Mazda CX-5; phía trước có xe tải mở cửa. | R-01, R-03 | Gán 1 box `truck` ôm trọn cả thân xe bán tải và toàn bộ các bao hàng/thùng carton trên thùng hở. Gán 1 box `car` cho xe Mazda CX-5. Gán `truck` cho xe tải mở thùng phía trước. | Hàng hóa trên thùng xe bán tải làm tăng kích thước thực tế của xe. Box phải bao cả hàng hóa để xe tự lái không cọ quẹt vào hàng khi lách qua đường hẹp. |
| 8 | `img_014.jpg` | 2 người đi bộ băng qua đường trên vạch kẻ giữa dòng xe máy chạy tốc độ cao bị nhòe chuyển động (Motion Blur) tại Sài Gòn. | R-01, R-09, R-10 | Gán 2 box `person` rõ nét cho 2 người đi bộ (túi xách của người phụ nữ tính gộp vào box). Các vệt xe máy nhòe chuyển động $\ge 20\text{ px}$ gán box với `occluded_level = 2` hoặc `3`, tick `can_review`. | Người đi bộ qua đường là VRU có độ ưu tiên an toàn cao nhất. Vệt nhòe chuyển động phản ánh thực tế camera xe tự lái khi chụp ở tốc độ màn trập thấp hoặc trời tối. |
| 9 | `img_015.jpg` | Đường ướt sau mưa, 2 xe ô tô con ở tiền cảnh bị cắt mép mạnh; mặt đường phản chiếu vệt đèn thẳng đứng; nhiều ô tô đỗ bên kia đường. | R-01 | Gán 2 box `car` cho 2 xe ở tiền cảnh (`truncated = yes`). Box chỉ lấy đến mép lốp tiếp đất, **không bao gồm vệt bóng phản chiếu trên mặt đường**. Gán `car` cho các xe đỗ bên đường. | Bóng xe phản chiếu trên mặt đường ướt là ảo ảnh quang học. Kéo box xuống bóng phản chiếu sẽ làm mô hình ước tính sai chiều cao và mặt phẳng tiếp đất (ground plane). |
| 10 | `img_016.jpg` | Phố Bangkok nhộn nhịp: xe Vespa chở cuộn dây thép, xe bán tải hoán cải chở người trong thùng sau, xe Tuk-tuk xanh lá, xe van HiAce. | R-02, R-03, R-06 | Gán `motorcycle` cho xe Vespa (ôm cả cuộn dây thép) + box `person` cho tài xế. Gán `truck` cho xe bán tải + box `person` cho người trong thùng hở. Gán `car` cho Tuk-tuk. Gán `van` cho HiAce. | Phân loại chuẩn xác 4 phương tiện đặc trưng châu Á. Người ngồi trên thùng hở không có đai an toàn là đối tượng cực kỳ dễ bị tổn thương, bắt buộc phải có box `person`. |
| 11 | `img_017.jpg` | Tượng đài Công tước Wellington bằng đồng cưỡi ngựa tại Glasgow, đầu đội chồng nón cọc tiêu giao thông; bồ câu đậu trên lưng ngựa. | R-07 | **TUYỆT ĐỐI KHÔNG GÁN BOX `person`** cho tượng đồng cưỡi ngựa. Không gán box cho con ngựa, cọc tiêu và chim bồ câu (ngoài taxonomy). | Tượng đồng nghệ thuật là vật thể tĩnh vĩnh cửu. Gán nhãn `person` sẽ khiến xe tự lái kích hoạt phanh gấp vô căn cứ (Phantom Braking) khi di chuyển gần các quảng trường công cộng. |
| 12 | `img_019.jpg` | Xe diễu hành độc dị hình chuột Danger Mouse màu vàng mang biển số YR04 NUM, có tài xế lái bên trong và mô hình đầu chuột khổng lồ; khán giả đông đúc. | R-01, R-06, R-07, R-08 | Gán 1 box `car` cho toàn bộ thân xe Danger Mouse. Gán 1 box `person` cho tài xế ngồi trong buồng lái mở. **KHÔNG gán `person` cho mô hình đầu chuột**. Khán giả nhìn rõ $\ge 20\text{ px}$ gán `person`. | Dù hình dáng dị biệt, xe vẫn có 4 bánh và khung gầm cơ giới lưu thông trên đường nên phải coi là `car`. Đầu chuột là mô hình xốp/nhựa, không phải người sống. |
| 13 | `img_022.jpg` | Xe mô tô cổ 3 bánh gắn thùng phụ chở khách bên hông (Sidecar) di chuyển trên đường rừng; 1 người cầm lái và 1 người ngồi trong thùng phụ. | R-02, R-05 | Gán 1 box `motorcycle` bao quát cả xe mô tô và thùng phụ bên cạnh. Gán 2 box `person` độc lập: 1 cho người điều khiển mô tô và 1 cho hành khách trong thùng phụ. | Thùng phụ gắn cố định vào xe máy tạo thành một khối cơ học đồng nhất. 2 người trên xe có tư thế và vùng chuyển động khác nhau nên phải theo dõi riêng biệt. |
| 14 | `img_023.jpg` | Xe xích lô đạp chân chở khách tại Huế đi trước xe ô tô sedan Honda Civic màu đen; người đi bộ và xe máy xung quanh. | R-01, R-02 | **KHÔNG gán box phương tiện** cho xe xích lô. Gán 2 box `person`: 1 cho người đạp xích lô phía sau, 1 cho du khách ngồi phía trước. Gán 1 box `car` cho xe Honda Civic. | Xích lô đạp chân là phương tiện thô sơ không có động cơ. Phân tách rõ 2 con người trên xích lô để hệ thống nhận diện nguy cơ va chạm đối với cả hai. |
| 15 | `img_025.jpg` | Đường đi dạo bờ biển Pattaya với người đi bộ, người ngồi ghế; xe máy gắn thùng lôi bên hông chạy trên đường; đám đông xa xôi. | R-05, R-08 | Gán `person` cho người đi bộ cự ly gần $\ge 20\text{ px}$. Đám đông người dạo bộ ở hậu cảnh xa xôi ($< 20\text{ px}$) khoanh 1 box `ignore_region`. Gán `motorcycle` ôm trọn cả xe máy và thùng lôi. | Ngưỡng 20 px giúp loại trừ các đối tượng quá nhỏ không đủ đặc trưng trích xuất; `ignore_region` giúp tránh phạt lỗi bỏ sót đối tượng ở khoảng cách quá xa. |
| 16 | `img_027.jpg` | Bia mộ điêu khắc hình xe bán tải cổ bằng đá granite đen bóng nguyên khối tại nghĩa trang Oak Hill Memorial Park. | R-07 | **TUYỆT ĐỐI KHÔNG GÁN BOX `truck`** cho bia mộ đá tạc hình xe. Bỏ qua không gán box hoặc khoanh `ignore_region`. | Bia mộ tạc bằng đá là vật thể tĩnh đặt trên bãi cỏ nghĩa trang, không có bánh xe quay, không có động cơ và không lưu thông trên đường. Gán `truck` sẽ làm ô nhiễm nặng dữ liệu nhận diện xe tải. |
| 17 | `img_028.jpg` | Xe bán tải cổ màu nâu chở một bức tượng ngựa trắng lớn trong thùng xe đỗ bên đường; xe ô tô con màu đỏ đỗ đối diện. | R-01, R-03, R-07 | Gán 1 box `truck` bao gồm toàn bộ thân xe bán tải và bức tượng ngựa trắng trong thùng xe. Không gán nhãn riêng cho tượng ngựa. Gán 1 box `car` cho xe ô tô đỏ phía sau. | Tượng ngựa là hàng hóa cứng gắn chặt vào thùng xe, định hình chiều cao va chạm tối đa của xe bán tải. Bounding box phải bao trọn tượng ngựa để cảnh báo va chạm gầm cầu/chướng ngại vật trên cao. |
| 18 | `img_030.jpg` | Bãi đỗ xe máy dày đặc tại Sài Gòn; người bảo vệ áo xanh đi giữa các xe; áp phích quảng cáo in hình chân dung ca sĩ trên cây. | R-01, R-07, R-08 | Gán box `motorcycle` cho từng xe máy ở hàng trước nhìn rõ. Cụm xe máy chen chúc chồng chất phía sau khoanh `ignore_region`. Gán `person` cho bảo vệ áo xanh. **KHÔNG gán `person` cho ảnh ca sĩ trên poster**. | Ảnh in 2D trên biển bạt quảng cáo là bẫy nhận diện thị giác kinh điển. Nếu gán `person`, xe tự lái sẽ phanh gấp trước mọi biển quảng cáo ven đường. |
| 19 | `img_032.jpg` | Trạm dừng xe buýt giờ cao điểm tại Canada; hơn 20 người xếp hàng chờ xe; người đứng sau vách kính nhà chờ; xe ô tô xanh ở góc trái; xe buýt đỗ. | R-01, R-06, R-08 | Gán box `person` cho từng người đứng xếp hàng $\ge 20\text{ px}$ (kể cả người nhìn qua vách kính trong suốt). Gán `car` cho xe xanh (`truncated = yes`). Xe buýt bỏ qua (ngoài taxonomy). | Vách kính nhà chờ xe buýt trong suốt cho phép quan sát rõ dáng người nên vẫn gán `person`. Trạm xe buýt là khu vực tập trung đông người, đòi hỏi độ nhạy nhận diện VRU tối đa. |
| 20 | `img_033.jpg` | Tủ kính cửa hàng thời trang: ma-nơ-canh khỏa thân, poster ảnh người mẫu thời trang cỡ lớn, vách kính phản chiếu hình ảnh xe Land Rover đỗ ngoài đường. | R-07 | **TUYỆT ĐỐI KHÔNG GÁN BẤT KỲ BOX NÀO** (`person` hay `car`) cho ma-nơ-canh, ảnh người mẫu hay hình xe phản chiếu qua kính. Có thể khoanh toàn bộ tủ kính vào 1 box `ignore_region`. | Ca bẫy 3 trong 1: ma-nơ-canh (vật nhân tạo), poster ảnh in (2D), hình phản chiếu xe (ảnh ảo). Gán nhãn bất kỳ yếu tố nào ở đây đều tạo ra nhiễu loạn tai hại cho mạng nơ-ron nhận thức xe tự lái. |
| 21 | `img_034.jpg` | Nam thanh niên đi bộ băng qua ngã tư rộng lúc hoàng hôn tại Huế; người đi xe máy đeo ba lô to; người đi xe đạp; xe ô tô con màu đen. | R-01, R-02, R-09 | Gán `person` cho nam thanh niên đi bộ. Gán `person` (bao gồm cả ba lô to đeo lưng) và `motorcycle` cho người lái xe máy. Gán `person` cho người đi xe đạp (không gán xe đạp). Gán `car` cho ô tô đen. | Ba lô đeo trên người thay đổi thể tích thực tế của người đi bộ/xe máy, bắt buộc phải tính vào box `person`. Người đi xe đạp là VRU cần được bảo vệ dù xe đạp nằm ngoài taxonomy. |
| 22 | `img_037.jpg` | Xe bán tải cứu hộ GO Transit lắp nắp thùng cao (Master 75 Truck Cap) đỗ cạnh xe buýt; 1 người đi bộ bên cạnh. | R-01, R-03 | Gán 1 box `truck` bao toàn bộ thân xe bán tải và nắp thùng cap cao Master 75 phía sau. Gán 1 box `person` cho người đi bộ. Xe buýt GO Transit bỏ qua (ngoài taxonomy). | Dù lắp nắp thùng kín, đây vẫn là khung gầm xe bán tải chuyên dụng (khoang lái tách rời thùng xe) nên xếp vào `truck`. Bounding box phải bao trọn nắp cap để định hình chuẩn thể tích xe. |
| 23 | `img_038.jpg` | Tủ kính cửa hàng đồ chơi ở Munich trưng bày hàng trăm mô hình xe ô tô thu nhỏ (Ferrari, Porsche...) cùng búp bê, thú bông, chuột Mickey. | R-07, R-08 | **TUYỆT ĐỐI KHÔNG GÁN NHÃN `car` HOẶC `person`** cho các mô hình đồ chơi thu nhỏ. Khoanh toàn bộ tủ kính vào 1 box `ignore_region` duy nhất. | Mô hình xe đồ chơi và búp bê trong tủ kính không tham gia giao thông. Nếu gán nhãn sẽ làm sụp đổ hoàn toàn khả năng phân biệt tỷ lệ kích thước vật lý của mạng nơ-ron nhận thức. |
| 24 | `img_039.jpg` | Phố ẩm thực xứ Huế: xe máy đỗ trong hiên nhà bị cắt mép; 3 phụ nữ đi bộ trên vỉa hè; người lái xe máy; phụ nữ đi xe đạp; ô tô trắng đỗ xa. | R-01, R-02, R-08 | Gán `motorcycle` cho xe máy đỗ trong hiên nhà (`truncated = yes`). Gán 3 box `person` riêng cho 3 phụ nữ đi bộ. Gán cặp box `person` + `motorcycle` cho người lái xe máy. Gán `person` cho người đi xe đạp. | Cảnh giao thông đô thị điển hình của Việt Nam với đầy đủ tương tác giữa người đi bộ, xe máy, xe thô sơ và xe cơ giới; đòi hỏi ranh giới box phải ôm sát từng cá thể riêng biệt. |

---

## 5. Quy trình Kiểm soát Chất lượng (QC) và Lịch sử Phiên bản

### 5.1 Tiêu chí nghiệm thu định lượng (Acceptance Thresholds)

Để đảm bảo tính nhất quán cao nhất giữa các thành viên gán nhãn độc lập trong nhóm T005, bộ chỉ số định lượng sau đây được thiết lập:
- **Độ trùng khớp hình học (Geometric IoU):**
  - Đối với vật thể lớn (chiều cao hình chiếu $\ge 50\text{ px}$): Chỉ số IoU giữa 2 annotator độc lập phải đạt $\ge 0.85$.
  - Đối với vật thể nhỏ (chiều cao hình chiếu từ $20\text{ px}$ đến $< 50\text{ px}$): Chỉ số IoU phải đạt $\ge 0.70$.
- **Độ đồng thuận phân lớp (Classification Agreement):** Tỉ lệ thống nhất nhãn lớp trên các cặp bounding box bắt cặp thành công phải đạt $\ge 95\%$.
- **Tỉ lệ lỗi tối đa cấp độ ảnh:** Tỉ lệ ảnh có sai sót nghiêm trọng (bỏ sót VRU, gán sai lớp cơ bản) không vượt quá $2.0\%$ toàn bộ tập dữ liệu.
- **Quy trình giải quyết tranh chấp (Dispute Resolution):**
  - Trong quá trình gán nhãn độc lập (Pha 3), mọi trường hợp lưỡng lự hoặc ca biên chưa được quy định rõ trong guideline đều phải bật cờ `can_review = true`.
  - Toàn bộ các ca có gắn cờ `can_review` và các cặp nhãn có IoU $< 0.85$ hoặc lệch nhãn lớp sẽ được đưa vào phiên thảo luận hiệu chuẩn nhóm (Calibration Session, Pha 5) để biểu quyết thống nhất quy tắc và cập nhật vào guideline nhóm `v1.1`.

### 5.2 Lịch sử phiên bản

| Phiên bản | Ngày | Người thực hiện | Nội dung thay đổi cốt lõi | Lý do / Cơ sở từ các ca thực tế |
|:---:|:---:|:---|:---|:---|
| `v0` | 2026-11-16 | Giảng viên | Bản khung sơ khởi: định nghĩa sơ lược 6 lớp, chưa có ngưỡng định lượng, chưa có quy tắc ca khó. | Cung cấp tài liệu xuất phát điểm cho lớp học. |
| `v0.1-individual` | 2026-11-16 | Phạm Mạnh Hùng (2A202603240 / Mã 240) | **Đề xuất cá nhân hoàn chỉnh:**<br>1. Thiết lập 10 quy chuẩn kỹ thuật (R-01 đến R-10) giải quyết triệt để bài toán an toàn xe tự lái.<br>2. Định lượng hóa 100% thuộc tính `occluded_level` (4 mức theo % diện tích) và `truncated` ($\ge 5\text{ px}$).<br>3. Quy định dứt khoát không gán xe trên rơ-moóc (`img_002`), không gán người trong cabin kín (`img_005`), phân loại Tuk-tuk vào `car` (`img_004`), phân loại xe bán tải vào `truck` (`img_012`, `img_028`, `img_037`).<br>4. Thiết lập quy tắc loại trừ nghiêm ngặt đối với tượng đài (`img_017`, `img_027`), đồ chơi (`img_038`), ma-nơ-canh & ảnh phản chiếu (`img_033`), poster quảng cáo (`img_030`) để ngăn chặn phanh ma (Phantom Braking).<br>5. Lập bảng phân tích chi tiết toàn bộ 24 ảnh trong Task 51 / Job 51. | Nghiên cứu trực tiếp và sâu sắc 24 ảnh thực tế thuộc nhiệm vụ được giao trên hệ thống CVAT. |
