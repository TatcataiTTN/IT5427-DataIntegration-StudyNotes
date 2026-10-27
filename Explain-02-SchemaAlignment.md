# Giải thích chi tiết — Schema Alignment and Query Rewriting (2_SchemaAlignment.pdf)

Nguồn: `slides/2_SchemaAlignment.pdf` — 43 trang PDF, mỗi trang in 2 slide gốc (2-up), tổng cộng
**86 slide**. Môn: Tích hợp và xử lý dữ liệu lớn (IT5427, GV Vũ Tuyết Trinh, SoICT-HUST). Đây chính
là slide nguồn của toàn bộ phần GAV/LAV/GLAV đã dùng để làm `Quiz-DI-DataIntegration-GAV-LAV-GLAV.md`
và bài tập Schema Matching S1/S2/S3→S trong `BT-EditDistance-TFIDF-SchemaMapping.md` — mỗi mục dưới
đây có ghi chú liên hệ ngược khi phù hợp.

Quy ước: "Trang N" = trang vật lý thứ N của file PDF (khớp `page-0N.png`). Mỗi trang chứa 2 slide,
đánh số slide liên tục 1-86 (số nhỏ in góc dưới mỗi khung slide).

---

## Trang 1 — Slide 1-2: Tiêu đề & "Schema Alignment / Schema matching"

**Slide 1 (tiêu đề):** "Schema Alignment and Query Rewriting" — Vũ Tuyết Trinh, SoICT-HUST. Đây là
bài giảng nối tiếp phần "Data Integration tổng quan" (Module 01), đi sâu vào bước kỹ thuật cốt lõi:
làm sao viết lại (rewrite) một truy vấn trên lược đồ trung gian thành truy vấn chạy được trên các
nguồn dữ liệu thật.

**Slide 2 — "Schema Alignment / Schema matching":** Nôm na: trước khi hệ thống tích hợp có thể trả
lời bất kỳ câu hỏi nào, nó phải biết "thuộc tính này ở nguồn A tương ứng với thuộc tính nào ở lược đồ
chung". Đó chính là **Schema Alignment/Matching** — bước "xác định sự tương ứng giữa các nguồn dữ
liệu khác nhau" (identifying correspondences). Sơ đồ minh hoạ: 1 truy vấn Q của người dùng gửi tới
**Mediated Schema** (lược đồ trung gian, hình ellipse ở giữa), sau đó lược đồ trung gian "dịch" Q
thành nhiều truy vấn con Q' gửi xuống từng **Source** (nguồn dữ liệu) ở dưới — đây chính là bức tranh
tổng quát mà cả slide 43 trang sau sẽ đi vào chi tiết kỹ thuật.

---

## Trang 2 — Slide 3-4: Kiến trúc xử lý truy vấn & Outline "(Traditional) Data Integration"

**Slide 3 (sơ đồ kiến trúc):** Mô tả luồng xử lý 1 truy vấn từ lúc người dùng gõ vào tới lúc có kết
quả — đây là bức tranh kỹ thuật đầy đủ hơn slide 2:
- **Query** → **Query reformulation** (viết lại truy vấn trên mediated schema thành truy vấn trên
  nguồn — đúng chủ đề chính của cả bài giảng này) → ra **Logical query plan**.
- Logical query plan → **Query optimizer** → **Physical query plan** (kế hoạch thực thi cụ thể, chọn
  thuật toán/thứ tự join...).
- Physical query plan → **Execution engine** → gửi xuống các **wrapper** (bộ chuyển đổi giao thức) →
  từng **source** (nguồn dữ liệu thật).
- Có mũi tên "**Re-planning request**" đi ngược từ execution engine về query optimizer — nghĩa là khi
  thực thi phát hiện vấn đề (vd nguồn chậm hơn dự kiến), hệ thống có thể yêu cầu lập lại kế hoạch.
- Khung nét đứt bao quanh "Query reformulation" + "Logical query plan" chính là phạm vi mà bài giảng
  này tập trung vào (phần Schema Alignment/Query Rewriting), còn Query optimizer/Execution engine
  thuộc phạm vi môn học khác (tối ưu truy vấn phân tán).

**Slide 4 (Outline):** liệt kê 4 phần của bài giảng — mục tiêu rõ ràng để theo dõi suốt 43 trang:
1. Introduction to semantic heterogeneity (dị biệt ngữ nghĩa)
2. Schema mapping languages (GAV/LAV/GLAV)
3. Data-level heterogeneity (dị biệt ở mức giá trị dữ liệu)
4. Query unfolding (kỹ thuật viết lại truy vấn)

---

## Trang 3 — Slide 5-6: "Schema Heterogeneity" (lý do & ví dụ xuyên suốt)

**Slide 5:** Luận điểm cốt lõi: **"Schema heterogeneity is a fact of life"** — dị biệt lược đồ là
điều TẤT YẾU, không phải lỗi thiết kế. Lý do: "bất cứ khi nào lược đồ được thiết kế bởi những người/
tổ chức khác nhau, chúng sẽ khác nhau, dù có mô hình hoá cùng 1 miền dữ liệu (domain)!" — ví dụ đơn
giản: 2 trường đại học khác nhau đều lưu thông tin sinh viên, nhưng một nơi đặt tên cột `MSSV`, nơi
khác đặt `StudentID`. Mục tiêu của **schema mapping** là "hoà giải" (reconcile) sự dị biệt này, chủ
yếu là giữa **mediated schema** và lược đồ của từng nguồn.

**Slide 6 — "Schema Heterogeneity by Example":** Đây là **ví dụ xuyên suốt** (running example) được
dùng lại ở HẦU HẾT các slide còn lại của bài — cần nhớ kỹ:
- **Mediated Schema** (lược đồ trung gian, mục tiêu cần đạt tới) gồm 4 quan hệ: `Movie(title,
  director, year, genre)`, `Actors(title, name)`, `Plays(movie, location, startTime)`,
  `Reviews(title, rating, description)`.
- **7 nguồn dữ liệu thật (S1-S7)**, mỗi nguồn có cấu trúc khác hẳn nhau:
  - S1: `Movie(title,director,year,genre)`, `Actor(AID,firstName,lastName,nationality,yearofBirth)`,
    `ActorPlays(AID,MID)`, `MovieDetails(MID,director,genre,year)` — nguồn "đầy đủ" nhất, tách actor
    thành bảng riêng theo chuẩn hoá.
  - S2: `Cinemas(place, movie, start)` — thông tin rạp chiếu, tên cột khác (place/movie/start thay
    vì location/title/startTime).
  - S3: `NYCCinemas(name, title, startTime)` — tương tự S2 nhưng chỉ phạm vi New York.
  - S4: `Reviews(title, date, grade, review)` — đánh giá phim, nhưng gọi điểm số là "grade" thay vì
    "rating".
  - S5/S6/S7: mỗi nguồn chỉ lưu ĐÚNG 1 khía cạnh — `MovieGenres(title,genre)`,
    `MovieDirectors(title,dir)`, `MovieYears(title,year)` — minh hoạ việc 1 thực thể "Movie" trong
    mediated schema bị PHÂN MẢNH ra nhiều nguồn khác nhau, mỗi nguồn chỉ biết 1 phần thông tin.
- Hình "logic" (ngôi sao ở giữa) tượng trưng cho LỚP ÁNH XẠ (mapping) cần xây dựng để nối 7 nguồn này
  với mediated schema — đây chính là nội dung toàn bộ phần còn lại của bài giảng.

---

## Trang 4 — Slide 7-8: "Table and Attribute Naming" & "Tabular Organization of Schema"

Hai slide này dùng LẠI đúng sơ đồ ở trang 3 (slide 6), chỉ khoanh tròn/làm nổi bật một khía cạnh dị
biệt cụ thể mỗi lần — đây là kỹ thuật trình bày "1 ví dụ, nhiều góc nhìn" để học viên thấy rõ TỪNG
LOẠI dị biệt riêng biệt thay vì gộp chung mơ hồ.

**Slide 7 — Table and Attribute Naming:** khoanh nổi bật các tên thuộc tính như `location`, `rating`,
`description` (mediated) đối chiếu với `place`, `grade`, `review` (nguồn) — minh hoạ dị biệt đơn giản
nhất: **cùng khái niệm, khác TÊN gọi**. Đây đúng loại dị biệt mà "name-based matching" cố giải quyết
(đã áp dụng thực hành ở bài tập S1/S2/S3→S, mục "Ánh xạ dựa trên tên").

**Slide 8 — Tabular Organization of Schema:** khoanh nổi bật khối S5/S6/S7 — minh hoạ dị biệt khác
hẳn: KHÔNG PHẢI khác tên, mà là khác **cách tổ chức bảng**. Mediated schema gộp genre+director+year
vào 1 bảng `Movie` duy nhất, nhưng nguồn lại TÁCH thành 3 bảng riêng `MovieGenres`, `MovieDirectors`,
`MovieYears`. Đây là dị biệt ở mức CẤU TRÚC (structural heterogeneity) — khó xử lý hơn dị biệt tên,
vì cần JOIN nhiều bảng nguồn mới dựng lại được 1 dòng của mediated schema (giống hệt tình huống
Bài 2 GAV trong bài tập: phải JOIN S2 và S3 theo khoá `ProductName` mới đủ 4 cột của S).

---

## Trang 5 — Slide 9-10: "Schema Coverage" & "Semantic Heterogeneity Summary"

**Slide 9 — Schema Coverage:** khoanh nổi bật cột `nationality, yearofBirth` của bảng `Actor` ở S1 và
cột `date` của `Reviews` ở S4 — ý muốn nói: các nguồn có **ĐỘ PHỦ (coverage) và mức độ chi tiết khác
nhau**. S1 biết cả quốc tịch, năm sinh diễn viên (thông tin mediated schema KHÔNG cần tới), còn S4 có
thêm cột `date` (ngày viết review) mà mediated schema `Reviews(title,rating,description)` không có.
Dị biệt này quan trọng vì: (a) 1 nguồn có thể biết NHIỀU HƠN mediated schema cần (dư thừa, phải lọc
bớt khi ánh xạ), (b) ngược lại nhiều nguồn khác thiếu thông tin mediated schema cần (phải chấp nhận
NULL hoặc tìm nguồn khác bù).

