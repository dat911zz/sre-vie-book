# Báo cáo Rà soát & Tuỳ biến Chất lượng Bản dịch SRE Book

**Ngày:** 2026-08-25 (lần đầu) / 2026-08-26 (rà soát lại ch.1,2,3) / 2026-08-27 (ch.4-9 Part 2) / 2026-08-28 (ch.10-27 Part 3, batch 1+2) / 2026-09-03 (ch.28-34 Part IV-V, QA lớp 2; ch.06-27 QA lượt 2) / 2026-09-05 (soften pass toàn sách)
**Phạm vi:** Toàn bộ 49 file `.md` trong `docs/sre/`
**Trạng thái:** Hoàn tất — ch.1-34 (toàn bộ Part I-V) đã chạy đủ Giai đoạn 4→7 (glossary, văn phong, semantic, QA độc lập) + QA lớp 2 + **soften pass** (2026-09-05: model local softdream viết lại văn phong theo chính sách 2 tầng thuật ngữ, 2 lượt QA người/Claude, áp 1.665/2.109 đoạn; đoạn kể chuyện dịch incident/outage/push/bug/team/engineer, đoạn kỹ thuật giữ glossary EN; chi tiết `labs/sre-qa-loop/reports/soften-book/QA-VERDICT.md` bên CommandCenter)

---

## 1. Tổng quan

| Chỉ số | Trước | Sau |
|--------|-------|-----|
| File có footer canonical | 48/49 | **49/49** |
| File có header `---` + blank line | 21/49 | **49/49** |
| File dùng header cũ (dòng 3 = "dịch sát nguyên bản") | 28/49 | **0/49** |
| File có heading H5/H6 | 19 | **0** |
| File có H1 cho section (thay vì H2) | ~40 | **0** |
| Tổng chú thích trong ngoặc (word + `(`) | 1395 | **1020** (−27%) |
| Ký tự CJK | 0 | **0** |
| Toc links (48) | 48 resolve | **48 resolve (0 missing)** |

---

## 2. Sửa theo file (tóm tắt)

### Front matter

| File | Trạng thái | Ghi chú |
|------|-----------|---------|
| `00-front/foreword.md` | CHANGED | ~10 câu viết lại; sửa calque, cắt cụm sáo |
| `00-front/preface.md` | CHANGED | ~14 câu; bỏ chú thích từ thông dụng; sửa "instance (hồi vị trí)" → "instance (thực thể)" |

### Part I — Introduction

| File | Trạng thái | Ghi chú |
|------|-----------|---------|
| `_part.md` | CHANGED | 1 câu (bỏ `production (sản xuất)`) |
| `01-introduction.md` | CHANGED | ~16 câu; sửa "dumb (vô tình)" → "đơn giản"; "instance (hồi vị trí)" → "thực thể" |
| `02-production-environment.md` | **PIPELINE HOÀN TẤT** | ~10 câu; sửa "dumb", "map (bản đồ)" → "cấu trúc ánh xạ" + QA lượt 2 (2026-09-03): gloss "binpack (gói chặt)"→"bin-packing", "điểm lỗi đơn lẻ"→"điểm thất bại duy nhất" (2) |

### Part II — Principles

| File | Trạng thái | Ghi chú |
|------|-----------|---------|
| `_part.md` | CHANGED | ~9 câu |
| `03-embracing-risk.md` | CHANGED | ~30 câu |
| `04-service-level-objectives.md` | CHANGED | ~30 câu |
| `05-eliminating-toil.md` | CHANGED | ~25 câu |
| `06-monitoring-distributed-systems.md` | **PIPELINE HOÀN TẤT** | ~40 câu; sửa 2 H1→H2, bỏ trùng heading + QA lượt 2 (2026-09-03): "bắn (firing)"→"fire (kích hoạt)" ×2, "chế độ thất bại"→"failure mode", gloss paging lặp, "với mắt hướng đến" calque, 5 câu văn phong |
| `07-automation-at-google.md` | **PIPELINE HOÀN TẤT** | ~40 câu; 7 H1→H2 + QA lượt 2 (2026-09-03): "hand-holding"→"giữ tay", markdown lồng *…* hỏng render, 3 gloss dùng đúng biến thể SAI của glossary (bin-packing/proof of concept/rate limiting), "tự soi xét nội tại"→"tự nội phản", heading "Cho phép Sự cố ở Quy mô"; từ chối 1 đề xuất SAI (đồng bộ 2 danh sách 5 bậc — nguyên bản cố ý khác nhau) |
| `08-release-engineering.md` | **PIPELINE HOÀN TẤT** | ~30 câu + QA lượt 2 (2026-09-03): bỏ câu chêm THÊM Ý SAI SỰ THẬT (First Folio/Bad Quarto là ấn bản in, không phải "tên vở kịch"), 4 heading lặp gloss "(X) (X)", hạ 12 heading con về ### |
| `09-simplicity.md` | **PIPELINE HOÀN TẤT** | ~18 câu + QA lượt 2 (2026-09-03): "shipped"→"vận chuyển", "rules of thumb"→"quy tắc ngón tay cái", "collaborators"→"cộng tác viên… mỗi người" (sai chủ thể), 3 văn phong |

