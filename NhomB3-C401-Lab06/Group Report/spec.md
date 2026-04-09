# SPEC - AI Product Hackathon

**Nhóm:** lab6_B3
**Track:** [x] VinFast | [ ] Vinmec | [ ] VinUni-VinSchool | [ ] XanhSM | [ ] Open
**Problem statement (1 câu):** Chủ xe máy điện VinFast cần một trợ lý AI có thể tra cứu bảo hành theo từng xe, giải thích chính sách, chẩn đoán sơ bộ từ telemetry và hỗ trợ đặt/đổi lịch dịch vụ nhanh hơn việc gọi hotline hoặc hỏi showroom.

---

## 1. AI Product Canvas

|   | Value | Trust | Feasibility |
|---|-------|-------|-------------|
| **Câu hỏi** | User nào? Pain gì? AI giải gì? | Khi AI sai thì sao? User sửa bằng cách nào? | Cost/latency bao nhiêu? Risk chính? |
| **Trả lời** | User chính là chủ xe máy điện VinFast đã đăng nhập, có thể sở hữu nhiều xe, cần biết xe nào còn bảo hành, pin có còn được hỗ trợ không, xe có dấu hiệu bất thường gì và nên đến xưởng nào. Hiện tại họ phải tự đọc policy, gọi hotline hoặc hỏi showroom. AI giải bài toán này bằng cách đọc dữ liệu xe đã có trong hệ thống, trả lời theo policy mock, phân tích telemetry và dẫn flow booking. | Failure mode nguy hiểm nhất là AI trả lời sai chính sách bảo hành, tự ý chọn xưởng/khung giờ, hoặc thông báo booking thành công khi backend chưa giữ được slot. Prototype giảm rủi ro bằng guardrail trong system prompt, clarification trước khi gọi tool, từ chối ngoài scope, chỉ tạo booking sau khi có `slot_id` thật và user vẫn phải bấm xác nhận trên UI. User có thể sửa bằng cách chọn lại xe, nói rõ thành phố/xưởng/ngày giờ, hoặc đổi lịch. | Prototype khả thi vì dữ liệu đều có cấu trúc rõ: 6 xe mock, 10 service center, 1 warranty policy JSON, booking CSV và slot in-memory. Stack hiện tại là FastAPI + LangGraph + ChatOpenAI + tool calling. Mục tiêu là trả lời trong vài giây và giữ đúng slot trong 5 phút. Risk chính là mock data không real-time, heuristic NLU tiếng Việt có thể bỏ sót cách nói lỏng, và chưa kết nối hệ thống VinFast thật. |

**Automation hay augmentation?** [ ] Automation | [x] Augmentation
Justify: User vẫn là người quyết định cuối cùng. AI chỉ tra cứu, giải thích, đề xuất xưởng/slot, giữ chỗ tạm thời và yêu cầu user xác nhận. Cost of reject thấp, trong khi cost của một booking sai hoặc một cam kết sai policy là cao.

**Learning signal:**

1. User correction đi vào đâu? User correction đến từ việc đổi xe đang tra cứu, đổi thành phố/xưởng/ngày giờ, bỏ booking pending, đổi lịch, hoặc hỏi lại vì AI hiểu sai chủ đề. Trong repo hiện tại chưa có correction log riêng, nhưng đã có thể bắt đầu thu từ `messages`, `selected_vehicle_id`, `tool_calls_log`, `booking`, `status`, `expired/confirmed`.
2. Product thu signal gì để biết tốt lên hay tệ đi? Tỷ lệ booking được xác nhận trong 5 phút, tỷ lệ booking hết hạn, tỷ lệ user bị hỏi lại quá nhiều, tỷ lệ user tiếp tục chat sau câu trả lời đầu tiên, tỷ lệ out-of-scope, tỷ lệ reschedule và tỷ lệ escalation sang hotline.
3. Data thuộc loại nào? [x] User-specific | [x] Domain-specific | [x] Real-time | [x] Human-judgment | [ ] Khác: ___
   Có marginal value không? Có. Model chung không biết xe nào thuộc user, không biết policy JSON trong repo, không biết slot nào đang AVAILABLE/PENDING/CONFIRMED, và không biết booking nào đã được persist trong `data/bookings.csv`.

---

## 2. User Stories - 4 paths

### Feature 1: Tra cứu bảo hành và giải thích policy theo xe đang chọn

