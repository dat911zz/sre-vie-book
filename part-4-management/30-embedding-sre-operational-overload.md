# Chương 30. Đặt một SRE vào để Phục hồi từ Quá tải Vận hành (Embedding an SRE to Recover from Operational Overload)

> **Nguyên bản:** [Chapter 30 - Embedding an SRE to Recover from Operational Overload](https://sre.google/sre-book/operational-overload/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Đội biên tập Softdreams RnD)

---

*Tác giả:* Randall Bosetti
*Biên tập:* Diane Bates

Đây là chính sách tiêu chuẩn cho các đội SRE của Google để chia đều thời gian của họ giữa các dự án và công việc vận hành phản ứng (reactive ops). Trong thực tế, sự cân bằng này có thể bị phá vỡ trong nhiều tháng do sự gia tăng số lượng ticket hàng ngày. Một lượng công việc vận hành nặng nề đặc biệt nguy hiểm vì đội SRE có thể kiệt sức hoặc không thể đạt được tiến bộ trong công việc dự án. Khi một đội phải phân bổ một phần thời gian không cân đối để giải quyết các ticket với cái giá là dành thời gian cải thiện dịch vụ, tính khả dụng mở rộng (scalability) và độ tin cậy bị ảnh hưởng.

Một cách để giảm nhẹ gánh nặng này là chuyển một SRE sang đội bị quá tải trong một thời gian tạm thời. Khi được embed vào một đội, SRE tập trung vào việc cải thiện các thực hành của đội thay vì đơn thuần giúp đội làm trống hàng đợi ticket. SRE quan sát thói quen hàng ngày của đội và đưa ra các khuyến nghị để cải thiện các thực hành của họ. Sự tham vấn này mang cho đội một góc nhìn mới mẻ về các thói quen của nó, điều mà các thành viên đội không thể tự cung cấp.

Khi bạn đang sử dụng cách tiếp cận này, không cần thiết phải chuyển nhiều hơn một kỹ sư. Hai SRE không nhất thiết tạo ra kết quả tốt hơn và trên thực tế có thể gây ra các vấn đề nếu đội phản ứng một cách phòng thủ.

Nếu bạn đang bắt đầu đội SRE đầu tiên của mình, cách tiếp cận được phác thảo trong chương này sẽ giúp bạn tránh trở thành một đội vận hành chỉ tập trung vào một vòng trực ticket. Nếu bạn quyết định đặt bản thân hoặc một trong những cấp dưới của mình vào một đội, hãy dành thời gian xem xét lại các thực hành và triết lý SRE trong [mở đầu của Ben Treynor Sloss](https://sre.google/sre-book/introduction/) và tài liệu về giám sát trong [Giám sát Các hệ thống Phân tán](https://sre.google/sre-book/monitoring-distributed-systems/).

Các mục tiếp theo cung cấp sự hướng dẫn cho SRE sẽ được đặt vào một đội.

## Giai đoạn 1: Học Dịch vụ và Lấy Ngữ cảnh (Phase 1: Learn the Service and Get Context)

Công việc của bạn trong khi được đặt vào với đội là diễn đạt tại sao các quy trình và thói quen đóng góp vào, hoặc làm suy giảm, tính khả dụng mở rộng của dịch vụ. Nhắc đội rằng nhiều ticket hơn không nên đòi hỏi nhiều SRE hơn: mục tiêu của mô hình SRE là chỉ giới thiệu thêm con người khi thêm nhiều độ phức tạp được thêm vào hệ thống. Thay vào đó, hãy cố gắng thu hút sự chú ý vào cách các thói quen làm việc lành mạnh giảm thời gian dành cho các ticket. Việc làm như vậy quan trọng như việc chỉ ra các cơ hội bị bỏ lỡ cho việc tự động hóa hoặc đơn giản hóa dịch vụ.

### Ops Mode Versus Nonlinear Scaling (Chế độ Ops So với mở rộng phi tuyến tính)

Thuật ngữ *ops mode* (chế độ vận hành) đề cập đến một phương pháp nhất định để giữ cho một dịch vụ chạy. Các công việc tăng lên theo kích thước của dịch vụ. Ví dụ, một dịch vụ cần một cách để tăng số lượng các máy ảo (VM) đã được cấu hình khi nó tăng lên. Một đội ở ops mode phản ứng bằng cách có thêm quản trị viên để quản lý những VM đó. SRE thay vào đó tập trung vào việc viết phần mềm hoặc loại bỏ các mối lo ngại về khả năng mở rộng, để số lượng người cần để chạy một dịch vụ không tăng như một hàm của tải trên dịch vụ.

Các đội trượt vào ops mode có thể tin rằng scale (quy mô) không quan trọng đối với họ ("dịch vụ của tôi nhỏ bé"). Shadow (theo dõi) một phiên on-call để xác định xem đánh giá đó có đúng không, vì yếu tố scale ảnh hưởng đến chiến lược của bạn.

Nếu dịch vụ chính quan trọng đối với business (kinh doanh) nhưng thực sự lại rất nhỏ (bao gồm ít tài nguyên hoặc độ phức tạp thấp), hãy tập trung nhiều hơn vào các cách mà cách tiếp cận hiện tại của đội ngăn họ cải thiện độ tin cậy của dịch vụ. Hãy nhớ rằng công việc của bạn là làm cho dịch vụ hoạt động, chứ không phải để che chở cho đội phát triển khỏi các cảnh báo.

Mặt khác, nếu dịch vụ mới chỉ bắt đầu, hãy tập trung vào các cách để chuẩn bị cho đội trước sự tăng trưởng bùng nổ. Một dịch vụ 100 request/giây có thể trở thành một dịch vụ 10k request/giây trong một năm.

## Xác định các Nguồn gây Căng thẳng Lớn nhất (Identify the Largest Sources of Stress)

Các đội SRE đôi khi rơi vào ops mode vì họ tập trung vào cách giải quyết các trường hợp khẩn cấp nhanh chóng thay vì cách giảm số lượng các trường hợp khẩn cấp. Việc mặc định vào ops mode thường xảy ra để đáp lại một áp lực áp đảo, *thật hoặc tưởng tượng*. Sau khi bạn đã học đủ về dịch vụ để đặt ra những câu hỏi khó về thiết kế và triển khai của nó, hãy dành thời gian ưu tiên hóa các outage khác nhau của dịch vụ theo mức độ ảnh hưởng của chúng đối với mức căng thẳng của đội. Hãy nhớ rằng, do góc nhìn và lịch sử của đội, một số vấn đề hoặc outage rất nhỏ có thể tạo ra một lượng căng thẳng không tương xứng.

## Xác định các Mồi lửa (Identify Kindling)

Một khi bạn [xác định các vấn đề lớn nhất hiện có của một đội](https://sre.google/workbook/overload/), hãy chuyển sang các trường hợp khẩn cấp đang chờ xảy ra. Đôi khi các trường hợp khẩn cấp sắp tới đến dưới hình thức của một hệ thống con mới không được thiết kế để tự quản lý. Các nguồn khác bao gồm:

Các khoảng trống kiến thức

Trong các đội lớn, mọi người có thể chuyên môn hóa quá mức mà không có hệ quả ngay lập tức. Khi một người chuyên môn hóa, họ chịu rủi ro hoặc không có kiến thức rộng mà họ cần để thực hiện hỗ trợ on-call hoặc cho phép các thành viên đội bỏ qua các phần quan trọng của hệ thống mà họ sở hữu.

Các dịch vụ do SRE phát triển đang âm thầm tăng lên tầm quan trọng

Những dịch vụ này thường không nhận được sự chú ý cẩn thận giống như các ra mắt tính năng mới vì chúng nhỏ hơn về quy mô và được ngầm đồng ý bởi ít nhất một SRE.

Sự phụ thuộc mạnh vào "thứ lớn tiếp theo"

Mọi người có thể bỏ qua các vấn đề trong nhiều tháng vì họ tin rằng giải pháp mới ở phía chân trời sẽ làm cho các sửa chữa tạm thời trở nên không cần thiết.

Các cảnh báo chung không được chẩn đoán bởi cả đội dev (phát triển) lẫn các SRE

Những cảnh báo như vậy thường được triage (phân loại) là *tạm thời*, nhưng vẫn làm xao nhãng các đồng nghiệp của bạn khỏi việc sửa các vấn đề thực. Hoặc hãy điều tra đầy đủ những cảnh báo như vậy, hoặc sửa các rule (quy tắc) cảnh báo.

Bất kỳ dịch vụ nào vừa là đối tượng của các khiếu nại từ các client (khách hàng) của bạn vừa thiếu một SLI/SLO/SLA (chỉ số/mục tiêu/thỏa thuận mức dịch vụ) chính thức

Xem [Service Level Objectives](https://sre.google/sre-book/service-level-objectives/) để thảo luận về các SLI, SLO, và SLA.

Bất kỳ dịch vụ nào có một kế hoạch năng lực thực chất là "Thêm các server: các server của chúng tôi tối qua sắp hết bộ nhớ (memory)"

Các kế hoạch năng lực nên đủ nhìn về phía trước. Nếu mô hình hệ thống của bạn dự đoán rằng các server cần 2 GB, một loadtest (kiểm thử tải) vượt qua trong ngắn hạn (cho thấy 1.99 GB trong lần chạy cuối) không nhất thiết có nghĩa là năng lực hệ thống của bạn ở trạng thái đủ tốt.

Các postmortem chỉ có các action item để rollback các thay đổi cụ thể đã gây ra một outage

Ví dụ, "Đổi timeout (thời gian chờ) streaming (luồng) trở lại thành 60 giây," thay vì "Tìm ra tại sao đôi khi mất 60 giây để lấy megabyte đầu tiên của các video quảng cáo của chúng tôi."

Bất kỳ thành phần phục vụ quan trọng nào mà các SRE hiện có phản hồi các câu hỏi bằng cách nói, "Chúng tôi không biết gì về điều đó; các dev sở hữu nó"

Để cung cấp hỗ trợ on-call (trực sự kiện) chấp nhận được cho một thành phần, bạn nên ít nhất biết được các hậu quả khi nó hỏng và mức độ khẩn cấp cần thiết để sửa các vấn đề.

## Giai đoạn 2: Chia sẻ Ngữ cảnh (Phase 2: Sharing Context)

Sau khi xác định phạm vi các động lực và các điểm đau của đội, hãy đặt nền móng cho cải tiến thông qua các thực hành tốt nhất như postmortem và bằng cách xác định các nguồn toil (việc chân tay) và cách tốt nhất để đối phó với chúng.

## Viết một Postmortem tốt cho Đội (Write a Good Postmortem for the Team)

Postmortem cung cấp nhiều hiểu biết sâu sắc vào sự suy luận tập thể của một đội. Các postmortem do các đội không lành mạnh thực hiện thường kém hiệu quả. Một số thành viên đội có thể coi postmortem là hình phạt, hoặc thậm chí vô dụng. Mặc dù bạn có thể bị cám dỗ xem xét các kho lưu trữ postmortem và để lại các bình luận để cải thiện, việc làm như vậy không giúp đội. Thay vào đó, bài tập này có thể đặt đội vào thế phòng thủ.

Thay vì cố gắng sửa chữa các sai lầm trước đó, hãy tiếp nhận trách nhiệm cho postmortem tiếp theo. *Sẽ* có một outage trong khi bạn được đặt vào. Nếu bạn không phải là người on-call, hãy kết hợp với SRE on-call để viết một postmortem tuyệt vời, không đổ lỗi. Tài liệu này là một cơ hội để chứng minh làm thế nào một sự chuyển dịch sang mô hình SRE mang lại lợi ích cho đội bằng cách làm cho các sửa lỗi (bug fix) vĩnh viễn hơn. Các sửa lỗi vĩnh viễn hơn giảm tác động của các outage đối với thời gian của các thành viên đội.

Như đã đề cập, bạn có thể gặp phải những phản hồi như "Tại sao là tôi?" Phản hồi này đặc biệt có khả năng xảy ra khi một đội tin rằng quy trình postmortem là trả đũa. Thái độ này đến từ việc tuân theo Lý thuyết Táo Tồi (Bad Apple Theory): hệ thống đang hoạt động ổn, và nếu chúng tôi loại bỏ tất cả những quả táo tồi cùng các sai lầm của chúng, hệ thống sẽ tiếp tục ổn. Lý thuyết Táo Tồi được chứng minh là sai, như bằng chứng [[Dek14]](https://sre.google/sre-book/bibliography#Dek14) từ một số ngành, bao gồm cả an toàn hàng không, cho thấy. Bạn nên chỉ ra sự sai lầm này. Cách diễn đạt hiệu quả nhất cho một postmortem là nói, "Những sai lầm là không thể tránh khỏi trong bất kỳ hệ thống nào có nhiều tương tác tinh tế. Bạn đã on-call, và tôi tin bạn sẽ đưa ra các quyết định đúng đắn với thông tin đúng. Tôi muốn bạn viết ra điều bạn đang nghĩ ở từng thời điểm, để chúng tôi có thể tìm ra hệ thống đã đánh lạc hướng bạn ở đâu, và ở đâu các đòi hỏi nhận thức quá cao."

## Sắp xếp các Cháy theo Loại (Sort Fires According to Type)

Trong mô hình được đơn giản hóa cho tiện lợi này, có hai loại cháy:

-   Một số cháy không nên tồn tại. Chúng gây ra điều thường được gọi là công việc ops (vận hành) hoặc toil (xem [Loại bỏ Toil](https://sre.google/sre-book/eliminating-toil/)).
-   Các cháy khác gây ra căng thẳng và/hoặc gõ điên cuồng thực sự là một phần của công việc.

Trong cả hai trường hợp, đội cần xây dựng các công cụ để kiểm soát việc đốt.

Sắp xếp các cháy của đội thành toil và không-toil. Khi bạn hoàn thành, hãy trình bày danh sách cho đội và giải thích rõ ràng tại sao mỗi cháy là công việc nên được tự động hóa hoặc chi phí phụ chấp nhận được để vận hành dịch vụ.

## Giai đoạn 3: Thúc đẩy Thay đổi (Phase 3: Driving Change)

Sức khỏe của đội là một quá trình. Vì vậy, đó không phải là thứ bạn có thể giải quyết bằng nỗ lực anh hùng. Để đảm bảo rằng đội có thể tự điều chỉnh, bạn có thể giúp họ xây dựng một mô hình tâm trí tốt cho một sự tham gia SRE lý tưởng.

#### Lưu ý (Note)

Con người khá giỏi trong việc giữ cân bằng nội môi (homeostasis), nên *tập trung vào việc tạo ra (hoặc khôi phục) các điều kiện ban đầu đúng và dạy bộ các nguyên lý nhỏ cần thiết để đưa ra các lựa chọn lành mạnh*.

## Bắt đầu với Cơ bản (Start with the Basics)

Các đội đang vật lộn với sự khác biệt giữa mô hình SRE và mô hình ops truyền thống thường không thể diễn đạt *tại sao* một số khía cạnh nhất định của code, quy trình, hoặc văn hóa của đội làm phiền họ. Thay vì cố gắng giải quyết từng vấn đề trong số này một điểm một, hãy tiến lên từ các nguyên lý được phác thảo trong các chương [Giới thiệu](https://sre.google/sre-book/introduction/) và [Giám sát Các hệ thống Phân tán](https://sre.google/sre-book/monitoring-distributed-systems/).

Mục tiêu đầu tiên của bạn cho đội nên là viết một service level objective (SLO — mục tiêu mức dịch vụ), nếu một SLO chưa tồn tại. SLO quan trọng vì nó cung cấp một phép đo định lượng về tác động của các outage, ngoài việc một thay đổi quy trình có thể quan trọng như thế nào. Một SLO có lẽ là đòn bẩy quan trọng duy nhất để chuyển một đội từ công việc vận hành phản ứng sang một sự tập trung SRE lành mạnh, dài hạn. Nếu sự thỏa thuận này thiếu, không lời khuyên khác trong chương này sẽ hữu ích. *Nếu bạn thấy mình trong một đội không có SLO, trước tiên hãy đọc [Service Level Objectives](https://sre.google/sre-book/service-level-objectives/), sau đó đưa các tech lead (quyền kỹ thuật) và quản lý vào một phòng và bắt đầu phân xử.*

## Có sự Giúp đỡ dọn Dọn các Mồi lửa (Get Help Clearing Kindling)

Bạn có thể có thôi thúc mạnh mẽ để đơn thuần sửa các vấn đề mà bạn xác định. Hãy kiềm chế thôi thúc tự sửa những vấn đề này, vì việc làm như vậy củng cố ý tưởng rằng "việc thực hiện các thay đổi là dành cho những người khác." Thay vào đó, hãy thực hiện các bước sau:

1.  Tìm một công việc hữu ích mà có thể hoàn thành bởi một thành viên đội.
2.  Giải thích rõ ràng làm thế nào công việc này giải quyết một vấn đề từ postmortem *theo cách vĩnh viễn*. Ngay cả các đội lành mạnh khác cũng có thể tạo ra các action item kém tầm nhìn.
3.  Đóng vai trò là người review (xem xét) cho các thay đổi code và các sửa đổi tài liệu.
4.  Lặp lại cho hai hoặc ba vấn đề.

Khi bạn xác định một vấn đề bổ sung, hãy đặt nó vào một bug report (báo cáo lỗi) hoặc một doc (tài liệu) để đội tham khảo. Việc làm như vậy phục vụ cho mục đích kép là phân phối thông tin và khuyến khích các thành viên đội viết doc (thường là nạn nhân đầu tiên của áp lực hạn chót). Luôn luôn giải thích suy luận của bạn, và nhấn mạnh rằng tài liệu tốt đảm bảo rằng đội không lặp lại các sai lầm cũ trong một ngữ cảnh hơi mới.

## Giải thích Suy luận của Bạn (Explain Your Reasoning)

Khi đội phục hồi đà và nắm bắt các cơ bản của các thay đổi bạn đề xuất, hãy chuyển sang đối phó với các quyết định thường nhật vốn đã dẫn đến [quá tải vận hành.](https://sre.google/sre-book/dealing-with-interrupts/) Hãy chuẩn bị cho việc thực hiện này bị thách thức. Nếu may mắn, sự thách thức sẽ theo đường nét "Giải thích tại sao. Ngay bây giờ. Giữa cuộc họp production hàng tuần."

Nếu bạn không may mắn, không ai đòi hỏi một lời giải thích. Lách hoàn toàn vấn đề này bằng cách đơn thuần giải thích tất cả các quyết định của bạn, bất kể ai đó có yêu cầu hay không. Hãy tham chiếu đến các cơ bản củng cố các đề xuất của bạn. Việc làm như vậy giúp xây dựng mô hình tâm trí của đội. *Sau khi bạn rời đi, đội nên có thể dự đoán bình luận của bạn về một thiết kế hoặc changelist sẽ ra sao.* Nếu bạn không giải thích suy luận của mình, hoặc giải thích kém, có rủi ro đội sẽ đơn thuần bắt chước hành vi cẩu thả đó, nên hãy rõ ràng.

Các ví dụ về một lời giải thích đầy đủ cho quyết định của bạn:

-   "Tôi không phản đối release (phát hành) gần nhất vì các test (kiểm thử) tệ. Tôi phản đối vì error budget (ngân sách lỗi) mà chúng tôi đặt cho các release đã cạn kiệt."
-   "Các release cần an toàn để rollback vì SLO của chúng tôi chặt. Đáp ứng SLO đó đòi hỏi rằng mean time to recovery (thời gian phục hồi trung bình) phải nhỏ, nên chẩn đoán sâu trước khi rollback là không thực tế."

Các ví dụ về một lời giải thích không đầy đủ cho quyết định của bạn:

-   "Tôi không nghĩ việc mỗi server tạo config định tuyến của nó là an toàn, vì chúng tôi không thể thấy nó."

Quyết định này có lẽ là đúng, nhưng suy luận thì kém (hoặc được giải thích kém). Đội không thể đọc tâm trí của bạn, nên họ rất có thể sẽ bắt chước suy luận kém được quan sát. Thay vào đó, hãy thử "\[…] không an toàn vì một bug trong code đó có thể gây ra một sự thất bại liên quan trên toàn dịch vụ, và code bổ sung là một nguồn bug có thể làm chậm rollback."

-   "Tự động hóa nên từ bỏ nếu nó gặp một deployment (triển khai) xung đột."

Giống như ví dụ trước, lời giải thích này có lẽ đúng, nhưng không đầy đủ. Thay vào đó, hãy thử "\[…] vì chúng tôi đang đưa ra giả định đơn giản hóa rằng tất cả các thay đổi đi qua tự động hóa, và rõ ràng có điều gì đó đã vi phạm quy tắc đó. Nếu điều này xảy ra thường xuyên, chúng tôi nên xác định và loại bỏ các nguồn của các thay đổi không có tổ chức."

## Đặt các Câu hỏi Dẫn dắt (Ask Leading Questions)

Các câu hỏi dẫn dắt không phải là các câu hỏi gán nợ (loaded questions). Khi nói chuyện với đội SRE, hãy cố gắng đặt các câu hỏi theo cách khuyến khích mọi người suy nghĩ về các nguyên lý cơ bản. Điều này đặc biệt có giá trị cho *bạn* để mô hình hóa hành vi này, vì theo định nghĩa, một đội ở ops mode bác bỏ loại suy luận này từ chính các thành viên của nó. Một khi bạn đã dành một ít thời gian giải thích suy luận của mình cho các câu hỏi chính sách khác nhau, thực hành này củng cố sự hiểu biết của đội về triết lý SRE.

Các ví dụ về các câu hỏi dẫn dắt:

-   "Tôi thấy rằng cảnh báo TaskFailures (các Thất bại Task) fire (bắn) thường xuyên, nhưng các kỹ sư on-call thường không làm gì để phản hồi cảnh báo. Điều này ảnh hưởng đến SLO như thế nào?"
-   "Quy trình turnup (khởi tạo) này trông khá phức tạp. Bạn có biết tại sao có nhiều file config (cấu hình) đến thế cần cập nhật khi tạo một instance (bản chạy) mới của dịch vụ không?"

Các ví dụ phản đề về các câu hỏi dẫn dắt:

-   "Chuyện gì đang xảy ra với tất cả các release cũ, đình trệ này?"
-   "Tại sao cái Frobnitzer lại làm nhiều việc đến thế?"

## Kết luận (Conclusion)

Việc tuân thủ các tín điều được phác thảo trong chương này cung cấp cho một đội SRE những thứ sau:

-   Một góc nhìn kỹ thuật, có lẽ định lượng, về lý do tại sao họ nên thay đổi.
-   Một ví dụ mạnh mẽ về việc thay đổi trông như thế nào.
-   Một lời giải thích logic cho phần lớn "sự khôn ngoan dân gian" (folk wisdom) được sử dụng bởi SRE.
-   Các nguyên lý cốt lõi cần thiết để đối phó với các tình huống mới theo cách có thể mở rộng.

Công việc cuối cùng của bạn là viết một báo cáo sau hành động (after-action report). Báo cáo này nên lặp lại góc nhìn, các ví dụ và lời giải thích của bạn. Nó cũng nên cung cấp một số action item cho đội để đảm bảo họ thực hành những gì bạn đã dạy. Bạn có thể tổ chức báo cáo như một postvitam,<sup>[1](#fn1)</sup> giải thích các quyết định quan trọng ở mỗi bước đã dẫn đến thành công.

Phần lớn sự tham gia giờ đây đã hoàn thành. Khi nhiệm vụ được embed của bạn kết thúc, bạn nên vẫn sẵn sàng cho các review thiết kế và code. Hãy để mắt đến đội trong vài tháng tiếp theo để xác nhận rằng họ đang từ từ cải thiện lập kế hoạch năng lực, phản ứng khẩn cấp và các quy trình rollout.

<a id="fn1"></a>[1](#fn1) Ngược lại với một postmortem.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
