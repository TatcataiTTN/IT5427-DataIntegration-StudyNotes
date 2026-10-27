# Bài tập — Edit Distance/TF-IDF & Schema Mapping (GAV/LAV)

Nguồn: `BT.pdf` (6 slide, in ngày 13/10/2026). Slide 2 có kèm link/QR Google Form Quizz
(`forms.gle/kbbwLG8autntPRms7`) — đã kiểm tra: form này trỏ tới **đúng cùng 1 form** đã chuyển thành
`Quiz-DI-DataIntegration-GAV-LAV-GLAV.md` (26 câu, cùng ID form `1FAIpQLSc8DR6HH0XTUUxeUus5KZYwhUeiYF70hqe3RUVkmgSFwlfxkA`
— đối chiếu câu đầu/câu cuối khớp 100%), nên **không tạo lại** — dùng file đó. Nội dung MỚI thật sự
trong `BT.pdf` là 2 bài tập tự luận ở slide 3-6, chuyển thành .md bên dưới.

Làm hết phần câu hỏi trước, đáp án ở cuối file.

---

## Bài 1 — Edit Distance & TF/IDF

Cho hai xâu: **x = hnoi** và **y = hanoi**. Tính độ tương đồng giữa 2 xâu theo:

**a.** Khoảng cách chỉnh sửa — Edit Distance (và suy ra similarity score).

**b.** TF/IDF (dùng đúng công thức đã học: tf(t,d) = số lần xuất hiện của term t trong d; idf(t) = N/N_t,
N = số "văn bản" trong tập hợp đang so sánh, N_t = số văn bản chứa t; v_d(t) = tf(t,d)·idf(t); điểm
tương đồng = cosine giữa 2 vector — đúng slide "TF/IDF Similarity Score", trang 12 của
`3_RecordLinkage_ER.pdf`). Lưu ý: trong ví dụ gốc của slide (x=aab, y=ac, z=a), "term" là **từng ký
tự**, không phải từng từ — áp dụng đúng quy ước đó cho "hnoi"/"hanoi" (2 xâu không có khoảng trắng
nên tách theo ký tự là cách duy nhất hợp lý).

---

## Bài 2 — Schema Matching, GAV, LAV

Cho lược đồ quan hệ của 3 nguồn dữ liệu:

- **S1**: {ProductDescription, BusinessUnit, Rating, Price} — sản phẩm của nhà cung cấp A
- **S2**: {ProductName, Unit} — sản phẩm của nhà cung cấp B
- **S3**: {ProductName, Rating, Price} — đánh giá của TẤT CẢ sản phẩm (cả A lẫn B)

Giả thiết **ProductDescription** và **ProductName** có cùng bộ giá trị (cùng domain, chỉ khác tên
thuộc tính).

Cho lược đồ trung gian **S**: {ProductDescription, BusinessUnit, Rating, Price}

**a.** Dùng kỹ thuật đối sánh dựa trên tên (name-based matching), mô tả ánh xạ giữa 3 lược đồ nguồn
và S.

**b.** Định nghĩa ánh xạ Global-as-View (GAV) giữa các nguồn và S.

**c.** Định nghĩa ánh xạ Local-as-View (LAV) giữa các nguồn và S.

---
---

# ĐÁP ÁN

## Bài 1

**a. Edit Distance.** Tính bằng DP chuẩn (chèn/xoá/thay thế, mỗi phép giá 1), bảng đầy đủ:

| | "" | h | a | n | o | i |
|---|---|---|---|---|---|---|
| **""** | 0 | 1 | 2 | 3 | 4 | 5 |
| **h** | 1 | 0 | 1 | 2 | 3 | 4 |
| **n** | 2 | 1 | 1 | 1 | 2 | 3 |
| **o** | 3 | 2 | 2 | 1 | 1 | 2 |
| **i** | 4 | 3 | 3 | 2 | 1 | **1** |

$d(x,y) = 1$ — chỉ cần **1 phép chèn** ký tự 'a' vào sau "h": h**n**oi → h**a**noi? (kiểm tra lại:
hnoi → chèn 'a' sau vị trí 1 → h-a-n-o-i = hanoi ✓, đúng 1 thao tác).

Similarity: $s(x,y) = 1 - \dfrac{d(x,y)}{\max(|x|,|y|)} = 1 - \dfrac{1}{5} = \dfrac{4}{5} = 0.8$

**b. TF/IDF.** Tách "term" theo từng KÝ TỰ (đúng quy ước ví dụ gốc slide, vì "hnoi"/"hanoi" không có
khoảng trắng để tách từ):

$B_x = \{h,n,o,i\}$, $B_y = \{h,a,n,o,i\}$ — tập hợp term $T = \{h,n,o,i,a\}$, $N=2$ (2 xâu đang so).

