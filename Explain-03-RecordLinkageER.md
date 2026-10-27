# Giải thích chi tiết — Record Linkage & Entity Resolution (3_RecordLinkage_ER.pdf)

Nguồn: `slides/3_RecordLinkage_ER.pdf`, GV Vũ Tuyết Trinh, SoICT-HUST. File PDF có **23 trang vật lý**,
nhưng mỗi trang in **2 slide/trang** (kiểu handout) — tổng cộng **46 slide logic** (đánh số góc dưới
phải mỗi khung từ 1 đến 46). Dưới đây giải thích đủ cả 46 slide, gộp theo 23 trang vật lý để dễ đối
chiếu ngược lại file PDF gốc.

> Lưu ý quan trọng đã xác minh lại trong lần đọc này: công thức TF/IDF dùng để giải bài tập
> "hnoi"/"hanoi" trong `BT-EditDistance-TFIDF-SchemaMapping.md` và `Quiz-BT-DataIntegration-IT5427.docx`
> nằm đúng ở **Trang vật lý 11, slide 21** (định nghĩa idf) và **Trang vật lý 11, slide 22** (ví dụ
> x=aab,y=ac,z=a) — khớp hoàn toàn với mô tả đã dùng trước đó trong phiên ("idf(t)=N/N_t, tách ký tự").

---

## Trang 1 (slide 1-2) — Mở đầu & Động lực

**Slide 1 — Record Linkage & Entity Resolution.** Trang bìa: môn do Vũ Tuyết Trinh (SoICT - HUST)
giảng dạy. Đây là chủ đề thứ 3 trong chuỗi bài giảng Data Integration (sau Data Integration overview và
Schema Alignment/GAV-LAV-GLAV), chuyển trọng tâm từ "khớp LƯỢC ĐỒ" (cột nào tương ứng cột nào) sang
"khớp DÒNG DỮ LIỆU" (bản ghi nào mô tả cùng 1 thực thể ngoài đời thực).

**Slide 2 — Motivation example.** Nêu 3 tình huống thực tế cần Record Linkage:
- **Merging two files** (gộp 2 tệp): ví dụ chấm điểm tín dụng (credit scoring, cần gộp hồ sơ từ nhiều
  ngân hàng), danh sách khách hàng (customer lists, công ty mua lại nhau cần hợp nhất CRM).
- **Survey** (khảo sát): CPS (Current Population Survey — khảo sát dân số Mỹ) và SIPP (Survey of
  Income and Program Participation) — dữ liệu giấy, phỏng vấn, nên cùng 1 người có thể xuất hiện nhiều
  lần với thông tin ghi hơi khác nhau.
- **Updating/unduplicating junk mail lists** (làm sạch danh sách gửi thư quảng cáo): gộp từ nhiều nguồn
  danh sách rồi phải loại trùng để khỏi gửi trùng.

**Nôm na:** đây chính là lý do "tại sao" cần cả chương này — dữ liệu thật luôn có nhiều bản ghi tưởng
khác nhau (sai chính tả, viết tắt, thiếu thông tin) nhưng thực ra nói về CÙNG MỘT người/vật/sự kiện.

---

## Trang 2 (slide 3-4) — Ví dụ minh hoạ & Định nghĩa

**Slide 3 — Flight records: one entity or two?** Bảng dữ liệu thật (Table 3.1 trong sách
*Big Data Integration* của Dong & Srivastava) liệt kê 11 bản ghi chuyến bay, cột gồm AL (hãng bay), FN
(số hiệu chuyến bay), DA (sân bay khởi hành), DD (ngày khởi hành), DT (giờ khởi hành), AA (sân bay đến),
AD (ngày đến), AT (giờ đến). Các dòng được tô xám (trong slide gốc là 2 màu: vàng và đỏ) để gợi ý: dòng
màu vàng mô tả CÙNG 1 chuyến bay thật (chỉ lệch nhau vài phút do nhập liệu/làm tròn), dòng màu đỏ mô tả
1 chuyến bay KHÁC. Câu hỏi đặt ra cho sinh viên: "giá trị nào là lỗi nhập liệu, giá trị nào thực sự
phân biệt 2 chuyến bay khác nhau?" — đây chính là bài toán cốt lõi của Record Linkage: không có quy tắc
"khớp tuyệt đối", phải dùng độ đo tương đồng và ngưỡng quyết định.

**Slide 4 — Record linking – Definition.** Định nghĩa gọn: Record Linking là (1) tìm các CẶP bản ghi
giữa HAI tệp mô tả cùng 1 thực thể, và (2) nhận diện bản ghi TRÙNG LẶP bên TRONG 1 tệp (dedup). Hai bài
toán nghe khác nhau nhưng dùng chung kỹ thuật độ đo tương đồng.

---

## Trang 3 (slide 5-6) — Thuật ngữ & Phát biểu bài toán

**Slide 5 — Terminology.** Trích định nghĩa học thuật (Burdick, Fagin, Kolaitis, Popa, Tan — *J.
Comput. Syst. Sci.* 2019): "**Entity Linking** là bài toán tạo liên kết giữa các bản ghi đại diện cho
các thực thể thế giới thực có liên quan theo một cách nào đó." Entity Resolution là TRƯỜNG HỢP RIÊNG
quan trọng của Entity Linking, cụ thể là nhận diện/liên kết các thực thể TRÙNG LẶP. Slide cũng liệt kê
các tên gọi đồng nghĩa hay gặp trong tài liệu khác nhau — Entity Linking ≈ Entity Linkage ≈ Record
Linkage; Entity Resolution ≈ Data Deduplication ≈ Entity Matching (cùng 1 khái niệm, nhiều cộng đồng
nghiên cứu đặt tên khác nhau, nên đọc báo đừng nhầm là 2 bài toán khác nhau). Ứng dụng thực tế: nhận
diện/gỡ nhập nhằng thực thể có tên (named entity recognition/disambiguation — vd phân biệt "Obama" là
Barack Obama hay người trùng tên khác), xây dựng tri thức/đồ thị tri thức, hệ gợi ý/mạng xã hội, tổ chức
tài chính (khớp hồ sơ cá nhân & pháp nhân để chống rửa tiền/trùng tài khoản), du lịch/vận tải/y tế.

