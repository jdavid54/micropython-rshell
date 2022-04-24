# micropython-rshell

## how to connect to ESP32/DHT11 with rshell

Connect to ESP32 with DHT11 with terminal.

```python
pi@raspberrypi:~ $ ls /dev/*USB*
/dev/ttyUSB0

pi@raspberrypi:~ $ rshell -p /dev/ttyUSB0

Using buffer-size of 32
Connecting to /dev/ttyUSB0 (buffer-size 32)...
Trying to connect to REPL  connected
Testing if ubinascii.unhexlify exists ... Y
Retrieving root directories ... /DHT11_pub.py/ /DHT11_pub2.py/ /boot.py/ /ftp.py/ /main.old/ /main.py/ /main.tmp/ /main2.py/ /main_moulin.py/ /main_old.py/ /microWebSocket.py/ /microWebSrv.py/ /microWebSrv2.png/ /microWebTemplate.py/ /min.py/ /start.py/ /start2.py/ /start2b.py/ /umqtt/ /uping.py/ /www/
Setting time ... Apr 24, 2022 08:57:36
Evaluating board_name ... pyboard
Retrieving time epoch ... Jan 01, 2000
Welcome to rshell. Use Control-D (or the exit command) to exit rshell.


/home/pi> ls /pyboard
umqtt/              main.py             microWebTemplate.py main.old           
www/                main2.py            min.py              main.tmp           
DHT11_pub.py        main_moulin.py      start.py            microWebSrv2.png   
DHT11_pub2.py       main_old.py         start2.py          
boot.py             microWebSocket.py   start2b.py         
ftp.py              microWebSrv.py      uping.py           

# main.py is launched when ESP32 reboots.
/home/pi> cat /pyboard/main.py

# python START

import network
from time import sleep
from umqtt.simple import MQTTClient
from machine import Pin
import uping
from dht import DHT11

led = Pin(2, Pin.OUT)
sensor = DHT11(Pin(15, Pin.IN, Pin.PULL_UP))

station = network.WLAN(network.STA_IF)
station.active(True)

# IP of broker server
SERVER = '192.168.0.184'
station.connect(SSID,PASSWORD)
while station.isconnected() == False:
	pass

print('Connection successful')
print(station.ifconfig())

#flash once
led.value(1)
sleep(0.1)
led.value(0)
sleep(1)

#SERVER = 'node02.myqtthub.com' #MQTT Server address ip = 185.195.98.58 using ping to get

CLIENT_ID = 'ESP8266_sensor'
TOPIC = b'temp_humidity'
USER = 'sensor'
PASSW = 'sensor01'
PORT = 0

#client = MQTTClient(CLIENT_ID, SERVER) #, PORT, USER, PASSW)
client = MQTTClient(CLIENT_ID, SERVER, PORT, USER, PASSW)
#print(client)
state = client.connect()
#print(state)

#flash once
led.value(1)
sleep(0.1)
led.value(0)
sleep(1)

#msg = (b'DH11 speaking ....')
#client.publish(TOPIC, msg)

while True:
    try:
        sensor.measure()
        sleep(0.1)
        t = sensor.temperature()
        h = sensor.humidity()
        if isinstance(t, int) and isinstance(h, int):
            #msg = (b'0, 0')
            #client.publish(TOPIC, msg)
            msg = (b'{0:3.1f}, {1:3.1f}'.format(t,h))
            client.publish(TOPIC, msg)
            #flash twice
            led.value(1)
            sleep(0.1)
            led.value(0)
            sleep(0.1)
            led.value(1)
            sleep(0.1)
            led.value(0)
            print(msg)
            sleep(20)
        else:
            print('Invalid sensor readings.')
    except OSError:
        print('Failed to read sensor.')
        try:
            client.connect()
        except:
            led.value(1)
            sleep(0.1)
            led.value(0)
# python END

/home/pi> 

CTRL-D to quit.
```
