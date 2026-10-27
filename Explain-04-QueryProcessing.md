# Giải thích chi tiết từng trang — 4_QueryProcessing.pdf

Nguồn: `slides/4_QueryProcessing.pdf` (GV Vũ Tuyết Trinh, SoICT-HUST). File PDF có 28 trang vật lý,
nhưng là bản in "2 slide/trang" (mỗi trang PDF chứa 2 slide bài giảng xếp chồng) — tổng cộng **56
slide bài giảng**. Dưới đây giải thích đủ cả 56 slide, nhóm theo đúng 28 trang PDF vật lý để bạn dễ
đối chiếu ngược lại file gốc. Không bịa thêm số liệu/công thức không có trên slide; chỗ nào mơ hồ ghi
rõ "⚠️ cần xác minh thêm".

---

## Trang PDF 1 — Slide 1-2: Mở đầu & kiến trúc tổng quan Query Processing

**Slide 1 — Tiêu đề: "Query Processing for Data Integration – Be Adaptive!"** (Vu Tuyet Trinh, SoICT-HUST).
Đây là slide mở đầu, không có nội dung kỹ thuật — nhưng tiêu đề đã nói lên "thông điệp cốt lõi" của cả
bộ slide: trong tích hợp dữ liệu, xử lý truy vấn KHÔNG thể làm theo kiểu "tối ưu 1 lần rồi chạy" như
CSDL truyền thống, mà phải **thích nghi (adaptive)** trong suốt quá trình thực thi.

**Slide 2 — "Query Processing" (sơ đồ kiến trúc tổng quát).** Nôm na: đây là bản đồ tổng thể cho thấy
1 câu truy vấn đi qua những "trạm" nào trước khi ra kết quả. Sơ đồ gồm:
- `Query` → đi vào khối **Query reformulation** (viết lại truy vấn từ mediated schema thành truy vấn
  trên các nguồn thật — đúng bước Query/View Unfolding của GAV hoặc Answering-Queries-Using-Views của
  LAV đã học ở `2_SchemaAlignment.pdf`) → sinh ra **Logical query plan**.
- Logical query plan → **Query optimizer** → sinh ra **Physical query plan** (chọn thuật toán join cụ
  thể, thứ tự join cụ thể).
- Physical query plan → **Execution engine** → gửi xuống 5 cặp **wrapper + source** (wrapper là tầng
  "phiên dịch" giữa mô hình dữ liệu chung và nguồn thật, đúng vai trò wrapper đã học ở Quiz DI câu 25).
- Có 1 mũi tên **"Replanning request"** đi ngược từ Execution engine quay lại Query optimizer — đây
  chính là điểm khác biệt lớn nhất so với CSDL truyền thống: Execution engine có thể YÊU CẦU tối ưu
  lại giữa chừng nếu phát hiện kế hoạch đang chạy không tốt (kế hoạch "tưởng hay" nhưng thực tế chạy
  chậm) — mầm mống của toàn bộ phần "Adaptive Query Processing" sẽ học ở các slide sau.

---

## Trang PDF 2 — Slide 3-4: Vì sao cần thích nghi? & ví dụ Approximate Query Processing

**Slide 3 — "Growing data in heterogeneous environment".** Liệt kê 3V kinh điển của Big Data áp vào
bài toán query processing: **Volume** (dữ liệu tăng → tốn lưu trữ/xử lý hơn), **Velocity** (dữ liệu
đến nhanh → áp lực phải phân tích kịp thời), **Variety** (dữ liệu có cấu trúc/bán cấu trúc/phi cấu
trúc → cần xử lý khác nhau). Kết luận của slide: tài nguyên tăng theo (mua thêm máy) **không bao giờ
đủ** ("never enough") — nên hướng đi đúng không phải là "tăng tài nguyên" mà là "xử lý **hiệu quả
hơn**" — đây là lý do dẫn tới 2 hướng kỹ thuật sẽ học: Adaptive Query Processing và Approximate Query
Processing (AQP).

**Slide 4 — "Example" (histogram xấp xỉ).** Đây là ví dụ đầu tiên minh hoạ ý tưởng AQP (Approximate
Query Processing — trả lời xấp xỉ nhanh thay vì trả lời chính xác chậm). Truy vấn
`SELECT COUNT(x) WHERE 5.1 < x < 10.3` trên 1 histogram có các khoảng [0-5]:4, [5-10]:21, [10-15]:13,
[15-20]:11 tuple. Đáp án CHÍNH XÁC là 21 (đếm đúng khoảng [5-10] chứa biên 5.1-10.3... thực ra khoảng
hỏi lệch biên histogram). Đáp án XẤP XỈ được tính bằng cách NỘI SUY theo tỉ lệ phần khoảng histogram
bị truy vấn phủ tới: `(4.9/5)*21 + (0.3/5)*13 = 21.36` — tức giả định dữ liệu phân bố ĐỀU trong mỗi
khoảng histogram, rồi lấy tỉ lệ phần trăm khoảng [5.1,10) thuộc bucket [5,10) (gần như trọn vẹn,
4.9/5) cộng với phần nhỏ [10,10.3) thuộc bucket [10,15) (0.3/5). Bài học: AQP đánh đổi ĐỘ CHÍNH XÁC
TUYỆT ĐỐI lấy TỐC ĐỘ — chỉ cần đọc histogram đã tổng hợp sẵn, không cần quét hết dữ liệu gốc.

---

## Trang PDF 3 — Slide 5-6: Thêm 2 ví dụ AQP (Sampling & Sketch)

**Slide 5 — "Example (2)" (Sampling).** Truy vấn `SELECT SUM(prod) FROM clicks GROUP BY prod` chạy
trên TOÀN BỘ bảng `Clicks` mất **Time: 3.10⁵** (300.000 đơn vị thời gian), còn chạy trên **mẫu 0.01%**
(`Clicks (0.01% Sample)`) chỉ mất **Time: 39** — nhanh hơn ~7700 lần. 2 biểu đồ tròn (pie chart) của
dữ liệu đầy đủ và dữ liệu mẫu có hình dạng GẦN GIỐNG NHAU (tỉ lệ Prod A/B/C/D/E xấp xỉ bằng nhau) — ý
tưởng là: nếu chỉ cần TỈ LỆ tương đối giữa các nhóm (không cần số tuyệt đối chính xác), lấy mẫu ngẫu
nhiên nhỏ là đủ, đổi lại tốc độ nhanh vượt trội. Đây chính là kỹ thuật **Sampling-based AQP**.

**Slide 6 — "Example (3)" (Sketch — đếm distinct values xấp xỉ).** Truy vấn
`SELECT COUNT(DISTINCT x)`. Đáp án CHÍNH XÁC = 4 (đếm thủ công số giá trị phân biệt trong dãy
5,3,5,2,2,1,1,3,2,5). Đáp án XẤP XỈ dùng kỹ thuật **sketch** (cấu trúc dữ liệu tóm tắt gọn, đây là
thuật toán kiểu Flajolet-Martin/HyperLogLog): băm (hash) từng giá trị bằng 1 hàm băm chuẩn hoá về
đoạn [0,1], giữ lại D=4 giá trị hash NHỎ NHẤT (ký hiệu U₍₁₎,U₍₂₎,U₍₃₎,U₍₄₎ = 0.226, 0.413, 0.599,
0.857), rồi dùng công thức kỳ vọng thống kê `E[U₍₂₎] = 2/(D+1)` để suy ngược ra số lượng giá trị
phân biệt thật `D ≈ 2/U₍₂₎ − 1 = 2/0.413 − 1 ≈ 3.84` — rất gần với đáp án đúng 4, mà KHÔNG cần lưu
toàn bộ tập giá trị, chỉ cần lưu D số hash nhỏ nhất. ⚠️ Công thức gốc của sketch loại này (AMS/FM
sketch) phức tạp hơn 1 dòng trên slide — nếu cần làm bài tập chính xác về sketch, nên tra thêm tài
liệu gốc (Flajolet-Martin 1985) thay vì chỉ dựa vào slide tóm tắt này.

