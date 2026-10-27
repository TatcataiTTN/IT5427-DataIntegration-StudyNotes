# Giải thích chi tiết — `2_mediationquery.pdf` (Generating Mediation Queries)

Nguồn: `slides/2_mediationquery.pdf` — 7 trang in (mỗi trang in 2 slide gốc), tổng cộng **14 slide gốc**
(đánh số 1-14 ở góc dưới-phải mỗi khung). Tác giả: Vũ Tuyết Trinh, SoICT-HUST. Bài giảng này là bước
**SAU** khi đã có kết quả Schema Matching/GAV-LAV-GLAV (từ `2_SchemaAlignment.pdf`) — tức giả định ta
ĐÃ BIẾT ánh xạ ngữ nghĩa giữa lược đồ trung gian và các nguồn, và giờ cần sinh ra **câu truy vấn trung
gian (mediation query)** thật sự chạy được để lấy dữ liệu từ các nguồn đó.

Dưới đây giải thích từng slide theo đúng thứ tự gốc (nôm na → kỹ thuật → công thức/ví dụ).

---

## Slide 1 — Trang bìa: "Generating mediation queries"

Slide tiêu đề, không có nội dung kỹ thuật. Chủ đề của bài: làm sao từ ánh xạ lược đồ (đã học ở
`2_SchemaAlignment.pdf`) **tự động/bán tự động sinh ra** câu truy vấn trung gian dùng để trả lời truy
vấn của người dùng trên lược đồ trung gian.

---

## Slide 2 — "A Semi-automatic approach"

**Nôm na:** đây là ví dụ khởi động bằng dữ liệu QUAN HỆ (bảng) đơn giản trước khi đi vào dữ liệu
XML/bán cấu trúc phức tạp hơn ở các slide sau.

**Kỹ thuật:** Lược đồ trung gian là `Films(ID, Title, Date, Director_name, Country)`. Có 2 nguồn dữ
liệu: `Films(ID, Title, Date, Director_ID)` (chỉ có khoá ngoại tới đạo diễn, không có tên/nước trực
tiếp) và 2 nguồn đạo diễn riêng biệt `Director1(ID, Name, Contry)`, `Director2(ID, Name, Country)`
(chú ý: slide gốc có lỗi chính tả "Contry" thay vì "Country" ở một vài chỗ — giữ nguyên để bạn nhận ra
khi đối chiếu slide thật, không phải tôi gõ nhầm).

Slide giới thiệu 3 view trung gian `T1(ID,Title,Date,Director_ID)`, `T2(ID,Name,Contry)`,
`T3(ID,Name,Contry)` — đây là các "mảnh" (fragment) lấy trực tiếp từ nguồn, dùng ký hiệu đồ hoạ
**⋈ (join)** và **∪ (union)** để biểu diễn cách ghép `T1` với `T2` HOẶC `T1` với `T3` (vì có 2 nguồn
đạo diễn khác nhau, cùng vai trò) rồi hợp (UNION) 2 kết quả lại — đúng ý tưởng: dữ liệu phim có thể
"vá" thông tin đạo diễn từ Director1 HOẶC Director2, kết quả cuối cùng là hợp của cả hai khả năng.

---

## Slide 3 — "A Semi-automatic approach (2)"

**Nôm na:** slide này viết TƯỜNG MINH ra 2 câu truy vấn trung gian ứng với sơ đồ ở Slide 2, rồi "mở"
(unfold) chúng xuống tận các bảng nguồn gốc.

**Kỹ thuật — 2 mediation query ứng với 2 lựa chọn nguồn đạo diễn:**

$$q_1: \pi_{ID,Title,Date,Name,Country}(T1 \Join T2)$$
$$q_2: \pi_{ID,Title,Date,Name,Country}(T1 \Join T3) \text{ (dòng này bị khung chữ nhật đè khuất một phần trên slide gốc, suy ra từ quy luật đối xứng với } q_1\text{)}$$

Sau đó slide **Rewriting** (viết lại) — đây chính là kỹ thuật **Query/View Unfolding** đã học trong
GAV (`Quiz-DI...` câu 11, 14): thay `T1` bằng chính định nghĩa của nó trên bảng nguồn `Films`, thay
`T2` bằng định nghĩa trên `Director1`:

$$q_1: \pi_{ID,Title,Date,Name,Country}\Big((\pi_{ID,Title,Date,Director\_ID}\ Films) \Join (\pi_{ID,Name,Country}\ Director1)\Big)$$

Đây chính là câu truy vấn THỰC THI ĐƯỢC cuối cùng — không còn tham chiếu tới view trung gian `T1,T2`
nữa, chỉ còn phép chiếu (π) và kết nối (⋈) trực tiếp trên 2 bảng nguồn thật.

---

