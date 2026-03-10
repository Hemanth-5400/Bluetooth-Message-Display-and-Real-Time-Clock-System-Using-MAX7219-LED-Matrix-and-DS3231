# Bluetooth-Message-Display-and-Real-Time-Clock-System-Using-MAX7219-LED-Matrix-and-DS3231
#include <MD_Parola.h>
#include <MD_MAX72xx.h>
#include <SPI.h>
#include <Wire.h>
#include <I2C_RTC.h>

static DS3231 RTC;

#define HARDWARE_TYPE MD_MAX72XX::FC16_HW
#define MAX_DEVICES 8
#define CS_PIN 10

MD_Parola P = MD_Parola(HARDWARE_TYPE, CS_PIN, MAX_DEVICES);

char btMessage[40] = "";
bool displayMessage = false;
unsigned long messageStartTime = 0;
const unsigned long displayDuration = 180000;  // 3 minutes

void setup() {
  P.begin();
  P.setIntensity(10);
  P.displayClear();

  Serial.begin(9600);

  RTC.begin();
  RTC.setHourMode(CLOCK_H12);

  // Optional: Set initial RTC time
  RTC.setDay(10);
  RTC.setMonth(4);
  RTC.setYear(2025);
  RTC.setHours(6);
  RTC.setMinutes(2);
  RTC.setSeconds(0);
  RTC.setWeek(4); // Assuming Thursday
}

void loop() {
  if (Serial.available()) {
    String receivedText = Serial.readStringUntil('\n');
    receivedText.trim();
    if (receivedText.length() > 0) {
      receivedText.toCharArray(btMessage, sizeof(btMessage));
      Serial.print("Received: ");
      Serial.println(btMessage);
      displayMessage = true;
      messageStartTime = millis();
      P.displayClear();
    }
  }

  if (displayMessage) {
    if (millis() - messageStartTime < displayDuration) {
      if (P.displayAnimate()) {
        P.displayText(btMessage, PA_CENTER, 100, 0, PA_SCROLL_LEFT);
      }
    } else {
      displayMessage = false;
      P.displayClear();
    }
  } else {
    displayDateTime();
  }

  delay(100);
}

void displayDateTime() {
  uint8_t day = RTC.getDay();
  uint8_t month = RTC.getMonth();
  uint16_t year = RTC.getYear();
  uint8_t hours = RTC.getHours();
  uint8_t minutes = RTC.getMinutes();
  uint8_t seconds = RTC.getSeconds();

  char displayText[40];
  snprintf(displayText, sizeof(displayText), "DATE:%02d-%02d-%d | TIME:%02d:%02d:%02d   ", day, month, year, hours, minutes, seconds);

  if (P.displayAnimate()) {
    P.displayText(displayText, PA_CENTER, 100, 0, PA_SCROLL_LEFT);
  }

  // Also print to Serial
  Serial.print(" ");
  Serial.print(day);
  Serial.print("-");
  Serial.print(month);
  Serial.print("-");
  Serial.print(year);
  Serial.print(" ");
  Serial.print(hours);
  Serial.print(":");
  Serial.print(minutes);
  Serial.print(":");
  Serial.print(seconds);

  if (RTC.getHourMode() == CLOCK_H12) {
    if (RTC.getMeridiem() == HOUR_AM)
      Serial.println(" AM");
    else
      Serial.println(" PM");
  } else {
    Serial.println();
  }
}
