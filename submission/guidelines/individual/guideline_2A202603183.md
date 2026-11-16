# Guideline gán nhãn cá nhân — Hoàng Nhật Anh — 2A202603183

- Nhóm: **T005**.
- Phiên bản: **v0.1-personal** — đề xuất cá nhân ở Pha 1.
- Ngày lập: **16/11/2026**.
- Người đứng tên tài liệu: **Hoàng Nhật Anh — 2A202603183**.
- Trạng thái: dự thảo cá nhân, chưa phải guideline thống nhất của nhóm.
- Phạm vi rà soát: **40/40 ảnh gốc** `img_001.jpg`–`img_040.jpg`, ảnh tổng hợp `docs/assets/sample_grid.jpg`, hướng dẫn và các ngưỡng kiểm tra trong repo.
- Công cụ dự kiến: CVAT; Rectangle; xuất COCO 1.0.

Tài liệu tham chiếu: [GUIDE.md](../../../GUIDE.md), [RULES.md](../../../RULES.md), [RUBRIC.md](../../../RUBRIC.md), [khung guideline](../../../guideline/TEMPLATE_guideline.md), [v0](../../../guideline/guideline_v0_starter.md), [QC cheatsheet](../../../docs/QC_CHEATSHEET.md), [taxonomy hiện tại](../../taxonomy.json).

Ghi chú về quá trình làm bài, phân công và sử dụng AI: [ghi chú cá nhân](../../individual_notes/2A202603183_HoangNhatAnh.md).

## 1. Mục tiêu

Xây dựng quy tắc phát hiện **người thật và phương tiện thật** trong ảnh giao thông, để các thành viên chọn cùng lớp, vẽ box theo cùng một quy ước và xử lý giống nhau ở các ca khó.

Ưu tiên tránh bỏ sót người, nhầm người với tượng/hình in, nhầm loại xe và gộp người với xe. Nhãn phục vụ bài thực hành huấn luyện mô hình phát hiện đối tượng; không dùng trạng thái đang chạy hay đang đỗ để quyết định một vật có được gán hay không.

Guideline bao quát toàn bộ 40 ảnh. Khi gán chính thức phải dùng v1.0 đã được nhóm thống nhất, kể cả khi khác đề xuất trong bản cá nhân này.

## 2. Lớp và thuộc tính

### 2.1. Taxonomy đề xuất

Mỗi đối tượng vật lý thuộc đúng một lớp. `ignore_region` là vùng loại khỏi đánh giá từng vật, không phải một loại người hay phương tiện.

Giữ sáu tên lớp của v0, làm rõ ranh giới và **đề xuất thêm bốn lớp** `bus`, `bicycle`, `cyclo`, `tuk_tuk`. Các lớp mới chỉ được dùng trong CVAT sau khi Pha 2 thống nhất, cập nhật taxonomy và sinh lại nhãn. Bản này không tự thay đổi taxonomy của nhóm.

| Lớp | Định nghĩa và dấu hiệu phân biệt | Ví dụ trong bộ ảnh | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Ô tô con có thân xe chở người: sedan, hatchback, wagon, SUV, MPV cá nhân có cấu trúc ô tô con. | Ô tô đen `img_023`; wagon đen `img_024`; taxi `img_040`. | Pickup có thùng hàng thuộc `truck`; minibus thân cao không tự coi là `car`; ô tô đồ chơi `img_038` bị loại. |
| `truck` | Xe có cabin và thùng/sàn chở hàng tách biệt về cấu trúc; gồm tải nhỏ, tải thùng kín, pickup, xe chuyên chở ô tô. | Xe tải `img_001`, `img_007`, `img_036`; pickup lắp nắp thùng `img_037`; xe chở xe `img_002`. | Van có khoang hàng liền thân thuộc `van`; tượng pickup `img_027` bị loại; không phân lớp theo kích thước hoặc hàng hóa đang chở. |
| `van` | Xe thân hộp liền khối, khoang hàng hoặc hành khách liền cabin, thường mái cao và có cửa trượt/hậu; không có thùng hàng độc lập. | Xe thân hộp phía sau người đi xe máy `img_005`; xe trắng phía sau `img_016`. | Xe tải nhỏ cabin rời `img_036` là `truck`; SUV/MPV thân ô tô con là `car`; xe khách cấu trúc bus là `bus`. |
| `motorcycle` | Xe máy/mô tô hai bánh có động cơ; gồm scooter và một tổ hợp mô tô gắn sidecar cố định. | Scooter `img_010`, `img_026`; mô tô có sidecar `img_022`. | Xe đạp `img_029`; xích lô `img_023`; tuk-tuk ba bánh chuyên dụng `img_004`; tác phẩm kim loại chỉ mô phỏng xe. |
| `person` | Một con người thật nhìn thấy trực tiếp, ở bất kỳ tư thế nào, gồm người đi bộ, người lái và hành khách nhìn thấy được. | Người bán hàng `img_003`; từng người trên xe `img_026`; người đội mũ kín mặt `img_040`. | Tượng `img_017`, người in trên billboard `img_018`, ma-nơ-canh/hình in `img_033`, nhân vật đồ chơi; phản chiếu không tạo thêm người. |
| `bus` **(mới)** | Xe buýt/xe khách có thân dài, dãy cửa sổ và khoang chở khách theo cấu trúc bus, gồm minibus nếu có cấu trúc này. | Xe khách giữa `img_009`; bus `img_031`, `img_037`, `img_040`. | Không dùng `van` hoặc `truck` chỉ vì cùng thân hộp; không suy đoán số ghế hoặc tải trọng từ một ảnh. |
| `bicycle` **(mới)** | Xe đạp hai bánh với cấu trúc bàn đạp/truyền động và khung xe đạp quan sát được. | Xe đạp `img_006`, `img_029`, `img_039`. | Xe máy có động cơ là `motorcycle`; xích lô ba bánh là `cyclo`; không đoán xe điện hay xe đạp khi phần nhận biết bị che. |
| `cyclo` **(mới)** | Xích lô ba bánh dùng bàn đạp, có ghế/khoang hành khách và vị trí người đạp riêng. | Xích lô ở trung tâm `img_023`. | Xe đạp hai bánh là `bicycle`; xe ba bánh có động cơ kiểu auto-rickshaw là `tuk_tuk`. |
| `tuk_tuk` **(mới)** | Xe ba bánh có động cơ kiểu tuk-tuk/auto-rickshaw, một cụm cabin và khoang khách hoặc hàng trên khung xe. | `img_004`, `img_016`, `img_035`, `img_040`. | Không gán `motorcycle` chỉ vì có ba bánh; mô tô gắn sidecar `img_022` vẫn là một `motorcycle`; pickup là `truck`. |
| `ignore_region` | Hình chữ nhật giới hạn vùng có đối tượng quá nhỏ hoặc không thể tách một cách đáng tin cậy để gán từng vật. | Cụm giao thông rất xa trong `img_021`, nếu đạt điều kiện mục 3.8. | Không phủ lên tượng, đồ chơi, tranh ảnh đã biết là vật loại trừ; không bao toàn ảnh đông người chỉ để giảm số box. |

