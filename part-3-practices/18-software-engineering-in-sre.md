# Chương 18. Kỹ thuật Phần mềm trong SRE (Software Engineering in SRE)

> **Nguyên bản:** [Chapter 18 - Software Engineering in SRE](https://sre.google/sre-book/software-engineering-in-sre/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Dave Helstroom và Trisha Weir cùng Evan Leonard và Kurt Delimon
*Biên tập:* Kavita Guliani

Hỏi ai đó nêu tên một nỗ lực kỹ thuật phần mềm của Google và họ có thể liệt kê một sản phẩm hướng người tiêu dùng như Gmail hay Maps; một số người có thể nhắc đến hạ tầng cơ bản như Bigtable hay Colossus. Nhưng thực tế, có một lượng khổng lồ công việc kỹ thuật phần mềm diễn ra sau cánh gà mà người tiêu dùng không bao giờ thấy. Một số sản phẩm đó được phát triển bên trong SRE.

Môi trường production của Google là — theo một số thước đo — một trong những cỗ máy phức tạp nhất mà nhân loại từng xây dựng. SRE có kinh nghiệm tận mắt với những tinh vi của production, khiến họ đặc biệt phù hợp để phát triển các công cụ giải quyết các vấn đề và use case nội bộ liên quan đến việc giữ production hoạt động. Phần lớn các công cụ này liên quan đến mục tiêu tổng thể duy trì uptime và giữ độ trễ (latency) thấp, nhưng mang nhiều hình thức: ví dụ bao gồm các cơ chế rollout binary, giám sát, hoặc một môi trường phát triển được xây dựng trên việc tổ hợp server động. Nhìn chung, những công cụ do SRE phát triển này là các dự án kỹ thuật phần mềm đầy đủ, khác với các giải pháp dùng một lần và các hack nhanh, và các SRE phát triển chúng đã áp dụng một tâm thế dựa trên sản phẩm (product-based mindset), tính đến cả khách hàng nội bộ lẫn một bản đồ đường (roadmap) cho các kế hoạch trong tương lai.

## Tại sao Kỹ thuật Phần mềm Bên trong SRE lại Quan trọng? (Why Is Software Engineering Within SRE Important?)

Theo nhiều cách, quy mô rộng lớn của production Google đã đòi hỏi phát triển phần mềm nội bộ, vì ít có công cụ bên thứ ba nào được thiết kế ở quy mô đủ lớn cho nhu cầu của Google. Lịch sử các dự án phần mềm thành công của công ty đã khiến chúng tôi đánh giá cao lợi ích của việc phát triển trực tiếp bên trong SRE.

Các SRE ở một vị trí độc đáo để phát triển phần mềm nội bộ một cách hiệu quả vì một số lý do:

-   Phạm vi và độ sâu của kiến thức production cụ thể của Google trong tổ chức SRE cho phép kỹ sư của nó thiết kế và tạo phần mềm với các cân nhắc phù hợp cho các chiều kích như khả năng scale, suy giảm nhẹ nhàng (graceful degradation) khi thất bại, và khả năng dễ dàng giao tiếp với hạ tầng hoặc công cụ khác.
-   Vì SRE gắn bó với đối tượng, họ dễ hiểu các nhu cầu và yêu cầu của công cụ đang phát triển.
-   Mối quan hệ trực tiếp với người dùng dự định — các SRE đồng nghiệp — dẫn đến phản hồi người dùng thẳng thắn và có tín hiệu cao (high-signal). Việc release một công cụ đến một khán giả nội bộ quen thuộc với không gian vấn đề nghĩa là đội phát triển có thể khởi động và lặp lại (iterate) nhanh hơn. Người dùng nội bộ thường dễ chịu hơn với UI tối thiểu và các vấn đề sản phẩm alpha.

Từ quan điểm thuần túy thực dụng, Google rõ ràng hưởng lợi từ việc có kỹ sư có kinh nghiệm SRE phát triển phần mềm. Theo thiết kế chủ ý, tốc độ tăng trưởng của các dịch vụ được SRE hỗ trợ vượt quá tốc độ tăng trưởng của tổ chức SRE; một trong những nguyên lý chỉ đường của SRE là "kích thước đội không nên scale trực tiếp với sự tăng trưởng dịch vụ." Việc đạt được tăng trưởng đội tuyến tính trước sự tăng trưởng dịch vụ theo hàm mũ đòi hỏi công việc tự động hóa không ngừng và các nỗ lực tinh giản công cụ, quy trình, và các khía cạnh khác của dịch vụ vốn giới thiệu sự kém hiệu quả vào vận hành production hàng ngày. Việc để những người có kinh nghiệm trực tiếp vận hành các hệ thống production phát triển các công cụ cuối cùng sẽ đóng góp vào các mục tiêu uptime và độ trễ là rất hợp lý.

Mặt khác, các SRE cá nhân, cũng như tổ chức SRE rộng lớn hơn, cũng hưởng lợi từ việc phát triển phần mềm do SRE thúc đẩy.

Các dự án phát triển phần mềm đầy đủ bên trong SRE cung cấp cơ hội phát triển nghề nghiệp cho SRE, cũng như một lối thoát cho các kỹ sư không muốn kỹ năng code của họ bị gỉ sét. Công việc dự án dài hạn cung cấp sự cân bằng cần thiết cho các ngắt (interrupts) và công việc on-call, và có thể mang lại sự hài lòng cho các kỹ sư muốn sự nghiệp của họ duy trì sự cân bằng giữa [kỹ thuật phần mềm và kỹ thuật hệ thống](https://sre.google/resources/practices-and-processes/enterprise-roadmap-to-sre/).

Ngoài việc thiết kế các công cụ tự động hóa và các nỗ lực khác để giảm khối lượng công việc cho kỹ sư trong SRE, các dự án phát triển phần mềm có thể tiếp tục mang lại lợi ích cho tổ chức SRE bằng cách thu hút và giữ chân các kỹ sư có nhiều loại kỹ năng khác nhau. Sự mong muốn của một đội đa dạng càng đúng với SRE, nơi một loạt nền tảng và cách tiếp cận giải quyết vấn đề có thể giúp ngăn các điểm mù (blind spots). Vì mục đích này, Google luôn cố bố trí nhân sự cho các đội SRE bằng một sự pha trộn giữa kỹ sư có kinh nghiệm phát triển phần mềm truyền thống và kỹ sư có kinh nghiệm kỹ thuật hệ thống.

## Nghiên cứu Tình huống Auxon: Bối cảnh Dự án và Không gian Vấn đề (Auxon Case Study: Project Background and Problem Space)

Nghiên cứu tình huống này xem xét Auxon, một công cụ mạnh mẽ được phát triển bên trong SRE để tự động hóa việc lập kế hoạch năng lực (capacity planning) cho các dịch vụ đang chạy trong production Google. Để hiểu tốt nhất Auxon ra đời thế nào và giải quyết những vấn đề gì, chúng tôi trước tiên xem xét không gian vấn đề liên quan đến lập kế hoạch năng lực, cùng những khó khăn mà các cách tiếp cận truyền thống đặt ra cho các dịch vụ tại Google và trong cả ngành công nghiệp. Để có thêm ngữ cảnh về cách Google dùng các thuật ngữ *dịch vụ* (service) và *cluster* (cụm máy), xem [The Production Environment at Google, from the Viewpoint of an SRE](https://sre.google/sre-book/production-environment/).

## Lập kế hoạch Năng lực Truyền thống (Traditional Capacity Planning)

Có vô số chiến thuật cho việc lập kế hoạch năng lực của các tài nguyên tính toán (xem [[Hix15a]](https://sre.google/sre-book/bibliography#Hix15a)), nhưng phần lớn các cách tiếp cận này rút gọn thành một *chu kỳ* (cycle) có thể xấp xỉ như sau:

1) Thu thập các dự báo nhu cầu (demand forecasts).

Cần bao nhiêu tài nguyên? Khi nào và ở đâu những tài nguyên này được cần?

-   Sử dụng dữ liệu tốt nhất khả dụng ngày hôm nay để lập kế hoạch vào tương lai
-   Thường bao phủ bất kỳ đâu từ vài quý đến vài năm

2) Xây dựng các kế hoạch xây dựng và cấp phát.

Với triển vọng dự báo này, cách tốt nhất để đáp ứng nhu cầu này bằng nguồn cung cấp bổ sung tài nguyên là gì? Bao nhiêu nguồn cung, và ở những vị trí nào?

3) Xem xét và ký phê duyệt kế hoạch.

Dự báo có hợp lý không? Kế hoạch có khớp với các cân nhắc ngân sách, cấp độ sản phẩm, và kỹ thuật không?

4) Triển khai và cấu hình tài nguyên.

Một khi các tài nguyên cuối cùng đến (có thể theo các giai đoạn trong suốt một khoảng thời gian được định nghĩa nhất định), dịch vụ nào được sử dụng các tài nguyên? Làm thế nào để tôi làm cho các tài nguyên cấp thấp hơn thông thường (CPU, disk (ổ đĩa), v.v.) hữu ích cho các dịch vụ?

Nên nhấn mạnh rằng lập kế hoạch năng lực là một *chu kỳ* không bao giờ kết thúc: các giả định thay đổi, các triển khai trượt (slip), và ngân sách bị cắt, dẫn đến những lần sửa đổi liên tiếp trên "Kế hoạch" (The Plan). Mỗi sửa đổi có các hiệu ứng lan truyền (trickle-down) phải lan ra xuyên suốt kế hoạch của tất cả các quý tiếp theo. Ví dụ, một sự thiếu hụt trong quý này phải được bù đắp trong các quý sau. Lập kế hoạch năng lực truyền thống dùng nhu cầu làm động lực chính, và thủ công tạo hình nguồn cung để khớp với nhu cầu đáp lại mỗi thay đổi.

### Vỡ vụn theo bản chất (Brittle by nature)

Lập kế hoạch năng lực truyền thống tạo ra một kế hoạch cấp phát tài nguyên có thể bị gián đoạn bởi bất kỳ thay đổi nào tưởng chừng nhỏ nhặt. Ví dụ:

-   Một dịch vụ trải qua suy giảm hiệu quả, và cần nhiều tài nguyên hơn dự kiến để phục vụ cùng nhu cầu.
-   Tỷ lệ chấp nhận của khách hàng tăng, dẫn đến nhu cầu dự kiến tăng.
-   Ngày giao hàng cho một cluster tài nguyên tính toán mới bị trượt.
-   Một quyết định sản phẩm về một mục tiêu hiệu năng thay đổi hình dạng của triển khai dịch vụ cần thiết (dấu chân của dịch vụ) và lượng tài nguyên được yêu cầu.

Các thay đổi nhỏ đòi hỏi đối chiếu toàn bộ kế hoạch cấp phát để đảm bảo kế hoạch vẫn khả thi; các thay đổi lớn hơn (như chậm trễ giao tài nguyên hoặc thay đổi chiến lược sản phẩm) có thể đòi hỏi tạo lại kế hoạch từ đầu. Một sự chậm trễ giao hàng trong một cluster đơn lẻ có thể ảnh hưởng đến các yêu cầu dự phòng (redundancy) hoặc độ trễ của nhiều dịch vụ: cấp phát tài nguyên trong các cluster khác phải được tăng lên để bù đắp, và những thay đổi này sẽ phải lan truyền xuyên suốt kế hoạch.

Ngoài ra, hãy cân nhắc rằng kế hoạch năng lực cho bất kỳ quý nào (hoặc khung thời gian khác) dựa trên kết quả dự kiến của kế hoạch năng lực các quý trước, nghĩa là một thay đổi trong bất kỳ quý nào dẫn đến công việc cập nhật các quý tiếp theo.

### Cồng kềnh và không chính xác (Laborious and imprecise)

Đối với nhiều đội, quy trình thu thập dữ liệu cần thiết để tạo ra các dự báo nhu cầu là chậm và dễ sai. Và khi đến lúc tìm năng lực đáp ứng nhu cầu tương lai, không phải tài nguyên nào cũng phù hợp như nhau. Ví dụ, nếu yêu cầu độ trễ nghĩa là dịch vụ phải cam kết phục vụ nhu cầu người dùng trên cùng một châu lục với người dùng, việc thu thêm tài nguyên ở Bắc Mỹ sẽ không giải quyết thiếu hụt năng lực ở châu Á. Mỗi dự báo có các *ràng buộc* (constraints), tức các tham số về cách nó có thể được đáp ứng; các ràng buộc về bản chất liên quan đến ý định (intent), thảo luận ở phần tiếp theo.

Việc ánh xạ các yêu cầu tài nguyên bị ràng buộc thành các cấp phát tài nguyên thực từ năng lực khả dụng cũng chậm chạp không kém: việc bin pack (đóng gói) các yêu cầu vào không gian hạn chế bằng tay, hoặc tìm giải pháp khớp với ngân sách hạn chế, vừa phức tạp vừa tẻ nhạt.

Quy trình này có thể đã vẽ một bức tranh ảm đạm, nhưng tệ hơn, các công cụ mà nó đòi hỏi thường không đáng tin cậy hoặc cồng kềnh. Các bảng tính (spreadsheets) bị ảnh hưởng nghiêm trọng bởi vấn đề scale và có khả năng kiểm tra lỗi hạn chế. Dữ liệu trở nên cũ (stale), và việc theo dõi các thay đổi trở nên khó khăn. Các đội thường bị buộc phải đưa ra các giả định đơn giản hóa và giảm độ phức tạp của yêu cầu, đơn giản để làm cho việc duy trì năng lực đầy đủ trở thành một vấn đề có thể xử lý được.

Khi chủ sở hữu dịch vụ đối mặt với thách thức khớp một chuỗi yêu cầu năng lực từ các dịch vụ khác nhau vào các tài nguyên khả dụng, theo cách đáp ứng các ràng buộc khác nhau mà một dịch vụ có thể có, sự không chính xác lại tiếp diễn. Bin packing là một vấn đề NP-hard khó cho con người tính bằng tay. Hơn nữa, yêu cầu năng lực từ một dịch vụ nhìn chung là một tập yêu cầu nhu cầu không linh hoạt: *X* cores trong cluster *Y*. Lý do cần *X* cores hay *Y* cluster, và bất kỳ mức độ tự do nào xoay quanh các tham số đó, đã mất từ lâu khi yêu cầu đến tay một con người đang cố khớp danh sách nhu cầu vào nguồn cung khả dụng.

Kết quả ròng là một sự chi tiêu khổng lồ nỗ lực con người để đạt được một bin packing, tốt nhất cũng chỉ là xấp xỉ. Quy trình vỡ vụn trước thay đổi, và không có giới hạn đã biết cho một giải pháp tối ưu.

## Giải pháp của Chúng tôi: Lập kế hoạch Năng lực Dựa trên Ý định (Our Solution: Intent-Based Capacity Planning)

*Hãy chỉ định các yêu cầu, chứ không phải cài đặt.*

Tại Google, nhiều đội đã chuyển sang một cách tiếp cận mà chúng tôi gọi là *Lập kế hoạch Năng lực Dựa trên Ý định* (Intent-based Capacity Planning). Tiền đề cơ bản của cách tiếp cận này là mã hóa một cách lập trình các phụ thuộc và tham số (*ý định*) của nhu cầu dịch vụ, và dùng mã hóa đó để tự động tạo ra một kế hoạch cấp phát chi tiết tài nguyên nào đi đến dịch vụ nào, trong cluster nào. Nếu nhu cầu, nguồn cung, hoặc yêu cầu dịch vụ thay đổi, chúng tôi có thể đơn giản tự động tạo một kế hoạch mới đáp lại các tham số đã thay đổi — giờ đây là sự phân phối tài nguyên tốt nhất mới.

Với các yêu cầu thực và sự linh hoạt của một dịch vụ được nắm bắt, kế hoạch năng lực giờ đây linh hoạt hơn đáng kể trước thay đổi, và chúng tôi có thể đạt được một giải pháp tối ưu đáp ứng nhiều tham số nhất có thể. Việc ủy thác bin packing cho máy tính giúp giảm đáng kể toil con người, và chủ sở hữu dịch vụ có thể tập trung vào các ưu tiên bậc cao như SLO, phụ thuộc production, và yêu cầu hạ tầng dịch vụ, thay vì lục lọi tài nguyên cấp thấp.

Như một lợi ích bổ sung, việc dùng tối ưu hóa tính toán để ánh xạ từ ý định đến cài đặt đạt được độ chính xác cao hơn rất nhiều, cuối cùng dẫn đến tiết kiệm chi phí cho tổ chức. Bin packing vẫn còn xa mới là một vấn đề đã giải quyết, vì một số loại vẫn được coi là NP-hard; tuy nhiên, các thuật toán ngày nay có thể giải đến một giải pháp tối ưu đã biết.

## Lập kế hoạch Năng lực Dựa trên Ý định (Intent-Based Capacity Planning)

*Ý định* (intent) là lý lẽ cho cách một chủ sở hữu dịch vụ muốn vận hành dịch vụ. Việc chuyển từ các yêu cầu tài nguyên cụ thể đến các lý do thúc đẩy, để đạt đến ý định lập kế hoạch năng lực thực, thường đòi hỏi một số lớp trừu tượng (abstraction). Hãy cân nhắc chuỗi trừu tượng sau:

1) "Tôi muốn 50 cores trong các cluster X, Y, và Z cho dịch vụ Foo."