**Slide 10 — Semantic Heterogeneity Summary:** tổng kết 4 LOẠI dị biệt vừa minh hoạ qua slide 6-9:
1. Naming of schema elements (tên gọi khác nhau — slide 7)
2. Organization of tables (tổ chức bảng khác nhau — slide 8)
3. Coverage and detail of schema (độ phủ/chi tiết khác nhau — slide 9)
4. Data-level representation (biểu diễn ở MỨC DỮ LIỆU khác nhau — ví dụ kinh điển "IBM" vs.
   "International Business Machines", cùng 1 thực thể nhưng 2 chuỗi ký tự khác hẳn — đây là tiền đề
   cho phần "Data-Level Heterogeneity" ở slide 59 sau này).

Lý do gốc rễ (slide nêu rõ): "Schemas probably designed for different applications/contexts" — mỗi
lược đồ sinh ra để phục vụ MỤC ĐÍCH riêng của tổ chức tạo ra nó, không ai "cố tình" làm khác nhau.

---

## Trang 6 — Slide 11-12: "More in Source Descriptions" & "Outline" (lặp lại, đánh dấu tiến độ)

**Slide 11 — More in Source Descriptions:** ngoài dị biệt SCHEMA, một "mô tả nguồn" (source
description) đầy đủ còn cần biết thêm 3 nhóm thông tin thực hành quan trọng (không phải lý thuyết
hình thức, mà là ràng buộc vận hành thật):
- **Access pattern** (mẫu truy cập): nguồn có cần input bắt buộc không (vd phải điền form, gọi web
  service mới lấy được dữ liệu)? Nguồn có xử lý được truy vấn phức tạp không, hay chỉ trả về toàn bộ
  dữ liệu thô?
- **Source completeness** (độ đầy đủ): nguồn có chứa TOÀN BỘ dữ liệu của miền đó không, hay chỉ một
  phần? (liên hệ trực tiếp khái niệm "Certain Answers" sẽ học ở slide 19, 31, 43 — việc 1 nguồn
  không đầy đủ ảnh hưởng tới việc câu trả lời có "chắc chắn đúng" hay không).
- **Reliability, load restrictions, mirror sites…**: độ tin cậy, giới hạn tải, có bản sao (mirror)
  dự phòng không — các yếu tố vận hành thực tế.

**Slide 12 — Outline (lặp lại):** đánh dấu ✓ mục "Introduction to semantic heterogeneity" đã xong,
chuẩn bị bước sang phần 2 "Schema mapping languages" — đúng kỹ thuật trình bày "outline mốc" giúp
người học theo dõi đã đi tới đâu trong bài giảng dài 86 slide.

---

## Trang 7 — Slide 13-14: "Principles of Schema Mappings" & "Semantics of Schema Mappings"

**Slide 13 — Principles of Schema Mappings:** định nghĩa hình thức: schema mapping mô tả MỐI QUAN HỆ
giữa 2 phía — hoặc đơn giản là **Source ↔ Target** (1 nguồn, 1 đích), hoặc tổng quát hơn là
**Mediated Schema ↔ {S1, S2, …, Sn}** (nhiều nguồn cùng lúc, đúng bài toán tích hợp dữ liệu thật).

**Slide 14 — Semantics of Schema Mappings (nôm na → hình thức):** Nôm na: mapping không chỉ là "tên
cột A tương ứng tên cột B" — nó phải trả lời được câu hỏi sâu hơn: "với 1 bộ dữ liệu THẬT đang có ở
các nguồn, thì bộ dữ liệu NÀO ở mediated schema được coi là 'hợp lệ' (consistent) với nó?" Hình thức
hoá: gọi $I(G)$ là tập TẤT CẢ các thể hiện (instance) CÓ THỂ có của mediated schema $G$, và $I(S_i)$
tương tự cho từng nguồn. Mapping $M$ chính là một **quan hệ (relation)** giữa các tập này:
$$M_R \subseteq I(G) \times I(S_1) \times \ldots \times I(S_n)$$
Tức $M_R$ là tập các bộ $(g, s_1, \ldots, s_n)$ "khớp nhau" — $g$ là 1 thể hiện hợp lệ của mediated
schema khi các nguồn đang có dữ liệu $s_1,\ldots,s_n$. Đây là nền tảng toán học cho MỌI định nghĩa
GAV/LAV/GLAV sẽ học sau — các slide 24, 32, 39, 40, 53 đều là các cách THU HẸP định nghĩa $M_R$ tổng
quát này theo từng kiểu ánh xạ cụ thể.

---

## Trang 8 — Slide 15-16: "Relations, explained" & "Possible Instances... Simple Example"

**Slide 15 — Relations, explained:** ôn lại khái niệm CSDL cơ bản để chuẩn bị hiểu slide 14: 1 quan
hệ (relation) chỉ đơn giản là 1 TẬP CON của tích Đề-các (Cartesian product) các miền giá trị của từng
cột. Ví dụ bảng bên trái {(1,1),(2,4),(3,9),(4,16),(5,25)} là tập con của Int × Int, biểu diễn quan
hệ "Squared" (x, x²). Tương tự, $M_R$ ở slide 14 cũng chỉ là 1 tập con của $I(G)\times I(S_1)\times
\ldots$ — không có gì huyền bí, chỉ là áp dụng đúng định nghĩa "quan hệ" lên một không gian lớn hơn
(không gian các THỂ HIỆN, không phải không gian các GIÁ TRỊ).

**Slide 16 — Possible Instances of Mediated Schema: Simple Example:** ví dụ đầu tiên, trường hợp ĐƠN
GIẢN NHẤT: Source 1 có `(Director,Title,Year)` = {(Allen,Manhattan,1979), (Coppola,GodFather,1972)}.
Mediated schema chỉ cần `(Title,Year)` — ÍT thông tin hơn nguồn → chỉ cần CHIẾU (projection) bỏ cột
Director đi. Vì phép chiếu luôn xác định (deterministic), nên chỉ có ĐÚNG 1 thể hiện khả dĩ của
mediated schema: {(Manhattan,1979),(GodFather,1972)} — không có gì mơ hồ.

---

## Trang 9 — Slide 17-18: "...Second Example" & "Answering Queries over Possible Instances"

**Slide 17 — Possible Instances: Second Example:** đảo NGƯỢC tình huống slide 16 — giờ Source 1 chỉ
có `(Title,Year)` = {(Manhattan,1979),(GodFather,1972)}, nhưng mediated schema cần NHIỀU hơn:
`(Director,Title,Year)`. Vì nguồn KHÔNG CÓ thông tin Director, nên có thể có **NHIỀU thể hiện khả dĩ**
khác nhau của mediated schema, miễn là chiếu xuống (Title,Year) khớp với nguồn — ví dụ Possible
instance 1: {(Allen,Manhattan,1979),(Coppola,GodFather,1972)}, Possible instance 2: {(Halevy,
Manhattan,1979),(Stonebraker,GodFather,1972)} — 2 bộ giá trị Director hoàn toàn khác nhau nhưng ĐỀU
hợp lệ vì nguồn không cho biết Director thật là ai.

**Slide 18 — Answering Queries over Possible Instances:** vì sao sự mơ hồ ở slide 17 lại QUAN TRỌNG —
nó ảnh hưởng trực tiếp tới việc TRẢ LỜI TRUY VẤN đúng đắn:
- Query Q1 "trả về tất cả năm chiếu phim": dù Director là Allen/Coppola hay Halevy/Stonebraker thì
  NĂM vẫn luôn là (1979, 1972) ở CẢ 2 thể hiện khả dĩ → đây là **certain answer** (câu trả lời CHẮC
  CHẮN đúng, bất kể thể hiện nào trong số các thể hiện khả dĩ).
- Query Q2 "trả về tất cả đạo diễn": KHÔNG có certain answer nào, vì không có đạo diễn nào xuất hiện
  ở TẤT CẢ các thể hiện khả dĩo (Allen chỉ đúng ở instance 1, Halevy chỉ đúng ở instance 2) — trả lời
  "Allen" có thể SAI nếu thực tế đúng là instance 2.

Đây chính là tiền đề trực tiếp cho khái niệm **Certain Answers** sẽ được định nghĩa hình thức ở
trang sau (slide 19) và nhắc lại xuyên suốt tới tận slide 43, 44, 45 cuối phần LAV.

---

## Trang 10 — Slide 19-20: "Certain Answers Makes this Formal" & "Desiderata..."

**Slide 19 — Certain Answers Makes this Formal:** định nghĩa CHÍNH XÁC khái niệm vừa minh hoạ cảm
tính ở slide 18: "Một câu trả lời là CERTAIN (chắc chắn) nếu nó ĐÚNG trong MỌI thể hiện của mediated
schema mà CONSISTENT (khớp) với: (1) các thể hiện hiện tại của nguồn, và (2) mapping $M$." Viết bằng
ký hiệu toán:
$$t \in Q(s_1,\ldots,s_n) \;\text{iff}\; t \in Q(g) \;\text{for } \forall g \text{ s.t. } (g,s_1,\ldots,s_n) \in M_R$$
Đọc: bộ $t$ là câu trả lời của truy vấn $Q$ chạy TRÊN NGUỒN khi và chỉ khi $t$ cũng là câu trả lời của
$Q$ chạy trên **MỌI** thể hiện $g$ hợp lệ của mediated schema (ứng với $M_R$) — "MỌI" ($\forall$)
chính là từ khoá quan trọng nhất, phân biệt certain answer với câu trả lời "có thể đúng".

