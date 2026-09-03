# Glossary thuật ngữ — Google SRE Book (bản dịch tiếng Việt)

Nguồn sự thật duy nhất (single source of truth) cho cách dịch thuật ngữ kỹ thuật xuyên suốt sách.
**Mọi lượt dịch/rà soát mới đều phải đọc file này trước khi bắt đầu, và bổ sung thuật ngữ mới vào đây trước khi kết thúc.**

Lý do file này tồn tại: đợt dịch 48 file gốc (2026-08-24) và đợt review 49 file (2026-08-25) đều
chỉ ép nhất quán chú thích **trong phạm vi 1 file** (xem `.kilo/plans/1787644138490-*.md` dòng 10 và
`.kilo/plans/1787644422483-*.md` dòng 60), không có glossary chung → cùng 1 thuật ngữ tiếng Anh bị dịch
5-6 kiểu khác nhau tùy chương (case điển hình: "graceful degradation").

## Quy tắc dùng bảng dưới

- Cột **Bản dịch chuẩn**: dùng CHÍNH XÁC cụm này khi thuật ngữ xuất hiện lần đầu trong 1 file mới.
- Cột **Biến thể đã phát hiện (SAI, cần thay)**: các cách dịch khác đang tồn tại rải rác trong sách —
  khi sửa file cũ hoặc gặp lại, thay về Bản dịch chuẩn.
- Thuật ngữ không có trong bảng: tra cứu, thêm dòng mới, rồi mới dịch — không tự bịa cách dịch mới cho
  thuật ngữ đã từng xuất hiện ở chương khác.
- Ưu tiên **giữ nguyên tiếng Anh** cho jargon đã phổ biến trong cộng đồng dev/SRE Việt Nam (xem mục
  "Nguyên tắc giữ nguyên tiếng Anh" bên dưới) — không ép dịch từng chữ.

## Bảng thuật ngữ đã chốt

