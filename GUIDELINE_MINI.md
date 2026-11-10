# Mini guideline — bài làm cá nhân

> Điền nội dung này trước khi bắt đầu gán nhãn. Giữ lại các quyết định sau khi self-QC.

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` (`0`) | Xe con cá nhân hoặc dịch vụ chở ít người (4–7 chỗ), bao gồm sedan, hatchback, SUV, crossover, taxi và xe bán tải cabin nhỏ. | Có phần mũi capo và cốp/đuôi xe thấp; xe bán tải (pickup) có thùng hở nhỏ phía sau vẫn tính vào `car`, không tính vào `truck`. |
| `truck` (`1`) | Xe tải chở hàng chuyên dụng (thùng kín, thùng bạt, thùng lửng tải trọng lớn) và xe đầu kéo/container. | Có cabin đầu vuông tách biệt với thùng hàng lớn phía sau, gầm cao, kích thước dài/cao hơn hẳn xe con và xe van. |
| `bus` (`2`) | Xe buýt công cộng, xe khách cỡ lớn hoặc minibus chở khách chuyên dụng (thường từ 16–45 chỗ trở lên). | Thân xe hình hộp dài, nhiều ô cửa sổ kính liên tiếp dọc thân xe, đầu phẳng, chiều dài và chiều cao lớn hơn `van`. |
| `van` (`3`) | Xe van/minivan dạng thân liền khối (MPV cỡ lớn/xe chở hàng hoặc chở người cỡ trung dưới 16 chỗ, đầu ngắn). | Dáng hình hộp gọn hơn `bus` (thân ngắn hơn, ít ô cửa sổ hơn), nóc cao hơn `car`/SUV và thân liền khối chứ không tách cabin-thùng như `truck`. |

## Quy tắc chung

- **Box bám sát vật thể nghĩa là:** Các cạnh của bounding box ôm sát phần cực trị nhìn thấy được của thân xe (bao gồm cản trước/sau, bánh xe và nóc xe), không lấy thừa mặt đường, bóng đổ dưới đất, bầu trời hay phần thân xe kế bên.
- **Vật thể bị che khuất:** Nếu xe bị che một phần bởi xe phía trước, cột đèn hoặc dải phân cách nhưng phần lộ ra vẫn đủ đặc trưng để xác định chắc chắn loại xe (`car`, `truck`, `bus`, `van`), vẫn vẽ 1 box bao quanh **phần thực tế nhìn thấy được** (không suy đoán phần bị khuất).
- **Vật thể bị cắt bởi rìa ảnh:** Khi xe đi vào/ra ở mép khung hình, kéo cạnh box chạm sát mép ảnh tại phần thân xe còn nằm trong khung hình (nếu phần còn lại đủ nhận diện được lớp), tuyệt đối không vẽ vượt ra ngoài biên `[0, 1]`.
- **Vật thể rất nhỏ/mơ hồ:** Các xe ở quá xa phía chân trời/đầu cầu vượt chỉ còn vài pixel mờ nhạt, không thể phân biệt chắc chắn thuộc lớp nào trong 4 lớp thì **không tự đoán/không gán nhãn**, ghi chú lại để đối chiếu theo guideline.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_008` | Xe dáng hình hộp lớn màu sáng ở làn bên phải (`x_center ≈ 0.76, y_center ≈ 0.35`) và xe chở khách dài phía xa giữa đường (`x_center ≈ 0.45, y_center ≈ 0.23`), dễ nhầm lẫn giữa `bus` và `van`/`truck`. | Gán xe ở `x ≈ 0.76` là `van` (`3`); gán xe dài nhiều cửa sổ ở `x ≈ 0.45` là `bus` (`2`). | Xe ở `x ≈ 0.76` là minivan/van thân ngắn liền khối chở người/hàng cỡ trung (`van`), trong khi xe ở `x ≈ 0.45` có thân dài và dãy kính hành khách đặc trưng của `bus`. |
| `drive_024` | Nhiều phương tiện cỡ lớn cùng xuất hiện ở làn giữa và làn phải (`x ≈ 0.78, y ≈ 0.49` và `x ≈ 0.83, y ≈ 0.41`) xen lẫn xe hình hộp ở làn trái (`x ≈ 0.15, y ≈ 0.28`), dễ nhầm giữa `truck` thùng kín và `van`/`bus`. | Gán hai xe tải thùng ở làn phải là `truck` (`1`); gán xe hộp ở làn trái (`x ≈ 0.15`) và xe phía trên (`x ≈ 0.43, y ≈ 0.13`) là `van` (`3`). | Hai xe ở làn phải có kết cấu cabin tách rời thùng chở hàng phía sau (`truck`), còn xe ở làn trái là thân liền khối dạng minivan (`van`). |
| `drive_028` | Chỉ có 3 xe ở tiền cảnh/trung cảnh rõ nét, nhưng phía rất xa gần chân trời có một cụm nhiều chấm xe nhỏ nối đuôi nhau bị mờ và che khuất lẫn nhau. | Chỉ gán 3 xe rõ đặc trưng lớp; **không gán** cụm xe quá nhỏ/mờ ở xa phía chân trời. | Theo quy tắc vật thể rất nhỏ/mơ hồ trong `GUIDE.md`: khi mảnh xe quá nhỏ ở xa không đủ đặc trưng xác định chắc chắn lớp thì không tự đoán để tránh sinh False Positive (FP). |
| `drive_042` | Các xe tải chở hàng di chuyển ở trung cảnh có phần thùng phía sau nhìn từ góc camera trên cao hơi giống xe khách/xe van. | Gán các xe có thùng hàng phía sau là `truck` (`1`), xe con bên cạnh là `car` (`0`). | Dựa vào tỷ lệ chiều rộng/cao của thùng hàng vuông vức và phần đầu kéo/cabin đặc trưng của xe tải chở hàng (`truck`). |
