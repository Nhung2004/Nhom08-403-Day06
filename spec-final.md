# SPEC — AI Product Hackathon

**Nhóm:** Nhóm 08
**Track:** ☐ VinFast · ☑ Vinmec · ☐ VinUni-VinSchool · ☐ XanhSM · ☐ Open
**Problem statement (1 câu):** Bệnh nhân khi có triệu chứng đau ốm thường lúng túng không biết phải khám ở chuyên khoa nào, dẫn đến việc đặt nhầm khoa hoặc phải gọi điện hỏi tổng đài gây tốn thời gian chờ đợi; AI sẽ đóng vai trò "điều dưỡng sơ yếu", trao đổi ngắn gọn để phân tích triệu chứng và tự động gợi ý đúng chuyên khoa, rút ngắn luồng booking (đặt lịch).

---

## 1. AI Product Canvas

|   | Value | Trust | Feasibility |
|---|-------|-------|-------------|
| **Câu hỏi** | User nào? Pain gì? AI giải gì? | Khi AI sai thì sao? User sửa bằng cách nào? | Cost/latency bao nhiêu? Risk chính? |
| **Trả lời** | Bệnh nhân cần đặt lịch khám nhưng ngợp trước hàng chục chuyên khoa. AI dựa vào text tự do để match triệu chứng ra chuyên khoa, xua tan sự băn khoăn. User trung bình tiết kiệm 5-10 phút so với gọi tổng đài trực tiếp. | AI gợi ý sai khoa -> User thấy không hợp lý hoặc nhận kết quả "Không chắc chắn", bèn ấn nút "Gặp lễ tân/Tổng đài". Feedback từ lễ tân/bác sĩ được lưu vào learning loop để optimize prompt. Bác sĩ có quyền ấn "Điều chuyển khoa" để sửa sai. | Latency < 2s/lượt chat, cost < $0.01/interaction. Model: GPT-4o mini hoặc Claude 3.5 Haiku. Risk chính: Bỏ sót các tín hiệu bệnh lý nguy kịch (Red Flags) cần cấp cứu gấp - mitigation bằng dynamic keyword detection + hardcoded urgent flows. |

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
| Happy — AI đúng, tự tin | User thấy gì? Flow kết thúc ra sao? | AI hỏi thêm: "Bạn có kèm sốt không?". User: "Có". AI hỏi tiếp: "Triệu chứng kéo dài bao lâu rồi?". AI trả kết quả: "Mức độ phù hợp 95%: Khám Ngoại Tiêu hoá" và hiện danh sách Bác sĩ Ngoại tiêu hoá có sẵn + lịch khám trong ngày. User ấn đặt lịch, thanh toán online (hoặc BH e-claims). Toàn bộ flow < 2 phút. |
| Low-confidence — AI không chắc | System báo "không chắc" bằng cách nào? User quyết thế nào? | Triệu chứng mập mờ ("Tôi bị mệt", "Tôi bị yếu"). AI cố hỏi 1-2 câu bổ sung như "Mệt lúc nào? Sáng? Cả ngày?" nhưng vẫn không gỡ được hoặc xác định độ tin cậy < 60%. AI báo: "Triệu chứng khá chung, có thể nhiều nguyên nhân khác nhau. Bạn muốn chọn: (1) Khám Tổng Quát đa chuyên khoa lần đầu, (2) Gặp Lễ tân để tư vấn thêm, hay (3) Gọi tổng đài 1900-1899?" User click một trong ba lựa chọn mà không bị cơ học. |
| Failure — AI sai | User biết AI sai bằng cách nào? Recover ra sao? | AI khuyên khám Da liễu do có từ khóa "nóng gan", nhưng bản chất bệnh nhân rối loạn tiêu hóa. Tại phòng khám, tiếp tân/bác sĩ dùng hệ thống: (1) Ấn check "Khoa này không phù hợp", hoặc (2) Bác sĩ ấn "Điều chuyển sang Nội tiêu hóa trực tiếp". Bệnh nhân được điều chuyển qua Nội tiêu hóa, tư vấn lại FREE không mất phí. Log sự kiện `(triệu chứng → khoa sai → khoa đúng)` tự động được đẩy vào Training DB. |
| Correction — user sửa | User sửa bằng cách nào? Data đó đi vào đâu? | Nút "Khoa này không phù hợp" hoặc "Điều chuyển khoa" của bác sĩ trên app clinic tự động kích hoạt lưu log transaction. Hệ thống capture: timestamp, user_id, original_transcript, recommended_dept, actual_dept, reason (nếu có). Cặp dữ liệu `(Triệu chứng gốc + metadata → Khoa đúng)` được nạp vào corrected_samples table. Hàng tuần, data team chạy fine-tuning job trên system prompt để co-optimize với fine-tuning data mới này. |