Đây là một yêu cầu tài nguyên rõ ràng. Nhưng…*tại sao chúng tôi cần nhiều tài nguyên như vậy cụ thể trong những cluster này?*

2) "Tôi muốn một dấu chân 50-core trong bất kỳ 3 cluster nào trong vùng địa lý YYY cho dịch vụ Foo."

Yêu cầu này giới thiệu nhiều mức độ tự do hơn và dễ đáp ứng hơn, dù không giải thích lý lẽ đằng sau các yêu cầu của nó. Nhưng…*tại sao chúng tôi cần lượng tài nguyên này, và tại sao 3 dấu chân?*

3) "Tôi muốn đáp ứng nhu cầu của dịch vụ Foo trong mỗi vùng địa lý, và có dự phòng N + 2."

Đột nhiên sự linh hoạt lớn hơn xuất hiện, và chúng tôi có thể hiểu ở cấp độ "con người" hơn điều gì xảy ra nếu dịch vụ Foo không nhận được những tài nguyên này. Nhưng…*tại sao chúng tôi cần N + 2 cho dịch vụ Foo?*

4) "Tôi muốn chạy dịch vụ Foo ở 5 nines độ tin cậy."

Đây là một yêu cầu trừu tượng hơn, và hệ quả nếu yêu cầu không được đáp ứng trở nên rõ ràng: độ tin cậy sẽ bị ảnh hưởng. Và chúng tôi có sự linh hoạt lớn hơn ở đây: có thể chạy ở *N* + 2 thực sự không đủ hoặc không tối ưu cho dịch vụ này, và một kế hoạch triển khai khác sẽ phù hợp hơn.

