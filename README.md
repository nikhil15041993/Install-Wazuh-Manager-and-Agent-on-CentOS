# Install-Wazuh-Manager-and-Agent-on-CentOS


## WAZUH

Wazuh is a free, open source and enterprise-ready security detection and monitoring solution.

Wazuh is born as a fork of OSSEC (HIDS) host based intrusion detection system. Later is was integrated with Elastic stack and OpenSCAP.

### Wazuh System consist with several components
```
OSSEC HIDS - Host Based Intrusion Detection System

OpenSCAP - Open Vulnerability Assessment Language

Elastic Stack - Filebeat, Elasticsearch, Kibana

Wazuh is loaded with number of valued capabilities.
```

## Step-by-step installation

## 1.Installing Wazuh
To start setting up Wazuh, add the Wazuh repository to the server.

### Adding the Wazuh repository

Install the necessary packages for the installation:


 ```yum install curl unzip wget libcap```

Import the GPG key:


```
rpm --import https://packages.wazuh.com/key/GPG-KEY-WAZUH
```
Add the repository

```
cat > /etc/yum.repos.d/wazuh.repo << EOF
[wazuh]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
enabled=1
name=EL-\$releasever - Wazuh
baseurl=https://packages.wazuh.com/4.x/yum/
protect=1
EOF

```

### Installing the Wazuh manager

Install the Wazuh manager package:

``` yum install wazuh-manager```

### Enable and start the Wazuh manager service:

```
 systemctl daemon-reload
 systemctl enable wazuh-manager
 systemctl start wazuh-manager
 
 ```
Allow Wazuh Manager Service Ports Through The Firewall
```
firewall-cmd --permanent --add-port={514,1514,1515,1516}/tcp
firewall-cmd --permanent --add-port={514,1514}/udp
firewall-cmd --reload
```


* 514 - send collected events from syslogs

* 1514 - Send collected event from agents

* 1515 - Agents registration service

* 1516 - Wazuh cluster communications


## 2.Install JAVA JDK

install OpenJDK 8 JDK using yum, run this command:

``` sudo yum install java-1.8.0-openjdk-devel ```

## 3. Install the Elastic Stack

### Add the Elastic repository and its GPG key:

``` 
rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
cat > /etc/yum.repos.d/elastic.repo << EOF
[elasticsearch-7.x]
name=Elasticsearch repository for 7.x packages
baseurl=https://artifacts.elastic.co/packages/7.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
```

### Install the Elasticsearch package:

``` yum -y install elasticsearch-7.10.2 ```


### Configure Elasticsearch

```
vim /etc/elasticsearch/elasticsearch.yml
```
Make Elasticsearch to be able to listening on non loopback IP address. In here you can add current machine IP address. I’m not going to change the loopback IP, because this is the one Elastic node I’m going to setup. So, It doesn’t need to talk outside.
```
node.name: node-1
network.host: 127.0.0.1
http.port: 9200
cluster.initial_master_nodes: ["node-1"]

```
```
cat /etc/elasticsearch/elasticsearch.yml | grep -v "^#"

node.name: node-1
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 127.0.0.1
http.port: 9200
cluster.initial_master_nodes: ["node-1"]
```

###  JVM Option Configuration

Set initial/maximum size of total heap space. If your system has less memory. You should configure it to use small megabytes of ram.

```
vim /etc/elasticsearch/jvm.options

-Xms2g
-Xmx2g

```

### Enable and start the Elasticsearch service:

```
systemctl daemon-reload
systemctl enable elasticsearch.service
systemctl start elasticsearch.service
```

## 4. Install Kibana

Kibana is a flexible and intuitive web interface for mining and visualizing the events and archives stored in Elasticsearch. More info at Kibana.

### Install the Kibana package:

```
yum install -y kibana-7.10.2
```

### Create the /usr/share/kibana/data directory:

```
 mkdir /usr/share/kibana/data
 chown -R kibana:kibana /usr/share/kibana
 
```
### Install the Wazuh plugin for Kibana:

Install from URL:

```
 cd /usr/share/kibana/
 sudo -u kibana bin/kibana-plugin install https://packages.wazuh.com/4.x/ui/kibana/wazuh_kibana-4.2.5_7.10.2-1.zip
```

### Configure Kibana

Make Kibana to be able to listening on outside of the network and provide Elasticsearch instance IP address.

In this case I’m going to keep loopback IP for Elasticsearch, Since I don’t want to expose outside network

```
vim /etc/kibana/kibana.yml


server.port: 5601
server.host: "172.25.10.50"
elasticsearch.hosts: ["http://127.0.0.1:9200"]

```

### Enable and start the Kibana service:

```
systemctl daemon-reload
systemctl enable kibana.service
systemctl start kibana.service
```
## 5.Installing Filebeat

Filebeat is the tool on the Wazuh server that securely forwards alerts and archived events to Elasticsearch.

### Install the Filebeat package:

``` 
yum install filebeat
```

Download the preconfigured Filebeat configuration file used to forward the Wazuh alerts to Elasticsearch:

