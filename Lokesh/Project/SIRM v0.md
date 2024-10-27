# **Smart IoT Room Monitoring System**
_____________________________________________________________________
## Overview
This project aims to create a smart IoT system that monitors various environmental parameters within a room, including temperature, humidity, and distance.
    
- The system utilizes an **`ESP32`** microcontroller to collect sensor data and transmit it to a **Node-RED** dashboard for _visualization_ and _analysis_.

## Hardware Components
- `ESP32 Development Board`: The core of the system, responsible for data acquisition and transmission.
- `DHT22 Temperature and Humidity Sensor`: Measures temperature and humidity.
- `HC-SR04 Ultrasonic Sensor`: Measures distance.
- `LED`: Indicates alert status.
- `Push Button`: Triggers alert mode.

## Software Components
- **ESP32 Firmware** : Written in C++, the firmware reads sensor data, connects to Wi-Fi, and publishes data to an MQTT broker.
- **Node-RED**: A flow-based programming tool used to visualize and control the IoT system.

## Functionality

- **Sensor Data Acquisition**:
The ESP32 periodically reads data from the DHT22 and HC-SR04 sensors.
- **Wi-Fi Connectivity**:
The ESP32 connects to a Wi-Fi network to enable communication with the MQTT broker.
- **MQTT Communication**:
Sensor data is published to an MQTT broker using a predefined topic.
- **Node-RED Dashboard**:
    - The Node-RED flow subscribes to the MQTT topic and receives sensor data.
    - The data is displayed on gauges, charts, and other visual elements.
    - Alert conditions are defined and triggered based on sensor values.
    - Alerts can be displayed on the dashboard and sent via notifications.
- **User Interaction**:
A push button on the ESP32 can be used to manually trigger alerts or change system settings.

## Working Procedure

 Step#1: The Espressif 32 is connected to the Arduino IDE, after configuring the correct **COM** port [ _`Silicon Labs CP210c USB to UART Bridge`_ ] from the **DeviceManager** ( _here, `COM12`_ ).

Step#2: From Boards Manager, we need to select `ESP32 Dev Module` to start working on it.

Step#3: Installing the **WiFi.h**, **PubSubClient**, **DHT**, and **ArduinoJson** libraries from the _Library Manager_.

Step#4: The following sketch is executed:

```c++
#include <DHT.h>
#include <WiFi.h>
#include <PubSubClient.h>
#include <ArduinoJson.h>

// WiFi credentials
const char* ssid = "Airtel_77";
const char* password = "380565750";

// MQTT Broker settings
const char* mqtt_server = "192.168.1.6";
const int mqtt_port = 1883;
const char* mqtt_topic = "home/sensors";

// Pin Definitions
#define TRIG_PIN 5
#define ECHO_PIN 18
#define DHT_PIN 4
#define LED_PIN 2
#define BUTTON_PIN 15
#define DHTTYPE DHT22

// Initialize objects
DHT dht(DHT_PIN, DHTTYPE);
WiFiClient espClient;
PubSubClient client(espClient);

// Variables
float temperature;
float humidity;
long duration;
float distance;
bool alertMode = false;
unsigned long lastReadingTime = 0;
const int readingInterval = 2000;

void setup_wifi() {
  delay(10);
  Serial.println("Connecting to WiFi");
  WiFi.begin(ssid, password);

  int retries = 0;
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
    retries++;

    if (retries >= 10) {
      Serial.println("\nWiFi connection failed. Retrying in 10 seconds.");
      delay(10000);
      retries = 0;
      WiFi.begin(ssid, password);
    }
  }
  Serial.println("\nWiFi connected");
}

void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    String clientId = "ESP32Client-";
    clientId += String(random(0xffff), HEX);

    if (client.connect(clientId.c_str())) {
      Serial.println("connected");
      client.subscribe("home/control"); // Subscribe to control messages
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" retrying in 5 seconds");
      delay(5000);
    }
  }
}

void callback(char* topic, byte* payload, unsigned int length) {
  // Handle incoming MQTT messages
  if (String(topic) == "home/control") {
    DynamicJsonDocument doc(1024);
    deserializeJson(doc, payload, length);

    if (doc.containsKey("alertMode")) {
      alertMode = doc["alertMode"].as<bool>();
    }
  }
}

void setup() {
  Serial.begin(115200);

  // Initialize sensors
  dht.begin();
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(LED_PIN, OUTPUT);
  pinMode(BUTTON_PIN, INPUT_PULLUP);

  setup_wifi();
  client.setServer(mqtt_server, mqtt_port);
  client.setCallback(callback);

  Serial.println("Smart Space Monitor Initialized!");
}

void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  // Check button state for alert mode toggle
  if (digitalRead(BUTTON_PIN) == LOW) {
    alertMode = !alertMode;

    // Publish alert mode change
    DynamicJsonDocument doc(1024);
    doc["alertMode"] = alertMode;
    char buffer[256];
    serializeJson(doc, buffer);
    client.publish("home/status", buffer);

    delay(500); // Debounce
  }

  // Read sensors every 2 seconds
  if (millis() - lastReadingTime >= readingInterval) {
    // Read DHT sensor
    temperature = dht.readTemperature();
    humidity = dht.readHumidity();

    // Read Ultrasonic sensor
    digitalWrite(TRIG_PIN, LOW);
    delayMicroseconds(2);
    digitalWrite(TRIG_PIN, HIGH);
    delayMicroseconds(10);
    digitalWrite(TRIG_PIN, LOW);

    duration = pulseIn(ECHO_PIN, HIGH);
    distance = duration * 0.034 / 2;

    // Prepare JSON payload
    DynamicJsonDocument doc(1024);
    doc["temperature"] = temperature;
    doc["humidity"] = humidity;
    doc["distance"] = distance;
    doc["alertMode"] = alertMode;

    char buffer[256];
    serializeJson(doc, buffer);
    client.publish(mqtt_topic, buffer);

    // LED Logic
    if (alertMode) {
      if (distance < 50) {
        digitalWrite(LED_PIN, HIGH);
        delay(100);
        digitalWrite(LED_PIN, LOW);
      }
    } else {
      int brightness = map(temperature, 20, 40, 0, 255);
      analogWrite(LED_PIN, ightness);
    }

    lastReadingTime = millis();
  }
}
```
>> ##### _Before execution, we must setup the NodeRED dashboard and the MQTT Broker on our local machine._
Step#5: SettingUp NodeRED dashboard as -

