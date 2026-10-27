# Giải thích chi tiết từng trang — Slide "1. Introduction.pdf"

Nguồn: `slides/1. Introduction.pdf` — bản PDF in dạng "2 slide/trang" (23 trang in = 46 slide trình
chiếu gốc). Giải thích dưới đây đi theo **số thứ tự slide gốc** (1-46, đúng số nhỏ in ở góc mỗi slide
trong PDF), không phải số trang PDF, vì đó mới là đơn vị nội dung thật sự của bài giảng. Đối chiếu
bằng `pdftotext -layout` + đọc trực tiếp ảnh render từng trang (150 dpi) để không bỏ sót chi tiết
hình/sơ đồ.

Toàn bộ 46/46 slide đã được xem và giải thích — không bỏ qua slide nào, kể cả slide chỉ có tiêu đề
hoặc chỉ có hình/sơ đồ không chữ.

---

## PHẦN MỞ ĐẦU

### Slide 1 — Trang bìa
"Big Data Integration & Processing" — Vu Tuyet Trinh, SoICT - HUST. Đây là slide tiêu đề môn học,
không có nội dung kỹ thuật.

### Slide 2 — Outline (mục lục bài giảng)
Liệt kê 6 phần sẽ học, theo đúng thứ tự trình bày trong cả file: (1) Data integration as a new
abstraction — tích hợp dữ liệu là một tầng trừu tượng hoá MỚI so với DBMS truyền thống; (2) Examples
of data integration applications; (3) Heterogeneity — các loại dị biệt dữ liệu; (4) Goal of data
integration; (5) Data integration architectures; (6) Big Data Integration - Challenges. Mục lục này
sẽ xuất hiện lại ở slide 28 (đã gạch dấu ✓ các mục đã học, chuyển sang phần Architecture) và slide 37
(gạch hết, chỉ còn phần Challenges) — đúng kỹ thuật "outline nhắc lại" kinh điển trong bài giảng dài.

---

## PHẦN 1 — Data Integration là một tầng trừu tượng hoá mới

### Slide 3 — Data Integration
Nôm na: CSDL (database) vốn được coi là công cụ mạnh để lưu trữ/truy vấn dữ liệu, NHƯNG điều đó chỉ
đúng NẾU toàn bộ dữ liệu đã nằm sẵn trong 1 lược đồ (schema) duy nhất. Vấn đề thực tế: các tập dữ liệu
thường được tạo ra ĐỘC LẬP bởi nhiều nhóm/người khác nhau, dùng hệ thống khác nhau, lược đồ khác nhau,
và giao diện truy cập hạn chế — rồi sau đó mới phát hiện ra cần phải kết hợp chúng lại. Định nghĩa cốt
lõi của cả môn học nằm ở slide này: **"Data integration: tie together different sources, controlled
by many people, under a common schema"** — tích hợp dữ liệu là việc GẮN KẾT nhiều nguồn dữ liệu (do
nhiều người/tổ chức khác nhau quản lý) lại dưới MỘT lược đồ chung.

### Slide 4 — DBMS: it's all about abstraction
Nhắc lại kiến thức nền CSDL: DBMS hoạt động được là nhờ tách biệt 2 tầng — **Logical** (người dùng chỉ
cần biết "cái gì" cần truy vấn, ví dụ viết SQL) và **Physical** (hệ thống tự lo "làm thế nào" để lưu
trữ/truy xuất thật). Ví dụ minh hoạ: 3 bảng `Students`, `Takes`, `Courses` và 1 câu SQL JOIN cả 3 bảng
để tìm các khoá học của sinh viên tên "Mary". Ý nghĩa sư phạm: Data Integration sẽ lặp lại Y HỆT ý
tưởng trừu tượng hoá này, nhưng ở MỨC CAO HƠN — không chỉ trừu tượng hoá vật lý↔logic trong 1 CSDL,
mà trừu tượng hoá NHIỀU nguồn dữ liệu khác nhau thành 1 giao diện truy vấn chung (xem slide 5).

### Slide 5 — Data Integration: A Higher-level Abstraction
Đây là slide trung tâm của cả phần mở đầu, có sơ đồ quan trọng: **Query** (truy vấn người dùng) đi vào
1 **Mediated Schema** (lược đồ trung gian, vẽ hình oval) ở trên cùng; bên dưới, 3 nguồn dữ liệu khác
hẳn nhau về định dạng — **S1** (1 bảng quan hệ, có cột SSN/Name/Category), **S2** (dữ liệu XML, ví dụ
`<cd><title>...</title><artist>Carreras</artist>...</cd>`), **S3** (1 bảng tính dạng spreadsheet) —
được nối lên Mediated Schema qua các mũi tên **"Semantic Mappings"**. Thông điệp: nhờ tầng Mediated
Schema + Semantic Mappings này, người dùng viết 1 truy vấn DUY NHẤT mà KHÔNG cần biết: (a) nguồn dữ
liệu nằm ở đâu (source & location), (b) data model/cú pháp của từng nguồn khác nhau thế nào, (c) các
biến thể ngữ nghĩa (semantic variations) giữa các nguồn — đúng 3 gạch đầu dòng "Independence of" ghi
bên phải slide. Đây chính là bản chất của "trừu tượng hoá mức cao hơn" đã giới thiệu ở slide 4.

