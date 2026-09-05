# Chương 7. Sự Tiến hóa của Tự động hóa tại Google (The Evolution of Automation at Google)

> **Nguyên bản:** [Chapter 7 - The Evolution of Automation at Google](https://sre.google/sre-book/automation-at-google/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Niall Murphy cùng John Looney và Michael Kacirek
*Biên tập:* Betsy Beyer

> Ngoài thuật đen ra, chỉ có tự động hóa và cơ khí hóa mà thôi.
>
> Federico García Lorca (1898–1936), nhà thơ và nhà viết kịch Tây Ban Nha

Đối với SRE (Site Reliability Engineering — kỹ thuật độ tin cậy trang web), tự động hóa đóng vai trò như một bộ khuếch đại lực lượng (force multiplier), chứ không phải liều panacea (thuốc chữa vạn năng). Dĩ nhiên, việc chỉ *khuếch đại* lực lượng không tự nó làm thay đổi độ chính xác của việc lực lượng đó được áp dụng vào đâu: tự động hóa thiếu suy nghĩ có thể tạo ra nhiều vấn đề ngang với số nó giải quyết. Vì vậy, dù chúng tôi tin rằng tự động hóa bằng phần mềm vượt trội hơn vận hành thủ công trong phần lớn hoàn cảnh, thì phương án vượt trội hơn cả hai lựa chọn là một thiết kế hệ thống cấp cao hơn, không cần đến bất kỳ thứ nào trong số đó — một hệ thống *tự trị* (autonomous). Hay nói cách khác, giá trị của tự động hóa đến từ cả những gì nó làm lẫn cách nó được áp dụng một cách sáng suốt. Chúng tôi sẽ bàn về giá trị của tự động hóa, và về cách suy nghĩ của chúng tôi đã tiến hóa theo thời gian.

## Giá trị của Tự động hóa (The Value of Automation)

Giá trị chính xác của tự động hóa là gì?<sup>[1](#fn1)</sup>

### Sự Nhất quán (Consistency)

Mặc dù quy mô (scale) là động lực rõ ràng cho tự động hóa, vẫn còn nhiều lý do khác để áp dụng nó. Hãy xem xét các hệ thống tính toán trong trường đại học, nơi nhiều người làm kỹ thuật hệ thống bắt đầu sự nghiệp. Các quản trị viên hệ thống (system administrator) có nền tảng này thường được giao vận hành một tập hợp máy tính (machine) hoặc một số phần mềm, và quen với việc thực hiện thủ công đủ thứ hành động để hoàn thành nhiệm vụ. Một ví dụ phổ biến là tạo tài khoản người dùng; các ví dụ khác gồm các công việc vận hành thuần túy như đảm bảo sao lưu (backup) diễn ra, quản lý chuyển giao (failover) của server, và các thao tác dữ liệu nhỏ như thay đổi *resolv.conf* của các server tên miền (DNS server) phía trên (upstream), dữ liệu zone của DNS server, và tương tự. Rốt cuộc, sự phổ biến của các tác vụ thủ công này không thỏa đáng cho cả tổ chức lẫn chính những người duy trì hệ thống theo cách này. Trước hết, bất kỳ hành động nào mà một hay nhiều người thực hiện hàng trăm lần sẽ không bao giờ giống hệt nhau mỗi lần: dù thiện chí đến đâu, rất ít người trong chúng tôi có thể nhất quán như một máy móc. Sự thiếu nhất quán không thể tránh này dẫn đến sai lầm, sơ suất, vấn đề chất lượng dữ liệu và, vâng, cả độ tin cậy (reliability). Trong lĩnh vực này — thực thi các quy trình đã biết, có phạm vi rõ ràng — giá trị của sự nhất quán, theo nhiều cách, chính là giá trị cốt lõi của tự động hóa.

### Một Nền tảng (A Platform)

Tự động hóa không chỉ mang lại sự nhất quán. Khi được thiết kế và thực hiện đúng, các hệ thống tự động còn tạo thành một *nền tảng* (platform) có thể mở rộng, áp dụng cho nhiều hệ thống hơn, hoặc thậm chí tách ra để sinh lời.<sup>[2](#fn2)</sup> (Phương án thay thế, tức là không dùng tự động hóa, vừa kém hiệu quả về chi phí vừa không thể mở rộng: nó giống như một loại thuế đánh lên việc vận hành hệ thống.)

Một nền tảng còn *tập trung hóa sai lầm*. Nói cách khác, một bug (lỗi) được sửa trong code sẽ chỉ cần sửa một lần và mãi mãi, khác với một nhóm đủ lớn con người thực hiện cùng một quy trình, như đã bàn trước đó. Một nền tảng có thể mở rộng thêm các tác vụ dễ dàng hơn so với việc ra lệnh cho con người thực hiện (hoặc đôi khi họ thậm chí không nhận ra phải làm). Tùy bản chất tác vụ, nó có thể chạy liên tục hoặc thường xuyên hơn nhiều so với mức con người có thể hoàn thành một cách thích hợp, hoặc vào những thời điểm bất tiện cho con người. Hơn nữa, một nền tảng có thể xuất khẩu các metrics (chỉ số) về hiệu năng của mình, hoặc giúp bạn phát hiện những chi tiết về quy trình mà trước đó không biết, vì các chi tiết này dễ đo lường hơn trong ngữ cảnh của một nền tảng.

### Sửa chữa Nhanh hơn (Faster Repairs)

Các hệ thống dùng tự động hóa để xử lý những lỗi thường gặp (một tình huống phổ biến trong tự động hóa do SRE tạo ra) còn mang lại một lợi ích bổ sung. Nếu tự động hóa chạy thường xuyên và đủ hiệu quả, mean time to repair (MTTR — thời gian trung bình để sửa chữa) của những lỗi này sẽ giảm. Khi đó, bạn có thể dành thời gian cho các tác vụ khác, từ đó đạt được developer velocity (tốc độ phát triển) cao hơn, thay vì phải mất thời gian ngăn ngừa một vấn đề hay (phổ biến hơn) dọn dẹp sau nó. Như ngành đã hiểu rõ, vấn đề càng được phát hiện muộn trong vòng đời sản phẩm thì chi phí sửa chữa càng lớn; xem [Testing for Reliability](https://sre.google/sre-book/testing-reliability/). Nhìn chung, các vấn đề xảy ra trong production thật sự đắt nhất để sửa, cả về thời gian lẫn tiền bạc. Do đó, một hệ thống tự động phát hiện vấn đề ngay khi chúng nảy sinh có nhiều khả năng làm giảm tổng chi phí của hệ thống, với điều kiện hệ thống đủ lớn.

### Hành động Nhanh hơn (Faster Action)

Trong các tình huống hạ tầng mà tự động hóa SRE thường được triển khai, con người không thể phản ứng nhanh bằng máy móc. Với phần lớn các trường hợp phổ biến, nơi mà, ví dụ, failover hay chuyển đổi traffic (lưu lượng) có thể được định nghĩa rõ ràng cho một ứng dụng cụ thể, việc bắt buộc một người thỉnh thoảng phải nhấn nút "Cho phép hệ thống tiếp tục chạy" là không hợp lý. (Đúng là đôi khi quy trình tự động có thể khiến tình hình tồi tệ hơn, nhưng đó chính là lý do các quy trình như vậy nên bị giới hạn trong các miền được định nghĩa rõ ràng.) Google có rất nhiều tự động hóa; trong nhiều trường hợp, các dịch vụ chúng tôi hỗ trợ không thể tồn tại lâu mà không có tự động hóa này, vì chúng đã từ lâu vượt qua ngưỡng mà vận hành thủ công có thể quản lý.

### Tiết kiệm Thời gian (Time Saving)

Cuối cùng, tiết kiệm thời gian là lý do thường được nêu ra khi nói về tự động hóa. Dù mọi người hay viện dẫn lý do này hơn các lý do khác, lợi ích của nó lại khó lượng hóa trực tiếp hơn theo nhiều cách. Các kỹ sư thường do dự không biết có nên viết một phần tự động hóa hay đoạn code cụ thể hay không, khi cân nhắc giữa công sức tiết kiệm được nhờ không phải làm thủ công và công sức bỏ ra để viết nó.<sup>[3](#fn3)</sup> Ta dễ bỏ qua thực tế rằng một khi đã đóng gói một số tác vụ vào tự động hóa, bất kỳ ai cũng có thể thực hiện được. Vì vậy, khoản tiết kiệm thời gian này áp dụng cho bất kỳ ai có thể sử dụng hợp lý hệ thống tự động hóa. Việc tách người vận hành khỏi chính công việc vận hành là vô cùng mạnh mẽ.

Joseph Bironas, một SRE từng phụ trách công tác khởi động (turnup) các datacenter của Google trong một thời gian, đã lập luận mạnh mẽ như sau:

"Nếu chúng tôi đang kỹ thuật hóa các quy trình và giải pháp không thể tự động hóa được, chúng tôi tiếp tục phải bố trí con người để duy trì hệ thống. Nếu chúng tôi phải bố trí con người để thực hiện công việc, chúng tôi đang cho các máy móc ăn bằng máu, mồ hôi và nước mắt của con người. Hãy nghĩ đến *The Matrix* (Ma trận) với ít hiệu ứng đặc biệt hơn và nhiều Quản trị viên Hệ thống (System Administrators) bực bội hơn."

<a id="gia-tri-cho-google-sre"></a>

## Giá trị cho Google SRE (The Value for Google SRE)

Những lợi ích và đánh đổi này không chỉ đúng với chúng tôi mà còn với bất kỳ ai, và Google thực sự có xu hướng mạnh mẽ về tự động hóa. Một phần sở thích đó bắt nguồn từ các thách thức kinh doanh cụ thể: các sản phẩm và dịch vụ mà chúng tôi vận hành có quy mô toàn cầu, nên thường không có thời gian để làm những việc “cầm tay chỉ việc” cho máy móc hoặc dịch vụ như cách phổ biến ở các tổ chức khác.<sup>[4](#fn4)</sup> Với các dịch vụ thực sự lớn, tính nhất quán, tốc độ và độ tin cậy chi phối phần lớn các cuộc thảo luận về đánh đổi khi triển khai tự động hóa.

Một lập luận khác ủng hộ tự động hóa, đặc biệt với Google, là môi trường production của chúng tôi phức tạp nhưng đồng nhất đến ngạc nhiên, được mô tả trong [The Production Environment at Google, from the Viewpoint of an SRE](https://sre.google/sre-book/production-environment/). Trong khi các tổ chức khác có thể có một thiết bị quan trọng không có API (giao diện lập trình ứng dụng) dễ truy cập, phần mềm không có mã nguồn khả dụng, hoặc một trở ngại khác ngăn cản kiểm soát hoàn toàn các hoạt động production, thì Google nhìn chung tránh các kịch bản như vậy. Chúng tôi đã xây dựng API cho các hệ thống khi nhà cung cấp không có sẵn API. Dù mua phần mềm cho một tác vụ cụ thể có thể rẻ hơn nhiều trong ngắn hạn, chúng tôi vẫn chọn viết giải pháp riêng, vì làm vậy tạo ra các API có tiềm năng mang lại lợi ích dài hạn lớn hơn nhiều. Chúng tôi đã dành nhiều thời gian vượt qua các trở ngại trong việc quản lý hệ thống tự động, rồi quyết tâm tự phát triển chính hệ thống quản lý tự động đó. Với cách Google quản lý mã nguồn của nó [[Pot16]](https://sre.google/sre-book/bibliography#Pot16), việc code đó khả dụng cho gần như bất kỳ hệ thống nào mà SRE chạm đến cũng có nghĩa sứ mệnh "sở hữu sản phẩm trong production" của chúng tôi dễ dàng hơn nhiều, vì chúng tôi kiểm soát toàn bộ stack.

Dĩ nhiên, dù Google mang xu hướng ý thức hệ là dùng máy để quản lý máy khi có thể, thực tế đòi hỏi điều chỉnh cách tiếp cận của chúng tôi. Tự động hóa mọi thành phần của mọi hệ thống không phải lúc nào cũng phù hợp, và không phải ai cũng có khả năng hay khuynh hướng phát triển tự động hóa vào một thời điểm cụ thể. Một số hệ thống thiết yếu ban đầu chỉ là các prototype (nguyên mẫu) nhanh, không được thiết kế để tồn tại lâu hay để giao tiếp với tự động hóa. Các đoạn trước thể hiện một quan điểm tối đa (maximalist) về lập trường của chúng tôi, nhưng là quan điểm mà chúng tôi đã thành công khá rộng rãi trong việc đưa vào thực tế ở bối cảnh Google. Nhìn chung, chúng tôi chọn tạo các nền tảng khi có thể, hoặc *xếp đặt* để có thể tạo ra các nền tảng theo thời gian. Chúng tôi coi cách tiếp cận dựa trên nền tảng này là cần thiết cho khả năng quản lý và khả năng scale.

## Các Use case cho Tự động hóa (The Use Cases for Automation)

Trong ngành, *tự động hóa* thường chỉ việc viết code để giải quyết một loạt vấn đề, dù động lực viết code cũng như bản thân các giải pháp thường khá khác nhau. Theo cách nhìn rộng hơn này, tự động hóa là "meta-phần mềm" (meta-software) — phần mềm dùng để tác động lên phần mềm.

Như chúng tôi đã ngụ ý trước đó, có một số use case cho tự động hóa. Dưới đây là một danh sách không đầy đủ các ví dụ:

-   Tạo tài khoản người dùng
-   Khởi động (turnup) và tắt (turndown) cluster (cụm máy) cho các dịch vụ
-   Chuẩn bị cài đặt và decommission (ngừng vận hành) phần mềm hoặc phần cứng
-   Rollout (triển khai) các phiên bản phần mềm mới
-   Thay đổi cấu hình thời gian chạy (runtime configuration)
-   Một trường hợp đặc biệt của thay đổi cấu hình thời gian chạy: các thay đổi đối với các sự phụ thuộc (dependencies) của bạn

Danh sách này có thể tiếp tục về cơ bản *đến vô cùng* (ad infinitum).

## Các Use case cho Tự động hóa của Google SRE

Trong Google, chúng tôi có tất cả các use case vừa liệt kê, và nhiều hơn nữa.

Tuy nhiên, trong nội bộ Google SRE, trọng tâm chính của chúng tôi thường là vận hành hạ tầng, chứ không phải quản lý chất lượng dữ liệu đi qua hạ tầng đó. Ranh giới này không hoàn toàn rõ ràng — ví dụ, chúng tôi rất quan tâm nếu một nửa tập dữ liệu biến mất sau một lần đẩy code, và do đó thiết lập cảnh báo cho các khác biệt thô như vậy, nhưng hiếm khi chúng tôi viết tương đương của việc thay đổi thuộc tính của một tập con tùy ý các tài khoản trên một hệ thống. Vì vậy, ngữ cảnh cho tự động hóa của chúng tôi thường là tự động hóa để quản lý vòng đời của các hệ thống, chứ không phải dữ liệu của chúng: ví dụ, triển khai một dịch vụ trong một cluster mới.

Đến mức đó, các nỗ lực tự động hóa của SRE không khác xa những gì nhiều người và tổ chức khác làm, chỉ khác ở chỗ chúng tôi dùng các công cụ khác nhau để quản lý và có một trọng tâm khác (như sẽ bàn sau).

Các công cụ phổ biến như Puppet, Chef, cfengine, và thậm chí Perl, tất cả đều cung cấp cách tự động hóa các tác vụ cụ thể, khác nhau chủ yếu ở mức độ trừu tượng (abstraction) của các thành phần hỗ trợ việc tự động hóa. Một ngôn ngữ đầy đủ như Perl cung cấp các affordance (khả năng) cấp POSIX, về lý thuyết cho một phạm vi tự động hóa gần như không giới hạn xuyên suốt các API khả dụng của hệ thống,<sup>[5](#fn5)</sup> trong khi Chef và Puppet cung cấp các trừu tượng out-of-the-box (sẵn sàng dùng ngay) để thao tác các dịch vụ hoặc các thực thể cấp cao hơn. Đánh đổi ở đây là kinh điển: trừu tượng cấp cao hơn dễ quản lý và lập luận hơn, nhưng khi bạn gặp một "trừu tượng rò rỉ" (leaky abstraction), bạn sẽ thất bại theo hệ thống, lặp đi lặp lại và có thể không nhất quán. Ví dụ, chúng tôi thường giả định rằng việc đẩy một binary (file thực thi) mới đến một cluster là nguyên tử (atomic); cluster sẽ hoặc kết thúc ở phiên bản cũ, hoặc phiên bản mới. Tuy nhiên, hành vi thực tế phức tạp hơn: mạng của cluster đó có thể hỏng giữa chừng; các machine có thể hỏng; truyền thông đến tầng quản lý cluster có thể hỏng, để lại hệ thống trong một trạng thái không nhất quán; tùy tình huống, binary mới có thể đã staged (chuẩn bị) nhưng chưa đẩy, hoặc đã đẩy nhưng chưa khởi động lại, hoặc đã khởi động lại nhưng không thể xác minh. Rất ít trừu tượng mô hình hóa thành công các kết quả như vậy, và phần lớn thường tự dừng lại và gọi can thiệp. Các hệ thống tự động hóa thực sự tồi tệ thậm chí còn không làm được cả điều đó.

SRE có một số triết lý và sản phẩm trong lĩnh vực tự động hóa. Một số trông giống các công cụ rollout (triển khai) tổng quát, không mô hình hóa chi tiết các thực thể cấp cao; số khác lại giống các ngôn ngữ dùng để mô tả triển khai dịch vụ (và tương tự) ở mức trừu tượng rất cao. Công việc thuộc nhóm sau thường tái sử dụng tốt hơn và đóng vai trò nền tảng chung hơn so với nhóm trước. Tuy nhiên, độ phức tạp của môi trường production đôi khi khiến cách tiếp cận của nhóm trước lại là lựa chọn khả thi nhất trong thời điểm đó.

## Một Hệ phân cấp của Các Lớp Tự động hóa (A Hierarchy of Automation Classes)

Mặc dù các bước tự động hóa này đều có giá trị, và một nền tảng tự động hóa tự nó đã là một tài sản quý giá, nhưng trong một thế giới lý tưởng, chúng tôi sẽ không cần đến bất kỳ tự động hóa bên ngoài (external automation) nào. Thực tế, thay vì một hệ thống *bắt buộc* phải có logic keo (glue logic) bên ngoài, tốt hơn là một hệ thống *không cần bất kỳ logic keo nào*. Lý do không chỉ nằm ở việc nội tại hóa mang lại hiệu quả cao hơn (dù hiệu quả đó vẫn hữu ích), mà còn vì hệ thống được thiết kế để không cần logic keo ngay từ đầu. Để đạt được điều này, cần xác định các use case của logic keo — nhìn chung là các thao tác "bậc nhất" của hệ thống, chẳng hạn như thêm tài khoản hoặc khởi động hệ thống — và tìm cách xử lý trực tiếp các use case đó trong ứng dụng.

Lấy một ví dụ cụ thể hơn, phần lớn các quy trình tự động hóa khởi động tại Google gặp trục trặc vì chúng được duy trì tách biệt khỏi hệ thống cốt lõi, dẫn đến tình trạng "bit rot" (mục nát bit), tức là không được cập nhật khi các hệ thống nền tảng thay đổi. Dù có thiện chí đến đâu, nỗ lực gắn chặt hai thành phần này (tự động hóa khởi động và hệ thống cốt lõi) thường thất bại do ưu tiên không đồng nhất, bởi các kỹ sư sản phẩm sẽ phản đối — một cách không thiếu lý do — trước yêu cầu phải chạy kiểm thử cho mọi thay đổi. Thứ hai, những quy trình tự động hóa thiết yếu nhưng chỉ chạy theo chu kỳ thưa thớt, khiến việc kiểm thử trở nên khó khăn, thường đặc biệt dễ hỏng hóc do vòng phản hồi bị kéo dài. Failover cluster (chuyển giao cluster) là ví dụ điển hình cho loại tự động hóa chạy không thường xuyên này: các sự kiện failover có thể chỉ xảy ra vài tháng một lần, hoặc không đủ thường xuyên để lộ ra các bất nhất quán giữa các instance (thực thể). Sự tiến hóa của tự động hóa theo một con đường:

1) Không có tự động hóa

Master (bản chính) database được failover thủ công giữa các vị trí.

2) Tự động hóa cụ thể hệ thống, duy trì bên ngoài

Một SRE có một script failover trong thư mục home của mình.

3) Tự động hóa tổng quát, duy trì bên ngoài

SRE thêm hỗ trợ database vào một script "failover tổng quát" mà mọi người dùng.

4) Tự động hóa cụ thể hệ thống, duy trì nội bộ

Database đi kèm script failover của riêng nó.

5) Các hệ thống không cần bất kỳ tự động hóa nào

Database tự nhận ra các vấn đề và tự động failover mà không cần can thiệp của con người.

SRE ghét các hoạt động thủ công, nên chúng tôi cố gắng tạo ra các hệ thống không cần chúng. Tuy nhiên, đôi khi các hoạt động thủ công là không thể tránh được.

Ngoài ra, còn có một biến thể của tự động hóa áp dụng các thay đổi không chỉ trên miền cấu hình của một hệ thống cụ thể, mà trên toàn bộ miền production. Trong môi trường production độc quyền, tập trung hóa cao như của Google, có rất nhiều thay đổi có phạm vi không gắn với một dịch vụ cụ thể — ví dụ thay đổi các Chubby server upstream, một thay đổi flag (cờ) cho thư viện client Bigtable để truy cập đáng tin cậy hơn, v.v. — nhưng vẫn cần được quản lý an toàn và rollback (hoàn tác) khi cần. Khi số lượng thay đổi vượt quá một ngưỡng nhất định, việc thực hiện thủ công các thay đổi toàn production trở nên bất khả thi; và trước điểm đó, việc giám sát thủ công một quy trình mà phần lớn các thay đổi là tầm thường hoặc được hoàn thành thành công bằng chiến lược khởi động lại và kiểm tra cơ bản là một sự lãng phí.

Hãy dùng các nghiên cứu tình huống nội bộ để minh họa chi tiết một số điểm trước đó. Nghiên cứu tình huống đầu tiên nói về cách, nhờ một số công việc tận tâm, nhìn xa, chúng tôi đã đạt được cõi cực lạc (nirvana) tự nhận của SRE: tự động hóa chính mình ra khỏi một công việc.

## Tự động hóa Chính mình Ra khỏi một Công việc: Tự động hóa TẤT CẢ Mọi thứ! (Automate Yourself Out of a Job: Automate ALL the Things!)

Suốt một thời gian dài, các sản phẩm Quảng cáo (Ads) của Google lưu trữ dữ liệu trên một database MySQL. Vì dữ liệu Ads đòi hỏi độ tin cậy cao, một nhóm SRE được giao phụ trách hạ tầng này. Từ 2005 đến 2008, Ads Database phần lớn vận hành ở trạng thái mà chúng tôi xem là trưởng thành và được quản lý tốt. Chẳng hạn, chúng tôi đã tự động hóa những phần tệ nhất — dù không phải tất cả — của công việc thường nhật nhằm thay thế các replica (bản sao) chuẩn. Chúng tôi tin rằng Ads Database được quản lý hiệu quả và rằng mình đã hái được phần lớn những quả thấp (low-hanging fruit — thứ dễ làm nhất) trong tối ưu hóa và mở rộng quy mô. Tuy nhiên, khi các hoạt động hàng ngày trở nên trơn tru, các thành viên trong nhóm bắt đầu hướng tới cấp phát triển hệ thống tiếp theo: di cư (migrate) MySQL lên hệ thống lập lịch cluster của Google, Borg.

Chúng tôi hy vọng cuộc di cư này sẽ cung cấp hai lợi ích chính:

-   *Hoàn toàn* loại bỏ bảo trì machine/replica: Borg sẽ tự động xử lý việc thiết lập/khởi động lại các task (nhiệm vụ) mới và hỏng.
-   Cho phép bin-packing nhiều instance MySQL trên cùng một máy vật lý: Borg giúp tận dụng hiệu quả hơn tài nguyên máy thông qua container.

Cuối năm 2008, chúng tôi triển khai thành công một instance MySQL proof of concept trên Borg. Tuy nhiên, điều này kéo theo một khó khăn mới đáng kể. Một đặc tính vận hành cốt lõi của Borg là các task của nó tự động di chuyển chỗ. Các task thường di chuyển trong Borg với tần suất một đến hai lần mỗi tuần. Tần suất này có thể chấp nhận được cho các replica database của chúng tôi, nhưng là không thể chấp nhận được cho các master.

Vào thời điểm đó, quy trình failover master mất từ 30 đến 90 phút cho mỗi instance. Nguyên nhân đơn giản là chúng tôi chạy trên các máy chia sẻ và phải chịu các lần khởi động lại để nâng cấp kernel, nên ngoài tỷ lệ hỏng máy thông thường, chúng tôi còn phải dự tính một số lần failover không liên quan khác mỗi tuần. Yếu tố này, kết hợp với số shard mà hệ thống của chúng tôi được lưu trữ, dẫn đến:

-   Failover thủ công sẽ ngốn rất nhiều giờ công con người và tối đa chỉ đạt được 99% uptime (thời gian hoạt động) — không đáp ứng được yêu cầu business thực sự của sản phẩm.
-   Để đáp ứng error budget (ngân sách lỗi), mỗi lần failover chỉ được phép có downtime (thời gian ngừng) dưới 30 giây. Không thể tối ưu một quy trình phụ thuộc con người để đưa downtime xuống mức này.

Vì vậy, chúng tôi chỉ còn cách tự động hóa failover. Thực tế, phạm vi cần tự động hóa còn rộng hơn cả failover. Năm 2009, nhóm SRE phụ trách Ads đã hoàn thành daemon (quá trình nền) failover tự động mang tên "Decider". Decider có thể xử lý failover MySQL, dù có kế hoạch hay không, trong dưới 30 giây ở 95% các trường hợp. Nhờ Decider, MySQL on Borg (MoB) cuối cùng đã thành hiện thực. Chúng tôi chuyển từ việc tối ưu hóa hạ tầng cho trạng thái không failover sang việc chấp nhận rằng sự cố là điều không thể tránh khỏi, từ đó tối ưu hóa để phục hồi nhanh thông qua tự động hóa.

Dù tự động hóa giúp chúng tôi đạt được khả năng sẵn sàng cao cho MySQL trong môi trường buộc phải khởi động lại hai lần mỗi tuần, nó cũng đi kèm một tập chi phí riêng. Tất cả ứng dụng của chúng tôi phải được sửa đổi để chứa nhiều logic xử lý lỗi hơn đáng kể so với trước. Vì chuẩn mực trong thế giới phát triển MySQL là giả định instance MySQL là thành phần ổn định nhất trong stack, sự chuyển đổi này có nghĩa là phải tùy chỉnh phần mềm như JDBC (Java Database Connectivity — Kết nối Cơ sở Dữ liệu Java) cho chịu được với môi trường dễ hỏng của chúng tôi. Tuy nhiên, lợi ích của việc di cư lên MoB với Decider đáng giá hơn những chi phí này. Một khi trên MoB, thời gian đội dành cho các tác vụ vận hành tầm thường giảm 95%. Các failover của chúng tôi được tự động hóa, nên một sự cố ngừng dịch vụ của một task database đơn lẻ không còn khiến ai đó bị gọi trực (page) nữa.

Hệ quả chính của tự động hóa mới này là chúng tôi có nhiều thời gian rảnh hơn để cải thiện các phần khác của hạ tầng. Những cải thiện đó có hiệu ứng lan truyền: càng tiết kiệm được nhiều thời gian, chúng tôi càng có thể dành cho việc tối ưu hóa và tự động hóa các công việc tẻ nhạt khác. Cuối cùng, chúng tôi tự động hóa được cả các thay đổi schema (mô hình dữ liệu), khiến chi phí bảo trì vận hành tổng thể của Ads Database giảm gần 95%. Có người sẽ nói rằng chúng tôi đã thành công tự động hóa chính mình ra khỏi công việc này. Phía phần cứng của miền chúng tôi cũng cải thiện. Di cư lên MoB giải phóng đáng kể tài nguyên vì chúng tôi có thể lên lịch nhiều instance MySQL trên các machine giống nhau, cải thiện mức sử dụng phần cứng. Nhìn chung, chúng tôi giải phóng được khoảng 60% phần cứng. Đội của chúng tôi giờ dồi dào phần cứng và tài nguyên kỹ thuật.

Ví dụ này cho thấy lợi ích của việc đầu tư thêm công sức để xây dựng nền tảng, thay vì chỉ thay thế các quy trình thủ công hiện có. Ví dụ tiếp theo đến từ nhóm hạ tầng cluster, nêu bật một số đánh đổi khó khăn hơn mà bạn có thể gặp trên con đường tự động hóa *tất cả* mọi thứ.

## Làm dịu Nỗi đau: Áp dụng Tự động hóa vào Việc Khởi động Cluster (Soothing the Pain: Applying Automation to Cluster Turnups)

Mười năm trước, đội SRE hạ tầng Cluster dường như nhận một nhân viên mới mỗi vài tháng. Hóa ra đó cũng đúng khoảng tần suất chúng tôi khởi động một cluster mới. Vì việc khởi động một dịch vụ trong một cluster mới giúp nhân viên mới tiếp xúc với phần bên trong của dịch vụ, tác vụ này có vẻ là một công cụ đào tạo tự nhiên và hữu ích.

Các bước được thực hiện để chuẩn bị một cluster sẵn sàng sử dụng giống như sau:

1.  Trang bị một tòa nhà datacenter cho điện và làm mát.
2.  Cài đặt và cấu hình các switch (bộ chuyển mạch) cốt lõi và kết nối đến backbone (mạng trục).
3.  Cài đặt một vài rack (giá máy) server ban đầu.
4.  Cấu hình các dịch vụ cơ bản như DNS và các trình cài đặt, sau đó cấu hình một dịch vụ khóa (lock service), lưu trữ, và tính toán.
5.  Triển khai các rack machine còn lại.
6.  Phân bổ tài nguyên cho các dịch vụ hướng người dùng, để các đội của họ có thể thiết lập các dịch vụ.

Bước 4 và 6 cực kỳ phức tạp. Trong khi các dịch vụ cơ bản như DNS tương đối đơn giản, các hệ thống con lưu trữ và tính toán vào thời điểm đó vẫn đang phát triển mạnh, nên các flag, thành phần và tối ưu hóa mới được thêm hàng tuần.

Một số dịch vụ bao gồm hơn một trăm hệ thống con, mỗi hệ thống lại có mạng lưới phụ thuộc phức tạp. Chỉ cần bỏ sót cấu hình một hệ thống con, hoặc cấu hình một hệ thống hay thành phần khác lệch so với các triển khai khác, là một sự cố ảnh hưởng đến khách hàng sẽ lập tức xảy ra.

Trong một trường hợp, một cluster Bigtable nhiều petabyte được cấu hình để không dùng disk (ổ đĩa) đầu tiên (đĩa log) trên các hệ thống 12-disk, vì lý do độ trễ. Một năm sau, một số tự động hóa giả định rằng nếu disk đầu tiên của một machine không được dùng thì machine đó không có lưu trữ nào được cấu hình; do đó, an toàn để wipe (xóa) machine và thiết lập lại từ đầu. Toàn bộ dữ liệu Bigtable bị xóa chỉ trong tích tắc. May mắn thay chúng tôi có nhiều replica thời gian thực của tập dữ liệu, nhưng những sự bất ngờ như vậy là không được mong đợi. Tự động hóa cần cẩn thận khi dựa vào các tín hiệu "an toàn" ngầm.

Ban đầu, tự động hóa tập trung vào việc tăng tốc giao hàng cluster. Cách tiếp cận này thường dùng SSH (Secure Shell — vỏ an toàn) một cách sáng tạo cho các tác vụ phân phối gói và khởi tạo dịch vụ tẻ nhạt. Chiến lược đó là một thắng lợi ban đầu, nhưng những script dạng tự do này dần trở thành thứ cholesterol của nợ kỹ thuật (technical debt).

## Phát hiện Các Bất nhất quán với Prodtest (Detecting Inconsistencies with Prodtest)

Khi số lượng cluster tăng, một số cluster đòi hỏi các flag và cài đặt được tinh chỉnh thủ công. Kết quả là các nhóm ngày càng tốn nhiều thời gian truy tìm những cấu hình sai khó thấy. Chẳng hạn, nếu một flag khiến GFS (Google File System — Hệ thống Tệp Google) phản hồi tốt hơn khi xử lý log bị rò rỉ vào các mẫu (templates) mặc định, những cell (ô) chứa nhiều tệp có thể hết bộ nhớ (out of memory) dưới tải. Các cấu hình sai gây bực bội và tốn thời gian, len lỏi vào gần như mọi thay đổi cấu hình lớn.

Những script shell (vỏ) sáng tạo — dù dễ vỡ — mà chúng tôi dùng để cấu hình cluster không scale được, cả về số người muốn thực hiện thay đổi lẫn về số tổ hợp (permutations) cluster thuần túy cần xây dựng. Các script này cũng không giải quyết được những mối quan tâm quan trọng hơn trước khi tuyên bố một dịch vụ đã ổn để nhận traffic hướng khách hàng, như:

-   Tất cả các sự phụ thuộc của dịch vụ có khả dụng và được cấu hình đúng không?
-   Tất cả các cấu hình và gói có nhất quán với các triển khai khác không?
-   Đội có thể xác nhận rằng mọi ngoại lệ cấu hình đều được mong muốn không?

Prodtest (Production Test — Kiểm thử Production) là giải pháp tinh tế cho những bất ngờ không mong muốn này. Chúng tôi mở rộng khung kiểm thử unit (đơn vị) Python để kiểm thử unit các dịch vụ thực tế. Các kiểm thử unit này phụ thuộc lẫn nhau, tạo thành một chuỗi; một kiểm thử thất bại sẽ nhanh chóng hủy bỏ phần còn lại. Hãy lấy kiểm thử trong [Hình 7-1](#hinh-7-1) làm ví dụ.


<a id="hinh-7-1"></a>![Hình 7-1](../assets/imgs/fig-7-1.jpg)

[Hình 7-1.](#hinh-7-1) ProdTest cho Dịch vụ DNS, cho thấy làm thế nào một kiểm thử thất bại hủy bỏ chuỗi các kiểm thử tiếp theo.

Prodtest của một đội cụ thể được cho tên cluster, và có thể xác thực các dịch vụ của đội đó trong cluster đó. Các bổ sung sau đó cho phép chúng tôi tạo một đồ thị (graph) của các kiểm thử unit và trạng thái của chúng. Tính năng này cho phép một kỹ sư nhanh chóng xem dịch vụ của mình có được cấu hình đúng trong tất cả các cluster không, và nếu không thì tại sao. Đồ thị làm nổi bật bước thất bại, và kiểm thử unit Python thất bại xuất ra một thông báo lỗi chi tiết hơn.

Khi một đội bị trì hoãn do cấu hình sai bất ngờ từ đội khác, họ có thể nộp bug để mở rộng Prodtest. Việc này giúp phát hiện sớm các vấn đề tương tự trong tương lai. Các SRE tự hào vì có thể cam kết với khách hàng rằng mọi dịch vụ — dù là dịch vụ mới vừa khởi động hay dịch vụ hiện có với cấu hình mới — đều phục vụ traffic production một cách đáng tin cậy.

Lần đầu tiên, các quản lý dự án của chúng tôi có thể dự đoán thời điểm một cluster có thể "đi vào hoạt động" (go live), và hiểu rõ *tại sao* mỗi cluster mất sáu tuần hoặc hơn để chuyển từ trạng thái "sẵn sàng về mặt mạng" (network-ready) sang "đang phục vụ traffic trực tiếp". Rồi bất ngờ, SRE nhận được một sứ mệnh từ ban quản lý cấp cao: *Trong ba tháng, năm cluster mới sẽ đạt "sẵn sàng về mặt mạng" vào cùng một ngày. Hãy khởi động chúng trong một tuần.*

## Giải quyết Các Bất nhất quán một cách Idempotent (Không phụ thuộc số lần chạy) (Resolving Inconsistencies Idempotently)

Một “One Week Turnup” (Khởi động trong một tuần) là một sứ mệnh đáng sợ. Chúng tôi có hàng chục nghìn dòng script shell do hàng chục nhóm sở hữu. Chúng tôi có thể nhanh chóng biết một cluster cụ thể đã chuẩn bị kém thế nào, nhưng sửa chữa nó đồng nghĩa hàng chục nhóm phải nộp hàng trăm lỗi, và sau đó chúng tôi chỉ có thể hy vọng những lỗi này sẽ được sửa kịp thời.

Chúng tôi nhận ra rằng việc chuyển từ viết unit test bằng Python để phát hiện cấu hình sai sang viết code bằng Python để tự động sửa cấu hình sai sẽ giúp xử lý các vấn đề này nhanh hơn.

Vì kiểm thử unit đã xác định được cluster đang xem xét và các kiểm thử cụ thể đang thất bại, chúng tôi ghép mỗi kiểm thử với một sửa chữa tương ứng. Nếu mỗi sửa chữa được viết theo nguyên tắc idempotent và có thể giả định rằng tất cả các sự phụ thuộc đều được đáp ứng, việc giải quyết vấn đề sẽ trở nên dễ dàng — và an toàn. Yêu cầu sửa chữa phải idempotent cho phép các đội chạy "script sửa" của họ mỗi 15 phút mà không lo gây hại cho cấu hình của cluster. Chẳng hạn, nếu kiểm thử của đội DNS bị chặn do cấu hình của đội Machine Database trên một cluster mới, thì ngay khi cluster đó xuất hiện trong database, các kiểm thử và sửa chữa của đội DNS sẽ bắt đầu hoạt động.

Hãy lấy kiểm thử trong [Hình 7-2](#hinh-7-2) làm ví dụ. Nếu `TestDnsMonitoringConfigExists` thất bại, như được hiển thị, chúng tôi có thể gọi `FixDnsMonitoringCreateConfig`. Hàm này sẽ vét cấu hình từ một database, sau đó đẩy một tệp cấu hình khung (skeleton) lên hệ thống kiểm soát phiên bản (revision control system) của chúng tôi. Khi retry, `TestDnsMonitoringConfigExists` sẽ vượt qua, và kiểm thử `TestDnsMonitoringConfigPushed` có thể được thực hiện. Nếu kiểm thử này thất bại, bước `FixDnsMonitoringPushConfig` sẽ chạy. Nếu một sửa chữa thất bại nhiều lần, tự động hóa sẽ giả định rằng sửa chữa đã thất bại và dừng lại, đồng thời thông báo cho người dùng.

Nhờ có các script này, một nhóm nhỏ kỹ sư có thể đưa hệ thống từ trạng thái “Mạng hoạt động, và các machine được liệt kê trong database” sang “Đang phục vụ 1% traffic websearch và ads” chỉ trong một hoặc hai tuần. Khi đó, điều này được xem là đỉnh cao của công nghệ tự động hóa.

Nhìn lại, cách tiếp cận này có những khiếm khuyết sâu sắc. Độ trễ giữa các bước kiểm thử, sửa chữa, rồi kiểm thử lại đã sinh ra các kiểm thử *flaky* (bất ổn — đôi khi chạy đôi khi không), lúc hoạt động, lúc thất bại. Không phải tất cả các sửa chữa tự nhiên đều idempotent, nên một kiểm thử flaky theo sau một sửa chữa có thể khiến hệ thống rơi vào trạng thái không nhất quán.


<a id="hinh-7-2"></a>![Hình 7-2](../assets/imgs/fig-7-2.jpg)

[Hình 7-2.](#hinh-7-2) ProdTest cho Dịch vụ DNS, cho thấy rằng một kiểm thử thất bại dẫn đến việc chỉ chạy đúng một sửa chữa.

## Khuynh hướng Chuyên môn hóa (The Inclination to Specialize)

Các quy trình tự động hóa có thể khác nhau theo ba khía cạnh:

-   *Năng lực* (Competence), tức là, độ chính xác của chúng
-   *Độ trễ* (Latency), tốc độ mà tất cả các bước được thực hiện khi được khởi động
-   *Mức độ liên quan* (Relevance), hay tỷ lệ quy trình thế giới thực được bao phủ bởi tự động hóa

Chúng tôi bắt đầu với một quy trình rất có năng lực (do các chủ sở hữu dịch vụ duy trì và vận hành), độ trễ cao (các chủ sở hữu dịch vụ thực hiện quy trình vào thời gian rảnh hoặc giao cho các kỹ sư mới), và rất liên quan (các chủ sở hữu dịch vụ biết khi nào thế giới thực thay đổi, và có thể sửa tự động hóa).

Để giảm độ trễ khởi động, nhiều đội sở hữu dịch vụ chỉ định một "đội khởi động" (turnup team) duy nhất để chạy tự động hóa cho họ. Đội khởi động dùng các ticket (yêu cầu) để bắt đầu mỗi giai đoạn trong khởi động, giúp chúng tôi theo dõi các tác vụ còn lại và tác vụ đó được giao cho ai. Nếu các tương tác con người liên quan đến các module tự động hóa diễn ra giữa những người trong cùng một phòng, việc khởi động cluster có thể diễn ra trong một khoảng thời gian ngắn hơn nhiều. Cuối cùng, chúng tôi có một quy trình tự động hóa có năng lực, chính xác và kịp thời!

Nhưng trạng thái này không kéo dài lâu. Thế giới thực vốn hỗn loạn: phần mềm, cấu hình, dữ liệu, v.v. liên tục thay đổi, tạo ra hơn một nghìn thay đổi riêng lẻ mỗi ngày lên các hệ thống chịu tác động. Những người bị ảnh hưởng nhiều nhất bởi các bug tự động hóa không còn là các chuyên gia miền nữa, khiến tự động hóa trở nên kém liên quan hơn (nghĩa là các bước mới bị bỏ sót) và kém năng lực hơn (các flag mới có thể đã khiến tự động hóa thất bại). Tuy nhiên, phải mất một lúc để sự sụt giảm chất lượng này ảnh hưởng đến tốc độ.

Code tự động hóa, như code kiểm thử unit, sẽ chết khi đội duy trì không ám ảnh việc giữ code đồng bộ với codebase (kho mã nguồn) mà nó bao phủ. Thế giới thay đổi xung quanh code: đội DNS thêm các tùy chọn cấu hình mới, đội lưu trữ đổi tên gói của họ, và đội mạng cần hỗ trợ các thiết bị mới.

Bằng cách giảm trách nhiệm duy trì và chạy code tự động hóa cho các đội vận hành dịch vụ, chúng tôi tạo ra các động lực tổ chức xấu xí:

- Một đội có nhiệm vụ chính là tăng tốc khởi động hiện tại không có động lực giảm nợ kỹ thuật của đội sở hữu dịch vụ, đội sẽ vận hành dịch vụ trong production sau này.
-   Một đội không vận hành tự động hóa không có động lực xây dựng các hệ thống dễ tự động hóa.
-   Một quản lý sản phẩm có lịch trình không bị ảnh hưởng bởi tự động hóa chất lượng thấp sẽ luôn ưu tiên tính năng mới hơn sự đơn giản và tự động hóa.

Công cụ hoạt động tốt nhất thường do chính người dùng viết. Lập luận tương tự cũng giải thích vì sao các đội phát triển sản phẩm cần giữ lại ít nhất một phần nhận thức về vận hành các hệ thống của họ trong môi trường production.

Việc khởi động cluster một lần nữa có độ trễ cao, không chính xác và không có năng lực — điều tồi tệ nhất trong tất cả các thế giới. Tuy nhiên, một chỉ thị bảo mật không liên quan đã cho phép chúng tôi thoát khỏi bẫy này. Phần lớn tự động hóa phân tán khi đó dựa vào SSH. Điều này vụng về về mặt bảo mật, vì con người phải có root (quyền quản trị tối cao) trên nhiều machine để chạy phần lớn các lệnh. Nhận thức ngày càng tăng về các mối đe dọa bảo mật tiên tiến, dai dẳng (persistent) thúc đẩy chúng tôi giảm các đặc quyền mà các SRE được hưởng xuống mức tối thiểu tuyệt đối cần thiết để làm công việc. Chúng tôi phải thay thế việc dùng sshd (daemon shell an toàn) bằng một Local Admin Daemon (Quá trình Quản trị viên Địa phương) được xác thực, điều khiển bởi ACL (Access Control List — danh sách kiểm soát truy cập), dựa trên RPC (Remote Procedure Call — lời gọi thủ tục từ xa), còn được gọi là Admin Servers (Các Server Quản trị viên), có đặc quyền để thực hiện các thay đổi địa phương đó. Kết quả là, không ai có thể cài đặt hoặc sửa đổi một server mà không để lại vết kiểm toán (audit trail). Các thay đổi đối với Local Admin Daemon và Package Repo (Kho Gói) đều cần phê duyệt qua các cuộc xem xét code (code reviews), khiến ai đó rất khó vượt quyền hạn của mình; chẳng hạn, cấp quyền cài đặt gói cho một người sẽ không cho phép họ xem các log (nhật ký) colocation. Admin Server ghi log người yêu cầu RPC, bất kỳ tham số nào, và kết quả của tất cả các RPC nhằm tăng cường debug (xử lý lỗi) và kiểm toán bảo mật.

## Khởi động Cluster Hướng đến Dịch vụ (Service-Oriented Cluster-Turnup)

Trong bản lặp lại tiếp theo, Admin Servers trở thành một phần của luồng công việc của các nhóm dịch vụ, bao gồm cả Admin Servers cụ thể cho từng máy (để cài đặt gói và khởi động lại) lẫn Admin Servers cấp cluster (cho các hành động như rút traffic hoặc khởi động một dịch vụ). Các SRE chuyển từ việc viết các script shell trong thư mục home của mình sang xây dựng các server RPC được đồng nghiệp xem xét, với các ACL chi tiết.

Sau này, khi nhận ra rằng các quy trình khởi động phải do chính các nhóm sở hữu dịch vụ thấu hiểu thật sự đảm nhiệm, chúng tôi coi đây là một cách để tiếp cận việc khởi động cluster như một vấn đề Kiến trúc Hướng đến Dịch vụ (Service-Oriented Architecture, SOA): các chủ sở hữu dịch vụ sẽ chịu trách nhiệm tạo một Admin Server để xử lý các RPC khởi động/tắt cluster, do hệ thống biết khi nào các cluster sẵn sàng gửi đến. Đổi lại, mỗi nhóm sẽ cung cấp hợp đồng (API) mà tự động hóa khởi động cần, trong khi vẫn được tự do thay đổi triển khai nền tảng. Khi một cluster đạt "sẵn sàng về mặt mạng", tự động hóa gửi một RPC đến mỗi Admin Server có vai trò trong việc khởi động cluster.

Giờ đây, chúng tôi đã có một quy trình phản hồi nhanh, hiệu quả và chính xác; quan trọng nhất, quy trình này vẫn vận hành ổn định dù tỷ lệ thay đổi, số lượng nhóm và số lượng dịch vụ dường như tăng gấp đôi mỗi năm.

Như đã đề cập trước đó, sự tiến hóa của tự động hóa khởi động của chúng tôi đi theo một con đường:

1.  Hành động thủ công do người vận hành kích hoạt (không có tự động hóa)
2.  Tự động hóa cụ thể hệ thống do người vận hành viết
3.  Tự động hóa tổng quát được duy trì bên ngoài
4.  Tự động hóa cụ thể hệ thống được duy trì nội bộ
5.  Các hệ thống tự trị không cần can thiệp của con người

Mặc dù quá trình tiến hóa này nhìn chung đã thành công, trường hợp của Borg lại cho thấy một góc nhìn khác mà chúng tôi dần hình thành về bài toán tự động hóa.

## Borg: Sự Ra đời của Máy tính Quy mô Kho (Borg: Birth of the Warehouse-Scale Computer)

Một cách khác để hiểu sự phát triển của thái độ của chúng tôi đối với tự động hóa, và khi nào, ở đâu tự động hóa đó được triển khai tốt nhất, là xem xét lịch sử phát triển của các hệ thống quản lý cluster của chúng tôi.<sup>[6](#fn6)</sup> Giống như MySQL on Borg đã minh họa thành công của việc chuyển đổi các hoạt động thủ công sang tự động, và quy trình khởi động cluster đã minh họa mặt tiêu cực của việc không suy nghĩ đủ cẩn thận về nơi và cách triển khai tự động hóa, việc phát triển quản lý cluster cũng rốt cuộc minh họa một bài học khác về cách tự động hóa nên được thực hiện. Giống như hai ví dụ trước, một thứ gì đó khá tinh vi đã được tạo ra như kết quả cuối cùng của một quá trình tiến hóa liên tục từ những khởi đầu đơn giản hơn.

Ban đầu, các cluster của Google được triển khai theo kiểu mạng nội bộ nhỏ: mỗi rack máy phục vụ một mục đích riêng và có cấu hình không đồng nhất. Các kỹ sư thường đăng nhập vào một máy chủ chính (master) quen thuộc để thực hiện các tác vụ quản trị; các binary “vàng” (golden) và cấu hình được đặt trên chính các máy chủ này. Vì chúng tôi chỉ dùng một nhà cung cấp colo (colocation — đặt cùng chỗ), phần lớn logic đặt tên ngầm định vị trí đó. Khi production mở rộng và chúng tôi bắt đầu sử dụng nhiều cluster, các miền tên khác nhau (tên cluster) xuất hiện trong hệ thống. Lúc này, cần một tệp mô tả vai trò của từng máy, đồng thời nhóm các máy lại dưới một chiến lược đặt tên tương đối linh hoạt. Tệp mô tả này, kết hợp với một công cụ tương đương SSH song song, cho phép chúng tôi khởi động lại (ví dụ) tất cả các máy tìm kiếm cùng lúc. Vào thời điểm đó, việc nhận các ticket như “tìm kiếm đã xong với máy `x1`, crawl (thu thập) có thể lấy máy ngay bây giờ” là chuyện thường tình.

Việc phát triển tự động hóa bắt đầu. Ban đầu tự động hóa bao gồm các script Python đơn giản cho các hoạt động như:

-   Quản lý dịch vụ: giữ các dịch vụ chạy (ví dụ, khởi động lại sau các segfault — lỗi phân đoạn bộ nhớ)
-   Theo dõi các dịch vụ nào nên chạy trên các machine nào
-   Phân tích thông báo log (nhật ký): SSH vào từng machine và tìm kiếm các regex (biểu thức chính quy)

Cuối cùng, hệ thống tự động hóa đã phát triển thành một cơ sở dữ liệu chính thức, theo dõi trạng thái của các máy và tích hợp các công cụ giám sát tinh vi hơn. Nhờ bộ công cụ tự động hóa này, chúng tôi có thể tự động quản lý phần lớn vòng đời của máy: phát hiện khi máy gặp sự cố, gỡ bỏ dịch vụ, gửi đi sửa chữa và khôi phục cấu hình khi máy trở lại.

Nhưng nhìn rộng ra, tự động hóa này tuy hữu ích lại bị giới hạn sâu sắc, do các trừu tượng của hệ thống bị ràng buộc cứng nhắc vào các machine vật lý. Chúng tôi cần một cách tiếp cận mới, do đó Borg [[Ver15]](https://sre.google/sre-book/bibliography#Ver15) ra đời: một hệ thống thoát khỏi mô hình gán host/cổng/task tương đối tĩnh của thế giới trước đó, hướng đến việc coi một tập hợp các machine như một biển tài nguyên được quản lý. Ở trung tâm của thành công — và cả của chính sự hình thành — hệ thống này là khái niệm biến quản lý cluster thành một thực thể có thể phát ra các lời gọi API, nhắm đến một bộ điều phối tập trung. Điều này giải phóng thêm các chiều hiệu quả, linh hoạt và đáng tin cậy: không giống như mô hình "sở hữu" machine trước đó, Borg có thể lên lịch cho các machine chạy, ví dụ, cả task batch lẫn task hướng người dùng trên cùng một machine.

Cuối cùng, chức năng này dẫn đến việc nâng cấp hệ điều hành liên tục và tự động với một lượng rất nhỏ nỗ lực không đổi<sup>[7](#fn7)</sup> — nỗ lực mà *không* scale theo tổng kích thước của các triển khai production. Những sai lệch nhỏ trong trạng thái machine giờ được tự động sửa chữa; sự hỏng hóc và quản lý vòng đời về cơ bản là no-ops (thao tác không làm gì) cho SRE vào thời điểm này. Hàng nghìn machine được sinh ra, chết và được đưa đi sửa mỗi ngày mà không cần SRE can thiệp. Để lặp lại lời của Ben Treynor Sloss: bằng cách coi đây là một vấn đề phần mềm, tự động hóa ban đầu đã mua cho chúng tôi đủ thời gian để biến quản lý cluster thành một thứ tự trị, chứ không chỉ là được tự động hóa. Chúng tôi đạt được mục tiêu này bằng cách áp dụng các ý tưởng về phân phối dữ liệu, API, kiến trúc hub-and-spoke (trục và nan hoa) và phát triển phần mềm hệ thống phân tán cổ điển vào lĩnh vực quản lý hạ tầng.

Một phép loại suy thú vị có thể áp dụng ở đây: chúng tôi có thể thiết lập một ánh xạ trực tiếp giữa trường hợp đơn machine và sự phát triển của các trừu tượng quản lý cluster. Dưới góc nhìn này, việc lên lịch lại trên một machine khác trông giống hệt như một process (quá trình) di chuyển từ CPU này sang CPU khác: dĩ nhiên, các tài nguyên tính toán đó nằm ở đầu kia của một liên kết mạng, nhưng điều đó thực sự quan trọng ở mức nào? Suy nghĩ theo những thuật ngữ này, việc lên lịch lại trông như một đặc tính vốn có của hệ thống chứ không phải một thứ gì đó mà bạn sẽ "tự động hóa" — con người dù sao cũng không thể phản ứng đủ nhanh. Tương tự trong trường hợp khởi động cluster: trong phép ẩn dụ này, khởi động cluster đơn giản là năng lực có thể lên lịch thêm, một chút giống như thêm disk hay RAM vào một máy tính đơn lẻ. Tuy nhiên, một máy tính đơn node, nhìn chung, không được kỳ vọng tiếp tục vận hành khi có một số lượng lớn các thành phần hỏng. Máy tính toàn cầu thì khác — nó *phải* tự sửa chữa để vận hành một khi lớn vượt quá một kích thước nhất định, do số lượng lớn sự hỏng hóc được thống kê đảm bảo là sẽ xảy ra mỗi giây. Điều này ngụ ý rằng khi chúng tôi di chuyển các hệ thống lên hệ phân cấp từ được kích hoạt thủ công, sang được kích hoạt tự động, sang tự trị, một mức độ tự nội phản (self-introspection) nhất định là cần thiết để tồn tại.

## Độ tin cậy là Tính năng Cốt lõi (Reliability Is the Fundamental Feature)

Dĩ nhiên, để debug hiệu quả, các chi tiết về hoạt động bên trong mà cơ chế tự nội phản dựa vào cũng cần được công khai cho những người quản lý hệ thống tổng thể. Các thảo luận tương tự về tác động của tự động hóa trong những lĩnh vực phi máy tính — chẳng hạn như hàng không<sup>[8](#fn8)</sup> hay các ứng dụng công nghiệp — thường chỉ ra mặt trái của tự động hóa hiệu suất cao:<sup>[9](#fn9)</sup> khi tự động hóa dần bao phủ nhiều hoạt động hàng ngày, con người vận hành ngày càng ít có cơ hội tiếp xúc trực tiếp hữu ích với hệ thống. Không thể tránh khỏi, sẽ có lúc tự động hóa thất bại, và khi đó con người không thể vận hành hệ thống thành công. Sự lưu loát trong phản ứng của họ đã bị mai một do thiếu luyện tập, và mô hình tâm trí về những gì hệ thống *nên* làm không còn phản ánh đúng thực tế về những gì nó *đang* làm.<sup>[10](#fn10)</sup> Tình huống này dễ xảy ra hơn ở các hệ thống không tự trị — tức là nơi tự động hóa thay thế các thao tác thủ công, và người ta mặc định rằng các thao tác thủ công này luôn có thể thực hiện và khả dụng như trước. Thật đáng buồn, theo thời gian, điều này rốt cuộc trở nên sai: các thao tác thủ công đó không phải lúc nào cũng có thể thực hiện được vì các chức năng hỗ trợ chúng không còn tồn tại nữa.

Chúng tôi cũng từng gặp một số trường hợp tự động hóa chủ động gây hại — xem [Tự động hóa: Cho phép Sự cố ở Quy mô](#tu-dong-hoa-cho-phe-that-bai-o-quy-mo) — nhưng theo kinh nghiệm của Google, có nhiều hệ thống mà với chúng, tự động hóa hoặc hành vi tự trị không còn là các tiện ích tùy chọn. Khi bạn scale, dĩ nhiên điều này đúng, nhưng vẫn có những lập luận mạnh mẽ cho hành vi tự trị hơn của các hệ thống bất kể kích thước. Độ tin cậy là tính năng cốt lõi, và hành vi tự trị, chống chịu là một cách hữu ích để đạt được điều đó.

## Các Khuyến nghị (Recommendations)

Đọc các ví dụ trong chương này, bạn có thể nghĩ rằng phải ở quy mô như Google thì mới có gì để làm với tự động hóa. Quan điểm này sai vì hai lý do. Thứ nhất, tự động hóa mang lại nhiều hơn cả việc tiết kiệm thời gian, nên đáng để triển khai trong nhiều trường hợp hơn so với những gì một phép tính đơn giản giữa chi phí và lợi ích có thể gợi ý. Thứ hai, cách tiếp cận có đòn bẩy cao nhất thực sự nằm ở giai đoạn thiết kế: việc vận chuyển và lặp lại nhanh giúp triển khai chức năng nhanh hơn, nhưng hiếm khi tạo ra một hệ thống chống chịu. Việc retrofit (trang bị bổ sung cho hệ thống sẵn có) các hệ thống đủ lớn để vận hành tự trị là điều khó thuyết phục, nhưng các thực hành tốt chuẩn trong kỹ thuật phần mềm sẽ giúp đáng kể: tách rời các hệ thống con, giới thiệu các API, tối thiểu hóa các tác dụng phụ, v.v.

<a id="tu-dong-hoa-cho-phe-that-bai-o-quy-mo"></a>

### Tự động hóa: Cho phép Sự cố ở Quy mô (Automation: Enabling Failure at Scale)

Google vận hành hơn một chục datacenter lớn của chính mình, nhưng chúng tôi cũng phụ thuộc vào các machine trong nhiều cơ sở colocation (đặt cùng chỗ) bên thứ ba (gọi tắt là "colos"). Các machine của chúng tôi trong các colo này được dùng để terminate (kết thúc) hầu hết các kết nối đến, hoặc như một cache (bộ đệm) cho Content Delivery Network (Mạng Phân phối Nội dung) của chính chúng tôi, nhằm giảm độ trễ người dùng cuối. Vào bất kỳ thời điểm nào, một số rack đang được cài đặt hoặc decommission; cả hai quy trình này phần lớn được tự động hóa. Một bước trong quá trình decommission là ghi đè toàn bộ nội dung disk của tất cả các machine trong rack, sau đó một hệ thống độc lập xác minh việc xóa thành công. Chúng tôi gọi quy trình này là "Diskerase" (Xóa Ổ đĩa).

Một thời gian, tự động hóa chịu trách nhiệm decommission một rack cụ thể đã thất bại, nhưng chỉ sau khi bước Diskerase đã hoàn thành thành công. Sau đó, quy trình decommission được khởi động lại từ đầu để debug sự cố. Trên bản lặp lại đó, khi cố gửi tập các machine trong rack đến Diskerase, tự động hóa xác định rằng tập các machine vẫn cần được Diskerase là (đúng) rỗng. Thật không may, tập rỗng được dùng như một giá trị đặc biệt, được diễn giải có nghĩa là "mọi thứ". Điều này có nghĩa là tự động hóa đã gửi gần như tất cả các machine mà chúng tôi có trong tất cả các colo đến Diskerase.

Trong vòng vài phút, Diskerase hiệu quả cao đã wipe (xóa) các disk trên tất cả các machine trong CDN (Mạng Phân phối Nội dung) của chúng tôi, và các machine không còn có thể terminate các kết nối từ người dùng (hoặc làm bất kỳ điều gì hữu ích khác). Chúng tôi vẫn có thể phục vụ tất cả người dùng từ các datacenter của chính mình, và sau vài phút, hiệu ứng duy nhất có thể nhìn thấy từ bên ngoài là một sự tăng nhẹ về độ trễ. Theo như chúng tôi có thể xác định, rất ít người dùng nhận thấy vấn đề, nhờ lập kế hoạch năng lực tốt (ít nhất chúng tôi đã làm đúng điều đó!). Trong khi đó, chúng tôi đã dành gần như trọn hai ngày để cài đặt lại các machine trong các rack colo bị ảnh hưởng; sau đó chúng tôi dành những tuần tiếp theo để kiểm toán và thêm nhiều kiểm tra hợp lý hơn — bao gồm cả rate limiting — vào tự động hóa, và làm cho luồng công việc decommission idempotent.

<a id="fn1"></a>[1](#fn1) Đối với những người đọc đã cảm thấy rằng họ hiểu chính xác giá trị của tự động hóa, hãy bỏ qua [Giá trị cho Google SRE](#gia-tri-cho-google-sre). Tuy nhiên, hãy lưu ý rằng mô tả của chúng tôi chứa một số sắc thái có thể hữu ích để giữ trong tâm trí khi đọc phần còn lại của chương.

<a id="fn2"></a>[2](#fn2) Chuyên môn thu được trong việc xây dựng tự động hóa như vậy cũng có giá trị ngay bản thân nó; các kỹ sư vừa hiểu sâu các quy trình hiện có mà họ đã tự động hóa, vừa có thể sau đó tự động hóa các quy trình mới nhanh hơn.

<a id="fn3"></a>[3](#fn3) Xem hoạt hình XKCD tiếp theo: [*https://xkcd.com/1205/*](https://xkcd.com/1205/).

<a id="fn4"></a>[4](#fn4) Xem, ví dụ, [*https://blog.engineyard.com/2014/pets-vs-cattle*](https://blog.engineyard.com/2014/pets-vs-cattle).

<a id="fn5"></a>[5](#fn5) Dĩ nhiên, không phải mọi hệ thống cần được quản lý thực sự cung cấp các API gọi được cho quản lý — buộc một số công cụ phải dùng, ví dụ, các lời gọi CLI (dòng lệnh) hoặc các cú nhấp chuột website tự động.

<a id="fn6"></a>[6](#fn6) Chúng tôi đã nén và đơn giản hóa lịch sử này để hỗ trợ hiểu biết.

<a id="fn7"></a>[7](#fn7) Như trong một số lượng nhỏ, không đổi.

<a id="fn8"></a>[8](#fn8) Xem, ví dụ, [*https://en.wikipedia.org/wiki/Air_France_Flight_447*](https://en.wikipedia.org/wiki/Air_France_Flight_447).

<a id="fn9"></a>[9](#fn9) Xem, ví dụ, [[Bai83]](https://sre.google/sre-book/bibliography#Bai83) và [[Sar97]](https://sre.google/sre-book/bibliography#Sar97).

<a id="fn10"></a>[10](#fn10) Đây lại là một lý do tốt nữa cho các buổi diễn tập định kỳ; xem [Disaster Role Playing](https://sre.google/sre-book/accelerating-sre-on-call#xref_training_disaster-rpg).

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
