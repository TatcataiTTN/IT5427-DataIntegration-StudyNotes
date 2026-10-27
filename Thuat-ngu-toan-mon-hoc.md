# Bảng chú giải thuật ngữ — IT5427 (tổng hợp từ cả 4 file slide)

Tra cứu từ toàn bộ nội dung đã học: `1. Introduction.pdf` (buổi 2), `2_SchemaAlignment.pdf` + `2_mediationquery.pdf` (buổi 3), `3_RecordLinkage_ER.pdf` (buổi 4). Mỗi thuật ngữ có giải thích ngắn gọn bằng tiếng Việt + nguồn slide để tra lại khi cần.

---

## A. Buổi 2 — Data Integration overview

| Thuật ngữ (EN) | Giải thích |
|---|---|
| **Data Integration** | Tích hợp dữ liệu — tạo một lớp trừu tượng bậc cao hơn DBMS, cho phép truy cập thống nhất tới nhiều nguồn dữ liệu độc lập (khác vị trí, mô hình dữ liệu, ngữ nghĩa) |
| **Mediated Schema** | Lược đồ trung gian/thống nhất — "giao diện" duy nhất mà người dùng thấy khi truy vấn, che giấu sự khác biệt giữa các nguồn phía sau |
| **Semantic Mappings** | Ánh xạ ngữ nghĩa giữa mediated schema và schema của từng nguồn — "logic" kết nối 2 bên |
| **Wrapper** | Bộ chuyển đổi — nhận truy vấn, gửi tới nguồn dữ liệu thật, chuyển kết quả trả về đúng mô hình dữ liệu nội bộ (VD XML → tuple) |
| **Heterogeneity (7 mức độ)** | Dị biệt giữa các nguồn, slide chia làm 7 tầng: (1) System/platform, (2) Syntactic/format, (3) Structural/schema, (4) Semantic/conceptual, (5) Instance/identity, (6) Temporal/process, (7) Quality/provenance/governance |
| **Structural/Schema Heterogeneity** | Khác biệt về cách tổ chức bảng, khóa, độ chi tiết (granularity) — VD 1 bảng gộp vs. nhiều bảng tách |
| **Semantic/Conceptual Heterogeneity** | Khác biệt ý nghĩa thuật ngữ — đồng nghĩa (synonym), đa nghĩa (homonym), cần ontology/glossary để giải quyết |
| **Instance/Identity Heterogeneity** | Cùng 1 thực thể thế giới thực xuất hiện khác nhau ở các nguồn — giải quyết bằng Entity Resolution/Record Linkage |
| **Data Value & Representation Heterogeneity** | Khác biệt về đơn vị đo, thang đo, mã hoá giá trị (VD USD vs VNĐ, thang điểm 5 vs 10, "active" vs "A") |
| **Virtual Data Integration** | Kiến trúc "ảo" — dữ liệu vẫn ở nguồn gốc, chỉ truy cập lúc có truy vấn (query-time), qua wrapper |
| **Data Warehousing** | Kiến trúc "vật lý" — copy dữ liệu về 1 kho tập trung định kỳ (ETL), truy vấn trên bản copy, không làm phiền nguồn gốc |
| **Query Reformulation** | Viết lại truy vấn trên mediated schema thành truy vấn thật thực thi được trên các nguồn |
| **Query Optimizer** | Thành phần chọn kế hoạch thực thi vật lý (physical query plan) tối ưu cho truy vấn đã reformulate |
| **5V của Big Data** | Volume (khối lượng), Velocity (tốc độ), Variety (đa dạng), Veracity (độ tin cậy), Value (giá trị) |
| **Schema Alignment → Entity Resolution → Data Fusion** | 3 bước pipeline tổng thể giải quyết Big Data Integration — đúng khung 3 buổi học liên tiếp |

---

## B. Buổi 3 — Schema Alignment & Query Rewriting

### B.1. Khái niệm nền