| term t | $N_t$ (số xâu chứa t) | $idf(t)=N/N_t$ | $tf(t,x)$ | $tf(t,y)$ |
|---|---|---|---|---|
| h | 2 | 1 | 1 | 1 |
| n | 2 | 1 | 1 | 1 |
| o | 2 | 1 | 1 | 1 |
| i | 2 | 1 | 1 | 1 |
| a | 1 | 2 | 0 | 1 |

$v_x(t) = tf(t,x)\cdot idf(t)$: $v_x = [h{:}1, n{:}1, o{:}1, i{:}1, a{:}0]$
$v_y(t) = tf(t,y)\cdot idf(t)$: $v_y = [h{:}1, n{:}1, o{:}1, i{:}1, a{:}2]$

$$s(x,y) = \frac{\sum_t v_x(t)\cdot v_y(t)}{\sqrt{\sum_t v_x(t)^2}\cdot\sqrt{\sum_t v_y(t)^2}}
= \frac{1{+}1{+}1{+}1{+}0}{\sqrt{1{+}1{+}1{+}1{+}0}\cdot\sqrt{1{+}1{+}1{+}1{+}4}}
= \frac{4}{2\cdot2\sqrt2} = \frac{1}{\sqrt2} \approx 0.707$$

**Nhận xét sư phạm** (đáng nêu khi chấm/giảng lại): nếu áp dụng TF/IDF ở MỨC TỪ thay vì mức ký tự
("hnoi" và "hanoi" mỗi xâu là đúng 1 "từ" duy nhất), thì 2 "từ" này khác nhau tuyệt đối về chuỗi ký
tự → $T=\{hnoi, hanoi\}$, mỗi term chỉ xuất hiện ở 1 trong 2 văn bản → tích vô hướng $=0$ → similarity
$=0$, một kết quả VÔ NGHĨA (2 xâu rõ ràng giống nhau 80% theo Edit Distance ở câu a). Đây chính là lý
do bài tập phải tách theo ký tự — minh hoạ đúng giới hạn đã học: TF/IDF mức từ chỉ có ý nghĩa khi
chuỗi có NHIỀU token (ví dụ "Apple Corporation, CA" ở slide gốc), còn với lỗi chính tả bên trong 1 từ
duy nhất phải tách ở mức ký tự (hoặc q-gram) mới bắt được độ tương đồng thật.

## Bài 2

> **Cập nhật 2026-10-27:** đã đối chiếu với `BT_chữa.pdf` (bản chữa có đáp án chính thức của giảng
> viên, 9 trang, gồm cả ảnh chụp bài giải tay). Câu 1 (Edit Distance + TF/IDF) khớp **CHÍNH XÁC 100%**
> với lời giải tôi tự tính ở trên (Edit Distance=1/sim=0.8, TF/IDF=1/√2) — không cần sửa gì. Câu 2
> (Schema Matching/GAV/LAV) đáp án chính thức dùng cách trình bày **lỏng/thực hành hơn** so với bản
> tôi tự suy diễn theo lý thuyết chặt chẽ ban đầu — đã thay bằng đáp án chính thức bên dưới, có ghi
> chú rõ chỗ khác biệt để bạn hiểu CẢ 2 cách nhìn.

**a. Ánh xạ dựa trên tên (theo đáp án chữa chính thức).** Đề bài đã cho sẵn giả thiết
"ProductDescription và ProductName có cùng bộ giá trị" — đáp án chính thức dùng giả thiết này NGAY
trong bước đối sánh tên (không chỉ đối sánh chuỗi ký tự tuyệt đối), tức "ánh xạ dựa trên tên" ở đây
được hiểu rộng là "ánh xạ dựa trên tên **và** thông tin ngữ nghĩa/domain đã biết trước":

| Thuộc tính của S (mediated) | Khớp với (các nguồn) |
|---|---|
| ProductDescription | S1.ProductDescription, S2.ProductName, S3.ProductName |
| BusinessUnit | S1.BusinessUnit, S2.Unit |
| Rating | S1.Rating, S3.Rating |
| Price | S1.Price, S3.Price |

