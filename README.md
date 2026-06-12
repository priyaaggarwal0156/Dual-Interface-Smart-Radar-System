# Dual-Interface-Smart-Radar-System
This repository consist of code for Dual Interface Smart Radar System Project
Code:-
#include <Servo.h>
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64

#define TRIG 9
#define ECHO 10
#define SERVO_PIN 6
#define BUZZER 3

Servo radarServo;
long duration;
int distance;

// OLED display object
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

void setup() {
  Serial.begin(9600);
  radarServo.attach(SERVO_PIN);

  pinMode(TRIG, OUTPUT);
  pinMode(ECHO, INPUT);
  pinMode(BUZZER, OUTPUT);

  // Initialize OLED
  if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
    while (true);
  }
  display.clearDisplay();
}

void loop() {
  // Sweep 0° → 180°
  for (int angle = 0; angle <= 180; angle++) {
    radarServo.write(angle);
    delay(20);

    distance = getDistance();
    updateOLED(angle, distance);
    buzzerAlert(distance);
    sendData(angle, distance);
  }

  // Sweep 180° → 0°
  for (int angle = 180; angle >= 0; angle--) {
    radarServo.write(angle);
    delay(20);

    distance = getDistance();
    updateOLED(angle, distance);
    buzzerAlert(distance);
    sendData(angle, distance);
  }
}

// Measure distance using HC-SR04
int getDistance() {
  digitalWrite(TRIG, LOW);
  delayMicroseconds(2);

  digitalWrite(TRIG, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG, LOW);

  duration = pulseIn(ECHO, HIGH);
  return duration * 0.034 / 2; // cm
}

// Update OLED display
void updateOLED(int angle, int distance) {
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(WHITE);

  display.setCursor(0, 0);
  display.println("Smart Radar System");

  display.setCursor(0, 20);
  display.print("Angle: ");
  display.println(angle);

  display.setCursor(0, 35);
  display.print("Distance: ");
  display.print(distance);
  display.println(" cm");

  if (distance < 20) {
    display.setCursor(0, 50);
    display.println("⚠ OBJECT ALERT!");
  }

  display.display();
}

// Turn buzzer ON if object < 20cm
void buzzerAlert(int distance) {
  if (distance < 20) {
    digitalWrite(BUZZER, HIGH);
  } else {
    digitalWrite(BUZZER, LOW);
  }
}

// Send data to p5.js
void sendData(int angle, int distance) {
  Serial.print(angle);
  Serial.print(",");
  Serial.print(distance);
  Serial.print("."); // delimiter for p5.js
}
