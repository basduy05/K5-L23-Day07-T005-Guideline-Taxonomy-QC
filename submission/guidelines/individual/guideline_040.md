# Guideline gán nhãn — v0.1 (cá nhân) — 2A202603040 - Nguyen Duc Cuong | Nhóm T005

- Phiên bản: v0.1 (bản cá nhân)
- Ngày: 2026-11-16
- Người viết: Nguyen Duc Cuong (2A202603040)

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện thật đang tham gia hoặc có thể tham gia giao thông** trên ảnh đường phố (camera giao thông, xe tự lái).

Bộ quy tắc được rút ra từ 15 ảnh mẫu (img_002 → img_040): đường phố Việt Nam và Thái Lan đông xe máy, bãi xe ở Séc, xe chở ô tô, bảo tàng xe, nghĩa trang, cửa hàng đồ chơi, tượng đài. Vì vậy bộ quy tắc chú trọng hai việc:

- Gán đủ và tách đúng **người – xe máy** trong giao thông dày đặc.
- **Không** gán những thứ trông giống người/xe nhưng không phải vật thật: ảnh quảng cáo, tượng, đồ chơi, mô hình, hình phản chiếu.

## 2. Lớp và thuộc tính

Mỗi vật thuộc đúng một lớp. So với v0, bản này thêm 3 lớp `bus`, `bicycle`, `three_wheeler` vì chúng xuất hiện trong ảnh mẫu (img_009, img_006, img_040) và không xếp vào lớp cũ mà không làm bẩn dữ liệu.

### Lớp

| Lớp | Định nghĩa | Gồm | Không gồm |
|---|---|---|---|
| `car` | Ô tô chở người cỡ nhỏ, ≤ 9 chỗ. | Sedan, hatchback, wagon, SUV, crossover, bán tải (pickup), taxi ô tô. | MPV/van thùng kín (→ `van`), ô tô đồ chơi, mô hình. |
| `truck` | Xe chở hàng cỡ trung/nặng, cabin tách rời thùng hàng. | Xe tải thùng, xe chở ô tô, đầu kéo + rơ-moóc (một box chung). | Bán tải (→ `car`). |
| `van` | Xe thùng kín cỡ nhỏ, thân liền khối, chở hàng hoặc ≤ 16 chỗ trên khung xe con. | Xe van giao hàng, xe 16 chỗ kiểu Hiace/Transit, xe cứu thương dạng van. | Xe khách có thân bus riêng (→ `bus`). |
| `bus` | Xe khách có thân bus riêng, cửa lên xuống kiểu bus. | Xe buýt, xe khách 29–45 chỗ, minibus kiểu County, xe song thảo (songthaew) có mái. | Van 16 chỗ (→ `van`). |
| `motorcycle` | Xe hai bánh có động cơ. | Xe máy số, tay ga, mô tô, xe máy điện không bàn đạp; cả xe đang chạy và đang đỗ. | Xe đạp điện có bàn đạp (→ `bicycle`), mô hình nghệ thuật. |
| `bicycle` | Xe đạp. | Xe đạp thường, xe đạp điện có bàn đạp. | — |
| `three_wheeler` | Xe ba bánh có động cơ. | Tuk-tuk, xe lôi, xe ba gác máy. | Xe máy gắn thùng phụ (sidecar vẫn là `motorcycle`). |
| `person` | Người thật. | Người đi bộ, người lái/ngồi sau xe máy, xe đạp, xe ba bánh hở; người đứng, ngồi, bán hàng. | Người trong ảnh quảng cáo, tượng, hình nhân vật, người ngồi trong xe kín. |
| `ignore_region` | Vùng có vật thuộc các lớp trên nhưng không thể gán từng vật. | Đám đông xa, dãy xe máy đỗ chồng lên nhau, vật quá nhỏ. | Vùng không có vật nào (không cần vẽ). |

### Thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Phần vật bị vật khác che (trong khung ảnh): `0` = < 10% (thấy rõ); `1` = 10–40% (che ít); `2` = 40–80% (che nhiều); `3` = > 80% hoặc không ước lượng được phần bị che. |
| `truncated` | `no`, `yes` | `yes` khi một phần vật nằm ngoài mép ảnh. Phần bị cắt **không** tính vào `occluded_level`. |
| `can_review` | tick / không | Tick khi không chắc về lớp, ranh giới box hoặc có nên gán hay không. Không dùng thay cho việc đọc mục 4. |

`ignore_region` không cần thuộc tính.

## 3. Quy tắc vẽ box

