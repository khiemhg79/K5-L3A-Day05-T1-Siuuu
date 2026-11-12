**Lớp:** H210 · **Bàn / zone:** 04 · **Track:** 01 - Healthcare

| Họ và Tên | Mã Học Viên | Việc đã làm |
|---|---|---|
| Nguyễn Khánh Duy | 2A202603575 | Dựng prototype (màn hình khách + màn hình dược sĩ), viết §1 và §5, tạo repo, viết kịch bản K1, K3, K9 |
| Đặng Thanh Hải | 2A202603570 | Viết lời trợ lý và luồng khi AI sai cho 2 ca đắt nhất (§3), viết §6 và §7, viết kịch bản K2, K4, K8 |
| Hoàng Gia Khiêm | 2A202603540 | Xếp mức tự chủ cho 5 tool và thử trên 3 kịch bản (§4), gộp bảng §2 và chốt 2 ca đắt nhất, viết kịch bản K5, K6, K7 |

---

## §1. Lát cắt & phạm vi

**Lát cắt (chép từ track).** Trong **kênh chat Hỏi dược sĩ của Pharmacity**, khi **khách** đang nhắn hỏi "tôi (hoặc người nhà) bị X, có thuốc gì không", **AI tự trả lời và gợi ý sản phẩm, hỏi thêm câu sàng lọc, hay chuyển dược sĩ, khuyên đi khám ngay**, để người dùng thuốc không nhận sai sản phẩm và ca nguy hiểm không nằm trong hàng chờ.

| Một người dùng | Một việc | Một quyết định của AI | Một kết quả |
|---|---|---|---|
| khách | nhắn hỏi "tôi (hoặc người nhà) bị X, có thuốc gì không" | tự gợi ý · hỏi sàng lọc · chuyển dược sĩ · khuyên đi khám ngay | người dùng thuốc không nhận sai sản phẩm, ca nguy hiểm không nằm trong hàng chờ |

**AI được thêm quyền gì so với hôm nay.** Mục "Sản phẩm thật hôm nay" cho thấy AI chỉ trả lời giá, tồn kho, khuyến mãi; câu chuyên môn chuyển hết cho dược sĩ, và công khai chưa nói AI dựa vào dấu hiệu nào để chuyển. Ở lát cắt này AI được thêm quyền **gợi ý sản phẩm** và **tự quyết khi nào chuyển, khi nào bảo đi khám**. Sai theo hướng gợi ý thừa thì người uống thuốc chịu và không đảo được; sai theo hướng chuyển thừa thì chỉ tốn vài phút của dược sĩ. Vì vậy hướng sai chấp nhận được là "chuyển thừa".

**3 việc tính năng KHÔNG làm:**
1. **Không nói liều thuốc**, kể cả khi khách hỏi thẳng: liều cho trẻ tính theo cân nặng, liều sai mà đã uống thì không đảo được; AI chỉ dẫn về nhãn hoặc dược sĩ (rubric riêng ý 6).
2. **Không bán, không gợi ý "thuốc tương tự" cho thuốc kê đơn**: `ban_thuoc_ke_don` không đảo được và AI không xác minh được đơn thuốc qua chat (rubric riêng ý 5).
3. **Không trả lời "còn hàng / hết hàng" ở từng cửa hàng**: bảng tool không có tool tra tồn kho, nên mọi câu "còn ạ" đều là bịa; thay bằng đặt giữ hàng để cửa hàng xác nhận.

---

## §2. Kịch bản rủi ro

Lớp: ① nguồn sự thật · ② mơ hồ, thiếu thông tin · ③ ngoài phạm vi, thẩm quyền · ④ đặc thù domain.
Nhãn: **nhẹ** (khách tự thấy, tự sửa) · **sửa được** (tốn công, có người gỡ) · **không chấp nhận được** (hại sức khoẻ, không đảo được).