| Thuật ngữ (EN) | Giải thích |
|---|---|
| **Schema Matching / Schema Alignment** | Tìm sự tương ứng (correspondence) giữa các phần tử của 2 schema khác nhau |
| **Schema Mapping** | Mô tả hình thức (công thức logic) quan hệ giữa mediated schema và schema nguồn — dùng để reformulate truy vấn |
| **Certain Answers** | Câu trả lời đúng trong **MỌI** instance khả dĩ của mediated schema nhất quán với dữ liệu nguồn + mapping — khái niệm cốt lõi để định nghĩa "câu trả lời đúng" khi có nhiều schema mapping khả dĩ |
| **Possible Instances** | Các instance của mediated schema tương thích với dữ liệu nguồn hiện có (có thể có nhiều hơn 1, tuỳ ngôn ngữ mapping) |
| **Open-world assumption** | Giả định "còn có thể có dữ liệu khác chưa biết" — dùng khi định nghĩa $G_i(\bar X) \supseteq Q(\bar S)$ |
| **Closed-world assumption** | Giả định "chỉ có đúng dữ liệu đã biết" — dùng khi định nghĩa $G_i(\bar X) = Q(\bar S)$ |

### B.2. Ba ngôn ngữ Schema Mapping

| Thuật ngữ (EN) | Giải thích |
|---|---|
| **GAV — Global-as-View** | Mediated schema được định nghĩa **là 1 view** trên các nguồn ($G \supseteq Q(sources)$). Reformulation = Query Unfolding (nhanh, đơn giản), nhưng cứng nhắc, không biểu diễn được thông tin không đầy đủ |
| **LAV — Local-as-View** | Mỗi nguồn được định nghĩa **là 1 view** trên mediated schema ($S_i \subseteq Q_i(G)$). Linh hoạt hơn (dễ thêm/bớt nguồn), nhưng reformulation khó hơn (= bài toán Answering Queries Using Views) |
| **GLAV** | Kết hợp GAV+LAV — cả 2 vế đều là truy vấn ($Q^S(\bar X) \subseteq Q^G(\bar X)$), biểu diễn bằng Tuple Generating Dependencies (TGD) |
| **Tuple Generating Dependency (TGD)** | Công thức logic dạng $\forall \bar X\, \phi(\bar X) \to \exists \bar Y\, \psi(\bar X,\bar Y)$ — cách hình thức khác để viết GLAV |
| **Query Unfolding** | Thuật toán reformulation trong GAV — thay thế subgoal bằng rule tương ứng, lặp tới khi hết. Không đảm bảo tạo query hiệu quả hơn, kích thước có thể tăng theo cấp số nhân |
| **Answering Queries Using Views** | Bài toán reformulation trong LAV — tìm cách trả lời truy vấn trên mediated schema chỉ dùng dữ liệu đã có ở các view (nguồn) |
| **Inverse Rules** | Kỹ thuật giải LAV — suy ngược từ định nghĩa view ra quy tắc suy luận dữ liệu gốc, rồi unfold như GAV |

### B.3. Data-level & mở rộng dữ liệu bán cấu trúc

| Thuật ngữ (EN) | Giải thích |
|---|---|
| **Data-level Heterogeneity** | Khác biệt cần **phép biến đổi tính toán** khi map, không chỉ đổi tên cột — VD Celsius↔Fahrenheit, giá có/không thuế |
| **Mediation Query (bán cấu trúc)** | Với dữ liệu XML/semi-structured: quy trình sinh truy vấn tự động bằng cách decompose mediation schema thành sub-trees, tìm partial mapping ở từng nguồn, rồi combine |
| **Partial Mapping** | Ánh xạ giữa 1 sub-tree của mediation schema với phần tương ứng ở 1 nguồn cụ thể |
| **Join Graph** | Đồ thị biểu diễn các phép join khả dĩ giữa các nguồn (node = nguồn, cạnh = điều kiện join dựa trên key/reference key) |

### B.4. Addressing Big Data Integration Challenges (mở rộng)

