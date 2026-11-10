# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe con dùng cho cá nhân hoặc gia đình, thường có 4 chỗ đến 7 chỗ, thân xe thấp và ngắn. Bao gồm sedan, SUV, hatchback, taxi, xe bán tải (pickup) có cabin nhỏ. | Sedan (4 cửa, thân dài thấp), SUV (thân cao hơn sedan nhưng không phải van), taxi (thường có màu vàng/cam đặc trưng), pickup. Phân biệt với `van`: car có chiều cao thân xe thấp hơn, không có khoang hàng hộp vuông phía sau. |
| `truck` | Xe tải chở hàng cỡ lớn, có thùng hàng rõ ràng (thùng lửng, thùng kín, container) tách biệt với cabin. Bao gồm xe đầu kéo, xe ben, xe tải nặng. | Xe ben chở đất/cát (thùng lật), xe tải thùng kín lớn, xe đầu kéo container. Phân biệt với `van`: truck có cabin tách biệt hẳn với thùng hàng và kích thước lớn hơn nhiều so với van. |
| `bus` | Xe buýt hoặc minibus chở khách, thân dài, có nhiều cửa sổ hành khách xếp hàng dọc thân xe, thường có 2 cửa lên xuống trở lên. | Xe buýt thành phố (thân dài, nhiều cửa kính), xe buýt liên tỉnh. Phân biệt với `van`: bus dài hơn rõ rệt, có hàng cửa sổ kéo dài, thường trên 8m. Phân biệt với `truck`: bus có nhiều cửa sổ hành khách, truck có thùng hàng kín/mở. |
| `van` | Xe van hoặc minivan, thân hộp vuông vức, khoang phía sau liền cabin (không tách cabin như truck), dùng chở hàng hoặc chở người. | Xe van chở hàng (thân hộp trắng, ít cửa sổ), minivan chở người (có cửa sổ hàng ghế sau). Phân biệt với `car`: van có thân cao và vuông hơn, khoang sau rộng hình hộp. Phân biệt với `bus`: van ngắn hơn, ít cửa sổ hơn, thường dưới 6m. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: cạnh box chạm đúng rìa ngoài cùng của phần thân xe nhìn thấy được (bao gồm gương chiếu hậu nếu thấy rõ), không lấy thêm mặt đường, vỉa hè, bầu trời hay phần thân xe khác bên cạnh. Giữ khoảng trống giữa box và xe ở mức tối thiểu (1–2 pixel).
- Vật thể bị che khuất: vẫn gán box bao quanh phần nhìn thấy được nếu phần lộ ra đủ để nhận diện chắc chắn lớp xe (ví dụ thấy rõ đuôi xe sedan → gán `car`). Nếu bị che quá nhiều (chỉ còn mảnh nhỏ, không thể xác định là car/truck/bus/van) thì không gán.
- Vật thể bị cắt bởi rìa ảnh: gán box với cạnh box chạm đúng rìa ảnh, không đoán phần nằm ngoài khung hình. Chỉ gán khi phần trong ảnh đủ để nhận diện lớp.
- Vật thể rất nhỏ/mơ hồ: nếu vật thể quá nhỏ (dưới ~15×15 pixel) hoặc quá mờ do chuyển động/ánh sáng yếu khiến không thể phân biệt chắc chắn giữa các lớp, không gán. Nếu nhỏ nhưng vẫn nhận diện được lớp (ví dụ thấy rõ hình dáng sedan ở xa) thì vẫn gán.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_008` | Xe van trắng nhỏ (bên trái ảnh) có thân hộp, kích thước gần giống car — khó phân biệt car hay van | Gán `van` (class 3) | Thân xe cao, vuông, khoang sau liền cabin dạng hộp kín — đặc trưng van chở hàng. Car có thân thấp hơn và không có khoang hộp. |
| `drive_005` | Ngã tư đông đúc, có xe máy/mô tô xen giữa các ô tô — có nên gán xe máy không | Không gán xe máy/mô tô | Schema chỉ có 4 lớp (car, truck, bus, van). Xe máy/mô tô không thuộc scope, bỏ qua dù thấy rõ trong ảnh. |
| `drive_020` | Nhiều xe bị che khuất lẫn nhau khi kẹt xe (drive_020 có xe chồng chéo), chỉ thấy nóc hoặc một phần đuôi xe | Gán box riêng cho từng xe nếu phần lộ ra đủ nhận diện lớp; bỏ qua xe chỉ thấy mảnh rất nhỏ (ví dụ chỉ thấy góc gương) | Mỗi xe vật lý cần một box riêng. Box ôm phần nhìn thấy, không gộp hai xe. Nếu không đủ để xác định class → không gán, tránh đoán. |
| `drive_010` | Ảnh chụp ban đêm, xe ở xa bị mờ và chỉ thấy đèn pha — có gán không | Không gán xe chỉ thấy ánh đèn mà không nhìn rõ hình dáng thân xe | Cần nhận diện được lớp xe từ hình dáng. Chỉ thấy đốm sáng đèn pha không đủ cơ sở phân lớp car/truck/bus/van. |
