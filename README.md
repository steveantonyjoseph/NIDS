NIDS IMPLEMENTATION USING EFK STACK,SURICATA,ZEEK,OPENCANARY HONEYPOT

1.Configuring Interfaces
An NIDS Solution requires 2 Interfaces Sniffing and Management respectively.Sniffing Interface is used to receive
all packets sent across the network. To accomplish this the interface must be put in what is called Promiscuous mode. 
Usually, an interface will only receive packets that are addressed to it, but when configured in promiscuous mode the interface will receive all packets.

In your Ubuntu or any Debian Based Distro terminal type
ip addr
sudo apt install -y ifupdown
sudo nano /etc/network/interfaces #Replace with interface names from above steps
#loopback
auto lo
iface lo inet loopback
#management interface
allow-hotplug enp0s17
iface enp0s17 inet dhcp
#sniffing interface
allow-hotplug enp0s8
iface enp0s8 inet manual
up ifconfig enp0s8 promisc up
down ifconfig enp0s8 promisc down
sudo service systemd-networkd stop
sudo apt remove -y netplan
sudo apt install net-tools -y
sudo reboot now
ip addr
If everything works perfectly you can see one of the interfaces having Promisc UP

2.Installing Suricata
Visit the documentation of Suricata and install Suricata or build from source file based on your needs

3.Configuring Suricata
nano /etc/suricata/suricata.yaml
Sample Configuration is there in the repository refer and change only the sniffing interface and add any ip ranges to the home-net variable list that are not already covered

4.Creating Suricata Service
A service makes it easier to run Suricata 
sudo nano /lib/systemd/system/suricata.service #Create the service file and copy the content below and paste to it
[Unit]
Description=Suricata Intrusion Detection Service
After=syslog.target network-online.target
[Service]
#Environment file to pick up $OPTIONS. On Fedora/EL this would be
#/etc/sysconfig/suricata, or on Debian/Ubuntu, /etc/default/suricata.
#EnvironmentFile=-/etc/sysconfig/suricata
#EnvironmentFile=-/etc/default/suricata
ExecStartPre=/bin/rm -f /var/run/suricata.pid
ExecStart=/usr/bin/suricata -c /etc/suricata/suricata.yaml --pidfile /var/run/suricata.pid --af-packet
ExecReload=/bin/kill -USR2 $MAINPID
[Install]
WantedBy=multi-user.target

4.Installing Zeek
Visit the documentation of Zeek and install Zeek or build from source file based on your needs

5.Configuring Zeek
sudo nano /usr/local/zeek/etc/node.cfg
Sample Configuration is there in the repository refer and change only the sniffing interface

sudo nano /usr/local/zeek/etc/zeekctl.cfg
Sample Configuration is there in the repository

We need to change the output format of the logs to json
sudo nano /usr/local/zeek/share/zeek/site/local.zeek
Enter these lines:
#Output to JSON
@load policy/tuning/json-logs.zeek

6.Installing Filebeat
Visit the documentation of Elastic and add their repository to your server 
sudo apt update
sudo apt install filebeat

7.Configuring Filebeat
sudo nano /etc/filebeat/filebeat.yml
Sample Configuration is there in the repository change 0.0.0.0 to your server ip

8.Installing Elasticsearch 
sudo apt update
sudo apt install elasticsearch

9.Configuring Elasticsearch
sudo nano /etc/elasticsearch/elasticsearch.yml
Sample Configuration is there in the repository

8.Installing Kibana 
sudo apt update
sudo apt install kibana

9.Configuring Kibana
sudo nano /etc/kibana/kibana.yml
Sample Configuration is there in the repository

10.Filebeat Modules Integration 
sudo filebeat modules enable suricata zeek system 
sudo nano /etc/filebeat/modules.d/zeek.yml
Uncomment var paths and replace with the directory of suricata logs 
sudo nano /etc/filebeat/modules.d/suricata.yml
Uncomment var paths and replace with the directory of suricata logs
sudo nano /etc/filebeat/modules.d/system.yml
Uncomment var paths and replace with the directory of system logs

11.Installing and Configuring Authelia
cd authelia
cd config
rm -rf db.sqlite3
cd ..
docker compose up
cd config
nano users_database.yml
For the password visit argon2.online iterations: 1 hash length: 32 salt: 16 memory cost: 1024 parallelism factor: 8 with the following parameters select argon2id add your
plaintext and generate hash
nano configuration.yml
Replace all placeholders

12.Installing Opencanary
Visit the documentation of Opencanary to install Opencanary

13.Configuring Opencanary
nano /etc/opencanaryd/opencanary.conf
Sample Configuration file is there in the repository

14.Final Step 
sudo systemctl start suricata
sudo systemctl enable suricata
sudo zeekctl deploy
sudo systemctl start filebeat
sudo systemctl enable filebeat
sudo systemctl start elasticsearch
sudo systemctl enable elasticsearch
sudo systemctl start kibana
sudo systemctl enable kibana
sudo systemctl status suricata #Ensure its active
sudo systemctl status elasticsearch #Ensure its active
sudo systemctl status kibana #Ensure its active
sudo systemctl status filebeat #Ensure its active

If you have configured NGINX PROXY MANAGER to reverse proxy to a domain you can navigate to it or else Kibana will be running at your server ip:5601. However for Authelia 
you would require a domain,login using your credentials and you can access Kibana you can create dashboards,go to discover to query logs , etc.

