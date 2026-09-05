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

Trong SRE (Site Reliability Engineering — kỹ thuật độ tin cậy trang web), chúng tôi muốn dành thời gian cho các dự án kỹ thuật (engineering) dài hạn thay vì công việc vận hành. Vì thuật ngữ *công việc vận hành* dễ gây hiểu nhầm, chúng tôi dùng một từ cụ thể hơn: *toil*.

## Định nghĩa Toil

Toil không chỉ đơn thuần là "công việc tôi không thích làm". Nó cũng không đồng nghĩa với các công việc hành chính hay những việc lặt vặt bẩn thỉu (grungy work). Sự thỏa mãn và thú vị trong công việc là khác nhau ở mỗi người; một số người thậm chí còn thích làm các việc thủ công, lặp đi lặp lại. Cũng có những việc hành chính bắt buộc phải làm, nhưng không nên xếp vào toil: đó là *overhead* (công việc phụ). Overhead thường là những việc không gắn trực tiếp đến việc vận hành một [dịch vụ production](https://sre.google/sre-book/service-best-practices/), bao gồm các tác vụ như họp đội, đặt và chấm điểm mục tiêu,<sup>[1](#fn1)</sup> snippets,<sup>[2](#fn2)</sup> và giấy tờ nhân sự (HR). Công việc lặt vặt bẩn thỉu đôi khi mang lại giá trị dài hạn, và trong trường hợp đó nó cũng không phải toil. Dọn dẹp toàn bộ cấu hình cảnh báo (alerting configuration) cho dịch vụ của bạn và loại bỏ phần lộn xộn có thể là việc lặt vặt, nhưng không phải toil.

Vậy *gì* mới là toil? Toil là loại công việc gắn với việc vận hành một dịch vụ production, thường mang tính thủ công, lặp đi lặp lại, có thể tự động hóa, mang tính chiến thuật (tactical), không có giá trị bền vững, và tăng tuyến tính khi dịch vụ lớn lên. Không phải mọi tác vụ bị coi là toil đều có đủ những thuộc tính này, nhưng công việc càng khớp với một hoặc nhiều mô tả sau thì càng có khả năng là toil:

#### Thủ công (Manual)

Việc này bao gồm cả việc chạy thủ công một script (chương trình) tự động hóa một số tác vụ. Chạy script có thể nhanh hơn làm thủ công từng bước trong script, nhưng khoảng thời gian *thực hiện bằng tay* (hands-on) mà một người bỏ ra để chạy script đó (không phải thời gian trôi qua) vẫn là toil.

#### Lặp đi lặp lại (Repetitive)

Nếu bạn đang làm một việc lần đầu tiên trong đời, hoặc thậm chí chỉ là lần thứ hai, thì đó không phải là toil. Toil là những việc bạn phải làm đi làm lại. Tương tự, nếu bạn đang giải quyết một vấn đề mới hoặc phát minh ra một giải pháp mới, công việc đó cũng không phải là toil.

#### Có thể tự động hóa (Automatable)

Nếu một máy có thể thực hiện tác vụ đó tốt như con người, hoặc yêu cầu của tác vụ có thể được thiết kế để loại bỏ, thì đó là toil. Nếu cần đến sự phán đoán của con người, khả năng cao nó không phải toil.<sup>[3](#fn3)</sup>

#### Mang tính chiến thuật (Tactical)

Toil mang tính phản ứng (reactive), bắt nguồn từ sự gián đoạn (interrupt-driven), chứ không phải chủ động (proactive) hay dựa trên chiến lược (strategy-driven). Việc xử lý các cảnh báo máy gọi trực (pager) chính là toil. Chúng tôi có thể không bao giờ loại bỏ hoàn toàn loại công việc này, nhưng phải liên tục hướng đến việc giảm thiểu nó.

#### Không có giá trị bền vững (No enduring value)

Nếu dịch vụ của bạn vẫn giữ nguyên trạng thái sau khi hoàn thành một tác vụ, khả năng cao đó là toil. Ngược lại, nếu tác vụ mang lại cải tiến lâu dài cho dịch vụ, nó có khả năng không phải toil, ngay cả khi kèm theo một lượng công việc lặt vặt — chẳng hạn đào sâu vào code và cấu hình cũ rồi sắp xếp chúng cho gọn gàng.

#### Tuyến tính O(n) theo sự tăng trưởng dịch vụ

Nếu khối lượng công việc của một tác vụ tăng tuyến tính theo kích thước dịch vụ, lưu lượng traffic hay số người dùng, tác vụ đó có khả năng là toil. Một dịch vụ được quản lý và thiết kế tốt có thể lớn lên ít nhất một bậc (order of magnitude) mà không cần thêm công việc, ngoài một vài nỗ lực một lần để bổ sung tài nguyên.

## Vì sao Ít Toil hơn Là Tốt hơn

Tổ chức SRE của chúng tôi công bố mục tiêu giữ công việc vận hành (tức toil) dưới 50% thời gian của mỗi SRE. Ít nhất 50% thời gian mỗi SRE nên dành cho công việc dự án kỹ thuật giúp giảm toil trong tương lai hoặc thêm tính năng dịch vụ. Phát triển tính năng thường tập trung vào cải thiện độ tin cậy (reliability), hiệu năng (performance) hoặc mức sử dụng (utilization), và thường giảm toil như một hiệu ứng bậc hai.

Chúng tôi đặt mục tiêu 50% này vì toil có xu hướng phình to nếu không được kiểm soát và có thể nhanh chóng chiếm trọn 100% thời gian của mọi người. Công việc giảm toil và scale các dịch vụ chính là phần "Engineering" (kỹ thuật) trong Site Reliability Engineering. Nhờ công việc kỹ thuật, tổ chức SRE có thể scale lên dưới tuyến tính (sublinearly) theo kích thước dịch vụ và quản lý các dịch vụ hiệu quả hơn so với cả một đội Dev (phát triển) thuần túy lẫn một đội Ops (vận hành) thuần túy.

Hơn nữa, khi tuyển SRE mới, chúng tôi cam kết với họ rằng SRE không phải là một tổ chức Ops điển hình, viện dẫn quy tắc 50% vừa nêu. Chúng tôi cần giữ lời hứa đó bằng cách không để tổ chức SRE hay bất kỳ nhóm con nào thoái hóa thành một nhóm Ops.

### Tính toán Toil (Calculating Toil)

Nếu chúng tôi tìm cách giới hạn thời gian một SRE dành cho toil ở mức 50%, thời gian đó được sử dụng như thế nào?

Mọi SRE đều phải chịu một mức sàn (floor) nhất định về lượng toil khi on-call (trực sự cố). Trong mỗi chu kỳ, một SRE điển hình sẽ đảm nhận một tuần on-call chính (primary) và một tuần on-call phụ (secondary) (để thảo luận về ca trực chính so với phụ, xem [Being On-Call](https://sre.google/sre-book/being-on-call/)). Do đó, trong một vòng trực 6 người, ít nhất 2 trên 6 tuần sẽ dành cho ca trực on-call và [xử lý sự gián đoạn](https://sre.google/sre-book/dealing-with-interrupts/) — tức là cận dưới (lower bound) của toil tiềm năng chiếm 2/6 = 33% thời gian của một SRE. Với vòng trực 8 người, con số này là 2/8 = 25%.

Đúng với dữ liệu này, các SRE cho biết nguồn toil lớn nhất đến từ những lần bị gián đoạn — tức các thông báo và email liên quan đến dịch vụ nhưng không khẩn cấp. Tiếp theo là việc phản hồi các sự cố on-call (khẩn cấp), rồi đến các lần phát hành và đẩy code. Mặc dù quy trình phát hành và đẩy code của chúng tôi đã được tự động hóa đáng kể, vẫn còn rất nhiều dư địa để cải thiện.

Các cuộc khảo sát hàng quý của SRE tại Google cho thấy thời gian trung bình dành cho toil rơi vào khoảng 33%, tốt hơn nhiều so với mục tiêu tổng thể 50%. Tuy nhiên, con số trung bình này không phản ánh được các trường hợp ngoại lệ: một số SRE báo cáo tỷ lệ toil là 0% (dự án phát triển thuần túy, không có on-call), trong khi những người khác lại lên tới 80%. Khi một SRE cá nhân báo cáo mức toil quá cao, điều đó thường cho thấy quản lý cần phân bố tải toil đều hơn trong nhóm và khuyến khích SRE đó tìm các dự án kỹ thuật thỏa mãn hơn.

## Những gì Đủ tiêu chuẩn là Kỹ thuật (Engineering)?

Công việc kỹ thuật mang tính mới mẻ (novel) và đòi hỏi sự phán đoán của con người. Nó tạo ra cải tiến lâu dài cho dịch vụ của bạn, xuất phát từ một chiến lược. Công việc này thường mang tính sáng tạo, đổi mới, áp dụng cách tiếp cận dựa trên thiết kế để giải quyết vấn đề — càng tổng quát hóa càng tốt. Nhờ đó, đội của bạn hoặc tổ chức SRE có thể xử lý một dịch vụ lớn hơn, hoặc nhiều dịch vụ hơn, với cùng mức nhân sự.

Các hoạt động SRE điển hình rơi vào các phạm trù gần đúng sau:

#### Kỹ thuật phần mềm (Software engineering)

Bao gồm việc viết hoặc sửa code, cùng với các công việc thiết kế và tài liệu liên quan (nếu có). Ví dụ như viết script tự động hóa, tạo công cụ hoặc framework, thêm tính năng dịch vụ để tăng khả năng scale và độ tin cậy, hoặc sửa code hạ tầng nhằm tăng tính chống chịu.

#### Kỹ thuật hệ thống (Systems engineering)

Bao gồm việc cấu hình các hệ thống production, sửa đổi cấu hình hoặc tài liệu hóa hệ thống theo cách tạo ra cải tiến lâu dài từ một nỗ lực một lần. Các ví dụ bao gồm thiết lập và cập nhật giám sát (monitoring), cấu hình load balancing (cân bằng tải), cấu hình server, điều chỉnh tham số hệ điều hành (OS) và thiết lập load balancer. Kỹ thuật hệ thống cũng bao gồm tư vấn về kiến trúc (architecture), thiết kế và production hóa (productionization) cho các đội developer.

**Toil**

Công việc gắn trực tiếp đến việc vận hành một dịch vụ, lặp đi lặp lại, thủ công, v.v.

**Overhead**

Công việc hành chính không gắn trực tiếp đến việc vận hành một dịch vụ. Các ví dụ bao gồm tuyển dụng, giấy tờ nhân sự, họp đội/công ty, vệ sinh hàng đợi bug (bug queue hygiene), snippets, đánh giá đồng nghiệp và tự đánh giá, và các khóa đào tạo.

Mỗi SRE cần dành ít nhất 50% thời gian cho công việc kỹ thuật, tính trung bình trên vài quý hoặc một năm. Vì Toil có xu hướng đột biến (spiky), việc duy trì mức 50% thời gian ổn định cho kỹ thuật có thể không thực tế với một số đội SRE, và họ có thể tụt xuống dưới mục tiêu trong vài quý. Tuy nhiên, nếu tỷ lệ thời gian dành cho dự án, xét về lâu dài, thấp hơn đáng kể so với 50%, đội bị ảnh hưởng cần dừng lại và tìm ra điều gì đang sai.

## Toil Có luôn luôn là xấu không?

Toil không phải lúc nào cũng gây khó chịu, nhất là khi khối lượng còn nhỏ. Những tác vụ lặp đi lặp lại, có thể dự đoán trước thường khá yên ả. Chúng mang lại cảm giác đạt được mục tiêu cùng những chiến thắng nhanh (quick wins). Đây có thể là các hoạt động rủi ro thấp và ít căng thẳng. Một số người thậm chí bị thu hút bởi các tác vụ thuộc toil và tận hưởng loại công việc này.

Toil không phải lúc nào cũng xấu. Cần nhận thức rõ rằng một lượng toil nhất định là không thể tránh khỏi trong vai trò SRE, và thực ra trong gần như mọi vai trò kỹ thuật. Ở liều lượng nhỏ thì không sao, và nếu bạn hài lòng với những liều nhỏ đó, toil không phải là vấn đề. Toil trở nên độc hại khi phải làm với khối lượng lớn. Nếu bạn bị gánh quá nhiều toil, bạn nên lo lắng và lên tiếng khiếu nại. Trong số nhiều lý do khiến quá nhiều toil là xấu, hãy xem các điều sau:

**Đình trệ sự nghiệp**

Sự nghiệp của bạn sẽ chững lại hoặc thậm chí đình trệ nếu bạn dành quá ít thời gian cho các dự án. Google ghi nhận những công việc lặt vặt, nhàm chán khi chúng không thể tránh khỏi và mang lại tác động tích cực lớn, nhưng bạn không thể xây dựng sự nghiệp dựa trên những việc lặt vặt đó.

**Tinh thần thấp**

Mỗi người có ngưỡng chịu đựng toil khác nhau, nhưng ai cũng có giới hạn nhất định. Khi toil quá nhiều, người ta sẽ rơi vào trạng thái kiệt sức (burnout), chán nản và bất mãn.

Hơn nữa, việc dành quá nhiều thời gian cho toil với giá phải trả là thời gian dành cho kỹ thuật gây tổn hại cho một tổ chức SRE theo các cách sau:

**Tạo sự nhầm lẫn**

Chúng tôi luôn cố gắng để mọi người làm việc trong hoặc cùng với tổ chức SRE hiểu rõ đây là một tổ chức kỹ thuật. Nếu các cá nhân hoặc đội trong SRE phải dành quá nhiều thời gian cho toil, thông điệp đó sẽ bị làm mờ và khiến người khác nhầm lẫn về vai trò của chúng tôi.

**Làm chậm tiến độ**

Toil quá mức khiến năng suất của đội giảm. Feature velocity (tốc độ tính năng) của một sản phẩm sẽ chậm lại nếu đội SRE quá bận với các công việc thủ công và chữa cháy (firefighting) đến mức không kịp triển khai các tính năng mới.

**Đặt tiền lệ**

Nếu bạn quá sẵn lòng nhận toil, các đối tác Dev của bạn sẽ có động lực chất thêm toil cho bạn, đôi khi đẩy sang SRE những tác vụ vận hành lẽ ra Dev nên làm. Các đội khác cũng có thể bắt đầu kỳ vọng SRE nhận công việc như vậy, điều này là xấu vì những lý do rõ ràng.

**Thúc đẩy sự rời bỏ**

Dù bản thân bạn có thể không bị ảnh hưởng nhiều bởi toil, các đồng đội hiện tại hoặc tương lai lại có thể cảm thấy điều này khó chịu hơn rất nhiều. Nếu bạn nhồi nhét quá nhiều toil vào quy trình của đội, bạn sẽ khiến những kỹ sư giỏi nhất bắt đầu tìm kiếm một công việc xứng đáng hơn ở nơi khác.

**Gây vi phạm niềm tin**

Các nhân viên mới hoặc người chuyển đến gia nhập SRE với lời hứa về công việc dự án sẽ cảm thấy bị lừa, điều này là xấu cho tinh thần.

## Kết luận

Nếu tất cả chúng ta cùng cam kết cắt giảm một chút toil mỗi tuần nhờ một số kỹ thuật tốt, chúng tôi sẽ dần dọn sạch các dịch vụ của mình, qua đó có thể chuyển nỗ lực tập thể sang kỹ thuật cho scale, kiến trúc thế hệ dịch vụ tiếp theo và xây dựng các toolchain (chuỗi công cụ) xuyên SRE. Hãy sáng chế nhiều hơn và toil ít hơn.

<a id="fn1"></a>[1](#fn1) Chúng tôi sử dụng hệ thống Objectives and Key Results (Mục tiêu và Kết quả Chính), do Andy Grove tại Intel tiên phong; xem [[Kla12]](https://sre.google/sre-book/bibliography#Kla12).

<a id="fn2"></a>[2](#fn2) Các Googler ghi lại các bản tóm tắt ngắn dạng tự do, hoặc "snippets", về những gì chúng tôi đã làm việc trong mỗi tuần.

<a id="fn3"></a>[3](#fn3) Chúng tôi phải cẩn thận khi nói một tác vụ là "không phải toil vì nó cần sự phán đoán của con người". Cần suy nghĩ kỹ xem bản chất tác vụ có thực sự đòi hỏi phán đoán của con người và không thể giải quyết bằng một thiết kế tốt hơn không. Ví dụ, một người có thể xây dựng (và một số đã xây dựng) một dịch vụ cảnh báo các SRE của nó vài lần mỗi ngày, trong đó mỗi cảnh báo đòi hỏi một phản hồi phức tạp cần nhiều phán đoán của con người. Một dịch vụ như vậy được thiết kế kém, với sự phức tạp không cần thiết. Hệ thống cần được đơn giản hóa và xây dựng lại, hoặc để loại bỏ các điều kiện hỏng hóc cơ bản hoặc để xử lý chúng tự động. Cho đến khi thiết kế lại và triển khai lại hoàn tất và dịch vụ cải tiến được đưa lên, công việc dùng phán đoán của con người để phản hồi mỗi cảnh báo chắc chắn là toil.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