Vậy cấp độ ý định nào nên được lập kế hoạch năng lực dựa trên ý định sử dụng? Lý tưởng, tất cả các cấp độ ý định nên được hỗ trợ cùng nhau, và dịch vụ hưởng lợi càng nhiều khi chuyển sang chỉ định ý định thay vì cài đặt. Trong kinh nghiệm của Google, các dịch vụ có xu hướng đạt được những chiến thắng tốt nhất khi bước sang bước 3: mức linh hoạt tốt khả dụng, và các hệ quả của yêu cầu được diễn đạt bằng thuật ngữ cấp cao hơn, dễ hiểu hơn. Các dịch vụ đặc biệt tinh vi có thể nhắm đến bước 4.

## Các Tiền thân của Ý định (Precursors to Intent)

Chúng tôi cần thông tin gì để nắm bắt ý định của một dịch vụ? Các phụ thuộc, metrics hiệu năng, và ưu tiên hóa (prioritization) sẽ vào cuộc.

### Các Sự phụ thuộc (Dependencies)

Các dịch vụ tại Google phụ thuộc vào nhiều dịch vụ hạ tầng và hướng người dùng khác, và các phụ thuộc này ảnh hưởng mạnh mẽ đến nơi một dịch vụ có thể được đặt. Ví dụ, hãy hình dung dịch vụ hướng người dùng Foo phụ thuộc vào Bar, một dịch vụ lưu trữ hạ tầng. Foo biểu đạt một yêu cầu rằng Bar phải được đặt trong vòng 30 mili giây độ trễ mạng của Foo. Yêu cầu này có những hệ quả quan trọng cho nơi đặt cả Foo *và* Bar, và lập kế hoạch năng lực dựa trên ý định phải tính đến những ràng buộc này.

Hơn nữa, các phụ thuộc production được lồng nhau (nested): tiếp tục ví dụ trước, hãy hình dung dịch vụ Bar có các phụ thuộc riêng vào Baz, một dịch vụ lưu trữ phân tán cấp thấp hơn, và Qux, một dịch vụ quản lý ứng dụng. Vì vậy, nơi bây giờ có thể đặt Foo phụ thuộc vào nơi có thể đặt Bar, Baz, và Qux. Một tập phụ thuộc production nhất định có thể được chia sẻ, có thể với các điều khoản khác nhau xoay quanh ý định.

### Các Metrics Hiệu năng (Performance metrics)

Nhu cầu của một dịch vụ lan truyền để dẫn đến nhu cầu của một hoặc nhiều dịch vụ khác. Việc hiểu chuỗi các phụ thuộc giúp định hình phạm vi chung của vấn đề bin packing, nhưng chúng tôi vẫn cần thêm thông tin về mức sử dụng tài nguyên dự kiến. Bao nhiêu tài nguyên tính toán mà dịch vụ Foo cần để phục vụ *N* truy vấn người dùng? Với mỗi *N* truy vấn của dịch vụ Foo, chúng tôi kỳ vọng bao nhiêu Mbps dữ liệu cho dịch vụ Bar?

Các metrics hiệu năng là chất kết dính giữa các phụ thuộc. Chúng chuyển đổi từ một hoặc nhiều loại tài nguyên cấp cao hơn sang một hoặc nhiều loại tài nguyên cấp thấp hơn. Việc suy ra các metrics hiệu năng phù hợp cho một dịch vụ có thể bao gồm kiểm thử tải (load testing) và giám sát mức sử dụng tài nguyên.

### Ưu tiên hóa (Prioritization)

Không thể tránh được, các ràng buộc tài nguyên dẫn đến những đánh đổi và quyết định khó khăn: trong số nhiều yêu cầu của các dịch vụ, yêu cầu nào nên được hy sinh trước sự thiếu hụt năng lực?

Có thể dự phòng *N* + 2 cho dịch vụ Foo quan trọng hơn dự phòng *N* + 1 cho dịch vụ Bar. Hoặc ra mắt tính năng *X* có thể ít quan trọng hơn dự phòng *N* + 0 cho dịch vụ Baz.

Lập kế hoạch dựa trên ý định buộc những quyết định này được đưa ra minh bạch, cởi mở, và nhất quán. Các ràng buộc tài nguyên luôn liên quan đến những đánh đổi như nhau, nhưng quá thường, việc ưu tiên hóa lại ad hoc (tùy hứng) và mờ mịt đối với chủ sở hữu dịch vụ. Lập kế hoạch dựa trên ý định cho phép ưu tiên hóa tinh tế hoặc thô tùy mức cần thiết.

## Giới thiệu về Auxon (Introduction to Auxon)

Auxon là phiên bản của Google cho một giải pháp lập kế hoạch năng lực và cấp phát tài nguyên dựa trên ý định, và là một ví dụ điển hình của một sản phẩm kỹ thuật phần mềm được thiết kế và phát triển bởi SRE: nó được xây dựng bởi một nhóm nhỏ kỹ sư phần mềm và một technical program manager (quản lý chương trình kỹ thuật) bên trong SRE trong suốt hai năm. Auxon là một nghiên cứu tình huống hoàn hảo để minh họa cách phát triển phần mềm có thể được nuôi dưỡng trong SRE.