## Slide 4 — "Extension for semi/un-structure data"

**Nôm na:** từ đây trở đi, bài chuyển sang bài toán khó hơn — lược đồ KHÔNG còn là bảng phẳng mà là
**cây** (dữ liệu bán cấu trúc kiểu XML), ví dụ kinh điển "thư viện sách" dùng xuyên suốt phần còn lại
của slide.

**Kỹ thuật:** Lược đồ trung gian (Mediation schema, biểu diễn bằng cây XML Schema) là:
`library → author(+) → {id, address, book(+) → {ISBN, title}}` (ký hiệu `+` nghĩa là node có thể lặp
nhiều lần — multi-value, ví dụ 1 author có nhiều book). 3 nguồn `S1`, `S2`, `S3` là 3 cây XML khác
nhau, mỗi cây chỉ chứa MỘT PHẦN thông tin của lược đồ trung gian (S1: library→book→{ISBN,title,
authorId}; S2: library→author→{id,address}; S3: library→{book→{ISBN,title}, author→{id,address}}).

Điểm mấu chốt của slide: 2 bong bóng thoại **"a mediation query"** (ứng với node `library/author` —
CÓ dữ liệu khớp nằm rải rác ở S1+S2 nên sinh truy vấn được) và **"No query"** (ứng với node
`library/author/book/title` nối sang 1 phần không khớp được — minh hoạ rằng KHÔNG PHẢI lúc nào ghép
nối cũng sinh ra truy vấn hợp lệ, cần kiểm tra điều kiện trước).

---

## Slide 5 — "Assumption"

**Nôm na:** đây là "dữ liệu đầu vào" mà thuật toán ở các slide sau giả định ĐÃ CÓ SẴN — tức kết quả
Schema Matching đã làm xong từ trước (không phải việc của bài này).

**Kỹ thuật:** cho trước **semantic correspondences** (tương ứng ngữ nghĩa) giữa đường dẫn (path) trên
lược đồ trung gian và đường dẫn trên từng nguồn, ví dụ:

$$SM/Library/Author/Id \;\cong\; S1/Library/book/AuthorId$$

(ký hiệu $\cong$ = "tương đương ngữ nghĩa", đúng khái niệm **semantic mapping** đã học ở Quiz DI câu
18). Slide liệt kê đầy đủ các tương ứng còn lại cho `address`, `book/ISBN`, `book/title`,
`chapter/Number`, `abstract` giữa lược đồ trung gian và 3 nguồn S1 (có thêm `chapter/Number[s:book]`,
`abstract` — thông tin về chương sách), S2 (chỉ có `author/{id,address}`), S3 (đầy đủ cả `book` lẫn
`author` nhưng tách biệt, không link với nhau qua `authorId`).

---

## Slide 6 — "Generating mediation queries" (nguyên lý tổng quát)

**Nôm na:** đây là "mục lục" của cả thuật toán — 3 bước lớn sẽ được từng slide sau giải thích kỹ hơn.

**Kỹ thuật — 3 bước (Principle):**
1. **Decomposing** — chia nhỏ lược đồ trung gian thành các cây con (sub-trees).
2. **Looking for partial mappings** — với mỗi cây con, tìm các "mảnh ghép" tương ứng ở các nguồn.
3. **Identifying mediation queries** — kết hợp (combine) các partial mapping lại thành truy vấn hoàn
   chỉnh.

Sơ đồ bên dưới minh hoạ trước bằng ví dụ: lược đồ trung gian được chẻ thành 2 cây con `{id,address}`
(thuộc author) và `{ISBN,title}` (thuộc book); với cây con `author`, S1 có `authorId` còn S2 có đủ
`id,address` → JOIN 2 nguồn này theo `id`; với cây con `book`, chỉ S2 có `ISBN,title` nên dùng thẳng.
Kết quả cuối (khung bên phải) là đoạn mã **XQuery** thật — ngôn ngữ truy vấn cho dữ liệu XML (tương tự
SQL nhưng cho cây, dùng `for...where...return`).

---

## Slide 7 — "Decomposing"

**Nôm na:** quy tắc CHÍNH XÁC để chẻ cây lược đồ trung gian thành các cây con hợp lệ — không được chẻ
tuỳ tiện.

**Kỹ thuật — điều kiện 1 cây con hợp lệ:**
- **Root** (gốc cây con): phải là 1 node multi-value (có dấu `+`, ví dụ `author`, `book`, `chapter`)
  HOẶC chính là gốc của lược đồ trung gian.
- **Node** (các node còn lại trong cây con): phải là mono-value (không lặp).
- Trong cây con phải tồn tại **ít nhất 1 text node** (node lá có giá trị thật, ví dụ `id`, `title`,
  không phải chỉ node cấu trúc rỗng).