---

## 3. Eval metrics + threshold

**Optimize precision hay recall?** ☐ Precision · ☑ Recall
Tại sao? Trong y tế, Recall là chỉ số sống còn đối với các loại bệnh nguy hiểm (Quy tắc Red Flag). Thà AI nhạy cảm quá mức, thấy "đau ở lồng ngực" là gợi ý Cấp Cứu ngay (mặc dù có thể đó chỉ là do trào ngược dạ dày bẩm sinh - False positive), còn hơn bỏ sót (False negative) coi đó là bệnh tiêu hóa thông thường, làm lỡ thời gian vàng cấp cứu nhồi máu cơ tim.

| Metric | Threshold | Red flag (dừng khi) |
|--------|-----------|---------------------|
| Recall đối với nhóm Keyword Khẩn Cấp (Red Flags) | = 100% | Có bất kỳ 1 ca cấp cứu nào bị AI phân loại là khám thông thường (Rủi ro tính mạng). |
| Precision đối với Top 1 recommendation | ≥ 85% | Rơi xuống dưới 75% (User mất tín tưởng vào AI). |
| Tỉ lệ khách tự book lịch qua Chatbot thành công (Self-serve) | > 50% | Rơi xuống dưới 20% (AI đang làm rườm rà hệ thống hơn lúc chưa có AI). |
| Tỉ lệ phản hồi "Khoa này không phù hợp" từ Bác Sĩ khám thực tế | < 3% | Vượt > 5% trong 1 tuần (Gây ức chế vận hành của cơ sở y tế). |
| Latency trung bình (end-to-end, từ gửi message đến nhận recommendation) | < 2s | Vượt > 5s liên tục (UX tồi, user bỏ cuộc). |
| CSAT (Customer Satisfaction) score từ trong-app survey | ≥ 4.0/5.0 | Xuống dưới 3.5/5.0 (Cần thay đổi chiến lược). |

---

## 4. Top 3 failure modes

| # | Trigger | Hậu quả | Mitigation |
|---|---------|---------|------------|
| 1 | Bệnh nhân dùng từ lóng vùng miền, sai chính tả nặng, diễn đạt lủng cũi | AI không hiểu, đoán bừa hoặc vòng lặp hỏi lại người bệnh liên hồi → User nản chí, rời khỏi flow. | (1) Baked-in keyword fuzzy matching + regional dialect mapping (e.g. "đau trúng gió" → ngoại tiêu hóa/thần kinh). (2) Đặt biến đếm (clarification_counter): Nếu consecutive "Tôi không hiểu" hơn 2 lần và confidence < 55% -> Ép hiển thị popup "Bạn muốn nói chuyện với Lễ tân?" kèm click-to-call. (3) Background: Fine-tune partial LLM trên 10k transcripts tiếng Việt địa phương từ Vinmec. |
| 2 | Prompt Injection hoặc hỏi lan man ngoài lề | Bệnh nhân hỏi các vấn đề sức khỏe sinh lý tế nhị nhưng không muốn đi khám, bắt AI tư vấn mẹo dân gian. AI bịa ra kiến thức y khoa bậy lố lăng. → Pháp lý/brand risk. | (1) System Prompt rõ ràng: "Bạn là Trợ lý Tiếp đón và Phân khoa (Triage), không phải Bác Sĩ. Không được chẩn bệnh, không được đưa ra liệu pháp". (2) Guardrails: Lọc input bằng regex/NER, từ chối category như "Địa chỉ mua thuốc trái phép", "Cách chữa tại nhà không đi khám". (3) Fallback response: "Tôi chỉ giúp xác định chuyên khoa phù hợp. Các vấn đề y tế cần được tư vấn bởi Bác Sĩ trực tiếp. Bạn muốn đặt lịch khám không?" |
| 3 | Bệnh nhân giấu bệnh, mô tả thiên lệch để giấu mức độ | AI đánh giá mức độ nhẹ, khuyên khám khoa thường, bỏ qua rủi ro nghiêm trọng gián tiếp → Delay in diagnosis, liability. | (1) Dynamic Red Flag keywords: Nếu detect "Khó thở", "Tức ngực", "Mờ mắt", "Đau đầu kinh hoàng", "Yếu tay chân" → Bypass triage logic bình thường, hiện alert banner đỏ kèm "Khuyến cáo: Hãy gọi 115 hoặc tới ER ngay" + quick-dial button. (2) Disclaimer trước chat + sau kết quả. (3) Terms of service: User confirm "Tôi hiểu rằng tư vấn này chỉ mang tính chỉ dẫn, không phải diagnosis. Nếu bệnh trầm trọng, tôi sẽ liên hệ Cấp cứu 115". |