---

## Trang PDF 4 — Slide 7-8: Thách thức & 3 hướng kỹ thuật mới

**Slide 7 — "Challenges".** Liệt kê 6 thách thức thực tế của query processing trong môi trường tích
hợp dữ liệu lớn: (1) **Incompleteness and heterogeneity** — dữ liệu vẫn còn lỗi/thiếu sau khi đã làm
sạch (không bao giờ sạch 100%); (2) **Scale** — tốc độ tăng dữ liệu có thể VƯỢT tốc độ tăng tài
nguyên tính toán; (3) **Changing I/O systems** ảnh hưởng tới thuật toán truy vấn, điều khiển tương
tranh, khôi phục lỗi; (4) **Replication and partitioning** làm phức tạp bộ điều phối tập trung
(coordinator); (5) **Resource failure** có thể làm GIÁN ĐOẠN quá trình thực thi phân tán; (6) **Data
movement và tài nguyên quá tải** làm chậm cả pipeline truy vấn. Đây đều là hệ quả trực tiếp của việc
dữ liệu nằm RẢI RÁC ở nhiều nguồn tự trị (autonomous sources) — khác hẳn CSDL tập trung truyền thống.

**Slide 8 — "New dimensions".** Đưa ra 3 "trục" kỹ thuật mới cần có để đối phó với 6 thách thức trên:
**Adaptive** (thích nghi), **Approximate** (xấp xỉ), **Scalable** (mở rộng được). Biểu đồ "Cambrian
Explosion of Data" minh hoạ dữ liệu tăng theo cấp số nhân (đặc biệt dữ liệu cảm biến/sensor tăng 50x
so với dữ liệu doanh nghiệp truyền thống). Sơ đồ cây quyết định "How to deal with data explosion?" chỉ
ra 2 nhánh: **Parallel/Distributed Computing** (nhược điểm: tốn kém tiền thuê máy chủ như EC2, không
thân thiện môi trường, không phải thuật toán nào cũng song song hoá được dễ dàng, nhiều truy vấn đồng
thời làm giảm hiệu năng) và **AQP** (ưu điểm: rẻ hơn, "xanh" hơn — ít tốn tài nguyên tính toán) — 2
nhánh này **có thể kết hợp cùng nhau** ("Can combine both"), không loại trừ nhau.

---

## Trang PDF 5 — Slide 9-10: Dàn ý bài giảng & nền tảng tối ưu truy vấn cổ điển

**Slide 9 — "Outline".** Dàn ý 4 phần của bài giảng: (1) ôn nhanh query processing truyền thống, (2)
thách thức trong bối cảnh tích hợp dữ liệu, (3) kỹ thuật Adaptive Query Processing, (4) kỹ thuật
Approximate Query Processing. (Phần còn lại của slide bám sát đúng 4 mục này.)

**Slide 10 — "Query Optimization and Processing (As Established in System R [SAC+'79])".** Đây là nền
tảng LÝ THUYẾT CỔ ĐIỂN mà mọi hệ quản trị CSDL quan hệ hiện đại đều kế thừa, đến từ hệ System R của
IBM (1979) — "ông tổ" của tối ưu hoá truy vấn dựa trên chi phí (cost-based optimization). Ý tưởng: hệ
thống `UPDATE STATISTICS` định kỳ để biết **cardinalities** (số dòng mỗi bảng) và **index lo/hi key**
(khoảng giá trị chỉ mục) của các bảng (vd Professor, Course, Student). Khi có truy vấn JOIN 3 bảng
(`SELECT * FROM Professor P, Course C, Student S WHERE P.pid=C.pid AND S.sid=C.sid`), optimizer dùng
**Dynamic Programming + Pruning Heuristics** để tìm thứ tự join tốt nhất trong số rất nhiều khả năng,
dựa trên các con số thống kê đã thu thập sẵn. ⚠️ Điểm mấu chốt cần nhớ cho các slide sau: TOÀN BỘ cách
làm này dựa trên giả định có sẵn **thống kê đáng tin cậy** về dữ liệu — giả định này SẼ SỤP ĐỔ trong
môi trường tích hợp dữ liệu (xem slide 22 "But in Data Integration…").

---

## Trang PDF 6 — Slide 11-12: Quy trình tối ưu truyền thống & siêu dữ liệu cần có

**Slide 11 — "Traditional Query Processing".** Ví dụ cụ thể: truy vấn tìm người bán (`S.sname`) từ
bảng Purchase/Person với điều kiện `city='seattle' AND phone>'5430000'`. Quy trình gồm 2 bước:
**(1) Query optimization** — biến đổi truy vấn SQL thành 1 cây kế hoạch vật lý cụ thể: dưới cùng là
2 phép quét bảng (Purchase bằng Table scan, Person bằng Index scan nhờ có điều kiện lọc theo cột có
chỉ mục), ở giữa là phép lọc theo City='seattle' và phone>'5430000' trên Person, trên cùng là phép
Join `Buyer=name` thực hiện bằng thuật toán **Simple Nested Loops**. **(2) Query execution** — chạy
đúng cây kế hoạch đó. Đây là mô hình "tối ưu 1 lần, chạy y nguyên" — điểm sẽ bị phá vỡ ở phần Adaptive
Query Processing phía sau.

**Slide 12 — "Meta-Data for Optimization".** Liệt kê các loại siêu dữ liệu (metadata) mà optimizer
truyền thống CẦN có để ra quyết định đúng: kích thước bảng (sizes of tables), thông tin độ chọn lọc
(selectivity — bao nhiêu dòng ứng với mỗi giá trị cột, bao nhiêu dòng trong 1 khoảng giá trị,
cardinality của view), và chỉ mục (B-tree, hash index, thứ tự sắp xếp). Nôm na: optimizer giống như
1 người lập kế hoạch đi đường — càng biết rõ "đoạn nào đông xe, đoạn nào có đường tắt" thì càng chọn
được lộ trình tốt. Trong tích hợp dữ liệu, các "bản đồ giao thông" này THƯỜNG KHÔNG CÓ SẴN hoặc không
đáng tin cậy vì nguồn dữ liệu là của bên thứ ba, tự trị.

---

## Trang PDF 7 — Slide 13-14: Các bước tối ưu & kỹ thuật biến đổi đại số

**Slide 13 — "Query Optimization Steps".** 3 bước chính của 1 bộ tối ưu truy vấn cổ điển: **(1) Biến
đổi đại số** (algebraic transformations — có thể rất phức tạp vì có nhiều phép biến đổi tương đương
về mặt kết quả nhưng khác nhau về chi phí); **(2) Chọn thứ tự join** (có rất nhiều cách sắp thứ tự
join giữa N bảng — số lượng tăng theo giai thừa, không thể duyệt hết); **(3) Chọn thuật toán cho mỗi
nút trong cây kế hoạch** (mỗi phép toán quan hệ như Join có nhiều cách hiện thực khác nhau — Nested
Loop, Hash Join, Sort-Merge Join, mỗi cách có chi phí khác nhau tuỳ tình huống — xem chi tiết ở các
slide 17-21 phía sau).