### Part III — Practices

| File | Trạng thái | Ghi chú |
|------|-----------|---------|
| `_part.md` | CHANGED | ~20 câu |
| `10-practical-alerting.md` | **PIPELINE HOÀN TẤT** | Ch.10: GĐ1 +28 thuật ngữ, GĐ2 30 sửa (glossary + nghĩa), GĐ3 +5 thuật ngữ, GĐ4 sạch, GĐ5 22 sửa (văn phong), GĐ6 2 sửa (in lockstep mistranslation, omission "chúng"), GĐ7 QA độc lập (Claude Opus 5) 2 lỗi thật + 1 borderline, GĐ8 áp fix. Số liệu verify khớp 100%. + QA lượt 2 (2026-09-03, 9 dòng sửa): gloss "fire (kích hoạt cảnh báo)" lặp ×4→1, 5 văn phong |
| `11-being-on-call.md` | **PIPELINE HOÀN TẤT** | Ch.11: GĐ4 sạch, GĐ5 7 sửa (văn phong), GĐ6 4 sửa (chase→follow the sun, embed≠cài đặt, omission), GĐ7 QA độc lập 2 sửa, QA thủ công thêm 7 lỗi (fix-sai-hướng L120, typo "mực trần", "đội đơn vị"→"một vị trí" x3, link hỏng, va chạm nghĩa). Số liệu verify khớp 100%. + QA lượt 2 (2026-09-03, 5 dòng sửa): "các ngắt (interrupt)"→"gián đoạn", "tốc độ tính năng"→"feature velocity", "Vòng bánh"→"Bánh xe Bất hạnh" |
| `12-effective-troubleshooting.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~45 câu ban đầu) + QA lớp 2 (Sonnet 5) tìm 1 lỗi: L47 idiom y khoa "zebra"→"kỳ lân" (SAI, đổi hẳn logic ẩn dụ) → sửa "ngựa vằn" + QA lượt 2 (2026-09-03, 16 dòng sửa): "probe (mồi)"→"(thăm dò)", "deadlock (tử tự khóa)"→"(khóa chết)", "yếu tố gây nhân"→"nhân quả", "rành rọt"→"dày dạn kinh nghiệm", 2 heading lặp gloss, 8 văn phong |
| `13-emergency-response.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~40 câu; "não đóng"→"động não") + QA lớp 2 tìm 1 lỗi format: L160 dấu phẩy thừa trong heading + QA lượt 2 (2026-09-03, 9 dòng sửa): 4 gloss đảo thứ tự EN/VN (page, pager, turndown, drain), "chế độ thất bại"→"failure mode", "Ít có chúng ta" ngữ pháp |
| `14-managing-incidents.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~25 câu) + QA lớp 2: sạch, không lỗi thật + QA lượt 2 (2026-09-03, 12 dòng sửa): "Independently"→"Đồng thời" (sai nghĩa), 8 heading lệch cấp, "bắn"→"fire" |
| `15-postmortem-culture.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~18 câu) + QA lớp 2 tìm 2 lỗi tại L40: từ bịa "vá liềng"→"vá víu", gloss thừa "backend (phía sau)" vi phạm rule + QA lượt 2 (2026-09-03, 4 dòng sửa): heading "Vòng bánh Bất hạnh"→"Bánh xe Bất hạnh", 4 văn phong |
| `16-tracking-outages.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~15 câu) + QA lớp 2 tìm 1 lỗi: L78 idiom SF "on the gripping hand" dịch đen "giữ chặt"→"quan trọng hơn cả" + QA lượt 2 (2026-09-03, 3 dòng sửa): "separator"→"bộ phận" (sai nghĩa), "switch"→"mạch chuyển", gloss client thừa |
| `17-testing-reliability.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~40 câu) + QA lớp 2: sạch, không lỗi thật (17/17 footnote khớp) + QA lượt 2 (2026-09-03, 8 dòng sửa): "short-circuit"→"ngắt ngắn mạch", "điểm liệt" cụt chữ, mảnh "not" sót trong footnote, "phòng thủ có độ sâu"→"nhiều lớp", "căn bậc" |
| `18-software-engineering-in-sre.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~35 câu; thêm footer bị thiếu) + QA lớp 2: sạch, verify "mixed-integer program" đúng 3/3 chỗ + QA lượt 2 (2026-09-03, 12 dòng sửa): "subject matter"→"đối tượng", "brittle"→"vỡ vụn", "laborious"→"cồng kềnh", "division"→"phân khu", "bám chấp" (từ vô nghĩa), "các ngắt"→"gián đoạn" |
| `19-load-balancing-frontend.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~30 câu) + QA lớp 2 tìm 2 lỗi: L26+L29 throughput dịch "băng thông"(=bandwidth, sai)→"thông lượng", L51 "for all users"→"phần lớn user" giảm nhẹ nghĩa→"tất cả user" + QA lượt 2 (2026-09-03, 6 dòng sửa): 2 PHỦ ĐỊNH NGƯỢC cùng đoạn L33 ("may be routed"→"không thể", "stateless"→"có trạng thái"), "underutilized" cụt chữ, heading thiếu gloss EN |
| `20-load-balancing-datacenter.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~32 câu) + QA lớp 2: sạch, toàn bộ số liệu/công thức khớp gốc + QA lượt 2 (2026-09-03, 20 dòng sửa): "switch (bo mạch chuyển)", "trường hợp góc"→"khó", "đại lý (proxy)"→"chỉ số đại diện", 12 heading lệch cấp (## phẳng → ###/####), "êm ả"→"nhẹ nhàng" |
| `21-handling-overload.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~38 câu) + QA lớp 2 tìm 1 lỗi nghiêm trọng: L76-78 công thức throttling thiếu hệ số K → sửa `max(0,(requests-K*accepts)/(requests+1))` + QA lượt 2 (2026-09-03, 10 dòng sửa): "êm ả"→"nhẹ nhàng" ×5, "đại lý (proxy)", "chi phí phụ"→"overhead (công việc phụ)", "bọt lên" calque, 4 văn phong |
| `22-addressing-cascading-failures.md` | **PIPELINE HOÀN TẤT** | Batch 2: GĐ4-7 (~50 câu, +retrofit "failure domain") + QA lớp 2 tìm 1 lỗi: L389 typo "hot standby (đ sẵn nóng)" thiếu chữ → "dự phòng nóng" + QA lượt 2 (2026-09-03, 17 dòng sửa): "can't make progress"→"không thể tiến bộ" ×2, slug anchor "do-ley", 7 gloss sai/thừa (frontend/backend/client/on-call/postmortem/watchdog), thiếu gloss (page), "chế độ thất bại"→"failure mode" ×3 |
| `23-managing-critical-state.md` | **PIPELINE HOÀN TẤT** | Batch 2: GĐ4-7 (~15 câu; bỏ annotation sai crash/quorum/STONITH/gossip) + QA lớp 2: sạch, verify 4 thuật ngữ giữ EN đúng + QA lượt 2 (2026-09-03, 8 dòng sửa): omission "by being implemented as an RSM", link chéo "Đánh Lành Các Thất Bại Dây Chuyền" (vô nghĩa)→tên ch.22 chính thức, heading "Thất Bại Phối Hợp"→"Sự Cố", 2 heading lệch cấp, "chỉnh chu" |
| `24-distributed-periodic-scheduling.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~8 câu) + QA lớp 2: sạch (L188 crontab lặp "ngày trong tuần" là lỗi có sẵn trong nguyên tác, không phải lỗi dịch) + QA lượt 2 (2026-09-03, 6 dòng sửa): tag [[Ver15]] gắn nhầm cho Paxos (Ver15 = paper Borg), "chế độ thất bại"→"failure mode" ×3, "ở thứ tự" calque |
| `25-data-processing-pipelines.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~12 câu) + QA lớp 2 tìm 1 lỗi: L72 "bầy đàn giông" sót 1/4 chỗ chưa áp glossary→"hiệu ứng bầy đàn" + QA lượt 2 (2026-09-03, 11 dòng sửa): "rogue"→"bất hợp pháp", "predicate"→"suy ra", "conclusively"→"quyết đoán", "phiên hóa"/"phác đồ đặt tên" (coinage lạ), "nút cổ chai"→"nút thắt cổ chai (bottleneck)" |
| `26-data-integrity.md` | **PIPELINE HOÀN TẤT** | Batch 2: GĐ4-7 (~20 câu; 5 H5→H3, 2 H4→H3) + QA lớp 2 tìm 1 lỗi: L540 chơi chữ "the system died" bị dịch phục hồi thành "bệnh nhân đã chết" → sửa "hệ thống đã chết" + QA lượt 2 (2026-09-03, 12 dòng sửa): "Thách Thúc" ×2 heading, "lưu trữ lưu" ×7→"kho lưu trữ", "data availability" dịch lệch 1 chỗ, 4 văn phong |
| `27-reliable-product-launches.md` | **PIPELINE HOÀN TẤT** | Batch 2: GĐ4-7 (~18 câu; sửa "Tý danh này"→"Điều này", retrofit "hiệu ứng bầy đàn") + QA lớp 2 tìm 3 lỗi: L16/L22 "Christmas Eve"→"đêm Giao Thừa" sai ngày lễ → "đêm Giáng Sinh", "hạn chát"→"hạn chót" (2 chỗ), "from chối"→"từ chối" (2 chỗ) + QA lượt 2 (2026-09-03, 25 dòng sửa): "individual features"→"tính năng cá nhân", "Ra Mát", "Từ Chậm", "thrashing (lẫy lạch)", "LCE thời gian toàn thời gian", 3 heading sai format, "điểm lỗi đơn lẻ"→"điểm thất bại duy nhất", "nguyên lý bậc nhất"→"cơ bản", 6 văn phong |

