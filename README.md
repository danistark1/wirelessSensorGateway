![Weather Station Logo](https://github.com/danistark1/weatherStation/blob/master/img/weatherLogo.png)


## Table of contents ##
1. [Intro](#weatherstation "weatherStation")
2. [Hardware](#hardware "Hardware")
3. [Software](#software "Software")
4. [Update the pi](#update-the-pi "Update the pi")
5. [Setup SoapySDR](#setup-soapysdr "Setup SoapySDR")
6. [Setup RTL_433](#setup-rtl_433 "Setup RTL_433")
7. [Reading Sensor Data](#reading-sensor-data "Reading Sensor Data")
8. [Setup Node-Red](#setup-node-red "Setup Node-Red")
9. [Setup MQTT](#setup-mqtt "Setup MQTT")
10. [Setup Grafana](#setup-grafana "Setup Grafana")
11. [Setup MySQL DB](#setup-mysql-db "Setup MySQL DB")
12. [Node Red Using REST APIs](#node-red-using-rest-apis "Node Red Using REST APIs")
13. [Loading Node-Red flow](#loading-node-red-flow "Loading Node-Red flow")
14. [Starting the station](#starting-the-station "Starting the station")
15. [Running RTL433 on boot](#running-rtl433-on-boot "Running RTL433 on boot")
16. [Running node-red on boot](#running-node-red-on-boot "Running node-red on boot")

# weatherStation

A Weather Station project using RPI, a software defined radio module with acurite wireless sensors.
![Grafana Station](https://github.com/danistark1/weatherStation/blob/master/img/weatherStationMain.png)



Weather Station REST APIs project at https://github.com/danistark1/weatherStationApiSymfony


# Hardware

- Accurite06002M wireless temperature and humidity sensor(433MHZ)
- NooElec Software defined radio(NTL SDR) to read the sensor's 433MHZ signal
![SDR](https://github.com/danimajdalani/weatherStation/blob/master/img/sdr.png)
- A raspberry-pi running raspbian https://www.raspberrypi.org/software/

# Software

- Node-Red with MySQL palette added(Not required is using the API). (menu->manage palette).
- MQTT	
- rtl_433 Generic data receiver, mainly for the 433.92 MHz, 868 MHz (SRD), 315 MHz, and 915 MHz ISM bands	
- Grafana

# Update the pi

- sudo apt-get update
- sudo apt-get upgrade -y
- sudo reboot

# Setup SoapySDR

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

# Setup RTL_433

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

# Setup Node-Red

- sudo apt install build-essential (installs required npm modules)
- bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)
- sudo systemctl start nodered (add node-red as a service to start on boot)
- sudo systemctl status nodered (to check the service status)


*Now that Node-RED is installed, install and configure mosquitto aka MQTT*

# Setup MQTT

- sudo apt-get update
- sudo apt-get upgrade -y
- sudo apt-get install mosquitto mosquitto-clients
- sudo systemctl enable mosquitto.service
- sudo systemctl start mosquitto.service
- sudo systemctl status mosquitto.service`

*Now send the rtl_433 output to as MQTT messages - -R 40(is the device type of the acurite sensor I am using)*

- rtl_433 -M notime -F json -R 40 | mosquitto_pub -t home/acurite -l

- in another terminal run the belelow

- mosquitto_sub -t home/acurite


*access your node-red from the top left menu under programming. Node-red uses port 1880*
Import the file under node-red

# Setup Grafana

- wget https://dl.grafana.com/oss/release/grafana_7.2.0_armhf.deb
- sudo dpkg -i grafana_7.2.0_armhf.deb (check latest version)
- sudo systemctl enable grafana-server (to start on boot)
- sudo systemctl start grafana-server

under /src/grapfana bckup

You can access grafana http://localhost:3000

# Setup MySQL DB

***(Not required if you are using weatherStation API at https://github.com/danistark1/weatherStationApiSymfony)***

From node-red, go to manage palette, add MySQL.

Use this function to read data from payload before inserting to MySQL db.

```JS
var date = new Date();
msg.topic="INSERT INTO sensor_data (room,temperature,humidity,station_id,insert_date_time) VALUES ('basement',?,?,?,?)";
msg.payload=[msg.payload.temperature,msg.payload.humidity,msg.payload.id,date];
return msg;
```
# Node Red Using REST APIs

To read/write to mySQL db, you can use https://github.com/danistark1/weatherStationApiSymfony which uses REST APIs to read/write to the db.

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
![Grafana Sensor](https://github.com/danimajdalani/weatherStation/blob/master/img/grafana-sensor.png)

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
- sudo nano /etc/supervisor/supervisord.conf

```JS
[program:rtl_433]
command=/home/pi/rtl433.sh
autostart=true
autorestart=true
stderr_logfile=/var/log/long.err.log
stdout_logfile=/var/log/long.out.log
```
do the same for mosquitto service

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
