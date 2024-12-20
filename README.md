# Embbed_Automotive
# BÀI 2: GPIO
# 1. Giới thiệu về SPL

- SPL (Standard Peripherals Library) là thư viện chuẩn từ STMicroelectronics hỗ trợ lập trình vi điều khiển STM32.
- Thư viện giúp đơn giản hóa lập trình và cấu hình các ngoại vi bằng API, thay vì thao tác trực tiếp với thanh ghi.

# 2.Giới thiệu về GPIO

- GPIO (General Purpose Input/Output) là ngoại vi của STM32, có thể cấu hình thông qua SPL.
- Các thư viện cần thiết: stm32f10x_gpio.h và stm32f10x_gpio.c.

Cấu hình cơ bản:

- RCC cấp clock cho GPIO: Sử dụng hàm RCC_APB2PeriphClockCmd() để cấp clock cho GPIO.
    ```c
    void RCC_Config(void){
        RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);
    }
    ```
    
- Cấu hình GPIO: Sử dụng GPIO_InitTypeDef để thiết lập các thuộc tính của GPIO (chân, chế độ, tốc độ) và khởi tạo qua hàm GPIO_Init().
    ```c
    void GPIO_Config(void){
        GPIO_InitTypeDef GPIO_InitStructure;
        GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;
        GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
        GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
        GPIO_Init(GPIOC, &GPIO_InitStructure);
    }
    ```

# 3.Thực hành

### a) Nháy LED chân PC13

- Cấu hình RCC và GPIO, sau đó điều khiển LED nháy trên chân PC13.
    ```c
    /* Hàm cấu hình RCC */
    void RCC_Config(void){
        RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);
    }

    /* Hàm cấu hình GPIO */
    void GPIO_Config(void){
        GPIO_InitTypeDef GPIO_InitStructure;
        GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;
        GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
        GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
        GPIO_Init(GPIOC, &GPIO_InitStructure);
    }

    /* Hàm main nháy LED */
    int main(void){
        RCC_Config();
        GPIO_Config();
        while(1){
            GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_SET);
            for (unsigned int i = 0; i < 1000000; i++);
            GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_RESET);
            for (unsigned int i = 0; i < 1000000; i++);
        }
    }
    ```
### b) Nháy đuổi LED

- Điều khiển LED theo hiệu ứng đuổi (từ chân GPIO 5 đến 8).
    ```c

    /* Hàm cấu hình RCC */
    void RCC_Config(void){
        RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);
    }

    /* Hàm cấu hình GPIO */
    void GPIO_Config(void){
        GPIO_InitTypeDef GPIO_InitStructure;
        GPIO_InitStructure.GPIO_Pin = GPIO_Pin_5 | GPIO_Pin_6 | GPIO_Pin_7 | GPIO_Pin_8;
        GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
        GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
        GPIO_Init(GPIOC, &GPIO_InitStructure);
    }

    /* Hàm đuổi LED */
    void chasing_LED(uint8_t chasing_Times){
        uint16_t value_LED;
        for (int j = 0; j < chasing_Times; j++){
            value_LED = 0x0010;
            for (int i = 0; i < 4; i++){
                value_LED <<= 1;
                GPIO_Write(GPIOC, value_LED);
                for (unsigned int i = 0; i < 1000000; i++);
            }
        }
    }

    /* Hàm main đuổi LED */
    int main(void){
        RCC_Config();
        GPIO_Config();
        while(1){
            chasing_LED(4); // 4 lần nháy đuổi LED
            break; // Dừng sau khi nháy
        }
    }
    ```
### c) Bật/tắt LED qua nút nhấn

- Sử dụng nút nhấn (GPIOA, PA0) để điều khiển trạng thái của LED PC13.
    ```c
    /* Cấu hình RCC */
    void RCC_Config(void){
        RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA | RCC_APB2Periph_GPIOC, ENABLE);
    }

    /* Cấu hình GPIO */
    void GPIO_Config(void){
        GPIO_InitTypeDef GPIO_InitStructure;
        // Cấu hình cho GPIOC chân PC13
        GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;
        GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
        GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
        GPIO_Init(GPIOC, &GPIO_InitStructure);
        // Cấu hình cho GPIOA chân PA0 (nút nhấn)
        GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
        GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU; // Input Pull-up
        GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
        GPIO_Init(GPIOA, &GPIO_InitStructure);
    }

    /* Hàm main cho nút nhấn */
    int main(void){
        RCC_Config();
        GPIO_Config();
        while(1){
            if (GPIO_ReadInputDataBit(GPIOA, GPIO_Pin_0) == Bit_RESET){ // Nút nhấn
                while (GPIO_ReadInputDataBit(GPIOA, GPIO_Pin_0) == Bit_RESET); // Chờ nhả nút
                // Đảo trạng thái LED PC13
                if (GPIO_ReadOutputDataBit(GPIOC, GPIO_Pin_13) == Bit_SET){
                    GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_RESET);
                } else {
                    GPIO_WriteBit(GPIOC, GPIO_Pin_13, Bit_SET);
                }
            }
        }
    }
    ```

# 4.Kết luận

- Việc sử dụng SPL giúp việc lập trình cho vi điều khiển STM32 trở nên đơn giản và dễ hiểu hơn, giúp người lập trình dễ dàng thao tác với các ngoại vi mà không cần phải làm việc trực tiếp với thanh ghi.