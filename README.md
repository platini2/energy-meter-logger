Update an modify scripts to run in python3 from original proyect https://github.com/samuelphy/energy-meter-logger, change to modbus_tk module and add new industrial devices metters.
Add support for ModbusTCP and add bridge TCP to RTU vía ESP8266. Added possibility to use more than one InfluxDB server (or database)

# Energy Meter Logger
Log your Energy Meter data on a Raspberry Pi/Orange Pi and plot graphs of your energy consumption.
Its been verified to work with a Raspberry Pi and Orange Pi One / Zero with a Linksprite RS485 shield and USB to RS485 adapter or use Modbus-Gateway-esp8266 for reading values from SDM120M (SDM120CTM, SDM120CT-MV), SDM220M, SDM230M, SDM630M, YG194E-9SY, YG194E-95Y, YG889E-9SY, PZEM-016 and DDS238-1 ZN. By changing the meters.yml file and making a corresponding [model].yml file it should be possible to use other modbus enabled models.

Add support for ModbusTCP and add bridge TCP to RTU vía ESP8266 and MAX485.

### Requirements

#### Hardware

* Raspberry Pi 3 / Orange Pi One / Orange Pi Zero H2
* [Linksprite RS485 Shield V3 for RPi](http://linksprite.com/wiki/index.php?title=RS485/GPIO_Shield_for_Raspberry_Pi) or a simple [USB RS485 adapter](https://es.aliexpress.com/item/HOT-SALE-2pcs-lot-USB-to-RS485-485-Converter-Adapter-Support-Win7-XP-Vista-Linux-Mac/1699271296.html) or a [Modbus-Gateway-esp8266](https://github.com/GuillermoElectrico/Modbus-Gateway-esp8266) 
* Modbus based Energy Meter, e.g Eastron SDM120M or Eastron SMD630M or PZEM-016 or DDS238-1 ZN or Industrial metter YG194E-9SY / YG889E-9SY.

#### Software

* Rasbian or armbian or dietpi (buster)
* Python 3.4 and PIP3
* PyYAML 5.1 ((pip3 install -U PyYAML or python3 -m pip install -U PyYAML) if installed)
* [modbus_tk](https://github.com/ljean/modbus-tk)
* [InfluxDB](https://docs.influxdata.com/influxdb/v1.3/)
* [Grafana](http://docs.grafana.org/)

### Prerequisite

This project has been documented at [Hackster](https://www.hackster.io/samuelphy/energy-meter-logger-6a3468). Please follow the instructions there for more detailed information.

### Installation
#### Install InfluxDB*

##### Step-by-step instructions
* Add the InfluxData repository
    ```sh
	$ sudo apt-get update && sudo apt-get install curl apt-transport-https -y
    $ curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
    $ source /etc/os-release
    $ test $VERSION_ID = "10" && echo "deb https://repos.influxdata.com/debian buster stable" | sudo tee /etc/apt/sources.list.d/influxdb.list
    ```
* Download and install
    ```sh
    $ sudo apt-get update && sudo apt-get install influxdb
    ```
* Start the influxdb service
    ```sh
    $ sudo service influxdb start
    ```
* Create the database with data retention of 6 months (if you want more increase the value of weeks or delete it).
    ```sh
    $ influx
    CREATE DATABASE db_meters WITH DURATION 24w
    exit
    ```
[*source](https://docs.influxdata.com/influxdb/v1.7/introduction/installation/)

#### Install Grafana*

##### Step-by-step instructions
* Add APT Repository
    ```sh
    $ echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
    ```
* Add Bintray key
    ```sh
    $ curl https://packages.grafana.com/gpg.key | sudo apt-key add -
    ```
* Now install
    ```sh
    $ sudo apt-get update && sudo apt-get install grafana
    ```
* Start the service using systemd:
    ```sh
    $ sudo systemctl daemon-reload
    $ sudo systemctl start grafana-server
    $ systemctl status grafana-server
    ```
* Enable the systemd service so that Grafana starts at boot.
    ```sh
    $ sudo systemctl enable grafana-server.service
    ```
* Go to http://localhost:3000 and login using admin / admin (remember to change password)
[*source](http://docs.grafana.org/installation/debian/)

#### Install Energy Meter Logger:
* Download and install from Github and install pip3
    ```sh
	$ sudo apt-get install git
    $ git clone https://github.com/GuillermoElectrico/energy-meter-logger
	$ sudo apt-get install python3-pip
    ```
* Run setup script (must be executed as root (sudo) if the application needs to be started from rc.local, see below)
    ```sh
    $ cd energy-meter-logger
    $ sudo python3 setup.py install
    ```    
	
	If error appears use previously:
	```sh
	$ sudo pip3 install setuptools
	```
	 
* Make script file executable
    ```sh
    $ chmod 777 read_energy_meter.py
    ```
* Edit meters.yml and influx_config.yml to match your configuration
* Test the configuration by running:
    ```sh
    ./read_energy_meter.py
    ./read_energy_meter.py --help # Shows you all available parameters
    ```
	
	If the error appears:
	```
	/usr/bin/env: ‘python3\r’: No such file or directory
	```
	Use dos2unix to fix it.
	```
	$ sudo apt install dos2unix
	$ dos2unix /PATH/TO/YOUR/FILE
	```
	
* To run the python script at system startup. Add to following lines to the end of /etc/rc.local but before exit:
    ```sh
    # Start Energy Meter Logger
    /home/pi/energy-meter-logger/read_energy_meter.py --interval 10 > /var/log/energy_meter.log &
    ```
    Log with potential errors are found in /var/log/energy_meter.log

#### Optional, Install and Configure RTC DS3231

In the case of not having internet in the installation where you have the meter with the raspberry pi, you can install an RTC DS3231 module to be able to correctly register the date and time in the database and grafana.

##### Step-by-step instructions
* First connect the RTC module
	Connect to the corresponding pins +3.3V, SDA1 (GPIO2), SCL1 (GPIO3) and GND of the raspberry pi (depending on the model, in google there are examples).  

* Enable I2C port vía raspi-config*
    ```sh
    $ sudo raspi-config
    ```
	
	Reboot after enabled.
	
	*If you use orange pi or similar, consult documentation.
	
*  Install i2c-tools and verify that the i2c bus and the RTC module are working (Optional)
    ```sh
    $ sudo apt-get install i2c-tools
	$ sudo i2cdetect -y 1
	    0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
	00:          -- -- -- -- -- -- -- -- -- -- -- -- --
	10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
	20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
	30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
	40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
	50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
	60: -- -- -- -- -- -- -- -- 68 -- -- -- -- -- -- --
	70: -- -- -- -- -- -- -- --
    ```
* Now check the time of the module, and if it is the case, update the date and time.
	
	Enable RTC module:
	```sh
	$ sudo bash
    # echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device
	$ exit
    ```
	
	With this the time is read from the RTC:
	```sh
    $ sudo hwclock -r --rtc /dev/rtc0
    ```
	
	*If you get an error or can not find /dev/rtc0, check the name of the rtc with:
	```sh
	$ ls /dev/rtc?
    ```
	
	The system time can be seen with: 
    ```sh
	$ date
	jue may  5 23:02:46 CLST 2016
	```
	
	To set the system time, this command is used:
    ```sh
	$ sudo date -s "may 5 2016 23:09:40 CLST"
	jue may  5 23:09:40 CLST 2016
    ```
	
	Now as the system clock is fine, you can set the time in the RTC as:
	```sh
    $ sudo hwclock -w --rtc /dev/rtc0
    ```


* To set the date from the rtc each time the system is started Add to following lines to the end of /etc/rc.local but before exit:
    ```sh
	$ sudo nano /etc/rc.local
	```
	
	```sh
    echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device
	sleep 1
	hwclock -s --rtc /dev/rtc0
    ```

#### Optional, Configure Grafana to anonymous login and redirect to port 80

If you need to access grafana without adding port 3000 in the address, and do not want to have to log in every time you want to see. Follow the steps below.

##### Step-by-step instructions
* First install nginx
    ```sh
    $ sudo apt-get update && sudo apt-get install nginx
    ```

* Configure nginx default config
    ```sh
	$ sudo mv /etc/nginx/sites-available/default /etc/nginx/sites-available/default.old
    $ sudo nano /etc/nginx/sites-available/default
    ```

	Add
	```sh
	server {
		listen 80;
		server_name your-domain-name.com;
		location / {
			proxy_set_header   X-Real-IP $remote_addr;
			proxy_set_header   Host      $http_host;
			proxy_pass         http://127.0.0.1:3000;
		}
	}
	```

	Exit and save.

*  Edit grafana config
    ```sh
    $ sudo nano /etc/grafana/grafana.ini
    ```

	Modify this line to enable Anonymoun Auth
	```sh
	[auth.anonymous]
	# enable anonymous access
	enabled = true
	```

	Exit and save.

	Reboot the system a enjoy it