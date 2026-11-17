# Guideline gán nhãn dữ liệu — Bản cá nhân độc lập (Pha 1 · CP1)

- **Người thực hiện:** ĐỖ HUY KIÊN
- **MSHV:** `2A202603216`
- **Nhóm:** T005
- **Phiên bản:** `v0.1-draft-DoHuyKien` (Bản độc lập, không tham khảo bài của thành viên khác)
- **Ngày hoàn thành:** 2026-11-16
- **Công cụ áp dụng:** CVAT (Computer Vision Annotation Tool), xuất định dạng COCO 1.0

---

## 1. Mục tiêu và Bối cảnh An toàn

### 1.1. Mục tiêu ứng dụng
Tập dữ liệu bao gồm 40 ảnh chụp quang cảnh giao thông thực tế tại các đô thị châu Á (Việt Nam, Thái Lan, Ấn Độ, Đài Loan...) với đặc trưng giao thông hỗn hợp, mật độ dày đặc và nhiều phương tiện phi quy chuẩn. Nhãn Bounding Box (hộp chữ nhật) được gán trong dự án này dùng để huấn luyện mô hình thị giác máy tính phục vụ:
1. **Hệ thống xe tự hành (Autonomous Vehicles - AV):** Định vị vật cản, dự đoán quỹ đạo di chuyển và lập kế hoạch phanh/tránh an toàn trong thời gian thực.
2. **Camera giám sát giao thông thông minh (ITS / Traffic Monitoring):** Đếm lưu lượng xe, phát hiện ùn tắc, nhận diện hành vi vi phạm giao thông.

### 1.2. Phân tích rủi ro an toàn (Safety Hazards)
Trong bối cảnh thị giác máy tính cho giao thông, sai sót gán nhãn dẫn đến các hiểm họa nghiêm trọng:
- **Nguy cơ từ Bỏ sót (False Negative):** Nếu người gán bỏ sót người đi bộ khuất một phần, người đi xe máy len lỏi hoặc xe dừng đỗ ven đường, mô hình học máy sẽ coi vùng đó là "đường trống" (free space). Xe tự hành sẽ không giảm tốc hoặc không phanh, dẫn đến tai nạn đâm va trực diện.
- **Nguy cơ từ Gán nhầm vật giả (False Positive - Toxic Gradients):** Nếu gán nhãn các đối tượng giả tạo như hình người/xe in trên biển quảng cáo (billboard), ma-nơ-canh trong tiệm thời trang, tượng đài bằng đá hoặc hình phản chiếu qua gương xe là vật thể thật, xe tự hành sẽ gặp hiện tượng "phanh ma" (phantom braking) giữa đường cao tốc, gây nguy cơ tai nạn liên hoàn cho các phương tiện phía sau.
- **Nguy cơ từ Sai phân loại lớp (Misclassification):** Gán nhầm xe tải chở hàng cồng kềnh thành xe con làm hệ thống ước tính sai kích thước vật lý, trọng tâm và khoảng cách phanh an toàn.

---

## 2. Danh mục Lớp và Thuộc tính Đối tượng (Taxonomy)

### 2.1. Danh mục Lớp (Classes)

Hệ thống tuân thủ 6 lớp khởi đầu theo `submission/taxonomy.json`, được định nghĩa chuẩn hóa kèm ví dụ và phản ví dụ thực tế:

