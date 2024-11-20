#include <Arduino.h>

// Pin Definitions
#define BTN 27           // Button pin
#define SPK 25           // Speaker pin
#define LED_PIN 13       // LED pin
#define POT_PIN 34       // Potentiometer pin (analog input)

// Constants
const int threshold = 500;      // Distance sensor threshold
const int freq = 5000;          // Speaker frequency
const int pwmChannel = 0;       // PWM channel
const int resolution = 8;       // PWM resolution
const int alarmTimeout = 5000;  // Alarm timeout duration in milliseconds

// Variables
volatile bool buttonPressed = false;  // Interrupt flag for button
volatile bool alarmTimedOut = false;  // Flag for alarm timeout
int state = 0;                        // 0: Unarmed, 1: Armed, 2: Alarming

// Timer variables
hw_timer_t *timer1 = NULL;            // Timer for alarm timeout
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;

// Interrupt Service Routine for Button
void IRAM_ATTR isr() {
  buttonPressed = true;
}

// Timer callback for alarm timeout
void IRAM_ATTR onTimer() {
  portENTER_CRITICAL_ISR(&timerMux);
  alarmTimedOut = true;
  portEXIT_CRITICAL_ISR(&timerMux);
  timerStop(timer1);
}

// Function Prototypes
bool CheckForButtonPress();
void sound_on();
void sound_off();
void led_on();
void led_off();
void startAlarmTimer();
void stopAlarmTimer();

void setup() {
  // Pin Setup
  pinMode(BTN, INPUT_PULLUP);
  pinMode(LED_PIN, OUTPUT);
  attachInterrupt(BTN, isr, FALLING);  // Attach button interrupt

  // PWM Setup
  ledcSetup(pwmChannel, freq, resolution);
  ledcAttachPin(SPK, pwmChannel);

  // Timer Setup
  timer1 = timerBegin(0, 80, true); // 80 prescaler: 1 tick = 1 microsecond
  timerAttachInterrupt(timer1, &onTimer, true);
  timerAlarmWrite(timer1, alarmTimeout * 1000, false); // Alarm for 5 seconds

  // Initial States
  led_off();
  sound_off();
}

void loop() {
  // Read the analog value from the potentiometer
  int distance = analogRead(POT_PIN);

  // State Machine
  switch (state) {
    case 0: // Unarmed State
      if (CheckForButtonPress()) {
        led_on();
        state = 1; // Transition to Armed State
      }
      break;

    case 1: // Armed State
      if (CheckForButtonPress()) {
        led_off();
        state = 0; // Transition to Unarmed State
      } else if (distance < threshold) {
        sound_on();
        startAlarmTimer(); // Start the alarm timeout timer
        state = 2; // Transition to Alarming State
      }
      break;

    case 2: // Alarming State
      if (CheckForButtonPress()) {
        sound_off();
        stopAlarmTimer(); // Stop the alarm timeout timer
        led_off();
        state = 0; // Transition to Unarmed State
      } else if (distance >= threshold) {
        sound_off();
        stopAlarmTimer(); // Stop the alarm timeout timer
        state = 1; // Transition to Armed State
      } else if (alarmTimedOut) {
        sound_off();
        led_off();
        alarmTimedOut = false; // Reset the timeout flag
        state = 0; // Transition to Unarmed State
      }
      break;
  }
}

// Event Checker for Button Press
bool CheckForButtonPress() {
  if (buttonPressed) {
    buttonPressed = false;
    return true;
  }
  return false;
}

// Timer Control Functions
void startAlarmTimer() {
  timerWrite(timer1, 0); // Reset timer count
  timerAlarmEnable(timer1);
}

void stopAlarmTimer() {
  timerAlarmDisable(timer1);
}

// Service Functions
void sound_on() {
  ledcWriteTone(pwmChannel, 1000); // Turn on sound
}

void sound_off() {
  ledcWriteTone(pwmChannel, 0); // Turn off sound
}

void led_on() {
  digitalWrite(LED_PIN, HIGH); // Turn on LED
}

void led_off() {
  digitalWrite(LED_PIN, LOW); // Turn off LED
}
