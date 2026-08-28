# Báo cáo Rà soát & Tuỳ biến Chất lượng Bản dịch SRE Book

**Ngày:** 2026-08-25 (lần đầu) / 2026-08-26 (rà soát lại ch.1,2,3) / 2026-08-27 (ch.4-9 Part 2) / 2026-08-28 (ch.10-27 Part 3, batch 1+2)
**Phạm vi:** Toàn bộ 49 file `.md` trong `docs/sre/`
**Trạng thái:** Hoàn tất — ch.1-27 (Part I-III trọn vẹn) đã chạy đủ Giai đoạn 4→7 (glossary, văn phong, semantic, QA độc lập) + QA lớp 2

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
| `02-production-environment.md` | CHANGED | ~10 câu; sửa "dumb", "map (bản đồ)" → "cấu trúc ánh xạ" |

### Part II — Principles

| File | Trạng thái | Ghi chú |
|------|-----------|---------|
| `_part.md` | CHANGED | ~9 câu |
| `03-embracing-risk.md` | CHANGED | ~30 câu |
| `04-service-level-objectives.md` | CHANGED | ~30 câu |
| `05-eliminating-toil.md` | CHANGED | ~25 câu |
| `06-monitoring-distributed-systems.md` | CHANGED | ~40 câu; sửa 2 H1→H2, bỏ trùng heading |
| `07-automation-at-google.md` | CHANGED | ~40 câu; 7 H1→H2 |
| `08-release-engineering.md` | CHANGED | ~30 câu |
| `09-simplicity.md` | CHANGED | ~18 câu |

### Part III — Practices

