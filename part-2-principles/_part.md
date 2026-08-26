# Phần II. Nguyên lý (Principles)

> **Nguyên bản:** [Part II - Principles](https://sre.google/sre-book/part-II-principles/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Đội biên tập Softdreams RnD)

---

Phần này xem xét các *nguyên lý* nền tảng cho [cách các đội SRE thường làm việc](https://sre.google/workbook/reaching-beyond/) — các khuôn mẫu, hành vi và các mối quan tâm ảnh hưởng đến lĩnh vực tổng quát của vận hành SRE.

Chương đầu tiên trong phần này, [Embracing Risk](03-embracing-risk.md), là phần đáng đọc nhất nếu bạn muốn có bức tranh rộng nhất về SRE thực sự làm gì và chúng ta suy luận về nó như thế nào. Nó nhìn SRE qua lăng kính của rủi ro — từ việc đánh giá, quản lý rủi ro đến việc dùng error budget (ngân sách lỗi) — nhằm mang lại những cách tiếp cận trung lập, hữu ích cho [quản lý dịch vụ](https://sre.google/sre-book/part-III-practices/).

Service level objective (mục tiêu mức dịch vụ) là một đơn vị khái niệm nền tảng khác của SRE. Ngành thường gộp nhiều khái niệm khác nhau dưới danh xưng chung là service level agreement (thỏa thuận mức dịch vụ), khiến việc suy nghĩ rõ ràng về chúng trở nên khó hơn. [Service Level Objectives](04-service-level-objectives.md) cố tách rời các indicator (chỉ báo), objective (mục tiêu) và agreement (thỏa thuận) khỏi nhau, xem xét cách SRE dùng mỗi thuật ngữ, và gợi ý cách tìm các metrics (chỉ số) hữu ích cho ứng dụng của bạn.

Loại bỏ toil (công việc vận hành lặp đi lặp lại) là một trong những nhiệm vụ quan trọng nhất của SRE và là chủ đề của [Eliminating Toil](05-eliminating-toil.md). Chúng tôi định nghĩa *toil* là công việc vận hành tầm thường, lặp lại, không mang lại giá trị lâu dài, và tăng tuyến tính theo quy mô dịch vụ.

Dù ở Google hay ở nơi khác, monitoring (giám sát) là thành phần thiết yếu để làm đúng việc trong production (môi trường chạy thật). Nếu không thể giám sát một dịch vụ, bạn không biết điều gì đang xảy ra; và nếu không thấy điều gì đang xảy ra, bạn không thể đáng tin cậy. Đọc [Monitoring Distributed Systems](06-monitoring-distributed-systems.md) để có một số khuyến nghị về việc giám sát cái gì và như thế nào, cùng một số best practices (thực hành tốt nhất) không phụ thuộc vào cách cài đặt.

Trong [The Evolution of Automation at Google](07-automation-at-google.md), chúng tôi xem xét cách tiếp cận của SRE đối với tự động hóa, và đi qua một số nghiên cứu tình huống về cách SRE đã triển khai tự động hóa, cả thành công lẫn thất bại.

Phần lớn các công ty coi release engineering (kỹ thuật phát hành) là điều nghĩ tới sau cùng. Tuy nhiên, như bạn sẽ thấy trong [Release Engineering](08-release-engineering.md), release engineering không chỉ then chốt cho sự ổn định tổng thể của hệ thống — vì phần lớn outage (mất dịch vụ) bắt nguồn từ việc đẩy một thay đổi nào đó — mà còn là cách tốt nhất để đảm bảo các release nhất quán.

Một nguyên lý then chốt của mọi hoạt động kỹ thuật phần mềm hiệu quả, không chỉ kỹ thuật định hướng độ tin cậy, là sự đơn giản — phẩm chất mà, một khi đã mất, rất khó lấy lại. Dù vậy, như thành ngữ xưa nói, một hệ thống phức tạp hoạt động được chắc chắn đã tiến hóa từ một hệ thống đơn giản hoạt động được. [Simplicity](09-simplicity.md) đi sâu vào chủ đề này.

## Đọc thêm từ Google SRE

Tăng tốc độ sản phẩm một cách an toàn là nguyên lý cốt lõi của mọi tổ chức. Trong "Making Push On Green a Reality" [[Kle14]](https://sre.google/sre-book/bibliography#Kle14), xuất bản tháng 10 năm 2014, chúng tôi chỉ ra rằng việc đưa con người ra khỏi quy trình release có thể ngược lại làm giảm toil của SRE trong khi vẫn *tăng* độ tin cậy của hệ thống.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
