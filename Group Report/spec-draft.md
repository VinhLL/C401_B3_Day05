# SPEC draft - VinFast Warranty Service Agent

## Track: VinFast Warranty Service Agent

## Problem statement
Người dùng VinFast sau khi đăng nhập thường có nhiều xe nhưng khó tra cứu nhanh tình trạng bảo hành, lịch sử sửa chữa, linh kiện còn hạn và nhu cầu bảo dưỡng; hiện họ phải gọi hotline, hỏi showroom hoặc tự tìm thủ công, mất thời gian và dễ hiểu sai chính sách. AI có thể dựa trên chiếc xe người dùng đang chọn để tra cứu dữ liệu bảo hành, chẩn đoán sơ bộ từ telemetry và gợi ý xưởng phù hợp hoặc đặt lịch kiểm tra.

## Canvas draft

| | Value | Trust | Feasibility |
|---|---|---|---|
| **Câu hỏi guide** | User nào? Pain gì? AI giải quyết gì? | Khi AI sai thì sao? Sửa thế nào? | Cost? Latency? Risk? |
| **Trả lời** | User chính là chủ xe máy điện VinFast như Evo200, Feliz S, thường là học sinh, sinh viên, người đi làm nội thành. Pain lớn là khó hiểu chính sách bảo hành theo dòng xe, loại pin, hợp đồng mua/thuê pin; khi xe tụt pin nhanh hoặc có tiếng kêu bất thường họ không biết có được bảo hành miễn phí không và nên mang ra xưởng nào. AI giúp tra cứu nhanh theo xe đang chọn, đọc ODO, SOH pin, số lần sạc, nhiệt độ vận hành và lịch sử sửa chữa để trả lời tình trạng bảo hành, số lần bảo hành, gợi ý bảo dưỡng và tìm xưởng gần nhất. | Failure mode chính là bịa chính sách, nhầm giữa các phiên bản pin, bị prompt injection, hoặc hứa quá quyền hạn như xác nhận đền bù, thay miễn phí hay chốt giờ cứng tại xưởng. Hậu quả là user mang xe đến xưởng và bị từ chối, gây mất niềm tin và rủi ro truyền thông. Cách giảm rủi ro: luôn trích nguồn chính sách, chặn cam kết tài chính/quà tặng, với lỗi vật lý hoặc case mơ hồ phải yêu cầu kiểm tra tại xưởng, luôn có fallback `Gửi Ticket Kỹ Thuật` và `Kết nối CSKH`. | Cost thấp vì tài liệu bảo hành xe máy điện khá hẹp, phù hợp RAG + rule guardrail. Latency mục tiêu < 3 giây để phù hợp tệp user trẻ. Risk chính là AI không phân biệt minor updates của cùng model, dữ liệu lịch xưởng không real-time, và mô hình NLU hiểu sai từ lóng hoặc mô tả chung chung. Cần red-team prompt injection định kỳ và check booking bằng transaction/lock ở backend. |

## Automation hay augmentation?

- `Augmentation` - AI chỉ phân tích dữ liệu, giải thích chính sách, gợi ý đặt lịch và gợi ý xưởng.
- User là người quyết định cuối cùng: có mang xe đi xưởng không, có đồng ý thay linh kiện không, có xác nhận booking không.
- `Cost of reject = 0`: nếu AI gợi ý chưa đúng, user có thể bỏ qua; không để AI xử lý, tự trừ tiền hay tự chốt quyền lợi bảo hành.

## Learning signal

| # | Câu hỏi | Trả lời |
|---|---|---|
| 1 | User correction đi vào đâu? | Khi user bác gợi ý của AI hoặc kỹ thuật viên kết luận khác với `ai_diagnosis_log`, hệ thống lưu correction log dạng `User nói X -> AI đoán Y -> Thực tế/Z`. Log này được gắn nhãn để cải thiện NLU và mô hình predictive maintenance cho các xe cùng dòng. |
| 2 | Product thu signal gì để biết tốt lên hay tệ đi? | Implicit: tỷ lệ bấm `Đặt lịch ngay`, `Tìm xưởng gần nhất`, tỷ lệ thoát sang hotline/cứu hộ. Explicit: thumbs up/down sau câu trả lời. Correction: độ lệch giữa chẩn đoán của AI và biên bản nghiệm thu thực tế của kỹ thuật viên. |
| 3 | Data thuộc loại nào? | `User-specific`: lịch sử sửa chữa, hợp đồng mua/thuê pin, xe đang chọn. `Domain-specific`: chính sách bảo hành, sổ tay HDSD, danh mục linh kiện. `Real-time`: ODO, SOH pin, nhiệt độ pin, cảnh báo lỗi, trạng thái slot đặt lịch. `Human-judgment`: biên bản nghiệm thu và xác nhận từ kỹ thuật viên xưởng. |

