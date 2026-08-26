> **Nguyên bản:** [Appendix E. Launch Coordination Checklist](https://sre.google/sre-book/launch-checklist/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

## Phụ lục E. Danh Mục Kiểm Tra Phối Hợp Ra Mắt (Appendix E. Launch Coordination Checklist)

Đây là Danh Mục Kiểm Tra Phối Hợp Ra Mắt (Launch Coordination Checklist) nguyên bản của Google, khoảng năm 2005, được lược giản đôi chút cho ngắn gọn:

#### Kiến trúc (Architecture)

-   Sơ đồ kiến trúc, các loại server, các loại yêu cầu từ client
-   Các yêu cầu client bằng chương trình (programmatic)

#### Máy chủ và datacenter (Machines and datacenters)

-   Máy và băng thông (bandwidth), datacenter, dự phòng N+2, QoS mạng
-   Các tên miền (domain name) mới, cân bằng tải DNS

#### Ước tính khối lượng, năng lực, và hiệu suất (Volume estimates, capacity, and performance)

-   Ước tính giao thông HTTP và băng thông, "đỉnh" (spike) khi ra mắt, tỷ lệ giao thông (traffic mix), trong 6 tháng tới
-   Kiểm thử tải, kiểm thử đầu-cuối (end-to-end test), năng lực mỗi datacenter ở độ trễ tối đa
-   Tác động đến các dịch vụ khác mà chúng tôi quan tâm nhất
-   Năng lực lưu trữ

#### Độ tin cậy hệ thống và dự phòng (System reliability and failover)

-   **Điều gì xảy ra khi:**
    -   Máy chết, rack (giá máy) lỗi, hoặc cụm (cluster) offline
    -   Mạng bị lỗi giữa hai datacenter

-   **Đối với mỗi loại server giao tiếp với các server khác (các backend của nó):**
    -   Cách phát hiện khi các backend chết, và phải làm gì khi chúng chết
    -   Cách kết thúc hoặc khởi động lại mà không ảnh hưởng đến client hay người dùng
    -   Hành vi cân bằng tải, giới hạn tỷ lệ (rate-limiting), thời gian chờ (timeout), thử lại (retry) và xử lý lỗi
-   Sao lưu/phục hồi dữ liệu (Data backup/restore), phục hồi thảm họa (disaster recovery)

#### Giám sát và quản lý server (Monitoring and server management)

-   Giám sát trạng thái nội bộ, giám sát hành vi đầu-cuối, quản lý cảnh báo
-   Giám sát cái giám sát (monitoring the monitoring)
-   Các cảnh báo và log quan trọng về mặt tài chính
-   Mẹo để chạy các server trong môi trường cụm
-   Đừng làm crash các mail server bằng cách tự gửi email cảnh báo cho chính mình trong mã nguồn server của chính bạn

#### Bảo mật (Security)

-   Rà soát thiết kế bảo mật, kiểm tra mã nguồn bảo mật, rủi ro spam, xác thực (authentication), SSL
-   Khả năng hiển thị/kiểm soát truy cập trước khi ra mắt, các loại danh sách đen (blacklist) khác nhau

#### Tự động hóa và tác vụ thủ công (Automation and manual tasks)

-   Các phương pháp và kiểm soát thay đổi (change control) để cập nhật server, dữ liệu, và cấu hình
-   Quy trình phát hành (release process), các bản build có thể lặp lại, canary dưới giao thông thực (live traffic), triển khai theo từng giai đoạn

#### Các vấn đề tăng trưởng (Growth issues)

-   Năng lực dự phòng, tăng trưởng 10 lần, cảnh báo tăng trưởng
-   Các nút cổ chai (bottleneck) của khả năng mở rộng, mở rộng tuyến tính (linear scaling), mở rộng theo phần cứng, các thay đổi cần thiết
-   Bộ nhớ đệm (Caching), phân mảnh dữ liệu (data sharding/resharding)

#### Các phụ thuộc bên ngoài (External dependencies)

-   Các hệ thống bên thứ ba, giám sát, mạng, khối lượng giao thông, các đỉnh khi ra mắt
-   Suy giảm tinh tế (Graceful degradation), cách tránh vô tình vượt quá các dịch vụ bên thứ ba
-   Ứng xử tử tế với các đối tác phối hợp (syndicated partners), các hệ thống thư, các dịch vụ trong Google

#### Lịch trình và quy hoạch triển khai (Schedule and rollout planning)

-   Các hạn chót cố định, các sự kiện bên ngoài, thứ Hai hoặc thứ Sáu
-   Các quy trình vận hành chuẩn (Standard operating procedures) cho dịch vụ này, cho các dịch vụ khác

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
