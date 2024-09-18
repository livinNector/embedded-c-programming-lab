---
number-depth: 2
---
# UART Serial Communication using STM32 HAL

## Objective:
By the end of this exercise, students will:
- Understand the basics of UART communication.
- Configure UART using STM32 HAL in CubeIDE for the STM Nucleo-144 F767ZI board.
- Transmit and receive data over USART3(It will be set to Asynchronous mode).
- Implement simple communication between the STM32 microcontroller and a serial terminal on a PC.

## Materials:
- STM Nucleo-144 F767ZI development board.
- USB to TTL Serial converter (or onboard ST-LINK USB).
- CubeIDE installed on the PC.
- Serial terminal software (e.g., PuTTY, Tera Term, or the serial monitor in CubeIDE).


## CubeMX Configuration:

### 1. Create a New Project:
   - Launch CubeIDE and create a new project.
   - Select the STM32F767ZI microcontroller (or the STM Nucleo-144 F767ZI board).

### 2. Enable USART3 Peripheral:
   - In the **Pinout & Configuration** tab, enable `USART3` and set mode to Asynchronous.
   - Assign the appropriate pins for TX (PD8) and RX (PD9) for USART3.

### 3. UART Configuration:
   - Set Baud Rate to **9600**.
   - Configure **8 data bits**, **No parity**, and **1 stop bit**.
   - Enable both **Transmit** and **Receive**.

### 4. Configure Clock:
   - Ensure the system clock is properly set to the required frequency (e.g., 216 MHz for STM32F767ZI).
   
### 5. Enable NVIC Settings (Optional for Interrupts):
   - If using interrupts, enable **USART3 global interrupt**.

### 6. Generate Code:
   - Click **Project > Generate Code** to create the project with the configured USART settings.


## Explanation of Key Functions:
Explanations of these functions can be found in the [UM1850](https://www.st.com/resource/en/user_manual/dm00154093-description-of-stm32f1-hal-and-lowlayer-drivers-stmicroelectronics.pdf#38%20HAL%20UART%20Generic%20Driver)

1. **HAL_UART_Transmit:**
   - This function sends data through the UART peripheral.
   - Syntax:
     ```c
     HAL_UART_Transmit(&huart3, data, length, timeout);
     ```
   - Example:
     ```c
     uint8_t msg[] = "Hello, UART!";
     HAL_UART_Transmit(&huart3, msg, sizeof(msg)-1, HAL_MAX_DELAY);  // Transmit the message
     ```

2. **HAL_UART_Receive:**
   - This function receives data through the UART peripheral.
   - Syntax:
     ```c
     HAL_UART_Receive(&huart3, buffer, length, timeout);
     ```
   - Example:
     ```c
     uint8_t rxBuffer[10];
     HAL_UART_Receive(&huart3, rxBuffer, 10, HAL_MAX_DELAY);  // Receive data
     ```

3. **HAL_UART_Receive_IT:**
   - This function enables interrupt-based data reception.
   - It allows the microcontroller to handle data asynchronously.
   - Syntax:
     ```c
     HAL_UART_Receive_IT(&huart3, buffer, length);
     ```

4. **HAL_TIM_Base_Start_IT:**
   - Used to start a timer in interrupt mode, which can be useful for periodic actions like transmitting data every second.
   - Example:
     ```c
     HAL_TIM_Base_Start_IT(&htim3);  // Start timer 3 in interrupt mode
     ```

5. **HAL_TIM_PeriodElapsedCallback:**
   - This function is called when a timer interrupt occurs.
   - It can be used to perform tasks at specific intervals, such as sending a string over UART.


## Tasks and Sample IO Behavior:

### Task 1: Simple UART Communication
- Transmit a string ("Hello, UART!") from the STM32 to the PC serial terminal via USART3.

#### Code Example:
```c
uint8_t msg[] = "Hello, UART!";
HAL_UART_Transmit(&huart3, msg, sizeof(msg)-1, HAL_MAX_DELAY);  // Transmit the message
```

#### Sample IO:
- **Output**: The string "Hello, UART!" appears in the serial terminal on the PC.
  

### Task 2: Implement a Loopback Test
- Modify the code so that the STM32 receives data from the serial terminal and echoes it back.

#### Code Example:
```c
uint8_t rxBuffer[10];  // Buffer to store received data

while (1) {
    // Receive data from the serial terminal
    HAL_UART_Receive(&huart3, rxBuffer, sizeof(rxBuffer), HAL_MAX_DELAY);
    
    // Echo the received data back to the terminal
    HAL_UART_Transmit(&huart3, rxBuffer, sizeof(rxBuffer), HAL_MAX_DELAY);
}
```

#### Sample IO:
- **Input**: If you type "STM32" in the serial terminal.
- **Output**: "STM32" is echoed back to the terminal.


### Task 3: Send a String Every 1 Second Using Timer Interrupt
- Use a timer interrupt to send a string ("STM32 Timer Test") every 1 second.

#### Code Example:
```c
// Start timer in interrupt mode
HAL_TIM_Base_Start_IT(&htim3);

// Timer interrupt callback function
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim) {
    if (htim->Instance == TIM3) {
        // Send message every second
        uint8_t msg[] = "STM32 Timer Test\n";
        HAL_UART_Transmit(&huart3, msg, sizeof(msg)-1, HAL_MAX_DELAY);
    }
}
```

#### Sample IO:
- **Output**: The string "STM32 Timer Test" is sent to the serial terminal every second.


### Task 4: Interrupt-Based UART Communication
- Configure UART communication using interrupts to receive and echo data asynchronously.

#### Code Example:
```c
uint8_t rxBuffer[1];

// Start interrupt-based reception
HAL_UART_Receive_IT(&huart3, rxBuffer, 1);

// Interrupt callback function
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart) {
    if (huart->Instance == USART3) {
        // Echo received data
        HAL_UART_Transmit(&huart3, rxBuffer, 1, HAL_MAX_DELAY);

        // Restart reception
        HAL_UART_Receive_IT(&huart3, rxBuffer, 1);
    }
}
```

#### Sample IO:
- **Input**: Type "STM32" in the serial terminal.
- **Output**: Each character is echoed back immediately, as data is received and transmitted using interrupts.


## Conclusion:

In this exercise, you explored USART serial communication on the STM Nucleo-144 F767ZI board using STM32 HAL in CubeIDE. We covered two primary communication methods: **polling** and **interrupts**.

- **Polling** was demonstrated using functions like `HAL_UART_Transmit` and `HAL_UART_Receive`, where the microcontroller continuously checks for data transmission or reception. While simple to implement, polling can block the CPU, preventing it from executing other tasks while waiting for the communication to complete.
  
- **Interrupts**, on the other hand, were introduced as a more efficient method of handling USART communication. In interrupt-based communication, the CPU can perform other tasks while waiting for data. When data is ready, an interrupt is triggered, and the appropriate callback function handles the data. This method enhances system performance and responsiveness, especially in real-time applications.

By implementing both approaches, you gained a clear understanding of how to manage serial communication effectively in embedded systems, choosing between polling and interrupts depending on system requirements. These concepts are crucial for building efficient, responsive applications in embedded environments.
