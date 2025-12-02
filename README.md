# Dual-sort-Fill-Sense

**Repository Title:** Dual Sort Fill Sense – Smart Wet & Dry Waste Segregation Bin with Fill‑Level Sensing  

***

## 1. Project overview

Dual Sort Fill Sense is a smart dustbin that **automatically separates wet and dry waste** and also **measures how full each bin is** using sensors and a microcontroller. Incoming waste is classified as wet or dry by a moisture sensor, diverted into the correct compartment by a servo‑controlled flap, and the fill level is monitored by an ultrasonic sensor so that alerts can be raised when the bin is nearly full.

***

## 2. Features

- Automatic segregation of **wet vs dry** waste with no manual sorting.
- Real‑time **fill‑level measurement** using ultrasonic sensing.  
- Visual or audible **full‑bin alerts** (LED/buzzer), easily extendable to IoT notifications.  
- Modular design: can be built on Arduino, NodeMCU, or ESP32.  

***

## 3. Hardware components

- 1 × Microcontroller board (Arduino Uno / Nano / NodeMCU / ESP32)  
- 1 × Moisture sensor module (soil‑moisture type) for wet/dry detection.
- 1–2 × Ultrasonic sensor HC‑SR04 (one for object detection at lid, one for fill level; or one reused by time‑sharing)
- 1–2 × Servo motors (to rotate diverting flap / lid)
- 1 × Dustbin with **two inner compartments** (wet and dry)  
- 1 × Buzzer (optional)  
- 1–2 × LEDs with resistors (status / full alert)  
- Power supply (5 V for logic, separate 5–6 V source for servos if needed)  
- Jumper wires, small perfboard/breadboard, mounting hardware and hinges  

***

## 4. System design

### Block diagram (text)

Waste inlet → Moisture sensor → Microcontroller → Servo flap →  
Wet compartment / Dry compartment  

Ultrasonic sensor (top) → Microcontroller → Fill‑percentage calculation → LED/Buzzer/IoT alert

### Key signal connections

- Moisture sensor signal → analog pin `A0`  
- Ultrasonic `TRIG`/`ECHO` → digital pins `D2`, `D3`  
- Servo signal(s) → PWM pin(s) `D5` (and `D6` if two servos)  
- Buzzer → `D8`, LED → `D9` (example mapping)  
- All sensor and servo grounds tied to controller GND; 5 V shared (or separate regulated rail for servos).  

***

## 5. Working principle

1. **Detection & classification**  
   - When an object comes near the inlet, the top ultrasonic sensor detects it (distance below threshold).  
   - The waste briefly contacts or passes over the **moisture sensor**.  
   - If the moisture reading is above a set value, the waste is treated as **wet**; otherwise **dry**.

2. **Mechanical sorting**  
   - For wet waste, the microcontroller rotates the servo flap to direct trash into the **wet compartment**.  
   - For dry waste, the flap rotates to the opposite side so that trash falls into the **dry compartment**.  

3. **Fill‑level sensing**  
   - An ultrasonic sensor facing downwards into the bin measures the distance from the sensor to the trash surface.  
   - The controller converts this distance into **percentage fill** using bin height calibration; when level exceeds a threshold (for example 80%), it turns on an LED or buzzer and can optionally send an IoT alert via Wi‑Fi platform.

4. **User feedback**  
   - LEDs can show which side is active (wet/dry) and whether the bin is “OK” or “FULL”.  
   - If IoT is added later, a dashboard can show fill level and counts of wet vs dry disposals.

***

## 6. Suggested repository structure

- `/hardware/`  
  - `dual_sort_fill_sense_schematic.png`  
  - `mechanical_layout.png` (bin + flap + compartment sketch)  
- `/firmware/`  
  - `dual_sort_fill_sense.ino` (Arduino/ESP code)  
- `/docs/`  
  - `README.md` (this document)  
  - `calibration.md` (how to set moisture thresholds and bin height)  
- `/media/`  
  - Photos of prototype  
  - Short demo video link  

***

## 7. How to run (short)

1. Assemble hardware as per schematic and ensure servos move freely.  
2. Flash `dual_sort_fill_sense.ino` with correct pin numbers and thresholds.  
3. Power the system; test with dry paper, then with wet tissue to tune the moisture threshold.  
4. Drop multiple items and observe automatic routing and fill‑level indication.  


## CODE:
#include <ESP32Servo.h>  // Make sure this library is added in Wokwi

// Ultrasonic pins
#define TRIG_WET 5
#define ECHO_WET 18
#define TRIG_DRY 19
#define ECHO_DRY 21

// LED pins
#define LED_WET_GREEN 12
#define LED_WET_RED 14
#define LED_DRY_GREEN 27
#define LED_DRY_RED 26

// Moisture sensor + Servo
#define MOISTURE_PIN 34
#define SERVO_PIN 25

Servo flap;
int moistureValue;
int threshold = 500;  // Adjust based on your sensor
int distanceWet, distanceDry;
int fullLevel = 5;    // 5 cm threshold for full bin

// Function to get distance from ultrasonic sensor
int getDistance(int trigPin, int echoPin) {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  long duration = pulseIn(echoPin, HIGH);
  int distance = duration * 0.034 / 2; // cm
  return distance;
}

void setup() {
  Serial.begin(115200);

  pinMode(TRIG_WET, OUTPUT);
  pinMode(ECHO_WET, INPUT);
  pinMode(TRIG_DRY, OUTPUT);
  pinMode(ECHO_DRY, INPUT);

  pinMode(LED_WET_GREEN, OUTPUT);
  pinMode(LED_WET_RED, OUTPUT);
  pinMode(LED_DRY_GREEN, OUTPUT);
  pinMode(LED_DRY_RED, OUTPUT);

  flap.attach(SERVO_PIN);
  flap.write(45);  // Neutral position
}

void loop() {
  // --- Waste type detection ---
  moistureValue = analogRead(MOISTURE_PIN);
  Serial.print("Moisture Value: ");
  Serial.println(moistureValue);

  if (moistureValue > threshold) {
    Serial.println("Wet waste detected → directing LEFT to WET bin");
    flap.write(0);   // Left for wet waste
    delay(1500);     // Simulate waste drop
    flap.write(45);  // Return to neutral
  } else {
    Serial.println("Dry waste detected → directing RIGHT to DRY bin");
    flap.write(90);  // Right for dry waste
    delay(1500);
    flap.write(45);  // Return to neutral
  }

  // --- Bin level detection ---
  distanceWet = getDistance(TRIG_WET, ECHO_WET);
  distanceDry = getDistance(TRIG_DRY, ECHO_DRY);

  Serial.print("Wet Bin: ");
  Serial.print(distanceWet);
  Serial.print(" cm | Dry Bin: ");
  Serial.println(distanceDry);

  // Wet bin LEDs
  if (distanceWet < fullLevel) {
    digitalWrite(LED_WET_RED, HIGH);
    digitalWrite(LED_WET_GREEN, LOW);
  } else {
    digitalWrite(LED_WET_RED, LOW);
    digitalWrite(LED_WET_GREEN, HIGH);
  }

  // Dry bin LEDs
  if (distanceDry < fullLevel) {
    digitalWrite(LED_DRY_RED, HIGH);
    digitalWrite(LED_DRY_GREEN, LOW);
  } else {
    digitalWrite(LED_DRY_RED, LOW);
    digitalWrite(LED_DRY_GREEN, HIGH);
  }

  delay(2000);
}