**Slide 14 — "Algebraic Transformation — Predicate Pushdown".** Minh hoạ kỹ thuật biến đổi đại số
kinh điển nhất: **đẩy phép chọn (selection) xuống càng sớm càng tốt** trong cây kế hoạch. Cây bên trái
thực hiện Join trước rồi mới lọc `bid=100` và `rating>5` ở TRÊN CÙNG (sau join) — tốn kém vì phải join
nhiều dòng chưa cần thiết. Cây bên phải ĐẨY 2 phép lọc XUỐNG DƯỚI, lọc ngay lúc quét bảng `Reserves`
(bid=100) và `Sailors` (rating>5) rồi mới join — giảm số dòng cần xử lý ở các bước sau. Nguyên lý nêu
rõ trên slide: "càng xử lý phép chọn sớm, càng ít tuple phải thao tác ở các tầng cao hơn trong cây".
Slide có để ngỏ câu hỏi "Disadvantages?" (nhược điểm?) — ⚠️ không có câu trả lời trên slide, nhưng về
mặt lý thuyết chung, nhược điểm là: nếu 1 phép lọc có độ chọn lọc THẤP (giữ lại gần hết dữ liệu) thì
việc đẩy xuống sớm không giúp ích nhiều, thậm chí có thể làm mất cơ hội dùng chỉ mục tối ưu hơn ở bước
join — cần ước lượng chi phí thực tế thay vì áp dụng máy móc.

---

## Trang PDF 8 — Slide 15-16: Cây join và cách sắp xếp Left-Deep

**Slide 15 — "Determining Join Ordering".** Với N bảng R₁, R₂, …, Rₙ cần join với nhau, có RẤT NHIỀU
cách sắp xếp thứ tự join khác nhau, mỗi cách tạo ra 1 **cây join** (join tree) khác nhau — vd với 4
bảng R₁,R₂,R₃,R₄, một cây có thể là `(R₃⋈R₁) ⋈ (R₂⋈R₄)` (join cân đối 2 nhánh). Ý chính của slide:
**1 cây join = 1 kế hoạch thực thi cụ thể** — bài toán tối ưu join ordering chính là bài toán tìm ra
cây join có chi phí thấp nhất trong số rất nhiều cây có thể.

**Slide 16 — "Left Deep Trees".** Một dạng cây join ĐẶC BIỆT, trong đó mỗi phép join luôn có 1 nhánh
là 1 bảng gốc (không phải kết quả join khác) — nhìn hình cây "lệch hẳn về bên trái" (R₃⋈R₁, rồi join
với R₅, rồi join với R₂, rồi join với R₄, tạo thành chuỗi nối tiếp chứ không phải cây cân đối). Ưu
điểm nêu trên slide: **tốt cho pipelining** — vì luôn có 1 "dòng chảy" tuyến tính xuyên suốt cây, kết
quả join ở bước trước có thể đưa NGAY vào bước join tiếp theo mà không cần chờ vật lý hoá (materialize)
toàn bộ, giúp giảm I/O trung gian. Đây là lý do hầu hết optimizer thực tế (bao gồm cả các hệ CSDL
thương mại) chỉ duyệt tìm trong không gian các cây Left-Deep thay vì mọi cây có thể, để giảm độ phức
tạp tìm kiếm.

---

## Trang PDF 9 — Slide 17-18: Thuật toán Nested-Loop Join và Hash Join

**Slide 17 — "Nested-loop-join (NLJ)".** Nguyên lý: đọc quan hệ R "bên ngoài" (external/outer), với
MỖI dòng của R, lặp (iterate) qua toàn bộ quan hệ S "bên trong" (internal/inner) để tìm dòng khớp
(matching). Đặc điểm: "**one-and-a-half pass, non-blocking**" — nghĩa là R chỉ cần đọc 1 lần, còn S
có thể phải đọc lại nhiều lần (1.5 lần đọc tổng thể theo cách tính riêng của thuật toán), và thuật
toán KHÔNG bị "block" (không cần đợi đọc hết 1 quan hệ mới bắt đầu sinh kết quả) — đây là tính chất
RẤT quan trọng cho pipelining/streaming. Có 3 biến thể: Tuple-based NLJ (so khớp từng dòng một — chậm
nhất), Block-based NLJ (so khớp theo từng khối dòng — giảm số lần đọc đĩa), Index-based NLJ (dùng chỉ
mục trên S để tìm khớp nhanh thay vì quét tuần tự).

**Slide 18 — "Hash Join (HJ)".** Nguyên lý: dùng 1 bảng băm (Hash Table — HT) xây dựng từ quan hệ R.
Đặc điểm: "**two-pass, blocking algorithm**" — PHẢI đọc xong toàn bộ R để xây xong HT trước (đây là
tính "blocking" — không sinh được kết quả cho tới khi hoàn tất bước build), rồi mới "probe" (dò) từng
dòng của S vào HT để tìm khớp. Quy trình 2 bước rõ ràng: **build** (xây HT của R) → **probe** (mỗi
dòng S tra vào HT, tìm dòng R có cùng giá trị hash). So với NLJ, Hash Join hiệu quả hơn khi 2 quan hệ
không có chỉ mục sẵn, nhưng đánh đổi lại là tính "blocking" (không streaming được ngay từ đầu) — điểm
yếu này chính là động lực cho thuật toán cải tiến **Symmetric Hash Join** sẽ học ở slide 27-28.

---

## Trang PDF 10 — Slide 19-20: Chi phí Hash Join & Sort-Merge Join

**Slide 19 — "Cost of Hash-Join".** Công thức chi phí I/O của Hash Join khi dữ liệu KHÔNG vừa hết
trong bộ nhớ (cần chia partition): ở **pha phân hoạch (partitioning)**, phải đọc+ghi CẢ HAI quan hệ,
chi phí = `2(M+N)` (M, N là số trang/block của 2 quan hệ R và S); ở **pha so khớp (matching)**, đọc cả
2 quan hệ 1 lần nữa, chi phí = `M+N`. Tổng chi phí xấp xỉ `3(M+N)` I/O.

**Slide 20 — "Sort-Merge Join".** Nguyên lý: SẮP XẾP cả R và S theo cột join, sau đó quét ("merge")
đồng thời cả 2 theo kiểu: tiến con trỏ quét R cho tới khi dòng R hiện tại ≥ dòng S hiện tại, rồi tiến
con trỏ quét S cho tới khi dòng S hiện tại ≥ dòng R hiện tại — lặp lại tới khi 2 dòng bằng nhau; khi
đó TẤT CẢ các dòng R và S có cùng giá trị khớp được xuất ra thành từng cặp `<r,s>`, rồi tiếp tục quét
tiếp. Đây là thuật toán tận dụng dữ liệu ĐÃ SẮP XẾP SẴN (hoặc chấp nhận trả chi phí sắp xếp trước) để
tránh phải so khớp kiểu "mọi cặp với mọi cặp" như NLJ.

---

## Trang PDF 11 — Slide 21-22: Chi phí Sort-Merge Join & vấn đề riêng của tích hợp dữ liệu

**Slide 21 — "Cost of Sort-Merge Join".** Chi phí: R được quét 1 lần; mỗi nhóm (group) của S được
quét 1 lần cho MỖI dòng R khớp với nó. Công thức tổng: `M·logM + N·logN + (M+N)` (hai số hạng đầu là
chi phí SẮP XẾP R và S bằng thuật toán sắp xếp ngoài — external sort — độ phức tạp N·logN; số hạng
cuối là chi phí quét-trộn sau khi đã sắp xếp). Lưu ý trên slide: trong trường hợp XẤU NHẤT (hiếm gặp),
chi phí quét-trộn `M+N` có thể phình ra tới `M*N` nếu có quá nhiều dòng trùng giá trị join. So sánh
với Hash Join: cả 2 đều có chi phí tối thiểu khoảng `3(M+N)` I/O khi đủ bộ nhớ, nhưng **Hash Join
vượt trội hơn khi kích thước 2 quan hệ CHÊNH LỆCH NHIỀU**, và Hash Join còn dễ **song song hoá**
(parallelize) hơn Sort-Merge Join.

