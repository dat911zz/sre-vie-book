# Báo cáo Rà soát & Tuỳ biến Chất lượng Bản dịch SRE Book

**Ngày:** 2026-08-25 (lần đầu) / 2026-08-26 (rà soát lại ch.1,2,3) / 2026-08-27 (ch.4-9 Part 2) / 2026-08-28 (ch.10 Part 3)
**Phạm vi:** Toàn bộ 49 file `.md` trong `docs/sre/`
**Trạng thái:** Hoàn tất — ch.1-10 đã chạy đủ Giai đoạn 4→7 (glossary, văn phong, semantic, QA độc lập)

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
| `11-being-on-call.md` | CHANGED | ~30 câu |
| `12-effective-troubleshooting.md` | CHANGED | ~45 câu |
| `13-emergency-response.md` | CHANGED | ~40 câu; sửa "não đóng" → "động não" |
| `14-managing-incidents.md` | CHANGED | ~25 câu |
| `15-postmortem-culture.md` | CHANGED | ~18 câu |
| `16-tracking-outages.md` | CHANGED | ~15 câu |
| `17-testing-reliability.md` | CHANGED | ~40 câu |
| `18-software-engineering-in-sre.md` | CHANGED | ~35 câu; thêm footer bị thiếu |
| `19-load-balancing-frontend.md` | CHANGED | ~30 câu |
| `20-load-balancing-datacenter.md` | CHANGED | ~32 câu |
| `21-handling-overload.md` | CHANGED | ~38 câu |
| `22-addressing-cascading-failures.md` | CHANGED | ~50 câu |
| `23-managing-critical-state.md` | CHANGED | ~15 câu; bỏ annotation sai (crash, quorum, STONITH, gossip) |
| `24-distributed-periodic-scheduling.md` | CHANGED | ~8 câu |
| `25-data-processing-pipelines.md` | CHANGED | ~12 câu |
| `26-data-integrity.md` | CHANGED | ~20 câu; 5 H5→H3, 2 H4→H3 |
| `27-reliable-product-launches.md` | CHANGED | ~18 câu; sửa "Tý danh này" → "Điều này" |

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