**Trigger:** User chọn xe trên sidebar hoặc nhắc đến một xe cụ thể, sau đó hỏi về bảo hành xe, bảo hành pin, linh kiện hoặc lịch bảo dưỡng.

| Path | Câu hỏi thiết kế | Mô tả |
|------|-------------------|-------|
| Happy - AI đúng, tự tin | User thấy gì? Flow kết thúc ra sao? | User hỏi "Xe này còn bảo hành không?". Agent dùng `selected_vehicle_id` hoặc vehicle reference, gọi `lookup_warranty_status`, trả về thời hạn bảo hành xe và pin, ngày hết hạn, số ngày còn lại và nếu cần thì gợi ý bước tiếp theo. |
| Low-confidence - AI không chắc | System báo "không chắc" bằng cách nào? User quyết thế nào? | User chỉ nói "Evo200" hoặc "xe này". Hệ thống không đoán user muốn hỏi bảo hành, chẩn đoán hay đặt lịch, mà kích hoạt `_should_clarify_topic` để hỏi lại user muốn hỗ trợ gì với xe này. |
| Failure - AI sai | User biết AI sai bằng cách nào? Recover ra sao? | User hỏi giá bán, khuyến mãi, trả góp hoặc thông số xe. Hệ thống nhận là ngoài scope bằng `_should_reject_out_of_scope`, từ chối lịch sự và hướng user quay lại các chủ đề bảo hành/dịch vụ. |
| Correction - user sửa | User sửa bằng cách nào? Data đó đi vào đâu? | User nói "không, tôi muốn hỏi policy pin". Agent chuyển hướng sang `explain_warranty_policy(category="pin")`. Trong V1 correction chưa thành dataset riêng, nhưng có thể suy ra từ message history và tool log. |

### Feature 2: Chẩn đoán telemetry và gợi ý xưởng dịch vụ

**Trigger:** User hỏi về tình trạng xe, hao pin, mã lỗi, lốp non, nhiệt độ bất thường, hoặc muốn tìm xưởng gần nhất.

| Path | Câu hỏi thiết kế | Mô tả |
|------|-------------------|-------|
| Happy - AI đúng, tự tin | User thấy gì? Flow kết thúc ra sao? | User hỏi "Chẩn đoán xe này cho tôi". Agent gọi `diagnose_telemetry`, phân tích `odo_km`, `battery_soh_percent`, `charge_cycles`, `operating_temp_avg_c`, áp suất lốp và `last_error_codes`, sau đó tóm tắt mức độ `GOOD/INFO/WARNING/CRITICAL` và gợi ý đến xưởng nếu cần. |
| Low-confidence - AI không chắc | System báo "không chắc" bằng cách nào? User quyết thế nào? | User mô tả quá mơ hồ như "xe hơi khác". Nếu thiếu xe hoặc thiếu thông tin, agent hỏi lại xe nào hoặc khuyến nghị kiểm tra trực tiếp tại xưởng thay vì kết luận quá tay. |
| Failure - AI sai | User biết AI sai bằng cách nào? Recover ra sao? | Telemetry mock có thể không đủ để kết luận lỗi vật lý. Guardrail cấm AI khẳng định miễn phí cho lỗi vật lý và không đưa ra kết luận cuối cùng cho trường hợp cần kiểm tra trực tiếp. Recover bằng cách khuyến nghị đặt lịch kiểm tra. |
| Correction - user sửa | User sửa bằng cách nào? Data đó đi vào đâu? | User nói "tôi muốn tìm xưởng ở Hải Phòng, không phải Hà Nội". Agent đọc lại history, gọi `find_nearest_service_center` theo địa điểm mới và nếu user đã nói giờ cũ thì sẽ hỏi lại ngày/giờ do đã đổi location. |

### Feature 3: Đặt lịch, giữ chỗ 5 phút, xác nhận và đổi lịch

**Trigger:** User muốn bảo dưỡng, kiểm tra, sửa chữa, bảo hành pin hoặc đổi lịch cho một booking đã có.

