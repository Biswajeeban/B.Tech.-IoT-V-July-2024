# Configuring MQTT in our Local Machine
____________________________________________________

## In SystemOS [ Windows11 ]:
	
	- Installed Mosquitto as a Service from Official Eclipse Page [ https://mosquitto.org/download/ ].

		- This allows the MQTT Broker to run automatically in the background.

	- Added mosquittio.exe to the System Environment Variables PATH [ ' C:\Program Files\mosquitto ' ], which allows us to use MQTT commands directly in the Command Prompt or, Terminal.
	
	> Starting @ boot byDefault:
		net start mosquitto
	> Stopping:
		In Elevated CMD > net stop mosquitto 

	> For Transmission: Navigate to [ cd C:/Program Files/mosquitto ]
		mosquitto.exe -v
		// -v is a Verbose Output flag, that enables us to see the backend processes, log messages, that'd help us to debug whenever necessary.

## In Linux [ WSL*: Ubuntu 22.04 LTS ]:

	-  In Terminal > wsl --install -d Ubuntu-22.04 > \ E / N \ T / E \ R /
	- Restart the machine, and Launch Ubuntu 22.04
		- $sudo apt update 
		- $sudo apt install mosquitto mosquitto-clients
	> Starting mosquitto services:
		- $sudo systemctl ( enable /start ) mosquitto

	- Mosquitto Broker Service Status can be checked here: 
		- $sudo systemctl status mosquitto
	
	- Once verified service status, transmission can be carried on.
	
	> Stopping mosquitto services:
		- $sudo systemctl stop mosquitto
*Windows Subsystem for Linux
## Testing MQTT Services [ Message Transmission: WinOS11 + Ubuntu 22.04 ]:

	- Open 2 Terminals:
		
		# 1st: mosquitto_sub.exe -h localhost -t test/topic

		# 2nd: mosquitto_pub.exe -h localhost -t test/topic -m " Light was here! "
# Light || DS/22-26/020