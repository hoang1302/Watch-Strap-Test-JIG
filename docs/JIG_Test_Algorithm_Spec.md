# Đặc tả thuật toán test — Watch Strap Test JIG

| Trường | Giá trị |
| --- | --- |
| Mã tài liệu | JIG-FW-TEST-001 |
| Phiên bản | Draft 0.2 — 2026-10-15 |
| Phần cứng | JIG_TEST_FLEX_V1.0 |
| Giao tiếp tham chiếu | [JIG_Hardware_Interface_Spec.md](JIG_Hardware_Interface_Spec.md) |
| Trạng thái | Đề xuất để triển khai prototype; chưa có firmware và dữ liệu xác nhận kèm gói |

Các giá trị khởi tạo dưới đây là đề xuất phát triển. Tất cả ngưỡng `TBD`, điều kiện comparator và thao tác SOP phải được xác nhận trước khi bật PASS sản xuất. Bản này cập nhật nguồn DUT 1,8 V, ADC PA2/PA3, LED PASS riêng và các GPIO hiện tại; không dùng bảng pin của Draft 0.1.

## 1. Điều kiện đạt

Một chu kỳ chỉ được PASS khi cả hai vòng PA và Hall đạt giới hạn điện trở/dòng, không có gián đoạn vượt tiêu chí trong bước uốn, và Hall hoàn thành chu trình HIGH → LOW → HIGH khi đặt rồi bỏ nam châm.

Không có cảm biến xác nhận thao tác uốn hoặc vị trí nam châm. START là xác nhận của người vận hành; SOP phải quy định vị trí nam châm, cách uốn, thời gian và quy tắc gá lại. Tín hiệu comparator là kênh bổ sung, cần xác nhận độc lập trước khi đưa vào quyết định sản xuất.

## 2. Ánh xạ firmware

| Biến | GPIO / phần cứng | Ý nghĩa |
| --- | --- | --- |
| pa_en | PA5 / Q102 | 1 bật đường hồi vòng PA |
| hall_loop_en | PA6 / Q103 | 1 bật đường hồi vòng Hall |
| v_pa_drop | PA0 / V_DUT_PA / INA OUT2 | Rơi áp vòng PA |
| v_pa_shunt | PA1 / V_SH_PA / INA OUT1 | Áp shunt RSH101 |
| v_hall_shunt | PA2 / V_SH_HALL / INA OUT4 | Áp shunt RSH102 |
| v_hall_drop | PA3 / V_DUT_HALL / INA OUT3 | Rơi áp vòng Hall |
| v_hall_out | PA4 / HALL_ADC | Điện áp logic Hall |
| start_pressed | PB1 / SW101 | 0 khi nhấn, debounce theo cạnh mới |
| sw2_pressed | PA7 / SW102 | Dự phòng; chưa dùng để chuyển bước |
| fault_pa_n | PB2 / U105 OUT A | Comparator PA, cực tính cần đo xác nhận |
| fault_hall_loop_n | PB3 / U105 OUT B | Comparator Hall, cực tính cần đo xác nhận |
| led_pass | PA15 / LED105 | 0 sáng |
| led_fail | PB4 / LED106 | 0 sáng |
| led_loop | PB5 / LED104 | 0 sáng |
| led_pa | PB6 / LED102 | 0 sáng |
| led_hall | PB7 / LED103 | 0 sáng |
| led_ready | PB8 / LED101 | 0 sáng |

ADC frame khi quét PA0–PA4 theo thứ tự tăng dần: `[PA_DROP, PA_SHUNT, HALL_SHUNT, HALL_DROP, HALL_OUT]`. Xác nhận cấu hình MCU/DMA trước khi dùng. Mỗi frame cần số thứ tự và dấu thời gian. ADC overrun, thiếu frame hoặc sai thứ tự phải hủy kết quả PASS.

## 3. Tham số