**Slide 22 — "But in Data Integration…".** Đây là slide BẢN LỀ — chuyển từ lý thuyết cổ điển sang vấn
đề thật của môn học. 3 vấn đề cốt lõi khi áp lý thuyết trên vào tích hợp dữ liệu: **(1) Hầu như không
có thống kê** (few statistics, if any) — vì các nguồn dữ liệu tự trị (autonomous sources), hệ tích hợp
không kiểm soát được, không thể "UPDATE STATISTICS" như CSDL nội bộ ở slide 10. **(2) Độ trễ và lỗi
không lường trước** (unanticipated delays and failures) — do các nguồn phụ thuộc mạng (network-bound),
dẫn tới hệ quả quan trọng: "**1 kế hoạch TRÔNG có vẻ tốt nhưng CHẠY lại không tốt**" (vì ước lượng chi
phí ban đầu dựa trên giả định sai). Từ đó suy ra 2 mục tiêu cần ưu tiên khác hẳn optimizer cổ điển:
tối ưu **thời gian ra tuple ĐẦU TIÊN** (thay vì tổng thời gian hoàn tất), và **giảm tải cho nguồn**
(vì nguồn là tài nguyên bên ngoài, dùng quá nhiều sẽ bị chặn/giảm hiệu năng).

---

## Trang PDF 12 — Slide 23-24: Tổng quan Adaptive Query Processing & chính sách định tuyến

**Slide 23 — "Overview of adaptive query processing".** Định nghĩa Adaptive Query Processing (theo
trích dẫn [HFC+00] — Hellerstein, Franklin, Chandrasekaran và đồng sự, 2000): 1 hệ thống "**nhận thông
tin từ môi trường xung quanh nó → dùng thông tin đó để quyết định hành vi → lặp lại 2 bước trên theo 1
vòng phản hồi (feedback loop)**" — đúng nguyên lý điều khiển học (control theory) áp vào xử lý truy
vấn. 3 **mức độ thích nghi** (levels of adaptation), từ "nhẹ" tới "nặng": **(1) Operators algorithms**
— tự thích nghi thuật toán của TỪNG toán tử riêng lẻ (vd Ripple Join, Symmetric Hash Join ở slide
25-28); **(2) Operators order** — thay đổi THỨ TỰ các toán tử giữa chừng (re-scheduling, vd Query
Scrambling ở slide 34-36); **(3) Structure of plan** — thay đổi hẳn CẤU TRÚC cây kế hoạch (re-
optimizing, vd Mid-query re-optimization ở slide 31-33).

**Slide 24 — "Some Possible Routing Policies".** 3 chính sách định tuyến tuple cụ thể cho Adaptive
Query Processing: **Deterministic** — theo dõi liên tục chi phí & độ chọn lọc (selectivity), định kỳ
tối ưu lại bằng cách xếp hạng (rank ordering), hoặc dùng thuật toán A-Greedy cho các vị từ (predicate)
có tương quan với nhau; **Lottery scheduling** — mỗi toán tử chạy trong 1 luồng (thread) riêng có hàng
đợi đầu vào, được cấp "vé số" (tickets) theo tỉ lệ số tuple vào/ra, tuple được định tuyến tới toán tử
còn chỗ trống dựa trên số vé và "áp lực ngược" (backpressure) — ý tưởng là "bốc thăm có trọng số"; và
**Content-based routing** — định tuyến khác nhau tuỳ theo GIÁ TRỊ thuộc tính của từng tuple cụ thể
(không phải tất cả tuple đi cùng 1 đường).

---

## Trang PDF 13 — Slide 25-26: Toán tử thích nghi Ripple Join

**Slide 25 & 26 — "Adaptive operator – RippleJoin"** (2 slide liên tiếp, cùng 1 chủ đề, phần 2 bổ
sung thêm chi tiết). RippleJoin là phần MỞ RỘNG của Nested-Loop Join (NLJ) đã học ở slide 17, với 2
đặc tính mới: **xác định "khoảnh khắc đối xứng"** (determining moments of symmetry — thời điểm có thể
đổi vai trò "bên ngoài"/"bên trong" giữa 2 quan hệ R và S mà không ảnh hưởng kết quả) và **xử lý được
tốc độ dữ liệu đến dao động thất thường** (fluctuant arrival data rate — dữ liệu từ nguồn không đến
đều đặn). Khác với NLJ cổ điển (luôn cố định R là "ngoài", S là "trong"), RippleJoin có thể đảo vai
trò linh hoạt — rất phù hợp khi 1 trong 2 nguồn đột nhiên chậm lại, tránh bị "đứng hình" chờ nguồn
chậm. Đây chính là ví dụ cụ thể cho mức độ thích nghi "Operators algorithms" đã nêu ở slide 23.

---

## Trang PDF 14 — Slide 27-28: Toán tử thích nghi Symmetric Hash Join (SHJ)

**Slide 27 & 28 — "Adaptive operator – Symmetric Hash Join (SHJ)"** (2 slide liên tiếp cùng chủ đề).
SHJ là phần mở rộng của Hash Join (HJ) đã học ở slide 18, giải quyết đúng điểm yếu "blocking" của HJ
thường: thay vì chỉ xây 1 Hash Table cho R rồi mới probe S, SHJ xây **2 Hash Table cùng lúc** — 1 cho
R, 1 cho S. Khi 1 dòng mới đến từ R, nó vừa được ĐƯA VÀO Hash Table R (build), vừa được DÒ NGAY
(probe) vào Hash Table S hiện có để tìm khớp tức thì — và ngược lại với dòng đến từ S. Nhờ "build và
probe đối xứng 2 chiều" như vậy, SHJ sinh được kết quả NGAY khi có dữ liệu tới, không cần đợi đọc hết
1 quan hệ nào — giải quyết đúng tính "blocking" là nhược điểm của HJ gốc, đồng thời vẫn xử lý được tốc
độ dữ liệu đến dao động thất thường (giống RippleJoin ở slide 25-26).

---

## Trang PDF 15 — Slide 29-30: Toán tử XJoin & kế hoạch truy vấn động

**Slide 29 — "Adaptive operator - XJoin".** XJoin là bước mở rộng TIẾP THEO của Symmetric Hash Join
(SHJ), giải quyết thêm vấn đề **bộ nhớ không đủ** (insufficient memory) mà SHJ chưa xử lý — khi dữ
liệu quá lớn, không thể giữ cả 2 Hash Table hoàn toàn trong RAM. Giải pháp: lưu 1 PHẦN của các Hash
Table ra đĩa (storing a part of HTs to disk) khi cần ("flush"). XJoin hoạt động theo **3 giai đoạn**:
**mem-mem** (cả 2 bên còn vừa bộ nhớ, giống SHJ bình thường), **mem-disk** (1 bên đã tràn ra đĩa, bên
kia dò cả phần trong RAM lẫn phần trên đĩa), và **cleaning** (giai đoạn dọn/hoàn thiện kết quả còn
thiếu sau khi nguồn dữ liệu đã đọc hết). Đây là lời giải cho câu hỏi "điều gì xảy ra khi SHJ gặp dữ
liệu QUÁ LỚN để giữ hết trong bộ nhớ" — một ví dụ cho thấy các toán tử thích nghi được xây dựng THEO
TỪNG LỚP, mỗi lớp giải quyết thêm 1 hạn chế của lớp trước.

**Slide 30 — "Dynamic query plan".** Vấn đề: với **truy vấn có tham số** (parameterized queries),
optimizer THIẾU thông tin tại thời điểm biên dịch (lack of knowledge at compilation time) — ví dụ
không biết trước giá trị tham số sẽ truyền vào lúc chạy thật, nên không chọn chắc chắn được 1 kế hoạch
tốt nhất. Giải pháp: **kết hợp nhiều phương án khác nhau** vào CÙNG 1 kế hoạch, dùng 1 toán tử đặc
biệt gọi là **Choose-Plan** đặt ở gốc cây — toán tử này CHỈ quyết định chọn phương án nào (vd
HashJoin+FileScan hay HashJoin+B-tree Scan) ngay tại THỜI ĐIỂM BẮT ĐẦU chạy truy vấn thật (khi đã biết
giá trị tham số cụ thể), thay vì phải quyết định cứng từ lúc biên dịch.