Ví dụ minh hoạ: cây `library→author(+)→{id,address,book(+)→{ISBN,title,chapter(+)→{Number,abstract}}}`
được chẻ thành đúng 3 cây con: `t1={author,id,address}`, `t2={book,ISBN,title}`,
`t3={chapter,Number,abstract}` — mỗi cây con có gốc là 1 node `+` (author/book/chapter) và có text
node lá bên trong.

---

## Slide 8 — "Partial mappings" (bước 1/2 của bước 2 lớn)

**Nôm na:** với mỗi cây con vừa chẻ ở Slide 7, đi "soi" xem nguồn nào có chứa phần dữ liệu tương ứng.

**Kỹ thuật (1):** For each sub-tree, identifying correspondent parties in sources. Với cây con
`t1={author,id,address}`, thuật toán soi cả 3 nguồn S1/S2/S3 và khoanh vùng (khung chữ nhật trên
slide) đúng phần `authorId` (ở S1), `author/{id,address}` (ở S2), `author/{id,address}` (ở S3) khớp
với `t1`. Đây là bước TRA CỨU lại đúng bảng tương ứng ngữ nghĩa đã cho ở Slide 5, không phải suy luận
mới.

---

## Slide 9 — "Partial mapping (2)"

**Nôm na:** biết nguồn nào có mảnh dữ liệu nào rồi, giờ cần biết CÁCH GHÉP các mảnh đó lại — tức tìm
các phép JOIN khả dĩ GIỮA các nguồn.

**Kỹ thuật (2):** Identifying possible join operations between sources, dựa trên chính các semantic
correspondences (tương ứng ngữ nghĩa) đã cho. Với ví dụ `t1` (author): slide tìm được 3 điều kiện
join khả dĩ: `j1: authorId=id` (nối S1.book.authorId với S3.author.id), `j2: authorId=id` (nối
S1.book.authorId với S2.author.id), `j3: id=id` (nối S2.author.id với S3.author.id) — 3 mảnh dữ liệu
tương ứng được gắn nhãn `p1` (ở S1), `p2` (ở S2), `p3` (ở S3).

---

## Slide 10 — "Partial mapping (3)"

**Nôm na:** ngoài join GIỮA 2 nguồn khác nhau (Slide 9), còn cần xét join NGAY TRONG 1 nguồn — dựa
trên quan hệ khoá chính/khoá ngoại nội tại của chính nguồn đó.

**Kỹ thuật:** Identifying possible inter-join operation dựa trên **Key & reference key** (khoá chính
và khoá tham chiếu). Ví dụ: trong nguồn S1, bảng `Author` có `Id` là khoá chính (gạch chân, ký hiệu
`p`), còn bảng `Address` (bên dưới `Author`, vẫn trong S1) có `AuthorId` là khoá ngoại tham chiếu tới
`Author.Id` (ký hiệu `p'`), điều kiện join nội tại là `authorId=id`. Việc này cần thiết vì mediation
schema (`MS`) coi `Author` là 1 thực thể gộp cả `Id, Name, Address` — nhưng trong S1 các thuộc tính
đó có thể nằm tách ở 2 bảng con khác nhau, phải tự join lại trước khi so khớp với `MS`.

---

## Slide 11 — "Partial mapping (4)"

**Nôm na:** hình thức hoá lại toàn bộ các khả năng join tìm được ở Slide 9-10 thành 1 **đồ thị join**
(join graph) để dễ xử lý bằng thuật toán.

**Kỹ thuật:** Graph of joins: **node = 1 nguồn** (hoặc 1 mảnh/fragment `p1,p2,p3` của nguồn đó),
**cạnh (axe) = 1 điều kiện join**. Khi đó, **partial mapping = 1 tập hợp con (sub-graph) của đồ thị
join này** — tức không bắt buộc dùng CẢ 3 cạnh `j1,j2,j3` cùng lúc (vì `p1,p2,p3` đã khoá vòng tam
giác, dùng 2 cạnh bất kỳ là đủ suy ra cạnh thứ 3), nên có NHIỀU partial mapping hợp lệ khác nhau cho
cùng 1 cây con — đây chính là lý do Slide 12 nói "có nhiều mediation query khác nhau".

---

## Slide 12 — "Generating mediation queries" (bước 3 lớn — kết hợp)

**Nôm na:** đây là bước GHÉP các partial mapping của từng cây con lại thành 1 truy vấn hoàn chỉnh —
và vì mỗi cây con có thể có NHIỀU partial mapping hợp lệ (Slide 11), nên có thể sinh ra NHIỀU truy vấn
trung gian khác nhau, mỗi truy vấn mang 1 "ý nghĩa" (semantics) hơi khác (ví dụ ưu tiên nguồn này hay
nguồn kia khi có trùng lặp).

