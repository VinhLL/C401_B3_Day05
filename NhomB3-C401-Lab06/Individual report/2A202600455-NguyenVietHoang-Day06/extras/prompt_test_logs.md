# Prompt Test Logs (Nguyen Viet Hoang)

## Scope
- Test luong dat lich, xac nhan, doi lich, TTL expiry, out-of-scope.
- Muc tieu: kiem tra tinh on dinh cua state machine va su nhat quan output cho agent/frontend.

## Test Log 01 - Booking happy path
- **Input:** "Dat lich bao duong cho VF8 luc 14h tai trung tam gan nhat"
- **Expected:** Tao booking `PENDING`, tra ve `booking_id`, `ttl_seconds`.
- **Observed:** Dat cho thanh cong, countdown hien thi dung.
- **Result:** Pass.

## Test Log 02 - Confirm bang cau ngan
- **Input:** "ok", "xac nhan"
- **Expected:** Booking `PENDING` chuyen `CONFIRMED`.
- **Observed:** Confirm thanh cong voi booking gan nhat trong session.
- **Result:** Pass.

## Test Log 03 - Slot da het
- **Input:** Dat lich vao slot vua bi user khac giu.
- **Expected:** Bao `slot_unavailable`, goi y slot khac.
- **Observed:** System tu choi slot cu va de xuat slot moi.
- **Result:** Pass.

## Test Log 04 - TTL expiry
- **Input:** Tao booking `PENDING` roi cho qua han.
- **Expected:** Chuyen `EXPIRED`, slot tro ve `AVAILABLE`.
- **Observed:** Worker xu ly dung, frontend khong cho confirm booking het han.
- **Result:** Pass.

## Test Log 05 - Confirm sat moc het han
- **Input:** Gui confirm khi countdown con 1-2s.
- **Expected:** He thong xu ly nhat quan (confirm thanh cong hoac tra ve expired ro rang).
- **Observed:** Co luc tra ket qua khong on dinh do race timing.
- **Result:** Need improvement (uu tien test concurrent + lock/atomic check).

## Test Log 06 - Out-of-scope
- **Input:** "Toi muon mua bao hiem than vo"
- **Expected:** Agent tu choi lich su, huong dan lien he kenh phu hop.
- **Observed:** Tu choi dung pham vi, khong hallucinate.
- **Result:** Pass.
