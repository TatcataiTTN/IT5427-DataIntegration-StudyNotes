# Quiz DI — Data Integration tổng quan & GAV/LAV/GLAV

Nguồn: `Quiz DI.pdf` (xuất từ Google Forms `docs.google.com/forms/d/e/1FAIpQLSc8DR6HH0XTUUxeUus5KZYwhUeiYF70hqe3RUVkmgSFwlfxkA`),
26 câu trắc nghiệm. **File PDF gốc KHÔNG có đáp án** (chỉ là bản in form trống) — toàn bộ đáp án và
giải thích bên dưới do tôi tự suy ra từ lý thuyết Data Integration đã xác minh (Lenzerini 2002,
Ullman 1997, Levy 2001, Doan-Halevy-Ives *Principles of Data Integration*, và đúng nội dung đã dựng
ở Module 01/02/03 của site), **không phải chép từ đáp án chính thức của giảng viên** — nhờ bạn/giảng
viên đối chiếu lại trước khi dùng để ôn thi thật. Những câu còn mơ hồ (đề gốc có lỗi/trùng ý ở các
phương án nhiễu) được ghi chú rõ trong phần đáp án, không che giấu.

Đa số câu dùng giao diện checkbox trên Google Form (có thể chọn nhiều đáp án); chỉ câu 18 dùng radio
button (chọn đúng 1). Tôi giữ đúng định dạng gốc — câu nào tôi xác định là multi-select thì ghi rõ
"(chọn nhiều)".

Làm hết phần câu hỏi trước, đáp án ở cuối file — đừng kéo xuống xem trước khi tự làm.

---

## Phần A — Khái niệm nền & dị biệt dữ liệu (câu 1-4, 19, 21-24)

**1.** (chọn nhiều) Mô tả nguồn dữ liệu trong hệ thống tích hợp dữ liệu dùng để làm gì?
a) Mô tả cách thức truy cập vào nguồn dữ liệu
b) Xác định nguồn nào liên quan đến truy vấn
c) Xóa các nguồn dữ liệu không cần thiết
d) Mô tả lược đồ tích hợp

**2.** Điều nào sau đây KHÔNG phải là vấn đề chính trong tích hợp dữ liệu?
a) Đồng nhất lược đồ
b) Xử lý truy vấn phân tán
c) Trực quan hóa dữ liệu tự động
d) Vấn đề về bảo mật và quyền riêng tư

**3.** Trong tích hợp dữ liệu, "lược đồ trung gian" đề cập đến
a) Lược đồ của nguồn dữ liệu lớn nhất
b) Một lược đồ tiêu chuẩn ánh xạ đến nhiều lược đồ nguồn khác nhau
c) Bản sao lưu của tất cả các cơ sở dữ liệu vật lý
d) Bản sao của kho dữ liệu mới nhất

**4.** Trường hợp nào sau đây là ví dụ về sự không đồng nhất ngữ nghĩa giữa hai nguồn dữ liệu?
a) Sử dụng tên bảng và thuộc tính khác nhau để biểu diễn cùng một đối tượng
b) Có lược đồ giống nhau nhưng dữ liệu khác nhau
c) Sử dụng 2 giá trị khác nhau để biểu diễn cùng 1 trạng thái của cùng 1 đối tượng
d) Hai nguồn chứa cùng dữ liệu và cùng lược đồ

**19.** Tính không đồng nhất về lược đồ (Schema heterogeneity) thể hiện:
a) Các hệ thống có thể cùng tên bảng và cùng thuộc tính
b) Các nguồn dữ liệu khác nhau về lược đồ
c) Giá trị dữ liệu trong các nguồn khác nhau có thể biểu diễn theo các cách khác nhau
d) Các nguồn sử dụng các công cụ lưu trữ khác nhau

**21.** (chọn nhiều) Yếu tố nào sau đây làm tích hợp dữ liệu trở nên khó?
a) Khác biệt nền tảng hệ thống
b) Khác biệt lược đồ và dữ liệu
c) Vấn đề bảo mật, riêng tư, hiệu năng
d) Kích thước dữ liệu

