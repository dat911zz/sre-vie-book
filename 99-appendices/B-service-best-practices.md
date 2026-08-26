> **Nguyên bản:** [Appendix B. A Collection of Best Practices for Production Services](https://sre.google/sre-book/service-best-practices/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Đội biên tập Softdreams RnD)

---

## Phụ lục B. Một Tập Hợp Các Thực Hành Tối Ưu Cho Dịch Vụ Sản Xuất (Appendix B. A Collection of Best Practices for Production Services)

Tác giả: Ben Treynor Sloss  
Biên tập: Betsy Beyer

## Thua một cách hợp lý (Fail Sanely)

Làm sạch và xác thực các đầu vào cấu hình, và phản hồi với các đầu vào phi lý bằng cách *vừa* tiếp tục vận hành ở trạng thái trước *vừa* cảnh báo việc nhận được đầu vào xấu. Đầu vào xấu thường rơi vào một trong những loại sau:

Dữ liệu sai

Xác thực cả cú pháp và, nếu có thể, ngữ nghĩa. Cảnh giác với dữ liệu trống và dữ liệu một phần hoặc bị cắt (ví dụ, cảnh báo nếu cấu hình nhỏ hơn bản trước *N*%).

Dữ liệu bị trì hoãn

Điều này có thể làm cho dữ liệu hiện tại trở nên không hợp lệ do hết thời gian chờ (timeout). Cảnh báo khá sớm trước khi dữ liệu được dự đoán sẽ hết hạn.

Thua theo cách giữ được chức năng, có thể phải đánh đổi bằng việc quá rộng rãi hoặc quá giản lược. Chúng tôi nhận thấy rằng nói chung an toàn hơn khi hệ thống tiếp tục hoạt động với cấu hình trước đó và chờ sự phê duyệt của con người trước khi sử dụng dữ liệu mới, có thể không hợp lệ.

### Ví dụ

Năm 2005, hệ thống cân bằng tải và cân bằng độ trễ DNS toàn cầu của Google đã nhận được một tệp mục nhập DNS trống do vấn đề quyền truy cập tệp. Hệ thống chấp nhận tệp trống này và trả về `NXDOMAIN` trong sáu phút cho tất cả các thuộc tính của Google. Để đáp ứng, hệ thống hiện thực hiện một số kiểm tra tính hợp lý (sanity checks) trên các cấu hình mới, bao gồm xác nhận sự hiện diện của các IP ảo (virtual IPs) cho *google.com*, và sẽ tiếp tục phục vụ các mục nhập DNS trước đó cho đến khi nhận được một tệp mới vượt qua các kiểm tra đầu vào của nó.

Năm 2009, dữ liệu sai (nhưng hợp lệ) đã dẫn đến việc Google đánh dấu toàn bộ Web là chứa phần mềm độc hại [[May09]](https://sre.google/sre-book/bibliography#May09). Một tệp cấu hình chứa danh sách các URL đáng ngờ đã bị thay thế bằng một ký tự dấu gạch chéo duy nhất (`/`), ký tự này khớp với mọi URL. Các kiểm tra về sự thay đổi đáng kể kích thước tệp và kiểm tra xem cấu hình có đang khớp với các trang web được tin là ít khả năng chứa phần mềm độc hại hay không đã có thể ngăn điều này đi vào môi trường sản xuất.

## Triển khai dần (Progressive Rollouts)

Các đợt triển khai không khẩn cấp *phải* diễn ra theo từng giai đoạn. Cả thay đổi cấu hình lẫn thay đổi binary đều tạo ra rủi ro, và bạn giảm thiểu rủi ro này bằng cách áp dụng thay đổi lên một tỷ lệ nhỏ giao thông (traffic) và năng lực (capacity) một lúc. Kích thước dịch vụ hoặc đợt triển khai của bạn, cũng như hồ sơ rủi ro, sẽ quyết định tỷ lệ phần trăm năng lực sản xuất mà đợt triển khai được đẩy tới và khoảng thời gian phù hợp giữa các giai đoạn. Cũng là một ý tưởng hay khi thực hiện các giai đoạn khác nhau ở các địa lý khác nhau, để phát hiện các vấn đề liên quan đến chu kỳ giao thông theo ngày và sự khác biệt về tỷ lệ giao thông theo địa lý.

Các đợt triển khai nên được giám sát. Để đảm bảo không có điều gì bất ngờ xảy ra trong quá trình triển khai, nó phải được theo dõi bởi kỹ sư đang thực hiện giai đoạn triển khai đó hoặc — tốt hơn — bởi một hệ thống giám sát đã được chứng minh là tin cậy. Nếu phát hiện hành vi bất ngờ, hãy hoàn tác (roll back) trước và chẩn đoán sau để giảm thiểu Mean Time to Recovery (thời gian phục hồi trung bình).

## Định nghĩa SLO như một người dùng (Define SLOs Like a User)

Đo lường độ khả dụng và hiệu suất bằng những thuật ngữ quan trọng đối với người dùng cuối. Xem [Mục tiêu Mức độ Dịch vụ (Service Level Objectives)](https://sre.google/sre-book/service-level-objectives/) để thảo luận thêm.

### Ví dụ

Việc đo lường tỷ lệ lỗi và độ trễ tại client Gmail, thay vì tại server, đã dẫn đến sự giảm đáng kể trong đánh giá của chúng tôi về độ khả dụng của Gmail và thúc đẩy những thay đổi đối với cả mã nguồn client lẫn server của Gmail. Kết quả là Gmail đã tăng từ khoảng 99.0% khả dụng lên hơn 99.9% khả dụng trong vòng vài năm.

## Ngân sách lỗi (Error Budgets)

Cân bằng giữa độ tin cậy (reliability) và tốc độ đổi mới bằng ngân sách lỗi (error budget) (xem [Động lực cho Ngân sách Lỗi (Motivation for Error Budgets)](https://sre.google/sre-book/embracing-risk#xref_risk-management_unreliability-budgets)), định nghĩa mức độ thất bại được chấp nhận cho một dịch vụ trong một khoảng thời gian nào đó; chúng tôi thường sử dụng một tháng. Một ngân sách đơn giản là 1 trừ đi SLO của dịch vụ; ví dụ, một dịch vụ có mục tiêu độ khả dụng 99.99% có một "ngân sách" 0.01% cho sự không khả dụng. Chừng nào dịch vụ chưa dùng hết ngân sách lỗi cho tháng đó qua tỷ lệ lỗi nền (background rate of errors) cộng thêm mọi thời gian ngừng hoạt động, thì nhóm phát triển được tự do (trong chừng mực) ra mắt các tính năng mới, bản cập nhật và tương tự.

Nếu ngân sách lỗi bị dùng hết, dịch vụ sẽ đóng băng (freeze) các thay đổi (ngoại trừ các bản sửa bảo mật khẩn cấp và các bản vá lỗi giải quyết bất kỳ nguyên nhân nào của sự gia tăng lỗi) cho đến khi dịch vụ lấy lại được chỗ trống trong ngân sách, hoặc tháng được đặt lại. Đối với các dịch vụ trưởng thành có SLO lớn hơn 99.99%, việc đặt lại ngân sách theo quý thay vì theo tháng là phù hợp, vì lượng thời gian ngừng hoạt động được phép là nhỏ.

Ngân sách lỗi loại bỏ căng thẳng cấu trúc có thể phát sinh giữa các nhóm SRE và nhóm phát triển sản phẩm bằng cách cung cấp cho họ một cơ chế chung, dựa trên dữ liệu, để đánh giá rủi ro ra mắt. Chúng cũng cung cấp cho cả nhóm SRE và nhóm phát triển sản phẩm một mục tiêu chung là phát triển các thực hành và công nghệ cho phép đổi mới nhanh hơn và ra mắt nhiều hơn mà không "vượt ngân sách" (blowing the budget).

## Giám sát (Monitoring)

Giám sát có thể chỉ có ba loại đầu ra:

#### Cảnh báo gọi người (Pages)

Một con người phải làm gì đó *ngay bây giờ*

#### Vé (Tickets)

Một con người phải làm gì đó trong vòng vài ngày

#### Ghi log (Logging)

Không ai cần xem đầu ra này ngay lập tức, nhưng nó có sẵn để phân tích sau này nếu cần

Nếu điều gì đủ quan trọng để quấy rầy một con người, thì nó nên *yêu cầu* hành động ngay lập tức (tức là page) hoặc được xử lý như một bug và đưa vào hệ thống theo dõi bug của bạn. Việc đưa các cảnh báo vào email và hy vọng rằng ai đó sẽ đọc tất cả chúng và nhận ra những cái quan trọng tương đương về mặt đạo đức với việc pipe chúng vào */dev/null*: chúng sẽ sớm bị bỏ qua. Lịch sử cho thấy chiến lược này là một sự phiền toái hấp dẫn (attractive nuisance) vì nó có thể hoạt động trong một thời gian, nhưng nó dựa vào sự cảnh giác vĩnh viễn của con người, và sự cố tất yếu xảy ra sẽ vì thế mà nghiêm trọng hơn khi nó đến.

## Postmortem (Bản đánh giá sự cố)

Postmortem (xem [Văn hóa Postmortem: Học hỏi từ Thất bại (Postmortem Culture: Learning from Failure)](https://sre.google/sre-book/postmortem-culture/)) nên vô tội (blameless) và tập trung vào quy trình và công nghệ, chứ không phải con người. Giả định rằng những người tham gia một sự cố là thông minh, có thiện chí, và đã đưa ra những lựa chọn tốt nhất mà họ có thể dựa trên thông tin họ có sẵn tại thời điểm đó. Điều đó có nghĩa là chúng tôi không thể "sửa" con người, mà phải thay vào đó sửa môi trường của họ: ví dụ, cải thiện thiết kế hệ thống để tránh cả một nhóm các vấn đề, làm cho thông tin phù hợp dễ dàng truy cập được, và tự động xác định các quyết định vận hành để khiến cho việc đặt các hệ thống vào trạng thái nguy hiểm trở nên khó khăn.

## Quy hoạch năng lực (Capacity Planning)

Cấp phát (provision) để xử lý đồng thời một sự cố có kế hoạch và một sự cố không có kế hoạch, mà không khiến trải nghiệm người dùng trở nên không chấp nhận được; điều này dẫn đến cấu hình "*N* + 2", trong đó giao thông đỉnh (peak traffic) có thể được xử lý bởi *N* instance (có thể ở chế độ suy giảm) trong khi 2 instance lớn nhất không khả dụng:

-   Xác thực các dự báo nhu cầu trước đó so với thực tế cho đến khi chúng khớp nhất quán. Sự lệch lạc ngụ ý dự báo không ổn định, cấp phát không hiệu quả và rủi ro thiếu hụt năng lực.
-   Sử dụng kiểm thử tải (load testing) thay vì truyền thống để thiết lập tỷ lệ nguồn lực so với năng lực: một cụm *X* máy có thể xử lý *Y* truy vấn mỗi giây cách đây ba tháng, nhưng nó có thể vẫn làm được điều đó không khi có các thay đổi đối với hệ thống?
-   Đừng nhầm lẫn tải ngày đầu tiên với tải trạng thái ổn định (steady-state load). Các lần ra mắt thường thu hút nhiều giao thông hơn, trong khi đó cũng là thời điểm bạn đặc biệt muốn đưa ra mặt tốt nhất của sản phẩm. Xem [Ra Mắt Sản Phẩm Tin Cậy Quy Mô Lớn (Reliable Product Launches at Scale)](https://sre.google/sre-book/reliable-product-launches/) và [Danh Mục Kiểm Tra Phối Hợp Ra Mắt (Launch Coordination Checklist)](https://sre.google/sre-book/launch-checklist/).

## Quá tải và Thất bại (Overloads and Failure)

Các dịch vụ nên tạo ra các kết quả hợp lý nhưng không tối ưu nếu bị quá tải. Ví dụ, Google Search sẽ tìm kiếm một phần nhỏ hơn của chỉ mục và ngừng phục vụ các tính năng như Instant để tiếp tục cung cấp kết quả tìm kiếm web chất lượng tốt khi bị quá tải. SRE của Search kiểm thử các cụm tìm kiếm web vượt quá năng lực định mức của chúng để đảm bảo chúng hoạt động chấp nhận được khi bị quá tải giao thông.

Đối với những thời điểm mà tải đủ cao đến mức ngay cả các phản hồi suy giảm cũng quá đắt đỏ cho tất cả các truy vấn, hãy thực hành việc loại bỏ tải một cách tinh tế (graceful load shedding), sử dụng hàng đợi (queuing) hoạt động tốt và các thời gian chờ động (dynamic timeouts); xem [Xử lý Quá tải (Handling Overload)](https://sre.google/sre-book/handling-overload/). Các kỹ thuật khác bao gồm việc trả lời các yêu cầu sau một độ trễ đáng kể ("tarpitting") và chọn một tập con nhất quán các client để nhận lỗi, giữ cho phần còn lại một trải nghiệm người dùng tốt.

Việc thử lại (retries) có thể khuếch đại các tỷ lệ lỗi thấp thành mức giao thông cao hơn, dẫn đến các sự cố lan truyền (cascading failures) (xem [Xử lý Sự Cố Lan Truyền (Addressing Cascading Failures)](https://sre.google/sre-book/addressing-cascading-failures/)). Phản hồi với các sự cố lan truyền bằng cách loại bỏ một phần giao thông (bao gồm cả các lần thử lại!) ở phía thượng nguồn (upstream) của hệ thống một khi tổng tải vượt quá tổng năng lực.

Mọi client thực hiện một RPC đều phải triển khai backoff hàm mũ (exponential backoff) (có jitter) cho các lần thử lại, để làm giảm sự khuếch đại lỗi. Các client di động đặc biệt gây rắc rối vì có thể có hàng triệu chiếc và việc cập nhật mã nguồn của chúng để sửa hành vi mất một lượng thời gian đáng kể — có thể là vài tuần — và yêu cầu người dùng cài đặt bản cập nhật.

## Các Nhóm SRE (SRE Teams)

Các nhóm SRE nên dành không quá 50% thời gian cho công việc vận hành (xem [Loại bỏ Toil (Eliminating Toil)](https://sre.google/sre-book/eliminating-toil/)); phần vận hành tràn ra (operational overflow) nên được chuyển hướng đến nhóm phát triển sản phẩm. Nhiều dịch vụ cũng bao gồm các nhà phát triển sản phẩm trong vòng on-call và xử lý vé (ticket handling), ngay cả khi hiện tại không có phần tràn ra. Điều này tạo ra động lực để thiết kế các hệ thống giảm thiểu hoặc loại bỏ toil vận hành, cùng với việc đảm bảo rằng các nhà phát triển sản phẩm tiếp xúc với mặt vận hành của dịch vụ. Một cuộc họp sản xuất (production meeting) định kỳ giữa các SRE và nhóm phát triển (xem [Giao tiếp và Hợp tác trong SRE (Communication and Collaboration in SRE)](https://sre.google/sre-book/communication-and-collaboration/)) cũng hữu ích.

Chúng tôi nhận thấy rằng ít nhất tám người cần phải là một phần của đội on-call, để tránh mệt mỏi và cho phép bố trí nhân sự bền vững và tỷ lệ rời bỏ thấp. Tốt nhất, những người on-call nên ở hai địa điểm địa lý tách biệt tốt (ví dụ, California và Ireland) để cung cấp chất lượng cuộc sống tốt hơn bằng cách tránh các cuộc gọi vào ban đêm; trong trường hợp này, sáu người tại mỗi địa điểm là kích thước đội tối thiểu.

Dự kiến xử lý không quá hai sự kiện (events) mỗi ca on-call (ví dụ, mỗi 12 giờ): cần thời gian để phản hồi và sửa các sự cố, bắt đầu postmortem, và nộp các bug liên quan. Các sự kiện thường xuyên hơn có thể làm giảm chất lượng phản hồi, và cho thấy rằng có điều gì đó sai với (ít nhất một trong) thiết kế của hệ thống, độ nhạy của giám sát và phản hồi với các bug postmortem.

Trớ trêu thay, nếu bạn triển khai các thực hành tối ưu này, nhóm SRE cuối cùng có thể bị mất tay trong việc phản hồi các sự cố do chúng hiếm xảy ra, biến một sự cố ngắn thành một sự cố dài. Hãy tập luyện xử lý các sự cố giả định (xem [Đóng Vai Thảm Họa (Disaster Role Playing)](https://sre.google/sre-book/accelerating-sre-on-call#xref_training_disaster-rpg)) thường xuyên và cải thiện tài liệu xử lý sự cố của bạn trong quá trình đó.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