**Slide 20 — Desiderata from Source Description Languages:** sau khi đã có nền tảng lý thuyết, slide
này đặt câu hỏi THỰC TIỄN: 1 ngôn ngữ mô tả nguồn (để viết mapping $M$) cần có những tính chất gì?
3 tiêu chí mong muốn — chính là 3 tiêu chí dùng để SO SÁNH GAV/LAV/GLAV xuyên suốt phần còn lại:
1. **Flexibility** (linh hoạt): biểu diễn được quan hệ giữa các lược đồ THẬT (phức tạp, không lý
   tưởng).
2. **Efficient reformulation** (viết lại hiệu quả): độ phức tạp tính toán của việc viết lại truy vấn
   và tìm câu trả lời phải chấp nhận được.
3. **Easy update** (dễ cập nhật): thêm/xoá nguồn mới phải DỄ, không được đòi hỏi sửa lại toàn bộ hệ
   thống mapping đã có.

---

## Trang 11 — Slide 21-22: "Languages for Schema Mapping" (giới thiệu GAV/LAV/GLAV) & ví dụ (lặp)

**Slide 21 — Languages for Schema Mapping:** đây là **slide bản lề** của toàn bài — giới thiệu 3 kiểu
ngôn ngữ mapping sẽ học chi tiết suốt phần còn lại, qua đúng sơ đồ "Q → Mediated Schema → nguồn" ở
slide 2, nhưng giờ dán nhãn RÕ 3 kiểu mũi tên khác nhau:
- **GAV** (Global-as-View): mũi tên từ nguồn ĐI LÊN mediated schema.
- **LAV** (Local-as-View): mũi tên 2 chiều ở giữa.
- **GLAV**: tổng quát hơn, kết hợp cả 2 chiều.

Đây chính xác là nội dung sẽ xuất hiện lại trong `Quiz-DI` câu 5-14 (GAV) và câu 6, 8, 9, 12, 13
(LAV/GLAV) — các câu hỏi đó đều bám sát định nghĩa được trình bày chi tiết từ slide 23 trở đi.

**Slide 22:** lặp lại y hệt sơ đồ "Schema Heterogeneity by Example" ở slide 6 — dùng làm điểm khởi
đầu cho phần ví dụ GAV ngay sau đây.

---

## Trang 12 — Slide 23-24: "Global-as-View (GAV)" & "GAV: Formal Definition"

**Slide 23 — Global-as-View (GAV) [nôm na trước]:** Nôm na: GAV giống như việc bạn ĐỊNH NGHĨA lược
đồ trung gian giống như 1 cái **VIEW** (khung nhìn) được "lắp ráp" từ các bảng nguồn có sẵn — giống
hệt việc viết `CREATE VIEW` trong SQL, vế phải luôn là biểu thức trên các bảng NGUỒN. Ví dụ cụ thể:
mediated `Movie(title,director,year,genre)` được định nghĩa bằng:
$$Movie(title,director,year,genre) \supseteq S1.Movie(MID,title), S1.MovieDetail(MID,director,genre,year)$$
— tức "join 2 bảng con của S1 theo khoá MID" chính là ĐỊNH NGHĨA của Movie trong mediated schema.

**Slide 24 — GAV: Formal Definition:** viết tổng quát bằng ký hiệu toán, một trong 2 dạng:
$$G_i(\bar X) \supseteq Q(\bar S) \quad\text{hoặc}\quad G_i(\bar X) = Q(\bar S)$$
- Dạng $\supseteq$ gọi là **open-world assumption** (giả định thế giới mở): mediated schema có thể
  chứa NHIỀU hơn những gì query trên nguồn tính ra — vẫn chấp nhận còn dữ liệu khác chưa biết.
- Dạng $=$ gọi là **closed-world assumption** (giả định thế giới đóng): mediated schema CHÍNH XÁC
  bằng kết quả query trên nguồn, không hơn không kém.
- $G_i$: 1 quan hệ trong mediated schema; $Q(\bar S)$: 1 truy vấn trên các quan hệ nguồn.

Đây chính là định nghĩa đã dùng để trả lời `Quiz-DI` câu 10 ("mediated schema trong GAV là tập các
view TRÊN nguồn, đáp án a") và câu 5.

---

## Trang 13 — Slide 25-26: "GAV Example" & "GAV Example (cont.)"

**Slide 25:** áp dụng định nghĩa GAV cho quan hệ `Movie`, với **2 CÁCH** định nghĩa khác nhau (vì
thông tin Movie bị phân mảnh ở nhiều nguồn, như đã thấy ở slide 6):
- Cách 1: từ S1 (join Movie + MovieDetail theo MID).
- Cách 2: từ S5+S6+S7 (join MovieGenres, MovieDirectors, MovieYears theo title).

Có 2 view định nghĩa cho CÙNG 1 quan hệ Movie — dưới GAV open-world, mediated Movie SẼ chứa HỢP
(union) của kết quả cả 2 view này.

**Slide 26 — GAV Example (cont.):** tương tự cho quan hệ `Plays(movie,location,startTime)` — cũng có
2 cách định nghĩa từ S2 (`Cinemas`) hoặc S3 (`NYCCinemas`), phản ánh đúng việc S2/S3 đều có thông tin
rạp chiếu nhưng khác tên cột (place/movie/start vs. name/title/startTime).

---

## Trang 14 — Slide 27-28: "Reformulation in GAV" & "...= Query/View Unfolding"

**Slide 27 — Reformulation in GAV:** đặt vấn đề: khi người dùng gửi 1 truy vấn Q TRÊN mediated
schema, hệ thống phải "Return the best query possible on the data sources" — tức tìm ra truy vấn
TƯƠNG ĐƯƠNG chạy được trực tiếp trên nguồn. Bài toán này gọi là **reformulation** (viết lại).

**Slide 28 — Reformulation in GAV = Query/View Unfolding:** GAV giải bài toán reformulation bằng kỹ
thuật cụ thể gọi là **Query/View Unfolding**: lấy truy vấn Q trên mediated schema, ví dụ
```
Q(title, location, startTime) :-
    Movie(title, director, year, "comedy"),
    Plays(title, location, st), st ≥ 8pm
```
rồi đơn giản THAY từng vị từ (`Movie`, `Plays`) bằng ĐÚNG định nghĩa GAV của nó đã xây dựng ở slide
25-26 — đây là lý do kỹ thuật này được gọi là Reformulation = Unfolding, hoàn toàn khác bài toán LAV
sẽ học sau (Answering Queries Using Views, khó hơn nhiều).

---

## Trang 15 — Slide 29-30: "First Reformulation" & "Another Reformulation"

Hai slide này thực hiện CỤ THỂ phép unfold ở slide 28 — minh hoạ rằng vì `Movie` và `Plays` MỖI quan
hệ có 2 cách định nghĩa GAV khác nhau (slide 25-26), nên Q có thể unfold ra NHIỀU truy vấn tương
đương khác nhau trên nguồn:
- **First Reformulation (slide 29):** dùng định nghĩa Plays từ S2 → `Q'` chứa `S2.Cinemas(...)`.
- **Another Reformulation (slide 30):** dùng định nghĩa Plays từ S3 → `Q'` chứa `S3.NYCCinemas(...)`.

Cả 2 đều ĐÚNG — hệ thống (Query optimizer, nhắc lại sơ đồ slide 3) sẽ chọn cách nào THỰC THI nhanh
hơn/rẻ hơn tuỳ tình huống (vd nguồn nào đang khả dụng, nguồn nào có index tốt hơn).

---

## Trang 16 — Slide 31-32: "Certain Answers (recall definition)" & "Semantics of GAV"

**Slide 31:** nhắc lại nguyên văn định nghĩa Certain Answers đã học ở slide 19 — kỹ thuật lặp lại có
chủ đích để chuẩn bị áp dụng NGAY vào ngữ cảnh GAV ở slide tiếp theo.

**Slide 32 — Semantics of GAV:** trả lời câu hỏi "với GAV, $(g,s_1,\ldots,s_n)\in M_R$ khi nào?" —
tức khi nào 1 thể hiện mediated schema $g$ được coi là HỢP LỆ với dữ liệu nguồn hiện có:
- Nếu dùng dạng $\supseteq$ (open-world): phần mở rộng (extension) của $G_i$ trong $g$ phải là
  **tập SIÊU (super-set)** của kết quả đánh giá $Q_i$ trên nguồn — $g$ được PHÉP chứa THÊM dữ liệu
  ngoài những gì tính được từ nguồn.
- Nếu dùng dạng $=$ (closed-world): phần mở rộng của $G_i$ trong $g$ phải **BẰNG CHÍNH XÁC** kết quả
  $Q_i$ trên nguồn — không hơn không kém.

Điểm quan trọng (sẽ thấy rõ ở slide 36 "GAV Summary"): dưới GAV, vì mediated schema được XÂY DỰNG
TRỰC TIẾP từ nguồn (không phải suy luận ngược), nên thường chỉ có **DUY NHẤT 1** thể hiện $g$ hợp lệ
— khác hẳn LAV (sẽ thấy ở slide 41-42 có NHIỀU thể hiện khả dĩ).

---

## Trang 17 — Slide 33-34: "Tricky Example for GAV" (2 phần)

**Slide 33:** đưa ra 1 tình huống "khó xử" cho GAV: thêm nguồn S8 lưu **CẶP** (actor, director) —
không tách riêng từng thực thể. Cố định nghĩa GAV:
$$Actors(NULL,actor) \supseteq S8(actor,director)$$
$$Movie(NULL,director,NULL,NULL) \supseteq S8(actor,director)$$
— phải điền NULL vào các cột mà S8 không cung cấp được (title, year, genre…) vì GAV BẮT BUỘC mỗi
quan hệ mediated phải được định nghĩa ĐẦY ĐỦ ngay từ 1 view cụ thể, dù view đó thiếu thông tin.

**Slide 34:** áp dụng với dữ liệu S8 thật: `({Keaton,Allen},{Pacino,Coppola})` → tuples sinh ra ở
mediated schema đều có nhiều cột NULL: `Actors: ({NULL,Keaton},{NULL,Pacino})`,
`Movie: ({NULL,Allen,NULL,NULL},{NULL,Coppola,NULL,NULL})` — dữ liệu bị "vỡ vụn" thành nhiều dòng
NULL rời rạc, MẤT liên kết giữa actor và director ban đầu (ai đóng phim của ai).

---

## Trang 18 — Slide 35-36: "Tricky Example (2)" & "GAV Summary"

**Slide 35 — Tricky Example (2):** hậu quả trực tiếp của vấn đề slide 34: hệ thống **KHÔNG THỂ** trả
lời truy vấn đơn giản "`Actors(title,actor), Movie(title,director,genre,year)` → trả về (actor,
director)" — vì liên kết giữa Keaton và Allen đã bị NULL hoá, không còn `title` chung để JOIN 2 bảng
lại. Slide chỉ ra giải pháp: **"LAV (Local as View) will solve this problem"** — đây là lý do chuyển
tiếp hợp lý sang phần LAV ngay sau.