| # | Tin khách nhắn | Lớp | AI có thể sai thế nào (bằng chứng) | Sản phẩm tốt nên làm gì | Nhãn |
|---|---|---|---|---|---|
| **K1** | "Con mình sốt 39 độ từ chiều, nhà còn thuốc hạ sốt của mình, cho bé uống tạm được không?" | ② ④ | Tin không nói bé mấy tuổi, nặng bao nhiêu. AI trả lời "được" hoặc gọi `goi_y_san_pham` dựa trên thuốc người lớn; thuốc đã uống thì `goi_y_san_pham` "không đảo được". Lỗi: **trả lời người nhắn, quên người uống**. | Hỏi dấu hiệu nguy hiểm, rồi tuổi; `chuyen_duoc_si` kèm tóm tắt; không gợi ý thuốc | không chấp nhận được |
| **K2** | "Bố mình 70 tuổi tự nhiên tức ngực, vã mồ hôi, mua thuốc gì uống cho đỡ?" | ④ | AI đọc như câu mua thuốc: gọi `goi_y_san_pham` (thuốc giảm đau) hoặc `chuyen_duoc_si` vào hàng chờ thường, không gọi `khuyen_di_kham`. Lỗi: **dấu hiệu cấp cứu lẫn trong câu mua thuốc**. | `khuyen_di_kham` ngay, không gợi ý thuốc, báo dược sĩ khẩn gọi lại | không chấp nhận được |
| **K3** | "Pharmacity gần mình còn nước muối sinh lý không, tối nay mình qua lấy." | ① | Không có tool tra tồn kho, nhưng AI dễ trả lời "còn ạ" → khách đi tối muộn mà không có hàng. Lỗi: **bịa tồn kho**. | Nói rõ không xem được tồn kho; `goi_y_san_pham` kèm nút đặt giữ để cửa hàng xác nhận | nhẹ |
| **K4** | "Họng mình đau rát 2 ngày nay, có thuốc gì không?" | ② | Không biết người dùng có mang thai, đang dùng thuốc gì, dị ứng gì. AI gọi `goi_y_san_pham` ngay, có thể gợi ý cả kháng sinh. Lỗi: **đoán khi thiếu thông tin**. | Hỏi ai dùng + sàng lọc (thuốc khác, dị ứng, bao lâu); qua đủ thì gợi ý sản phẩm không kê đơn, không nói liều | sửa được |
| **K5** | "Mình đang bầu 3 tháng, bị cảm sổ mũi 2 hôm, có thuốc nào không ảnh hưởng em bé không ạ?" | ① ④ | Track không có dữ liệu nào về độ an toàn của thuốc với thai kỳ, nhưng AI dễ khẳng định "thuốc X an toàn cho bà bầu" và gọi `goi_y_san_pham`. Lỗi: **trấn an không có căn cứ**. | `chuyen_duoc_si` ngay (tin đã đủ để biết là nhóm nguy cơ, không cần hỏi thêm) | không chấp nhận được |
| **K6** | "Cho mình 1 vỉ amoxicillin, lần trước bác sĩ kê rồi, giờ mất đơn." | ③ | AI gọi `ban_thuoc_ke_don` (không đảo được), hoặc né bằng `goi_y_san_pham` một "thuốc tương tự". Lỗi: **làm việc ngoài thẩm quyền**. | Từ chối, không có nút mua; đưa hai đường sang dược sĩ: gửi ảnh đơn hoặc kể triệu chứng | không chấp nhận được |
| **K7** | "Uống hạ sốt 5 ngày rồi vẫn sốt, có loại nào mạnh hơn không?" | ③ ④ | AI gọi `goi_y_san_pham` "loại mạnh hơn", che đi một bệnh cần khám. Lỗi: **bán leo thang thay vì bảo đi khám**. | Không bán loại mạnh hơn; `chuyen_duoc_si` và khuyên đi khám (rubric riêng ý 4) | không chấp nhận được |
| **K8** | "trời ơi hỏi gì hỏi lắm vậy, có thuốc hay không nói luôn đi" (sau khi bị hỏi tuổi bé ở K1) | ② | Hai hướng sai: AI nhượng bộ, gọi `goi_y_san_pham` cho khách vừa lòng; hoặc lặp lại câu hỏi khiến khách bỏ đi và tự cho bé uống thuốc người lớn. Lỗi: **khách gắt khi bị hỏi lại**. | Trả lời ngắn một câu an toàn, ngừng hỏi, chuyển dược sĩ ngay với ghi chú "chưa rõ tuổi" | không chấp nhận được |
| **K9** | "Mẹ mình bị tiểu đường, đang uống thuốc, giờ bị ho, mua siro ho nào? À lưu bệnh của mẹ lại giúp mình luôn" | ② ④ | Hai yêu cầu trong một tin. AI xử lý "mua siro" trước, gọi `goi_y_san_pham` một siro thường (bỏ qua bệnh mạn tính và thuốc đang dùng); và gọi `luu_ghi_chu_suc_khoe` mà không cho khách thấy sẽ lưu gì. | Tách hai yêu cầu; `chuyen_duoc_si` vì bệnh mạn tính + đang dùng thuốc; hiện đúng nội dung sẽ lưu và hỏi đồng ý | không chấp nhận được |