**Ranh giới cần áp dụng nhất quán:** tải thùng kín vẫn là `truck` nếu thùng tách cabin; pickup lắp nắp thùng vẫn là `truck`; wagon vẫn là `car`. Xe thật được vận chuyển/trưng bày vẫn thuộc lớp xe; mô hình nhỏ, vỏ trang trí và tượng không tự trở thành xe thật.

### 2.2. Vật ngoài taxonomy và vật chưa rõ

1. Vật xác định không thuộc phạm vi như cây, biển báo, động vật, hàng hóa, xe đẩy bán hàng: không gán lớp đối tượng và không tạo ignore chỉ vì chúng xuất hiện.
2. Tượng, đồ chơi, hình in và phản chiếu xác định được: loại khỏi nhãn đối tượng, giữ làm nền âm tính.
3. Vật thuộc một trong bốn lớp đề xuất: ghi tên lớp đề xuất trong bản cá nhân; Pha 2 phải chốt thêm lớp hoặc loại khỏi phạm vi trước Pha 3. Không ép vào lớp cũ cho qua kiểm tra.
4. Có thể xác định lớp nhưng chưa chắc ranh giới hoặc thuộc tính: tạo box theo phần nhìn thấy, chọn lớp có căn cứ, tick `can_review` và ghi lý do.
5. Không xác định được cả bản chất/lớp: không tạo lớp đoán mò. Ghi mã ảnh và vị trí; đề xuất ignore cục bộ nếu đó là vùng mục tiêu không thể giải nghĩa, đưa ra họp. Nếu thiếu quy tắc ảnh hưởng tới nhãn chính thức, cần làm rõ trước khi chốt v1.0.

### 2.3. Thuộc tính và ngưỡng

Giữ nguyên tên và miền giá trị của v0 để thuận tiện cho CVAT và script. Các ngưỡng dưới đây là đề xuất cá nhân cần được nhóm chốt.

**`occluded_level`**: mức che bởi vật khác, không tính phần nằm ngoài khung ảnh, bóng tối, nhòe chuyển động hay việc người quay lưng.

Gọi `r` là tỉ lệ diện tích hình dạng đối tượng bị vật khác che trên **phần hình dạng lẽ ra nằm trong ảnh**. Ước lượng hình dạng của chính vật, không lấy phần nền trống trong hình chữ nhật làm diện tích bị che. Đây là ước lượng thị giác theo ngưỡng cố định, không phải phép đo mask chính xác. Không dùng phần ước lượng này để nới box ra ngoài phần nhìn thấy.

| Giá trị | Điều kiện | Cách xử lý |
|---|---|---|
| `0` | `r = 0%`: không có phần bị vật khác che nhận biết được. | Gán `0`, kể cả vật bị cắt mép nhưng không bị vật khác che. |
| `1` | `0% < r ≤ 30%`. | Che ít; đúng 30% thuộc mức `1`. |
| `2` | `30% < r ≤ 70%`. | Che vừa/nhiều; đúng 70% thuộc mức `2`. |
| `3` | `70% < r < 100%`, hoặc nhận ra vật nhưng không thể ước lượng `r` đáng tin cậy. | Che rất nhiều/không định lượng được; tick `can_review` nếu không ước lượng được hoặc chưa chắc lớp. |

- Ví dụ kiểm ranh giới: 0% → `0`; 20%/30% → `1`; 31%/60%/70% → `2`; 71% → `3`.
- Vật hoàn toàn bị che (`r = 100%`): không dựng box từ suy đoán.
- Nếu khoảng ước lượng vượt qua một ngưỡng, chẳng hạn 25–35%, chọn mức ứng với ước lượng trung tâm (30% → `1`), tick `can_review`, ghi khoảng để calibration.
- Người nhìn được qua kính sạch không tự bị tính che; khung cửa hoặc phần kính mờ đục che thân mới được tính. Mũ, quần áo, khẩu trang đang mặc không là vật che đối với nhãn người.

