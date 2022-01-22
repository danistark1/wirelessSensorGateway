
 <p align="center">
  <b>
   🌧 Wireless Sensor Gateway Project 🌧
  </b>
 </p>
<p align="center">
  <img src="https://github.com/danistark1/weatherStation/blob/master/img/weatherLogo.png" />
</p>

## 📔 What is it? ##

A gateway for recording wireless sensor readings using a raspberry pi in a MySQL database using a software defined radio (SDR), MQTT, acurite wireless temp/humidity sensors(other sensors can be used, check rtl_433 repo for support/config) & rtl_433 Generic data receiver.

## 💢 Table of contents ##
- [Devices](#devices "Devices")
- [Software](#software "Software")
- [RasPi Setup](#raspi-setup "Raspi Setup")
- [SoapySDR Setup ](#soapysdr-setup "SoapySDR Setup")
- [RTL_433 Setup](#rtl_433-setup "RTL_433 Setup")
- [Reading Sensor Data](#reading-sensor-data "Reading Sensor Data")
- [Node-Red Setup ](#node-red-setup "Node-Red Setup")
- [MQTT Setup](#mqtt-setup "MQTT Setup")
- [Running Multiple Devices with Different Frequencies](#running-multiple-devices-with-different-frequencies "Running Multiple Devices with Different Frequencies")
- [Grafana Setup](#grafana-setup "Grafana Setup")
- [MySQL DB Setup](#mysql-db-setup "MySQL DB Setup")
- [Node Red Using APIs](#node-red-using-apis "Node Red Using APIs")
- [Loading Node-Red flow](#loading-node-red-flow "Loading Node-Red flow")
- [Starting the station](#starting-the-station "Starting the station")
- [Running RTL433 on boot](#running-rtl433-on-boot "Running RTL433 on boot")
- [Running node-red on boot](#running-node-red-on-boot "Running node-red on boot")

![Grafana Station](https://github.com/danistark1/weatherStation/blob/master/img/weatherStationMain.png)

# Sensor Gateway APIs (Required)

**Frameworks**

- Symfony https://github.com/danistark1/sensorGatewayAPISymf
- Laravel v8.60.0 (Latest, in-progress) https://github.com/danistark1/sensorGatewayAPILrv

# Devices

- Accurite06002M wireless temperature and humidity sensor(433MHZ)
- NooElec Software defined radio(NTL SDR) to read the sensor's 433MHZ signal
![SDR](https://github.com/danimajdalani/weatherStation/blob/master/img/sdr.png)
- A raspberry-pi running raspbian https://www.raspberrypi.org/software/

# Software

- Node-Red with MySQL palette added(Not required is using the API). (menu->manage palette).
- MQTT	
- rtl_433 Generic data receiver, mainly for the 433.92 MHz, 868 MHz (SRD), 315 MHz, and 915 MHz ISM bands	
- Grafana

# Raspi Setup

- sudo apt-get update
- sudo apt-get upgrade -y
- sudo reboot

# SoapySDR Setup

- git clone https://github.com/merbanan/rtl_433.git
- git clone https://github.com/pothosware/SoapySDR.git
- cd SoapySDR/
- git pull origin master
- mkdir build
- cd build
- sudo apt-get install cmake
- cmake ..
- sudo make install
- SoapySDRUtil --info

# RTL_433 Setup

- sudo apt-get install libtool libusb-1.0.0-dev librtlsdr-dev rtl-sdr build-essential autoconf cmake pkg-config
- cd rtl_433/
- mkdir build
- cd build
- cmake ..
- sudo make install
- sudo ldconfig

# Reading Sensor Data

- rtl_433 -v
You should see something like 

`Found 1 device(s)
trying device  0:  Realtek, RTL2838UHIDIR, SN: 00000001`

# Node-Red Setup

- sudo apt install build-essential (installs required npm modules)
- bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
- sudo systemctl start nodered (add node-red as a service to start on boot)
- sudo systemctl status nodered (to check the service status)


*Now that Node-RED is installed, install and configure mosquitto aka MQTT*

# MQTT Setup

- sudo apt-get update
- sudo apt-get upgrade -y
- sudo apt-get install mosquitto mosquitto-clients
- sudo systemctl enable mosquitto.service
- sudo systemctl start mosquitto.service
- sudo systemctl status mosquitto.service

*Now send the rtl_433 output to as MQTT messages - -R 40(is the device type of the acurite sensor I am using)*

- rtl_433 -M notime -F json -R 40 | mosquitto_pub -t home/acurite -l

- in another terminal run the belelow

- mosquitto_sub -t home/acurite


*access your node-red from the top left menu under programming. Node-red uses port 1880*
Import the file under node-red

# Running Multiple Devices with Different Frequencies

**Frreuqncy 915 MHZ + 433 MHZ, Devices 142 & 40 **

Setting the hop to 80 seconds, to switch the frequency

- rtl_433 -f 915M -f 433920000 -M notime -F json -R 142 -R 40 -H 80 -E hop | mosquitto_pub -t home/sensors -l

**Multiples devices, same frequency**

- rtl_433 -F json -f 433920000 -R 103 -R 40 | mosquitto_pub -t home/sensors -l

# Grafana Setup

- wget https://dl.grafana.com/oss/release/grafana_7.2.0_armhf.deb
- sudo dpkg -i grafana_7.2.0_armhf.deb (check latest version)
- sudo systemctl enable grafana-server (to start on boot)
- sudo systemctl start grafana-server

under /src/grapfana bckup

You can access grafana http://localhost:3000

# MySQL DB Setup

***(Not required if you are using SensorGateway API at https://github.com/danistark1/weatherStationApiSymfony)***

From node-red, go to manage palette, add MySQL.

Use this function to read data from payload before inserting to MySQL db.

```JS
var date = new Date();
msg.topic="INSERT INTO sensor_data (room,temperature,humidity,station_id,insert_date_time) VALUES ('basement',?,?,?,?)";
msg.payload=[msg.payload.temperature,msg.payload.humidity,msg.payload.id,date];
return msg;
```
# Node Red Using APIs

To read/write to mySQL db, you can use https://github.com/danistark1/wirelessSensorGatewayAPI which uses APIs.

**Sample Node Red Http Request Flow**

![Node Red HTTP Request](https://github.com/danistark1/wirelessSensorGateway/blob/master/img/nodeRedHttpRequest.png)

**Payload**

```js
return {
    payload: {
        id: msg.payload.id,
        temperature: msg.payload.temperature_C,
        humidity: msg.payload.humidity,
        room: 'basement',
        station_id: 3026
    }
};
```

**Humidity Query**

```sql

SELECT
  insert_date_time AS "time",
  humidity
FROM sensor_data
WHERE
  station_id = 6126
ORDER BY insert_date_time`
```

**Temperature Query**

```sql

SELECT
  insert_date_time AS "time",
  temperature
FROM sensor_data
WHERE
  station_id = 6126
ORDER BY insert_date_time`
```
![Grafana Sensor](https://github.com/danistark1/wirelessSensorGateway/blob/master/img/grafana-sensor.png)

# Loading Node-Red flow

- backup under /src/node-red flow
- CLI: Start node-red  from the command line, type node-red
- URL: You can then access node-red's dashbaord from http://localhost:1880

# Starting the station

***(automated last step)***

- node-red start
- rtl_433 -M notime -F json -R 40 | mosquitto_pub -t home/acurite -l
- mosquitto_sub -t home/acurite


# Running RTL433 on boot

- sudo apt-get install supervisor
- sudo nano rtl433.sh 
in the file above paste

```JS
#!/bin/bash
rtl_433 -M notime -F json -R 40 | mosquitto_pub -t home/acurite -l
```
- sudo chmod +x rtl433.sh
- sudo nano /etc/supervisor/conf.d/rtl_433.conf

```yaml
[program:rtl_433service]
command=/home/pi/Desktop/weatherStationFiles/rtl_433service.sh  --sleep=10 --tries=3 --daemon
autostart=true
autorestart=true
stderr_logfile=/var/log/long.err.log
stdout_logfile=/var/log/long.out
startsecs=0
exitcodes=0


[program:rtl_433mosquito]
command=/home/pi/Desktop/weatherStationFiles/rtl_433mosquito.sh
autostart=true
autorestart=true
stderr_logfile=/var/log/long.err.log
stdout_logfile=/var/log/long.out
```
do the same for mosquitto servi

**Troublshooting supervisor**

- sudo supervisorctl (check the running services)
- sudo service supervisor restart (or reload)

# Running node-red on boot

- sudo npm install -g pm2
- which node-red (to get current path)
- pm2 start /usr/bin/node-red --node-args="--max-old-space-size=128"

to access the logs

- pm2 info node-red
- pm2 logs node-red
to run on startup
- pm2 save
- pm2 startup
