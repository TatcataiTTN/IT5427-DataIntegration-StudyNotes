# Giải thích chi tiết — Sources Profiling & Selection (slide 5, IT5427)

Nguồn: `slides/5_SourcesSelection.pdf` — 25 trang PDF vật lý, mỗi trang in 2 slide gốc theo chiều dọc (tổng cộng 50 slide đánh số 1-50 ở góc mỗi khung). Giải thích dưới đây đi theo đúng thứ tự 25 trang vật lý, mỗi trang gồm 2 mục con ứng với 2 slide bên trong.

Bài giảng này là **bước cuối cùng** trong toàn bộ quy trình Data Integration đã học: sau khi đã có mediated schema (Schema Alignment), mapping GAV/LAV/GLAV, và cách viết lại truy vấn (Query Processing/Mediation Query), câu hỏi còn lại là — làm sao biết nguồn dữ liệu có gì bên trong (**Source Profiling**), làm sao tự động "bóc" dữ liệu từ các trang web không có API chuẩn (**Wrapper**), và khi có quá nhiều nguồn trùng lặp/dư thừa thì nên chọn tập con nào để tích hợp cho tiết kiệm mà vẫn đủ tốt (**Source Selection**).

---

## Trang 1 — Tiêu đề & Kiến trúc hệ thống tích hợp (slide 1-2)

**Slide 1 (bìa):** "Sources Profiling & Selection" — Vu Tuyet Trinh, SoICT-HUST.

**Slide 2 — Sơ đồ kiến trúc tổng thể của một hệ thống truy vấn tích hợp dữ liệu:**
Nôm na: đây là "bản đồ" toàn bộ các bước một câu truy vấn của người dùng phải đi qua trước khi thực sự chạm được vào dữ liệu thật.
- **Query** (câu hỏi người dùng gõ trên mediated schema) → đi vào khối **Query reformulation** (viết lại truy vấn, chính là GAV/LAV/GLAV unfolding đã học ở bài Query Processing/Schema Alignment trước đó) → sinh ra **Logical query plan**.
- Logical query plan → **Query optimizer** (chọn thứ tự join, chọn nguồn nào chạy trước) → sinh ra **Physical query plan**.
- Physical query plan → **Execution engine** (bộ thực thi, thực sự gửi lệnh đi) → phân phối truy vấn xuống nhiều **wrapper**, mỗi wrapper gắn với 1 **source** (nguồn dữ liệu thật — DB, trang web, API...).
- Có 1 mũi tên ngược từ Execution engine quay lại Query optimizer, nhãn **"Re-planning request"**: nghĩa là nếu lúc thực thi phát hiện kế hoạch không ổn (vd 1 nguồn quá chậm, trả về ít dữ liệu hơn dự kiến), hệ thống có thể yêu cầu optimizer lập lại kế hoạch — đây là ý tưởng "adaptive query processing", không lập kế hoạch 1 lần rồi chạy cứng nhắc.

→ Toàn bộ nội dung còn lại của slide này (wrapper, source profiling, source selection) đều là để trả lời câu hỏi: **5 ô "wrapper" và 5 ô "source" ở đáy sơ đồ thực chất hoạt động ra sao, và nên chọn mấy trong số chúng?**

---

## Trang 2 — Wrapper là gì? Ví dụ cụ thể (slide 3-4)

**Slide 3 — Định nghĩa Wrapper:**
Nôm na: wrapper giống như một "người phiên dịch" đứng giữa 1 trang web/tài liệu lộn xộn và hệ thống tích hợp — nó đọc trang đó rồi "bóc" ra đúng những ô dữ liệu hệ thống cần, theo đúng khuôn mẫu (schema) hệ thống muốn.
Kỹ thuật: 1 wrapper W là 1 trích xuất có cấu trúc từ các trang của nguồn S, hình thức hoá thành cặp **(T_W, E_W)**:
- **T_W** (target schema): lược đồ ĐÍCH mà wrapper trả về — KHÔNG nhất thiết giống hệt lược đồ gốc T_S của trang (vì có thể chỉ cần trích 1 phần thuộc tính của T_S, không cần tất cả).
- **E_W** (extraction program): chương trình trích xuất, dùng định dạng hiển thị F_S của trang để lấy ra 1 instance dữ liệu đúng khuôn T_W. Thường viết bằng ngôn ngữ script (Perl) hoặc ngôn ngữ khai báo cấp cao hơn.

**Slide 4 — Ví dụ cụ thể (countries.com):**
Một trang World Factbook hiển thị "GERMANY / Capital: Berlin / Population: 81757600 / Continent: Europe". Wrapper ở đây có T_W = T_S = (country, capital, population, continent) — lấy đủ toàn bộ 4 thuộc tính. Chương trình trích E_W là 1 script Perl với quy tắc kiểu: "lấy chuỗi VIẾT HOA TOÀN BỘ đầu tiên làm country", "lấy chuỗi ngay sau nhãn 'Capital:' làm capital", v.v. Đây chính là minh hoạ bằng tay cho khái niệm trừu tượng ở slide 3.

---

## Trang 3 — Lược đồ nguồn vs. định dạng trang; ví dụ sách (slide 5-6)

**Slide 5 — Phân biệt Schema T_S và Format F_S:**
Đây là 1 phân biệt rất quan trọng hay bị nhầm: **Schema** (T_S) là các TRƯỜNG LOGIC và cách chúng lồng nhau (vd (country, capital, population, continent)) — thường KHÔNG được ghi rõ ràng trên 1 trang được sinh tự động. **Format** (F_S) là CÁCH trang hiển thị các trường đó ra màn hình — dùng thẻ HTML, nhãn chữ, và các vùng lặp lại (vd "capital đi ngay sau nhãn 'Capital:'"). Nói cách khác: schema là "có cái gì", format là "nó được trình bày thế nào" — 2 nguồn có thể CÙNG schema nhưng KHÁC format (vd 1 trang dùng bảng HTML, 1 trang dùng danh sách).

