> **Nguyên bản:** [Appendix F. Example Production Meeting Minutes](https://sre.google/sre-book/production-meeting/)
> **Nguồn:** Google SRE Book (O'Reilly)
> **Bản dịch tiếng Việt** (Đội biên tập Softdreams RnD)

---

## Phụ lục F. Ví Dụ Biên Bản Họp Sản Xuất (Appendix F. Example Production Meeting Minutes)

**Ngày (Date)**: 2015-10-23

**Người tham dự (Attendees)**: agoogler, clarac, docbrown, jennifer, martym

**Thông báo (Announcements)**:

-   Sự cố lớn (outage) (#465), đã xuyên thủng ngân sách lỗi (error budget)

#### Xem xét mục hành động trước đó (Previous Action Item Review)

-   Chứng nhận Bộ teleported Dê (Goat Teleporter) để sử dụng với gia súc (bug 1011101)
    -   Các tính phi tuyến trong tăng tốc khối lượng nay có thể dự đoán, nên sẽ có thể nhắm chính xác trong vài ngày tới.

#### Xem xét sự cố (Outage Review)

-   Sonnet mới (outage 465)
    -   1.21B truy vấn bị mất do sự cố lan truyền sau tương tác giữa lỗi tiềm ẩn (rò file descriptor ở các lần tìm kiếm không có kết quả) + không có sonnet mới trong tập hợp + khối lượng giao thông chưa từng có và không ngờ tới
    -   Lỗi rò rỉ file descriptor đã được sửa (bug 5554825) và triển khai đến prod
    -   Đang xem xét việc sử dụng bộ tụ thông lượng (flux capacitor) để cân bằng tải (bug 5554823) và sử dụng loại bỏ tải (load shedding) (bug 5554826) để ngăn tái diễn
    -   Đã hủy diệt ngân sách lỗi độ khả dụng; việc đẩy lên prod bị đóng băng trong 1 tháng trừ khi docbrown có thể xin được ngoại lệ với lý do sự kiện là kỳ quặc & không thể lường trước (nhưng sự đồng thuận chung là ngoại lệ ít có khả năng)

#### Các Sự kiện Gọi người (Paging Events)

-   `AnnotationConsistencyTooEventual`: đã gọi người 5 lần tuần này, có khả năng do độ trễ sao chép liên vùng (cross-regional) giữa các Bigtable.
    -   Việc điều tra vẫn đang tiếp diễn, xem bug 4821600
    -   Không có bản sửa nào được dự kiến sớm, sẽ nâng ngưỡng tính nhất quán (consistency) được chấp nhận để giảm các cảnh báo không thể hành động

#### Các Sự kiện Không Gọi Người (Nonpaging Events)

-   Không có

#### Các Thay đổi Giám sát và/hoặc Im lặng (Silences)

-   `AnnotationConsistencyTooEventual`, ngưỡng độ trễ được chấp nhận đã được nâng từ 60s lên 180s, xem bug 4821600; TODO(martym).

#### Các Thay đổi Sản Xuất Có Kế hoạch (Planned Production Changes)

-   Cụm USA-1 sẽ offline để bảo trì giữa 2015-10-29 và 2015-11-02.
    -   Không cần phản hồi, giao thông sẽ tự động được định tuyến đến các cụm khác trong vùng.

#### Tài nguyên (Resources)

-   Tài nguyên đã mượn để phản hồi sự cố sonnet++, sẽ giảm (spin down) các instance server bổ sung và hoàn trả tài nguyên tuần tới
-   Mức sử dụng (Utilization) ở 60% CPU, 75% RAM, 44% đĩa (tăng từ 40%, 70%, 40% tuần trước)

#### Các Chỉ số Dịch vụ Chính (Key Service Metrics)

-   **OK** độ trễ 99ile: 88 ms < 100 ms mục tiêu SLO \[trailing 30 days\]
-   **BAD** độ khả dụng: 86.95% < 99.99% mục tiêu SLO \[trailing 30 days\]

#### Thảo luận / Cập nhật Dự án (Discussion / Project Updates)

-   Dự án Molière sẽ ra mắt trong hai tuần nữa.

#### Các Mục Hành Động Mới (New Action Items)

-   TODO(martym): Nâng ngưỡng `AnnotationConsistencyTooEventual`.
-   TODO(docbrown): Hoàn trả số lượng instance về bình thường và hoàn trả tài nguyên.

---

Copyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](https://creativecommons.org/licenses/by-nc-nd/4.0/)
