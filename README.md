# Heartbeat-monitoring-Sensor
This Arduino code uses a Pulse Sensor and I2C LCD display to monitor and show heart rate. It reads the pulse sensor data, calculates the BPM, and displays it along with a custom heart icon on the LCD. The system also sends the raw signal and BPM values to the Serial Monitor for debugging.
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <PulseSensorPlayground.h>

// Pins and objects
const int pulsePin = A1;
PulseSensorPlayground pulseSensor;
LiquidCrystal_I2C lcd(0x27, 16, 2);  // Change address if needed

// Heart icon (custom character)
byte heart[8] = {
  0b00000,
  0b01010,
  0b11111,
  0b11111,
  0b11111,
  0b01110,
  0b00100,
  0b00000
};

void setup() {
  Serial.begin(9600);

  // Set up pulse sensor
  pulseSensor.analogInput(pulsePin);
  pulseSensor.setThreshold(300);

  // Set up LCD
  lcd.begin(16, 2);
  lcd.createChar(0, heart);  // Load heart icon into memory
  lcd.backlight();

  // Welcome screen
  lcd.setCursor(0, 0);
  lcd.print("  Heart Monitor  ");
  lcd.setCursor(0, 1);
  lcd.print("Initializing...");
  delay(2000);
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Heart Rate:");

  // Start pulse sensor
  if (pulseSensor.begin()) {
    Serial.println("PulseSensor ready");
  } else {
    Serial.println("PulseSensor not found");
  }
}

void loop() {
  int signal = analogRead(pulsePin);
  Serial.print("Signal: ");
  Serial.println(signal);

  int bpm = pulseSensor.getBeatsPerMinute();

  if (pulseSensor.sawStartOfBeat()) {
    Serial.print("BPM: ");
    Serial.println(bpm);

    lcd.setCursor(0, 1);
    lcd.write(byte(0));      // Heart icon
    lcd.print(" BPM: ");
    lcd.print(bpm);
    lcd.print("     ");       // Padding to clear previous digits
  }

  delay(1000);
}