**Slide 6 — Ví dụ worked: 1 trang sách:**
Nội dung trang: `<b>Data Integration</b>`, `<u>A. Author</u>`, "Price: $45.00", "Publisher: Academic Press". Target T_W = Book(title, price) — wrapper CHỈ cần 2 trường, bỏ qua tác giả/nhà xuất bản dù chúng có mặt trên trang (minh hoạ đúng ý "T_W không cần giống T_S" ở slide 3). Quy tắc trích: title = chữ nằm trong thẻ `<b>`, price = phần số ngay sau "Price: $". Kết quả: `('Data Integration', 45.00)`.

---

## Trang 4 — Hai bài toán xây dựng wrapper; các thách thức (slide 7-8)

**Slide 7 — Hai biến thể bài toán xây dựng wrapper:**
| Biến thể | Đầu vào đã biết | Đầu ra cần tìm |
|---|---|---|
| Biết trước target schema | Các trang + các trường mong muốn | Chương trình trích E_W |
| Không biết schema nguồn | Chỉ có các trang của 1 nguồn | Cả schema T_S lẫn chương trình E_W |

Ghi chú quan trọng của slide: "Học được CẤU TRÚC không tự động cho biết Ý NGHĨA của các trường" — tức dù máy có tự dò ra được "có 4 cột dữ liệu lặp lại" trên 1 trang, nó vẫn không tự biết cột nào là "giá", cột nào là "tên sản phẩm" nếu không có thêm thông tin khác (nhãn text, ví dụ mẫu...).

**Slide 8 — Các khó khăn khi xây wrapper:**
- Suy luận ra 1 "văn phạm" (grammar) chỉ từ các ví dụ ĐÚNG (không có ví dụ sai để đối chiếu) vốn là bài toán khó về mặt lý thuyết học máy.
- Dù đã biết target schema, vẫn có RẤT NHIỀU chương trình trích xuất khác nhau cùng khớp với dữ liệu mẫu — chọn cái nào mới "đúng" cho các trang mới?
- Trường tuỳ chọn (optional) và danh sách lặp lại làm mờ ranh giới bắt đầu/kết thúc của 1 giá trị.
- Có ngoại lệ định dạng (formatting exception) làm hỏng 1 quy tắc tưởng chừng hợp lý.
- Một quy tắc khớp hoàn hảo với trang huấn luyện vẫn có thể SAI khi gặp trang mới (overfitting) — đúng tinh thần "học máy" áp dụng vào bài toán trích xuất web.

---

## Trang 5 — 4 hướng giải pháp chính; bảng tổng hợp công cụ (slide 9-10)

**Slide 9 — 4 cách tiếp cận chính để xây wrapper:**
- **Manual (thủ công):** lập trình viên tự tay viết T_W và E_W — chính xác nhất nhưng tốn công, không scale.
- **Learning (học có giám sát):** lập trình viên đánh dấu (highlight) các thuộc tính của T_W trên 1 tập trang mẫu, rồi áp dụng thuật toán học để tự suy ra E_W.
- **Automatic (tự động hoàn toàn):** tự học CẢ T_S lẫn E_W chỉ từ 1 tập trang web, không cần người đánh nhãn.
- **Interactive (tương tác):** kết hợp học + tự động — chương trình hỏi lại người dùng feedback tới khi hội tụ ra kết quả đúng.
→ Đây chính là "kim chỉ nam" cho toàn bộ phần còn lại của bài: HLRT/Stalker (thuộc nhóm Learning), RoadRunner (thuộc nhóm Automatic), Poly/Lixto/Stalker co-testing (thuộc nhóm Interactive).

**Slide 10 — Bảng đối chiếu input của con người ↔ phương pháp đại diện:**
| Hướng | Input của con người | Phương pháp đại diện |
|---|---|---|
| Manual | Quy tắc/code trích xuất | Regex, DOM paths |
| Learning | Target schema + trang đã gán nhãn | HLRT, Stalker |
| Automatic | Trang chưa gán nhãn | RoadRunner |
| Interactive | Feedback trong lúc tìm kiếm | Co-testing, Poly, Lixto |
Slide còn liệt kê thêm các công cụ/thư viện SCRAPING THỰC TẾ ngày nay hay dùng (không thuộc 4 phương pháp học thuật trên, chỉ là ví dụ công cụ công nghiệp): Beautiful Soup, Pandas, Selenium, Scrapy, Instant Data Scraper, Firecrawl — ý muốn nói các công cụ hiện đại thường kết hợp cách tiếp cận Manual (cho developer viết selector) với 1 phần hỗ trợ tự động.

---

## Trang 6 — Trích xuất thủ công: mô hình String vs. mô hình DOM (slide 11-12)

**Slide 11 — So sánh 2 mô hình trích xuất tay:**
| | Mô hình String | Mô hình DOM |
|---|---|---|
| Coi trang như | Chuỗi ký tự/token tuần tự | Cây các phần tử HTML |
| Cách định vị | Dùng dấu phân cách (delimiter)/regex | Dùng đường dẫn (path) trỏ tới 1 trường/bản ghi |
| Hoạt động tốt khi | Ranh giới cục bộ ổn định | Có ngữ cảnh cấu trúc quanh giá trị |
| Dễ hỏng khi | Cùng 1 thẻ xuất hiện ở vùng không liên quan | Thay đổi layout làm lệch đường dẫn |

Nôm na: mô hình String giống như "tìm từ khoá rồi lấy phần sau nó" (dễ vỡ nếu từ khoá đó xuất hiện ở chỗ khác ngoài ý muốn); mô hình DOM giống như "đi theo bản đồ thư mục cây HTML tới đúng ô" (dễ vỡ nếu ai đó chèn thêm 1 thẻ làm lệch vị trí trong cây).

**Slide 12 — Ví dụ DOM path cụ thể:**
| Trường | Đường dẫn DOM | Giá trị trích được |
|---|---|---|
| title | `/html/body/div[1]/table/td[2]/text()` | Inception |
| rating | `/html/body/div[2]/table/td[2]/text()` | 8.4 |
| runtime | `/html/body/div[2]/table/td[4]/text()` | 148 mins |
Ghi chú slide: "Một `div` hoặc ô bảng bị chèn thêm có thể làm lệch đường dẫn vị trí (positional path)" — đây chính là minh hoạ thực tế cho điểm yếu "dễ hỏng khi layout thay đổi" đã nêu ở slide 11.

---

## Trang 7 — HLRT: mô hình delimiter (slide 13-14)