**Slide 6 — Problem.** Định nghĩa kinh điển của **Fellegi & Sunter (1969)**, "A Theory for Record
Linkage" — bài báo nền tảng của toàn bộ lĩnh vực: "Entity Resolution là nhận diện những bản ghi trong 2
tệp đại diện cho CÙNG người/vật/sự kiện." Cho 2 tập dữ liệu A, B, với mỗi cặp bản ghi (a_i, b_j) ∈ A×B
phải quyết định 1 trong 3 nhãn: **link** (khớp), **non-link** (không khớp), hoặc **possible-link**
(chưa đủ bằng chứng — cần con người xem xét thêm). Đối lập với cách làm NGÂY THƠ **Naïve Deduplication**
(dùng `UNION DISTINCT`/group-by theo hash hoặc sort — chỉ bắt được bản ghi KHỚP TUYỆT ĐỐI từng ký tự,
ví dụ bảng "Matthias Boehm" ở IBM Research vs. "Matthias Böhm" ở TU Graz sẽ KHÔNG bị coi là trùng dù rõ
ràng cùng 1 người, chỉ khác dấu ö/oe và đổi chỗ công tác) → cần **Similarity Measures** (độ đo tương
đồng): Jaccard theo token J(A,B)=|A∩B|/|A∪B|, Edit-based (Levenshtein), hoặc phonetic (Soundex,
metaphone — có sẵn trong thư viện Python `Jellyfish`). Đây là slide bắc cầu dẫn vào toàn bộ phần còn
lại của bài.

---

## Trang 4 (slide 7-8) — Similarity Measures (mục lớn) & Ví dụ các loại lỗi

**Slide 7 — Similarity Measures.** Slide chuyển tiếp (divider), chỉ có tiêu đề, báo hiệu bắt đầu mục
lớn thứ 2 của bài: các phương pháp đo độ tương đồng giữa 2 chuỗi.

**Slide 8 — Examples.** Liệt kê 5 LOẠI lỗi/khác biệt thường gặp khiến 2 chuỗi tả cùng 1 thực thể nhưng
không giống hệt nhau — đây là "động lực" cho việc cần nhiều phương pháp đo khác nhau (không có 1
phương pháp nào bắt được hết mọi loại lỗi):
1. Lỗi gõ phím/lỗi OCR: "David Smith" vs "Davod Smith" (1 ký tự sai) → hợp với **Edit Distance**.
2. Khác quy ước định dạng: "10/8" vs "Oct 8" (cùng ngày tháng, viết khác kiểu) → cần chuẩn hoá trước
   khi so sánh, không phương pháp string similarity nào tự xử lý được.
3. Viết tắt/rút gọn tuỳ ý: "Daniel Walker Herbert Smith" vs "Dan W. Smith".
4. Tên khác nhau, biệt danh: "William Smith" vs "Bill Smith" (không phải lỗi gõ, mà là tên gọi khác
   nhau hoàn toàn về mặt chuỗi ký tự — Edit Distance sẽ cho điểm RẤT THẤP dù đúng là cùng 1 người).
5. Đảo thứ tự các phần trong chuỗi: "Dept. of Computer Science, UW-Madison" vs "Computer Science Dept.,
   UW-Madison" (cùng nội dung, khác thứ tự từ) → hợp với **Set-based measures** (Jaccard/TF-IDF, vì các
   phương pháp này coi chuỗi là 1 TẬP token, không quan tâm thứ tự) hơn là Edit Distance (vốn tính theo
   thứ tự ký tự nên sẽ cho điểm thấp sai lệch với trường hợp này).

---

## Trang 5 (slide 9-10) — Phát biểu hình thức & 2 thách thức

**Slide 9 — Problem Description.** Phát biểu hình thức: cho 2 tập chuỗi X, Y, tìm MỌI cặp (x,y) với
x∈X, y∈Y mà cùng chỉ 1 thực thể thật — gọi (x,y) là 1 **match**. Ví dụ minh hoạ (Set X = {Dave Smith,
Joe Wilson, Dan Smith}, Set Y = {David D. Smith, Daniel W. Smith}) → đáp án match đúng: (x₁,y₁) = "Dave
Smith"↔"David D. Smith", (x₃,y₂) = "Dan Smith"↔"Daniel W. Smith" (chú ý x₂ "Joe Wilson" KHÔNG match với
ai cả — không phải lúc nào mọi bản ghi cũng có cặp). Slide chốt: bài toán có **2 thách thức lớn**:
**accuracy** (độ chính xác — quyết định đúng/sai) và **scalability** (khả năng mở rộng — chạy nổi với
dữ liệu lớn).

**Slide 10 — Accuracy Challenges.** Giải pháp cho thách thức ĐỘ CHÍNH XÁC: dùng 1 độ đo tương đồng
s(x,y) ∈ [0,1] — s càng cao, x và y càng có khả năng là match; khai báo x,y match nếu s(x,y) ≥ ngưỡng
t. Ghi chú quan trọng: cũng có khái niệm **distance/cost measure** — cùng ý tưởng nhưng NGƯỢC chiều
(giá trị càng NHỎ thì càng giống nhau) — ví dụ chính Edit Distance ở các slide sau là 1 "distance
measure" (không phải similarity measure trực tiếp, phải chuyển đổi qua công thức `s=1-d/max(len)`).

---

## Trang 6 (slide 11-12) — Thách thức quy mô & Tổng quan các phương pháp

**Slide 11 — Scalability Challenges.** Thách thức thứ 2: áp dụng s(x,y) cho MỌI cặp là không khả thi vì
độ phức tạp **bậc 2 theo kích thước dữ liệu** (n bản ghi → n² cặp cần so). Giải pháp: chỉ áp dụng s(x,y)
cho các cặp "có triển vọng nhất", dùng 1 hàm `FindCands` (tìm ứng viên) — với mỗi x ∈ X, dùng
`FindCands` để tìm tập ứng viên Z ⊆ Y (nhỏ hơn nhiều so với toàn bộ Y), rồi chỉ tính s(x,y) cho y ∈ Z.
Đây chính là ý tưởng gốc của **Blocking** sẽ học kỹ ở phần Entity Resolution Pipeline phía sau (slide
38-39) — slide này gieo trước khái niệm, chưa đi sâu kỹ thuật.

**Slide 12 — Methods.** Bảng phân loại tổng quan 4 NHÓM phương pháp đo tương đồng chuỗi, là "mục lục"
cho toàn bộ phần còn lại của nửa đầu bài giảng:
- **Sequence-based** (dựa trên THỨ TỰ ký tự): edit distance, Needleman-Wunch, affine gap, Smith-
  Waterman, Jaro, Jaro-Winkler.
