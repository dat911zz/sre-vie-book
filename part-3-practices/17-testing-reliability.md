# Chương 17. Kiểm thử cho Độ tin cậy (Testing for Reliability)

> **Nguyên bản:** [Chapter 17 - Testing for Reliability](https://sre.google/sre-book/testing-reliability/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Alex Perry và Max Luebbe
*Biên tập:* Diane Bates

> Nếu bạn chưa thử nó, hãy giả định rằng nó bị hỏng.
>
> Không rõ tác giả

Một trong những trách nhiệm chính của Site Reliability Engineer (SRE) là định lượng mức độ tin cậy của các hệ thống mà họ vận hành. Để làm điều này, các SRE áp dụng các [kỹ thuật kiểm thử phần mềm](https://sre.google/sre-book/testing-reliability/) cổ điển, điều chỉnh cho phù hợp với hệ thống quy mô lớn.<sup>[1](#fn1)</sup> Mức độ tin cậy có thể được đánh giá dựa trên cả độ tin cậy trong quá khứ lẫn dự báo cho tương lai. Trong đó, độ tin cậy quá khứ được xác định thông qua việc phân tích dữ liệu từ hoạt động giám sát hệ thống trước đó, còn độ tin cậy tương lai được định lượng bằng cách đưa ra các dự đoán dựa trên dữ liệu lịch sử đó. Để những dự đoán này đủ độ tin cậy và hữu ích, một trong các điều kiện sau phải được đáp ứng:

-   Site không thay đổi theo thời gian, không có release phần mềm hay thay đổi trong fleet server, nghĩa là hành vi trong tương lai sẽ giống hành vi trong quá khứ.
-   Bạn có thể mô tả tin cậy mọi thay đổi đối với site, để phân tích có thể tính đến sự không chắc chắn (uncertainty) do mỗi thay đổi gây ra.

Kiểm thử là cơ chế bạn dùng để minh họa các vùng tương đương cụ thể khi có thay đổi xảy ra.<sup>[2](#fn2)</sup> Mỗi kiểm thử vượt qua cả trước lẫn sau một thay đổi đều giúp giảm bớt sự không chắc chắn mà phân tích cần tính đến. Kiểm thử kỹ lưỡng giúp chúng tôi dự đoán độ tin cậy trong tương lai của một site nhất định với mức chi tiết đủ để hữu ích trên thực tế.

Lượng kiểm thử cần thiết phụ thuộc vào yêu cầu độ tin cậy của hệ thống. Khi tỷ lệ codebase được bao phủ bởi kiểm thử tăng, bạn giảm được sự không chắc chắn và nguy cơ giảm độ tin cậy từ mỗi thay đổi. Bao phủ kiểm thử đầy đủ nghĩa là bạn có thể thực hiện nhiều thay đổi hơn trước khi độ tin cậy rơi xuống dưới mức chấp nhận được. Nếu bạn thực hiện quá nhiều thay đổi quá nhanh, độ tin cậy dự đoán sẽ tiến đến giới hạn chấp nhận được. Khi đó, bạn có thể muốn tạm dừng các thay đổi cho đến khi dữ liệu giám sát mới tích lũy đủ. Dữ liệu đang tích lũy bổ sung cho phần bao phủ đã kiểm thử, xác nhận độ tin cậy đang được khẳng định cho các đường thực thi đã sửa. Giả định các client được phục vụ phân bố ngẫu nhiên [[Woo96]](https://sre.google/sre-book/bibliography#Woo96), thống kê lấy mẫu có thể ngoại suy từ các metrics được giám sát để biết hành vi tổng hợp có đang dùng các đường mới hay không. Những thống kê này xác định các khu vực cần kiểm thử tốt hơn hoặc các cải tiến (retrofitting) khác.

### Mối quan hệ Giữa Kiểm thử và Thời gian Trung bình Để Sửa chữa (Relationships Between Testing and Mean Time to Repair)

Việc vượt qua một kiểm thử hoặc một chuỗi kiểm thử không nhất thiết chứng minh độ tin cậy. Tuy nhiên, các kiểm thử đang thất bại nhìn chung chứng minh sự vắng mặt của độ tin cậy.

Hệ thống giám sát có thể phát hiện bug, nhưng tốc độ phản ứng chỉ nhanh bằng đường ống báo cáo (reporting pipeline). *Mean Time to Repair* (MTTR) đo thời gian đội vận hành mất để sửa bug, thông qua việc hoàn tác (rollback) hoặc một hành động khác.

Một hệ thống kiểm thử có thể phát hiện bug với MTTR bằng 0. Điều này xảy ra khi một kiểm thử cấp hệ thống được áp dụng cho một hệ thống con, và kiểm thử đó bắt đúng vấn đề mà giám sát sẽ phát hiện. Kiểm thử như vậy cho phép chặn push để bug không bao giờ đến production (mặc dù vẫn cần sửa trong code nguồn). Việc sửa các bug MTTR-0 bằng cách chặn push vừa nhanh vừa tiện lợi. Càng nhiều bug bạn tìm thấy với MTTR bằng 0, *Mean Time Between Failures* (MTBF) mà người dùng trải nghiệm càng cao.

Khi MTBF tăng nhờ kiểm thử tốt hơn, các developer được khuyến khích release tính năng nhanh hơn. Tất nhiên, một số tính năng sẽ có bug. Khi phát hiện và sửa các bug này, tốc độ release sẽ được điều chỉnh ngược lại.

Phần lớn các tác giả viết về kiểm thử phần mềm đều đồng thuận về mức bao phủ (coverage) cần thiết. Những bất đồng chủ yếu xuất phát từ sự mâu thuẫn trong thuật ngữ, cách nhấn mạnh khác nhau về tác động của kiểm thử ở từng giai đoạn vòng đời phần mềm, hoặc do đặc thù của các hệ thống mà họ đã kiểm thử. Để tìm hiểu thêm về kiểm thử tại Google nói chung, hãy xem [[Whi12]](https://sre.google/sre-book/bibliography#Whi12). Các phần tiếp theo sẽ làm rõ cách sử dụng các thuật ngữ liên quan đến kiểm thử phần mềm trong chương này.

## Các Loại Kiểm thử Phần mềm (Types of Software Testing)

Kiểm thử phần mềm thường được chia thành hai nhóm: truyền thống và production. Trong quá trình phát triển, kiểm thử truyền thống phổ biến hơn, dùng để đánh giá tính chính xác của phần mềm ở trạng thái offline. Ngược lại, kiểm thử production được thực hiện trên dịch vụ web đang chạy, nhằm xác nhận hệ thống phần mềm đã triển khai có hoạt động đúng hay không.

## Các Kiểm thử Truyền thống (Traditional Tests)

Như [Hình 17-1](#hinh-17-1) cho thấy, kiểm thử phần mềm truyền thống bắt đầu với kiểm thử unit. Các bài kiểm thử cho chức năng phức tạp hơn được xếp chồng lên trên kiểm thử unit.


<a id="hinh-17-1"></a>![Hình 17-1](../assets/imgs/fig-17-1.jpg)

[Hình 17-1.](#hinh-17-1) Hệ phân cấp của các kiểm thử truyền thống.

### Kiểm thử Unit (Unit tests)

Một *kiểm thử unit* là dạng kiểm thử phần mềm nhỏ nhất và đơn giản nhất. Chúng đánh giá một đơn vị phần mềm có thể tách rời, như một class hoặc function, về tính chính xác độc lập với hệ thống phần mềm lớn hơn chứa đơn vị đó. Kiểm thử unit cũng được dùng như một dạng đặc tả (specification) để đảm bảo một function hoặc module thực hiện đúng hành vi mà hệ thống yêu cầu. Kiểm thử unit thường được dùng để giới thiệu khái niệm phát triển theo kiểm thử (test-driven development).

### Kiểm thử Tích hợp (Integration tests)

Các thành phần phần mềm vượt qua kiểm thử unit sẽ được ghép lại thành những khối lớn hơn. Sau đó, kỹ sư chạy *kiểm thử tích hợp* trên khối đã ghép để xác nhận nó hoạt động đúng. Tiêm phụ thuộc (dependency injection), thực hiện bằng các công cụ như Dagger,<sup>[3](#fn3)</sup> là kỹ thuật rất mạnh để tạo ra các mock của các phụ thuộc phức tạp, giúp kỹ sư kiểm thử sạch một thành phần. Ví dụ phổ biến của dependency injection là thay thế một database có trạng thái (stateful) bằng một mock nhẹ có hành vi được quy định chính xác.

### Kiểm thử Hệ thống (System tests)

*Kiểm thử hệ thống* là loại kiểm thử có quy mô lớn nhất mà kỹ sư thực hiện trên hệ thống chưa triển khai. Trong đó, các module thuộc một thành phần (ví dụ: server) đã vượt qua kiểm thử tích hợp được lắp ráp thành hệ thống, sau đó kỹ sư kiểm tra chức năng đầu-cuối (end-to-end) của hệ thống. Các kiểm thử hệ thống có nhiều biến thể khác nhau:

#### Kiểm thử khói (Smoke tests)

*Smoke tests* — nơi kỹ sư kiểm tra các hành vi rất đơn giản nhưng quan trọng — là một trong những loại kiểm thử hệ thống đơn giản nhất. Smoke tests còn gọi là *sanity testing* (kiểm thử tính hợp lý), dùng để bỏ qua sớm (short-circuit) các kiểm thử bổ sung, tốn kém hơn.

#### Kiểm thử Hiệu năng (Performance tests)

Sau khi smoke test xác lập được các tính chính xác cơ bản, bước tiếp theo thường là viết một biến thể khác của kiểm thử hệ thống nhằm đảm bảo hiệu năng vẫn ở mức chấp nhận được trong suốt vòng đời. Do thời gian phản hồi của các phụ thuộc hay yêu cầu tài nguyên có thể thay đổi đáng kể trong quá trình phát triển, hệ thống cần được kiểm thử để tránh tình trạng chậm dần mà không ai nhận ra (trước khi release đến người dùng). Chẳng hạn, một chương trình có thể tiến hóa để cần 32 GB bộ nhớ thay vì 8 GB, hoặc thời gian phản hồi 10 ms có thể tăng lên 50 ms rồi 100 ms. Kiểm thử hiệu năng giúp đảm bảo hệ thống không suy giảm hoặc trở nên quá tốn kém theo thời gian.

#### Kiểm thử Regression (Regression tests)

Một loại kiểm thử hệ thống khác giúp ngăn các bug lén lút quay lại codebase. Kiểm thử regression có thể ví như một phòng trưng bày các bug "vô lại" từng khiến hệ thống thất bại hoặc cho ra kết quả sai. Bằng cách ghi tài liệu những bug này thành kiểm thử ở cấp hệ thống hoặc tích hợp, kỹ sư đang refactor codebase có thể yên tâm rằng họ không vô tình tái giới thiệu các bug mà họ đã tốn thời gian và công sức để loại bỏ.

Lưu ý rằng kiểm thử có chi phí, cả về thời gian lẫn tài nguyên tính toán. Ở một đầu, kiểm thử unit rất rẻ ở cả hai chiều, vì chúng thường hoàn thành trong vài mili giây trên tài nguyên của một laptop. Ở đầu kia, việc khởi động một server hoàn chỉnh với các phụ thuộc cần thiết (hoặc mock tương đương) để chạy các kiểm thử liên quan có thể mất nhiều thời gian hơn đáng kể — từ vài phút đến vài giờ — và có thể đòi hỏi tài nguyên tính toán chuyên dụng. Nhận thức về các chi phí này là thiết yếu cho năng suất developer, và khuyến khích sử dụng hiệu quả tài nguyên kiểm thử.

## Các Kiểm thử Production (Production Tests)

Kiểm thử production tương tác trực tiếp với hệ thống production đang chạy, khác với hệ thống trong môi trường kiểm thử cô lập (hermetic). Về nhiều mặt, chúng tương tự giám sát hộp đen (black-box monitoring) (xem [Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/)), và do đó đôi khi được gọi là *kiểm thử hộp đen* (black-box testing). Kiểm thử production là thiết yếu để vận hành dịch vụ production đáng tin cậy.

### Các Rollout (Triển khai) Làm rối lẫn nhau các Kiểm thử (Rollouts Entangle Tests)

Thường nói rằng kiểm thử là (hoặc nên được) thực hiện trong môi trường cô lập [[Nar12]](https://sre.google/sre-book/bibliography#Nar12). Điều này ngụ ý rằng production không phải là cô lập. Đúng vậy, production thường không cô lập, vì nhịp độ rollout (rollout cadence) tạo ra các thay đổi đang chạy trong môi trường production thành từng phần nhỏ, được hiểu rõ.

Để kiểm soát sự không chắc chắn và che giấu rủi ro khỏi người dùng, các thay đổi có thể không được push lên trực tuyến theo đúng thứ tự chúng được thêm vào source control. Rollout thường diễn ra theo từng giai đoạn, sử dụng các cơ chế dần dần xáo trộn (shuffle) người dùng, kèm theo giám sát ở mỗi giai đoạn nhằm đảm bảo môi trường mới không gặp phải những vấn đề đã dự đoán nhưng bất ngờ. Hệ quả là toàn bộ môi trường production cố ý không đại diện cho bất kỳ phiên bản cụ thể nào của một binary đã được check vào source control.

Source control có thể chứa nhiều hơn một phiên bản của một binary cùng tệp cấu hình liên quan, đang chờ được đưa lên trực tuyến. Kịch bản này có thể gây vấn đề khi kiểm thử chạy trên môi trường đang chạy. Ví dụ, kiểm thử có thể dùng phiên bản mới nhất của một tệp cấu hình trong source control cùng với một phiên bản cũ hơn của binary đang trực tuyến. Hoặc nó có thể kiểm thử một phiên bản cũ hơn của tệp cấu hình và tìm thấy bug đã được sửa trong phiên bản mới hơn của tệp.

Tương tự, một kiểm thử hệ thống có thể dùng các tệp cấu hình để lắp ráp các module trước khi chạy. Nếu kiểm thử vượt qua, nhưng phiên bản của nó lại là phiên bản mà kiểm thử cấu hình (thảo luận ở phần tiếp theo) thất bại, thì kết quả kiểm thử hợp lệ về mặt cô lập nhưng không hợp lệ về mặt vận hành. Kết quả như vậy là bất tiện.

### Kiểm thử Cấu hình (Configuration test)

Tại Google, cấu hình dịch vụ web được mô tả trong các tệp lưu trữ trên hệ thống kiểm soát phiên bản. Mỗi tệp cấu hình có một *kiểm thử cấu hình* riêng, xem xét production để xác định cách một binary thực sự được cấu hình và báo cáo các khác biệt so với tệp đó. Những kiểm thử này về bản chất không cô lập, vì chúng vận hành bên ngoài sandbox của hạ tầng kiểm thử.

Kiểm thử cấu hình được xây dựng và chạy cho một phiên bản cụ thể của tệp cấu hình đã check-in. So sánh phiên bản đang vượt qua kiểm thử với phiên bản mục tiêu của tự động hóa cho biết production thực sự đang tụt hậu bao xa so với công việc kỹ thuật đang diễn ra.

Những kiểm thử cấu hình không cô lập này thường đặc biệt hữu ích khi nằm trong giải pháp giám sát phân tán, vì mẫu vượt qua/thất bại xuyên suốt production có thể chỉ ra các đường đi xuyên suốt stack dịch vụ thiếu tổ hợp hợp lý của các cấu hình cục bộ. Các rule của giải pháp giám sát cố khớp các đường của yêu cầu người dùng thực (từ log truy vết) với tập hợp các đường không mong muốn đó. Bất kỳ sự khớp nào mà các rule tìm thấy đều trở thành cảnh báo rằng các release đang diễn ra và/hoặc các push không an toàn, cần hành động sửa chữa.

Kiểm thử cấu hình có thể rất đơn giản khi triển khai production dùng nội dung tệp thực và cung cấp truy vấn thời gian thực để lấy bản sao nội dung. Khi đó, code kiểm thử chỉ cần phát truy vấn và so sánh (diff) phản hồi với tệp. Kiểm thử phức tạp hơn khi cấu hình thực hiện một trong các điều sau:

-   Tích hợp ngầm các giá trị mặc định được xây dựng vào binary (khiến kiểm thử phải được phiên bản hóa riêng biệt)
-   Truyền qua một trình tiền xử lý (preprocessor) như bash thành các cờ dòng lệnh (khiến kiểm thử phải chịu các quy tắc mở rộng)
-   Quy định ngữ cảnh hành vi cho một runtime được chia sẻ (khiến kiểm thử phụ thuộc lịch trình release của runtime đó)

### Kiểm thử Áp lực (Stress test)

Để vận hành an toàn một hệ thống, SRE cần nắm rõ giới hạn của toàn bộ hệ thống cũng như từng thành phần. Trong nhiều trường hợp, các thành phần riêng lẻ không suy giảm nhẹ nhàng (gracefully degrade) khi vượt quá một ngưỡng nhất định, mà thay vào đó sẽ thất bại thảm khốc (catastrophically fail). Kỹ sư sử dụng *stress test* (kiểm thử áp lực) để xác định các giới hạn của một dịch vụ web. Stress test giúp trả lời những câu hỏi như:

-   Một database có thể đầy đến mức nào trước khi các ghi (writes) bắt đầu thất bại?
-   Bao nhiêu truy vấn mỗi giây có thể gửi đến một ứng dụng server trước khi nó quá tải, khiến yêu cầu thất bại?

### Kiểm thử Canary (Canary test)

*Kiểm thử canary* (canary test) vắng mặt một cách nổi bật khỏi danh sách kiểm thử production này. Thuật ngữ *canary* bắt nguồn từ thành ngữ "canary in a coal mine" (chim canary trong mỏ than), ám chỉ việc dùng một con chim sống để phát hiện khí độc trước khi con người bị đầu độc.

Để chạy canary test, hệ thống nâng cấp một tập con các server lên phiên bản hoặc cấu hình mới, sau đó để chúng trong một thời kỳ ấp ủ (incubation period). Nếu không có sai lệch (variance) bất ngờ nào, release tiếp tục và phần còn lại của các server được nâng cấp theo cách tiến bộ.<sup>[4](#fn4)</sup> Nếu có trục trặc, các server đã sửa đổi có thể nhanh chóng được hoàn tác về một trạng thái tốt đã biết. Chúng tôi thường gọi thời kỳ ấp ủ của các server đã nâng cấp là "baking the binary" (nướng binary).

Canary test thực ra không phải là kiểm thử; đúng hơn, nó là chấp nhận người dùng có cấu trúc (structured user acceptance). Trong khi kiểm thử cấu hình và áp lực xác nhận sự tồn tại của một điều kiện cụ thể trên phần mềm tất định (deterministic), canary test mang tính ad hoc hơn. Nó chỉ phơi bày code đang kiểm thử với traffic production thực tế, vốn khó dự đoán hơn nhiều, do đó không hoàn hảo và không phải lúc nào cũng bắt được lỗi mới.

Để lấy ví dụ cụ thể về cách một canary có thể diễn ra: hãy cân nhắc một lỗi cơ bản tương đối hiếm khi ảnh hưởng đến traffic người dùng, đang được triển khai với rollout nâng cấp theo hàm mũ (exponential). Chúng tôi kỳ vọng một số lũy tiến tăng dần của các sai lệch được báo cáo CU = RK, trong đó R là tốc độ của các báo cáo đó, U là bậc (order) của lỗi (định nghĩa sau), và K là khoảng thời gian mà trong đó traffic tăng một hệ số e, tức 172%.<sup>[5](#fn5)</sup>

Để tránh ảnh hưởng đến người dùng, một lần rollout gây ra các sai lệch không mong muốn cần được hoàn tác nhanh chóng về cấu hình trước đó. Trong khoảng thời gian ngắn để hệ thống tự động phát hiện sai lệch và phản ứng, nhiều khả năng sẽ xuất hiện thêm một số báo cáo. Khi mọi chuyện ổn định, những báo cáo này có thể dùng để ước tính cả số lũy tiến C lẫn tốc độ R.

Việc chia và hiệu chỉnh cho K cho ta ước tính của U, bậc của lỗi cơ bản.<sup>[6](#fn6)</sup> Một số ví dụ:

-   U=1: Yêu cầu của người dùng chạm phải code đơn giản là bị hỏng.
-   U=2: Yêu cầu của người dùng này ngẫu nhiên hư hại dữ liệu mà yêu cầu của một người dùng trong tương lai có thể nhìn thấy.
-   U=3: Dữ liệu bị hư hại ngẫu nhiên đó cũng là một định danh hợp lệ cho một yêu cầu trước đó.

Phần lớn bug là bậc một: chúng tăng tuyến tính theo lượng traffic người dùng [[Per07]](https://sre.google/sre-book/bibliography#Per07). Bạn thường có thể truy tìm các bug này bằng cách chuyển log của tất cả yêu cầu có phản hồi bất thường thành kiểm thử regression mới. Chiến lược này không hiệu quả với bug bậc cao hơn: một yêu cầu thất bại khi lặp lại nếu tất cả yêu cầu trước đó được thử theo thứ tự sẽ đột nhiên vượt qua nếu một số yêu cầu bị bỏ qua. Việc bắt được các bug bậc cao hơn này trong lúc release là quan trọng, vì nếu không, khối lượng công việc vận hành có thể tăng rất nhanh.

Hãy luôn ghi nhớ động lực học giữa bug bậc cao và bậc thấp. Khi áp dụng chiến lược rollout theo hàm mũ, bạn không cần cố gắng cân bằng các phân số traffic người dùng. Miễn là mỗi phương pháp thiết lập một phân số đều dùng cùng khoảng K, ước tính của U sẽ hợp lệ, ngay cả khi bạn chưa xác định được phương pháp nào là then chốt để làm sáng tỏ lỗi. Việc dùng nhiều phương pháp tuần tự, cho phép một số chồng chéo, giúp giữ giá trị K ở mức nhỏ. Chiến lược này tối thiểu hóa tổng số sai lệch nhìn thấy được bởi người dùng C, trong khi vẫn cho phép ước tính sớm của U (hy vọng là 1).

## Tạo một Môi trường Kiểm thử và Dựng (Creating a Test and Build Environment)

Lý tưởng nhất là nên nghĩ đến các loại kiểm thử và kịch bản thất bại ngay từ ngày đầu của dự án, nhưng thực tế, SRE thường tham gia đội developer khi dự án đã đang chạy suôn sẻ — chẳng hạn khi dự án của đội xác thực được mô hình nghiên cứu, thư viện chứng minh thuật toán cơ bản có thể scale, hoặc khi tất cả mock giao diện người dùng cuối cùng cũng chấp nhận được. Lúc này, codebase của đội vẫn là nguyên mẫu (prototype) và kiểm thử toàn diện chưa được thiết kế hay triển khai. Trong những tình huống như vậy, nỗ lực kiểm thử của bạn nên bắt đầu từ đâu? Việc thực hiện kiểm thử unit cho mọi function và class là một viễn cảnh hoàn toàn áp đảo nếu bao phủ kiểm thử hiện tại thấp hoặc không tồn tại. Thay vào đó, hãy bắt đầu với những kiểm thử mang lại tác động lớn nhất với ít nỗ lực nhất.

Bạn có thể bắt đầu cách tiếp cận của mình bằng cách đặt các câu hỏi sau:

-   Bạn có thể ưu tiên hóa codebase theo cách nào không? Hãy áp dụng một kỹ thuật từ phát triển tính năng và quản lý dự án: nếu mọi tác vụ đều được coi là ưu tiên cao, thì thực chất không có tác vụ nào là ưu tiên cao. Bạn có thể xếp hạng các thành phần của hệ thống đang kiểm thử dựa trên bất kỳ thước đo tầm quan trọng nào không?
-   Có function hay class nào thực sự quan trọng đối với sứ mệnh hoặc hoạt động kinh doanh không? Chẳng hạn, code liên quan đến tính phí (billing) thường rất quan trọng cho business. Phần code này cũng thường được tách biệt rõ ràng khỏi các thành phần khác của hệ thống.
-   Các API mà các đội khác đang tích hợp (integrate) là gì? Ngay cả loại hỏng hóc không bao giờ vượt qua kiểm thử release để đến người dùng cũng có thể cực kỳ có hại nếu nó khiến một đội developer khác nhầm lẫn, dẫn đến họ viết các client sai (hoặc thậm chí chỉ là không tối ưu) cho API của bạn.

Việc phát hành phần mềm rõ ràng bị lỗi là một trong những “tội lỗi cơ bản” (cardinal sins) của một developer. Việc tạo ra một chuỗi smoke test để chạy cho mỗi release không tốn nhiều công sức. Bước đầu tiên này, với chi phí thấp nhưng tác động cao, có thể dẫn đến phần mềm được kiểm thử kỹ lưỡng và đáng tin cậy.

Một cách để xây dựng văn hóa kiểm thử mạnh mẽ<sup>[7](#fn7)</sup> là bắt đầu ghi tài liệu mọi bug được báo cáo dưới dạng các trường hợp kiểm thử. Nếu mỗi bug đều được chuyển thành một kiểm thử, thì mỗi kiểm thử đó được giả định là ban đầu sẽ thất bại do bug chưa được sửa. Khi kỹ sư sửa các bug, phần mềm sẽ vượt qua kiểm thử và bạn đang tiến đến một bộ kiểm thử regression toàn diện.

Một nhiệm vụ quan trọng khác để tạo ra phần mềm được kiểm thử tốt là xây dựng hạ tầng kiểm thử. Nền tảng của một hệ thống kiểm thử vững chắc là hệ thống kiểm soát phiên bản (versioned source control) theo dõi mọi thay đổi trong codebase.

Khi đã có source control, bạn có thể bổ sung một hệ thống dựng liên tục (continuous build) để tự động dựng phần mềm và chạy kiểm thử mỗi khi có code được đệ trình. Chúng tôi nhận thấy hiệu quả nhất là hệ thống dựng phải thông báo cho kỹ sư ngay lập tức khi một thay đổi làm hỏng dự án phần mềm. Mặc dù điều này nghe có vẻ hiển nhiên, nhưng yêu cầu cốt lõi là phiên bản mới nhất của dự án phần mềm trong source control phải hoạt động hoàn toàn. Khi hệ thống dựng phát hiện code bị lỗi, kỹ sư cần tạm dừng mọi tác vụ khác để ưu tiên xử lý vấn đề. Việc coi trọng các khiếm khuyết là cần thiết vì một số lý do:

-   Thường khó sửa hơn nếu có các thay đổi đối với codebase sau khi khiếm khuyết được giới thiệu.
-   Phần mềm bị hỏng làm chậm đội vì họ phải làm việc quanh sự hỏng hóc.
-   Nhịp độ release, như build hàng đêm và hàng tuần, mất giá trị.
-   Khả năng của đội phản ứng với yêu cầu release khẩn cấp (ví dụ, để đáp lại một tiết lộ lỗ hổng bảo mật) trở nên phức tạp và khó khăn hơn nhiều.

Trong thế giới SRE, hai khái niệm ổn định và linh hoạt (agility) truyền thống luôn tồn tại mâu thuẫn. Gạch đầu dòng cuối cùng chỉ ra một trường hợp thú vị: sự ổn định thực sự lại thúc đẩy sự linh hoạt. Khi build được dự đoán là vững chắc và đáng tin cậy, developer có thể lặp lại (iterate) nhanh hơn!

Một số hệ thống build như Bazel<sup>[8](#fn8)</sup> có các tính năng cho phép kiểm soát chính xác hơn đối với kiểm thử. Ví dụ, Bazel tạo ra đồ thị phụ thuộc (dependency graph) cho các dự án phần mềm. Khi một thay đổi được thực hiện trên một tệp, Bazel chỉ dựng lại phần phần mềm phụ thuộc vào tệp đó. Những hệ thống như vậy cung cấp build có thể tái lập (reproducible). Thay vì chạy tất cả kiểm thử ở mỗi lần submit, kiểm thử chỉ chạy cho code đã thay đổi. Kết quả, kiểm thử chạy rẻ hơn và nhanh hơn.

Một số công cụ hỗ trợ định lượng và trực quan hóa mức bao phủ kiểm thử cần thiết [[Cra10]](https://sre.google/sre-book/bibliography#Cra10). Hãy dùng chúng để xác định trọng tâm kiểm thử: việc tạo ra code có độ bao phủ kiểm thử cao nên được xem là một dự án kỹ thuật, chứ không phải bài tập mang tính triết lý hay tâm lý học. Thay vì lặp lại điệp khúc mơ hồ "Chúng tôi cần nhiều kiểm thử hơn", hãy đặt ra mục tiêu và hạn chót cụ thể.

Cần nhớ rằng không phải phần mềm nào cũng ngang hàng. Các hệ thống quan trọng cho sự sống (life-critical) hoặc quan trọng cho doanh thu (revenue-critical) đòi hỏi mức chất lượng và bao phủ kiểm thử cao hơn đáng kể so với một script nonproduction có vòng đời ngắn (short shelf life).

## Kiểm thử ở Quy mô (Testing at Scale)

Sau khi đã xem xét các nền tảng của kiểm thử, chúng ta hãy cùng tìm hiểu cách SRE tiếp cận kiểm thử từ góc nhìn hệ thống nhằm thúc đẩy độ tin cậy ở quy mô lớn.

Một kiểm thử unit nhỏ thường chỉ phụ thuộc vào một danh sách ngắn các thành phần: tệp nguồn, thư viện kiểm thử, các thư viện runtime, trình biên dịch (compiler) và phần cứng cục bộ chạy kiểm thử. Để có môi trường kiểm thử vững chắc, mỗi phụ thuộc này cần có bao phủ kiểm thử riêng, trong đó các kiểm thử đặc thù phải xử lý những use case mà các phần khác của môi trường kỳ vọng. Nếu việc cài đặt kiểm thử unit đó phụ thuộc vào một đường code bên trong thư viện runtime mà không có bao phủ kiểm thử, một thay đổi không liên quan trong môi trường<sup>[9](#fn9)</sup> có thể khiến kiểm thử unit luôn vượt qua, bất kể code đang kiểm thử có lỗi hay không.

Ngược lại, một kiểm thử release có thể phụ thuộc vào nhiều phần đến mức phát sinh sự phụ thuộc truyền đạt (transitive dependency) vào mọi object trong kho code. Nếu kiểm thử này dựa trên một bản sao sạch của môi trường production, về nguyên tắc, mọi bản vá (patch) nhỏ đều đòi hỏi một vòng lặp phục hồi thảm họa (disaster recovery) đầy đủ. Các môi trường kiểm thử thực tế thường chọn các điểm phân nhánh (branch point) giữa các phiên bản và các merge. Cách này giúp giải quyết phần lớn sự không chắc chắn phụ thuộc với số vòng lặp tối thiểu. Tất nhiên, khi một vùng không chắc chắn được xác định là lỗi, bạn cần chọn thêm các điểm phân nhánh.

<a id="kiem-thu-cac-cong-cu-scale-duoc"></a>

## Kiểm thử Các Công cụ Scale được (Testing Scalable Tools)

Với tư cách là các mảnh phần mềm, công cụ SRE cũng cần kiểm thử.<sup>[10](#fn10)</sup> Công cụ do SRE phát triển có thể thực hiện các tác vụ như:

-   Lấy và lan truyền các metrics hiệu năng database
-   Dự đoán các metrics sử dụng để lập kế hoạch cho rủi ro năng lực
-   Refactor dữ liệu bên trong một replica dịch vụ không thể truy cập bởi người dùng
-   Thay đổi các tệp trên một server

Các công cụ SRE chia sẻ hai đặc tính:

-   Các tác dụng phụ (side effects) của chúng nằm trong phạm vi API mainstream (chủ lưu) đã được kiểm thử
-   Chúng được cô lập khỏi production hướng người dùng bởi một rào chắn xác thực và release hiện có

### Các Phòng thủ Rào chắn Chống lại Phần mềm Rủi ro (Barrier Defenses Against Risky Software)

Phần mềm vượt qua API thông thường, dù đã được kiểm thử kỹ (ngay cả khi vì lý do tốt), vẫn có thể gây hỗn loạn cho một dịch vụ đang chạy. Ví dụ, một engine database có thể cho phép quản trị viên tạm thời tắt các giao dịch (transaction) để rút ngắn cửa sổ bảo trì. Nếu engine này phục vụ cho phần mềm cập nhật theo lô (batch update), sự cô lập hướng người dùng có thể bị mất nếu tiện ích đó tình cờ được khởi động trên một replica hướng người dùng. Tránh rủi ro này bằng thiết kế:

1.  Dùng một công cụ riêng để đặt rào chắn trong cấu hình nhân bản (replication), ngăn replica vượt qua kiểm tra sức khỏe (health check). Do đó, replica không được release đến người dùng.
2.  Cấu hình phần mềm rủi ro để kiểm tra rào chắn khi khởi động. Cho phép phần mềm rủi ro chỉ truy cập các replica không khỏe mạnh.
3.  Dùng công cụ xác thực sức khỏe replica (cùng dùng cho giám sát hộp đen) để loại bỏ rào chắn.

Công cụ tự động hóa cũng là phần mềm. Vì dấu chân rủi ro của chúng xuất hiện out-of-band (ngoài băng) cho một tầng khác của dịch vụ, nên nhu cầu kiểm thử của chúng tinh tế hơn. Công cụ tự động hóa thực hiện các tác vụ như:

-   Chọn index (chỉ mục) database
-   Cân bằng tải (load balancing) giữa các datacenter
-   Xáo trộn (shuffle) các log relay cho việc remastering (làm chủ lại) nhanh

Các công cụ tự động hóa chia sẻ hai đặc tính:

-   Thao tác thực tế được thực hiện là đối với một API vững chắc, có thể dự đoán, và được kiểm thử kỹ
-   Mục đích thật sự của thao tác nằm ở tác dụng phụ của nó — một sự gián đoạn vô hình đối với một client API khác

Kiểm thử có thể minh họa hành vi mong muốn của các tầng dịch vụ khác, cả trước lẫn sau thay đổi. Thường có thể kiểm tra xem trạng thái nội bộ, khi nhìn qua API, có được giữ nguyên xuyên suốt thao tác hay không. Ví dụ, database phải luôn trả về câu trả lời đúng, ngay cả khi một index phù hợp không khả dụng cho truy vấn. Ngược lại, một số bất biến (invariant) API được ghi tài liệu (như cache DNS giữ cho đến khi hết TTL) có thể không được duy trì xuyên suốt thao tác. Chẳng hạn, nếu một thay đổi runlevel thay thế nameserver cục bộ bằng một caching proxy, cả hai lựa chọn đều có thể cam kết giữ các tra cứu đã hoàn thành trong nhiều giây, nhưng khả năng cao là trạng thái cache không được bàn giao từ cái này sang cái kia.

Giả sử công cụ tự động hóa ngụ ý việc chạy thêm các kiểm thử release cho các binary khác nhằm xử lý các sự cố nhất thời của môi trường, thì bạn định nghĩa môi trường chạy của chúng như thế nào? Rốt cuộc, tự động hóa cho việc xáo trộn container để cải thiện việc sử dụng nhiều khả năng sẽ cố xáo trộn chính nó vào một lúc nào đó nếu nó cũng chạy trong một container. Sẽ là điều đáng xấu hổ nếu một release mới của thuật toán nội bộ tạo ra các trang bộ nhớ bẩn (dirty memory page) nhanh đến mức băng thông mạng của việc mirror liên quan cuối cùng ngăn code hoàn tất di chuyển đang chạy (live migration). Ngay cả khi có kiểm thử tích hợp mà binary cố ý xáo trộn chính nó, kiểm thử nhiều khả năng không dùng mô hình hạm đội container có kích thước production. Gần như chắc chắn nó không được phép dùng băng thông liên châu lục khan hiếm, độ trễ cao để kiểm thử những cuộc đua như vậy.

Thú vị hơn, một công cụ tự động hóa có thể đang thay đổi môi trường mà một công cụ tự động hóa khác chạy. Hoặc cả hai có thể đang thay đổi môi trường của nhau đồng thời! Ví dụ, công cụ nâng cấp hạm đội nhiều khả năng tiêu thụ nhiều tài nguyên nhất khi push các nâng cấp. Kết quả, việc cân bằng lại container sẽ bị cám dỗ để di chuyển công cụ. Đổi lại, công cụ cân bằng lại container thỉnh thoảng cần nâng cấp. Sự phụ thuộc tuần hoàn này không thành vấn đề nếu các API liên quan có ngữ nghĩa khởi động lại (restart semantics), ai đó nhớ cài bao phủ kiểm thử cho các ngữ nghĩa đó, và sức khỏe checkpoint được đảm bảo độc lập.

## Kiểm thử Thảm họa (Testing Disaster)

Nhiều công cụ phục hồi thảm họa có thể được thiết kế cẩn thận để vận hành *offline* (ngoại tuyến). Chúng thực hiện:

-   Tính toán một trạng thái *checkpoint* tương đương với việc dừng dịch vụ sạch sẽ
-   Push trạng thái checkpoint để *tải được* (loadable) bởi các công cụ xác thực phi thảm họa hiện có
-   Hỗ trợ các công cụ *rào chắn* release thông thường, kích hoạt quy trình *khởi động sạch* (clean start)

Trong nhiều trường hợp, bạn có thể thiết lập các giai đoạn này để việc viết kiểm thử liên quan trở nên dễ dàng hơn và đạt độ bao phủ tốt. Nếu bất kỳ ràng buộc nào (offline, checkpoint, tải được, rào chắn, hoặc khởi động sạch) bị vi phạm, việc chứng minh rằng cài đặt công cụ liên quan sẽ hoạt động khi có thông báo ngắn sẽ khó khăn hơn rất nhiều.

Công cụ sửa chữa online về bản chất hoạt động bên ngoài API chính (mainstream), nên việc kiểm thử chúng cũng thú vị hơn. Trong hệ thống phân tán, một thách thức là xác định xem hành vi bình thường vốn mang tính nhất quán theo sự kiện (eventual consistency) có tương tác xấu với quá trình sửa chữa hay không. Ví dụ, hãy xét một race condition mà bạn có thể cố phân tích bằng công cụ offline. Các công cụ offline thường được viết dựa trên giả định về sự nhất quán tức thì (instant consistency) thay vì nhất quán theo sự kiện, vì nhất quán tức thì dễ kiểm thử hơn. Tình huống này trở nên phức tạp do binary sửa chữa thường được biên dịch riêng biệt so với binary production mà nó đang đua. Do đó, bạn có thể cần biên dịch một binary thống nhất có đo lường (instrumented) để chạy trong các bài kiểm thử này, nhằm giúp công cụ quan sát được các giao dịch.

### Sử dụng Các Kiểm thử Thống kê (Using Statistical Tests)

Các kỹ thuật thống kê như Lemon [[Ana07]](https://sre.google/sre-book/bibliography#Ana07) dùng cho fuzzing, hay Chaos Monkey<sup>[11](#fn11)</sup> và Jepsen<sup>[12](#fn12)</sup> dùng cho trạng thái phân tán, không nhất thiết là kiểm thử có thể lặp lại. Việc chỉ đơn thuần chạy lại các kiểm thử đó sau một thay đổi code không chứng minh chắc chắn rằng lỗi quan sát được đã được sửa.<sup>[13](#fn13)</sup> Tuy nhiên, chúng có thể hữu ích:

-   Ghi log tất cả hành động được chọn ngẫu nhiên trong một lần chạy nhất định — đôi khi chỉ cần ghi log hạt (seed) của trình tạo số ngẫu nhiên.
-   Nếu log này được refactor ngay thành kiểm thử release, việc chạy vài lần trước khi viết báo cáo bug thường hữu ích. Tốc độ không-thất bại khi phát lại (replay) cho bạn biết sẽ khó đến mức nào để sau đó khẳng định lỗi đã được sửa.
-   Các biến thể trong cách lỗi được biểu đạt giúp bạn định vị chính xác các vùng đáng ngờ trong code.
-   Một số lần chạy sau đó có thể cho thấy tình huống thất bại nghiêm trọng hơn so với lần chạy ban đầu. Khi đó, bạn có thể muốn leo thang mức nghiêm trọng và tác động của bug.

## Nhu cầu về Tốc độ (The Need for Speed)

Với mỗi phiên bản (patch) trong kho code, mỗi kiểm thử sẽ cho ra một chỉ thị vượt qua hoặc thất bại. Chỉ thị này có thể thay đổi giữa các lần chạy lặp lại trông giống hệt nhau. Bạn có thể ước tính xác suất thực tế mà một kiểm thử vượt qua hay thất bại bằng cách lấy trung bình trên các lần chạy đó và tính sự không chắc chắn thống kê của khả năng này. Tuy nhiên, thực hiện phép tính này cho mỗi kiểm thử ở mỗi điểm phiên bản là không thể về mặt tính toán.

Thay vào đó, bạn cần xây dựng các giả thuyết cho nhiều kịch bản quan tâm và chạy đủ số lần lặp lại cho từng kiểm thử cũng như từng phiên bản để có cơ sở suy luận hợp lý. Trong số đó, một số kịch bản vô hại về mặt chất lượng code, còn những kịch bản khác lại có thể xử lý được. Các kịch bản này tác động đến mọi nỗ lực kiểm thử ở những mức độ khác nhau; do chúng có mối liên kết (coupled) với nhau, việc nhanh chóng thu thập một danh sách tin cậy các giả thuyết có thể xử lý (tức các thành phần thực sự bị hỏng) đồng nghĩa với việc phải ước tính tất cả các kịch bản cùng lúc.

Kỹ sư dùng hạ tầng kiểm thử cần biết code của mình — thường chỉ chiếm một phần nhỏ trong tổng lượng nguồn của một lần chạy — có bị lỗi không. Nếu code không lỗi, mọi thất bại quan sát được thường do code của người khác gây ra. Nói cách khác, họ muốn xác định xem code của mình có chứa race condition bất ngờ khiến kiểm thử flaky (bất ổn) — hoặc flaky hơn so với mức vốn đã flaky do các yếu tố khác — hay không.

### Các Hạn chót Kiểm thử (Testing Deadlines)

Phần lớn kiểm thử khá đơn giản, theo nghĩa chúng chạy như một binary cô lập, tự chứa (self-contained hermetic), hoàn tất trong vài giây trên một container tính toán nhỏ. Những kiểm thử này giúp kỹ sư nhận phản hồi tương tác về các sai lầm trước khi chuyển ngữ cảnh (context) sang bug hay tác vụ tiếp theo.

Các kiểm thử đòi hỏi điều phối (orchestration) xuyên suốt nhiều binary và/hoặc hạm đội nhiều container thường mất vài giây để khởi động. Do khó có phản hồi tương tác, chúng thường được xếp vào nhóm kiểm thử lô (batch). Thay vì nhắc kỹ sư "đừng đóng tab trình soạn thảo", những thất bại kiểm thử này đang gửi tín hiệu đến người xem xét code rằng: "code này chưa sẵn sàng để xem xét".

Hạn chót không chính thức cho kiểm thử là thời điểm kỹ sư chuyển sang ngữ cảnh tiếp theo. Kết quả kiểm thử cần được cung cấp trước khi chuyển ngữ cảnh, nếu không, ngữ cảnh tiếp theo có thể liên quan đến việc biên dịch XKCD.<sup>[14](#fn14)</sup>

Giả sử một kỹ sư đang làm việc trên dịch vụ có hơn 21.000 kiểm thử đơn giản và thỉnh thoảng đề xuất một bản vá cho codebase. Để kiểm thử bản vá, bạn muốn so sánh vector các kết quả vượt qua/thất bại từ codebase trước bản vá với vector từ codebase sau bản vá. Nếu hai vector này tương đồng, codebase tạm thời đủ điều kiện để release. Sự đủ điều kiện này tạo động lực để chạy nhiều kiểm thử release và tích hợp, cũng như các kiểm thử binary phân tán khác nhằm xem xét việc scale của hệ thống (nếu bản vá dùng nhiều tài nguyên tính toán cục bộ hơn đáng kể) và độ phức tạp (nếu bản vá tạo ra khối lượng công việc siêu tuyến tính ở nơi khác).

Với tần suất nào thì việc bạn tính sai độ flaky của môi trường sẽ khiến một bản vá vô hại của người dùng bị gắn cờ nhầm là có hại? Có vẻ người dùng sẽ phàn nàn gay gắt nếu 1 trong 10 bản vá bị từ chối. Nhưng việc từ chối 1 bản vá giữa 100 bản vá hoàn hảo có thể đi qua mà không ai bình luận.

Điều này có nghĩa là bạn cần quan tâm đến căn bậc 42.000 (một cho mỗi kiểm thử được định nghĩa trước bản vá, và một cho mỗi kiểm thử sau bản vá) của 0.99 (tỷ lệ các bản vá được chấp nhận). Phép tính này:

```
0.99^(1/42000) ≈ 0.9999995
```

gợi ý rằng những kiểm thử cá nhân đó phải chạy đúng hơn 99.9999% thời gian. Chà.

## Push đến Production (Pushing to Production)

Dù cấu hình production thường được quản lý trong kho source control, nó vẫn tách biệt khỏi code nguồn. Tương tự, hạ tầng kiểm thử phần mềm thường không truy cập được cấu hình production. Ngay cả khi cả hai cùng nằm trong một kho, các thay đổi cấu hình thường diễn ra trên các nhánh (branch) và/hoặc trong cây thư mục riêng, vốn đã bị các quy trình tự động hóa kiểm thử bỏ qua từ lâu.

Trong môi trường doanh nghiệp kế thừa (legacy), nơi kỹ sư phần mềm phát triển các binary rồi ném chúng qua bức tường đến quản trị viên cập nhật server, sự tách biệt giữa hạ tầng kiểm thử và cấu hình production tốt nhất cũng phiền toái, tệ nhất có thể làm hỏng độ tin cậy và sự linh hoạt. Sự tách biệt như vậy cũng có thể dẫn đến trùng lặp công cụ. Trong môi trường Ops (Vận hành) tích hợp danh nghĩa, sự tách biệt này suy giảm khả năng chống chịu (resilience) vì nó tạo ra các bất nhất quán tinh tế giữa hành vi của hai tập công cụ. Nó cũng giới hạn tốc độ dự án vì các cuộc đua commit giữa các hệ thống phiên bản hóa.

Trong mô hình SRE, việc tách biệt hạ tầng kiểm thử khỏi cấu hình production gây tác động tiêu cực hơn đáng kể, vì nó cắt đứt mối liên hệ giữa mô hình mô tả production và mô hình mô tả hành vi ứng dụng. Sự khác biệt này ảnh hưởng đến kỹ sư khi họ tìm kiếm các bất nhất quán thống kê trong các kỳ vọng tại thời điểm phát triển. Tuy nhiên, sự tách biệt này không làm chậm phát triển cho bằng là ngăn kiến trúc hệ thống thay đổi, bởi không có cách nào loại bỏ rủi ro di cư (migration).

Hãy cân nhắc một kịch bản của việc phiên bản hóa thống nhất và kiểm thử thống nhất, để phương pháp luận SRE áp dụng được. Sự thất bại của một cuộc di cư kiến trúc phân tán sẽ gây tác động gì? Một lượng lớn kiểm thử nhiều khả năng sẽ diễn ra. Cho đến nay, đã giả định rằng kỹ sư phần mềm nhiều khả năng chấp nhận hệ thống kiểm thử trả lời sai 1 trong khoảng 10 lần. Bạn sẵn sàng chấp nhận rủi ro nào với cuộc di cư nếu biết kiểm thử có thể trả về âm tính giả (false negative) và tình hình có thể trở nên thực sự thú vị, thực sự nhanh chóng? Rõ ràng, một số khu vực bao phủ kiểm thử cần mức độ đa nghi (paranoia) cao hơn. Sự phân biệt này có thể tổng quát hóa: một số thất bại kiểm thử cho thấy rủi ro tác động lớn hơn so với các thất bại khác.

## Kỳ vọng Kiểm thử Thất bại (Expect Testing Fail)

Trước đây, một sản phẩm phần mềm có thể chỉ phát hành một lần mỗi năm. Các binary được tạo ra bởi một chuỗi công cụ biên dịch (compiler toolchain) trong nhiều giờ hoặc nhiều ngày, và phần lớn việc kiểm thử do con người thực hiện theo các hướng dẫn viết tay. Quy trình phát hành này kém hiệu quả, nhưng nhu cầu tự động hóa lại không cao. Công việc phát hành bị chi phối bởi tài liệu, di cư dữ liệu, đào tạo lại người dùng và các yếu tố khác. MTBF cho những lần phát hành đó là một năm, bất kể có bao nhiêu kiểm thử được thực hiện. Mỗi lần phát hành có quá nhiều thay đổi đến mức chắc chắn sẽ có một số lỗi mà người dùng nhìn thấy được ẩn trong phần mềm. Về mặt hiệu quả, dữ liệu độ tin cậy từ lần phát hành trước không liên quan đến lần phát hành tiếp theo.

Các công cụ quản lý API/ABI hiệu quả và các ngôn ngữ được giải thích (interpreted) scale đến lượng lớn code giờ đây hỗ trợ việc dựng và chạy một phiên bản phần mềm mới mỗi vài phút. Về nguyên tắc, một đội quân đủ lớn con người<sup>[15](#fn15)</sup> có thể hoàn thành kiểm thử trên mỗi phiên bản mới bằng các phương pháp đã mô tả và đạt được cùng thanh chuẩn chất lượng cho mỗi phiên bản gia tăng. Ngay cả khi cuối cùng chỉ cùng các kiểm thử áp dụng cho cùng code, phiên bản phần mềm cuối cùng có chất lượng cao hơn trong release được phát hành hàng năm. Điều này là vì ngoài các phiên bản hàng năm, các phiên bản trung gian của code cũng được kiểm thử. Dùng các phiên bản trung gian, bạn có thể ánh xạ không mơ hồ các vấn đề tìm thấy khi kiểm thử về các nguyên nhân gốc của chúng, và tự tin rằng toàn bộ vấn đề — không chỉ triệu chứng hạn chế được phơi bày — đã được sửa. Nguyên lý của chu kỳ phản hồi ngắn này cũng hiệu quả như nhau khi áp dụng cho bao phủ kiểm thử tự động.

Khi để người dùng thử nhiều phiên bản phần mềm hơn trong năm, MTBF sẽ bị ảnh hưởng do người dùng có nhiều cơ hội hơn để phát hiện lỗi. Tuy nhiên, bạn cũng có thể xác định được các khu vực cần bổ sung bao phủ kiểm thử. Nếu cài đặt các kiểm thử này, mỗi cải tiến sẽ giúp ngăn chặn một số thất bại trong tương lai. Việc quản lý độ tin cậy cẩn thận sẽ kết hợp các giới hạn về sự không chắc chắn do bao phủ kiểm thử với các giới hạn về lỗi mà người dùng nhìn thấy để điều chỉnh nhịp độ release. Sự kết hợp này tối đa hóa kiến thức bạn thu được từ vận hành và người dùng cuối. Những lợi ích này thúc đẩy bao phủ kiểm thử và, theo đó, tốc độ release sản phẩm.

Nếu một SRE sửa đổi một tệp cấu hình hoặc điều chỉnh chiến lược của một công cụ tự động hóa (thay vì cài đặt một tính năng người dùng), công việc kỹ thuật khớp với cùng mô hình khái niệm. Khi định nghĩa nhịp độ release dựa trên độ tin cậy, thường có ý nghĩa để phân đoạn ngân sách độ tin cậy theo chức năng, hoặc (thuận tiện hơn) theo đội. Trong kịch bản đó, đội kỹ thuật tính năng nhắm đến việc đạt được một giới hạn không chắc chắn nhất định ảnh hưởng đến nhịp độ release mục tiêu. Đội SRE có ngân sách riêng với sự không chắc chắn liên quan riêng, và do đó một giới hạn trên cho tốc độ release của họ.

Để duy trì độ tin cậy và tránh phải scale số lượng SRE hỗ trợ một dịch vụ theo tuyến tính, môi trường production phải chạy phần lớn không có người giám sát (unattended). Để duy trì được điều đó, môi trường phải chống chịu với các lỗi nhỏ. Khi một sự kiện lớn đòi hỏi can thiệp thủ công của SRE xảy ra, các công cụ SRE dùng phải được kiểm thử phù hợp. Nếu không, sự can thiệp đó làm giảm sự tin tưởng rằng dữ liệu lịch sử còn áp dụng được cho tương lai gần. Sự giảm tin tưởng buộc phải chờ phân tích dữ liệu giám sát để loại bỏ sự không chắc chắn phát sinh. Trong khi thảo luận trước đó trong [Kiểm thử Các Công cụ Scale được](#kiem-thu-cac-cong-cu-scale-duoc) tập trung vào cách tận dụng cơ hội bao phủ kiểm thử cho một công cụ SRE, ở đây bạn thấy kiểm thử xác định bao lâu thì phù hợp để dùng công cụ đó trên production.

Tệp cấu hình nhìn chung tồn tại vì việc thay đổi cấu hình nhanh hơn dựng lại một công cụ. Độ trễ thấp này thường là một yếu tố giữ MTTR thấp. Tuy nhiên, những tệp đó cũng được thay đổi thường xuyên vì các lý do không cần độ trễ giảm. Khi nhìn từ quan điểm của độ tin cậy:

-   Tệp cấu hình tồn tại để giữ MTTR thấp và chỉ được sửa đổi khi có sự cố, nên nhịp độ release của nó chậm hơn MTBF. Có thể tồn tại một mức độ không chắc chắn đáng kể về việc một sửa đổi thủ công cụ thể có thực sự tối ưu hay không, mà không làm ảnh hưởng đến độ tin cậy tổng thể của site.
-   Tệp cấu hình thay đổi nhiều hơn một lần mỗi release ứng dụng hướng người dùng (ví dụ, vì nó giữ trạng thái release) có thể là rủi ro lớn nếu những thay đổi này không được đối xử như các release ứng dụng. Nếu bao phủ kiểm thử và giám sát của tệp đó không tốt hơn đáng kể so với ứng dụng người dùng, tệp đó sẽ chi phối độ tin cậy site theo cách tiêu cực.

Một cách xử lý tệp cấu hình là phân loại mỗi tệp vào đúng một trong các tùy chọn trong danh sách trước, đồng thời thực thi quy tắc này theo một cách nào đó. Nếu chọn chiến lược sau, hãy đảm bảo:

-   Mỗi tệp cấu hình có đủ bao phủ kiểm thử để hỗ trợ việc chỉnh sửa thường quy.
-   Trước các release, việc sửa tệp bị trì hoãn một chút trong khi chờ kiểm thử release.
-   Cung cấp cơ chế phá kính (break-glass) để push tệp lên trực tuyến trước khi hoàn thành kiểm thử. Vì phá kính làm suy giảm độ tin cậy, tốt hơn nên để sự phá vỡ này "ồn ào" bằng cách (ví dụ) đệ trình một bug yêu cầu giải pháp vững chắc hơn cho lần sau.

### Phá kính (Break-Glass) và Kiểm thử

Bạn có thể cài đặt một cơ chế phá kính để vô hiệu hóa kiểm thử release. Điều này nghĩa là bất kỳ ai thực hiện sửa thủ công vội vàng sẽ không được báo về bất kỳ sai lầm nào cho đến khi tác động người dùng thực được giám sát báo cáo. Tốt hơn là để các kiểm thử tiếp tục chạy, liên kết sự kiện push sớm với sự kiện kiểm thử đang chờ, và (nhanh nhất có thể) chú thích ngược (back-annotate) push với bất kỳ kiểm thử nào bị hỏng. Bằng cách này, một push thủ công khiếm khuyết có thể nhanh chóng được theo sau bởi một push thủ công khác (hy vọng ít khiếm khuyết hơn). Lý tưởng, cơ chế phá kính đó tự động tăng ưu tiên của các kiểm thử release để chúng có thể tiền xử lý (preempt) khối lượng công việc xác thực và bao phủ gia tăng thường quy mà hạ tầng kiểm thử đang xử lý.

## Tích hợp (Integration)

Ngoài việc kiểm thử unit từng tệp cấu hình để giảm thiểu rủi ro cho độ tin cậy, cũng cần xem xét kiểm thử tích hợp. Nội dung tệp cấu hình, xét về mặt kiểm thử, có thể mang tính thù địch (hostile) đối với trình giải thích (interpreter) đọc cấu hình. Các ngôn ngữ được giải thích như Python thường được dùng cho tệp cấu hình vì trình giải thích của chúng có thể nhúng (embedded), đồng thời có một số cơ chế sandboxing đơn giản để bảo vệ khỏi các lỗi code không ác ý.

Việc dùng ngôn ngữ được giải thích để viết tệp cấu hình tiềm ẩn rủi ro, bởi cách tiếp cận này chứa đầy các lỗi khó xử lý dứt điểm. Bản chất của việc tải nội dung là thực thi một chương trình, nên không có giới hạn trên về mức độ kém hiệu quả mà thao tác này có thể gây ra. Ngoài các kiểm thử khác, bạn nên kết hợp loại kiểm thử tích hợp này với việc đặt thời gian chờ (deadline) cẩn thận cho tất cả các phương pháp kiểm thử tích hợp, nhằm đánh dấu là thất bại những kiểm thử không chạy đến hoàn tất trong khoảng thời gian hợp lý.

Nếu cấu hình được viết dưới dạng văn bản theo một cú pháp (syntax) tùy chỉnh, mỗi danh mục kiểm thử sẽ phải bao phủ riêng biệt từ đầu. Việc dùng một cú pháp hiện có như YAML, kết hợp với một trình phân tích (parser) đã được kiểm thử kỹ như `safe_load` của Python, giúp loại bỏ một số công việc thủ công (toil) phát sinh từ tệp cấu hình. Chọn kỹ cú pháp và trình phân tích có thể đảm bảo một giới hạn trên cứng về thời gian thao tác tải. Tuy nhiên, người cài đặt vẫn phải xử lý các lỗi schema, và phần lớn các chiến lược đơn giản để làm điều đó không có giới hạn trên về thời gian thực thi. Tồi tệ hơn, những chiến lược này thường không được kiểm thử unit vững chắc.

Lợi ích của việc dùng protocol buffers<sup>[16](#fn16)</sup> là schema được định nghĩa trước và tự động kiểm tra tại thời điểm tải, giúp loại bỏ thêm nhiều toil, trong khi vẫn đảm bảo thời gian thực thi có giới hạn.

Vai trò của SRE nhìn chung bao gồm việc viết các công cụ kỹ thuật hệ thống<sup>[17](#fn17)</sup> (nếu chưa ai khác đang làm việc này) và bổ sung xác thực vững chắc cùng bao phủ kiểm thử. Vì mọi công cụ đều có thể hành vi bất ngờ do các bug không được kiểm thử bắt được, nên phòng thủ nhiều lớp (defense in depth) là cần thiết. Khi một công cụ hành vi bất ngờ, kỹ sư cần tự tin nhất có thể rằng phần lớn công cụ khác của họ đang hoạt động đúng, từ đó có thể giảm nhẹ hoặc giải quyết tác dụng phụ của sự sai lầm đó. Một yếu tố chính của việc cung cấp độ tin cậy site là tìm ra mỗi dạng sai lầm được dự kiến và đảm bảo một kiểm thử nào đó (hoặc bộ xác thực input được kiểm thử của công cụ khác) báo cáo nó. Công cụ tìm ra vấn đề có thể không thể sửa hoặc thậm chí dừng nó, nhưng nên ít nhất báo cáo vấn đề trước khi một outage thảm khốc xảy ra.

Ví dụ, hãy xem xét danh sách cấu hình của tất cả người dùng (như */etc/passwd* trên một máy Unix không có mạng) và hình dung một lỗi khiến trình phân tích vô tình dừng lại sau khi chỉ xử lý được một nửa tệp. Do những người dùng mới tạo gần đây chưa được nạp, hệ thống nhiều khả năng vẫn chạy bình thường và nhiều người dùng có thể không nhận ra sự cố. Công cụ quản lý các thư mục home có thể dễ dàng phát hiện sự không khớp giữa các thư mục thực tế và các thư mục được suy ra từ danh sách người dùng (vốn được duy trì tách biệt), từ đó khẩn cấp báo cáo sự khác biệt này. Giá trị của công cụ nằm ở việc báo cáo vấn đề, và nó nên tránh tự động khắc phục (bằng cách xóa rất nhiều dữ liệu người dùng).

## Các Mồi Production (Production Probes)

Giả định rằng kiểm thử quy định hành vi chấp nhận được trước dữ liệu đã biết, trong khi giám sát xác nhận hành vi chấp nhận được trước dữ liệu người dùng chưa biết, thì sự kết hợp của kiểm thử và giám sát dường như bao phủ cả hai nguồn rủi ro chính, dù đã biết hay chưa biết. Thật không may, rủi ro thực tế phức tạp hơn.

Các yêu cầu tốt đã biết nên hoạt động, trong khi các yêu cầu xấu đã biết nên lỗi. Cài đặt cả hai loại bao phủ như một kiểm thử tích hợp nhìn chung là ý tưởng tốt. Bạn có thể phát lại (replay) cùng ngân hàng các yêu cầu kiểm thử như một kiểm thử release. Việc chia các yêu cầu tốt đã biết thành những cái có thể phát lại đối với production và những cái không thể tạo ra ba tập yêu cầu:

-   Các yêu cầu xấu đã biết
-   Các yêu cầu tốt đã biết có thể được phát lại đối với production
-   Các yêu cầu tốt đã biết không thể được phát lại đối với production

Bạn có thể dùng mỗi tập như cả kiểm thử tích hợp lẫn release. Phần lớn các kiểm thử này cũng có thể dùng như mồi giám sát (monitoring probe).

Triển khai giám sát như vậy có vẻ thừa và, về nguyên tắc, vô nghĩa, bởi chính xác các yêu cầu đó đã được thử theo hai cách khác. Tuy nhiên, hai cách này khác nhau vì một số lý do:

-   Kiểm thử release nhiều khả năng đã bọc server tích hợp với một frontend và một backend giả (fake backend).
-   Kiểm thử mồi nhiều khả năng đã bọc binary release với một frontend cân bằng tải và một backend bền (persistent) scale được riêng biệt.
-   Các frontend và backend nhiều khả năng có các chu kỳ release độc lập, với lịch trình diễn ra ở các tốc độ khác nhau (do nhịp độ release thích nghi của chúng).

Vì vậy, mồi giám sát đang chạy trong production là một cấu hình chưa được kiểm thử trước đó.

Những mồi đó không bao giờ nên thất bại, nhưng nếu chúng thất bại thì có nghĩa là gì? Hoặc API frontend (từ load balancer) hoặc API backend (đến kho lưu trữ bền) không tương đương giữa môi trường production và release. Trừ khi bạn đã biết lý do các môi trường production và release không tương đương, site nhiều khả năng bị hỏng.

Cùng trình cập nhật production đang dần dần thay thế ứng dụng cũng dần dần thay thế các mồi, để tất cả bốn tổ hợp của mồi cũ-hay-mới gửi yêu cầu đến ứng dụng cũ-hay-mới được liên tục tạo ra. Trình cập nhật đó có thể phát hiện khi một trong bốn tổ hợp đang tạo lỗi và hoàn tác về trạng thái tốt đã biết cuối cùng. Thường, trình cập nhật kỳ vọng mỗi instance ứng dụng mới khởi động ở trạng thái không khỏe mạnh trong một thời gian ngắn khi nó chuẩn bị bắt đầu nhận nhiều traffic người dùng. Nếu các mồi đã được kiểm tra như một phần của kiểm tra sẵn sàng (readiness check), việc cập nhật an toàn sẽ thất bại vô hạn, và không có traffic người dùng nào bao giờ được định tuyến đến phiên bản mới. Việc cập nhật tiếp tục bị tạm dừng cho đến khi kỹ sư có thời gian và xu hướng để chẩn đoán điều kiện lỗi, rồi khuyến khích trình cập nhật production hoàn tác sạch sẽ.

Việc kiểm thử production bằng mồi thực sự vừa bảo vệ site, vừa mang lại phản hồi rõ ràng cho kỹ sư. Phản hồi càng sớm thì càng hữu ích. Ngoài ra, nên tự động hóa quá trình kiểm thử để việc gửi cảnh báo cho kỹ sư có thể scale được.

Giả sử mỗi thành phần đang chạy phiên bản phần mềm cũ, sắp được thay thế bằng phiên bản mới (ngay bây giờ hoặc rất sớm). Trong quá trình rollout, phiên bản mới có thể giao tiếp với peer của phiên bản cũ, buộc peer đó phải dùng API đã bị hủy bỏ (deprecated). Ngược lại, phiên bản cũ có thể giao tiếp với peer ở phiên bản mới hơn, sử dụng API mà tại thời điểm release của phiên bản cũ chưa hoạt động đúng. Nhưng giờ nó đã hoạt động rồi, thành thật mà nói! Tốt nhất hãy hy vọng rằng các kiểm thử về sự tương thích tương lai (đang chạy như mồi giám sát) có bao phủ tốt cho API.

### Các Phiên bản Backend Giả (Fake Backend Versions)

Khi cài đặt các kiểm thử release, backend giả thường do đội kỹ thuật của dịch vụ đối tác duy trì và chỉ được tham chiếu như một phụ thuộc build. Kiểm thử cô lập do hạ tầng kiểm thử thực hiện luôn kết hợp backend giả và frontend kiểm thử ở cùng điểm build trong lịch sử kiểm soát revision.

Phụ thuộc build đó có thể đang cung cấp một binary cô lập có thể chạy và, lý tưởng, đội kỹ thuật duy trì nó cắt một release của binary backend giả đó cùng thời điểm họ cắt ứng dụng backend chính và các mồi của họ. Nếu release backend đó khả dụng, có thể đáng giá để bao gồm các kiểm thử release frontend cô lập (không có binary backend giả) trong gói release frontend.

Hệ thống giám sát cần nắm rõ tất cả các phiên bản release ở cả hai phía của một giao diện dịch vụ nhất định giữa hai peer. Nhờ đó, việc thu thập mọi tổ hợp của hai release và kiểm tra xem kiểm thử có vượt qua hay không sẽ không đòi hỏi thêm nhiều cấu hình. Giám sát này không cần chạy liên tục — bạn chỉ cần kiểm tra các tổ hợp mới phát sinh khi một đội cắt release mới. Những vấn đề như vậy không nhất thiết phải chặn chính release mới đó.

Ngược lại, quy trình tự động hóa rollout lý tưởng cần chặn các lần triển khai production liên quan cho đến khi các tổ hợp có vấn đề không còn khả thi. Tương tự, hệ thống tự động hóa của đội đối tác có thể cân nhắc rút (draining) (và nâng cấp) các replica chưa di chuyển khỏi tổ hợp có vấn đề.

## Kết luận (Conclusion)

Kiểm thử là một trong những khoản đầu tư sinh lời nhất giúp kỹ sư cải thiện độ tin cậy của sản phẩm. Đây không phải hoạt động chỉ diễn ra một hoặc hai lần trong vòng đời dự án, mà mang tính liên tục. Việc viết kiểm thử tốt đòi hỏi nỗ lực đáng kể, tương tự như công sức cần thiết để xây dựng và duy trì hạ tầng thúc đẩy văn hóa kiểm thử mạnh mẽ. Bạn không thể sửa một vấn đề cho đến khi hiểu rõ nó, và trong kỹ thuật, bạn chỉ có thể hiểu vấn đề bằng cách đo lường. Các phương pháp luận và kỹ thuật trong chương này cung cấp nền tảng vững chắc để đo lường các lỗi và sự không chắc chắn trong hệ thống phần mềm, đồng thời giúp kỹ sư suy luận về độ tin cậy của phần mềm trong quá trình viết và release đến người dùng.

<a id="fn1"></a>[1](#fn1) Chương này giải thích cách tối đa hóa giá trị thu được từ việc đầu tư nỗ lực kỹ thuật vào kiểm thử. Một khi một kỹ sư định nghĩa các kiểm thử phù hợp (cho một hệ thống nhất định) một cách tổng quát, công việc còn lại là chung giữa tất cả các đội SRE và do đó có thể coi là hạ tầng chia sẻ. Hạ tầng đó bao gồm một trình lên lịch (scheduler, chia sẻ tài nguyên được ngân sách hóa xuyên suốt các dự án vốn không liên quan) và các trình thực thi (executors, sandbox hóa các binary kiểm thử để ngăn chúng được coi là tin cậy). Hai thành phần hạ tầng này mỗi cái có thể coi là một dịch vụ được SRE hỗ trợ bình thường (giống lưu trữ scale cụm), và do đó sẽ không được thảo luận thêm.

<a id="fn2"></a>[2](#fn2) Để đọc thêm về sự tương đương, xem [*https://stackoverflow.com/questions/1909280/equivalence-class-testing-vs-boundary-value-testing*](https://stackoverflow.com/questions/1909280/equivalence-class-testing-vs-boundary-value-testing).

<a id="fn3"></a>[3](#fn3) Xem [*https://dagger.dev/*](https://dagger.dev/).

<a id="fn4"></a>[4](#fn4) Một nguyên tắc kinh nghiệm tiêu chuẩn là bắt đầu bằng việc để release ảnh hưởng 0.1% traffic người dùng, và sau đó scale theo các bậc độ lớn mỗi 24 giờ trong khi thay đổi vị trí địa lý của các server đang được nâng cấp (sau đó ngày 2: 1%, ngày 3: 10%, ngày 4: 100%).

<a id="fn5"></a>[5](#fn5) Ví dụ, giả định một khoảng cách 24 giờ của sự tăng trưởng hàm mũ liên tục giữa 1% và 10%, khoảng 37.500 giây, hoặc khoảng 10 giờ và 25 phút.

<a id="fn6"></a>[6](#fn6) Chúng tôi đang sử dụng "order" (bậc) ở đây theo nghĩa bậc trong "ký hiệu big O" (big O notation). Để có thêm ngữ cảnh, xem [*https://en.wikipedia.org/wiki/Big\_O\_notation*](https://en.wikipedia.org/wiki/Big_O_notation).

<a id="fn7"></a>[7](#fn7) Để có thêm về chủ đề này, chúng tôi rất khuyên [[Bla14]](https://sre.google/sre-book/bibliography#Bla14) bởi đồng nghiệp trước đây của chúng tôi và cựu Googler, Mike Bland.

<a id="fn8"></a>[8](#fn8) Xem [*https://github.com/google/bazel*](https://github.com/google/bazel).

<a id="fn9"></a>[9](#fn9) Ví dụ, code đang kiểm thử bao bọc một API không tầm thường để cung cấp một trừu tượng đơn giản hơn và tương thích ngược. API vốn đồng bộ (synchronous) thay vào đó trả về một future. Các lỗi tham số gọi vẫn giao một ngoại lệ (exception), nhưng không cho đến khi future được đánh giá. Code đang kiểm thử truyền kết quả API trực tiếp trở lại cho người gọi. Nhiều trường hợp lạm dụng tham số có thể không được bắt.

<a id="fn10"></a>[10](#fn10) Phần này nói cụ thể về các công cụ được sử dụng bởi SRE cần phải scale được. Tuy nhiên, SRE cũng phát triển và sử dụng các công cụ không nhất thiết cần phải scale được. Các công cụ không cần phải scale được cũng cần được kiểm thử, nhưng những công cụ này nằm ngoài phạm vi của phần này, và do đó sẽ không được thảo luận ở đây. Vì dấu chân rủi ro của chúng tương tự như các ứng dụng hướng người dùng, các chiến lược kiểm thử tương tự áp dụng được cho những công cụ do SRE phát triển đó.

<a id="fn11"></a>[11](#fn11) Xem [*https://github.com/Netflix/SimianArmy/wiki/Chaos-Monkey*](https://github.com/Netflix/SimianArmy/wiki/Chaos-Monkey).

<a id="fn12"></a>[12](#fn12) Xem [*https://github.com/aphyr/jepsen*](https://github.com/aphyr/jepsen).

<a id="fn13"></a>[13](#fn13) Ngay cả nếu lần chạy kiểm thử được lặp lại với cùng hạt ngẫu nhiên để các sự giết (kills) của task ở cùng thứ tự, không có sự tuần tự hóa (serialization) giữa các lần giết và traffic người dùng giả. Vì vậy, không có bảo đảm rằng đường code thực tế quan sát được trước đó sẽ bây giờ được thực thi lại.

<a id="fn14"></a>[14](#fn14) Xem [*https://xkcd.com/303/*](https://xkcd.com/303/).

<a id="fn15"></a>[15](#fn15) Có thể thu được qua *Mechanical Turk* hoặc các dịch vụ tương tự.

<a id="fn16"></a>[16](#fn16) Xem [*https://github.com/google/protobuf*](https://github.com/google/protobuf).

<a id="fn17"></a>[17](#fn17) Không phải vì các kỹ sư phần mềm không nên viết chúng. Các công cụ vượt giữa các mảng công nghệ và trải qua các tầng trừu tượng có xu hướng có các liên kết yếu với nhiều đội phần mềm và một liên kết hơi mạnh hơn với các đội hệ thống.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