**Slide 13 — Định nghĩa HLRT (Head-Left-Right-Tail):**
Nôm na: HLRT là cách "đóng khung" 1 vùng dữ liệu bằng các chuỗi mốc (delimiter) đặt ở đầu/cuối, giống như dùng dấu ngoặc để khoanh vùng cần lấy.
Kỹ thuật: với n thuộc tính cần trích, cần xác định **2n + 2 chuỗi mốc**: W = (h, t, l₁, r₁, …, lₙ, rₙ) — trong đó **h** và **t** (head/tail) đánh dấu ranh giới của CẢ VÙNG dữ liệu trên trang, còn **lᵢ** và **rᵢ** (left/right) đánh dấu ranh giới của RIÊNG giá trị thuộc tính thứ i.

**Slide 14 — Ví dụ worked HLRT:**
Đoạn HTML: `<b>Calling codes</b><P>` ... `<B>Australia</B> <I>61</I><BR>` ... `<B>East Timor</B> <I>670</I><BR>` ... `<HR><b>Copyright</b>`.
Tham số suy ra: h = `<P>`, t = `<HR>` (khoanh toàn vùng danh sách mã quốc gia); l₁ = `<B>`, r₁ = `</B>` (khoanh tên nước); l₂ = `<I>`, r₂ = `</I>` (khoanh mã số điện thoại). Kết quả trích: (Australia, 61), (East Timor, 670) — đúng 2n+2 = 6 chuỗi mốc cho n=2 thuộc tính như công thức ở slide 13.

---

## Trang 8 — Học tham số HLRT; giới hạn của HLRT (slide 15-16)

**Slide 15 — Quy trình học tham số HLRT từ dữ liệu mẫu:**
Gán nhãn điểm bắt đầu/kết thúc của MỌI giá trị cần trích trên các trang mẫu → tìm ứng viên h, t là các chuỗi xuất hiện chung ở ranh giới trang được chia sẻ giữa các mẫu → tìm ứng viên delimiter TRÁI là hậu tố chung đứng NGAY TRƯỚC các giá trị → tìm ứng viên delimiter PHẢI là tiền tố chung đứng NGAY SAU các giá trị → tìm kiếm 1 tổ hợp tham số khớp với TẤT CẢ nhãn đã gán. Đây là 1 bài toán tìm kiếm trong không gian tổ hợp các chuỗi ứng viên.

**Slide 16 — Giới hạn của mô hình HLRT:**
- Giả định lược đồ là 1 TUPLE PHẲNG (flat) — không xử lý được cấu trúc lồng nhau.
- MỌI trường đều cần có chuỗi delimiter đáng tin cậy — nếu 1 trường không có mốc rõ ràng thì HLRT bó tay.
- Danh sách lồng nhau (nested list) không khớp với mô hình tuần tự đơn giản này.
- Định dạng pha trộn (mixed format) khiến khó tìm ra delimiter chung.
→ Slide tự chỉ ra lối thoát: **"Stalker giới thiệu trích xuất phân cấp (hierarchical) và landmark phong phú hơn"** — đây chính là cầu nối dẫn sang 2 slide tiếp theo.

---

## Trang 9 — Stalker: lược đồ tuple lồng nhau; quy tắc & landmark (slide 17-18)

**Slide 17 — Lược đồ lồng nhau của Stalker (ví dụ nhà hàng):**
| Node | Kiểu | Ví dụ nội dung |
|---|---|---|
| restaurant | Tuple | name, cuisine, addresses |
| addresses | Danh sách các tuple | 1 hoặc nhiều địa điểm |
| address | Tuple | street, city, state, zip, phone |
| city | Vô hướng (scalar) | Phoenix |
Nguyên lý: "Trích xuất ĐI XUỐNG DẦN qua lược đồ, dùng chính văn bản đã trích của nút CHA làm ngữ cảnh cho nút CON" — đây chính là cách Stalker khắc phục giới hạn "chỉ xử lý tuple phẳng" của HLRT ở trang trước: thay vì 1 lần quét toàn trang, Stalker quét nhiều lần, mỗi lần thu hẹp dần vùng tìm kiếm.

**Slide 18 — Lệnh và landmark của Stalker:**
| Lệnh | Ý nghĩa | Landmark |
|---|---|---|
| SkipTo(x) | quét tới trước, đi qua landmark x | Token chữ nghĩa đen: `<b>`, "Cuisine:", dấu phẩy |
| BackTo(x) | quét lùi lại, đi qua x | Lớp token: Numeric, HTMLTag, Punctuation |
| BackUntil(x) | dừng tại x nhưng KHÔNG tiêu thụ nó | Luật có thể gồm các phương án có thứ tự ưu tiên |
Một CHUỖI các lệnh sẽ thu hẹp dần ngữ cảnh tìm kiếm; luật bắt đầu/kết thúc xác định 1 đoạn con cụ thể. Nôm na: thay vì chỉ dùng 1 cặp mốc trái/phải cứng nhắc như HLRT, Stalker cho phép "đi tới mốc này, rồi lùi qua mốc kia" — linh hoạt hơn nhiều khi dữ liệu không đều.

---

## Trang 10 — Dấu vết trích xuất Stalker; học bằng sequential covering (slide 19-20)

**Slide 19 — Dấu vết (trace) trích xuất Stalker qua ví dụ nhà hàng:**
| Bước | Ngữ cảnh | Kết quả |
|---|---|---|
| 1 | Toàn bộ trang nhà hàng | name và cuisine |
| 2 | Toàn bộ trang nhà hàng | Đoạn con danh sách địa chỉ |
| 3 | Đoạn con danh sách địa chỉ | Các đoạn con địa chỉ riêng lẻ |
| 4 | 1 đoạn con địa chỉ | street, city, state, zip, phone |
Ghi chú: "1 luật con chỉ nhìn thấy vùng đã trích của luật cha, chứ không phải toàn trang" — minh hoạ cụ thể cho nguyên lý "đi xuống dần qua lược đồ" đã nêu ở slide 17.