---

## Trang PDF 16 — Slide 31-32: Tối ưu lại giữa chừng (mid-query re-optimization)

**Slide 31 — "Mid-query re-optimization".** Vấn đề: thống kê KHÔNG CHÍNH XÁC hoặc KHÔNG BIẾT TRƯỚC
(inaccurate or unknown statistics) — đúng hệ quả của slide 22 ("few statistics, if any" trong tích
hợp dữ liệu). Giải pháp: **thu thập thống kê NGAY GIỮA CHỪNG lúc truy vấn đang chạy** (collecting
statistics at mid-query), bằng cách chèn 1 toán tử đặc biệt gọi là **Statistics-Collector** vào cây kế
hoạch, có nhiệm vụ: thu thập thống kê về dữ liệu ĐANG chảy qua nó, VÀ vật lý hoá (materialize) kết quả
trung gian để dùng lại sau. Cây ví dụ trên slide: Aggregate(G) → Indexed-Join(F4) → Hash-Join(F3) →
Statistic-Collector(S) → Filter(F1^F2) → R — Statistic-Collector được đặt NGAY SAU bước Filter, trước
khi dữ liệu đi vào Hash-Join, để "đo" đặc tính thật của dữ liệu đã lọc trước khi quyết định cách join
tiếp theo.

**Slide 32 — "Statistic collection".** Vẽ lại CHI TIẾT HƠN quy trình ở slide 31: so sánh cây kế hoạch
GỐC (trái, có Statistic Collector thu thập Histogram trên `R.joinattr3` và Unique values trên
`R.groupattr`) với cây kế hoạch MỚI (phải) — cho thấy rõ Statistic Collector được giữ nguyên vị trí
nhưng thông tin nó thu thập được (histogram, số giá trị duy nhất) sẽ được dùng để quyết định LẠI các
bước optimize phía trên nó (Hash-Join F3, Indexed-Join F4, Aggregate G) — minh hoạ bằng việc 2 cây gần
như giống hệt nhau về CẤU TRÚC, chỉ khác ở việc cây bên phải đã "học" được thống kê thật.

---

## Trang PDF 17 — Slide 33-34: Vật lý hoá kết quả trung gian & Query Scrambling

**Slide 33 — "Materialization".** Tiếp nối trực tiếp slide 31-32: sau khi Statistic-Collector đã thu
thập xong thống kê thật (Histogram trên `Rel1.joinattr3`, Unique values trên `Rel.groupattr`), bước
kế tiếp là **vật lý hoá (materialize)** kết quả trung gian của nhánh Hash-Join(F3) thành 1 bảng tạm
gọi là `Temp1`, rồi dùng LẠI `Temp1` này (đã biết chính xác kích thước/thống kê thật, không còn là ước
lượng) để tối ưu lại các bước join/aggregate PHÍA TRÊN nó (vd đổi Indexed-Join thành Hash-Join với
`T`, dựa trên câu lệnh SQL tương đương được vẽ ở giữa slide: `SELECT avg(...) FROM Temp1, T WHERE
Temp1.joinattr3=T.joinattr3 GROUP BY Temp1.groupattr`). Đây là ví dụ HOÀN CHỈNH của "mid-query
re-optimization" đã học: thu thập thống kê → vật lý hoá → dùng thống kê THẬT để chọn lại kế hoạch tốt
hơn cho phần CÒN LẠI của truy vấn.

**Slide 34 — "Query scrambling".** Chuyển sang mức độ thích nghi THỨ 2 đã nêu ở slide 23: **thay đổi
thứ tự toán tử** (operators order). Vấn đề: **độ trễ mạng** (network delays) — 1 hoặc vài nguồn dữ
liệu phản hồi chậm bất thường. Giải pháp: **"giấu" các nguồn bị chậm** và **chạy trước các toán tử
không bị chặn** (non-blocked operators). Sơ đồ minh hoạ: cây kế hoạch có 6 toán tử join/lọc (đánh số
1-6) trên 5 nguồn R1-R5; nguồn R1 có biểu tượng tia sét (⚡) nghĩa là ĐANG BỊ CHẬM/TRỄ. 2 vùng được
khoanh tròn nét đứt: vùng {toán tử 4, join R4⋈R5} và vùng {toán tử 3,5,6 cùng R3} — đây là 2 "nhánh
độc lập" KHÔNG phụ thuộc vào nguồn R1 đang chậm, nên có thể XỬ LÝ TRƯỚC trong khi chờ R1. Quy trình
gồm **2 pha**: rescheduling (lên lịch lại) và operators synthesis (tổng hợp toán tử) — chi tiết ở
slide 35-36.

---

## Trang PDF 18 — Slide 35-36: Query Scrambling — 2 pha chi tiết

**Slide 35 — "Phase 1: Rescheduling".** Nguyên lý: chạy các "nhánh con KHÔNG bị chặn" (non-blocked
sub-tree) — tức các nhánh không phụ thuộc vào nguồn đang delay hoặc toán tử đang bị block. Quy trình:
xử lý 1 nhánh con không bị chặn → vật lý hoá kết quả trung gian của nhánh đó → nếu nguồn bị delay đã
quay lại (return), tiếp tục xử lý bình thường; nếu chưa, chuyển sang xử lý nhánh con không bị chặn
TIẾP THEO; nếu không còn nhánh nào để xử lý, chuyển sang **Phase 2**. Sơ đồ trước/sau cho thấy cây kế
hoạch được "tháo rời" thành các nhánh con đã xử lý xong (ký hiệu dấu nháy `'`, `''` trên các toán tử)
chờ ghép lại.

**Slide 36 — "Phase 2: Operator synthesis".** Nguyên lý: KẾT HỢP các dữ liệu đã vật lý hoá ở Phase 1
bằng các toán tử JOIN MỚI được sinh ra (generating new join operators for materialized intermediate
result). Quy trình: sinh toán tử join mới cho kết quả trung gian đã vật lý hoá → nếu vẫn còn delay,
tiếp tục chờ; nếu không, sinh thêm toán tử khác cho dữ liệu vật lý hoá mới; nếu không còn việc gì làm,
chờ dữ liệu (waiting data). Nhãn "Reordering join" trên sơ đồ cho thấy thứ tự join CUỐI CÙNG đã bị
THAY ĐỔI so với kế hoạch gốc ban đầu — đúng ý nghĩa "operators order" (mức độ thích nghi thứ 2) đã nêu
ở slide 23.

---

## Trang PDF 19 — Slide 37-38: Toán tử Eddy — định tuyến từng tuple

**Slide 37 — "Eddy [AH00]"** (trích dẫn Avnur-Hellerstein 2000). Đây là mức độ thích nghi THỨ 3 và
"triệt để" nhất đã nêu ở slide 23 — không còn khái niệm "1 cây kế hoạch cố định" nữa. Vấn đề: chi phí,
độ chọn lọc, tốc độ dữ liệu DAO ĐỘNG liên tục lúc chạy (run-time fluctuations). Giải pháp: **định
tuyến từng tuple một** (per-tuple operator reordering) thông qua 1 thành phần trung tâm gọi là **Eddy**
— sơ đồ cho thấy dữ liệu từ 3 nguồn R, S, T đi VÀO Eddy (hình oval ở giữa), Eddy quyết định đẩy từng
tuple qua toán tử join nào trước (RS hay ST), rồi tuple tiếp tục quay LẠI Eddy để được định tuyến tiếp
cho tới khi ra kết quả cuối (RST). Dòng chữ nhấn mạnh trên slide: "**no query plan, but Eddy & modules**"
— nghĩa là KHÔNG CÒN khái niệm "kế hoạch truy vấn cố định" nữa, chỉ còn Eddy (bộ định tuyến) và các
"module" (toán tử) mà Eddy điều phối động theo từng tuple.