**Slide 36 — GAV Summary:** tổng kết điểm MẠNH và YẾU của GAV:
- **Mạnh:** Reformulation (viết lại truy vấn) đơn giản về mặt khái niệm, chạy trong **thời gian đa
  thức (polynomial-time)** — đây là lý do GAV được ưa chuộng khi số lượng nguồn ít và ổn định.
- **Yếu:** GAV ép MỌI THỨ phải nhìn theo góc độ của mediated schema → "Cannot capture a variety of
  tabular organizations" (không nắm bắt được các cách tổ chức bảng đa dạng ở nguồn, như vừa thấy ở
  ví dụ S8). Đây chính là câu trả lời của `Quiz-DI` câu 7: "nhược điểm GAV: không linh hoạt với thay
  đổi nguồn vì mỗi thay đổi cần cập nhật lại lược đồ trung gian."

---

## Trang 19 — Slide 37-38: "Local-as-View (LAV)" (2 ví dụ)

**Slide 37 (nôm na trước):** LAV làm NGƯỢC HẲN GAV — thay vì định nghĩa mediated schema từ nguồn,
LAV định nghĩa từng **NGUỒN** là 1 view TRÊN mediated schema. Ví dụ với S5/S6/S7 (mỗi nguồn chỉ biết
1 khía cạnh của Movie):
$$S5.MovieGenres(title,genre) \subseteq Movie(title,dir,year,genre)$$
$$S6.MovieDirectors(title,dir) \subseteq Movie(title,dir,year,genre)$$
— đọc là: "mọi dòng của MovieGenres PHẢI là 1 phần (chiếu xuống 2 cột) của 1 dòng nào đó trong
mediated Movie" — dùng dấu $\subseteq$ (tập con), NGƯỢC CHIỀU với $\supseteq$ của GAV.

**Slide 38 (ví dụ S8, khó với GAV ở slide 33-35):** LAV định nghĩa S8 như sau:
$$S8.ActorDirectors(actor,dir) \subseteq Movie(title,dir,year,genre), Actor(title,actor), year \geq 1980$$
Khác biệt mấu chốt so với cách GAV xử lý S8 (slide 33): LAV **KHÔNG BẮT BUỘC** phải điền NULL vào các
cột không biết — nó chỉ nói "S8 là 1 PHẦN của (hệ quả join Movie và Actor theo title, lọc year≥1980)"
mà không cần биết chính xác title/year/genre là gì — đây chính là lý do LAV "giải quyết được" vấn đề
ở slide 35.

---

## Trang 20 — Slide 39-40: "LAV: Formal Definition" & "Semantics of LAV"

**Slide 39 — LAV: Formal Definition:** định nghĩa hình thức, đối xứng CHÍNH XÁC với GAV (slide 24)
nhưng ĐẢO CHIỀU vai trò 2 bên:
$$S_i(\bar X) \subseteq Q_i(G) \quad\text{hoặc}\quad S_i(\bar X) = Q_i(G)$$
- $S_i(\bar X)$: 1 quan hệ NGUỒN (ngược với $G_i$ của GAV là quan hệ MEDIATED).
- $Q_i(G)$: 1 truy vấn TRÊN mediated schema (ngược với $Q(\bar S)$ của GAV là truy vấn trên nguồn).
- Vẫn có 2 biến thể open-world ($\subseteq$) / closed-world ($=$), giống hệt GAV.

**Slide 40 — Semantics of LAV:** tương tự slide 32 (Semantics of GAV) nhưng đảo chiều: bộ
(g, s1,…,sn) thuộc mapping MR khi kết quả của truy vấn Qi đánh giá TRÊN g là tập SIÊU (hoặc BẰNG) của
si — nói cách khác, lược đồ g phải "giải thích được" đầy đủ dữ liệu nguồn si khi chạy Qi lên nó.

---

## Trang 21 — Slide 41-42: "Possible Databases" (2 phần)

**Slide 41:** điểm KHÁC BIỆT CĂN BẢN giữa LAV và GAV (nhắc lại ở slide 36, giờ chứng minh bằng ví
dụ): "Unlike GAV, LAV definitions imply a SET of possible databases for the mediated schema." Với
ràng buộc `S8.ActorDirectors(actor,dir) ⊆ Movie(...),Actor(...)` và dữ liệu `S8:
{(Keaton,Allen)}`, có ÍT NHẤT 2 thể hiện mediated schema khác nhau đều thoả mãn ràng buộc này — ví dụ
dùng title="manhattan" hoặc title="foobar" (title BẤT KỲ cũng được, miễn khớp join) — vì LAV không
nói title CỤ THỂ là gì.