**Ghi chú về sự khác biệt với lý thuyết chặt chẽ** (để hiểu sâu hơn, không phải để "sửa" đáp án
chính thức): nếu áp dụng **name-based matching THUẦN TUÝ** (chỉ so khớp CHUỖI TÊN THUỘC TÍNH, không
dùng thêm thông tin nào khác) thì `ProductName` sẽ KHÔNG khớp được với `ProductDescription` (2 chuỗi
khác hẳn nhau) — đây chính là **false negative** kinh điển của name-matching thuần, và là lý do giáo
trình (Doan-Halevy-Ives) luôn nói name-matching phải kết hợp thêm **instance-based matching** (so bộ
giá trị) mới đủ mạnh. Đáp án chính thức ở đây thực chất đã NGẦM dùng cả 2 kỹ thuật cùng lúc (tên +
giả thiết domain đã cho), đúng với cách làm thực tế trong lớp — không phải lỗi, chỉ là thuật ngữ
"dựa trên tên" được dùng theo nghĩa rộng hơn định nghĩa sách giáo khoa.

**b. GAV (Global-as-View) — theo đáp án chữa chính thức.** Mediated schema S có thể được "phủ" bằng
3 cách kết hợp nguồn khác nhau (ký hiệu $\supseteq$ nghĩa là "S bao phủ được bằng"):

$$S\{PD,BU,R,P\} \;\supseteq\; S1\{PD,BU,R,P\}$$
$$S\{PD,BU,R,P\} \;\supseteq\; S2\{PN,U\} \Join S3\{PN,R,P\}$$

Bản chụp bài giải tay còn có 1 dòng thứ 3 ghi `S1:{ProductName, Unit}, S3:{ProductName, Rating,
Price}` — **đây gần như chắc chắn là lỗi đánh máy/chép nhầm trong chính slide chữa** (S1 không hề có
thuộc tính `ProductName`/`Unit` theo đề bài, trong khi dòng này giống hệt dòng thứ 2 chỉ khác mỗi
nhãn nguồn `S1` thay vì `S2`) — có ghi chú tay màu đỏ "ProductDescription" cạnh đó nhưng không đủ rõ
để khẳng định ý sửa là gì; nên hỏi lại giảng viên nếu cần dùng ý này để thi.

Về bản chất, 2 dòng đầu diễn đạt đúng 2 cách "phủ" S: **toàn bộ S1** là 1 view hợp lệ của S (S1 đã đủ
4 cột), và **JOIN(S2, S3) theo khoá ProductName** là 1 view hợp lệ khác của S (ghép đủ 4 cột từ 2
nguồn thiếu cột). Viết tương đương bằng SQL cho dễ hình dung (cách viết này KHÔNG có trong bài chữa,
là tôi thêm vào để làm rõ ý — nội dung tương đương, chỉ khác hình thức biểu diễn):
```sql
CREATE VIEW S AS
SELECT ProductDescription AS PD, BusinessUnit AS BU, Rating AS R, Price AS P FROM S1
UNION
SELECT S2.ProductName AS PD, S2.Unit AS BU, S3.Rating AS R, S3.Price AS P
FROM S2 JOIN S3 ON S2.ProductName = S3.ProductName;
```
Đúng tinh thần GAV: mediated schema được định nghĩa THẲNG bằng truy vấn cụ thể trên các nguồn — thêm/
bớt nguồn sau này phải SỬA LẠI chính định nghĩa này (đúng nhược điểm GAV đã nêu ở
`Quiz-DI-DataIntegration-GAV-LAV-GLAV.md` câu 7).

**c. LAV (Local-as-View) — theo đáp án chữa chính thức.** Mỗi NGUỒN được viết là tập con (⊆) của
mediated schema S, không chia nhỏ theo cột:

$$S1\{PD,BU,R,P\} \;\subseteq\; S\{PD,BU,R,P\}$$
$$S2\{PN,U\} \;\subseteq\; S\{PD,BU,R,P\}$$
$$S3\{PN,R,P\} \;\subseteq\; S\{PD,BU,R,P\}$$

**Ghi chú làm rõ thêm** (bổ sung, không có trong bài chữa): viết `S2 ⊆ S` như trên là cách viết
RÚT GỌN — về mặt hình thức, vì S2 chỉ có 2/4 cột của S, phát biểu chính xác hơn phải dùng PHÉP CHIẾU
(projection): $S2\{PN,U\} \subseteq \pi_{PD,BU}(S)$ và $S3\{PN,R,P\} \subseteq \pi_{PD,R,P}(S)$ —
tức "S2 là 1 tập con của S SAU KHI đã chiếu S xuống đúng 2 cột tương ứng". Bài chữa bỏ qua bước viết
phép chiếu này (chấp nhận được ở mức làm tay/thi), nhưng hiểu được vì sao cần phép chiếu sẽ giúp bạn
viết đúng mapping LAV tổng quát hơn (khi S2, S3 không đơn giản là "trùng khớp cột" mà còn lọc/biến
đổi dữ liệu). Đúng tinh thần LAV: nguồn mới thêm vào chỉ cần 1 định nghĩa LAV mới, KHÔNG phải sửa S —
khác hẳn GAV ở câu b.