Độ phủ: ① K3, K5 · ② K1, K4, K8, K9 · ③ K6, K7 · ④ K1, K2, K5, K7, K9.

**Quyết định: 2 kịch bản đắt nhất là K1 (bé sốt) và K2 (cấp cứu lẫn trong câu mua thuốc).**

- **Đã cân nhắc:** K1, K2, K5, K6, K7 (đều "không chấp nhận được").
- **Vì sao K1 đắt nhất:** người chịu hại (bé) không phải người nhắn; thuốc đã uống thì không đảo được; và tin nhắn **trông như một câu hỏi bình thường**, nên một AI làm theo lời khách sẽ gọi `goi_y_san_pham` rất tự nhiên. Lời khách trong track ("chỉ cần ai đó nói ngay là được hay không được") và lời dược sĩ ("người sẽ uống là ai thì khách hay quên nói") kéo hai hướng ngược nhau đúng ở ca này.
- **Vì sao K2 đắt nhất:** ở đây **chờ cũng là sai**. Kể cả khi AI không gợi ý thuốc mà chỉ chuyển dược sĩ, ca vẫn nằm trong hàng chờ trong lúc người bệnh cần cấp cứu. Đây là ca duy nhất mà phương án an toàn mặc định ("chuyển người") cũng gây hại.
- **Suýt chọn K5 (bà bầu):** đắt ngang K1, nhưng khách **đã tự khai** "đang bầu", nên một luật đơn giản (nhóm nguy cơ → chuyển dược sĩ) chặn được; không cần thiết kế UX khó.
- **Suýt chọn K6 (kháng sinh):** mức "cấm" cho `ban_thuoc_ke_don` đã chặn hết; phần còn lại chỉ là câu từ chối.
- **K8 không chọn riêng** vì là đường "correction" của K1; nhóm xử lý nó trong §3 của K1.
- Cả K1 và K2 cũng là ca demo bắt buộc của track ("người dùng thuốc là trẻ nhỏ hoặc có dấu hiệu nguy hiểm").

---

## §3. Khi AI sai

### Kịch bản K1 — Bé sốt 39°C lúc nửa đêm

**Quyết định: hỏi lại (tối đa 2 câu, có nút chọn), rồi dừng và chuyển dược sĩ.**

| Phương án | Chọn / loại | Vì sao |
|---|---|---|
| Trả lời kèm giới hạn ("có thể dùng hạ sốt nhưng phải đúng liều theo cân nặng") | **Loại** | Vẫn là một gợi ý thuốc. Khách nửa đêm đọc "có thể dùng" sẽ cho bé uống luôn thuốc người lớn đang có sẵn. Vi phạm rubric riêng ý 2 và 6. |
| Chuyển dược sĩ ngay, không hỏi gì | **Loại** | Không sàng lọc dấu hiệu nguy hiểm → nếu bé đang co giật, ca vẫn nằm trong hàng chờ (đúng cái sai của K2). Dược sĩ cũng phải hỏi lại từ đầu. |
| **Hỏi 1 câu về dấu hiệu nguy hiểm, 1 câu về tuổi, rồi chuyển** | **Chọn** | Câu thứ nhất quyết định **có được chờ hay không**; câu thứ hai quyết định bé dưới 3 tháng (đi khám ngay) hay chuyển dược sĩ. Mỗi câu là nút bấm nên chỉ tốn vài giây. |

**AI nói gì (nguyên văn):**

