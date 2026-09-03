# Phần IV. Quản lý (Part IV - Management)

> **Nguyên bản:** [Part IV - Management](https://sre.google/sre-book/part-IV-management/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

Bộ chủ đề cuối cùng của chúng tôi bao gồm việc làm việc cùng nhau trong một đội, và việc làm việc như những đội. Không có SRE (Site Reliability Engineer — Kỹ sư Độ tin cậy Trang web) nào là một hòn đảo, và có một số cách riêng biệt mà chúng tôi làm việc.

Bất kỳ tổ chức nào nghiêm túc vận hành một nhóm SRE hiệu quả đều phải nghĩ đến đào tạo. Dạy các SRE cách tư duy trong một môi trường phức tạp và thay đổi nhanh — bằng một [chương trình đào tạo được thiết kế tốt](https://sre.google/resources/practices-and-processes/training-site-reliability-engineers/) — có thể gieo các thực hành tốt nhất trong vài tuần đến vài tháng đầu của nhân viên mới, điều mà nếu không sẽ mất nhiều tháng hoặc nhiều năm để tích lũy. Chúng tôi thảo luận các chiến lược để làm điều đó trong [Tăng tốc SRE đến On-Call và Hơn thế nữa](https://sre.google/sre-book/accelerating-sre-on-call/).

Bất kỳ ai trong thế giới vận hành cũng biết: phụ trách một dịch vụ lớn đi kèm rất nhiều sự gián đoạn (interrupt) — production rơi vào trạng thái xấu, mọi người đòi cập nhật binary yêu thích của họ, một hàng dài yêu cầu tham vấn… Quản lý các interrupt trong điều kiện bất ổn là một kỹ năng cần thiết, như chúng tôi sẽ thảo luận trong [Đối phó với Interrupt](https://sre.google/sre-book/dealing-with-interrupts/).

Nếu điều kiện bất ổn đã kéo dài đủ lâu, một đội SRE cần phải bắt đầu phục hồi từ tình trạng quá tải vận hành (operational overload). Chúng tôi có đúng kế hoạch bay cho bạn trong [Đặt một SRE vào để Phục hồi từ Quá tải Vận hành](https://sre.google/sre-book/operational-overload/).

Chúng tôi viết trong [Giao tiếp và Hợp tác trong SRE](https://sre.google/sre-book/communication-and-collaboration/) về các vai trò khác nhau bên trong SRE; giao tiếp giữa các đội, giữa các site (cơ sở) và giữa các châu lục; điều hành các cuộc họp production; và các nghiên cứu tình huống về cách SRE đã hợp tác tốt.

Cuối cùng, [Mô hình Tham gia SRE đang Tiến hóa](https://sre.google/sre-book/evolving-sre-engagement-model/) xem xét một viên đá góc của hoạt động SRE: production readiness review (PRR — đánh giá sẵn sàng production), một bước quan trọng trong việc onboarding (đưa vào) một dịch vụ mới. Chúng tôi thảo luận cách tiến hành các PRR, và cách đi xa hơn mô hình thành công nhưng cũng có giới hạn này.

## Đọc thêm từ Google SRE (Further Reading from Google SRE)

Xây dựng các hệ thống đáng tin cậy đòi hỏi một sự pha trộn các kỹ năng được hiệu chuẩn cẩn thận, trải dài từ phát triển phần mềm đến các ngành phân tích và kỹ thuật hệ thống có lẽ ít được biết đến hơn. Chúng tôi viết về các ngành sau trong "The Systems Engineering Side of Site Reliability Engineering" (Khía cạnh Kỹ thuật Hệ thống của SRE) [[Hix15b]](https://sre.google/sre-book/bibliography#Hix15b).

Thuê các SRE giỏi là then chốt để có một tổ chức độ tin cậy hoạt động hiệu quả, như được khám phá trong "Hiring Site Reliability Engineers" (Thuê các Kỹ sư Độ tin cậy Trang web) [[Jon15]](https://sre.google/sre-book/bibliography#Jon15). Các thực hành tuyển dụng của Google đã được chi tiết hóa trong các tác phẩm như *Work Rules!* [[Boc15]](https://sre.google/sre-book/bibliography#Boc15),<sup>[1](#fn1)</sup> nhưng việc thuê các SRE có một bộ các đặc thù riêng. Ngay cả theo các tiêu chuẩn tổng thể của Google, các ứng viên SRE cũng khó tìm và thậm chí còn khó phỏng vấn hiệu quả hơn.

<a id="fn1"></a>[1](#fn1) Được viết bởi Laszlo Bock, Phó chủ tịch cấp cao (Senior VP) của Google phụ trách Vận hành Nhân sự (People Operations).

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
