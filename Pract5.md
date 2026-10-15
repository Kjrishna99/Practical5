# IoT Practicals for Wokwi

**How to use:** In Wokwi create a new project (board noted per practical). Then for each practical:
1. Paste the **sketch.ino** tab.
2. Open the **diagram.json** tab and paste it (replaces everything).
3. Open the **libraries.txt** tab (click the blue "Library Manager" / create `libraries.txt`) and paste the lines. If it says "none", skip it.

**Important:** Wokwi cannot simulate a full Linux Raspberry Pi (3/4/5). So all "Raspberry Pi" practicals use the **Raspberry Pi Pico W** (same 40-pin style GPIO idea, RP2040, Arduino-Pico core). Start those projects with **"Raspberry Pi Pico W"** + Arduino/C++ template.
WiFi in Wokwi always uses the network `Wokwi-GUEST` (no password).

---

## Practical 1: Raspberry Pi 40-pin GPIO study
Board: Raspberry Pi Pico W (Arduino C++). Shows digital output (3 LEDs) and digital input (button).

### sketch.ino
```cpp
// Practical 1: GPIO study - output (LEDs) and input (push button)
const int ledPins[] = {15, 14, 13};   // GP15, GP14, GP13
const int numLeds = 3;
const int BUTTON = 16;                // GP16, active LOW (internal pull-up)

void setup() {
  Serial.begin(115200);
  for (int i = 0; i < numLeds; i++) pinMode(ledPins[i], OUTPUT);
  pinMode(BUTTON, INPUT_PULLUP);

  Serial.println("=== GPIO Study ===");
  Serial.println("Outputs : GP15, GP14, GP13 -> LEDs via 220 ohm resistors");
  Serial.println("Input   : GP16 -> push button (pull-up, pressed = LOW)");
  Serial.println("Power   : 3V3 (pin 36), VBUS 5V (pin 40), GND pins 3,8,13,18,23,28,33,38");
}

void loop() {
  if (digitalRead(BUTTON) == LOW) {
    // Button pressed: all LEDs ON
    for (int i = 0; i < numLeds; i++) digitalWrite(ledPins[i], HIGH);
    Serial.println("Button PRESSED -> all LEDs ON");
    delay(200);
  } else {
    // Chaser pattern
    for (int i = 0; i < numLeds; i++) {
      digitalWrite(ledPins[i], HIGH);
      delay(250);
      digitalWrite(ledPins[i], LOW);
      if (digitalRead(BUTTON) == LOW) break;
    }
  }
}
```

### diagram.json
```json
{
  "version": 1,
  "author": "Practical 1",
  "editor": "wokwi",
  "parts": [
    { "type": "board-pi-pico-w", "id": "pico", "top": 0, "left": 0, "attrs": {} },
    { "type": "wokwi-led", "id": "led1", "top": -20, "left": 160, "attrs": { "color": "red" } },
    { "type": "wokwi-led", "id": "led2", "top": 30, "left": 160, "attrs": { "color": "green" } },
    { "type": "wokwi-led", "id": "led3", "top": 80, "left": 160, "attrs": { "color": "blue" } },
    { "type": "wokwi-resistor", "id": "r1", "top": -10, "left": 100, "attrs": { "value": "220" } },
    { "type": "wokwi-resistor", "id": "r2", "top": 40, "left": 100, "attrs": { "value": "220" } },
    { "type": "wokwi-resistor", "id": "r3", "top": 90, "left": 100, "attrs": { "value": "220" } },
    { "type": "wokwi-pushbutton", "id": "btn1", "top": 150, "left": 160, "attrs": { "color": "yellow" } }
  ],
  "connections": [
    [ "pico:GP15", "r1:1", "green", [] ],
    [ "r1:2", "led1:A", "green", [] ],
    [ "led1:C", "pico:GND.8", "black", [] ],
    [ "pico:GP14", "r2:1", "green", [] ],
    [ "r2:2", "led2:A", "green", [] ],
    [ "led2:C", "pico:GND.8", "black", [] ],
    [ "pico:GP13", "r3:1", "green", [] ],
    [ "r3:2", "led3:A", "green", [] ],
    [ "led3:C", "pico:GND.8", "black", [] ],
    [ "btn1:1.l", "pico:GP16", "yellow", [] ],
    [ "btn1:2.l", "pico:GND.8", "black", [] ]
  ],
  "dependencies": {}
}
```

