# Individual Reflection — Nguyễn Việt Hoàng (2A202600455)

## 1. Role
Data & State Machine Owner, đồng thời phụ trách thêm phần phân tích nghiệp vụ (business analysis) cho flow bảo hành/đặt lịch.

## 2. Đóng góp cụ thể
- Thiết kế và hoàn thiện lớp dữ liệu trung tâm trong `app/data.py`: load dữ liệu seed, quản lý trạng thái slot, và đồng bộ vòng đời booking.
- Xây dựng state machine booking theo 4 trạng thái `AVAILABLE -> PENDING -> CONFIRMED -> EXPIRED`, đảm bảo logic hold/confirm/reschedule nhất quán.
- Triển khai cơ chế TTL worker để tự động hết hạn booking `PENDING`, giải phóng slot đúng hạn và hạn chế ghost booking.
- Bổ sung cơ chế lock ở các thao tác nhạy cảm (hold, confirm, reschedule, expire) để giảm nguy cơ race condition khi có nhiều request gần thời điểm nhau.
- Quản lý dữ liệu seed và runtime:
  - Seed: `data/vehicles.json`, `data/service_centers.json`, `data/warranty_policy.json`
  - Runtime: `data/bookings.csv` (không commit mặc định, chỉ dùng khi cần dữ liệu demo có chủ đích)
- Phối hợp với phần agent/tool để đảm bảo output từ data layer đủ thông tin cho phản hồi nhất quán ở frontend.

## 3. SPEC mạnh/yếu
- **Mạnh nhất:** định nghĩa luồng booking theo trạng thái và TTL rõ ràng, giúp hệ thống tránh over-booking và phản ánh đúng thực tế vận hành.
- **Mạnh thứ hai:** phân tách seed data với runtime data giúp dễ tái lập môi trường test/demo và tránh lỗi dữ liệu bẩn.
- **Yếu nhất:** vẫn còn vùng rủi ro ở tình huống xác nhận booking sát thời điểm TTL về 0; cần test concurrent dày hơn để chứng minh tính ổn định trong tải thực.

## 4. Đóng góp phân tích nghiệp vụ
- Xây dựng góc nhìn đo lường theo 2 lớp:
  - **Product/Business metrics:** Task Success (đặt lịch/tìm xưởng sau tư vấn), Human Escalation Rate, CSAT.
  - **AI/Technical metrics:** Diagnosis Accuracy, Hallucination/Over-promising Rate, Response Latency.
- Liên kết metric với hành vi người dùng (implicit + explicit signals) để team có cơ sở đánh giá không chỉ "chạy được" mà còn "tạo giá trị".
- Đề xuất dùng các failure mode như booking expiry, policy misquote, và confirm conflict làm tín hiệu ưu tiên cho vòng lặp cải tiến tiếp theo.

## 5. Điều học được
Trước khi làm dự án, tôi nghĩ data layer chỉ là phần "lưu và trả dữ liệu". Sau khi triển khai state machine + TTL, tôi nhận ra data layer thực chất là nơi giữ tính đúng đắn của sản phẩm: nếu trạng thái không chặt hoặc chuyển trạng thái không nguyên tử, AI trả lời hay đến đâu vẫn có thể dẫn user vào trải nghiệm sai.

Bài học thứ hai là phân tích nghiệp vụ cần đi song song với code. Nếu không định nghĩa metric từ sớm, nhóm rất khó biết thay đổi nào thực sự cải thiện trải nghiệm người dùng và hiệu quả vận hành.

## 6. Nếu làm lại
- Viết test concurrency và TTL sớm hơn, đặc biệt cho kịch bản confirm gần mốc hết hạn.
- Thêm bộ test tách riêng cho state transitions (table-driven test cho từng trạng thái và sự kiện).
- Thiết kế sẵn dashboard log tối thiểu cho các chỉ số nghiệp vụ quan trọng để theo dõi hằng ngày thay vì tổng hợp thủ công.

## 7. AI giúp gì / AI sai gì
- **Giúp:** AI hỗ trợ brainstorm edge cases cho state machine (double confirm, stale pending, reschedule conflict) và gợi ý khung metric để đánh giá hiệu quả sản phẩm.
- **Sai/mislead:** AI có xu hướng đề xuất giải pháp nặng (stream/event bus, realtime sync phức tạp) vượt quá scope hackathon. Bài học là dùng AI để mở rộng ý tưởng, nhưng quyết định cuối cùng phải bám chặt business constraint, thời gian, và độ phức tạp triển khai.

## 8. Extras (Bonus submission)
Các tài liệu nộp thêm để lấy bonus được đặt trong thư mục `extras/`:
- `extras/prompt_test_logs.md`: log test prompt theo từng nhóm tình huống (booking, TTL, reschedule, out-of-scope).
- `extras/research_notes.md`: ghi chú phân tích nghiệp vụ, metric và failure modes.
- `extras/design_iterations.md`: các vòng chỉnh sửa thiết kế state machine và API/data contract.
- `extras/ai_conversation_notes.md`: tóm tắt các trao đổi với AI, phần nào được áp dụng và phần nào bị loại.

Các file này nhằm chứng minh quá trình làm việc cá nhân và quyết định kỹ thuật theo từng iteration, không chỉ nộp kết quả cuối.