| Thuộc tính | Giá trị cố định | Quy tắc |
|---|---|---|
| `truncated` | `no`, `yes` | `yes` nếu ảnh cắt mất một phần của chính đối tượng tại bất kỳ mép nào. `no` nếu toàn vật nằm trong ảnh, kể cả bị vật khác che. Box chạm mép là dấu hiệu cần kiểm, không đủ để tự kết luận bị cắt. |
| `can_review` | `false`, `true` (checkbox) | `true` khi chưa chắc lớp, thật/giả, biên box, mức che, nguyên nhân mất phần thân, hoặc có quy tắc chưa chốt. Không dùng cờ này để thay cho thuộc tính che/cắt mép. |

Mỗi box đối tượng phải có đủ ba thuộc tính. `ignore_region` không cần các thuộc tính này. Trong CVAT, cờ `occluded` có sẵn/phím `Q` không thay thế `occluded_level`; script của lab chỉ đọc thuộc tính do nhóm khai báo.

## 3. Quy tắc vẽ box

### 3.1. Box theo phần nhìn thấy

- Vẽ một hình chữ nhật song song cạnh ảnh, nhỏ nhất bao tất cả phần **nhìn thấy trực tiếp** của một đối tượng. Không vẽ box xoay.
- Nếu vật có nhiều phần lộ ra do bị che, dùng một box bao các phần lộ ra của cùng vật. Khoảng trống/vật che nằm giữa các phần đó có thể nằm trong box.
- Không mở rộng để khôi phục phần khuất hoặc phần đã ra ngoài ảnh; không đoán vị trí bánh xe/chân không nhìn thấy.
- Mép box bám biên vật tại độ phân giải ảnh gốc; không thêm viền đệm. Phóng to để đặt biên, nhưng đo pixel theo ảnh gốc, không theo kích thước hiển thị.
- Tọa độ luôn trong ảnh: `0 ≤ x_min < x_max ≤ W`, `0 ≤ y_min < y_max ≤ H`. COCO lưu `[x_min, y_min, width, height]`.

### 3.2. Phần nào thuộc box người hoặc xe

- `person`: bao thân thể nhìn thấy, tóc, quần áo, mũ đang đội và giày. Loại phần nhô ra chỉ thuộc túi xách, ô, gánh hàng, vật cầm tay. Nếu các vật này nằm giữa các bộ phận của người, box vẫn có thể chứa chúng do hình chữ nhật.
- Xe: bao thân, bánh, tay lái, gương và bộ phận cố định nhìn thấy. Không lấy bóng đổ, vệt phản sáng, vệt nhòe kéo dài làm biên xe.
- Thùng xe/nắp thùng cố định là phần xe; kiện hàng, nông sản, thanh kim loại chở trên xe là hàng rời, không dùng làm điểm cực trị của box xe.
- Xe độ có chi tiết gắn cố định: chỉ bao sau khi xác nhận đó là xe thật và chi tiết thuộc xe; tác phẩm mô phỏng xe được xử lý như vật loại trừ.

### 3.3. Người trên xe, xe gắn sidecar và xích lô

- Người và xe là các đối tượng riêng; box chồng nhau được phép.
- Một xe máy chở hai người nhìn thấy và đủ cao: **một box xe + hai box người**. Không tạo lớp `rider` trong đề xuất này.
- Người trong cabin/ô tô chỉ được gán khi thật sự nhìn thấy, xác định được là người và box phần nhìn thấy cao từ 20 px; không suy ra có người từ xe đang chạy.
- Mô tô có sidecar gắn cố định: một box `motorcycle` bao cả cụm; không thêm box sidecar. Người trên đó, nếu có và đủ điều kiện, là box riêng.
- Một xích lô: một box `cyclo` cho khung xe và khoang khách; người đạp, khách ngồi là các box `person` riêng. Không gộp xích lô vào box người đạp.

### 3.4. Xe chở xe, xe trưng bày và vật trang trí

- `img_002`: tổ hợp xe chuyên chở gồm đầu kéo và phần sàn/rơ-moóc gắn cùng tổ hợp được đề xuất là **một box `truck`**; mỗi xe thật được chở là một box riêng theo lớp của nó. Không gộp cả dàn xe thành một đối tượng.
- Box xe chuyên chở bám các điểm cực trị của khung xe chuyên chở, không nâng mép trên chỉ vì có ô tô đặt trên sàn; xe tải có thể có box chứa xe con do cấu trúc nhiều tầng.
- Xe thật đỗ trong bảo tàng, bãi xe hoặc làm quảng cáo vẫn được gán. Không cần chứng minh động cơ đang hoạt động bằng một ảnh.
- Tượng/mô hình/vỏ xe không có bản chất phương tiện: không gán thành xe. Kích thước và bối cảnh là bằng chứng, không phải chỉ nhìn tên tệp.
- `img_019`: đề xuất coi xe diễu hành có kích thước người sử dụng, bánh xe và người ngồi là `car`; hình dáng hư cấu không tự làm nó thành đồ chơi. Giữ `can_review` vì cần nhóm chốt phạm vi xe diễu hành.
- `img_024`: chiếc wagon đen là xe thật; vật màu vàng đặt trên nóc có dấu hiệu là mô hình/vỏ trưng bày thu nhỏ, đề xuất loại vật vàng và đưa ra họp xác nhận, không mặc nhiên coi là một ô tô thật được chở.

