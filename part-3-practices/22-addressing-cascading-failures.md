# Chương 22. Đối phó với Các Sự cố Lan truyền (Addressing Cascading Failures)

> **Nguyên bản:** [Chapter 22 - Addressing Cascading Failures](https://sre.google/sre-book/addressing-cascading-failures/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Mike Ulrich

> Nếu lần đầu tiên bạn không thành công, hãy lùi lại theo cấp số nhân.
>
> Dan Sandler, Google Software Engineer (Kỹ sư Phần mềm Google)

> Tại sao mọi người luôn quên rằng bạn cần thêm một chút jitter (sự nhiễu ngẫu nhiên)?
>
> Ade Oshineye, Google Developer Advocate (Nhà truyền bá Developer của Google)

Sự cố lan truyền (cascading failure) là sự cố ngày càng trầm trọng theo thời gian do cơ chế phản hồi tích cực (positive feedback).<sup>[1](#fn1)</sup> Hiện tượng này xảy ra khi một phần của hệ thống gặp sự cố, khiến xác suất các phần khác cũng gặp sự cố tăng lên. Chẳng hạn, một replica (bản sao) đơn lẻ của dịch vụ có thể bị quá tải (overload) và gặp sự cố, điều này làm tăng tải lên các replica còn lại, khiến chúng cũng dễ gặp sự cố hơn, tạo thành hiệu ứng domino kéo toàn bộ các replica của dịch vụ đi xuống.

Trong chương này, chúng tôi sẽ dùng dịch vụ tìm kiếm Shakespeare (đã đề cập tại [Shakespeare: A Sample Service](https://sre.google/sre-book/production-environment#xref_production-environment_shakespeare)) làm ví dụ xuyên suốt. Cấu hình production (sản xuất) của dịch vụ này có thể như [Hình 22-1](#hinh-22-1).


<a id="hinh-22-1"></a>![Hình 22-1](../assets/imgs/fig-22-1.jpg)

[Hình 22-1.](#hinh-22-1) Ví dụ cấu hình production cho dịch vụ tìm kiếm Shakespeare.

## Các Nguyên nhân của Các Sự cố Lan truyền và Thiết kế để Tránh chúng (Causes of Cascading Failures and Designing to Avoid Them)

Thiết kế hệ thống có tính toán kỹ lưỡng nên bao quát một số kịch bản điển hình giải thích cho phần lớn các sự cố lan truyền.

<a id="server-qua-tai"></a>

## Quá tải Server (Server Overload)

Quá tải là nguyên nhân phổ biến nhất của các sự cố lan truyền. Phần lớn các sự cố lan truyền được mô tả ở đây hoặc do trực tiếp server quá tải, hoặc do các dạng mở rộng hay biến thể của kịch bản này.

Giả sử frontend trong cluster (cụm máy) A đang xử lý 1.000 yêu cầu mỗi giây (QPS), như trong [Hình 22-2](#hinh-22-2).


<a id="hinh-22-2"></a>![Hình 22-2](../assets/imgs/fig-22-2.jpg)

[Hình 22-2.](#hinh-22-2) Sự phân phối tải server bình thường giữa các cluster A và B.

Nếu cluster B gặp sự cố ([Hình 22-3](#hinh-22-3)), lượng yêu cầu đổ về cluster A tăng lên 1.200 QPS. Các frontend trong A không thể xử lý ở mức 1.200 QPS, dẫn đến cạn kiệt tài nguyên, gây sập (crash), bỏ lỡ các hạn chót (deadlines) hoặc hành xử sai theo cách khác. Hệ quả là, tốc độ xử lý thành công các yêu cầu trong A giảm xuống đáng kể, dưới 1.000 QPS.


<a id="hinh-22-3"></a>![Hình 22-3](../assets/imgs/fig-22-3.jpg)

[Hình 22-3.](#hinh-22-3) Cluster B gặp sự cố, gửi tất cả traffic đến cluster A.

Sự sụt giảm này trong tốc độ xử lý công việc hữu ích có thể lan sang các failure domain khác, thậm chí gây ra sự cố toàn cục. Chẳng hạn, quá tải cục bộ trong một cluster có thể khiến các server của nó sập; để ứng phó, bộ điều khiển cân bằng tải (load balancing controller) chuyển các yêu cầu sang các cluster khác, làm quá tải các server ở đó và dẫn đến sự cố quá tải toàn dịch vụ. Những sự kiện này có thể diễn ra rất nhanh (ví dụ, chỉ trong vòng vài phút), vì [bộ cân bằng tải](https://sre.google/sre-book/handling-overload/) và các hệ thống lên lịch task (nhiệm vụ) liên quan có thể phản ứng tức thì.

## Hết Tài nguyên (Resource Exhaustion)

Việc cạn tài nguyên có thể khiến độ trễ (latency) tăng, tốc độ lỗi (error rates) cao hơn, hoặc xuất hiện các kết quả kém chất lượng. Đây thực chất là những hiệu ứng được mong đợi khi tài nguyên cạn: rốt cuộc, khi tải vượt quá khả năng xử lý của server, một thứ gì đó phải nhượng bộ.

Tùy vào loại tài nguyên nào cạn kiệt trên server và cách server được cấu hình, tình trạng cạn tài nguyên có thể khiến server hoạt động kém hiệu quả hơn hoặc dẫn đến sập, buộc bộ cân bằng tải phải chuyển hướng lưu lượng sang các server khác. Khi điều này xảy ra, tốc độ xử lý các yêu cầu thành công có thể giảm, thậm chí kéo cả cluster hoặc toàn bộ dịch vụ vào một sự cố lan truyền.

Các loại tài nguyên khác nhau có thể cạn, dẫn đến các hiệu ứng khác nhau lên các server.

### CPU

Nếu không có đủ CPU để xử lý tải yêu cầu, thường tất cả các yêu cầu sẽ trở nên chậm hơn. Kịch bản này có thể dẫn đến các hiệu ứng thứ cấp khác nhau, bao gồm:

#### Tăng số lượng các yêu cầu đang bay (in-flight requests)

Do thời gian xử lý mỗi yêu cầu kéo dài, hệ thống phải xử lý đồng thời nhiều yêu cầu hơn (cho đến khi chạm ngưỡng năng lực tối đa, lúc đó hiện tượng xếp hàng có thể xảy ra). Điều này tác động đến gần như mọi loại tài nguyên, bao gồm bộ nhớ, số lượng thread (luồng) đang hoạt động (trong mô hình server thread-mỗi-yêu-cầu), số lượng file descriptor (mô tả tệp), và các tài nguyên backend (những tài nguyên mà lần lượt có thể gây ra các hiệu ứng khác).

**Các độ dài hàng đợi (queue) quá dài**

Nếu server không đủ năng lực xử lý toàn bộ yêu cầu ở trạng thái ổn định (steady state), các hàng đợi của nó sẽ bị bão hòa. Hệ quả là độ trễ tăng (yêu cầu phải xếp hàng lâu hơn) và hàng đợi chiếm nhiều bộ nhớ hơn. Xem [Quản lý Hàng đợi](#quan-ly-hang-doi) để tìm hiểu các chiến lược giảm nhẹ.

#### Thread đói (Thread starvation)

Khi một thread bị chặn do chờ lock (khóa), các health checks có thể thất bại nếu endpoint (điểm cuối) kiểm tra sức khỏe không kịp phục vụ.

#### CPU hoặc yêu cầu bị đói (CPU or request starvation)

Các watchdog (chó canh) nội bộ<sup>[2](#fn2)</sup> trong server phát hiện server không tiến triển được, dẫn đến sập server do CPU bị đói, hoặc do yêu cầu bị đói nếu các sự kiện watchdog được kích hoạt từ xa và xử lý như một phần của hàng đợi yêu cầu.

#### Bỏ lỡ các hạn chót RPC (Remote Procedure Call — lời gọi thủ tục từ xa)

Khi server quá tải, phản hồi cho các RPC từ client sẽ đến muộn hơn, có thể vượt qua hạn chót mà client đặt ra. Công việc server đã thực hiện để xử lý các phản hồi đó bị lãng phí, trong khi client có thể thử lại RPC, khiến tình trạng quá tải trở nên trầm trọng hơn.

#### Giảm lợi ích của cache CPU (CPU caching)

Khi số CPU tăng, khả năng tràn (spilling) sang nhiều core (nhân) hơn cũng tăng, khiến việc sử dụng cache (bộ đệm) cục bộ giảm và hiệu năng CPU suy giảm.

### Bộ nhớ (Memory)

Nếu không có gì khác, số lượng yêu cầu tăng cao sẽ tiêu tốn nhiều RAM (bộ nhớ truy cập trực tiếp) hơn do việc cấp phát (allocating) các object (đối tượng) cho yêu cầu, phản hồi, và RPC. Tình trạng cạn bộ nhớ có thể gây ra các hiệu ứng sau:

#### Các task chết (dying tasks)

Ví dụ, một task có thể bị trình quản lý container (hộp) (VM hoặc khác) trục xuất (evicted) do vượt quá giới hạn tài nguyên khả dụng, hoặc các sự cố sập cụ thể của ứng dụng có thể khiến task chết.

**Tăng tốc độ thu gom rác (garbage collection, GC) trong Java, dẫn đến tăng việc sử dụng CPU**

Trong kịch bản này, một vòng lặp luẩn quẩn (vicious cycle) có thể xảy ra: CPU khả dụng giảm khiến yêu cầu xử lý chậm hơn, kéo theo mức dùng RAM tăng, khiến GC chạy nhiều hơn, và điều đó lại làm CPU khả dụng giảm thêm. Trong ngôn ngữ thông tục, hiện tượng này được gọi là "xoáy cái chết GC" (GC death spiral).

#### Giảm các tỷ lệ hit cache (cache hit rates)

RAM khả dụng giảm có thể làm giảm tỷ lệ hit cache cấp ứng dụng, dẫn đến số lượng RPC gửi đến các backend tăng lên và có thể khiến các backend quá tải.

### Threads (Luồng)

Thread bị đói có thể trực tiếp gây lỗi hoặc khiến kiểm tra sức khỏe thất bại. Nếu server thêm thread khi cần, overhead thread có thể ngốn quá nhiều RAM. Trong các trường hợp cực đoan, thread bị đói thậm chí có thể khiến bạn cạn hết process ID (định danh tiến trình).

### File descriptors (Mô tả tệp) (File descriptors)

Việc cạn các file descriptor có thể dẫn đến không thể khởi tạo các kết nối mạng, và điều này lần lượt có thể khiến các kiểm tra sức khỏe thất bại.

### Các Sự phụ thuộc giữa các Tài nguyên (Dependencies among resources)

Cần lưu ý rằng nhiều kịch bản cạn tài nguyên có thể làm trầm trọng thêm lẫn nhau — một [dịch vụ đang quá tải](https://sre.google/sre-book/handling-overload/) thường xuất hiện hàng loạt triệu chứng thứ cấp trông giống nguyên nhân gốc rễ (root cause), khiến việc debug (gỡ lỗi) trở nên khó khăn.

Ví dụ, hãy hình dung kịch bản sau:

1.  Một frontend Java có các tham số thu gom rác (GC) được tinh chỉnh kém.
2.  Dưới tải cao (nhưng vẫn nằm trong dự kiến), frontend cạn CPU do GC.
3.  Việc cạn CPU làm chậm việc hoàn thành các yêu cầu.
4.  Số lượng yêu cầu đang tiến hành tăng lên khiến nhiều RAM hơn được sử dụng để xử lý các yêu cầu.
5.  Áp lực bộ nhớ từ các yêu cầu, kết hợp với một mức cấp phát bộ nhớ cố định cho tiến trình frontend nói chung, để lại ít RAM hơn cho cache.
6.  Kích thước cache giảm đồng nghĩa với ít entry (bản ghi) trong cache hơn, bên cạnh tỷ lệ hit thấp hơn.
7.  Sự tăng trong số lần bỏ lỡ cache (cache miss) đồng nghĩa với việc nhiều yêu cầu hơn rơi xuống (fall through) đến backend để được phục vụ.
8.  Backend, lần lượt, cạn CPU hoặc thread.
9.  Cuối cùng, sự thiếu CPU khiến các kiểm tra sức khỏe cơ bản thất bại, khởi đầu một sự cố lan truyền.

Trong những tình huống phức tạp như kịch bản trên, khó có thể chẩn đoán trọn vẹn chuỗi nhân quả chỉ trong một lần outage (mất dịch vụ). Việc xác định rằng sự sập backend bắt nguồn từ một sự giảm tỷ lệ cache trong frontend có thể rất khó khăn, đặc biệt khi các thành phần frontend và backend thuộc về những chủ sở hữu khác nhau.

## Sự Không khả dụng của Dịch vụ (Service Unavailability)

Việc cạn tài nguyên có thể khiến các server sập; ví dụ, khi quá nhiều RAM được cấp phát cho một container, server có thể sập. Một khi vài server sập do quá tải, tải trên các server còn lại sẽ tăng lên, khiến chúng cũng sập. Vấn đề có xu hướng lan rộng như quả bóng tuyết và chẳng bao lâu tất cả các server bắt đầu crash-loop (vòng lặp sập). Thường rất khó để thoát khỏi kịch bản này, bởi vì ngay khi các server quay lại online, chúng lập tức bị dồn một tỷ lệ yêu cầu cực kỳ cao và gần như ngay lập tức gặp sự cố.

Ví dụ, nếu một dịch vụ đang vận hành ổn định ở 10.000 QPS, nhưng bắt đầu xảy ra sự cố lan truyền do các lần sập ở mức 11.000 QPS, việc giảm tải xuống 9.000 QPS gần như chắc chắn sẽ không ngăn được các lần sập. Nguyên nhân là dịch vụ vẫn phải xử lý nhu cầu tăng lên trong khi sức chứa đã giảm; thường chỉ một phần nhỏ các server đủ khỏe để xử lý yêu cầu. Tỷ lệ server đủ khỏe phụ thuộc vào một số yếu tố: tốc độ hệ thống khởi chạy các task, tốc độ binary bắt đầu phục vụ ở sức chứa đầy đủ, và thời gian một task mới khởi chạy có thể trụ được trước tải. Trong ví dụ này, nếu 10% server đủ khỏe để xử lý yêu cầu, thì tỷ lệ yêu cầu sẽ cần giảm xuống khoảng 1.000 QPS để hệ thống có thể ổn định và phục hồi.

Tương tự, các server có thể hiển thị trạng thái không khỏe đối với tầng cân bằng tải, khiến sức chứa của tầng này giảm: server có thể chuyển sang trạng thái “lame duck” (xem [Một Cách tiếp cận Vững chắc cho các Task Không khỏe mạnh: Trạng thái Lame Duck](https://sre.google/sre-book/load-balancing-datacenter/#robust-approach-unhealthy-tasks-lame-duck-state)) hoặc thất bại các kiểm tra sức khỏe mà không sập. Hiện tượng này có thể rất giống với việc sập: nhiều server hơn hiển thị trạng thái không khỏe, các server khỏe chỉ chấp nhận yêu cầu trong một khoảng thời gian rất ngắn trước khi trở nên không khỏe, và ít server hơn tham gia xử lý yêu cầu.

Các chính sách cân bằng tải né tránh những server đã trả về lỗi có thể khiến vấn đề trầm trọng hơn — một vài backend đang gặp lỗi nên không còn đóng góp vào sức chứa khả dụng của dịch vụ. Điều này làm tăng tải lên các server còn lại, tạo ra hiệu ứng tuyết lăn.

<a id="phong-tranh-server-qua-tai"></a>

## Phòng Tránh Server Quá Tải (Preventing Server Overload)

Danh sách dưới đây trình bày các chiến lược [tránh server quá tải](https://sre.google/sre-book/handling-overload/) theo thứ tự ưu tiên xấp xỉ:

Kiểm thử tải các giới hạn sức chứa của server, và kiểm thử failure mode khi quá tải

Đây là bài tập quan trọng nhất bạn nên thực hiện để phòng tránh server quá tải. Trừ khi kiểm thử trong môi trường thực tế, rất khó để dự đoán chính xác tài nguyên nào sẽ cạn và việc cạn kiệt đó sẽ biểu hiện ra sao. Để biết chi tiết, xem [Kiểm thử cho các Sự cố Lan truyền](#kiem-thu-cho-cac-su-that-bai-lan-truyen).

Phục vụ các kết quả suy giảm

Cung cấp cho người dùng các kết quả kém chất lượng hơn nhưng rẻ hơn về chi phí tính toán. Chiến lược cụ thể sẽ tùy thuộc vào dịch vụ. Xem [Gánh nhẹ Tải và Suy giảm Nhẹ nhàng](#ganh-nhe-tai-va-suy-giam-nhe-nhan).

Trang bị (instrument) server để từ chối yêu cầu khi quá tải

Các server cần tự bảo vệ mình khỏi tình trạng quá tải và sập. Khi quá tải xảy ra ở tầng frontend hoặc backend, hãy thất bại sớm và rẻ. Để biết chi tiết, xem [Gánh nhẹ Tải và Suy giảm Nhẹ nhàng](#ganh-nhe-tai-va-suy-giam-nhe-nhan).

Trang bị các hệ thống cấp cao hơn để từ chối yêu cầu, thay vì làm các server quá tải

Lưu ý rằng vì giới hạn tốc độ (rate limiting) thường không xét đến sức khỏe tổng thể của dịch vụ, nó có thể không ngăn được sự cố đã khởi đầu. Các cài đặt giới hạn tốc độ đơn giản cũng thường để lại sức chứa không được sử dụng. Việc giới hạn tốc độ có thể được thực hiện ở một số nơi:

-   *Tại các reverse proxy*, giới hạn khối lượng yêu cầu dựa trên các tiêu chí như địa chỉ IP nhằm giảm nhẹ các cuộc tấn công phủ nhận dịch vụ (denial-of-service) và các client lạm dụng.
-   *Tại các load balancer*, bằng cách từ chối các yêu cầu khi dịch vụ rơi vào trạng thái quá tải toàn cục. Tùy thuộc vào bản chất và độ phức tạp của dịch vụ, cơ chế giới hạn tốc độ này có thể áp dụng theo cách không phân biệt ("bỏ mọi traffic vượt X yêu cầu mỗi giây") hoặc có chọn lọc hơn ("bỏ các yêu cầu không đến từ những người dùng gần đây đã tương tác với dịch vụ" hoặc "bỏ các yêu cầu cho các thao tác ưu tiên thấp như đồng bộ hóa nền, nhưng vẫn phục vụ các phiên người dùng tương tác").
-   *Tại các task riêng lẻ*, để ngăn các dao động ngẫu nhiên trong cân bằng tải làm choáng ngợp server.

Thực hiện lập kế hoạch sức chứa

Lập kế hoạch sức chứa tốt giúp giảm xác suất xảy ra sự cố lan truyền. Cần kết hợp lập kế hoạch sức chứa với kiểm thử hiệu năng để xác định mức tải khiến dịch vụ gặp sự cố. Ví dụ, nếu điểm gãy của mọi cluster là 5.000 QPS, tải được phân bố đều giữa các cluster,<sup>[3](#fn3)</sup> và tải đỉnh của dịch vụ là 19.000 QPS, thì cần khoảng sáu cluster để chạy dịch vụ ở mức *N* + 2.

Lập kế hoạch sức chứa giúp giảm xác suất xảy ra sự cố lan truyền, nhưng không đủ để bảo vệ dịch vụ khỏi chúng. Khi mất một phần lớn hạ tầng do sự kiện đã hoặc chưa được lên kế hoạch, không mức lập kế hoạch sức chứa nào có thể ngăn được sự cố lan truyền. Các vấn đề cân bằng tải, partition mạng, hoặc traffic tăng không mong đợi có thể tạo ra các túi tải cao vượt quá mức đã lên kế hoạch. Một số hệ thống có thể tự động tăng số lượng task cho dịch vụ của bạn theo yêu cầu, điều này có thể ngăn quá tải; tuy nhiên, lập kế hoạch sức chứa thích hợp vẫn là cần thiết.

<a id="quan-ly-hang-doi"></a>

## Quản lý Hàng đợi (Queue Management)

Hầu hết các server thread-per-request (một thread cho mỗi yêu cầu) dùng một hàng đợi đặt trước thread pool để xử lý yêu cầu. Khi có yêu cầu đến, chúng sẽ nằm chờ trong hàng đợi, sau đó các thread lấy ra để thực hiện công việc thực tế (bất kỳ hành động nào server yêu cầu). Thông thường, nếu hàng đợi đầy, server sẽ từ chối các yêu cầu mới.

Nếu tỷ lệ yêu cầu và độ trễ của một task nhất định không đổi, thì không cần xếp hàng các yêu cầu: chỉ cần chiếm dụng một số lượng thread cố định. Trong kịch bản lý tưởng này, yêu cầu chỉ bị xếp hàng khi tỷ lệ yêu cầu đến ở trạng thái ổn định vượt quá khả năng xử lý của server, dẫn đến cả thread pool lẫn hàng đợi đều bị bão hòa.

Các yêu cầu xếp hàng tiêu tốn bộ nhớ và làm tăng độ trễ. Ví dụ, nếu kích thước hàng đợi là 10 lần số thread và thời gian để xử lý một yêu cầu trên một thread là 100 mili-giây, thì khi hàng đợi đầy, một yêu cầu sẽ mất 1,1 giây để xử lý, phần lớn thời gian đó dành cho việc ngồi trên hàng đợi.

Với hệ thống có traffic khá ổn định theo thời gian, thường nên giữ chiều dài hàng đợi nhỏ hơn kích thước thread pool (ví dụ, 50% hoặc ít hơn) để server có thể từ chối yêu cầu sớm khi không thể duy trì tỷ lệ các yêu cầu đến. Chẳng hạn, Gmail thường dùng các server không có hàng đợi, thay vào đó dựa vào việc failover sang các server task khác khi các thread đầy. Ở đầu kia của phổ, các hệ thống với tải “bursty” (bùng phát) — nơi các mẫu traffic dao động mạnh — có thể hoạt động tốt hơn với một kích thước hàng đợi dựa trên số thread hiện tại đang được dùng, thời gian xử lý cho mỗi yêu cầu, và kích thước cùng tần suất của các đợt bùng phát.

<a id="ganh-nhe-tai-va-suy-giam-nhe-nhan"></a>

## Gánh nhẹ Tải và Suy giảm Nhẹ nhàng (Load Shedding and Graceful Degradation)

*Gánh nhẹ tải (Load shedding)* bỏ đi một phần tải bằng cách loại bỏ traffic khi server tiếp cận các điều kiện quá tải. Mục tiêu là giữ cho server không bị cạn RAM, thất bại các kiểm tra sức khỏe, phục vụ với độ trễ cực kỳ cao, hoặc bất kỳ triệu chứng nào khác liên quan đến quá tải, trong khi vẫn làm được nhiều công việc hữu ích nhất có thể.

Một cách đơn giản để giảm tải là thực hiện throttle (giới hạn) theo task dựa trên CPU, bộ nhớ hoặc chiều dài hàng đợi; việc giới hạn chiều dài hàng đợi như đã thảo luận trong [Quản lý Hàng đợi](#quan-ly-hang-doi) là một dạng của chiến lược này. Ví dụ, một cách tiếp cận hiệu quả là trả về HTTP 503 (dịch vụ không khả dụng) cho bất kỳ yêu cầu đến nào khi có nhiều hơn một số lượng cho trước các yêu cầu client đang hoạt động.

Việc chuyển từ phương pháp xếp hàng *first-in, first-out* (FIFO, vào trước ra trước) chuẩn sang *last-in, first-out* (LIFO, vào sau ra trước), hoặc áp dụng thuật toán *controlled delay* (CoDel, độ trễ được kiểm soát) [[Nic12]](https://sre.google/sre-book/bibliography#Nic12) cùng các cách tiếp cận tương tự, có thể giảm tải bằng cách loại bỏ những yêu cầu ít có khả năng đáng để xử lý [[Mau15]](https://sre.google/sre-book/bibliography#Mau15). Giả sử một lần tìm kiếm web của người dùng bị chậm do một RPC đã nằm trong hàng đợi suốt 10 giây, thì rất có thể người dùng đã bỏ cuộc và tải lại trình duyệt, phát ra một yêu cầu mới. Khi đó, việc phản hồi yêu cầu đầu tiên trở nên vô nghĩa vì nó sẽ bị bỏ qua! Chiến lược này phát huy hiệu quả khi kết hợp với việc lan truyền deadline RPC xuyên suốt stack, như đã mô tả trong [Độ trễ và Deadline](#do-tre-va-deadline).

Các cách tiếp cận tinh vi hơn bao gồm việc xác định các client để có chọn lọc hơn về công việc nào bị bỏ, hoặc chọn các yêu cầu quan trọng hơn và ưu tiên chúng. Các chiến lược như vậy nhiều khả năng cần thiết cho các dịch vụ dùng chung.

*Suy giảm nhẹ nhàng (Graceful degradation)* đưa khái niệm gánh nhẹ tải tiến thêm một bước bằng cách giảm lượng công việc cần thực hiện. Trong một số ứng dụng, có thể giảm đáng kể lượng công việc hoặc thời gian cần thiết bằng cách hạ chất lượng của các phản hồi. Ví dụ, một ứng dụng tìm kiếm có thể chỉ tìm kiếm một tập con của dữ liệu được lưu trong một cache bộ nhớ (in-memory) thay vì cơ sở dữ liệu trên đĩa đầy đủ, hoặc dùng một thuật toán xếp hạng kém chính xác hơn (nhưng nhanh hơn) khi quá tải.

Khi đánh giá các tùy chọn gánh nhẹ tải hoặc suy giảm nhẹ nhàng cho dịch vụ của bạn, hãy cân nhắc những điều sau:

-   Bạn nên dùng các metrics nào để xác định thời điểm kích hoạt giảm tải hoặc suy giảm nhẹ nhàng (ví dụ: mức sử dụng CPU, độ trễ, chiều dài hàng đợi, số thread đang dùng, dịch vụ của bạn có tự động chuyển sang chế độ suy giảm hay cần can thiệp thủ công)?
-   Những hành động nào nên được thực hiện khi server ở chế độ suy giảm?
-   Gánh nhẹ tải và suy giảm nhẹ nhàng nên được thực hiện ở tầng nào? Việc áp dụng các chiến lược này ở mọi tầng trong stack có ý nghĩa không, hay chỉ cần một điểm nghẽn (choke-point) cấp cao là đủ?

Khi đánh giá các tùy chọn và triển khai, hãy lưu ý những điều sau:

-   Không nên kích hoạt suy giảm nhẹ nhàng quá thường xuyên — thường chỉ dùng khi kế hoạch sức chứa thất bại hoặc tải thay đổi đột ngột. Hãy giữ hệ thống đơn giản và dễ hiểu, nhất là khi ít khi phải dùng đến.
-   Hãy nhớ rằng đường code bạn không bao giờ sử dụng chính là đường code (thường) không hoạt động. Trong hoạt động ở trạng thái ổn định, chế độ suy giảm nhẹ nhàng sẽ không được dùng, nghĩa là bạn sẽ có ít kinh nghiệm vận hành hơn với chế độ này và bất kỳ quirk (đặc điểm kỳ quặc) nào của nó, điều này *làm tăng* mức độ rủi ro. Bạn có thể đảm bảo rằng suy giảm nhẹ nhàng vẫn hoạt động bằng cách thường xuyên chạy một tập con nhỏ các server gần quá tải để tập luyện đường code này.
-   Giám sát và cảnh báo khi quá nhiều server đi vào các chế độ này.
-   Gánh nhẹ tải và suy giảm nhẹ nhàng phức tạp có thể tự gây ra các vấn đề của riêng chúng — độ phức tạp quá mức có thể khiến server vấp phải chế độ suy giảm khi không mong muốn, hoặc đi vào các chu kỳ phản hồi vào những thời điểm không mong muốn. Hãy thiết kế một cách để nhanh chóng tắt suy giảm nhẹ nhàng phức tạp hoặc tinh chỉnh các tham số nếu cần. Việc lưu cấu hình này trong một hệ thống nhất quán mà mỗi server có thể theo dõi các thay đổi, chẳng hạn như Chubby, có thể tăng tốc độ triển khai, nhưng cũng tạo ra những rủi ro thất bại đồng bộ của riêng nó.

## Thử lại (Retries)

Giả sử code trong frontend nói chuyện với backend thực hiện retries (thử lại) một cách ngây thơ. Nó thử lại sau mỗi lần thất bại và giới hạn số lượng RPC backend cho mỗi yêu cầu logic ở mức 10. Hãy xem xét code sau trong frontend, sử dụng gRPC trong Go:

```go
func exampleRpcCall(client pb.ExampleClient, request pb.Request) *pb.Response {
    // Set RPC timeout to 5 seconds.
    opts := grpc.WithTimeout(5 * time.Second)

    // Try up to 10 times to make the RPC call.
    attempts := 10
    for attempts > 0 {
        conn, err := grpc.Dial(*serverAddr, opts...)
        if err != nil {
            // Something went wrong in setting up the connection. Try again.
            attempts--
            continue
        }
        defer conn.Close()

        // Create a client stub and make the RPC call.
        client := pb.NewBackendClient(conn)
        response, err := client.MakeRequest(context.Background, request)
        if err != nil {
            // Something went wrong in making the call. Try again.
            attempts--
            continue
        }

        return response
    }

    grpclog.Fatalf("ran out of attempts")
}
```

Hệ thống này có thể lan truyền theo cách sau:

1.  Giả sử backend của chúng ta có một giới hạn đã biết là 10.000 QPS mỗi task, vượt quá điểm đó mọi yêu cầu tiếp theo đều bị từ chối như một nỗ lực suy giảm nhẹ nhàng.
2.  Frontend gọi `MakeRequest` ở một tỷ lệ không đổi 10.100 QPS và làm backend quá tải 100 QPS, số lượng mà backend từ chối.
3.  100 QPS thất bại đó được thử lại trong `MakeRequest` mỗi 1.000 mili-giây, và có lẽ thành công. Nhưng chính các lần thử lại này lại đang cộng thêm vào lượng yêu cầu gửi đến backend, khiến hệ thống giờ nhận được 10.200 QPS — trong đó 200 QPS đang thất bại do quá tải.
4. Khối lượng thử lại tăng dần: 100 QPS thử lại trong giây đầu tiên, sau đó là 200 QPS, rồi 300 QPS, cứ thế tiếp diễn. Số yêu cầu thành công ngay lần thử đầu tiên ngày càng ít, khiến tỷ lệ công việc hữu ích so với tổng số yêu cầu đến backend giảm xuống.
5.  Nếu task backend không chịu nổi sự tăng tải — điều đang ngốn file descriptor, bộ nhớ và thời gian CPU trên backend — nó có thể tan rã (melt down) và sập dưới khối lượng yêu cầu cùng các lệnh thử lại thuần túy. Sau khi sập, các yêu cầu mà nó đang nhận sẽ được phân phối lại sang các task backend còn lại, khiến các task này lần lượt quá tải hơn nữa.

Một số giả định đơn giản hóa đã được đưa ra ở đây để minh họa kịch bản này,<sup>[4](#fn4)</sup> nhưng điểm mấu chốt vẫn là retries có thể gây mất ổn định một hệ thống. Hãy lưu ý rằng cả các đỉnh tải tạm thời lẫn các sự tăng chậm trong mức sử dụng đều có thể gây ra hiệu ứng này.

Ngay cả khi tỷ lệ các cuộc gọi đến `MakeRequest` giảm về mức trước khi sụp (ví dụ, 9.000 QPS), vấn đề vẫn có thể không biến mất, tùy thuộc vào mức độ mà việc trả về một thất bại tiêu tốn tài nguyên của backend. Hai yếu tố đang phát huy tác dụng ở đây:

- Nếu backend tiêu tốn một lượng tài nguyên đáng kể để xử lý các yêu cầu vốn sẽ thất bại do quá tải, thì chính các lần thử lại có thể đang khiến backend duy trì trạng thái quá tải.
-   Bản thân các server backend có thể không ổn định. Retries có thể khuếch đại các hiệu ứng được thấy trong [Server Quá Tải](#server-qua-tai).

Nếu một trong hai điều kiện này xảy ra, để thoát khỏi sự cố, bạn phải giảm đáng kể hoặc loại bỏ tải trên các frontend cho đến khi các retries dừng lại và các backend ổn định.

Mẫu hình này đã góp phần gây ra một vài sự cố lan truyền, dù các frontend và backend liên lạc qua các tin nhắn RPC, “frontend” là code JavaScript client phát ra các cuộc gọi `XmlHttpRequest` đến một endpoint và thử lại khi thất bại, hay các retries bắt nguồn từ một giao thức đồng bộ offline vốn thử lại một cách quyết liệt khi gặp thất bại.

Khi phát ra các retries tự động, hãy lưu ý các cân nhắc sau:

-   Hầu hết các chiến lược bảo vệ backend được mô tả trong [Phòng Tránh Server Quá Tải](#phong-tranh-server-qua-tai) đều áp dụng được. Cụ thể, kiểm thử hệ thống có thể làm nổi bật các vấn đề, và suy giảm nhẹ nhàng có thể giảm hiệu ứng của các retries lên backend.
-   Luôn dùng exponential backoff (lùi theo hàm mũ) có ngẫu nhiên hóa khi lên lịch các retries. Xem thêm ["Exponential Backoff and Jitter"](https://www.awsarchitectureblog.com/2015/03/backoff.html) trong AWS Architecture Blog [[Bro15]](https://sre.google/sre-book/bibliography#Bro15). Nếu các retries không được phân bố ngẫu nhiên trên cửa sổ thử lại, một nhiễu nhỏ (ví dụ, một blip mạng) có thể khiến các gợn sóng retry được lên lịch tại cùng thời điểm, và điều đó có thể tự khuếch đại [[Flo94]](https://sre.google/sre-book/bibliography#Flo94).
-   Giới hạn số retries cho mỗi yêu cầu. Đừng thử lại một yêu cầu nhất định vô hạn.
-   Cân nhắc có một ngân sách retry (retry budget) cho toàn bộ server. Ví dụ, chỉ cho phép 60 retries mỗi phút trong một process, và nếu ngân sách retry bị vượt quá, đừng thử lại; chỉ cần để yêu cầu thất bại. Chiến lược này có thể giữ hiệu ứng retry trong tầm kiểm soát và là sự khác biệt giữa một thất bại lập kế hoạch sức chứa dẫn đến một số truy vấn bị bỏ và một sự cố lan truyền toàn cục.
-   Hãy nhìn nhận dịch vụ một cách tổng thể và cân nhắc xem có thực sự cần thực hiện retries ở một cấp cụ thể hay không. Đặc biệt, cần tránh việc khuếch đại retries bằng cách phát ra chúng ở nhiều cấp: một yêu cầu đơn lẻ ở tầng cao nhất có thể tạo ra số lần thử rất lớn, bằng *tích* của số lần thử ở mỗi tầng từ trên xuống dưới. Ví dụ, nếu cơ sở dữ liệu không thể xử lý yêu cầu do quá tải, và cả ba tầng backend, frontend, JavaScript đều phát ra 3 retries (tương đương 4 lần thử), thì chỉ một thao tác của người dùng cũng có thể gây ra 64 lần thử (4^3) lên cơ sở dữ liệu. Đây là hành vi không mong muốn, nhất là khi cơ sở dữ liệu đang trả về lỗi do quá tải.
-   Sử dụng các mã phản hồi rõ ràng và cân nhắc cách xử lý các failure mode khác nhau. Ví dụ, cần tách biệt các điều kiện lỗi có thể thử lại và không thể thử lại. Đừng thử lại các lỗi vĩnh viễn hoặc các yêu cầu sai định dạng trong một client, vì cả hai sẽ không bao giờ thành công. Khi quá tải, hãy trả về một trạng thái cụ thể để các client và các tầng khác lùi lại, không thử lại.

Trong tình huống khẩn cấp, không phải lúc nào cũng dễ nhận ra rằng outage do hành vi retry kém gây ra. Biểu đồ tỷ lệ retry có thể là dấu hiệu của vấn đề này, nhưng dễ bị nhầm lẫn là triệu chứng thay vì nguyên nhân cộng hưởng. Về mặt giảm nhẹ, đây là trường hợp đặc biệt của vấn đề sức chứa không đủ, với lưu ý rằng bạn phải hoặc sửa hành vi retry (thường đòi hỏi một code push), giảm tải đáng kể, hoặc cắt đứt các yêu cầu hoàn toàn.

<a id="do-tre-va-deadline"></a>

## Độ trễ và Deadline (Latency and Deadlines)

Khi một frontend gửi một RPC đến một server backend, frontend tiêu tốn tài nguyên để chờ một phản hồi. Các deadline RPC xác định một yêu cầu có thể chờ bao lâu trước khi frontend bỏ cuộc, qua đó giới hạn thời gian mà backend có thể tiêu tốn tài nguyên của frontend.

### Chọn một deadline

Thường nên đặt một deadline. Nếu không đặt deadline, hoặc đặt deadline quá cao, các vấn đề ngắn hạn đã tồn tại từ lâu vẫn có thể tiếp tục chiếm dụng tài nguyên server cho đến khi máy chủ được khởi động lại.

Các deadline cao có thể khiến tài nguyên bị tiêu tốn ở các tầng cao hơn của stack khi các tầng thấp hơn đang gặp sự cố. Ngược lại, deadline quá ngắn có thể khiến một số yêu cầu đắt tiền liên tục thất bại. Việc cân bằng các ràng buộc này để chọn ra deadline phù hợp đôi khi là cả một nghệ thuật.

### Bỏ lỡ deadline

Một chủ đề phổ biến trong nhiều [outage lan truyền](https://sre.google/sre-book/tracking-outages/) là các server tiêu tốn tài nguyên để xử lý các yêu cầu sẽ vượt quá deadline của chúng ở phía client. Kết quả là, tài nguyên bị tiêu tốn trong khi không có tiến triển nào được thực hiện: bạn không được ghi công cho các bài tập muộn với RPC.

Giả sử một RPC có deadline 10 giây do client đặt. Server bị quá tải nặng, khiến thời gian chờ từ hàng đợi đến thread pool lên tới 11 giây. Lúc này, client đã từ bỏ yêu cầu. Trong hầu hết các trường hợp, nếu server vẫn cố xử lý, đó là hành động dại dột, vì nó sẽ làm một việc không được ghi nhận — client không quan tâm server làm gì sau khi deadline đã trôi qua, bởi nó đã từ bỏ yêu cầu.

Khi xử lý một yêu cầu trải qua nhiều giai đoạn (ví dụ, có một vài callback và các cuộc gọi RPC), server nên kiểm tra deadline còn lại ở mỗi giai đoạn trước khi thực hiện thêm công việc nào cho yêu cầu đó. Chẳng hạn, nếu một yêu cầu được chia thành các giai đoạn phân tích, yêu cầu backend và xử lý, việc kiểm tra xem còn đủ thời gian để xử lý trước mỗi giai đoạn có thể là hợp lý.

### Lan truyền deadline (Deadline propagation)

Thay vì tự đặt một deadline khi gửi RPC đến các backend, các server nên áp dụng lan truyền deadline.

Với cơ chế lan truyền deadline, một deadline được đặt ở tầng cao trong stack (ví dụ, trong frontend). Toàn bộ cây RPC phát sinh từ một yêu cầu ban đầu sẽ chia sẻ cùng một deadline tuyệt đối. Chẳng hạn, nếu server *A* chọn deadline 30 giây và xử lý yêu cầu trong 7 giây trước khi gửi RPC đến server *B*, thì RPC từ *A* đến *B* sẽ có deadline 23 giây. Tương tự, nếu server *B* mất 4 giây để xử lý và gửi RPC đến server *C*, RPC từ *B* đến *C* sẽ có deadline 19 giây, và cứ tiếp tục như vậy. Về lý tưởng, mỗi server trong cây yêu cầu đều phải thực hiện lan truyền deadline.

Không có lan truyền deadline, kịch bản sau có thể xảy ra:

1.  Server *A* gửi một RPC đến server *B* với một deadline 10 giây.
2.  Server *B* mất 8 giây để bắt đầu xử lý yêu cầu, rồi gửi một RPC đến server *C*.
3.  Nếu server *B* dùng lan truyền deadline, nó nên đặt deadline 2 giây, nhưng giả sử thay vào đó nó dùng deadline 20 giây được ghi cứng cho RPC đến server *C*.
4.  Server *C* lấy yêu cầu ra khỏi hàng đợi của nó sau 5 giây.

Nếu server *B* đã lan truyền deadline, server *C* có thể lập tức từ chối yêu cầu vì deadline 2 giây đã bị vượt quá. Tuy nhiên, trong kịch bản này, server *C* vẫn xử lý yêu cầu với suy nghĩ rằng mình còn 15 giây dự phòng, nhưng thực ra không làm được việc hữu ích nào, vì yêu cầu từ server *A* đến server *B* đã quá hạn.

Bạn có thể muốn giảm nhẹ deadline gửi đi (ví dụ, vài trăm mili-giây) để tính đến thời gian truyền qua mạng và hậu xử lý trong client.

Cũng nên cân nhắc đặt giới hạn trên cho thời hạn gửi đi. Bạn có thể muốn giới hạn thời gian server chờ các RPC gửi đến các backend không quan trọng, hoặc các backend thường hoàn thành trong một khoảng thời gian ngắn. Tuy nhiên, hãy đảm bảo hiểu rõ mix traffic của mình, vì nếu không, bạn có thể vô tình khiến một số kiểu yêu cầu nhất định thất bại vĩnh viễn (ví dụ, các yêu cầu có payload lớn, hoặc các yêu cầu đòi hỏi phản hồi sau một lượng lớn tính toán).

Có một số ngoại lệ cho phép server tiếp tục xử lý yêu cầu sau khi deadline đã trôi qua. Chẳng hạn, nếu server nhận được yêu cầu thực hiện các thao tác bắt kịp (catchup) tốn kém và định kỳ kiểm tra điểm kiểm soát (checkpoint) tiến độ, thì nên chỉ kiểm tra deadline sau khi đã ghi checkpoint, thay vì sau thao tác tốn kém.

### Lan truyền hủy bỏ (Cancellation propagation)

Việc lan truyền các lệnh hủy bỏ giúp loại bỏ những công việc không cần thiết hoặc chắc chắn sẽ thất bại, bằng cách thông báo cho các server trong một stack gọi RPC rằng nỗ lực của chúng không còn cần thiết. Để giảm độ trễ, một số hệ thống sử dụng “hedged requests” [[Dea13]](https://sre.google/sre-book/bibliography#Dea13): client gửi RPC đến một server chính, sau đó, nếu server này phản hồi chậm, client sẽ gửi cùng yêu cầu đến các instance khác của dịch vụ. Khi nhận được phản hồi từ bất kỳ server nào, client gửi tin nhắn đến các server còn lại để hủy bỏ những yêu cầu giờ đã thừa. Vì các yêu cầu này có thể tự lan truyền sang nhiều server khác một cách chuyển tiếp, nên các lệnh hủy bỏ cũng cần được lan truyền xuyên suốt toàn bộ stack.

Cách tiếp cận này cũng giúp tránh nguy cơ rò rỉ tài nguyên trong trường hợp một RPC ban đầu có deadline dài, nhưng các RPC quan trọng ở các tầng sâu hơn của stack gặp lỗi không thể khắc phục khi thử lại, hoặc có deadline ngắn và bị time out (hết thời gian). Nếu chỉ dùng lan truyền deadline đơn giản, cuộc gọi ban đầu sẽ tiếp tục chiếm dụng tài nguyên server cho đến khi hết deadline, dù chắc chắn sẽ thất bại. Việc đẩy các lỗi nghiêm trọng hoặc time out lên stack và hủy bỏ các RPC khác trong cây gọi sẽ ngăn được công việc không cần thiết khi yêu cầu nói chung không thể được đáp ứng.

### Độ trễ hai chế độ (Bimodal latency)

Giả sử frontend trong ví dụ trước gồm 10 server, mỗi server chạy 100 worker thread, tương đương tổng sức chứa 1.000 thread. Khi hoạt động bình thường, các frontend xử lý 1.000 QPS với thời gian hoàn thành mỗi yêu cầu là 100 mili-giây. Do đó, số worker thread đang bị chiếm dụng thường là 100 trên tổng số 1.000 worker thread được cấu hình (1.000 QPS \* 0,1 giây).

Giả sử một sự kiện khiến 5% các yêu cầu không bao giờ hoàn thành. Nguyên nhân có thể là một số row range trong Bigtable không khả dụng, khiến các yêu cầu tương ứng với keyspace Bigtable đó không thể được phục vụ. Kết quả là, 5% các yêu cầu chạm deadline, trong khi 95% yêu cầu còn lại mất 100 mili-giây như bình thường.

Với deadline 100 giây, 5% các yêu cầu sẽ tiêu tốn 5.000 thread (50 QPS \* 100 giây), trong khi frontend không có nhiều thread khả dụng đến vậy. Giả sử không có các hiệu ứng thứ cấp nào khác, frontend chỉ xử lý được 19,6% các yêu cầu (1.000 thread khả dụng / (5.000 + 95) khối lượng công việc tính bằng thread), khiến tỷ lệ lỗi lên tới 80,4%.

Do đó, thay vì chỉ 5% các yêu cầu nhận được một lỗi (những yêu cầu không hoàn thành do keyspace không khả dụng), hầu hết các yêu cầu đều nhận được một lỗi.

Các hướng dẫn sau có thể giúp giải quyết lớp vấn đề này:

-   Việc phát hiện vấn đề này có thể rất khó. Cụ thể, khi chỉ nhìn vào độ trễ *trung bình*, bạn có thể không nhận ra rằng độ trễ hai chế độ chính là nguyên nhân gây ra outage. Khi thấy độ trễ tăng, hãy xem xét *phân bố* của các độ trễ bên cạnh các giá trị trung bình.
-   Vấn đề này có thể tránh được nếu các yêu cầu không hoàn thành trả về lỗi sớm, thay vì chờ hết deadline. Ví dụ, nếu một backend không khả dụng, cách tốt nhất thường là lập tức trả về lỗi cho backend đó, thay vì tiêu tốn tài nguyên cho đến khi backend khả dụng. Nếu tầng RPC của bạn hỗ trợ tùy chọn fail-fast (thất bại nhanh), hãy sử dụng nó.
-   Đặt deadline dài hơn nhiều bậc độ lớn so với độ trễ trung bình của yêu cầu thường là xấu. Trong ví dụ trước, một số lượng nhỏ các yêu cầu ban đầu chạm deadline, nhưng deadline đã lớn hơn ba bậc độ lớn so với độ trễ trung bình bình thường, dẫn đến việc cạn kiệt thread.
- Khi sử dụng các tài nguyên dùng chung có thể bị một keyspace làm cạn kiệt, hãy cân nhắc giới hạn số lượng yêu cầu đang hoạt động của keyspace đó hoặc áp dụng các cơ chế theo dõi lạm dụng khác. Giả sử backend của bạn xử lý các yêu cầu từ những client khác nhau, trong đó hiệu năng và đặc điểm yêu cầu của chúng rất khác biệt. Bạn có thể cân nhắc chỉ cho phép tối đa 25% thread bị chiếm dụng bởi bất kỳ client nào, nhằm đảm bảo sự công bằng trước tải nặng do bất kỳ client nào đang hành xử sai gây ra.

## Khởi động Chậm và Cache Lạnh (Slow Startup and Cold Caching)

Các process thường phản hồi chậm hơn ngay sau khi khởi động so với khi đã đạt trạng thái ổn định. Hiện tượng này có thể do một hoặc cả hai nguyên nhân sau:

Khởi tạo bắt buộc

Thiết lập các kết nối khi nhận được yêu cầu đầu tiên cần một backend nhất định

Cải thiện hiệu năng thời gian chạy trong một số ngôn ngữ, đặc biệt là Java

Biên dịch Just-In-Time (JIT), tối ưu hotspot, và nạp lớp (class loading) bị hoãn

Tương tự, một số binary hoạt động kém hiệu quả hơn khi các cache chưa được lấp đầy. Ví dụ, với một số dịch vụ của Google, phần lớn yêu cầu được phục vụ từ cache, nên các yêu cầu bỏ lỡ cache (miss cache) tốn kém hơn đáng kể. Khi hệ thống chạy ở trạng thái ổn định với cache ấm (warm cache), chỉ có một vài lần bỏ lỡ cache, nhưng nếu cache hoàn toàn trống, 100% các yêu cầu đều tốn kém. Các dịch vụ khác có thể dùng cache để giữ trạng thái người dùng trong RAM. Điều này có thể đạt được nhờ sự stickiness (dính bám) cứng hoặc mềm giữa các reverse proxy và frontend dịch vụ.

Nếu dịch vụ không được cấp đủ tài nguyên để xử lý các yêu cầu khi cache lạnh, nó sẽ có nguy cơ cao hơn xảy ra outage và cần thực hiện các biện pháp phòng ngừa.

Các kịch bản sau có thể dẫn đến một cache lạnh:

Bật một cluster mới

Một cluster vừa được thêm vào sẽ có một cache trống.

Trả lại một cluster cho dịch vụ sau khi bảo trì

Cache có thể đã bị lỗi thời (stale).

Khởi động lại

Nếu một task có cache vừa được khởi động lại, việc lấp đầy cache của nó sẽ mất một chút thời gian. Có thể đáng giá khi di chuyển việc cache từ một server sang một binary riêng biệt như memcache, điều này cũng cho phép chia sẻ cache giữa nhiều server, dù có chi phí thêm một RPC và một chút độ trễ bổ sung.

Nếu việc cache có một hiệu ứng đáng kể lên dịch vụ,<sup>[5](#fn5)</sup> bạn có thể muốn sử dụng một hoặc một số chiến lược sau:

-   Cấp quá mức (overprovision) dịch vụ. Cần phân biệt rõ giữa latency cache (cache độ trễ) và capacity cache (cache sức chứa): khi áp dụng latency cache, dịch vụ vẫn duy trì được tải kỳ vọng ngay cả khi cache trống; ngược lại, dịch vụ dùng capacity cache không thể duy trì tải kỳ vọng trong tình huống cache trống. Chủ sở hữu dịch vụ cần thận trọng khi thêm cache, đảm bảo cache mới là latency cache hoặc được thiết kế đủ tốt để hoạt động an toàn như capacity cache. Đôi khi, cache được thêm vào để cải thiện hiệu năng nhưng thực chất lại trở thành phụ thuộc cứng.
-   Áp dụng các kỹ thuật phòng chống sự cố lan truyền nói chung. Cụ thể, các server nên từ chối yêu cầu khi quá tải hoặc chuyển sang chế độ suy giảm, đồng thời cần kiểm thử để xem dịch vụ hoạt động ra sao sau các sự kiện như một lần khởi động lại lớn.
- Khi tăng tải cho một cluster, hãy tăng dần. Tỷ lệ yêu cầu ban đầu thấp sẽ giúp làm ấm cache; khi cache đã ấm, có thể đưa thêm traffic vào. Đây là cách tốt để đảm bảo mọi cluster đều mang tải định mức và cache luôn được giữ ấm.

## Luôn Đi Xuống Trong Stack (Always Go Downward in the Stack)

Trong dịch vụ Shakespeare ví dụ, frontend giao tiếp với một backend, và backend lại giao tiếp với tầng lưu trữ. Một vấn đề xuất hiện ở tầng lưu trữ có thể gây ảnh hưởng đến các server đang giao tiếp với nó, nhưng việc khắc phục tầng lưu trữ thường sẽ đồng thời sửa chữa cả tầng backend lẫn frontend.

Tuy nhiên, giả sử các backend liên lạc chéo với nhau. Ví dụ, khi tầng lưu trữ không thể xử lý một yêu cầu, các backend có thể proxy yêu cầu đó cho nhau để thay đổi chủ sở hữu của một người dùng. Sự liên lạc nội tầng (intra-layer communication) này có thể gây ra vấn đề vì một số lý do:

-   Sự liên lạc dễ bị tổn thương trước một deadlock (khóa chết) phân tán. Các backend có thể dùng cùng một thread pool để chờ các RPC gửi đến các backend từ xa, trong khi các backend từ xa đó đồng thời đang nhận các yêu cầu từ các backend từ xa. Giả sử thread pool của backend *A* đầy. Backend *B* gửi một yêu cầu đến backend *A* và chiếm dụng một thread trong backend *B* cho đến khi thread pool của backend *A* trống. Hành vi này có thể gây ra sự bão hòa thread pool lan truyền.
    
-   Nếu sự liên lạc nội tầng tăng lên để xử lý một số kiểu thất bại hoặc điều kiện tải nặng (ví dụ, cân bằng tải lại tích cực hơn dưới tải cao), mức liên lạc nội tầng có thể nhanh chóng chuyển từ thấp sang cao khi tải tăng đủ.
    
    Ví dụ, giả sử một người dùng có một backend chính và một backend phụ hot standby (dự phòng nóng) được xác định trước trong một cluster khác, có thể tiếp quản người dùng. Backend chính proxy các yêu cầu đến backend phụ do các lỗi từ tầng thấp hơn hoặc để ứng phó với tải nặng trên master. Nếu toàn bộ hệ thống quá tải, việc proxy từ chính sang phụ có khả năng tăng lên và thêm nhiều tải hơn nữa cho hệ thống, do backend chính phải tốn thêm chi phí phân tích và chờ yêu cầu chuyển đến phụ.
    
-   Tùy thuộc vào mức độ quan trọng của sự liên lạc giữa các tầng, việc khởi tạo (bootstrap) hệ thống có thể trở nên phức tạp hơn.
    
Thường nên tránh các vòng tuần hoàn có thể xảy ra trong đường dẫn liên lạc nội tầng (tức là giữa các thành phần cùng tầng) của đường dẫn yêu cầu người dùng. Thay vào đó, hãy để client thực hiện việc liên lạc. Ví dụ, nếu một frontend giao tiếp với một backend nhưng chọn sai, backend không nên proxy đến backend đúng. Thay vào đó, backend nên thông báo cho frontend biết để thử lại yêu cầu trên backend đúng.

## Các Điều kiện Kích hoạt cho Sự cố Lan truyền (Triggering Conditions for Cascading Failures)

Khi một dịch vụ dễ bị tổn thương trước các sự cố lan truyền, chỉ cần một số nhiễu nhỏ cũng có thể khởi đầu hiệu ứng domino. Phần này xác định một số yếu tố kích hoạt các sự cố lan truyền.

## Cái chết của Process (Process Death)

Một số server task có thể chết, làm giảm sức chứa khả dụng. Nguyên nhân có thể là Query of Death (một RPC mà nội dung của nó kích hoạt sự cố trong process), các vấn đề cluster, các thất bại assertion (khẳng định), hoặc một số lý do khác. Một sự kiện rất nhỏ (ví dụ, một vài lần sập, hoặc các task được lên lịch lại sang các máy khác) có thể khiến một dịch vụ đang bên bờ vực sụp đổ gục hẳn.

## Cập nhật Process (Process Updates)

Việc đẩy một phiên bản binary mới hoặc cập nhật cấu hình có thể gây ra sự cố lan truyền nếu ảnh hưởng đồng thời đến một lượng lớn task. Để ngăn kịch bản này, hãy tính đến lượng overhead sức chứa cần thiết khi thiết lập hạ tầng cập nhật của dịch vụ, hoặc đẩy ngoài giờ cao điểm (off-peak). Một cách tiếp cận khả thi là điều chỉnh động số lượng task cập nhật đang hoạt động dựa trên khối lượng yêu cầu và sức chứa khả dụng.

## Các Triển khai Mới (New Rollouts)

Một binary mới, các thay đổi cấu hình, hoặc một thay đổi đối với stack hạ tầng cơ sở có thể dẫn đến thay đổi trong hồ sơ yêu cầu, việc sử dụng và giới hạn tài nguyên, các backend, hoặc một số thành phần hệ thống khác, và có thể kích hoạt một sự cố lan truyền.

Khi xảy ra sự cố lan truyền, việc kiểm tra các thay đổi gần đây và cân nhắc hoàn tác chúng thường là lựa chọn khôn ngoan, nhất là khi những thay đổi đó đã tác động đến sức chứa hoặc làm thay đổi hồ sơ yêu cầu.

Dịch vụ của bạn nên thực hiện một số hình thức ghi log thay đổi (change logging), điều có thể giúp nhanh chóng xác định các thay đổi gần đây.

## Tăng trưởng Hữu cơ (Organic Growth)

Trong nhiều trường hợp, sự cố lan truyền không bắt nguồn từ một thay đổi dịch vụ cụ thể, mà do mức sử dụng tăng lên nhưng sức chứa không được điều chỉnh tương ứng.

## Các Thay đổi Đã lên Kế hoạch, Drains, hoặc Turndowns (Planned Changes, Drains, or Turndowns)

Nếu dịch vụ của bạn là multihomed (đa nhà cung cấp), một phần sức chứa có thể không khả dụng do bảo trì hoặc các outage trong một cluster. Tương tự, một trong những phụ thuộc quan trọng của dịch vụ có thể bị drain (rút dần), khiến dịch vụ phía trên giảm sức chứa do drain dependencies (phụ thuộc rút dần), hoặc tăng độ trễ vì phải gửi yêu cầu đến một cluster xa hơn.

### Thay đổi hồ sơ yêu cầu (Request profile changes)

Một dịch vụ backend có thể nhận yêu cầu từ nhiều cluster khác nhau, do một dịch vụ frontend đã chuyển hướng traffic của nó vì thay đổi cấu hình cân bằng tải, thay đổi trong mix traffic, hoặc do cluster bị đầy. Ngoài ra, chi phí trung bình để xử lý một payload riêng lẻ cũng có thể thay đổi do các thay đổi về code hoặc cấu hình frontend. Tương tự, dữ liệu mà dịch vụ xử lý có thể thay đổi một cách tự nhiên do mức sử dụng tăng hoặc do sự khác biệt từ những người dùng hiện có: ví dụ, với một dịch vụ lưu trữ ảnh, cả số lượng lẫn kích thước của các hình ảnh *trên mỗi người dùng* có xu hướng tăng theo thời gian.

### Giới hạn tài nguyên (Resource limits)

Một số hệ điều hành cluster cho phép overcommitment (cam kết vượt mức) tài nguyên. CPU là tài nguyên có thể chuyển đổi (fungible); thường thì một số máy có sẵn lượng slack CPU (CPU dư), đóng vai trò như một lưới an toàn nhất định trước các đỉnh CPU. Lượng slack CPU này không đồng nhất giữa các cell (ô), và cũng khác nhau giữa các máy trong cùng một cell.

Việc coi lượng slack CPU này như lưới an toàn là rất nguy hiểm. Sự khả dụng của nó phụ thuộc hoàn toàn vào hành vi của các job khác trong cluster, nên có thể biến mất bất cứ lúc nào. Chẳng hạn, nếu một đội khởi chạy MapReduce tiêu thụ nhiều CPU và được lên lịch trên nhiều máy, tổng lượng slack CPU có thể giảm đột ngột, gây ra tình trạng thiếu CPU cho các job không liên quan. Khi chạy kiểm thử tải, hãy đảm bảo bạn vẫn nằm trong giới hạn tài nguyên đã cam kết.

<a id="kiem-thu-cho-cac-su-that-bai-lan-truyen"></a>

## Kiểm thử cho các Sự cố Lan truyền (Testing for Cascading Failures)

Rất khó dự đoán từ các nguyên lý cơ bản về những cách cụ thể mà một dịch vụ có thể gặp sự cố. Phần này thảo luận các chiến lược kiểm thử nhằm phát hiện xem các dịch vụ có dễ bị tổn thương trước các sự cố lan truyền hay không.

Bạn nên kiểm thử dịch vụ của mình dưới tải nặng để xác định cách nó hoạt động, từ đó có cơ sở tin rằng nó sẽ không gây ra sự cố lan truyền trong các hoàn cảnh khác nhau.

## Kiểm thử Cho đến Thất bại và Hơn Thế Nữa (Test Until Failure and Beyond)

Hiểu rõ hành vi của dịch vụ dưới tải nặng có lẽ là bước đầu tiên quan trọng nhất để tránh các sự cố lan truyền. Biết hệ thống của bạn hoạt động ra sao khi quá tải giúp xác định những nhiệm vụ kỹ thuật nào cần ưu tiên sửa chữa lâu dài; ít nhất, kiến thức này cũng giúp các kỹ sư on-call (trực sự cố) khởi động quá trình debug khi tình huống khẩn cấp xảy ra.

Kiểm thử tải các thành phần cho đến khi chúng gãy. Khi tải tăng, một thành phần thường xử lý các yêu cầu thành công cho đến khi đạt đến điểm không thể xử lý thêm. Ở điểm này, lý tưởng nhất là thành phần nên bắt đầu phục vụ các lỗi hoặc kết quả suy giảm để đáp ứng tải bổ sung, nhưng không làm giảm đáng kể tỷ lệ yêu cầu xử lý thành công. Một thành phần rất dễ bị tổn thương trước sự cố lan truyền sẽ bắt đầu sập hoặc phục vụ tỷ lệ lỗi rất cao khi quá tải; ngược lại, một thành phần được thiết kế tốt hơn sẽ có thể từ chối một vài yêu cầu và sống sót.

Kiểm thử tải cũng chỉ ra điểm gãy, một kiến thức nền tảng cho quy trình lập kế hoạch sức chứa. Nhờ đó, bạn có thể kiểm thử các lỗi hồi quy (regression), cấp tài nguyên cho các ngưỡng trường hợp xấu nhất, và đánh đổi giữa mức sử dụng (utilization) với các biên an toàn.

Do ảnh hưởng của cache, việc tăng tải từ từ có thể cho kết quả khác so với việc tăng ngay lập tức lên các mức tải được kỳ vọng. Vì vậy, hãy cân nhắc kiểm thử cả các mẫu tải tăng dần lẫn tải xung (impulse).

Bạn cũng nên kiểm thử và nắm rõ cách thành phần hoạt động khi quay lại mức tải định mức sau khi đã bị đẩy vượt xa ngưỡng đó. Những kiểm thử như vậy có thể giúp trả lời các câu hỏi như:

-   Nếu một thành phần rơi vào trạng thái suy giảm dưới tải nặng, nó có khả năng tự phục hồi mà không cần con người can thiệp không?
-   Nếu một vài server sập dưới tải nặng, tải cần giảm bao nhiêu để hệ thống có thể ổn định?

Khi kiểm thử tải một dịch vụ có trạng thái (stateful) hoặc dịch vụ dùng cache, bạn cần theo dõi trạng thái xuyên suốt các lần tương tác và kiểm tra tính đúng đắn dưới tải cao, bởi đó chính là nơi các bug concurrency (song song) tinh vi thường bộc lộ.

Lưu ý rằng các thành phần riêng lẻ có thể có các điểm gãy khác nhau, do đó cần kiểm thử tải từng thành phần một. Bạn không thể biết trước thành phần nào sẽ chạm tường trước, và cần nắm rõ hệ thống hoạt động ra sao khi điều đó xảy ra.

Nếu bạn tin rằng hệ thống đã có các biện pháp bảo vệ chống quá tải phù hợp, hãy cân nhắc chạy các kiểm thử thất bại trên một phần nhỏ production để tìm điểm mà các thành phần trong hệ thống của bạn thất bại dưới traffic thực. Các giới hạn này có thể không được phản ánh đầy đủ bởi traffic kiểm thử tải tổng hợp, do đó kiểm thử traffic thực có thể cho kết quả sát thực tế hơn, dù đi kèm rủi ro gây ra sự cố mà người dùng có thể nhận thấy. Hãy thận trọng khi kiểm thử trên traffic thực: đảm bảo bạn có sẵn sức chứa dự phòng trong trường hợp các biện pháp bảo vệ tự động không hoạt động và bạn phải failover thủ công. Bạn có thể cân nhắc một số kiểm thử production sau:

-   Giảm số lượng task nhanh hoặc chậm theo thời gian, vượt quá các mẫu traffic được kỳ vọng
-   Nhanh chóng mất một lượng sức chứa của một cluster
-   Blackholing (chặn hoàn toàn) các backends khác nhau

## Kiểm thử các Client Phổ biến (Test Popular Clients)

Hiểu cách các client lớn sử dụng dịch vụ của bạn. Ví dụ, bạn muốn biết xem các client:

-   Có thể xếp hàng công việc trong khi dịch vụ đang xuống không
-   Sử dụng exponential backoff ngẫu nhiên hóa khi gặp lỗi
-   Dễ bị tổn thương trước các kích hoạt từ bên ngoài có thể tạo ra một lượng lớn tải (ví dụ, một bản cập nhật phần mềm được kích hoạt từ bên ngoài có thể làm trống cache của một client offline)

Tùy vào dịch vụ của bạn, bạn có thể hoặc không kiểm soát được toàn bộ code client tương tác với dịch vụ đó. Dù vậy, việc nắm rõ cách các client lớn tương tác với dịch vụ của bạn vẫn là một ý tưởng hay.

Các nguyên lý tương tự cũng áp dụng cho các client nội bộ lớn. Hãy kịch tính hóa (chạy tình huống giả định) các sự cố hệ thống với những client lớn nhất để xem cách họ phản ứng. Đồng thời, hỏi các client nội bộ về cách họ truy cập dịch vụ của bạn và các cơ chế họ sử dụng để xử lý sự cố phía backend.

## Kiểm thử các Backend Không Quan trọng (Test Noncritical Backends)

Hãy kiểm thử các backend không quan trọng và đảm bảo rằng việc chúng không khả dụng không gây ảnh hưởng đến các thành phần cốt lõi của dịch vụ.

Ví dụ, giả sử frontend của bạn kết nối với các backend quan trọng và không quan trọng. Thông thường, một yêu cầu cụ thể sẽ bao gồm cả thành phần quan trọng (ví dụ: kết quả truy vấn) lẫn thành phần không quan trọng (ví dụ: gợi ý chính tả). Khi chờ các backend không quan trọng hoàn tất, hệ thống có thể bị chậm đáng kể và tiêu tốn nhiều tài nguyên.

Ngoài việc kiểm thử hành vi khi backend không quan trọng không khả dụng, hãy kiểm thử cách frontend hoạt động nếu backend không quan trọng không bao giờ phản hồi (ví dụ, nếu nó đang blackholing các yêu cầu). Các backend được tuyên bố là không quan trọng vẫn có thể gây ra vấn đề trên các frontends khi các yêu cầu có các deadline dài. Frontend không nên bắt đầu từ chối nhiều yêu cầu, hết tài nguyên, hoặc phục vụ với độ trễ rất cao chỉ vì một backend không quan trọng đang blackholing.

## Các Bước Lập Tức để Giải quyết Sự cố Lan truyền (Immediate Steps to Address Cascading Failures)

Khi đã xác định dịch vụ đang gặp sự cố lan truyền, bạn có thể áp dụng một số chiến lược khác nhau để xử lý — và tất nhiên, đây cũng là dịp tốt để thực thi giao thức quản lý sự cố của mình ([Quản lý Sự cố](https://sre.google/sre-book/managing-incidents/)).

## Tăng Tài nguyên (Increase Resources)

Nếu hệ thống của bạn đang chạy ở sức chứa suy giảm và bạn có các tài nguyên nhàn rỗi, thêm các task có thể là cách nhanh nhất để phục hồi từ outage. Tuy nhiên, nếu dịch vụ đã rơi vào một vòng xoáy cái chết (death spiral), việc thêm nhiều tài nguyên hơn có thể không đủ để phục hồi.

## Dừng các Kiểm tra sức khỏe Thất bại/Cái chết (Stop Health Check Failures/Deaths)

Một số hệ thống lên lịch cluster, chẳng hạn như Borg, kiểm tra sức khỏe của các task trong một job và khởi động lại các task không khỏe mạnh. Thực hành này có thể tạo ra một failure mode mà ở đó chính việc kiểm tra sức khỏe lại làm cho dịch vụ không khỏe. Ví dụ, nếu một nửa các task không thể thực hiện bất kỳ công việc nào vì chúng đang khởi động, và một nửa còn lại sẽ sớm bị giết vì chúng quá tải và thất bại các kiểm tra sức khỏe, việc tạm thời vô hiệu hóa các kiểm tra sức khỏe có thể cho phép hệ thống ổn định cho đến khi tất cả các task đang chạy.

Kiểm tra sức khỏe process (“binary này có đang phản hồi *bất kỳ điều gì* không?”) và kiểm tra sức khỏe dịch vụ (“binary này có khả năng phản hồi *lớp yêu cầu này* ngay bây giờ không?”) là hai thao tác khác nhau về mặt khái niệm. Kiểm tra sức khỏe process liên quan đến bộ lên lịch cluster, trong khi kiểm tra sức khỏe dịch vụ liên quan đến load balancer. Việc phân biệt rõ ràng giữa hai loại kiểm tra sức khỏe có thể giúp tránh kịch bản này.

## Khởi động lại Servers (Restart Servers)

Nếu các server bị kẹt (wedged) và không tiến triển, việc khởi động lại có thể giúp ích. Hãy thử khởi động lại các server khi:

-   Các server Java đang trong một vòng xoáy cái chết GC
-   Một số yêu cầu đang hoạt động không có deadline nhưng đang tiêu tốn tài nguyên, dẫn đến chúng chặn các thread
-   Các server đang bị deadlock

Trước khi khởi động lại server, hãy xác định rõ nguồn gốc của sự cố lan truyền và đảm bảo rằng việc này không chỉ đơn thuần là đẩy tải sang nơi khác. Hãy triển khai canary cho thay đổi này và thực hiện từ từ. Nếu outage thực sự do một vấn đề như cache lạnh, hành động của bạn có thể khuếch đại sự cố lan truyền hiện có.

## Bỏ Traffic (Drop Traffic)

Bỏ tải là biện pháp mạnh, thường chỉ dùng khi sự cố đang lan rộng và không thể xử lý bằng cách khác. Ví dụ, nếu tải quá nặng khiến hầu hết server sập ngay sau khi vừa ổn định, bạn có thể khôi phục dịch vụ bằng cách:

1.  Giải quyết điều kiện kích hoạt ban đầu (bằng cách thêm sức chứa, ví dụ).
2.  Giảm tải đủ để các lần sập dừng lại. Hãy cân nhắc quyết đoán ở đây — nếu toàn bộ dịch vụ đang crash-loop, chỉ cho phép, chẳng hạn, 1% traffic đi qua.
3.  Chờ cho phần lớn các server trở nên khỏe mạnh.
4.  Từ từ tăng tải lên.

Chiến lược này cho phép các cache được làm ấm, các kết nối được thiết lập, v.v., trước khi tải quay lại các mức bình thường.

Rõ ràng, chiến thuật này sẽ gây ra rất nhiều tổn hại mà người dùng có thể thấy. Việc bạn có thể (hoặc thậm chí *có nên*) bỏ traffic một cách không phân biệt hay không phụ thuộc vào cách dịch vụ được cấu hình. Nếu có cơ chế để bỏ các traffic ít quan trọng hơn (ví dụ, prefetching, nạp trước), hãy sử dụng cơ chế đó trước.

Cần lưu ý rằng chiến lược này chỉ cho phép bạn phục hồi từ một outage lan truyền khi vấn đề nền tảng đã được sửa. Nếu nguyên nhân khởi đầu sự cố lan truyền chưa được xử lý (ví dụ, sức chứa toàn cục không đủ), sự cố có thể bị kích hoạt lại ngay sau khi toàn bộ traffic quay lại. Vì vậy, trước khi áp dụng chiến lược này, hãy cân nhắc sửa (hoặc ít nhất là che đậy) nguyên nhân gốc rễ hoặc điều kiện kích hoạt. Chẳng hạn, nếu dịch vụ đã hết bộ nhớ và đang rơi vào vòng xoáy cái chết, việc bổ sung thêm bộ nhớ hoặc task nên là bước đầu tiên.

## Đi vào các Chế độ Suy giảm (Enter Degraded Modes)

Phục vụ các kết quả suy giảm bằng cách giảm tải hoặc loại bỏ traffic không quan trọng. Chiến lược này cần được tích hợp sẵn vào dịch vụ, và chỉ khả thi khi bạn xác định được traffic nào có thể bị suy giảm cũng như có khả năng phân biệt giữa các payload khác nhau.

## Loại bỏ Tải Batch (Eliminate Batch Load)

Một số dịch vụ có tải quan trọng nhưng không khẩn cấp. Hãy cân nhắc tắt các nguồn tải đó. Ví dụ, nếu việc cập nhật index, sao chép dữ liệu, hoặc thu thập thống kê đang tiêu tốn tài nguyên của đường phục vụ (serving path), hãy cân nhắc tắt các nguồn tải đó trong một outage.

## Loại bỏ Traffic Xấu (Eliminate Bad Traffic)

Nếu một số truy vấn đang tạo ra tải nặng hoặc gây sập (ví dụ, các query of death), hãy cân nhắc chặn chúng hoặc loại bỏ chúng bằng các phương tiện khác.

### Sự cố Lan truyền và Shakespeare

Một bộ phim tài liệu về các tác phẩm của Shakespeare được phát sóng ở Nhật Bản, trong đó giới thiệu dịch vụ Shakespeare của chúng ta như một nguồn tài liệu lý tưởng để nghiên cứu sâu hơn. Sau khi chương trình lên sóng, lượng truy cập (traffic) đến datacenter châu Á của chúng ta tăng vọt, vượt quá sức chứa của dịch vụ. Tình trạng quá tải này càng trở nên nghiêm trọng hơn do một bản cập nhật lớn cho dịch vụ Shakespeare cũng được triển khai tại datacenter đó cùng thời điểm.

May mắn thay, một số biện pháp an toàn đã được thiết lập để giảm thiểu nguy cơ xảy ra sự cố. Quy trình Production Readiness Review đã chỉ ra một số vấn đề mà nhóm đã xử lý. Chẳng hạn, các nhà phát triển đã tích hợp cơ chế suy giảm nhẹ nhàng vào dịch vụ. Khi tài nguyên trở nên khan hiếm, dịch vụ sẽ không còn trả về hình ảnh kèm văn bản hay các bản đồ nhỏ minh họa vị trí diễn ra câu chuyện. Tùy theo mục đích, một RPC có thể bị time out và không được thử lại (ví dụ, với các hình ảnh nêu trên), hoặc được thử lại với exponential backoff ngẫu nhiên hóa. Dù đã có các biện pháp an toàn này, các task vẫn lần lượt gặp sự cố và sau đó được Borg khởi động lại, khiến số lượng task đang hoạt động giảm xuống còn ít hơn nữa.

Hệ quả là, một số biểu đồ trên dashboard dịch vụ chuyển sang sắc đỏ đáng báo động và SRE bị gọi trực (page). Để xử lý, các SRE tạm thời bổ sung sức chứa cho datacenter châu Á bằng cách tăng số lượng task khả dụng cho job Shakespeare. Nhờ đó, họ khôi phục được dịch vụ Shakespeare trong cluster châu Á.

Sau đó, đội SRE viết postmortem (báo cáo sau sự cố) mô tả chi tiết chuỗi sự kiện, những gì đã diễn ra tốt, những gì có thể làm tốt hơn, cùng một số hạng mục công việc nhằm ngăn kịch bản này tái diễn. Chẳng hạn, khi một dịch vụ quá tải, load balancer GSLB sẽ chuyển một phần traffic sang các datacenter lân cận. Bên cạnh đó, đội SRE kích hoạt autoscaling (tự động điều chỉnh sức chứa) để số lượng task tự động tăng theo traffic, giúp họ không còn phải lo ngại về loại vấn đề này.

## Lời kết (Closing Remarks)

Khi hệ thống quá tải, bắt buộc phải có sự nhượng bộ để xử lý tình huống. Nếu một dịch vụ vượt quá ngưỡng chịu tải, tốt hơn hết là chấp nhận để một số lỗi mà người dùng có thể nhìn thấy hoặc kết quả kém chất lượng hơn lọt qua, thay vì cố gắng đáp ứng đầy đủ mọi yêu cầu. Việc xác định các điểm gãy này nằm ở đâu và hệ thống phản ứng ra sao khi vượt qua chúng là điều thiết yếu đối với các chủ sở hữu dịch vụ muốn ngăn chặn sự cố lan truyền.

Nếu không được chăm sóc đúng cách, một số thay đổi hệ thống nhằm giảm lỗi nền tảng hoặc cải thiện độ ổn định có thể khiến dịch vụ phơi bày trước rủi ro lớn hơn về một outage hoàn toàn. Thử lại khi thất bại, dịch chuyển tải khỏi các server không khỏe, giết các server không khỏe, thêm cache để cải thiện hiệu năng hoặc giảm độ trễ: tất cả những điều này có thể được thực hiện để cải thiện trường hợp bình thường, nhưng cũng có thể làm tăng cơ hội gây ra một sự cố quy mô lớn. Hãy cẩn thận khi đánh giá các thay đổi để đảm bảo rằng một outage không bị đánh đổi lấy một outage khác.

<a id="fn1"></a>[1](#fn1) Xem Wikipedia, “Positive feedback,” [*https://en.wikipedia.org/wiki/Positive_feedback*](https://en.wikipedia.org/wiki/Positive_feedback).

<a id="fn2"></a>[2](#fn2) Một watchdog thường được cài đặt như một thread thức dậy định kỳ để xem liệu công việc đã được thực hiện kể từ lần kiểm tra trước hay không. Nếu không, nó giả định rằng server bị kẹt và giết nó. Ví dụ, các yêu cầu của một loại đã biết có thể được gửi đến server ở các khoảng thời gian đều đặn; nếu một yêu cầu không được nhận hoặc xử lý như mong đợi, điều này có thể chỉ ra sự cố — của server, của hệ thống đang gửi yêu cầu, hoặc của mạng trung gian.

<a id="fn3"></a>[3](#fn3) Đây thường không phải là một giả định tốt do yếu tố địa lý; xem thêm [Tổ chức Job và Dữ liệu](https://sre.google/sre-book/production-environment/#t-to-chuc-job-va-du-lieu).

<a id="fn4"></a>[4](#fn4) Một bài tập hữu ích, để lại cho người đọc: viết một trình mô phỏng đơn giản và xem lượng công việc hữu ích mà backend có thể thực hiện thay đổi như thế nào theo mức độ nó bị quá tải và bao nhiêu retries được cho phép.

<a id="fn5"></a>[5](#fn5) Đôi khi bạn phát hiện ra rằng một tỷ lệ đáng kể sức chứa phục vụ thực tế của bạn là một hàm của việc phục vụ từ một cache, và nếu bạn mất quyền truy cập vào cache đó, bạn thực sự sẽ không thể phục vụ nhiều truy vấn như vậy. Một quan sát tương tự cũng đúng với độ trễ: một cache có thể giúp bạn đạt được các mục tiêu độ trễ (bằng cách giảm thời gian phản hồi trung bình khi truy vấn có thể được phục vụ từ cache) mà có lẽ bạn không thể đáp ứng nếu không có cache đó.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
