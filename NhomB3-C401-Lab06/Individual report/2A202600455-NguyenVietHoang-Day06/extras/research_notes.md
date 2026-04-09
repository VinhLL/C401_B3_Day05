# Research Notes - Business and System Analysis

## 1) Problem framing
- Bai toan khong chi la chatbot tra loi, ma la tro ly co the dan user den hanh dong dat lich hop le.
- Rui ro lon nhat: AI tra loi dung ngon ngu nhung sai trang thai he thong (booking, slot, warranty scope).

## 2) Metrics de theo doi
### Product/Business
- Task Success Rate: user chot dat lich/tim xưởng sau tu van.
- Human Escalation Rate: user bo qua AI de chuyen CSKH.
- CSAT: danh gia sau hoi thoai.

### AI/Technical
- Diagnosis Accuracy.
- Hallucination/Over-promising Rate (muc tieu ~0).
- Response Latency.

## 3) Failure modes uu tien
- Confirm booking dung luc TTL ve 0.
- Session stale sau reload tab.
- Slot conflict khi nhieu request cung luc.
- Policy misquote trong tinh huong borderline.

## 4) Huong giai quyet
- State machine ro rang voi transition constraints.
- TTL worker + lock cho thao tac nhay cam.
- Contract response thong nhat de frontend render khong sai.
- Test matrix gom happy path + edge cases + concurrent scenarios.
