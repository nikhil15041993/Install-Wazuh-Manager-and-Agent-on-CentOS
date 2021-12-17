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