**Slide 20 — Học Stalker bằng Sequential Covering:**
Dùng các trang có gán nhãn sẵn ví dụ cho từng nút trong lược đồ → học 1 luật PHỦ ĐÚNG một phần ví dụ huấn luyện → LOẠI BỎ các ví dụ đã được phủ đó rồi học tiếp 1 luật khác cho phần còn lại → kết hợp các luật đã học bằng phép HOẶC (disjunction) → tinh chỉnh các ứng viên bằng cách mở rộng landmark hoặc thêm lệnh. Đây là thuật toán học quy tắc kinh điển (giống ý tưởng "rule induction" trong học máy symbolic) áp dụng cho bài toán trích xuất web.

---

## Trang 11 — RoadRunner: học tự động từ trang chưa gán nhãn (slide 21-22)

**Slide 21 — Nguyên lý RoadRunner:**
Đầu vào: các trang CHƯA GÁN NHÃN nhưng cùng sinh ra từ 1 nguồn (vd cùng 1 template website). Đầu ra: cả lược đồ lẫn chương trình trích khớp. Cách làm: khởi tạo chương trình bằng CHÍNH 1 trang đầu tiên (coi trang đó là "khuôn mẫu" ban đầu) → TỔNG QUÁT HOÁ dần khuôn mẫu đó để khớp được với mỗi trang bổ sung tiếp theo → các trường cấu trúc học được sẽ mang tên chung chung như A, B (vì máy không biết ý nghĩa thật của chúng — đúng đúng cảnh báo đã nêu ở slide 7: "học cấu trúc không tự động cho biết ý nghĩa").

**Slide 22 — 3 loại "điểm không khớp" (mismatch) và cách tổng quát hoá:**
| Loại không khớp | Diễn giải | Cách tổng quát hoá |
|---|---|---|
| Văn bản khác nhau | Giá trị trường biến đổi | `#PCDATA` (ký hiệu "đây là dữ liệu tự do") |
| Có thêm 1 vùng cấu trúc | Nội dung tuỳ chọn | `(pattern)?` (0 hoặc 1 lần) |
| Số lần lặp khác nhau | Danh sách hoặc bộ lặp | `(pattern)+` (1 hoặc nhiều lần) |
Ghi chú: "Với 1 điểm không khớp về THẺ, thuật toán thử bộ lặp (iterator) TRƯỚC khi thử tuỳ chọn (optional)" — tức có 1 thứ tự ưu tiên cố định khi máy phải đoán xem sự khác biệt giữa 2 trang là do "có/không có" hay do "lặp bao nhiêu lần".

---

## Trang 12 — Ví dụ tổng quát hoá RoadRunner; xây dựng tương tác (slide 23-24)

**Slide 23 — Ví dụ worked tổng quát hoá:**
| Trang quan sát | Mẫu tổng quát hoá |
|---|---|
| Trang 1: chủ đề 'Database', không ảnh, 2 sách | Chủ đề: `<B>#PCDATA</B>` |
| Trang 2: chủ đề 'Data Integration', có ảnh, 3 sách | Ảnh tuỳ chọn: `(<IMG…/>)?` |
| Chủ đề và tên sách khác nhau | Danh sách sách: `(<LI>…#PCDATA…</LI>)+` |
| Số lượng mục sách khác nhau | Phép lặp lồng nhau có thể cần khớp đệ quy |
Đây chính là áp dụng cụ thể 3 loại mismatch ở slide 22 vào 1 ví dụ 2 trang web sách thực tế — cho thấy làm sao từ 2 trang cụ thể, máy tự suy ra 1 "khuôn mẫu chung" biểu diễn cả 2.

**Slide 24 — Xây dựng wrapper kiểu tương tác (Interactive):**
Duy trì nhiều wrapper ứng viên cùng lúc và phát hiện khi nào máy "không chắc chắn" → hỏi người dùng feedback trên 1 trang/kết quả trích xuất hữu ích (không hỏi lan man, chỉ hỏi đúng chỗ còn mơ hồ) → dùng câu trả lời để loại bớt hoặc tinh chỉnh các ứng viên còn lại → lặp lại tới khi wrapper làm người dùng hài lòng. Ghi chú quan trọng: "Lựa chọn thiết kế then chốt là KHI NÀO hỏi và HỎI GÌ" — đây là bài toán active learning áp dụng vào xây dựng wrapper.

---

## Trang 13 — So sánh 3 hệ tương tác; chuyển sang Big Data Integration (slide 25-26)

**Slide 25 — Bảng so sánh 3 hệ thống tương tác cụ thể:**
| Hệ thống | Tương tác với người dùng | Cơ chế tìm kiếm |
|---|---|---|
| Stalker co-testing | Gán nhãn 1 trang có bất đồng (disagreement) | So sánh luật chạy xuôi và chạy ngược |
| Poly | Chọn ra bộ tuple trích xuất ĐÚNG | Thu hẹp ứng viên dựa trên DOM |
| Lixto | Highlight bản ghi và trường | Xây luật trích xuất kiểu Datalog |

**Slide 26 — Chuyển chủ đề: Big Data Integration (mở đầu phần 2 của bài giảng):**
Đặt lại 2 câu hỏi cốt lõi cho cả phần còn lại: **Source profiling** — mỗi nguồn chứa gì, dữ liệu của nó phù hợp tới đâu? **Source selection** — nên tích hợp nguồn nào, với mục tiêu và ngân sách ra sao? 3 chủ đề con sẽ triển khai tiếp: Dynamic source selection, Source summarization, Profiling at scale.
Có kèm 1 hình ảnh minh hoạ ghép hình dạng mảnh ghép (puzzle), biểu diễn **5V của Big Data** quen thuộc: **VELOCITY** (tốc độ dữ liệu được sinh ra/phân tích), **VALUE** (tiềm năng kinh tế-xã hội của big data), **VOLUME** (số lượng/độ chính xác/độ bất định của dữ liệu và nguồn), **VARIETY** (nhiều loại/định dạng dữ liệu khác nhau kể cả phi cấu trúc), **VERACITY** (lượng khổng lồ dữ liệu sinh ra qua số hoá quy mô lớn). Đây là khung quen thuộc đã nhắc tới ở các bài slide trước trong cùng môn học, dùng để nhấn mạnh: khi dữ liệu đạt quy mô "big", việc profiling/selection nguồn không còn làm thủ công được nữa.

---