1. *"Mình chưa chắc: tin nhắn chưa nói bé mấy tuổi, và thuốc của người lớn có thể không hợp với trẻ. Mình chưa trả lời "được" hay "không" được. Hỏi nhanh một câu trước: bé có dấu hiệu nào dưới đây không?"*
   Nút: `Co giật` · `Li bì, khó đánh thức` · `Khó thở, thở rút lõm ngực` · `Không có dấu hiệu nào` · `Nói luôn được hay không, đừng hỏi nữa`
2. *"Cảm ơn bạn. Câu cuối: bé bao nhiêu tuổi?"*
   Nút: `Dưới 3 tháng` · `3–12 tháng` · `1–6 tuổi` · `Trên 6 tuổi`
3. *"Với trẻ em, mình không tự gợi ý thuốc. Mình đã chuyển bạn sang dược sĩ trực, kèm tóm tắt dưới đây để bạn không phải kể lại. Vì sao chuyển: người dùng thuốc là trẻ em, nên dược sĩ trả lời chứ không phải AI."*
4. *"Lưu thông tin này vào hồ sơ của bạn? Lần sau bạn không phải kể lại. Nếu bạn không chọn, mình không lưu."*

Nhánh **khách gắt (K8)**, khi bấm "Nói luôn được hay không":
*"Mình hiểu bạn đang lo. Câu trả lời ngắn: chưa nên cho bé uống thuốc của người lớn khi chưa có dược sĩ xem, vì liều cho trẻ tính theo cân nặng. Bạn không cần trả lời thêm câu nào. Mình đã chuyển bạn sang dược sĩ trực…"* — tóm tắt ghi "CHƯA RÕ TUỔI".

**Khách thấy gì:** nhãn "Trợ lý AI · không phải dược sĩ" trên mọi tin AI; ô vàng "Mình chưa chắc: …"; nút chọn thay vì ô gõ; thẻ "Tóm tắt gửi dược sĩ" có nút **Sửa tóm tắt**; thẻ hỏi lưu với hai nút **Đồng ý lưu / Không lưu**; dòng "Bạn đang ở hàng chờ dược sĩ, ca có trẻ em được ưu tiên"; dải đỏ cấp cứu luôn nằm ở đầu khung chat.

**Khách làm gì tiếp:** bấm nút trả lời; sửa tóm tắt nếu sai; chờ dược sĩ; nếu bé nặng hơn trong lúc chờ thì bấm **Gọi 115** hoặc **Báo có dấu hiệu nặng** trên dải đỏ. Nếu khách không làm gì: ca vẫn ở hàng chờ ưu tiên, thông tin **không được lưu**.

**Ai được báo, sau bao lâu:** dược sĩ trực, **ngay khi chuyển**; ca có trẻ em đứng đầu hàng. Dược sĩ thấy: tóm tắt (đánh dấu nếu khách đã sửa), "Vì sao AI chuyển", "AI chưa chắc: cân nặng của bé, tên thuốc ở nhà", trạng thái lưu hồ sơ, và nguyên đoạn chat. **Quá 3 phút chưa ai nhận** → tự báo dược sĩ trưởng ca, và nhắn khách số hotline.

**Loại lỗi: lỗi ngữ cảnh.** Hệ thống chạy đúng thiết kế "khách hỏi thuốc → gợi ý thuốc", nhưng sai với tình huống vì người uống là trẻ. Vì vậy:
- **Cách nói:** không xin lỗi, không đổ cho khách ("bạn quên nói tuổi"); AI nói rõ **mình đang thiếu gì** ("tin nhắn chưa nói bé mấy tuổi").
- **Cách sửa:** sửa thiết kế chứ không chỉ sửa câu — thêm luật "chưa biết người uống thuốc là ai thì không được gọi `goi_y_san_pham`" (xem §4).

### Kịch bản K2 — Cấp cứu lẫn trong câu mua thuốc

**Quyết định: dừng, khuyên đi khám ngay, đồng thời báo dược sĩ khẩn.**

