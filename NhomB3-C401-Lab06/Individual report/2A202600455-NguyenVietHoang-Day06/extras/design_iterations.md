# Design Iterations - Data and State Machine

## Iteration 1 - Baseline flow
- Trang thai booking ban dau: `PENDING`, `CONFIRMED`.
- Van de: khong mo ta ro expiry, de tao ghost booking.

## Iteration 2 - Add TTL and EXPIRED
- Bo sung trang thai `EXPIRED`.
- Them countdown 5 phut va worker expire tu dong.
- Loi ich: slot duoc tra lai, giam dead inventory.

## Iteration 3 - Standardize transitions
- Chuan hoa transitions:
  - `AVAILABLE -> PENDING`
  - `PENDING -> CONFIRMED`
  - `PENDING -> EXPIRED`
  - `CONFIRMED -> PENDING` (reschedule theo quy trinh)
- Chan chuyen trang thai khong hop le.

## Iteration 4 - Concurrency hardening
- Them lock cho hold/confirm/reschedule/expire.
- Muc tieu: tranh double confirm va stale update.
- Gioi han con lai: race condition sat moc TTL can test day hon.

## Iteration 5 - Contract alignment
- Dong bo field booking cho frontend/agent:
  - `booking_id`
  - `status`
  - `ttl_seconds`
  - `service_center`
  - `slot_time`
- Loai bo hardcode countdown phia client.
