# Chương 20. Cân bằng Tải trong Datacenter (Load Balancing in the Datacenter)

> **Nguyên bản:** [Chapter 20 - Load Balancing in the Datacenter](https://sre.google/sre-book/load-balancing-datacenter/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Alejandro Forero Cuervo
*Biên tập:* Sarah Chavis

Chương này tập trung vào cân bằng tải (load balancing) bên trong datacenter (trung tâm dữ liệu). Cụ thể, nội dung xoay quanh các thuật toán phân phối công việc cho một luồng truy vấn (query) trong cùng một datacenter. Chúng tôi xem xét các chính sách cấp ứng dụng (application-level policies) nhằm định tuyến các yêu cầu (request) đến những server đơn lẻ có khả năng xử lý. Các nguyên lý mạng cấp thấp hơn (ví dụ, switch (thiết bị chuyển mạch), định tuyến packet (gói tin)) và việc chọn datacenter nằm ngoài phạm vi của chương này.

Giả sử có một luồng truy vấn đến datacenter — có thể từ chính datacenter, từ các datacenter từ xa, hoặc một hỗn hợp của cả hai — ở tốc độ không vượt quá tài nguyên mà datacenter có để xử lý (hoặc chỉ vượt quá trong các khoảng thời gian rất ngắn). Cũng giả sử rằng bên trong datacenter có các *dịch vụ* (services) mà những truy vấn này hướng tới. Những dịch vụ này được cài đặt dưới dạng nhiều tiến trình server đồng nhất (homogeneous), thay thế được, phần lớn chạy trên các máy khác nhau. Các dịch vụ nhỏ nhất thường có ít nhất ba tiến trình như vậy (dùng ít hơn có nghĩa là mất 50% hoặc nhiều hơn năng lực khi mất một máy đơn lẻ), còn những cái lớn nhất có thể có hơn 10.000 tiến trình (tùy thuộc vào kích thước datacenter). Trong trường hợp điển hình, các dịch vụ gồm từ 100 đến 1.000 tiến trình. Chúng tôi gọi những tiến trình này là *backend tasks* (nhiệm vụ phía sau, hoặc đơn giản là *backends*). Các task (nhiệm vụ) khác, gọi là *client tasks* (nhiệm vụ khách hàng), giữ các kết nối đến các backend task. Với mỗi truy vấn đến, một client task phải quyết định backend task nào nên xử lý nó. Các client giao tiếp với các backends bằng một giao thức cài đặt trên sự kết hợp của TCP và UDP.

Chúng tôi nên lưu ý rằng các datacenter của Google chứa một tập dịch vụ đa dạng rộng lớn, cài đặt các kết hợp khác nhau của các chính sách được thảo luận trong chương này. Ví dụ làm việc của chúng tôi, như vừa mô tả, không khớp trực tiếp với bất kỳ dịch vụ nào. Nó là một kịch bản tổng quát cho phép chúng tôi thảo luận các kỹ thuật khác nhau mà chúng tôi thấy hữu ích cho các dịch vụ khác nhau. Một số kỹ thuật có thể áp dụng được nhiều hơn (hoặc ít hơn) cho các use case (trường hợp sử dụng) cụ thể, nhưng tất cả đều được thiết kế và cài đặt bởi nhiều kỹ sư Google trong suốt nhiều năm.

Chúng tôi áp dụng các kỹ thuật này ở nhiều tầng của stack. Chẳng hạn, phần lớn yêu cầu HTTP từ bên ngoài sẽ đến GFE (Google Frontend), hệ thống proxy ngược HTTP của chúng tôi. GFE sử dụng các thuật toán này, kết hợp với những thuật toán được mô tả trong [Cân bằng Tải ở Frontend](https://sre.google/sre-book/load-balancing-frontend/), để định tuyến payload (bộ dữ liệu mang) và metadata (dữ liệu mô tả) của yêu cầu đến các tiến trình đơn lẻ đang chạy ứng dụng có khả năng xử lý. Việc này dựa trên cấu hình ánh xạ các mẫu URL (địa chỉ) khác nhau đến từng ứng dụng, mỗi ứng dụng do một đội riêng quản lý. Để tạo ra payload phản hồi (gửi lại cho GFE, rồi chuyển đến trình duyệt), các ứng dụng này thường tái sử dụng chính các thuật toán đó để giao tiếp với hạ tầng hoặc các dịch vụ bổ sung mà chúng phụ thuộc. Đôi khi, stack phụ thuộc có thể khá sâu, khiến một yêu cầu HTTP đơn lẻ kích hoạt một chuỗi phụ thuộc truyền dẫn (transitive chain) dài, lan tỏa đến nhiều hệ thống, có thể với mức fan-out (sự tán ra) cao ở các điểm khác nhau.

## Trường hợp Lý tưởng (The Ideal Case)

Trong trường hợp lý tưởng, tải của một dịch vụ nhất định được phân bổ hoàn hảo trên tất cả các backend task của nó, và tại bất kỳ thời điểm nào, backend task ít tải nhất và nhiều tải nhất tiêu thụ chính xác cùng một lượng CPU.

Chúng tôi chỉ có thể gửi traffic (lưu lượng) đến một datacenter cho đến khi task nhiều tải nhất chạm đến giới hạn năng lực của nó; điều này được mô tả trong [Hình 20-1](#hinh-20-1) cho hai kịch bản trong cùng một khoảng thời gian. Trong khoảng thời gian đó, [thuật toán cân bằng tải cross-datacenter (liên datacenter)](https://sre.google/workbook/managing-load/) phải tránh gửi thêm bất kỳ traffic nào vào datacenter, vì việc làm như vậy có nguy cơ quá tải một số task.


<a id="hinh-20-1"></a>![Hình 20-1](../assets/imgs/fig-20-1.jpg)

[Hình 20-1.](#hinh-20-1) Hai kịch bản của sự phân phối tải mỗi task theo thời gian.

Như đồ thị bên trái trong [Hình 20-2](#hinh-20-2) cho thấy, một lượng năng lực đáng kể bị lãng phí: phần năng lực rảnh của mọi task ngoại trừ task nhiều tải nhất.


<a id="hinh-20-2"></a>![Hình 20-2](../assets/imgs/fig-20-2.jpg)

[Hình 20-2.](#hinh-20-2) Histogram (đồ thị phân bố tần số) của CPU được sử dụng và lãng phí trong hai kịch bản.

Cụ thể hơn, ký hiệu *CPU[i]* là tốc độ CPU mà task *i* tiêu thụ tại một thời điểm, và giả sử task 0 là task có tải cao nhất. Khi đó, nếu mức trải rộng (spread) lớn, chúng ta đang lãng phí tổng các chênh lệch CPU của mọi task so với *CPU[0]*: cụ thể, tổng của *(CPU[0] – CPU[i])* trên tất cả các task *i* sẽ bị lãng phí. Ở đây, "lãng phí" nghĩa là phần đã được đặt trước (reserved) nhưng không được sử dụng.

Ví dụ này cho thấy các thực hành cân bằng tải trong-datacenter kém cỏi tạo ra giới hạn nhân tạo cho tính khả dụng của tài nguyên: bạn có thể đặt trước 1.000 CPUs cho dịch vụ của mình trong một datacenter, nhưng không thể thực sự sử dụng nhiều hơn, ví dụ, 700 CPUs.

## Xác định các Task Xấu: Flow Control (Kiểm soát Luồng) và Lame Ducks (Vịt què) (Identifying Bad Tasks: Flow Control and Lame Ducks)

Trước khi quyết định giao yêu cầu client cho backend task nào, chúng tôi cần xác định và tránh các task không khỏe mạnh (unhealthy) trong bể backend.

### Một Cách tiếp cận Đơn giản cho các Task Không khỏe mạnh: Flow Control (A Simple Approach to Unhealthy Tasks: Flow Control)

Giả sử các client task của chúng tôi theo dõi số lượng yêu cầu đang hoạt động (active) mà chúng đã gửi trên mỗi kết nối đến một backend task. Khi số đếm này chạm đến một giới hạn được cấu hình, client coi backend đó là không khỏe mạnh và ngừng gửi yêu cầu cho nó. Đối với phần lớn các backends, 100 là một giới hạn hợp lý; trong trường hợp trung bình, các yêu cầu có xu hướng kết thúc đủ nhanh để rất hiếm khi số yêu cầu đang hoạt động từ một client nhất định đạt đến giới hạn này trong điều kiện vận hành bình thường. Hình thức (rất cơ bản!) của flow control này cũng đóng vai trò như một dạng cân bằng tải đơn giản: nếu một backend task nhất định trở nên quá tải và các yêu cầu bắt đầu xếp hàng, các client sẽ tránh backend đó, và khối lượng công việc trải ra một cách tự nhiên giữa các backend task khác.

Thật không may, cách tiếp cận đơn giản này chỉ bảo vệ các backend task khỏi những dạng quá tải cực đoan, trong khi các backend rất dễ bị quá tải trước khi chạm đến giới hạn đó. Ngược lại, có những trường hợp client chạm đến giới hạn này trong khi backend vẫn còn nhiều tài nguyên dự phòng. Chẳng hạn, một số backend có các yêu cầu sống rất dài (very long-lived) khiến chúng không thể phản hồi nhanh. Chúng tôi đã gặp những trường hợp giới hạn mặc định này phản tác dụng, khiến tất cả backend task không thể truy cập, các yêu cầu bị chặn ở client cho đến khi hết thời gian (time out) và thất bại. Việc tăng giới hạn yêu cầu đang hoạt động có thể tránh tình huống này, nhưng không giải quyết được vấn đề cốt lõi: làm thế nào để phân biệt một task thực sự không khỏe mạnh với một task chỉ đơn giản là phản hồi chậm.

### Một Cách tiếp cận Vững chắc cho các Task Không khỏe mạnh: Trạng thái Lame Duck (A Robust Approach to Unhealthy Tasks: Lame Duck State)

Từ quan điểm của client, một backend task có thể ở trong một trong các trạng thái sau:

#### Khỏe mạnh (Healthy)

Backend task đã khởi tạo đúng và đang xử lý các yêu cầu.

#### Từ chối kết nối (Refusing connections)

Backend task không phản hồi. Nguyên nhân có thể là task đang khởi động hoặc tắt, hoặc backend đang ở trạng thái bất thường (mặc dù hiếm khi một backend ngừng nghe (listen) trên port (cổng) của nó nếu không đang tắt).

#### Vịt què (Lame duck)

Backend task đang nghe trên port của nó và có thể phục vụ, nhưng đang yêu cầu một cách rõ ràng các client ngừng gửi các yêu cầu.

Khi một task chuyển sang trạng thái vịt què, nó phát sóng (broadcast) thông báo này đến tất cả các client đang hoạt động. Vậy còn các client không hoạt động? Trong hệ thống RPC (Remote Procedure Call — lời gọi thủ tục từ xa) của Google, các client không hoạt động (tức là không có kết nối TCP đang hoạt động) vẫn gửi các gói kiểm tra sức khỏe UDP định kỳ. Nhờ đó, thông tin vịt què được lan truyền nhanh chóng đến mọi client — thường chỉ trong 1 hoặc 2 RTT (thời gian đi-và-đến) — bất kể trạng thái hiện tại của chúng.

Lợi ích chính của việc cho phép một task tồn tại trong trạng thái vịt què bán-vận hành là nó đơn giản hóa việc tắt sạch (clean shutdown), từ đó tránh việc trả về lỗi cho tất cả các yêu cầu xui xẻo đang hoạt động trên các backend task đang tắt. Nhờ đó, việc hạ một backend task đang có yêu cầu hoạt động sẽ không làm phát sinh lỗi nào — điều này tạo thuận lợi cho việc push code (đẩy mã), bảo trì, hay xử lý sự cố máy, những việc có thể đòi hỏi khởi động lại toàn bộ các task liên quan. Một quá trình tắt như vậy sẽ đi qua các bước tổng quát sau:

1.  Trình lên lịch job (nhiệm vụ) gửi một tín hiệu SIGTERM đến backend task.
2. Backend task chuyển sang trạng thái vịt què và yêu cầu các client của nó gửi các yêu cầu mới đến các backend task khác. Việc này được thực hiện thông qua một lời gọi API trong cài đặt RPC, được gọi một cách rõ ràng trong bộ xử lý SIGTERM (SIGTERM handler).
3.  Bất kỳ yêu cầu nào đang diễn ra, được bắt đầu trước khi backend task rơi vào trạng thái vịt què (hoặc sau khi nó vào trạng thái đó nhưng trước khi một client phát hiện), đều được thực thi bình thường.
4.  Khi các phản hồi chảy ngược về các client, số lượng yêu cầu đang hoạt động cho backend dần dần giảm về zero.
5.  Sau một khoảng thời gian được cấu hình, backend task hoặc thoát sạch hoặc bị trình lên lịch job giết (kill). Khoảng thời gian này cần đủ lớn để tất cả các yêu cầu điển hình có đủ thời gian kết thúc. Giá trị cụ thể phụ thuộc vào dịch vụ, nhưng một nguyên tắc kinh nghiệm tốt là từ 10s đến 150s tùy theo độ phức tạp của client.

Chiến lược này cũng cho phép client thiết lập kết nối đến các backend task trong khi chúng đang chạy các thủ tục khởi tạo có thể kéo dài (và do đó chưa sẵn sàng phục vụ). Các backend task có thể chọn chỉ bắt đầu nghe kết nối khi đã sẵn sàng, nhưng điều này sẽ trì hoãn việc đàm phán kết nối một cách không cần thiết. Ngay khi backend task sẵn sàng phục vụ, nó sẽ gửi tín hiệu rõ ràng cho các client.

## Giới hạn Bể Kết nối bằng Subsetting (Phân tập) (Limiting the Connections Pool with Subsetting)

Ngoài việc quản lý sức khỏe, một cân nhắc khác cho cân bằng tải là *subsetting*: giới hạn bể các backend task tiềm tàng mà một client task tương tác.

Mỗi client trong hệ thống RPC của chúng tôi duy trì một bể các kết nối sống dài đến các backend của nó để gửi các yêu cầu mới. Những kết nối này thường được thiết lập sớm khi client khởi động và thường tiếp tục mở, với các yêu cầu chảy qua chúng, cho đến khi client kết thúc. Một mô hình thay thế là thiết lập và tháo gỡ một kết nối cho mỗi yêu cầu, nhưng mô hình này có chi phí tài nguyên và độ trễ đáng kể. Trong trường hợp khó (corner case) mà một kết nối vẫn rảnh trong một thời gian dài, cài đặt RPC của chúng tôi có một tối ưu hóa chuyển kết nối sang một chế độ "không hoạt động" rẻ hơn, trong đó, ví dụ, tần suất kiểm tra sức khỏe được giảm và kết nối TCP cơ bản bị hủy bỏ để nhường chỗ cho UDP.

Mỗi kết nối đều tiêu tốn một lượng bộ nhớ và CPU (do việc kiểm tra sức khỏe định kỳ) ở cả hai đầu. Về mặt lý thuyết, overhead (công việc phụ) này là nhỏ, nhưng khi tích lũy trên nhiều máy, nó có thể nhanh chóng trở nên đáng kể. Subsetting giúp tránh việc một client kết nối đến quá nhiều backend task, hoặc một backend task nhận kết nối từ quá nhiều client task. Trong cả hai trường hợp, bạn sẽ lãng phí rất nhiều tài nguyên mà chỉ thu được lợi ích rất ít.

### Chọn Tập phù hợp (Picking the Right Subset)

Việc chọn đúng tập (subset) rút gọn về việc chọn bao nhiêu backend task mà mỗi client kết nối đến — kích thước tập (subset size) — và thuật toán chọn. Chúng tôi thường dùng một kích thước tập từ 20 đến 100 backend task, nhưng kích thước tập "đúng" cho một hệ thống phụ thuộc mạnh mẽ vào hành vi điển hình của dịch vụ bạn. Ví dụ, bạn có thể muốn dùng một kích thước tập lớn hơn nếu:

-   Số lượng client thường nhỏ hơn đáng kể so với số lượng backends. Trong trường hợp này, bạn cần đảm bảo mỗi client có đủ nhiều backends để tránh tình trạng một số backend task không nhận được bất kỳ traffic nào.
-   Tải trọng thường xuyên mất cân bằng giữa các job client (tức là một client task gửi nhiều yêu cầu hơn những client khác). Kịch bản này điển hình khi các client thỉnh thoảng gửi các đợt (bursts) yêu cầu. Trong trường hợp này, chính các client nhận yêu cầu từ các client khác đôi khi có fan-out lớn (ví dụ, "đọc tất cả thông tin của tất cả những người theo dõi của một user nhất định"). Vì một đợt yêu cầu sẽ tập trung vào tập được gán cho client, bạn cần một kích thước tập lớn hơn để đảm bảo tải được trải đều trên một tập lớn hơn các backend task khả dụng.

Khi đã xác định kích thước tập, chúng tôi cần một thuật toán để định nghĩa tập các backend task mà mỗi client task sẽ sử dụng. Nghe có vẻ đơn giản, nhưng việc này nhanh chóng trở nên phức tạp khi làm việc với các hệ thống quy mô lớn, nơi việc provision (cấp phát) hiệu quả là thiết yếu và các lần khởi động lại hệ thống là điều chắc chắn xảy ra.

Thuật toán chọn tập mà các client sử dụng nên gán backend một cách đồng đều nhằm tối ưu hóa việc provision tài nguyên. Chẳng hạn, nếu [subsetting làm quá tải](https://sre.google/sre-book/load-balancing-datacenter/) một backend 10%, toàn bộ tập các backends phải được provision vượt (overprovisioned) 10%. Thuật toán cũng cần xử lý các lần khởi động lại và sự cố một cách nhẹ nhàng, vững chắc bằng cách tiếp tục tải các backend đồng đều nhất có thể, đồng thời giảm thiểu sự churn (sự thay đổi). Ở đây, "churn" đề cập đến việc chọn lại backend thay thế. Ví dụ, khi một backend task không còn khả dụng, các client của nó có thể phải tạm thời chuyển sang một backend khác. Khi backend thay thế được chọn, các client phải thiết lập các kết nối TCP mới (và nhiều khả năng thực hiện đàm phán cấp ứng dụng), gây ra overhead bổ sung. Tương tự, khi một client task khởi động lại, nó cần mở lại các kết nối đến tất cả các backend của mình.

Thuật toán cũng cần xử lý các lần thay đổi kích thước (resize) số lượng client và/hoặc backend, với mức churn kết nối tối thiểu và không cần biết trước các con số đó. Tính năng này đặc biệt quan trọng (và khó khăn) khi toàn bộ tập client hoặc backend task được khởi động lại lần lượt một (ví dụ, để push một phiên bản mới). Khi các backend được push, chúng tôi muốn các client tiếp tục phục vụ một cách trong suốt, với mức churn kết nối ít nhất có thể.

### Một Thuật toán Chọn Tập: Random Subsetting (Phân tập Ngẫu nhiên) (A Subset Selection Algorithm: Random Subsetting)

Một cách cài đặt ngây thơ (naive) của thuật toán chọn tập có thể cho mỗi client xáo trộn (shuffle) danh sách các backends một lần, sau đó lấp đầy tập của mình bằng cách chọn các backend có thể phân giải/khỏe mạnh từ danh sách. Việc xáo trộn một lần rồi chọn các backend từ đầu danh sách xử lý các lần khởi động lại và sự cố một cách vững chắc (ví dụ, với sự churn tương đối ít) vì nó loại trừ rõ ràng chúng khỏi việc xem xét. Tuy nhiên, chúng tôi nhận thấy rằng chiến lược này thực tế hoạt động rất kém trong hầu hết các kịch bản vì nó trải tải rất không đều.

Trong công việc ban đầu về cân bằng tải, chúng tôi đã cài đặt random subsetting và tính toán tải dự kiến cho các trường hợp khác nhau. Như một ví dụ, hãy cân nhắc:

-   300 clients
-   300 backends
-   Một kích thước tập 30% (mỗi client kết nối đến 90 backends)

Như [Hình 20-3](#hinh-20-3) cho thấy, backend ít tải nhất chỉ chịu 63% tải trung bình (57 kết nối, trong khi mức trung bình là 90 kết nối), còn backend nhiều tải nhất chịu 121% (109 kết nối). Trong phần lớn các trường hợp, một kích thước tập 30% đã lớn hơn mức chúng tôi muốn dùng trong thực tế. Phân phối tải được tính thay đổi mỗi lần chạy mô phỏng, trong khi mẫu chung vẫn giữ nguyên.


<a id="hinh-20-3"></a>![Hình 20-3](../assets/imgs/fig-20-3.jpg)

[Hình 20-3.](#hinh-20-3) Phân phối kết nối với 300 clients, 300 backends, và một kích thước tập 30%.

Thật không may, khi giảm kích thước tập, sự mất cân bằng lại càng trầm trọng hơn. Chẳng hạn, [Hình 20-4](#hinh-20-4) cho thấy kết quả khi kích thước tập bị thu nhỏ xuống 10% (30 backend mỗi client). Lúc này, backend ít tải nhất chỉ nhận 50% tải trung bình (15 kết nối), trong khi backend nhiều tải nhất phải gánh 150% (45 kết nối).


<a id="hinh-20-4"></a>![Hình 20-4](../assets/imgs/fig-20-4.jpg)

[Hình 20-4.](#hinh-20-4) Phân phối kết nối với 300 clients, 300 backends, và một kích thước tập 10%.

Chúng tôi kết luận rằng để random subsetting phân bổ tải tương đối đều trên tất cả các task khả dụng, kích thước tập cần lớn đến 75%. Một tập lớn như vậy đơn giản là không thực tế; phương sai (variance) trong số lượng client kết nối đến một task quá lớn, khiến random subsetting không thể coi là chính sách chọn tập tốt ở quy mô lớn.

### Một Thuật toán Chọn Tập: Deterministic Subsetting (Phân tập Xác định) (A Subset Selection Algorithm: Deterministic Subsetting)

Giải pháp của Google cho các giới hạn của random subsetting là phân tập *xác định* (deterministic). Code sau đây cài đặt thuật toán này, được mô tả chi tiết bên dưới:

```python
def Subset(backends, client_id, subset_size):
  subset_count = len(backends) / subset_size

  # Group clients into rounds; each round uses the same shuffled list:
  round = client_id / subset_count
  random.seed(round)
  random.shuffle(backends)

  # The subset id corresponding to the current client:
  subset_id = client_id % subset_count

  start = subset_id * subset_size
  return backends[start:start + subset_size]
```

Chúng tôi chia các *client* task thành các "rounds" (vòng). Trong đó, `round i` bao gồm `subset_count` client task liên tiếp, bắt đầu từ task `subset_count × i`; `subset_count` là số lượng các tập (tức là số lượng backend task chia cho kích thước tập mong muốn). Mỗi vòng, mỗi backend được gán cho chính xác một client, trừ vòng cuối cùng có thể không đủ client nên một số backend không được gán.

Ví dụ, nếu có 12 backend task [0, 11] và kích thước tập mong muốn là 3, mỗi vòng sẽ chứa 4 client (`subset_count = 12/3`). Với 10 client, thuật toán trước đó có thể tạo ra *shuffled_backends* (backends đã xáo trộn) như sau:

-   Round 0: [0, 6, 3, 5, 1, 7, 11, 9, 2, 4, 8, 10]
-   Round 1: [8, 11, 4, 0, 5, 6, 10, 3, 2, 7, 9, 1]
-   Round 2: [8, 3, 7, 2, 1, 4, 9, 10, 6, 5, 0, 11]

Điểm chính cần lưu ý là trong mỗi vòng, mỗi backend trong danh sách chỉ được gán cho đúng một client (ngoại trừ vòng cuối, khi chúng tôi đã hết client). Trong ví dụ này, mọi backend được gán cho chính xác hai hoặc ba client.

Danh sách cần được xáo trộn; nếu không, các client sẽ được gán một nhóm backend task liên tiếp, và nhóm đó có thể tất cả trở nên tạm thời không khả dụng (ví dụ, vì backend job đang được cập nhật dần dần theo thứ tự, từ task đầu tiên đến task cuối cùng). Các vòng khác nhau dùng một hạt (seed) xáo trộn khác nhau. Nếu không, khi một backend thất bại, tải mà nó đang nhận chỉ được trải trên các backend còn lại *trong tập của nó*. Nếu các backend khác trong tập cũng thất bại, hiệu ứng cộng dồn và tình hình có thể nhanh chóng tồi tệ đi đáng kể: nếu `N` backend trong một tập bị xuống, tải tương ứng của chúng được trải trên (`subset_size - N`) backend còn lại. Một cách tiếp cận tốt hơn nhiều là trải tải này trên tất cả các backend còn lại bằng cách dùng một sự xáo trộn khác cho mỗi vòng.

Khi áp dụng một sự xáo trộn khác cho mỗi vòng, các client trong cùng một vòng sẽ bắt đầu với cùng một danh sách đã xáo trộn, trong khi các client ở các vòng khác sẽ có các danh sách xáo trộn khác nhau. Từ đó, thuật toán xây dựng các *định nghĩa* tập dựa trên danh sách backends đã xáo trộn và kích thước tập mong muốn. Ví dụ:

-   `Subset[0]` = `shuffled_backends[0]` đến `shuffled_backends[2]`
-   `Subset[1]` = `shuffled_backends[3]` đến `shuffled_backends[5]`
-   `Subset[2]` = `shuffled_backends[6]` đến `shuffled_backends[8]`
-   `Subset[3]` = `shuffled_backends[9]` đến `shuffled_backends[11]`

trong đó `shuffled_backends` là danh sách đã xáo trộn do mỗi client tạo ra. Để gán một tập cho một client task, chúng tôi chỉ cần lấy tập tương ứng với vị trí của nó trong vòng (ví dụ, `(i % 4)` cho `client[i]` với bốn tập):

-   `client[0]`, `client[4]`, `client[8]` sẽ sử dụng `subset[0]`
-   `client[1]`, `client[5]`, `client[9]` sẽ sử dụng `subset[1]`
-   `client[2]`, `client[6]`, `client[10]` sẽ sử dụng `subset[2]`
-   `client[3]`, `client[7]`, `client[11]` sẽ sử dụng `subset[3]`

Vì các client ở các vòng khác nhau sẽ dùng giá trị khác cho `shuffled_backends` (và do đó cho `subset`), trong khi các client trong cùng một vòng dùng các tập khác nhau, tải kết nối được phân bổ đều. Khi tổng số backend không chia hết cho kích thước tập mong muốn, chúng tôi cho phép một số tập lớn hơn một chút so với các tập khác; tuy nhiên, trong phần lớn các trường hợp, số lượng client gán cho mỗi backend chỉ chênh lệch tối đa là 1.

Như [Hình 20-5](#hinh-20-5) cho thấy, với ví dụ 300 client trước đó, mỗi client kết nối đến 10 trong số 300 backends, kết quả đạt được rất tốt: mỗi backend nhận đúng một lượng kết nối như nhau.


<a id="hinh-20-5"></a>![Hình 20-5](../assets/imgs/fig-20-5.jpg)

[Hình 20-5.](#hinh-20-5) Phân phối kết nối với 300 clients và deterministic subsetting đến 10 của 300 backends.

## Các Chính sách Cân bằng Tải (Load Balancing Policies)

Bây giờ khi đã thiết lập nền tảng cho cách một client task duy trì một tập các kết nối được biết là khỏe mạnh, hãy xem xét *các chính sách cân bằng tải*. Đây là các cơ chế mà các client task sử dụng để chọn backend task nào trong tập của mình nhận một yêu cầu client. Nhiều sự phức tạp trong các chính sách cân bằng tải bắt nguồn từ bản chất phân tán của quá trình ra quyết định: các client phải quyết định, trong thời gian thực (và chỉ với thông tin trạng thái backend một phần và/hoặc cũ), backend nào nên được dùng cho mỗi yêu cầu.

Các chính sách cân bằng tải có thể rất đơn giản, không dựa vào bất kỳ thông tin nào về trạng thái của các backend (ví dụ, *Round Robin* — Vòng quay), hoặc có thể vận hành dựa trên nhiều thông tin hơn về các backend (ví dụ, *Least-Loaded Round Robin* — Vòng quay ít tải nhất, hoặc *Weighted Round Robin* — Vòng quay có trọng số).

### Round Robin Đơn giản (Simple Round Robin)

Một cách tiếp cận rất đơn giản cho [cân bằng tải](https://sre.google/sre-book/handling-overload/) là mỗi client gửi yêu cầu luân phiên đến các backend task trong tập mà nó kết nối thành công và không ở trạng thái vịt què. Trong nhiều năm, đây là cách tiếp cận phổ biến nhất của chúng tôi, và nhiều dịch vụ vẫn đang sử dụng.

Thật không may, dù Round Robin rất đơn giản và hiệu quả hơn đáng kể so với việc chọn ngẫu nhiên các backend task, kết quả của chính sách này lại có thể rất kém. Mặc dù các con số thực tế phụ thuộc vào nhiều yếu tố như chi phí truy vấn thay đổi và sự đa dạng máy, chúng tôi nhận thấy rằng Round Robin có thể khiến mức tiêu thụ CPU giữa task ít tải nhất và task nhiều tải nhất chênh lệch lên đến 2 lần. Sự chênh lệch như vậy cực kỳ lãng phí và xảy ra vì một số lý do, bao gồm:

-   Subsetting nhỏ
-   Chi phí truy vấn thay đổi
-   Sự đa dạng máy
-   Các yếu tố hiệu năng không thể dự đoán

#### Subsetting nhỏ (Small subsetting)

Một trong những lý do đơn giản nhất khiến Round Robin phân phối tải kém là các client không phát ra yêu cầu với cùng một tốc độ. Sự chênh lệch tốc độ này đặc biệt dễ xảy ra khi các tiến trình khác biệt lớn cùng chia sẻ các backend. Trong trường hợp đó, nhất là khi bạn dùng các kích thước tập tương đối nhỏ, các backend trong tập của những client tạo ra nhiều traffic nhất sẽ tự nhiên chịu tải nhiều hơn.

#### Chi phí truy vấn thay đổi (Varying query costs)

Nhiều dịch vụ phải xử lý các yêu cầu có mức độ tiêu thụ tài nguyên xử lý chênh lệch rất lớn. Trong thực tế, chúng tôi nhận thấy ngữ nghĩa của nhiều dịch vụ tại Google khiến các yêu cầu đắt nhất tiêu thụ CPU gấp 1000 lần (hoặc hơn) so với các yêu cầu rẻ nhất. Cân bằng tải theo Round Robin càng trở nên khó khăn hơn khi chi phí truy vấn không thể dự đoán trước. Chẳng hạn, một truy vấn dạng "trả về tất cả email mà user XYZ nhận được trong ngày qua" có thể rất rẻ (nếu user nhận ít email trong ngày) hoặc cực kỳ đắt tiền.

Cân bằng tải trong hệ thống có sự chênh lệch lớn về chi phí truy vấn tiềm tàng là một bài toán rất khó. Đôi khi, bạn phải điều chỉnh các giao diện (interfaces) dịch vụ để giới hạn khối lượng công việc cho mỗi yêu cầu. Lấy ví dụ từ trường hợp truy vấn email đã đề cập trước đó, bạn có thể thêm giao diện phân trang (pagination) và thay đổi ngữ nghĩa yêu cầu thành "trả về 100 email gần đây nhất (hoặc ít hơn) mà user XYZ nhận được trong ngày qua." Tuy nhiên, việc đưa ra những thay đổi ngữ nghĩa như vậy thường không dễ dàng. Không chỉ đòi hỏi phải sửa đổi toàn bộ client code, mà nó còn kéo theo các cân nhắc bổ sung về tính nhất quán (consistency). Chẳng hạn, user có thể đang nhận email mới hoặc xóa email trong khi client đang lấy email từng trang. Với use case này, một client ngây thơ lặp lại qua các kết quả và nối các phản hồi (thay vì phân trang dựa trên một tầm nhìn cố định của dữ liệu) nhiều khả năng sẽ tạo ra một tầm nhìn không nhất quán, dẫn đến việc lặp lại một số thông điệp và/hoặc bỏ sót những thông điệp khác.

Để giữ cho các giao diện (cùng với cấu hình của chúng) đơn giản, các dịch vụ thường được định nghĩa sao cho những yêu cầu tốn kém nhất được phép tiêu thụ nhiều tài nguyên hơn gấp 100, 1.000, hoặc thậm chí 10.000 lần so với những yêu cầu rẻ nhất. Tuy nhiên, việc mức tiêu thụ tài nguyên thay đổi theo từng yêu cầu đồng nghĩa với việc một số backend task sẽ xui xẻo và thỉnh thoảng nhận nhiều yêu cầu đắt tiền hơn những task khác. Mức độ ảnh hưởng của tình trạng này đến cân bằng tải phụ thuộc vào việc các yêu cầu đắt tiền nhất tốn kém đến mức nào. Ví dụ, với một trong những backends Java của chúng tôi, các truy vấn tiêu thụ trung bình khoảng 15 ms CPU, nhưng một số truy vấn có thể dễ dàng đòi hỏi lên đến 10 giây. Mỗi task trong backend này đặt trước nhiều CPU core (nhân), điều này giúp giảm độ trễ bằng cách cho phép một số phép tính diễn ra song song. Nhưng bất chấp những core đặt trước này, khi một backend nhận một trong những truy vấn lớn, tải của nó tăng đáng kể trong vài giây. Một task hoạt động kém có thể hết bộ nhớ hoặc thậm chí ngừng phản hồi hoàn toàn (ví dụ, do memory thrashing — dao động bộ nhớ), nhưng ngay cả trong trường hợp bình thường (tức là backend có đủ tài nguyên và tải của nó trở lại bình thường một khi truy vấn lớn hoàn thành), độ trễ của các yêu cầu khác vẫn bị ảnh hưởng do sự cạnh tranh tài nguyên với yêu cầu đắt tiền.

#### Sự đa dạng máy (Machine diversity)

Một thách thức khác của Simple Round Robin là thực tế rằng không phải tất cả các máy trong cùng một datacenter nhất thiết giống nhau. Một datacenter có thể chứa các máy với CPU hiệu năng khác nhau, do đó cùng một yêu cầu có thể tương ứng với lượng công việc khác nhau đáng kể trên các máy khác nhau.

Việc ứng phó với sự đa dạng máy — mà *không* đòi hỏi sự đồng nhất nghiêm ngặt — là một thách thức kéo dài nhiều năm tại Google. Về mặt lý thuyết, giải pháp cho việc làm việc với năng lực tài nguyên dị thể (heterogeneous) trong một hạm đội (fleet) là đơn giản: scale các đặt trước CPU tùy theo loại trình xử lý/máy. Tuy nhiên, trong thực tế, việc triển khai giải pháp này đòi hỏi nỗ lực đáng kể, vì nó yêu cầu trình lên lịch job của chúng tôi phải tính đến các mức tương đương tài nguyên dựa trên hiệu năng máy trung bình qua việc lấy mẫu các dịch vụ. Ví dụ, 2 đơn vị CPU trong máy X (một máy "chậm") tương đương với 0,8 đơn vị CPU trong máy Y (một máy "nhanh"). Với thông tin này, trình lên lịch job sau đó được yêu cầu điều chỉnh các đặt trước CPU cho một tiến trình dựa trên hệ số tương đương và loại máy mà tiến trình được lên lịch. Để giảm nhẹ độ phức tạp này, chúng tôi đã tạo ra một đơn vị ảo cho tốc độ CPU gọi là "GCU" (Google Compute Units — Đơn vị Tính toán Google). GCU trở thành tiêu chuẩn để mô hình hóa tốc độ CPU, và được dùng để duy trì một ánh xạ từ mỗi kiến trúc CPU trong các datacenter của chúng tôi đến GCU tương ứng dựa trên hiệu năng của nó.

#### Các yếu tố hiệu năng không thể dự đoán (Unpredictable performance factors)

Có lẽ điều khiến Simple Round Robin trở nên phức tạp nhất là các máy — hay chính xác hơn, hiệu năng của các backend task — có thể chênh lệch rất lớn do một số yếu tố *không thể dự đoán*, thứ mà ta không thể tính toán trước một cách tĩnh.

Hai trong số nhiều yếu tố không thể dự đoán góp phần vào hiệu năng bao gồm:

#### Các hàng xóm đối kháng (Antagonistic neighbors)

Các tiến trình khác (thường hoàn toàn không liên quan và do các đội khác vận hành) có thể ảnh hưởng đáng kể đến hiệu năng của tiến trình bạn. Chúng tôi từng ghi nhận sự chênh lệch hiệu năng lên tới 20% do nguyên nhân này. Phần lớn sự khác biệt bắt nguồn từ việc cạnh tranh các tài nguyên chia sẻ, chẳng hạn không gian trong memory cache (bộ đệm bộ nhớ) hoặc băng thông, theo những cách có thể không trực tiếp và rõ ràng. Ví dụ, nếu độ trễ của các yêu cầu đến từ một backend task tăng lên (do cạnh tranh tài nguyên mạng với một hàng xóm đối kháng), số lượng yêu cầu đang hoạt động cũng sẽ tăng, từ đó có thể kích hoạt việc thu gom rác (garbage collection) nhiều hơn.

#### Các lần khởi động lại task (Task restarts)

Khi một task được khởi động lại, nó thường tiêu tốn nhiều tài nguyên hơn đáng kể trong vài phút. Ví dụ, chúng tôi nhận thấy hiện tượng này ảnh hưởng đến các nền tảng như Java (tối ưu hóa code động) nhiều hơn so với những nền tảng khác. Để ứng phó, chúng tôi đã thêm vào logic của một số server: giữ các server ở trạng thái vịt què và làm ấm chúng trước (prewarm, kích hoạt các tối ưu hóa này) trong một khoảng thời gian sau khi khởi động, cho đến khi hiệu năng đạt mức danh nghĩa. Hiệu ứng của các lần khởi động lại task có thể trở thành vấn đề đáng kể, đặc biệt khi cân nhắc rằng chúng tôi cập nhật nhiều server (ví dụ, đẩy các build mới lên, đòi hỏi khởi động lại những task này) mỗi ngày.

Nếu chính sách cân bằng tải của bạn không thể thích nghi với các hạn chế hiệu năng không lường trước, bạn sẽ về cơ bản kết thúc với một sự phân phối tải không tối ưu khi vận hành ở quy mô.

### Least-Loaded Round Robin (Vòng quay ít tải nhất) (Least-Loaded Round Robin)

Một cách tiếp cận thay thế cho Simple Round Robin là để mỗi client task theo dõi số lượng yêu cầu đang hoạt động đến từng backend task trong tập của nó, sau đó áp dụng Round Robin *trong số các task có ít yêu cầu đang hoạt động nhất*.

Ví dụ, giả sử một client dùng một tập các backend task *t0* đến *t9*, và hiện tại có số lượng yêu cầu đang hoạt động sau đây cho mỗi backend:

**t0** | **t1** | **t2** | **t3** | **t4** | **t5** | **t6** | **t7** | **t8** | **t9**
---|---|---|---|---|---|---|---|---|---
`2` | `1` | `0` | `0` | `1` | `0` | `2` | `0` | `0` | `1`

Khi có yêu cầu mới, client sẽ lọc danh sách các backend task tiềm năng, chỉ giữ lại những task có ít kết nối nhất (*t2*, *t3*, *t5*, *t7*, và *t8*), sau đó chọn một backend từ danh sách này. Giả sử nó chọn *t2*. Bảng trạng thái kết nối của client lúc này sẽ như sau:

**t0** | **t1** | **t2** | **t3** | **t4** | **t5** | **t6** | **t7** | **t8** | **t9**
---|---|---|---|---|---|---|---|---|---
`2` | `1` | `1` | `0` | `1` | `0` | `2` | `0` | `0` | `1`

Giả sử không có yêu cầu hiện tại nào đã hoàn thành, ở yêu cầu tiếp theo, bể ứng viên backend trở thành *t3*, *t5*, *t7*, và *t8*.

Hãy tua nhanh đến khi chúng tôi đã phát ra bốn yêu cầu mới. Vẫn giả sử không có yêu cầu nào kết thúc trong khoảng thời gian đó, bảng trạng thái kết nối sẽ trông như sau:

**t0** | **t1** | **t2** | **t3** | **t4** | **t5** | **t6** | **t7** | **t8** | **t9**
---|---|---|---|---|---|---|---|---|---
`2` | `1` | `1` | `1` | `1` | `1` | `2` | `1` | `1` | `1`

Lúc này, tập ứng viên backend bao gồm toàn bộ task, trừ *t0* và *t6*. Tuy nhiên, nếu yêu cầu của task *t4* kết thúc, trạng thái hiện tại của nó sẽ chuyển thành "0 yêu cầu đang hoạt động" và một yêu cầu mới sẽ được gán cho *t4*.

Cài đặt này thực sự dùng Round Robin, nhưng nó được áp dụng xuyên suốt tập các task có số yêu cầu đang hoạt động nhỏ nhất. Nếu không có cơ chế lọc này, chính sách có thể không phân bổ yêu cầu đủ đều, dẫn đến việc một phần các backend task khả dụng không được sử dụng. Ý tưởng đằng sau chính sách ít tải nhất là các task nhiều tải thường có độ trễ cao hơn so với những task còn năng lực dự phòng, và chiến lược này sẽ tự động giảm tải cho những task nhiều tải đó.

Tất cả những gì đã nói, chúng tôi đã học (theo cách khó khăn!) về một cạm bẫy rất nguy hiểm của cách tiếp cận Least-Loaded Round Robin: nếu một task thực sự không khỏe mạnh, nó có thể bắt đầu phục vụ 100% là lỗi. Tùy thuộc vào bản chất của những lỗi đó, chúng có thể có độ trễ rất thấp; việc đơn giản trả về một lỗi "tôi không khỏe mạnh!" thường nhanh hơn đáng kể so với việc thực sự xử lý một yêu cầu. Kết quả là, các client có thể bắt đầu gửi một lượng traffic rất lớn đến task không khỏe mạnh, nhầm tưởng rằng task khả dụng, thay vì làm thất bại các yêu cầu đó nhanh chóng (fast-fail)! Chúng tôi nói rằng task không khỏe mạnh lúc này đang *sinkholing* (bắt bẫy) traffic. May mắn thay, cạm bẫy này có thể được giải quyết tương đối dễ dàng bằng cách sửa đổi chính sách để đếm các lỗi gần đây như thể chúng là các yêu cầu đang hoạt động. Bằng cách này, nếu một backend task trở nên không khỏe mạnh, chính sách cân bằng tải sẽ bắt đầu chuyển tải khỏi nó, giống như cách nó chuyển tải khỏi một task bị quá gánh.

Least-Loaded Round Robin có hai giới hạn quan trọng:

**Số đếm các yêu cầu đang hoạt động có thể không phải là một chỉ số đại diện (proxy) rất tốt cho khả năng của một backend nhất định**

Nhiều yêu cầu dành phần đáng kể thời gian sống của chúng chỉ để chờ phản hồi từ mạng (tức là chờ các phản hồi cho các yêu cầu mà chúng khởi tạo đến các backend khác) và rất ít thời gian cho việc xử lý thực. Ví dụ, một backend task có thể xử lý gấp đôi số yêu cầu so với một task khác (ví dụ, vì nó chạy trên một máy với CPU nhanh gấp đôi), nhưng độ trễ của các yêu cầu của nó có thể vẫn xấp xỉ giống như độ trễ của các yêu cầu trong task khác (vì các yêu cầu dành phần lớn thời gian chỉ để chờ mạng phản hồi). Trong trường hợp này, vì việc chặn (blocking) trên I/O (nhập/xuất) thường tiêu thụ zero CPU, rất ít RAM, và không dùng băng thông, chúng tôi vẫn muốn gửi gấp đôi số yêu cầu đến backend nhanh hơn. Tuy nhiên, Least-Loaded Round Robin sẽ coi cả hai backend task là nhiều tải như nhau.

**Số đếm các yêu cầu đang hoạt động trong mỗi client không bao gồm các yêu cầu từ các client khác đến cùng các backends**

Tức là, mỗi client task chỉ có một tầm nhìn rất hạn chế vào trạng thái của các backend task của nó: chỉ qua các yêu cầu của chính nó.

Thực tế cho thấy, với các dịch vụ lớn áp dụng Least-Loaded Round Robin, backend task chịu tải nặng nhất tiêu thụ gấp đôi CPU so với task ít tải nhất, khiến hiệu năng hoạt động kém xấp xỉ Round Robin.

### Weighted Round Robin (Vòng quay có trọng số) (Weighted Round Robin)

Weighted Round Robin là chính sách cân bằng tải quan trọng, nâng cao hơn Simple và Least-Loaded Round Robin nhờ đưa thông tin do backend cung cấp vào quá trình ra quyết định.

Weighted Round Robin khá đơn giản về mặt nguyên lý: mỗi client task giữ một điểm "khả năng" (capability) cho từng backend trong tập của nó. Các yêu cầu được phân phối theo kiểu vòng quay, nhưng các client trọng số hóa sự phân phối này đến các backend một cách tỷ lệ. Trong mỗi phản hồi (bao gồm cả các phản hồi cho kiểm tra sức khỏe), các backend cung cấp tốc độ quan sát hiện tại của số truy vấn và lỗi mỗi giây, cùng với mức sử dụng (utilization, thường là mức dùng CPU). Các client điều chỉnh định kỳ các điểm khả năng để chọn backend task dựa trên số lượng yêu cầu thành công hiện tại đã xử lý và mức chi phí sử dụng tương ứng; các yêu cầu thất bại sẽ gây ra một hình phạt, ảnh hưởng đến các quyết định trong tương lai.

Trong thực tế, Weighted Round Robin đã hoạt động rất tốt và giảm đáng kể sự khác biệt giữa task được sử dụng nhiều nhất và ít nhất. [Hình 20-6](#hinh-20-6) hiển thị các tốc độ CPU cho một tập ngẫu nhiên các backend task quanh thời điểm các client của nó chuyển từ Least-Loaded sang Weighted Round Robin. Sự trải rộng từ task ít tải nhất đến task nhiều tải nhất giảm mạnh.


<a id="hinh-20-6"></a>![Hình 20-6](../assets/imgs/fig-20-6.jpg)

[Hình 20-6.](#hinh-20-6) Phân phối CPU trước và sau khi bật Weighted Round Robin.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