| Lớp | Định nghĩa chuẩn | Ví dụ điển hình trong 40 ảnh | Phản ví dụ / Dễ nhầm lẫn | Quy tắc biên & Xử lý |
|---|---|---|---|---|
| `car` | Phương tiện giao thông cơ giới 4 bánh trở lên chở người (4–9 chỗ), gồm sedan, hatchback, SUV, crossover, ô tô con gia đình, taxi. | Xe sedan đen trong `img_024.jpg`, xe taxi, SUV trong `img_040.jpg`. | **Không phải:** Xe tải thùng kín (`van`), xe bán tải thùng hàng hở lớn (`truck`), tượng đá xe ô tô (`img_027.jpg`), xe đồ chơi diễu hành (`img_019.jpg`). | Gán ôm sát toàn bộ thân xe nhìn thấy (kể cả cản trước, cản sau, gương chiếu hậu). |
| `truck` | Xe tải cơ giới từ 1.5 tấn trở lên hoặc xe có cabin điều khiển tách biệt với thùng chở hàng phía sau. | Xe chuyên dụng chở ô tô (`img_002.jpg`), xe tải chở hàng ban đêm (`img_007.jpg`), xe tải nhỏ thùng hở (`img_036.jpg`). | **Không phải:** Xe bán tải dân dụng chở đồ lặt vặt (`img_001.jpg`, `img_012.jpg` $\rightarrow$ xem ca khó bên dưới), xe thùng kín liền khối (`van`). | Box ôm sát cabin và thùng xe cơ học. Không vẽ trùm lên phần hàng hóa nhô cao/xòe ra ngoài thùng xe. |
| `van` | Xe ô tô thân liền khối (monocoque), khoang chở người và khoang chở hàng nằm chung dưới một mái xe, cửa hông dạng trượt hoặc mở ngang. | Xe cứu thương, xe microbus, xe đưa đón học sinh/chở hàng cỡ nhỏ thùng kín. | **Không phải:** Xe bán tải có nắp chụp thùng sau (`img_037.jpg`), xe buýt cỡ lớn 45 chỗ (`img_032.jpg`). | Nếu khó phân biệt giữa `van` và `car` (SUV to) hoặc `truck`, ưu tiên xem khoang cabin và thùng xe có vách liền không. |
| `motorcycle`| Phương tiện 2 bánh hoặc 3 bánh gắn động cơ đốt trong/động cơ điện, bao gồm xe số, xe tay ga, mô tô phân khối lớn, xe có thùng bên (sidecar). | Hàng trăm xe máy trong `img_005.jpg`, `img_008.jpg`, `img_026.jpg`, mô tô sidecar trong `img_022.jpg`. | **Không phải:** Xe đạp thông thường, xe xích lô chở khách (`img_006.jpg`, `img_009.jpg`, `img_023.jpg`, `img_034.jpg`). | Box ôm sát từ bánh trước đến đuôi xe, ghi-đông xe. Tách rời với box người lái. |
| `person` | Con người sinh học có thật trong không gian thực tế, bao gồm người đi bộ, người đứng, ngồi, nằm, người điều khiển phương tiện hoặc hành khách. | Người đi bộ qua đường trong `img_014.jpg`, người lái xe máy trong `img_005.jpg`, người bán hàng rong `img_003.jpg`, người mặc đồ cosplay `img_011.jpg`. | **Không phải:** Ma-nơ-canh (`img_033.jpg`), tượng danh nhân (`img_017.jpg`), chân dung in trên billboard quảng cáo (`img_018.jpg`), bóng phản chiếu trong gương (`img_020.jpg`). | Ôm sát đỉnh đầu đến gót chân (hoặc phần cơ thể nhìn thấy). Không gán người ngồi kín bên trong ô tô/xe tải nếu cửa kính tối màu. |
| `ignore_region` | Vùng hình chữ nhật đánh dấu khu vực không gán chi tiết từng đối tượng, bỏ sót vật thể trong vùng này không bị coi là lỗi QC. | Đám đông dày đặc ở cự ly xa trên bãi biển (`img_025.jpg`), khu chợ đêm lùi sâu trong ngõ tối (`img_008.jpg`), bến xe buýt hỗn loạn cự ly xa (`img_032.jpg`). | **Không phải:** Các vật thể đơn lẻ đứng gần camera nhìn rõ hình dáng $> 20\text{ px}$. | Vẽ 1 bounding box bao quanh cụm vật thể nhỏ/che khuất nặng. |

