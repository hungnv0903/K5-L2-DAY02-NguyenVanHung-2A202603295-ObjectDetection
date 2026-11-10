# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- || `car` | Xe con cá nhân/dịch vụ từ 4–9 chỗ, sedan, hatchback, SUV, CUV, taxi và xe bán tải (pickup). | Phân biệt với `truck` ở chỗ xe bán tải chở người kiêm chở hàng nhẹ vẫn thuộc `car`. Không gán cho xe buýt, xe tải nặng, mô tô. |
| `truck` | Xe tải chở hàng chuyên dụng, xe tải thùng kín, xe ben, xe đầu kéo container, xe bồn. | Có kết cấu cabin và thùng chở hàng/chassis tách biệt hoặc tải trọng lớn; không gán cho xe bán tải (`car`) hoặc xe van liền khối (`van`). |
| `bus` | Xe buýt công cộng nội đô, xe khách liên tỉnh cỡ lớn, xe du lịch 29–45 chỗ. | Thân xe dài, nhiều hàng ghế lớn, thiết kế chở hành khách số lượng lớn; phân biệt với `van` chở người loại nhỏ (9–16 chỗ). |
| `van` | Xe van chở hàng liền khối, minivan chở người/gia đình, xe thương mại 9–16 chỗ (như Ford Transit, Solati). | Thân xe dạng khối hộp liền mạch (box-shaped), không có thùng hàng rời như `truck` và không lớn như `bus`. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: Bounding box ôm sát phần thân xe nhìn thấy được (tight fit), không lấy thêm mặt đường, bóng xe, bầu trời hay lấn sang xe bên cạnh.
- Vật thể bị che khuất: Vẫn gán box ôm trọn phần nhìn thấy được của xe nếu nhận diện được lớp; nếu bị che quá nhiều chỉ còn mảnh nhỏ không đủ nhận diện thì bỏ qua.
- Vật thể bị cắt bởi rìa ảnh: Box chạm đúng biên mép ảnh, không tự vẽ đoán phần xe nằm ngoài khung hình; tọa độ chuẩn hóa không được vượt quá khoảng [0, 1].
- Vật thể rất nhỏ/mơ hồ: Chỉ gán khi đủ đặc điểm nhận diện thuộc 4 lớp của schema; bỏ qua các vệt sáng/xe ở đường chân trời quá xa, không gán hình xe trên biển quảng cáo hoặc hình phản chiếu qua gương/kính.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_001` | Xe bán tải (pickup truck) di chuyển trên làn đường. | Gán `car` (class 0) | Schema quy định rõ xe bán tải thuộc lớp `car`, không xếp vào `truck`. |
| `drive_002` | Xe 16 chỗ dạng Ford Transit/Solati chở khách/học sinh. | Gán `van` (class 3) | Xe van/minivan chở người thuộc lớp `van`, không gán vào `bus` (dành cho xe buýt/xe khách lớn). |
| `drive_015` | Xe tải nhỏ chở hàng có cabin đơn và thùng hàng phía sau. | Gán `truck` (class 1) | Xe có kết cấu thùng chở hàng chuyên dụng phía sau thuộc `truck`, không phải xe van thân liền. |
| `drive_006` | Xe con bị cắt một phần thân bởi mép ảnh bên phải. | Gán `car` (class 0) bám sát mép viền | Nhận diện rõ phần thân là xe con; box vẽ chạm sát biên ảnh, không ước lượng phần bị khuất ngoài ảnh. |
 |
| `drive_...` | | | |