### Part IV — Management

| File | Trạng thái | Ghi chú |
|------|-----------|---------|
| `_part.md` | **PIPELINE HOÀN TẤT** | QA lớp 2 (2026-09-03): 1 sửa gloss "interrupt (gián đoạn)"→"sự gián đoạn (interrupt)" |
| `28-accelerating-sre-on-call.md` | **PIPELINE HOÀN TẤT** | QA lớp 2 (2026-09-03, Sonnet 5): 7 sai nghĩa (L169 chủ thể "developers behind the failing system", L235 "shakes out bugs"→"phơi bày", typo mơ-típ/đứt đứt/tài nguồn, H4 lạc), 7 thuật ngữ (5 "page"→gọi trực, "êm ả"→nhẹ nhàng, sự cố), 7 văn phong |
| `29-dealing-with-interrupts.md` | **PIPELINE HOÀN TẤT** | QA lớp 2 (2026-09-03): retrofit HÀNG LOẠT ~40 "interrupt"→"gián đoạn" + ~15 page/pager→gọi trực/máy gọi trực (file bỏ qua quy ước glossary đã chốt), đổi tên chương "Đối phó với Gián đoạn" (toc ghi "Các Ngắt" lệch với file); 4 sai nghĩa (languishing→"mòn mỏi", needy customers, thừa từ), 9 văn phong (4× thiếu "ở" trong "ở trong các gián đoạn") |
| `30-embedding-sre-operational-overload.md` | **PIPELINE HOÀN TẤT** | QA lớp 2 (2026-09-03): 6 sai nghĩa (scalability→"tính khả dụng mở rộng" ×2, tech lead "(quyền kỹ thuật)", loaded questions→"gán nợ", "along the lines of"→"theo đường nét"), 5 thuật ngữ (on-call gloss, overhead, sự cố, fire (bắn), turnup), 7 văn phong; từ chối 1 đề xuất SAI của agent (L66 "passes in the short term" dịch đúng) |
| `31-communication-and-collaboration.md` | **PIPELINE HOÀN TẤT** | QA lớp 2 (2026-09-03): 12 sai nghĩa (performance envelope→"phong bì", in the flesh→"xác thịt", catch-all→"cái bắt tất cả" ×2, literature→"văn học", ignorance→"ngu dốt", index file→"bảng mục lục", cụt chữ "chuyển đổi trực"/"khá động"), 8 thuật ngữ (mục "Các sự kiện Page (lỗi)" + 8 "page" trần→gọi trực, pager), 10 văn phong |
| `32-evolving-sre-engagement-model.md` | **PIPELINE HOÀN TẤT** | QA lớp 2 (2026-09-03): 4 sai nghĩa (short-circuit→"ngắn mạch", orders of magnitude→"bậc đại số", progressive→"tiến bộ", visible→"tầm nhìn"), 3 thuật ngữ (pager, turndown "giảm cấu hình"→tắt, gloss PRR lặp), 10 văn phong ("các người" ×4→"những người") |

