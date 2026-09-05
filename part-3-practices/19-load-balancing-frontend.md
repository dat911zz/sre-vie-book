# Chương 19. Cân bằng Tải ở Frontend (Load Balancing at the Frontend)

> **Nguyên bản:** [Chapter 19 - Load Balancing at the Frontend](https://sre.google/sre-book/load-balancing-frontend/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Piotr Lewandowski
*Biên tập:* Sarah Chavis

Chúng tôi phục vụ hàng triệu yêu cầu mỗi giây và, như bạn có thể đã đoán, chúng tôi dùng nhiều hơn một máy tính để xử lý chúng. Nhưng ngay cả khi *có* một siêu máy tính (supercomputer) theo cách nào đó xử lý được tất cả những yêu cầu đó (hãy tưởng tượng kết nối mạng mà một cấu hình như vậy sẽ đòi hỏi!), chúng tôi vẫn sẽ không chọn chiến lược dựa vào một điểm thất bại duy nhất (single point of failure); khi đối phó với các hệ thống quy mô lớn, đặt tất cả trứng vào một giỏ là một công thức dẫn đến thảm họa.

Chương này tập trung vào cân bằng tải (load balancing) cấp cao — cách chúng tôi cân bằng traffic (lưu lượng) người dùng *giữa* các datacenter (trung tâm dữ liệu). Chương tiếp theo sẽ phóng to để khám phá cách chúng tôi cài đặt cân bằng tải *bên trong* một datacenter.

## Sức mạnh Không phải là Câu trả lời (Power Isn't the Answer)

Vì mục đích tranh luận, hãy giả sử chúng tôi có một máy cực kỳ mạnh mẽ và một mạng không bao giờ thất bại. Liệu cấu hình *đó* có đủ đáp ứng nhu cầu của Google không? Không. Ngay cả cấu hình này vẫn bị giới hạn bởi các ràng buộc vật lý liên quan đến hạ tầng mạng. Ví dụ, tốc độ ánh sáng là một yếu tố hạn chế tốc độ truyền thông của cáp quang, tạo ra một giới hạn trên về tốc độ phục vụ dữ liệu dựa trên khoảng cách mà dữ liệu phải di chuyển. Ngay cả trong một thế giới lý tưởng, việc dựa vào một hạ tầng có một điểm thất bại duy nhất vẫn là một ý tưởng tồi.

Trong thực tế, Google vận hành hàng nghìn máy và phục vụ lượng người dùng còn lớn hơn, với nhiều người gửi đồng thời nhiều yêu cầu. *[Traffic load balancing](https://sre.google/workbook/managing-load/)* (Cân bằng tải traffic) là cách chúng tôi xác định máy nào trong số vô vàn máy tại các datacenter sẽ xử lý một yêu cầu cụ thể. Về lý tưởng, traffic được phân phối xuyên suốt nhiều liên kết mạng, datacenter và máy theo cách "tối ưu". Vậy "tối ưu" ở đây có nghĩa là gì? Thực tế không có một câu trả lời duy nhất, vì giải pháp tối ưu phụ thuộc mạnh mẽ vào một số yếu tố:

-   Cấp bậc phân cấp mà chúng tôi đánh giá vấn đề (toàn cầu đối với cục bộ)
-   Cấp kỹ thuật mà chúng tôi đánh giá vấn đề (phần cứng đối với phần mềm)
-   Bản chất của traffic mà chúng tôi đang đối phó

Hãy bắt đầu bằng cách xem xét hai kịch bản traffic phổ biến: một yêu cầu tìm kiếm cơ bản và một yêu cầu tải video (upload). Vì người dùng muốn nhận kết quả truy vấn nhanh chóng, nên biến quan trọng nhất cho yêu cầu tìm kiếm là độ trễ (latency). Ngược lại, người dùng chấp nhận việc upload video mất một khoảng thời gian không thể bỏ qua, nhưng cũng muốn những yêu cầu đó thành công ngay lần đầu, nên biến quan trọng nhất cho video upload là thông lượng (throughput). Những nhu cầu khác nhau của hai loại yêu cầu này đóng vai trò trong cách chúng tôi xác định sự phân phối tối ưu cho mỗi yêu cầu ở cấp *toàn cầu*:

-   Yêu cầu tìm kiếm được gửi đến datacenter khả dụng gần nhất — tính theo thời gian đi-và-đến (round-trip time, RTT) — nhằm tối thiểu hóa độ trễ của yêu cầu.
-   Luồng video upload được định tuyến qua một đường khác — có thể là một liên kết đang được sử dụng dưới mức — nhằm tối đa hóa thông lượng, chấp nhận đánh đổi bằng độ trễ.

Nhưng ở cấp *cục bộ*, bên trong một datacenter, chúng tôi thường giả định rằng tất cả các máy trong tòa nhà đều cách người dùng như nhau và được kết nối với cùng một mạng. Vì vậy, sự phân phối tải tối ưu tập trung vào việc sử dụng tài nguyên một cách tối ưu và bảo vệ một server đơn lẻ khỏi quá tải (overload).

Tất nhiên, ví dụ này chỉ phác họa một bức tranh đơn giản hóa đáng kể. Trong thực tế, việc phân phối [tải tối ưu](https://sre.google/sre-book/load-balancing-datacenter/) phải tính đến nhiều yếu tố hơn: một số yêu cầu có thể được định tuyến đến một datacenter hơi xa hơn để giữ cache (bộ đệm) ấm, hoặc traffic không tương tác (non-interactive) có thể được chuyển đến một vùng hoàn toàn khác nhằm tránh tắc nghẽn mạng. Cân bằng tải, đặc biệt cho các hệ thống lớn, không hề đơn giản hay tĩnh. Tại Google, chúng tôi giải quyết vấn đề này bằng cách cân bằng tải ở nhiều cấp, trong đó hai cấp được mô tả trong các phần sau. Để thảo luận cụ thể, chúng tôi sẽ xem xét các yêu cầu HTTP được gửi qua TCP. Cân bằng tải cho các dịch vụ không có trạng thái (stateless, như DNS qua UDP) khác đi một chút, nhưng phần lớn các cơ chế được mô tả ở đây vẫn áp dụng được cho cả các dịch vụ không trạng thái (stateless).

## Cân bằng Tải Sử dụng DNS (Load Balancing Using DNS)

Trước khi một client thậm chí có thể gửi một yêu cầu HTTP, nó thường phải tra cứu một địa chỉ IP thông qua DNS. Điều này mở ra cơ hội hoàn hảo để giới thiệu tầng cân bằng tải đầu tiên của chúng tôi: *cân bằng tải DNS*. Giải pháp đơn giản nhất là trả về nhiều bản ghi `A` hoặc `AAAA` trong phản hồi DNS và để client chọn một địa chỉ IP tùy ý. Trong khi đơn giản về mặt khái niệm và dễ cài đặt, giải pháp này đặt ra nhiều thách thức.

Vấn đề đầu tiên là cách này chỉ cho phép kiểm soát hành vi client ở mức rất hạn chế: các bản ghi được chọn ngẫu nhiên, và mỗi bản sẽ thu hút một lượng traffic xấp xỉ bằng nhau. Chúng tôi có thể giảm nhẹ vấn đề này không? Về mặt lý thuyết, chúng tôi có thể dùng các bản ghi `SRV` để chỉ định trọng số và thứ tự ưu tiên, nhưng các bản ghi `SRV` chưa được áp dụng cho HTTP.

Một vấn đề tiềm tàng khác bắt nguồn từ việc client thường không thể xác định được địa chỉ gần nhất. Chúng tôi *có thể* giảm nhẹ kịch bản này bằng cách dùng một địa chỉ anycast cho các nameserver (server tên) có thẩm quyền, tận dụng thực tế rằng các truy vấn DNS sẽ chảy đến địa chỉ gần nhất. Trong phản hồi, server có thể trả về các địa chỉ được định tuyến đến datacenter gần nhất. Một cải tiến hơn nữa là xây dựng một bản đồ của tất cả các mạng cùng vị trí vật lý xấp xỉ của chúng, rồi phục vụ các phản hồi DNS dựa trên ánh xạ đó. Tuy nhiên, giải pháp này đến với cái giá của một cài đặt server DNS phức tạp hơn nhiều, cùng việc duy trì một đường ống (pipeline) giữ cho bản đồ vị trí luôn cập nhật.

Tất nhiên, không có giải pháp nào trong số này là tầm thường, do một đặc tính cơ bản của DNS: người dùng cuối hiếm khi nói chuyện trực tiếp với nameserver có thẩm quyền. Thay vào đó, một server DNS đệ quy (recursive) thường nằm ở đâu đó giữa người dùng cuối và nameserver. Server này proxy (chuyển tiếp) các truy vấn giữa người dùng cuối và server, và thường cung cấp một tầng cache (bộ đệm). Người trung gian DNS có ba hệ quả rất quan trọng đối với việc quản lý traffic:

-   Phân giải đệ quy các địa chỉ IP
-   Các đường phản hồi không tất định (nondeterministic)
-   Các biến thể cache bổ sung

Phân giải đệ quy các địa chỉ IP gây ra vấn đề, vì địa chỉ IP mà nameserver có thẩm quyền nhìn thấy không thuộc về user, mà thuộc về trình phân giải đệ quy. Đây là một hạn chế nghiêm trọng, vì nó chỉ cho phép tối ưu hóa phản hồi cho khoảng cách ngắn nhất giữa trình phân giải và nameserver. Một giải pháp khả thi là dùng phần mở rộng EDNS0 được đề xuất trong [[Con15]](https://sre.google/sre-book/bibliography#Con15), bao gồm thông tin về subnet (mạng con) của client trong truy vấn DNS do trình phân giải đệ quy gửi. Bằng cách này, nameserver có thẩm quyền trả về một phản hồi tối ưu từ quan điểm của user, thay vì từ quan điểm của trình phân giải. Dù chưa phải là tiêu chuẩn chính thức, các lợi ích hiển nhiên của nó đã khiến các trình phân giải DNS lớn nhất (như OpenDNS và Google<sup>[1](#fn1)</sup>) hỗ trợ nó từ trước.

Khó khăn không chỉ nằm ở việc tìm địa chỉ IP tối ưu để trả về cho một yêu cầu nhất định của user, mà còn ở chỗ chính nameserver đó có thể phải phục vụ hàng nghìn hoặc hàng triệu user, trải dài qua các vùng từ một văn phòng đơn lẻ đến cả một châu lục. Ví dụ, một ISP (nhà cung cấp dịch vụ Internet) quốc gia lớn có thể chạy nameserver cho toàn bộ mạng của mình từ một datacenter, nhưng lại có kết nối mạng ở mỗi khu vực đô thị. Khi đó, nameserver của ISP sẽ trả về một phản hồi với địa chỉ IP phù hợp nhất cho datacenter của họ, mặc dù tồn tại các đường mạng tốt hơn cho tất cả user!

Cuối cùng, các trình phân giải đệ quy thường cache các phản hồi và chuyển tiếp chúng trong các giới hạn được chỉ định bởi trường time-to-live (thời gian sống, TTL) trong bản ghi DNS. Hệ quả cuối cùng là việc ước tính tác động của một phản hồi nhất định trở nên khó khăn: một phản hồi có thẩm quyền đơn lẻ có thể chỉ đến một user, hoặc đến hàng nghìn user. Chúng tôi giải quyết vấn đề này theo hai cách:

-   Chúng tôi phân tích các thay đổi traffic và liên tục cập nhật danh sách các trình phân giải DNS đã biết, kèm kích thước xấp xỉ của cơ sở user đằng sau từng trình phân giải, qua đó cho phép chúng tôi theo dõi tác động tiềm tàng của bất kỳ trình phân giải nào.
-   Chúng tôi ước tính sự phân bố địa lý của các user đằng sau mỗi trình phân giải được theo dõi, nhằm tăng khả năng định hướng những user đó đến vị trí tối ưu nhất.

Việc ước tính sự phân bố địa lý đặc biệt khó khăn khi cơ sở user trải dài qua các vùng lớn. Trong những trường hợp như vậy, chúng tôi thực hiện các đánh đổi để chọn vị trí tốt nhất và tối ưu hóa trải nghiệm cho đa số user.

Vậy “vị trí tốt nhất” thực sự có nghĩa là gì trong ngữ cảnh cân bằng tải DNS? Câu trả lời hiển nhiên nhất là vị trí gần user nhất. Tuy nhiên, ngoài việc xác định vị trí của user vốn đã khó, còn có các tiêu chí bổ sung. Bộ cân bằng tải DNS cần đảm bảo rằng datacenter được chọn có đủ năng lực phục vụ các yêu cầu từ những user có khả năng cao sẽ nhận được phản hồi. Nó cũng cần biết rằng datacenter được chọn cùng kết nối mạng của nó đang ở trạng thái tốt, vì việc định hướng các yêu cầu user đến một datacenter đang gặp sự cố điện hoặc mạng là điều không mong muốn. May mắn thay, chúng tôi có thể tích hợp server DNS có thẩm quyền với các hệ thống kiểm soát toàn cầu theo dõi traffic, năng lực và trạng thái hạ tầng của mình.

Hệ quả thứ ba của người trung gian DNS liên quan đến cache. Giả định rằng nameserver có thẩm quyền không thể flush (xóa) cache của trình phân giải, các bản ghi DNS cần một TTL tương đối thấp. Điều này thực chất đặt một giới hạn dưới cho tốc độ mà các thay đổi DNS có thể lan truyền đến user.<sup>[2](#fn2)</sup> Thật không may, ngoài việc ghi nhớ điều này khi đưa ra các quyết định cân bằng tải, chúng tôi có rất ít điều có thể làm khác.

Dù còn nhiều hạn chế, DNS vẫn là phương án đơn giản và hiệu quả nhất để cân bằng tải ngay trước khi kết nối của user được thiết lập. Tuy nhiên, cần nhấn mạnh rằng chỉ dùng DNS để cân bằng tải là chưa đủ. Lưu ý rằng mọi phản hồi DNS đều phải nằm trong giới hạn 512 byte<sup>[3](#fn3)</sup> theo quy định của RFC 1035 [[Moc87]](https://sre.google/sre-book/bibliography#Moc87). Giới hạn này quy định số lượng địa chỉ tối đa có thể đưa vào một phản hồi DNS, và con số đó gần như chắc chắn nhỏ hơn tổng số server của chúng tôi.

Để *thực sự* giải quyết vấn đề cân bằng tải frontend, sau cấp cân bằng tải DNS ban đầu, cần bổ sung một cấp sử dụng các địa chỉ IP ảo (virtual IP).

## Cân bằng Tải tại Địa chỉ IP Ảo (Load Balancing at the Virtual IP Address)

Các địa chỉ IP ảo (VIPs) không gắn với bất kỳ giao diện mạng cụ thể nào, mà thường được chia sẻ trên nhiều thiết bị. Tuy nhiên, từ góc nhìn của user, VIP vẫn là một địa chỉ IP đơn lẻ, bình thường. Về mặt lý thuyết, cách làm này giúp chúng tôi che giấu các chi tiết cài đặt (như số lượng máy phía sau một VIP nhất định) và tạo thuận lợi cho việc bảo trì, vì có thể lên lịch nâng cấp hoặc thêm máy vào bể (pool) mà user không hề hay biết.

Trong thực tế, phần quan trọng nhất của cài đặt VIP là một thiết bị gọi là *network load balancer* (bộ cân bằng tải mạng). Bộ cân bằng nhận các packet (gói tin) và chuyển chúng đến một trong các máy phía sau VIP. Những backend (phía sau) này sau đó có thể tiếp tục xử lý yêu cầu.

Có một số cách tiếp cận khả thi mà bộ cân bằng có thể áp dụng để quyết định backend nào nên nhận yêu cầu. Cách tiếp cận đầu tiên (và có lẽ trực quan nhất) là luôn ưu tiên backend ít tải nhất. Về mặt lý thuyết, cách tiếp cận này nên mang lại trải nghiệm user cuối tốt nhất vì các yêu cầu luôn được định tuyến đến máy bận ít nhất. Thật không may, logic này vỡ vụn nhanh chóng trong trường hợp các giao thức có trạng thái (stateful), những giao thức phải dùng cùng một backend trong suốt thời gian của một yêu cầu. Yêu cầu này có nghĩa là bộ cân bằng phải theo dõi tất cả các kết nối đi qua nó để đảm bảo tất cả các packet tiếp theo được gửi đến đúng backend. Một tùy chọn thay thế là dùng một phần của packet để tạo một ID kết nối (có thể dùng một hàm băm (hash function) cùng một số thông tin từ packet), rồi dùng ID kết nối đó để chọn backend. Ví dụ, ID kết nối có thể được biểu diễn như:

```
id(packet) mod N
```

trong đó `id` là một hàm lấy `packet` làm input (đầu vào) và tạo ra một ID kết nối, còn `N` là số lượng backend được cấu hình.

Điều này tránh việc lưu trữ trạng thái, và tất cả các packet thuộc về một kết nối đơn lẻ luôn được chuyển đến cùng một backend. Thành công? Chưa hẳn. Điều gì xảy ra nếu một backend thất bại và phải được loại khỏi danh sách? Đột nhiên `N` trở thành `N-1`, và `id(packet) mod N` thành `id(packet) mod N-1`. Gần như mọi packet đột nhiên ánh xạ đến một backend khác! Nếu các backend không chia sẻ bất kỳ trạng thái nào, sự ánh xạ lại này buộc phải reset gần như tất cả các kết nối hiện có. Kịch bản này chắc chắn *không* phải trải nghiệm user tốt nhất, ngay cả khi những sự kiện như vậy hiếm gặp.

May mắn thay, có một giải pháp thay thế không đòi hỏi giữ trạng thái của mọi kết nối trong bộ nhớ, nhưng cũng không buộc tất cả các kết nối phải reset khi một máy đơn lẻ đi xuống: *consistent hashing* (băm nhất quán). Được đề xuất vào năm 1997, consistent hashing [[Kar97]](https://sre.google/sre-book/bibliography#Kar97) mô tả một cách để có một thuật toán ánh xạ vẫn tương đối ổn định ngay cả khi các backend mới được thêm vào hoặc loại bỏ khỏi danh sách. Cách tiếp cận này tối thiểu hóa sự gián đoạn đối với các kết nối hiện có khi bể backend thay đổi. Kết quả là, chúng tôi thường có thể dùng theo dõi kết nối đơn giản, nhưng sẽ chuyển sang dùng consistent hashing làm phương án dự phòng khi hệ thống chịu áp lực (ví dụ, trong khi một cuộc tấn công từ chối dịch vụ (denial of service) đang diễn ra).

Quay lại câu hỏi lớn hơn: chính xác một network load balancer nên chuyển các packet đến backend VIP đã chọn như thế nào? Một giải pháp là thực hiện Network Address Translation (chuyển đổi địa chỉ mạng). Tuy nhiên, cách này đòi hỏi phải giữ một entry (bản ghi) cho mọi kết nối đơn lẻ trong bảng theo dõi, do đó loại trừ khả năng có một cơ chế dự phòng (fallback) hoàn toàn không trạng thái.

Một giải pháp khác là sửa đổi thông tin trên tầng liên kết dữ liệu (data link layer, tầng 2 của mô hình mạng OSI). Bằng cách thay đổi địa chỉ MAC đích của packet được chuyển, bộ cân bằng có thể giữ nguyên tất cả thông tin trong các tầng cao hơn, để backend nhận được đúng các địa chỉ IP nguồn và đích ban đầu. Backend sau đó có thể gửi phản hồi trực tiếp đến người gửi ban đầu — một kỹ thuật gọi là *Direct Server Response* (Phản hồi Server Trực tiếp, DSR). Nếu yêu cầu user nhỏ và phản hồi lớn (ví dụ, phần lớn các yêu cầu HTTP), DSR mang lại sự tiết kiệm lớn, vì chỉ một phần nhỏ traffic cần đi qua load balancer. Tốt hơn nữa, DSR không đòi hỏi giữ trạng thái trên thiết bị load balancer. Thật không may, việc dùng tầng 2 cho cân bằng tải nội bộ *thực sự* phải chịu các nhược điểm nghiêm trọng khi triển khai ở quy mô: tất cả các máy (tức là tất cả load balancer và tất cả backend của chúng) phải có thể đạt được nhau ở tầng liên kết dữ liệu. Điều này không phải là vấn đề nếu mạng hỗ trợ được kết nối như vậy và số lượng máy không tăng quá mức, vì tất cả các máy cần cư trú trong một miền phát sóng (broadcast domain) đơn lẻ. Như bạn có thể hình dung, Google đã vượt ra ngoài giải pháp này từ khá lâu, và phải tìm một cách tiếp cận thay thế.

Giải pháp cân bằng tải VIP hiện tại của chúng tôi [[Eis16]](https://sre.google/sre-book/bibliography#Eis16) dựa trên kỹ thuật đóng gói packet (packet encapsulation). Network load balancer sẽ nhúng packet cần chuyển vào một packet IP khác bằng Generic Routing Encapsulation (đóng gói định tuyến tổng quát, GRE) [[Han94]](https://sre.google/sre-book/bibliography#Han94), sau đó định tuyến đến địa chỉ của một backend. Khi nhận được packet, backend sẽ bóc tách lớp IP+GRE bên ngoài và xử lý packet IP bên trong như thể nó được gửi trực tiếp đến giao diện mạng của mình. Nhờ đó, network load balancer và backend không nhất thiết phải nằm trong cùng một miền phát sóng; chúng thậm chí có thể ở các châu lục khác nhau, miễn là có đường truyền kết nối giữa hai bên.

Việc đóng gói packet là một cơ chế mạnh mẽ, mang lại sự linh hoạt lớn trong thiết kế và tiến hóa của các mạng. Tuy nhiên, nó cũng đi kèm một cái giá: kích thước packet phình to. Đóng gói packet tạo ra overhead (chi phí phụ) (chính xác hơn là 24 byte trong trường hợp IPv4+GRE), điều này có thể khiến packet vượt quá kích thước Maximum Transmission Unit (đơn vị truyền tải tối đa, MTU) khả dụng và đòi hỏi phải phân mảnh (fragmentation).

Một khi packet đạt đến datacenter, việc phân mảnh có thể được tránh bằng cách dùng một MTU lớn hơn bên trong datacenter; tuy nhiên, cách tiếp cận này đòi hỏi một mạng hỗ trợ các Protocol Data Unit (đơn vị dữ liệu giao thức) lớn. Như với nhiều thứ ở quy mô, cân bằng tải nghe có vẻ đơn giản ở bề mặt — cân bằng tải sớm và cân bằng tải thường xuyên — nhưng sự khó khăn nằm ở các chi tiết, cả cho cân bằng tải frontend lẫn cho việc xử lý các packet một khi chúng đến datacenter.

<a id="fn1"></a>[1](#fn1) Xem [*https://groups.google.com/forum/#!topic/public-dns-announce/67oxFjSLeUM*](https://groups.google.com/forum/#!topic/public-dns-announce/67oxFjSLeUM).

<a id="fn2"></a>[2](#fn2) Thật đáng buồn, không phải tất cả các trình phân giải DNS tôn trọng giá trị TTL được đặt bởi các nameserver có thẩm quyền.

<a id="fn3"></a>[3](#fn3) Nếu không, các user phải thiết lập một kết nối TCP chỉ để nhận một danh sách các địa chỉ IP.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