| Path | Câu hỏi thiết kế | Mô tả |
|------|-------------------|-------|
| Happy - AI đúng, tự tin | User thấy gì? Flow kết thúc ra sao? | User chọn xe, nói thành phố, chọn xưởng, chọn ngày giờ. Agent gọi `find_nearest_service_center` nếu cần, sau đó `get_available_time_slots`, rồi `create_appointment`. Frontend hiện booking card `PENDING` với countdown 5 phút. User bấm `Xác nhận` -> `POST /api/booking/confirm` -> `CONFIRMED`. |
| Low-confidence - AI không chắc | System báo "không chắc" bằng cách nào? User quyết thế nào? | User chỉ nói "bảo dưỡng xe này". `_should_clarify_booking_details` sẽ ép hỏi location trước. Nếu user mới nói thành phố thì agent chỉ được liệt kê các xưởng. Chỉ khi user chọn xưởng cụ thể mới được hỏi ngày/giờ. |
| Failure - AI sai | User biết AI sai bằng cách nào? Recover ra sao? | Nếu slot đã hết hạn hoặc không còn trong, `create_appointment` hoặc `confirm_booking` trả lời thất bại. UI hiện booking hết hạn và mời user đặt lại. Hệ thống không được confirm ảo. |
| Correction - user sửa | User sửa bằng cách nào? Data đó đi vào đâu? | User muốn đổi lịch. Agent có thể dùng `lookup_my_bookings` để tìm booking, `get_available_time_slots` để lấy slot mới, rồi `reschedule_appointment` để cập nhật. Dữ liệu booking được ghi đè vào `bookings.csv`, đồng thời cập nhật state `_bookings` và `_time_slots`. |

---

## 3. Eval metrics + threshold

**Optimize precision hay recall?** [x] Precision | [ ] Recall
Tại sao? Bài toán này liên quan đến trust, policy và booking thật. Trả lời sai nhưng tự tin cao nguy hiểm hơn việc hỏi lại thêm 1 lần.
Nếu sai ngược lại thì chuyện gì xảy ra? Nếu chọn precision quá cao thì agent có thể hỏi lại nhiều, flow booking dài hơn và conversion giảm. Nhưng đây vẫn an toàn hơn việc tự ý booking sai hoặc over-promise warranty.

| Metric | Threshold | Red flag (dừng khi) |
|--------|-----------|---------------------|
| Factual safety / policy-grounded response rate | >= 95% trên tập test nội bộ | < 85% hoặc có 1 case over-promising nghiêm trọng |
| Booking consistency (`AVAILABLE -> PENDING -> CONFIRMED/EXPIRED`) | 100% trong test create/confirm/reschedule | Bất kỳ trường hợp confirm ảo, slot bị double-book hoặc mismatch state |
| Clarification-before-action rate | >= 95% cho case thiếu xe, thiếu location, thiếu ngày giờ | < 85%, agent tự ý chọn xe/xưởng/giờ cho user |
| P95 chat latency | < 3s happy path, < 5s với tool chain dài | > 6s ổn định hoặc loop tool |
| Booking confirmation rate sau khi AI đề xuất slot | >= 35% trong pilot hợp lý | < 15% trong 2 chu kỳ liên tiếp |
| Regression test pass rate | 100% cho test clarification, slot generation, reschedule, CSV persistence | Có test fail sau mỗi lần sửa logic booking/clarification |

---

## 4. Top 3 failure modes

| # | Trigger | Hậu quả | Mitigation |
|---|---------|---------|------------|
| 1 | AI trả lời sai quyền lợi bảo hành hoặc hàm ý miễn phí cho lỗi vật lý | User đến xưởng với kỳ vọng sai, dễ tranh chấp và mất trust | Guardrail trong `SYSTEM_PROMPT`, chỉ được dựa vào tool/data trong repo, không kết luận miễn phí cho lỗi vật lý, fallback hotline `1900 23 23 89` |
| 2 | AI tự ý chọn xưởng, ngày hoặc khung giờ khi user chưa cung cấp đủ thông tin | Booking sai nhu cầu, user bỏ flow, giảm trust | `_should_clarify_booking_details`, booking rule trong prompt, và test cho các case "chỉ nói thành phố", "chỉ nói xe này", "đổi địa điểm" |
| 3 | Slot availability không nhất quán giữa lúc gợi ý và lúc xác nhận | User thấy booking card nhưng bấm xác nhận thất bại | State machine với `_lock`, `hold_slot`, TTL worker, `confirm_booking` re-check trạng thái, frontend countdown và thông báo hết hạn rõ ràng |

Failure mode nguy hiểm nhất mà user không để ý ngay là câu trả lời sai policy nhưng nghe hợp lý. Vì vậy quality bar của sản phẩm phải ưu tiên grounded response hơn là trả lời "mượt" cho mọi tình huống.

---

