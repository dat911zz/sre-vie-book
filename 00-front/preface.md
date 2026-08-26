# Lời nói đầu (Preface)

> **Nguyên bản:** [Preface](https://sre.google/sre-book/preface/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

Kỹ thuật phần mềm (software engineering) có điểm chung với việc sinh con: sự vất vả *trước* khi sinh là đau đớn và khó khăn, nhưng phần lớn nỗ lực thực sự đến *sau* khi sinh. Thế nhưng, kỹ thuật phần mềm như một ngành lại dành nhiều thời gian hơn để nói về giai đoạn đầu, dù các ước tính cho rằng 40–90% tổng chi phí của một hệ thống phát sinh sau khi "sinh ra".<sup>[1](#fn1)</sup> Mô hình phổ biến trong ngành coi phần mềm đã triển khai, đang vận hành là đã "ổn định" trong môi trường production và do đó cần ít sự chú ý hơn từ các kỹ sư phần mềm, là sai lầm. Qua lăng kính đó, chúng ta thấy rằng nếu kỹ thuật phần mềm có xu hướng tập trung vào thiết kế và xây dựng các hệ thống phần mềm, thì phải có một ngành khác lo *toàn bộ* vòng đời của các đối tượng phần mềm, từ lúc khởi đầu, qua triển khai và vận hành, tinh chỉnh, và cuối cùng là decommissioning (ngừng vận hành) một cách bình yên. Ngành này sử dụng — và cần phải sử dụng — một phạm vi kỹ năng rộng, nhưng có những mối quan tâm riêng biệt so với các loại kỹ sư khác. Ngày nay, câu trả lời của chúng ta là ngành mà Google gọi là Site Reliability Engineering (Kỹ thuật Độ tin cậy Trang web).

Vậy chính xác thì SRE là gì? Chúng tôi thừa nhận rằng đó không phải là một cái tên đặc biệt rõ ràng cho những gì chúng tôi làm — hầu như mọi site reliability engineer tại Google đều thường xuyên bị hỏi rằng chính xác đó là gì, và họ thực sự làm gì.

Giải mã từ ngữ một chút: trước hết và quan trọng nhất, SRE là *kỹ sư*. Chúng tôi áp dụng các nguyên lý của khoa học máy tính và kỹ thuật vào việc thiết kế và phát triển các hệ thống máy tính: thường là các hệ thống phân tán (distributed system) lớn. Đôi khi, nhiệm vụ của chúng tôi là viết phần mềm cho những hệ thống đó cùng với đồng nghiệp phát triển sản phẩm; đôi khi là xây dựng tất cả các phần bổ sung mà những hệ thống đó cần, như backup (sao lưu) hoặc load balancing (cân bằng tải), lý tưởng là để chúng có thể được tái sử dụng giữa các hệ thống; và đôi khi là tìm ra cách áp dụng các giải pháp hiện có vào các vấn đề mới.

Tiếp theo, chúng tôi tập trung vào *độ tin cậy* (reliability) của hệ thống. Ben Treynor Sloss, VP của Google phụ trách 24/7 Operations, người khởi xướng thuật ngữ SRE, tuyên bố rằng độ tin cậy là tính năng cơ bản nhất của bất kỳ sản phẩm nào: một hệ thống không thực sự hữu ích nếu không ai có thể sử dụng nó! Vì độ tin cậy<sup>[2](#fn2)</sup> quan trọng đến vậy, các SRE tập trung vào việc tìm ra các cách cải thiện thiết kế và vận hành hệ thống để làm cho chúng scale (mở rộng) hơn, đáng tin cậy hơn và hiệu quả hơn. Tuy nhiên, chúng tôi chỉ bỏ công sức theo hướng này đến một mức độ nhất định: khi các hệ thống "đáng tin cậy đủ", chúng tôi thay vào đó đầu tư nỗ lực vào việc thêm tính năng hoặc xây dựng sản phẩm mới.<sup>[3](#fn3)</sup>

Cuối cùng, các SRE tập trung vào việc vận hành các *dịch vụ* (services) được xây dựng trên các hệ thống máy tính phân tán của chúng tôi, bất kể những dịch vụ đó là lưu trữ ở quy mô hành tinh, email cho hàng trăm triệu người dùng, hay nơi Google bắt đầu — tìm kiếm web. "Site" trong tên của chúng tôi ban đầu ám chỉ vai trò của SRE trong việc giữ cho trang web *google.com* hoạt động, mặc dù giờ đây chúng tôi vận hành nhiều dịch vụ hơn, nhiều trong số đó không phải là website — từ cơ sở hạ tầng nội bộ như Bigtable đến các sản phẩm cho nhà phát triển bên ngoài như Google Cloud Platform.

Mặc dù chúng tôi đã thể hiện SRE như một ngành rộng, không có gì ngạc nhiên khi nó ra đời trong thế giới dịch vụ web đầy biến động, và có lẽ về nguồn gốc có phần nào chịu ảnh hưởng của những đặc thù của cơ sở hạ tầng của chúng tôi. Cũng không có gì ngạc nhiên khi, trong số các đặc điểm của phần mềm sau khi triển khai mà chúng tôi có thể chọn để chú trọng, độ tin cậy là thứ chúng tôi coi là ưu tiên hàng đầu.<sup>[4](#fn4)</sup> Lĩnh vực dịch vụ web — vừa vì quy trình cải thiện và thay đổi phần mềm phía server tương đối giới hạn, vừa vì việc quản lý sự thay đổi bản thân nó gắn chặt với các sự cố dưới mọi hình thức — là một nền tảng tự nhiên mà từ đó cách tiếp cận của chúng tôi có thể xuất hiện.

Mặc dù ra đời tại Google, và trong cộng đồng web nói chung, chúng tôi cho rằng ngành này có những bài học áp dụng được cho các cộng đồng và tổ chức khác. Quyển sách này là một nỗ lực để giải thích cách chúng tôi làm việc: cả để các tổ chức khác có thể sử dụng những gì chúng tôi đã học, và để chúng tôi có thể định nghĩa rõ hơn vai trò và ý nghĩa của thuật ngữ này. Để đạt được điều đó, chúng tôi đã tổ chức sách sao cho các nguyên lý chung và các thực hành cụ thể hơn được tách ra khi có thể, và khi phù hợp để thảo luận một chủ đề cụ thể với thông tin đặc thù Google, chúng tôi tin rằng người đọc sẽ thông cảm cho chúng tôi và không ngại rút ra những kết luận hữu ích cho môi trường của chính họ.

Chúng tôi cũng cung cấp một số tài liệu định hướng — một mô tả về môi trường production của Google và một bản đối chiếu giữa một số phần mềm nội bộ của chúng tôi và phần mềm công khai có sẵn — điều này nên giúp đặt ngữ cảnh cho những gì chúng tôi đang nói và làm cho nó trực tiếp hơn.

Cuối cùng, dĩ nhiên, phần mềm và kỹ thuật hệ thống định hướng độ tin cậy hơn vốn dĩ là tốt. Tuy nhiên, chúng tôi thừa nhận rằng các tổ chức nhỏ hơn có thể đang tự hỏi làm thế nào để sử dụng tốt nhất kinh nghiệm được thể hiện ở đây: giống như bảo mật (security), bạn càng quan tâm đến độ tin cậy sớm bao nhiêu thì càng tốt. Điều này có nghĩa là dù một tổ chức nhỏ có nhiều mối quan tâm cấp bách và các lựa chọn phần mềm bạn đưa ra có thể khác với những gì Google đã chọn, vẫn đáng để đặt nền tảng hỗ trợ độ tin cậy nhẹ nhàng từ sớm, vì chi phí để mở rộng một cấu trúc sau này thấp hơn so với việc phải giới thiệu một cấu trúc chưa tồn tại. Phần [Management](https://sre.google/sre-book/part-IV-management/) chứa một số best practices (thực hành tốt nhất) về đào tạo, giao tiếp và họp mà chúng tôi thấy hoạt động tốt, nhiều trong số đó tổ chức của bạn có thể sử dụng ngay.

Nhưng với các quy mô nằm giữa một startup và một công ty đa quốc gia, có thể đã có ai đó trong tổ chức của bạn đang làm công việc SRE, dù chưa nhất thiết được gọi bằng cái tên đó hay được công nhận như vậy. Một cách khác để bắt đầu cải thiện độ tin cậy cho tổ chức của bạn là chính thức công nhận công việc đó, hoặc tìm ra những người này và nuôi dưỡng những gì họ làm — thưởng cho họ. Họ là những người đứng ở ranh giới giữa một cách nhìn thế giới và một cách nhìn khác: giống như Newton, người đôi khi được gọi không phải là nhà vật lý đầu tiên của thế giới, mà là nhà giả kim cuối cùng của thế giới.

Và xét từ góc nhìn lịch sử, ai, khi nhìn lại, có thể là SRE đầu tiên?

Chúng tôi thích nghĩ rằng Margaret Hamilton, người làm việc trong chương trình Apollo với tư cách nhân sự do MIT cử sang, có tất cả các đặc điểm quan trọng của một SRE đầu tiên.<sup>[5](#fn5)</sup> Theo chính lời bà, "một phần của văn hóa là học hỏi từ tất cả mọi người và mọi thứ, kể cả từ những điều mà bạn ít ngờ tới nhất."

Một trường hợp điển hình là khi con gái út Lauren của bà đến làm việc cùng bà một ngày, trong khi một số thành viên nhóm đang chạy các kịch bản nhiệm vụ trên máy tính mô phỏng hybrid. Như trẻ con thường làm, Lauren đi khám phá, và cô bé đã khiến một "nhiệm vụ" bị crash (sập) bằng cách chọn các phím DSKY theo một cách không ngờ tới. Điều này cảnh báo cho nhóm biết điều gì sẽ xảy ra nếu chương trình trước khi phóng, P01, bị một phi hành gia thực sự vô tình chọn trong một nhiệm vụ thực sự, trong giai đoạn điều chỉnh quỹ đạo thực sự. (Việc vô tình phóng P01 trong một nhiệm vụ thực sự sẽ là một vấn đề lớn, vì nó xóa bỏ dữ liệu định hướng, và máy tính không được trang bị để điều khiển tàu vũ trụ mà không có dữ liệu định hướng.)

Với bản năng của một SRE, Margaret đã nộp một yêu cầu thay đổi chương trình để thêm code kiểm tra lỗi đặc biệt trong phần mềm bay trên tàu, phòng trường hợp một phi hành gia vô tình chọn P01 trong khi bay. Nhưng việc này được "cấp trên" tại NASA coi là không cần thiết: tất nhiên, điều đó không bao giờ xảy ra! Vì vậy, thay vì thêm code kiểm tra lỗi, Margaret đã cập nhật tài liệu quy cách nhiệm vụ với nội dung tương đương "Không chọn P01 trong khi bay." (Rõ ràng sự cập nhật này gây buồn cười cho nhiều người trong dự án, những người đã được nói nhiều lần rằng các phi hành gia sẽ không mắc bất kỳ sai lầm nào — sau cùng, họ được đào tạo để hoàn hảo.)

Chà, biện pháp an toàn do Margaret đề xuất chỉ bị coi là không cần thiết cho đến chính nhiệm vụ tiếp theo, trên Apollo 8, chỉ vài ngày sau sự cập nhật quy cách. Trong giai đoạn điều chỉnh quỹ đạo của ngày bay thứ tư, với các phi hành gia Jim Lovell, William Anders và Frank Borman trên tàu, Jim Lovell đã chọn nhầm P01 — tình cờ, vào ngày Giáng sinh — gây ra nhiều hỗn loạn cho tất cả những người liên quan. Đây là một vấn đề nghiêm trọng, vì khi không có giải pháp thay thế, việc thiếu dữ liệu định hướng có nghĩa là các phi hành gia sẽ không bao giờ quay về nhà. May mắn thay, sự cập nhật tài liệu đã nêu rõ khả năng này, và vô giá trong việc tìm ra cách tải lên dữ liệu khả dụng và khôi phục nhiệm vụ, trong khi còn rất ít thời gian.

Như Margaret nói, "sự hiểu biết kỹ lưỡng về cách vận hành các hệ thống là chưa đủ để ngăn chặn các sai lầm của con người", và yêu cầu thay đổi để thêm phần mềm phát hiện và khôi phục lỗi vào chương trình trước khi phóng P01 đã được phê duyệt ngay sau đó.

Mặc dù sự cố Apollo 8 xảy ra từ nhiều thập kỷ trước, có nhiều điều trong các đoạn văn trước đó liên quan trực tiếp đến đời sống của các kỹ sư ngày nay, và nhiều điều sẽ tiếp tục liên quan trực tiếp trong tương lai. Do đó, đối với các hệ thống bạn chăm sóc, đối với các nhóm bạn làm việc, hoặc đối với các tổ chức bạn đang xây dựng, vui lòng hãy ghi nhớ SRE Way (Đạo SRE): sự kỹ lưỡng và tận tụy, niềm tin vào giá trị của sự chuẩn bị và tài liệu, và nhận thức về những gì có thể sai, kết hợp với mong muốn mạnh mẽ để ngăn chặn nó. Chào mừng bạn đến với nghề nghiệp đang hình thành của chúng tôi!

### Cách đọc quyển sách này

Quyển sách này là một loạt các bài luận do các thành viên và cựu thành viên của tổ chức Site Reliability Engineering của Google viết. Nó giống với một tập bài trình diễn tại hội nghị hơn là một quyển sách tiêu chuẩn của một tác giả hoặc một số ít tác giả. Mỗi chương được dự định đọc như một phần của một tổng thể nhất quán, nhưng bạn vẫn có thể thu được nhiều điều đáng kể từ bất kỳ chủ đề nào đặc biệt thu hút bạn. (Nếu có các bài viết khác hỗ trợ hoặc cung cấp thông tin cho văn bản, chúng tôi trích dẫn chúng để bạn có thể theo dõi.)

Bạn không cần đọc theo bất kỳ trình tự nào, mặc dù chúng tôi đề xuất ít nhất bắt đầu với chương [The Production Environment at Google, from the Viewpoint of an SRE](https://sre.google/sre-book/production-environment/) và [Embracing Risk](https://sre.google/sre-book/embracing-risk/), lần lượt mô tả môi trường production của Google và phác thảo cách SRE tiếp cận rủi ro. (Rủi ro, theo nhiều cách, là chất lượng then chốt của nghề nghiệp của chúng tôi.) Đọc từ bìa này sang bìa kia, dĩ nhiên, cũng hữu ích và khả thi; các chương của chúng tôi được nhóm theo chủ đề, vào Principles ([Principles](https://sre.google/sre-book/part-II-principles/)), Practices ([Practices](https://sre.google/sre-book/part-III-practices/)), và Management ([Management](https://sre.google/sre-book/part-IV-management/)). Mỗi phần có một phần giới thiệu nhỏ nổi bật những phần riêng lẻ là về gì, và trích dẫn các bài viết khác do Google SREs xuất bản, bao phủ các chủ đề cụ thể chi tiết hơn. Ngoài ra, trang web đồng hành với quyển sách này, [*https://g.co/SREBook*](https://g.co/SREBook), có một số tài nguyên hữu ích.

Chúng tôi hy vọng rằng quyển sách này sẽ ít nhất cũng hữu ích và thú vị đối với bạn như việc tạo ra nó đã dành cho chúng tôi.

 — Các biên tập viên

## Các quy ước sử dụng trong quyển sách này

Các quy ước typography sau đây được sử dụng trong quyển sách này:

*Chữ in nghiêng*

Chỉ các thuật ngữ mới, URL, địa chỉ email, tên file, và phần mở rộng file.

`Constant width`

Được sử dụng cho các danh sách chương trình, cũng như trong các đoạn văn để tham chiếu đến các phần tử chương trình như tên biến hoặc tên hàm, cơ sở dữ liệu, kiểu dữ liệu, biến môi trường, phát biểu, và từ khóa.

`**Constant width bold**`

Chỉ các lệnh hoặc văn bản khác nên được người dùng nhập theo đúng như vậy.

*`Constant width italic`*

Chỉ văn bản nên được thay thế bằng các giá trị do người dùng cung cấp hoặc bởi các giá trị được xác định bởi ngữ cảnh.

### Gợi ý (Tip)

Thành phần này biểu thị một gợi ý hoặc đề xuất.

### Ghi chú (Note)

Thành phần này biểu thị một ghi chú chung.

### Cảnh báo (Warning)

Thành phần này biểu thị một cảnh báo hoặc thận trọng.

## Sử dụng ví dụ code

Tài liệu bổ sung có sẵn tại [*https://g.co/SREBook*](https://g.co/SREBook).

Quyển sách này ở đây để giúp bạn hoàn thành công việc của mình. Nhìn chung, nếu có code mẫu đi kèm với quyển sách này, bạn có thể sử dụng nó trong các chương trình và tài liệu của bạn. Bạn không cần liên hệ với chúng tôi xin phép trừ khi bạn tái tạo một phần đáng kể của code. Ví dụ, viết một chương trình sử dụng một số đoạn code từ quyển sách này không cần xin phép. Bán hoặc phân phối một CD-ROM chứa các ví dụ từ các quyển sách của O'Reilly cần xin phép. Trả lời một câu hỏi bằng cách trích dẫn quyển sách này và trích dẫn code mẫu không cần xin phép. Tích hợp một lượng đáng kể code mẫu từ quyển sách này vào tài liệu sản phẩm của bạn cần xin phép.

Chúng tôi đánh giá cao, nhưng không yêu cầu, việc ghi nhận. Một lời ghi nhận thường bao gồm tiêu đề, tác giả, nhà xuất bản, và ISBN. Ví dụ: "*Site Reliability Engineering*, edited by Betsy Beyer, Chris Jones, Jennifer Petoff, and Niall Richard Murphy (O'Reilly). Copyright 2016 Google, Inc., 978-1-491-92912-4."

Nếu bạn cảm thấy việc sử dụng code mẫu của bạn nằm ngoài fair use (sử dụng hợp lý) hoặc quyền cho phép ở trên, vui lòng liên hệ với chúng tôi tại [*permissions@oreilly.com*](mailto:permissions@oreilly.com).

## Safari® Books Online

### Ghi chú (Note)

[*Safari Books Online*](https://safaribooksonline.com) là một thư viện kỹ thuật số on-demand (theo yêu cầu) cung cấp nội dung chuyên gia dưới dạng sách và video từ các tác giả hàng đầu thế giới về công nghệ và kinh doanh.

Các chuyên gia công nghệ, nhà phát triển phần mềm, nhà thiết kế web, và các chuyên gia kinh doanh và sáng tạo sử dụng Safari Books Online như tài nguyên chính cho nghiên cứu, giải quyết vấn đề, học tập, và đào tạo chứng chỉ.

Safari Books Online cung cấp nhiều [kế hoạch và giá](https://www.safaribooksonline.com/pricing/) cho [doanh nghiệp](https://www.safaribooksonline.com/enterprise/), [chính phủ](https://www.safaribooksonline.com/government/), [giáo dục](https://learning.oreilly.com/), và cá nhân.

Các thành viên có quyền truy cập vào hàng nghìn sách, video đào tạo, và bản thảo trước khi xuất bản trong một cơ sở dữ liệu tìm kiếm đầy đủ từ các nhà xuất bản như O'Reilly Media, Prentice Hall Professional, Addison-Wesley Professional, Microsoft Press, Sams, Que, Peachpit Press, Focal Press, Cisco Press, John Wiley & Sons, Syngress, Morgan Kaufmann, IBM Redbooks, Packt, Adobe Press, FT Press, Apress, Manning, New Riders, McGraw-Hill, Jones & Bartlett, Course Technology, và hàng trăm [nữa](https://learning.oreilly.com/). Để biết thêm thông tin về Safari Books Online, vui lòng truy cập [online](https://safaribooksonline.com).

## Liên hệ với chúng tôi

Vui lòng gửi bình luận và câu hỏi về quyển sách này đến nhà xuất bản:

-   O'Reilly Media, Inc.
-   1005 Gravenstein Highway North
-   Sebastopol, CA 95472
-   800-998-9938 (tại Hoa Kỳ hoặc Canada)
-   707-829-0515 (quốc tế hoặc địa phương)
-   707-829-0104 (fax)

Chúng tôi có một trang web cho quyển sách này, nơi chúng tôi liệt kê các lỗi, ví dụ, và bất kỳ thông tin bổ sung nào. Bạn có thể truy cập trang này tại [*https://bit.ly/site-reliability-engineering*](https://bit.ly/site-reliability-engineering).

Để bình luận hoặc hỏi các câu hỏi kỹ thuật về quyển sách này, gửi email đến [*bookquestions@oreilly.com*](mailto:bookquestions@oreilly.com).

Để biết thêm thông tin về các sách, khóa học, hội nghị, và tin tức của chúng tôi, xem trang web của chúng tôi tại [*https://www.oreilly.com*](https://www.oreilly.com).

Tìm chúng tôi trên Facebook: [*https://facebook.com/oreilly*](https://facebook.com/oreilly)

Theo dõi chúng tôi trên Twitter: [*https://twitter.com/oreillymedia*](https://twitter.com/oreillymedia)

Xem chúng tôi trên YouTube: [*https://www.youtube.com/oreillymedia*](https://www.youtube.com/oreillymedia)

## Lời cảm tạ

Quyển sách này sẽ không thể có được nếu không có những nỗ lực không biết mệt mỏi của các tác giả và nhà văn kỹ thuật của chúng tôi. Chúng tôi cũng muốn cảm ơn các người xem xét nội bộ sau đây đã cung cấp phản hồi đặc biệt quý giá: Alex Matey, Dermot Duffy, JC van Winkel, John T. Reese, Michael O'Reilly, Steve Carstensen, và Todd Underwood. Ben Lutch và Ben Treynor Sloss là các nhà bảo trợ của quyển sách này trong Google; niềm tin của họ vào dự án này và chia sẻ những gì chúng tôi đã học về việc vận hành các dịch vụ quy mô lớn là thiết yếu để tạo ra quyển sách này.

Chúng tôi muốn gửi lời cảm ơn đặc biệt đến Rik Farrow, biên tập viên của *;login:*, vì đã hợp tác với chúng tôi trong một số đóng góp trước khi xuất bản thông qua USENIX.

Trong khi các tác giả được ghi nhận cụ thể trong mỗi chương, chúng tôi muốn dành thời gian để công nhận những người đã đóng góp cho mỗi chương bằng cách cung cấp đầu vào có suy nghĩ, thảo luận, và xem xét.

[Embracing Risk](https://sre.google/sre-book/embracing-risk/): Abe Rahey, Ben Treynor Sloss, Brian Stoler, Dave O'Connor, David Besbris, Jill Alvidrez, Mike Curtis, Nancy Chang, Tammy Capistrant, Tom Limoncelli

[Eliminating Toil](https://sre.google/sre-book/eliminating-toil/): Cody Smith, George Sadlier, Laurence Berland, Marc Alvidrez, Patrick Stahlberg, Peter Duff, Pim van Pelt, Ryan Anderson, Sabrina Farmer, Seth Hettich

[Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/): Mike Curtis, Jamie Wilkinson, Seth Hettich

[Release Engineering](https://sre.google/sre-book/release-engineering/): David Schnur, JT Goldstone, Marc Alvidrez, Marcus Lara-Reinhold, Noah Maxwell, Peter Dinges, Sumitran Raghunathan, Yutong Cho

[Simplicity](https://sre.google/sre-book/simplicity/): Ryan Anderson

[Practical Alerting from Time-Series Data](https://sre.google/sre-book/practical-alerting/): Jules Anderson, Max Luebbe, Mikel Mcdaniel, Raul Vera, Seth Hettich

[Being On-Call](https://sre.google/sre-book/being-on-call/): Andrew Stribblehill, Richard Woodbury

[Effective Troubleshooting](https://sre.google/sre-book/effective-troubleshooting/): Charles Stephen Gunn, John Hedditch, Peter Nuttall, Rob Ewaschuk, Sam Greenfield

[Emergency Response](https://sre.google/sre-book/emergency-response/): Jelena Oertel, Kripa Krishnan, Sergio Salvi, Tim Craig

[Managing Incidents](https://sre.google/sre-book/managing-incidents/): Amy Zhou, Carla Geisser, Grainne Sheerin, Hildo Biersma, Jelena Oertel, Perry Lorier, Rune Kristian Viken

[Postmortem Culture: Learning from Failure](https://sre.google/sre-book/postmortem-culture/): Dan Wu, Heather Sherman, Jared Brick, Mike Louer, Štěpán Davidovič, Tim Craig

[Tracking Outages](https://sre.google/sre-book/tracking-outages/): Andrew Stribblehill, Richard Woodbury

[Testing for Reliability](https://sre.google/sre-book/testing-reliability/): Isaac Clerencia, Marc Alvidrez

[Software Engineering in SRE](https://sre.google/sre-book/software-engineering-in-sre/): Ulric Longyear

[Load Balancing at the Frontend](https://sre.google/sre-book/load-balancing-frontend/): Debashish Chatterjee, Perry Lorier

Các chương [Load Balancing in the Datacenter](https://sre.google/sre-book/load-balancing-datacenter/) và [Handling Overload](https://sre.google/sre-book/handling-overload/): Adam Fletcher, Christoph Pfisterer, Lukáš Ježek, Manjot Pahwa, Micha Riser, Noah Fiedel, Pavel Herrmann, Paweł Zuzelski, Perry Lorier, Ralf Wildenhues, Tudor-Ioan Salomie, Witold Baryluk

[Addressing Cascading Failures](https://sre.google/sre-book/addressing-cascading-failures/): Mike Curtis, Ryan Anderson

[Managing Critical State: Distributed Consensus for Reliability](https://sre.google/sre-book/managing-critical-state/): Ananth Shrinivas, Mike Burrows

[Distributed Periodic Scheduling with Cron](https://sre.google/sre-book/distributed-periodic-scheduling/): Ben Fried, Derek Jackson, Gabe Krabbe, Laura Nolan, Seth Hettich

[Data Processing Pipelines](https://sre.google/sre-book/data-processing-pipelines/): Abdulrahman Salem, Alex Perry, Arnar Mar Hrafnkelsson, Dieter Pearcey, Dylan Curley, Eivind Eklund, Eric Veach, Graham Poulter, Ingvar Mattsson, John Looney, Ken Grant, Michelle Duffy, Mike Hochberg, Will Robinson

[Data Integrity: What You Read Is What You Wrote](https://sre.google/sre-book/data-integrity/): Corey Vickrey, Dan Ardelean, Disney Luangsisongkham, Gordon Prioreschi, Kristina Bennett, Liang Lin, Michael Kelly, Sergey Ivanyuk

[Reliable Product Launches at Scale](https://sre.google/sre-book/reliable-product-launches/): Vivek Rau

[Accelerating SREs to On-Call and Beyond](https://sre.google/sre-book/accelerating-sre-on-call/): Melissa Binde, Perry Lorier, Preston Yoshioka

[Dealing with Interrupts](https://sre.google/sre-book/dealing-with-interrupts/): Ben Lutch, Carla Geisser, Dzevad Trumic, John Turek, Matt Brown

[Embedding an SRE to Recover from Operational Overload](https://sre.google/sre-book/operational-overload/): Charles Stephen Gunn, Chris Heiser, Max Luebbe, Sam Greenfield

[Communication and Collaboration in SRE](https://sre.google/sre-book/communication-and-collaboration/): Alex Kehlenbeck, Jeromy Carriere, Joel Becker, Sowmya Vijayaraghavan, Trevor Mattson-Hamilton

[The Evolving SRE Engagement Model](https://sre.google/sre-book/evolving-sre-engagement-model/): Seth Hettich

[Lessons Learned from Other Industries](https://sre.google/sre-book/lessons-learned/): Adrian Hilton, Brad Kratochvil, Charles Ballowe, Dan Sheridan, Eddie Kennedy, Erik Gross, Gus Hartmann, Jackson Stone, Jeff Stevenson, John Li, Kevin Greer, Matt Toia, Michael Haynie, Mike Doherty, Peter Dahl, Ron Heiby

Chúng tôi cũng biết ơn các người đóng góp sau đây, những người đã cung cấp tài liệu đáng kể, làm tốt công việc xem xét, đồng ý được phỏng vấn, cung cấp chuyên môn hoặc tài nguyên đáng kể, hoặc có một số hiệu ứng tốt khác cho tác phẩm này:

Abe Hassan, Adam Rogoyski, Alex Hidalgo, Amaya Booker, Andrew Fikes, Andrew Hurst, Ariel Goh, Ashleigh Rentz, Ayman Hourieh, Barclay Osborn, Ben Appleton, Ben Love, Ben Winslow, Bernhard Beck, Bill Duane, Bill Patry, Blair Zajac, Bob Gruber, Brian Gustafson, Bruce Murphy, Buck Clay, Cedric Cellier, Chiho Saito, Chris Carlon, Christopher Hahn, Chris Kennelly, Chris Taylor, Ciara Kamahele-Sanfratello, Colin Phipps, Colm Buckley, Craig Paterson, Daniel Eisenbud, Daniel V. Klein, Daniel Spoonhower, Dan Watson, Dave Phillips, David Hixson, Dina Betser, Doron Meyer, Dmitry Fedoruk, Eric Grosse, Eric Schrock, Filip Zyzniewski, Francis Tang, Gary Arneson, Georgina Wilcox, Gretta Bartels, Gustavo Franco, Harald Wagener, Healfdene Goguen, Hugo Santos, Hyrum Wright, Ian Gulliver, Jakub Turski, James Chivers, James O'Kane, James Youngman, Jan Monsch, Jason Parker-Burlingham, Jason Petsod, Jeffry McNeil, Jeff Dean, Jeff Peck, Jennifer Mace, Jerry Cen, Jess Frame, John Brady, John Gunderman, John Kochmar, John Tobin, Jordyn Buchanan, Joseph Bironas, Julio Merino, Julius Plenz, Kate Ward, Kathy Polizzi, Katrina Sostek, Kenn Hamm, Kirk Russell, Kripa Krishnan, Larry Greenfield, Lea Oliveira, Luca Cittadini, Lucas Pereira, Magnus Ringman, Mahesh Palekar, Marco Paganini, Mario Bonilla, Mathew Mills, Mathew Monroe, Matt D. Brown, Matt Proud, Max Saltonstall, Michal Jaszczyk, Mihai Bivol, Misha Brukman, Olivier Oansaldi, Patrick Bernier, Pierre Palatin, Rob Shanley, Robert van Gent, Rory Ward, Rui Zhang-Shen, Salim Virji, Sanjay Ghemawat, Sarah Coty, Sean Dorward, Sean Quinlan, Sean Sechrist, Shari Trumbo-McHenry, Shawn Morrissey, Shun-Tak Leung, Stan Jedrus, Stefano Lattarini, Steven Schirripa, Tanya Reilly, Terry Bolt, Tim Chaplin, Toby Weingartner, Tom Black, Udi Meiri, Victor Terron, Vlad Grama, Wes Hertlein, and Zoltan Egyed.

Chúng tôi rất trân trọng phản hồi có suy nghĩ và chuyên sâu mà chúng tôi đã nhận được từ các người xem xét bên ngoài: Andrew Fong, Björn Rabenstein, Charles Border, David Blank-Edelman, Frossie Economou, James Meickle, Josh Ryder, Mark Burgess, and Russ Allbery.

Chúng tôi muốn gửi lời cảm ơn đặc biệt đến Cian Synnott, thành viên ban đầu của nhóm sách và đồng âm mưu, người đã rời Google trước khi dự án này hoàn thành nhưng có ảnh hưởng sâu sắc đến nó, và Margaret Hamilton, người đã lịch sự cho phép chúng tôi tham chiếu câu chuyện của bà trong lời nói đầu của chúng tôi. Ngoài ra, chúng tôi muốn gửi lời cảm ơn đặc biệt đến Shylaja Nukala, người đã hào phóng cho đi thời gian của các nhà văn kỹ thuật của bà và ủng hộ trọn vẹn những nỗ lực cần thiết và quý giá của họ.

Các biên tập viên cũng muốn cảm ơn cá nhân các người sau:

Betsy Beyer: Cho bà ngoại (người hùng cá nhân của tôi), vì đã cung cấp vô số cuộc động viên qua điện thoại và bỏng ngô, và cho Riba, vì đã cung cấp cho tôi quần sweatpants cần thiết để cung cấp năng lượng cho nhiều đêm khuya. Những điều này, dĩ nhiên, ngoài dàn diễn viên all-stars của SRE thực sự là những cộng tác viên tuyệt vời.

Chris Jones: Cho Michelle, vì đã cứu tôi khỏi một cuộc đời tội phạm trên biển cả và vì khả năng kỳ lạ của bà trong việc tìm thấy những manzana (phố) ở những nơi không ngờ tới, và cho những người đã dạy tôi về kỹ thuật trong những năm qua.

Jennifer Petoff: Cho chồng tôi Scott vì đã ủng hộ vô cùng trong suốt quy trình hai năm viết quyển sách này và vì đã giữ cho các biên tập viên được cung cấp đủ đường trên "Hòn đảo bánh tráng miệng" của chúng tôi.

Niall Murphy: Cho Léan, Oisín, và Fiachra, những người kiên nhẫn nhiều hơn rất nhiều so với những gì tôi có quyền kỳ vọng với một người cha và chồng nóng nảy hơn bình thường rất nhiều, trong nhiều năm. Cho Dermot, vì lời đề nghị chuyển nhượng.

<a id="fn1"></a>[1](#fn1) Rất nhiều sự biến động trong các ước tính này cho bạn biết điều gì đó về kỹ thuật phần mềm như một ngành, nhưng xem, ví dụ, [[Gla02]](https://sre.google/sre-book/bibliography#Gla02) để biết chi tiết hơn.

<a id="fn2"></a>[2](#fn2) Đối với mục đích của chúng tôi, độ tin cậy là "Xác suất rằng [một hệ thống] sẽ thực hiện một chức năng yêu cầu mà không có sự cố trong các điều kiện được mô tả trong một khoảng thời gian được mô tả", theo định nghĩa trong [[Oco12]](https://sre.google/sre-book/bibliography#Oco12).

<a id="fn3"></a>[3](#fn3) Các hệ thống phần mềm mà chúng tôi quan tâm chủ yếu là các website và các dịch vụ tương tự; chúng tôi không thảo luận về các mối quan tâm độ tin cậy đối mặt với phần mềm dự định cho nhà máy điện hạt nhân, máy bay, thiết bị y tế, hoặc các hệ thống an toàn quan trọng khác. Tuy nhiên, chúng tôi so sánh cách tiếp cận của mình với những cách được sử dụng trong các ngành khác trong [Lessons Learned from Other Industries](https://sre.google/sre-book/lessons-learned/).

<a id="fn4"></a>[4](#fn4) Về điều này, chúng tôi khác biệt với thuật ngữ ngành DevOps, vì mặc dù chúng tôi chắc chắn coi cơ sở hạ tầng là code (infrastructure as code), chúng tôi có *độ tin cậy* là mối quan tâm chính. Ngoài ra, chúng tôi định hướng mạnh mẽ vào việc loại bỏ sự cần thiết cho vận hành — xem [The Evolution of Automation at Google](https://sre.google/sre-book/automation-at-google/) để biết thêm chi tiết.

<a id="fn5"></a>[5](#fn5) Ngoài câu chuyện tuyệt vời này, bà cũng có một yêu cầu đáng kể trong việc phổ biến thuật ngữ "kỹ thuật phần mềm".

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