**Marginal value:** Có. Dữ liệu kết hợp giữa telemetry thực tế của xe, policy nội bộ và kết luận kỹ thuật viên tạo thành feedback loop khó thay thế bằng chatbot FAQ thông thường.

## User stories - 4 paths

### Path 1 - Happy path
- User hỏi: "Pin xe tôi còn bảo hành không?"
- AI nhận diện đúng intent tra cứu bảo hành, đúng xe đang chọn, đúng loại pin và loại hợp đồng.
- Output mẫu: pin LFP của Evo200 còn hạn đến ngày cụ thể, lần kiểm tra gần nhất là khi nào, và nếu pin tụt nhanh thì có thể đặt lịch kiểm tra tại xưởng.
- Value moment là user biết ngay quyền lợi và bước tiếp theo mà không cần gọi hotline.

### Path 2 - Low-confidence path
- User mô tả mơ hồ như "đầu xe kêu lạch cạch" hoặc dùng từ lóng như "phuộc xì nhớt".
- AI map được một phần ý nghĩa nhưng confidence thấp vì có nhiều linh kiện khả dĩ hoặc user chưa nói rõ trước/sau.
- Thay vì kết luận, AI chuyển sang suggestion UI: yêu cầu user chọn đúng bộ phận như `Cổ phốt/Chảng ba`, `Phuộc trước`, `Phuộc sau`, `Nhựa ốp đầu xe`.
- Hệ thống chỉ tra cứu hoặc gợi ý tiếp sau khi user xác nhận.

### Path 3 - Over-promising path
- User hỏi case nhạy cảm như vỡ màn hình do ngã xe hoặc đặt lịch tại một khung giờ cụ thể.
- AI không được hứa "thay miễn phí" hoặc "đã đặt thành công 14:00" theo kiểu tuyệt đối.
- UX phải dùng ngôn ngữ mở: lịch là dự kiến, thời gian phục vụ thực tế có thể xê dịch; lỗi do tác động vật lý cần kỹ thuật viên kiểm tra trực tiếp.
- Luôn có guardrail để chặn cam kết tài chính, quà tặng, đền bù và quyền lợi ngoài policy.

### Path 4 - User mất kiên nhẫn
- Sau 2 lần AI hiểu sai, user muốn bỏ chatbot.
- Hệ thống phải luôn cho lối thoát: `Tự chọn linh kiện thủ công` và `Kết nối nhân viên CSKH VinFast`.
- Thao tác sửa tay của user trở thành explicit correction signal để huấn luyện lại NLU.
- Mục tiêu là graceful failure: không nhốt user trong vòng lặp chat.

## Eval metrics

### Product & business metrics

| Metric | Target | Signal | Ý nghĩa |
|---|---|---|---|
| Task Success / Acceptance Rate | > 40% | Tỷ lệ user bấm `Đặt lịch ngay` hoặc `Tìm xưởng gần nhất` sau tư vấn | Đo AI có tạo ra value và thúc đẩy hành động hay không |
| Human Escalation Rate | < 15% | Tỷ lệ bấm `Kết nối nhân viên CSKH` hoặc `Tự chọn linh kiện thủ công` | Đo mức độ mất kiên nhẫn và mức under-trust |
| CSAT | > 4.0/5.0 | Thumbs up/down, rating sau câu trả lời | Đo mức hài lòng với cách AI giải thích và hỗ trợ |
| Booking Uplift | > 15% ở kịch bản realistic | So sánh số booking trước/sau rollout | Đo tác động business trực tiếp |

### AI & technical metrics

| Metric | Target | Signal | Ý nghĩa |
|---|---|---|---|
| Diagnosis Accuracy | > 85% | So sánh `ai_diagnosis_log` với biên bản nghiệm thu thực tế | Thước đo quan trọng nhất cho chất lượng chẩn đoán |
| Hallucination / Over-promising Rate | 0% strict | Log các lần AI bịa policy, hứa bảo hành sai, chốt giờ cứng | Guardrail bắt buộc để tránh khủng hoảng trust |
| Response Latency | < 3 giây | Thời gian từ user input đến AI output | Giữ trải nghiệm nhanh, phù hợp mobile-first |
| Slot Booking Consistency | 100% | Tỷ lệ booking confirm khớp slot backend đã commit | Ngăn xác nhận ảo và overbooking |

## Failure modes

