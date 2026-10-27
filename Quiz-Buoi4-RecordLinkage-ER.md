# Quiz ôn tập — Buổi 4 (29/9): Record Linkage & Entity Resolution

Dựa trên `3_RecordLinkage_ER.pdf` (46 slide). 20 câu, chia 4 phần. Làm hết phần câu hỏi trước, đáp án ở cuối file — đừng kéo xuống xem trước khi tự làm.

---

## Phần A — Khái niệm nền tảng

**A1.** Theo slide "Terminology", *Entity Resolution* là một trường hợp đặc biệt của khái niệm rộng hơn nào? Khái niệm đó được định nghĩa ra sao?

**A2.** Phân biệt "Entity Resolution" (khử trùng lặp) với "Entity Linking" nói chung — cho 1 ví dụ Entity Linking KHÔNG phải là khử trùng lặp giữa 2 file dữ liệu có cấu trúc.

**A3.** Phương pháp "Naïve Deduplication" (UNION DISTINCT qua hash/group-by) có nhược điểm cốt lõi gì khiến nó không đủ dùng trong thực tế?

**A4.** Theo định nghĩa của Fellegi & Sunter (1969), khi so khớp một cặp bản ghi $a_i - b_j$, có 3 khả năng kết luận. Đó là 3 khả năng nào, và "possible-link" nghĩa là gì?

**A5.** Similarity measure $s(x,y)$ và distance/cost measure khác nhau như thế nào về hướng diễn giải giá trị (giá trị cao/thấp thì tương ứng với điều gì)?

**A6.** Vì sao "áp dụng $s(x,y)$ cho tất cả các cặp" bị coi là *impractical* (không khả thi) ở quy mô lớn? Nêu độ phức tạp cụ thể.

---

## Phần B — Similarity Measures (có tính toán)

**B1.** Tính Edit Distance (Levenshtein) giữa `"color"` và `"colour"`. Liệt kê thao tác cần dùng.

**B2.** Từ kết quả B1, tính similarity score bằng công thức $s(x,y) = 1 - \dfrac{d(x,y)}{\max(|x|,|y|)}$.

**B3.** Cho $B_x = \{a, b, c, d\}$ và $B_y = \{b, c, d, e\}$ (tập token của 2 chuỗi x, y). Tính:
  a) Overlap measure $O(x,y)$
  b) Jaccard measure $J(x,y)$

**B4.** Trong TF/IDF measure, tại sao khi so khớp `"Apple Corporation, CA"` với `"IBM Corporation, CA"` và `"Apple Corp"`, phương pháp Edit Distance/Jaccard thuần dễ cho kết quả **sai** (khớp nhầm với IBM), còn TF/IDF lại đúng?

**B5.** Áp dụng đúng 4 bước thuật toán Soundex trong slide để tính mã Soundex của tên **"Williams"**. Trình bày từng bước (giữ chữ đầu → loại bỏ W/H → mã hoá chữ cái còn lại thành số → gộp số liên tiếp giống nhau → bỏ chữ không phải số, lấy 4 ký tự đầu).

**B6.** Generalized Jaccard Measure khác Jaccard measure thông thường ở điểm nào? Nó giải quyết vấn đề gì mà Jaccard chuẩn gặp phải (gợi ý: ví dụ "energy" vs "eneryg" trong slide)?

---

## Phần C — Entity Resolution Pipeline & thuật toán

**C1.** Entity Resolution Pipeline gồm 4 bước theo đúng thứ tự. Hãy sắp xếp lại 4 bước sau cho đúng: *Matching, Prepare Data, Clustering, Blocking/Sorting*.

**C2.** Vì sao cần bước Blocking/Sorting trước khi Matching, thay vì so khớp trực tiếp toàn bộ Naïve All-Pairs?

**C3.** Nêu 2 kỹ thuật Blocking đã học trong slide (ngoài Naïve All-Pairs) và điểm khác biệt cơ bản giữa chúng.

