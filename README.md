# smart-trash
<img width="400" height="400" alt="image" src="https://github.com/user-attachments/assets/58973aea-d865-45e7-81ab-87fa1e084e8b" />

## Introduction

Smart Trash is an IoT-based reminder system that helps households remember waste collection days.
The system retrieves waste collection data from an online API and translates this information into a physical reminder inside the home.

When a waste collection day is approaching, the device lights up in a specific color that represents the type of waste. 
The reminder stays active until the user presses a physical button to confirm that the waste has been taken outside.

### The manual is divided into 5 steps
1. Connecting the LED strip and button
2. Setting up the Waste Calendar API
3. Installing libraries
4. Writing the code
5. Putting everything together

## Prerequisites

When following this manual, I assume that you have the following hardware & software installed. If this is not the case, please set-up your Microcontroller correctly before following this manual.

### Hardware
- NodeMCU ESP8266 Microcontroller (or similar board)
- Led strip
- Push button
- Jumpers

### Software
- Arduino IDE
- Wi-Fi credentials
- API access key (if you dont have the API key go to step 1, otherwise go to step 1.5)
  
### Required Libraries
Install these libraries using Arduino IDE → Library Manager:
- ArduinoJson
- Adafruit NeoPixel

The following libraries are included by default for ESP8266 boards:
- ESP8266WiFi.h
- ESP8266HTTPClient.h

## Step 1 – Hardware setup

### 3.1: Connect ledstrip
Connect the LED strip as follows:

5V of the LED strip (red) - VIN 
GND of the LED strip (black) - GND 
D of the LED strip (yellow) - D5 

### 3.2  Button
Connect the button as follows with jumper wires:

vcc - 3v3
GND - GND 
OUT - D2

## Step 2:API access

Smart Trash uses the Amsterdam data API to retrieve waste collection data. To access the API, an API key is required.

Register a client at:
https://keys.api.data.amsterdam.nl/clients/v1/register/

Store your API key in a separate configuration file (e.g. config.h).

Never commit your API key to GitHub!!!

### Create a config file
Create a file called config.h in your Arduino project and add this information:

#define WIFI_SSID "YOUR_WIFI_NAME"
#define WIFI_PASSWORD "YOUR_WIFI_PASSWORD"
#define API_KEY "YOUR_API_KEY"

When the device starts, it connects to the Wi-Fi network before making the API request.

## Step 3: Install libraries

Open Arduino IDE and go to:
Sketch → Include Library → Manage Libraries

Install:
- ArduinoJson
- Adafruit NeoPixel

These libraries are required for:
- Reading JSON data from the API
- Controlling the LED strip

## Step 4: The code

Create a new Arduino sketch and paste everything below.

```ruby
Smart Trash Code
#include <ESP8266WiFi.h>
#include <ArduinoJson.h>
#include <Adafruit_NeoPixel.h>
#include "config.h"

#define LED_PIN D5
#define LED_COUNT 8
#define BUTTON_PIN D6

Adafruit_NeoPixel strip(LED_COUNT, LED_PIN, NEO_GRB + NEO_KHZ800);

bool reminderActive = false;

void setup() {
  Serial.begin(9600);

  pinMode(BUTTON_PIN, INPUT_PULLUP);

  strip.begin();
  strip.show();

  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
  Serial.print("Connecting to WiFi");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nConnected to WiFi");
  getWasteData();
}

void loop() {
  if (reminderActive && digitalRead(BUTTON_PIN) == LOW) {
    turnOffLED();
    reminderActive = false;
    Serial.println("Trash confirmed");
    delay(500);
  }
}

void getWasteData() {
  WiFiClient client;
  HTTPClient http;

  String url = "https://api.data.amsterdam.nl/afvalwijzer/afvalkalender/";
  http.begin(client, url);
  http.addHeader("Authorization", "Bearer " + String(API_KEY));

  int httpCode = http.GET();

  if (httpCode > 0) {
    Serial.println("Waste data received");
    activateReminder();
  } else {
    Serial.println("API request failed");
  }

  http.end();
}

void activateReminder() {
  for (int i = 0; i < LED_COUNT; i++) {
    strip.setPixelColor(i, strip.Color(0, 255, 0)); // Green for GFT
  }
  strip.show();
  reminderActive = true;
}

void turnOffLED() {
  strip.clear();
  strip.show();
}

```





eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCJ9.eyJzdWIiOjI0MDU0OTc0ODkzMjkyODgzODMsImV4cCI6MTc5ODgzMjkzMn0.Yj1S5WL0nyAXjuQ6N6lbvvBNiGczZlOuzU3sUxOREcgKPtoDZm71oSHIr2n4RzoHOESyrAxgw80uf2-aV2c0BA
eyJhbGciOiJFZERTQSIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI4OTk0NjcwMTg1ODY4NzcxMzg4IiwiZXhwIjoxODAwMzg2OTMwfQ.H-aJdPgkxXIhbK3Pjf_gip3Z9qH3FC3RqgfVZZ-n8HkXC57ixJFpm5DpzxrfhEMHOArC0EcIs0klfknafLrYDQ

