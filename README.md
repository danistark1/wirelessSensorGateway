# weatherStation
A Weather Station project using RPI, a software defined radio module with acurite wireless sensors.

# Tools used

- Accurite06002M wireless temperature and humidity sensor(433MHZ).

![Sensor](https://github.com/danimajdalani/weatherStation/blob/master/img/sensor.png)
- NooElec Software defined radio(NTL SDR) to read the sensor's 433MHZ signal.
![SDR](https://github.com/danimajdalani/weatherStation/blob/master/img/sdr.png)
- A raspberry-pi

# Software used
- Node-Red
- MQTT	
- rtl_433 Generic data receiver, mainly for the 433.92 MHz, 868 MHz (SRD), 315 MHz, and 915 MHz ISM bands	
- Grafana
- InfluxDB - Time Series Database
- Chronograf (optional)

# Update the pi
- >>sudo apt-get update
- >>sudo apt-get upgrade -y
- >>sudo reboot

# RTL433 & SoapySDR (build the software for rtl_433)

- >>git clone https://github.com/merbanan/rtl_433.git
- >>git clone https://github.com/pothosware/SoapySDR.git
- >>cd SoapySDR/
- >>git pull origin master
- >>mkdir build
- >>cd build
- >>cmake ..
- >>sudo make install
- >>SoapySDRUtil --info

# Reading from the Sensor
>>rtl_433 -v
You should see something like 
>>Found 1 device(s)
>>trying device  0:  Realtek, RTL2838UHIDIR, SN: 00000001

# Setup Node-Red
Node-red comes installed with Rpi, go ahead and update it.

>>bash <(curl -sL https://raw.githubusercontent.com/node-red/raspbian-deb-package/master/resources/update-nodejs-and-nodered)
>>cd ~/.node-red
>>npm rebuild
>>npm ls --depth=0
>>npm outdated
>>npm update

*Now that Node-RED is updated lets install and configure mosquitto aka MQTT*

# Setup MQTT

>>sudo apt-get update
>>sudo apt-get upgrade -y
>>sudo apt-get install mosquitto mosquitto-clients
>>sudo systemctl enable mosquitto.service
>>sudo systemctl start mosquitto.service
>>sudo systemctl status mosquitto.service

*Now send the rtl_433 output to as MQTT messages - -R 40(is the device type of the acurite sensor I am using)*

- >>rtl_433 -M notime -F json -R 40 | mosquitto_pub -t home/acurite -l

- in another terminal run the belelow

- >>mosquitto_sub -t home/acurite

*access your node-red from the top left menu under programming. Node-red uses port 1880*
Import the file under node-red
# Setup Grafana
under /src/grapfana bckup
You can access grafana http://localhost:3000
# Setup InfluxDB
TODO 
# Setup Chronograf
# Loading Node-Red flow
backup under /src/node-red flow
To start node-red, from the command line, type node-red.
You can then access node-red's dashbaord from http://localhost:1880
# Starting the station
node-red start
rtl_433 -M notime -F json -R 40 | mosquitto_pub -t home/acurite -l
mosquitto_sub -t home/acurite
