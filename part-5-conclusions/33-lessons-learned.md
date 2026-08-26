> **Nguyên bản:** [Chapter 33 - Lessons Learned from Other Industries](https://sre.google/sre-book/lessons-learned/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

## Chương 33. Bài Học Học Được Từ Các Ngành Khác (Lessons Learned from Other Industries)

Tác giả: Jennifer Petoff  
Biên tập: Betsy Beyer

Một cuộc đi sâu vào văn hóa và thực hành SRE tại Google tự nhiên sẽ dẫn đến câu hỏi: các ngành khác quản lý độ tin cậy trong doanh nghiệp của họ như thế nào? Khi biên soạn cuốn sách về Google SRE này, chúng tôi đã trao đổi với một số kỹ sư của Google về kinh nghiệm làm việc trước đây của họ ở nhiều lĩnh vực độ tin cậy cao khác nhau, nhằm giải quyết các câu hỏi so sánh sau:

-   Các nguyên lý được sử dụng trong Site Reliability Engineering có quan trọng cả bên ngoài Google không, hay các ngành khác giải quyết các yêu cầu về độ tin cậy cao theo những cách khác biệt rõ rệt?
-   Nếu các ngành khác cũng tuân theo các nguyên lý SRE, các nguyên lý ấy được thể hiện ra sao?
-   Những điểm giống nhau và khác nhau trong việc triển khai các nguyên lý này giữa các ngành là gì?
-   Những yếu tố nào thúc đẩy sự giống nhau và khác nhau trong triển khai?
-   Google và ngành công nghệ có thể học được gì từ những sự so sánh này?

Một số nguyên lý nền tảng của Site Reliability Engineering tại Google được thảo luận xuyên suốt cuốn sách này. Để đơn giản hóa việc so sánh các thực hành tốt nhất của các ngành khác, chúng tôi đã cô đọng các khái niệm này thành bốn chủ đề chính:

-   Chuẩn bị và Kiểm thử Thảm họa (Preparedness and Disaster Testing)

-   Văn hóa Postmortem (Postmortem Culture)

-   Tự động hóa và Giảm gánh nặng Vận hành (Automation and Reduced Operational Overhead)
-   Ra quyết định Có cấu trúc và Hợp lý (Structured and Rational Decision Making)

Chương này giới thiệu các ngành mà chúng tôi đã nghiên cứu và các chuyên gia lâu năm trong ngành mà chúng tôi đã phỏng vấn. Chúng tôi xác định các chủ đề SRE chính, thảo luận cách các chủ đề này được triển khai tại Google, và đưa ra các ví dụ về cách các nguyên lý này thể hiện ở các ngành khác để phục vụ so sánh. Chúng tôi kết thúc bằng một số nhận xét và thảo luận về các pattern và anti-pattern mà chúng tôi đã phát hiện ra.

## Làm Quen Với Các Chuyên Gia Lâu Năm Trong Ngành (Meet Our Industry Veterans)

**Peter Dahl** là một Principal Engineer tại Google. Trước đó, ông làm việc cho nhà thầu quốc phòng trong một số hệ thống độ tin cậy cao, bao gồm nhiều hệ thống dẫn đường GPS và quán tính (inertial guidance) trên máy bay và xe bánh. Hậu quả của một sự suy giảm độ tin cậy trong những hệ thống như vậy bao gồm sự trục trặc hoặc mất phương tiện, cùng với những hậu quả tài chính đi kèm.

**Mike Doherty** là một Site Reliability Engineer tại Google. Ông đã làm việc mười năm với tư cách là cứu hộ viên (lifeguard) và huấn luyện viên cứu hộ viên tại Canada. Bản chất của lĩnh vực này đòi hỏi độ tin cậy tuyệt đối, vì mỗi ngày sự sống đều được đặt lên bàn cân.

**Erik Gross** hiện là một kỹ sư phần mềm tại Google. Trước khi gia nhập công ty, ông đã dành bảy năm thiết kế thuật toán và mã cho các laser và hệ thống dùng để thực hiện phẫu thuật khúc xạ mắt (ví dụ, LASIK). Đây là một lĩnh vực rủi ro cao, độ tin cậy cao, trong đó nhiều bài học về độ tin cậy — trước các quy định của chính phủ và rủi ro cho con người — đã được tích lũy khi công nghệ này được FDA phê duyệt, dần hoàn thiện và cuối cùng trở nên phổ biến.

**Gus Hartmann** và **Kevin Greer** có kinh nghiệm trong ngành viễn thông, bao gồm việc duy trì hệ thống phản ứng khẩn cấp E911.<sup>[1](#fn1)</sup> Kevin hiện là một kỹ sư phần mềm trong đội Google Chrome, còn Gus là một kỹ sư hệ thống cho đội Corporate Engineering của Google. Kỳ vọng của người dùng đối với ngành viễn thông đòi hỏi độ tin cậy cao. Hệ quả của một sự suy giảm dịch vụ có thể trải dài từ sự bất tiện cho người dùng do một sự cố hệ thống (outage) cho đến tử vong nếu E911 gặp sự cố.

**Ron Heiby** là một Technical Program Manager cho Site Reliability Engineering tại Google. Ron có kinh nghiệm phát triển cho điện thoại di động, thiết bị y tế và ngành công nghiệp ô tô. Trong một số trường hợp, ông đã làm việc trên các thành phần giao diện của những ngành này (ví dụ, trên một thiết bị cho phép các số đo EKG được truyền đi qua mạng điện thoại không dây kỹ thuật số).<sup>[2](#fn2)</sup> Trong những ngành này, tác động của một vấn đề độ tin cậy có thể trải dài từ thiệt hại cho doanh nghiệp do việc thu hồi thiết bị cho đến gián tiếp ảnh hưởng đến sự sống và sức khỏe (ví dụ, mọi người không nhận được sự chăm sóc y tế cần thiết nếu hệ thống EKG không thể truyền thông với bệnh viện).

**Adrian Hilton** là một Launch Coordination Engineer tại Google. Trước đó, ông làm việc trên các máy bay quân sự của Anh và Mỹ, hệ thống điện tử hàng hải và hệ thống quản lý vũ khí trên máy bay, cùng các hệ thống tín hiệu đường sắt của Anh. Độ tin cậy là yếu tố sống còn trong lĩnh vực này vì tác động của các sự cố trải dài từ việc mất thiết bị trị giá hàng triệu đô la cho đến chấn thương và tử vong.

**Eddie Kennedy** là một project manager cho đội Global Customer Experience tại Google và là kỹ sư cơ khí theo trình độ đào tạo. Eddie đã dành sáu năm làm việc với tư cách là một kỹ sư quy trình Six Sigma Black Belt tại một cơ sở sản xuất làm kim cương tổng hợp. Ngành công nghiệp này được đặc trưng bởi sự tập trung không khoan nhượng vào sự an toàn, bởi những điều kiện nhiệt độ và áp suất khắc nghiệt của quy trình đặt ra mức nguy hiểm cao cho người lao động mỗi ngày.

**John Li** hiện là một Site Reliability Engineer tại Google. John trước đó đã làm việc với tư cách là quản trị viên hệ thống và nhà phát triển phần mềm tại một công ty môi giới giao dịch (proprietary trading) trong ngành tài chính. Các vấn đề độ tin cậy trong khu vực tài chính được xem xét hết sức nghiêm túc vì chúng có thể dẫn đến những hậu quả tài chính nghiêm trọng.

**Dan Sheridan** là một Site Reliability Engineer tại Google. Trước khi gia nhập công ty, ông đã làm việc với tư cách là một cố vấn an toàn trong ngành công nghiệp hạt nhân dân sự tại Anh. Độ tin cậy là quan trọng trong ngành hạt nhân vì một sự cố có thể có những tác động nghiêm trọng: các sự cố ngừng hoạt động có thể gây ra mất doanh thu hàng triệu đô la mỗi ngày, trong khi rủi ro cho người lao động và những người trong cộng đồng còn nghiêm trọng hơn, đòi hỏi mức độ không dung lỗi nào cho sự thất bại. Hạ tầng hạt nhân được thiết kế với một loạt các cơ chế dự phòng (failsafe) sẽ dừng hoạt động trước khi một sự cố đạt đến quy mô nghiêm trọng như vậy.

**Jeff Stevenson** hiện là một hardware operations manager tại Google. Ông có kinh nghiệm trước đây với tư cách là một kỹ sư hạt nhân trong Hải quân Hoa Kỳ trên một con tàu ngầm. Mức độ rủi ro về độ tin cậy trong Hải quân hạt nhân là rất cao — những vấn đề phát sinh khi có sự cố có thể trải dài từ thiết bị bị hư hại, đến tác động môi trường kéo dài, cho đến khả năng mất mạng sống.

**Matthew Toia** là một Site Reliability Manager tập trung vào các hệ thống lưu trữ. Trước khi đến Google, ông đã làm việc về phát triển và triển khai phần mềm của các hệ thống kiểm soát không lưu (air traffic control). Tác động từ các sự cố trong ngành này trải dài từ sự bất tiện cho hành khách và các hãng hàng không (ví dụ, chuyến bay bị hoãn, máy bay phải đổi hướng) cho đến khả năng mất mạng sống nếu xảy ra một vụ tai nạn. Bảo vệ nhiều lớp (defense in depth) là một chiến lược then chốt để tránh các sự thất bại thảm khốc.

Giờ đây, khi bạn đã được gặp các chuyên gia của chúng tôi và có được một sự hiểu biết tổng quan về lý do độ tin cậy quan trọng trong từng lĩnh vực trước đây của họ, chúng ta sẽ đi sâu vào bốn chủ đề chính về độ tin cậy.

## Chuẩn Bị và Kiểm thử Thảm họa (Preparedness and Disaster Testing)

"Hy vọng không phải là một chiến lược." Câu khẩu hiệu này của đội SRE tại Google tóm tắt ý nghĩa mà chúng tôi muốn nói về việc chuẩn bị và kiểm thử thảm họa. Văn hóa SRE luôn tỉnh táo cảnh giác và không ngừng tự chất vấn: điều gì có thể xảy ra sai? Chúng tôi có thể thực hiện hành động nào để giải quyết những vấn đề đó trước khi chúng dẫn đến một sự cố ngừng hoạt động hoặc mất dữ liệu? Các cuộc diễn tập Disaster and Recovery Testing (DiRT) hàng năm của chúng tôi nhằm giải quyết trực tiếp những câu hỏi này [[Kri12]](https://sre.google/sre-book/bibliography#Kri12). Trong các bài tập DiRT, các SRE đẩy các hệ thống production đến giới hạn và cố tình gây ra các sự cố ngừng hoạt động thực sự nhằm:

-   Đảm bảo các hệ thống phản ứng theo cách chúng tôi nghĩ rằng chúng sẽ phản ứng
-   Xác định các điểm yếu không lường trước
-   Tìm cách làm cho các hệ thống vững chắc hơn nhằm ngăn chặn các sự cố ngừng hoạt động không được kiểm soát

Một số chiến lược để kiểm tra khả năng sẵn sàng ứng phó thảm họa và đảm bảo sự chuẩn bị trong các ngành khác đã xuất hiện từ các cuộc trò chuyện của chúng tôi. Các chiến lược bao gồm những điều sau:

-   Sự tập trung không khoan nhượng của tổ chức vào sự an toàn
-   Sự chú ý đến từng chi tiết
-   Năng lực dự phòng (Swing capacity)
-   Mô phỏng và diễn tập thực tế (Simulations and live drills)
-   Đào tạo và chứng nhận
-   Sự tập trung ám ảnh vào việc thu thập yêu cầu chi tiết và thiết kế
-   Bảo vệ nhiều lớp và đa chiều (Defense in depth and breadth)

## Sự Tập Trung Không Khoan Nhượng của Tổ Chức vào Sự An Toàn (Relentless Organizational Focus on Safety)

Nguyên lý này đặc biệt quan trọng trong bối cảnh kỹ thuật công nghiệp. Theo Eddie Kennedy, người đã làm việc trên một sàn sản xuất nơi người lao động đối mặt với các mối nguy hiểm về an toàn, "mọi cuộc họp quản lý đều bắt đầu bằng một cuộc thảo luận về sự an toàn." Ngành công nghiệp sản xuất chuẩn bị cho những điều bất ngờ bằng cách thiết lập các quy trình được định nghĩa rõ ràng cao và được tuân thủ nghiêm ngặt ở mọi cấp độ của tổ chức. Điều hết sức quan trọng là tất cả nhân viên đều coi sự an toàn là nghiêm túc, và người lao động cảm thấy được trao quyền để lên tiếng khi bất cứ điều gì có vẻ không ổn. Trong các ngành năng lượng hạt nhân, máy bay quân sự và tín hiệu đường sắt, các tiêu chuẩn an toàn cho phần mềm được mô tả rất chi tiết (ví dụ, UK Defence Standard 00-56, IEC 61508, IEC513, US DO-178B/C, và DO-254) và các mức độ tin cậy của những hệ thống như vậy được xác định rõ ràng (ví dụ, Safety Integrity Level (SIL) 1–4),<sup>[3](#fn3)</sup> với mục tiêu là xác định các cách tiếp cận có thể chấp nhận được để cung cấp một sản phẩm.

## Sự Chú Ý Đến Từng Chi Tiết (Attention to Detail)

Từ thời gian ở trong Hải quân Hoa Kỳ, Jeff Stevenson nhớ lại sự nhạy bén cấp thiết về cách sự thiếu tận tụy trong các nhiệm vụ nhỏ (ví dụ, bảo trì dầu bôi trơn) có thể dẫn đến sự thất bại lớn của tàu ngầm. Một sai sót hoặc sai lầm rất nhỏ có thể có tác động lớn. Các hệ thống được liên kết chặt chẽ với nhau, nên một tai nạn ở một khu vực có thể ảnh hưởng đến nhiều thành phần liên quan. Hải quân hạt nhân tập trung vào bảo trì thường xuyên để đảm bảo các vấn đề nhỏ không bị leo thang.

## Năng Lực Dự Phòng (Swing Capacity)

Tỷ lệ sử dụng hệ thống trong ngành viễn thông có thể rất khó dự đoán. Năng lực tuyệt đối có thể bị căng thẳng bởi các sự kiện không thể dự đoán trước như thiên tai, cũng như các sự kiện lớn có thể dự đoán được như Thế vận hội Olympic. Theo Gus Hartmann, ngành công nghiệp xử lý những trường hợp này bằng cách triển khai năng lực dự phòng dưới hình thức một SOW (switch on wheels — máy chuyển mạch đặt trên xe), một văn phòng viễn thông di động. Năng lực dư thừa này có thể được triển khai trong một trường hợp khẩn cấp hoặc để chuẩn bị cho một sự kiện đã biết có khả năng làm quá tải hệ thống. Các vấn đề về năng lực cũng có thể đi chệch sang những điều không ngờ tới trong các tình huống không liên quan đến năng lực tuyệt đối. Ví dụ, khi số điện thoại riêng tư của một người nổi tiếng bị rò rỉ vào năm 2005 và hàng nghìn người hâm mộ đồng thời cố gọi cho cô ấy, hệ thống viễn thông đã biểu hiện các triệu chứng tương tự một cuộc tấn công DDoS hoặc một lỗi định tuyến khổng lồ.

## Mô Phỏng và Diễn Tập Thực Tế (Simulations and Live Drills)

Các bài kiểm thử Khôi phục Thảm họa (Disaster Recovery) của Google có rất nhiều điểm chung với các mô phỏng và diễn tập thực tế — trọng tâm chính của nhiều ngành công nghiệp lâu đời mà chúng tôi đã nghiên cứu. Hậu quả tiềm tàng của một sự cố ngừng hoạt động hệ thống quyết định việc dùng mô phỏng hay diễn tập thực tế là phù hợp. Ví dụ, Matthew Toia chỉ ra rằng ngành hàng không không thể thực hiện một bài kiểm thử thực tế "trong production" mà không khiến thiết bị và hành khách rơi vào nguy hiểm. Thay vào đó, họ sử dụng các máy mô phỏng cực kỳ chân thực với các luồng dữ liệu trực tiếp, trong đó các phòng điều khiển và thiết bị được mô hình hóa đến những chi tiết nhỏ nhất, đảm bảo trải nghiệm chân thực mà không đặt con người thật vào nguy hiểm. Gus Hartmann cho biết ngành viễn thông thường tập trung vào các diễn tập thực tế xoay quanh việc sống sót qua các cơn bão và các tình trạng thời tiết khẩn cấp khác. Việc mô hình hóa như vậy đã dẫn đến các cơ sở chống chịu thời tiết với máy phát điện trong tòa nhà có khả năng vận hành qua một cơn bão.

Hải quân hạt nhân Hoa Kỳ sử dụng sự pha trộn giữa các bài tập tư duy "nếu... thì sao" và các diễn tập thực tế. Theo Jeff Stevenson, các diễn tập thực tế liên quan đến "việc thực sự phá hủy những thứ thật nhưng với các tham số kiểm soát. Các diễn tập thực tế được thực hiện một cách thành tâm, mỗi tuần, hai đến ba ngày một tuần." Đối với Hải quân hạt nhân, các bài tập tư duy thì hữu ích, nhưng không đủ để chuẩn bị cho các sự cố thực sự. Các phản ứng phải được luyện tập để không bị quên đi.

Theo Mike Doherty, các cứu hộ viên đối mặt với những bài kiểm thử thảm họa mang tính chất giống trải nghiệm "khách hàng bí ẩn" (mystery shopper) hơn. Thông thường, một quản lý cơ sở làm việc với một đứa trẻ hoặc một cứu hộ viên huấn luyện đang ẩn danh để dàn dựng một vụ đuối nước giả định. Các kịch bản này được thực hiện càng chân thực càng tốt để các cứu hộ viên không thể phân biệt giữa tình huống khẩn cấp thật và dàn dựng.

## Đào Tạo và Chứng Nhận (Training and Certification)

Các cuộc phỏng vấn của chúng tôi gợi ý rằng việc đào tạo và chứng nhận đặc biệt quan trọng khi sự sống bị đe dọa. Ví dụ, Mike Doherty đã mô tả cách các cứu hộ viên hoàn thành một quy trình chứng nhận đào tạo nghiêm ngặt, bên cạnh quy trình chứng nhận lại định kỳ. Các khóa học bao gồm thành phần thể lực (ví dụ, một cứu hộ viên phải có thể giữ một người nặng hơn mình với vai nổi trên mặt nước), các thành phần kỹ thuật như sơ cứu và CPR (hồi sức tim phổi), và các yếu tố vận hành (ví dụ, nếu một cứu hộ viên xuống nước, các thành viên khác của đội phản ứng như thế nào?). Mỗi cơ sở cũng có đào tạo đặc thù theo địa điểm, vì việc cứu hộ trong hồ bơi khác biệt rõ rệt so với cứu hộ trên bãi biển ven hồ hoặc trên đại dương.

## Tập Trung vào Việc Thu thập Yêu cầu và Thiết kế Chi tiết (Focus on Detailed Requirements Gathering and Design)

Một số kỹ sư mà chúng tôi phỏng vấn đã thảo luận về tầm quan trọng của việc thu thập yêu cầu chi tiết và các tài liệu thiết kế (design docs). Thực hành này đặc biệt quan trọng khi làm việc với các thiết bị y tế. Trong nhiều trường hợp như vậy, việc sử dụng thực tế hoặc bảo trì thiết bị không nằm trong phạm vi của các nhà thiết kế sản phẩm. Do đó, các yêu cầu về sử dụng và bảo trì phải được thu thập từ các nguồn khác.

Ví dụ, theo Erik Gross, các máy phẫu thuật mắt bằng laser được thiết kế để chống sai lầm (foolproof) nhất có thể. Do đó, việc tham vấn các bác sĩ phẫu thuật thực sự sử dụng những máy này và các kỹ thuật viên chịu trách nhiệm bảo trì chúng là đặc biệt quan trọng. Trong một ví dụ khác, cựu nhà thầu quốc phòng Peter Dahl đã mô tả một nền văn hóa thiết kế rất chi tiết, trong đó việc tạo ra một hệ thống quốc phòng mới thường đòi hỏi cả một năm thiết kế, rồi chỉ ba tuần viết mã để hiện thực hóa thiết kế đó. Cả hai ví dụ này đều khác xa so với nền văn hóa ra mắt và lặp đi lặp lại (launch and iterate) của Google, thúc đẩy tốc độ thay đổi nhanh hơn nhiều ở một mức rủi ro đã được tính toán. Các ngành khác (ví dụ, ngành y tế và quân đội, như đã thảo luận trước đó) có những áp lực, khẩu vị rủi ro và yêu cầu rất khác nhau, và các quy trình của họ được định hình rất nhiều bởi những hoàn cảnh này.

## Bảo Vệ Nhiều Lớp và Đa Chiều (Defense in Depth and Breadth)

Trong ngành công nghiệp năng lượng hạt nhân, bảo vệ nhiều lớp là một yếu tố then chốt của sự chuẩn bị [[IAEA12]](https://sre.google/sre-book/bibliography#IAEA12). Các lò phản ứng hạt nhân có tính dự phòng (redundancy) trên mọi hệ thống và áp dụng một phương pháp luận thiết kế đòi hỏi các hệ thống dự phòng (fallback) nằm phía sau các hệ thống chính trong trường hợp xảy ra sự cố. Hệ thống được thiết kế với nhiều lớp bảo vệ, bao gồm cả một rào cản vật lý cuối cùng chống lại sự phát tán chất phóng xạ bao quanh chính nhà máy. Bảo vệ nhiều lớp đặc biệt quan trọng trong ngành hạt nhân do mức độ không dung lỗi cho các sự thất bại và sự cố.

## Văn Hóa Postmortem (Postmortem Culture)

Corrective and preventative action (CAPA)<sup>[4](#fn4)</sup> là một khái niệm nổi tiếng để cải thiện độ tin cậy, tập trung vào việc điều tra có hệ thống các nguyên nhân gốc rễ của các vấn đề hoặc rủi ro đã được xác định nhằm ngăn ngừa sự lặp lại. Nguyên lý này được thể hiện qua văn hóa mạnh mẽ của SRE về các postmortem không đổ lỗi (blameless). Khi có điều gì đó sai (và với quy mô, độ phức tạp và tốc độ thay đổi nhanh tại Google, điều gì đó *đáng lẽ* sẽ sai), cần đánh giá tất cả những điều sau:

-   Điều gì đã xảy ra
-   Hiệu quả của sự phản ứng
-   Lần tới chúng ta sẽ làm gì khác đi
-   Những hành động nào sẽ được thực hiện để đảm bảo một sự cố cụ thể không xảy ra lần nữa

Bài tập này được thực hiện mà không chỉ trích bất kỳ cá nhân nào. Thay vì gán trách nhiệm, quan trọng hơn nhiều là tìm ra điều gì đã sai, và như thế nào, và, như một tổ chức, chúng ta sẽ đoàn kết ra sao để đảm bảo nó không xảy ra lần nữa. Dằn vặt về *ai* có thể đã gây ra sự cố ngừng hoạt động là phản tác dụng. Các postmortem được tiến hành sau các sự cố và được công bố cho các đội SRE để tất cả có thể hưởng lợi từ những bài học đã học.

Các cuộc phỏng vấn của chúng tôi cho thấy nhiều ngành thực hiện một phiên bản của postmortem (mặc dù nhiều ngành không sử dụng danh xưng cụ thể này, vì những lý do hiển nhiên). *Động lực* đằng sau những bài tập này dường như là yếu tố khác biệt chính giữa các thực hành của các ngành.

Nhiều ngành bị kiểm soát chặt chẽ và phải chịu trách nhiệm trước các cơ quan chính phủ cụ thể khi có điều gì đó sai. Sự kiểm soát như vậy đặc biệt ăn sâu khi mức độ rủi ro của sự thất bại là cao (ví dụ, sự sống bị đe dọa). Các cơ quan chính phủ liên quan bao gồm FCC (viễn thông), FAA (hàng không), OSHA (các ngành công nghiệp sản xuất và hóa chất), FDA (thiết bị y tế), và các National Competent Authorities khác nhau trong EU.<sup>[5](#fn5)</sup> Các ngành công nghiệp năng lượng hạt nhân và giao thông vận tải cũng bị kiểm soát chặt chẽ.

Các cân nhắc về an toàn là một yếu tố động lực khác đằng sau các postmortem. Trong các ngành công nghiệp sản xuất và hóa chất, rủi ro bị thương hoặc tử vong luôn hiện hữu do bản chất của các điều kiện được yêu cầu để tạo ra sản phẩm cuối cùng (nhiệt độ cao, áp suất, tính độc hại, và tính ăn mòn, để nêu tên một vài cái). Ví dụ, Alcoa có một nền văn hóa an toàn đáng chú ý. Cựu CEO Paul O'Neill đã yêu cầu nhân viên thông báo cho ông trong vòng 24 giờ bất kỳ chấn thương nào khiến một công nhân mất ngày công. Ông thậm chí còn phân phát số điện thoại nhà của mình cho người lao động trên sàn nhà máy để họ có thể cá nhân thông báo cho ông về các mối lo ngại an toàn.<sup>[6](#fn6)</sup>

Mức độ rủi ro trong các ngành công nghiệp sản xuất và hóa chất cao đến mức ngay cả các "near miss" (sự suýt gặp nạn) — khi một sự kiện cụ thể có thể gây ra tổn hại nghiêm trọng nhưng thực tế không xảy ra — cũng được xem xét kỹ lưỡng. Các kịch bản này hoạt động như một loại postmortem mang tính phòng ngừa. Theo VM Brasseur trong một bài thuyết trình tại YAPC NA 2015, "Có rất nhiều near miss trong hầu như mọi thảm họa và khủng hoảng kinh doanh, và thường chúng bị bỏ qua vào thời điểm chúng xảy ra. Lỗi tiềm ẩn, cộng với một điều kiện cho phép, bằng với các thứ không hoạt động đúng như bạn đã dự định" [[Bra15]](https://sre.google/sre-book/bibliography#Bra15). Các near miss thực chất là những thảm họa đang chờ xảy ra. Ví dụ, các kịch bản trong đó một công nhân không tuân theo quy trình vận hành chuẩn, một nhân viên né tránh vào giây cuối cùng để tránh một vũng chất lỏng bắn tung tóe, hoặc một vũng chất lỏng đổ trên cầu thang không được dọn sạch, tất cả đều đại diện cho các near miss và những cơ hội để học hỏi và cải thiện. Lần tới, nhân viên và công ty có thể sẽ không may mắn như vậy. CHIRP của Vương quốc Anh (Confidential Reporting Programme for Aviation and Maritime — Chương trình Báo cáo Mật cho Hàng không và Hàng hải) tìm cách nâng cao nhận thức về những sự cố như vậy trên toàn ngành bằng cách cung cấp một điểm báo cáo trung tâm, nơi nhân viên hàng không và hàng hải có thể báo cáo các near miss một cách kín đáo. Các báo cáo và phân tích về những near miss này sau đó được công bố trong các bản tin định kỳ.

Cứu hộ biển có một nền văn hóa phân tích sau sự cố và lập kế hoạch hành động ăn sâu. Mike Doherty dí dỏm, "Nếu chân của một cứu hộ viên chạm vào nước, sẽ có giấy tờ!" Một bản ghi chép chi tiết được yêu cầu sau bất kỳ sự cố nào ở hồ bơi hoặc trên bãi biển. Trong trường hợp các sự cố nghiêm trọng, cả đội cùng nhau xem xét sự cố từ đầu đến cuối, thảo luận điều gì đã đúng và điều gì đã sai. Sau đó, các thay đổi vận hành được thực hiện dựa trên những phát hiện này, và việc đào tạo thường được lên lịch để giúp mọi người xây dựng sự tự tin về khả năng xử lý một sự cố tương tự trong tương lai. Trong các trường hợp sự cố đặc biệt gây sốc hoặc sang chấn, một chuyên viên tư vấn được mời đến hiện trường để giúp nhân viên đối phó với hậu quả tâm lý. Các cứu hộ viên có thể đã chuẩn bị tốt cho những gì xảy ra trên thực tế, nhưng có thể *cảm thấy* như họ đã không hoàn thành công việc một cách đầy đủ. Giống như Google, cứu hộ biển chào đón một nền văn hóa phân tích sự cố không đổ lỗi. Các sự cố là hỗn loạn, và nhiều yếu tố góp phần vào bất kỳ sự cố cụ thể nào. Trong lĩnh vực này, việc gán trách nhiệm cho một cá nhân đơn lẻ không hữu ích.

## Loại Bỏ Công Việc Lặp đi Lặp lại và Gánh Nặng Vận hành Bằng Tự Động Hóa (Automating Away Repetitive Work and Operational Overhead)

Ở cốt lõi, các Site Reliability Engineer của Google là những kỹ sư phần mềm có mức độ dung nạp thấp đối với công việc phản ứng lặp đi lặp lại. Văn hóa của chúng tôi thấm nhuần việc tránh lặp lại một phép vận hành không tạo thêm giá trị cho một dịch vụ. Nếu một tác vụ có thể được loại bỏ bằng tự động hóa, thì tại sao bạn lại vận hành một hệ thống dựa trên công việc lặp đi lặp lại có giá trị thấp? Tự động hóa làm giảm gánh nặng vận hành và giải phóng thời gian để kỹ sư của chúng tôi chủ động đánh giá và cải thiện các dịch vụ mà họ hỗ trợ.

Các ngành mà chúng tôi đã khảo sát có những quan điểm khác nhau về việc họ có, như thế nào và vì sao chấp nhận tự động hóa. Một số ngành tin vào con người hơn là vào máy móc. Trong suốt nhiệm kỳ của chuyên gia lâu năm trong ngành của chúng tôi, Hải quân hạt nhân Hoa Kỳ đã tránh tự động hóa, ưu tiên một loạt các khóa liên động (interlock) và các quy trình hành chính. Ví dụ, theo Jeff Stevenson, việc vận hành một van yêu cầu một người vận hành, một cấp trên, và một thành viên tổ đang nói chuyện qua điện thoại với sĩ quan trực kỹ thuật (engineering watch officer) được giao nhiệm vụ theo dõi phản ứng với hành động đã thực hiện. Các phép vận hành này rất thủ công do lo ngại rằng một hệ thống tự động có thể không phát hiện ra một vấn đề mà một con người chắc chắn sẽ nhận ra. Các phép vận hành trên một tàu ngầm được chi phối bởi một chuỗi quyết định của con người đáng tin cậy — một *chuỗi* con người, chứ không phải một cá nhân duy nhất. Hải quân hạt nhân cũng lo ngại rằng tự động hóa và máy tính vận hành nhanh đến mức hoàn toàn có khả năng phạm phải một sai lầm lớn, không thể sửa chữa. Khi bạn đang đối phó với các lò phản ứng hạt nhân, một phương pháp tiếp cận chậm và kiên nhẫn, có phương pháp, quan trọng hơn so với việc hoàn thành một tác vụ nhanh chóng.

Theo John Li, ngành công nghiệp môi giới giao dịch (proprietary trading) đã trở nên ngày càng thận trọng trong việc áp dụng tự động hóa trong những năm gần đây. Kinh nghiệm đã cho thấy rằng tự động hóa được cấu hình sai có thể gây ra thiệt hại đáng kể, dẫn đến tổn thất tài chính lớn trong một khoảng thời gian rất ngắn. Ví dụ, vào năm 2012, Knight Capital Group đã gặp phải một "lỗi phần mềm" (software glitch) dẫn đến tổn thất 440 triệu đô la trong chỉ vài giờ.<sup>[7](#fn7)</sup> Tương tự, vào năm 2010, thị trường chứng khoán Hoa Kỳ đã trải qua một vụ Flash Crash (sụp đổ nhanh), cuối cùng bị quy cho một nhà giao dịch phi pháp (rogue trader) cố gắng thao túng thị trường bằng các phương tiện tự động. Mặc dù thị trường đã phục hồi nhanh chóng, vụ Flash Crash đã dẫn đến tổn thất ở quy mô hàng nghìn tỷ đô la trong chỉ *30 phút*.<sup>[8](#fn8)</sup> Máy tính có thể thực hiện các tác vụ rất nhanh, và tốc độ có thể trở thành điều tiêu cực nếu những tác vụ này được cấu hình sai.

Ngược lại, một số công ty chấp nhận tự động hóa chính xác *vì* máy tính hành động nhanh hơn con người. Theo Eddie Kennedy, hiệu quả và tiết kiệm tiền bạc là then chốt trong ngành công nghiệp sản xuất, và tự động hóa cung cấp một phương tiện để hoàn thành các tác vụ hiệu quả và hiệu quả về chi phí hơn. Hơn nữa, tự động hóa nhìn chung đáng tin cậy và có thể lặp lại hơn so với công việc được thực hiện thủ công bởi con người, điều này có nghĩa là nó tạo ra các tiêu chuẩn chất lượng cao hơn và các dung sai (tolerance) chặt chẽ hơn. Dan Sheridan đã thảo luận về tự động hóa như được triển khai trong ngành công nghiệp hạt nhân của Anh. Tại đây, một quy tắc kinh nghiệm cho rằng nếu một nhà máy được yêu cầu phản ứng với một tình huống cụ thể trong ít hơn 30 phút, thì phản ứng đó phải được tự động hóa.

Theo kinh nghiệm của Matt Toia, ngành hàng không áp dụng tự động hóa một cách chọn lọc. Ví dụ, việc failover vận hành được thực hiện tự động, nhưng với một số tác vụ khác, ngành công nghiệp chỉ tin tưởng vào tự động hóa khi nó được xác minh bởi một con người. Trong khi ngành công nghiệp sử dụng rất nhiều giám sát tự động, các triển khai thực tế của hệ thống kiểm soát không lưu phải được con người kiểm tra thủ công.

Theo Erik Gross, tự động hóa đã khá hiệu quả trong việc giảm lỗi người dùng trong phẫu thuật mắt bằng laser. Trước khi phẫu thuật LASIK được thực hiện, bác sĩ đo bệnh nhân bằng một bài kiểm tra khúc xạ mắt. Ban đầu, bác sĩ sẽ gõ các con số và nhấn một nút, và laser sẽ hoạt động để sửa thị lực của bệnh nhân. Tuy nhiên, các lỗi nhập dữ liệu có thể là một vấn đề lớn. Quy trình này cũng bao gồm khả năng nhầm lẫn dữ liệu bệnh nhân hoặc đảo lộn các con số cho mắt trái và mắt phải.

Tự động hóa hiện giảm đáng kể khả năng con người mắc phải một sai lầm ảnh hưởng đến thị lực của ai đó. Một bước kiểm tra tính hợp lý (sanity check) bằng máy tính của dữ liệu nhập thủ công là cải tiến tự động hóa lớn đầu tiên: nếu một người vận hành nhập các phép đo bên ngoài một phạm vi được kỳ vọng, tự động hóa sẽ nhanh chóng và nổi bật gắn cờ trường hợp này là bất thường. Các cải tiến tự động hóa khác đã theo sau sự phát triển này: giờ đây, mống mắt được chụp ảnh trong quá trình kiểm tra khúc xạ mắt sơ bộ. Khi đến thời điểm thực hiện ca phẫu thuật, mống mắt của bệnh nhân được tự động khớp với mống mắt trong ảnh, do đó loại bỏ khả năng nhầm lẫn dữ liệu bệnh nhân. Khi giải pháp tự động hóa này được triển khai, cả một lớp lỗi y tế đã biến mất.

## Ra Quyết Định Có Cấu Trúc và Hợp Lý (Structured and Rational Decision Making)

Tại Google nói chung, và cụ thể hơn trong Site Reliability Engineering, dữ liệu là yếu tố then chốt. Đội ngũ hướng đến việc ra quyết định có cấu trúc và hợp lý bằng cách đảm bảo rằng:

-   Cơ sở cho quyết định được thống nhất từ trước, thay vì được biện minh một cách ex post facto (sau khi sự việc đã xảy ra)
-   Các đầu vào cho quyết định là rõ ràng
-   Mọi giả định đều được nêu rõ
-   Những quyết định dựa trên dữ liệu thắng những quyết định dựa trên cảm xúc, trực giác, hoặc ý kiến của nhân viên cấp cao nhất trong phòng

Google SRE vận hành dựa trên giả định nền tảng rằng mọi người trong đội:

-   Luôn đặt lợi ích tốt nhất của người dùng dịch vụ lên hàng đầu
-   Có thể tìm ra cách tiếp tục dựa trên dữ liệu có sẵn

Các quyết định nên được định hướng bởi thông tin chứ không mang tính chỉ đạo (prescriptive), và được đưa ra mà không nể nang các ý kiến cá nhân — ngay cả ý kiến của người có cấp cao nhất trong phòng, người mà Eric Schmidt và Jonathan Rosenberg gọi là "HiPPO," viết tắt của "Highest-Paid Person's Opinion" (Ý kiến của Người Được Trả Lương Cao nhất) [[Sch14]](https://sre.google/sre-book/bibliography#Sch14).

Việc ra quyết định trong các ngành khác nhau rất đa dạng. Chúng tôi nhận thấy một số ngành sử dụng cách tiếp cận *nếu nó chưa hỏng, đừng sửa nó... bao giờ*. Các ngành có các hệ thống mà thiết kế của chúng đòi hỏi nhiều suy nghĩ và nỗ lực thường đặc trưng bởi sự miễn cưỡng thay đổi công nghệ nền tảng. Ví dụ, ngành viễn thông vẫn sử dụng các máy chuyển mạch đường dài được triển khai vào những năm 1980. Tại sao họ lại dựa vào công nghệ phát triển từ vài thập kỷ trước? Theo Gus Hartmann, những máy chuyển mạch này "gần như bất khả xâm phạm và dự phòng một cách mạnh mẽ." Như Dan Sheridan đã báo cáo, ngành công nghiệp hạt nhân cũng chậm thay đổi một cách tương tự. Mọi quyết định đều được nâng đỡ bởi suy nghĩ: *nếu nó hoạt động bây giờ, đừng thay đổi nó*.

Nhiều ngành tập trung mạnh vào các playbook (sổ tay sách lược) và quy trình thay vì giải quyết vấn đề mở. Mọi kịch bản có thể xảy ra đều được ghi lại trong một checklist hoặc trong "the binder" (sổ tay tổng hợp). Khi có điều gì đó sai, nguồn tài nguyên này là nguồn có thẩm quyền cho cách phản ứng. Cách tiếp cận mang tính chỉ đạo này phù hợp với các ngành tiến hóa và phát triển tương đối chậm, vì các kịch bản về những điều có thể xảy ra sai không liên tục thay đổi do các bản cập nhật hoặc thay đổi hệ thống. Cách tiếp cận này cũng phổ biến trong các ngành mà trình độ kỹ năng của người lao động có thể bị hạn chế, và cách tốt nhất để đảm bảo mọi người sẽ phản ứng phù hợp trong một trường hợp khẩn cấp là cung cấp một bộ hướng dẫn đơn giản, rõ ràng.

Các ngành khác cũng có một cách tiếp cận rõ ràng, dựa trên dữ liệu, cho việc ra quyết định. Theo kinh nghiệm của Eddie Kennedy, các môi trường nghiên cứu và sản xuất được đặc trưng bởi một nền văn hóa thực nghiệm nghiêm ngặt, phụ thuộc rất nhiều vào việc xây dựng và kiểm thử các giả thuyết. Các ngành này thường xuyên tiến hành các thí nghiệm có kiểm soát để đảm bảo rằng một thay đổi cụ thể mang lại kết quả được kỳ vọng ở mức có ý nghĩa thống kê và rằng không có điều gì bất ngờ xảy ra. Các thay đổi chỉ được triển khai khi dữ liệu do thí nghiệm tạo ra ủng hộ quyết định.

Cuối cùng, một số ngành, như môi giới giao dịch, chia nhỏ việc ra quyết định để quản lý rủi ro tốt hơn. Theo John Li, ngành công nghiệp này có một đội thực thi (enforcement team) tách biệt khỏi các nhà giao dịch để đảm bảo không có rủi ro quá mức nào được chấp nhận trong khi theo đuổi lợi nhuận. Đội thực thi chịu trách nhiệm theo dõi các sự kiện trên sàn và dừng giao dịch nếu các sự kiện mất kiểm soát. Nếu một bất thường của hệ thống xảy ra, phản ứng đầu tiên của đội thực thi là tắt hệ thống. Như John Li nói, "Nếu chúng ta không giao dịch, chúng ta không mất tiền. Chúng ta cũng không kiếm được tiền, nhưng ít nhất chúng ta không mất tiền." Chỉ đội thực thi mới có thể khởi động lại hệ thống, bất kể sự chậm trễ có thể đau đớn đến mức nào đối với các nhà giao dịch đang bỏ lỡ một cơ hội có thể có lợi nhuận.

## Kết Luận (Conclusions)

Nhiều nguyên lý cốt lõi của Site Reliability Engineering tại Google được thể hiện rõ ràng trên phạm vi rộng các ngành. Những bài học đã được các ngành công nghiệp lâu đời tích lũy có lẽ đã truyền cảm hứng cho một số thực hành đang được sử dụng tại Google ngày nay.

Một điểm chính thu được từ cuộc khảo sát liên ngành của chúng tôi là trong nhiều mảng doanh nghiệp phần mềm, Google có khẩu vị cao hơn cho tốc độ (velocity) so với các nhà chơi trong hầu hết các ngành khác. Khả năng di chuyển hoặc thay đổi nhanh chóng phải được cân nhắc với những hệ quả khác nhau của một sự thất bại. Trong ngành công nghiệp hạt nhân, hàng không, hoặc y tế, ví dụ, con người có thể bị thương hoặc thậm chí tử vong trong trường hợp xảy ra một sự cố ngừng hoạt động. Khi mức độ rủi ro là cao, một cách tiếp cận thận trọng để đạt được độ tin cậy cao là xứng đáng.

Tại Google, chúng tôi liên tục đi trên một sợi dây căng giữa kỳ vọng của người dùng về độ tin cậy cao và sự tập trung sắc bén như laser vào tốc độ thay đổi và đổi mới nhanh chóng. Mặc dù Google rất nghiêm túc về độ tin cậy, chúng tôi phải thích ứng các cách tiếp cận của mình với tốc độ thay đổi cao. Như đã thảo luận trong các chương trước, nhiều doanh nghiệp phần mềm của chúng tôi như Search đưa ra những quyết định có ý thức về việc "đáng tin cậy đủ" thực sự là bao nhiêu.

Google có sự linh hoạt đó trong hầu hết các sản phẩm và dịch vụ phần mềm của chúng tôi, những thứ vận hành trong môi trường mà sự sống không bị đe dọa trực tiếp nếu có điều gì đó sai. Do đó, chúng tôi có khả năng sử dụng các công cụ như error budget (ngân sách lỗi) ([Motivation for Error Budgets](https://sre.google/sre-book/embracing-risk#xref_risk-management_unreliability-budgets)) như một phương tiện để "tài trợ" cho một nền văn hóa đổi mới và chấp nhận rủi ro đã được tính toán. Về bản chất, Google đã thích ứng các nguyên lý độ tin cậy đã biết — mà trong nhiều trường hợp đã được phát triển và mài giũa trong các ngành khác — để tạo ra nền văn hóa độ tin cậy độc đáo của riêng mình, một nền văn hóa giải quyết phương trình phức tạp cân bằng giữa quy mô, độ phức tạp và tốc độ với độ tin cậy cao.

<a id="fn1"></a>[1](#fn1) E911 (Enhanced 911): Đường dây phản ứng khẩn cấp tại Hoa Kỳ sử dụng dữ liệu vị trí.
<a id="fn2"></a>[2](#fn2) Electrocardiogram readings (Các số đo điện tâm đồ): [*https://en.wikipedia.org/wiki/Electrocardiography*](https://en.wikipedia.org/wiki/Electrocardiography).
<a id="fn3"></a>[3](#fn3) [*https://en.wikipedia.org/wiki/Safety_integrity_level*](https://en.wikipedia.org/wiki/Safety_integrity_level)
<a id="fn4"></a>[4](#fn4) [*https://en.wikipedia.org/wiki/Corrective_and_preventive_action*](https://en.wikipedia.org/wiki/Corrective_and_preventive_action)
<a id="fn5"></a>[5](#fn5) [*https://en.wikipedia.org/wiki/Competent_authority*](https://en.wikipedia.org/wiki/Competent_authority)
<a id="fn6"></a>[6](#fn6) [*https://ehstoday.com/safety/nsc-2013-oneill-exemplifies-safety-leadership*](https://ehstoday.com/safety/nsc-2013-oneill-exemplifies-safety-leadership).
<a id="fn7"></a>[7](#fn7) Xem "FACTS, Section B" để thảo luận về phần mềm của Knight và Power Peg trong [[Sec13]](https://sre.google/sre-book/bibliography#Sec13).
<a id="fn8"></a>[8](#fn8) "Regulators blame computer algorithm for stock market 'flash crash'," Computerworld, [*https://www.computerworld.com/article/2516076/financial-it/regulators-blame-computer-algorithm-for-stock-market—flash-crash-.html*](https://www.computerworld.com/article/2516076/financial-it/regulators-blame-computer-algorithm-for-stock-market—flash-crash-.html).

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
