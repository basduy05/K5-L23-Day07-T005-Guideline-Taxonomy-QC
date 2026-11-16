# Nhãn của từng thành viên

Mỗi thành viên **một file COCO 1.0** export từ CVAT: `<MSHV>_<HoTen>.json`, họ tên không dấu viết liền.

File của Hoàng Nhật Anh: **`2A202603183_HoangNhatAnh.json`**.
Chỉ chứa các ảnh được giao trong `submission/qc/assignments/2A202603183.txt`; script tách MSHV từ tên file để đối chiếu với `assignment.csv`.

```powershell
python scripts/validate_labels.py submission/labels/2A202603183_HoangNhatAnh.json
```

Các script vẫn đọc được tên mã ngắn cũ như `101.json` cho dữ liệu đã có. Mỗi MSHV chỉ có một file nhãn, tránh nộp cả hai cách đặt tên cho cùng một người.