**Slide 38 — "Eddy: easily routing?".** Nêu 2 tính chất kỹ thuật BẮT BUỘC mà thuật toán toán tử phải
có để Eddy định tuyến được: **(1) Synchronization barrier** (rào cản đồng bộ) — tức thời điểm 1 quan
hệ đầu vào bị "đóng băng" chờ quan hệ kia; Eddy cần toán tử có rào cản đồng bộ **thích nghi hoặc không
tồn tại** (càng ít rào cản, Eddy càng định tuyến tự do). **(2) Moment of symmetry** (khoảnh khắc đối
xứng) — điểm cho phép ĐỔI THỨ TỰ 2 quan hệ mà không ảnh hưởng kết quả (đã nhắc ở RippleJoin slide
25-26); Eddy cần toán tử có **khoảnh khắc đối xứng xảy ra thường xuyên**. Về chính sách định tuyến:
nguyên lý chung là "ưu tiên sinh ra tuple kết quả" (favors production of result tuples), với 2 chính
sách cụ thể: **Naïve** và **Fast** (chi tiết ở slide 40-41).

---

## Trang PDF 20 — Slide 39-40: Cấu trúc tuple của Eddy & chính sách Naïve

**Slide 39 — "Eddy: tuple structure".** Mỗi tuple đi qua Eddy được gắn thêm 2 vector bit: **Ready
Bits** (toán tử nào SẴN SÀNG nhận tuple này — vd `[1,1]` nghĩa là cả op1 và op2 đều sẵn sàng) và
**Done Bits** (toán tử nào đã XỬ LÝ XONG tuple này — vd `[0,0]` nghĩa là chưa toán tử nào xử lý).
Việc định tuyến dựa HOÀN TOÀN vào giá trị của vector Ready — đây là cơ chế kỹ thuật cụ thể giúp Eddy
biết tuple nào cần đi đâu tiếp theo mà không cần duyệt lại toàn bộ kế hoạch.

**Slide 40 — "Naïve Eddy".** Ví dụ cụ thể: truy vấn `SELECT * FROM R WHERE s1() AND s2()` có 2 điều
kiện lọc độc lập. Theo kiểu tĩnh (static) truyền thống, phải CHỌN CỨNG thứ tự: hoặc chạy s1() trước
s2(), hoặc ngược lại — trong khi thống kê cho biết: cả 2 có độ chọn lọc 50%, nhưng **chi phí khác
nhau RÕ RỆT**: Cost(s2)=5 giây cố định, còn Cost(s1) dao động TỪ 1 ĐẾN 9 giây (không ổn định) — nếu
chọn cứng s1 trước mà s1 đang chạy chậm (9s), sẽ chậm hơn hẳn so với chạy s2 trước. **Chính sách định
tuyến Naïve Eddy**: mọi toán tử đều cố gắng lấy tuple từ Eddy NHANH NHẤT có thể (as fast as possible),
và tuple đã "được thấy" trước đó (previously-seen) được ưu tiên xử lý trước tuple mới — tức là Eddy để
THỊ TRƯỜNG TỰ NHIÊN (toán tử nào rảnh thì lấy tuple) tự quyết định thứ tự, thay vì áp đặt 1 thứ tự cố
định từ đầu — né được rủi ro "chọn nhầm" s1 trước khi s1 đang chậm.

---

## Trang PDF 21 — Slide 41-42: Fast Eddy & vấn đề xử lý truy vấn "không phù hợp"

**Slide 41 — "Fast Eddy".** Cải tiến của Naïve Eddy bằng cơ chế **"vé số" (lottery)** tương tự chính
sách Lottery Scheduling đã nêu ở slide 24: Vòng lặp — nhận 1 tuple → xác định toán tử nào ĐỦ ĐIỀU KIỆN
xử lý tuple này → nếu có HƠN 1 toán tử đủ điều kiện, tổ chức "bốc thăm" (lottery) với xác suất thắng
TỈ LỆ THUẬN với số vé (# tickets) mỗi toán tử đang có → toán tử ĐƯỢC CẤP THÊM 1 vé mỗi khi nó NHẬN
được 1 tuple, nhưng BỊ TRỪ 1 vé mỗi khi nó TRẢ LẠI (return) 1 tuple → lặp tới khi không còn toán tử
nào xử lý được tuple nữa. Ý nghĩa của cơ chế "vé": số vé của 1 toán tử phản ánh HIỆU QUẢ của nó trong
việc "loại bỏ" (giảm số lượng) tuple ra khỏi hệ thống — toán tử càng hiệu quả càng được ưu tiên nhận
thêm tuple. Ghi chú trên slide: "**hệ thống TỰ HỌC được thứ tự của các toán tử**" — không cần con người
can thiệp.

**Slide 42 — "What is 'inappropriate' in DB query processing?"**. Chuyển sang 1 vấn đề khác (không
còn thuộc Adaptive QP mà là bước đệm dẫn sang chủ đề "Partial Results"): cách CSDL truyền thống xử lý
truy vấn có 2 điểm "không phù hợp" với nhu cầu thực tế hiện đại — **(1) Truy vấn CSDL** chỉ nói "tôi
muốn (lấy) dữ liệu gì" nhưng (có thể) kết quả quá lớn và KHÔNG phù hợp để người dùng xem hết cùng lúc;
**(2) Xử lý theo lô (batch processing)**: người dùng phải CHỜ truy vấn CHẠY XONG HẲN mới thấy gì — có
thể rất lâu, thậm chí "interminable" (vô tận). Nhu cầu đặt ra: cần cho phép **client chủ động kiểm
soát** và hệ thống cần **trả kết quả từng phần (feedback)** liên tục thay vì đợi tới cuối — đây là động
lực cho chủ đề "partial results" ở slide 43-47.

---

## Trang PDF 22 — Slide 43-44: Trả kết quả từng phần & kỹ thuật Juggle

**Slide 43 — "Building partial results".** Vấn đề: dữ liệu KHÔNG sẵn có ngay (unavailable data — vd
nguồn chậm) VÀ người dùng có THIÊN KIẾN/ưu tiên riêng về dữ liệu nào muốn xem trước (client preference
on some data). Hướng giải quyết tổng quát: **trả về kết quả KHÔNG ĐẦY ĐỦ** (returning incomplete
results) thay vì đợi đầy đủ, và **định tuyến dữ liệu tới client theo đúng sở thích của họ** (routing
data to client according to client interest) — sơ đồ vẽ pipeline join R, S, T đơn giản để làm nền cho
các kỹ thuật cụ thể ở slide 44-47.

**Slide 44 — "Juggle".** Bối cảnh: **xử lý truy vấn trực tuyến (online query processing)** — người
dùng CẢM NHẬN được dữ liệu đang được xử lý DẦN THEO THỜI GIAN (users perceive data being processed
over time), và sở thích người dùng được biểu diễn thành "**user interests as preferences**" (danh
sách ưu tiên). Giải pháp Juggle: **ưu tiên xử lý các tuple "thú vị"** (interesting tuples) dựa trên sở
thích người dùng đã khai báo, bằng 2 kỹ thuật: **Prefetch & Spool (P&S)** và **reorder** (sắp xếp lại
thứ tự trả về) — sơ đồ cho thấy pipeline R→S→T có thêm bước "reorder" chèn vào trước khi trả kết quả
ra ngoài.

---

## Trang PDF 23 — Slide 45-46: Kỹ thuật Prefetch&Spool và JuggleEddy

**Slide 45 — "P&S technique"** (Prefetch & Spool — chi tiết kỹ thuật của Juggle ở slide 44). Nguyên
lý: **lấy trước** (prefetch) dữ liệu từ đầu vào, **đổ ra đĩa phụ** (spools onto auxiliary side disk)
nếu cần; "tung hứng" (juggle) dữ liệu giữa buffer (bộ nhớ đệm) và đĩa phụ để LUÔN giữ buffer đầy những
mục "thú vị" nhất — hàm `getNext` sẽ chọn mục TỐT NHẤT hiện có trong buffer để trả về. **Chính sách
sắp xếp lại (reordering policy)** quyết định 2 việc: `getNext` trả về gì, VÀ quyết định khi nào nên
làm giàu thêm (enrich)/đổ ra đĩa (spool) dữ liệu.

**Slide 46 — "JuggleEddy".** Kết hợp ý tưởng Eddy (định tuyến động theo tuple, slide 37-41) với Juggle
(ưu tiên theo sở thích người dùng, slide 44-45). Bối cảnh: dữ liệu không sẵn có (unavailable data),
sở thích người dùng có thể THAY ĐỔI ngay trong lúc đang chạy truy vấn (may be changed during QE —
Query Execution), và ứng dụng khách là dạng **hiển thị kiểu bảng tính** (spreadsheet display client
application — người dùng cuộn xem từng phần bảng kết quả). Giải pháp: liên tục trả kết quả về client
(return continually results to client), định tuyến tuple dựa trên **băm theo cột khoá do client định
nghĩa** (hash client-defining key columns) — nghĩa là việc định tuyến không chỉ dựa trên hiệu năng hệ
thống (như Eddy gốc) mà còn dựa trên PHẦN BẢNG mà người dùng đang thực sự nhìn vào.

---

## Trang PDF 24 — Slide 47-48: Định nghĩa Partial Result & chuyển sang Approximate QP

**Slide 47 — "Partial results in JuggleEddy".** Đưa ra các ĐỊNH NGHĨA chính thức cho khái niệm "kết
quả từng phần": **partial result** (kết quả từng phần) ~ bất kỳ tuple nào được sinh ra bởi Query
Processing; **intermediate result** (kết quả trung gian) ~ kết quả CHƯA ĐẦY ĐỦ đang hiển thị ở client;
**key column** (cột khoá) ~ tập cột đủ để xác định DUY NHẤT 1 tuple kết quả; **column properties
(CPs)**, **row properties (RPs)**, **row-column properties (RCPs)** — các thuộc tính mô tả "độ quan
trọng" theo cột, theo dòng, theo từng ô cụ thể (⚠️ slide không định nghĩa chi tiết CP/RP/RCP là gì,
chỉ liệt kê tên — cần xem thêm tài liệu gốc nếu cần dùng chính xác). Công thức tính **độ lợi (benefit)**
của 1 ô kết quả: `Benefit(cell[r,c]) = RP(c)·CP(c)·RCP(r,c)` — nhân 3 yếu tố độ quan trọng lại với
nhau để ra 1 điểm số duy nhất, dùng để quyết định ô nào nên được "juggle" ưu tiên trả về trước. Sơ đồ
bên phải cho thấy pipeline hoàn chỉnh: nguồn R,S,T → các join RS, ST → JuggleEddy → ra bảng kết quả có
các cột a1,a2,…,name,a4,a5 (CP = Column Property gắn ở đầu bảng).