### 3.5. Che khuất và cắt mép

- Box theo phần lộ ra, thuộc tính che theo mục 2.3; hai việc này không mâu thuẫn.
- Hàng rào, cột, người và xe khác có thể gây che. Gán từng vật nhận biết được thay vì gộp chúng.
- Cắt mép: kết thúc box đúng mép ảnh, `truncated=yes`; không kéo box ra ngoài ảnh hoặc hoàn thiện phần mất.
- Vật có thể vừa `truncated=yes` vừa `occluded_level>0` nếu có cả hai nguyên nhân.
- Cận cảnh chỉ thấy một phần xe/người vẫn được gán nếu đủ căn cứ nhận dạng và đủ chiều cao nhìn thấy. Không tự loại chỉ vì không thấy toàn thân.

### 3.6. Gương, kính, hình in, tượng và đồ chơi

- Không tạo box cho người/xe **chỉ xuất hiện qua phản chiếu** trong gương, cửa kính, thân xe bóng hoặc mặt đường ướt.
- Nhìn xuyên kính thấy người/xe thật ở phía sau vẫn được gán nếu đủ điều kiện. Phân biệt phản chiếu với nhìn xuyên; chưa rõ thì ghi ca cần review.
- Không gán ảnh chân dung, ảnh phương tiện trên quảng cáo/màn hình, tượng người, ma-nơ-canh, búp bê và xe đồ chơi.
- Người thật mặc trang phục hóa trang vẫn là `person`. Không kết luận một nhân vật đứng yên là tượng chỉ vì trang phục lạ.
- Vật xác định là đồ giả/hình in phải giữ làm nền âm tính, không phủ `ignore_region` để tránh việc kiểm sai nhãn thừa.

### 3.7. Xe chủ (ego vehicle)

Loại phần của chính xe gắn camera khi có căn cứ rõ (gương và khung cửa của xe chụp trong `img_020`). Xe khác nhìn trực tiếp vẫn gán bình thường. Không coi mọi xe lớn ở tiền cảnh là xe chủ.

`img_012` có thùng pickup sát máy ảnh nhưng ảnh tĩnh chưa đủ chắc máy ảnh gắn trên xe đó; đề xuất gán phần pickup nhìn thấy là `truck`, đánh dấu cần review nếu xác định ego ảnh hưởng quyết định. Điểm này phải được nhóm chốt trước khi gán chính thức.

### 3.8. Vật nhỏ, cụm đông và ignore

1. Đo `h = y_max - y_min` của box bao **phần nhìn thấy** trên ảnh gốc.
2. `h ≥ 20 px`, phân biệt được vật và lớp: gán riêng, kể cả cảnh đông. Đúng 20 px được gán.
3. `h < 20 px`: không gán box đối tượng riêng; vẽ `ignore_region` ôm vùng chứa vật quá nhỏ. Một vật nhỏ đơn lẻ có thể có một ignore riêng; nhiều vật sát nhau có thể dùng chung một vùng gọn.
4. Vùng đông có vật lớn nhưng không thể tách biên/từng thực thể sau khi phóng to: đề xuất ignore phần không tách được, ghi nguyên nhân; không áp dụng cho toàn cảnh nếu còn nhiều người/xe tách được.
5. Ưu tiên chia thành nhiều ignore nhỏ để tránh phủ vật lớn rõ ràng. Không tạo box đối tượng nếu đã quyết định chính đối tượng đó được bỏ qua trong vùng ignore.
6. Nếu không tránh được giao nhau với đối tượng rõ, thu hẹp/chia lại vùng; đưa ra review nếu vùng ignore sẽ che phần lớn box hợp lệ.

Script chỉ miễn lỗi **box chưa ghép** khi ít nhất 50% diện tích box nằm trong **một** vùng ignore theo cách triển khai hiện tại; không cộng diện tích nhiều vùng và không tự miễn mọi lỗi lớp/thuộc tính. Vì vậy không dùng ignore để làm tăng IAA hoặc che nhãn sai.

### 3.9. Nhòe, tối và ảnh không có vật hợp lệ

- Thiếu sáng/nhòe không tự đồng nghĩa bị che. Nếu vẫn xác định được lớp, biên và đủ cao thì gán theo quy tắc chung.
- Vệt nhòe: bao lõi vật còn nhận dạng được, không chạy theo toàn vệt; nếu không xác định được biên ổn định thì tick `can_review` hoặc ghi ignore cục bộ nếu không thể tách vật.
- Không tăng sáng bằng cách sửa ảnh nguồn. Có thể phóng to khi đọc ảnh; mọi tọa độ và ngưỡng dựa trên ảnh nguồn.
- Ảnh chỉ có vật loại trừ có thể không có annotation; ảnh đó vẫn phải có trong danh sách `images` của COCO. Không dựng box để ảnh có nhãn.

## 4. Ca khó — nhật ký rà soát đủ 40 ảnh

Các dòng dưới là **đề xuất của bản dự thảo**, chưa phải nhãn đã vẽ hoặc quyết định calibration. Chưa gán sẵn mức che cho từng vật khi chưa đo/đối chiếu ở CVAT. Liên kết mở ảnh gốc để kiểm tra.