**22.** Trong kiến trúc kho dữ liệu (data warehouse), dữ liệu được tích hợp bằng cách nào?
a) Chỉ dùng dữ liệu XML
b) Biểu diễn bởi cùng 1 lược đồ
c) Đưa dữ liệu vào một kho dữ liệu vật lý chung
d) Không cần ánh xạ dữ liệu

**23.** Kiến trúc tích hợp dữ liệu ảo (virtual data integration) hoạt động như thế nào?
a) Luôn sao chép toàn bộ dữ liệu về một nơi
b) Để dữ liệu tại nguồn và truy cập khi có truy vấn
c) Không sử dụng truy vấn
d) Chỉ hoạt động với một nguồn dữ liệu duy nhất

**24.** Điểm chung của kho dữ liệu và tích hợp ảo dữ liệu là gì?
a) Cả hai đều gặp vấn đề dị biệt ngữ nghĩa
b) Cả hai đều có thể gặp vấn đề tính không đồng nhất ngữ nghĩa
c) Cả hai đều không cần lược đồ
d) Cả hai đều không cần nguồn dữ liệu

---

## Phần B — GAV (Global-as-View) (câu 5, 7, 10, 11, 14)

**5.** Trong cách ánh xạ lược đồ Global-as-View (GAV)
a) Mỗi quan hệ trong lược đồ trung gian được định nghĩa là một view từ các nguồn dữ liệu
b) Mỗi quan hệ nguồn là một view trên lược đồ trung gian
c) Định nghĩa ánh xạ phụ thuộc tập hợp các nguồn tham gia vào hệ thống
d) Lược đồ trung gian không cần đến dữ liệu nguồn

**7.** Điểm yếu lớn nhất của ánh xạ theo kiểu GAV là gì?
a) Không hỗ trợ ánh xạ từ nhiều nguồn
b) Không hỗ trợ các bảng có quan hệ khóa ngoại
c) Không thể xử lý dữ liệu phi cấu trúc
d) Không linh hoạt với thay đổi nguồn vì mỗi thay đổi cần cập nhật lại lược đồ trung gian

**10.** Trong GAV, mediated schema được định nghĩa như thế nào?
a) Là tập các view trên các nguồn dữ liệu
b) Là tập các nguồn dữ liệu trên mediated schema
c) Là tập các khóa ngoại
d) Là tập các file vật lý

**11.** Viết lại truy vấn trong GAV thường tương ứng với:
a) Query/View unfolding
b) Định vị các nguồn dữ liệu
c) Answering queries using views
d) Định vị các phân đoạn dữ liệu

**14.** Query unfolding có nghĩa là:
a) Mở rộng truy vấn bằng cách thay view bằng định nghĩa của view đó
b) Tối ưu truy vấn bằng cách rút gọn các bảng biểu diễn nó
c) Sử dụng khóa chính để kết nối
d) Viết lại truy vấn

---

## Phần C — LAV, GLAV & ngôn ngữ mô tả nguồn (câu 6, 8, 9, 12, 13)

**6.** Mô hình ánh xạ lược đồ nào định nghĩa mỗi nguồn như một view trên lược đồ trung gian?
a) Local-as-View (LAV)
b) Global-as-View (GAV)
c) Cả hai GAV và LAV
d) Không có đáp án nào đúng

**8.** Trong bối cảnh dữ liệu không đầy đủ, ánh xạ nào phù hợp nhất để bảo toàn tính linh hoạt và mở rộng các truy vấn có tính chất suy diễn?
a) GAV
b) LAV
c) GLAV
d) ETL Pipeline

**9.** (chọn nhiều) Ngôn ngữ mô tả nguồn dữ liệu cần đáp ứng
a) Hỗ trợ cập nhật nguồn
b) Linh hoạt trong biểu diễn quan hệ giữa các lược đồ thực tế
c) Chỉ dùng cho một nguồn dữ liệu
d) Hỗ trợ biểu diễn các nguồn có cấu trúc không nhất quán

**12.** Trong LAV, các nguồn dữ liệu được định nghĩa như thế nào?
a) Là khóa của mediated schema
b) Là view trên mediated schema
c) Dựa trên cách thực thi tối ưu truy vấn
d) Không phải cách nào

**13.** Trong GLAV, biểu thức mapping thường liên hệ giữa
a) Query trên mediated schema và query trên data sources
b) Người dùng và trình duyệt
c) Xóa các tuple trùng lặp
d) File và thư mục

