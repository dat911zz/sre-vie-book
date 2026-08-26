# Chương 1. Giới thiệu (Introduction)

> **Nguyên bản:** [Chapter 1 - Introduction](https://sre.google/sre-book/introduction/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Đội biên tập Softdreams RnD)

---

*Tác giả:* Benjamin Treynor Sloss<sup>[1](#fn1)</sup>
*Biên tập:* Betsy Beyer

> Hy vọng không phải là một chiến lược.
>
> Một câu nói quen thuộc trong SRE

Một sự thật mà ai cũng thừa nhận là các hệ thống không thể tự vận hành. Vậy thì, một hệ thống — đặc biệt là một hệ thống máy tính phức tạp vận hành ở quy mô lớn — *nên* được vận hành như thế nào?

## Cách tiếp cận của Sysadmin đối với quản lý dịch vụ

Trước nay, các công ty vẫn thuê quản trị viên hệ thống (system administrator, hay sysadmin) để vận hành những hệ thống máy tính phức tạp.

Theo cách tiếp cận sysadmin, người ta lắp ráp các thành phần phần mềm có sẵn rồi triển khai chúng phối hợp với nhau thành một dịch vụ. Sysadmin nhận nhiệm vụ vận hành dịch vụ đó, xử lý các sự kiện (events) và các đợt cập nhật mỗi khi phát sinh. Hệ thống càng phức tạp, traffic càng lớn thì sự kiện và cập nhật càng nhiều, kéo theo đội sysadmin cũng phải phình to để kham hết khối lượng công việc tăng thêm. Vì vai trò sysadmin đòi hỏi bộ kỹ năng khác hẳn đội phát triển sản phẩm, developer và sysadmin được tách thành hai đội riêng: "development" (phát triển) và "operations" (vận hành, hay "ops").

Mô hình quản lý dịch vụ kiểu sysadmin có một số ưu điểm. Đối với các công ty đang cân nhắc cách vận hành và nhân sự cho một dịch vụ, cách tiếp cận này tương đối dễ triển khai: vì đây là mô hình quen thuộc trong ngành, có rất nhiều ví dụ để học hỏi và noi theo. Nhân sự phù hợp cũng không thiếu. Hàng loạt công cụ, thành phần phần mềm (mua sẵn hay tự làm), và các công ty tích hợp đều có sẵn để hỗ trợ vận hành những hệ thống đã lắp ráp, nên một đội sysadmin mới vào việc không cần phải phát minh lại bánh xe hay thiết kế hệ thống từ con số không.

Nhưng cách tiếp cận sysadmin, cùng lối phân tách development/ops đi kèm, cũng có không ít nhược điểm và cạm bẫy. Nhìn chung, những nhược điểm này có thể chia thành hai nhóm: chi phí trực tiếp và chi phí gián tiếp.

Chi phí trực tiếp thì hiển hiện, chẳng có gì mơ hồ. Vận hành dịch vụ bằng một đội chuyên can thiệp thủ công — cả trong quản lý thay đổi (change management) lẫn xử lý sự kiện (event handling) — sẽ ngày càng tốn kém khi dịch vụ lớn lên hoặc traffic tăng, vì quy mô đội buộc phải tăng theo lượng tải mà hệ thống sinh ra.

Chi phí gián tiếp của sự phân tách development/ops có thể tinh vi, nhưng thường đắt đỏ hơn cho tổ chức so với chi phí trực tiếp. Những chi phí này bắt nguồn từ chỗ hai đội có nền tảng, kỹ năng và động lực hoàn toàn khác nhau. Họ dùng ngôn ngữ khác nhau để mô tả tình huống; họ mang những giả định khác nhau về cả rủi ro lẫn khả năng của các giải pháp kỹ thuật; họ có những kỳ vọng khác nhau về mức độ ổn định mục tiêu của sản phẩm. Sự chia rẽ giữa hai nhóm dễ lan từ chuyện động lực sang cách giao tiếp, mục tiêu, rồi cuối cùng là cả niềm tin lẫn sự tôn trọng dành cho nhau. Đến mức đó thì sự chia rẽ đã thành một dạng bệnh lý.

Các đội ops truyền thống và đối tác của họ trong phát triển sản phẩm vì thế thường rơi vào xung đột, rõ nhất là về việc phần mềm có thể được release (phát hành) lên production nhanh đến đâu. Với đội development, điều quan trọng nhất là ra mắt tính năng mới và thấy người dùng đón nhận. Với đội ops, điều quan trọng nhất lại là làm sao cho dịch vụ đừng hỏng trong ca mình cầm pager (thiết bị gọi trực). Mà hầu hết các outage (mất dịch vụ) đều bắt nguồn từ một thay đổi nào đó — cấu hình mới, tính năng mới ra mắt, hay một kiểu traffic người dùng mới — nên mục tiêu của hai đội, xét đến cùng, đối chọi trực tiếp với nhau.

Cả hai nhóm đều hiểu rằng không thể nói toạc mong muốn của mình một cách trần trụi như vậy ("Chúng tôi muốn ra mắt bất cứ thứ gì, bất cứ lúc nào, không bị cản trở" đối lại "Chúng tôi không bao giờ muốn thay đổi bất cứ thứ gì một khi hệ thống đã chạy ổn"). Và vì ngôn ngữ lẫn giả định về rủi ro của họ khác nhau, hai bên thường sa vào một cuộc giằng co dai dẳng quen thuộc, mỗi bên tìm cách đẩy lợi ích của mình lên. Đội ops cố bảo vệ hệ thống đang chạy trước rủi ro từ thay đổi bằng cách dựng lên các cổng kiểm soát ra mắt và thay đổi (launch and change gates). Ví dụ, một buổi xem xét ra mắt (launch review) có thể kèm theo danh mục rà soát *mọi* vấn đề từng gây outage trong quá khứ — một danh sách có thể dài vô hạn định, mà không phải mục nào cũng đáng giá như nhau. Đội dev nhanh chóng học được cách đối phó: bớt "launch" (ra mắt) đi, thay vào đó là nhiều "flag flip" (đổi cờ tính năng), "incremental update" (cập nhật từng phần), hay "cherrypick" (cherry-pick) hơn. Họ còn dùng những chiến thuật như sharding (chia nhỏ) sản phẩm, để càng ít tính năng phải qua launch review càng tốt.

## Cách tiếp cận của Google đối với quản lý dịch vụ: Site Reliability Engineering

Xung đột không phải là một phần tất yếu của việc cung cấp một dịch vụ phần mềm. Google chọn một cách khác để vận hành hệ thống của mình: các [đội Site Reliability Engineering](https://sre.google/sre-book/software-engineering-in-sre/) của chúng tôi tập trung tuyển kỹ sư phần mềm vào vận hành sản phẩm, và [xây dựng những hệ thống đảm nhận phần việc](https://sre.google/sre-book/distributed-periodic-scheduling/) mà trước đây các sysadmin thường phải làm thủ công.

Chính xác thì Site Reliability Engineering, theo cách nó đã được định nghĩa tại Google, là gì? Cách giải thích của tôi rất đơn giản: SRE là những gì bạn có được khi giao cho một kỹ sư phần mềm thiết kế đội vận hành. Trước khi gia nhập Google năm 2003 và nhận nhiệm vụ dẫn dắt một "Production Team" gồm bảy kỹ sư, tôi thuần túy chỉ làm kỹ thuật phần mềm. Vì vậy, tôi thiết kế và quản lý nhóm theo đúng cách mà *tôi* mong muốn, nếu chính tôi là một SRE trong nhóm. Nhóm đó dần lớn thành đội SRE ngày nay của Google, và vẫn giữ đúng tinh thần mà một người cả đời làm phần mềm đã hình dung từ đầu.

Một nền tảng quan trọng trong cách Google quản lý dịch vụ là cơ cấu nhân sự của mỗi đội SRE. Nhìn chung, SRE có thể chia thành hai nhóm chính.

50–60% là Google Software Engineer, hay chính xác hơn, những người được tuyển dụng thông qua quy trình chuẩn cho Google Software Engineer. 40–50% còn lại là các ứng viên rất gần với tiêu chuẩn của Google Software Engineer (tức là đạt 85–99% bộ kỹ năng yêu cầu), và *ngoài ra* có một bộ kỹ năng kỹ thuật hữu ích cho SRE nhưng hiếm gặp ở phần lớn các kỹ sư phần mềm. Đại thể, kiến thức nội tại hệ thống UNIX (UNIX system internals) và mạng (networking, từ Layer 1 đến Layer 3) là hai loại kỹ năng kỹ thuật thay thế mà chúng tôi tìm kiếm nhiều nhất.

Điểm chung của mọi SRE là đều tin vào giá trị của việc phát triển hệ thống phần mềm để giải quyết vấn đề phức tạp, và đều đủ năng lực làm việc đó. Trong SRE, chúng tôi theo dõi sát bước tiến nghề nghiệp của cả hai nhóm, và đến nay chưa thấy khác biệt thực tế nào về hiệu suất giữa kỹ sư của hai nguồn tuyển. Thực tế, nền tảng tương đối đa dạng của đội SRE thường dẫn đến những hệ thống thông minh, chất lượng cao, rõ ràng là sản phẩm của sự kết hợp giữa nhiều bộ kỹ năng.

Kết quả của cách chúng tôi tuyển dụng cho SRE là chúng tôi có được một đội gồm những người (a) sẽ nhanh chóng cảm thấy nhàm chán khi thực hiện các tác vụ bằng tay, và (b) có đủ kỹ năng để viết phần mềm thay cho phần việc thủ công trước kia, kể cả khi lời giải không hề đơn giản. Các SRE cũng chia sẻ nền tảng học thuật và trí tuệ với phần còn lại của tổ chức phát triển. Do đó, SRE về cơ bản đang thực hiện công việc mà trước đây do một đội ops đảm nhận, nhưng bằng các kỹ sư có chuyên môn phần mềm, và đặt cược rằng những kỹ sư này vốn vừa có thiên hướng, vừa có khả năng thiết kế và [triển khai tự động hóa](https://sre.google/sre-book/automation-at-google/) bằng phần mềm để thay cho sức người.

Ngay từ thiết kế, điều cốt yếu là các đội SRE phải tập trung vào công việc kỹ thuật (engineering). Thiếu hoạt động kỹ thuật liên tục, tải vận hành sẽ tăng dần và đội sẽ phải thêm người chỉ để theo kịp khối lượng công việc. Rốt cuộc, một nhóm thiên về ops truyền thống sẽ phình to tuyến tính theo quy mô dịch vụ: sản phẩm mà dịch vụ hỗ trợ càng thành công, tải vận hành càng tăng theo traffic, nghĩa là lại phải tuyển thêm người để làm đi làm lại cùng những tác vụ cũ.

Để tránh số phận này, đội được giao quản lý một dịch vụ phải viết code, nếu không sẽ bị nhấn chìm. Vì vậy, Google đặt *mức trần 50% cho tổng khối lượng công việc "ops" của tất cả các SRE* — ticket, on-call (trực sự cố), tác vụ thủ công, v.v. Mức trần này đảm bảo đội SRE còn đủ quỹ thời gian để làm cho dịch vụ ổn định và dễ vận hành. Đây là giới hạn trên; nếu cứ để mọi việc diễn ra tự nhiên, theo thời gian đội SRE nên tiến dần về trạng thái gần như không còn tải vận hành, dành trọn thời gian cho công việc phát triển, vì khi đó dịch vụ về cơ bản đã tự chạy và tự sửa chữa: chúng tôi muốn những hệ thống *tự động* (automatic), chứ không chỉ *được tự động hóa* (automated). Còn trên thực tế, chuyện mở rộng quy mô và tính năng mới chẳng bao giờ để SRE được ngơi tay.

Nguyên tắc của Google là đội SRE phải dành 50% thời gian còn lại để thực sự làm công việc phát triển. Vậy chúng tôi thực thi ngưỡng đó như thế nào? Trước tiên phải đo xem thời gian của SRE đang được dùng vào đâu. Có số liệu trong tay rồi, đội nào liên tục dành chưa tới 50% thời gian cho việc phát triển sẽ phải điều chỉnh cách làm. Thường thì điều đó nghĩa là chuyển bớt gánh nặng vận hành về cho đội phát triển, hoặc bổ sung người cho đội mà không giao thêm trách nhiệm vận hành. Chủ động giữ thế cân bằng ops–phát triển này giúp chúng tôi bảo đảm SRE có đủ thời gian cho công việc kỹ thuật sáng tạo, tự chủ, mà vẫn giữ được vốn hiểu biết đúc kết từ chính việc vận hành dịch vụ.

Chúng tôi thấy rằng cách tiếp cận của Google SRE trong việc vận hành các hệ thống quy mô lớn có nhiều ưu điểm. Vì SRE trực tiếp sửa code để theo đuổi mục tiêu làm cho hệ thống của Google tự vận hành, các đội SRE vừa đổi mới rất nhanh, vừa rộng lòng đón nhận thay đổi. Những đội như vậy cũng tương đối rẻ — cùng một dịch vụ đó mà giao cho một đội thiên về ops hỗ trợ thì sẽ cần nhiều người hơn đáng kể. Trong khi đó, số SRE cần để vận hành, bảo trì và cải thiện một hệ thống tăng chậm hơn tuyến tính (sublinear) so với quy mô hệ thống. Cuối cùng, SRE không chỉ tránh được thế rối loạn của sự phân tách dev/ops, mà cấu trúc này còn giúp chính các đội phát triển sản phẩm tốt lên: việc luân chuyển dễ dàng giữa đội phát triển sản phẩm và SRE tạo cơ hội đào tạo chéo cho cả hai phía, và nâng tay nghề cho những developer vốn khó có dịp nào khác để học cách xây dựng một hệ thống phân tán một triệu core.

Dù lợi ích tổng thể là vậy, [mô hình SRE](https://sre.google/sre-book/engagement-model/) cũng có những thách thức riêng. Thách thức thường trực với Google là chuyện tuyển SRE: SRE không chỉ phải giành cùng một nguồn ứng viên với kênh tuyển dụng phát triển sản phẩm, mà vì chúng tôi đặt chuẩn tuyển rất cao ở cả kỹ năng coding lẫn [kỹ thuật hệ thống](https://sre.google/sre-book/release-engineering/), nguồn ứng viên đạt yêu cầu ắt hẳn rất hẹp. Vì lĩnh vực này còn tương đối mới mẻ và khác biệt, trong ngành chưa có nhiều tài liệu về [cách xây dựng và quản lý một đội SRE](https://sre.google/sre-book/part-IV-management/) (hy vọng quyển sách này sẽ góp một bước theo hướng đó!). Và một khi đội SRE đã thành hình, cách quản lý dịch vụ có phần khác lối thường của họ sẽ cần sự hậu thuẫn mạnh mẽ từ cấp quản lý. Ví dụ, quyết định ngừng release cho hết quý một khi ngân sách lỗi (error budget) đã cạn sẽ khó được đội phát triển sản phẩm chấp nhận, trừ khi có chỉ thị từ chính cấp quản lý của họ.

## DevOps hay SRE?

Thuật ngữ "DevOps" xuất hiện trong ngành vào cuối năm 2008 và cho đến thời điểm viết bài này (đầu năm 2016) vẫn đang trong trạng thái biến động. Các nguyên lý cốt lõi của nó — sự tham gia của chức năng IT trong mỗi giai đoạn thiết kế và phát triển hệ thống, sự phụ thuộc lớn vào tự động hóa thay vì nỗ lực con người, việc áp dụng các thực hành và công cụ kỹ thuật vào các tác vụ vận hành — nhất quán với nhiều [nguyên lý và thực hành của SRE](https://sre.google/sre-book/part-II-principles/). Có thể coi DevOps như một sự khái quát hóa của một số nguyên lý cốt lõi của SRE cho một phạm vi rộng hơn các tổ chức, cấu trúc quản lý và nhân sự. Ngược lại, cũng có thể coi SRE là một triển khai cụ thể của DevOps, kèm theo một số mở rộng đặc thù.

## Các nguyên tắc của SRE

Mặc dù chi tiết về luồng công việc, thứ tự ưu tiên và hoạt động hàng ngày mỗi đội SRE mỗi khác, tất cả đều gánh chung một bộ trách nhiệm cơ bản đối với dịch vụ mình hỗ trợ, và cùng tuân theo những nguyên tắc cốt lõi giống nhau. Nhìn chung, một đội SRE chịu trách nhiệm về *khả dụng (availability), độ trễ (latency), hiệu năng (performance), hiệu quả (efficiency), quản lý thay đổi (change management), giám sát (monitoring), phản ứng khẩn cấp (emergency response), và lập kế hoạch năng lực (capacity planning)* cho dịch vụ của họ. Chúng tôi đã đúc kết thành quy tắc và nguyên lý hẳn hoi cách các đội SRE tương tác với môi trường xung quanh — không chỉ [môi trường production](https://sre.google/sre-book/production-environment/), mà cả các đội phát triển sản phẩm, đội kiểm thử, người dùng, v.v. Những quy tắc và thực hành làm việc đó giúp chúng tôi duy trì sự tập trung vào công việc kỹ thuật, thay vì công việc vận hành.

Phần tiếp theo thảo luận từng nguyên tắc cốt lõi của Google SRE.

## Đảm bảo sự tập trung bền bỉ vào kỹ thuật

Như đã thảo luận, Google đặt trần công việc vận hành cho SRE ở mức 50% thời gian của họ. Thời gian còn lại, họ nên dùng kỹ năng coding của mình cho công việc dự án. Trên thực tế, điều này được thực hiện bằng cách giám sát khối lượng công việc vận hành mà SRE đang làm, và chuyển hướng [công việc vận hành thừa](https://sre.google/sre-book/dealing-with-interrupts/) sang các đội phát triển sản phẩm: giao lại các bug và ticket cho quản lý phát triển, [tái] tích hợp các developer vào vòng trực pager on-call, v.v. Việc chuyển bớt này dừng lại khi tải vận hành đã xuống mức 50% hoặc thấp hơn. Điều này cũng cung cấp một cơ chế phản hồi hiệu quả, hướng các developer xây dựng các hệ thống không cần can thiệp thủ công. Cách tiếp cận này hoạt động tốt khi toàn bộ tổ chức — cả SRE lẫn phát triển — hiểu tại sao cơ chế van an toàn tồn tại, và cùng hướng tới mục tiêu không để xảy ra sự kiện tràn (overflow events) — tức sản phẩm không sinh ra lượng tải vận hành lớn đến mức phải cần tới van an toàn này.

Trong thời gian làm công việc vận hành, trung bình mỗi ca trực on-call 8–12 giờ, một SRE chỉ nên nhận tối đa hai sự kiện. Mức khối lượng mục tiêu này cho kỹ sư on-call đủ thời gian để xử lý sự kiện nhanh chóng và chính xác, dọn dẹp và khôi phục dịch vụ về bình thường, rồi viết một bản postmortem (báo cáo sau sự cố). Nếu thường xuyên có nhiều hơn hai sự kiện mỗi ca trực, các vấn đề không thể được điều tra kỹ lưỡng và các kỹ sư bị quá tải đến mức không thể học hỏi từ những sự kiện này. Tình trạng kiệt sức vì pager (pager fatigue) cũng không hề giảm khi hệ thống scale lên. Ngược lại, nếu SRE on-call liên tục nhận chưa tới một sự kiện mỗi ca, bắt họ ngồi trực chỉ lãng phí thời gian của họ.

Postmortem nên được viết cho mọi incident (sự cố) quan trọng, bất kể có kích hoạt pager hay không; những postmortem không đi kèm pager thậm chí còn giá trị hơn, vì nhiều khả năng chúng phơi ra các lỗ hổng giám sát rõ ràng. Cuộc điều tra cần dựng lại chi tiết chuyện gì đã xảy ra, tìm cho ra mọi nguyên nhân gốc rễ, và giao việc cụ thể để khắc phục vấn đề hoặc ứng phó tốt hơn ở lần sau. Google vận hành theo *văn hóa postmortem không đổ lỗi* (blame-free postmortem culture), với mục tiêu phơi bày sai sót và dùng kỹ thuật để sửa chữa, thay vì né tránh hay xem nhẹ chúng.

## Theo đuổi tốc độ thay đổi tối đa mà không vi phạm SLO của dịch vụ

Các đội phát triển sản phẩm và SRE có thể tận hưởng một [mối quan hệ làm việc hiệu quả](https://sre.google/sre-book/engagement-model/) bằng cách loại bỏ mâu thuẫn cấu trúc trong các mục tiêu riêng của họ. Mâu thuẫn cấu trúc ở đây nằm giữa nhịp độ đổi mới và độ ổn định của sản phẩm; như đã mô tả ở trên, mâu thuẫn này thường chỉ bộc lộ một cách gián tiếp. Trong SRE, chúng tôi đưa mâu thuẫn đó ra ánh sáng, rồi giải quyết bằng khái niệm *ngân sách lỗi (error budget)*.

Error budget bắt nguồn từ quan sát rằng *100% là mục tiêu độ tin cậy sai cho hầu như mọi thứ* (máy tạo nhịp tim và hệ thống chống bó cứng phanh là những ngoại lệ đáng chú ý). Nhìn chung, đối với bất kỳ dịch vụ hay hệ thống phần mềm nào, 100% không phải là mục tiêu độ tin cậy đúng bởi chẳng người dùng nào phân biệt nổi một hệ thống khả dụng 100% với một hệ thống khả dụng 99.999%. Giữa người dùng và dịch vụ còn vô số hệ thống khác nằm trên đường truyền (laptop của họ, WiFi nhà họ, ISP, lưới điện…) và những hệ thống đó gộp lại có khả dụng thấp hơn rất nhiều so với 99.999%. Vì vậy, chút khác biệt giữa 99.999% và 100% chìm lẫn trong nhiễu từ những nguồn bất khả dụng khác, và người dùng chẳng hưởng lợi gì từ nỗ lực khổng lồ bỏ ra chỉ để thêm 0.001% khả dụng đó.

Nếu 100% là mục tiêu độ tin cậy sai cho một hệ thống, vậy thì mục tiêu độ tin cậy đúng cho hệ thống đó là gì? Thực ra đây không phải là một câu hỏi kỹ thuật mà là một câu hỏi sản phẩm, cần cân nhắc các yếu tố sau:

-   Xét theo cách người dùng sử dụng sản phẩm, họ sẽ hài lòng với mức khả dụng nào?
-   Nếu không hài lòng với mức khả dụng của sản phẩm, người dùng có những lựa chọn thay thế nào?
-   Ở từng mức khả dụng khác nhau, cách người dùng sử dụng sản phẩm thay đổi ra sao?

Bên business hay bên sản phẩm phải là người xác lập mục tiêu khả dụng của hệ thống. Có mục tiêu rồi, error budget chính là một trừ đi mục tiêu khả dụng. Một dịch vụ khả dụng 99.99% nghĩa là được phép không khả dụng 0.01%. Phần 0.01% được phép đó chính là *error budget* của dịch vụ. Chúng ta có thể tiêu ngân sách này vào bất cứ thứ gì mình muốn, miễn đừng tiêu lố.

Vậy chúng ta muốn tiêu error budget như thế nào? Đội phát triển muốn ra mắt tính năng và thu hút người dùng mới. Lý tưởng nhất, chúng ta sẽ tiêu hết error budget bằng cách chấp nhận rủi ro khi ra mắt, đổi lấy tốc độ ra mắt nhanh hơn. Toàn bộ mô hình error budget gói gọn trong tiền đề cơ bản đó. Một khi hoạt động của SRE được nhìn qua lăng kính này, thì việc tiết kiệm error budget nhờ những chiến thuật như rollout phân giai đoạn (phased rollouts) hay thí nghiệm 1% đều có thể dùng để ra mắt nhanh hơn.

Việc sử dụng error budget giải quyết mâu thuẫn cấu trúc về động lực giữa phát triển và SRE. Mục tiêu của SRE không còn là "zero outages" (không có sự cố); thay vào đó, SRE và developer sản phẩm nhắm đến việc tiêu error budget để đạt tốc độ tính năng (feature velocity) tối đa. Sự thay đổi này tạo ra khác biệt to lớn. Outage không còn là một điều "xấu" — nó là phần được dự liệu trước của quá trình đổi mới, là thứ mà đội phát triển và SRE cùng nhau quản lý chứ không nơm nớp lo sợ.

## Giám sát (Monitoring)

Giám sát (monitoring) là một trong những phương tiện chính để chủ sở hữu dịch vụ theo dõi sức khỏe và khả dụng của hệ thống. Do đó, chiến lược giám sát cần được xây dựng một cách kỹ lưỡng, có chủ đích. Một cách tiếp cận giám sát cổ điển và phổ biến là quan sát một giá trị hoặc điều kiện cụ thể, rồi kích hoạt một cảnh báo email khi giá trị đó vượt quá hoặc điều kiện đó xảy ra. Tuy nhiên, kiểu cảnh báo email này không phải giải pháp hiệu quả: một hệ thống mà phải có người đọc email rồi tự phán đoán xem có cần làm gì hay không thì ngay từ gốc đã là thiếu sót. Giám sát không bao giờ nên bắt con người phải diễn giải bất kỳ phần nào của miền cảnh báo. Việc diễn giải hãy để phần mềm lo; con người chỉ nên được gọi đến khi cần họ hành động.

Có ba loại đầu ra giám sát hợp lệ:

#### Cảnh báo (Alerts)

Báo hiệu rằng con người cần hành động ngay lập tức trước điều đang xảy ra hoặc sắp xảy ra, để cải thiện tình hình.

#### Ticket (Yêu cầu xử lý)

Báo hiệu rằng con người cần hành động, nhưng chưa phải ngay lập tức. Hệ thống không thể tự xử lý tình huống, nhưng chỉ cần có người xử lý trong vài ngày tới thì sẽ không có thiệt hại gì.

#### Ghi log (Logging)

Không ai cần phải xem thông tin này; nó được ghi lại để phục vụ chẩn đoán hoặc điều tra pháp y kỹ thuật (forensic). Mặc định là sẽ chẳng ai đọc log, trừ khi có chuyện gì đó buộc họ phải đọc.

## Phản ứng khẩn cấp (Emergency Response)

Độ tin cậy là một hàm của thời gian trung bình đến khi hỏng (mean time to failure, MTTF) và thời gian trung bình để sửa chữa (mean time to repair, MTTR) [[Sch15]](https://sre.google/sre-book/bibliography#Sch15). Để đánh giá hiệu quả của phản ứng khẩn cấp, chỉ số sát sườn nhất là đội phản ứng đưa được hệ thống trở lại trạng thái khỏe mạnh nhanh đến đâu — tức MTTR.

Con người làm tăng độ trễ. Một hệ thống tránh được những tình huống khẩn cấp cần con người can thiệp, dù có gặp nhiều hỏng hóc *thực tế* hơn, vẫn sẽ đạt khả dụng cao hơn một hệ thống cứ phải có bàn tay người xử lý. Khi buộc phải cần đến con người, chúng tôi nhận thấy việc nghĩ trước và ghi sẵn các thực hành tốt nhất vào một "playbook" (sách hướng dẫn) giúp MTTR cải thiện khoảng 3 lần so với lối ứng biến tại trận. Kỹ sư on-call kiểu "nghề gì cũng biết" (jack-of-all-trades) vẫn hữu dụng, nhưng kỹ sư được rèn luyện bài bản và có sẵn playbook thì hiệu quả hơn nhiều. Dĩ nhiên không playbook nào, dù toàn diện đến đâu, thay thế được những kỹ sư thông minh ứng biến tại chỗ, nhưng các [bước và mẹo troubleshooting (xử lý sự cố)](https://sre.google/sre-book/effective-troubleshooting/) rõ ràng, kỹ lưỡng vẫn rất đáng giá khi phải phản ứng với một page (cảnh báo) rủi ro cao hoặc gấp gáp về thời gian. Do đó, Google SRE dựa vào các playbook on-call, bên cạnh các bài tập như "Wheel of Misfortune" (Bánh xe Xui xẻo)<sup>[2](#fn2)</sup>, để chuẩn bị cho kỹ sư phản ứng với các sự kiện on-call.

## Quản lý thay đổi (Change Management)

SRE nhận thấy khoảng 70% outage bắt nguồn từ thay đổi trên hệ thống đang chạy. Các thực hành tốt nhất trong lĩnh vực này sử dụng tự động hóa để đạt được:

-   Triển khai rollout tăng dần (progressive rollouts)
-   Phát hiện vấn đề nhanh và chính xác
-   Rollback (hoàn tác) thay đổi một cách an toàn khi vấn đề nảy sinh

Bộ ba thực hành này giúp giảm tối đa tổng số người dùng và thao tác phải hứng chịu các thay đổi lỗi. Nhờ loại con người ra khỏi vòng xử lý, chúng cũng tránh được những vấn đề cố hữu của các tác vụ lặp đi lặp lại nhiều: mệt mỏi, quen tay sinh chủ quan, và lơ đễnh. Kết quả là cả tốc độ release lẫn độ an toàn đều tăng.

## Dự báo nhu cầu và lập kế hoạch năng lực (Demand Forecasting and Capacity Planning)

Có thể hiểu dự báo nhu cầu và lập kế hoạch năng lực là việc đảm bảo đủ năng lực và mức dự phòng (redundancy) để phục vụ nhu cầu tương lai theo dự báo, ở mức khả dụng yêu cầu. Bản thân các khái niệm này chẳng có gì đặc biệt — điều đáng ngạc nhiên là rất nhiều dịch vụ và đội nhóm lại không làm những bước cần thiết để chắc rằng năng lực cần có sẽ sẵn sàng đúng lúc. Lập kế hoạch năng lực nên tính đến cả tăng trưởng hữu cơ (organic growth, bắt nguồn từ việc khách hàng chấp nhận và sử dụng sản phẩm một cách tự nhiên) lẫn tăng trưởng vô cơ (inorganic growth, kết quả từ các sự kiện như ra mắt tính năng, chiến dịch marketing, hay các thay đổi do business thúc đẩy khác).

Một số bước là bắt buộc trong lập kế hoạch năng lực:

-   Dự báo chính xác nhu cầu hữu cơ, với tầm nhìn xa hơn khoảng thời gian cần để bổ sung năng lực
-   Gộp chính xác các nguồn nhu cầu vô cơ vào bản dự báo nhu cầu
-   Chạy load test (kiểm thử tải) hệ thống thường xuyên để đối chiếu năng lực thô (server, disk, v.v.) với năng lực dịch vụ

Vì năng lực là yếu tố then chốt cho khả dụng, hệ quả tất yếu là đội SRE phải chịu trách nhiệm lập kế hoạch năng lực, và kéo theo đó là cả trách nhiệm provision (cấp phát tài nguyên).

## Cấp phát tài nguyên (Provisioning)

Provisioning kết hợp cả quản lý thay đổi và lập kế hoạch năng lực. Theo kinh nghiệm của chúng tôi, provisioning phải được thực hiện nhanh chóng và chỉ khi cần thiết, vì năng lực thì đắt. Nhưng việc này cũng phải làm cho đúng, nếu không phần năng lực thêm vào sẽ không chạy được lúc cần. Thêm năng lực mới thường bao gồm khởi động một instance (thực thể) hoặc địa điểm mới, sửa đổi đáng kể các hệ thống hiện có (tệp cấu hình, load balancer, mạng), và xác minh rằng năng lực mới hoạt động và cho kết quả đúng. Vì vậy, thao tác này rủi ro hơn load shifting (chuyển tải, vốn diễn ra nhiều lần mỗi giờ), và đòi hỏi mức cẩn trọng cao hơn tương xứng.

## Hiệu quả và hiệu năng (Efficiency and Performance)

Hễ dịch vụ còn phải tính đến tiền bạc thì việc dùng tài nguyên hiệu quả còn quan trọng. Vì SRE là người nắm quyền quyết định sau cùng về provisioning, họ cũng phải tham gia mọi công việc liên quan đến mức sử dụng (utilization), bởi mức sử dụng là hàm của cách dịch vụ hoạt động và cách nó được provision. Nói cách khác, chăm chút cho chiến lược provision của một dịch vụ — và qua đó là mức sử dụng của nó — chính là một đòn bẩy rất, rất lớn đối với tổng chi phí dịch vụ.

Việc sử dụng tài nguyên là một hàm của nhu cầu (tải), năng lực, và hiệu quả phần mềm. SRE dự báo nhu cầu, provision năng lực, và có thể sửa đổi phần mềm. Ba yếu tố này là một phần lớn (dù không phải toàn bộ) của hiệu quả dịch vụ.

Hệ thống phần mềm nào cũng chậm dần khi tải tăng lên. Dịch vụ chậm đi đồng nghĩa với mất bớt năng lực. Đến một ngưỡng nào đó, hệ thống chậm dần rồi sẽ ngừng phục vụ hẳn — tức là chậm vô hạn. SRE provision để đáp ứng một mục tiêu năng lực *ở một tốc độ phản hồi cụ thể*, và vì vậy quan tâm sâu sắc đến hiệu năng của dịch vụ. SRE và developer sản phẩm sẽ (và nên) giám sát và sửa đổi một dịch vụ để cải thiện hiệu năng, qua đó tăng thêm năng lực và cải thiện hiệu quả.<sup>[3](#fn3)</sup>

## Kết thúc của phần khởi đầu

Site Reliability Engineering là một bước rẽ đáng kể khỏi những thực hành tốt nhất hiện có trong ngành về quản lý các dịch vụ lớn, phức tạp. Khởi nguồn từ một cảm nhận rất đỗi quen thuộc — "là kỹ sư phần mềm, đây là cách tôi muốn đầu tư thời gian của mình để giải quyết một chuỗi tác vụ lặp đi lặp lại" — SRE nay đã lớn hơn thế rất nhiều: một bộ nguyên lý, một bộ thực hành, một bộ động lực khuyến khích, và một địa hạt riêng trong lòng ngành kỹ thuật phần mềm rộng lớn hơn. Phần còn lại của quyển sách sẽ khám phá chi tiết SRE Way (Đạo SRE).

<a id="fn1"></a>[1](#fn1) Vice President, Google Engineering, người sáng lập Google SRE

<a id="fn2"></a>[2](#fn2) Xem [Disaster Role Playing](https://sre.google/sre-book/accelerating-sre-on-call#xref_training_disaster-rpg).

<a id="fn3"></a>[3](#fn3) Để thảo luận thêm về cách sự hợp tác này có thể hoạt động trên thực tế, xem [Communications: Production Meetings](https://sre.google/sre-book/operational-overload#xref_comms-collab_production-meetings).

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
