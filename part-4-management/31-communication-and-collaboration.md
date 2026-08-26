# Chương 31. Giao tiếp và Hợp tác trong SRE (Communication and Collaboration in SRE)

> **Nguyên bản:** [Chapter 31 - Communication and Collaboration in SRE](https://sre.google/sre-book/communication-and-collaboration/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Đội biên tập Softdreams RnD)

---

*Tác giả:* Niall Murphy cùng Alex Rodriguez, Carl Crous, Dario Freni, Dylan Curley, Lorenzo Blanco, và Todd Underwood
*Biên tập:* Betsy Beyer

Vị thế tổ chức của SRE trong Google thú vị, và có các tác động đến cách chúng tôi giao tiếp và hợp tác.

Trước hết, có một sự đa dạng to lớn trong những gì SRE làm và cách chúng tôi làm. Chúng tôi có các đội hạ tầng, các đội dịch vụ và các đội sản phẩm ngang (horizontal). Mối quan hệ của chúng tôi với các đội phát triển sản phẩm trải dài từ những đội lớn gấp nhiều lần kích thước của chúng tôi, đến những đội có kích thước xấp xỉ bằng đối tác, và cả những tình huống mà chúng tôi *chính là* đội phát triển sản phẩm. Các đội SRE được tạo thành từ những người có kỹ năng kỹ thuật hệ thống hoặc kiến trúc (xem [[Hix15b]](https://sre.google/sre-book/bibliography#Hix15b)), kỹ năng kỹ thuật phần mềm, kỹ năng quản lý dự án, bản năng lãnh đạo, xuất thân từ đủ loại ngành (xem [Bài học Học được từ Các ngành khác](https://sre.google/sre-book/lessons-learned/)), và vân vân. Chúng tôi không chỉ có một mô hình, và đã tìm thấy nhiều cấu hình hoạt động tốt; sự linh hoạt này phù hợp với bản chất thực dụng của chúng tôi.

Cũng đúng là SRE không phải là một tổ chức ra lệnh và kiểm soát (command-and-control). Nói chung, chúng tôi nợ lòng trung thành cho ít nhất hai ông chủ: đối với các [đội SRE](https://sre.google/sre-book/part-II-principles/) dịch vụ hoặc hạ tầng, chúng tôi làm việc chặt chẽ với các đội phát triển sản phẩm tương ứng đang làm việc trên các dịch vụ hoặc hạ tầng đó; chúng tôi cũng rõ ràng làm việc trong ngữ cảnh của SRE nói chung. Mối quan hệ dịch vụ rất mạnh, vì chúng tôi chịu trách nhiệm về hiệu năng của các hệ thống đó, nhưng bất chấp điều này, các tuyến báo cáo thực tế của chúng tôi là thông qua SRE nói chung. Ngày nay, chúng tôi dành nhiều thời gian hơn để hỗ trợ các dịch vụ riêng lẻ của mình hơn là công việc production chéo, nhưng văn hóa và các giá trị chung của chúng tôi tạo ra các cách tiếp cận rất đồng nhất đối với các vấn đề. Đây là do thiết kế.<sup>[1](#fn1)</sup>

Hai sự thật trước đó đã định hướng tổ chức SRE theo một số hướng nhất định về hai khía cạnh quan trọng của cách các đội chúng tôi hoạt động — giao tiếp và hợp tác. Luồng dữ liệu là một ẩn dụ tính toán phù hợp cho giao tiếp của chúng tôi: giống như dữ liệu phải chảy quanh production, dữ liệu cũng phải chảy quanh một đội SRE — dữ liệu về các dự án, trạng thái của các dịch vụ, production và trạng thái của các cá nhân. Để đạt hiệu quả tối đa của một đội, dữ liệu phải chảy đáng tin cậy từ bên quan tâm này sang bên khác. Một cách để nghĩ về luồng này là nghĩ về giao diện mà một đội SRE phải thể hiện cho các đội khác, chẳng hạn như một API. Giống như một API, một thiết kế tốt là thiết yếu cho hoạt động hiệu quả, và nếu API sai, việc sửa sau đó có thể rất đau đớn.

Ẩn dụ API-như-hợp đồng cũng liên quan đến hợp tác, cả giữa các đội SRE lẫn giữa SRE và các đội phát triển sản phẩm — tất cả đều phải đạt được tiến bộ trong một môi trường thay đổi không ngừng. Ở mức độ đó, sự hợp tác của chúng tôi trông khá giống với sự hợp tác trong bất kỳ công ty nhanh chóng nào khác. Sự khác biệt là sự pha trộn các kỹ năng kỹ thuật phần mềm, chuyên môn kỹ thuật hệ thống và sự khôn ngoan từ kinh nghiệm production mà SRE áp dụng vào sự hợp tác đó. Các thiết kế và triển khai tốt nhất ra đời từ việc các mối quan tâm chung của production và sản phẩm gặp nhau trong một bầu không khí tôn trọng lẫn nhau. Đây là lời hứa của SRE: một tổ chức được giao trách nhiệm độ tin cậy, với cùng kỹ năng như các đội phát triển sản phẩm, sẽ cải thiện các chỉ số đo được. Kinh nghiệm của chúng tôi cho thấy đơn thuần có ai đó phụ trách độ tin cậy, mà không có cả bộ kỹ năng đầy đủ, là không đủ.

## Giao tiếp: Các Cuộc họp Production (Communications: Production Meetings)

Mặc dù văn học về việc điều hành các cuộc họp hiệu quả tràn ngập [[Kra08]](https://sre.google/sre-book/bibliography#Kra08), nhưng khó tìm ai đủ may mắn để *chỉ* có những cuộc họp hữu ích, hiệu quả. Điều này đúng như nhau đối với SRE.

Tuy nhiên, có một loại cuộc họp mà chúng tôi thấy hữu ích hơn mức trung bình, được gọi là *cuộc họp production*. Các cuộc họp production là một loại cuộc họp đặc biệt trong đó một đội SRE cẩn thận diễn đạt cho chính mình — và cho những khách mời — trạng thái của các dịch vụ trong sự giám hộ của họ, để tăng nhận thức chung giữa tất cả những người quan tâm và để cải thiện hoạt động của các dịch vụ. Nói chung, những cuộc họp này là *hướng dịch vụ*; chúng không trực tiếp về các cập nhật trạng thái của các cá nhân. Mục tiêu là để mọi người rời cuộc họp với cùng một ý tưởng về điều gì đang xảy ra. Mục tiêu lớn khác là cải thiện các dịch vụ của chúng tôi bằng cách áp dụng sự khôn ngoan của production. Điều đó có nghĩa là chúng tôi nói chi tiết về hiệu năng vận hành của dịch vụ, liên kết hiệu năng vận hành đó với thiết kế, cấu hình hoặc triển khai, và đưa ra các khuyến nghị về cách sửa các vấn đề. Việc liên kết hiệu năng của dịch vụ với các quyết định thiết kế trong một cuộc họp định kỳ là một vòng lặp phản hồi (feedback loop) vô cùng mạnh mẽ.

Các cuộc họp production của chúng tôi thường diễn ra hàng tuần; xét đến sự ghét bỏ của SRE đối với các cuộc họp vô ích, tần suất này có vẻ vừa phải đúng: đủ thời gian để cho phép đủ tài liệu liên quan tích lũy để khiến cuộc họp đáng giá, mà không thường xuyên đến mức mọi người tìm lý do để không tham dự. Chúng thường kéo dài khoảng 30 đến 60 phút. Ít hơn, có lẽ bạn đang cắt ngắn điều gì đó không cần thiết, hoặc có lẽ nên mở rộng danh mục dịch vụ của mình. Nhiều hơn, có lẽ bạn đang lún sâu vào chi tiết, hoặc có quá nhiều điều để nói và nên chia shard (chẻ nhỏ) đội hoặc tập hợp dịch vụ.

Giống như bất kỳ cuộc họp nào khác, cuộc họp production nên có một chủ tọa. Nhiều đội SRE luân phiên chủ tọa qua các thành viên đội, điều có lợi thế là làm cho mọi người cảm thấy họ có một phần trong dịch vụ và một số sự sở hữu lý thuyết đối với các vấn đề. Đúng là không phải ai cũng có mức độ kỹ năng chủ tọa bằng nhau, nhưng giá trị của sự sở hữu nhóm lớn đến mức sự đánh đổi của sự kém tối ưu tạm thời là đáng. Hơn nữa, đây là một cơ hội để gieo các kỹ năng chủ tọa, rất hữu ích trong loại tình huống phối hợp incident mà SRE thường gặp.

Trong các trường hợp hai đội SRE họp bằng video và một đội lớn hơn nhiều so với đội kia, chúng tôi đã nhận thấy một động lực thú vị đang diễn ra. Chúng tôi khuyến nghị đặt chủ tọa của bạn ở phía *nhỏ hơn* của cuộc gọi theo mặc định. Phía lớn hơn tự nhiên có xu hướng im lặng, và một số tác động xấu của kích thước đội không cân bằng (trở nên tồi tệ hơn do các độ trễ vốn có của hội nghị video) sẽ cải thiện.<sup>[2](#fn2)</sup> Chúng tôi không biết liệu kỹ thuật này có bất kỳ cơ sở khoa học nào không, nhưng nó có xu hướng hoạt động.

## Lịch trình (Agenda)

Có nhiều cách để điều hành một cuộc họp production, chứng thực cho sự đa dạng của những gì SRE trông nom và cách chúng tôi làm. Ở mức độ đó, không phù hợp để chỉ định cách điều hành một trong những cuộc họp này. Tuy nhiên, một lịch trình mặc định (xem [Phút ghi Hội họp Production Ví dụ](https://sre.google/sre-book/production-meeting/) cho một ví dụ) có thể trông như sau:

Các thay đổi production sắp tới

Các cuộc họp theo dõi thay đổi được biết đến rộng rãi trong suốt ngành, và thực sự các cuộc họp nguyên vẹn thường được dành riêng để dừng thay đổi. Tuy nhiên, trong môi trường production của chúng tôi, chúng tôi thường mặc định cho phép thay đổi, điều đòi hỏi việc theo dõi tập hợp các thuộc tính hữu ích của thay đổi đó: thời gian bắt đầu, thời lượng, hiệu ứng được kỳ vọng, và vân vân. Đây là khả năng nhìn thấy ở chân trời gần.

#### Metrics (Các chỉ số)

Một trong những cách chính mà chúng tôi tiến hành một cuộc thảo luận hướng dịch vụ là bằng cách nói về các metrics cốt lõi của các hệ thống được thảo luận; xem [Service Level Objectives](https://sre.google/sre-book/service-level-objectives/). Ngay cả khi các hệ thống không thất bại đáng kể trong tuần đó, rất phổ biến để ở trong vị trí mà bạn đang nhìn vào tải (load) tăng dần (hoặc đột ngột!) trong suốt năm. Việc theo dõi làm thế nào các con số độ trễ (latency) của bạn, các con số sử dụng CPU, v.v. thay đổi theo thời gian là vô cùng có giá trị cho việc phát triển một cảm giác về phong bì hiệu năng (performance envelope) của một hệ thống.

Một số đội theo dõi mức sử dụng và hiệu quả tài nguyên, đây cũng là một chỉ báo hữu ích cho các thay đổi hệ thống chậm hơn, có lẽ gian ác hơn.

#### Các Outage (Mất dịch vụ)

Mục này giải quyết các vấn đề có kích thước xấp xỉ postmortem, và là một cơ hội không thể thiếu để học hỏi. Một phân tích postmortem (báo cáo sau sự cố) tốt, như được thảo luận trong [Văn hóa Postmortem: Học hỏi từ Thất bại](https://sre.google/sre-book/postmortem-culture/), nên luôn luôn làm cho máu nóng chảy.

#### Các sự kiện Page (lỗi)

Đây là các page từ hệ thống giám sát (monitoring) của bạn, liên quan đến các vấn đề mà *có thể* đáng postmortem, nhưng thường thì không. Trong mọi trường hợp, trong khi phần Outages xem xét bức tranh lớn hơn của một outage, mục này xem xét góc nhìn chiến thuật: danh sách các page, ai đã bị page, điều gì xảy ra sau đó, và vân vân. Có hai câu hỏi ngầm cho mục này: cảnh báo đó có nên page theo cách mà nó đã làm không, và nó có nên page không? Nếu câu trả lời cho câu hỏi cuối cùng là không, hãy loại bỏ các page không thể hành động đó.

#### Các sự kiện không-Page (Nonpaging events)

Thùng chứa này chứa ba mục:

-   *Một vấn đề mà có lẽ đã nên page, nhưng không*. Trong những trường hợp này, bạn có lẽ nên sửa giám sát để các sự kiện như vậy thực sự kích hoạt một page. Thường bạn gặp phải vấn đề khi bạn đang cố sửa một thứ khác, hoặc nó liên quan đến một metric mà bạn đang theo dõi nhưng mà bạn không có một cảnh báo.
-   *Một vấn đề mà không thể page nhưng đòi hỏi sự chú ý*, chẳng hạn như hỏng dữ liệu (data corruption) tác động thấp hoặc sự chậm chạp trong một chiều không hướng người dùng của hệ thống. Việc theo dõi công việc vận hành phản ứng cũng phù hợp ở đây.
-   *Một vấn đề mà không thể page và không đòi hỏi sự chú ý*. Những cảnh báo này nên được loại bỏ, vì chúng tạo ra thêm tiếng ồn xao nhãng các kỹ sư khỏi các vấn đề mà xứng đáng sự chú ý.

Các action item (mục công việc) trước đó

Các cuộc thảo luận chi tiết trước đó thường dẫn đến các hành động mà SRE cần thực hiện — sửa cái này, giám sát cái kia, phát triển một hệ thống con để làm cái kia. Theo dõi các cải tiến này cũng giống như trong bất kỳ cuộc họp nào khác: gán các action item cho mọi người và theo dõi tiến độ của chúng. Là một ý tưởng tốt để có một mục lịch trình rõ ràng đóng vai trò như một cái bắt tất cả, nếu không có gì khác. Việc giao nhất quán cũng là một người xây dựng uy tín và niềm tin tuyệt vời. Không quan trọng việc giao được thực hiện như thế nào, chỉ cần nó *được* thực hiện.

## Sự Tham dự (Attendance)

Sự tham dự là bắt buộc cho tất cả các thành viên của đội SRE được thảo luận. Điều này đặc biệt đúng nếu đội của bạn trải dài trên nhiều quốc gia và/hoặc múi giờ, vì đây là cơ hội chính của bạn để tương tác như một nhóm.

Các bên liên quan chính cũng nên tham dự cuộc họp này. Bất kỳ đội phát triển sản phẩm đối tác nào mà bạn có thể có cũng nên tham dự. Một số đội SRE chia shard (chẻ nhỏ) cuộc họp của họ để các vấn đề chỉ-SRE được giữ trong nửa đầu; thực hành đó là ổn, miễn là mọi người, như đã nói trước đó, rời đi với cùng một ý tưởng về điều gì đang xảy ra. Đôi khi các đại diện từ các đội SRE khác có thể xuất hiện, đặc biệt nếu có một vấn đề chéo-đội lớn hơn để thảo luận, nhưng nói chung, đội SRE được thảo luận cùng các đội khác lớn nên tham dự. Nếu mối quan hệ của bạn như vậy mà bạn không thể mời các đối tác phát triển sản phẩm của bạn, bạn cần sửa mối quan hệ đó: có lẽ bước đầu tiên là mời một đại diện từ đội đó, hoặc tìm một trung gian tin cậy để ủy quyền truyền thông hoặc mô hình các tương tác lành mạnh. Có nhiều lý do tại sao các đội không hợp nhau, và một khối lượng lớn văn viết về cách giải quyết vấn đề đó: thông tin này cũng áp dụng cho các đội SRE, nhưng quan trọng là mục tiêu cuối cùng của việc có một vòng lặp phản hồi từ vận hành được đáp ứng, hoặc một phần lớn giá trị của việc có một đội SRE bị mất.

Đôi khi bạn sẽ có quá nhiều đội hoặc các người tham dự bận rộn-nhưng-cực kỳ quan trọng để mời. Có một số kỹ thuật bạn có thể sử dụng để xử lý những tình huống đó:

-   Các dịch vụ ít hoạt động hơn có thể được tham dự bởi một đại diện đơn lẻ từ đội phát triển sản phẩm, hoặc chỉ có cam kết từ đội phát triển sản phẩm để đọc và bình luận về phút ghi lịch trình.
-   Nếu đội phát triển production khá lớn, hãy đề cử một tập hợp con các đại diện.
-   Các người tham dự bận rộn-nhưng-cực kỳ quan trọng có thể cung cấp phản hồi và/hoặc định hướng trước cho các cá nhân, hoặc sử dụng kỹ thuật lịch trình được điền trước (được mô tả tiếp theo).

Phần lớn các chiến lược cuộc họp mà chúng tôi đã thảo luận là lẽ thường, với một nét hướng dịch vụ. Một nét xoay độc đáo trong việc làm cho các cuộc họp hiệu quả hơn *và* bao gồm hơn là sử dụng các tính năng hợp tác thời gian thực của Google Docs. Nhiều đội SRE có một doc (tài liệu) như vậy, với một địa chỉ được biết đến rộng rãi mà bất kỳ ai trong kỹ thuật cũng có thể truy cập. Việc có một doc như vậy cho phép hai thực hành tuyệt vời:

-   Điền trước lịch trình với các ý tưởng, bình luận, và thông tin "từ dưới lên."
-   Chuẩn bị lịch trình song song *và* trước là thực sự hiệu quả.

Sử dụng đầy đủ các tính năng hợp tác nhiều người do sản phẩm cho phép. Không có gì sánh bằng việc thấy một chủ tọa cuộc họp gõ một câu, sau đó thấy ai đó cung cấp một liên kết đến tài liệu nguồn trong ngoặc sau khi họ đã hoàn thành việc gõ, và sau đó thấy một người khác dọn dẹp chính tả và ngữ pháp trong câu gốc. Sự hợp tác như vậy khiến các thứ được hoàn thành nhanh hơn, và làm cho nhiều người cảm thấy họ sở hữu một lát cắt của những gì đội làm.

## Hợp tác trong SRE (Collaboration within SRE)

Rõ ràng, Google là một tổ chức đa quốc gia. Vì thành phần phản ứng khẩn cấp và vòng trực pager của vai trò chúng tôi, chúng tôi có các lý do kinh doanh rất tốt để là một tổ chức phân tán, được tách ra bởi ít nhất một vài múi giờ. Tác động thực tế của sự phân tán này là chúng tôi có các định nghĩa rất lưu động cho "đội" so với, ví dụ, đội phát triển sản phẩm trung bình. Chúng tôi có các đội cục bộ, đội trên site, đội chéo châu lục, các đội ảo với đủ kích thước và sự nhất quán, và tất cả mọi thứ ở giữa. Điều này tạo ra một sự pha trộn hỗn loạn vui vẻ của các trách nhiệm, kỹ năng và cơ hội. Phần lớn các động lực tương tự có thể được kỳ vọng áp dụng cho bất kỳ công ty đủ lớn nào (mặc dù chúng có thể đặc biệt gay gắt cho các công ty công nghệ). Vì phần lớn sự hợp tác cục bộ không gặp trở ngại nào đặc biệt, trường hợp thú vị về mặt hợp tác là chéo-đội, chéo-site, xuyên một đội ảo, và tương tự.

Mô hình phân tán này cũng định hình cách các đội SRE có xu hướng được tổ chức. Vì *sứ mệnh tồn tại* (raison d'être) của chúng tôi là mang giá trị thông qua sự làm chủ kỹ thuật, và sự làm chủ kỹ thuật có xu hướng khó khăn, vì vậy chúng tôi cố gắng tìm một cách để có sự làm chủ đối với một tập hợp con liên quan các hệ thống hoặc hạ tầng, để giảm tải nhận thức. Chuyên môn hóa là một cách để đạt được mục tiêu này; tức là, đội X chỉ làm việc trên sản phẩm Y. Chuyên môn hóa là tốt, vì nó dẫn đến cơ hội cao hơn cho sự làm chủ kỹ thuật được cải thiện, nhưng nó cũng xấu, vì nó dẫn đến sự ngăn cách (siloization) và sự ngu dốt về bức tranh rộng hơn. Chúng tôi cố gắng có một hiến chương đội rõ ràng để định nghĩa điều gì mà một đội sẽ — và quan trọng hơn, sẽ *không* — hỗ trợ, nhưng chúng tôi không luôn luôn thành công.

## Thành phần Đội (Team Composition)

Chúng tôi có một loạt rộng các bộ kỹ năng trong SRE, trải dài từ kỹ thuật hệ thống qua kỹ thuật phần mềm, và đến tổ chức và quản lý. Một điều chúng tôi có thể nói về hợp tác là cơ hội của bạn để hợp tác thành công — và thực sự gần như bất kỳ thứ gì khác — được cải thiện bằng cách có nhiều sự đa dạng hơn trong đội của bạn. Có một khối lượng bằng chứng cho thấy các đội đa dạng đơn thuần là các đội tốt hơn [[Nel14]](https://sre.google/sre-book/bibliography#Nel14). Việc điều hành một đội đa dạng ngụ ý sự chú ý đặc biệt đến giao tiếp, các thiên kiến nhận thức (cognitive biases), và vân vân, điều mà chúng tôi không thể bao gồm chi tiết ở đây.

Chính thức, các đội SRE có các vai trò "tech lead" (TL), "manager" (quản lý, SRM) và "project manager" (quản lý dự án, còn gọi là PM, TPM, PgM). Một số người hoạt động tốt nhất khi những vai trò đó có các trách nhiệm được định nghĩa rõ ràng: lợi ích chính là họ có thể đưa ra các quyết định trong phạm vi một cách nhanh chóng và an toàn. Những người khác hoạt động tốt nhất trong một môi trường lưu động hơn, với các trách nhiệm thay đổi tùy theo thương lượng động. Nói chung, đội càng lưu động, nó càng phát triển về mặt năng lực của các cá nhân, và càng có khả năng thích ứng với các tình huống mới — nhưng với cái giá là phải giao tiếp ngày càng thường xuyên hơn, vì ít bối cảnh hơn có thể được giả định.

Bất kể những vai trò này được định nghĩa tốt như thế nào, ở mức cơ bản, tech lead chịu trách nhiệm cho hướng kỹ thuật trong đội, và có thể dẫn dắt theo nhiều cách — từ việc cẩn thận bình luận code của mọi người, đến việc giữ các bài trình bày hướng hàng quý, đến việc xây dựng sự đồng thuận trong đội. Tại Google, các TL có thể làm gần như tất cả công việc của một quản lý, vì các quản lý của chúng tôi rất kỹ thuật, nhưng quản lý có hai trách nhiệm đặc biệt mà một TL không có: chức năng quản lý hiệu năng, và là một cái bắt tất cả chung cho mọi thứ không được ai khác xử lý. Các TL, SRM và TPM tuyệt vời có một bộ kỹ năng đầy đủ và có thể vui vẻ xoay tay để tổ chức một dự án, bình luận về một doc thiết kế, hoặc viết code khi cần.

## Các Kỹ thuật để Làm việc Hiệu quả (Techniques for Working Effectively)

Có một số cách để kỹ thuật một cách hiệu quả trong SRE.

Nói chung, các dự án đơn (singleton) thất bại trừ khi người đó đặc biệt có tài hoặc vấn đề là đơn giản. Để đạt được bất kỳ thứ gì đáng kể, bạn khá cần nhiều người. Vì vậy, bạn cũng cần các kỹ năng hợp tác tốt. Một lần nữa, nhiều tài liệu đã được viết về chủ đề này, và phần lớn văn học này áp dụng cho SRE.

Nói chung, công việc SRE tốt đòi hỏi các kỹ năng giao tiếp xuất sắc khi bạn làm việc ngoài ranh giới của đội cục bộ thuần túy của bạn. Đối với các hợp tác ngoài tòa nhà, việc làm việc hiệu quả xuyên múi giờ ngụ ý hoặc giao tiếp viết tuyệt vời, hoặc nhiều đi lại để cung cấp trải nghiệm trực tiếp có thể hoãn lại nhưng cuối cùng là cần thiết cho một mối quan hệ chất lượng cao. Ngay cả khi bạn là một người viết tuyệt vời, theo thời gian bạn suy giảm thành chỉ đơn thuần là một địa chỉ email cho đến khi bạn xuất hiện bằng xác thịt một lần nữa.

## Nghiên cứu Tình huống về Hợp tác trong SRE: Viceroy (Case Study of Collaboration in SRE: Viceroy)

Một ví dụ về một hợp tác chéo-SRE thành công là một dự án gọi là Viceroy, một framework và dịch vụ dashboard giám sát. Kiến trúc tổ chức hiện tại của SRE có thể dẫn đến các đội tạo ra nhiều bản sao hơi khác nhau của cùng một phần công việc; vì nhiều lý do, các framework dashboard giám sát là mảnh đất màu mỡ đặc biệt cho sự sao chép công việc.<sup>[3](#fn3)</sup>

Các động lực dẫn đến vấn đề rác thải nghiêm trọng — nhiều xác tàu khung giám sát đang cháy âm ỉ, bị bỏ hoang nằm rải rác — khá đơn giản: mỗi đội được phần thưởng cho việc phát triển giải pháp của riêng họ, làm việc ngoài ranh giới đội là khó khăn, và hạ tầng thường được cung cấp trên toàn SRE thường gần hơn với một bộ công cụ (toolkit) hơn là một sản phẩm. Môi trường này khuyến khích các kỹ sư cá nhân dùng bộ công cụ để tạo ra một đống cháy khác thay vì sửa vấn đề cho số lượng người lớn nhất có thể (một nỗ lực do đó sẽ mất nhiều thời gian hơn).

## Sự Đến của Viceroy (The Coming of the Viceroy)

Viceroy là khác. Nó bắt đầu vào năm 2012 khi một số đội đang xem xét cách chuyển sang Monarch, hệ thống giám sát mới tại Google. SRE cực kỳ bảo thủ đối với các hệ thống giám sát, nên Monarch hơi mỉa mai mất một thời gian dài hơn để đạt được đà trong SRE so với trong các đội không-SRE. Nhưng không ai có thể tranh luận rằng hệ thống giám sát kế thừa của chúng tôi, Borgmon (xem [Cảnh báo Thực tiễn từ Dữ liệu Chuỗi thời gian](https://sre.google/sre-book/practical-alerting/)), không có chỗ để cải thiện. Ví dụ, các console (bảng điều khiển) của chúng tôi cồng kềnh vì chúng sử dụng một hệ thống template (mẫu) HTML tùy chỉnh được xử lý đặc biệt, đầy các trường hợp biên kỳ lạ, và khó kiểm thử. Vào thời điểm đó, Monarch đã trưởng thành đủ để được chấp nhận về nguyên tắc như giải pháp thay thế cho hệ thống kế thừa và do đó đang được các đội ngày càng nhiều trên Google áp dụng, nhưng hóa ra chúng tôi vẫn có vấn đề với các console.

Những người trong chúng tôi đã thử sử dụng Monarch cho các dịch vụ của mình sớm phát hiện ra rằng nó thiếu sót trong hỗ trợ console vì hai lý do chính:

-   Các console dễ thiết lập cho một dịch vụ nhỏ, nhưng không mở rộng tốt đến các dịch vụ với các console phức tạp.
-   Chúng cũng không hỗ trợ hệ thống giám sát kế thừa, làm cho việc chuyển sang Monarch rất khó khăn.

Vì không có lựa chọn thay thế khả thi nào để triển khai Monarch theo cách này vào thời điểm đó, một số dự án cụ thể cho đội đã khởi động. Vì có rất ít giải pháp phát triển có phối hợp hoặc thậm chí theo dõi chéo-nhóm vào thời điểm đó (một vấn đề đã được sửa kể từ đó), chúng tôi cuối cùng lại sao chép nỗ lực. Nhiều đội từ Spanner, Ads Frontend (Mặt trước Quảng cáo), và một loạt các dịch vụ khác đã khởi động nỗ lực của riêng họ (một ví dụ đáng chú ý được gọi là Consoles++) trong suốt 12–18 tháng, và cuối cùng sự hợp lý đã lên ngôi khi các kỹ sư từ tất cả những đội đó tỉnh dậy và phát hiện ra nỗ lực tương ứng của nhau. Họ quyết định làm điều hợp lý và hợp lực để tạo ra một giải pháp chung cho tất cả SRE. Như vậy, dự án Viceroy được sinh ra vào giữa năm 2012.

Vào đầu năm 2013, Viceroy bắt đầu thu hút sự quan tâm từ các đội vẫn chưa chuyển khỏi hệ thống kế thừa, nhưng đang tìm cách đặt một chân vào nước. Rõ ràng, các đội có các dự án giám sát hiện có lớn hơn thì ít động lực hơn để chuyển sang hệ thống mới: khó cho những đội này biện minh cho việc vứt bỏ chi phí bảo trì thấp của giải pháp hiện có — vốn về cơ bản hoạt động ổn — để đổi lấy một thứ tương đối mới, chưa được chứng minh và đòi hỏi nhiều nỗ lực. Sự đa dạng đơn thuần của các yêu cầu đã thêm vào sự do dự của những đội này, ngay cả khi tất cả các dự án console giám sát chia sẻ hai yêu cầu chính:

-   Hỗ trợ các dashboard đã được tuyển chọn phức tạp
-   Hỗ trợ cả Monarch và hệ thống giám sát kế thừa

Mỗi dự án *cũng* có một tập hợp các yêu cầu kỹ thuật của riêng nó, phụ thuộc vào sở thích hoặc kinh nghiệm của tác giả. Ví dụ:

-   Nhiều nguồn dữ liệu bên ngoài các hệ thống giám sát cốt lõi
-   Định nghĩa các console bằng cấu hình (configuration) so với layout (bố cục) HTML rõ ràng
-   Không JavaScript so với sự áp dụng đầy đủ JavaScript với AJAX
-   Sử dụng đơn thuần nội dung tĩnh, để các console có thể được cache (lưu bộ nhớ đệm) trong trình duyệt

Mặc dù một số yêu cầu này dai dẳng hơn những cái khác, tổng thể chúng làm cho việc gộp các nỗ lực trở nên khó khăn. Thật vậy, dù đội Consoles++ quan tâm đến việc xem dự án của họ so với Viceroy như thế nào, cuộc xem xét ban đầu của họ vào nửa đầu năm 2013 xác định rằng các khác biệt cơ bản giữa hai dự án đủ lớn để ngăn chặn tích hợp. Khó khăn lớn nhất là Viceroy theo thiết kế không sử dụng nhiều JavaScript, trong khi Consoles++ hầu như được viết bằng JavaScript. Tuy nhiên, có một tia hy vọng ở chỗ hai hệ thống thực sự có một số điểm tương đồng cơ bản:

-   Chúng sử dụng các cú pháp tương tự cho việc render (hiển thị) template HTML.
-   Chúng chia sẻ một số mục tiêu dài hạn mà cả hai đội đều chưa bắt đầu giải quyết. Ví dụ, cả hai hệ thống đều muốn cache dữ liệu giám sát và hỗ trợ một pipeline (luồng) ngoại tuyến để định kỳ tạo ra dữ liệu mà console có thể sử dụng, nhưng quá tốn tính toán để tạo ra theo yêu cầu.

Chúng tôi cuối cùng đã tạm dừng cuộc thảo luận console thống nhất trong một thời gian. Tuy nhiên, vào cuối năm 2013, cả Consoles++ và Viceroy đều đã phát triển đáng kể. Các khác biệt kỹ thuật của chúng thu hẹp, vì Viceroy bắt đầu sử dụng JavaScript để render các biểu đồ giám sát. Hai đội gặp nhau và phát hiện ra rằng việc tích hợp dễ dàng hơn nhiều, giờ việc tích hợp quy về việc phục vụ dữ liệu Consoles++ ra từ server Viceroy. Các nguyên mẫu tích hợp đầu tiên được hoàn thành vào đầu năm 2014, chứng minh rằng các hệ thống có thể hoạt động tốt cùng nhau. Cả hai đội cảm thấy thoải mái cam kết cho một nỗ lực chung vào thời điểm đó, và vì Viceroy đã thiết lập thương hiệu của nó như một giải pháp giám sát chung, dự án kết hợp giữ tên Viceroy. Việc phát triển đầy đủ chức năng mất một vài quý, nhưng vào cuối năm 2014, hệ thống kết hợp đã hoàn thành.

Việc hợp lực đã gặt hái những lợi ích to lớn:

-   Viceroy nhận được một loạt các nguồn dữ liệu và các client (khách hàng) JavaScript để truy cập chúng.
-   Biên dịch (compilation) JavaScript được viết lại để hỗ trợ các module (mô-đun) riêng biệt có thể được chọn bao gồm. Đây là thiết yếu để mở rộng hệ thống đến bất kỳ số lượng đội nào với code JavaScript của riêng họ.
-   Consoles++ được hưởng lợi từ nhiều cải tiến đang được tích cực thực hiện cho Viceroy, chẳng hạn như việc thêm cache và pipeline dữ liệu nền của nó.
-   Tổng thể, tốc độ phát triển trên *một* giải pháp lớn hơn nhiều so với tổng của tất cả tốc độ phát triển của các dự án sao chép.

Cuối cùng, tầm nhìn tương lai chung là yếu tố then chốt để kết hợp các dự án. Cả hai đội đều thấy giá trị trong việc mở rộng đội phát triển và được hưởng lợi từ các đóng góp của nhau. Động lực đến mức, vào cuối năm 2014, Viceroy được chính thức tuyên bố là giải pháp giám sát chung cho tất cả SRE. Có lẽ đặc trưng cho Google, tuyên bố này không đòi hỏi các đội áp dụng Viceroy: thay vào đó, nó khuyến nghị rằng các đội nên dùng Viceroy thay vì viết một console giám sát khác.

## Các Thách thức (Challenges)

Mặc dù cuối cùng là thành công, Viceroy không phải không có khó khăn, và nhiều trong số đó nảy sinh do bản chất chéo-site của dự án.

Một khi đội Viceroy mở rộng được thiết lập, sự phối hợp ban đầu giữa các thành viên đội từ xa chứng tỏ là khó khăn. Khi gặp mọi người lần đầu tiên, các tín hiệu tinh tế trong viết và nói có thể bị hiểu sai, vì các phong cách truyền thông thay đổi đáng kể từ người này sang người khác. Vào đầu dự án, các thành viên đội không có mặt ở Mountain View cũng bỏ lỡ các cuộc thảo luận bàn nước đột xuất thường xảy ra ngay trước và sau các cuộc họp (mặc dù truyền thông kể từ đó đã cải thiện đáng kể).

Trong khi đội Viceroy cốt lõi vẫn khá ổn định, đội mở rộng của các người đóng góp khá động. Các người đóng góp có các trách nhiệm khác thay đổi theo thời gian, và do đó nhiều người có thể dành từ một đến ba tháng cho dự án. Như vậy, hồ người đóng góp nhà phát triển, vốn tự nhiên lớn hơn đội Viceroy cốt lõi, được đặc trưng bởi một lượng churn (chuyển đổi) đáng kể.

Thêm người mới vào dự án đòi hỏi đào tạo mỗi người đóng góp về thiết kế tổng thể và cấu trúc của hệ thống, điều mất một ít thời gian. Mặt khác, khi một SRE đóng góp cho chức năng cốt lõi của Viceroy và sau đó trở về đội của họ, họ là một chuyên gia cục bộ về hệ thống. Sự phân tán không lường trước của các chuyên gia Viceroy cục bộ này thúc đẩy nhiều hơn việc sử dụng và áp dụng.

Khi mọi người tham gia và rời khỏi đội, chúng tôi phát hiện ra rằng các đóng góp tùy tiện vừa hữu ích vừa tốn kém. Chi phí chính là sự pha loãng sự sở hữu: một khi các tính năng được giao và người đó rời đi, các tính năng dần trở nên không được hỗ trợ theo thời gian và thường bị bỏ.

Hơn nữa, phạm vi của dự án Viceroy mở rộng theo thời gian. Nó có các mục tiêu tham vọng khi khởi động nhưng phạm vi ban đầu bị hạn chế. Tuy nhiên, khi phạm vi mở rộng, chúng tôi vật lộn để giao các tính năng cốt lõi đúng hạn, và phải cải thiện quản lý dự án, đặt ra một hướng rõ ràng hơn để đảm bảo dự án vẫn trên đường ray.

Cuối cùng, đội Viceroy phát hiện ra khó khăn để hoàn toàn sở hữu một thành phần có các đóng góp đáng kể (quyết định) từ các site phân tán. Ngay cả với thiện chí tốt nhất trên thế giới, mọi người thường mặc định theo đường có ít kháng cự nhất và thảo luận các vấn đề hoặc đưa ra các quyết định cục bộ mà không liên quan đến các chủ sở hữu từ xa, điều có thể dẫn đến xung đột.

## Các Khuyến nghị (Recommendations)

Bạn chỉ nên phát triển các dự án chéo-site khi bắt buộc, nhưng thường có những lý do tốt để làm vậy. Chi phí làm việc xuyên site là độ trễ cao hơn cho các hành động và đòi hỏi nhiều truyền thông hơn; lợi ích là — nếu bạn làm đúng các cơ chế — thông lượng (throughput) cao hơn nhiều. Dự án đơn-site cũng có thể mắc phải tình trạng không ai bên ngoài site đó biết bạn đang làm gì, nên có chi phí cho cả hai cách tiếp cận.

Các người đóng góp có động lực là có giá trị, nhưng không phải tất cả các đóng góp đều có giá trị như nhau. Hãy đảm bảo rằng các người đóng góp dự án thực sự cam kết, và không chỉ tham gia với một mục tiêu tự hiện thực hóa mơ hồ (muốn gắn tên của họ vào một dự án bóng bẩy; muốn code trên một dự án mới thú vị mà không cam kết bảo trì dự án đó). Các người đóng góp có một mục tiêu cụ thể để đạt được thường có động lực tốt hơn và sẽ bảo trì các đóng góp của họ tốt hơn.

Khi các dự án phát triển, chúng thường mở rộng, và bạn không phải lúc nào cũng ở vị trí may mắn có những người trong đội cục bộ của bạn để đóng góp cho dự án. Vì vậy, hãy suy nghĩ cẩn thận về cấu trúc dự án. Các lãnh đạo dự án là quan trọng: họ cung cấp tầm nhìn dài hạn cho dự án và đảm bảo tất cả công việc phù hợp với tầm nhìn đó và được ưu tiên hóa đúng. Bạn cũng cần có một cách được đồng ý để đưa ra các quyết định, và nên đặc biệt tối ưu hóa để đưa ra nhiều quyết định cục bộ hơn nếu có một mức độ đồng ý và tin tưởng cao.

Chiến lược "chia để trị" tiêu chuẩn áp dụng cho các dự án chéo-site; bạn giảm các chi phí truyền thông chủ yếu bằng cách chia dự án thành nhiều thành phần có kích thước hợp lý nhất có thể, và cố gắng đảm bảo rằng mỗi thành phần có thể được gán cho một nhóm nhỏ, ưu tiên là trong một site. Chia những thành phần này giữa các nhóm con của dự án, và thiết lập các sản phẩm giao và hạn chót rõ ràng. (Cố gắng đừng để định luật Conway làm méo hình dạng tự nhiên của phần mềm quá sâu.)<sup>[4](#fn4)</sup>

Một mục tiêu cho một đội dự án hoạt động tốt nhất khi nó hướng đến việc cung cấp một số chức năng hoặc giải quyết một số vấn đề. Cách tiếp cận này đảm bảo rằng các cá nhân làm việc trên một thành phần biết điều gì được kỳ vọng từ họ, và rằng công việc của họ chỉ hoàn thành một khi thành phần đó được tích hợp hoàn toàn và được sử dụng trong dự án chính.

Rõ ràng, các thực hành kỹ thuật tốt nhất thông thường áp dụng cho các dự án hợp tác: mỗi thành phần nên có các doc thiết kế và các review với đội. Bằng cách này, mọi người trong đội được trao cơ hội để cập nhật về các thay đổi, ngoài cơ hội để ảnh hưởng và cải thiện các thiết kế. Việc viết ra là một trong những kỹ thuật chính mà bạn có để bù đắp khoảng cách vật lý và/hoặc logic — hãy sử dụng nó.

Các tiêu chuẩn là quan trọng. Các hướng dẫn phong cách code là một khởi đầu tốt, nhưng chúng thường khá chiến thuật và do đó chỉ là một điểm khởi đầu để thiết lập các chuẩn mực của đội. Mỗi khi có một cuộc tranh luận về việc chọn lựa nào để thực hiện đối với một vấn đề, hãy tranh luận hết cỡ với đội nhưng với một giới hạn thời gian nghiêm ngặt. Sau đó chọn một giải pháp, tài liệu hóa nó, và chuyển sang. Nếu bạn không thể đồng ý, bạn cần chọn một trọng tài mà mọi người tôn trọng, và một lần nữa đơn thuần tiến về phía trước. Theo thời gian bạn sẽ xây dựng một tập hợp các thực hành tốt nhất này, điều sẽ giúp mọi người mới theo kịp.

Cuối cùng, không có gì thay thế được tương tác trực tiếp, mặc dù một phần tương tác mặt-đối-mặt có thể được hoãn lại bằng cách sử dụng tốt VC (hội nghị video) và giao tiếp viết tốt. Nếu bạn có thể, hãy để các lãnh đạo của dự án gặp phần còn lại của đội trực tiếp. Nếu thời gian và ngân sách cho phép, hãy tổ chức một hội nghị đỉnh (summit) đội để tất cả các thành viên của đội có thể tương tác trực tiếp. Một summit cũng cung cấp một cơ hội tuyệt vời để bàn bạc các thiết kế và mục tiêu. Đối với các tình huống mà tính trung lập là quan trọng, có lợi thế khi tổ chức các summit đội ở một vị trí trung lập để không site cá nhân nào có "lợi thế sân nhà."

Cuối cùng, hãy sử dụng phong cách quản lý dự án phù hợp với dự án ở trạng thái hiện tại của nó. Ngay cả các dự án với các mục tiêu tham vọng cũng sẽ bắt đầu nhỏ, nên chi phí phát sinh nên tương ứng thấp. Khi dự án lớn lên, phù hợp để thích ứng và thay đổi cách dự án được quản lý. Với sự tăng trưởng đủ, quản lý dự án hoàn chỉnh sẽ là cần thiết.

## Hợp tác Bên ngoài SRE (Collaboration Outside SRE)

Như chúng tôi đã gợi ý, và [Mô hình Tham gia SRE đang Tiến hóa](https://sre.google/sre-book/evolving-sre-engagement-model/) thảo luận, sự hợp tác giữa tổ chức phát triển sản phẩm và SRE thực sự ở trạng thái tốt nhất khi nó xảy ra sớm trong giai đoạn thiết kế, lý tưởng nhất là trước khi bất kỳ dòng code nào được commit. Các SRE ở vị trí tốt nhất để đưa ra các khuyến nghị về kiến trúc và hành vi phần mềm mà có thể khá khó khăn (nếu không phải bất khả thi) để lắp ráp sau. Việc có giọng nói đó hiện diện trong phòng khi một hệ thống mới được thiết kế là tốt hơn cho mọi người. Nói chung, chúng tôi sử dụng quy trình Objectives & Key Results (OKR) [[Kla12]](https://sre.google/sre-book/bibliography#Kla12) để theo dõi các công việc như vậy. Đối với một số đội dịch vụ, sự hợp tác như vậy là trụ cột của những gì họ làm — theo dõi các thiết kế mới, đưa ra các khuyến nghị, giúp triển khai chúng, và đưa chúng đến production.

## Nghiên cứu Tình huống: Chuyển DFP sang F1 (Case Study: Migrating DFP to F1)

Các dự án chuyển đổi lớn của các dịch vụ hiện có khá phổ biến tại Google. Các ví dụ điển hình bao gồm port (chuyển) các thành phần dịch vụ sang một công nghệ mới hoặc cập nhật các thành phần để hỗ trợ một định dạng dữ liệu mới. Với sự giới thiệu gần đây của các công nghệ cơ sở dữ liệu có thể mở rộng đến mức toàn cầu như Spanner [[Cor12]](https://sre.google/sre-book/bibliography#Cor12) và F1 [[Shu13]](https://sre.google/sre-book/bibliography#Shu13), Google đã thực hiện một số dự án chuyển đổi quy mô lớn liên quan đến cơ sở dữ liệu. Một trong những dự án như vậy là việc chuyển đổi cơ sở dữ liệu chính của DoubleClick for Publishers (DFP)<sup>[5](#fn5)</sup> từ MySQL sang F1. Cụ thể, một số tác giả của chương này chịu trách nhiệm cho một phần của hệ thống phục vụ (được hiển thị trong [Hình 31-1](#hinh-31-1)) liên tục trích xuất và xử lý dữ liệu từ cơ sở dữ liệu, để tạo ra một tập hợp các file index (bảng mục lục) sau đó được nạp và phục vụ trên toàn thế giới. Hệ thống này được phân tán trên một số datacenter (trung tâm dữ liệu) và sử dụng khoảng 1.000 CPU và 8 TB RAM để index 100 TB dữ liệu mỗi ngày.

<a id="hinh-31-1"></a>        ![A generic ads serving system.](../assets/imgs/fig-31-1.jpg)

**Hình 31-1.** Một hệ thống phục vụ quảng cáo chung

Việc chuyển đổi không tầm thường: ngoài việc chuyển sang một công nghệ mới, schema cơ sở dữ liệu được refactor và đơn giản hóa đáng kể nhờ khả năng của F1 để lưu trữ và index dữ liệu protocol buffer trong các cột bảng. Mục tiêu là chuyển đổi hệ thống xử lý để nó có thể tạo ra một đầu ra hoàn toàn giống hệt với hệ thống hiện có. Điều này cho phép chúng tôi giữ hệ thống phục vụ không thay đổi và thực hiện, từ góc nhìn của người dùng, một chuyển đổi liền mạch. Như một ràng buộc bổ sung, sản phẩm đòi hỏi rằng chúng tôi hoàn thành một chuyển đổi trực mà không có sự gián đoạn nào của dịch vụ đối với người dùng vào bất kỳ lúc nào. Để đạt được điều này, đội phát triển sản phẩm và đội SRE bắt đầu làm việc chặt chẽ ngay từ đầu để phát triển dịch vụ index mới.

Với tư cách là các nhà phát triển chính, các đội phát triển sản phẩm thường quen thuộc hơn với Logic Kinh doanh (Business Logic — BL) của phần mềm, và cũng ở gần hơn với các Product Manager (Quản lý Sản phẩm) và thành phần "nhu cầu kinh doanh" thực tế của các sản phẩm. Mặt khác, các đội SRE thường có nhiều chuyên môn hơn liên quan đến các thành phần hạ tầng của phần mềm (ví dụ, các thư viện để nói chuyện với các hệ thống lưu trữ phân tán hoặc cơ sở dữ liệu), vì các SRE thường tái sử dụng cùng các khối xây dựng xuyên qua các dịch vụ khác nhau, học được nhiều lưu ý và sắc thái cho phép phần mềm chạy có khả năng mở rộng và đáng tin cậy theo thời gian.

Từ đầu dự án chuyển đổi, phát triển sản phẩm và SRE biết rằng họ sẽ phải hợp tác chặt chẽ hơn, tiến hành các cuộc họp hàng tuần để đồng bộ về tiến độ của dự án. Trong trường hợp cụ thể này, các thay đổi BL một phần phụ thuộc vào các thay đổi hạ tầng. Vì lý do này, dự án bắt đầu với thiết kế của hạ tầng mới; các SRE, những người có kiến thức sâu rộng về lĩnh vực trích xuất và xử lý dữ liệu ở quy mô, đã dẫn dắt thiết kế của các thay đổi hạ tầng. Điều này bao gồm việc thiết kế cách trích xuất các bảng khác nhau từ F1, cách lọc và join (kết nối) dữ liệu, cách chỉ trích xuất dữ liệu đã thay đổi (so với toàn bộ cơ sở dữ liệu), cách duy trì khi mất một số machine (máy) mà không ảnh hưởng đến dịch vụ, cách đảm bảo rằng mức sử dụng tài nguyên tăng tuyến tính với lượng dữ liệu trích xuất, lập kế hoạch năng lực, và nhiều khía cạnh tương tự khác. Hạ tầng mới được đề xuất tương tự các dịch vụ khác đã đang trích xuất và xử lý dữ liệu từ F1. Do đó, chúng tôi có thể chắc chắn về tính vững chắc của giải pháp và tái sử dụng các phần của giám sát và công cụ.

Trước khi tiếp tục phát triển hạ tầng mới này, hai SRE đã tạo ra một doc thiết kế chi tiết. Sau đó, cả hai đội phát triển sản phẩm và SRE xem xét kỹ lưỡng doc, chỉnh sửa giải pháp để xử lý một số trường hợp biên, và cuối cùng đồng ý về một kế hoạch thiết kế. Một kế hoạch như vậy xác định rõ ràng loại thay đổi mà hạ tầng mới sẽ mang đến cho BL. Ví dụ, chúng tôi thiết kế hạ tầng mới để chỉ trích xuất dữ liệu đã thay đổi, thay vì lặp đi lặp lại trích xuất toàn bộ cơ sở dữ liệu; BL phải tính đến cách tiếp cận mới này. Sớm, chúng tôi định nghĩa các giao diện mới giữa hạ tầng và BL, và việc làm như vậy cho phép đội phát triển sản phẩm làm việc độc lập trên các thay đổi BL. Tương tự, đội phát triển sản phẩm giữ cho SRE được thông báo về các thay đổi BL. Nơi chúng tương tác (ví dụ, các thay đổi BL phụ thuộc vào hạ tầng), cấu trúc phối hợp này cho phép chúng tôi biết các thay đổi đang xảy ra, và xử lý chúng nhanh chóng và đúng đắn.

Trong các giai đoạn sau của dự án, các SRE bắt đầu triển khai dịch vụ mới trong một môi trường kiểm thử giống với môi trường production hoàn thành cuối cùng của dự án. Bước này là thiết yếu để đo lường hành vi được kỳ vọng của dịch vụ — đặc biệt là hiệu năng và mức sử dụng tài nguyên — trong khi việc phát triển BL vẫn đang diễn ra. Đội phát triển sản phẩm sử dụng môi trường kiểm thử này để thực hiện xác thực dịch vụ mới: index của các quảng cáo được tạo ra bởi dịch vụ cũ (chạy trong production) phải khớp hoàn hảo với index được tạo ra bởi dịch vụ mới (chạy trong môi trường kiểm thử). Như đã nghi ngờ, quy trình xác thực đã làm nổi bật các khác biệt giữa các dịch vụ cũ và mới (do một số trường hợp biên trong định dạng dữ liệu mới), mà đội phát triển sản phẩm đã có thể giải quyết theo vòng lặp: cho mỗi quảng cáo họ debug nguyên nhân của sự khác biệt và sửa BL tạo ra đầu ra xấu. Trong khi đó, đội SRE bắt đầu chuẩn bị môi trường production: phân bổ các tài nguyên cần thiết trong một datacenter khác, thiết lập các quy trình và rule giám sát, và đào tạo các kỹ sư được chỉ định để on-call cho dịch vụ. Đội SRE cũng thiết lập một quy trình release cơ bản bao gồm xác thực, một task thường được hoàn thành bởi đội phát triển sản phẩm hoặc bởi các Release Engineer (Kỹ sư Phát hành) nhưng trong trường hợp cụ thể này được hoàn thành bởi các SRE để tăng tốc chuyển đổi.

Khi dịch vụ sẵn sàng, các SRE đã chuẩn bị một kế hoạch rollout (phát hành) trong hợp tác với đội phát triển sản phẩm và khởi động dịch vụ mới. Việc khởi động rất thành công và diễn ra suôn sẻ, không có tác động nhìn thấy được nào đối với người dùng.

## Kết luận (Conclusion)

Với bản chất phân tán toàn cầu của các đội SRE, giao tiếp hiệu quả luôn luôn là một ưu tiên cao trong SRE. Chương này đã thảo luận về các công cụ và kỹ thuật mà các đội SRE sử dụng để duy trì các mối quan hệ hiệu quả giữa đội của họ và với các đội đối tác khác nhau của họ.

Sự hợp tác giữa các đội SRE có các thách thức, nhưng cũng có các phần thưởng to lớn tiềm năng, bao gồm các cách tiếp cận chung cho các nền tảng để giải quyết các vấn đề, cho phép chúng tôi tập trung vào việc giải quyết các vấn đề khó khăn hơn.

<a id="fn1"></a>[1](#fn1) Và, như tất cả chúng ta đều biết, văn hóa đánh bại chiến lược mỗi lần: [[Mer11]](https://sre.google/sre-book/bibliography#Mer11).

<a id="fn2"></a>[2](#fn2) Đội lớn hơn thường có xu hướng vô tình nói đè lên đội nhỏ hơn, khó kiểm soát hơn các cuộc trò chuyện bên gây xao nhãng, v.v.

<a id="fn3"></a>[3](#fn3) Trong trường hợp cụ thể này, con đường đến địa ngục thực sự được lát bằng JavaScript.

<a id="fn4"></a>[4](#fn4) Tức là, phần mềm có cùng cấu trúc như cấu trúc truyền thông của tổ chức tạo ra phần mềm — xem [*https://en.wikipedia.org/wiki/Conway's_law*](https://en.wikipedia.org/wiki/Conway%27s_law).

<a id="fn5"></a>[5](#fn5) DoubleClick for Publishers là một công cụ để các nhà xuất bản quản lý các quảng cáo được phục vụ trên các website và trong các ứng dụng của họ.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
