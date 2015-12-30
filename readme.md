Electronic Logbook
==================

Powered by Etherpad Lite
------------------------

This repository is for a dedicated electronic logbook on a Raspberry Pi using
Etherpad-Lite. Local system monitoring is achieved using RPi-Monitor. An off
button is provided to safely shut down while operating headless.

### Usage <a name="h3-usage"></a>

#### Normal Use

**TODO**

* system boots within ~30 sec, the logbook server takes ~2 min to get online
* hold power button several seconds to turn off and wait for green LED to
  quiet before removing power
* has hostname `logbook` but requests IP via DHCP
* logbook server operates on normal port, 80 (i.e http://logbook.local)
    * no login to use the logbooks
    * can edit only, not create (see admin)
* things to highlight in etherpad:
    * author names
    * headings
    * history
    * chat
* security warnings:
    * no authentication, use alternate secure mechanism (ours is wifi wpa2)
    * no apparent way for admin to archive (lock) a pad

related:

* rpi-monitor for users to view system load, etc
* samba share for users to get how-to instructions, archived pads



### Administration <a name="h3-admin"></a>

**TODO**

* administer etherpad-lite at `/admin` webpage
    * enable/disable pad creation
    * ???
* view system status using rpi-monitor
* administer the samba shares via `/etc/samba.conf` ????
* system administration tasks
    * time settings: timezone, ntp source
    * package updates: manually? how frequently?
    * network settings: looking up



* template sheet "Useful facts about this computer"
    * description
    * setup by facts (date, name, email)
    * associated repositories
    * network info
        * hostname
        * LAN mac address
        * samba shares
    * credentials
        * raspbian root user
        * etherpad administrator
        * mysql administrator
        * mysql etherpad user
        * samba share users, if any


### Bill of Materials <a name="h3-bom"></a>

| Description                                             | ~USD$ |
|---------------------------------------------------------|-------|
| [Raspberry Pi Model B+][bom0]                           |   30  |
| [4GB uSD card (w/ Jessie Lite)][bom1]                   |   10  |
| Momentary switch, [such as][bom2]                       |    1  |
| Case for RPi (we prefer [this one*][bom3])              |    7  |
| *Total*                                                 |   48  |

*\* It's inexpensive, easy to modify for a button and lacks unnecessary slots.*

  [bom0]: https://www.adafruit.com/products/1914
  [bom1]: https://www.adafruit.com/products/2820
  [bom2]: http://www.amazon.com/gp/product/B00HG7GWRK
  [bom3]: http://www.amazon.com/gp/product/B00ONOKPHC


### Initial Setup <a name="h3-setup"></a>

#### Getting Started

If you didn't purchase a microSD card with a clean Raspbian Jessie (Nov 2015)
install, you'll need to prepare one according to your [favorite method]

* Start with clean image: `2015-11-21-raspbian-jessie-lite.img`
* Do basic config: `sudo raspi-config`
    * Expand file system: yes
    * i18n
        * locale: US.UTF_8
        * keyboard: US english (defaults)
        * timezone: GMT+8 (Pacific Standard)
    * Advanced options
        * set hostname `logbook`
        * enable SSH
* Reboot
* Add Adafruit repos: `curl -sLS https://apt.adafruit.com/add | sudo bash`
* Update system:

````
pi@logbook:~ $ sudo apt-get update
pi@logbook:~ $ sudo apt-get upgrade -y
pi@logbook:~ $ sudo reboot
````

#### Install Etherpad 

* Install packages:
    * node (0.12.6-1)
    * git
    * libssl-dev
* Grab etherpad lite source, checkout latest release then create and
  configure settings

````
pi@logbook:~ $ sudo git clone https://github.com/ether/etherpad-lite.git /srv/etherpad-lite
Cloning into '/srv/etherpad-lite'...
remote: Counting objects: 27466, done.
remote: Total 27466 (delta 0), reused 0 (delta 0), pack-reused 27466
Receiving objects: 100% (27466/27466), 19.15 MiB | 284.00 KiB/s, done.
Resolving deltas: 100% (19467/19467), done.
Checking connectivity... done.
pi@logbook:~ $ cd /srv/etherpad-lite
pi@logbook:/srv/etherpad-lite $ sudo cp settings.json.template settings.json
pi@logbook:/srv/etherpad-lite $ sudo nano settings.json
````
````
enable admin by uncommenting user section
````

* Test run: 

````
pi@logbook:/srv/etherpad-lite $ sudo bin/run.sh

````
Works OK, but not when added to cron table


#### Promote Etherpad to service

<https://github.com/ether/etherpad-lite/wiki/How-to-deploy-Etherpad-Lite-as-a-service>

create new user, grant that user access and create new service file

````
pi@logbook:~ $ sudo adduser --system --home=/srv/etherpad-lite --group etherpad-lite
pi@logbook:~ $ sudo chown -R etherpad-lite:etherpad-lite /srv/etherpad-lite
pi@logbook:~ $ sudo nano /etc/systemd/system/etherpad-lite.service
````
````
[Unit]
Description=etherpad-lite (real-time collaborative document editing)
After=syslog.target network.target

[Service]
Type=simple
User=etherpad-lite
Group=etherpad-lite
ExecStart=/bin/sh /srv/etherpad-lite/bin/run.sh

[Install]
WantedBy=multi-user.target
````

Save.

````
pi@logbook:~ $ sudo systemctl enable etherpad-lite
Created symlink from /etc/systemd/system/multi-user.target.wants/etherpad-lite.service to /etc/systemd/system/etherpad-lite.service
pi@logbook:~ $ sudo systemctl start etherpad-lite
````

Works! Now, edit the newly created configuration file:

````
pi@logbook:~ $ sudo nano /srv/etherpad-lite/settings.json
````

Tasks you might consider:

* Change the bound port (9001) to something else (say, 80)
* Enable users (for admin capabilities)
* ????

#### Enable Etherpad on port 80

* http://stackoverflow.com/a/27805105
* https://www.digitalocean.com/community/tutorials/how-to-use-pm2-to-setup-a-node-js-production-environment-on-an-ubuntu-vps

````
pi@logbook:~ $ which node
/usr/local/bin/node
pi@logbook:~ $ sudo setcap cap_net_bind_service=+ep /usr/local/bin/node
````

After restarting etherpad, it can be on port 80


#### Install "real" database

Install database

````
pi@logbook:~ $ sudo apt-get install mysql-server -y
````

It will prompt (twice) for a mysql root user password: `defense_contractors`
Then it takes a while to finish installing.

Launch a mysql session:

````
pi@logbook:~ $ mysql -u root -p
Enter password: 
````

It provides a new prompt. Create a database and grant a new user rights to it.
In this example, the credentials are `USERNAME`/`PASSPHRASE`.

> I've actually used `ep`/`thundercats`

````
mysql> create database `etherpad-lite`;
Query OK, 1 row affected (0.00 sec)

mysql> grant CREATE,ALTER,SELECT,INSERT,UPDATE,DELETE on `etherpad-lite`.* to 'USERNAME'@'localhost' identified by 'PASSPHRASE';
Query OK, 0 rows affected (0.00 sec)

mysql> exit
Bye
````

Update etherpad to use mysql:

````
pi@logbook:~ $ sudo nano /srv/etherpad-lite/settings.json
````
````
  ...
  //You shouldn't use "dirty" for for anything else than testing or development
- "dbType" : "dirty",
+ //"dbType" : "dirty",
  //the database specific settings
- "dbSettings" : {
-                "filename" : "var/dirty.db"
-              },
+ //"dbSettings" : {
+ //               "filename" : "var/dirty.db"
+ //             },

- /* An Example of MySQL Configuration
-  "dbType" : "mysql",
-  "dbSettings" : {
-                   "user"    : "root",
-                   "host"    : "localhost",
-                   "password": "",
-                   "database": "store"
-                 },
- */
+ /* An Example of MySQL Configuration */
+  "dbType" : "mysql",
+  "dbSettings" : {
+                   "user"    : "ep",
+                   "host"    : "localhost",
+                   "password": "thundercats",
+                   "database": "etherpad-lite"
+                 },
+ /**/
  ...
````

and restart etherpad. wait for it to load completely (up to 2 minutes); use `top`
to watch progress (CPU usage drops to near 0%)

````
pi@logbook:~ $ sudo service etherpad-lite restart
````

go back into mysql

````
pi@logbook:~ $ mysql -u root -p
Enter password: 
````
````
mysql> ALTER DATABASE `etherpad-lite` CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
Query OK, 1 row affected (0.00 sec)

mysql> USE `etherpad-lite`;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> ALTER TABLE `store` CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;
Query OK, 1 row affected (0.04 sec)
Records: 1 Duplicates: 0 Warnings: 0

mysql> ALTER TABLE `store` ENGINE = MyISAM;
Query OK, 1 row affected (0.04 sec)
Records: 1 Duplicates: 0 Warnings: 0

mysql> quit
Bye
````

It's not 100% the choice to revert from the InnoDB to MyISAM engine is 
really appropriate. The etherpad-lite wiki claims it's a 10x performance
increase but not much 3rd-party information supports that claim. It's
possible, but probably not 10x

http://dba.stackexchange.com/questions/17431/which-is-faster-innodb-or-myisam

restart etherpad service

````
pi@logbook:~ $ sudo service etherpad-lite restart
````

viola!



#### Custom favicon

Retrieve the WSU favicon:

````
pi@logbook:~ $ cd /srv
pi@logbook:/srv $ sudo wget http://images.wsu.edu/favicon.ico
pi@logbook:/srv $ sudo chown etherpad-lite:etherpad-lite favicon.ico
````

#### TODO

* serve over HTTPS? letsencrypt.org
* install useful plugins
    * pad manager
    * ?
* apply WSU color scheme


#### Add system monitoring page

<http://rpi-experiences.blogspot.fr/>
<https://github.com/XavierBerger/RPi-Monitor-deb>

not necessary to update any packages, it already has latest `dpkg-dev`, 
`apt-transport-https`, and `ca-certificates`. 

First add the new repo source url then add developer's key (2C0D3C0F).
Next update packages list to realize RPi-Monitor exists. Get stuck on error.

````
pi@logbook:~ $ sudo bash -c 'echo "deb https://github.com XavierBerger/RPi-Monitor-deb/raw/master/repo/" >> /etc/apt/sources.list.d/rpimonitor.list' 
pi@logbook:~ $ sudo apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 2C0D3C0F
...
pi@logbook:~ $ sudo apt-get update
... produces error
````

Try direct file download and install:

````
pi@logbook:~ $ wget https://github.com/XavierBerger/RPi-Monitor-deb/raw/master/repo/rpimonitor_2.10-1_all.deb
pi@logbook:~ $ sudo dpkg -k rpimonitor_2.10-1.deb
````

It installs, with dependencies missing. 

````
pi@logbook:~ $ sudo apt-get install -f -y
...lots of output
````

Now run post-install configuration script:

````
pi@logbook:~ $ sudo apt-get update && sudo /usr/share/rpimonitor/scripts/updatePackagesStatus.pl
````


