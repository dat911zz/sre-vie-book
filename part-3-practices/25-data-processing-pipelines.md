> **Nguyên bản:** [Chapter 25 - Data Processing Pipelines](https://sre.google/sre-book/data-processing-pipelines/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

## Chương 25. Các Pipeline Xử lý Dữ liệu (Data Processing Pipelines)

*Tác giả:* Dan Dennison
*Biên tập:* Tim Harvey

Chương này đi sâu vào những thách thức thực tế khi quản lý các pipeline (đường ống) xử lý dữ liệu có chiều sâu và độ phức tạp cao. Nội dung bao quát phổ tần suất từ các pipeline định kỳ chạy rất ít thường xuyên cho đến những pipeline liên tục không bao giờ ngừng, đồng thời phân tích các bước nhảy (discontinuities) có thể dẫn đến những vấn đề vận hành nghiêm trọng. Một cách tiếp cận mới với mô hình leader-follower được giới thiệu như một giải pháp thay thế đáng tin cậy hơn và có khả năng mở rộng tốt hơn so với pipeline định kỳ trong việc xử lý Big Data.

## Nguồn gốc của Mẫu thiết kế Pipeline (Origin of the Pipeline Design Pattern)

Cách tiếp cận cổ điển trong xử lý dữ liệu là viết một chương trình đọc dữ liệu đầu vào, biến đổi theo ý muốn và xuất dữ liệu mới. Thông thường, chương trình này được lên lịch chạy dưới sự điều khiển của một [chương trình lên lịch định kỳ](https://sre.google/sre-book/distributed-periodic-scheduling/) như cron. Mẫu thiết kế này được gọi là *data pipeline* (pipeline dữ liệu). Các data pipeline có lịch sử lâu đời, bắt đầu từ co-routines [[Con63]](https://sre.google/sre-book/bibliography/#Con63), các file truyền thông DTSS [[Bul80]](https://sre.google/sre-book/bibliography/#Bul80), pipe của UNIX [[McI86]](https://sre.google/sre-book/bibliography/#McI86), và sau đó là các pipeline ETL (extract, transform, load — trích xuất, biến đổi, tải),<sup>[1](#fn1)</sup>. Tuy nhiên, các pipeline này chỉ thực sự thu hút sự chú ý khi "Big Data" (dữ liệu lớn) trỗi dậy, hay nói cách khác, khi xuất hiện "các tập dữ liệu lớn và phức tạp đến mức các ứng dụng xử lý dữ liệu truyền thống không đủ khả năng xử lý".<sup>[2](#fn2)</sup>

## Tác động Ban đầu của Big Data lên Mẫu Pipeline Đơn giản (Initial Effect of Big Data on the Simple Pipeline Pattern)

Các chương trình thực hiện các phép biến đổi định kỳ hoặc liên tục trên Big Data thường được gọi là "pipeline đơn giản, một pha" (simple, one-phase pipelines).

Do quy mô và độ phức tạp vốn có của Big Data, các chương trình thường được tổ chức thành chuỗi nối tiếp, trong đó đầu ra của chương trình này là đầu vào của chương trình kế tiếp. Cách sắp xếp này có thể xuất phát từ nhiều lý do, nhưng chủ yếu nhằm mục đích dễ suy luận về hệ thống hơn là tối ưu hiệu quả vận hành. Các chương trình được tổ chức theo mô hình này gọi là *multiphase pipelines* (pipeline nhiều pha), vì mỗi chương trình trong chuỗi đóng vai trò như một pha xử lý dữ liệu rời rạc.

Số lượng chương trình được nối tiếp trong một pipeline chính là *chiều sâu* (depth) của pipeline đó. Vì vậy, một pipeline nông có thể chỉ gồm một chương trình, tương ứng với chiều sâu bằng một, trong khi một pipeline sâu có thể đạt chiều sâu lên đến hàng chục hoặc hàng trăm chương trình.

## Những Thách thức của Mẫu Pipeline Định kỳ (Challenges with the Periodic Pipeline Pattern)

Các pipeline định kỳ nhìn chung ổn định khi có đủ worker cho lượng dữ liệu và nhu cầu thực thi nằm trong khả năng tính toán. Ngoài ra, các bất ổn như nút thắt cổ chai (bottleneck) xử lý được tránh khi số lượng các job được nối tiếp và thông lượng tương đối giữa các job vẫn đồng đều.

Các pipeline định kỳ này hữu ích và thực tế, nên chúng tôi chạy chúng thường xuyên tại Google. Chúng được viết bằng các framework như MapReduce [[Dea04]](https://sre.google/sre-book/bibliography/#Dea04) và Flume [[Cha10]](https://sre.google/sre-book/bibliography/#Cha10), cùng nhiều framework khác.

Tuy nhiên, kinh nghiệm tổng hợp của các SRE (Site Reliability Engineer) cho thấy mô hình pipeline định kỳ rất dễ vỡ. Chúng tôi nhận thấy rằng khi một pipeline định kỳ lần đầu được cài đặt với việc xác định kích thước worker, tính định kỳ, kỹ thuật chia chunk và các tham số khác được tinh chỉnh cẩn thận, hiệu năng ban đầu là đáng tin cậy. Nhưng sự tăng trưởng và thay đổi hữu cơ không thể tránh khỏi bắt đầu gây áp lực lên hệ thống, dẫn đến các vấn đề phát sinh. Các ví dụ về những vấn đề như vậy bao gồm các job vượt quá hạn chót chạy, cạn kiệt tài nguyên, và các chunk xử lý treo kéo theo tải vận hành tương ứng.

## Khó khăn Gây ra bởi Sự Phân phối Công việc Không đồng đều (Trouble Caused By Uneven Work Distribution)

Đột phá then chốt của Big Data là việc áp dụng rộng rãi các thuật toán "song song hoàn toàn (không phụ thuộc lẫn nhau)" (embarrassingly parallel) [[Mol86]](https://sre.google/sre-book/bibliography/#Mol86) để chia nhỏ khối lượng công việc lớn thành các chunk vừa sức với từng máy riêng lẻ. Tuy nhiên, các chunk đôi khi đòi hỏi lượng tài nguyên không đồng đều, và ban đầu thường khó nhận ra nguyên nhân khiến chúng khác nhau về mức độ tài nguyên. Chẳng hạn, trong một khối lượng công việc được phân vùng theo khách hàng, dữ liệu của một số khách hàng có thể lớn hơn hẳn so với các chunk còn lại. Do khách hàng là đơn vị không thể chia nhỏ hơn, thời gian chạy đầu-cuối vì thế bị giới hạn bởi thời gian chạy của khách hàng lớn nhất.

Vấn đề "chunk treo" (hanging chunk) có thể xảy ra khi tài nguyên được phân bổ do sự khác biệt giữa các máy trong một cluster (cụm máy) hoặc do phân bổ quá mức cho một job. Nguyên nhân là một số thao tác thời gian thực trên các stream, chẳng hạn như sắp xếp dữ liệu "đang nóng", gặp nhiều khó khăn. Người dùng thường chờ cho đến khi toàn bộ phép tính hoàn thành trước khi chuyển sang giai đoạn pipeline tiếp theo, đặc biệt khi giai đoạn đó liên quan đến sắp xếp, vì thao tác này đòi hỏi toàn bộ dữ liệu phải đi qua. Điều này có thể làm trì hoãn đáng kể thời gian hoàn thành pipeline, bởi tiến độ bị chặn bởi hiệu năng trường hợp xấu nhất do phương pháp chia chunk đang sử dụng quy định.

Nếu các kỹ sư hoặc hệ thống giám sát cluster phát hiện ra vấn đề này, phản ứng của họ có thể khiến tình hình trở nên tồi tệ hơn. Chẳng hạn, cách xử lý "hợp lý" hoặc "mặc định" khi một chunk bị treo là giết job ngay lập tức rồi cho phép job khởi động lại, vì sự tắc nghẽn có thể chính là hệ quả của các yếu tố không xác định. Tuy nhiên, do các triển khai pipeline về thiết kế thường không bao gồm checkpointing, toàn bộ công việc trên các chunk bị khởi động lại từ đầu, dẫn đến lãng phí thời gian, chu kỳ CPU và công sức con người đã bỏ ra trong chu kỳ trước đó.

## Nhược điểm của Pipeline Định kỳ trong Môi trường Phân tán (Drawbacks of Periodic Pipelines in Distributed Environments)

Tại Google, các pipeline Big Data định kỳ được sử dụng rất phổ biến, nên hệ thống quản lý cluster của công ty tích hợp sẵn cơ chế lên lịch thay thế cho chúng. Cơ chế này cần thiết vì, khác với các pipeline chạy liên tục, pipeline định kỳ thường hoạt động dưới dạng job batch có độ ưu tiên thấp. Việc gán độ ưu tiên thấp phù hợp ở đây do công việc batch không nhạy cảm với độ trễ như các dịch vụ web hướng Internet. Hơn nữa, để kiểm soát chi phí bằng cách tối đa hóa tải trên máy, Borg (hệ thống quản lý cluster của Google, [[Ver15]](https://sre.google/sre-book/bibliography/#Ver15)) phân bổ job batch cho các máy khả dụng. Độ ưu tiên này có thể khiến thời gian khởi động kéo dài, dẫn đến việc các job pipeline phải chịu sự trì hoãn khởi đầu không giới hạn.

Các job chạy theo cơ chế này chịu một số giới hạn tự nhiên, dẫn đến những hành vi khác nhau. Chẳng hạn, nếu được lên lịch vào các khoảng trống do các job dịch vụ web hướng người dùng để lại, chúng có thể bị ảnh hưởng về khả dùng tài nguyên độ trễ thấp, chi phí và độ ổn định khi truy cập tài nguyên. Chi phí thực thi tỷ lệ nghịch với độ trễ khởi động yêu cầu và tỷ lệ thuận với lượng tài nguyên tiêu thụ. Dù lên lịch batch thường hoạt động suôn sẻ trên thực tế, việc sử dụng quá mức bộ lên lịch batch ([Lên lịch Định kỳ Phân tán bằng Cron](https://sre.google/sre-book/distributed-periodic-scheduling/)) khiến các job đối mặt rủi ro bị thu hồi sớm (xem mục 2.5 của [[Ver15]](https://sre.google/sre-book/bibliography/#Ver15)) khi tải cluster cao, do các user khác đã cạn tài nguyên batch. Xét về các đánh đổi rủi ro, việc chạy thành công một pipeline định kỳ được tinh chỉnh tốt là sự cân bằng tinh tế giữa chi phí tài nguyên cao và rủi ro bị thu hồi sớm.

Với các pipeline chạy hàng ngày, độ trễ lên đến vài giờ hoàn toàn có thể chấp nhận được. Tuy nhiên, khi tần suất thực thi đã lên lịch tăng lên, thời gian tối thiểu giữa các lần thực thi có thể nhanh chóng chạm tới điểm độ trễ trung bình tối thiểu, tạo thành một cận dưới cho độ trễ mà một pipeline định kỳ có thể kỳ vọng đạt được. Việc rút ngắn khoảng thời gian thực thi job xuống dưới cận dưới hiệu dụng này đơn giản chỉ dẫn đến hành vi không mong muốn chứ không phải tiến bộ tăng thêm. Failure mode cụ thể phụ thuộc vào chính sách lên lịch batch đang được sử dụng. Ví dụ, mỗi lần chạy mới có thể chồng chất lên bộ lên lịch cluster vì lần chạy trước chưa hoàn thành. Tệ hơn, lần chạy đang thực thi và gần như hoàn thành có thể bị giết khi lần thực thi tiếp theo được lên lịch bắt đầu, kết quả là dù chạy nhiều lần hơn, tiến độ thực tế vẫn hoàn toàn đình trệ.

Hãy chú ý đến vị trí đường dốc thể hiện khoảng thời gian nhàn rỗi cắt ngang đường độ trễ lên lịch trong [Hình 25-1](#hinh-25-1). Trong kịch bản này, việc rút ngắn khoảng thời gian thực thi xuống dưới 40 phút cho job ~20 phút này có thể khiến các lần thực thi chồng lấp lên nhau, gây ra những hệ quả không mong muốn.

<a id="hinh-25-1"></a>        ![Khoảng thời gian thực thi của pipeline định kỳ so với thời gian nhàn rỗi (thang log)](../assets/imgs/fig-25-1.jpg)

Hình 25-1. Khoảng thời gian thực thi của pipeline định kỳ so với thời gian nhàn rỗi (thang log)

Giải pháp cho vấn đề này là đảm bảo đủ sức chứa server để hệ thống vận hành đúng cách. Tuy nhiên, trong môi trường phân tán chia sẻ, việc thu thập tài nguyên phải tuân theo quy luật cung – cầu. Như dự kiến, các đội phát triển thường miễn cưỡng thực hiện quy trình thu thập tài nguyên khi phải đóng góp vào một pool chung. Để giải quyết điều này, cần phân biệt giữa tài nguyên lên lịch batch và tài nguyên độ ưu tiên production nhằm hợp lý hóa chi phí thu thập tài nguyên.

## Các Vấn đề Giám sát trong Pipeline Định kỳ (Monitoring Problems in Periodic Pipelines)

Với các pipeline có thời gian thực thi đủ dài, việc nắm được các chỉ số hiệu năng theo thời gian thực có thể quan trọng, thậm chí quan trọng hơn, so với việc biết các chỉ số tổng thể. Nguyên nhân là dữ liệu thời gian thực đóng vai trò then chốt trong việc hỗ trợ vận hành, bao gồm cả ứng phó khẩn cấp. Thực tế, mô hình giám sát chuẩn thường chỉ thu thập các chỉ số trong quá trình thực thi job và báo cáo chúng khi job hoàn thành. Do đó, nếu job thất bại trong quá trình thực thi, sẽ không có thống kê nào được cung cấp.

Các pipeline liên tục không gặp phải những vấn đề này, bởi các task của chúng chạy liên tục và telemetry được thiết kế để thường xuyên cung cấp các chỉ số thời gian thực. Các pipeline định kỳ không nên có các vấn đề giám sát vốn có, nhưng chúng tôi đã quan sát thấy một mối liên hệ mạnh.

## Các Vấn đề "Hiệu ứng Bầy đàn" ("Thundering Herd" Problems)

Ngoài các thách thức về thực thi và giám sát, các hệ thống phân tán còn phải đối mặt với vấn đề "hiệu ứng bầy đàn" (thundering herd), một chủ đề cũng được đề cập trong [Lên lịch Định kỳ Phân tán bằng Cron](https://sre.google/sre-book/distributed-periodic-scheduling/). Với một pipeline định kỳ đủ lớn, mỗi chu kỳ có thể kích hoạt hàng nghìn worker chạy đồng thời. Nếu số lượng worker quá nhiều, hoặc do cấu hình sai, hoặc do logic thử lại lỗi khiến chúng được gọi lại, các server nơi chúng chạy, các dịch vụ cluster chia sẻ nền tảng và toàn bộ hạ tầng mạng liên quan sẽ bị quá tải.

Tình hình càng trở nên nghiêm trọng hơn nếu không có logic thử lại: các vấn đề về tính đúng đắn có thể phát sinh khi công việc bị bỏ rơi sau khi thất bại, và job sẽ không được chạy lại. Ngược lại, nếu logic thử lại có mặt nhưng được triển khai một cách ngây thơ hoặc kém hiệu quả, việc thử lại khi thất bại có thể khiến vấn đề trầm trọng hơn.

Sự can thiệp của con người cũng có thể góp phần vào kịch bản này. Những kỹ sư còn hạn chế kinh nghiệm trong việc quản lý pipeline thường khuếch đại vấn đề bằng cách thêm nhiều worker hơn vào pipeline khi job không hoàn thành trong khoảng thời gian mong muốn.

Dù nguyên nhân của hiện tượng “hiệu ứng bầy đàn” là gì, không có gì gây tổn hại cho hạ tầng cluster và các SRE phụ trách các dịch vụ khác nhau trong một cluster bằng một job pipeline 10.000 worker có lỗi.

## Mẫu tải Moiré (Moiré Load Pattern)

Đôi khi, hiệu ứng bầy đàn không dễ nhận ra nếu chỉ xem xét từng trường hợp riêng lẻ. Một vấn đề liên quan mà chúng tôi gọi là "mẫu tải Moiré" (Moiré load pattern) xảy ra khi hai hoặc nhiều pipeline chạy đồng thời và các trình tự thực thi của chúng đôi khi chồng lấp, khiến chúng cùng lúc tiêu thụ một tài nguyên chia sẻ chung. Vấn đề này có thể xuất hiện ngay cả trong các pipeline liên tục, mặc dù nó ít phổ biến hơn khi tải đến đều hơn.

Các mẫu tải Moiré rõ nhất xuất hiện trên các biểu đồ theo dõi tài nguyên chia sẻ của pipeline. Chẳng hạn, [Hình 25-2](#hinh-25-2) cho thấy mức sử dụng tài nguyên của ba pipeline chạy định kỳ. Còn [Hình 25-3](#hinh-25-3) là bản chồng chất dữ liệu từ đồ thị trước, trong đó các đỉnh gây áp lực nặng nề cho nhóm on-call xuất hiện khi tải tổng hợp tiệm cận 1,2M.

<a id="hinh-25-2"></a>        ![Mẫu tải Moiré trên hạ tầng riêng biệt](../assets/imgs/fig-25-2.jpg)

Hình 25-2. Mẫu tải Moiré trên hạ tầng riêng biệt

<a id="hinh-25-3"></a>        ![Mẫu tải Moiré trên hạ tầng chia sẻ](../assets/imgs/fig-25-3.jpg)

Hình 25-3. Mẫu tải Moiré trên hạ tầng chia sẻ

## Giới thiệu Google Workflow (Introduction to Google Workflow)

Khi một pipeline batch vốn chỉ chạy một lần bị áp đảo bởi các yêu cầu kinh doanh về kết quả được cập nhật liên tục, đội phát triển pipeline thường xem xét hoặc tái cấu trúc thiết kế ban đầu để đáp ứng các yêu cầu hiện tại, hoặc chuyển sang mô hình pipeline liên tục. Thật không may, các yêu cầu kinh doanh thường xuất hiện vào thời điểm không thuận tiện nhất để tái cấu trúc hệ thống pipeline thành một hệ thống xử lý liên tục online. Các khách hàng mới và lớn hơn, phải đối mặt với các vấn đề mở rộng bắt buộc, thường cũng muốn đưa vào các tính năng mới, và kỳ vọng rằng các yêu cầu này tuân theo các hạn chót bất di bất dịch. Khi dự đoán thách thức này, việc xác định một số chi tiết ở đầu việc thiết kế một hệ thống liên quan đến một pipeline dữ liệu được đề xuất là quan trọng. Đảm bảo xác định phạm vi quỹ đạo tăng trưởng được kỳ vọng,<sup>[3](#fn3)</sup> nhu cầu về các thay đổi thiết kế, tài nguyên bổ sung được kỳ vọng, và các yêu cầu độ trễ được kỳ vọng từ phía kinh doanh.

Để đáp ứng những nhu cầu này, Google đã phát triển hệ thống "Workflow" vào năm 2003, giúp khả dụng hóa xử lý liên tục ở quy mô lớn. Workflow sử dụng mẫu thiết kế hệ thống phân tán leader-follower (workers) [[Sha00]](https://sre.google/sre-book/bibliography/#Sha00) và mẫu thiết kế system prevalence.<sup>[4](#fn4)</sup> Sự kết hợp này cho phép các [data pipeline](https://sre.google/workbook/data-processing/) xử lý giao dịch ở quy mô rất lớn, đảm bảo tính đúng đắn với ngữ nghĩa chính xác một lần (exactly-once semantics).

## Workflow như Mẫu Model-View-Controller (Workflow as Model-View-Controller Pattern)

Do cách system prevalence hoạt động, có thể hữu ích khi xem Workflow như tương đương hệ thống phân tán của mẫu model-view-controller quen thuộc từ phát triển giao diện người dùng.<sup>[5](#fn5)</sup> Như [Hình 25-4](#hinh-25-4) cho thấy, mẫu thiết kế này chia một ứng dụng phần mềm được cho thành ba phần liên kết với nhau để tách các biểu diễn nội bộ của thông tin khỏi các cách mà thông tin được trình bày đến hoặc nhận từ người dùng.<sup>[6](#fn6)</sup>

<a id="hinh-25-4"></a>        ![Mẫu model-view-controller được sử dụng trong thiết kế giao diện người dùng.](../assets/imgs/fig-25-4.jpg)

Hình 25-4. Mẫu model-view-controller được sử dụng trong thiết kế giao diện người dùng

Áp dụng lại mẫu này cho Workflow, *model* được đặt trên một server có tên "Task Master" (Master của Task). Task Master dùng mẫu system prevalence để giữ toàn bộ trạng thái job trong bộ nhớ nhằm đảm bảo khả năng truy cập nhanh, đồng thời ghi nhật ký đồng bộ các biến đổi (mutations) lên ổ đĩa bền vững. *View* là các worker liên tục cập nhật trạng thái hệ thống theo cách giao dịch với master, dưới góc nhìn của chúng như một thành phần con của pipeline. Mặc dù toàn bộ dữ liệu pipeline có thể lưu trong Task Master, hiệu năng thường đạt mức tốt nhất khi chỉ các pointer đến công việc được lưu ở đây, còn dữ liệu đầu vào và đầu ra thực tế được đặt trong một hệ thống tệp chung hoặc bộ lưu trữ khác. Để hỗ trợ cho sự tương tự này, các worker hoàn toàn không có trạng thái và có thể bị loại bỏ bất kỳ lúc nào. Một *controller* có thể được thêm tùy chọn như thành phần hệ thống thứ ba, nhằm hỗ trợ hiệu quả một số hoạt động phụ trợ ảnh hưởng đến pipeline, chẳng hạn như mở rộng theo thời gian chạy, snapshotting, kiểm soát trạng thái workcycle, đưa trạng thái pipeline quay ngược, hoặc thậm chí thực hiện sự ngăn chặn toàn cục cho tính liên tục kinh doanh. [Hình 25-5](#hinh-25-5) minh họa mẫu thiết kế.

<a id="hinh-25-5"></a>        ![Mẫu thiết kế Model-View-Controller được sử dụng lại cho Google Workflow.](../assets/imgs/fig-25-5.jpg)

Hình 25-5. Mẫu thiết kế Model-View-Controller được sử dụng lại cho Google Workflow

## Các Giai đoạn Thực thi trong Workflow (Stages of Execution in Workflow)

Trong Workflow, chúng ta có thể tăng chiều sâu pipeline lên bất kỳ mức nào bằng cách chia nhỏ việc xử lý thành các nhóm task (task groups) do Task Master quản lý. Mỗi nhóm task đảm nhận công việc của một giai đoạn pipeline và có thể thực hiện các thao tác tùy ý trên một phần dữ liệu. Việc thực hiện mapping, shuffling, sorting, splitting, merging, hay bất kỳ thao tác nào khác ở bất kỳ giai đoạn nào đều khá đơn giản.

Một giai đoạn thường gắn với một số loại worker. Có thể chạy đồng thời nhiều instance của cùng một loại worker, và các worker có thể tự lên lịch, nghĩa là chúng có thể tìm kiếm các loại công việc khác nhau và chọn loại nào để thực hiện.

Worker lấy các đơn vị công việc từ giai đoạn trước để xử lý, sau đó tạo ra các đơn vị đầu ra. Đầu ra này có thể là một điểm cuối hoặc làm đầu vào cho một giai đoạn xử lý khác. Trong hệ thống, việc đảm bảo mọi công việc đều được thực thi, hoặc ít nhất được ghi nhận chính xác một lần vào trạng thái bền vững, là điều dễ dàng.

## Các Cam kết Đảm bảo Tính Đúng đắn của Workflow (Workflow Correctness Guarantees)

Việc lưu *mọi* chi tiết trạng thái pipeline trong Task Master là không thực tế do giới hạn kích thước RAM. Tuy nhiên, hệ thống duy trì cam kết đảm bảo tính đúng đắn kép (double correctness guarantee) nhờ master giữ một tập hợp các pointer đến dữ liệu được đặt tên duy nhất, đồng thời mỗi đơn vị công việc có một lease duy nhất. Các worker thu thập công việc kèm lease và chỉ có thể commit công việc từ những task mà chúng đang nắm giữ lease hợp lệ.

Để tránh trường hợp một worker mồ côi (orphaned worker) tiếp tục xử lý đơn vị công việc và làm hỏng kết quả của worker hiện tại, mỗi file đầu ra do worker mở sẽ có tên duy nhất. Nhờ đó, ngay cả worker mồ côi vẫn có thể ghi độc lập với master cho đến khi chúng cố gắng commit. Khi đó, thao tác commit sẽ thất bại vì một worker khác đang nắm giữ lease cho đơn vị công việc này. Hơn nữa, worker mồ côi không thể phá hủy công việc do worker hợp lệ tạo ra, bởi quy ước đặt tên file duy nhất đảm bảo mỗi worker ghi vào một file riêng biệt. Cách này giúp duy trì tính đúng đắn kép: các file đầu ra luôn duy nhất, và trạng thái pipeline luôn chính xác nhờ các task có lease.

Như thể cam kết đảm bảo tính đúng đắn kép là chưa đủ, Workflow còn đánh phiên bản (version) cho tất cả các task. Khi task cập nhật hoặc lease task thay đổi, mỗi thao tác sẽ tạo ra một task mới duy nhất để thay thế task trước đó, kèm theo một ID mới được gán cho task. Vì toàn bộ cấu hình pipeline trong Workflow được lưu bên trong Task Master dưới cùng một dạng như chính các đơn vị công việc, nên để commit công việc, một worker phải sở hữu một lease hoạt động *và* tham chiếu đến số ID task của cấu hình mà nó đã dùng để tạo ra kết quả. Nếu cấu hình thay đổi trong khi đơn vị công việc đang bay (in flight), tất cả các worker thuộc loại đó sẽ không thể commit, bất kể chúng có đang sở hữu các lease hiện hành hay không. Do đó, mọi công việc được thực hiện sau một thay đổi cấu hình đều nhất quán với cấu hình mới, với cái giá là công việc bị vứt bỏ bởi những worker xui xẻo vẫn đang nắm giữ các lease cũ.

Các biện pháp này tạo thành ba lớp cam kết đảm bảo tính đúng đắn: cấu hình, sở hữu lease và sự duy nhất của tên file. Tuy nhiên, ngay cả điều này cũng không đủ cho mọi trường hợp.

Ví dụ, điều gì sẽ xảy ra nếu địa chỉ mạng của Task Master thay đổi và một Task Master khác thay thế nó tại cùng địa chỉ đó? Hay nếu sự hỏng hóc bộ nhớ làm thay đổi địa chỉ IP hoặc số cổng, khiến một Task Master khác xuất hiện ở đầu bên kia? Thậm chí phổ biến hơn, điều gì xảy ra nếu ai đó cấu hình sai Task Master của mình bằng cách chèn một load balancer trước một tập hợp các Task Master độc lập?

Workflow nhúng một server token (định danh duy nhất cho Task Master cụ thể này) vào metadata của mỗi task nhằm ngăn một Task Master giả mạo hoặc được cấu hình sai làm hỏng pipeline. Cả client và server đều kiểm tra token ở mỗi thao tác, qua đó tránh được một lỗi cấu hình rất tinh vi, trong đó tất cả các thao tác chạy suôn sẻ cho đến khi xảy ra va chạm định danh task.

Tóm lại, bốn cam kết đảm bảo tính đúng đắn của Workflow là:

-   Đầu ra của worker thông qua các task cấu hình tạo ra các rào cản (barriers) làm căn cứ cho công việc tiếp theo.
-   Tất cả công việc được commit đòi hỏi một lease hiện hành hợp lệ được worker nắm giữ.
-   Các file đầu ra được đặt tên duy nhất bởi các worker.
-   Client và server xác thực chính Task Master bằng cách kiểm tra một server token ở mỗi thao tác.

Lúc này, bạn có thể nghĩ rằng sẽ đơn giản hơn nếu bỏ qua Task Master chuyên dụng và dùng Spanner [[Cor12]](https://sre.google/sre-book/bibliography/#Cor12) hoặc một cơ sở dữ liệu khác. Tuy nhiên, Workflow có điểm đặc biệt: mỗi task đều duy nhất và bất biến. Hai thuộc tính này giúp ngăn ngừa nhiều vấn đề tiềm tàng tinh vi khi phân phối công việc ở quy mô lớn.

Ví dụ, lease do worker thu thập là một phần của chính task, nên ngay cả khi chỉ thay đổi lease cũng đòi hỏi phải tạo một task mới hoàn toàn. Nếu dùng database trực tiếp và coi các log giao dịch của nó như một "nhật ký" (journal), thì mọi thao tác đọc, dù lớn hay nhỏ, đều phải nằm trong một giao dịch chạy dài. Cấu hình này chắc chắn khả thi, nhưng vô cùng kém hiệu quả.

## Đảm bảo Tính liên tục Kinh doanh (Ensuring Business Continuity)

Các pipeline Big Data phải duy trì hoạt động bất kể loại sự cố nào, từ đứt cáp quang, biến động thời tiết cho đến sự cố lan truyền trên lưới điện. Những sự cố này có thể khiến toàn bộ datacenter ngừng hoạt động. Hơn nữa, các pipeline không dùng system prevalence để đảm bảo cam kết hoàn thành job thường bị vô hiệu hóa và rơi vào trạng thái không xác định. Lỗ hổng kiến trúc này khiến chiến lược tính liên tục kinh doanh trở nên dễ vỡ, đồng thời kéo theo việc nhân bản ồ ạt các nỗ lực tốn kém nhằm khôi phục pipeline và dữ liệu.

Workflow giải quyết triệt để vấn đề này cho các pipeline xử lý liên tục. Để đảm bảo tính nhất quán toàn cục, Task Master lưu nhật ký trên Spanner, coi đây là một hệ thống tệp toàn cục có tính nhất quán toàn cục nhưng thông lượng thấp. Mỗi Task Master dùng dịch vụ khóa phân tán Chubby [[Bur06]](https://sre.google/sre-book/bibliography/#Bur06) để bầu ra người ghi, sau đó lưu kết quả lên Spanner. Cuối cùng, các client tra cứu Task Master hiện hành thông qua các dịch vụ đặt tên nội bộ.

Vì Spanner không phải là hệ thống tệp thông lượng cao, các Workflow phân tán toàn cục được xây dựng từ hai hoặc nhiều Workflow cục bộ chạy trên các cluster riêng biệt, kết hợp với khái niệm task tham chiếu lưu trong Workflow toàn cục. Khi các đơn vị công việc đi qua pipeline, binary gắn nhãn "stage 1" trong [Hình 25-6](#hinh-25-6) sẽ chèn các task tham chiếu tương ứng vào Workflow toàn cục. Khi các task hoàn thành, hệ thống gỡ bỏ các task tham chiếu này khỏi Workflow toàn cục theo cách giao dịch, như minh họa ở "stage n" của [Hình 25-6](#hinh-25-6). Nếu không thể gỡ bỏ task khỏi Workflow toàn cục, Workflow cục bộ sẽ bị chặn cho đến khi Workflow toàn cục khả dụng trở lại, nhằm đảm bảo tính đúng đắn giao dịch.

Để tự động hóa failover, một binary phụ trợ được gắn nhãn "stage 1" trong [Hình 25-6](#hinh-25-6) chạy bên trong mỗi Workflow cục bộ. Workflow cục bộ không thay đổi, như hộp "do work" trong sơ đồ mô tả. Binary phụ trợ này đóng vai trò "controller" theo nghĩa MVC, chịu trách nhiệm tạo các task tham chiếu và cập nhật một task heartbeat đặc biệt trong Workflow toàn cục. Nếu task heartbeat không được cập nhật trong thời gian chờ, binary phụ trợ của Workflow từ xa sẽ chiếm lấy công việc đang tiến hành (theo ghi nhận của các task tham chiếu) và pipeline tiếp tục, không bị cản trở bởi bất kỳ tác động nào của môi trường lên công việc đó.

<a id="hinh-25-6"></a>        ![Một ví dụ về luồng dữ liệu và quy trình phân tán sử dụng các pipeline Workflow.](../assets/imgs/fig-25-6.jpg)

Hình 25-6. Một ví dụ về luồng dữ liệu và quy trình phân tán sử dụng các pipeline Workflow

## Tóm tắt và Nhận xét Kết luận (Summary and Concluding Remarks)

Các pipeline định kỳ có giá trị. Tuy nhiên, nếu một vấn đề xử lý dữ liệu mang tính liên tục hoặc sẽ phát triển tự nhiên để trở nên liên tục, hãy tránh dùng pipeline định kỳ. Thay vào đó, hãy sử dụng một công nghệ có các đặc tính tương tự Workflow.

Chúng tôi nhận thấy rằng khi xử lý dữ liệu liên tục với các cam kết mạnh do Workflow cung cấp, hệ thống vận hành và mở rộng tốt trên hạ tầng cluster phân tán. Nó thường xuyên cho ra kết quả mà người dùng có thể tin tưởng, đồng thời là một hệ thống ổn định, đáng tin cậy để đội Site Reliability Engineering quản lý và bảo trì.

<a id="fn1"></a>[1](#fn1) Wikipedia: Extract, transform, load, [*https://en.wikipedia.org/wiki/Extract,_transform,_load*](https://en.wikipedia.org/wiki/Extract,_transform,_load)

<a id="fn2"></a>[2](#fn2) Wikipedia: Big data, [*https://en.wikipedia.org/wiki/Big\_data*](https://en.wikipedia.org/wiki/Big_data)

<a id="fn3"></a>[3](#fn3) Bài giảng của Jeff Dean về "Software Engineering Advice from Building Large-Scale Distributed Systems" là một tài nguyên tuyệt vời: [[Dea07]](https://sre.google/sre-book/bibliography/#Dea07).

<a id="fn4"></a>[4](#fn4) Wikipedia: System Prevalence, [*https://en.wikipedia.org/wiki/System\_Prevalence*](https://en.wikipedia.org/wiki/System_Prevalence)

<a id="fn5"></a>[5](#fn5) Mẫu "model-view-controller" là một phép tương tự cho các hệ thống phân tán được vay mượn rất nới lỏng từ Smalltalk, vốn ban đầu được dùng để mô tả cấu trúc thiết kế của các giao diện người dùng đồ họa [[Fow08]](https://sre.google/sre-book/bibliography/#Fow08).

<a id="fn6"></a>[6](#fn6) Wikipedia: Model-view-controller, [*https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller*](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