1. **Gán đủ.** Vẽ box cho mọi người và phương tiện thật có chiều cao ≥ 12 px và còn nhận ra được lớp. Vật nhỏ hơn hoặc không tách được → đưa vào `ignore_region`.
2. **Ôm sát phần nhìn thấy.** Box chạm mép ngoài cùng của phần vật thấy được, sai số ≤ 2 px. Không đoán phần bị che hoặc nằm ngoài ảnh.
3. **Vật che mảnh không cắt box.** Hàng rào song sắt, cột điện, dây điện, giọt mưa trên kính: box vẫn phủ cả vật phía sau, chỉ tăng `occluded_level` (thường là `1`).
4. **Người và xe là hai box riêng.** Người lái và mỗi người ngồi sau đều có box `person` riêng; xe có box `motorcycle`/`bicycle` riêng. Hai box được phép chồng nhau.
5. **Box `person`** gồm đầu đến chân, kể cả mũ bảo hiểm, nón lá, quần áo. **Không** gồm đồ cầm/đeo tách khỏi thân: túi xách, gánh hàng, đòn gánh, ô, chân máy ảnh.
6. **Box phương tiện** gồm gương, thùng hàng gắn cố định (thùng giao đồ ăn, giá nóc, rơ-moóc nối liền). **Không** gồm hàng hoá chất rời hoặc phương tiện khác đặt trên xe.
7. **Nhóm đông.** Khi ≥ 3 vật cùng lớp chồng nhau mà không xác định được ranh giới từng vật, vẽ một `ignore_region` bao cả nhóm. Vật nào ở rìa nhóm vẫn tách được thì vẫn gán riêng.
8. **Chọn đúng lớp và thuộc tính** theo mục 2. Không chắc → chọn lớp gần nhất và tick `can_review`.
9. **Chỉ vật thật.** Không gán người/xe trong ảnh in, biển quảng cáo, màn hình, hình phản chiếu (gương, kính, sơn xe), tượng, đồ chơi, mô hình. Chi tiết xem mục 4.
10. **Xe của chính người chụp** (khung cửa, gương, nắp capo lọt vào ảnh) không gán.

## 4. Ca khó

Mỗi ca dưới đây lấy từ ít nhất một ảnh mẫu; quyết định trong cột "Cách gán" là bắt buộc.

### 4.1. Thứ trông giống người/xe nhưng không phải vật thật

| # | Tình huống | Ảnh | Cách gán |
|---|---|---|---|
| 1 | Người trên biển quảng cáo, poster tranh cử | img_018 | Không gán. |
| 2 | Ảnh/sticker có mặt người dán trên kính xe | img_040 | Không gán. Xe vẫn gán bình thường. |
| 3 | Xe phản chiếu trong gương chiếu hậu | img_020 | Không gán xe trong gương; không gán thân xe người chụp. |
| 4 | Mô hình ô tô cỡ nhỏ chở trên nóc xe thật | img_024 | Không gán mô hình (không phải xe thật, kích thước sai tỉ lệ). Xe bên dưới gán `car`. |
| 5 | Bia mộ tạc hình xe bán tải | img_027 | Không gán. |
| 6 | Ô tô đồ chơi, tượng nhân vật trong tủ kính | img_038 | Không gán. |
| 7 | Tượng người cưỡi ngựa | img_017 | Không gán `person`. |
| 8 | Tượng sinh vật hình người, xe nghệ thuật làm từ sắt vụn trưng bày | img_011 | Không gán tượng. Xe sắt vụn: không gán nếu rõ là vật trưng bày không chạy được; nếu nghi là xe độ chạy thật → gán `motorcycle` + tick `can_review`. |

### 4.2. Phương tiện thật trong bối cảnh lạ

| # | Tình huống | Ảnh | Cách gán |
|---|---|---|---|
| 9 | Xe cổ đỗ trong bảo tàng, trong nhà | img_024 | Gán bình thường (`car`). Bối cảnh không đổi lớp. Xe bị cắt ở mép → `truncated: yes`. |
| 10 | Xe chở ô tô có xe con trên hai tầng | img_002 | Xe chở: một box `truck` bao cabin + khung + rơ-moóc. Mỗi xe con trên đó: box `car` riêng; xe tầng dưới bị khung che → thường `occluded_level: 2`. |
| 11 | Bãi xe sau hàng rào song sắt | img_018 | Gán từng xe; hàng rào không cắt box, `occluded_level: 1`. Xe sau bị xe trước che thêm thì tính tổng. |
| 12 | Ảnh chụp qua kính xe có lóa/mưa | img_018, img_020 | Vẫn gán vật phía sau kính; lóa/giọt nước không tính là che trừ khi mất hẳn chi tiết. |

