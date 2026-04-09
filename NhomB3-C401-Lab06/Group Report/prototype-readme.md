# Prototype — AI Warranty Agent VinFast (VinBot)

## Mô tả
Chatbot đóng vai trò trợ lý ảo (VinBot) hỗ trợ mảng bảo hành xe máy điện cho khách hàng VinFast. Chức năng chính bao gồm: 
- Nhận diện thông tin người dùng và xe (VIN, ODO, Battery SOH).
- Kiểm tra tính hợp lệ của chính sách bảo hành.
- Chẩn đoán sơ bộ các mã lỗi xe.
- Tra cứu danh sách xưởng dịch vụ gần nhất.
- Hỗ trợ đặt lịch khám/bảo dưỡng xe với luồng xác nhận qua tin nhắn.

## Level: Working prototype
- **UI:** Xây dựng bằng HTML/CSS/JS thuần (Vanilla) tích hợp giao diện Chat hiện đại.
- **Backend:** Xây dựng bằng FastAPI (Python) phục vụ API và Static Files.
- **Flow chính chạy thật:** Người dùng nhập yêu cầu → AI phân tích intent → Gọi các Function Tool tương ứng lấy dữ liệu thực từ JSON/CSV → AI tổng hợp câu trả lời tự nhiên kèm action (VD: Nút xác nhận đặt lịch) → Hiển thị kết quả lên UI.

## Links
- **Source Code / Folder:** Nằm toàn bộ trong thư mục hiện tại.
- **Local Application:** Sau khi chạy bằng `uvicorn app.main:app --reload`, truy cập vào `http://127.0.0.1:8000`
- **Data (Mock):** Các file `.json` và `.csv` tại thư mục `data/` mô phỏng Database.
- **Hướng dẫn gốc:** Xem `README.md` cũ trong thư mục.

## Tools
- **UI:** HTML5, Vanilla JavaScript, CSS3.
- **Backend:** FastAPI, Uvicorn, Python.
- **AI Core:** OpenAI API (LLM) hỗ trợ Function Calling.
- **Framework:** Thư viện Pydantic, Python Dotenv.
- **Prompt & Logic:** System prompt kết hợp với function tool schema (định nghĩa các tools như tra cứu bảo hành, đặt lịch, tìm xưởng) thay vì chỉ prompt chay.

## Phân công

| Thành viên     | Phần                                                                                                         | Output                                                                            |
| -------------- | ------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------- |
| Quang Minh     | Backend API (FastAPI endpoints, response schema, error handling, CORS, integration test)                     | `app/main.py`, `tests/test_api.py`, `README.md`, `walkthrough.md`                 |
| Bình Minh      | Agent Core (LangGraph flow, routing, clarification logic, booking flow, prompt/system message)               | `app/agent.py`, `tests/test_agent_topic_clarification.py`                         |
| Vinh           | Tool Logic (business logic tools, TOOL_MAP, execute_tool, edge cases)                                        | `app/tools.py`, `tests/test_tools.py`                                             |
| Hoàng          | Data & State Machine (in-memory store, slot generation, TTL, booking state machine, race condition handling) | `app/data.py`, `data/*.json`, `tests/test_data.py`, `tests/test_booking_state.py` |
| Việt           | Frontend UX Flow (chat UI, booking card, countdown, localStorage, network error UX)                          | `static/index.html`, `static/app.js`, `static/style.css`                          |
| Ngô Quang Phúc | QA + E2E + Release (test matrix, regression, demo script, release checklist)                                 | `tests/*`, `README.md`, `walkthrough.md`, `implementation_plan.md`                |