| Khi nào xảy ra | Hậu quả | Giải pháp |
|---|---|---|
| AI xác nhận lịch nhưng backend chưa giữ chỗ thật | User đến xưởng bị từ chối, mất niềm tin | Soft-lock slot trước khi gợi ý, chỉ confirm khi transaction commit thành công |
| Dữ liệu lịch xưởng cũ hoặc không real-time | Trùng lịch, overbooking | Re-check slot trong DB khi bắt đầu flow đặt lịch và trước khi confirm |
| AI hiểu sai ý định người dùng | Tạo booking hoặc tra cứu sai linh kiện | Hỏi lại bằng suggestion UI trước khi hành động |
| AI bịa chính sách hoặc bị prompt injection | Hứa sai quyền lợi, rủi ro truyền thông | RAG + cited policy, rule guardrail, chặn cam kết tài chính/quà tặng |
| AI suy diễn lỗi vật lý là lỗi bảo hành | User đến xưởng và tranh cãi với nhân viên | Với từ khóa nhạy cảm như `vỡ`, `gãy`, `đổ`, bắt buộc thêm điều kiện "cần kiểm tra vật lý tại xưởng" |

## ROI - 3 kịch bản

Giả định hiện tại công ty có khoảng 10 nhân sự CSKH hỗ trợ hotline và đặt lịch.

| Kịch bản | Assumption | Tác động dự kiến |
|---|---|---|
| Conservative | Task success 25-30%, escalation ~25%, CSAT 3.5-3.8, accuracy 65-70% | Giảm tải hotline 10-15%, booking tăng 3-5%, tiết kiệm tương đương 1-2 nhân sự support |
| Realistic | Task success 40-50%, escalation 10-15%, CSAT > 4.0, accuracy 80-85% | Hotline giảm 30-40%, booking tăng 15-25%, tiết kiệm 30-40% chi phí support và tăng lợi nhuận nhờ booking |
| Optimistic | Task success > 60%, escalation < 5%, CSAT > 4.5, accuracy > 90%, latency < 2s | Hotline gần như chỉ còn case khó, booking tăng 40-60%, có thể upsell gói dịch vụ, team support tinh gọn mạnh |

**Kill criteria sau 8 tuần**

- Diagnosis Accuracy < 65%
- Task Success < 30%
- Escalation Rate > 25%
- Booking uplift < 5%
- Cost per user > Revenue per user

Nếu vi phạm từ 2 điều kiện trở lên thì nên kill hoặc pivot.

## Mini AI spec

### Scope V1
- Tra cứu tình trạng bảo hành theo xe đang chọn.
- Giải thích quyền lợi pin và linh kiện theo policy hiện hành.
- Chẩn đoán sơ bộ từ telemetry: ODO, SOH pin, số lần sạc, nhiệt độ vận hành, mã lỗi.
- Gợi ý xưởng gần nhất và tạo lịch kiểm tra/bảo dưỡng.

### Core data
- `xuong_dich_vu(center_id, name, address, hotline, capacity)`
- `lich_hen_bao_duong(booking_id, user_id, vin_number, center_id, booking_date, time_slot, ai_diagnosis_log, service_type, status)`
- Policy/RAG sources: CSBH xe máy điện, sổ tay HDSD, danh mục linh kiện, mapping từ lóng -> linh kiện chuẩn
- Telemetry sources: ODO, SOH pin, nhiệt độ, số lần sạc, cảnh báo lỗi, lịch sử sửa chữa

### Booking flow
- Slot có 3 trạng thái: `AVAILABLE`, `PENDING`, `CONFIRMED`.
- Khi AI gợi ý một slot, backend atomic update từ `AVAILABLE -> PENDING` và gắn TTL 5 phút.
- Nếu user xác nhận trong TTL, backend commit booking và chuyển sang `CONFIRMED`.
- Nếu hết TTL mà user không chốt, worker tự trả slot về `AVAILABLE`.
- UI nên hiển thị đồng hồ đếm ngược và câu nhắn kiểu: "Em sẽ giữ chỗ này cho anh trong 5 phút."

### Guardrails
- Không cam kết tài chính, quà tặng, đền bù, đổi xe mới.
- Không tự xác nhận miễn phí với lỗi vật lý hoặc case cần kiểm tra trực tiếp.
- Không chốt giờ phục vụ như cam kết cứng; chỉ xác nhận booking khi backend commit thành công.
- Luôn có fallback sang ticket kỹ thuật hoặc CSKH người thật.

### Output mẫu

```json
{
  "booking_id": "BK_001",
  "user_id": "U_VIN_001",
  "vin_number": "EVO200_9999",
  "center": "VinFast Ocean Park",
  "booking_date": "2026-04-10",
  "time_slot": "08:30 AM",
  "ai_diagnosis_log": "Xe hao điện bất thường. Xác nhận lốp non hơi 1.8 bar. Khuyến nghị KTV kiểm tra áp suất lốp vành trước.",
  "status": "CONFIRMED"
}
```

## Phân công nhiệm vụ

- Lê Quang Minh: AI Product Canvas
- Ngô Quang Phúc: User Stories x 4 paths
- Nguyễn Việt Hoàng: Eval metrics - 3 metrics + threshold + red flag
- Trần Quốc Việt: Top 3 failure modes
- Nguyễn Bình Minh: ROI 3 kịch bản
- Bùi Quang Vinh: Mini AI spec 1 trang
