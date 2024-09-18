---
number-depth: 2
---

# 3-bit LED Counter Using HAL Delay, Timer Interrupts, and Button Interrupt

## Objective:
By the end of this exercise, students will:
- Implement a binary counter using three LEDs (LD1, LD2, LD3) on the STM Nucleo-144 F767ZI board.
- Explore three methods of controlling the LEDs: using **HAL delay** and **for loops**, **timer interrupts**, and **push button interrupts**.
- Understand the difference between using blocking delays, interrupt-driven operations, and external interrupts in embedded systems.

## Materials:
- STM Nucleo-144 F767ZI development board.
- Onboard user LEDs:
  - **LD1 (Green)**: PB0 (or PA5).
  - **LD2 (Blue)**: PB7.
  - **LD3 (Red)**: PB14.
- Onboard user push button (USER_BUTTON): Connected to **PC13**.
- CubeIDE installed on the PC.

---

## CubeMX Configuration:

### 1. GPIO Configuration for LEDs:
   - **LD1 (Green)**: Set PB0 (or PA5 depending on SB settings) as a GPIO Output.
   - **LD2 (Blue)**: Set PB7 as a GPIO Output.
   - **LD3 (Red)**: Set PB14 as a GPIO Output.

### 2. Timer Configuration (For Task 2):
   - Enable **TIM3** (or any other available timer).
   - Configure the timer with a **prescaler** and **period** to generate an interrupt every 1 second.
   - Enable the **Update Interrupt** for the timer (TIM3).

### 3. Button Configuration (For Task 3):
   - Set **PC13** (USER_BUTTON) as a GPIO Input.
   - Enable the **EXTI Line 13** interrupt for the button in the **NVIC Settings**.

### 4. Generate Code:
   - Click **Project > Generate Code** after setting up GPIO, Timer, and Button.

---

## Explanation of Key Functions:

1. **HAL_Delay:**
   - Generates a blocking delay, pausing program execution for the specified number of milliseconds.
   - Syntax:
     ```c
     HAL_Delay(milliseconds);
     ```

2. **HAL_GPIO_WritePin:**
   - Sets the state of a GPIO pin to HIGH or LOW, used to control the LEDs.
   - Syntax:
     ```c
     HAL_GPIO_WritePin(GPIOx, GPIO_Pin, PinState);
     ```
   - Example:
     ```c
     HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_SET);  // Set PB0 (LD1) to HIGH
     HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, GPIO_PIN_RESET); // Set PB0 (LD1) to LOW
     ```

3. **HAL_TIM_Base_Start_IT:**
   - Starts a timer in interrupt mode.
   - Syntax:
     ```c
     HAL_TIM_Base_Start_IT(&htim3);  // Start TIM3 in interrupt mode
     ```

4. **HAL_TIM_PeriodElapsedCallback:**
   - The interrupt callback function triggered when the timer reaches the specified period.
   - Syntax:
     ```c
     void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim);
     ```

5. **HAL_GPIO_EXTI_IRQHandler:**
   - Handles external interrupt requests for GPIO pins.
   - Syntax:
     ```c
     void HAL_GPIO_EXTI_IRQHandler(uint16_t GPIO_Pin);
     ```

6. **__HAL_GPIO_EXTI_CLEAR_IT:**
   - Clears the pending bit for the external interrupt line.
   - Syntax:
     ```c
     __HAL_GPIO_EXTI_CLEAR_IT(GPIO_Pin);
     ```

---

## Tasks and Sample IO Behavior:

### Task 1: 3-bit LED Counter Using HAL Delay and For Loop

- **Objective**: Create a 3-bit binary counter using the LEDs, with a delay between each count, controlled by `HAL_Delay()` in the main loop.

- **Description**: 
  - Use a for loop in the `main()` function to count from 0 to 7 (binary `000` to `111`).
  - Control the LEDs based on the binary value of the counter.
  - Use `HAL_Delay()` to wait for 1 second between each count.

