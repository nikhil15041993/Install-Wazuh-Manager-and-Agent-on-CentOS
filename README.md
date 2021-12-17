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

### Enable and start the Elasticsearch service:

```
systemctl daemon-reload
systemctl enable elasticsearch.service
systemctl start elasticsearch.service
```

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
