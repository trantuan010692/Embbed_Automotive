# Embbed_Automotive
# BÀI 3: INTERRUPT và TIMER: 
## 1. Khái niệm Ngắt (Interrupt):

- Ngắt là sự kiện khẩn cấp yêu cầu vi điều khiển dừng chương trình chính và chuyển đến xử lý ngắt (ISR).
- Khi ngắt xảy ra, chương trình lưu giá trị thanh ghi PC vào MSP và thực thi ISR. Sau khi hoàn thành, vi điều khiển tiếp tục thực thi chương trình chính từ địa chỉ đã lưu trong MSP.
- Các khái niệm:
    - PC (Program Counter): Chỉ đến địa chỉ lệnh tiếp theo.
    - ISR (Interrupt Service Routine): Hàm xử lý ngắt.
    - MSP (Main Stack Pointer): Lưu trữ dữ liệu tạm thời khi thực thi chương trình.

## 2. Timer:

- Timer là ngoại vi đếm xung clock. STM32F103C8 có 7 Timer, gồm 4 Timer chính (TIM1, TIM2, TIM3, TIM4) và 3 Timer phụ (Systick, Watchdog).
- Ba yếu tố cần chú ý:
    - Timer Clock: Các Timer có clock là 72 MHz.
    - Prescaler: Bộ chia tần số, tối đa 65535.
    - Period: Giá trị nạp tối đa 65535.

## 3. Cấu hình Timer:

- Cấu hình Timer để đếm đến giá trị 65535 mà không sử dụng ngắt.
- Ví dụ cấu hình Timer 2 để tạo độ trễ 1ms:
    - Prescaler = 7199 (để tạo độ trễ 0,1ms).
    - Period = 65535 (mỗi lần tràn).
    - Hàm delay sử dụng Timer để tạo độ trễ.
        ```c
        #include "stm32f10x.h"  // Thư viện cho STM32F103C8

        // Cấu hình RCC cấp clock cho Timer
        void RCC_Config(){
            RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM2, ENABLE);  // Bật clock cho Timer 2
        }

        // Cấu hình Timer 2
        void TIM_Config(void){
            TIM_TimeBaseInitTypeDef TIM_InitStructure;
            
            // Cấu hình bộ chia clock cho Timer (Prescaler)
            TIM_InitStructure.TIM_ClockDivision = TIM_CKD_DIV1;  // Bộ chia clock cho Timer
            // Cấu hình Timer đếm đến 65535 trong 0,1ms
            // 0,0001s = 0,1ms = 1 / (72 * 10^6) * (PSC + 1) -> PSC = 7199
            TIM_InitStructure.TIM_Prescaler = 7200 - 1;  // Đặt giá trị prescaler là 7199
            TIM_InitStructure.TIM_Period = 0xFFFF;  // Đặt giá trị Period là 65535
            // Cấu hình Timer đếm lên
            TIM_InitStructure.TIM_CounterMode = TIM_CounterMode_Up;
            
            // Cấu hình cho Timer 2
            TIM_TimeBaseInit(TIM2, &TIM_InitStructure);
            
            // Bật Timer 2
            TIM_Cmd(TIM2, ENABLE);
        }

        // Hàm delay 1ms
        void delay_ms(uint32_t delay_Time){
            // Đặt giá trị bộ đếm Timer về 0
            TIM_SetCounter(TIM2, 0);
            
            // Chờ cho đến khi giá trị của bộ đếm lớn hơn delay_Time * 10 (vì timer đếm 0,1ms)
            while (TIM_GetCounter(TIM2) < delay_Time * 10);
        }

        int main(void)
        {
            // Cấu hình RCC và Timer
            RCC_Config();
            TIM_Config();
            
            // Dùng hàm delay để tạo độ trễ 1ms
            while(1){
                delay_ms(1);  // Gọi hàm delay 1ms
                // Thực hiện các công việc khác trong vòng lặp chính
            }
        }
        ```

## 4.Kết luận:

Bài học này hướng dẫn cấu hình Timer và tạo hàm delay 1ms mà không sử dụng ngắt.