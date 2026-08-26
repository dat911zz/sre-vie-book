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
| graceful degradation | **suy giảm nhẹ nhàng** | suy giảm tinh tế; suy giảm êm ả; suy giảm từ từ | Chốt theo bản đã dùng làm tên mục lớn ở ch.22 |
| graceful load shedding | **gánh nhẹ tải một cách nhẹ nhàng** | loại bỏ tải một cách tinh tế | Đồng bộ tính từ "nhẹ nhàng" với graceful degradation |
| thundering herd | **hiệu ứng bầy đàn** | bầy đàn giông; "bầy thú đang gầm gừ" | |
| distributed consensus | **nhất trí phân tán** | đồng thuận phân tán | Không dùng "đồng thuận" — dành riêng cho "consensus" thường (không phân tán) nếu cần phân biệt |
| failure domain | **failure domain** (giữ nguyên) | domain lỗi; miền thất bại; miền lỗi | Jargon phổ biến, không dịch |
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
| throughput | **thông lượng (throughput)** | lưu lượng | "lưu lượng" đang dùng cho traffic — 1 từ Việt cho 2 khái niệm khác nhau gây va chạm. Phát hiện ở ch.4 (3 chỗ dùng "lưu lượng" cho throughput) — đã sửa |

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
