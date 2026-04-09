# Individual reflection — Bùi Quang Vinh (2A202600007)

## 1. Role
Tool Logic Owner. Phụ trách lớp nghiệp vụ trong [tools.py](/d:/Work/lab6_B3/app/tools.py), làm cầu nối giữa agent và data layer để các tool trả về dữ liệu đúng schema, đúng trạng thái, và đủ an toàn cho flow đặt lịch hoặc đổi lịch.

## 2. Đóng góp cụ thể
- Chuẩn hóa và hoàn thiện các tool chính trong [tools.py](/d:/Work/lab6_B3/app/tools.py): `lookup_warranty_status`, `explain_warranty_policy`, `diagnose_telemetry`, `find_nearest_service_center`, `get_available_time_slots`, `create_appointment`, `reschedule_appointment`, `lookup_my_bookings`.
- Thiết kế phần `TOOL_DEFINITIONS`, `TOOL_MAP` và `execute_tool()` ngay trong [tools.py](/app/tools.py) để agent gọi tool nhất quán, có schema rõ ràng và output dễ dùng lại trong LangGraph flow.
- Xử lý edge case nghiệp vụ quan trọng: xe không tồn tại, xưởng không tồn tại, slot không còn khả dụng, booking không hợp lệ khi đổi lịch, và đảm bảo reschedule cập nhật đúng booking cũ thay vì tạo booking mới.

## 3. SPEC mạnh/yếu
- Phần mạnh nhất của SPEC là failure modes và booking consistency. Nhóm xác định khá rõ các rủi ro như over-promising bảo hành, xác nhận booking ảo, hoặc gợi ý slot không còn khả dụng. Điều này rất sát với phần tool vì tool là nơi biến các rule đó thành kiểm tra nghiệp vụ cụ thể.
- Phần yếu nhất là measurement sau triển khai, đặc biệt là learning signal và ROI thực tế. Spec có nêu conversion, expired booking và escalation, nhưng chưa mô tả rõ cách log từng tool failure hoặc cách phân loại nguyên nhân fail theo từng loại nghiệp vụ như `vehicle_not_found`, `slot_unavailable`, `booking_not_found`, `reschedule_conflict`.

## 4. Đóng góp khác
- Hỗ trợ debug các case agent gọi sai tool hoặc sai thứ tự nghiệp vụ, đặc biệt ở flow xác nhận slot ngắn và flow đổi lịch.
- Hỗ trợ kiểm tra các test liên quan đến booking và reschedule trong để bảo đảm output của tool khớp với kỳ vọng của agent và frontend.
- Tham gia thực hiện demo bên cạnh trình chiếu slide và giải thích agent hoạt động

## 5. Điều học được
Trước hackathon, tôi nghĩ tool chỉ là lớp wrapper kỹ thuật để model gọi hàm. Sau khi làm phần này, tôi hiểu tool thực chất là nơi giữ các quy luật giữa AI và hệ thống thật. Nếu output tool không chặt, naming không nhất quán, hoặc thiếu kiểm tra trạng thái, thì agent có prompt tốt vẫn dễ trả lời sai hoặc dẫn user vào flow sai. Nói cách khác, chất lượng AI product không chỉ đến từ prompt mà còn đến từ chất lượng contract của tools.

## 6. Nếu làm lại
Nếu làm lại, tôi sẽ tách sớm hơn bộ test riêng cho tools thay vì dồn phần lớn regression vào test agent. Cách làm hiện tại vẫn bắt được nhiều bug, nhưng khi một test fail thì khó biết lỗi nằm ở agent reasoning, tool output hay data state. 
Thực hiện test nhiều hơn để đảm bảo việc hoạt động test được trơn tru

## 7. AI giúp gì / AI sai gì
- **Giúp:** AI hỗ trợ brainstorm edge cases khá tốt, nhất là các case booking conflict, slot hết hạn, và sự khác nhau giữa `create_appointment` với `reschedule_appointment`. AI cũng giúp soạn nhanh schema mô tả tool và gợi ý cách chuẩn hóa output dict để agent đọc dễ hơn.
- **Sai/mislead:** AI có xu hướng đề xuất abstraction hơi quá tay, ví dụ tách thêm một file adapter riêng cho tool schema dù codebase này còn nhỏ. Điều đó nghe có vẻ clean nhưng lại làm repo khó hiểu hơn với nhóm. Ngoài ra AI đôi lúc còn gợi ý fallback sang tạo booking mới khi reschedule thiếu context; nếu làm theo mà không kiểm tra kỹ sẽ gây lỗi nghiệp vụ nghiêm trọng. Bài học của tôi là AI hữu ích khi brainstorm và tăng tốc, nhưng mọi quyết định ở lớp tool phải được kiểm tra theo business rule thực tế chứ không thể tin hoàn toàn vào gợi ý sinh ra từ AI.