## Trang 14 — Bảng Profiling vs. Selection; các chủ đề nổi lên (slide 27-28)

**Slide 27 — Bảng đối chiếu Source Profiling vs. Source Selection (bảng quan trọng nhất của bài):**
| Khía cạnh | Source profiling | Source selection |
|---|---|---|
| Mục đích chính | Hiểu & mô tả đặc điểm nguồn | Chọn 1 tập con nguồn hữu ích |
| Đầu vào | Dữ liệu, lược đồ, metadata, lịch sử cập nhật | Hồ sơ nguồn (profile), mục tiêu tích hợp, chi phí |
| Hoạt động chính | Phát hiện kiểu dữ liệu, giá trị thiếu, khoá, phụ thuộc, ngữ nghĩa và chất lượng | So sánh độ phủ, độ chính xác, độ mới, độ trùng lặp và giá trị biên |
| Đầu ra | 1 hồ sơ mô tả mỗi nguồn | Tập nguồn đã chọn, và với dữ liệu động là tần suất thu thập |
| Thách thức chính | Quy mô, định dạng không đồng nhất, ngữ nghĩa không rõ | Phụ thuộc giữa các nguồn, lợi ích không chắc chắn, chất lượng thay đổi |
Nôm na: Profiling trả lời "nguồn này CÓ GÌ", Selection trả lời "nên LẤY nguồn nào" — Profiling luôn phải làm TRƯỚC Selection vì Selection cần input chính là hồ sơ do Profiling tạo ra (đúng như mũi tên "Input: Source profiles" trong bảng).

**Slide 28 — 3 chủ đề nổi lên (emerging topics) của Big Data Integration sẽ triển khai tiếp:** Crowdsourcing, Source selection, Source profiling — đây là slide mục lục cho phần còn lại.

---

## Trang 15 — Crowdsourcing trong tích hợp dữ liệu; Hybrid record linkage (slide 29-30)

**Slide 29 — Crowdsourcing trong tích hợp dữ liệu:**
Nôm na: có những việc máy làm chưa tốt (vd phân biệt 2 bản ghi có PHẢI cùng 1 thực thể hay không khi dữ liệu mơ hồ) — crowdsourcing nghĩa là nhờ CON NGƯỜI (đám đông) làm phần việc đó thay máy. Con người đóng góp các phán đoán mà việc tự động hoá vẫn còn khó khăn; ứng dụng gồm schema alignment và record linkage (2 chủ đề cốt lõi đã học ở các bài trước). Hệ thống phải: (1) tuyển và giữ chân người đóng góp, (2) kết hợp các đóng góp thành 1 lời giải thống nhất, (3) đánh giá người đóng góp và chất lượng công việc của họ.

**Slide 30 — Hybrid Record Linkage (kết hợp máy + người):**
Gắn nhãn thủ công từng cặp bản ghi theo kiểu "naive" cần tới O(n²) lượt gán nhãn của con người — quá tốn kém khi n lớn. Giải pháp: **lọc bằng tính toán** (computational filtering, tương đương bước Blocking đã học ở Record Linkage) loại bỏ trước các cặp chắc chắn KHÔNG khớp → **đám đông chỉ gán nhãn các cặp ứng viên còn lại** (đã được thu hẹp nhiều) → dùng **quan hệ bắc cầu** (transitive) để suy ra thêm nhãn cho các cặp khác mà không cần hỏi người → thứ tự gán nhãn ảnh hưởng tới số lượng việc còn lại phải làm bằng tay.

---

## Trang 16 — Suy luận bắc cầu hợp lệ/không hợp lệ; ví dụ cụ thể (slide 31-32)

**Slide 31 — Bảng quy tắc suy luận bắc cầu (transitive inference):**
| Nhãn đã biết | Suy luận | Trạng thái |
|---|---|---|
| A = B và B = C | A = C | Hợp lệ |
| A = B và B ≠ C | A ≠ C | Hợp lệ |
| A ≠ B và B ≠ C | A ? C | KHÔNG suy luận được |
Ghi chú quan trọng: "Các quy tắc này giả định nhãn định danh NHẤT QUÁN. Nhãn sai có thể LAN TRUYỀN" — đây là cảnh báo rủi ro cốt lõi: nếu con người gán nhãn SAI 1 cặp, lỗi đó có thể kéo theo nhiều suy luận sai khác qua bắc cầu (dòng cuối cùng của bảng ở slide 32 bên dưới minh hoạ trực tiếp rủi ro này).

**Slide 32 — Ví dụ cụ thể 8 cặp ứng viên (R1-R6):**
| Cặp | Khả năng khớp | Nhãn thật |
|---|---|---|
| L1: (R2,R3) | 0.85 | Match |
| L2: (R1,R2) | 0.75 | Match |
| L3: (R1,R6) | 0.72 | Non-match |
| L4: (R1,R3) | 0.65 | Match |
| L5: (R4,R5) | 0.55 | Match |
| L6: (R4,R6) | 0.48 | Non-match |
| L7: (R2,R4) | 0.45 | Non-match |
| L8: (R5,R6) | 0.42 | Non-match |
Ghi chú bên: sau khi biết L1 (R2=R3 match) và L2 (R1=R2 match), suy luận bắc cầu cho ngay L4 (R1=R3) là match MÀ KHÔNG CẦN hỏi người — tiết kiệm 1 lượt gán nhãn. Tương tự sau L5, L6 suy ra L8 là non-match. Kết quả: chỉ cần 6 nhãn do người gán (thay vì cả 8) là đủ xác định toàn bộ bảng — slide còn gợi ý lịch gán nhãn song song: vòng 1 gán L1,L2,L3,L5,L6 cùng lúc, vòng 2 chỉ còn L7.

---

## Trang 17 — Bằng chứng & giới hạn của crowd labeling; Corleone (slide 33-34)