---

## Phần D — Kiến trúc tích hợp, mediated schema, wrapper (câu 15-18, 20, 25-26)

**15.** Mediated Query là
a) câu truy vấn của người dùng hệ tích hợp
b) câu truy vấn trên các nguồn dữ liệu
c) câu truy vấn biểu diễn ánh xạ giữa lược đồ tích hợp và lược đồ nguồn
d) câu truy vấn phân tán trên các nguồn

**16.** Mục tiêu chính của tích hợp dữ liệu là gì?
a) Xóa bỏ tất cả các cơ sở dữ liệu cũ
b) Kết nối nhiều nguồn dữ liệu khác nhau dưới một lược đồ chung
c) Tăng khả năng xử lý truy vấn
d) Chuyển toàn bộ dữ liệu sang định dạng văn bản

**17.** Trong kiến trúc tích hợp dữ liệu, lược đồ tích hợp (mediated schema) có vai trò gì?
a) Là nơi lưu duy nhất toàn bộ dữ liệu vật lý
b) Là ánh xạ giữa các nguồn dữ liệu
c) Là giao diện/lược đồ chung để người dùng truy vấn nhiều nguồn dữ liệu
d) Là khóa chính của tất cả bảng

**18.** (radio — chọn 1) Ánh xạ ngữ nghĩa giữa các lược đồ (Semantic mappings) dùng để làm gì?
a) Nén dữ liệu
b) Mô tả quan hệ ngữ nghĩa giữa lược đồ chung và các nguồn dữ liệu
c) Xóa các bảng trùng nhau
d) Tạo định dạng chung biểu diễn yêu cầu truy vấn

**20.** Mục tiêu của tích hợp dữ liệu là cung cấp:
a) Một hệ thống phần mềm mới
b) Một ngôn ngữ lập trình mới
c) Một tổ chức lưu trữ mới
d) Một truy vấn thống nhất đến nhiều nguồn dữ liệu

**25.** Bộ trích lọc/trình bao bọc (Wrapper/Extractor) trong kiến trúc tích hợp dữ liệu có nhiệm vụ gì?
a) Thay thế cho nguồn dữ liệu
b) Gửi truy vấn đến nguồn dữ liệu và chuyển kết quả về mô hình dữ liệu chung
c) Xóa dữ liệu lỗi
d) Biểu diễn dữ liệu được trích chọn từ các nguồn sử dụng ngôn ngữ biểu diễn chung

**26.** (chọn nhiều) Nhược điểm của kho dữ liệu là
a) Truy vấn luôn làm gián đoạn nguồn dữ liệu
b) Dữ liệu thường không hoàn toàn cập nhật do làm mới offline/lazy refresh
c) Xử lý tính toán phức tạp
d) Không cần thiết kế lược đồ vật lý

---
---

# ĐÁP ÁN

## Phần A

**1.** **a, b** — mô tả nguồn trong Data Integration gồm 2 mặt kinh điển (Doan-Halevy-Ives): *content
description* (nội dung/độ phủ, giúp xác định nguồn liên quan tới truy vấn → b) và *capability
description* (cách truy cập, binding pattern → a). (c) là việc quản trị hệ thống, không phải "mô
tả nguồn". (d) mô tả lược đồ trung gian, không phải lược đồ nguồn — nhầm vai trò 2 phía.

**2.** **c** — Trực quan hóa dữ liệu tự động là việc của tầng BI/phân tích, không phải vấn đề cốt lõi
của Data Integration (vốn xoay quanh schema, truy vấn phân tán, bảo mật/quyền riêng tư — đúng 3 ý
còn lại).

**3.** **b** — định nghĩa chuẩn: mediated/global schema là 1 lược đồ duy nhất mà nhiều lược đồ nguồn
khác nhau được ánh xạ vào, KHÔNG phải bản sao vật lý của dữ liệu (loại c, d) hay lược đồ của 1 nguồn
cụ thể nào (loại a).

