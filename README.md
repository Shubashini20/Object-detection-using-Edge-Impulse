# Vegetable Object Detection with ESP32-CAM and Edge Impulse

This project uses an AI Thinker ESP32-CAM to identify vegetables in camera frames. An object-detection model is trained with Edge Impulse, exported as an embedded library, and run locally on the ESP32-CAM. When a vegetable is detected, its label and confidence are shown on a small OLED display and the prediction is printed to the serial monitor.

## Project Overview

The camera captures QVGA images (320 × 240). The firmware passes each image to the Edge Impulse model and checks its detection boxes. Detections below 80% confidence are ignored; the first qualifying detection is shown on the display after it remains present for about one second. When no qualifying object is detected, the display shows `NO OBJECT`.

Inference runs on the ESP32-CAM, so the basic detection workflow does not require an internet connection. Detection quality depends on the training images, lighting, camera placement, and the model that is deployed.

## Hardware

- AI Thinker ESP32-CAM board with a camera and PSRAM
- SSD1306 128 × 64 I2C OLED display
- Stable 5 V supply for the ESP32-CAM
- USB-to-serial adapter for programming and serial output
- Jumper wires

### OLED Wiring

| OLED | ESP32-CAM |
| --- | --- |
| SDA | GPIO 14 |
| SCL | GPIO 15 |
| VCC | Suitable supply for the specific OLED module |
| GND | GND |

The firmware expects the OLED at I2C address `0x3C`. Check the display module's voltage requirements before wiring. GPIO 14 and 15 are also commonly used for the microSD interface; leave the SD card interface unused in this setup.

## Train the Model

1. Create an Edge Impulse project and choose an object-detection project configuration.
2. Collect images of each vegetable class you want to recognize. Include varied backgrounds, distances, orientations, lighting, and examples with multiple objects. Include images without target vegetables to help the model handle empty scenes.
3. Upload the images and annotate each vegetable with a bounding box and the correct class label. Keep the labels consistent.
4. Create an impulse using image input and an object-detection learning block. Choose image dimensions and model settings suitable for the ESP32-CAM's memory and processing limits.
5. Generate features, train the model, and review validation results. Test it with images that were not used for training and adjust the dataset or model if needed.
6. In Edge Impulse, export the trained impulse as an Arduino library. Add the exported library to the Arduino environment so the firmware can include `Object_detection_inferencing.h` and the Edge Impulse SDK.

## Build and Run

The firmware is written for the Arduino framework and the AI Thinker ESP32-CAM pin layout. The included source file currently has no `.ino` extension; in Arduino IDE, create a sketch and place the source in its `.ino` file.

1. Install the ESP32 board support package and select the AI Thinker ESP32-CAM board.
2. Install the Adafruit GFX and Adafruit SSD1306 libraries, along with the Arduino library exported by Edge Impulse.
3. Make sure the exported model is configured for camera image input and is an object-detection model. The model header must match the `Object_detection_inferencing.h` include used by the firmware.
4. Connect the OLED as shown above, connect the ESP32-CAM to a suitable USB-to-serial adapter, and select the appropriate upload settings for the board.
5. Upload the sketch and open the serial monitor at 115200 baud. The OLED shows the detected class and confidence; inference timing and bounding-box predictions are printed to serial.

The confidence threshold is currently `0.80` in the firmware. Lowering it can show more detections but may increase false positives; tune it against real test images. The model, generated Edge Impulse library, and training dataset are not included in this repository and must be supplied from your Edge Impulse project.

## Future Integration

This prototype can be extended into a practical produce-monitoring system. For example, detections could be sent over Wi-Fi to an inventory dashboard, logged with timestamps for stock tracking, or connected to a conveyor and actuator for basic sorting. Additional classes and quality-related labels could support produce inspection, while a cloud or mobile application could collect results and provide remote monitoring. These extensions would need their own networking, storage, and hardware-control logic; they are not part of the current firmware.

## Limitations

- The model recognizes only the classes represented in its training data.
- Small memory and compute budgets on the ESP32-CAM constrain model size and inference speed.
- The current firmware displays only the first bounding box above the confidence threshold.
- Reliable results require representative training data and consistent camera conditions.