**Slide 33 — Bảng bằng chứng thực nghiệm (trích từ 1 chương sách/paper):**
| Kết quả báo cáo | Bối cảnh thử nghiệm | Diễn giải |
|---|---|---|
| Giảm 95% số cặp cần gán nhãn | Mô phỏng trên tập Cora | Nhiều bản ghi khớp cho mỗi thực thể |
| Giảm khoảng 20% số cặp | Mô phỏng trên tập Abt-Buy | Ít thực thể có nhiều bản ghi hơn |
| Giảm từ 1.237 xuống 14 vòng lặp | Cora, ngưỡng 0.3 | Lập lịch song song giúp giảm số vòng |
| Giảm 96.5% số HIT, mất ~5% chất lượng | Cora trên Amazon Mechanical Turk (AMT) | Nhãn sai đã LAN TRUYỀN qua suy luận |
⚠️ Lưu ý: đây là các con số TRÍCH TỪ TÀI LIỆU GỐC của slide (không phải tôi tự tính) — nếu cần trích dẫn chính xác nguồn paper, nên hỏi lại giảng viên vì slide không ghi rõ citation đầy đủ.

**Slide 34 — Corleone: hệ thống crowdsourcing đầu-cuối (end-to-end):**
| Module | Mục đích |
|---|---|
| Blocker | Học luật loại bỏ các cặp chắc chắn KHÔNG khớp |
| Matcher | Huấn luyện 1 random forest bằng nhãn của đám đông |
| Accuracy estimator | Ước lượng precision/recall dù dữ liệu mất cân bằng lớp (ít cặp match hơn nhiều so với non-match) |
| Difficult pairs locator | Xác định các cặp cần thêm công sức đối sánh |
Input của người dùng chỉ cần: 2 bảng dữ liệu, hướng dẫn đối sánh, và 2 ví dụ dương + 2 ví dụ âm — rất ít so với việc phải tự viết toàn bộ luật blocking/matching bằng tay.

---

## Trang 18 — Học & đánh giá của Corleone; Lợi ích-chi phí khi chọn nguồn (slide 35-36)

**Slide 35 — Chi tiết học & đánh giá trong Corleone:**
Đặc trưng tương đồng (similarity features) đại diện cho các cặp ứng viên; **active learning** nhắm tới các nhãn crowd CÓ GIÁ TRỊ nhất (không hỏi tràn lan); luật "negative forest" tận dụng ngược làm luật blocking ứng viên; ước lượng độ chính xác phải xử lý được tình huống cặp dương (match) RẤT HIẾM; các luật độ chính xác cao cô lập trước các ca "dễ" để phần còn lại tập trung vào các cặp thật sự khó.

**Slide 36 — Lợi ích và chi phí khi chọn nguồn (mở đầu phần Source Selection):**
Một nguồn MỚI có thể tăng độ phủ (coverage) hoặc cải thiện độ chính xác khi hợp nhất dữ liệu (data fusion). Một nguồn DƯ THỪA (trùng lặp với nguồn đã có) có thể gần như không mang lại lợi ích gì thêm. Một nguồn CHẤT LƯỢNG THẤP có thể làm GIẢM độ chính xác của dữ liệu đã tích hợp (chứ không chỉ là "không giúp ích"). Chi phí phát sinh ở cả 3 khâu: thu thập (acquisition), xây dựng kỹ thuật (engineering), và tính toán (computation). → Source selection chính là bài toán SO SÁNH các tập con nguồn TRƯỚC KHI tích hợp đầy đủ, để tránh lãng phí.

---

## Trang 19 — Mục tiêu lựa chọn nguồn; Giới hạn của thuật toán tham lam (slide 37-38)

**Slide 37 — 3 dạng mục tiêu khi chọn nguồn (bài toán tối ưu có ràng buộc):**
| Mục tiêu | Ràng buộc |
|---|---|
| Tối đa benefit_I(S) | cost_I(S) ≤ ngân sách |
| Tối thiểu cost_I(S) | benefit_I(S) ≥ mục tiêu |
| Tối đa benefit_I(S) − cost_I(S) | cost_I(S) ≤ ngân sách |
Ghi chú: "Mục tiêu lợi nhuận (profit) đòi hỏi lợi ích và chi phí phải cùng đơn vị đo" — nói cách khác, muốn trừ benefit cho cost thì cả 2 đại lượng phải quy đổi được về cùng 1 thang đo (vd cùng ra tiền, hoặc cùng ra 1 điểm số chuẩn hoá), nếu không phép trừ này vô nghĩa.

**Slide 38 — Giới hạn của chiến lược tham lam (greedy) dựa trên lợi ích biên:**
Lợi ích biên (marginal benefit) của 1 nguồn PHỤ THUỘC vào các nguồn ĐÃ CHỌN trước đó (không cố định) — vì các nguồn có thể trùng lặp (overlap) hoặc bổ sung (complement) lẫn nhau. "Lợi ích giảm dần" (diminishing returns), vốn là giả định nền tảng cho nhiều thuật toán tham lam có đảm bảo lý thuyết, KHÔNG CHẮC đúng với bài toán data fusion. Dừng lại ở 1 điểm biên có thể BỎ LỠ 1 tập con tốt hơn ở xa. Và quan trọng nhất: bài toán chọn nguồn tĩnh (static) ở dạng tổng quát là **NP-complete** — tức không có thuật toán nhanh (đa thức) giải đúng tuyệt đối cho mọi trường hợp, phải dùng heuristic.

---

## Trang 20 — GRASP cho bài toán chọn nguồn tĩnh; ví dụ số cụ thể (slide 39-40)

**Slide 39 — GRASP (Greedy Randomized Adaptive Search Procedure) áp dụng cho chọn nguồn:**
Xây dựng 1 tập ứng viên bằng các lựa chọn NGẪU NHIÊN (không đi theo 1 thứ tự tham lam cứng nhắc duy nhất) → tại mỗi bước chọn trong số các ứng viên TRIỂN VỌNG (không chỉ đúng 1 lựa chọn tốt nhất) → cải thiện tập đã chọn bằng tìm kiếm cục bộ (local search) → lặp lại việc xây dựng + tìm kiếm NHIỀU LẦN → trả về tập con TỐT NHẤT tìm được qua tất cả các lần lặp. Đây chính là 1 kỹ thuật metaheuristic kinh điển để đối phó với tính NP-complete đã nêu ở slide 38 — không đảm bảo tối ưu tuyệt đối nhưng thực tế cho kết quả tốt.