### libraries.txt
```
none needed
```

---

## Practical 2: Connect Arduino to a WiFi network
Board: **ESP32 DevKit** (Arduino boards like Uno have no WiFi; ESP32 is programmed with the Arduino IDE and is the standard "Arduino WiFi" choice).

### sketch.ino
```cpp
// Practical 2: Connect to WiFi and show connection details
#include <WiFi.h>

const char* ssid = "Wokwi-GUEST";   // Wokwi's built-in WiFi
const char* password = "";          // no password
const int LED = 2;

void setup() {
  Serial.begin(115200);
  pinMode(LED, OUTPUT);

  Serial.print("Connecting to ");
  Serial.println(ssid);
  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password, 6);    // channel 6 = faster connect on Wokwi

  while (WiFi.status() != WL_CONNECTED) {
    digitalWrite(LED, !digitalRead(LED));   // blink while connecting
    delay(300);
    Serial.print(".");
  }

  digitalWrite(LED, HIGH);          // solid ON = connected
  Serial.println("\nWiFi connected!");
  Serial.print("IP address : "); Serial.println(WiFi.localIP());
  Serial.print("Gateway    : "); Serial.println(WiFi.gatewayIP());
  Serial.print("MAC        : "); Serial.println(WiFi.macAddress());
  Serial.print("Signal RSSI: "); Serial.print(WiFi.RSSI()); Serial.println(" dBm");
}

void loop() {
  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("WiFi lost, reconnecting...");
    digitalWrite(LED, LOW);
    WiFi.reconnect();
    delay(3000);
  }
  delay(1000);
}
```

### diagram.json
```json
{
  "version": 1,
  "author": "Practical 2",
  "editor": "wokwi",
  "parts": [
    { "type": "board-esp32-devkit-c-v4", "id": "esp", "top": 0, "left": 0, "attrs": {} },
    { "type": "wokwi-led", "id": "led1", "top": -10, "left": 170, "attrs": { "color": "green" } },
    { "type": "wokwi-resistor", "id": "r1", "top": 60, "left": 170, "attrs": { "value": "220" } }
  ],
  "connections": [
    [ "esp:TX", "$serialMonitor:RX", "", [] ],
    [ "esp:RX", "$serialMonitor:TX", "", [] ],
    [ "esp:2", "r1:1", "green", [] ],
    [ "r1:2", "led1:A", "green", [] ],
    [ "led1:C", "esp:GND.1", "black", [] ]
  ],
  "dependencies": {}
}
```

### libraries.txt
```
none needed
```

---

## Practical 3: Retrieve temperature & humidity from ThingSpeak cloud
Board: Raspberry Pi Pico W (Arduino C++). Reads the latest values from a ThingSpeak channel over HTTP.

Setup: the code defaults to ThingSpeak's public weather-station channel **12397** (field 4 = temperature, field 3 = humidity). Check those field names on the channel page. To use your own data, change `CHANNEL_ID`, the field numbers, and (for a private channel) set `READ_KEY`.

### sketch.ino
```cpp
// Practical 3: Read temperature & humidity from ThingSpeak
#include <WiFi.h>
#include <HTTPClient.h>

const char* ssid = "Wokwi-GUEST";
const char* password = "";

const char* CHANNEL_ID = "12397";   // change to your channel ID
const int FIELD_TEMP = 4;           // field number holding temperature
const int FIELD_HUM  = 3;           // field number holding humidity
const char* READ_KEY = "";          // leave empty for public channels

String readField(int field) {
  String url = "http://api.thingspeak.com/channels/" + String(CHANNEL_ID) +
               "/fields/" + String(field) + "/last.txt";
  if (strlen(READ_KEY) > 0) url += "?api_key=" + String(READ_KEY);

  HTTPClient http;
  http.begin(url);
  int code = http.GET();
  String payload = (code == 200) ? http.getString() : "ERR " + String(code);
  http.end();
  payload.trim();
  return payload;
}

void setup() {
  Serial.begin(115200);
  WiFi.begin(ssid, password);
  Serial.print("Connecting");
  while (WiFi.status() != WL_CONNECTED) { delay(300); Serial.print("."); }
  Serial.println("\nConnected! IP: " + WiFi.localIP().toString());
}

void loop() {
  String temp = readField(FIELD_TEMP);
  String hum  = readField(FIELD_HUM);

  Serial.println("------ ThingSpeak data ------");
  Serial.println("Temperature : " + temp);
  Serial.println("Humidity    : " + hum);

  delay(16000);   // ThingSpeak free tier: at most 1 request per ~15 s
}
```

