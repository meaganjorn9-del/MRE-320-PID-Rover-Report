# MRE-320-PID-Rover-Report
# Code Presentation
// -------- Motor Pins --------
// Motor A
const int ENA = 5;
const int IN1 = 22;
const int IN2 = 23;

// Motor B
const int ENB = 6;
const int IN3 = 24;
const int IN4 = 25;

// -------- Ultrasonic Pins --------
const int TRIG = 30;
const int ECHO = 31;

// -------- Adjustable Settings --------
int speedVal = 255;     // motor speed (0–255)
int turnTime = 5000;     // adjust for ~180° turn
int stopTime = 3000;     // pause before turning

// -------- Setup --------
void setup() {
  pinMode(ENA, OUTPUT);
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);

  pinMode(ENB, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);

  pinMode(TRIG, OUTPUT);
  pinMode(ECHO, INPUT);

  Serial.begin(9600);
}

// -------- Main Loop --------
void loop() {
  int distance = getDistance();

  Serial.print("Distance: ");
  Serial.println(distance);

  if (distance <= 10 && distance > 0) {
    avoidObstacle();
  } else {
    moveForward();
  }

  delay(100);
}

// -------- Ultrasonic Function --------
int getDistance() {
  digitalWrite(TRIG, LOW);
  delayMicroseconds(2);

  digitalWrite(TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG, LOW);

  long duration = pulseIn(ECHO, HIGH);
  int distance = duration * 0.034 / 2;

  return distance;
}

// -------- Movement Functions --------
void moveForward() {
  motorAForward(speedVal);
  motorBForward(speedVal);
}

void turnInPlace() {
  // Spin: one forward, one reverse
  motorAForward(speedVal);
  motorBReverse(speedVal);
}

void stopMotors() {
  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
  analogWrite(ENA, 0);
  analogWrite(ENB, 0);
}

// -------- Motor Control --------
void motorAForward(int speedVal) {
  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);
  analogWrite(ENA, speedVal);
}

void motorBForward(int speedVal) {
  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
  analogWrite(ENB, speedVal);
}

void motorBReverse(int speedVal) {
  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
  analogWrite(ENB, speedVal);
}

// -------- Obstacle Avoidance --------
void avoidObstacle() {
  // 1. Stop
  stopMotors();
  delay(stopTime);

  // 2. Turn
  turnInPlace();
  delay(turnTime);

  // 3. Stop briefly before moving forward again
  stopMotors();
  delay(200);
}

# How It Works

1) The rover uses two DC motors, and each motor has two direction pins and one PWM-enabled speed pin that helps the motors control the speed and direction of each motor. The motorForward, motorReverse, and turnInPlace functions controls the forward, turning, reverse movement of each motor.
   
2) The ultrasonic sensor provides the rover an environmental input with the detDistance function sending trigger pulses and measuring the time of the echo return, converting time into distance.
   
3) The main loop implements a two-state behavior that controls when the motors should stop, turn in place, or resume forward motion. This helps the rover avoids obstacles. The set distance the rover should stop is 30 centimeters.
   
4) The system uses fixed delays to ensure consistent behavior between sensor update rates and stopping rates. The delays inside the avoidObstacle function determines how long it takes for the motors to stop once it senses an obstacle and how far it should turn in place.
