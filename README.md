# GambiConf21

## Mosquitto

```
sudo apt install mosquitto

sudo touch /etc/mosquitto/passwd

sudo mosquitto_passwd /etc/mosquitto/passwd gambiconf eL6HwXyW

sudo vim /etc/mosquitto/mosquitto.conf

password_file /etc/mosquitto/passwd
allow_anonymous false


sudo systemctl restart mosquitto

sudo tail -f /var/log/mosquitto/mosquitto.log

1638662488: Config loaded from /etc/mosquitto/mosquitto.conf.
1638662488: Opening ipv4 listen socket on port 1883.
1638662488: Opening ipv6 listen socket on port 1883.
1638662541: New connection from 192.168.88.203 on port 1883.
```


## Influx DB

```
wget -qO- https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/os-release
echo "deb https://repos.influxdata.com/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

sudo apt install influxdb chronograf

sudo systemctl unmask influxdb.service
sudo systemctl enable influxdb.service
sudo systemctl start influxdb.service
```


influx
```
create database gambiconf
use gambiconf

create user nodered with password 'aqch63RF' with all privileges
grant all privileges on gambiconf to nodered

show users

user    admin
----    -----
nodered true
```



## Node-RED
```
bash <(curl -sL https://raw.githubusercontent.com/node-red/linux-installers/master/deb/update-nodejs-and-nodered)

node-red admin init
sudo systemctl enable nodered.service
sudo systemctl start nodered.service
```

Palette:
node-red-contrib-influxdb



## InfluxDB Web UI
http://192.168.0.2:8888

## Node-RED Web UI
http://192.168.0.2:1880



## Espruino Code

```
// Config values
const wifiSSID = "GambiConf";
const wifiPWD = "gambiconf";
const mqttServer = "192.168.0.2";
const mqttOptions = {
  port: 1883,
  username: "gambiconf",
  password: "eL6HwXyW",
  client_id : "espruino-pico",

  keep_alive: 60,
  clean_session: true,
  protocol_name: "MQTT",
  protocol_level: 4,
};
const mqttTopic = "weather";



// Setup ESP8266 on Serial2
Serial2.setup(9600, { rx: A3, tx : A2 });
var wifi = require("ESP8266WiFi");

// Setup BME680 on I2C
const i2c = new I2C();
i2c.setup({sda:B4,scl:A8});
const bme = require("BME680").connectI2C(i2c);

// Setup MQTT client
const mqtt = require("MQTT").create(mqttServer, mqttOptions);


mqtt.on('connected', function() {
  console.log("MQTT Connected!");
});

mqtt.on('disconnected', function() {
  console.log("MQTT disconnected! Reconnecting...");
  setTimeout(function() { mqtt.connect(); }, 1000);
});




console.log("Connecting to ESP8266...");
wifi.connect(Serial2, function() {
  console.log("Connecting to WiFi...");
  wifi.connect(wifiSSID, wifiPWD, function() {
    console.log("WiFi Connected!");
    console.log("Connecting to MQTT...");
    //mqtt.connect();
  });
});




const collect = function() {

    const data = bme.get_sensor_data();
    bme.perform_measurement();
    if(!mqtt.connected) {
      console.log("Not connected. Ignoring data points");
      return;
    }

    const message = JSON.stringify(data,null,2);
    console.log("Sending data points...");
    mqtt.publish(mqttTopic, message);
};

setInterval(collect, 5000);
```