**C4.** Trong bước Matching, "Learned Matchers" (cả ML truyền thống lẫn Deep Learning) gặp trở ngại lớn nhất nào về dữ liệu huấn luyện? Nêu 2 hướng giải pháp được nhắc tới trong slide.

**C5.** Phương pháp Clustering "Connected Components / Transitive Closure" (dùng ngưỡng similarity trên cạnh của đồ thị) có nhược điểm gì đã được slide chỉ ra rõ?

**C6.** Incremental Data Deduplication giải quyết vấn đề gì so với việc chạy lại toàn bộ pipeline (batch) mỗi khi có dữ liệu mới? Nêu ngắn gọn ý tưởng "greedy update".

---

## Phần D — Liên hệ với đề tài Project của nhóm bạn

**D1.** Trong project khí tượng của nhóm bạn (tích hợp ERA5 lưới + trạm NOAA điểm), bước nào trong Entity Resolution Pipeline tương ứng với việc "gán một trạm quan trắc vào đúng ô lưới ERA5 gần nhất"? Vì sao bước này ở đây **không thể** chỉ dùng similarity measure thông thường (Jaccard/Edit Distance) như với text, mà cần thêm kỹ thuật gì?

**D2.** Giả sử batdongsan.com.vn và nhatot.com cùng đăng một căn hộ, nhưng giá lệch nhau 5% (một bên tính thêm phí môi giới). Bạn sẽ chọn loại similarity measure nào cho trường "giá" khi làm Record Linkage giữa 2 nguồn này, và vì sao **không nên** dùng so khớp chính xác (exact match)?

---
---

# ĐÁP ÁN

## Phần A

**A1.** Entity Resolution là trường hợp đặc biệt của **Entity Linking**. Entity Linking = "bài toán tạo các liên kết giữa các bản ghi đại diện cho các thực thể thế giới thực có liên quan theo một cách nào đó" (Burdick et al. 2019). Entity Resolution là trường hợp đặc biệt khi liên kết đó là "cùng một thực thể" (identifying/linking duplicate entities).

**A2.** Entity Resolution = khử trùng lặp / liên kết các bản ghi trỏ tới cùng 1 thực thể (Record Linkage, Data Deduplication). Entity Linking rộng hơn — ví dụ Named Entity Recognition & Disambiguation (nhận diện "Barack Obama" trong văn bản và liên kết nó với đúng thực thể trong knowledge base) không phải là khử trùng lặp giữa 2 file có cấu trúc, mà là liên kết văn bản tự do với một thực thể đã biết.

**A3.** Naïve Deduplication chỉ tìm được **exact match** (trùng khớp tuyệt đối) — không xử lý được các biến thể như lỗi gõ, viết tắt, khác định dạng (ví dụ "Matthias Boehm" vs "Matthias Böhm" trong ví dụ slide).

**A4.** Ba khả năng: **match (link)**, **no match (non-link)**, và **not enough evidence (possible-link)**. "Possible-link" nghĩa là dựa trên bằng chứng hiện có, hệ thống không đủ tin cậy để khẳng định chắc chắn là match hay non-match — cần thêm thông tin hoặc con người xác nhận.

**A5.** Similarity measure: **giá trị càng cao → càng giống nhau** (thang [0,1]). Distance/cost measure: cùng khái niệm nhưng ngược chiều — **giá trị càng nhỏ → càng giống nhau**.

**A6.** Vì so khớp tất cả các cặp có độ phức tạp **bậc hai (quadratic) theo kích thước dữ liệu** — với $n$ bản ghi mỗi tập, số cặp cần xét là $O(n^2)$ (thực chất $O(|X|\times|Y|)$), không khả thi khi $n$ lớn.

## Phần B

**B1.** `color` (5 ký tự) → `colour` (6 ký tự): chỉ cần **1 thao tác** — chèn (insert) ký tự `u` vào giữa `colo` và `r` (color → colo + u + r = colour). **d(x,y) = 1**.

