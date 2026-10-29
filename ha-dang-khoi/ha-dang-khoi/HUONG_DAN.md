# Hướng dẫn chung — Building a Chinese-Vietnamese parallel corpus on Genealogy

Đợt giao: 2026-10-26. Căn cứ: biên bản họp 26/10/2026.

## Quy trình mỗi cuốn
- **B0** Điền `phieu_B0.csv`: dạng tư liệu, số trang thật, có bản dịch hay phiên âm không, chất lượng ảnh, trang thiếu hoặc trùng.
- **B1** `ocr/`: chạy OCR nhiều công cụ có bỏ phiếu của lab, rồi soát nhanh trang có nhiều ký tự không chắc. Chỉ chạy khi đã được cấp quyền và hạn mức.
- **B2** `dich_may/`: đã có chữ Hán, cần dịch máy, sau đó soát như B3.
- **B3** `soat/`: mở `cap_cau.csv` (Excel hoặc Google Sheets). Mỗi dòng là một cặp câu Hán – bản dịch máy. Đối chiếu với ảnh trong `anh/` và ngữ cảnh trong `trang.jsonl`.
  - Bản dịch đúng: để trống `viet_sua`, đặt `trang_thai` = `da_soat`.
  - Bản dịch sai hoặc thiếu: ghi bản đúng vào `viet_sua`, đặt `trang_thai` = `da_soat`.
  - Câu ghép sai (một câu Hán bị cắt đôi, hai câu bị gộp): ghi vào `ghi_chu`, đặt `trang_thai` = `lech_nghia`.
  - Không chắc nghĩa: đặt `trang_thai` = `can_hoi`, không tự đoán.
- **B4** `dich_nguoi/`: ghép câu Hán với câu trong bản dịch người (không phải bản dịch máy). Ghi `nguon_dich` = `nguoi`.

## Khung chữ và ảnh crop: raw/ → final/ (định dạng PPOCRLabel)
Cuốn nào đã có khung chữ Paddle thì thư mục cuốn có sẵn:
- `raw/Label.txt`: khung tự động, mỗi dòng `anh/<trang>.jpg<TAB>[{"transcription": ..., "points": [[x,y]×4], "difficult": false}, ...]`, xếp theo thứ tự đọc (cột phải sang trái, trong cột trên xuống dưới). Toạ độ tính trên ảnh trong `anh/`.
- `raw/rec_gt.txt`: `crop_img/<trang>_crop_<i>.jpg<TAB><chữ OCR>`.
- `crop_img/`: mỗi khung một ảnh; cột chữ dọc đã được xoay 90° giống cách PPOCRLabel tự cắt.
- `raw/fileState.txt` để trống: chưa trang nào được người xác nhận.

Lưu ý: chữ trong `raw/` là chữ do riêng Paddle nhận dạng, có cột sai nặng (ví dụ ra toàn chữ giản thể vô nghĩa). Khi sửa chữ, đối chiếu ảnh với chữ đã bỏ phiếu giữa nhiều công cụ trong `trang.jsonl` (trường `han`), thường đúng hơn. Khung trùng với dấu thư viện, số trang (ví dụ `R.42`, `-57-`) thì xoá.

Cách làm:
1. Chép `raw/Label.txt` vào thư mục `anh/`, mở `anh/` bằng PPOCRLabel (https://github.com/PFCCLab/PPOCRLabel).
2. Sửa khung: thẳng, sát chữ, không thừa nền, không cắt thiếu chữ, mỗi cột (dòng) một khung, không bỏ sót cột.
3. Sửa chữ: chép đúng chữ trên ảnh, giữ chữ dị thể, không đổi sang giản thể, không "sửa lỗi" của bản gốc.
4. Xác nhận từng trang, rồi xuất kết quả nhận dạng (Export Recognition Result).
5. Chép `Label.txt`, `fileState.txt`, `rec_gt.txt` mới vào `final/`, ảnh crop mới vào `final_crop/`. **Không sửa gì trong `raw/`** để còn đo độ chính xác của OCR tự động.
Cuốn chưa có khung (thư mục `ocr/` và vài cuốn khác) thì tự tạo khung bằng PPOCRLabel sau khi OCR, rồi làm tiếp như trên.

## Nộp lại
- `phieu_B0.csv` đã điền, và mỗi cuốn một tệp `cap_cau.csv` đã soát (giữ nguyên cột `id`).
- Khoảng 5% số trang sẽ được một bạn khác soát lại để đo độ đồng thuận.

## Quy tắc
- Không đưa ảnh, PDF hay dữ liệu này lên mạng công khai hoặc git.
- OCR và dịch máy dùng hạn mức của lab; hết hạn mức thì dừng và báo.
- Giữ đúng `doc_id`; không tự đặt mã định danh F.
