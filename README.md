Here are some simple instructions for setting up a Debian/Ubuntu based OpenNMS install using the repos, along side Grafana, Syslog-NG, and postgresql10.
Have a great OpenNMS dev experience with these notebooks, in a hurry.

0) Install Ubuntu Server 18.04 LTS. Command line only. NO GUI. You don't need it, unless you are going to access the web GUI on the same machine you are installing it on.

1) Use IP tables. Don't use ufw, as we are setting up iptables-persistent in a later step. See commands->

sudo apt-get purge ufw* -y

2) Install syslog repo for latest greatest syslog-ng repo. See commands->
wget -qO - http://download.opensuse.org/repositories/home:/laszlo_budai:/syslog-ng/xUbuntu_18.04/Release.key | sudo apt-key add -

3) Create a file in the following directory-> 
sudo nano /etc/apt/sources.list.d/syslog-ng-obs.list

4) Add this to the file and save it-> 
deb http://download.opensuse.org/repositories/home:/laszlo_budai:/syslog-ng/xUbuntu_18.04 ./

5) Run update/upgrade. If syslog is installed, it will update-> 
sudo apt-get update && sudo apt-get upgrade -y && sudo apt-get dist-upgrade -y && sudo apt-get autoremove -y

6) If syslog-ng is not installed, install it-> 
sudo apt-get install syslog-ng* -y

7) Install dependencies: (Defaults are fine. If some of these are outdated, then remove those from line, paste edited command and run again.)-> 
sudo apt-get install iptables iptables-dev netfilter-persistent iptables-persistent smitools unzip clamav lm-sensors unzip snmp net-tools snmpd cmdtest golang libsnmp-dev apt-transport-* libdbi1 libdbd-mysql r-recommended devscripts dpkg-sig expect nsis babel-core* snmptrapd snmp-mibs-downloader rrdtool osm2pgsql php-bcmath nmap* clamav* rkhunter openjdk-11-* openjdk-11-jdk liboce-* libxext-doc -y moreutils -y default-jre* -y x509-util -y vnstat -y traceroute -y dom4j* -y java-se* -y xerces* -y zookeeper -y scala-library* -y libosgi-core-* libosgi-foundation-ee-java -y dnsutils -y bind9-host -y libdns* -y

8) Install more dependencies:  (If some of these are outdated, then remove those from line, paste edited command and run again. This is split up to avoid conflicts.)-> 
sudo apt-get install libcurl4-openssl-dev -y libcurl4-gnutls-dev -y libcurl4-nss-dev -y update-alternatives* -y git* -y

9) Reboot computer-> 
sudo reboot

10) Install opennms thru repos using this convenient script. Login to rebooted system. Commands below. Follow all instructions and Document EVERYTHING. Defaults are fine.-> 
git clone git clone https://github.com/opennms-forge/opennms-install/
cd opennms-install
sudo chmod +x bootstrap-debian.sh
sudo bash ./bootstrap-debian.sh

11) Once install is complete run these commands.
sudo apt-get install jrrd* -y
sudo /usr/share/opennms/bin
sudo ./install -dis

12) start and enable postgresql for boot
sudo systemctl start postgresql
sudo systemctl enable postgresql

13) Update and start OpenNMS
cd /usr/share/opennms/bin
sudo ./upgrade
sudo systemctl start opennms
sudo systemctl enable opennms

14) Wait a while, make sure everything is at status `running` by running this command.
sudo opennms -v status

15) Once all services show running, then access web gui from another workstation, or if you installed a gui/desktop, access it from a web browser on that machine. I don't recommend a GUI. Build it CLI only and use a workstation to access the web GUI from. Here is the URL to start with. Default user/pass is admin/admin
http://type IP address of server instead of this string:8980

16) Go back to CLI. Run these commands.
sudo apt-get install opennms-plugin-* -y
sudo apt-get install opennms-webapp* -y

17) Let's install grafana and openhelm plugin so opennms, grafana, and postgresql can work together.
sudo curl https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
sudo apt-get update -y && sudo apt-get upgrade -y && sudo apt-get dist-upgrade -y && sudo apt-get autoremove -y
sudo apt-get install grafana* snmp snmpd snmptrapd snmp-mibs-downloader rrdtool

18) Start grafana and enable on boot. Then check GUI in a web browser. Here is URL:  http://Type IP address of server here instead of string:3000
sudo systemctl start grafana-server
sudo systemctl enable grafana-server

19) Go back to command line. Install more dependencies.
sudo apt-get install mib2opennms
sudo apt-get install mib2events
sudo apt-get install opennms-jmx-config-generator
sudo apt-get install opennms-plugins

20) Install Openhelm OpenNMS plugin for grafana, then install in grafana
sudo apt-get install opennms-helm* -y
grafana-cli plugins install opennms-helm-app

21) Log into grafana web gui, or refresh with F5. You will have to enter a new admin password. default user/pass is admin/admin
Here is URL again:  http://Type IP address of server here instead of string:3000

22) Add the Postgresql data source in grafana GUI and connect to your postgresql database.
Import the Postgresql dashboard in the GUI. Don't worry about the JSON box. Just put this number in the smaller box and import:  6048.
Now you have monitoring of your postgresql database that OpenNMS uses in grafana!

23) Enable opennms-helm in the GUI, or at the command line.

24) Restart grafana services
sudo systemctl restart grafana-server

25) Reboot server to see that everything comes up on boot, and talks to each other properly.

26) Biggest issues you may run into: JAVA!!!!!
You will have to google some of that, but here are some helpful java reference commands. Those are backticks in the third command, not single quotes.

java -version
which java
ls -lash `which java`
sudo update-alternatives --config java


27) If you want to set java home manually, then do this.
cd /etc/profile.d
sudo nano java.sh

28) Add this to the new and blank java.sh you have opened/created. Take a look at the helpful java reference commands in step 26) and see if what I have matches your installation. Save and close.
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk

29) And finally, place ALL opennms package upgrades on HOLD. This is important, because once you start altering the default install in ANY way, the automated repo upgrades will break your installation.
Here is commands for that. If some of the services are a little different, you can look to see what you are missing, but this next command should get it.
sudo apt-mark hold opennms \
	opennms-db \
	opennms-server \
	opennms-webapp-jetty \
	opennms-contrib \
	opennms-server \
	opennms-plugin-* \
	opennms-source \
	libopennms-java \
    	libopennmsdeps-java \
    	opennms-common \
	opennms-webapp-hawtio \
	opennms-webapp-remoting \
	opennms-alec-plugin \
	opennms-jmx-config-generator \
	opennms-plugins \
	opennms-doc

30) If the above command freaks out saying it can't find something, then remove that line from the command, re-paste and run, we can look for any new services and hold them next.
Whatever output this next command spits out, you'll be able to see which packages you haven't marked for hold status.
sudo apt-cache search opennms

31) In closing, if you made it this far, then you are ready for the bigger stuff in the following notebooks on my github. Good luck and have fun with OpenNMS Horizon. I am.