**B2.** $s(x,y) = 1 - \dfrac{1}{\max(5,6)} = 1 - \dfrac{1}{6} = \dfrac{5}{6} \approx 0.833$.

**B3.**
a) $O(x,y) = |B_x \cap B_y| = |\{b,c,d\}| = 3$
b) $J(x,y) = \dfrac{|B_x \cap B_y|}{|B_x \cup B_y|} = \dfrac{3}{|\{a,b,c,d,e\}|} = \dfrac{3}{5} = 0.6$

**B4.** Vì "Corporation" và "CA" là các **term phổ biến** (xuất hiện trong nhiều chuỗi trong collection) → **IDF thấp**, trong khi "Apple" là **term hiếm/phân biệt** → **IDF cao**. Edit Distance/Jaccard thuần chỉ đếm số ký tự/token trùng mà không phân biệt được "Apple" quan trọng hơn "Corporation, CA" — nên bị "Corporation, CA" (dài, trùng nhiều) làm lệch kết quả, khớp nhầm x với y (cùng "IBM") thay vì x với z (cùng "Apple"). TF/IDF nhân trọng số theo độ hiếm của từ nên nhận diện đúng "Apple" là từ quyết định.

**B5.** `Williams`:
- Bước 1: giữ chữ đầu **W**, xử lý phần còn lại `illiams`
- Bước 2: không có W/H trong phần còn lại; mã hoá: i(bỏ qua, nguyên âm), l→4, l→4, i(bỏ qua), a(bỏ qua), m→5, s→2 → chuỗi tạm: `W i 4 4 i a 5 2`
- Bước 3: gộp số liên tiếp giống nhau: hai số `4,4` liền kề (từ l,l) gộp thành `4` → `W i 4 i a 5 2`
- Bước 4: bỏ hết ký tự không phải số (i, i, a), lấy chữ cái đầu + 3 chữ số đầu tiên: **W452**

*(Đối chiếu: đây đúng bằng giá trị Soundex chuẩn thực tế của "Williams" = W452.)*

**B6.** Jaccard chuẩn chỉ tính token **trùng tuyệt đối** vào tập overlap — nếu 1 token bị viết sai chính tả nhẹ (VD "energy" vs "eneryg" — đảo 2 ký tự cuối) thì bị coi là *không* overlap dù về bản chất là cùng 1 từ, làm Jaccard tụt thấp sai lệch. Generalized Jaccard "nới lỏng" điều kiện: dùng một similarity measure $s$ để tìm các cặp token *đủ giống* (score ≥ ngưỡng $\alpha$), tạo đồ thị hai phía (bipartite), rồi tìm **maximum-weight matching** trên đồ thị đó để tính overlap "mềm" — nhờ vậy "energy" vẫn được ghép với "eneryg".

## Phần C

**C1.** Thứ tự đúng: **Prepare Data → Blocking/Sorting → Matching → Clustering**.

**C2.** Vì Naïve All-Pairs có độ phức tạp $O(n^2)$ — với $n$ lớn (big data) số cặp cần so khớp bằng similarity measure (vốn đã tốn chi phí tính toán) sẽ bùng nổ, không khả thi. Blocking/Sorting giúp **giảm số cặp ứng viên** cần đưa vào bước Matching, chỉ giữ lại các cặp "có khả năng match cao" (VD cùng blocking key), do đó biến bài toán $O(n^2)$ thành khả thi ở quy mô lớn.