---

## 5. ROI 3 kịch bản

|   | Conservative | Realistic | Optimistic |
|---|-------------|-----------|------------|
| **Assumption** | 500 khách dùng App Đặt Lịch/ngày, 40% chịu chat với Bot, 80% hoàn thành booking | 1,500 khách/ngày, 65% dùng Bot thành công trơn tru, 90% hoàn thành booking | 3,000 lượt booking/ngày, 85% tỉ lệ quét hoàn toàn bằng Bot, 95% hoàn thành booking |
| **Cost** | ~$10/ngày (API Inference, 400-500 interactions/ngày @ ~0.01 + infrastructure) | ~$30/ngày (1000 interactions, slight optimization từ learning data) | ~$60/ngày (2500 interactions, well-tuned system) |
| **Benefit** | • Giảm 1-2 nhân sự lễ tân trực luồng tiếp đón online (~500k USD/năm) • Tăng booking rate do UX tốt hơn • Baseline: ROI trong 6 tháng | • Tự động hóa 60% workload của Call Center phân khoa khám (~1.5M USD/năm) • Tăng conversion booking từ abandon customer (~15% additional) • Giảm average call duration từ 5p xuống 2p • ROI dương, breakeven tháng 3-4 | • Thay đổi thói quen dùng app. Tiết kiệm OPEX khổng lồ khi mở rộng chuỗi phòng khám • Mở rộng sang các bệnh viện khác trong network Vinmec (20+ cơ sở) • Network effect: data từ tất cả cơ sở pooling lại, model AI ngày càng thông minh • Revenue add-on: licensing model cho đối tác |
| **Net** | Payback period: 6-9 tháng. Cost/booking KPI dưới 5% tổng chi phí booking | Payback period: 3-4 tháng. Cost/booking KPI dưới 2% tổng chi phí booking. Điểm balanc với phần ứng phó 24/7 cảnh báo cấp cứu. | Payback period: 1-2 tháng. Cost/booking KPI < 1%. Chiến lược mở rộng: licensing sang các chuỗi y tế khác. Potential revenue: +$100k-500k/năm từ licensing. |

**Kill criteria:** Nhận > 5 báo cáo (Ticket) khiếu nại (đánh giá 1 sao) về việc ứng dụng "bắt chat với BOT mất thời gian mà cuối cùng vẫn không ra khoa ưng ý", thể hiện Friction (ma sát) lớn hơn Value (giá trị). Dừng để điều chỉnh luồng UI/UX.

---

## 6. Mini AI spec (1 trang)

### **Tên dự án:** Vinmec AI Triage (Cô điều dưỡng sơ tuyển thông minh)

**Vấn đề & Target User:**
Khi đau ốm, bệnh nhân và người nhà thường băn khoăn "Mình bị đau bụng kiểu này thì rốt cuộc đăng ký khám khoa nào?". Màn hình đặt khám với danh sách thả xuống hàng loạt chuyên khoa (Nội tiêu hóa, Nội tiết, Hô hấp, Gan Mật...) làm họ bị "khớp". Đa số họ sẽ phải Google lung tung, chọn bừa khoa, hoặc gọi lên tổng đài để hỏi. Điều này tạo ra nút thắt cổ chai tắc nghẽn đáng kể ở khâu tiếp đón bệnh nhân từ xa.

**Giải pháp: Trợ lý Sàng lọc Tự Động (Triage Augmentation)**

