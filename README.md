# SRE Book (Bản dịch tiếng Việt)

Bản dịch tiếng Việt của [Google SRE Book](https://sre.google/sre-book/table-of-contents/) (Site Reliability Engineering) — O'Reilly, 2017. Dịch bởi Đội biên tập Softdreams RnD, rà soát nhiều lượt để đảm bảo thuật ngữ nhất quán và văn phong tự nhiên.

**Đọc online:** https://dat911zz.github.io/sre-vie-book/

## Nội dung

34 chương + Lời tựa/Lời nói đầu + 6 phụ lục + Bibliography, chia theo 5 phần đúng cấu trúc sách gốc:

- Phần I — Giới thiệu
- Phần II — Nguyên lý
- Phần III — Thực hành
- Phần IV — Quản lý
- Phần V — Kết luận
- Phụ lục A–F

Xem đầy đủ tại [toc.md](toc.md) hoặc mục lục bên trái trang web.

## Đọc offline / tự host

Không cần build. Chỉ cần serve tĩnh thư mục này:

```bash
python -m http.server 8080
# hoặc: npx serve .
```

Rồi mở `http://localhost:8080/`. Site dùng [Docsify](https://docsify.js.org/) — toàn bộ style/search/dark-mode cấu hình trong [index.html](index.html).

## Chất lượng dịch & cách đóng góp

Bản dịch được duy trì theo quy trình 6 giai đoạn (glossary chung → dịch → merge → áp ngược → làm mượt văn phong → kiểm tra nghĩa), định nghĩa tại `.kilo/command/translate-book.md` trong repo gốc (không đi kèm repo này). Thuật ngữ chuẩn hóa xem tại [GLOSSARY.md](GLOSSARY.md); lịch sử rà soát xem [REVIEW-REPORT.md](REVIEW-REPORT.md).

Root cause đã biết: các đợt dịch/rà soát ban đầu chỉ đảm bảo nhất quán thuật ngữ **trong 1 file**, không có glossary dùng chung toàn sách — nên một số chương vẫn có thể còn sót thuật ngữ chưa đồng bộ hoặc câu dịch chưa tự nhiên. Phát hiện lỗi thì mở issue kèm trích dẫn câu gốc + vị trí file.

## Giấy phép

Nội dung gốc: Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc., licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/). Bản dịch giữ nguyên giấy phép này — không dùng cho mục đích thương mại, không tạo bản phái sinh ngoài phạm vi dịch thuật.