| Thuật ngữ (EN) | Bản dịch chuẩn | Biến thể đã phát hiện (SAI, cần thay) | Ghi chú |
|---|---|---|---|
| page (danh từ/động từ, on-call) | **gọi trực**, gloss `(page)` ở lần xuất hiện đầu mỗi file, sau đó chỉ "gọi trực" | page (giữ nguyên EN — ĐÃ ĐỔI quy ước 2026-08-28, không còn dùng); paging (-ing); "trang" | page = tin nhắn/cuộc gọi khẩn tới người on-call. Dùng "gọi trực" làm danh từ ("một lần gọi trực", "3 lần gọi trực") và cụm động từ ("gọi ai đó đi trực", "bị gọi trực"). KHÔNG nhầm với memory page (trang bộ nhớ — ngữ cảnh OS/kỹ thuật khác hẳn, xem ch.14 L22, ch.17 L209, luôn có gloss rõ "(memory page)"/"bộ nhớ" đi kèm, KHÔNG đổi) |
| pager | **máy gọi trực**, gloss `(pager)` ở lần xuất hiện đầu mỗi file, sau đó chỉ "máy gọi trực" | pager (giữ nguyên EN — ĐÃ ĐỔI quy ước 2026-08-28); "thiết bị gọi trực" (quá dài) | pager = thiết bị/kênh nhận page. Phân biệt rõ với "gọi trực": máy gọi trực là vật/kênh, gọi trực là sự kiện/tin nhắn. Thành ngữ "give back the pager" → "trả lại máy gọi trực" (giữ nghĩa ẩn dụ nhường quyền on-call, không chỉ vật lý) |
| graceful degradation | **suy giảm nhẹ nhàng** | suy giảm tinh tế; suy giảm êm ả; suy giảm từ từ | Chốt theo bản đã dùng làm tên mục lớn ở ch.22 |
| graceful load shedding | **gánh nhẹ tải một cách nhẹ nhàng** | loại bỏ tải một cách tinh tế | Đồng bộ tính từ "nhẹ nhàng" với graceful degradation |
| thundering herd | **hiệu ứng bầy đàn (thundering herd)** | bầy đàn giông; "bầy thú đang gầm gừ" | Ch.22, 25, 27. Nhiều client đồng thời thực hiện cùng hành động |
| distributed consensus | **nhất trí phân tán** | đồng thuận phân tán | Không dùng "đồng thuận" — dành riêng cho "consensus" thường (không phân tán) nếu cần phân biệt |
| failure domain | **failure domain** (giữ nguyên) | domain lỗi; miền thất bại; miền lỗi | Jargon phổ biến, không dịch |
| failure (danh từ, sự kiện hệ thống/dịch vụ hỏng) | **sự cố** | sự thất bại | "sự thất bại" nghe nặng nề, sách vở — quyết định 2026-09-03 sau khi phát hiện lặp 30 lần ở ch.22. Dùng "sự cố" cho failure-là-sự-kiện (device failure, cascading failure, component failure). Vẫn giữ "thất bại" cho failure-là-động-từ/kết-quả-nỗ-lực (nỗ lực thất bại, lần thử thất bại) — không đổi cách dùng đó |
| churn | **churn** (giữ nguyên) | sự thay đổi; sự churn | |
| lease | **lease** (giữ nguyên) | thuê quyền (thời hạn) | |
| reverse engineering | **reverse engineering** (giữ nguyên) | phân tích ngược | |
| proxy (kỹ thuật, "chỉ số/vật đại diện") | **chỉ số đại diện (proxy)** hoặc giữ nguyên "proxy" | đại lý | "đại lý" là nghĩa thương mại (dealer), sai hoàn toàn trong ngữ cảnh kỹ thuật |
| embarrassingly parallel | **song song hoàn toàn (không phụ thuộc lẫn nhau)** | song song dễ chịu | Dịch khái niệm, không dịch từng chữ ("embarrassingly" không phải cảm xúc) |
| agnostic / agnosticism (thiết kế, công nghệ) | **trung lập (không ràng buộc công nghệ)** | thờ ơ | "thờ ơ" = indifference, sai nghĩa hoàn toàn |
| crowdsourcing | **huy động đám đông** | huy động đám mây | Nhầm "crowd" (đám đông) với "cloud" (đám mây) |
| fail sanely | **thất bại một cách hợp lý** | thua một cách hợp lý | "fail" = hệ thống gặp lỗi, không phải "thua" (thua trận) |
| lost our touch (thành ngữ) | **mất nhịp / giảm phản xạ** | mất tay | Lặp lỗi y hệt ở 2 file khác nhau — xác nhận lỗi hệ thống |
| on-call | **on-call** (giữ nguyên) | trực sự cố; trực sự kiện | Nếu cần chú thích lần đầu: `on-call (trực sự cố)` |
| postmortem | **postmortem** (giữ nguyên) | báo cáo sau sự cố | Nếu cần chú thích lần đầu: `postmortem (báo cáo sau sự cố)` |
| defense in depth | **phòng thủ nhiều lớp** | bảo vệ nhiều lớp (đa chiều) | |
| load shedding | **loại bỏ tải (load shedding)** | thứ tự chú thích đảo ngược tùy chỗ | Cố định thứ tự: tiếng Việt trước, `(load shedding)` sau |
| progressive rollout | **rollout tăng dần (progressive rollouts)** | rollout tiến bộ | "tiến bộ" = advanced/progress (sai), "progressive" ở đây nghĩa "mở rộng từng bước" |
| frontend / backend | **giữ nguyên "frontend"/"backend"** (xem mục giữ nguyên EN) | giao diện phía trước | "giao diện" = UI, sai trong ngữ cảnh SRE. CHỈ dùng gloss `(phía trước)`/`(phía sau)` khi câu đang ĐỊNH NGHĨA quan hệ vai trò (vd "frontend gọi là client") — không thay thế "frontend"/"backend" bằng tiếng Việt ở chỗ dùng thông thường |
| client / server (khi câu đang định nghĩa vai trò, vd "frontend gọi là client") | **client (phía gọi)** / **server (phía phục vụ)** | khách hàng / máy chủ | Chỉ áp dụng cho câu ĐANG ĐỊNH NGHĨA quan hệ gọi-phục vụ (tạo chuỗi song song với phía trước/phía sau). Dùng thông thường ở chỗ khác: giữ nguyên "client"/"server", KHÔNG đổi hàng loạt — "khách hàng"/"máy chủ" bản thân không sai, chỉ là lựa chọn khác |
| bisection bandwidth | **giữ nguyên "bisection bandwidth"**, nếu cần gloss: `(băng thông khi chia đôi mạng)` | băng thông đường chéo | "đường chéo" là suy diễn hình học SAI, không có trong định nghĩa gốc (bisection = chia đôi mạng ra 2 nửa, đo băng thông tối thiểu giữa 2 nửa đó) |
| virtual switch | **virtual switch**, nếu cần gloss: `(bộ chuyển mạch ảo)` | công tắc ảo; bo mạch chuyển mạch ảo | "công tắc" = light switch (sai hoàn toàn); "bo mạch" = circuit board (cũng sai). Switch mạng = thiết bị chuyển mạch, không phải công tắc hay bo mạch |
| service latency | **độ trễ dịch vụ (service latency)** | độ trễ của dịch vụ; độ trễ hệ thống | Chốt theo cách dùng ở ch.3 §"Các metrics dịch vụ khác". Dùng "độ trễ" chung cho latency ở chỗ khác là ổn, nhưng thuật ngữ đầy đủ cố định thứ tự gloss: tiếng Việt trước, `(service latency)` sau |
| reverse proxy | **reverse proxy**, nếu cần gloss: `(proxy ngược)` | proxy đảo; máy đại diện ngược | Giữ nguyên "reverse proxy" là chuẩn; chỉ gloss `(proxy ngược)` khi cần. "proxy đảo" nghe calque, tránh |
| explicit (risk-taking: "explicit, thoughtful risktaking") | **rõ ràng / cụ thể** | chủ động | "chủ động" = proactive, SAI cho "explicit" (rõ ràng, nói thẳng ra). Lặp lỗi ở 2 dòng ch.3 (L26, L115) — đã sửa |
| thoughtful (risk-taking) | **có suy xét / cân nhắc** | chủ động | "chủ động" = initiative, SAI cho "thoughtful" (cân nhắc, có suy xét) |
| slack capacity | **slack capacity** (giữ nguyên), nếu cần gloss: `(năng lực dự phòng)` | — | Jargon SRE (năng lực dự phòng/nhàn rỗi) |
| product velocity | **product velocity** (giữ nguyên), nếu cần gloss: `(tốc độ sản phẩm)` | — | Jargon SRE |
| continuum (risk) | **phổ (continuum)** | liên tục | Bị lệch ngay trong ch.3: dòng đầu dịch đúng "phổ", 1 dòng sau lại đổi thành "liên tục rủi ro" — soát cả trong-file, không chỉ cross-file |
| forcing function | **yếu tố ép buộc (forcing function)** | hàm ép buộc | "hàm" = function (toán học), SAI cho "forcing function" (cơ chế/yếu tố buộc hành động). Calque điển hình — đã sửa ở ch.4 L167 |
| loosely coupled (distributed systems) | **có liên kết lỏng lẻo** | liên kết lỏng lẻo (thiếu trợ từ "có") | Cụm SRE phổ biến (loose coupling). Cần trợ từ "có" cho đúng cú pháp VN — đã sửa ở ch.4 L50 |
| user studies | **nghiên cứu về người dùng (user studies)** | — | Thuật ngữ UX, gloss lần đầu. Đã dùng ở ch.4 L96 |
| interrupt / interrupt-driven | **sự gián đoạn (interrupt)** / **xuất phát từ sự gián đoạn (interrupt-driven)** | — | Ch.29 "Dealing with Interrupts" sẽ dùng lại. "xuất phát từ" thay cho "được dẫn dắt bởi" (bị động kiểu Anh) |
| firefighting | **chữa cháy (firefighting)** | — | Jargon SRE phổ biến, dùng ẩn dụ "chữa cháy" cho xử lý sự cố khẩn cấp |
| grungy work | **công việc lặt vặt bẩn thỉu** | — | Đặc trưng ch.5, dùng nhất quán trong file |
| overhead | **overhead (công việc phụ)** | — | Giữ EN + gloss, dùng nhất quán |
| feature velocity | **feature velocity** (giữ nguyên), nếu cần gloss: `(tốc độ tính năng)` | Tốc độ tính năng | Ghép với product velocity (đã có). Giữ EN cho nhất quán — "velocity" là jargon agile |
| sublinearly | **dưới tuyến tính (sublinearly)** | — | Ghép với "tuyến tính" (linear) |
| throughput | **thông lượng (throughput)** | lưu lượng, băng thông | "lưu lượng" đang dùng cho traffic — 1 từ Việt cho 2 khái niệm khác nhau gây va chạm. Phát hiện ở ch.4 (3 chỗ dùng "lưu lượng" cho throughput) — đã sửa. "băng thông" = bandwidth (khái niệm khác: dung lượng kênh truyền tối đa, không phải thông lượng thực đạt được) — phát hiện ở ch.19 (2 chỗ) — đã sửa |
| hermetic (build) | **hermetic** (giữ nguyên) | hấp thụ | "hấp thụ" = absorb, SAI cho "hermetic build" (build tự chứa, không phụ thuộc môi trường). Calque điển hình — đã sửa ở ch.8 (4 chỗ) |
| cherry pick (release) | **cherry pick** (giữ nguyên) | chốt quả | "chốt quả" = calque dịch nghĩa đen từng từ. Jargon git phổ biến — giữ EN. Đã sửa ở ch.8 L42 |
| force multiplier | **bộ khuếch đại lực lượng** | — | Jargon quân sự dùng trong kỹ thuật. Đã dùng ở ch.7 L16 |
| panacea | **panacea (thuốc chữa vạn năng)** | — | Từ Latinh, gloss lần đầu |
| turnup / turndown | **khởi động (turnup)** / **tắt (turndown)** | — | Jargon SRE cho việc bật/tắt cluster. Dùng ~15 chỗ ở ch.7 |
| bin-packing | **bin-packing** (giữ nguyên) | gói chặt | Jargon thuật toán. Giữ EN |
| glue logic | **logic keo** | — | Jargon kỹ thuật cho code kết nối các phần |
| bit rot | **bit rot (mục nát bit)** | mục nát dữ liệu | "mục nát bit" = bit rot (lỗi byte theo thời gian). Khác "mục nát dữ liệu" (data rot) |
| proof of concept | **proof of concept** (giữ nguyên) | bằng chứng khái niệm | Jargon kỹ thuật. Giữ EN |
| drain (traffic) | **rút traffic (draining)** | — | Jargon SRE cho việc rút dần traffic khỏi 1 service |
| warehouse-scale computer | **máy tính quy mô kho** | — | Jargon Google (warehouse-scale) |
| segfault | **segfault (lỗi phân đoạn bộ nhớ)** | lỗi trỏ; lỗi trỏ riêng | "segfault" = segmentation fault (lỗi truy cập vùng nhớ không hợp lệ), không phải "lỗi trỏ" (pointer error — khái niệm khác) |
| audit trail | **vết kiểm toán (audit trail)** | vết kiểm toán đơn | Gloss nhất quán: VN + (EN) |
| rate limiting | **rate limiting** (giữ nguyên) | hạn chế tỷ lệ | Jargon kỹ thuật. Giữ EN |
| self-introspection | **tự nội phản (self-introspection)** | tự kiểm tra | "tự nội phản" = introspection (kiểm tra bên trong). Dùng ở ch.7 |
| no-ops | **no-ops (thao tác không làm gì)** | — | Jargon kỹ thuật (no-operation) |
| leaky abstraction | **trừu tượng rò rỉ (leaky abstraction)** | — | Jargon kỹ thuật |
| hub-and-spoke | **hub-and-spoke (trục và nan hoa)** | — | Jargon kiến trúc |
| codebase | **codebase (kho mã nguồn)** | kho code | Gloss nhất quán: EN + (VN) |
| colocation (colo) | **colocation** (giữ nguyên), gloss: `(đặt cùng chỗ)` | đặt cùng chỗ (không có EN) | Jargon kỹ thuật. Giữ EN + gloss VN |
| mainline | **mainline** (giữ nguyên) | — | Nhánh chính của cây mã nguồn (trunk), nơi các thay đổi được commit trực tiếp; các dự án production KHÔNG phát hành trực tiếp từ đây (release cắt nhánh riêng). Giữ EN vì "nhánh chính"/"main branch" dễ gây hiểu nhầm là nhánh release |
| build target | **mục tiêu build** | — | Jargon build system |
| build label / build ID | **build label / build ID** (giữ nguyên) | — | Jargon build system. Giữ EN |
| canary deployment | **triển khai canary** | — | Jargon release engineering |
| skew (config drift) | **sự lệch (skew)** | lệch (thiếu "sự") | Gloss nhất quán: VN + (EN) |
| gated operations | **các thao tác cần phê duyệt** | các thao tác bị chặn | "gated" = cần phê duyệt (gate = cổng kiểm soát), không phải "bị chặn". Đã sửa ở ch.8 L46 |
| white-box / black-box monitoring | **giám sát hộp trắng / giám sát hộp đen** | — | Ch.10. White-box = kiểm tra trạng thái nội bộ target; black-box = nhìn từ phía người dùng. Gloss nhất quán VN |
| time-series arena | **vùng arena chuỗi thời gian** | arena thời gian (thiếu "chuỗi") | Ch.10. Khối bộ nhớ cố định lưu chuỗi thời gian. Giữ "arena" là thuật ngữ kỹ thuật (giữ EN) |
| labelset | **labelset (tập nhãn)** | tập nhãn (thiếu EN) | Ch.10. Tập hợp label mô tả 1 chuỗi thời gian. Giữ EN + gloss VN |
| vector (ch.10: lát cắt ma trận điểm dữ liệu) | **vector (giữ nguyên)** | dãy số; mảng giá trị | Ch.10. "vector" trong ngữ cảnh chuỗi thời gian = lát cắt/mặt cắt ngang của ma trận nhiều chiều. Giữ EN (jargon đại số tuyến tính + monitoring) |
| horizon (tầm nhìn — lượng dữ liệu có thể truy vấn trong RAM) | **horizon (tầm nhìn)** | tầm nhìn (thiếu EN) | Ch.10. Khoảng thời gian giữa entry mới nhất và cũ nhất trong arena. Giữ EN + gloss VN |
| counter / gauge (metrics) | **counter (bộ đếm) / gauge (thước đo)** | — | Ch.10. Counter = đơn điệu không giảm (chỉ tăng); gauge = mang bất kỳ giá trị nào. Gloss VN lần đầu |
| in lockstep (đồng bộ nhịp với peers) | **chạy cùng nhịp (in lockstep)** | đồng bộ (thiếu "nhịp"); song song đều | Ch.10. "in lockstep" = in step together (cùng nhịp), KHÔNG phải "đồng bộ" chung chung. "Không in lockstep" = "không chạy cùng nhịp". Đã sửa ở ch.10 L74 (GĐ6) — "đồng bộ" imprecise, "cùng nhịp" chính xác hơn |
| flap (alert toggling state nhanh) | **dao động (flap)** | nhấp nháy; rung lắc | Ch.10. Alert "flap" = chuyển trạng thái bật/tắt nhanh liên tục (false alert). "dao động" = oscilate (đúng nghĩa kỹ thuật) |
| fire (alert) | **fire (kích hoạt cảnh báo)** | bắn; nổ | Ch.10. Alert "fire" = cảnh báo được kích hoạt (gửi ra). Không phải "bắn/nổ" nghĩa đen. Giữ EN "fire" là chuẩn SRE |
| pending (alert state) | **pending (đang chờ)** | chờ duyệt | Ch.10. Alert "pending" = ở trạng thái chờ (chưa fire). Khác "pending approval" |
| fan-in / fan-out | **fan-in (hội tụ) / fan-out (tán ra)** | — | Ch.10. Fan-in = nhiều nguồn về 1 điểm; fan-out = 1 điểm tỏa ra nhiều đích. Gloss VN |
| Alertmanager | **Alertmanager** (giữ nguyên) | Quản lý Cảnh báo | Ch.10. Tên riêng của dịch vụ (giống Borgmon, TSDB). Giữ EN là tên riêng, không dịch |
| Prober | **Prober** (giữ nguyên) | Máy dò | Ch.10. Tên riêng của công cụ. Giữ EN là tên riêng |
| TSDB (Time-Series Database) | **TSDB (CSDL chuỗi thời gian)** | CSDL thời gian (thiếu "chuỗi") | Ch.10. CSDL lưu chuỗi thời gian. Giữ EN acronym + gloss VN |
| in-memory database | **cơ sở dữ liệu trong bộ nhớ (in-memory database)** | CSDL RAM | Ch.10. Gloss nhất quán VN + (EN) |
| checkpoint (disk) | **checkpoint (ghi điểm kiểm tra)** | lưu điểm kiểm tra; ghi điểm | Ch.10. "checkpoint" = ghi trạng thái ra disk định kỳ. Giữ EN + gloss VN |
| garbage collector | **bộ thu gom rác (garbage collector)** | bộ dọn rác; bộ thu gom | Ch.10. GC = thu gom bộ nhớ. Giữ EN + gloss VN |
| service discovery | **service discovery (khám phá dịch vụ)** | khám phá dịch vụ (thiếu EN) | Ch.10. Giữ EN + gloss VN. Jargon hạ tầng |
| name resolution | **phân giải tên (name resolution)** | — | Ch.10. Gloss VN |
| subprocess | **subprocess (tiến trình con)** | con tiến trình | "tiến trình con" là thứ tự chuẩn trong tiếng Việt (danh từ chính trước, tính từ bổ nghĩa sau — như "quy trình con", "hàm con"); "con tiến trình" bị đảo ngược, đọc lạ. Ch.10 — LẶP LẠI LẦN 2 (L41, L172) trong đợt rà soát 2026-08-28, đã sửa |
| drill down (vào dữ liệu) | **đào sâu (drill down)** | đào xuống; đi sâu | Ch.10. "drill down" = xem chi tiết hơn 1 phần của dữ liệu. Jargon phân tích dữ liệu |
| shared fate | **shared fate (số phận chung)** | chung số phận (thiếu EN) | Ch.10. Nhóm component có "shared fate" = hỏng cùng nhau. Giữ EN + gloss VN |
| relabeling | **relabeling (dán nhãn lại)** | dán nhãn lại (thiếu EN) | Ch.10. Giữ EN + gloss VN |
| magic number (con số ma thuật) | **con số ma thuật (magic number)** | — | Ch.10. Gloss VN lần đầu. "magic number" = hằng số chọn theo kinh nghiệm, không có công thức chính xác |
| counter reset | **đặt lại counter (counter reset)** | — | Ch.10. Jargon monitoring. "counter reset" = counter bị đặt lại về 0 |
| corner cases | **trường hợp khó (corner cases)** | trường hợp góc; trường hợp biên | Ch.10. Jargon kỹ thuật. "corner cases" = các trường hợp cực đoan/hiếm gặp |
| untyped | **untyped (không có kiểu)** | không kiểu (thiếu EN) | Ch.10. Jargon kiểu dữ liệu. Giữ EN + gloss VN |
| footprint (service) | **footprint (dấu chân) dịch vụ** | dấu chân dịch vụ (thiếu EN) | Ch.10. "footprint" = phạm vi/chiều phủ của 1 dịch vụ trên hạ tầng. Giữ EN + gloss VN |
| Cambrian Explosion (giám sát) | **Sự bùng nổ Cambrian (Cambrian Explosion)** | bùng nổ Cambrian (thiếu "Sự") | Ch.10. Ẩn dụ sinh học (bùng nổ đa dạng sinh học kỷ Cambrian) dùng cho sự bùng nổ công cụ giám sát. Giữ EN + gloss VN |
| threadpool | **threadpool (bể luồng)** | — | Ch.10. Giữ EN + gloss VN. Jargon hệ điều hành |
| bottleneck | **nút thắt cổ chai (bottleneck)** | — | Ch.10. Jargon kỹ thuật. Gloss VN lần đầu |
| single point of failure | **điểm thất bại duy nhất (single point of failure)** | — | Ch.10. Jargon kỹ thuật. Gloss VN lần đầu |
| histogram | **histogram (đồ thị phân bố tần số)** | — | Ch.10. Giữ EN + gloss VN. Jargon thống kê/data viz |
| homed (be homed in) | **được đặt (homed) trong** | — | Ch.10. "be homed in" = được đặt/có trụ ở (1 datacenter). Giữ EN + gloss VN lần đầu |
| follow the sun | **follow the sun (theo mặt trời)** | chase the sun (đuổi theo mặt trời) | Ch.11. Thuật ngữ SRE cho mô hình on-call phân bố theo múi giờ để tránh ca đêm. EN gốc = "follow the sun" (KHÔNG phải "chase the sun") |
| regression | **lỗi hồi quy (regression)** | lỗi thoái hóa | Ch.13. Jargon QA. "hồi quy" = regression (lỗi quay lại), không phải "thoái hóa" (degradation) |
| panic room | **phòng khẩn cấp (panic room)** | phòng an toàn | Ch.13. Phòng có đường truy cập dự phòng vào production khi network bị ngắt |
| game out | **kịch bản hóa (game out)** | mô phỏng | Ch.14. Diễn tập/kịch bản hóa phản ứng trước khi incident xảy ra. "mô phỏng"=simulate, sai |
| telemetry | **telemetry** (giữ nguyên) | truyền liệu | Ch.12. Dữ liệu đo xa/giám sát. "truyền liệu"=telemedicine, sai. CHƯA CHẮC gloss VN |
| span (tracing) | **span** (giữ nguyên) | màn | Ch.12. Khoảng thời gian 1 operation trong distributed tracing. "màn"=calque sai |
| profiling | **profiling (đo hiệu năng)** | đo hồ sơ | Ch.12. Đo hiệu năng hệ thống. "đo hồ sơ"=calque sai |
| fire-and-forget | **bắn rồi quên (fire-and-forget)** | — | Ch.24. Idiom: gửi đi không cần phản hồi. Đã gloss trong file |
| hot spare | **hot spare (bản sao dự phòng nóng)** | — | Ch.24. Máy dự phòng sẵn sàng nhận traffic ngay |
| Moiré load pattern | **mẫu tải Moiré (Moiré load pattern)** | — | Ch.25._pattern tải chồng chéo khi nhiều pipeline chạy đồng thời |
| sinkholing | **sinkholing (bắt bẫy)** | — | Ch.20-21. Task không khỏe mạnh bắt bẫy traffic thay vì trả lỗi |
| debug | **debug (gỡ lỗi)** | xử lý lỗi | Ch.12, 22. "debug" = gỡ lỗi, KHÔNG phải "xử lý lỗi" (error handling) |
| mixed-integer program | **chương trình nguyên hỗn hợp (mixed-integer program)** | chương trình nguyên tố hỗn hợp | Ch.18. "integer"=nguyên (số nguyên), KHÔNG phải "nguyên tố" (element). Lặp 3 chỗ |
| over-perform | **over-perform (vận hành vượt mức)** | quá tải | Ch.16. "over-performing"=chạy vượt mức cần (dư sức), KHÔNG phải "quá tải" (overload) |
| MTBF | **MTBF (Mean Time Between Failures — thời gian trung bình giữa các lần hỏng)** | — | Ch.17. Jargon độ tin cậy |
| break-glass | **break-glass (phá kính)** | — | Ch.17. Cơ chế vượt an toàn khẩn cấp. Giữ EN + gloss |
| choke point | **choke point (điểm nghẽn)** | — | Ch.13. Điểm nghẽn trong hệ thống |
| sister office | **sister office (văn phòng chị em)** | văn phòng bên | Ch.14. Văn phòng đối tác ở địa điểm khác |
| melt down | **tan rã (melt down)** | sụp | Ch.22. "melt down" = tan rã/đổ vỡ do quá tải, khác "crash" (sập) |
| vice president (VP) | **phó chủ tịch (vice president)** | phó tổng giám đốc | Ch.27. "VP" = Vice President, không phải "phó tổng giám đốc" (deputy director) |
| soft deletion | **xóa mềm (soft deletion)** | — | Ch.26. Dữ liệu bị xóa đánh dấu chứ không xóa thực. Dùng nhất quán 20+ chỗ |
| silver bullet | **giải pháp vạn năng (silver bullet)** | — | Ch.26. Idiom: giải pháp kỳ diệu/all-in-one. "giải pháp vạn năng" là dịch đúng |
| combinatorial explosion | **bùng nổ tổ hợp (combinatorial explosion)** | — | Ch.27. Số tổ hợp tăng theo cấp số nhân khi thêm tính năng |
| hot standby | **hot standby (dự phòng nóng)** | đ sẵn nóng (typo) | Ch.22. Hệ thống sẵn sàng nhận traffic ngay, giống hot spare |