### diagram.json
```json
{
  "version": 1,
  "author": "Practical 3",
  "editor": "wokwi",
  "parts": [
    { "type": "board-pi-pico-w", "id": "pico", "top": 0, "left": 0, "attrs": {} }
  ],
  "connections": [],
  "dependencies": {}
}
```

### libraries.txt
```
none needed
```

---

## Practical 4: Bluetooth control of LED (ON/OFF from smartphone)
Board: Raspberry Pi Pico W (Arduino C++).

**Wokwi limitation:** Wokwi has no Bluetooth/HC-05 simulation. This code reads commands from a serial port. In Wokwi, type `1` or `0` in the Serial Monitor (acts as the phone). On real hardware, connect an **HC-05** (HC-05 TX → GP1, RX → GP0 through a voltage divider, VCC → 5V/VBUS, GND → GND), set `USE_HC05 true`, pair the phone, and send `1`/`0` from a "Serial Bluetooth Terminal" app.

### sketch.ino
```cpp
// Practical 4: LED ON/OFF via Bluetooth (HC-05) / Serial Monitor in Wokwi
#define USE_HC05 false          // set true on real hardware with HC-05

const int LED = 15;             // GP15

#if USE_HC05
  #define BT Serial1            // UART0: GP0 = TX, GP1 = RX
#else
  #define BT Serial             // Wokwi: use Serial Monitor
#endif

void setup() {
  Serial.begin(115200);
#if USE_HC05
  Serial1.setTX(0);
  Serial1.setRX(1);
  Serial1.begin(9600);          // HC-05 default baud
#endif
  pinMode(LED, OUTPUT);
  digitalWrite(LED, LOW);
  Serial.println("Send '1' = LED ON, '0' = LED OFF");
}

void loop() {
  if (BT.available()) {
    char c = BT.read();
    if (c == '1') {
      digitalWrite(LED, HIGH);
      Serial.println("LED ON");
      BT.println("LED ON");
    } else if (c == '0') {
      digitalWrite(LED, LOW);
      Serial.println("LED OFF");
      BT.println("LED OFF");
    }
  }
}
```

### diagram.json
```json
{
  "version": 1,
  "author": "Practical 4",
  "editor": "wokwi",
  "parts": [
    { "type": "board-pi-pico-w", "id": "pico", "top": 0, "left": 0, "attrs": {} },
    { "type": "wokwi-led", "id": "led1", "top": 0, "left": 170, "attrs": { "color": "red" } },
    { "type": "wokwi-resistor", "id": "r1", "top": 70, "left": 170, "attrs": { "value": "220" } }
  ],
  "connections": [
    [ "pico:GP15", "r1:1", "green", [] ],
    [ "r1:2", "led1:A", "green", [] ],
    [ "led1:C", "pico:GND.8", "black", [] ]
  ],
  "dependencies": {}
}
```

### libraries.txt
```
none needed
```

---

## Practical 5: Publish temperature data to an MQTT broker
Board: Raspberry Pi Pico W (Arduino C++) + DHT22.
Verify the data: open https://www.hivemq.com/demos/websocket-client/ , connect, and subscribe to `iot/practical5/temperature`.

