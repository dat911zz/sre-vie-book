# Chương 8. Release Engineering (Kỹ thuật Phát hành)

> **Nguyên bản:** [Chapter 8 - Release Engineering](https://sre.google/sre-book/release-engineering/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Thực hiện bởi Đội ngũ R&D Softdreams)

---

*Tác giả:* Dinah McNutt
*Biên tập:* Betsy Beyer và Tim Harvey

Release engineering (kỹ thuật phát hành) là một lĩnh vực tương đối mới trong kỹ thuật phần mềm, đang phát triển nhanh, có thể tóm tắt ngắn gọn là xây dựng và cung cấp phần mềm [[McN14a]](https://sre.google/sre-book/bibliography#McN14a). Các kỹ sư phát hành (release engineer) nắm vững (dù không phải chuyên gia) về quản lý mã nguồn, trình biên dịch (compiler), ngôn ngữ cấu hình build (dựng), các công cụ build tự động, trình quản lý gói (package manager) và các trình cài đặt. Bộ kỹ năng của họ bao gồm kiến thức sâu về nhiều lĩnh vực: phát triển, quản lý cấu hình, tích hợp kiểm thử, quản trị hệ thống và hỗ trợ khách hàng.

Để vận hành các dịch vụ đáng tin cậy, cần có quy trình phát hành đáng tin cậy. Các Site Reliability Engineer (SRE — Kỹ sư Độ tin cậy Trang web) cần đảm bảo rằng các binary (file thực thi) và cấu hình họ sử dụng được xây dựng theo cách tái lập được và tự động hóa, nhằm giúp các release (phát hành) có thể lặp lại được thay vì trở thành "những bông tuyết độc nhất" (unique snowflakes). Mọi thay đổi đối với bất kỳ khía cạnh nào của quy trình phát hành đều phải có chủ đích, không được để xảy ra vô tình. Các SRE quan tâm đến quy trình này từ mã nguồn cho đến triển khai (deployment).

Release engineering là một chức năng công việc cụ thể tại Google. Các kỹ sư phát hành phối hợp với kỹ sư phần mềm (SWE) trong phát triển sản phẩm và SRE để xác định toàn bộ các bước cần thiết để phát hành phần mềm — từ cách phần mềm được lưu trong repository (kho lưu trữ) mã nguồn, đến các quy tắc build cho biên dịch, cho đến cách kiểm thử, đóng gói (packaging) và triển khai được thực hiện.

## Vai trò của một Kỹ sư Phát hành (The Role of a Release Engineer)

Google là một công ty dựa trên dữ liệu (data-driven), và [release engineering](https://sre.google/workbook/canarying-releases/) cũng vậy. Chúng tôi có các công cụ báo cáo về một loạt các metrics (chỉ số), như mất bao nhiêu thời gian để một thay đổi code được triển khai vào production (nói cách khác, tốc độ phát hành — release velocity) và thống kê về các tính năng nào đang được dùng trong các tệp cấu hình build [[Ada15]](https://sre.google/sre-book/bibliography#Ada15). Các kỹ sư phát hành đã hình dung và phát triển phần lớn các công cụ này.

Các kỹ sư phát hành định nghĩa các best practices (thực hành tốt nhất) cho việc sử dụng các công cụ của chúng tôi để đảm bảo các dự án được phát hành bằng các phương pháp nhất quán và lặp lại được. Các best practices của chúng tôi bao phủ tất cả các yếu tố của quy trình phát hành. Các ví dụ bao gồm các cờ trình biên dịch (compiler flags), các định dạng cho các thẻ định danh build (build identification tags) và các bước bắt buộc trong một build. Đảm bảo rằng các công cụ của chúng tôi hoạt động đúng theo mặc định và được tài liệu hóa đầy đủ giúp các đội dễ dàng tập trung vào các tính năng và người dùng, thay vì dành thời gian phát minh lại bánh xe (kém hiệu quả) khi nói đến việc phát hành phần mềm.

Google có đội ngũ SRE đông đảo, chịu trách nhiệm triển khai an toàn các sản phẩm và duy trì hoạt động của các dịch vụ. Để quy trình phát hành đáp ứng yêu cầu kinh doanh, kỹ sư phát hành và SRE phối hợp xây dựng [chiến lược canary (chạy thử nghiệm)](https://sre.google/workbook/canarying-releases/) cho các thay đổi, giúp đẩy release mới ra mà không làm gián đoạn dịch vụ và rollback (hoàn tác) các tính năng có vấn đề.

## Triết lý (Philosophy)

Một triết lý kỹ thuật và dịch vụ dẫn dắt release engineering, thể hiện qua bốn nguyên lý chính, được trình bày chi tiết trong các phần sau.

### Mô hình Tự phục vụ (Self-Service Model)

Để vận hành ở quy mô lớn, các đội phải tự chủ. Release engineering đã xây dựng các best practices và công cụ, giúp các đội phát triển sản phẩm của chúng tôi tự kiểm soát và vận hành quy trình phát hành. Dù có hàng nghìn kỹ sư và sản phẩm, chúng tôi vẫn duy trì tốc độ phát hành cao vì mỗi đội có quyền quyết định thời điểm và tần suất ra mắt phiên bản mới. Quy trình phát hành có thể tự động hóa đến mức kỹ sư chỉ cần can thiệp tối thiểu; nhiều dự án thậm chí được build và phát hành hoàn toàn tự động nhờ kết hợp hệ thống build tự động hóa với các công cụ triển khai của chúng tôi. Các release thực sự là tự động, kỹ sư chỉ tham gia khi có vấn đề phát sinh.

### Tốc độ Cao (High Velocity)

Phần mềm hướng người dùng (như nhiều thành phần của Google Search) được build lại thường xuyên, vì chúng tôi muốn triển khai các tính năng cho khách hàng càng nhanh càng tốt. Chúng tôi theo triết lý: release thường xuyên giúp giảm thiểu thay đổi giữa các phiên bản, từ đó việc kiểm thử và debug trở nên dễ dàng hơn. Một số đội thực hiện build theo giờ, sau đó chọn phiên bản để triển khai vào production từ hồ (pool) các build kết quả. Việc lựa chọn dựa trên kết quả kiểm thử và các tính năng chứa trong một build cụ thể. Các đội khác áp dụng mô hình phát hành "Push on Green" (Đẩy khi Xanh), triển khai mọi build vượt qua tất cả các kiểm thử [[Kle14]](https://sre.google/sre-book/bibliography#Kle14).

### Các Build Hermetic (Hermetic Builds)

Công cụ build phải đảm bảo tính nhất quán và khả năng lặp lại. Khi hai người build cùng một sản phẩm ở cùng một revision (phiên bản) trong repository mã nguồn trên các machine khác nhau, kết quả phải giống hệt nhau.<sup>[1](#fn1)</sup> Các build của chúng tôi là hermetic, tức là không bị ảnh hưởng bởi các thư viện (libraries) hay phần mềm khác đã cài đặt trên machine build. Thay vào đó, build chỉ phụ thuộc vào các phiên bản đã biết của công cụ build (như trình biên dịch) và các sự phụ thuộc (như thư viện). Quy trình build là tự chứa (self-contained), không dựa vào các dịch vụ bên ngoài môi trường build.

Việc build lại các release cũ khi cần sửa một bug trong phần mềm đang chạy trên production có thể là một thách thức. Chúng tôi xử lý bằng cách build lại ở cùng revision với build ban đầu, đồng thời bao gồm các thay đổi cụ thể được nộp sau thời điểm đó. Chiến thuật này được gọi là *cherry picking*. Các công cụ build của chúng tôi cũng được phiên bản hóa dựa trên revision trong repository mã nguồn của dự án đang build. Do đó, nếu cần cherry pick, một dự án build tháng trước sẽ không dùng phiên bản trình biên dịch của tháng này, vì phiên bản đó có thể chứa các tính năng không tương thích hoặc không mong muốn.

### Thi hành Các Chính sách và Quy trình (Enforcement of Policies and Procedures)

Nhiều tầng bảo mật và kiểm soát truy cập quy định ai được phép thực hiện thao tác nào khi phát hành dự án. Các thao tác cần phê duyệt bao gồm:

-   Phê duyệt các thay đổi mã nguồn — thao tác này được quản lý thông qua các tệp cấu hình rải rác trong codebase (kho mã nguồn)
-   Quy định các hành động được thực hiện trong quy trình phát hành
-   Tạo một release mới
-   Phê duyệt đề xuất tích hợp ban đầu (yêu cầu thực hiện build tại một số revision cụ thể trong repository mã nguồn) và các cherry pick tiếp theo
-   Triển khai một release mới
-   Thực hiện các thay đổi đối với cấu hình build của một dự án

Gần như mọi thay đổi trên codebase đều phải qua code review, một bước đã được tinh gọn và tích hợp vào luồng công việc thông thường của developer. Hệ thống phát hành tự động hóa tạo ra báo cáo liệt kê tất cả các thay đổi trong một release, lưu trữ cùng với các sản phẩm build khác. Nhờ báo cáo này, SRE dễ dàng xác định những thay đổi nào nằm trong release mới của dự án, từ đó đẩy nhanh quá trình debug khi release gặp sự cố.

## Build và Triển khai Liên tục (Continuous Build and Deployment)

Google đã phát triển hệ thống phát hành tự động hóa mang tên *Rapid*. Hệ thống này tận dụng một số công nghệ của Google để cung cấp khung (framework) tạo ra các release có khả năng mở rộng, hermetic và đáng tin cậy. Các phần sau mô tả vòng đời phần mềm tại Google và cách nó được quản lý bằng Rapid cùng các công cụ liên quan khác.

### Build (Dựng)

Blaze<sup>[2](#fn2)</sup> là công cụ build được Google chọn. Nó hỗ trợ build các binary từ nhiều ngôn ngữ, bao gồm các ngôn ngữ chuẩn của chúng tôi: C++, Java, Python, Go và JavaScript. Các kỹ sư dùng Blaze để định nghĩa các mục tiêu build (build targets, ví dụ, đầu ra của một build, như một tệp JAR) và quy định các sự phụ thuộc cho mỗi mục tiêu [[Kem11]](https://sre.google/sre-book/bibliography#Kem11). Khi thực hiện một build, Blaze tự động build các mục tiêu phụ thuộc.

Các mục tiêu build cho binary và kiểm thử unit được định nghĩa trong tệp cấu hình dự án của Rapid. Rapid truyền các cờ cụ thể của dự án, chẳng hạn như bộ định danh build độc nhất, cho Blaze. Tất cả binary đều hỗ trợ cờ hiển thị ngày build, số revision và bộ định danh build, giúp chúng tôi dễ dàng liên kết một binary với bản ghi về cách nó được build.

### Nhánh (Branching)

Toàn bộ code được check vào nhánh chính (main branch) của cây mã nguồn (mainline). Tuy nhiên, phần lớn các dự án lớn không phát hành trực tiếp từ mainline. Thay vào đó, chúng tôi tạo nhánh (branch) từ mainline tại một revision cụ thể và không bao giờ merge (hợp nhất) các thay đổi từ nhánh đó trở lại mainline. Các bản sửa lỗi được nộp vào mainline, sau đó cherry pick sang nhánh để đưa vào release. Cách làm này giúp tránh vô tình kéo theo những thay đổi không liên quan đã được nộp vào mainline kể từ lần build ban đầu. Nhờ sử dụng phương pháp nhánh và cherry pick, chúng tôi nắm rõ chính xác nội dung của từng release.

### Kiểm thử (Testing)

Một hệ thống kiểm thử liên tục chạy [các kiểm thử unit](https://sre.google/sre-book/testing-reliability/) đối với code trong mainline mỗi khi có thay đổi được nộp, giúp chúng tôi phát hiện nhanh các lỗi build và kiểm thử. Release engineering khuyến nghị các mục tiêu kiểm thử build liên tục nên trùng khớp với các mục tiêu kiểm thử chặn (gate) release của dự án. Chúng tôi cũng khuyến nghị tạo release tại số revision (phiên bản) của build kiểm thử liên tục cuối cùng đã hoàn thành thành công tất cả các kiểm thử. Những biện pháp này giúp giảm nguy cơ các thay đổi tiếp theo trên mainline gây ra thất bại build tại thời điểm phát hành.

Trong quy trình phát hành, chúng tôi chạy lại các kiểm thử unit trên nhánh release để tạo vết kiểm toán (audit trail) xác nhận tất cả kiểm thử đều vượt qua. Bước này rất quan trọng, bởi nếu release liên quan đến cherry pick, nhánh release có thể chứa phiên bản code không tồn tại ở đâu trên mainline. Chúng tôi cần đảm bảo các kiểm thử vượt qua trong đúng ngữ cảnh của những gì thực sự được phát hành.

Bổ sung cho hệ thống kiểm thử liên tục, chúng tôi dùng một môi trường kiểm thử độc lập để chạy các kiểm thử cấp hệ thống trên các bản build đã đóng gói. Các kiểm thử này có thể khởi động thủ công hoặc từ Rapid.

### Đóng gói (Packaging)

Phần mềm được phân phối đến các machine production của chúng tôi qua Trình quản lý Gói Midas (Midas Package Manager, MPM) [[McN14c]](https://sre.google/sre-book/bibliography#McN14c). MPM lắp ráp các gói dựa trên các quy tắc Blaze liệt kê các sản phẩm build cần bao gồm, cùng với các chủ sở hữu và đặc quyền của chúng. Các gói được đặt tên (ví dụ, *search/shakespeare/frontend*), được phiên bản hóa với một hash (mã băm) độc nhất và được ký để đảm bảo tính xác thực. MPM hỗ trợ việc áp dụng các nhãn (labels) cho một phiên bản cụ thể của một gói. Rapid áp dụng một nhãn chứa ID build, điều này đảm bảo rằng một gói có thể được tham chiếu độc nhất bằng cách dùng tên của gói và nhãn này.

Các nhãn có thể được áp dụng cho một gói MPM để chỉ ra vị trí của gói trong quy trình phát hành (ví dụ, `dev` (phát triển), `canary` (thử nghiệm) hoặc `production` (sản xuất)). Nếu bạn áp dụng một nhãn hiện có cho một gói mới, nhãn tự động được chuyển từ gói cũ sang gói mới. Ví dụ: nếu một gói được gắn nhãn `canary`, ai đó sau đó cài đặt phiên bản canary của gói đó sẽ tự động nhận phiên bản mới nhất của gói với nhãn `canary`.

### Rapid

[Hình 8-1](#hinh-8-1) hiển thị các thành phần chính của hệ thống Rapid. Hệ thống này được cấu hình thông qua các tệp gọi là *blueprints* (bản thiết kế). Blueprints được viết bằng một ngôn ngữ cấu hình nội bộ, dùng để định nghĩa các mục tiêu build và kiểm thử, các quy tắc cho triển khai, cùng thông tin quản trị (như các chủ sở hữu dự án). Các danh sách kiểm soát truy cập dựa trên vai trò (role-based access control lists) xác định ai có thể thực hiện các hành động cụ thể trên một dự án Rapid.


<a id="hinh-8-1"></a>![Hình 8-1](../assets/imgs/fig-8-1.jpg)

[Hình 8-1.](#hinh-8-1) Khung nhìn đơn giản hóa về kiến trúc Rapid hiển thị các thành phần chính của hệ thống.

Mỗi dự án Rapid định nghĩa các luồng công việc (workflows) quy định những hành động cần thực hiện trong quy trình phát hành. Các hành động này có thể chạy tuần tự hoặc song song, và một luồng công việc có thể khởi động các luồng khác. Rapid điều phối (dispatch) các yêu cầu công việc đến các task chạy dưới dạng job Borg trên các server production của chúng tôi. Nhờ sử dụng hạ tầng production, Rapid có khả năng xử lý hàng nghìn yêu cầu phát hành đồng thời.

Một quy trình phát hành điển hình diễn ra như sau:

1.  Rapid dùng số revision tích hợp được yêu cầu (thường được lấy tự động từ hệ thống kiểm thử liên tục của chúng tôi) để tạo một nhánh release.
2.  Rapid dùng Blaze để biên dịch tất cả các binary và chạy kiểm thử unit, thường thực hiện hai bước này song song. Quá trình biên dịch và kiểm thử diễn ra trong các môi trường dành riêng cho từng tác vụ, chứ không chạy trong job Borg nơi luồng công việc Rapid đang thực thi. Sự tách biệt này giúp chúng tôi dễ dàng song song hóa công việc.
3.  Các bản build sau đó sẵn sàng để kiểm thử hệ thống và triển khai canary. Một lần triển khai canary điển hình sẽ khởi động một vài job trong môi trường production sau khi hoàn tất các kiểm thử hệ thống.
4.  Kết quả của mỗi bước của quy trình được ghi log (nhật ký). Một báo cáo về tất cả các thay đổi kể từ release trước được tạo ra.

Rapid giúp chúng tôi quản lý các nhánh release và cherry pick; mỗi yêu cầu cherry pick riêng lẻ đều có thể được phê duyệt hoặc từ chối để đưa vào một release.

### Triển khai (Deployment)

Rapid thường dùng để trực tiếp điều khiển các triển khai đơn giản. Hệ thống này cập nhật các job Borg nhằm sử dụng các gói MPM mới được build, dựa trên định nghĩa triển khai trong các tệp blueprint và các trình thực thi task chuyên dụng.

Đối với các triển khai phức tạp hơn, chúng tôi dùng Sisyphus, một khung tự động hóa rollout tổng quát do SRE phát triển. Một rollout là một đơn vị công việc logic được tạo thành từ một hoặc nhiều task riêng lẻ. Sisyphus cung cấp một tập các lớp Python có thể được mở rộng để hỗ trợ bất kỳ quy trình triển khai nào. Nó có một dashboard (bảng điều khiển) cho phép kiểm soát chi tiết hơn về cách rollout được thực hiện và cung cấp một cách để giám sát tiến độ của rollout.

Trong một tích hợp điển hình, Rapid tạo một rollout trong job Sisyphus chạy dài. Rapid biết nhãn build (build label) liên quan đến gói MPM mà nó đã tạo ra, và có thể quy định nhãn build đó khi tạo rollout trong Sisyphus. Sisyphus dùng nhãn build để quy định phiên bản nào của các gói MPM nên được triển khai.

Với Sisyphus, quy trình rollout có thể đơn giản hoặc phức tạp tùy theo nhu cầu. Chẳng hạn, hệ thống có thể cập nhật ngay lập tức tất cả các job liên quan, hoặc lần lượt rollout binary mới đến các cluster trong vài giờ.

Mục tiêu của chúng tôi là điều chỉnh quy trình triển khai cho phù hợp với hồ sơ rủi ro của từng dịch vụ. Trong môi trường phát triển hoặc trước-production, chúng ta có thể build theo giờ và push (đẩy) các release tự động khi tất cả các kiểm thử vượt qua. Với các dịch vụ hướng người dùng lớn, chúng ta có thể bắt đầu push từ một cluster rồi mở rộng theo hàm mũ cho đến khi tất cả các cluster được cập nhật. Đối với các thành phần hạ tầng nhạy cảm, chúng ta có thể kéo dài quá trình rollout trong vài ngày, xen kẽ trên các instance ở những khu vực địa lý khác nhau.

## Quản lý Cấu hình (Configuration Management)

Quản lý cấu hình là mảng hợp tác đặc biệt chặt chẽ giữa các kỹ sư phát hành và SRE. Thoạt nhìn, quản lý cấu hình có vẻ đơn giản, nhưng thực tế các thay đổi cấu hình lại là nguồn tiềm ẩn gây mất ổn định. Do đó, cách chúng tôi tiếp cận việc phát hành và quản lý cấu hình hệ thống, dịch vụ đã thay đổi đáng kể theo thời gian. Hiện tại, chúng tôi sử dụng một số mô hình để phân phối tệp cấu hình, như sẽ mô tả ở các đoạn sau. Tất cả các schema đều liên quan đến việc lưu trữ cấu hình trong repository mã nguồn chính và thực thi yêu cầu xem xét code nghiêm ngặt.

*Sử dụng mainline cho cấu hình.* Đây là phương pháp đầu tiên được dùng để cấu hình các dịch vụ trong Borg (và các hệ thống trước đó của Borg). Với schema này, các developer và SRE sửa đổi các tệp cấu hình ở đầu nhánh chính. Các thay đổi được xem xét và sau đó được áp dụng cho hệ thống đang chạy. Kết quả là, các release binary và các thay đổi cấu hình được tách rời. Kỹ thuật này tuy đơn giản về mặt khái niệm và quy trình, nhưng thường dẫn đến sự lệch (skew) giữa phiên bản check-in của các tệp cấu hình và phiên bản chạy của tệp cấu hình, vì các job phải được cập nhật để nhặt lên các thay đổi.

*Bao gồm các tệp cấu hình và binary trong cùng một gói MPM.* Với những dự án có ít tệp cấu hình, hoặc các dự án mà một số tệp (hoặc toàn bộ) thay đổi theo từng chu kỳ release, ta có thể đóng gói các tệp cấu hình này cùng với binary trong gói MPM. Cách làm này hạn chế tính linh hoạt do gắn chặt binary với tệp cấu hình, nhưng đổi lại giúp việc triển khai đơn giản hơn vì chỉ cần cài đặt một gói duy nhất.

*Đóng gói các tệp cấu hình vào các "gói cấu hình" MPM.* Chúng tôi có thể áp dụng nguyên lý hermetic cho quản lý cấu hình. Vì các cấu hình binary thường bị ràng buộc chặt với các phiên bản cụ thể của binary, nên chúng tôi tận dụng các hệ thống build và đóng gói để chụp nhanh (snapshot) và phát hành các tệp cấu hình cùng với các binary tương ứng. Tương tự cách xử lý binary, chúng tôi có thể dùng ID build để tái tạo cấu hình tại một thời điểm cụ thể.

Ví dụ, một thay đổi để triển khai tính năng mới có thể được phát hành kèm theo một flag cấu hình cho tính năng đó. Bằng cách tách thành hai gói MPM, một cho binary và một cho cấu hình, chúng tôi có thể thay đổi từng gói độc lập. Chẳng hạn, nếu tính năng được phát hành với flag `first_folio` nhưng sau đó chúng tôi nhận ra cần dùng `bad_quarto`, chúng tôi chỉ cần cherry pick thay đổi đó lên nhánh release, build lại gói cấu hình rồi triển khai. Cách tiếp cận này có lợi thế là không đòi hỏi phải build lại binary.

Chúng tôi có thể dùng chức năng nhãn của MPM để chỉ ra các phiên bản gói MPM nào nên được cài đặt cùng nhau. Ví dụ, nhãn `much_ado` có thể được gán cho các gói MPM đã đề cập ở đoạn trước, giúp chúng tôi lấy cả hai gói mang nhãn này. Mỗi khi build một phiên bản mới của dự án, nhãn `much_ado` sẽ được áp dụng cho các gói mới. Do các thẻ này là duy nhất trong không gian tên của một gói MPM, chỉ phiên bản mới nhất mang thẻ đó sẽ được sử dụng.

*Đọc các tệp cấu hình từ một kho bên ngoài.* Một số dự án có các tệp cấu hình cần thay đổi thường xuyên hoặc động (tức là, trong khi binary đang chạy). Các tệp này có thể được lưu trữ trong Chubby, Bigtable, hoặc hệ thống tệp dựa trên mã nguồn của chúng tôi [[Yor11]](https://sre.google/sre-book/bibliography#Yor11).

Tóm lại, chủ sở hữu dự án sẽ cân nhắc các phương án khác nhau để phân phối và quản lý tệp cấu hình, sau đó chọn ra phương án phù hợp nhất cho từng trường hợp cụ thể.

## Kết luận

Mặc dù chương này tập trung vào cách tiếp cận của Google đối với release engineering và cách các kỹ sư phát hành phối hợp với SRE, những thực hành này hoàn toàn có thể áp dụng rộng hơn.

### Không chỉ dành cho các Googler (It's Not Just for Googlers)

Khi có đủ công cụ phù hợp, tự động hóa thích hợp và chính sách rõ ràng, developer và SRE không cần lo lắng về việc phát hành phần mềm. Các release có thể đơn giản chỉ là nhấn một nút.

Dù quy mô hay công cụ ra sao, phần lớn các công ty đều phải đối mặt với cùng một nhóm vấn đề về release engineering: Phiên bản hóa các gói nên được xử lý như thế nào? Nên áp dụng mô hình build và triển khai liên tục hay thực hiện các build định kỳ? Chu kỳ phát hành bao lâu một lần? Nên dùng chính sách quản lý cấu hình nào? Và những metrics phát hành nào đáng quan tâm?

Các kỹ sư phát hành tại Google đã tự phát triển công cụ vì các giải pháp mã nguồn mở hoặc của nhà cung cấp không đáp ứng được quy mô hoạt động của chúng tôi. Những công cụ tùy chỉnh này cho phép tích hợp chức năng hỗ trợ, thậm chí thực thi, các chính sách quy trình phát hành. Tuy nhiên, trước tiên cần định nghĩa rõ các chính sách này để thêm tính năng phù hợp vào công cụ; đồng thời, mọi công ty nên dành công sức để xác định quy trình phát hành của mình, bất kể quy trình đó có thể được tự động hóa và/hoặc thực thi hay không.

### Bắt đầu Release Engineering Ngay từ Đầu (Start Release Engineering at the Beginning)

Release engineering thường bị xem là điều cần lo sau cùng, nhưng khi các nền tảng và dịch vụ ngày càng lớn và phức tạp hơn, cách nghĩ này cần phải thay đổi.

Các đội nên dự trù tài nguyên cho release engineering ngay từ đầu chu kỳ phát triển sản phẩm. Việc thiết lập các thực hành và quy trình tốt ngay từ đầu sẽ tiết kiệm chi phí hơn so với việc phải retrofit (trang bị bổ sung cho hệ thống sẵn có) về sau.

Việc các developer, SRE và kỹ sư phát hành phối hợp chặt chẽ là điều thiết yếu. Kỹ sư phát hành cần nắm rõ ý định về cách code nên được build và triển khai. Các developer không nên build xong rồi “ném kết quả qua hàng rào” để các kỹ sư phát hành xử lý.

Mỗi đội dự án tự quyết định thời điểm đưa release engineering vào. Vì đây vẫn là lĩnh vực tương đối mới, các quản lý không phải lúc nào cũng lên kế hoạch và dự trù cho release engineering ngay từ đầu. Do đó, khi xem xét cách tích hợp các thực hành release engineering, hãy đảm bảo bạn nhìn nhận vai trò của nó trong toàn bộ vòng đời sản phẩm hoặc dịch vụ — đặc biệt là ở các giai đoạn đầu.

### Thông tin thêm (More Information)

Để biết thêm thông tin về release engineering, xem các bài thuyết trình sau, mỗi bài đều có video khả dụng trực tuyến:

-   [*How Embracing Continuous Release Reduced Change Complexity*](https://usenix.org/conference/ures14west/summit-program/presentation/dickson) (Làm thế nào việc áp dụng Phát hành Liên tục giúp giảm độ phức tạp của Thay đổi), USENIX Release Engineering Summit West 2014, [[Dic14]](https://sre.google/sre-book/bibliography#Dic14)
-   [*Maintaining Consistency in a Massively Parallel Environment*](https://www.usenix.org/conference/ucms13/summit-program/presentation/mcnutt) (Duy trì Sự nhất quán trong một Môi trường Song song Lớn), USENIX Configuration Management Summit 2013, [[McN13]](https://sre.google/sre-book/bibliography#McN13)
-   [*The 10 Commandments of Release Engineering*](https://www.youtube.com/watch?v=RNMjYV_UsQ8) (10 Điều răn của Release Engineering), 2nd International Workshop on Release Engineering 2014, [[McN14b]](https://sre.google/sre-book/bibliography#McN14b)
-   [*Distributing Software in a Massively Parallel Environment*](https://www.usenix.org/conference/lisa14/conference-program/presentation/mcnutt) (Phân phối Phần mềm trong một Môi trường Song song Lớn), LISA 2014, [[McN14c]](https://sre.google/sre-book/bibliography#McN14c)

<a id="fn1"></a>[1](#fn1) Google dùng một repository mã nguồn thống nhất monolithic (toàn khối); xem [[Pot16]](https://sre.google/sre-book/bibliography#Pot16).

<a id="fn2"></a>[2](#fn2) Blaze đã được mã nguồn mở dưới tên Bazel. Xem "Bazel FAQ" trên website Bazel, [*https://bazel.build/faq.html*](https://bazel.build/faq.html).

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
