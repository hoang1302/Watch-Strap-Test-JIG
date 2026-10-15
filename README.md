# JIG kiểm tra quai đồng hồ chống tháo, chống cắt

JIG dùng để kiểm tra độ dẫn điện, điện trở hai vòng mạch trên quai và hoạt động của Hall sensor thông qua giao tiếp DUT gồm sáu tiếp điểm.

**Trạng thái: phần cứng đang ở giai đoạn prototype, tiếp tục hoàn thiện.** Bộ này gồm thiết kế Altium, thư viện symbol và footprint, BOM lắp ráp, đặc tả phần cứng và thuật toán test dự kiến. Chưa có mã nguồn firmware hoặc kết quả thử nghiệm trên phần cứng thực tế trong bộ tài liệu này.

| Hạng mục | Thông tin |
| --- | --- |
| Tên thiết kế phần cứng | JIG_TEST_FLEX_V1.0 |
| Ngày đóng gói | 15/10/2026 |
| Vi điều khiển | STM32F030C8T6 |
| Nguồn đầu vào | 5 V qua USB Type-C |
| Khối nguồn chính | TLV62569DBVR tạo 3V3; AMS1117-1.8 tạo nguồn cho vòng mạch DUT |
| Khối đo analog | INA4180A1IPWR, bốn kênh |
| Comparator | LMV393IDR, hai kênh |
| Giao tiếp DUT | J102, sáu tiếp điểm; cần bổ sung bản vẽ cơ khí cụm pogo |
| Giao tiếp debug | H101, SWD và UART mức 3,3 V |
| BOM lắp ráp | 34 nhóm, 113 linh kiện |

## Mục đích

JIG được thiết kế để kiểm tra hai vòng dẫn điện trên quai và xác nhận tín hiệu Hall khi đặt, sau đó bỏ nam châm. Việc đo liên tục trong lúc uốn quai nhằm phát hiện kết nối chập chờn. Giới hạn phát hiện lỗi, ngưỡng chấp nhận, dữ liệu hiệu chuẩn và quy trình vận hành cần được xác nhận trên phần cứng thực tế.

## Cấu trúc thư mục

```text
.
├── README.md
├── .gitignore
├── .gitattributes
├── hardware/
│   ├── altium/
│   │   ├── JIG_TEST_FLEX_V1.0.PrjPcb
│   │   ├── JIG_TEST_FLEX_V1.0.SchDoc
│   │   ├── JIG_TEST_FLEX_V1.0.PcbDoc
│   │   ├── JIG_TEST_FLEX_V1.0.BomDoc
│   │   ├── jig.SchLib
│   │   ├── jig.PcbLib
│   │   └── Constraints.xml
│   └── bom/
│       ├── JIG_TEST_FLEX_V1.0_BOM_2026-10-09.xlsx
│       └── README.md
└── docs/
    ├── JIG_Hardware_Interface_Spec.md
    ├── JIG_Test_Algorithm_Spec.md
    └── package-manifest.json
```

## Cách mở thiết kế

1. Tải về hoặc clone toàn bộ repo.
2. Mở [JIG_TEST_FLEX_V1.0.PrjPcb](hardware/altium/JIG_TEST_FLEX_V1.0.PrjPcb) bằng Altium Designer.
3. Giữ nguyên các file trong `hardware/altium/`. Thư viện symbol và footprint đã được thêm vào project.
4. Chạy compile và kiểm tra project trên phiên bản Altium đang sử dụng. Xử lý các thông báo về thư viện hoặc model trước khi cập nhật linh kiện hay xuất dữ liệu.

Phiên bản Altium dùng để tạo thiết kế chưa được ghi nhận. Khi đóng gói, bộ này đã được kiểm tra tính toàn vẹn file; chưa xác nhận việc mở và compile trong Altium.

## Tài liệu

- [Đặc tả giao tiếp phần cứng](docs/JIG_Hardware_Interface_Spec.md): các rail nguồn, connector DUT, chân MCU, kênh đo và giới hạn triển khai.
- [Đặc tả thuật toán test](docs/JIG_Test_Algorithm_Spec.md): các trạng thái dự kiến, xử lý phép đo, mã kết quả, hiệu chuẩn và trường hợp kiểm thử.
- [BOM lắp ráp và phạm vi linh kiện](hardware/bom/README.md): ý nghĩa các cột, linh kiện không nằm trong BOM và ghi chú mã mua H101.
- [Danh sách file và mã kiểm tra](docs/package-manifest.json): mã SHA-256 và các thay đổi khi đóng gói.

## Ghi chú về bản đóng gói

Nội dung schematic, PCB, ActiveBOM, cấu hình ràng buộc và thư viện nguồn được sao chép nguyên trạng, không sửa thiết kế điện. Thư viện được đặt tên `jig.SchLib` và `jig.PcbLib` để khớp tên thư viện trước đây trong project và symbol. Bản sao project đã tham chiếu tới các tên file này.

File `.PrjPcb` trong gói đã được bỏ tham chiếu tới tài liệu CAM cũ, dữ liệu xuất tự động và thư viện tích hợp cũ. Đường dẫn tới schematic/PCB trong cấu hình xuất đã được chuyển sang đường dẫn tương đối, dùng đúng tên file đi kèm. Gói chưa có thư viện tích hợp được compile lại.

Hai đặc tả đã được cập nhật cho bản đóng gói này, thay thế đặc tả cũ dùng vòng nguồn 3,3 V. PCB đã lưu hiện dùng `LOOP_1V8_A/B`, thứ tự chân J102 và cách gán ADC/LED đã thay đổi. Bảng chân được đọc từ net trên PCB, đối chiếu với tên chân schematic. Vẫn cần xác nhận toàn bộ kết nối schematic–PCB trong Altium.

## Các việc cần hoàn tất trước khi phát hành cho sản xuất

- Chạy và lưu báo cáo ERC/DRC trên đúng phiên bản thiết kế cần phát hành.
- Xác nhận kết nối schematic–PCB, model trong thư viện nguồn và hướng lắp linh kiện.
- Cập nhật ActiveBOM; file `.BomDoc` đi kèm có thể còn dữ liệu cache cũ. Với bản này, dùng BOM Excel đã xuất và đọc kỹ danh sách linh kiện không nằm trong BOM.
- Đối chiếu mã mua H101 với connector thực tế và footprint.
- Bổ sung schematic PDF hiện tại, ảnh PCB và bản vẽ cơ khí cụm pogo, đồ gá.
- Triển khai firmware, hiệu chuẩn và ghi log; chốt mọi ngưỡng `TBD` bằng mẫu DUT tốt và mẫu lỗi.
- Xác nhận đúng biến thể Hall sensor, hoạt động tại điện áp cấp DUT và các mức HIGH/LOW khi bình thường, khi hở ngõ ra.
- Xác nhận khả năng phát hiện xung lỗi, cực tính và ngưỡng comparator, hoạt động của nguồn và quy trình vận hành trên JIG đã lắp ráp.

Gerber, CPL và bộ dữ liệu đã chốt để chế tạo chưa thuộc phạm vi bản đóng gói này. Phiên bản phần cứng V1.0 dùng để nhận diện thiết kế, chưa thể hiện việc đã hoàn tất xác nhận cho sản xuất.

## Đưa lên GitHub

Đưa **nội dung bên trong thư mục này** vào thư mục gốc của repo để GitHub hiển thị README và các liên kết đúng vị trí. Bộ này gồm thiết kế và tài liệu prototype. Các mục `TBD`, ngưỡng PASS/FAIL và kết quả đo thực tế vẫn cần hoàn tất trước khi dùng cho sản xuất.