### sketch.ino
```cpp
// Practical 5: Publish DHT22 temperature to a public MQTT broker
#include <WiFi.h>
#include <PubSubClient.h>
#include <DHT.h>

#define DHTPIN 15
#define DHTTYPE DHT22

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "broker.hivemq.com";
const int   mqtt_port = 1883;
const char* topic_temp = "iot/practical5/temperature";
const char* topic_hum  = "iot/practical5/humidity";

WiFiClient espClient;
PubSubClient client(espClient);
DHT dht(DHTPIN, DHTTYPE);

void connectMQTT() {
  while (!client.connected()) {
    Serial.print("Connecting to MQTT... ");
    String id = "pico-" + String(random(0xffff), HEX);
    if (client.connect(id.c_str())) {
      Serial.println("connected");
    } else {
      Serial.print("failed, rc=");
      Serial.println(client.state());
      delay(2000);
    }
  }
}

void setup() {
  Serial.begin(115200);
  dht.begin();
  WiFi.begin(ssid, password);
  Serial.print("Connecting WiFi");
  while (WiFi.status() != WL_CONNECTED) { delay(300); Serial.print("."); }
  Serial.println("\nWiFi connected");
  client.setServer(mqtt_server, mqtt_port);
}

void loop() {
  if (!client.connected()) connectMQTT();
  client.loop();

  float t = dht.readTemperature();
  float h = dht.readHumidity();
  if (isnan(t) || isnan(h)) {
    Serial.println("DHT read failed");
  } else {
    char tbuf[10], hbuf[10];
    dtostrf(t, 4, 1, tbuf);
    dtostrf(h, 4, 1, hbuf);
    client.publish(topic_temp, tbuf);
    client.publish(topic_hum, hbuf);
    Serial.print("Published  Temp: "); Serial.print(tbuf);
    Serial.print(" C   Hum: ");        Serial.print(hbuf);
    Serial.println(" %");
  }
  delay(5000);
}
```

### diagram.json
```json
{
  "version": 1,
  "author": "Practical 5",
  "editor": "wokwi",
  "parts": [
    { "type": "board-pi-pico-w", "id": "pico", "top": 0, "left": 0, "attrs": {} },
    { "type": "wokwi-dht22", "id": "dht1", "top": -20, "left": 170, "attrs": { "temperature": "28", "humidity": "55" } }
  ],
  "connections": [
    [ "dht1:VCC", "pico:3V3", "red", [] ],
    [ "dht1:SDA", "pico:GP15", "green", [] ],
    [ "dht1:GND", "pico:GND.8", "black", [] ]
  ],
  "dependencies": {}
}
```

### libraries.txt
```
PubSubClient
DHT sensor library
Adafruit Unified Sensor
```

---

## Practical 6: Connect Raspberry Pi to your existing system
Your syllabus line is open-ended, so this uses the most common interpretation: the Pi/Pico is connected to an **existing system (your PC)** over a USB-serial link. It sends sensor data to the PC and accepts commands back (two-way communication). In Wokwi the Serial Monitor plays the role of the PC. On real hardware you would also use SSH/VNC/Ethernet to reach a Pi over the network.

Commands to type in the Serial Monitor: `READ`, `LED ON`, `LED OFF`, `HELP`.

### sketch.ino
```cpp
// Practical 6: Two-way link between Pi/Pico and an existing system (PC)
#include <DHT.h>

#define DHTPIN 15
#define DHTTYPE DHT22
const int LED = 14;

DHT dht(DHTPIN, DHTTYPE);
String cmd = "";

void printHelp() {
  Serial.println("Commands: READ | LED ON | LED OFF | HELP");
}

void sendReading() {
  float t = dht.readTemperature();
  float h = dht.readHumidity();
  Serial.print("DATA,temp=");  Serial.print(t, 1);
  Serial.print(",hum=");       Serial.println(h, 1);
}

void setup() {
  Serial.begin(115200);
  dht.begin();
  pinMode(LED, OUTPUT);
  Serial.println("Pi <-> System link ready");
  printHelp();
}

void loop() {
  while (Serial.available()) {
    char c = Serial.read();
    if (c == '\n' || c == '\r') {
      cmd.trim();
      cmd.toUpperCase();
      if (cmd == "READ")        sendReading();
      else if (cmd == "LED ON") { digitalWrite(LED, HIGH); Serial.println("ACK: LED ON"); }
      else if (cmd == "LED OFF"){ digitalWrite(LED, LOW);  Serial.println("ACK: LED OFF"); }
      else if (cmd == "HELP")   printHelp();
      else if (cmd.length() > 0) Serial.println("ERR: unknown command");
      cmd = "";
    } else {
      cmd += c;
    }
  }

  // Automatic report every 10 seconds
  static unsigned long last = 0;
  if (millis() - last > 10000) {
    last = millis();
    sendReading();
  }
}
```