## Nguyên tắc giữ nguyên tiếng Anh (không ép dịch)

Các thuật ngữ sau LUÔN giữ nguyên tiếng Anh, không dịch, không chú thích lặp lại sau lần đầu:
SRE, SLI, SLO, SLA, error budget, toil, on-call, incident, postmortem, outage, canary, quorum,
rollout, rollback, sharding, RPC, API, cache, proxy, load balancer, MTTR, MTTF, thread, deadlock,
race condition, canary, playbook, flaky, idempotent, backend, frontend, binpack, crashloop,
replication, eventual consistency, indirection, load shifting.

Quy tắc chọn dịch vs giữ nguyên cho thuật ngữ MỚI (chưa có trong bảng):
1. Nếu thuật ngữ đã phổ biến trong cộng đồng dev/SRE Việt Nam (thường thấy trong blog kỹ thuật,
   Slack/Discord dev VN) → **giữ nguyên tiếng Anh**.
2. Nếu chỉ dịch được bằng cách ghép từng chữ qua từ điển và nghe kỳ quặc khi đọc thành câu
   (tự kiểm bằng cách đọc to) → **giữ nguyên tiếng Anh**, không cố dịch cho bằng được.
3. Chỉ dịch sang tiếng Việt khi có 1 cụm tự nhiên, ai đọc cũng hiểu ngay, không cần suy luận ngược
   lại tiếng Anh mới hiểu (vd: "root cause" → "nguyên nhân gốc rễ" — ổn vì tự nhiên).