### Part V — Conclusions

| File | Trạng thái | Ghi chú |
|------|-----------|---------|
| `_part.md` | **PIPELINE HOÀN TẤT** | QA lớp 2 (2026-09-03): sạch; 1 sửa "Bài học học được"→"Bài học rút ra" (đồng bộ tên ch.33) |
| `33-lessons-learned.md` | **PIPELINE HOÀN TẤT** | QA lớp 2 (2026-09-03): 12 tên chuyên gia + toàn bộ số liệu verify khớp; 2 sai nghĩa (religiously→"thành tâm", URL footnote 8 hỏng vì em-dash), 9 thuật ngữ ("bảo vệ nhiều lớp" ×5→"phòng thủ nhiều lớp", sự thất bại ×6→sự cố), 2 văn phong; đổi tên chương "Bài học Rút ra" (bỏ lặp âm "học học", đồng bộ toc/sidebar/ch.18/27/31/32) |
| `34-conclusion.md` | **PIPELINE HOÀN TẤT** | QA lớp 2 (2026-09-03): 4 sai nghĩa (cockpit→"cabin" ×5 lặp hệ thống→"buồng lái", pleasure→"đặc quyền"), 3 thuật ngữ (sự cố), 1 văn phong |

### Appendices

| File | Trạng thái | Ghi chú |
|------|-----------|---------|
| `A-availability-table.md` | CHANGED | 1 câu; bảng giữ nguyên |
| `B-service-best-practices.md` | CHANGED | ~12 câu; sửa "gạch chéo ngược" → "gạch chéo" (forward slash) |
| `C-incident-document.md` | PASS | Văn bản tài liệu ví dụ, không sửa |
| `D-example-postmortem.md` | PASS | Văn bản ví dụ + footnotes, không sửa |
| `E-launch-checklist.md` | CHANGED | 1 câu (intro); checklist giữ nguyên |
| `F-production-meeting.md` | PASS | Biên bản họp ví dụ, không sửa |
| `bibliography.md` | PASS | Giữ nguyên toàn bộ entry citation |

