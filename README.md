# STM32F103C8T6 – Ví dụ ADC với DMA và USART

Dự án này minh họa cách sử dụng **ADC1 kết hợp với DMA (Direct Memory Access)** trên STM32F103C8T6 (Blue Pill) để liên tục đọc tín hiệu analog từ chân **PA0 (ADC Channel 0)** và gửi kết quả sang máy tính qua **USART1** (TX: PA9, RX: PA10).
Code sử dụng **Standard Peripheral Library (SPL)**.
---
## Tính năng
* Cấu hình **USART1** ở tốc độ 9600 baud để truyền dữ liệu nối tiếp.
* Cấu hình **ADC1** ở chế độ liên tục, đọc từ **Channel 0 (PA0)**.
* Sử dụng **DMA1 Channel1** để tự động chuyển dữ liệu ADC vào RAM mà không cần CPU can thiệp.
* Khi DMA truyền xong, phát sinh ngắt và gửi giá trị ADC ra USART.
* CPU trong vòng lặp chính hoàn toàn rảnh, mọi việc do ngoại vi xử lý.
---
## Kết nối chân

* **PA0** → Ngõ vào analog (kết nối biến trở hoặc cảm biến 0–3.3V).
* **PA9** → USART1 TX (nối với RX của USB–TTL).
* **PA10** → USART1 RX (nối với TX của USB–TTL, tùy chọn nếu chỉ cần truyền).
* **GND** → Chung mass với USB–TTL.
---
## Quy trình hoạt động
1. **USART1_Init()**
   * Bật clock cho GPIOA và USART1.
   * Cấu hình PA9 là **AF Push-Pull** (TX), PA10 là **Input Floating** (RX).
   * Khởi tạo USART1: 9600 baud, 8 bit dữ liệu, 1 stop bit, không parity.
2. **ADC1_DMA_Config()**
   * Bật clock cho ADC1, GPIOA và DMA1.
   * PA0 cấu hình làm analog input.
   * Cấu hình **DMA1 Channel1**:
     * Địa chỉ ngoại vi = `ADC1->DR`.
     * Địa chỉ bộ nhớ = biến `ADC_ConvertedValue`.
     * Kích thước dữ liệu 16 bit.
     * Chế độ **Circular** để chạy liên tục.
     * Cho phép ngắt khi truyền xong.
   * Cấu hình ADC1:
     * Independent mode, single channel, continuous conversion.
     * Kích hoạt align phải, số kênh = 1.
     * Channel 0, sample time 55.5 cycles.
   * Bật DMA cho ADC1.
   * Reset và hiệu chuẩn ADC.
   * Bắt đầu quá trình chuyển đổi bằng phần mềm.
3. **Ngắt DMA**
   * Mỗi lần ADC hoàn tất, DMA sẽ ghi giá trị vào `ADC_ConvertedValue`.
   * Khi DMA báo **Transfer Complete**, ngắt xảy ra.
     Trong ISR (`DMA1_Channel1_Event()`) giá trị ADC được format bằng `sprintf()` và gửi qua USART
4. **Hàm main()**
   * Vòng lặp chính không cần làm gì, dữ liệu được tự động cập nhật và gửi đi.
---
## Kết quả mong đợi
Khi nối PA0 vào điện áp thay đổi (ví dụ biến trở) và mở Serial Terminal ở **9600 baud**, bạn sẽ thấy dữ liệu liên tục in ra:
```
Gia tri ADC: 1023
Gia tri ADC: 2047
Gia tri ADC: 3071
...
```
Giá trị trong khoảng **0 – 4095** (12-bit ADC).
---

## Yêu cầu
* Kit STM32F103C8T6 (Blue Pill).
* Thư viện SPL cài đặt sẵn.
* Bộ chuyển đổi USB–TTL.
* IDE: Keil uVision / STM32CubeIDE hoặc bất kỳ toolchain ARM nào hỗ trợ SPL.
---
## Lưu ý
* ISR mặc định của DMA1 là `DMA1_Channel1_IRQHandler`, bạn cần gọi `DMA1_Channel1_Event()` trong đó:
  ```c
  void DMA1_Channel1_IRQHandler(void) {
      if(DMA_GetITStatus(DMA1_IT_TC1)) {
          DMA_ClearITPendingBit(DMA1_IT_TC1);
          DMA1_Channel1_Event(DMA_ISR_TCIF1);
      }
  }
  ```
* Clock hệ thống cần được cấu hình đúng (thường HSE = 8 MHz, SYSCLK = 72 MHz).
* Có thể thay đổi baudrate hoặc bật RX để nhận dữ liệu từ PC nếu cần.
---



Ví dụ hệ thống nhúng – STM32F103C8T6
