# Ghi chú cá nhân — 2A202603040 - Nguyễn Đức Cường

## 1. Pha 1: ba ca khó tôi tìm ra trong 40 ảnh

| Ảnh | v0 không trả lời được gì | Quyết định ban đầu của tôi |
|---|---|---|
| img_002 | Xe tải chở nhiều xe con: v0 chỉ nói "gán mọi phương tiện", không nói phương tiện chở trên phương tiện khác gán thế nào. | 1 box `truck` cho xe tải, thêm 1 box `car` cho mỗi xe con nhìn rõ trên thùng. |
| img_011 | Phương tiện chỉ để trưng bày: v0 không phân biệt xe thật đang lưu thông với vật trưng bày. | Không gán box phương tiện; vùng lớn thì khoanh `ignore_region`. |
| img_023 | Xích lô (xe đạp) chở thêm đồ: v0 không có lớp cho xe không động cơ, không nói đồ chở trên xe tính thế nào. | Không gán box xe và đồ chở; chỉ gán `person` cho người đạp / người ngồi. |

Các ca khác tôi ghi nhận: img_003 (người trên xe đạp / xe máy), img_005 (chỉ thấy nửa người), img_014 (người bị nhoè), img_017 (tượng người).

## 2. Pha 2: chỗ tôi bất đồng với nhóm

- **Mở rộng taxonomy cho giao thông Đông Nam Á:**
  - Quan điểm ban đầu của tôi: Form gốc chỉ có 6 lớp, không có chỗ cho xe buýt, xe đạp và tuk-tuk. Nếu gộp xe buýt nhỏ (img_009) vào `van`/`truck` hay gộp tuk-tuk (img_040, img_035) vào `motorcycle` thì dữ liệu bị bẩn. Tôi đề xuất thêm `bus`, `bicycle`, `three_wheeler`, nâng lên 9 lớp.
- **Xích lô (img_023):**
  - Quan điểm của tôi: Guideline v1.0 không có lớp nào khớp. `three_wheeler` chỉ dành cho xe ba bánh có động cơ, còn `bicycle` mô tả xe hai bánh. Khi gán, tôi xếp xích lô vào `bicycle` vì nó chạy bằng bàn đạp, người đạp gán `person` riêng, và tick `can_review`.
- **Hàng hoá rời cồng kềnh (img_035):**
  - Quan điểm của tôi: Box xe chỉ bao thân xe và phụ kiện gắn cố định (gương, thùng giao hàng, giá nóc). Hàng chất rời thì không bao, kể cả khi nó cao gấp đôi xe, như chồng bao tải trên tuk-tuk ở img_035 (quy tắc 3.6).
- **Người trong xe kín:** Tôi chủ trương không gán `person` cho người ngồi trong ô tô, cabin xe tải, van hay xe buýt, chỉ gán người trên xe hở (ca khó #21). 

## 3. Pha 3: ảnh tôi đã gán

- Số ảnh được giao: 24 (theo `submission/qc/assignments/2A202603040.txt`). Số box đã vẽ: ~226 (trên 23 ảnh; ~105 `person`, ~70 `motorcycle`, ~35 `car`, còn lại là `truck`/`van`/`bus`/`bicycle`/`three_wheeler`/`ignore_region`). Số lần tôi tick `can_review`: 7
- Ca lưỡng lự nhất: img_023, chiếc xích lô. Guideline v1.0 không có lớp nào khớp: `three_wheeler` chỉ dành cho xe ba bánh có động cơ, còn `bicycle` là xe hai bánh. Tôi gán `bicycle` vì xe chạy bằng bàn đạp, người đạp gán `person` riêng, và tick `can_review`. Đề xuất cho v1.1: ghi rõ xích lô và xe ba gác đạp thuộc `bicycle`, hoặc tách thành một lớp riêng.

## 4. Pha 4–5: tôi lệch ở đâu

- Các ca có tên tôi trong `calibration_candidates.csv`: […số ca, ảnh nào…]. Dựa trên những ca tôi đã tick `can_review` lúc gán, tôi dự đoán mình dễ lệch nhất ở:
  - Xe máy bị cắt mép hoặc bị người lái che gần hết (img_040), cùng mức `occluded_level` của người ngồi sau.
  - Ngưỡng che. Guideline v1.0 của tôi dùng 10 / 40 / 80%. 
  - Ranh giới giữa vùng `ignore_region` và vùng gán từng xe ở các dãy xe máy đỗ dày (img_030, img_021).


## 5. Dùng AI

- Công cụ: Claude (Claude Code).
- Dùng cho: đọc repo và tóm tắt các bước làm lab; sửa lỗi chạy script (`python` không có trên PATH, lỗi đăng nhập CVAT); rút gọn và định dạng file guideline cá nhân; viết định nghĩa chung cho các lớp/thuộc tính; gợi ý phương án A/B cho các ca khó tôi tự tìm (img_002, img_011, img_014) để tôi chọn; chuyển nội dung guideline từ doc sang Markdown.
- Không dùng AI để xem ảnh và quyết định gán gì: các ca khó ở mục 1 do tôi tự tìm khi xem ảnh.
- Cách kiểm tra: *(điền: ví dụ đối chiếu lại từng ca khó với ảnh thật, chạy `validate_labels.py`)*

## 6. Bài học

- *(điền cuối bài)*
