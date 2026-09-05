> **Nguyên bản:** [Chapter 27 - Reliable Product Launches at Scale](https://sre.google/sre-book/reliable-product-launches/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

## Chương 27. Ra Mắt Sản Phẩm Đáng Tin Cậy Ở Quy Mô Lớn (Reliable Product Launches at Scale)

Tác giả: Rhandeev Singh và Sebastian Kirsch cùng Vivek Rau  
Biên tập: Betsy Beyer

Các công ty Internet như Google có thể ra mắt (launch) sản phẩm và tính năng mới với chu kỳ nhanh hơn nhiều so với các công ty truyền thống. Trong quá trình này, vai trò của Site Reliability là đảm bảo tốc độ thay đổi nhanh mà không làm ảnh hưởng đến sự ổn định của dịch vụ. Chúng tôi đã thành lập một nhóm chuyên trách gồm các "Kỹ sư Điều Phối Ra Mắt (Launch Coordination Engineers)" để tư vấn cho các nhóm kỹ thuật về những khía cạnh kỹ thuật cần thiết cho một lần ra mắt thành công.

Nhóm này cũng biên soạn một “checklist ra mắt” gồm các câu hỏi phổ biến cần đặt ra cho mỗi lần ra mắt, cùng các công thức (recipes) để xử lý những vấn đề thường gặp. Checklist đã chứng minh là công cụ hữu ích, giúp đảm bảo các lần ra mắt đáng tin cậy có thể tái lập.

Hãy xem xét một dịch vụ Google thông thường—ví dụ, Keyhole, thứ cung cấp ảnh vệ tinh cho Google Maps và Google Earth. Vào một ngày bình thường, Keyhole phục vụ đến vài nghìn ảnh vệ tinh mỗi giây. Nhưng vào đêm Giáng Sinh năm 2011, nó nhận được lượng traffic cao gấp 25 lần đỉnh thông thường—lên đến một triệu request mỗi giây. Điều gì đã gây ra sự tăng vọt traffic khổng lồ này?

## Ông già Noel đang đến. (Santa was coming.)

Vài năm trước, Google đã phối hợp với NORAD (North American Aerospace Defense Command - Bộ Tư Lệnh Phòng Thủ Vũ Trụ và Không Gian Bắc Mỹ) để ra mắt một website theo dõi hành trình của Ông già Noel quanh thế giới trong dịp Giáng Sinh, cho phép người dùng xem ông phân phát quà theo thời gian thực. Một phần của trải nghiệm này là tính năng "bay qua ảo (virtual fly-over)", sử dụng ảnh vệ tinh để theo dõi hành trình của Ông già Noel trên một thế giới mô phỏng.

Dù dự án như NORAD Tracks Santa có vẻ lãng mạn, nó mang đầy đủ đặc điểm của một lần ra mắt khó khăn và rủi ro: một hạn chót cứng (Google không thể xin Ông già Noel đến muộn một tuần nếu website chưa sẵn sàng), sự chú ý của công chúng rất lớn, khán giả hàng triệu người, và lượng traffic tăng rất dốc (mọi người sẽ cùng xem website vào đêm Giáng Sinh). Đừng bao giờ đánh giá thấp sức mạnh của hàng triệu trẻ nhỏ nóng lòng chờ quà—dự án này có khả năng rất thực sự để quật ngã các server của Google.

Nhóm Site Reliability Engineering của Google đã nỗ lực chuẩn bị hạ tầng cho lần ra mắt này, đảm bảo Ông già Noel kịp thời phân phát toàn bộ quà tặng trước ánh mắt chăm chú của khán giả đang hồi hộp chờ đợi. Điều cuối cùng chúng tôi muốn là để trẻ con khóc vì không thể xem Ông già Noel tặng quà. Thực tế, chúng tôi đã đặt tên cho các kill switch được tích hợp vào trải nghiệm nhằm bảo vệ dịch vụ là những “công tắc khiến trẻ con khóc”. Việc dự đoán các kịch bản sự cố có thể xảy ra trong lần ra mắt và điều phối giữa các nhóm kỹ thuật tham gia là nhiệm vụ của một nhóm đặc biệt trong Site Reliability Engineering: các Kỹ sư Điều Phối Ra mắt (LCE).

Ra mắt một sản phẩm hoặc tính năng mới là khoảnh khắc quyết định đối với mọi công ty—thời điểm mà hàng tháng hoặc hàng năm nỗ lực được trình bày ra trước thế giới. Các công ty truyền thống ra mắt sản phẩm mới với tần suất khá thấp. Chu kỳ ra mắt ở các công ty Internet khác biệt rõ rệt. Việc ra mắt và lặp lại nhanh trở nên dễ dàng hơn nhiều, bởi các tính năng mới có thể triển khai ở phía server, thay vì phải cài đặt phần mềm trên từng workstation cá nhân của khách hàng.

Google định nghĩa một lần ra mắt là bất kỳ mã mới nào gây ra thay đổi có thể quan sát được từ bên ngoài của ứng dụng. Quy trình ra mắt có thể khác nhau đáng kể tùy thuộc vào các đặc điểm của lần ra mắt đó, bao gồm sự kết hợp các thuộc tính, thời điểm, số bước liên quan và độ phức tạp. Theo định nghĩa này, Google đôi khi thực hiện tới 70 lần ra mắt mỗi tuần.

Tốc độ thay đổi nhanh này vừa là lý do, vừa là cơ hội để xây dựng quy trình ra mắt tinh gọn. Một công ty chỉ ra mắt sản phẩm mỗi ba năm chẳng cần quy trình chi tiết, bởi đến lúc ra mắt mới diễn ra, hầu hết các thành phần của quy trình cũ đã lỗi thời. Các công ty truyền thống cũng không có cơ hội thiết kế quy trình ra mắt chi tiết, vì họ không tích lũy đủ kinh nghiệm từ các lần ra mắt để tạo ra một quy trình vững chắc và trưởng thành.

## Kỹ Thuật Điều Phối Ra Mắt (Launch Coordination Engineering)

Kỹ sư phần mềm giỏi thường có chuyên môn sâu về lập trình và thiết kế, đồng thời nắm vững công nghệ của chính sản phẩm mình đang phát triển. Tuy nhiên, họ có thể chưa quen với những thách thức và bẫy khi ra mắt sản phẩm cho hàng triệu người dùng, trong khi vẫn phải giảm thiểu sự cố và tối đa hóa hiệu suất.

Để giải quyết những thách thức vốn có khi ra mắt sản phẩm, Google lập ra một nhóm tư vấn chuyên trách thuộc SRE, chịu trách nhiệm về mặt kỹ thuật cho việc ra mắt các sản phẩm hoặc tính năng mới. Nhóm này gồm các kỹ sư phần mềm và kỹ sư hệ thống—một số từng làm việc ở các nhóm SRE khác—chuyên hướng dẫn các nhà phát triển nhanh chóng xây dựng những sản phẩm đáng tin cậy, đáp ứng các tiêu chuẩn của Google về sự vững chắc, khả năng mở rộng và độ tin cậy. Nhóm tư vấn Launch Coordination Engineering (LCE) tạo thuận lợi cho một quy trình ra mắt suôn sẻ theo một vài cách:

-   Thăm dò (audit) các sản phẩm và dịch vụ để đánh giá mức độ tuân thủ [các tiêu chuẩn độ tin cậy của Google](https://sre.google/resources/practices-and-processes/product-focused-reliability-for-sre/) và các thực hành tốt nhất, đồng thời đề xuất các hành động cụ thể nhằm cải thiện độ tin cậy
-   Đóng vai trò cầu nối giữa các nhóm khác nhau tham gia vào một lần ra mắt
-   Thúc đẩy các khía cạnh kỹ thuật của một lần ra mắt bằng cách đảm bảo rằng các tác vụ duy trì động lực
-   Đóng vai trò những người gác cổng và ký duyệt các lần ra mắt được xác định là "an toàn"
-   Hướng dẫn các nhà phát triển về các thực hành tốt nhất và [cách tích hợp với các dịch vụ của Google](https://sre.google/sre-book/service-best-practices/), đồng thời cung cấp [tài liệu và tài nguyên đào tạo nội bộ](https://sre.google/resources/) để giúp họ học nhanh hơn

Các thành viên nhóm LCE thăm dò dịch vụ ở những thời điểm khác nhau trong vòng đời của dịch vụ. Phần lớn các cuộc thăm dò diễn ra trước khi một sản phẩm hoặc dịch vụ mới ra mắt. Nếu một nhóm phát triển sản phẩm thực hiện lần ra mắt mà không có hỗ trợ SRE, LCE sẽ cung cấp kiến thức chuyên ngành phù hợp để đảm bảo quá trình ra mắt suôn sẻ. Tuy nhiên, ngay cả những sản phẩm đã có hỗ trợ SRE mạnh vẫn thường xuyên tương tác với nhóm LCE trong các lần ra mắt quan trọng. Những thách thức mà các nhóm phải đối mặt khi ra mắt sản phẩm mới khác biệt đáng kể so với hoạt động hàng ngày của một dịch vụ đáng tin cậy (một nhiệm vụ mà các nhóm SRE đã làm rất tốt), và nhóm LCE có thể rút kinh nghiệm từ hàng trăm lần ra mắt. Nhóm LCE cũng tạo thuận lợi cho các cuộc thăm dò dịch vụ khi các dịch vụ mới bắt đầu tương tác với SRE.

### Vai Trò của Kỹ Sư Điều Phối Ra Mắt (The Role of the Launch Coordination Engineer)

Nhóm Launch Coordination Engineering của chúng tôi gồm các Kỹ sư Điều Phối Ra Mắt (LCE). Đây là những người được tuyển dụng trực tiếp vào vị trí này, hoặc là các SRE có kinh nghiệm thực chiến trong việc vận hành các dịch vụ của Google. Các LCE phải đáp ứng cùng những yêu cầu kỹ thuật như bất kỳ SRE nào khác, đồng thời được kỳ vọng có kỹ năng giao tiếp và lãnh đạo vững chắc. Một LCE sẽ tập hợp các bên liên quan để cùng hướng tới mục tiêu chung, dàn xếp những xung đột thỉnh thoảng xảy ra, và hướng dẫn, huấn luyện, giáo dục các kỹ sư đồng nghiệp.

Một nhóm chuyên trách điều phối các lần ra mắt mang lại những lợi thế sau:

#### Chiều rộng kinh nghiệm (Breadth of experience)

Là nhóm thực sự liên sản phẩm (cross-product), các thành viên hoạt động trên gần như tất cả các lĩnh vực sản phẩm của Google. Nhờ kiến thức liên sản phẩm rộng rãi và mối quan hệ với nhiều nhóm trên khắp công ty, các LCE trở thành phương tiện xuất sắc để chuyển giao kiến thức.

#### Góc nhìn liên chức năng (Cross-functional perspective)

LCE nắm được bức tranh tổng thể của lần ra mắt, nhờ đó có thể điều phối giữa các nhóm SRE, phát triển và quản lý sản phẩm. Cách tiếp cận này đặc biệt quan trọng với những lần ra mắt phức tạp, có thể trải dài hơn nửa tá nhóm trên nhiều múi giờ khác nhau.

#### Tính khách quan (Objectivity)

LCE đóng vai trò cố vấn trung lập, giúp cân bằng và dàn xếp giữa các bên liên quan như SRE, nhà phát triển sản phẩm, quản lý sản phẩm và tiếp thị.

Vì Launch Coordination Engineer là một vai trò SRE, các LCE luôn có động lực ưu tiên độ tin cậy hơn các mối quan tâm khác. Một công ty không chia sẻ các mục tiêu độ tin cậy của Google, nhưng chia sẻ tốc độ thay đổi nhanh của Google, có thể chọn một cấu trúc động lực khác.

## Thiết Lập Một Quy Trình Ra Mắt (Setting Up a Launch Process)

Hơn 10 năm qua, Google không ngừng hoàn thiện quy trình ra mắt. Trong quá trình đó, chúng tôi đã xác định được một số tiêu chí làm nên một quy trình ra mắt tốt:

#### Lightweight (Nhẹ nhàng)

Dễ chịu với các nhà phát triển

#### Robust (Vững chắc)

Bắt được các lỗi hiển nhiên

#### Thorough (Chu đáo)

Giải quyết các chi tiết quan trọng một cách nhất quán và có thể tái lập

#### Scalable (Có thể mở rộng)

Ứng phó được với cả một số lượng lớn các lần ra mắt đơn giản và ít lần ra mắt phức tạp hơn

#### Adaptable (Có thể thích ứng)

Hoạt động tốt cho cả các loại ra mắt phổ biến (ví dụ, thêm một ngôn ngữ UI mới vào một sản phẩm) lẫn các loại ra mắt mới (ví dụ, lần ra mắt ban đầu của trình duyệt Chrome hoặc Google Fiber)

Như bạn có thể thấy, một số yêu cầu này xung đột rõ ràng với nhau. Ví dụ, khó mà thiết kế một quy trình vừa nhẹ nhàng vừa chu đáo. Việc cân bằng các tiêu chí này đòi hỏi sự điều chỉnh liên tục. Google đã áp dụng thành công một vài chiến thuật giúp chúng tôi đạt được những tiêu chí đó:

#### Simplicity (Sự đơn giản)

Làm đúng những điều cơ bản. Đừng lên kế hoạch cho mọi sự cố.

#### A high touch approach (Một cách tiếp cận chăm chút)

Các kỹ sư giàu kinh nghiệm tùy chỉnh quy trình cho phù hợp với từng lần ra mắt.

#### Fast common paths (Các đường chung nhanh)

Xác định các lớp lần ra mắt luôn tuân theo một pattern (hình mẫu) chung (chẳng hạn như ra mắt một sản phẩm ở một quốc gia mới), và cung cấp một quy trình ra mắt được tinh giản cho lớp này.

Kinh nghiệm đã chứng minh rằng các kỹ sư có khả năng lách qua các quy trình mà họ cho là quá nặng nề hoặc mang lại giá trị không đủ—đặc biệt khi một nhóm đã ở chế độ crunch (tăng tốc gấp), và quy trình ra mắt được xem như chỉ là một mục khác đang chặn lần ra mắt của họ. Vì lý do này, LCE phải tối ưu hóa trải nghiệm ra mắt liên tục để đạt được sự cân bằng đúng giữa chi phí và lợi ích.

### Checklist Ra Mắt (The Launch Checklist)

Checklist được dùng để giảm thiểu sai sót, đồng thời đảm bảo tính nhất quán và đầy đủ trong nhiều lĩnh vực khác nhau. Một số ví dụ phổ biến là checklist trước khi bay trong ngành hàng không và checklist phẫu thuật [[Gaw09]](https://sre.google/sre-book/bibliography#Gaw09). Tương tự, LCE sử dụng checklist ra mắt để xác nhận (qualify) một lần ra mắt. Checklist ([Launch Coordination Checklist](https://sre.google/sre-book/launch-checklist/)) hỗ trợ LCE đánh giá lần ra mắt, đồng thời cung cấp cho nhóm ra mắt các hành động cần thực hiện và đường dẫn đến nhiều thông tin chi tiết hơn. Dưới đây là một số ví dụ về các mục có thể xuất hiện trong checklist:

-   **Câu hỏi**: Bạn có cần một tên miền (domain name) mới?
    
    -   **Hành động cần thực hiện**: Phối hợp với tiếp thị về tên miền mong muốn của bạn, và yêu cầu đăng ký tên miền. Đây là một liên kết đến biểu mẫu tiếp thị.
-   **Câu hỏi**: Bạn có đang lưu trữ dữ liệu lâu dài (persistent data) không?
    
    -   **Hành động cần thực hiện**: Đảm bảo rằng bạn triển khai backup. Đây là các hướng dẫn để triển khai backup.
-   **Câu hỏi**: Một người dùng có thể lạm dụng dịch vụ của bạn không?
    
    -   **Hành động cần thực hiện**: Triển khai rate limiting và quota (định mức). Sử dụng dịch vụ chia sẻ sau.

Thực tế, số câu hỏi có thể đặt ra cho bất kỳ hệ thống nào gần như vô hạn, và checklist rất dễ phình to đến mức không kiểm soát nổi. Muốn gánh nặng đặt lên vai các nhà phát triển ở mức chấp nhận được, cần biên soạn checklist thật kỹ lưỡng. Để kìm đà tăng trưởng ấy, có thời điểm việc thêm câu hỏi mới vào checklist ra mắt của Google phải được một phó chủ tịch phê duyệt. Hiện tại, LCE áp dụng các hướng dẫn sau:

-   Sự quan trọng của mọi câu hỏi phải được chứng minh, lý tưởng là bằng một thảm họa ra mắt trước đó.
-   Mọi chỉ dẫn phải cụ thể, thực tế, và hợp lý để các nhà phát triển hoàn thành.

Checklist cần được chú ý liên tục để vẫn còn giá trị và cập nhật: các khuyến nghị thay đổi theo thời gian, các hệ thống nội bộ bị thay thế, và những lo ngại từ các lần ra mắt trước đó trở nên lỗi thời do chính sách và quy trình mới. Các LCE biên soạn checklist liên tục và thực hiện các cập nhật nhỏ khi thành viên nhóm phát hiện mục cần sửa. Một hoặc hai lần một năm, một thành viên nhóm xem xét toàn bộ checklist để xác định các mục lỗi thời, sau đó làm việc với chủ sở hữu dịch vụ và chuyên gia về chủ đề để hiện đại hóa các phần tương ứng.

### Thúc Đẩy Sự Hội Tụ và Đơn Giản Hóa (Driving Convergence and Simplification)

Trong một tổ chức lớn, các kỹ sư có thể không hay biết về hạ tầng sẵn có cho những tác vụ phổ biến (chẳng hạn như rate limiting). Nếu thiếu sự hướng dẫn phù hợp, họ dễ dàng tự xây dựng lại các giải pháp vốn đã tồn tại. Việc hội tụ về một tập các thư viện hạ tầng chung giúp tránh kịch bản này và mang lại những lợi ích hiển nhiên cho công ty: nó cắt giảm công việc trùng lặp, giúp kiến thức dễ chuyển giao hơn giữa các dịch vụ, và nâng cao mức độ chất lượng kỹ thuật cũng như dịch vụ nhờ sự chú ý được tập trung vào hạ tầng.

Gần như tất cả các nhóm tại Google đều tham gia một quy trình ra mắt chung, qua đó checklist ra mắt trở thành công cụ thúc đẩy sự hội tụ về hạ tầng chung. Thay vì triển khai giải pháp tùy chỉnh, LCE có thể khuyến nghị sử dụng hạ tầng hiện có như các khối xây dựng—loại hạ tầng đã được củng cố qua nhiều năm kinh nghiệm, giúp giảm thiểu rủi ro về năng lực, hiệu suất hoặc khả năng mở rộng. Các ví dụ bao gồm hạ tầng chung cho rate limiting hoặc quota người dùng, đẩy dữ liệu mới đến các server, hoặc phát hành các phiên bản mới của một binary (file nhị phân). Việc chuẩn hóa này đã giúp đơn giản hóa đáng kể checklist ra mắt: chẳng hạn, các phần dài xử lý yêu cầu về rate limiting có thể được thay bằng một dòng duy nhất ghi, "Triển khai rate limiting sử dụng hệ thống X."

Nhờ bề dày kinh nghiệm trải dài trên toàn bộ các sản phẩm của Google, các LCE cũng nắm giữ một vị thế độc đáo để nhận diện những cơ hội đơn giản hóa. Trong quá trình tham gia một lần ra mắt, họ trực tiếp chứng kiến các chướng ngại vật: phần nào của quy trình ra mắt gây ra nhiều rắc rối nhất, bước nào tiêu tốn thời gian không tương xứng, vấn đề nào được giải quyết độc lập lặp đi lặp lại theo những cách tương tự, hạ tầng chung bị thiếu ở đâu, hoặc sự trùng lặp tồn tại ở đâu trong hạ tầng chung. Các LCE có nhiều cách để tinh giản trải nghiệm ra mắt và đóng vai trò những người ủng hộ cho các nhóm ra mắt. Ví dụ, các LCE có thể làm việc với các chủ sở hữu của một quy trình phê duyệt đặc biệt vất vả để đơn giản hóa các tiêu chí của họ và triển khai các phê duyệt tự động cho các trường hợp phổ biến. Các LCE cũng có thể đẩy cao (escalate) các điểm đau đến các chủ sở hữu của hạ tầng chung và tạo ra một cuộc đối thoại với các khách hàng. Bằng cách tận dụng kinh nghiệm đạt được qua nhiều lần ra mắt trước đó, các LCE có thể dành nhiều sự chú ý hơn cho các mối quan tâm và đề xuất cá nhân.

### Ra Mắt Điều Không Mong Đợi (Launching the Unexpected)

Khi một dự án mở rộng sang không gian hoặc ngành dọc sản phẩm mới, LCE có thể phải xây dựng checklist từ đầu. Quá trình này thường dựa trên kinh nghiệm tổng hợp từ các chuyên gia trong các lĩnh vực liên quan. Khi phác thảo checklist mới, nên cấu trúc nó xoay quanh các chủ đề rộng như độ tin cậy, các chế độ lỗi và các quy trình.

Ví dụ, trước khi ra mắt Android, Google hiếm khi xử lý các thiết bị tiêu dùng hàng loạt có logic phía client mà chúng tôi không trực tiếp kiểm soát. Trong khi chúng tôi có thể khá dễ dàng sửa một bug trong Gmail trong vòng vài giờ hoặc vài ngày bằng cách đẩy các phiên bản mới của JavaScript đến các trình duyệt, những bản sửa như vậy không phải là một tùy chọn với các thiết bị di động. Do đó, các LCE làm việc trên các lần ra mắt di động đã thu hút các chuyên gia trong lĩnh vực di động để xác định những phần của các checklist hiện có áp dụng hoặc không áp dụng, và ở đâu các câu hỏi checklist mới cần thiết. Trong những cuộc trò chuyện như vậy, điều quan trọng là giữ trong tâm trí *ý định* của mỗi câu hỏi để tránh áp dụng một cách máy móc một câu hỏi hoặc hành động cụ thể không liên quan đến thiết kế của sản phẩm độc đáo đang ra mắt. Một LCE đối mặt với một lần ra mắt bất thường phải quay về các nguyên lý cơ bản (first principles) về cách thực hiện một lần ra mắt an toàn, rồi từ đó cụ thể hóa lại để làm cho checklist cụ thể và hữu ích cho các nhà phát triển.

## Phát Triển Một Checklist Ra Mắt (Developing a Launch Checklist)

Checklist là yếu tố bắt buộc để ra mắt dịch vụ và sản phẩm mới với độ tin cậy có thể tái lập. Checklist ra mắt của chúng tôi không ngừng mở rộng theo thời gian và được các thành viên nhóm Launch Coordination Engineering cập nhật định kỳ. Nội dung checklist ra mắt sẽ khác nhau giữa các công ty, bởi các chi tiết cụ thể phải được tùy chỉnh cho phù hợp với dịch vụ và hạ tầng nội bộ của từng nơi. Trong các phần tiếp theo, chúng tôi trích xuất một số chủ đề từ checklist LCE của Google và đưa ra ví dụ về cách cụ thể hóa những chủ đề này.

### Kiến Trúc và Các Sự Phụ Thuộc (Architecture and Dependencies)

Việc rà soát kiến trúc giúp bạn kiểm tra xem dịch vụ có sử dụng hạ tầng chia sẻ đúng cách hay không, đồng thời xác định các chủ sở hữu của hạ tầng đó như những bên liên quan bổ sung trong lần ra mắt. Google sở hữu một lượng lớn dịch vụ nội bộ, đóng vai trò như các khối xây dựng cho các sản phẩm mới. Ở các giai đoạn sau của việc lập kế hoạch năng lực (xem [[Hix15a]](https://sre.google/sre-book/bibliography#Hix15a)), danh sách các sự phụ thuộc được xác định trong phần này của checklist có thể dùng để đảm bảo mọi sự phụ thuộc đều được cung cấp đúng cách.

#### Các câu hỏi checklist ví dụ

-   Dòng request của bạn từ user đến frontend đến backend là gì?
-   Có các loại request khác nhau với các yêu cầu latency khác nhau không?

#### Các hành động cần thực hiện ví dụ

-   Cô lập các request hướng đến người dùng khỏi các request không hướng đến người dùng.
-   Xác thực các giả định về lượng request. Một lượt xem trang có thể biến thành nhiều request.

### Tích Hợp (Integration)

Nhiều dịch vụ của các công ty vận hành trong một hệ sinh thái nội bộ, đòi hỏi phải có hướng dẫn về cách thiết lập máy chủ, cấu hình dịch vụ mới, thiết lập giám sát, tích hợp load balancing, cấu hình địa chỉ DNS, v.v. Những hệ sinh thái nội bộ này thường phát triển theo thời gian và mang những đặc điểm riêng cùng các bẫy cần lưu ý khi vận hành. Do đó, phần này của checklist sẽ khác nhau rất nhiều giữa các công ty.

#### Các hành động cần thực hiện ví dụ

-   Thiết lập một tên DNS mới cho dịch vụ của bạn.
-   Thiết lập các load balancer để nói chuyện với dịch vụ của bạn.
-   Thiết lập giám sát cho dịch vụ mới của bạn.

### Lập Kế Hoạch Năng Lực (Capacity Planning)

Các tính năng mới có thể gây ra một đợt tăng sử dụng tạm thời khi ra mắt, nhưng thường sẽ hạ nhiệt trong vòng vài ngày. Loại workload hoặc sự pha trộn traffic từ đợt tăng đột biến này có thể khác biệt đáng kể so với trạng thái ổn định, khiến kết quả kiểm thử tải bị lệch. Sự quan tâm của công chúng nổi tiếng là khó dự đoán, và một số sản phẩm của Google đã phải đối mặt với các đợt tăng đột biến khi ra mắt cao gấp 15 lần so với ước tính ban đầu. Việc ra mắt ban đầu ở một khu vực hoặc một quốc gia mỗi lần giúp xây dựng niềm tin để xử lý các lần ra mắt lớn hơn.

Năng lực gắn liền chặt chẽ với mức dự phòng và khả năng vận hành liên tục. Chẳng hạn, nếu cần ba triển khai được nhân bản để xử lý 100% traffic vào giờ cao điểm, bạn phải duy trì bốn hoặc năm triển khai, trong đó một hoặc hai cái làm dự phòng, nhằm bảo vệ người dùng khỏi các hoạt động bảo trì và sự cố bất ngờ. Các tài nguyên datacenter và mạng thường có thời gian chờ (lead time) dài, nên cần yêu cầu đủ sớm để công ty có thể tiếp nhận chúng.

#### Các câu hỏi checklist ví dụ

-   Lần ra mắt này có được gắn với một thông cáo báo chí, quảng cáo, bài blog, hoặc một hình thức tiếp thị khác không?
-   Bạn kỳ vọng bao nhiêu traffic và tốc độ tăng trưởng trong và sau lần ra mắt?
-   Bạn đã đạt được tất cả các tài nguyên tính toán cần thiết để hỗ trợ traffic của bạn chưa?

### Các Chế Độ Lỗi (Failure Modes)

Xét một cách có hệ thống các chế độ lỗi có thể xảy ra của dịch vụ mới là cách đảm bảo độ tin cậy cao ngay từ đầu. Trong phần này của checklist, hãy xem xét từng thành phần và sự phụ thuộc, đồng thời xác định tác động khi chúng hỏng. Dịch vụ có chịu được sự cố của một máy đơn lẻ không? Sự cố datacenter? Sự cố mạng? Chúng ta xử lý dữ liệu đầu vào xấu ra sao? Đã sẵn sàng cho khả năng xảy ra cuộc tấn công từ chối dịch vụ (denial-of-service - DoS) chưa? Dịch vụ có thể tiếp tục phục vụ ở chế độ suy giảm (degraded mode) nếu một trong các sự phụ thuộc bị hỏng không? Chúng ta xử lý sự không khả dụng của một sự phụ thuộc vào lúc khởi động dịch vụ như thế nào? Trong khi chạy (during runtime)?

#### Các câu hỏi checklist ví dụ

-   Bạn có bất kỳ điểm thất bại duy nhất (single point of failure) nào trong thiết kế của bạn không?
-   Bạn giảm thiểu sự không khả dụng của các sự phụ thuộc của mình như thế nào?

#### Các hành động cần thực hiện ví dụ

-   Triển khai các deadline cho request để tránh cạn kiệt tài nguyên cho các request chạy lâu.
-   Triển khai loại bỏ tải (load shedding) để từ chối các request mới sớm trong các tình trạng quá tải.

### Hành Vi của Client (Client Behavior)

Trên website truyền thống, hiếm khi cần lo ngại về hành vi lạm dụng từ người dùng hợp pháp. Vì mọi request đều do hành động của người dùng kích hoạt, chẳng hạn như một cú click vào liên kết, nên tần suất request bị giới hạn bởi tốc độ click của họ. Muốn tăng gấp đôi tải, số lượng người dùng cũng phải tăng gấp đôi.

Điều này không còn đúng khi xét đến các client khởi tạo hành động mà không có đầu vào từ người dùng—ví dụ, một app điện thoại định kỳ đồng bộ hóa (sync) dữ liệu lên cloud, hoặc một website định kỳ làm mới (refresh). Trong cả hai kịch bản này, hành vi lạm dụng của client có thể rất dễ dàng đe dọa sự ổn định của dịch vụ. (Ngoài ra còn có chủ đề bảo vệ dịch vụ khỏi traffic lạm dụng, chẳng hạn như các scraper (trình cào dữ liệu) và các cuộc tấn công từ chối dịch vụ—điều này khác với việc thiết kế hành vi an toàn cho các client bên thứ nhất (first-party).)

#### Câu hỏi checklist ví dụ

-   Bạn có tính năng auto-save/auto-complete/heartbeat không?

#### Các hành động cần thực hiện ví dụ

-   Đảm bảo rằng client của bạn back off theo cấp số mũ khi thất bại.
-   Đảm bảo rằng bạn jitter các request tự động.

### Các Quy Trình và Tự Động Hóa (Processes and Automation)

Google khuyến khích kỹ sư dùng các công cụ chuẩn để tự động hóa những quy trình phổ biến. Dù vậy, tự động hóa không bao giờ hoàn hảo; mọi dịch vụ đều có những thao tác bắt buộc phải do con người thực hiện, chẳng hạn như tạo một release (phiên bản phát hành) mới, di chuyển dịch vụ sang datacenter khác, hoặc khôi phục dữ liệu từ backup. Vì lý do độ tin cậy, chúng tôi nỗ lực giảm thiểu các điểm thất bại duy nhất, kể cả con người.

Cần tài liệu hóa các quy trình còn lại trước khi ra mắt, để kịp ghi lại thông tin khi nó còn mới trong đầu kỹ sư, đồng thời đảm bảo có sẵn khi xảy ra sự cố khẩn cấp. Nội dung tài liệu phải đủ rõ ràng để bất kỳ thành viên nào trong nhóm cũng có thể thực hiện quy trình tương ứng khi cần.

#### Câu hỏi checklist ví dụ

-   Có bất kỳ quy trình thủ công nào được yêu cầu để giữ cho dịch vụ chạy không?

#### Các hành động cần thực hiện ví dụ

-   Tài liệu hóa tất cả các quy trình thủ công.
-   Tài liệu hóa quy trình để di chuyển dịch vụ của bạn đến một datacenter mới.
-   Tự động hóa quy trình để xây dựng và phát hành một phiên bản mới.

### Quy Trình Phát Triển (Development Process)

Google sử dụng hệ thống kiểm soát phiên bản rất rộng rãi, và hầu hết các quy trình phát triển đều gắn chặt với hệ thống này. Nhiều thực hành tốt nhất của chúng tôi xoay quanh cách khai thác hiệu quả hệ thống kiểm soát phiên bản. Chẳng hạn, phần lớn công việc phát triển được thực hiện trên nhánh mainline, trong khi các release được xây dựng trên các nhánh riêng tương ứng. Cách tổ chức này giúp việc sửa lỗi trong một release trở nên dễ dàng mà không kéo theo những thay đổi không liên quan từ mainline.

Google cũng dùng version control cho nhiều mục đích khác, chẳng hạn như lưu trữ các file cấu hình. Nhiều lợi ích của version control — theo dõi lịch sử, gán các thay đổi cho các cá nhân, và code review — cũng áp dụng cho các file cấu hình. Trong một số trường hợp, chúng tôi tự động đẩy các thay đổi từ hệ thống kiểm soát phiên bản đến các server trực tiếp, giúp một kỹ sư chỉ cần đệ trình một thay đổi là có thể đưa nó lên production.

#### Các hành động cần thực hiện ví dụ

-   Kiểm tra tất cả mã và file cấu hình vào hệ thống version control.
-   Cắt mỗi release trên một nhánh release mới.

### Các Sự Phụ Thuộc Bên Ngoài (External Dependencies)

Đôi khi, một lần ra mắt phụ thuộc vào các yếu tố nằm ngoài tầm kiểm soát của công ty. Việc xác định những yếu tố này giúp bạn giảm thiểu sự khó lường mà chúng gây ra. Chẳng hạn, sự phụ thuộc có thể là một thư viện mã do bên thứ ba duy trì, hoặc một dịch vụ, dữ liệu do công ty khác cung cấp. Khi một sự cố từ nhà cung cấp, bug, lỗi có hệ thống, vấn đề bảo mật, hoặc giới hạn khả năng mở rộng bất ngờ xảy ra, việc lập kế hoạch trước sẽ giúp bạn ngăn chặn hoặc giảm nhẹ tổn hại cho người dùng. Trong lịch sử ra mắt của Google, chúng tôi đã sử dụng các proxy lọc và/hoặc viết lại, các pipeline transcoding (chuyển đổi mã hóa) dữ liệu, và cache để giảm thiểu một số rủi ro này.

#### Các câu hỏi checklist ví dụ

-   Mã, dữ liệu, dịch vụ, hoặc sự kiện bên thứ ba nào mà dịch vụ hoặc lần ra mắt phụ thuộc vào?
-   Có đối tác nào phụ thuộc vào dịch vụ của bạn không? Nếu có, họ có cần được thông báo về lần ra mắt của bạn không?
-   Điều gì sẽ xảy ra nếu bạn hoặc nhà cung cấp không thể đáp ứng một hạn chót ra mắt cứng?

### Lập Kế Hoạch Triển Khai (Rollout Planning)

Trong các hệ thống phân tán lớn, hiếm khi có sự kiện nào xảy ra tức thời. Vì lý do độ tin cậy, sự tức thời như vậy thường không lý tưởng dù sao. Một lần ra mắt phức tạp có thể yêu cầu kích hoạt từng tính năng riêng lẻ trên một số hệ thống phụ khác nhau, và mỗi thay đổi cấu hình đó có thể mất hàng giờ để hoàn thành. Một cấu hình chạy tốt trên instance thử nghiệm không có nghĩa là cùng cấu hình đó có thể được triển khai đến instance trực tiếp. Đôi khi cần một vũ điệu phức tạp hoặc tính năng đặc biệt để làm cho tất cả các thành phần ra mắt sạch sẽ và theo đúng thứ tự.

Các yêu cầu từ bên ngoài, chẳng hạn từ bộ phận tiếp thị hay PR (quan hệ công chúng), cũng có thể gây thêm phiền toái. Chẳng hạn, một nhóm có thể cần một tính năng sẵn sàng kịp cho bài thuyết trình chính (keynote) tại hội nghị, nhưng lại phải giữ tính năng đó ở trạng thái ẩn cho đến trước khi keynote diễn ra.

Các biện pháp dự phòng là một phần khác của lập kế hoạch triển khai. Nếu không kịp kích hoạt tính năng cho bài keynote thì sao? Đôi khi những biện pháp dự phòng này đơn giản như chuẩn bị một bộ slide dự phòng ghi, "Chúng tôi sẽ ra mắt tính năng này trong vài ngày tới" thay vì "Chúng tôi đã ra mắt tính năng này."

#### Các hành động cần thực hiện ví dụ

-   Thiết lập một kế hoạch ra mắt xác định các hành động cần thực hiện để ra mắt dịch vụ. Xác định ai chịu trách nhiệm cho mỗi mục.
-   Xác định rủi ro trong các bước ra mắt cá nhân và triển khai các biện pháp dự phòng.

## Các Kỹ Thuật Được Chọn cho Các Lần Ra Mắt Đáng Tin Cậy (Selected Techniques for Reliable Launches)

Như đã đề cập ở các phần khác của cuốn sách, Google đã phát triển nhiều kỹ thuật để vận hành các hệ thống đáng tin cậy trong suốt nhiều năm. Một số kỹ thuật này đặc biệt hữu ích khi ra mắt sản phẩm một cách an toàn. Chúng cũng mang lại lợi ích trong quá trình vận hành bình thường của dịch vụ, nhưng việc thực hiện đúng các kỹ thuật này trong giai đoạn ra mắt lại đặc biệt quan trọng.

### Triển Khai Từ Từ và Theo Giai Đoạn (Gradual and Staged Rollouts)

Một châm ngôn trong quản trị hệ thống là "đừng bao giờ thay đổi một hệ thống đang chạy". Mọi thay đổi đều tiềm ẩn rủi ro, và rủi ro cần được giảm thiểu để đảm bảo [độ tin cậy của hệ thống.](https://sre.google/resources/practices-and-processes/product-focused-reliability-for-sre/) Điều này càng đúng hơn với các hệ thống được nhân bản cao, phân phối toàn cầu như những hệ thống do Google vận hành.

Rất ít lần ra mắt tại Google mang tính chất "nút bấm (push-button)", tức là tung một sản phẩm mới ra toàn cầu vào một thời điểm cụ thể. Theo thời gian, Google đã xây dựng một số pattern giúp ra mắt sản phẩm và tính năng một cách từ từ, qua đó giảm thiểu rủi ro; xem [A Collection of Best Practices for Production Services](https://sre.google/sre-book/service-best-practices/).

Gần như tất cả các bản cập nhật cho các dịch vụ của Google đều diễn ra từ từ, tuân theo một quy trình xác định, với các bước xác thực phù hợp được xen kẽ. Một server mới có thể được cài đặt trên một vài máy trong một datacenter và được quan sát trong một khoảng thời gian xác định. Nếu mọi thứ có vẻ ổn, server sẽ được cài đặt trên tất cả các máy trong datacenter đó, tiếp tục được quan sát, rồi sau đó mới triển khai trên tất cả các máy trên toàn cầu. Các giai đoạn đầu của một lần triển khai thường được gọi là "canaries (chim canari)"—một sự ám chỉ đến những con chim canari mà các thợ mỏ đưa vào hầm than để phát hiện khí độc. Các server canary của chúng tôi giúp phát hiện những tác động nguy hiểm từ hành vi của phần mềm mới dưới traffic người dùng thực.

Canary testing (kiểm thử canari) là một khái niệm được tích hợp vào nhiều công cụ nội bộ của Google, dùng để thực hiện các thay đổi tự động cũng như cho phép các hệ thống thay đổi file cấu hình. Các công cụ quản lý việc cài đặt phần mềm mới thường theo dõi server vừa khởi động trong một khoảng thời gian, nhằm đảm bảo server không crash (sập) hay có hành vi sai lệch. Nếu thay đổi không vượt qua giai đoạn xác thực, hệ thống sẽ tự động rollback (hoàn tác).

Khái niệm triển khai từ từ thậm chí áp dụng cho cả phần mềm không chạy trên các server của Google. Các phiên bản mới của một app Android có thể được triển khai theo cách này, trong đó bản cập nhật chỉ được cung cấp cho một tập con các cài đặt để nâng cấp. Tỷ lệ phần trăm các instance được nâng cấp sẽ tăng dần theo thời gian cho đến khi đạt 100%. Loại triển khai này đặc biệt hữu ích nếu phiên bản mới làm tăng traffic đến các server backend trong các datacenter của Google. Bằng cách này, chúng tôi có thể quan sát tác động lên các server của mình khi dần dần triển khai phiên bản mới và phát hiện các vấn đề sớm.

Hệ thống mời (invite system) là một dạng triển khai từ từ khác. Thay vì mở đăng ký miễn phí cho dịch vụ mới, hệ thống này thường chỉ cho phép một số lượng người dùng hạn chế đăng ký mỗi ngày. Việc giới hạn tỷ lệ đăng ký thường đi kèm với cơ chế mời, trong đó mỗi người dùng có thể gửi một số lượng lời mời nhất định cho bạn bè.

### Các Framework cờ tính năng (Feature Flag Frameworks)

Google thường bổ sung các bài kiểm thử trước khi ra mắt, kết hợp với các chiến lược giảm thiểu rủi ro sự cố. Một cơ chế để triển khai thay đổi từ từ, cho phép quan sát hành vi tổng thể của hệ thống dưới các workload thực, có thể tự trang trải chi phí đầu tư kỹ thuật nhờ cải thiện độ tin cậy, velocity kỹ thuật và thời gian ra thị trường. Những cơ chế này đã chứng minh là đặc biệt hữu ích trong các trường hợp mà việc thiết lập môi trường kiểm thử thực tế là không khả thi, hoặc cho các lần ra mắt đặc biệt phức tạp mà tác động của chúng khó dự đoán.

Hơn nữa, không phải mọi thay đổi đều giống nhau. Đôi khi bạn chỉ muốn kiểm tra xem một điều chỉnh nhỏ trên giao diện người dùng có cải thiện trải nghiệm của người dùng hay không. Những thay đổi nhỏ như vậy không nên kéo theo hàng nghìn dòng mã hay một quy trình ra mắt nặng nề. Bạn có thể muốn kiểm thử hàng trăm thay đổi như vậy song song.

Cuối cùng, đôi khi bạn muốn kiểm tra xem một nhóm người dùng nhỏ có hứng thú với nguyên mẫu sơ khai của một tính năng mới khó triển khai hay không. Bạn không muốn bỏ ra hàng tháng công sức kỹ thuật để hoàn thiện tính năng đó cho hàng triệu người dùng, chỉ để rồi phát hiện ra rằng nó thất bại.

Để đáp ứng các kịch bản được đề cập ở trên, một số sản phẩm của Google đã tạo ra các framework cờ tính năng (feature flag). Một số framework trong số đó được thiết kế để triển khai các tính năng mới từ từ từ 0% đến 100% người dùng. Bất cứ khi nào một sản phẩm giới thiệu một framework như vậy, chính framework đó được củng cố nhiều nhất có thể sao cho phần lớn các ứng dụng của nó không cần bất kỳ sự tham gia nào của LCE. Những framework như vậy thường đáp ứng các yêu cầu sau:

-   Triển khai nhiều thay đổi song song, mỗi cái đến một vài server, người dùng, thực thể, hoặc datacenter
-   Tăng dần đến một nhóm người dùng lớn hơn nhưng có giới hạn, thường từ 1 đến 10 phần trăm
-   Định tuyến traffic qua các server khác nhau tùy theo người dùng, phiên (session), object, và/hoặc vị trí
-   Xử lý tự động sự hỏng hóc của các đường dẫn mã mới bằng thiết kế, mà không ảnh hưởng đến người dùng
-   Hoàn tác độc lập mỗi thay đổi như vậy ngay lập tức trong trường hợp có bug nghiêm trọng hoặc tác dụng phụ
-   Đo lường mức độ mà mỗi thay đổi cải thiện trải nghiệm người dùng

Các framework cờ tính năng của Google rơi vào hai lớp tổng quát:

-   Những cái chủ yếu tạo thuận lợi cho các cải tiến giao diện người dùng
-   Những cái hỗ trợ các thay đổi logic phía server và kinh doanh tùy ý

Với các thay đổi giao diện người dùng trong dịch vụ không trạng thái (stateless), framework cờ tính năng đơn giản nhất là một bộ viết lại (rewriter) payload HTTP chạy trên server ứng dụng frontend, bị giới hạn ở một tập con các cookie hoặc một thuộc tính request/response HTTP tương tự khác. Một cơ chế cấu hình có thể xác định định danh (identifier) gắn với các đường dẫn mã mới, phạm vi thay đổi (ví dụ, cookie hash mod range), cùng các whitelist (danh sách cho phép) và blacklist (danh sách cấm).

Các dịch vụ có trạng thái (stateful) thường giới hạn các cờ tính năng cho một tập con các định danh người dùng đăng nhập duy nhất hoặc các thực thể sản phẩm thực tế được truy cập, chẳng hạn như ID của tài liệu, bảng tính hoặc object lưu trữ. Thay vì viết lại các payload HTTP, các dịch vụ này có xu hướng proxy (trình trung gian) hoặc định tuyến lại các request đến các server khác nhau tùy theo thay đổi, nhờ đó cải thiện khả năng kiểm thử logic kinh doanh và hỗ trợ các tính năng mới phức tạp hơn.

### Đối Phó Với Hành Vi Client Lạm Dụng (Dealing with Abusive Client Behavior)

Ví dụ đơn giản nhất cho hành vi client lạm dụng là việc đánh giá sai tỷ lệ cập nhật. Một client mới đồng bộ hóa mỗi 60 giây thay vì mỗi 600 giây sẽ gây ra tải lên dịch vụ gấp 10 lần. Hành vi retry có một số bẫy ảnh hưởng đến cả request do người dùng khởi tạo lẫn request do client khởi tạo. Xét một dịch vụ đang quá tải và vì thế làm thất bại một số request: nếu các client thử lại những request thất bại này, chúng sẽ thêm tải vào dịch vụ vốn đã quá tải, dẫn đến nhiều lần thử lại hơn và thậm chí nhiều request hơn. Thay vào đó, các client cần giảm tần suất thử lại, thường bằng cách thêm độ trễ tăng theo cấp số mũ giữa các lần thử lại, đồng thời cân nhắc kỹ lưỡng những loại lỗi nào xứng đáng để thử lại. Chẳng hạn, một lỗi mạng thường xứng đáng để thử lại, nhưng một lỗi HTTP 4xx (cho thấy lỗi ở phía client) thì thường không.

Sự đồng bộ hóa có chủ ý hoặc vô tình các request tự động trong một "hiệu ứng bầy đàn" (thundering herd, giống như những thứ được mô tả trong các chương [Định Kỳ Phân Tán với Cron](https://sre.google/sre-book/distributed-periodic-scheduling/) và [Pipeline Xử Lý Dữ Liệu](https://sre.google/sre-book/data-processing-pipelines/)) là một ví dụ phổ biến khác của hành vi client lạm dụng. Một nhà phát triển app điện thoại có thể quyết định rằng 2 giờ sáng là một thời điểm tốt để tải xuống các bản cập nhật, vì người dùng có khả năng đang ngủ và sẽ không bị phiền toái bởi việc tải xuống. Tuy nhiên, một thiết kế như vậy dẫn đến một loạt các request đến server tải xuống lúc 2 giờ sáng mỗi đêm, và gần như không có request vào bất kỳ thời điểm nào khác. Thay vào đó, mọi client nên chọn thời điểm cho loại request này một cách ngẫu nhiên.

Tính ngẫu nhiên cũng cần được đưa vào các quy trình định kỳ khác. Quay lại ví dụ về các lần thử lại đã đề cập trước đó: một client gửi request, khi gặp lỗi sẽ thử lại sau 1 giây, rồi 2 giây, 4 giây, và cứ thế tiếp tục. Nếu không có tính ngẫu nhiên, một đợt tăng request ngắn khiến tỷ lệ lỗi tăng cao có thể tự lặp lại chính nó do các lần thử lại diễn ra đúng sau 1 giây, 2 giây, 4 giây. Để phá vỡ sự đồng bộ của các sự kiện này, mỗi độ trễ cần được jitter (tức là điều chỉnh bằng một lượng ngẫu nhiên).

Khả năng server kiểm soát hành vi của client đã chứng minh là một công cụ quan trọng. Đối với một app trên thiết bị, sự kiểm soát này có thể bao gồm việc hướng dẫn client định kỳ check in với server và tải xuống file cấu hình. File đó có thể bật hoặc tắt một số tính năng, hoặc đặt các tham số như tần suất đồng bộ hóa hay tần suất thử lại của client.

Cấu hình client thậm chí có thể kích hoạt một tính năng hoàn toàn mới dành cho người dùng. Bằng cách đưa mã hỗ trợ tính năng mới vào ứng dụng client trước khi bật tính năng đó, chúng tôi giảm đáng kể rủi ro của một lần ra mắt. Việc phát hành phiên bản mới trở nên dễ dàng hơn nhiều nếu không cần duy trì các đường release song song cho phiên bản có tính năng mới và phiên bản không có. Điều này đặc biệt đúng khi chúng tôi không chỉ xử lý một tính năng mới đơn lẻ, mà là một tập các tính năng độc lập có thể phát hành theo các lịch trình khác nhau; khi đó, số phiên bản cần duy trì sẽ tăng theo cấp số nhân (combinatorial explosion).

Việc có loại tính năng ngủ đông (dormant) này cũng giúp việc hủy bỏ các lần ra mắt trở nên dễ dàng hơn khi phát hiện tác động bất lợi trong quá trình triển khai. Trong những trường hợp như vậy, chúng tôi chỉ cần chuyển tính năng sang tắt, lặp lại (iterate), và phát hành một phiên bản cập nhật của app. Nếu không có loại cấu hình client này, chúng tôi sẽ phải cung cấp một phiên bản app mới không có tính năng đó, và cập nhật app trên điện thoại của tất cả người dùng.

### Hành Vi Quá Tải và Kiểm Thử Tải (Overload Behavior and Load Tests)

Các tình trạng quá tải là một chế độ lỗi đặc biệt phức tạp, và do đó xứng đáng được chú ý thêm. Sự thành công mất kiểm soát thường là nguyên nhân đáng được chào đón nhất của quá tải khi một dịch vụ mới ra mắt, nhưng có vô số các nguyên nhân khác, bao gồm các sự cố cân bằng tải, các sự cố máy, hành vi client đồng bộ hóa, và các cuộc tấn công bên ngoài.

Một mô hình đơn giản giả định rằng mức sử dụng CPU trên một máy cung cấp dịch vụ tăng tuyến tính theo tải (ví dụ, số request hoặc lượng dữ liệu được xử lý), và khi CPU khả dụng cạn kiệt, quá trình xử lý đơn giản là trở nên chậm hơn. Tuy nhiên, trong thực tế, các dịch vụ hiếm khi hoạt động theo cách lý tưởng này. Nhiều dịch vụ thậm chí còn chậm hơn khi chưa chịu tải, chủ yếu do ảnh hưởng của các loại cache khác nhau như CPU caches, JIT caches và các cache dữ liệu đặc thù của dịch vụ. Khi tải tăng, thường có một khoảng mà mức sử dụng CPU và tải trên dịch vụ tương ứng tuyến tính, đồng thời thời gian phản hồi giữ ở mức tương đối ổn định.

Ở một số điểm, nhiều dịch vụ chạm đến ngưỡng phi tuyến tính khi tiến gần quá tải. Trong các trường hợp nhẹ nhất, thời gian phản hồi đơn giản bắt đầu tăng, khiến trải nghiệm người dùng giảm sút nhưng chưa chắc đã gây sự cố (dù một phụ thuộc chậm có thể làm phát sinh lỗi mà người dùng nhìn thấy ở các tầng trên của stack, do vượt quá deadline RPC). Trong các trường hợp nặng nhất, dịch vụ bị khóa hoàn toàn (locks up) khi đối mặt với quá tải.

Để dẫn ra một ví dụ cụ thể về hành vi quá tải: một dịch vụ đã ghi log (nhật ký) thông tin gỡ rối để phản ứng với các lỗi backend. Hóa ra là việc ghi log thông tin gỡ rối đắt đỏ hơn việc xử lý phản ứng backend trong một trường hợp bình thường. Do đó, khi dịch vụ trở nên quá tải và timeout các phản ứng backend bên trong stack RPC của chính nó, dịch vụ dành thêm thời gian CPU để ghi log những phản ứng này, rồi timeout nhiều request hơn, cứ thế cho đến khi dịch vụ dừng hẳn. Trong các dịch vụ chạy trên Java Virtual Machine (JVM), một hiệu ứng tương tự của việc dừng hẳn đôi khi được gọi là "GC (garbage collection - thu gom rác) thrashing (giằng co)." Trong kịch bản này, việc quản lý bộ nhớ nội bộ của máy ảo chạy trong các chu kỳ ngày càng gần hơn, cố gắng giải phóng bộ nhớ cho đến khi phần lớn thời gian CPU bị tiêu thụ bởi việc quản lý bộ nhớ.

Thật không may, rất khó để dự đoán từ các nguyên lý cơ bản (first principles) một dịch vụ sẽ phản ứng thế nào với quá tải. Vì vậy, load tests (kiểm thử tải) là công cụ vô giá, vừa vì lý do độ tin cậy vừa vì lập kế hoạch năng lực, và kiểm thử tải được yêu cầu cho phần lớn các lần ra mắt.

## Sự Phát Triển của LCE (Development of LCE)

Trong những năm đầu hình thành của Google, kích thước của nhóm kỹ thuật tăng gấp đôi mỗi năm trong nhiều năm liền, phân mảnh phòng ban kỹ thuật thành nhiều nhóm nhỏ làm việc trên nhiều sản phẩm và tính năng mới thử nghiệm. Trong một bầu không khí như vậy, các kỹ sư non trẻ có nguy cơ lặp lại những sai lầm của những người đi trước, đặc biệt là khi nói đến việc ra mắt thành công các tính năng và sản phẩm mới.

Để tránh lặp lại những sai lầm tương tự, một nhóm nhỏ kỹ sư dày dạn kinh nghiệm đã tình nguyện đảm nhận vai trò tư vấn, được gọi là các “Launch Engineer (Kỹ sư Ra mắt)”. Họ ghi lại những bài học từ các lần ra mắt trước đó và xây dựng các checklist cho việc ra mắt sản phẩm mới, bao phủ các chủ đề như:

-   Khi nào nên tham vấn phòng ban pháp lý
-   Cách chọn các tên miền
-   Cách đăng ký các tên miền mới mà không cấu hình sai DNS
-   Các bẫy phổ biến trong thiết kế kỹ thuật và triển khai production

"Launch Reviews (Rà soát Ra mắt)" — tên gọi dành cho các buổi tư vấn của Launch Engineers — đã trở thành một thực hành phổ biến, diễn ra từ vài ngày đến vài tuần trước khi ra mắt nhiều sản phẩm mới.

Trong vòng hai năm, các yêu cầu triển khai sản phẩm trong checklist ra mắt ngày càng dài và phức tạp. Kết hợp với sự phức tạp ngày càng tăng của môi trường triển khai của Google, việc cập nhật về cách thực hiện các thay đổi một cách an toàn trở nên ngày càng khó khăn cho các kỹ sư sản phẩm. Đồng thời, tổ chức SRE đang phát triển nhanh, và các SRE thiếu kinh nghiệm đôi khi quá thận trọng và e dè trước sự thay đổi. Google đối mặt với rủi ro rằng các cuộc đàm phán phát sinh giữa hai bên này sẽ làm giảm velocity của các lần ra mắt sản phẩm/tính năng.

Về mặt kỹ thuật, để hạn chế tình trạng này, SRE đã thành lập một nhóm LCE toàn thời gian nhỏ vào năm 2004. Nhóm này phụ trách đẩy nhanh tiến độ ra mắt các sản phẩm và tính năng mới, đồng thời vận dụng chuyên môn SRE để đảm bảo Google phát hành các sản phẩm đáng tin cậy, có khả năng hoạt động cao và latency thấp.

LCE chịu trách nhiệm đảm bảo các lần ra mắt diễn ra nhanh chóng mà không làm dịch vụ sụp đổ, đồng thời nếu một lần ra mắt thất bại, nó không kéo theo các sản phẩm khác. LCE cũng phải đảm bảo các bên liên quan được thông báo về bản chất và khả năng của những sự cố như vậy, bất cứ khi nào có những góc cạnh bị cắt để tăng tốc thời gian ra thị trường. Các buổi tư vấn của họ được chính thức hóa thành các Production Review (Rà soát Production).

### Sự Tiến Hóa của Checklist LCE (Evolution of the LCE Checklist)

Khi môi trường của Google trở nên phức tạp hơn, cả checklist Launch Coordination Engineering (xem [Launch Coordination Checklist](https://sre.google/sre-book/launch-checklist)) và lượng các lần ra mắt cũng tăng lên. Trong 3,5 năm, một LCE đã chạy 350 lần ra mắt qua LCE Checklist. Khi nhóm trung bình năm kỹ sư trong khoảng thời gian này, điều này tương đương với một năng suất ra mắt của Google là hơn 1.500 lần ra mắt trong 3,5 năm!

Mỗi câu hỏi trong LCE Checklist tuy đơn giản, nhưng sự phức tạp nằm ở lý do đặt ra câu hỏi đó và hệ quả của câu trả lời. Để nắm bắt trọn vẹn mức độ phức tạp này, một nhân viên LCE mới cần khoảng sáu tháng đào tạo.

Khi lượng các lần ra mắt tăng lên, theo kịp sự tăng gấp đôi hàng năm của nhóm kỹ thuật của Google, các LCE tìm kiếm các cách để tinh giản các cuộc rà soát của họ. Các LCE xác định các danh mục các lần ra mắt rủi ro thấp rất ít khả năng phải đối mặt hoặc gây ra các sự cố. Ví dụ, một lần ra mắt tính năng không liên quan đến các file thực thi server mới và một sự tăng traffic dưới 10% sẽ được coi là rủi ro thấp. Những lần ra mắt như vậy phải đối mặt với một checklist gần như tầm thường, trong khi các lần ra mắt rủi ro cao hơn trải qua toàn bộ các kiểm tra và cân bằng. Đến năm 2008, 30% các cuộc rà soát được coi là rủi ro thấp.

Cùng lúc đó, quy mô của Google ngày càng mở rộng, giúp xóa bỏ những ràng buộc từng kìm hãm các lần ra mắt. Chẳng hạn, thương vụ mua lại YouTube đã buộc Google phải nâng cấp hạ tầng mạng và khai thác băng thông hiệu quả hơn. Nhờ vậy, nhiều sản phẩm nhỏ hơn có thể “lách” vào các khoảng trống còn lại, qua đó né tránh các quy trình lập kế hoạch năng lực mạng và cung cấp vốn phức tạp, đồng thời đẩy nhanh tốc độ ra mắt. Google cũng bắt đầu xây dựng các datacenter quy mô lớn, cho phép đặt nhiều dịch vụ phụ thuộc dưới một mái nhà. Xu hướng này đơn giản hóa việc ra mắt các sản phẩm mới cần huy động lượng lớn năng lực từ nhiều dịch vụ hiện có mà chúng đang phụ thuộc.

### Những Vấn Đề LCE Không Giải Quyết Được (Problems LCE Didn't Solve)

Dù các LCE đã cố gắng giảm thiểu tính quan liêu trong các cuộc rà soát, nỗ lực đó vẫn không đủ. Đến năm 2009, việc ra mắt một dịch vụ mới nhỏ tại Google đã trở thành một huyền thoại. Các dịch vụ phát triển ở quy mô lớn hơn phải đối mặt với một bộ vấn đề riêng mà Launch Coordination không thể giải quyết.

#### Những thay đổi khả năng mở rộng (Scalability changes)

Khi sản phẩm thành công vượt xa mọi ước tính ban đầu và lượng sử dụng tăng hơn hai bậc độ lớn, việc theo kịp tải đòi hỏi nhiều thay đổi thiết kế. Những thay đổi khả năng mở rộng như vậy, kết hợp với việc thêm tính năng liên tục, thường khiến sản phẩm trở nên phức tạp hơn, mong manh hơn và khó vận hành hơn. Đến một số điểm, kiến trúc sản phẩm ban đầu trở nên không thể quản lý được và sản phẩm cần được kiến trúc lại hoàn toàn. Việc kiến trúc lại sản phẩm và sau đó di chuyển tất cả người dùng từ kiến trúc cũ sang kiến trúc mới đòi hỏi một đầu tư lớn về thời gian và nguồn lực từ cả các nhà phát triển và SRE, làm chậm tỷ lệ phát triển tính năng mới trong giai đoạn đó.

#### Gánh nặng vận hành tăng lên (Growing operational load)

Sau khi ra mắt, operational load (gánh nặng vận hành) — tức lượng công việc kỹ thuật thủ công và lặp lại cần thiết để duy trì hệ thống — có xu hướng tăng theo thời gian nếu không có biện pháp kiểm soát. Sự ồn ào từ các thông báo tự động, độ phức tạp của quy trình triển khai và overhead (công việc phụ) trong bảo trì thủ công đều tăng dần, chiếm ngày càng nhiều bandwidth của chủ sở hữu dịch vụ, khiến nhóm còn ít thời gian hơn cho việc phát triển tính năng. SRE đặt ra mục tiêu nội bộ là giữ công việc vận hành dưới mức tối đa 50%; xem [Loại Bỏ Toil (Công Việc Lặp Đi Lặp Lại)](https://sre.google/sre-book/eliminating-toil/). Để duy trì dưới ngưỡng này, cần theo dõi liên tục các nguồn gây ra công việc vận hành và có nỗ lực có mục tiêu nhằm loại bỏ chúng.

#### Sự biến động hạ tầng (Infrastructure churn)

Nếu hạ tầng nền tảng (chẳng hạn như các hệ thống cho quản lý cụm, lưu trữ, giám sát, cân bằng tải, và chuyển dữ liệu) đang thay đổi do các nhóm hạ tầng phát triển tích cực, các chủ sở hữu dịch vụ chạy trên hạ tầng phải đầu tư một lượng lớn công việc chỉ để theo kịp những thay đổi đó. Khi các tính năng hạ tầng mà các dịch vụ phụ thuộc bị khai tử và thay thế bằng các tính năng mới, các chủ sở hữu dịch vụ phải liên tục sửa đổi cấu hình và xây dựng lại các file thực thi, dẫn đến tình trạng "chạy nhanh chỉ để giữ nguyên vị trí." Giải pháp cho kịch bản này là áp dụng một loại chính sách giảm churn nào đó, cấm các kỹ sư hạ tầng phát hành các tính năng không tương thích ngược cho đến khi họ tự động hóa việc di chuyển các client của mình sang tính năng mới. Việc tạo các công cụ di chuyển tự động đi kèm với các tính năng mới giúp giảm thiểu khối lượng công việc mà các chủ sở hữu dịch vụ phải bỏ ra để theo kịp sự biến động của hạ tầng.

Để giải quyết những vấn đề này, cần có các nỗ lực trên toàn công ty vượt xa phạm vi của LCE: kết hợp các API và framework nền tảng tốt hơn (xem [Mô Hình Tham Gia SRE Đang Tiến Hóa](https://sre.google/sre-book/evolving-sre-engagement-model/)), tự động hóa build và kiểm thử liên tục, cùng với việc chuẩn hóa và tự động hóa tốt hơn cho các dịch vụ production của Google.

## Kết Luận (Conclusion)

Các công ty tăng trưởng nhanh, với tỷ lệ thay đổi cao về sản phẩm và dịch vụ, có thể hưởng lợi từ một vai trò tương đương Launch Coordination Engineering. Một nhóm như vậy đặc biệt có giá trị nếu công ty lên kế hoạch tăng gấp đôi số nhà phát triển sản phẩm mỗi một hoặc hai năm, nếu phải mở rộng scale các dịch vụ của mình đến hàng trăm triệu người dùng, và nếu độ tin cậy bất chấp một tỷ lệ thay đổi cao là quan trọng đối với người dùng của nó.

Nhóm LCE là cách Google giải quyết bài toán cân bằng giữa an toàn và tốc độ thay đổi. Trong suốt 10 năm, với vai trò LCE độc đáo, chúng tôi đã tích lũy được nhiều kinh nghiệm trong chính những hoàn cảnh như vậy. Chúng tôi hy vọng cách tiếp cận này sẽ là nguồn cảm hứng cho những ai đang đối mặt với thách thức tương tự tại tổ chức của mình.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