| Thuật ngữ (EN) | Giải thích |
|---|---|
| **PAYGO — Pay-As-You-Go Integration** | Triết lý tích hợp dần dần — bắt đầu với ít ràng buộc ngữ nghĩa, cải thiện mapping dần theo thời gian/phản hồi người dùng, thay vì thiết kế mediated schema hoàn chỉnh ngay từ đầu |
| **Dataspace / DSSP (Dataspace Support Platform)** | Mô hình quản lý dữ liệu thay thế tích hợp truyền thống khi có quá nhiều nguồn (hàng triệu) — không đòi hỏi mediated schema chính xác ngay, chấp nhận uncertainty ở mọi tầng |
| **Probabilistic Mediated Schema** | Mediated schema được tạo tự động bằng clustering thuộc tính nguồn, có nhiều phiên bản khả dĩ kèm xác suất — do Volume/Variety lớn không đủ thời gian thiết kế thủ công |
| **Probabilistic Mapping** | Ánh xạ schema có gắn xác suất — nhiều mapping khả dĩ, mỗi cái 1 trọng số tin cậy |
| **WebTables / Keyword-search-based Integration** | Hướng tích hợp không cần mediated schema tường minh — tìm/ghép bảng trên Web bằng keyword + ranking theo độ liên quan (FeatureRank, SchemaRank) |
| **Uncertainty at All Levels** | 1 trong 2 ý tưởng cốt lõi của PAYGO — mô hình hoá độ không chắc chắn thống nhất ở cả dữ liệu, mapping, và truy vấn |

---

## C. Buổi 4 — Record Linkage & Entity Resolution

### C.1. Khái niệm nền & Terminology

| Thuật ngữ (EN) | Giải thích |
|---|---|
| **Record Linkage** | Tìm các cặp bản ghi ở 2 file khác nhau đại diện cho cùng 1 thực thể |
| **Entity Resolution (ER) / Deduplication** | Trường hợp đặc biệt của Entity Linking — tìm/gộp các bản ghi trùng lặp (có thể trong cùng 1 file) |
| **Entity Linking** | Khái niệm rộng hơn ER — tạo liên kết giữa các bản ghi đại diện cho các thực thể có liên quan (không nhất thiết "giống nhau") |
| **Match / Non-match / Possible-match** | 3 kết luận khi so khớp 1 cặp bản ghi (Fellegi & Sunter 1969) — "possible-match" nghĩa là chưa đủ bằng chứng kết luận chắc chắn |
| **Naïve Deduplication** | Cách khử trùng lặp đơn giản bằng UNION DISTINCT/group-by — chỉ bắt được exact match |

### C.2. Similarity Measures

| Thuật ngữ (EN) | Giải thích |
|---|---|
| **Similarity Measure $s(x,y)$** | Hàm đo độ giống nhau, giá trị $\in [0,1]$, càng cao càng giống |
| **Distance/Cost Measure** | Ngược lại similarity — càng nhỏ càng giống |
| **Edit Distance (Levenshtein Distance)** | Số phép biến đổi tối thiểu (chèn/xoá/thay) để đổi chuỗi x thành y — tính bằng Dynamic Programming, $O(|x||y|)$ |
| **q-grams** | Chuỗi con độ dài q, dùng làm token khi áp dụng similarity measure dạng tập hợp cho văn bản |
| **Overlap Measure** | $O(x,y) = |B_x \cap B_y|$ — số token chung giữa 2 tập token |
| **Jaccard Measure** | $J(x,y) = |B_x \cap B_y| / |B_x \cup B_y|$ — tỷ lệ phần chung trên tổng, rất phổ biến |
| **TF/IDF Measure** | Đo similarity có trọng số theo độ hiếm của từ (Term Frequency × Inverse Document Frequency) — nhận diện đúng từ "phân biệt" (VD "Apple") quan trọng hơn từ phổ biến (VD "Corporation") |
| **Feature Vector** | Biểu diễn 1 chuỗi thành vector số theo TF/IDF của từng term, dùng cosine similarity để so 2 vector |
| **Generalized Jaccard Measure** | Mở rộng Jaccard — cho phép token "gần giống" (không cần trùng tuyệt đối) vào tập overlap, dùng similarity threshold + maximum-weight bipartite matching |
| **Soft TF/IDF Measure** | Kết hợp TF/IDF với similarity mềm ở mức token — xử lý được lỗi chính tả nhẹ trong khi vẫn giữ trọng số theo độ hiếm |
| **Phonetic Similarity / Soundex** | So khớp theo cách phát âm thay vì ký tự — thuật toán Soundex mã hoá tên thành 1 chữ + 3 số, có điểm yếu với tên gốc Á Đông (bỏ qua nguyên âm) |
| **FindCands** | Phương pháp tìm tập ứng viên $Z$ khả dĩ khớp cho 1 chuỗi $x$, tránh phải so toàn bộ cặp $O(|X||Y|)$ |

