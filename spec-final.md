# SPEC — AI Product Hackathon

**Nhóm:** (Điền tên nhóm của bạn)
**Track:** ☐ VinFast · ☑ Vinmec · ☐ VinUni-VinSchool · ☐ XanhSM · ☐ Open
**Problem statement (1 câu):** Bệnh nhân khi có triệu chứng đau ốm thường lúng túng không biết phải khám ở chuyên khoa nào, dẫn đến việc đặt nhầm khoa hoặc phải gọi điện hỏi tổng đài gây tốn thời gian chờ đợi; AI sẽ đóng vai trò "điều dưỡng sơ yếu", trao đổi ngắn gọn để phân tích triệu chứng và tự động gợi ý đúng chuyên khoa, rút ngắn luồng booking (đặt lịch).

---

## 1. AI Product Canvas

|   | Value | Trust | Feasibility |
|---|-------|-------|-------------|
| **Câu hỏi** | User nào? Pain gì? AI giải gì? | Khi AI sai thì sao? User sửa bằng cách nào? | Cost/latency bao nhiêu? Risk chính? |
| **Trả lời** | Bệnh nhân cần đặt lịch khám nhưng ngợp trước hàng chục chuyên khoa. AI dựa vào text tự do để match triệu chứng ra chuyên khoa, xua tan sự băn khoăn. | AI gợi ý sai khoa -> User thấy không hợp lý hoặc nhận kết quả "Không chắc chắn", bèn ấn nút "Gặp lễ tân/Tổng đài". | Latency < 2s/lượt chat, cost < $0.01. Risk chính: Bỏ sót các tín hiệu bệnh lý nguy kịch (Red Flags) cần cấp cứu gấp. |

**Automation hay augmentation?** ☑ Augmentation (Tăng cường)
Justify: AI ở đây là bước đệm sơ khảo (Augmentation) để định hướng và giảm tải cho tổng đài viên. AI đưa ra Top 3 dự đoán khoa khám, nhưng quyền click chọn khoa nào để thanh toán (hoặc quyền yêu cầu gọi Human Lễ tân) hoàn toàn nằm trong tay người dùng.

**Learning signal:**

1. User correction đi vào đâu? Khi user từ chối gợi ý của AI và yêu cầu tiếp tân hỗ trợ thủ công, bản Text Log triệu chứng và Khoa thực tế được book sẽ được gom vào DB. Hoặc khi bác sĩ khám thấy sai chuyên môn, ấn chuyển khoa trên hệ thống.
2. Product thu signal gì để biết tốt lên hay tệ đi? Tỷ lệ User hoàn tất book lịch tĩnh mà không cần nhảy sang gọi tổng đài; Tỷ lệ bác sĩ dùng tính năng "Điều chuyển nhầm khoa".
3. Data thuộc loại nào? ☐ User-specific · ☑ Domain-specific · ☐ Real-time · ☑ Human-judgment · ☐ Khác
   Có marginal value không? Có. Các thuật ngữ mô tả đau ốm dân dã ("đau trúng gió", "nhức mỏi ráng") ban đầu AI có thể không map chuẩn. Qua dữ liệu Correction của các ca thực tế, AI của Vinmec sẽ am hiểu từ ngữ bệnh trạng của người Việt hơn hẳn mô hình gốc.

---

## 2. User Stories — 4 paths

### Feature: Chatbot Triage (Sàng lọc triệu chứng và gợi ý chuyên khoa)

**Trigger:** User mở ứng dụng Vinmec, vào mục "Đặt lịch khám", nhập text: *“Tôi bị đau quặn bụng dưới bên phải từ tối qua.”*

| Path | Câu hỏi thiết kế | Mô tả |
|------|-------------------|-------|
| Happy — AI đúng, tự tin | User thấy gì? Flow kết thúc ra sao? | AI hỏi thêm: "Bạn có kèm sốt không?". User: "Có". AI trả kết quả: "Mức độ phù hợp 95%: Khám Ngoại Tiêu hoá" và hiện danh sách Bác sĩ để User click đặt lịch. |
| Low-confidence — AI không chắc | System báo "không chắc" bằng cách nào? User quyết thế nào? | Triệu chứng mập mờ ("Tôi bị mệt"). AI cố hỏi 1 câu nhưng vẫn không gỡ được. AI báo: "Triệu chứng khá chung. Bạn muốn chọn Khám Tổng Quát hay gặp [Lễ tân] để được tư vấn thêm?" |
| Failure — AI sai | User biết AI sai bằng cách nào? Recover ra sao? | AI khuyên khám Da liễu do có từ khóa "nóng gan", nhưng bản chất bệnh nhân rối loạn tiêu hóa. Tại phòng khám, bác sĩ dùng hệ thống ấn check "Nhầm khoa", bệnh nhân được điều chuyển qua Nội tiêu hóa không mất phí. |
| Correction — user sửa | User sửa bằng cách nào? Data đó đi vào đâu? | Nút "Nhầm khoa" của bác sĩ kích hoạt lưu log. Cặp dữ liệu `(Triệu chứng ban đầu -> Khoa đúng)` được nạp vào DB để tối ưu hóa bộ system prompt cho cycle tới. |

