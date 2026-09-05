# Chương 30. Đặt một SRE vào để Phục hồi từ Quá tải Vận hành (Embedding an SRE to Recover from Operational Overload)

> **Nguyên bản:** [Chapter 30 - Embedding an SRE to Recover from Operational Overload](https://sre.google/sre-book/operational-overload/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Randall Bosetti
*Biên tập:* Diane Bates

Đây là chính sách tiêu chuẩn của các đội SRE tại Google, nhằm chia đều thời gian giữa các dự án và công việc vận hành phản ứng (reactive ops). Tuy nhiên, trong thực tế, sự cân bằng này có thể bị phá vỡ trong nhiều tháng do số lượng ticket hàng ngày tăng cao. Khối lượng công việc vận hành quá lớn đặc biệt nguy hiểm, vì đội SRE có thể kiệt sức hoặc không đạt được tiến bộ trong các dự án. Khi một đội phải dành một phần thời gian không cân đối để xử lý ticket, dẫn đến thiếu thời gian cải thiện dịch vụ, khả năng mở rộng (scalability) và độ tin cậy sẽ bị ảnh hưởng.

Một cách để giảm nhẹ gánh nặng này là chuyển một SRE sang đội bị quá tải trong một thời gian tạm thời. Khi được embed vào một đội, SRE tập trung vào việc cải thiện các thực hành của đội thay vì đơn thuần giúp đội làm trống hàng đợi ticket. SRE quan sát thói quen hàng ngày của đội và đưa ra các khuyến nghị để cải thiện các thực hành của họ. Sự tham vấn này mang lại cho đội một góc nhìn mới mẻ về các thói quen của nó, điều mà các thành viên đội không thể tự cung cấp.

Khi áp dụng cách tiếp cận này, không cần điều động nhiều hơn một kỹ sư. Hai SRE không nhất thiết mang lại kết quả tốt hơn; trên thực tế, nếu đội phản ứng theo hướng phòng thủ, điều này còn có thể gây ra các vấn đề.

Nếu bạn đang xây dựng đội SRE đầu tiên, cách tiếp cận trong chương này sẽ giúp bạn tránh biến đội thành một nhóm vận hành chỉ xoay quanh việc xử lý ticket. Nếu bạn hoặc một cấp dưới của mình tham gia vào đội, hãy dành thời gian xem xét lại các thực hành và triết lý SRE trong [mở đầu của Ben Treynor Sloss](https://sre.google/sre-book/introduction/) và tài liệu về giám sát trong [Giám sát Các hệ thống Phân tán](https://sre.google/sre-book/monitoring-distributed-systems/).

Các mục tiếp theo cung cấp sự hướng dẫn cho SRE sẽ được đặt vào một đội.

## Giai đoạn 1: Học Dịch vụ và Lấy Ngữ cảnh (Phase 1: Learn the Service and Get Context)

Khi được đưa vào đội, nhiệm vụ của bạn là chỉ ra vì sao các quy trình và thói quen hiện tại đang hỗ trợ hay cản trở khả năng mở rộng của dịch vụ. Hãy nhắc đội rằng số lượng ticket tăng không có nghĩa là cần thêm SRE: mục tiêu của mô hình SRE là chỉ bổ sung nhân sự khi độ phức tạp của hệ thống tăng lên. Thay vào đó, hãy hướng sự chú ý vào cách các thói quen làm việc lành mạnh giúp giảm thời gian xử lý ticket. Việc này quan trọng ngang với việc chỉ ra các cơ hội bị bỏ lỡ để tự động hóa hoặc đơn giản hóa dịch vụ.

### Ops Mode Versus Nonlinear Scaling (Chế độ Ops So với mở rộng phi tuyến tính)

Thuật ngữ *ops mode* (chế độ vận hành) chỉ một cách tiếp cận nhất định để duy trì hoạt động của dịch vụ. Khối lượng công việc trong chế độ này tăng theo quy mô dịch vụ. Chẳng hạn, khi một dịch vụ mở rộng, nó cần cơ chế để tăng số lượng máy ảo (VM) đã cấu hình. Đội ngũ theo *ops mode* sẽ phản ứng bằng cách bổ sung quản trị viên để quản lý các VM đó. Ngược lại, SRE tập trung vào việc viết phần mềm hoặc loại bỏ các điểm nghẽn về khả năng mở rộng, nhằm đảm bảo số người cần vận hành dịch vụ không tăng theo hàm của tải trên dịch vụ.

Các đội chuyển sang chế độ vận hành (ops mode) thường cho rằng quy mô (scale) không liên quan đến mình (“dịch vụ của tôi nhỏ bé”). Hãy theo dõi (shadow) một phiên on-call để kiểm chứng nhận định này, vì yếu tố scale ảnh hưởng trực tiếp đến chiến lược của bạn.

Nếu dịch vụ chính đóng vai trò quan trọng đối với kinh doanh nhưng quy mô thực tế lại rất nhỏ (sử dụng ít tài nguyên hoặc có độ phức tạp thấp), hãy tập trung vào những điểm mà cách tiếp cận hiện tại của đội đang cản trở việc cải thiện độ tin cậy của dịch vụ. Hãy nhớ rằng nhiệm vụ của bạn là đảm bảo dịch vụ hoạt động, chứ không phải che chắn cho đội phát triển khỏi các cảnh báo.

Ngược lại, nếu dịch vụ mới chỉ vừa khởi động, hãy tập trung vào các biện pháp chuẩn bị cho đội trước sự tăng trưởng bùng nổ. Một dịch vụ xử lý 100 request/giây có thể tăng lên 10k request/giây chỉ trong vòng một năm.

## Xác định các Nguồn gây Căng thẳng Lớn nhất (Identify the Largest Sources of Stress)

Các đội SRE đôi khi sa vào ops mode vì mải tập trung xử lý sự cố khẩn cấp nhanh chóng, thay vì tìm cách giảm số lượng sự cố. Việc này thường xảy ra khi đội phải đối mặt với một áp lực áp đảo, *thật hoặc tưởng tượng*. Khi đã hiểu đủ về dịch vụ để đặt ra những câu hỏi khó về thiết kế và triển khai, hãy dành thời gian ưu tiên hóa các outage khác nhau dựa trên mức độ ảnh hưởng của chúng đối với áp lực của đội. Cần lưu ý rằng, do góc nhìn và lịch sử của đội, một số vấn đề hoặc outage rất nhỏ có thể gây ra mức căng thẳng không tương xứng.

## Xác định các Mồi lửa (Identify Kindling)

Sau khi [xác định các vấn đề lớn nhất hiện có của một đội](https://sre.google/workbook/overload/), hãy chuyển sang các trường hợp khẩn cấp đang chờ xảy ra. Đôi khi, các trường hợp khẩn cấp sắp tới đến dưới hình thức của một hệ thống con mới không được thiết kế để tự quản lý. Các nguồn khác bao gồm:

Các khoảng trống kiến thức

Trong các đội lớn, mọi người có thể chuyên môn hóa quá mức mà không có hệ quả ngay lập tức. Khi một người chuyên môn hóa, họ có nguy cơ thiếu kiến thức rộng cần để thực hiện hỗ trợ on-call, hoặc khiến các thành viên khác trong đội bỏ qua các phần quan trọng của hệ thống mà họ sở hữu.

Các dịch vụ do SRE phát triển đang âm thầm tăng lên tầm quan trọng

Những dịch vụ này thường ít được quan tâm kỹ lưỡng như các lần ra mắt tính năng mới, do quy mô nhỏ hơn và đã được ít nhất một SRE ngầm đồng ý.

Sự phụ thuộc mạnh vào "thứ lớn tiếp theo"

Mọi người có thể bỏ qua các vấn đề trong nhiều tháng vì tin rằng một giải pháp mới sắp xuất hiện sẽ khiến các biện pháp sửa chữa tạm thời trở nên không cần thiết.

Các cảnh báo chung không được chẩn đoán bởi cả đội dev (phát triển) lẫn các SRE

Những cảnh báo như vậy thường được triage (phân loại) là *tạm thời*, nhưng vẫn khiến đồng nghiệp của bạn xao nhãng khỏi việc xử lý các vấn đề thực sự. Hãy hoặc điều tra kỹ lưỡng những cảnh báo này, hoặc chỉnh sửa các rule (quy tắc) cảnh báo.

Bất kỳ dịch vụ nào vừa nhận khiếu nại từ client (khách hàng) của bạn, vừa thiếu SLI/SLO/SLA (chỉ số/mục tiêu/thỏa thuận mức dịch vụ) chính thức

Xem [Service Level Objectives](https://sre.google/sre-book/service-level-objectives/) để thảo luận về các SLI, SLO, và SLA.

Bất kỳ dịch vụ nào có một kế hoạch năng lực thực chất là "Thêm các server: các server của chúng tôi tối qua sắp hết bộ nhớ (memory)"

Kế hoạch năng lực cần nhìn xa hơn về phía trước. Nếu mô hình hệ thống dự báo các server cần 2 GB, thì một loadtest (kiểm thử tải) vượt qua trong ngắn hạn (chỉ ghi nhận 1.99 GB ở lần chạy cuối) chưa chắc đã chứng tỏ năng lực hệ thống đang ở mức đủ tốt.

Các postmortem chỉ có các action item để rollback các thay đổi cụ thể đã gây ra một outage

Ví dụ, “Đổi timeout (thời gian chờ) streaming (luồng) trở lại thành 60 giây,” thay vì “Tìm ra tại sao đôi khi mất 60 giây để lấy megabyte đầu tiên của các video quảng cáo của chúng tôi.”

Bất kỳ thành phần phục vụ quan trọng nào mà các SRE hiện có phản hồi các câu hỏi bằng cách nói, "Chúng tôi không biết gì về điều đó; các dev sở hữu nó"

Để hỗ trợ on-call (trực sự cố) một thành phần ở mức chấp nhận được, bạn cần biết ít nhất hậu quả khi nó hỏng và mức độ khẩn cấp cần thiết để khắc phục.

## Giai đoạn 2: Chia sẻ Ngữ cảnh (Phase 2: Sharing Context)

Sau khi xác định phạm vi các động lực và điểm đau của đội, hãy đặt nền móng cho cải tiến bằng cách áp dụng các thực hành tốt nhất như postmortem, đồng thời xác định các nguồn toil (việc chân tay) và cách xử lý chúng hiệu quả nhất.

## Viết một Postmortem tốt cho Đội (Write a Good Postmortem for the Team)

Postmortem cho thấy cách một đội suy luận tập thể. Các postmortem do những đội không lành mạnh thực hiện thường kém hiệu quả. Một số thành viên có thể coi postmortem là hình phạt, hoặc thậm chí vô dụng. Dù bạn có thể muốn xem qua các kho lưu trữ postmortem và để lại bình luận nhằm cải thiện, việc đó không giúp ích gì cho đội. Thay vào đó, bài tập này có thể khiến đội rơi vào thế phòng thủ.

Thay vì mải mê vá lỗi trước đó, hãy tập trung vào việc viết postmortem cho sự cố tiếp theo. *Chắc chắn* sẽ có một lần outage xảy ra khi bạn được giao nhiệm vụ. Nếu không phải là người on-call, hãy phối hợp với SRE on-call để viết một postmortem tuyệt vời, không đổ lỗi. Tài liệu này là cơ hội để chứng minh việc chuyển sang mô hình SRE mang lại lợi ích cho đội bằng cách giúp các bản sửa lỗi (bug fix) trở nên vĩnh viễn hơn. Những bản sửa lỗi bền vững này sẽ giảm bớt tác động của các lần outage lên thời gian của các thành viên trong đội.

Như đã đề cập, bạn có thể gặp phải những phản hồi như "Tại sao là tôi?" Phản hồi này đặc biệt có khả năng xảy ra khi một đội tin rằng quy trình postmortem là trả đũa. Thái độ này đến từ việc tuân theo Lý thuyết Táo Tồi (Bad Apple Theory): hệ thống đang hoạt động ổn, và nếu chúng tôi loại bỏ tất cả những quả táo tồi cùng các sai lầm của chúng, hệ thống sẽ tiếp tục ổn. Lý thuyết Táo Tồi được chứng minh là sai, như bằng chứng [[Dek14]](https://sre.google/sre-book/bibliography#Dek14) từ một số ngành, bao gồm cả an toàn hàng không, cho thấy. Bạn nên chỉ ra sự sai lầm này. Cách diễn đạt hiệu quả nhất cho một postmortem là nói, "Những sai lầm là không thể tránh khỏi trong bất kỳ hệ thống nào có nhiều tương tác tinh tế. Bạn đã on-call, và tôi tin bạn sẽ đưa ra các quyết định đúng đắn với thông tin đúng. Tôi muốn bạn viết ra điều bạn đang nghĩ ở từng thời điểm, để chúng tôi có thể tìm ra hệ thống đã đánh lạc hướng bạn ở đâu, và ở đâu các đòi hỏi nhận thức quá cao."

## Sắp xếp các Cháy theo Loại (Sort Fires According to Type)

Trong mô hình được đơn giản hóa cho tiện lợi này, có hai loại cháy:

-   Một số cháy không nên tồn tại. Chúng gây ra những gì thường được gọi là công việc ops (vận hành) hoặc toil (xem [Loại bỏ Toil](https://sre.google/sre-book/eliminating-toil/)).
-   Các cháy khác gây ra căng thẳng và/hoặc gõ điên cuồng thực sự là một phần của công việc.

Trong cả hai trường hợp, đội cần xây dựng các công cụ để kiểm soát việc đốt.

Hãy phân loại các cháy của đội thành toil và không-toil. Khi hoàn thành, trình bày danh sách này cho đội và giải thích rõ lý do mỗi cháy là công việc nên được tự động hóa hoặc overhead (công việc phụ) chấp nhận được để vận hành dịch vụ.

## Giai đoạn 3: Thúc đẩy Thay đổi (Phase 3: Driving Change)

Sức khỏe của đội là một quá trình, nên không thể giải quyết bằng những nỗ lực anh hùng. Để đội có thể tự điều chỉnh, bạn hãy giúp họ xây dựng mô hình tâm trí đúng đắn về sự tham gia SRE lý tưởng.

#### Lưu ý (Note)

Con người khá giỏi trong việc giữ cân bằng nội môi (homeostasis), nên *tập trung vào việc tạo ra (hoặc khôi phục) các điều kiện ban đầu đúng và dạy bộ các nguyên lý nhỏ cần thiết để đưa ra các lựa chọn lành mạnh*.

## Bắt đầu với Cơ bản (Start with the Basics)

Các đội đang chật vật với sự khác biệt giữa mô hình SRE và mô hình ops truyền thống thường không thể diễn đạt rõ *tại sao* một số khía cạnh nhất định của code, quy trình, hoặc văn hóa của đội lại khiến họ khó chịu. Thay vì cố gắng giải quyết riêng lẻ từng vấn đề một trong số này, hãy tiến lên từ các nguyên lý được phác thảo trong các chương [Giới thiệu](https://sre.google/sre-book/introduction/) và [Giám sát Các hệ thống Phân tán](https://sre.google/sre-book/monitoring-distributed-systems/).

Mục tiêu đầu tiên của bạn cho đội nên là viết một service level objective (SLO — mục tiêu mức dịch vụ), nếu một SLO chưa tồn tại. SLO quan trọng vì nó cung cấp một phép đo định lượng về tác động của các outage, ngoài việc một thay đổi quy trình có thể quan trọng như thế nào. Một SLO có lẽ là đòn bẩy quan trọng duy nhất để chuyển một đội từ công việc vận hành phản ứng sang một sự tập trung SRE lành mạnh, dài hạn. Nếu sự thỏa thuận này thiếu, không lời khuyên khác trong chương này sẽ hữu ích. *Nếu bạn thấy mình trong một đội không có SLO, trước tiên hãy đọc [Service Level Objectives](https://sre.google/sre-book/service-level-objectives/), sau đó đưa các tech lead (trưởng nhóm kỹ thuật) và quản lý vào một phòng và bắt đầu phân xử.*

## Nhờ Giúp đỡ Dọn các Mồi lửa (Get Help Clearing Kindling)

Bạn có thể rất muốn sửa ngay những vấn đề đã xác định. Hãy kìm lại thôi thúc đó, vì việc tự tay sửa sẽ củng cố suy nghĩ rằng “việc thực hiện các thay đổi là dành cho những người khác.” Thay vào đó, hãy thực hiện các bước sau:

1.  Tìm một công việc hữu ích mà một thành viên đội có thể hoàn thành.
2.  Giải thích rõ cách công việc này giải quyết triệt để một vấn đề từ postmortem. Ngay cả các đội vận hành tốt cũng có thể đặt ra những action item thiếu tầm nhìn.
3.  Đóng vai trò là người review (xem xét) cho các thay đổi code và các sửa đổi tài liệu.
4.  Lặp lại cho hai hoặc ba vấn đề.

Khi phát hiện thêm vấn đề, hãy ghi vào bug report (báo cáo lỗi) hoặc doc (tài liệu) để cả đội tham khảo. Cách làm này vừa giúp lan tỏa thông tin, vừa khuyến khích các thành viên viết doc (những người thường chịu áp lực deadline đầu tiên). Hãy luôn giải thích rõ suy luận của mình và nhấn mạnh rằng tài liệu tốt sẽ giúp đội tránh lặp lại những sai lầm cũ trong các tình huống tương tự.

## Giải thích Suy luận của Bạn (Explain Your Reasoning)

Khi đội đã lấy lại đà và nắm được các yếu tố cơ bản của những thay đổi bạn đề xuất, hãy chuyển sang xử lý các quyết định thường nhật vốn đã dẫn đến [quá tải vận hành.](https://sre.google/sre-book/dealing-with-interrupts/) Hãy chuẩn bị tinh thần cho việc thực hiện này bị chất vấn. Nếu may mắn, sự chất vấn sẽ chỉ dừng lại ở mức: “Giải thích lý do. Ngay bây giờ. Giữa cuộc họp production hàng tuần.”

Nếu chẳng may gặp phải tình huống không ai đòi hỏi giải thích, cách tốt nhất là chủ động trình bày rõ ràng mọi quyết định của bạn, bất kể có ai yêu cầu hay không. Hãy viện dẫn các nguyên tắc cơ bản làm nền tảng cho các đề xuất. Việc này giúp xây dựng mô hình tâm trí chung của đội. *Sau khi bạn rời đi, đội nên có thể dự đoán được bình luận của bạn về một thiết kế hoặc changelist sẽ ra sao.* Nếu bạn không giải thích suy luận của mình, hoặc giải thích không rõ ràng, đội có nguy cơ đơn thuần bắt chước hành vi cẩu thả đó, nên hãy trình bày thật rành mạch.

Các ví dụ về một lời giải thích đầy đủ cho quyết định của bạn:

-   "Tôi không phản đối release (phát hành) gần nhất vì các test (kiểm thử) tệ. Tôi phản đối vì error budget (ngân sách lỗi) mà chúng tôi đặt cho các release đã cạn kiệt."
-   "Các release cần có khả năng rollback an toàn vì SLO của chúng tôi rất chặt. Để đáp ứng SLO đó, mean time to recovery (thời gian phục hồi trung bình) phải thấp, nên việc chẩn đoán sâu trước khi rollback là không thực tế."

Các ví dụ về một lời giải thích không đầy đủ cho quyết định của bạn:

-   "Tôi không nghĩ việc mỗi server tạo config định tuyến của nó là an toàn, vì chúng tôi không thể thấy nó."

Quyết định này có lẽ là đúng, nhưng lập luận thì yếu (hoặc cách giải thích chưa rõ ràng). Vì đội không thể đọc suy nghĩ của bạn, họ rất có thể sẽ bắt chước cách lập luận kém cỏi đó. Thay vào đó, hãy thử: “\[…] không an toàn vì một bug trong code đó có thể gây ra sự cố liên quan trên toàn dịch vụ, và code bổ sung là một nguồn bug có thể làm chậm quá trình rollback.”

-   "Tự động hóa nên từ bỏ nếu nó gặp một deployment (triển khai) xung đột."

Giống như ví dụ trước, lời giải thích này có lẽ đúng, nhưng không đầy đủ. Thay vào đó, hãy thử “\[…] vì chúng tôi đang đưa ra giả định đơn giản hóa rằng tất cả các thay đổi đều đi qua tự động hóa, và rõ ràng có điều gì đó đã vi phạm quy tắc đó. Nếu điều này xảy ra thường xuyên, chúng tôi nên xác định và loại bỏ các nguồn của các thay đổi không có tổ chức.”

## Đặt các Câu hỏi Dẫn dắt (Ask Leading Questions)

Các câu hỏi dẫn dắt không phải là các câu hỏi cài bẫy (loaded questions). Khi nói chuyện với đội SRE, hãy cố gắng đặt các câu hỏi theo cách khuyến khích mọi người suy nghĩ về các nguyên lý cơ bản. Điều này đặc biệt có giá trị cho *bạn* để mô hình hóa hành vi này, vì theo định nghĩa, một đội ở ops mode bác bỏ loại suy luận này từ chính các thành viên của nó. Một khi bạn đã dành một ít thời gian giải thích suy luận của mình cho các câu hỏi chính sách khác nhau, thực hành này củng cố sự hiểu biết của đội về triết lý SRE.

Các ví dụ về các câu hỏi dẫn dắt:

-   "Tôi thấy rằng cảnh báo TaskFailures (các Thất bại Task) fire (kích hoạt) thường xuyên, nhưng các kỹ sư on-call thường không làm gì để phản hồi cảnh báo. Điều này ảnh hưởng đến SLO như thế nào?"
-   "Quy trình turnup (khởi động) này trông khá phức tạp. Bạn có biết tại sao khi tạo một instance (bản chạy) mới của dịch vụ lại cần cập nhật nhiều file config (cấu hình) đến vậy không?"

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

Phần lớn công việc tham gia đã hoàn tất. Sau khi nhiệm vụ embed của bạn kết thúc, hãy tiếp tục sẵn sàng cho các buổi review thiết kế và code. Trong vài tháng tới, hãy theo dõi đội để xác nhận rằng họ đang dần cải thiện lập kế hoạch năng lực, phản ứng khẩn cấp và các quy trình rollout.

<a id="fn1"></a>[1](#fn1) Ngược lại với một postmortem.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
