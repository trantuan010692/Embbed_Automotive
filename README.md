# Embbed_Automotive
# BÀI 1: FIRST PROJECT WITH KEILC
# 1. Tổng quan về KeilC

- **KeilC** là IDE hỗ trợ viết, biên dịch, debug code, upload chương trình vào MCU.  
- Tích hợp mô phỏng, chạy thử code mà không cần phần cứng.  
- Hỗ trợ nhiều vi điều khiển: 8051, ARM, AVR,...  
- Trình biên dịch tương thích chuẩn C/C++.  

---

# 2. Lập trình thanh ghi

## Đề bài: Nháy LED PC13
Để lập trình nháy LED PC13, chúng ta cần xác định được thanh ghi nào cấp clock điều khiển cho ngoại vi GPIO, và một thanh ghi nữa dùng để cấu hình và điều khiển GPIO (dựa trên RM của MCU).

### a) Thanh ghi RCC (Reset & Clock CONTROL)

Trong thanh ghi **RCC**, tìm đến **APB2 PERIPHERAL CLOCK ENABLE**:
- **Địa chỉ offset:** `0x18`
- **Bit từ 2 -> 8** là các bit bật xung cho các IOP từ A -> G. Để bật IOP, set bit tương ứng lên `1`.

```c
#define RCC_APB2ENR *((unsigned int *)0x40021018)
RCC_APB2ENR |= (1 << 4); // Bật clock điều khiển cho GPIOC
```
### Giải thích

- **`unsigned int`**:  
  Đảm bảo giá trị trỏ tới là kiểu `int` 32 bit không âm, phù hợp với kích thước thanh ghi trên STM32 MCU.

- **Địa chỉ `0x40021018`**:  
  - Xem bảng "Register boundary addresses" trong "Memory map" của Reference Manual (RM).  
  - **RCC** bắt đầu từ địa chỉ: `0x40021000`.  
  - **Offset:** `0x18`.  
  - Địa chỉ cuối cùng: `0x40021000 + 0x18 = 0x40021018`.

## b) Thanh ghi GPIO (GPIO Register)

Trong thanh ghi này, sử dụng các thanh ghi cấu hình và dữ liệu:

- **`GPIOx_CRH` (Port configuration register high):**  
  - **Địa chỉ offset:** `0x04`  
  - Cấu hình các chân từ 8 -> 15 với cặp bit `MODE` và `CNF`.

- **`GPIOx_ODR` (Port output data register):**  
  - **Địa chỉ offset:** `0x0C`  
  - Điều khiển mức logic của các chân ở chế độ đầu ra.

```c
#define GPIOC_CRH (*(unsigned int *)0x40011004)
#define GPIOC_ODR *((unsigned int *)0x4001100C)
```

## c) Source code "Nháy LED PC13"

```c
#define RCC_APB2ENR *((unsigned int *)0x40021018)
#define GPIOC_CRH *((unsigned int *)0x40011004)
#define GPIOC_ODR *((unsigned int *)0x4001100C)

int main(void)
{
    // Bật clock điều khiển APB2ENR cho GPIOC
    RCC_APB2ENR |= (1 << 4);
    
    // Cấu hình GPIOC
    GPIOC_CRH &= ~((1 << 22) | (1 << 23)); // Chế độ 00: Output push-pull
    GPIOC_CRH |= (1 << 20) | (1 << 21);    // Chế độ 11: Output mode, 50MHz

    while (1)
    {
        GPIOC_ODR |= (1 << 13); // Bật LED
        for (unsigned int i = 0; i < 1000000; i++);
        GPIOC_ODR &= ~(1 << 13); // Tắt LED
        for (unsigned int i = 0; i < 1000000; i++);
    }
    return 0;
}
```
---

# 3. Dùng API  

## Đề bài: Đọc trạng thái nút nhấn  

Tương tự như trên, chúng ta sẽ bật clock điều khiển và cấu hình GPIO, nhưng bài này sử dụng API và `struct` để truy cập các thanh ghi.  

---

