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

Các hệ thống phần mềm vốn động (dynamic) và không ổn định.<sup>[1](#fn1)</sup> Một hệ thống phần mềm chỉ có thể ổn định hoàn hảo nếu nó tồn tại trong chân không. Nếu ngừng thay đổi codebase (kho mã nguồn) thì sẽ ngừng gây ra bug. Nếu phần cứng nền tảng hay thư viện không bao giờ thay đổi thì không thành phần nào trong số chúng gây ra bug. Nếu đóng băng nhóm người dùng hiện tại thì không bao giờ phải scale (mở rộng) hệ thống. Thực tế, một tóm tắt hay về cách tiếp cận SRE (Site Reliability Engineering — Kỹ thuật Độ tin cậy Trang web) đối với việc quản lý các hệ thống là: "Cuối cùng thì, công việc của chúng tôi là giữ cho sự linh hoạt (agility) và sự ổn định (stability) cân bằng trong hệ thống."<sup>[2](#fn2)</sup>

## Ổn định Hệ thống so với Sự Linh hoạt (System Stability Versus Agility)

Đôi khi hy sinh sự ổn định vì lợi ích của sự linh hoạt là đáng. Tôi thường tiếp cận một miền vấn đề không quen thuộc bằng thứ tôi gọi là code khám phá (exploratory coding) — đặt một hạn sử dụng (shelf life) rõ ràng cho bất kỳ code nào mình viết, với sự hiểu biết rằng mình sẽ cần thử và thất bại một lần để thực sự hiểu nhiệm vụ cần hoàn thành. Code đến với một ngày hết hạn có thể tự do hơn rất nhiều trong việc bao phủ kiểm thử (test coverage) và quản lý phát hành (release management), vì nó sẽ không bao giờ được vận chuyển đến production (sản xuất) hoặc bị người dùng nhìn thấy.

Đối với phần lớn [các hệ thống phần mềm production](https://sre.google/sre-book/production-environment/), chúng tôi muốn một hỗn hợp cân bằng giữa sự ổn định và sự linh hoạt. Các SRE làm việc để tạo ra các quy trình, thực hành và công cụ làm cho phần mềm đáng tin cậy hơn, đồng thời đảm bảo rằng công việc này có ít tác động nhất có thể đến sự linh hoạt của developer. Thực tế, kinh nghiệm của SRE cho thấy các quy trình đáng tin cậy có xu hướng thực sự tăng sự linh hoạt của developer: các rollout production nhanh, đáng tin cậy làm cho các thay đổi trong production dễ thấy hơn. Kết quả là, một khi một bug xuất hiện, chúng tôi mất ít thời gian hơn để tìm và sửa nó. Xây dựng độ tin cậy vào phát triển cho phép các developer tập trung chú ý vào những gì chúng tôi thực sự quan tâm — chức năng và hiệu năng của phần mềm và hệ thống của họ.

## Đức hạnh của Sự Tẻ nhạt (Virtue of Boring)

Khác gần như mọi thứ khác trong cuộc sống, "tẻ nhạt" (boring) thực sự là một thuộc tính tích cực khi nói đến phần mềm! Chúng tôi không muốn các chương trình của mình tự phát và thú vị; chúng tôi muốn chúng bám sát kịch bản và đạt được các mục tiêu business một cách có thể dự đoán được. Trong lời của kỹ sư Google Robert Muth, "Không giống như một câu chuyện điều tra, sự thiếu kích thích, kịch tính và câu đố thực sự là một thuộc tính đáng mong muốn của mã nguồn." Những bất ngờ trong production là kẻ thù không đội trời chung của SRE.

Như Fred Brooks gợi ý trong bài luận "No Silver Bullet" (Không Có Viên Đạn Bạc) [[Bro95]](https://sre.google/sre-book/bibliography#Bro95), cần xem xét kỹ sự khác biệt giữa độ phức tạp cơ bản (essential complexity) và độ phức tạp ngẫu nhiên (accidental complexity). Độ phức tạp cơ bản là độ phức tạp vốn có trong một tình huống cụ thể, không thể gỡ bỏ khỏi định nghĩa vấn đề; trong khi độ phức tạp ngẫu nhiên linh hoạt hơn và giải quyết được bằng nỗ lực kỹ thuật. Ví dụ, viết một web server (server web) đòi hỏi phải đối phó với độ phức tạp cơ bản của việc phục vụ các trang web nhanh chóng. Tuy nhiên, nếu viết web server bằng Java, chúng tôi có thể đưa thêm độ phức tạp ngẫu nhiên vào khi cố tối thiểu hóa tác động hiệu năng của thu gom rác (garbage collection).

Với mục tiêu tối thiểu hóa độ phức tạp ngẫu nhiên, các đội SRE nên:

-   Đẩy lại (push back) khi có độ phức tạp ngẫu nhiên bị đưa vào các hệ thống mà họ chịu trách nhiệm
-   Liên tục nỗ lực loại bỏ độ phức tạp trong các hệ thống mà họ đưa vào (onboard) và tiếp nhận trách nhiệm vận hành

## Tôi Sẽ Không Từ bỏ Code của Mình! (Won't Give Up My Code!)

Kỹ sư là con người, thường hình thành gắn bó cảm xúc với những sáng tạo của họ, nên những cuộc đối đầu về các đợt thanh trừng quy mô lớn trên cây mã nguồn (source tree) là không hiếm. Một số người có thể phản đối, "Sao nếu chúng tôi cần code đó sau này?" "Sao không comment (ghi chú) code ra để dễ thêm lại sau này?" hoặc "Sao không gate (chặn) code bằng một flag (cờ) thay vì xóa?" Tất cả đều là những gợi ý khủng khiếp. Các hệ thống kiểm soát phiên bản (source control systems) giúp đảo ngược các thay đổi dễ dàng, trong khi hàng trăm dòng code bị comment tạo ra xao nhãng và nhầm lẫn (đặc biệt khi các tệp mã nguồn tiếp tục phát triển); còn code không bao giờ được thực thi, nằm sau một flag luôn bị vô hiệu hóa, là một quả bom hẹn giờ ẩn dụ đang chờ phát nổ, như Knight Capital đã trải nghiệm đau đớn (xem "Order In the Matter of Knight Capital Americas LLC" [[Sec13]](https://sre.google/sre-book/bibliography#Sec13)).

Ở một mức độ nào đó, mỗi dòng code mới được viết là một gánh nặng (liability) — nhất là khi bạn xem xét một dịch vụ web được kỳ vọng khả dụng 24/7, dù điều này nghe có vẻ cực đoan. SRE thúc đẩy các thực hành giúp tất cả code đều có một mục đích thiết yếu, như xem xét kỹ code để đảm bảo nó thực sự thúc đẩy các mục tiêu business, thường xuyên xóa code chết (dead code) và xây dựng phát hiện phình to (bloat detection) vào tất cả các cấp của kiểm thử.

## Metrics "Số dòng Code Âm" (The "Negative Lines of Code" Metric)

Thuật ngữ "phần mềm phình to" (software bloat) được tạo ra để mô tả xu hướng phần mềm trở nên chậm hơn và lớn hơn theo thời gian do một dòng liên tục các tính năng bổ sung. Phần mềm phình to dường như trực giác là không mong muốn, và các khía cạnh tiêu cực của nó trở nên rõ ràng hơn nhiều khi xem xét từ góc nhìn SRE: mỗi dòng code được thay đổi hoặc thêm vào một dự án đều có khả năng gây ra khiếm khuyết và bug mới. Dự án nhỏ hơn thì dễ hiểu hơn, dễ kiểm thử hơn và thường có ít khiếm khuyết hơn. Giữ góc nhìn này trong tâm trí, chúng tôi có lẽ nên dè dặt trước ham muốn thêm tính năng mới vào một dự án. Một số lần viết code thỏa mãn nhất mà tôi từng có là xóa hàng nghìn dòng code một khi nó không còn hữu ích.

## API Tối thiểu (Minimal APIs)

Nhà thơ Pháp Antoine de Saint Exupery viết, "sự hoàn hảo cuối cùng đạt được không phải khi không còn gì thêm vào nữa, mà khi không còn gì để lấy ra nữa" [[Sai39]](https://sre.google/sre-book/bibliography#Sai39). Nguyên lý này cũng áp dụng cho việc thiết kế và xây dựng phần mềm. Các API (Application Programming Interface — Giao diện Lập trình Ứng dụng) là một biểu đạt đặc biệt rõ ràng của lý do tại sao quy tắc này nên được làm theo.

Viết các API rõ ràng, tối thiểu là một khía cạnh thiết yếu của việc quản lý sự đơn giản trong một hệ thống phần mềm. Càng ít phương thức (methods) và đối số (arguments) mà chúng tôi cung cấp cho người dùng của API, API đó càng dễ hiểu, và chúng tôi càng có thể dành nhiều nỗ lực để làm cho các phương thức đó tốt nhất có thể. Một lần nữa, một chủ đề lặp lại xuất hiện: quyết định có ý thức không nhận lấy một số vấn đề cho phép chúng tôi tập trung vào vấn đề cốt lõi và làm cho các giải pháp mà chúng tôi xác định rõ ràng trở nên tốt hơn đáng kể. Trong phần mềm, ít hơn là nhiều hơn! Một API nhỏ, đơn giản thường cũng là dấu ấn của một vấn đề được hiểu rõ.

## Tính Module hóa (Modularity)

Vượt ra ngoài các API và các binary đơn lẻ, nhiều quy tắc ngón tay cái (rules of thumb) áp dụng cho lập trình hướng đối tượng (object-oriented programming) cũng áp dụng cho việc thiết kế các hệ thống phân tán (distributed systems). Khả năng thực hiện các thay đổi trên các phần của hệ thống một cách cô lập là thiết yếu để tạo ra một hệ thống có thể hỗ trợ được. Cụ thể, liên kết lỏng lẻo (loose coupling) giữa các binary, hoặc giữa các binary và cấu hình, là một khuôn mẫu đơn giản (simplicity pattern) đồng thời thúc đẩy sự linh hoạt của developer và sự ổn định của hệ thống. Nếu một bug được phát hiện trong một chương trình là thành phần của một hệ thống lớn hơn, bug đó có thể được sửa và push (đẩy) đến production độc lập với phần còn lại của hệ thống.

Tính module hóa mà các API cung cấp có thể có vẻ đơn giản, nhưng không phải rõ ràng rằng khái niệm module hóa còn mở rộng đến cả cách các thay đổi đối với các API được đưa vào. Chỉ một thay đổi duy nhất đối với một API cũng có thể buộc các developer phải build lại toàn bộ hệ thống của họ và chấp nhận rủi ro gây ra các bug mới. Phiên bản hóa (versioning) các API cho phép các developer tiếp tục sử dụng phiên bản mà hệ thống của họ phụ thuộc vào, trong khi họ nâng cấp lên một phiên bản mới theo một cách an toàn và được suy nghĩ. Nhịp độ phát hành (release cadence) có thể thay đổi xuyên suốt một hệ thống, thay vì đòi hỏi một push production đầy đủ của toàn bộ hệ thống mỗi khi một tính năng được thêm vào hoặc cải thiện.

Khi một hệ thống trở nên phức tạp hơn, sự phân tách trách nhiệm giữa các API và giữa các binary trở nên ngày càng quan trọng. Đây là một phép loại suy trực tiếp đến thiết kế class (lớp) hướng đối tượng: cũng như ai cũng hiểu rằng viết một class "túi trộn" (grab bag) chứa các hàm không liên quan là một thực hành tồi, thì tạo và đưa vào production một binary "util" (tiện ích) hoặc "misc" (khác) cũng là một thực hành tồi. Một hệ thống phân tán được thiết kế tốt gồm các cộng tác viên (collaborators), mỗi người có một mục đích rõ ràng và có phạm vi rõ ràng.

Khái niệm module hóa cũng áp dụng cho các định dạng dữ liệu (data formats). Một trong những điểm mạnh cốt lõi và mục tiêu thiết kế của protocol buffers (bộ đệm giao thức) của Google<sup>[3](#fn3)</sup> là tạo ra một định dạng dây (wire format) tương thích ngược và tương thích tiến.

## Đơn giản trong Phát hành (Release Simplicity)

Các release đơn giản nhìn chung tốt hơn các release phức tạp. Đo lường và hiểu tác động của một thay đổi đơn lẻ dễ dàng hơn nhiều so với một lô các thay đổi được phát hành đồng thời. Nếu chúng tôi phát hành 100 thay đổi không liên quan đến một hệ thống cùng một lúc và hiệu năng trở nên tệ hơn, việc hiểu những thay đổi nào đã ảnh hưởng đến hiệu năng, và chúng làm điều đó như thế nào, sẽ tốn nhiều nỗ lực đáng kể hoặc cần thêm các công cụ đo lường. Nếu release được thực hiện trong các lô nhỏ hơn, chúng tôi có thể di chuyển nhanh hơn với nhiều niềm tin hơn, vì mỗi thay đổi code có thể được hiểu một cách cô lập trong hệ thống lớn hơn. Cách tiếp cận này đối với các release có thể được so sánh với gradient descent (xuống dốc gradient) trong machine learning (học máy), trong đó chúng tôi tìm một giải pháp tối ưu bằng cách thực hiện từng bước nhỏ một và xem xét liệu mỗi thay đổi dẫn đến một cải thiện hay suy giảm.

## Kết luận Đơn giản (Simple Conclusion)

Chương này đã lặp đi lặp lại một chủ đề: [sự đơn giản phần mềm](https://sre.google/workbook/simplicity/) là một tiên quyết cho độ tin cậy. Chúng tôi không lười biếng khi xem xét cách có thể đơn giản hóa mỗi bước của một tác vụ cụ thể, mà đang làm rõ những gì chúng tôi thực sự muốn đạt được và làm thế nào có thể thực hiện điều đó một cách dễ dàng nhất. Mỗi lần chúng tôi nói "không" với một tính năng, chúng tôi không giới hạn đổi mới, mà đang giữ cho môi trường không bừa bộn với các xao nhãng để sự tập trung luôn hướng vào đổi mới, và kỹ thuật thực sự có thể tiến hành.

<a id="fn1"></a>[1](#fn1) Điều này thường đúng với các hệ thống phức tạp nói chung; xem [[Per99]](https://sre.google/sre-book/bibliography#Per99) và [[Coo00]](https://sre.google/sre-book/bibliography#Coo00).

<a id="fn2"></a>[2](#fn2) Cụm này do quản lý trước đây của tôi, Johan Anderson, tạo ra, vào khoảng thời gian tôi trở thành một SRE.

<a id="fn3"></a>[3](#fn3) Protocol buffers, còn được gọi là "protobufs", là một cơ chế mở rộng trung lập ngôn ngữ, trung lập nền tảng, để phân chuỗi (serialization) dữ liệu có cấu trúc. Để biết thêm chi tiết, xem [*https://developers.google.com/protocol-buffers/docs/overview#a-bit-of-history*](https://developers.google.com/protocol-buffers/docs/overview#a-bit-of-history).

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