**Slide 48 — "Approximate Query Processing"** (slide tiêu đề, chuyển chủ đề). Đây là slide TIÊU ĐỀ
CHUYỂN PHẦN — đánh dấu kết thúc toàn bộ phần "Adaptive Query Processing" (slide 9-47) và bắt đầu phần
thứ 2 của dàn ý đã nêu ở slide 9: **Approximate Query Processing (AQP)** — quay trở lại đúng ý tưởng
đã làm quen ở slide 4-6 (histogram, sampling, sketch) nhưng sẽ đi SÂU HƠN về mặt hệ thống/lý thuyết.

---

## Trang PDF 25 — Slide 49-50: Phân loại AQP & hệ thống AQP công nghiệp đầu tiên

**Slide 49 — "Taxonomy"** (bảng phân loại). Đây là 1 bảng 2 chiều: **hàng** phân theo LOẠI TRUY VẤN
(Static queries — truy vấn cố định biết trước; Predictive queries and data — truy vấn có yếu tố dự
đoán; Ad hoc queries — truy vấn tuỳ biến không biết trước); **cột** phân theo LOẠI PHÂN TÍCH (Simple
analytics; Complex analytics; Machine Learning). Mỗi ô liệt kê các kỹ thuật/cấu trúc dữ liệu AQP phù
hợp — ví dụ ô (Static queries × Simple analytics) gồm Heavy hitters, Max/min, Quantiles, Distinct
values, Frequency moments (đều là các bài toán "tổng hợp xấp xỉ" kinh điển), còn cột "Sketches"
(FM/AMS/LSH…) chính là kỹ thuật đã làm quen ở slide 6. 1 vài ô còn để dấu "**?**" (ví dụ Predictive×ML,
Ad hoc×ML) — cho thấy đây là các hướng NGHIÊN CỨU CHƯA CÓ lời giải chuẩn hoá, không phải slide thiếu
sót. Bảng này giúp định vị: các ví dụ đã học (histogram, sampling, sketch) chỉ là phần nhỏ trong 1 bức
tranh lớn hơn nhiều của AQP.

**Slide 50 — "Industrial Strength APQ Systems".** Giới thiệu hệ thống AQP thực tế đầu tiên: **Verdict**
(nguồn: Mozafari 2017, verdictdb.org). Thách thức triển khai thực tế: cần **middleware tương thích với
engine CSDL đã có sẵn** (không thay thế toàn bộ hệ thống), đòi hỏi **viết lại truy vấn tự động**
(automatic query rewrite). Kiến trúc Verdict (sơ đồ): `Application/User` gửi **Original Query** vào
`Verdict` → Verdict sinh ra **New Query(s)** gửi xuống `Database` → Database trả về **Raw results** →
Verdict xử lý và trả lại cho người dùng **Approximate Answers & Error Estimation** (kết quả xấp xỉ KÈM
ước lượng sai số — đây là điểm khác biệt quan trọng so với chỉ trả 1 con số xấp xỉ suông). Vấn đề còn
lại nêu ở cuối slide: cần tương thích với GIAO DIỆN hiện có (compatibility and user friendliness), và
cần "**hợp đồng độ chính xác ở mức cao**" (high-level accuracy contracts) — ví dụ cam kết kiểu "chính
xác ít nhất p% với xác suất p%, và tồn tại với xác suất p%" (ký hiệu khá trừu tượng trên slide, ⚠️ nên
xem thêm tài liệu gốc Mozafari 2017 nếu cần hiểu chính xác công thức cam kết sai số này).

---

## Trang PDF 26 — Slide 51-52: Hỗ trợ công nghiệp thực tế & lập kế hoạch truy vấn AQP

**Slide 51 — "Industrial Supports".** Bảng so sánh các hệ thống AQP thực tế đã triển khai cho các
workload OLAP khác nhau (nguồn: Mozafari 2017): **ABM** (dùng cho TPC-H, hỗ trợ 68% truy vấn, tăng tốc
10x), **QuickR** (TPC-DS, >90% truy vấn được hỗ trợ, tăng tốc 2x — ngoại trừ full outer join), **BlinkDB**
(Facebook, >96% truy vấn, tăng tốc không xác định "?" — ngoại trừ join nhiều fact table), hệ thống kết
hợp [1]+[3] (Conviva Inc., 91% truy vấn, tăng tốc 10-200x), và **Verdict** (khách hàng tuỳ chỉnh, 74%
truy vấn, tăng tốc 2-20x — hạn chế với join nhiều fact table, truy vấn lồng nhau, lọc văn bản). Kết
luận cuối slide: "**So far: relatively simple SQL queries**" — tức CÁC HỆ THỐNG THỰC TẾ HIỆN NAY mới
chỉ xử lý tốt truy vấn SQL tương đối ĐƠN GIẢN, chưa giải quyết triệt để các truy vấn phức tạp (sẽ bàn
ở slide 53).