- Installing the required nodes:
    - `node-red-dashboard`
    - `node-red-contrib-ui-level`
    - `node-red-contrib-mqtt-broker`
- Designing a Flow w/ nodes & UI elements as:
    - MQTT Nodes:
        - `MQTT In`
        - `MQTT Out`
    - Functions:
        - `Alert Setup`
        - `Process Data`
    - Alert Switch
    - Debugger Node
    - Gauges:
        - `Temperature`
        - `Humidity`
    - Distance Level Node
    - Historical Graph: Line Chart
    - Notifications Node
- Configuring the nodes:
    - MQTT In: 
        - Server: 192.168.1.6
        - Port: 1883
        - Topic: `home/sensors`
        - Output: `a parsed JSON Object`
    - Dashboard UI:
        - In the Dashboard Layout, a new tab is created as **`SIRM v1`**, under which new groups are created as:
            - Envirnoment Sensors [ _Width: 12_ ]
            - Current Readings [ _Width: 12_ ]
            - Controls [ _Width: 6_ ]
            - Historical Data [ _Width: 12_ ]
    - UI Elements:
        - Temperature Gauge:
            - Group: Environment Sensors
            - Size: 4 x 4
            - Unit: °C
            - Range: 0 ~ 50
        - Humidity Gauge:
            - Group: Environment Sensors
            - Size: 4 x 4
            - Unit: %
            - Range: 0 ~ 100
        - Distance Level:
            - Group: Current Readings
            - Size: 4 x 4
            - Unit: cms
            - Range: 0 ~ 400
        - Alert Switch:
            - Group: Controls
            - Topic: `home/sensors`
        - History Chart:
            - Group: Historical Data
            - Type: Line Chart
            - Size: 12 x 6
            - Time: HH:mm:ss
            - Store: 1 Hour Data
    - Functions:
        - **`Process Data`**
            - Input: MQTT In
            - Outputs: 5
                - Alert Setup
                - Temperatue Gauge
                - Humidity Gauge
                - Distance Level
                - Histoorical Chart
            - On Message:
            ```c++
            let data = msg.payload;
            return [
                { payload: data.temperature },
                { payload: data.humidity },
                { payload: data.distance },
                { payload: data.alertMode }
            ];
            ```
        - **`Alert Setup`**
            - Input: Process Data
            - Outputs: 6
                - DeBugger Node
                - Temeperature Gauge
                - Humidity Gauge
                - Notifications
                - Alert Switch
                - MQTT Out
            - On Message:
            ```c++
            if (msg.payload.temperature > 30 || msg.payload.humidity > 70) {
            return { payload: "Alert: Environmental conditions exceeded threshold!" };
            }
            return null;
            ```
    - MQTT Out:
        - Topic: `home/sensors`

Step#6: Initializing **MQTT Broker** as our mainframe:
- In elevated CMD, `cd C:\Program Files\mosquitto` > ` mosquitto.exe -v `
- For Transmission, Open 2 Terminals:
		
		# 1st: mosquitto_sub.exe -h localhost -t home/sensors

		# 2nd: mosquitto_pub.exe -h localhost -t home/sensors -m " Light was here! "

> After setting up the software parts, _moving onto the HardWare Configuration_:

Step#7: **Connecting Sensors, LED & Push Button w/ the Espressif Board**
- DHT22
    - VCC [Voltage Common Collector]: 5v
    - SDA [Serial Data]: GPIO4 
    - ~~NC [NoConnection]~~
    - GND [Ground]: GND Pin
- HC-SR04 [UltraSonic]
    - VCC: 5v
    - Trigger: GPIO5
    - Echo: GPIO18
    - GND: GND
- Light Emitting Diode:
    - Cathode [ _`-ve`_ ]: GND
    - Anode [ _`+ve`_ ]: GPIO2
- Push Button:
    - 1L: GPIO15
    - 2L: GND
#### Finally, after all the configurations of Hardwares and Softwares, we can deploy the NodeRED Dashboard & execute the entire project of [`Smart IoT Room Monitoring`](https://wokwi.com/projects/412824490331720705).

<h1 align = 'center'> Light || DS/22-26/020 </h1>