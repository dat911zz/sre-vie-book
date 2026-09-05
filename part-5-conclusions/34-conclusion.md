> **Nguyên bản:** [Chapter 34 - Conclusion](https://sre.google/sre-book/conclusion/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

## Chương 34. Kết Luận (Conclusion)

Tác giả: Benjamin Lutch<sup>[1](#fn1)</sup>  
Biên tập: Betsy Beyer

Tôi đọc hết cuốn sách này với niềm tự hào vô hạn. Kể từ khi bắt đầu làm việc tại Excite vào đầu thập niên 90, nơi nhóm của tôi là một dạng nhóm SRE thời đồ đá mang tên "Software Operations" (Vận hành Phần mềm), tôi đã dành sự nghiệp của mình vấp váp qua quy trình xây dựng các hệ thống. Nhìn lại những trải nghiệm qua nhiều năm trong ngành công nghệ, thật đáng kinh ngạc khi thấy ý tưởng về SRE đã bén rễ tại Google và phát triển nhanh đến mức nào. SRE đã phát triển từ vài trăm kỹ sư khi tôi gia nhập Google vào năm 2006 lên hơn 1.000 người ngày nay, trải dài trên hơn chục địa điểm và vận hành những gì tôi cho là hạ tầng tính toán thú vị nhất hành tinh.

Điều gì đã giúp tổ chức SRE tại Google phát triển trong suốt thập kỷ qua, duy trì cơ sở hạ tầng khổng lồ này một cách thông minh, hiệu quả và có khả năng mở rộng? Tôi cho rằng chìa khóa cho thành công áp đảo của SRE nằm ở bản chất của các nguyên lý mà nó vận hành.

Các đội SRE được tổ chức để kỹ sư của chúng tôi chia đều thời gian cho hai loại công việc có tầm quan trọng ngang nhau. Một mặt, SRE trực on-call, đòi hỏi chúng tôi phải trực tiếp thao tác trên các hệ thống, quan sát chúng hỏng ở đâu và như thế nào, đồng thời hiểu rõ các thách thức về cách mở rộng hệ thống một cách tối ưu. Mặt khác, chúng tôi cũng có thời gian để suy ngẫm và quyết định xây dựng những gì nhằm giúp các hệ thống đó dễ quản lý hơn. Về bản chất, chúng tôi được tận hưởng niềm vui khi đảm nhận cả vai trò người lái máy bay *và* kỹ sư/thiết kế. Những trải nghiệm của chúng tôi trong quá trình vận hành cơ sở hạ tầng tính toán khổng lồ được mã hóa thành mã thực tế, sau đó đóng gói thành một sản phẩm riêng biệt.

Các giải pháp này sau đó được các nhóm SRE khác dễ dàng áp dụng, và cuối cùng bất kỳ ai tại Google (hay thậm chí bên ngoài Google... hãy nghĩ đến Google Cloud!) đều có thể sử dụng hoặc cải tiến, dựa trên kinh nghiệm tích lũy và các hệ thống mà chúng tôi đã xây dựng.

Khi xây dựng đội ngũ hay hệ thống, nền tảng lý tưởng cần là một bộ quy tắc hoặc tiên đề đủ tổng quát để phát huy tác dụng ngay, nhưng vẫn giữ được giá trị trong tương lai. Phần lớn những gì Ben Treynor Sloss phác thảo trong phần giới thiệu của cuốn sách này chính là điều đó: một bộ trách nhiệm linh hoạt, gần như bất tử theo thời gian, vẫn đúng đắn ngay cả 10 năm sau khi được thai nghén, bất chấp những thay đổi và tăng trưởng mà hạ tầng của Google và đội SRE đã trải qua.

Khi SRE phát triển, chúng tôi nhận thấy một vài động lực khác nhau đang tác động. Thứ nhất là sự nhất quán trong các trách nhiệm và mối quan tâm chính của SRE theo thời gian: dù các hệ thống có thể lớn hơn hoặc nhanh hơn 1.000 lần, nhưng cuối cùng, chúng vẫn cần duy trì độ tin cậy, linh hoạt, dễ quản lý trong trường hợp khẩn cấp, được giám sát tốt và được lên kế hoạch năng lực. Đồng thời, các hoạt động điển hình do SRE thực hiện buộc phải tiến hóa khi các dịch vụ của Google và năng lực của SRE ngày càng trưởng thành. Ví dụ, điều từng là mục tiêu "xây dựng một dashboard (bảng điều khiển) cho 20 máy" giờ đây có thể thay bằng "tự động hóa việc phát hiện, xây dựng dashboard và cảnh báo trên một hạm đội gồm hàng chục nghìn máy."

Với những ai chưa từng trải qua thực tế SRE trong thập kỷ qua, việc so sánh cách SRE tư duy về các hệ thống phức tạp với cách ngành hàng không tiếp cận vấn đề bay sẽ giúp hình dung rõ hơn về quá trình tiến hóa và trưởng thành của SRE. Dù mức độ rủi ro khi xảy ra sự cố giữa hai ngành rất khác nhau, một số điểm tương đồng cốt lõi vẫn đúng.

Hãy tưởng tượng bạn muốn bay giữa hai thành phố một trăm năm trước. Máy bay của bạn có lẽ chỉ có một động cơ (hai, nếu bạn may mắn), vài túi hàng hóa, và một phi công. Phi công cũng đảm nhận vai trò thợ cơ khí, và có thể ngoài ra còn đóng vai trò người xếp và dỡ hàng hóa. Buồng lái (cockpit) có chỗ cho phi công, và nếu bạn may mắn, một phi công phụ/nhà hàng hải. Máy bay nhỏ của bạn sẽ bật khỏi một đường băng trong thời tiết đẹp, và nếu mọi thứ suôn sẻ, bạn sẽ từ từ leo lên qua bầu trời và cuối cùng hạ cánh ở một thành phố khác, có lẽ cách vài trăm dặm. Sự cố ở bất kỳ hệ thống nào của máy bay là thảm khốc, và không phải điều hiếm có khi một phi công phải trèo ra khỏi buồng lái để thực hiện sửa chữa trong khi đang bay! Các hệ thống cấp vào buồng lái là thiết yếu, đơn giản và mong manh, và nhiều khả năng không có tính dự phòng.

Nhanh tiến lên một trăm năm, ta thấy một chiếc 747 khổng lồ đang đậu trên bãi đỗ. Hàng trăm hành khách đang lên ở cả hai tầng, trong khi hàng tấn hàng hóa đồng thời được xếp vào khoang chứa bên dưới. Máy bay chứa đầy các hệ thống dự phòng đáng tin cậy. Nó là một mô hình về sự an toàn và độ tin cậy; thực tế, bạn an toàn hơn trên không so với trên mặt đất trong một chiếc ô tô. Máy bay của bạn sẽ cất cánh từ một đường vạch đứt trên một đường băng ở một châu lục, và hạ cánh dễ dàng trên một đường vạch đứt trên một đường băng khác cách 6.000 dặm, đúng lịch trình — trong vài phút so với thời gian hạ cánh được dự báo. Nhưng hãy nhìn vào trong buồng lái và bạn thấy gì? Lại chỉ có hai phi công!

Làm thế nào mà mọi yếu tố khác của trải nghiệm bay — sự an toàn, năng lực, tốc độ và độ tin cậy — đã mở rộng tuyệt vời đến vậy, trong khi vẫn chỉ có hai phi công? Câu trả lời cho câu hỏi này là một sự song song tuyệt vời với cách Google tiếp cận các hệ thống khổng lồ, phức tạp một cách kỳ diệu mà SRE vận hành. Các giao diện đến các hệ thống vận hành của máy bay được thiết kế kỹ lưỡng và đủ dễ tiếp cận để việc học cách lái chúng trong các điều kiện bình thường không phải là một nhiệm vụ không thể vượt qua. Tuy nhiên, những giao diện này cũng cung cấp đủ sự linh hoạt, và những người vận hành chúng được đào tạo đủ, để các phản ứng với các trường hợp khẩn cấp là vững chắc và nhanh chóng. Buồng lái được thiết kế bởi những người hiểu các hệ thống phức tạp và cách trình bày chúng cho con người theo cách vừa hấp thụ được vừa có thể mở rộng. Các hệ thống bên dưới buồng lái có tất cả các thuộc tính đã được thảo luận trong cuốn sách này: khả dụng, tối ưu hóa hiệu suất, quản lý thay đổi, giám sát và cảnh báo, lập kế hoạch năng lực, và phản ứng khẩn cấp.

Cuối cùng, mục tiêu của SRE là đi theo một lộ trình tương tự. Một đội SRE nên nhỏ gọn nhất có thể và vận hành ở mức độ trừu tượng cao, dựa vào rất nhiều hệ thống dự phòng như các cơ chế dự phòng (failsafe) và các API (giao diện lập trình ứng dụng) được thiết kế chu đáo để giao tiếp với các hệ thống. Đồng thời, đội SRE cũng nên có kiến thức toàn diện về các hệ thống — cách chúng vận hành, cách chúng thất bại, và cách phản ứng với các sự cố — đến từ việc vận hành chúng hàng ngày.

<a id="fn1"></a>[1](#fn1) Phó Chủ tịch, Site Reliability Engineering, của Google, Inc.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
