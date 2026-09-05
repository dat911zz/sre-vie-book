# Chương 9. Sự Đơn giản (Simplicity)

> **Nguyên bản:** [Chapter 9 - Simplicity](https://sre.google/sre-book/simplicity/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Max Luebbe
*Biên tập:* Tim Harvey

> Giá của độ tin cậy là sự theo đuổi sự đơn giản tối thượng.
>
> C.A.R. Hoare, bài thuyết trình giải Turing Award

Phần mềm vốn dĩ biến động (dynamic) và thiếu ổn định.<sup>[1](#fn1)</sup> Một hệ thống phần mềm chỉ có thể đạt trạng thái ổn định hoàn hảo nếu nó tồn tại trong chân không. Nếu ngừng thay đổi codebase (kho mã nguồn), hệ thống sẽ không còn phát sinh bug. Nếu phần cứng nền tảng hay các thư viện không bao giờ thay đổi, không thành phần nào trong số đó gây ra lỗi. Nếu đóng băng nhóm người dùng hiện tại, ta sẽ không bao giờ phải scale (mở rộng) hệ thống. Thực tế, một cách tóm tắt hay về cách tiếp cận SRE (Site Reliability Engineering — Kỹ thuật Độ tin cậy Trang web) trong việc quản lý các hệ thống là: "Cuối cùng thì, công việc của chúng tôi là giữ cho sự linh hoạt (agility) và sự ổn định (stability) cân bằng trong hệ thống."<sup>[2](#fn2)</sup>

## Ổn định Hệ thống so với Sự Linh hoạt (System Stability Versus Agility)

Đôi khi, hy sinh sự ổn định để đổi lấy sự linh hoạt là điều đáng làm. Khi tiếp cận một miền vấn đề mới, tôi thường dùng cách tiếp cận mà mình gọi là code khám phá (exploratory coding) — đặt một hạn sử dụng (shelf life) rõ ràng cho bất kỳ code nào mình viết, với nhận thức rằng sẽ cần thử và thất bại một lần để thực sự hiểu nhiệm vụ cần hoàn thành. Code có ngày hết hạn có thể tự do hơn rất nhiều trong việc bao phủ kiểm thử (test coverage) và quản lý phát hành (release management), vì nó sẽ không bao giờ được đưa lên production hoặc bị người dùng nhìn thấy.

Đối với phần lớn [các hệ thống phần mềm production](https://sre.google/sre-book/production-environment/), chúng tôi muốn một hỗn hợp cân bằng giữa sự ổn định và sự linh hoạt. Các SRE làm việc để tạo ra các quy trình, thực hành và công cụ làm cho phần mềm đáng tin cậy hơn, đồng thời đảm bảo rằng công việc này có ít tác động nhất có thể đến sự linh hoạt của developer. Thực tế, kinh nghiệm của SRE cho thấy các quy trình đáng tin cậy có xu hướng thực sự tăng sự linh hoạt của developer: các rollout production nhanh, đáng tin cậy làm cho các thay đổi trong production dễ thấy hơn. Kết quả là, một khi một bug xuất hiện, chúng tôi mất ít thời gian hơn để tìm và sửa nó. Xây dựng độ tin cậy vào phát triển cho phép các developer tập trung chú ý vào những gì chúng tôi thực sự quan tâm — chức năng và hiệu năng của phần mềm và hệ thống của họ.

## Đức hạnh của Sự Tẻ nhạt (Virtue of Boring)

Khác gần như mọi thứ khác trong cuộc sống, "tẻ nhạt" (boring) thực sự là một thuộc tính tích cực khi nói đến phần mềm! Chúng tôi không muốn các chương trình của mình tự phát và thú vị; chúng tôi muốn chúng bám sát kịch bản và đạt được các mục tiêu business một cách có thể dự đoán được. Trong lời của kỹ sư Google Robert Muth, "Không giống như một câu chuyện điều tra, sự thiếu kích thích, kịch tính và câu đố thực sự là một thuộc tính đáng mong muốn của mã nguồn." Những bất ngờ trong production là kẻ thù không đội trời chung của SRE.

Như Fred Brooks đã gợi ý trong bài luận "No Silver Bullet" (Không Có Viên Đạn Bạc) [[Bro95]](https://sre.google/sre-book/bibliography#Bro95), cần phân biệt rõ giữa độ phức tạp cơ bản (essential complexity) và độ phức tạp ngẫu nhiên (accidental complexity). Độ phức tạp cơ bản là phần vốn có của một tình huống cụ thể, không thể tách khỏi định nghĩa của vấn đề; ngược lại, độ phức tạp ngẫu nhiên linh hoạt hơn và có thể giải quyết bằng nỗ lực kỹ thuật. Chẳng hạn, việc viết một web server (server web) đòi hỏi phải xử lý độ phức tạp cơ bản trong việc phục vụ các trang web nhanh chóng. Tuy nhiên, nếu chọn Java để viết web server, chúng ta sẽ phải đối mặt với độ phức tạp ngẫu nhiên phát sinh khi cố gắng tối thiểu hóa tác động của thu gom rác (garbage collection) lên hiệu năng.

Với mục tiêu tối thiểu hóa độ phức tạp ngẫu nhiên, các đội SRE nên:

-   Đẩy lại (push back) khi có độ phức tạp ngẫu nhiên bị đưa vào các hệ thống mà họ chịu trách nhiệm
-   Liên tục nỗ lực loại bỏ độ phức tạp trong các hệ thống mà họ đưa vào (onboard) và tiếp nhận trách nhiệm vận hành

## Tôi Sẽ Không Từ bỏ Code của Mình! (Won't Give Up My Code!)

Kỹ sư là con người, thường gắn bó cảm xúc với những sáng tạo của mình, nên những cuộc tranh cãi về việc xóa bỏ hàng loạt mã nguồn không phải là hiếm. Một số người có thể phản đối: “Sao nếu chúng tôi cần đoạn mã đó sau này?”, “Sao không ghi chú (comment) mã nguồn để dễ thêm lại sau này?” hoặc “Sao không chặn (gate) mã nguồn bằng một cờ (flag) thay vì xóa?” Tất cả đều là những gợi ý khủng khiếp. Các hệ thống kiểm soát phiên bản giúp đảo ngược các thay đổi dễ dàng, trong khi hàng trăm dòng mã nguồn bị ghi chú tạo ra xao nhãng và nhầm lẫn (đặc biệt khi các tệp mã nguồn tiếp tục phát triển); còn mã nguồn không bao giờ được thực thi, nằm sau một cờ luôn bị vô hiệu hóa, là một quả bom hẹn giờ ẩn dụ đang chờ phát nổ, như Knight Capital đã trải nghiệm đau đớn (xem “Order In the Matter of Knight Capital Americas LLC” [[Sec13]](https://sre.google/sre-book/bibliography#Sec13)).

Ở một mức độ nào đó, mỗi dòng code mới được viết đều là một gánh nặng (liability) — nhất là khi bạn xem xét một dịch vụ web được kỳ vọng khả dụng 24/7, dù điều này nghe có vẻ cực đoan. SRE thúc đẩy các thực hành giúp tất cả code đều có một mục đích thiết yếu, chẳng hạn như xem xét kỹ code để đảm bảo nó thực sự thúc đẩy các mục tiêu business, thường xuyên xóa code chết (dead code) và xây dựng phát hiện phình to (bloat detection) vào tất cả các cấp của kiểm thử.

## Metrics "Số dòng Code Âm" (The "Negative Lines of Code" Metric)

Thuật ngữ "phần mềm phình to" (software bloat) được tạo ra để mô tả xu hướng phần mềm trở nên chậm hơn và lớn hơn theo thời gian do một dòng liên tục các tính năng bổ sung. Phần mềm phình to dường như trực giác là không mong muốn, và các khía cạnh tiêu cực của nó trở nên rõ ràng hơn nhiều khi xem xét từ góc nhìn SRE: mỗi dòng code được thay đổi hoặc thêm vào một dự án đều có khả năng gây ra khiếm khuyết và bug mới. Dự án nhỏ hơn thì dễ hiểu hơn, dễ kiểm thử hơn và thường có ít khiếm khuyết hơn. Giữ góc nhìn này trong tâm trí, chúng tôi có lẽ nên dè dặt trước ham muốn thêm tính năng mới vào một dự án. Một số lần viết code thỏa mãn nhất mà tôi từng có là xóa hàng nghìn dòng code một khi nó không còn hữu ích.

## API Tối thiểu (Minimal APIs)

Nhà thơ Pháp Antoine de Saint Exupery viết, "sự hoàn hảo cuối cùng đạt được không phải khi không còn gì thêm vào nữa, mà khi không còn gì để lấy ra nữa" [[Sai39]](https://sre.google/sre-book/bibliography#Sai39). Nguyên lý này cũng áp dụng cho việc thiết kế và xây dựng phần mềm. Các API (Application Programming Interface — Giao diện Lập trình Ứng dụng) là một biểu đạt đặc biệt rõ ràng của lý do tại sao quy tắc này nên được làm theo.

Viết các API rõ ràng, tối giản là một khía cạnh thiết yếu của việc quản lý sự đơn giản trong hệ thống phần mềm. Càng ít phương thức (methods) và đối số (arguments) mà chúng tôi cung cấp cho người dùng, API đó càng dễ hiểu, và chúng tôi càng có thể dồn nhiều nỗ lực để tối ưu các phương thức đó. Một lần nữa, một chủ đề lặp lại xuất hiện: quyết định có chủ đích không nhận lấy một số vấn đề cho phép chúng tôi tập trung vào vấn đề cốt lõi và làm cho các giải pháp mà chúng tôi xác định rõ ràng trở nên tốt hơn đáng kể. Trong phần mềm, ít hơn là nhiều hơn! Một API nhỏ, đơn giản thường cũng là dấu ấn của một vấn đề được hiểu rõ.

## Tính Module hóa (Modularity)

Vượt ra ngoài các API và các binary đơn lẻ, nhiều nguyên tắc kinh nghiệm (rules of thumb) áp dụng cho lập trình hướng đối tượng (object-oriented programming) cũng áp dụng cho việc thiết kế các hệ thống phân tán (distributed systems). Khả năng thực hiện các thay đổi trên các phần của hệ thống một cách cô lập là thiết yếu để tạo ra một hệ thống có thể hỗ trợ được. Cụ thể, liên kết lỏng lẻo (loose coupling) giữa các binary, hoặc giữa các binary và cấu hình, là một khuôn mẫu đơn giản (simplicity pattern) đồng thời thúc đẩy sự linh hoạt của developer và sự ổn định của hệ thống. Nếu một bug được phát hiện trong một chương trình là thành phần của một hệ thống lớn hơn, bug đó có thể được sửa và push (đẩy) đến production độc lập với phần còn lại của hệ thống.

Tính module hóa mà các API mang lại có thể trông đơn giản, nhưng chưa rõ liệu khái niệm này có mở rộng đến cả cách các thay đổi của API được tích hợp hay không. Chỉ một thay đổi duy nhất trên một API cũng có thể buộc các developer phải build lại toàn bộ hệ thống và chấp nhận rủi ro phát sinh bug mới. Phiên bản hóa (versioning) API cho phép các developer tiếp tục sử dụng phiên bản mà hệ thống của họ phụ thuộc, trong khi nâng cấp lên phiên bản mới một cách an toàn và có cân nhắc. Nhịp độ phát hành (release cadence) có thể thay đổi xuyên suốt hệ thống, thay vì đòi hỏi một push production đầy đủ của toàn bộ hệ thống mỗi khi một tính năng được thêm vào hoặc cải thiện.

Khi hệ thống ngày càng phức tạp, việc phân tách trách nhiệm giữa các API và giữa các binary trở nên quan trọng hơn. Đây là phép loại suy trực tiếp với thiết kế class (lớp) hướng đối tượng: ai cũng hiểu rằng viết một class "túi trộn" (grab bag) chứa các hàm không liên quan là một thực hành tồi, tương tự như việc tạo và đưa vào production một binary "util" (tiện ích) hoặc "misc" (khác). Một hệ thống phân tán được thiết kế tốt bao gồm các thành phần cộng tác (collaborators), mỗi thành phần có một mục đích rõ ràng và phạm vi rõ ràng.

Khái niệm module hóa cũng áp dụng cho các định dạng dữ liệu (data formats). Một trong những điểm mạnh cốt lõi và mục tiêu thiết kế của protocol buffers (bộ đệm giao thức) của Google<sup>[3](#fn3)</sup> là tạo ra một định dạng dây (wire format) tương thích ngược và tương thích tiến.

## Đơn giản trong Phát hành (Release Simplicity)

Các release đơn giản nhìn chung tốt hơn release phức tạp. Việc đo lường và hiểu tác động của một thay đổi đơn lẻ dễ dàng hơn nhiều so với một lô thay đổi được phát hành đồng thời. Nếu chúng tôi phát hành 100 thay đổi không liên quan đến một hệ thống cùng lúc và hiệu năng trở nên tệ hơn, việc xác định những thay đổi nào đã ảnh hưởng đến hiệu năng, cũng như cơ chế gây ra ảnh hưởng đó, sẽ đòi hỏi nỗ lực đáng kể hoặc cần thêm các công cụ đo lường. Nếu release được thực hiện trong các lô nhỏ hơn, chúng tôi có thể di chuyển nhanh hơn với nhiều niềm tin hơn, vì mỗi thay đổi code có thể được hiểu một cách cô lập trong hệ thống lớn hơn. Cách tiếp cận này đối với các release có thể được so sánh với gradient descent (xuống dốc gradient) trong machine learning (học máy), trong đó chúng tôi tìm một giải pháp tối ưu bằng cách thực hiện từng bước nhỏ một và xem xét liệu mỗi thay đổi dẫn đến một cải thiện hay suy giảm.

## Kết luận Đơn giản (Simple Conclusion)

Chương này đã nhiều lần nhấn mạnh một chủ đề: [sự đơn giản phần mềm](https://sre.google/workbook/simplicity/) là điều kiện tiên quyết cho độ tin cậy. Khi xem xét cách đơn giản hóa từng bước của một tác vụ cụ thể, chúng tôi không hề lười biếng; thay vào đó, chúng tôi đang làm rõ mục tiêu thực sự cần đạt được và cách thực hiện nó một cách dễ dàng nhất. Mỗi lần từ chối một tính năng, chúng tôi không hề kìm hãm đổi mới, mà đang giữ cho môi trường không bị bừa bộn bởi những yếu tố gây xao nhãng, nhờ đó sự tập trung luôn hướng vào đổi mới và kỹ thuật thực sự có thể tiến hành.

<a id="fn1"></a>[1](#fn1) Điều này thường đúng với các hệ thống phức tạp nói chung; xem [[Per99]](https://sre.google/sre-book/bibliography#Per99) và [[Coo00]](https://sre.google/sre-book/bibliography#Coo00).

<a id="fn2"></a>[2](#fn2) Cụm này do quản lý trước đây của tôi, Johan Anderson, tạo ra, vào khoảng thời gian tôi trở thành một SRE.

<a id="fn3"></a>[3](#fn3) Protocol buffers, còn được gọi là "protobufs", là một cơ chế mở rộng trung lập ngôn ngữ, trung lập nền tảng, để phân chuỗi (serialization) dữ liệu có cấu trúc. Để biết thêm chi tiết, xem [*https://developers.google.com/protocol-buffers/docs/overview#a-bit-of-history*](https://developers.google.com/protocol-buffers/docs/overview#a-bit-of-history).

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