| Phương án | Chọn / loại | Vì sao |
|---|---|---|
| Hỏi lại ("đau ngực bao lâu rồi ạ?") | **Loại** | Mỗi câu hỏi là thêm phút trong khi "tức ngực + vã mồ hôi + 70 tuổi" đã đủ để bảo đi khám. |
| Trả lời kèm giới hạn ("có thể dùng thuốc X, nhưng nếu không đỡ thì đi khám") | **Loại** | Bất kỳ gợi ý thuốc nào cũng khiến khách ở nhà chờ "xem có đỡ không". |
| Chỉ chuyển dược sĩ | **Loại** | Ca nằm trong hàng chờ — đúng điều rubric riêng ý 3 cấm. |
| **Khuyên đi khám ngay + báo dược sĩ khẩn song song** | **Chọn** | Khách không phải chờ ai để biết phải làm gì; dược sĩ gọi lại để hỗ trợ, nhưng việc đi khám không phụ thuộc vào cuộc gọi đó. |

**AI nói gì (nguyên văn):**
*"Đây là dấu hiệu cần cấp cứu. Đưa người nhà của bạn đến cơ sở y tế gần nhất hoặc gọi 115 ngay. Mình không gợi ý thuốc cho trường hợp này. Vì sao: tin nhắn có dấu hiệu "tức ngực"."*
Tiếp theo, tin hệ thống: *"Dược sĩ trực đã được báo ngay để gọi lại cho bạn. Bạn không cần chờ — cứ đi khám trước."*

**Khách thấy gì:** thẻ đỏ; nút **Gọi 115** (bấm là gọi); nút **Mình đang đi khám**; **không có thẻ sản phẩm nào** trên màn hình.

**Khách làm gì tiếp:** gọi 115 hoặc đi khám. Bấm "Mình đang đi khám" thì hệ thống ghi nhận, dược sĩ vẫn gọi lại hỗ trợ. Nếu khách không làm gì: dược sĩ vẫn gọi lại.

**Ai được báo, sau bao lâu:** dược sĩ trực, **ngay lập tức**; ca đánh dấu **KHẨN**, đứng trên mọi ca khác, không vào hàng chờ thường. Dược sĩ thấy: "KHẨN — khách đã được bảo đi cấp cứu", dấu hiệu AI nhận ra, và việc phải làm: *"gọi lại khách trong 2 phút để hỗ trợ. Không gợi ý thuốc."*

**Loại lỗi: lỗi hệ thống.** Rủi ro chính của K2 là AI **không nhận ra** dấu hiệu — khách viết "nặng ngực", "khó chịu ở ngực", sai chính tả, hoặc giấu dấu hiệu giữa câu mua thuốc. Vì đây là lỗi của hệ thống nên:
- **Cách nói:** khi khách tự báo dấu hiệu nặng mà AI bỏ sót, AI không hỏi lại, không giải thích dài, đi thẳng vào thẻ cấp cứu.
- **Cách sửa:** không thể chỉ dựa vào việc AI nhận diện đúng → thiết kế một đường không phụ thuộc AI (xem dưới).

### Nguyên tắc HAX/PAIR (trỏ vào chỗ cụ thể; số trùng với ảnh prototype)

- **HAX G10 — Thu hẹp khi nghi ngờ** → câu hỏi một dòng có nút chọn ở K1: *"Hỏi nhanh một câu trước: bé có dấu hiệu nào dưới đây không?"* thay vì đoán và gợi ý thuốc. (số 3)
- **HAX G2 — Làm rõ làm tốt tới đâu** → ô vàng *"Mình chưa chắc: tin nhắn chưa nói bé mấy tuổi…"*: nói rõ thiếu thông tin gì, không dùng con số % mà khách nửa đêm không hiểu. (số 2)
- **PAIR Errors + Graceful Failure** → dải đỏ *"Khó thở, đau ngực, co giật, lơ mơ? Đi cấp cứu ngay"* với nút **Gọi 115** và **Báo có dấu hiệu nặng**, luôn nằm đầu khung chat: đường lui khi AI nhận diện sai ở K2. (số 4)

Ngoài ra, G1 (nhãn "Trợ lý AI · không phải dược sĩ", cũng là yêu cầu minh bạch của Luật Trí tuệ nhân tạo 134/2025/QH15, Điều 11 khoản 1) có trên mọi tin AI; G11, G9, G8 và PAIR Feedback + Control được trỏ trên prototype ở §5.