---

## 3. Eval metrics + threshold

**Optimize precision hay recall?** ☐ Precision · ☑ Recall
Tại sao? Trong y tế, Recall là chỉ số sống còn đối với các loại bệnh nguy hiểm (Quy tắc Red Flag). Thà AI nhạy cảm quá mức, thấy "đau ở lồng ngực" là gợi ý Cấp Cứu ngay (mặc dù có thể đó chỉ là do trào ngược dạ dày bẩm sinh - False positive), còn hơn bỏ sót (False negative) coi đó là bệnh tiêu hóa thông thường, làm lỡ thời gian vàng cấp cứu nhồi máu cơ tim.

| Metric | Threshold | Red flag (dừng khi) |
|--------|-----------|---------------------|
| Recall đối với nhóm Keyword Khẩn Cấp (Red Flags) | = 100% | Có bất kỳ 1 ca cấp cứu nào bị AI phân loại là khám thông thường (Rủi ro tính mạng). |
| Tỉ lệ khách tự book lịch qua Chatbot thành công (Self-serve) | > 50% | Rơi xuống dưới 20% (AI đang làm rườm rà hệ thống hơn lúc chưa có AI). |
| Tỉ lệ phản hồi "Nhầm khoa" từ Bác Sĩ khám thực tế | < 3% | Vượt > 5% trong 1 tuần (Gây ức chế vận hành của cơ sở y tế). |

---

## 4. Top 3 failure modes

| # | Trigger | Hậu quả | Mitigation |
|---|---------|---------|------------|
| 1 | Bệnh nhân dùng từ lóng vùng miền, sai chính tả nặng, diễn đạt lủng củng | AI không hiểu, đoán bừa hoặc vòng lặp hỏi lại người bệnh liên hồi | Đặt biến đếm (counter): Nếu AI gặp rào cản và phải hỏi lại khách quá 2 lần -> Ép hiển thị popup Nút Gọi Tổng Đài/Lễ Tân để giải thoát người dùng. |
| 2 | Prompt Injection hoặc hỏi lan man ngoài lề | Bệnh nhân hỏi các vấn đề sức khỏe sinh lý tế nhị nhưng không muốn đi khám, bắt AI tư vấn mẹo dân gian. AI bịa ra kiến thức y khoa bậy lố lăng. | System Prompt dùng Railguard nghiêm ngặt. Bot có persona rõ là Lễ Tân phân khoa, bị cấm tuyệt đối vai trò Bác Sĩ chẩn bệnh online. Từ chối trả lời mọi mưu đồ mớm bệnh. |
| 3 | Bệnh nhân giấu bệnh, mô tả thiên lệch để giấu mức độ | AI đánh giá mức độ nhé, khuyên khám khoa thường, bỏ qua rủi ro nghiêm trọng gián tiếp | Gắn nhãn Disclaimer rõ ràng trước và sau khi chat: "Các tư vấn chỉ mang tính tham khảo sơ bộ. Nếu có bất kỳ cơn đau/khó thở dữ dội nào, xin vui lòng tới thẳng phòng Cấp cứu". |

---

## 5. ROI 3 kịch bản