| Tham số | Khởi tạo prototype | Trước sản xuất |
| --- | --- | --- |
| F_FRAME | Đề xuất 5 000 frame/s, 5 kênh/frame | Đo xác nhận tốc độ và nhiễu |
| T_ADC_SETTLE | Đề xuất 20 ms | Xác nhận với mạch thực |
| T_HALL_SETTLE | Đề xuất 250 ms | Xác nhận với đúng sensor và SOP |
| T_BEND_MIN | Đề xuất 10 s | Chốt thao tác và thời gian uốn |
| T_BUTTON_DEBOUNCE | Đề xuất 30 ms | Xác nhận nút thật |
| N_OPEN | Đề xuất 3 frame liên tiếp | Thử xung lỗi và pha lấy mẫu |
| I_PA_MIN, I_HALL_MIN | TBD | Bắt buộc |
| R_PA_MIN/MAX, R_HALL_MIN/MAX | TBD | Bắt buộc; chỉ dùng R_MIN nếu đủ khả năng phân biệt bypass |
| DELTA_R_PA_MAX, DELTA_R_HALL_MAX | TBD | Bắt buộc |
| HALL_LOW_MAX_V, HALL_HIGH_MIN_V | TBD | Đo HIGH/LOW/hở OUT với nguồn Hall 1,8 V |
| N_GLITCH_LIMIT, T_FAULT_MIN | TBD | Bắt buộc |
| T_STEP_TIMEOUT | TBD | Chốt theo SOP |
| Comparator polarity/qualification/filter | TBD | Xác nhận trước khi bật PASS sản xuất |

Không dùng hai tỷ lệ 0,2/0,8 của VDDA làm ngưỡng Hall mặc định. Mức HIGH của Hall được cấp 1,8 V không bằng rail ADC 3,3 V. Điện áp OUT hở lý tưởng khoảng 0,9 V cũng cần đo xác nhận. Lưu `parameter_set_id` và `calibration_id` cho từng bộ cấu hình.

## 4. Khởi tạo và kiểm tra JIG

1. Đặt PA_EN/HALL_EN = 0, tắt mọi LED theo cực tính active-low.
2. Kiểm tra phiên bản firmware, hardware revision, checksum tham số và dữ liệu hiệu chuẩn. Thiếu dữ liệu hợp lệ thì `JIG_FAULT`.
3. Khởi tạo ADC/DMA, xác nhận thứ tự năm kênh, kiểm tra overrun và frame counter.
4. Hiệu chuẩn Vref/VDDA theo phương pháp đã xác nhận cho MCU. Thiết kế không có kênh riêng đo trực tiếp rail 1,8 V; phải đo nguồn trong quy trình kiểm tra JIG, không giả định firmware đã giám sát rail này.
5. Cấu hình và kiểm tra kênh comparator theo cực tính đã đo. Cấu hình chưa xác nhận phải khóa chế độ PASS sản xuất.
6. Vào IDLE. Kiểm tra đầu ca bằng DUT tốt/điện trở chuẩn là bước SOP riêng, không được thay bằng kiểm tra checksum.

Tắt Q102/Q103 không ngắt nguồn toàn bộ DUT. Không dùng số đo nhánh đang tắt để quyết định chất lượng vòng.

## 5. Máy trạng thái

Từ BASELINE đến hết BEND, giữ cả hai nhánh bật và giám sát liên tục. START chỉ được nhận theo cạnh nhấn mới sau debounce; giữ nút không chuyển nhiều bước.