**Slide 52 — "Query planning".** Bàn về sự khác biệt giữa lập kế hoạch truy vấn AQP so với tối ưu hoá
truyền thống (slide 10-21): tiêu chí chọn kế hoạch KHÁC HẲN — thay vì tối thiểu hoá tổng thời gian,
AQP cần **tối thiểu hoá thời gian đạt được sai số CHẤP NHẬN ĐƯỢC, hoặc giữ sai số trong giới hạn thời
gian cho phép**; đồng thời "**sai số RẤT KHÓ dự đoán và kiểm soát**" — các hướng tiếp cận hiện có: công
thức giải tích (analytical formulas), mô hình Bayesian, bootstrap kiểu giải tích/Poisson, hoặc đảm bảo
sai số TIÊN NGHIỆM (a priori) bằng kỹ thuật sample+seek kết hợp lấy mẫu có trọng số đo lường (measure-
biased sampling) và chỉ mục. Thêm 1 khó khăn: **độ trễ (latency) RẤT khó dự đoán**, đặc biệt trong môi
trường song song/phân tán. Hướng giải quyết nêu ra: tự động chọn "synopsis" (bản tóm tắt dữ liệu, vd
sketch/sample) phù hợp bằng cách CHẠY SONG SONG nhiều synopsis cạnh tranh rồi kết hợp kết quả (run a
competing set of synopses and combine answers), có cơ sở lý thuyết (vd phân tích độ phức tạp không
gian — [Kaushik et al. 2005]), và dùng **học máy dựa trên kết quả trước đó + khám phá thêm**
(learning based on prior results + exploration) để mở rộng sang dữ liệu biến động (dynamic data) —
biểu đồ nhỏ bên phải minh hoạ: sai số (error) GIẢM DẦN theo thời gian (time), và có 1 ngưỡng sai số
mục tiêu ε cần đạt được trước mốc thời gian t.

---

## Trang PDF 27 — Slide 53-54: Phân tích phức tạp & khung kiến trúc AQP thích nghi

**Slide 53 — "Handling Complex analytics".** Bàn tiếp vấn đề còn bỏ ngỏ ở slide 51 ("so far: relatively
simple SQL queries") — làm sao mở rộng AQP sang truy vấn PHỨC TẠP hơn: **truy vấn tổng hợp SQL bất kỳ**
(arbitrary SQL aggregate queries, gồm cả truy vấn con — subqueries; kỹ thuật Quickr — chèn "distinct-
samplers" vào kế hoạch truy vấn, chạy nhiều lượt/multiple passes — xem sơ đồ nhỏ bên phải: Query →
ASALQA (thuật toán xấp xỉ) → Plans with samplers, dựa trên Stats thu thập offline/online từ Data);
**truy vấn trả về tập giá trị** (set-valued queries); **truy vấn hiện đại** (graph queries; Machine
Learning — coreSets, model management, sampleClean); và 2 vấn đề lý thuyết còn mở: **lan truyền sai số
qua chuỗi phép toán phân tích** (sequences of analytical operations: error propagation?) và **ước
lượng/đảm bảo sai số NGAY CẢ trong tình huống "đơn giản"** (SPJ+Agg — Select/Project/Join + Aggregate
— có GROUP BY và điều kiện lọc). Nguồn: Kandula et al. 2016.

**Slide 54 — "Adaptive Query Processing Framework"** (sơ đồ tổng kết, GẦN NHƯ lặp lại sơ đồ mở đầu ở
slide 2 nhưng bổ sung thêm chi tiết của TOÀN BỘ bài giảng). Query → Parse → (sinh "internal
representation of query") → Optimization (có nhãn "re-ordering joins" — đúng kỹ thuật query scrambling
đã học ở slide 34-36) → sinh Execution plan → Execution Engine (chạy join trên dữ liệu R, S, T thật,
có mốc Timeout) → Data. Bên phải sơ đồ là khối **Monitoring** (giám sát — trả lời câu hỏi "What? How
often?" — giám sát CÁI GÌ và TẦN SUẤT bao nhiêu) kết nối 2 chiều với khối **Reaction** (phản ứng — trả
lời câu hỏi "When? How?" — PHẢN ỨNG KHI NÀO và BẰNG CÁCH NÀO) — cặp Monitoring/Reaction này chính là
hiện thực hoá đúng định nghĩa "vòng phản hồi" (feedback loop) đã nêu ở slide 23, áp dụng xuyên suốt cả
3 mức thích nghi (toán tử, thứ tự, cấu trúc kế hoạch) đã học trong toàn bộ phần Adaptive QP.

---

## Trang PDF 28 — Slide 55-56: Tổng kết & tài liệu tham khảo

**Slide 55 — "Query Processing Summary".** Tổng kết 4 ý chính của toàn bộ bài giảng: **(1)** trong
tích hợp dữ liệu, KHÔNG có sẵn thống kê như thông thường (we don't have the usual statistics) → do đó
PHẢI thích nghi (hence, need to be adaptive) — đúng mạch logic xuyên suốt từ slide 22 tới hết phần
Adaptive QP; **(2)** tính thích nghi đòi hỏi PHẢI ĐÁNH ĐỔI (trade off) giữa nhiều yếu tố (chi phí giám
sát vs. lợi ích thích nghi, độ trễ vs. độ chính xác kế hoạch…); **(3)** xét về mặt LÝ THUYẾT, đây vẫn
còn là **bài toán MỞ** (open problem) — chưa có lời giải tối ưu tổng quát; **(4)** câu hỏi mở rộng
thêm: "còn nếu CHẤP NHẬN kết quả xấp xỉ thì sao?" — câu hỏi này chính là cầu nối dẫn sang toàn bộ phần
Approximate Query Processing (slide 48-54) đã học.

**Slide 56 — "References"** (danh mục tài liệu tham khảo gốc của toàn bộ slide — liệt kê nguyên văn,
không diễn giải thêm vì đây là trích dẫn học thuật cần giữ chính xác tuyệt đối):
- Hellerstein, Franklin, Chandrasekaran, Deshpande, Hildrum, Madden, Raman, Shah. *Adaptive Query
  Processing: Technology in Evolution*. IEEE Data Engineering Bulletin 23(2), 2000. (nguồn gốc định
  nghĩa Adaptive QP ở slide 23, và Eddy ở slide 37-41, ký hiệu [HFC+00]/[AH00] trên các slide trước)
- Goetz Graefe, Karen Ward. *Dynamic query evaluation plans*. ACM SIGMOD 1989. (nguồn Dynamic query
  plan — slide 30)
- Navin Kabra, David J. DeWitt. *Efficient Mid-Query Re-Optimization of Sub-Optimal Query Execution
  Plans*. ACM SIGMOD 1998. (nguồn Mid-query re-optimization — slide 31-33)
- Tolga Urhan, Michael Franklin. *XJoin: A Reactively-Scheduled Pipelined Join Operator*. IEEE Data
  Engineering Bulletin, 2000. (nguồn XJoin — slide 29)
- Peter J. Haas, Joseph M. Hellerstein. *Ripple Joins for Online Aggregation*. SIGMOD 1999. (nguồn
  RippleJoin — slide 25-26)
- Mozafari. *Approximate query engines: Commercial challenges and research opportunities*. SIGMOD 2017.
  (nguồn bảng Industrial Supports — slide 51)
- Mozafari. *Verdict: A system for stochastic query planning*. CIDR 2015. (nguồn hệ thống Verdict —
  slide 50)
- Kandula et al. *Quickr: Lazily approximating complex ad hoc queries in bigData clusters*. SIGMOD 2016.
  (nguồn Quickr — slide 53)
