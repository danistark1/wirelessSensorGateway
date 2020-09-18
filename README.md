# Source
https://www.hackster.io/andersot72/acurite-sensors-through-to-grafana-dashboard-7a1433
# weatherStation
A Weather Station project using RPI, a software defined radio module with acurite wireless sensors.
![Grafana Station](https://github.com/danimajdalani/weatherStation/blob/master/img/station_grafana.png)
# Hardware

- Accurite06002M wireless temperature and humidity sensor(433MHZ).
- NooElec Software defined radio(NTL SDR) to read the sensor's 433MHZ signal.
![SDR](https://github.com/danimajdalani/weatherStation/blob/master/img/sdr.png)
- A raspberry-pi

# Software
- Node-Red with MySQL palette added. (menu->manage palette).
- MQTT	
- rtl_433 Generic data receiver, mainly for the 433.92 MHz, 868 MHz (SRD), 315 MHz, and 915 MHz ISM bands	
- Grafana

# Update the pi
- sudo apt-get update
- sudo apt-get upgrade -y
- sudo reboot

# RTL433 & SoapySDR (build the software for rtl_433)

- git clone https://github.com/merbanan/rtl_433.git
- git clone https://github.com/pothosware/SoapySDR.git
- cd SoapySDR/
- git pull origin master
- mkdir build
- cd build
- cmake ..
- sudo make install
- SoapySDRUtil --info

# Reading from the Sensor

- rtl_433 -v
You should see something like 
`Found 1 device(s)
trying device  0:  Realtek, RTL2838UHIDIR, SN: 00000001`

# Setup Node-Red
Node-red comes installed with Rpi, go ahead and update it.

`bash <(curl -sL https://raw.githubusercontent.com/node-red/raspbian-deb-package/master/resources/update-nodejs-and-nodered)
cd ~/.node-red
npm rebuild
npm ls --depth=0
npm outdated
npm update
`
*Now that Node-RED is updated lets install and configure mosquitto aka MQTT*

# Setup MQTT

`>>sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install mosquitto mosquitto-clients
sudo systemctl enable mosquitto.service
sudo systemctl start mosquitto.service
sudo systemctl status mosquitto.service`

*Now send the rtl_433 output to as MQTT messages - -R 40(is the device type of the acurite sensor I am using)*

- rtl_433 -M notime -F json -R 40 | mosquitto_pub -t home/acurite -l

- in another terminal run the belelow

- mosquitto_sub -t home/acurite


*access your node-red from the top left menu under programming. Node-red uses port 1880*
Import the file under node-red

# Setup Grafana
under /src/grapfana bckup
You can access grafana http://localhost:3000

# Setup MySQL DB
From node-red, go to manage palette, add MySQL.

Use this function to read data from payload before inserting to MySQL db.

`var date = new Date();
msg.topic="INSERT INTO sensor_data (room,temperature,humidity,station_id,insert_date_time) VALUES ('basement',?,?,?,?)";
msg.payload=[msg.payload.temperature,msg.payload.humidity,msg.payload.id,date];
return msg;`

**Humidity Query**

`SELECT
  insert_date_time AS "time",
  humidity
FROM sensor_data
WHERE
  station_id = 6126
ORDER BY insert_date_time`

**Temperature Query**

`SELECT
  insert_date_time AS "time",
  temperature
FROM sensor_data
WHERE
  station_id = 6126
ORDER BY insert_date_time`
![Grafana Sensor](https://github.com/danimajdalani/weatherStation/blob/master/img/grafana-sensor.png)

# Loading Node-Red flow
backup under /src/node-red flow
To start node-red, from the command line, type node-red.
You can then access node-red's dashbaord from http://localhost:1880

# Starting the station
- node-red start
- rtl_433 -M notime -F json -R 40 | mosquitto_pub -t home/acurite -l
- mosquitto_sub -t home/acurite


# Running RTL433 on boot

- sudo apt-get install supervisor
- sudo nano rtl433.sh 
in the file above paste

```
#!/bin/bash
rtl_433 -M notime -F json -R 40 | mosquitto_pub -t home/acurite -l
```
- sudo chmod +x rtl433.sh
- sudo nano /etc/supervisor/supervisord.conf

```
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