- **Set-based** (coi chuỗi là TẬP token, bỏ qua thứ tự): overlap, Jaccard, TF/IDF.
- **Hybrid** (lai giữa 2 nhóm trên): generalized Jaccard, soft TF/IDF, Monge-Elkan.
- **Phonetic** (dựa trên CÁCH PHÁT ÂM): Soundex.

---

## Trang 7 (slide 13-14) — Edit Distance: định nghĩa & chuyển sang similarity

**Slide 13 — Edit Distance.** Còn gọi là **Levenshtein distance**. d(x,y) = chi phí NHỎ NHẤT để biến
đổi x thành y, dùng dãy các phép toán, MỖI PHÉP TỐN CHI PHÍ 1: xoá 1 ký tự, chèn 1 ký tự, thay thế 1 ký
tự bằng ký tự khác. Ví dụ: x="David Smiths", y="Davidd Simth" → d(x,y)=4, qua dãy: chèn 'd' (sau
"David"), thay 'm' bằng 'i', thay 'i' bằng 'm', xoá ký tự cuối 's' của x. (Đây CHÍNH LÀ phương pháp đã
dùng để giải bài tập "hnoi"/"hanoi" trong `BT.pdf` — chỉ khác là bài tập đơn giản hơn, chỉ cần 1 phép
chèn.)

**Slide 14 — Edit Distance (similarity).** Edit Distance mô hình hoá đúng các lỗi gõ phím thường gặp
(chèn thừa ký tự, đảo 2 ký tự,...) nên distance CÀNG NHỎ thì độ tương đồng CÀNG CAO. Công thức chuyển
từ distance sang similarity: **s(x,y) = 1 − d(x,y) / max(length(x), length(y))**. Áp dụng cho ví dụ
trên: s("David Smiths","Davidd Simth") = 1 − 4/max(12,12) = **0,67**. (Đây CHÍNH LÀ công thức đã dùng
để tính s("hnoi","hanoi") = 1 − 1/5 = 0,8 trong bài tập đã giải — khớp đúng 100% với đáp án chữa chính
thức của giảng viên.)

---

## Trang 8 (slide 15-16) — Quy hoạch động tính Edit Distance

**Slide 15 — Computing Edit Distance using Dynamic Programming.** Đặt x = x₁x₂…xₙ, y = y₁y₂…yₘ.
d(i,j) = edit distance giữa TIỀN TỐ i ký tự đầu của x và TIỀN TỐ j ký tự đầu của y. Công thức truy hồi
(2 cách viết tương đương trên slide):
```
d(i,j) = min { d(i-1,j-1)            nếu xᵢ = yⱼ        // copy, không tốn chi phí
               d(i-1,j-1) + 1        nếu xᵢ ≠ yⱼ        // thay thế (substitute)
               d(i-1,j) + 1                              // xoá xᵢ (delete)
               d(i,j-1) + 1 }                             // chèn yⱼ (insert)
```
hoặc gộp gọn: d(i,j) = min{ d(i-1,j-1)+c(xᵢ,yⱼ), d(i-1,j)+1, d(i,j-1)+1 }, với c(xᵢ,yⱼ)=0 nếu
xᵢ=yⱼ, ngược lại =1. Đây CHÍNH LÀ công thức đã dùng để dựng bảng DP cho "hnoi"/"hanoi" trong bài tập đã
giải trước đó.

**Slide 16 — Example (x=dva, y=dave).** Ví dụ minh hoạ ĐẦY ĐỦ việc dựng bảng DP kèm MŨI TÊN truy vết
(backtrace): bảng 4×5 (hàng x0..x3 ứng "","d","v","a"; cột y0..y4 ứng "","d","a","v","e"). Kết quả cuối
d(3,4) = **2**: đường truy vết cho thấy phép biến đổi tối ưu là **thay 'a' thành 'e'** (vị trí cuối) và
**chèn 'a' (ngay sau 'd')** — đúng 2 thao tác biến "dva" thành "dave". Chi phí tính DP là O(|x|·|y|) —
ghi chú về độ phức tạp thuật toán, quan trọng khi bàn tới khả năng mở rộng (scalability) ở phần sau.

---

## Trang 9 (slide 17-18) — Set-based Similarity: sinh token & Overlap Measure