#### Code Example
```c
#include "main.h"

volatile uint8_t counter = 0;  // 3-bit counter

void increment_counter(void) {
    counter = (counter + 1) % 8;  // Increment counter and wrap around using modulo
    // Set LED states based on counter value
    HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, (counter & 0x01) ? GPIO_PIN_SET : GPIO_PIN_RESET);  // LD1 (LSB)
    HAL_GPIO_WritePin(GPIOB, GPIO_PIN_7, (counter & 0x02) ? GPIO_PIN_SET : GPIO_PIN_RESET);  // LD2
    HAL_GPIO_WritePin(GPIOB, GPIO_PIN_14, (counter & 0x04) ? GPIO_PIN_SET : GPIO_PIN_RESET); // LD3 (MSB)
}

int main(void) {
    HAL_Init();  // Initialize the HAL Library
    SystemClock_Config();  // Configure the system clock

    MX_GPIO_Init();  // Initialize GPIO for LEDs

    while (1) {
        // 3-bit binary counter loop (0 to 7)
        for (uint8_t i = 0; i < 8; i++) {
            increment_counter();  // Update LED states based on counter value
            HAL_Delay(1000);  // 1-second delay
        }
    }
}
```

#### Sample IO:
- **Output**: The LEDs will count in binary, with a 1-second delay between each count.
  - **000**: All LEDs off.
  - **001**: LD1 on.
  - **010**: LD2 on.
  - **011**: LD1 and LD2 on.
  - **100**: LD3 on.
  - **101**: LD1 and LD3 on.
  - **110**: LD2 and LD3 on.
  - **111**: All LEDs on.

---

### Task 2: 3-bit LED Counter Using Timer Interrupt

- **Objective**: Create a 3-bit binary counter using the LEDs, with the counter updated by a timer interrupt instead of using `HAL_Delay()`.

- **Description**: 
  - Use a timer to generate an interrupt every 1 second.
  - Inside the interrupt handler, increment the counter and update the LEDs to reflect the binary count.

#### Code Example
```c
#include "main.h"

volatile uint8_t counter = 0;  // 3-bit counter

void increment_counter(void) {
    counter = (counter + 1) % 8;  // Increment counter and wrap around using modulo
    // Set LED states based on counter value
    HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, (counter & 0x01) ? GPIO_PIN_SET : GPIO_PIN_RESET);  // LD1 (LSB)
    HAL_GPIO_WritePin(GPIOB, GPIO_PIN_7, (counter & 0x02) ? GPIO_PIN_SET : GPIO_PIN_RESET);  // LD2
    HAL_GPIO_WritePin(GPIOB, GPIO_PIN_14, (counter & 0x04) ? GPIO_PIN_SET : GPIO_PIN_RESET); // LD3 (MSB)
}

int main(void) {
    HAL_Init();  // Initialize the HAL Library
    SystemClock_Config();  // Configure the system clock

    MX_GPIO_Init();  // Initialize GPIO for LEDs
    MX_TIM3_Init();  // Initialize Timer (TIM3)

    // Start the timer in interrupt mode
    HAL_TIM_Base_Start_IT(&htim3);

    // Infinite loop
    while (1) {
        // No additional logic needed in the main loop, everything is handled in the timer interrupt
    }
}

// Timer interrupt callback
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim) {
    if (htim->Instance == TIM3) {
        increment_counter();  // Update LED states based on counter value
    }
}
```

#### Sample IO:
- **Output**: The LEDs will count in binary, with a 1-second delay between each count, controlled by the timer interrupt.
  - **000**: All LEDs off.
  - **001**: LD1 on.
  - **010**: LD2 on.
  - **011**: LD1 and LD2 on.
  - **100**: LD3 on.
  - **101**: LD1 and LD3 on.
  - **110**: LD2 and LD3 on.
  - **111**: All LEDs on.

---

### Task 3: 3-bit LED Counter with Push Button Interrupt and Debounce

- **Objective:** Create a 3-bit binary counter using the LEDs, where the counter increments each time the push button is pressed, with debounce handling to ensure reliable button presses.

