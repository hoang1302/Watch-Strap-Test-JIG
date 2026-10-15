# Đặc tả giao tiếp phần cứng — Watch Strap Test JIG

| Trường | Giá trị |
| --- | --- |
| Mã tài liệu | JIG-HW-IF-001 |
| Phiên bản | Draft 0.2 — 2026-10-15 |
| Phần cứng | JIG_TEST_FLEX_V1.0 |
| Nguồn | [Schematic](../hardware/altium/JIG_TEST_FLEX_V1.0.SchDoc), [PCB](../hardware/altium/JIG_TEST_FLEX_V1.0.PcbDoc) |
| Trạng thái | Đặc tả prototype; chưa xác nhận bằng phép đo board thật |

Tài liệu cập nhật từ Draft 0.1 ngày 01/10. Bảng net/pin dưới đây đọc từ PCB đã lưu, đối chiếu tên chân trong schematic. Đây chưa phải kết quả compile hoặc xác nhận toàn bộ connectivity schematic–PCB. Cần chạy lại kiểm tra trong Altium trước khi chốt firmware/phát hành.

## 1. Kiến trúc nguồn và mạch đo

- J101 nhận nguồn 5 V USB-C; F101 là PTC đầu vào ghi 500 mA.
- U101 TLV62569DBVR tạo rail `3V3` cho MCU U103, INA U104, comparator U105 và LED.
- U102 AMS1117-1.8 tạo rail `LOOP_1V8_A` cho giao tiếp DUT và hai nhánh dòng đo.
- U103 là STM32F030C8T6. U104 là INA4180A1IPWR, bốn kênh gain danh định 20 V/V theo linh kiện đã chọn.
- U105 LMV393IDR tạo hai tín hiệu comparator `FAULT_PA_N`, `FAULT_LOOP_HALL_N`.

Đường dòng theo net PCB:

```text
Vòng PA:
LOOP_1V8_A → R106 (20 Ω) → PA1 → DUT → PA2
           → RSH101 (1 Ω) → Q102 → GND

Vòng Hall:
LOOP_1V8_A → DUT → LOOP_1V8_B → R108 (20 Ω)
           → RSH102 (1 Ω) → Q103 → GND
```

`PA_EN` điều khiển Q102; `HALL_EN` điều khiển Q103. Hai tín hiệu này đóng/ngắt đường hồi dòng, không phải công tắc nguồn toàn bộ DUT. Khi JIG bật, `LOOP_1V8_A` vẫn có nguồn dù Q102/Q103 tắt. Rail 1,8 V là thiết kế danh định; phải đo xác nhận điện áp, tải và nhiệt độ thực tế.

## 2. Connector DUT J102

Thứ tự là số chân điện trên PCB. Chưa có bản vẽ cơ khí xác định chiều nhìn của cụm pogo; phải kiểm tra chân 1 và chiều gá trước khi đấu dây.

| J102 | Net PCB | Chức năng |
| --- | --- | --- |
| 1 | GND | Mass DUT |
| 2 | HALL_OUT | Tín hiệu Hall từ DUT |
| 3 | LOOP_1V8_A | Nguồn DUT và đầu đi của vòng Hall |
| 4 | LOOP_1V8_B | Đầu về vòng Hall |
| 5 | PA2 | Đầu về vòng PA |
| 6 | PA1 | Đầu cấp dòng vòng PA qua R106 |

D102–D106 là các linh kiện PESD5V0U1UA trên đường DUT. Không suy ra mức chịu ESD của cả hệ thống chỉ từ mã linh kiện; cần kiểm tra layout và thử nghiệm thực tế.

## 3. MCU và tín hiệu analog

### 3.1 ADC

| GPIO U103 | Chân package | Net ADC | Nguồn |
| --- | --- | --- | --- |
| PA0 | 10 | V_DUT_PA | U104 OUT2, chân 7; rơi áp vòng PA |
| PA1 | 11 | V_SH_PA | U104 OUT1, chân 1; shunt RSH101 |
| PA2 | 12 | V_SH_HALL | U104 OUT4, chân 14; shunt RSH102 |
| PA3 | 13 | V_DUT_HALL | U104 OUT3, chân 8; rơi áp vòng Hall |
| PA4 | 14 | HALL_ADC | HALL_OUT qua R111 |

**PA2 là shunt Hall, PA3 là rơi áp vòng Hall.** Firmware phải kiểm tra thứ tự DMA theo bảng này. VDDA nối `3V3`; các kênh được quét tuần tự, không đồng thời.

Các đầu vào U104 dùng điện trở RS101–RS108 = 4,7 Ω và tụ vi sai C121/C123/C125/C129 = 1 nF. Các đầu ra đi qua điện trở 470 Ω và tụ 6,8 nF trước ADC. C112 = 100 nF nối từ PA1 xuống GND. Tất cả các bộ lọc và pha lấy mẫu phải được tính đến khi xác nhận khả năng bắt xung lỗi.

### 3.2 GPIO điều khiển và input

| GPIO | Chân | Net | Chức năng |
| --- | --- | --- | --- |
| PA5 | 15 | PA_EN | Điều khiển gate Q102 qua R113; mức cao dự kiến bật nhánh PA |
| PA6 | 16 | HALL_EN | Điều khiển gate Q103 qua R114; mức cao dự kiến bật nhánh Hall |
| PB1 | 19 | START_BUTTON | SW101, active-low; cần debounce |
| PA7 | 17 | SW2 | SW102, active-low; chưa gán chức năng trong thuật toán draft |
| PB11 | 22 | SW3 | Mạch input dự phòng qua R136; chưa có nút SW3 vật lý |
| PB2 | 20 | FAULT_PA_N | Comparator U105 OUT A, chân 1 |
| PB3 | 39 | FAULT_LOOP_HALL_N | Comparator U105 OUT B, chân 7 |

