> **Nguyên bản:** [Appendix C. Example Incident State Document](https://sre.google/sre-book/incident-document/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (do AI hỗ trợ)

---

## Phụ lục C. Ví Dụ Tài Liệu Trạng Thái Sự Cố (Appendix C. Example Incident State Document)

**Quá tải Shakespeare Sonnet++: 2015-10-21**  
Thông tin quản lý sự cố (incident management): *https://incident-management-cheat-sheet*

*(Người phụ trách truyền thông (Communications lead) hãy giữ cho bản tóm tắt được cập nhật.)*  
**Tóm tắt (Summary)**: Dịch vụ tìm kiếm Shakespeare đang ở trạng thái sự cố lan truyền (cascading failure) do một sonnet mới vừa được phát hiện không có trong chỉ mục tìm kiếm.

**Trạng thái (Status)**: đang hoạt động (active), sự cố #465

**Trạm chỉ huy (Command Post(s))**: `#shakespeare` trên IRC

**Hệ thống chỉ huy (Command Hierarchy)** *(tất cả những người phản hồi)*

-   Incident Commander (Người chỉ huy sự cố) hiện tại: jennifer
    
    -   Người phụ trách vận hành (Operations lead): docbrown
    -   Người phụ trách lập kế hoạch (Planning lead): jennifer
    -   Người phụ trách truyền thông (Communications lead): jennifer
-   Incident Commander tiếp theo: *sẽ được xác định*
    

*(Cập nhật ít nhất mỗi bốn giờ và khi bàn giao vai trò Communications Lead.)*  
**Trạng thái chi tiết (Detailed Status)** (cập nhật lần cuối lúc 2015-10-21 15:28 UTC bởi jennifer)

**Tiêu chí thoát (Exit Criteria):**

-   Thêm sonnet mới vào tập hợp (corpus) tìm kiếm Shakespeare **TODO**
-   Đạt trong ngưỡng độ khả dụng (99.99%) và độ trễ (99%ile < 100 ms) SLO trong 30+ phút **TODO**

**Danh sách TODO và các bug đã được nộp:**

-   Chạy công việc MapReduce để lập chỉ mục lại tập hợp Shakespeare **DONE**
-   Mượn các tài nguyên khẩn cấp để đưa thêm năng lực lên **DONE**
-   Bật bộ tụ thông lượng (flux capacitor) để cân bằng tải giữa các cụm (Bug 5554823) **TODO**

**Trình tự thời gian của sự cố (Incident timeline)** *(mới nhất trước: thời gian tính theo UTC)*

-   2015-10-21 15:28 UTC jennifer
    
    -   Tăng năng lực phục vụ toàn cầu lên 2 lần
-   2015-10-21 15:21 UTC jennifer
    
    -   Điều hướng toàn bộ giao thông đến cụm hy sinh (sacrificial cluster) USA-2 và rút giao thông khỏi các cụm khác để chúng có thể phục hồi từ sự cố lan truyền trong khi khởi động thêm các task
    -   Công việc lập chỉ mục MapReduce hoàn tất, chờ sao chép Bigtable đến tất cả các cụm
-   2015-10-21 15:10 UTC martym
    
    -   Thêm sonnet mới vào tập hợp Shakespeare và khởi động MapReduce lập chỉ mục
-   2015-10-21 15:04 UTC martym
    
    -   Lấy được văn bản của sonnet mới được phát hiện từ danh sách thư (mailing list) *shakespeare-discuss@*
-   2015-10-21 15:01 UTC docbrown
    
    -   Tuyên bố sự cố do sự cố lan truyền
-   2015-10-21 14:55 UTC docbrown
    
    -   Bão trang (pager storm), `ManyHttp500s` ở tất cả các cụm

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
