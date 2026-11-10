# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Tất cả các loại xe con cá nhân, xe gia đình chở người từ 4 đến 7 chỗ (sedan, hatchback, SUV, crossover, coupe), taxi và xe bán tải (pickup). | Kích thước nhỏ hoặc vừa. Xe bán tải cabin nhỏ có thùng hở phía sau vẫn phân loại là `car` theo schema, không gán là `truck`. |
| `truck` | Các loại xe tải chở hàng hóa thương mại vừa và lớn, xe tải thùng kín, xe ben, xe bồn, xe đầu kéo container. | Kết cấu cabin tách biệt rõ ràng với phần thùng chở hàng lớn phía sau; thường có bánh kép hoặc nhiều trục bánh. Khác `car` ở thùng hàng chuyên dụng, khác `van` ở chỗ thùng hàng không liền khối cabin. |
| `bus` | Xe buýt công cộng, xe khách liên tỉnh cỡ lớn, xe đưa đón đoàn thể (> 16-29 chỗ). | Kích thước dài và cao vượt trội, nhiều cửa sổ dọc hai bên hông xe, đầu xe phẳng hoặc dốc đứng. Phân biệt với `van` ở kích thước lớn hơn rõ rệt và sức chứa hành khách lớn. |
| `van` | Xe van chở hàng hoặc chở người dạng hình hộp liền khối (minivan, xe 9-16 chỗ như Ford Transit, Hyundai Solati, minivan gia đình). | Thân xe liền khối từ khoang lái đến khoang sau, trần xe cao hơn xe con (`car`), thường có cửa trượt bên hông. Khác `bus` vì kích thước nhỏ gọn hơn (< 16 chỗ), khác `car` ở kiểu dáng khối hộp. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: Bounding box phải ôm khít các mép ngoài cùng của thân xe (cản trước, cản sau, nóc xe, điểm lốp xe tiếp đất). Không vẽ box lỏng lấn ra mặt đường, bầu trời hay xe bên cạnh; mỗi xe là một box riêng biệt, tuyệt đối không gộp chung.
- Vật thể bị che khuất: Nếu xe bị che khuất một phần (bởi xe khác, cột đèn, dải phân cách) nhưng vẫn đủ đặc điểm nhận diện loại xe, chỉ vẽ box quanh **phần thân xe thực tế nhìn thấy được (visible region)**. Không vẽ ước lượng phần bị che. Nếu bị che quá nhiều (> 80%) không thể nhận diện chắc chắn thì bỏ qua.
- Vật thể bị cắt bởi rìa ảnh: Xe đang đi vào hoặc ra khỏi khung hình bị cắt một phần thân xe thì cạnh box phải chạm sát đúng mép ảnh ($x=0, x=1, y=0, y=1$). Không cố vẽ tràn ra ngoài hoặc suy đoán phần ngoài khung hình.
- Vật thể rất nhỏ/mơ hồ: Các phương tiện ở quá xa tại đường chân trời (kích thước quá nhỏ, mờ) hoặc nằm ở làn đường phụ phía xa ngoài luồng giao thông giám sát chính của camera thì không gán. Không gán người đi bộ, xe máy, xe đạp, biển báo hoặc hình ảnh phản chiếu trên kính.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_003` | Phương tiện ở làn giữa $(x \approx 0.50, y \approx 0.26)$, nhìn từ xa có kiểu dáng cabin và thùng hàng nhỏ phía sau, dễ nhầm giữa `car` (SUV/pickup) và `truck`. | Gán nhãn `truck` (class 1) | Thân xe có cabin tách rời với thùng hàng chuyên dụng phía sau, thuộc loại xe tải chở hàng cỡ nhỏ theo schema chứ không phải xe con cá nhân. |
| `drive_002` | Hai xe chạy nối đuôi nhau ở làn giữa, xe phía sau bị đuôi xe phía trước che khuất một phần góc đầu xe (occlusion). | Vẽ 2 bounding box riêng biệt; box của xe sau chỉ ôm sát phần thân xe nhìn thấy được. | Tuân thủ quy tắc mỗi xe một box riêng và chỉ bao quanh phần diện tích nhìn thấy thực tế, không vẽ gộp và không đoán phần bị che. |
| `drive_001` | Xe con ở mép phải dưới khung hình $(x \approx 0.93, y \approx 0.33)$ bị rìa ảnh cắt ngang nửa thân xe bên phải (truncation). | Gán nhãn `car` (class 0) với cạnh phải của box chạm sát mép biên ảnh ($x_{\text{max}} = 1.0$). | Xe bị cắt khung nhưng vẫn nhận diện rõ là xe con; box chạm đúng rìa ảnh và không ước lượng phần ngoài ảnh. |
| `drive_001` | Nhiều xe nhỏ nằm ở làn đường phía xa trên cao ($y < 0.22$) có kích thước rất nhỏ và mờ. | Quyết định không gán box cho các xe ở làn xa này. | Các xe này nằm ngoài phạm vi luồng giao thông chính của góc quay camera CCTV, kích thước quá nhỏ và mờ, không đảm bảo nhận diện chính xác theo gold set. |