### TOC

| File | Trạng thái | Ghi chú |
|------|-----------|---------|
| `toc.md` | PASS | 48/48 links resolve; văn phong đã ổn |

---

## 3. Chỗ nghi ngờ nghĩa gốc — ĐÃ DUYỆT

| # | File | Trích dẫn | Quyết định | Trạng thái |
|---|------|-----------|-----------|------------|
| 1 | `00-front/foreword.md` | "Bài học đó thôi đã đáng giá bằng trọng lượng giấy điện tử của nó." | Thay → "Bài học đó thôi đã có giá trị vô cùng." | **Đã sửa** |
| 2 | `00-front/preface.md` | "SRE Way (Đạo SRE)" | Giữ nguyên | **Giữ** |
| 3 | `99-appendices/B-service-best-practices.md` | "dấu gạch chéo (forward slash)" | Bỏ chú thích `(forward slash)`, bọc `/` trong backtick → `` dấu gạch chéo duy nhất (`/`) `` | **Đã sửa** |
| 4 | `part-4-management/28-accelerating-sre-on-call.md` | "Kết quả? Chà, có lẽ nó chính xác và hữu ích hơn so với nếu *tôi* đã đưa ra bài nói chuyện" | Đoạn trích dẫn tự thuật của Paul Cowan (Google SRE). Bỏ "Chà" (suồng sã), "bài nói chuyện" → "tự thuyết trình", "cái này" → "hệ thống này" | **Đã sửa** |

---

## 4. Quy ước đã áp dụng

- **Header canonical:** `# Title` → blockquote (3 dòng) → `---` → blank line → body
- **Footer canonical:** `\n\n---\n\nCopyright © 2017 Google, Inc. Published by O'Reilly Media, Inc. Licensed under [CC BY-NC-ND 4.0](...)`
- **Heading:** H1 chỉ cho tiêu đề chương/bổ sung; H2 cho section chính; H3 cho subsection. Không dùng H5/H6.
- **Thuật ngữ:** Giữ tiếng Anh cho thuật ngữ kỹ thuật (SRE, SLI, SLO, toil, on-call, incident, postmortem, quorum, canary, ...).
- **Chú thích:** Chỉ ở lần xuất hiện đầu tiên trong 1 file. Không chú thích từ phổ thông.

---

## 5. Không đụng

- `part-1-introduction.rar` (file .rar lạ)
- Code blocks, tables, URLs, citations `[[Key]]`, footnote definitions
- Bibliography entries (tất cả giữ nguyên)
- Example document content trong appendices C, D, F