## Lịch sử thay đổi

- 2026-08-26: Khởi tạo, seed từ kết quả rà soát 48 file (phát hiện qua workflow đa-agent, xem hội
  thoại "rà soát dịch thuật" cùng ngày). 14 thuật ngữ đầu tiên.
- 2026-08-26: `part-1-introduction/01-introduction.md` — chạy thử nghiệm đầy đủ quy trình
  `/translate-book` (Giai đoạn 4 áp glossary, Giai đoạn 5 làm mượt văn phong, Giai đoạn 6 semantic
  check) lần đầu tiên. Sửa 1 lệch thứ tự chú thích glossary, 1 lỗi nghĩa ("bản chất có lỗi" →
  "vốn dĩ có lỗi"), 6 câu calque/lủng củng còn sót từ lượt review văn phong trước đó. Chương 1 hiện
  coi là **hoàn tất** (baseline để nhân rộng cho các chương còn lại).
- 2026-08-26: `part-1-introduction/02-production-environment.md` — chạy đầy đủ quy trình. Sửa 1 lỗi
  glossary (graceful degradation "suy giảm từ từ" → "suy giảm nhẹ nhàng"), ~25 câu calque/lủng củng ở
  Giai đoạn 5. Qwen tự đề xuất đổi frontend/backend/client/server trong câu định nghĩa vai trò (giữ lại,
  hợp lý) — **nhưng Qwen chạy trước khi rào chắn "đừng sửa thuật ngữ đã chuẩn hóa" được thêm vào command,
  nên KHÔNG áp dụng rào chắn đó cho lượt này**. Soát tay lại sau: phát hiện 2 lỗi bịa định nghĩa
  (`bisection bandwidth→"băng thông đường chéo"`, `virtual switch→"công tắc ảo"`) — cả 2 đã revert. Thêm
  rào chắn thứ 2 vào command (cấm bịa định nghĩa khi không chắc nghĩa gốc) + note vận hành (sửa command
  giữa lúc agent chạy không có tác dụng ngay).
- 2026-08-26: `part-2-principles/03-embracing-risk.md` — chạy đủ Giai đoạn 5 (văn phong) + Giai đoạn 6
  (semantic). Giai đoạn 5 sửa 13 chỗ (L1: 4 — bị động "bởi"→chủ động, xóa đại từ chỉ định thừa, tách câu
  dài, gộp 3 câu "Chúng tôi..." lặp; L2: 6 — mệnh đề chêm mồ côi, khung song song, chỉ định mơ hồ; L3: 3 —
  calque "captured by"→chủ động, "externalize"→"chuyển sang phía client", danh hóa "sự thất bại"). Giai đoạn
  6 sửa 3 chỗ lệch nghĩa: `explicit→chủ động`→`rõ ràng` (L26, L115 — "explicit"=rõ ràng, không phải
  proactive), `eventually consistent` bỏ gloss tiếng Việt giữ EN thuần (L143, khớp mục giữ nguyên EN). Không
  lỗi chính tả. Giai đoạn 3 merge: +4 thuật ngữ (explicit, thoughtful, slack capacity, product velocity),
  1 xung đột đã xử (explicit/thoughtful bị dịch "chủ động" ở 2 dòng — chọn "rõ ràng"/"có suy xét"). Giai đoạn
  4 áp ngược: rà 14 thuật ngữ đã chốt trong file, không phát hiện biến thể SAI cần thay.
- 2026-08-26: `part-2-principles/04-service-level-objectives.md` — chạy đủ Giai đoạn 5 (văn phong) +
  Giai đoạn 6 (semantic). Giai đoạn 5 sửa 6 chỗ (L1: 3 — bị động "bởi"→chủ động "do", danh hóa "việc dịch
  vụ chậm"→"chuyện dịch vụ chậm", đổi nhịp "X bằng Y"→"dùng X để quy định Y"; L2: 2 — đưa điều kiện lên đầu
  câu L16, bị động "được chọn"→chủ động "do bạn chọn" L36; L3: 1 — calque "the service being slow"). Giai
  đoạn 6 sửa 5 chỗ: 2 lỗi nghĩa thật (`đại diện (proxy)`→`chỉ số đại diện (proxy)` L28 — khớp glossary;
  `forcing function (hàm ép buộc)`→`(yếu tố ép buộc)` L167 — "hàm"=function toán học, calque sai) + 3 chuẩn
  hóa (`giữ lại`→`lưu giữ` L30, `liên kết lỏng lẻo`→`có liên kết lỏng lẻo` L50, thêm gloss `user studies`
  L96). GĐ4 pre-flight sửa 1 SAI: client/server gloss "khách hàng"/"máy chủ" L28 (chỉ dùng khi định nghĩa
vai trò, chỗ này là dùng thông thường → giữ EN). Giai đoạn 3 merge: +3 thuật ngữ (forcing function,
   loosely coupled, user studies), không có xung đột. Giai đoạn 4 áp ngược: rà 30 thuật ngữ đã chốt, chỉ còn
   L28 (đã sửa ở pre-flight) — file sạch.
- 2026-08-26: `part-2-principles/05-eliminating-toil.md` — chạy đủ Giai đoạn 5 (văn phong) + Giai đoạn 6
   (semantic). Giai đoạn 5 sửa 10 chỗ (L1: 7 — bị động "được...bởi"→chủ động L38, L68, L130; calque "lý
   tưởng"→"tốt" L46; "Nhất quán với"→"Đúng với" L62; xóa "một cách" L134; L2: 3 — "Sở thích về"→"Loại
   công việc nào" L20, "2 trong mỗi 6 tuần"→"2 trên 6 tuần" L60, "được tiên phong bởi"→"do...tiên phong"
   L130; L3: 0). Giai đoạn 6: 2 chỗ nghi vấn L50 ("công bố mục tiêu", "hiệu ứng thứ hai") — đối chiếu
   source xác nhận ĐÚNG, revert lại. Không lỗi nghĩa, không lỗi chính tả. Giai đoạn 3 merge: +6 thuật ngữ
   (interrupt/interrupt-driven, firefighting, grungy work, overhead, feature velocity, sublinearly),
   không có xung đột. Giai đoạn 4 áp ngược: rà 30 thuật ngữ đã chốt trong file, không phát hiện biến thể
   SAI cần thay.
- 2026-08-26: QA thủ công (agent Fable, độc lập với pipeline) trên ch.4 + ch.5 trước khi push, vì Giai
  đoạn 4/6 của Qwen tự báo "sạch"/"đã verify" nhưng thực tế còn sót lỗi — không nên tin self-report mà
  cần spot-check tay. Phát hiện & sửa: (1) ch.4 L30 lỗi factual nghiêm trọng — "hai phẩy năm nines" cho
  99.95% (đúng phải là "ba phẩy rưỡi nines", vì 99%=2 nines, 99.9%=3, 99.95%=3.5); (2) ch.5 L112 tự vi
  phạm glossary vừa chốt (dịch "Tốc độ tính năng" dù entry feature velocity liệt kê đúng cụm này là biến
  thể SAI); (3) ch.4 L38 gloss frontend ở câu dùng thông thường (vi phạm rule frontend/backend); (4) ch.4
  L167 đảo ngược thứ tự gloss forcing function; (5) ch.4 L77 "per se (vốn dĩ)" sai nghĩa → "(tự thân nó)";
  (6) ch.4 L81 "instrument (lắp thiết bị đo)" hiểu nhầm sang phần cứng → "(nhúng phép đo)"; (7) throughput
  dịch "lưu lượng" ở ch.4 (3 chỗ) trùng từ với "traffic" — thêm entry throughput mới, đổi thành "thông