|   | Conservative | Realistic | Optimistic |
|---|-------------|-----------|------------|
| **Assumption** | 500 khách dùng App Đặt Lịch/ngày, 40% chịu chat với Bot | 1,500 khách/ngày, 65% dùng Bot thành công trơn tru | 3,000 lượt booking/ngày, 85% tỉ lệ quét hoàn toàn bằng Bot |
| **Cost** | ~$10/ngày (API Inference text-only) | ~$30/ngày | ~$60/ngày |
| **Benefit** | Giảm bớt 1-2 nhân sự lễ tân trực luồng tiếp đón online | Tự động hóa 60% workload của bộ phận Call Center phân khoa khám | Thay đổi thói quen dùng app. Tiết kiệm OPEX khổng lồ khi mở rộng chuỗi phòng khám. |
| **Net** | ROI dương ngay lập tức về chi phí tiền lương | Chạy trơn tru, sinh lời chi phí vận hành rõ rệt | Chuyển đổi số lõi thành công. Tăng độ trung thành của khách vì tiện lợi. |

**Kill criteria:** Nhận > 5 báo cáo (Ticket) khiếu nại (đánh giá 1 sao) về việc ứng dụng "bắt chat với BOT mất thời gian mà cuối cùng vẫn không ra khoa ưng ý", thể hiện Friction (ma sát) lớn hơn Value (giá trị). Dừng để điều chỉnh luồng UI/UX.

---

## 6. Mini AI spec (1 trang)

### **Tên dự án:** Vinmec AI Triage (Cô điều dưỡng sơ tuyển thông minh)

**Vấn đề & Target User:**
Khi đau ốm, bệnh nhân và người nhà thường băn khoăn "Mình bị đau bụng kiểu này thì rốt cuộc đăng ký khám khoa nào?". Màn hình đặt khám với danh sách thả xuống hàng loạt chuyên khoa (Nội tiêu hóa, Nội tiết, Hô hấp, Gan Mật...) làm họ bị "khớp". Đa số họ sẽ phải Google lung tung, chọn bừa khoa, hoặc gọi lên tổng đài để hỏi. Điều này tạo ra nút thắt cổ chai tắc nghẽn đáng kể ở khâu tiếp đón bệnh nhân từ xa.

**Giải pháp: Trợ lý Sàng lọc Tự Động (Triage Augmentation)**
Chatbot sẽ xuất hiện ngay giao diện "Đặt lịch bằng mô tả". Bệnh nhân gõ/nói triệu chứng. Chatbot (các LLM phân tích quy trình ngôn ngữ tự nhiên hiểu domain y tế) sẽ mapping logic triệu chứng bệnh nhân với catalog chuyên khoa của bệnh viện. Nó sẽ chủ động hỏi ngược lại 1-2 câu khóa mấu chốt để "chốt hạ" (ví dụ: "Cơn đau bụng của anh/chị là quặn thắt hay râm ran?", "Có ói mửa theo không?").
Sau cùng, hệ thống trả ra Top 3 chuyên khoa chuẩn nhất và % độ khớp, giúp người dùng an tâm ấn đặt lịch. Nó không hề tước đi quyền lựa chọn cuối cùng của khách hàng.

**Quality & Risk:**
Chỉ số rủi ro quan trọng nhất của hệ thống này là đối phó với **Các tình huống khẩn cấp (Cấp cứu / Red flags)**, tức là phải đảm bảo KPI **Recall 100%**. Để chặn nguy cơ bỏ lọt bệnh nguy kịch (Làm chậm trễ cứu người vì Chatbot cứ vòng vo hỏi han lịch khám), AI được nhúng sẵn list các từ khóa Động. Hễ phát hiện ra chữ "Khá khó thở", "Tức lồng ngực", "Mờ mắt", Chatbot sẽ bỏ qua cơ chế lọc Khoa khám mạn tính, bung ra cảnh báo màu Đỏ rực kèm nút "KẾT NỐI TỔNG ĐÀI CẤP CỨU 115" ngay lập tức. Risk sinh mạng ở ca này là không được phép xảy ra.

**Data Flywheel:**
Cơ chế "Học hỏi vạn dặm" xảy ra tại phòng lâm sàng bằng vòng lặp Humans-in-the-loop: Khi bác sĩ tiếp nhận một bệnh nhân do AI điều hướng nhầm vào khoa mình -> Bác sĩ ấn click điều chuyển khoa trên máy. Hệ thống tự động bắt lại "Bản ghi lời nói gốc của bệnh nhân bữa nọ" và liên kết nó với "Khoa đích đã sửa". Sau 6 tháng, con AI của Vinmec sẽ sở hữu bộ dữ liệu ngôn ngữ dân dã và từ địa phương về miêu tả ốm đau khổng lồ nhất Việt Nam, đập tan rào cản thuật ngữ vùng miền, điều mà không một model AI nhập khẩu nào bắt chước được.
