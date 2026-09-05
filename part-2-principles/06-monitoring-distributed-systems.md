# Chương 6. Giám sát các Hệ thống Phân tán (Monitoring Distributed Systems)

> **Nguyên bản:** [Chapter 6 - Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Rob Ewaschuk
*Biên tập:* Betsy Beyer

Các đội SRE (Site Reliability Engineering — kỹ thuật độ tin cậy trang web) của Google tuân theo một số nguyên lý cơ bản và best practice (thực hành tốt nhất) để xây dựng các hệ thống [monitoring (giám sát) và alerting (cảnh báo)](https://sre.google/workbook/monitoring/) hiệu quả. Chương này hướng dẫn cách xác định những vấn đề nào đủ nghiêm trọng để gọi trực (page) một người, cũng như cách xử lý các vấn đề chưa đến mức cần kích hoạt gọi trực.

## Các Định nghĩa (Definitions)

Không tồn tại một bộ từ vựng chung, thống nhất cho mọi chủ đề liên quan đến giám sát. Ngay cả trong Google, cách dùng các thuật ngữ dưới đây cũng không đồng nhất, nhưng các cách hiểu phổ biến nhất được liệt kê tại đây.

#### Giám sát (Monitoring)

Thu thập, xử lý, gộp (aggregate) và hiển thị dữ liệu định lượng thời gian thực về một hệ thống, bao gồm số lượng và kiểu các query (yêu cầu truy vấn), số lượng và kiểu các lỗi, thời gian xử lý, cùng vòng đời của server (máy chủ).

#### Giám sát white-box (White-box monitoring)

Giám sát dựa trên các metrics (chỉ số) do chính hệ thống phát ra, bao gồm logs (nhật ký), các giao diện như Java Virtual Machine Profiling Interface, hoặc một HTTP handler (bộ xử lý) cung cấp thống kê nội bộ.

#### Giám sát black-box (Black-box monitoring)

Kiểm thử hành vi có thể nhìn thấy từ bên ngoài như một người dùng sẽ thấy.

#### Dashboard (Bảng điều khiển)

Một ứng dụng (thường dựa trên web) cung cấp cái nhìn tóm tắt về các metrics (chỉ số) cốt lõi của một dịch vụ. Dashboard có thể bao gồm bộ lọc, bộ chọn, v.v., nhưng được thiết kế sẵn để làm nổi bật các metrics quan trọng nhất cho người dùng. Ngoài ra, dashboard cũng có thể hiển thị thông tin của đội như độ dài hàng đợi ticket (yêu cầu), danh sách các bug ưu tiên cao, kỹ sư on-call hiện tại của một khu vực trách nhiệm nhất định, hoặc các push gần đây.

#### Cảnh báo (Alert)

Thông báo được tạo ra để con người đọc, sau đó đẩy đến một hệ thống như hàng đợi bug, ticket, alias (biệt danh) email hoặc máy gọi trực (pager). Tương ứng, các cảnh báo này được phân loại thành *ticket* (yêu cầu xử lý), *email alert* (cảnh báo email),<sup>[1](#fn1)</sup> và *gọi trực* (page).

#### Nguyên nhân gốc rễ (Root cause)

Một khiếm khuyết trong hệ thống phần mềm hoặc trong cách con người vận hành, mà nếu khắc phục được thì ta có thể tin rằng sự kiện đó sẽ không tái diễn theo cùng một cách. Một incident (sự cố) cụ thể có thể có nhiều nguyên nhân gốc rễ: ví dụ, nó có thể do kết hợp của tự động hóa quy trình không đủ, phần mềm crash (sập) với đầu vào sai, *và* kiểm thử không đủ cho script dùng để tạo cấu hình. Mỗi yếu tố trong số này có thể đứng riêng như một nguyên nhân gốc rễ, và mỗi yếu tố đều cần sửa.

#### Node (Nút) và machine (máy)

Thuật ngữ này dùng lẫn lộn để chỉ một instance (thực thể) duy nhất của một kernel (lõi) đang chạy trên server vật lý, máy ảo (virtual machine) hoặc container. Một machine (máy) đơn lẻ có thể chứa nhiều *dịch vụ* cần giám sát. Các dịch vụ có thể:

-   Liên quan đến nhau: ví dụ, một caching server (server bộ đệm) và một web server (server web)
-   Các dịch vụ không liên quan nhưng cùng dùng chung phần cứng: chẳng hạn, một code repository (kho mã nguồn) và một master (máy chủ chính) cho hệ thống cấu hình như [Puppet](https://puppetlabs.com/puppet/puppet-open-source) hoặc [Chef](https://www.chef.io/chef/)

#### Push (Đẩy)

Bất kỳ thay đổi nào đối với phần mềm đang chạy của một dịch vụ hoặc cấu hình của nó.

## Vì sao Giám sát? (Why Monitor?)

Có nhiều lý do để giám sát một hệ thống, bao gồm:

#### Phân tích các xu hướng dài hạn

Database (cơ sở dữ liệu) của tôi lớn cỡ nào và lớn nhanh ra sao? Số người dùng hoạt động hàng ngày của tôi tăng nhanh thế nào?

#### So sánh theo thời gian hoặc các nhóm thí nghiệm

Query có nhanh hơn khi dùng Acme Bucket of Bytes 2.72 so với Ajax DB 3.14 không? Tỷ lệ hit (trúng) memcache (bộ nhớ đệm) của tôi cải thiện bao nhiêu khi thêm một node (nút)? Website của tôi có chậm hơn so với tuần trước không?

#### Cảnh báo (Alerting)

Có gì đó đang hỏng và ai đó cần sửa ngay! Hoặc, có gì đó có thể sắp hỏng, nên ai đó nên xem xét sớm.

#### Xây dựng các dashboard

Các dashboard nên trả lời những câu hỏi cơ bản về dịch vụ của bạn và thường bao gồm một số dạng của bốn tín hiệu vàng (golden signal, được bàn trong [Bốn Tín hiệu Vàng](#bon-tin-hieu-vang)).

#### Thực hiện phân tích hồi cứu *ad hoc* (tức là, debug — xử lý lỗi)

Độ trễ (latency) của chúng tôi vừa tăng vọt; điều gì khác đã xảy ra quanh cùng thời gian đó?

Giám sát hệ thống cũng hữu ích trong việc cung cấp dữ liệu thô cho phân tích kinh doanh (business analytics) và hỗ trợ phân tích các vụ vi phạm bảo mật (security breach). Vì cuốn sách này tập trung vào các lĩnh vực kỹ thuật mà SRE có chuyên môn đặc biệt, chúng tôi sẽ không bàn các ứng dụng giám sát này ở đây.

Giám sát và cảnh báo giúp hệ thống thông báo cho chúng ta khi nó gặp sự cố, hoặc dự báo trước những phần sắp hỏng. Khi hệ thống không thể tự động khắc phục, chúng ta cần một người vào cuộc để điều tra cảnh báo, xác minh xem có vấn đề thực sự tồn tại hay không, giảm thiểu tác động và tìm ra nguyên nhân gốc rễ. Trừ khi bạn đang thực hiện kiểm toán bảo mật (security auditing) trên các thành phần có phạm vi rất hẹp của một hệ thống, bạn không bao giờ nên kích hoạt cảnh báo chỉ vì "có gì đó có vẻ hơi lạ".

Việc gọi ai đó đi trực là một cách sử dụng khá đắt thời gian của một nhân viên. Nếu nhân viên đang ở nơi làm việc, một lần gọi trực sẽ ngắt luồng công việc của họ. Nếu nhân viên ở nhà, một lần gọi trực sẽ ngắt thời gian cá nhân của họ, và có thể cả giấc ngủ. Khi gọi trực xảy ra quá thường xuyên, nhân viên sẽ nghi ngờ, lướt qua hoặc thậm chí bỏ qua các cảnh báo đến, đôi khi bỏ qua cả một lần gọi trực "thực sự" bị che khuất bởi nhiễu. Các outage (mất dịch vụ) có thể bị kéo dài vì những nhiễu khác cản trở việc chẩn đoán và sửa chữa nhanh chóng. Các [hệ thống cảnh báo hiệu quả](https://sre.google/sre-book/practical-alerting/) có tín hiệu tốt và nhiễu rất thấp.

## Đặt Các Kỳ vọng Hợp lý cho Giám sát

Việc giám sát một ứng dụng phức tạp tự nó đã là một nỗ lực kỹ thuật không nhỏ. Dù đã có sẵn hạ tầng đo lường (instrumentation), thu thập, hiển thị và cảnh báo đáng kể, một nhóm SRE tại Google với 10–12 thành viên thường vẫn có một, đôi khi hai, người mà nhiệm vụ chính là xây dựng và duy trì các hệ thống giám sát cho dịch vụ của nhóm. Con số này đã giảm theo thời gian khi chúng tôi tổng quát hóa và tập trung hóa hạ tầng giám sát chung, nhưng mỗi nhóm SRE thường có ít nhất một "người giám sát". (Nói vậy không có nghĩa là, dù truy cập các dashboard đồ thị traffic (lưu lượng) và tương tự có thể rất thú vị, các nhóm SRE cẩn thận tránh mọi tình huống đòi hỏi ai đó phải "ngồi nhìn màn hình để canh chừng vấn đề.")

Nhìn chung, Google ưu tiên các hệ thống giám sát đơn giản và nhanh, kết hợp công cụ tốt hơn cho phân tích *post hoc* (sau sự kiện). Chúng tôi tránh những hệ thống "ma thuật" cố tự học ngưỡng (threshold) hoặc tự động phát hiện quan hệ nhân quả. Quy tắc phát hiện thay đổi bất thường trong tỷ lệ yêu cầu của người dùng cuối là một phản ví dụ: dù được giữ ở mức đơn giản nhất có thể, chúng vẫn cho phép phát hiện rất nhanh các dị thường (anomaly) đơn giản, cụ thể và nghiêm trọng. Các ứng dụng khác của dữ liệu giám sát, chẳng hạn [lập kế hoạch năng lực](https://sre.google/sre-book/addressing-cascading-failures/) và dự đoán traffic, có thể chấp nhận mức độ dễ vỡ (fragility) cao hơn, do đó cũng phức tạp hơn. Tương tự, các thí nghiệm quan sát chạy trên chân trời thời gian dài (vài tháng hoặc vài năm) với tần suất lấy mẫu thấp (vài giờ hoặc vài ngày) cũng thường cho phép mức độ dễ vỡ cao hơn, bởi việc thỉnh thoảng bỏ sót mẫu sẽ không che giấu được một xu hướng kéo dài.

Google SRE chỉ đạt thành công hạn chế với các hệ phân cấp phụ thuộc (dependency hierarchy) phức tạp. Chúng tôi hiếm khi dùng các quy tắc kiểu "Nếu tôi biết database đang chậm, cảnh báo database chậm; nếu không, cảnh báo website nhìn chung đang chậm." Các quy tắc phụ thuộc lẫn nhau thường gắn với những phần rất ổn định của hệ thống chúng tôi, như hệ thống rút traffic người dùng khỏi một datacenter (trung tâm dữ liệu). Ví dụ, "Nếu một datacenter đã được rút traffic, thì đừng cảnh báo tôi về độ trễ của nó" là một quy tắc cảnh báo datacenter phổ biến. Ít đội tại Google duy trì các hệ phân cấp phụ thuộc phức tạp vì hạ tầng của chúng tôi liên tục được refactor (tái cấu trúc) ở một nhịp ổn định.

Một số ý tưởng trong chương này vẫn mang tính lý tưởng: luôn có dư địa để rút ngắn thời gian từ triệu chứng (symptom) đến nguyên nhân gốc rễ, đặc biệt trong các hệ thống liên tục thay đổi. Vì vậy, dù chương này nêu ra một số mục tiêu cho hệ thống giám sát cùng các cách để đạt được chúng, điều quan trọng là các hệ thống giám sát — nhất là đường dẫn quan trọng (critical path) từ lúc một vấn đề production xuất hiện, qua một cuộc gọi trực đến một người, rồi đến phân loại cơ bản và debug sâu — phải luôn đơn giản và dễ hiểu với mọi thành viên trong đội.

Tương tự, để giữ nhiễu thấp và tín hiệu cao, các phần của hệ thống giám sát dẫn đến việc gọi trực cần phải rất đơn giản và vững chắc. Các quy tắc tạo cảnh báo cho người nên dễ hiểu và thể hiện một sự cố rõ ràng.

## Triệu chứng so với Nguyên nhân (Symptoms Versus Causes)

Hệ thống giám sát của bạn nên giải quyết hai câu hỏi: cái gì đang hỏng, và tại sao?

"Cái gì đang hỏng" nêu rõ triệu chứng, còn "tại sao" chỉ ra một nguyên nhân (có thể là trung gian). [Bảng 6-1](#bang-6-1) liệt kê một số triệu chứng giả định cùng nguyên nhân tương ứng.

<a id="bang-6-1"></a>Bảng 6-1. Các triệu chứng và nguyên nhân ví dụ

| **Triệu chứng** | **Nguyên nhân** |
|---|---|
| **Tôi đang phục vụ các HTTP 500 hoặc 404** | Các server database đang từ chối kết nối |
| **Các phản hồi của tôi đang chậm** | Các CPU đang bị quá tải bởi một bogosort, hoặc một cáp Ethernet đang bị ép dưới một rack (giá máy), nhìn thấy được dưới dạng mất gói tin một phần |
| **Người dùng ở châu Nam Cực không nhận được các GIF mèo hoạt hình** | Content Distribution Network (Mạng phân phối nội dung) của bạn ghét các nhà khoa học và loài mèo, và do đó đã đưa một số IP client vào danh sách đen |
| **Nội dung riêng tư có thể đọc được bởi cả thế giới** | Một push phần mềm mới khiến các ACL (Access Control List — danh sách kiểm soát truy cập) bị quên và cho phép tất cả các yêu cầu |

"Cái gì" so với "tại sao" là một trong những phân biệt quan trọng nhất khi viết giám sát tốt với tín hiệu tối đa và nhiễu tối thiểu.

## Black-box so với White-box

Chúng tôi kết hợp mạnh mẽ giám sát white-box với các ứng dụng vừa phải nhưng quan trọng của giám sát black-box. Cách đơn giản nhất để phân biệt black-box và white-box là: giám sát black-box định hướng theo triệu chứng, phản ánh các vấn đề đang diễn ra..., chứ không phải dự đoán: "Hệ thống không hoạt động đúng, ngay bây giờ." Trong khi đó, giám sát white-box dựa vào khả năng kiểm tra bên trong hệ thống, chẳng hạn như log hoặc các endpoint (điểm cuối) HTTP, thông qua đo lường. Nhờ vậy, giám sát white-box cho phép phát hiện các vấn đề sắp xảy ra, các sự cố bị che giấu bởi việc thử lại (retry), v.v.

Lưu ý rằng trong một hệ thống nhiều tầng, triệu chứng của người này lại là nguyên nhân của người kia. Ví dụ, giả sử [hiệu năng của một database](https://sre.google/sre-book/data-integrity/) đang chậm. Các lần đọc database chậm là một triệu chứng đối với SRE database phát hiện ra chúng. Tuy nhiên, với SRE frontend quan sát một website chậm, chính những lần đọc database chậm đó lại là một nguyên nhân. Do đó, giám sát white-box đôi khi định hướng theo triệu chứng, đôi khi theo nguyên nhân, tùy thuộc vào mức độ thông tin mà white-box của bạn cung cấp.

Khi thu thập dữ liệu đo lường (telemetry) để debug, giám sát white-box là thiết yếu. Nếu các web server có vẻ chậm với các yêu cầu nặng database, bạn cần biết cả tốc độ mà web server cảm nhận database đang chạy và tốc độ mà database tự tin rằng mình đang chạy. Nếu không, bạn không thể phân biệt một server database thực sự chậm với một vấn đề mạng giữa web server và database.

Trong việc gọi trực, lợi ích chính của giám sát black-box là buộc kỷ luật chỉ quấy rầy một người khi vấn đề vừa đang tiếp diễn vừa đang góp phần vào các triệu chứng thực. Ngược lại, đối với các vấn đề chưa xảy ra nhưng sắp xảy ra, giám sát black-box khá vô dụng.

<a id="bon-tin-hieu-vang"></a>

## Bốn Tín hiệu Vàng (The Four Golden Signals)

Bốn tín hiệu vàng của giám sát là độ trễ (latency), traffic, lỗi (error) và độ bão hòa (saturation). Nếu bạn chỉ có thể đo bốn metrics của hệ thống hướng người dùng, hãy tập trung vào bốn cái này.

#### Độ trễ (Latency)

Thời gian cần để xử lý một yêu cầu. Cần phân biệt rõ độ trễ của các yêu cầu thành công và độ trễ của các yêu cầu thất bại. Ví dụ, một lỗi HTTP 500 do mất kết nối đến database hoặc backend quan trọng khác có thể được trả về rất nhanh; tuy nhiên, vì lỗi HTTP 500 cho thấy yêu cầu đã thất bại, việc gộp các 500 vào độ trễ tổng thể có thể dẫn đến các phép tính gây nhầm lẫn. Mặt khác, một lỗi chậm còn tệ hơn một lỗi nhanh! Vì vậy, cần theo dõi độ trễ của các lỗi, thay vì chỉ lọc bỏ chúng.

**Traffic**

Một phép đo phản ánh mức độ tải đang đè lên hệ thống của bạn, dựa trên một metrics cấp cao cụ thể. Với dịch vụ web, phép đo này thường là số yêu cầu HTTP mỗi giây, có thể tách theo bản chất các yêu cầu (ví dụ nội dung tĩnh so với nội dung động). Với hệ thống phát trực tuyến audio, phép đo này có thể tập trung vào tỷ lệ I/O mạng hoặc các phiên đồng thời (concurrent session). Với hệ thống lưu trữ key-value (khóa–giá trị), phép đo này có thể là số giao dịch và truy xuất mỗi giây.

#### Lỗi (Errors)

Tỷ lệ yêu cầu thất bại có thể do nhiều nguyên nhân: lỗi rõ ràng (ví dụ HTTP 500), lỗi ngầm (ví dụ phản hồi HTTP 200 nhưng nội dung sai), hoặc vi phạm chính sách (ví dụ cam kết thời gian phản hồi một giây, mọi yêu cầu vượt quá ngưỡng này đều tính là lỗi). Khi mã phản hồi giao thức không đủ để biểu đạt mọi điều kiện thất bại, cần dùng các giao thức phụ (nội bộ) để theo dõi các failure mode một phần. Việc giám sát các trường hợp này có thể khác nhau đáng kể: bắt HTTP 500 ở load balancer (bộ cân bằng tải) có thể xử lý tốt các yêu cầu thất bại hoàn toàn, nhưng chỉ kiểm thử hệ thống đầu-cuối (end-to-end) mới phát hiện được việc bạn đang phục vụ nội dung sai.

#### Độ bão hòa (Saturation)

Mức độ "đầy" của dịch vụ bạn. Đây là phép đo phần hệ thống, tập trung vào các tài nguyên bị ràng buộc nhất (ví dụ: hệ thống bị ràng buộc về bộ nhớ thì hiển thị bộ nhớ; hệ thống bị ràng buộc về I/O thì hiển thị I/O). Lưu ý rằng nhiều hệ thống suy giảm hiệu năng trước khi đạt 100% mức sử dụng, nên việc đặt ra mục tiêu mức sử dụng là thiết yếu.

Trong các hệ thống phức tạp, độ bão hòa có thể được bổ sung bằng các phép đo tải ở cấp cao hơn: dịch vụ của bạn có thể xử lý gấp đôi traffic, chỉ xử lý thêm 10% traffic, hay xử lý ít traffic hơn mức hiện đang nhận không? Với các dịch vụ rất đơn giản, không có tham số làm thay đổi độ phức tạp của yêu cầu (ví dụ "Cho tôi một nonce" hay "Tôi cần một số nguyên đơn điệu toàn cầu duy nhất") và hiếm khi thay đổi cấu hình, một giá trị tĩnh từ một load test (kiểm thử tải) có thể là đủ. Tuy nhiên, như đã bàn ở đoạn trước, hầu hết các dịch vụ cần dùng các tín hiệu gián tiếp như mức sử dụng CPU hoặc băng thông mạng có cận trên đã biết. Độ trễ tăng thường là chỉ báo dẫn trước của độ bão hòa. Đo thời gian phản hồi phân vị thứ 99 (99th percentile) trên một cửa sổ nhỏ (ví dụ một phút) có thể cho một tín hiệu rất sớm về độ bão hòa.

Cuối cùng, độ bão hòa cũng liên quan đến các dự đoán về sự bão hòa sắp xảy ra, như "Trông database của bạn sẽ lấp đầy ổ cứng trong 4 giờ nữa."

Nếu bạn đo cả bốn tín hiệu vàng và gọi một người đi trực khi một tín hiệu có vấn đề (hoặc, với độ bão hòa, sắp có vấn đề), dịch vụ của bạn sẽ ít nhất được giám sát bao phủ đàng hoàng.

## Lo lắng về Đuôi của Bạn (hay, Đo lường và Hiệu năng) (Worrying About Your Tail)

Khi xây dựng hệ thống giám sát từ đầu, ta thường có xu hướng thiết kế dựa trên giá trị trung bình của một đại lượng: độ trễ trung bình, mức sử dụng CPU trung bình của các node, hay mức độ đầy trung bình của các database. Nguy hiểm của hai trường hợp sau là rõ ràng: CPU và database có thể dễ dàng bị sử dụng theo cách rất mất cân bằng. Điều tương tự đúng với độ trễ. Nếu bạn chạy một dịch vụ web với độ trễ trung bình 100 ms ở 1.000 yêu cầu mỗi giây, 1% các yêu cầu có thể dễ dàng mất 5 giây.<sup>[2](#fn2)</sup> Nếu người dùng của bạn phụ thuộc vào một vài dịch vụ web như vậy để hiển thị trang của họ, phân vị thứ 99 của một backend có thể dễ dàng trở thành thời gian phản hồi trung vị (median) của frontend.

Cách đơn giản nhất để phân biệt giữa một giá trị trung bình chậm và một "đuôi" (tail) các yêu cầu rất chậm là thu thập số lượng yêu cầu chia theo các khoảng độ trễ (phù hợp để vẽ một biểu đồ histogram), chứ không phải các độ trễ thực: bạn đã phục vụ bao nhiêu yêu cầu mất giữa 0 ms và 10 ms, giữa 10 ms và 30 ms, giữa 30 ms và 100 ms, giữa 100 ms và 300 ms, và v.v.? Việc chia các ranh giới của biểu đồ histogram theo cách gần như mũ (exponential) (trong trường hợp này theo các hệ số xấp xỉ 3) thường là một cách dễ dàng để trực quan hóa phân bố các yêu cầu.

## Chọn Một Độ phân giải Phù hợp cho các Phép đo

Các khía cạnh khác nhau của một hệ thống nên được đo với các mức độ hạt (granularity) khác nhau. Ví dụ:

-   Quan sát tải CPU trên khoảng thời gian một phút sẽ không tiết lộ ngay cả các đỉnh (spike) tồn tại khá lâu gây ra độ trễ tail cao.
-   Ngược lại, với một dịch vụ web chỉ cho phép tổng cộng không quá 9 giờ downtime (thời gian ngừng) mỗi năm (uptime 99.9% hàng năm), việc thăm dò (probe) trạng thái 200 (thành công) nhiều hơn một hoặc hai lần mỗi phút có thể là quá thường xuyên.
-   Tương tự, kiểm tra mức độ đầy của ổ cứng cho một dịch vụ nhắm đến khả dụng 99.9% nhiều hơn một lần mỗi 1–2 phút có thể là không cần thiết.

Hãy cân nhắc kỹ khi xác định độ hạt cho các phép đo. Việc thu thập dữ liệu tải CPU mỗi giây có thể mang lại thông tin hữu ích, nhưng tần suất đo cao như vậy thường khiến chi phí thu thập, lưu trữ và phân tích tăng vọt. Nếu mục tiêu giám sát của bạn cần độ phân giải cao nhưng không yêu cầu độ trễ cực thấp, bạn có thể tiết kiệm chi phí bằng cách lấy mẫu nội bộ trên server, sau đó cấu hình một hệ thống bên ngoài để thu thập và gộp phân bố dữ liệu đó theo thời gian hoặc xuyên suốt các server. Bạn có thể:

1.  Ghi lại mức sử dụng CPU hiện tại mỗi giây.
2.  Sử dụng các bucket (khoảng giá trị) có độ hạt 5%, tăng bucket mức sử dụng CPU phù hợp mỗi giây.
3.  Gộp các giá trị đó mỗi phút.

Chiến lược này cho phép bạn quan sát các điểm nóng CPU ngắn mà không phải gánh chi phí rất cao của việc thu thập và giữ lại dữ liệu.

## Đơn giản nhất có thể, không đơn giản hơn (As Simple as Possible, No Simpler)

Việc chất chồng tất cả các yêu cầu này lên nhau có thể cộng dồn thành một hệ thống giám sát rất phức tạp — hệ thống của bạn có thể kết thúc với các mức độ phức tạp sau:

-   Các cảnh báo trên các ngưỡng độ trễ khác nhau, ở các phân vị khác nhau, trên đủ mọi loại metrics khác nhau
-   Code bổ sung để phát hiện và phơi bày các nguyên nhân có thể
-   Các dashboard liên quan cho mỗi nguyên nhân có thể trong số này

Nguồn gốc của sự phức tạp tiềm năng là vô tận. Giống như mọi hệ thống phần mềm, hệ thống giám sát có thể trở nên phức tạp đến mức dễ vỡ, khó thay đổi và gây gánh nặng bảo trì.

Vì vậy, hãy thiết kế hệ thống giám sát của bạn theo hướng đơn giản. Khi quyết định giám sát những gì, hãy lưu ý các nguyên tắc sau:

-   Các quy tắc bắt các incident thực sự thường xuyên nhất nên đơn giản, có thể dự đoán, và đáng tin cậy nhất có thể.
- Việc thu thập dữ liệu, gộp và [cấu hình cảnh báo](https://sre.google/workbook/alerting-on-slos/) hiếm khi được thực hành (ví dụ, ít hơn một lần một quý đối với một số đội SRE) nên được cân nhắc loại bỏ.
-   Các tín hiệu đã thu thập nhưng không xuất hiện trên bất kỳ dashboard có sẵn nào, cũng không được dùng cho cảnh báo nào, là ứng viên để loại bỏ.

Theo kinh nghiệm của Google, việc thu thập và gộp cơ bản các metrics, kết hợp với cảnh báo và dashboard, đã hoạt động tốt như một hệ thống tương đối độc lập. (Thực tế hệ thống giám sát của Google chia thành một vài binary (file thực thi), nhưng thường mọi người đều làm quen với tất cả các khía cạnh của các binary này.) Có cám dỗ kết hợp giám sát với các khía cạnh khác của việc kiểm tra hệ thống phức tạp, như profiling (đo hiệu năng) hệ thống chi tiết, debug đơn process, theo dõi chi tiết các ngoại lệ (exception) hoặc crash, load test, thu thập và phân tích log, hoặc kiểm tra traffic. Dù phần lớn các chủ đề này chia sẻ điểm chung với giám sát cơ bản, pha trộn quá nhiều sẽ dẫn đến các hệ thống quá phức tạp và dễ vỡ. Như trong nhiều khía cạnh khác của kỹ thuật phần mềm, duy trì các hệ thống riêng biệt với các điểm tích hợp rõ ràng, đơn giản, có liên kết lỏng lẻo là chiến lược tốt hơn (ví dụ dùng web API để kéo dữ liệu tóm tắt ở một định dạng có thể giữ ổn định trong một khoảng thời gian dài).

## Gắn kết Những Nguyên lý Này Lại với Nhau

Các nguyên lý bàn trong chương này có thể gắn kết thành một triết lý về giám sát và cảnh báo được tán thành và làm theo rộng rãi trong các đội SRE Google. Dù triết lý giám sát này hơi mang tính lý tưởng, nó là điểm khởi đầu tốt cho việc viết hoặc xem xét một cảnh báo mới, và có thể giúp tổ chức của bạn đặt những câu hỏi đúng, bất kể kích thước tổ chức hay độ phức tạp của dịch vụ hoặc hệ thống.

Khi thiết lập các quy tắc cho giám sát và cảnh báo, hãy tự đặt những câu hỏi dưới đây để tránh dương tính giả (false positives) và kiệt sức vì máy gọi trực (pager burnout):<sup>[3](#fn3)</sup>

-   Quy tắc này có phát hiện *một điều kiện chưa từng được phát hiện trước đó*, mang tính khẩn cấp, có thể hành động, và đang hoặc sắp diễn ra theo cách người dùng có thể thấy được không?<sup>[4](#fn4)</sup>
-   Tôi có bao giờ có thể bỏ qua cảnh báo này vì biết nó vô hại không? Khi nào và vì sao tôi có thể bỏ qua, và làm thế nào để tránh kịch bản này?
-   Cảnh báo này có thực sự cho thấy người dùng đang chịu tác động tiêu cực không? Có những trường hợp phát hiện được nhưng người dùng không bị ảnh hưởng tiêu cực, chẳng hạn traffic đã được rút hoặc các triển khai kiểm thử, thì nên lọc ra không?
-   Tôi có thể làm gì để xử lý cảnh báo này? Việc đó có khẩn cấp hay có thể để đến sáng? Có thể tự động hóa an toàn không? Đây là giải pháp lâu dài hay chỉ là biện pháp tạm thời?
-   Có người khác cũng đang bị gọi trực cho vấn đề này không, khiến ít nhất một trong các lần gọi trực trở nên không cần thiết?

Những câu hỏi này phản ánh một triết lý cơ bản về gọi trực và máy gọi trực:

-   Mỗi lần máy gọi trực reo, tôi nên có thể phản ứng với một cảm giác khẩn cấp. Tôi chỉ có thể phản ứng với cảm giác khẩn cấp vài lần một ngày trước khi kiệt sức.
-   Mọi lần gọi trực nên có thể hành động được.
-   Mọi phản ứng với một lần gọi trực đều cần được suy xét. Nếu một lần gọi trực chỉ cần phản ứng theo kiểu máy móc, thì nó không nên là một lần gọi trực.
-   Các lần gọi trực nên dành cho một vấn đề mới hoặc một sự kiện chưa từng thấy trước.

Cách nhìn này xóa nhòa một số ranh giới: nếu một lần gọi trực đáp ứng đủ bốn gạch đầu dòng trước, thì không cần quan tâm nó được kích hoạt bởi giám sát white-box hay black-box. Đồng thời, góc nhìn này cũng nhấn mạnh một số khác biệt: cần dồn nhiều nỗ lực hơn vào việc bắt các triệu chứng thay vì nguyên nhân; còn với nguyên nhân, chỉ nên lo lắng khi chúng đã rất chắc chắn và sắp xảy ra.

## Giám sát cho Lâu dài (Monitoring for the Long Term)

Trong các hệ thống production hiện đại, hệ thống giám sát phải theo dõi một hệ thống luôn phát triển, với kiến trúc phần mềm, [đặc tính tải,](https://sre.google/workbook/managing-load/) và mục tiêu hiệu năng liên tục thay đổi. Một cảnh báo hiện tại hiếm gặp bất thường và khó tự động hóa có thể trở nên thường xuyên, thậm chí đến mức cần một script chắp vá (hacked-together) để xử lý. Khi đó, ai đó nên tìm và loại bỏ nguyên nhân gốc rễ của vấn đề; nếu không thể, phản ứng cảnh báo xứng đáng được tự động hóa hoàn toàn.

Điều quan trọng là các quyết định về giám sát phải được đưa ra với mục tiêu dài hạn trong tâm trí. Mỗi lần gọi trực hôm nay đều khiến một người mất tập trung khỏi việc cải thiện hệ thống cho ngày mai, nên thường có lý do để chấp nhận một cú đánh ngắn hạn vào khả dụng hoặc hiệu năng nhằm cải thiện triển vọng dài hạn của hệ thống. Hãy xem hai nghiên cứu tình huống minh họa sự đánh đổi này.

## SRE Bigtable: Một Câu chuyện về Cảnh báo Quá mức (A Tale of Over-Alerting)

Hạ tầng nội bộ của Google thường được cung cấp và đo lường theo một service level objective (SLO — mục tiêu mức dịch vụ; xem [Service Level Objectives](04-service-level-objectives.md)). Nhiều năm trước, SLO của dịch vụ Bigtable dựa trên hiệu năng trung bình của một client tổng hợp (synthetic) có hành vi tốt. Do các vấn đề trong Bigtable và các tầng thấp hơn của stack lưu trữ, hiệu năng trung bình bị chi phối bởi một "đuôi" lớn: 5% yêu cầu tệ nhất thường chậm hơn đáng kể so với phần còn lại.

Cảnh báo email kích hoạt khi SLO tiến gần, và cảnh báo gọi trực kích hoạt khi SLO bị vượt quá. Cả hai loại cảnh báo đều fire (kích hoạt) với số lượng lớn, tiêu tốn một lượng thời gian kỹ thuật không thể chấp nhận được: đội dành rất nhiều thời gian phân loại các cảnh báo để tìm ra ít cảnh báo thực sự có thể hành động, và chúng tôi thường bỏ lỡ các vấn đề thực sự ảnh hưởng đến người dùng, vì rất ít cảnh báo làm vậy. Nhiều lần gọi trực không khẩn cấp, do các vấn đề đã được hiểu rõ trong hạ tầng, và hoặc có các phản ứng máy móc (rote) hoặc không nhận được phản hồi.

Để khắc phục, nhóm áp dụng cách tiếp cận ba mũi nhọn: trong khi dốc sức cải thiện hiệu năng Bigtable, chúng tôi cũng tạm thời hạ mục tiêu SLO, dùng độ trễ yêu cầu phân vị thứ 75 (75th percentile). Chúng tôi cũng vô hiệu hóa các cảnh báo email, vì có quá nhiều đến mức dành thời gian chẩn đoán chúng là bất khả thi.

Chiến lược này cho chúng tôi đủ không gian thở để thực sự sửa các vấn đề dài hạn trong Bigtable và các tầng thấp hơn của stack lưu trữ, thay vì liên tục vá các vấn đề chiến thuật. Các kỹ sư on-call thực sự có thể hoàn thành công việc khi không bị đánh thức bởi các cuộc gọi trực vào mọi lúc. Cuối cùng, việc lùi lại tạm thời các cảnh báo cho phép chúng tôi tiến nhanh hơn về phía một dịch vụ tốt hơn.

## Gmail: Các Phản ứng Có thể Dự đoán được, Có thể Script được từ Con người

Giai đoạn đầu của Gmail, dịch vụ chạy trên Workqueue, một hệ thống quản lý tiến trình phân tán được cải tiến từ hệ thống ban đầu dùng để xử lý các lô (batch) phần của chỉ mục tìm kiếm. Workqueue được điều chỉnh để hỗ trợ các tiến trình chạy dài hạn rồi áp dụng cho Gmail, nhưng một số lỗi trong kho mã nguồn tương đối mờ mịt của bộ lập lịch (scheduler) tỏ ra rất khó khắc phục.

Lúc bấy giờ, hệ thống giám sát Gmail được thiết kế để phát cảnh báo khi các task đơn lẻ bị Workqueue hủy lịch. Cách cấu hình này không tối ưu, bởi khi đó Gmail đã vận hành hàng nghìn task, mỗi task phục vụ một phần nghìn người dùng. Chúng tôi rất coi trọng việc mang lại trải nghiệm tốt cho người dùng Gmail, nhưng với cấu hình cảnh báo như vậy, việc duy trì hệ thống là bất khả thi.

Để giải quyết, SRE Gmail xây dựng một công cụ giúp “gõ” (poke) scheduler theo cách vừa phải nhằm giảm thiểu tác động đến người dùng. Nhóm đã có một vài cuộc thảo luận về việc có nên đơn giản tự động hóa toàn bộ vòng từ phát hiện vấn đề đến thúc đẩy bộ lập lịch lại, cho đến khi đạt một giải pháp dài hạn tốt hơn không, nhưng một số lo ngại loại giải pháp tình huống này sẽ trì hoãn một sửa chữa thực sự.

Loại căng thẳng này khá phổ biến trong các đội, thường phản ánh sự thiếu tin tưởng cơ bản vào khả năng tự kỷ luật của đội: trong khi một số thành viên muốn triển khai một "hack" (mẹo vá) để mua thêm thời gian cho một sửa chữa đúng đắn, những người khác lo ngại hack sẽ bị quên hoặc sửa chữa đúng đắn sẽ bị hạ thấp ưu tiên vô thời hạn. Lo ngại này có cơ sở, vì dễ dàng tích lũy các tầng nợ kỹ thuật (technical debt) không thể duy trì bằng cách vá các vấn đề thay vì thực hiện sửa chữa thực sự. Các quản lý và lãnh đạo kỹ thuật đóng vai trò quan trọng trong việc triển khai các sửa chữa thực sự, dài hạn bằng cách hỗ trợ và ưu tiên hóa các sửa chữa dài hạn có thể tốn thời gian, ngay cả khi cơn "đau" ban đầu của việc gọi trực đã dịu đi.

Việc xử lý các lần gọi trực theo phản xạ máy móc, kiểu thuật toán, là một dấu hiệu cảnh báo. Nếu đội của bạn không sẵn lòng tự động hóa những lần gọi trực như vậy, điều đó cho thấy họ thiếu niềm tin vào khả năng xử lý nợ kỹ thuật. Đây là vấn đề nghiêm trọng cần được leo thang (escalate).

## Về Lâu dài (The Long Run)

Một chủ đề phổ biến nối liền các ví dụ Bigtable và Gmail trước đó: một căng thẳng giữa khả dụng ngắn hạn và dài hạn. Thường thì, sức mạnh thuần túy của nỗ lực có thể giúp một hệ thống chệch choạc đạt khả dụng cao, nhưng con đường này thường ngắn ngủi, đầy kiệt sức và phụ thuộc vào một vài thành viên đội anh hùng. Chấp nhận một sự giảm khả dụng ngắn hạn có kiểm soát thường là một đánh đổi đau đớn, nhưng là đánh đổi chiến lược cho sự ổn định dài hạn của hệ thống. Quan trọng là không nên coi mỗi lần gọi trực như một sự kiện cô lập, mà xem xét liệu mức độ *tổng thể* của việc gọi trực có dẫn đến một hệ thống khỏe mạnh, khả dụng phù hợp với một đội khỏe mạnh, khả thi và có triển vọng dài hạn không. Chúng tôi xem xét các thống kê về tần suất gọi trực trong các báo cáo hàng quý với quản lý (thường biểu đạt như số incident mỗi ca trực, trong đó một incident có thể gồm một vài lần gọi trực liên quan). Việc này đảm bảo những người ra quyết định luôn được cập nhật về tải máy gọi trực và sức khỏe tổng thể của các đội.

## Kết luận

Một hệ thống giám sát và cảnh báo hiệu quả thường đơn giản và dễ giải thích. Nó tập trung chủ yếu vào các triệu chứng thuộc diện gọi trực, đồng thời sử dụng các heuristic (quy tắc kinh nghiệm) định hướng theo nguyên nhân để hỗ trợ debug. Việc giám sát triệu chứng sẽ dễ dàng hơn khi bạn thực hiện ở các tầng cao trong stack, mặc dù việc giám sát độ bão hòa và hiệu năng của các hệ thống con như database thường phải được thực hiện trực tiếp trên chính hệ thống con đó. Các cảnh báo qua email có giá trị rất hạn chế và dễ bị tràn ngập bởi nhiễu; thay vào đó, bạn nên ưu tiên một dashboard để theo dõi tất cả các vấn đề subcritical (dưới mức nghiêm trọng) đang tiếp diễn, nhằm xử lý loại thông tin thường xuất hiện trong các cảnh báo email. Dashboard này cũng có thể kết hợp với log để phân tích các tương quan lịch sử.

Về lâu dài, để duy trì chế độ trực on-call và có được một sản phẩm thành công, cần chọn cảnh báo dựa trên các triệu chứng hoặc vấn đề thực sự sắp xảy ra, điều chỉnh các mục tiêu sao cho thực sự khả thi, và đảm bảo hệ thống giám sát hỗ trợ chẩn đoán nhanh chóng.

<a id="fn1"></a>[1](#fn1) Đôi khi gọi là "alert spam" (spam cảnh báo), vì chúng hiếm khi được đọc hoặc hành động.

<a id="fn2"></a>[2](#fn2) Nếu 1% các yêu cầu của bạn chậm gấp 50 lần giá trị trung bình, điều đó có nghĩa phần còn lại các yêu cầu của bạn nhanh gấp đôi giá trị trung bình. Nhưng nếu bạn không đang đo phân bố, ý tưởng rằng phần lớn các yêu cầu nằm gần giá trị trung bình chỉ là suy nghĩ theo hướng lạc quan.

<a id="fn3"></a>[3](#fn3) Xem *Applying Cardiac Alarm Management Techniques to Your On-Call* (Áp dụng Các Kỹ thuật Quản lý Còi báo Tim vào On-call của Bạn) [[Hol14]](https://sre.google/sre-book/bibliography#Hol14) cho một ví dụ về sự mệt mỏi cảnh báo trong một ngữ cảnh khác.

<a id="fn4"></a>[4](#fn4) Các tình huống không có dự phòng (*N* + 0) được tính là sắp xảy ra, cũng như các phần "gần như đầy" của dịch vụ bạn! Để biết thêm chi tiết về khái niệm dự phòng, xem [*https://en.wikipedia.org/wiki/N%2B1_redundancy*](https://en.wikipedia.org/wiki/N%2B1_redundancy).

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