## 5. ROI 3 kịch bản

|   | Conservative | Realistic | Optimistic |
|---|-------------|-----------|------------|
| **Assumption** | 50 hội thoại/ngày, 20% có ý định đặt lịch, 20% booking completion | 150 hội thoại/ngày, 30% có ý định đặt lịch, 35% booking completion | 400 hội thoại/ngày, 35% có ý định đặt lịch, 45% booking completion |
| **Cost** | ~150k-300k VND/ngày cho inference `gpt-4o-mini`, hosting nhẹ, monitoring cơ bản | ~400k-700k VND/ngày | ~1.2M-2.0M VND/ngày |
| **Benefit** | Giảm 1-2 giờ hotline/ngày, giảm câu hỏi lặp lại về policy, có thêm booking từ kênh web | Giảm 4-6 giờ hotline/ngày, booking dịch vụ tăng 15-25%, CSKH tập trung vào case khó | Giảm 10+ giờ hotline/ngày, booking tăng 30-50%, mở rộng được sang nhắc bảo dưỡng và retention |
| **Net** | Dương nhẹ nếu support cost hiện tại đang cao và user có nhiều câu hỏi lặp lại | Dương rõ ràng nếu booking uplift và hotline reduction đạt mức realistic | Rất tốt nếu được nối vào dữ liệu thật, slot real-time và scale cho nhiều user |

**Kill criteria:** Dừng nếu booking consistency không đạt 100%, có case over-promising nghiêm trọng, factual safety < 85%, booking completion < 15% sau pilot, hoặc chi phí vận hành vượt lợi ích quy đổi trong 2 chu kỳ liên tiếp.

---

## 6. Mini AI spec (1 trang)

VinBot là một AI service agent cho chủ xe máy điện VinFast đã đăng nhập. Product giải 3 nhu cầu rõ ràng: tra cứu bảo hành theo từng xe cụ thể, giải thích policy và lịch bảo dưỡng, chẩn đoán sơ bộ từ telemetry, và hỗ trợ đặt/đổi lịch tại xưởng. Đây là bài toán augmentation, không phải automation full. User vẫn giữ quyền quyết định và xác nhận booking cuối cùng trên giao diện.

V1 trong repo `lab6_B3` đã có backend FastAPI (`app/main.py`), agent LangGraph (`app/agent.py`), business tools (`app/tools.py`), state/data layer (`app/data.py`) và frontend web (`static/index.html`, `static/app.js`). Agent hiện có 8 tool chính: `lookup_warranty_status`, `explain_warranty_policy`, `diagnose_telemetry`, `find_nearest_service_center`, `get_available_time_slots`, `create_appointment`, `lookup_my_bookings`, `reschedule_appointment`. Frontend đã có vehicle selector, chat UI, booking card và countdown timer cho booking pending.

Data hiện tại gồm `vehicles.json` với 6 xe mock của user `U_VIN_001`, `warranty_policy.json` version `2025-08-15`, `service_centers.json` với 10 xưởng tại Hà Nội, TP.HCM, Đà Nẵng, Hải Phòng, Cần Thơ, Đồng Nai, và `bookings.csv` để persist lịch hẹn. Phần state vận hành được quản lý bằng `_time_slots` và `_bookings`, theo machine `AVAILABLE -> PENDING -> CONFIRMED`, với nhánh `PENDING -> EXPIRED` sau 300 giây.

Quality bar của hệ thống là precision-first: chỉ trả lời trong scope warranty/diagnostics/booking, hỏi lại nếu thiếu xe hoặc thiếu thông tin booking, không tự ý chọn slot thay user, và không bao giờ thông báo thành công nếu backend chưa xác nhận state hợp lệ. Các test trong `tests/test_agent_topic_clarification.py` đã cover các tình huống quan trọng như clarify topic, clarify booking, relative date resolution, slot generation theo giờ làm việc, CSV persistence và reschedule.

Data flywheel cho các vòng sau nên tập trung vào logging bài bản hơn: lưu correction, tỷ lệ clarification, booking conversion, booking expiration, lý do reschedule và explicit feedback cho câu trả lời. Rủi ro lớn nhất hiện nay vẫn là mock data, lack of real-time VinFast integration, và chưa có policy source-of-truth bên ngoài JSON nội bộ. Bước tiếp theo hợp lý là thêm analytics, feedback log, red-team test cho policy safety, và nối dữ liệu slot/service center thực.
