#define Relay 13
#define buzzer A0
static const int sensorPin = 10;                    // sensor input pin 
int SensorStatePrevious = LOW;                      // previous state of the sensor

unsigned long minSensorDuration = 3000; // Time we wait before the sensor active as long 
unsigned long minSensorDuration2 = 6000;
unsigned long SensorLongMillis;                // Time in ms when the sensor was active
bool SensorStateLongTime = false;                  // True if it is a long active

const int intervalSensor = 50;                      // Time between two readings sensor state
unsigned long previousSensorMillis;                 // Timestamp of the latest reading

unsigned long SensorOutDuration;                  // Time the sensor is active in ms

unsigned long currentMillis;          // Variable to store the number of milliseconds since the Arduino has started

void setup() {
  Serial.begin(9600);                 // Initialize the serial monitor

  pinMode(sensorPin, INPUT);          // Set sensorPin as input
  pinMode(Relay, OUTPUT);
  pinMode(buzzer, OUTPUT);
  
  // GSM Module setup
  Serial.begin(9600);   // Setting the baud rate of GSM Module  
  delay(100);
}

void readSensorState() {
  if (currentMillis - previousSensorMillis > intervalSensor) {
    int SensorState = digitalRead(sensorPin);

    if (SensorState == LOW && SensorStatePrevious == HIGH && !SensorStateLongTime) {
      SensorLongMillis = currentMillis;
      SensorStatePrevious = LOW;
      Serial.println("Button pressed");
    }

    SensorOutDuration = currentMillis - SensorLongMillis;

    if (SensorState == LOW && !SensorStateLongTime && SensorOutDuration >= minSensorDuration) {
      SensorStateLongTime = true;
      digitalWrite(Relay, HIGH);
      Serial.println("Button long pressed");
      
      // GSM Module message sending
      Serial.println("AT+CMGF=1");    // Sets the GSM Module in Text Mode
      delay(1000);  // Delay of 1000 milliseconds or 1 second
      Serial.println("AT+CMGS=\"+91########\"\r"); // Replace x with mobile number
      delay(1000);
      Serial.println("Eyeblink detected - Check immediately!");  // The SMS text you want to send
      delay(100);
      Serial.println((char)26);  // ASCII code of CTRL+Z
      delay(1000);
    }

    if (SensorState == LOW && SensorStateLongTime && SensorOutDuration >= minSensorDuration2) {
      SensorStateLongTime = true;
      digitalWrite(buzzer, HIGH);
      delay(1000);
      Serial.println("Button long pressed");
    }

    if (SensorState == HIGH && SensorStatePrevious == LOW) {
      SensorStatePrevious = HIGH;
      SensorStateLongTime = false;
      digitalWrite(Relay, LOW);
      digitalWrite(buzzer, LOW);
      Serial.println("Button released");
    }

    previousSensorMillis = currentMillis;
  }
}

void loop() {
  currentMillis = millis();    // Store the current time
  readSensorState();           // Read the sensor state
}