``` 
curl -so /etc/filebeat/filebeat.yml https://packages.wazuh.com/resources/4.2/open-distro/filebeat/7.x/filebeat_all_in_one.yml 

```

### Download the alerts template for Elasticsearch:

``` 
curl -so /etc/filebeat/wazuh-template.json https://raw.githubusercontent.com/wazuh/wazuh/4.2/extensions/elasticsearch/7.x/wazuh-template.json

```

``` 
chmod go+r /etc/filebeat/wazuh-template.json 

```

### Download the Wazuh module for Filebeat:

``` 
curl -s https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.1.tar.gz | tar -xvz -C /usr/share/filebeat/module

```

### Enable and start the Filebeat service:

```
systemctl daemon-reload
systemctl enable filebeat
systemctl start filebeat

```

#### To ensure that Filebeat is successfully installed, run the following command:

```
filebeat test output
```

An example response should look as follows:

Output
```
 elasticsearch: https://127.0.0.1:9200...
   parse url... OK
   connection...
     parse host... OK
     dns lookup... OK
     addresses: 127.0.0.1
     dial up... OK
   TLS...
     security: server's certificate chain verification is enabled
     handshake... OK
     TLS version: TLSv1.3
     dial up... OK
   talk to server... OK
   version: 7.10.2
   
   ```
   
   # CLIENT SIDE CONFIGURATION - Adding Wazuh Agent
   
   ## Wazuh agent
The Wazuh agent is multi-platform and runs on the hosts that the user wants to monitor. It communicates with the Wazuh manager, sending data in near real time through an encrypted and authenticated channel.

### Allow Wazuh-Agent Service Port Through The Firewall

```
514,1514 / udp

```

### Add the Wazuh repository

Import the GPG key:

```
rpm --import https://packages.wazuh.com/key/GPG-KEY-WAZUH
```

Add the repository:


```
cat > /etc/yum.repos.d/wazuh.repo << EOF


[wazuh]
gpgcheck=1
gpgkey=https://packages.wazuh.com/key/GPG-KEY-WAZUH
enabled=1
name=EL-\$releasever - Wazuh
baseurl=https://packages.wazuh.com/4.x/yum/
protect=1
EOF
```
## Deploy a Wazuh agent

```
WAZUH_MANAGER="10.0.0.2" yum install wazuh-agent

```

### Enable and start the Wazuh agent service.

```
systemctl daemon-reload
systemctl enable wazuh-agent
systemctl start wazuh-agent

```



## Add an agent - Add a CentOS agent

On the Manager:

Execute this command to add new agent to wazuh-manager.

```
/var/ossec/bin/manage_agents -a -n
```

### Output

```
****************************************
* Wazuh v3.11.1 Agent manager.         *
* The following options are available: *
****************************************
   (A)dd an agent (A).
   (E)xtract key for an agent (E).
   (L)ist already added agents (L).
   (R)emove an agent (R).
   (Q)uit.
Choose your action: A,E,L,R or Q: 
- Adding a new agent (use 'q' to return to the main menu).
  Please provide the following:
   * A name for the new agent:    * The IP Address of the new agent: Confirm adding it?(y/n): Agent added with ID 001.

manage_agents: Exiting.

```

### List The Agents
Execute this command to list the agent to get the ID of the added agent

```
/var/ossec/bin/manage_agents -l
```

### Extract Newly Added Agent’s Key

Execute this command with the agent “ID” and extract the new agent’s key. Cpy this key you will need it for the agent registration.

```
 /var/ossec/bin/manage_agents -e 001
 
 
 Agent key information for '001' is: 
MDAxIGNsMSAxNzIuMjUuMTAuMTAwIDU4N2Y2ZmYyMDA0NmY1NzZmMDc2ODJkOWFlMzM5ZmM1OWY0YzQzYjBhZTk0MjI3NTQyZTkxYTczZmEyZTk4Njk=

```

Now, Let’s head over to agent host.

On the Agent

### Import the Key To The Agent and Connect Agent To The Manger


```
/var/ossec/bin/manage_agents -i MDAxIGNsMSAxNzIuMjUuMTAuMTAwIDU4N2Y2ZmYyMDA0NmY1NzZmMDc2ODJkOWFlMzM5ZmM1OWY0YzQzYjBhZTk0MjI3NTQyZTkxYTczZmEyZTk4Njk=


Agent information:
   ID:001
   Name:cl1
   IP Address:172.25.10.100

Confirm adding it?(y/n): y
Added.

```

### Check for Wazuh Manager’s IP

Edit the Wazuh agent configuration in “/var/ossec/etc/ossec.conf” to add/change the Wazuh Manager Server IP address.

### Restart Wazuh Agent Service

```
 systemctl restart wazuh-agent
 systemctl status -l  wazuh-agent
```


# Troubleshooting

Kibana version can be checked by executing the following command:

```
cat /usr/share/kibana/package.json | grep version

Output

"version": "7.10.2",
```
The Wazuh version can be checked by executing the following command:

```
 /var/ossec/bin/wazuh-control info | grep WAZUH_VERSION
 
Output

WAZUH_VERSION="v4.2.5"
```
