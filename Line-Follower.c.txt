#include <Motor_Shield.h> // Include motor shield library for motor-related functions

#define IR_LEFT 2        // Left IR sensor input pin
#define IR_RIGHT 5       // Right IR sensor input pin
#define trigPin A0
#define echoPin A1

DCMotor Rmotor(1);    // Motor 1 instance
DCMotor Lmotor(2);    // Motor 2 instance

// Speed settings
const int BASE_SPEED = 120;  // Slower base speed for forward motion
const int TURN_SPEED = 100;  // Slightly higher speed for turning
const int OBSTACLE_DISTANCE = 12; // Distance in cm to detect obstacles

void setup() {
  Serial.begin(9600);
  pinMode(IR_LEFT, INPUT);   // Left IR sensor as input
  pinMode(IR_RIGHT, INPUT);  // Right IR sensor as input
  pinMode(trigPin, OUTPUT);  // Ultrasonic trigger pin
  pinMode(echoPin, INPUT);   // Ultrasonic echo pin
}

void loop() {
  // Measure distance using ultrasonic sensor
  int distance = getUltrasonicDistance();

  // Check for obstacles
  if (distance <= OBSTACLE_DISTANCE) {
    avoidObstacle();
  } else {
    followLine();
  }
}

// Function to get distance from ultrasonic sensor
int getUltrasonicDistance() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);
  
  long duration = pulseIn(echoPin, HIGH);
  return duration * 0.0343 / 2; // Convert to cm
}

// Function to handle obstacle avoidance
void avoidObstacle() {
  Serial.println("Obstacle detected. Avoiding...");

  // Step 1: Turn right to avoid the obstacle
  Lmotor.run(BACKWARD);
  Rmotor.run(FORWARD);
  Lmotor.setSpeed(120);
  Rmotor.setSpeed(120);
  delay(250); // Turn duration - adjust as needed

  // Step 2: Move forward to clear the obstacle
  Lmotor.run(FORWARD);
  Rmotor.run(FORWARD);
  Lmotor.setSpeed(100);
  Rmotor.setSpeed(100);
  delay(1600); // Move forward - adjust as needed

  // Step 3: Turn left to re-align with the line
  Lmotor.run(FORWARD);
  Rmotor.run(BACKWARD);
  Lmotor.setSpeed(120);
  Rmotor.setSpeed(120);
  delay(600); // Turn duration - adjust as needed

  // Step 4: Move forward slowly until the line is detected
  while (true) {
    int leftSensorValue = digitalRead(IR_LEFT);
    int rightSensorValue = digitalRead(IR_RIGHT);

    if (leftSensorValue == 0 || rightSensorValue == 0) {
      // Stop moving forward when the line is detected
      Serial.println("Line detected. Returning to line-following mode.");
      break;
    }

    // Keep moving forward
    Lmotor.run(FORWARD);
    Rmotor.run(FORWARD);
    Lmotor.setSpeed(80);  // Slow forward speed for better detection
    Rmotor.setSpeed(80);
  }

  // Stop motors briefly before resuming line following
  Lmotor.setSpeed(0);
  Rmotor.setSpeed(0);
  delay(100);
}

// Function to follow the line
void followLine() {
  // Read sensor values
  int leftSensorValue = digitalRead(IR_LEFT);
  int rightSensorValue = digitalRead(IR_RIGHT);

  if (leftSensorValue == 1 && rightSensorValue == 0) {
    // Turn left
    Lmotor.run(BACKWARD);  // Slow left motor
    Rmotor.run(FORWARD);   // Speed up right motor
    Lmotor.setSpeed(BASE_SPEED - 20);
    Rmotor.setSpeed(BASE_SPEED + 20);
    Serial.println("Turning Left");
  } 
  else if (leftSensorValue == 0 && rightSensorValue == 1) {
    // Turn right
    Lmotor.run(FORWARD);   // Speed up left motor
    Rmotor.run(BACKWARD);  // Slow right motor
    Lmotor.setSpeed(BASE_SPEED + 20);
    Rmotor.setSpeed(BASE_SPEED - 20);
    Serial.println("Turning Right");
  } 
  else if (leftSensorValue == 0 && rightSensorValue == 0) {
    // Move forward
    Lmotor.run(FORWARD);
    Rmotor.run(FORWARD);
    Lmotor.setSpeed(BASE_SPEED);
    Rmotor.setSpeed(BASE_SPEED);
    Serial.println("Moving Forward");
  } 
  else if (leftSensorValue == 1 && rightSensorValue == 1) {
    // Stop
    delay(200);
    Lmotor.setSpeed(0);
    Rmotor.setSpeed(0);
    Serial.println("Stopping");
  }
}
