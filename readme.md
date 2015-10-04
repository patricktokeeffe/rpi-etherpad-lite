Setup Etherpad Lite server on Raspberry Pi
==========================================

Done using Raspberry Pi Model 1 B+ with Raspbian Wheezy (2015 May)

1. Complete basic configuration: `sudo raspi-config`
    * expand file system
    * change locale to US/english
    * change keyboard to US/english
    * verify SSH is enabled
2. Add Adafruit repository: `curl -sLS https://apt.adafruit.com/add | sudo bash`
3. Install node (0.12.6): `sudo apt-get install node`
4. Install systemd: `sudo apt-get install systemd`
   and enable: add "init=/bin/systemd" to `/boot/cmdline.txt`
5. Create service for Etherpad Lite
    * make user: `sudo adduser --system --group --home=/srv/etherpad-lite etherpad-lite`
    * ensure perms: `sudo chown -R etherpad-lite /srv/etherpad-lite
    * create service file: `sudo nano /etc/systemd/system/etherpad-lite.service`

        [Unit]
        Description=etherpad-lite (real-time collaborative document editing)
        After=syslog.target network.target

        [Service]
        Type=simple
        User=etherpad-lite
        Group=etherpad-lite
        ExecStart=/srv/etherpad-lite/bin/run.sh

        [Install]
        WantedBy=multi-user.target

6. Install Etherpad Lite
    * clone: `sudo git clone https://github.com/ether/etherpad-lite.git /srv/etherpad-lite`
    * checkout release: `sudo git checkout -b 1.5.7
7. Setup Etherpad Lite
    * enter: `cd /srv/etherpad-lite`
    * create settings file: `sudo cp settings.json.template settings.json`
    * edit settings: `sudo nano setting.json`
        * enable admin
    * test-run as user: `sudo -u etherpad-lite bin/run.sh` ... success
8. Reboot: `sudo reboot`
9. Enable the service
    * Fails: `sudo systemctl enable etherpad-lite`
    * Succeeds: `sudo systemctl enable etherpad-lite.service`
    * ...however, can't access Etherpad Lite
    * try adding `local-fs.target` as requirement/after to service file.... nope
    * change from simple to forking... not running after reboot, so manually start...