| File | Trạng thái | Ghi chú |
|------|-----------|---------|
| `_part.md` | CHANGED | ~20 câu |
| `10-practical-alerting.md` | **PIPELINE HOÀN TẤT** | Ch.10: GĐ1 +28 thuật ngữ, GĐ2 30 sửa (glossary + nghĩa), GĐ3 +5 thuật ngữ, GĐ4 sạch, GĐ5 22 sửa (văn phong), GĐ6 2 sửa (in lockstep mistranslation, omission "chúng"), GĐ7 QA độc lập (Claude Opus 5) 2 lỗi thật + 1 borderline, GĐ8 áp fix. Số liệu verify khớp 100%. |
| `11-being-on-call.md` | **PIPELINE HOÀN TẤT** | Ch.11: GĐ4 sạch, GĐ5 7 sửa (văn phong), GĐ6 4 sửa (chase→follow the sun, embed≠cài đặt, omission), GĐ7 QA độc lập 2 sửa, QA thủ công thêm 7 lỗi (fix-sai-hướng L120, typo "mực trần", "đội đơn vị"→"một vị trí" x3, link hỏng, va chạm nghĩa). Số liệu verify khớp 100%. |
| `12-effective-troubleshooting.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~45 câu ban đầu) + QA lớp 2 (Sonnet 5) tìm 1 lỗi: L47 idiom y khoa "zebra"→"kỳ lân" (SAI, đổi hẳn logic ẩn dụ) → sửa "ngựa vằn" |
| `13-emergency-response.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~40 câu; "não đóng"→"động não") + QA lớp 2 tìm 1 lỗi format: L160 dấu phẩy thừa trong heading |
| `14-managing-incidents.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~25 câu) + QA lớp 2: sạch, không lỗi thật |
| `15-postmortem-culture.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~18 câu) + QA lớp 2 tìm 2 lỗi tại L40: từ bịa "vá liềng"→"vá víu", gloss thừa "backend (phía sau)" vi phạm rule |
| `16-tracking-outages.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~15 câu) + QA lớp 2 tìm 1 lỗi: L78 idiom SF "on the gripping hand" dịch đen "giữ chặt"→"quan trọng hơn cả" |
| `17-testing-reliability.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~40 câu) + QA lớp 2: sạch, không lỗi thật (17/17 footnote khớp) |
| `18-software-engineering-in-sre.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~35 câu; thêm footer bị thiếu) + QA lớp 2: sạch, verify "mixed-integer program" đúng 3/3 chỗ |
| `19-load-balancing-frontend.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~30 câu) + QA lớp 2 tìm 2 lỗi: L26+L29 throughput dịch "băng thông"(=bandwidth, sai)→"thông lượng", L51 "for all users"→"phần lớn user" giảm nhẹ nghĩa→"tất cả user" |
| `20-load-balancing-datacenter.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~32 câu) + QA lớp 2: sạch, toàn bộ số liệu/công thức khớp gốc |
| `21-handling-overload.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~38 câu) + QA lớp 2 tìm 1 lỗi nghiêm trọng: L76-78 công thức throttling thiếu hệ số K → sửa `max(0,(requests-K*accepts)/(requests+1))` |
| `22-addressing-cascading-failures.md` | **PIPELINE HOÀN TẤT** | Batch 2: GĐ4-7 (~50 câu, +retrofit "failure domain") + QA lớp 2 tìm 1 lỗi: L389 typo "hot standby (đ sẵn nóng)" thiếu chữ → "dự phòng nóng" |
| `23-managing-critical-state.md` | **PIPELINE HOÀN TẤT** | Batch 2: GĐ4-7 (~15 câu; bỏ annotation sai crash/quorum/STONITH/gossip) + QA lớp 2: sạch, verify 4 thuật ngữ giữ EN đúng |
| `24-distributed-periodic-scheduling.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~8 câu) + QA lớp 2: sạch (L188 crontab lặp "ngày trong tuần" là lỗi có sẵn trong nguyên tác, không phải lỗi dịch) |
| `25-data-processing-pipelines.md` | **PIPELINE HOÀN TẤT** | Batch 1: GĐ4-7 (~12 câu) + QA lớp 2 tìm 1 lỗi: L72 "bầy đàn giông" sót 1/4 chỗ chưa áp glossary→"hiệu ứng bầy đàn" |
| `26-data-integrity.md` | **PIPELINE HOÀN TẤT** | Batch 2: GĐ4-7 (~20 câu; 5 H5→H3, 2 H4→H3) + QA lớp 2 tìm 1 lỗi: L540 chơi chữ "the system died" bị dịch phục hồi thành "bệnh nhân đã chết" → sửa "hệ thống đã chết" |
| `27-reliable-product-launches.md` | **PIPELINE HOÀN TẤT** | Batch 2: GĐ4-7 (~18 câu; sửa "Tý danh này"→"Điều này", retrofit "hiệu ứng bầy đàn") + QA lớp 2 tìm 3 lỗi: L16/L22 "Christmas Eve"→"đêm Giao Thừa" sai ngày lễ → "đêm Giáng Sinh", "hạn chát"→"hạn chót" (2 chỗ), "from chối"→"từ chối" (2 chỗ) |

### Part IV — Management

| File | Trạng thái | Ghi chú |
|------|-----------|---------|
| `_part.md` | CHANGED | ~3 câu; sửa "ĐỐI phó" → "Đối phó" |
| `28-accelerating-sre-on-call.md` | CHANGED | ~35 câu |
| `29-dealing-with-interrupts.md` | CHANGED | ~20 câu |
| `30-embedding-sre-operational-overload.md` | CHANGED | ~15 câu; H5→H3, H6→H4 |
| `31-communication-and-collaboration.md` | CHANGED | ~25 câu |
| `32-evolving-sre-engagement-model.md` | CHANGED | ~12 câu; sửa typo "chuẩn hóa hóa" → "chuẩn hóa" |

### Part V — Conclusions

| File | Trạng thái | Ghi chú |
|------|-----------|---------|
| `_part.md` | CHANGED | ~3 câu |
| `33-lessons-learned.md` | CHANGED | ~30 câu; giảm 13 chú thích |
| `34-conclusion.md` | CHANGED | ~10 câu |

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