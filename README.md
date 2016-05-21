Hardware

Raspberry Pi 2
Logitech C525 HD Webcam
EDIMAX Wireless USB Adapter
Betriebssystem

Zunächst muss Raspbian als Betriebssystem auf der Micro-SD-Karte des Raspberrys installiert werden. Eine Anleitung dazu findet sich hier.

Das Passwort und den Hostnamen des Raspberry PI kann man mit folgendem Tool anpassen:
sudo raspi-config
Im Anschluss sollte das OS auf den neuesten Stand gebracht werden:
sudo apt-get update && sudo apt-get upgrade -y
Wifi Adapter

Energiesparmodus abschalten

Datei anlegen
sudo nano /etc/modprobe.d/8192cu.conf
Inhalt
options 8192cu rtw_power_mgnt=0 rtw_enusbss=0
Konfigurationsdatei editieren

sudo nano /etc/network/interfaces
Inhalt anpassen zu
auto lo
iface lo inet loopback
iface eth0 inet dhcp
auto wlan0
allow-hotplug wlan0
iface wlan0 inet dhcp
wpa-ap-scan 1
wpa-scan-ssid 1
wpa-ssid "<Ihre SSID>"
wpa-psk "<Ihr PSK>"
#wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
iface default inet dhcp
Änderungen übernehmen

sudo service networking restart
motion installieren

sudo apt-get install motion
sudo nano /etc/motion/motion.conf



daemon off -> daemon on
process_id_file -> /home/raspbian/motion/motion.pid
width 320 -> 640 //Width of the camera feed
height 240 -> 480 //Height of the camera feed
framerate 15
event_gap 30 //motion end after 30 seconds
max_mpeg_time 12000 //Larger the value, smaller the video file
ffmpeg_cap_new off //Set to OFF as video is not recorded
webcam_quality 75
webcam_motion off -> on //Changed to 'on' so only 1 frame per second is broadcast to the web cam server until actual motion is detected)
webcam_localhost on -> off (change this to 'off' so you can view the webcam stream remotely)
webcam_localhost on -> off //Changed to 'off' so that webcam stream can be viewed remotely
webcam_maxrate 10 // The maximum number of frames per second the web cam server will show in your browser
on_event_start curl --header "Content-Type: text/plain" --request PUT --data "ON" http://yourOpenHabServer:8080/rest/items/itemId/state
on_event_end curl --header "Content-Type: text/plain" --request PUT --data "OFF" http://yourOpenHabServer:8080/rest/items/itemId/state

http://sirlagz.net/2013/02/12/quickie-getting-motion-working-on-the-raspberry-pi/

openHAB

https://github.com/openhab/openhab/wiki/Samples-REST#curl
mjpg-streamer installieren (statt motion)

Benötigte Pakete
sudo apt-get install subversion-tools libjpeg8-dev imagemagick
Sourcen abrufen
svn co https://svn.code.sf.net/p/mjpg-streamer/code/mjpg-streamer mjpg-streamer
Bauen
cd mjpg-streamer
make

Installieren
sudo make install
Automatisch starten / Dienst einrichten
sudo nano /etc/init.d/mjpg_streamer
#!/bin/sh
# /etc/init.d/mjpg_streamer
#
# Creation:    04.02.2013
# Last Update: 04.02.2013
#
# Written by Georg Kainzbauer (http://www.gtkdb.de)
#
### BEGIN INIT INFO
# Provides:          mjpg_streamer
# Required-Start:    $all
# Required-Stop:     $all
# Default-Start:     2 3 4 5
# Default-Stop:      0 1 6
# Short-Description: MJPG-Streamer
# Description:       MJPG-Streamer takes JPGs from Linux-UVC compatible webcams and streams them as M-JPEG via HTTP.
### END INIT INFO
start()
{
  echo "Starting mjpg-streamer..."
  /usr/local/bin/mjpg_streamer -i "/usr/local/lib/input_uvc.so -d /dev/video0 -n -y -r 640x480 -f 15" -o "/usr/local/lib/output_http.so -n -w /usr/local/www -p 80" >/dev/null 2>&1 &
}
stop()
{
  echo "Stopping mjpg-streamer..."
  kill -9 $(pidof mjpg_streamer) >/dev/null 2>&1https://raw.githubusercontent.com/mpodroid/door-berry/master/doorberry.install
}
case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  restart)
    stop
    starthttps://raw.githubusercontent.com/mpodroid/door-berry/master/doorberry.install-dep
    ;;
  *)
    echo "Usage: $0 {start|stop|restart}"
    ;;
esac
exit 0
sudo chmod 0755 /etc/init.d/mjpg_streamer
sudo update-rc.d mjpg_streamer defaults
door-berry installieren

door-berry von https://github.com/mpodroid/door-berry installiert einen SIP-Client auf dem Raspberry, der per Kommandozeile oder Python-Skript gesteuert werden kann. Es wird auch ein Dienst eingerichtet, der die GPIO-Pins überwacht und bei entsprechenden Ereignissen ein Anruf einleitet.
Install-Skipte herunterladen

wget https://raw.githubusercontent.com/mpodroid/door-berry/master/doorberry.prepare
wget https://raw.githubusercontent.com/mpodroid/door-berry/master/doorberry.install-dep
wget https://raw.githubusercontent.com/mpodroid/door-berry/master/doorberry.install
Ausführbar machen

chmod +x doorberry.prepare
chmod +x doorberry.install-dep
chmod +x doorberry.install
Install-Skipte ausführen

./doorberry.prepare
Aktualisiert OS und Raspberry PI Firmware.
./doorberry.install-dep
Installiert notwendige Pakete wie z.B. Python und PJSIP.
./doorberry.install
Installiert Python-Skript zur Überwachung der GPIO-Pins als Dienst.

Abschließend muss noch eine Änderung eines Install-Skripts rückgängig gemacht werden, da ansonsten mjpg-streamer nicht mehr auf die Kamera zugreifen kann.
sudo rm /boot/cmdline.txt
sudo reboot
Konfiguration

SIP-Einstellungen

nano ~/door-berry/station/station.py
SIP_SERVER="fritzBoxIP"
SIP_USER="interenNummerDesDoorberry"
SIP_PASS="Geheimnis"
SIP_REALM="*"
Verwendete I/Os

OUT1=24
OUT2=23
OUT3=18
IN1=22
IN2=17
IN3=4
Zielrufnummern

nano ~/door-berry/station/configuration.py

Zusammenbau