**Kỹ thuật:**
- A mediation query ~ a combine of partial mappings: chọn đúng 1 partial mapping cho MỖI cây con, rồi
  ghép lại.
- Having some mediation queries (with different semantics): vì có nhiều lựa chọn partial mapping, nên
  có nhiều mediation query khả dĩ, không phải chỉ 1 câu "đúng nhất".
- **Union of 2 mediation queries ~ a mediation query**: nếu 2 truy vấn trung gian khác nhau đều hợp
  lệ (ví dụ 1 cái lấy dữ liệu tác giả từ S1⋈S3, 1 cái lấy từ S2), thì HỢP (UNION) chúng lại cũng VẪN
  LÀ 1 mediation query hợp lệ — đúng tinh thần đã thấy ở ví dụ quan hệ Slide 2-3 (UNION của T1⋈T2 và
  T1⋈T3).

---

## Slide 13 — "Generating ... (2)" — kiểm tra quan hệ cha-con

**Nôm na:** đây là một "cái bẫy" quan trọng của dữ liệu cây (khác hẳn dữ liệu bảng phẳng): chỉ khớp
GIÁ TRỊ KHOÁ thôi là CHƯA ĐỦ, còn phải chắc chắn 2 mảnh dữ liệu khớp nhau đó thực sự **cùng thuộc về 1
phần tử cha** trong cây gốc.

**Kỹ thuật:** Verifying parent-child relations in sub-trees. Ví dụ cụ thể trên slide: trong S1, node
`chapter/Number` (p6) muốn join với node `Number/abstract` ở S3 (p7) — điều kiện join đầy đủ là
**`j5: Number=Number VÀ ISBN=ISBN`**, tức không chỉ so khớp `Number` (số chương) mà còn phải so khớp
LUÔN `ISBN` của cuốn sách cha chứa chương đó. Nếu chỉ so `Number=Number` mà bỏ qua `ISBN=ISBN`, có
thể vô tình nối nhầm "chương số 3 của sách A" với "chương số 3 của sách B" — sai hoàn toàn về ngữ
nghĩa dù khoá `Number` trùng nhau. Đây chính là lý do thuật toán phải "verify parent-child" trước khi
chấp nhận 1 điều kiện join là hợp lệ.

---

## Slide 14 — "Generating mediation queries" (kết quả cuối: mã XQuery hoàn chỉnh)

**Nôm na:** đây là sản phẩm CUỐI CÙNG của toàn bộ quy trình 3 bước — 1 câu truy vấn XQuery thật sự
chạy được, ghép đúng tất cả partial mapping đã xác định ở các slide trước cho ví dụ "thư viện sách"
xuyên suốt bài.

**Kỹ thuật:** đoạn mã XQuery (rút gọn, phần cuối bị `………` che bớt trên slide gốc — không bịa thêm
phần bị cắt):

```
<Library>{
for $id = distinct(S1/Library/book/authorId, S2/authors/author/Id)
for $p1 in S1/Library/book/authorId
for $p2 in S2/authors/author
where $p1 = $p2/id and $p1 = id
return <Author>{
    <id>{$p1/text()}</id>,
    <Address>{$p2/address/text()}</Address>
    for $isbn = distinct(S1/Library/book/ISBN)
    for $p4 in S1/Library/book[authorId = $p1]
    where $p4/ISBN = $isbn
    return <Book>{
        <ISBN>{$p4/ISBN/text()}</ISBN>,
        <title>{$p4/title/text()}</title>
        for $num = distinct(S1/Library/book/Chapter/Number, S3/Library/book/Chapter/Number)
        for $p6 in S1/Library/book[ISBN = $p4/ISBN]/Chapter
        for $p7 in S3/Library/book/Chapter
        where $p6/Number = $p7/Number and $p6/Number = $num
              and $p6/PARENT::book/ISBN = $p7/PARENT::book/ISBN
        return <Chapter>{
            ……………………………………..
}</Library>
```

Đọc kỹ sẽ thấy cấu trúc truy vấn này LỒNG NHAU đúng 3 tầng của cây lược đồ trung gian (author → book →
chapter), và mỗi tầng `for...where...return` tương ứng đúng 1 partial mapping đã xác định ở các slide
trước: tầng `Author` dùng join S1⋈S2 theo `id` (đúng `j2` ở Slide 9), tầng `Chapter` dùng điều kiện
kép `Number=Number AND ISBN=ISBN` (đúng `j5` + ràng buộc cha-con ở Slide 13). Đây là minh chứng cụ thể
cho thấy toàn bộ lý thuyết "Decomposing → Partial mapping → Combine" ở các slide trước thực sự SINH RA
được 1 câu truy vấn thực thi được, không chỉ là lý thuyết suông.
