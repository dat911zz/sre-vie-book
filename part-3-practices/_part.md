# Phần III. Thực hành (Practices)

> **Nguyên bản:** [Part III - Practices](https://sre.google/sre-book/part-III-practices/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Đội biên tập Softdreams RnD)

---

Nói ngắn gọn, SRE (Site Reliability Engineering — Kỹ thuật Độ tin cậy Trang web) vận hành các dịch vụ — một tập hợp các hệ thống liên quan, được vận hành cho người dùng, dù là nội bộ hay bên ngoài — và chịu trách nhiệm cuối cùng về sức khỏe của các dịch vụ này. Vận hành [thành công một dịch vụ](https://sre.google/sre-book/introduction/) liên quan đến một loạt hoạt động: phát triển các hệ thống giám sát (monitoring), lập kế hoạch năng lực (capacity), phản ứng với các incident (sự cố), đảm bảo các nguyên nhân gốc rễ của các outage (mất dịch vụ) được giải quyết, v.v. Phần này đề cập đến lý thuyết và thực hành về hoạt động hàng ngày của một SRE: xây dựng và vận hành các [hệ thống tính toán phân tán](https://sre.google/sre-book/managing-critical-state/) lớn.

Chúng tôi có thể đặc trưng hóa sức khỏe của một dịch vụ — theo cách rất giống cách Abraham Maslow phân loại các nhu cầu của con người [[Mas43]](https://sre.google/sre-book/bibliography#Mas43) — từ các yêu cầu cơ bản nhất để một hệ thống hoạt động như một dịch vụ, cho đến các cấp chức năng cao hơn — cho phép tự hiện thực hóa (self-actualization) và chủ động định hình hướng đi của dịch vụ thay vì phản ứng kiểu chữa cháy. Cách hiểu này nền tảng đến mức đối với việc chúng tôi đánh giá các dịch vụ tại Google, nên nó không được trình bày rõ ràng cho đến khi một số Google SRE — trong đó có đồng nghiệp cũ của chúng tôi là Mikey Dickerson,<sup>[1](#fn1)</sup> — tạm thời rời sang nền văn hóa hoàn toàn khác của chính phủ Hoa Kỳ để hỗ trợ ra mắt *healthcare.gov* vào cuối năm 2013 và đầu năm 2014: họ cần một cách để giải thích cách tăng độ tin cậy của các hệ thống.

Chúng tôi sẽ dùng hệ phân cấp này, được minh họa trong [Hình III-1](#hinh-iii-1), để xem xét các yếu tố góp phần làm cho một dịch vụ đáng tin cậy, từ cơ bản nhất đến tinh vi nhất.


<a id="hinh-iii-1"></a>![Hình III-1](../assets/imgs/fig-III-1.jpg)

[Hình III-1.](#hinh-iii-1) Hệ phân cấp Độ tin cậy Dịch vụ (Service Reliability Hierarchy).

#### Giám sát (Monitoring)

Không có giám sát, bạn không có cách nào để biết liệu dịch vụ thậm chí có đang hoạt động hay không; thiếu một hạ tầng giám sát được thiết kế có suy nghĩ, bạn đang bay mù. Có lẽ tất cả những người cố dùng website đều nhận một lỗi, có lẽ không — nhưng bạn muốn nhận ra vấn đề trước khi người dùng của mình nhận ra. Chúng tôi bàn về các công cụ và triết lý trong [Practical Alerting from Time-Series Data](10-practical-alerting.md).

#### Phản ứng Sự cố (Incident Response)

Các SRE không đi on-call (trực sự cố) chỉ vì bản thân việc đó: thay vào đó, hỗ trợ on-call là một công cụ chúng tôi dùng để đạt được sứ mệnh lớn hơn và giữ cho mình liên lạc với cách các hệ thống tính toán phân tán thực sự vận hành (và thất bại!). Nếu có thể tìm cách tự giải thoát khỏi việc đeo pager (thiết bị gọi trực), chúng tôi sẽ làm vậy. Trong [Being On-Call](11-being-on-call.md), chúng tôi giải thích cách cân bằng các nhiệm vụ on-call với các trách nhiệm khác.

Một khi nhận ra có vấn đề, bạn làm cho nó biến mất như thế nào? Điều đó không nhất thiết có nghĩa là sửa nó một lần và mãi mãi — có lẽ bạn có thể cầm máu bằng cách giảm độ chính xác của hệ thống hoặc tạm thời tắt một số tính năng, cho phép nó suy giảm êm ả (gracefully degrade), hoặc có lẽ bạn có thể hướng traffic (lưu lượng) đến một instance (hồi) khác của dịch vụ đang chạy bình thường. Chi tiết của giải pháp bạn chọn để triển khai nhất thiết phụ thuộc vào dịch vụ và tổ chức của bạn. Tuy nhiên, việc phản ứng hiệu quả với các incident là điều áp dụng được cho mọi đội.

Tìm ra điều gì sai là bước đầu tiên; chúng tôi cung cấp một cách tiếp cận có cấu trúc trong [Effective Troubleshooting](12-effective-troubleshooting.md).

Trong một incident, thường có cám dỗ nhượng bộ adrenaline và bắt đầu phản ứng kiểu ad hoc (tùy hứng). Chúng tôi khuyên chống lại cám dỗ này trong [Emergency Response](13-emergency-response.md), và trong [Managing Incidents](14-managing-incidents.md) chúng tôi khuyên rằng việc quản lý các incident hiệu quả nên giảm tác động của chúng và hạn chế lo âu do outage gây ra.

#### Postmortem và Phân tích Nguyên nhân Gốc rễ (Postmortem and Root-Cause Analysis)

Chúng tôi nhắm đến việc chỉ được cảnh báo về và giải quyết thủ công các vấn đề mới và thú vị mà dịch vụ của chúng tôi đặt ra; "sửa" đi sửa lại cùng một vấn đề là buồn tẻ đến thảm hại. Thực tế, tư duy này là một trong những yếu tố phân biệt chính giữa triết lý SRE và một số môi trường thiên về vận hành truyền thống hơn. Chủ đề này được khám phá trong hai chương.

Xây dựng một văn hóa postmortem (báo cáo sau sự cố) không đổ lỗi (blameless) là bước đầu tiên để hiểu điều gì đã sai (và điều gì đã đúng!), như mô tả trong [Postmortem Culture: Learning from Failure](15-postmortem-culture.md).

Liên quan đến thảo luận đó, trong [Tracking Outages](16-tracking-outages.md), chúng tôi mô tả ngắn gọn một công cụ nội bộ, outage tracker (trình theo dõi outage), cho phép các đội SRE theo dõi các incident production gần đây, nguyên nhân của chúng và các hành động được thực hiện để ứng phó.

#### Kiểm thử (Testing)

Một khi hiểu điều gì hay sai, bước tiếp theo của chúng tôi là cố gắng ngăn chặn nó, vì một ounce (28g) phòng bệnh đáng một pound (454g) chữa bệnh. Các bộ kiểm thử (test suites) mang lại một mức độ đảm bảo rằng phần mềm của chúng tôi không mắc các lớp lỗi nhất định trước khi phát hành lên production; chúng tôi bàn về cách dùng các bộ này hiệu quả nhất trong [Testing for Reliability](17-testing-reliability.md).

#### Lập kế hoạch Năng lực (Capacity Planning)

Trong [Software Engineering in SRE](18-software-engineering-in-sre.md), chúng tôi cung cấp một nghiên cứu tình huống về kỹ thuật phần mềm trong SRE với [Auxon](https://sre.google/sre-book/automation-at-google/), một công cụ để tự động hóa lập kế hoạch năng lực.

Tiếp theo một cách tự nhiên của việc lập kế hoạch năng lực, load balancing (cân bằng tải) đảm bảo rằng chúng tôi đang sử dụng đúng đắn năng lực đã xây dựng. Chúng tôi bàn về cách các yêu cầu đến các dịch vụ được định tuyến đến các datacenter (trung tâm dữ liệu) trong [Load Balancing at the Frontend](19-load-balancing-frontend.md). Sau đó chúng tôi tiếp tục trong [Load Balancing in the Datacenter](20-load-balancing-datacenter.md) và [Handling Overload](21-handling-overload.md), cả hai đều thiết yếu để đảm bảo độ tin cậy dịch vụ.

Cuối cùng, trong [Addressing Cascading Failures](22-addressing-cascading-failures.md), chúng tôi đưa ra lời khuyên để giải quyết các sự cố lan truyền (cascading failures), cả trong thiết kế hệ thống lẫn trong trường hợp dịch vụ của bạn bị cuốn vào một sự cố lan truyền.

#### Phát triển (Development)

Một trong những khía cạnh chính của cách tiếp cận Site Reliability Engineering của Google là chúng tôi thực hiện một lượng đáng kể công việc thiết kế hệ thống và kỹ thuật phần mềm quy mô lớn ngay trong tổ chức.

Trong [Managing Critical State: Distributed Consensus for Reliability](23-managing-critical-state.md), chúng tôi giải thích sự nhất trí phân tán (distributed consensus), mà (dưới dạng Paxos) nằm ở cốt lõi của nhiều hệ thống phân tán của Google, bao gồm hệ thống Cron phân tán toàn cầu của chúng tôi. Trong [Distributed Periodic Scheduling with Cron](24-distributed-periodic-scheduling.md), chúng tôi phác thảo một hệ thống mở rộng (scale) ra toàn bộ các datacenter và xa hơn — một nhiệm vụ không dễ dàng.

[Data Processing Pipelines](25-data-processing-pipelines.md) thảo luận về các hình thức khác nhau mà các đường ống xử lý dữ liệu (data processing pipelines) có thể mang: từ các job MapReduce chạy một lần định kỳ đến các hệ thống vận hành gần như thời gian thực. Các kiến trúc khác nhau có thể dẫn đến những thách thức bất ngờ và phản trực giác.

Đảm bảo rằng dữ liệu bạn lưu trữ vẫn còn đó khi bạn muốn đọc nó là cốt lõi của tính toàn vẹn dữ liệu (data integrity); trong [Data Integrity: What You Read Is What You Wrote](26-data-integrity.md), chúng tôi giải thích cách giữ dữ liệu an toàn.

#### Sản phẩm (Product)

Cuối cùng, sau khi đã vượt lên đỉnh kim tự tháp độ tin cậy, chúng tôi tìm thấy mình ở vị trí có một sản phẩm khả dụng. Trong [Reliable Product Launches at Scale](27-reliable-product-launches.md), chúng tôi viết về cách Google thực hiện các lần ra mắt sản phẩm đáng tin cậy ở quy mô lớn, để mang lại cho người dùng trải nghiệm tốt nhất có thể ngay từ Ngày Zero (Day Zero).

## Đọc thêm từ Google SRE (Further Reading from Google SRE)

Như đã bàn trước đó, kiểm thử là tinh tế, và làm sai nó có thể gây ảnh hưởng lớn đến sự ổn định tổng thể. Trong một bài viết ACM [[Kri12]](https://sre.google/sre-book/bibliography#Kri12), chúng tôi giải thích cách Google thực hiện kiểm thử chống chịu *toàn công ty* (company-wide resilience testing) để đảm bảo khả năng vượt qua những điều bất ngờ nếu một cuộc tận thế zombie (zombie apocalypse) hay thảm họa khác ập đến.

Mặc dù thường được coi là một nghệ thuật tối tăm, đầy những bảng tính (spreadsheets) huyền bí bói cho tương lai, việc lập kế hoạch năng lực vẫn là thiết yếu; và như [[Hix15a]](https://sre.google/sre-book/bibliography#Hix15a) cho thấy, bạn thực sự không *cần* một quả cầu pha lê để làm điều đó đúng.

Cuối cùng, một cách tiếp cận mới mẻ và thú vị với bảo mật mạng doanh nghiệp được trình bày trong [[War14]](https://sre.google/sre-book/bibliography#War14), một sáng kiến thay thế các intranet (mạng nội bộ) đặc quyền bằng các chứng chỉ thiết bị và người dùng. Được thúc đẩy bởi các SRE ở cấp độ hạ tầng, đây chắc chắn là một cách tiếp cận đáng ghi nhớ khi bạn xây mạng tiếp theo.

<a id="fn1"></a>[1](#fn1) Mikey rời Google vào mùa hè năm 2014 để trở thành quản trị viên đầu tiên của US Digital Service ([*https://www.usds.gov/*](https://www.usds.gov/)), một cơ quan có dự định (một phần) mang các nguyên lý và thực hành SRE đến các hệ thống IT của chính phủ Hoa Kỳ.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
