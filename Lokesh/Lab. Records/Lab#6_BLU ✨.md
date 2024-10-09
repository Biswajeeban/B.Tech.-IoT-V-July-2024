# Working w/ Button, LED & an UltraSonic [ HC-SR04 ]

_____________________________________________________

## Button: Held or, Released?

Step#1: Connect 2 jumper wires _diagonally_ to the Button.

Step#2: One wire to Digital Pin [ _here, 2_ ], and the other wire to **`GND`**.

Step#3: After configuring the ` Arduino UNO R3 `, this sketch is to be uploaded and executed in the IDE:

```c++
const int buttonPin = 2;  // Pin where the button is connected

void setup() {
  Serial.begin(115200);              // Initialize serial communication at 115200 baud
  pinMode(buttonPin, INPUT_PULLUP);  // Set the button pin as input with internal pull-up resistor
             }

void loop() {
    int buttonState = digitalRead(buttonPin);  // Read the state of the button

    if (buttonState == LOW) {                  // Check if the button is pressed
    Serial.println("Button Held!");       // Print message to Serial Monitor
                            }
    else  {
    Serial.println("Button Released!");   // Print message to Serial Monitor
          }

  delay(500);  // Add a small delay to debounce the button
            }
```

###### Output: _Button Held! ; Button Released!_

### Button + External LED: Blinks?

Step#4: Connect an LED to the board as Anode [+ve] to a digital pin ( _here, 13_ ), and Cathode [ -ve, _flat side_ ], to **`GND`**.

Step#5: After configuring the **` Button `** & the `LED`, this sketch is to be uploaded and executed in the IDE:

```c++
const int buttonPin = 2;   // Pin where the button is connected
const int ledPin = 13;     // Pin where the LED is connected (built-in LED for most Arduino boards)

void setup() {
  pinMode(buttonPin, INPUT_PULLUP);  // Set the button pin as input with internal pull-up resistor
  pinMode(ledPin, OUTPUT);           // Set the LED pin as output
}

void loop() {
  int buttonState = digitalRead(buttonPin);  // Read the state of the button

  if (buttonState == LOW) {                  // Check if the button is pressed
    digitalWrite(ledPin, HIGH);              // Turn the LED on
    delay(500);                              // Wait for 500 milliseconds
    digitalWrite(ledPin, LOW);               // Turn the LED off
    delay(500);                              // Wait for 500 milliseconds
  } else {
    digitalWrite(ledPin, LOW);               // Ensure the LED stays off when the button is not pressed
  }
}
```

###### Output: _External LED blinks @ 5ms when the button is held._

## UltraSonic [ HC-SR04 ]: Measures Distance 🤔

Step#1: Connect 4 jumper wires to the UltraSonic Sensor as [`VCC`: 5v], [`Trigger`: 9], [`Echo`: 8], & **`GND`**.

Step#2: After configuring the **HC-SR04** w/ **UNO R3**, this sketch is to be uploaded and executed in the IDE:

```c++
#define PIN_TRIG 9 // Define the pin for the trigger
#define PIN_ECHO 8 // Define the pin for the echo

void setup() {
  Serial.begin(9600); // Initialize serial communication at 9600 baud
  pinMode(PIN_TRIG, OUTPUT); // Set the trigger pin as output
  pinMode(PIN_ECHO, INPUT);  // Set the echo pin as input
}

void loop() {
  // Start a new measurement:
  digitalWrite(PIN_TRIG, HIGH);     // Set the trigger pin high
  delayMicroseconds(10);            // Wait for 10 microseconds
  digitalWrite(PIN_TRIG, LOW);      // Set the trigger pin low

  // Read the result:
  int duration = pulseIn(PIN_ECHO, HIGH); // Read the duration of the pulse from the echo pin

  // Calculate distance in centimeters:
  Serial.print("Distance in CM: ");
  Serial.println(duration / 58);    // Print the distance in centimeters

  // Calculate distance in inches:
  Serial.print("Distance in inches: ");
  Serial.println(duration / 148);   // Print the distance in inches

  delay(1000); // Wait for 1 second before taking the next measurement
}
```

###### Output: _Distance in CM: 121 ; Distance in inches: 47_

## HC-SR04 + PushButton

Step#3: Now, a button can be connected to Digital `2` and diagonally to **`GND`**.

Step#4: After configuring the Button & the sensor, this sketch is to be uploaded and executed in the IDE:

