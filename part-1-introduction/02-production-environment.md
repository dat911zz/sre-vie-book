# Chương 2. Môi trường Production tại Google, từ góc nhìn của một SRE (The Production Environment at Google, from the Viewpoint of an SRE)

> **Nguyên bản:** [Chapter 2 - The Production Environment at Google, from the Viewpoint of an SRE](https://sre.google/sre-book/production-environment/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* JC van Winkel
*Biên tập:* Betsy Beyer

Các datacenter (trung tâm dữ liệu) của Google rất khác so với phần lớn datacenter thông thường và các trang farm server quy mô nhỏ. Những khác biệt này mang đến cả vấn đề lẫn cơ hội. Chương này thảo luận những thách thức và cơ hội đặc trưng cho các datacenter của Google và giới thiệu các thuật ngữ được sử dụng xuyên suốt quyển sách.

## Phần cứng (Hardware)

Phần lớn tài nguyên tính toán (compute resources) của Google nằm trong các datacenter mà Google tự thiết kế, với hệ thống phân phối điện, làm mát, mạng và phần cứng tính toán độc quyền (xem [[Bar13]](https://sre.google/sre-book/bibliography#Bar13)). Khác với datacenter colocation (thuê chỗ đặt thiết bị) "chuẩn", phần cứng tính toán trong datacenter do Google thiết kế là đồng nhất từ trên xuống dưới.<sup>[1](#fn1)</sup> Để loại bỏ sự nhầm lẫn giữa phần cứng server và phần mềm server, chúng tôi sử dụng các thuật ngữ sau xuyên suốt quyển sách:

#### Máy (Machine)

Một phần cứng (hoặc có thể là một VM — máy ảo)

**Server**

Một phần mềm triển khai một dịch vụ

Vì mỗi máy có thể chạy bất kỳ server nào, chúng tôi không gán riêng các máy cụ thể cho các chương trình server cụ thể. Chẳng hạn, không có máy nào cố định chạy mail server của chúng tôi. Thay vào đó, hệ điều hành cluster (cụm máy) *Borg* của chúng tôi đảm nhận việc phân bổ tài nguyên.

Chúng tôi thừa nhận cách dùng từ *server* ở đây khá khác thường. Theo nghĩa thông thường, *server* gộp chung “binary (file thực thi) nhận kết nối mạng” với *machine*, nhưng khi bàn về tính toán tại Google, việc phân biệt hai khái niệm này lại rất quan trọng. Khi quen với cách dùng *server* của chúng tôi, bạn sẽ hiểu vì sao thuật ngữ chuyên biệt này có ý nghĩa — không chỉ trong Google mà còn xuyên suốt quyển sách.

[Hình 2-1](#fig-2-1) minh họa topology (sơ đồ mạng) của một datacenter Google:

-   Hàng chục machine được đặt trong một *rack* (giá rack).
-   Các rack đứng trong một *row* (hàng).
-   Một hoặc nhiều row tạo thành một *cluster* (cụm máy).
-   Thông thường một tòa nhà *datacenter* chứa nhiều cluster.
-   Nhiều tòa nhà datacenter nằm gần nhau tạo thành một *campus* (khu).


<a id="fig-2-1"></a>![Hình 2-1](../assets/imgs/fig-2-1.jpg)

[Hình 2-1.](#fig-2-1) Topology ví dụ của một campus datacenter Google.

Các machine trong datacenter cần giao tiếp với nhau, nên chúng tôi tạo ra một virtual switch (bộ chuyển mạch ảo) rất nhanh với hàng chục nghìn cổng — kết nối hàng trăm switch do Google tự xây trong mạng dạng Clos [[Clos53]](https://sre.google/sre-book/bibliography#Clos53), đặt tên *Jupiter* [[Sin15]](https://sre.google/sre-book/bibliography#Sin15). Trong cấu hình lớn nhất, Jupiter hỗ trợ bisection bandwidth (băng thông khi chia đôi mạng) 1.3 Pbps giữa các server.

Các datacenter được kết nối với nhau qua mạng backbone toàn cầu *B4* [[Jai13]](https://sre.google/sre-book/bibliography#Jai13) của chúng tôi — một kiến trúc SDN (mạng định nghĩa bằng phần mềm) chạy trên giao thức chuẩn mở OpenFlow. Hệ thống này cung cấp băng thông lớn cho một số lượng site vừa phải, đồng thời sử dụng cơ chế phân bổ băng thông co giãn (elastic bandwidth allocation) nhằm tối đa hóa băng thông trung bình [[Kum15]](https://sre.google/sre-book/bibliography#Kum15).

## Phần mềm hệ thống "tổ chức" phần cứng

Phần cứng của chúng tôi phải được kiểm soát và quản trị bằng phần mềm có khả năng xử lý quy mô khổng lồ. Hỏng hóc phần cứng là vấn đề nổi bật mà chúng tôi xử lý bằng phần mềm. Với số lượng lớn các thành phần phần cứng trong một cluster, sự hỏng hóc phần cứng xảy ra khá thường xuyên. Chỉ trong một cluster, một năm điển hình đã có hàng nghìn machine hỏng và hàng nghìn ổ cứng (hard disk) hỏng; nhân lên với số cluster trên toàn cầu, con số này trở nên rất lớn. Do đó, chúng tôi muốn che giấu những vấn đề này khỏi người dùng, và các nhóm vận hành dịch vụ cũng không muốn bị làm phiền bởi sự hỏng hóc phần cứng. Mỗi campus datacenter có các nhóm chuyên [bảo trì phần cứng và cơ sở hạ tầng datacenter](https://sre.google/sre-book/managing-critical-state/).

## Quản lý các machine (Managing Machines)

*Borg* (xem [Hình 2-2](#fig-2-2)) là hệ điều hành cluster phân tán [[Ver15]](https://sre.google/sre-book/bibliography#Ver15), tương tự Apache Mesos.<sup>[2](#fn2)</sup> Hệ thống này quản lý job (công việc) ở cấp độ cluster.


<a id="fig-2-2"></a>![Hình 2-2](../assets/imgs/fig-2-2.jpg)

[Hình 2-2.](#fig-2-2) Kiến trúc cấp cao của cluster Borg.

Borg đảm nhận việc chạy các *job* của người dùng, bao gồm cả server chạy vĩnh viễn lẫn các quá trình batch (lô) như MapReduce [[Dea04]](https://sre.google/sre-book/bibliography#Dea04). Một job có thể bao gồm nhiều (đôi khi lên tới hàng nghìn) *task* (nhiệm vụ) giống hệt nhau, nhằm đảm bảo độ tin cậy và vì một process (quá trình) đơn lẻ thường không xử lý nổi toàn bộ traffic của cluster. Khi khởi động một job, Borg sẽ tìm các machine cho các task và ra lệnh cho chúng khởi động chương trình server. Sau đó, Borg liên tục giám sát các task này. Nếu một task gặp sự cố, nó sẽ bị kill (giết) và khởi động lại, có thể trên một machine khác.

Do task được phân bổ linh hoạt trên các machine, chúng tôi không thể chỉ dựa vào địa chỉ IP và số cổng để tham chiếu. Giải pháp là thêm một tầng gián tiếp (indirection): khi khởi động một job, Borg cấp phát một tên và số index cho mỗi task bằng *Borg Naming Service* (BNS). Thay vì dùng địa chỉ IP và số cổng, các process khác kết nối đến task của Borg qua tên BNS — BNS sẽ dịch tên đó thành địa chỉ IP và số cổng. Ví dụ, đường dẫn BNS có thể là một chuỗi như `/bns/<*cluster*>/<*user*>/<*job name*>/<*task number*>`, sẽ được phân giải thành `<*IP address*>:<*port*>`.

Borg cũng đảm nhận việc phân bổ tài nguyên cho job. Mỗi job phải khai báo tài nguyên cần dùng (ví dụ: 3 core CPU, 2 GiB RAM). Dựa trên danh sách yêu cầu của toàn bộ job, Borg thực hiện bin-packing task lên các machine theo cách tối ưu, đồng thời tính đến failure domain. Chẳng hạn, Borg không chạy tất cả task của một job trên cùng một rack — làm vậy sẽ biến switch đầu rack thành điểm thất bại duy nhất (single point of failure) cho job đó.

Nếu một task cố dùng nhiều tài nguyên hơn mức đã yêu cầu, Borg sẽ kill task đó và khởi động lại (một task rơi vào crashloop — vòng lặp sập — còn tốt hơn một task không bao giờ được khởi động lại).

## Lưu trữ (Storage)

Task có thể dùng disk cục bộ trên machine như scratch pad (vùng đệm tạm), nhưng chúng tôi có nhiều tùy chọn lưu trữ cluster cho lưu trữ vĩnh viễn (và cả không gian tạm cũng sẽ chuyển sang mô hình lưu trữ cluster). Những tùy chọn này tương đương Lustre và Hadoop Distributed File System (HDFS) — đều là filesystem (hệ thống tệp) cluster mã nguồn mở.

Tầng lưu trữ đảm bảo người dùng truy cập dễ dàng và đáng tin cậy vào dung lượng lưu trữ có sẵn của cluster. Như [Hình 2-3](#fig-2-3) cho thấy, lưu trữ được chia thành nhiều tầng:

1.  Tầng thấp nhất, gọi là *D* (disk — ổ đĩa, dù D dùng cả ổ quay lẫn flash storage), là fileserver (server tệp) chạy trên hầu hết machine trong cluster. Tuy nhiên, người dùng không muốn phải nhớ machine nào đang giữ dữ liệu của mình — đây là lúc tầng tiếp theo vào cuộc.
2.  Tầng nằm trên D, gọi là *Colossus*, tạo ra một filesystem toàn cluster với ngữ nghĩa (semantics) chuẩn của filesystem, kèm sao chép (replication) và mã hóa (encryption). Colossus là người kế nhiệm của GFS, Google File System [[Ghe03]](https://sre.google/sre-book/bibliography#Ghe03).
3.  Có một số dịch vụ giống database (cơ sở dữ liệu) được xây dựng trên Colossus:
    1.  Bigtable [[Cha06]](https://sre.google/sre-book/bibliography#Cha06) là hệ thống database NoSQL xử lý được database hàng petabyte. Một Bigtable là map (cấu trúc ánh xạ) đa chiều, phân tán, bền vững, thưa (sparse), index theo row key (chìa hàng), column key (chìa cột) và timestamp (dấu thời gian); mỗi giá trị là mảng byte không diễn giải. Bigtable hỗ trợ replication xuyên datacenter theo kiểu eventually consistent (nhất quán cuối cùng).
    2.  Spanner [[Cor12]](https://sre.google/sre-book/bibliography#Cor12) cung cấp giao diện giống SQL cho người dùng cần consistency (nhất quán) thực sự trên toàn cầu.
    3.  Vài hệ thống database khác như *Blobstore* cũng có sẵn. Mỗi tùy chọn có trade-off (đánh đổi) riêng (xem [Data Integrity: What You Read Is What You Wrote](https://sre.google/sre-book/data-integrity/)).


<a id="fig-2-3"></a>![Hình 2-3](../assets/imgs/fig-2-3.jpg)

[Hình 2-3.](#fig-2-3) Các thành phần của stack lưu trữ Google.

## Mạng (Networking)

Phần cứng mạng của Google được quản lý theo nhiều cách. Như đã đề cập, chúng tôi sử dụng mạng định nghĩa bằng phần mềm (SDN) dựa trên OpenFlow. Thay vì dùng phần cứng định tuyến “thông minh”, chúng tôi chọn các thiết bị chuyển mạch “đơn giản” (dumb) có giá thành thấp hơn, kết hợp với một bộ điều khiển tập trung (có bản sao) để tính toán trước đường đi tối ưu xuyên suốt mạng. Nhờ đó, các quyết định định tuyến tiêu tốn tài nguyên tính toán được tách khỏi router, thay thế bằng phần cứng chuyển mạch đơn giản.

Băng thông mạng cần được phân bổ khôn ngoan. Tương tự cách Borg giới hạn tài nguyên tính toán cho task, Bandwidth Enforcer (BwE) quản lý băng thông có sẵn nhằm tối đa hóa băng thông trung bình. Tối ưu băng thông không chỉ là chuyện chi phí: centralized traffic engineering (kỹ thuật traffic tập trung) đã chứng minh khả năng giải quyết nhiều vấn đề mà định tuyến phân tán truyền thống cực kỳ khó xử lý [[Kum15]](https://sre.google/sre-book/bibliography#Kum15).

Một số dịch vụ chạy job trên nhiều cluster trải khắp toàn cầu. Để giảm độ trễ cho các dịch vụ phân tán này, chúng tôi định tuyến người dùng đến datacenter gần nhất có đủ năng lực. *Global Software Load Balancer* (GSLB) của chúng tôi thực hiện load balancing (cân bằng tải) ở ba cấp:

-   Load balancing địa lý cho các yêu cầu DNS (ví dụ, đến *www.google.com*), được mô tả trong [Load Balancing at the Frontend](https://sre.google/sre-book/load-balancing-frontend/)
-   Load balancing ở cấp độ dịch vụ người dùng (ví dụ, YouTube hay Google Maps)
-   Load balancing ở cấp độ Remote Procedure Call (RPC — lời gọi thủ tục từ xa), được mô tả trong [Load Balancing in the Datacenter](https://sre.google/sre-book/load-balancing-datacenter/)

Chủ sở hữu dịch vụ sẽ đặt một symbolic name (tên biểu tượng) cho dịch vụ, liệt kê các địa chỉ BNS của server và chỉ rõ năng lực có sẵn tại từng địa điểm (đo bằng queries per second — QPS). Sau đó, GSLB sẽ hướng traffic đến các địa chỉ BNS này.

## Các phần mềm hệ thống khác

Một số thành phần khác trong datacenter cũng quan trọng.

## Dịch vụ khóa (Lock Service)

Dịch vụ khóa *Chubby* [[Bur06]](https://sre.google/sre-book/bibliography#Bur06) cung cấp API giống filesystem để duy trì lock (khóa). Chubby xử lý lock xuyên suốt các datacenter. Nó dùng giao thức Paxos cho Consensus (nhất trí) bất đồng bộ (xem [Managing Critical State: Distributed Consensus for Reliability](https://sre.google/sre-book/managing-critical-state/)).

Chubby cũng đóng vai trò quan trọng trong master election (bầu chọn master). Khi một dịch vụ chạy năm replica của một job vì độ tin cậy nhưng chỉ một replica được phép thực hiện công việc, Chubby sẽ chọn *replica nào* được tiếp tục.

Chubby phù hợp để lưu dữ liệu cần tính nhất quán. Do đó, BNS dùng Chubby để lưu mapping (ánh xạ) giữa đường dẫn BNS và các cặp `địa chỉ IP:cổng`.

## Giám sát và cảnh báo (Monitoring and Alerting)

Chúng tôi muốn đảm bảo mọi dịch vụ đều hoạt động đúng yêu cầu. Vì vậy, chúng tôi triển khai nhiều instance của *Borgmon* — chương trình giám sát nội bộ (xem [Practical Alerting from Time-Series Data](https://sre.google/sre-book/practical-alerting/)). Borgmon định kỳ "scrape" (vét) metrics (chỉ số) từ các server. Các metrics này vừa dùng ngay cho cảnh báo, vừa được lưu trữ để xem tổng quan lịch sử (ví dụ: biểu đồ). Chúng tôi có thể sử dụng hệ thống giám sát theo một số cách:

-   Thiết lập cảnh báo cho các vấn đề cấp bách.
-   So sánh hành vi: một bản cập nhật phần mềm có khiến server nhanh hơn không?
-   Xem xét cách hành vi tiêu thụ tài nguyên diễn tiến theo thời gian, điều này là thiết yếu cho lập kế hoạch năng lực.

## Cơ sở hạ tầng phần mềm của chúng tôi

Kiến trúc phần mềm được thiết kế nhằm khai thác hiệu quả nhất cơ sở hạ tầng phần cứng. Code đa luồng (multithreaded) mạnh, một task có thể dùng nhiều core dễ dàng. Để hỗ trợ dashboard, giám sát và debug, mỗi server có một HTTP server cung cấp thông tin chẩn đoán và thống kê cho task.

Tất cả dịch vụ của Google liên lạc với nhau qua hạ tầng Remote Procedure Call (RPC) mang tên *Stubby*; phiên bản mã nguồn mở gRPC cũng có sẵn.<sup>[3](#fn3)</sup> Thông thường, một lời gọi RPC vẫn được phát ra dù thực chất chỉ là gọi subroutine (tiểu trình) trong chương trình cục bộ. Cách làm này giúp việc refactor lời gọi sang server khác trở nên dễ dàng khi cần mô-đun hóa hơn, hoặc khi codebase của server lớn lên. GSLB có thể load balance RPC theo cách tương tự như load balance các dịch vụ bên ngoài.

Server nhận yêu cầu RPC từ *frontend* (phía trước) và gửi RPC đến *backend* (phía sau). Theo thuật ngữ truyền thống, frontend đóng vai trò client (phía gọi), còn backend đóng vai trò server (phía phục vụ).

Dữ liệu truyền đến và từ RPC dùng *protocol buffers* (bộ đệm giao thức),<sup>[4](#fn4)</sup> viết tắt "protobufs", tương tự Apache Thrift. So với XML, protocol buffers vượt trội hơn trong serialization (phân chuỗi) dữ liệu có cấu trúc: đơn giản hơn, nhỏ hơn 3–10 lần, nhanh hơn 20–100 lần, và ít mơ hồ hơn.

## Môi trường phát triển của chúng tôi (Development Environment)

Tốc độ phát triển (development velocity) rất quan trọng với Google, nên chúng tôi xây dựng môi trường phát triển hoàn chỉnh để tận dụng hạ tầng [[Mor12b]](https://sre.google/sre-book/bibliography#Mor12b).

Ngoại trừ vài nhóm có repository mã nguồn mở riêng (ví dụ: Android, Chrome), kỹ sư phần mềm tại Google làm việc từ một repository (kho lưu trữ) chia sẻ duy nhất [[Pot16]](https://sre.google/sre-book/bibliography#Pot16). Điều này kéo theo vài hệ quả thực tiễn quan trọng:

-   Kỹ sư gặp vấn đề trong thành phần ngoài dự án của mình có thể sửa luôn, gửi thay đổi ("changelist," hay *CL*) cho chủ sở hữu review, rồi nộp CL vào mainline.
-   Các thay đổi đối với mã nguồn trong dự án riêng của một kỹ sư yêu cầu một cuộc xem xét. Tất cả phần mềm đều được xem xét trước khi được nộp.

Khi build (dựng) phần mềm, yêu cầu build được gửi đến build server trong datacenter. Nhờ nhiều build server compile (biên dịch) song song, ngay cả build lớn cũng chạy nhanh. Hạ tầng này cũng phục vụ cho continuous testing (kiểm thử liên tục). Mỗi khi nộp CL, kiểm thử sẽ chạy trên toàn bộ phần mềm có thể phụ thuộc vào CL đó, dù là trực tiếp hay gián tiếp. Nếu framework phát hiện thay đổi có nguy cơ làm hỏng các phần khác trong hệ thống, nó sẽ thông báo cho chủ sở hữu. Một số dự án áp dụng push-on-green (đẩy khi xanh): phiên bản mới được tự động đẩy lên production sau khi vượt qua kiểm thử.

## Shakespeare: Một dịch vụ mẫu (A Sample Service)

Để hình dung cách một dịch vụ được triển khai trong môi trường production của Google, hãy xem xét một dịch vụ ví dụ tương tác với nhiều công nghệ Google. Giả sử chúng tôi muốn cung cấp dịch vụ cho phép tìm xem một từ nhất định xuất hiện ở đâu trong tất cả tác phẩm của Shakespeare.

Chúng ta có thể chia hệ thống này thành hai phần:

-   Thành phần batch đọc toàn bộ văn bản Shakespeare, tạo index (chỉ mục) rồi ghi vào Bigtable. Job này chỉ cần chạy một lần, hoặc rất hiếm khi (bảo chưa thể có văn bản mới xuất hiện!).
-   Frontend ứng dụng xử lý yêu cầu từ người dùng cuối. Job này luôn chạy vì người dùng ở mọi múi giờ đều có thể muốn tìm trong tác phẩm Shakespeare.

Thành phần batch là một MapReduce gồm ba giai đoạn.

Giai đoạn mapping đọc văn bản Shakespeare, tách thành các từ riêng lẻ — nhanh hơn khi nhiều worker (công nhân) chạy song song.

Giai đoạn shuffle sắp xếp các tuple (bộ) theo từ.

Trong giai đoạn reduce, một tuple (*word — từ*, *list of locations — danh sách các vị trí*) được tạo ra.

Mỗi tuple được viết vào một hàng trong Bigtable, sử dụng từ làm key (chìa khóa).

## Vòng đời của một yêu cầu (Life of a Request)

[Hình 2-4](#fig-2-4) minh họa cách xử lý yêu cầu từ người dùng. Đầu tiên, người dùng trỏ trình duyệt đến *shakespeare.google.com*. Thiết bị sẽ phân giải địa chỉ thông qua [DNS server](https://sre.google/sre-book/load-balancing-frontend/) (1). Yêu cầu được chuyển đến DNS server của Google, nơi này liên lạc với GSLB. Vì GSLB theo dõi tải traffic giữa các frontend server ở các khu vực khác nhau, nên nó sẽ chọn địa chỉ IP server phù hợp để gửi lại cho người dùng.


<a id="fig-2-4"></a>![Hình 2-4](../assets/imgs/fig-2-4.jpg)

[Hình 2-4.](#fig-2-4) Vòng đời của một yêu cầu.

Trình duyệt kết nối đến HTTP server trên IP này. Server đó (Google Frontend — GFE) đóng vai trò reverse proxy (proxy ngược) và terminate (kết thúc) kết nối TCP (2). GFE tra cứu dịch vụ được yêu cầu (tìm kiếm web, maps, hay — ở đây — Shakespeare). Tiếp đó, nhờ GSLB, server tìm một frontend server Shakespeare khả dụng và gửi RPC chứa yêu cầu HTTP (3).

Server Shakespeare phân tích yêu cầu HTTP và xây dựng protobuf chứa từ cần tra cứu. Để liên lạc với backend server, frontend server Shakespeare hỏi GSLB lấy địa chỉ BNS của một backend server phù hợp, không quá tải (4). Sau đó, backend server này liên lạc với Bigtable server để lấy dữ liệu (5).

Kết quả được ghi vào protobuf reply và gửi về backend server Shakespeare. Backend chuyển protobuf chứa kết quả sang frontend server Shakespeare, nơi tiến hành assemble (lắp ráp) HTML trước khi trả về cho người dùng.

Toàn bộ chuỗi sự kiện này diễn ra trong nháy mắt — chỉ vài trăm mili giây! Vì liên quan đến nhiều bộ phận nên có nhiều điểm lỗi tiềm năng; đặc biệt, nếu GSLB hỏng sẽ gây hỗn loạn. Tuy nhiên, chính sách kiểm thử nghiêm ngặt và quy trình triển khai cẩn thận của Google, cùng các phương pháp phục hồi lỗi chủ động như graceful degradation (suy giảm nhẹ nhàng), cho phép chúng tôi cung cấp [dịch vụ đáng tin cậy](https://sre.google/sre-book/part-III-practices/) mà người dùng đã quen. Sau cùng, người ta thường dùng *www.google.com* để kiểm tra kết nối Internet.

## Tổ chức job và dữ liệu (Job and Data Organization)

Kiểm tra tải cho thấy máy chủ backend xử lý được khoảng 100 query/giây (QPS). Thử nghiệm với nhóm người dùng giới hạn cho thấy kỳ vọng tải đỉnh khoảng 3.470 QPS, do đó cần ít nhất 35 task. Tuy nhiên, các cân nhắc sau đòi hỏi ít nhất 37 task (N+2):

-   Trong quá trình cập nhật, một task một lúc sẽ không khả dụng, còn lại 36 task.
-   Một sự hỏng hóc machine có thể xảy ra trong khi cập nhật một task, còn lại chỉ 35 task, vừa đủ để phục vụ tải đỉnh.<sup>[5](#fn5)</sup>

Đào sâu vào traffic người dùng, chúng tôi thấy peak usage phân bố khắp toàn cầu: 1.430 QPS từ Bắc Mỹ, 290 từ Nam Mỹ, 1.400 từ châu Âu và châu Phi, 350 từ châu Á và Australia. Thay vì đặt toàn bộ backend tại một site, chúng tôi trải chúng ra khắp nước Mỹ, Nam Mỹ, châu Âu và châu Á. Áp dụng dự phòng N+2 cho từng khu vực, kết quả là 17 task ở nước Mỹ, 16 ở châu Âu và 6 ở châu Á. Riêng tại Nam Mỹ, chúng tôi chỉ chạy 4 task (thay vì 5) để hạ mức dự phòng từ N+2 xuống N+1 — chấp nhận rủi ro nhỏ về độ trễ cao hơn nhằm đổi lấy chi phí phần cứng thấp hơn: nếu GSLB chuyển traffic sang lục địa khác khi datacenter Nam Mỹ quá tải, hệ thống sẽ tiết kiệm 20% chi phí phần cứng. Ở các khu vực lớn hơn, task được trải trên 2–3 cluster để tăng resilience (khả năng phục hồi).

Vì backend phải liên lạc với Bigtable để truy xuất dữ liệu, phần lưu trữ cũng đòi hỏi một chiến lược thiết kế phù hợp. Nếu backend tại châu Á gọi Bigtable đặt ở Mỹ, độ trễ sẽ tăng đáng kể, nên chúng tôi đã sao chép (replicate) Bigtable tại mỗi khu vực. Việc sao chép này mang lại hai lợi ích: giúp hệ thống phục hồi khi một server Bigtable gặp sự cố, đồng thời giảm độ trễ khi truy cập dữ liệu. Bigtable chỉ đảm bảo tính nhất quán cuối cùng (eventual consistency), nhưng điều này không gây trở ngại vì chúng tôi không cập nhật nội dung thường xuyên.

Chương này giới thiệu rất nhiều thuật ngữ; dù không cần nhớ tất cả, chúng sẽ là nền tảng cho nhiều hệ thống được tham khảo ở các chương sau.

<a id="fn1"></a>[1](#fn1) Chà, *xấp xỉ* giống nhau. Phần lớn là. Ngoại trừ những thứ khác nhau. Một số datacenter kết thúc với nhiều thế hệ phần cứng tính toán, và đôi khi chúng tôi bổ sung các datacenter sau khi chúng được xây dựng. Nhưng phần lớn, phần cứng datacenter của chúng tôi là đồng nhất.

<a id="fn2"></a>[2](#fn2) Một số người đọc có thể quen thuộc hơn với hậu duệ của Borg, Kubernetes — một khung điều phối Container Cluster mã nguồn mở do Google khởi động vào năm 2014; xem [*https://kubernetes.io*](https://kubernetes.io) và [[Bur16]](https://sre.google/sre-book/bibliography#Bur16). Để biết thêm chi tiết về sự tương đồng giữa Borg và Apache Mesos, xem [[Ver15]](https://sre.google/sre-book/bibliography#Ver15).

<a id="fn3"></a>[3](#fn3) Xem [*https://grpc.io*](https://grpc.io).

<a id="fn4"></a>[4](#fn4) Protocol buffers là một cơ chế mở rộng trung lập ngôn ngữ, trung lập nền tảng, để phân chuỗi dữ liệu có cấu trúc. Để biết thêm chi tiết, xem [*https://developers.google.com/protocol-buffers/*](https://developers.google.com/protocol-buffers/).

<a id="fn5"></a>[5](#fn5) Chúng tôi giả định rằng xác suất hai task hỏng đồng thời trong môi trường của chúng tôi đủ thấp để có thể bỏ qua. Các điểm thất bại duy nhất, như switch đầu rack hoặc phân phối điện, có thể làm cho giả định này không hợp lệ trong các môi trường khác.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
