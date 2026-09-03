# Chương 11. On-Call (Trực sự cố)

> **Nguyên bản:** [Chapter 11 - Being On-Call](https://sre.google/sre-book/being-on-call/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Andrea Spadaccini<sup>[1](#fn1)</sup>
*Biên tập:* Kavita Guliani

On-call (trực sự cố) là một nhiệm vụ quan trọng mà nhiều đội vận hành và kỹ thuật phải đảm nhận để giữ các dịch vụ của họ đáng tin cậy và khả dụng. Tuy nhiên, có không ít cạm bẫy trong cách tổ chức các vòng on-call và phân chia trách nhiệm, có thể dẫn đến hậu quả nghiêm trọng cho cả dịch vụ lẫn đội nếu không được né tránh. Chương này trình bày các nguyên tắc chính của cách tiếp cận on-call mà các Site Reliability Engineer (SRE) của Google đã phát triển qua nhiều năm, và giải thích cách tiếp cận đó dẫn đến các dịch vụ đáng tin cậy cùng một khối lượng công việc bền vững theo thời gian.

## Giới thiệu (Introduction)

Nhiều nghề đòi hỏi nhân viên thực hiện một dạng nhiệm vụ on-call, bao gồm việc sẵn sàng nhận các cuộc gọi cả trong lẫn ngoài giờ làm việc. Trong lĩnh vực IT, các hoạt động on-call về mặt lịch sử do các đội Ops chuyên trách đảm nhận, với trách nhiệm chính là giữ cho dịch vụ họ phụ trách ở trạng thái tốt.

Nhiều dịch vụ quan trọng tại Google, ví dụ Search, Ads và Gmail, có các đội SRE chuyên trách chịu trách nhiệm hiệu năng và độ tin cậy của chúng. Do đó, [các SRE trực on-call](https://sre.google/workbook/on-call/) cho các dịch vụ mà họ hỗ trợ. Các đội SRE khác biệt rõ với các đội thuần vận hành ở chỗ chúng tôi đặt trọng tâm lớn vào việc dùng kỹ thuật (engineering) để tiếp cận vấn đề. Những vấn đề này, thường thuộc lĩnh vực vận hành, nằm ở một quy mô mà không có giải pháp kỹ thuật phần mềm thì không thể xử lý nổi.

Để thực hiện kiểu giải quyết vấn đề này, Google tuyển những người có nền tảng đa dạng về kỹ thuật hệ thống và phần mềm vào các đội SRE. Chúng tôi giới hạn thời gian SRE dành cho công việc thuần vận hành ở mức 50%; tối thiểu 50% thời gian của một SRE phải dành cho các dự án kỹ thuật giúp mở rộng tác động của đội thông qua tự động hóa, song song với việc cải thiện dịch vụ.

## Cuộc sống của một Kỹ sư On-Call (Life of an On-Call Engineer)

Phần này mô tả các hoạt động điển hình của một kỹ sư on-call và cung cấp một số bối cảnh cho phần còn lại của chương.

Với tư cách là người canh giữ các hệ thống production, [các kỹ sư on-call](https://sre.google/workbook/on-call/) lo các vận hành được giao của mình bằng cách xử lý các outage (mất dịch vụ) ảnh hưởng đến đội và thực hiện và/hoặc xem xét (vet) các thay đổi production.

Khi on-call, một kỹ sư phải sẵn sàng thực hiện các thao tác trên hệ thống production trong vòng vài phút, theo thời gian phản hồi page (gọi trực) đã thỏa thuận giữa đội và chủ sở hữu hệ thống business. Giá trị điển hình là 5 phút cho các dịch vụ hướng người dùng hoặc rất nhạy cảm về thời gian, và 30 phút cho các hệ thống ít nhạy cảm hơn. Công ty cấp thiết bị nhận page, thường là một chiếc điện thoại. Google có các hệ thống gửi cảnh báo linh hoạt, có thể điều phối page qua nhiều cơ chế (email, SMS, cuộc gọi tự động, ứng dụng) trên nhiều thiết bị.

Thời gian phản hồi liên quan đến mức khả dụng dịch vụ mong muốn, như ví dụ đơn giản sau: nếu một hệ thống hướng người dùng phải đạt 4 nines khả dụng trong một quý (99.99%), thời gian downtime (ngừng dịch vụ) hàng quý cho phép là khoảng 13 phút ([Availability Table](https://sre.google/sre-book/availability-table#appendix_table-of-nines)). Ràng buộc này có nghĩa thời gian phản ứng của kỹ sư on-call phải ở mức phút (chính xác hơn là 13 phút). Với các hệ thống có SLO (Service Level Objective — Mục tiêu Mức Dịch vụ) nới lỏng hơn, thời gian phản ứng có thể ở mức vài chục phút.

Ngay khi nhận và xác nhận một page, kỹ sư on-call được kỳ vọng sẽ phân loại (triage) vấn đề và tiến hành giải quyết, có thể kéo các thành viên khác trong đội vào và leo thang (escalate) khi cần.

Kỹ sư on-call cũng có thể xử lý và/hoặc xem xét trong giờ làm việc các sự kiện production không-page, chẳng hạn các cảnh báo ưu tiên thấp hơn hoặc các release (phát hành) phần mềm. Những hoạt động này ít khẩn cấp hơn các sự kiện page — thứ được ưu tiên hơn gần như mọi tác vụ khác, kể cả công việc dự án. Để hiểu thêm về các ngắt (interrupt) và các sự kiện không-page khác góp phần vào khối lượng công việc vận hành, xem [Dealing with Interrupts](https://sre.google/sre-book/dealing-with-interrupts/).

Nhiều đội có cả vòng on-call chính (primary) và phụ (secondary). Cách phân chia nhiệm vụ giữa chính và phụ khác nhau tùy đội. Đội này có thể dùng người phụ làm điểm rơi (fall-through) cho các page mà on-call chính bỏ lỡ. Đội khác có thể quy định on-call chính chỉ xử lý page, còn người phụ đảm nhận mọi hoạt động production khẩn cấp khác.

Ở những đội không bắt buộc phải có vòng phụ để phân chia nhiệm vụ, thường thấy hai đội liên quan làm on-call phụ cho nhau, kiêm luôn việc xử lý điểm rơi. Cách sắp xếp này loại bỏ nhu cầu về một vòng on-call phụ riêng biệt.

Có nhiều cách để tổ chức các vòng on-call; để có phân tích chi tiết, tham khảo chương "Oncall" của [[Lim14]](https://sre.google/sre-book/bibliography#Lim14).

<a id="on-call-can-bang"></a>

## On-Call Cân bằng (Balanced On-Call)

Các đội SRE có những ràng buộc cụ thể về số lượng và chất lượng của các ca on-call. Số lượng on-call có thể đo bằng phần trăm thời gian kỹ sư dành cho nhiệm vụ on-call. Chất lượng on-call có thể đo bằng số incident (sự cố) xảy ra trong một ca on-call.

Các quản lý SRE có trách nhiệm giữ cho khối lượng công việc on-call cân bằng và bền vững trên cả hai trục này.

## Cân bằng về Số lượng (Balance in Quantity)

Chúng tôi tin chắc rằng chữ "E" trong "SRE" là đặc tính định nghĩa của tổ chức mình, nên cố gắng đầu tư ít nhất 50% thời gian SRE vào kỹ thuật: phần còn lại, tối đa 25% được dành cho on-call, để lại tối đa 25% nữa cho các loại công việc vận hành không thuộc dự án.

Áp dụng quy tắc 25% on-call, chúng tôi có thể suy ra số lượng tối thiểu SRE cần để duy trì một vòng on-call 24/7. Giả sử luôn có hai người on-call (chính và phụ, nhiệm vụ khác nhau), số kỹ sư tối thiểu cần cho nhiệm vụ on-call của một đội một vị trí (single-site) là tám: với ca tính theo tuần, mỗi kỹ sư on-call (chính hoặc phụ) một tuần mỗi tháng. Với các đội hai vị trí (dual-site), kích thước tối thiểu hợp lý của mỗi đội là sáu, vừa đáp ứng quy tắc 25% vừa đảm bảo đủ số lượng và khối lượng tới hạn (critical mass) kỹ sư cho đội.

Nếu một dịch vụ có đủ công việc để hợp lý hóa việc thành lập một đội một vị trí, chúng tôi vẫn thích tạo một đội đa vị trí (multi-site). Đội đa vị trí có lợi thế về hai mặt:

-   Ca đêm có hại cho sức khỏe con người [[Dur05]](https://sre.google/sre-book/bibliography#Dur05), và một vòng on-call "follow the sun" (theo mặt trời) đa vị trí cho phép các đội tránh hoàn toàn ca đêm.
-   Giới hạn số kỹ sư trong vòng on-call đảm bảo các kỹ sư không mất liên hệ với hệ thống production (xem [Một kẻ thù hiểm nguy: Chở quá ít công việc vận hành](#mot-ke-thu-hiem-nguy-cho-qua-it-cong-viec-van-hanh)).

Tuy nhiên, các đội đa vị trí phát sinh chi phí giao tiếp và phối hợp. Vì vậy, quyết định đi đa vị trí hay một vị trí nên dựa trên những đánh đổi của mỗi lựa chọn, tầm quan trọng của hệ thống, và khối lượng công việc mà hệ thống đó tạo ra.

## Cân bằng về Chất lượng (Balance in Quality)

Mỗi ca on-call, một kỹ sư nên có đủ thời gian để xử lý mọi incident và các hoạt động tiếp theo như viết postmortem (báo cáo sau sự cố) [[Loo10]](https://sre.google/sre-book/bibliography#Loo10). Ta định nghĩa một incident là một chuỗi sự kiện và cảnh báo liên quan đến cùng một nguyên nhân gốc rễ (root cause), sẽ được thảo luận trong cùng một postmortem. Chúng tôi nhận thấy trung bình, việc xử lý các tác vụ của một [on-call incident](https://sre.google/sre-book/emergency-response/) — phân tích nguyên nhân gốc rễ, khắc phục, và các bước tiếp theo như viết postmortem, sửa bug — mất 6 giờ. Suy ra, số incident tối đa mỗi ngày là 2 cho một ca on-call 12 giờ. Để ở dưới giới hạn này, sự phân bố các sự kiện page phải rất phẳng theo thời gian, với giá trị trung vị (median) có thể là 0: nếu một thành phần hay vấn đề cụ thể gây page mỗi ngày (trung vị incidents/ngày > 1), nhiều khả năng một thứ gì đó khác sẽ hỏng vào lúc nào đó, kéo theo nhiều incident hơn mức cho phép.

Nếu giới hạn này bị vượt quá tạm thời, ví dụ trong một quý, cần có các biện pháp điều chỉnh để khối lượng công việc vận hành quay lại trạng thái bền vững (xem [Chở quá nhiều công việc vận hành](#cho-qua-nhieu-cong-viec-van-hanh) và [Đưa một SRE vào Đội để Phục hồi từ Chở quá nhiều Công việc Vận hành](https://sre.google/sre-book/operational-overload)).

## Bù đắp (Compensation)

Cần xem xét bù đắp đầy đủ cho việc hỗ trợ ngoài giờ. Các tổ chức khác nhau xử lý việc bù đắp on-call khác nhau; Google cho nghỉ bù (time-off-in-lieu) hoặc bù tiền mặt, giới hạn ở một tỷ lệ nào đó của tổng lương. Mức trần bù đắp, trên thực tế, thể hiện giới hạn cho lượng công việc on-call mà bất kỳ cá nhân nào đảm nhận. Cấu trúc bù đắp này vừa tạo động lực tham gia nhiệm vụ on-call như đội yêu cầu, vừa thúc đẩy phân phối công việc on-call cân bằng và hạn chế các tác dụng phụ của việc on-call quá mức, như kiệt sức (burnout) hay thiếu thời gian cho công việc dự án.

## Cảm thấy An toàn (Feeling Safe)

Như đã đề cập, các đội SRE hỗ trợ những hệ thống quan trọng nhất của Google. Là SRE on-call thường có nghĩa là gánh trách nhiệm cho các hệ thống hướng người dùng, quan trọng về doanh thu, hoặc cho hạ tầng cần thiết để giữ những hệ thống này chạy. Phương pháp luận SRE trong cách suy nghĩ và giải quyết vấn đề là thiết yếu cho việc vận hành dịch vụ đúng cách.

Nghiên cứu hiện đại xác định hai cách suy nghĩ riêng biệt mà một người có thể chọn, có ý thức hoặc tiềm thức, khi đối mặt với thách thức [[Kah11]](https://sre.google/sre-book/bibliography#Kah11):

-   Hành động trực giác, tự động và nhanh chóng
-   Các chức năng nhận thức lý tính, tập trung và có chủ đích

Khi đối phó với các outage của những hệ thống phức tạp, lựa chọn thứ hai nhiều khả năng cho kết quả tốt hơn và dẫn đến cách xử lý incident được lên kế hoạch kỹ lưỡng.

Để kỹ sư có trạng thái tinh thần phù hợp, tận dụng được kiểu suy nghĩ thứ hai, việc giảm căng thẳng khi on-call là quan trọng. Tầm quan trọng, tác động của dịch vụ và hậu quả của các outage tiềm tàng có thể tạo áp lực lớn lên kỹ sư on-call, làm hại sức khỏe của từng thành viên trong đội và có thể khiến SRE đưa ra lựa chọn sai, đe dọa khả dụng của dịch vụ. Các hormone căng thẳng như cortisol và corticotropin-releasing hormone (CRH) vốn được biết là gây ra các hệ quả về hành vi — bao gồm cả nỗi sợ — có thể làm suy giảm chức năng nhận thức và khiến quyết định đưa ra không tối ưu [[Chr09]](https://sre.google/sre-book/bibliography#Chr09).

Dưới ảnh hưởng của những hormone căng thẳng này, cách suy nghĩ có chủ đích thường bị lấn át bởi hành động không suy ngẫm, không cân nhắc (nhưng tức thì), dẫn đến việc lạm dụng các heuristic (quy tắc gần đúng). Heuristic là hành vi rất cám dỗ khi đang on-call. Ví dụ, khi cùng một cảnh báo page lần thứ tư trong tuần, và ba page trước đó đều do một hệ thống hạ tầng bên ngoài gây ra, người ta rất dễ bị cám dỗ thực hiện thiên kiến xác nhận (confirmation bias), tự động gán lần xuất hiện thứ tư này cho nguyên nhân cũ.

Dù trực giác và phản ứng nhanh có vẻ là những phẩm chất đáng có giữa lúc quản lý incident, chúng cũng có mặt tiêu cực. Trực giác có thể sai và thường không có dữ liệu rõ ràng làm cơ sở. Vì vậy, làm theo trực giác có thể khiến kỹ sư lãng phí thời gian truy theo một đường suy luận sai ngay từ đầu. Phản ứng nhanh đã ăn sâu thành thói quen, mà phản ứng theo thói quen là không cân nhắc — nghĩa là nó có thể dẫn đến thảm họa. Phương pháp luận lý tưởng trong quản lý incident là đạt được sự cân bằng: thực hiện các bước với tốc độ mong muốn khi có đủ dữ liệu để quyết định hợp lý, đồng thời xem xét phê phán các giả định của mình.

Quan trọng là [các SRE on-call](https://sre.google/sre-book/accelerating-sre-on-call/) hiểu rằng họ có thể dựa vào một số nguồn lực giúp trải nghiệm on-call bớt đáng sợ hơn so với vẻ bề ngoài. Các nguồn lực on-call quan trọng nhất là:

-   Các đường leo thang rõ ràng (escalation paths)
-   Các quy trình quản lý incident được định nghĩa rõ ràng
-   Một văn hóa postmortem không đổ lỗi ([[Loo10]](https://sre.google/sre-book/bibliography#Loo10), [[All12]](https://sre.google/sre-book/bibliography#All12))

Các đội developer (phát triển) của những hệ thống do SRE hỗ trợ thường tham gia một vòng on-call 24/7, và luôn có thể leo thang đến các đội đối tác này khi cần. Việc leo thang outage đúng lúc nhìn chung là cách có nguyên tắc để phản ứng với các outage nghiêm trọng có nhiều yếu tố chưa rõ.

Khi xử lý incident, nếu vấn đề đủ phức tạp để kéo nhiều đội vào, hoặc nếu sau một số điều tra vẫn chưa ước lượng được giới hạn trên của thời gian kéo dài incident, thì việc áp dụng một giao thức quản lý incident chính thức sẽ hữu ích. Google SRE dùng giao thức mô tả trong [Managing Incidents](https://sre.google/sre-book/managing-incidents/): một tập các bước dễ theo dõi, định nghĩa rõ ràng, giúp kỹ sư on-call theo đuổi hợp lý một giải pháp incident thỏa mãn với đầy đủ sự hỗ trợ cần thiết. Bên trong công ty, một công cụ web hỗ trợ giao thức này bằng cách tự động hóa phần lớn hành động quản lý incident, từ bàn giao vai trò, ghi lại, đến truyền thông các cập nhật trạng thái. Nhờ vậy, người quản lý incident tập trung vào việc xử lý incident thay vì mất thời gian và công sức nhận thức cho những việc vụn vặt như định dạng email hay cập nhật nhiều kênh truyền thông cùng lúc.

Cuối cùng, khi một incident xảy ra, quan trọng là đánh giá điều gì đã sai, nhận ra điều gì đã tốt, và hành động để ngăn các lỗi tương tự lặp lại. Các đội SRE phải viết postmortem sau các incident đáng kể, nêu chi tiết một dòng thời gian đầy đủ các sự kiện đã xảy ra. Bằng cách tập trung vào sự kiện thay vì con người, những postmortem này mang lại giá trị lớn. Thay vì đổ lỗi cho cá nhân, chúng rút ra bài học từ việc phân tích có hệ thống các incident production. Sai lầm sẽ xảy ra, và phần mềm phải giúp chúng tôi mắc càng ít sai lầm càng tốt. Nhận ra các cơ hội tự động hóa là một trong những cách tốt nhất để ngăn lỗi con người [[Loo10]](https://sre.google/sre-book/bibliography#Loo10).

## Tránh Khối lượng Công việc Vận hành Không thích hợp (Avoiding Inappropriate Operational Load)

Như đã đề cập trong [On-Call Cân bằng](#on-call-can-bang), SRE dành tối đa 50% thời gian cho công việc vận hành. Điều gì xảy ra nếu công việc vận hành vượt quá giới hạn này?

<a id="cho-qua-nhieu-cong-viec-van-hanh"></a>

## Chở quá nhiều Công việc Vận hành (Operational Overload)

Đội SRE và ban lãnh đạo chịu trách nhiệm đưa các mục tiêu cụ thể vào kế hoạch công việc hàng quý để đảm bảo khối lượng công việc quay về mức bền vững. Cho "vay" tạm thời một SRE giàu kinh nghiệm cho một đội đang quá tải — như thảo luận trong [Đưa một SRE vào Đội để Phục hồi từ Chở quá nhiều Công việc Vận hành](https://sre.google/sre-book/operational-overload/) — có thể tạo đủ khoảng trống để đội tiến triển trong việc giải quyết vấn đề.

Về lý tưởng, các triệu chứng của việc chở quá nhiều công việc vận hành nên đo được, để mục tiêu có thể định lượng (ví dụ, số ticket mỗi ngày < 5, sự kiện page mỗi ca < 2).

Cấu hình giám sát sai là nguyên nhân phổ biến của việc chở quá nhiều công việc vận hành. Các cảnh báo page nên được căn chỉnh với các triệu chứng đe dọa SLO của dịch vụ. Mọi cảnh báo page cũng phải có thể hành động được (actionable). Các cảnh báo ưu tiên thấp làm phiền kỹ sư on-call mỗi giờ (hoặc thường xuyên hơn) gây gián đoạn năng suất, và sự mệt mỏi chúng gây ra có thể khiến các cảnh báo nghiêm trọng bị xử lý với ít sự chú ý hơn mức cần. Xem [Dealing with Interrupts](https://sre.google/sre-book/dealing-with-interrupts/) để thảo luận thêm.

Cũng quan trọng là kiểm soát số cảnh báo mà kỹ sư on-call nhận cho một incident đơn lẻ. Đôi khi một điều kiện bất thường đơn lẻ có thể tạo ra nhiều cảnh báo, nên cần điều tiết việc fan-out (tán ra) cảnh báo bằng cách đảm bảo các cảnh báo liên quan được hệ thống giám sát hoặc cảnh báo nhóm lại với nhau. Nếu vì lý do nào đó mà các cảnh báo trùng lặp hoặc không mang thông tin được tạo ra trong một incident, việc tắt (silencing) chúng có thể mang lại sự yên lặng cần thiết để kỹ sư on-call tập trung vào chính incident. Các cảnh báo nhiễu tạo ra hơn một cảnh báo mỗi incident nên được tinh chỉnh để tiệm cận tỷ lệ cảnh báo/incident 1:1. Làm vậy giúp kỹ sư on-call tập trung vào incident thay vì phải phân loại các cảnh báo trùng lặp.

Đôi khi những thay đổi gây ra việc chở quá nhiều công việc vận hành không nằm trong tầm kiểm soát của các đội SRE. Ví dụ, developer ứng dụng có thể đưa vào các thay đổi khiến hệ thống trở nên nhiễu hơn, kém tin cậy hơn, hoặc cả hai. Trong trường hợp này, hợp lý là làm việc cùng developer ứng dụng để đặt các mục tiêu chung cải thiện hệ thống.

Trong các trường hợp cực đoan, các đội SRE có thể có tùy chọn "trả lại pager (máy gọi trực)" — SRE có thể yêu cầu đội developer on-call độc quyền cho hệ thống cho đến khi nó đáp ứng các tiêu chuẩn của đội SRE. Việc trả lại pager không xảy ra thường xuyên, vì gần như luôn có thể làm việc với đội developer để giảm khối lượng công việc vận hành và làm hệ thống cụ thể đó đáng tin cậy hơn. Tuy nhiên, trong một số trường hợp, các thay đổi phức tạp hoặc kiến trúc trải qua nhiều quý có thể cần thiết để làm hệ thống bền vững từ góc độ vận hành. Trong những trường hợp đó, đội SRE không nên phải chịu một khối lượng công việc vận hành quá mức. Thay vào đó, hợp lý là đàm phán tái tổ chức trách nhiệm on-call với đội phát triển, có thể định tuyến một số hoặc tất cả cảnh báo page đến developer on-call. Giải pháp như vậy thường là biện pháp tạm thời, trong khoảng thời gian đó các đội SRE và developer làm việc cùng nhau để đưa dịch vụ vào trạng thái có thể được SRE onboard (tiếp nhận) lại.

Khả năng đàm phán lại trách nhiệm on-call giữa các đội SRE và phát triển sản phẩm cho thấy sự cân bằng quyền lực giữa các đội.<sup>[2](#fn2)</sup> Mối quan hệ làm việc này cũng là ví dụ điển hình cho cách căng thẳng lành mạnh giữa hai đội — và các giá trị họ đại diện, độ tin cậy so với tốc độ tính năng — thường được giải quyết bằng cách mang lại lợi ích lớn cho dịch vụ và, qua đó, cho toàn bộ công ty.

<a id="mot-ke-thu-hiem-nguy-cho-qua-it-cong-viec-van-hanh"></a>

## Một kẻ thù hiểm nguy: Chở quá ít Công việc Vận hành (A Treacherous Enemy: Operational Underload)

On-call cho một hệ thống yên tĩnh thì tuyệt vời, nhưng sẽ ra sao nếu hệ thống quá yên tĩnh, hoặc khi SRE không on-call đủ thường xuyên? Chở quá ít công việc vận hành (operational underload) là điều không mong muốn với một đội SRE. Mất liên lạc với production trong thời gian dài có thể gây ra vấn đề về sự tự tin — cả tự tin thái quá lẫn thiếu tự tin — trong khi các khoảng trống kiến thức chỉ bị lộ ra khi một incident xảy ra.

Để chống lại điều này, các đội SRE nên được định biên sao cho mỗi kỹ sư on-call ít nhất một hoặc hai lần mỗi quý, qua đó đảm bảo mỗi thành viên được tiếp xúc đủ với production. Các bài tập "Wheel of Misfortune" (Vòng bánh Bất hạnh) (thảo luận trong [Accelerating SREs to On-Call and Beyond](https://sre.google/sre-book/accelerating-sre-on-call/)) cũng là hoạt động đội hữu ích, giúp mài giũa và cải thiện kỹ năng, kiến thức về dịch vụ. Google còn có một sự kiện phục hồi thảm họa hàng năm trên toàn công ty tên là DiRT (Disaster Recovery Training — Đào tạo Phục hồi Thảm họa), kết hợp bài tập lý thuyết và thực hành để kiểm thử nhiều ngày các hệ thống hạ tầng và các dịch vụ riêng lẻ; xem [[Kri12]](https://sre.google/sre-book/bibliography#Kri12).

## Kết luận (Conclusions)

Cách tiếp cận on-call được mô tả ở chương này đóng vai trò hướng dẫn cho tất cả các đội SRE trong Google và là chìa khóa để nuôi dưỡng một môi trường làm việc bền vững, có thể quản lý được. Nhờ đó, cách tiếp cận on-call của Google cho phép chúng tôi dùng công việc kỹ thuật làm phương tiện chính để mở rộng (scale) trách nhiệm production và duy trì độ tin cậy, khả dụng cao, bất chấp sự phức tạp và số lượng ngày càng tăng của các hệ thống, dịch vụ mà SRE chịu trách nhiệm.

Dù cách tiếp cận này có thể không áp dụng được ngay cho mọi ngữ cảnh mà kỹ sư phải on-call cho dịch vụ IT, chúng tôi tin rằng nó là một mô hình vững chắc mà các tổ chức có thể áp dụng để mở rộng và đáp ứng khối lượng công việc on-call ngày càng tăng.

<a id="fn1"></a>[1](#fn1) Một phiên bản trước đó của chương này xuất hiện như một bài viết trong *;login:* (tháng 10 năm 2015, tập 40, số 5).

<a id="fn2"></a>[2](#fn2) Để có thêm thảo luận về căng thẳng tự nhiên giữa các đội SRE và phát triển sản phẩm, xem [Introduction](https://sre.google/sre-book/introduction/).

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