1. **Giao diện & Flow:**
   - Chatbot xuất hiện ngay giao diện "Đặt lịch bằng mô tả" (text/voice input hỗ trợ)
   - Bệnh nhân gõ hoặc nói triệu chứng bằng lời nói tự do (không cần diagnosis chính xác)
   - Chatbot phản hồi trong < 2s với câu hỏi follow-up thông minh (1-3 câu)

2. **Xử lý Logic:**
   - LLM phân tích text tự do bằng prompt engineering + retrieval-augmented generation (RAG) trên vector DB chuyên khoa Vinmec
   - Mapping logic: Symptom tokens → Medical concept → Specialty recommendation
   - Dynamic severity scoring: Red flag detection vs normal triage flow
   - Output: Top 3 specialty + confidence score (%) + brief reasoning

3. **User Experience:**
   - Trả ra Top 3 chuyên khoa chuẩn nhất và % độ khớp (e.g. "95% - Ngoại Tiêu hóa, 40% - Nội Tiêu hóa, 15% - Tâm thần")
   - User hoàn toàn chủ động chọn specialty hoặc yêu cầu "Gặp Lễ tân tư vấn thêm" hoặc "Gọi tổng đài"
   - Không hề tước đi quyền lựa chọn cuối cùng của khách hàng

**Quality & Risk Management:**

1. **Red Flag Handling (Critical):**
   - Hardcoded urgent keywords: Khó thở, Tức lồng ngực, Chảy máu, Mờ mắt, Yếu tay chân, Đau đầu kinh hoàng, v.v.
   - KPI: Recall = 100% trên Red Flag category (không được bỏ sót ca nào)
   - Khi detect: Bypass triage logic, hiển thị banner đỏ + "GỌI 115 / ĐI ER NGAY" + quick-dial button
   - System Prompt có "System instruction" rõ: "Nếu detect Red Flag, immediate fallback to emergency routing"

2. **Guardrails & Safety:**
   - Input filtering: Regex + NER (Named Entity Recognition) để detect malicious patterns, prompt injection
   - Refuse categories: Không chẩn bệnh online, không tư vấn medication without doctor approval, không cung cấp mẹo dân gian thay thế điều trị
   - Persona lock: Bot là Lễ tân Triage, cấm tuyệt đối đóng vai Bác sĩ
   - Disclaimer: "Các tư vấn có tính chỉ dẫn sơ bộ. Chẩn đoán chính xác từ Bác sĩ trực tiếp."

3. **Evaluation Metrics:**
   - Recall على Red Flags: 100%
   - Precision on Top 1: ≥ 85%
   - Self-serve booking rate: > 50%
   - Specialty mismatch rate: < 3%
   - Latency: < 2s
   - CSAT score: ≥ 4.0/5.0

**Data Flywheel & Continuous Learning:**

1. **Learning Loop (Humans-in-the-loop):**
   - **Episode 1:** Bệnh nhân nhập triệu chứng → AI recommend Khoa X → Bệnh nhân booking
   - **Episode 2:** Bác sĩ Khoa X tiếp nhận, nhận ra bệnh nhân thuộc Khoa Y → Bác sĩ click "Điều chuyển sang Khoa Y" + leave feedback reason
   - **Episode 3:** System tự động capture: `(original_transcript, metadata) → actual_department`
   - **Episode 4:** Engineers/Data team xếp loại correction (true positive, false positive, etc.), dump vào corrected_samples table

2. **Model Improvement Cycle:**
   - **Weekly job:** Fine-tune system prompt dựa trên:
      - Top 50 misclassified samples từ tuần trước
      - Cập nhật/mở rộng specialty-to-symptom mapping dictionary
      - A/B test: Prompt v1 vs v2 trên backtested samples
   - **Monthly:** Evaluate model performance tổng quát, update guardrails nếu cần
   - **Quarterly:** Hợp tác với Bác sĩ domain experts để review edge cases, update Red Flag keywords

3. **Competitive Moat (Data as Asset):**
   - Sau 6 tháng, Vinmec sẽ sở hữu bộ dữ liệu ngôn ngữ dân dã & từ địa phương về miêu tả ốm đau **tại Việt Nam** khổng lồ nhất (10k-50k transcripts)
   - Model này sẽ amhiểu từ lóng vùng miền ("đau trúng gió" = ngoại tiêu hóa, "nhức mỏi ráng" → thần kinh), thứ mà model AI nhập khẩu không bắt bước được
   - Có thể fine-tune riêng cho từng vùng địa lý trong Vinmec network
 