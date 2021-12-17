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

### Installing Wazuh
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