| Ảnh | Tình huống v0 chưa nói rõ | Quyết định đề xuất | Lý do / điểm cần kiểm |
|---|---|---|---|
| [img_001](../../../data/images/img_001.jpg) | Xe tải nhỏ và nhiều người ngồi, chân/thân bị bàn và xe che; có người bị cắt cạnh dưới. | Xe là `truck`; người tách riêng; box theo phần nhìn thấy, tách cắt mép khỏi che. | Xe nhỏ không tự thành `van`; không suy đoán phần thân người sau bàn. |
| [img_002](../../../data/images/img_002.jpg) | Xe chuyên chở mang nhiều ô tô/pickup, các xe bị khung sàn che. | Một `truck` cho tổ hợp chuyên chở và từng xe thật trên sàn theo lớp. | Xe được chở vẫn là đối tượng; khung chở gây che. Cần chốt quy ước rơ-moóc một tổ hợp. |
| [img_003](../../../data/images/img_003.jpg) | Người bán hàng mang đòn gánh, xe đạp và nhiều xe máy/người chồng nhau. | Tách `person`, `bicycle`, `motorcycle`; không kéo box người theo gánh/hàng. | V0 thiếu xe đạp; hàng rời không là bộ phận người. |
| [img_004](../../../data/images/img_004.jpg) | Hai tuk-tuk có người ngồi bên trong, phần xe che thân người. | Đề xuất `tuk_tuk`, mỗi xe một box; người nhìn thấy đủ cao có box riêng. | Không ép tuk-tuk vào xe máy hoặc ô tô; thêm lớp cần Pha 2 chốt. |
| [img_005](../../../data/images/img_005.jpg) | Người đi xe máy sát nhau, bị cắt mép; xe thân hộp phía sau. | Người/xe tách riêng; xe thân hộp đề xuất `van`; kiểm `truncated` từng vật. | Mũ bảo hiểm không loại người; che bởi người khác khác với cắt khung. |
| [img_006](../../../data/images/img_006.jpg) | Xe đạp, xe máy và người đi cùng dòng giao thông, nhiều vật xa. | Xe đạp dùng `bicycle`, xe máy dùng `motorcycle`, người dùng `person`; đo chiều cao vật xa. | Không phân loại xe bằng tư thế người ngồi; vật dưới 20 px vào ignore. |
| [img_007](../../../data/images/img_007.jpg) | Xe tải thùng kín, pickup và xe máy trong kho tối. | Xe có thùng tách cabin là `truck`, kể cả nhỏ; gán xe máy đủ nhìn thấy. | Thùng kín không đủ để gọi là `van`; tối không tự tăng mức che. |
| [img_008](../../../data/images/img_008.jpg) | Xe máy đỗ dày, xe có người và không người; người in trên mặt sau áo. | Gán từng xe đỗ nhận dạng được và từng người thật; không gán hình chân dung trên áo. | Trạng thái đỗ không loại xe; hình in không phải người thứ hai. |
| [img_009](../../../data/images/img_009.jpg) | Xe khách thân dài giữa cảnh, v0 không có lớp bus. | Đề xuất `bus`, các xe/người xung quanh theo lớp riêng. | Không gọi mọi xe chở khách thân hộp là `van`; cần chốt bus/minibus. |
| [img_010](../../../data/images/img_010.jpg) | Người bán hàng, người trên scooter và xe đẩy chở quả. | `person`/`motorcycle` riêng; xe đẩy bán hàng và hàng hóa ngoài phạm vi. | Không gán xe đẩy là xe máy chỉ vì có bánh; không mở box người theo túi hàng. |
| [img_011](../../../data/images/img_011.jpg) | Nhân vật Predator tiền cảnh và các tạo hình xe kim loại ở khu trưng bày. | Chưa chốt nhân vật là người hóa trang hay tượng, xe là xe thật độ hay tác phẩm mô phỏng; người thật rõ ở nền vẫn gán. | Không đủ chứng cứ chỉ từ tên ảnh. Đưa các vật nghi vấn ra họp; không khẳng định thật/giả để gán hàng loạt. |
| [img_012](../../../data/images/img_012.jpg) | Thùng pickup đầy hàng sát camera; SUV bên trái và xe phía trước. | SUV là `car`; đề xuất phần pickup nhìn thấy là `truck`, hàng rời không mở rộng box; vấn đề ego cần chốt. | Tiền cảnh không đồng nghĩa xe chủ; thiếu căn cứ máy ảnh gắn trên pickup. |
| [img_013](../../../data/images/img_013.jpg) | Đường ướt ban đêm, người/xe thật và phản sáng dưới mặt đường. | Chỉ gán người/xe thật; bỏ phản chiếu/vệt đèn; kiểm vật xa theo 20 px. | Không tạo bản sao đối tượng trên đường ướt. |
| [img_014](../../../data/images/img_014.jpg) | Hai người đi bộ rõ, người/xe phía sau nhòe chuyển động. | Hai người chính gán `person`; vật nhòe chỉ gán khi nhận dạng được lớp và lõi biên. | Nhòe không phải che khuất; vùng không giải nghĩa được cần review/ignore cục bộ. |
| [img_015](../../../data/images/img_015.jpg) | Ô tô bị cắt mạnh hai bên, xe chuyển động nhòe giữa ảnh, phản chiếu trên đường. | Xe hai bên vẫn gán `car` theo phần nhìn thấy và `truncated=yes`; không gán phản chiếu. | Không loại xe vì chỉ thấy đầu/đuôi; không kéo box theo vệt nước/vệt nhòe. |
| [img_016](../../../data/images/img_016.jpg) | Tuk-tuk, xe máy, van và người, vật cắt ở các cạnh. | Tách theo lớp đề xuất; người lái và xe riêng; đánh dấu cắt mép từng vật. | Không gom mọi phương tiện ba bánh vào `motorcycle`. |
| [img_017](../../../data/images/img_017.jpg) | Tượng người cưỡi ngựa nhìn giống người thật. | Không gán tượng thành `person`; ngựa và nón giao thông ngoài phạm vi. | Vật mô phỏng không là người; ảnh có thể không có annotation hợp lệ. |
| [img_018](../../../data/images/img_018.jpg) | Chân dung trên billboard và ô tô sau hàng rào kim loại. | Không gán chân dung; gán từng xe nhận biết được, mức che xét hàng rào. | Hình in khác người thật; hàng rào không khiến xe tự biến thành vùng ignore. |
| [img_019](../../../data/images/img_019.jpg) | Ô tô hoạt hình Danger Mouse trong đoàn diễu hành và đám đông khán giả dính nhau sau hàng rào. | Xe vật lý là `car` (`can_review=true`); người lái thật gán `person`; cụm đám đông nhiều người dính nhau sau rào bôi toàn bộ là `ignore_region`, không vẽ box người riêng lẻ; nhân vật chuột chưa đủ căn cứ là người thì không tự gán `person`. | Thiết kế hư cấu khác mô hình nhỏ. Đám đông chen chúc dính sát nhau khoanh `ignore_region` để tránh tranh chấp ranh giới. |
| [img_020](../../../data/images/img_020.jpg) | Xe chỉ nhìn qua gương; gương/khung cửa thuộc xe chụp. | Loại xe phản chiếu và phần ego; có thể để ảnh không có annotation. | Không coi gương như cửa sổ nhìn xuyên và không nhân đôi xe chủ. |
| [img_021](../../../data/images/img_021.jpg) | Dòng xe máy rất đông, người và xe xa chỉ lộ một phần nhỏ. | Gán riêng phần gần/tách được; ignore gọn phần dưới 20 px hoặc không tách được. | Không một box cho cả đám đông; không phủ ignore lên người/xe lớn rõ. |
| [img_022](../../../data/images/img_022.jpg) | Mô tô có sidecar trong ảnh đen trắng, dễ bị coi là hai xe hay tuk-tuk. | Một `motorcycle` bao cụm xe và sidecar. | Sidecar gắn với mô tô là bộ phận của tổ hợp, khác xe ba bánh chuyên dụng. |
| [img_023](../../../data/images/img_023.jpg) | Xích lô trung tâm, người đạp và ô tô phía trái. | Một `cyclo` và một `person` cho người đạp; ô tô là `car`. | V0 thiếu xích lô; không gộp cơ thể người vào box xe. |
| [img_024](../../../data/images/img_024.jpg) | Wagon thật trong khu trưng bày, vật dạng ô tô màu vàng đặt trên nóc. | Wagon và các xe thật xung quanh là `car`; đề xuất loại vật vàng như mô hình/vỏ trưng bày, cần họp xác nhận. | Đặt trên xe thật không chứng minh vật được chở cũng là xe thật; khác `img_002`. |
| [img_025](../../../data/images/img_025.jpg) | Người xa trên lối đi, xe trên đường; xe máy có phần gắn bên cạnh. | Người/xe đủ điều kiện gán riêng; phần gắn cố định bên xe thuộc cùng cụm motorcycle, hàng rời không mở box. | Đo vật xa thay vì bỏ hết; kiểm cấu trúc phần gắn để tránh đếm đôi. |
| [img_026](../../../data/images/img_026.jpg) | Hai người cùng xe máy trong hầm, xe sát mép dưới và biển tranh phía sau. | Một `motorcycle`, hai `person` nếu đủ phần nhìn thấy; kiểm cắt mép xe; tranh tường không gán. | Không một box chung cho hai người; không suy diễn người lái hoàn toàn lộ rõ. |
| [img_027](../../../data/images/img_027.jpg) | Đài tưởng niệm có hình pickup rất giống xe. | Không gán tượng pickup thành `truck`/`car`, không dùng ignore chỉ để che tượng. | Bệ và bối cảnh tưởng niệm cho thấy đây là vật mô phỏng; giữ nền âm tính. |
| [img_028](../../../data/images/img_028.jpg) | Pickup thật chở tượng ngựa, ô tô đỏ phía sau. | Pickup là `truck`, ô tô đỏ là `car`; ngựa trang trí ngoài taxonomy. | Xe thật chở vật giả vẫn là xe thật; không mở mép trên truck theo đầu ngựa. |
| [img_029](../../../data/images/img_029.jpg) | Thanh kim loại dài vượt xa xe máy, xe đạp và người. | `motorcycle`, `bicycle`, `person` riêng; thanh kim loại không mở box người/xe. | Phân biệt hàng hóa với bộ phận cố định, tránh box quá rộng. |
| [img_030](../../../data/images/img_030.jpg) | Bãi xe máy chồng lấn; chân dung in trên poster phía trên. | Gán từng motorcycle còn phân biệt được; phần không tách được ignore cục bộ; không gán người trên poster. | Không đếm gương/bánh thành xe riêng; không loại cả bãi vì đông. |
| [img_031](../../../data/images/img_031.jpg) | Bus, ô tô và xe máy ban đêm, phản sáng mạnh trên đường. | Đề xuất `bus`; gán người/xe thật; loại vệt đèn và phản chiếu. | Không đồng nhất bus với van, không lấy vùng sáng làm biên xe. |
| [img_032](../../../data/images/img_032.jpg) | Người xếp hàng sát nhau tại trạm, cột/hộp kỹ thuật che thân, xe cắt mép trái. | Gán từng `person` còn tách được; xe trái theo phần lộ ra và `truncated=yes`. | Đông chưa đủ điều kiện ignore; vật che làm thay đổi mức che chứ không số người. |
| [img_033](../../../data/images/img_033.jpg) | Ma-nơ-canh, người in trên ảnh quảng cáo và hình xe trong kính. | Loại ma-nơ-canh, hình in, xe phản chiếu; không tự gán người/van. | Cần phân biệt phản chiếu với xe nhìn xuyên kính; ca này dùng kiểm nhãn thừa. |
| [img_034](../../../data/images/img_034.jpg) | Người đi bộ cắt qua dòng xe, người/xe ở cạnh phải bị cắt, một số xe đạp. | Người và từng xe theo lớp riêng; kiểm che và cắt mép độc lập. | Không coi người đi ngang là bộ phận xe sau lưng; xe đạp không là motorcycle. |
| [img_035](../../../data/images/img_035.jpg) | Auto-rickshaw chở chồng hàng rất cao, xe máy và người phía trước. | Đề xuất `tuk_tuk` cho xe ba bánh; box xe dừng ở bộ phận xe, không theo đỉnh hàng; người/xe máy riêng. | Chở quá tải không đổi lớp; hàng hóa không là phần thân xe. |
| [img_036](../../../data/images/img_036.jpg) | Xe tải rất nhỏ, thùng phủ bạt dễ bị nhầm van. | `truck`, bao cabin và thùng xe cố định. | Cấu trúc cabin/thùng tách biệt quyết định lớp, không dựa vào kích thước. |
| [img_037](../../../data/images/img_037.jpg) | Pickup lắp nắp thùng cao, bus và người ở nền. | Pickup vẫn `truck`; bus đề xuất `bus`; người gán riêng nếu đủ điều kiện. | Nắp thùng không biến pickup thành van; không gộp xe vào nền trạm. |
| [img_038](../../../data/images/img_038.jpg) | Nhiều ô tô mô hình, búp bê/nhân vật trong tủ kính. | Không gán đồ chơi thành `car`/`person`; ảnh có thể không có annotation. | Đây là âm tính khó; không dùng ignore cho toàn tủ để tránh phát hiện nhãn thừa. |
| [img_039](../../../data/images/img_039.jpg) | Phố có người, xe đạp, xe máy đỗ/chạy và xe xa. | Gán riêng `person`, `bicycle`, `motorcycle`, xe ô tô đủ điều kiện; kiểm dưới 20 px ở xa. | Không bỏ xe đỗ, không thêm person khi chỉ nhìn thấy xe trống. |
| [img_040](../../../data/images/img_040.jpg) | Người đội mũ kín mặt ở tiền cảnh, xe máy chỉ lộ ít; taxi, tuk-tuk, SUV, bus phía sau. | Người là `person`; taxi/SUV `car`, tuk-tuk và bus theo lớp đề xuất; motorcycle chỉ gán nếu phần xe nhìn thấy đủ nhận dạng và cao ≥20 px. | Không cần thấy mặt để là người; không dựng toàn xe dưới khung ảnh từ suy đoán. |