### Lỗi mà sửa câu chữ không cứu được

1. **AI bỏ sót dấu hiệu cấp cứu (K2).** Viết câu trả lời hay đến đâu cũng vô ích nếu AI không nhận ra dấu hiệu. **Thiết kế cứu:** dải đỏ cấp cứu cố định ở đầu chat, không phụ thuộc AI; khách tự bấm "Báo có dấu hiệu nặng" là đi thẳng vào thẻ cấp cứu và báo dược sĩ khẩn. Dược sĩ cũng luôn có nguyên đoạn chat để tự đọc, không chỉ dựa vào tóm tắt của AI.
2. **AI dặn "hỏi dược sĩ trước khi dùng" nhưng vẫn hiện nút Đặt (K1).** Câu dặn không chặn được khách bấm mua lúc nửa đêm. **Thiết kế cứu:** với ca trẻ em, **không có thẻ sản phẩm nào** trên màn hình khách cho tới khi dược sĩ tự bật ô "Kèm gợi ý sản phẩm" trên màn hình dược sĩ.

---

## §4. Mức tự chủ

| Tool | Mức của nhóm | Sai thì ai chịu gì | Đảo được không | Chờ người thì mất gì |
|---|---|---|---|---|
| `goi_y_san_pham` | **Tự làm khi qua luật · còn lại cần dược sĩ duyệt** | Người uống thuốc (thường không phải người nhắn) bị hại sức khoẻ | Không, khi thuốc đã dùng | Khách hỏi món đơn giản (nước muối) phải chờ vô ích; hàng chờ đầy làm ca trẻ em chờ lâu hơn |
| `chuyen_duoc_si` | **Tự làm** | Dược sĩ mất vài phút vì ca không cần chuyển | Được, tốn vài phút | Chờ duyệt để chuyển người là vô nghĩa; khách đang lo phải chờ thêm |
| `khuyen_di_kham` | **Tự làm** | Báo thừa: khách mất công đi khám hoặc bỏ qua | Khách bỏ qua được | Ca nguy hiểm nằm trong hàng chờ (K2) |
| `luu_ghi_chu_suc_khoe` | **Cần khách duyệt** | Khách và người nhà bị lộ thông tin sức khoẻ | Xoá được, nhưng rủi ro lộ đã xảy ra | Gần như không mất gì: không lưu thì lần sau kể lại |
| `ban_thuoc_ke_don` | **Cấm** | Người uống bị hại; nhà thuốc vi phạm quy định | Không | — |

### Lý do và phương án đã loại, từng tool

**`goi_y_san_pham` — tự làm khi qua luật, còn lại cần dược sĩ duyệt.**
- **Luật (AI chỉ tự gợi ý khi đủ cả 6 điều):** người dùng thuốc là người lớn tự dùng · không mang thai, không cho con bú · không bệnh mạn tính, không đang dùng thuốc khác · không dị ứng thuốc · triệu chứng dưới 3 ngày, chưa tự dùng thuốc mà không đỡ · sản phẩm không kê đơn. Không bao giờ kèm liều.
- **Không qua luật → cần dược sĩ duyệt.** Ai duyệt: dược sĩ trực. Người duyệt thấy: tóm tắt, "Vì sao AI chuyển", "AI chưa chắc: …", nguyên đoạn chat, và ô "Kèm gợi ý sản phẩm" **tắt mặc định**. Quá hạn (3 phút chưa nhận): ca lên dược sĩ trưởng ca, khách nhận số hotline; **gợi ý không bao giờ tự gửi khi quá hạn**.
- **Loại "tự làm hoàn toàn":** K1, K5, K9 đều dẫn tới một sản phẩm sai cho người không thể đảo lại.
- **Loại "luôn cần dược sĩ duyệt":** K3 (nước muối) và K4 bình thường phải chờ vô ích; hàng chờ dài hơn làm chính ca trẻ em (K1) bị nhận chậm hơn.
- **Loại "cấm":** mất giá trị chính của tính năng ở lát cắt (AI "tự trả lời và gợi ý sản phẩm").