### 2.2. Xử lý các vật thể ngoài Taxonomy (Out-of-Taxonomy)
Trong 40 ảnh khảo sát xuất hiện các phương tiện không thuộc 6 lớp trên:
1. **Xe xích lô đạp chân / Cyclo (`img_006.jpg`, `img_009.jpg`, `img_023.jpg`, `img_034.jpg`, `img_039.jpg`):**
   - *Đặc điểm:* Là phương tiện thô sơ 3 bánh do người đạp chân điều khiển.
   - *Quy tắc độc lập của Kiên:* **Không gán box phương tiện** cho xe xích lô vì taxonomy không có lớp `bicycle` hay `cyclo`. Tuy nhiên, **BẮT BUỘC gán box `person`** cho người đạp xích lô và khách ngồi trên xe nếu nhìn thấy rõ.
2. **Xe buýt cỡ lớn / Transit Bus (`img_032.jpg`):**
   - *Đặc điểm:* Xe chở khách cỡ lớn công cộng, không phải `car` cũng không phải `truck`.
   - *Quy tắc độc lập của Kiên:* Tạm thời vẽ box lớp `ignore_region` bao quanh xe buýt lớn này để tránh phạt mô hình, và đánh dấu `can_review = true` để đưa ra biểu quyết ở buổi họp nhóm Pha 2 (xem xét bổ sung lớp `bus` vào taxonomy v1.0).
3. **Xe ba bánh chở khách (Tuk-tuk / Auto-rickshaw trong `img_004.jpg`, `img_035.jpg`):**
   - *Đặc điểm:* Phương tiện 3 bánh đặc thù châu Á có động cơ, cabin liền khung chở khách.
   - *Quy tắc độc lập của Kiên:* Tạm thời xếp vào `car` (vì có động cơ, chở khách, kích thước chiếm làn đường tương đương xe ô tô mini) kèm tick `can_review = true` để thảo luận nhóm.

---

### 2.3. Quy định Thuộc tính với Ngưỡng đo lường Cụ thể (Attributes)

Để tránh bất đồng chủ quan giữa các thành viên, các mức thuộc tính được định lượng hóa bằng tỷ lệ phần trăm diện tích:

#### 1. Thuộc tính `occluded_level` (Mức độ che khuất)
Đo bằng tỷ lệ diện tích bề mặt đối tượng bị che khuất bởi vật thể khác nằm phía trước:
- **Mức `0` (Không bị che):** Nhìn thấy rõ $\ge 90\%$ diện tích cơ thể hoặc thân xe. Vật thể đứng độc lập, không bị cản trở.
- **Mức `1` (Che khuất nhẹ):** Bị che khuất từ $10\%$ đến dưới $35\%$ diện tích. Ví dụ: Bánh xe máy bị che bởi mép vỉa hè, chân người đi bộ bị che bởi gờ dải phân cách.
- **Mức `2` (Che khuất vừa):** Bị che khuất từ $35\%$ đến dưới $70\%$ diện tích. Ví dụ: Nửa thân xe bị che bởi xe khác đi song song, người ngồi sau xe máy bị che phần thân dưới bởi người lái.
- **Mức `3` (Che khuất nặng):** Bị che khuất $\ge 70\%$ diện tích. Chỉ còn nhìn thấy một phần nhỏ (ví dụ chỉ thấy đầu và vai người, chỉ thấy góc đuôi xe ô tô), nhưng vẫn nhận dạng được nhờ ngữ cảnh xung quanh.

#### 2. Thuộc tính `truncated` (Bị cắt bởi mép ảnh)
- **Giá trị `yes`:** Bounding box của đối tượng chạm sát vào ít nhất một trong 4 cạnh mép ảnh ($x=0, y=0, x=\text{width}$, hoặc $y=\text{height}$) VÀ phần đối tượng ngoài khung hình ước tính chiếm $> 5\text{ px}$ chiều dài/rộng.
- **Giá trị `no`:** Toàn bộ đối tượng nằm trọn vẹn $100\%$ bên trong khung hình (hoặc mép box cách biên ảnh $\ge 2\text{ px}$).

