# Chương 13. Phản ứng Khẩn cấp (Emergency Response)

> **Nguyên bản:** [Chapter 13 - Emergency Response](https://sre.google/sre-book/emergency-response/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Corey Adam Baye
*Biên tập:* Diane Bates

Hệ thống sẽ hỏng — đó là lẽ thường.

Dù mức độ rủi ro hay quy mô tổ chức lớn nhỏ, điều quyết định sức khỏe lâu dài của một tổ chức — và cũng là điều phân biệt nó với những tổ chức khác — là cách những người trong cuộc phản ứng với một tình huống khẩn cấp. Rất ít người trong chúng ta phản ứng tốt một cách tự nhiên khi gặp tình huống khẩn cấp. Một phản ứng thích hợp đòi hỏi sự chuẩn bị và đào tạo thực hành định kỳ, có liên quan. Thiết lập và duy trì các quy trình đào tạo, kiểm thử đầy đủ đòi hỏi sự hậu thuẫn của hội đồng quản trị và ban quản lý, cùng sự chú tâm của nhân viên. Tất cả những yếu tố này đều cần thiết để nuôi dưỡng một môi trường cho phép các đội chi tiền, thời gian, năng lượng — và đôi khi cả uptime — nhằm đảm bảo rằng hệ thống, quy trình và con người đều phản ứng hiệu quả khi có tình huống khẩn cấp.

Lưu ý rằng chương về văn hóa postmortem (báo cáo sau sự cố) đã thảo luận chi tiết cách viết postmortems, nhằm đảm bảo các incident (sự cố) đòi hỏi phản ứng khẩn cấp cũng trở thành cơ hội học tập (xem [Postmortem Culture: Learning from Failure](https://sre.google/sre-book/postmortem-culture/)). Chương này cung cấp các ví dụ cụ thể hơn về những incident như vậy.

## Phải Làm Gì Khi Các Hệ thống Hỏng (What to Do When Systems Break)

Trước hết, đừng hoảng! Bạn không đơn độc, và bầu trời cũng chẳng sụp. Bạn là chuyên gia, được đào tạo để xử lý đúng loại tình huống này. Thông thường, không ai gặp nguy hiểm về thể chất — chỉ có những electron đáng thương là gặp nguy hiểm. Trong trường hợp tồi tệ nhất, một nửa Internet bị sập. Vì vậy, hãy hít một hơi sâu… và tiếp tục.

Nếu bạn thấy quá tải, hãy kéo thêm người vào. Đôi khi thậm chí cần phải gọi trực (page) toàn bộ công ty. Nếu công ty bạn có quy trình phản ứng sự cố (incident response process) (xem [Managing Incidents](https://sre.google/sre-book/managing-incidents/)), hãy nắm rõ và tuân thủ quy trình đó.

## Tình huống Khẩn cấp do Kiểm thử Gây ra (Test-Induced Emergency)

Google chủ động áp dụng phương pháp kiểm thử thảm họa và khẩn cấp (xem [[Kri12]](https://sre.google/sre-book/bibliography#Kri12)). SRE cố tình làm hỏng hệ thống để quan sát cách chúng thất bại, từ đó thực hiện các thay đổi nhằm nâng cao độ tin cậy và ngăn sự cố tái diễn. Phần lớn thời gian, những sự cố có kiểm soát này diễn ra đúng kế hoạch: hệ thống mục tiêu (target) và các hệ thống phụ thuộc hành xử như mong đợi. Chúng tôi xác định một số điểm yếu hoặc các sự phụ thuộc ẩn, đồng thời ghi tài liệu các hành động tiếp theo để sửa chữa những khiếm khuyết đã phát hiện. Tuy nhiên, đôi khi các giả định của chúng tôi và kết quả thực tế lại là hai thế giới khác nhau.

Dưới đây là một ví dụ về một kiểm thử đã đào lên một số sự phụ thuộc bất ngờ.

## Chi tiết (Details)

Chúng tôi muốn phát hiện các sự phụ thuộc ẩn vào một database kiểm thử nằm trong một trong những database MySQL phân tán lớn của mình. Kế hoạch là chặn mọi truy cập vào đúng một trong số một trăm database. Không ai lường trước được những gì sắp xảy ra.

## Phản ứng (Response)

Chỉ vài phút sau khi bắt đầu kiểm thử, nhiều dịch vụ phụ thuộc đã báo rằng cả người dùng bên ngoài lẫn nội bộ đều không thể truy cập các hệ thống chính. Một số hệ thống chỉ hoạt động chập chờn hoặc chỉ mở một phần.

Cho rằng bài kiểm thử chính là thủ phạm, SRE lập tức hủy bỏ bài tập. Chúng tôi thử rollback thay đổi quyền truy cập nhưng không thành công. Thay vì hoảng loạn, chúng tôi lập tức tìm cách khôi phục quyền truy cập đúng. Bằng một cách tiếp cận đã được kiểm thử trước đó, chúng tôi khôi phục quyền truy cập cho các replica và failover. Song song, chúng tôi liên hệ các developer chính để sửa lỗi trong thư viện tầng ứng dụng của database.

Chỉ một giờ sau quyết định ban đầu, mọi quyền truy cập đã được khôi phục hoàn toàn và tất cả dịch vụ đều kết nối lại. Tác động rộng của bài kiểm thử đã thúc đẩy việc vá lỗi nhanh và kỹ lưỡng cho các thư viện, kèm theo kế hoạch kiểm thử lại định kỳ nhằm ngăn một khiếm khuyết lớn như vậy tái diễn.

## Kết quả (Findings)

### Điều gì đã diễn ra tốt (What went well)

Các dịch vụ phụ thuộc bị sự cố ảnh hưởng đã lập tức leo thang vấn đề trong công ty. Chúng tôi đoán đúng rằng thí nghiệm có kiểm soát của mình đã vượt khỏi tầm kiểm soát và lập tức hủy bỏ bài kiểm thử.

Chúng tôi khôi phục toàn bộ quyền truy cập chỉ trong một giờ kể từ khi có báo cáo đầu tiên, và đến lúc đó các hệ thống đã bắt đầu vận hành bình thường. Một số nhóm lại chọn cách khác: cấu hình lại hệ thống của họ để tránh database kiểm thử. Những nỗ lực song song này giúp khôi phục dịch vụ nhanh nhất có thể.

Chúng tôi xử lý nhanh và kỹ lưỡng các hành động khắc phục để ngăn chặn sự cố ngừng dịch vụ tương tự, đồng thời thiết lập kiểm thử định kỳ nhằm đảm bảo các khiếm khuyết như vậy không tái diễn.

### Những gì chúng tôi đã học (What we learned)

Dù bài kiểm thử đã được rà soát kỹ lưỡng và được cho là có phạm vi rõ ràng, thực tế cho thấy chúng tôi vẫn chưa nắm bắt đầy đủ cách các hệ thống phụ thuộc này tương tác với nhau.

Chúng tôi đã không tuân thủ quy trình phản ứng sự cố, vốn mới được thiết lập vài tuần trước đó và chưa được phổ biến rộng rãi. Nếu có quy trình này, tất cả dịch vụ và khách hàng đều sẽ biết về sự cố ngừng dịch vụ. Để tránh các kịch bản tương tự, SRE liên tục hoàn thiện và kiểm thử các công cụ, quy trình phản ứng sự cố, đồng thời đảm bảo mọi cập nhật quy trình quản lý sự cố được thông báo rõ ràng đến tất cả bên liên quan.

Vì chưa từng chạy thử quy trình hoàn tác trong môi trường thử nghiệm, các bước này còn nhiều sơ hở và đã khiến sự cố ngừng dịch vụ kéo dài. Giờ đây, chúng tôi bắt buộc phải kiểm tra kỹ lưỡng quy trình hoàn tác trước khi thực hiện bất kỳ bài kiểm thử quy mô lớn nào.

## Tình huống Khẩn cấp do Thay đổi Gây ra (Change-Induced Emergency)

Dễ hình dung, Google có rất nhiều cấu hình phức tạp và chúng tôi liên tục thay đổi chúng. Để tránh làm hỏng toàn bộ hệ thống, chúng tôi kiểm thử nhiều lần các thay đổi cấu hình nhằm đảm bảo chúng không gây ra hành vi bất ngờ hay không mong muốn. Tuy nhiên, quy mô và độ phức tạp của hạ tầng Google khiến không thể lường trước mọi sự phụ thuộc hay tương tác; đôi khi các thay đổi cấu hình không diễn ra đúng kế hoạch.

Dưới đây là một ví dụ như vậy.

## Chi tiết (Details)

Một thay đổi cấu hình trên hạ tầng bảo vệ dịch vụ khỏi lạm dụng đã được triển khai toàn cầu vào một ngày thứ Sáu. Hạ tầng này tương tác với gần như tất cả hệ thống hướng bên ngoài; thay đổi đó đã kích hoạt một lỗi gây crash-loop, khiến toàn bộ hạm đội (fleet) gần như đồng thời rơi vào vòng lặp sập. Vì hạ tầng nội bộ của Google cũng phụ thuộc vào chính các dịch vụ của mình, nhiều ứng dụng nội bộ đột ngột không còn khả dụng.

## Phản ứng (Response)

Chỉ trong vài giây, các cảnh báo giám sát bắt đầu bắn, cho thấy một số site đã sập. Một số kỹ sư on-call đồng thời gặp tình trạng mà họ cho là sự cố mạng công ty, và chuyển sang các phòng khẩn cấp (panic room) có đường truy cập dự phòng vào môi trường production. Họ được các kỹ sư khác đang chật vật với quyền truy cập công ty gia nhập.

Chỉ năm phút sau lần đẩy cấu hình đầu tiên, kỹ sư phụ trách — khi đó mới chỉ nhận ra sự cố nội bộ, chưa hay biết sự cố lan rộng hơn — đã đẩy một thay đổi cấu hình khác để hoàn tác thao tác ban đầu. Từ lúc đó, các dịch vụ bắt đầu hồi phục.

10 phút sau lần đẩy đầu tiên, nhóm on-call chính thức công bố sự cố và xử lý theo quy trình ứng phó nội bộ. Họ bắt đầu thông báo tình hình cho toàn công ty. Người triển khai cho nhóm on-call biết sự cố nhiều khả năng do thay đổi vừa đẩy lên rồi được hoàn tác. Dù vậy, một số dịch vụ gặp lỗi hoặc cấu hình sai không liên quan bị sự kiện ban đầu kích hoạt, và mất đến một giờ mới phục hồi hoàn toàn.

## Kết quả (Findings)

### Điều gì đã diễn ra tốt (What went well)

Một số yếu tố đã giúp ngăn incident này thành một outage kéo dài ở nhiều hệ thống nội bộ của Google.

Đầu tiên, giám sát gần như lập tức phát hiện và cảnh báo vấn đề. Tuy nhiên, cần lưu ý rằng trong trường hợp này giám sát của chúng tôi không tối ưu: cảnh báo fire (kích hoạt) lặp đi lặp lại liên tục, áp đảo các kỹ sư on-call và làm tràn (spam) các kênh truyền thông thường và khẩn cấp.

Khi vấn đề được phát hiện, việc quản lý sự cố nhìn chung diễn ra tốt và các cập nhật được truyền thông thường xuyên, rõ ràng. Hệ thống truyền thông out-of-band của chúng tôi giữ cho mọi người duy trì liên lạc ngay cả khi một số stack phần mềm phức tạp hơn không dùng được. Trải nghiệm này nhắc chúng tôi tại sao SRE luôn duy trì các hệ thống dự phòng độ tin cậy cao, chi phí thấp — thứ chúng tôi sử dụng thường xuyên.

Bên cạnh các hệ thống truyền thông out-of-band, Google còn trang bị các công cụ dòng lệnh (command-line) và phương pháp truy cập thay thế, giúp chúng tôi cập nhật cũng như hoàn tác thay đổi ngay cả khi các giao diện khác không truy cập được. Trong suốt sự cố outage, những công cụ và phương pháp này hoạt động hiệu quả, tuy nhiên các kỹ sư cần làm quen sâu hơn và kiểm thử chúng thường xuyên hơn.

Hạ tầng của Google còn có một lớp bảo vệ khác: hệ thống bị ảnh hưởng đã giới hạn tốc độ gửi các bản cập nhật đầy đủ cho các client mới. Hành vi này có thể đã kìm hãm vòng lặp sập, ngăn một sự cố ngừng dịch vụ hoàn toàn, giúp các job chạy đủ lâu để xử lý vài yêu cầu giữa các lần sập.

Cuối cùng, không nên bỏ qua yếu tố may mắn trong việc xử lý nhanh sự cố này: kỹ sư triển khai tình cờ đang theo dõi các kênh truyền thông thời gian thực — một sự cẩn trọng thêm không thuộc quy trình phát hành thông thường. Người này nhận thấy hàng loạt khiếu nại về truy cập công ty ngay sau lần đẩy thay đổi lên và đã hoàn tác gần như lập tức. Nếu không có lần hoàn tác nhanh đó, sự cố ngừng dịch vụ có thể kéo dài đáng kể và khó xử lý hơn rất nhiều.

### Những gì chúng tôi đã học (What we learned)

Một lần triển khai trước đó của tính năng mới đã chạy canary kỹ lưỡng nhưng không kích hoạt cùng lỗi, vì nó không chạy một từ khóa cấu hình rất hiếm và cụ thể kết hợp với tính năng mới. Thay đổi cụ thể kích hoạt lỗi này không được coi là rủi ro, nên chỉ qua một quy trình canary nới hơn. Khi thay đổi được đẩy lên toàn cầu, chính tổ hợp từ khóa/tính năng chưa được kiểm thử đó đã gây ra sự cố.

Trớ trêu thay, các cải tiến về canarying và tự động hóa vốn được lên lịch nâng ưu tiên trong quý sau. Sự cố này lập tức đẩy ưu tiên của chúng lên, đồng thời nhấn mạnh nhu cầu canary kỹ lưỡng, bất kể mức rủi ro nhận thức.

Cảnh báo dồn dập trong suốt sự cố này là điều dễ hiểu, vì mọi vị trí về cơ bản đều offline trong vài phút. Điều này làm gián đoạn công việc thực sự của các kỹ sư on-call và khiến việc truyền thông giữa những người tham gia sự cố trở nên khó khăn hơn.

Google phụ thuộc vào chính các công cụ của mình. Phần lớn stack phần mềm dùng để xử lý sự cố và truyền thông nằm sau các job đang crash-loop. Nếu sự cố ngừng dịch vụ kéo dài thêm, việc debug sẽ bị cản trở nghiêm trọng.

## Tình huống Khẩn cấp do Quy trình Gây ra (Process-Induced Emergency)

Chúng tôi đã dành rất nhiều thời gian và công sức để tự động hóa việc quản lý hạm đội máy. Thật đáng kinh ngạc khi có thể khởi động, dừng hoặc tái thiết lập hàng loạt job trên toàn hạm đội với rất ít công sức. Tuy nhiên, đôi khi chính hiệu quả của tự động hóa lại khiến người ta hơi rùng mình khi mọi thứ không diễn ra đúng kế hoạch.

Đây là một ví dụ mà việc đi nhanh không hẳn là điều tốt.

## Chi tiết (Details)

Trong một phần kiểm thử tự động hóa thường quy, hai yêu cầu tắt (turndown) liên tiếp được đệ trình cho cùng một cài đặt server sắp ngừng vận hành. Với yêu cầu turndown thứ hai, một lỗi tinh vi trong hệ thống tự động hóa đã đẩy toàn bộ máy trong tất cả các cài đặt này trên toàn cầu vào hàng đợi Diskerase, nơi ổ cứng của chúng sẽ bị xóa; xem [Tự động hóa: Cho phép Thất bại ở Quy mô](https://sre.google/sre-book/automation-at-google#xref_automation_diskerase-sidebar) để biết thêm.

## Phản ứng (Response)

Ngay sau khi yêu cầu turndown thứ hai được phát ra, các kỹ sư on-call nhận được một lần gọi trực khi cài đặt server nhỏ đầu tiên bị đưa offline để decommission. Điều tra cho thấy các máy đã được chuyển vào hàng đợi Diskerase. Vì máy ở vị trí này đã bị xóa nên không thể phản hồi yêu cầu; để tránh làm các yêu cầu đó thất bại hoàn toàn, các kỹ sư on-call rút traffic (drain) khỏi vị trí đó theo đúng quy trình. Traffic được định tuyến lại sang các vị trí có thể phản hồi đúng.

Chẳng bao lâu, các máy gọi trực (pager) khắp nơi liên tục báo cho mọi cài đặt server như vậy trên toàn thế giới. Trước tình hình đó, các kỹ sư on-call vô hiệu hóa toàn bộ tự động hóa của đội để ngăn thiệt hại thêm. Ngay sau đó họ dừng hoặc đóng băng các tự động hóa bổ sung và bảo trì production.

Trong vòng một giờ, toàn bộ traffic được chuyển sang các vị trí khác. Người dùng có thể đã chịu độ trễ cao hơn, nhưng các yêu cầu vẫn được đáp ứng. Sự cố ngừng dịch vụ chính thức kết thúc.

Giờ đến phần khó: phục hồi. Một số liên kết mạng báo cáo tắc nghẽn nặng, nên các kỹ sư mạng phải vào cuộc xử lý khi các điểm nghẽn (choke point) xuất hiện. Một cài đặt server ở một vị trí như vậy được chọn là cái đầu tiên trong số nhiều cái "đứng dậy từ tro tàn". Trong vòng ba giờ kể từ sự cố ngừng dịch vụ ban đầu, và nhờ sự bền bỉ của một số kỹ sư, cài đặt được dựng lại và đưa lên online, lại chấp nhận các yêu cầu người dùng.

Các nhóm tại Mỹ bàn giao cho đối tác châu Âu, và SRE lên kế hoạch ưu tiên việc cài đặt lại bằng một quy trình thủ công nhưng tinh giản. Nhóm được chia làm ba phần, mỗi phần phụ trách một bước trong quy trình cài đặt lại thủ công. Trong vòng ba ngày, phần lớn năng lực đã lên lại online, và phần còn lại sẽ được khôi phục trong một đến hai tháng tiếp theo.

## Kết quả (Findings)

### Điều gì đã diễn ra tốt (What went well)

Trong các hệ thống server quy mô lớn, cách quản lý reverse proxy (bộ proxy ngược) hoàn toàn khác so với các hệ thống nhỏ, nên chúng không bị ảnh hưởng. Các kỹ sư on-call nhanh chóng chuyển traffic từ các hệ thống nhỏ sang các hệ thống lớn. Theo thiết kế, các hệ thống lớn này có thể xử lý toàn bộ tải mà không gặp khó khăn. Tuy nhiên, một số liên kết mạng bị tắc nghẽn, buộc các kỹ sư mạng phải tìm giải pháp thay thế (workaround). Để giảm tác động đến người dùng cuối, các kỹ sư on-call đã ưu tiên xử lý các mạng tắc nghẽn này lên hàng đầu.

Quy trình turndown cho các cài đặt nhỏ hoạt động hiệu quả. Từ đầu đến cuối, mất chưa tới một giờ để turndown thành công và xóa an toàn một lượng lớn các cài đặt này.

Dù tự động hóa turndown đã nhanh chóng gỡ giám sát cho các cài đặt nhỏ, các kỹ sư on-call vẫn kịp đảo ngược những thay đổi đó để đánh giá mức độ thiệt hại.

Các kỹ sư nhanh chóng áp dụng các giao thức phản ứng sự cố, vốn đã hoàn thiện đáng kể sau một năm kể từ sự cố ngừng dịch vụ đầu tiên được mô tả ở chương này. Việc truyền thông và phối hợp xuyên công ty, giữa các nhóm, diễn ra xuất sắc — minh chứng rõ ràng cho chương trình quản lý sự cố và đào tạo. Tất cả thành viên trong các nhóm liên quan đều chung tay, mang theo kinh nghiệm phong phú của mình vào quá trình xử lý.

### Những gì chúng tôi đã học (What we learned)

Nguyên nhân gốc rễ nằm ở server tự động hóa turndown: nó thiếu các kiểm tra hợp lý (sanity check) phù hợp với các lệnh mà mình gửi đi. Khi server chạy lại để xử lý lần turndown thất bại ban đầu, nó nhận được một phản hồi rỗng cho rack máy. Thay vì lọc phản hồi này, server lại truyền bộ lọc rỗng sang database máy, khiến database máy Diskerase tất cả các máy liên quan. Đúng vậy, đôi khi “không” thực sự có nghĩa là “tất cả”. Database máy tuân theo, và luồng turndown bắt đầu nghiền qua các máy nhanh nhất có thể.

Việc cài đặt lại máy vừa chậm vừa thiếu ổn định. Nguyên nhân chính là hệ thống dùng Giao thức Truyền tệp Tối giản (Trivial File Transfer Protocol, TFTP) ở mức Chất lượng Dịch vụ (Quality of Service, QoS) mạng thấp nhất từ các vị trí xa. BIOS (Basic Input/Output System — Hệ thống Nhập/Xuất Cơ bản) của mỗi máy xử lý kém các sự cố.<sup>[1](#fn1)</sup> Tùy vào card mạng, BIOS hoặc dừng hoặc rơi vào vòng lặp khởi động lại liên tục. Ở mỗi chu kỳ, chúng không truyền được các tệp boot (khởi động) và càng làm quá tải các trình cài đặt. Nhóm on-call đã khắc phục bằng cách nâng mức ưu tiên của traffic cài đặt lên chút ít và dùng tự động hóa để khởi động lại các máy bị kẹt.

Hạ tầng cài đặt lại máy không thể xử lý việc thiết lập đồng thời hàng nghìn máy. Phần nguyên nhân là do một lỗi hồi quy (regression) khiến hạ tầng không chạy quá hai task thiết lập trên mỗi máy worker (máy công nhân). Regression này còn dùng cài đặt QoS sai để truyền tệp và có timeout tinh chỉnh kém. Nó bắt buộc cài đặt lại kernel (lõi), kể cả trên các máy vẫn có kernel đúng và chưa bị Diskerase. Để xử lý, các kỹ sư on-call leo thang lên các bên phụ trách hạ tầng này, những người nhanh chóng tinh chỉnh lại để chịu được tải bất thường.

## Mọi Vấn đề đều Có Giải pháp (All Problems Have Solutions)

Thời gian và kinh nghiệm cho thấy các hệ thống không chỉ sẽ hỏng, mà sẽ hỏng theo những cách không ai lường trước được. Một trong những bài học lớn nhất Google rút ra là: một giải pháp luôn tồn tại, ngay cả khi nó không hiển nhiên, nhất là với người đang bị máy gọi trực hét vào tai. Nếu bạn không nghĩ ra được giải pháp, hãy mở rộng phạm vi tìm kiếm. Kéo thêm đồng đội, tìm sự giúp đỡ, làm bất cứ điều gì cần làm, nhưng làm nhanh. Ưu tiên cao nhất là giải quyết vấn đề trước mắt thật nhanh. Thường thì người nắm nhiều trạng thái (state) nhất lại là người mà hành động của mình đã vô tình kích hoạt sự kiện. Hãy tận dụng người đó.

Đặc biệt quan trọng, một khi tình huống khẩn cấp đã được kiểm soát, đừng quên dành thời gian dọn dẹp, viết báo cáo về incident, và…

## Học từ Quá khứ. Đừng Lặp lại nó. (Learn from the Past. Don't Repeat It.)

## Giữ một Lịch sử của các Outage (Keep a History of Outages)

Không có cách học nào hiệu quả hơn việc ghi lại những gì đã hỏng trong quá khứ. Lịch sử tồn tại để chúng ta học hỏi từ sai lầm của tất cả mọi người. Hãy cẩn thận, trung thực, và trên hết, đặt ra những câu hỏi khó. Tìm các hành động cụ thể có thể ngăn một outage như vậy tái diễn, không chỉ ở tầm chiến thuật mà còn ở tầm chiến lược. Đảm bảo tất cả mọi người trong công ty có thể học được những gì bạn đã học, bằng cách công bố và tổ chức các postmortem.

Cả bản thân lẫn người khác đều phải chịu trách nhiệm theo dõi các hành động cụ thể được nêu trong những postmortem này. Làm vậy sẽ ngăn một outage trong tương lai gần như giống hệt — do gần như cùng các tác nhân kích hoạt — với một outage đã được ghi lại. Khi đã có một hồ sơ vững chắc về việc học từ các outage quá khứ, hãy xem bạn có thể làm gì để ngăn các outage tương lai.

## Đặt những Câu hỏi Lớn, Ngay cả Không thể: Nếu Như…? (Ask the Big, Even Improbable, Questions: What If…?)

Không có bài kiểm thử nào lớn hơn thực tế. Hãy tự đặt cho mình vài câu hỏi lớn, mang tính mở. Nếu điện tòa nhà mất…? Nếu các rack thiết bị mạng đang đứng trong hai feet nước…? Nếu datacenter chính đột ngột tắt…? Nếu ai đó xâm phạm web server của bạn…? Bạn sẽ làm gì? Bạn gọi ai? Ai sẽ ký tấm séc? Bạn có kế hoạch không? Bạn có biết cách phản ứng không? Bạn có biết hệ thống của mình sẽ phản ứng thế nào không? Nếu nó xảy ra ngay bây giờ, bạn có thể giảm thiểu tác động không? Người ngồi cạnh bạn có làm được tương tự không?

## Khuyến khích Kiểm thử Chủ động (Encourage Proactive Testing)

Trong lĩnh vực xử lý sự cố, lý thuyết và thực tế thường cách biệt rất xa. Cho đến khi hệ thống thực sự gặp sự cố, bạn sẽ không thể biết chính xác hệ thống đó, các thành phần phụ thuộc, hay người dùng sẽ phản ứng ra sao. Hãy tránh dựa vào những giả định hoặc các yếu tố chưa được kiểm thử. Bạn muốn sự cố xảy ra lúc 2 giờ sáng thứ Bảy, khi phần lớn nhân sự đang vắng mặt tại buổi offsite (hội thảo ngoài) xây dựng đội ở Rừng Đen (Black Forest), hay khi những người giỏi nhất, sáng tạo nhất đang ở ngay bên cạnh, giám sát bài kiểm thử mà họ đã xem xét kỹ lưỡng trong nhiều tuần trước đó?

## Kết luận (Conclusion)

Chúng tôi đã xem xét ba trường hợp khác nhau khi các phần hệ thống gặp sự cố. Dù cả ba tình huống khẩn cấp được kích hoạt theo những cách khác nhau — một do kiểm thử chủ động, một do thay đổi cấu hình, và một do tự động hóa turndown — các phản ứng đều có nhiều điểm chung. Người xử lý không hoảng loạn. Họ kéo thêm người vào khi thấy cần. Họ nghiên cứu và học từ các sự cố ngừng dịch vụ trước đó, rồi xây dựng hệ thống để phản ứng tốt hơn với những loại sự cố đó. Mỗi khi một failure mode mới xuất hiện, họ ghi lại failure mode ấy. Việc theo dõi này giúp các nhóm khác học cách xử lý lỗi tốt hơn và củng cố hệ thống của mình chống lại các sự cố ngừng dịch vụ tương tự. Họ chủ động kiểm thử hệ thống. Những kiểm thử như vậy đảm bảo các thay đổi thực sự sửa được vấn đề gốc, và phát hiện các điểm yếu khác trước khi chúng trở thành sự cố ngừng dịch vụ.

Và khi các hệ thống của chúng tôi tiến hóa, chu kỳ vẫn tiếp diễn: mỗi outage hoặc mỗi bài kiểm thử đều mang lại những cải thiện dần cho cả quy trình lẫn hệ thống. Dù các nghiên cứu tình huống ở chương này đặc thù cho Google, cách tiếp cận phản ứng khẩn cấp này có thể áp dụng theo thời gian cho bất kỳ tổ chức nào, ở bất kỳ quy mô nào.

<a id="fn1"></a>[1](#fn1) BIOS: Basic Input/Output System (Hệ thống Nhập/Xuất Cơ bản). BIOS là phần mềm được xây dựng vào một máy tính để gửi các lệnh đơn giản đến phần cứng, cho phép nhập và xuất trước khi hệ điều hành được tải.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