lượng"; (8) ch.5 L50 "hiệu ứng thứ hai" thiếu chữ "bậc" (second-order); (9) ch.5 L94, L120 calque nhẹ;
   (10) ch.5 L44 heading dùng bold lệch format `####` so với các heading cùng cấp khác trong file.
- 2026-08-27: `part-2-principles/06-monitoring-distributed-systems.md` — fan out 1 agent. GĐ4: 1 sửa
   (loosely coupled L188). GĐ5: ~25 sửa (bị động→chủ động, xóa "của nó", "một cách" thừa, "xử lý thế
   nào"→"cách xử lý", tách câu dài, đưa điều kiện lên đầu, blind re-translation cho các câu calque).
   GĐ6: 3 sửa (bỏ gloss thừa "backend (phía sau)" L130, bỏ gloss sai "stack (tòa)" L219, "bạn nên ưa"→
   "bạn nên ưu tiên" L245 — "ưa"=like, "ưu tiên"=prioritize, khớp "favor" EN). Không thuật ngữ mới.
- 2026-08-27: `part-2-principles/07-automation-at-google.md` — fan out 1 agent. GĐ4: 3 sửa (switch "bo
   mạch chuyển mạch"→"bộ chuyển mạch", client (khách hàng)→khách, machine "máy"→"máy tính" L24). GĐ5:
   ~33 sửa (xóa "một cách", bị động→chủ động, giảm danh hóa "sự", tách câu dài, đưa điều kiện lên đầu,
   đổi khung câu song song, sửa lỗi trùng từ do edit). GĐ6: ~14 sửa, 1 lỗi nghĩa nghiêm trọng: sshd
   "thư mục đăng nhập an toàn"→"daemon shell an toàn" (sshd = Secure Shell daemon). +18 thuật ngữ mới
   (force multiplier, panacea, turnup/turndown, bin-packing, glue logic, bit rot, proof of concept,
   drain, warehouse-scale computer, segfault, audit trail, rate limiting, self-introspection, no-ops,
   leaky abstraction, hub-and-spoke, codebase, colocation).
- 2026-08-27: `part-2-principles/08-release-engineering.md` — fan out 1 agent. GĐ4: 0 (file sạch). GĐ5:
   9 sửa (bị động "bởi"→chủ động L20/L28/L109, "rằng"→mật điểm L36, "một cách"→"đến mức" L119, khung
   song song L32, câu dài L73, trạng ngữ mồ côi "Phần lớn" L143, calque "scale"→"có khả năng mở rộng"
   L59). GĐ6: 7 sửa, 4 lỗi nghĩa: `hấp thụ`→`hermetic` (4 chỗ — "hấp thụ"=absorb, SAI cho hermetic
   build), `cherry picking (chốt quả)`→`cherry picking` (calque sai), `bị chặn`→`cần phê duyệt`
   (L46 — "gated"=cần phê duyệt), `blueprints (bản thảo)`→`blueprints (bản thiết kế)` (L87 — "bản thảo"=
   draft, SAI cho blueprint config file). +8 thuật ngữ mới (hermetic, cherry pick, blueprint, mainline,
   build target, build label/ID, canary deployment, skew, gated operations).
- 2026-08-27: `part-2-principles/09-simplicity.md` — fan out 1 agent. GĐ4: 2 sửa (loose coupling→liên
   kết lỏng lẻo L53, 5 heading bỏ art "The/A/I/My"). GĐ5: 22 sửa (tách 3 câu "Nếu...thì..." L16, "Tôi
   đã thường"→"Tôi thường" L20, xóa "của chúng" L20/L26, "được nhìn thấy bởi"→"bị người dùng nhìn thấy"
   L20, gộp 2 câu song song L22, "Mang góc nhìn"→"Giữ góc nhìn" L43, "Trong khi"→bỏ L43/L55, "người
   tiêu dùng"→"người dùng" L49 — "consumer" trong API = người dùng, "Đo lường và hiểu... dễ dàng hơn
   nhiều" L63, "Thay vào đó"→", mà đang" L67, "Được tạo ra bởi"→"do...tạo ra" L71, blind re-translation
   L20/L39/L53/L67). GĐ6: 0. Không thuật ngữ mới.
- 2026-08-27: **Part 2 hoàn tất** (ch.3-9). Tổng cộng 4 chương fan out song song, ~115 sửa (GĐ4: 6, GĐ5:
   ~89, GĐ6: ~24). Merge +26 thuật ngữ mới vào glossary (hermetic, cherry pick, force multiplier, panacea,
   turnup/turndown, bin-packing, glue logic, bit rot, proof of concept, drain, warehouse-scale computer,
   segfault, audit trail, rate limiting, self-introspection, no-ops, leaky abstraction, hub-and-spoke,
   codebase, colocation, mainline, build target, build label/ID, canary deployment, skew, gated
   operations). 2 xung đột đã xử: sshd (ch.7) — "daemon shell an toàn" đúng source; hermetic (ch.8) —
   "hấp thụ" SAI, giữ EN. Glossary hiện có 52 thuật ngữ.
- 2026-08-27: QA thủ công (agent Fable, độc lập với pipeline) trên ch.6-9 trước khi push — pipeline tự
  báo GĐ4/6 "sạch"/"đã verify" 3 lỗi (sshd, hermetic, cherry picking) đều landed đúng, số liệu kỹ thuật
  sạch, nhưng vẫn còn ~10 lỗi nghĩa/gloss bịa bị bỏ sót. Phát hiện & sửa: (1) ch.7 "Hệ thống Quản trị
  viên" đảo ngược nghĩa (phải là "Quản trị viên Hệ thống"); (2) ch.7 backbone "(tủy sống)" — gloss giải
  phẫu học bịa cho mạng, đổi thành "(mạng trục)"; (3) ch.7 "toàn bộ stack (tòa)" và Container "(hộp)" —
  gloss thừa/sai, bỏ; (4) ch.6 key-value "(giá trị-chìa)" → "(khóa–giá trị)"; (5) ch.6 heading "Vạch về
  Lâu dài" vô nghĩa → "Về Lâu dài"; (6) retrofit dịch 2 kiểu đều sai (ch.7 "cài đặt bổ sung", ch.8 "cài
  đặt lại"=reinstall!) → thống nhất "trang bị bổ sung cho hệ thống sẵn có"; (7) "phép loại" thiếu chữ
  "suy" ở ch.7 + ch.9 (2 chỗ) → "phép loại suy"; (8) ch.8 `bad_quarto`/`first_folio`/`much_ado` — đây là
  tên vở kịch Shakespeare dùng làm flag name chơi chữ, KHÔNG phải nghĩa đen ("bản sao thứ tư tồi"/"bản
  thảo đầu tiên"/"nhiều ồn ào" là dịch sai bản chất) → bỏ gloss, giữ nguyên tên; (9) ch.9 calque "giới
  thiệu" cho introduce (bug/complexity) lặp ~7 chỗ → "gây ra"/"đưa vào"; (10) ch.6 heading trộn `####`
  và bold không nhất quán — fix. Sửa 2 entry glossary sai định nghĩa: segfault "(lỗi trỏ)" → "(lỗi phân
đoạn bộ nhớ)" (tự mâu thuẫn với chính cột Ghi chú cũ); mainline note "là nhánh phát hành chính" SAI →
   sửa thành mô tả đúng (nhánh chính/trunk, KHÔNG phải nhánh release).
- 2026-08-28: `part-3-practices/10-practical-alerting.md` (Ch.10) — chạy đủ GĐ1→GĐ8. GĐ1 seed +28
    thuật ngữ ch.10 (white-box/black-box monitoring, time-series arena, labelset, vector, horizon,
    counter/gauge, in lockstep, flap, fire, pending, fan-in/fan-out, Alertmanager, Prober, TSDB,
    in-memory database, checkpoint, garbage collector, service discovery, name resolution, subprocess,
    drill down, shared fate, relabeling, magic number, counter reset, corner cases, untyped, footprint,
    Cambrian Explosion). GĐ2 (1 agent): 30 sửa (9 vi phạm glossary — bỏ gloss sai Alertmanager "Quản lý
    Cảnh báo"/Prober "Máy dò"/frontend "mặt trước"/client "khách hàng", fire "bắn/nổ"→"kích hoạt cảnh
    báo", chuẩn hóa thứ tự gloss TSDB/footprint/service discovery; 14 lỗi nghĩa/calque; 7 chuẩn hóa
    gloss fire). GĐ3 merge: +5 thuật ngữ (threadpool, bottleneck, single point of failure, histogram,
    homed) — tất cả nhãn 'Đã biết'/'Đã tra cứu', KHÔNG có 'CHƯA CHẮC', 0 xung đột. GĐ4: 0 (file sạch).
    GĐ5 (văn phong, 1 agent): 22 sửa (L1: 7, L2: 6, L3: 9) — bị động "bởi"→chủ động, xóa "một cách",
    "thực hiện việc"→"thu thập", xóa "của nó" thừa, blind re-translation. GĐ6 (semantic, 1 agent): 2 sửa
    (L74 "đồng bộ (in lockstep)"→"chạy cùng nhịp (in lockstep)" — Mistranslation, "đồng bộ" imprecise;
    L322 thiếu "chúng" — Omission). Sửa entry in lockstep trong glossary: "đồng bộ"→"chạy cùng nhịp".
    GĐ7 QA độc lập (Claude Opus 5, model khác): 2 lỗi thật + 1 borderline — (L58 "biến map-valued như
    ví dụ"→"biến map-valued, ví dụ dưới đây" — Mistranslation; L347 "mã hóa hóa"→"mã hóa" — typo thừa
"hóa"; L37 "vốn là nền tảng"→"vốn nền tảng" — borderline, thừa "là"). GĐ7 verify tất cả số liệu
     khớp, không gloss bịa, không terminology drift, không entry glossary định nghĩa sai. GĐ8: áp 3 fix
     GĐ7. Ch.10 hoàn tất. Glossary hiện có 85 thuật ngữ.
- 2026-08-28: Ch.10 — rà soát lại GĐ5-6-7 (1 lượt review mới). GĐ5 (văn phong): 0 sửa — không còn trigger
  L1 (L345 "một cách" là false positive — cụm "a way" của tiếng Anh, không phải calque "một cách + adj";
  các "của nó/chúng/họ" đều là sở hữu từ đúng, không phải calque thừa). GĐ6 (semantic): 1 sửa L23 —
  "Yêu cầu duy trì gánh nặng bảo trì…ở mức hợp lý thấp cho các kỹ sư" → "Cần giữ gánh nặng bảo trì (maintenance
  burden) trên các kỹ sư phụ trách hệ thống ở mức thấp hợp lý" (Mistranslation+Omission: nguồn gốc = "the need
  to maintain a reasonably low maintenance burden **on the engineers**", bản cũ bỏ "on the engineers" và đọc
  ngược thành "yêu cầu duy trì gánh nặng"). Verify toàn bộ số liệu khớp nguồn (24 byte, 1 triệu, 12 giờ,
  17 GB, 0.01, 1/s, 0.15, "hai chu kỳ", "1 hoặc 10 giây", "1 phút"). GĐ7 (QA độc lập, agent khác): 2 sửa
  thật — (1) L41 + L172 terminology drift: "con tiến trình (subprocess)" → "subprocess (tiến trình con)"
  (vi phạm entry subprocess trong chính glossary — "con tiến trình" là biến thể SAI đã liệt kê, lặp lại lần 2
  ở ch.10); (2) L170 Mistranslation sắc thái: "necessary evils" → "điều ác cần thiết" (sai — "necessary evils"
  là idiom nghĩa TIÊU CỰC: phiền phức nhưng bắt buộc phải có, không phải biện minh) → "điều phiền phức nhưng
  bắt buộc phải có". CHƯA CHẮC đã xác minh: `http://webserver:80/varz` (L49) khớp bản in O'Reilly 2017 gốc,
  trang web đổi sang `https` sau này — không phải lỗi dịch, giữ nguyên. Số liệu/factual + gloss bịa +
  terminology drift + entry glossary: SẠCH. Ch.10 hoàn tất (lần 2). Glossary vẫn 85 thuật ngữ (không thêm mới).