Hai net comparator có pull-up 4,7 kΩ lên 3V3 và tụ 1 nF xuống GND. Tên `_N` biểu thị ý định lỗi mức thấp; cần đo xác nhận cực tính, điểm chuyển, nhiễu và độ rộng xung trước khi dùng làm điều kiện quyết định. Comparator không thay thế hiệu chuẩn và phép đo ADC.

### 3.3 LED

| GPIO | Chân | Net | LED |
| --- | --- | --- | --- |
| PA15 | 38 | PASS_LED | LED105 |
| PB4 | 40 | FAIL_LED | LED106 |
| PB5 | 41 | 1V8_LOOP_LED | LED104 |
| PB6 | 42 | PA_LED | LED102 |
| PB7 | 43 | HALL_LED | LED103 |
| PB8 | 45 | READY_LED | LED101 |

Các LED được cấp từ 3V3 qua điện trở 680 Ω; phía MCU kéo xuống để sáng. Mức GPIO 0 = sáng, 1 = tắt theo kết nối thiết kế. Firmware phải khởi tạo trạng thái tắt trước khi hiển thị kết quả. Trạng thái LED khi reset/hi-Z không phải kết quả test.

## 4. Header debug H101

| H101 | Net | Kết nối MCU |
| --- | --- | --- |
| 1 | 3V3 | Điện áp tham chiếu probe |
| 2 | NRST | U103 chân 7 |
| 3 | SWCLK | PA14, chân 37 |
| 4 | SWDIO | PA13, chân 34 |
| 5 | MCU_RX | PA10, chân 31 |
| 6 | MCU_TX | PA9, chân 30 |
| 7 | GND | GND chung |

Ưu tiên nạp qua SWD. UART dùng mức logic 3,3 V; baud rate và protocol chưa được chốt. Nối TX của USB-UART sang RX của MCU và ngược lại. Xác nhận cấu hình nguồn của probe trước khi kết nối chân 3V3.

## 5. Mô hình đo và Hall

Với G là gain đã hiệu chuẩn, V là điện áp sau trừ offset, shunt danh định 1 Ω:

```text
I_PA   = V_SH_PA   / (G_SH_PA × RSH101)
I_HALL = V_SH_HALL / (G_SH_HALL × RSH102)
R_PA   = (V_DUT_PA   / G_DUT_PA)   / I_PA
R_HALL = (V_DUT_HALL / G_DUT_HALL) / I_HALL
```

Chỉ tính R khi nhánh bật, dòng đủ lớn, ADC hợp lệ và INA không bão hòa. R gồm đường DUT, pogo/tiếp xúc và phần đường dẫn liên quan; không tự phân biệt được nứt Flex với lỗi tiếp xúc từ một lần đo.

R111 = 1 kΩ nối HALL_OUT đến HALL_ADC; R110 = 100 kΩ kéo HALL_ADC lên rail 1,8 V, R112 = 100 kΩ kéo xuống GND; C120 = 1 nF. Khi OUT hở, mô hình điện trở lý tưởng cho mức khoảng 0,9 V, nhưng rò/cấp ngược có thể làm thay đổi giá trị thực.

DUT mục tiêu trong đặc tả trước dùng TMAG5131G1D, đầu ra push-pull active-low. Sensor thuộc DUT, không nằm trong BOM JIG. Cần xác nhận biến thể, dải nguồn và chu trình HIGH → LOW → HIGH trên DUT thật. **Không dùng ngưỡng HIGH = 0,8 × VDDA của bản draft 3,3 V cũ:** Hall được cấp 1,8 V trong thiết kế hiện tại. Ngưỡng phân loại phải dựa trên điện áp đo thực tế với Vref ADC đã hiệu chuẩn.

## 6. Các mục phải xác nhận

- Compile và kiểm tra connectivity schematic–PCB; ERC/DRC trên đúng bản phát hành.
- Footprint, cực tính linh kiện và chiều connector/pogo.
- Nguồn 3V3/1V8 khi hai nhánh bật; nhiệt độ LDO và điện trở hạn dòng.
- Offset/gain, vùng bão hòa và độ phân giải của bốn kênh INA.
- ADC/DMA: thứ tự kênh, tốc độ frame và xử lý overrun.
- Comparator: cực tính, ngưỡng, hysteresis/nhiễu và thời gian đáp ứng.
- Hall: mức HIGH/LOW/MID trên mẫu tốt, hở OUT, hở nguồn và hở GND.
- Hiệu chuẩn và ngưỡng PASS/FAIL theo [đặc tả thuật toán](JIG_Test_Algorithm_Spec.md).

## 7. Tài liệu linh kiện

- [TI INA4180](https://www.ti.com/lit/ds/symlink/ina4180.pdf).
- [TI LMV393](https://www.ti.com/lit/ds/symlink/lmv393.pdf).
- [TI TMAG5131-Q1](https://www.ti.com/lit/ds/symlink/tmag5131-q1.pdf).
- [ST STM32F030C8](https://www.st.com/en/microcontrollers-microprocessors/stm32f030c8.html).

Các liên kết hỗ trợ đối chiếu linh kiện. Trạng thái prototype trong tài liệu này không phải chứng nhận tuân thủ hoặc kết quả đo phần cứng.