### a) Tạo `struct` cho RCC và GPIO  
```c
typedef struct
{
    unsigned int CR;
    unsigned int CFGR;
    unsigned int CIR;
    unsigned int APB2RSTR;
    unsigned int APB1RSTR;
    unsigned int AHBENR;
    unsigned int APB2ENR;
    unsigned int APB1ENR;
    unsigned int BDCR;
    unsigned int CSR;
} RCC_TypeDef;

typedef struct
{
    unsigned int CRL;
    unsigned int CRH;
    unsigned int IDR;
    unsigned int ODR;
    unsigned int BSRR;
    unsigned int BRR;
    unsigned int LCKR;
} GPIO_TypeDef;
```

Địa chỉ cơ bản của RCC và GPIO:
```c
Địa chỉ cơ bản của RCC và GPIO:
```

### b) Hàm ghi giá trị GPIO dùng API
```c
void WritePin(GPIO_TypeDef *GPIO_Port, unsigned char Pin, unsigned char state)
{
    if (state == 1)
        GPIO_Port->ODR |= (1 << Pin);
    else
        GPIO_Port->ODR &= ~(1 << Pin);
}
```
Ví dụ: Ghi chân PC13 lên 1:

```c
WritePin(GPIOC, 13, 1);
```

### c) Cấu hình GPIOA để nhận tín hiệu nút nhấn
Bật clock điều khiển APB2ENR cho GPIOA:

```c
RCC->APB2ENR |= (1 << 2);
```
Cấu hình PA0 là chế độ Input pull-up/pull-down:

```c
GPIOA->CRL &= ~((1 << 0) | (1 << 1) | (1 << 2)); // MODE = 00: Input mode
GPIOA->CRL |= (1 << 3);                          // CNF = 10: Input pull-up/pull-down
GPIOA->ODR |= (1 << 0);                          // Chốt pull-up
```
Đọc trạng thái nút nhấn:

```c
if ((GPIOA->IDR & (1 << 0)) == 0)
{
    WritePin(GPIOC, 13, 0); // Tắt LED
}
else
{
    WritePin(GPIOC, 13, 1); // Bật LED
}
```

### d) Source code "Đọc trạng thái nút nhấn"

```c
typedef struct
{
    unsigned int CR;
    unsigned int CFGR;
    unsigned int CIR;
    unsigned int APB2RSTR;
    unsigned int APB1RSTR;
    unsigned int AHBENR;
    unsigned int APB2ENR;
    unsigned int APB1ENR;
    unsigned int BDCR;
    unsigned int CSR;
} RCC_TypeDef;

typedef struct
{
    unsigned int CRL;
    unsigned int CRH;
    unsigned int IDR;
    unsigned int ODR;
    unsigned int BSRR;
    unsigned int BRR;
    unsigned int LCKR;
} GPIO_TypeDef;

#define RCC ((RCC_TypeDef *)0x40021000)
#define GPIOA ((GPIO_TypeDef *)0x40010800)
#define GPIOC ((GPIO_TypeDef *)0x40011000)

void WritePin(GPIO_TypeDef *GPIO_Port, unsigned char Pin, unsigned char state)
{
    if (state == 1)
        GPIO_Port->ODR |= (1 << Pin);
    else
        GPIO_Port->ODR &= ~(1 << Pin);
}

void ConfigureGPIO(void)
{
    RCC->APB2ENR |= (1 << 2) | (1 << 4); // Bật clock cho GPIOA, GPIOC

    // Cấu hình PA0: Input pull-up/pull-down
    GPIOA->CRL &= ~((1 << 0) | (1 << 1) | (1 << 2)); 
    GPIOA->CRL |= (1 << 3);                         
    GPIOA->ODR |= (1 << 0);                        

    // Cấu hình PC13: Output push-pull
    GPIOC->CRH &= ~((1 << 20) | (1 << 21));         
    GPIOC->CRH |= ((1 << 22) | (1 << 23));         
}

int main(void)
{
    ConfigureGPIO();

    while (1)
    {
        if ((GPIOA->IDR & (1 << 0)) == 0)
        {
            WritePin(GPIOC, 13, 0);
        }
        else
        {
            WritePin(GPIOC, 13, 1);
        }
    }
    return 0;
}

```

# Tóm tắt lợi ích của API
- Code dễ đọc, dễ bảo trì.
- Định nghĩa và sử dụng struct giúp truy cập thanh ghi linh hoạt hơn.
