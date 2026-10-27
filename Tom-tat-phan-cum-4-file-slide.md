# Tóm tắt & phân cụm nội dung — 4 file slide IT5427

Đã đọc, OCR và phân cụm toàn bộ 4 file trong folder `slides/` (188 slide). Dưới đây là bản đồ nội dung theo từng khung/cụm, dùng để ôn tập nhanh trước khi vào quiz.

---

## File 1 — `1. Introduction.pdf` (Buổi 2, 15/9) — 46 slide

**Chủ đề:** Big Data Integration & Processing — tổng quan

| Cụm | Nội dung | Slide chính |
|---|---|---|
| A. Data Integration là gì | DBMS = abstraction (logical vs physical); Data Integration = abstraction bậc cao hơn — 1 mediated schema che giấu vị trí/mô hình dữ liệu/khác biệt ngữ nghĩa của nhiều nguồn | 3–6 |
| B. Ứng dụng thực tế | Business (EII, 50% chi phí IT), Science (biomedical: OMIM/HUGO/GO...), Web (deep web, hàng trăm triệu bảng), ví dụ VN (Baomoi.com) | 7–15 |
| C. **6 mức độ dị biệt (Heterogeneity)** — khung khái niệm quan trọng nhất | 1) System/platform, 2) Syntactic/format, 3) Structural/schema, 4) Semantic/conceptual, 5) Instance/identity (→ dẫn tới Entity Resolution), 6) Temporal/process, 7) Quality/provenance/governance | 16–25 |
| D. Mục tiêu & vấn đề tích hợp | Cung cấp truy vấn thống nhất; Data Integration issues (modeling, profiling, cleansing, warehousing...) | 26–27 |
| E. Kiến trúc tích hợp | Virtual (query-time, dùng wrapper) vs Warehousing (offline replication, ETL) — ưu/nhược mỗi loại; Mediation languages; Query Processing pipeline (reformulation → optimizer → execution engine → wrappers) | 29–36 |
| F. Big Data context | Thống kê tăng trưởng dữ liệu (175 ZB năm 2025), 5V, Big Data tech stack, **Task pipeline: Schema Alignment → Entity Resolution → Data Fusion** (khung tổng thể cho cả 3 buổi tiếp theo) | 38–46 |

---

## File 2 — `2_SchemaAlignment.pdf` (Buổi 3, 22/9) — 86 slide

**Chủ đề:** Schema Alignment & Query Rewriting

| Cụm | Nội dung | Slide chính |
|---|---|---|
| A. Ví dụ dị biệt schema | Domain Movie: mediated schema (Movie/Actors/Plays/Reviews) vs 8 nguồn S1–S8 khác nhau về đặt tên, tổ chức bảng, độ phủ | 5–10 |
| B. Ngữ nghĩa hình thức của Schema Mapping | I(G), I(Si), quan hệ M_R ⊆ I(G) × I(S1) × ... × I(Sn); Possible instances; **Certain Answers** (câu trả lời đúng trong MỌI instance khả dĩ) — khái niệm lõi xuyên suốt cả 3 ngôn ngữ mapping | 13–20 |
| C. **3 ngôn ngữ Schema Mapping — trọng tâm của buổi** | **GAV** (Global-as-View: mediated schema = view trên sources; reformulation = unfolding, nhanh nhưng cứng nhắc, không biểu diễn được incomplete info); **LAV** (Local-as-View: source = view trên mediated schema; reformulation = "answering queries using views", biểu diễn linh hoạt hơn nhưng khó hơn); **GLAV** (kết hợp cả hai, dùng Tuple Generating Dependencies) | 21–58 |
| D. Data-level heterogeneity | Mapping kèm transformation (đổi đơn vị, tính toán: Celsius↔Fahrenheit, giá có/không thuế) | 59–60 |
| E. Query Unfolding | Thuật toán unfold (thay subgoal bằng rule tương ứng, lặp tới khi hết); lưu ý: unfolding KHÔNG chắc tạo query hiệu quả hơn, kích thước có thể tăng theo cấp số nhân | 61–66 |
| F. **Addressing Big Data challenges** (nối sang hướng nghiên cứu mới) | Vì sao tích hợp truyền thống không đủ (Deep Web scale, Schema Explosion, Keyword queries); **PAYGO** (Pay-as-you-go) & **Dataspace approach**; Probabilistic Mediated Schema & Probabilistic Mappings (xử lý uncertainty ở scale lớn); Keyword-search-based integration; WebTables keyword ranking | 69–86 |

