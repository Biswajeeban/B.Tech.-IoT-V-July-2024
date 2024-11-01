# ESPressif32 DevKit v1 [ NodeMCU: MicroController Unit ]

_____________________________________________________________________

## Setting Up ESPressif32

Step#1: Inside `Arduino IDE`, Navigate to Files > Preferences > Additional Boards Manager URL > <https://dl.espressif.com/dl/package_esp32_index.json> < _paste this and click OK_

Step#2: Then head into **`BOARDS MANAGER`** > and Install esp32 by ESPressif Systems.

Step#3: Now, for establishing the connection, we would need to configure our mainframe w/ the `CP210x USB-to-UART Bridge Virtual COM Port (VCP) Driver` avaliable at [CP210x USBtoUART Driver](https://www.silabs.com/documents/public/software/CP210x_Windows_Drivers.zip).

Step#4: After a superfluous reboot, the mainframe is ready to be used w/ an ESPressif32, as in the Arduino IDE, we first select the correct COM port (_here, `COM12`_), and `ESP32 Dev Module` as the board.

* NOTE: While executing a sketch, the Board reuires to be in DOWNLOAD MODE /BOOT MODE, so for, the BOOT button is to be pressed while uploading the code onto the board, exactly post **Connecting...** for 3-4 seconds.
    * A simple way around for this redundancy, would be:
        * >Holding the `BOOT` button (GPIO0), the `EN` (Enable Pin) button is pressed for a second.
        * > Post releasing the `EN` button, finally, the `BOOT` button is let go.
    * _This would keep the Espressif32 in the bootloader mode, so it could be equipped w/ any further sketch executions._

Step#5: To blink the internal LED, the following is to be executed in the IDE:

```c++
// Define the LED pin
#define LED_PIN 2  // The onboard LED is connected to GPIO 2 on most ESP32 boards

void setup() {
  // Initialize the LED pin as an output
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  // Turn the LED on (HIGH is the voltage level)
  digitalWrite(LED_PIN, HIGH);
  // Wait for a second
  delay(1000);
  // Turn the LED off by making the voltage LOW
  digitalWrite(LED_PIN, LOW);
  // Wait for a second
  delay(1000);
}
```
###### Output: _Internal LED (blue) **blinks** @ **1s** duration._

## Configuring the ESPressif to SCAN Wireless-Fidelity Signals!

Step#6: The ESP32 can be used to scan nearby Wi-Fi signals using the following sketch:

```c++
#include "WiFi.h"

void setup() {
  Serial.begin(115200);

  // Set WiFi to station mode and disconnect from an AP if it was previously connected.
  WiFi.mode(WIFI_STA);
  WiFi.disconnect();
  delay(100);

  Serial.println("Setup done");
}

void loop() {
  Serial.println("Scan start");

  // WiFi.scanNetworks will return the number of networks found.
  int n = WiFi.scanNetworks();
  Serial.println("Scan done");
  if (n == 0) {
    Serial.println("no networks found");
  } else {
    Serial.print(n);
    Serial.println(" networks found");
    Serial.println("Nr | SSID                             | RSSI | CH | Encryption");
    for (int i = 0; i < n; ++i) {
      // Print SSID and RSSI for each network found
      Serial.printf("%2d", i + 1);
      Serial.print(" | ");
      Serial.printf("%-32.32s", WiFi.SSID(i).c_str());
      Serial.print(" | ");
      Serial.printf("%4ld", WiFi.RSSI(i));
      Serial.print(" | ");
      Serial.printf("%2ld", WiFi.channel(i));
      Serial.print(" | ");
      switch (WiFi.encryptionType(i)) {
        case WIFI_AUTH_OPEN:            Serial.print("open"); break;
        case WIFI_AUTH_WEP:             Serial.print("WEP"); break;
        case WIFI_AUTH_WPA_PSK:         Serial.print("WPA"); break;
        case WIFI_AUTH_WPA2_PSK:        Serial.print("WPA2"); break;
        case WIFI_AUTH_WPA_WPA2_PSK:    Serial.print("WPA+WPA2"); break;
        case WIFI_AUTH_WPA2_ENTERPRISE: Serial.print("WPA2-EAP"); break;
        case WIFI_AUTH_WPA3_PSK:        Serial.print("WPA3"); break;
        case WIFI_AUTH_WPA2_WPA3_PSK:   Serial.print("WPA2+WPA3"); break;
        case WIFI_AUTH_WAPI_PSK:        Serial.print("WAPI"); break;
        default:                        Serial.print("unknown");
      }
      Serial.println();
      delay(10);
    }
  }
  Serial.println("");

  // Delete the scan result to free memory for code below.
  WiFi.scanDelete();

  // Wait a bit before scanning again.
  delay(5000);
}
```
###### Output: _n Networks Found! ; Light's Space Stone ; Airtel77_

# Light || DS/22-26/020