**4.** **c** — "Sử dụng 2 giá trị khác nhau biểu diễn cùng 1 trạng thái của cùng 1 đối tượng" là dị
biệt ở MỨC GIÁ TRỊ/NGỮ NGHĨA (ví dụ: mã giới tính 'M/F' vs '1/2' cho cùng ý nghĩa) — đúng tinh thần
Semantic/Instance heterogeneity. (a) là dị biệt tên bảng/thuộc tính — thuộc tầng Structural/Schema
(đặt tên khác nhau), không phải ngữ nghĩa giá trị. (b), (d) không mô tả một dạng dị biệt cụ thể nào.

**19.** **b** — đúng định nghĩa "schema heterogeneity" theo phân tầng dị biệt dữ liệu đã dựng ở Module
01 của site (7 tầng: System/Platform, Syntactic/Format, Structural/Schema, Semantic/Conceptual,
Instance/Identity, Temporal/Process, Quality/Provenance). (a) mô tả trường hợp KHÔNG có dị biệt. (c)
là Instance/Semantic heterogeneity (mức giá trị), (d) là System/Platform heterogeneity (mức công cụ
lưu trữ) — cả 2 đều là tầng dị biệt KHÁC, không phải "schema".

**21.** **a, b, c, d** — cả 4 đều là nguyên nhân thật khiến tích hợp dữ liệu khó, khớp đúng cả 7 tầng
dị biệt (a, b) lẫn thách thức Big Data Integration (c: Quality/Provenance/Governance; d: Volume) đã
học. Không có đáp án nào sai ở đây.

**22.** **c** — Data Warehousing = ETL copy dữ liệu định kỳ về 1 kho vật lý tập trung (đúng bảng so
sánh Virtual vs Warehouse ở Module 01). (a) sai vì không giới hạn chỉ XML, (b) mô tả 1 phần nhưng
chưa đủ (bản chất là VẬT LÝ hoá, không chỉ "cùng lược đồ"), (d) sai hiển nhiên — warehouse luôn cần
ánh xạ dữ liệu nguồn sang lược đồ kho.

**23.** **b** — Virtual Integration: dữ liệu ở nguyên nguồn, chỉ truy cập qua wrapper tại thời điểm có
truy vấn — đối lập hẳn với (a) (đó là warehouse). (c), (d) sai hiển nhiên.

**24.** **b** *(lưu ý lỗi đề gốc)* — (a) và (b) trong file gốc gần như trùng nghĩa nhau ("dị biệt ngữ
nghĩa" ≈ "tính không đồng nhất ngữ nghĩa") — khả năng cao là lỗi soạn đề (2 phương án nhiễu bị trùng
ý thay vì khác nhau). Về nội dung: cả Data Warehouse lẫn Virtual Integration đều phải tích hợp nhiều
nguồn có lược đồ khác nhau, nên cả hai đều CÓ THỂ gặp vấn đề không đồng nhất ngữ nghĩa — chọn (b)
nhưng (a) cũng hợp lý nếu đề thật sự phân biệt được 2 ý. Nên hỏi lại giảng viên để biết ý định thật
của 2 phương án gần giống nhau này.

## Phần B

**5.** **a** — đúng định nghĩa GAV: mỗi quan hệ mediated schema = 1 view trên các nguồn, $G \supseteq
Q(\text{sources})$.

**7.** **d** — nhược điểm cốt lõi của GAV: mediated schema "cứng", mỗi lần thêm/bớt nguồn phải viết
lại định nghĩa view GAV tương ứng (ngược với LAV, nơi thêm nguồn chỉ cần thêm 1 định nghĩa LAV mới
mà không đụng tới mediated schema).

**10.** **a** — mediated schema trong GAV = tập các view TRÊN các nguồn (không phải ngược lại).

**11.** **a** — Reformulation trong GAV chính là Query/View Unfolding: thay từng subgoal bằng định
nghĩa GAV tương ứng, lặp tới hết. (c) "Answering queries using views" là bài toán reformulation của
LAV (khó hơn), không phải GAV.

**14.** **a** — định nghĩa chính xác của unfolding: mở rộng truy vấn bằng cách THAY view bằng ĐỊNH
NGHĨA của view đó (substitute view reference → view definition). (d) "viết lại truy vấn" quá chung
chung, không phải định nghĩa kỹ thuật cụ thể của riêng "unfolding".

## Phần C