**Slide 42:** mở rộng thêm: vì nguồn CÓ THỂ không đầy đủ (incomplete, nhắc lại slide 11 "source
completeness"), mediated schema còn có thể chứa THÊM các dòng khác mà S8 hoàn toàn không biết tới
(ví dụ phim "leatherheads" đạo diễn Clooney 2008) — miễn là không mâu thuẫn với ràng buộc LAV đã có.

---

## Trang 22 — Slide 43-44: "Certain Answers: by now you know this slide by heart" & "...Example 1"

**Slide 43:** đúng như tên slide đùa ("giờ chắc bạn đã thuộc lòng") — lặp lại NGUYÊN VĂN định nghĩa
Certain Answers lần thứ 3 (sau slide 19, 31) — kỹ thuật sư phạm lặp lại có chủ đích vì đây là khái
niệm TRUNG TÂM xuyên suốt cả GAV lẫn LAV.

**Slide 44 — Certain Answers Example 1:** áp dụng vào ví dụ S8 (slide 38, 41): với ràng buộc LAV
`S8.ActorDirectors ⊆ Movie(title,dir,year,genre),Actor(title,actor)` và dữ liệu `S8:
{(Keaton,Allen)}`, chạy truy vấn `Q(actor,dir):- Movie(title,dir,year,genre),Actor(title,actor)` →
DÙ có nhiều thể hiện mediated khả dĩ khác nhau (slide 41), cặp (Keaton,Allen) luôn xuất hiện ở MỌI
thể hiện đó → đây là **certain answer DUY NHẤT**: (Keaton, Allen).

---

## Trang 23 — Slide 45-46: "Certain Answers Example 2" & "Reformulation in LAV"

**Slide 45 — Certain Answers Example 2:** một ví dụ khác đối chiếu rõ ảnh hưởng của **giả định thế
giới đóng/mở** (closed/open-world, đã nhắc ở slide 24, 39) lên kết quả certain answer: với 2 view
$V_8(dir)=\{Allen\}$, $V_9(actor)=\{Keaton\}$ và truy vấn join Director-Actor theo ID — dưới
**closed-world**: chỉ có DUY NHẤT 1 CSDL khả dĩ → certain answer = (Allen, Keaton). Nhưng dưới
**open-world**: vì không biết CHẮC 2 giá trị này có cùng 1 ID hay không (có thể còn Director/Actor
khác chưa biết) → **KHÔNG CÓ certain answer nào cả**. Đây là minh hoạ rất rõ: lựa chọn giả định
(open/closed-world) ảnh hưởng TRỰC TIẾP tới việc 1 câu trả lời có được công nhận là "chắc chắn" hay
không — không phải chi tiết kỹ thuật phụ, mà là quyết định thiết kế quan trọng.

**Slide 46 — Reformulation in LAV:** đặt lại bài toán reformulation (như slide 27 cho GAV) nhưng cho
LAV: ta CÓ dữ liệu của các nguồn $S_1,\ldots,S_n$ (biểu diễn dưới dạng view LAV), nhưng KHÔNG CÓ dữ
liệu trực tiếp của mediated schema — chỉ có 1 truy vấn Q viết TRÊN mediated schema. Slide chỉ thẳng:
"This is exactly the problem of: **Answering Queries Using Views**" — khác hẳn GAV (chỉ là Unfolding
đơn giản), LAV đòi hỏi 1 bài toán khó hơn nhiều về mặt lý thuyết tính toán.

---

## Trang 24 — Slide 47-48: "Example" (MovieYear/MovieRev) & "LaV - rewriting"

**Slide 47 — Example mới:** đổi sang 1 ví dụ KHÁC đơn giản hơn để minh hoạ thuật toán Answering
Queries Using Views: mediated schema gồm `MovieYear(title,year)` và `MovieRev(title,director,
review)`; 3 nguồn LAV:
- `S1: V1(title,year,director) :- MovieYear(title,year) & MovieRev(title,director,review)`
- `S2: V2(title,year,director) :- MovieYear(title,year) & MovieRev(title,director,review)` (giống
  hệt V1 — 2 nguồn trùng định nghĩa, minh hoạ việc dữ liệu có thể LẶP ở nhiều nguồn khác nhau).
- `S3: V3(title,review) :- MovieRev(title,director,review)`.

**Slide 48 — LaV - rewriting:** đặt truy vấn cụ thể "tìm review của phim ra năm 1997":
```
q(title,review):- MovieYear(title,1997) & MovieReview(title,director,review)
```
Dấu "???" thể hiện ĐÚNG khó khăn cốt lõi của LAV reformulation: KHÔNG THỂ unfold đơn giản như GAV
(vì không có định nghĩa view CHO mediated schema, mà ngược lại) — slide gợi ý 2 cách viết lại khả dĩ
bằng V1/V2 kết hợp V3 (`q1` dùng V1+V3, `q2` dùng V2+V3) nhưng chưa giải thích NGUYÊN TẮC — để dành
cho slide tiếp theo.

---

## Trang 25 — Slide 49-50: "Principle of rewriting in LaV" & "Local-as-View Summary"

**Slide 49 — Principle of rewriting in LaV (thuật toán 3 bước):**
1. **Inverse rules** (quy tắc đảo ngược): từ mỗi định nghĩa view LAV (vd `V1(title,year,director) :-
   MovieYear(title,year) & MovieRev(title,director,review)`), ĐẢO NGƯỢC thành quy tắc suy luận NGƯỢC
   LẠI: `MovieYear(title,year) :- V1(title,year,_)` và `MovieRev(title,director,?) :- V1(title,_,
   director)` — dấu `?`/`_` đánh dấu các cột không chắc chắn/không quan tâm. Đây là ý tưởng cốt lõi:
   coi view như một "nguồn suy luận ngược" để TÁI TẠO lại các vị từ mediated schema.
2. **Unfolding:** thay các vị từ mediated schema trong truy vấn gốc (q1/q2 ở slide 48) bằng quy tắc
   nghịch đảo vừa tạo — ra `q1(title,review):- V1(title,1997,_) & V1(title,_,director)` v.v.
3. **Cleaning:** kiểm tra lại tính HỢP LỆ của các truy vấn đã unfold (vì bước đảo ngược có thể sinh
   ra truy vấn vô nghĩa/dư thừa, cần loại bỏ).

**Slide 50 — Local-as-View Summary:** tổng kết:
- Reformulation LAV = bài toán Answering Queries Using Views (khác GAV).
- Thuật toán thực tế CHẠY TỐT (không phải bottleneck chính của hệ thống).
- Dưới một số điều kiện, ĐẢM BẢO tìm được TOÀN BỘ certain answers (trong thực tế hầu như luôn tìm
  được).
- **Điểm khác biệt cốt lõi với GAV** (nhắc lại slide 41): "LAV expresses incomplete information" —
  LAV biểu diễn được tình huống dữ liệu KHÔNG ĐẦY ĐỦ (nhiều thể hiện khả dĩ), còn GAV thì không (chỉ
  có 1 thể hiện duy nhất nhất quán với nguồn).

---

## Trang 26 — Slide 51-52: "LAV Limitation" & "Comparaison: GaV vs. LaV" (sơ đồ)

**Slide 51 — LAV Limitation:** nhược điểm đối lập với GAV: nếu nguồn S1 có 1 **khoá nội bộ** (internal
key, ví dụ MID trong `Movie(MID,title)` join với `MovieDetails(MID,...)`) thì **LAV KHÔNG THỂ sử dụng
khoá đó** — vì LAV chỉ mô tả S1 như 1 view TRÊN mediated schema (nhìn từ ngoài vào), không "biết" gì
về cấu trúc join NỘI BỘ bên trong S1. Đây là nhược điểm đối xứng với nhược điểm GAV (slide 36): GAV
mạnh về cấu trúc nội bộ nguồn nhưng yếu khi thêm/sửa nguồn; LAV mạnh khi thêm/sửa nguồn nhưng yếu về
khai thác cấu trúc nội bộ.

**Slide 52 — Comparaison: GaV vs. LaV (sơ đồ tổng kết trực quan):** 1 sơ đồ duy nhất tóm tắt TOÀN BỘ
sự khác biệt GAV/LAV học từ đầu bài tới giờ — mediated schema ở trên với quan hệ `R(#K,A,B,C)` và
truy vấn `Q: R1⋈R2⋈R3`; nguồn ở dưới với 2 lược đồ nguồn `R1(#K,A),R2(#A,B)` và `R3(#B,C)`. Mũi tên
**LaV đi XUỐNG** (từ mediated → nguồn, vì LAV định nghĩa NGUỒN dựa trên mediated), mũi tên **GaV đi
LÊN** (từ nguồn → mediated, vì GAV định nghĩa MEDIATED dựa trên nguồn) — đúng bản chất "chiều ánh xạ
ngược nhau" đã nhấn mạnh nhiều lần. Sơ đồ này lặp lại nguyên văn ở slide 67 (trang 34) để đóng phần
GLAV lại bằng đúng hình ảnh đã mở đầu.

---

## Trang 27 — Slide 53-54: "GLAV – a better solution?" & "GLAV Example"

**Slide 53 — GLAV (nôm na trước):** sau khi thấy GAV và LAV đều có nhược điểm RIÊNG (slide 36, 51),
GLAV ra đời để **TỔNG QUÁT HOÁ cả 2**, không ép phải chọn 1 chiều duy nhất. Định nghĩa hình thức:
$$Q^S(\bar X) \subseteq Q^G(\bar X) \quad\text{hoặc}\quad Q^S(\bar X) = Q^G(\bar X)$$
- $Q^G$: truy vấn trên mediated schema (như LAV).
- $Q^S$: truy vấn trên NGUỒN (như GAV) — **khác biệt mấu chốt**: cả 2 vế đều là TRUY VẤN (không chỉ
  1 quan hệ đơn lẻ như GAV/LAV), nên linh hoạt hơn nhiều.

**Slide 54 — GLAV Example:** ví dụ cụ thể:
$$S1.Movie(MID,title), S1.MovieDetail(MID,dir,genre,year) \subseteq Movie(title,dir,\text{"comedy"},year), year \geq 1970$$
— vế trái là truy vấn JOIN trên nguồn (kiểu GAV), vế phải là truy vấn CÓ ĐIỀU KIỆN LỌC trên mediated
schema (kiểu LAV, với điều kiện `genre="comedy"` và `year≥1970`) — chứng tỏ GLAV biểu diễn được CẢ
2 kiểu ràng buộc cùng lúc, điều GAV/LAV riêng lẻ không làm được.

---

## Trang 28 — Slide 55-56: "Reformulation in GLAV" & "...Tuple Generating Dependencies"

**Slide 55 — Reformulation in GLAV:** thuật toán viết lại truy vấn với GLAV, kết hợp cả 2 kỹ thuật
đã học: (1) tìm 1 cách viết lại Q' dùng các VIEW (giống bài toán Answering Queries Using Views của
LAV), (2) sau đó THAY (Create Q'' by replacing) và **Unfold** (giống kỹ thuật GAV) — đúng tinh thần
"tổng quát hoá, kế thừa cả 2" đã nêu ở slide 53.

**Slide 56 — An Alternative Notation for GLAV: Tuple Generating Dependencies (TGD):** giới thiệu 1
CÁCH VIẾT KHÁC tương đương về mặt toán học cho GLAV, dùng khái niệm **Tuple Generating Dependency**
(một dạng ràng buộc toàn vẹn kinh điển trong lý thuyết CSDL — slide ghi chú "Chapter 2.1.2" tức có
tham chiếu tới 1 giáo trình gốc, có thể là Abiteboul/Hull/Vianu hoặc tương tự, slide không nêu tên
sách cụ thể nên không khẳng định thêm): dạng TGD $(\forall \bar X)s_1(\bar X_1),\ldots,s_m(\bar X_m)
\to (\exists \bar Y) t_1(\bar Y_1),\ldots,t_k(\bar Y_k)$ tương đương với biểu thức GLAV $Q^S(\bar X)
\subseteq Q^G(\bar X)$ — ký hiệu $\exists$ (tồn tại) ở vế phải chính là điểm khác biệt quan trọng,
cho phép biểu diễn thông tin KHÔNG ĐẦY ĐỦ tốt hơn (đúng như đã nêu trong `Quiz-DI` câu 8: GLAV dùng
TGD với biến tồn tại để biểu diễn incomplete information tốt hơn GAV/LAV riêng lẻ).

---

## Trang 29 — Slide 57-58: "GLAV -> TGD Example" & "Outline" (đánh dấu hoàn thành 3/4)

**Slide 57 — GLAV -> TGD Example:** áp dụng tương đương TGD vào đúng ví dụ GLAV ở slide 54 — viết
lại biểu thức $\subseteq$ thành dạng mũi tên TGD. Ghi chú quan trọng ở cuối slide: "Reformulation với
mô tả TGD có thể làm TRỰC TIẾP bằng **Inverse Rules Algorithm**" — liên hệ ngược lại đúng thuật toán
3 bước (inverse rules → unfolding → cleaning) đã học ở slide 49 cho LAV, nay áp dụng được luôn cho
GLAV nhờ cách viết TGD.

**Slide 58 — Outline (đánh dấu tiến độ):** 3/4 mục đã hoàn thành (✓ Introduction, ✓ Schema mapping
languages), chuẩn bị sang phần 3 "Data-level heterogeneity".

---

## Trang 30 — Slide 59-60: "Data-Level Heterogeneity" & "Mappings with Transformations"

**Slide 59 — Data-Level Heterogeneity (nôm na trước):** chuyển hẳn sang 1 TẦNG dị biệt KHÁC — không
còn là dị biệt CẤU TRÚC/TÊN (đã học suốt 58 slide trước), mà là dị biệt ở MỨC GIÁ TRỊ DỮ LIỆU THẬT,
dù 2 cột đã "khớp tên" với nhau. Nguyên văn: "Huge problem in practice: Data coming from different
sources RARELY joins perfectly." 4 ví dụ cụ thể, dễ hình dung:
- Celsius vs. Fahrenheit (đơn vị đo khác nhau)
- Numeric vs. letter grades (thang điểm số vs. thang chữ A/B/C)
- First Name, Last Name vs. FullName (tách cột vs. gộp cột)
- Giá có thuế hay không có thuế

Đây CHÍNH LÀ "tầng 4" đã nêu tóm tắt ở slide 10 ("Data-level representation") — giờ mới đi sâu vào
chi tiết kỹ thuật xử lý.