| Trạng thái | Thao tác / hiển thị dự kiến | Điều kiện chuyển |
| --- | --- | --- |
| IDLE | READY sáng; đặt DUT, nam châm ở xa | START → BASELINE |
| BASELINE | READY nhấp nháy; bật nhánh, chờ settle | Dòng/R hợp lệ, Hall HIGH ổn định → WAIT_MAGNET |
| WAIT_MAGNET | HALL nhấp nháy chậm; đặt nam châm đúng vị trí | START → VERIFY_MAGNET |
| VERIFY_MAGNET | Chờ phản hồi sensor | LOW ổn định trong thời gian cho phép → WAIT_REMOVE |
| WAIT_REMOVE | Mẫu HALL riêng theo SOP; bỏ nam châm | START → VERIFY_REMOVE |
| VERIFY_REMOVE | Chờ sensor trở lại | HIGH ổn định → BEND |
| BEND | PA/LOOP nhấp nháy; uốn theo SOP | Hết thời gian tối thiểu và START mới → đánh giá kết quả |
| PASS | PASS sáng, FAIL tắt | Chốt log, tắt nhánh; START mới bắt đầu chu kỳ tiếp |
| FAIL | FAIL sáng, LED nhóm lỗi nhấp nháy | Chốt log, tắt nhánh; thực hiện quy tắc test lại |
| JIG_FAULT | FAIL nhấp nháy khác lỗi DUT, PASS tắt | Cấm PASS tới khi kiểm tra và khắc phục |

Trong BEND, nhấn START quá sớm không kết thúc giám sát. WAIT_MAGNET/WAIT_REMOVE có timeout. Mẫu nhấp nháy, thao tác khởi động chu kỳ tiếp theo và quy tắc gá lại phải được chốt trong SOP.

## 6. Quy đổi và phân loại vòng

Hiệu chuẩn offset/gain riêng cho bốn kênh INA và dùng điện áp ADC đã quy đổi theo Vref thực tế:

```text
Vdiff_PA   = (v_pa_drop    - offset_PA_DROP)  / gain_PA_DROP
Vsh_PA     = (v_pa_shunt   - offset_PA_SHUNT) / gain_PA_SHUNT
Vdiff_HALL = (v_hall_drop  - offset_H_DROP)   / gain_H_DROP
Vsh_HALL   = (v_hall_shunt - offset_H_SHUNT)  / gain_H_SHUNT

I_PA   = Vsh_PA   / RSH101_effective
I_HALL = Vsh_HALL / RSH102_effective

R_PA   = Vdiff_PA   / I_PA      # chỉ khi I_PA hợp lệ
R_HALL = Vdiff_HALL / I_HALL    # chỉ khi I_HALL hợp lệ
```

Đánh dấu INVALID khi ADC gần rail, INA bão hòa, dòng quá nhỏ để chia, giá trị bất thường hoặc mất frame. Không diễn đạt hở vòng bằng một phép chia cho 0 hoặc một số điện trở rất lớn giả tạo.

Dùng hai đường xử lý:

- Đường ổn định: median/average ngắn để lập baseline và kiểm tra cửa sổ R/I.
- Đường lỗi thoáng qua: theo dõi dữ liệu gần raw, lưu thời điểm, độ sâu dòng, đỉnh R và số frame liên tiếp; không dùng bộ lọc dài che xung lỗi.

Một mẫu bất thường được ghi SUSPECT_GLITCH. Quy tắc nâng thành FAIL dựa trên N_OPEN, N_GLITCH_LIMIT và T_FAULT_MIN đã xác nhận. Không loại bỏ mọi mẫu đơn lẻ mà không lưu thống kê.

| Quan sát khi nhánh bật | Phân loại dự kiến |
| --- | --- |
| Dòng dưới I_MIN, rơi áp lớn/bão hòa | Hở vòng hoặc mất tiếp xúc |
| R vượt R_MAX hoặc tăng quá DELTA_R_MAX | Điện trở tăng, nứt Flex hoặc tiếp xúc kém |
| Dòng/R biến động lặp lại khi uốn | Chập chờn |
| R quá thấp trong vùng phân biệt đã chứng minh | Bypass/chập có thể xảy ra |
| Hai vòng cùng bất thường hoặc ADC/nguồn lỗi | Kiểm tra JIG/nguồn/gá trước khi quy lỗi DUT |

R đo bao gồm pogo và tiếp xúc. Hiệu chuẩn không loại bỏ biến thiên tiếp xúc của từng lần gá. Không tuyên bố phân biệt chắc chắn lỗi Flex với lỗi pogo chỉ bằng phép đo hai đầu.