- 2026-08-28: `part-3-practices/11-being-on-call.md` (Ch.11) — chạy đủ GĐ4→GĐ8 (file đã được dịch sẵn,
  không cần GĐ1-3). GĐ4: 0 (file sạch, glossary đúng). GĐ5 (văn phong): 7 sửa (L1: 2 — xóa "một cách" L96/L98;
  L2: 5 — bị động "bởi"→chủ động L34/L98, khung song song L134, "thì" L98, "Về lý tưởng" L112). GĐ6 (semantic,
  2 lượt): 4 sửa — (1) L58 "chase the sun"→"follow the sun" (Mistranslation — EN gốc = "follow the sun");
  (2) L67+L110 link ch.30 "Cài đặt một SRE"→"Đưa một SRE vào Đội" (Terminology drift — "embed"=đưa vào đội,
  không phải "cài đặt"=install); (3) L120 "cảnh báo paging"→"các trang (paging) cảnh báo" (Mistranslation nhẹ);
  (4) L134 thiếu chủ ngữ "Google's approach to on-call has enabled us"→bổ sung "cách tiếp cận on-call của
  Google cho phép chúng tôi" (Omission). GĐ7 (QA độc lập): 2 sửa — (1) L86 "rất dễ rơi vào thiên kiến"→
  "rất dễ bị cám dỗ thực hiện thiên kiến" (Mistranslation sắc thái — "extremely tempting"=cám dỗ, không phải
  "dễ rơi vào"); (2) L71 "Mực trần bù đắp là giới hạn"→"thể hiện giới hạn" (Omission — EN "represents, in
  practice, a limit"). 1 fix phụ: L98 "theo đuổi hợp lý"→bổ sung "thì việc...sẽ hữu ích". Tổng: 14 sửa.
  +1 thuật ngữ mới (follow the sun). Glossary hiện có 86 thuật ngữ.
- 2026-08-28: QA thủ công (agent Fable, độc lập) trên ch.11 trước khi push — 6 fix pipeline tự khai đều
  landed đúng, số liệu sạch, nhưng phát hiện thêm 7 lỗi thật, đáng chú ý: (1) chính fix GĐ6 của pipeline
  ở L120 ("cảnh báo paging"→"các trang (paging) cảnh báo") làm câu TỆ HƠN — "trang" là false-friend
  (trang sách/web), và L114 (2 chỗ) vẫn còn "cảnh báo paging" y hệt cụm bị coi là sai — sửa thống nhất về
  "page" giữ nguyên EN, khớp cách dùng L28/L32/L36/L65/L86; (2) L71 typo "Mực trần"→"Mức trần"; (3) L34
  rơi nghĩa so sánh "ưu tiên gần như mọi tác vụ khác"→"ưu tiên HƠN gần như mọi tác vụ khác" (EN "priority
  over"); (4) L130 từ garbled "định kích để"→"định biên sao cho"; (5) L54/L56/L61 "đội đơn vị"→"đội một
  vị trí" (single-site) — "đơn vị"=unit, sai nghĩa và lệch với "hai vị trí"/"đa vị trí" ngay cạnh; (6) L88
  va chạm nghĩa "thiếu cơ sở dữ liệu rõ ràng"→"không có dữ liệu rõ ràng làm cơ sở" (tránh đọc thành
  "database"); (7) L90 link relative path hỏng (`../../sre-book/...`) → đổi URL đầy đủ, khớp toàn bộ link
  khác trong file. Ch.11 hoàn tất.
- 2026-08-28: QA thủ công (agent độc lập) trên `00-front/foreword.md` + `00-front/preface.md` — 2 file
  dịch từ đợt đầu (trước khi có pipeline glossary), chưa qua GĐ1-8. Kết quả: sạch hơn dự kiến, chỉ 2 lỗi.
  (1) foreword.md L11: "hệ thống quản trị viên"→"quản trị viên hệ thống" — LẶP LẠI y hệt bug đã sửa ở
  ch.7 (2026-08-27, "Hệ thống Quản trị viên" đảo ngược nghĩa System Administrator); (2) preface.md L206:
  "manzana (phố)" → bỏ gloss "(phố)" — gloss bịa cho câu đùa cá nhân trong lời cảm tạ (manzana tiếng Tây
  Ban Nha = "quả táo"/nghĩa bóng "khu phố", không phải "phố"/street; bản gốc EN không chú giải gì). Không
  phát hiện: thuật ngữ sai so glossary, số liệu/trích dẫn sai, anchor/heading gãy, terminology drift với
  tên chương ở các file `##-*.md` khác (tên chương trong preface chỉ là link label giữ nguyên EN, không
  phải prose dịch nên không có rủi ro drift).
- 2026-08-28: Batch 1 ch.12-21, 24-25 (9 chương, 3 team parallel) — Bro làm trung tâm, 3 worker
  parallel (Team A: ch.12-14, Team B: ch.15-18, Team C: ch.19-21+24-25). GĐ4: 8 sửa (crowdsourcing
  "đám mây"→"đám đông", graceful degradation "êm ả"→"nhẹ nhàng"×2, agnostic "thờ ơ"→"trung lập
  công nghệ"×3, failure domain "miền thất bại"→giữ EN, đồng thuận phân tán→nhất trí phân tán,
  embarrassingly parallel "dễ chịu"→"hoàn toàn"). GĐ5: ~35 sửa (bị động "bởi"→chủ động, xóa "một cách",
  calque "máy móc nặng"→"công cụ hạng nặng", "bắn (fire)"→"fire (kích hoạt)", "động não (brainstorm)"→
  "tìm cách", "xối xả nghiền"→"nghiền", "mê mệt giấc ngủ"→"còn mơ ngủ", "đổ chuông"→"reo", "Trò chuyện
  Trung chuyển Internet"→bỏ gloss IRC, "nhà phát triển trực"→"developer on-call", "nhìn thấy được bởi"→
  "bị...nhìn thấy", "mô phỏng (game out)"→"kịch bản hóa", "ngỗng hoang"→"không đâu", "đo hồ sơ"→"đo
  hiệu năng", "đo lường (instrument)"→"gắn phép đo", "Cổng xấu"→bỏ gloss HTTP 502, "Nền tảng Mây"→bỏ
  gloss Cloud Platform, "hệ thống tệp cụm"→bỏ gloss, "không-production"→bỏ, "span (màn)"→bỏ "(màn)",
  "ping (bật)"→bỏ "(bật)", "instance (hồi)"→bỏ "(hồi)", "data race (đua dữ liệu)"→bỏ gloss, "chạy thử
  nghiệm (canary)"→"triển khai canary", "phòng an toàn (panic room)"→"phòng khẩn cấp", "bắn lặp"→"fire
  (kích hoạt) lặp", "lỗi thoái hóa (regression)"→"lỗi hồi quy", "xối xả nghiền"→"nghiền", "tương tác
  (interactive)"→"không tương tác (non-interactive)", "chuỗi phụ dependence"→"chuỗi phụ thuộc", "một
  cách êm ả"→"êm ả"×2, "tạo thành metric"→"là metric", "xác suất có thể"→"khả năng cao", "bầy đàn giông"
  →"hiệu ứng bầy đàn"×3, "thiếu tài nguyên"→"cạn tài nguyên", "nổi bật lên"→"chỉ ra", "giới thiệu giao
  diện"→"thêm giao diện", "nhầm lẫn cho rằng"→"nhầm tưởng rằng", "mô phỏng"→"kịch bản hóa", "độ bám
  dính CPU"→bỏ gloss, "mô-đun"→"module", "bot (robot)"→"bot", "đổ chuông"→"reo", "máy gọi trực"→"thiết
  bị gọi trực"). GĐ6: ~25 sửa (ch.12: SRE "Trang web"→bỏ, telemetry "truyền liệu"→bỏ, wild goose chase
  "ngỗng hoang"→"không đâu", debug "xử lý lỗi"→"gỡ lỗi", instrument "đo lường"→"gắn phép đo", HTTP 502
  "Cổng xấu"→bỏ, non-production "không-production"→bỏ, Profiling "đo hồ sơ"→"đo hiệu năng", Cloud
  Platform "Nền tảng Mây"→bỏ, instance "hồi"→bỏ, "máy móc nặng"→"công cụ hạng nặng", span "màn"→bỏ,
  cluster filesystem "hệ thống tệp cụm"→bỏ, "phải phải" typo→"phải", "ít gì thay thế"→"không có gì
  thay thế"; ch.13: "gọi (page)"→"page", "hoàn tác (roll back)"→"rollback", "động não (brainstorm)"→
  "tìm cách", "phòng an toàn (panic room)"→"phòng khẩn cấp", "bắn lặp"→"fire (kích hoạt) lặp", "canary
  (chạy thử nghiệm)"→"triển khai canary", "quy trình cài đặt lại"→"+thủ công", "xối xả nghiền"→"nghiền",
  "lỗi thoái hóa"→"lỗi hồi quy"; ch.14: "mô phỏng (game out)"→"kịch bản hóa", "đổ chuông"→"reo",
  "mô-đun"→"module", "hoàn tác (rollback)"→"rollback", "page"→"page bộ nhớ", "mê mệt giấc ngủ"→"còn mơ
  ngủ", "độ bám dính CPU"→bỏ, "Trò chuyện Trung chuyển Internet"→bỏ, "bắn (fire)"→"fire (kích hoạt)",
  "nhà phát triển trực"→"developer on-call", "nhìn thấy được bởi"→"bị...nhìn thấy", "bot (robot)"→"bot";
  ch.16: "quá tải"→"over-perform"; ch.17: "gây thù địch"→"mang tính thù địch", "phiên bản"→"phiên bản
  (patch)", "một cách tuyến tính"→"theo tuyến tính"; ch.18: "nguyên tố hỗn hợp"→"nguyên hỗn hợp"×3,
  "một cách mạnh mẽ"→"một cách quyết liệt"; ch.25: "bầy đàn giông"→"hiệu ứng bầy đàn"×3, "thiếu tài
  nguyên"→"cạn tài nguyên"). GĐ7: ~10 lỗi (ch.12: "ít gì"→"không có gì", "đang diễn ra"→"đang được
  triển khai"; ch.13: "lớn"→"lớn hơn"; ch.14: "nhìn thấy được bởi"→"bị...nhìn thấy"; ch.19: "tương tác"
  →"không tương tác" (bỏ mất "non-"); ch.20: "giới thiệu giao diện"→"thêm giao diện"; ch.21: "xác suất
  có thể"→"khả năng cao"; ch.24: "nổi bật lên"→"chỉ ra"; ch.25: "bầy đàn giông"×2 xác nhận). Tổng:
  ~70 sửa. +18 thuật ngữ mới (regression, panic room, game out, telemetry, span, profiling,
  fire-and-forget, hot spare, Moiré load pattern, sinkholing, mixed-integer program, over-perform,
  MTBF, break-glass, choke point, sister office, idiosyncrasy, confounding factor). Glossary hiện có
  104 thuật ngữ.
- 2026-08-28: QA độc lập lớp 2 batch 1 (ch.12-21, 24-25), 12 agent Sonnet 5 riêng biệt, không tin
  self-report GĐ4-7. 7/12 chương có lỗi thật (ch.14, 17, 18, 20, 24 sạch). Sửa: ch.12 L47 idiom y khoa
  "zebra" dịch nhầm "kỳ lân" (unicorn) → "ngựa vằn" (đổi hẳn logic ẩn dụ phổ biến-vs-hiếm thành
  thật-vs-hư cấu); ch.13 L160 dấu phẩy thừa trước dấu hai chấm trong heading; ch.15 L40 từ bịa "vá
  liềng" (vô nghĩa) → "vá víu", + gloss thừa "backend (phía sau)" vi phạm rule chỉ gloss khi định nghĩa
  vai trò (tự mâu thuẫn với L44 cùng file không gloss) → bỏ gloss; ch.16 L78 idiom SF "on the gripping
  hand" (nhại "on the one/other hand", ý ~"và quan trọng hơn cả") dịch đen "giữ chặt" → sửa nghĩa; ch.19
  L26+L29 throughput dịch "băng thông" (=bandwidth, khái niệm khác) → "thông lượng" đúng glossary, +L51
  "for all users" bị giảm nhẹ thành "phần lớn user" → "tất cả user"; ch.21 L76-78 công thức throttling
  sai nghiêm trọng, thiếu hẳn hệ số K (`1 - accepts/requests` thay vì `max(0, (requests - K·accepts)/
  (requests+1))`) khiến đoạn giải thích K ngay sau (L82-89) không khớp công thức; ch.25 L72 "bầy đàn
  giông" sót lại 1/4 chỗ chưa áp glossary "hiệu ứng bầy đàn" (3 chỗ khác trong cùng file đã đúng). +1
  lưu ý biến thể SAI mới cho throughput (băng thông). Xác nhận lại pattern: self-report pipeline luôn
  sai — batch này tự báo "sạch" GĐ4 nhưng QA lớp 2 vẫn tìm thêm 9 lỗi thật ở 7/12 file.
- 2026-08-28: Batch 2 Part III (ch.22, 23, 26, 27 — 4 chương nặng nhất, 1989 dòng), GĐ4 áp glossary
  ngược retrofit "failure domain"/"hiệu ứng bầy đàn" vào ch.22+ch.27 (từ batch 1). QA độc lập lớp 2
  (4 agent Sonnet 5) tìm 3/4 chương có lỗi thật (ch.23 sạch): ch.22 L389 typo "hot standby (đ sẵn nóng)"
  thiếu chữ → "dự phòng nóng"; ch.26 L540 chơi chữ có chủ đích của nguyên tác "the operation was a
  success but the system died" (biến tấu idiom y khoa "the patient died" thành "the system died" cho
  hợp ngữ cảnh kỹ thuật) bị dịch phục hồi lại "bệnh nhân đã chết" → sửa "hệ thống đã chết"; ch.27 L16+L22
  "Christmas Eve" (đêm Giáng Sinh) dịch nhầm "đêm Giao Thừa" (New Year's Eve, sai ngày lễ, lệch bối cảnh
  câu chuyện NORAD Tracks Santa) → sửa "đêm Giáng Sinh", + lỗi chính tả lặp "hạn chát"→"hạn chót" (2 chỗ:
  L22, L245), "from chối dịch vụ"→"từ chối dịch vụ" (2 chỗ: L183, L199). Batch 2 hoàn tất — Part III (ch.12-27) xong toàn bộ pipeline + QA lớp 2, tổng 27/34
  chương đã hoàn tất (Part I-III).
- 2026-08-28: Batch 2 ch.22, 23, 26, 27 (4 chương, sequential) — GĐ4: 2 sửa (ch.22 "miền thất bại"→giữ
   EN "failure domain"; ch.27 "bầy thú đang gầm gừ"→"hiệu ứng bầy đàn"). Ch.22 GĐ5+6+7: 5 sửa (debug
   "xử lý lỗi"→"gỡ lỗi" L116; "sụp và sập"→"tan rã (melt down) và sập" L255; "bịa ra"→"tự đặt" L301;
   regression "thoái hóa"→"lỗi hồi quy" L447; canary "thử nghiệm nhỏ trước"→"triển khai canary" L508).
   Ch.23 GĐ5+6+7: 10 sửa (lease "thuê quyền thời hạn"→bỏ gloss×2 L19+L82; renewable leases "làm mới"→
   "có thể làm mới" L156; "một cách tinh vi hơn"→"tinh vi hơn" L28; "một cách không cần thiết"→"mà
   không cần thiết" L28; "một cách phát biểu lại của"→"cách phát biểu lại" L68; "bằng cách dùng
   heartbeat"→"bằng heartbeat" L68; "đã được chứng minh bởi"→"đã chứng minh" L92; "một cách nhất
   quán"→"nhất quán" L82; "Như được mô tả bởi"→"Theo" L122; "theo một cách nhất quán"→"một cách nhất
   quán" L268). Ch.26 GĐ5+6+7: 5 sửa ("Truyền thống,"→"Theo truyền thống," L92; "sử dụng được bởi"→
   "khả dụng cho" L52; "đối với"→"trong mắt" L56; "không liên kết...một cách riêng lẻ"→"độc lập...đơn
   lẻ" L212; "sự đánh đòn"→"tác động" L351). Ch.27 GĐ5+6+7: 10 sửa (xóa "việc/sự" thừa L26/L28/L30/
   L120/L306/L314/L322; "tùy biến"→"tùy chỉnh" L98; "triển khai các giải pháp hiện có"→"tự xây dựng
   các giải pháp đã tồn tại" L129; "phó tổng giám đốc"→"phó chủ tịch (vice president)" L120; "sự bùng
   nổ tổ hợp"→"bùng nổ tổ hợp" L314). Tổng: 32 sửa. +6 thuật ngữ mới (melt down, vice president, soft
   deletion, silver bullet, combinatorial explosion, hot standby). +1 entry cập nhật (thundering herd
   thêm biến thể "bầy thú đang gầm gừ"). +1 entry mới (debug). Glossary hiện có 111 thuật ngữ.
- 2026-08-28: Rà soát tập trung "page"/"pager" ch.06-27 (phản hồi Discord: dịch lỏng lẻo ở ch.06),
  5 agent Sonnet 5 độc lập, phát hiện GLOSSARY.md thiếu hẳn entry chính thức cho page/pager (nguyên nhân
  gốc gây thiếu nhất quán) — đã bổ sung 2 entry mới (page, pager). Sửa 18 chỗ trên 6 chương:
  ch.06 (7 sửa: L194 "kiệt quệ pager"→"kiệt sức vì pager" là lỗi ngữ pháp thật — đọc thành pager bị kiệt
  quệ thay vì kiệt sức VÌ pager, đúng khớp nghi vấn Discord; L81/L93/L200/L206/L209/L237 dịch cứng/calque
  theo cấu trúc câu tiếng Anh); ch.10 L314 "đáng page" hậu tố hóa lai căng→"đáng gọi trực (page-worthy)";
  ch.11 (6 sửa: L28/L34×3/L65/L112 lẫn lộn "paging" dạng -ing với "page" dạng trần ngay trong cùng
  đoạn/câu — chuẩn hóa về "page" trần; L114/L120 rút gọn "paging alerts" thành "page" gây tối nghĩa→thêm
  lại "cảnh báo page"); ch.13 (2 sửa: L22 thiếu gloss lần đầu trong file; L116 gloss lệch "(cuộc gọi
  trực)"→"(gọi trực)" cho đồng bộ); ch.14 L92 "sắc sảo" sai sắc thái cho "harsh" (chói tai, không phải
  sharp-witted)→"chói tai"; ch.26 L484 trật tự từ "pager production" calque→"pager của hệ thống
  production". Ch.07/15/17/22/23/24/25 rà soát sạch không cần sửa (2 agent độc lập cùng xác nhận).
  1 false positive phát hiện và loại bỏ (báo cáo omission "unrelated" ở ch.26 L484 — thực tế đã có sẵn
  "không liên quan" trong bản dịch, agent đọc nhầm).
- 2026-08-28: ĐỔI QUY ƯỚC page/pager — người dùng quyết định dịch hẳn sang tiếng Việt thay vì giữ nguyên
  tiếng Anh, vì "gọi trực"/"máy gọi trực" đã dùng làm gloss quen thuộc xuyên suốt sách. Cập nhật entry
  page/pager trong bảng (dòng 25-26): "page"→"gọi trực" là từ chính, "pager"→"máy gọi trực" là từ chính,
  gloss `(page)`/`(pager)` chỉ ở lần xuất hiện đầu mỗi file. Retrofit toàn bộ 9 file đã dịch trước đó
  (6 agent Sonnet 5 song song, mỗi agent tự Grep quét sạch rồi Edit, không bỏ sót): ch.06 (~20 chỗ),
  ch.07 (1), ch.10 (1, giữ nguyên code literal `severity=page` và epigraph gloss), ch.11 (12 chỗ, chương
  dùng nhiều nhất), ch.13 (2), ch.14 (2, giữ nguyên joke "trang (page) bộ nhớ"), ch.15 (2, giữ nguyên tên
  riêng "Larry Page" ×2), ch.17 (0 — chỉ có memory page, không đụng), ch.22 (1), ch.26 (2). Verify cuối
  bằng Grep toàn bộ 9 file: chỉ còn sót các trường hợp CỐ Ý giữ nguyên (gloss lần đầu, memory page, code
  literal, tên riêng) — không còn "page"/"pager" on-call tiếng Anh trần nào sót.
- 2026-09-03: Rà soát văn phong "lủng củng" toàn bộ ch.06-27 (22 chương, ~1.1MB), theo yêu cầu người
  dùng sau đợt sửa page/pager. 14 agent Sonnet 5 độc lập rà tìm câu dịch cứng/calque theo cấu trúc tiếng
  Anh (không xét sai nghĩa — MQM đã làm ở lớp khác), tổng ~230 điểm được đề xuất. Sau khi lọc bỏ các đề
  xuất chỉ mang tính sở thích cá nhân và 1 đề xuất SAI (gợi ý đổi lại ch.26 L540 "hệ thống đã chết"→"bệnh
  nhân đã chết" — bị từ chối vì đây là bản dịch ĐÚNG đã xác nhận trước đó, nguyên tác cố tình chơi chữ
  "system died"), đã áp dụng ~99 fix qua 14 agent fix song song trên 22 file: câu dài quá nhiều mệnh đề
  được tách/nối lại, bị động calque chuyển chủ động, thiếu từ nối được bổ sung, trật tự từ Anh-hóa được
  đảo lại tự nhiên. Phát hiện thêm 2 lỗi thật ngoài phạm vi văn phong thuần túy: ch.07 L87 "tự động hóa
  bên ngoài hóa" (lỗi ghép từ vô nghĩa, đã sửa "tự động hóa bên ngoài"), ch.26 L552 "hạn chát" (lỗi gõ
  lặp lại của lỗi đã biết ở ch.27, đã sửa "hạn chót"); ch.17 L205/L311 và ch.23 L182 có lỗi ngữ pháp/thiếu
  từ thật (đã sửa). Ch.22 phát hiện thêm 1 vấn đề thuật ngữ toàn cục CHƯA quyết định: "sự thất bại" (dịch
  "failure") lặp hàng chục lần trong file, nghe nặng nề hơn "sự cố" — CHƯA áp dụng, cần người dùng chốt
  trước khi đổi hàng loạt.
