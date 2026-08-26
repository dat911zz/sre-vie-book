> **Nguyên bản:** [Chapter 27 - Reliable Product Launches at Scale](https://sre.google/sre-book/reliable-product-launches/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Đội biên tập Softdreams RnD)

---

## Chương 27. Ra Mắt Sản Phẩm Đáng Tin Cậy Ở Quy Mô Lớn (Reliable Product Launches at Scale)

Tác giả: Rhandeev Singh và Sebastian Kirsch cùng Vivek Rau  
Biên tập: Betsy Beyer

Các công ty Internet như Google có thể ra mắt (launch) các sản phẩm và tính năng mới trong các vòng lặp nhanh hơn nhiều so với các công ty truyền thống. Vai trò của Site Reliability trong quá trình này là cho phép một tốc độ thay đổi nhanh mà không làm tổn hại đến sự ổn định của dịch vụ. Chúng tôi đã tạo ra một nhóm chuyên trách gồm các "Kỹ sư Điều Phối Ra Mắt (Launch Coordination Engineers)" để tư vấn cho các nhóm kỹ thuật về các khía cạnh kỹ thuật của một lần ra mắt thành công.

Nhóm này cũng biên soạn một "checklist ra mắt" gồm các câu hỏi phổ biến cần hỏi về một lần ra mắt, và các công thức (recipes) để giải quyết các vấn đề phổ biến. Checklist đã chứng minh là một công cụ hữu ích để đảm bảo các lần ra mắt đáng tin cậy có thể tái lập.

Hãy xem xét một dịch vụ Google thông thường—ví dụ, Keyhole, thứ cung cấp ảnh vệ tinh cho Google Maps và Google Earth. Vào một ngày bình thường, Keyhole phục vụ đến vài nghìn ảnh vệ tinh mỗi giây. Nhưng vào đêm Giao Thừa 2011, nó nhận được lượng traffic cao gấp 25 lần đỉnh thông thường—lên đến một triệu request mỗi giây. Điều gì đã gây ra sự tăng vọt traffic khổng lồ này?

## Ông già Noel đang đến. (Santa was coming.)

Vài năm trước, Google đã hợp tác với NORAD (North American Aerospace Defense Command - Bộ Tư Lệnh Phòng Thủ Vũ Trụ và Không Gian Bắc Mỹ) để đăng tải một website mang chủ đề Giáng Sinh theo dõi hành trình của Ông già Noel quanh thế giới, cho phép người dùng xem ông phân phát quà theo thời gian thực. Một phần của trải nghiệm là một "bay qua ảo (virtual fly-over)", sử dụng ảnh vệ tinh để theo dõi hành trình của Ông già Noel trên một thế giới mô phỏng.

Trong khi một dự án như NORAD Tracks Santa có thể có vẻ lãng mạn, nó có tất cả các đặc điểm xác định một lần ra mắt khó khăn và rủi ro: một hạn chát (hard deadline) (Google không thể xin Ông già Noel đến muộn một tuần nếu website chưa sẵn sàng), rất nhiều sự chú ý của công chúng, một khán giả hàng triệu người, và một sự tăng traffic rất dốc (mọi người sẽ cùng xem website vào đêm Giao Thừa). Đừng bao giờ đánh giá thấp sức mạnh của hàng triệu trẻ nhỏ nóng lòng chờ quà—dự án này có một khả năng rất thực sự để quật ngã các server của Google.

Nhóm Site Reliability Engineering của Google đã làm việc chăm chỉ để chuẩn bị hạ tầng của chúng tôi cho lần ra mắt này, đảm bảo rằng Ông già Noel có thể phân phát tất cả quà của ông đúng giờ dưới ánh mắt chăm chú của một khán giả đang mong chờ. Điều cuối cùng chúng tôi muốn là làm trẻ con khóc vì chúng không thể xem Ông già Noel phân phát quà. Thật vậy, chúng tôi đã đặt tên cho các kill switch khác nhau được xây dựng vào trong trải nghiệm để bảo vệ các dịch vụ của chúng tôi là các "công tắc khiến trẻ con khóc." Dự đoán nhiều cách khác nhau mà lần ra mắt này có thể đi sai và điều phối giữa các nhóm kỹ thuật khác nhau tham gia vào lần ra mắt là việc của một nhóm đặc biệt trong Site Reliability Engineering: các Kỹ sư Điều Phối Ra Mắt (LCE).

Ra mắt một sản phẩm hoặc tính năng mới là khoảnh khắc quyết định cho mọi công ty—điểm mà tại đó hàng tháng hoặc hàng năm nỗ lực được trình bày ra trước thế giới. Các công ty truyền thống ra mắt các sản phẩm mới với một tỷ lệ khá thấp. Chu kỳ ra mắt ở các công ty Internet khác biệt rõ rệt. Các lần ra mắt và lặp lại nhanh dễ dàng hơn nhiều vì các tính năng mới có thể được triển khai ở phía server, thay vì đòi hỏi việc triển khai phần mềm trên các workstation cá nhân của từng khách hàng.

Google định nghĩa một lần ra mắt là bất kỳ mã mới nào đưa đến một sự thay đổi nhìn thấy được từ bên ngoài của một ứng dụng. Tùy thuộc vào các đặc điểm của một lần ra mắt—sự tổ hợp các thuộc tính, thời điểm, số bước liên quan, và độ phức tạp—quy trình ra mắt có thể thay đổi rất nhiều. Theo định nghĩa này, Google đôi khi thực hiện lên đến 70 lần ra mắt mỗi tuần.

Tốc độ thay đổi nhanh này vừa cung cấp lý lẽ vừa tạo cơ hội cho việc tạo ra một quy trình ra mắt được tinh giản. Một công ty chỉ ra mắt một sản phẩm mỗi ba năm không cần một quy trình ra mắt chi tiết. Vào thời điểm một lần ra mắt mới diễn ra, hầu hết các thành phần của quy trình ra mắt đã phát triển trước đó sẽ lỗi thời. Các công ty truyền thống cũng không có cơ hội để thiết kế một quy trình ra mắt chi tiết, vì họ không tích lũy đủ kinh nghiệm thực hiện các lần ra mắt để tạo ra một quy trình vững chắc và trưởng thành.

## Kỹ Thuật Điều Phối Ra Mắt (Launch Coordination Engineering)

Các kỹ sư phần mềm giỏi có rất nhiều chuyên môn trong việc viết mã và thiết kế, và hiểu rất tốt công nghệ của chính sản phẩm của họ. Tuy nhiên, chính những kỹ sư đó có thể không quen thuộc với những thách thức và bẫy của việc ra mắt một sản phẩm đến hàng triệu người dùng trong khi đồng thời giảm thiểu các sự cố và tối đa hóa hiệu suất.

Google tiếp cận những thách thức vốn có của việc ra mắt bằng cách tạo ra một nhóm tư vấn chuyên trách trong SRE chịu trách nhiệm về mặt kỹ thuật của việc ra mắt một sản phẩm hoặc tính năng mới. Được bố trí bởi các kỹ sư phần mềm và kỹ sư hệ thống—một số có kinh nghiệm trong các nhóm SRE khác—nhóm này chuyên hướng dẫn các nhà phát triển xây dựng các sản phẩm đáng tin cậy và nhanh đáp ứng các tiêu chuẩn của Google về sự vững chắc, khả năng mở rộng, và độ tin cậy. Nhóm tư vấn này, Launch Coordination Engineering (LCE), tạo thuận lợi cho một quy trình ra mắt suôn sẻ theo một vài cách:

-   Thăm dò (audit) các sản phẩm và dịch vụ về sự tuân thủ [các tiêu chuẩn độ tin cậy của Google](https://sre.google/resources/practices-and-processes/product-focused-reliability-for-sre/) và các thực hành tốt nhất, đồng thời cung cấp các hành động cụ thể để cải thiện độ tin cậy
-   Đóng vai trò cầu nối giữa các nhóm khác nhau tham gia vào một lần ra mắt
-   Thúc đẩy các khía cạnh kỹ thuật của một lần ra mắt bằng cách đảm bảo rằng các tác vụ duy trì động lực
-   Đóng vai trò những người gác cổng và ký duyệt các lần ra mắt được xác định là "an toàn"
-   Giáo dục các nhà phát triển về các thực hành tốt nhất và về [cách tích hợp với các dịch vụ của Google](https://sre.google/sre-book/service-best-practices/), trang bị cho họ [tài liệu và tài nguyên đào tạo nội bộ](https://sre.google/resources/) để tăng tốc việc học của họ

Các thành viên của nhóm LCE thăm dò các dịch vụ vào các thời điểm khác nhau trong vòng đời của dịch vụ. Phần lớn các cuộc thăm dò được tiến hành trước khi một sản phẩm hoặc dịch vụ mới ra mắt. Nếu một nhóm phát triển sản phẩm thực hiện một lần ra mắt mà không có hỗ trợ SRE, LCE cung cấp kiến thức chuyên ngành phù hợp để đảm bảo một lần ra mắt suôn sẻ. Nhưng ngay cả các sản phẩm đã có hỗ trợ SRE mạnh thường xuyên tương tác với nhóm LCE trong các lần ra mắt quan trọng. Những thách thức mà các nhóm phải đối mặt khi ra mắt một sản phẩm mới khác biệt đáng kể so với hoạt động hàng ngày của một dịch vụ đáng tin cậy (một nhiệm vụ mà các nhóm SRE đã giỏi), và nhóm LCE có thể rút kinh nghiệm từ hàng trăm lần ra mắt. Nhóm LCE cũng tạo thuận lợi cho các cuộc thăm dò dịch vụ khi các dịch vụ mới bắt đầu tương tác với SRE.

### Vai Trò của Kỹ Sư Điều Phối Ra Mắt (The Role of the Launch Coordination Engineer)

Nhóm Launch Coordination Engineering của chúng tôi được cấu thành từ các Kỹ sư Điều Phối Ra Mắt (LCE), những người hoặc được tuyển dụng trực tiếp vào vai trò này, hoặc là các SRE có kinh nghiệm thực chiến vận hành các dịch vụ của Google. Các LCE được giữ theo cùng các yêu cầu kỹ thuật như bất kỳ SRE nào khác, và cũng được kỳ vọng có kỹ năng giao tiếp và lãnh đạo mạnh mẽ—một LCE đưa các bên khác nhau lại với nhau để làm việc hướng tới một mục tiêu chung, dàn xếp các cuộc xung đột thỉnh thoảng, và hướng dẫn, huấn luyện, và giáo dục các kỹ sư đồng nghiệp.

Một nhóm chuyên trách điều phối các lần ra mắt mang lại những lợi thế sau:

#### Breadth of experience (Breadth of experience - chiều rộng kinh nghiệm)

Là một nhóm thực sự liên sản phẩm (cross-product), các thành viên hoạt động trên gần như tất cả các lĩnh vực sản phẩm của Google. Kiến thức liên sản phẩm rộng rãi và các mối quan hệ với nhiều nhóm trên khắp công ty làm cho các LCE trở thành những phương tiện xuất sắc để chuyển giao kiến thức.

Cross-functional perspective (Cross-functional perspective - góc nhìn liên chức năng)

Các LCE có một góc nhìn toàn diện về lần ra mắt, điều cho phép họ điều phối giữa các nhóm khác nhau trong SRE, phát triển, và quản lý sản phẩm. Cách tiếp cận toàn diện này đặc biệt quan trọng cho các lần ra mắt phức tạp có thể trải dài hơn nửa tá nhóm trên nhiều múi giờ khác nhau.

#### Objectivity (Objectivity - tính khách quan)

Với tư cách là một cố vấn không thiên vị, một LCE đóng vai trò cân bằng và dàn xếp giữa các bên liên quan bao gồm SRE, nhà phát triển sản phẩm, quản lý sản phẩm, và tiếp thị.

Vì Launch Coordination Engineer là một vai trò SRE, các LCE được tạo động lực để ưu tiên độ tin cậy trên các mối quan tâm khác. Một công ty không chia sẻ các mục tiêu độ tin cậy của Google, nhưng chia sẻ tốc độ thay đổi nhanh của Google, có thể chọn một cấu trúc động lực khác.

## Thiết Lập Một Quy Trình Ra Mắt (Setting Up a Launch Process)

Google đã mài giũa quy trình ra mắt của mình trong một khoảng thời gian hơn 10 năm. Theo thời gian, chúng tôi đã xác định một số tiêu chí xác định một quy trình ra mắt tốt:

#### Lightweight (Nhẹ nhàng)

Dễ chịu với các nhà phát triển

#### Robust (Vững chắc)

Bắt được các lỗi hiển nhiên

#### Thorough (Chu đáo)

Đặt giải quyết các chi tiết quan trọng một cách nhất quán và có thể tái lập

#### Scalable (Có thể mở rộng)

Ứng phó được với cả một số lượng lớn các lần ra mắt đơn giản và ít lần ra mắt phức tạp hơn

#### Adaptable (Có thể thích ứng)

Hoạt động tốt cho các loại ra mắt phổ biến (ví dụ, thêm một ngôn ngữ UI mới vào một sản phẩm) và các loại ra mắt mới (ví dụ, lần ra mắt ban đầu của trình duyệt Chrome hoặc Google Fiber)

Như bạn có thể thấy, một số yêu cầu này xung đột rõ ràng với nhau. Ví dụ, khó mà thiết kế một quy trình vừa nhẹ nhàng vừa chu đáo. Cân bằng những tiêu chí này đối với nhau đòi hỏi một công việc liên tục. Google đã thành công áp dụng một vài chiến thuật để giúp chúng tôi đạt được những tiêu chí này:

#### Simplicity (Sự đơn giản)

Làm đúng những điều cơ bản. Đừng lên kế hoạch cho mọi sự cố.

#### A high touch approach (Một cách tiếp cận chăm chút)

Các kỹ sư giàu kinh nghiệm tùy biến quy trình cho phù hợp với từng lần ra mắt.

#### Fast common paths (Các đường chung nhanh)

Xác định các lớp các lần ra mắt luôn tuân theo một pattern (hình mẫu) chung (chẳng hạn như ra mắt một sản phẩm ở một quốc gia mới), và cung cấp một quy trình ra mắt được tinh giản cho lớp này.

Kinh nghiệm đã chứng minh rằng các kỹ sư có khả năng lách qua các quy trình mà họ cho là quá nặng nề hoặc mang lại giá trị không đủ—đặc biệt khi một nhóm đã ở chế độ crunch (tăng tốc gấp), và quy trình ra mắt được xem như chỉ là một mục khác đang chặn lần ra mắt của họ. Vì lý do này, LCE phải tối ưu hóa trải nghiệm ra mắt liên tục để đạt được sự cân bằng đúng giữa chi phí và lợi ích.

### Checklist Ra Mát (The Launch Checklist)

Các checklist được sử dụng để giảm thất bại và đảm bảo tính nhất quán và đầy đủ qua nhiều ngành nghề khác nhau. Các ví dụ phổ biến bao gồm các checklist trước khi bay của hàng không và các checklist phẫu thuật [[Gaw09]](https://sre.google/sre-book/bibliography#Gaw09). Tương tự, LCE sử dụng một checklist ra mắt để chứng nhận (qualify) một lần ra mắt. Checklist ([Launch Coordination Checklist](https://sre.google/sre-book/launch-checklist/)) giúp một LCE đánh giá lần ra mắt và cung cấp cho nhóm ra mắt các hành động cần thực hiện và các chỉ dẫn đến nhiều thông tin hơn. Dưới đây là một số ví dụ về các mục mà một checklist có thể bao gồm:

-   **Câu hỏi**: Bạn có cần một tên miền (domain name) mới?
    
    -   **Hành động cần thực hiện**: Phối hợp với tiếp thị về tên miền mong muốn của bạn, và yêu cầu đăng ký tên miền. Đây là một liên kết đến biểu mẫu tiếp thị.
-   **Câu hỏi**: Bạn có đang lưu trữ dữ liệu lâu dài (persistent data) không?
    
    -   **Hành động cần thực hiện**: Đảm bảo rằng bạn triển khai backup. Đây là các hướng dẫn để triển khai backup.
-   **Câu hỏi**: Một người dùng có thể lạm dụng dịch vụ của bạn không?
    
    -   **Hành động cần thực hiện**: Triển khai rate limiting và quota (định mức). Sử dụng dịch vụ chia sẻ sau.

Trên thực tế, có một số lượng gần như vô hạn các câu hỏi có thể hỏi về bất kỳ hệ thống nào, và thật dễ dàng để checklist tăng lên đến một kích thước không thể quản lý. Duy trì một gánh nặng có thể quản lý được cho các nhà phát triển đòi hỏi sự biên soạn cẩn thận của checklist. Trong nỗ lực kiềm chế sự tăng trưởng của nó, vào một thời điểm, việc thêm các câu hỏi mới vào checklist ra mắt của Google đòi hỏi sự phê duyệt từ một phó tổng giám đốc. LCE hiện sử dụng các hướng dẫn sau:

-   Sự quan trọng của mọi câu hỏi phải được chứng minh, lý tưởng là bằng một thảm họa ra mắt trước đó.
-   Mọi chỉ dẫn phải cụ thể, thực tế, và hợp lý để các nhà phát triển hoàn thành.

Checklist cần sự chú ý liên tục để vẫn có liên quan và cập nhật: các khuyến nghị thay đổi theo thời gian, các hệ thống nội bộ được thay thế bởi các hệ thống khác, và các lĩnh vực lo ngại từ các lần ra mắt trước đó trở nên lỗi thời do các chính sách và quy trình mới. Các LCE biên soạn checklist liên tục và thực hiện các cập nhật nhỏ khi các thành viên nhóm chú ý đến các mục cần được sửa đổi. Một hoặc hai lần một năm, một thành viên nhóm xem xét toàn bộ checklist để xác định các mục lỗi thời, và sau đó làm việc với các chủ sở hữu dịch vụ và các chuyên gia về chủ đề để hiện đại hóa các phần của checklist.

### Thúc Đẩy Sự hội Tụ và Đơn Giản Hóa (Driving Convergence and Simplification)

Trong một tổ chức lớn, các kỹ sư có thể không hay biết về hạ tầng khả dụng cho các tác vụ phổ biến (chẳng hạn như rate limiting). Thiếu sự hướng dẫn phù hợp, họ có khả năng lại triển khai các giải pháp hiện có. Việc hội tụ về một tập các thư viện hạ tầng chung tránh kịch bản này và mang lại những lợi ích hiển nhiên cho công ty: nó cắt giảm công việc trùng lặp, làm cho kiến thức dễ chuyển giao hơn giữa các dịch vụ, và dẫn đến một mức độ chất lượng kỹ thuật và dịch vụ cao hơn do sự chú ý được tập trung dành cho hạ tầng.

Gần như tất cả các nhóm ở Google tham gia vào một quy trình ra mắt chung, điều này làm cho checklist ra mắt trở thành một phương tiện để thúc đẩy sự hội tụ về hạ tầng chung. Thay vì triển khai một giải pháp tùy chỉnh, LCE có thể khuyến nghị hạ tầng hiện có như các khối xây dựng—hạ tầng đã được củng cố qua nhiều năm kinh nghiệm và có thể giúp giảm thiểu các rủi ro về năng lực, hiệu suất, hoặc khả năng mở rộng. Các ví dụ bao gồm hạ tầng chung cho rate limiting hoặc quota người dùng, đẩy dữ liệu mới đến các server, hoặc phát hành các phiên bản mới của một binary (file nhị phân). Loại chuẩn hóa này đã giúp đơn giản hóa đáng kể checklist ra mắt: ví dụ, các phần dài của checklist xử lý các yêu cầu về rate limiting có thể được thay thế bằng một dòng duy nhất ghi, "Triển khai rate limiting sử dụng hệ thống X."

Do chiều rộng kinh nghiệm của họ trên tất cả các sản phẩm của Google, các LCE cũng ở một vị trí độc đáo để xác định các cơ hội đơn giản hóa. Trong khi làm việc trên một lần ra mắt, họ chứng kiến các chướng ngại vật một cách trực tiếp: những phần nào của một lần ra mắt đang gây ra nhiều rắc rối nhất, những bước nào mất một lượng thời gian không tương xứng, những vấn đề nào được giải quyết độc lập lặp đi lặp lại theo những cách tương tự, hạ tầng chung bị thiếu ở đâu, hoặc sự trùng lặp tồn tại ở đâu trong hạ tầng chung. Các LCE có nhiều cách để tinh giản trải nghiệm ra mắt và đóng vai trò những người ủng hộ cho các nhóm ra mắt. Ví dụ, các LCE có thể làm việc với các chủ sở hữu của một quy trình phê duyệt đặc biệt vất vả để đơn giản hóa các tiêu chí của họ và triển khai các phê duyệt tự động cho các trường hợp phổ biến. Các LCE cũng có thể đẩy cao (escalate) các điểm đau đến các chủ sở hữu của hạ tầng chung và tạo ra một cuộc đối thoại với các khách hàng. Bằng cách tận dụng kinh nghiệm đạt được qua nhiều lần ra mắt trước đó, các LCE có thể dành nhiều sự chú ý hơn cho các mối quan tâm và đề xuất cá nhân.

### Ra Mắt Điều Không Mong Đợi (Launching the Unexpected)

Khi một dự án bước vào một không gian hoặc ngành dọc sản phẩm mới, một LCE có thể cần tạo một checklist phù hợp từ đầu. Việc làm này thường liên quan đến việc tổng hợp kinh nghiệm từ các chuyên gia trong các lĩnh vực liên quan. Khi phác thảo một checklist mới, có thể hữu ích khi cấu trúc checklist quanh các chủ đề rộng như độ tin cậy, các chế độ lỗi, và các quy trình.

Ví dụ, trước khi ra mắt Android, Google hiếm khi xử lý các thiết bị tiêu dùng hàng loạt có logic phía client mà chúng tôi không trực tiếp kiểm soát. Trong khi chúng tôi có thể khá dễ dàng sửa một bug trong Gmail trong vòng vài giờ hoặc vài ngày bằng cách đẩy các phiên bản mới của JavaScript đến các trình duyệt, những bản sửa như vậy không phải là một tùy chọn với các thiết bị di động. Do đó, các LCE làm việc trên các lần ra mắt di động đã thu hút các chuyên gia trong lĩnh vực di động để xác định những phần của các checklist hiện có áp dụng hoặc không áp dụng, và ở đâu các câu hỏi checklist mới cần thiết. Trong những cuộc trò chuyện như vậy, điều quan trọng là giữ trong tâm trí *ý định* của mỗi câu hỏi để tránh áp dụng một cách máy móc một câu hỏi hoặc hành động cụ thể không liên quan đến thiết kế của sản phẩm độc đáo đang ra mắt. Một LCE đối mặt với một lần ra mắt bất thường phải quay lại các nguyên lý trừu tượng bậc nhất về cách thực hiện một lần ra mắt an toàn, sau đó tái chuyên biệt hóa để làm cho checklist cụ thể và hữu ích cho các nhà phát triển.

## Phát Triển Một Checklist Ra Mắt (Developing a Launch Checklist)

Một checklist là thiết yếu để ra mắt các dịch vụ và sản phẩm mới với độ tin cậy có thể tái lập. Checklist ra mắt của chúng tôi tăng lên theo thời gian và được định kỳ biên soạn bởi các thành viên của nhóm Launch Coordination Engineering. Các chi tiết của một checklist ra mắt sẽ khác nhau ở mọi công ty, vì các chi tiết cụ thể phải được tùy biến cho các dịch vụ và hạ tầng nội bộ của một công ty. Trong các phần sau, chúng tôi trích xuất một số chủ đề từ các checklist LCE của Google và cung cấp các ví dụ về cách những chủ đề như vậy có thể được cụ thể hóa.

### Kiến Trúc và Các Sự Phụ Thuộc (Architecture and Dependencies)

Một cuộc rà soát kiến trúc cho phép bạn xác định xem dịch vụ có đang sử dụng hạ tầng chia sẻ đúng cách không và xác định các chủ sở hữu của hạ tầng chia sẻ như các bên liên quan bổ sung trong lần ra mắt. Google có một số lượng lớn các dịch vụ nội bộ được sử dụng như các khối xây dựng cho các sản phẩm mới. Trong các giai đoạn sau của việc lập kế hoạch năng lực (xem [[Hix15a]](https://sre.google/sre-book/bibliography#Hix15a)), danh sách các sự phụ thuộc được xác định trong phần này của checklist có thể được sử dụng để đảm bảo rằng mọi sự phụ thuộc được cung cấp đúng cách.

#### Các câu hỏi checklist ví dụ

-   Dòng request của bạn từ user đến frontend đến backend là gì?
-   Có các loại request khác nhau với các yêu cầu latency khác nhau không?

#### Các hành động cần thực hiện ví dụ

-   Cô lập các request hướng đến người dùng khỏi các request không hướng đến người dùng.
-   Xác thực các giả định về lượng request. Một lượt xem trang có thể biến thành nhiều request.

### Tích Hợp (Integration)

Nhiều dịch vụ của các công ty chạy trong một hệ sinh thái nội bộ đòi hỏi các hướng dẫn về cách thiết lập các máy, cấu hình các dịch vụ mới, thiết lập giám sát, tích hợp với load balancing, thiết lập các địa chỉ DNS, và vân vân. Những hệ sinh thái nội bộ này thường phát triển theo thời gian và thường có những đặc điểm riêng và bẫy của chúng để điều hướng. Do đó, phần này của checklist sẽ khác nhau rất nhiều từ công ty này sang công ty khác.

#### Các hành động cần thực hiện ví dụ

-   Thiết lập một tên DNS mới cho dịch vụ của bạn.
-   Thiết lập các load balancer để nói chuyện với dịch vụ của bạn.
-   Thiết lập giám sát cho dịch vụ mới của bạn.

### Lập Kế Hoạch Năng Lực (Capacity Planning)

Các tính năng mới có thể thể hiện một sự tăng sử dụng tạm thời vào lúc ra mắt mà lắng xuống trong vòng vài ngày. Loại workload hoặc sự pha trộn traffic từ một lần tăng đột biến ra mắt có thể khác biệt đáng kể so với trạng thái ổn định, làm lệch kết quả kiểm thử tải. Sự quan tâm của công chúng nổi tiếng là khó dự đoán, và một số sản phẩm của Google đã phải ứng phó với các lần tăng đột biến ra mắt cao gấp 15 lần so với ước tính ban đầu. Ra mắt ban đầu ở một khu vực hoặc một quốc gia mỗi lần giúp xây dựng niềm tin để xử lý các lần ra mắt lớn hơn.

Năng lực tương tác với sự dự phòng và khả năng hoạt động. Ví dụ, nếu bạn cần ba triển khai được nhân bản để phục vụ 100% traffic của bạn vào lúc đỉnh, bạn cần duy trì bốn hoặc năm triển khai, một hoặc hai cái trong số đó là dự phòng, để che chở người dùng khỏi việc bảo trì và các sự cố bất ngờ. Các tài nguyên datacenter và mạng thường có một thời gian lead dài và cần được yêu cầu đủ sớm để công ty bạn có thể đạt được chúng.

#### Các câu hỏi checklist ví dụ

-   Lần ra mắt này có được gắn với một thông cáo báo chí, quảng cáo, bài blog, hoặc một hình thức tiếp thị khác không?
-   Bạn kỳ vọng bao nhiêu traffic và tốc độ tăng trưởng trong và sau lần ra mắt?
-   Bạn đã đạt được tất cả các tài nguyên tính toán cần thiết để hỗ trợ traffic của bạn chưa?

### Các Chế Độ Lỗi (Failure Modes)

Một cái nhìn có hệ thống vào các chế độ lỗi có thể của một dịch vụ mới đảm bảo độ tin cậy cao ngay từ đầu. Trong phần này của checklist, hãy xem xét mỗi thành phần và sự phụ thuộc và xác định tác động của sự hỏng hóc của chúng. Dịch vụ có thể xử lý được các sự hỏng hóc máy đơn lẻ không? Các sự cố datacenter? Các sự hỏng hóc mạng? Chúng ta xử lý dữ liệu đầu vào xấu như thế nào? Chúng ta có sẵn sàng cho khả năng của một cuộc tấn công from chối dịch vụ (denial-of-service - DoS) không? Dịch vụ có thể tiếp tục phục vụ ở chế độ suy giảm (degraded mode) nếu một trong các sự phụ thuộc của nó bị hỏng không? Chúng ta xử lý sự không khả dụng của một sự phụ thuộc vào lúc khởi động của dịch vụ như thế nào? Trong khi chạy (during runtime)?

#### Các câu hỏi checklist ví dụ

-   Bạn có bất kỳ điểm lỗi đơn lẻ (single points of failure) nào trong thiết kế của bạn không?
-   Bạn giảm thiểu sự không khả dụng của các sự phụ thuộc của mình như thế nào?

#### Các hành động cần thực hiện ví dụ

-   Triển khai các deadline cho request để tránh cạn kiệt tài nguyên cho các request chạy lâu.
-   Triển khai load shedding để từ chối các request mới sớm trong các tình trạng quá tải.

### Hành Vi của Client (Client Behavior)

Trên một website truyền thống, hiếm khi có nhu cầu phải tính đến hành vi lạm dụng từ những người dùng hợp pháp. Khi mọi request được kích hoạt bởi một hành động của người dùng chẳng hạn như một cú click vào một liên kết, các tỷ lệ request bị giới hạn bởi tốc độ mà người dùng có thể click. Để tăng gấp đôi tải, số lượng người dùng phải tăng gấp đôi.

Điều này không còn đúng khi chúng ta xét các client khởi tạo các hành động mà không có đầu vào của người dùng—ví dụ, một app điện thoại định kỳ đồng bộ hóa (sync) dữ liệu của nó lên cloud, hoặc một website định kỳ làm mới (refresh). Trong cả hai kịch bản này, hành vi client lạm dụng có thể rất dễ dàng đe dọa sự ổn định của một dịch vụ. (Còn có chủ đề bảo vệ một dịch vụ khỏi traffic lạm dụng chẳng hạn như các scraper (trình cào dữ liệu) và các cuộc tấn công from chối dịch vụ—điều này khác với việc thiết kế hành vi an toàn cho các client bên thứ nhất (first-party).)

#### Câu hỏi checklist ví dụ

-   Bạn có tính năng auto-save/auto-complete/heartbeat không?

#### Các hành động cần thực hiện ví dụ

-   Đảm bảo rằng client của bạn back off theo cấp số mũ khi thất bại.
-   Đảm bảo rằng bạn jitter các request tự động.

### Các Quy Trình và Tự Động Hóa (Processes and Automation)

Google khuyến khích các kỹ sư sử dụng các công cụ chuẩn để tự động hóa các quy trình phổ biến. Tuy nhiên, tự động hóa không bao giờ hoàn hảo, và mọi dịch vụ đều có các quy trình cần được thực hiện bởi một con người: tạo một release (phiên bản phát hành) mới, di chuyển dịch vụ đến một datacenter khác, khôi phục dữ liệu từ backup, và vân vân. Vì lý do độ tin cậy, chúng tôi nỗ lực để giảm thiểu các điểm lỗi đơn lẻ, bao gồm cả con người.

Các quy trình còn lại này nên được tài liệu hóa trước khi ra mắt để đảm bảo rằng thông tin được chuyển từ tâm trí của một kỹ sư ra giấy trong khi nó vẫn còn mới, và rằng nó khả dụng trong một tình trạng khẩn cấp. Các quy trình nên được tài liệu hóa theo cách mà bất kỳ thành viên nào của nhóm cũng có thể thực hiện một quy trình nhất định trong một tình trạng khẩn cấp.

#### Câu hỏi checklist ví dụ

-   Có bất kỳ quy trình thủ công nào được yêu cầu để giữ cho dịch vụ chạy không?

#### Các hành động cần thực hiện ví dụ

-   Tài liệu hóa tất cả các quy trình thủ công.
-   Tài liệu hóa quy trình để di chuyển dịch vụ của bạn đến một datacenter mới.
-   Tự động hóa quy trình để xây dựng và phát hành một phiên bản mới.

### Quy Trình Phát Triển (Development Process)

Google là một người dùng rộng rãi của version control (kiểm soát phiên bản), và hầu như tất cả các quy trình phát triển đều được tích hợp sâu với hệ thống này. Nhiều thực hành tốt nhất của chúng tôi xoay quanh cách sử dụng hệ thống kiểm soát phiên bản một cách hiệu quả. Ví dụ, chúng tôi thực hiện phần lớn việc phát triển trên nhánh mainline (nhánh chính), nhưng các release được xây dựng trên các nhánh riêng cho từng release. Việc thiết lập này làm cho việc sửa các bug trong một release trở nên dễ dàng mà không kéo theo những thay đổi không liên quan từ mainline.

Google cũng sử dụng version control cho các mục đích khác, chẳng hạn như lưu trữ các file cấu hình. Nhiều lợi ích của version control — theo dõi lịch sử, gán các thay đổi cho các cá nhân, và code review — cũng áp dụng cho các file cấu hình. Trong một số trường hợp, chúng tôi cũng tự động đẩy các thay đổi từ hệ thống kiểm soát phiên bản đến các server trực tiếp, sao cho một kỹ sư chỉ cần đệ trình một thay đổi để đưa nó lên production.

#### Các hành động cần thực hiện ví dụ

-   Kiểm tra tất cả mã và file cấu hình vào hệ thống version control.
-   Cắt mỗi release trên một nhánh release mới.

### Các Sự Phụ Thuộc Bên Ngoài (External Dependencies)

Đôi khi một lần ra mắt phụ thuộc vào các yếu tố vượt quá tầm kiểm soát của công ty. Việc xác định những yếu tố này cho phép bạn giảm thiểu sự khó lường mà chúng kéo theo. Ví dụ, sự phụ thuộc có thể là một thư viện mã được duy trì bởi các bên thứ ba, hoặc một dịch vụ hoặc dữ liệu được cung cấp bởi một công ty khác. Khi một sự cố nhà cung cấp, bug, lỗi có hệ thống, vấn đề bảo mật, hoặc giới hạn khả năng mở rộng bất ngờ thực sự xảy ra, việc lập kế hoạch trước sẽ cho phép bạn ngăn chặn hoặc giảm nhẹ tổn hại cho người dùng của bạn. Trong lịch sử ra mắt của Google, chúng tôi đã sử dụng các proxy lọc và/hoặc viết lại, các pipeline transcoding (chuyển đổi mã hóa) dữ liệu, và cache để giảm thiểu một số những rủi ro này.

#### Các câu hỏi checklist ví dụ

-   Mã, dữ liệu, dịch vụ, hoặc sự kiện bên thứ ba nào mà dịch vụ hoặc lần ra mắt phụ thuộc vào?
-   Có đối tác nào phụ thuộc vào dịch vụ của bạn không? Nếu có, họ có cần được thông báo về lần ra mắt của bạn không?
-   Điều gì sẽ xảy ra nếu bạn hoặc nhà cung cấp không thể đáp ứng một hạn chát ra mắt cứng?

### Lập Kế Hoạch Triển Khai (Rollout Planning)

Trong các hệ thống phân tán lớn, ít sự kiện nào xảy ra tức thời. Vì lý do độ tin cậy, sự tức thời như vậy thường không lý tưởng dù sao. Một lần ra mắt phức tạp có thể yêu cầu kích hoạt các tính năng cá nhân trên một số hệ thống phụ khác nhau, và mỗi thay đổi cấu hình đó có thể mất hàng giờ để hoàn thành. Có một cấu hình hoạt động trong một instance thử nghiệm không đảm bảo rằng cùng cấu hình đó có thể được triển khai đến instance trực tiếp. Đôi khi một vũ điệu phức tạp hoặc tính năng đặc biệt được yêu cầu để làm cho tất cả các thành phần ra mắt sạch sẽ và theo đúng thứ tự.

Các yêu cầu bên ngoài từ các nhóm như tiếp thị và PR (quan hệ công chúng) có thể thêm các rắc rối khác. Ví dụ, một nhóm có thể cần một tính năng khả dụng kịp thời cho bài thuyết trình chính (keynote) tại một hội nghị, nhưng cần giữ tính năng ẩn cho đến trước bài keynote.

Các biện pháp dự phòng là một phần khác của lập kế hoạch triển khai. Điều gì nếu bạn không kịp kích hoạt tính năng cho bài keynote? Đôi khi những biện pháp dự phòng này đơn giản như chuẩn bị một bộ slide dự phòng ghi, "Chúng tôi sẽ ra mắt tính năng này trong vài ngày tới" thay vì "Chúng tôi đã ra mắt tính năng này."

#### Các hành động cần thực hiện ví dụ

-   Thiết lập một kế hoạch ra mắt xác định các hành động cần thực hiện để ra mắt dịch vụ. Xác định ai chịu trách nhiệm cho mỗi mục.
-   Xác định rủi ro trong các bước ra mắt cá nhân và triển khai các biện pháp dự phòng.

## Các Kỹ Thuật Được Chọn cho Các Lần Ra Mắt Đáng Tin Cậy (Selected Techniques for Reliable Launches)

Như đã mô tả trong các phần khác của cuốn sách này, Google đã phát triển một số kỹ thuật để vận hành các hệ thống đáng tin cậy qua nhiều năm. Một số kỹ thuật trong số này đặc biệt phù hợp để ra mắt các sản phẩm một cách an toàn. Chúng cũng mang lại các lợi thế trong quá trình vận hành bình thường của dịch vụ, nhưng đặc biệt quan trọng là thực hiện chúng đúng trong giai đoạn ra mắt.

### Triển Khai Từ Chậm và Theo Giai Đoạn (Gradual and Staged Rollouts)

Một châm ngôn của quản trị hệ thống là "đừng bao giờ thay đổi một hệ thống đang chạy." Bất kỳ thay đổi nào đều đại diện cho rủi ro, và rủi ro nên được giảm thiểu để đảm bảo [độ tin cậy của một hệ thống.](https://sre.google/resources/practices-and-processes/product-focused-reliability-for-sre/) Điều gì đúng cho một hệ thống nhỏ bất kỳ thì đúng gấp đôi cho các hệ thống được nhân bản cao, phân phối toàn cầu như những hệ thống do Google vận hành.

Rất ít các lần ra mắt ở Google thuộc loại "nút bấm (push-button)", trong đó chúng tôi ra mắt một sản phẩm mới tại một thời điểm cụ thể cho toàn thế giới sử dụng. Theo thời gian, Google đã phát triển một số pattern cho phép chúng tôi ra mắt các sản phẩm và tính năng một cách từ từ và qua đó giảm thiểu rủi ro; xem [A Collection of Best Practices for Production Services](https://sre.google/sre-book/service-best-practices/).

Gần như tất cả các bản cập nhật cho các dịch vụ của Google diễn ra từ từ, theo một quy trình được xác định, với các bước xác thực phù hợp được xen kẽ. Một server mới có thể được cài đặt trên một vài máy trong một datacenter và được quan sát trong một khoảng thời gian xác định. Nếu mọi thứ có vẻ ổn, server được cài đặt trên tất cả các máy trong một datacenter, được quan sát lại, và sau đó được cài đặt trên tất cả các máy trên toàn cầu. Các giai đoạn đầu của một lần triển khai thường được gọi là "canaries (chim canari)"—một sự ám chỉ đến những con chim canari được các thợ mỏ đưa vào một mỏ than để phát hiện các khí độc hại. Các server canary của chúng tôi phát hiện các tác động nguy hiểm từ hành vi của phần mềm mới dưới traffic người dùng thực.

Canary testing (kiểm thử canari) là một khái niệm được nhúng vào nhiều công cụ nội bộ của Google được sử dụng để thực hiện các thay đổi tự động, cũng như cho các hệ thống thay đổi các file cấu hình. Các công cụ quản lý việc cài đặt phần mềm mới thường quan sát server vừa được khởi động trong một thời gian, đảm bảo rằng server không crash (sập) hoặc hành vi sai lệch. Nếu thay đổi không vượt qua thời kỳ xác thực, nó được tự động rollback (hoàn tác).

Khái niệm triển khai từ từ thậm chí áp dụng cho phần mềm không chạy trên các server của Google. Các phiên bản mới của một app Android có thể được triển khai một cách từ từ, trong đó phiên bản cập nhật được cung cấp cho một tập con các cài đặt để nâng cấp. Tỷ lệ phần trăm các instance được nâng cấp tăng dần theo thời gian cho đến khi đạt 100%. Loại triển khai này đặc biệt hữu ích nếu phiên bản mới dẫn đến thêm traffic đến các server backend trong các datacenter của Google. Bằng cách này, chúng tôi có thể quan sát tác động lên các server của mình khi chúng tôi dần dần triển khai phiên bản mới và phát hiện các vấn đề sớm.

Hệ thống mời (invite system) là một loại triển khai từ từ khác. Thường, thay vì cho phép đăng ký miễn phí vào một dịch vụ mới, chỉ một số lượng hạn chế người dùng được phép đăng ký mỗi ngày. Các đăng ký bị giới hạn tỷ lệ thường được kết hợp với một hệ thống mời, trong đó một người dùng có thể gửi một số lượng hạn chế các lời mời đến bạn bè.

### Các Framework cờ tính năng (Feature Flag Frameworks)

Google thường bổ sung các bài kiểm thử trước khi ra mắt bằng các chiến lược giảm thiểu rủi ro của một sự cố. Một cơ chế để triển khai các thay đổi từ từ, cho phép quan sát hành vi tổng thể của hệ thống dưới các workload thực, có thể tự trang trải đầu tư kỹ thuật của nó về độ tin cậy, velocity kỹ thuật, và thời gian ra thị trường. Những cơ chế này đã chứng minh là đặc biệt hữu ích trong các trường hợp mà các môi trường kiểm thử thực tế là không thực tế, hoặc cho các lần ra mắt đặc biệt phức tạp mà các tác động của chúng có thể khó dự đoán.

Hơn nữa, không phải tất cả các thay đổi đều bằng nhau. Đôi khi bạn đơn giản chỉ muốn kiểm tra xem một điều chỉnh nhỏ cho giao diện người dùng có cải thiện trải nghiệm của người dùng của bạn không. Những thay đổi nhỏ như vậy không nên liên quan đến hàng nghìn dòng mã hoặc một quy trình ra mắt nặng nề. Bạn có thể muốn kiểm thử hàng trăm những thay đổi như vậy song song.

Cuối cùng, đôi khi bạn muốn tìm hiểu xem liệu một mẫu người dùng nhỏ có thích sử dụng một nguyên mẫu sớm của một tính năng mới khó triển khai hay không. Bạn không muốn dành hàng tháng nỗ lực kỹ thuật để củng cố một tính năng mới để phục vụ hàng triệu người dùng, chỉ để phát hiện ra rằng tính năng đó là một sự thất bại.

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

Framework cờ tính năng đơn giản nhất cho các thay đổi giao diện người dùng trong một dịch vụ không trạng thái (stateless) là một bộ viết lại (rewriter) payload HTTP trên các server ứng dụng frontend, bị giới hạn ở một tập con các cookie hoặc một thuộc tính request/response HTTP tương tự khác. Một cơ chế cấu hình có thể xác định một định danh (identifier) liên quan đến các đường dẫn mã mới và phạm vi của thay đổi (ví dụ, cookie hash mod range), các whitelist (danh sách cho phép), và các blacklist (danh sách cấm).

Các dịch vụ có trạng thái (stateful) có xu hướng giới hạn các cờ tính năng ở một tập con các định danh người dùng đăng nhập duy nhất hoặc đến các thực thể sản phẩm thực tế được truy cập, chẳng hạn như ID của các tài liệu, bảng tính, hoặc object lưu trữ. Thay vì viết lại các payload HTTP, các dịch vụ có trạng thái có nhiều khả năng proxy (trình trung gian) hoặc định tuyến lại các request đến các server khác nhau tùy theo thay đổi, mang lại khả năng kiểm thử logic kinh doanh cải thiện và các tính năng mới phức tạp hơn.

### Đối Phó Với Hành Vi Client Lạm Dụng (Dealing with Abusive Client Behavior)

Ví dụ đơn giản nhất của hành vi client lạm dụng là một sự đánh giá sai về các tỷ lệ cập nhật. Một client mới đồng bộ hóa mỗi 60 giây, thay vì mỗi 600 giây, gây ra gấp 10 lần tải lên dịch vụ. Hành vi retry có một số bẫy ảnh hưởng đến cả các request do người dùng khởi tạo, cũng như các request do client khởi tạo. Hãy lấy ví dụ về một dịch vụ đang bị quá tải và vì vậy đang thất bại một số request: nếu các client thử lại các request thất bại, chúng thêm tải vào một dịch vụ đã bị quá tải, dẫn đến nhiều lần thử lại hơn và thậm chí nhiều request hơn. Thay vào đó, các client cần giảm tần suất thử lại, thường bằng cách thêm một độ trễ tăng theo cấp số mũ giữa các lần thử lại, ngoài việc cẩn thận cân nhắc các loại lỗi xứng đáng để thử lại. Ví dụ, một lỗi mạng thường xứng đáng để thử lại, nhưng một lỗi HTTP 4xx (cho thấy một lỗi ở phía client) thường thì không.

Sự đồng bộ hóa có chủ ý hoặc vô tình các request tự động trong một "bầy thú đang gầm gừ" (thundering herd, giống như những thứ được mô tả trong các chương [Định Kỳ Phân Tán với Cron](https://sre.google/sre-book/distributed-periodic-scheduling/) và [Pipeline Xử Lý Dữ Liệu](https://sre.google/sre-book/data-processing-pipelines/)) là một ví dụ phổ biến khác của hành vi client lạm dụng. Một nhà phát triển app điện thoại có thể quyết định rằng 2 giờ sáng là một thời điểm tốt để tải xuống các bản cập nhật, vì người dùng có khả năng đang ngủ và sẽ không bị phiền toái bởi việc tải xuống. Tuy nhiên, một thiết kế như vậy dẫn đến một loạt các request đến server tải xuống lúc 2 giờ sáng mỗi đêm, và gần như không có request vào bất kỳ thời điểm nào khác. Thay vào đó, mọi client nên chọn thời điểm cho loại request này một cách ngẫu nhiên.

Tính ngẫu nhiên cũng cần được tiêm vào các quy trình định kỳ khác. Quay lại các lần thử lại được đề cập trước đó: hãy lấy ví dụ về một client gửi một request, và khi nó gặp một sự thất bại, thử lại sau 1 giây, sau đó 2 giây, sau đó 4 giây, và vân vân. Nếu không có tính ngẫu nhiên, một đợt tăng request ngắn dẫn đến tỷ lệ lỗi tăng có thể lặp lại chính nó do các lần thử lại sau 1 giây, sau đó 2 giây, sau đó 4 giây. Để làm cho đều các sự kiện được đồng bộ này, mỗi độ trễ cần được jitter (tức là, điều chỉnh bằng một lượng ngẫu nhiên).

Khả năng kiểm soát hành vi của một client từ phía server đã chứng minh là một công cụ quan trọng. Đối với một app trên một thiết bị, sự kiểm soát như vậy có thể có nghĩa là hướng dẫn client kiểm tra (check in) định kỳ với server và tải xuống một file cấu hình. File đó có thể bật hoặc tắt một số tính năng hoặc đặt các tham số, chẳng hạn như client đồng bộ hóa bao lâu một lần hoặc thử lại bao lâu một lần.

Cấu hình client thậm chí có thể bật một tính năng hoàn toàn mới hướng đến người dùng. Bằng cách đăng tải mã hỗ trợ tính năng mới trong ứng dụng client trước khi chúng tôi kích hoạt tính năng đó, chúng tôi giảm đáng kể rủi ro liên quan đến một lần ra mắt. Việc phát hành một phiên bản mới trở nên dễ dàng hơn nhiều nếu chúng tôi không cần duy trì các đường release song song cho một phiên bản có tính năng mới so với không có tính năng. Điều này đặc biệt đúng nếu chúng tôi không đang xử lý một mảnh tính năng mới đơn lẻ, mà là một tập các tính năng độc lập có thể được phát hành theo các lịch trình khác nhau, điều đó đòi hỏi phải duy trì một sự bùng nổ tổ hợp (combinatorial explosion) của các phiên bản khác nhau.

Việc có loại tính năng ngủ đông (dormant) này cũng làm cho việc hủy bỏ các lần ra mắt trở nên dễ dàng hơn khi các tác động bất lợi được phát hiện trong quá trình triển khai. Trong những trường hợp như vậy, chúng tôi đơn giản có thể chuyển tính năng sang tắt, lặp lại (iterate), và phát hành một phiên bản cập nhật của app. Nếu không có loại cấu hình client này, chúng tôi sẽ phải cung cấp một phiên bản app mới không có tính năng đó, và cập nhật app trên điện thoại của tất cả người dùng.

### Hành Vi Quá Tải và Kiểm Thử Tải (Overload Behavior and Load Tests)

Các tình trạng quá tải là một chế độ lỗi đặc biệt phức tạp, và do đó xứng đáng được chú ý thêm. Sự thành công mất kiểm soát thường là nguyên nhân đáng được chào đón nhất của quá tải khi một dịch vụ mới ra mắt, nhưng có vô số các nguyên nhân khác, bao gồm các sự hỏng hóc cân bằng tải, các sự cố máy, hành vi client đồng bộ hóa, và các cuộc tấn công bên ngoài.

Một mô hình ngây thơ giả định rằng mức sử dụng CPU trên một máy cung cấp một dịch vụ nhất định tăng tuyến tính theo tải (ví dụ, số request hoặc lượng dữ liệu được xử lý), và một khi CPU khả dụng bị cạn kiệt, việc xử lý đơn giản là trở nên chậm hơn. Thật không may, các dịch vụ hiếm khi hành xử theo cách lý tưởng này trong thế giới thực. Nhiều dịch vụ chậm hơn nhiều khi chúng không được tải, thường do tác động của các loại cache khác nhau chẳng hạn như CPU caches, JIT caches, và các cache dữ liệu đặc thù cho dịch vụ. Khi tải tăng lên, thường có một khoảng trong đó mức sử dụng CPU và tải trên dịch vụ tương ứng tuyến tính, và thời gian phản hồi giữ tương đối ổn định.

Ở một số điểm, nhiều dịch vụ đạt đến một điểm phi tuyến tính khi chúng tiến đến quá tải. Trong các trường hợp lành tính nhất, thời gian phản hồi đơn giản bắt đầu tăng lên, dẫn đến một trải nghiệm người dùng suy giảm nhưng không nhất thiết gây ra một sự cố (mặc dù một sự phụ thuộc chậm có thể gây ra các lỗi nhìn thấy được bởi người dùng lên trên stack, do vượt quá các deadline RPC). Trong các trường hợp gay gắt nhất, một dịch vụ bị khóa hoàn toàn (locks up) để phản ứng với quá tải.

Để dẫn ra một ví dụ cụ thể về hành vi quá tải: một dịch vụ đã ghi log (nhật ký) thông tin gỡ rối để phản ứng với các lỗi backend. Hóa ra là việc ghi log thông tin gỡ rối đắt đỏ hơn việc xử lý phản ứng backend trong một trường hợp bình thường. Do đó, khi dịch vụ trở nên quá tải và timeout các phản ứng backend bên trong stack RPC của chính nó, dịch vụ dành thêm thời gian CPU để ghi log những phản ứng này, rồi timeout nhiều request hơn, cứ thế cho đến khi dịch vụ dừng hẳn. Trong các dịch vụ chạy trên Java Virtual Machine (JVM), một hiệu ứng tương tự của việc dừng hẳn đôi khi được gọi là "GC (garbage collection - thu gom rác) thrashing (lẫy lạch)." Trong kịch bản này, việc quản lý bộ nhớ nội bộ của máy ảo chạy trong các chu kỳ ngày càng gần hơn, cố gắng giải phóng bộ nhớ cho đến khi phần lớn thời gian CPU bị tiêu thụ bởi việc quản lý bộ nhớ.

Thật không may, rất khó để dự đoán từ các nguyên lý bậc nhất một dịch vụ sẽ phản ứng thế nào với quá tải. Do đó, load tests (kiểm thử tải) là một công cụ vô giá, cả vì lý do độ tin cậy và lập kế hoạch năng lực, và kiểm thử tải được yêu cầu cho phần lớn các lần ra mắt.

## Sự Phát Triển của LCE (Development of LCE)

Trong những năm đầu hình thành của Google, kích thước của nhóm kỹ thuật tăng gấp đôi mỗi năm trong nhiều năm liền, phân mảnh phòng ban kỹ thuật thành nhiều nhóm nhỏ làm việc trên nhiều sản phẩm và tính năng mới thử nghiệm. Trong một bầu không khí như vậy, các kỹ sư non trẻ có nguy cơ lặp lại những sai lầm của những người đi trước, đặc biệt là khi nói đến việc ra mắt thành công các tính năng và sản phẩm mới.

Để giảm thiểu việc lặp lại những sai lầm như vậy bằng cách ghi lại những bài học học được từ các lần ra mắt trước đó, một nhóm nhỏ các kỹ sư giàu kinh nghiệm, được gọi là các "Launch Engineer (Kỹ sư Ra mắt)," đã tình nguyện đóng vai trò một nhóm tư vấn. Các Launch Engineer đã phát triển các checklist cho các lần ra mắt sản phẩm mới, bao phủ các chủ đề như:

-   Khi nào nên tham vấn phòng ban pháp lý
-   Cách chọn các tên miền
-   Cách đăng ký các tên miền mới mà không cấu hình sai DNS
-   Các bẫy phổ biến trong thiết kế kỹ thuật và triển khai production

"Launch Reviews (Rà soát Ra mắt)," như các buổi tư vấn của Launch Engineers được gọi, đã trở thành một thực hành phổ biến từ vài ngày đến vài tuần trước khi ra mắt nhiều sản phẩm mới.

Trong vòng hai năm, các yêu cầu triển khai sản phẩm trong checklist ra mắt ngày càng dài và phức tạp. Kết hợp với sự phức tạp ngày càng tăng của môi trường triển khai của Google, việc cập nhật về cách thực hiện các thay đổi một cách an toàn trở nên ngày càng khó khăn cho các kỹ sư sản phẩm. Đồng thời, tổ chức SRE đang phát triển nhanh, và các SRE thiếu kinh nghiệm đôi khi quá thận trọng và e dè trước sự thay đổi. Google đối mặt với rủi ro rằng các cuộc đàm phán phát sinh giữa hai bên này sẽ làm giảm velocity của các lần ra mắt sản phẩm/tính năng.

Để giảm thiểu kịch bản này từ góc nhìn kỹ thuật, SRE đã bố trí một nhóm nhỏ các LCE thời gian toàn thời gian vào năm 2004. Họ chịu trách nhiệm tăng tốc các lần ra mắt các sản phẩm và tính năng mới, trong khi đồng thời áp dụng chuyên môn SRE để đảm bảo rằng Google phát hành các sản phẩm đáng tin cậy với khả năng hoạt động cao và latency thấp.

Các LCE chịu trách nhiệm đảm bảo các lần ra mắt được thực hiện nhanh chóng mà không để các dịch vụ sụp đổ, và rằng nếu một lần ra mắt thất bại, nó không kéo theo các sản phẩm khác. Các LCE cũng chịu trách nhiệm giữ cho các bên liên quan được thông báo về bản chất và khả năng của những sự thất bại như vậy bất cứ khi nào những góc cạnh bị cắt để tăng tốc thời gian ra thị trường. Các buổi tư vấn của họ được chính thức hóa thành các Production Review (Rà soát Production).

### Sự Tiến Hóa của Checklist LCE (Evolution of the LCE Checklist)

Khi môi trường của Google trở nên phức tạp hơn, cả checklist Launch Coordination Engineering (xem [Launch Coordination Checklist](https://sre.google/sre-book/launch-checklist)) và lượng các lần ra mắt cũng tăng lên. Trong 3,5 năm, một LCE đã chạy 350 lần ra mắt qua LCE Checklist. Khi nhóm trung bình năm kỹ sư trong khoảng thời gian này, điều này tương đương với một năng suất ra mắt của Google là hơn 1.500 lần ra mắt trong 3,5 năm!

Trong khi mỗi câu hỏi trên LCE Checklist là đơn giản, nhiều sự phức tạp ẩn chứa trong lý do mà câu hỏi đó được đặt ra và các hệ quả của câu trả lời. Để hoàn toàn hiểu được mức độ phức tạp này, một nhân viên LCE mới cần khoảng sáu tháng đào tạo.

Khi lượng các lần ra mắt tăng lên, theo kịp sự tăng gấp đôi hàng năm của nhóm kỹ thuật của Google, các LCE tìm kiếm các cách để tinh giản các cuộc rà soát của họ. Các LCE xác định các danh mục các lần ra mắt rủi ro thấp rất ít khả năng phải đối mặt hoặc gây ra các sự cố. Ví dụ, một lần ra mắt tính năng không liên quan đến các file thực thi server mới và một sự tăng traffic dưới 10% sẽ được coi là rủi ro thấp. Những lần ra mắt như vậy phải đối mặt với một checklist gần như tầm thường, trong khi các lần ra mắt rủi ro cao hơn trải qua toàn bộ các kiểm tra và cân bằng. Đến năm 2008, 30% các cuộc rà soát được coi là rủi ro thấp.

Đồng thời, môi trường của Google đang mở rộng scale, loại bỏ các ràng buộc trên nhiều lần ra mắt. Ví dụ, việc mua lại YouTube đã buộc Google cải tiến mạng của mình và sử dụng băng thông hiệu quả hơn. Điều này có nghĩa là nhiều sản phẩm nhỏ hơn sẽ "nhích lọt vào các khe hở," tránh các quy trình lập kế hoạch năng lực mạng và cung cấp phức tạp, qua đó tăng tốc các lần ra mắt của chúng. Google cũng bắt đầu xây dựng các datacenter rất lớn có khả năng đăng ký nhiều dịch vụ phụ thuộc dưới một mái nhà. Sự phát triển này đơn giản hóa việc ra mắt các sản phẩm mới cần một lượng lớn năng lực trên nhiều dịch vụ hiện có mà chúng phụ thuộc.

### Những Vấn Đề LCE Không Giải Quyết Được (Problems LCE Didn't Solve)

Mặc dù các LCE đã cố gắng giữ sự quan liêu của các cuộc rà soát ở mức tối thiểu, những nỗ lực đó là không đủ. Đến năm 2009, khó khăn của việc ra mắt một dịch vụ mới nhỏ ở Google đã trở thành một huyền thoại. Các dịch vụ phát triển đến một quy mô lớn hơn phải đối mặt với một bộ các vấn đề riêng mà Launch Coordination không thể giải quyết.

#### Những thay đổi khả năng mở rộng (Scalability changes)

Khi các sản phẩm thành công vượt xa mọi ước tính ban đầu, và sự sử dụng của chúng tăng hơn hai bậc độ lớn, việc theo kịp tải của chúng đòi hỏi nhiều thay đổi thiết kế. Những thay đổi khả năng mở rộng như vậy, kết hợp với việc thêm tính năng liên tục, thường làm cho sản phẩm trở nên phức tạp hơn, mong manh hơn, và khó vận hành hơn. Đến một số điểm, kiến trúc sản phẩm ban đầu trở nên không thể quản lý được và sản phẩm cần được kiến trúc lại hoàn toàn. Việc kiến trúc lại sản phẩm và sau đó di chuyển tất cả người dùng từ kiến trúc cũ sang kiến trúc mới đòi hỏi một đầu tư lớn về thời gian và nguồn lực từ cả các nhà phát triển và SRE, làm chậm tỷ lệ phát triển tính năng mới trong giai đoạn đó.

#### Gánh nặng vận hành tăng lên (Growing operational load)

Khi vận hành một dịch vụ sau khi ra mắt, operational load (gánh nặng vận hành), lượng kỹ thuật thủ công và lặp lại cần thiết để giữ cho một hệ thống hoạt động, có xu hướng tăng theo thời gian trừ khi có các nỗ lực để kiểm soát gánh nặng đó. Sự ồn ào của các thông báo tự động, sự phức tạp của các quy trình triển khai, và chi phí phụ của công việc bảo trì thủ công có xu hướng tăng theo thời gian và tiêu thụ ngày càng nhiều bandwidth của chủ sở hữu dịch vụ, để lại cho nhóm ít thời gian hơn cho phát triển tính năng. SRE có một mục tiêu được quảng bá nội bộ là giữ công việc vận hành dưới một mức tối đa 50%; xem [Loại Bỏ Toil (Công Việc Lặp Đi Lặp Lại)](https://sre.google/sre-book/eliminating-toil/). Giữ dưới mức tối đa này đòi hỏi việc theo dõi liên tục các nguồn của công việc vận hành, cũng như nỗ lực có mục tiêu để loại bỏ những nguồn này.

#### Sự biến động hạ tầng (Infrastructure churn)

Nếu hạ tầng nền tảng (chẳng hạn như các hệ thống cho quản lý cụm, lưu trữ, giám sát, cân bằng tải, và chuyển dữ liệu) đang thay đổi do phát triển tích cực bởi các nhóm hạ tầng, các chủ sở hữu của các dịch vụ chạy trên hạ tầng phải đầu tư một lượng lớn công việc chỉ để theo kịp những thay đổi hạ tầng. Khi các tính năng hạ tầng mà các dịch vụ phụ thuộc bị khai tử và được thay thế bởi các tính năng mới, các chủ sở hữu dịch vụ phải liên tục sửa đổi các cấu hình của họ và xây dựng lại các file thực thi của họ, hậu quả là "chạy nhanh chỉ để giữ nguyên vị trí." Giải pháp cho kịch bản này là thực hiện một loại chính sách giảm churn nào đó, cấm các kỹ sư hạ tầng phát hành các tính năng không tương thích ngược cho đến khi họ cũng tự động hóa việc di chuyển các client của họ sang tính năng mới. Việc tạo các công cụ di chuyển tự động đi kèm với các tính năng mới làm giảm thiểu công việc áp đặt lên các chủ sở hữu dịch vụ để theo kịp sự biến động hạ tầng.

Giải quyết những vấn đề này đòi hỏi các nỗ lực trên toàn công ty vượt xa phạm vi của LCE: một sự kết hợp của các API và framework nền tảng tốt hơn (xem [Mô Hình Tham Gia SRE Đang Tiến Hóa](https://sre.google/sre-book/evolving-sre-engagement-model/)), tự động hóa build và kiểm thử liên tục, và sự chuẩn hóa và tự động hóa tốt hơn trên các dịch vụ production của Google.

## Kết Luận (Conclusion)

Các công ty đang trải qua tăng trưởng nhanh với một tỷ lệ thay đổi cao đối với các sản phẩm và dịch vụ có thể có lợi từ tương đương của một vai trò Launch Coordination Engineering. Một nhóm như vậy đặc biệt có giá trị nếu một công ty lên kế hoạch tăng gấp đôi số nhà phát triển sản phẩm của mình mỗi một hoặc hai năm, nếu nó phải mở rộng scale các dịch vụ của mình đến hàng trăm triệu người dùng, và nếu độ tin cậy bất chấp một tỷ lệ thay đổi cao là quan trọng đối với người dùng của nó.

Nhóm LCE là giải pháp của Google cho vấn đề đạt được sự an toàn mà không gây cản trở sự thay đổi. Chương này đã giới thiệu một số kinh nghiệm được tích lũy bởi vai trò LCE độc đáo của chúng tôi trong một khoảng thời gian 10 năm dưới đúng những hoàn cảnh như vậy. Chúng tôi hy vọng rằng cách tiếp cận của chúng tôi sẽ truyền cảm hứng cho những người khác đối mặt với những thách thức tương tự trong các tổ chức của họ.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
