# Ecocoders-_-08
device codes
#include <WiFi.h>
#include <HTTPClient.h>
#include "DHT.h"

// ----------- WiFi Credentials -----------
const char* ssid = "YOUR_WIFI_NAME";
const char* password = "YOUR_WIFI_PASSWORD";

// ----------- Cloud Endpoint -----------
String serverName = "https://api.thingspeak.com/update?api_key=YOUR_API_KEY";

// ----------- Sensor Pins -----------
#define DHTPIN 4
#define DHTTYPE DHT11
#define AIR_SENSOR 34   // MQ-135 analog pin

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);
  dht.begin();

  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("\nConnected to WiFi");
}

void loop() {
  if (WiFi.status() == WL_CONNECTED) {

    float temperature = dht.readTemperature();
    float humidity = dht.readHumidity();
    int airQuality = analogRead(AIR_SENSOR);

    if (isnan(temperature) || isnan(humidity)) {
      Serial.println("Sensor error");
      return;
    }

    String url = serverName +
                 "&field1=" + String(temperature) +
                 "&field2=" + String(humidity) +
                 "&field3=" + String(airQuality);

    HTTPClient http;
    http.begin(url);
    int httpResponseCode = http.GET();

    Serial.println("Data Sent:");
    Serial.println(url);

    http.end();
  }

  delay(15000); // 15 seconds
}






import requests
import time

API_URL = "https://api.thingspeak.com/channels/YOUR_CHANNEL_ID/feeds.json?results=1"

AIR_QUALITY_LIMIT = 700

def fetch_data():
    response = requests.get(API_URL)
    data = response.json()
    feed = data["feeds"][0]

    temperature = float(feed["field1"])
    humidity = float(feed["field2"])
    air_quality = int(feed["field3"])

    return temperature, humidity, air_quality

while True:
    temp, hum, air = fetch_data()

    print(f"Temperature: {temp} °C")
    print(f"Humidity: {hum} %")
    print(f"Air Quality Index: {air}")

    if air > AIR_QUALITY_LIMIT:
        print("⚠ ALERT: Poor Air Quality Detected!")

    print("-" * 40)
    time.sleep(20)