- 2026-09-03: Người dùng chốt quyết định "sự thất bại"→"sự cố" (xem entry mới "failure" trong bảng).
  Retrofit 11 file có dùng "sự thất bại" như danh từ chỉ sự kiện hệ thống/dịch vụ hỏng (2 agent Sonnet 5
  song song): ch.22 (34 chỗ, kể cả heading chương "Đối phó với Các Sự thất bại Lan truyền"→"...Sự cố Lan
  truyền"), ch.24 (8), ch.07 (2), ch.06/10/12/15/17/21/25/27 (1-2 mỗi file). Phân biệt tốt và GIỮ NGUYÊN
  "thất bại" khi mô tả kết quả một nỗ lực/hành động cụ thể (ví dụ "một sự thất bại của một cuộc di cư",
  "khi nó gặp một sự thất bại, thử lại sau 1 giây" — request cụ thể fail, không phải sự kiện hệ thống) —
  5 chỗ giữ nguyên có chủ đích ở ch.12/17/21/27. "failure domain" không bị đụng (giữ nguyên EN theo quy
  ước riêng). Glossary bổ sung entry "failure" mới để chuẩn hóa cho các chương chưa dịch (Phần IV, V).
- 2026-09-03: Người dùng phát hiện qua diff review: retrofit "sự thất bại"→"sự cố" đợt trước bỏ sót vài
  chỗ gây MẤT NHẤT QUÁN trong cùng đoạn (ví dụ ch.25 L143: câu trước dùng "sự cố", câu ngay sau chỉ lại
  đúng đối tượng đó bằng "Các loại thất bại này"). Dispatch 6 agent Sonnet 5 "chuyên gia" đọc kỹ ngữ cảnh
  từng đoạn (không tìm-thay-thế máy móc) rà lại toàn bộ 11 file đã retrofit. Tìm thêm 16 lỗi nhất quán
  thật, sửa hết: ch.22 nặng nhất (12 chỗ — 9 câu văn + PHÁT HIỆN QUAN TRỌNG: 3 tiêu đề mục con vẫn ghi
  "Thất bại Lan truyền" thay vì "Sự cố Lan truyền" dù đã đổi tên chương chính, dòng 395/486/537), ch.24
  (2 chỗ, section "Giải quyết Các Sự cố Một phần"), ch.21 (1 chỗ, 3 lần chỉ cùng khái niệm lẫn từ giữa
  câu), ch.25 (1 chỗ L143 nêu trên). Ch.06/07/10/12/15/17/27 rà kỹ nhưng không tìm thêm lỗi — xác nhận
  các chỗ "thất bại" còn lại đều đúng (động từ/thuật ngữ cố định như "single point of failure", "failure
  mode", "assertion failure", "fail-fast"). Bài học: đợt retrofit thuật ngữ hàng loạt cần luôn có 1 lượt
  QA thứ 2 đọc-theo-ngữ-cảnh riêng, vì tìm-thay-thế máy móc không bắt được các câu chỉ-lại (anaphora).
