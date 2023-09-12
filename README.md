![Scherm­afbeelding 2023-02-11 om 17 16 43](https://user-images.githubusercontent.com/123865090/218268856-645c052f-e9c9-4f45-8e22-15b283a116dc.png)

This is a modified version of TWC Manager that reads the current amps used from an MQTT feed (as provided by DSMR Reader for a Dutch smart meter) and makes the remaining amps available to the TWC. It no longer relies on the Tesla API and the functionality for green energy tracking has been removed. Use cases:

- Higher charging speeds if you have a 17 or 22 kW onboard charger
- Maximize charging during off-peak hours on a dynamic energy contract (sometimes 1 or 2 hours windows 

Note: Make sure the circuit breaker for your TWC actually supports the higher amps :-)

You'll need an RS-485 adapter and Raspberry Pi or similar, wiring and setting up is not covered here but instructions can be found at https://github.com/dracoventions/TWCManager/blob/master/TWCManager%20Installation.pdf

To do:
- Make load balancing possible for scheduled charging
- Fail safe to default to 16 amps when MQTT feed is no longer updated

Install to following packages:
sudo apt-get install -y mosquitto mosquitto-clients git screen python3 python3-pip lighttpd php7.4-cgi 

Setup MQTT

Add the following lines to /etc/mosquitto/mosquitto.conf

listener 1883 0.0.0.0
allow_anonymous true

and then 
sudo systemctl restart mosquitto

Configure DSMR Reader as follows:
- Configure an MQTT broker with the IP address of you RPi | Port 1883 | Insecure (no SSL/TLS)
- Activate sending of (Databron) Telegram: JSON messages

Setup web server

sudo lighty-enable-mod fastcgi-php
service lighttpd force-reload
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 775 /var/www/html
sudo usermod -a -G www-data <your_current_username>

Install Python packages:
sudo python3 -m pip install pyserial
sudo python3 -m pip install sysv_ipc
sudo python3 -m pip install paho-mqtt

cd ~
git clone https://github.com/Moving4407/TWCP1LoadBalancer.git TWC
cp TWC/HTML/* /var/www/html
sudo nano ~/TWC/TWCManager.py

Edit the script to, at a minimum, set:
wiringMaxAmpsAllTWCs = 25 # This is the maximum power made available to you, equal to the amps on your main circuit breaker 
wiringMaxAmpsPerTWC = 25 # If you have multiple TWC's installed they will all have their own circuit breaker and you can set this value here. Otherwise 

You can now run the script by typing:

cd TWC/
sudo chmod +x TWCManager.py
./TWCManager.py 

If everything works as advertised you can run the script at startup:

sudo nano /etc/rc.local

Near the end of the file, before the exit 0 line, add:

su - pi -c "screen -dm -S TWCManager ~/TWC/TWCManager.py"
