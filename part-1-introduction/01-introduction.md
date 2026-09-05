# Chương 1. Giới thiệu (Introduction)

> **Nguyên bản:** [Chapter 1 - Introduction](https://sre.google/sre-book/introduction/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Benjamin Treynor Sloss<sup>[1](#fn1)</sup>
*Biên tập:* Betsy Beyer

> Hy vọng không phải là một chiến lược.
>
> Một câu nói quen thuộc trong SRE

Một sự thật mà ai cũng thừa nhận là các hệ thống không thể tự vận hành. Vậy thì, một hệ thống — đặc biệt là một hệ thống máy tính phức tạp vận hành ở quy mô lớn — *nên* được vận hành như thế nào?

## Cách tiếp cận của Sysadmin đối với quản lý dịch vụ

Trước nay, các công ty vẫn thuê quản trị viên hệ thống (system administrator, hay sysadmin) để vận hành những hệ thống máy tính phức tạp.

Theo cách tiếp cận sysadmin, người ta lắp ráp các thành phần phần mềm có sẵn rồi triển khai chúng phối hợp với nhau thành một dịch vụ. Sysadmin chịu trách nhiệm vận hành dịch vụ đó, xử lý các sự kiện (events) và các đợt cập nhật mỗi khi phát sinh. Hệ thống càng phức tạp, traffic càng lớn thì sự kiện và cập nhật càng nhiều, kéo theo đội sysadmin cũng phải phình to để kham hết khối lượng công việc tăng thêm. Vì vai trò sysadmin đòi hỏi bộ kỹ năng khác hẳn đội phát triển sản phẩm, developer và sysadmin được tách thành hai đội riêng: "development" (phát triển) và "operations" (vận hành, hay "ops").

Mô hình quản lý dịch vụ kiểu sysadmin có một số ưu điểm. Với các công ty đang cân nhắc cách vận hành và bố trí nhân sự cho một dịch vụ, cách tiếp cận này tương đối dễ triển khai: đây là mô hình quen thuộc trong ngành, nên có rất nhiều ví dụ để học hỏi và noi theo. Nguồn nhân sự phù hợp cũng không thiếu. Hàng loạt công cụ, thành phần phần mềm (mua sẵn hay tự làm), và các công ty tích hợp đều có sẵn để hỗ trợ vận hành những hệ thống đã lắp ráp, nên một đội sysadmin mới vào việc không cần phải phát minh lại bánh xe hay thiết kế hệ thống từ con số không.

Tuy nhiên, cách tiếp cận sysadmin, cùng với sự phân tách development/ops đi kèm, cũng bộc lộ không ít hạn chế và rủi ro. Nhìn chung, các nhược điểm này có thể chia thành hai nhóm: chi phí trực tiếp và chi phí gián tiếp.

Chi phí trực tiếp thì hiển hiện, chẳng có gì mơ hồ. Vận hành dịch vụ bằng một đội chuyên can thiệp thủ công — cả trong quản lý thay đổi (change management) lẫn xử lý sự kiện (event handling) — sẽ ngày càng tốn kém khi dịch vụ lớn lên hoặc traffic tăng, vì quy mô đội buộc phải tăng theo lượng tải mà hệ thống sinh ra.

Chi phí gián tiếp do sự tách biệt giữa development và ops có thể khó nhận thấy, nhưng thường gây thiệt hại lớn hơn cho tổ chức so với chi phí trực tiếp. Nguyên nhân nằm ở chỗ hai đội có nền tảng, kỹ năng và động lực hoàn toàn khác biệt. Họ dùng ngôn ngữ khác nhau để mô tả tình huống; mang những giả định khác nhau về rủi ro cũng như khả năng của các giải pháp kỹ thuật; và có kỳ vọng khác nhau về mức độ ổn định mục tiêu của sản phẩm. Sự chia rẽ giữa hai nhóm dễ lan từ động lực sang cách giao tiếp, mục tiêu, rồi cuối cùng là cả niềm tin lẫn sự tôn trọng dành cho nhau. Khi đạt đến mức đó, sự chia rẽ đã trở thành một dạng bệnh lý.

Các đội ops truyền thống và đối tác của họ trong phát triển sản phẩm vì thế thường rơi vào xung đột, rõ nhất là về việc phần mềm có thể được release (phát hành) lên production nhanh đến đâu. Với đội development, điều quan trọng nhất là ra mắt tính năng mới và thấy người dùng đón nhận. Với đội ops, điều quan trọng nhất lại là làm sao cho dịch vụ đừng hỏng trong ca mình cầm máy gọi trực (pager). Mà hầu hết các outage (mất dịch vụ) đều bắt nguồn từ một thay đổi nào đó — cấu hình mới, tính năng mới ra mắt, hay một kiểu traffic người dùng mới — nên mục tiêu của hai đội, xét đến cùng, đối chọi trực tiếp với nhau.

Cả hai nhóm đều hiểu rằng không thể nói thẳng thừng mong muốn của mình một cách trần trụi như vậy ("Chúng tôi muốn ra mắt bất cứ thứ gì, bất cứ lúc nào, không bị cản trở" đối lại "Chúng tôi không bao giờ muốn thay đổi bất cứ thứ gì một khi hệ thống đã chạy ổn"). Vì ngôn ngữ lẫn giả định về rủi ro khác nhau, hai bên thường sa vào cuộc giằng co dai dẳng quen thuộc, mỗi bên tìm cách đẩy lợi ích của mình lên. Nhóm ops cố bảo vệ hệ thống đang chạy trước rủi ro từ thay đổi bằng cách dựng lên các cổng kiểm soát ra mắt và thay đổi (launch and change gates). Ví dụ, một buổi xem xét ra mắt (launch review) có thể kèm theo danh mục rà soát *mọi* vấn đề từng gây sự cố ngừng dịch vụ trong quá khứ — một danh sách có thể dài vô hạn định, mà không phải mục nào cũng đáng giá như nhau. Nhóm dev nhanh chóng học được cách đối phó: bớt "launch" (ra mắt) đi, thay vào đó là nhiều "flag flip" (đổi cờ tính năng), "incremental update" (cập nhật từng phần), hay cherry pick hơn. Họ còn dùng những chiến thuật như sharding (chia nhỏ) sản phẩm, để càng ít tính năng phải qua launch review càng tốt.

## Cách tiếp cận của Google đối với quản lý dịch vụ: Site Reliability Engineering

Xung đột không phải là điều tất yếu khi cung cấp dịch vụ phần mềm. Google chọn cách vận hành khác: các [đội Site Reliability Engineering](https://sre.google/sre-book/software-engineering-in-sre/) của chúng tôi tập trung tuyển kỹ sư phần mềm vào vận hành sản phẩm, đồng thời [xây dựng những hệ thống đảm nhận phần việc](https://sre.google/sre-book/distributed-periodic-scheduling/) mà trước đây các sysadmin thường phải làm thủ công.

Site Reliability Engineering, theo cách Google định nghĩa, chính xác là gì? Cách giải thích của tôi rất đơn giản: SRE là kết quả khi giao cho một kỹ sư phần mềm thiết kế đội vận hành. Trước khi gia nhập Google năm 2003 và nhận nhiệm vụ dẫn dắt một "Production Team" gồm bảy kỹ sư, tôi thuần túy chỉ làm kỹ thuật phần mềm. Vì vậy, tôi thiết kế và quản lý nhóm theo đúng cách mà *tôi* mong muốn, nếu chính tôi là một SRE trong nhóm. Nhóm đó dần lớn thành đội SRE ngày nay của Google, và vẫn giữ đúng tinh thần mà một người cả đời làm phần mềm đã hình dung từ đầu.

Một nền tảng quan trọng trong cách Google quản lý dịch vụ là cơ cấu nhân sự của mỗi đội SRE. Nhìn chung, SRE có thể chia thành hai nhóm chính.

50–60% là Google Software Engineer, hay chính xác hơn, những người được tuyển dụng thông qua quy trình chuẩn cho Google Software Engineer. 40–50% còn lại là các ứng viên rất gần với tiêu chuẩn của Google Software Engineer (tức là đạt 85–99% bộ kỹ năng yêu cầu), và *ngoài ra* có một bộ kỹ năng kỹ thuật hữu ích cho SRE nhưng hiếm gặp ở phần lớn các kỹ sư phần mềm. Đại thể, kiến thức nội tại hệ thống UNIX (UNIX system internals) và mạng (networking, từ Layer 1 đến Layer 3) là hai loại kỹ năng kỹ thuật thay thế mà chúng tôi tìm kiếm nhiều nhất.

Điểm chung của mọi SRE là đều tin vào giá trị của việc phát triển hệ thống phần mềm để giải quyết các vấn đề phức tạp, và đều đủ năng lực thực hiện công việc đó. Trong SRE, chúng tôi theo dõi sát bước tiến nghề nghiệp của cả hai nhóm, và đến nay chưa thấy khác biệt thực tế nào về hiệu suất giữa kỹ sư từ hai nguồn tuyển. Thực tế, nền tảng tương đối đa dạng của đội SRE thường dẫn đến những hệ thống thông minh, chất lượng cao, rõ ràng là sản phẩm của sự kết hợp giữa nhiều bộ kỹ năng.

Cách tuyển dụng SRE của chúng tôi giúp xây dựng được một đội ngũ gồm những người (a) sẽ nhanh chóng chán ngán các tác vụ thủ công, và (b) đủ năng lực để viết phần mềm thay thế cho phần việc thủ công trước đó, ngay cả khi giải pháp không hề đơn giản. Các SRE cũng có nền tảng học thuật và trí tuệ tương đồng với phần còn lại của tổ chức phát triển. Về cơ bản, SRE đang đảm nhận công việc mà trước đây thuộc về đội ops, nhưng được thực hiện bởi các kỹ sư có chuyên môn phần mềm. Chúng tôi đặt cược rằng những kỹ sư này vừa có thiên hướng, vừa có khả năng thiết kế và [triển khai tự động hóa](https://sre.google/sre-book/automation-at-google/) bằng phần mềm để thay thế sức người.

Ngay từ khâu thiết kế, điều cốt yếu là các đội SRE phải tập trung vào công việc kỹ thuật (engineering). Nếu thiếu hoạt động kỹ thuật liên tục, tải vận hành sẽ tăng dần, buộc đội phải tuyển thêm người chỉ để theo kịp khối lượng công việc. Rốt cuộc, một nhóm thiên về ops truyền thống sẽ phình to tuyến tính theo quy mô dịch vụ: sản phẩm mà dịch vụ hỗ trợ càng thành công, tải vận hành càng tăng theo traffic, nghĩa là lại phải tuyển thêm người để làm đi làm lại cùng những tác vụ cũ.

Để tránh rơi vào cảnh đó, đội phụ trách một dịch vụ buộc phải viết code, nếu không sẽ bị nhấn chìm. Vì thế, Google đặt *mức trần 50% cho tổng khối lượng công việc "ops" của tất cả các SRE* — ticket, on-call (trực sự cố), tác vụ thủ công, v.v. Mức trần này đảm bảo đội SRE còn đủ quỹ thời gian để làm cho dịch vụ ổn định và dễ vận hành. Đây là giới hạn trên; nếu cứ để mọi việc diễn ra tự nhiên, theo thời gian đội SRE nên tiến dần về trạng thái gần như không còn tải vận hành, dành trọn thời gian cho công việc phát triển, vì khi đó dịch vụ về cơ bản đã tự chạy và tự sửa chữa: chúng tôi muốn những hệ thống *tự động* (automatic), chứ không chỉ *được tự động hóa* (automated). Còn trên thực tế, chuyện mở rộng quy mô và tính năng mới chẳng bao giờ để SRE được ngơi tay.

Theo nguyên tắc của Google, đội SRE phải dành 50% thời gian còn lại cho công việc phát triển. Chúng tôi thực thi ngưỡng này như thế nào? Trước tiên, cần đo lường xem thời gian của SRE đang được phân bổ ra sao. Khi đã có số liệu, đội nào liên tục dành chưa tới 50% thời gian cho phát triển sẽ phải điều chỉnh cách làm. Thường thì điều đó có nghĩa là chuyển bớt gánh nặng vận hành về cho đội phát triển, hoặc bổ sung nhân sự cho đội mà không giao thêm trách nhiệm vận hành. Chủ động giữ thế cân bằng giữa ops và phát triển giúp chúng tôi bảo đảm SRE có đủ thời gian cho công việc kỹ thuật sáng tạo, tự chủ, đồng thời vẫn giữ được vốn hiểu biết đúc kết từ chính việc vận hành dịch vụ.

Chúng tôi nhận thấy cách tiếp cận của Google SRE trong việc vận hành các hệ thống quy mô lớn có nhiều ưu điểm. Vì SRE trực tiếp sửa code nhằm mục tiêu giúp hệ thống của Google tự vận hành, các đội SRE vừa đổi mới nhanh, vừa cởi mở đón nhận thay đổi. Những đội như vậy cũng tương đối rẻ — cùng một dịch vụ đó mà giao cho một đội thiên về ops hỗ trợ thì sẽ cần nhiều người hơn đáng kể. Trong khi đó, số SRE cần để vận hành, bảo trì và cải thiện một hệ thống tăng chậm hơn tuyến tính (sublinear) so với quy mô hệ thống. Cuối cùng, SRE không chỉ tránh được thế rối loạn của sự phân tách dev/ops, mà cấu trúc này còn giúp chính các đội phát triển sản phẩm tốt lên: việc luân chuyển dễ dàng giữa đội phát triển sản phẩm và SRE tạo cơ hội đào tạo chéo cho cả hai phía, và nâng tay nghề cho những developer vốn khó có dịp nào khác để học cách xây dựng một hệ thống phân tán một triệu core.

Dù mang lại nhiều lợi ích, [mô hình SRE](https://sre.google/sre-book/engagement-model/) vẫn đối mặt với những thách thức riêng. Tại Google, việc tuyển dụng SRE là một bài toán thường trực: SRE không chỉ phải cạnh tranh cùng một nguồn ứng viên với kênh tuyển dụng phát triển sản phẩm, mà do chúng tôi đặt chuẩn tuyển rất cao ở cả kỹ năng coding lẫn [kỹ thuật hệ thống](https://sre.google/sre-book/release-engineering/), nên nguồn ứng viên đạt yêu cầu ắt hẳn rất hạn chế. Vì lĩnh vực này còn tương đối mới mẻ và khác biệt, trong ngành chưa có nhiều tài liệu về [cách xây dựng và quản lý một đội SRE](https://sre.google/sre-book/part-IV-management/) (hy vọng quyển sách này sẽ góp một bước theo hướng đó!). Và một khi đội SRE đã thành hình, cách quản lý dịch vụ có phần khác lối thường của họ sẽ cần sự hậu thuẫn mạnh mẽ từ cấp quản lý. Ví dụ, quyết định ngừng release cho hết quý một khi ngân sách lỗi (error budget) đã cạn sẽ khó được đội phát triển sản phẩm chấp nhận, trừ khi có chỉ thị từ chính cấp quản lý của họ.

## DevOps hay SRE?

Thuật ngữ "DevOps" xuất hiện trong ngành vào cuối năm 2008 và cho đến thời điểm viết bài này (đầu năm 2016) vẫn đang trong trạng thái biến động. Các nguyên lý cốt lõi của nó — sự tham gia của chức năng IT trong mỗi giai đoạn thiết kế và phát triển hệ thống, sự phụ thuộc lớn vào tự động hóa thay vì nỗ lực con người, việc áp dụng các thực hành và công cụ kỹ thuật vào các tác vụ vận hành — nhất quán với nhiều [nguyên lý và thực hành của SRE](https://sre.google/sre-book/part-II-principles/). Có thể coi DevOps như một sự khái quát hóa của một số nguyên lý cốt lõi của SRE cho một phạm vi rộng hơn các tổ chức, cấu trúc quản lý và nhân sự. Ngược lại, cũng có thể coi SRE là một triển khai cụ thể của DevOps, kèm theo một số mở rộng đặc thù.

## Các nguyên tắc của SRE

Mỗi đội SRE có cách tổ chức luồng công việc, thứ tự ưu tiên và hoạt động hàng ngày khác nhau, nhưng tất cả đều chung một bộ trách nhiệm cơ bản đối với dịch vụ mình hỗ trợ và cùng tuân theo những nguyên tắc cốt lõi giống nhau. Nhìn chung, một đội SRE chịu trách nhiệm về *khả dụng (availability), độ trễ (latency), hiệu năng (performance), hiệu quả (efficiency), quản lý thay đổi (change management), giám sát (monitoring), phản ứng khẩn cấp (emergency response), và lập kế hoạch năng lực (capacity planning)* cho dịch vụ của họ. Chúng tôi đã đúc kết thành các quy tắc và nguyên lý rõ ràng về cách các đội SRE tương tác với môi trường xung quanh — không chỉ [môi trường production](https://sre.google/sre-book/production-environment/), mà còn với các đội phát triển sản phẩm, đội kiểm thử, người dùng, v.v. Những quy tắc và thực hành làm việc này giúp chúng tôi duy trì sự tập trung vào công việc kỹ thuật, thay vì công việc vận hành.

Phần tiếp theo thảo luận từng nguyên tắc cốt lõi của Google SRE.

## Đảm bảo sự tập trung bền bỉ vào kỹ thuật

Như đã thảo luận, Google giới hạn khối lượng công việc vận hành của SRE ở mức 50% thời gian. Phần thời gian còn lại, họ nên dành cho các dự án phát triển phần mềm. Trên thực tế, điều này được thực hiện bằng cách theo dõi khối lượng công việc vận hành mà SRE đang đảm nhận, sau đó chuyển hướng [công việc vận hành thừa](https://sre.google/sre-book/dealing-with-interrupts/) sang các đội phát triển sản phẩm: trả lại các bug và ticket cho quản lý phát triển, [tái] đưa các developer vào vòng trực pager on-call, v.v. Quá trình chuyển giao này sẽ dừng lại khi tải vận hành giảm xuống mức 50% hoặc thấp hơn. Đồng thời, đây cũng là một cơ chế phản hồi hiệu quả, thúc đẩy các developer xây dựng hệ thống không cần can thiệp thủ công. Cách tiếp cận này phát huy hiệu quả khi toàn bộ tổ chức — bao gồm cả SRE và đội phát triển — đều hiểu rõ lý do tồn tại của cơ chế van an toàn này, và cùng hướng tới mục tiêu không để xảy ra sự kiện tràn (overflow events) — tức là sản phẩm không tạo ra lượng tải vận hành lớn đến mức phải kích hoạt van an toàn.

Trong quá trình vận hành, mỗi ca trực on-call kéo dài trung bình 8–12 giờ, một SRE chỉ nên nhận tối đa hai sự kiện. Mức khối lượng mục tiêu này giúp kỹ sư on-call có đủ thời gian xử lý sự kiện nhanh chóng và chính xác, dọn dẹp, khôi phục dịch vụ về trạng thái bình thường, rồi viết bản postmortem (báo cáo sau sự cố). Nếu thường xuyên có nhiều hơn hai sự kiện mỗi ca trực, các vấn đề sẽ không được điều tra kỹ lưỡng và kỹ sư bị quá tải đến mức không thể rút ra bài học từ những sự kiện này. Tình trạng kiệt sức do máy gọi trực (pager fatigue) cũng không hề giảm khi hệ thống scale lên. Ngược lại, nếu SRE on-call liên tục nhận chưa tới một sự kiện mỗi ca, việc bắt họ ngồi trực chỉ là lãng phí thời gian.

Mọi incident (sự cố) quan trọng đều cần có postmortem, bất kể có kích hoạt pager hay không; những postmortem không kèm theo pager thậm chí còn giá trị hơn, vì chúng nhiều khả năng sẽ phơi bày các lỗ hổng giám sát rõ ràng. Cuộc điều tra phải dựng lại chi tiết diễn biến sự việc, tìm ra mọi nguyên nhân gốc rễ, và phân công cụ thể việc khắc phục hoặc cải thiện khả năng ứng phó cho lần sau. Google vận hành theo *văn hóa postmortem không đổ lỗi* (blame-free postmortem culture), với mục tiêu là phơi bày sai sót và dùng kỹ thuật để sửa chữa, thay vì né tránh hay xem nhẹ chúng.

## Theo đuổi tốc độ thay đổi tối đa mà không vi phạm SLO của dịch vụ

Các đội phát triển sản phẩm và SRE có thể xây dựng [mối quan hệ làm việc hiệu quả](https://sre.google/sre-book/engagement-model/) bằng cách loại bỏ mâu thuẫn cấu trúc trong các mục tiêu riêng. Mâu thuẫn cấu trúc ở đây nằm giữa nhịp độ đổi mới và độ ổn định của sản phẩm; như đã mô tả ở trên, mâu thuẫn này thường chỉ bộc lộ một cách gián tiếp. Trong SRE, chúng tôi đưa mâu thuẫn đó ra ánh sáng, rồi giải quyết bằng khái niệm *ngân sách lỗi (error budget)*.

Error budget bắt nguồn từ quan sát rằng *100% là mục tiêu độ tin cậy sai cho hầu như mọi thứ* (máy tạo nhịp tim và hệ thống chống bó cứng phanh là những ngoại lệ đáng chú ý). Nhìn chung, đối với bất kỳ dịch vụ hay hệ thống phần mềm nào, 100% không phải là mục tiêu độ tin cậy đúng, bởi chẳng người dùng nào phân biệt nổi một hệ thống khả dụng 100% với một hệ thống khả dụng 99.999%. Giữa người dùng và dịch vụ còn vô số hệ thống khác nằm trên đường truyền (laptop của họ, WiFi nhà họ, ISP, lưới điện…) và những hệ thống đó gộp lại có khả dụng thấp hơn rất nhiều so với 99.999%. Vì vậy, chút khác biệt giữa 99.999% và 100% chìm lẫn trong nhiễu từ những nguồn bất khả dụng khác, và người dùng chẳng hưởng lợi gì từ nỗ lực khổng lồ bỏ ra chỉ để thêm 0.001% khả dụng đó.

Nếu 100% không phải là mục tiêu độ tin cậy phù hợp cho một hệ thống, vậy mục tiêu đúng cho hệ thống đó là bao nhiêu? Thực chất, đây là câu hỏi về sản phẩm chứ không phải kỹ thuật, đòi hỏi phải cân nhắc các yếu tố sau:

-   Xét theo cách người dùng sử dụng sản phẩm, họ sẽ hài lòng với mức khả dụng nào?
-   Nếu không hài lòng với mức khả dụng của sản phẩm, người dùng có những lựa chọn thay thế nào?
-   Ở từng mức khả dụng khác nhau, cách người dùng sử dụng sản phẩm thay đổi ra sao?

Bên business hay bên sản phẩm phải là người xác lập mục tiêu khả dụng của hệ thống. Có mục tiêu rồi, error budget chính là một trừ đi mục tiêu khả dụng. Một dịch vụ khả dụng 99.99% nghĩa là được phép không khả dụng 0.01%. Phần 0.01% được phép đó chính là *error budget* của dịch vụ. Chúng ta có thể tiêu ngân sách này vào bất cứ thứ gì mình muốn, miễn đừng tiêu lố.

Vậy chúng ta muốn tiêu error budget như thế nào? Đội phát triển muốn ra mắt tính năng và thu hút người dùng mới. Lý tưởng nhất, chúng ta sẽ tiêu hết error budget bằng cách chấp nhận rủi ro khi ra mắt, đổi lấy tốc độ ra mắt nhanh hơn. Toàn bộ mô hình error budget gói gọn trong tiền đề cơ bản đó. Một khi hoạt động của SRE được nhìn qua lăng kính này, thì việc tiết kiệm error budget nhờ những chiến thuật như rollout phân giai đoạn (phased rollouts) hay thí nghiệm 1% đều có thể dùng để ra mắt nhanh hơn.

Việc áp dụng error budget giúp tháo gỡ mâu thuẫn về động lực vốn tồn tại giữa đội phát triển và SRE. Mục tiêu của SRE không còn là “zero outages” (không có sự cố); thay vào đó, SRE và developer sản phẩm cùng hướng đến việc tiêu error budget nhằm đạt tốc độ tính năng (feature velocity) tối đa. Sự chuyển dịch này tạo ra khác biệt to lớn. Outage không còn là điều “xấu” — nó trở thành phần được dự liệu trước trong quá trình đổi mới, là thứ mà đội phát triển và SRE cùng nhau quản lý thay vì nơm nớp lo sợ.

## Giám sát (Monitoring)

Giám sát (monitoring) là một trong những phương tiện chính để chủ sở hữu dịch vụ theo dõi sức khỏe và khả dụng của hệ thống. Do đó, chiến lược giám sát cần được xây dựng một cách kỹ lưỡng, có chủ đích. Một cách tiếp cận giám sát cổ điển và phổ biến là quan sát một giá trị hoặc điều kiện cụ thể, rồi kích hoạt một cảnh báo email khi giá trị đó vượt quá hoặc điều kiện đó xảy ra. Tuy nhiên, kiểu cảnh báo email này không phải giải pháp hiệu quả: một hệ thống mà phải có người đọc email rồi tự phán đoán xem có cần làm gì hay không thì ngay từ gốc đã là thiếu sót. Giám sát không bao giờ nên bắt con người phải diễn giải bất kỳ phần nào của miền cảnh báo. Việc diễn giải hãy để phần mềm lo; con người chỉ nên được gọi đến khi cần họ hành động.

Có ba loại đầu ra giám sát hợp lệ:

#### Cảnh báo (Alerts)

Báo hiệu rằng con người cần hành động ngay lập tức trước điều đang xảy ra hoặc sắp xảy ra, để cải thiện tình hình.

#### Ticket (Yêu cầu xử lý)

Đây là tín hiệu cho thấy cần có người vào cuộc, nhưng chưa phải ngay lập tức. Hệ thống không thể tự xử lý, song chỉ cần con người can thiệp trong vài ngày tới là sẽ không gây thiệt hại gì.

#### Ghi log (Logging)

Không ai cần xem thông tin này; nó được ghi lại để phục vụ chẩn đoán hoặc điều tra pháp y kỹ thuật (forensic). Mặc định là chẳng ai đọc log, trừ khi có sự cố buộc họ phải làm vậy.

## Phản ứng khẩn cấp (Emergency Response)

Độ tin cậy là hàm của thời gian trung bình đến khi hỏng (mean time to failure, MTTF) và thời gian trung bình để sửa chữa (mean time to repair, MTTR) [[Sch15]](https://sre.google/sre-book/bibliography#Sch15). Khi đánh giá hiệu quả phản ứng khẩn cấp, chỉ số sát sườn nhất là tốc độ đội phản ứng đưa hệ thống trở lại trạng thái khỏe mạnh — tức MTTR.

Con người là yếu tố làm tăng độ trễ. Một hệ thống có thể gặp nhiều hỏng hóc *thực tế* hơn nhưng vẫn đạt khả dụng cao hơn so với hệ thống luôn cần con người can thiệp, miễn là nó tránh được các tình huống khẩn cấp đòi hỏi sự can thiệp đó. Khi bắt buộc phải có sự tham gia của con người, chúng tôi nhận thấy việc chuẩn bị trước và ghi lại các thực hành tốt nhất vào một "playbook" (sách hướng dẫn) giúp cải thiện MTTR khoảng 3 lần so với cách ứng biến tại chỗ. Kỹ sư on-call kiểu "nghề gì cũng biết" (jack-of-all-trades) vẫn hữu dụng, nhưng hiệu quả sẽ cao hơn nhiều nếu kỹ sư được đào tạo bài bản và có sẵn playbook. Dĩ nhiên, không playbook nào, dù toàn diện đến đâu, có thể thay thế được những kỹ sư thông minh biết cách ứng biến tại chỗ; tuy nhiên, các [bước và mẹo troubleshooting (xử lý sự cố)](https://sre.google/sre-book/effective-troubleshooting/) được trình bày rõ ràng, kỹ lưỡng vẫn rất giá trị khi phải xử lý một lần gọi trực (page) rủi ro cao hoặc gấp gáp về thời gian. Do đó, Google SRE dựa vào các playbook on-call, cùng với các bài tập như "Wheel of Misfortune" (Bánh xe Xui xẻo)<sup>[2](#fn2)</sup>, để chuẩn bị cho kỹ sư phản ứng với các sự kiện on-call.

## Quản lý thay đổi (Change Management)

SRE nhận thấy khoảng 70% outage bắt nguồn từ thay đổi trên hệ thống đang chạy. Các thực hành tốt nhất trong lĩnh vực này sử dụng tự động hóa để đạt được:

-   Triển khai rollout tăng dần (progressive rollouts)
-   Phát hiện vấn đề nhanh và chính xác
-   Rollback (hoàn tác) thay đổi một cách an toàn khi vấn đề nảy sinh

Bộ ba thực hành này giúp giảm tối đa số người dùng và thao tác phải hứng chịu các thay đổi lỗi. Việc loại con người ra khỏi vòng xử lý cũng giúp tránh những vấn đề cố hữu của các tác vụ lặp đi lặp lại nhiều: mệt mỏi, quen tay sinh chủ quan, và lơ đễnh. Kết quả là cả tốc độ release lẫn độ an toàn đều tăng.

## Dự báo nhu cầu và lập kế hoạch năng lực (Demand Forecasting and Capacity Planning)

Nói cách khác, dự báo nhu cầu và lập kế hoạch năng lực là việc đảm bảo đủ năng lực cùng mức dự phòng (redundancy) để đáp ứng nhu cầu tương lai theo dự báo, ở mức khả dụng yêu cầu. Bản thân các khái niệm này chẳng có gì đặc biệt — điều đáng ngạc nhiên là rất nhiều dịch vụ và đội nhóm lại không thực hiện những bước cần thiết để chắc rằng năng lực cần có sẽ sẵn sàng đúng lúc. Khi lập kế hoạch năng lực, cần tính đến cả tăng trưởng hữu cơ (organic growth, bắt nguồn từ việc khách hàng chấp nhận và sử dụng sản phẩm một cách tự nhiên) lẫn tăng trưởng vô cơ (inorganic growth, kết quả từ các sự kiện như ra mắt tính năng, chiến dịch marketing, hay các thay đổi do business thúc đẩy khác).

Một số bước là bắt buộc trong lập kế hoạch năng lực:

-   Dự báo chính xác nhu cầu hữu cơ, với tầm nhìn xa hơn khoảng thời gian cần để bổ sung năng lực
-   Gộp chính xác các nguồn nhu cầu vô cơ vào bản dự báo nhu cầu
-   Chạy load test (kiểm thử tải) hệ thống thường xuyên để đối chiếu năng lực thô (server, disk, v.v.) với năng lực dịch vụ

Vì năng lực là yếu tố then chốt quyết định khả dụng, đội SRE phải chịu trách nhiệm lập kế hoạch năng lực, đồng thời đảm nhận cả việc provision (cấp phát tài nguyên).

## Cấp phát tài nguyên (Provisioning)

Provisioning bao hàm cả quản lý thay đổi lẫn lập kế hoạch năng lực. Theo kinh nghiệm của chúng tôi, cần thực hiện provisioning nhanh chóng và chỉ khi thực sự cần, bởi năng lực có chi phí cao. Tuy nhiên, thao tác này phải được thực hiện chính xác; nếu không, phần năng lực bổ sung sẽ không hoạt động khi cần. Việc thêm năng lực mới thường bao gồm khởi động một instance (thực thể) hoặc địa điểm mới, sửa đổi đáng kể các hệ thống hiện có (tệp cấu hình, load balancer, mạng), và xác minh rằng năng lực mới hoạt động và cho kết quả đúng. Do đó, thao tác này rủi ro hơn load shifting (chuyển tải, vốn diễn ra nhiều lần mỗi giờ) và đòi hỏi mức độ cẩn trọng cao tương xứng.

## Hiệu quả và hiệu năng (Efficiency and Performance)

Hễ dịch vụ còn phải tính đến tiền bạc thì việc dùng tài nguyên hiệu quả vẫn quan trọng. Vì SRE là người nắm quyền quyết định cuối cùng về provisioning, họ cũng phải tham gia mọi công việc liên quan đến mức sử dụng (utilization), bởi mức sử dụng là hàm của cách dịch vụ hoạt động và cách nó được provision. Nói cách khác, chăm chút cho chiến lược provision của một dịch vụ — và qua đó là mức sử dụng của nó — chính là một đòn bẩy rất, rất lớn đối với tổng chi phí dịch vụ.

Việc sử dụng tài nguyên phụ thuộc vào nhu cầu (tải), năng lực và hiệu quả phần mềm. SRE dự báo nhu cầu, provision năng lực và có thể điều chỉnh phần mềm. Ba yếu tố này chiếm phần lớn (dù không phải toàn bộ) hiệu quả dịch vụ.

Hệ thống phần mềm nào cũng chậm dần khi tải tăng lên. Dịch vụ chậm đi đồng nghĩa với mất bớt năng lực. Đến một ngưỡng nào đó, hệ thống chậm dần rồi sẽ ngừng phục vụ hẳn — tức là chậm vô hạn. SRE provision để đáp ứng một mục tiêu năng lực *ở một tốc độ phản hồi cụ thể*, và vì vậy quan tâm sâu sắc đến hiệu năng của dịch vụ. SRE và developer sản phẩm sẽ (và nên) giám sát và sửa đổi một dịch vụ để cải thiện hiệu năng, qua đó tăng thêm năng lực và cải thiện hiệu quả.<sup>[3](#fn3)</sup>

## Kết thúc của phần khởi đầu

Site Reliability Engineering đánh dấu một bước rẽ đáng kể so với các thực hành tốt nhất hiện hành trong ngành về quản lý dịch vụ lớn, phức tạp. Khởi nguồn từ một cảm nhận rất đỗi quen thuộc — “là kỹ sư phần mềm, đây là cách tôi muốn đầu tư thời gian của mình để giải quyết một chuỗi tác vụ lặp đi lặp lại” — SRE nay đã lớn hơn thế rất nhiều: một bộ nguyên lý, một bộ thực hành, một bộ động lực khuyến khích, và một địa hạt riêng trong lòng ngành kỹ thuật phần mềm rộng lớn hơn. Phần còn lại của quyển sách sẽ khám phá chi tiết SRE Way (Đạo SRE).

<a id="fn1"></a>[1](#fn1) Vice President, Google Engineering, người sáng lập Google SRE

<a id="fn2"></a>[2](#fn2) Xem [Disaster Role Playing](https://sre.google/sre-book/accelerating-sre-on-call#xref_training_disaster-rpg).

<a id="fn3"></a>[3](#fn3) Để thảo luận thêm về cách sự hợp tác này có thể hoạt động trên thực tế, xem [Communications: Production Meetings](https://sre.google/sre-book/operational-overload#xref_comms-collab_production-meetings).

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
