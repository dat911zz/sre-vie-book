> **Nguyên bản:** [Chapter 34 - Conclusion](https://sre.google/sre-book/conclusion/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

## Chương 34. Kết Luận (Conclusion)

Tác giả: Benjamin Lutch<sup>[1](#fn1)</sup>  
Biên tập: Betsy Beyer

Tôi đọc hết cuốn sách này với niềm tự hào vô hạn. Kể từ khi tôi bắt đầu làm việc tại Excite vào đầu thập niên 90, nơi nhóm của tôi là một dạng nhóm SRE thời đồ đá được gọi là "Software Operations" (Vận hành Phần mềm), tôi đã dành sự nghiệp của mình vấp váp qua quy trình xây dựng các hệ thống. Trong ánh sáng của những trải nghiệm qua nhiều năm trong ngành công nghệ, thật đáng kinh ngạc khi thấy ý tưởng về SRE đã bén rễ tại Google và phát triển nhanh đến mức nào. SRE đã phát triển từ vài trăm kỹ sư khi tôi gia nhập Google vào năm 2006 lên hơn 1.000 người ngày nay, trải dài trên hơn chục địa điểm và vận hành những gì tôi cho là hạ tầng tính toán thú vị nhất hành tinh.

Vậy điều gì đã cho phép tổ chức SRE tại Google phát triển qua thập kỷ qua để duy trì cơ sở hạ tầng khổng lồ này theo cách thức thông minh, hiệu quả và có thể mở rộng? Tôi nghĩ rằng chìa khóa cho thành công áp đảo của SRE là bản chất của các nguyên lý mà nó vận hành.

Các đội SRE được xây dựng sao cho kỹ sư của chúng tôi chia thời gian giữa hai loại công việc có tầm quan trọng như nhau. Các SRE trực on-call (trực sự cố), điều này đòi hỏi chúng tôi đặt tay lên các hệ thống, quan sát nơi và như thế nào các hệ thống này vỡ, và hiểu các thách thức như làm thế nào để mở rộng chúng một cách tốt nhất. Nhưng chúng tôi cũng có thời gian để suy ngẫm và quyết định xây dựng điều gì để làm cho những hệ thống đó dễ quản lý hơn. Về bản chất, chúng tôi có đặc quyền đảm nhận cả vai trò của người lái máy bay *và* kỹ sư/thiết kế. Những trải nghiệm của chúng tôi khi vận hành cơ sở hạ tầng tính toán khổng lồ được mã hóa trong mã thực tế và sau đó được đóng gói thành một sản phẩm riêng biệt.

Những giải pháp này sau đó dễ dàng được các đội SRE khác sử dụng và cuối cùng được bất kỳ ai tại Google (hoặc thậm chí bên ngoài Google... hãy nghĩ đến Google Cloud!) muốn sử dụng hoặc cải thiện, dựa trên trải nghiệm mà chúng tôi đã tích lũy và các hệ thống mà chúng tôi đã xây dựng.

Khi bạn tiếp cận việc xây dựng một đội hoặc một hệ thống, lý tưởng là nền tảng của nó nên là một bộ quy tắc hoặc tiên đề đủ tổng quát để ngay lập tức hữu ích, nhưng vẫn còn liên quan trong tương lai. Phần lớn những gì Ben Treynor Sloss phác thảo trong phần giới thiệu của cuốn sách này đại diện cho chính điều đó: một bộ trách nhiệm linh hoạt, phần lớn bất tử theo thời gian, vẫn đúng đắn ngay cả 10 năm sau khi chúng được thai nghén, bất chấp những thay đổi và tăng trưởng mà hạ tầng của Google và đội SRE đã trải qua.

Khi SRE phát triển, chúng tôi nhận thấy một vài động lực khác nhau đang hoạt động. Thứ nhất là bản chất nhất quán của các trách nhiệm và mối quan tâm chính của SRE theo thời gian: các hệ thống của chúng tôi có thể lớn hơn hoặc nhanh hơn 1.000 lần, nhưng cuối cùng, chúng vẫn cần phải duy trì độ tin cậy, linh hoạt, dễ quản lý trong một trường hợp khẩn cấp, được giám sát tốt, và được lên kế hoạch năng lực. Đồng thời, các hoạt động điển hình do SRE thực hiện buộc phải tiến hóa khi các dịch vụ của Google và năng lực của SRE trở nên trưởng thành. Ví dụ, điều từng là một mục tiêu "xây dựng một dashboard (bảng điều khiển) cho 20 máy" giờ đây có thể thay bằng "tự động hóa việc phát hiện, xây dựng dashboard và cảnh báo trên một hạm đội gồm hàng chục nghìn máy."

Đối với những người chưa ở trong chiến hào của SRE trong thập kỷ qua, một phép loại suy giữa cách SRE suy nghĩ về các hệ thống phức tạp và cách ngành công nghiệp máy bay tiếp cận việc bay là hữu ích trong việc khái niệm hóa cách SRE đã tiến hóa và trưởng thành theo thời gian. Trong khi mức độ rủi ro của sự thất bại giữa hai ngành rất khác nhau, một số điểm tương đồng cốt lõi vẫn đúng.

Hãy tưởng tượng bạn muốn bay giữa hai thành phố một trăm năm trước. Máy bay của bạn có lẽ chỉ có một động cơ (hai, nếu bạn may mắn), vài túi hàng hóa, và một phi công. Phi công cũng đảm nhận vai trò thợ cơ khí, và có thể ngoài ra còn đóng vai trò người xếp và dỡ hàng hóa. Cabin có chỗ cho phi công, và nếu bạn may mắn, một phi công phụ/nhà hàng hải. Máy bay nhỏ của bạn sẽ bật khỏi một đường băng trong thời tiết đẹp, và nếu mọi thứ suôn sẻ, bạn sẽ từ từ leo lên qua bầu trời và cuối cùng hạ cánh ở một thành phố khác, có lẽ cách vài trăm dặm. Sự thất bại của bất kỳ hệ thống nào của máy bay là thảm khốc, và không phải điều hiếm có khi một phi công phải trèo ra khỏi cabin để thực hiện sửa chữa trong khi đang bay! Các hệ thống cấp vào cabin là thiết yếu, đơn giản và mong manh, và nhiều khả năng không có tính dự phòng.

Nhanh tiến lên một trăm năm đến một chiếc 747 khổng lồ đang đậu trên bãi đỗ. Hàng trăm hành khách đang lên ở cả hai tầng, trong khi hàng tấn hàng hóa đồng thời được xếp vào khoang chứa bên dưới. Máy bay chứa đầy các hệ thống dự phòng đáng tin cậy. Nó là một mô hình về sự an toàn và độ tin cậy; thực tế, bạn an toàn hơn trên không so với trên mặt đất trong một chiếc ô tô. Máy bay của bạn sẽ cất cánh từ một đường vạch đứt trên một đường băng ở một châu lục, và hạ cánh dễ dàng trên một đường vạch đứt trên một đường băng khác cách 6.000 dặm, đúng lịch trình — trong vài phút so với thời gian hạ cánh được dự báo. Nhưng hãy nhìn vào trong cabin và bạn thấy gì? Lại chỉ có hai phi công!

Làm thế nào mà mọi yếu tố khác của trải nghiệm bay — sự an toàn, năng lực, tốc độ và độ tin cậy — đã mở rộng tuyệt vời đến vậy, trong khi vẫn chỉ có hai phi công? Câu trả lời cho câu hỏi này là một sự song song tuyệt vời với cách Google tiếp cận các hệ thống khổng lồ, phức tạp một cách kỳ diệu mà SRE vận hành. Các giao diện đến các hệ thống vận hành của máy bay được suy nghĩ kỹ lưỡng và đủ dễ tiếp cận để việc học cách lái chúng trong các điều kiện bình thường không phải là một nhiệm vụ không thể vượt qua. Tuy nhiên, những giao diện này cũng cung cấp đủ sự linh hoạt, và những người vận hành chúng được đào tạo đủ, để các phản ứng với các trường hợp khẩn cấp là vững chắc và nhanh chóng. Cabin được thiết kế bởi những người hiểu các hệ thống phức tạp và cách trình bày chúng cho con người theo cách vừa hấp thụ được vừa có thể mở rộng. Các hệ thống bên dưới cabin có tất cả các thuộc tính đã được thảo luận trong cuốn sách này: khả dụng, tối ưu hóa hiệu suất, quản lý thay đổi, giám sát và cảnh báo, lập kế hoạch năng lực, và phản ứng khẩn cấp.

Cuối cùng, mục tiêu của SRE là đi theo một lộ trình tương tự. Một đội SRE nên nhỏ gọn nhất có thể và vận hành ở mức độ trừu tượng cao, dựa vào rất nhiều hệ thống dự phòng như các cơ chế dự phòng (failsafe) và các API (giao diện lập trình ứng dụng) được thiết kế chu đáo để giao tiếp với các hệ thống. Đồng thời, đội SRE cũng nên có kiến thức toàn diện về các hệ thống — cách chúng vận hành, cách chúng thất bại, và cách phản ứng với các sự thất bại — đến từ việc vận hành chúng hàng ngày.

<a id="fn1"></a>[1](#fn1) Phó Chủ tịch, Site Reliability Engineering, của Google, Inc.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