**C3.** Hai kỹ thuật:
- **Sorted Neighborhood**: định nghĩa sorting key, sắp xếp toàn bộ bản ghi theo key đó, rồi chỉ so khớp các bản ghi nằm trong một **cửa sổ trượt (sliding window)** kích thước $m$ (ví dụ 100) — giả định các bản ghi trùng nhau sẽ nằm gần nhau sau khi sort.
- **Blocking via Word Embeddings + LSH (Locality-Sensitive Hashing)**: tính embedding cho từ/thuộc tính/tuple, rồi dùng $K$ hash function tạo $k$-dimensional hash-code và $L$ hash table để nhóm các bản ghi có embedding gần nhau vào cùng bucket (không cần định nghĩa blocking key thủ công như Sorted Neighborhood).

**C4.** Trở ngại lớn nhất: **dữ liệu gán nhãn (labeled data) khan hiếm** (cần chuyên gia gán nhãn) và bị **class skew** (số cặp match thường ít hơn rất nhiều so với số cặp non-match). Hai hướng giải pháp: **Transfer Learning** (học mô hình từ kịch bản ER có nhiều tài nguyên, sau đó fine-tune bằng ít nhãn ở kịch bản mới — low-resource) và **Active Learning** (chủ động chọn ra những cặp nào cần gán nhãn để tối thiểu hoá công sức labeling mà vẫn cải thiện mô hình tốt nhất).

**C5.** Nhược điểm: dễ tạo ra **"big clusters and dissimilar records"** — tức các cụm quá lớn, trong đó chứa các bản ghi thực chất không giống nhau (do tính chất bắc cầu: nếu A~B và B~C nhưng A không thực sự giống C, transitive closure vẫn gộp cả A,B,C vào 1 cụm chỉ vì có "cầu nối" B).

**C6.** Batch recomputation (chạy lại toàn bộ pipeline mỗi khi có bản ghi mới) rất tốn kém khi luồng cập nhật liên tục — kết quả tính trước đó (previously computed results) trở nên lạc hậu (obsolete) nhưng việc tính lại từ đầu quá chậm. Incremental Data Deduplication duy trì **similarity graph** và **graph clustering** theo kiểu tăng dần: khi có $\Delta G$ (thay đổi), dùng **greedy update trong thời gian đa thức** — trực tiếp nối các thành phần liên thông mới vào hàng đợi $Q$, sau đó xét xem có nên **merge** (gộp 2 cụm), **split** (tách 1 cụm) hay **move** (chuyển 1 node giữa 2 cụm) để cải thiện kết quả, thay vì tính lại toàn bộ correlation clustering ban đầu — cho kết quả tương đương nhưng **nhanh hơn đáng kể** so với batch.

## Phần D

**D1.** Bước tương ứng là **Matching** (so khớp trạm điểm với ô lưới). Tuy nhiên đây **không thể** dùng trực tiếp similarity measure kiểu text (Jaccard/Edit Distance đo độ giống chuỗi) vì bài toán ở đây là **khoảng cách không gian địa lý** giữa 1 toạ độ điểm và 1 ô lưới có độ phủ ~28km — hai bên không "giống nhau về ký tự" mà cần xác định "trạm này thuộc/gần ô lưới nào". Vì vậy cần thêm kỹ thuật **Spatial Interpolation** (Kriging hoặc Bilinear Interpolation) để tính giá trị tương ứng tại vị trí trạm dựa trên các ô lưới lân cận, trước khi có thể "join"/liên kết 2 bên — đây chính là lý do đề tài Project gọi bước này là "Schema Alignment kèm nội suy không gian" thay vì Record Linkage thuần túy.

**D2.** Nên dùng **similarity/threshold-based measure cho giá trị số** (ví dụ: coi 2 giá match nếu $|price_1 - price_2| / \max(price_1, price_2) \le$ một ngưỡng, chẳng hạn 10%), **không nên dùng exact match** vì với dữ liệu thực tế giá gần như luôn lệch nhau chút ít giữa các nguồn (phí môi giới, làm tròn, thời điểm cập nhật khác nhau) — exact match sẽ **bỏ sót phần lớn các cặp tin trùng thực sự** (recall thấp), giống hệt vấn đề mà Naïve Deduplication gặp phải ở câu A3.