## 7. Hall và comparator

Phân loại Hall bằng điện áp đã hiệu chuẩn: LOW khi `v_hall_out <= HALL_LOW_MAX_V`, HIGH khi `v_hall_out >= HALL_HIGH_MIN_V`, MID ở khoảng giữa. ADC/Vref bất thường cho INVALID. Hai ngưỡng phải có khoảng phân biệt rõ và được đo trên mẫu tốt/hở OUT tại nguồn DUT 1,8 V.

Yêu cầu HIGH khi không có nam châm, LOW khi đặt nam châm đủ mạnh, rồi HIGH khi bỏ nam châm cho đúng biến thể sensor mục tiêu. Hall chuyển logic không thay thế phép thử dẫn điện của vòng Hall. Cần thử hở GND, hở nguồn, hở OUT và cấp ngược qua mạng phân cực.

Comparator U105 so sánh hai đường analog của từng vòng. Firmware cần ghi nhận mức/cạnh và dấu thời gian song song với ADC. Chỉ áp dụng cực tính và thời gian xác nhận đã đo. Khi kênh comparator được dùng làm điều kiện FAIL và đã xác nhận lỗi, chốt FAIL dù đường ADC đã lọc không thấy lỗi. Nếu ADC/comparator mâu thuẫn, lưu cả hai và cấm tự bỏ qua lỗi để PASS; xử lý theo quy tắc JIG_FAULT/FAIL được xác nhận trong validation.

## 8. Kết quả và log

Các mã dự kiến: PASS; PA_OPEN, PA_HIGH_R, PA_GLITCH; HLOOP_OPEN, HLOOP_HIGH_R, HLOOP_GLITCH; HALL_NO_HIGH, HALL_NO_LOW, HALL_NO_RETURN, HALL_MID; OPERATOR_TIMEOUT; JIG_FAULT. Có thể bổ sung cờ comparator khi xác nhận được ý nghĩa lỗi.

Log tối thiểu: timestamp, DUT_ID nếu có, jig_id, hw_revision, fw_version, parameter_set_id, calibration_id, result_code, baseline/min/max I và R của hai vòng, glitch_count, longest_fault_us, chuyển trạng thái Hall, cờ comparator, ADC overrun/mất frame và thao tác START. UART baud rate/protocol chưa được chốt trong snapshot này.

## 9. Bộ kiểm thử trước phát hành

| Mẫu / tác động | Kết quả cần chứng minh |
| --- | --- |
| DUT tốt, đặt/bỏ nam châm, uốn đúng SOP | PASS ổn định, tỷ lệ báo lỗi giả trong mục tiêu |
| Hở PA1/PA2 hoặc vòng Hall | FAIL đúng nhóm, không chia 0 |
| Chèn điện trở chuẩn tại nhiều mức | R chính xác đủ dùng, chuyển ngưỡng đúng |
| Hở OUT, hở GND, hở nguồn Hall | Không PASS; phân loại dựa trên số đo thật |
| Cố định Hall HIGH/LOW | Không hoàn thành chu trình, FAIL |
| Xung hở nhiều độ rộng và nhiều pha ADC | Xác nhận độ rộng lỗi nhỏ nhất và tỷ lệ phát hiện |
| Pogo bẩn/lệch, rung trong giới hạn SOP | Xác nhận quy tắc gá lại và false reject |
| Nguồn yếu/mất, thiếu calibration, ADC overrun | Không phát sinh PASS |
| Thay đổi dòng/R quanh ngưỡng comparator | Xác nhận cực tính, nhiễu, thời gian và tương quan ADC |
| Nhiều lỗi đồng thời | Log được nhiều cờ lỗi |

Thử xung lỗi phải đi qua mạch đo thật, gồm R106/R108, tụ C112, bộ lọc INA/comparator và ADC; dữ liệu giả trong firmware không chứng minh khả năng phát hiện của JIG. Hardware revision, firmware version, calibration và parameter set phải được đóng băng cùng bản phát hành.
