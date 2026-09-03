> **Nguyên bản:** [Chapter 26 - Data Integrity: What You Read Is What You Wrote](https://sre.google/sre-book/data-integrity/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

## Chương 26. Tính Toàn Vẹn Dữ Liệu: Điều Bạn Đọc Chính Là Điều Bạn Đã Ghi (Data Integrity: What You Read Is What You Wrote)

Tác giả: Raymond Blum và Rhandeev Singh  
Biên tập: Betsy Beyer

"Data integrity" (tính toàn vẹn dữ liệu) là gì? Khi đặt người dùng lên hàng đầu, data integrity chính là bất cứ điều gì người dùng cho rằng nó là.

Chúng ta có thể nói *data integrity là thước đo mức độ truy cập được và độ chính xác của các datastore cần thiết để cung cấp cho người dùng một mức độ dịch vụ thỏa đáng*. Nhưng định nghĩa này là chưa đủ.

Ví dụ, nếu một bug trong giao diện người dùng của Gmail hiển thị một hộp thư trống trong thời gian quá lâu, người dùng có thể tin rằng dữ liệu đã bị mất. Do đó, ngay cả khi không có dữ liệu nào *thực sự* bị mất, cả thế giới sẽ chất vấn khả năng của Google trong việc đóng vai trò là người quản trị dữ liệu có trách nhiệm, và tính khả thi của cloud computing sẽ bị đe dọa. Nếu Gmail hiển thị một thông báo lỗi hoặc bảo trì trong thời gian quá lâu trong khi "chỉ một chút metadata" đang được sửa chữa, sự tin tưởng của người dùng Google cũng sẽ bị xói mòn tương tự.

Data không khả dụng trong bao lâu thì được coi là "quá lâu"? Như minh chứng từ một incident thực tế của Gmail năm 2011 [[Hic11]](https://sre.google/sre-book/bibliography#Hic11), bốn ngày là một khoảng thời gian dài — có lẽ là "quá lâu." Tiếp đó, chúng tôi cho rằng 24 giờ là một điểm khởi đầu tốt để thiết lập ngưỡng "quá lâu" cho Google Apps.

Lý luận tương tự áp dụng cho các ứng dụng như Google Photos, Drive, Cloud Storage, và Cloud Datastore, bởi vì người dùng không nhất thiết phân biệt giữa những sản phẩm rời rạc này (họ lý luận rằng "sản phẩm này vẫn là Google" hay "Google, Amazon, hay cái gì đi nữa; sản phẩm này vẫn là một phần của cloud"). Data loss, data corruption, và không khả dụng kéo dài thường không thể phân biệt được đối với người dùng. Do đó, data integrity áp dụng cho mọi loại dữ liệu trên tất cả các dịch vụ. Khi xem xét data integrity, điều quan trọng là *các dịch vụ trên cloud vẫn khả dụng cho người dùng. Quyền truy cập dữ liệu của người dùng đặc biệt quan trọng*.

## Yêu Cầu Nghiêm Khắc của Data Integrity (Data Integrity's Strict Requirements)

Khi xem xét nhu cầu độ tin cậy của một hệ thống nhất định, có thể có vẻ như nhu cầu về uptime là nghiêm khắc hơn so với data integrity. Ví dụ, người dùng có thể thấy một giờ email downtime là không thể chấp nhận được, trong khi họ có thể chấp nhận một cách càu nhàu với một khung thời gian bốn ngày để khôi phục hộp thư. Tuy nhiên, có một cách xem xét phù hợp hơn về yêu cầu của uptime so với data integrity.

Một SLO (service level objective) uptime 99,99% chỉ để lại khoảng trống cho một giờ downtime trong suốt cả năm. SLO này đặt ra một tiêu chuẩn khá cao, nhiều khả năng vượt quá kỳ vọng của phần lớn người dùng Internet và Enterprise.

Ngược lại, một SLO 99,99% byte tốt trong một artifact 2 GB sẽ làm hỏng tài liệu, file thực thi, và cơ sở dữ liệu (lên đến 200 KB bị hỏng). Mức độ corruption này là *thảm khốc* trong phần lớn các trường hợp — dẫn đến các file thực thi có opcode ngẫu nhiên và các cơ sở dữ liệu hoàn toàn không thể nạp được.

Từ góc nhìn của người dùng, mỗi dịch vụ có các yêu cầu độc lập về uptime và data integrity, ngay cả khi những yêu cầu này là ngầm hiểu. Thời điểm tồi tệ nhất để bất đồng với người dùng về những yêu cầu này là sau khi dữ liệu của họ đã bị tiêu diệt!

![srle 26in01](../assets/imgs/fig-26-1.jpg)

Hình 26-1.

Để sửa đổi định nghĩa trước đó của chúng ta về data integrity, chúng ta có thể nói rằng *data integrity có nghĩa là các dịch vụ trên cloud vẫn khả dụng cho người dùng. Quyền truy cập dữ liệu của người dùng đặc biệt quan trọng, nên quyền truy cập này cần được duy trì trong tình trạng hoàn hảo*.

Giả sử bây giờ một artifact bị corruption hoặc mất đi đúng một lần trong một năm. Nếu sự mất mát không thể khôi phục được, uptime của artifact bị ảnh hưởng đã *bị mất* trong năm đó. Cách có khả năng nhất để tránh bất kỳ sự mất mát như vậy là thông qua việc phát hiện chủ động, kết hợp với việc sửa chữa nhanh chóng.

Trong một vũ trụ khác, giả sử sự corruption được phát hiện ngay lập tức trước khi người dùng bị ảnh hưởng và artifact đã được gỡ bỏ, sửa chữa, và đưa trở lại dịch vụ trong vòng nửa giờ. Bỏ qua bất kỳ downtime nào khác trong 30 phút đó, một object như vậy sẽ khả dụng 99,99% trong năm đó.

Đáng kinh ngạc, ít nhất từ góc nhìn của người dùng, trong kịch bản này, data integrity vẫn là 100% (hoặc gần 100%) trong vòng đời khả dụng của object. Như được minh chứng bằng ví dụ này, *bí quyết để có data integrity vượt trội là phát hiện chủ động và sửa chữa, khôi phục nhanh chóng.*

## Lựa Chọn Chiến Lược cho Data Integrity Vượt Trội (Choosing a Strategy for Superior Data Integrity)

Có nhiều chiến lược khả thi để phát hiện, sửa chữa, và khôi phục nhanh dữ liệu bị mất. Tất cả những chiến lược này đều đánh đổi uptime so với data integrity đối với những người dùng bị ảnh hưởng. Một số chiến lược hiệu quả hơn số khác, và một số chiến lược đòi hỏi đầu tư kỹ thuật phức tạp hơn. Với nhiều tùy chọn sẵn có như vậy, bạn nên sử dụng chiến lược nào? Câu trả lời phụ thuộc vào hệ thống tính toán (paradigm) của bạn.

Phần lớn các ứng dụng cloud computing tìm cách tối ưu hóa cho một tổ hợp nào đó của uptime, latency, scale, velocity, và privacy. Để đưa ra một định nghĩa vận hành cho mỗi thuật ngữ này:

Uptime

Còn được gọi là *availability*, tỷ lệ thời gian mà một dịch vụ khả dụng cho người dùng của nó.

Latency

Mức độ một dịch vụ phản hồi nhanh như thế nào trong mắt người dùng của nó.

Scale

Số lượng người dùng của một dịch vụ và sự pha trộn các khối lượng công việc mà dịch vụ có thể xử lý trước khi latency bị ảnh hưởng hoặc dịch vụ sụp đổ.

Velocity

Một dịch vụ có thể đổi mới nhanh đến mức nào để cung cấp cho người dùng giá trị vượt trội với chi phí hợp lý.

Privacy

Khái niệm này đặt ra các yêu cầu phức tạp. Để đơn giản hóa, chương này giới hạn phạm vi thảo luận về privacy trong phạm vi xóa dữ liệu: dữ liệu phải bị phá hủy trong một thời gian hợp lý sau khi người dùng xóa nó.

Nhiều ứng dụng cloud không ngừng tiến hóa trên nền một hỗn hợp của các API ACID<sup>[1](#fn1)</sup> và BASE<sup>[2](#fn2)</sup> để đáp ứng yêu cầu của năm thành phần này.<sup>[3](#fn3)</sup> BASE cho phép khả năng hoạt động cao hơn so với ACID, đổi lại là một bảo đảm nhất quán phân tán mềm hơn. Cụ thể, BASE chỉ bảo đảm rằng một khi một phần dữ liệu không còn được cập nhật nữa, giá trị của nó sẽ *cuối cùng* trở nên nhất quán trên các vị trí lưu trữ (có thể phân tán).

Kịch bản sau đây cung cấp một ví dụ về cách các đánh đổi giữa uptime, latency, scale, velocity, và privacy có thể diễn ra.

Khi velocity vượt trội hơn các yêu cầu khác, các ứng dụng kết quả dựa vào một tập hợp các API tùy ý mà các lập trình viên nhất định đang làm việc trên ứng dụng đó quen thuộc nhất.

Ví dụ, một ứng dụng có thể tận dụng một API lưu trữ BLOB (Binary Large Object)<sup>[4](#fn4)</sup> hiệu quả, chẳng hạn như Blobstore, thứ xem nhẹ sự nhất quán phân tán để đổi lấy việc mở rộng scale cho các khối lượng công việc nặng với uptime cao, latency thấp và với chi phí thấp. Để bù đắp:

-   Cùng ứng dụng đó có thể giao phó một lượng nhỏ metadata quan trọng (authoritative) liên quan đến các blob của nó cho một dịch vụ dựa trên Paxos có latency cao hơn, khả năng hoạt động thấp hơn, đắt đỏ hơn như Megastore [[Bak11]](https://sre.google/sre-book/bibliography#Bak11), [[Lam98]](https://sre.google/sre-book/bibliography#Lam98).
-   Một số client của ứng dụng có thể cache một số metadata đó cục bộ và truy cập các blob trực tiếp, giảm latency thêm nữa từ góc nhìn của người dùng.
-   Một ứng dụng khác có thể giữ metadata trong Bigtable, hy sinh sự nhất quán phân tán mạnh vì các lập trình viên của nó tình cờ quen thuộc với Bigtable.

Những ứng dụng cloud như vậy đối mặt với nhiều thách thức data integrity lúc chạy, chẳng hạn như tính toàn vẹn tham chiếu (referential integrity) giữa các datastore (trong ví dụ trước, Blobstore, Megastore, và cache phía client). Sự biến đổi bất thường của velocity cao quy định rằng những thay đổi schema, các cuộc di chuyển dữ liệu, việc chất chồng tính năng mới lên trên tính năng cũ, các cuộc viết lại, và các điểm tích hợp đang phát triển với các ứng dụng khác đồng âm mưu tạo ra một môi trường tràn ngập những mối quan hệ phức tạp giữa các mảnh dữ liệu khác nhau mà không một lập trình viên nào hiểu trọn vẹn.

Để ngăn dữ liệu của một ứng dụng như vậy bị suy thoái trước mắt người dùng của nó, cần một hệ thống kiểm tra và cân bằng ngoài vòng (out-of-band) bên trong và giữa các datastore của nó. [Tầng thứ ba: Phát hiện sớm](#tang-thu-ba-phat-hien-som) thảo luận về một hệ thống như vậy.

Ngoài ra, nếu một ứng dụng như vậy dựa vào các bản sao lưu độc lập, không phối hợp của một vài datastore (trong ví dụ trước, Blobstore và Megastore), thì khả năng khai thác hiệu quả dữ liệu đã khôi phục trong một nỗ lực khôi phục dữ liệu bị làm phức tạp bởi sự đa dạng của các mối quan hệ giữa dữ liệu đã khôi phục và dữ liệu trực tiếp. Ứng dụng ví dụ của chúng ta sẽ phải sàng lọc và phân biệt giữa các blob đã khôi phục so với Megastore trực tiếp, Megastore đã khôi phục so với các blob trực tiếp, các blob đã khôi phục so với Megastore đã khôi phục, và các tương tác với cache phía client.

Với sự xem xét đến những sự phụ thuộc và những điều phức tạp này, bao nhiêu nguồn lực nên được đầu tư vào các nỗ lực data integrity, và ở đâu?

## Backup Đối Chọi Với Lưu Trữ (Backups Versus Archives)

Theo truyền thống, các công ty "bảo vệ" dữ liệu khỏi việc mất mát bằng cách đầu tư vào các chiến lược backup. Tuy nhiên, trọng tâm thực sự của những nỗ lực backup như vậy nên là khôi phục dữ liệu, điều mà phân biệt các bản *backup thật* khỏi các kho lưu trữ (archive). Như đôi khi được quan sát: Không ai thực sự *muốn* tạo backup; điều mà mọi người *thực sự* muốn là *khôi phục* (restore).

"Backup" của bạn có thực sự là một kho lưu trữ, chứ không phải phù hợp để sử dụng trong khôi phục thảm họa?

![srle 26in02](../assets/imgs/fig-26-2.jpg)

Hình 26-2.

Sự khác biệt quan trọng nhất giữa backup và kho lưu trữ là backup *có thể* được nạp lại vào một ứng dụng, trong khi kho lưu trữ *không thể*. Do đó, backup và kho lưu trữ có các trường hợp sử dụng khác biệt khá lớn.

*Kho lưu trữ* an toàn giữ dữ liệu trong thời gian dài để đáp ứng các nhu cầu kiểm toán, truy tìm, và tuân thủ. Khôi phục dữ liệu cho các mục đích như vậy thường không cần phải hoàn thành trong các yêu cầu uptime của một dịch vụ. Ví dụ, bạn có thể cần giữ lại dữ liệu giao dịch tài chính trong bảy năm. Để đạt được mục tiêu này, bạn có thể di chuyển các log kiểm toán tích lũy đến kho lưu trữ dài hạn tại một vị trí ngoài địa điểm (offsite) một lần một tháng. Việc truy xuất và khôi phục các log trong suốt một cuộc kiểm toán tài chính kéo dài một tháng có thể mất một tuần, và khung thời gian khôi phục một tuần này có thể chấp nhận được đối với một kho lưu trữ.

Mặt khác, khi thảm họa ập đến, dữ liệu phải được khôi phục từ *các bản backup thật* một cách nhanh chóng, tốt nhất là hoàn toàn nằm trong các nhu cầu uptime của một dịch vụ. Nếu không, những người dùng bị ảnh hưởng sẽ bị bỏ lại mà không có quyền truy cập hữu ích vào ứng dụng từ khi vấn đề data integrity bắt đầu cho đến khi nỗ lực khôi phục hoàn tất.

Cũng quan trọng cần xem xét rằng vì dữ liệu mới nhất có nguy cơ cho đến khi được sao lưu an toàn, có thể là tối ưu để lên lịch cho các bản backup thật (đối với kho lưu trữ) diễn ra hàng ngày, hàng giờ, hoặc thường xuyên hơn, sử dụng các cách tiếp cận full và incremental hoặc continuous (liên tục, dạng streaming).

Do đó, khi xây dựng một chiến lược backup, hãy cân nhắc xem bạn cần phải có khả năng khôi phục từ một vấn đề nhanh đến mức nào, và bạn có thể chấp nhận mất bao nhiêu dữ liệu gần đây.

## Yêu Cầu của Môi Trường Cloud Trong Bối Cảnh (Requirements of the Cloud Environment in Perspective)

Các môi trường cloud đưa đến một tổ hợp các thách thức kỹ thuật độc đáo:

-   Nếu môi trường sử dụng một hỗn hợp của các giải pháp backup và khôi phục có giao dịch và không có giao dịch, dữ liệu được khôi phục không nhất thiết sẽ đúng.
-   Nếu các dịch vụ phải tiến hóa mà không ngừng hoạt động để bảo trì, các phiên bản khác nhau của logic kinh doanh có thể hoạt động song song trên dữ liệu.
-   Nếu các dịch vụ tương tác được phân phiên bản độc lập, các phiên bản không tương thích của các dịch vụ khác nhau có thể tương tác trong khoảnh khắc, làm tăng thêm khả năng data corruption hoặc data loss ngẫu nhiên.

Ngoài ra, để duy trì kinh tế theo quy mô, các nhà cung cấp dịch vụ chỉ có thể cung cấp một số lượng giới hạn các API. Những API này phải đơn giản và dễ sử dụng cho phần lớn các ứng dụng, nếu không sẽ ít khách hàng sử dụng chúng. Đồng thời, các API phải đủ mạnh mẽ để hiểu được những điều sau:

-   Tính cục bộ của dữ liệu và caching
-   Phân phối dữ liệu cục bộ và toàn cục
-   Nhất quán mạnh và/hoặc nhất quán cuối cùng (eventual consistency)
-   Độ bền dữ liệu, backup, và khôi phục

Nếu không, những khách hàng phức tạp không thể di chuyển các ứng dụng của họ lên cloud, và các ứng dụng đơn giản trở nên phức tạp và lớn sẽ cần được viết lại hoàn toàn để sử dụng các API khác, phức tạp hơn.

Những vấn đề nảy sinh khi các tính năng API được đề cập ở trên được sử dụng trong một số tổ hợp nhất định. Nếu nhà cung cấp dịch vụ không giải quyết những vấn đề này, thì các ứng dụng phải đối mặt với những thách thức này phải tự xác định và giải quyết chúng một cách độc lập.

## Mục Tiêu của Google SRE Trong Việc Duy Trì Data Integrity và Khả Năng Hoạt Động (Google SRE Objectives in Maintaining Data Integrity and Availability)

Trong khi mục tiêu của SRE về "duy trì tính toàn vẹn của dữ liệu lưu trữ lâu dài" là một tầm nhìn tốt, chúng tôi phát triển mạnh dựa trên những mục tiêu cụ thể với các chỉ số có thể đo lường được. SRE xác định các metric chính mà chúng tôi sử dụng để thiết lập kỳ vọng về khả năng của hệ thống và quy trình của chúng tôi thông qua các bài kiểm thử, và để theo dõi hiệu suất của chúng trong một sự kiện thực tế.

### Data Integrity Là Phương Tiện; Data Availability Là Mục Tiêu (Data Integrity Is the Means; Data Availability Is the Goal)

Data integrity đề cập đến độ chính xác và tính nhất quán của dữ liệu trong suốt vòng đời của nó. Người dùng cần biết rằng thông tin sẽ đúng và không thay đổi theo một cách bất ngờ nào đó từ thời điểm nó được ghi lại lần đầu tiên cho đến lần quan sát cuối cùng. Nhưng sự đảm bảo như vậy có đủ không?

Hãy xem xét trường hợp của một nhà cung cấp email đã phải chịu một đợt data outage kéo dài một tuần [[Kinc09]](https://sre.google/sre-book/bibliography#Kinc09). Trong khoảng 10 ngày, người dùng phải tìm các phương pháp tạm thời khác để tiến hành công việc của họ, với kỳ vọng rằng họ sẽ sớm quay trở lại các tài khoản email, danh tính, và lịch sử tích lũy đã được thiết lập của mình.

Rồi, tin tức tồi tệ nhất đã đến: nhà cung cấp tuyên bố rằng bất chấp những kỳ vọng trước đó, kho email và danh bạ trong quá khứ thực chất đã biến mất — bốc hơi và không bao giờ được nhìn thấy nữa. Có vẻ như một loạt những sơ suất trong việc quản lý data integrity đã đồng âm mưu để lại cho nhà cung cấp dịch vụ không có bản backup nào có thể sử dụng. Những người dùng phẫn nộ hoặc bám víu vào danh tính tạm thời của họ hoặc thiết lập các danh tính mới, bỏ rơi nhà cung cấp email đầy rắc rối trước đó của họ.

Nhưng khoan đã! Vài ngày sau tuyên bố mất mát tuyệt đối, nhà cung cấp tuyên bố rằng thông tin cá nhân của người dùng *có thể* được khôi phục. Không có data loss; đây chỉ là một đợt outage. Mọi thứ đều ổn!

Trừ phi, *mọi thứ không hề ổn*. Dữ liệu người dùng đã được bảo toàn, nhưng dữ liệu không thể truy cập được bởi những người cần nó trong một thời gian quá lâu.

Bài học từ ví dụ này: Từ góc nhìn của người dùng, data integrity mà không có data availability như mong đợi và thường xuyên thì về cơ bản giống như không có dữ liệu nào cả.

### Giao Một Hệ Thống Khôi Phục, Thay Vì Một Hệ Thống Backup (Delivering a Recovery System, Rather Than a Backup System)

Việc tạo backup là một nhiệm vụ bị bỏ bê kinh điển, được ủy thác, và bị trì hoãn trong quản trị hệ thống. Backup không phải là ưu tiên cao của bất kỳ ai — đó là một sự tiêu hao liên tục về thời gian và nguồn lực, và không mang lại lợi ích tức thì nào nhìn thấy được. Vì lý do này, việc thiếu tận tâm trong triển khai chiến lược backup thường chỉ nhận được một cái nhún vai thông cảm. Người ta có thể lập luận rằng, giống như hầu hết các biện pháp bảo vệ khỏi những nguy hiểm rủi ro thấp, thái độ như vậy là thực dụng. Vấn đề cơ bản với chiến lược cẩu thả này là những nguy hiểm mà nó kéo theo có thể là rủi ro thấp, nhưng chúng cũng có tác động cao. Khi dữ liệu của dịch vụ bạn không khả dụng, phản ứng của bạn có thể quyết định sự sống còn của dịch vụ, sản phẩm, và thậm chí công ty bạn.

Thay vì tập trung vào công việc ít được cảm ơn là tạo backup, nó hữu ích hơn nhiều, không nói thì cũng dễ dàng hơn, để tạo động lực tham gia vào việc tạo backup bằng cách tập trung vào một nhiệm vụ có phần thưởng nhìn thấy được: *khôi phục* (restore)! *Backup là một loại thuế*, được trả liên tục cho dịch vụ cộng đồng là data availability được đảm bảo. Thay vì nhấn mạnh vào thuế, hãy thu hút sự chú ý vào dịch vụ mà thuế tài trợ: data availability. Chúng tôi không bắt các đội "luyện tập" backup của họ, thay vào đó:

-   Các đội xác định các SLO cho data availability trong nhiều chế độ lỗi khác nhau.
-   Một đội luyện tập và chứng minh khả năng đáp ứng những SLO đó.

### Các Loại Lỗi Dẫn Đến Data Loss (Types of Failures That Lead to Data Loss)

Như được minh họa bằng [Hình 26-3](#hinh-26-3-cac-nhan-to-cua-cham-che-loi-data-integrity), ở mức rất cao, có 24 loại lỗi riêng biệt khi 3 nhân tố có thể xảy ra theo bất kỳ tổ hợp nào. Bạn nên xem xét từng lỗi tiềm năng này khi thiết kế một chương trình data integrity. Các nhân tố của các chế độ lỗi data integrity như sau:

Nguyên nhân gốc

Một sự mất mát dữ liệu không thể khôi phục có thể do một số nhân tố: hành động của người dùng, lỗi vận hành, bug ứng dụng, khiếm khuyết trong hạ tầng, phần cứng lỗi, hoặc thảm họa tại địa điểm.

Phạm vi

Một số sự mất mát là phổ biến, ảnh hưởng đến nhiều thực thể. Một số sự mất mát là hẹp và có chủ đích, xóa hoặc làm hỏng dữ liệu đặc thù cho một nhóm nhỏ người dùng.

Tốc độ

Một số data loss là một sự kiện bùng nổ (ví dụ, 1 triệu dòng được thay thế bằng chỉ 10 dòng trong một phút), trong khi một số data loss là leo dần (ví dụ, 10 dòng dữ liệu bị xóa mỗi phút trong suốt nhiều tuần).

<a id="hinh-26-3-cac-nhan-to-cua-cham-che-loi-data-integrity"></a>        ![The factors of data integrity failure modes.](../assets/imgs/fig-26-3.jpg)

Hình 26-3. Các nhân tố của các chế độ lỗi data integrity

Một kế hoạch khôi phục hiệu quả phải tính đến bất kỳ chế độ lỗi nào trong số này xảy ra theo bất kỳ tổ hợp nào có thể hình dung được. Một chiến lược hoàn toàn hiệu quả để phòng chống một data loss do một bug ứng dụng leo dần gây ra có thể không giúp ích gì cả khi trung tâm dữ liệu colocation của bạn bị cháy.

Một nghiên cứu về 19 nỗ lực khôi phục dữ liệu tại Google phát hiện rằng các kịch bản data loss nhìn thấy được bởi người dùng phổ biến nhất liên quan đến việc xóa dữ liệu hoặc mất tính toàn vẹn tham chiếu do bug phần mềm gây ra. Các biến thể thách thức nhất liên quan đến corruption hoặc xóa ở mức độ thấp được phát hiện từ vài tuần đến vài tháng sau khi các bug lần đầu tiên được phát hành vào môi trường production. Do đó, các biện pháp bảo vệ mà Google sử dụng nên phù hợp tốt để ngăn chặn hoặc khôi phục khỏi những loại mất mát này.

Để khôi phục từ những kịch bản như vậy, một ứng dụng lớn và thành công cần truy xuất dữ liệu cho có thể là hàng triệu người dùng trải dài trong nhiều ngày, tuần, hoặc tháng. Ứng dụng cũng có thể cần khôi phục mỗi artifact bị ảnh hưởng đến một thời điểm duy nhất. Kịch bản khôi phục dữ liệu này được gọi là "point-in-time recovery" bên ngoài Google, và "time-travel" bên trong Google.

Một giải pháp backup và khôi phục cung cấp point-in-time recovery cho một ứng dụng xuyên suốt các datastore ACID và BASE của nó trong khi đáp ứng các mục tiêu uptime, latency, khả năng mở rộng, velocity, và chi phí nghiêm ngặt là một loài huyền thoại (chimera) ngày nay!

Giải quyết vấn đề này bằng chính các lập trình viên của bạn liên quan đến việc hy sinh velocity. Nhiều dự án thỏa hiệp bằng cách áp dụng một chiến lược backup theo tầng mà không có point-in-time recovery. Ví dụ, các API bên dưới ứng dụng của bạn có thể hỗ trợ nhiều cơ chế khôi phục dữ liệu. Các snapshot cục bộ đắt đỏ có thể cung cấp sự bảo vệ hạn chế khỏi bug ứng dụng và cung cấp chức năng khôi phục nhanh, nên bạn có thể giữ lại vài ngày của những snapshot cục bộ như vậy, được chụp cách nhau vài giờ. Các bản sao full và incremental hiệu quả về chi phí mỗi hai ngày có thể được giữ lâu hơn. Point-in-time recovery là một tính năng rất đáng có nếu một hoặc nhiều hơn những chiến lược này hỗ trợ nó.

Hãy cân nhắc các tùy chọn khôi phục dữ liệu được cung cấp bởi các API cloud mà bạn sắp sử dụng. Hãy đánh đổi point-in-time recovery với một chiến lược theo tầng nếu cần thiết, nhưng đừng hạ xuống việc không sử dụng cái nào cả! Nếu bạn có thể có cả hai tính năng, hãy sử dụng cả hai tính năng. Mỗi tính năng trong số này (hoặc cả hai) sẽ có giá trị vào một thời điểm nào đó.

### Thách Thức Trong Việc Duy Trì Data Integrity Sâu và Rộng (Challenges of Maintaining Data Integrity Deep and Wide)

Trong việc thiết kế một chương trình data integrity, điều quan trọng là phải nhận ra rằng *replication và redundancy không phải là recoverability (khả năng khôi phục)*.

#### Vấn đề mở rộng: Full, incremental, và các lực lượng cạnh tranh giữa backup và restore (Scaling issues: Fulls, incrementals, and the competing forces of backups and restores)

Một phản hồi kinh điển nhưng thiếu sót cho câu hỏi "Bạn có backup không?" là "Chúng tôi có thứ thậm chí còn tốt hơn backup — replication!" Replication mang lại nhiều lợi ích, bao gồm tính cục bộ của dữ liệu và bảo vệ khỏi một thảm họa riêng biệt tại địa điểm, nhưng nó không thể bảo vệ bạn khỏi nhiều nguồn data loss. Các datastore tự động đồng bộ hóa nhiều bản replica đảm bảo rằng một hàng cơ sở dữ liệu bị hỏng hoặc một lệnh xóa nhầm sẽ được đẩy đến tất cả các bản sao của bạn, có khả năng trước khi bạn có thể cô lập vấn đề.

Để giải quyết mối lo ngại này, bạn có thể tạo các bản sao không phục vụ dữ liệu của bạn ở một định dạng khác, chẳng hạn như xuất cơ sở dữ liệu thường xuyên ra một file bản địa. Biện pháp bổ sung này thêm sự bảo vệ khỏi các loại lỗi mà replication không bảo vệ — lỗi người dùng và bug ở tầng ứng dụng — nhưng không làm gì để phòng chống những mất mát được đưa vào ở một tầng thấp hơn. Biện pháp này cũng đưa đến nguy cơ có bug trong quá trình chuyển đổi dữ liệu (theo cả hai chiều) và trong quá trình lưu trữ file bản địa, ngoài khả năng không khớp về ngữ nghĩa giữa hai định dạng. Hãy tưởng tượng một cuộc tấn công zero-day<sup>[5](#fn5)</sup> ở một mức thấp trong stack của bạn, chẳng hạn như filesystem hoặc device driver. Bất kỳ bản sao nào dựa vào thành phần phần mềm bị xâm phạm, bao gồm các bản xuất cơ sở dữ liệu được ghi ra cùng filesystem làm nền cho cơ sở dữ liệu của bạn, đều dễ bị tổn thương.

Như vậy, chúng ta thấy rằng sự đa dạng là chìa khóa: bảo vệ khỏi một sự cố ở tầng X đòi hỏi phải lưu trữ dữ liệu trên các thành phần đa dạng ở tầng đó. Sự cô lập phương tiện truyền thông bảo vệ khỏi các khiếm khuyết của phương tiện: một bug hoặc cuộc tấn công trong một device driver ổ đĩa khó có thể ảnh hưởng đến các ổ băng. Nếu có thể, chúng tôi sẽ tạo các bản backup dữ liệu có giá trị của mình trên các phiến đất nung.<sup>[6](#fn6)</sup>

Nhu cầu về độ mới của dữ liệu và tốc độ khôi phục thường mâu thuẫn với nhu cầu bảo vệ toàn diện. Càng đẩy một snapshot của dữ liệu bạn xuống sâu trong stack, việc tạo bản sao mất càng lâu, điều đó có nghĩa là tần suất các bản sao giảm đi. Ở mức cơ sở dữ liệu, một giao dịch có thể mất theo đơn vị giây để nhân bản. Xuất một snapshot cơ sở dữ liệu ra filesystem bên dưới có thể mất 40 phút. Một bản backup full của filesystem bên dưới có thể mất hàng giờ.

Trong kịch bản này, bạn có thể mất đến 40 phút dữ liệu mới nhất khi bạn khôi phục snapshot mới nhất. Một lần khôi phục từ backup filesystem có thể gây ra hàng giờ giao dịch bị thiếu. Ngoài ra, việc khôi phục có thể mất lâu như việc backup, nên việc thực sự nạp dữ liệu có thể mất hàng giờ. Rõ ràng bạn muốn có dữ liệu tươi nhất trở lại càng nhanh càng tốt, nhưng tùy thuộc vào loại lỗi, bản sao tươi nhất và khả dụng ngay lập tức đó có thể không phải là một lựa chọn.

#### Thời gian giữ (Retention)

Thời gian giữ — bạn giữ các bản sao dữ liệu của mình trong bao lâu — là một nhân tố khác cần cân nhắc trong kế hoạch khôi phục dữ liệu của bạn.

Trong khi có khả năng là bạn hoặc khách hàng của bạn sẽ nhanh chóng nhận thấy việc một cơ sở dữ liệu bị đổ trống đột ngột, một sự mất mát dữ liệu dần dần hơn có thể mất vài ngày để thu hút sự chú ý của người phù hợp. Khôi phục dữ liệu bị mất trong kịch bản sau đòi hỏi các snapshot được chụp từ xa hơn trong quá khứ. Khi đạt đến xa như vậy, bạn có thể muốn gộp dữ liệu đã khôi phục với trạng thái hiện tại. Việc làm như vậy làm phức tạp đáng kể quá trình khôi phục.

## Cách Google SRE Đối Mặt Với Các Thách Thức của Data Integrity (How Google SRE Faces the Challenges of Data Integrity)

Giống như giả định của chúng tôi rằng các hệ thống nền tảng của Google dễ xảy ra lỗi, chúng tôi giả định rằng bất kỳ cơ chế bảo vệ nào của chúng tôi cũng chịu các lực lượng tương tự và có thể hỏng theo những cách giống nhau và vào những thời điểm bất tiện nhất. Duy trì một bảo đảm về data integrity ở quy mô lớn, một thách thức còn được làm phức tạp thêm bởi tốc độ thay đổi cao của các hệ thống phần mềm liên quan, đòi hỏi một số thực hành bổ sung nhưng độc lập, mỗi cái được chọn để đơn lẻ mang lại một mức độ bảo vệ cao.

### 24 Tổ Hợp của Các Chế Độ Lỗi Data Integrity (The 24 Combinations of Data Integrity Failure Modes)

Với nhiều cách dữ liệu có thể bị mất (như đã mô tả trước đó), không có một giải pháp vạn năng nào bảo vệ khỏi nhiều tổ hợp của các chế độ lỗi. Thay vào đó, bạn cần phòng thủ nhiều lớp (defense in depth). Phòng thủ nhiều lớp bao gồm nhiều tầng, với mỗi tầng phòng thủ tiếp theo mang lại sự bảo vệ khỏi các kịch bản data loss ít phổ biến dần. [Hình 26-4](#hinh-26-4-hanh-trinh-cua-mot-object-tu-xoa-mem-den-huy-hoai) minh họa hành trình của một object từ xóa mềm đến hủy hoại, và các chiến lược khôi phục dữ liệu nên được triển khai dọc theo hành trình này để đảm bảo phòng thủ nhiều lớp.

Tầng đầu tiên là *xóa mềm* (hoặc "lazy deletion" trong trường hợp cung cấp API cho lập trình viên), đã chứng minh là một phòng thủ hiệu quả đối với các kịch bản xóa dữ liệu vô tình. Phòng tuyến thứ hai là *backup và các phương pháp khôi phục liên quan*. Tầng thứ ba và cuối cùng là *kiểm tra dữ liệu định kỳ*, được đề cập trong [Tầng thứ ba: Phát hiện sớm](#tang-thu-ba-phat-hien-som). Xuyên suốt tất cả các tầng này, sự hiện diện của *replication* đôi khi hữu ích cho khôi phục dữ liệu trong các kịch bản cụ thể (mặc dù kế hoạch khôi phục dữ liệu không nên dựa vào replication).

<a id="hinh-26-4-hanh-trinh-cua-mot-object-tu-xoa-mem-den-huy-hoai"></a>        ![An object's journey from soft deletion to destruction.](../assets/imgs/fig-26-4.jpg)

Hình 26-4. Hành trình của một object từ xóa mềm đến hủy hoại

### Tầng Thứ Nhất: Xóa Mềm (First Layer: Soft Deletion)

Khi velocity cao và privacy quan trọng, các bug trong ứng dụng chiếm phần lớn sự cố data loss và corruption. Thật vậy, các bug xóa dữ liệu có thể trở nên phổ biến đến mức khả năng khôi phục dữ liệu đã xóa (undelete) trong một khoảng thời gian hạn chế trở thành phòng tuyến chính chống lại phần lớn data loss vô tình, nếu không sẽ là vĩnh viễn.

Bất kỳ sản phẩm nào tôn trọng quyền riêng tư của người dùng của nó phải cho phép người dùng xóa các tập hợp con được chọn và/hoặc toàn bộ dữ liệu của họ. Những sản phẩm như vậy phải chịu gánh nặng hỗ trợ do việc xóa nhầm gây ra. Cho người dùng khả năng khôi phục dữ liệu đã xóa của họ (ví dụ, qua một thư mục rác) làm giảm nhưng không thể hoàn toàn loại bỏ gánh nặng hỗ trợ này, đặc biệt nếu dịch vụ của bạn cũng hỗ trợ các tiện ích bổ sung bên thứ ba cũng có thể xóa dữ liệu.

Xóa mềm có thể giảm đáng kể hoặc thậm chí hoàn toàn loại bỏ gánh nặng hỗ trợ này. Xóa mềm có nghĩa là dữ liệu đã xóa ngay lập tức được đánh dấu như vậy, làm cho nó không thể sử dụng được bởi tất cả các đường dẫn mã quản trị của ứng dụng. Các đường dẫn mã quản trị có thể bao gồm truy tìm pháp lý, khôi phục tài khoản bị đánh cắp, quản trị doanh nghiệp, hỗ trợ người dùng, và gỡ rối vấn đề cùng các tính năng liên quan. Thực hiện xóa mềm khi một người dùng làm trống thùng rác của họ, và cung cấp một công cụ hỗ trợ người dùng cho phép các quản trị viên được ủy quyền khôi phục bất kỳ mục nào bị người dùng xóa nhầm. Google áp dụng chiến lược này cho các ứng dụng năng suất phổ biến nhất của chúng tôi; nếu không, gánh nặng kỹ thuật hỗ trợ người dùng sẽ không thể chấp nhận được.

Bạn có thể mở rộng chiến lược xóa mềm thêm nữa bằng cách cung cấp cho người dùng tùy chọn khôi phục dữ liệu đã xóa. Ví dụ, thùng rác của Gmail cho phép người dùng truy cập các tin nhắn bị xóa ít hơn 30 ngày trước.

Một nguồn phổ biến khác của việc xóa dữ liệu không mong muốn xảy ra do kết quả của việc tài khoản bị đánh cắp. Trong các kịch bản tài khoản bị đánh cắp, kẻ đánh cắp thường xóa dữ liệu của người dùng gốc trước khi sử dụng tài khoản cho việc gửi spam và các mục đích bất hợp pháp khác. Khi bạn kết hợp sự phổ biến của việc người dùng xóa nhầm với nguy cơ dữ liệu bị xóa bởi những kẻ đánh cắp, lập luận cho một giao diện xóa mềm và khôi phục đã xóa theo chương trình bên trong và/hoặc bên dưới ứng dụng của bạn trở nên rõ ràng.

Xóa mềm ngụ ý rằng một khi dữ liệu được đánh dấu như vậy, nó sẽ bị phá hủy sau một độ trễ hợp lý. Độ dài của độ trễ phụ thuộc vào chính sách và luật áp dụng của một tổ chức, nguồn lực và chi phí lưu trữ khả dụng, và giá cả sản phẩm và định vị trên thị trường, đặc biệt trong các trường hợp liên quan đến nhiều dữ liệu có vòng đời ngắn. Các lựa chọn phổ biến cho độ trễ xóa mềm là 15, 30, 45, hoặc 60 ngày. Theo kinh nghiệm của Google, phần lớn các vấn đề tài khoản bị đánh cắp và data integrity được báo cáo hoặc phát hiện trong vòng 60 ngày. Do đó, lập luận cho việc xóa mềm dữ liệu lâu hơn 60 ngày có thể không mạnh.

Google cũng phát hiện rằng các trường hợp xóa dữ liệu cấp tính gây thảm hại nhất là do các nhà phát triển ứng dụng không quen thuộc với mã hiện có nhưng đang làm việc trên mã liên quan đến xóa, đặc biệt là các pipeline xử lý batch (ví dụ, một pipeline MapReduce hoặc Hadoop offline). Có lợi khi thiết kế các giao diện của bạn để cản trở các nhà phát triển không quen thuộc với mã của bạn tránh qua các tính năng xóa mềm bằng mã mới. Một cách hiệu quả để đạt được điều này là triển khai các dịch vụ cung cấp cloud computing bao gồm các API xóa mềm và khôi phục đã xóa có sẵn, đảm bảo *bật tính năng đó*.<sup>[7](#fn7)</sup> Ngay cả bộ giáp tốt nhất cũng vô dụng nếu bạn không mặc nó lên.

Các chiến lược xóa mềm bao phủ các tính năng xóa dữ liệu trong các sản phẩm tiêu dùng như Gmail hoặc Google Drive, nhưng điều gì nếu bạn hỗ trợ một dịch vụ cung cấp cloud computing thay vào đó? Giả sử dịch vụ cung cấp cloud computing của bạn đã hỗ trợ tính năng xóa mềm và khôi phục đã xóa theo chương trình với các giá trị mặc định hợp lý, các kịch bản xóa dữ liệu nhầm còn lại sẽ bắt nguồn từ những sai lầm do chính các nhà phát triển nội bộ của bạn hoặc các khách hàng lập trình viên của bạn gây ra.

Trong những trường hợp như vậy, có thể hữu ích khi đưa vào một tầng xóa mềm bổ sung, mà chúng tôi sẽ gọi là "lazy deletion." Bạn có thể nghĩ về lazy deletion như việc làm sạch (purge) diễn ra ở hậu trường, được điều khiển bởi hệ thống lưu trữ (trong khi xóa mềm được điều khiển bởi và thể hiện ra ứng dụng hoặc dịch vụ client). Trong một kịch bản lazy deletion, dữ liệu bị xóa bởi một ứng dụng cloud trở nên không thể truy cập ngay lập tức bởi ứng dụng, nhưng được nhà cung cấp dịch vụ cloud giữ lại trong tối đa vài tuần trước khi bị phá hủy. Lazy deletion không được khuyến nghị trong tất cả các chiến lược phòng thủ nhiều lớp: một khoảng thời gian lazy deletion dài là tốn kém trong các hệ thống có nhiều dữ liệu vòng đời ngắn, và không thực tế trong các hệ thống phải đảm bảo phá hủy dữ liệu đã xóa trong một khung thời gian hợp lý (tức là, những hệ thống cung cấp các bảo đảm về privacy).

Để tóm tắt tầng đầu tiên của phòng thủ nhiều lớp:

-   Một thư mục rác cho phép người dùng khôi phục dữ liệu đã xóa là phòng thủ chính chống lại lỗi người dùng.
-   Xóa mềm là phòng thủ chính chống lại lỗi nhà phát triển và là phòng thủ thứ cấp chống lại lỗi người dùng.
-   Trong các dịch vụ cung cấp cho lập trình viên, lazy deletion là phòng thủ chính chống lại lỗi nhà phát triển nội bộ và là phòng thủ thứ cấp chống lại lỗi nhà phát triển bên ngoài.

Vậy còn *revision history*? Một số sản phẩm cung cấp khả năng hoàn tác các mục về các trạng thái trước đó. Khi một tính năng như vậy khả dụng cho người dùng, nó là một dạng thùng rác. Khi khả dụng cho các nhà phát triển, nó có thể hoặc không thay thế cho xóa mềm, tùy thuộc vào cách triển khai của nó.

Tại Google, revision history đã chứng minh là hữu ích trong việc khôi phục khỏi một số kịch bản data corruption, nhưng không phải trong việc khôi phục khỏi hầu hết các kịch bản data loss liên quan đến việc xóa nhầm, dù theo chương trình hay không. Đây là vì một số triển khai revision history xử lý việc xóa như một trường hợp đặc biệt trong đó các trạng thái trước đó phải bị xóa bỏ, thay vì biến đổi một mục mà lịch sử của nó có thể được giữ lại trong một khoảng thời gian nhất định. Để cung cấp sự bảo vệ đầy đủ chống lại việc xóa không mong muốn, hãy áp dụng các nguyên lý lazy và/hoặc xóa mềm cho revision history cũng vậy.

### Tầng Thứ Hai: Backup và Các Phương Pháp Khôi Phục Liên Quan (Second Layer: Backups and Their Related Recovery Methods)

Backup và khôi phục dữ liệu là phòng tuyến thứ hai sau xóa mềm. Nguyên tắc quan trọng nhất trong tầng này là backup không quan trọng; điều quan trọng là khôi phục. Các nhân tố hỗ trợ khôi phục thành công nên thúc đẩy các quyết định backup của bạn, chứ không phải ngược lại.

Nói cách khác, các kịch bản trong đó bạn muốn backup của mình giúp bạn khôi phục nên quy định những điều sau:

-   Các phương pháp backup và khôi phục nào để sử dụng
-   Tần suất bạn thiết lập các điểm khôi phục bằng cách tạo backup full hoặc incremental dữ liệu của bạn
-   Bạn lưu trữ backup ở đâu
-   Bạn giữ backup trong bao lâu

Bạn có thể chấp nhận mất bao nhiêu dữ liệu gần đây trong một nỗ lực khôi phục? Càng ít dữ liệu bạn có thể chấp nhận mất, bạn càng nên nghiêm túc về một chiến lược backup incremental. Trong một trong những trường hợp cực đoan nhất của Google, chúng tôi đã sử dụng một chiến lược backup streaming gần thời gian thực cho một phiên bản cũ hơn của Gmail.

Ngay cả khi tiền không phải là hạn chế, các bản backup full thường xuyên là đắt đỏ theo những cách khác. Đáng chú ý nhất, chúng áp đặt một gánh nặng tính toán lên các datastore trực tiếp của dịch vụ bạn trong khi nó đang phục vụ người dùng, đẩy dịch vụ của bạn đến gần hơn với các giới hạn khả năng mở rộng và hiệu suất. Để giảm nhẹ gánh nặng này, bạn có thể tạo các bản backup full trong các giờ ít bận rộn, và sau đó một chuỗi các bản backup incremental khi dịch vụ của bạn bận hơn.

Bạn cần khôi phục nhanh đến mức nào? Càng nhanh người dùng của bạn cần được giải cứu, backup của bạn càng nên ở gần (local). Google thường giữ các snapshot<sup>[8](#fn8)</sup> đắt đỏ nhưng nhanh để khôi phục trong một khoảng thời gian rất ngắn trong nội bộ instance lưu trữ, và lưu trữ các bản backup ít gần đây hơn trên lưu trữ phân tán truy cập ngẫu nhiên trong cùng một (hoặc gần) datacenter trong một khoảng thời gian hơi lâu hơn. Một chiến lược như vậy đơn lẻ sẽ không bảo vệ khỏi các sự cố ở mức địa điểm, nên những backup đó thường được chuyển đến các vị trí nearline hoặc offline trong một khoảng thời gian dài hơn trước khi chúng hết hạn để nhường chỗ cho các backup mới hơn.

Backup của bạn nên đạt ngược về trước xa đến mức nào? Chiến lược backup của bạn trở nên đắt đỏ hơn theo mức bạn đạt ngược về trước xa hơn, trong khi các kịch bản mà bạn có thể hy vọng khôi phục tăng lên (mặc dù sự tăng này chịu sự giới hạn của hiệu quả giảm dần).

Theo kinh nghiệm của Google, các bug biến đổi hoặc xóa dữ liệu ở mức độ thấp trong mã ứng dụng đòi hỏi khả năng đạt ngược về trước xa nhất, vì một số bug trong số đó được chú ý nhiều tháng sau khi data loss đầu tiên bắt đầu. Những trường hợp như vậy gợi ý rằng bạn muốn có khả năng đạt ngược về trước xa nhất có thể.

Mặt khác, trong một môi trường phát triển velocity cao, những thay đổi mã và schema có thể làm cho các bản backup cũ đắt đỏ hoặc không thể sử dụng. Hơn nữa, việc khôi phục các tập dữ liệu khác nhau đến các điểm khôi phục khác nhau là thách thức, vì việc làm đó sẽ liên quan đến nhiều bản backup. Tuy nhiên, đó chính xác là loại nỗ lực khôi phục được đòi hỏi bởi các kịch bản data corruption hoặc xóa ở mức độ thấp.

Các chiến lược được mô tả trong [Tầng thứ ba: Phát hiện sớm](#tang-thu-ba-phat-hien-som) được nhằm vào việc tăng tốc phát hiện các bug biến đổi hoặc xóa dữ liệu ở mức độ thấp trong mã ứng dụng, ít nhất một phần chống lại nhu cầu cho loại nỗ lực khôi phục phức tạp này. Dù vậy, làm sao bạn có thể mang lại sự bảo vệ hợp lý trước khi bạn biết những loại vấn đề nào cần phát hiện? Google đã chọn vạch ranh giới giữa 30 và 90 ngày backup cho nhiều dịch vụ. Vị trí của một dịch vụ nằm trong khoảng này phụ thuộc vào mức độ chấp nhận data loss của nó và đầu tư tương đối của nó vào phát hiện sớm.

Để tóm tắt lời khuyên của chúng tôi cho việc phòng chống 24 tổ hợp của các chế độ lỗi data integrity: xử lý một phạm vi rộng các kịch bản với chi phí hợp lý đòi hỏi một chiến lược backup theo tầng. Tầng đầu tiên bao gồm nhiều bản backup thường xuyên và khôi phục nhanh được lưu trữ gần nhất với các datastore trực tiếp, có thể sử dụng cùng hoặc tương tự các công nghệ lưu trữ như nguồn dữ liệu. Việc làm như vậy mang lại sự bảo vệ khỏi phần lớn các kịch bản liên quan đến bug phần mềm và lỗi nhà phát triển. Do chi phí tương đối, các bản backup được giữ lại trong tầng này từ vài giờ đến vài ngày, và có thể mất vài phút để khôi phục.

Tầng thứ hai bao gồm ít bản backup hơn được giữ trong khoảng một đến vài chục ngày trên các filesystem phân tán truy cập ngẫu nhiên ở local với địa điểm. Những backup này có thể mất hàng giờ để khôi phục và mang lại sự bảo vệ bổ sung khỏi những sự cố ảnh hưởng đến các công nghệ lưu trữ nhất định trong stack phục vụ của bạn, nhưng không phải các công nghệ được sử dụng để chứa các bản backup. Tầng này cũng bảo vệ khỏi các bug trong ứng dụng của bạn được phát hiện quá muộn để dựa vào tầng đầu tiên của chiến lược backup của bạn. Nếu bạn đang giới thiệu các phiên bản mới của mã của mình vào production hai lần một tuần, có thể hợp lý khi giữ những bản backup này trong ít nhất một hoặc hai tuần trước khi xóa chúng.

Các tầng tiếp theo tận dụng lưu trữ nearline như các thư viện băng chuyên dụng và lưu trữ offsite của phương tiện backup (ví dụ, băng hoặc ổ đĩa). Các bản backup trong các tầng này mang lại sự bảo vệ khỏi các vấn đề ở mức địa điểm, chẳng hạn như mất điện datacenter hoặc filesystem phân tán bị hỏng do một bug.

Việc di chuyển một lượng lớn dữ liệu đến và đi giữa các tầng là tốn kém. Mặt khác, dung lượng lưu trữ ở các tầng sau không cạnh tranh với sự tăng trưởng của các instance lưu trữ production trực tiếp của dịch vụ bạn. Kết quả là, các bản backup trong các tầng này có xu hướng được tạo ít thường xuyên hơn nhưng được giữ lâu hơn.

### Tầng Bao Quát: Replication (Overarching Layer: Replication)

Trong một thế giới lý tưởng, mọi instance lưu trữ, bao gồm cả các instance chứa backup của bạn, đều được nhân bản. Trong một nỗ lực khôi phục dữ liệu, điều cuối cùng bạn muốn là phát hiện ra rằng chính bản backup của bạn đã mất dữ liệu cần thiết hoặc rằng datacenter chứa bản backup hữu ích nhất đang được bảo trì.

Khi lượng dữ liệu tăng lên, việc nhân bản mọi instance lưu trữ không phải lúc nào cũng khả thi. Trong những trường hợp như vậy, hợp lý khi dàn xếp các bản backup liên tiếp qua các địa điểm khác nhau, mỗi địa điểm có thể bị hỏng độc lập, và ghi backup của bạn bằng một phương pháp dự phòng như RAID, mã xóa Reed-Solomon, hoặc replication kiểu GFS.<sup>[9](#fn9)</sup>

Khi chọn một hệ thống dự phòng, đừng dựa vào một scheme ít được sử dụng mà các "bài kiểm tra" về hiệu quả duy nhất là những nỗ lực khôi phục dữ liệu ít thường xuyên của chính bạn. Thay vào đó, hãy chọn một scheme phổ biến được sử dụng phổ biến và liên tục bởi nhiều người dùng của nó.

### 1T Đối Chọi 1E: Không "Chỉ" Là Một Bản Backup Lớn Hơn (1T Versus 1E: Not "Just" a Bigger Backup)

Các quy trình và thực hành được áp dụng cho các lượng dữ liệu đo bằng T (terabyte) không mở rộng scale tốt sang dữ liệu đo bằng E (exabyte). Việc xác thực, sao chép, và thực hiện các bài kiểm tra vòng tròn trên vài gigabyte dữ liệu có cấu trúc là một vấn đề thú vị. Tuy nhiên, giả sử bạn có đủ kiến thức về schema và mô hình giao dịch của mình, bài tập này không đặt ra bất kỳ thách thức đặc biệt nào. Bạn thường chỉ cần có được các nguồn lực máy để lặp qua dữ liệu của mình, thực hiện một số logic xác thực, và phân bổ đủ lưu trữ để giữ một vài bản sao dữ liệu của mình.

Bây giờ hãy nâng mức cược lên: thay vì vài gigabyte, hãy thử bảo mật và xác thực 700 petabyte dữ liệu có cấu trúc. Giả sử hiệu suất SATA 2.0 lý tưởng là 300 MB/s, một tác vụ duy nhất lặp qua tất cả dữ liệu của bạn và thực hiện ngay cả các kiểm tra xác thực cơ bản nhất sẽ mất 8 thập kỷ. Tạo một vài bản backup full, giả sử bạn có phương tiện, sẽ mất ít nhất bằng bấy lâu. Thời gian khôi phục, với một số xử lý hậu, sẽ mất lâu hơn. Chúng ta đang hướng đến gần như một thế kỷ để khôi phục một bản backup có tuổi lên đến 80 năm khi bạn bắt đầu khôi phục. Rõ ràng, một chiến lược như vậy cần phải được suy nghĩ lại.

Kỹ thuật phổ biến nhất và phần lớn hiệu quả được sử dụng để backup một lượng dữ liệu khổng lồ là thiết lập các "điểm tin cậy" trong dữ liệu của bạn — những phần dữ liệu đã lưu trữ của bạn được xác thực sau khi trở nên bất biến, thường là qua sự trôi qua của thời gian. Một khi chúng ta biết rằng một hồ sơ người dùng hoặc giao dịch nhất định đã được cố định và sẽ không bị thay đổi thêm, chúng ta có thể xác thực trạng thái bên trong của nó và tạo các bản sao phù hợp cho mục đích khôi phục. Sau đó bạn có thể tạo các bản backup incremental chỉ bao gồm dữ liệu đã được sửa đổi hoặc thêm vào kể từ lần backup cuối của bạn. Kỹ thuật này đưa thời gian backup của bạn vào cùng hàng với thời gian xử lý "mainline" của bạn, có nghĩa là các bản backup incremental thường xuyên có thể cứu bạn khỏi công việc xác thực và sao chép đơn thể 80 năm.

Tuy nhiên, hãy nhớ rằng chúng ta quan tâm đến *restores* (khôi phục), không phải backup. Giả sử chúng tôi đã tạo một bản backup full ba năm trước và đã tạo các bản backup incremental hàng ngày kể từ đó. Một lần khôi phục full dữ liệu của chúng ta sẽ xử lý tuần tự một chuỗi gồm hơn 1.000 bản backup phụ thuộc lẫn nhau cao. Mỗi bản backup độc lập gây thêm rủi ro bị lỗi, không nói đến gánh nặng logistics của việc lên lịch và chi phí thời gian chạy của những tác vụ đó.

Một cách khác mà chúng ta có thể giảm thời gian thực của các công việc sao chép và xác thực của mình là phân phối tải. Nếu chúng ta shard dữ liệu của mình tốt, là có thể chạy *N* tác vụ song song, với mỗi tác vụ chịu trách nhiệm sao chép và xác thực 1/*N* dữ liệu của chúng ta. Việc làm như vậy đòi hỏi một số suy nghĩ trước và lập kế hoạch trong thiết kế schema và triển khai vật lý dữ liệu của chúng ta để:

-   Cân bằng dữ liệu đúng cách
-   Đảm bảo tính độc lập của mỗi shard
-   Tránh tranh chấp giữa các tác vụ anh em song song

Giữa việc phân phối tải theo chiều ngang và giới hạn công việc vào các lát dọc của dữ liệu được phân tách bằng thời gian, chúng ta có thể giảm tám thập kỷ thời gian thực đó đi vài bậc độ lớn, làm cho việc khôi phục của chúng ta trở nên có liên quan.

<a id="tang-thu-ba-phat-hien-som"></a>

### Tầng Thứ Ba: Phát Hiện Sớm (Third Layer: Early Detection)

Dữ liệu "xấu" không ngồi yên chờ đợi, nó lan truyền. Các tham chiếu đến dữ liệu bị thiếu hoặc bị hỏng được sao chép, các liên kết tỏa ra, và với mỗi lần cập nhật, chất lượng tổng thể của datastore của bạn giảm xuống. Các giao dịch phụ thuộc sau đó và những thay đổi định dạng dữ liệu tiềm năng làm cho việc khôi phục từ một backup nhất định trở nên khó khăn hơn theo thời gian trôi qua. Càng sớm bạn biết về một data loss, việc khôi phục của bạn càng dễ dàng và hoàn chỉnh hơn.

#### Những thách thức mà các nhà phát triển cloud phải đối mặt

Trong các môi trường velocity cao, các dịch vụ ứng dụng và hạ tầng cloud đối mặt với nhiều thách thức data integrity khi chạy, chẳng hạn như:

-   Tính toàn vẹn tham chiếu giữa các datastore
-   Thay đổi schema
-   Mã già cỗi
-   Data migration không dừng
-   Các điểm tích hợp đang phát triển với các dịch vụ khác

Nếu không có nỗ lực kỹ thuật có ý thức để theo dõi các mối quan hệ đang nổi lên trong dữ liệu của mình, chất lượng dữ liệu của một dịch vụ thành công và đang tăng trưởng sẽ suy giảm theo thời gian.

Thường thì, nhà phát triển cloud non trẻ chọn một API lưu trữ nhất quán phân tán (như Megastore) ủy thác tính toàn vẹn dữ liệu của ứng dụng cho thuật toán nhất quán phân tán được triển khai bên dưới API (như Paxos; xem [Quản Lý Trạng Thái Quan Trọng: Sự Đồng Ý Phân Tán Cho Độ Tin Cậy](https://sre.google/sre-book/managing-critical-state/)). Nhà phát triển lập luận rằng riêng API được chọn sẽ giữ dữ liệu của ứng dụng trong tình trạng tốt. Kết quả là, họ thống nhất tất cả dữ liệu ứng dụng vào một giải pháp lưu trữ duy nhất đảm bảo nhất quán phân tán, tránh các vấn đề toàn vẹn tham chiếu đổi lại hiệu suất và/hoặc scale giảm.

Trong khi các thuật toán như vậy là không thể sai trong lý thuyết, các triển khai của chúng thường tràn ngập các mánh khóe (hacks), các tối ưu hóa, bug, và các phỏng đoán có căn cứ. Ví dụ: trong lý thuyết, Paxos bỏ qua các nút tính toán bị hỏng và có thể tiến lên chừng nào một quorum (đa số) các nút đang hoạt động được duy trì. Trong thực tế, tuy nhiên, việc bỏ qua một nút bị hỏng có thể tương ứng với timeout, retry, và các cách tiếp cận xử lý lỗi khác bên dưới một triển khai Paxos cụ thể [[Cha07]](https://sre.google/sre-book/bibliography#Cha07). Paxos nên cố gắng liên lạc với một nút không phản hồi trong bao lâu trước khi timeout nó? Khi một máy nhất định bị hỏng (có thể ngắt quãng) theo một cách nhất định, với một thời điểm nhất định, và tại một datacenter nhất định, một hành vi không thể dự đoán xảy ra. Càng lớn quy mô của một ứng dụng, ứng dụng càng thường xuyên bị ảnh hưởng, mà không hay biết, bởi những sự bất nhất như vậy. Nếu logic này đúng ngay cả khi được áp dụng cho các triển khai Paxos (như đã đúng với Google), thì nó phải càng đúng hơn cho các triển khai nhất quán cuối cùng như Bigtable (điều này cũng đã được chứng minh là đúng). Các ứng dụng bị ảnh hưởng không có cách nào để biết rằng 100% dữ liệu của chúng là tốt cho đến khi chúng kiểm tra: hãy tin tưởng các hệ thống lưu trữ, nhưng hãy xác minh!

Để làm phức tạp thêm vấn đề này, để khôi phục khỏi các kịch bản data corruption hoặc xóa ở mức độ thấp, chúng ta phải khôi phục các tập dữ liệu khác nhau đến các điểm khôi phục khác nhau bằng các bản backup khác nhau, trong khi những thay đổi mã và schema có thể làm cho các bản backup cũ không hiệu quả trong các môi trường velocity cao.

#### Xác thực dữ liệu ngoài vòng (Out-of-band data validation)

Để ngăn chất lượng dữ liệu suy giảm trước mắt người dùng, và để phát hiện các kịch bản data corruption hoặc data loss ở mức độ thấp trước khi chúng trở nên không thể khôi phục, một hệ thống kiểm tra và cân bằng ngoài vòng là cần thiết cả bên trong và giữa các datastore của một ứng dụng.

Phần lớn, các [pipeline xác thực dữ liệu](https://sre.google/workbook/data-processing/) này được triển khai dưới dạng các tập hợp map-reduction hoặc Hadoop jobs. Thường, các pipeline như vậy được thêm như một ý nghĩ muộn cho các dịch vụ đã phổ biến và thành công. Đôi khi, các pipeline như vậy lần đầu tiên được thử khi các dịch vụ đạt đến giới hạn khả năng mở rộng và được xây dựng lại từ đầu. Google đã xây dựng các bộ xác thực (validators) để phản ứng với mỗi một trong những tình huống này.

Việc chuyển một số nhà phát triển sang làm việc trên một pipeline xác thực dữ liệu có thể làm chậm velocity kỹ thuật trong ngắn hạn. Tuy nhiên, việc dành nguồn lực kỹ thuật cho xác thực dữ liệu mang lại cho các nhà phát triển khác lòng can đảm để di chuyển nhanh hơn trong dài hạn, bởi vì các kỹ sư biết rằng các bug data corruption ít có khả năng lẻn vào production mà không bị chú ý. Tương tự như những lợi ích đạt được khi unit test được giới thiệu sớm trong vòng đời dự án, một pipeline xác thực dữ liệu dẫn đến sự tăng tốc tổng thể của các dự án phát triển phần mềm.

Để dẫn ra một ví dụ cụ thể: Gmail sở hữu một số bộ xác thực dữ liệu, mỗi cái đã phát hiện các vấn đề data integrity thực tế trong production. Các nhà phát triển Gmail tìm được sự an tâm từ kiến thức rằng các bug đưa đến những sự bất nhất trong dữ liệu production được phát hiện trong vòng 24 giờ, và rùng mình trước ý nghĩ chạy các bộ xác thực dữ liệu của họ ít thường xuyên hơn hàng ngày. Những bộ xác thực này, cùng với một văn hóa unit test và regression test và các thực hành tốt nhất khác, đã mang lại cho các nhà phát triển Gmail lòng can đảm để giới thiệu các thay đổi mã vào triển khai lưu trữ production của Gmail thường xuyên hơn một lần một tuần.

Xác thực dữ liệu ngoài vòng là tinh tế để triển khai đúng cách. Khi quá chặt chẽ, thậm chí những thay đổi đơn giản, phù hợp cũng gây ra xác thực thất bại. Kết quả là, các kỹ sư bỏ xác thực dữ liệu hoàn toàn. Nếu xác thực dữ liệu không chặt chẽ đủ, data corruption ảnh hưởng đến trải nghiệm người dùng có thể lọt qua mà không bị phát hiện. Để tìm sự cân bằng đúng, hãy chỉ xác thực các bất biến (invariants) gây ra sự tàn phá cho người dùng.

Ví dụ, Google Drive định kỳ xác định rằng nội dung file khớp với danh sách trong các thư mục Drive. Nếu hai yếu tố này không khớp, một số file sẽ bị thiếu dữ liệu — một kết quả thảm khốc. Các nhà phát triển hạ tầng Drive đã tham gia sâu vào data integrity đến mức họ cũng nâng cao các bộ xác thực của mình để tự động sửa các sự bất nhất như vậy. Biện pháp bảo vệ này đã biến một tình trạng data loss khẩn cấp tiềm năng "tất cả mọi người vào việc — ôi trời — file đang biến mất!" năm 2013 thành một tình trạng hoạt động bình thường, "hãy về nhà và sửa nguyên nhân gốc vào thứ Hai." Bằng cách biến các tình trạng khẩn cấp thành hoạt động bình thường, các bộ xác thực cải thiện tinh thần kỹ thuật, chất lượng cuộc sống, và tính dự đoán được.

Các bộ xác thực ngoài vòng có thể đắt đỏ ở quy mô lớn. Một phần đáng kể dấu chân tài nguyên tính toán của Gmail hỗ trợ một tập hợp các bộ xác thực hàng ngày. Để cộng thêm chi phí này, các bộ xác thực này cũng làm giảm tỷ lệ cache hit phía server, làm giảm độ phản hồi phía server mà người dùng trải nghiệm. Để giảm nhẹ tác động này lên độ phản hồi, Gmail cung cấp một loạt các núm chỉnh để giới hạn tỷ lệ các bộ xác thực của nó và định kỳ refactor lại các bộ xác thực để giảm tranh chấp ổ đĩa. Trong một nỗ lực refactor như vậy, chúng tôi đã cắt giảm tranh chấp cho các đầu đĩa đi 60% mà không giảm đáng kể phạm vi các bất biến mà chúng bao phủ. Trong khi phần lớn các bộ xác thực của Gmail chạy hàng ngày, khối lượng công việc của bộ xác thực lớn nhất được chia thành 10–14 shard, với một shard được xác thực mỗi ngày vì lý do quy mô.

Google Compute Storage là một ví dụ khác của những thách thức mà quy mô đưa đến cho xác thực dữ liệu. Khi các bộ xác thực ngoài vòng của nó không còn có thể hoàn thành trong vòng một ngày, các kỹ sư Compute Storage đã phải tìm ra một cách hiệu quả hơn để xác minh metadata của nó thay vì chỉ sử dụng brute force. Tương tự như ứng dụng của nó trong khôi phục dữ liệu, một chiến lược theo tầng cũng có thể hữu ích trong xác thực dữ liệu ngoài vòng. Khi một dịch vụ mở rộng scale, hãy hy sinh sự chặt chẽ trong các bộ xác thực hàng ngày. Đảm bảo rằng các bộ xác thực hàng ngày tiếp tục bắt được các kịch bản thảm hại nhất trong vòng 24 giờ, nhưng tiếp tục với xác thực chặt chẽ hơn ở tần suất thấp hơn để kiểm soát chi phí và latency.

Gỡ rối các xác thực thất bại có thể mất một nỗ lực đáng kể. Nguyên nhân của một xác thực thất bại ngắt quãng có thể biến mất trong vòng vài phút, vài giờ, hoặc vài ngày. Do đó, khả năng nhanh chóng đào sâu vào các log kiểm toán xác thực là thiết yếu. Các dịch vụ Google trưởng thành cung cấp cho các kỹ sư on-call với tài liệu và công cụ toàn diện để gỡ rối. Ví dụ, các kỹ sư on-call của Gmail được cung cấp với:

-   Một bộ mục playbook mô tả cách phản ứng với một cảnh báo xác thực thất bại
-   Một công cụ điều tra kiểu BigQuery
-   Một dashboard xác thực dữ liệu

Xác thực dữ liệu ngoài vòng hiệu quả đòi hỏi tất cả những điều sau:

-   Quản lý tác vụ xác thực
-   Giám sát, cảnh báo, và dashboard
-   Các tính năng rate-limiting
-   Công cụ gỡ rối
-   Playbook production
-   Các API xác thực dữ liệu làm cho việc thêm và refactor bộ xác thực dễ dàng

Phần lớn các nhóm kỹ thuật nhỏ hoạt động với velocity cao không thể chi trả để thiết kế, xây dựng, và duy trì tất cả những hệ thống này. Nếu họ bị ép buộc làm như vậy, kết quả thường là những thứ làm một lần mong manh, hạn chế, và lãng phí nhanh chóng rơi vào tình trạng hư hỏng. Do đó, hãy cấu trúc các nhóm kỹ thuật của bạn sao cho một nhóm hạ tầng trung tâm cung cấp một framework xác thực dữ liệu cho nhiều nhóm kỹ thuật sản phẩm. Nhóm hạ tầng trung tâm duy trì framework xác thực dữ liệu ngoài vòng, trong khi các nhóm kỹ thuật sản phẩm duy trì logic kinh doanh tùy chỉnh ở trung tâm bộ xác thực để theo kịp các sản phẩm đang phát triển của họ.

### Biết Rằng Khôi Phục Dữ Liệu Sẽ Hoạt Động (Knowing That Data Recovery Will Work)

Một bóng đèn bị vỡ khi nào? Khi gạt công tắc không bật đèn được? Không phải luôn luôn—thường thì bóng đèn đã hỏng từ trước, và bạn đơn giản chỉ nhận thấy sự hỏng hóc tại lần gạt công tắc không phản hồi. Vào lúc đó, căn phòng đã tối và bạn đã đau chân.

Tương tự, các sự phụ thuộc khôi phục của bạn (nghĩa là chủ yếu, nhưng không chỉ, backup của bạn), có thể đang ở một trạng thái hỏng tiềm ẩn, mà bạn không hay biết cho đến khi bạn cố gắng khôi phục dữ liệu.

Nếu bạn phát hiện ra rằng quy trình khôi phục của bạn bị hỏng trước khi bạn cần dựa vào nó, bạn có thể giải quyết lỗ hổng trước khi bạn trở thành nạn nhân của nó: bạn có thể tạo một bản backup khác, cung cấp thêm tài nguyên, và thay đổi SLO của bạn. Nhưng để thực hiện các hành động này một cách chủ động, trước tiên bạn phải biết chúng là cần thiết. Để phát hiện những lỗ hổng này:

-   Liên tục kiểm tra quy trình khôi phục như một phần của các hoạt động bình thường của bạn
-   Thiết lập các cảnh báo được kích hoạt khi một quy trình khôi phục không cung cấp một chỉ báo heartbeat (nhịp tim) về sự thành công của nó

Điều gì có thể sai với quy trình khôi phục của bạn? Bất cứ thứ gì và mọi thứ—đó là lý do tại sao bài kiểm tra duy nhất nên để bạn ngủ yên ban đêm là một bài kiểm tra end-to-end (từ đầu đến cuối) đầy đủ. Hãy để bằng chứng nằm trong chính sự vật. Ngay cả khi bạn mới đây đã thực hiện một cuộc khôi phục thành công, các phần của quy trình khôi phục của bạn vẫn có thể bị hỏng. Nếu bạn chỉ nhớ một bài học từ chương này, hãy nhớ rằng *bạn chỉ biết rằng bạn có thể khôi phục trạng thái gần đây của mình nếu bạn thực sự làm điều đó*.

Nếu các bài kiểm tra khôi phục là một sự kiện thủ công, được dàn dựng, việc kiểm thử trở thành một chút công việc nhàm chán không được chào đón mà không được thực hiện đủ sâu hoặc đủ thường xuyên để xứng đáng với sự tin tưởng của bạn. Do đó, hãy tự động hóa những bài kiểm tra này bất cứ khi nào có thể và sau đó chạy chúng liên tục.

Các khía cạnh của kế hoạch khôi phục của bạn mà bạn nên xác nhận là vô vàn:

-   Các bản backup của bạn có hợp lệ và đầy đủ, hay chúng rỗng?
-   Bạn có đủ tài nguyên máy để chạy tất cả các tác vụ thiết lập, khôi phục, và xử lý hậu cấu thành nên việc khôi phục của bạn?
-   Quy trình khôi phục có hoàn thành trong thời gian thực hợp lý?
-   Bạn có khả năng giám sát trạng thái của quy trình khôi phục của bạn khi nó tiến triển?
-   Bạn có thoát khỏi các sự phụ thuộc quan trọng vào các tài nguyên ngoài tầm kiểm soát của bạn, chẳng hạn như quyền truy cập vào một kho lưu trữ phương tiện offsite không khả dụng 24/7?

Các bài kiểm thử của chúng tôi đã phát hiện ra những sự hỏng hóc được nêu ở trên, cũng như sự hỏng hóc của nhiều thành phần khác của một khôi phục dữ liệu thành công. Nếu chúng tôi không phát hiện ra những sự hỏng hóc này trong các bài kiểm thử thường xuyên—tức là, nếu chúng tôi chỉ gặp phải các sự hỏng hóc khi chúng tôi cần khôi phục dữ liệu người dùng trong các tình trạng khẩn cấp thực sự—hoàn toàn có khả năng là một số sản phẩm thành công nhất của Google ngày nay có thể đã không chịu đựng được thử thách của thời gian.

Các sự hỏng hóc là không thể tránh khỏi. Nếu bạn chờ đến khi phát hiện ra chúng khi bạn đang bị dồn ép, đối mặt với một data loss thực sự, bạn đang đùa với lửa. Nếu kiểm thử buộc các sự hỏng hóc xảy ra trước khi thảm họa thực sự ập đến, bạn có thể sửa các vấn đề trước khi bất kỳ tổn hại nào trổ quả.

## Các Nghiên Cứu Thực Tế (Case Studies)

Đời thực mô phỏng nghệ thuật (hoặc trong trường hợp này, khoa học), và như chúng tôi đã dự đoán, đời thực đã mang đến cho chúng tôi những cơ hội đáng tiếc và không thể tránh được để kiểm tra các hệ thống và quy trình khôi phục dữ liệu của chúng tôi dưới áp lực thế giới thực. Hai trong số những cơ hội nổi bật và thú vị hơn được thảo luận ở đây.

### Gmail—Tháng 2 năm 2011: Khôi Phục từ GTape (Gmail—February, 2011: Restore from GTape)

Trường hợp nghiên cứu khôi phục đầu tiên mà chúng ta sẽ xem xét là duy nhất theo một vài cách: số lượng các sự hỏng hóc trùng hợp dẫn đến data loss, và thực tế rằng nó là lần sử dụng lớn nhất hệ thống backup offline GTape, phòng tuyến cuối cùng của chúng tôi.

### Chủ nhật, ngày 27 tháng 2 năm 2011, muộn trong buổi tối

Máy gọi trực (pager) của hệ thống backup Gmail được kích hoạt, hiển thị một số điện thoại để tham gia một cuộc gọi hội nghị. Sự kiện mà chúng tôi đã lo lắng từ lâu—thực sự, lý do cho sự tồn tại của hệ thống backup—đã xảy ra: Gmail đã mất một lượng đáng kể dữ liệu người dùng. Bất chấp nhiều biện pháp bảo vệ, kiểm tra nội bộ, và sự dự phòng của hệ thống, dữ liệu đã biến mất khỏi Gmail.

Đây là lần sử dụng quy mô lớn đầu tiên của GTape, một hệ thống backup toàn cầu cho Gmail, để khôi phục dữ liệu khách hàng trực tiếp. May mắn, đây không phải là lần khôi phục như vậy đầu tiên, vì những tình huống tương tự đã được mô phỏng nhiều lần trước đó. Do đó, chúng tôi có thể:

-   Đưa ra một ước tính về thời gian sẽ mất để khôi phục phần lớn các tài khoản người dùng bị ảnh hưởng
-   Khôi phục tất cả các tài khoản trong vòng vài giờ của ước tính ban đầu của chúng tôi
-   Khôi phục 99%+ dữ liệu trước thời điểm hoàn thành ước tính

Khả năng đưa ra một ước tính như vậy có phải là may mắn? Không—sự thành công của chúng tôi là quả ngọt của kế hoạch, sự tuân thủ các thực hành tốt nhất, công việc chăm chỉ, và sự hợp tác, và chúng tôi vui mừng thấy đầu tư của chúng tôi vào mỗi thành phần trong số này được trả lời tốt như vậy. Google đã có thể khôi phục dữ liệu bị mất kịp thời bằng cách thực hiện một kế hoạch được thiết kế theo các thực hành tốt nhất của *Phòng thủ nhiều lớp* và *Chuẩn Bị Khẩn Cấp*.

Khi Google công khai tiết lộ rằng chúng tôi đã khôi phục dữ liệu này từ hệ thống backup băng chưa từng được công bố trước đó của chúng tôi [[Slo11]](https://sre.google/sre-book/bibliography#Slo11), phản ứng của công chúng là một sự pha trộn giữa ngạc nhiên và thích thú. Băng? Google không có nhiều ổ đĩa và một mạng nhanh để nhân bản dữ liệu quan trọng như vậy sao? Tất nhiên Google có những tài nguyên như vậy, nhưng nguyên tắc Phòng thủ nhiều lớp đòi hỏi phải có nhiều tầng bảo vệ để phòng khi một cơ chế bảo vệ đơn lẻ nào đó suy sụp hoặc bị xâm phạm. Việc backup các hệ thống online như Gmail cung cấp phòng thủ nhiều lớp ở hai tầng:

-   Một sự hỏng hóc của các hệ thống phụ dự phòng và backup nội bộ của Gmail
-   Một sự hỏng hóc phổ biến hoặc lỗ hổng zero-day trong một device driver hoặc filesystem ảnh hưởng đến phương tiện lưu trữ nền tảng (ổ đĩa)

Sự cố cụ thể này là kết quả của kịch bản đầu tiên—trong khi Gmail có các phương tiện nội bộ để khôi phục dữ liệu bị mất, sự mất mát này vượt quá những gì các phương tiện nội bộ có thể khôi phục.

Một trong những khía cạnh được ca ngợi nhất bên trong của việc khôi phục dữ liệu Gmail là mức độ hợp tác và phối hợp suôn sẻ cấu thành nên việc khôi phục. Nhiều nhóm, một số hoàn toàn không liên quan đến Gmail hoặc khôi phục dữ liệu, đã góp sức giúp đỡ. Việc khôi phục đã không thể thành công suôn sẻ như vậy mà không có một kế hoạch trung tâm để dàn dựng một nỗ lực khổng lồ phân phối rộng rãi như vậy; kế hoạch này là sản phẩm của các buổi diễn tập và chạy thử (dry runs) định kỳ. Sự tận tâm của Google đối với việc chuẩn bị khẩn cấp khiến chúng tôi xem những sự hỏng hóc như vậy là không thể tránh được. Chấp nhận sự không thể tránh được này, chúng tôi không hy vọng hoặc cá cược để tránh những thảm họa như vậy, mà dự kiến rằng chúng sẽ xảy ra. Do đó, chúng tôi cần một kế hoạch để đối phó không chỉ với các sự hỏng hóc có thể lường trước, mà còn với một mức độ hư hỏng ngẫu nhiên không phân biệt, nữa.

Nói ngắn gọn, chúng tôi luôn *biết* rằng việc tuân thủ các thực hành tốt nhất là quan trọng, và thật tốt khi thấy châm ngôn đó được chứng minh là đúng.

### Google Music—Tháng 3 năm 2012: Phát Hiện Xóa Tràn Lan (Google Music—March 2012: Runaway Deletion Detection)

Sự cố thứ hai mà chúng ta sẽ xem xét bao gồm những thách thức về logistics duy nhất đối với quy mô của datastore được khôi phục: bạn lưu trữ hơn 5.000 băng ở đâu, và làm sao đọc nhiều dữ liệu như vậy từ phương tiện offline (một cách hiệu quả hoặc thậm chí khả thi) trong một khoảng thời gian hợp lý?

### Thứ Ba, ngày 6 tháng 3 năm 2012, giữa buổi chiều

### Phát hiện vấn đề

Một người dùng Google Music báo cáo rằng các track nhạc trước đó không có vấn đề đang bị bỏ qua. Nhóm chịu trách nhiệm tiếp xúc với người dùng của Google Music thông báo cho các kỹ sư Google Music. Vấn đề được điều tra như một vấn đề streaming phương tiện tiềm năng.

Vào ngày 7 tháng 3, kỹ sư điều tra phát hiện rằng metadata của track không thể phát thiếu một tham chiếu lẽ ra phải trỏ đến dữ liệu âm thanh thực. Anh ta ngạc nhiên. Cách sửa hiển nhiên là định vị dữ liệu âm thanh và tái lập tham chiếu đến dữ liệu. Tuy nhiên, Google tự hào về một văn hóa sửa các vấn đề ở gốc, nên kỹ sư đào sâu hơn.

Khi anh ta tìm thấy nguyên nhân của sự suy giảm data integrity, anh ta suýt nữa bị đau tim: tham chiếu âm thanh đã bị xóa bởi một pipeline xóa dữ liệu bảo vệ quyền riêng tư. Phần này của Google Music được thiết kế để xóa một số lượng rất lớn các track âm thanh trong thời gian kỷ lục.

### Đánh giá thiệt hại

Chính sách quyền riêng tư của Google bảo vệ dữ liệu cá nhân của người dùng. Khi áp dụng cụ thể cho Google Music, chính sách quyền riêng tư của chúng tôi có nghĩa là các file nhạc và metadata liên quan được xóa trong một khoảng thời gian hợp lý sau khi người dùng xóa chúng. Khi độ phổ biến của Google Music tăng vọt, lượng dữ liệu tăng nhanh, nên triển khai xóa ban đầu cần được thiết kế lại vào năm 2012 để hiệu quả hơn. Vào ngày 6 tháng 2, pipeline xóa dữ liệu cập nhật đã có lần chạy đầu tiên, để xóa metadata liên quan. Lúc đó dường như không có gì sai, nên một giai đoạn thứ hai của pipeline được cho phép xóa cả dữ liệu âm thanh liên quan.

Có phải cơn ác mộng tồi tệ nhất của kỹ sư là sự thật? Anh ta ngay lập tức kéo còi báo động, nâng mức độ ưu tiên của vụ hỗ trợ lên phân loại khẩn cấp nhất của Google và báo cáo vấn đề cho ban quản lý kỹ thuật và Site Reliability Engineering. Một nhóm nhỏ các nhà phát triển Google Music và SRE tập hợp để giải quyết vấn đề, và pipeline gây lỗi được tạm thời vô hiệu hóa để ngăn dòng tổn thất người dùng bên ngoài.

Việc kiểm tra thủ công metadata cho hàng triệu đến hàng tỷ file được tổ chức trên nhiều datacenter là không tưởng. Vì vậy, nhóm đã vội vã viết một MapReduce job để đánh giá thiệt hại và tuyệt vọng chờ job hoàn thành. Họ đông cứng khi kết quả đến vào ngày 8 tháng 3: pipeline xóa dữ liệu đã refactor xóa bỏ khoảng 600.000 tham chiếu âm thanh lẽ ra không nên bị xóa, ảnh hưởng đến các file âm thanh của 21.000 người dùng. Vì pipeline chẩn đoán vội vã đã làm một số đơn giản hóa, phạm vi thiệt hại thực có thể tồi tệ hơn.

Đã hơn một tháng kể từ khi pipeline xóa dữ liệu có bug chạy lần đầu, và chính lần chạy đầu tiên đó đã xóa hàng trăm nghìn track âm thanh lẽ ra không nên bị xóa. Có còn hy vọng nào để lấy lại dữ liệu không? Nếu các track không được khôi phục, hoặc không được khôi phục đủ nhanh, Google sẽ phải đối mặt với sự phẫn nộ từ người dùng của mình. Làm sao chúng tôi có thể không chú ý đến sự trục trặc này?

### Giải quyết vấn đề

### Nỗ lực song song xác định bug và khôi phục

Bước đầu tiên trong việc giải quyết vấn đề là xác định bug thực và xác định cách và tại sao bug xảy ra. Chừng nào nguyên nhân gốc không được xác định và sửa chữa, mọi nỗ lực khôi phục sẽ vô ích. Chúng tôi sẽ chịu áp lực để kích hoạt lại pipeline để tôn trọng các yêu cầu của những người dùng đã xóa các track âm thanh, nhưng việc làm như vậy sẽ gây hại cho những người dùng vô tội, những người sẽ tiếp tục mất nhạc mua từ cửa hàng, hoặc tệ hơn, các file âm thanh do chính họ ghi công phu. Cách duy nhất để thoát khỏi tình thế Catch-22 (dở khóc dở cười)<sup>[10](#fn10)</sup> là sửa vấn đề ở gốc của nó, và sửa nó nhanh chóng.

Tuy nhiên, không có thời gian để lãng phí trước khi triển khai nỗ lực khôi phục. Chính các track âm thanh đã được backup ra băng, nhưng không giống như trường hợp nghiên cứu Gmail của chúng tôi, các bản backup băng mã hóa cho Google Music được chở bằng xe tải đến các vị trí lưu trữ offsite, vì tùy chọn đó cung cấp nhiều không gian hơn cho các bản backup cồng kềnh dữ liệu âm thanh của người dùng. Để khôi phục nhanh trải nghiệm của những người dùng bị ảnh hưởng, nhóm đã quyết định gỡ rối nguyên nhân gốc trong khi lấy các bản backup băng offsite (một tùy chọn khôi phục khá tốn thời gian) song song.

Các kỹ sư chia thành hai nhóm. Các SRE giàu kinh nghiệm nhất làm việc cho nỗ lực khôi phục, trong khi các nhà phát triển phân tích mã xóa dữ liệu và cố gắng sửa bug data loss ở gốc của nó. Do kiến thức không đầy đủ về vấn đề gốc, việc khôi phục sẽ phải được dàn dựng qua nhiều lượt. Lốc đầu tiên gồm gần một nửa triệu track âm thanh được xác định, và nhóm duy trì hệ thống backup băng được thông báo về nỗ lực khôi phục khẩn cấp lúc 4:34 chiều (giờ Thái Bình Dương) ngày 8 tháng 3.

Nhóm khôi phục có một yếu tố ủng hộ họ: nỗ lực khôi phục này diễn ra chỉ vài tuần sau bài tập kiểm tra disaster recovery thường niên của công ty (xem [[Kri12]](https://sre.google/sre-book/bibliography#Kri12)). Nhóm backup băng đã biết các khả năng và giới hạn của các hệ thống phụ của họ, vốn là đối tượng của các bài kiểm tra DiRT (Disaster Recovery Testing), và bắt đầu phủi bụi một công cụ mới mà họ đã kiểm thử trong một bài tập DiRT. Sử dụng công cụ mới, nhóm khôi phục kết hợp bắt đầu công việc tỉ mỉ ánh xạ hàng trăm nghìn file âm thanh đến các bản backup được đăng ký trong hệ thống backup băng, và sau đó ánh xạ các file từ các bản backup đến các băng thực tế.

Theo cách này, nhóm xác định rằng nỗ lực khôi phục ban đầu sẽ liên quan đến việc gọi về hơn 5.000 băng backup bằng xe tải. Sau đó, các kỹ thuật viên datacenter sẽ phải dọn không gian cho các băng tại các thư viện băng. Một quá trình dài và phức tạp để đăng ký các băng và trích xuất dữ liệu từ các băng sẽ theo sau, liên quan đến các biện pháp vòng qua (workarounds) và giảm thiểu trong trường hợp có băng hỏng, ổ hỏng, và các tương tác hệ thống bất ngờ.

Không may, chỉ 436.223 trong số khoảng 600.000 track âm thanh bị mất được tìm thấy trên các bản backup băng, điều đó có nghĩa là khoảng 161.000 track âm thanh khác đã bị "ăn" trước khi chúng có thể được backup. Nhóm khôi phục quyết định tìm cách khôi phục 161.000 track thiếu còn lại sau khi họ khởi động quy trình khôi phục cho các track có backup băng.

Trong khi đó, nhóm nguyên nhân gốc đã theo đuổi và từ bỏ một mồi nhử (red herring): ban đầu họ cho rằng một dịch vụ lưu trữ mà Google Music phụ thuộc đã cung cấp dữ liệu có bug gây nhầm lẫn cho các pipeline xóa dữ liệu để xóa nhầm dữ liệu âm thanh. Sau khi điều tra kỹ hơn, giả thuyết đó bị chứng minh là sai. Nhóm nguyên nhân gốc gãi đầu và tiếp tục cuộc tìm kiếm cho bug lẩn tránh.

### Lốc khôi phục đầu tiên

Một khi nhóm khôi phục đã xác định các băng backup, lốc khôi phục đầu tiên khởi động vào ngày 8 tháng 3. Việc yêu cầu 1,5 petabyte dữ liệu phân phối trên hàng nghìn băng từ lưu trữ offsite là một chuyện, nhưng việc trích xuất dữ liệu từ các băng là một chuyện hoàn toàn khác. Stack phần mềm backup băng tự chế không được thiết kế để xử lý một thao tác khôi phục đơn lẻ có kích thước lớn như vậy, nên lần khôi phục ban đầu được chia thành 5.475 tác vụ khôi phục. Một người vận hành gõ một lệnh khôi phục mỗi phút sẽ mất hơn ba ngày để yêu cầu nhiều lần khôi phục như vậy, và bất kỳ người vận hành nào chắc chắn sẽ mắc nhiều sai lầm. Chỉ riêng việc yêu cầu khôi phục từ hệ thống backup băng cần SRE phát triển một giải pháp theo chương trình.<sup>[11](#fn11)</sup>

Đến nửa đêm ngày 9 tháng 3, Music SRE hoàn tất việc yêu cầu cả 5.475 lần khôi phục. Hệ thống backup băng bắt đầu thực hiện phép thuật của nó. Bốn giờ sau, nó phun ra một danh sách 5.337 băng backup cần gọi về từ các vị trí offsite. Trong tám giờ nữa, các băng đến một datacenter trong một loạt các lần giao bằng xe tải.

Trong khi các xe tải đang trên đường, các kỹ thuật viên datacenter đã hạ một số thư viện băng xuống để bảo trì và loại bỏ hàng nghìn băng để nhường chỗ cho chiến dịch khôi phục dữ liệu khổng lồ. Sau đó, các kỹ thuật viên bắt đầu tỉ mỉ nạp các băng bằng tay khi hàng nghìn băng đến vào những giờ sáng sớm. Trong các bài tập DiRT trước đây, quá trình thủ công này đã chứng minh là nhanh hơn hàng trăm lần so với các phương pháp dựa trên robot được cung cấp bởi các nhà cung cấp thư viện băng cho các lần khôi phục khổng lồ. Trong vòng ba giờ, các thư viện hoạt động lại, quét các băng và thực hiện hàng nghìn tác vụ khôi phục lên lưu trữ tính toán phân tán.

Bất chấp kinh nghiệm DiRT của nhóm, cuộc khôi phục khổng lồ 1,5 petabyte mất lâu hơn hai ngày ước tính. Đến sáng ngày 10 tháng 3, chỉ 74% trong số 436.223 file âm thanh đã được chuyển thành công từ 3.475 băng backup được gọi về đến lưu trữ filesystem phân tán tại một cụm tính toán gần đó. 1.862 băng backup khác đã bị loại khỏi quy trình gọi băng bởi một nhà cung cấp. Ngoài ra, quy trình khôi phục đã bị giữ lại bởi 17 băng hỏng. Dự phòng cho một sự hỏng hóc do băng hỏng, một mã hóa dự phòng (redundant encoding) đã được sử dụng để ghi các file backup. Các lần giao bằng xe tải bổ sung được xuất phát để gọi về các băng dự phòng, cùng với 1.862 băng khác đã bị loại bỏ bởi lần gọi offsite đầu tiên.

Đến sáng ngày 11 tháng 3, hơn 99,95% thao tác khôi phục đã hoàn thành, và việc gọi về các băng dự phòng bổ sung cho các file còn lại đang diễn ra. Mặc dù dữ liệu đã an toàn trên các filesystem phân tán, các bước khôi phục dữ liệu bổ sung vẫn cần thiết để làm cho chúng có thể truy cập được bởi người dùng. Nhóm Google Music bắt đầu thực hành các bước cuối cùng này của quy trình khôi phục dữ liệu song song trên một mẫu nhỏ các file âm thanh đã khôi phục để đảm bảo quy trình vẫn hoạt động như mong đợi.

Vào khoảnh khắc đó, các máy gọi trực của hệ thống production Google Music reo lên do một sự cố production ảnh hưởng người dùng nghiêm trọng nhưng không liên quan—một sự cố đã kéo toàn bộ nhóm Google Music vào trong hai ngày. Nỗ lực khôi phục dữ liệu được tiếp tục vào ngày 13 tháng 3, khi tất cả 436.223 track âm thanh một lần nữa có thể truy cập được bởi người dùng của chúng. Chỉ trong chưa đầy 7 ngày, 1,5 petabyte dữ liệu âm thanh đã được tái lập cho người dùng với sự giúp đỡ của các bản backup băng offsite; 5 trong số 7 ngày cấu thành nên nỗ lực khôi phục dữ liệu thực.

### Lốc khôi phục thứ hai

Với lốc khôi phục đầu tiên nằm lại phía sau, nhóm chuyển trọng tâm sang 161.000 file âm thanh thiếu còn lại đã bị bug xóa trước khi chúng được backup. Phần lớn các file này là các track mua từ cửa hàng và quảng bá, và các bản sao cửa hàng gốc không bị ảnh hưởng bởi bug. Những track này được nhanh chóng tái lập để những người dùng bị ảnh hưởng có thể lại thưởng thức nhạc của họ.

Tuy nhiên, một phần nhỏ trong số 161.000 file âm thanh đã được chính người dùng tải lên. Nhóm Google Music đã yêu cầu server của họ yêu cầu các client Google Music của những người dùng bị ảnh hưởng tải lại các file có ngày từ 14 tháng 3 trở đi. Quá trình này kéo dài hơn một tuần. Như vậy kết thúc nỗ lực khôi phục hoàn chỉnh cho sự cố.

### Giải quyết nguyên nhân gốc

Cuối cùng, nhóm Google Music đã xác định được khiếm khuyết trong pipeline xóa dữ liệu đã refactor của họ. Để hiểu khiếm khuyết này, trước tiên bạn cần ngữ cảnh về cách các [hệ thống xử lý dữ liệu](https://sre.google/sre-book/data-processing-pipelines/) offline tiến hóa ở quy mô lớn.

Đối với một dịch vụ lớn và phức tạp bao gồm một số hệ thống phụ và dịch vụ lưu trữ, ngay cả một tác vụ đơn giản như xóa dữ liệu đã xóa cũng cần được thực hiện qua các giai đoạn, mỗi giai đoạn liên quan đến các datastore khác nhau.

Để xử lý dữ liệu hoàn thành nhanh chóng, việc xử lý được song song hóa để chạy trên hàng chục nghìn máy gây ra một tải lớn lên các hệ thống phụ khác nhau. Sự phân phối này có thể làm chậm dịch vụ cho người dùng, hoặc gây ra dịch vụ sụp đổ dưới tải nặng.

Để tránh những kịch bản không mong muốn này, các kỹ sư cloud computing thường tạo một bản sao ngắn hạn của dữ liệu trên lưu trữ thứ cấp, nơi việc xử lý dữ liệu sau đó được thực hiện. Nếu tuổi tương đối của các bản sao thứ cấp của dữ liệu không được phối hợp cẩn thận, thực hành này đưa đến các điều kiện đua (race conditions).

Ví dụ, hai giai đoạn của một pipeline có thể được thiết kế để chạy theo đúng thứ tự, cách nhau ba giờ, để giai đoạn thứ hai có thể đưa ra một giả định đơn giản hóa về tính đúng đắn của đầu vào của nó. Nếu không có giả định đơn giản hóa này, logic của giai đoạn thứ hai có thể khó song song hóa. Nhưng các giai đoạn có thể mất lâu hơn để hoàn thành khi lượng dữ liệu tăng lên. Cuối cùng, các giả định thiết kế ban đầu có thể không còn đúng cho một số dữ liệu nhất định cần thiết cho giai đoạn thứ hai.

Ban đầu, điều kiện đua này có thể xảy ra cho một phần nhỏ dữ liệu. Nhưng khi lượng dữ liệu tăng lên, một phần ngày càng lớn dữ liệu có nguy cơ kích hoạt một điều kiện đua. Một kịch bản như vậy là xác suất—pipeline hoạt động đúng cho phần lớn dữ liệu và cho phần lớn thời gian. Khi những điều kiện đua như vậy xảy ra trong một pipeline xóa dữ liệu, dữ liệu sai có thể bị xóa một cách phi tất định (nondeterministic).

Pipeline xóa dữ liệu của Google Music được thiết kế với sự phối hợp và các biên độ lỗi lớn được đặt sẵn. Nhưng khi các giai đoạn phía trước của pipeline bắt đầu yêu cầu thời gian tăng lên khi dịch vụ phát triển, các tối ưu hóa hiệu suất được đặt vào để Google Music có thể tiếp tục đáp ứng các yêu cầu về quyền riêng tư. Kết quả là, xác suất của một điều kiện đua xóa dữ liệu vô tình trong pipeline này bắt đầu tăng lên. Khi pipeline được refactor, xác suất này một lần nữa tăng đáng kể, đến một mức mà các điều kiện đua xảy ra thường xuyên hơn.

Trong hậu quả của nỗ lực khôi phục, Google Music đã thiết kế lại pipeline xóa dữ liệu của mình để loại bỏ loại điều kiện đua này. Ngoài ra, chúng tôi nâng cao các hệ thống giám sát và cảnh báo production để phát hiện các bug xóa tràn lan quy mô lớn tương tự với mục tiêu phát hiện và sửa các vấn đề như vậy trước khi người dùng chú ý đến bất kỳ vấn đề nào.<sup>[12](#fn12)</sup>

## Các Nguyên Tắc Chung của SRE Được Áp Dụng cho Data Integrity (General Principles of SRE as Applied to Data Integrity)

Các nguyên tắc chung của SRE có thể được áp dụng cho các chi tiết của data integrity và cloud computing như được mô tả trong phần này.

### Tâm Trí Mới (Beginner's Mind)

Các dịch vụ phức tạp quy mô lớn có các bug vốn có không thể hiểu trọn vẹn. Đừng bao giờ nghĩ rằng bạn hiểu đủ một hệ thống phức tạp để nói rằng nó sẽ không hỏng theo một cách nhất định. Hãy tin tưởng nhưng hãy xác minh, và áp dụng phòng thủ nhiều lớp. (Lưu ý: "Tâm trí mới" *không* gợi ý việc đặt một nhân viên mới chịu trách nhiệm cho pipeline xóa dữ liệu đó!)

### Tin Tưởng Nhưng Hãy Xác Minh (Trust but Verify)

Bất kỳ API nào mà bạn phụ thuộc sẽ không hoạt động hoàn hảo *tất cả* thời gian. Đó là điều hiển nhiên rằng bất kể chất lượng kỹ thuật của bạn hay sự nghiêm ngặt của kiểm thử, API sẽ có các khiếm khuyết. Hãy kiểm tra tính đúng đắn của các yếu tố quan trọng nhất dữ liệu của bạn bằng các bộ xác thực dữ liệu ngoài vòng, ngay cả khi ngữ nghĩa API gợi ý rằng bạn không cần làm như vậy. Các thuật toán hoàn hảo có thể không có các triển khai hoàn hảo.

### Hy Vọng Không Phải Là Một Chiến Lược (Hope Is Not a Strategy)

Các thành phần hệ thống không được liên tục sử dụng sẽ hỏng khi bạn cần chúng nhất. Hãy chứng minh rằng khôi phục dữ liệu hoạt động bằng việc tập luyện thường xuyên, nếu không khôi phục dữ liệu sẽ không hoạt động. Con người thiếu kỷ luật để liên tục sử dụng các thành phần hệ thống, nên tự động hóa là bạn của bạn. Tuy nhiên, khi bạn bố trí các nỗ lực tự động hóa này với các kỹ sư có các ưu tiên cạnh tranh, bạn có thể kết thúc với những biện pháp tạm thời.

### Phòng Thủ Nhiều Lớp (Defense in Depth)

Ngay cả hệ thống chống đạn nhất cũng dễ bị tổn thương trước các bug và lỗi vận hành. Để các vấn đề data integrity có thể sửa chữa được, các dịch vụ phải phát hiện những vấn đề đó một cách nhanh chóng. Mọi chiến lược cuối cùng đều thất bại trong các môi trường thay đổi. Các chiến lược data integrity tốt nhất là đa tầng—nhiều chiến lược dựa dẫm vào lẫn nhau và cùng nhau giải quyết một dải rộng các kịch bản với chi phí hợp lý.

### Xem Xét Lại và Kiểm Tra Lại (Revisit and Reexamine)

Thực tế rằng dữ liệu của bạn "đã an toàn hôm qua" sẽ không giúp bạn ngày mai, hoặc thậm chí hôm nay. Các hệ thống và hạ tầng thay đổi, và bạn phải chứng minh rằng các giả định và quy trình của bạn vẫn có liên quan trước sự tiến bộ. Hãy cân nhắc điều sau đây.

Dịch vụ Shakespeare đã nhận được khá nhiều báo chí tích cực, và cơ sở người dùng của nó đang tăng đều. Không có sự chú ý thực sự nào được dành cho data integrity khi dịch vụ được xây dựng. Tất nhiên, chúng ta không muốn phục vụ các bit *xấu*, nhưng nếu Bigtable chỉ mục bị mất, chúng ta có thể dễ dàng tạo lại nó từ các văn bản Shakespeare gốc và một MapReduce. Việc làm đó sẽ mất rất ít thời gian, nên chúng ta chưa bao giờ tạo backup của chỉ mục.

Bây giờ một tính năng mới cho phép người dùng tạo các chú thích văn bản. Đột nhiên, bộ dữ liệu của chúng ta không còn có thể tạo lại dễ dàng, trong khi dữ liệu người dùng ngày càng có giá trị hơn đối với người dùng của chúng ta. Do đó, chúng ta cần xem xét lại các tùy chọn replication của mình—chúng ta không chỉ replicate cho latency và bandwidth, mà còn cho data integrity, nữa. Do đó, chúng ta cần tạo và kiểm tra một thủ tục backup và khôi phục. Thủ tục này cũng được định kỳ kiểm tra bởi một bài tập DiRT để đảm bảo rằng chúng ta có thể khôi phục các chú thích của người dùng từ các bản backup trong khoảng thời gian được đặt bởi SLO.

## Kết Luận (Conclusion)

Data availability phải là mối quan tâm hàng đầu của bất kỳ hệ thống lấy dữ liệu làm trung tâm. Thay vì tập trung vào phương tiện dẫn đến mục đích, Google SRE thấy hữu ích khi học một trang từ phát triển hướng kiểm thử (test-driven development) bằng cách chứng minh rằng các hệ thống của chúng tôi có thể duy trì data availability với một thời gian ngừng tối đa được dự đoán. Các phương tiện và cơ chế mà chúng tôi sử dụng để đạt được mục tiêu cuối cùng này là những điều ác cần thiết. Bằng cách giữ đôi mắt trên mục tiêu, chúng tôi tránh rơi vào cái bẫy mà "Ca phẫu thuật thành công, nhưng hệ thống đã chết."

Nhận ra rằng không chỉ *bất cứ thứ gì* có thể sai, mà là *mọi thứ* sẽ sai là một bước đáng kể hướng tới sự chuẩn bị cho bất kỳ tình trạng khẩn cấp thực sự nào. Một ma trận của tất cả các tổ hợp thảm họa có thể với các kế hoạch để giải quyết mỗi thảm họa trong số này cho phép bạn ngủ ngon ít nhất một đêm; giữ các kế hoạch khôi phục của bạn cập nhật và được tập luyện cho phép bạn ngủ 364 đêm còn lại trong năm.

Khi bạn trở nên giỏi hơn trong việc khôi phục từ bất kỳ hư hỏng nào trong một khoảng thời gian hợp lý *N*, hãy tìm cách mài nhỏ thời gian đó thông qua phát hiện mất mát nhanh hơn và chi tiết hơn, với mục tiêu tiếp cận N = 0. Sau đó bạn có thể chuyển từ việc lập kế hoạch khôi phục sang lập kế hoạch phòng ngừa, với mục tiêu đạt được chiếc cốc thánh (holy grail) của *tất cả dữ liệu, mọi lúc*. Đạt được mục tiêu này, và bạn có thể ngủ trên bãi biển trong kỳ nghỉ đáng mong đợi đó.

<a id="fn1"></a>[1](#fn1) Atomicity (tính nguyên tử), Consistency (tính nhất quán), Isolation (tính cô lập), Durability (tính bền vững); xem [*https://en.wikipedia.org/wiki/ACID*](https://en.wikipedia.org/wiki/ACID). Các cơ sở dữ liệu SQL như MySQL và PostgreSQL hướng tới việc đạt được các tính chất này.
<a id="fn2"></a>[2](#fn2) Basically Available (khả dụng cơ bản), Soft state (trạng thái mềm), Eventual consistency (nhất quán cuối cùng); xem [*https://en.wikipedia.org/wiki/Eventual_consistency*](https://en.wikipedia.org/wiki/Eventual_consistency). Các hệ thống BASE, như Bigtable và Megastore, thường cũng được mô tả là "NoSQL."
<a id="fn3"></a>[3](#fn3) Để đọc thêm về các API ACID và BASE, xem [[Gol14]](https://sre.google/sre-book/bibliography#Gol14) và [[Bai13]](https://sre.google/sre-book/bibliography#Bai13).
<a id="fn4"></a>[4](#fn4) Binary Large Object (đối tượng nhị phân lớn); xem [*https://en.wikipedia.org/wiki/Binary_large_object*](https://en.wikipedia.org/wiki/Binary_large_object).
<a id="fn5"></a>[5](#fn5) Xem [*https://en.wikipedia.org/wiki/Zero-day_(computing)*](https://en.wikipedia.org/wiki/Zero-day_(computing)).
<a id="fn6"></a>[6](#fn6) Các phiến đất nung là những ví dụ viết lách cổ nhất được biết đến. Đối với một thảo luận rộng hơn về việc bảo quản dữ liệu trong dài hạn, xem [[Con96]](https://sre.google/sre-book/bibliography#Con96).
<a id="fn7"></a>[7](#fn7) Khi đọc lời khuyên này, một người có thể hỏi: vì bạn phải cung cấp một API trên datastore để triển khai xóa mềm, tại sao chỉ dừng ở xóa mềm, trong khi bạn có thể cung cấp nhiều tính năng khác bảo vệ chống lại việc xóa nhầm dữ liệu bởi người dùng? Để dẫn ra một ví dụ cụ thể từ kinh nghiệm của Google, hãy xem xét Blobstore: thay vì cho phép khách hàng xóa dữ liệu và metadata Blob trực tiếp, các API Blob triển khai nhiều tính năng an toàn, bao gồm các chính sách backup mặc định (các bản replica offline), các checksum (bảng băm kiểm tra) end-to-end, và thời hạn tồn tại mặc định của các tombstone (bia mộ) (xóa mềm). Hóa ra là vào nhiều dịp, xóa mềm đã cứu các client của Blobstore khỏi data loss có thể đã tồi tệ hơn rất, rất nhiều. Chắc chắn có nhiều tính năng bảo vệ chống xóa đáng gọi tên, nhưng đối với các công ty có các hạn chót xóa dữ liệu bắt buộc, xóa mềm là sự bảo vệ liên quan nhất chống lại các bug và việc xóa nhầm trong trường hợp các client của Blobstore.
<a id="fn8"></a>[8](#fn8) "Snapshot" ở đây đề cập đến một góc nhìn chỉ đọc, tĩnh của một instance lưu trữ, như các snapshot của các cơ sở dữ liệu SQL. Các snapshot thường được triển khai sử dụng ngữ nghĩa copy-on-write (chép-để-viết) cho hiệu quả lưu trữ. Chúng có thể đắt đỏ vì hai lý do: thứ nhất, chúng cạnh tranh cùng dung lượng lưu trữ với các datastore trực tiếp, và thứ hai, dữ liệu của bạn càng biến đổi nhanh, hiệu quả càng ít được đạt được từ việc copy-on-write.
<a id="fn9"></a>[9](#fn9) Để biết thêm thông tin về replication kiểu GFS, xem [[Ghe03]](https://sre.google/sre-book/bibliography#Ghe03). Để biết thêm thông tin về mã xóa Reed-Solomon, xem [*https://en.wikipedia.org/wiki/Reed–Solomon_error_correction*](https://en.wikipedia.org/wiki/Reed–Solomon_error_correction).
<a id="fn10"></a>[10](#fn10) Xem [*https://en.wikipedia.org/wiki/Catch-22_(logic)*](https://en.wikipedia.org/wiki/Catch-22_(logic)).
<a id="fn11"></a>[11](#fn11) Trên thực tế, việc tìm ra một giải pháp theo chương trình không phải là một trở ngại vì phần lớn các SRE là những kỹ sư phần mềm giàu kinh nghiệm, như trong trường hợp này. Kỳ vọng về kinh nghiệm như vậy làm cho các SRE trở nên nổi tiếng là khó tìm và thuê, và từ trường hợp nghiên cứu này và các điểm dữ liệu khác, bạn có thể bắt đầu đánh giá tại sao SRE thuê các kỹ sư phần mềm đang hành nghề; xem [[Jon15]](https://sre.google/sre-book/bibliography#Jon15).
<a id="fn12"></a>[12](#fn12) Theo kinh nghiệm của chúng tôi, các kỹ sư cloud computing thường e dè trong việc thiết lập các cảnh báo production cho các tỷ lệ xóa dữ liệu do sự biến đổi tự nhiên của tỷ lệ xóa dữ liệu theo người dùng theo thời gian. Tuy nhiên, vì ý định của một cảnh báo như vậy là phát hiện các bất thường tỷ lệ xóa toàn cục chứ không phải cục bộ, sẽ hữu ích hơn để cảnh báo khi tỷ lệ xóa dữ liệu toàn cục, được tổng hợp trên tất cả người dùng, vượt qua một ngưỡng cực đoan (như 10 lần bách phân vị thứ 95 được quan sát), so với các cảnh báo tỷ lệ xóa theo người dùng ít hữu ích hơn.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