### Slide 6 — Applications of Data Integration
Slide chuyển cảnh (transition), liệt kê 5 lĩnh vực ứng dụng sẽ được minh hoạ ở các slide tiếp theo:
Business, Science, Government, The Web, và "Pretty much everywhere" (gần như ở khắp mọi nơi) — ý nói
Data Integration không phải bài toán riêng của 1 ngành, mà là nhu cầu phổ quát bất cứ đâu có nhiều hơn
1 nguồn dữ liệu.

### Slide 7 — Application Area 1: Business
Sơ đồ: bên trái là **Enterprise Databases** (nhiều hình trụ = nhiều CSDL doanh nghiệp) và **Legacy
Databases**/**Services and Applications** (hệ thống cũ + dịch vụ), tất cả gom vào 1 ô **"Single
Mediated View"** ở giữa, rồi từ đó toả ra các **EII Apps** (Enterprise Information Integration
Applications) bên phải: CRM, ERP, Portals, v.v. Dòng chữ lớn cuối slide **"50% of all IT $$$ spent
here!"** — số liệu nhấn mạnh rằng một nửa ngân sách CNTT doanh nghiệp được chi cho chính bài toán tích
hợp dữ liệu (không phải xây hệ thống mới), cho thấy quy mô kinh tế thực sự của môn học này.

### Slide 8 — Application Area 2: Science
Minh hoạ lĩnh vực khoa học (sinh — y, cụ thể là tin sinh học/bioinformatics): sơ đồ mạng lưới liên kết
các khái niệm (Phenotype, Gene, Sequenceable Entity, Protein, Nucleotide Sequence, Structured
Vocabulary, Experiment, Microarray Experiment) với các CSDL chuyên ngành thật (OMIM, HUGO, Swiss-Prot,
GO, Gene-Clinics, Locus-Link, Entrez, GEO) — mỗi CSDL là 1 nguồn độc lập do 1 tổ chức khoa học khác
nhau duy trì. Dòng chú thích: **"Hundreds of biomedical data sources available; growing rapidly!"** —
hàng trăm nguồn dữ liệu y-sinh đang tồn tại và tăng nhanh, nên bài toán tích hợp ở đây cực kỳ cấp
thiết trong thực tế nghiên cứu khoa học.

### Slide 9 — Application Area 3: The Web
Chỉ có hình (không chữ ngoài tiêu đề): tập hợp logo của hàng chục website bán sách khác nhau (Virtual
Bookstore, amazon.com, Borders.com, Barnes & Noble, v.v.). Ý nghĩa: minh hoạ trực quan cho việc có RẤT
NHIỀU nguồn dữ liệu web (ở đây là các site bán sách) cùng chứa thông tin tương tự nhưng mỗi site có
cấu trúc/giao diện riêng — tiền đề cho khái niệm "Deep Web" ở slide 11.

### Slide 10 — Hundreds of millions of high-quality tables on the Web
Ảnh chụp màn hình 1 trang web thật (EnchantedLearning.com — bảng danh sách các đời Tổng thống Mỹ, có
cột President/Party/Term/Vice-President) với dòng chữ nổi bật đè lên: **"Hundreds of millions of
high-quality tables on the Web"**. Thông điệp: web không chỉ có văn bản tự do, mà còn chứa HÀNG TRĂM
TRIỆU bảng dữ liệu có cấu trúc (HTML tables) — 1 nguồn "dữ liệu lớn" khổng lồ và ít được khai thác, là
động lực cho các kỹ thuật trích xuất bảng tự động (WebTables) sẽ gặp lại ở phần nghiên cứu khoa học
của môn học.

### Slide 11 — The Deep Web
Giới thiệu khái niệm "Deep Web" (web sâu, đối lập với web bề mặt mà công cụ tìm kiếm crawl được dễ
dàng): có HÀNG TRIỆU form HTML chất lượng cao (ô tìm kiếm, bộ lọc...) ngoài kia, mỗi form có giao diện
RIÊNG BIỆT của nó → rất khó khám phá/truy vấn dữ liệu XUYÊN NHIỀU site cùng lúc. Mục tiêu đặt ra (cho
1 số lĩnh vực nhất định): xây 1 giao diện DUY NHẤT để truy vấn được nhiều nguồn deep-web khác nhau —
đây chính là 1 bài toán Data Integration cụ thể.

### Slide 12 — (ảnh minh hoạ, không có tiêu đề riêng)
2 ảnh chụp màn hình form tìm việc thật: Yahoo! HotJobs ("Search for Jobs Across the Web" — ô Keyword,
City & State, Job Category) và USAJOBS.gov ("Keyword Search", "Location Search", "Job Category
Search", "Salary Range"). Đây là ví dụ cụ thể cho khái niệm "mỗi form có giao diện riêng" vừa nêu ở
slide 11 — 2 form tìm việc có mục đích giống hệt nhau nhưng cấu trúc ô nhập liệu hoàn toàn khác nhau.

### Slide 13 — Create a single site to search for jobs/rentals/…
Ảnh chụp trang EveryClassified.com thật — 1 site tổng hợp (mediator) thật sự được xây để giải quyết
đúng vấn đề ở slide 11-12: có 6 khối tìm kiếm con (Autos, Rentals, Real Estate, Jobs, Personals,
General), mỗi khối gom kết quả từ NHIỀU site chuyên biệt khác nhau vào 1 giao diện tìm kiếm chung duy
nhất. Đây là ví dụ THỰC TẾ (không phải sơ đồ lý thuyết) của kiến trúc Mediated Schema đã học ở slide 5.

### Slide 14 — Easily traverse between the site by clicking its name
Ảnh chụp tiếp theo của EveryClassified: sau khi gõ tìm kiếm ô tô (Acura Integra), trang hiển thị kết
quả gộp từ nhiều site con (MercuryNews, Cars.com...) kèm danh sách các site nguồn bên trái (Yahoo
Autos, Cars.com, MercuryNews, Craigslist...) — minh hoạ việc người dùng có thể dễ dàng "đi xuyên" giữa
các site nguồn gốc chỉ bằng cách bấm vào tên site, dù dữ liệu đã được tích hợp hiển thị chung 1 nơi.

### Slide 15 — Baomoi.com
Ảnh chụp trang Báo Mới (baomoi.com) — 1 ví dụ Việt Nam: đây là trang TỔNG HỢP TIN TỨC, gom bài viết từ
nhiều báo điện tử khác nhau (Tin Tức TTXVN, Tổ Quốc, VNEWS...) về 1 giao diện duy nhất. Giảng viên
chèn ví dụ nội địa này để học viên liên hệ trực tiếp với 1 sản phẩm Data Integration họ dùng hằng
ngày, không chỉ xem ví dụ nước ngoài trừu tượng.

### Slide 16 — Heterogeneity
Slide mở đầu phần "Dị biệt dữ liệu" (Heterogeneity) — phần lõi lý thuyết quan trọng nhất của chương
Introduction. Sơ đồ: 3 nguồn (CRM, ERP, Web logs, vẽ hình trụ) đi qua 1 đám mây **"heterogeneity
barrier"** (rào cản dị biệt) để tạo ra **"Unified view — clean, linked, queryable"** (view thống nhất
— sạch, đã liên kết, truy vấn được) bên phải. Bên dưới liệt kê 5 NGUYÊN NHÂN cụ thể gây ra rào cản dị
biệt: Duplicate entities (thực thể trùng lặp), Conflicting values (giá trị mâu thuẫn), Ambiguous
meaning (ý nghĩa mập mờ), Different update times (thời điểm cập nhật khác nhau), Trust & access rules
(quy tắc tin cậy & quyền truy cập). Đây là bản tóm tắt NGẮN của 8 loại dị biệt sẽ được giảng viên "mổ
xẻ" chi tiết từng loại ở các slide 18-25 tiếp theo.

### Slide 17 — Levels of heterogeneity
Bảng phân loại chính thức, 6 MỨC dị biệt dữ liệu (đánh số 1-6), mỗi mức có 1 câu mô tả ngắn:

| # | Mức dị biệt | Mô tả |
|---|---|---|
| 1 | System / platform | Dữ liệu được lưu/truy cập Ở ĐÂU và NHƯ THẾ NÀO |
| 2 | Syntactic / format | Định dạng file, bảng mã, cú pháp ngày tháng/số |
| 3 | Structural / schema | Bảng, thuộc tính, khoá, lồng ghép, độ chi tiết (granularity) |
| 4 | Semantic / conceptual | Ý nghĩa của thuật ngữ, đơn vị đo, định nghĩa nghiệp vụ |
| 5 | Instance / identity | Cùng 1 đối tượng thực tế xuất hiện khác nhau giữa các nguồn |
| 6 | Quality / governance | Tính đầy đủ, nguồn gốc (provenance), độ tin cậy, quyền hạn |

**⚠️ Ghi chú đối chiếu quan trọng:** bảng này của giảng viên liệt kê CHÍNH THỨC 6 mức, nhưng các slide
minh hoạ chi tiết theo sau (18→25) lại trình bày **8 slide ví dụ** — vì mức 4 "Semantic/conceptual"
được tách thành 2 slide minh hoạ riêng (slide 21: ý nghĩa thuật ngữ/khái niệm, và slide 22: giá trị dữ
liệu/đơn vị đo — về bản chất cũng là 1 dạng semantic heterogeneity nhưng ở mức GIÁ TRỊ thay vì mức KHÁI
NIỆM), và có thêm 1 slide "Temporal and process heterogeneity" (slide 24) KHÔNG xuất hiện như 1 hàng
riêng trong bảng 6 mức này. Đây không phải lỗi — chỉ là cách giảng viên mở rộng ví dụ dạy học rộng hơn
khung phân loại chính thức một chút; nên khi ôn thi, có thể ghi nhớ bảng 6 mức làm GỐC, và coi
"Temporal/process" + "Data value" là các ví dụ MINH HOẠ SÂU HƠN cho mức Semantic/Structural, không
phải 1 mức độc lập thứ 7-8 chính thức.

### Slide 18 — System and platform heterogeneity (mức 1)
Sơ đồ: 4 loại hệ thống lưu trữ khác nhau ở trên — **Relational DB** (giao tiếp qua SQL, JDBC/ODBC),
**NoSQL store** (Document API, JSON), **Message bus** (Streaming, Kafka topics), **Files/lake**
(Parquet/CSV, Object storage) — tất cả đi qua 1 tầng **"Integration middleware / wrappers /
connectors"** để tạo ra **"Common access layer"** (tầng truy cập chung, hỗ trợ query/ingest/metadata/
governance). Dòng cuối liệt kê các PHƯƠNG PHÁP KỸ THUẬT thường dùng để giải quyết dị biệt mức này:
adapters, APIs, virtualization, CDC (Change Data Capture), schema registry.

### Slide 19 — Syntactic and format heterogeneity (mức 2)
Ví dụ cụ thể 2 nguồn có CÙNG Ý NGHĨA dữ liệu nhưng khác CÚ PHÁP biểu diễn: Source A ghi
`"2026-09-15"`/`"HUST"`/`"1,200.50"`, Source B ghi `"15/09/26"`/`"ĐHBKHN"`/`"1.200,50"` (chú ý: dấu
phẩy/chấm thập phân bị ĐẢO NGƯỢC giữa 2 nguồn — bẫy kinh điển khi tích hợp dữ liệu quốc tế). Sau bước
**"Parsing & standardization"** (phân tích cú pháp & chuẩn hoá), cả 2 được đưa về 1 dạng thống nhất:
`date = 2026-09-15`, `name = Hanoi University of Science and Technology`, `amount = 1200.50`. Dòng chú
thích liệt kê các xung đột cú pháp thường gặp: loại file, bảng mã (encoding), ký tự phân cách
(delimiter), định dạng ngày, dấu phân cách thập phân, viết tắt, ngôn ngữ.

### Slide 20 — Structural and schema heterogeneity (mức 3)
**Đây chính là ví dụ gốc mà bài tập BT.pdf (Bài 2 — GAV/LAV) đã dùng lại gần như nguyên văn** — chỉ
đổi tên thuộc tính một chút: Source S1 có `CourseDescription | Faculty | Rating | TuitionFee`, S2 có
`CourseName | School`, S3 có `CourseName | Rating | TuitionFee`. Qua bước **"Schema matching + mapping
rules"** (khớp lược đồ + luật ánh xạ), 3 nguồn được gộp vào 1 **Integrated schema S**:
`CourseDescription, Faculty, Rating, TuitionFee`. Dòng chú thích liệt kê các loại xung đột cấu trúc:
thiếu thuộc tính, chuẩn hoá (normalization) khác nhau, lồng ghép (nested) vs phẳng (flat), lệch khoá,
độ chi tiết (granularity) khác nhau. **Liên hệ trực tiếp:** đây là lý do vì sao bài tập
`BT-EditDistance-TFIDF-SchemaMapping.md` (Bài 2) dùng đúng mô hình 3 nguồn S1/S2/S3 này để luyện viết
mapping GAV/LAV — bài tập đó chính là bài thực hành cho đúng khái niệm ở slide này.

### Slide 21 — Semantic and conceptual heterogeneity (mức 4a — ý nghĩa khái niệm)
Sơ đồ cây khái niệm: "Course" có thể liên hệ tới "CourseName", "Course Description", "Program",
"Module", "Faculty"/"School" — minh hoạ 2 hiện tượng ngôn ngữ học kinh điển trong tích hợp dữ liệu:
**Synonyms** (từ đồng nghĩa, ví dụ "Faculty" ≈ "School" — 2 TÊN khác nhau nhưng CÙNG Ý NGHĨA) và
**Homonyms** (từ đồng âm khác nghĩa, ví dụ từ "course" có thể nghĩa là 1 môn học, 1 chương trình đào
tạo, HOẶC 1 lộ trình bằng cấp — CÙNG 1 TỪ nhưng NHIỀU Ý NGHĨA khác nhau tuỳ ngữ cảnh). Chú thích dưới
cùng: việc xử lý dạng dị biệt này cần kiến thức miền (domain knowledge) — ontology, bộ từ vựng kiểm
soát (controlled vocabulary), từ điển thuật ngữ nghiệp vụ (business glossary), hoặc thẩm định bởi
chuyên gia.

### Slide 22 — Data value and representation heterogeneity (mức 4b — giá trị dữ liệu)
Bảng ví dụ rất cụ thể, nên nhớ kỹ vì hay ra đề:

| Attribute | Source A | Source B | Integration action |
|---|---|---|---|
| TuitionFee | 1200 USD | 28.800.000 VND | currency conversion (quy đổi tiền tệ) |
| Rating | 4.7/5 | 92/100 | scale normalization (chuẩn hoá thang đo) |
| Status | active | A | code dictionary (từ điển mã hoá) |
| Missing | NULL | N/A / 999 | missing-value policy (chính sách xử lý giá trị thiếu) |

Mục tiêu nêu rõ ở cuối slide: mọi giá trị phải được quy về CÙNG 1 kiểu dữ liệu, đơn vị, thang đo, và
cách diễn giải đã thống nhất. Kỹ thuật thường dùng: data profiling (khảo sát dữ liệu), type inference
(suy luận kiểu), unit conversion (quy đổi đơn vị), code mapping (ánh xạ mã), outlier checks (kiểm tra
giá trị bất thường), standard reference data (dữ liệu tham chiếu chuẩn).

### Slide 23 — Instance and identity heterogeneity (mức 5)
Đây chính là nền tảng lý thuyết cho cả Module "Record Linkage/Entity Resolution" (slide
`3_RecordLinkage_ER.pdf`) sẽ học sau này! Ví dụ: Record A (`ID: U001, Name: HUST, City: Hanoi`) và
Record B (`ID: 3872, Name: ĐHBK Hà Nội, City: Ha Noi`) — 2 bản ghi có ID khác hẳn và tên viết khác hẳn
nhau, nhưng cùng ÁM CHỈ 1 thực thể thật ngoài đời (Đại học Bách khoa Hà Nội). Qua pipeline **"Entity
resolution / record linkage / deduplication"**, cả 2 được hợp nhất thành 1 **Canonical entity** (thực
thể chuẩn hoá): "Hanoi University of Science and Technology", Aliases: HUST/ĐHBK Hà Nội, Confidence:
0.96. Sơ đồ dưới liệt kê đúng 3 bước của pipeline Entity Resolution: **blocking** (lọc sơ bộ cặp ứng
viên) → **similarity** (đo độ tương đồng) → **classification** (phân loại trùng/không trùng) → cuối
cùng **merge/survivorship** (hợp nhất + chọn giá trị nào "thắng" khi có mâu thuẫn).

### Slide 24 — Temporal and process heterogeneity
Minh hoạ dòng thời gian trong 1 ngày: `08:00 CRM update` → `10:00 ERP batch` → `12:00 web event` →
`18:00 warehouse load` → `next day reporting` — các hệ thống khác nhau CẬP NHẬT dữ liệu vào các THỜI
ĐIỂM khác nhau trong ngày. 3 khái niệm liên quan: **Freshness** (độ mới — liên hệ Snapshot consistency
và Late-arriving data/dữ liệu đến muộn), và **Versioning** (quản lý phiên bản). Ô "Design choice" ở
cuối slide nêu 1 quyết định thiết kế quan trọng: hệ thống tích hợp phải CHỌN RÕ truy vấn sẽ dùng "dữ
liệu mới nhất hiện có" (latest available data) hay 1 bản chụp nhất quán theo mốc thời gian (consistent
time-based snapshot) — 2 lựa chọn này cho kết quả truy vấn khác nhau nếu dữ liệu nguồn cập nhật lệch
giờ nhau.

### Slide 25 — Quality, provenance, and governance heterogeneity (mức 6)
Câu mở đầu quan trọng: "Tích hợp không chỉ là gộp dữ liệu lại — nó còn phải BẢO TOÀN độ tin cậy và các
ràng buộc sử dụng." Sơ đồ hoa thị quanh "Integrated data product" ở giữa: **Completeness** (tính đầy
đủ — thiếu dòng/cột nào không), **Lineage** (phả hệ — đã qua những phép biến đổi nào), **Provenance**
(nguồn gốc — dữ liệu đến từ đâu), **Accuracy** (độ chính xác — có sự kiện sai/cũ không),
**Survivorship** (khi 2 nguồn mâu thuẫn, nguồn nào "thắng"), **Access rules** (ai được phép dùng dữ
liệu này). Câu kết luận đáng nhớ: "Một phép tích hợp ĐÚNG VỀ KỸ THUẬT vẫn có thể KHÔNG DÙNG ĐƯỢC nếu
người dùng không thể đánh giá được độ tin cậy, phả hệ, và tuân thủ quy định của dữ liệu."

### Slide 26 — Goal of Data Integration
Phát biểu lại mục tiêu cốt lõi sau khi đã học hết 6 loại dị biệt: cung cấp 1 TRUY CẬP TRUY VẤN THỐNG
NHẤT (uniform query access) tới 1 tập hợp các nguồn dữ liệu, đồng thời phải XỬ LÝ ĐƯỢC: (a) SỐ LƯỢNG
nguồn — từ vài chục tới HÀNG TRIỆU nguồn, và (b) các MỨC DỊ BIỆT khác nhau (chính là 6 mức vừa học).

### Slide 27 — Data Integration issues
Sơ đồ hoa thị (infographic) tóm tắt 8 vấn đề kỹ thuật cụ thể khi triển khai thực tế, xoay quanh tâm
"DATA INTEGRATION": Data Modeling (mô hình hoá dữ liệu), Legacy & 3rd Party Interface (giao diện hệ
thống cũ/bên thứ 3), Data Warehousing, ETLT Design & Development, Data Profiling (khảo sát dữ liệu),
Data Cleansing (làm sạch dữ liệu), (un-)structured Data (dữ liệu có/không cấu trúc), Data Merging
Migration & Replication (gộp/di chuyển/nhân bản dữ liệu). Đây là góc nhìn THỰC HÀNH/CÔNG NGHIỆP, bổ
sung cho góc nhìn LÝ THUYẾT (6 mức dị biệt) đã học ở các slide trước.

### Slide 28 — Outline (nhắc lại lần 2)
Mục lục được lặp lại với dấu ✓ (tick) trước 4 mục ĐÃ học (Data integration as a new abstraction,
Examples, Heterogeneity, Goal), mũi tên ➢ chỉ vào mục SẮP học ("Data integration architectures"), và
mục cuối "Big Data Integration - Challenges" vẫn ở dạng gạch đầu dòng thường (chưa tới). Đánh dấu rõ
ràng vị trí hiện tại trong bài giảng trước khi chuyển sang phần kiến trúc hệ thống.

---

## PHẦN 2 — Data Integration Architectures (Kiến trúc hệ thống tích hợp)

### Slide 29 — Virtual, Warehousing and in Between
Giới thiệu 2 kiến trúc tích hợp dữ liệu CHÍNH (sẽ là nội dung xuyên suốt cả môn học): **Virtual data
integration** (tích hợp ảo — để dữ liệu NGUYÊN TẠI nguồn, chỉ truy cập khi có truy vấn) và **Data
warehousing** (kho dữ liệu — tích hợp bằng cách SAO CHÉP dữ liệu vào 1 kho vật lý DUY NHẤT). Ghi chú
cuối: cả 2 kiến trúc đều phải đối mặt với "semantic heterogeneity" (dị biệt ngữ nghĩa — không kiến
trúc nào "miễn nhiễm" khỏi vấn đề này), và trong thực tế có RẤT NHIỀU kiến trúc trung gian giữa 2 thái
cực Virtual/Warehouse này (không phải chỉ chọn 1 trong 2).

### Slide 30 — Virtual Data Integration Architecture
Sơ đồ kiến trúc kỹ thuật chi tiết, từ trên xuống: **Mediated Schema or Warehouse** (hộp trên cùng, có
5 ô vuông nhỏ đại diện các quan hệ/relation) nhận **Query reformulation / Query over materialized
data** (viết lại truy vấn, hoặc chạy trực tiếp trên dữ liệu đã vật lý hoá); bên dưới là các mũi tên từ
**Wrapper/Extractor** đi lên, nhãn **"Source descriptions/Transforms"** (mô tả nguồn/phép biến đổi);
dưới cùng là 4 nguồn dữ liệu cụ thể: `RDBMS1`, `HTML1` (biểu tượng ảnh), `RDBMS2`, `XML1` (biểu tượng
tài liệu), mỗi nguồn có 1 Wrapper/Extractor RIÊNG của nó. Đây là sơ đồ TỔNG QUÁT nhất của kiến trúc
Virtual Integration, dùng lại đúng các khối đã thấy ở slide 5 (Mediated Schema) và sẽ dùng lại ở slide
34 (Query Processing).

### Slide 31 — Example
Ví dụ cụ thể hoá sơ đồ slide 30: Mediated schema gồm 4 quan hệ `Movie(title, director, year, genre)`,
`Actors(title, actor)`, `Plays(movie, location, startTime)`, `Reviews(title, rating, description)` —
được ánh xạ từ 5 nguồn S1-S5 có cấu trúc KHÁC NHAU: S1 `Movies(name, actors, director, genre)`, S2
`Cinemas(place, movie, start)`, S3 `CinemasInNYC(cinema, title, startTime)`, S4
`CinemasInSF(location, movie, startingTime)`, S5 `Reviews(title, date, grade, review)`. Chú ý: tên
thuộc tính giữa mediated schema và các nguồn KHÔNG trùng khớp hoàn toàn (vd `name` ở S1 ứng với
`title` ở Movie) — đây chính là bài toán Schema Matching/Mapping sẽ học sâu ở slide `2_SchemaAlignment.pdf`.

### Slide 32 — Wrappers
Giải thích vai trò của **Wrapper**: gửi truy vấn xuống nguồn dữ liệu thật và CHUYỂN ĐỔI kết quả trả về
thành dạng tuple (hoặc mô hình dữ liệu nội bộ khác mà hệ thống tích hợp dùng). Ví dụ minh hoạ: 1 trang
kết quả tìm kiếm Amazon (ảnh chụp, hiển thị các album CD) được wrapper chuyển thành dữ liệu XML có cấu
trúc rõ ràng: `<cd><title>The best of…</title><artist>Abiteboul</artist>...<price>19.95</price></cd>`.
Đây chính là ví dụ cụ thể cho khối "Wrapper/Extractor" đã thấy ở sơ đồ slide 30.

### Slide 33 — Mediation Languages
Giới thiệu khái niệm "ngôn ngữ mediation" — dùng để MÔ TẢ mối quan hệ giữa mediated schema và các
nguồn dữ liệu. Sơ đồ ví dụ: Mediated Schema có 2 quan hệ `CD: ASIN, Title, Genre,...` và
`Artist: ASIN, name, ...`; đám mây "logic" ở giữa biểu thị các QUY TẮC ánh xạ; bên dưới là 5 lược đồ
nguồn thật: `CDs` (Album, ASIN, Price, DiscountPrice, Studio), `Books` (Title, ISBN, Price,
DiscountPrice, Edition), `Authors` (ISBN, FirstName, LastName), `Artists` (ASIN, ArtistName,
GroupName), `CDCategories`/`BookCategories`. Đây chính là phần dẫn nhập cho khái niệm **GAV/LAV/GLAV**
— các "ngôn ngữ mediation" cụ thể — sẽ được học chi tiết ở file `2_SchemaAlignment.pdf` và đã luyện
tập trong `Quiz-DI-DataIntegration-GAV-LAV-GLAV.md`/`BT-EditDistance-TFIDF-SchemaMapping.md`.

### Slide 34 — Query Processing
Sơ đồ pipeline xử lý truy vấn, từ trên xuống: **Query** → **Query reformulation** (viết lại truy vấn
theo mediated schema, xuất ra **Logical query plan**) → **Query optimizer** (tối ưu hoá, xuất ra
**Physical query plan**) → **Execution engine** (thực thi) → phân phối xuống 5 cặp **wrapper/source**
ở dưới cùng. Có 1 vòng lặp phản hồi **"Replanning request"** đi từ Execution engine ngược lại Query
optimizer — nghĩa là nếu trong lúc thực thi phát hiện kế hoạch không tối ưu/thất bại, hệ thống có thể
YÊU CẦU lập lại kế hoạch. Đây là bản đồ tổng quan cho module `4_QueryProcessing.pdf` sẽ học chi tiết
sau.

### Slide 35 — Data Warehouses – Offline Replication
Quy trình xây dựng Data Warehouse, 4 bước: (1) Xác định lược đồ vật lý, (2) Định nghĩa 1 CSDL với
lược đồ đó, (3) Định nghĩa các "mapping" thủ tục trong 1 công cụ ETL để nhập + làm sạch dữ liệu, (4)
SAO CHÉP ĐỊNH KỲ toàn bộ dữ liệu từ các nguồn về. Ghi chú quan trọng: "các nguồn và kho dữ liệu về cơ
bản ĐỘC LẬP với nhau tại thời điểm này" — tức sau khi đã copy xong, truy vấn trên warehouse hoàn toàn
không đụng tới nguồn gốc nữa (khác hẳn Virtual Integration ở slide 30, nơi mỗi truy vấn đều phải chạm
tới nguồn thật qua wrapper).

### Slide 36 — Pros and Cons of Data Warehouses
Bảng cân đối ưu/nhược điểm (dùng ký hiệu ✗ cho nhược điểm, ✓ cho ưu điểm):
- ✗ Cần đầu tư nhiều thời gian thiết kế layout CSDL vật lý LẪN logic — "tốn rất nhiều công sức!"
- ✗ Dữ liệu thường KHÔNG cập nhật tức thời (lazy hoặc offline refresh)
- ✓ Truy vấn trên warehouse KHÔNG làm gián đoạn các nguồn dữ liệu gốc
- ✓ Có thể chạy các tính toán NẶNG (data mining, data cleaning) thoải mái trên warehouse

Đây chính là nguồn gốc của đáp án câu 26 trong `Quiz-DI-DataIntegration-GAV-LAV-GLAV.md` (nhược điểm
"lazy refresh" của data warehouse) và câu 23 (Virtual Integration không làm gián đoạn nguồn — chính
xác hơn là NGƯỢC LẠI, warehouse mới là kiến trúc không làm gián đoạn nguồn tại thời điểm truy vấn).

---

## PHẦN 3 — Big Data Integration: Challenges

### Slide 37 — Outline (nhắc lại lần cuối)
Toàn bộ 5 mục đầu đã có dấu ✓, chỉ còn 1 mục cuối "Big Data Integration - Challenges" được đánh dấu
mũi tên ➢ — xác nhận đây là phần cuối cùng của bài giảng Introduction.

### Slide 38 — How big is big data?
Biểu đồ cột thật: "Annual Size of the Global Datasphere" (quy mô dữ liệu toàn cầu theo năm), trục tung
đơn vị Zettabytes, từ 2010 tới 2025, tăng từ gần 0 lên **175 ZB** vào năm 2025 (mũi tên nhấn mạnh con
số này). Nguồn: Data Age 2025, tài trợ bởi Seagate, dữ liệu từ IDC Global DataSphere, tháng 11/2018 —
đây là số liệu DỰ BÁO (không phải số liệu đo đạc thật tới 2025), cần hiểu đúng bản chất là ước tính.

### Slide 39 — Big Data Growth Statistics
Danh sách số liệu thống kê về tốc độ tăng trưởng dữ liệu (nguồn: brimco.io):
- Người dùng Internet tạo ra ~1.7 MB dữ liệu MỖI GIÂY
- 2022: ~97 zettabyte dữ liệu được tạo ra trên toàn thế giới
- 2023: ~2/3 dân số thế giới online; người dùng Internet tạo ra lượng dữ liệu gần GẤP 3 LẦN so với 2019
- 2025: con người sẽ tạo ra hơn 181 ZB dữ liệu (181 theo sau 21 số 0); sẽ có 55,7 tỷ thiết bị IoT kết
  nối, riêng các thiết bị IoT này tạo ra gần 80 ZB
- 1 người dùng Internet cần HƠN 180 TRIỆU NĂM để tải hết dữ liệu từ web
- Gần 80% công ty ước tính 50%-90% dữ liệu của họ là PHI CẤU TRÚC (văn bản, video, audio, log server,
  mạng xã hội)

**⚠️ Lưu ý:** đây là số liệu từ nguồn thứ 3 (blog ngành), không phải số liệu giảng viên tự đo — nên
dùng để CẢM NHẬN QUY MÔ bài toán, không trích dẫn như số liệu học thuật chính thức khi làm bài báo
cáo/luận văn.

### Slide 40 — Big Data Market
Biểu đồ cột: "Big data market size revenue forecast worldwide from 2011 to 2027" (đơn vị: tỷ USD) —
tăng từ ~10 tỷ USD (2011) lên hơn 115 tỷ USD (dự báo 2027), tăng trưởng gần như liên tục mỗi năm.
Nguồn: Statista, qua innowise.com. Ý nghĩa: không chỉ LƯỢNG dữ liệu tăng (slide 38-39), mà THỊ TRƯỜNG
công nghệ xử lý dữ liệu lớn cũng tăng trưởng tương ứng — minh chứng giá trị kinh tế thực của môn học.

### Slide 41 — (infographic "Big data analytics trends", không có dòng tiêu đề Beamer riêng)
Sơ đồ chuỗi các xu hướng phân tích dữ liệu lớn hiện nay (nguồn: innowise.com), đi theo 2 nhánh gặp
nhau ở giữa: AI-driven data insights in real time → ESG reporting and data consolidation → **Data
integration and centralization** → Quantum computing and big data → Democratizing data access |
Data governance and security ← Ethical considerations and societal impact ← Industry-specific
solutions ← IoT and big data Integration ← Focus on data visualization. Đáng chú ý: "Data integration
and centralization" nằm NGAY GIỮA chuỗi xu hướng này — khẳng định lại Data Integration vẫn là mắt xích
trung tâm ngay cả trong các xu hướng công nghệ mới nhất (AI, Quantum computing, IoT).

### Slide 42 — Industry-specific Solutions
Sơ đồ hình tròn: "Big data" ở giữa, toả ra 7 ngành ứng dụng cụ thể: Telecom (viễn thông), Retailing
(bán lẻ), Media and entertainment (truyền thông & giải trí), Energy (năng lượng), Finance (tài
chính), Health (y tế), Government (chính phủ) — mở rộng cụ thể hơn cho danh sách tổng quát "Business/
Science/Government/Web" đã thấy ở slide 6 đầu bài giảng.

### Slide 43 — Big data 5'V
Giới thiệu khung lý thuyết kinh điển nhất của Big Data — mô hình **5V**, trình bày dạng 5 mảnh ghép:
**Velocity** (tốc độ dữ liệu được tạo ra và phân tích), **Variety** (nhiều loại/dạng dữ liệu khác
nhau, gồm cả dữ liệu phi cấu trúc số lượng lớn), **Value** (tiềm năng của big data cho phát triển kinh
tế-xã hội), **Veracity** (mức độ chất lượng, chính xác, và độ KHÔNG CHẮC CHẮN của dữ liệu và nguồn dữ
liệu), **Volume** (khối lượng khổng lồ dữ liệu được tạo ra qua số hoá quy mô lớn). Định nghĩa Big Data
trích Wikipedia ở cuối slide: "tập dữ liệu lớn/phức tạp tới mức phần mềm xử lý dữ liệu truyền thống
không đủ khả năng xử lý." **Liên hệ:** chính khái niệm Veracity + Variety này là lý do Data Integration
trở thành bài toán CÀNG KHÓ HƠN trong bối cảnh Big Data — không chỉ nhiều nguồn, mà còn nhiều DẠNG và
kém tin cậy hơn.

### Slide 44 — Big data technology stack
Sơ đồ 4 cột công nghệ, dữ liệu (`Data`) đi vào từ trái, giá trị (`Value`) đi ra bên phải, có dải
`Security Governance`/`Management Privacy` chạy dọc bên phải xuyên suốt cả 4 cột: **Big Data
Management** (Data Centers: IaaS/Cloud/Virtualization/Network/Compute/Storage/DBMS/NoSQL — "Scalable
Data Storage") → **Big Data Platforms** (Hadoop Ecosystem: Efficiency/Trust/Workload/Governance/
Tools/Platform/Programming/Parallel — "Data Ingestion And Processing") → **Big Data Analytics** (Data
Science: Machine Learning/Analysis/Integration/Query/Performance/Transform/Warehousing — "Transform
question to algorithm") → **Big Data Utilization** (Domain Expertise: Reporting & Dashboards/
Alerting & Recommendations/Business Intelligence/Text Analysis and Search — "Asking the right
question"). Đây là bức tranh tổng quan công nghệ GIÚP đặt Data Integration (nằm trong cột Analytics,
mục "Integration") vào đúng VỊ TRÍ của nó trong toàn bộ hệ sinh thái Big Data.

### Slide 45 — Challenges
Sơ đồ 6 thách thức thực tế khi tích hợp dữ liệu (nguồn: cloverdx.com), đánh số #1-#6 theo vòng tròn:
**#1** Dữ liệu của bạn không nằm ở nơi nó cần có mặt; **#2** Dữ liệu của bạn CÓ ở đó, nhưng nó ĐẾN
MUỘN; **#3** Dữ liệu của bạn không được định dạng đúng; **#4** Bạn có dữ liệu CHẤT LƯỢNG KÉM; **#5**
Có các bản ghi TRÙNG LẶP xuyên suốt pipeline của bạn; **#6** Không có sự HIỂU CHUNG RÕ RÀNG về dữ liệu
của bạn. Đây là góc nhìn CÔNG NGHIỆP/THỰC HÀNH, tổng kết hoá lại toàn bộ lý thuyết dị biệt dữ liệu đã
học (slide 16-27) thành 6 vấn đề cụ thể mà kỹ sư dữ liệu gặp hằng ngày.

### Slide 46 — Task for Big Data Integration (slide kết, quan trọng nhất để định hướng cả môn học)
Sơ đồ 3 khối mảnh ghép nối tiếp nhau theo đúng PIPELINE xử lý: **Schema Alignment** (căn chỉnh lược
đồ) → **Entity Resolution** (phân giải thực thể) → **Data Fusion** (hợp nhất dữ liệu). Mỗi khối có 3
kỹ thuật con bên dưới:
- Schema Alignment: Global-As-View (GAV), Local-As-View (LAV), Global-Local-As-View (GLAV)
- Entity Resolution: Dedupe (khử trùng lặp), Record Linkage (liên kết bản ghi), Dis-Ambiguation (khử
  mập mờ)
- Data Fusion: Voting (bỏ phiếu chọn giá trị), Source Quality (chất lượng nguồn), Copy Detection
  (phát hiện bản sao)

**Đây chính là slide bản đồ tổng thể của CẢ MÔN HỌC**, khớp CHÍNH XÁC với cấu trúc các file slide
khác trong cùng thư mục: `2_SchemaAlignment.pdf` (ứng với khối 1 — và đúng là nơi dạy sâu GAV/LAV/GLAV
đã luyện tập ở `Quiz-DI...md`/`BT-EditDistance...md`), `3_RecordLinkage_ER.pdf` (ứng với khối 2 —
Entity Resolution/Record Linkage, đúng cách đặt tên file), còn `Data Fusion` (khối 3) là phần lý
thuyết nối tiếp sẽ học sau Record Linkage. Nắm chắc đúng 3 khối + 9 kỹ thuật con này là nắm được bộ
khung toàn bộ nội dung môn học, trước khi đi sâu vào từng file slide riêng.