#### 3. Thuộc tính `can_review` (Đánh dấu cần xem xét)
- **Giá trị `true` (tick chọn):** Gán cho bất kỳ box nào mà người gán cảm thấy phân vân, thuộc ca biên chưa rõ ràng trong guideline v0 (ví dụ: xe lai giữa pickup và van, góc chụp quá tối, nghi ngờ tượng hay người thật).
- **Giá trị `false` (mặc định):** Áp dụng cho các trường hợp đã tuân thủ rõ ràng quy tắc.

---

## 3. Quy tắc Vẽ Bounding Box Chuyên sâu

1. **Nguyên tắc bám sát đường biên thực (Tight Bounding Box):**
   - Box phải ôm sát điểm cực trị ngoài cùng nhìn thấy của đối tượng (visible boundary pixels).
   - Không được để viền đệm (padding) khoảng trống nền quá lớn ($> 5\text{ px}$).
   - Không được cắt lẹm vào chi tiết của đối tượng (thiếu gương chiếu hậu, thiếu bánh xe, thiếu cản va).

2. **Nguyên tắc "Chỉ gán phần nhìn thấy" (Visible Parts Only):**
   - Khi đối tượng bị che khuất (occluded), box **chỉ bao bọc phần pixel thực sự nhìn thấy**, tuyệt đối **KHÔNG vẽ ước lượng phần vô hình bị che lấp**.
   - *Lý do kỹ thuật:* Đoán phần vô hình sẽ dẫn đến độ lệch IoU cực lớn giữa các annotator (người đoán box dài, người đoán box ngắn), làm sụp đổ chỉ số đồng thuận IAA.

3. **Tương tác giữa Người và Phương tiện (Person on Vehicle):**
   - **Người lái và hành khách trên xe máy:** Gán **1 box `person`** riêng biệt cho từng người (từ đỉnh mũ bảo hiểm đến chân hoặc phần thân nhìn thấy), và **1 box `motorcycle`** riêng biệt bao trọn toàn bộ xe (bao gồm 2 bánh xe, gương, biển số). Hai box này được phép chồng đè tự nhiên lên nhau.
   - **Người bên trong xe ô tô con / xe tải / xe buýt:** **KHÔNG gán box `person`** cho người ngồi trong khoang kín của ô tô nếu kính xe đóng hoặc bị lóa bóng, vì box xe đã đại diện cho toàn bộ vật thể cơ học đó. Chỉ gán `person` nếu là xe mui trần mở hoàn toàn và thân người nhô hẳn ra ngoài không gian.

4. **Hàng hóa chuyên chở trên phương tiện (Cargo & Load):**
   - **Hàng hóa rời, cồng kềnh chở trên xe máy / xe tải (`img_010.jpg`, `img_012.jpg`, `img_029.jpg`):**
     - Bounding box của phương tiện (`motorcycle` hoặc `truck`) **CHỈ ôm sát thân xe cơ học** (khung sắt, bánh xe, tay lái, thùng xe kim loại).
     - **KHÔNG mở rộng box phương tiện để bao trọn khối hàng** (như đống quả phật thủ, bó xốp, thùng carton cồng kềnh). Hàng hóa không phải là bộ phận cơ học của xe; mở rộng box sẽ làm sai lệch kích thước chuẩn của phương tiện khi xe tự hành ước lượng kích thước vật cản.
     - Nếu khối hàng quá khổng lồ che khuất toàn bộ xe máy đến mức không xác định được hình dạng xe (`img_029.jpg`), tick `can_review = true`.

