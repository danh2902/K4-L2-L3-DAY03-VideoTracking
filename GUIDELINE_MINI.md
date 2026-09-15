# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Ngô Lê Đức Anh`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): `không có`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới ... frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | `Vẫn có khả năng xác định đó là cùng một xe dựa vào vị trí, hình dạng và đặc điểm trước và sau khi bị che` |
| Xe bị che lâu hơn ngưỡng trên | `Tạo track mới` | `Khoảng thời gian mất dấu quá dài, khó đảm bảo xe xuất hiện lại là cùng đối tượng` |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `Không có đủ thông tin liên tục để đảm bảo danh tính giữa hai lần xuất hiện` |
| Hai xe cắt nhau / chồng lên nhau | `Giữ ID theo từng xe, dựa vào vị trí, hướng di chuyển và đặc điểm nhận dạng, không đổi ID chỉ vì bị chồng lấp` | `Tránh ID switch khi hai xe đi qua nhau` |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `...` |
| Xe đang đỗ, không di chuyển | `vẫn giữ cùng ID và bbox quanh phần xe nhìn thấy, cập nhật keyframe khi hình dạng hoặc vị trí bbox thay đổi` |
| Keyframe đặt dày ở đâu | `đặt dày tại các đoạn xe vào hoặc ra khung hình, bị che khuất, giao nhau với xe khác hoặc bbox thay đổi nhanh` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip 01 / frame 100 / ID 3`
- Tình huống: `ô tô bị cột điện che khuất một phần ở giữa`
- Quyết định: `chỉ box phần nhìn thấy rõ`
- Lý do: `không dự đoán phần không thấy`

### Ca 2
- Clip / frame / ID: `clip 01 / frame 61 / ID 8`
- Tình huống: `xe bus xuất hiện`
- Quyết định: `box khi đủ bằng chứng xác định xe bus`
- Lý do: `khi mới xuất hiện thì chỉ lộ một phần đầu nên không đủ bằng chứng`

### Ca 3
- Clip / frame / ID: `clip 01 / frame 107 / ID 8`
- Tình huống: `vẽ box cho xe bus nhưng có một phần gương chiếu hậu lòi ra khá xa xe và model không vẽ box cho phần gương lòi ra`
- Quyết định: `vẽ box cho phần gương`
- Lý do: `trong hướng dẫn ghi phải vẽ box hết toàn bộ xe`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Quy tắc bbox khi xe bị che cần ghi rõ hơn: chỉ bao phần xe nhìn thấy, không suy đoán phần bị che; cần kiểm tra kỹ bbox tại các frame có occlusion/crossing.`
- `Cần bổ sung quy tắc kiểm tra track trước khi xe xuất hiện: không để bbox/track tồn tại ở các frame trước khi xe có thể xác định chắc chắn.`