**Slide 60 — Mappings with Transformations:** giải pháp kỹ thuật: mapping không chỉ ánh xạ TÊN cột,
mà còn phải nhúng CÔNG THỨC BIẾN ĐỔI ngay trong định nghĩa mapping:
$$S(city, temp-32*5/9, month) \subseteq Weather(city,temp,humidity,month)$$
— chuyển Fahrenheit sang Celsius ngay trong mapping bằng công thức $(F-32)\times 5/9$. Ví dụ thứ 2
phức tạp hơn, kết hợp 2 nguồn (`CDPrices` và `LocalTaxes`) để tính GIÁ CÓ THUẾ:
$$CDStore(cd,price) \subseteq CDPrices(cd,state,price*(1+rate)), LocalTaxes(state,rate)$$
— minh hoạ việc transformation có thể cần JOIN thêm 1 bảng phụ (LocalTaxes) để lấy tham số biến đổi
(rate), không chỉ là công thức tĩnh đơn giản như ví dụ nhiệt độ.

---

## Trang 31 — Slide 61-62: "Outline" (4/4) & "Query Unfolding" (giới thiệu lại)

**Slide 61:** đánh dấu mục 3 "Data-level heterogeneity" đã xong, chuyển sang mục cuối "Query
unfolding" — lưu ý: unfolding ĐÃ được nhắc tới ở slide 28 (trong ngữ cảnh riêng GAV), nhưng giờ được
học lại một cách TỔNG QUÁT, độc lập với GAV/LAV/GLAV.

**Slide 62 — Query Unfolding (tổng quát):** định nghĩa lại rộng hơn: "Query composition" (xây truy
vấn từ các view theo kiểu TỪ DƯỚI LÊN - bottom up) là cơ chế quan trọng để viết truy vấn phức tạp;
"Query unfolding" chính là quá trình NGƯỢC LẠI — "unwinds" (tháo gỡ) composition đó ra thành truy vấn
phẳng. 3 lý do unfolding quan trọng: (1) so sánh các truy vấn được biểu diễn qua view khác nhau, (2)
tối ưu truy vấn (để trình tối ưu xem xét MỌI thứ tự join có thể), (3) — điểm thú vị — unfolding có
thể PHÁT HIỆN RA rằng 2 truy vấn, mỗi cái riêng lẻ đều thoả mãn được (satisfiable), nhưng khi GHÉP
LẠI (composition) thì lại KHÔNG THOẢ MÃN ĐƯỢC (unsatisfiable)! Slide để ngỏ như bài tập ("exercise:
find such an example") chứ không đưa ví dụ cụ thể.

---

## Trang 32 — Slide 63-64: "Query Unfolding Example" & "Query Unfolding Algorithm"

**Slide 63 — ví dụ cụ thể (bài toán chuyến bay):**
```
Q1(X,Y):- Flight(X,Z), Hub(Z), Flight(Z,Y)
Q2(X,Y):- Hub(Z), Flight(Z,X), Flight(X,Y)
Q3(X,Z):- Q1(X,Y), Q2(Y,Z)
```
Q3 được xây DỰA TRÊN Q1 và Q2 (composition). Unfolding Q3 = thay Q1(X,Y) và Q2(Y,Z) bằng chính định
nghĩa của chúng, đổi tên biến cho khỏi trùng (Q1 dùng biến U, Q2 dùng biến W) — kết quả là 1 truy vấn
DUY NHẤT, dài hơn, chỉ còn chứa `Flight` và `Hub` (không còn tham chiếu Q1/Q2 nữa).

**Slide 64 — Query Unfolding Algorithm (thuật toán tổng quát hoá từ ví dụ):**
1. Tìm 1 subgoal $p(X_1,\ldots,X_n)$ được định nghĩa bởi 1 rule $r$.
2. Hợp nhất (unify) $p(X_1,\ldots,X_n)$ với phần đầu (head) của $r$.
3. Thay $p(X_1,\ldots,X_n)$ bằng kết quả áp dụng unifier lên các subgoal của $r$ (dùng biến MỚI cho
   các biến tồn tại của $r$, tránh trùng tên — đúng như đã làm thủ công ở ví dụ slide 63 với biến U,
   W).
4. Lặp lại tới khi không còn hợp nhất được nữa.
5. Nếu $p$ được định nghĩa bởi HỢP (union) của nhiều rule $r_1,\ldots,r_n$ → tạo ra $n$ bản sao,
   mỗi bản ứng với 1 rule.

---

## Trang 33 — Slide 65-66: "Query Unfolding Summary" & "Summary" (tổng kết toàn bài)

**Slide 65 — Query Unfolding Summary:** 2 điểm cảnh báo quan trọng, tránh ngộ nhận:
- Unfolding **KHÔNG CHẮC** tạo ra truy vấn hiệu quả hơn! Nó chỉ cho phép trình tối ưu (query
  optimizer, nhắc lại sơ đồ slide 3) khám phá NHIỀU chiến lược thực thi hơn — bản thân unfolding
  không tự động "tốt hơn".
- Unfolding chính là phép toán NGƯỢC của "rewriting queries using views" (bài toán LAV đã học) —
  2 kỹ thuật đi 2 chiều đối nghịch nhau, giống hệt tinh thần sơ đồ GaV↕LaV ở slide 52.
- Kích thước truy vấn kết quả có thể tăng theo cấp SỐ MŨ (exponential) — để ngỏ làm bài tập.

**Slide 66 — Summary (tổng kết CẢ 4 phần của bài giảng):**
- Mô tả nguồn (source description) gồm: schema mappings, completeness/access-pattern limitations
  (slide 11), data transformations (slide 60), và các khả năng xử lý truy vấn bổ sung khác.
- 2 chiến lược reformulation: GAV dùng unfolding (đơn giản, nhanh); LAV/GLAV dùng Answering Queries
  Using Views (khó hơn, nhưng linh hoạt/biểu diễn được incomplete information tốt hơn).
- Binding pattern và ràng buộc toàn vẹn (integrity constraints) có thể gây ra các trường hợp khó xử
  — "Recursive rewritings" (viết lại đệ quy) thường là hướng giải quyết, dù slide không đi sâu thêm.

---

## Trang 34 — Slide 67-68: "Comparaison: GaV vs. LaV" (lặp lại sơ đồ đóng bài) & "References"

**Slide 67:** lặp lại ĐÚNG sơ đồ tổng kết GAV vs. LAV đã thấy ở slide 52 (trang 26) — kỹ thuật "đóng
bài bằng đúng hình ảnh đã dùng giữa bài" để nhấn mạnh đây là kết luận cốt lõi nhất của toàn bộ phần
Schema Mapping Languages (GAV/LAV/GLAV).

**Slide 68 — References:** danh sách 7 tài liệu tham khảo gốc của phần GAV/LAV/GLAV (không phải phần
Big Data ở cuối bài, phần đó có References riêng ở slide 86) — đây CHÍNH LÀ các tài liệu đã dùng làm
căn cứ lý thuyết cho `Quiz-DI` và bài tập Schema Matching trong phiên làm việc này:
- Krishnamurthy, Litwin, Kent — SIGMOD 1991 (schematic discrepancies)
- Lakshmanan, Sadri, Subramanian — SchemaSQL, ACM TODS 2001
- **Jeffrey D. Ullman** — "Information Integration Using Logical Views", ICDT 1997 (nguồn gốc lý
  thuyết GAV/query unfolding)
- **M. Lenzerini** — "Data Integration: A Theoretical Perspective", PODS 2002 (bài tổng quan kinh
  điển nhất của cả lĩnh vực — cũng là nguồn trong danh sách Literature Review Papers của dự án)
- **Alon Y. Halevy** — "Theory of Answering Queries Using Views", SIGMOD Record 2000
- **Alon Y. Levy** — "Answering Queries Using Views: A Survey", VLDB Journal 2001 (nguồn gốc lý
  thuyết LAV)
- Miller, Haas, Hernandez — "Schema Mapping as Query Discovery", VLDB'00

---

## Trang 35 — Slide 69-70: "Addressing Big Data challenges" (tiêu đề phần mới) & "Big data 5'V"

**Slide 69:** tiêu đề phần CUỐI của bài giảng — chuyển hẳn bối cảnh từ Data Integration "truyền
thống" (vài nguồn, biết trước, mapping chính xác do chuyên gia soạn — đúng như toàn bộ 68 slide
trước) sang bối cảnh **Big Data**: hàng triệu nguồn, không thể soạn mapping thủ công.

**Slide 70 — Big data 5'V:** nhắc lại khung 5V kinh điển (đã xuất hiện trong nội dung Module 01 của
site IT5427): **Velocity** (tốc độ sinh/xử lý), **Variety** (đa dạng định dạng, kể cả dữ liệu phi cấu
trúc), **Value** (tiềm năng giá trị kinh tế-xã hội), **Veracity** (chất lượng/độ tin cậy/không chắc
chắn), **Volume** (khối lượng khổng lồ). Định nghĩa trích Wikipedia: "Big data is a term for data
sets that are so large or complex that traditional data processing application software is
inadequate to deal with them."

---

## Trang 36 — Slide 71-72: "Positioning different approaches" & "WebTables: Keyword Ranking [CHW+08]"

**Slide 71 — Positioning different approaches (biểu đồ 2 trục):** trục ngang "Level of Semantic
Integration" (Low→Medium→High), trục dọc "Availability of Integration Results" (Now→Soon→Tomorrow).
Định vị 4 cách tiếp cận: **Keyword Search** và **Probabilistic Integration** ở góc "có kết quả NGAY
nhưng mức tích hợp ngữ nghĩa THẤP"; **Domain Specific Integration** ở mức trung bình cả 2 trục; **Full
semantic integration** (chính là GAV/LAV/GLAV "truyền thống" đã học suốt từ đầu bài) ở góc "mức tích
hợp CAO nhưng phải chờ lâu (Tomorrow)" — biểu đồ này giải thích ĐỘNG LỰC chuyển sang các kỹ thuật mới
(PAYGO, keyword search...) cho bài toán Big Data: đánh đổi độ chính xác ngữ nghĩa lấy TỐC ĐỘ có kết
quả.

