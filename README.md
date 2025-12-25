# project
Automatic Humidifier for Textile Industry
#define BLYNK_PRINT Serial

#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <DHT.h>

/* Blynk Credentials */
char auth[] = "YOUR_BLYNK_AUTH_TOKEN";
char ssid[] = "YOUR_WIFI_NAME";
char pass[] = "YOUR_WIFI_PASSWORD";

/* Pin Definitions */
#define DHTPIN D4
#define DHTTYPE DHT11
#define RELAY_PIN D1

DHT dht(DHTPIN, DHTTYPE);

/* Variables */
float humidity = 0;
float temperature = 0;

int setHumidity = 65;   // Default required humidity
int manualSwitch = 0;  // Manual ON/OFF
bool humidifierState = false;

/* Timer */
BlynkTimer timer;

/* Read Sensor Data */
void readDHT() {
  humidity = dht.readHumidity();
  temperature = dht.readTemperature();

  if (isnan(humidity) || isnan(temperature)) {
    Serial.println("DHT Sensor Error");
    return;
  }

  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.print("%  Temperature: ");
  Serial.print(temperature);
  Serial.println("°C");

  /* Send data to Blynk */
  Blynk.virtualWrite(V0, humidity);
  Blynk.virtualWrite(V1, temperature);

  controlHumidifier();
}

/* Control Logic */
void controlHumidifier() {

  if (manualSwitch == 1) {
    digitalWrite(RELAY_PIN, LOW);
    humidifierState = true;
  } 
  else {
    if (humidity < setHumidity) {
      digitalWrite(RELAY_PIN, LOW);   // ON
      humidifierState = true;
    } else {
      digitalWrite(RELAY_PIN, HIGH);  // OFF
      humidifierState = false;
    }
  }

  /* Update Status LED */
  if (humidifierState)
    Blynk.virtualWrite(V3, 255);
  else
    Blynk.virtualWrite(V3, 0);
}

/* Blynk Slider for Set Humidity */
BLYNK_WRITE(V2) {
  setHumidity = param.asInt();
  Serial.print("Set Humidity: ");
  Serial.println(setHumidity);
}

/* Manual Switch */
BLYNK_WRITE(V4) {
  manualSwitch = param.asInt();
  Serial.print("Manual Switch: ");
  Serial.println(manualSwitch);
}

void setup() {
  Serial.begin(9600);

  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, HIGH); // Humidifier OFF initially

  dht.begin();

  Blynk.begin(auth, ssid, pass);

  timer.setInterval(2000L, readDHT);
}

void loop() {
  Blynk.run();
  timer.run();
}
