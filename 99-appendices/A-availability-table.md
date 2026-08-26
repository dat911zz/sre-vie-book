> **Nguyên bản:** [Appendix A. Availability Table](https://sre.google/sre-book/availability-table/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Đội biên tập Softdreams RnD)

---

## Phụ lục A. Bảng Độ khả dụng (Appendix A. Availability Table)

Độ khả dụng (availability) nói chung được tính dựa trên khoảng thời gian một dịch vụ không khả dụng trong suốt một giai đoạn nào đó. Giả định không có thời gian ngừng hoạt động có kế hoạch, [Bảng 1-1](#bang-a-1) cho biết lượng thời gian ngừng hoạt động được phép là bao nhiêu để đạt đến một mức độ khả dụng cho trước.

<a id="bang-a-1"></a>Bảng 1-1. Bảng độ khả dụng

**Mức độ khả dụng** | (Cửa sổ không khả dụng được phép) | **mỗi năm** | **mỗi quý** | **mỗi tháng** | **mỗi tuần** | **mỗi ngày** | **mỗi giờ**
--- | --- | --- | --- | --- | --- | --- | ---
90% | | 36.5 ngày | 9 ngày | 3 ngày | 16.8 giờ | 2.4 giờ | 6 phút
95% | | 18.25 ngày | 4.5 ngày | 1.5 ngày | 8.4 giờ | 1.2 giờ | 3 phút
99% | | 3.65 ngày | 21.6 giờ | 7.2 giờ | 1.68 giờ | 14.4 phút | 36 giây
99.5% | | 1.83 ngày | 10.8 giờ | 3.6 giờ | 50.4 phút | 7.20 phút | 18 giây
99.9% | | 8.76 giờ | 2.16 giờ | 43.2 phút | 10.1 phút | 1.44 phút | 3.6 giây
99.95% | | 4.38 giờ | 1.08 giờ | 21.6 phút | 5.04 phút | 43.2 giây | 1.8 giây
99.99% | | 52.6 phút | 12.96 phút | 4.32 phút | 60.5 giây | 8.64 giây | 0.36 giây
99.999% | | 5.26 phút | 1.30 phút | 25.9 giây | 6.05 giây | 0.87 giây | 0.04 giây

Sử dụng một chỉ số không khả dụng tổng hợp (tức là "*X*% tổng số các thao tác đã thất bại") hữu dụng hơn so với việc chỉ tập trung vào độ dài của các sự cố, đối với những dịch vụ có thể chỉ khả dụng một phần — chẳng hạn, do có nhiều bản sao (replica) mà chỉ một số trong số chúng không khả dụng — và đối với những dịch vụ mà tải thay đổi trong suốt một ngày hoặc một tuần thay vì giữ nguyên không đổi.

Xem các phương trình [Độ khả dụng theo thời gian](https://sre.google/sre-book/embracing-risk#risk-management_measuring-service-risk_time-availability-equation) và [Độ khả dụng tổng hợp](https://sre.google/sre-book/embracing-risk#risk-management_measuring-service-risk_aggregate-availability-equation) trong [Đón nhận Rủi ro (Embracing Risk)](https://sre.google/sre-book/embracing-risk/) để tính toán.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
