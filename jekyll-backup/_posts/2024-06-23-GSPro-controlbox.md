---
title: GSPRO Control box
date: 2024-06-23 00:27:00 -0000
categories: [Automation]
tags: [esp32,automation]     # TAG names should always be lowercase
image: "/assets/img/blog/gspro/ras.png"
---
## esp32 code to create GSPRO Control box

![Desktop View](/assets/img/blog/gspro/bord1.png){: width="350" height="200" }
![Desktop View](/assets/img/blog/gspro/box.png){: width="350" height="200" }



```yaml
// Benedikt Gabriel Egilsson - bensiegils.com
// 1= 5 - Camera to ball
// 2= KEY_M_CTRL - Muligan 
// 3= T - Scorecard display
// 4= P - See the pin
// 5= O - Flyover of the hole
// 6= . - shadow quality
// 7= B - Hide - Make objects in your line of sight invisible 
// 8= Z - Hide or show the 3D gras
// 9= KEY_UP_ARROW
// 10= KEY_DOWN_ARROW
// 11= KEY_LEFT_ARROW
// 12= KEY_RIGHT_ARROW

#define USE_NIMBLE
#include <BleKeyboard.h>

BleKeyboard bleKeyboard("GSPRO BOX");

#define NUM_KEYS 12

struct Key {
  int pin;
  uint8_t code;
  bool state;
  bool isCtrl;  // New field to indicate if Ctrl needs to be held
};

Key keys[NUM_KEYS] = {
  {16, '5', false, false},
  {17, 'm', false, true},  // 'm' with Ctrl modifier
  {18, 't', false, false},
  {19, 'p', false, false},
  {21, 'o', false, false},
  {22, '.', false, false},
  {32, 'b', false, false},
  {33, 'z', false, false},
  {27, KEY_UP_ARROW, false, false},
  {14, KEY_DOWN_ARROW, false, false},
  {12, KEY_LEFT_ARROW, false, false},
  {13, KEY_RIGHT_ARROW, false, false}
};

bool connectNotificationSent = false;

void setup() {
  Serial.begin(115200);
  Serial.println("Code running...");
  
  // Set pin modes
  for (int i = 0; i < NUM_KEYS; i++) {
    pinMode(keys[i].pin, INPUT_PULLUP);
  }
  
  // Initialize BLE keyboard
  bleKeyboard.begin();
}

void loop() {
  if (bleKeyboard.isConnected()) {
    if (!connectNotificationSent) {
      Serial.println("BLE connected...");
      connectNotificationSent = true;
    }
    
    for (int i = 0; i < NUM_KEYS; i++) {
      handleButton(i);
    }
  } else {
    if (connectNotificationSent) {
      Serial.println("BLE disconnected...");
      connectNotificationSent = false;
    }
  }
  delay(10); // Small delay to stabilize readings
}

void handleButton(int keyIndex) {
  bool currentState = !digitalRead(keys[keyIndex].pin); // Read the button state (active low)
  
  if (currentState != keys[keyIndex].state) {
    keys[keyIndex].state = currentState;
    
    if (currentState) {
      if (keys[keyIndex].isCtrl) { // Check if Ctrl modifier is needed
        bleKeyboard.press(KEY_LEFT_CTRL);
      }
      bleKeyboard.press(keys[keyIndex].code);
      Serial.print("Key pressed: ");
    } else {
      bleKeyboard.release(keys[keyIndex].code);
      if (keys[keyIndex].isCtrl) { // Check if Ctrl modifier is needed
        bleKeyboard.release(KEY_LEFT_CTRL);
      }
      Serial.print("Key released: ");
    }
    Serial.println(keys[keyIndex].code);
  }
}
```
