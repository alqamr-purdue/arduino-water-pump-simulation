# Automatic Water Pump Simulation (Arduino)

An Arduino project that simulates an automatic water pump system. A light sensor (LDR) stands in for a water-level sensor, and the Arduino reacts to the level with a servo motor, LEDs, a buzzer, and a live readout on an LCD.

Built for **ENGR 128** at Purdue University Fort Wayne (October 2026).


---

## How it works

1. The Arduino reads the LDR on analog pin A0. `analogRead()` returns a number from 0 to 1023, where a higher number means more light.
2. The reading is shown on a 16x2 LCD, refreshed every 500 ms.
3. The program compares the reading to a **threshold of 550**:
   - **Above 550**: the water level is treated as high. The servo moves to 90 degrees, both LEDs turn on, and the buzzer sounds as a warning.
   - **550 or below**: the servo returns to 0 degrees, LED 3 and the buzzer turn off, and LED 1 stays on.

The servo stands in for a pump valve or gate, and the LEDs and buzzer stand in for status indicators on a real system.

## Behavior

| Sensor reading | Servo | LED 1 | LED 3 | Buzzer |
|---|---|---|---|---|
| Above 550 | 90 degrees | On | On | On |
| 550 or below | 0 degrees | On | Off | Off |

## Hardware

| Component | Arduino pin |
|---|---|
| LDR (light sensor) | A0 (analog input) |
| I2C LCD, 16x2, address 0x27 | I2C (SDA/SCL) |
| Servo motor | D10 |
| LED 1 (red) | D13 |
| LED 3 (blue) | D4 |
| Buzzer | D9 |

<!-- Add: board model (Uno, etc.), the resistor values you used for the LEDs, and the fixed resistor in the LDR voltage divider -->

## Code walkthrough

**Libraries.** `Servo.h` controls the servo, and `Wire.h` and `LiquidCrystal_I2C.h` handle the I2C LCD, which needs only two data wires instead of many.

**Setup.** `pinMode(..., OUTPUT)` makes the LED and buzzer pins outputs. `myServo.attach(10)` connects the servo to pin 10. `lcd.init()` and `lcd.backlight()` start the display.

**Loop.** Each pass reads the sensor, updates the LCD, waits half a second, then compares the reading to the threshold and sets every output.

**LCD cleanup.** After the number, the program prints a few spaces. Without them, a reading like `1000` followed by `95` would show as `950`, because the old digits stay on screen.

**Threshold.** The value 550 is stored in one variable, `threshold`, so it can be changed in one place to match different lighting or a different sensor.

## Ideas to improve it

- **Add hysteresis.** A reading hovering near 550 can make the servo and buzzer flip on and off rapidly. Using two thresholds (for example, turn on above 570 and turn off below 530) would stop that.
- **Replace `delay()` with `millis()` timing.** The 500 ms delay blocks the whole program, so the system can't react during that time.
- **Give the servo more time to move.** The 15 ms delay is short for a servo to reach 90 degrees.
- **Use a real water sensor** in place of the LDR to build the actual pump system.

## What I learned

- How to read an analog sensor and turn a raw number into a decision with a threshold.
- How to drive several outputs (servo, LEDs, buzzer, LCD) from one program.
- How I2C simplifies wiring for a display.
- How a small typo in the code or documentation can change what a circuit does, so it's worth checking the sketch against the diagram and truth table.

## Code

`water_pump_simulation.ino`

```cpp
// Automatic Water Pump Simulation
// An LDR simulates the water level. A high reading means the tank is "full".

#include <Servo.h>              // Servo control
#include <Wire.h>               // I2C communication
#include <LiquidCrystal_I2C.h>  // I2C LCD control

Servo myServo;

// 0x27 = I2C address of the LCD, 16 columns, 2 rows
LiquidCrystal_I2C lcd(0x27, 16, 2);

// -------------------- PINS --------------------
const int ldrPin    = A0;  // LDR (water level sensor)
const int ledPin1   = 13;  // LED 1
const int ledPin3   = 4;   // LED 3
const int BuzzerPin = 9;   // Buzzer

// Reading above this value means the water level is high
const int threshold = 550;

// -------------------- SETUP --------------------
void setup() {
  pinMode(ledPin1, OUTPUT);
  pinMode(ledPin3, OUTPUT);
  pinMode(BuzzerPin, OUTPUT);

  myServo.attach(10);       // Servo on pin 10

  lcd.init();
  lcd.backlight();

  Serial.begin(9600);       // Optional: lets us print to the Serial Monitor
}

// -------------------- MAIN LOOP --------------------
void loop() {
  // analogRead() returns 0 to 1023
  int ldrValue = analogRead(ldrPin);

  // Show the reading on the LCD
  lcd.setCursor(0, 0);
  lcd.print("Water Level: ");
  lcd.setCursor(0, 1);
  lcd.print(ldrValue);
  lcd.print("    ");         // Spaces clear leftover digits from older readings

  delay(500);

  if (ldrValue > threshold) {
    // High water level
    myServo.write(90);
    delay(15);              // Short pause for the servo to respond
    digitalWrite(ledPin3, HIGH);
    digitalWrite(ledPin1, HIGH);
    digitalWrite(BuzzerPin, HIGH);   // Warning buzzer
  } else {
    // Reading is 550 or lower
    digitalWrite(ledPin1, HIGH);     // LED 1 stays on
    digitalWrite(ledPin3, LOW);
    digitalWrite(BuzzerPin, LOW);
    myServo.write(0);                // Servo returns to 0 degrees
  }
}
```

---

*ENGR 128, Purdue University Fort Wayne, October 2026*