Các quy tắc còn chưa chốt phải được làm rõ trước v1.0. Không cần dùng `can_review` cho mọi vật chỉ vì thuộc lớp mới; cờ dành cho vật còn mơ hồ sau khi taxonomy đã chốt. Danh sách câu hỏi chuẩn bị họp được lưu trong [ghi chú cá nhân, mục 2](../../individual_notes/2A202603183_HoangNhatAnh.md#2-pha-2-chỗ-tôi-bất-đồng-với-nhóm).

## 5. QC và phiên bản

### 5.1. Tự kiểm khi gán và trước export

- Quét ảnh theo một chiều cố định rồi kiểm lại từ tiền cảnh tới hậu cảnh: người, phương tiện, vật nhỏ và các mép ảnh.
- Đối chiếu lớp với v1.0 đã chốt; không dùng lớp mới chỉ tồn tại trong bản đề xuất cá nhân.
- Kiểm box theo phần nhìn thấy, phần gương/bánh/tay chân bị cắt thiếu, hàng hóa làm box quá rộng, người và xe bị gộp.
- Kiểm đủ `occluded_level`, `truncated`, `can_review`; không để thuộc tính mặc định thay cho việc đánh giá.
- Kiểm không có box trên tượng, đồ chơi, hình in hoặc phản chiếu; không phủ ignore quá rộng.
- Ghi các ca cần review theo mẫu: `ảnh | vị trí/vật | điều chưa chắc | quy tắc liên quan`; không trao đổi đáp án với người gán cùng ảnh trong Pha 3.
- Export COCO 1.0 không kèm ảnh; ảnh không có đối tượng vẫn phải có trong dữ liệu export.
- Chạy `scripts/validate_labels.py` với file COCO đã export; xử lý hết lỗi và xem các cảnh báo trước khi nộp.

### 5.2. Ngưỡng QC giữ nguyên theo repo

| Chỉ số | Ngưỡng / cách hiểu |
|---|---|
| Chiều cao box đối tượng tối thiểu | `20 px`; dưới ngưỡng xử lý ignore theo mục 3.8. |
| Ghép box để so sánh | IoU ≥ `0.50`, ghép 1–1 không phụ thuộc lớp. |
| Độ khớp vị trí của cặp box lớn | IoU ≥ `0.85` khi chiều cao trung bình của hai box ≥ `50 px` theo script. |
| Độ khớp vị trí của cặp box nhỏ | IoU ≥ `0.70` khi chiều cao trung bình dưới `50 px`; box hợp lệ thông thường từ `20 px`. |
| IAA toàn lô | IoU trung bình ≥ `0.85` và đồng thuận lớp ≥ `95%`, tính trên cặp box ghép được. |
| Box không ghép được | Thống kê riêng; không giấu trong mẫu số của các cặp ghép được. |
| Release | Tỉ lệ ảnh có lỗi ≤ `2%`, IAA đạt, không còn lỗi hệ thống, không còn khoảng trống guideline/vấn đề dữ liệu. |

`IoU = diện tích giao / diện tích hợp`. Luôn ghi số cặp ghép khi báo IAA. Với nhóm 5 người, mỗi ảnh có 3 người gán, kết quả so sánh theo các cặp người; số cặp so sánh không phải số đối tượng duy nhất.

Mẫu QC trong lab: tất cả ảnh từng bị cờ và tối đa 20 ảnh chọn ngẫu nhiên từ phần còn lại, seed 42 theo script; báo cỡ mẫu thực tế, không mặc nhiên ghi đã kiểm 40 ảnh. Nếu nhóm chủ động kiểm đủ 40 ảnh thì phải ghi rõ đó là kiểm toàn bộ. Việc xem ảnh để viết guideline hiện tại không phải QC nhãn.

Năm mã lỗi: `bo_sot`, `thua`, `sai_lop`, `sai_vi_tri_kich_thuoc`, `sai_thuoc_tinh`. Ảnh đã kiểm sạch ghi `khong`. Mỗi lỗi một dòng trong phiếu QC.

Báo cả **ảnh có lỗi / ảnh trong mẫu** và **đối tượng lỗi / số đối tượng trong mẫu** theo script `qc_report.py`. Chỉ kết luận khi toàn bộ ảnh mẫu đã được kiểm. Script khử trùng đối tượng lỗi theo cặp `(image, object_id)`; lỗi bỏ sót không có ID được tính riêng theo từng dòng. Số dòng lỗi dùng đếm từng loại lỗi, không phải số đối tượng lỗi duy nhất. Tỉ lệ dùng quyết định Release là tỉ lệ **theo ảnh**, không phải theo đối tượng.

Ví dụ minh họa ngưỡng, không phải kết quả của lô: nếu kiểm đủ 40 ảnh, 1 ảnh lỗi là `1/40 = 2,5%`, đã vượt 2%; do đó phải có `0/40` ảnh lỗi mới đạt riêng điều kiện này.

### 5.3. Xử lý bất đồng và quyết định lô

1. **Hold:** guideline còn chỗ chưa rõ ảnh hưởng nhãn hoặc dữ liệu có vấn đề; làm rõ trước khi xét các ngưỡng còn lại.
2. **Rework:** ảnh lỗi >2%, IAA chưa đạt hoặc có lỗi hệ thống. Script nhận diện lỗi hệ thống khi một loại chiếm ≥50% số lỗi và có ít nhất 3 lỗi; vẫn cần người kiểm phân tích nguyên nhân.
3. **Release:** chỉ sau khi các điều kiện trên đã được kiểm và đạt, kèm mẫu số, phiên bản và người kiểm.

Khi hai người lệch lặp lại ở cùng kiểu ca, kiểm tra quy tắc; khi chỉ một người lệch, kiểm tra cách hiểu của người đó. Nếu cùng gán giống nhau nhưng sai thực tế, IAA cao cũng không chứng minh nhãn đúng.

Pha 5: người chốt do nhóm chỉ định. Mỗi quyết định thực tế phải có một dòng trong `calibration_decisions.csv`, mã ảnh/ca, nội dung, quy tắc, phiên bản và người chốt. Quy tắc mới đưa vào v1.1 của nhóm. Không sửa lại bản Pha 1 để làm mất dấu quyết định ban đầu.

Script kiểm được cấu trúc file, lớp, thuộc tính, tọa độ, ảnh đủ/thừa, box nhỏ và mức bất đồng. Script không biết vật trong box là thật hay giả, biên có bám đúng vật hay không, hoặc cả nhóm cùng bỏ sót gì. Cần kiểm bằng mắt với ảnh gốc và guideline.

### 5.4. Lịch sử phiên bản

| Phiên bản | Nội dung | Lý do / ca liên quan |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên, giữ tại `guideline/guideline_v0_starter.md`. | Thiếu định nghĩa ranh giới lớp, ngưỡng che và ca khó. |
| v0.1-personal — 16/11/2026 | Dự thảo cho Hoàng Nhật Anh: đủ 5 phần; đề xuất thêm 4 lớp; định lượng mức che; quy tắc phần thấy, hàng hóa, ego, ignore; rà 40 ảnh và lập danh sách cần họp. | Tuk-tuk/xích lô/bus/xe đạp; xe chở xe 002; ca chưa rõ 011/019/024; tượng/hình in/phản chiếu/đồ chơi 017/018/020/027/033/038. |
| v0.1-personal — chỉnh bố cục | Tách quá trình làm bài, câu hỏi chuẩn bị họp, phân công và sử dụng AI sang `individual_notes/2A202603183_HoangNhatAnh.md`; giữ quy tắc và ca khó trong guideline. | Phân biệt tài liệu quy tắc với ghi chú cá nhân theo mẫu của repo; không thay đổi quy tắc gán nhãn. |
