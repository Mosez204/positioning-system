
#include <Stepper.h>
#include <LiquidCrystal_74HC595.h>

#define STEPS_PER_REV 2048
#define SCAN_STEPS 1593
#define THRESHOLD 20

// ===== 74HC595 LCD PINS =====


#define DS 2
#define SHCP 3
#define STCP 4
#define RS 15
#define E 1
#define D4 14
#define D5 13
#define D6 12
#define D7 11

LiquidCrystal_74HC595 lcd(DS, SHCP, STCP, RS, E, D4, D5, D6, D7);


// ===== ULTRASONIC =====
#define trigPin 13
#define echoPin 12

// ===== MOTORS =====
Stepper motor1(STEPS_PER_REV, 8, 10, 9, 11);
Stepper motor2(STEPS_PER_REV, 5, 7, 6, A0);

bool obstacleFound = false;

void setup() {

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);

  motor1.setSpeed(10);
  motor2.setSpeed(10);

  lcd.begin(16, 2);
  lcd.print("System Ready");
  delay(2000);
}

long getDistance() {

  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  long duration = pulseIn(echoPin, HIGH, 30000); // timeout 30ms
  long distance = duration * 0.034 / 2;

  return distance;
}

void loop() {

  obstacleFound = false;

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Searching...");

  for (int i = 0; i < SCAN_STEPS; i++) {

    motor1.step(1);

    long distance = getDistance();

    lcd.setCursor(0, 1);

    if (distance > 0 && distance < THRESHOLD) {
      obstacleFound = true;
      lcd.print("Obstacle     ");
    } else {
      lcd.print("No Obstacle  ");
    }
  }

  delay(1000);

  if (!obstacleFound) {

    lcd.clear();
    lcd.print("Area Free");
    lcd.setCursor(0, 1);
    lcd.print("Motor2 Moving");

    for (int i = 0; i < SCAN_STEPS; i++) {
      motor2.step(1);
    }

  } else {

    lcd.clear();
    lcd.print("Obstacle Found");
    lcd.setCursor(0, 1);
    lcd.print("Rescanning...");
  }

  delay(2000);
}
