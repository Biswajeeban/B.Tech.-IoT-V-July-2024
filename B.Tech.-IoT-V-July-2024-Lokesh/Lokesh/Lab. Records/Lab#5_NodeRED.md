# Real-Time DHT11 Data on Node-RED Dashboard
____________________________________________________

## Install Node.js :
	
	- Installed NodeJS from Official Eclipse Page [ https://nodejs.org/en/download/package-manager ].

	- Added node.js to the System Environment Variables PATH [ ' C:\Users\Lokesh Patra\AppData\Roaming\npm ' ], which allows us to use npm commands directly in the Command Prompt or, Terminal.

## Installing & Initialising NodeRED:

    - Open Node.js > npm install node-red-dashboard
    - [PostInstallation] > Elevated CMD: node-red
                         > In Client Application, browsed localhost:1880 [ Accessing NodeRED ]
    - Inside the NodeRED window, a flow was created w/ the nodes as:
            > SERIAL-IN ( Arduino Uno R3 Board )
            > DEBUGGER 
            > DHT FUNCTION
            > 2 GAUGES ( Humidity & Temperature )
    - Serial In Node: Configured it to read from the correct serial port where my Arduino is connected (e.g., COM11) > Set the baud rate to 9600.
    - Configure the DHT Function as:

        var m = msg.payload.split(',');

        if (m.length === 2) {
            var H = { payload: parseFloat(m[0]) };
            var T = { payload: parseFloat(m[1]) };

            return [H, T];
        } else {
            return null;
                }
    - Adjusting Gauge Nodes:
        > Humidity:
        - Title as "Humidity".
        - Value format as `{{value}}%`.
        - Minimum value to 0 and the maximum to 100.

        > Temperatue:
        - Title as ' Temperature '
        - Value format as {{value}}°C.

    - Ensure that Humidity & Temperature are in the same group.

## Deployment: 

    - Uploaded DHT11 /22 Sketch to the Arduino Board through its IDE:

        #include <DHT.h>

        #define DHTPIN 3
        #define DHTTYPE DHT11

        DHT dht(DHTPIN, DHTTYPE);

        void setup() {
            Serial.begin(9600);
            dht.begin();
                    }   

        void loop()  {
        float H = dht.readHumidity();
        float T = dht.readTemperature();

        if (isnan(H) || isnan(T)) {
            Serial.println("Failed to read from DHT sensor!");
        } else {
            Serial.println(String(H) + "," + String(T));
            } 

        delay(2000);
        }

    - After uploading this sketch, close the IDE.
    - Deploy the flow in NodeRED.
    - Check the Dashboard in the upper-right corner, for the Humidity and Temperature Gauge.