**Slide 40 — Ví dụ số cụ thể minh hoạ bài toán chọn nguồn:**
| Tập con đã chọn | Benefit ($) | Cost ($) | Profit ($) |
|---|---|---|---|
| A | 100 | 40 | 60 |
| B | 80 | 30 | 50 |
| C | 60 | 20 | 40 |
| A+B | 130 | 70 | 60 |
| A+C | 150 | 60 | **90** |
| B+C | 120 | 50 | 70 |
| A+B+C | 160 | 90 | 70 |
Với ngân sách $70: các tập con được PHÉP chọn (cost ≤ 70) là A, B, C, A+B (cost đúng 70), A+C, B+C — trong số đó A+C cho profit cao nhất = 90 (đã kiểm tra lại: A+B+C có cost=90 vượt ngân sách nên KHÔNG hợp lệ dù profit danh nghĩa 70 thấp hơn A+C). Đúng với kết luận ghi trên slide: "Với ngân sách $70, A+C cho lợi nhuận lớn nhất trong các tập con này" — ví dụ này minh hoạ trực tiếp vì sao greedy đơn giản (vd cứ chọn nguồn có benefit/cost tốt nhất trước) có thể bỏ lỡ tổ hợp A+C nếu không xét đủ các cặp.

---

## Trang 21 — Nguồn dữ liệu động: độ mới; Lựa chọn có nhận biết thời gian (slide 41-42)

**Slide 41 — Phân biệt Update frequency vs. Freshness (độ mới):**
| | Update frequency (tần suất cập nhật) | Freshness (độ mới/độ tươi) |
|---|---|---|
| Là gì | Nguồn thay đổi nội dung THƯỜNG XUYÊN ra sao | Giá trị của nguồn PHẢN ÁNH ĐÚNG thế giới đang thay đổi tới mức nào |
| Hệ quả | Thêm dữ liệu mới liên tục vẫn có thể tồn tại SONG SONG với các bản ghi CŨ đã lỗi thời | Cần PHÁT HIỆN thay đổi và XOÁ bản ghi đã lỗi thời |
| Bẫy thường gặp | Cập nhật hằng ngày KHÔNG đảm bảo giá trị hiện tại luôn đúng | Tập con nguồn TỐT NHẤT có thể thay đổi theo thời gian |
Nôm na: 1 nguồn có thể "cập nhật liên tục" (thêm dòng mới mỗi ngày) nhưng vẫn "không tươi" nếu nó không bao giờ XOÁ hay SỬA các dòng cũ đã sai — đây là 2 khái niệm khác nhau, không đồng nhất.

**Slide 42 — Lựa chọn nguồn có nhận biết thời gian (Time-aware selection):**
Mô hình hoá độ phủ, độ chính xác VÀ độ mới theo THỜI GIAN (không chỉ 1 lần tĩnh); ước lượng những thay đổi trong thế giới thực VÀ trong từng nguồn riêng lẻ; chọn ra tập con nguồn CÙNG với tần suất thu thập dữ liệu phù hợp; 1 số trường hợp đặc biệt có thể dùng phương pháp tối ưu hoá submodular (1 lớp hàm có tính chất toán học đặc biệt cho phép có thuật toán xấp xỉ tốt); các kết quả xấp xỉ chỉ đúng khi thoả mãn đúng điều kiện về mục tiêu và chi phí đã nêu.

---

## Trang 22 — Rủi ro khi mô hình hoá chọn nguồn; Source profiling (slide 43-44)

**Slide 43 — Các rủi ro khi mô hình hoá bài toán chọn nguồn:**
Hiện tượng SAO CHÉP (copying) và TƯƠNG QUAN (correlation) giữa các nguồn làm YẾU ĐI giả định độc lập (vốn nhiều mô hình toán học dựa vào) — vd nếu 2 nguồn thực ra cùng sao chép từ 1 nguồn gốc, coi chúng "độc lập" sẽ đánh giá sai lợi ích tổng hợp. Chất lượng ƯỚC LƯỢNG trước có thể khác với chất lượng THỰC TẾ sau khi hợp nhất (fusion). Thay đổi tần suất thu thập có thể làm thay đổi cả chi phí lẫn độ mới. Schema alignment và record linkage (2 bước đã học) CŨNG ảnh hưởng tới giá trị tích hợp cuối cùng — tức chọn nguồn không tách rời hoàn toàn khỏi 2 bước đó. Cuối cùng: cần ĐÁNH GIÁ LẠI tập nguồn đã chọn khi các nguồn tiếp tục biến đổi theo thời gian (không chọn 1 lần rồi để mãi).

**Slide 44 — Source Profiling (chuyển sang phần cuối của bài):**
Profiling giúp người dùng hiểu 1 nguồn còn XA LẠ; gắn các thuộc tính với khái niệm, thực thể và quan hệ; định lượng chất lượng cho các phần LIÊN QUAN của nguồn; phát hiện các cột hữu ích và khả năng quan hệ join tiềm năng; profiling cung cấp BẰNG CHỨNG cho bước source selection ở sau (đúng với mũi tên "Input: Source profiles" đã thấy ở bảng so sánh tại slide 27).

---

## Trang 23 — Bellman: tóm tắt thuộc tính; Công thức Min-hash (slide 45-46)

**Slide 45 — Bellman: tóm tắt thuộc tính cô đọng:**
Nôm na: khi có cả ngàn bảng dữ liệu không ai còn nhớ rõ ý nghĩa, Bellman là 1 công cụ giúp "tóm tắt nhanh" nội dung của từng cột mà không cần đọc hết dữ liệu thô. Cơ sở dữ liệu lớn có thể có HÀNG NGÀN bảng được tài liệu hoá kém; khoá ngoại lưu trữ và tài liệu có thể KHÔNG ĐẦY ĐỦ; Bellman tóm tắt giá trị cột bằng các "chữ ký" (signature) cô đọng; **Min-hash** ước lượng độ chồng lấp (overlap) giữa 2 tập giá trị; tóm tắt kiểu **q-gram** và **multiset** nắm bắt độ tương đồng văn bản và số lượng.

