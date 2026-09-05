> **Nguyên bản:** [Chapter 24 - Distributed Periodic Scheduling with Cron](https://sre.google/sre-book/distributed-periodic-scheduling/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

## Chương 24. Lên lịch Định kỳ Phân tán bằng Cron (Distributed Periodic Scheduling with Cron)

*Tác giả:* Štěpán Davidovič<sup>[1](#fn1)</sup>
*Biên tập:* Kavita Guliani

Chương này trình bày cách Google xây dựng dịch vụ cron (trình lên lịch) phân tán, phục vụ phần lớn các nhóm nội bộ cần lên lịch định kỳ cho các job tính toán. Trong suốt thời gian vận hành, chúng tôi rút ra nhiều bài học về thiết kế và triển khai một dịch vụ mà thoạt nhìn có vẻ rất cơ bản. Dưới đây, chúng tôi thảo luận các vấn đề mà cron phân tán phải đối mặt và phác thảo một số giải pháp khả thi.

Cron là tiện ích Unix phổ biến, dùng để chạy các job định kỳ theo thời điểm hoặc chu kỳ do người dùng xác định. Trước tiên, chúng tôi phân tích các nguyên lý cơ bản của cron cùng những cách triển khai phổ biến nhất. Sau đó, chúng tôi xem xét một ứng dụng tương tự cron có thể hoạt động ra sao trong môi trường phân tán quy mô lớn, nhằm tăng độ tin cậy của hệ thống trước sự cố của một máy riêng lẻ. Chúng tôi mô tả một hệ thống cron phân tán chạy trên một số lượng nhỏ các máy, nhưng có khả năng khởi chạy job cron trên toàn bộ datacenter, kết hợp với hệ thống lên lịch datacenter như Borg [[Ver15]](https://sre.google/sre-book/bibliography#Ver15).

## Cron

Trước khi đi sâu vào việc vận hành cron như một dịch vụ xuyên datacenter, hãy thảo luận về cách cron thường được sử dụng trong trường hợp đơn máy.

## Giới thiệu (Introduction)

Cron được thiết kế để cả quản trị viên hệ thống lẫn người dùng phổ thông đều có thể chỉ định các lệnh cần chạy cùng thời điểm thực thi. Cron xử lý nhiều loại job khác nhau, từ thu gom rác (garbage collection) đến phân tích dữ liệu định kỳ. Định dạng chỉ định thời gian phổ biến nhất là "crontab". Định dạng này hỗ trợ các khoảng thời gian đơn giản (ví dụ, "một lần mỗi ngày vào 12 giờ trưa" hoặc "mỗi giờ đúng giờ"). Các khoảng thời gian phức tạp hơn, chẳng hạn "mỗi thứ Bảy, cũng là ngày 30 trong tháng", cũng có thể được cấu hình.

Cron thường được triển khai dưới dạng một thành phần duy nhất, thường gọi là `crond`. `crond` là một daemon nạp danh sách các job cron đã lên lịch. Các job này được khởi chạy theo thời gian thực thi đã chỉ định.

## Góc nhìn Độ tin cậy (Reliability Perspective)

Một số khía cạnh của dịch vụ cron đáng chú ý từ góc độ độ tin cậy:

-   Failure domain của cron về cơ bản chỉ là một máy. Nếu máy này gặp sự cố, cả bộ lên lịch cron lẫn các job do nó khởi chạy đều không thể hoạt động.<sup>[2](#fn2)</sup> Hãy xem xét một trường hợp phân tán rất đơn giản với hai máy, trong đó bộ lên lịch cron của bạn khởi chạy job trên một máy worker khác (ví dụ, bằng SSH). Kịch bản này đặt ra hai failure domain khác nhau có thể ảnh hưởng đến khả năng khởi chạy job của chúng ta: máy lên lịch hoặc máy đích đều có thể thất bại.
-   Trạng thái (state) duy nhất cần được duy trì vượt qua các lần khởi động lại `crond` (bao gồm cả khởi động lại máy) chính là cấu hình crontab. Việc khởi chạy cron là kiểu "bắn rồi quên" (fire-and-forget), và `crond` không hề cố gắng theo dõi những lần khởi chạy này.
-   anacron là một ngoại lệ đáng chú ý. anacron cố gắng khởi chạy các job lẽ ra đã được thực hiện trong khi hệ thống đang tắt. Các nỗ lực khởi chạy lại chỉ giới hạn ở các job chạy theo ngày hoặc ít thường xuyên hơn. Chức năng này rất hữu ích để chạy các job bảo trì trên các workstation và notebook, và được hỗ trợ bởi một file lưu giữ dấu thời gian của lần khởi chạy cuối cùng cho tất cả các job cron đã đăng ký.

## Các Job Cron và Tính Idempotent (Cron Jobs and Idempotency)

Các job cron được thiết kế để chạy các tác vụ định kỳ, nhưng ngoài ra, rất khó đoán trước chức năng cụ thể của chúng. Sự đa dạng trong các yêu cầu mà hệ thống job cron phong phú này đặt ra rõ ràng tác động đến yêu cầu về độ tin cậy.

Một số job cron, chẳng hạn các quy trình thu gom rác, có tính idempotent (chạy nhiều lần cho cùng một kết quả). Vì vậy, nếu hệ thống gặp trục trặc, việc khởi chạy các job này nhiều lần vẫn an toàn. Ngược lại, các job cron khác, chẳng hạn quy trình gửi một bản tin email đến một danh sách người nhận lớn, không nên được khởi chạy hơn một lần.

Thêm vào đó, việc không khởi chạy được có thể chấp nhận được với một số job cron nhưng không phải với tất cả. Chẳng hạn, một job cron thu gom rác được lên lịch chạy mỗi năm phút có thể bỏ qua một lần chạy, nhưng một job cron tính lương được lên lịch chạy một lần mỗi tháng thì không được phép bỏ lỡ.

Sự đa dạng lớn của các job cron khiến việc suy luận về các failure mode trở nên khó khăn: trong một hệ thống như dịch vụ cron, không có một câu trả lời duy nhất phù hợp với mọi tình huống. Nhìn chung, chúng tôi nghiêng về việc bỏ qua các lần khởi chạy thay vì chấp nhận rủi ro khởi chạy trùng lặp, trong chừng mực hạ tầng cho phép. Lý do là việc phục hồi từ một lần khởi chạy bị bỏ qua có thể chấp nhận được hơn so với việc phục hồi từ một lần khởi chạy trùng lặp. Chủ sở hữu của các job cron có thể (và nên!) giám sát job cron của họ; ví dụ, một chủ sở hữu có thể để dịch vụ cron phơi bày trạng thái cho các job cron do nó quản lý, hoặc thiết lập giám sát độc lập về hiệu ứng của các job cron. Trong trường hợp một lần khởi chạy bị bỏ qua, chủ sở hữu job cron có thể thực hiện hành động phù hợp với bản chất của job cron đó. Tuy nhiên, việc hoàn tác một lần khởi chạy trùng lặp, như ví dụ bản tin email nêu trước đó, có thể khó khăn hoặc thậm chí hoàn toàn bất khả thi. Do đó, chúng tôi ưu tiên "thất bại đóng" (fail closed) để tránh tạo ra trạng thái xấu mang tính hệ thống.

## Cron ở Quy mô Lớn (Cron at Large Scale)

Chuyển từ các máy riêng lẻ sang triển khai quy mô lớn đòi hỏi phải xem xét lại cơ bản cách đưa cron hoạt động hiệu quả trong môi trường đó. Trước khi đi vào chi tiết giải pháp cron của Google, chúng tôi sẽ thảo luận về sự khác biệt giữa triển khai quy mô nhỏ và quy mô lớn, đồng thời mô tả những thay đổi thiết kế cần thiết cho triển khai quy mô lớn.

## Hạ tầng Mở rộng (Extended Infrastructure)

Trong các cách triển khai "thông thường", cron bị giới hạn trong một máy. Việc triển khai hệ thống quy mô lớn mở rộng giải pháp cron của chúng tôi sang nhiều máy.

Việc đặt dịch vụ cron trên một máy duy nhất có thể gây ra thảm họa về độ tin cậy. Giả sử máy này nằm trong một datacenter gồm đúng 1.000 máy. Chỉ cần 1/1000 số máy khả dụng gặp sự cố là toàn bộ dịch vụ cron có thể bị đánh sập. Vì những lý do rõ ràng, cách triển khai này không thể chấp nhận được.

Để tăng độ tin cậy của cron, chúng tôi tách rời các tiến trình khỏi các máy. Nếu bạn muốn chạy một dịch vụ, chỉ cần chỉ định các yêu cầu của dịch vụ và datacenter mà nó nên chạy. Hệ thống lên lịch datacenter (chính nó cũng nên có độ tin cậy) sẽ quyết định máy hoặc các máy để triển khai dịch vụ của bạn, đồng thời xử lý sự chết máy. Khi đó, việc khởi chạy một job trong datacenter hiệu quả trở thành việc gửi một hoặc nhiều RPC (Remote Procedure Call — lời gọi thủ tục từ xa) đến bộ lên lịch datacenter.

Tuy nhiên, quá trình này không diễn ra tức thì. Việc phát hiện một máy chết phải chờ các lần kiểm tra sức khỏe, trong khi việc lên lịch lại dịch vụ sang máy khác cũng cần thời gian để cài đặt phần mềm và khởi động tiến trình mới.

Việc di chuyển tiến trình sang máy khác có thể khiến mất toàn bộ trạng thái cục bộ trên máy cũ (trừ khi dùng live migration), trong khi thời gian lên lịch lại có thể vượt quá mức tối thiểu là một phút. Do đó, cần có các biện pháp để giảm thiểu cả tình trạng mất dữ liệu lẫn yêu cầu thời gian quá mức. Để giữ trạng thái cục bộ của máy cũ, bạn có thể lưu trạng thái trên một distributed filesystem như GFS, sau đó dùng hệ thống tệp này khi khởi động để xác định các job chưa chạy được do lên lịch lại. Tuy nhiên, giải pháp này không đáp ứng được yêu cầu về tính kịp thời: nếu chạy một job cron mỗi năm phút, sự chậm trễ một đến hai phút do tổng overhead (công việc phụ) của việc lên lịch lại hệ thống cron có thể đáng kể đến mức không thể chấp nhận. Trong trường hợp này, các hot spare (bản sao dự phòng nóng) có thể nhanh chóng vào cuộc và tiếp tục vận hành, giúp rút ngắn đáng kể khung thời gian này.

## Yêu cầu Mở rộng (Extended Requirements)

Các hệ thống đơn máy thường chỉ chạy chung tất cả các tiến trình với mức cách ly hạn chế. Dù container đã trở nên phổ biến, việc dùng container để cách ly từng thành phần riêng lẻ của một dịch vụ trên cùng một máy không phải là điều bắt buộc hay thông dụng. Vì vậy, nếu cron được triển khai trên một máy, `crond` và tất cả các job cron mà nó chạy có lẽ sẽ không được cách ly.

Triển khai ở quy mô datacenter thường đồng nghĩa với việc triển khai vào các container để thực thi sự cách ly. Sự cách ly là cần thiết vì kỳ vọng cơ bản là các tiến trình độc lập chạy trong cùng một datacenter không nên ảnh hưởng xấu đến lẫn nhau. Để thực thi kỳ vọng đó, bạn cần biết lượng tài nguyên cần thu thập trước cho bất kỳ tiến trình nào bạn muốn chạy — cả cho hệ thống cron lẫn các job mà nó khởi chạy. Một job cron có thể bị trì hoãn nếu datacenter không có tài nguyên khả dụng để đáp ứng nhu cầu của job cron đó. Yêu cầu về tài nguyên, cùng với nhu cầu của người dùng về giám sát các lần khởi chạy job cron, có nghĩa là chúng tôi cần theo dõi toàn bộ trạng thái của các lần khởi chạy job cron, từ lúc lên lịch khởi chạy cho đến khi kết thúc.

Việc tách rời các lần khởi chạy tiến trình khỏi các máy cụ thể khiến hệ thống cron phơi bày trước sự cố khởi chạy một phần. Vì cấu hình job cron rất đa dạng, việc khởi chạy một job cron mới trong datacenter có thể cần nhiều RPC, sao cho đôi khi chúng tôi gặp kịch bản trong đó một số RPC thành công nhưng số khác không (ví dụ, vì tiến trình gửi các RPC đã chết giữa chừng khi thực hiện các tác vụ này). Thủ tục phục hồi của cron cũng phải tính đến kịch bản này.

Về failure mode, một datacenter là một hệ sinh thái phức tạp hơn đáng kể so với một máy riêng lẻ. Dịch vụ cron, vốn bắt đầu là một binary tương đối đơn giản trên một máy, giờ có nhiều phụ thuộc rõ ràng lẫn không rõ ràng khi được triển khai ở quy mô lớn hơn. Đối với một dịch vụ cơ bản như cron, chúng tôi muốn đảm bảo rằng ngay cả khi datacenter chịu một sự cố một phần (ví dụ, mất điện cục bộ hoặc các vấn đề với dịch vụ lưu trữ), dịch vụ vẫn có thể hoạt động. Bằng cách yêu cầu bộ lên lịch datacenter đặt các replica của cron ở các vị trí đa dạng trong datacenter, chúng tôi tránh được kịch bản trong đó sự cố của một đơn vị phân phối điện kéo sập tất cả các tiến trình của dịch vụ cron.

Có thể triển khai một dịch vụ cron duy nhất trên toàn cầu, nhưng việc triển khai cron trong một datacenter đơn lẻ mang lại những lợi ích: dịch vụ có độ trễ thấp và cùng số phận với bộ lên lịch datacenter, vốn là phụ thuộc cốt lõi của cron.

## Xây dựng Cron tại Google (Building Cron at Google)

Phần này tập trung vào những vấn đề cần giải quyết để triển khai cron phân tán quy mô lớn một cách đáng tin cậy. Đồng thời, nó cũng làm nổi bật một số quyết định quan trọng liên quan đến cron phân tán tại Google.

## Theo dõi Trạng thái của Các Job Cron (Tracking the State of Cron Jobs)

Như đã đề cập ở các phần trước, hệ thống cần lưu trữ một phần trạng thái của các job cron và phải khôi phục được thông tin này nhanh chóng khi xảy ra sự cố. Ngoài ra, tính nhất quán của trạng thái là yếu tố tối quan trọng. Cần lưu ý rằng nhiều job cron, chẳng hạn như chạy tính lương hay gửi bản tin email, không có tính idempotent.

Chúng tôi có hai lựa chọn để theo dõi trạng thái của các job cron:

-   Lưu dữ liệu bên ngoài trong distributed storage nói chung khả dụng
-   Sử dụng một hệ thống lưu trữ một lượng trạng thái nhỏ như một phần của chính dịch vụ cron

Khi thiết kế cron phân tán, chúng tôi chọn phương án thứ hai. Chúng tôi đưa ra lựa chọn này vì một số lý do:

-   Các distributed filesystem như GFS hay HDFS thường nhắm đến trường hợp sử dụng với các file rất lớn (ví dụ, đầu ra của các chương trình web crawling), trong khi thông tin chúng tôi cần lưu về các job cron lại rất nhỏ. Các thao tác ghi nhỏ trên distributed filesystem rất đắt đỏ và đi kèm độ trễ cao, bởi vì hệ thống tệp không được tối ưu cho các kiểu ghi như vậy.
-   Các dịch vụ cơ bản mà sự cố của chúng gây tác động rộng (như cron) nên có rất ít phụ thuộc. Ngay cả khi một phần của datacenter biến mất, dịch vụ cron vẫn nên có thể hoạt động trong ít nhất một khoảng thời gian. Nhưng yêu cầu này không có nghĩa là bộ lưu trữ phải là một phần trực tiếp của tiến trình cron (việc lưu trữ được xử lý như thế nào về cơ bản là một chi tiết triển khai). Tuy nhiên, cron nên có thể hoạt động độc lập với các hệ thống phía sau phục vụ một số lượng lớn người dùng nội bộ.

## Việc Sử dụng Paxos (The Use of Paxos)

Chúng tôi triển khai nhiều replica cho dịch vụ cron và dùng thuật toán nhất trí phân tán Paxos (xem [Quản lý Trạng thái Quan trọng: Nhất trí Phân tán cho Độ tin cậy](https://sre.google/sre-book/managing-critical-state/)) để đảm bảo trạng thái nhất quán. Khi đa số thành viên trong nhóm khả dụng, hệ thống phân tán nhìn chung vẫn xử lý thành công các thay đổi trạng thái mới, ngay cả khi một phần hạ tầng gặp sự cố.

Như [Hình 24-1](#hinh-24-1) cho thấy, cron phân tán chỉ có một job leader duy nhất. Replica này là duy nhất có quyền sửa đổi trạng thái chia sẻ và khởi chạy các job cron. Chúng tôi tận dụng đặc điểm của Fast Paxos [[Lam06]](https://sre.google/sre-book/bibliography#Lam06) — biến thể Paxos mà chúng tôi sử dụng — vốn dùng một replica leader bên trong như một tối ưu. Theo đó, replica leader của Fast Paxos đồng thời đảm nhận vai trò leader của dịch vụ cron.

<a id="hinh-24-1"></a>        ![Tương tác giữa các replica của cron phân tán.](../assets/imgs/fig-24-1.jpg)

Hình 24-1. Tương tác giữa các replica của cron phân tán

Nếu replica leader chết, cơ chế kiểm tra sức khỏe của nhóm Paxos phát hiện sự kiện này một cách nhanh chóng (trong vòng vài giây). Vì một tiến trình cron khác đã được khởi động và khả dụng, chúng tôi có thể bầu ra một leader mới. Ngay khi leader mới được bầu, chúng tôi tuân theo một giao thức bầu leader riêng của dịch vụ cron, chịu trách nhiệm tiếp quản toàn bộ công việc còn dở dang mà leader trước để lại. Leader riêng của dịch vụ cron giống với leader Paxos, nhưng dịch vụ cron cần thực hiện thêm hành động khi được thăng chức. Thời gian phản hồi nhanh cho việc bầu lại leader cho phép chúng tôi duy trì trong ngưỡng failover một phút thường được chấp nhận.

Trạng thái quan trọng nhất mà chúng tôi lưu trữ trong Paxos là thông tin về các job cron đã được khởi chạy. Chúng tôi đồng bộ hóa một quorum (đa số) các replica để thông báo về thời điểm bắt đầu và kết thúc của mỗi lần khởi chạy đã lên lịch cho từng job cron.

## Vai trò của Leader và Follower (The Roles of the Leader and the Follower)

Như đã trình bày, hệ thống cron của chúng tôi sử dụng Paxos với hai vai trò được gán: leader và follower. Các phần tiếp theo sẽ mô tả chi tiết từng vai trò.

### Leader (The leader)

Replica leader là replica duy nhất chủ động khởi chạy các job cron. Leader có bộ lên lịch bên trong, hoạt động tương tự `crond` đơn giản được mô tả ở đầu chương này, để duy trì danh sách các job cron sắp xếp theo thời gian khởi chạy đã lên lịch. Replica leader sẽ chờ đến khi tới thời điểm khởi chạy của job đầu tiên.

Đến giờ khởi chạy đã lên lịch, replica leader thông báo rằng nó sắp bắt đầu thực thi job cron cụ thể này, đồng thời tính toán giờ khởi chạy tiếp theo, tương tự như một tiến trình `crond` thông thường. Cũng giống như `crond` thông thường, định nghĩa của job cron có thể đã thay đổi kể từ lần thực thi trước, do đó định nghĩa này cũng phải được đồng bộ hóa với các follower. Việc chỉ nhận dạng job cron là chưa đủ: chúng tôi cần xác định duy nhất lần khởi chạy cụ thể bằng thời điểm bắt đầu; nếu không, việc theo dõi các lần khởi chạy job cron có thể gây nhầm lẫn. (Sự mơ hồ này đặc biệt dễ xảy ra với các job cron tần suất cao, chẳng hạn như những job chạy mỗi phút.) Như minh họa trong [Hình 24-2](#hinh-24-2), quá trình giao tiếp này được thực hiện thông qua Paxos.

Điều quan trọng là giao tiếp Paxos phải duy trì tính đồng bộ, và việc khởi chạy job cron thực tế chỉ được tiến hành sau khi nhận được xác nhận rằng quorum Paxos đã nhận thông báo khởi chạy. Dịch vụ cron cần biết liệu mỗi job cron có được khởi chạy hay không để quyết định hướng xử lý tiếp theo trong trường hợp failover của leader. Nếu không thực hiện đồng bộ, toàn bộ lần khởi chạy job cron có thể diễn ra trên leader mà không thông báo cho các replica follower. Khi đó, trong trường hợp failover, các replica follower có thể cố gắng thực hiện lại cùng lần khởi chạy đó vì chúng không hay biết rằng nó đã diễn ra.

<a id="hinh-24-2"></a>        ![Minh họa tiến trình của một lần khởi chạy job cron, từ góc nhìn của leader.](../assets/imgs/fig-24-2.jpg)

Hình 24-2. Minh họa tiến trình của một lần khởi chạy job cron, từ góc nhìn của leader

Việc hoàn thành lần khởi chạy job cron được thông báo đồng bộ qua Paxos đến các replica khác. Lưu ý rằng việc khởi chạy thành công hay thất bại do các nguyên nhân bên ngoài không quan trọng (ví dụ, nếu bộ lên lịch datacenter không khả dụng). Ở đây, chúng tôi đơn giản là theo dõi thực tế rằng dịch vụ cron đã cố gắng khởi chạy tại thời điểm đã lên lịch. Chúng tôi cũng cần có khả năng giải quyết các sự cố của hệ thống cron giữa chừng của thao tác này, như thảo luận trong phần tiếp theo.

Một tính năng cực kỳ quan trọng khác của leader là ngay khi mất vai trò này vì bất kỳ lý do nào, nó phải lập tức dừng tương tác với bộ lên lịch datacenter. Việc giữ vai trò leader phải đảm bảo sự loại trừ tương hỗ khi truy cập bộ lên lịch datacenter. Nếu thiếu điều kiện loại trừ tương hỗ này, leader cũ và leader mới có thể thực hiện các hành động mâu thuẫn trên bộ lên lịch datacenter.

### Follower (The follower)

Các replica follower theo dõi trạng thái hệ thống do leader cung cấp, sẵn sàng tiếp quản ngay khi cần. Mọi thay đổi trạng thái mà các follower ghi nhận đều được truyền qua Paxos từ replica leader. Tương tự leader, các follower cũng duy trì danh sách toàn bộ job cron trong hệ thống; danh sách này phải được giữ nhất quán giữa các replica (bằng cách sử dụng Paxos).

Khi nhận được thông báo về một lần khởi chạy đã bắt đầu, replica follower cập nhật thời gian khởi chạy đã lên lịch tiếp theo cục bộ của nó cho job cron được cho. Thay đổi trạng thái cực kỳ quan trọng này (được thực hiện đồng bộ) đảm bảo rằng tất cả các lịch cron job trong hệ thống là nhất quán. Chúng tôi theo dõi tất cả các lần khởi chạy đang mở (các lần khởi chạy đã bắt đầu nhưng chưa hoàn thành).

Nếu replica leader chết hoặc gặp trục trặc theo cách khác (ví dụ, bị tách rời khỏi các replica khác trên mạng), một follower cần được bầu làm leader mới. Cuộc bầu cử phải hội tụ trong thời gian ngắn hơn một phút, nhằm tránh rủi ro bỏ lỡ hoặc trì hoãn không hợp lý một lần khởi chạy job cron. Sau khi bầu xong leader, tất cả các lần khởi chạy đang mở (tức là các sự cố một phần) phải được kết thúc. Quá trình này có thể khá phức tạp, đặt ra các yêu cầu bổ sung cho cả hệ thống cron lẫn hạ tầng datacenter. Phần tiếp theo sẽ thảo luận cách xử lý các sự cố một phần kiểu này.

### Giải quyết Các Sự cố Một phần (Resolving partial failures)

Như đã đề cập, quá trình tương tác giữa replica leader và bộ lên lịch datacenter có thể gặp sự cố khi gửi nhiều RPC mô tả một lần khởi chạy job cron logic duy nhất. Hệ thống của chúng tôi cần xử lý được tình huống này.

Hãy nhớ rằng mỗi lần khởi chạy job cron có hai điểm đồng bộ hóa:

-   Khi chúng tôi sắp thực hiện lần khởi chạy
-   Khi chúng tôi đã hoàn thành lần khởi chạy

Hai điểm này giúp chúng tôi xác định giới hạn của lần khởi chạy. Ngay cả khi lần khởi chạy chỉ gồm một RPC duy nhất, làm thế nào chúng tôi biết liệu RPC đã thực sự được gửi hay chưa? Hãy xem xét trường hợp trong đó chúng tôi biết rằng lần khởi chạy đã lên lịch đã bắt đầu, nhưng chúng tôi không được thông báo về việc hoàn thành của nó trước khi replica leader chết.

Để xác định liệu RPC đã thực sự được gửi hay chưa, một trong các điều kiện sau phải được đáp ứng:

-   Mọi thao tác trên các hệ thống bên ngoài, mà chúng tôi có thể cần tiếp tục sau khi được bầu lại, phải có tính idempotent (tức là chúng tôi có thể an toàn thực hiện lại các thao tác đó)
-   Chúng tôi phải tra cứu được trạng thái của mọi thao tác trên các hệ thống bên ngoài để xác định rõ ràng chúng đã hoàn thành hay chưa

Mỗi điều kiện trong số này đều đặt ra ràng buộc đáng kể và có thể khó triển khai, nhưng đáp ứng được ít nhất một trong các điều kiện là nền tảng để dịch vụ cron hoạt động chính xác trong môi trường phân tán, nơi có thể xảy ra một hoặc nhiều sự cố một phần. Nếu không xử lý đúng cách, hệ thống có thể bỏ lỡ hoặc chạy trùng lặp cùng một job cron.

Hầu hết hạ tầng khởi chạy job logic trong datacenter (ví dụ, Mesos) đều hỗ trợ đặt tên cho các job, cho phép tra cứu trạng thái, dừng job hoặc thực hiện các thao tác bảo trì khác. Một giải pháp hợp lý cho vấn đề idempotent là xây dựng trước các tên job (để tránh gây ra bất kỳ thao tác thay đổi nào trên bộ lên lịch datacenter), sau đó phân phối các tên này đến tất cả các replica của dịch vụ cron. Nếu leader của dịch vụ cron chết trong khi đang khởi chạy, leader mới chỉ cần tra cứu trạng thái của tất cả các tên đã tính trước và khởi chạy những job còn thiếu.

Lưu ý rằng, tương tự như cách chúng tôi nhận dạng từng lần khởi chạy job cron riêng lẻ bằng tên và thời gian khởi chạy, các tên job được xây dựng trên bộ lên lịch datacenter phải bao gồm thời gian khởi chạy đã lên lịch cụ thể đó (hoặc có thông tin này có thể truy hồi được theo cách khác). Trong hoạt động thông thường, dịch vụ cron nên failover nhanh khi leader gặp sự cố, nhưng một failover nhanh không luôn xảy ra.

Hãy nhớ rằng chúng tôi theo dõi thời gian khởi chạy đã lên lịch khi giữ trạng thái nội bộ giữa các replica. Tương tự, chúng tôi cần phân biệt tương tác của mình với bộ lên lịch datacenter, cũng bằng cách sử dụng thời gian khởi chạy đã lên lịch. Ví dụ, hãy xem xét một job cron sống ngắn nhưng chạy thường xuyên. Job cron khởi chạy, nhưng trước khi lần khởi chạy được truyền đạt đến tất cả các replica, leader crash và một failover bất thường dài — đủ dài để job cron hoàn thành thành công — diễn ra. Leader mới tra cứu trạng thái của job cron, quan sát thấy nó hoàn thành, và cố gắng khởi chạy job đó lại. Nếu thời gian khởi chạy đã được bao gồm, leader mới sẽ biết rằng job trên bộ lên lịch datacenter là kết quả của lần khởi chạy job cron cụ thể này, và lần khởi chạy trùng lặp này đã không xảy ra.

Hệ thống tra cứu trạng thái trong thực tế thường phức tạp hơn, chịu ảnh hưởng bởi các chi tiết triển khai của hạ tầng nền tảng. Tuy nhiên, mô tả ở trên bao quát các yêu cầu độc lập với triển khai của bất kỳ hệ thống nào như vậy. Tùy thuộc vào hạ tầng khả dụng, bạn cũng có thể cần xem xét sự đánh đổi giữa việc chấp nhận rủi ro khởi chạy trùng lặp và rủi ro bỏ lỡ một lần khởi chạy.

## Lưu Trữ Trạng thái (Storing the State)

Việc dùng Paxos để đạt đồng thuận mới chỉ giải quyết một phần bài toán xử lý trạng thái. Về bản chất, Paxos là một log liên tục các thay đổi trạng thái, được ghi đồng bộ mỗi khi có thay đổi xảy ra. Đặc điểm này kéo theo hai hệ quả:

-   Log cần được nén (compact), để ngăn nó tăng trưởng vô hạn
-   Chính log phải được lưu ở đâu đó

Để ngăn log Paxos tăng trưởng vô hạn, chúng tôi có thể chụp snapshot của trạng thái hiện tại, tức là tái tạo trạng thái mà không cần phát lại toàn bộ các mục log thay đổi trạng thái dẫn đến trạng thái đó. Ví dụ, nếu các thay đổi trạng thái trong log là "Tăng một bộ đếm (counter) lên 1", thì sau một nghìn lần lặp lại, thay vì có một nghìn mục log, chúng tôi chỉ cần một snapshot "Đặt bộ đếm thành 1.000".

Khi mất log, chúng tôi chỉ mất trạng thái từ sau snapshot cuối cùng. Thực tế, snapshot mới là trạng thái quan trọng nhất — nếu mất snapshot, về cơ bản chúng tôi phải bắt đầu lại từ số không do đã mất toàn bộ trạng thái nội bộ. Ngược lại, việc mất log chỉ gây ra tổn thất trạng thái có giới hạn, khiến hệ thống cron quay ngược về thời điểm snapshot gần nhất được chụp.

Chúng tôi có hai lựa chọn chính để lưu trữ dữ liệu của mình:

-   Bên ngoài trong distributed storage nói chung khả dụng
-   Trong một hệ thống lưu trữ lượng trạng thái nhỏ như một phần của chính dịch vụ cron

Khi thiết kế hệ thống, chúng tôi kết hợp các yếu tố của cả hai lựa chọn.

Chúng tôi lưu log Paxos trên ổ đĩa cục bộ của các máy chạy replica dịch vụ cron. Vì cấu hình mặc định gồm ba replica, nên hệ thống có ba bản sao log. Các snapshot cũng được lưu trên ổ đĩa cục bộ. Tuy nhiên, do tính quan trọng của chúng, chúng tôi sao lưu thêm lên distributed filesystem để phòng ngừa trường hợp sự cố ảnh hưởng đồng thời đến cả ba máy.

Chúng tôi không lưu các log trên distributed filesystem của mình. Chúng tôi có chủ đích quyết định rằng việc mất các log, vốn đại diện cho một lượng nhỏ các thay đổi trạng thái gần nhất, là một rủi ro có thể chấp nhận được. Việc lưu log trên distributed filesystem có thể kéo theo một mức phạt hiệu năng đáng kể do các thao tác ghi nhỏ thường xuyên. Việc mất đồng thời cả ba máy là ít có khả năng, và nếu sự mất đồng thời thực sự xảy ra, chúng tôi tự động phục hồi từ snapshot. Bằng cách đó, chúng tôi chỉ mất một lượng nhỏ log: những cái được chụp kể từ snapshot cuối cùng, mà chúng tôi thực hiện theo các khoảng thời gian có thể cấu hình. Tất nhiên, các đánh đổi này có thể khác nhau tùy thuộc vào chi tiết của hạ tầng, cũng như các yêu cầu đặt ra cho hệ thống cron.

Ngoài các log và snapshot lưu trên ổ đĩa cục bộ cùng các bản sao lưu snapshot trên distributed filesystem, một replica vừa khởi động có thể lấy snapshot trạng thái và toàn bộ log từ một replica đang chạy qua mạng. Nhờ đó, quá trình khởi động replica không phụ thuộc vào bất kỳ trạng thái nào trên máy cục bộ. Vì vậy, việc lên lịch lại một replica sang máy khác khi khởi động lại (hoặc khi máy chết) về cơ bản không phải là vấn đề đối với độ tin cậy của dịch vụ.

## Vận hành Cron Lớn (Running Large Cron)

Việc vận hành một triển khai cron quy mô lớn cũng mang lại những hệ quả nhỏ hơn nhưng không kém phần thú vị. Một cron truyền thống thường rất nhỏ, có lẽ chỉ chứa vài chục job. Tuy nhiên, khi chạy dịch vụ cron cho hàng nghìn máy trong một datacenter, mức sử dụng sẽ tăng lên và bạn có thể gặp phải các vấn đề.

Cảnh giác với một vấn đề lớn và nổi tiếng của các hệ thống phân tán: hiệu ứng bầy đàn (thundering herd). Tùy theo cấu hình của người dùng, dịch vụ cron có thể tạo ra các đỉnh sử dụng datacenter đáng kể. Khi nghĩ về một "job cron hàng ngày", người dùng thường đặt lịch chạy vào lúc nửa đêm. Cách thiết lập này hoạt động tốt nếu job cron khởi chạy trên cùng một máy, nhưng điều gì xảy ra nếu job cron của bạn có thể sinh ra một MapReduce với hàng nghìn worker? Và điều gì nếu 30 nhóm khác nhau quyết định chạy một job cron hàng ngày như vậy, trong cùng một datacenter? Để giải quyết vấn đề này, chúng tôi đã giới thiệu một sự mở rộng của định dạng crontab.

Trong crontab thông thường, người dùng chỉ định phút, giờ, ngày trong tháng (hoặc tuần) và tháng để job cron khởi chạy, hoặc dùng dấu sao cho bất kỳ giá trị nào. Ví dụ, để chạy lúc nửa đêm hàng ngày, chỉ định crontab là `"0 0 * * *"` (tức là phút số không, giờ số không, mọi ngày trong tuần, mọi tháng, và mọi ngày trong tuần). Chúng tôi cũng giới thiệu việc sử dụng dấu hỏi, có nghĩa là bất kỳ giá trị nào đều được chấp nhận, và hệ thống cron được tự do chọn giá trị đó. Người dùng chọn giá trị này bằng cách băm cấu hình job cron trên khoảng thời gian được cho (ví dụ, `0..23` cho giờ), do đó phân phối các lần khởi chạy đó đều hơn.

Dù đã có thay đổi đó, tải do các job cron tạo ra vẫn rất gập ghềnh. Đồ thị trong [Hình 24-3](#hinh-24-3) cho thấy tổng số lần khởi chạy job cron trên toàn cầu tại Google. Đồ thị này cho thấy các đỉnh thường xuyên xuất hiện khi khởi chạy job cron, chủ yếu là do các job cron cần được chạy vào một thời điểm cụ thể — chẳng hạn, vì chúng phụ thuộc theo thời gian vào các sự kiện bên ngoài.

<a id="hinh-24-3"></a>        ![Số lượng job cron được khởi chạy trên toàn cầu.](../assets/imgs/fig-24-3.jpg)

Hình 24-3. Số lượng job cron được khởi chạy trên toàn cầu

## Tóm tắt (Summary)

Dịch vụ cron đã là tính năng cơ bản của các hệ thống UNIX suốt nhiều thập kỷ. Khi ngành công nghiệp chuyển dịch sang các hệ thống phân tán quy mô lớn, nơi datacenter có thể là đơn vị phần cứng hiệu dụng nhỏ nhất, phần lớn các lớp trong stack đều phải thay đổi. Cron cũng không nằm ngoài xu hướng này. Việc xem xét kỹ lưỡng các thuộc tính cần thiết của một dịch vụ cron cùng những yêu cầu đặt ra cho các job cron đã định hình thiết kế mới của Google.

Chúng tôi đã bàn về những ràng buộc mới do môi trường hệ thống phân tán đặt ra, cùng một phương án thiết kế cho dịch vụ cron dựa trên giải pháp của Google. Giải pháp này đòi hỏi các cam kết đảm bảo nhất quán mạnh trong môi trường phân tán. Vì vậy, cốt lõi của việc triển khai cron phân tán là Paxos, một thuật toán phổ biến để đạt được đồng thuận trong môi trường thiếu tin cậy. Việc sử dụng Paxos và phân tích chính xác các chế độ lỗi mới của các job cron trong môi trường phân tán quy mô lớn đã giúp chúng tôi xây dựng một dịch vụ cron mạnh mẽ, được sử dụng rộng rãi tại Google.

<a id="fn1"></a>[1](#fn1) Chương này trước đây đã được xuất bản một phần trong *ACM Queue* (tháng Ba 2015, vol. 13, số 3).

<a id="fn2"></a>[2](#fn2) Sự cố của từng job riêng lẻ nằm ngoài phạm vi của phân tích này.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
