# BOM lắp ráp

File: [JIG_TEST_FLEX_V1.0_BOM_2026-10-09.xlsx](JIG_TEST_FLEX_V1.0_BOM_2026-10-09.xlsx).

File Excel được giữ nguyên dữ liệu và định dạng, gồm 34 nhóm với 113 mã linh kiện riêng biệt. Khi đóng gói ngày 15/10/2026, từng trường trong BOM đã được đối chiếu với parameter của linh kiện trong schematic đi kèm. Chỉ có ba trường thông tin mua hàng của H101 khác biệt như ghi chú bên dưới. Phép đối chiếu này chưa xác nhận hình học footprint, vị trí linh kiện hoặc khả năng chế tạo.

| Cột | Ý nghĩa |
| --- | --- |
| Des | Designator của các linh kiện được gom nhóm theo các trường BOM đã chọn |
| Type | Loại linh kiện |
| Value1 | Giá trị hoặc chức năng chính |
| Value2 | Thông số hoặc định mức bổ sung |
| Case | Package |
| Manu | Nhà sản xuất |
| MPN | Mã linh kiện của nhà sản xuất |
| JPN | Mã mua hàng JLCPCB/LCSC |

Số lượng của mỗi dòng là số designator được phân tách bằng dấu phẩy trong cột `Des`.

## Phạm vi

Schematic và PCB đều có 135 linh kiện. BOM này là danh sách linh kiện đã chọn để lắp ráp, mua hàng; chưa liệt kê toàn bộ đối tượng trên PCB. Phạm vi được giữ theo BOM đã chọn trước đó.

- Linh kiện DNP không đưa vào BOM: C102, R128, R129.
- Connector DUT J102 không nằm trong BOM lắp ráp đã chọn; cần chuẩn bị riêng hoặc bổ sung khi lập đơn lắp ráp đầy đủ.
- Testpoint không đưa vào BOM: 101, 102, TP101–TP114, VIN101, V_BUS101.

Tổng cộng 22 mã không nằm trong BOM, tương ứng chênh lệch giữa 135 linh kiện trong thiết kế và 113 linh kiện trong BOM. Testpoint có thể là điểm đồng trên PCB, không phải linh kiện mua riêng. Việc không đưa vào BOM không xóa các đối tượng này khỏi thiết kế.

## Thông tin mua hàng H101

H101 trong schematic đi kèm chưa có các parameter `Manu`, `MPN` và `JLCPCB Part #`. BOM Excel giữ các giá trị đã chọn trước đó:

| Trường | Giá trị |
| --- | --- |
| Nhà sản xuất | LXWCONN |
| MPN | 254PZ-1x7P-V |
| JPN | C53055676 |

Schematic vẫn giữ mã connector nội bộ MH003369. Cần xác nhận linh kiện mua thực tế và độ phù hợp cơ khí trước khi đặt hàng. Thiết kế trong gói chưa được sửa để đồng bộ khác biệt thông tin này.

## Cập nhật BOM

Cập nhật ActiveBOM sau khi thay đổi schematic. Không mặc định dữ liệu cache của `.BomDoc` đi kèm khớp với BOM Excel này. Kiểm tra lại designator, trạng thái DNP, package, phạm vi connector và mã mua hàng trước khi xuất dữ liệu chế tạo.