Auxon được dùng tích cực để lập kế hoạch cho hàng triệu đô la tài nguyên máy tại Google. Nó đã trở thành một thành phần quan trọng của lập kế hoạch năng lực cho một số division (phân khu) lớn trong Google.

Với tư cách một sản phẩm, Auxon cung cấp các phương tiện để thu thập mô tả dựa trên ý định của các yêu cầu tài nguyên và phụ thuộc của một dịch vụ. Các ý định người dùng này được biểu đạt như những yêu cầu về cách chủ sở hữu muốn dịch vụ được provision (cấp phát). Các yêu cầu có thể được chỉ định như, "Dịch vụ của tôi phải là *N* + 2 mỗi châu lục" hoặc "Các server frontend (mặt trước) phải cách các server backend (phía sau) không quá 50 ms." Auxon thu thập thông tin này qua một ngôn ngữ cấu hình cho người dùng hoặc qua một API (Application Programming Interface — Giao diện Lập trình Ứng dụng) lập trình, qua đó chuyển đổi ý định con người thành các ràng buộc mà máy có thể phân tích. Các yêu cầu có thể được ưu tiên hóa, một tính năng hữu ích khi tài nguyên không đủ để đáp ứng tất cả các yêu cầu, và do đó phải thực hiện các đánh đổi. Những yêu cầu này — ý định — cuối cùng được biểu diễn bên trong như một chương trình nguyên hỗn hợp (mixed-integer) hoặc tuyến tính (linear program) khổng lồ. Auxon giải quyết chương trình tuyến tính, và dùng giải pháp bin packing thu được để định hình một kế hoạch cấp phát cho các tài nguyên.

