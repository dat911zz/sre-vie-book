> **Nguyên bản:** [Chapter 33 - Lessons Learned from Other Industries](https://sre.google/sre-book/lessons-learned/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

## Chương 33. Bài Học Rút Ra Từ Các Ngành Khác (Lessons Learned from Other Industries)

Tác giả: Jennifer Petoff  
Biên tập: Betsy Beyer

Khi tìm hiểu sâu về văn hóa và thực hành SRE tại Google, câu hỏi tự nhiên nảy sinh là: các ngành khác quản lý độ tin cậy trong doanh nghiệp của họ như thế nào? Trong quá trình biên soạn cuốn sách về Google SRE này, chúng tôi đã trao đổi với một số kỹ sư của Google về kinh nghiệm làm việc trước đây của họ ở nhiều lĩnh vực độ tin cậy cao khác nhau, nhằm giải quyết các câu hỏi so sánh sau:

-   Các nguyên lý của Site Reliability Engineering có thực sự quan trọng cả bên ngoài Google không, hay các ngành khác lại giải quyết yêu cầu về độ tin cậy cao theo những cách hoàn toàn khác?
-   Nếu các ngành khác cũng tuân theo các nguyên lý SRE, các nguyên lý ấy được thể hiện ra sao?
-   Những điểm giống nhau và khác nhau trong việc triển khai các nguyên lý này giữa các ngành là gì?
-   Những yếu tố nào thúc đẩy sự giống nhau và khác nhau trong triển khai?
-   Google và ngành công nghệ có thể học được gì từ những sự so sánh này?

Cuốn sách này xuyên suốt bàn về một số nguyên lý nền tảng của Site Reliability Engineering tại Google. Để việc so sánh các thực hành tốt nhất từ những ngành khác được đơn giản hơn, chúng tôi đã cô đọng các khái niệm này thành bốn chủ đề chính:

-   Chuẩn bị và Kiểm thử Thảm họa (Preparedness and Disaster Testing)

-   Văn hóa Postmortem (Postmortem Culture)

-   Tự động hóa và Giảm gánh nặng Vận hành (Automation and Reduced Operational Overhead)
-   Ra quyết định Có cấu trúc và Hợp lý (Structured and Rational Decision Making)

Chương này giới thiệu các lĩnh vực mà chúng tôi đã nghiên cứu cùng những chuyên gia dày dạn kinh nghiệm trong ngành mà chúng tôi đã phỏng vấn. Chúng tôi xác định các chủ đề SRE cốt lõi, thảo luận cách triển khai chúng tại Google, đồng thời đưa ra ví dụ về việc các nguyên lý này được áp dụng ở những ngành khác để so sánh. Cuối chương, chúng tôi nêu một số nhận xét và bàn về các pattern và anti-pattern đã phát hiện được.

## Làm Quen Với Các Chuyên Gia Lâu Năm Trong Ngành (Meet Our Industry Veterans)

**Peter Dahl** là một Principal Engineer tại Google. Trước đó, ông từng làm việc cho một nhà thầu quốc phòng, phụ trách một số hệ thống độ tin cậy cao, trong đó có nhiều hệ thống dẫn đường GPS và quán tính (inertial guidance) trên máy bay cũng như các phương tiện có bánh. Nếu những hệ thống này xảy ra sự suy giảm độ tin cậy, hậu quả có thể là sự trục trặc hoặc mất phương tiện, kèm theo những thiệt hại tài chính đi kèm.

**Mike Doherty** là một kỹ sư SRE tại Google. Trước đó, ông đã làm việc mười năm với vai trò cứu hộ viên và huấn luyện viên cứu hộ viên tại Canada. Đặc thù của lĩnh vực này đòi hỏi độ tin cậy tuyệt đối, bởi mỗi ngày sự sống đều được đặt lên bàn cân.

**Erik Gross** hiện là kỹ sư phần mềm tại Google. Trước khi về đầu quân cho công ty, ông đã dành bảy năm thiết kế thuật toán và mã cho các laser cùng hệ thống dùng trong phẫu thuật khúc xạ mắt (ví dụ, LASIK). Đây là lĩnh vực rủi ro cao, đòi hỏi độ tin cậy cao, nơi nhiều bài học về độ tin cậy — trước các quy định của chính phủ và rủi ro cho con người — đã được tích lũy trong quá trình công nghệ này được FDA phê duyệt, dần hoàn thiện và cuối cùng trở nên phổ biến.

**Gus Hartmann** và **Kevin Greer** từng làm việc trong ngành viễn thông, bao gồm cả việc vận hành hệ thống phản ứng khẩn cấp E911.<sup>[1](#fn1)</sup> Hiện tại, Kevin là kỹ sư phần mềm thuộc nhóm Google Chrome, còn Gus đảm nhận vai trò kỹ sư hệ thống cho nhóm Corporate Engineering của Google. Ngành viễn thông đặt ra yêu cầu độ tin cậy rất cao đối với người dùng. Hậu quả của một sự suy giảm dịch vụ có thể dao động từ sự bất tiện do một sự cố ngừng dịch vụ (outage) cho đến nguy cơ tử vong nếu E911 gặp sự cố.

**Ron Heiby** là Technical Program Manager phụ trách Site Reliability Engineering tại Google. Ron từng có kinh nghiệm phát triển cho điện thoại di động, thiết bị y tế và ngành công nghiệp ô tô. Trong một số trường hợp, ông đã làm việc trên các thành phần giao diện của những ngành này (ví dụ, trên một thiết bị cho phép các số đo EKG được truyền đi qua mạng điện thoại không dây kỹ thuật số).<sup>[2](#fn2)</sup> Trong những ngành này, tác động của một vấn đề độ tin cậy có thể trải dài từ thiệt hại cho doanh nghiệp do việc thu hồi thiết bị cho đến gián tiếp ảnh hưởng đến sự sống và sức khỏe (ví dụ, mọi người không nhận được sự chăm sóc y tế cần thiết nếu hệ thống EKG không thể truyền thông với bệnh viện).

**Adrian Hilton** là kỹ sư điều phối triển khai tại Google. Trước đó, ông từng làm việc với các máy bay quân sự của Anh và Mỹ, hệ thống điện tử hàng hải, hệ thống quản lý vũ khí trên máy bay cùng hệ thống tín hiệu đường sắt của Anh. Trong lĩnh vực này, độ tin cậy là yếu tố sống còn, bởi hậu quả của các sự cố có thể kéo dài từ việc mất thiết bị trị giá hàng triệu đô la cho đến chấn thương và tử vong.

**Eddie Kennedy** là quản lý dự án cho nhóm Global Customer Experience tại Google, đồng thời có nền tảng kỹ sư cơ khí. Trước đó, Eddie đã dành sáu năm làm kỹ sư quy trình Six Sigma Black Belt tại một cơ sở sản xuất kim cương tổng hợp. Ngành này đòi hỏi sự tập trung tuyệt đối vào an toàn, bởi các điều kiện nhiệt độ và áp suất khắc nghiệt trong quy trình luôn đặt người lao động vào mức nguy hiểm cao mỗi ngày.

**John Li** hiện là một Site Reliability Engineer tại Google. John trước đó đã làm việc với tư cách là quản trị viên hệ thống và nhà phát triển phần mềm tại một công ty môi giới giao dịch (proprietary trading) trong ngành tài chính. Các vấn đề độ tin cậy trong khu vực tài chính được xem xét hết sức nghiêm túc vì chúng có thể dẫn đến những hậu quả tài chính nghiêm trọng.

**Dan Sheridan** là một Site Reliability Engineer tại Google. Trước khi gia nhập công ty, ông đã làm việc với tư cách là một cố vấn an toàn trong ngành công nghiệp hạt nhân dân sự tại Anh. Độ tin cậy là quan trọng trong ngành hạt nhân vì một sự cố có thể có những tác động nghiêm trọng: các sự cố ngừng hoạt động có thể gây ra mất doanh thu hàng triệu đô la mỗi ngày, trong khi rủi ro cho người lao động và những người trong cộng đồng còn nghiêm trọng hơn, đòi hỏi mức độ không dung lỗi nào cho sự cố. Hạ tầng hạt nhân được thiết kế với một loạt các cơ chế dự phòng (failsafe) sẽ dừng hoạt động trước khi một sự cố đạt đến quy mô nghiêm trọng như vậy.

**Jeff Stevenson** hiện là quản lý vận hành phần cứng tại Google. Trước đó, ông từng là kỹ sư hạt nhân trong Hải quân Hoa Kỳ, phục vụ trên một con tàu ngầm. Trong lĩnh vực hải quân hạt nhân, mức độ rủi ro về độ tin cậy rất cao — khi xảy ra sự cố, hậu quả có thể bao gồm thiết bị hư hại, tác động môi trường kéo dài, thậm chí là mất mạng người.

**Matthew Toia** là một Site Reliability Manager chuyên về các hệ thống lưu trữ. Trước khi gia nhập Google, ông từng tham gia phát triển và triển khai phần mềm cho các hệ thống kiểm soát không lưu (air traffic control). Hậu quả từ các sự cố trong lĩnh vực này có thể chỉ là sự bất tiện cho hành khách và các hãng hàng không (chẳng hạn như chuyến bay bị hoãn, máy bay phải đổi hướng), nhưng cũng có thể dẫn đến mất mạng người nếu xảy ra tai nạn. Chiến lược phòng thủ nhiều lớp (defense in depth) đóng vai trò then chốt nhằm ngăn chặn những sự cố thảm khốc.

Giờ đây, sau khi đã trao đổi với các chuyên gia của chúng tôi và nắm được bức tranh tổng quan về tầm quan trọng của độ tin cậy trong từng lĩnh vực trước đây của họ, chúng ta sẽ đi sâu vào bốn chủ đề chính về độ tin cậy.

## Chuẩn Bị và Kiểm thử Thảm họa (Preparedness and Disaster Testing)

“Hy vọng không phải là một chiến lược.” Câu khẩu hiệu này của đội SRE tại Google nói lên đúng tinh thần mà chúng tôi muốn truyền tải về việc chuẩn bị và kiểm thử thảm họa. Văn hóa SRE luôn đề cao sự cảnh giác và thói quen tự chất vấn: điều gì có thể xảy ra sai? Chúng tôi có thể thực hiện hành động nào để giải quyết những vấn đề đó trước khi chúng dẫn đến một sự cố ngừng hoạt động hoặc mất dữ liệu? Các cuộc diễn tập Disaster and Recovery Testing (DiRT) hàng năm của chúng tôi nhằm giải quyết trực tiếp những câu hỏi này [[Kri12]](https://sre.google/sre-book/bibliography#Kri12). Trong các bài tập DiRT, các SRE đẩy các hệ thống production đến giới hạn và cố tình gây ra các sự cố ngừng hoạt động thực sự nhằm:

-   Đảm bảo các hệ thống phản ứng theo cách chúng tôi nghĩ rằng chúng sẽ phản ứng
-   Xác định các điểm yếu không lường trước
-   Tìm cách làm cho các hệ thống vững chắc hơn nhằm ngăn chặn các sự cố ngừng hoạt động không được kiểm soát

Từ các cuộc trò chuyện của chúng tôi, một số chiến lược nhằm kiểm tra khả năng ứng phó thảm họa và đảm bảo sự chuẩn bị trong các ngành khác đã được rút ra. Các chiến lược này bao gồm:

-   Sự tập trung không khoan nhượng của tổ chức vào sự an toàn
-   Sự chú ý đến từng chi tiết
-   Năng lực dự phòng (Swing capacity)
-   Mô phỏng và diễn tập thực tế (Simulations and live drills)
-   Đào tạo và chứng nhận
-   Sự tập trung ám ảnh vào việc thu thập yêu cầu chi tiết và thiết kế
-   Phòng thủ nhiều lớp và đa chiều (Defense in depth and breadth)

## Sự Tập Trung Không Khoan Nhượng của Tổ Chức vào Sự An Toàn (Relentless Organizational Focus on Safety)

Nguyên lý này đặc biệt quan trọng trong bối cảnh kỹ thuật công nghiệp. Theo Eddie Kennedy, người từng làm việc tại một nhà máy nơi công nhân đối mặt với nhiều mối nguy hiểm về an toàn, “mọi cuộc họp quản lý đều bắt đầu bằng phần thảo luận về an toàn.” Ngành sản xuất ứng phó với những tình huống bất ngờ bằng cách xây dựng các quy trình được định nghĩa rõ ràng và tuân thủ nghiêm ngặt ở mọi cấp độ tổ chức. Điều then chốt là tất cả nhân viên đều coi trọng an toàn, và người lao động cảm thấy được trao quyền để lên tiếng khi phát hiện điều gì đó bất thường. Trong các lĩnh vực như năng lượng hạt nhân, máy bay quân sự và tín hiệu đường sắt, các tiêu chuẩn an toàn phần mềm được mô tả rất chi tiết (ví dụ: UK Defence Standard 00-56, IEC 61508, IEC513, US DO-178B/C, và DO-254), đồng thời các mức độ tin cậy của những hệ thống này cũng được xác định rõ ràng (ví dụ: Safety Integrity Level (SIL) 1–4),<sup>[3](#fn3)</sup> nhằm xác định các phương pháp tiếp cận chấp nhận được để cung cấp sản phẩm.

## Sự Chú Ý Đến Từng Chi Tiết (Attention to Detail)

Thời gian phục vụ trong Hải quân Hoa Kỳ đã giúp Jeff Stevenson nhận ra mối liên hệ mật thiết giữa những nhiệm vụ nhỏ và các sự cố nghiêm trọng. Ông nhớ lại cách việc thiếu sự tận tâm trong các công việc đơn giản, chẳng hạn như bảo trì dầu bôi trơn, có thể dẫn đến sự cố lớn trên tàu ngầm. Chỉ một sai sót hay sai lầm nhỏ cũng có thể gây ra hậu quả nghiêm trọng. Do các hệ thống được liên kết chặt chẽ, một tai nạn ở một khu vực có thể ảnh hưởng đến nhiều thành phần liên quan. Vì vậy, Hải quân hạt nhân tập trung vào bảo trì thường xuyên để đảm bảo các vấn đề nhỏ không bị leo thang.

## Năng Lực Dự Phòng (Swing Capacity)

Trong ngành viễn thông, việc dự đoán tỷ lệ sử dụng hệ thống có thể vô cùng khó khăn. Năng lực tuyệt đối có thể bị quá tải do những sự kiện bất ngờ như thiên tai, hay những sự kiện lớn đã được lên kế hoạch như Thế vận hội Olympic. Theo Gus Hartmann, ngành công nghiệp thường ứng phó bằng cách triển khai năng lực dự phòng dưới dạng SOW (switch on wheels — máy chuyển mạch đặt trên xe), tức là một văn phòng viễn thông di động. Nguồn lực dự phòng này có thể được huy động trong trường hợp khẩn cấp hoặc để chuẩn bị cho các sự kiện đã biết có nguy cơ làm quá tải hệ thống. Các vấn đề về năng lực cũng có thể phát sinh theo những hướng bất ngờ, ngay cả trong các tình huống không liên quan đến giới hạn năng lực tuyệt đối. Chẳng hạn, vào năm 2005, khi số điện thoại cá nhân của một người nổi tiếng bị lộ và hàng nghìn người hâm mộ cùng lúc gọi đến, hệ thống viễn thông đã xuất hiện các dấu hiệu tương tự như một cuộc tấn công DDoS hoặc một lỗi định tuyến nghiêm trọng.

## Mô Phỏng và Diễn Tập Thực Tế (Simulations and Live Drills)

Các bài kiểm thử Khôi phục Thảm họa (Disaster Recovery) của Google có nhiều điểm chung với các mô phỏng và diễn tập thực tế — trọng tâm chính của nhiều ngành công nghiệp lâu đời mà chúng tôi đã nghiên cứu. Hậu quả tiềm tàng của một sự cố ngừng dịch vụ quyết định việc dùng mô phỏng hay diễn tập thực tế là phù hợp. Ví dụ, Matthew Toia chỉ ra rằng ngành hàng không không thể thực hiện một bài kiểm thử thực tế trong production mà không khiến thiết bị và hành khách rơi vào nguy hiểm. Thay vào đó, họ sử dụng các máy mô phỏng cực kỳ chân thực với các luồng dữ liệu trực tiếp, trong đó các phòng điều khiển và thiết bị được mô hình hóa đến những chi tiết nhỏ nhất, đảm bảo trải nghiệm chân thực mà không đặt con người thật vào nguy hiểm. Gus Hartmann cho biết ngành viễn thông thường tập trung vào các diễn tập thực tế xoay quanh việc sống sót qua các cơn bão và các tình trạng thời tiết khẩn cấp khác. Việc mô hình hóa như vậy đã dẫn đến các cơ sở chống chịu thời tiết với máy phát điện trong tòa nhà có khả năng vận hành qua một cơn bão.

Hải quân hạt nhân Hoa Kỳ kết hợp các bài tập tư duy “nếu... thì sao” với các cuộc diễn tập thực tế. Theo Jeff Stevenson, diễn tập thực tế nghĩa là “thực sự phá hủy những thứ thật, nhưng có kiểm soát tham số. Các cuộc diễn tập này được tiến hành đều đặn, không sai lệch, mỗi tuần hai đến ba ngày.” Với Hải quân hạt nhân, bài tập tư duy thì hữu ích, nhưng chưa đủ để ứng phó với sự cố thực sự. Các phản ứng phải được luyện tập thường xuyên để không bị quên.

Theo Mike Doherty, các cứu hộ viên đối mặt với những bài kiểm thử thảm họa mang tính chất giống trải nghiệm "khách hàng bí ẩn" (mystery shopper) hơn. Thông thường, một quản lý cơ sở làm việc với một đứa trẻ hoặc một cứu hộ viên huấn luyện đang ẩn danh để dàn dựng một vụ đuối nước giả định. Các kịch bản này được thực hiện càng chân thực càng tốt để các cứu hộ viên không thể phân biệt giữa tình huống khẩn cấp thật và dàn dựng.

## Đào Tạo và Chứng Nhận (Training and Certification)

Các cuộc phỏng vấn của chúng tôi gợi ý rằng việc đào tạo và chứng nhận đặc biệt quan trọng khi sự sống bị đe dọa. Ví dụ, Mike Doherty đã mô tả cách các cứu hộ viên hoàn thành một quy trình chứng nhận đào tạo nghiêm ngặt, bên cạnh quy trình chứng nhận lại định kỳ. Các khóa học bao gồm thành phần thể lực (ví dụ, một cứu hộ viên phải có thể giữ một người nặng hơn mình với vai nổi trên mặt nước), các thành phần kỹ thuật như sơ cứu và CPR (hồi sức tim phổi), và các yếu tố vận hành (ví dụ, nếu một cứu hộ viên xuống nước, các thành viên khác của đội phản ứng như thế nào?). Mỗi cơ sở cũng có đào tạo đặc thù theo địa điểm, vì việc cứu hộ trong hồ bơi khác biệt rõ rệt so với cứu hộ trên bãi biển ven hồ hoặc trên đại dương.

## Tập Trung vào Việc Thu thập Yêu cầu và Thiết kế Chi tiết (Focus on Detailed Requirements Gathering and Design)

Trong các cuộc phỏng vấn, một số kỹ sư nhấn mạnh tầm quan trọng của việc thu thập yêu cầu chi tiết và tài liệu thiết kế (design docs). Thực hành này đặc biệt quan trọng khi làm việc với thiết bị y tế. Trong nhiều trường hợp như vậy, việc sử dụng thực tế hoặc bảo trì thiết bị không thuộc phạm vi của các nhà thiết kế sản phẩm. Do đó, các yêu cầu về sử dụng và bảo trì phải được thu thập từ các nguồn khác.

Ví dụ, theo Erik Gross, các máy phẫu thuật mắt bằng laser được thiết kế để chống sai lầm (foolproof) nhất có thể. Vì vậy, việc tham vấn các bác sĩ phẫu thuật thực sự sử dụng những máy này và các kỹ thuật viên chịu trách nhiệm bảo trì chúng là đặc biệt quan trọng. Trong một ví dụ khác, cựu nhà thầu quốc phòng Peter Dahl đã mô tả một nền văn hóa thiết kế rất chi tiết, trong đó việc tạo ra một hệ thống quốc phòng mới thường đòi hỏi cả một năm thiết kế, rồi chỉ ba tuần viết mã để hiện thực hóa thiết kế đó. Cả hai ví dụ này đều khác xa so với nền văn hóa ra mắt và lặp đi lặp lại (launch and iterate) của Google, thúc đẩy tốc độ thay đổi nhanh hơn nhiều ở một mức rủi ro đã được tính toán. Các ngành khác (ví dụ, ngành y tế và quân đội, như đã thảo luận trước đó) có những áp lực, khẩu vị rủi ro và yêu cầu rất khác nhau, và các quy trình của họ được định hình rất nhiều bởi những hoàn cảnh này.

## Phòng Thủ Nhiều Lớp và Đa Chiều (Defense in Depth and Breadth)

Trong ngành công nghiệp năng lượng hạt nhân, phòng thủ nhiều lớp là yếu tố then chốt của sự chuẩn bị [[IAEA12]](https://sre.google/sre-book/bibliography#IAEA12). Các lò phản ứng hạt nhân được trang bị tính dự phòng (redundancy) cho mọi hệ thống, đồng thời áp dụng phương pháp luận thiết kế yêu cầu các hệ thống dự phòng (fallback) phải nằm phía sau các hệ thống chính để xử lý khi xảy ra sự cố. Hệ thống được thiết kế với nhiều lớp bảo vệ, bao gồm cả rào cản vật lý cuối cùng ngăn chặn sự phát tán chất phóng xạ bao quanh nhà máy. Phòng thủ nhiều lớp đặc biệt quan trọng trong ngành hạt nhân do mức độ không dung lỗi cho các sự cố và tai nạn.

## Văn Hóa Postmortem (Postmortem Culture)

Corrective and preventative action (CAPA)<sup>[4](#fn4)</sup> là khái niệm phổ biến nhằm nâng cao độ tin cậy, tập trung vào việc điều tra có hệ thống nguyên nhân gốc rễ của các vấn đề hoặc rủi ro đã xác định để ngăn chúng tái diễn. Nguyên lý này được thể hiện qua văn hóa mạnh mẽ của SRE về các postmortem không đổ lỗi (blameless). Khi có sự cố (và với quy mô, độ phức tạp cùng tốc độ thay đổi nhanh tại Google, điều gì đó *đáng lẽ* sẽ sai), cần đánh giá tất cả những điều sau:

-   Điều gì đã xảy ra
-   Hiệu quả của sự phản ứng
-   Lần tới chúng ta sẽ làm gì khác đi
-   Những hành động nào sẽ được thực hiện để đảm bảo một sự cố cụ thể không xảy ra lần nữa

Bài tập này không nhằm chỉ trích bất kỳ cá nhân nào. Thay vì gán trách nhiệm, điều quan trọng hơn là tìm ra điều gì đã sai, sai như thế nào, và tổ chức sẽ đoàn kết ra sao để đảm bảo sự cố không tái diễn. Việc dằn vặt về *ai* có thể đã gây ra sự cố ngừng hoạt động là phản tác dụng. Các postmortem được thực hiện sau sự cố và công bố cho các đội SRE để tất cả cùng hưởng lợi từ những bài học rút ra.

Kết quả phỏng vấn cho thấy nhiều ngành đều thực hiện một dạng postmortem (dù nhiều nơi không dùng đúng tên gọi này, vì những lý do dễ hiểu). *Động lực* đằng sau các hoạt động này dường như là yếu tố chính tạo nên sự khác biệt giữa các thực hành.

Nhiều ngành chịu sự giám sát chặt chẽ và phải giải trình trước các cơ quan chính phủ cụ thể khi xảy ra sự cố. Sự giám sát này đặc biệt nghiêm ngặt khi rủi ro cao (ví dụ, đe dọa đến tính mạng). Các cơ quan liên quan gồm FCC (viễn thông), FAA (hàng không), OSHA (sản xuất và hóa chất), FDA (thiết bị y tế), cùng các National Competent Authorities khác nhau trong EU.<sup>[5](#fn5)</sup> Ngành năng lượng hạt nhân và giao thông vận tải cũng chịu sự giám sát chặt chẽ tương tự.

Yếu tố an toàn cũng là một động lực quan trọng thúc đẩy việc thực hiện postmortem. Trong các ngành công nghiệp sản xuất và hóa chất, nguy cơ bị thương hoặc tử vong luôn rình rập do chính bản chất của các điều kiện cần thiết để tạo ra sản phẩm cuối cùng (nhiệt độ cao, áp suất, tính độc hại, và tính ăn mòn, chỉ để nêu vài ví dụ). Chẳng hạn, Alcoa nổi tiếng với văn hóa an toàn đặc biệt. Cựu CEO Paul O'Neill từng yêu cầu nhân viên phải báo cáo cho ông trong vòng 24 giờ bất kỳ chấn thương nào khiến công nhân phải nghỉ việc. Ông thậm chí còn chia sẻ số điện thoại cá nhân cho người lao động trên sàn nhà máy để họ có thể trực tiếp thông báo cho ông về những lo ngại liên quan đến an toàn.<sup>[6](#fn6)</sup>

Trong các ngành công nghiệp sản xuất và hóa chất, mức độ rủi ro cao đến mức ngay cả các "near miss" (sự suýt gặp nạn) — tức là khi một sự kiện cụ thể có khả năng gây tổn hại nghiêm trọng nhưng thực tế không xảy ra — cũng được xem xét kỹ lưỡng. Những kịch bản này đóng vai trò như một dạng postmortem mang tính phòng ngừa. Theo VM Brasseur trong một bài thuyết trình tại YAPC NA 2015, "Có rất nhiều near miss trong hầu như mọi thảm họa và khủng hoảng kinh doanh, và thường chúng bị bỏ qua vào thời điểm xảy ra. Lỗi tiềm ẩn, cộng với một điều kiện cho phép, bằng với các thứ không hoạt động đúng như bạn đã dự định" [[Bra15]](https://sre.google/sre-book/bibliography#Bra15). Các near miss thực chất là những thảm họa đang chờ xảy ra. Ví dụ, các tình huống như một công nhân không tuân thủ quy trình vận hành chuẩn, một nhân viên né tránh vào giây cuối cùng để tránh vũng chất lỏng bắn tung tóe, hoặc một vũng chất lỏng đổ trên cầu thang không được dọn sạch, tất cả đều là near miss và là cơ hội để học hỏi, cải thiện. Lần tới, nhân viên và công ty có thể sẽ không may mắn như vậy. CHIRP của Vương quốc Anh (Confidential Reporting Programme for Aviation and Maritime — Chương trình Báo cáo Mật cho Hàng không và Hàng hải) tìm cách nâng cao nhận thức về những sự cố như vậy trên toàn ngành bằng cách cung cấp một điểm báo cáo trung tâm, nơi nhân viên hàng không và hàng hải có thể báo cáo các near miss một cách kín đáo. Các báo cáo và phân tích về những near miss này sau đó được công bố trong các bản tin định kỳ.

Lực lượng cứu hộ biển có truyền thống phân tích sau sự cố và lập kế hoạch hành động rất sâu sắc. Mike Doherty từng nói đùa: “Nếu chân của một cứu hộ viên chạm vào nước, sẽ có giấy tờ!” Sau bất kỳ sự cố nào ở hồ bơi hoặc trên bãi biển, đều phải có bản ghi chép chi tiết. Với những sự cố nghiêm trọng, cả nhóm cùng ngồi lại xem xét toàn bộ diễn biến từ đầu đến cuối, bàn bạc xem điều gì đã đúng và điều gì đã sai. Dựa trên những phát hiện này, họ thực hiện các thay đổi trong vận hành, đồng thời thường xuyên lên lịch đào tạo để mọi người tự tin hơn khi xử lý các tình huống tương tự trong tương lai. Trong những trường hợp đặc biệt gây sốc hoặc sang chấn, một chuyên viên tư vấn sẽ được mời đến hiện trường để giúp nhân viên đối phó với hậu quả tâm lý. Các cứu hộ viên có thể đã chuẩn bị tốt cho những gì xảy ra trên thực tế, nhưng vẫn có thể *cảm thấy* như mình đã không hoàn thành công việc một cách đầy đủ. Giống như Google, cứu hộ biển đón nhận một nền văn hóa phân tích sự cố không đổ lỗi. Các sự cố vốn hỗn loạn, và nhiều yếu tố góp phần vào bất kỳ sự cố cụ thể nào. Trong lĩnh vực này, việc gán trách nhiệm cho một cá nhân đơn lẻ không hữu ích.

## Loại Bỏ Công Việc Lặp đi Lặp lại và Gánh Nặng Vận hành Bằng Tự Động Hóa (Automating Away Repetitive Work and Operational Overhead)

Bản chất của các Site Reliability Engineer tại Google là những kỹ sư phần mềm có mức độ dung nạp thấp đối với các công việc phản ứng lặp đi lặp lại. Văn hóa của chúng tôi thấm nhuần việc tránh thực hiện một phép vận hành không tạo thêm giá trị cho dịch vụ. Nếu một tác vụ có thể được loại bỏ bằng tự động hóa, thì tại sao lại vận hành một hệ thống dựa trên công việc lặp đi lặp lại có giá trị thấp? Tự động hóa giúp giảm gánh nặng vận hành và giải phóng thời gian để kỹ sư của chúng tôi chủ động đánh giá, cải thiện các dịch vụ mà họ hỗ trợ.

Các ngành chúng tôi khảo sát có quan điểm khác nhau về việc họ có chấp nhận tự động hóa hay không, bằng cách nào và vì sao. Một số ngành tin vào con người hơn là máy móc. Trong suốt nhiệm kỳ của chuyên gia lâu năm trong ngành, Hải quân hạt nhân Hoa Kỳ đã tránh tự động hóa, ưu tiên sử dụng các khóa liên động (interlock) và quy trình hành chính. Ví dụ, theo Jeff Stevenson, để vận hành một van, cần có một người vận hành, một cấp trên và một thành viên tổ đang nói chuyện qua điện thoại với sĩ quan trực kỹ thuật (engineering watch officer) được giao nhiệm vụ theo dõi phản ứng với hành động đã thực hiện. Các thao tác này rất thủ công do lo ngại rằng một hệ thống tự động có thể không phát hiện ra vấn đề mà một con người chắc chắn sẽ nhận ra. Các thao tác trên tàu ngầm được chi phối bởi một chuỗi quyết định của con người đáng tin cậy — một *chuỗi* con người, chứ không phải một cá nhân duy nhất. Hải quân hạt nhân cũng lo ngại rằng tự động hóa và máy tính vận hành nhanh đến mức hoàn toàn có khả năng phạm phải một sai lầm lớn, không thể sửa chữa. Khi đối phó với các lò phản ứng hạt nhân, một phương pháp tiếp cận chậm và kiên nhẫn, có phương pháp, quan trọng hơn so với việc hoàn thành một tác vụ nhanh chóng.

Theo John Li, trong những năm gần đây, ngành môi giới giao dịch (proprietary trading) ngày càng thận trọng hơn khi áp dụng tự động hóa. Kinh nghiệm cho thấy, nếu cấu hình sai, tự động hóa có thể gây ra thiệt hại đáng kể, dẫn đến tổn thất tài chính lớn chỉ trong thời gian rất ngắn. Chẳng hạn, năm 2012, Knight Capital Group gặp một “lỗi phần mềm” (software glitch), khiến công ty mất 440 triệu đô la chỉ trong vài giờ.<sup>[7](#fn7)</sup> Tương tự, năm 2010, thị trường chứng khoán Hoa Kỳ trải qua vụ Flash Crash (sụp đổ nhanh), nguyên nhân cuối cùng được xác định là do một nhà giao dịch phi pháp (rogue trader) cố thao túng thị trường bằng các phương tiện tự động. Dù thị trường phục hồi nhanh, vụ Flash Crash vẫn gây tổn thất lên tới hàng nghìn tỷ đô la chỉ trong *30 phút*.<sup>[8](#fn8)</sup> Máy tính thực hiện tác vụ rất nhanh, và tốc độ này có thể trở thành bất lợi nếu các tác vụ bị cấu hình sai.

Ngược lại, một số công ty lại chọn tự động hóa chính xác *vì* máy tính phản ứng nhanh hơn con người. Theo Eddie Kennedy, trong ngành sản xuất, hiệu suất và chi phí là yếu tố sống còn, và tự động hóa chính là công cụ giúp hoàn thành tác vụ nhanh hơn, rẻ hơn. Hơn nữa, so với thao tác thủ công, tự động hóa thường đáng tin cậy và có tính lặp lại cao hơn, nhờ đó đảm bảo chất lượng tốt hơn và dung sai (tolerance) chặt chẽ hơn. Dan Sheridan từng bàn về việc triển khai tự động hóa trong ngành công nghiệp hạt nhân tại Anh. Ở đó, có một quy tắc kinh nghiệm: nếu nhà máy phải xử lý một tình huống cụ thể trong vòng dưới 30 phút, thì quy trình đó bắt buộc phải được tự động hóa.

Theo kinh nghiệm của Matt Toia, ngành hàng không chỉ áp dụng tự động hóa một cách chọn lọc. Chẳng hạn, quá trình failover được thực hiện hoàn toàn tự động, nhưng với một số tác vụ khác, ngành này chỉ tin tưởng vào tự động hóa khi có con người xác minh. Dù sử dụng rất nhiều giám sát tự động, các triển khai thực tế của hệ thống kiểm soát không lưu vẫn phải được con người kiểm tra thủ công.

Theo Erik Gross, tự động hóa đã phát huy hiệu quả rõ rệt trong việc giảm thiểu lỗi do con người gây ra trong các ca phẫu thuật mắt bằng laser. Trước khi tiến hành phẫu thuật LASIK, bác sĩ sẽ đo thị lực cho bệnh nhân thông qua bài kiểm tra khúc xạ. Ở giai đoạn đầu, bác sĩ phải tự nhập các số liệu và nhấn nút kích hoạt để laser bắt đầu điều chỉnh thị lực. Tuy nhiên, thao tác nhập liệu thủ công này tiềm ẩn nhiều rủi ro nghiêm trọng. Quy trình còn dễ dẫn đến tình trạng nhầm lẫn dữ liệu giữa các bệnh nhân hoặc sai sót khi hoán đổi số liệu của mắt trái và mắt phải.

Tự động hóa hiện nay đã giảm đáng kể nguy cơ con người mắc sai lầm gây ảnh hưởng đến thị lực. Bước kiểm tra tính hợp lý (sanity check) bằng máy tính đối với dữ liệu nhập thủ công là cải tiến tự động hóa lớn đầu tiên: nếu người vận hành nhập các phép đo nằm ngoài phạm vi kỳ vọng, hệ thống sẽ nhanh chóng và nổi bật gắn cờ trường hợp này là bất thường. Nhiều cải tiến tự động hóa khác đã tiếp nối sau đó: hiện tại, mống mắt được chụp ảnh trong quá trình kiểm tra khúc xạ mắt sơ bộ. Khi đến thời điểm phẫu thuật, mống mắt của bệnh nhân được tự động khớp với mống mắt trong ảnh, qua đó loại bỏ khả năng nhầm lẫn dữ liệu bệnh nhân. Khi giải pháp tự động hóa này được triển khai, cả một lớp lỗi y tế đã biến mất.

## Ra Quyết Định Có Cấu Trúc và Hợp Lý (Structured and Rational Decision Making)

Ở Google nói chung, và cụ thể hơn trong Site Reliability Engineering, dữ liệu đóng vai trò then chốt. Đội ngũ hướng tới việc ra quyết định có cấu trúc và hợp lý bằng cách đảm bảo rằng:

-   Cơ sở cho quyết định được thống nhất từ trước, thay vì được biện minh một cách ex post facto (sau khi sự việc đã xảy ra)
-   Các đầu vào cho quyết định là rõ ràng
-   Mọi giả định đều được nêu rõ
-   Những quyết định dựa trên dữ liệu thắng những quyết định dựa trên cảm xúc, trực giác, hoặc ý kiến của nhân viên cấp cao nhất trong phòng

Google SRE vận hành dựa trên giả định nền tảng rằng mọi người trong đội:

-   Luôn đặt lợi ích tốt nhất của người dùng dịch vụ lên hàng đầu
-   Có thể tìm ra cách tiếp tục dựa trên dữ liệu có sẵn

Quyết định cần dựa trên dữ liệu thay vì áp đặt (prescriptive), và phải được đưa ra khách quan, không bị chi phối bởi ý kiến cá nhân — kể cả của người có cấp cao nhất trong phòng, hay còn gọi là "HiPPO" (viết tắt của "Highest-Paid Person's Opinion", Ý kiến của Người Được Trả Lương Cao nhất) theo cách gọi của Eric Schmidt và Jonathan Rosenberg [[Sch14]](https://sre.google/sre-book/bibliography#Sch14).

Cách ra quyết định ở các lĩnh vực khác nhau rất khác biệt. Chúng tôi nhận thấy một số ngành áp dụng triết lý *nếu nó chưa hỏng, đừng sửa nó... bao giờ*. Những ngành có hệ thống đòi hỏi thiết kế phải được suy nghĩ và đầu tư công sức kỹ lưỡng thường tỏ ra e dè trước việc thay đổi công nghệ nền tảng. Chẳng hạn, ngành viễn thông vẫn đang vận hành các máy chuyển mạch đường dài được triển khai từ những năm 1980. Tại sao họ lại dựa vào công nghệ phát triển từ vài thập kỷ trước? Theo Gus Hartmann, những máy chuyển mạch này "gần như bất khả xâm phạm và dự phòng một cách mạnh mẽ." Như Dan Sheridan đã báo cáo, ngành công nghiệp hạt nhân cũng chậm thay đổi một cách tương tự. Mọi quyết định đều được nâng đỡ bởi suy nghĩ: *nếu nó hoạt động bây giờ, đừng thay đổi nó*.

Nhiều ngành ưu tiên xây dựng playbook (sổ tay sách lược) và quy trình hơn là giải quyết các vấn đề mở. Mọi kịch bản có thể xảy ra đều được ghi lại trong một checklist hoặc trong "the binder" (sổ tay tổng hợp). Khi có sự cố, đây là nguồn tài liệu có thẩm quyền để xác định cách phản ứng. Cách tiếp cận mang tính chỉ đạo này phù hợp với các ngành tiến hóa và phát triển tương đối chậm, vì các kịch bản về những điều có thể xảy ra sai không liên tục thay đổi do các bản cập nhật hoặc thay đổi hệ thống. Phương pháp này cũng phổ biến trong các ngành mà trình độ kỹ năng của người lao động có thể bị hạn chế, và cách tốt nhất để đảm bảo mọi người phản ứng phù hợp trong một trường hợp khẩn cấp là cung cấp một bộ hướng dẫn đơn giản, rõ ràng.

Các ngành khác cũng có cách tiếp cận rõ ràng, dựa trên dữ liệu, cho việc ra quyết định. Theo kinh nghiệm của Eddie Kennedy, các môi trường nghiên cứu và sản xuất mang đặc trưng của một nền văn hóa thực nghiệm nghiêm ngặt, phụ thuộc rất nhiều vào việc xây dựng và kiểm thử các giả thuyết. Các ngành này thường xuyên tiến hành các thí nghiệm có kiểm soát để đảm bảo rằng một thay đổi cụ thể mang lại kết quả được kỳ vọng ở mức có ý nghĩa thống kê và rằng không có điều gì bất ngờ xảy ra. Các thay đổi chỉ được triển khai khi dữ liệu do thí nghiệm tạo ra ủng hộ quyết định.

Cuối cùng, một số ngành, như môi giới giao dịch, chia nhỏ việc ra quyết định để quản lý rủi ro tốt hơn. Theo John Li, ngành công nghiệp này có một đội thực thi (enforcement team) tách biệt khỏi các nhà giao dịch để đảm bảo không có rủi ro quá mức nào được chấp nhận trong khi theo đuổi lợi nhuận. Đội thực thi chịu trách nhiệm theo dõi các sự kiện trên sàn và dừng giao dịch nếu các sự kiện mất kiểm soát. Nếu một bất thường của hệ thống xảy ra, phản ứng đầu tiên của đội thực thi là tắt hệ thống. Như John Li nói, "Nếu chúng ta không giao dịch, chúng ta không mất tiền. Chúng ta cũng không kiếm được tiền, nhưng ít nhất chúng ta không mất tiền." Chỉ đội thực thi mới có thể khởi động lại hệ thống, bất kể sự chậm trễ có thể đau đớn đến mức nào đối với các nhà giao dịch đang bỏ lỡ một cơ hội có thể có lợi nhuận.

## Kết Luận (Conclusions)

Nhiều nguyên lý cốt lõi của Site Reliability Engineering tại Google được thể hiện rõ ràng trên phạm vi rộng các ngành. Những bài học mà các ngành công nghiệp lâu đời đã tích lũy có lẽ đã truyền cảm hứng cho một số thực hành đang được sử dụng tại Google ngày nay.

Một điểm chính rút ra từ cuộc khảo sát liên ngành của chúng tôi là trong nhiều mảng doanh nghiệp phần mềm, Google chấp nhận tốc độ (velocity) cao hơn so với các đối thủ ở hầu hết các ngành khác. Khả năng di chuyển hoặc thay đổi nhanh chóng cần được cân nhắc cùng với những hệ quả khác nhau của một sự cố. Ví dụ, trong ngành công nghiệp hạt nhân, hàng không hoặc y tế, con người có thể bị thương hoặc thậm chí tử vong nếu xảy ra sự cố ngừng hoạt động. Khi mức độ rủi ro cao, một cách tiếp cận thận trọng nhằm đạt được độ tin cậy cao là xứng đáng.

Tại Google, chúng tôi luôn phải cân bằng giữa kỳ vọng của người dùng về độ tin cậy cao và sự tập trung sắc bén vào tốc độ thay đổi, đổi mới nhanh chóng. Dù rất coi trọng độ tin cậy, chúng tôi vẫn phải điều chỉnh cách tiếp cận cho phù hợp với tốc độ thay đổi cao. Như đã đề cập ở các chương trước, nhiều doanh nghiệp phần mềm của chúng tôi, chẳng hạn như Search, đưa ra những quyết định có chủ đích về mức độ "đáng tin cậy đủ" thực sự cần thiết.

Google áp dụng sự linh hoạt này cho hầu hết các sản phẩm và dịch vụ phần mềm, vốn hoạt động trong môi trường mà sự cố không đe dọa trực tiếp đến tính mạng. Vì vậy, chúng tôi có thể dùng các công cụ như error budget (ngân sách lỗi) ([Motivation for Error Budgets](https://sre.google/sre-book/embracing-risk#xref_risk-management_unreliability-budgets)) để "tài trợ" cho văn hóa đổi mới và chấp nhận rủi ro có kiểm soát. Về bản chất, Google đã điều chỉnh các nguyên lý độ tin cậy đã được biết đến — trong nhiều trường hợp là những nguyên lý được phát triển và hoàn thiện từ các ngành khác — nhằm xây dựng văn hóa độ tin cậy đặc trưng của riêng mình, một văn hóa giải quyết bài toán cân bằng phức tạp giữa quy mô, độ phức tạp, tốc độ và độ tin cậy cao.

<a id="fn1"></a>[1](#fn1) E911 (Enhanced 911): Đường dây phản ứng khẩn cấp tại Hoa Kỳ sử dụng dữ liệu vị trí.
<a id="fn2"></a>[2](#fn2) Electrocardiogram readings (Các số đo điện tâm đồ): [*https://en.wikipedia.org/wiki/Electrocardiography*](https://en.wikipedia.org/wiki/Electrocardiography).
<a id="fn3"></a>[3](#fn3) [*https://en.wikipedia.org/wiki/Safety_integrity_level*](https://en.wikipedia.org/wiki/Safety_integrity_level)
<a id="fn4"></a>[4](#fn4) [*https://en.wikipedia.org/wiki/Corrective_and_preventive_action*](https://en.wikipedia.org/wiki/Corrective_and_preventive_action)
<a id="fn5"></a>[5](#fn5) [*https://en.wikipedia.org/wiki/Competent_authority*](https://en.wikipedia.org/wiki/Competent_authority)
<a id="fn6"></a>[6](#fn6) [*https://ehstoday.com/safety/nsc-2013-oneill-exemplifies-safety-leadership*](https://ehstoday.com/safety/nsc-2013-oneill-exemplifies-safety-leadership).
<a id="fn7"></a>[7](#fn7) Xem "FACTS, Section B" để thảo luận về phần mềm của Knight và Power Peg trong [[Sec13]](https://sre.google/sre-book/bibliography#Sec13).
<a id="fn8"></a>[8](#fn8) "Regulators blame computer algorithm for stock market 'flash crash'," Computerworld, [*https://www.computerworld.com/article/2516076/financial-it/regulators-blame-computer-algorithm-for-stock-market-flash-crash-.html*](https://www.computerworld.com/article/2516076/financial-it/regulators-blame-computer-algorithm-for-stock-market-flash-crash-.html).

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
