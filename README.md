
# 🤖 STM32 AI Handwritten Digit Recognition ✍️



> A real-time handwritten digit recognizer running entirely on an STM32 microcontroller. This project leverages ST's X-CUBE-AI to perform on-device inference of a neural network, classifying digits drawn on an LCD touchscreen.



---
![alt text](https://i.imgur.com/0YPdsCz.png "Hand written digits recognition on STM32F4")
## ✨ Key Features

-   ✍️ **Interactive Drawing:** A 240x240 pixel canvas to draw digits using the touchscreen.
-   🤖 **On-Device AI Inference:** Uses a pre-trained Convolutional Neural Network (CNN) to classify digits without needing a cloud connection.
-   📊 **Real-Time Results:** Instantly displays the top two predictions with their confidence scores.
-   🎨 **Simple GUI:** A clean interface with a large drawing area, a "CLEAR" button, and a dedicated results panel.
-   🔬 **Serial Debug Output:** `printf` is redirected to UART for easy debugging and monitoring.

---

## ⚙️ How It Works

The application flow is straightforward: **Draw**, **Infer**, and **Display**.

### 1. Capturing the Drawing

The main loop continuously polls the touchscreen. When a touch is detected inside the drawing area, its coordinates are scaled down from the 240x240 display resolution to a 28x28 array (`in_data`), which matches the neural network's input size. To ensure the lines are thick enough for the AI model, a small 3x3 block of pixels in the `in_data` array is activated for each touch point.

```c
// In the main `while(1)` loop
BSP_TS_GetState(&screen_state);

if(screen_state.TouchDetected) {
    // Check if touch is within the drawing boundaries
    if((screen_state.X > DRAW_IMGAE_X1 && screen_state.X < DRAW_IMGAE_X2) &&
       (screen_state.Y > DRAW_IMGAE_Y1 && screen_state.Y < DRAW_IMGAE_Y2 )) {

        // Scale coordinates from 240x240 down to 28x28
        int x = screen_state.Y * ((float)28 / 240);
        int y = screen_state.X * ((float)28 / 240);

        // Set pixel values in the 28x28 input array to create a thick line
        in_data[x][y] = 0.99;
        in_data[x+1][y] = 0.99;
        in_data[x-1][y] = 0.99;
        // ...and so on for a 3x3 block

        // Draw a circle on the screen for visual feedback
        BSP_LCD_FillCircle(screen_state.X, screen_state.Y, 5);
    }
}
```

### 2. Triggering Inference

Inference is not continuous. It's triggered manually by pressing the blue **User Button** (`BUTTON_KEY`) on the board. This gives the user time to finish drawing. When pressed, the 28x28 `in_data` array is passed to the X-CUBE-AI processing function.

```c
// In the main `while(1)` loop, after the touch handling
if (BSP_PB_GetState(BUTTON_KEY)) {
    // Pass the input data to the AI model for processing.
    // The model populates 'out_data' with prediction probabilities.
    MX_X_CUBE_AI_Process(in_data, out_data, 1);

    // ... code to process results ...
}
```

### 3. Processing and Displaying Results

After the model runs, the `out_data` array contains 10 floating-point values, each representing the model's confidence for a digit (0-9). The code iterates through this array to find the two highest scores and their corresponding digits. These are then formatted and displayed on the LCD.

```c
// After MX_X_CUBE_AI_Process() is called
for(int i = 0; i < NUM_CLASSES; ++i) {
    // Find the highest probability
    if(first_guess.prob < out_data[i]) {
        // Shift the previous best to second place
        second_guess.id = first_guess.id;
        second_guess.prob = first_guess.prob;
        // Store the new best guess
        first_guess.prob = out_data[i];
        first_guess.id = i;
    }
    // Find the second highest probability
    else if(second_guess.prob < out_data[i]) {
        second_guess.id = i;
        second_guess.prob = out_data[i];
    }
}

// Format the results into strings
sprintf(first_guess_str, "%d (%.4f)", first_guess.id, first_guess.prob);
sprintf(second_guess_str, "%d (%.4f)", second_guess.id, second_guess.prob);

// Display the strings on the LCD
BSP_LCD_DisplayStringAt(100, 250, (uint8_t*) first_guess_str, LEFT_MODE);
BSP_LCD_DisplayStringAt(100, 270, (uint8_t*) second_guess_str, LEFT_MODE);

// Reset variables for the next run
Reset_Pred(&in_data, &first_guess, &second_guess);
```

---

## 🛠️ Hardware & Software Stack

| Category  | Item                                                               |
| --------- | ------------------------------------------------------------------ |
| **Board** | STM32F746G-DISCOVERY          |
| **IDE**   | [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html) |
| **Libs**  | STM32Cube HAL, Board Support Package (BSP)                         |
| **AI**    | [X-CUBE-AI](https://www.st.com/en/embedded-software/x-cube-ai.html) |

### Peripherals Used

-   **LTDC & DMA2D:** For driving the LCD and accelerating graphics.
-   **FMC & SDRAM:** The external SDRAM is used as the LCD frame buffer.
-   **I2C:** For the touchscreen controller.
-   **GPIO:** For the User Button.
-   **UART:** For serial debug messages.
-   **CRC:** Required dependency for X-CUBE-AI.

---

## 🚀 Getting Started

1.  **Clone the Repository:**
    ```bash
    git clone https://your-repository-url-here.git
    ```

2.  **Open in STM32CubeIDE:**
    -   Launch STM32CubeIDE.
    -   Go to `File > Open Projects from File System...` and select the cloned project directory.

3.  **Build the Project:**
    -   Right-click the project in the *Project Explorer* and select `Build Project` (or press `Ctrl+B`).

4.  **Flash the Board:**
    -   Connect your STM32 board to your PC via the ST-LINK USB port.
    -   Click the `Run` button (green play icon) to compile and flash the code to the board.

5.  **Start Drawing!**
    -   The UI will appear on the screen.
    -   Draw a digit (0-9) in the large box.
    -   Press the blue **User Button** on the board to run the AI classification.
    -   Press the on-screen **"CLEAR"** button to start over.


 