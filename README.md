/*!
 * @file  coil_winding_controller.ino
 * @brief TB6600 Stepper Motor Driver with manual controls for Tesla coil winding
 * @version  V1.0.0
 * @date  2025-11-27
 */

// Motor driver pins
#define PUL_PIN  8
#define DIR_PIN  9
#define ENA_PIN  10

// Control inputs (all INPUT_PULLUP)
#define POT_PIN       A0
#define DIR_CW        2   // DPDT left position - white wire
#define DIR_CCW       5   // DPDT right position - green wire
#define POWER_SWITCH  3   // Power toggle - yellow wire
#define MOMENTARY_BTN 4   // Momentary start - orange wire

// Optional LED indicators
#define LED_POWER  11
#define LED_RUN    12
#define LED_DIR    13

void setup() {
  pinMode(PUL_PIN, OUTPUT);
  pinMode(DIR_PIN, OUTPUT);
  pinMode(ENA_PIN, OUTPUT);
  
  pinMode(DIR_CW, INPUT_PULLUP);
  pinMode(DIR_CCW, INPUT_PULLUP);
  pinMode(POWER_SWITCH, INPUT_PULLUP);
  pinMode(MOMENTARY_BTN, INPUT_PULLUP);
  
  pinMode(LED_POWER, OUTPUT);
  pinMode(LED_RUN, OUTPUT);
  pinMode(LED_DIR, OUTPUT);
  
  digitalWrite(ENA_PIN, LOW);
  digitalWrite(PUL_PIN, LOW);
  digitalWrite(LED_POWER, HIGH);
}

void loop() {
  // Read inputs - LOW = active due to INPUT_PULLUP
  bool dirCW = (digitalRead(DIR_CW) == LOW);
  bool dirCCW = (digitalRead(DIR_CCW) == LOW);
  bool powerOn = (digitalRead(POWER_SWITCH) == LOW);
  bool momentaryOn = (digitalRead(MOMENTARY_BTN) == LOW);
  
  // Direction selected = not in centre-off
  bool directionSelected = dirCW || dirCCW;
  
  // Motor enabled (holding torque) when direction selected
  // Motor runs when enabled AND (power OR momentary)
  bool motorEnabled = directionSelected;
  bool motorShouldRun = motorEnabled && (powerOn || momentaryOn);
  
  if (motorEnabled) {
    digitalWrite(ENA_PIN, HIGH);
    
    // Set direction based on which switch position
    if (dirCW) {
      digitalWrite(DIR_PIN, LOW);
      digitalWrite(LED_DIR, LOW);
    } else {
      digitalWrite(DIR_PIN, HIGH);
      digitalWrite(LED_DIR, HIGH);
    }
    
    if (motorShouldRun) {
      int potValue = analogRead(POT_PIN);
      int stepDelay = map(potValue, 0, 1023, 500, 10000);
      
      digitalWrite(PUL_PIN, HIGH);
      delayMicroseconds(stepDelay);
      digitalWrite(PUL_PIN, LOW);
      delayMicroseconds(stepDelay);
      
      digitalWrite(LED_RUN, HIGH);
    } else {
      // Holding - enabled but not stepping
      digitalWrite(LED_RUN, LOW);
    }
    
  } else {
    // Centre-off - full disable, motor free to spin
    digitalWrite(ENA_PIN, LOW);
    digitalWrite(LED_RUN, LOW);
  }
}