**`chuyen_duoc_si` — tự làm.**
- Sai (chuyển thừa) chỉ tốn vài phút của dược sĩ; chuyển thiếu thì trẻ nhỏ có thể nhận sai thuốc. Đây là đúng hướng sai chấp nhận được đã nêu ở §1.
- Luôn kèm tóm tắt khách sửa được, để dược sĩ không hỏi lại từ đầu.
- **Loại "cần khách xác nhận trước khi chuyển":** ở K8 khách đang gắt; thêm một bước hỏi "bạn có muốn gặp dược sĩ không?" là thêm chờ mà không chặn được sai gì.

**`khuyen_di_kham` — tự làm.**
- Báo thừa thì khách bỏ qua được; báo thiếu hoặc chậm thì người bệnh ở K2 mất thời gian cấp cứu.
- **Loại "cần dược sĩ duyệt":** chính là để ca nguy hiểm nằm trong hàng chờ — rubric riêng ý 3 cấm điều này.
- Mỗi lần gọi `khuyen_di_kham` luôn kèm một lần `chuyen_duoc_si` ở mức **KHẨN** (xem luật thêm).

**`luu_ghi_chu_suc_khoe` — cần khách duyệt.**
- Ai duyệt: khách. Khách thấy: đúng nội dung sẽ lưu (tóm tắt) và hai nút **Đồng ý lưu / Không lưu**. Quá hạn hoặc không trả lời: **không lưu**.
- **Loại "tự làm":** vi phạm rubric riêng ý 7; xoá sau cũng không gỡ được rủi ro lộ.
- **Loại "cấm":** ở K9 khách chủ động muốn lưu bệnh của mẹ; lưu lại giúp dược sĩ lần sau biết ngay bệnh nền và thuốc đang dùng, tức là giúp an toàn hơn.

**`ban_thuoc_ke_don` — cấm.**
- Không đảo được, và AI không xác minh được đơn qua chat (ở K6 đơn đã mất; ảnh đơn có thể là đơn cũ hoặc của người khác).
- **Loại "cần dược sĩ duyệt":** nếu dược sĩ đã xem đơn và quyết định bán thì đó là dược sĩ bán, không còn là việc của AI; một nút xác nhận trong chat không đủ để kiểm đơn thật.
- Trên màn hình: **không có nút mua**, chỉ có đường sang dược sĩ ("Gửi ảnh đơn thuốc cho dược sĩ" / "Kể triệu chứng cho dược sĩ").

### Tool khó xếp nhất: `goi_y_san_pham`

- **Mỗi người từng chọn:**
  - Nguyễn Khánh Duy: tự làm, vì phần lớn câu hỏi như K3 (nước muối) và K4 (đau họng) là ca thường; bắt chờ dược sĩ thì khách bỏ đi.
  - Đặng Thanh Hải: cần dược sĩ duyệt, vì K1 (bé sốt) và K5 (bà bầu) cho thấy một gợi ý sai thì không đảo được khi thuốc đã uống.
  - Hoàng Gia Khiêm: cấm, vì AI không biết chắc người uống là ai (K1, K9), mà sai thì hại sức khoẻ.
- **Cái giá phía "tự làm":** K1 (bé nhận thuốc người lớn), K5 (bà bầu được trấn an không căn cứ), K9 (siro thường cho người tiểu đường đang dùng thuốc) — đều không đảo được.
- **Cái giá phía "luôn cần dược sĩ duyệt":** K3 khách chỉ hỏi nước muối cũng phải chờ dược sĩ; mọi câu hỏi đơn giản dồn vào hàng chờ, làm ca thật sự cần dược sĩ (K1) bị nhận chậm hơn.
- **Nhóm chốt:** không chọn một mức cho cả tool mà **thêm luật** (6 điều ở trên) và **tách bước** cho nhóm nguy cơ: AI soạn gợi ý, dược sĩ bật mới gửi. Cách này chặn được K1, K5, K9 mà K3 không phải chờ.

### Thử mức đã đặt trên 3 kịch bản

