# Chương 15. Văn hóa Postmortem: Học từ Thất bại (Postmortem Culture: Learning from Failure)

> **Nguyên bản:** [Chapter 15 - Postmortem Culture: Learning from Failure](https://sre.google/sre-book/postmortem-culture/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* John Lunney và Sue Lueder
*Biên tập:* Gary O'Connor

> Chi phí của thất bại là sự giáo dục.
>
> Devin Carraway

Là SRE (Site Reliability Engineering — Kỹ thuật Độ tin cậy Trang web), chúng tôi vận hành các hệ thống phân tán quy mô lớn, phức tạp. Chúng tôi liên tục nâng cấp dịch vụ bằng các tính năng mới và bổ sung hệ thống mới. Với quy mô và tốc độ thay đổi như vậy, incident (sự cố) và outage (mất dịch vụ) là điều khó tránh khỏi. Khi incident xảy ra, chúng tôi xử lý nguyên nhân gốc rễ để đưa dịch vụ về trạng thái vận hành bình thường. Tuy nhiên, nếu không có quy trình chính thức hóa việc rút kinh nghiệm từ các incident, chúng có thể tái diễn vô tận (ad infinitum). Nếu không kiểm soát, incident có thể leo thang về độ phức tạp, thậm chí lan truyền (cascade), khiến hệ thống và đội ngũ vận hành quá tải, cuối cùng ảnh hưởng đến người dùng. Vì vậy, postmortem (báo cáo sau sự cố) là công cụ thiết yếu của SRE.

Trong ngành công nghệ, khái niệm postmortem đã được biết đến rộng rãi [[All12]](https://sre.google/sre-book/bibliography#All12). Một postmortem là bản ghi văn bản về một incident, bao gồm tác động của nó, các hành động đã thực hiện để giảm nhẹ hoặc giải quyết, nguyên nhân gốc rễ (root cause), và các hành động tiếp theo nhằm ngăn incident tái phát. Chương này mô tả các tiêu chí để quyết định khi nào cần làm postmortem, một số thực hành tốt nhất xoay quanh postmortem, cùng lời khuyên về cách nuôi dưỡng văn hóa postmortem dựa trên kinh nghiệm chúng tôi tích lũy qua nhiều năm.

## Triết lý Postmortem của Google (Google's Postmortem Philosophy)

Mục tiêu chính của [việc viết postmortem](https://sre.google/sre-book/example-postmortem/) là đảm bảo incident được ghi lại, tất cả các nguyên nhân gốc rễ đóng góp đều được hiểu rõ, và đặc biệt là các hành động phòng ngừa hiệu quả được triển khai đúng chỗ nhằm giảm khả năng và/hoặc tác động của sự tái phát. Một khảo sát chi tiết về các kỹ thuật phân tích nguyên nhân gốc rễ nằm ngoài phạm vi chương này (xem [[Roo04]](https://sre.google/sre-book/bibliography#Roo04) thay thế); tuy nhiên, các bài viết, thực hành tốt nhất và công cụ trong lĩnh vực chất lượng hệ thống đều rất phong phú. Các đội chúng tôi sử dụng nhiều kỹ thuật cho phân tích nguyên nhân gốc rễ và chọn kỹ thuật phù hợp nhất với dịch vụ của mình. Chúng tôi kỳ vọng sẽ có postmortem sau bất kỳ sự kiện bất lợi đáng kể nào. Việc viết postmortem không phải là hình phạt — nó là một cơ hội học tập cho toàn công ty. Quy trình postmortem có một chi phí vốn có về thời gian và nỗ lực, nên chúng tôi cân nhắc kỹ khi nào viết. Các đội có một mức linh hoạt nội bộ nhất định, nhưng các [triggers (tác nhân kích hoạt) postmortem](https://sre.google/workbook/postmortem-analysis/) phổ biến bao gồm:

-   Downtime (ngừng dịch vụ) hoặc suy giảm nhìn thấy được bởi người dùng vượt quá một ngưỡng nhất định
-   Mất dữ liệu (data loss) bất kỳ loại nào
-   Sự can thiệp của kỹ sư on-call (trực sự cố) (hoàn tác release — release rollback, định tuyến lại traffic — rerouting of traffic, v.v.)
-   Một thời gian giải quyết trên một ngưỡng nào đó
-   Một sự cố của giám sát (monitoring) (thường ngụ ý việc phát hiện incident thủ công)

Cần định nghĩa rõ các tiêu chí postmortem trước khi incident xảy ra để mọi người biết khi nào phải viết. Ngoài các triggers khách quan này, bất kỳ bên liên quan nào cũng có thể yêu cầu thực hiện postmortem cho một sự kiện.

Postmortem không đổ lỗi (blameless) là một tín điều của văn hóa SRE. Để thực sự không đổ lỗi, postmortem phải tập trung vào việc xác định các nguyên nhân đóng góp của incident, thay vì kết án bất kỳ cá nhân hay đội nào vì hành vi sai hoặc không phù hợp. Một postmortem viết theo cách không đổ lỗi giả định rằng mọi người tham gia incident đều có thiện chí tốt và đã làm đúng với thông tin mà họ có. Nếu văn hóa hay chỉ trích, khiến cá nhân hoặc đội phải xấu hổ vì đã làm "sai", thì mọi người sẽ không đưa vấn đề ra ánh sáng vì sợ bị trừng phạt.

Văn hóa không đổ lỗi bắt nguồn từ ngành y tế và hàng không, những lĩnh vực mà sai lầm có thể gây chết người. Tại đây, mỗi "sai lầm" được xem như một cơ hội để củng cố hệ thống. Khi postmortem chuyển từ việc phân bổ lỗi sang việc điều tra các nguyên nhân có hệ thống khiến cá nhân hay đội nhóm có thông tin không đầy đủ hoặc không chính xác, các biện pháp phòng ngừa hiệu quả mới có thể được đưa vào. Bạn không thể "sửa" con người, nhưng có thể sửa các hệ thống và quy trình để hỗ trợ tốt hơn cho con người đưa ra lựa chọn đúng khi thiết kế và duy trì các hệ thống phức tạp.

Khi một sự cố thực sự xảy ra, postmortem không được viết theo kiểu thủ tục hình thức rồi bị lãng quên. Thay vào đó, kỹ sư coi postmortem là cơ hội không chỉ để vá một điểm yếu, mà còn để giúp Google nói chung trở nên chống chịu (resilient) hơn. Dù postmortem không đổ lỗi không đơn thuần là nơi để xả bực dọc bằng cách chỉ trích, nhưng nó *nên* chỉ ra nơi và cách các dịch vụ có thể được cải thiện. Dưới đây là hai ví dụ:

#### Chỉ trích (Pointing fingers)

"Chúng tôi cần viết lại toàn bộ hệ thống backend phức tạp này! Nó đã hỏng hàng tuần trong ba quý qua và tôi chắc tất cả chúng ta đều chán ngán việc vá víu từng chút một. Nói thật, nếu bị gọi trực (page) thêm một lần nữa tôi sẽ tự viết lại nó…"

#### Không đổ lỗi (Blameless)

Một mục hành động để viết lại toàn bộ hệ thống backend thực sự có thể ngăn những lần gọi trực phiền toái này tiếp diễn, và cuốn sổ tay bảo trì cho phiên bản này khá dài, rất khó để đào tạo đầy đủ. Tôi chắc những on-caller (người trực) trong tương lai sẽ cảm ơn chúng ta!

### Thực hành Tốt nhất: Tránh Đổ lỗi và Giữ nó Xây dựng (Avoid Blame and Keep It Constructive)

Postmortem không đổ lỗi có thể khó viết, vì định dạng này chỉ rõ các hành động đã dẫn đến incident. Loại bỏ sự đổ lỗi khỏi postmortem giúp mọi người tự tin leo thang (escalate) vấn đề mà không lo sợ. Cũng quan trọng là không kỳ thị việc một người hay đội thường xuyên viết postmortem. Một bầu không khí đổ lỗi có nguy cơ tạo ra văn hóa nơi các incident và vấn đề bị che giấu, dẫn đến rủi ro lớn hơn cho tổ chức [[Boy13]](https://sre.google/sre-book/bibliography#Boy13).

## Hợp tác và Chia sẻ Kiến thức (Collaborate and Share Knowledge)

Chúng tôi coi trọng sự hợp tác, và quy trình postmortem cũng không phải ngoại lệ. Luồng công việc postmortem đòi hỏi sự hợp tác và chia sẻ kiến thức ở mọi giai đoạn.

Chúng tôi dùng Google Docs cho tài liệu postmortem, kèm theo một template nội bộ (xem [Example Postmortem](https://sre.google/sre-book/example-postmortem/)). Dù bạn chọn công cụ nào, hãy đảm bảo nó có các tính năng chính sau:

#### Hợp tác thời gian thực (Real-time collaboration)

Cho phép việc thu thập dữ liệu và ý tưởng nhanh chóng. Thiết yếu trong giai đoạn tạo ban đầu của một postmortem.

#### Một hệ thống bình luận/ghi chú mở (An open commenting/annotation system)

Làm cho việc crowdsourcing (huy động đám đông) các giải pháp dễ dàng và cải thiện phạm vi bao phủ.

#### Thông báo email (Email notifications)

Có thể được chỉ định đến các cộng tác viên trong tài liệu hoặc được sử dụng để kéo người khác vào vòng để cung cấp input (đầu vào).

Viết postmortem cũng liên quan đến việc xem xét và phát hành chính thức. Trong thực tế, các đội chia sẻ bản thảo postmortem trong nội bộ trước và nhờ một nhóm kỹ sư cấp cao đánh giá mức độ đầy đủ của bản thảo. Các tiêu chí xem xét có thể bao gồm:

-   Dữ liệu incident chính có được thu thập để lưu trữ cho hậu thế không?
-   Các đánh giá tác động có đầy đủ không?
-   Nguyên nhân gốc rễ có đủ sâu không?
-   Kế hoạch hành động có phù hợp và các bug fix tương ứng có ở mức ưu tiên phù hợp không?
-   Chúng tôi có chia sẻ kết quả với các bên liên quan thích hợp không?

Sau khi rà soát ban đầu hoàn tất, postmortem được chia sẻ rộng rãi hơn, thường là với đội kỹ thuật lớn hơn hoặc trên danh sách email nội bộ. Mục tiêu của chúng tôi là đưa postmortem đến nhóm độc giả rộng nhất có thể hưởng lợi từ kiến thức hay bài học trong đó. Google có các quy tắc nghiêm ngặt về việc truy cập bất kỳ thông tin nào có thể xác định một người dùng cuối,<sup>[1](#fn1)</sup> và ngay cả tài liệu nội bộ như postmortem cũng không bao giờ chứa thông tin như vậy.

### Thực hành Tốt nhất: Không có Postmortem nào Bị bỏ qua Không được Xem xét (No Postmortem Left Unreviewed)

Một postmortem chưa được xem xét thì coi như không tồn tại. Để đảm bảo mọi bản thảo hoàn thành đều được xem xét, chúng tôi khuyến khích tổ chức các phiên xem xét postmortem định kỳ. Trong những cuộc họp này, việc chốt các cuộc thảo luận, bình luận đang dang dở, nắm bắt ý tưởng và hoàn thiện trạng thái là rất quan trọng.

Khi các bên liên quan đã hài lòng với tài liệu và các mục hành động, postmortem sẽ được đưa vào kho lưu trữ (repository) nội bộ của đội hoặc tổ chức, nơi tập hợp các incident trong quá khứ.<sup>[2](#fn2)</sup> Việc chia sẻ minh bạch giúp người khác dễ dàng tìm kiếm và học hỏi từ postmortem hơn.

## Giới thiệu một Văn hóa Postmortem (Introducing a Postmortem Culture)

Việc đưa văn hóa postmortem vào tổ chức của bạn dễ nói hơn làm; một nỗ lực như vậy đòi hỏi việc nuôi dưỡng và củng cố liên tục. Chúng tôi củng cố một văn hóa postmortem hợp tác thông qua sự tham gia tích cực của ban lãnh đạo cấp cao trong quy trình xem xét và hợp tác. Ban quản lý có thể khuyến khích văn hóa này, nhưng postmortem không đổ lỗi, lý tưởng nhất, là sản phẩm của sự tự thúc đẩy từ chính kỹ sư. Trong tinh thần nuôi dưỡng văn hóa postmortem, các SRE chủ động tạo ra những hoạt động phổ biến hóa những gì chúng tôi học được về hạ tầng hệ thống. Một số ví dụ về hoạt động bao gồm:

#### Postmortem của tháng (Postmortem of the month)

Trong một bản tin hàng tháng, một postmortem thú vị và được viết tốt được chia sẻ với toàn bộ tổ chức.

#### Nhóm postmortem Google+ (Google+ postmortem group)

Nhóm này chia sẻ và thảo luận các postmortem nội bộ và bên ngoài, các thực hành tốt nhất, và các bình luận về postmortem.

#### Câu lạc bộ đọc postmortem (Postmortem reading clubs)

Các đội tổ chức câu lạc bộ đọc postmortem định kỳ, nơi một postmortem thú vị hoặc có tác động được mang ra bàn (kèm một vài đồ uống ngon) cho cuộc đối thoại mở giữa những người có liên quan, những người không liên quan và các Googler (nhân viên Google) mới, về điều gì đã xảy ra, bài học mà incident truyền tải và hậu quả của nó. Thường, postmortem được xem xét đã có từ nhiều tháng hoặc nhiều năm trước!

#### Bánh xe Bất hạnh (Wheel of Misfortune)

Các SRE mới thường được “thưởng thức” bài tập Wheel of Misfortune (xem [Disaster Role Playing](https://sre.google/sre-book/accelerating-sre-on-call#xref_training_disaster-rpg)), trong đó một postmortem trước đó được diễn lại với một dàn kỹ sư đóng các vai trò như được nêu trong postmortem. Người chỉ huy sự cố ban đầu tham gia để trải nghiệm “thật” nhất có thể.

Một trong những thách thức lớn nhất khi đưa postmortem vào một tổ chức là một số người có thể hoài nghi về giá trị của chúng khi cân nhắc chi phí chuẩn bị. Các chiến lược sau có thể giúp đối mặt với thách thức này:

-   Đưa postmortem vào luồng công việc một cách nhẹ nhàng. Một giai đoạn thử nghiệm với vài postmortem hoàn chỉnh, thành công có thể giúp chứng minh giá trị, đồng thời giúp xác định tiêu chí nào nên khởi tạo một postmortem.
-   Đảm bảo việc viết postmortem hiệu quả là một thực hành được khen ngợi và ăn mừng, cả công khai qua các phương tiện xã hội nêu trước đó lẫn qua quản lý hiệu suất cá nhân và đội.
-   Khuyến khích sự công nhận và tham gia của ban lãnh đạo cấp cao. Ngay cả Larry Page cũng lên tiếng về giá trị cao của postmortem!

### Thực hành Tốt nhất: Khen thưởng Con người một cách Hiển nhiên vì Làm điều Đúng (Visibly Reward People for Doing the Right Thing)

Larry Page và Sergey Brin, hai nhà sáng lập của Google, chủ trì TGIF — cuộc họp toàn thể nhân viên hàng tuần được tổ chức trực tiếp tại trụ sở ở Mountain View, California và phát sóng đến các văn phòng Google trên toàn thế giới. Một kỳ TGIF năm 2014 tập trung vào "Nghệ thuật của Postmortem" (The Art of the Postmortem), với các cuộc thảo luận của SRE về những sự cố có tác động lớn. Một SRE chia sẻ về một bản phát hành mà anh vừa đẩy lên; dù đã kiểm thử kỹ lưỡng, một tương tác bất ngờ đã vô tình làm sập một dịch vụ quan trọng trong bốn phút. Sự cố chỉ kéo dài bốn phút vì SRE có đủ bình tĩnh để hoàn tác thay đổi ngay lập tức, tránh một sự cố ngừng dịch vụ dài và lớn hơn rất nhiều. Kỹ sư này không chỉ nhận được hai peer bonus (thưởng đồng nghiệp)<sup>[3](#fn3)</sup> ngay sau đó để ghi nhận cách xử lý sự cố nhanh nhẹn, bình tĩnh, mà còn nhận được một tràng pháo tay lớn từ khán giả TGIF, gồm cả các nhà sáng lập và hàng nghìn Googler. Ngoài một diễn đàn rõ ràng như vậy, Google có nhiều mạng xã hội nội bộ thúc đẩy lời khen ngợi đồng nghiệp cho các [postmortem](https://sre.google/sre-book/managing-incidents/) được viết tốt và cách xử lý sự cố xuất sắc. Đây là một trong nhiều ví dụ mà sự công nhận cho những đóng góp này đến từ đồng nghiệp, từ các CEO và tất cả những người ở giữa.<sup>[4](#fn4)</sup>

### Thực hành Tốt nhất: Yêu cầu Phản hồi về Hiệu quả Postmortem (Ask for Feedback on Postmortem Effectiveness)

Tại Google, chúng tôi chủ động giải quyết vấn đề ngay khi nó xuất hiện và chia sẻ các đổi mới nội bộ. Chúng tôi thường xuyên khảo sát các nhóm để xem quy trình postmortem đang hỗ trợ mục tiêu của họ như thế nào và có thể cải thiện ra sao. Các câu hỏi chúng tôi đặt ra bao gồm: Văn hóa có đang hỗ trợ công việc của bạn không? Việc viết postmortem có kèm theo quá nhiều toil (công việc nhàm chán, thủ công) không (xem [Eliminating Toil](https://sre.google/sre-book/eliminating-toil/))? Thực hành tốt nhất nào mà nhóm bạn khuyến nghị cho các nhóm khác? Loại công cụ nào bạn muốn thấy được phát triển? Kết quả khảo sát cho các SRE tuyến đầu cơ hội yêu cầu các cải tiến nhằm nâng cao hiệu quả của văn hóa postmortem.

Ngoài các khía cạnh vận hành của quản lý incident và theo dõi, thực hành postmortem đã ăn sâu vào văn hóa tại Google: giờ đây, việc bất kỳ incident đáng kể nào cũng được theo sau bởi một postmortem toàn diện đã trở thành một chuẩn mực văn hóa.

## Kết luận và Những Cải tiến Liên tục (Conclusion and Ongoing Improvements)

Nhờ liên tục đầu tư vào văn hóa postmortem, chúng tôi có thể tự tin khẳng định rằng Google gặp ít sự cố ngừng dịch vụ hơn và mang lại trải nghiệm người dùng tốt hơn. Nhóm “Postmortems at Google” là minh chứng cho cam kết của chúng tôi đối với văn hóa postmortem không đổ lỗi. Nhóm này điều phối các hoạt động postmortem trên toàn công ty: tổng hợp các mẫu postmortem, tự động hóa việc tạo postmortem từ dữ liệu của các công cụ dùng trong sự cố, và hỗ trợ tự động hóa việc trích xuất dữ liệu từ postmortem để chúng tôi có thể phân tích xu hướng. Chúng tôi đã hợp tác để chia sẻ các thực hành tốt nhất từ nhiều sản phẩm khác nhau như YouTube, Google Fiber, Gmail, Google Cloud, AdWords và Google Maps. Dù những sản phẩm này khá đa dạng, tất cả đều thực hiện postmortem với cùng một mục tiêu phổ quát: học từ những khoảnh khắc tối tăm nhất.

Mỗi tháng, Google tạo ra một lượng lớn postmortem, khiến các công cụ tổng hợp chúng ngày càng hữu ích. Nhờ đó, chúng tôi có thể xác định các chủ đề chung và những lĩnh vực cần cải thiện xuyên suốt các ranh giới sản phẩm. Để hỗ trợ việc hiểu và phân tích tự động, gần đây chúng tôi đã nâng cấp template postmortem (xem [Example Postmortem](https://sre.google/sre-book/example-postmortem/)) bằng cách thêm các trường metadata (dữ liệu mô tả). Công việc tương lai trong lĩnh vực này bao gồm machine learning (học máy) nhằm giúp dự đoán các điểm yếu, tạo thuận lợi cho việc điều tra incident thời gian thực, và giảm các incident trùng lặp.

<a id="fn1"></a>[1](#fn1) Xem [*https://www.google.com/policies/privacy/*](https://www.google.com/policies/privacy/).

<a id="fn2"></a>[2](#fn2) Nếu bạn muốn bắt đầu kho lưu trữ của riêng mình, Etsy đã phát hành [*Morgue*](https://github.com/etsy/morgue), một công cụ để quản lý postmortem.

<a id="fn3"></a>[3](#fn3) Chương trình Peer Bonus (Thưởng đồng nghiệp) của Google là một cách để các Googler đồng nghiệp công nhận các đồng nghiệp vì những nỗ lực xuất sắc và bao gồm một phần thưởng tiền mặt tượng trưng.

<a id="fn4"></a>[4](#fn4) Để có thêm thảo luận về incident cụ thể này, xem [Emergency Response](https://sre.google/sre-book/emergency-response/).

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
