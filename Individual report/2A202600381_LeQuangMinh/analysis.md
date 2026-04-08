# 📊 UX Exercise — MoMo Moni AI

## 🧾 Sản phẩm
**MoMo — Moni AI Assistant**  
(Tính năng: Tự động phân loại chi tiêu)

---

## 🔍 4 Paths

### 🟢 1. AI đúng

**Kịch bản:**
- User chi tiêu 50k tại Circle K → Moni gợi ý "Ăn uống"

**Trải nghiệm:**
- User thấy tag đúng, không cần làm gì thêm

**UI:**
- Hiện tag + icon category
- Không hỏi confirm

**Đánh giá UX:**
- Tốt: Giảm thao tác thừa, trải nghiệm mượt mà

---

### 🟡 2. AI không chắc

**Kịch bản:**
- User chuyển tiền 200k cho bạn → Moni không tag hoặc tag "Khác"

**Trải nghiệm:**
- Không có gợi ý
- User phải tự vào chỉnh

**UI:**
- Không hiện prompt / suggestion

**Vấn đề:**
- Không có cơ chế:
  > "Bạn muốn phân loại giao dịch này không?"

**Đánh giá UX:**
- Kém: Thiếu chủ động hỗ trợ user

---

### 🔴 3. AI sai

**Kịch bản:**
- User mua sách trên Shopee → Moni tag "Mua sắm" thay vì "Học tập"

**Trải nghiệm:**
- User phát hiện khi xem báo cáo tháng
- Sửa bằng cách:
  1. Vào chi tiết giao dịch  
  2. Đổi category  
  (3 bước)

**UI:**
- Không có thông báo sai
- Không có xác nhận sau khi sửa

**Vấn đề:**
- Không rõ AI có học từ correction này không

**Đánh giá UX:**
- Yếu: Quy trình sửa lỗi mất nhiều bước, thiếu phản hồi

---

### ⚫ 4. User mất niềm tin

**Kịch bản:**
- Sau nhiều lần tag sai, user không tin auto-tag nữa

**Trải nghiệm:**
- Muốn kiểm soát thủ công

**UI:**
- Không có option:
  - Tắt auto-tag  
  - Tag thủ công trước  

**Vấn đề:**
- Không có fallback rõ ràng ngoài việc sửa từng giao dịch

**Đánh giá UX:**
- Nghiêm trọng: Thiếu lối thoát cho user

---

## 🔴 Path yếu nhất: Path 3 + Path 4

**Vấn đề chính:**
- Khi AI sai, recovery flow mất quá nhiều bước
- Không có feedback loop rõ:
  - User sửa nhưng không biết AI có học không
- Không có exit/fallback cho user mất niềm tin

---

## 📉 Gap Marketing vs Thực tế

**Marketing:**
> "Moni giúp quản lý chi tiêu thông minh"

**Thực tế:**
- Auto-tag chỉ đúng ~70% cho giao dịch phổ biến
- Các trường hợp edge case:
  - Chuyển tiền  
  - Mua online  
→ thường bị tag sai hoặc không tag

**Gap lớn nhất:**
- Marketing không nói về khi AI sai  
→ User kỳ vọng 100% chính xác

---

## ✏️ Sketch

**As-is:** 
Giao dịch → Auto-tag → User thấy kết quả
↓
Nếu sai → sửa thủ công


**To-be:**
Giao dịch → Auto-tag
↓
Nếu confidence thấp
↓
Hiện: "Bạn muốn phân loại?"
↓
User chọn
↓
AI ghi nhận correction
↓
Hiện: "Đã học, lần sau sẽ chính xác hơn"