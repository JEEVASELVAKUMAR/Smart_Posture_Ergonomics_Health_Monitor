#include <Wire.h>
#include <Adafruit_SSD1306.h>
#include <VL53L0X.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);
VL53L0X sensor;

const int buzzerPin = 8;

// Settings
const float STABLE_THRESHOLD = 3.0;       // cm difference allowed
const unsigned long ALERT_TIME = 30000;   // 30 seconds = 30000 ms
const unsigned long BUZZER_DURATION = 5000; // 5 seconds = 5000 ms

// Variables
float standardDistance = 0;               // fixed posture distance
bool personSeated = false;
unsigned long seatedTime = 0;
bool alertActive = false;
unsigned long buzzerStartTime = 0;
bool buzzerOn = false;

void setup() {
  Serial.begin(115200);
  Wire.begin();

  // OLED setup
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    Serial.println(F("SSD1306 allocation failed"));
    while (1);
  }

  display.clearDisplay();
  display.setTextColor(WHITE);
  display.setTextSize(1);
  display.setCursor(15, 20);
  display.println("SMART POSTURE");
  display.setCursor(20, 35);
  display.println("Initializing...");
  display.display();
  delay(2000);

  // Sensor setup
  sensor.setTimeout(500);
  if (!sensor.init()) {
    Serial.println("Sensor not found!");
    display.clearDisplay();
    display.setCursor(10, 25);
    display.println("Sensor Error!");
    display.display();
    while (1);
  }

  sensor.setMeasurementTimingBudget(200000);
  pinMode(buzzerPin, OUTPUT);
  noTone(buzzerPin);

  display.clearDisplay();
  display.setCursor(20, 25);
  display.println("System Ready!");
  display.display();
  delay(1500);
}

void loop() {
  float currentDistance = sensor.readRangeSingleMillimeters() / 10.0; // convert mm → cm

  if (sensor.timeoutOccurred()) {
    Serial.println("Sensor timeout");
    return;
  }

  // Detect if person just sat down (take first stable reading)
  if (!personSeated) {
    standardDistance = currentDistance;
    personSeated = true;
    seatedTime = millis();
    alertActive = false;
    buzzerOn = false;
    noTone(buzzerPin);
  }

  // Check posture movement
  float diff = abs(currentDistance - standardDistance);

  if (diff > STABLE_THRESHOLD) {
    // Person moved → reset everything
    personSeated = false;
    alertActive = false;
    buzzerOn = false;
    noTone(buzzerPin);
  }

  // If seated, calculate duration
  unsigned long duration = millis() - seatedTime;

  // Activate alert after 30 seconds
  if (personSeated && duration >= ALERT_TIME && !alertActive) {
    alertActive = true;
    buzzerOn = true;
    buzzerStartTime = millis();
    tone(buzzerPin, 1000); // Start buzzer
  }

  // Stop buzzer after 5 seconds
  if (buzzerOn && millis() - buzzerStartTime >= BUZZER_DURATION) {
    noTone(buzzerPin);
    buzzerOn = false;
  }

  // --- OLED Display ---
  display.clearDisplay();
  display.setTextColor(WHITE);
  display.setTextSize(1);
  display.setCursor(0, 0);
  display.println("Sitting Distance:");

  display.setTextSize(2);
  display.setCursor(25, 18);
  display.print(standardDistance, 1);
  display.println(" cm");

  display.setTextSize(1);
  display.setCursor(0, 50);

  if (alertActive && buzzerOn) {
    display.println("MOVE! Alert Active!");
  } else if (alertActive && !buzzerOn) {
    display.println("MOVE ALERT DONE!");
  } else if (personSeated) {
    display.print("Still: ");
    display.print(duration / 1000);
    display.println("s");
  } else {
    display.println("Waiting for position...");
  }

  display.display();

  // Debug info
  Serial.print("Standard: ");
  Serial.print(standardDistance, 1);
  Serial.print(" cm | Diff: ");
  Serial.print(diff, 1);
  Serial.print(" cm | Time: ");
  Serial.print(duration / 1000);
  Serial.print(" s | Alert: ");
  Serial.print(alertActive ? "ON" : "OFF");
  Serial.print(" | Buzzer: ");
  Serial.println(buzzerOn ? "ON" : "OFF");

  delay(500);
}
