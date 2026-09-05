> **Nguyên bản:** [Chapter 23 - Managing Critical State: Distributed Consensus for Reliability](https://sre.google/sre-book/managing-critical-state/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

## Chương 23. Quản Lý Trạng Thái Quan Trọng: Nhất Trí Phân Tán Vì Độ Tin Cậy (Managing Critical State: Distributed Consensus for Reliability)

Tác giả: Laura Nolan
Biên tập: Tim Harvey

Các tiến trình sẽ crash hoặc có thể cần khởi động lại. Ổ cứng sẽ hỏng. Thảm họa tự nhiên có thể khiến nhiều datacenter trong một khu vực bị tê liệt. Các Site Reliability Engineer (SRE) cần dự liệu được những kiểu lỗi này và xây dựng các chiến lược để hệ thống vẫn hoạt động bất chấp chúng. Những chiến lược này thường bao gồm việc chạy hệ thống trải dài trên nhiều site. Việc phân bố một hệ thống theo địa lý là tương đối đơn giản, nhưng đồng thời phát sinh nhu cầu duy trì một cái nhìn nhất quán về trạng thái hệ thống — một bài toán tinh tế và khó khăn hơn.

Các nhóm tiến trình có thể muốn thống nhất một cách tin cậy về những câu hỏi như:

- Tiến trình nào là leader của một nhóm tiến trình?
- Tập hợp các tiến trình trong một nhóm là gì?
- Một tin nhắn đã được commit thành công vào một hàng đợi phân tán (distributed queue) chưa?
- Một tiến trình đang giữ lease hay không?
- Giá trị trong một datastore cho một key nhất định là gì?

Chúng tôi nhận thấy distributed consensus rất hiệu quả khi xây dựng các hệ thống tin cậy, có khả dụng cao, đòi hỏi một cái nhìn nhất quán về một phần nào đó của trạng thái hệ thống. Bài toán distributed consensus giải quyết việc đạt được sự đồng thuận giữa một nhóm các tiến trình được kết nối qua một mạng truyền thông không tin cậy. Ví dụ, một vài tiến trình trong một [hệ thống phân tán](https://sre.google/sre-book/monitoring-distributed-systems/) có thể cần tạo ra được một cái nhìn nhất quán về một phần cấu hình quan trọng, về việc một distributed lock có đang được giữ hay không, hay một tin nhắn trên hàng đợi đã được xử lý hay chưa. Đây là một trong những khái niệm nền tảng nhất của tính toán phân tán và là thứ mà chúng tôi dựa vào cho gần như mọi dịch vụ chúng tôi cung cấp. [Hình 23-1](#hinh-23-1) minh họa một mô hình đơn giản về cách một nhóm tiến trình có thể đạt được một cái nhìn nhất quán về trạng thái hệ thống thông qua distributed consensus.

<a id="hinh-23-1"></a>        ![Nhất trí phân tán: sự đồng thuận giữa một nhóm các tiến trình.](../assets/imgs/fig-23-1.jpg)

Hình 23-1. Distributed consensus: sự đồng thuận giữa một nhóm các tiến trình

Khi gặp leader election, trạng thái chia sẻ quan trọng, hay distributed locking, chúng tôi khuyên bạn nên dùng *các hệ thống distributed consensus đã được chứng minh chính thức và thử nghiệm kỹ lưỡng*. Những cách tiếp cận không chính thức để giải quyết bài toán này có thể gây ra các sự cố (outage), và tinh vi hơn, dẫn đến những vấn đề nhất quán dữ liệu vi tế, khó sửa, khiến sự cố trong hệ thống của bạn kéo dài không cần thiết.

## Định lý CAP (CAP Theorem)

Định lý CAP ([\[Fox99\]](https://sre.google/sre-book/bibliography#Fox99), [\[Bre12\]](https://sre.google/sre-book/bibliography#Bre12)) khẳng định rằng một hệ thống phân tán không thể đồng thời có cả ba tính chất sau:

- Các cái nhìn nhất quán về dữ liệu tại mỗi node (nút)
- Khả dụng của dữ liệu tại mỗi node
- Khả năng chịu được việc phân đoạn mạng (network partition) [\[Gil02\]](https://sre.google/sre-book/bibliography#Gil02)

Logic ở đây rất trực quan: nếu hai node không thể giao tiếp được (do mạng bị phân đoạn), toàn bộ hệ thống buộc phải chọn một trong hai hướng: hoặc ngừng phục vụ một phần hay toàn bộ các request tại một số hay toàn bộ các node (khiến khả dụng giảm), hoặc tiếp tục phục vụ các request như bình thường, dẫn đến việc mỗi node có những cái nhìn không nhất quán về dữ liệu.

Vì network partition là điều không thể tránh khỏi (do cáp bị cắt, gói tin bị mất hoặc trễ vì tắc nghẽn, phần cứng hỏng, cấu hình sai các thành phần mạng, v.v.), việc nắm bắt distributed consensus thực chất là hiểu cách tính nhất quán (consistency) và khả dụng (availability) vận hành trong ứng dụng cụ thể của bạn. Áp lực thương mại thường đòi hỏi mức độ khả dụng cao, và nhiều ứng dụng yêu cầu các cái nhìn nhất quán về dữ liệu của chúng.

Các kỹ sư hệ thống và phần mềm thường quen thuộc với ngữ nghĩa datastore truyền thống ACID (Atomicity, Consistency, Isolation, Durability), nhưng ngày càng nhiều công nghệ datastore phân tán cung cấp một bộ ngữ nghĩa khác gọi là BASE (Basically Available, Soft state, Eventual consistency). Các datastore hỗ trợ ngữ nghĩa BASE có những ứng dụng hữu ích cho một số kiểu dữ liệu nhất định và có thể xử lý các khối lượng dữ liệu và giao dịch lớn mà sẽ tốn kém hơn nhiều — và có thể là bất khả thi — với các datastore hỗ trợ ngữ nghĩa ACID.

Phần lớn các hệ thống hỗ trợ ngữ nghĩa BASE đều dùng multimaster replication (nhân bản đa master), cho phép commit các phép ghi đồng thời đến nhiều tiến trình khác nhau, kèm theo cơ chế giải quyết xung đột (thường chỉ đơn giản là "thời gian gần nhất thắng"). Cách tiếp cận này thường được gọi là *eventual consistency*. Tuy nhiên, eventual consistency có thể gây ra những kết quả bất ngờ [\[Lu15\]](https://sre.google/sre-book/bibliography#Lu15), nhất là khi xảy ra *clock drift* (điều không thể tránh khỏi trong các hệ thống phân tán) hoặc network partition [\[Kin15\]](https://sre.google/sre-book/bibliography#Kin15).<sup>[1](#fn1)</sup>

Các nhà phát triển cũng gặp khó khăn trong việc thiết kế các hệ thống hoạt động tốt với các datastore chỉ hỗ trợ ngữ nghĩa BASE. Jeff Shute [\[Shu13\]](https://sre.google/sre-book/bibliography#Shu13), ví dụ, đã phát biểu: "chúng tôi thấy các nhà phát triển dành một phần đáng kể thời gian để xây dựng những cơ chế cực kỳ phức tạp và dễ sai khi phải đối phó với eventual consistency và xử lý dữ liệu có thể đã lỗi thời. Chúng tôi cho rằng đây là một gánh nặng không thể chấp nhận được đặt lên các nhà phát triển, và các vấn đề nhất quán nên được giải quyết ở tầng database."

Các nhà thiết kế hệ thống không thể đánh đổi tính đúng đắn lấy độ tin cậy hay hiệu năng, nhất là ở những trạng thái quan trọng. Ví dụ, trong một hệ thống xử lý giao dịch tài chính, các yêu cầu về độ tin cậy hoặc hiệu năng sẽ không có nhiều ý nghĩa nếu dữ liệu tài chính không chính xác. Hệ thống cần khả năng đồng bộ hóa tin cậy các trạng thái quan trọng giữa nhiều tiến trình, và các thuật toán distributed consensus đảm nhận chức năng này.

## Động Lực Sử Dụng Consensus: Sự Cố Phối Hợp Trong Hệ Thống Phân Tán (Motivating the Use of Consensus: Distributed Systems Coordination Failure)

Các hệ thống phân tán rất phức tạp và tinh vi, khiến việc hiểu, giám sát và gỡ lỗi chúng trở nên khó khăn. Các kỹ sư vận hành những hệ thống này thường bất ngờ trước hành vi của chúng khi xảy ra lỗi. Lỗi là những sự kiện tương đối hiếm, và việc kiểm thử hệ thống trong các điều kiện này không phải là thực hành thông thường. Rất khó để suy luận về hành vi hệ thống khi có lỗi xảy ra. Các network partition đặc biệt gây thách thức — một vấn đề tưởng như do một partition hoàn toàn gây ra thực ra có thể là kết quả của:

- Một mạng rất chậm
- Một số (nhưng không phải tất cả) các tin nhắn bị rơi
- Việc throttle xảy ra theo một hướng nhưng không theo hướng kia

Các phần sau đưa ra những ví dụ về các vấn đề từng xảy ra trong các hệ thống phân tán thực tế, đồng thời thảo luận cách leader election và các thuật toán distributed consensus có thể được dùng để ngăn chặn những vấn đề tương tự.

## Nghiên Cứu Trường Hợp 1: Vấn Đề Split-Brain (Case Study 1: The Split-Brain Problem)

Một dịch vụ là kho lưu trữ nội dung hỗ trợ cộng tác giữa nhiều người dùng. Để đảm bảo độ tin cậy, dịch vụ sử dụng các cặp file server được nhân bản, đặt ở các rack khác nhau. Dịch vụ cần tránh ghi dữ liệu đồng thời vào cả hai file server trong cùng một cặp, vì điều này có thể gây hư hỏng dữ liệu (và có thể là dữ liệu không thể khôi phục).

Mỗi cặp file server gồm một leader và một follower. Các server theo dõi nhau qua heartbeat. Nếu một file server không liên lạc được với đối tác, nó sẽ gửi lệnh STONITH (Shoot The Other Node in the Head — "bắn vào đầu node kia") đến node kia để tắt nó, sau đó tự nhận quyền mastership trên các tệp của mình. Đây là phương pháp chuẩn trong ngành để giảm các vụ split-brain, mặc dù như chúng ta sẽ thấy, về mặt khái niệm nó không vững chắc.

Điều gì sẽ xảy ra nếu mạng bị chậm hoặc bắt đầu làm rơi gói tin? Trong kịch bản này, các file server sẽ vượt quá timeout heartbeat và, theo thiết kế, gửi lệnh STONITH đến các node đối tác rồi tự nhận quyền mastership. Tuy nhiên, do mạng đã bị suy giảm, một số lệnh có thể không được chuyển đến. Các cặp file server có thể rơi vào trạng thái mà cả hai node đều được kỳ vọng đang active cho cùng một tài nguyên, hoặc cả hai đều down vì cả hai đều đã phát và nhận lệnh STONITH. Điều này dẫn đến dữ liệu bị hư hỏng hoặc không khả dụng.

Vấn đề ở đây là hệ thống đang dùng các timeout đơn giản để giải quyết bài toán leader election. Leader election chính là cách phát biểu lại bài toán nhất trí phân tán bất đồng bộ (asynchronous consensus), và không thể giải đúng bằng heartbeat.

## Nghiên Cứu Trường Hợp 2: Failover Yêu Cầu Can Thiệp Con Người (Case Study 2: Failover Requires Human Intervention)

Hệ thống database chia shard dày đặc này có một primary cho mỗi shard, được nhân bản đồng bộ đến một secondary ở datacenter khác. Một hệ thống bên ngoài sẽ kiểm tra mức độ lành mạnh của các primary; nếu phát hiện primary không còn lành mạnh, hệ thống này sẽ thăng secondary lên thành primary. Trong trường hợp primary không thể xác định được mức độ lành mạnh của secondary, nó sẽ tự đặt mình vào trạng thái không khả dụng và báo cho con người can thiệp, nhằm tránh kịch bản split-brain đã thấy trong Nghiên cứu Trường Hợp 1.

Giải pháp này không gây nguy cơ mất dữ liệu, nhưng nó lại ảnh hưởng tiêu cực đến khả dụng của dữ liệu. Nó cũng làm tăng không cần thiết tải vận hành lên các kỹ sư vận hành hệ thống, và can thiệp của con người không mở rộng tốt. Kiểu sự kiện như vậy, trong đó một primary và secondary gặp vấn đề trong việc giao tiếp, rất có thể xảy ra khi có một vấn đề hạ tầng lớn hơn, lúc mà các kỹ sư ứng phó có thể đã bị quá tải bởi các tác vụ khác. Nếu mạng bị ảnh hưởng nặng đến mức một hệ thống distributed consensus không thể bầu được một master, thì một con người cũng khó mà ở vào vị trí tốt hơn để làm việc đó.

## Nghiên Cứu Trường Hợp 3: Thuật Toán Thành Viên Nhóm Hư Lỗi (Case Study 3: Faulty Group-Membership Algorithms)

Một hệ thống có thành phần đảm nhận dịch vụ lập index và tìm kiếm. Khi khởi động, các node dùng giao thức gossip để phát hiện lẫn nhau và gia nhập cluster. Cluster bầu ra một leader để phối hợp hoạt động. Nếu network partition chia đôi cluster, mỗi phía (sai lầm) sẽ bầu một master và chấp nhận các ghi và xóa, dẫn đến kịch bản split-brain và hư hỏng dữ liệu.

Việc xác định một cái nhìn nhất quán về thành viên nhóm (group membership) xuyên qua một nhóm tiến trình là một trường hợp khác của bài toán distributed consensus.

Thực tế, nhiều vấn đề trong hệ thống phân tán hóa ra chỉ là những biến thể khác nhau của distributed consensus, chẳng hạn như bầu master, quản lý thành viên nhóm, đủ loại khóa và lease phân tán (leasing), hàng đợi và nhắn tin phân tán tin cậy, hay việc duy trì bất kỳ loại trạng thái chia sẻ quan trọng nào sao cho nhất quán xuyên suốt một nhóm tiến trình. Tất cả những vấn đề này chỉ nên được giải quyết bằng các thuật toán distributed consensus đã được chứng minh đúng đắn một cách chính thức, cùng với các bản cài đặt đã được thử nghiệm rộng rãi. Các phương pháp tự phát (ad hoc) để xử lý những kiểu vấn đề này (như heartbeat và giao thức gossip) sẽ luôn tồn tại các vấn đề về độ tin cậy trên thực tế.

## Cách Distributed Consensus Hoạt Động (How Distributed Consensus Works)

Bài toán consensus có nhiều biến thể. Khi làm việc với các hệ thống phần mềm phân tán, chúng ta quan tâm đến *asynchronous distributed consensus* (nhất trí phân tán bất đồng bộ), áp dụng cho các môi trường có độ trễ truyền tin nhắn có thể không bị giới hạn. (*Synchronous consensus* áp dụng cho các hệ thống thời gian thực, trong đó phần cứng chuyên dụng đảm bảo rằng các tin nhắn luôn được truyền đi với các cam kết về thời điểm cụ thể.)

Các thuật toán distributed consensus có thể là *crash-fail* (giả định rằng các node đã crash sẽ không bao giờ quay lại hệ thống) hoặc *crash-recover* (crash rồi phục hồi). Thuật toán crash-recover hữu ích hơn nhiều, vì hầu hết các vấn đề trong hệ thống thực đều mang tính nhất thời, do mạng chậm, việc khởi động lại, v.v.

Các thuật toán có thể xử lý các lỗi Byzantine hoặc phi-Byzantine. *Byzantine failure* xảy ra khi một tiến trình truyền đi những tin nhắn sai do bug hoặc hoạt động ác ý; nó tương đối tốn kém để xử lý và ít gặp hơn.

Về mặt kỹ thuật, việc giải bài toán asynchronous distributed consensus trong thời gian có giới hạn là bất khả thi. Như *kết quả bất khả thi FLP* đạt giải Dijkstra Prize [\[Fis85\]](https://sre.google/sre-book/bibliography#Fis85) đã chứng minh, không có thuật toán asynchronous distributed consensus nào có thể đảm bảo tiến triển khi mạng không tin cậy.

Trong thực tế, chúng ta tiệm cận bài toán distributed consensus trong thời gian có giới hạn bằng cách đảm bảo hệ thống có đủ các bản sao (replica) lành mạnh và kết nối mạng để tiến triển một cách tin cậy trong phần lớn thời gian. Ngoài ra, hệ thống nên có các backoff với độ trễ ngẫu nhiên. Cách thiết lập này vừa ngăn các lần thử lại gây ra các hiệu ứng dây chuyền, vừa tránh được vấn đề dueling proposers (các proposer đấu nhau) được mô tả sau trong chương này. Các giao thức đảm bảo an toàn (safety), và độ dự phòng đủ trong hệ thống khuyến khích hoạt động sống (liveness).

Giải pháp ban đầu cho bài toán distributed consensus là giao thức Paxos của Lamport [\[Lam98\]](https://sre.google/sre-book/bibliography#Lam98), bên cạnh đó còn có các giao thức khác như Raft [\[Ong14\]](https://sre.google/sre-book/bibliography#Ong14), Zab [\[Jun11\]](https://sre.google/sre-book/bibliography#Jun11) và Mencius [\[Mao08\]](https://sre.google/sre-book/bibliography#Mao08). Bản thân Paxos cũng có nhiều biến thể nhằm tăng hiệu năng [\[Zoo14\]](https://sre.google/sre-book/bibliography#Zoo14). Các biến thể này thường chỉ khác nhau ở một chi tiết duy nhất, chẳng hạn như trao vai trò leader đặc biệt cho một tiến trình để đơn giản hóa giao thức.

## Tổng Quan Về Paxos: Một Giao Thức Ví Dụ (Paxos Overview: An Example Protocol)

Paxos hoạt động dựa trên các proposal (đề xuất), có thể được hoặc không được đa số tiến trình trong hệ thống chấp nhận. Nếu một proposal không được chấp nhận, nó sẽ thất bại. Mỗi proposal có một sequence number, qua đó áp đặt một thứ tự nghiêm ngặt lên tất cả các thao tác trong hệ thống.

Trong giai đoạn đầu tiên của giao thức, proposer gửi một sequence number đến các acceptor (người chấp nhận). Mỗi acceptor chỉ chấp nhận proposal nếu nó chưa thấy một proposal nào có sequence number cao hơn. Các proposer có thể thử lại với một sequence number cao hơn nếu cần. Các proposer phải dùng các sequence number duy nhất (lấy từ các tập rời nhau, hoặc ghép tên máy chủ vào sequence number, chẳng hạn).

Nếu một proposer nhận được sự đồng ý từ đa số các acceptor, nó có thể commit proposal bằng cách gửi một tin nhắn commit kèm theo một giá trị.

Việc định thứ tự nghiêm ngặt các proposal giải quyết mọi vấn đề liên quan đến thứ tự tin nhắn trong hệ thống. Yêu cầu cần một đa số (quorum) để commit có nghĩa là không thể commit hai giá trị khác nhau cho cùng một proposal, bởi vì bất kỳ hai đa số nào cũng sẽ chồng lấn nhau trên ít nhất một node. Các acceptor phải ghi journal vào bộ lưu trữ bền vững mỗi khi chấp nhận một proposal, vì chúng cần tuân thủ những cam kết này sau khi khởi động lại.

Paxos đơn lẻ không thực sự hữu ích: nó chỉ cho phép đồng thuận về một giá trị và một số proposal mỗi lần. Vì chỉ cần một quorum các node đồng ý về một giá trị, nên một node bất kỳ có thể không nắm được toàn bộ tập các giá trị đã được đồng thuận. Hạn chế này đúng với hầu hết các thuật toán distributed consensus.

## Các Mẫu Kiến Trúc Hệ Thống Cho Distributed Consensus (System Architecture Patterns for Distributed Consensus)

Các thuật toán distributed consensus hoạt động ở tầng thấp và mang tính nguyên thủy: chúng chỉ giúp một tập các node đạt được sự đồng thuận về một giá trị, mỗi lần một. Do đó, chúng không phù hợp để ánh xạ trực tiếp vào các tác vụ thiết kế thực tế. Giá trị thực sự của distributed consensus nằm ở việc kết hợp với các thành phần hệ thống ở tầng cao hơn như datastore, kho cấu hình, hàng đợi, khóa và dịch vụ bầu leader, từ đó cung cấp các chức năng hệ thống thực tiễn mà bản thân các thuật toán distributed consensus không thể giải quyết. Việc sử dụng các thành phần tầng cao này giúp giảm bớt độ phức tạp cho các nhà thiết kế hệ thống. Đồng thời, nó cho phép thay thế các thuật toán distributed consensus ở tầng dưới khi cần thiết, nhằm thích ứng với những thay đổi trong môi trường triển khai hoặc các yêu cầu phi chức năng.

Nhiều hệ thống vận hành thành công dựa trên các thuật toán consensus, nhưng thực chất chúng chỉ đóng vai trò là client của một số dịch vụ đã cài đặt sẵn các thuật toán đó, chẳng hạn như ZooKeeper, Consul và etcd. ZooKeeper [\[Hun10\]](https://sre.google/sre-book/bibliography#Hun10) là hệ thống consensus mã nguồn mở đầu tiên được giới công nghiệp chấp nhận rộng rãi nhờ tính dễ sử dụng, ngay cả với những ứng dụng không được thiết kế để dùng distributed consensus. Dịch vụ Chubby lấp đầy một ngách tương tự tại Google. Các tác giả của nó chỉ ra [\[Bur06\]](https://sre.google/sre-book/bibliography#Bur06) rằng việc cung cấp các nguyên thủy consensus dưới dạng một dịch vụ, thay vì dưới dạng các thư viện để kỹ sư nhúng vào ứng dụng, giúp các nhà duy trì ứng dụng không phải lo triển khai hệ thống sao cho tương thích với một dịch vụ consensus khả dụng cao (chạy đúng số lượng bản sao, xử lý thành viên nhóm, xử lý hiệu năng, v.v.).

## Các Máy Trạng Thái Nhân Bản Tin Cậy (Reliable Replicated State Machines)

Một *replicated state machine* (máy trạng thái nhân bản, RSM) là hệ thống thực thi cùng một tập thao tác, theo cùng một thứ tự, trên nhiều tiến trình. RSM đóng vai trò là khối xây dựng nền tảng cho các thành phần và dịch vụ hệ thống phân tán hữu ích, chẳng hạn như lưu trữ dữ liệu hoặc cấu hình, khóa, và bầu leader (sẽ được mô tả chi tiết hơn ở phần sau).

Các thao tác trên một RSM được định thứ tự toàn cầu thông qua một thuật toán consensus. Đây là một khái niệm mạnh mẽ: một số bài báo ([\[Agu10\]](https://sre.google/sre-book/bibliography#Agu10), [\[Kir08\]](https://sre.google/sre-book/bibliography#Kir08), [\[Sch90\]](https://sre.google/sre-book/bibliography#Sch90)) chỉ ra rằng bất kỳ chương trình nào tất định (deterministic) đều có thể được cài đặt như một dịch vụ nhân bản khả dụng cao bằng cách triển khai nó dưới dạng một RSM.

Như [Hình 23-2](#hinh-23-2) cho thấy, các máy trạng thái nhân bản là một hệ thống được cài đặt ở tầng logic phía trên thuật toán consensus. Thuật toán consensus chịu trách nhiệm đạt đồng thuận về thứ tự các thao tác, còn RSM thực thi các thao tác đó theo đúng thứ tự. Vì không phải mọi thành viên của nhóm consensus đều nhất thiết thuộc mỗi quorum consensus, nên các RSM có thể cần đồng bộ hóa trạng thái từ các tiến trình cùng cấp (peer). Theo Kirsch và Amir [\[Kir08\]](https://sre.google/sre-book/bibliography#Kir08), bạn có thể dùng một *giao thức cửa sổ trượt* để đối chiếu trạng thái giữa các tiến trình cùng cấp trong một RSM.

<a id="hinh-23-2"></a>        ![Mối quan hệ giữa các thuật toán consensus và các máy trạng thái nhân bản.](../assets/imgs/fig-23-2.jpg)

Hình 23-2. Mối quan hệ giữa các thuật toán consensus và các máy trạng thái nhân bản

## Các Kho Dữ Liệu Nhân Bản và Kho Cấu Hình Tin Cậy (Reliable Replicated Datastores and Configuration Stores)

Các datastore nhân bản tin cậy là một ứng dụng của các máy trạng thái nhân bản. Chúng sử dụng các thuật toán consensus trên đường dẫn quan trọng (critical path) của công việc. Do đó, hiệu năng, thông lượng (throughput) và khả năng mở rộng đóng vai trò then chốt trong kiểu thiết kế này. Giống như các datastore được xây dựng bằng các công nghệ khác ở phần dưới, các datastore dựa trên consensus có thể cung cấp nhiều mức ngữ nghĩa nhất quán khác nhau cho các thao tác đọc, điều này ảnh hưởng đáng kể đến cách datastore mở rộng. Các đánh đổi này được thảo luận trong [Hiệu Năng Distributed Consensus](#hieu-nang-distributed-consensus).

Các hệ thống khác (không dựa trên distributed consensus) thường đơn giản dựa vào các timestamp để cung cấp các cận về độ tuổi của dữ liệu được trả về. Trong các hệ thống phân tán, timestamp gây ra vấn đề rất lớn vì không thể đảm bảo rằng các đồng hồ được đồng bộ xuyên qua nhiều máy. Spanner [\[Cor12\]](https://sre.google/sre-book/bibliography#Cor12) giải quyết vấn đề này bằng cách mô hình hóa sự không chắc chắn trong trường hợp xấu nhất và làm chậm việc xử lý ở những nơi cần thiết để giải quyết sự không chắc chắn đó.

## Xử Lý Khả Dụng Cao Dùng Việc Bầu Leader (Highly Available Processing Using Leader Election)

Leader election trong các hệ thống phân tán là một bài toán tương đương với distributed consensus. Việc các dịch vụ nhân bản dùng một leader duy nhất để thực hiện một kiểu công việc cụ thể trong hệ thống là rất phổ biến; cơ chế leader đơn là một cách để đảm bảo loại trừ lẫn nhau (mutual exclusion) ở cấp độ thô.

Kiểu thiết kế này phù hợp khi công việc của leader dịch vụ có thể chạy trong một tiến trình hoặc được chia shard. Để xây dựng dịch vụ khả dụng cao, các nhà thiết kế hệ thống có thể viết dịch vụ như một chương trình đơn giản, nhân bản tiến trình và dùng leader election để đảm bảo chỉ có một leader hoạt động tại mỗi thời điểm (như minh họa trong [Hình 23-3](#hinh-23-3)). Thông thường, leader điều phối một số lượng worker trong hệ thống. Mẫu này được áp dụng trong GFS [\[Ghe03\]](https://sre.google/sre-book/bibliography#Ghe03) (nay đã được thay thế bởi Colossus) và kho key-value Bigtable [\[Cha06\]](https://sre.google/sre-book/bibliography#Cha06).

<a id="hinh-23-3"></a>        ![Hệ thống khả dụng cao dùng một dịch vụ nhân bản cho việc bầu master.](../assets/imgs/fig-23-3.jpg)

Hình 23-3. Hệ thống khả dụng cao dùng một dịch vụ nhân bản cho việc bầu master

Trong kiểu thành phần này, thuật toán consensus không nằm trên đường dẫn quan trọng của công việc chính mà hệ thống đang xử lý, khác với datastore nhân bản, nên thông lượng thường không phải là mối quan tâm lớn.

## Các Dịch Vụ Phối Hợp và Khóa Phân Tán (Distributed Coordination and Locking Services)

Trong một phép tính phân tán, *barrier* (rào chắn) là một nguyên thủy ngăn một nhóm tiến trình tiếp tục cho đến khi một điều kiện nào đó được thỏa mãn (ví dụ, cho đến khi tất cả các phần của một giai đoạn trong phép tính được hoàn thành). Việc sử dụng barrier thực chất chia phép tính phân tán thành các giai đoạn logic. Ví dụ, như thể hiện trong [Hình 23-4](#hinh-23-4), barrier có thể được dùng trong việc cài đặt mô hình MapReduce [\[Dea04\]](https://sre.google/sre-book/bibliography#Dea04) để đảm bảo toàn bộ giai đoạn Map hoàn thành trước khi phần Reduce của phép tính tiếp tục.

<a id="hinh-23-4"></a>        ![Các barrier để phối hợp tiến trình trong phép tính MapReduce.](../assets/imgs/fig-23-4.jpg)

Hình 23-4. Các barrier để phối hợp tiến trình trong phép tính MapReduce

Barrier có thể được cài đặt bằng một tiến trình điều phối (coordinator) đơn, nhưng cách làm này tạo ra một điểm thất bại duy nhất (single point of failure) thường không thể chấp nhận được. Barrier cũng có thể được cài đặt như một RSM. Dịch vụ consensus ZooKeeper có thể cài đặt mẫu barrier: xem [\[Hun10\]](https://sre.google/sre-book/bibliography#Hun10) và [\[Zoo14\]](https://sre.google/sre-book/bibliography#Zoo14).

*Lock* là một nguyên thủy phối hợp hữu ích khác có thể được cài đặt như một RSM. Hãy xét một hệ thống phân tán trong đó các tiến trình worker tiêu thụ một cách nguyên tử một số tệp đầu vào và ghi các kết quả. Các distributed lock có thể được dùng để ngăn nhiều worker xử lý cùng một tệp đầu vào. Trên thực tế, việc dùng các lease có thể làm mới kèm timeout, thay vì khóa vô hạn, là điều thiết yếu, vì làm vậy ngăn các khóa bị giữ vô thời hạn bởi những tiến trình đã crash. Distributed locking nằm ngoài phạm vi của chương này, nhưng hãy lưu ý rằng distributed lock là một nguyên thủy hệ thống ở tầng thấp nên được dùng một cách cẩn thận. Hầu hết các ứng dụng nên dùng một hệ thống ở tầng cao hơn cung cấp các giao dịch phân tán.

## Hàng Đợi và Nhắn Tin Phân Tán Tin Cậy (Reliable Distributed Queuing and Messaging)

Hàng đợi (queue) là một cấu trúc dữ liệu phổ biến, thường được dùng như một cách để phân phối các tác vụ giữa một số tiến trình worker.

Các hệ thống dựa trên hàng đợi thường chịu được sự cố và mất mát các node worker khá dễ dàng. Tuy nhiên, hệ thống phải đảm bảo rằng các tác vụ đã được nhận (claimed) được xử lý thành công. Vì mục đích đó, một *hệ thống lease* (được thảo luận trước đó liên quan đến khóa) được khuyến nghị thay vì việc xóa thẳng khỏi hàng đợi. Điểm bất lợi của các hệ thống dựa trên hàng đợi là việc mất hàng đợi khiến toàn bộ hệ thống không thể hoạt động. Việc cài đặt hàng đợi như một RSM có thể làm giảm thiểu rủi ro và làm toàn bộ hệ thống bền bỉ hơn nhiều.

*Atomic broadcast* là một nguyên thủy hệ thống phân tán, trong đó mọi bên tham gia đều nhận tin nhắn một cách tin cậy và theo cùng một thứ tự. Đây là khái niệm hệ thống phân tán cực kỳ mạnh mẽ, rất hữu ích khi thiết kế các hệ thống thực tiễn. Hiện có nhiều cơ sở hạ tầng nhắn tin publish-subscribe để các nhà thiết kế hệ thống lựa chọn, dù không phải tất cả đều cung cấp các bảo đảm nguyên tử. Chandra và Toueg [\[Cha96\]](https://sre.google/sre-book/bibliography#Cha96) đã chứng minh tính tương đương giữa atomic broadcast và consensus.

Mẫu *queuing-as-work-distribution* (hàng đợi như cơ chế phân phối công việc), dùng hàng đợi như một thiết bị cân bằng tải, như được thể hiện trong [Hình 23-5](#hinh-23-5), có thể được coi là nhắn tin điểm-đến-điểm. Các hệ thống nhắn tin thường cũng cài đặt một hàng đợi publish-subscribe, trong đó các tin nhắn có thể được tiêu thụ bởi nhiều client đã đăng ký một kênh hoặc chủ đề. Trong trường hợp một-đến-nhiều này, các tin nhắn trên hàng đợi được lưu trữ dưới dạng một danh sách có thứ tự bền vững. Các hệ thống publish-subscribe có thể được dùng cho nhiều loại ứng dụng đòi hỏi các client đăng ký để nhận thông báo về một kiểu sự kiện nào đó. Các hệ thống publish-subscribe cũng có thể được dùng để cài đặt các cache phân tán nhất quán.

<a id="hinh-23-5"></a>        ![Hệ thống phân phối công việc hướng hàng đợi dùng một thành phần hàng đợi dựa trên consensus tin cậy.](../assets/imgs/fig-23-5.jpg)

Hình 23-5. Hệ thống phân phối công việc hướng hàng đợi dùng một thành phần hàng đợi dựa trên consensus tin cậy

Các hệ thống hàng đợi và nhắn tin thường đòi hỏi thông lượng cao, nhưng không yêu cầu độ trễ (latency) cực thấp (do ít khi trực tiếp tiếp xúc người dùng). Tuy nhiên, trong hệ thống như vừa mô tả, nơi nhiều worker nhận tác vụ từ một hàng đợi, độ trễ rất cao có thể trở thành vấn đề nếu thời gian xử lý mỗi tác vụ tăng đáng kể.

<a id="hieu-nang-distributed-consensus"></a>

## Hiệu Năng Distributed Consensus (Distributed Consensus Performance)

Quan niệm thông thường cho rằng các thuật toán consensus là quá chậm và tốn kém để dùng cho nhiều hệ thống đòi hỏi thông lượng cao *và* độ trễ thấp [\[Bol11\]](https://sre.google/sre-book/bibliography#Bol11). Quan niệm này đơn giản là không đúng — trong khi các bản cài đặt có thể chậm, có một số thủ thuật có thể cải thiện hiệu năng. Các thuật toán distributed consensus nằm ở trung tâm của nhiều hệ thống quan trọng của Google, được mô tả trong [\[Ana13\]](https://sre.google/sre-book/bibliography#Ana13), [\[Bur06\]](https://sre.google/sre-book/bibliography#Bur06), [\[Cor12\]](https://sre.google/sre-book/bibliography#Cor12), và [\[Shu13\]](https://sre.google/sre-book/bibliography#Shu13), và chúng đã chứng tỏ cực kỳ hiệu quả trên thực tế. Quy mô của Google không phải là một lợi thế ở đây; thực tế, quy mô của chúng tôi là một bất lợi, vì nó kéo theo hai thách thức chính: các tập dữ liệu của chúng tôi thường lớn và các hệ thống của chúng tôi chạy trải rộng một khoảng cách địa lý lớn. Các tập dữ liệu lớn nhân lên bởi một vài bản sao đại diện cho chi phí tính toán đáng kể, và các khoảng cách địa lý lớn hơn làm tăng độ trễ giữa các bản sao, từ đó làm giảm hiệu năng.

Không tồn tại một thuật toán distributed consensus và nhân bản máy trạng thái nào là “tốt nhất” duy nhất về mặt hiệu năng, bởi hiệu năng phụ thuộc vào một số yếu tố liên quan đến khối lượng công việc (workload), các mục tiêu hiệu năng của hệ thống, và cách hệ thống được triển khai.<sup>[2](#fn2)</sup> Mặc dù một số phần sau trình bày các nghiên cứu nhằm tăng cường hiểu biết về những gì có thể đạt được với distributed consensus, nhưng nhiều hệ thống được mô tả hiện đã có sẵn và đang được sử dụng.

*Workload* có thể biến đổi theo nhiều cách, và việc nắm rõ các dạng biến đổi này là then chốt để thảo luận về hiệu năng. Trong một hệ thống consensus, workload có thể biến đổi theo các khía cạnh sau:

- Thông lượng: số lượng các proposal được thực hiện mỗi đơn vị thời gian tại tải đỉnh (peak load)
- Kiểu các request: tỷ lệ các thao tác làm thay đổi trạng thái
- Ngữ nghĩa nhất quán được yêu cầu cho các thao tác đọc
- Kích thước các request, nếu kích thước payload dữ liệu có thể biến đổi

Các chiến lược triển khai cũng khác nhau. Ví dụ:

- Việc triển khai là mạng diện rộng (wide area) hay mạng cục bộ (local area)?
- Các kiểu quorum nào được dùng, và đa số các tiến trình nằm ở đâu?
- Hệ thống có dùng sharding, pipelining, và batching không?

Nhiều hệ thống consensus sử dụng một tiến trình leader đặc biệt, bắt buộc mọi request đều phải gửi đến node này. Như [Hình 23-6](#hinh-23-6) cho thấy, hiệu năng mà các client ở những vị trí địa lý khác nhau cảm nhận được có thể chênh lệch rất lớn, đơn giản vì các node ở xa có thời gian vòng đi-về (round-trip) đến tiến trình leader dài hơn.

<a id="hinh-23-6"></a>        ![Tác động của khoảng cách tới một tiến trình server lên độ trễ cảm nhận tại client.](../assets/imgs/fig-23-6.jpg)

Hình 23-6. Tác động của khoảng cách tới một tiến trình server lên độ trễ cảm nhận tại client

<a id="multi-paxos-luon-tin-nhan-chi-tiet"></a>

## Multi-Paxos: Luồng Tin Nhắn Chi Tiết (Multi-Paxos: Detailed Message Flow)

Giao thức Multi-Paxos sử dụng một *strong leader process* (tiến trình leader mạnh): trừ khi chưa bầu được leader hoặc xảy ra sự cố, nó chỉ cần một vòng đi-về duy nhất từ proposer đến một quorum các acceptor để đạt consensus. Việc dùng một tiến trình leader mạnh là tối ưu về số tin nhắn phải truyền và là đặc trưng của nhiều giao thức consensus.

[Hình 23-7](#hinh-23-7) mô tả trạng thái ban đầu, trong đó một proposer mới đang thực thi giai đoạn đầu tiên `Prepare`/`Promise` của giao thức. Giai đoạn này thiết lập một view đánh số mới, hay một leader term (nhiệm kỳ leader). Ở các lần thực thi sau, nếu view không đổi, proposer đã thiết lập view có thể bỏ qua giai đoạn đầu tiên và chỉ cần gửi các tin nhắn `Accept`. Consensus được đạt khi nhận đủ một quorum các phản hồi (bao gồm cả chính proposer).

<a id="hinh-23-7"></a>        ![Luồng tin nhắn Multi-Paxos cơ bản.](../assets/imgs/fig-23-7.jpg)

Hình 23-7. Luồng tin nhắn Multi-Paxos cơ bản

Một tiến trình khác trong nhóm có thể đảm nhận vai trò proposer để đề xuất tin nhắn bất kỳ lúc nào, nhưng việc thay đổi proposer sẽ phát sinh chi phí hiệu năng. Nó đòi hỏi một vòng đi-về thêm để thực thi Giai đoạn 1 của giao thức; quan trọng hơn, nó có thể dẫn đến tình huống *dueling proposers* (các proposer đấu nhau), trong đó các proposal liên tục ngắt nhau và không proposal nào được chấp nhận, như minh họa trong [Hình 23-8](#hinh-23-8). Vì đây là một dạng livelock, tình trạng này có thể kéo dài vô thời hạn.

<a id="hinh-23-8"></a>        ![Dueling proposers trong Multi-Paxos.](../assets/imgs/fig-23-8.jpg)

Hình 23-8. Dueling proposers trong Multi-Paxos

Mọi hệ thống consensus thực tế đều xử lý tình trạng va chạm này, chủ yếu bằng cách bầu một tiến trình proposer đảm nhận toàn bộ các proposal trong hệ thống, hoặc dùng một proposer luân phiên (rotating) để phân bổ các khe (slots) nhất định cho proposal của từng tiến trình.

Với các hệ thống sử dụng một tiến trình leader, quá trình bầu leader cần được thiết kế tỉ mỉ nhằm cân bằng giữa nguy cơ hệ thống không khả dụng khi thiếu leader và rủi ro dueling proposers. Việc cấu hình đúng các timeout và chiến lược backoff là yếu tố then chốt. Nếu nhiều tiến trình cùng phát hiện ra không có leader và đồng loạt cố gắng trở thành leader, sẽ không có tiến trình nào thành công (lại một lần nữa, dueling proposers). Cách tiếp cận hiệu quả nhất là đưa vào yếu tố ngẫu nhiên. Raft [\[Ong14\]](https://sre.google/sre-book/bibliography#Ong14), chẳng hạn, áp dụng một phương pháp được suy nghĩ kỹ lưỡng cho quá trình bầu leader này.

## Mở Rộng Các Workload Nặng Đọc (Scaling Read-Heavy Workloads)

Việc mở rộng workload đọc thường là then chốt vì nhiều workload nặng đọc. Các datastore nhân bản có lợi thế là dữ liệu khả dụng ở nhiều nơi, có nghĩa là nếu không yêu cầu tính nhất quán mạnh (strong consistency) cho tất cả các thao tác đọc, thì dữ liệu có thể được đọc từ *bất kỳ* bản sao nào. Kỹ thuật đọc từ các bản sao này hoạt động tốt cho một số ứng dụng, chẳng hạn như hệ thống Photon của Google [\[Ana13\]](https://sre.google/sre-book/bibliography#Ana13), dùng distributed consensus để phối hợp công việc của nhiều pipeline. Photon dùng một thao tác compare-and-set nguyên tử để sửa đổi trạng thái (lấy cảm hứng từ các bộ đăng ký nguyên tử), thao tác này phải nhất quán một cách tuyệt đối; nhưng các thao tác đọc có thể được phục vụ từ bất kỳ bản sao nào, vì dữ liệu lỗi thời dẫn đến việc thực hiện thêm công việc chứ không phải kết quả sai [\[Gup15\]](https://sre.google/sre-book/bibliography#Gup15). Sự đánh đổi này là đáng giá.

Để đảm bảo dữ liệu đọc được là mới nhất và nhất quán với mọi thay đổi đã diễn ra trước thao tác đọc, cần thực hiện một trong các điều sau:

- Thực hiện một thao tác consensus chỉ-đọc (read-only).
- Đọc dữ liệu từ một bản sao được đảm bảo là mới nhất. Trong hệ thống sử dụng một tiến trình leader ổn định (như nhiều bản cài đặt distributed consensus), leader có thể cung cấp sự đảm bảo này.
- Dùng quorum lease, trong đó một số bản sao được cấp lease trên toàn bộ hoặc một phần dữ liệu trong hệ thống, cho phép các thao tác đọc cục bộ nhất quán mạnh với chi phí là một chút hiệu năng ghi. Kỹ thuật này được thảo luận chi tiết trong phần tiếp theo.

## Quorum Lease (Quorum Leases)

Quorum lease [\[Mor14\]](https://sre.google/sre-book/bibliography#Mor14) là một tối ưu hóa hiệu năng cho distributed consensus, được phát triển gần đây nhằm giảm độ trễ và tăng thông lượng cho các thao tác đọc. Như đã đề cập, trong Paxos cổ điển và hầu hết các giao thức distributed consensus khác, để thực hiện một thao tác đọc nhất quán mạnh (tức là thao tác được đảm bảo có cái nhìn mới nhất về trạng thái), hệ thống cần hoặc một thao tác distributed consensus đọc từ một quorum các bản sao, hoặc một bản sao leader ổn định được đảm bảo đã thấy tất cả các thao tác thay đổi trạng thái gần đây. Vì trong nhiều hệ thống, số lượng thao tác đọc vượt xa số thao tác ghi, nên sự phụ thuộc vào thao tác phân tán hoặc một bản sao đơn làm hại độ trễ và thông lượng hệ thống.

Kỹ thuật quorum leasing đơn giản cấp một read lease (ủy quyền đọc) trên một tập con nào đó của trạng thái datastore nhân bản cho một quorum các bản sao. Lease này có hiệu lực trong một khoảng thời gian cụ thể (thường ngắn). Mọi thao tác làm thay đổi trạng thái dữ liệu đó đều phải được tất cả các bản sao trong quorum đọc xác nhận. Nếu bất kỳ bản sao nào trong số này không khả dụng, dữ liệu sẽ không thể sửa đổi cho đến khi lease hết hạn.

Quorum lease đặc biệt hữu ích cho các workload nặng đọc, khi các thao tác đọc trên những tập con dữ liệu cụ thể được tập trung tại một khu vực địa lý duy nhất.

## Hiệu Năng Distributed Consensus và Độ Trễ Mạng (Distributed Consensus Performance and Network Latency)

Khi commit các thay đổi trạng thái, các hệ thống consensus phải đối mặt với hai ràng buộc vật lý lớn về hiệu năng: thời gian vòng đi-về mạng và thời gian ghi dữ liệu vào bộ lưu trữ bền vững (sẽ được xem xét sau).

Thời gian vòng đi-về mạng biến động rất lớn, phụ thuộc vào cả khoảng cách vật lý giữa nguồn và đích lẫn mức độ tắc nghẽn trên mạng. Trong một datacenter đơn, thời gian vòng đi-về giữa các máy nên ở mức cỡ một mili-giây. Một RTT (thời gian vòng đi-về) điển hình trong nước Mỹ là 45 mili-giây, và từ New York đến London là 70 mili-giây.

Hiệu năng của hệ thống consensus trên mạng diện cục bộ có thể sánh ngang với hệ thống nhân bản leader-follower bất đồng bộ [\[Bol11\]](https://sre.google/sre-book/bibliography#Bol11) — kiểu nhân bản mà nhiều database truyền thống sử dụng. Tuy nhiên, để khai thác phần lớn lợi ích của các hệ thống distributed consensus, các bản sao cần được đặt "xa nhau" nhằm rơi vào các domain lỗi khác nhau.

Nhiều hệ thống consensus dùng TCP/IP làm giao thức truyền thông của chúng. TCP/IP có định hướng kết nối và cung cấp một số bảo đảm độ tin cậy mạnh về thứ tự FIFO của các tin nhắn. Tuy nhiên, việc thiết lập một kết nối TCP/IP mới đòi hỏi một vòng đi-về mạng để thực hiện bắt tay ba bước trước khi bất kỳ dữ liệu nào có thể được gửi hoặc nhận. TCP/IP slow start ban đầu giới hạn băng thông của kết nối cho đến khi các giới hạn của nó được thiết lập. Các kích thước cửa sổ TCP/IP ban đầu dao động từ 4 đến 15 KB.

TCP/IP slow start có lẽ không phải là vấn đề cho các tiến trình tạo thành một nhóm consensus: chúng sẽ thiết lập các kết nối với nhau và giữ các kết nối này mở để dùng lại vì chúng liên lạc thường xuyên. Tuy nhiên, đối với các hệ thống có số lượng client rất cao, việc tất cả các client giữ một kết nối bền vững đến các cluster consensus mở có thể không thực tiễn, vì các kết nối TCP/IP mở cũng tiêu tốn một số tài nguyên, ví dụ như các file descriptor, bên cạnh việc tạo ra lưu lượng keepalive. Chi phí này có thể là một vấn đề quan trọng cho các ứng dụng dùng các datastore dựa trên consensus được chia shard rất cao chứa hàng nghìn bản sao và một số lượng client còn lớn hơn. Một giải pháp là dùng một pool các proxy theo khu vực, như được thể hiện trong [Hình 23-9](#hinh-23-9), để giữ các kết nối TCP/IP bền vững đến nhóm consensus, tránh chi phí thiết lập trên các khoảng cách dài. Các proxy cũng có thể là một cách tốt để đóng gói các chiến lược sharding và cân bằng tải, cũng như việc khám phá các thành viên và leader của cluster.

<a id="hinh-23-9"></a>        ![Dùng proxy để giảm nhu cầu các client mở kết nối TCP/IP xuyên qua các khu vực.](../assets/imgs/fig-23-9.jpg)

Hình 23-9. Dùng proxy để giảm nhu cầu các client mở kết nối TCP/IP xuyên qua các khu vực

<a id="suy-luan-ve-hieu-nang-fast-paxos"></a>

## Suy Luận Về Hiệu Năng: Fast Paxos (Reasoning About Performance: Fast Paxos)

Fast Paxos [\[Lam06\]](https://sre.google/sre-book/bibliography#Lam06) là một phiên bản của thuật toán Paxos, được thiết kế nhằm cải thiện hiệu năng trên các mạng diện rộng. Khi sử dụng Fast Paxos, mỗi client có thể gửi các tin nhắn `Propose` trực tiếp đến từng thành viên của nhóm acceptor, thay vì phải thông qua một leader như trong Classic Paxos hoặc Multi-Paxos. Ý tưởng là thay thế một phép gửi tin nhắn song song duy nhất từ client đến tất cả các acceptor trong Fast Paxos bằng hai phép gửi tin nhắn trong Classic Paxos:

- Một tin nhắn từ client đến một proposer đơn
- Một phép gửi tin nhắn song song từ proposer đến các bản sao khác

Trực quan, dường như Fast Paxos luôn phải nhanh hơn Classic Paxos. Tuy nhiên, điều đó không đúng: nếu client trong hệ thống Fast Paxos có một RTT cao đến các acceptor, và các acceptor có các kết nối nhanh với nhau, thì chúng ta đã thay thế *N* tin nhắn song song xuyên qua các liên kết mạng chậm hơn (trong Fast Paxos) bằng một tin nhắn xuyên qua liên kết chậm hơn cộng với *N* tin nhắn song song xuyên qua các liên kết nhanh hơn (Classic Paxos). Do hiệu ứng đuôi độ trễ, phần lớn thời gian, một vòng đi-về đơn xuyên qua một liên kết chậm với một phân bố độ trễ sẽ nhanh hơn một quorum (như được thể hiện trong [\[Jun07\]](https://sre.google/sre-book/bibliography#Jun07)), và vì vậy, Fast Paxos chậm hơn Classic Paxos trong trường hợp này.

Nhiều hệ thống gom nhóm (batch) nhiều thao tác vào một giao dịch duy nhất tại acceptor để tăng thông lượng. Việc để các client đóng vai trò proposer cũng khiến việc gom nhóm các proposal trở nên khó khăn hơn rất nhiều. Lý do là các proposal đến các acceptor một cách độc lập, nên bạn không thể gom nhóm chúng một cách nhất quán.

## Các Leader Ổn Định (Stable Leaders)

Trước đó, chúng ta đã xem xét cách Multi-Paxos bầu ra một leader ổn định nhằm cải thiện hiệu năng. Zab [\[Jun11\]](https://sre.google/sre-book/bibliography#Jun11) và Raft [\[Ong14\]](https://sre.google/sre-book/bibliography#Ong14) cũng là những ví dụ về các giao thức bầu leader ổn định vì lý do hiệu năng. Cách tiếp cận này cho phép thực hiện các tối ưu hóa đọc, do leader nắm giữ trạng thái mới nhất, nhưng cũng tồn tại một số vấn đề:

- Tất cả các thao tác thay đổi trạng thái phải được gửi qua leader, một yêu cầu thêm độ trễ mạng cho các client không nằm gần leader.
- Băng thông mạng phát ra của tiến trình leader là một nút thắt (bottleneck) của hệ thống [\[Mao08\]](https://sre.google/sre-book/bibliography#Mao08). Nguyên nhân là tin nhắn `Accept` từ leader chứa toàn bộ dữ liệu liên quan đến bất kỳ proposal nào, trong khi các tin nhắn khác chỉ mang xác nhận cho một giao dịch đánh số, không kèm payload dữ liệu.
- Nếu tình cờ leader nằm trên một máy có vấn đề về hiệu năng, thì thông lượng của toàn bộ hệ thống sẽ bị giảm.

Gần như tất cả các hệ thống distributed consensus được thiết kế với hiệu năng làm trung tâm đều dùng mẫu leader ổn định đơn, hoặc một hệ thống lãnh đạo luân phiên, trong đó mỗi thuật toán distributed consensus đánh số được gán trước cho một bản sao (thường bằng một phép lấy dư đơn giản của ID giao dịch). Các thuật toán dùng cách tiếp cận này bao gồm Mencius [\[Mao08\]](https://sre.google/sre-book/bibliography#Mao08) và Egalitarian Paxos [\[Mor12a\]](https://sre.google/sre-book/bibliography#Mor12a).

Trên mạng diện rộng, khi các client phân bố theo địa lý và các bản sao của nhóm consensus đặt gần client, việc bầu leader như vậy giúp giảm độ trễ cảm nhận. Lý do là RTT từ client đến bản sao gần nhất, tính trung bình, sẽ nhỏ hơn so với RTT đến một leader bất kỳ.

## Gom Nhóm (Batching)

Việc gom nhóm (batching), như đã mô tả trong [Suy Luận Về Hiệu Năng: Fast Paxos](#suy-luan-ve-hieu-nang-fast-paxos), giúp tăng thông lượng hệ thống, nhưng các bản sao vẫn phải nằm chờ phản hồi cho những tin nhắn đã gửi. Để khắc phục tình trạng kém hiệu năng này, ta có thể dùng *pipelining*, cho phép nhiều proposal đang trong lúc bay (in-flight) cùng lúc. Cách tối ưu hóa này rất giống với trường hợp TCP/IP, nơi giao thức cố gắng "giữ ống đầy" bằng cách tiếp cận cửa sổ trượt. Trên thực tế, pipelining thường được dùng kết hợp với gom nhóm.

Các nhóm request trong đường ống vẫn được định thứ tự toàn cầu bằng view number và transaction number, nên phương pháp này không vi phạm các tính chất định thứ tự toàn cầu cần thiết để chạy một máy trạng thái nhân bản. Phương pháp tối ưu hóa này được thảo luận trong [\[Bol11\]](https://sre.google/sre-book/bibliography#Bol11) và [\[San11\]](https://sre.google/sre-book/bibliography#San11).

## Truy Cập Ổ Đĩa (Disk Access)

Việc ghi nhật ký vào bộ lưu trữ bền vững là bắt buộc để một node, sau khi crash và quay lại cluster, có thể tuân thủ các cam kết trước đó liên quan đến các giao dịch consensus đang diễn ra. Trong giao thức Paxos, ví dụ, các acceptor không thể chấp nhận một proposal nếu chúng đã chấp nhận một proposal có sequence number cao hơn. Nếu chi tiết của các proposal đã đồng ý và đã commit không được ghi nhật ký vào bộ lưu trữ bền vững, một acceptor có thể vi phạm giao thức khi crash và được khởi động lại, dẫn đến trạng thái không nhất quán.

Thời gian ghi một mục vào log trên đĩa dao động rất lớn tùy phần cứng hoặc môi trường ảo hóa, nhưng thường nằm trong khoảng từ một đến vài mili-giây.

Luồng tin nhắn cho Multi-Paxos đã được thảo luận trong [Multi-Paxos: Luồng Tin Nhắn Chi Tiết](#multi-paxos-luon-tin-nhan-chi-tiet), nhưng phần đó chưa chỉ rõ vị trí giao thức ghi các thay đổi trạng thái ra đĩa. Bất cứ khi nào một tiến trình thực hiện cam kết mà nó phải tuân thủ, một phép ghi đĩa phải diễn ra. Trong giai đoạn thứ hai quan trọng về hiệu năng của Multi-Paxos, các điểm này xảy ra trước khi acceptor gửi tin nhắn `Accepted` để phản hồi proposal, và trước khi proposer gửi tin nhắn `Accept`, vì tin nhắn `Accept` này cũng là một tin nhắn `Accepted` ngầm (implicit) [\[Lam98\]](https://sre.google/sre-book/bibliography#Lam98).

Điều này có nghĩa là độ trễ cho một thao tác consensus đơn bao gồm những điều sau:

- Một phép ghi đĩa trên proposer
- Các tin nhắn song song đến các acceptor
- Các phép ghi đĩa song song tại các acceptor
- Các tin nhắn trả về

Có một biến thể của [giao thức Multi-Paxos](https://sre.google/sre-book/distributed-periodic-scheduling/) hữu ích cho các trường hợp thời gian ghi đĩa là yếu tố chi phối. Biến thể này không coi tin nhắn `Accept` của proposer là một tin nhắn `Accepted` ngầm. Thay vào đó, proposer ghi ra đĩa song song với các tiến trình khác và gửi một tin nhắn `Accept` tường minh. Khi đó, độ trễ tỷ lệ với thời gian cần để gửi hai tin nhắn và để một quorum các tiến trình thực thi phép ghi đồng bộ ra đĩa một cách song song.

Nếu độ trễ ghi ngẫu nhiên xuống đĩa chỉ cỡ 10 mili-giây, tốc độ các thao tác consensus sẽ bị giới hạn ở khoảng 100 thao tác mỗi giây. Những con số thời gian này giả định rằng thời gian vòng đi-về mạng là không đáng kể và proposer thực hiện việc ghi nhật ký của nó song song với các acceptor.

Như đã thấy, các thuật toán distributed consensus thường đóng vai trò nền tảng để xây dựng máy trạng thái nhân bản. RSM cũng cần lưu log giao dịch để phục hồi, với cùng lý do như bất kỳ datastore nào. Log của thuật toán consensus và log giao dịch của RSM có thể gộp thành một log duy nhất. Việc này giúp tránh phải liên tục chuyển đổi giữa hai vị trí vật lý khác nhau trên đĩa [\[Bol11\]](https://sre.google/sre-book/bibliography#Bol11), từ đó giảm thời gian seek. Nhờ vậy, ổ đĩa xử lý được nhiều thao tác hơn mỗi giây và toàn bộ hệ thống thực hiện được nhiều giao dịch hơn.

Trong một datastore, các ổ đĩa không chỉ dùng để duy trì log mà còn lưu trữ trạng thái hệ thống. Các phép ghi log phải được đẩy (flush) trực tiếp ra đĩa, trong khi các phép ghi cho thay đổi trạng thái có thể được viết vào cache bộ nhớ và đẩy ra đĩa sau, với thứ tự được sắp xếp lại nhằm tối ưu lịch trình [\[Bol11\]](https://sre.google/sre-book/bibliography#Bol11).

Một tối ưu hóa khả dĩ khác là gom nhóm nhiều thao tác của client lại thành một thao tác duy nhất tại proposer ([\[Ana13\]](https://sre.google/sre-book/bibliography#Ana13), [\[Bol11\]](https://sre.google/sre-book/bibliography#Bol11), [\[Cha07\]](https://sre.google/sre-book/bibliography#Cha07), [\[Jun11\]](https://sre.google/sre-book/bibliography#Jun11), [\[Mao08\]](https://sre.google/sre-book/bibliography#Mao08), [\[Mor12a\]](https://sre.google/sre-book/bibliography#Mor12a)). Cách làm này giúp phân bổ chi phí cố định cho việc ghi nhật ký đĩa và độ trễ mạng trên một số lượng thao tác lớn hơn, từ đó tăng thông lượng.

## Triển Khai Các Hệ Thống Dựa Trên Distributed Consensus (Deploying Distributed Consensus-Based Systems)

Khi triển khai hệ thống dựa trên consensus, các nhà thiết kế phải đưa ra những quyết định quan trọng nhất về số lượng replica cần triển khai và vị trí đặt chúng.

## Số Lượng Bản Sao (Number of Replicas)

Nói chung, các hệ thống dựa trên consensus vận hành bằng *majority quorums* (quorum đa số), tức là một nhóm 2f + 1 bản sao có thể chịu được f lỗi (nếu cần Byzantine fault tolerance, trong đó hệ thống có khả năng chống lại các bản sao trả về kết quả sai, thì 3f + 1 bản sao có thể chịu được f lỗi [\[Cas99\]](https://sre.google/sre-book/bibliography#Cas99)). Đối với các lỗi phi-Byzantine, số lượng bản sao tối thiểu có thể triển khai là ba — nếu chỉ triển khai hai, thì không có khả năng chịu lỗi của bất kỳ tiến trình nào. Ba bản sao có thể chịu được một lỗi. Phần lớn thời gian ngừng của hệ thống là kết quả của việc bảo trì có kế hoạch [\[Ken12\]](https://sre.google/sre-book/bibliography#Ken12): ba bản sao cho phép hệ thống hoạt động bình thường khi một bản sao down để bảo trì (giả định rằng hai bản sao còn lại có thể xử lý tải hệ thống ở một hiệu năng chấp nhận được).

Nếu sự cố ngoài kế hoạch xảy ra trong cửa sổ bảo trì, hệ thống consensus sẽ không khả dụng. Vì sự cố này thường không thể chấp nhận được, nên cần chạy năm bản sao để hệ thống có thể vận hành với tối đa hai lỗi. Khi bốn trên năm bản sao còn hoạt động, không nhất thiết phải can thiệp; nhưng nếu chỉ còn ba, nên bổ sung thêm một hoặc hai bản sao.

Nếu một hệ thống consensus mất quá nhiều bản sao đến mức không thể tạo thành quorum, thì về mặt lý thuyết, hệ thống đó rơi vào trạng thái không thể khôi phục, vì log bền vững của ít nhất một bản sao đã mất không còn truy cập được. Khi không còn quorum, có khả năng một quyết định chỉ tồn tại trên các bản sao bị mất đã được đưa ra. Quản trị viên có thể buộc thay đổi thành viên nhóm và thêm các bản sao mới đồng bộ từ một bản sao hiện có để hệ thống tiếp tục hoạt động, nhưng nguy cơ mất dữ liệu vẫn luôn hiện hữu — đây là tình huống cần tránh nếu có thể.

Trong một thảm họa, các quản trị viên phải quyết định xem có nên thực hiện cấu hình lại cưỡng bức hay không, hoặc chờ một khoảng thời gian để trạng thái hệ thống của các máy trở nên khả dụng. Khi đưa ra những quyết định như vậy, việc xử lý log của hệ thống (bên cạnh giám sát) trở nên quan trọng. Các bài báo lý thuyết thường chỉ ra rằng consensus có thể được dùng để xây dựng một log nhân bản, nhưng lại không thảo luận cách xử lý các bản sao có thể bị lỗi và phục hồi (và do đó bỏ lỡ một chuỗi các quyết định consensus) hoặc bị chậm do sự chậm chạp. Để duy trì sự bền bỉ của hệ thống, điều quan trọng là các bản sao này phải bắt kịp.

*Replicated log* không phải lúc nào cũng được coi là yếu tố trung tâm trong lý thuyết distributed consensus, nhưng lại đóng vai trò then chốt trong các hệ thống sản xuất. Raft đề xuất cách quản lý tính nhất quán của các log nhân bản [\[Ong14\]](https://sre.google/sre-book/bibliography#Ong14) bằng việc xác định rõ cách lấp đầy các khoảng trống trong log của từng bản sao. Trong một hệ thống Raft gồm năm instance, nếu chỉ còn lại leader sau khi mất tất cả các thành viên khác, leader vẫn đảm bảo nắm đầy đủ thông tin về mọi quyết định đã commit. Ngược lại, nếu đa số thành viên bị mất bao gồm cả leader, ta không thể đưa ra các đảm bảo mạnh về mức độ cập nhật của các bản sao còn lại.

Hiệu năng và số lượng bản sao không thuộc quorum có mối liên hệ nhất định: một thiểu số bản sao chậm có thể bị tụt lại, nhờ đó quorum gồm các bản sao hoạt động tốt sẽ chạy nhanh hơn (miễn là leader hoạt động bình thường). Nếu hiệu năng giữa các bản sao chênh lệch đáng kể, mỗi sự cố lỗi đều có thể kéo giảm hiệu năng toàn hệ thống, bởi các ngoại lệ chậm vẫn phải tham gia tạo thành quorum. Hệ thống càng chịu được nhiều sự lỗi hoặc nhiều bản sao bị tụt lại thì hiệu năng tổng thể càng có xu hướng tốt hơn.

Khi quản lý các bản sao, cần lưu ý đến vấn đề chi phí vì mỗi bản sao đều tiêu tốn tài nguyên tính toán đắt đỏ. Nếu hệ thống đang xét là một cluster đơn các tiến trình, chi phí chạy các bản sao có lẽ không phải là yếu tố lớn. Tuy nhiên, đối với các hệ thống như Photon [\[Ana13\]](https://sre.google/sre-book/bibliography#Ana13) sử dụng cấu hình chia shard, trong đó mỗi shard là một nhóm đầy đủ các tiến trình chạy thuật toán consensus, chi phí của các bản sao có thể trở thành yếu tố nghiêm trọng. Khi số lượng shard tăng lên, chi phí cho mỗi bản sao bổ sung cũng tăng, bởi hệ thống phải thêm vào một số lượng tiến trình bằng với số lượng shard.

Vì vậy, quyết định về số lượng bản sao cho bất kỳ hệ thống nào là một sự đánh đổi giữa các yếu tố sau:

- Nhu cầu về độ tin cậy
- Tần suất của việc bảo trì có kế hoạch ảnh hưởng đến hệ thống
- Rủi ro
- Hiệu năng
- Chi phí

Phép tính này sẽ khác nhau tùy hệ thống: mỗi hệ thống đặt service level objective (SLO) về khả dụng khác nhau; một số tổ chức thực hiện bảo trì thường xuyên hơn; và các tổ chức sử dụng phần cứng có chi phí, chất lượng, độ tin cậy khác nhau.

## Vị Trí Của Các Bản Sao (Location of Replicas)

Việc quyết định triển khai các tiến trình tạo thành một cluster consensus dựa trên hai yếu tố: sự đánh đổi giữa các domain lỗi mà hệ thống cần xử lý và yêu cầu về độ trễ. Nhiều vấn đề phức tạp cùng tác động đến quyết định đặt các bản sao ở đâu.

Một *failure domain* (domain lỗi) là tập các thành phần của hệ thống có thể bị không khả dụng do một sự cố đơn lẻ. Các ví dụ về domain lỗi bao gồm:

- Một máy vật lý
- Một rack trong datacenter được cấp điện bởi một nguồn cung cấp điện đơn
- Vài rack trong datacenter được cấp bởi một thành phần thiết bị mạng duy nhất
- Một datacenter có thể bị làm không khả dụng bởi một đường cắt cáp quang
- Một tập các datacenter trong một khu vực địa lý đơn có thể tất cả đều bị ảnh hưởng bởi một thảm họa tự nhiên đơn như một cơn bão

Nói chung, khi khoảng cách giữa các bản sao tăng lên, thời gian vòng đi-về giữa chúng cũng tăng, kéo theo kích thước sự lỗi mà hệ thống có thể chịu đựng cũng lớn hơn. Đối với phần lớn các hệ thống consensus, việc tăng thời gian vòng đi-về giữa các bản sao cũng sẽ làm tăng độ trễ của các thao tác.

Mức độ quan trọng của độ trễ, cũng như khả năng sống sót khi một domain cụ thể gặp sự cố, phụ thuộc rất nhiều vào hệ thống. Một số kiến trúc hệ thống consensus không đòi hỏi thông lượng đặc biệt cao hoặc độ trễ thấp: ví dụ, một hệ thống consensus chỉ để cung cấp dịch vụ thành viên nhóm và bầu leader cho một dịch vụ khả dụng cao có lẽ không bị tải nặng, và nếu thời gian giao dịch consensus chỉ chiếm một phần nhỏ thời gian lease leader, thì hiệu năng của nó không phải là then chốt. Các hệ thống hướng batch cũng ít bị ảnh hưởng bởi độ trễ hơn: kích thước các batch thao tác có thể được tăng lên để tăng thông lượng.

Không phải lúc nào cũng nên liên tục mở rộng domain lỗi mà hệ thống có thể chịu đựng khi bị mất. Ví dụ, nếu tất cả client của một hệ thống consensus đều nằm trong cùng một domain lỗi (chẳng hạn khu vực New York), và việc triển khai hệ thống distributed consensus trên phạm vi địa lý rộng hơn sẽ giúp nó tiếp tục hoạt động khi domain lỗi đó gặp sự cố (chẳng hạn Bão Sandy), thì liệu điều này có đáng không? Có lẽ là không, vì các client cũng sẽ down, khiến hệ thống không nhận được lưu lượng nào. Chi phí phát sinh về độ trễ, thông lượng và tài nguyên tính toán sẽ không mang lại lợi ích gì.

Khi quyết định đặt các bản sao ở đâu, bạn cần tính đến việc phục hồi thảm họa (disaster recovery). Trong một hệ thống lưu trữ dữ liệu quan trọng, các bản sao consensus về cơ bản cũng là các bản sao online của dữ liệu hệ thống. Tuy nhiên, khi dữ liệu quan trọng bị đe dọa, việc sao lưu các snapshot định kỳ ở nơi khác là rất quan trọng, ngay cả trong trường hợp các hệ thống dựa trên consensus vững chắc được triển khai ở một vài domain lỗi đa dạng. Có hai domain lỗi mà bạn không bao giờ có thể thoát khỏi: chính phần mềm, và sai sót của con người từ phía các quản trị viên của hệ thống. Các bug trong phần mềm có thể xuất hiện trong những hoàn cảnh bất thường và gây mất dữ liệu, trong khi cấu hình sai hệ thống có thể có các tác động tương tự. Các vận hành viên con người cũng có thể sai sót, hoặc thực hiện phá hoại gây mất dữ liệu.

Khi quyết định vị trí đặt các bản sao, cần lưu ý rằng thước đo hiệu năng quan trọng nhất là trải nghiệm của client: lý tưởng nhất, thời gian vòng đi-về mạng từ các client đến các bản sao của hệ thống consensus nên được tối thiểu hóa. Trên mạng diện rộng, các giao thức không-leader như Mencius hoặc Egalitarian Paxos có thể mang lại lợi thế về hiệu năng, đặc biệt nếu các ràng buộc nhất quán của ứng dụng cho phép thực hiện thao tác chỉ-đọc trên bất kỳ bản sao nào mà không cần chạy thao tác consensus.

## Dung Lượng và Cân Bằng Tải (Capacity and Load Balancing)

Khi thiết kế một bản triển khai, bạn phải đảm bảo có đủ dung lượng (capacity) để xử lý tải. Với *sharded deployments* (các bản triển khai chia shard), bạn có thể điều chỉnh dung lượng bằng cách thay đổi số lượng shard. Tuy nhiên, đối với các hệ thống cho phép đọc từ các thành viên của nhóm consensus không phải là leader, bạn có thể tăng dung lượng đọc bằng cách thêm nhiều bản sao hơn. Việc này có một chi phí: trong thuật toán dùng một leader mạnh, thêm bản sao sẽ làm tăng tải lên tiến trình leader, còn trong giao thức peer-to-peer, nó làm tăng tải lên tất cả các tiến trình. Dù vậy, nếu hệ thống đã đủ dung lượng cho các thao tác ghi nhưng một workload nặng đọc đang gây quá tải, việc thêm bản sao có thể là cách tiếp cận tốt nhất.

Cần lưu ý rằng việc thêm một bản sao vào hệ thống quorum đa số có thể làm giảm nhẹ khả dụng của hệ thống (như thể hiện trong [Hình 23-10](#hinh-23-10)). Một bản triển khai điển hình cho ZooKeeper hoặc Chubby dùng năm bản sao, do đó quorum đa số đòi hỏi ba bản sao. Hệ thống vẫn tiến triển được nếu hai bản sao, tức 40%, không khả dụng. Với sáu bản sao, quorum đòi hỏi bốn bản sao: chỉ 33% các bản sao được phép không khả dụng nếu hệ thống muốn vẫn hoạt động.

Do đó, các cân nhắc về domain lỗi càng trở nên quan trọng hơn khi thêm bản sao thứ sáu: nếu một tổ chức có năm datacenter và thường chạy các nhóm consensus với năm tiến trình (mỗi datacenter một tiến trình), thì khi mất một datacenter, mỗi nhóm vẫn còn lại một bản sao dự phòng. Tuy nhiên, nếu triển khai bản sao thứ sáu trong một trong năm datacenter đó, sự cố tại datacenter này sẽ loại bỏ cả hai bản sao dự phòng trong nhóm, khiến dung lượng giảm 33%.

<a id="hinh-23-10"></a>        ![Thêm một bản sao phụ trong một khu vực có thể làm giảm khả dụng của hệ thống. Chụm nhiều bản sao trong một datacenter đơn có thể làm giảm khả dụng của hệ thống: ở đây có một quorum mà không còn độ dự phòng nào.](../assets/imgs/fig-23-10.jpg)

Hình 23-10. Thêm một bản sao phụ trong một khu vực có thể làm giảm khả dụng của hệ thống. Chụm nhiều bản sao trong một datacenter đơn có thể làm giảm khả dụng của hệ thống: ở đây, có một quorum mà không còn độ dự phòng nào.

Nếu các client dày đặc trong một khu vực địa lý cụ thể, tốt nhất là đặt các bản sao gần các client. Tuy nhiên, việc quyết định chính xác đặt các bản sao ở đâu có thể cần một chút suy nghĩ cẩn thận về cân bằng tải và cách một hệ thống xử lý quá tải. Như được thể hiện trong [Hình 23-11](#hinh-23-11), nếu một hệ thống đơn giản định tuyến các request đọc của client đến bản sao gần nhất, thì một cơn tăng vọt tải lớn tập trung trong một khu vực có thể làm choáng bản sao gần nhất, rồi đến bản sao gần thứ hai, và cứ thế — đây là *cascading failure* (xem [Đối phó với Các Sự cố Lan truyền](https://sre.google/sre-book/addressing-cascading-failures/)). Kiểu quá tải này thường có thể xảy ra do kết quả của việc các batch job bắt đầu, đặc biệt nếu một vài lô bắt đầu cùng lúc.

Chúng ta đã thấy lý do vì sao nhiều hệ thống distributed consensus dùng một tiến trình leader để cải thiện hiệu năng. Tuy nhiên, cần hiểu rằng các bản sao leader sẽ tiêu tốn nhiều tài nguyên tính toán hơn, đặc biệt là năng lực mạng phát ra. Nguyên nhân là leader gửi các tin nhắn proposal chứa dữ liệu được đề xuất, trong khi các bản sao chỉ gửi các tin nhắn nhỏ hơn, thường chỉ bao gồm sự đồng ý với một ID giao dịch consensus cụ thể. Các tổ chức vận hành hệ thống consensus được chia shard cao với số lượng tiến trình rất lớn có thể thấy cần đảm bảo các tiến trình leader cho các shard khác nhau được phân bổ tương đối đều trên các datacenter khác nhau. Việc này ngăn toàn bộ hệ thống bị nút thắt về năng lực mạng phát ra tại một datacenter, đồng thời tạo ra dung lượng hệ thống tổng thể lớn hơn.

<a id="hinh-23-11"></a>        ![Chụm các tiến trình leader dẫn đến việc sử dụng băng thông không đều.](../assets/imgs/fig-23-11.jpg)

Hình 23-11. Chụm các tiến trình leader dẫn đến việc sử dụng băng thông không đều

Một bất lợi khác khi triển khai các nhóm consensus trên nhiều datacenter (xem [Hình 23-11](#hinh-23-11)) là hệ thống có thể trải qua những thay đổi cực đoan nếu datacenter chứa các leader gặp sự cố diện rộng (ví dụ mất điện, lỗi thiết bị mạng, hoặc đứt cáp quang). Như [Hình 23-12](#hinh-23-12) cho thấy, trong kịch bản này, tất cả các leader sẽ fail over sang một datacenter khác, hoặc chia đều, hoặc dồn hết vào một datacenter. Trong cả hai trường hợp, liên kết giữa hai datacenter còn lại sẽ đột ngột phải gánh một lượng lưu lượng mạng lớn hơn rất nhiều từ hệ thống. Đây sẽ là thời điểm không may để phát hiện ra rằng dung lượng của liên kết đó không đủ.

<a id="hinh-23-12"></a>        ![Khi các leader chụm fail over cả đám, các mẫu sử dụng mạng thay đổi kịch tính.](../assets/imgs/fig-23-12.jpg)

Hình 23-12. Khi các leader chụm fail over cả đám, các mẫu sử dụng mạng thay đổi kịch tính

Tuy nhiên, kiểu triển khai này có thể dễ dàng trở thành kết quả không chủ ý của các quá trình tự động trong hệ thống, vốn ảnh hưởng đến cách các leader được chọn. Ví dụ:

- Các client sẽ có độ trễ tốt hơn cho bất kỳ thao tác nào được xử lý qua leader nếu leader nằm gần họ nhất. Một thuật toán cố gắng đặt các leader gần khối các client có thể tận dụng điểm nhìn này.
- Một thuật toán có thể cố gắng đặt các leader trên các máy có hiệu năng tốt nhất. Một cái bẫy của cách tiếp cận này là nếu một trong ba datacenter chứa các máy nhanh hơn, thì một tỷ lệ không cân xứng các lưu lượng sẽ được gửi đến datacenter đó, dẫn đến các thay đổi lưu lượng cực đoan nếu datacenter đó đi offline. Để tránh vấn đề này, thuật toán cũng phải tính đến sự cân bằng phân bố đối với khả năng máy khi chọn máy.
- Thuật toán bầu leader có thể thiên vị các tiến trình đã chạy lâu hơn. Nếu việc phát hành phần mềm được thực hiện theo từng datacenter, thời gian chạy của tiến trình sẽ có tương quan khá cao với vị trí.

## Thành phần của Quorum (Quorum composition)

Khi xác định vị trí đặt các bản sao trong một nhóm consensus, cần xem xét tác động của việc phân bố địa lý (hay chính xác hơn là các độ trễ mạng giữa các bản sao) đến hiệu năng của nhóm.

Một cách tiếp cận là phân tán các bản sao sao cho đều nhất có thể, đảm bảo RTT giữa tất cả các bản sao tương đương nhau. Khi các yếu tố khác (workload, phần cứng, hiệu năng mạng) giữ nguyên, cách sắp xếp này sẽ mang lại hiệu năng khá nhất quán trên mọi khu vực, bất kể leader nhóm đặt ở đâu (hoặc, nếu dùng giao thức không-leader, thì với từng thành viên của nhóm consensus).

Địa lý có thể khiến cách tiếp cận này trở nên phức tạp hơn rất nhiều, đặc biệt khi so sánh lưu lượng nội-châu lục với lưu lượng xuyên Thái Bình Dương và xuyên Đại Tây Dương. Xét một hệ thống trải rộng Bắc Mỹ và Châu Âu: không thể đặt các bản sao cách đều nhau, vì lưu lượng xuyên Đại Tây Dương luôn có độ trễ dài hơn so với lưu lượng nội-châu lục. Dù bằng cách nào, các giao dịch từ một khu vực vẫn phải thực hiện một vòng đi-về xuyên Đại Tây Dương để đạt được consensus.

Tuy nhiên, như [Hình 23-13](#hinh-23-13) cho thấy, để phân phối lưu lượng đều nhất có thể, các nhà thiết kế hệ thống có thể chọn đặt năm bản sao: hai bản sao ở vị trí trung tâm tại Mỹ, một ở bờ đông, và hai ở Châu Âu. Với cách bố trí này, trong trường hợp trung bình, consensus có thể đạt được ở Bắc Mỹ mà không cần đợi phản hồi từ Châu Âu; hoặc từ Châu Âu, consensus có thể đạt được bằng cách chỉ trao đổi tin nhắn với bản sao bờ đông. Bản sao bờ đông đóng vai trò như một mối liên kết (linchpin), nơi hai quorum có thể chồng lấn nhau.

<a id="hinh-23-13"></a>        ![Các quorum chồng lấn với một bản sao đóng vai trò liên kết.](../assets/imgs/fig-23-13.jpg)

Hình 23-13. Các quorum chồng lấn với một bản sao đóng vai trò liên kết

Như [Hình 23-14](#hinh-23-14) cho thấy, việc mất bản sao này có thể khiến độ trễ hệ thống thay đổi đáng kể: thay vì chủ yếu phụ thuộc vào RTT từ trung tâm Mỹ đến bờ đông hoặc RTT từ EU đến bờ đông, độ trễ sẽ dựa trên RTT từ EU đến trung tâm, con số cao hơn khoảng 50% so với RTT từ EU đến bờ đông. Khoảng cách địa lý và RTT mạng giữa quorum gần nhất có thể tăng lên rất lớn.

<a id="hinh-23-14"></a>        ![Việc mất bản sao liên kết ngay lập tức dẫn đến một RTT dài hơn cho bất kỳ quorum nào.](../assets/imgs/fig-23-14.jpg)

Hình 23-14. Việc mất bản sao liên kết ngay lập tức dẫn đến một RTT dài hơn cho bất kỳ quorum nào

Kịch bản này là một điểm yếu chính của quorum đa số đơn giản khi áp dụng cho các nhóm gồm các bản sao có RTT rất khác nhau giữa các thành viên. Trong những trường hợp như vậy, một cách tiếp cận quorum phân cấp (hierarchical quorum) có thể hữu ích. Như được mô tả trong [Hình 23-15](#hinh-23-15), chín bản sao có thể được triển khai trong ba nhóm ba. Một quorum có thể được tạo thành bởi một đa số các nhóm, và một nhóm có thể được đưa vào quorum nếu một đa số các thành viên của nhóm đó khả dụng. Điều này có nghĩa là một bản sao có thể bị mất trong nhóm trung tâm mà không gây ra một tác động lớn lên hiệu năng hệ thống tổng thể, vì nhóm trung tâm vẫn có thể bỏ phiếu cho các giao dịch với hai trong ba bản sao của nó.

Tuy nhiên, chạy nhiều bản sao hơn sẽ phát sinh chi phí tài nguyên. Trong hệ thống chia shard cao, nếu workload chủ yếu là đọc và có thể đáp ứng bằng các bản sao, ta có thể giảm chi phí này bằng cách dùng ít nhóm consensus hơn. Chiến lược này đồng nghĩa với việc tổng số tiến trình trong hệ thống có thể không thay đổi.

<a id="hinh-23-15"></a>        ![Các quorum phân cấp có thể được dùng để giảm sự phụ thuộc vào bản sao trung tâm.](../assets/imgs/fig-23-15.jpg)

Hình 23-15. Các quorum phân cấp có thể được dùng để giảm sự phụ thuộc vào bản sao trung tâm

## Giám Sát Các Hệ Thống Distributed Consensus (Monitoring Distributed Consensus Systems)

Như đã thấy, các thuật toán distributed consensus đóng vai trò trung tâm trong nhiều hệ thống quan trọng của Google ([\[Ana13\]](https://sre.google/sre-book/bibliography#Ana13), [\[Bur06\]](https://sre.google/sre-book/bibliography#Bur06), [\[Cor12\]](https://sre.google/sre-book/bibliography#Cor12), [\[Shu13\]](https://sre.google/sre-book/bibliography#Shu13)). Mọi hệ thống sản xuất quan trọng đều cần giám sát nhằm phát hiện sự cố hoặc vấn đề và hỗ trợ gỡ lỗi. Qua thực tế, chúng tôi nhận thấy một số khía cạnh cụ thể của các hệ thống distributed consensus cần được đặc biệt lưu ý. Đó là:

Số lượng các thành viên đang chạy trong mỗi nhóm consensus, và trạng thái của mỗi tiến trình (lành mạnh hay không lành mạnh)

Một tiến trình có thể đang chạy nhưng không thể tiến triển vì một lý do nào đó (ví dụ, liên quan đến phần cứng).

### Các bản sao liên tục bị tụt lại (persistently lagging replicas)

Các thành viên lành mạnh trong một nhóm consensus vẫn có thể ở nhiều trạng thái khác nhau. Một thành viên có thể đang khôi phục trạng thái từ các peer sau khi khởi động, hoặc đang tụt lại phía sau quorum, hoặc đã cập nhật và tham gia đầy đủ, thậm chí có thể là leader.

Liệu có tồn tại một leader hay không

Hệ thống dựa trên thuật toán kiểu Multi-Paxos, nơi có vai trò leader, cần được giám sát để đảm bảo luôn có một leader tồn tại; nếu không có leader, hệ thống sẽ hoàn toàn không khả dụng.

Số lần thay đổi leader

Việc thay đổi leader liên tục làm giảm hiệu năng của các hệ thống consensus vốn dựa trên một leader ổn định, do đó cần giám sát số lần thay đổi này. Các thuật toán consensus thường đánh dấu mỗi lần thay đổi bằng một term (nhiệm kỳ) hoặc view number mới, nên con số này là chỉ số hữu ích để theo dõi. Nếu số lần thay đổi leader tăng quá nhanh, đó là dấu hiệu leader đang flapping, có thể do sự cố kết nối mạng. Ngược lại, nếu view number giảm, điều này có thể báo hiệu một bug nghiêm trọng.

Số giao dịch consensus

Nhà vận hành cần biết hệ thống consensus có đang tiến triển hay không. Hầu hết các thuật toán consensus dùng một số giao dịch consensus tăng dần để chỉ báo sự tiến triển. Nếu hệ thống lành mạnh, con số này nên được thấy đang tăng dần theo thời gian.

Số proposal đã nhìn thấy; số proposal được đồng thuận

Những con số này chỉ báo liệu hệ thống có đang hoạt động đúng hay không.

Thông lượng và độ trễ

Mặc dù không đặc thù cho các hệ thống distributed consensus, những đặc tính này của hệ thống consensus nên được các quản trị viên giám sát và hiểu.

Để hiểu hiệu năng hệ thống và giúp gỡ lỗi các vấn đề hiệu năng, bạn cũng có thể giám sát những điều sau:

- Các phân bố độ trễ cho việc chấp nhận proposal
- Các phân bố của các độ trễ mạng quan sát được giữa các phần của hệ thống ở các vị trí khác nhau
- Thời lượng mà các acceptor dành cho việc ghi nhật ký bền vững
- Tổng số byte được chấp nhận mỗi giây trong hệ thống

## Kết Luận (Conclusion)

Chúng ta đã tìm hiểu định nghĩa của bài toán distributed consensus, giới thiệu một số mẫu kiến trúc hệ thống cho các hệ thống dựa trên distributed consensus, đồng thời xem xét các đặc tính hiệu năng và một số mối quan tâm vận hành liên quan đến các hệ thống này.

Trong chương này, chúng tôi cố ý không đi sâu vào các thuật toán, giao thức hay bản cài đặt cụ thể. Các hệ thống phối hợp phân tán và công nghệ nền tảng của chúng đang thay đổi rất nhanh, nên những chi tiết kỹ thuật sẽ sớm trở nên lỗi thời. Tuy nhiên, với các nền tảng được đề cập ở đây cùng những bài báo được trích dẫn xuyên suốt chương, bạn sẽ có đủ cơ sở để làm việc với các công cụ phối hợp phân tán hiện có cũng như phần mềm trong tương lai.

Nếu chỉ nhớ được một điều từ chương này, hãy ghi nhớ các loại vấn đề mà distributed consensus có thể giải quyết, cũng như những rủi ro phát sinh khi dùng các phương pháp tự phát như heartbeat thay cho distributed consensus. Mỗi khi gặp leader election, trạng thái chia sẻ quan trọng hoặc distributed locking, hãy nghĩ ngay đến distributed consensus: mọi cách tiếp cận kém hơn đều là quả bom hẹn giờ chực chờ phát nổ trong hệ thống của bạn.

<a id="fn1"></a>[1](#fn1) Kyle Kingsbury đã viết một loạt bài báo rộng rãi về tính đúng đắn của các hệ thống phân tán, trong đó chứa nhiều ví dụ về hành vi bất ngờ và sai trong các kiểu datastore này. Xem [*https://aphyr.com/tags/jepsen*](https://aphyr.com/tags/jepsen).

<a id="fn2"></a>[2](#fn2) Cụ thể, hiệu năng của thuật toán Paxos nguyên thủy là không tối ưu, nhưng đã được cải thiện rất nhiều qua nhiều năm.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