```c++
#define PIN_TRIG 9 // Define the pin for the trigger
#define PIN_ECHO 8 // Define the pin for the echo
#define buttonPin 2 // Define the pin for the button

void setup() {
  Serial.begin(9600);                // Initialize serial communication at 9600 baud
  pinMode(PIN_TRIG, OUTPUT);         // Set the trigger pin as output
  pinMode(PIN_ECHO, INPUT);          // Set the echo pin as input
  pinMode(buttonPin, INPUT_PULLUP);  // Set the button pin as input with internal pull-up resistor
}

void loop() {
  int buttonState = digitalRead(buttonPin); // Read the state of the button

  if (buttonState == LOW) { // Check if the button is pressed
    // Start a new measurement:
    digitalWrite(PIN_TRIG, HIGH);     // Set the trigger pin high
    delayMicroseconds(10);            // Wait for 10 microseconds
    digitalWrite(PIN_TRIG, LOW);      // Set the trigger pin low

    // Read the result:
    int duration = pulseIn(PIN_ECHO, HIGH); // Read the duration of the pulse from the echo pin

    // Calculate distance in centimeters:
    Serial.print("Distance in CM: ");
    Serial.println(duration / 58);    // Print the distance in centimeters

    // Calculate distance in inches:
    Serial.print("Distance in Inches: ");
    Serial.println(duration / 148);   // Print the distance in inches
  } else {
    Serial.println("Button Released!"); // Print message to Serial Monitor
  }

  delay(500); // Add a small delay to debounce the button
}
```

###### Output: _Button Released! ... ; Distance in  CM: 24 ; Distance in Inches: 9_

# Combining PushButton + UltraSonic Sensor + External LED + DHT Sensor

Step#1:  Get 9 Jumper wires [ `M2F` ], and connect them as:

* HC-SR04:
  * Trigger [ _Digital 9_ ]
  * Echo [ _Digital 8_ ]
  * VCC [ _5v_ ]
  * `GND`
* DHT 11:
  * VCC [ _3.3v_ ]
  * Data [ _Digital 4_ ]
  * `GND`
* Button:
  * Corner 1 [ _Digital 2_ ]
  * Diagonal Corner 4 [ `GND` ]
* External LED:
  * Anode [ +ve, _Digital 13_ ]
  * Cathode [ -ve, `GND`]  

Step#2: After configuring the entire Arduino Uno R3 with proper wire connections, the following could be executed in the IDE:

```c++
#include <DHT.h>

#define DHTPIN 4         // Pin where the DHT sensor is connected
#define DHTTYPE DHT11    // Define the type of DHT sensor (DHT11 in this case)
#define PIN_TRIG 9       // Define the pin for the ultrasonic sensor trigger
#define PIN_ECHO 8       // Define the pin for the ultrasonic sensor echo
#define BUTTON_PIN 2     // Define the pin for the button
#define LED_PIN 13       // Define the pin for the LED

DHT dht(DHTPIN, DHTTYPE);

void setup() {
  Serial.begin(9600);                // Initialize serial communication at 9600 baud
  dht.begin();                       // Initialize the DHT sensor
  pinMode(PIN_TRIG, OUTPUT);         // Set the ultrasonic sensor trigger pin as output
  pinMode(PIN_ECHO, INPUT);          // Set the ultrasonic sensor echo pin as input
  pinMode(BUTTON_PIN, INPUT_PULLUP); // Set the button pin as input with internal pull-up resistor
  pinMode(LED_PIN, OUTPUT);          // Set the LED pin as output
}

void loop() {
  int buttonState = digitalRead(BUTTON_PIN); // Read the state of the button

  if (buttonState == LOW) { // Check if the button is pressed
    // Ultrasonic Sensor Measurement:
    digitalWrite(PIN_TRIG, HIGH); // Set the trigger pin high
    delayMicroseconds(10);        // Wait for 10 microseconds
    digitalWrite(PIN_TRIG, LOW);  // Set the trigger pin low

    int duration = pulseIn(PIN_ECHO, HIGH); // Read the duration of the pulse from the echo pin
    float distanceCm = duration / 58.0;     // Calculate the distance in centimeters

    Serial.print("Distance in CM: ");
    Serial.println(distanceCm);

    if (distanceCm < 20) { // Example condition: if distance is less than 20 cm
      digitalWrite(LED_PIN, HIGH); // Turn the LED on
    } else {
      digitalWrite(LED_PIN, LOW); // Turn the LED off
    }

    // DHT Sensor Measurement:
    float humidity = dht.readHumidity();
    float temperature = dht.readTemperature();

    if (isnan(humidity) || isnan(temperature)) {
      Serial.println("Failed to read from DHT sensor!");
    } else {
      Serial.print("Humidity: ");
      Serial.print(humidity);
      Serial.print(" %\t");
      Serial.print("Temperature: ");
      Serial.print(temperature);
      Serial.println(" *C");
    }

  } else {
    Serial.println("Button Released!");
  }

  delay(1500); // Delay for 1.5 seconds before the next loop
}
```

###### Output: _Button Released!; Distance in CM: 18.45 -- Humidity: 79.00 % -- Temperature: 30.80 *C [ LED TURNED ON ] ; Distance in CM: 24.28 -- Humidity: 65.00 % -- Temperature: 30.80 *C [ LED TURNED OFF ] ; Button Released!_

# Light || DS/22-26/020