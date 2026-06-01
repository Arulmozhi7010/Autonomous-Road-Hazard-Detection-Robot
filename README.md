#include <SoftwareSerial.h>

// ESP32-CAM Serial
SoftwareSerial ESPCam(11, 12); // RX, TX

// Bluetooth HC-05
SoftwareSerial BT(6, 7); // RX, TX

// Ultrasonic Pins
#define trigPin 9
#define echoPin 10

// Vibration Sensor
#define vibPin 2

// Motor Driver Pins
#define IN1 3
#define IN2 4
#define IN3 5
#define IN4 8

long duration;
int distance;

char command;

void setup() {

  Serial.begin(9600);

  ESPCam.begin(9600);

  BT.begin(9600);

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  pinMode(vibPin, INPUT);

  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(IN3, OUTPUT);
  pinMode(IN4, OUTPUT);

  stopRobot();

  Serial.println("Robot Started");
}

void loop() {

  // ---------- Ultrasonic Distance ----------
  
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);

  digitalWrite(trigPin, LOW);

  duration = pulseIn(echoPin, HIGH);

  distance = duration * 0.034 / 2;

  Serial.print("Distance: ");
  Serial.println(distance);

  // Send distance to ESP32-CAM
  ESPCam.print("DIST:");
  ESPCam.println(distance);

  // ---------- Hole Detection ----------
  
  int vibration = digitalRead(vibPin);

  if (vibration == HIGH) {

    Serial.println("Hole Detected");

    ESPCam.println("HOLE DETECTED");

    stopRobot();
    delay(500);

    backward();
    delay(1000);

    rightTurn();
    delay(700);

    stopRobot();
  }

  // ---------- Obstacle Detection ----------
  
  else if (distance > 0 && distance < 20) {

    Serial.println("Obstacle Detected");

    ESPCam.println("OBSTACLE DETECTED");

    stopRobot();
    delay(500);

    backward();
    delay(800);

    leftTurn();
    delay(700);

    stopRobot();
  }

  // ---------- Normal Forward ----------
  
  else {

    forward();
  }

  // ---------- Bluetooth Manual Control ----------
  
  if (BT.available()) {

    command = BT.read();

    Serial.print("Bluetooth Command: ");
    Serial.println(command);

    if (command == 'F') {
      forward();
    }

    else if (command == 'B') {
      backward();
    }

    else if (command == 'L') {
      leftTurn();
    }

    else if (command == 'R') {
      rightTurn();
    }

    else if (command == 'S') {
      stopRobot();
    }
  }

  delay(100);
}

// ================= MOTOR FUNCTIONS =================

void forward() {

  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);

  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

void backward() {

  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);

  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
}

void leftTurn() {

  digitalWrite(IN1, LOW);
  digitalWrite(IN2, HIGH);

  digitalWrite(IN3, HIGH);
  digitalWrite(IN4, LOW);
}

void rightTurn() {

  digitalWrite(IN1, HIGH);
  digitalWrite(IN2, LOW);

  digitalWrite(IN3, LOW);
  digitalWrite(IN4, HIGH);
}

void stopRobot() {

  digitalWrite(IN1, LOW);
  digitalWrite(IN2, LOW);

  digitalWrite(IN3, LOW);
  digitalWrite(IN4, LOW);
}
