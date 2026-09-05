# Chương 14. Quản lý Incident (Managing Incidents)

> **Nguyên bản:** [Chapter 14 - Managing Incidents](https://sre.google/sre-book/managing-incidents/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Andrew Stribblehill<sup>[1](#fn1)</sup>
*Biên tập:* Kavita Guliani

Quản lý incident (sự cố) hiệu quả là chìa khóa để hạn chế sự gián đoạn mà incident gây ra và khôi phục hoạt động business bình thường nhanh nhất có thể. Nếu bạn không hình dung và diễn tập trước [phản ứng của mình với các incident tiềm tàng](https://sre.google/resources/practices-and-processes/anatomy-of-an-incident/), cách quản lý incident có nguyên tắc có thể bị phá vỡ ngay trong các tình huống thực tế.

Chương này xem xét chân dung của một incident mất kiểm soát do các thực hành quản lý incident ad hoc (tùy hứng), phác thảo cách tiếp cận có quản lý đối với incident, rồi xem lại cùng incident đó có thể đã diễn ra thế nào nếu được xử lý với một quy trình quản lý incident tốt.

## Các Incident Không có Quản lý (Unmanaged Incidents)

Hãy đặt mình vào vị trí của Mary, kỹ sư on-call (trực sự cố) của The Firm. Đã 2 giờ chiều thứ Năm và máy gọi trực (pager) của bạn vừa reo. Giám sát hộp đen (black-box monitoring) cho biết dịch vụ của bạn đã ngừng phục vụ *bất kỳ* traffic (lưu lượng) nào trong cả một datacenter (trung tâm dữ liệu). Với một tiếng thở dài, bạn đặt ly cà phê xuống và bắt tay vào việc. Vài phút sau, một cảnh báo khác cho biết datacenter thứ hai cũng ngừng phục vụ. Rồi cái thứ ba trong số năm datacenter của bạn cũng hỏng. Để tệ hơn, có nhiều traffic hơn các datacenter còn lại có thể xử lý, nên chúng bắt đầu quá tải (overload). Trước khi bạn kịp nhận ra, dịch vụ quá tải và không thể phục vụ bất kỳ yêu cầu nào.

Bạn dán mắt vào các log (nhật ký) trong khoảng thời gian tưởng chừng như cả một thế kỷ. Hàng nghìn dòng log gợi ý có một lỗi trong một trong các module vừa được cập nhật gần đây, nên bạn quyết định hoàn tác (revert) các server về release (phát hành) trước đó. Khi thấy rollback không giúp gì, bạn gọi cho Josephine, người đã viết phần lớn code cho dịch vụ đang "chảy máu" này. Nhắc rằng ở múi giờ của cô ấy đã 3:30 sáng, cô lờ mờ đồng ý đăng nhập xem qua. Các đồng nghiệp Sabrina và Robin bắt đầu "chọc" quanh từ các terminal của riêng họ. "Chỉ đang nhìn thôi," họ nói với bạn.

Giờ đây, một trong những ông/bà “bộ vest” đã gọi cho sếp bạn, nóng giận đòi biết lý do vì sao ông ấy không được thông báo về “sự sụp đổ hoàn toàn của dịch vụ quan trọng này”. Song song đó, các phó tổng giám đốc (vice president) cũng thúc giục bạn đưa ra ETA (ước tính thời gian hoàn thành), lặp đi lặp lại hỏi “Làm sao điều này có thể xảy ra được?”. Bạn muốn thông cảm, nhưng làm vậy sẽ đòi hỏi sự tập trung mà bạn đang dành dụm cho công việc. Các VP dựa vào kinh nghiệm kỹ thuật trước đó và đưa ra những bình luận không liên quan nhưng khó bác bỏ như “Tăng kích thước trang (page) bộ nhớ đi!”.

Thời gian trôi qua; hai datacenter còn lại cũng hỏng hoàn toàn. Trong khi bạn không hay biết, Josephine còn mơ ngủ đã gọi cho Malcolm. Anh ấy lóe lên một ý tưởng: có gì đó liên quan đến CPU affinity. Anh tin chắc mình có thể tối ưu hóa các tiến trình server còn lại nếu chỉ cần triển khai một thay đổi đơn giản này xuống môi trường production, nên anh đã làm vậy. Chỉ trong vài giây, các server khởi động lại, nạp thay đổi. Rồi chết.

## Giải phẫu một Incident Không có Quản lý (The Anatomy of an Unmanaged Incident)

Lưu ý rằng trong kịch bản trên, mọi người đều đang làm đúng việc của mình, theo cách họ nhìn nhận. Làm sao mọi thứ có thể sai đến vậy? Một số mối nguy hiểm chung đã khiến sự cố này mất kiểm soát.

### Sự Tập trung Sắc nét vào Vấn đề Kỹ thuật (Sharp Focus on the Technical Problem)

Chúng tôi thường tuyển những người như Mary vì năng lực kỹ thuật. Vì thế, việc cô ấy đang mải thực hiện các thay đổi vận hành trên hệ thống để cố giải quyết vấn đề là điều dễ hiểu. Cô không có thời gian nghĩ đến bức tranh tổng thể — cách giảm nhẹ vấn đề — vì nhiệm vụ kỹ thuật trước mắt đang đè nặng lên cô.

### Truyền thông Kém (Poor Communication)

Cùng vì lý do đó, Mary quá bận nên không thể truyền thông rõ ràng. Không ai biết đồng nghiệp của mình đang làm gì. Các lãnh đạo kinh doanh bực bội, khách hàng thất vọng, và các kỹ sư khác lẽ ra có thể hỗ trợ xử lý lỗi hay khắc phục sự cố thì không được sử dụng hiệu quả.

### Làm việc Đơn độc (Freelancing)

Malcolm đang thực hiện các thay đổi trên hệ thống với thiện chí tốt nhất. Tuy nhiên, anh không phối hợp với đồng nghiệp — ngay cả Mary, người về mặt kỹ thuật đang chịu trách nhiệm xử lý lỗi. Những thay đổi của anh đã đẩy một tình hình vốn đã tồi tệ thành tồi tệ hơn rất nhiều.

## Các Yếu tố của Quy trình Quản lý Incident (Elements of Incident Management Process)

Kỹ năng và thực hành quản lý incident tồn tại nhằm điều tiết năng lượng của những cá nhân nhiệt tình. Hệ thống quản lý incident của Google dựa trên [Incident Command System (Hệ thống Chỉ huy Sự cố),](https://sre.google/resources/practices-and-processes/incident-management-guide/)<sup>[2](#fn2)</sup>, nổi tiếng nhờ sự rõ ràng và khả năng mở rộng (scale).

Một quy trình quản lý incident được thiết kế tốt có các tính năng sau.

### Tách biệt Trách nhiệm Tái đệ quy (Recursive Separation of Responsibilities)

Điều quan trọng là mọi người tham gia incident phải nắm rõ vai trò của mình, tránh can thiệp vào phần việc của người khác. Nghe có vẻ ngược trực giác, nhưng việc phân định trách nhiệm rõ ràng lại giúp các cá nhân có nhiều quyền tự chủ hơn, vì họ không phải lo lắng hay nghi ngờ đồng nghiệp.

Nếu một thành viên bị quá tải, người đó cần yêu cầu trưởng nhóm lập kế hoạch (planning lead) bổ sung nhân sự. Sau đó, họ nên ủy thác (delegate) công việc cho người khác — một tác vụ có thể dẫn đến tạo ra các subincident (sự cố con). Thay vào đó, một người lãnh đạo vai trò có thể ủy thác các thành phần hệ thống cho đồng nghiệp, những người sẽ báo cáo thông tin cấp cao ngược lại cho các nhà lãnh đạo.

Nhiều vai trò riêng biệt nên được ủy thác cho các cá nhân cụ thể:

#### Chỉ huy Incident (Incident Command)

Người chỉ huy sự cố (incident commander) nắm giữ bức tranh tổng thể về sự cố. Họ tổ chức lực lượng phản ứng, phân công trách nhiệm dựa trên nhu cầu và mức độ ưu tiên. *De facto* (trên thực tế), người chỉ huy đảm nhận mọi vai trò chưa được ủy quyền. Khi cần thiết, họ có thể loại bỏ các trở ngại (roadblock) đang cản trở đội Ops (Vận hành) hoạt động hiệu quả nhất.

#### Công việc Vận hành (Operational Work)

Người lãnh đạo Ops (Operations lead) làm việc với người chỉ huy incident để phản ứng với incident bằng cách áp dụng các công cụ vận hành cho tác vụ trước mắt. Đội vận hành nên là nhóm duy nhất được phép sửa đổi hệ thống trong một incident.

#### Truyền thông (Communication)

Người này là "khuôn mặt" công khai của lực lượng đặc nhiệm phản ứng incident. Nhiệm vụ của họ chắc chắn bao gồm việc phát hành các bản cập nhật định kỳ cho đội phản ứng incident và các bên liên quan (thường qua email), và có thể mở rộng sang các tác vụ như giữ tài liệu incident chính xác, cập nhật.

#### Lập kế hoạch (Planning)

Vai trò lập kế hoạch hỗ trợ Ops bằng cách xử lý các vấn đề dài hạn hơn, chẳng hạn như đệ trình bug (lỗi), đặt giao đồ ăn, sắp xếp các lần bàn giao (handoff), và theo dõi cách hệ thống đã lệch khỏi trạng thái chuẩn để có thể hoàn tác khi incident được giải quyết.

### Một Trạm Chỉ huy Được Nhận dạng (A Recognized Command Post)

Các bên liên quan cần nắm rõ cách thức tương tác với người chỉ huy incident. Trong nhiều trường hợp, việc tập hợp các thành viên lực lượng đặc nhiệm incident tại một "War Room" (Phòng chỉ huy) trung tâm là hợp lý. Các đội khác có thể ưu tiên làm việc tại chỗ, theo dõi các cập nhật incident qua email và IRC.

Google nhận thấy IRC là một lợi ích lớn trong phản ứng sự cố. IRC rất đáng tin cậy và có thể dùng như một log các trao đổi về sự kiện, một bản ghi như vậy vô giá trong việc giữ các thay đổi trạng thái chi tiết trong tâm trí. Chúng tôi cũng viết các bot ghi log traffic liên quan đến incident (hữu ích cho phân tích postmortem) và các bot khác ghi log các sự kiện như cảnh báo đến kênh. IRC cũng là môi trường thuận tiện để các đội phân tán về mặt địa lý phối hợp với nhau.

### Tài liệu Trạng thái Incident Sống (Live Incident State Document)

Trách nhiệm quan trọng nhất của người chỉ huy incident là duy trì một tài liệu incident "sống". Tài liệu này có thể nằm trong một wiki (trang wiki), nhưng lý tưởng là nên cho phép nhiều người chỉnh sửa đồng thời. Phần lớn các đội chúng tôi dùng Google Docs, mặc dù các SRE phụ trách Google Docs lại dùng Google Sites: sau cùng, việc phụ thuộc vào phần mềm mà bạn đang cố sửa như một phần của hệ thống quản lý incident của mình khó có thể kết thúc tốt đẹp.

Xem [Example Incident State Document](https://sre.google/sre-book/incident-document/) để tham khảo một tài liệu incident mẫu. Đây là tài liệu sống nên có thể bừa bộn, nhưng phải hữu dụng. Dùng một template (mẫu) giúp tạo tài liệu dễ hơn, và giữ thông tin quan trọng nhất ở đầu giúp tài liệu dễ dùng hơn. Lưu lại tài liệu này để phục vụ phân tích postmortem (báo cáo sau sự cố) và, nếu cần, phân tích ở cấp cao hơn (meta analysis).

### Bàn giao (Handoff) Rõ ràng, Sống (Clear, Live Handoff)

Điều thiết yếu là vai trò người chỉ huy incident phải được bàn giao rõ ràng vào cuối ngày làm việc. Nếu bạn bàn giao quyền chỉ huy cho ai đó ở một vị trí khác, bạn có thể cập nhật cho người chỉ huy mới qua điện thoại hay cuộc gọi video. Khi người chỉ huy mới đã được cập nhật đầy đủ, người chỉ huy sắp bàn giao nên nói rõ trong lúc bàn giao, cụ thể là "Giờ bạn là người chỉ huy incident nhé?", và không nên rời cuộc gọi cho đến khi có xác nhận bàn giao chắc chắn. Việc bàn giao nên được thông báo cho những người khác đang làm việc trên incident để rõ ai đang dẫn dắt nỗ lực quản lý incident vào mọi thời điểm.

## Một Incident Có Quản lý (A Managed Incident)

Giờ hãy xem incident này có thể đã diễn ra thế nào nếu được xử lý bằng các nguyên lý của quản lý incident.

Đã 2 giờ chiều, và Mary đang dùng ly cà phê thứ ba trong ngày. Tiếng máy gọi trực chói tai khiến cô giật mình, vội nuốt thức uống xuống. Vấn đề: một datacenter đã ngừng phục vụ traffic. Cô bắt đầu điều tra. Chẳng bao lâu sau, một cảnh báo khác được kích hoạt — datacenter thứ hai trong năm cũng đang hỏng. Vì đây là một vấn đề đang leo thang nhanh, cô biết mình sẽ được lợi từ cấu trúc khung quản lý sự cố.

Mary kéo Sabrina lại. “Bạn có thể nhận quyền chỉ huy không?” Gật đầu đồng ý, Sabrina nhanh chóng nhận bản tóm tắt từ Mary về những gì đã xảy ra đến giờ. Cô ghi lại các chi tiết này trong một email gửi đến một danh sách email đã thỏa thuận trước. Sabrina nhận ra mình chưa xác định được phạm vi tác động của sự cố, nên xin đánh giá của Mary. Mary trả lời: “Người dùng chưa bị ảnh hưởng; hy vọng ta không mất thêm datacenter thứ ba.” Sabrina ghi lại phản hồi của Mary vào một tài liệu sự cố sống.

Khi cảnh báo thứ ba fire (kích hoạt), Sabrina kịp nhìn thấy nó giữa các tin nhắn debug trên IRC và nhanh chóng cập nhật chuỗi email. Chuỗi này giữ cho các VP nắm được trạng thái cấp cao mà không khiến họ lún vào chi tiết. Sabrina yêu cầu một người đại diện truyền thông bên ngoài bắt đầu phác thảo thông điệp gửi người dùng. Sau đó cô hỏi ý Mary xem có nên liên hệ với developer on-call (lúc này là Josephine) không. Được Mary đồng ý, Sabrina kéo Josephine vào cuộc.

Khi Josephine đăng nhập, Robin đã tình nguyện hỗ trợ. Sabrina nhắc cả hai rằng họ phải ưu tiên mọi tác vụ Mary giao, đồng thời cập nhật cho Mary về bất kỳ hành động bổ sung nào họ thực hiện. Robin và Josephine nhanh chóng nắm bắt tình hình bằng cách đọc tài liệu sự cố.

Cho đến lúc đó, Mary đã thử release binary (file thực thi) cũ và thấy nó không ổn: cô lẩm bẩm điều này với Robin, người cập nhật IRC để nói nỗ lực sửa này không hiệu quả. Sabrina dán cập nhật này vào tài liệu quản lý incident sống.

5 giờ chiều, Sabrina bắt đầu tìm người thay thế để xử lý sự cố, vì cô và các đồng nghiệp sắp về nhà. Cô cập nhật tài liệu sự cố. 5 giờ 45, nhóm họp qua điện thoại ngắn để nắm tình hình hiện tại. 6 giờ, họ bàn giao trách nhiệm cho các đồng nghiệp ở văn phòng bên.

Mary quay lại làm việc sáng hôm sau và thấy các đồng nghiệp bên kia đại dương đã tiếp quản bug, đã giảm nhẹ vấn đề, đóng incident và bắt đầu viết postmortem. Vấn đề đã được giải quyết, cô pha một chút cà phê tươi và ngồi xuống lập kế hoạch các cải tiến cấu trúc để những vấn đề kiểu này không còn ảnh hưởng đến đội nữa.

## Khi Nào Nên Tuyên bố một Incident (When to Declare an Incident)

Thay vì đợi đến khi sự cố đã kéo dài nhiều giờ rồi mới kích hoạt quy trình quản lý, tốt hơn nên tuyên bố incident sớm, tìm giải pháp đơn giản và đóng incident. Hãy xác định rõ các điều kiện để tuyên bố incident. Đội tôi tuân theo các nguyên tắc chung sau — nếu bất kỳ điều nào dưới đây đúng, sự kiện đó được coi là incident:

-   Bạn có cần kéo một đội thứ hai vào việc sửa vấn đề không?
-   Outage (mất dịch vụ) có bị khách hàng nhìn thấy không?
-   Vấn đề có không được giải quyết ngay cả sau một giờ phân tích tập trung không?

Năng lực quản lý incident sẽ nhanh chóng mai một nếu không được thực hành thường xuyên. Vậy kỹ sư cần làm gì để giữ kỹ năng này luôn sắc bén, đủ sức xử lý nhiều incident hơn? May mắn là khung quản lý incident có thể áp dụng cho cả các thay đổi vận hành khác, nhất là những thay đổi cần trải qua nhiều múi giờ và/hoặc nhiều đội. Nếu bạn thường xuyên sử dụng khung này như một phần bình thường của quy trình quản lý thay đổi, bạn sẽ dễ dàng tuân theo nó khi một incident thực sự xảy ra. Nếu tổ chức của bạn thực hiện kiểm thử phục hồi thảm họa (disaster-recovery testing) (bạn nên làm, vì nó *thú vị*: xem [[Kri12]](https://sre.google/sre-book/bibliography#Kri12)), thì quản lý incident nên là một phần của quy trình kiểm thử đó. Chúng tôi thường đóng vai phản ứng với một vấn đề on-call đã được giải quyết, có thể bởi đồng nghiệp ở một vị trí khác, để làm quen hơn với quản lý incident.

## Tóm lại (In Summary)

Chúng tôi nhận thấy rằng việc xây dựng trước chiến lược quản lý incident, thiết kế để mở rộng (scale) mượt mà và thường xuyên đưa vào thực hành giúp giảm thời gian phục hồi trung bình (mean time to recovery), đồng thời mang lại cho nhân viên cách xử lý các vấn đề khẩn cấp ít căng thẳng hơn. Bất kỳ tổ chức nào quan tâm đến độ tin cậy đều sẽ được lợi từ việc theo đuổi một chiến lược tương tự.

## Các Thực hành Tốt nhất cho Quản lý Incident (Best Practices for Incident Management)

**Ưu tiên (Prioritize).** Chặn sự chảy máu, khôi phục dịch vụ, và bảo tồn bằng chứng cho việc tìm nguyên nhân gốc rễ.

**Chuẩn bị (Prepare).** Xây dựng và ghi lại các quy trình quản lý incident của bạn trước, tham vấn với các thành viên tham gia incident.

**Tin tưởng (Trust).** Trao toàn quyền tự chủ trong vai trò được giao cho tất cả các thành viên tham gia incident.

**Nội quan (Introspect).** Hãy để ý trạng thái cảm xúc của bản thân khi phản ứng với một incident. Nếu bắt đầu thấy hoảng loạn hoặc choáng ngợp, hãy kêu gọi thêm sự hỗ trợ.

**Cân nhắc các lựa chọn thay thế (Consider alternatives).** Định kỳ xem xét các tùy chọn và đánh giá lại xem nên tiếp tục hướng đi hiện tại hay chuyển sang một cách tiếp cận khác trong phản ứng sự cố.

**Luyện tập (Practice).** Dùng quy trình thường xuyên để nó trở thành bản năng thứ hai.

**Đổi vai (Change it around).** Lần trước bạn là người chỉ huy incident, phải không? Lần này thử một vai trò khác. Khuyến khích mọi thành viên đội làm quen với mỗi vai trò.

<a id="fn1"></a>[1](#fn1) Một phiên bản trước đó của chương này xuất hiện như một bài viết trong *;login:* (tháng 4 năm 2015, tập 40, số 2).

<a id="fn2"></a>[2](#fn2) Xem [*https://www.fema.gov/national-incident-management-system*](https://www.fema.gov/national-incident-management-system) để có thêm chi tiết.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