**Slide 72 — WebTables: Keyword Ranking [CHW+08]:** giới thiệu bài toán cụ thể đầu tiên của hướng
"tích hợp quy mô web": WebTables (Cafarella, Halevy, Wang, Wu, Zhang — PVLDB 2008, trích dẫn đầy đủ ở
slide 86). Mục tiêu: XẾP HẠNG các BẢNG (table) trên web theo từ khoá truy vấn — chú ý: KHÔNG phải xếp
hạng TRANG WEB (như Google search thông thường), cũng KHÔNG phải xếp hạng từng BẢN GHI riêng lẻ, mà
là xếp hạng nguyên cả 1 BẢNG DỮ LIỆU. 3 thách thức: (1) đặc trưng của trang web áp dụng MƠ HỒ cho
bảng nhúng bên trong nó (1 trang có thể chứa nhiều bảng khác nhau), (2) không phải mọi bảng trên 1
trang đều liên quan tới truy vấn, (3) bảng web có đặc trưng RIÊNG (vd có schema/tiêu đề cột) mà trang
web thường không có.

---

## Trang 37 — Slide 73-74: "WebTables: Keyword Ranking" (2 kỹ thuật xếp hạng, có số liệu thật)

**Slide 73 — FeatureRank:** dùng đặc trưng RIÊNG của bảng (không phải đặc trưng trang web chung
chung): số dòng, số cột, có header hay không, số ô NULL, thứ hạng tìm kiếm tài liệu của trang chứa
bảng, số lượt khớp từ khoá ở header/cột trái nhất/cột thứ nhì/toàn thân bảng — rồi dùng **hồi quy
tuyến tính (linear regression)** để ước lượng độ liên quan, với trọng số LỚN cho các đặc trưng quan
trọng nhất (vd "hits on header", "hits on leftmost column" được khoanh nổi bật trong slide). Bảng số
liệu đo chất lượng (tỉ lệ bảng điểm cao THỰC SỰ liên quan, tại các ngưỡng top-k = 10/20/30):
Naïve 0.26→0.33→0.34 so với FeatureRank 0.43→0.56→0.66 — cải thiện RÕ RỆT so với baseline ngây thơ.

**Slide 74 — SchemaRank (cải tiến thêm):** bổ sung thêm tiêu chí **"schema coherency"** (độ mạch lạc
của lược đồ) dùng **pointwise mutual information (pmi)** tính từ ACSDb (Attribute Correlation
Statistics Database — 1 CSDL thống kê riêng của nhóm tác giả, không giải thích thêm chi tiết trong
slide): $p(S)$ = tỉ lệ lược đồ DUY NHẤT có chứa tập thuộc tính $S$; $pmi(a,b) = \log(p(a,b)/(p(a)
{\cdot}p(b)))$ — công thức PMI kinh điển đo mức độ 2 thuộc tính a, b "thường đi cùng nhau" nhiều hơn
NGẪU NHIÊN; Coherency = trung bình pmi(a,b) trên MỌI cặp thuộc tính của bảng R. Kết quả thực nghiệm
cải thiện thêm: SchemaRank đạt 0.47→0.59→0.68, tốt hơn cả FeatureRank — minh hoạ nguyên tắc: thêm tín
hiệu về MỐI QUAN HỆ giữa các cột (không chỉ đặc trưng từng cột riêng lẻ) giúp xếp hạng chính xác hơn.

---

## Trang 38 — Slide 75-76: "Why is traditional integration no longer enough?" & "Traditional DI → PAYGO Architecture"

**Slide 75 — 4 lý do cụ thể (có số liệu thật, cần đọc kỹ vì đây là slide trình bày dưới dạng card
hiện đại, không phải bullet truyền thống như các slide trước):**
- **Deep Web at Scale:** ước tính khoảng **25 triệu** nguồn "deep web" (dữ liệu ẩn sau form HTML,
  không index được bởi search engine thông thường) trong mẫu nghiên cứu — nội dung đó còn LỚN HƠN cả
  "surface web" (web bề mặt có thể crawl trực tiếp). Số liệu cụ thể: "647K forms trong mẫu 25 triệu
  trang".
- **Schema Explosion:** Google Base (sản phẩm cũ của Google, tiền thân khái niệm cho Google Shopping/
  Structured Data ngày nay) có hơn **10.000 loại mặt hàng (item types)** và khoảng **100.000 lược đồ
  riêng biệt** — ví dụ riêng khái niệm "vehicle" (xe cộ) đã có hơn **50 thuộc tính khác nhau** — minh
  hoạ "database design by the masses" (thiết kế CSDL bởi đám đông, không kiểm soát tập trung) tạo ra
  dị biệt KHÔNG GIỚI HẠN.
- **Modeling Everything:** trên web, dữ liệu trải khắp MỌI lĩnh vực tri thức con người, các miền
  chồng lấp phức tạp — 1 mediated schema DUY NHẤT là bất khả thi và dễ vỡ (brittle). Ví dụ nêu: dự án
  **Cyc** (dự án AI/tri thức biểu tượng nổi tiếng, chạy khoảng 20 năm) chỉ đạt thành công HẠN CHẾ khi
  cố mô hình hoá "mọi thứ".
- **Keyword Queries:** người dùng web gõ TỪ KHOÁ, không viết truy vấn SQL có cấu trúc — hệ thống phải
  tự ĐỊNH TUYẾN (route) truy vấn tới đúng nguồn liên quan mà không có 1 lược đồ chung để dựa vào.

**Slide 76 — Traditional DI → PAYGO Architecture (bảng so sánh trực tiếp, điểm bản lề của cả phần
cuối bài):** đối chiếu song song, mỗi dòng 1 cặp đối lập:

| Traditional Data Integration | PAYGO (Pay-As-You-Go) |
|---|---|
| Single Mediated Schema — thiết kế THỦ CÔNG cho 1 miền | Schema Clusters — kho lược đồ tổ chức theo chủ đề |
| ✗ Exact Schema Mappings — bán tự động, chuyên gia xác minh | Approximate Mappings — xác suất, giữa BẤT KỲ cặp nguồn nào |
| Structured Queries — SQL/XQuery trên mediated schema | Keyword Queries + Routing — viết lại + định tuyến |
| ✗ Deterministic Answers — giả định biết đủ thông tin TỪ ĐẦU | Heterogeneous Ranking — câu trả lời XẾP HẠNG tốt nhất có thể, dưới điều kiện bất định |

Dấu ✗ trên 2 dòng "Exact Schema Mappings" và "Deterministic Answers" của cột Traditional chính là 2
điểm KHÔNG CÒN KHẢ THI ở quy mô Big Data — lý do trực tiếp dẫn tới kiến trúc PAYGO.

---

## Trang 39 — Slide 77-78: "Two ideas that define PAYGO" & "Dataspace Approach [FHM05, HFM06]"

**Slide 77 — 2 ý tưởng cốt lõi định nghĩa PAYGO:**
1. **Pay-As-You-Go Integration:** bắt đầu với RẤT ÍT "keo ngữ nghĩa" (semantic glue — tức mapping sơ
   sài ban đầu), trả lời truy vấn TỐT NHẤT CÓ THỂ với những gì đã biết, rồi LIÊN TỤC cải thiện mapping
   bằng phương pháp tự động + phản hồi người dùng. 3 cơ chế cụ thể: schema matching tự động để
   "bootstrap" (khởi động), phản hồi người dùng (cả ngầm định lẫn tường minh) để tinh chỉnh, và
   **Value of Perfect Information (VPI)** — 1 khái niệm từ lý thuyết quyết định, dùng để ưu tiên nên
   cải thiện mapping nào trước (mapping nào "đáng giá nhất" nếu biết chính xác).
2. **Uncertainty at All Levels:** dùng 1 **hình thức luận THỐNG NHẤT** để mô hình hoá độ bất định ở
   CẢ 3 tầng cùng lúc — dữ liệu, mapping ngữ nghĩa, và diễn giải truy vấn — để quyết định bất định ở
   tầng này LAN TRUYỀN gọn gàng (propagate cleanly) sang các tầng khác, không bị mâu thuẫn/mất mát
   thông tin khi truyền qua các bước xử lý.

**Slide 78 — Dataspace Approach [FHM05, HFM06] (nền tảng lý thuyết đứng sau PAYGO):**
- **Động lực:** cách tiếp cận SDI (Structured/Semantic Data Integration — tức Data Integration
  "truyền thống" đã học suốt 68 slide đầu) là **bất khả thi** (infeasible) cho BDI (Big Data
  Integration) — cụ thể: **Volume, Variety** của nguồn → chi phí mô hình hoá TRƯỚC (up-front) là
  không thể chấp nhận; **Velocity** của nguồn → duy trì kết quả tích hợp tốn kém (nguồn thay đổi quá
  nhanh so với tốc độ cập nhật mapping thủ công).
- **Nhận định mấu chốt:** cách tiếp cận **pay-as-you-go** có thể khả thi — bắt đầu với 1 dịch vụ ĐƠN
  GIẢN, hữu ích PHỔ QUÁT, rồi LẶP ĐI LẶP LẠI thêm độ phức tạp khi và nơi cần thiết [JFH08].
- Cách tiếp cận này "đã từng thành công" (approach has worked) với RDBMS, Web, Hadoop — slide liệt kê
  các ví dụ lịch sử tương tự nhưng KHÔNG giải thích chi tiết từng trường hợp, nên không thể diễn giải
  thêm nếu không tra cứu nguồn gốc cụ thể hơn (⚠️ cần xác minh thêm nếu muốn trích dẫn sâu hơn).

---

## Trang 40 — Slide 79-80: "Probabilistic Mediated Schemas [DDH08]" (2 phần)

