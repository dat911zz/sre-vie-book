# Chương 29. Đối phó với Interrupt (Dealing with Interrupts)

> **Nguyên bản:** [Chapter 29 - Dealing with Interrupts](https://sre.google/sre-book/dealing-with-interrupts/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Đội biên tập Softdreams RnD)

---

*Tác giả:* Dave O'Connor
*Biên tập:* Diane Bates

"Operational load" (tải vận hành), khi áp dụng cho các hệ thống phức tạp, là công việc cần thực hiện để duy trì hệ thống ở trạng thái hoạt động. Ví dụ, nếu bạn sở hữu một chiếc xe hơi, bạn (hoặc ai đó bạn trả tiền) luôn phải bảo trì nó, đổ xăng, hoặc thực hiện các bảo trì định kỳ khác để giữ cho nó chạy đúng chức năng.

Bất kỳ hệ thống phức tạp nào cũng kém hoàn hảo như những người sáng tạo của nó. Trong việc quản lý tải vận hành do những hệ thống này tạo ra, hãy nhớ rằng những người sáng tạo của nó cũng là những cỗ máy kém hoàn hảo.

*Operational load*, khi áp dụng cho việc quản lý các hệ thống phức tạp, mang nhiều hình thức, một số rõ ràng hơn những cái khác. Thuật ngữ có thể thay đổi, nhưng tải vận hành rơi vào ba danh mục chung: pages (gọi trực), tickets (yêu cầu), và các hoạt động vận hành liên tục.

*Pages* liên quan đến các cảnh báo production và hậu quả của chúng, được kích hoạt để đáp lại các trường hợp khẩn cấp production. Đôi khi chúng đơn điệu và lặp đi lặp lại, đòi hỏi ít suy nghĩ. Chúng cũng có thể hấp dẫn và đòi hỏi suy nghĩ chiến thuật sâu. Các page luôn có một thời gian phản hồi được kỳ vọng (SLO), đôi khi đo bằng phút.

*Tickets* liên quan đến các yêu cầu của khách hàng đòi hỏi bạn thực hiện một hành động. Giống page, ticket có thể đơn giản và nhàm chán, hoặc đòi hỏi suy nghĩ thực sự. Một ticket đơn giản có thể yêu cầu code review cho một config mà đội sở hữu. Một ticket phức tạp hơn có thể là yêu cầu đặc biệt hoặc bất thường, nhờ giúp đỡ với một thiết kế hoặc kế hoạch năng lực (capacity). Ticket cũng có thể có SLO, nhưng thời gian phản hồi nhiều khả năng đo bằng giờ, ngày, hoặc tuần.

*Trách nhiệm vận hành liên tục* (còn gọi là "Kicking the can down the road" và "toil"; xem [Loại bỏ Toil](https://sre.google/sre-book/eliminating-toil/)) bao gồm các hoạt động như rollout code hoặc flag do đội sở hữu, hoặc phản hồi các câu hỏi ad hoc, nhạy cảm thời gian từ khách hàng. Mặc dù có thể không có SLO được định nghĩa, nhưng các task này có thể làm gián đoạn bạn.

Một số loại tải vận hành dễ dự đoán hoặc lên kế hoạch, nhưng phần lớn tải là không có kế hoạch, hoặc có thể làm gián đoạn ai đó vào một thời điểm không cụ thể, đòi hỏi người đó phải xác định xem vấn đề có thể chờ được không.

## Quản lý Tải Vận hành (Managing Operational Load)

Google có một số phương pháp để quản lý từng loại tải vận hành ở cấp độ đội.

*Pages* thường được quản lý nhất bởi một kỹ sư on-call chính được chỉ định. Đây là một người đơn lẻ phản hồi các page và quản lý các incident hoặc outage do chúng gây ra. Kỹ sư [on-call](https://sre.google/sre-book/being-on-call/) chính cũng có thể quản lý các truyền thông hỗ trợ người dùng, leo thang đến các nhà phát triển sản phẩm, và vân vân. Để vừa giảm thiểu sự gián đoạn mà một page gây ra cho một đội, vừa tránh hiệu ứng người ngoài cuộc (bystander effect), các ca on-call của Google được trực bởi một kỹ sư đơn lẻ. Kỹ sư on-call có thể leo thang các page đến một thành viên đội khác nếu một vấn đề không được hiểu rõ.

Thường thì, một kỹ sư on-call thứ cấp đóng vai trò là dự phòng cho người chính. Nhiệm vụ của kỹ sư thứ cấp thay đổi. Trong một số vòng trực, nhiệm vụ duy nhất của người thứ cấp là liên hệ với người chính nếu các page rơi qua. Trong trường hợp này, người thứ cấp có thể ở một đội khác. Kỹ sư thứ cấp có thể hoặc không coi mình là *on interrupts* (đang trong các gián đoạn), tùy thuộc vào nhiệm vụ.

*Tickets* được quản lý theo một số cách khác nhau, tùy đội SRE: một kỹ sư on-call chính có thể làm việc với các ticket trong khi on-call, một kỹ sư thứ cấp có thể làm việc với các ticket trong khi on-call, hoặc một đội có thể có một người ticket được chỉ định mà *không* on-call. Các ticket có thể được tự động phân phối ngẫu nhiên giữa các thành viên đội, hoặc các thành viên đội có thể được kỳ vọng xử lý các ticket ad hoc.

*Trách nhiệm vận hành liên tục* cũng được quản lý theo nhiều cách khác nhau. Đôi khi, kỹ sư on-call thực hiện công việc (các push (đẩy), flip (đảo) flag, v.v.). Thay vào đó, mỗi trách nhiệm có thể được gán cho các thành viên đội một cách ad hoc, hoặc một kỹ sư on-call có thể tiếp nhận một trách nhiệm kéo dài (tức là, một rollout hoặc ticket nhiều tuần) kéo dài vượt qua tuần ca của họ.

## Các Yếu tố trong Việc Xác định Cách Interrupt được Xử lý (Factors in Determining How Interrupts Are Handled)

Để lùi một bước khỏi các cơ chế của việc tải vận hành được quản lý như thế nào, có một số metrics (chỉ số) đóng vai trò trong cách mỗi loại interrupt này được xử lý. Một số đội SRE tại Google đã tìm thấy các metrics sau đây hữu ích trong việc quyết định cách quản lý các interrupt:

-   SLO của interrupt hoặc thời gian phản hồi được kỳ vọng
-   Số lượng interrupt thường bị tồn đọng (backlogged)
-   Mức độ nghiêm trọng của các interrupt
-   Tần suất của các interrupt
-   Số người có sẵn để xử lý một loại interrupt nhất định (ví dụ, một số đội yêu cầu một lượng công việc ticket nhất định trước khi đi on-call)

Bạn có thể nhận thấy rằng tất cả các metrics này đều phù hợp để đáp ứng thời gian phản hồi thấp nhất có thể, mà không tính đến các chi phí con người hơn. Cố gắng đánh giá chi phí con người và năng suất là khó khăn.

## Những Cỗ máy Kém hoàn hảo (Imperfect Machines)

Con người là những cỗ máy kém hoàn hảo. Họ có thể chán nản, có các bộ xử lý (và đôi khi các UI) không được hiểu rõ lắm, và không hiệu quả lắm. Việc công nhận yếu tố con người là "Hoạt động như Ý định" và cố gắng làm việc quanh hoặc cải thiện cách con người làm việc có thể lấp đầy một không gian lớn hơn nhiều so với phần dành ở đây; trước mắt, một số ý tưởng cơ bản có thể hữu ích trong việc xác định các interrupt nên hoạt động như thế nào.

## Trạng thái Dòng chảy Nhận thức (Cognitive Flow State)

Khái niệm *trạng thái flow*<sup>[1](#fn1)</sup> được chấp nhận rộng rãi và hầu hết mọi người làm việc trong Kỹ thuật Phần mềm, Sysadmin, SRE, hoặc các ngành khác đòi hỏi các khoảng thời gian tập trung, đều công nhận nó bằng thực nghiệm. Đang ở trong "vùng" có thể tăng năng suất, nhưng cũng có thể tăng sự sáng tạo nghệ thuật và khoa học. Đạt được trạng thái này khuyến khích mọi người thực sự làm chủ và cải thiện task hoặc dự án họ đang làm. Việc bị gián đoạn có thể đá bạn ra khỏi trạng thái này ngay, nếu interrupt gây đủ gián đoạn. Bạn muốn tối đa hóa thời gian dành trong trạng thái này.

Dòng chảy nhận thức cũng có thể áp dụng cho các lĩnh vực ít sáng tạo hơn mà ở đó mức kỹ năng yêu cầu thấp hơn, và các yếu tố cốt lõi của flow vẫn được đáp ứng (mục tiêu rõ ràng, phản hồi tức thời, cảm giác kiểm soát, và sự méo thời gian liên quan); các ví dụ bao gồm việc nhà hoặc lái xe.

Bạn có thể vào vùng (zone) bằng cách làm việc trên các vấn đề kỹ năng thấp, độ khó thấp, chẳng hạn như chơi một trò chơi video lặp đi lặp lại. Bạn cũng có thể dễ dàng đến đó bằng cách thực hiện các vấn đề kỹ năng cao, độ khó cao, chẳng hạn như những gì một kỹ sư có thể đối mặt. Các phương pháp để đạt đến trạng thái dòng chảy nhận thức khác nhau, nhưng kết quả về cơ bản giống nhau.

### Trạng thái dòng chảy nhận thức: Sáng tạo và tập trung (Creative and engaged)

Đây là vùng (zone): ai đó làm việc trên một vấn đề trong một thời gian, nhận thức được và thoải mái với các tham số của vấn đề, và cảm thấy rằng họ có thể sửa hoặc giải quyết nó. Người đó làm việc chăm chỉ trên vấn đề, mất track (theo dõi) thời gian và phớt lờ các interrupt chừng nào có thể. Tối đa hóa lượng thời gian một người có thể dành trong trạng thái này là rất mong muốn — họ sẽ tạo ra các kết quả sáng tạo và làm tốt công việc về số lượng. Họ sẽ hạnh phúc hơn với công việc họ đang làm.

Đáng tiếc, nhiều người trong các vai trò kiểu SRE dành phần lớn thời gian của họ hoặc cố gắng và thất bại để đạt đến trạng thái này và trở nên bực bội khi họ không thể, hoặc thậm chí không bao giờ cố gắng đạt đến trạng thái này, thay vào đó lười biếng trong trạng thái bị gián đoạn.

### Trạng thái dòng chảy nhận thức: Angry Birds (Gà tức giận)

Mọi người thích thực hiện những công việc mà họ biết cách làm. Thực ra, thực hiện những task như vậy là một trong những con đường rõ ràng nhất đến dòng chảy nhận thức. Một số SRE đi vào on-call khi họ đạt đến trạng thái dòng chảy nhận thức. Truy tìm nguyên nhân của các vấn đề, làm việc với người khác, và cải thiện sức khỏe tổng thể của hệ thống theo cách hữu hình có thể rất thỏa mãn. Ngược lại, với phần lớn các kỹ sư on-call căng thẳng, sự căng thẳng đến từ hoặc số lượng pager, hoặc từ việc họ xử lý on-call như một interrupt. Họ đang cố code hoặc làm việc trên các dự án trong khi đồng thời on-call hoặc trong các gián đoạn toàn thời gian. Những kỹ sư này tồn tại trong một trạng thái gián đoạn liên tục, hoặc *khả năng bị gián đoạn* (interruptability). Môi trường làm việc này cực kỳ căng thẳng.

Mặt khác, khi một người tập trung toàn thời gian vào các interrupt, *các interrupt ngừng là interrupt*. Ở mức rất trực giác, việc thực hiện các cải tiến tăng dần cho hệ thống, đập các ticket, và sửa các vấn đề và outage trở thành một bộ mục tiêu, ranh giới, và phản hồi rõ ràng: bạn đóng X bug, hoặc bạn ngừng bị page. Điều còn lại chỉ là các sự xao nhãng. *Khi bạn đang làm interrupt, các dự án của bạn là một sự xao nhãng*. Mặc dù interrupt có thể là cách dùng thời gian thỏa mãn trong ngắn hạn, trong một môi trường dự án/on-call hỗn hợp, mọi người cuối cùng sẽ hạnh phúc hơn với một sự cân bằng giữa hai loại công việc này. Sự cân bằng lý tưởng thay đổi từ kỹ sư này sang kỹ sư khác. Quan trọng là nhận thức rằng một số kỹ sư có thể không thực sự biết sự cân bằng nào truyền động lực cho họ tốt nhất (hoặc có thể nghĩ rằng họ biết, nhưng bạn có thể không đồng ý).

## Làm một Thứ tốt (Do One Thing Well)

Bạn có thể đang tự hỏi về ý nghĩa thực tiễn của những gì đã đọc cho đến nay.

Các gợi ý sau đây, dựa trên những gì đã hoạt động cho các đội SRE khác nhau mà tôi quản lý tại Google, chủ yếu vì lợi ích của các quản lý đội hoặc người ảnh hưởng. Tài liệu này không quan tâm đến các thói quen cá nhân — mọi người tự do quản lý thời gian của họ theo cách họ thấy phù hợp. Sự tập trung ở đây là định hướng cấu trúc mà chính đội quản lý các interrupt, để mọi người không bị đặt vào thế thất bại vì chức năng hoặc cấu trúc của đội.

### Dễ bị xao nhãng (Distractibility)

Các cách mà một kỹ sư có thể bị xao nhãng và do đó bị ngăn cản đạt đến trạng thái dòng chảy nhận thức là nhiều. Ví dụ: hãy xem xét một SRE ngẫu nhiên tên là Fred. Fred đến làm việc vào sáng thứ Hai. Fred không on-call hoặc trong các gián đoạn hôm nay, nên rõ ràng Fred muốn làm việc trên các dự án của anh ấy. Anh ấy lấy một ly cà phê, đeo tai nghe "không làm phiền" của anh ấy, và ngồi vào bàn của anh ấy. Giờ vùng (zone), phải không?

Ngoại trừ, vào bất kỳ lúc nào, bất kỳ điều gì sau đây có thể xảy ra:

-   Đội của Fred sử dụng một hệ thống ticket tự động để phân công các ticket ngẫu nhiên cho đội. Một ticket được gán cho anh ấy, đến hạn hôm nay.
-   Đồng nghiệp của Fred on-call và nhận được một page về một thành phần mà Fred là chuyên gia, và làm gián đoạn anh ấy để hỏi về nó.
-   Một người dùng của dịch vụ Fred nâng mức ưu tiên của một ticket đã được gán cho anh ấy kể từ tuần trước, khi anh ấy on-call.
-   Một rollout flag đang rollout trong 3–4 tuần và được gán cho Fred bị sai, buộc Fred phải bỏ hết mọi thứ để xem xét rollout, hoàn tác thay đổi, và vân vân.
-   Một người dùng của dịch vụ Fred liên hệ Fred để hỏi một câu hỏi, vì Fred là một anh chàng rất sẵn lòng giúp đỡ.
-   Và vân vân.

Kết quả cuối cùng là mặc dù Fred có cả ngày lịch trống để làm việc trên các dự án, anh ấy vẫn rất dễ bị xao nhãng. Một số sự xao nhãng này anh ấy có thể tự quản lý bằng cách đóng email, tắt IM, hoặc thực hiện các biện pháp tương tự. Một số sự xao nhãng do chính sách, hoặc do các giả định xung quanh các interrupt và các trách nhiệm liên tục.

Bạn có thể tuyên bố rằng một mức độ xao nhãng nào đó là không thể tránh khỏi và được thiết kế như vậy. Giả định này đúng: mọi người thực sự giữ các bug mà họ là liên hệ chính, và cũng tích lũy các trách nhiệm và nghĩa vụ khác. Tuy nhiên, có những cách một đội có thể quản lý phản ứng interrupt để nhiều người (trung bình) có thể đến làm việc vào buổi sáng và *cảm thấy không thể bị xao nhãng*.

### Phân cực thời gian (Polarizing time)

Để hạn chế mức độ dễ bị xao nhãng, bạn nên cố gắng giảm thiểu các context switch. Một số interrupt là không thể tránh khỏi. Tuy nhiên, xem xét một kỹ sư như một đơn vị công việc có thể bị gián đoạn, mà các context switch của họ là miễn phí, là kém tối ưu nếu bạn muốn mọi người hạnh phúc và có năng suất. Hãy gán một chi phí cho các context switch. Một sự gián đoạn 20 phút khi đang làm việc trên một dự án bao gồm hai context switch; thực tế, sự gián đoạn này dẫn đến mất một vài giờ làm việc thực sự năng suất. Để tránh sự mất năng suất xảy ra liên tục, hãy nhắm đến thời gian phân cực giữa các phong cách làm việc, với mỗi khoảng thời gian làm việc kéo dài lâu nhất có thể. Lý tưởng nhất là một tuần, nhưng một ngày hoặc thậm chí nửa ngày có thể thực tế hơn. Chiến lược này cũng phù hợp với khái niệm bổ sung *make time* [[Gra09]](https://sre.google/sre-book/bibliography#Gra09).

Phân cực thời gian có nghĩa là khi một người đến làm việc mỗi ngày, họ nên biết liệu họ đang làm *chỉ* công việc dự án hay *chỉ* interrupt. Phân cực thời gian theo cách này giúp họ tập trung trong những khoảng thời gian dài hơn cho task trước mắt. Họ không bị căng thẳng vì bị kéo vào các task khiến họ rời khỏi công việc mà lẽ ra phải đang làm.

## Nghiêm túc, Hãy nói cho tôi biết phải làm gì (Seriously, Tell Me What to Do)

Nếu mô hình chung được trình bày trong chương này không hoạt động cho bạn, đây là một số gợi ý cụ thể về các thành phần bạn có thể triển khai từng phần.

### Các gợi ý chung

Đối với bất kỳ lớp interrupt nào, nếu số lượng interrupt quá cao cho một người, *thêm một người nữa*. Khái niệm này rõ ràng nhất áp dụng cho các ticket, nhưng cũng có thể áp dụng cho các page — on-caller có thể bắt đầu đẩy các thứ lên người thứ cấp của họ, hoặc hạ cấp các page xuống các ticket.

### On-call

Kỹ sư on-call chính nên tập trung hoàn toàn vào công việc on-call. Nếu pager im lặng cho dịch vụ của bạn, các ticket hoặc công việc dựa trên interrupt khác có thể bỏ khá nhanh nên là một phần của nhiệm vụ on-call. Khi một kỹ sư on-call trong một tuần, tuần đó nên được coi là không tính đến đối với công việc dự án. Nếu một dự án quan trọng đến mức không thể để trôi qua một tuần, người đó không nên on-call. Hãy leo thang để gán ai đó khác cho ca on-call. *Một người không bao giờ nên được kỳ vọng vừa on-call vừa đạt tiến bộ trong các dự án (hoặc bất kỳ thứ gì khác có chi phí context switch cao)*.

Nhiệm vụ thứ cấp phụ thuộc vào việc những nhiệm vụ đó nặng nề như thế nào. Nếu chức năng của người thứ cấp là dự phòng cho người chính trong trường hợp rơi qua, thì có lẽ bạn có thể an toàn giả định rằng người thứ cấp cũng có thể hoàn thành công việc dự án. Nếu ai đó khác với người thứ cấp được gán để xử lý các ticket, hãy xem xét gộp các vai trò. Nếu người thứ cấp được kỳ vọng thực sự giúp đỡ người chính trong trường hợp số lượng pager cao, họ nên làm công việc interrupt, nữa.

(Ngoài lề: *Bạn không bao giờ hết công việc dọn dẹp*. Số ticket của bạn có thể ở mức không, nhưng luôn luôn có tài liệu cần cập nhật, các config cần dọn dẹp, v.v. Các kỹ sư on-call tương lai của bạn sẽ cảm ơn bạn, và điều đó có nghĩa là họ ít có khả năng làm gián đoạn bạn trong khoảng thời gian make time quý giá của bạn).

### Tickets (Các yêu cầu)

Nếu hiện tại bạn đang phân công các ticket ngẫu nhiên cho các nạn nhân trong đội, *hãy dừng lại*. Việc làm như vậy cực kỳ bất kính với thời gian của đội bạn, và hoàn toàn ngược với nguyên lý của việc không thể bị gián đoạn nhiều nhất có thể.

Các ticket nên là một vai trò thời gian toàn thời gian, trong một khoảng thời gian mà một người có thể quản lý được. Nếu bạn tình cờ ở trong vị trí đáng buồn phải chịu nhiều ticket hơn những gì có thể được đóng bởi các kỹ sư on-call chính và thứ cấp cộng lại, thì cấu trúc vòng trực ticket của bạn để có hai người xử lý các ticket tại bất kỳ thời điểm nào. Đừng phân tán tải khắp đội. Mọi người không phải là máy móc, và bạn chỉ đang gây ra các context switch ảnh hưởng đến thời gian flow quý giá.

### Các trách nhiệm liên tục (Ongoing responsibilities)

Càng nhiều càng tốt, hãy định nghĩa các vai trò cho phép bất kỳ ai trong đội tiếp nhận mantle (trách nhiệm). Nếu có một quy trình được định nghĩa rõ ràng để thực hiện và xác thực các push hoặc flip flag, thì không có lý do gì mà một người phải đưa thay đổi đó trong suốt vòng đời của nó, ngay cả sau khi họ ngừng on-call hoặc trong các gián đoạn. Định nghĩa một vai trò *push manager* (người quản lý push) có thể điều phối (juggling) các push trong suốt thời gian on-call hoặc trong các gián đoạn của họ. Chính thức hóa quy trình bàn giao — đó là một cái giá nhỏ để trả cho khoảng thời gian make time không bị gián đoạn cho những người không on-call.

### Hãy trong các gián đoạn, hoặc đừng (Be on interrupts, or don't be)

Đôi khi khi một người không trong các gián đoạn, đội nhận được một interrupt mà người đó có năng lực duy nhất để xử lý. Trong khi lý tưởng là kịch bản này không bao giờ nên xảy ra, nhưng đôi khi nó vẫn xảy ra. Bạn nên làm việc để làm cho những sự kiện như vậy hiếm.

Đôi khi mọi người làm việc trên các ticket khi họ không được gán để xử lý các ticket vì đó là một cách dễ dàng để trông bận rộn. Hành vi như vậy không có ích. Nó có nghĩa là người đó kém hiệu quả hơn họ lẽ ra nên. Họ làm méo các số liệu về việc tải ticket có thể quản lý như thế nào. Nếu một người được gán cho các ticket, nhưng hai hoặc ba người khác cũng thử với hàng đợi ticket, bạn có thể vẫn có một hàng đợi ticket không thể quản lý mặc dù bạn không nhận ra điều đó.

## Giảm các Interrupt (Reducing Interrupts)

Tải interrupt của đội bạn có thể không thể quản lý nếu nó đòi hỏi quá nhiều thành viên đội phải đồng thời trực các interrupt tại bất kỳ thời điểm nào. Có một số kỹ thuật bạn có thể sử dụng để giảm tải ticket của bạn nói chung.

### Thực sự phân tích các ticket (Actually analyze tickets)

Nhiều vòng trực ticket hoặc vòng trực on-call hoạt động như một cuộc vượt qua chướng ngại vật (gauntlet). Điều này đặc biệt đúng đối với các vòng trực ở các đội lớn. Nếu bạn chỉ trong các gián đoạn mỗi vài tháng, thì dễ dàng để chạy cuộc vượt qua chướng ngại vật,<sup>[2](#fn2)</sup> thở dài nhẹ nhõm, và sau đó quay lại các nhiệm vụ bình thường của bạn. Người kế nhiệm của bạn sau đó làm điều tương tự, và các nguyên nhân gốc rễ của các ticket không bao giờ được điều tra. Thay vì đạt được tiến bộ, đội bạn bị lầy lội bởi một loạt những người bực mình với cùng các vấn đề.

Nên có một sự bàn giao cho các ticket, cũng như cho công việc on-call. Một quy trình bàn giao duy trì trạng thái chia sẻ giữa các người xử lý ticket khi trách nhiệm chuyển giao. Ngay cả một chút nội quan vào các nguyên nhân gốc rễ của các interrupt cũng có thể cung cấp các giải pháp tốt cho việc giảm tốc độ tổng thể. *Nhiều* đội thực hiện các bàn giao on-call và các xem xét page. *Rất ít* đội làm điều tương tự cho các ticket.

Đội bạn nên thực hiện một cuộc rà soát (scrub) định kỳ cho các ticket và page, trong đó bạn xem xét các lớp interrupt để xem liệu bạn có thể xác định một nguyên nhân gốc rễ hay không. Nếu bạn nghĩ rằng nguyên nhân gốc rễ có thể được sửa trong một khoảng thời gian hợp lý, thì *im lặng các interrupt cho đến khi nguyên nhân gốc rễ được kỳ vọng sẽ được sửa*. Việc làm như vậy cung cấp sự cứu trợ cho người xử lý các interrupt và tạo ra một sự thực thi hạn chót tiện lợi cho người sửa nguyên nhân gốc rễ.

### Tôn trọng bản thân, cũng như các khách hàng của bạn (Respect yourself, as well as your customers)

Câu nói này áp dụng nhiều hơn cho các interrupt của người dùng hơn là các interrupt tự động, mặc dù các nguyên tắc đứng vững cho cả hai kịch bản. Nếu các ticket đặc biệt khó chịu hoặc nặng nề để giải quyết, bạn có thể hiệu quả sử dụng chính sách để giảm nhẹ gánh nặng.

Hãy nhớ rằng:

-   Đội bạn thiết lập mức độ dịch vụ được cung cấp bởi dịch vụ của bạn.
-   OK (Không sao) để đẩy một số nỗ lực lên các khách hàng của bạn.

Nếu đội bạn chịu trách nhiệm xử lý các ticket hoặc interrupt cho khách hàng, bạn thường có thể sử dụng chính sách để làm cho tải công việc của bạn có thể quản lý hơn. Một sửa chữa chính sách có thể là tạm thời hoặc vĩnh viễn, tùy thuộc vào điều gì có ý nghĩa. Một sửa chữa như vậy nên đạt được một sự cân bằng tốt giữa sự tôn trọng khách hàng và sự tôn trọng bản thân. Chính sách có thể là một công cụ mạnh mẽ như code.

Ví dụ, nếu bạn hỗ trợ một công cụ đặc biệt không ổn định không có nhiều tài nguyên nhà phát triển, và một số lượng nhỏ các khách hàng có nhu cầu dùng nó, hãy xem xét các lựa chọn khác. Hãy nghĩ về giá trị của thời gian bạn dành để làm interrupt cho hệ thống này, và liệu bạn đang dành thời gian này một cách khôn ngoan hay không. Vào một lúc nào đó, nếu bạn không thể có được sự chú ý cần thiết để sửa nguyên nhân gốc rễ của các vấn đề gây ra các interrupt, có lẽ thành phần bạn đang hỗ trợ không quan trọng đến thế. Bạn nên cân nhắc trả lại pager, khai tử nó, thay thế nó, hoặc một chiến lược khác theo hướng này.

Nếu có các bước cụ thể cho một interrupt tốn thời gian hoặc phức tạp, nhưng không đòi hỏi đặc quyền của bạn để hoàn thành, hãy xem xét dùng chính sách để đẩy yêu cầu trở lại cho người yêu cầu. Ví dụ, nếu mọi người cần quyên góp các tài nguyên tính toán, hãy chuẩn bị một thay đổi code hoặc config hoặc một số bước tương tự, sau đó hướng dẫn khách hàng thực hiện bước đó và gửi nó để bạn xem xét. Hãy nhớ rằng nếu khách hàng muốn một task nhất định được hoàn thành, họ nên sẵn sàng dành một chút nỗ lực để đạt được điều họ muốn.

Một lưu ý đối với các giải pháp trước đó là bạn cần tìm một sự cân bằng giữa sự tôn trọng khách hàng và bản thân. Nguyên lý dẫn dắt của bạn trong việc xây dựng một chiến lược để đối phó với các yêu cầu của khách hàng là yêu cầu nên có ý nghĩa, hợp lý, và cung cấp tất cả thông tin và công sức mà bạn cần để đáp ứng yêu cầu. Đổi lại, phản hồi của bạn nên hữu ích và kịp thời.

<a id="fn1"></a>[1](#fn1) Xem Wikipedia: Flow (psychology) (Dòng chảy (tâm lý học)), [*https://en.wikipedia.org/wiki/Flow_(psychology)*](https://en.wikipedia.org/wiki/Flow_(psychology)).

<a id="fn2"></a>[2](#fn2) Xem [*https://en.wikipedia.org/wiki/Running_the_gauntlet*](https://en.wikipedia.org/wiki/Running_the_gauntlet).

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