### 4.3. Người – xe máy – xe đạp trong giao thông dày

| # | Tình huống | Ảnh | Cách gán |
|---|---|---|---|
| 13 | Hai, ba người trên một xe máy | img_005 | Mỗi người một box `person`, xe một box `motorcycle`. Người ngồi sau bị người lái che → tăng `occluded_level`. |
| 14 | Người đi xe đạp | img_006, img_003 | `person` + `bicycle`. Xe đạp dựng không người vẫn gán `bicycle`. |
| 15 | Xe máy chỉ thấy gương/tay lái, người lái chiếm gần hết khung | img_040, img_005 | Gán `motorcycle` nếu thấy đủ để biết là xe máy (gương + tay lái); `truncated: yes`, `occluded_level` theo phần bị người che. Chỉ thấy một chiếc gương → không gán. |
| 16 | Dãy xe máy đỗ sát nhau trên vỉa hè | img_003, img_008 | Xe gần, tách được: gán từng chiếc. Phần xa, chồng lên nhau: một `ignore_region`. |
| 17 | Đám đông/dòng xe ở xa | img_006, img_011 | Người/xe cao < 12 px hoặc không tách được → `ignore_region`. |
| 18 | Người bán hàng gánh đòn gánh | img_003 | Box `person` chỉ bao người, không bao gánh hàng và đòn gánh. |
| 19 | Người đi bộ giữa dòng xe máy bị nhoè chuyển động | img_014 | Gán người nét. Xe/người nhoè: box phần thân chính (không bao vệt nhoè kéo dài); nhoè đến mức không xác định được ranh giới → `ignore_region`. |
| 20 | Xe ôm mặc áo váy, xe có thùng giao hàng | img_008 | `person` + `motorcycle`; thùng gắn cố định thuộc box xe. |
| 21 | Người ngồi trong xe kín (ô tô, cabin tải, van, bus) | img_005, img_009 | Không gán `person`. Người trên xe hở (xe máy, xe đạp, tuk-tuk, sau bán tải) thì gán. |
| 22 | Người ở ban công, trong cửa hàng nhìn thấy từ đường | img_009 | Gán `person` nếu ≥ 12 px và nhìn rõ là người. |

### 4.4. Phân biệt lớp phương tiện

| # | Tình huống | Ảnh | Cách gán |
|---|---|---|---|
| 23 | Minibus thân bus (kiểu County) | img_009 | `bus`. |
| 24 | Xe van/16 chỗ thân liền khung xe con | img_005, img_011 | `van`. |
| 25 | Tuk-tuk | img_040 | `three_wheeler`. |
| 26 | Xe song thảo, xe bán tải chở khách có mái | img_008 | `bus` nếu phần sau là khoang khách có mái; không chắc → tick `can_review`. |
| 27 | SUV, bán tải, wagon | img_018, img_024, img_040 | `car`. |
| 28 | Taxi ô tô | img_040, img_006 | `car` (không có lớp taxi riêng). |

Không gán: động vật (chó, chim), chân máy ảnh, biển báo, cột cảnh báo giao thông (kể cả cột trên đầu tượng ở img_017).

## 5. QC và phiên bản

- Mọi thay đổi làm đổi nhãn (thêm/bỏ lớp, đổi ngưỡng 12 px, đổi cách gán một ca khó) đều tăng phiên bản guideline. Sửa câu chữ không đổi nhãn thì không tăng.
- Mỗi ảnh ghi phiên bản guideline đã dùng khi gán. Khi lên phiên bản, lọc lại các ảnh chứa ca bị đổi.
- Ca mơ hồ: tick `can_review`. Người duyệt quyết định; ca lặp lại ≥ 3 lần → thêm vào mục 4 ở phiên bản sau.
- Kiểm tra chéo: người duyệt xem lại 10% ảnh mỗi lô và 100% ảnh có `can_review`. Lỗi hay gặp cần soát: gộp người với xe máy thành một box, gán người trên poster/tượng, bỏ sót người ngồi sau.

| Phiên bản | Ngày | Thay đổi |
|---|---|---|
| v0.1 | 2026-11-16 | Bản cá nhân, rút từ 24 ảnh mẫu. Thêm lớp `bus`, `bicycle`, `three_wheeler`; định lượng `occluded_level`; ngưỡng 12 px; 28 ca khó. |