**Slide 46 — Công thức Min-hash resemblance (độ giống nhau ước lượng):**
$$\rho(A,B) = \frac{|A \cap B|}{|A \cup B|}$$
Đây chính là hệ số Jaccard quen thuộc (đã học ở bài Record Linkage — similarity measures) áp dụng cho 2 TẬP GIÁ TRỊ CỘT thay vì 2 bản ghi. Với hàm băm hᵢ: định nghĩa chữ ký sᵢ(A) = giá trị NHỎ NHẤT của hᵢ(a) với mọi a thuộc A — đây chính là kỹ thuật MinHash kinh điển. Ước lượng ρ bằng TỈ LỆ các thành phần chữ ký BẰNG NHAU giữa 2 tập — cách ước lượng này tránh phải quét lại toàn bộ giá trị cột mỗi lần so sánh (rất quan trọng khi có hàng ngàn cột cần so đôi một). Độ chồng lấp càng cao càng gợi ý 1 quan hệ ứng viên (vd khả năng join được) đáng để điều tra thêm.

---

## Trang 24 — Ví dụ worked Min-hash; Tóm tắt lược đồ (slide 47-48)

**Slide 47 — Ví dụ số minh hoạ công thức Min-hash ở trang trước:**
| Đại lượng | Giá trị |
|---|---|
| A | {1, 2, 3, 4} |
| B | {2, 3, 4, 5} |
| Kích thước giao | 3 |
| Kích thước hợp | 5 |
| Độ giống chính xác | 3/5 = 0,60 |
| Số thành phần khớp trong chữ ký đồ chơi 10 thành phần | 6/10 = 0,60 |
Đã kiểm chứng lại: |A∩B| = {2,3,4} → 3 phần tử, |A∪B| = {1,2,3,4,5} → 5 phần tử, ρ = 3/5 = 0.60 — khớp đúng công thức Jaccard ở slide 46. Dòng dưới minh hoạ thêm: dùng 1 chữ ký MinHash 10 thành phần (giả lập) để ƯỚC LƯỢNG, kết quả 6/10 = 0,60 trùng khớp với giá trị CHÍNH XÁC — minh hoạ trực quan rằng ước lượng MinHash có thể cho ra đúng con số thật dù không cần so sánh trực tiếp toàn bộ tập giá trị.

**Slide 48 — Tóm tắt lược đồ (Schema summarization):**
Biểu diễn các bảng và quan hệ join dưới dạng 1 ĐỒ THỊ LƯỢC ĐỒ (schema graph); PHÂN CỤM (cluster) các bảng thành k nhóm; chọn 1 bảng ĐẠI DIỆN cho mỗi nhóm; dùng các đại diện làm nhãn cho 1 bản tóm tắt lược đồ GỌN; cảnh báo quan trọng: "chỉ dựa vào KÍCH THƯỚC bảng hoặc BẬC (degree) của nó trong đồ thị có thể đánh giá SAI mức độ quan trọng" — tức 1 bảng nhỏ vẫn có thể cực kỳ quan trọng nếu nó là điểm nối trung tâm giữa nhiều bảng khác (ý tưởng gần giống "centrality" trong lý thuyết đồ thị mạng).

---

## Trang 25 — Độ quan trọng, độ mạnh và phân cụm; Kết thúc Q&A (slide 49-50)

**Slide 49 — Bảng mô hình hoá độ quan trọng/độ mạnh/phân cụm (hoàn thiện ý tưởng Schema summarization ở trang trước):**
| Thành phần | Mô hình |
|---|---|
| Thông tin của bảng (table information) | Tổng entropy của các thuộc tính |
| Độ quan trọng của bảng (table importance) | Phân phối thông tin ỔN ĐỊNH trên đồ thị lược đồ |
| Độ mạnh đường join (join-path strength) | Tích các độ mạnh của từng cạnh join trên đường đi |
| Độ mạnh giữa 2 bảng | Độ mạnh LỚN NHẤT trong số các đường nối chúng |
| Khoảng cách giữa 2 bảng | 1 − độ mạnh |
| Phân cụm | Thuật toán tham lam weighted k-center |
Nôm na: 1 bảng "quan trọng" không chỉ vì nó to hay có nhiều liên kết, mà vì nó giữ 1 vị trí ỔN ĐỊNH trong "dòng chảy thông tin" của toàn bộ đồ thị lược đồ (ý tưởng này gần giống thuật toán PageRank — độ quan trọng của 1 nút phụ thuộc vào độ quan trọng của các nút nối tới nó, lặp tới khi ổn định). Khoảng cách = 1 − độ mạnh nghĩa là 2 bảng nối với nhau càng "mạnh" (join path càng chắc chắn/phổ biến) thì càng được coi là "gần nhau" trong không gian phân cụm.

**Slide 50 — Slide kết (Q&A):**
Chỉ có biểu tượng hộp thoại "Q & A" (hỏi-đáp), không có nội dung chữ — đây là slide đóng bài giảng, báo hiệu phần hỏi đáp trực tiếp trên lớp, không mang thông tin học thuật cần giải thích thêm.

---

## Tổng kết liên hệ với các bài trước trong môn học

- **Wrapper** (trang 1-12) là mảnh ghép còn thiếu trong sơ đồ kiến trúc "Query reformulation → Optimizer → Execution engine → Wrapper → Source" đã nhắc ở các bài Mediation Query/Query Processing — nó chính là thứ đứng SAU CÙNG, chạm trực tiếp vào dữ liệu thô.
- **Min-hash/Jaccard** (trang 23-24) dùng lại đúng công thức Jaccard similarity đã học ở bài Record Linkage/Entity Resolution, chỉ đổi đối tượng so sánh từ "2 bản ghi" sang "2 tập giá trị cột".
- **Crowdsourcing cho record linkage** (trang 15-18) là phần MỞ RỘNG trực tiếp của quy trình Blocking → Matching → Clustering đã học, thêm 1 lớp "con người" vào giữa để xử lý các ca khó máy chưa quyết được.
- **Source selection như bài toán tối ưu có ràng buộc, NP-complete** (trang 19-21) là 1 góc nhìn "hệ thống/thuật toán" bổ sung cho phần lý thuyết GAV/LAV/GLAV thuần logic đã học ở bài Schema Alignment — tới đây, Data Integration không chỉ còn là bài toán BIỂU DIỄN ánh xạ đúng, mà còn là bài toán CHỌN LỰA tối ưu dưới ràng buộc thực tế (ngân sách, thời gian, chất lượng).
