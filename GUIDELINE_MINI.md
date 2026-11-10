# Mini guideline — bài làm cá nhân

> Học viên: Nguyễn Duy Khánh — 2A202603055. Guideline được chốt trước khi gán nhãn và cập nhật sau vòng self-QC (đối chiếu lại schema trong README).

## Định nghĩa lớp

| Lớp | Định nghĩa của bạn | Ví dụ/điểm phân biệt |
| --- | --- | --- |
| `car` | Xe chở người cỡ nhỏ: sedan, hatchback, SUV, taxi, xe bán tải. Mui thấp, có nắp ca-pô và phần đuôi rõ ràng. | Taxi vàng Bắc Kinh, sedan, SUV đen. Xe to hoặc tối màu nhưng mui thấp, đầu dốc vẫn là `car`. Không gán xe buýt, xe tải, mô tô. |
| `truck` | Xe tải chở hàng và xe chuyên dụng trên khung xe tải: xe thùng, xe ben, xe bồn, xe cứu hộ, xe đầu kéo. Cabin tách biệt với thùng hoặc cụm thiết bị phía sau. | Xe ben chở đất (drive_008, drive_040), xe thùng bưu chính (drive_009), xe bồn (drive_037), xe tải thùng trắng (drive_033, drive_047). Bán tải cabin nhỏ là `car`. |
| `bus` | Xe buýt và minibus chở khách: thân dài, nhiều hàng cửa sổ, có cửa lên xuống khách. | Buýt thường, buýt nối toa, buýt hai tầng. Van chở người cỡ nhỏ là `van`, không phải `bus`. |
| `van` | Xe van, minivan chở hàng hoặc chở người: thân hình hộp, mui cao và phẳng, cửa sổ hông dài thẳng hàng, đầu xe ngắn. | Minivan màu be/bạc kiểu Wuling/Jinbei (drive_027, drive_024), van đỏ (drive_014, drive_038). Ngắn hơn rõ rệt so với `bus`; mui cao, dạng hộp so với `car`. |

## Quy tắc chung

- Box bám sát vật thể nghĩa là: 4 cạnh box chạm mép ngoài cùng của phần thân xe nhìn thấy được (gồm gương, cản), không lấy bóng đổ, mặt đường hay xe bên cạnh. Zoom vào xe nhỏ trước khi vẽ và chỉnh lại các cạnh sau khi vẽ.
- Vật thể bị che khuất: vẫn vẽ box quanh phần nhìn thấy được nếu nhận ra được lớp; mỗi xe một box riêng, các box được phép chồng lên nhau; không gộp hai xe thành một box và không đoán phần bị che.
- Vật thể bị cắt bởi rìa ảnh: cạnh box chạm đúng mép ảnh, không kéo ra ngoài, không đoán phần nằm ngoài khung.
- Vật thể rất nhỏ/mơ hồ: vẫn gán nếu sau khi zoom còn nhận ra được là xe thuộc schema và xác định được lớp; nếu chỉ là mảnh nhỏ không xác định chắc lớp thì không gán và ghi lại để hỏi giảng viên.
- Xe đang đỗ vẫn gán theo loại xe. Không gán người đi bộ, xe đạp, xe máy, xe ba bánh, biển báo, xe trên bảng quảng cáo hoặc hình phản chiếu.

## Ghi lại quyết định cho ít nhất ba ca mơ hồ

| Ảnh | Tình huống | Quyết định cuối | Lý do theo schema |
| --- | --- | --- | --- |
| `drive_014`, `drive_019`, `drive_034`, `drive_038`, `drive_046` | Xe cứu hộ công an màu trắng, có cần cẩu phía sau. Lần gán đầu không nhất quán (lúc `van`, lúc `car`). | `truck` ở cả 5 ảnh | Đây là xe chuyên dụng trên khung xe tải, cabin tách khỏi cụm cẩu phía sau. `car` loại trừ xe tải; không phải dạng thân hộp chở người/hàng của `van`. |
| `drive_027`, `drive_024`, `drive_021`, `drive_031`, `drive_043` | Minivan thân hình hộp (màu be/bạc/trắng) ban đầu bị gán `car`. | `van` | Schema: `van` gồm "xe van, minivan chở hàng hoặc chở người". Xe có mui cao phẳng, thân hộp, cửa sổ hông dài là `van`; xe mui thấp có ca-pô và đuôi rõ (kể cả xe tối màu đứng cạnh, drive_024) giữ là `car`. |
| `drive_024` | Xe ba bánh chở hàng lẫn trong dòng xe, lần đầu bị gán `van`. | Không gán (xóa box) | Không thuộc bốn lớp của schema (gần với xe máy/xe ba bánh); GUIDE yêu cầu không gán vật thể ngoài scope. |
| `drive_008`, `drive_009`, `drive_033`, `drive_037`, `drive_047` | Xe ben, xe tải thùng, xe bồn ban đầu bị gán `van` hoặc `car`. | `truck` | Schema: `truck` là "xe tải chở hàng". Thùng hàng/bồn tách khỏi cabin; giữ nhất quán với xe ben ở drive_040. |

## Ghi chú cho QC với gold set

- Bộ nhãn gán cả xe nhỏ ở xa nếu còn nhận ra lớp (trung bình ~24 box/ảnh). Nếu gold bỏ qua xe quá nhỏ ở xa, phần này có thể làm giảm precision; sẽ phân tích ở bước rework, không tự xóa trước khi có gold.
- Quan sát trên nhãn eval (`dataset/labels/eval/`): các box ID 1 là xe van, ID 2 là xe tải, ID 3 là xe buýt, tức lệch so với `data.yaml` (1=truck, 2=bus, 3=van). Bộ nhãn này vẫn theo schema chính thức trong README; cần giảng viên xác nhận mapping lớp của gold.