**Slide 17 — Set-based Similarity Measures.** Thay vì coi chuỗi là DÃY ký tự có thứ tự (như Edit
Distance), coi chuỗi là TẬP (hoặc multi-set) các "token", rồi dùng các phép toán tập hợp để tính độ
tương đồng. Hai cách sinh token phổ biến: (1) **tách theo từ** (phân cách bởi khoảng trắng), có thể kèm
stemming (đưa về gốc từ) và loại bỏ stop-word (the, and, of,...) — vd "david smith" → {"david",
"smith"}; (2) **q-gram**: các chuỗi con độ dài q liên tiếp, có thêm ký tự đặc biệt `#` đánh dấu
đầu/cuối chuỗi — vd "david smith" với q=3 → {##d, #da, dav, avi, ..., h##}. **Đây chính là phương pháp
"tách theo ký tự" đã dùng trong bài tập TF/IDF "hnoi"/"hanoi"** (thực chất là trường hợp đặc biệt q=1,
không ký tự biên #, đúng theo ví dụ gốc x=aab ở slide 22 phía sau).

**Slide 18 — The Overlap Measure.** Bx = tập token của x, By = tập token của y. O(x,y) = |Bx ∩ By| —
đơn giản là ĐẾM SỐ TOKEN CHUNG. Ví dụ: x="dave", y="dav" (q-gram q=2, có ký tự biên #): Bx={#d, da, av,
ve, e#}, By={#d, da, av, v#} → O(x,y) = 3 (3 token chung: #d, da, av). Overlap Measure là độ đo ĐƠN GIẢN
NHẤT trong nhóm set-based, nhưng có nhược điểm không chuẩn hoá theo độ dài (2 chuỗi dài sẽ có overlap
cao hơn 2 chuỗi ngắn một cách "giả tạo") — dẫn tới cần Jaccard ở slide tiếp theo.

---

## Trang 10 (slide 19-20) — Jaccard Measure & Động lực TF/IDF

**Slide 19 — The Jaccard Measure.** J(x,y) = |Bx ∩ By| / |Bx ∪ By| — khắc phục nhược điểm của Overlap
bằng cách CHUẨN HOÁ theo kích thước HỢP của 2 tập (không chỉ đếm số chung, mà đếm theo TỶ LỆ so với tổng
số token khác nhau). Cùng ví dụ x="dave", y="dav": J(x,y) = 3/6 = 0,5 (6 = |Bx∪By| = {#d,da,av,ve,e#,v#}).
Ghi chú: Jaccard là độ đo **rất phổ biến trong thực tế** — nhiều hệ thống blocking/schema matching dùng
Jaccard làm mặc định.

**Slide 20 — The TF/IDF Measure: Motivation.** Nêu ĐIỂM YẾU chung của Edit Distance VÀ Jaccard thuần:
cả 2 đều coi MỌI token/ký tự "quan trọng như nhau". Ví dụ kinh điển: x="Apple Corporation, CA",
y="IBM Corporation, CA", z="Apple Corp". Dùng Edit Distance hay Jaccard thuần, s(x,y) > s(x,z) (vì x,y
dài và giống nhau nhiều ký tự/token hơn) → hệ thống sẽ SAI khi ghép x với y (2 công ty khác nhau: Apple
và IBM!) thay vì x với z (cùng là Apple, chỉ viết tắt khác). TF/IDF giải quyết bằng cách NHẬN RA "Apple"
là từ PHÂN BIỆT (hiếm, mang nhiều thông tin) còn "Corporation" và "CA" là từ PHỔ BIẾN (ít thông tin) —
nhờ đó ghép đúng x với z. Đây là động lực lý thuyết thông tin (Information Retrieval) đứng sau toàn bộ
công thức TF/IDF ở các slide kế tiếp.

---

## Trang 11 (slide 21-22) — Công thức TF/IDF & Ví dụ (QUAN TRỌNG — nguồn gốc bài tập đã giải)

**Slide 21 — Term Frequencies and Inverse Document Frequencies.** Giả sử x, y lấy từ 1 TẬP HỢP (collection)
các chuỗi; mỗi chuỗi được chuyển thành 1 "túi term" (bag of terms) gọi là **document**. Định nghĩa:
- **tf(t,d)** = số lần term t xuất hiện trong document d (term frequency).
- **idf(t) = N / N_t** — N là số document trong tập hợp, N_t là số document có chứa t (inverse document
  frequency). **Ghi chú NGUYÊN VĂN trên slide, cực kỳ quan trọng:** "trong thực tế, idf(t) thường được
  định nghĩa là log(N/N_t), ở đây ta dùng công thức ĐƠN GIẢN trên để định nghĩa idf(t)" — tức bản thân
  giảng viên XÁC NHẬN đây là phiên bản ĐƠN GIẢN HOÁ (không log) dùng riêng cho mục đích giảng dạy/làm
  bài tập trong môn này, khác với công thức log chuẩn trong sách IR. Đây CHÍNH LÀ căn cứ đã dùng để giải
  bài tập TF/IDF "hnoi"/"hanoi" bằng `idf=N/N_t` (không log) — khớp đúng 100% với đáp án chữa của BT.pdf.

**Slide 22 — Example.** Ví dụ gốc minh hoạ công thức trên, với 3 "document" x="aab", y="ac", z="a"
(chính là ví dụ mà bài tập "hnoi"/"hanoi" mô phỏng lại theo cùng khuôn mẫu):
- Bx = {a, a, b} (multi-set — chú ý 'a' xuất hiện 2 lần trong "aab")
- By = {a, c}
- Bz = {a}
- tf(a,x)=2, tf(b,x)=1, ... tf(c,z)=0 (minh hoạ term không xuất hiện trong document thì tf=0)
- idf(a) = 3/3 = 1 (vì 'a' xuất hiện trong CẢ 3 document x,y,z → N_t=3=N → idf thấp nhất, 'a' là term
  PHỔ BIẾN nhất, ít mang thông tin phân biệt)
- idf(b) = 3/1 = 3 (b chỉ xuất hiện trong 1 document duy nhất là x → idf cao → term HIẾM, phân biệt mạnh)
- idf(c) = 3/1 = 3 (tương tự, c chỉ có ở y)

**Liên hệ trực tiếp với bài tập đã giải:** bài "hnoi"/"hanoi" chỉ có N=2 (2 xâu đang so, không phải 3),
và tách theo TỪNG KÝ TỰ thay vì ví dụ gốc tách theo... thực ra ví dụ gốc NÀY cũng đã tách theo ký tự
(document "aab" → bag of CHARACTERS {a,a,b}, không phải bag of words) — xác nhận lại: quy ước "tách
term theo ký tự cho chuỗi 1 từ duy nhất" trong bài tập đã giải là ĐÚNG theo đúng tinh thần ví dụ gốc này,
không phải suy diễn tuỳ tiện.

---

## Trang 12 (slide 23-24) — Feature Vectors & Công thức Similarity cosine

**Slide 23 — Feature Vectors.** Mỗi document d được chuyển thành 1 **feature vector v_d**: với mỗi term
t, có 1 thành phần v_d(t) = tf(t,d) × idf(t) (công thức đơn giản nhất — có dampening phức tạp hơn ở
slide 25). Bảng kết quả cho ví dụ x=aab,y=ac,z=a (3 cột ứng 3 term a,b,c):
```
       a    b    c
v_x    2    3    0     (= tf(a,x)·idf(a)=2·1=2;  tf(b,x)·idf(b)=1·3=3;  tf(c,x)·idf(c)=0·3=0)
v_y    1    0    3     (= tf(a,y)·idf(a)=1·1=1;  tf(b,y)=0;             tf(c,y)·idf(c)=1·3=3)
v_z    1    0    0     (= tf(a,z)·idf(a)=1·1=1;  tf(b,z)=0; tf(c,z)=0)
```

**Slide 24 — TF/IDF Similarity Score.** Cho p, q là 2 chuỗi, T = tập TẤT CẢ term trong toàn bộ tập hợp.
Feature vector v_p, v_q là các vector trong không gian |T| chiều (mỗi chiều ứng 1 term). **Điểm TF/IDF
của p,q = cosine góc giữa v_p và v_q**:
$$s(p,q) = \frac{\sum_{t\in T} v_p(t)\cdot v_q(t)}{\sqrt{\sum_{t\in T}v_p(t)^2}\cdot\sqrt{\sum_{t\in T}v_q(t)^2}}$$
**Đây CHÍNH XÁC là công thức đã dùng để tính Sim(x,y)=1/√2 cho bài "hnoi"/"hanoi"** — khớp từng ký
hiệu (tử số = tích vô hướng, mẫu số = tích 2 độ dài Euclid của 2 vector).

---

## Trang 13 (slide 25-26) — Dampening/chuẩn hoá TF/IDF & Generalized Jaccard

**Slide 25 — TF/IDF Similarity Score (tiếp).** Giải thích SAO điểm TF/IDF "đúng": điểm CAO khi 2 chuỗi
chia sẻ nhiều term có tf CAO (xuất hiện nhiều) **nhưng** idf THẤP tức term đó KHÔNG phổ biến ở các
document khác (⇒ term "phân biệt" thật sự). Trong THỰC TẾ (không phải bài tập đơn giản của môn này),
người ta hay **dampen** (giảm độ nhạy) TF và IDF bằng log: `v_d(t) = log(tf(t,d)+1) · log(idf(t))` thay
vì nhân thẳng `tf·idf` — tránh 1 term xuất hiện QUÁ NHIỀU lần lấn át hoàn toàn các term khác. Cũng có
bước **chuẩn hoá vector** (normalize) để vector có độ dài 1: `v_d(t) = v_d(t) / √(Σv_d(t)²)` — sau bước
này, công thức cosine ở slide trước sẽ rút gọn chỉ còn tích vô hướng (vì mẫu số đã =1). *(Bài tập
"hnoi"/"hanoi" trong môn này dùng công thức ĐƠN GIẢN tf·idf không dampening/không chuẩn hoá trước —
đúng theo đúng công thức gốc ở slide 21, KHÔNG áp dụng biến thể "thực tế" này.)*

**Slide 26 — Generalized Jaccard Measure.** Nhược điểm của Jaccard THUẦN: 2 token phải GIỐNG HỆT NHAU
mới tính là "chung" — quá cứng nhắc trong 1 số trường hợp. Ví dụ: so khớp 2 "taxonomic node" mô tả công
ty — "Energy & Transportation" vs "Transportation, Energy, & Gas": về lý thuyết Jaccard hợp (vì đều có
cùng từ, chỉ đảo thứ tự — set-based nên thứ tự không ảnh hưởng), nhưng trong THỰC TẾ nếu 1 token bị viết
sai chính tả nhẹ (vd "energy" vs "eneryg") thì Jaccard thuần sẽ coi 2 token này là KHÁC NHAU HOÀN TOÀN
(0 hoặc 1, không có "gần giống") → cần **Generalized Jaccard** để "làm mềm" phép so khớp token.

---

## Trang 14 (slide 27-28) — Thuật toán & ví dụ Generalized Jaccard

**Slide 27 — Generalized Jaccard Measure (thuật toán).** Cho Bx={x1,...,xn}, By={y1,...,ym}. **Bước 1:**
tìm các CẶP token sẽ vào tập "overlap mềm" — áp dụng 1 độ đo tương đồng s (vd Jaro-Winkler) cho MỌI cặp
(xi,yj), chỉ giữ các cặp có điểm ≥ ngưỡng α, tạo thành 1 **đồ thị hai phía (bipartite graph) G**; tìm
**maximum-weight matching M** trong G (mỗi token chỉ được ghép với tối đa 1 token phía bên kia, chọn tổ
hợp ghép có TỔNG trọng số lớn nhất — bài toán ghép cặp tối ưu kinh điển trong lý thuyết đồ thị). **Bước
2:** trả về trọng số CHUẨN HOÁ của M làm điểm Generalized Jaccard:
`GJ(x,y) = Σ_{(xi,yj)∈M} s(xi,yj) / (|Bx| + |By| - |M|)`.

**Slide 28 — An Example.** Hình minh hoạ đồ thị 2 phía: Bx={a,b,c}, By={p,q}, các cạnh có trọng số
tương đồng (a-p: 0.7, a-q: 0.5, b-p: 0.8, b-q: 0.9, c-p: 0.2, c-q: 0.1). Hình (a) vẽ TẤT CẢ cạnh có thể;
hình (b) lọc còn các cạnh "đáng chú ý"; hình (c) chỉ giữ lại MATCHING TỐI ƯU: a↔p (0.7) và b↔q (0.9) —
c không ghép được với ai (bỏ dư). Điểm Generalized Jaccard = (0,7+0,9)/(3+2−2) = **0,53** (mẫu số: |Bx|=3,
|By|=2, |M|=2 cặp ghép được).

---

## Trang 15 (slide 29-30) — Soft TF/IDF & ví dụ

**Slide 29 — The Soft TF/IDF Measure.** Dùng TF/IDF làm độ đo "tầng cao", nhưng cho phép token GẦN
GIỐNG (không nhất thiết giống hệt) vẫn được tính vào — khắc phục trường hợp lỗi chính tả làm mất điểm
TF/IDF thuần hoàn toàn (vd "Apple Corporation, CA" so với "Aple Corp" — "Apple" bị viết sai thành
"Aple"). **Bước 1:** tính `close(x,y,k)` = tập các term t ∈ Bx có ÍT NHẤT 1 term u ∈ By "gần" nó, tức
s'(t,u) ≥ k (s' là 1 độ đo cơ bản như Jaro-Winkler, k là ngưỡng định trước). **Bước 2:** tính s(x,y) như
công thức TF/IDF truyền thống, nhưng MỖI thành phần được NHÂN THÊM trọng số s' của cặp token gần nhất:
`s(x,y) = Σ_{t∈close(x,y,k)} v_x(t)·v_y(u*)·s'(t,u*)`, với u*∈By là token tối đa hoá s'(t,u) trong số
mọi u∈By.

**Slide 30 — An Example.** x="abcd" → Bx={a,b,c,d}; y="aa'b'cd'" → By={a,a',b',c,d'} (các ký tự có dấu
phẩy trên — ví dụ trừu tượng mô phỏng "gần giống nhưng không giống hệt"). Các cặp token gần nhau có điểm
tương đồng ghi trên hình: a↔a:1(giống hệt), a↔a':0.7, b↔b':0.8, d↔d':0.6. Với ngưỡng 0,75:
close(x,y,0.75) = {a,b,c} (chỉ "d" bị loại vì cặp gần nhất d↔d'=0.6 < 0.75). Công thức tính:
`s(x,y) = v_x(a)·v_y(a)·1 + v_x(b)·v_y(b')·0,8 + v_x(c)·v_y(c)·1` (c tự nó khớp tuyệt đối nên hệ số 1,
không cần thêm biến thể c').

---

## Trang 16 (slide 31-32) — Phonetic Similarity & bắt đầu Soundex

**Slide 31 — Phonetic Similarity Measures.** Nhóm phương pháp thứ 4: so khớp chuỗi dựa trên CÁCH PHÁT
ÂM thay vì cách VIẾT. Rất hiệu quả cho TÊN RIÊNG (names) — vốn hay xuất hiện ở nhiều biến thể viết khác
nhau nhưng ĐỌC GIỐNG NHAU: "Meyer, Meier, Mire" hoặc "Smith, Smithe, Smythe" — các cặp này có Edit
Distance/Jaccard khá thấp (nhiều ký tự khác) nhưng về bản chất là CÙNG 1 TÊN. **Soundex** là thuật toán
phổ biến nhất trong nhóm này.

**Slide 32 — The Soundex Measure.** Dùng chủ yếu để khớp HỌ (surname). Ánh xạ 1 họ x thành **mã 4 ký
tự**; 2 họ được coi là tương đồng nếu CÙNG MÃ. Thuật toán 4 bước: (1) giữ nguyên ký tự ĐẦU TIÊN của x,
các bước sau chỉ xử lý PHẦN CÒN LẠI; (2) loại bỏ mọi chữ W, H, rồi thay các chữ còn lại bằng chữ số theo
bảng: B,F,P,V→1; C,G,J,K,Q,S,X,Z→2; D,T→3; L→4; M,N→5; R→6 (các chữ số này nhóm các âm "nghe giống nhau"
— vd B/F/P/V đều là âm môi); (3) rút gọn dãy chữ số GIỐNG NHAU LIÊN TIẾP thành 1 chữ số duy nhất; (4) bỏ
mọi ký tự KHÔNG phải chữ số, trả về 4 ký tự đầu làm mã Soundex (tự động gồm 1 chữ cái + 3 chữ số).

---

## Trang 17 (slide 33-34) — Ví dụ Soundex & chuyển tiếp Entity Resolution

**Slide 33 — The Soundex Measure (ví dụ).** x="Ashcraft": sau bước 2 → "A226a13" (chữ thường = ký tự bị
loại W/H), sau bước 3 (rút gọn chữ số liên tiếp trùng) → "A26a13", bước 4 (bỏ ký tự không phải số, lấy 4
ký tự) → trả về **"A261"**. Mã Soundex được ĐỆM THÊM số 0 nếu không đủ 4 ký tự. Ví dụ khác: "Robert" và
"Rupert" → cùng mã **R163** (minh hoạ đúng mục đích: 2 tên NGHE gần giống nhau, VIẾT khác nhau, vẫn bắt
được). **Hạn chế quan trọng** (giảng viên nêu rõ, không giấu điểm yếu): Soundex THẤT BẠI với "Gough" và
"Goff" (đọc gần giống nhưng mã khác), hay "Jawornicki" và "Yavornitzky". Lý do gốc rễ: Soundex được
**thiết kế ban đầu cho tên gốc Caucasian** (dù sau này chứng minh hoạt động tốt với tên nhiều nguồn gốc
khác), nhưng **không hoạt động tốt với tên gốc Đông Á** — vì tiếng các ngôn ngữ Đông Á dùng NGUYÊN ÂM để
phân biệt nghĩa, trong khi Soundex **bỏ qua hoàn toàn nguyên âm** (chỉ mã hoá phụ âm).

**Slide 34 — Entity Resolution.** Slide chuyển tiếp (divider) — kết thúc phần 1 (các độ đo tương đồng
string-level), bắt đầu phần 2: đặt các độ đo này vào 1 **QUY TRÌNH** hoàn chỉnh để giải bài toán Entity
Resolution ở quy mô thực tế.

---

## Trang 18 (slide 35-36) — Pipeline tổng thể & Lịch sử 50 năm

**Slide 35 — Entity Resolution Pipeline.** Sơ đồ PIPELINE 4 BƯỚC (hình mũi tên liên tiếp): **Prepare
Data → Blocking/Sorting → Matching → Clustering**, có 1 "người" (hình icon) giám sát/can thiệp ở các
bước. Ví dụ minh hoạ CHẠY QUA cả 4 bước với 8 bản ghi thô {A1,C1,D1,A2,B1,B2,C2,B3}:
1. **Blocking/Sorting**: nhóm sơ bộ các bản ghi "có khả năng" trùng nhau vào các CỤM ỨNG VIÊN nhỏ — ở
   đây tạo 3 cụm ứng viên: {A1,A2}, {C1,C2,D1}, {B1,B2,B3} (giảm số cặp cần so sánh thật kỹ từ bậc 2
   xuống nhỏ hơn nhiều — đúng ý tưởng `FindCands` đã gieo ở slide 11).
2. **Matching**: trong MỖI cụm ứng viên, so khớp CHI TIẾT từng cặp bằng độ đo tương đồng — kết quả xác
   định các liên kết thật: A1-A2 khớp, C1-C2 khớp, D1 không khớp với C1/C2 (dù cùng cụm ứng viên — đây
   chính là lý do Matching cần làm SAU Blocking, Blocking chỉ là bước LỌC THÔ, không phải quyết định
   cuối cùng), B1-B3 khớp, B2 không khớp với B1/B3.
3. **Clustering**: gộp các liên kết đã xác định thành các CỤM THỰC THỂ cuối cùng — kết quả: cụm A gồm
   bản ghi gốc {r1,r4}, cụm C gồm {r2,r7}, cụm D chỉ có {r3} (không khớp ai), cụm B gồm {r5,r6,r8}.

**Slide 36 — Entity Linking Approaches.** Trích hình "50 Years of Entity Linkage" từ bài báo Dong &
Rekatsinas, *Data Integration and Machine Learning: A Natural Synergy* (PVLDB 2018) — dòng thời gian
phát triển kỹ thuật của CẢ lĩnh vực qua 4 giai đoạn:
- **1969 (Pre-ML)**: Rule-based và stats-based — Blocking theo tên giống hệt, Matching theo độ tương
  đồng TRUNG BÌNH của các thuộc tính, Clustering bằng bao đóng bắc cầu (transitive closure).
- **~2000 (Early ML) → ~2015 (ML)**: Supervised/Unsupervised learning — Matching bằng Decision Tree,
  SVM (F-measure 70-90% với ~500 nhãn), Clustering bằng Correlation clustering, Markov clustering.
- **2018 (Deep ML)**: Supervised learning hiện đại hơn — Random Forest cho matching (F-measure >95% với
  ~1 triệu nhãn), Active learning cho blocking & matching (F-measure 80-98% chỉ cần ~1000 nhãn — ít nhãn
  hơn nhiều nhưng vẫn đạt độ chính xác cao, nhờ chọn LỌC mẫu cần gán nhãn một cách thông minh).
- Deep learning / Entity embedding — xu hướng mới nhất (sẽ nói kỹ ở slide 41).

---

## Trang 19 (slide 37-38) — Chuẩn bị dữ liệu & Blocking/Sorting

**Slide 37 — Data Preparation.** Bước 1 của pipeline, gồm 3 việc: **Schema Matching and Mapping** (tạo
lược đồ ĐỒNG NHẤT để so sánh được — chính là nội dung GAV/LAV/GLAV đã học ở `2_SchemaAlignment.pdf`,
cộng thêm tách các thuộc tính GHÉP/composite thành thuộc tính đơn); **Normalization** (chuẩn hoá: bỏ ký
tự đặc biệt/khoảng trắng thừa, stemming, viết hoa/thường thống nhất, loại từ thừa và giải nghĩa viết
tắt); **Data Cleaning** (sửa lỗi hỏng/không nhất quán trong dữ liệu). Ghi chú bên lề "Autonomous,
heterogeneous systems" nhắc lại bối cảnh: dữ liệu đến từ NHIỀU HỆ THỐNG ĐỘC LẬP, không đồng bộ.

**Slide 38 — Blocking & Sorting.** **Naïve All-Pairs**: cách brute-force, so TẤT CẢ cặp — n(n-1)/2 cặp,
độ phức tạp O(n²), không khả thi với dữ liệu lớn. **Blocking/Partitioning**: tạo các KHỐI (block) nhỏ
gồm các bản ghi "giống nhau SƠ BỘ" để chỉ so khớp CHI TIẾT trong CÙNG 1 khối — cách cơ bản nhất: nhóm
theo giá trị GIỐNG HỆT trên 1 thuộc tính chọn trước (vd cùng họ "name"). Có nhiều kiểu **predicate**
(tiêu chí blocking) phức tạp hơn: so khớp cả trường (whole field), so khớp 1 token trong trường (token
field), số nguyên chung (common integer), cùng n ký tự đầu (same n char start), n-gram. Có thể kết hợp
nhiều predicate bằng **hybrid** (hội/tuyển — disjunction/conjunction).

---

## Trang 20 (slide 39-40) — Blocking nâng cao & bắt đầu Matching

**Slide 39 — Blocking & Sorting (2).** **Sorted Neighborhood**: định nghĩa "khoá sắp xếp" (sorting key,
tương tự blocking key), SẮP XẾP toàn bộ bản ghi theo khoá này, rồi định nghĩa 1 **cửa sổ trượt** kích
thước m (vd 100) và chỉ so khớp TOÀN BỘ CẶP TRONG cửa sổ đó — ý tưởng: nếu 2 bản ghi thật sự trùng nhau
thì sau khi sắp xếp chúng sẽ đứng GẦN NHAU. **Blocking via Word Embeddings và LSH** (Locality-Sensitive
Hashing — kỹ thuật hiện đại hơn): tính embedding cho từng từ/thuộc tính rồi gộp thành embedding cho cả
bộ (tuple), dùng K hàm băm h(t) tạo ra 1 mã băm K-chiều, dùng L bảng băm (mỗi bảng có K hàm băm riêng) —
ý tưởng LSH: các vector GẦN NHAU trong không gian embedding có xu hướng rơi vào CÙNG 1 "xô" (bucket)
băm, nên chỉ cần so khớp chi tiết trong cùng 1 xô.

**Slide 40 — Matching.** Bắt đầu phần Matching (bước 3 pipeline): **Basic Similarity Measures** — chọn
1 độ đo tương đồng sim(r,r') và 2 ngưỡng: ngưỡng CAO θ_h và ngưỡng THẤP θ_l; độ tương đồng của cả bản
ghi = TRUNG BÌNH độ tương đồng từng thuộc tính; quyết định: **Match** nếu sim(r,r') > θ_h, **Non-match**
nếu sim(r,r') < θ_l, **Possible match** (cần xem xét thêm, thường do con người) nếu θ_l < sim(r,r') <
θ_h — đúng tinh thần "3 nhãn link/non-link/possible-link" đã nêu ở Fellegi & Sunter (slide 6).
**Learned Matchers (ML truyền thống)**: Phase 1 học độ đo tương đồng chuỗi cho từng thuộc tính được
chọn; Phase 2 học QUYẾT ĐỊNH match từ các độ đo tương đồng đó; cần chọn mẫu để gán nhãn (đủ số lượng,
phù hợp, cân bằng lớp); các mô hình dùng: SVM, decision tree, logistic regression, random forest,
XGBoost. Trích 3 bài báo tham khảo (Bilenko&Mooney KDD 2003; Köpcke/Thor/Rahm PVLDB 2010; Dong ICDE
2019 — "Building a Broad Knowledge Graph for Products", liên quan hệ tri thức sản phẩm Amazon).

---

## Trang 21 (slide 41-42) — Matching bằng Deep Learning & dữ liệu nhãn/Transfer Learning

**Slide 41 — Matching (2): Deep Learning for ER.** Học biểu diễn TỰ ĐỘNG từ văn bản (tránh phải tự tay
thiết kế đặc trưng — feature engineering), tận dụng pre-trained word embedding để nắm bắt NGỮ NGHĨA
(không bị giới hạn bởi khác biệt CÚ PHÁP/chính tả như các độ đo string truyền thống). **Ví dụ DeepER**
(Ebraheem et al., PVLDB 2018): kiến trúc mạng — mỗi "tuple" (bản ghi) được đưa qua embedding lookup
layer cho từng từ, rồi 1 composition layer (trung bình hoặc LSTM) gộp thành 1 vector đại diện cho CẢ bộ
(Distributed Tuple Representation), 2 vector của 2 bản ghi được so khớp bằng similarity layer (hiệu
tuyệt đối / tích Hadamard), rồi qua dense layer + classification layer ra quyết định match/non-match.
**Ví dụ Magellan** (Mudgal et al., SIGMOD 2018): hệ thống deep learning cho cả văn bản SẠCH lẫn văn bản
BẨN (dirty data).

**Slide 42 — Matching (3): dữ liệu nhãn & chuyển giao học tập.** **Labeled Data** (dữ liệu có nhãn) —
2 vấn đề lớn: khan hiếm (cần chuyên gia gán nhãn, tốn kém) và lệch lớp (class skew — số cặp "match" ít
hơn RẤT NHIỀU so với "non-match" trong thực tế, vì đa số cặp ngẫu nhiên không khớp). Biểu đồ F1 trên tập
dữ liệu chuẩn **DBLP-ACM** cho thấy Deep Learning đạt F1 cao nhất (~98) ngay cả khi số lượng mẫu gán
nhãn còn ít, vượt Decision Tree/SVM/Naive Bayes — nhưng cần NHIỀU dữ liệu nhãn hơn ở điểm khởi đầu (xem
đường cong deep learning xuất phát thấp nhất ở 0 mẫu). Công thức F1 được nhắc lại: F1 = 2·(precision·
recall)/(precision+recall). 2 hướng khắc phục vấn đề thiếu nhãn: **Transfer Learning** (học mô hình từ 1
bối cảnh ER có NHIỀU dữ liệu, rồi tinh chỉnh bằng ít dữ liệu ở bối cảnh MỚI) và **Active Learning**
(chọn LỌC mẫu cần gán nhãn để giảm thiểu số lượng nhãn cần có mà vẫn đạt hiệu quả cao).

---

## Trang 22 (slide 43-44) — Clustering & Khử trùng lặp tăng dần

**Slide 43 — Clustering.** **Nhắc lại Connected Components** (thành phần liên thông — thuật toán phân
tán quen thuộc): xác định các thành phần liên thông của 1 đồ thị (các đồ thị con gồm toàn nút liên kết
với nhau); thuật toán lan truyền giá trị LỚN NHẤT (max) hiện có tới các nút lân cận nếu khác giá trị
hiện tại, dừng khi không còn thông điệp nào lan truyền nữa. Hình minh hoạ 4 bước: Step 0 (giá trị khởi
tạo rời rạc: nút 1,2,3,4 ở 1 cụm với các nhãn khác nhau; nút 5,6,7 ở cụm khác) → Step 1, Step 2 → Step 3
(hội tụ: cụm trái toàn bộ về nhãn "4", cụm phải toàn bộ về nhãn "7"). **3 cách tiếp cận Clustering**: (1)
**Basic**: connected components/bao đóng bắc cầu trên các cạnh có sim > θ_h — nhược điểm: dễ tạo ra CỤM
QUÁ LỚN gồm cả bản ghi không giống nhau (nếu có 1 "cầu nối" sai làm 2 cụm lẽ ra tách biệt bị nối lại);
(2) **Correlation clustering**: cắt đồ thị dựa trên cạnh +/- theo độ tương đồng, bài toán tối ưu TOÀN
CỤC là NP-hard (không có thuật toán nhanh đảm bảo tối ưu, phải dùng heuristic); (3) **Markov
clustering**: mô phỏng dòng chảy ngẫu nhiên (stochastic flow) qua random walk trên đồ thị. Trích dẫn
Hassanzadeh/Chiang/Miller/Lee, PVLDB 2009.

**Slide 44 — Incremental Data Deduplication.** Vấn đề: dữ liệu cập nhật LIÊN TỤC (incremental stream of
updates) khiến kết quả đã tính TRƯỚC ĐÓ bị lỗi thời — mục tiêu là đạt kết quả GIỐNG HOẶC GẦN GIỐNG kết
quả tính lại từ đầu (batch) nhưng NHANH HƠN ĐÁNG KỂ. Cách tiếp cận: record linkage TĂNG DẦN đầu-cuối cho
bản ghi mới/thay đổi; duy trì TĂNG DẦN đồ thị tương đồng và việc gom cụm (không tính lại từ đầu mỗi lần
có bản ghi mới); đồ thị khởi tạo ban đầu dựng bằng correlation clustering; cập nhật THAM LAM (greedy)
trong thời gian đa thức, gồm 4 thao tác: nối trực tiếp các thành phần từ phần đồ thị tăng thêm ΔG; GỘP
(merge) 2 cụm nếu cho kết quả tốt hơn; TÁCH (split) 1 cụm thành 2 nếu cho kết quả tốt hơn; DI CHUYỂN
(move) 1 nút giữa 2 cụm nếu cho kết quả tốt hơn. Trích dẫn Gruenheid, Dong, Srivastava — "Incremental
Record Linkage", PVLDB 2014.

---

## Trang 23 (slide 45-46) — Tài liệu tham khảo & Hỏi đáp

**Slide 45 — References.** 4 tài liệu tham khảo GỐC của toàn bộ bài giảng:
1. Dong & Srivastava, *Big Data Integration*, Chapter 3 (nguồn của bảng flight records ở slide 3).
2. Peter Christen, *Data Matching: Concepts and Techniques for Record Linkage*.
3. Fellegi & Sunter, 1969, "A Theory for Record Linkage" (nguồn định nghĩa 3-nhãn link/non-link/
   possible-link ở slide 6 — bài báo NỀN TẢNG của toàn bộ lĩnh vực Record Linkage).
4. Winkler, 1995, "Matching and Record Linkage," trong B.G. Cox et al. (chủ biên), *Business Survey
   Methods*, New York, J. Wiley.

**Slide 46 — Q&A.** Slide kết thúc bài giảng (chỉ có biểu tượng hỏi-đáp), không có nội dung chữ.

---

## Tổng kết liên hệ với các bài tập đã làm trong phiên này

- Công thức **Edit Distance → similarity** (slide 14: `s=1-d/max(len)`) và **thuật toán DP** (slide 15)
  chính là công thức đã dùng cho câu a bài "hnoi"/"hanoi" trong `BT-EditDistance-TFIDF-SchemaMapping.md`
  — khớp 100% với đáp án chữa chính thức của giảng viên.
- Công thức **idf(t)=N/N_t** (KHÔNG log, slide 21) và ví dụ tách TERM theo KÝ TỰ (slide 22, x=aab,y=ac,
  z=a) cùng công thức **cosine similarity** (slide 24) chính là 3 mảnh ghép đã dùng cho câu b bài
  "hnoi"/"hanoi" — khớp 100% với đáp án chữa chính thức.
- Phần **Entity Resolution Pipeline** (slide 35: Prepare Data → Blocking → Matching → Clustering) chính
  là khung lý thuyết đứng sau các "Case thật" (Papadakis2020 về Blocking, Zeakis2023 về Matching bằng
  embedding) đã chèn vào Module 04 của site IT5427 tự học trong phiên trước.