5. **Xe tải chuyên dụng chở phương tiện khác (`img_002.jpg`):**
   - Gán 1 box `truck` bao toàn bộ xe tải chở hàng mẹ (từ đầu kéo đến đuôi sàn rơ-moóc).
   - Mỗi chiếc xe ô tô con được xếp trên sàn xe tải nếu nhìn thấy rõ ràng hình dạng sẽ được gán **1 box `car` riêng biệt**, thuộc tính đặt `occluded_level = 1` hoặc `2` (do bị rào chắn và khung xe tải che một phần).

6. **Phân loại vật thể giả tạo và hiện tượng quang học (Non-real Objects & Optical Artifacts):**
   - **Tượng đài, tác phẩm điêu khắc (`img_017.jpg`, `img_027.jpg`):** **TUYỆT ĐỐI KHÔNG GÁN BOX.** Tượng đá danh nhân Wellington cưỡi ngựa hay Tượng đài tạc hình xe pickup ở nghĩa trang không phải là thực thể tham gia giao thông.
   - **Ma-nơ-canh trưng bày thời trang (`img_033.jpg`):** **TUYỆT ĐỐI KHÔNG GÁN BOX.** Dù mặc quần áo giống người, ma-nơ-canh là vật thể tĩnh trong cửa hàng.
   - **Tranh ảnh, hình chân dung in trên Billboard quảng cáo (`img_018.jpg`):** **TUYỆT ĐỐI KHÔNG GÁN BOX.** Hình ảnh 2D trên biển hiệu không có nguy cơ chuyển động.
   - **Đồ chơi, mô hình diễu hành giả tưởng (`img_019.jpg`, `img_038.jpg`):** **TUYỆT ĐỐI KHÔNG GÁN BOX.**
   - **Người thật mặc trang phục hóa trang cosplay (`img_011.jpg` - Nhân vật Predator):** **BẮT BUỘC GÁN `person`** vì bên trong bộ trang phục là cơ thể người sinh học thực tế đang đi lại tại sự kiện ngoài trời, có hành vi động học của con người.
   - **Bóng phản chiếu qua gương chiếu hậu (`img_020.jpg`):** **TUYỆT ĐỐI KHÔNG GÁN BOX** cho ảnh ảo phản chiếu trong mặt gương. Chỉ gán các vật thể thật tồn tại trực tiếp trong trường nhìn thế giới thực.

7. **Ngưỡng kích thước tối thiểu và Vùng bỏ qua (`ignore_region`):**
   - Đối tượng có chiều cao $< 20\text{ px}$: Không gán box đơn lẻ.
   - Nếu có cụm người/xe nhỏ hoặc quá đông đúc ở hậu cảnh xa: Khoanh một box rộng lớp `ignore_region`.

---

## 4. Bảng Phân Tích Ca Khó Thực Tế Trong 40 Ảnh

Dưới đây là 8 ca khó điển hình được Kiên trực tiếp phát hiện và khảo sát độc lập từ 40 ảnh gốc trong `data/images/`, chứng minh các điểm thiếu sót chí mạng của `guideline_v0_starter.md`:

| STT | Tên file ảnh | Hiện tượng thực tế quan sát được | Điểm bế tắc của guideline v0 | Quyết định đề xuất của Kiên | Lý giải logic kỹ thuật & an toàn |
|:---:|:---|:---|:---|:---|:---|
| 1 | `img_004.jpg` & `img_035.jpg` | Hai chiếc xe tuk-tuk đỗ trên vỉa hè (`img_004`) và xe auto-rickshaw chở đầy học sinh (`img_035`). Phương tiện 3 bánh có động cơ, cabin liền khung. | v0 chỉ định nghĩa `car` (ô tô con) và `motorcycle` (xe máy), không nói xe 3 bánh xếp vào đâu. Hai người gán sẽ người chọn `car`, người chọn `motorcycle` hoặc `truck`. | Gán tạm vào **`car`**, đặt `can_review = true` để đưa ra biểu quyết ở Pha 2. | Có động cơ, chiếm diện tích mặt đường tương đương xe ô tô mini. Tạm gán `car` để mô hình nhận biết vật cản cỡ trung bình, nhưng cần họp nhóm chốt taxonomy. |
| 2 | `img_002.jpg` | Xe tải Renault cỡ lớn chuyên dụng đang chở nhiều xe ô tô con mới trên sàn rơ-moóc 2 tầng. | v0 quy định "mỗi vật thuộc đúng một lớp" nhưng không hướng dẫn ca "vật nằm trên vật". Gán 1 box xe tải hay gán cả các xe con bên trên? | Gán **1 box `truck`** cho xe tải lớn, và gán **từng box `car` riêng lẻ** cho các xe con nhìn thấy trên giàn giáo (đặt `occluded_level = 1` hoặc `2`). | Đảm bảo mô hình học được đặc trưng hình ảnh của xe con ngay cả khi được vận chuyển, đồng thời nhận diện được tổng thể xe vận tải hạng nặng. |
| 3 | `img_011.jpg` | Một người hóa trang trang phục quái vật ngoài hành tinh Predator đứng cạnh hàng xe mô tô độ. | v0 chỉ nói `person` là "người". Người mặc đồ cosplay dị thường có tính là người không? | Bắt buộc **gán `person`**, thuộc tính `occluded_level = 0`. | Bên trong trang phục là con người thật có hành vi đi lại tự do. Mô hình phát hiện người đi bộ phải phát hiện được cả người mặc trang phục đặc thù để bảo vệ an toàn tính mạng. |
| 4 | `img_027.jpg` | Tượng đài tạc bằng đá hoa cương nguyên khối mô phỏng hình dáng chiếc xe tải pickup tại nghĩa trang Oak Hill. | v0 định nghĩa `truck` là xe tải. Bức tượng bằng đá có hình dáng xe tải thì có gán không? | **TUYỆT ĐỐI KHÔNG GÁN BOX.** | Đây là khối đá tĩnh, không phải phương tiện giao thông. Gán box sẽ làm mô hình học sai tính chất vật thể (False Positive độc hại). |
| 5 | `img_029.jpg` | Xe máy chở một khối hàng hóa cồng kềnh khổng lồ che khuất gần như toàn bộ người lái và thân xe. | v0 yêu cầu "box ôm sát vật", nhưng hàng hóa cồng kềnh có được vẽ gộp vào xe máy không? | Chỉ vẽ box **`motorcycle`** ôm sát phần bánh xe và ghi-đông kim loại nhìn thấy. Gán box **`person`** cho phần mũ/đầu người lái (nếu thấy). Tick `can_review = true`. | Không bao gồm khối hàng vào box xe máy để tránh làm sai lệch kích thước chuẩn của phương tiện giao thông 2 bánh trong mắt mô hình. |
| 6 | `img_033.jpg` | Ma-nơ-canh mặc áo khoác thời trang đứng trưng bày trên phố mua sắm Buchanan. | v0 chỉ nói `person` là "người", người gán dễ nhầm ma-nơ-canh ở cự ly xa là người thật đang đứng. | **TUYỆT ĐỐI KHÔNG GÁN BOX.** | Ma-nơ-canh là vật trưng bày bất động, không được dạy cho mô hình xe tự lái coi là người đi bộ nhằm tránh hiện tượng xe phanh đột ngột. |
| 7 | `img_037.jpg` | Xe bán tải pickup có gắn thêm nắp đậy thùng hàng cao ngang nóc cabin (Master 75 Truck Cap). | Hình dáng lai giữa xe bán tải (`truck`), xe con SUV (`car`) và xe thùng kín (`van`). v0 không phân biệt được. | Gán tạm là **`car`** (hoặc `truck` tùy công năng), tick `can_review = true` chờ phiên họp Pha 2 chốt quy chuẩn phân loại xe bán tải. | Tránh tranh cãi cá nhân. Bản chất đây là khung gầm xe bán tải nhưng được hoán cải thành dạng thùng kín như van. |
| 8 | `img_018.jpg` | Biển quảng cáo chính trị gia kích thước lớn dựng bên đường, có ảnh chụp chân dung người đàn ông rất rõ nét. | v0 không quy định về hình in/ảnh chụp chân dung 2D. | **TUYỆT ĐỐI KHÔNG GÁN BOX.** | Hình ảnh 2D trên biển hiệu phẳng. Không bao giờ được nhận diện là người đi bộ tham gia giao thông. |

