#include <DHT.h>

#define DHTPIN 4
#define DHTTYPE DHT22
#define HEATER_PIN 2
#define BUZZER_PIN 5

float threshold_low = 20.0;
float threshold_high = 30.0;
float threshold_max = 40.0;

DHT dht(DHTPIN, DHTTYPE);

enum State {
  IDLE,
  HEATING,
  STABILIZING,
  TARGET_REACHED,
  OVERHEAT
};

State currentState = IDLE;

void setup() {
  Serial.begin(9600);       // REQUIRED: Starts Serial Monitor
  dht.begin();
  pinMode(HEATER_PIN, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);
}

void loop() {
  float temp = dht.readTemperature();
  if (isnan(temp)) {
    Serial.println("Failed to read temperature");
    return;
  }

  updateState(temp);
  logStatus(temp);   // Show current temp + state
  delay(1000);       // 1 second delay
}

void updateState(float temp) {
  switch(currentState) {
    case IDLE:
      if (temp < threshold_low)
        currentState = HEATING;
      else if (temp >= threshold_low && temp < threshold_high)
        currentState = STABILIZING;
      else if (temp >= threshold_high && temp <= threshold_max)
        currentState = TARGET_REACHED;
      else if (temp > threshold_max)
        currentState = OVERHEAT;
      break;

    case HEATING:
      digitalWrite(HEATER_PIN, HIGH);
      if (temp >= threshold_low && temp < threshold_high)
        currentState = STABILIZING;
      else if (temp >= threshold_high)
        currentState = TARGET_REACHED;
      else if (temp > threshold_max)
        currentState = OVERHEAT;
      break;

    case STABILIZING:
      digitalWrite(HEATER_PIN, HIGH);
      if (temp >= threshold_high)
        currentState = TARGET_REACHED;
      else if (temp < threshold_low)
        currentState = HEATING;
      break;

    case TARGET_REACHED:
      digitalWrite(HEATER_PIN, LOW);
      if (temp > threshold_max)
        currentState = OVERHEAT;
      else if (temp < threshold_high && temp >= threshold_low)
        currentState = STABILIZING;
      break;

    case OVERHEAT:
      digitalWrite(HEATER_PIN, LOW);
      digitalWrite(BUZZER_PIN, HIGH);
      if (temp <= threshold_max)
        currentState = TARGET_REACHED;
      break;
  }
}

// New function: Serial logging for debugging and demonstration
void logStatus(float temp) {
  Serial.print("Temp: ");
  Serial.print(temp);
  Serial.print(" °C | State: ");

  switch(currentState) {
    case IDLE: Serial.println("IDLE"); break;
    case HEATING: Serial.println("HEATING"); break;
    case STABILIZING: Serial.println("STABILIZING"); break;
    case TARGET_REACHED: Serial.println("TARGET_REACHED"); break;
    case OVERHEAT: Serial.println("OVERHEAT"); break;
  }
}