[Hình 18-1](#hinh-18-1) và các giải thích tiếp theo phác thảo các thành phần chính của Auxon.


<a id="hinh-18-1"></a>![Hình 18-1](../assets/imgs/fig-18-1.jpg)

[Hình 18-1.](#hinh-18-1) Các thành phần chính của Auxon.

*Dữ liệu Hiệu năng* (Performance Data) mô tả cách một dịch vụ scale: với mỗi đơn vị nhu cầu *X* trong cluster *Y*, bao nhiêu đơn vị phụ thuộc *Z* được sử dụng? Dữ liệu scale này có thể được suy ra theo một số cách tùy thuộc vào độ trưởng thành của dịch vụ liên quan. Một số dịch vụ được kiểm thử tải, trong khi những dịch vụ khác suy ra mức scale của chúng dựa trên hiệu năng trong quá khứ.

*Dữ liệu Dự báo Nhu cầu Mỗi Dịch vụ* (Per-Service Demand Forecast Data) mô tả xu hướng sử dụng cho các tín hiệu nhu cầu được dự báo. Một số dịch vụ suy ra mức sử dụng tương lai của chúng từ các dự báo nhu cầu — một dự báo các truy vấn mỗi giây được chia theo châu lục. Không phải dịch vụ nào cũng có một dự báo nhu cầu: một số dịch vụ (ví dụ, một dịch vụ lưu trữ như Colossus) suy ra nhu cầu của chúng thuần túy từ các dịch vụ phụ thuộc vào chúng.

*Nguồn cung Tài nguyên* (Resource Supply) cung cấp dữ liệu về khả dụng của các tài nguyên cơ bản, nền tảng: ví dụ, số máy dự kiến khả dụng tại một thời điểm tương lai nhất định. Trong thuật ngữ chương trình tuyến tính, nguồn cung tài nguyên đóng vai trò như một *giới hạn trên* (upper bound) giới hạn cách các dịch vụ có thể phát triển và nơi chúng có thể được đặt. Cuối cùng, chúng tôi muốn sử dụng tốt nhất nguồn cung tài nguyên này theo cách mà mô tả dựa trên ý định của các nhóm dịch vụ cho phép.

*Định giá Tài nguyên* (Resource Pricing) cung cấp dữ liệu về việc các tài nguyên cơ bản, nền tảng tốn bao nhiêu. Ví dụ, chi phí của máy có thể khác nhau trên toàn cầu dựa trên các phí không gian/điện của một cơ sở nhất định. Trong thuật ngữ chương trình tuyến tính, các giá trị này phản ánh chi phí tính toán tổng thể, và đóng vai trò là *mục tiêu* (objective) mà chúng tôi muốn tối thiểu hóa.

*Cấu hình Ý định* (Intent Config) là chìa khóa cho cách thông tin dựa trên ý định được đưa vào Auxon. Nó định nghĩa điều gì cấu thành một dịch vụ, và cách các dịch vụ liên quan đến nhau. Config đóng vai trò như một tầng cấu hình cho phép tất cả các thành phần khác được nối với nhau. Nó được thiết kế để con người đọc được và cấu hình được.

*Engine Ngôn ngữ Cấu hình Auxon* (Auxon Configuration Language Engine) hoạt động dựa trên thông tin nó nhận từ Intent Config. Thành phần này định hình một yêu cầu mà máy có thể đọc: một protocol buffer (bộ đệm giao thức) có thể được hiểu bởi Auxon Solver (Bộ giải). Nó áp dụng một số kiểm tra hợp lý (sanity checking) nhẹ đối với cấu hình, và được thiết kế như một cổng giữa định nghĩa ý định do con người cấu hình và yêu cầu tối ưu hóa mà máy có thể phân tích.

*Auxon Solver* là bộ não của công cụ. Nó định hình chương trình nguyên hỗn hợp hoặc tuyến tính khổng lồ dựa trên yêu cầu tối ưu hóa nhận từ Configuration Language Engine. Nó được thiết kế để scale mạnh, cho phép bộ giải chạy song song trên hàng trăm hoặc thậm chí hàng nghìn máy trong các cluster của Google. Ngoài các công cụ lập trình tuyến tính nguyên hỗn hợp, còn có các thành phần trong Auxon Solver xử lý các tác vụ như lên lịch, quản lý một bể (pool) các worker (máy công nhân), và đi xuống các cây quyết định (decision trees).

*Kế hoạch Cấp phát* (Allocation Plan) là đầu ra của Auxon Solver. Nó quy định tài nguyên nào nên được cấp phát cho dịch vụ nào ở vị trí nào. Nó là các chi tiết cài đặt được tính toán từ định nghĩa dựa trên ý định của các yêu cầu trong vấn đề lập kế hoạch năng lực. Kế hoạch Cấp phát cũng bao gồm thông tin về bất kỳ yêu cầu nào không thể được đáp ứng — ví dụ, khi một yêu cầu không thể đáp ứng do thiếu tài nguyên, hoặc các yêu cầu cạnh tranh quá nghiêm khắc.

## Các Yêu cầu và Cài đặt: Những Thành công và Bài học Học được (Requirements and Implementation: Successes and Lessons Learned)

Auxon lần đầu tiên được hình dung bởi một SRE và một technical program manager, người đã riêng biệt được các đội tương ứng của họ giao nhiệm vụ lập kế hoạch năng lực cho các phần lớn hạ tầng của Google. Từng thực hiện lập kế hoạch năng lực thủ công trong các bảng tính, họ ở vị trí tốt để hiểu những điểm kém hiệu quả và các cơ hội cải thiện thông qua tự động hóa, cũng như các tính năng mà một công cụ như vậy có thể đòi hỏi.

Trong suốt quá trình phát triển Auxon, đội SRE đằng sau sản phẩm tiếp tục gắn chặt vào thế giới production. Đội duy trì một vai trò trong các vòng on-call cho một số dịch vụ của Google, và tham gia vào các cuộc thảo luận thiết kế cũng như lãnh đạo kỹ thuật của những dịch vụ này. Qua những tương tác liên tục, đội giữ được cảm giác chân thực với production: họ vừa là người dùng vừa là người phát triển của chính sản phẩm. Khi sản phẩm thất bại, đội bị ảnh hưởng trực tiếp. Các yêu cầu tính năng được thúc đẩy bởi chính trải nghiệm thực tế của đội. Kinh nghiệm trực tiếp với không gian vấn đề không chỉ mang lại cảm giác sở hữu lớn đối với sự thành công của sản phẩm, mà còn giúp tăng cường độ tin cậy và tính chính đáng của sản phẩm trong SRE.

### Xấp xỉ (Approximation)

Đừng bám chấp vào sự hoàn hảo hay tính thuần túy của giải pháp, đặc biệt khi các giới hạn của vấn đề chưa rõ. Hãy khởi động và lặp lại.

Bất kỳ nỗ lực kỹ thuật phần mềm đủ phức tạp nào cũng chắc chắn sẽ gặp sự không chắc chắn về cách thiết kế một thành phần hoặc cách tiếp cận một vấn đề. Auxon gặp phải điều này sớm trong quá trình phát triển, vì thế giới lập trình tuyến tính là lãnh thổ chưa được khai phá đối với các thành viên đội. Các giới hạn của lập trình tuyến tính — dường như là một phần trung tâm của cách sản phẩm nhiều khả năng sẽ hoạt động — không được hiểu rõ. Để vượt qua sự lúng túng này, chúng tôi chọn ban đầu xây dựng một engine bộ giải đơn giản hóa (còn gọi là "Stupid Solver" — Bộ giải ngốc) áp dụng một số heuristics (thuật toán gần đúng) đơn giản về cách sắp xếp các dịch vụ dựa trên yêu cầu do người dùng chỉ định. Trong khi Stupid Solver sẽ không bao giờ tạo ra một giải pháp thực sự tối ưu, nó cho đội cảm giác rằng tầm nhìn của chúng tôi cho Auxon là có thể đạt được, ngay cả khi chúng tôi không xây dựng một thứ hoàn hảo từ ngày đầu tiên.

Khi triển khai xấp xỉ để tăng tốc phát triển, quan trọng là thực hiện công việc theo cách cho phép đội nâng cấp trong tương lai và xem xét lại việc xấp xỉ. Trong trường hợp của Stupid Solver, toàn bộ giao diện bộ giải được trừu tượng hóa trong Auxon để các phần tử bên trong bộ giải có thể được hoán đổi sau này. Cuối cùng, khi chúng tôi xây dựng được sự tin tưởng vào một mô hình lập trình tuyến tính thống nhất, việc hoán đổi Stupid Solver bằng một thứ, chà, thông minh hơn chỉ là một thao tác đơn giản.

Các yêu cầu sản phẩm của Auxon cũng có một số điều không xác định. Xây dựng phần mềm với các yêu cầu mơ hồ có thể là một thách thức gây thất vọng, nhưng một mức độ không chắc chắn nhất định không cần phải là một điểm dừng. Hãy dùng sự mơ hồ đó như động lực để đảm bảo phần mềm được thiết kế vừa tổng quát vừa có tính module (mô-đun). Ví dụ, một trong những mục tiêu của dự án Auxon là tích hợp với các hệ thống tự động hóa bên trong Google để một Kế hoạch Cấp phát có thể được thực thi trực tiếp trên production (gán tài nguyên và khởi động/tắt/điều chỉnh kích thước các dịch vụ tương ứng). Tuy nhiên, vào thời điểm đó, thế giới các hệ thống tự động hóa đang biến động lớn, với vô số cách tiếp cận đang được sử dụng. Thay vì thiết kế các giải pháp riêng biệt để Auxon hoạt động với từng công cụ, chúng tôi tạo hình Kế hoạch Cấp phát để đủ phổ quát, giúp những hệ thống tự động hóa này tích hợp ở các điểm riêng của chúng. Cách tiếp cận "agnostic" (trung lập công nghệ) này trở thành chìa khóa cho quy trình onboard (tiếp nhận) khách hàng mới của Auxon, vì nó cho phép khách hàng bắt đầu sử dụng mà không cần cam kết với một công cụ tự động hóa, công cụ dự báo, hay công cụ dữ liệu hiệu năng cụ thể.

Chúng tôi cũng tận dụng thiết kế có tính module để đối phó với các yêu cầu mơ hồ khi xây dựng một mô hình hiệu năng máy trong Auxon. Dữ liệu về hiệu năng nền tảng máy trong tương lai (ví dụ, CPU) rất khan hiếm, nhưng người dùng muốn một cách để mô hình hóa các kịch bản khác nhau của sức mạnh máy. Chúng tôi trừu tượng hóa dữ liệu máy phía sau một giao diện đơn lẻ, cho phép người dùng hoán đổi các mô hình hiệu năng máy trong tương lai khác nhau. Sau đó, dựa trên các yêu cầu được định nghĩa ngày càng nhiều, chúng tôi mở rộng tính module này để cung cấp một thư viện các mô hình hiệu năng máy đơn giản hoạt động trong giao diện đó.

Nếu có một chủ đề để rút ra từ nghiên cứu tình huống Auxon, đó là khẩu hiệu cũ "[khởi động và lặp lại](https://sre.google/sre-book/reliable-product-launches/)" đặc biệt liên quan trong các dự án phát triển phần mềm SRE. Đừng chờ thiết kế hoàn hảo; thay vào đó, giữ tầm nhìn tổng thể trong tâm trí trong khi tiến lên với thiết kế và phát triển. Khi gặp các vùng không chắc chắn, hãy thiết kế phần mềm đủ linh hoạt để nếu quy trình hay chiến lược thay đổi ở cấp cao hơn, bạn không phải chịu một chi phí làm lại khổng lồ. Nhưng đồng thời, giữ chân mình bằng cách đảm bảo các giải pháp tổng quát có một cài đặt cụ thể trong thế giới thực minh họa tính hữu dụng của thiết kế.

## Nâng cao Nhận thức và Thúc đẩy Chấp nhận (Raising Awareness and Driving Adoption)

Giống như bất kỳ sản phẩm nào, phần mềm do SRE phát triển phải được thiết kế dựa trên hiểu biết về người dùng và yêu cầu của họ. Nó cần thúc đẩy việc chấp nhận thông qua tính hữu dụng, hiệu năng, và khả năng chứng minh được vừa phục vụ các mục tiêu độ tin cậy production của Google vừa cải thiện cuộc sống của các SRE. Việc xã hội hóa một sản phẩm và đạt được sự cam kết (buy-in) xuyên suốt một tổ chức là chìa khóa cho thành công của dự án.

Đừng đánh giá thấp nỗ lực cần thiết để nâng cao nhận thức và sự quan tâm đối với sản phẩm phần mềm của bạn — một bài thuyết trình hay một email thông báo đơn lẻ là không đủ. Việc xã hội hóa các công cụ phần mềm nội bộ đến một khán giả lớn đòi hỏi tất cả những điều sau:

-   Một cách tiếp cận nhất quán và mạch lạc
-   Sự ủng hộ của người dùng
-   Sự bảo trợ của các kỹ sư cấp cao và ban quản lý, những người bạn sẽ phải thuyết phục bằng tính hữu dụng của sản phẩm

Quan trọng là phải xem xét quan điểm của khách hàng khi làm cho sản phẩm dễ sử dụng. Một kỹ sư có thể không có thời gian hay xu hướng để đào sâu vào code nguồn để tìm ra cách dùng một công cụ. Dù các khách hàng nội bộ nhìn chung bao dung hơn với các cạnh thô và các bản alpha (thử nghiệm) sớm so với khách hàng bên ngoài, việc cung cấp tài liệu vẫn cần thiết. Các SRE bận rộn, và nếu giải pháp của bạn quá khó hay gây nhầm lẫn, họ sẽ tự viết giải pháp của riêng họ.

### Đặt kỳ vọng (Set expectations)

Khi một kỹ sư có nhiều năm kinh nghiệm trong một không gian vấn đề bắt đầu thiết kế một sản phẩm, dễ dàng hình dung một trạng thái cuối cùng lý tưởng cho công việc. Tuy nhiên, quan trọng là phải phân biệt các mục tiêu mong muốn của sản phẩm với các tiêu chí thành công tối thiểu (hay Sản phẩm Tối thiểu Khả thi — Minimum Viable Product). Dự án có thể mất uy tín và thất bại khi hứa hẹn quá nhiều, quá sớm; nhưng cùng lúc, nếu một sản phẩm không hứa hẹn một kết quả đủ mang lại lợi ích, có thể khó vượt qua năng lượng kích hoạt cần thiết để thuyết phục các đội nội bộ thử một thứ mới. Minh họa sự tiến bộ ổn định, tăng dần qua các release nhỏ giúp củng cố niềm tin của người dùng vào khả năng cung cấp phần mềm hữu ích của đội bạn.

Trong trường hợp của Auxon, chúng tôi đạt được sự cân bằng bằng cách lập một bản đồ đường dài hạn cùng với các sửa chữa ngắn hạn. Các đội được hứa rằng:

-   Bất kỳ nỗ lực onboard và cấu hình nào đều mang lại lợi ích ngay lập tức là giảm sự đau đớn của việc bin pack thủ công các yêu cầu tài nguyên ngắn hạn.
-   Khi các tính năng bổ sung được phát triển cho Auxon, các tệp cấu hình sẽ được mang sang và mang lại những tiết kiệm chi phí dài hạn mới, rộng lớn hơn, cùng các lợi ích khác. Bản đồ đường dự án cho phép các dịch vụ nhanh chóng xác định liệu use case hay tính năng mà chúng yêu cầu có được cài đặt trong các phiên bản ban đầu không. Trong khi đó, cách tiếp cận phát triển lặp đi lặp lại của Auxon liên tục được đưa vào các ưu tiên phát triển và các mốc mới của bản đồ đường.

### Xác định các Khách hàng Phù hợp (Identify appropriate customers)

Đội phát triển Auxon nhận ra rằng một giải pháp kích thước đơn (one-size) có thể không phù hợp với tất cả; nhiều đội lớn hơn đã có các giải pháp tự phát triển cho lập kế hoạch năng lực hoạt động khá tốt. Dù các công cụ tùy chỉnh của họ không hoàn hảo, những đội này không trải qua đủ đau đớn trong quy trình lập kế hoạch năng lực để thử một công cụ mới, đặc biệt là một release alpha còn nhiều cạnh thô.

Các phiên bản ban đầu của Auxon cố ý nhắm đến các đội không có quy trình lập kế hoạch năng lực hiện có. Những đội này sẽ phải đầu tư nỗ lực cấu hình bất kể họ chọn một công cụ hiện có hay cách tiếp cận mới của chúng tôi, nên họ quan tâm đến việc chấp nhận công cụ mới nhất. Những thành công ban đầu mà Auxon đạt được với những đội đó minh họa tính hữu dụng của dự án, và biến chính những khách hàng đó thành những người ủng hộ công cụ. Việc định lượng tính hữu dụng của sản phẩm chứng tỏ là một lợi ích liên tục; khi onboard một trong các Business Area (Khu vực Kinh doanh) của Google, đội đã tạo ra một nghiên cứu tình huống chi tiết về quy trình và so sánh kết quả trước và sau. Chỉ riêng việc tiết kiệm thời gian và giảm toil con người đã là một động lực lớn để các đội khác thử Auxon.

### Dịch vụ Khách hàng (Customer service)

Ngay cả khi phần mềm phát triển trong SRE nhắm đến một khán giả gồm các TPM (Technical Program Manager — Quản lý Chương trình Kỹ thuật) và kỹ sư có khả năng kỹ thuật cao, bất kỳ phần mềm đủ đổi mới nào vẫn có một đường cong học tập cho người dùng mới. Đừng ngại cung cấp hỗ trợ khách hàng găng tay trắng (white glove) cho những người chấp nhận sớm để giúp họ vượt qua quy trình onboard. Đôi khi tự động hóa còn đi kèm một loạt mối lo ngại cảm xúc, như nỗi sợ rằng công việc của ai đó sẽ bị thay thế bởi một shell script (lệnh vỏ). Bằng cách làm việc một-một với các người dùng sớm, bạn có thể giải quyết những nỗi sợ đó một cách cá nhân, và chỉ ra rằng thay vì trực tiếp làm một tác vụ tẻ nhạt, đội giờ đây sở hữu các cấu hình, quy trình, và kết quả cuối cùng của công việc kỹ thuật của họ. Những người chấp nhận muộn hơn sẽ được thuyết phục bởi những ví dụ thành công của những người chấp nhận sớm.

Hơn nữa, vì các đội SRE của Google phân bố trên toàn cầu, những người ủng hộ chấp nhận sớm cho một dự án đặc biệt có lợi, vì họ có thể đóng vai trò như các chuyên gia địa phương cho những đội khác quan tâm đến việc thử dự án.

### Thiết kế ở Cấp độ Đúng (Designing at the right level)

Một ý tưởng mà chúng tôi gọi là *trung lập công nghệ* (agnosticism) — viết phần mềm đủ tổng quát để chấp nhận vô số nguồn dữ liệu làm input (đầu vào) — là một nguyên lý chính của thiết kế Auxon. Trung lập công nghệ có nghĩa là khách hàng không bị yêu cầu cam kết với bất kỳ công cụ nào để sử dụng khung Auxon. Cách tiếp cận này giúp Auxon duy trì đủ hữu dụng và tổng quát ngay cả khi các đội với use case khác nhau bắt đầu sử dụng nó. Chúng tôi tiếp cận những người dùng tiềm năng với thông điệp, "đến theo cách bạn là; chúng tôi sẽ làm việc với những gì bạn có." Bằng cách tránh tùy chỉnh quá mức cho một hoặc hai người dùng lớn, chúng tôi đạt được sự chấp nhận rộng hơn xuyên suốt tổ chức và giảm rào cản gia nhập cho các dịch vụ mới.

Chúng tôi cũng có ý thức tránh cạm bẫy của việc định nghĩa thành công là 100% sự chấp nhận xuyên suốt tổ chức. Trong nhiều trường hợp, lợi ích giảm dần khi khép lại những dặm cuối cùng nhằm đưa tập tính năng đủ cho mọi dịch vụ trong phần đuôi dài (long tail) tại Google.

## Động lực Đội (Team Dynamics)

Khi chọn các kỹ sư làm việc trên một sản phẩm phát triển phần mềm SRE, chúng tôi thấy lợi ích lớn từ việc tạo ra một đội hạt giống (seed team) kết hợp các người đa năng (generalists) có thể bắt kịp nhanh chóng một chủ đề mới với các kỹ sư có phạm vi rộng kiến thức và kinh nghiệm. Sự đa dạng kinh nghiệm che phủ các điểm mù cũng như cạm bẫy của việc giả định rằng use case của mọi đội giống như của bạn.

Điều thiết yếu là đội bạn thiết lập mối quan hệ làm việc với các chuyên gia cần thiết, và cho các kỹ sư của bạn thoải mái làm việc trong một không gian vấn đề mới. Đối với các đội SRE tại phần lớn các công ty, việc dấn vào không gian vấn đề mới này đòi hỏi thuê ngoài các tác vụ hoặc làm việc với các nhà tư vấn, nhưng các đội SRE tại các tổ chức lớn hơn có thể hợp tác với các chuyên gia nội bộ. Trong các giai đoạn đầu của việc hình dung và thiết kế Auxon, chúng tôi đã trình bày tài liệu thiết kế của mình đến các đội nội bộ của Google chuyên về Operations Research (Nghiên cứu Vận hành) và Quantitative Analysis (Phân tích Định lượng) để tận dụng chuyên môn của họ và khởi tạo kiến thức của đội Auxon về lập kế hoạch năng lực.

Khi phát triển dự án tiếp tục và tập tính năng của Auxon trở nên rộng và phức tạp hơn, đội có thêm các thành viên có nền tảng trong thống kê và tối ưu hóa toán học, điều mà ở một công ty nhỏ hơn có thể tương đương với việc đưa một nhà tư vấn bên ngoài vào nội bộ. Những thành viên mới này có thể xác định các khu vực cần cải thiện khi chức năng cơ bản của dự án hoàn thành và việc bổ sung sự tinh tế trở thành ưu tiên hàng đầu.

Thời điểm đúng để tiếp xúc với các chuyên gia, tất nhiên, sẽ khác nhau từ dự án này sang dự án khác. Như một hướng dẫn thô, dự án nên khởi động thành công và chứng minh được thành công, để kỹ năng của đội hiện tại có thể được củng cố đáng kể bởi chuyên môn bổ sung.

## Nuôi dưỡng Kỹ thuật Phần mềm trong SRE (Fostering Software Engineering in SRE)

Điều gì khiến một dự án trở thành ứng cử viên tốt để thực hiện bước nhảy từ công cụ dùng một lần sang một nỗ lực kỹ thuật phần mềm đầy đủ? Các tín hiệu tích cực mạnh bao gồm các kỹ sư có kinh nghiệm trực tiếp trong lĩnh vực liên quan, quan tâm đến việc làm trên dự án, và một cơ sở người dùng mục tiêu rất kỹ thuật (và do đó có thể cung cấp các báo cáo bug chất lượng cao trong các giai đoạn đầu phát triển). Dự án nên mang lại các lợi ích đáng kể, như giảm toil cho các SRE, cải thiện một phần hạ tầng hiện có, hoặc tinh giản một quy trình phức tạp.

Quan trọng là dự án phải khớp với tổng thể bộ mục tiêu của tổ chức, để các lãnh đạo kỹ thuật có thể cân nhắc tác động tiềm tàng của nó và sau đó ủng hộ dự án của bạn, cả với các đội trực thuộc lẫn với các đội khác có thể giao tiếp với các đội của họ. Việc xã hội hóa và xem xét liên tổ chức giúp ngăn các nỗ lực rời rạc hoặc chồng chéo, và một sản phẩm dễ được định vị là thúc đẩy một mục tiêu trên toàn bộ phòng ban sẽ dễ được bố trí nhân sự và hỗ trợ hơn.

Điều gì khiến một dự án thành ứng cử viên kém? Nhiều "cờ đỏ" (red flags) cùng lúc, thứ mà bạn có thể trực giác nhận ra trong bất kỳ dự án phần mềm nào, như phần mềm chạm vào nhiều bộ phận chuyển động cùng một lúc, hoặc thiết kế đòi hỏi cách tiếp cận tất-cả-hoặc-không-gì (all-or-nothing) ngăn phát triển lặp đi lặp lại. Vì các đội SRE Google hiện được tổ chức xoay quanh các dịch vụ mà họ vận hành, các dự án do SRE phát triển đặc biệt có nguy cơ trở thành công việc quá cụ thể, chỉ mang lại lợi ích cho một phần nhỏ tổ chức. Vì động lực của các đội được căn chỉnh chủ yếu để mang lại trải nghiệm tuyệt vời cho người dùng của một dịch vụ cụ thể, các dự án thường thất bại trong việc tổng quát hóa đến một use case rộng hơn khi sự chuẩn hóa xuyên suốt các đội SRE bị xếp xuống sau. Ở đầu kia của phổ, các khung quá tổng quát cũng có thể gây vấn đề tương đương; nếu một công cụ cố gắng quá linh hoạt và quá phổ quát, nó chạy rủi ro không vừa khít với bất kỳ use case nào, và do đó không đủ giá trị trong bản thân nó. Các dự án với phạm vi rộng lớn và mục tiêu trừu tượng thường đòi hỏi nỗ lực phát triển đáng kể, nhưng thiếu các use case cụ thể cần thiết để mang lại lợi ích cho người dùng cuối trong một khung thời gian hợp lý.

Lấy một ví dụ về use case rộng: một load balancer (bộ cân bằng tải) tầng-3 do các SRE Google phát triển đã thành công đến mức, qua nhiều năm, nó được tái sử dụng như một đề xuất sản phẩm hướng khách hàng thông qua Google Cloud Load Balancer [[Eis16]](https://sre.google/sre-book/bibliography#Eis16).

## Thành công Xây dựng một Văn hóa Kỹ thuật Phần mềm trong SRE: Nhân sự và Thời gian Phát triển (Successfully Building a Software Engineering Culture in SRE: Staffing and Development Time)

Các SRE thường là những người đa năng, vì mong muốn học theo chiều rộng hơn là chiều sâu — và điều đó phù hợp tốt cho việc hiểu bức tranh lớn hơn (đặc biệt khi có rất ít bức tranh lớn hơn so với sự tinh vi nội tại của hạ tầng kỹ thuật hiện đại). Những kỹ sư này thường có kỹ năng code và phát triển phần mềm mạnh mẽ, nhưng có thể thiếu kinh nghiệm SWE (Kỹ sư Phần mềm) truyền thống của việc là một phần của một đội sản phẩm hay phải suy nghĩ về các yêu cầu tính năng khách hàng. Một trích dẫn từ một kỹ sư trong một dự án phát triển phần mềm SRE ban đầu tóm tắt cách tiếp cận SRE truyền thống đối với phần mềm: "Tôi có một tài liệu thiết kế; tại sao chúng tôi cần các yêu cầu?" Hợp tác với các kỹ sư, TPM, hoặc PM (Quản lý Sản phẩm) quen thuộc với phát triển phần mềm hướng người dùng có thể giúp xây dựng một văn hóa phát triển phần mềm của đội, kết hợp những điều tốt nhất của cả phát triển sản phẩm phần mềm lẫn kinh nghiệm production trực tiếp.

Thời gian làm việc chuyên tâm, không bị ngắt, là thiết yếu cho bất kỳ nỗ lực phát triển phần mềm nào. Thời gian chuyên tâm như vậy cần thiết để có tiến bộ trên một dự án, vì gần như không thể viết code — chứ đừng nói đến tập trung vào các dự án lớn hơn, có tác động hơn — khi bạn đang nhảy nhót giữa nhiều tác vụ trong suốt một giờ. Vì vậy, khả năng làm việc trên một dự án phần mềm mà không bị ngắt thường là một lý do hấp dẫn để các kỹ sư bắt đầu một dự án phát triển. Những khoảng thời gian như vậy phải được bảo vệ một cách quyết liệt.

Phần lớn các sản phẩm phần mềm phát triển trong SRE bắt đầu như các dự án phụ, mà tính hữu dụng của chúng dẫn đến việc chúng phát triển và được chính thức hóa. Tại thời điểm này, một sản phẩm có thể rẽ sang một trong số một vài hướng khả thi:

-   Tiếp tục là một nỗ lực cơ sở được phát triển trong thời gian rảnh của các kỹ sư
-   Được thiết lập như một dự án chính thức thông qua các quy trình có cấu trúc (xem [Đi đến đó](#di-den-do))
-   Đạt được sự bảo trợ điều hành từ bên trong lãnh đạo SRE để mở rộng thành một nỗ lực phát triển phần mềm được bố trí nhân sự đầy đủ

Tuy nhiên, trong bất kỳ kịch bản nào trong số này — và đây là một điểm đáng nhấn mạnh — các SRE tham gia vào nỗ lực phát triển phải tiếp tục làm việc như các SRE, thay vì trở thành các developer toàn thời gian gắn vào tổ chức SRE. Việc luôn đứng trong thế giới production mang lại cho các SRE làm công việc phát triển một góc nhìn vô giá, vì họ vừa là người sáng tạo vừa là khách hàng của bất kỳ sản phẩm nào.

<a id="di-den-do"></a>

## Đi đến đó (Getting There)

Nếu bạn thích ý tưởng về phát triển phần mềm có tổ chức trong SRE, bạn có thể đang tự hỏi làm thế nào để đưa một mô hình phát triển phần mềm vào một tổ chức SRE vốn tập trung vào hỗ trợ production.

Đầu tiên, hãy nhận ra rằng mục tiêu này vừa là một sự thay đổi tổ chức vừa là một thách thức kỹ thuật. Các SRE quen làm việc chặt chẽ với đồng đội, nhanh chóng phân tích và phản ứng với các vấn đề. Vì vậy, bạn đang làm việc chống lại bản năng tự nhiên của một SRE là nhanh chóng viết một số code để đáp ứng nhu cầu tức thời. Nếu đội SRE của bạn nhỏ, cách tiếp cận này có thể không gây vấn đề. Tuy nhiên, khi tổ chức của bạn phát triển, cách tiếp cận ad hoc này sẽ không scale, mà dẫn đến các giải pháp phần mềm phần lớn hoạt động được, nhưng hẹp hoặc chỉ dùng cho một mục đích, không thể chia sẻ, điều tất yếu dẫn đến nỗ lực trùng lặp và thời gian bị lãng phí.

Tiếp theo, hãy suy nghĩ về điều bạn muốn đạt được bằng cách phát triển phần mềm trong SRE. Bạn chỉ muốn nuôi dưỡng các thực hành phát triển phần mềm tốt hơn trong đội của mình, hay bạn quan tâm đến việc phát triển phần mềm tạo ra các kết quả có thể được sử dụng xuyên suốt các đội, có thể như một tiêu chuẩn cho tổ chức? Trong các tổ chức lớn đã ổn định, sự thay đổi sau sẽ mất nhiều thời gian, có thể trải qua nhiều năm. Một sự thay đổi như vậy cần được xử lý trên nhiều mặt, nhưng đổi lại có hiệu quả cao hơn. Dưới đây là một số hướng dẫn từ kinh nghiệm của Google:

**Tạo và truyền thông một thông điệp rõ ràng**

Quan trọng là phải định nghĩa và truyền thông chiến lược, kế hoạch, và — quan trọng nhất — các lợi ích SRE đạt được từ nỗ lực này. Các SRE là một giống hoài nghi (thực tế, sự hoài nghi là một đặc tính mà chúng tôi chủ động tuyển dụng); phản hồi ban đầu của một SRE đối với một nỗ lực như vậy nhiều khả năng là, "điều đó nghe có vẻ như quá nhiều gánh nặng" hoặc "nó sẽ không bao giờ hoạt động." Hãy bắt đầu bằng cách tạo một lập luận thuyết phục về cách chiến lược này sẽ giúp SRE; ví dụ:

-   Các giải pháp phần mềm nhất quán và được hỗ trợ tăng tốc quá trình ramp-up (chạy lên) cho các SRE mới.
-   Giảm số cách để thực hiện cùng một tác vụ cho phép toàn bộ phòng ban hưởng lợi từ kỹ năng mà bất kỳ đội đơn lẻ nào đã phát triển, qua đó giúp kiến thức và nỗ lực có thể di chuyển xuyên suốt các đội.

Khi các SRE bắt đầu đặt câu hỏi về *cách* chiến lược của bạn sẽ hoạt động, thay vì *liệu* chiến lược có nên được theo đuổi, bạn biết mình đã vượt qua rào cản đầu tiên.

**Đánh giá các khả năng của tổ chức bạn**

Các SRE có nhiều kỹ năng, nhưng khá phổ biến khi một SRE thiếu kinh nghiệm với tư cách là một phần của một đội đã xây dựng và phát hành một sản phẩm đến một tập người dùng. Để phát triển phần mềm hữu ích, về bản chất bạn đang tạo ra một đội sản phẩm. Đội đó bao gồm các vai trò và kỹ năng cần thiết mà tổ chức SRE của bạn có thể chưa từng đòi hỏi trước đó. Liệu có ai đó đóng vai trò quản lý sản phẩm, hoạt động như người ủng hộ khách hàng không? Liệu tech lead (lãnh đạo kỹ thuật) hay quản lý dự án của bạn có kỹ năng và/hoặc kinh nghiệm để chạy một quy trình phát triển agile (linh hoạt) không?

Bắt đầu lấp đầy những khoảng trống này bằng cách tận dụng các kỹ năng đã có sẵn trong công ty của bạn. Hãy để đội phát triển sản phẩm của bạn giúp thiết lập các thực hành agile thông qua đào tạo hoặc hướng dẫn. Xin thời gian tư vấn từ một quản lý sản phẩm để giúp định nghĩa các yêu cầu sản phẩm và ưu tiên hóa công việc tính năng. Với một cơ hội phát triển phần mềm đủ lớn, có thể có một lập luận để tuyển dụng những người chuyên tâm cho những vai trò này. Việc lập luận để tuyển dụng cho những vai trò này sẽ dễ dàng hơn một khi bạn đã có một số kết quả thử nghiệm tích cực.

**Khởi động và lặp lại**

Khi bạn khởi động một chương trình phát triển phần mềm SRE, nỗ lực của bạn sẽ được theo dõi bởi nhiều con mắt chăm chú. Quan trọng là phải thiết lập uy tín bằng cách giao một số sản phẩm có giá trị trong một khoảng thời gian hợp lý. Vòng sản phẩm đầu tiên nên nhắm đến các mục tiêu tương đối đơn giản và có thể đạt được — những cái không gây tranh cãi hay có sẵn giải pháp. Chúng tôi cũng tìm thấy thành công trong việc ghép cặp cách tiếp cận này với nhịp điệu sáu tháng của các release cập nhật sản phẩm mang lại các tính năng hữu ích bổ sung. Chu kỳ release này cho phép các đội tập trung vào việc xác định đúng tập tính năng để xây dựng, rồi xây dựng những tính năng đó trong khi đồng thời học cách trở thành một đội phát triển phần mềm năng suất. Sau lần khởi động ban đầu, một số đội Google chuyển sang mô hình push-on-green (đẩy khi xanh) để giao hàng và phản hồi nhanh hơn nữa.

**Đừng hạ thấp các chuẩn mực của bạn**

Khi bắt đầu phát triển phần mềm, bạn có thể bị cám dỗ cắt góc (cut corners). Hãy chống lại sự cám dỗ này bằng cách giữ bản thân ở cùng các chuẩn mực mà các đội phát triển sản phẩm của bạn được yêu cầu. Ví dụ:

-   Tự hỏi mình: nếu sản phẩm này được tạo ra bởi một đội dev (phát triển) riêng biệt, bạn có sẵn sàng onboard sản phẩm không?
-   Nếu giải pháp của bạn hưởng lợi từ việc được chấp nhận rộng rãi, nó có thể trở thành thiết yếu để các SRE thực hiện công việc của họ thành công. Vì vậy, độ tin cậy là quan trọng nhất. Bạn có các thực hành xem xét code phù hợp không? Bạn có kiểm thử đầu-cuối hoặc tích hợp không? Hãy để một đội SRE khác xem xét sản phẩm về sự sẵn sàng production, như cách họ sẽ làm khi onboard bất kỳ dịch vụ nào khác.

Mất rất nhiều thời gian để xây dựng uy tín cho nỗ lực phát triển phần mềm của bạn, nhưng chỉ cần một bước đi sai là uy tín đó có thể tan biến.

## Kết luận (Conclusions)

Các dự án kỹ thuật phần mềm trong Google SRE đã phát triển thịnh vượng cùng với sự phát triển của tổ chức, và trong nhiều trường hợp, những bài học rút ra từ và sự thành công trong triển khai của các dự án phát triển phần mềm trước đó đã mở đường cho những nỗ lực tiếp theo. Kinh nghiệm production trực tiếp độc đáo mà các SRE mang đến việc phát triển công cụ có thể dẫn đến các cách tiếp cận đổi mới cho những vấn đề lâu năm, như được thấy với việc phát triển Auxon để giải quyết vấn đề phức tạp của lập kế hoạch năng lực. Các dự án phần mềm do SRE thúc đẩy cũng đáng chú ý ở chỗ mang lại lợi ích cho công ty trong việc xây dựng một mô hình bền vững để hỗ trợ các dịch vụ ở quy mô. Vì các SRE thường phát triển phần mềm để tinh giản các quy trình kém hiệu quả hoặc tự động hóa các tác vụ chung, những dự án này có nghĩa là đội SRE không phải scale tuyến tính với kích thước của các dịch vụ mà họ hỗ trợ. Cuối cùng, các lợi ích của việc có SRE dành một phần thời gian cho phát triển phần mềm được công ty, tổ chức SRE, và chính các SRE cùng thu hoạch.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