---

## 5. Quy trình Kiểm Soát Chất Lượng (QC) và Lịch Sử Phiên Bản

### 5.1. Ngưỡng kiểm soát chất lượng áp dụng
Bản guideline cá nhân này được thiết kế để chuẩn bị đáp ứng các ngưỡng khắt khe trong `submission/taxonomy.json` và `docs/QC_CHEATSHEET.md`:
- **Ngưỡng kích thước box tối thiểu:** Chiều cao $\ge 20\text{ px}$. Box nhỏ hơn bắt buộc gom vào `ignore_region`.
- **Ngưỡng sai lệch IoU:**
  - Box có chiều cao $\ge 50\text{ px}$: Yêu cầu $\text{IoU} \ge 0.85$.
  - Box có chiều cao $20\text{ px} \le h < 50\text{ px}$: Yêu cầu $\text{IoU} \ge 0.70$.
- **Ngưỡng đo độ đồng thuận liên gán nhãn (IAA):**
  - $\text{IoU trung bình} \ge 0.85$ trên các cặp box ghép thành công.
  - $\text{Tỷ lệ đồng thuận lớp} \ge 95\%$.
- **Ngưỡng nghiệm thu phát hành (Release):** Tỷ lệ ảnh có lỗi sau kiểm định $\le 2\%$.

### 5.2. Nguyên tắc xử lý các ca còn nghi vấn
Mọi trường hợp Kiên chưa hoàn toàn chắc chắn trong quá trình gán nhãn cá nhân ở Pha 3 sẽ được xử lý nghiêm ngặt theo quy trình:
1. Gán theo phán đoán logic hợp lý nhất của mình.
2. Đánh dấu `can_review = true`.
3. Ghi chép mã ảnh và lý do vào sổ tay cá nhân để mang ra phản biện tại phiên họp Hiệu chuẩn (Calibration - Pha 5).
4. Tuyệt đối không tự ý hỏi ý kiến của các thành viên khác trong khi đang gán nhãn độc lập.

### 5.3. Bảng Lịch sử Phiên bản Guideline

| Phiên bản | Ngày | Người thực hiện | Tóm tắt thay đổi chính | Lý do / Ca thực tế kích hoạt |
|:---:|:---:|:---:|:---|:---|
| `v0` | 2026-11-16 | Giảng viên | Bản khởi đầu sơ sài gồm 6 lớp và 3 thuộc tính cơ bản. | Bản đề bài cung cấp (cố ý thiếu ca khó). |
| `v0.1-draft-DoHuyKien` | 2026-11-16 | ĐỖ HUY KIÊN | 1. Bổ sung mục tiêu ứng dụng và phân tích an toàn thị giác máy tính.<br/>2. Định nghĩa chi tiết 6 lớp kèm ví dụ và phản ví dụ.<br/>3. Định lượng ngưỡng phần trăm diện tích cho `occluded_level` ($0\%, 10-35\%, 35-70\%, \ge 70\%$).<br/>4. Định lượng ngưỡng mép ảnh cho `truncated` ($> 5\text{ px}$).<br/>5. Thiết lập quy tắc xử lý ma-nơ-canh, tượng đá, hình in quảng cáo, người cosplay, xe tuk-tuk, xe chở xe.<br/>6. Phân tích 8 ca khó cụ thể có mã ảnh từ 40 ảnh thực tế. | Khảo sát thực tế 40 ảnh trong `data/images/` nhằm chuẩn bị cho phiên họp thống nhất v1.0 của Nhóm T005. |