**Slide 79 (nôm na trước):** thay vì 1 chuyên gia NGỒI THIẾT KẾ mediated schema thủ công (như toàn bộ
ví dụ Movie/Actors/Plays/Reviews xuyên suốt 68 slide đầu), ở quy mô Big Data mediated schema được
**TỰ ĐỘNG tạo ra** bằng cách soi xét (inspecting) các nguồn — cụ thể bằng kỹ thuật **phân cụm thuộc
tính nguồn (clustering of source attributes)**. Vì có nhiều nguồn (Volume) và đa dạng (Variety), kết
quả phân cụm mang tính **BẤT ĐỊNH** (uncertainty) — không còn 1 mediated schema "đúng duy nhất" nữa,
mà là NHIỀU khả năng với mức độ tin cậy khác nhau. Sơ đồ minh hoạ: 3 nguồn S1 (cột name/games/runs),
S2 (name/team/score), S4 (name/club/matches) — các đường cong nối các cột CÓ THỂ thuộc cùng 1 cụm
ngữ nghĩa (ví dụ "games" của S1 có thể cùng nhóm với "matches" của S4, đều chỉ "số trận đấu").

**Slide 80 (ví dụ P-mediated schema cụ thể):** với CÙNG sơ đồ nguồn ở slide 79, liệt kê ra NHIỀU
phương án phân cụm khả dĩ, mỗi phương án gắn 1 XÁC SUẤT:
- M1({S1.games, S4.matches}, {S1.runs, S2.score})
- M2({S1.games, S2.score}, {S1.runs, S4.matches})
- $M = \{(M1, 0.6), (M2, 0.2), (M3, 0.1), (M4, 0.1)\}$ — phân bố xác suất trên các phương án mediated
  schema khả dĩ, tổng = 1.0 — đây chính là ý nghĩa "Probabilistic" trong tên gọi: thay vì 1 schema cố
  định, hệ thống giữ cả 1 PHÂN BỐ xác suất trên nhiều schema khả dĩ.

---

## Trang 41 — Slide 81-82: "Probabilistic Mappings [DHY07, DDH09]" (2 phần)

**Slide 81:** sau khi có mediated schema XÁC SUẤT (P-mediated schema, slide 79-80), bước tiếp theo tự
nhiên là MAPPING giữa P-mediated schema và từng nguồn CŨNG phải mang tính xác suất — ký hiệu các cột
của mediated schema giờ viết tắt (n, t, g, s — có thể hiểu là name/team/games/score, dù slide không
chú giải rõ từng chữ cái tương ứng khái niệm gì — ⚠️ cần xem lại bài báo gốc DHY07/DDH09 nếu cần giải
nghĩa chính xác). Ví dụ mapping:
- G1({MS.t, S2.team, S4.club}, {MS.g, S4.matches}, {MS.s, S2.score})
- G2({MS.t, S2.team, S4.club}, {MS.g, S2.score}, {MS.s, S4.matches})
- $G = \{(G1,0.6),(G2,0.2),(G3,0.1),(G4,0.1)\}$ — cùng cấu trúc phân bố xác suất như slide 80, nhưng
  áp dụng cho MAPPING thay vì cho SCHEMA.

**Slide 82:** câu hỏi quan trọng tiếp theo — khi ĐÃ có cả schema VÀ mapping xác suất, làm sao TRẢ LỜI
TRUY VẤN? Slide nêu 2 ngữ nghĩa (semantics) khác nhau để trả lời:
- **By table semantics:** chọn 1 mapping DUY NHẤT, áp dụng CHO TOÀN BỘ các dòng (tuple) của bảng kết
  quả — đơn giản nhưng có thể không chính xác cho từng dòng riêng lẻ.
- **By tuple semantics:** các mapping KHÁC NHAU có thể đúng cho các dòng KHÁC NHAU — chính xác hơn
  nhưng phức tạp hơn nhiều về mặt tính toán. Slide chỉ nêu tên 2 hướng, KHÔNG đi sâu thuật toán cụ
  thể cho từng hướng (nằm ngoài phạm vi slide này).

---

## Trang 42 — Slide 83-84: "Keyword Search Based Integration [TJM+08]" (2 phần, ví dụ minh hoạ dần)

**Slide 83 (nôm na trước):** đây là bài toán ĐÓNG VAI TRÒ đầu ra cuối cùng của cả chuỗi kỹ thuật PAYGO
(schema clustering → probabilistic mapping → giờ là tầng người dùng tương tác): **"information need
driven integration"** — tích hợp dẫn dắt bởi NHU CẦU THÔNG TIN của người dùng, thể hiện qua TỪ KHOÁ
tìm kiếm (đúng tinh thần "Keyword Queries + Routing" đã nêu ở bảng so sánh slide 76). 3 ý tưởng:
- **Search graph:** các bảng nguồn được biểu diễn thành 1 ĐỒ THỊ với các liên kết có TRỌNG SỐ (weighted
  associations) — trọng số thể hiện mức độ liên quan/khớp ngữ nghĩa giữa các cột.
- **Query keywords:** mỗi từ khoá truy vấn được khớp (matched) tới các PHẦN TỬ cụ thể ở các nguồn
  khác nhau.
- **Derive top-k SQL view, dùng Steiner tree trên search graph:** đây là kỹ thuật thuật toán cụ thể —
  bài toán **Steiner Tree** (tìm cây có tổng trọng số nhỏ nhất nối các điểm bắt buộc trên 1 đồ thị,
  bài toán kinh điển trong lý thuyết đồ thị) được dùng để tìm ra CÁCH NỐI các bảng chứa từ khoá lại
  với nhau thành 1 view SQL hợp lý nhất.

Ví dụ minh hoạ bước đầu: 2 từ khoá "**7661**" (một con số, có thể là mã số áo cầu thủ hay tương tự)
và "**Queensland**" (tên địa danh) được khớp vào các cột cụ thể của S1 (name/games/runs) và S4
(name/club/matches) qua đường nét đứt thẳng đứng, rồi các đường cong (trọng số) nối chéo giữa các
bảng thể hiện các liên kết CÓ THỂ dùng để ghép truy vấn.

**Slide 84 (tiếp diễn, thêm 1 từ khoá):** CÙNG kịch bản slide 83 nhưng bổ sung thêm từ khoá thứ 3
"**Allan Border**" (rõ ràng là TÊN NGƯỜI — có thể là 1 cầu thủ/vận động viên cricket nổi tiếng người
Úc, dù slide không ghi chú gì thêm — ⚠️ đây là suy đoán hợp lý dựa trên tên riêng, không phải thông
tin xác nhận trong slide) khớp vào bảng S2 (name/team/score) — minh hoạ việc Search Graph MỞ RỘNG
linh hoạt khi truy vấn có NHIỀU từ khoá hơn, mỗi từ khoá thêm 1 "điểm neo" (anchor point) cho bài
toán Steiner Tree.

---

## Trang 43 — Slide 85-86: "System Architecture" & "References" (đóng bài)

**Slide 85 — System Architecture (sơ đồ tổng kết kiến trúc hệ thống PAYGO hoàn chỉnh):** chia 3 tầng,
đọc từ DƯỚI LÊN TRÊN theo đúng chú thích ở cuối sơ đồ ("Metadata flows upward · Queries flow
downward · System evolves with feedback" — metadata chảy lên, truy vấn chảy xuống, hệ thống tiến hoá
nhờ phản hồi):
- Tầng dưới cùng (nguồn dữ liệu thô): Web Sources, Deep Web, Google Base, Annotations.
- Tầng giữa **Relationship Discovery**: Crawling (thu thập), Clustering (phân cụm — đúng kỹ thuật đã
  học ở slide 79), Mapping Discovery (phát hiện mapping — đúng kỹ thuật slide 81).
- Tầng **Repository**: lưu trữ metadata đã khám phá được dưới 4 dạng chỉ mục (index): Structure
  Index, Value Index, Domain Index, Mapping Index.
- Tầng trên cùng **Query Analysis**: Query Routing (định tuyến truy vấn), Query Reformulation (viết
  lại truy vấn — đúng kỹ thuật đã học CHO Schema Mapping truyền thống ở nửa đầu bài, nay áp dụng lại
  ở quy mô lớn hơn), Query Refinement (tinh chỉnh truy vấn), Source Ranking (xếp hạng nguồn — liên hệ
  trực tiếp kỹ thuật WebTables Ranking ở slide 73-74).

Đây là slide TỔNG KẾT THỊ GIÁC cho toàn bộ phần PAYGO (slide 69-85) — mọi khái niệm rời rạc đã học
(schema clusters, probabilistic mapping, keyword search, Steiner tree...) đều có vị trí CỤ THỂ trong
đúng 1 kiến trúc hệ thống thống nhất này.

**Slide 86 — References (phần Big Data, tách riêng khỏi References ở slide 68):**
- **[CHW+08]** Cafarella, Halevy, Wang, Wu, Zhang — "WebTables: exploring the power of tables on the
  web." PVLDB 1(1):538-549, 2008 — nguồn gốc kỹ thuật WebTables slide 72-74.
- **[DDH08]** Das Sarma, Dong, Halevy — "Bootstrapping pay-as-you-go data integration systems."
  SIGMOD 2008:861-874 — nguồn gốc Probabilistic Mediated Schemas slide 79-80.
- **[DDH09]** Das Sarma, Dong, Halevy — "Data Modeling in Dataspace Support Platforms." Conceptual
  Modeling 2009:122-138.
- **[HFM06]** Halevy, Franklin, Maier — "Principles of dataspace systems." PODS 2006:1-9 — nguồn gốc
  khái niệm Dataspace ở slide 78.
- **[TJM+08]** Talukdar, Jacob, Mehmood, Crammer, Ives, Pereira, Guha — "Learning to create
  data-integrating queries." PVLDB 1(1):785-796, 2008 — nguồn gốc kỹ thuật Keyword Search Based
  Integration slide 83-84.

Toàn bộ 5 bài báo này **TRÙNG KHỚP** với danh sách tài liệu literature-review đã dùng để xây dựng nội
dung Module 05 "Bài tập lớn" của site IT5427 trong phiên làm việc trước — xác nhận các case thật đã
viết trong site đều bám đúng nguồn gốc học thuật của chính bài giảng này.