---

## File 3 — `2_mediationquery.pdf` (bổ sung cho buổi 3) — 14 slide

**Chủ đề:** Generating mediation queries (mở rộng cho dữ liệu bán cấu trúc/XML)

| Cụm | Nội dung |
|---|---|
| A | Cách sinh mediation query tự động (semi-automatic) từ mapping quan hệ (ví dụ Films/Director) |
| B | Mở rộng cho dữ liệu semi/unstructured (cây XML): decompose mediation schema thành sub-trees (t1, t2, t3...) |
| C | Partial mappings: tìm phần tương ứng trong từng nguồn cho mỗi sub-tree, xác định phép join khả dĩ (dựa trên key/reference key) |
| D | Kết hợp partial mappings → sinh mediation query hoàn chỉnh dưới dạng XQuery |

---

## File 4 — `3_RecordLinkage_ER.pdf` (Buổi 4, **29/9 — gần nhất**) — 46 slide

**Chủ đề:** Record Linkage & Entity Resolution

| Cụm | Nội dung | Slide chính |
|---|---|---|
| A. Đặt vấn đề | Motivation (merge file, survey, dedupe mailing list); ví dụ "flight records"; Định nghĩa Record Linkage; Terminology (Entity Linking vs Entity Resolution — ER là trường hợp đặc biệt của Entity Linking = dedupe); Problem formalization: match/non-match/possible-link | 2–6 |
| B. **Similarity Measures — trọng tâm kỹ thuật** | **Edit Distance** (Levenshtein, tính bằng DP, O(\|x\|\|y\|)); **Set-based**: Overlap O(x,y)=\|Bx∩By\|, **Jaccard** J(x,y)=\|Bx∩By\|/\|Bx∪By\|, **TF/IDF** (tf, idf=N/Nd, feature vector, cosine similarity), **Generalized Jaccard** (bipartite matching cho token gần giống chứ không cần trùng tuyệt đối), **Soft TF/IDF** (kết hợp TF/IDF với similarity mềm ở mức token); **Phonetic**: Soundex (mã hoá theo âm đọc, 4 bước, có điểm yếu với tên gốc Á Đông vì bỏ qua nguyên âm) | 7–33 |
| C. **Entity Resolution Pipeline** | 4 bước: **Prepare Data** (schema matching, normalization, cleaning) → **Blocking/Sorting** (Naive all-pairs O(n²) vs Blocking/Partitioning vs Sorted Neighborhood vs LSH) → **Matching** (threshold-based đơn giản; Learned Matchers — traditional ML 2 pha, Deep Learning: DeepER/Magellan; vấn đề Labeled Data khan hiếm → Transfer Learning, Active Learning) → **Clustering** (Connected Components/transitive closure, Correlation Clustering, Markov Clustering) | 34–43 |
| D. Mở rộng | Incremental Data Deduplication (cập nhật kết quả linkage khi có dữ liệu mới, nhanh hơn tính lại từ đầu — greedy update: connect/merge/split/move) | 44 |

---

## Liên kết giữa 4 file (bức tranh tổng thể của môn học tới nay)

```
[Buổi 2] Data Integration overview
      → 6 mức dị biệt: 1.system 2.syntactic 3.structural/schema 4.semantic
                        5.instance/identity 6.temporal 7.quality
      → Task pipeline: Schema Alignment → Entity Resolution → Data Fusion
                              │                    │
[Buổi 3] ─────────────────────┘                    │
   Giải quyết dị biệt #3 (structural/schema) và #4 (semantic)
   bằng GAV/LAV/GLAV + certain answers + query unfolding/rewriting
                                                     │
[Buổi 4, gần nhất] ──────────────────────────────────┘
   Giải quyết dị biệt #5 (instance/identity)
   bằng Similarity Measures + pipeline Blocking→Matching→Clustering
```

Đây chính là lý do 2 buổi liên tiếp (Schema Alignment và Record Linkage) khớp trực tiếp với đề tài Project của nhóm bạn (tích hợp dữ liệu khí tượng đa nguồn — cũng cần Mediated Schema/GAV-LAV và Record Linkage giữa trạm điểm với ô lưới).

---

Quiz cho buổi gần nhất (Record Linkage & Entity Resolution) nằm ở file `Quiz-Buoi4-RecordLinkage-ER.md` cùng thư mục.