### diagram.json
```json
{
  "version": 1,
  "author": "Practical 6",
  "editor": "wokwi",
  "parts": [
    { "type": "board-pi-pico-w", "id": "pico", "top": 0, "left": 0, "attrs": {} },
    { "type": "wokwi-dht22", "id": "dht1", "top": -20, "left": 170, "attrs": { "temperature": "27", "humidity": "60" } },
    { "type": "wokwi-led", "id": "led1", "top": 90, "left": 170, "attrs": { "color": "yellow" } },
    { "type": "wokwi-resistor", "id": "r1", "top": 150, "left": 170, "attrs": { "value": "220" } }
  ],
  "connections": [
    [ "dht1:VCC", "pico:3V3", "red", [] ],
    [ "dht1:SDA", "pico:GP15", "green", [] ],
    [ "dht1:GND", "pico:GND.8", "black", [] ],
    [ "pico:GP14", "r1:1", "green", [] ],
    [ "r1:2", "led1:A", "green", [] ],
    [ "led1:C", "pico:GND.8", "black", [] ]
  ],
  "dependencies": {}
}
```

### libraries.txt
```
DHT sensor library
Adafruit Unified Sensor
```

---

## Practical 7: IoT-based DC motor speed control (Arduino)
Board: **Arduino Uno**. A potentiometer sets the speed using PWM on pin 9.

**Wokwi limitation:** Wokwi has no plain DC motor part, so an LED stands in for the motor (brightness = speed) and the Serial Monitor prints the speed %. On real hardware, pin 9 goes to an L298N/L293D enable pin (or a transistor/MOSFET with a flyback diode) driving the motor from its own supply. The IoT part (remote speed) is the optional `S<number>` serial command, which can be replaced by WiFi/MQTT/Bluetooth.

### sketch.ino
```cpp
// Practical 7: DC motor speed control using PWM (LED simulates motor in Wokwi)
const int POT = A0;
const int MOTOR_PWM = 9;      // PWM pin -> motor driver enable / LED in Wokwi

int speedPercent = 0;
bool remoteMode = false;      // true when a speed is set by serial command

void setup() {
  Serial.begin(9600);
  pinMode(MOTOR_PWM, OUTPUT);
  Serial.println("Turn the pot to change speed, or send S50 (0-100) for remote control, P to return to pot");
}

void loop() {
  if (Serial.available()) {
    char c = Serial.read();
    if (c == 'S' || c == 's') {
      int v = Serial.parseInt();
      speedPercent = constrain(v, 0, 100);
      remoteMode = true;
    } else if (c == 'P' || c == 'p') {
      remoteMode = false;
    }
  }

  if (!remoteMode) {
    int raw = analogRead(POT);                  // 0-1023
    speedPercent = map(raw, 0, 1023, 0, 100);
  }

  int pwm = map(speedPercent, 0, 100, 0, 255);
  analogWrite(MOTOR_PWM, pwm);

  Serial.print(remoteMode ? "[REMOTE] " : "[POT] ");
  Serial.print("Speed: ");
  Serial.print(speedPercent);
  Serial.print("%  PWM: ");
  Serial.println(pwm);
  delay(300);
}
```