**6.** **a** — định nghĩa LAV: mỗi NGUỒN được định nghĩa như 1 view trên mediated schema (ngược
hướng với GAV).

**8.** **c** — GLAV tổng quát hoá cả GAV lẫn LAV bằng Tuple Generating Dependency (TGD) với biến tồn
tại (∃), biểu diễn được thông tin KHÔNG ĐẦY ĐỦ tốt hơn 2 mô hình kia, đồng thời giữ được tính linh
hoạt khi mở rộng nguồn (thừa hưởng ưu điểm LAV) — đúng tinh thần Lenzerini (2002) khi giới thiệu GLAV
như mô hình tổng quát nhất cho các bài toán ánh xạ phức tạp/thiếu thông tin.

**9.** **b, d** — ngôn ngữ mô tả nguồn cần LINH HOẠT biểu diễn quan hệ lược đồ thực tế (b) và biểu
diễn được các nguồn có cấu trúc KHÔNG nhất quán (d) — đúng động lực ra đời GLAV/các ngôn ngữ mapping
tổng quát (Ullman 1997, Levy 2001). (c) sai hiển nhiên (ngôn ngữ mô tả phải dùng được cho NHIỀU
nguồn). (a) "hỗ trợ cập nhật nguồn" là mối quan tâm của view maintenance/hệ thống vận hành, không
phải yêu cầu cốt lõi của riêng NGÔN NGỮ mô tả — không chọn.

**12.** **b** — định nghĩa LAV: nguồn = view trên mediated schema (đối xứng ngược với GAV ở câu 10).

**13.** **a** — GLAV: $Q^S(\bar X) \subseteq Q^G(\bar X)$ — biểu thức mapping liên hệ 1 query trên
schema NGUỒN với 1 query trên schema TOÀN CỤC (mediated).

## Phần D

**15.** **c** — theo đúng quy trình sinh Mediation Query đã học ở Module 03 (Partial Mapping → Join
Graph → Mediation Query): mediation query LÀ câu truy vấn thể hiện ánh xạ giữa lược đồ tích hợp và
lược đồ nguồn, không phải truy vấn "thô" của người dùng (a) hay truy vấn chạy trực tiếp trên nguồn
(b, d).

**16.** **b** — mục tiêu cốt lõi, kinh điển của Data Integration.

**17.** **c** — mediated schema = giao diện/lược đồ DUY NHẤT người dùng nhìn thấy để truy vấn nhiều
nguồn — không phải kho vật lý (a, đó là warehouse) hay bản thân phép ánh xạ (b, ánh xạ là thứ KẾT NỐI
mediated schema với nguồn, không phải chính nó).

**18.** **b** — semantic mapping mô tả quan hệ Ý NGHĨA giữa lược đồ chung và các nguồn (vd thuộc tính
nào ở nguồn tương ứng với thuộc tính nào ở mediated schema, dưới điều kiện gì) — không liên quan nén
dữ liệu, xoá trùng lặp, hay định dạng truy vấn.

**20.** **d** — diễn đạt lại mục tiêu cốt lõi giống câu 16, dưới góc nhìn "truy vấn thống nhất".

**25.** **b** — đúng vai trò wrapper đã định nghĩa ở Module 01: nhận truy vấn, GỬI xuống nguồn thật
(gọi API/chạy SQL cục bộ/parse HTML...), rồi CHUYỂN kết quả về đúng mô hình dữ liệu chung. (d) chỉ
nói một nửa vai trò (biểu diễn dữ liệu), thiếu phần "gửi truy vấn" — (b) đầy đủ và khớp định nghĩa
hơn.

**26.** **b** — nhược điểm kinh điển của data warehouse: dữ liệu có ĐỘ TRỄ do ETL/làm mới theo chu kỳ
(lazy refresh), không phải luôn mới nhất như virtual integration. (a) thực ra mô tả nhược điểm của
VIRTUAL integration (mỗi truy vấn làm phiền nguồn gốc), không phải warehouse — warehouse KHÔNG làm
gián đoạn nguồn tại thời điểm truy vấn vì dữ liệu đã copy sẵn. (c) cũng bị đảo ngược — warehouse vốn
dễ tính toán HƠN virtual (đã vật lý hoá sẵn), không phải nhược điểm của nó. (d) sai hiển nhiên.