### C.3. Entity Resolution Pipeline

| Thuật ngữ (EN) | Giải thích |
|---|---|
| **Data Preparation** | Bước 1 — Schema Matching/Mapping, Normalization (loại ký tự đặc biệt, stemming, viết hoa/thường), Data Cleaning |
| **Blocking / Sorting** | Bước 2 — giảm số cặp cần so khớp từ $O(n^2)$ xuống thấp hơn nhiều |
| **Naïve All-Pairs** | So khớp mọi cặp có thể — $O(n^2)$, không khả thi ở quy mô lớn |
| **Blocking / Partitioning** | Nhóm bản ghi có khả năng khớp cao vào cùng "block" dựa trên blocking key (VD cùng tên) trước khi so khớp trong block |
| **Sorted Neighborhood** | Sắp xếp toàn bộ bản ghi theo sorting key, chỉ so khớp trong 1 cửa sổ trượt (sliding window) kích thước cố định |
| **Locality-Sensitive Hashing (LSH)** | Blocking bằng cách hash embedding của bản ghi — các bản ghi gần nhau trong không gian embedding rơi vào cùng bucket với xác suất cao |
| **Matching** | Bước 3 — quyết định 1 cặp có khớp hay không, dựa trên threshold similarity hoặc mô hình học máy |
| **Learned Matchers** | Dùng ML (truyền thống: SVM/decision tree/random forest/XGBoost; hoặc Deep Learning: DeepER, Magellan) thay vì luật thủ công để quyết định match |
| **Labeled Data / Class Skew** | Vấn đề khi huấn luyện Learned Matchers — dữ liệu gán nhãn khan hiếm và số cặp match ít hơn nhiều so với non-match |
| **Transfer Learning (trong ER)** | Học mô hình từ kịch bản có nhiều dữ liệu, fine-tune bằng ít nhãn ở kịch bản mới |
| **Active Learning (trong ER)** | Chủ động chọn cặp cần gán nhãn để tối ưu hiệu quả huấn luyện với ít nhãn nhất |
| **Clustering** | Bước 4 — gộp các cặp match đã xác định thành các cụm, mỗi cụm = 1 thực thể thống nhất |
| **Connected Components / Transitive Closure** | Clustering đơn giản nhất — coi các bản ghi nối với nhau qua cạnh similarity là 1 cụm; nhược điểm: dễ tạo cụm quá lớn gồm bản ghi không thực sự giống nhau |
| **Correlation Clustering** | Clustering dựa trên cả cạnh dương (+) lẫn âm (−), tối ưu toàn cục — NP-hard |
| **Markov Clustering** | Clustering bằng mô phỏng luồng ngẫu nhiên (stochastic flow simulation) trên đồ thị |

### C.4. Mở rộng

| Thuật ngữ (EN) | Giải thích |
|---|---|
| **Incremental Data Deduplication** | Cập nhật kết quả linkage khi có dữ liệu mới, nhanh hơn tính lại toàn bộ — dùng greedy update (connect/merge/split/move) trên similarity graph đã có |
| **Data Fusion** | Bước sau Entity Resolution trong pipeline tổng thể (buổi 2) — quyết định giá trị "thắng" khi các bản ghi đã gộp có thông tin xung đột (= Survivorship Rule, áp dụng cụ thể trong project nhóm) |

---

## Ghi chú sử dụng

- Thuật ngữ **in đậm trong Giải thích** là các khái niệm bạn cần nắm chắc nhất để trả lời phản biện, vì đây chính là các thuật ngữ đã được project của nhóm áp dụng cụ thể (xem `IT5427-Nhom16-TichHopKhiTuong/docs/06-truong-du-lieu-va-tich-hop.md` và `07-bang-va-giai-phap-tich-hop.md`).
- File này bổ sung cho `Tom-tat-phan-cum-4-file-slide.md` (đã có ở cùng thư mục) — file đó tóm tắt theo **cấu trúc từng buổi học**, file này tra cứu theo **từng thuật ngữ**.