### diagram.json
```json
{
  "version": 1,
  "author": "Practical 7",
  "editor": "wokwi",
  "parts": [
    { "type": "wokwi-arduino-uno", "id": "uno", "top": 0, "left": 0, "attrs": {} },
    { "type": "wokwi-potentiometer", "id": "pot1", "top": -90, "left": 200, "attrs": {} },
    { "type": "wokwi-led", "id": "led1", "top": -90, "left": 320, "attrs": { "color": "red" } },
    { "type": "wokwi-resistor", "id": "r1", "top": -30, "left": 320, "attrs": { "value": "220" } }
  ],
  "connections": [
    [ "pot1:VCC", "uno:5V", "red", [] ],
    [ "pot1:GND", "uno:GND.2", "black", [] ],
    [ "pot1:SIG", "uno:A0", "green", [] ],
    [ "uno:9", "r1:1", "green", [] ],
    [ "r1:2", "led1:A", "green", [] ],
    [ "led1:C", "uno:GND.1", "black", [] ]
  ],
  "dependencies": {}
}
```

### libraries.txt
```
none needed
```

---

## Practical 8: Observe real-time "spikes" on ThingSpeak
Board: Raspberry Pi Pico W (Arduino C++) + DHT22. Sends temperature and humidity to your ThingSpeak channel every 20 s. While it runs, click the DHT22 in Wokwi and drag the temperature slider to create a sudden spike, then watch it appear on the ThingSpeak chart.

ThingSpeak setup: sign in at thingspeak.com → New Channel → enable **Field 1** (Temperature) and **Field 2** (Humidity) → Save → API Keys tab → copy the **Write API Key** into the code. Use the channel's Private/Public View to watch the live chart.

### sketch.ino
```cpp
// Practical 8: Send live data to ThingSpeak and observe spikes in real time
#include <WiFi.h>
#include <HTTPClient.h>
#include <DHT.h>

#define DHTPIN 15
#define DHTTYPE DHT22

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* WRITE_API_KEY = "YOUR_WRITE_API_KEY";   // <-- paste your key

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(115200);
  dht.begin();
  WiFi.begin(ssid, password);
  Serial.print("Connecting WiFi");
  while (WiFi.status() != WL_CONNECTED) { delay(300); Serial.print("."); }
  Serial.println("\nConnected: " + WiFi.localIP().toString());
}

void loop() {
  float t = dht.readTemperature();
  float h = dht.readHumidity();

  if (isnan(t) || isnan(h)) {
    Serial.println("DHT read failed");
  } else {
    String url = "http://api.thingspeak.com/update?api_key=" + String(WRITE_API_KEY) +
                 "&field1=" + String(t, 1) + "&field2=" + String(h, 1);

    HTTPClient http;
    http.begin(url);
    int code = http.GET();
    String entry = http.getString();     // entry number, 0 = rejected
    http.end();

    Serial.print("Temp: "); Serial.print(t, 1);
    Serial.print(" C  Hum: "); Serial.print(h, 1);
    Serial.print(" %  -> HTTP "); Serial.print(code);
    Serial.print(", entry #"); Serial.println(entry);
  }
  delay(20000);   // ThingSpeak free limit: 15 s minimum between updates
}
```

### diagram.json
```json
{
  "version": 1,
  "author": "Practical 8",
  "editor": "wokwi",
  "parts": [
    { "type": "board-pi-pico-w", "id": "pico", "top": 0, "left": 0, "attrs": {} },
    { "type": "wokwi-dht22", "id": "dht1", "top": -20, "left": 170, "attrs": { "temperature": "26", "humidity": "50" } }
  ],
  "connections": [
    [ "dht1:VCC", "pico:3V3", "red", [] ],
    [ "dht1:SDA", "pico:GP15", "green", [] ],
    [ "dht1:GND", "pico:GND.8", "black", [] ]
  ],
  "dependencies": {}
}
```

### libraries.txt
```
DHT sensor library
Adafruit Unified Sensor
```

---

## Quick troubleshooting
- **"Library not found":** the lines in `libraries.txt` must match Library Manager names exactly (one per line).
- **A pin name error in diagram.json:** open the part in Wokwi and hover the pin to see its exact name, or just draw the wire by hand and Wokwi rewrites the JSON.
- **WiFi won't connect:** use exactly `Wokwi-GUEST` with an empty password.
- **ThingSpeak returns entry 0:** you are updating faster than every 15 seconds, or the API key is wrong.