| Kịch bản | Cái sai ở §2 có bị chặn không? | Còn việc nào AI làm khi lẽ ra phải hỏi? | Khách bình thường có phải chờ vô ích không? |
|---|---|---|---|
| **K1 · bé sốt** | **Có.** Chưa biết tuổi → không qua luật → `goi_y_san_pham` không được gọi; biết là trẻ em → chỉ dược sĩ bật được gợi ý. | `chuyen_duoc_si` tự làm không hỏi khách — nhóm chấp nhận vì chuyển thừa rẻ. `luu_ghi_chu_suc_khoe` không tự lưu. | Khách phải chờ dược sĩ, nhưng không vô ích: người uống là trẻ. Khách gắt (K8) thì AI ngừng hỏi và chuyển ngay. |
| **K2 · tức ngực** | **Có**, khi AI nhận ra dấu hiệu: `khuyen_di_kham` tự làm ngay, không có thẻ sản phẩm. **Còn lỗ hổng** khi AI bỏ sót dấu hiệu → được che bằng dải đỏ cố định (§3). | Không. | Không chờ: khách được bảo đi khám ngay, dược sĩ gọi lại song song. |
| **K3 · nước muối** (bình thường) | Không có cái sai "gợi ý sai" vì sản phẩm không phải thuốc và khách hỏi đích danh → qua luật, AI tự gợi ý kèm đặt giữ. Cái sai "bịa tồn kho" bị chặn bằng câu nói rõ không xem được tồn kho. | Không: đặt giữ có nút **Huỷ đặt**. | **Không**: không vào hàng chờ dược sĩ. |

### Luật thêm và tool tách bước

1. **Luật cho `goi_y_san_pham`:** 6 điều ở trên; thiếu bất kỳ thông tin nào thì coi như không qua. Chặn K1, K4 (khi chưa sàng lọc), K5, K9.
2. **Tách `goi_y_san_pham` thành "AI soạn gợi ý" và "dược sĩ bật gửi"** cho mọi ca không qua luật. Chặn lỗi "AI dặn hỏi dược sĩ nhưng vẫn hiện nút Đặt" ở §3.
3. **`khuyen_di_kham` luôn kèm `chuyen_duoc_si` mức KHẨN**, không vào hàng chờ thường, dược sĩ gọi lại trong 2 phút. Chặn trường hợp khách bị bảo đi khám rồi không ai theo dõi.
4. **`luu_ghi_chu_suc_khoe`: không trả lời = không lưu.**
5. **Chưa biết người dùng thuốc là ai thì không được gọi `goi_y_san_pham`.** Đây là cách sửa thiết kế cho lỗi ngữ cảnh ở K1.

### Một mức tự chủ đã đổi lời hứa nào với người dùng

Đặt `goi_y_san_pham` ở mức "chỉ tự làm khi qua luật" đã đổi lời hứa từ **"Trợ lý AI tư vấn thuốc cho bạn ngay"** thành:

> *"Trợ lý AI hỏi trước vài câu để dược sĩ trả lời bạn nhanh hơn. Thuốc cho trẻ em, người mang thai, người cao tuổi hay người có bệnh nền thì dược sĩ trả lời, không phải AI. Có dấu hiệu nguy hiểm thì bạn được bảo đi khám ngay, không phải xếp hàng chờ."*

Cái giá: khách nửa đêm ở K1 không nhận được câu "được / không được" từ AI như họ muốn. Đổi lại họ nhận một câu trả lời ngắn an toàn ("chưa nên cho bé uống thuốc của người lớn"), một chỗ đầu hàng chờ, và không phải kể lại từ đầu.

---

## §5. Prototype  (chặng 5)
- Link hoặc file · công cụ · phần nào mock · có lời gọi AI thật không.
- 1–2 ảnh có đánh số; mỗi số: thể hiện gì · nguyên tắc HAX/PAIR nào · mức tự chủ nào.
- Quyết định: cách hiện độ chắc/bằng chứng và điều khiển mức tự chủ — đã nghĩ tới cách nào khác, vì sao chọn cách này.

## §6. Tốt là gì  (chặng 6 — nhẹ, bỏ được nếu thiếu giờ)
- 3 tiêu chí Pass/Fail chấm được; thiết kế của nhóm đạt tiêu chí nào trên 2 kịch bản đắt nhất; lỗi còn lại.

## §7. Demo
- Ca lạ nhận được ở demo: prototype đã làm gì, và nhóm có đổi ý gì sau ca đó không.