- **Description:**
  - Configure the push button (USER) connected to **PC13** to generate an interrupt on a button press.
  - Implement a debounce mechanism to filter out false triggers due to bouncing.
  - In the interrupt handler, increment the counter and update the LEDs to reflect the new binary count.

#### Code Example

```c
#include "main.h"

#define DEBOUNCE_DELAY_MS 50  // Debounce delay in milliseconds

volatile uint8_t counter = 0;  // 3-bit counter
volatile uint32_t last_interrupt_time = 0;  // Last interrupt time in milliseconds

void increment_counter(void) {
    counter = (counter + 1) % 8;  // Increment counter and wrap around using modulo
    // Set LED states based on counter value
    HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0, (counter & 0x01) ? GPIO_PIN_SET : GPIO_PIN_RESET);  // LD1 (LSB)
    HAL_GPIO_WritePin(GPIOB, GPIO_PIN_7, (counter & 0x02) ? GPIO_PIN_SET : GPIO_PIN_RESET);  // LD2
    HAL_GPIO_WritePin(GPIOB, GPIO_PIN_14, (counter & 0x04) ? GPIO_PIN_SET : GPIO_PIN_RESET); // LD3 (MSB)
}

void delay_ms(uint32_t ms) {
    HAL_Delay(ms);  // Simple delay function using HAL_Delay
}

int main(void) {
    HAL_Init();  // Initialize the HAL Library
    SystemClock_Config();  // Configure the system clock

    MX_GPIO_Init();  // Initialize GPIO for LEDs and Button
    MX_TIM3_Init();  // Initialize Timer (TIM3)
    
    // Start the timer in interrupt mode
    HAL_TIM_Base_Start_IT(&htim3);

    // Infinite loop
    while (1) {
        // No additional logic needed in the main loop, everything is handled in interrupts
    }
}

// Timer interrupt callback
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim) {
    if (htim->Instance == TIM3) {
        // Timer interrupt logic can be added here if needed
    }
}

// Button interrupt callback
void EXTI15_10_IRQHandler(void) {
    uint32_t current_time = HAL_GetTick();  // Get current time in milliseconds

    // Check if the interrupt is from the button pin (PC13) and debounce
    if (__HAL_GPIO_EXTI_GET_IT(GPIO_PIN_13) != RESET) {
        // Simple debounce logic
        if ((current_time - last_interrupt_time) > DEBOUNCE_DELAY_MS) {
            last_interrupt_time = current_time;  // Update last interrupt time
            
            __HAL_GPIO_EXTI_CLEAR_IT(GPIO_PIN_13);  // Clear the interrupt flag
            increment_counter();  // Update LED states based on counter value
        }
    }
}
```

##### Sample IO
- **Output:** Each press of the push button increments the counter and updates the LEDs to reflect the binary count.
  - **000:** All LEDs off.
  - **001:** LD1 on.
  - **010:** LD2 on.
  - **011:** LD1 and LD2 on.
  - **100:** LD3 on.
  - **101:** LD1 and LD3 on.
  - **110:** LD2 and LD3 on.
  - **111:** All LEDs on.

**Note:** The debounce delay is set to 50 milliseconds in this example, but you can adjust this value depending on your specific hardware and requirements.


## Conclusion:

In this exercise, you implemented a 3-bit binary counter using three different methods of LED control on the STM Nucleo-144 F767ZI board:

- **HAL Delay and For Loop:** Demonstrated a straightforward approach for controlling LEDs with blocking delays, suitable for understanding basic timing but less effective for handling concurrent tasks.

- **Timer Interrupt:** Showcased how to use a timer to handle periodic tasks efficiently, enabling the microcontroller to perform other operations while managing timing in the background. This method is particularly useful for tasks requiring precise timing without blocking the main execution.

- **Push Button Interrupt:** Introduced the concept of external interrupts triggered by user input, allowing real-time interaction with the system. This method illustrated how to handle interrupts for external events, such as button presses, and provided a way to interact with the system dynamically.

These methods provide a comprehensive understanding of managing timing and interrupts in embedded systems, helping you to appreciate the trade-offs and applications of different techniques in real-time system design.