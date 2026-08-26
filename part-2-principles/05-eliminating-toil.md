# Chương 5. Loại bỏ Toil (Eliminating Toil)

> **Nguyên bản:** [Chapter 5 - Eliminating Toil](https://sre.google/sre-book/eliminating-toil/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Vivek Rau
*Biên tập:* Betsy Beyer

> Nếu một người vận hành (operator) phải chạm vào hệ thống của bạn trong các hoạt động bình thường, thì bạn đang có một bug. Định nghĩa của "bình thường" sẽ thay đổi khi hệ thống của bạn lớn lên.
>
> Carla Geisser, Google SRE

Trong SRE (Site Reliability Engineering — kỹ thuật độ tin cậy trang web), chúng tôi muốn dành thời gian cho công việc dự án kỹ thuật (engineering) dài hạn thay vì công việc vận hành. Vì thuật ngữ *công việc vận hành* có thể gây hiểu nhầm, chúng tôi dùng một từ cụ thể hơn: *toil*.

## Định nghĩa Toil

Toil không chỉ là "công việc tôi không thích làm". Nó cũng không đơn giản đồng nghĩa với các công việc hành chính hay công việc lặt vặt bẩn thỉu (grungy work). Loại công việc nào thỏa mãn và thú vị khác nhau ở mỗi người; một số người thậm chí thích làm các việc thủ công, lặp đi lặp lại. Cũng có những việc hành chính phải làm, nhưng không nên xếp vào toil: đó là *overhead* (công việc phụ). Overhead thường là việc không gắn trực tiếp đến việc vận hành một [dịch vụ production](https://sre.google/sre-book/service-best-practices/), bao gồm các tác vụ như họp đội, đặt và chấm điểm mục tiêu,<sup>[1](#fn1)</sup> snippets,<sup>[2](#fn2)</sup> và giấy tờ nhân sự (HR). Công việc lặt vặt bẩn thỉu đôi khi có giá trị dài hạn, và trong trường hợp đó nó cũng không phải toil. Dọn dẹp toàn bộ cấu hình cảnh báo (alerting configuration) cho dịch vụ của bạn và bỏ đi phần lộn xộn có thể là việc lặt vặt, nhưng không phải toil.

Vậy *gì* mới là toil? Toil là loại công việc gắn với việc vận hành một dịch vụ production, thường mang tính thủ công, lặp đi lặp lại, có thể tự động hóa, mang tính chiến thuật (tactical), không có giá trị bền vững, và tăng tuyến tính khi dịch vụ lớn lên. Không phải mọi tác vụ bị coi là toil đều có đủ những thuộc tính này, nhưng công việc càng khớp với một hoặc nhiều mô tả sau thì càng có khả năng là toil:

#### Thủ công (Manual)

Bao gồm các việc như chạy thủ công một script (chương trình) tự động hóa một số tác vụ. Chạy script có thể nhanh hơn việc làm thủ công từng bước trong script, nhưng thời gian *thực hiện bằng tay* (hands-on) mà một người dành để chạy script đó (không phải thời gian trôi qua) vẫn là thời gian toil.

#### Lặp đi lặp lại (Repetitive)

Nếu bạn đang thực hiện một tác vụ lần đầu tiên trong đời, hoặc thậm chí lần thứ hai, công việc này không phải là toil. Toil là công việc bạn làm đi làm lại. Nếu bạn đang giải quyết một vấn đề mới hoặc phát minh ra một giải pháp mới, công việc này không phải là toil.

#### Có thể tự động hóa (Automatable)

Nếu một máy có thể làm tác vụ đó tốt như một con người, hoặc nhu cầu của tác vụ có thể được thiết kế để loại bỏ, thì tác vụ đó là toil. Nếu cần đến sự phán đoán của con người, khả năng cao nó không phải toil.<sup>[3](#fn3)</sup>

#### Mang tính chiến thuật (Tactical)

Toil được dẫn dắt bởi sự gián đoạn (interrupt-driven) và mang tính phản ứng (reactive), chứ không phải được dẫn dắt bởi chiến lược (strategy-driven) và chủ động (proactive). Xử lý các cảnh báo pager (thiết bị gọi trực) là toil. Chúng tôi có thể không bao giờ loại bỏ hoàn toàn loại công việc này, nhưng phải liên tục hướng đến việc giảm thiểu nó.

#### Không có giá trị bền vững (No enduring value)

Nếu dịch vụ của bạn vẫn ở cùng một trạng thái sau khi bạn hoàn thành một tác vụ, tác vụ đó có khả năng là toil. Nếu tác vụ tạo ra một cải tiến lâu dài cho dịch vụ, nó có khả năng không phải toil, ngay cả khi có kèm theo một lượng công việc lặt vặt — chẳng hạn đào sâu vào code và cấu hình cũ rồi sắp xếp chúng cho gọn gàng.

**Tuyến tính O(n) theo sự tăng trưởng dịch vụ**

Nếu khối lượng công việc của một tác vụ tăng tuyến tính theo kích thước dịch vụ, lưu lượng traffic hay số người dùng, tác vụ đó có khả năng là toil. Một dịch vụ được quản lý và thiết kế lý tưởng có thể lớn lên ít nhất một bậc (order of magnitude) mà không cần thêm công việc, ngoài một số nỗ lực một lần để thêm tài nguyên.

## Vì sao Ít Toil hơn Là Tốt hơn

Tổ chức SRE của chúng tôi công bố mục tiêu giữ công việc vận hành (tức toil) dưới 50% thời gian của mỗi SRE. Ít nhất 50% thời gian mỗi SRE nên dành cho công việc dự án kỹ thuật giúp giảm toil trong tương lai hoặc thêm tính năng dịch vụ. Phát triển tính năng thường tập trung vào cải thiện độ tin cậy (reliability), hiệu năng (performance) hoặc mức sử dụng (utilization), và thường giảm toil như một hiệu ứng thứ hai.

Chúng tôi chia sẻ mục tiêu 50% này vì toil có xu hướng phình to nếu không được kiểm soát và có thể nhanh chóng lấp đầy 100% thời gian của mọi người. Công việc giảm toil và scale các dịch vụ chính là phần "Engineering" (kỹ thuật) trong Site Reliability Engineering. Công việc kỹ thuật giúp tổ chức SRE scale lên dưới tuyến tính (sublinearly) theo kích thước dịch vụ và quản lý các dịch vụ hiệu quả hơn so với cả một đội Dev (phát triển) thuần túy lẫn một đội Ops (vận hành) thuần túy.

Hơn nữa, khi tuyển SRE mới, chúng tôi hứa với họ rằng SRE không phải là một tổ chức Ops điển hình, viện dẫn quy tắc 50% vừa nêu. Chúng tôi cần giữ lời hứa đó bằng cách không để tổ chức SRE hay bất kỳ đội con nào trong nó thoái hóa thành một đội Ops.

### Tính toán Toil (Calculating Toil)

Nếu chúng tôi tìm cách giới hạn thời gian một SRE dành cho toil ở mức 50%, thời gian đó được sử dụng như thế nào?

Có một mức sàn (floor) cho lượng toil mà bất kỳ SRE nào cũng phải xử lý nếu họ đang on-call (trực sự cố). Một SRE điển hình có một tuần on-call chính (primary) và một tuần on-call phụ (secondary) trong mỗi chu kỳ (để thảo luận về ca trực chính so với phụ, xem [Being On-Call](https://sre.google/sre-book/being-on-call/)). Suy ra, trong một vòng trực 6 người, ít nhất 2 trong mỗi 6 tuần dành cho ca trực on-call và [xử lý sự gián đoạn](https://sre.google/sre-book/dealing-with-interrupts/), nghĩa là cận dưới (lower bound) của toil tiềm năng là 2/6 = 33% thời gian của một SRE. Trong một vòng trực 8 người, cận dưới là 2/8 = 25%.

Nhất quán với dữ liệu này, các SRE báo cáo nguồn toil hàng đầu của họ là các sự gián đoạn (interrupt) — tức các thông báo và email liên quan đến dịch vụ nhưng không khẩn cấp. Nguồn tiếp theo là phản hồi on-call (khẩn cấp), rồi đến các release (phát hành) và push (đẩy code). Dù các quy trình release và push của chúng tôi thường được tự động hóa đáng kể, vẫn còn rất nhiều dư địa để cải thiện.

Các cuộc khảo sát hàng quý của SRE tại Google cho thấy thời gian trung bình dành cho toil khoảng 33%, tốt hơn nhiều so với mục tiêu tổng thể 50%. Tuy nhiên, giá trị trung bình không phản ánh các ngoại lệ: một số SRE báo cáo toil 0% (dự án phát triển thuần túy, không có on-call) và những người khác báo cáo toil 80%. Khi một SRE cá nhân báo cáo toil quá mức, điều đó thường cho thấy quản lý cần phân bố tải toil đều hơn trong đội và khuyến khích SRE đó tìm các dự án kỹ thuật thỏa mãn hơn.

## Những gì Đủ tiêu chuẩn là Kỹ thuật (Engineering)?

Công việc kỹ thuật mang tính mới mẻ (novel) và bản chất đòi hỏi sự phán đoán của con người. Nó tạo ra một cải tiến lâu dài cho dịch vụ của bạn và được dẫn dắt bởi một chiến lược. Nó thường sáng tạo, đổi mới, áp dụng cách tiếp cận dựa trên thiết kế để giải quyết một vấn đề — càng tổng quát hóa càng tốt. Công việc kỹ thuật giúp đội của bạn hoặc tổ chức SRE xử lý một dịch vụ lớn hơn, hoặc nhiều dịch vụ hơn, với cùng mức nhân sự.

Các hoạt động SRE điển hình rơi vào các phạm trù gần đúng sau:

#### Kỹ thuật phần mềm (Software engineering)

Bao gồm viết hoặc sửa đổi code, ngoài bất kỳ công việc thiết kế và tài liệu liên quan nào. Các ví dụ bao gồm viết các script tự động hóa, tạo các công cụ hoặc khung (framework), thêm các tính năng dịch vụ cho khả năng scale và độ tin cậy, hoặc sửa đổi code hạ tầng để làm cho nó chống chịu hơn.

#### Kỹ thuật hệ thống (Systems engineering)

Bao gồm cấu hình các hệ thống production, sửa đổi các cấu hình hoặc tài liệu hóa các hệ thống theo cách tạo ra cải tiến lâu dài từ một nỗ lực một lần. Các ví dụ bao gồm thiết lập và cập nhật giám sát (monitoring), cấu hình load balancing (cân bằng tải), cấu hình server, điều chỉnh tham số hệ điều hành (OS) và thiết lập load balancer. Kỹ thuật hệ thống cũng bao gồm tư vấn về kiến trúc (architecture), thiết kế và production hóa (productionization) cho các đội developer.

**Toil**

Công việc gắn trực tiếp đến việc vận hành một dịch vụ, lặp đi lặp lại, thủ công, v.v.

**Overhead**

Công việc hành chính không gắn trực tiếp đến việc vận hành một dịch vụ. Các ví dụ bao gồm tuyển dụng, giấy tờ nhân sự, họp đội/công ty, vệ sinh hàng đợi bug (bug queue hygiene), snippets, đánh giá đồng nghiệp và tự đánh giá, và các khóa đào tạo.

Mọi SRE cần dành ít nhất 50% thời gian cho công việc kỹ thuật, xét trung bình trên một vài quý hoặc một năm. Toil có xu hướng đột biến (spiky), nên mức 50% thời gian ổn định dành cho kỹ thuật có thể không thực tế với một số đội SRE, và họ có thể sụt xuống dưới mục tiêu trong một vài quý. Nhưng nếu tỷ lệ thời gian dành cho dự án, xét về lâu dài, thấp hơn đáng kể so với 50%, đội bị ảnh hưởng cần lùi lại và tìm ra điều gì đang sai.

## Toil Có luôn luôn là xấu không?

Toil không khiến mọi người không hài lòng mọi lúc, đặc biệt khi ở lượng nhỏ. Các tác vụ có thể dự đoán và lặp đi lặp lại có thể khá yên ả. Chúng tạo ra cảm giác đạt được và những chiến thắng nhanh (quick wins). Chúng có thể là các hoạt động rủi ro thấp và ít căng thẳng. Một số người bị thu hút về phía các tác vụ thuộc toil và thậm chí tận hưởng loại công việc đó.

Toil không phải lúc nào cũng xấu, và mọi người cần nhận thức rõ rằng một lượng toil nhất định là không thể tránh khỏi trong vai trò SRE, và thực ra trong gần như mọi vai trò kỹ thuật. Ở liều lượng nhỏ thì không sao, và nếu bạn hài lòng với những liều nhỏ đó, toil không phải là vấn đề. Toil trở nên độc hại khi trải nghiệm ở số lượng lớn. Nếu bạn bị gánh quá nhiều toil, bạn nên lo lắng và lên tiếng khiếu nại. Trong số nhiều lý do khiến quá nhiều toil là xấu, hãy xem các điều sau:

**Đình trệ sự nghiệp**

Tiến triển sự nghiệp của bạn sẽ chậm lại hoặc dừng hẳn nếu bạn dành quá ít thời gian cho các dự án. Google ghi nhận công việc lặt vặt bẩn thỉu khi nó không thể tránh được và có tác động tích cực lớn, nhưng bạn không thể xây dựng sự nghiệp dựa trên lặt vặt.

**Tinh thần thấp**

Mọi người có các giới hạn khác nhau về lượng toil họ có thể chịu đựng, nhưng mọi người đều có một giới hạn. Quá nhiều toil dẫn đến kiệt sức (burnout), nhàm chán, và bất mãn.

Hơn nữa, việc dành quá nhiều thời gian cho toil với giá phải trả là thời gian dành cho kỹ thuật gây tổn hại cho một tổ chức SRE theo các cách sau:

**Tạo sự nhầm lẫn**

Chúng tôi nỗ lực để đảm bảo mọi người làm việc trong hoặc cùng với tổ chức SRE hiểu rằng chúng tôi là một tổ chức kỹ thuật. Các cá nhân hoặc đội trong SRE tham gia quá nhiều toil sẽ làm mờ đi thông điệp đó và khiến mọi người nhầm lẫn về vai trò của chúng tôi.

**Làm chậm tiến độ**

Toil quá mức khiến một đội kém năng suất hơn. Tốc độ tính năng (feature velocity) của một sản phẩm sẽ chậm lại nếu đội SRE quá bận với việc thủ công và chữa cháy (firefighting) đến mức không kịp triển khai các tính năng mới.

**Đặt tiền lệ**

Nếu bạn quá sẵn lòng nhận toil, các đối tác Dev của bạn sẽ có động lực chất thêm toil cho bạn, đôi khi đẩy sang SRE những tác vụ vận hành lẽ ra Dev nên làm. Các đội khác cũng có thể bắt đầu kỳ vọng SRE nhận công việc như vậy, điều này là xấu vì những lý do rõ ràng.

**Thúc đẩy sự rời bỏ**

Ngay cả khi bạn cá nhân không bất hạnh với toil, các đồng đội hiện tại hoặc tương lai của bạn có thể thích nó ít hơn nhiều. Nếu bạn xây dựng quá nhiều toil vào các quy trình của đội, bạn tạo động lực cho các kỹ sư giỏi nhất của đội bắt đầu tìm kiếm nơi khác một công việc có phần thưởng hơn.

**Gây vi phạm niềm tin**

Các nhân viên mới hoặc người chuyển đến gia nhập SRE với lời hứa về công việc dự án sẽ cảm thấy bị lừa, điều này là xấu cho tinh thần.

## Kết luận

Nếu tất cả chúng ta cam kết loại bỏ một chút toil mỗi tuần bằng một số kỹ thuật tốt, chúng tôi sẽ dần dần dọn sạch các dịch vụ của mình và có thể chuyển nỗ lực tập thể sang kỹ thuật cho scale, kiến trúc thế hệ dịch vụ tiếp theo và xây dựng các toolchain (chuỗi công cụ) xuyên SRE. Hãy sáng chế nhiều hơn và toil ít hơn.

<a id="fn1"></a>[1](#fn1) Chúng tôi sử dụng hệ thống Objectives and Key Results (Mục tiêu và Kết quả Chính), được tiên phong bởi Andy Grove tại Intel; xem [[Kla12]](https://sre.google/sre-book/bibliography#Kla12).

<a id="fn2"></a>[2](#fn2) Các Googler ghi lại các bản tóm tắt ngắn dạng tự do, hoặc "snippets", về những gì chúng tôi đã làm việc trong mỗi tuần.

<a id="fn3"></a>[3](#fn3) Chúng tôi phải cẩn thận khi nói một tác vụ là "không phải toil vì nó cần sự phán đoán của con người". Cần suy nghĩ kỹ xem bản chất tác vụ có thực sự đòi hỏi phán đoán của con người và không thể giải quyết bằng một thiết kế tốt hơn không. Ví dụ, một người có thể xây dựng (và một số đã xây dựng) một dịch vụ cảnh báo các SRE của nó vài lần mỗi ngày, trong đó mỗi cảnh báo đòi hỏi một phản hồi phức tạp cần nhiều phán đoán của con người. Một dịch vụ như vậy được thiết kế kém, với sự phức tạp không cần thiết. Hệ thống cần được đơn giản hóa và xây dựng lại, hoặc để loại bỏ các điều kiện hỏng hóc cơ bản hoặc để xử lý chúng một cách tự động. Cho đến khi thiết kế lại và triển khai lại hoàn tất và dịch vụ cải tiến được đưa lên, công việc dùng phán đoán của con người để phản hồi mỗi cảnh báo chắc chắn là toil.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
