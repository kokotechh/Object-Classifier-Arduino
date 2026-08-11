# 🤖📷 Object Classifier + Arduino Buddy

A little CNN that looks through a webcam, figures out what object it's seeing,
and tells an Arduino to react — printing the object's name on an OLED screen
and swinging a servo motor to a matching angle! 🎯✨

## 🧠 How it works

1. 📸 A CNN (trained on photos I took myself!) learns to recognize **5 objects**:
   🍎 apple · 🍌 banana · ☕ cup · 🧃 juice · 📱 phone
2. 🎥 The webcam feed gets classified frame-by-frame in real time with OpenCV
3. ✅ When the model is confident (>80%) *and* the prediction changed, it
   sends the object's name to the Arduino over serial
4. 🖥️ The Arduino shows the name on an OLED display and 🦾 moves a servo to
   a specific angle depending on what was detected

## 🏗️ Model architecture

- 🖼️ Input: 128×128 RGB images
- 🧱 3× (Conv2D + MaxPooling2D) → Flatten → Dropout → Dense
- 🔄 Data augmentation (flip, rotation, zoom, contrast) — since the dataset
  is homemade, this helps it generalize to different lighting/angles
- 📊 Trained on 679 images, tested on 237 images

## 🛠️ Requirements

- 🐍 Python 3.11
- 🔶 TensorFlow / Keras
- 👁️ OpenCV (`opencv-python`)
- 🔌 pyserial
- 🤝 Arduino with OLED display + servo motor

## 🚀 Setup

1. Install the goodies:
   ```bash
   pip install tensorflow opencv-python pyserial
   ```
2. 📁 Update `train_dir` / `test_dir` to point at your dataset
3. 🔌 Update the serial port (`COM7` on Windows, `/dev/ttyUSB0` on Linux/Mac)
   to match your Arduino
4. ▶️ Run the training cells, then fire up the real-time detection cell!

## 📦 Project structure

```
📓 object_classifier.ipynb   — training + real-time inference notebook
🧠 object_classifier.keras   — saved trained model
📝 class_names.txt           — class labels (must match training order)
🔧 arduino/                  — Arduino sketch for OLED + servo control
```

## 🔧 Arduino Sketch

This runs on the Arduino side — it listens for the object name over serial,
moves the servo to a matching angle, and shows the name on the OLED. 🖥️🦾

```cpp
#include <Arduino.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// 1. Includes
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Servo.h>

// 2. Defines
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
#define OLED_RESET -1
#define SCREEN_ADDRESS 0x3C

// 3. Object declarations
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);
Servo myServo;
const int servoPin = 8;

// 4. THEN setup() and loop() below all of the above
void setup() {
  Serial.begin(9600);

  myServo.attach(servoPin);
  myServo.write(0);

  if (!display.begin(SSD1306_SWITCHCAPVCC, SCREEN_ADDRESS)) {
    Serial.println("OLED not found");
    while (true);
  }

  display.clearDisplay();
  display.setTextSize(2);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 20);
  display.println("Ready");
  display.display();
}

void loop() {
  if (Serial.available() > 0) {
    String detected = Serial.readStringUntil('\n');
    detected.trim(); // remove any extra whitespace/newline characters

    int angle = 90; // default (juice / center)

    if (detected == "apple") {
      angle = 0;
    } else if (detected == "banana") {
      angle = 180;
    } else if (detected == "cup") {
      angle = 45;
    } else if (detected == "phone") {
      angle = 135;
    } else if (detected == "juice") {
      angle = 90;
    }

    myServo.write(angle);

    display.clearDisplay();
    display.setTextSize(2);
    display.setTextColor(SSD1306_WHITE);
    display.setCursor(0, 20);
    display.println(detected);
    display.display();

    Serial.print("Moved to: ");
    Serial.println(detected);
  }
}
```

**Wiring notes:**
- 🖥️ OLED (SSD1306) connects via I2C at address `0x3C`
- 🦾 Servo signal pin → Arduino pin `8`
- 🔌 Baud rate must match the Python side: `9600`

## 💡 Notes

- 🎥 Camera index may need a tweak (`cv2.VideoCapture(0)` vs `(1)`) depending
  on your webcam setup
- 🤫 Only pings the Arduino when the prediction *changes* and confidence is
  high, so it's not spamming the serial port every single frame

---
Made with 🧠 + ☕ + a lot of homemade object photos 📷




