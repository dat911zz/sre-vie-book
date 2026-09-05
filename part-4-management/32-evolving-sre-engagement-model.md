# Chương 32. Mô hình Tham gia SRE đang Tiến hóa (The Evolving SRE Engagement Model)

> **Nguyên bản:** [Chapter 32 - The Evolving SRE Engagement Model](https://sre.google/sre-book/evolving-sre-engagement-model/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Acacio Cruz và Ashish Bhambhani
*Biên tập:* Betsy Beyer và Tim Harvey

## Tham gia SRE: Cái gì, Như thế nào, và Tại sao (SRE Engagement: What, How, and Why)

Phần lớn cuốn sách này đã bàn về việc SRE phụ trách một dịch vụ như thế nào. Vì ít dịch vụ nào ngay từ đầu vòng đời đã có SRE hỗ trợ, nên cần một quy trình để đánh giá dịch vụ, xác nhận nó đủ điều kiện nhận hỗ trợ SRE, thương lượng cách khắc phục các khiếm khuyết đang cản trở việc hỗ trợ, và thiết lập hỗ trợ SRE. Chúng tôi gọi quy trình này là *onboarding* (đưa vào). Nếu bạn làm việc trong môi trường có nhiều dịch vụ hiện tại ở các mức độ hoàn thiện khác nhau, đội SRE của bạn có thể sẽ phải xử lý hàng đợi onboarding được ưu tiên trong một thời gian khá dài, cho đến khi hoàn tất việc tiếp nhận các mục tiêu có giá trị cao nhất.

Mặc dù cách làm này rất phổ biến và hoàn toàn hợp lý trong một môi trường *đã thành sự thực* (fait accompli), nhưng thực ra có ít nhất hai cách tốt hơn để mang sự khôn ngoan của production, cùng sự hỗ trợ từ SRE, đến cả các dịch vụ cũ lẫn mới.

Trong trường hợp đầu tiên, giống như trong kỹ thuật phần mềm — nơi phát hiện bug càng sớm thì chi phí sửa càng thấp — cuộc tham vấn của đội SRE diễn ra càng sớm thì dịch vụ càng ổn định và nhanh chóng hưởng lợi. Khi SRE tham gia ngay từ giai đoạn *thiết kế* sớm nhất, thời gian onboarding được rút ngắn và dịch vụ đáng tin cậy hơn “ngay từ cửa xuất phát”, thường là vì chúng tôi không phải mất thời gian tháo gỡ một thiết kế hoặc triển khai kém tối ưu.

Một cách khác, có lẽ là cách tốt nhất, là bỏ qua quy trình mà theo đó các hệ thống được tạo ra với nhiều biến thể cá nhân rồi cuối cùng mới “đến” trước cửa SRE. Thay vào đó, hãy cung cấp cho đội phát triển sản phẩm một *nền tảng* (platform) hạ tầng đã được SRE xác thực, để họ có thể xây dựng hệ thống của mình trên đó. Nền tảng này mang lại lợi ích kép: vừa đáng tin cậy, vừa có khả năng mở rộng (scalable). Cách làm này loại bỏ hoàn toàn một số lớp vấn đề về tải nhận thức; đồng thời, nhờ giải quyết các thực hành hạ tầng chung, nó cho phép các đội phát triển sản phẩm tập trung vào đổi mới ở tầng ứng dụng – nơi mà công việc này chủ yếu thuộc về họ.

Trong các mục tiếp theo, chúng tôi sẽ lần lượt xem xét từng mô hình, bắt đầu với mô hình “kinh điển” do PRR dẫn dắt.

## Mô hình PRR (The PRR Model)

Bước khởi đầu điển hình nhất của [sự tham gia SRE](https://sre.google/sre-book/communication-and-collaboration/) là Production Readiness Review (PRR — Đánh giá Sẵn sàng Production), quy trình xác định các nhu cầu độ tin cậy của dịch vụ dựa trên chi tiết cụ thể của nó. Thông qua PRR, các SRE vận dụng những gì đã học và kinh nghiệm thực tế để đảm bảo độ tin cậy cho dịch vụ khi chạy trong production. PRR được xem là điều kiện tiên quyết để đội SRE nhận trách nhiệm [quản lý các khía cạnh production của dịch vụ.](https://sre.google/sre-book/service-best-practices/)

[Hình 32-1](#hinh-32-1) minh họa vòng đời của một dịch vụ điển hình. Production Readiness Review có thể bắt đầu ở bất kỳ điểm nào trong vòng đời này, nhưng phạm vi các giai đoạn có sự tham gia của SRE đã mở rộng theo thời gian. Chương này mô tả Mô hình PRR Đơn giản, sau đó thảo luận về cách biến đổi nó thành Mô hình Tham gia Mở rộng và cách cấu trúc Frameworks và Nền tảng SRE giúp SRE mở rộng quy trình tham gia cũng như tác động của họ.

<a id="hinh-32-1"></a>        ![A typical service lifecycle.](../assets/imgs/fig-32-1.jpg)

**Hình 32-1.** Một vòng đời dịch vụ điển hình

## Mô hình Tham gia SRE (The SRE Engagement Model)

SRE tìm kiếm trách nhiệm production cho các dịch vụ quan trọng mà nó có thể đưa ra các đóng góp cụ thể cho độ tin cậy. SRE quan tâm đến một số khía cạnh của một dịch vụ, được gọi tập thể là *production*. Những khía cạnh này bao gồm:

-   Kiến trúc hệ thống và các phụ thuộc giữa các dịch vụ
-   Đo lường (instrumentation), metrics (chỉ số), và giám sát (monitoring)
-   Phản ứng khẩn cấp
-   Lập kế hoạch năng lực (capacity planning)
-   Quản lý thay đổi (change management)
-   Hiệu năng: khả dụng (availability), độ trễ (latency), và hiệu quả (efficiency)

Khi các SRE tham gia vào một dịch vụ, chúng tôi hướng đến việc cải thiện nó trên tất cả các trục này, qua đó giúp [việc quản lý production cho dịch vụ](https://sre.google/sre-book/production-environment/) dễ dàng hơn.

## Hỗ trợ Thay thế (Alternative Support)

Không phải tất cả các dịch vụ Google nhận được sự tham gia SRE chặt chẽ. Một số yếu tố đang diễn ra ở đây:

-   Nhiều dịch vụ không cần độ tin cậy và khả dụng cao, nên hỗ trợ có thể được cung cấp bằng các phương tiện khác.
-   Theo thiết kế, số lượng đội phát triển cần hỗ trợ SRE sẽ vượt quá năng lực xử lý của các đội SRE (xem [Giới thiệu](https://sre.google/sre-book/introduction/)).

Khi SRE không thể cung cấp hỗ trợ đầy đủ, nó cung cấp các lựa chọn khác để cải thiện production, chẳng hạn như tài liệu và tham vấn.

### Tài liệu (Documentation)

Các hướng dẫn phát triển có sẵn cho các công nghệ nội bộ và các client (khách hàng) của các hệ thống được sử dụng rộng rãi. Production Guide (Hướng dẫn Production) của Google tài liệu hóa [các thực hành tốt nhất production cho các dịch vụ](https://sre.google/sre-book/service-best-practices/), dựa trên kinh nghiệm của cả các đội SRE và phát triển. Các nhà phát triển có thể áp dụng các giải pháp và khuyến nghị trong các tài liệu như vậy để cải thiện các dịch vụ của họ.

### Tham vấn (Consultation)

Các nhà phát triển cũng có thể tìm kiếm tham vấn SRE để thảo luận về các dịch vụ hoặc lĩnh vực vấn đề cụ thể. Đội Launch Coordination Engineering (LCE — Kỹ thuật Phối hợp Ra mắt) (xem [Các Ra mắt Sản phẩm Đáng tin cậy ở Quy mô](https://sre.google/sre-book/reliable-product-launches/)) dành phần lớn thời gian của mình để tham vấn với các đội phát triển. Các đội SRE không đặc biệt dành riêng cho tham vấn ra mắt cũng tham gia tham vấn với các đội phát triển.

Khi triển khai dịch vụ hoặc tính năng mới, các nhà phát triển thường trao đổi với SRE để xin tư vấn chuẩn bị cho giai đoạn Launch (Ra mắt). Quá trình tư vấn này thường do một hoặc hai SRE thực hiện, dành vài giờ để xem xét thiết kế và phương án triển khai ở cấp cao. Sau đó, các SRE này làm việc cùng đội phát triển để chỉ ra những rủi ro cần lưu ý và thảo luận về các mẫu hoặc giải pháp phổ biến có thể áp dụng nhằm cải thiện dịch vụ trong production. Một số khuyến nghị có thể được trích dẫn từ Production Guide đã đề cập ở trên.

Các phiên tham vấn cần có phạm vi rộng, bởi lẽ trong khoảng thời gian hạn chế, không thể nào nắm bắt sâu sắc một hệ thống cụ thể. Với một số đội phát triển, chỉ tham vấn là chưa đủ:

-   Các dịch vụ đã tăng lên nhiều bậc độ lớn kể từ khi chúng ra mắt, giờ đòi hỏi nhiều thời gian hơn để hiểu hơn là khả thi thông qua tài liệu và tham vấn.
-   Các dịch vụ mà nhiều dịch vụ khác sau đó đã dựa vào, giờ phục vụ đáng kể nhiều traffic (lưu lượng) hơn từ nhiều client khác nhau.

Những dịch vụ này có thể đã phát triển đến mức bắt đầu gặp khó khăn đáng kể trong production, đồng thời trở nên quan trọng đối với người dùng. Trong những trường hợp như vậy, cần có sự tham gia dài hạn của SRE để đảm bảo chúng được duy trì đúng đắn trong production khi tiếp tục tăng trưởng.

## Production Readiness Reviews: Mô hình PRR Đơn giản (Production Readiness Reviews: Simple PRR Model)

Khi một đội phát triển đề nghị SRE tiếp quản production của dịch vụ, SRE sẽ đánh giá cả tầm quan trọng của dịch vụ lẫn năng lực sẵn sàng của các đội SRE. Nếu dịch vụ đủ điều kiện nhận hỗ trợ SRE, và cả hai bên (đội SRE cùng tổ chức phát triển) thống nhất về mức độ nhân sự cần thiết để triển khai hỗ trợ, SRE sẽ khởi động Production Readiness Review với đội phát triển.

Các mục tiêu của Production Readiness Review như sau:

-   Xác minh rằng dịch vụ đáp ứng các tiêu chuẩn chấp nhận được về thiết lập production và sẵn sàng vận hành, đồng thời các chủ sở hữu dịch vụ sẵn sàng phối hợp với SRE và tận dụng chuyên môn của họ.
-   Nâng cao độ tin cậy của dịch vụ trong môi trường production, đồng thời giảm thiểu số lượng và mức độ nghiêm trọng của các incident (sự cố) có thể xảy ra. Một PRR bao trùm mọi khía cạnh production mà SRE quan tâm.

Sau khi đủ các cải tiến được thực hiện và dịch vụ được coi là sẵn sàng cho hỗ trợ SRE, một đội SRE tiếp nhận các trách nhiệm production của nó.

Điều này dẫn chúng ta đến chính quy trình Production Readiness Review. Ba mô hình liên quan nhưng khác nhau — Mô hình PRR Đơn giản, Mô hình Tham gia Sớm và Frameworks và Nền tảng SRE — sẽ được thảo luận lần lượt.

Trước tiên, chúng tôi sẽ mô tả Mô hình PRR Đơn giản, thường áp dụng cho một dịch vụ đã ra mắt và được bàn giao cho đội SRE. PRR trải qua một số giai đoạn, khá giống vòng đời phát triển, dù có thể chạy song song và độc lập với vòng đời đó.

## Tham gia (Engagement)

Lãnh đạo SRE trước tiên xác định đội SRE nào phù hợp để tiếp nhận dịch vụ. Thông thường, một đến ba SRE được chọn hoặc tự đề cử để thực hiện quy trình PRR. Nhóm nhỏ này sau đó bắt đầu thảo luận với đội phát triển. Nội dung thảo luận bao gồm các vấn đề như:

-   Thiết lập một SLO/SLA (mục tiêu/thỏa thuận mức dịch vụ) cho dịch vụ
-   Lập kế hoạch cho các thay đổi thiết kế có thể gây gián đoạn được yêu cầu để cải thiện độ tin cậy
-   Các lịch trình lập kế hoạch và đào tạo

Mục tiêu là đạt được sự đồng thuận chung về quy trình, các mục tiêu cuối cùng và những kết quả cần thiết để đội SRE phối hợp với đội phát triển cùng dịch vụ của họ.

## Phân tích (Analysis)

Phân tích là phân đoạn công việc lớn đầu tiên. Trong giai đoạn này, các SRE phụ trách review tìm hiểu về dịch vụ và bắt đầu phân tích để xác định các khiếm khuyết production. Mục tiêu là đo lường mức độ trưởng thành của dịch vụ trên các trục quan tâm khác nhau của SRE. Họ cũng xem xét thiết kế và triển khai của dịch vụ để kiểm tra mức độ tuân thủ các thực hành tốt nhất production. Thông thường, đội SRE thiết lập và duy trì một checklist PRR rõ ràng riêng cho giai đoạn Phân tích. Checklist này cụ thể cho từng dịch vụ, thường dựa trên chuyên môn lĩnh vực, kinh nghiệm với các hệ thống liên quan hoặc tương tự, và các thực hành tốt nhất từ Production Guide. Đội SRE cũng có thể tham vấn các đội khác có nhiều kinh nghiệm hơn với một số thành phần hoặc phụ thuộc nhất định của dịch vụ.

Một vài ví dụ về các mục checklist bao gồm:

-   Các cập nhật cho dịch vụ có ảnh hưởng đến một tỷ lệ phần trăm lớn một cách không hợp lý của hệ thống cùng một lúc không?
-   Dịch vụ có kết nối đến instance (bản chạy) phục vụ phù hợp của các phụ thuộc của nó không? Ví dụ, các yêu cầu của người dùng cuối đến một dịch vụ không nên phụ thuộc vào một hệ thống được thiết kế cho một use case (trường hợp sử dụng) xử lý batch (hàng loạt).
-   Dịch vụ có yêu cầu một chất lượng dịch vụ mạng đủ cao khi nói chuyện với một dịch vụ từ xa quan trọng không?
- Dịch vụ có gửi các lỗi lên hệ thống log (nhật ký) tập trung để phân tích không? Nó có ghi nhận mọi điều kiện ngoại lệ dẫn đến phản hồi suy giảm hoặc thất bại cho người dùng cuối không?
-   Tất cả các thất bại yêu cầu nhìn thấy được bởi người dùng có được đo lường và giám sát tốt, với các cảnh báo được cấu hình phù hợp không?

Checklist cũng có thể bao gồm các tiêu chuẩn và thực hành tốt nhất vận hành mà một đội SRE cụ thể tuân theo. Ví dụ, một cấu hình dịch vụ hoạt động hoàn hảo nhưng không tuân theo “tiêu chuẩn vàng” (gold standard) của đội SRE có thể được refactor để tương thích tốt hơn với các công cụ SRE, từ đó hỗ trợ quản lý các cấu hình có khả năng mở rộng. Các SRE cũng xem xét các incident và postmortem gần đây của dịch vụ, cùng với các task theo dõi cho các incident. Đánh giá này đo lường mức độ đòi hỏi của phản ứng khẩn cấp đối với dịch vụ và sự sẵn có của các biện pháp kiểm soát vận hành đã được thiết lập tốt.

## Cải tiến và Refactoring (Improvements and Refactoring)

Giai đoạn Phân tích dẫn đến việc xác định các cải tiến được khuyến nghị cho dịch vụ. Giai đoạn tiếp theo được tiến hành như sau:

1.  Các cải tiến được ưu tiên hóa dựa trên tầm quan trọng đối với độ tin cậy dịch vụ.
2.  Các ưu tiên được thảo luận và thương lượng với đội phát triển, và một kế hoạch thực thi được đồng ý.
3.  Cả hai đội SRE và phát triển sản phẩm đều tham gia và hỗ trợ nhau trong việc refactor các phần của dịch vụ hoặc triển khai các tính năng bổ sung.

Giai đoạn này thường biến động nhất về thời lượng và mức độ nỗ lực. Thời gian và công sức cần bỏ ra phụ thuộc vào việc có đủ thời gian kỹ thuật để refactor hay không, mức độ trưởng thành và độ phức tạp của dịch vụ tại thời điểm bắt đầu review, cùng vô số yếu tố khác.

## Đào tạo (Training)

Việc quản lý một dịch vụ trong production thường do toàn bộ đội SRE đảm nhận. Để đảm bảo đội đã sẵn sàng, những người review SRE đã dẫn dắt PRR sẽ tiếp nhận trách nhiệm đào tạo đội, bao gồm cả việc cung cấp tài liệu cần thiết để hỗ trợ dịch vụ. Thường thì, với sự giúp đỡ và tham gia của đội phát triển, các kỹ sư này tổ chức một loạt các phiên và bài tập đào tạo. Hướng dẫn có thể bao gồm:

-   Các tổng quan thiết kế
-   Các đào sâu vào các luồng yêu cầu khác nhau trong hệ thống
-   Một mô tả về thiết lập production
-   Các bài tập thực hành cho các khía cạnh khác nhau của vận hành hệ thống

Khi đào tạo kết thúc, đội SRE nên đã sẵn sàng để quản lý dịch vụ.

## Onboarding (Đưa vào)

Giai đoạn Đào tạo mở khóa quy trình onboarding dịch vụ do đội SRE thực hiện. Giai đoạn này bao gồm việc chuyển giao dần dần trách nhiệm và quyền sở hữu đối với các khía cạnh production khác nhau của dịch vụ, chẳng hạn như các phần của vận hành, quy trình quản lý thay đổi, các quyền truy cập, và vân vân. Đội SRE tiếp tục tập trung vào các khu vực production khác nhau đã được đề cập trước đó. Để hoàn tất sự chuyển đổi, đội phát triển phải có sẵn để dự phòng và cố vấn cho đội SRE trong một khoảng thời gian khi đội này ổn định trong việc quản lý production cho dịch vụ. Mối quan hệ này trở thành nền tảng cho công việc liên tục giữa các đội.

## Cải tiến Liên tục (Continuous Improvement)

Các dịch vụ liên tục thay đổi để đáp ứng những đòi hỏi và điều kiện mới, chẳng hạn như yêu cầu tính năng mới từ người dùng, sự tiến hóa của các phụ thuộc hệ thống, hay các nâng cấp công nghệ, cùng nhiều yếu tố khác. Để duy trì các tiêu chuẩn độ tin cậy trước những thay đổi này, đội SRE phải thúc đẩy cải tiến liên tục. Trong quá trình vận hành, đội SRE tự nhiên tích lũy chuyên môn về dịch vụ thông qua việc xem xét các thay đổi mới, xử lý incident, và đặc biệt là khi thực hiện postmortem/phân tích nguyên nhân gốc rễ. Chuyên môn này được chia sẻ với đội phát triển dưới dạng các đề xuất và khuyến nghị cho những thay đổi đối với dịch vụ, bất cứ khi nào có thể thêm các tính năng, thành phần, hay phụ thuộc mới. Những bài học từ việc quản lý dịch vụ cũng được đóng góp vào các thực hành tốt nhất, được tài liệu hóa trong Production Guide và các nơi khác.

### Tham gia với Shakespeare (Engaging with Shakespeare)

Ban đầu, các nhà phát triển của dịch vụ Shakespeare chịu trách nhiệm cho sản phẩm, bao gồm việc mang máy gọi trực (pager) cho phản ứng khẩn cấp. Tuy nhiên, với việc sử dụng dịch vụ tăng lên và sự tăng trưởng của doanh thu đến từ dịch vụ, hỗ trợ SRE trở nên mong muốn. Sản phẩm đã được ra mắt, nên SRE đã tiến hành một Production Readiness Review. Một trong những điều họ phát hiện là các dashboard không hoàn toàn bao phủ một số metrics được định nghĩa trong SLO, nên điều đó cần phải được sửa. Sau khi tất cả các vấn đề đã được khai báo được sửa, SRE tiếp nhận máy gọi trực cho dịch vụ, mặc dù hai nhà phát triển cũng trong vòng on-call. Các nhà phát triển đang tham gia vào cuộc họp on-call hàng tuần thảo luận các vấn đề của tuần trước và cách xử lý các hoạt động bảo trì quy mô lớn hoặc tắt (turndown) cluster sắp tới. Ngoài ra, các kế hoạch tương lai cho dịch vụ giờ được thảo luận với các SRE để đảm bảo rằng các ra mắt mới sẽ diễn ra hoàn hảo (mặc dù định luật Murphy luôn luôn tìm kiếm các cơ hội để phá hỏng điều đó).

## Tiến hóa Mô hình PRR Đơn giản: Tham gia Sớm (Evolving the Simple PRR Model: Early Engagement)

Cho đến nay, chúng tôi đã thảo luận về Production Readiness Review theo cách nó được áp dụng trong Mô hình PRR Đơn giản, vốn chỉ giới hạn ở các dịch vụ đã vào giai đoạn Launch (Ra mắt). Mô hình này đi kèm một số hạn chế và chi phí nhất định. Ví dụ:

-   Giao tiếp bổ sung giữa các đội có thể làm tăng một số chi phí phát sinh quy trình cho đội phát triển, và gánh nặng nhận thức cho những người review SRE.
-   Những người review SRE phù hợp phải sẵn sàng, và có khả năng quản lý thời gian và ưu tiên của họ đối với các sự tham gia hiện có của họ.
-   Công việc của SRE cần có độ minh bạch cao và được đội phát triển review kỹ lưỡng để đảm bảo chia sẻ kiến thức hiệu quả. Về cơ bản, SRE nên làm việc như một phần của đội phát triển, thay vì là một đơn vị bên ngoài.

Tuy nhiên, các giới hạn chính của Mô hình PRR bắt nguồn từ thực tế là dịch vụ đã được ra mắt và đang phục vụ ở quy mô lớn, trong khi sự tham gia của SRE lại diễn ra rất muộn trong vòng đời phát triển. Nếu PRR được áp dụng sớm hơn, SRE sẽ có nhiều cơ hội hơn để khắc phục các vấn đề tiềm ẩn, từ đó cải thiện cả hiệu quả tham gia của SRE lẫn triển vọng thành công của dịch vụ. Ngược lại, những tác động kéo theo có thể gây ra thách thức đáng kể cho cả hai yếu tố này.

## Các Ứng viên cho Tham gia Sớm (Candidates for Early Engagement)

Mô hình Tham gia Sớm đưa SRE vào vòng đời phát triển sớm hơn nhằm đạt được những lợi thế bổ sung đáng kể. Để áp dụng mô hình này, cần xác định tầm quan trọng và/hoặc giá trị kinh doanh của dịch vụ ngay từ đầu, đồng thời đánh giá xem dịch vụ có đủ quy mô hoặc độ phức tạp để hưởng lợi từ chuyên môn SRE hay không. Các dịch vụ áp dụng thường có những đặc điểm sau:

-   Dịch vụ triển khai các chức năng mới đáng kể và sẽ là một phần của một hệ thống hiện có đã được SRE quản lý.
-   Dịch vụ là một bản viết lại hoặc lựa chọn thay thế đáng kể cho một hệ thống hiện có, nhắm đến cùng các trường hợp sử dụng.
-   Đội phát triển đã tìm kiếm lời khuyên SRE hoặc tiếp cận SRE để tiếp nhận khi ra mắt.

Mô hình Tham gia Sớm đặt SRE vào ngay trong quy trình phát triển. Mục tiêu của SRE không đổi, chỉ khác ở cách thức để đạt được dịch vụ production tốt hơn. SRE tham gia từ khâu Thiết kế và các giai đoạn tiếp theo, sau đó tiếp nhận dịch vụ vào bất kỳ thời điểm nào trong hoặc sau giai đoạn Build (Xây dựng). Mô hình này dựa trên sự hợp tác chủ động giữa đội phát triển và SRE.

## Các Lợi ích của Mô hình Tham gia Sớm (Benefits of the Early Engagement Model)

Mô hình Tham gia Sớm tuy đi kèm một số rủi ro và thách thức đã được đề cập trước đó, nhưng nhờ chuyên môn SRE cùng sự hợp tác bổ sung xuyên suốt vòng đời sản phẩm, mô hình này mang lại lợi ích đáng kể so với việc tham gia muộn hơn trong vòng đời dịch vụ.

### Giai đoạn Thiết kế (Design phase)

Sự tham gia của SRE ở giai đoạn Thiết kế có thể ngăn chặn nhiều vấn đề hoặc incident phát sinh sau này trên production. Dù các quyết định thiết kế vẫn có thể đảo ngược hoặc chỉnh sửa trong vòng đời phát triển, những thay đổi như vậy đòi hỏi chi phí lớn về nỗ lực và độ phức tạp. Tốt nhất là các incident production không bao giờ xảy ra!

Đôi khi, những đánh đổi (trade-off) khó khăn buộc ta phải chọn một thiết kế không hoàn hảo. Việc tham gia vào giai đoạn Thiết kế giúp các SRE nhận ra những đánh đổi này ngay từ đầu và cùng đưa ra quyết định chọn phương án kém lý tưởng hơn. Sự tham gia sớm của SRE nhằm giảm thiểu các tranh chấp về lựa chọn thiết kế khi dịch vụ đã chạy trên production.

### Xây dựng và triển khai (Build and implementation)

Giai đoạn Build tập trung vào các khía cạnh production như đo lường (instrumentation) và metrics, các biện pháp kiểm soát vận hành và khẩn cấp, mức sử dụng tài nguyên, cùng hiệu quả. Ở giai đoạn này, SRE có thể tác động và cải thiện quá trình triển khai bằng cách đề xuất các thư viện, thành phần hiện có cụ thể, hoặc hỗ trợ tích hợp một số biện pháp kiểm soát vào hệ thống. Việc SRE tham gia sớm giúp tạo tiền đề cho khả năng vận hành dễ dàng hơn về sau, đồng thời cho phép họ tích lũy kinh nghiệm vận hành trước khi hệ thống ra mắt.

### Ra mắt (Launch)

SRE cũng có thể giúp triển khai các mẫu và biện pháp kiểm soát ra mắt được sử dụng rộng rãi. Ví dụ, SRE có thể giúp triển khai một thiết lập "dark launch" (ra mắt tối), trong đó một phần traffic từ những người dùng hiện có được gửi đến dịch vụ mới, bên cạnh việc được gửi đến dịch vụ production trực. Các phản hồi từ dịch vụ mới là "tối" vì chúng bị vứt bỏ và không thực sự hiển thị cho người dùng. Các thực hành như ra mắt tối cho phép đội đạt được hiểu biết vận hành, giải quyết các vấn đề mà không ảnh hưởng đến những người dùng hiện có, và giảm rủi ro gặp các vấn đề sau ra mắt. Một ra mắt suôn sẻ vô cùng hữu ích trong việc giữ gánh nặng vận hành thấp và duy trì động lực phát triển sau ra mắt. Các gián đoạn xung quanh ra mắt dễ dàng dẫn đến các thay đổi khẩn cấp đối với code nguồn và production, và làm gián đoạn công việc của đội phát triển trên các tính năng tương lai.

### Sau ra mắt (Post-launch)

Hệ thống ổn định ngay từ thời điểm ra mắt giúp đội phát triển giảm bớt xung đột ưu tiên giữa việc cải thiện độ tin cậy dịch vụ và thêm tính năng mới. Ở các giai đoạn sau, những bài học từ trước có thể hỗ trợ tốt hơn cho quá trình refactor hoặc thiết kế lại.

Nhờ sự tham gia mở rộng, đội SRE có thể sẵn sàng tiếp nhận dịch vụ mới sớm hơn nhiều so với khả năng của Mô hình PRR Đơn giản. Việc các đội SRE và phát triển tham gia sâu hơn, kéo dài hơn cũng tạo nên mối quan hệ hợp tác có thể duy trì lâu dài. Mối quan hệ chéo-đội tích cực này nuôi dưỡng cảm giác đoàn kết lẫn nhau, đồng thời giúp SRE thiết lập sự sở hữu đối với trách nhiệm production.

### Rút khỏi một dịch vụ (Disengaging from a service)

Đôi khi, một dịch vụ không cần đến sự quản lý đầy đủ của đội SRE. Việc xác định này có thể diễn ra sau khi ra mắt, hoặc SRE có thể tham gia hỗ trợ một dịch vụ nhưng không chính thức tiếp nhận nó. Đây là kết quả tích cực, vì dịch vụ đã được xây dựng để đảm bảo độ tin cậy và có mức bảo trì thấp, nên có thể tiếp tục do đội phát triển phụ trách.

SRE cũng có thể tham gia sớm vào một dịch vụ không đạt được mức sử dụng như dự báo. Trong những trường hợp như vậy, công sức SRE bỏ ra đơn thuần chỉ là một phần của rủi ro kinh doanh tổng thể đi kèm với các dự án mới, và là chi phí nhỏ so với thành công của những dự án đạt quy mô kỳ vọng. Đội SRE có thể được phân công lại, và các bài học rút ra có thể được tích hợp vào quy trình tham gia.

## Tiến hóa Phát triển Dịch vụ: Frameworks và Nền tảng SRE (Evolving Services Development: Frameworks and SRE Platform)

Mô hình Tham gia Sớm đã giúp sự tham gia của SRE tiến xa hơn so với Mô hình PRR Đơn giản, vốn chỉ áp dụng cho các dịch vụ đã ra mắt. Tuy nhiên, vẫn cần nỗ lực thêm để mở rộng sự tham gia của SRE lên cấp độ tiếp theo thông qua thiết kế cho độ tin cậy.

## Các Bài học Rút ra (Lessons Learned)

Theo thời gian, mô hình tham gia SRE được mô tả cho đến nay đã tạo ra một số mẫu riêng biệt:

-   Mỗi dịch vụ khi onboarding cần hai hoặc ba SRE và thường mất hai hoặc ba quý. Thời gian chờ trước khi thực hiện một PRR cũng khá dài (vài quý). Khối lượng công việc tăng theo số dịch vụ được review, trong khi nguồn lực SRE lại hạn chế, khiến các PRR bị tắc nghẽn. Do đó, quá trình tiếp nhận dịch vụ phải chạy tuần tự và cần ưu tiên chặt chẽ.
-   Do các thực hành phần mềm khác nhau giữa các dịch vụ, mỗi tính năng production được triển khai theo những cách riêng. Để đáp ứng các tiêu chuẩn do PRR dẫn dắt, các tính năng thường phải được triển khai lại cụ thể cho từng dịch vụ hoặc, tốt nhất, một lần cho mỗi tập hợp con nhỏ các dịch vụ chia sẻ code. Những lần triển khai lại này gây lãng phí nỗ lực kỹ thuật. Một ví dụ điển hình là việc triển khai lặp đi lặp lại các framework logging có chức năng tương tự bằng cùng một ngôn ngữ, do các dịch vụ khác nhau không sử dụng cùng cấu trúc code.
-   Một review các vấn đề và outage dịch vụ chung đã tiết lộ một số mẫu, nhưng không có cách nào dễ dàng nhân bản các sửa chữa và cải tiến xuyên qua các dịch vụ. Các ví dụ điển hình bao gồm các tình huống quá tải dịch vụ và việc hotspot (điểm nóng) dữ liệu.
-   Các đóng góp kỹ thuật phần mềm của SRE thường chỉ mang tính cục bộ cho từng dịch vụ, nên việc xây dựng các giải pháp tổng quát để tái sử dụng là khó khăn. Hệ quả là, không có cách nào dễ dàng để triển khai các bài học mới mà từng đội SRE học được và các thực hành tốt nhất xuyên qua các dịch vụ đã được onboarding.

## Các Yếu tố Bên ngoài Ảnh hưởng đến SRE (External Factors Affecting SRE)

Các yếu tố bên ngoài truyền thống đã tạo áp lực lên tổ chức SRE và các tài nguyên của nó theo một số cách.

Google ngày càng theo xu hướng ngành chuyển sang microservices (vi dịch vụ).<sup>[1](#fn1)</sup> Hệ quả là, cả số lượng yêu cầu hỗ trợ SRE lẫn tính đa dạng (cardinality) của các dịch vụ cần hỗ trợ đều tăng. Vì mỗi dịch vụ có một chi phí vận hành cố định cơ bản, ngay cả các dịch vụ đơn giản cũng đòi hỏi nhiều nhân sự hơn. Các microservices cũng ngụ ý một kỳ vọng về thời gian dẫn trước triển khai thấp hơn, điều mà không thể với mô hình PRR trước đó (đã có thời gian dẫn trước là nhiều tháng).

Việc tuyển dụng SRE có kinh nghiệm, đáp ứng yêu cầu là điều khó khăn và tốn kém. Dù bộ phận tuyển dụng đã nỗ lực hết sức, số lượng SRE vẫn không bao giờ đủ để hỗ trợ tất cả các dịch vụ cần đến chuyên môn của họ. Sau khi tuyển được, quá trình đào tạo SRE cũng kéo dài hơn so với mức thông thường của các kỹ sư phát triển.

Cuối cùng, tổ chức SRE phải đáp ứng nhu cầu của ngày càng nhiều đội phát triển chưa từng được hỗ trợ trực tiếp. Điều này đòi hỏi mở rộng mô hình hỗ trợ SRE vượt xa khái niệm và mô hình tham gia ban đầu.

## Hướng đến một Giải pháp Cấu trúc: Frameworks (Toward a Structural Solution: Frameworks)

Để phản ứng hiệu quả với những điều kiện này, đã cần thiết phải phát triển một mô hình cho phép các nguyên tắc sau:

Thực hành tốt nhất được mã hóa

Khả năng đưa những gì đã hoạt động tốt trong production vào code, giúp các dịch vụ đơn thuần có thể sử dụng code này và trở nên “sẵn sàng production” ngay từ khâu thiết kế.

Các giải pháp có thể tái sử dụng

Các triển khai chung và dễ chia sẻ của các kỹ thuật được sử dụng để giảm nhẹ các vấn đề khả năng mở rộng và độ tin cậy.

Một nền tảng production chung với một bề mặt điều khiển chung

Các tập hợp giao diện nhất quán đến các cơ sở production, các tập hợp biện pháp kiểm soát vận hành nhất quán, cùng giám sát, logging và cấu hình nhất quán cho tất cả các dịch vụ.

Tự động hóa dễ dàng hơn và các hệ thống thông minh hơn

Một bề mặt điều khiển chung cho phép tự động hóa và các hệ thống thông minh hoạt động ở cấp độ mà trước đây không thể. Ví dụ, các SRE có thể dễ dàng có được góc nhìn duy nhất về thông tin liên quan đến một sự cố outage, thay vì phải thu thập và phân tích thủ công phần lớn dữ liệu thô từ các nguồn khác nhau (log, dữ liệu giám sát, và vân vân).

Từ các nguyên tắc này, một bộ framework nền tảng và dịch vụ do SRE hỗ trợ đã ra đời, mỗi môi trường chúng tôi hỗ trợ (Java, C++, Go) có một bộ riêng. Các dịch vụ xây dựng trên những framework này chia sẻ các triển khai được thiết kế để chạy trên nền tảng do SRE hỗ trợ, và được cả đội SRE lẫn đội phát triển cùng duy trì. Thay đổi lớn nhất mà các framework mang lại là cho phép các đội phát triển sản phẩm thiết kế ứng dụng dựa trên các giải pháp framework đã được xây dựng và SRE phê duyệt, thay vì phải tự lắp ráp ứng dụng theo các quy định SRE sau này, hoặc phải huy động thêm nhiều SRE hơn để hỗ trợ một dịch vụ khác biệt đáng kể so với các dịch vụ khác của Google.

Một ứng dụng thường chứa một số logic kinh doanh, vốn phụ thuộc vào các thành phần hạ tầng khác nhau. Trong môi trường production, mối quan tâm chính của SRE thường nằm ở các phần hạ tầng của dịch vụ. Các framework dịch vụ giúp triển khai code hạ tầng một cách chuẩn hóa, đồng thời giải quyết các mối quan tâm production khác nhau. Mỗi mối quan tâm được đóng gói trong một hoặc nhiều module framework, mỗi module cung cấp một giải pháp nhất quán cho một lĩnh vực vấn đề hoặc phụ thuộc hạ tầng cụ thể. Các module framework giải quyết những mối quan tâm SRE đã liệt kê trước đó, chẳng hạn như:

-   Đo lường (Instrumentation) và metrics
-   Request logging (ghi log yêu cầu)
-   Các hệ thống điều khiển liên quan đến quản lý traffic và tải

SRE xây dựng các module framework nhằm triển khai các giải pháp chuẩn cho khu vực production được quan tâm. Nhờ đó, các đội phát triển có thể tập trung vào logic kinh doanh, vì framework đã đảm bảo việc sử dụng hạ tầng đúng đắn.

Về cơ bản, một framework là một triển khai quy định cách sử dụng một tập hợp các thành phần phần mềm và cách chuẩn để kết hợp chúng. Framework cũng có thể phơi bày các tính năng để kiểm soát các thành phần khác nhau theo cách nhất quán. Ví dụ, một framework có thể cung cấp:

-   Logic kinh doanh được tổ chức như các thành phần ngữ nghĩa được định nghĩa rõ ràng có thể được tham chiếu bằng các thuật ngữ chuẩn
-   Các chiều chuẩn cho đo lường giám sát
-   Một định dạng chuẩn cho log debug yêu cầu
-   Một định dạng cấu hình chuẩn để quản lý việc loại bỏ tải (load shedding)
-   Năng lực của một server đơn lẻ và việc xác định "quá tải", cả hai đều có thể dùng chung một phép đo ngữ nghĩa nhất quán để phản hồi cho các hệ thống điều khiển khác nhau

Các framework mang lại nhiều lợi ích trước mắt về tính nhất quán và hiệu quả. Chúng giúp các nhà phát triển không phải tự dán ghép và cấu hình từng thành phần riêng lẻ theo cách tùy hứng, không tương thích cho từng dịch vụ, rồi để các SRE phải review thủ công. Thay vào đó, chúng thúc đẩy một giải pháp tái sử dụng duy nhất cho các mối quan tâm production xuyên suốt các dịch vụ, nhờ đó người dùng framework có cùng một triển khai chung và chỉ cần tối thiểu hóa các khác biệt về cấu hình.

Google hỗ trợ một số ngôn ngữ chính cho phát triển ứng dụng, và các framework được triển khai xuyên qua tất cả những ngôn ngữ này. Trong khi các triển khai khác nhau của framework (chẳng hạn trong C++ so với Java) không thể chia sẻ code, mục tiêu là phơi bày cùng API (giao diện lập trình ứng dụng), hành vi, cấu hình, và các biện pháp điều khiển cho chức năng giống hệt nhau. Do đó, các đội phát triển có thể chọn nền tảng ngôn ngữ phù hợp với nhu cầu và kinh nghiệm của họ, trong khi các SRE vẫn có thể kỳ vọng cùng hành vi quen thuộc trong production và các công cụ chuẩn để quản lý dịch vụ.

## Các Lợi ích Dịch vụ Mới và Quản lý (New Service and Management Benefits)

Cách tiếp cận mang tính cấu trúc, được xây dựng dựa trên các framework dịch vụ, một nền tảng production và bề mặt điều khiển chung, đã mang lại nhiều lợi ích mới.

### Chi phí phát sinh vận hành thấp hơn đáng kể (Significantly lower operational overhead)

Một nền tảng production được xây dựng trên các framework với các quy ước mạnh hơn đã giảm đáng kể chi phí phát sinh vận hành, vì các lý do sau:

-   Nó hỗ trợ các kiểm thử tuân thủ chặt chẽ cho cấu trúc code, các phụ thuộc, các test, các hướng dẫn phong cách code, và vân vân. Tính năng này cũng cải thiện quyền riêng tư dữ liệu người dùng, kiểm thử, và tuân thủ bảo mật.
-   Nó có tính năng triển khai, giám sát, và tự động hóa dịch vụ tích hợp sẵn cho tất cả các dịch vụ.
-   Nó tạo điều kiện quản lý dễ dàng hơn cho một số lượng lớn các dịch vụ, đặc biệt là các micro-service, đang tăng về số lượng.
-   Nó giúp triển khai nhanh hơn đáng kể: một ý tưởng có thể đạt chất lượng production cấp SRE và được triển khai hoàn toàn chỉ trong vài ngày!

### Hỗ trợ phổ quát theo thiết kế (Universal support by design)

Số lượng dịch vụ tại Google tăng liên tục, khiến phần lớn trong số chúng không đủ tầm quan trọng để SRE tham gia hay duy trì. Tuy nhiên, ngay cả những dịch vụ không có hỗ trợ SRE đầy đủ vẫn có thể được xây dựng dựa trên các tính năng production do SRE phát triển và bảo trì. Cách làm này giúp phá vỡ rào cản về nhân sự SRE. Việc cung cấp các tiêu chuẩn và công cụ production có hỗ trợ SRE cho tất cả các đội giúp nâng cao chất lượng dịch vụ tổng thể trên toàn Google. Hơn nữa, mọi dịch vụ triển khai bằng các framework tự động đều hưởng lợi từ những cải tiến được thực hiện dần cho các module framework theo thời gian.

### Các sự tham gia nhanh hơn, chi phí phát sinh thấp hơn (Faster, lower overhead engagements)

Cách tiếp cận framework kết quả trong việc thực hiện PRR nhanh hơn vì chúng tôi có thể dựa vào:

-   Các tính năng dịch vụ tích hợp sẵn như một phần của triển khai framework
-   Onboarding dịch vụ nhanh hơn (thường được hoàn thành bởi một SRE duy nhất trong một quý)
-   Gánh nặng nhận thức ít hơn cho các đội SRE quản lý các dịch vụ được xây dựng sử dụng các framework

Nhờ những thuộc tính này, các đội SRE có thể giảm bớt công sức đánh giá và xác nhận khi onboarding dịch vụ, đồng thời vẫn duy trì tiêu chuẩn chất lượng production cao.

### Một mô hình tham gia mới dựa trên trách nhiệm chia sẻ (A new engagement model based on shared responsibility)

Mô hình tham gia SRE ban đầu chỉ trình bày hai lựa chọn: hoặc hỗ trợ SRE đầy đủ, hoặc xấp xỉ không có sự tham gia SRE.<sup>[2](#fn2)</sup>

Nhờ có chung một cấu trúc dịch vụ, các quy ước và hạ tầng phần mềm, nền tảng production này cho phép đội SRE hỗ trợ hạ tầng “nền tảng”, trong khi các đội phát triển đảm nhận on-call cho các vấn đề chức năng của dịch vụ — cụ thể là các bug trong code ứng dụng. Theo mô hình này, SRE chịu trách nhiệm phát triển và duy trì phần lớn hạ tầng phần mềm của dịch vụ, đặc biệt là các hệ thống điều khiển như load shedding (loại bỏ tải), quá tải (overload), tự động hóa, quản lý traffic, logging và giám sát.

Mô hình này đánh dấu sự thay đổi lớn so với cách quản lý dịch vụ ban đầu, thể hiện qua hai khía cạnh: một là mô hình tương tác mới giữa SRE và các đội phát triển, hai là mô hình nhân sự mới cho việc quản lý dịch vụ do SRE hỗ trợ.<sup>[3](#fn3)</sup>

## Kết luận (Conclusion)

Độ tin cậy dịch vụ có thể được cải thiện nhờ sự tham gia của SRE, thông qua quy trình review và cải thiện có hệ thống các khía cạnh production. Simple Production Readiness Review (Đánh giá Sẵn sàng Production Đơn giản) là cách tiếp cận có hệ thống đầu tiên của Google SRE, giúp chuẩn hóa mô hình tham gia SRE, nhưng chỉ áp dụng cho các dịch vụ đã vào giai đoạn Launch.

Theo thời gian, SRE đã mở rộng và cải thiện mô hình này. Mô hình Tham gia Sớm đưa SRE vào vòng đời phát triển sớm hơn nhằm “thiết kế cho độ tin cậy.” Khi nhu cầu về chuyên môn SRE tiếp tục tăng, sự cần thiết của một mô hình tham gia có khả năng mở rộng hơn trở nên rõ ràng. Các framework cho dịch vụ production đã được phát triển để đáp ứng nhu cầu này: các mẫu code dựa trên thực hành tốt nhất production được chuẩn hóa và đóng gói trong framework, giúp việc sử dụng framework trở thành cách tiếp cận được khuyến nghị, nhất quán và tương đối đơn giản để xây dựng dịch vụ sẵn sàng production.

Cả ba mô hình tham gia được mô tả đều vẫn được áp dụng tại Google. Tuy nhiên, việc sử dụng các framework đang trở thành yếu tố nổi bật trong quá trình xây dựng các dịch vụ sẵn sàng production tại Google, đồng thời mở rộng sâu sắc đóng góp của SRE, giảm chi phí phát sinh do quản lý dịch vụ và cải thiện chất lượng dịch vụ cơ bản trên toàn tổ chức.

<a id="fn1"></a>[1](#fn1) Xem trang Wikipedia về microservices tại [*https://en.wikipedia.org/wiki/Microservices*](https://en.wikipedia.org/wiki/Microservices).

<a id="fn2"></a>[2](#fn2) Đôi khi, có các sự tham gia tham vấn bởi các đội SRE với một số dịch vụ chưa được onboarding, nhưng các tham vấn là một cách tiếp cận cố gắng hết sức và bị giới hạn về số lượng và phạm vi.

<a id="fn3"></a>[3](#fn3) Mô hình mới của quản lý dịch vụ thay đổi mô hình nhân sự SRE theo hai cách: (1) vì nhiều công nghệ dịch vụ là chung, nó giảm số lượng SRE yêu cầu cho mỗi dịch vụ; (2) nó cho phép việc tạo ra các nền tảng production với sự tách biệt mối quan tâm giữa hỗ trợ nền tảng production (do các SRE thực hiện) và hỗ trợ logic kinh doanh cụ thể cho dịch vụ, cái mà tiếp tục ở lại với đội phát triển. Những nền tảng này được bố trí nhân sự dựa trên nhu cầu duy trì nền tảng chứ không phải trên số lượng dịch vụ, và có thể được chia sẻ xuyên qua các sản phẩm.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
