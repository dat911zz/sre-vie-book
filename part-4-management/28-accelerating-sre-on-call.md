# Chương 28. Tăng tốc SRE đến On-Call và Hơn thế nữa (Accelerating SREs to On-Call and Beyond)

> **Nguyên bản:** [Chapter 28 - Accelerating SREs to On-Call and Beyond](https://sre.google/sre-book/accelerating-sre-on-call/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

**Làm thế nào tôi có thể gắn một chiếc jetpack (bộ siêu tăng tốc) cho những người mới của mình trong khi vẫn giúp các SRE cấp cao theo kịp?**

*Tác giả:* Andrew Widdowson
*Biên tập:* Shylaja Nukala

## Bạn Đã Thuê SRE Tiếp Theo, Vậy Tiếp Theo Là Gì? (You've Hired Your Next SRE(s), Now What?)

Bạn đã tuyển dụng nhân viên mới và họ bắt đầu với vai trò Site Reliability Engineer (SRE). Giờ đây, bạn phải đào tạo họ ngay tại chỗ. Nếu đầu tư ngay từ đầu vào giáo dục và định hướng kỹ thuật cho các SRE mới, bạn sẽ giúp họ trở thành những kỹ sư giỏi hơn. Loại đào tạo này giúp họ đạt thành thạo nhanh hơn, đồng thời làm cho bộ kỹ năng của họ vững vàng và cân bằng hơn.

Các đội SRE thành công được xây dựng trên nền tảng niềm tin. Để duy trì một dịch vụ nhất quán ở quy mô toàn cầu, bạn cần tin rằng đồng sự on-call của mình hiểu rõ hệ thống hoạt động ra sao,<sup>[1](#fn1)</sup> có khả năng chẩn đoán các hành vi bất thường, sẵn lòng nhờ sự giúp đỡ khi cần và có thể phản ứng hiệu quả dưới áp lực để cứu nguy. Do đó, việc xem xét giáo dục SRE qua lăng kính "một người mới cần học gì để đi on-call?" là thiết yếu nhưng chưa đủ. Với yêu cầu về niềm tin, bạn cũng cần đặt ra những câu hỏi như:

-   Các on-caller hiện có của tôi có thể đánh giá mức độ sẵn sàng đi on-call của người mới như thế nào?
-   Chúng tôi có thể khai thác sự nhiệt huyết và tò mò của nhân viên mới để giúp các SRE hiện tại hưởng lợi từ điều đó như thế nào?
-   Những hoạt động nào mà tôi có thể cam kết cho đội của mình, vừa mang lại lợi ích về mặt giáo dục cho mọi người, vừa được mọi người yêu thích?

Học viên có sở thích học tập rất đa dạng. Vì bạn sẽ tuyển dụng những người mang nhiều phong cách khác nhau, việc chỉ đáp ứng một phong cách mà hy sinh các phong cách còn lại là một cách nhìn ngắn hạn. Do đó, không có phong cách giáo dục nào tối ưu nhất để đào tạo SRE mới, và chắc chắn không có công thức bí truyền duy nhất cho tất cả các đội SRE. [Bảng 28-1](#bang-28-1) liệt kê các [thực hành đào tạo](https://sre.google/resources/practices-and-processes/training-site-reliability-engineers/) (và các anti-pattern tương ứng) phổ biến trong SRE tại Google. Các thực hành này là một phạm vi rộng các lựa chọn giúp đội của bạn được trang bị tốt về các khái niệm SRE, cả ở hiện tại lẫn liên tục.

<a id="bang-28-1"></a>**Bảng 28-1. Các thực hành giáo dục SRE**

| **Mô-típ được đề xuất** | **Anti-patterns** |
|---|---|
| Thiết kế các trải nghiệm học tập cụ thể, tuần tự để học viên theo dõi | Trút cho học viên nhiều công việc vặt (ví dụ, triage alert/ticket) để đào tạo họ; "thử thách bằng lửa" |
| Khuyến khích reverse engineering, tư duy thống kê và làm việc từ các nguyên lý cơ bản | Đào tạo thuần túy qua các quy trình vận hành, checklist và playbook |
| Tôn vinh việc phân tích thất bại bằng cách đề xuất các postmortem cho học viên đọc | Xử lý các outage như những bí mật phải chôn vùi để tránh đổ lỗi |
| Tạo ra các sự hỏng hóc bị chứa gọn nhưng thực tế để học viên sửa chữa bằng các công cụ monitoring thật | Để cơ hội đầu tiên để sửa thứ gì đó chỉ xảy ra sau khi học viên đã on-call |
| Đóng vai các thảm họa lý thuyết như một nhóm, để đan xen các cách tiếp cận giải quyết vấn đề của một đội | Tạo ra các chuyên gia trong đội mà kỹ thuật và kiến thức của họ bị chia ngăn cách |
| Cho phép học viên shadow vòng on-call sớm, so sánh ghi chú với on-caller | Đẩy học viên vào làm on-call chính trước khi họ đạt được sự hiểu biết toàn diện về dịch vụ của mình |
| Ghép học viên với các SRE chuyên gia để xem lại các mục có tính mục tiêu của kế hoạch đào tạo on-call | Xử lý các kế hoạch đào tạo on-call như tĩnh và bất khả xâm phạm, ngoại trừ do các chuyên gia chuyên ngành |
| Dành ra công việc dự án không tầm thường để học viên thực hiện, cho phép họ đạt được sự sở hữu một phần trong stack | Gán tất cả công việc dự án mới cho các SRE cấp cao nhất, để các SRE cấp thấp nhặt lại phần còn thừa |

Phần còn lại của chương này trình bày các chủ đề lớn mà chúng tôi thấy hiệu quả trong việc tăng tốc SRE đến on-call và hơn thế. Các khái niệm này có thể hình dung trong một bản thiết kế (blueprint) để khởi tạo (bootstrap) các SRE ([Hình 28-1](#hinh-28-1)).

<a id="hinh-28-1"></a>        ![A blueprint for bootstrapping an SRE to on-call and beyond](../assets/imgs/fig-28-1.jpg)

**Hình 28-1.** Một bản thiết kế để khởi tạo một SRE đến on-call và hơn thế nữa

Hình minh họa này tổng hợp các thực hành tốt nhất mà các đội SRE có thể áp dụng để hỗ trợ thành viên mới hòa nhập, đồng thời giúp nhân tài cấp cao duy trì sự mới mẻ. Trong số nhiều công cụ được đề cập, bạn có thể chọn lọc những hoạt động phù hợp nhất với đội của mình.

Hình minh họa có hai trục:

-   Trục x đại diện cho *quang phổ giữa các loại công việc khác nhau*, trải dài từ các hoạt động trừu tượng đến các hoạt động áp dụng.
-   Trục y thể hiện *thời gian*. Khi đọc từ trên xuống, các SRE mới thường thiếu kiến thức về các hệ thống và dịch vụ mà họ sắp phụ trách, do đó việc bắt đầu bằng các postmortem mô tả chi tiết cách các hệ thống này từng gặp sự cố là một điểm khởi đầu hợp lý. Vì xuất phát điểm là con số không, các SRE mới cũng có thể thử reverse engineer các hệ thống từ những nguyên lý cơ bản. Sau khi nắm rõ hơn về hệ thống và tích lũy được một số kinh nghiệm thực tế, các SRE sẽ sẵn sàng shadow on-call và bắt đầu vá lại những tài liệu còn thiếu sót hoặc lỗi thời.

Mẹo để diễn giải hình minh họa này:

-   *Đi on-call* là một mốc trong sự nghiệp của SRE mới. Sau mốc này, việc học trở nên mơ hồ, không xác định và mang tính tự định hướng cao hơn rất nhiều — đó là lý do các hoạt động diễn ra tại hoặc sau khi SRE đi on-call được bao quanh bằng đường đứt nét.
-   Hình dạng tam giác của *công việc dự án & sở hữu* cho thấy rằng công việc dự án bắt đầu nhỏ và xây dựng theo thời gian, trở nên phức tạp hơn và nhiều khả năng tiếp tục rất lâu sau khi đi on-call.
-   Một số hoạt động và thực hành này mang tính trừu tượng/bị động, trong khi một số khác lại mang tính ứng dụng/chủ động. Có những hoạt động là sự kết hợp của cả hai. Việc có sự đa dạng trong các phương pháp học tập là điều tốt, giúp phù hợp với những phong cách học tập khác nhau.
-   Để đạt hiệu quả tối đa, các hoạt động và thực hành đào tạo cần được điều chỉnh nhịp độ phù hợp: một số nên thực hiện ngay, một số nên diễn ra ngay trước khi SRE chính thức đi on-call, và một số cần duy trì liên tục, kể cả với các SRE kỳ cựu. *Các trải nghiệm học tập cụ thể* nên trải dài trong suốt quá trình chuẩn bị cho việc [SRE đi on-call](https://sre.google/sre-book/being-on-call/).

## Những Trải nghiệm Học Tập Ban Đầu: Lập luận cho Cấu trúc hơn là Hỗn loạn (Initial Learning Experiences: The Case for Structure Over Chaos)

Như đã đề cập ở các phần khác trong cuốn sách, các đội SRE thường kết hợp một cách tự nhiên giữa công việc chủ động (proactive)<sup>[2](#fn2)</sup> và phản ứng (reactive)<sup>[3](#fn3)</sup>. Mục tiêu cốt lõi của mỗi đội SRE là kiểm soát và giảm thiểu công việc phản ứng thông qua các hoạt động chủ động, và quy trình onboarding người mới cũng không nằm ngoài nguyên tắc này. Hãy xem xét quy trình onboarding sau đây — một quy trình rất phổ biến nhưng đáng tiếc là chưa được tối ưu:

> John là thành viên mới nhất của đội SRE FooServer. Các SRE cấp cao trong đội này được giao rất nhiều công việc chân tay, như phản hồi các ticket (yêu cầu), đối phó với các alert (cảnh báo), và thực hiện các lần rollout (phát hành) binary tẻ nhạt. Vào ngày làm việc đầu tiên của John, anh ấy được gán tất cả các ticket đến mới. Anh ấy được nói rằng anh ấy có thể hỏi bất kỳ thành viên nào của đội SRE giúp anh ấy thu thập nền tảng cần thiết để giải mã một ticket. "Chắc chắn sẽ có rất nhiều việc học ban đầu mà bạn sẽ phải làm," quản lý của John nói. "Nhưng cuối cùng bạn sẽ nhanh hơn nhiều với những ticket này. Một ngày nào đó, nó sẽ *click* và bạn sẽ biết rất nhiều về tất cả các công cụ chúng tôi sử dụng, các quy trình chúng tôi tuân theo, và các hệ thống chúng tôi duy trì." Một thành viên cấp cao của đội bình luận, "Chúng tôi đang ném bạn vào đầu sâu của bể bơi đây."

Phương pháp "thử thách bằng lửa" này để định hướng người mới thường sinh ra từ môi trường hiện tại của đội; các đội SRE do ops điều hành, phản ứng, "đào tạo" thành viên mới bằng cách bắt họ…chà, phản ứng! Lặp đi lặp lại. Nếu may mắn, những kỹ sư đã giỏi điều hướng sự mơ hồ sẽ bò ra khỏi cái hố bạn đã đặt họ vào. Nhưng nhiều khả năng, chiến lược này đã làm xa lánh một số kỹ sư có năng lực. Cách tiếp cận như vậy có thể cuối cùng tạo ra các nhân viên vận hành giỏi, nhưng kết quả sẽ không đạt chuẩn. Phương pháp thử thách bằng lửa cũng giả định rằng nhiều hoặc phần lớn khía cạnh của một đội có thể dạy thuần túy bằng việc làm, chứ không phải bằng suy luận. Nếu tập hợp công việc trong một hàng đợi ticket đã đủ đào tạo cho công việc đó, thì đây không phải là vị trí SRE.

Các học viên SRE sẽ có những câu hỏi như:

-   Tôi đang làm việc về cái gì?
-   Tôi đã đạt được bao nhiêu tiến bộ?
-   Khi nào những hoạt động này tích lũy đủ kinh nghiệm để tôi đi on-call?

Việc chuyển từ công ty hoặc trường đại học trước đó, đồng thời đổi vai trò (từ kỹ sư phần mềm hoặc quản trị viên hệ thống truyền thống) sang vai trò *Site Reliability Engineer* mơ hồ này, thường đủ để làm lung lay sự tự tin của học viên. Đối với những người có tính cách nội tâm hơn (đặc biệt liên quan đến câu hỏi #2 và #3), sự bất định do các câu trả lời mơ hồ gây ra có thể khiến quá trình phát triển chậm lại hoặc dẫn đến các vấn đề về giữ chân. Thay vào đó, hãy xem xét các cách tiếp cận được phác thảo trong các mục tiếp theo. Những gợi ý này cụ thể như bất kỳ ticket hoặc alert nào, nhưng cũng tuần tự, và do đó bổ ích hơn nhiều.

## Các Lối đi Học Tập Tích lũy và Có Trật tự (Learning Paths That Are Cumulative and Orderly)

Hãy tạo ra một lộ trình học tập nhất định trong hệ thống của bạn để các SRE mới có thể nhìn thấy con đường phía trước. Bất kỳ hình thức đào tạo nào cũng tốt hơn việc phải đối mặt với các ticket và interrupt ngẫu nhiên, nhưng cần chú ý cân bằng đúng giữa lý thuyết và thực hành: những khái niệm trừu tượng sẽ xuất hiện nhiều lần trong quá trình làm quen nên được đưa lên trước, đồng thời học viên cũng cần được thực hành càng sớm càng tốt.

Để bắt đầu tìm hiểu về stack và subsystem (hệ thống con) của bạn, cần một điểm khởi đầu. Hãy cân nhắc xem nên nhóm các nội dung đào tạo theo sự tương đồng về mục đích hay theo thứ tự thực thi trong điều kiện bình thường. Chẳng hạn, nếu đội của bạn phụ trách một stack thời gian thực, hướng đến người dùng, hãy tham khảo trình tự chương trình học sau:

1) Cách một query (truy vấn) đi vào hệ thống

Cơ sở hạ tầng mạng và datacenter (trung tâm dữ liệu), load balancing (cân bằng tải) frontend (mặt trước), proxies (bộ định tuyến trung gian), v.v.

2) Phục vụ frontend

Các frontend ứng dụng, query logging (ghi log truy vấn), SLO (mục tiêu mức dịch vụ) của trải nghiệm người dùng, v.v.

3) Dịch vụ mid-tier (tầng trung gian)

Caches (bộ nhớ đệm), load balancing backend (phía sau)

4) Hạ tầng

Backends (hệ thống phía sau), hạ tầng, và các tài nguyên tính toán

5) Kết nối tất cả

Kỹ thuật debug (xử lý lỗi), quy trình leo thang (escalation), và các kịch bản khẩn cấp

Cách bạn chọn trình bày các cơ hội học tập (trò chuyện bảng trắng không chính thức, bài giảng chính thức, hay bài tập khám phá thực hành) do bạn và các SRE hỗ trợ bạn trong việc cấu trúc, thiết kế và triển khai đào tạo quyết định. Nhóm SRE của Google Search tổ chức việc học này thông qua một tài liệu gọi là "on-call learning checklist" (danh sách kiểm tra học tập on-call). Một phần được đơn giản hóa của một on-call learning checklist có thể trông như sau:

**The Results Mixing Server ("Mixer")**

**Frontended by**: Frontend server

**Backends called**: Results Retrieval Server, Geolocation Server, Personalization Database

**SRE experts**: Sally W, Dave K, Jen P

**Developer contacts**: Jim T, *results-team@*

**Know before moving on** (Cần biết trước khi tiếp tục):

-   Các cluster (cụm) nào có Mixer được triển khai
-   Cách rollback (hoàn tác) một release của Mixer
-   Các backends của Mixer nào được coi là "critical path" (đường lối quan trọng) và tại sao

**Read and understand the following docs** (Đọc và hiểu các tài liệu sau):

-   Results Mixing Overview: section "Query execution"
-   Results Mixing Overview: section "Production"
-   Playbook: How to Roll Out a New Results Mixing Server
-   A Performance Analysis of Mixer

**Comprehension questions** (Câu hỏi kiểm tra mức độ hiểu):

-   H: Lịch trình release thay đổi như thế nào nếu một ngày lễ của công ty xảy ra vào ngày build release bình thường?
-   H: Bạn có thể sửa một lần push (phát hành) xấu của dataset (bộ dữ liệu) geolocation (định vị địa lý) như thế nào?

Cần lưu ý rằng phần trên không trực tiếp mã hóa các quy trình, các bước chẩn đoán hay playbook. Thay vào đó, đây là một bản viết mang tính bất tử, tập trung nghiêm ngặt vào việc liệt kê các liên hệ chuyên gia, nổi bật các nguồn tài liệu hữu ích nhất, thiết lập kiến thức cơ bản bạn phải thu thập và nội hóa, đồng thời đặt ra các câu hỏi sâu sắc chỉ có thể trả lời khi đã hấp thụ kiến thức cơ bản đó. Phần này cũng nêu rõ các kết quả cụ thể, giúp học viên biết họ sẽ đạt được loại kiến thức và kỹ năng nào sau khi hoàn thành.

Điều hay là tất cả các bên liên quan đều nắm được mức độ thông tin mà người học đang giữ lại. Cơ chế phản hồi này có lẽ không cần chính thức như một bài kiểm tra, nhưng đây là thực hành tốt khi có các phần bài tập hoàn chỉnh đặt ra các câu hỏi về cách dịch vụ của bạn hoạt động. Những câu trả lời thỏa đáng, được mentor (người cố vấn) của học viên kiểm tra, là dấu hiệu cho thấy việc học nên tiếp tục sang giai đoạn tiếp theo. Các câu hỏi về cơ chế bên trong của dịch vụ của bạn có thể trông giống như:

-   Các backends nào của server này được coi là "trong critical path," và tại sao?
-   Những khía cạnh nào của server này có thể được đơn giản hóa hoặc tự động hóa?
-   Bạn nghĩ nút thắt cổ chai (bottleneck) đầu tiên trong kiến trúc này nằm ở đâu? Nếu phần đó bị quá tải, bạn có thể thực hiện những bước nào để giảm áp lực?

Tùy thuộc vào cách quyền truy cập được cấu hình cho dịch vụ, bạn có thể xem xét triển khai một mô hình truy cập phân tầng. Tầng đầu tiên cho phép học viên truy cập chỉ-đọc vào cơ chế bên trong của các thành phần, và một tầng sau đó cho phép họ thay đổi trạng thái production. Hoàn thành các phần của on-call learning checklist một cách thỏa đáng sẽ giúp học viên đạt truy cập ngày càng sâu hơn vào hệ thống. Đội Search SRE gọi các cấp độ đạt được này là "powerups"<sup>[4](#fn4)</sup> trên con đường đến on-call, vì học viên cuối cùng được thêm vào cấp truy cập hệ thống cao nhất.

## Công việc Dự án Có Tính Mục tiêu, Không phải Công việc Vặt (Targeted Project Work, Not Menial Work)

SRE là những người giải quyết vấn đề, nên hãy cho họ một vấn đề đàng hoàng để giải quyết! Khi mới bắt đầu, ngay cả một cảm giác sở hữu nhỏ đối với dịch vụ của đội cũng có thể làm những điều kỳ diệu cho việc học. Sự sở hữu như vậy còn tạo bước đột phá lớn cho việc xây dựng niềm tin giữa các đồng nghiệp cấp cao, vì họ sẽ tìm đến đồng nghiệp cấp thấp hơn để hiểu về các thành phần hoặc quy trình mới. Các cơ hội sở hữu ban đầu là tiêu chuẩn trên toàn Google: mọi kỹ sư đều được giao một dự án khởi đầu cung cấp một tour tham quan hạ tầng đủ để họ sớm đưa ra một đóng góp nhỏ nhưng hữu ích. Việc để SRE mới chia thời gian giữa việc học *và* công việc dự án cũng cho họ cảm giác mục đích và năng suất, điều sẽ không có nếu họ chỉ học *hoặc* chỉ làm dự án. Một số mô hình dự án khởi đầu có vẻ hiệu quả bao gồm:

-   Thực hiện một thay đổi giao diện cho người dùng thông thường trên một stack phục vụ, sau đó đưa tính năng đó lên production. Việc nắm vững cả toolchain phát triển lẫn quy trình release binary sẽ giúp bạn đồng cảm hơn với các nhà phát triển.
-   Bổ sung monitoring vào các dịch vụ đang là điểm mù. Người mới sẽ phải tự suy luận logic monitoring, đồng thời đối chiếu hiểu biết của họ về hệ thống với cách nó thực sự vận hành (đôi khi có sự sai lệch).
-   Tự động hóa một điểm đau chưa đủ nghiêm trọng để đã được xử lý, giúp SRE mới thấy được giá trị mà các SRE khác đặt vào việc loại bỏ toil (việc chân tay) khỏi các hoạt động hàng ngày.

## Tạo ra các Lập trình viên Phân tích Ngược và Nhà tư duy Tạm ứng Xuất sắc (Creating Stellar Reverse Engineers and Improvisational Thinkers)

Chúng tôi có thể đề xuất một bộ hướng dẫn về *cách* đào tạo SRE mới, nhưng *nên đào tạo họ về cái gì*? [Tài liệu đào tạo](https://sre.google/resources/practices-and-processes/training-site-reliability-engineers/) sẽ phụ thuộc vào công nghệ được sử dụng trong công việc, nhưng câu hỏi quan trọng hơn là: chúng ta đang muốn tạo ra những kỹ sư như thế nào? Ở quy mô và độ phức tạp mà SRE vận hành, họ không thể đơn thuần chỉ tập trung vào vận hành như những quản trị viên hệ thống truyền thống. Ngoài tâm thế kỹ thuật ở quy mô lớn, SRE nên thể hiện các đặc điểm sau:

-   Trong suốt công việc của họ, họ sẽ gặp các hệ thống mà họ chưa bao giờ thấy trước, nên họ cần phải có *kỹ năng reverse engineering mạnh mẽ*.
-   Ở quy mô lớn, sẽ có các bất thường khó phát hiện, nên họ sẽ cần khả năng *suy nghĩ thống kê*, chứ không phải thủ tục, để lột mặt các vấn đề.
-   Khi các quy trình vận hành tiêu chuẩn bị phá vỡ, họ sẽ cần phải có thể *tạm ứng hoàn toàn*.

Hãy xem xét các thuộc tính này kỹ hơn, để chúng ta có thể hiểu cách trang bị cho các SRE của chúng tôi những kỹ năng và hành vi này.

## Người Phân tích Ngược: Tìm ra Cách Các Thứ Hoạt Động (Reverse Engineers: Figuring Out How Things Work)

Các kỹ sư tò mò về cách các hệ thống mà họ chưa từng thấy hoạt động — hoặc, có khả năng hơn, cách các phiên bản hiện tại của những hệ thống họ từng biết khá kỹ đang hoạt động. Với một sự hiểu biết cơ sở về cách các hệ thống vận hành trong công ty, cộng với ý chí đào sâu vào các công cụ debug, các ranh giới RPC và các log của các binary, các SRE sẽ trở nên hiệu quả hơn khi truy tìm các vấn đề bất ngờ trong các kiến trúc hệ thống bất ngờ. Hãy dạy các SRE của bạn về các bề mặt chẩn đoán và debug của ứng dụng, và để họ thực hành rút ra các suy luận từ thông tin mà các bề mặt này tiết lộ, để hành vi đó thành phản xạ khi đối phó với các outage sau này.

## Người tư duy Thống kê và So sánh: Những Người Quản gia của Phương pháp Khoa học dưới Áp lực (Statistical and Comparative Thinkers: Stewards of the Scientific Method Under Pressure)

Bạn có thể hình dung cách SRE xử lý incident trên các hệ thống quy mô lớn giống như việc điều hướng qua một cây quyết định khổng lồ đang mở ra trước mắt. Trong khoảng thời gian hạn hẹp do yêu cầu phản ứng incident đặt ra, SRE chỉ có thể thực hiện một vài hành động trong số hàng trăm khả năng, nhằm giảm nhẹ sự cố (outage) cả về ngắn hạn lẫn dài hạn. Vì thời gian thường là yếu tố quan trọng nhất, SRE phải biết cách cắt tỉa cây quyết định này một cách hiệu quả. Khả năng đó một phần đến từ kinh nghiệm, thứ chỉ có được theo thời gian qua việc tiếp xúc với nhiều hệ thống production. Kinh nghiệm này cần được kết hợp với việc xây dựng cẩn thận các giả thuyết; khi được chứng minh hoặc bác bỏ, chúng sẽ giúp thu hẹp hơn nữa không gian quyết định. Nói cách khác, quá trình truy tìm sự cố thường giống một trò chơi "trong số những thứ này, thứ nào không giống?", trong đó "những thứ" có thể là phiên bản kernel, kiến trúc CPU, phiên bản binary trong stack, sự pha trộn traffic theo vùng, hoặc hàng trăm yếu tố khác. Về mặt kiến trúc, trách nhiệm của đội là đảm bảo các yếu tố này có thể được kiểm soát và so sánh từng cái một. Tuy nhiên, chúng ta cũng nên đào tạo các SRE mới nhất trở thành những nhà phân tích, người so sánh giỏi ngay từ những khoảnh khắc đầu tiên.

## Nghệ sĩ Tạm ứng: Khi Điều Bất Ngờ Xảy Ra (Improv Artists: When the Unexpected Happens)

Bạn thử một cách sửa nhưng không thành công, trong khi các nhà phát triển phụ trách hệ thống đang gặp sự cố thì không liên lạc được. Lúc này bạn làm gì? Bạn tạm ứng! Việc nắm vững nhiều công cụ để xử lý từng phần của vấn đề giúp bạn áp dụng chiến lược phòng thủ nhiều lớp trong quá trình giải quyết sự cố. Nếu quá phụ thuộc vào quy trình trước một outage mà quên mất kỹ năng phân tích, bạn có thể sẽ bị mắc kẹt thay vì tìm ra nguyên nhân gốc rễ. Một tình huống troubleshooting bế tắc có thể xảy ra khi SRE đưa ra quyết định dựa trên quá nhiều giả định chưa được kiểm chứng về nguyên nhân của outage. Việc nhận ra rằng SRE dễ mắc phải nhiều bẫy phân tích, từ đó cần "zoom out" (co nhỏ) và tiếp cận giải pháp theo cách khác, là bài học quý giá mà SRE nên nắm bắt sớm.

Với ba thuộc tính đầy tham vọng này của các SRE hiệu suất cao, chúng ta có thể cung cấp những khóa học và trải nghiệm nào cho các SRE mới để giúp họ đi đúng hướng? Bạn cần tự thiết kế nội dung khóa học phản ánh những thuộc tính này, cùng với các đặc điểm khác phù hợp với văn hóa SRE của tổ chức. Dưới đây là một lớp học mà chúng tôi tin rằng đáp ứng được tất cả các yêu cầu nêu trên.

## Kết nối Tất cả lại: Reverse Engineering một Dịch vụ Production (Tying This Together: Reverse Engineering a Production Service)

> Khi đến lúc phải học [một phần của stack Google Maps], [một SRE mới] hỏi rằng, thay vì thụ động để ai đó giải thích dịch vụ, cô ấy có thể tự làm điều này không — học mọi thứ thông qua các kỹ thuật lớp reverse engineering, và để phần còn lại của chúng tôi sửa cho cô ấy hoặc điền vào những khoảng trống cho bất kỳ thứ gì cô ấy bỏ lỡ hoặc sai. Kết quả? Có lẽ nó chính xác và hữu ích hơn so với nếu *tôi* đã tự thuyết trình, và tôi đã on-call cho hệ thống này hơn 5 năm rồi!
> 
> Paul Cowan, Google Site Reliability Engineer

Một lớp phổ biến mà chúng tôi tổ chức tại Google mang tên “Reverse Engineering a Production Service (without help from its owners)”. Kịch bản vấn đề ban đầu có vẻ đơn giản. Toàn bộ đội Google News — SRE, kỹ sư phần mềm, quản lý sản phẩm, v.v. — đã đi một chuyến công ty: du thuyền quanh Tam giác Bermuda. Chúng tôi mất liên lạc với đội suốt 30 ngày, nên các học viên của chúng tôi là đội SRE Google News vừa được bổ nhiệm. Họ cần tìm ra cách stack phục vụ hoạt động từ đầu đến cuối để nắm quyền điều hành và giữ cho nó chạy.

Sau khi nắm được kịch bản, học viên sẽ thực hiện các bài tập tương tác có chủ đích, trong đó họ theo dõi hành trình của query từ trình duyệt web xuyên qua hạ tầng Google. Ở mỗi giai đoạn, chúng tôi nhấn mạnh tầm quan trọng của việc học nhiều phương pháp để khám phá tính kết nối giữa các server production, nhằm tránh bỏ sót bất kỳ kết nối nào. Trong giờ học, chúng tôi thách thức học viên tìm một endpoint khác cho traffic đến, qua đó chứng minh rằng giả định ban đầu của chúng tôi có phạm vi quá hẹp. Tiếp đó, chúng tôi yêu cầu họ tìm các lối vào khác của stack. Chúng tôi khai thác đặc điểm đo lường cao của các binary production, vốn tự động báo cáo tính kết nối RPC, cùng với các công cụ giám sát hộp trắng và hộp đen (black-box monitoring) có sẵn, để xác định query của người dùng đi theo đường nào.<sup>[5](#fn5)</sup> Trong quá trình này, chúng tôi xây dựng sơ đồ hệ thống và thảo luận về các thành phần hạ tầng dùng chung mà học viên có khả năng sẽ gặp lại.

Cuối buổi học, mỗi học viên về nhóm và nhờ một SRE cấp cao chọn một stack, hoặc một phần của stack, mà họ sẽ on-call. Áp dụng kỹ năng đã học, học viên tự vẽ sơ đồ stack đó và trình bày những phát hiện cho SRE cấp cao. Chắc chắn sẽ có một vài chi tiết tinh tế bị bỏ sót, tạo nên cuộc thảo luận thú vị. SRE cấp cao cũng có thể học được điều gì đó từ bài tập, qua đó thấy rõ sự lệch lạc trong hiểu biết trước đó của họ về hệ thống luôn thay đổi. Vì các hệ thống production thay đổi nhanh, nhóm của bạn cần đón nhận mọi cơ hội làm quen lại với hệ thống, kể cả học từ những thành viên mới nhất chứ không chỉ những người cũ nhất.

## Năm Thực hành cho những Người Muốn On-Call (Five Practices for Aspiring On-Callers)

On-call không phải là mục đích quan trọng duy nhất của bất kỳ SRE nào, nhưng trách nhiệm kỹ thuật production thường đi kèm với một dạng phủ thông báo khẩn cấp nào đó. Người có khả năng nhận on-call là người hiểu hệ thống họ làm việc ở một chiều sâu và phạm vi hợp lý. Vì vậy, chúng tôi dùng "có khả năng nhận on-call" như một biến thay hữu ích cho "biết đủ và có thể tìm ra phần còn lại".

## Một Sự Đói khát Thất bại: Đọc và Chia sẻ các Postmortem (A Hunger for Failure: Reading and Sharing Postmortems)

> Những ai không thể nhớ quá khứ bị kết án phải lặp lại nó.
> 
> George Santayana, triết gia và nhà văn tiểu luận

Postmortem (xem [Văn hóa Postmortem: Học hỏi từ Thất bại](https://sre.google/sre-book/postmortem-culture/)) là một phần quan trọng của cải tiến liên tục. Chúng là cách không đổ lỗi để đi đến nhiều nguyên nhân gốc rễ của một outage đáng kể hoặc có thể nhìn thấy. Khi viết postmortem, hãy nhớ rằng khán giả trân trọng nó nhất có thể là một kỹ sư chưa được thuê. Không cần chỉnh sửa triệt để; những thay đổi tinh tế có thể biến các postmortem tốt nhất của chúng tôi thành các postmortem "có thể dạy được".

Ngay cả những postmortem tốt nhất cũng vô dụng nếu chỉ nằm im trong một tủ hồ sơ ảo. Vì vậy, đội của bạn nên tuyển chọn và lưu trữ các postmortem có giá trị làm [nguồn tài liệu giáo dục](https://sre.google/resources/) cho nhân sự mới. Dù một số postmortem mang tính lặp lại, nhưng những "postmortem có thể dạy được" — cung cấp hiểu biết sâu sắc về các sự cố cấu trúc hoặc mới mẻ trên hệ thống quy mô lớn — lại quý giá như vàng đối với người học.

Trách nhiệm với postmortem không chỉ nằm ở khâu viết. Với nhiều đội, việc vượt qua và ghi lại tài liệu về các outage lớn nhất là một điểm tự hào. Hãy tập hợp những postmortem tốt nhất của bạn và đặt ở nơi dễ thấy để người mới — cùng các bên liên quan từ các đội liên quan và/hoặc tích hợp — có thể đọc. Đồng thời, yêu cầu các đội liên quan công bố postmortem tốt nhất của họ ở nơi bạn có thể truy cập.

Một số nhóm SRE tại Google duy trì “câu lạc bộ đọc postmortem”, nơi những bài postmortem thú vị và sâu sắc được chia sẻ, đọc trước rồi cùng nhau thảo luận. Người viết postmortem gốc có thể được mời làm khách danh dự tại cuộc họp. Các nhóm khác lại tổ chức những buổi “tales of fail” (lời kể về thất bại), trong đó tác giả postmortem trình bày bán chính thức, kể lại sự cố ngừng dịch vụ và tự mình điều khiển phần thảo luận.

Việc đọc hoặc trình bày định kỳ các outage, bao gồm điều kiện kích hoạt và các bước giảm nhẹ, giúp SRE mới xây dựng bản đồ tâm trí và hiểu rõ hơn về production cũng như phản ứng on-call. Các postmortem cũng là nguồn tài liệu tuyệt vời để xây dựng các kịch bản thảm họa trừu tượng sau này.

## Đóng vai Thảm họa (Disaster Role Playing)

> Mỗi tuần một lần chúng tôi có một cuộc họp trong đó một nạn nhân được chọn để bị đặt vào tình thế khó xử trước nhóm, và một kịch bản — thường là một kịch bản thật được lấy từ các biên niên sử của lịch sử Google — được ném vào anh hoặc cô ấy. Nạn nhân, người mà tôi nghĩ như một người chơi show trò chơi, nói với người dẫn chương trình show trò chơi rằng họ sẽ làm gì hoặc truy vấn gì để hiểu hoặc giải quyết vấn đề, và người dẫn chương trình nói cho nạn nhân điều gì xảy ra với mỗi hành động hoặc quan sát. Nó giống như *SRE Zork*. Bạn trong một mê cung của các console giám sát ngoằn ngoèo, tất cả giống hệt nhau. Bạn phải cứu những người dùng vô tội khỏi việc trượt vào Hẻm vực Độ trễ Truy vấn Quá mức, cứu các datacenter khỏi Sự Cháy nổ Gần như Chắc chắn, và tha cho tất cả chúng ta sự xấu hổ của Việc Hiển thị Doodle Google Sai lầm.
> 
> Robert Kennedy, cựu Site Reliability Engineer cho Google Search và healthcare.gov<sup>[6](#fn6)</sup>

Khi một nhóm SRE có sự chênh lệch kinh nghiệm rất lớn, làm thế nào để gắn kết mọi người và tạo điều kiện cho họ học hỏi lẫn nhau? Bạn truyền đạt văn hóa SRE cùng tinh thần giải quyết vấn đề của đội cho người mới ra sao, trong khi vẫn giúp các thành viên kỳ cựu cập nhật những thay đổi và tính năng mới trong stack? Các nhóm SRE tại Google đối mặt với thách thức này bằng một truyền thống được coi trọng: thường xuyên diễn tập thảm họa. Ngoài nhiều tên gọi khác, bài tập này hay được gọi là "Wheel of Misfortune" hoặc "Walk the Plank". Cảm giác nguy hiểm mang tính hài hước từ những cái tên như vậy giúp các SRE mới tuyển bớt e ngại hơn khi tham gia.

Ở mức tốt nhất, những bài tập này trở thành nghi lễ hàng tuần, giúp mọi thành viên nhóm đều học được điều gì đó. Công thức này trực tiếp và có nét tương đồng với tabletop RPG (trò chơi nhập vai): "game master" (GM) chọn hai thành viên đội làm on-call chính và thứ cấp; hai SRE này cùng GM đứng ở phía trước phòng. Một lượt gọi trực (page) được thông báo, và đội on-call phản ứng bằng cách mô tả những gì họ sẽ làm để giảm nhẹ và điều tra outage.

GM đã chuẩn bị kỹ lưỡng cho kịch bản sắp diễn ra. Kịch bản này có thể dựa trên một sự cố outage trước đó mà các thành viên mới của đội chưa từng chứng kiến, hoặc mà các thành viên cũ đã quên. Hoặc có thể đó là một cuộc thử nghiệm giả định về sự hỏng hóc của một tính năng mới hoặc sắp ra mắt trong stack, khiến tất cả mọi người trong phòng đều ở trạng thái chưa sẵn sàng đồng đều để ứng phó. Tốt hơn nữa, một đồng nghiệp có thể vừa phát hiện một lỗi mới trong production, và kịch bản hôm nay sẽ được xây dựng mở rộng từ mối đe dọa mới này.

Trong 30–60 phút tiếp theo, on-caller chính và thứ cấp tập trung tìm nguyên nhân gốc rễ. Khi sự cố bắt đầu, GM sẵn lòng cung cấp thêm ngữ cảnh, ví dụ như cho on-caller (và khán giả) thấy các biểu đồ trên dashboard giám sát có thể hiển thị ra sao trong suốt thời gian outage. Nếu incident cần leo thang ra ngoài đội chủ, GM sẽ đóng vai một thành viên của đội kia để phục vụ kịch bản. Vì không có kịch bản ảo nào là hoàn hảo, đôi khi GM phải đưa người tham gia về đúng hướng bằng cách chuyển on-caller khỏi các mồi nhử (red herring), tạo thêm sự khẩn cấp và rõ ràng thông qua các kích thích khác,<sup>[7](#fn7)</sup> hoặc đặt những câu hỏi khẩn cấp và thấu đáo.<sup>[8](#fn8)</sup>

Khi RPG thảm họa của bạn thành công, mọi người sẽ học được điều gì đó: có lẽ một công cụ hoặc mẹo mới, một góc nhìn khác về cách giải quyết một vấn đề, hoặc (đặc biệt làm hài lòng các thành viên đội mới) một sự xác nhận rằng bạn đã có thể giải quyết vấn đề của tuần này nếu bạn được chọn. Với một chút may mắn, bài tập này sẽ truyền cảm hứng cho các đồng đội háo hức mong đợi cuộc phiêu lưu của tuần tới hoặc để yêu cầu trở thành game master cho một tuần sắp tới.

## Phá Vỡ Các Thứ Thật, Sửa Các Thứ Thật (Break Real Things, Fix Real Things)

Người mới có thể học hỏi nhiều về SRE thông qua tài liệu, postmortem và các khóa đào tạo. Đóng vai thảm họa giúp đưa người mới vào cuộc. Tuy nhiên, kinh nghiệm từ việc thực sự phá vỡ và/hoặc sửa chữa các hệ thống production *thật* còn giá trị hơn. Người mới sẽ có đủ thời gian để tích lũy kinh nghiệm thực hành khi đi on-call, nhưng quá trình học này nên diễn ra *trước* khi SRE mới đến giai đoạn đó. Vì vậy, hãy cung cấp những trải nghiệm thực hành này sớm hơn nhiều, để phát triển phản xạ ứng phó của học viên khi sử dụng các công cụ và monitoring của công ty để tiếp cận một outage đang phát triển.

Trong những tương tác này, tính chân thực là yếu tố tối thượng. Tốt nhất, đội bạn nên có một stack được multihomed và provision sao cho ít nhất một instance có thể tách khỏi traffic trực tiếp để cho mượn tạm thời phục vụ bài tập. Nếu không, bạn có thể dùng một instance staging hoặc QA nhỏ hơn nhưng vẫn đầy đủ tính năng của stack, mượn trong thời gian ngắn. Nếu có thể, hãy chủ động chạy stack dưới tải tổng hợp (synthetic load) xấp xỉ traffic người dùng/client thực, kèm theo mức tiêu thụ tài nguyên tương ứng.

Hệ thống production thực chịu tải tổng hợp mang lại vô vàn cơ hội học hỏi. Các SRE cấp cao đã từng đối mặt đủ loại rắc rối: cấu hình sai, rò rỉ bộ nhớ, suy giảm hiệu năng, truy vấn sập, nút thắt cổ chai lưu trữ, và nhiều vấn đề khác. Trong môi trường thực tế nhưng tương đối an toàn này, người giám sát (proctor) có thể thao tác tập hợp job để thay đổi hành vi của stack, buộc SRE mới phải tìm ra các khác biệt, xác định các yếu tố đóng góp, và cuối cùng sửa chữa hệ thống nhằm khôi phục hành vi phù hợp.

Thay vì bắt một SRE cấp cao phải lên kế hoạch chi tiết cho một sự cố cụ thể để các SRE mới xử lý, bạn có thể làm ngược lại với một bài tập giúp cả nhóm cùng tham gia: bắt đầu từ một cấu hình hoạt động tốt, sau đó dần làm suy yếu hệ thống tại các nút thắt cổ chai được chọn, đồng thời quan sát phản ứng của các dịch vụ phía trước và phía sau qua hệ thống giám sát. Nhóm SRE của Google Search rất coi trọng bài tập này, nơi họ gọi nó là "Let's burn a search cluster to the ground!" (Hãy đốt cháy một cụm tìm kiếm đến đống tro!). Cách thực hiện bài tập như sau:

1.  Như một nhóm, chúng tôi thảo luận về những đặc tính hiệu năng quan sát được nào có thể thay đổi khi chúng tôi làm tê liệt stack.
2.  Trước khi gây ra hư hại đã lên kế hoạch, chúng tôi thăm dò những người tham gia về những phỏng đoán và suy luận của họ về các dự đoán của họ về cách hệ thống sẽ phản ứng.
3.  Chúng tôi xác thực các giả định và biện minh cho suy luận đằng sau các hành vi mà chúng tôi thấy.

Bài tập này, mà chúng tôi thực hiện hàng quý, phơi bày các lỗi mới mà chúng tôi nhiệt tình sửa chữa, vì các hệ thống của chúng tôi không phải lúc nào cũng suy giảm một cách nhẹ nhàng như chúng tôi mong đợi.

<a id="bang-28-2"></a>

## Tài liệu như Sự Học việc (Documentation as Apprenticeship)

Nhiều đội SRE duy trì một "on-call learning checklist", tức danh sách các công nghệ và khái niệm liên quan đến hệ thống mà họ vận hành, để người học đọc và nắm bắt. Trước khi đủ điều kiện làm on-caller shadow, người học phải nội hóa danh sách này. Hãy dành một lát xem lại ví dụ về on-call learning checklist ở [Bảng 28-2](#bang-28-2). Danh sách kiểm tra học tập phục vụ các mục đích khác nhau cho những người khác nhau:

-   **Đối với học viên**:
    
    -   Tài liệu này giúp thiết lập các ranh giới của hệ thống mà đội của họ hỗ trợ.
    -   Bằng cách nghiên cứu danh sách này, học viên đạt được một cảm giác về hệ thống nào quan trọng nhất và tại sao. Khi họ hiểu thông tin ở đó, họ có thể chuyển sang các chủ đề khác mà họ cần học, thay vì sa vào việc học các chi tiết bí ẩn có thể học được theo thời gian.
-   **Đối với những người cố vấn và quản lý**: Có thể theo dõi tiến độ của học viên qua danh sách kiểm tra học tập. Danh sách này trả lời các câu hỏi như:
    
    -   Bạn đang làm việc trên phần nào hôm nay?
    -   Những phần nào gây bối rối nhất?
-   **Đối với tất cả các thành viên đội**: Tài liệu đóng vai trò như một hợp đồng xã hội, theo đó (khi đã thành thạo) học viên sẽ gia nhập hàng ngũ on-call. Danh sách kiểm tra học tập xác lập tiêu chuẩn mà mọi thành viên trong đội cần hướng tới và duy trì.
    

Trong môi trường thay đổi nhanh, tài liệu dễ bị lỗi thời. Với các SRE cấp cao đã nắm bắt kịp tiến độ, tài liệu lỗi thời ít khi là vấn đề, vì họ luôn giữ trong đầu toàn bộ trạng thái hệ thống cùng những thay đổi liên quan. Ngược lại, SRE mới lại cần tài liệu cập nhật hơn nhiều, nhưng thường không cảm thấy được trao quyền hoặc đủ kiến thức để chỉnh sửa. Nếu được thiết kế với mức độ cấu trúc phù hợp, tài liệu on-call có thể trở thành một khối lượng công việc thích ứng, tận dụng sự nhiệt huyết của người mới và kiến thức của người cấp cao để giúp tất cả mọi người luôn cập nhật thông tin.

Trong Search SRE, chúng tôi chuẩn bị đón các thành viên đội mới bằng cách rà soát on-call learning checklist và sắp xếp các phần theo mức độ cập nhật. Khi thành viên mới đến, chúng tôi chỉ họ đến danh sách kiểm tra học tập tổng thể, đồng thời giao cho họ nhiệm vụ đại tu một hoặc hai phần lỗi thời nhất. Như trong [Bảng 28-2](#bang-28-2), chúng tôi dán nhãn liên hệ SRE cấp cao và nhà phát triển cho mỗi công nghệ. Chúng tôi khuyến khích học viên thiết lập kết nối sớm với những chuyên gia đó để có thể học trực tiếp cơ chế bên trong của công nghệ đã chọn. Sau đó, khi quen hơn với phạm vi và giọng điệu của danh sách kiểm tra, họ được kỳ vọng đóng góp một phần danh sách đã sửa đổi, phần này phải được peer-review bởi một hoặc nhiều SRE cấp cao được liệt kê làm chuyên gia.

## Shadow On-Call Sớm và Thường xuyên (Shadow On-Call Early and Often)

Cuối cùng, không có lượng bài tập thảm họa giả định hay cơ chế đào tạo nào khác có thể chuẩn bị hoàn toàn cho một SRE đi on-call. Về bản chất, đối phó với các outage thật luôn mang lại giá trị học tập cao hơn so với việc tham gia các giả định. Tuy nhiên, không công bằng khi bắt người mới phải đợi đến lượt gọi trực thật đầu tiên mới có cơ hội học và giữ kiến thức.

Sau khi học viên đã làm quen với các nền tảng hệ thống (ví dụ, bằng cách hoàn thành một on-call learning checklist), hãy cân nhắc cấu hình hệ thống cảnh báo để sao chép các lượt gọi trực đến cho người mới, ban đầu chỉ trong giờ làm việc. Hãy để sự tò mò của họ dẫn đường. Những ca on-call "shadow" này là cách tuyệt vời để người cố vấn thấy được tiến độ của học viên, và để học viên hiểu rõ các trách nhiệm của on-call. Bằng cách sắp xếp cho người mới shadow nhiều thành viên đội, đội sẽ ngày càng thoải mái với ý nghĩ người này sẽ vào [vòng on-call.](https://sre.google/sre-book/being-on-call/) Việc gieo niềm tin theo cách này là một phương pháp hiệu quả xây dựng tin tưởng, cho phép các thành viên cấp cao hơn tách ra khi không on-call, do đó giúp tránh sự kiệt sức của đội.

Khi có cuộc gọi trực, SRE mới không phải là on-caller được chỉ định, nên không chịu áp lực về thời gian. Thay vì chỉ biết sự cố sau khi đã xử lý xong, họ được chứng kiến trực tiếp quá trình outage diễn ra. Học viên và on-caller chính có thể cùng dùng một phiên terminal hoặc ngồi cạnh nhau để tiện so sánh ghi chú. Sau khi outage kết thúc, vào thời điểm phù hợp, on-caller sẽ cùng học viên xem lại các suy luận và quy trình đã thực hiện. Cách làm này giúp on-caller shadow ghi nhớ tốt hơn những gì thực sự đã xảy ra.

### Mẹo (Tip)

Nếu một outage xảy ra và việc viết postmortem là có lợi, on-caller nên mời người mới tham gia với tư cách đồng tác giả. *Đừng giao toàn bộ bài viết cho học viên, vì điều đó có thể khiến người khác hiểu nhầm rằng postmortem là một loại công việc chân tay cần chuyển cho những người trẻ nhất. Tạo ra ấn tượng như vậy sẽ là một sai lầm.*

Một số đội còn thêm bước cuối: để on-caller có kinh nghiệm "reverse shadow" học viên. Người mới trở thành on-call chính và nhận tất cả các leo thang, trong khi on-caller có kinh nghiệm ẩn trong bóng tối, độc lập chẩn đoán tình huống mà không thay đổi bất kỳ trạng thái nào. SRE có kinh nghiệm luôn sẵn sàng hỗ trợ chủ động, giúp đỡ, xác thực và gợi ý khi cần.

## On-Call và Hơn thế: Nghi lễ Qua cửa ải, và Thực hành Giáo dục Liên tục (On-Call and Beyond: Rites of Passage, and Practicing Continuing Education)

Khi kiến thức tích lũy đủ, học viên sẽ đến một giai đoạn trong sự nghiệp mà họ có thể suy luận thoải mái qua phần lớn stack, và tạm ứng qua phần còn lại. Lúc này, họ nên bắt đầu đi on-call cho dịch vụ của mình. Một số đội tổ chức kỳ thi cuối cùng để đánh giá học viên lần chót trước khi trao quyền và trách nhiệm on-call. Các SRE mới khác sẽ nộp on-call learning checklist đã hoàn thành như bằng chứng cho thấy họ đã sẵn sàng. Dù bạn kiểm soát mốc này theo cách nào, việc đi on-call là một nghi lễ qua cửa ải và nên được cả đội ăn mừng.

Việc học có dừng lại khi một học viên gia nhập hàng ngũ on-call? Tất nhiên là không! Để vẫn cảnh giác như các SRE, đội bạn luôn cần chủ động và nhận thức về các thay đổi sắp tới. Trong khi sự chú ý của bạn ở nơi khác, các phần stack của bạn có thể được tái kiến trúc và mở rộng, để lại kiến thức vận hành của đội bạn ở mức tốt nhất là lịch sử.

Hãy thiết lập chuỗi học tập định kỳ cho cả đội, trong đó các SRE trực tiếp thực hiện thay đổi sẽ trình bày tổng quan về những cập nhật mới và sắp tới của stack; khi cần, họ có thể mời nhà phát triển đồng trình bày. Nếu có thể, hãy thu âm các bài trình bày để xây dựng thư viện đào tạo cho các học viên sau này.

Chỉ cần luyện tập một chút, bạn sẽ nhận được sự phối hợp kịp thời từ cả SRE trong đội lẫn các nhà phát triển làm việc sát sao với đội, đồng thời giúp mọi người luôn có cái nhìn tươi mới về tương lai. Ngoài ra, bạn có thể tận dụng các kênh giáo dục khác: hãy cân nhắc việc mời SRE chia sẻ với các đồng nghiệp phát triển. Đồng nghiệp phát triển càng hiểu rõ công việc và những thách thức mà đội bạn đang đối mặt, thì việc đưa ra các quyết định có cơ sở vững chắc cho các dự án sau này sẽ càng dễ dàng.

## Những Suy nghĩ Kết thúc (Closing Thoughts)

Khoản đầu tư ban đầu vào đào tạo SRE chắc chắn đáng giá, cả với học viên háo hức nắm bắt môi trường production lẫn với các đội vui mừng được chào đón học viên vào hàng ngũ on-call. Thông qua việc áp dụng các thực hành khả thi được phác thảo trong chương này, bạn sẽ tạo ra các SRE toàn diện nhanh hơn, trong khi liên tục mài nhọn kỹ năng của đội. Cách bạn áp dụng các thực hành này là do bạn, nhưng mệnh lệnh là rõ ràng: với tư cách SRE, bạn phải scale con người nhanh hơn scale máy móc. Chúc may mắn cho bạn và các đội trong việc tạo ra một văn hóa học và dạy!

<a id="fn1"></a>[1](#fn1) Và không hoạt động!

<a id="fn2"></a>[2](#fn2) Các ví dụ về công việc SRE chủ động bao gồm tự động hóa phần mềm, tham vấn thiết kế, và phối hợp ra mắt.

<a id="fn3"></a>[3](#fn3) Các ví dụ về công việc SRE phản ứng bao gồm debug, troubleshooting (xử lý lỗi), và xử lý các leo thang on-call.

<a id="fn4"></a>[4](#fn4) Một lời chào đến các trò chơi điện tử của thời xa xưa.

<a id="fn5"></a>[5](#fn5) Cách tiếp cận "theo dõi RPC" này cũng hoạt động tốt cho các hệ thống batch/pipeline (luồng); bắt đầu với thao tác khởi động hệ thống. Đối với các hệ thống batch, thao tác này có thể là dữ liệu đến cần được xử lý, một giao dịch cần được xác thực, hoặc nhiều sự kiện khác.

<a id="fn6"></a>[6](#fn6) Xem ["Life in the Trenches of healthcare.gov"](https://www.thedotpost.com/2014/05/robert-kennedy-life-in-the-trenches-of-healthcare-gov) (Sự sống trong Trenches của healthcare.gov).

<a id="fn7"></a>[7](#fn7) Ví dụ: "Bạn đang bị gọi trực bởi một đội khác mang đến cho bạn nhiều thông tin hơn. Đây là điều họ nói…"

<a id="fn8"></a>[8](#fn8) Ví dụ: "Chúng tôi đang mất tiền nhanh! Làm thế nào bạn có thể chặn sự chảy máu trong ngắn hạn?"

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
