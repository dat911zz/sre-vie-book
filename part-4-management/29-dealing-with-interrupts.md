# Chương 29. Đối phó với Gián đoạn (Dealing with Interrupts)

> **Nguyên bản:** [Chapter 29 - Dealing with Interrupts](https://sre.google/sre-book/dealing-with-interrupts/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Dave O'Connor
*Biên tập:* Diane Bates

"Operational load" (tải vận hành), khi áp dụng cho các hệ thống phức tạp, là khối lượng công việc cần thiết để duy trì hệ thống ở trạng thái hoạt động. Ví dụ, nếu bạn sở hữu một chiếc xe hơi, bạn (hoặc người bạn trả tiền) luôn phải bảo trì, đổ xăng hoặc thực hiện các công việc bảo trì định kỳ khác để giữ cho xe chạy đúng chức năng.

Bất kỳ hệ thống phức tạp nào cũng kém hoàn hảo như những người tạo ra nó. Khi quản lý tải vận hành do các hệ thống này gây ra, hãy nhớ rằng chính những người tạo ra chúng cũng là những cỗ máy kém hoàn hảo.

*Tải vận hành*, khi áp dụng cho việc quản lý các hệ thống phức tạp, mang nhiều hình thức, một số rõ ràng hơn những cái khác. Thuật ngữ có thể thay đổi, nhưng tải vận hành rơi vào ba danh mục chung: gọi trực (page), tickets (yêu cầu), và các hoạt động vận hành liên tục.

*Gọi trực* liên quan đến các cảnh báo production và hậu quả của chúng, được kích hoạt khi xảy ra sự cố khẩn cấp trên production. Đôi khi các cuộc gọi này đơn điệu, lặp đi lặp lại và đòi hỏi ít suy nghĩ. Nhưng cũng có lúc chúng hấp dẫn, buộc người nhận phải tư duy chiến thuật sâu. Mọi cuộc gọi trực đều có thời gian phản hồi kỳ vọng (SLO), đôi khi chỉ tính bằng phút.

*Tickets* liên quan đến các yêu cầu của khách hàng đòi hỏi bạn thực hiện một hành động. Giống như cuộc gọi trực, ticket có thể đơn giản và nhàm chán, hoặc đòi hỏi suy nghĩ thực sự. Một ticket đơn giản có thể yêu cầu code review cho một config mà đội sở hữu. Một ticket phức tạp hơn có thể là yêu cầu đặc biệt hoặc bất thường, nhờ giúp đỡ với một thiết kế hoặc kế hoạch năng lực (capacity). Ticket cũng có thể có SLO, nhưng thời gian phản hồi nhiều khả năng đo bằng giờ, ngày, hoặc tuần.

*Trách nhiệm vận hành liên tục* (còn gọi là "Kicking the can down the road" và "toil"; xem [Loại bỏ Toil](https://sre.google/sre-book/eliminating-toil/)) bao gồm các hoạt động như rollout code hoặc flag do đội sở hữu, hoặc phản hồi các câu hỏi ad hoc, nhạy cảm thời gian từ khách hàng. Mặc dù có thể không có SLO được định nghĩa, nhưng các task này có thể làm gián đoạn bạn.

Một số loại tải vận hành dễ dự đoán hoặc lên kế hoạch, nhưng phần lớn tải là không có kế hoạch, hoặc có thể làm gián đoạn ai đó vào một thời điểm không cụ thể, đòi hỏi người đó phải xác định xem vấn đề có thể chờ được không.

## Quản lý Tải Vận hành (Managing Operational Load)

Google có một số phương pháp để quản lý từng loại tải vận hành ở cấp độ đội.

*Gọi trực* thường do một kỹ sư on-call chính chịu trách nhiệm quản lý. Người này trực tiếp phản hồi các cuộc gọi và xử lý các incident hoặc outage phát sinh. Kỹ sư [on-call](https://sre.google/sre-book/being-on-call/) chính cũng có thể đảm nhận việc hỗ trợ người dùng, leo thang vấn đề lên các nhà phát triển sản phẩm, v.v. Để giảm thiểu gián đoạn cho đội ngũ và tránh hiệu ứng người ngoài cuộc (bystander effect), các ca on-call tại Google luôn được phân công cho một kỹ sư duy nhất. Nếu gặp vấn đề chưa rõ ràng, kỹ sư on-call có thể leo thang cuộc gọi cho thành viên khác trong đội.

Thông thường, kỹ sư on-call thứ cấp đóng vai trò dự phòng cho người chính. Nhiệm vụ của người thứ cấp có thể thay đổi. Trong một số vòng trực, việc duy nhất họ phải làm là liên hệ với người chính nếu các cuộc gọi trực bị bỏ lỡ. Khi đó, người thứ cấp có thể thuộc một đội khác. Tùy thuộc vào nhiệm vụ, kỹ sư thứ cấp có thể coi mình đang trong trạng thái gián đoạn (interrupt) hoặc không.

*Tickets* được quản lý theo nhiều cách khác nhau tùy thuộc vào đội SRE: một kỹ sư on-call chính có thể xử lý các ticket trong thời gian trực, một kỹ sư thứ cấp có thể làm tương tự, hoặc đội có thể chỉ định một người phụ trách ticket mà *không* cần trực. Các ticket có thể được tự động phân bổ ngẫu nhiên cho các thành viên, hoặc các thành viên có thể được kỳ vọng xử lý chúng theo hình thức ad hoc.

*Trách nhiệm vận hành liên tục* cũng được quản lý theo nhiều cách khác nhau. Đôi khi, kỹ sư on-call trực tiếp thực hiện các thao tác (push, flip flag, v.v.). Trong trường hợp khác, mỗi trách nhiệm có thể được giao cho các thành viên đội một cách ad hoc, hoặc một kỹ sư on-call có thể tiếp nhận một trách nhiệm kéo dài (ví dụ, một rollout hoặc ticket nhiều tuần) vượt qua tuần ca của họ.

## Các Yếu tố trong Việc Xác định Cách Gián đoạn được Xử lý (Factors in Determining How Interrupts Are Handled)

Nhìn rộng hơn ra cách vận hành được quản lý, có một số metrics (chỉ số) ảnh hưởng đến việc xử lý từng loại gián đoạn. Một số đội SRE tại Google đã nhận thấy các metrics sau đây hữu ích khi quyết định cách quản lý các gián đoạn:

-   SLO của gián đoạn hoặc thời gian phản hồi được kỳ vọng
-   Số lượng gián đoạn thường bị tồn đọng (backlogged)
-   Mức độ nghiêm trọng của các gián đoạn
-   Tần suất của các gián đoạn
-   Số người có sẵn để xử lý một loại gián đoạn nhất định (ví dụ, một số đội yêu cầu một lượng công việc ticket nhất định trước khi đi on-call)

Bạn có thể nhận thấy rằng tất cả các metrics này đều hướng đến việc đạt thời gian phản hồi thấp nhất có thể, mà không tính thêm chi phí con người. Việc cố gắng đánh giá chi phí con người và năng suất là rất khó khăn.

## Những Cỗ máy Kém hoàn hảo (Imperfect Machines)

Con người vốn không hoàn hảo. Họ có thể bị chán nản, có những bộ xử lý (và đôi khi là các UI) chưa được hiểu rõ, đồng thời hoạt động kém hiệu quả. Việc thừa nhận yếu tố con người trong nguyên tắc "Hoạt động như Ý định" và tìm cách làm việc linh hoạt hoặc cải thiện quy trình của họ là một chủ đề rộng lớn hơn nhiều so với phạm vi bài viết này; trước mắt, một số ý tưởng cơ bản có thể giúp xác định cách các gián đoạn nên hoạt động.

## Trạng thái Dòng chảy Nhận thức (Cognitive Flow State)

Khái niệm *trạng thái flow*<sup>[1](#fn1)</sup> được chấp nhận rộng rãi và hầu hết mọi người làm việc trong Kỹ thuật Phần mềm, Sysadmin, SRE, hoặc các ngành khác đòi hỏi các khoảng thời gian tập trung, đều công nhận nó bằng thực nghiệm. Đang ở trong "vùng" có thể tăng năng suất, nhưng cũng có thể tăng sự sáng tạo nghệ thuật và khoa học. Đạt được trạng thái này khuyến khích mọi người thực sự làm chủ và cải thiện task hoặc dự án họ đang làm. Việc bị gián đoạn có thể đá bạn ra khỏi trạng thái này ngay, nếu sự gián đoạn đó đủ lớn. Bạn muốn tối đa hóa thời gian dành trong trạng thái này.

Dòng chảy nhận thức cũng có thể xuất hiện ở những lĩnh vực ít sáng tạo hơn, nơi yêu cầu về kỹ năng thấp hơn, miễn là các yếu tố cốt lõi của flow vẫn được đáp ứng (mục tiêu rõ ràng, phản hồi tức thời, cảm giác kiểm soát, và sự méo thời gian liên quan); các ví dụ bao gồm việc nhà hoặc lái xe.

Bạn có thể vào vùng (zone) bằng cách làm những việc đòi hỏi kỹ năng thấp, độ khó thấp, chẳng hạn như chơi đi chơi lại một trò chơi video. Bạn cũng có thể dễ dàng đến đó bằng cách thực hiện những việc đòi hỏi kỹ năng cao, độ khó cao, chẳng hạn như những gì một kỹ sư có thể đối mặt. Các phương pháp để đạt đến trạng thái dòng chảy nhận thức khác nhau, nhưng kết quả về cơ bản giống nhau.

### Trạng thái dòng chảy nhận thức: Sáng tạo và tập trung (Creative and engaged)

Đây là vùng (zone): khi một người tập trung giải quyết vấn đề trong một khoảng thời gian, nắm rõ và thoải mái với các tham số của nó, đồng thời tin rằng mình có thể sửa hoặc giải quyết được. Trong trạng thái này, họ làm việc chăm chỉ, quên mất thời gian và phớt lờ các yếu tố gây gián đoạn chừng nào có thể. Việc tối đa hóa thời gian một người có thể duy trì trạng thái này là rất đáng mong đợi — họ sẽ tạo ra những kết quả sáng tạo và đạt hiệu suất cao về số lượng. Họ cũng sẽ hài lòng hơn với công việc mình đang làm.

Đáng tiếc, nhiều người ở các vai trò kiểu SRE lại dành phần lớn thời gian để cố gắng (và thất bại) đạt đến trạng thái này, rồi trở nên bực bội khi không thể; hoặc thậm chí không bao giờ cố, mà chỉ mòn mỏi trong trạng thái bị gián đoạn.

### Trạng thái dòng chảy nhận thức: Angry Birds

Con người thường thích làm những việc mình đã quen thuộc. Thực tế, xử lý các task như vậy là một trong những con đường rõ ràng nhất để đạt đến trạng thái dòng chảy nhận thức. Một số SRE thậm chí tìm thấy trạng thái này khi trực on-call. Việc truy tìm nguyên nhân sự cố, phối hợp với đồng nghiệp và cải thiện sức khỏe tổng thể của hệ thống một cách hữu hình có thể mang lại cảm giác rất thỏa mãn. Ngược lại, với phần lớn kỹ sư on-call, nguồn gốc của sự căng thẳng nằm ở số lượng ca trực hoặc cách họ coi on-call là một sự gián đoạn. Họ vừa cố code hoặc làm việc trên các dự án, vừa phải trực on-call hoặc đối mặt với các gián đoạn toàn thời gian. Những kỹ sư này sống trong trạng thái gián đoạn liên tục, hay còn gọi là *khả năng bị gián đoạn* (interruptability). Môi trường làm việc này gây ra áp lực cực kỳ lớn.

Ngược lại, khi một người dành toàn thời gian cho các gián đoạn, *chúng không còn mang tính gián đoạn nữa*. Theo bản năng, việc thực hiện các cải tiến tăng dần cho hệ thống, xử lý ticket và khắc phục sự cố hay outage tạo thành một bộ mục tiêu, ranh giới và phản hồi rõ ràng: bạn đóng X bug, hoặc bạn ngừng bị gọi trực. Phần còn lại chỉ là những sự xao nhãng. *Khi bạn đang xử lý gián đoạn, các dự án của bạn lại trở thành sự xao nhãng*. Dù việc xử lý gián đoạn có thể mang lại sự thỏa mãn trong ngắn hạn, trong môi trường kết hợp dự án/on-call, mọi người cuối cùng sẽ hài lòng hơn với sự cân bằng giữa hai loại công việc này. Mức cân bằng lý tưởng khác nhau ở mỗi kỹ sư. Điều quan trọng là nhận ra rằng một số kỹ sư có thể không thực sự biết sự cân bằng nào truyền động lực cho họ tốt nhất (hoặc có thể nghĩ rằng họ biết, nhưng bạn có thể không đồng ý).

## Làm một Thứ tốt (Do One Thing Well)

Bạn có thể đang tự hỏi về ý nghĩa thực tiễn của những gì đã đọc cho đến nay.

Dưới đây là những gợi ý rút ra từ kinh nghiệm thực tế của các đội SRE khác nhau mà tôi từng quản lý tại Google, chủ yếu dành cho quản lý đội hoặc người có ảnh hưởng. Tài liệu này không đi sâu vào thói quen cá nhân — mỗi người tự do sắp xếp thời gian theo cách mình thấy phù hợp. Trọng tâm ở đây là định hướng về cấu trúc để đội chủ động quản lý các gián đoạn, giúp mọi người không bị đặt vào thế bất lợi do chức năng hay cấu trúc của đội.

### Dễ bị xao nhãng (Distractibility)

Các cách mà một kỹ sư có thể bị xao nhãng và do đó bị ngăn cản đạt đến trạng thái dòng chảy nhận thức là nhiều. Ví dụ: hãy xem xét một SRE ngẫu nhiên tên là Fred. Fred đến làm việc vào sáng thứ Hai. Fred không on-call hoặc không ở trong các gián đoạn hôm nay, nên rõ ràng Fred muốn làm việc trên các dự án của anh ấy. Anh ấy lấy một ly cà phê, đeo tai nghe "không làm phiền" của anh ấy, và ngồi vào bàn của anh ấy. Giờ vùng (zone), phải không?

Ngoại trừ, vào bất kỳ lúc nào, bất kỳ điều gì sau đây có thể xảy ra:

-   Đội của Fred sử dụng một hệ thống ticket tự động để phân công các ticket ngẫu nhiên cho đội. Một ticket được gán cho anh ấy, đến hạn hôm nay.
-   Đồng nghiệp của Fred on-call và nhận được một lần gọi trực về một thành phần mà Fred là chuyên gia, và làm gián đoạn anh ấy để hỏi về nó.
-   Một người dùng của dịch vụ Fred nâng mức ưu tiên của một ticket đã được gán cho anh ấy kể từ tuần trước, khi anh ấy on-call.
-   Một cờ rollout đang được triển khai trong 3–4 tuần và được gán cho Fred đã bị cấu hình sai, buộc Fred phải bỏ hết mọi việc để xem xét quá trình triển khai, hoàn tác thay đổi, v.v.
-   Một người dùng của dịch vụ Fred liên hệ Fred để hỏi một câu hỏi, vì Fred là một anh chàng rất sẵn lòng giúp đỡ.
-   Và vân vân.

Kết quả là, dù Fred có cả ngày rảnh rỗi để tập trung vào các dự án, anh vẫn rất dễ bị phân tâm. Một số nguồn gây xao nhãng, anh có thể tự kiểm soát bằng cách tắt email, tắt IM hoặc áp dụng các biện pháp tương tự. Số còn lại thì bắt nguồn từ chính sách, hoặc từ những mặc định về việc bị gián đoạn và các trách nhiệm liên tục.

Bạn có thể cho rằng một mức độ xao nhãng nhất định là không thể tránh khỏi và được thiết kế như vậy. Giả định này đúng: mọi người thực sự giữ các bug mà họ là liên hệ chính, và cũng tích lũy các trách nhiệm và nghĩa vụ khác. Tuy nhiên, có những cách một đội có thể quản lý phản ứng gián đoạn để nhiều người (trung bình) có thể đến làm việc vào buổi sáng và *cảm thấy không thể bị xao nhãng*.

### Phân cực thời gian (Polarizing time)

Để giảm bớt sự xao nhãng, bạn nên hạn chế tối đa các context switch. Dù một số gián đoạn là không thể tránh khỏi, nhưng nếu coi kỹ sư như một đơn vị công việc có thể bị ngắt quãng mà các context switch của họ lại miễn phí, thì cách tiếp cận này kém tối ưu nếu bạn muốn mọi người vừa hạnh phúc vừa có năng suất. Hãy gán một chi phí cho các context switch. Một lần gián đoạn 20 phút khi đang làm việc trên một dự án bao gồm hai context switch; thực tế, sự gián đoạn này dẫn đến mất một vài giờ làm việc thực sự năng suất. Để tránh tình trạng mất năng suất xảy ra liên tục, hãy hướng tới thời gian phân cực giữa các phong cách làm việc, với mỗi khoảng thời gian làm việc kéo dài lâu nhất có thể. Lý tưởng nhất là một tuần, nhưng một ngày hoặc thậm chí nửa ngày có thể thực tế hơn. Chiến lược này cũng phù hợp với khái niệm bổ sung *make time* [[Gra09]](https://sre.google/sre-book/bibliography#Gra09).

Phân cực thời gian có nghĩa là mỗi ngày đi làm, một người cần biết rõ mình đang xử lý *chỉ* công việc dự án hay *chỉ* các gián đoạn. Cách phân cực này giúp họ tập trung vào task trước mắt trong những khoảng thời gian dài hơn, đồng thời tránh căng thẳng do bị kéo vào các task khiến họ rời khỏi công việc lẽ ra phải đang làm.

## Nghiêm túc, Hãy nói cho tôi biết phải làm gì (Seriously, Tell Me What to Do)

Nếu mô hình chung được trình bày trong chương này không phù hợp với bạn, dưới đây là một số gợi ý cụ thể về các thành phần bạn có thể triển khai từng phần.

### Các gợi ý chung

Với bất kỳ lớp gián đoạn nào, nếu số lượng quá tải cho một người, hãy *thêm một người nữa*. Khái niệm này rõ ràng nhất khi áp dụng cho ticket, nhưng cũng có thể dùng cho các cuộc gọi trực — on-caller có thể chuyển các vấn đề lên người thứ cấp, hoặc hạ các cuộc gọi trực xuống thành ticket.

### On-call

Kỹ sư on-call chính cần tập trung toàn bộ vào nhiệm vụ trực. Nếu pager của bạn im lặng, các ticket hay công việc gián đoạn khác có thể xử lý nhanh nên vẫn nằm trong phạm vi on-call. Trong tuần trực, thời gian đó không được tính vào tiến độ dự án. Nếu dự án quan trọng đến mức không thể bỏ trống một tuần, người đó không nên nhận ca on-call; hãy leo thang để phân công người khác. *Không ai nên bị kỳ vọng vừa on-call vừa đạt tiến độ dự án (hoặc bất kỳ công việc nào khác đòi hỏi chi phí context switch cao)*.

Nhiệm vụ thứ cấp phụ thuộc vào việc những nhiệm vụ đó nặng nề như thế nào. Nếu chức năng của người thứ cấp là dự phòng cho người chính trong trường hợp bị bỏ lỡ, thì có lẽ bạn có thể an toàn giả định rằng người thứ cấp cũng có thể hoàn thành công việc dự án. Nếu ai đó khác với người thứ cấp được gán để xử lý các ticket, hãy xem xét gộp các vai trò. Nếu người thứ cấp được kỳ vọng thực sự giúp đỡ người chính trong trường hợp số lượng gọi trực cao, họ cũng nên làm công việc gián đoạn.

(Ngoài lề: *Bạn không bao giờ hết công việc dọn dẹp*. Số ticket của bạn có thể ở mức không, nhưng luôn luôn có tài liệu cần cập nhật, các config cần dọn dẹp, v.v. Các kỹ sư on-call tương lai của bạn sẽ cảm ơn bạn, và điều đó có nghĩa là họ ít có khả năng làm gián đoạn bạn trong khoảng thời gian make time quý giá của bạn).

### Tickets (Các yêu cầu)

Nếu hiện tại bạn đang phân công các ticket ngẫu nhiên cho các nạn nhân trong đội, *hãy dừng lại*. Việc làm như vậy cực kỳ bất kính với thời gian của đội bạn, và hoàn toàn ngược với nguyên lý của việc không thể bị gián đoạn nhiều nhất có thể.

Mỗi ticket nên được giao cho một người phụ trách toàn thời gian trong khoảng thời gian mà họ có thể xử lý được. Nếu bạn rơi vào tình thế khó xử khi số ticket vượt quá khả năng đóng của cả kỹ sư on-call chính lẫn thứ cấp, hãy thiết lập vòng trực ticket sao cho luôn có hai người xử lý tại bất kỳ thời điểm nào. Đừng dàn trải khối lượng công việc khắp đội. Con người không phải là máy móc, và việc này chỉ gây ra các context switch, làm ảnh hưởng đến thời gian flow quý giá.

### Các trách nhiệm liên tục (Ongoing responsibilities)

Càng nhiều càng tốt, hãy định nghĩa các vai trò cho phép bất kỳ ai trong đội tiếp nhận trách nhiệm. Nếu có một quy trình được định nghĩa rõ ràng để thực hiện và xác thực các push hoặc flip flag, thì không có lý do gì mà một người phải đưa thay đổi đó trong suốt vòng đời của nó, ngay cả sau khi họ ngừng on-call hoặc trong các gián đoạn. Định nghĩa một vai trò *push manager* (người quản lý push) có thể xoay xở điều phối các push trong suốt thời gian on-call hoặc trong các gián đoạn của họ. Chính thức hóa quy trình bàn giao — đó là một cái giá nhỏ để trả cho khoảng thời gian make time không bị gián đoạn cho những người không on-call.

### Hãy ở trong các gián đoạn, hoặc đừng (Be on interrupts, or don't be)

Đôi khi, khi một người không nằm trong danh sách trực, đội lại nhận được sự cố mà chỉ có người đó mới có khả năng xử lý. Mặc dù lý tưởng là kịch bản này không bao giờ xảy ra, nhưng thực tế vẫn có những lúc như vậy. Bạn nên nỗ lực để những sự kiện tương tự trở nên hiếm gặp.

Đôi khi, mọi người lại xử lý các ticket không thuộc phần việc của mình chỉ để trông có vẻ bận rộn. Cách làm này không mang lại lợi ích gì, mà còn khiến hiệu suất của người đó thấp hơn mức lý tưởng. Nó cũng làm sai lệch các số liệu về khả năng quản lý tải ticket. Ví dụ, nếu một người được gán ticket nhưng hai hoặc ba người khác cũng cùng can thiệp vào hàng đợi, bạn có thể vẫn đang đối mặt với một hàng đợi quá tải mà không hề hay biết.

## Giảm các Gián đoạn (Reducing Interrupts)

Tải gián đoạn của đội bạn có thể vượt quá khả năng kiểm soát nếu đòi hỏi quá nhiều thành viên phải trực đồng thời tại bất kỳ thời điểm nào. Bạn có thể áp dụng một số kỹ thuật để giảm tổng lượng ticket.

### Thực sự phân tích các ticket (Actually analyze tickets)

Nhiều vòng trực ticket hoặc on-call giống như một cuộc vượt chướng ngại vật (gauntlet). Điều này đặc biệt đúng với các đội lớn. Nếu bạn chỉ phải đối mặt với các gián đoạn vài tháng một lần, việc chạy qua cuộc vượt chướng ngại vật,<sup>[2](#fn2)</sup> thở phào nhẹ nhõm rồi quay lại công việc thường nhật sẽ rất dễ dàng. Người tiếp quản sau đó cũng làm tương tự, và nguyên nhân gốc rễ của các ticket không bao giờ được điều tra. Thay vì đạt được tiến bộ, đội bạn bị lún vào vòng lặp của một loạt người bực mình vì cùng các vấn đề.

Cần có quy trình bàn giao cho cả ticket lẫn công việc on-call. Quy trình này giúp duy trì trạng thái chia sẻ giữa những người xử lý ticket khi trách nhiệm được chuyển giao. Ngay cả việc nắm bắt đôi chút về nguyên nhân gốc rễ của các sự cố gián đoạn cũng có thể mang lại giải pháp tốt để giảm tốc độ tổng thể. *Nhiều* đội thực hiện bàn giao on-call và xem xét các cuộc gọi trực. *Rất ít* đội làm tương tự cho ticket.

Đội bạn nên định kỳ rà soát (scrub) các ticket và cuộc gọi trực, xem xét các lớp gián đoạn để xác định nguyên nhân gốc rễ. Nếu bạn cho rằng nguyên nhân gốc rễ có thể được khắc phục trong khoảng thời gian hợp lý, hãy *tắt các gián đoạn đó cho đến khi nguyên nhân gốc rễ được kỳ vọng sẽ được sửa*. Việc này giúp giảm tải cho người xử lý gián đoạn và tạo ra một hạn chót tiện lợi cho người sửa nguyên nhân gốc rễ.

### Tôn trọng bản thân, cũng như các khách hàng của bạn (Respect yourself, as well as your customers)

Câu nói này đúng hơn với các sự gián đoạn do người dùng gây ra so với các sự gián đoạn tự động, dù các nguyên tắc vẫn áp dụng được cho cả hai kịch bản. Nếu các ticket đặc biệt khó chịu hoặc nặng nề để xử lý, bạn có thể dùng chính sách một cách hiệu quả để giảm bớt gánh nặng.

Hãy nhớ rằng:

-   Đội bạn thiết lập mức độ dịch vụ được cung cấp bởi dịch vụ của bạn.
-   OK (Không sao) để đẩy một số nỗ lực lên các khách hàng của bạn.

Nếu đội bạn chịu trách nhiệm xử lý các ticket hoặc gián đoạn cho khách hàng, bạn thường có thể sử dụng chính sách để làm cho tải công việc của bạn có thể quản lý hơn. Một sửa chữa chính sách có thể là tạm thời hoặc vĩnh viễn, tùy thuộc vào điều gì có ý nghĩa. Một sửa chữa như vậy nên đạt được một sự cân bằng tốt giữa sự tôn trọng khách hàng và sự tôn trọng bản thân. Chính sách có thể là một công cụ mạnh mẽ như code.

Ví dụ, nếu bạn đang hỗ trợ một công cụ đặc biệt không ổn định, thiếu nguồn lực phát triển, và một số ít khách hàng lại đòi hỏi nhiều công sức để hỗ trợ việc sử dụng nó, hãy xem xét các phương án khác. Hãy cân nhắc giá trị của thời gian bạn bỏ ra để xử lý các sự cố gây gián đoạn cho hệ thống này, và liệu cách bạn đang phân bổ thời gian đó có thực sự hợp lý không. Đến một lúc nào đó, nếu bạn không thể giành được sự chú ý cần thiết để giải quyết nguyên nhân gốc rễ của các vấn đề gây gián đoạn, có lẽ thành phần bạn đang hỗ trợ không quan trọng đến vậy. Bạn nên cân nhắc việc trả lại máy gọi trực, khai tử, thay thế nó, hoặc áp dụng một chiến lược khác theo hướng này.

Nếu một sự gián đoạn tốn thời gian hoặc phức tạp, nhưng không đòi hỏi đặc quyền của bạn để xử lý, hãy cân nhắc dùng chính sách để đẩy yêu cầu trở lại cho người đề xuất. Ví dụ, nếu mọi người cần quyên góp tài nguyên tính toán, hãy chuẩn bị sẵn một thay đổi code, config hoặc các bước tương tự, sau đó hướng dẫn khách hàng thực hiện và gửi lại để bạn xem xét. Hãy nhớ rằng nếu khách hàng muốn hoàn thành một task nhất định, họ cần sẵn sàng bỏ ra một chút công sức để đạt được điều đó.

Một lưu ý đối với các giải pháp trước đó là bạn cần tìm sự cân bằng giữa việc tôn trọng khách hàng và bản thân. Nguyên lý dẫn dắt khi xây dựng chiến lược ứng phó với yêu cầu của khách hàng là: yêu cầu phải có ý nghĩa, hợp lý, và cung cấp đủ thông tin cùng công sức cần thiết để bạn có thể đáp ứng. Đổi lại, phản hồi của bạn nên hữu ích và kịp thời.

<a id="fn1"></a>[1](#fn1) Xem Wikipedia: Flow (psychology) (Dòng chảy (tâm lý học)), [*https://en.wikipedia.org/wiki/Flow_(psychology)*](https://en.wikipedia.org/wiki/Flow_(psychology)).

<a id="fn2"></a>[2](#fn2) Xem [*https://en.wikipedia.org/wiki/Running_the_gauntlet*](https://en.wikipedia.org/wiki/Running_the_gauntlet).

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
