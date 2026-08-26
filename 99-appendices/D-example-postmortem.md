> **Nguyên bản:** [Appendix D. Example Postmortem](https://sre.google/sre-book/example-postmortem/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Đội biên tập Softdreams RnD)

---

## Phụ lục D. Ví Dụ Postmortem (Appendix D. Example Postmortem)

## Postmortem Shakespeare Sonnet++ (sự cố #465)

**Ngày (Date)**: 2015-10-21

**Tác giả (Authors)**: jennifer, martym, agoogler

**Trạng thái (Status)**: Hoàn tất, các mục hành động (action items) đang được thực hiện

**Tóm tắt (Summary)**: Shakespeare Search ngừng hoạt động trong 66 phút trong khoảng thời gian có sự quan tâm rất cao đối với Shakespeare do việc phát hiện một sonnet mới.

**Tác động (Impact)**:<sup>[1](#fn1)</sup> Ước tính 1.21B truy vấn bị mất, không có tác động về doanh thu.

**Nguyên nhân gốc rễ (Root Causes)**:<sup>[2](#fn2)</sup> Sự cố lan truyền do sự kết hợp giữa tải cực cao và một rò rỉ tài nguyên khi các lần tìm kiếm thất bại do các từ ngữ không có trong tập hợp (corpus) Shakespeare. Sonnet mới được phát hiện sử dụng một từ chưa bao giờ xuất hiện trước đó trong một trong các tác phẩm của Shakespeare, và tình cờ lại là từ mà người dùng tìm kiếm. Trong điều kiện bình thường, tỷ lệ task thất bại do rò rỉ tài nguyên thấp đến mức không bị nhận ra.

**Chất xúc tác (Trigger)**: Lỗi tiềm ẩn (latent bug) bị kích hoạt do sự tăng vọt đột ngột về giao thông.

**Giải quyết (Resolution)**: Điều hướng giao thông đến cụm hy sinh và thêm 10 lần năng lực để giảm nhẹ sự cố lan truyền. Bản cập nhật chỉ mục được triển khai, giải quyết tương tác với lỗi tiềm ẩn. Duy trì năng lực bổ sung cho đến khi sự quan tâm của công chúng đối với sonnet mới hạ nhiệt. Rò rỉ tài nguyên được xác định và bản vá được triển khai.

**Phát hiện (Detection)**: Borgmon phát hiện mức HTTP 500 cao và gọi (page) người on-call.

**Các Mục Hành Động (Action Items)**:<sup>[3](#fn3)</sup>

Mục Hành Động | Kiểu | Người Phụ Trách | Bug
--- | --- | --- | ---
Cập nhật playbook với các hướng dẫn phản hồi sự cố lan truyền | mitigate | jennifer | n/a **DONE**
Sử dụng bộ tụ thông lượng (flux capacitor) để cân bằng tải giữa các cụm | prevent | martym | Bug 5554823 **TODO**
Lập lịch kiểm thử sự cố lan truyền trong đợt DiRT tiếp theo | process | docbrown | n/a **TODO**
Khảo sát việc chạy index MR/fusion liên tục | prevent | jennifer | Bug 5554824 **TODO**
Bịt lỗ rò file descriptor trong hệ thống phụ xếp hạng tìm kiếm | prevent | agoogler | Bug 5554825 **DONE**
Thêm khả năng loại bỏ tải (load shedding) cho tìm kiếm Shakespeare | prevent | agoogler | Bug 5554826 **TODO**
Xây dựng các kiểm thử hồi quy (regression tests) để đảm bảo các server phản hồi hợp lý với các truy vấn gây chết (queries of death) | prevent | clarac | Bug 5554827 **TODO**
Triển khai hệ thống phụ xếp hạng tìm kiếm cập nhật đến prod | prevent | jennifer | n/a **DONE**
Đóng băng sản xuất cho đến 2015-11-20 do ngân sách lỗi (error budget) bị cạn kiệt, hoặc tìm kiếm ngoại lệ do các tình huống dị thường, khó tin, kỳ quặc và chưa từng có tiền lệ | other | docbrown | n/a **TODO**

## Bài học kinh nghiệm (Lessons Learned)

**Những điều làm tốt**

-   Giám sát (Monitoring) đã cảnh báo chúng tôi nhanh chóng về tỷ lệ HTTP 500 cao (đạt tới ~100%)
-   Phân phối tập hợp Shakespeare cập nhật nhanh chóng đến tất cả các cụm

**Những điều làm sai**

-   Chúng tôi đã mất tay trong việc phản hồi sự cố lan truyền
-   Chúng tôi đã vượt quá ngân sách lỗi độ khả dụng (của mình) (nhiều bậc về số mũ) do sự tăng vọt giao thông phi thường mà hầu như tất cả đều dẫn đến thất bại

**Những nơi chúng tôi may mắn<sup>[4](#fn4)</sup>**

-   Danh sách thư của các người hâm mộ Shakespeare có một bản sao của sonnet mới có sẵn
-   Log của server có các vết stack (stack traces) chỉ đến việc cạn kiệt file descriptor là nguyên nhân gây crash
-   Truy vấn gây chết (query-of-death) được giải quyết bằng cách đẩy chỉ mục mới chứa từ ngữ tìm kiếm phổ biến

## Trình tự thời gian (Timeline)<sup>[5](#fn5)</sup>

2015-10-21 *(mọi thời gian tính theo UTC)*

-   14:51 Tin tức đưa tin rằng một sonnet Shakespeare mới đã được phát hiện trong ngăn đựng găng tay của một chiếc xe Delorean
-   14:53 Giao thông đến tìm kiếm Shakespeare tăng 88 lần sau khi bài đăng trên */r/shakespeare* chỉ đến động cơ tìm kiếm Shakespeare là nơi để tìm sonnet mới (ngoại trừ việc chúng tôi chưa có sonnet)
-   14:54 **SỰ CỐ BẮT ĐẦU** — Các backend tìm kiếm bắt đầu tan rã (melting down) dưới tải
-   14:55 docbrown nhận được bão trang (pager storm), `ManyHttp500s` từ tất cả các cụm
-   14:57 Toàn bộ giao thông đến tìm kiếm Shakespeare đang thất bại: xem *https://monitor*
-   14:58 docbrown bắt đầu điều tra, phát hiện tỷ lệ crash backend rất cao
-   15:01 **SỰ CỐ BẮT ĐẦU** docbrown tuyên bố sự cố #465 do sự cố lan truyền, phối hợp trên `#shakespeare`, chỉ định jennifer làm incident commander
-   15:02 một người tình cờ gửi email đến *shakespeare-discuss@* về việc phát hiện sonnet, mà tình cờ lại nằm ở đầu hộp thư của martym
-   15:03 jennifer thông báo cho danh sách *shakespeare-incidents@* về sự cố
-   15:04 martym tìm được văn bản của sonnet mới và tìm tài liệu về việc cập nhật tập hợp
-   15:06 docbrown phát hiện các triệu chứng crash giống hệt nhau ở tất cả các task trong tất cả các cụm, đang điều tra nguyên nhân dựa trên ứng dụng log
-   15:07 martym tìm thấy tài liệu, bắt đầu công việc chuẩn bị cho việc cập nhật tập hợp
-   15:10 martym thêm sonnet vào các tác phẩm đã biết của Shakespeare, bắt đầu công việc lập chỉ mục
-   15:12 docbrown liên lạc với clarac & agoogler (từ nhóm phát triển Shakespeare) để giúp xem xét codebase tìm các nguyên nhân có thể
-   15:18 clarac tìm thấy bằng chứng (smoking gun) trong log chỉ đến việc cạn kiệt file descriptor, xác nhận đối chiếu với code rằng rò rỉ tồn tại nếu tìm kiếm một từ ngữ không có trong tập hợp
-   15:20 công việc lập chỉ mục MapReduce của martym hoàn tất
-   15:21 jennifer và docbrown quyết định tăng đủ số lượng instance để giảm tải trên các instance mà họ có thể làm được công việc đáng kể trước khi chết và được khởi động lại
-   15:23 docbrown cân bằng tải toàn bộ giao thông đến cụm USA-2, cho phép tăng số lượng instance ở các cụm khác mà không có server nào chết ngay lập tức
-   15:25 martym bắt đầu sao chép chỉ mục mới đến tất cả các cụm
-   15:28 docbrown bắt đầu tăng số lượng instance 2 lần
-   15:32 jennifer thay đổi cân bằng tải để tăng giao thông đến các cụm không hy sinh
-   15:33 các task trong các cụm không hy sinh bắt đầu thất bại, cùng triệu chứng như trước
-   15:34 phát hiện sai lệch một bậc về số mũ trong các tính toán trên bảng trắng (whiteboard) cho việc tăng số lượng instance
-   15:36 jennifer hoàn tác cân bằng tải để hy sinh lại cụm USA-2 chuẩn bị cho việc tăng số lượng instance 5 lần toàn cầu bổ sung (lên tổng cộng 10 lần năng lực ban đầu)
-   15:36 **SỰ CỐ ĐƯỢC GIẢM NHẸ**, chỉ mục cập nhật đã được sao chép đến tất cả các cụm
-   15:39 docbrown bắt đầu đợt tăng số lượng instance thứ hai lên 10 lần năng lực ban đầu
-   15:41 jennifer khôi phục cân bằng tải trên tất cả các cụm cho 1% giao thông
-   15:43 tỷ lệ HTTP 500 của các cụm không hy sinh ở mức bình thường, các task thất bại ngắt quãng ở mức thấp
-   15:45 jennifer cân bằng 10% giao thông trên các cụm không hy sinh
-   15:47 tỷ lệ HTTP 500 của các cụm không hy sinh vẫn nằm trong SLO, không quan sát được task nào thất bại
-   15:50 30% giao thông được cân bằng trên các cụm không hy sinh
-   15:55 50% giao thông được cân bằng trên các cụm không hy sinh
-   16:00 **SỰ CỐ KẾT THÚC**, toàn bộ giao thông được cân bằng trên tất cả các cụm
-   16:30 **SỰ CỐ KẾT THÚC**, đạt tiêu chí thoát là 30 phút hiệu suất bình thường

## Thông tin hỗ trợ (Supporting information):<sup>[6](#fn6)</sup>

-   Bảng điều khiển giám sát (Monitoring dashboard),
    
    https://monitor/shakespeare?end\_time=20151021T160000  
    &duration=7200

<a id="fn1"></a>[1](#fn1) Tác động là hiệu quả đối với người dùng, doanh thu, v.v.

<a id="fn2"></a>[2](#fn2) Một lời giải thích về các tình huống mà sự cố này đã xảy ra. Thường hữu ích khi sử dụng một kỹ thuật như 5 Whys [[Ohn88]](https://sre.google/sre-book/bibliography#Ohn88) để hiểu các yếu tố góp phần.

<a id="fn3"></a>[3](#fn3) Các AI "phản xạ co giật" (knee-jerk) thường quá cực đoan hoặc tốn kém để triển khai, và cần đến sự phán xét để định phạm vi lại chúng trong một ngữ cảnh rộng hơn. Có rủi ro tối ưu quá mức cho một vấn đề cụ thể, chẳng hạn như thêm giám sát/cảnh báo đặc thù trong khi các cơ chế tin cậy như unit tests có thể phát hiện vấn đề sớm hơn nhiều trong quy trình phát triển.

<a id="fn4"></a>[4](#fn4) Phần này thực sự dành cho các lần suýt mất (near misses), ví dụ, "Bộ teleported dê đã có sẵn để sử dụng khẩn cấp với các loài động vật khác bất chấp việc thiếu chứng nhận."

<a id="fn5"></a>[5](#fn5) Một "kịch bản" (screenplay) của sự cố; sử dụng trình tự thời gian sự cố từ tài liệu Quản lý Sự Cố để bắt đầu điền vào trình tự thời gian của postmortem, sau đó bổ sung bằng các mục nhập liên quan khác.

<a id="fn6"></a>[6](#fn6) Thông tin hữu ích, các liên kết, log, ảnh chụp màn hình, biểu đồ, log IRC, log IM, v.v.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
